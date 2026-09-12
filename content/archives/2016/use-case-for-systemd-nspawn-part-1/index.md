+++
date = '2016-09-24T01:35:48-05:00'
title = 'A Use Case for Systemd Nspawn (Part 1)'
tags = ['systemd-nspawn', 'rpm', 'rhel7', 'containers']
+++

Docker is all the rage in the world of containerization, but there is another container technology 
that may prove to be invaluable for certain use cases… Enter [systemd-nspawn](http://0pointer.de/public/systemd-man/systemd-nspawn.html).

In this article I will outline a use case for systemd-nspawn. All examples below use the RedHat variant (centos).

## Nspawn container as an RPM build host

Part of my daily routine at work is to manage specific RPM packages for inclusion in internal 
repositories. An interesting use of nspawn is as a build/test host. Setting up nspawn on a RHEL7 
(Oracle, Centos, Scientific, etc.) is pretty trivial:

## RHEL7 Style Setup

```
yum -y --release=7 --installroot=/srv/ns-host install systemd passwd yum rpmdevtools yum-utils
cp /etc/os-release /srv/ns-host/etc/os-release
systemd-nspawn -D /srv/ns-host
passwd root
useradd rpm999
passwd rpm999
exit
```

### What did we do?

We installed a release 7 filesystem into the directory `/srv/ns-host` with some required packages. 
The second step copies over the os-release file to the container (this is required as of systemd-219). 
Next we temporarily start the container by passing the `-D` flag and filesystem location. Finally we 
add the root password and create a user for our RPM builds. It is best practice to run your rpmbuilds 
as a non-root user, even in a container.

## Boot your new container

```
systemd-nspawn -bD /srv/ns-host
login as rpm999
rpmdev-setuptree && mkdir incoming
```

Copy your source data to the rpmbuild directory. You may utilize external tools (ie - tools outside 
of the container - git/vim) for editing text or revision control.

## Systemd’s machinectl interface

The machinectl interface gathers data published by the kernel and can be used to start/stop containers 
(and vms).

### Setup:

```
ln -s /srv/ns-test /var/lib/machines/.
machinectl start ns-test
machinectl (Shows running vms/containers)
machinectl enable ns-test (Enable container start at boot time)
```

In order to ensure that your builds are isolated from the network it is important to disable network 
sharing to the container by changing the `--network` flag to `--private-network`. The config is located 
at `/etc/systemd/system/machine.target.wants`.

```
# /etc/systemd/system/machine.target.wants
# This file is part of systemd.
#
# systemd is free software; you can redistribute it and/or modify it
# under the terms of the GNU Lesser General Public License as published by
# the Free Software Foundation; either version 2.1 of the License, or
# (at your option) any later version.
[Unit]
Description=Container %I
Documentation=man:systemd-nspawn(1)
PartOf=machines.target
Before=machines.target
[Service]
ExecStart=/usr/bin/systemd-nspawn --quiet --keep-unit --boot --link-journal=try-guest --private-network --machine=%I
KillMode=mixed
Type=notify
RestartForceExitStatus=133
SuccessExitStatus=133
Slice=machine.slice
Delegate=yes
[Install]
WantedBy=machines.target
```

In the next post I will show you how you can make an ephemeral container instance using btrfs and nspawn.
