---
title: Running Laravels scheduler as a non-root user
date: 2020-02-18 15:31:42
tags:
- Laravel
- Cron
- Root
- Security

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1582034180/posts/vienna-hofbug-palace.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1582034180/posts/vienna-hofbug-palace.jpg
thumbnailImagePosition: right
---
I recently watched the following great talk on hacking laravel apps.

<iframe width="560" height="315" src="https://www.youtube.com/embed/kKGGVGiq2y8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Towards the end of the talk Antti shows how it is possible to potentially gain root access to a server if your scheduler is running as root too.

As soon as I saw it I know I had a couple of apps where this vulnerability could have been exploited and so went to patch them straight away. 

Whilst I knew what needed to be done I wasn't 100% on how exactly I'd add an entry into another user's crontab that wasn't my own or root.

Turns out it was quite simple, acting as root use the `-u` argument to specify the target user. 

````bash
sudo crontab -e -u www-data
```

In the above example the crontab for the user www-data would be opened. Since my php-fpm instance is run by www-data and therefore has access to all the application code already this made sense to me.

Hopefully I'll never make this mistake again. If you haven't already seen Antti's talk above I'd highly recommend doing so asap!

<!-- more -->

