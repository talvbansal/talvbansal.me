---
title: Optimising Moment.js with Laravel Mix
date: 2018-04-18 15:56:44
tags:
- Laravel
- Javascript
- Ecmascript
- Tooling
- PHP
- Moment.js

autoThumbnailImage: yes
coverImage: ./mexico-city-street-art.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: mexico-city-street-art.jpg
thumbnailImagePosition: right
---

The PHP library [Carbon](https://carbon.nesbot.com/docs/) is hands down my favourite way to work with dates within PHP. 
When using Javascript the closes thing ive found to it is a library called [Moment.js](https://momentjs.com).

By default Moment.js is bundled with a plethora of locales which might not all be relavent to you or your users. In this article I want to look at how much of a size reduction we can use by stripping out unwanted locales using [webpack](https://webpack.js.org) via [Laravel-Mix](https://github.com/JeffreyWay/laravel-mix/tree/master/docs#readme).

<!-- more -->

First lets get a benchmark filesize by seeing what the minified and unminfied/optimised versions of the default laravel code base look like:

```
composer create-project --prefer-dist laravel/laravel moment-test
cd moment-test
yarn
yarn dev
```

The console should show the following:

```
 DONE  Compiled successfully in 7577ms                                                                          11:41:37 AM

       Asset     Size  Chunks                    Chunk Names
  /js/app.js  1.38 MB       0  [emitted]  [big]  /js/app
/css/app.css   195 kB    0, 0  [emitted]         /js/app, /js/app
✨  Done in 11.55s.
```

So 1.38mb for our unminified code.

```
yarn prod

 DONE  Compiled successfully in 26310ms                                                                         11:44:13 AM

       Asset    Size  Chunks                    Chunk Names
  /js/app.js  332 kB       0  [emitted]  [big]  /js/app
/css/app.css  153 kB    0, 0  [emitted]         /js/app, /js/app
✨  Done in 30.29s.

```

and 332kb for minified/optimised.

Now lets add moment.js to our project and re-run the same commands above to see their minified and unminified sizes.

```
yarn add -D moment
echo "\nimport moment from 'moment';\n
window.moment = moment;" >> resources/assets/js/bootstrap.js
yarn dev
```

After the last command has finished you should see something like:
```
 DONE  Compiled successfully in 8771ms                                                                                                11:33:21 AM

       Asset     Size  Chunks                    Chunk Names
  /js/app.js  1.92 MB       0  [emitted]  [big]  /js/app
/css/app.css   195 kB    0, 0  [emitted]         /js/app, /js/app
✨  Done in 12.92s.
```

Our unminifed JS is a whopping 1.92mb.
Lets see what its like after minifiying and optimising that same code:

```
yarn prod

 DONE  Compiled successfully in 33731ms                                                                         11:36:59 AM

       Asset    Size  Chunks                    Chunk Names
  /js/app.js  562 kB       0  [emitted]  [big]  /js/app
/css/app.css  153 kB    0, 0  [emitted]         /js/app, /js/app
✨  Done in 38.20s.

```

Much better at 562kb but still thats pretty huge given that we actually haven't written code for our application yet!

By default Moment.js comes with (at the time of writing) 123 locales. A lot of the work I do is entirely written for British consumers and so there are 122 additional locales being bundled that the consumers of my work will never need. I looked into stripping the additional locales out and ended up with the following code in my projects `webpack.mix.js` file:

```
```javascript
const webpack = require('webpack');
let mix = require('laravel-mix');

/*
 |--------------------------------------------------------------------------
 | Custom Mix setup
 |--------------------------------------------------------------------------
 |
 */

mix.webpackConfig({

  plugins: [
    new webpack.ContextReplacementPlugin(
      /moment[\/\\]locale/,
      // A regular expression matching files that should be included
      /(en-gb)\.js/
    )
  ]
});

/*
 |--------------------------------------------------------------------------
 | Mix Asset Management
 |--------------------------------------------------------------------------
 |
 | Mix provides a clean, fluent API for defining some Webpack build steps
 | for your Laravel application. By default, we are compiling the Sass
 | file for the application as well as bundling up all the JS files.
 |
 */

mix.js('resources/assets/js/app.js', 'public/js')
   .sass('resources/assets/sass/app.scss', 'public/css');
```

Re-running the `yarn dev` and `yarn prod` commands resulted in some substancial file size savings which can be seen in the table below:

| Configuration                               | Dev    | Prod  |
| ------------------------------------------- |:------:|:-----:|
| Stock Laravel                               | 1.38MB | 332kB | 
| Laravel with Moment.js                      | 1.92MB | 562kB |
| Laravel with Moment.js and locales stripped | 1.53MB | 385kB |
 
You can obviouly replace the string en-gb with a pipe delimted list of locales from the `node_modules/moment/locale/` folder, for example:

```
// A regular expression matching files that should be included
/(en-gb|en-au|en-ie)\.js/
```

Hopefully this should help you feel better about the amount of data we're pushing down the wire when using a library as useful as Moment.js.

If you have any further tips for optimising Moment.js or any other library I'd love to hear about them!