---
title: Send notifications to Microsoft Teams with Laravel
date: 2019-11-12 19:47:20
tags:
- Laravel
- MS Teams
- Microsoft Teams
- Notifications

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1573588975/posts/south-africa-lilac-breasted-roller.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1573588975/posts/south-africa-lilac-breasted-roller.jpg
thumbnailImagePosition: right
---
Today at work we switched away from Telegram to using MS Teams for our company messenger. 

Since we we're already paying for it it made sense to plus it offers a lot more than just a messenger such as custom wiki functionality.

One of the things we were making use of was telegram bots for our applications to sent us updates on key events of interest. 

I've only recently started to make use of [Laravels notifications](https://laravel.com/docs/master/notifications) in the projects I've been working on and to send messages via telegram I'd been using [this package](https://github.com/laravel-notification-channels/telegram).

However whilst I'd found documentation that said that sending messages to MS Teams was possible there didn't seem to be a notification channel for laravel.

So I went about building a basic one today:

### [Laravel Notifications for Microsoft Teams](https://github.com/talvbansal/laravel-ms-teams-notification-channel)
<!-- more -->

There were other packages for MS Teams that already existed like:
- [Laravel Microsoft Teams Connector](https://github.com/sebbmeyer/laravel-teams-connector)
- [Monolog Microsoft Teams](https://github.com/cmdisp/monolog-microsoft-teams)

but I really wanted to leverage Laravel's notification mechanism so that i could switch out the Telegram driver with this new one and be up and running.

The [github page](https://github.com/talvbansal/laravel-ms-teams-notification-channel/) has a lot more detail but after installing the package with composer and doing some minimal configuration I was able to switch all of our notifications away from telegram in a matter of minutes.

```php
    return MsTeamsMessage::create()
        ->title('Here is a test notification sent from the '.ucfirst(app()->environment()).' environment')
        ->content("Here is some content for the notification.
            > That also supports markdown formatting in the body too!
            Below are optional buttons and images.
        ")
        ->button('And some clickable buttons', 'https://nx-technology.com')
        ->image('https://source.unsplash.com/random/800x800?animals,nature&q='.now());
```
The code above produced the following notification in one of our team rooms.

![MS Teams Notification Sent By Laravel](https://res.cloudinary.com/www-talvbansal-me/image/upload/v1573591481/posts/ms-teams-laravel-notification.png)

I know we'll be making use of this in a lot of our projects at work, hopefully this will help someone else out too!
