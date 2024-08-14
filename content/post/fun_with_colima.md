+++
title = 'Fun with Colima'
date = 2024-08-14T10:28:59-05:00
draft = false
summary = "How I prefer to run containers on my Apple Silicon Mac."
tags = ["colima", "docker", "lima", "qemu", "arm", "x86", "macos"]
+++
# Running Containers on my (M1Max) Mac

Obviously, in today's environment, you're going to at least need to run docker containers from time to time. Some folks are "all in" and run them for nearly everything, and some, need to just run them occasionally. I'm not sure where I fit in that. I guess I'm more the occasional category. For simple development tasks, I'm not going to go through the trouble of creating a container just for development purposes and instead will just develop directly against MacOS. But often, you may need or want to run additional pieces of software that are best run from a container.

Obviously, if [licensing issues](https://www.docker.com/products/docker-desktop/) aren't a concern, then you can just run Docker Desktop. But since their change to a "license required" for certain categories of businesses, I like using something that'll be consistent regardless of whether I'm doing work or personal stuff. Enter [Colima](https://github.com/abiosoft/colima). But the super cool part of Colima is that you can run both x86 and Arm containers on an Apple Silicon Mac.

## Prerequisites

On my Mac, I've `brew` installed both Colima and docker (for the command line interface).

```shell
brew install colima
```

## My Setup

So for my default, I allow Colima to follow its default architecture, which is Arm (aka `aarch64`). The first time you start up `colima`... it'll download the vm setup for your current architecture. And subsequent starts look roughly like this after completing:

```shell
❯ colima start
INFO[0000] starting colima
INFO[0000] runtime: docker
INFO[0001] starting ...                                  context=vm
INFO[0012] provisioning ...                              context=docker
INFO[0012] starting ...                                  context=docker
INFO[0013] done
```

Then you can also setup an secondary profile for running x86 containers, I named the profile "x86" for clarity:

```shell
❯ colima start -a x86_64 -p x86 --vz-rosetta
INFO[0000] starting colima [profile=x86]
INFO[0000] runtime: docker
INFO[0001] starting ...                                  context=vm
INFO[0049] provisioning ...                              context=docker
INFO[0049] starting ...                                  context=docker
INFO[0055] done
```

For subsequent starts of this profile, you can just do `colima start x86`

{{< callout note >}}
This will take much longer to start (sometimes a minute or two) than a native VM, even on subsequent starts, it's not fast.
{{< /callout >}}

{{< callout warning >}}
I am careful to run just one profile at a time. Colima will allow you to run multiple, but the `unix:///var/run/docker.sock` is only going to be bound to the first running Colima instance. You can work around this with additional docker settings, but I'm trying to keep things super simple.
{{< /callout >}}

To stop the `default` Colima instance:

```shell
> colima stop
```

And to stop the `x86` profile:
```shell
> colima stop x86
```

So initially, you should now see this (with all VMs stopped):

```shell
❯ colima list
PROFILE    STATUS     ARCH       CPUS    MEMORY    DISK     RUNTIME    ADDRESS
default    Stopped    aarch64    2       2GiB      60GiB
x86        Stopped    x86_64     2       2GiB      60GiB
```

To run an x86 container, you start the `x86` profile again and use your docker containers like normal. But these will be x86 containers.

```shell
❯ colima start x86
...

❯ docker run -it --rm python python -c "import platform;print(platform.platform())"
Linux-6.8.0-36-generic-x86_64-with-glibc2.36
```

Likewise, if you (stop the x86 VM first) start the default (Arm) VM in Colima, you will be using Arm containers.

```shell
❯ colima start
...

❯ docker run -it --rm python python -c "import platform;print(platform.platform())"
Linux-6.8.0-31-generic-aarch64-with-glibc2.36
```

Obviously, there are tons of other options and things to set and try and ways to optimize, but wanted to point out just how simple it can be to run basic containers for both architectures.
