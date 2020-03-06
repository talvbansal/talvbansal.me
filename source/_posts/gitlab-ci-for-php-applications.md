---
title: Gitlab CI for PHP applications
date: 2017-02-24 16:24:17
tags: 
- Gitlab
- Laravel
- PHP
- Devops
- CI/CD

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1555352188/posts/cancun-sunset.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1555352188/posts/cancun-sunset.jpg
thumbnailImagePosition: right
---

** Update March 2020 - I've given written a newer more updated version of this article, [check it out over here.](/blog/in-depth-gitlab-ci-cd-with-laravel-apps/) **

## Introduction

Gitlab provides some good documentation on getting build runners for your projects set up which can be found [here](https://about.gitlab.com/2016/03/01/gitlab-runner-with-docker/). However I haven't found a good article on setting up Gitlab CI for PHP applications yet. 

This article is not going to discuss setting up shared runners on Gitlab since I felt the documentation provided by them in the link above was easy enough to follow, what this I am going to focus on is the `.gitlab-ci.yml` configuration that I'm currently using in my projects to do the following: 

- Run PHP CS against our code base to ensure that it conforms to PSR-2
- Run PHPUnit against our code base to ensure that our unit tests pass against different versions of PHP (5.6, 7.0, 7.1)

<!-- more -->

## Preparation

I'm using Gitlab CI to test (deploy in the next article) a number of Laravel applications which not only contain 3rd party packages but a number of our own internal packages hosted within our Gitlab CE instance.

There are a few of things we need to before we start:
- Make sure that both HTTPS and SSH git cloning are enabled. It seems that the runners use the HTTPS protocol to clone the projects into the docker container. This option is the default option and can be found by click on the **Administration Area -> Settings Cog -> Visibility and Access Controls -> Enabled Git access protocols**
- Make sure we have access to a **private** key portion of a user that can log into Gitlab. We usually give Gitlab our public key to authenticate us, however for this we'll need our docker containers to act as us and therefore they'll need a private key to authenticate with when contacting Gitlab when cloning down our project. You'll need to add the private portion of the ssh key as a **Secret Variable** with the name of **SSH_PRIVATE_KEY** and the value of the Private key. ** Project -> Settings Cog -> CI / CD Pipelines -> Secret Variables**
- Adding [PHP Code Sniffer](https://github.com/squizlabs/PHP_CodeSniffer) to our projects dev dependencies:
	-  `composer require squizlabs/php_codesniffer --dev`
 

## Getting Started

Lets get straight into the good stuff - 4 main files that are necessary to make my builds run that exist all within my project root:

- `phpunit.xml` - I'm not going to cover this since Laravel ships with one 
- `phpcs.xml` - the PHP CS configuration file
- `.gitlab-ci.yml` - the core configuration file for Gitlab CI
- `.gitlab-ci.sh` - an additional configuration file for building our docker image

And below are their contents:

#### phpcs.xml

My PHP CS file is pretty simple it just enforces the PSR-2 coding standard within the `app` folder of my project, but should you need to tailor your rules they can be added here too.
```xml
<?xml version="1.0"?>
<ruleset name="PSR2">
    <description>The PSR2 coding standard.</description>
    <rule ref="PSR2"/>

    <file>app/</file>

    <exclude-pattern>vendor</exclude-pattern>

</ruleset>
```

#### .gitlab-ci.yml
From the Gitlab link above:
> This file specifies how the build environment should be set up and what commands to be executed to build, test, and deploy our project in a series of jobs that can be parallelized.

```yml
# Select a base image to start working with...
image: tetraweb/php:7.0

# Configure the base image
before_script:
  - bash .gitlab-ci.sh

# Select folders to cache...
cache:
  paths:
    - vendor/

services:
  - mysql

variables:
  WITH_XDEBUG: "1"
  MYSQL_ROOT_PASSWORD: secret
  MYSQL_DATABASE: homestead
  MYSQL_USER: homestead
  MYSQL_PASSWORD: secret

# Define pipline stages
stages:
  - syntax
  - tests

phpcs:
  stage: syntax
  script:
    - ./vendor/bin/phpcs --error-severity=1 --warning-severity=8 --extensions=php

php-5.6:
  type: tests
  image: tetraweb/php:5.6
  script:
    - php vendor/bin/phpunit --colors=never --coverage-text

php-7.0:
  type: tests
  image: tetraweb/php:7.0
  script:
    - php vendor/bin/phpunit --colors=never --coverage-text

php-7.1:
  type: tests
  image: tetraweb/php:7.1
  script:
    - php vendor/bin/phpunit --colors=never --coverage-text
  allow_failure: true
```


#### .gitlab-ci.sh
This file as described above will configure the environment and handle setting up the SSH agent. One thing to note is part way down you'll need to change `ssh git@gitlab.yourserver.com` to match your server name.

```bash
#!/bin/bash

# Install dependencies only for Docker.
[[ ! -e /.dockerinit ]] && [[ ! -e  /.dockerenv ]] && exit 0

# Enable modules
docker-php-ext-enable zip
docker-php-ext-enable pdo_mysql
docker-php-ext-enable mcrypt

# install ssh-agent
which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )

# run ssh-agent
eval $(ssh-agent -s)

# add ssh key stored in SSH_PRIVATE_KEY variable to the agent store
ssh-add <(echo "$SSH_PRIVATE_KEY")

# disable host key checking (NOTE: makes you susceptible to man-in-the-middle attacks)
# WARNING: use only in docker container, if you use it with shell you will overwrite your user's ssh config
mkdir -p ~/.ssh
echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config

# try to connect to GitLab.com
ssh git@gitlab.yourserver.com

# Install project dependencies.
composer install --no-suggest --prefer-dist

# Copy over testing configuration.
cp .env.example .env

# Generate an application key. Re-cache.
php artisan key:generate
php artisan config:cache
php artisan config:clear

```

## Wrapping up

If you're going to come across issues with Gitlab CI it'll be with the SSH config in my experience. However hopefully with the preparation and scripts above you shouldn't run into these problems.

Remember that you'll need to add the **SSH_PRIVATE_KEY** Secret variable **FOR EACH** of your your projects - this caught me out a couple of times and hopefully should save you A LOT of time!