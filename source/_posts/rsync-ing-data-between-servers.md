---
title: RSync-ing Data Between Servers
date: 2017-08-20 15:10:44
tags:
- Rsync
- Linux

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1555352680/posts/new-york-skyline.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1555352680/posts/new-york-skyline.jpg
thumbnailImagePosition: right

---
After moving to NYC this week, I've found the need to move sets of data from my local machine to a server I have back at home in the UK.

Whilst I use [Resilio sync](https://www.resilio.com/) to keep things that i use regularly in sync between a number of devices this was more of a one off type deal so I decided to use the trusty command line tool rsync.
<!-- more -->

The apartment block uses time warner cable as its ISP which I've experienced a number of frequent connection drops with. So transferring the files had to be something that was resumable for those scenarios.

The command i ended up with was this:

```
rsync -avz --protect-args --progress --partial --append-verify local-folder user@remote-server:/path/to/data
```

Lets break all of the flags down:

```
-avvz
```

Archive, Verbose, Compress. These 3 arguments will ensure that data is compressed and recursively sent to the destination server so that the file and folder structure on the receiving server will be as close to that of the local server as possible. The double v gives an increased level of verbosity since rsync runs silently by default.

```
--protect-args
```
This is needed for paths that have spaces within them. If this isn't used folder paths get broken at the first space that rsync encounters even if the paths are escaped using a backslash.

```
-- partial --append-verify
```
Keep partial transfers and continue to append to them when resuming transfers. Without these flags if there is a scenario where a file transfer fails or becomes interrupted re-running rsync will re-start all incomplete files from the start, so these are super useful when transferring larger files that take time to send.

In summary rsync can be used to send recursive folder structures from one server to another in a one off manner. With some minor configuration we can make it traverse multi directory structures and make resumable transfers!

*-- EDIT --*

I've since had to amend the command above to send files to a server which has SSH running on a port other than 22. To do that i had to use the `-e` flag, if the port was 222 the new command would look like:

```
rsync -avz --protect-args --progress --partial -e 'ssh -p 222' --append-verify local-folder user@remote-server:/path/to/data
```


