---
layout: post
title:  "State of the Cloud 2019"
categories: openstack, nectar
---

Now that 2020 is upon us, I thought it might be a good idea to generate some
statistics about the Nectar Cloud for 2019.

# Instances #

In 2019, Nectar Cloud ran a total of **70,371** instances.

## VCPU time ##

These instances ran for a total of **9,203,375 days, 19 hours 17 minutes and 59
seconds** of *VCPU time*[^1]. That is around **25,214** VCPU years!

The *mean VCPU time* is about **130 days**.

The *mode VCPU time* is **365 days**, which means there were lots of single core
instances running through the year.

## Flavour ##

The most popular flavour is **m2.large** (4 VCPU). There were **26,750** of such
instances.

## End ##

Statistics were generated from [Gnocchi](https://gnocchi.xyz/). Nectar logs the
start/end times of each instance in Gnocchi, as well as a host of other data. As
a Nectar user, you can use the Gnocchi API to access metrics for your resources.

Let me know if this has been interesting, or if there are any other stats you
want to see!

### Footnote ###

[^1]: VCPU time is (Number of VCPU) * (Running time). For example, if an
    instance has 2 VCPU and has been running for 1 hour, VCPU time is 2 hours.
