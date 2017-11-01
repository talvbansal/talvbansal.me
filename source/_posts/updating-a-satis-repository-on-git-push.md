---
title: Updating a Satis Repository on Git Push
date: 2016-09-27 16:31:04
tags:
- Gitlab
- PHP
- Composer

autoThumbnailImage: yes
coverImage: ./santorini-domes.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: santorini-domes.jpg
thumbnailImagePosition: right
---

**Update May 2017, looks like with the Gitab update to V9, which uses a new Gitlab API v4, Packages will not let you enable new repositories**

I've made amendments to the necessary files and packages on this fork:

https://github.com/talvbansal/packages on the "gitlab" branch.

As long as you change step 2 in the guide below everything should work as expected:

```
git clone https://github.com/talvbansal/packages
cd packages
git checkout gitlab
composer install -o --no-dev
```

<hr>

After moving our source control server from **[gitolite](http://gitolite.com/gitolite/index.html)** to a privately hosted **[gitlab](https://about.gitlab.com)** instance the next thing I wanted to do was get our internal **[Satis](https://github.com/composer/satis)** server to automatically update whenever code was pushed to a repository.
<!-- more -->

Before starting lets talk about what Satis is and why we'd want to use it:

From the [Satis github page](https://github.com/composer/satis):

>## Satis - Package Repository Generator

>Simple static Composer repository generator.

>It uses any composer.json file as input and dumps all the required (according to their version constraints) packages into a Composer Repository file.

In essence Satis will generate us a [packagist](https://packagist.org/) like repository for our own `composer` projects - theres no frontend however just the bit that registers all of the packages submitted to it. What that means for us in practise is that we can do the following to access all of our private repositories that could be held within gitolite or gitlab by just adding the following to our `composer.json` file:

```
  "repositories": [
    {
      "type": "composer",
      "url":  "https://path.to.our.satis.server.com"
    }
  ],
	```
	
Without this you'd have to require each repository you want to use with their full git repository path, but with this we can simply `composer require my-company/my-package`.

The only thing with Satis is that as per its description its a "Simple **static** Composer repository generator" - Static being the important part. What that means is that when code is added to a repository, Satis doesn't know about it so we have to manually tell Satis to re-scan all of our projects to get the latest information about them.

What I'd set up prior to this was a cron job that ran every 5 minutes to run the Satis build command. I knew at the time that it wasn't ideal but it worked and we didn't have the time or resource to explore things any further at the time. 5 minutes isn't very long but it can feel like an eternity when you're waiting for a commit to be registered and as our development team has grown its become increasingly frustrating even after reducing that time to every 2 minutes.

**What I really wanted was an update whenever code was pushed to a registered repository.**

### Enter [Packages](https://github.com/terramar-labs/packages).

Packages builds on top of Satis and allow you to set to up hooks with gitlab and github which will let you update its internal Satis instance every time code is pushed to a repository - exactly what we're after!

Lets get started on setting it up. [I ran this the same Ubuntu 16.04 server that I set our private Gitlab up on](https://talvbansal.me/blog/setting-up-a-private-gitlab-server-on-ubuntu-1604).

1. Setup and configure a `nginx` web server to serve the Satis repository information and the `php-sqlite` database driver:
```
sudo apt install nginx php php-fpm php-sqlite3
```

2. Clone out the repository and install the dependencies:
```
git clone https://github.com/terramar-labs/packages
cd packages
composer install -o --no-dev
```

3. Copy the example config file and edit any values in it if necessary:
```
cp config.yml.dist config.yml
nano config.yml
```

4. Create an sqlite database for the Satis package information to be stored and then change its permissions:
```
touch database.sqlite
chmod 775 database.sqlite
```

5. Run the migrations to build the database:
```
bin/console orm:schema-tool:create
```

6. Set up an `nginx` virtual host to serve the Satis information:
```
sudo nano /etc/nginx/sites-enabled/default
```
You'll need to amend the following information in the file below `root`,	`ssl_certificate`,	`ssl_certificate_key`,	`server_name `:
	
	```
	server {

		# SSL configuration
		listen 443 ssl default_server;
		listen [::]:443 ssl default_server;

		ssl_certificate /etc/ssl/my-ssl-cert.pem;
		ssl_certificate_key /etc/ssl/my-ssl-key.key;

		root /media/data/packages/web;

		index index.html index.htm index.nginx-debian.html index.php;

		server_name source.my-server-name.com www.source.my-server-name.com;

		location / {
			try_files $uri $uri/ /index.php$is_args$args;
		}

		location ~ \.php$ {
			include snippets/fastcgi-php.conf;
			fastcgi_pass unix:/run/php/php7.0-fpm.sock;
		}
	}
	```

7. Restart `nginx` 
```
sudo systemctl restart nginx
```

8. Point your web browser at the `https://{your-server-name}`, log into `Packages` and click on remotes.

9. Add a new remote which points at the `gitlab` instance. You can create an access token within `gitlab` by clicking on your avatar and pressing `Profile Settings` -> `Access Tokens`: 
![2016-09-27 12-55-59.png](./2016-09-27-12-55-59.png)
10. After the remote is created press the `Sync` button to fetch repository information from your `gitlab` instance: 
![2016-09-27 13-07-36.png](./2016-09-27-13-07-36.png)
11. Head over to the dashboard section and you should see a count of the number of repositories that `Packages` has found from `gitlab`.

12. Within the packages section you'll find more information about each repository. Click on each one and enable the `Composer Satis` section: ![2016-09-27 13-11-21.png](/./2016-09-27-13-11-21.png)

13. Back on the packages list enable all of your repositories - theres no fast way to do this so click each one: ![2016-09-27 13-12-57.png](./2016-09-27-13-12-57.png)

14. Back on the terminal within the `Packages` folder, set up a worker to process background tasks:
```
bin/console resque:worker:start
```

15. Commit some code to a repository and push it!

16. Start the watcher on boot. There probably is a better way of doing this so if someone know how then please let me know!
	- Create a new script in your "Packages" folder called `worker.sh` and add the following content
	```
	#!/bin/bash
cd {/path/to/packages}
bin/console resque:worker:start
	```
	- Add the following to `sudo nano /etc/rc.local` before the `exit 0` line:
```
su {your-user-name} - -c /path/to/packages/worker.sh
```
We have to run the script as a user since the api token that `Packages` uses is linked to a gitlab user and in turn uses that users SSH key,

### Wrapping up

At this point now you should be able to gitlabs web hook tests to check that the worker is running. Watching the log will give you an indication of what is going on:
```
tailf /path/to/packages/logs/resque.log
```
If everything is well you should be automatically generating new satis listings whenever you push code to your repositories!