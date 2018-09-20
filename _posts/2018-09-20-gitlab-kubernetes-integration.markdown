---
layout: post
title: GitLab and Kubernetes Integration
categories: nectar openstack
---

In the previous blog post we've described [how to spin up a kubernetes cluster on
Nectar]({% post_url 2018-09-03-openstack-magnum %}). Around the same time, I
also got to know that University of Melbourne has a [self-hosted
gitlab](https://gitlab.unimelb.edu.au/). To my delight, I found out that GitLab
has [Kubernetes integration](https://about.gitlab.com/kubernetes/).

This means that, if you are in UniMelb (or have a self-hosted gitlab), you can
run CI/CD using Nectar cloud, without having to set up any infrastructure!

# Spin up cluster #
1. Spin up a kubernetes (k8s) cluster and create the config directory.
   [Instructions]({% post_url 2018-09-03-openstack-magnum %}).

1. Run the following
```
kubectl create clusterrolebinding permissive-binding --clusterrole=cluster-admin
--user=admin --user=kubelet --group=system:serviceaccounts
```

1. Get the default secret name (in format `default-token-xxxx`)
```
kubectl get secrets
```

1. Get the token for this secret
```
kubectl describe secrets/default-token-xxxx
```

1. Get the API URL.
```
cat $KUBECONFIG
```
Look for the line like 
```
clusters:
- cluster:
    server: https://192.168.1.1:6443
```

1. Get the CA cert. In the directory where `$KUBECONFIG` is stored, do
```
cat ca.pem
```

# Configure Repo #
1. In the GitLab repo, navigate to **Operations** - **Kubernetes**

1. Fill in the cluster details that you got from the previous steps.

1. In the list of Applications, install the following in order:
   1. Helm Tiller
   1. GitLab Runner

1. Create a `.gitlab-ci.yml` file. For example, if I want to run yamllint on my
   code, an example file will look like:

    ```
    before_script:
      - apt-get update
      - apt-get install -y python yamllint
      - apt-get install -y python3-pkg-resources python3-setuptools
      - python --version
      - which python
    yaml-lint:
      script:
        - find . -type f -iname "*.yaml" -o -iname "*.eyaml" | xargs yamllint
    ```

1. Commit and push the change. When the change is pushed to GitLab, a runner
   will start up and run the job specified by `.gitlab-ci.yml`. Jobs can be
viewed from the **CI/CD** tab.

# Limitations #
At this point in time, GitLab CE integration can only do one kubernetes cluster
per repo. No ability to do dev/test/prod clusters per repo.

Also, it [does not support
RBAC](https://gitlab.com/gitlab-org/gitlab-ce/issues/29398), so it means that
the integration will have full permissions to the cluster. So you really want to
dedicate 1 k8s cluster to 1 repo, and not have any other containers running on
that cluster.
