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
You can create a cluster using either the Dashboard or CLI tools.

## Using Dashboard ##
1. Log on to the [BETA dashboard](https://dashboard-next.rc.nectar.org.au)

1. Navigate to **Container Infra**.

1. Click on **Clusters**, then **Create Cluster**.

1. Give your cluster a name.

1. Choose a cluster template. We have pre-defined global templates (in format
   *kubernetes-{az}*) to help you get started easier. Choose the template that you
want your cluster to be in.

1. Go to the **Misc** tab, and select your **Keypair**.

1. Click **Submit**.

## Using CLI ##
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

# Operating your Cluster using CLI #

Once your cluster is up (NOTE: It takes about 20 mins for a cluster to build),
you can control it using *kubectl*.

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

1. You can now use `kubectl` to run your images. If you are familiar with
   `docker`, this [kubernetes
document](https://kubernetes.io/docs/reference/kubectl/docker-cli-to-kubectl/)
lists the equivalent commands in `kubectl`.

# Operating your Cluster using web interface #

Alternatively, you can also administer it from the web.

1. Set up a role for the service account
   ```
   kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
   ```

1. List the secrets
   ```
   kubectl -n kube-system get secret
   ```

1. Get the secret token. It will be in format `kubernetes-dashboard-token-XXXXX`
   ```
   kubectl -n kube-system describe secret kubernetes-dashboard-token-XXXXX
   ```

1. Copy the token

1. Start the web interface
   ```
   kubectl proxy
   ```

1. In your browser, go to the following URL
   ```
   http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/login
   ```

1. Use your token to log in

Please send feedback!

# Tips #

### Availability Zone ###

You can boot in a different availability zone by using `--labels`. E.g.
```
openstack coe cluster create --cluster-template mytemplate \
--keypair <mykey> --labels availability_zone <AZ> mycluster
```
