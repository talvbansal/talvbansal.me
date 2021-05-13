---
title: SASS with Tailwind 2.x and Laravel Mix
date: 2021-05-13 11:06:44
tags:
- Laravel
- Laravel Mix
- Tooling
- Tailwind
- Sass

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1620900836/posts/east-river-skyline-view.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1620900836/posts/east-river-skyline-view.jpg
thumbnailImagePosition: right
---

Tailwind 2.x recommends using PostCSS for its preprocessor. However, if you still want to use Sass the documentation isn't the clearest on how to set it up.

There is [a page](https://tailwindcss.com/docs/using-with-preprocessors#sass) but it doesn't seem to give you the step by step breakdown needed - here document whats needed to use Sass with Tailwind 2.x and Laravel-Mix.

First rename the `resources/css/app.css` file to use the `scss` extension:

```bash
mv resources/css/app.css resources/css/app.scss
```

Next remove the default `postCss` config within the `webpack.mix.js` config file:

```javascript
// snip...
mix.postCss('resources/css/app.css', 'public/css', [
    require('postcss-import'),
    require('tailwindcss'),
    require('autoprefixer')
])
```

Now add the `tailwind` module and use the `sass` plugin and configure `postCss` to use the `tailwind.config.js` file:
```javascript
const tailwindcss = require('tailwindcss')
// snip...
mix.sass('resources/css/app.scss', 'public/css')
    .options({ postCss: [ tailwindcss('./tailwind.config.js')]})
```

Thats it!

When you run `npm run dev` the needed `sass` and other js dependencies will be detected as missing and automatically installed.
<!-- more -->