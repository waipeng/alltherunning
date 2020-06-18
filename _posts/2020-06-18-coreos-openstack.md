---
layout: post
title:  "Using CoreOS on OpenStack"
categories: openstack coreos
---

Most instances on the Nectar Cloud runs Linux (Ubuntu, CentOS). On Nectar's
Linux images, a provisioning tool call [cloud-init](https://coreos.com/ignition/docs/latest/) runs on first boot, which
inserts your SSH key and other user data into the instance. This allows you to
log in to your instance securely using SSH keys, and also run any scripts for
software installation when your instance first boot up.

CoreOS uses a different provisioning tool called
[Ignition](https://coreos.com/ignition/docs/latest/) instead of cloud-init. This
means that extra steps are necessary to boot up a CoreOS instance and inject
your SSH key.

# Short way (just SSH key)

1. If you do not know where your SSH public key is, you can get it from Nectar
   Dashboard, on the left menu under *Project* > *Compute* > *Key Pairs*. Or you
can use the CLI to get it
```
openstack keypair show --public-key <NAME>
```

1. Create the following file as `user-data.json`
```
{
  "ignition": {
    "version": "3.0.0"
  },
  "passwd": {
    "users": [
      {
        "name": "core",
        "sshAuthorizedKeys": [
          "ssh-rsa AAAAB3NzaC1c2EAA...dzP"
        ]
      }
    ]
  }
}
```

1. Boot up using CLI
```
openstack server boot --image fedora-coreos-31 --flavor m3.small \
--user-data user_data.json fedora-coreos-instance
```


# Long way

To build an Ignition configuration file, one has to create a YAML config and
use the FCOS Configuration Transpiler (FCCT) to convert it into JSON. See
[Fedora CoreOS pages for more
examples](https://docs.fedoraproject.org/en-US/fedora-coreos/producing-ign/).

FCCT is provided as a container, but to run it we need something like podman.
This is not installed on Ubuntu by default, so we need to install it.

1. Create a `user-data.yaml` like this. In this example we only set an SSH key,
   but this method is not limited to SSH keys.

        variant: fcos
        version: 1.0.0
        passwd:
          users:
            - name: core
              ssh_authorized_keys:
                - "ssh-rsa AAAAB3NzaC1c2EAA...dzP"

1. Install `podman` by following the [Ubuntu
   instructions on podman's
site](https://podman.io/getting-started/installation.html).

1. Run fcct
```
podman run -i --rm quay.io/coreos/fcct:release --pretty \
--strict < user-data.yaml > user-data.json
```

1. You should get the same `user-data.json` as the previous (short) example.
