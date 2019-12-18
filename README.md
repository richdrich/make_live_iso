make_live_iso
=============

# Introduction

This repo builds a Docker image that can then be used to create a bootable Ubuntu Live ISO file.

I've based this on the instructions at https://github.com/mvallim/live-custom-ubuntu-from-scratch

You can use this as it stands to build a basic installable (with the [Xfce desktop](https://www.xfce.org/)) 
or you can customise to add additional packages (such as a browser) to meet specific needs.

# Prerequisites

- An OS/X or Linux (not yet tested) desktop, or compatible shell.
- [Docker](https://docs.docker.com/install/) (I have desktop 2.1.0.4, engine 19.03.4)
- About 30G of free disk

A virtualisation application such as VirtualBox is useful for testing

To boot on an actual machine, you can write a USB or SD card, or if you have retro hardware, a DVD.
 
# Usage (from repo)

Ensure you have Docker running (and no other Docker containers active (fixme))

Clone this repo: `git clone https://github.com/richdrich/make_live_iso.git` and cd to it `cd make_live_iso`

Build the image (takes a while): `./docker_build_image`

Generate the iso `builder/generate_iso`

The ISO file will be in 'out/live_stick.iso'.

To write this to a USB stick or SD card, [Balena Etcher](https://www.balena.io/etcher/) is recommended.

If you have an issue in the build and a Docker container is left running, `./docker_rm` will fix.

# ssh access

If you want to configure your booted OS for ssh access (e.g. for development) then setting `SSH_PUB_KEY` to your public key path before 
running `generate_iso` will do this. 
This copies the public key into the `~ubuntu/.ssh/authorized_keys` which grants an SSH user access.
You can then configure your ssh client to use the corresponding private key and log in as ubuntu.
You may also want `StrictHostKeyChecking no` as the host id will change every time you build.

* NOTE: this will authorize access to every system booted from the ISO and is of limited use for 'production' *  
 
# Customization

You can customise the ISO by extending the `make_live_iso` image.

- Create a Dockerfile in your own directory with:
```
FROM make_live_iso
```

The install process uses three customisation files:

*chroot_custom* runs as chroot (e.g. as if it was in the booted system) and installs any packages you require.

It looks like: 
```
#!/bin/bash -x

echo "chroot_custom"

mount none -t proc /proc
mount none -t sysfs /sys
mount none -t devpts /dev/pts
export HOME=/root
export LC_ALL=C

echo "APT::Acquire::Retries \"3\";" > /etc/apt/apt.conf.d/80-retries

[ INSTALL, ETC ]

apt-get clean
rm -rf /tmp/* ~/.bash_history
umount /proc
umount /sys
umount /dev/pts
export HISTSIZE=0

echo "chroot_custom"
```

Use a stanza like:

```
ADD chroot_custom /live-ubuntu-from-scratch/chroot_custom
RUN chmod a+x /live-ubuntu-from-scratch/chroot/chroot_custom
```

in your Dockerfile to install this.

*pre_custom* runs outside chroot and can be used to build any required custom code, etc

Install with e.g:

```
ADD pre_custom /root
RUN chmod a+x /root/pre_custom
```

*post_custom* runs after and outside the chroot and can be used for any other customisation commands.

 
