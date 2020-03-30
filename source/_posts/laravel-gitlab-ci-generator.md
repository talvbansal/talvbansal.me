---
title: Laravel Gitlab CI Generator
date: 2020-03-30 13:19:52
tags:
- Gitlab
- Laravel
- PHP
- Devops
- CI/CD
- Build
- Deploy

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1585573513/posts/santorini-oia.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1585573513/posts/santorini-oia.jpg
thumbnailImagePosition: right
---

At our company setting up gitlab ci configuration is one of the jobs I end up doing by default.

This weekend I wrote a package to help speed that process up by generating a .gitlab-ci.yml file as well as installing some of the packages and configuration files to make the following possible:

- Setting an [edbizarro docker image](https://github.com/edbizarro/gitlab-ci-pipeline-php) based on your target php version
- Handle Javascript asset compilation using NPM or Yarn
- Lint Javascript and Vue files with [Eslint](https://eslint.org/)
- Lint php files using the [laravel shift rules](https://gist.github.com/laravel-shift/cab527923ed2a109dda047b97d53c200) for [PHP-Code-Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer)
- Perform static analysis with [Larastan](https://github.com/nunomaduro/larastan)
- Run phpunit tests

The package currently provides a single artisan command to do all of the above after answering a few simple questions.

Check the repo out here:

[https://github.com/talvbansal/laravel-gitlab-ci-config-generator](https://github.com/talvbansal/laravel-gitlab-ci-config-generator)


<!-- more -->


Currently the package doesn't build anything for automatically deploying code but in time I hope to add this in using Laravel Deployer. 

This should mean that eventually the sum of these 2 blog posts ([Config](/blog/in-depth-gitlab-ci-cd-with-laravel-apps/) and [Deploying](/blog/easy-push-to-deploy-for-laravel-with-gitlab-ci/)) could be replaced with a single package and artisan command. 

In order to make the yaml generation easier this package requires the `php-yaml` extension. Which is installable as follows:

```bash
// ubuntu
sudo apt install php-yaml

// others
pecl install yaml
```

Currently `php-yaml` is installed in the CI process but I think I should be able to drop that requirement since we're not going to be generating yaml files in our pipelines.

Hopefully this should help speed things up for people setting up new projects.