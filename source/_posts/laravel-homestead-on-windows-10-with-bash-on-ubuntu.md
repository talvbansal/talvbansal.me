---
title: Laravel Homestead on Windows 10 with Bash On Ubuntu
date: 2016-08-11 19:17:46
tags:
- Windows
- Ubuntu
- Laravel
- PHP

autoThumbnailImage: yes
coverImage: ./south-africa-lion-male.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: south-africa-lion-male.jpg
thumbnailImagePosition: right
---

In preparation for [Laracon EU](http://laracon.eu/2016/) in a couple of weeks i figured I'd need to take a device along with me - paper and pen would probably have been fine but Laracon looks to be huge and i dont want to be unprepared, plus you always see rows of silver lids and glowing apple symbols in the photos from developer conferences! 

I do almost all of my work on a Dell Precision 7710 which dual boots Ubuntu for development purposes and Windows for my photo editing. I also have a Surface Pro 4 which i use for note taking and photo editing on the go - *I'll be writing about how I keep all of my photos in sync in a future post*!

Neither of the above are Macbooks but whilst the Dell Precision runs Ubuntu and anything else I can throw at it, it is HUGE and not ideal to take well ... anywhere! So I figured I'd try and get a development environment set up on my Surface Pro 4 instead. 

I'd heard of setups using Homestead / Vagrant / Virtualbox before, but since I was using Ubuntu, I've never had any need to explore it any further so this really is a guide for first timer noobs. The [official Laravel Homestead guide](https://laravel.com/docs/5.2/homestead) appears to be geared up mostly for Mac users, Windows users will need a bit more help which is where I'm hoping this post will help.

Anyway by the end of this guide I'll have a Surface Pro 4 running Laravel Homestead and fingers crossed you'll have a Windows device doing the same thing too! This has been a really long introduction, lets get to it!

<!-- more -->

A couple of weeks ago Microsoft released their [Anniversary update for Windows 10](https://www.microsoft.com/en-US/windows/features) and one of the features I was excited for but wasn't quite sure why was the inclusion of **"Bash On Ubuntu On Windows"**, a ridiculous name for an Ubuntu subsystem on Windows. Since it's release I'd only really used it to replace Putty and ssh onto other machines, so I was really hoping I'd be able to leverage some of its features for the development environment on the Surface. 

## 1. Enable Bash On Ubuntu On Windows

1. In Windows 10 with the Anniversary Update applied,  open the start menu and type `update` to bring up the settings menu and click on `Update & Security`, 
2. On the left hand side of the new screen towards the bottom there is a section called `For Developers`, click on that and in the right hand panel press the radio button for `Developer Mode`![01.enable-developer-mode.png](https://www.talvbansal.me/storage/blog/2016-08-WindowsDev/01.enable-developer-mode.png)
3. Open the start menu and type in `programs`  and open `Programs and Features`, then on the left hand side click on `Turn Windows features on or off` and scroll down until you see `Windows Subsystem for Linux (Beta)`![02.enable-bash-on-windows.png](https://www.talvbansal.me/storage/blog/2016-08-WindowsDev/02.enable-bash-on-windows.png)
4. Open the start menu and type `bash` and open `bash.exe`, follow the commands that come up to create your Linux user - Bash on Ubuntu On Windows should now be installed! 

From here forward I'll differenciate between the two terminals like this
```
> an arrow means a Windows cmd prompt
```

```
$ a dollar meaning a Bash prompt
```

## 2. Install Virtualbox and Vagrant
This is pretty straight forward, install [Virtualbox](https://www.virtualbox.org/wiki/Downloads) and the [Vagrant](http://www.vagrantup.com/downloads.html) the traditional way on Windows.

I installed them to their default locations, you'll need to restart your machine once they're installed.

Finally we need to pull down the `laravel\homestead` box to your machine. Since Bash on Windows doesn't have access to your Windows "Path" we can't use the Vagrant command within it so open a normal Windows cmd prompt and run:
```
> vagrant box add laravel/homestead
```

- This takes a while so its probably a good time to make a cup of tea.

## 3. Install and Configure Homestead
Now we need to use git to clone the Laravel Homestead github repository. Normally we'd have to install something like Git-Bash for Windows but with Bash for Windows and access to the Ubuntu respositories we no longer need to. 

- Open a Bash Terminal and work out where you are:

```
$ pwd
$ /home/talv
```

What I've learned so far about Bash On Windows is that Bash can see the Windows file system but Windows can NOT see the Linux file system. So we'll need to navigate to the Windows filesystem if we want Windows to have access to the Homestead files that we're about to clone down.

In Windows your home folder is:
```
C:\Users\{username}\
```

So for me thats ```C:\Users\talvb\```.  

Within the Bash terminal go to your Windows home folder
```
$ cd /mnt/c/Users/talvb
```

Clone down the Homestead repository and run the `init.sh` file from within the newly cloned down folder
```
$ git clone https://github.com/laravel/homestead.git Homestead
$ cd Homestead
$ sh init.sh
```

Running `sh.init` will unfortunately generate a hidden `.homestead` folder in your ***Linux*** home folder, we need it to be in our ***Windows*** home folder so lets copy that over.

```
$ cp -R ~/.homestead /mnt/c/Users/talvb/.homestead
```

Next we'll need to tell the Homestead box which folders to share with your Windows environment. 

So lets create a folder for our code to live. Still within our `/mnt/c/Users/talvb/` folder, create a new folder
```
$ mkdir Code
```
We now need to edit the `Homestead.yaml` file within the previously copied folder and change the folders "map" value to match our new `Code` folder

```
$ nano /mnt/c/Users/talvb/.homestead/Homestead.yaml
```

```
folders:
    - map: c:\Users\talvb\Code
      to: /home/vagrant/Code
```

Whilst you're in here notice that theres an `authorize:` and `keys` section which point to `~/.ssh/id_rsa.pub` and `~/.ssh/id_rsa` we'll need to generate them now. Also make sure that you're happy with the IP address given, if not amend it accordingly (by default its `192.168.10.10`)

Press `Control + x` followed by `y` to save and exit the `nano` editor.

### Host names

Lets add a hostname to our Windows `hosts` file

Open the start menu and type `notepad`, right click on it and press `Run as administrator`.

Open the `hosts` file located at

```
C:\Windows\System32\drivers\etc\hosts
```

You will need to change the bottom right hand corner of the dialog from `Text Documents (.txt)` to `All Files (*.*)`

Once the file is open add the following line to the bottom but adjust the IP address if necessary

```
192.168.10.10  homestead.app
```

### SSH Keys

Lets generate the ssh keys mentioned earlier:
```
$ ssh-keygen -t rsa -C "your-email@address.com"
```
This will create you an ssh key pair that'll be used to log into your Homestead box. Again the generated files will end up in the  ***Linux*** home folder, so lets copy them back over to our ***Windows*** home folder

```
$ cp -R ~/.ssh /mnt/c/Users/talvb/.ssh
```

### Lets Go!

Right now we're pretty much ready to fire up our VM and start doing some web  development on our Windows Machine.

Back over on a Windows command prompt, change directory into our Homestead folder

```
> cd c:\Users\talvb\Projects\Homestead\
```

And fire up vagrant

```
> vagrant up
```

You should see something like this:
![03.starting-vagrant.png](./03.starting-vagrant.png)
And if all is well you should now be able to ssh (from your bash prompt) into the Homestead machine with the following command

```
$ ssh vagrant@127.0.0.1 -p 2222
```

if you see this then everything has worked:

![04.ssh-into-homestead.png](./04.ssh-into-homestead.png)

Woohoo, change directory into the `Code` folder develop away!

Once you have code accessible in your `~/Code/public` folder you should be able to access it in you web browser by going to 

```
http://homestead.app
```

The only exception to this (at least in my experience) is if you're using Microsoft's Edge browser 

When you're done open a Windows command prompt and use `vagrant halt` to safely shut down the VM.

```
> vagrant halt
```

## 4. Final things!

### SSH Forwarding

If you want your host ssh certificates to be accessible in your VM you'll need to make the following adjustments. 

1. Make an ssh agent launch when opening bash
2. Configure ssh to forward access to the certificate files

First open your bash.rc file - `nano bash.rc` and add the following to the bottom of it:

```
#!/bin/bash

# Set up ssh-agent
SSH_ENV="$HOME/.ssh/environment"

function start_agent {
    echo "Initializing new SSH agent..."
    touch $SSH_ENV
    chmod 600 "${SSH_ENV}"
    /usr/bin/ssh-agent | sed 's/^echo/#echo/' >> "${SSH_ENV}"
    . "${SSH_ENV}" > /dev/null
    /usr/bin/ssh-add
}

# Source SSH settings, if applicable
if [ -f "${SSH_ENV}" ]; then
    . "${SSH_ENV}" > /dev/null
    kill -0 $SSH_AGENT_PID 2>/dev/null || {
        start_agent
    }
else
    start_agent
fi
```

*(credit to the above from[ this repository](https://github.com/abergs/ubuntuonwindows#start-an-bash-ssh-agent-on-launch))
*

Then edit (or create if it doesn't exist) your ssh config file

```
$ nano ~/.ssh/config
```

And add the following:
```
Host homestead
  HostName 127.0.0.1
  Port 2222
  ForwardAgent yes
  User vagrant
```

**The spaces above are really important. The first line has 0 spaces in front of it and and every thing else has 2 spaces in front of them**

Once you've saved the two files restart bash and you should be able to log into the server with:

```
$ ssh homestead
```

You should now be able to clone / pull / push from your private github / bitbucket / anything else that requires an ssh key to access it.

### Startup and Shutdown scripts!
It might seem weird having to open a Windows command prompt to start things up and shut things down but use the Linux Bash prompt to ssh onto the VM. So I decided to create 2 Windows .bat files to start up and shutdown my VM instance

Their contents are as follows:

```
cd c:\Users\talvb\Projects\Homestead\
vagrant up
```

and

```
cd c:\Users\talvb\Projects\Homestead\
vagrant halt
```

So when i want to start things up and shut things down I just use the scripts which close automatically and I only ever really have my Linux Bash prompt open.

### Accessing Sites With Edge
So this is totally weird for me, but I could access `http://homestead.app` from ***EVERY*** browser I tried *including* the Surface's own in built version of Internet Explorer 11 but for some bizarre reason Edge refuses to open local sites from my VM. It seems that I'm not alone in this I tried the instructions from the following link but had no luck, [Good luck!](http://www.hanselman.com/blog/FixedMicrosoftEdgeCantSeeOrOpenVirtualBoxhostedLocalWebSites.aspx). The only way I managed to get edge to load anything was to use the following IP address 
```
http://127.0.0.1:8000
```

### What about installing and running Virtualbox / Vagrant through Ubuntu Bash for Windows
Short answer : It won't work.

## 5. Conclusion

This was pretty long, but hopefully you have a working development environment on your Windows 10 device! 

If any of the guide is not clear or its been helpful please let me know in the comments!

I'm off to have another cup of tea!