---
title: (Finally) Contributing to open source
date: 2016-08-16 19:24:17
tags:
- PHP
- Laravel

autoThumbnailImage: yes
coverImage: ./south-africa-penguins.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: ./south-africa-penguins.jpg
thumbnailImagePosition: right
---

I once read an article on the merits of contributing to open source projects, within it i remember the author saying that the even smallest things can make all the difference. They even went on to say that their first commit was fixing a one character spelling mistake.

Open source software has provided me the tools to make my living as a software engineer over the past 9ish years, over that period I've always wanted to contribute back to the community to but for some reason I never actually have. Maybe I wasn't looking in the right places, perhaps i was looking at the wrong projects when my skills weren't where they needed who knows. I just knew that eventually I wanted to.

<!-- more -->

### Canvas and my first (open source) commit

Recently I came across a great blogging app called [Canvas](http://canvas.toddaustin.io/) which was built on Laravel, I checked it out and was in love with the UI! The author [Todd Austin](https://twitter.com/austintoddj)  has done a fantastic job of creating a blogging app that is simple, fast, and has a consistent, responsive user interface.

After checking out the code I and using the app for some time I figured that there were a number of improvements to the system that I could make that were small, attainable and in my opinion would be useful to other users. So i forked the repository and started work on my first improvement: being able to use the `control + s` key combination to submit the blog post form. Its a key combination that I regularly use and it feels natural to do when doing any sort of word processing. Part way into coding I spotted a spelling mistake and it reminded me of the article I mentioned first! 

Reminding me of that article I'd read many moons ago, fixing it HAD to be my first commit and so (git) history was made!

[My first REAL open source commit](https://github.com/talvbansal/Canvas/commit/3ec645329d4c858f4d3d9d162fd7c28b09b3675f)

**What an awful commit. ** My fix that involved swapping two letters some how involved 2 files, adding 11 new lines and removing 2 existing ones.

As you can unfortunately see in my excitement it looks like I did a `git commit -am "typo correction"` followed quickly by `git push` and accidentally added in some of the other code I'd been working on. It wasn't the best or most exciting of commits and probably didn't add that much value to the project but it was mine.

Since that first commit I've added a handful of features that are actually useful to the Canvas project as well as fixing a number of bugs along the way. 

Contributing to it became something I really looked forward to daily, it's really satisfying having your ideas become part of something bigger, having people recognise what you're bringing to a project and getting feedback on improving the ideas you're bringing to the table too! Hopefully over time it'll help me be more thorough with my commit content too! - Although since then I've still had plenty of questionable commits!

I've even started work on my own splinter (??? I'm not sure what else to call it) of Canvas which I'm calling **[Easel](https://github.com/talvbansal/easel)** which of course is open source.
 
 ### My first open source project - Easel!
 
**Easel** takes the excellent UI work that Todd Austin and other Canvas contributors have done and builds on it with *one fundamental difference* - it is focused on being a Laravel Package to be added on to an existing or new project rather than a whole project in itself. I'm hoping to have to the first release finished before I head to Laracon EU and from then I'll be looking forward to other people (hopefully!) contributing towards the project!

To wrap this up, I've loved contributing to an open source project and plan to continue doing so. If you've found yourself like I did for many years, where you couldn't find a project that you think you could add value to, look for newly announced projects and just look for things as little as typo's! 