---
layout: post
title: Alpine on a router
---

In this article I'll show simple steps to install Alpine in a chroot environment
on a SOHO router.

## Motivation

Nowadays many SOHO routers are quite powerful devices. Probably, not as powerful
in some aspects as latest Raspberry Pi but powerful enough to run several
additional services besides basic routing functionality. In fact, many routers
provide Samba and other file servers for years. Some powerful models contain
even BitTorrent clients, DLNA servers, VPN clients and servers in the stock
firmware.

On the other side, router vendors firmware is rarely ready for extension. Some
vendors even lock their firmware so enthusiasts need to hack it first if they
want to add anything to their devices. Luckily, there are many vendors which do
provide telnet or ssh access to the device console.

Still, even if one gets to the console, those devices don't contain any
development tools and there are no vendor repositories for third party software.
So enthusiasts need to create their own repositories. One of well known examples
is Entware which can be used on many device brands, models and flavours.

It worth noting that router firmwares are usually quite different from a typical
multipurpose Linux distribution. As a result, Entware adopts the practice of
installing everything under `/opt` filesystem hierarchy. Installing software
under a prefix like `/usr/local` is not something unknown in a *NIX world,
however, the weird thing is that Entware can't rely on "host" layout and
libraries at all, so it needs even its own `/opt/etc` and its own C library, as
host can use anything of uClibc, musl or glibc.

Does it remind you of anything? When one wants to run applications independently
of the host system, they could run them in a virtual machine. Or in the modern
Linux world, in a container.

Well, I'm not crazy enough to run Docker on a router (yet). But what about
chroot? Some linux distributions have support for running in chroot. The
distribution used for lightweight containers most often, Alpine, also has
[instructions how to setup a chroot](https://wiki.alpinelinux.org/wiki/Alpine_Linux_in_a_chroot).

## But why, actually?

- There's enough spare power on the router to run some services 24x7 and I don't
need an additional box for that.
- Current Entware repository for ARM x64 architecture
[contains about 2800 packages](https://bin.entware.net/aarch64-k3.10/Packages.html).
Alpine contains more than 5000 packages in its "stable" repo and 14000 in
"community".
- Entware is maintained by a few enthusiasts. Alpine has thousands of users in
production including big enterprises so it gets updates faster.
- Entware adopts some compromises to run on a low-resource hardware in a
non-standard system layout and without conflicts with the firmware. Alpine is
purposely built for low resource usage but follows standard layout.
- Alpine uses a lightweight musl C library (which has its own downsides).
- Alpine is used by many Docker images so if I need to run some application
which is not packaged for Alpine, it's likely it has an Alpine-based Docker
image so the recipe can be extracted from it.

## Okay, let's give it a try

My router runs a so called
[Asuswrt-merlin firmware](https://www.asuswrt-merlin.net/).
I already have an attached USB drive formatted to ext4 and Entware installed on
it.

Probably, it should be possible to follow
[instructions from the Alpine wiki](https://wiki.alpinelinux.org/wiki/Alpine_Linux_in_a_chroot)
step by step without installed Entware. However, someone already scripted that.
Unfortunately, the script uses some utilities not provided by the limited router
firmware so I need Entware.

I've made a couple of modifications so I don't need additional configuration of
the script for my router, they are in
[in the aarch64 branch of my fork](https://github.com/pelepelin/alpine-chroot-install/tree/aarch64).
See the README there for more details.

> **Disclaimer:** I'm writing these instructions from memory. I did this a
> couple of times but something can be missing. I'll fix that when I'll use my
> own recipe next time.

So, first I need to install some Entware packages for that script to work:

```shell
opkg install coreutils-id coreutils-mktemp bash coreutils-sha256sum
```

You may probably need to install `curl` also because router `wget` may lack
HTTPS support.

Download the script with `wget` or `curl`.

```shell
wget https://github.com/pelepelin/alpine-chroot-install/raw/aarch64/alpine-chroot-install
chmod +x alpine-chroot-install
```

Find the path for the installation. In the example below the storage is mounted
as `/mnt/disk` so the command will install Alpine under `/mnt/disk/alpine` which
should not exist before installation.

```shell
./alpine-chroot-install -d /mnt/disk/alpine
```

It should download and install a minimal alpine environment. I can chroot into
it with an installed script:

```shell
/mnt/disk/alpine/enter-chroot
```

Exit chroot by exiting the shell.

## Further steps

Now I have a way to enter the chroot environment, play with Alpine packages
there and do all the things.

Unfortunately, once the router is rebooted, mount points created by install
script will disappear. So I need to put their creation into the `enter-chroot`
script (and ultimately, fix it in the install script). Unmount script (called
`destroy`) is not yet robust too.

Chroot environment provides only limited isolation. First problem I've found is
that ID mappings used by the router in its `/etc/passwd` and `/etc/group` don't
match Alpine base ID mappings. I had to find some compromise. In theory, it
should be possible to use `cgroups` for ID mapping and probably more isolation,
up to the level which is provided by Linux containers but I'm not sure if I
really need it. Anyway, Entware provides `unshare` utility to play with that.

On the other side, it would be good to make installation script independent of
Entware utilities.

If I want any Alpine service to auto-run when the router boots up, I need to
hook it into router scripts. AsusWRT-Merlin firmware provides a lot of hooks for
that. At least, I'll use
[`post-mount`](https://github.com/RMerl/asuswrt-merlin.ng/wiki/User-scripts#post-mount),
[`unmount`](https://github.com/RMerl/asuswrt-merlin.ng/wiki/User-scripts#unmount)
and
[`services-stop`](https://github.com/RMerl/asuswrt-merlin.ng/wiki/User-scripts#services-stop).

For starting and stopping services Alpine uses
[OpenRC](https://docs.alpinelinux.org/user-handbook/0.1a/Working/openrc.html)
init system. OpenRC provides some support for chroot environment but that
requires
[additional configuration](https://wiki.gentoo.org/wiki/OpenRC#Chroot_support).
