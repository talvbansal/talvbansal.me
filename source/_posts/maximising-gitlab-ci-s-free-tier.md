---
title: Maximizing Gitlab's CI's Free Tier 
date: 2019-11-11 17:07:30
tags:
- Gitlab
- Devops
- CI/CD
- Ubuntu
- Raspberry Pi

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1573492411/posts/welvegonden-lion.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1573492411/posts/welvegonden-lion.jpg
thumbnailImagePosition: right
---

Last year I began working at a new startup - [NX Technology](https://nx-technology.com). 

As a start up with limited funding we're constantly looking for efficiencies and ways to get the most out of what we already have whether that be in hardware or software. 

Whilst previously I've set up and [self hosted a gitlab instance](/blog/setting-up-a-private-gitlab-server-on-ubuntu-1604/) at NX we needed to get things up and running asap so we decided to use the gitlabs free tier.

At the time of writing the bronze tier comes with unlimited private repositories and 2000 gitlab ci minutes using gitlab's shared runners.

One of the things I love about this is that once those minutes are up you can either [pay for more](https://customers.gitlab.com/plans) or use your own runners to process jobs for your projects.

We have an old -nothing special- server which we've set up a gitlab runner on that will handle jobs for us along with gitlab's shared runners and when our free minutes run out all jobs will run through that server.

Since most of the time when you're pushing code to gitlab your laptop will be on, you can use your development laptop to help out too!

<!-- more -->
### TLDR

Install and configure a gitlab runner on any spare hardware including your development device so that once your Gitlab CI/CD minutes run out your hardware can pick up jobs and keep your projects continuously deployed.

### Preparation and installation

I've almost always worked on a Ubuntu machine for my development - tried a mac for a year came back to a dual boot Windows and Ubuntu for a number of reasons that deserve their own blog post. 

As part of the build scripts I used to build my machine I installed a gitlab runner in the exact same way I did when I built the self hosted gitlab instance.

For our purposes we made all of our runners shared for the business' gitlab group so that any runner could be used for any project. You can of course set runners per project but given the scale of our business at  

You can navigate to your group's setup page from the [group list url](https://gitlab.com/dashboard/groups).

After selecting your group, in the sidebar select "Settings" -> "CI / CD", then expand the "Runners" section. 

You'll see a section called "Set up a group Runner manually", here are 2 pieces of information you'll need to set up your runner.
- https://gitlab.com/
- A runner registration token

You'll need these for the runner registration in the following steps as well as picking the docker executor when asked.

If you're on Ubuntu you'll need to run the following to get up and running:

```bash
# Install Docker...
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh

# Install Gitlab Runner...
sudo wget -O /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
sudo chmod +x /usr/local/bin/gitlab-runner
sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start
```

If you're on OSX the command differ slightly. Install docker for mac first.
```bash
sudo curl --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-darwin-amd64
sudo chmod +x /usr/local/bin/gitlab-runner
cd ~
gitlab-runner install
gitlab-runner start
```

At this point you should have a runner registered and running on your system.
The runner itself should be visible on the "Group" -> "Settings" -> "CI \ CD"  page and hopefully listed as online (green light next to it).

### Running multiple concurrent jobs

By default a runner registered this way will be set up to run a single job at a time. If you want to allow for multiple concurrent jobs you can edit the runners config file:

```bash
sudo nano /etc/gitlab-runner/config.toml
```
Change the `concurrent` value on the first line from 1 to whatever number you're happy with. 
I keep my laptop at 1 and honestly can't notice when the runner is actually being used. We have another small machine thats set to run 4 concurrent jobs.

You'll also be able to see the information we entered when creating the runner in this file so if you decide to change anything in future here's where that would happen.

After saving an exiting you'll need to restart the gitlab-runner service

```bash
sudo systemctl restart gitlab-runner
```

### Gotchas

Raspberry Pi. Don't.

We have a Pi Hole running on our network and I figured that if you could install a gitlab-runner on the pi that would be the ultimate low cost, low maintenance device.
So I went about installing and configuring the Pi to do just that. 

Everything looked good, it was registered and running however when it was actually handed a job this happened:

![gitlab arm failure](https://res.cloudinary.com/www-talvbansal-me/image/upload/v1573510511/posts/failed-arm-job.png)

It turns out that docker images are architecture specific, so this cryptic error:

```bash
standard_init_linux.go:211: exec user process caused "exec format error"
```

meant that the docker image that had been built for x86_64 and therefore could not be run on the Pi's architecture. 

The sad part for me was this, in the aim of being proactive I wrote this blog post originally to show of all my learnings on how to get a runner configured and optimised for the raspberry pi!