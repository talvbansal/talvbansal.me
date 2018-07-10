---
title: Testing your Gitlab CI config locally
date: 2018-04-20 11:29:30
tags:
- Gitlab
- PHP
- PHP-7.2
- Docker
- Devops
- CI/CD

autoThumbnailImage: yes
coverImage: ./mexico-monument.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: mexico-monument.jpg
thumbnailImagePosition: right
---
# Introduction
Recently at work we started using PHP7.2 for all of our new projects. We'd previously been using the [TetraWeb docker images](https://github.com/TetraWeb/docker) for our Gitlab CI needs, however they only (at the time of writing) have a docker images for versions up to PHP7.1.

I tasked one of the junior members of my team with upgrading one of our projects and came back to our repositories commit history looking like this:

![Gitlab commit history](/blog/testing-gitlab-ci-config-locally/gitlab-commit-history.png)

Kind of ugly and not so useful when scanning our commit history. 

In this article I want to show how we can test our Gitlab CI config locally whilst we're working out how to set it up properly before we push it to Gitlab as well as keep our commit history cleaner in the process.

<!-- more -->
# Setup
Before starting in order to run the isolated docker images we'll need to make sure that docker is installed on your machine. 
Since I primarily work in Ubuntu the first parts of the guide will be specific to Ubuntu but the guide for OSX or windows should be easy to find online.

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt update
sudo apt install -y docker-ce
```

Next we'll need to download the gitlab-runner, this is the same runner thats used on our gitlab server to read and execute the jobs defined in our `.gitlab-ci.yml` file. We'll also create a user for the gitlab-runner to run as.

```bash
sudo wget -O /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
sudo chmod +x /usr/local/bin/gitlab-runner
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start
```

# Using the Gitlab Runner
With everything set up we can change to a directory which has a `.gitlab-ci.yml` file in and use the runner to run one of the build commands defined.

For example if we had a `tests` stage which had the following command defined:

```
php-7.2:
  type: tests
  script:
    - ./vendor/bin/phpunit --colors=never --coverage-text
```

We can run it using the `gitlab-runner` as follows:

```bash
gitlab-runner exec docker php-7.2
```

If you get a permission denied error when running the above command you may need to run it as sudo. This is because the docker socket has the following permissions by default `root:docker` and since we're running the `gitlab-runner` as your local user it wont have permission to connect to docker.

Thats it! - If everything runs as expected you should see your command being executed on the terminal as it would be within gitlab!


# Notes
Since a number of projects that I work on reference other private packages that exist on our self hosted gitlab instance I came across another problem when trying to run the `gitlab-runner`. Gitlab was looking for a private key to authenticate against when trying to perform a clone. We'd normally solve this issue on the hosted instance by setting a SSH_PRIVATE_KEY secret variable within a projects CI/CD config - ([see here for a guide on that](blog/gitlab-ci-for-php-applications))). However since we're running this on our machine that config does not exist.

We can provide thats SSH_PRIVATE_KEY variable to our `.gitlab-ci.yml` config by passing it as a commandline argument:

```bash
sudo gitlab-runner exec docker php-7.2 --env SSH_PRIVATE_KEY="-----paste your private key here------"
```

Be careful when doing the above since by default your private key will get written into your shell's history file. You could use something like `HISTIGNORE=gitlab-runner exec docker *` to stop all `gitlab-runner exec docker` commands from being stored in history.

