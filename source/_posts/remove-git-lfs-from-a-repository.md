---
title: Remove git-lfs from a repository
date: 2020-04-07 12:31:00
tags:
- git
- github
- git-lfs
---

Today I tried to set up my [jigsaw-photo-stream project](https://github.com/talvbansal/jigsaw-photo-stream) to use netlify large media with git-lfs.

What I failed to read was this key part of the netlify documentation:

> Files tracked with Large Media are uploaded directly to the Netlify Large Media storage service on push, completely bypassing the site build.

<div style="margin-bottom: -2rem"></div>
As part of the build process for the project I resize the images that are uploaded. 
The netlify documentation suggests that you could use netlify's image transformation to get around this but ideally I dont want the project to be tied into the service provider should I want to migrate to another static site host.

At this point I'd already set up `git-lfs` on the repo and my netlify builds were failing. 
I found [this link](https://github.com/git-lfs/git-lfs/issues/3026#issuecomment-451598434) to remove `git-lfs` from an existing repo

The steps being:

```bash
git lfs uninstall
```
remove lfs stuff from `.gitattributes`
```
git lfs ls-files | sed -r 's/^.{13}//' > files.txt
while read line; do git rm --cached "$line"; done < files.txt
while read line; do git add "$line"; done < files.txt
git add .gitattributes
git commit -m "unlfs"
git push origin
git lfs ls-files
rm -rf .git/lfs
```