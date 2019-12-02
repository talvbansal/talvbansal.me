---
title: Updating Laravel's .env file with Production Credentials in Gitlab CI with Laravel Mix
date: 2019-12-02 20:58:47
tags:
- Laravel
- Laravel Echo
- Laravel Mix
- Javascript
- Tooling
- Gitlab
- CI/CD
- Pusher

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1575320794/posts/india-huyumans-tomb-chains.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1575320794/posts/india-huyumans-tomb-chains.jpg
thumbnailImagePosition: right
---

### Introduction
Laravel apps read sensitive information from their `.env` file.

Recently I found out that Laravel Mix can [pass values from the same `.env` file](https://laravel.com/docs/master/mix#environment-variables) to the js portion of your app as long as they are prefixed with `MIX_`  

I use Gitlab ci pipelines to build production assets so that I dont need that additional tooling on the production servers the main one being:

- `yarn run production`

This is preceded with `cp .env.example .env` meaning when the build commands are being run, they are going to use values from the `.example.env` file.

If your project doesn't make use of anything from the `.env` file then this is totally fine, however in scenarios where you do, since production applications will almost certainly  have different `.env` values to those in the `.example.env` file (Never commit credentials to source control!) the resulting file will have been built with the wrong credentials.

In this article I'm going to show how you can use gitlab CI to build those assets with updated environmental variables so that they function as expected when deployed to your production servers.

<!-- more -->

### Laravel's stock example

Lets start by looking at a real example of how these environmental variables are configured and used.

The default Laravel `.env` has the following section in that is used to configure the [Pusher.com](www.pusher.com) library.

```bash
PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=eu

MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```

The second part says for the `MIX_PUSHER_APP_KEY` key use the same value as the `PUSHER_APP_KEY` key and the same for `MIX_PUSHER_APP_CLUSTER` to use `PUSHER_APP_CLUSTER` as its value.

So if we set `PUSHER_APP_KEY` to be "TEST_PUSHER_KEY" then `MIX_PUSHER_APP_KEY` will also have the value of "TEST_PUSHER_KEY".

Those values are used in the `resources/js/boostrap.js` file to configure the Laravel Echo pusher subscriber:

```javascript
window.Echo = new Echo({
	broadcaster: 'pusher',
	key: process.env.MIX_PUSHER_APP_KEY,
	cluster: process.env.MIX_PUSHER_APP_CLUSTER,
	encrypted: true
});
```

So when `yarn run production` is run the key and cluster properties will be populated with the values in the `.env` file. 

### Updating the .env in Gitlab CI

Now lets look at the relevant parts of the projects `.gitlab.yml` file:

```yaml
# .gitlab.yml
image: edbizarro/gitlab-ci-pipeline-php:7.3

stages:
  - preparation
  - building
...

yarn:
  stage: preparation
  script:
    - yarn --version
    - yarn install --pure-lockfile
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 days
    when: always
  cache:
    paths:
    - node_modules/

build-production-assets:
  stage: building
  dependencies:
    - yarn
  script:
    - cp .env.example .env
    - yarn --version
    - yarn run production --progress false
  artifacts:
    paths:
      - public/css/
      - public/js/
      - public/fonts/
      - public/mix-manifest.json
    expire_in: 1 days
    when: always
  only:
    - master
```

There are 2 stages, *preparation* then *building*. 

The preparation stage simply runs `yarn install`  and downloads all of the projects JS dependencies from the NPM registry. Then it creates a gitlab artifact that its keeps for 1 day. 

> In Gitlab, artifacts are objects that can be passed between build stages, where as cache is used to speed up things like npm / yarn / composer installs on subsequent runs.

With the dependencies downloaded we no longer have to waste time or CI minutes in other stages for them to be downloaded again. 

The `build-production-assets` stage is where we want to focus.

The dependencies section tells the stage to download the artifacts generated in the stage named yarn.
```yaml
  dependencies:
    - yarn
```

The script section is where Laravel Mix is being run and our production assets are being generated.
```yaml
  script:
    - cp .env.example .env
    - yarn --version
    - yarn run production --progress false
```

What we want to do is kind of like this:

```yaml
  script:
    - cp .env.example .env
# *** write our production values into the .env file here ***
    - yarn --version
    - yarn run production --progress false
```

Luckily its not to difficult to do, however as much as I love Gitlab and Gitlab CI it sometimes feels like its got too much going on and I've forgotten where I can add production information to Gitlab CI far too many times.

So here's a quick gif of where to go:

<blockquote class="imgur-embed-pub" lang="en" data-id="ZNwEWqG"><a href="//imgur.com/ZNwEWqG">Add Environmental Variable to Gitlab CI</a></blockquote><script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

Within the project page on gitlab, in the sidebar under set the settings section open the CI / CD page. From here scroll down to the `Variables` section and expand it.

From here we can begin to add keys and values that will be exposed to our gitlab ci runner. 

Using our Pusher example from earlier. We know that if we were to update the `PUSHER_APP_KEY` key then the `MIX_PUSHER_APP_KEY` that gets used by Laravel mix will automatically be updated.

```bash
PUSHER_APP_KEY=
MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
```

Our variables in Gitlab CI don't need to match, so lets add a new variable to the gitlab runner for our production Pusher Keys called `PUSHER_LIVE_APP_KEY` and give it the value of our Pusher production key. Remember to save the variable.

![Gitlab Environmental Variables](https://res.cloudinary.com/www-talvbansal-me/image/upload/v1575327081/posts/gitlab-ci-enviromental-variables.png)

> Since we might be building for multiple environments in our pipelines I like to include the product and environment in my variable names whilst keeping them close to the variable im hoping to replace. Gitlab's UI is a little congested so I tend to put the environment as the second word so the product is quicker to scan for me.

With the environmental variable in place we now need to use it in our pipeline.

Trying to parse the `.env` file for the correct key and then update it with the variable is beyond my command line skill set however, if you append a key that already exists to the bottom of an `.env` the [Dotenv library used by Laravel](https://github.com/vlucas/phpdotenv) picks up the last instance. 

So lets update our .gitlab.yml to do that!


```yaml
  script:
    - cp .env.example .env
    - echo "PUSHER_APP_KEY=$PUSHER_LIVE_APP_KEY" >> .env
    - yarn --version
    - yarn run production --progress false
```

And that is it! Next time you run a pipeline the environmental variable from gitlab will be appended to the bottom of your projects `.env` file and then picked up by Laravel Mix when it does its magic!

