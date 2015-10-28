---
layout: post
title:  "Docker error on Debian"
date:   2015-10-28 00:00:00 -3
categories: docker debian
---

I created a simple docker file with Ubuntu + Python:

```
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get -y install python2.7-dev python-virtualenv
RUN apt-get clean
```

So, to build the image, I run:

``` bash
docker build -t my_app:`git rev-parse HEAD` .
```

(yes, I include the git hash of the repository to get the exactly deployed version)
And get this error:

``` bash
Sending build context to Docker daemon 44.03 kB
Sending build context to Docker daemon 
Step 0 : FROM ubuntu:14.04
 ---> a5a467fddcb8
Step 1 : RUN apt-get update
 ---> Running in 68244d77765e
[8] System error: open /sys/fs/cgroup/cpu,cpuacct/init.scope/system.slice/docker-68244d77765e809d677eff58bdf77f88f721935b1e80871ce09aae111d0e3b11.scope/cpu.shares: no such file or directory
```

This is because docker is incompatible with systemd >= 226. There is a [Debian bug report] and a [docker issue] too.


To solve that, just add this configuration to /etc/default/docker:

```
DOCKER_OPTS="--exec-opt native.cgroupdriver=cgroupfs"
```

After that, you should restart docker service (as root):

``` bash
systemctl restart docker.service
```

[debian bug report]:https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=798784
[docker issue]: https://github.com/docker/docker/issues/16256
