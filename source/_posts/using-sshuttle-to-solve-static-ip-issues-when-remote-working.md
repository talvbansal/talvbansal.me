---
title: Using sshuttle to Solve Static IP Issues When Remote Working
date: 2019-11-21 15:51:28
tags:
- SSHuttle
- SSH
- VPN
- Devops
- Remote working
- Ubuntu
- OSX
- NordVPN

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1574239597/posts/tribeca-manhattan-nyc.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1574239597/posts/tribeca-manhattan-nyc.jpg
thumbnailImagePosition: right
---
### Introduction and Prerequisites 

I've been remote working full time for over 3 years now. 

In that time I've had to work with a number of clients who restrict access to their internal systems via IP Address whitelisting.

As a developer who often works from different locations and countries this can quickly become problematic. 

A paid for solution is to purchase a Dedicated Static IP VPN from a reputable provider like [NordVPN.](https://nordvpn.com/features/dedicated-ip/)  and ask the clients IT team to whitelist your new Dedicated Static IP.

This is something I've been doing for a while and whilst the initial set up is a little fiddly, (You have to set up 2 accounts and get their team to link them together) its worked flawlessly for me.

However if you have the following:
 - SSH access to a device in a location with a Static IP (Perhaps you get one from your ISP or your office has one)
 - You use Linux or OSX (Sorry windows guys I don't think even with WSL this will work). 
 
 Then you can use the super handy command line tool [SSHUTTLE](https://github.com/sshuttle/sshuttle) to route all (or some) of your traffic through that device.

Lets take a look at getting it working...

<!-- more -->

### Configuring your devices
> If you're already able to SSH into your target device you might want to skip ahead.

First we'll need to generate an SSH Key pair.

```bash
ssh-keygen -t rsa -b 4096 -C {your_email}@{address}.com
```

This will generate a key pair that we will use to act as your user and machines unique identity.

Next you'll need to make sure that you can ssh to the target device locally.
```bash
ssh {user}@{ip_address}
```

If you can log in with your password then great. Lets log out and copy our SSH Key over so that you can login without using a password.

```bash
ssh-copy-id {user}@{ip_address}
```

You should now be able to log into the same machine without putting your password in. Its generally good practise to disable password access to a device, if the device is going to be used by multiple people I'd recommend getting them to do the above steps before turning off password logins. 

On the target device's terminal edit the ssh config file.

```bash
sudo nano /etc/ssh/sshd_config
```

Scroll down until you find `PasswordAuthentication` and set it to `no`. If it doesn't exist then create this option instead. Save and exit this file then finally restart the ssh daemon.

```bash
// Newer ubuntu servers
sudo systemctl reload ssh

// older Ubuntu servers
sudo service ssh reload
```

Now if you try and log in to the device without using your SSH key you'll be denied.

Next you'll have to open your device up for public access. For this you'll need to refer to your router or ask someone from your IT team to open a public port and route it through to port 22 on your target device.

Once that's done you'll need to confirm that you can still ssh on to your target machine from a none local environment:

```bash
ssh {user}@{wan_ip_address or hostname}
```

If that's working as expected we can move on to routing traffic through this device.

### Setting up SSHUTTLE

On your development device we'll need to install shuttle, luckily its super easy!
```bash
// Ubuntu 
sudo apt install sshuttle

// OSX
brew install sshuttle
```

### Routing traffic through the remote device

To route all of your TCP traffic through the remote device (not DNS requests), run the following command on your local machine. If you're using port 22 externally then you can ommit the `:{port}` portion:

```bash
sshuttle -r {user}@{wan_ip_address or hostname}:{port} 0.0.0.0/0 -vv
```

Once its up and running you'll see continuous output in the terminal until you either close the terminal or terminate shuttle.

Now you'll be able to confirm your IP address has changed by running this in your terminal:

```bash
curl ipinfo.io
```

And seeing that your IP address is that of the device with the static IP.

You might find that you only want to push certain traffic over this tunnel and if that the case you can change the `0.0.0.0/0` part to match a range eg: `x.x.x.0/24`.

### Bonus

Since SSHUTTLE is acting as a poor mans VPN you've now got access to all of the device on your local network too!
