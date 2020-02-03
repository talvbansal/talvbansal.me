---
title: Resizing images for static sites with php when deploying to Netlify
date: 2020-02-02 12:11:39
tags:
- Netlify
- PHP
- Instagram
- PHP-gd
- Jigsaw
- Static Site

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1580755252/posts/scotland-balvenie-castle.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1580755252/posts/scotland-balvenie-castle.jpg
thumbnailImagePosition: right
---

### Introduction

I recently build my first site with [Jigsaw](https://jigsaw.tighten.co) and deployed it on [Netlify](https://www.netlify.com).

As part of that project I had to get some data from the Instagram API and present a feed of the latest 5 images on the site.

Rather than dealing with CORS errors in javascript I wondered what I could get away with in PHP during the build phase on a statically generated site.

Would I be able to:
- Query some form of public json endpoint
- Find the urls for the latest 5 images
- Download them locally resize them for efficiency
- Display them using Jigsaw

<!--more-->

### PHP module support on Netlify

After realising that you could even deploy a static site written in PHP on netlify the first thing I looked into was which PHP modules were enabled at build time.

[This pull request](https://github.com/netlify/build-image/pull/366/files) for supporting PHP 7.4 on netlify shows the proposed modules for the docker build image and also show the existing modules in use on the PHP 7.2 image currently supported by netlify. 

```bash
...
        php5.6 \
        php5.6-xml \
        php5.6-mbstring \
        php5.6-gd \
        php5.6-sqlite3 \
        php5.6-curl \
        php5.6-zip \
        php7.2 \
        php7.2-xml \
        php7.2-mbstring \
        php7.2-gd \
        php7.2-sqlite3 \
        php7.2-curl \
        php7.2-zip \
        php7.4 \
        php7.4-xml \
        php7.4-mbstring \
        php7.4-gd \
        php7.4-sqlite3 \
        php7.4-curl \
        php7.4-zip \
...
```

The key module I was interested in here was _php-gd_. The GD library is one used for dynamically manipulating images and *php-gd* provides bindings for the PHP language.
Previously I've used Imagik for image manipulation since its also got an easy to use API within PHP but its not available here on the netlify build.

Knowing what I wanted to do was in theory possible I then had to figure out a way to get Jigsaw to read the image files I'd created.

I actually chose to just write out a JSON file to the disk and get a vue component to query that json file. I might look at switching this out for something more lightwight like [Alpine JS](https://github.com/alpinejs/alpine) in the future.

The biggest drawback of this solution was that the Instagram feed would not be real time and would only update on a site deploy. 

In the case of this site that really didn't matter too much. However I did find [an integration with Zapier that could trigger a netlify site build when new media had been posted on instagram!](https://zapier.com/apps/instagram/integrations/netlify)

### Implementation

Jump to the end of the article if you just want to see the code.

[The Jigsaw docs](https://jigsaw.tighten.co/docs/event-listeners/) mention a beforeBuild event hook that allow you to run PHP code before any of your static site has begun to be created.

Within this method you can run any php code and make use of the `$jigsaw` instance.

The key parts for image resizing using php-gd are as follows:

```php
    $image = imagecreatefromjpeg($thumbnailPath);

    $scaled = imagescale($image, 320);
    imagejpeg($scaled, 'source/'.$fileSystemPath);
```

I also made use of Collections to extract out the key json items and create an array of items that can json encoded and then written to the filesystem using the `$jigsaw` object.

```php
    $newJsonString = json_encode($response, JSON_PRETTY_PRINT);
    $jigsaw->writeSourceFile('assets/instagram.json', stripslashes($newJsonString));
```

Next time you build your code a file will be created at `source/assets/instagram.json` which can be consumed by your frontend by making a request to `/assets/instagram.json`.

### Full snippet

Here's the full snippet if you want to use it in your own project, be sure to change the `$name` variable to suit your public instagram feed:

<script src="https://gitlab.com/snippets/1936052.js"></script>

In my haste of trying to build this I didn't realise I actually didn't need to manually resize things with this feed i could have just scrolled down and seen that instagram has already resized the assets for me that I can pull out.

However I still thought I'd write this up to show you can still do things in PHP on a static site that might not be obviously possible.

