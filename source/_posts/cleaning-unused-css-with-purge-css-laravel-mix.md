---
title: Cleaning unused CSS with Purgecss Laravel Mix
date: 2019-05-06 19:48:07
tags:
- Laravel
- Laravel Mix
- Javascript
- CSS
- Tooling

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1557168888/posts/joburg-graffiti.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1557168888/posts/joburg-graffiti.jpg
---
I'm always looking for easy wins to optimise my development workflow and improve the end user experience, so figured after hearing about [Purgecss](https://www.purgecss.com/) I'd look into it and see how I could integrate it into my work flow. 

> "Purgecss is a tool to remove unused CSS"

With frameworks like [Bootstrap](https://getbootstrap.com/) and [Zurb](https://foundation.zurb.com/sites/docs/) providing so many CSS classes that often don't get used, this looked like it really would be an easy win situation!

<!--more-->

This week I deployed my first site over at [Netlify](https://www.netlify.com/) for the [new company I'm working at](https://nx-technology.com). 

The site I'd been working on is written in Laravel and  as it didn't have a backend it was a great candidate for extracting it out to a static site. Converting a laravel app to a static site is now totally doable using the new [Laravel Export](https://github.com/spatie/laravel-export) package from the excellent folks over at [Spatie](https://spatie.be/). I figured this would also be a great candidate for trying out Purgecss.

First lets look at the processes already in place and the filesize being output.

As this is a Laravel app it makes sense that the project would use Laravel Mix for its asset bundling.

With an out of the box Laravel 5.8 config our `webpack.mix.js` file looks like this:

```javascript
const mix = require('laravel-mix');

mix.js('resources/js/app.js', 'public/js')
.sass('resources/sass/app.scss', 'public/css');
```

After building the site this config built assets of the following sizes in dev and production mode.

| File           | Dev Size | Prod Size | 
| ---------------|:--------:|:---------:|
|  /css/app.CSS  | 183 KiB  |   147 KiB |
|  /js/app.js    | 1.39 MiB |   432 KiB |

We're not going to interested in the size of the JS file since we'd expect Purgecss to only impact the outputted CSS files.

Adding Purgecss to the site was even easier than I'd expected, the team at Spatie had built a plugin for laravel-mix.

```bash
yarn add -D laravel-mix-purgecss
```
Or if you use npm 
```bash
npm install laravel-mix-purgecss --save-dev
```

Update `webpack.mix.js` to use the new plugin:

```javascript
const mix = require('laravel-mix');
require('laravel-mix-purgecss');

mix.js('resources/js/app.js', 'public/js')
   .sass('resources/sass/app.scss', 'public/css')
   .purgeCss();
```

Re-running the build process gave us the following size for our CSS file:

| File           | Dev Size | Prod Size | 
| ---------------|:--------:|:---------:|
|  /css/app.CSS  | 183 KiB  |  16.5 KiB |

The Purgecss plugin is only run during production builds but the size of the production file is tiny in comparison. 147kb down to 16.5kb. 
That's an 88.7% file size saving just from running one command and adding 2 lines of code.

I did find one problem with the outputted CSS, the [cookie banner package](https://apertureless.github.io/vue-cookie-law/) we'd been using had a custom theme that wasn't being detected and thus leaving our cookie banner unstyled. 
So I had to make a minor change to the config and add a whitelist rule for the generated classes:

```javascript
mix.js('resources/js/app.js', 'public/js')
    .sass('resources/sass/app.scss', 'public/css')
   .purgeCss({
        whitelistPatterns: [/Cookie--nx-theme$/],
        whitelistPatternsChildren: [/Cookie--nx-theme$/]
    });
```

The final CSS file generated was 22kb and after the browser gzipping the file only 5.18kb of data is actually sent down the wire. 
Compared with the 24kb gzipped file that was being sent prior to using Purgecss.

Wait, the gzipped file being sent originally was bigger than the actual file after using Purgecss? 

This really is an easy win!
