---
layout: post
title:  "The year in review"
date:   2019-12-19 00:00:00
categories: others
---

Things are winding down at the end of the year, so I thought it might be helpful
to myself to jot down some of what I did this year, and keep gauge if I am
growing professionally.

# Puppet catalog difference tests

This took up a lot of time, but I felt it was really necessary. We wanted to
greatly improve our testing to make sure we don't merge puppet code that breaks
the cloud.
[Many](https://status.cloud.google.com/incident/cloud-networking/19009)
[outages](https://blog.cloudflare.com/details-of-the-cloudflare-outage-on-july-2-2019/)
in major cloud providers last year were due to config changes, so preventing
this from happening to NeCTAR was a big priority.

Our new tests now generates a list of differences in catalogs for puppet nodes.
This greatly helps human reviewing the changes, as they can now see actual
resources and nodes being updated by each change.

Key things in this topic are:

1. Getting all sites' control repos under control of our CI/CD - so that Jenkins
   can trigger tests on changes
1. Wrangling >5 years of legacy puppet code into something that resembles modern
   day puppet best practices, and into a consistent format across sites - so
tests work across all sites.
1. Figuring out deprecation strategy for code/config - so we don't break
   existing code and yet allow us to move forward quicker

This is not totally done yet, but all the technical challenges have already been
resolved.

# CellsV1 to CellsV2

Huge effort by the whole team. The fact that we managed to pull it off without
downtime is impressive. Basically, in CellsV1, Core Services database holds all
the information about instances, but in CellsV2 the sites' databases holds this
information.

For my part, the bulk of the work involves making sure the CellsV1 and CellsV2
databases are consistent before the switch, and writing code to fix up any
inconsistencies. Boring manual work, which had to be done. This allowed us to
finally get rid of all the legacy CellsV1 patches!

# Rollout of Yubikeys

With the increase in attacks, I felt that it was time to beef up our security.
Fortunately, our manager was supportive and we manage to buy some YubiKeys. I
have been experimenting with integrating them into our systems. Right now we
have started using Yubikeys for:

1. Keystone credentials safe using `pass`
1. Shared passwords using the same
1. SSH forwarding
1. EYAML

# Unified user handling in Puppet

Our puppet code has grown over the years, so code to manage users (for different
systems) where in multiple places. Because of this, adding a new operator to the
cloud means making multiple changes in different repos. This is proving to be a
fair bit of technical debt, and is also a security issue when an operator
account wasn't removed cleanly in all places when they leave.

To solve this, I created a way to define users in just one place in Puppet. From
this, different systems which need to create users can just read it. Yes this is
the [universal standard](https://xkcd.com/927/), I promise.

# Security, security, security

1. Added eyaml support in Puppet
1. Moved rsyslog to using SSL; this will allow for centralised logging. Future
   work will let sites send their logs to us so we have a single pane of glass
for observing events. This can be useful, for example, in tracing a user's
request to boot an instance - we will be able to trace it all the way from API
(in Core Services) down to the compute node (at the site).
