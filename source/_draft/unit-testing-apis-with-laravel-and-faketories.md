---
title: Unit testing APIs with Laravel and faketories
date: 2017-11-02 08:53:28
tags:
- php
- laravel
- api
- testing
- package
- gitlab

autoThumbnailImage: yes
coverImage: ./hawaii-beach-sunrise.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: hawaii-beach-sunrise.jpg
thumbnailImagePosition: right

---

Recently at work our team has been working on a project that has involved checking data with a number of third party api's.

We wrote tests for the api's but without bundling our credentials with our application the tests would of course fail during the test phase of our CI process.

*[See more on setting up gitlab's ci/cd pipelines here](/tags/Gitlab/)*

In order to get the tests passing in an isolated environment which did not need to talk out to the internet we'd need to "fake" our api requests.
<!-- more -->
Adam Wathan's [Test driven Laravel](https://adamwathan.me/test-driven-laravel/) series goes through the idea of "fake" gateways or objects that can be swapped into your Laravel app's container.

My interpretation of the idea's from the series can be summarised as follows:

Part 1:
- Consider the api's integration purposes within your application.
- Review the api's documentation. 
- Design an interface that your implementing class can use.

Part 2:
- Use TDD to write tests for and create a class for the real api as well writing a "fake" class that implements the same interface and behaves in a similar fashion to the real implementation.
- Use dependency injection within your application code to pass in the original interface that resolves the real implementation class.
- Within your fake class tests swap out the real implementation for the fake class.

Part 3:
- Find a way to tightly couple the tests for the fake and real implementations.
- Exclude the real api tests from the CI process.

