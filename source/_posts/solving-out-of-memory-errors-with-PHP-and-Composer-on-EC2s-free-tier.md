---
title: Solving out of memory errors with PHP and Composer on EC2s free tier
date: 2020-02-05 20:45:36
tags:
- AWS
- EC2
- SES
- PHP
- Composer
autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1582033481/posts/istanbul-galata-tower.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1582033481/posts/istanbul-galata-tower.jpg
thumbnailImagePosition: right
---

For one of my recent projects I wanted to make use of the free allowance that AWS gives for SES.

One of the conditions of the SES allowance was that your calling app needs to be hosted on EC2.

I've not used EC2 before so I figured this would be a good way to dive into it.

Whilst I would never usually install software like composer on a production server, this was purely to test things out.
 
 So after signing up for AWS and creating a local ubuntu server on an EC2 t2micro instance then cloning down the project I ran composer install to come across the following message: 

```bash
composer install
Loading composer repositories with package information
Updating dependencies (including require-dev)

mmap() failed: [12] Cannot allocate memory

mmap() failed: [12] Cannot allocate memory
PHP Fatal error:  Out of memory (allocated 822091776) (tried to allocate 4096 bytes) in phar:///usr/local/bin/composer/src/Composer/DependencyResolver/Solver.php on line 223

Fatal error: Out of memory (allocated 822091776) (tried to allocate 4096 bytes) in phar:///usr/local/bin/composer/src/Composer/DependencyResolver/Solver.php on line 223

```

822091776bytes is over 800mb of memory being consumed by composer.

<!-- more -->

Whilst php-fpm has a 128mb max memory limit per script by default,
 php-cli has a value of -1 which means use unlimited memory.
 
 In this instance attempting to allocate over 800mb of memory on a server with 1gb was not going to end well.

The solve was to create a swap file on the disk.

A swap file or partition is space on disk allocated that can used when the physical memory allocation is exhausted. Once the allocation is exhausted, older items in memory get offloaded to the swap partition.


The actual solve for this i found over on [this github issue](https://github.com/composer/composer/issues/7348). 

The following code will create a 1gb swap file in `/var/swap.1` for our instance to use

```bash
sudo /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024
sudo /sbin/mkswap /var/swap.1
sudo /sbin/swapon /var/swap.1
```
