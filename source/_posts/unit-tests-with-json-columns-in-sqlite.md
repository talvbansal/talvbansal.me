---
title: Unit tests with JSON columns in SQLite
date: 2019-04-15 01:59:08
tags: 
- Gitlab
- Laravel
- PHP
- CI/CD
- SQLite

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1900/v1555344239/posts/jaipur-hawa-mahal.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,h_140/v1555344239/posts/jaipur-hawa-mahal.jpg
---
When doing any sort of development work I've always preferred working on an Ubuntu machine.
Doing so has helped me understand the servers that our production code has always been run on.

Towards the end of last year I started work on a new project that made heavy use of [Spatie's Event Projector package](https://github.com/spatie/laravel-event-projector).
Without getting into the package too much, at its core it stores the payloads of all recorded events in a `JSON` column within a `stored_events` table. 

I hadn't needed to use `JSON` columns before and since they were a part of MySQL 5.7 which was released in 2015 I had assumed there'd be no issues with unit testing them.

For the most part I was right, writing unit tests with Laravel's build in tools has made me come to love writing tests and made TDD part of my daily workflow.
However when I tried to run the same tests within Gitlabs CI/CD pipelines I saw the following:

```bash
PHPUnit 7.5.6 by Sebastian Bergmann and contributors.

Runtime:       PHP 7.2.15
Configuration: /builds/secret-project/phpunit.xml

 ==> Tests\Unit\ExampleTest ✓ 
"{"errors":["SQLSTATE[HY000]: General error: 1 no such function: json_extract (SQL: select * from \"stored_events\" where \"event_class\" in (App\\Events\\ClaimReported, App\\Events\\ClaimUpdatedByApp, App\\Events\\ClaimCreated, App\\Events\\ClaimUpdated, App\\Events\\ClaimUploadCompleted) and \"id\" < 1 and json_extract(\"event_properties\", '$.\"claimUuid\"') = 0dab4335-c5e8-3564-9a0a-d0537cd697f4 order by \"id\" desc limit 1)"]}"
```

<!--more-->

Looks like the `json_extract` function was missing from SQLite. 
I have always managed to get by using SQLite's in memory databases for my unit tests and have previously had instances where I'd need to come up with a work around for missing SQLite features.
However this was different since tests were passing locally using an SQLite in memory database, but just failing on the CI/CD pipelines.

Some searching confirmed that as of SQLite 3.9 there was support for a `JSON` column extension, which indeed implemented the [json_extract](https://www.sqlite.org/json1.html#jex) function. 

I figured there must be some discrepancy between the version of SQLite within the pipeline and what I was running locally.

Locally I'm running Ubuntu 18.10 with the following version of SQLite:
```bash
SQLite3 --version
3.24.0 2018-06-04 19:24:41 c7ee0833225bfd8c5ec2f9bf62b97c4e04d03bd9566366d5221ac8fb199aalt1
```

I then pulled in the docker image I was using for the CI/CD pipeline, dropped into an interactive shell on the container and found the SQLite version indeed was different: 
```bash
sudo docker pull edbizarro/gitlab-ci-pipeline-php:7.2
sudo docker run -it 0ff5c5077939
Interactive shell

php > print_r(SQLite3::version());
Array
(
    [versionString] => 3.20.1
    [versionNumber] => 3020001
)
php > 
```

Whilst the version of SQLite being using, 3.20 was greater than 3.9 which had the JSON extension available it seemed that it was not being loaded on this image.
I spent some time trying to find a way to load the extension or pull a ppa into the image to install a newer version of SQLite but didn't get anywhere, plus my `.gitlab.yml` file became a hot mess.
At the time I switched to using a real MySQL database to run my unit tests against which of course had all the functionality I needed.

Recently following the release of PHP 7.3 in December 2018, a newer version of the the docker image was released with PHP 7.3 support.
Upgrading to the newest version of the image brought in a newer version of SQLite too!

```bash
sudo docker pull edbizarro/gitlab-ci-pipeline-php:7.3
sudo docker run -it 55bc11cd6386
Interactive shell

php > print_r(SQLite3::version());
Array
(
    [versionString] => 3.24.0
    [versionNumber] => 3024000
)
php > 
```
The new image used SQLite 3.24 which is the same version that I was running on my local development environment. It also had the `JSON` extension loaded so my unit tests passed as expected.

I know not everyone will be in a position to upgrade to the latest and greatest versions of PHP but its been something I've been trying to make a priority with all of my new projects.