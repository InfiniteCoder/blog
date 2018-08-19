---
date: "2018-03-29T00:00:00Z"
gh-badge:
- star
- fork
- follow
gh-repo: infinitecoder/gcc-docker
published: true
aliases:
    - /2018-03-29-building-gcc-docker-image/
title: Building GCC docker image
---

When you look at the various official images for docker, you'll notice that they are built with stability in mind more than anything. This can be noted by the fact that they use Debian as their base, instead of the much lighter Alpine Linux. Using Debian provides a rock solid base for whatever you build upon it. However, this comes at the cost of size. Official docker images can be humongous compared to their community built alternatives.

I needed a GCC 7.3 image for my project. The official image for GCC stands at a compressed size of 526MB, while when uncompressed, it's size reaches 1.6GB. So I decided to build my own GCC image, using Alpine Linux as a base.


## Basics of building GCC
The process to build and install GCC might look straighforward at first. All you have to do is install dependencies, download and extract the source code and then execute following commands
``` sh
./configure
make bootstrap
make install
```

However, a multitude of configuration options, along with requirement of many dependencies and platform specific requirements can make this process somewhat complicated.

## Why use Docker?
Docker is an easy way to get a clean slate everytime you want to try something different. As I was trying out various configuration options, and installing and removing various packages, using docker meant I could easily figure out what needs to be done, and what needs not be done.
Also, the Dockerfile thus created can easily be shared with others, so that others could build GCC easily with minor changes made as required.

## Configuring GCC
There are a few things that you should note when building GCC,

1. **Native build or cross compilation?**  
GCC allows you to build a compiler either for the native system, or some other system. You can build a cross compiler by specifying the `--target` option. Which one you choose would depend upon many factors such as,
   1. **Is the target powerful enough to build GCC?**  
For less powerful target systems such as embedded system, it might be best, or even necessary to build the compiler on another more powerful system. Due to RAM restrictions, many embedded systems won't even be able to build GCC.
   2. **Does the target already have g++?**  
Some systems might not provide the GCC compiler package. Some systems like NAS Servers won't even have a package manager/repositories. For such systems, building a cross compiler is the only option.
   3. **Glibc vs Musl**  
If you are using musl instead of glibc, then you must use cross compiler as that is the only way to tell GCC to use musl instead of glibc.

2. **Multilib or not?**  
Multilib allows 64bit systems to produce binaries for 32 bit systems. If you want to enable this, make sure you have the 32 bit libs for target system installed as well. Otherwise, disable it with `--disable-multilib`.

3. **Languages**  
You would want to specify only those languages that you use. You can do this using the `--enable-languages` option. I selected `C` and `C++` languages.  

> **Parts of C compiler are written in C++, so C and C++ are the minimal set of languages you can use.**

## Building GCC
Once done with the configuration, you can build GCC. You can use

    make bootstrap
    
if you want to use the recommended way of building GCC 3 times. To improve it's performance more, I choosed

    make profiledbootstrap
    
which profiles the compiler at each stage of the build.
You can also provide optimisation level and debugging information level, using `BOOT_CFLAGS` as such

    BOOT_CFLAGS='-O3 -g0'
    
You can use multiple threads using `-j` option of make.

> **Use `nproc --all` to get number of processors, and pass that value to `-j`. Ideal value lies between X and 2X of processors.**

The make command I used looks like

    make -j$(($(nproc --all) * 2))  BOOT_CFLAGS='-O3 -g0' profiledbootstrap

## Gotchas when Building
1. If you get errors during building, make sure you are using single thread, as using **multiple threads will entangle the logs** of different threads.
2. When checking the logs, make sure you **check the logs from correct directory**. The log in root directory might not be the correct one. 
3. Also **check the logs from bottom up**, trying to find the last error that occurred. Many errors would have occurred while GCC tried different options, and the last one is generally the one you would be interested in.
4. Check available disk space with

        df
       
    which should give output like below
    
    ``` sh
    $ df
    Filesystem     1K-blocks    Used Available Use% Mounted on
    udev             3928264       0   3928264   0% /dev
    tmpfs             787252   82480    704772  11% /run
    /dev/nvme0n1p1  20263528 6897228  13349916  35% /
    tmpfs            3936248       0   3936248   0% /dev/shm
    tmpfs               5120       0      5120   0% /run/lock
    tmpfs            3936248       0   3936248   0% /sys/fs/cgroup
    tmpfs             787252       0    787252   0% /run/user/1000
    ```
    
    If you are running out of space, remove unused docker containers and images with the commands
    
       ``` sh
       docker container prune
       docker image prune
       ```
    
    
    An example output could look like,
    
    ``` sh
    docker image prune
    WARNING! This will remove all dangling images.
    Are you sure you want to continue? [y/N] y
    Deleted Images:
    deleted: sha256:9a2d252b9015272eb8ee3b44c90eb13132ef95127ae2cf1738f3770b0a1e3491
    deleted: sha256:22c2a75428d929c989b97181076330ff055724e5213ad46409fd1d09e6e1eb5c
    deleted: sha256:e8e34282a265c989d0a7e0678babe37e0a25c061bea8a61d62bbaad85b8d1fe0
    deleted: sha256:a9a515f086bbb042f2f0af0558ebbd54d4274af89054a47fb744b89a3ff9abf1
    deleted: sha256:77f717e737025fa4b595064dd3214ee726f1d21f113067b6be4a8ee772fd80bd
    deleted: sha256:953a9b3a694a2b2c2e97f0d8e17d8209049d17455427e54bd074f60a043cfe71
    deleted: sha256:59d0199fa53ea985c25dc0108e688c08fe4339cb4727949f940cda7085a48d7a
    deleted: sha256:380fb7cf74eea115cc527e2f3f87a50b8b33ad579355e0fdea21439b8a98b677
    deleted: sha256:6cf15d487d230e57ef9ea51b3a26d772a0390f2d397a2dccccb95eaf3fc356db
    deleted: sha256:b7d698cae9bd949d8f206dcf972db24547d877fd2ace30b6bc22f9e3337973d1
    deleted: sha256:51b68531cfb4762bc16a56437c2057df9efef9cf410fcffa713b6378875238fa
    deleted: sha256:f6675c88247cf68ea48d93233fefb9d168df46e515536e6d3ae0b7547708721c
    deleted: sha256:3ba3b9e70e7110bb0d5d946e371154c4ad24d91b289ac484282cce51ddcd4904
    deleted: sha256:8c12bbc7fd1e444f61a7feabbd698f9a784dc5df79bcdfe5d958aeaefd5beffa
    deleted: sha256:68facfd6610c8c7e66f7eb913aa799b067560c732a3b92e6195d003bf225a569
    deleted: sha256:86571571e5b340f201236b81f024a8366796619c5b5a3047309219c6980d7e3a
    deleted: sha256:c3b95b846470d4c412b1e14a7c7f64b3ddff0a973ddc4d605d51fcd6eb356884
    deleted: sha256:ee87d8dcfbcfbc487b31b9566cc1baa4c89ce3977a6b48eed87df102c4201af7
    deleted: sha256:53adecc4e5d9ced48b4af060c6eabc7886084429e2a1e7c18cdc322a5a5b2062
    deleted: sha256:17f9763c498717d25560313cef8b380a9779bb123e2183bda26cdcbacc25339d
    deleted: sha256:4796ae729edd6d1d4e2f59263281d1b880b001d1b0288a198d5ff9f726761cf5

    Total reclaimed space: 4.89GB
    ```
    
## Build System
The system you use to build will make a lot of difference on the time requried to build GCC. The AWS's `m5.large` system with `2vCore and 8GB RAM` system takes 4+ hours, while the much more powerful `c5.18xlarge` with `72vCore and 144GB RAM` takes only 15 minutes. However while the m5.large gets maxed out, the c5.18xlarge uses it's full power only occasionally. This is probably the reason why the speedup is not linear in this case.

## Installation
You can install GCC with all it's meta information(eg: debugging symbols) with

    make install
    
or install a stripped version with

    make install-strip
    
By default, the files are stored in /usr/local. If you were building a cross compiler, copy files from this directory into appropriate location.

## The Dockerfile
While you could run the previous commands manually into a interactive session of Alpine Linux as follows,

    docker run -it alpine:3.7
    
Writing a Dockerfile will allow you to easily build and share your image.

Previously, dockerfile contained linux trickery like using && to put all commands in a sinle `RUN` command of docker. This was done to avoid the overhead of multiple layers created due to multiple RUN commands. However, the newer version of docker provides ability to create multi-stage docker builds.

In multi-stage docker builds, you can seperate your commands into multiple stages, where each stage starts with a clean base image and copies only those files that it needs from the previous stage. You can have multiple `RUN` commands in a single stage, and not worry about all the unnecessary files that come with intermediate layers, as you will be throwing them away.

The complete dockerfile is available on [GitHub here](https://github.com/InfiniteCoder/gcc-docker).
