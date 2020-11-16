---
title: Bitbucket CI for Laravel PHP applications
date: 2020-11-15 16:40:39
tags: 
- Bitbucket
- Gitlab
- Laravel
- PHP
- Devops
- CI/CD
- Build
- Yarn

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1600846337/posts/nyc-freedom-tower.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1600846337/posts/nyc-freedom-tower.jpg
thumbnailImagePosition: right
---

Adding A CI/CD process to my work flow is one of the really quick wins I do on every serious project I work on.

Whilst most of my personal work is hosted on Gitlab, a recent project I was working had its code in Bitbucket. This was my first time working with Bitbucket, so I wanted to document how I built assets, linted and ran tests using its pipelines. 

### What will these pipelines do?

By the end of this article you should have a working `bitbucket-pipelines.yml` file for your Laravel project which will do the following:
- Use `composer` to install your projects dependencies
- Run `php-cs-fixer` to enforce a code style
- Run `larastan` to run static analysis against the code base
- Run `php-cs-fixer` and `larastan` in parallel 
- Run `phpunit` to run our projects test suite
- For a production build `yarn run production`
- Allow us to manually trigger a deployment to production using [Laravel Deployer.](https://github.com/lorisleiva/laravel-deployer)

<!-- more -->

### Gitlab CI vs Bitbucket

If you've used [Gitlab CI](/blog/in-depth-gitlab-ci-cd-with-laravel-apps/) before working with bitbucket-pipelines should feel similar even if the configuration is slightly different. 

Here are the key differences I found:
- Instead of using a `.gitlab-ci.yml` file you'll need to create a `bitbucket-pipelines.yml` in the root of your project.
- Instead of defining stages bitbucket uses pipelines which are broken down by branches or tags  and then into "steps"

That's basically it! 

### Getting started

Start by adding a `bitbucket-pipeline.yml` file in the root of your project and add the following lines:

```yaml
image: edbizarro/gitlab-ci-pipeline-php:7.4

pipelines:
  default:
    - step: *composer
    - step: *php-cs-fixer
    - step: *phpstan
    - step: *phpunit

```

Here we've picked a base docker image which has enough of the utilities we'll need to run our pipeline steps against PHP 7.4. 

We've also set up a default pipeline with 4 steps. 

This will be the pipeline that gets run when we push code to our repository and there isn't a specific set of rules defined for the given branch / tag that has been pushed to.
Each step has a yaml anchor associated with at the moment (step: *anchor-name). We'll be making heavy use of these anchors throughout our pipelines in order to give us reusable steps.

Its worth noting that these steps get run in sequence - one after the other, we'll come on to how we can run certain jobs in parallel shortly.

With our pipe steps defined we now need to define what those steps actually do, lets start by creating a new root level item called `definitions` which has a key of `steps`.
Now we can start to flesh out the `*composer` step defined in our pipeline from before.

```yaml
definitions:
  steps:
    - step: &composer
        name: Composer
        caches:
          - composer
        script:
          - php -v
          - composer install --prefer-dist --no-ansi --no-interaction --no-progress --no-scripts --no-suggest
          - cp .env.example .env
          - php artisan key:generate
          - composer dump
        artifacts:
          - vendor/**
          - .env
```

Here we create a step called `&composer` that the previously defined `*composer` symbol refers to. It does the following:

- Checks the PHP version
- Runs composer install
- Creates a `.env` file
- Creates an application key in the `.env` file
- Caches all of the composer dependencies that have been downloaded for 1 week (bitbucket default)
- Creates artifacts of the .env file and vendor folder that will be passed to every subsequent step in the pipeline

With us having got through that step the rest of the steps in this pipeline should be pretty easy to follow:
```yaml
    - step: &php-cs-fixer
        name: PHP-CS-Fixer
        script:
          - vendor/bin/php-cs-fixer fix --config=.php_cs.php --verbose --diff --dry-run
    - step: &phpstan
        name: PHPStan
        script:
          - vendor/bin/phpstan analyse
    - step: &phpunit
        name: PHPUnit
        script:
          - vendor/bin/phpunit --exclude-group=integration
        artifacts:
          - storage/logs/*.log
```

This gives us a basic sequential pipeline that will run on every push to our bitbucket repo where each step is after the former step succeeds.

### A more advanced configuration

Lets add something to build our frontend assets. I'm going to use Yarn however you can easily amend these steps to use NPM. First we'll need to add some new steps do our definitions object:

```yaml
definitions:
  steps:
    - step: &yarn
        name: Yarn
        caches:
          - node
        script:
          - yarn --version
          - yarn install --pure-lockfile
        artifacts:
          - node_modules/**
    - step: &yarnDev
        name: Build development assets
        script:
          - yarn --version
          - yarn run development --progress false
        artifacts:
          - public/css/**
          - public/js/**
          - public/mix-manifest.json
    - step: &yarnProd
        name: Build production assets
        script:
          - yarn --version
          - yarn run production --progress false
        artifacts:
          - public/css/**
          - public/js/**
          - public/mix-manifest.json
```

Here we'll define a step to run yarn to pull in all of our front end packages and cache them for use in our asset build steps which we'll define next.
This is broken out into 2 separate steps one for our development builds and one for production. We'll cache their output to be deployed to our target environment. 

If we wanted to run a different set of steps for a given branch or tag we can create new keys under the "pipelines" root level item:

```yaml
pipelines:
  default:
    ...
  branches:
    develop:
      - step: *composer
      - step: *yarn
      - step: *php-cs-fixer
      - step: *phpstan
      - step: *phpunit
      - step: *yarnDev
  tags:
    release/*:
      - step: *composer
      - step: *yarn
      - step: *php-cs-fixer
      - step: *phpstan
      - step: *phpunit
      - step: *yarnProd
```

Those of us familiar with gitlab will be used to being able to run "stages" or groups of tasks in parallel. We can do that here to by putting steps under the `parallel` keyword:

```yaml
pipelines:
  branches:
    develop:
      - parallel:
        - step: *composer
        - step: *yarn
      - parallel:
          - step: *php-cs-fixer
          - step: *phpstan
          - step: *phpunit
      - step: *yarnDev
```

Finally if we wanted to manually trigger the running of the step we can use the trigger keyword:

```yaml
pipelines:
  branches:
    master:
      - parallel:
        - step: *composer
        - step: *yarn
      - parallel:
          - step: *php-cs-fixer
          - step: *phpstan
          - step: *phpunit
      - step: *yarnProd
      - step:
          trigger: manual
          deployment: production
          name: Deploy to production
          script:
            - php artisan deploy production -s upload
```

> The "php artisan deploy" command is what I use to deploy simple sites which ive [written about here.](https://www.talvbansal.me/blog/easy-push-to-deploy-for-laravel-with-gitlab-ci/)

Notice the deployment keyword on the final step. Bitbucket allows us to track code changes between environments as well as link in any JIRA issue (if you happen to be using JIRA).
