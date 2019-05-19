---
title: Replacing Moment.js With Dayjs
date: 2018-10-02 19:01:00
tags:
- Laravel
- Javascript
- Ecmascript
- Tooling
- Moment.js
- Day.js
- Dayjs

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1555352638/posts/mexico-city-bellas-artes.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1555352638/posts/mexico-city-bellas-artes.jpg
thumbnailImagePosition: right
---

Earlier this year I blogged about reducing the file size of moment.js by stripping out additional locales. Whilst stripping the locales made a good saving in file size, really all I was using it for was to format dates within my user interfaces. So I set out on finding a lightweight alternative to do just that...

<!-- more -->

In my quest to reduce my bundle size I came across a few libraries and the easiest one I found to swap in was one called [dayjs](https://github.com/iamkun/dayjs). It weighed in at 2kB and had the same (well very similar) API to that of moment.js.

Implementing it was easy - first add it to your `packages.json`:

```bash
yarn add -D dayjs
```

With the library installed I set out to search for all of the references to moment within my project:

```bash
grep -i -r "moment" resources/js/
```

And then replaced the relevant imports from `moment` to `dayjs` and the method calls from `moment` to `dayjs`.
I did have to update a couple of the [formatters](https://github.com/iamkun/dayjs/blob/master/docs/en/API-reference.md#format-formatstringwithtokens-string) as they weren't exactly the same as moment.js's but my test suite picked those errors up.

Following a production build I was able to compare the bundle sizes between `moment.js` with locales remove and `dayjs` in the project I was working on:


| Configuration                                     |  Bundle Size  |
| ------------------------------------------------- |:-------------:|
| [Media-manager with moment.js](https://github.com/talvbansal/media-manager/blob/ee76639b154a88f58f05a97c5533922a23448274/public/js/media-manager.js) |          468kB |                   
| [Media-manager with moment.js and locales stripped](https://github.com/talvbansal/media-manager/blob/37936444d356e41aab395c402c5837e957f241b0/public/js/media-manager.js) |         281kB |
| [Media-manager with dayjs](https://github.com/talvbansal/media-manager/blob/ea555c4219cad9581edd3e1ef5673f9ae5bd11a6/public/js/media-manager.js) |         237kB |

So from using moment.js without any optimisation there a pretty huge file size saving of nearly 50%! After optimising moment by stripping out excess locales we're still seeing a 44kB file size saving giving us an additional 15% saving on this particular bundle.

As well as giving a smaller bundle there was no longer any need to have [additional configuration within my `webpack.mix.js` file](https://github.com/talvbansal/media-manager/commit/ea555c4219cad9581edd3e1ef5673f9ae5bd11a6#diff-61877038a5575038809abf03f0009520)!

Don't forget to remove `momentjs` from your `packages.json` file and run `yarn` to rebuild the lockfile.

