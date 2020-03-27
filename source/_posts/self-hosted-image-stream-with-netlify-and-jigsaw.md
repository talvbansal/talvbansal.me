---
title: Self Hosted Image Stream With Netlify and Jigsaw
date: 2020-03-27 12:58:09
tags:
- Jigsaw
- Netlify
- Instagram
- PHP-gd
- PHP
- Static Site
- Photography

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1585315498/posts/south-africa-sunset.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1585315498/posts/south-africa-sunset.jpg
thumbnailImagePosition: right
---

Over the last couple of days of social distancing I spent some time working on a photostream site for some of my travel photos.
 
Whilst I usually post them on mine and my wife's Instagram page [iwantthewindowseat](https://instagram.com/iwantthewindowseat) I've not found myself having the motivation to select an image, think of a caption, find hashtags and post at the best time for visibility as much as I used to.

I also wanted somewhere where copyright ownership wasn't an issue. Much like this site is an archive of my ramblings and things I've worked on, I thought it would be cool to hav something similar for my photos.

This project was also a great opportunity to look at some of the newer browser features like lazy loading and leverage them - as of writing Firefox 74 is out and native lazy loading is due in Firefox 75. Native lazy loading is in chrome, the project uses a polyfill to bring lazy loading to older browsers.

![Photo Stream](/blog/self-hosted-image-stream-with-netlify-and-jigsaw/photo-stream-demo.gif)

The project itself can be seen at [iwantthewindowseat.netlify.com](https://iwantthewindowseat.netlify.com/).

The code repository can be forked and cloned over [here](https://github.com/talvbansal/jigsaw-photo-stream). 
