---
layout: post
title:  "Puppet development in production"
date:   2015-12-13 12:00:00
categories: puppet
---
Over here at NeCTAR Research Cloud, we have a bunch of hosts,
mainly computes and a few other control plane hosts. When I started, there
wasn't a great puppet infrastructure set up - people were mostly still able to
edit on the production puppet master. Given that this host holds all the
configuration for NeCTAR RC, and one mistake could stop the whole cloud, it was
something we thought would be good to change from day one.

Because of our unique way of handling environments (this is another long
story), it was not possible to just simply create another environment and point
puppet clients to that for testing. In the end, we decided to create a seperate
server, named "_puppetdev_", which acts as a puppetmaster but allows us to
develop on it without crashing the production puppetmaster.

Note that this is __not__ a very comprehensive change management solution like
R10K, etc. This is a very simplistic solution we have put in to solve our
current issue quickly without making major changes to the environment; it can
always be improved.

# Setup

We have set up puppetdev as a _puppetmaster_ and _puppetdb_, but rely on the
production puppetmaster for CA. This means that if you have new hosts, you have
to point them to the production puppetmaster to be signed before pointing them
to puppetdev.

Below is an example of our puppet.conf

~~~
[main]
...
pluginsync=true
basemodulepath = $confdir/modules/production
environmentpath = $confdir/environments
ca = false
 
[master]
...
reports = puppetdb
storeconfigs = true
storeconfigs_backend = puppetdb
~~~

# Workflow

The workflow now goes like this:

1. Log into puppetdev and create your own environment. Each DevOps has his own
   environment, and you can create multiple environments. Environments are
   created by a custom script - the script just checks out all puppet
   _modules_, _hieradata_ and _manifests_ from our git repos.

1. Do your changes in the environment.

1. On the host you are testing, stop puppet to prevent a periodic puppet run
   from reverting all the changes you will make later.

    `service puppet stop`

1. Run puppet against _puppetdev_ by using:

   `puppet agent -t --environment <env> --server <puppetdev.domain.com> --noop`

    We are using the `--noop` flag for the first run so that you can check that
    only what you are changing is being applied.

1. Run puppet for real.

    `puppet agent -t --environment <env> --server <puppetdev.domain.com>`

1. If all goes well, commit your code, and send it up to
   [gerrit](https://review.rc.nectar.org.au/) for review.

1. Bug your colleagues to review the changes, and merge them.

1. Merge them into production and watch it get applied across all the hosts.

1. On the test host, flick it back to production puppet

    `puppet agent -t --server <puppetmaster.domain.com>`

1. Restart puppet. `service puppet start`
