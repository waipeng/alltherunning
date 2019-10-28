---
layout: post
title:  "Passing entrophy to virtual machines"
date:   2019-10-01 12:00:00
categories: openstack
---
Recently, when we were working on testing new images with Magnum, I found that
the newest Fedora Atomic 29 images were taking a long time to boot up. A closer
look using `nova console-log` revealed that they were getting stuck at boot
with the following error.

```
[   12.220574] audit: type=1130 audit(1555723526.895:78): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=systemd-machine-id-commit comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
[   12.248050] audit: type=1130 audit(1555723526.906:79): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=systemd-journal-catalog-update comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=success'
[ 1061.103725] random: crng init done
[ 1061.108094] random: 7 urandom warning(s) missed due to ratelimiting
[ 1063.306231] IPv6: ADDRCONF(NETDEV_UP): eth0: link is not ready
```

The number between the `[]` shows the number of seconds since boot, as you can
see the VM was stuck for >1000 secs waiting for `crng init`.

It turns out that in some newer images, boot will block waiting for sufficient
entropy. Entropy, or randomness, is used in operating systems for important
things like random number generation. Traditionally machines generate their
entropy by looking at inputs that are random, e.g. disk writes, mouse movement.
(Fun experiment: On a Linux machine do a `cat /dev/random`, wait for the output
to stop and move your mouse.)

A fresh VM has very little avenues to collect entropy, so unfortunately if
doesn't have enough entropy it may block. Luckily the smart people at QEMU has
a solution called [VirtIO RNG](https://wiki.qemu.org/Features/VirtIORNG), which
involves passing entropy from the host hypervisor to the VM. This allows the
VM to seed it's entropy pool and happily continue booting.

[Openstack config docs](https://wiki.openstack.org/wiki/LibvirtVirtioRng)
points out that you need to set this in two places, both at flavor and at
image. The flavor setting controls whether an image booted with this flavor is
allowed to drain the host's entropy, and what rates they are allowed to drain.
Finding the correct rate is a bit of trial and error, as you want a high enough
rate so that the VM will not block, but low enough that a malicious VM will not
be able to totally drain a hypervisor's entropy.

With a bit of testing, and advice from our fellow OpenStack Operators at
Catalyst NZ, we have found the following values to work for us:

* `hw_rng:allowed='True'`
* `hw_rng:rate_bytes='24'`
* `hw_rng:rate_period='5000'`

NOTE: as our friends from Catalyst points out, `rate_period` is specified in
milliseconds and not seconds like what some documentation states.

### Same rate, different periods?
One of the question that we had to answer was, what is the effect of the period
setting? For example, if we know that we need 5 bytes/second of information, is
it better to set

1. 1 byte in 1 second, or
2. 5 bytes in 5 seconds?

Both of this settings translate to the same effective _rate_, but their
performance can be very different.

In [this
note](https://wiki.qemu.org/Features/VirtIORNG#Effect_of_the_period_parameter),
it was suggested that a smaller period is better because it means your process
will block for a shorter time. However, a longer period can have some advantages
in a cloud environment.

1. in case of a benign VM, we want it to be able to burst as much as possible.
   If it needs 5 bytes it can get it in the first sec rather than wait till the
5th sec

2. in case of a malicious VM, being able to block it for 5 secs means other
   VMs will be able to have a better chance to consume entropy

Hence, we have set a relatively large period on our environment.
