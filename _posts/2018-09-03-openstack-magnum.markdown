---
layout: post
title: Kubernetes now available on Nectar!
categories: nectar openstack
---

We've just deploy OpenStack Magnum (Container Infrastructure as a Service) on
Nectar cloud. This allows a user to spin up a container cluster (kubernetes or
docker swarm) on Nectar.

We are in the process of coming out with official documentation,
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


# Creating a Cluster #

1. Install python-magnumclient. You need python-magnumclient >= 2.9.0
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

# Operating your Cluster #

1. [Install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

1. Set up the credentials to connect to the cluster. Firstly, create an config
   dir
   ```
   mkdir ~/kubernetes/
   cd ~/kubernetes/
   ```

1. Create the config files
   ```
   openstack coe cluster config mycluster
   ```

1. Set the ENV by copying the output from the previous command
   ```
   export KUBECONFIG=$HOME/kubernetes/config
   ```

1. Use kubectl to connect to it
   ```
   kubectl get all
   ```


Please send feedback!

# Tips #

### Availability Zone ###

You can boot in an availability zone by using `--labels`. E.g.
```
openstack coe cluster create --cluster-template mytemplate \
--keypair <mykey> --labels availability_zone <AZ> mycluster
```
