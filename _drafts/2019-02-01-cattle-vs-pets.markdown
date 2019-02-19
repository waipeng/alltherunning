---
layout: post
title:  Cattle vs Pets, Ansible vs Puppet
tags:
- thoughts
---

### Puppet in Nectar ###

In Nectar, most things are provisioned with Puppet. This has served us well over
the years, setting up and managing VMs for the control plane that the cloud runs
on.

Recently, there has been an increased interest in container technology. It seems
that in many cases, the provisioning tool of choice is Ansible instead of
Puppet, e.g. [Kolla Ansible](https://docs.openstack.org/kolla-ansible/latest/).
This got me thinking why, and maybe this has to do with the idea of "Cattle not
Pets". 

In the DevOps world, there has been an idea that your infrastructure should be
"Cattle, not Pets". This means that if something breaks, you shoot it and spawn
a new one, instead of nursing that back to health like you would do for a pet.

### Cattle vs pets ###

Puppet is a configuration management tool. It keeps state; you define a desired
state a machine should have and Puppet knows the ways to transit to it. But why
is config management needed, if we plan to treat infra like cattle? We don't
care about keeping state. If configuration has to change, just kill the current
running instance and start up a new one.

To me still treat infrastructure like pets. You need to sign your nodes - which
means you can't just swap in a new running instance easily. Maybe not like your
pet dog Lassie, and more like your 3 pet goldfish in the tank. When one of them
dies you might miss it. And you still have to clean it up and get a new one from
the shop.

Ultimately, I guess the question could be - Is user intervention needed when it
dies? With this insight we are going to think about how to architect Nectar for
the future.
