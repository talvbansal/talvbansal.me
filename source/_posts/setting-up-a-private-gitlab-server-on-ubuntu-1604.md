---
title: Setting up a private GitLab server on Ubuntu 16.04
date: 2016-09-16 19:09:13
tags: 
- Gitlab
- Laravel
- PHP

autoThumbnailImage: yes
coverImage: ./st-basils-moscow.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: ./st-basils-moscow.jpg
thumbnailImagePosition: right
---

## Introduction
Where I work, we've been using an Ubuntu Server running [gitolite](http://gitolite.com/gitolite/index.html) to act as our "Source control server" and manage permissions and access. Its worked well, and after some initial configuration (which I recall being pretty tricky) looking after the gitolite instance has been pretty easy and could easily continue to serve our needs as a small business well.

Adding new users required you to get an ssh key from the new user, add it to an "administration repository" and then add their name to a config file within that repository. The same config file could then be used to determine which access rights they had to repositories as well as defining new repositories.

We also use a [satis](https://getcomposer.org/doc/articles/handling-private-packages-with-satis.md) install to act as a private composer for our internal private packages.

One of the things gitolite doesn't provide you with is any sort of front end at all, everything has to be done through the terminal. It also requires you to have all of the following tools separately:

- Issue tracker
- File browser
- Activity stream
- Pull request management

Which are all features that github has built in natively. 

<!-- more -->

Since I used github for my own projects it would have been ideal for us to move to the paid plan over there to keep our repositories private. However our business wanted to keep everything on our own servers - hence the original gitolite set up. 

[Gitlab ](http://www.gitlab.com) is very similar to github in what it offers (free for open source projects, paid for private) but with one big difference, it offers a community version that you can self host. 

Mostly for my own reference I'm going to detail the steps I've gone through to get our gitlab instance set up. The gitlab install process is straight forward, just follow the [CE documentation](https://about.gitlab.com/downloads/#ubuntu1604).

Before i started anything I checked that all of the projects I wanted to migrate had been checked out on my local machine along with all of the branches of those projects that I wanted to keep (another benefit over gitolite should bring is the ease of identifying old branches within a repository that can be removed!).

 Beyond the base install I'll be covering the following:

- Changing the storage directory for git repositories
- Setting up postfix to send emails from out mail server using SMTP
- Creating groups within gitlab
- Importing old repositories into the new gitlab instance

I'm fortunate to be starting the process off with a freshly installed Ubuntu 16.04 LTS server, thats been fully upgraded.

## Installing and configuring gitlab

1. Install dependencies
	
	```
sudo apt-get install curl openssh-server ca-certificates postfix
	```

2. Install gitlab

	```
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
sudo apt-get install gitlab-ce
	```

3. Configure gitlab

	```
sudo gitlab-ctl reconfigure
	```

4. Configure gitlab to send E-mails via SMTP

	Edit the gitlab.rb file
	```
sudo nano /etc/gitlab/gitlab.rb
	```
Find the "GitLab" email server settings" section update and it and uncomment it to suite your email server settings
	```
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "mail.yourserver.com"
gitlab_rails['smtp_port'] = 587
gitlab_rails['smtp_domain'] = "mail.yourserver.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false
gitlab_rails['smtp_openssl_verify_mode'] = 'none'
	```

5. Set the git data directory

	Edit the gitlab.rb file
	```
sudo nano /etc/gitlab/gitlab.rb
	```
	 Add the following to the bottom of the file
	 ```
 git_data_dirs({"default" => "/my/custom/git-data"})
	 ```
		
6. Reconfigure gitlab to make changes take effect

	```
sudo gitlab-ctl reconfigure
	```
	
7. Log into gitlab and start setting up your users and repositories!


The next thing I did was create some of the users for the system set up SSH keys (really easy to do within the admin area), create some groups and then begin importing our existing projects into the gitlab.
	
## Gitlab groups

By default if I as a user were to create a repository within gitlab it would exist under my username's namespace, forexample : `talv/my-project`. Normally this would be the expected behaviour and totally fine for my own work, but when working on company projects it wont do, we'd want something like `company-name/project-name` which could then be forked by other users.

Originally i created a user called with the name of our company and figured that admin users could just *** "impersonate"*** (an awesome admin feature that lets you pretend to be another user and see / interact with the system as them) the company user to create new project repositories. This 100% was not the right thing to do, since deleting that user would delete all of the companys codebase along with it! 

I then came across the **Groups** feature within the admin area. Creating a group makes a namespace for projects to become a part of, so if you had a bunch of projects that are only ever used internally by other projects (as our company does) you could create an **"Internal"** group. 

After creating a group every project you create going forward can belong to either your **User namespace** (`talv/my-project`) *or* a **Group namespace** (`internal/important-api`)

The other thing with groups are that you can not delete a user if it is the **Owner** of a group. You have to transfer ownership of the group to another user then delete the user account. Much safer than what I originally had!
	
## Importing existing repositories
Gitlab provides a number of hooks to import existing repositories from some of the most popular services (github, bitbucket etc) into it using the **New Project** section, it even allows you to add a repo from a url!

This should be fine for most use case scenarios however gitolite uses ssh keys to authenticate use access and whilst you can add username / password authentication to your url strings, I couldn't figure out how to get gitlab to authenticate with an ssh key. 

*To be honest i didn't really look for very long as i knew of a way that was a bit more labour intensive but didn't take too long.*

I knew from past experience you can edit a projects `.git/config` file to point at a new location and push all of the projects history there too.

So to begin with, within gitlab I created blank projects for all of the projects i wanted to import. 

Then back on my development machine (with all of our work projects checked out on.) I went into each project folder and then opened the `.git/config` file I edited the `origin` remote's `url` key to point at the new server and repository name.

```
[remote "origin"]
	url = git@server-name:/group-name/project-name
	fetch = +refs/heads/*:refs/remotes/origin/*
```

Then after saving the changes above, i then had to checkout each branch that I wanted to save to our new server and run `git push` to push that branch to gitlab.

Like i said this is a labour intensive process so if anyone knows of a better way of doing this let me know and I'll update the post!

## End
You should now have a configured gitlab instance up and running with groups created with repositories imported and ready for you to continue using.

What else do you do with your gitlab installs let me know in the comments!