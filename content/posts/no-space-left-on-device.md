---
title: "error: no space left on this device "
date: 2020-05-18T00:00:03+00:00
# weight: 1
# aliases: ["/first"]
tags: ["terminal", "linux", "docker","storage","default","error"]
author: "mrturkmen"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
description: "How to solve issue of 'No space left on device' which is caused by Docker. "
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
cover:
    image: "../../images/docker_logo.png" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/mrtrkmn/mrtrkmn.github.io/edit/master/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

# Summary

In this post, one of the well known (-open to discuss :)-) error of "No space left on device" __which is caused due to Docker__ will be solved with different approaches. 

Note: "No space left on device" error can be caused due to any other reason than docker itself. Hence, it would be nice to make sure that the error is caused due to docker volumes. 

You can check whether it is caused due to docker volumes or not by following steps over [here](#ensure-about-the-problem)

Note that the explanations and examples may differ according to your environment, hence instead of taking these information as rules, applying them as a reference would be much better approach. 

The system information for particular scenario described on post is given below: 

## System information 

- __PRETTY_NAME:__  Debian GNU/Linux 9 (stretch) 
- __NAME:__ Debian GNU/Linux
- __VERSION_ID:__ 9
- __VERSION:__ 9 (stretch)
- __KERNEL_RELEASE:__  4.9.0-6-amd64

## Docker daemon version

__Engine:__

   - Version:          19.03.8
   - API version:      1.40 (minimum version 1.12)
   - Go version:       go1.12.17
   - Git commit:       afacb8b7f0
   - Built:            Wed Mar 11 01:24:36 2020
   - OS/Arch:          linux/amd64
   - Experimental:     false

Although, it is NOT fully required to have same or somehow closer version of OS and Docker daemon, it could be nice to know exactly which conditions the following fix can work. 

Most of the cases, the following information could be valid for all Debian based systems however it is not for sure. 


# The Problem 

Docker can be used for most of the cases although now Kubernetes or any other orchestration systems preffered to used. That kind of orchestration systems may not cause this problem, hence they mostly provide auto scaling and runs on cloud. However, when you have a server with all responsibility, you may face this exact problem.

The problem is installation of docker into any system with default settings may create head ache in the future. If you are planning to use docker, intensively, the main reason of that, docker is generally using 

* `/var/lib/docker` 

place for docker volumes. In most scenarios, servers do not use root path for storing information, instead the appropriate approach for storing information on server is creating data volumes which has huge capacity and ability to extend or shrink time to time. The problem starts to show off itself when docker containers have been used for long period of time. 

## Ensure about the problem

It is quite handy to check whether the error of "No space left on device"  caused by docker volumes. To do that following simple bash commands can be used. 

```bash
$ df -h /var/lib
```

This will display free and used disk space for `/var/lib` path. Afterwards, you can see how is the difference between free and used spaces among usage percentage. An example is given below, it is taken from the system that I mentioned at the beginning in [system information](#system-information) section. 


```bash
$ df -h /var/lib

Filesystem             Size  Used Avail Use% Mounted on
/dev/sda3              173G  162G  2.4G  99% /
/dev/sda3              173G  162G  2.4G  99% /
/dev/mapper/data-data  9.1T  402G  8.2T   5% /data
udev                   252G     0  252G   0% /dev
/dev/sda3              173G  162G  2.4G  99% /
/dev/sda3              173G  162G  2.4G  99% /
/dev/sda3              173G  162G  2.4G  99% /
/dev/sda3              173G  162G  2.4G  99% /
/dev/sda3              173G  162G  2.4G  99% /
/dev/sda3              173G  162G  2.4G  99% /
/dev/sda3              173G  162G  2.4G  99% /
/dev/sda3              173G  162G  2.4G  99% /
/dev/sda3              173G  162G  2.4G  99% /
/dev/sda3              173G  162G  2.4G  99% /
proc                      0     0     0    - /proc
/dev/sda3              173G  162G  2.4G  99% /
tmpfs                   51G  275M   51G   1% /run
/dev/sda3              173G  162G  2.4G  99% /
/dev/nvme0n1           1.5T  775G  618G  56% /scratch
/dev/sda3              173G  162G  2.4G  99% /
sysfs                     0     0     0    - /sys
/dev/sda3              173G  162G  2.4G  99% /
/dev/sda3              173G  162G  2.4G  99% /
/dev/sda3              173G  162G  2.4G  99% /
/dev/sda3              173G  162G  2.4G  99% /
/dev/sda3              173G  162G  2.4G  99% /
```

As you can observe from the output above, I have plenty of spaces for volume `/dev/nvme0n1` and `/dev/mapper/data-data`, however I was getting "No space left on the device" error, because root path became full due to docker volumes. 

It can be quickly checked by the command below. 

```bash

$ df /var/lib/docker

Filesystem     1K-blocks      Used Available Use% Mounted on
/dev/sda3      180572828 166898524   4432040  98% /
```

## Temporary solution
A temporary solution would be pruning docker volumes by running ; 

```bash
$ docker volume prune 
```

it will prune all volumes which are NOT in-use,  if the volumes which are creating this bunch of data are in-use, prune command will not have any effect on the error. Although it works, it is a temporary solution, it may re-trigger the error in future. To resolve this issue permanently following approach could be used: [Permanent Solution](#permanent-solution)

The approach of having temporary solution can be extended with cronjobs, however, it is definitely not nice way of handling issues. 

However, it could be nice to have  cron jobs which prune docker system time to time, because old docker images, containers and some other leftovers can create bunch of reclaimable storage. Docker prune commands do not have effect on resources (containers, volumes and networks) which are under use.   

You can setup a cronjob which weekly prunes system, as provided below. 

```bash

$ crontab -e 

0 0 * * 0 docker system prune -f  # this will run at 00:00 on Sunday everyweek
0 0 * * 0 docker volume prune -f # will remove all (NOT in use) volumes weekly
0 0 * * 0 docker network prune -f # will remove all (NOT in use) networks 

```

*You can write a script according to your needs then place it to cron job as well. Before adding cron jobs, make sure $USER has valid permissions for docker group.*

Now, we are sure that the error caused due to docker volumes which means we can proceed with a permanent solution. 

If it the error is not caused due to docker volumes, it could be caused from running short of inodes or basically not enough storage area. 

# Permanent Solution 

Permanent solution is using one of storage volume for storing docker volumes, instead of using root path. These are the steps that you may take; 
 
__Stop docker service !__
   
```bash
 $ systemctl stop docker
```

__Rsync all data under `/var/lib/docker` to other directory under storage volume__
 
```bash
 $ mkdir -p /data/mnt  # creates plave to use for docker volumes
 $ rsync -a /var/lib/docker /data/mnt/  # will sync everything 
```

__Rename default docker volumes path, this for taking a backup until we have success at the end.__
 
```bash
 $ mv /var/lib/docker /var/lib/dockerbckp
```

__Create symbolic link to actual place__

```bash
 $ ln -s /data/mnt/docker /var/lib/docker 
```

__Enable `DOCKER_OPTS`__

```bash
$ vim /etc/default/docker
 DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4 -g /data/mnt/docker"
```

Uncomment `DOCEKR_OPTS` add `-g /data/mnt/docker` as shown above in `/etc/default/docker`

__Start Docker daemon__
 
```bash
$ systemctl start docker
```

If there was no error during implementation of steps, you can now test your setup. 

*Note that thee should NOT be space between curly brackets when listing mount point only.*

```bash
$ docker volume create 
2f2bd462b89c39bb641e7daf01048c5d811dd7796d7f89d250ea82c4532d2707
$ docker volume inspect --format { {.Mountpoint} } 2f2bd462b89c39b641e7daf01048c5d811dd7796d7f89d250ea82c4532d2707 
/data/mnt/docker/volumes/2f2bd462b89c39bb641e7daf01048c5d811dd7796d7f89d250ea82c4532d2707/_data
```



As you can observe above docker is using `/data/mnt/docker` for docker volumes, now we can safely remove old backup data. `rm -rf /var/lib/dockerbckp`

I would like to mention that there is an information about changing docker volume instead implementing all steps mentioned above, just specifiying path under `/etc/docker/daemon.js` file would work (-example daemon.js is given-). However, when I tried that approach, it did not worked. 

__Example__

```bash
$ vim /etc/docker/daemon.js

{
 "data-root": "/mnt/docker-data",
 "storage-driver": "overlay2"
}
```

*Check this documentation:* <a href="https://docs.docker.com/config/daemon/systemd/">https://docs.docker.com/config/daemon/systemd/</a>

I hope, this post can help others to find required information quickly and implement necessary steps to get to work. 
 



# References:

-  <a href="https://github.com/moby/moby/issues/10613">https://github.com/moby/moby/issues/10613</a>
-  <a href="https://docs.docker.com/config/daemon/systemd/">https://github.com/moby/moby/issues/10613</a>
-  <a href="https://success.docker.com/article/error-message-no-space-left-on-device-in-default-machine">https://success.docker.com/article/error-message-no-space-left-on-device-in-default-machine</a>
- <a href="https://forums.docker.com/t/how-do-i-change-the-docker-image-installation-directory/1169">https://forums.docker.com/t/how-do-i-change-the-docker-image-installation-directory/1169</a>
