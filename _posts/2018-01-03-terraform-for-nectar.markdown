---
layout: post
title:  Terraform for Nectar
categories: nectar
---

If you are interested in using [Terraform](https://www.terraform.io/) and having
Nectar has one of your cloud providers, below is how you can get started.

1. [Install Terraform](https://www.terraform.io/intro/getting-started/install.html)

1. Create an `example.tf` in it's own directory, with the following config.
   Substitute in your credentials 

    ```
    # Configure the OpenStack Provider
    provider "openstack" {
      user_name   = "<user_name>"
      tenant_name = "<project_name>"
      password    = "<password>"
      auth_url    = "https://keystone.rc.nectar.org.au:5000/v3"
      domain_name = "Default"
    }

    # Create a web server
    resource "openstack_compute_instance_v2" "test-server" {
      name      = "<instance_name>"
      image_id  = "e4d127a9-458e-42a6-8401-2221e7fdc581" # Ubuntu 16.04
      flavor_id = "7da7cebe-68d1-4838-9c2a-c091c087bdbb" # m2.xsmall
      key_pair  = "<key_name>"
    }

    ```

1. Run the following

    ```
    $ terraform init
    $ terraform apply
    ```

1. You should see an instance being created like so.

    ```
    openstack_compute_instance_v2.test-server: Creating...
      access_ip_v4:        "" => "<computed>"
      access_ip_v6:        "" => "<computed>"
      all_metadata.%:      "" => "<computed>"
      availability_zone:   "" => "<computed>"
      flavor_id:           "" => "7da7cebe-68d1-4838-9c2a-c091c087bdbb"
      flavor_name:         "" => "<computed>"
      force_delete:        "" => "false"
      image_id:            "" => "e4d127a9-458e-42a6-8401-2221e7fdc581"
      image_name:          "" => "<computed>"
      key_pair:            "" => "jake"
      name:                "" => "jake_terrainstance"
      network.#:           "" => "<computed>"
      region:              "" => "<computed>"
      security_groups.#:   "" => "<computed>"
      stop_before_destroy: "" => "false"
    openstack_compute_instance_v2.test-server: Still creating... (10s elapsed)
    openstack_compute_instance_v2.test-server: Still creating... (20s elapsed)
    openstack_compute_instance_v2.test-server: Still creating... (30s elapsed)
    openstack_compute_instance_v2.test-server: Still creating... (40s elapsed)
    openstack_compute_instance_v2.test-server: Creation complete after 43s (ID:
    3f1799d3-0b62-4849-922b-b793eb398021)

    Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
    ```

1. That's it! Enjoy your new instance!

For more information, refer to:
- [Terraform Getting Started](https://www.terraform.io/intro/getting-started/build.html)
- [Terraform OpenStack Provider](https://www.terraform.io/docs/providers/openstack/)
