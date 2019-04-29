---
layout: post
categories: openstack kubernetes
---

In [Kubernetes Part I]({% post_url 2018-09-03-openstack-magnum %}), we've discussd how to spin up a kubernetes cluster easily
on Nectar. In this post, we will discuss how to host an application and access
it externally.

To being, you should already have a working cluster. If you do not, head back to
the previous post and follow the steps.

1. Check that you cluster is working
   ```
   kubectl get nodes
   ```

1. Start a container image. We use nginx as an example
   ```
   kubectl run nginx --image nginx
   ```
   This command will start a *pod* with a *container* inside it running the
nginx image.  On Kubernetes, the smallest runnable unit is a *pod*, which holds
one (or more) *containers*.  

1. Check that your pod has started up and is running.
   ```
   kubectl get pods
   ```

1. Now that you have a pod working, we need a way of getting to it from the
   Internet. In Nectar Cloud, we can do this by creating a load balancer. A
load balancer has a public (floating ip), and redirects traffic to this public
IP to one or more private addresses. Use the following yaml to create your load
balancer. Save it as `nginxservice.yaml`.

   ```
   apiVersion: v1
   kind: Service
   metadata:
     name: nginxservice
     labels:
       app: nginx
     annotations:
       loadbalancer.openstack.org/floating-network-id: 'e48bdd06-cc3e-46e1-b7ea-64af43c74ef8'
   spec:
     ports:
     - port: 80
       targetPort: 80
       protocol: TCP
     selector:
       run: nginx
     type: LoadBalancer
   ```
   Note that the uuid in the `loadbalancer.openstack.org/floating-network-id`
refers to a network in `melbourne`. If your cluster is in a different AZ, you
might want to choose a floating IP network closer to where your cluster is for
routing efficiency. However, without it, things still work though! That's the
beauty of Nectar Advanced Network - no matter which AZ the traffic ingress
from, it still is able to make the way to your VM on Nectar Cloud.

1. Run it as
   ```
   kubectl create -f nginxservice.yaml
   ```

1. Get the public IP of the load balancer
   ```
   kubectl get services
   ```
   
1. You should be able to browse to `http://<ip>` and see the nginx welcome page.

1. This is an external load balancer (external to kubernetes), and is created in
   Neutron. You can see the loadbalancer in Neutron by doing
   ```
   neutron lbaas-loadbalancer-list
   ```

More details on what we have just did.

1. We started an external `LoadBalancer` service in Kubernetes. 

1. Kubernetes understands that it has to create this loadbalancer (externally)
   by calling out to the openstack neutron provider.

1. The `cloud-provider-openstack` plugin in kubernetes then create the different
   pieces that makes it all work, namely floating ip, load balancer, pool,
listener and members. These are all openstack resources. It mirrors this to the
`LoadBalancer` service you see in kubernetes when you do a `kubectl get
services`.

1. The plugin configs all of them and get the floating IP to be displayed in
   `kubectl get services`
