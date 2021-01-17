---
title: Using sshuttle with docker
date: 2021-01-17 19:53:46
tags:
- SSHuttle
- SSH
- VPN
- Devops
- Remote working
- Ubuntu
- Docker

autoThumbnailImage: yes
coverImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_1600/v1610917121/posts/istanbul-lamps.jpg
coverSize: partial
coverMeta: in
metaAlignment: center
thumbnailImage: https://res.cloudinary.com/www-talvbansal-me/image/upload/c_scale,w_280/v1610917121/posts/istanbul-lamps.jpg
thumbnailImagePosition: right
---
In 2019 I wrote about how I get around static IP restrictions when working in different places.
The world is very different now and whilst I'm not travelling anymore I still use sshuttle regularly to make my traffic look like its coming from a given location.

Recently I started switching some of my projects to run on docker locally (annoyingly this was before Laravel Sail was announced). However one of the challenges with sshuttle + docker in the way I'd originally got things set up is your networking gets messed up and consequently nothing really works for the project - not ideal.

Thats not the end of the story though, there's an easy fix here - exclude the docker subnets from sshuttle. 

<!--more-->

First I added all of the services for my project to a shared network within my `docker-compose.yml` file:
```dockerfile
services:
  app:
    image: my_project
    container_name: my_project_api
    networks:
      - laravel
```

Restart the containers: 
```
docker-compose up -d
```

Then list the networks in use: 
```bash
docker network ls
```

```bash
NETWORK ID          NAME                        DRIVER              SCOPE
1d6a445c077d        bridge                      bridge              local
927e9f1c0500        host                        host                local
6f0edf512205        laravel                     bridge              local
d5ef6d6fc71f        none                        null                local
3fdcdb1f7f2d        my_project_default          bridge              local
```

Here we can see the new `laravel` network. We need to find the subnet that its running on:

```bash
docker network inspect laravel
```

```bash
[
    {
        "Name": "laravel",
        "Id": "6f0edf5122051b3c92b65b35c5d858aebd699b81c676c951b00125cd6c09506d",
        "Created": "2021-01-17T13:44:39.92981407Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.19.0.0/16",
                    "Gateway": "172.19.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": true,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "3aaa2b230ed3422e7aaa960e9de89eba212b76af5952c27f4720575bee96f4c6": {
                "Name": "my_project_scheduler",
                "EndpointID": "8e6cd83abe4775f96b06bd749d251babbd4d8755f566f90eb079d5ddab4d4002",
                "MacAddress": "02:42:ac:13:00:02",
                "IPv4Address": "172.19.0.2/16",
                "IPv6Address": ""
            },
            "3aac9f134acfe9a163637e9dd52c54cb237b892a2399c8d50ee250b8b0f28edd": {
                "Name": "my_project_api",
                "EndpointID": "e8aa678e07509d1a98b350eaa0b6b3f8b8bfdc0a9ee58201624cc685d540418d",
                "MacAddress": "02:42:ac:13:00:04",
                "IPv4Address": "172.19.0.4/16",
                "IPv6Address": ""
            },
            "49793e9efa3ba61c9df6cb629bb1658fed1eb2ec47129a3fad104a61028a5e15": {
                "Name": "my_project_redis",
                "EndpointID": "685a07f281ced6034985c6a2aeeea841b9d8072bf25706cebff248d27e788492",
                "MacAddress": "02:42:ac:13:00:03",
                "IPv4Address": "172.19.0.3/16",
                "IPv6Address": ""
            },
            "809a4b320770a674d702461a4f9c38488a684d7cf988a04d285f0d43183e6104": {
                "Name": "my_project_queue",
                "EndpointID": "1c3a655b87fac2689b28404c63d95597fcb801d70e4eb907eea73c6ef8ad932f",
                "MacAddress": "02:42:ac:13:00:05",
                "IPv4Address": "172.19.0.5/16",
                "IPv6Address": ""
            },
            "83fcc85f64e7c0f7b44df471f9ab2d71ec1685355f6f3226bbe3a15c5deadf72": {
                "Name": "my_project_mysql",
                "EndpointID": "7ace3f6986011f0ec643944c9ba12d192a8cec661e93e977752737394765bab6",
                "MacAddress": "02:42:ac:13:00:07",
                "IPv4Address": "172.19.0.7/16",
                "IPv6Address": ""
            },
            "a8308cce9c98b6d071d50716c432a4c710e838ade5cb7e2df947e62147e641ac": {
                "Name": "my_project_npm_run_dc58894d335f",
                "EndpointID": "3aed18c542d25a808f0e66ba9cc0d2bc60118db61bfe6eff13bdc5ed11e4a9be",
                "MacAddress": "02:42:ac:13:00:06",
                "IPv4Address": "172.19.0.6/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "laravel",
            "com.docker.compose.project": "my_project",
            "com.docker.compose.version": "1.25.5"
        }
    }
]
```

What we're looking for here i the network subnet (line 14), in this instance "172.19.0.0/16". 
With that we can change our sshuttle command to exclude traffic on that subnet.

```bash
sshuttle -l 0.0.0.0 -r {user}@{host}:{port} -x 172.19.0.0/16 0.0.0.0/0 --dns -v
```

This should get things up and running again. But we can still do better. Everytime you start your containers up that subnet could change. We should be able to figure it out and script this procces right?

Lets start by installing jq to help us parse docker cli output. 

```bash
// ubuntu
sudo apt install jq

// osx 
brew install jq
```

Now we can use the cli to determine the network subnet:

```bash
$NETWORK = docker network inspect laravel | jq -r '.[0].IPAM.Config[0].Subnet'
echo $NETWORK

// 172.19.0.0/16
```
So now lets script the whole thing, create a new file called `docker-sshuttle`

```bash
cd ~/
nano docker-sshuttle
```

And put the following in:

```bash
#!/bin/bash

# set docker network name and target server
NETWORK=$1
SERVER=$2

SUBNET=`docker network inspect $NETWORK | jq -r '.[0].IPAM.Config[0].Subnet'`

sshuttle -l 0.0.0.0 -r $SERVER -x $SUBNET 0.0.0.0/0 --dns -v
```

Now make the script executable:

```bash
chmod +x docker-sshuttle
```

and finally lets run it:

```bash
./docker-sshuttle laravel user@host
```

After asking for your password you should be up and running!
This way we can customise the docker network and target server without having to remember all of the flags for the command.
