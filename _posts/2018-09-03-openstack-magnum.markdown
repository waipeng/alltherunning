---
layout: post
title:  OpenStack Magnum available in Nectar
categories: nectar openstack
---

We've just deploy OpenStack Magnum (Container Infrastructure as a Service) on
Nectar cloud. We are in the process of coming out with official documentation,
but in the meantime if you would like to test drive it, here are the steps to do
so.

First of all, you need the following

1. Quotas for
   - Floating IP
   - Network
   - Subnet

   If you have requested for floating IPs for your project, you will be fine. If
not, [request for floating ip
quota](https://support.ehelp.edu.au/solution/articles/6000170753-private-networks#requesting_quota).

1. Install python-magnumclient
   ```
   pip install python-magnumclient
   ```

1. Create a template
   ```
   openstack coe cluster template create --coe kubernetes \
   --image fedora-atomic-latest --external-network <floating-ip-network-id> \
   --master-flavor m2.xsmall --flavor m2.small --dns-nameserver 172.26.21.141 \
   --docker-storage-driver overlay --public mytemplate
   ```

1. Boot a cluster
   ```
   openstack coe cluster create --cluster-template mytemplate \
   --keypair <mykey> mycluster
   ```

Please send feedback!

# Tips #

### Availability Zone ###

You can boot in an availability zone by doing the following
```
openstack coe cluster create --cluster-template mytemplate \
--keypair <mykey> --labels availability-zone <AZ> mycluster
```
