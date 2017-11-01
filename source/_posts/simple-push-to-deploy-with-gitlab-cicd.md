---
title: Simple push to deploy with gitlab CI/CD
date: 2017-05-24 16:15:27
tags:
- Gitlab
- PHP
- Laravel
- Devops

autoThumbnailImage: yes
coverImage: ./westminister-sunset.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: westminister-sunset.jpg
thumbnailImagePosition: right
---

## Introduction

This article follows directly on from my [previous article](https://talvbansal.me/blog/gitlab-ci-for-php-applications) on adding gitlab's CI to my PHP projects.

Once we're happy with our code being automatically tested we might want it to be automatically deployed to a testing server / environment or maybe even production. 
<!-- more -->
Ultimately the deployment is just some script or set of commands that the runner will call. 

A super simple deployment process might involve the following steps:
- SSH into you remote server
- Change to project directory
- Put application into maintenance mode 
- Undo any local changes 
- Checkout the correct branch 
- Pull in new changes
- Install new dependencies
- Migrate database
- Take application out of maintenance mode

Obviously there are a number of issues with this process if you were using it in production for example: 
- Requires the project to be checked out in the first place
- No back out process if a step fails

Some of these issues could be fixed using a deployment tool like [Capistrano](http://capistranorb.com/) - (if you have any other recommendations id love to hear them!).

*Scroll to the bottom for a complete sample script to add to your `gitlab-ci.yml`*

## Getting started with Gitlab CI/CD

Start by defining a new pipeline stage called "deploy" within your projects `gitlab-ci.yml` file:

```bash
...

stages:
   - syntax
   - tests
   - deploy

...
```

Next create a new section where we can define what will occur during the `deploy` stage and then provide it with a base docker image to use:

```bash
deploy-master:
    stage: deploy
	image: tetraweb/php:7.0
```

Next this section we need to give our docker image an ssh key needed to get into our remote server. This can be done in the `before_script` section. 

```bash
before_script:
	- mkdir -p ~/.ssh
	- echo -e "$DEPLOY_KEY" > ~/.ssh/id_rsa
	- chmod 600 ~/.ssh/id_rsa
	- '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
```

In the section above pay attention to the $DEPLOY_KEY variable Within Gitlab we'll need to set up a **Secret Variable** that holds the contents of the private portion of an SSH key that has access to your deployment server.

>Its highly recommended to **NOT** use your main user to deploy your project code with, for more information on how to do that check out [Chris Fidao's ](https://twitter.com/fideloper) excellent server management book : [Servers For Hackers](https://serversforhackers.com/).

To add a new secret variable to a Gitlab project, go to the Project Page -> Settings -> CI / CD Pipelines. Then scroll down to **Secret Variables**. Call the variable `DEPLOY_KEY` and paste the **PRIVATE** portion of your deployment users SSH key into the `Value` field.

### Deployment

The commands to deploy outlined at the top of this article can be translated into:

- ssh user@server-name.com
- cd /project/path
- php artisan down
- git checkout .
- git checkout master
- git pull
- composer install
- php artisan migrate
- php artisan up

Once we've ssh'd into the remote server we can only run a single command before the process will terminate, so we can join our commands together using `&&`:

```bash
script:
	- ssh user@server-name.com "
	    cd /var/www/ &&
		php artisan down &&
		git checkout . &&
		git checkout master &&
		git pull &&
		composer install &&
		php artisan migrate &&
		php artisan up
		"
```

### Limiting releases to a specific branch

If we put everything together now and used the resulting script we'd have something that deploys code from any branch any time we pushed to Gitlab. Not ideal since we only want code to be deployed when we merge into a particular branch, say master or develop. We can limit when a section of a stage is run using the `only` command.

```bash
only:
	- master
```

## Putting it all together:

Heres an almost complete deployment script (with the syntax and tests sections omitted): 

```bash

stages:
   - syntax
   - tests
   - deploy
           

deploy-master:
    stage: deploy
        image: tetraweb/php:7.0       
                        
    before_script:
            - mkdir -p ~/.ssh
            - echo -e "$DEPLOY_KEY" > ~/.ssh/id_rsa
            - chmod 600 ~/.ssh/id_rsa
            - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
        
        script:
            - ssh user@server-name.com "
                cd /var/www/ &&
                php artisan down &&
                git checkout . &&
                git checkout master &&
                git pull &&
                composer install &&
                php artisan migrate &&
                php artisan up
            "
                                    
        environment:
            name: production
            
        only:
            - master
````


Hopefully this has been useful, if you have suggestions on how i can improve the article / process I'd love to hear from you.