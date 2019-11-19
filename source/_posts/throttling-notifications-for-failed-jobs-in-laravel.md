---
title: Throttling Notifications For Failed Jobs In Laravel
date: 2019-11-19 16:20:18
tags:
- Laravel
- Queues
- Jobs
- Failed Jobs
- Throttle
- Throttling
- MS Teams
- Microsoft Teams
- Notifications

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1574185225/posts/south-africa-zebra-back.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1574185225/posts/south-africa-zebra-back.jpg
thumbnailImagePosition: right
---
One of the projects I've been working recently has involved writing a system to communicate with a Clients pre-existing Legacy system. The system doesn't have the ability to "talk out" but can be queried using a SOAP service. 

The legacy system is:
- Slow to interact with
- Prone to crashing regularly

Here's how we dealt with those 2 issues:

Since we don't want to impact our end users experience the project has been set up to make use of Laravel queuing system with Redis and those queues are configured with Laravel Horizon.

When these jobs failed for whatever reason - including the clients system going down we wanted to be notified so that someone could look into what was happening.

The team at [spatie.be](https://spatie.be/) created a great [package to do that](https://github.com/spatie/laravel-failed-job-monitor) handle notifying us when a failed job occurred.

However there are multiple jobs being run concurrently that run as frequently as every minute to check for things on their server. Which mean't that once the legacy system went down we would receive multiple notifications until the system came back up, (In some cases this has been hours - not ideal). 

After a while being flooded with these sorts of notifications makes them become an annoyance rather than useful and you begin to start ignoring them.

So I set out to write something that gave me everything the Spatie package did but also allowed me to throttle how frequently notification of a given type would be sent to us.

[Laravel throttled failed jobs!](https://github.com/talvbansal/laravel-throttled-failed-jobs)

<!-- more -->

The package is effectively a mash up of the Spatie package, [this gist](https://gist.github.com/scottwakefield/81da8c00e63101a20fe6ea3735c21f02#file-routesthrottlednotifications-php) and then made to work with Laravel 6.x.

Setting the package up is easy:

```php
composer require talvbansal/laravel-throttled-failed-jobs
php artisan vendor:publish --provider="TalvBansal\ThrottledFailedJobMonitor\FailedThrottledJobsServiceProvider"
```

This will install everything needed and publish a config file `config/throttled-failed-jobs.php`- that's it! 

At this point restart your queue worker and your throttled failed jobs should start working (as long as you have failed jobs of course!).

In the config file you'll need to choose how you send notifications to yourself, by default the package handles mail, slack and [ms-teams](/blog/send-notifications-to-ms-teams-with-laravel) or you can use another of the [laravel notification channels](https://laravel-notification-channels.com).

If its another of the channels you'll need to extend the default notifiable class `\TalvBansal\ThrottledFailedJobMonitor\Notifiable::class` and add a `routeNotificationFor{channel}` method.

Next you might want to change the throttle delay. By default its set to 10 minutes. The idea being here that we should only see a given type of failed job once every x minutes.

So in the example of the sync job we have running every minute if the client system went down we would see at least 60 notifications per hour that's not counting any retried job that we might have set up.

Add in multiple jobs failing as a result of a third party being unavailable and you'll quickly want to throw your phone / laptop / self out of a window. 

The throttling part of this package should reduce those to a maximum of 1 notification for the sync job including job retries every 10 minutes until the failures cease. 

So in 10 minutes instead of seeing 10 failures + 10 retry notifications we'd just see 1.
If there were 3 different jobs all running on a 1 minute sync instead of 3 job types x 10 minutes x 2 one extra retry = 60 notifications we'd just see 3.

This has made our notifications useful again and reduced our stress levels greatly! Hopefully it'll help you too!
