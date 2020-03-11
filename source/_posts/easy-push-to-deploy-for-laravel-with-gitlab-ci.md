---
title: Easy push to deploy for Laravel with Gitlab CI
date: 2020-03-10 10:13:49
tags: 
- Gitlab
- Laravel
- PHP
- Devops
- CI/CD
- Build
- Deploy
- Horizon
- AWS

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1583496063/posts/franklin-street-train.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1583496063/posts/franklin-street-train.jpg
thumbnailImagePosition: right
---
Recently I wrote about [my current Gitlab CI process](/blog/in-depth-gitlab-ci-with-laravel-apps/), when it came to the deployment part of the process I showed how I was handling it using a tool called [Laravel deployer](https://github.com/lorisleiva/laravel-deployer) but I didn't breakdown what laravel deployer was doing and how I had it configured.

The Laravel deployer docs are pretty good however I found a couple of server config issues that I always find myself referring back to when setting up auto deployment. Mostly to automatically restart Laravel Horizon and restarting Php-fpm without needing sudo privileges.

Lets imagine I was going to a set up Gitlab CI / CD for a fictional project hosted over on the fictional domain of `deployer.talvbansal.com` with a [real repository here](https://gitlab.com/talvbansal/deployer-example) that is hosted on somewhere like Linode, Digital Ocean or even AWS.

<!--more-->

### Getting started

After creating a new project and changing to its directory on your local machine, the first thing we need to do is add laravel deployer to our project and the initialise it.
```bash
composer require lorisleiva/laravel-deployer
php artisan deploy:init
```

The init step asks a number of questions to help write out the default configuration file, mine look something like this:
![Laravel deployer init questions](https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1024/v1583860867/posts/laravel-deployer-init.png)

This will create a new file which has your base deployment configuration in at `config/deployer.php`.

```php
return [
    'default' => 'basic',
    'strategies' => [],
    'hooks' => [
        'start' => [],      
        'build' => [],
        'ready' => [
            'artisan:storage:link',
            'artisan:view:clear',
            'artisan:cache:clear',
            'artisan:config:cache',
            'artisan:migrate',
            'artisan:horizon:terminate',
        ],     
        'done' => [
            'fpm:reload',
        ],
        'success' => [],
        'fail' => [],
        'rollback' => [
            'fpm:reload',
        ],
    ],

    'options' => [
        'application' => env('APP_NAME', 'Laravel'),
        'repository' => 'git@gitlab.com:talvbansal/deployer-example.git',
        'php_fpm_service' => 'php7.4-fpm',
    ],

    'hosts' => [
        'deployer.talvbansal.com' => [
            'deploy_path' => '/media/websites/deployer.talvbansal.com',
            'user' => 'root',
        ],
    ],

    'localhost' => [],
    'include' => [],
    'custom_deployer_file' => false,
];
```

Here we'll need to change our `hosts.deployer.talvbansal.com.user` to a user other than `root` for example `deployer` that has access to our projects path on the target deployment server.

Next lets add `php_cs_fixer` so that our ci pipeline has something to do. I use a wrapper made by the [madewithlove](https://madewithlove.com/) team which has a Laravel preset:

```bash
composer require madewithlove/php-cs-fixer-config
```

Next create the config file and put the following contents into it

```bash
nano .php_cs.php
```

```php
<?php
require 'vendor/autoload.php';

return Madewithlove\PhpCsFixer\Config::forLaravel()->mergeRules([
    'psr0' => false,
]);
```

And then run the CS fixer so that we can be confident that our pipeline will pass.
```bash
vendor/bin/php-cs-fixer fix --config .php_cs.php
```

### Gitlab CI/CD

Next lets add a `.gitlab-ci.yml` file to our project. Rather than repeat the contents of the [previous article]('/blog/in-depth-gitlab-ci-with-laravel-apps/'), here's a link to a file being used:
 
[.gitlab.yml](https://gitlab.com/talvbansal/deployer-example/-/blob/master/.gitlab-ci.yml)

The key part we're interested in here is this section:

```yaml
.init_ssh_live: &init_ssh_live |
  mkdir -p ~/.ssh
  echo -e "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
  chmod 600 ~/.ssh/id_rsa
  [[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config

live:
  stage: deploy
  script:
    - *init_ssh_live
    - php artisan deploy deployer.talvbansal.com -s upload
  environment:
    name: live
    url: https://deployer.talvbansal.com
  tags:
    - deploy
  only:
    - master
```

Which says run the deploy command to `deployer.talvbansal` using the [upload](https://github.com/lorisleiva/laravel-deployer/blob/master/docs/strategy-upload.md) strategy, only when code is pushed to the `master` branch and only use gitlab runners that have been tagged with `deploy`.

We use the upload strategy to make use of the artifacts we've built along the way of the pipeline, like our production compiled and minified front end assets.

When deploying code I always use a runner I control rather than a public runner just so I know my private SSH key has not entered any machines I'm not in control of. After registering my runner, [see here for setting one up](/blog/maximising-gitlab-ci-s-free-tier/), I then gave it a tag of "deploy". 

To tag a runner you need to go to your project -> settings -> ci / cd. Expand the runners section, find the available runners section then press the pencil icon next to the runner name (not the name itself).

We'll now need to give the docker image running the deployment, SSH access to our target server. This is handled within the `init_ssh_live` section of the code above where we can see the contents of the `$SSH_PRIVATE_KEY` being put into the the default ssh private key file.

```yaml
.init_ssh_live: &init_ssh_live |
  mkdir -p ~/.ssh
  echo -e "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
  chmod 600 ~/.ssh/id_rsa
  [[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config
```

Within Gitlab go to your project -> settings -> ci / cd and expand the Variable section. Here we can create a new key called `SSH_PRIVATE_KEY` and paste the private key of a key pair that has access to your server into the value field.

> Its highly recommended to NOT use your main user to deploy your project code with, for more information on how to do that check out [Chris Fidao’s](https://twitter.com/fideloper) excellent server management book : [Servers For Hackers.](https://serversforhackers.com/)

### Update supervisor
Many of my projects make use of Laravel Horizon to manage my redis queues. As per the [horizon docs](https://laravel.com/docs/master/horizon#deploying-horizon), im using `supervisor` to monitor the horizon processes.

Supervisor runs with root privileges. What we need here is for it to be reloaded by our webserver (which si going to be running some of the deployment code) which we obviously dont want to have root permissions.

```bash
sudo nano /etc/supervisor/supervisord.conf
```
Update the top lines to match the following:
```bash
[unix_http_server]
file=/var/tmp/supervisord.sock
chmod=0770
chown=nobody:www-data
```

We should be able to test that we dont need root privileges now on the machine locally by running `supevisorctl reload horizon` and seeing `Restarted supervisord`.

### Restart php-fpm
Php-fpm usually requires also requires root permissions to be able to be restarted. Lets change it so our `deployer` user can restart php-fpm without needing the elevated privilege.

```bash
sudo visudo
```

```bash
# add to end of file
deployer    ALL=NOPASSWD:/bin/systemctl reload php7.4-fpm
```

## AWS EC2
For people deploying on to AWS EC2 instances there are a couple of things we'll need to do differently. AWS gives you a .pem file used to access your server.

In the `config/deploy.php` file update the hosts section accordingly

```php
...
    'hosts' => [
        'deployer.talvbansal.com' => [
            'deploy_path' => '/media/websites/deployer.talvbansal.com',
            'user' => 'deployer',
            'identityFile' => '~/.ssh/aws-deployer-key.pem',
            'forwardAgent' => true,
        ],
    ],
```

Where `~/.ssh/aws-deployer-key.pem` matches the path to the key given to you by amazon.
You'll also need to to make sure the ssh private key we defined as `$SSH_PRIVATE_KEY` in gitlab matches the content of this file too.

### Deploying

Providing permissions to your target deployment folder are set correctly we should be in a position to do an initial deployment. From your local machine run the following:

```bash
php artisan deploy deployer.talvbansal.com -o git_tty=true
```

*Aren't we supposed to be deploying from gitlab?* 

Yes we are but just incase there are any errors I've found it to be faster to do a manual deploy the first time around rather than going through all of the project pipelines and then realising we have something wrong. 

Once your project is deployed (it now will live under a new subfolder of `current` - `/media/websites/deployer.talvbansal.com/current`) you will need to ssh onto your server and update your `.env` file.

If your app is functional then we should be ready to start deploying via gitlab. 

Commit all of your changes and push gitlab and if everything has gone to plan (and you've just pushed or merged into your master branch!) then soon enough you should see your deploy pipeline being run and your new code live on your server!