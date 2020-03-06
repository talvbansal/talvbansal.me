---
title: In depth Gitlab CI/CD with Laravel Apps
date: 2020-03-06 10:13:49
tags: 
- Gitlab
- Laravel
- PHP
- Devops
- CI/CD
- Build
- Deploy
- Horizon

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1583496063/posts/long-island-city-skyline.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1583496063/posts/long-island-city-skyline.jpg
thumbnailImagePosition: right
---

## Introduction
 
My most read articles on this blog are about [Gitlab CI/CD](/all-tags/#Gitlab-list) with PHP. They cover a basic linting, testing and crude deploying process.
 
Today I want to look at my current CI/CD process for my Laravel projects in more depth. Currently the pipelines of my projects might vary slightly but is very similar to this:
 
 ![Gitlab Pipeline](https://res.cloudinary.com/www-talvbansal-me/image/upload/v1583495330/posts/gitlab-ci-pipelines.png)
 
 So there are 5 main stages in the process:
 - **Preparation** - The pulling down of dependencies and storing them in an artifact
 - **Syntax** - Check code syntax
 - **Testing** - Run unit tests
 - **Building** - Build assets
 - **Deployment** - Deploying to an appropriate server

The stages are processed in order with each stage containing one or many tasks. Should one task fails in a stage then the whole pipeline stops and is marked as failed.

To run the pipelines I make use of gitlabs free tier which gives you access to 2000 shared minutes per month as well as a runner on a server I have. More about setting that up can be found [here](/blog/maximising-gitlab-ci-s-free-tier/).

<!-- more -->

## Getting started

Lets start by creating a config file and defining these stages.

In your project root create a new file called `.gitlab-ci.yml` and add the following lines:

```yaml
image: edbizarro/gitlab-ci-pipeline-php:7.4

stages:
  - preparation
  - building
  - syntax
  - testing
  - deploy
```

So firstly we're picking a base docker image to use for each of the tasks in the stages. I've been using the [edbizarro](https://github.com/edbizarro/gitlab-ci-pipeline-php) image for a while now since they come with everything I've needed to test my apps so far:

> All images come with PHP (with all laravel required extensions), Composer (with hirak/prestissimo to speed up installs), Node and Yarn.

This image is overridable at the task level but for every task that does not explicitly have an `image:` defined it will fall back to this image.

Next we have the stages we want to create which is a simple list that we described before.

## Stages

Now I'll try show and break down the contents of each stage that has just been defined.

Before the doing so lets add a cache to help speed up builds. 

```yaml
cache:
  key: "$CI_JOB_NAME-$CI_COMMIT_REF_SLUG"
```

#### Cache vs Artifacts

[This article](https://docs.gitlab.com/ee/ci/caching/#cache-vs-artifacts) explains the difference between what gitlab calls cache and artifacts. The top line is Artifacts can be passed between stages where as cache is used on the same server for subsequent runs of the same task. 

### Preparation

```yaml
composer:
  stage: preparation
  script:
    - composer install --prefer-dist --no-ansi --no-interaction --no-progress --no-scripts --no-suggest
    - cp .env.example .env
    - php artisan key:generate
  artifacts:
    paths:
      - vendor/
      - .env
    expire_in: 1 days
    when: always
  cache:
    paths:
      - vendor/

yarn:
  stage: preparation
  script:
    - yarn install --pure-lockfile
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 days
    when: always
  cache:
    paths:
    - node_modules/
```

In the previous articles what I'd been doing is fetching dependencies like composer install and yarn on pretty much every task on the pipeline. This wasted precious CI minutes and was inefficient. 

Now I run those tasks once store them as what artifacts and then fetch those artifacts in subsequent tasks on the pipeline.

In these two tasks I also cache the `vendor` and `node_modules` too so for subsequent pipelines being run on. Since these jobs are being allocated to either my server running the gitlab runner or the shared runners this cache doesn't get hit very often. If you were to completely restrict your builds to run on a limited set of runners you'd see it being hit regularly giving you an additional improvement.

### Building

```yaml
build-dev-assets:
  stage: building
  dependencies:
    - yarn
  script:
    - echo "PUSHER_APP_KEY=$PUSHER_TEST_APP_KEY" >> .env
    - yarn run dev --progress false
  artifacts:
    paths:
      - public/css/
      - public/js/
      - public/fonts/
      - public/mix-manifest.json
    expire_in: 1 days
    when: always
  except:
    - master

build-production-assets:
  stage: building
  dependencies:
    - yarn
  script:
    - echo "PUSHER_APP_KEY=$PUSHER_LIVE_APP_KEY" >> .env
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
Here are two tasks which are almost the same. They're both updating the `.env` file, then both running a yarn command to build the compiled front end assets and then creating artifacts of them. The key difference is one runs `yarn production` and is only run on the `master` branch where as the other runs `yarn dev` and runs on everything *except* `master`.

I've written before about when you might need to update values on your [.env](/blog/updating-laravels-env-file-with-production-credentials-in-gitlab-ci-with-laravel-mix/) file, here is an example of where I do that and how to create the variables being used above.

### Syntax

```yaml
php-cs-fixer:
  stage: syntax
  dependencies:
    - composer
  script:
    - ./vendor/bin/php-cs-fixer fix --config=.php_cs.php --verbose --diff --dry-run

eslint:
  stage: syntax
  dependencies:
    - yarn
  script:
    - yarn eslint
```

These are the two tasks I run to check the syntax of the code being pushed up. One for php code and the other for the js code each pulling down the relevant artifacts to suit.

### Testing

```yaml
phpstan:
  stage: testing
  script:
    - php artisan code:analyse

phpunit:
  stage: testing
  dependencies:
    - composer
  script:
    - php -v
    - php artisan storage:link
    - sudo cp /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini /usr/local/etc/php/conf.d/docker-php-ext-xdebug.bak
    - echo "" | sudo tee /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
    - ./vendor/phpunit/phpunit/phpunit --version
    - php -d short_open_tag=off ./vendor/phpunit/phpunit/phpunit -v --colors=never --stderr --exclude-group integration
    - sudo cp /usr/local/etc/php/conf.d/docker-php-ext-xdebug.bak /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
  artifacts:
    paths:
      - ./storage/logs
    expire_in: 1 days
    when: on_failure
```

Here I use [Larastan](https://github.com/nunomaduro/larastan) to perform static analysis on my projects in one task and phpunit in a second task. In the phpunit task the log file of the project is stored for a day when the task fails so we can refer back to it if necessary.

### Deploy

```yaml
# Add a `.` in front of a job to make it hidden.
# Add a `&reference` to make it a reusable template.
# Note that we don't have dashes anymore.
.init_ssh_live: &init_ssh_live |
  mkdir -p ~/.ssh
  echo -e "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
  chmod 600 ~/.ssh/id_rsa
  [[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config

live:
  stage: deploy
  script:
    - *init_ssh_live
    - php artisan deploy my-project.com -s upload
  environment:
    name: live
    url: https://my-project.com
  tags:
    - deploy
  only:
    - master
```

I use [Laravel deployer](https://github.com/lorisleiva/laravel-deployer) to handle the deployment of my projects. I intend to do a whole other article about how I configure it and what needs to be done on the server to make this work.

As for this task itself it very simply adds the private key of a user who has access to the server into the docker container being run. Then uses deployer to handle everything else.

I usually have a second version of this that deploys the develop branch of my projects to a staging server for our teams to test out the latest code changes before they are merged into master too.

## Wrapping up

That concludes the breakdown of my current `.gitlab.yml` file. If you felt that this lacked clarity please reach out to me on twitter [@talv](https://twitter.com/talv) and let me know how I can improve this.


