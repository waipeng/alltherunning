---
layout: post
title:  "Tips for running NeCTAR VMs"
date:   2016-04-18 12:00:00
categories: NeCTAR
---

It has been a real privilege to be able to work on _NeCTAR Research Cloud_ over
the last year and half. It has given me much experience on how Melbourne Node's
hardware and OpenStack work together. With this knowledge, I've put together
some tips on spinning up a NeCTAR VM, or ___"What should I do when I launch a
NeCTAR VM?"___

# Spinning up a VM

## Flavor

Which flavor your choose depends on how much resources you need. There are 3
things in mind - _CPUs_, _Disk_ and _RAM_. Spin up as _small_ a flavor as
possible - this ensures you don't burn through your quota, and also leaves
capacity for other users to spin up their VMs. If it is possible, try spinning
up multiple smaller instances rather than one big one. A big instance consumes
a big chunk of the compute host it resides on, making it difficult to pack
multiple instances onto each host.

For example, some of our older compute hosts have 32 CPUS. With 1 CPU reserved
for the OS, we can allocate either:

- 1 * 16 VCPUs instance, or
- 3 * 8 VCPUs instances, or
- 7 * 4 VCPUs instances.

It is clear that the number of 16 VCPUs instances that can be launched will
never exceed the number of hosts, and thus it is far harder to find a space to
launch a big instance than a small one.

### m1 vs m2 flavors

The default flavors currently are the _m2_ flavors. They differ from _m1_
flavors as they do not have ephemeral storage for the smaller sizes. They are
also much more reasonably sized (VPU to RAM to Disk ratios). I would strongly
recommend using m2 flavors, it makes things much easier for you and us. I also do
not use the ephemeral storage, but put my data in _Volumes_ (more on that later).

## Image

NeCTAR official images includes _CentOS_, _Debian_ and _Ubuntu_. My favorite image
will probably have to be Ubuntu, in part because NeCTAR RC runs on Ubuntu! As
such, I would think that you'll have better support using Ubuntu than others,
due to NeCTAR Operators' experience and familiarity.

## Availability Zone
NeCTAR has multiple _Availability Zones_ (AZ) all over Australia. When choosing
your AZ, remember that it has to be in the __same__ AZ as your volume storage, if
you want to attach volumes to your instances. Where your volumes can be
provision will depend on your allocation(s), unless you are using a _pt-*_
(personal tenancy) project.

## Spin it up
For a normal test VM, I will usually spin up an _Ubuntu_ _m2.small_ VM. Smaller
flavors might not work for Ubuntu, and the bigger flavors are just a waste of
resources for normal use (web service, scripting, etc).

# Volumes
While the instance is spinning up, I will normally create a _Volume_ for any
data files this VM will use. The advantages of using _Volumes_ over _root_ or
_ephemeral_ storage are:

- Ephemeral storage are __not__ backed up. Snapshots of your instance only saves
  the _root_ disk, and not the _ephemeral_.
- Snapshots of instances are slow. While snapshotting, the instance needs to be
  paused, snapshotted, and resumed. Some disruption might be encountered if
your applications are sensitive.
- Snapshot of _Volumes_ are (almost) instaneous. A _Volume_ snapshot is a
  backend operation - this means that a command is sent to the storage backend
to do a copy-on-write snapshot. There is no disruption while snapshotting _Volumes_.
- It is far easier to move or clone a _Volume_. If your VM breaks, you can easily
  detach your _Volumes_ and reattach them to a newly rebuild VM.
- _Volumes_ can be extended, but not _root_ or _ephemeral_ disks.

## Attach and format your volume
Create and attach your volume to your instance. You now need to format it. As `root`

    root@tools:~# mkfs.ext4 /dev/vdb
    mke2fs 1.42.12 (29-Aug-2014)
    Creating filesystem with 262144 4k blocks and 65536 inodes
    Filesystem UUID: a424b665-554f-4451-b8de-fa38dc31c790
    Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376

    Allocating group tables: done                            
    Writing inode tables: done                            
    Creating journal (8192 blocks): done
    Writing superblocks and filesystem accounting information: done


Take note of the _UUID_ `a424b665-554f-4451-b8de-fa38dc31c790`, you need this
for creating your fstab entry.

Create a mountpoint for your volume

    mkdir /mnt/apache

Edit `/etc/fstab` and add the following

    UUID=a424b665-554f-4451-b8de-fa38dc31c790 /mnt/apache   ext4  rw,nofail 0 0

We use _UUID_ instead of the convention _device file_ `/dev/vdb`, because device
mapping might change between reboots if you have multiple devices and you
remove one of them. The `nofail` option allows your VM to continue booting if
you detach the _Volume_.

I also find that it's a good idea to create a different volume for each service
(apache, mysql), so that you can move volumes to new VMs to scale them up,
etc.

That's all for now, I'll be adding more information when things come up, so
feel free to check back!
