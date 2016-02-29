---
layout: post
title:  "Running Tempest for NeCTAR"
date:   2016-02-01 12:00:00
categories: NeCTAR
---
Over the past year, we (Melbourne node of NeCTAR) has pushed in 2 new batches of
compute hardware. One of the thing that bugs me was that the testing of compute
hardware was terribly inefficient - operators were manually creating new
instances, volumes, attaching volumes to instances, etc, to make sure that each
host is working before we move it to production. On top of it being a terrible
waste of time, we were also prone to missing out on different test cases (boot
from volume? oops! boot from _resized_ volume? oops!). This led us to look at a
better way to do testing when we started off with a hardware refresh this year
(+4000 VCPUs yay!).

Testing has already been long solved over at OpenStack - whenever a change is
merge in [gerrit](https://review.openstack.org/) test suites are kicked off.
This automated testing ensures that code is working before being merged, which
is awesome. With that in mind, I looked at how we can enable this for our
cloud.

# Difference in tempest tests

Before we being, there are some things to note:

1. OpenStack tempest tests are quite comprehensive - they test all API calls.
   However, from Melbourne node's POV, what we really want to test is the
correct working configuration on hosts/controllers. Hence, some filtering is
needed for to only run a subset of the tests we are interested in. Also, some
tests don't work for cells.

1. As an addition to the above, we also want to limit the tests to run on
   Melbourne node (or even 1 particular host), instead of relying on the
default scheduler to assign it to any host. This scenerio is useful as a final
QA before adding compute hosts to production aggregates.

1. Ideally, NeCTAR Core Services will run a set of high level API tests to make
   sure that the messages get to the correct cell. Each node like us will run a
different set of tempest tests for each host in the cell. This ensures that we
cover both the control plane and also each host.

# Set up

Here is a rough guide to our set up.

1. Clone [tempest](https://github.com/openstack/tempest) and
   [tempest-lib](https://github.com/openstack/tempest-lib)

1. Run pip install in each repo

        $ cd tempest-lib
        $ pip install .
        $ cd ../tempest
        $ pip install .

1. By default, tempest uses the default scheduler to launch instances/volumes.
   If you, like us, needs to launch on specific AZs, you might want the
following patches.

    * https://github.com/waipeng/tempest/commit/9b736de9d442ce69bd060b1a03c2483b35db3390
    * https://github.com/waipeng/tempest/commit/e55c8ec36d9a8f4ac96e77bfea1b843a257c6177
    * https://github.com/waipeng/tempest/commit/2000ff543845f59093ccd856be225091eac21301
    * https://github.com/waipeng/tempest/commit/372a989e7cc453a397125106c71963b8c464c991
    * https://github.com/waipeng/tempest-lib/commit/a83a541115a5ce82a9138eaa1d720bac9f33a5e7
    * https://github.com/waipeng/tempest-lib/commit/73002d9eea458014752c84164e07ece3cb5361d2

1. Set up the configuration file

        [auth]
        use_dynamic_credentials = false

        [compute]
        # test.tiny
        flavor_ref = '<flavor_id>'
        # ubuntu
        image_ref = '<image_id>'
        fixed_network_name = '<network>'
        availability_zone = '<AZ>'
        build_interval = 2
        volume_device_name = vdc

        [identity]
        auth_version = v2
        uri = '<keystone_url>'
        v2_admin_endpoint_type = 'publicURL'
        catalog_type = identity
        username = '<user>'
        password = '<password>'
        tenant_name = '<tenant>'

        [identity-feature-enabled]
        api_v3 = false

        [validation]
        run_validation=true
        network_for_ssh = '<network>'
        connect_method = 'fixed'
        image_ssh_user = 'ubuntu'

        [volume]
        availability_zone = '<AZ>
        build_interval = 2
        build_timeout = 60

1. If you don't want to run all the tests, you can create a whitelist file e.g.
   `etc/whitelist.txt`. An example of ours:

        # compute
        tempest.api.compute.servers.test_create_server.ServersTestJSON.test_host_name_is_same_as_server_name
        tempest.api.compute.servers.test_create_server.ServersTestJSON.test_verify_created_server_vcpus
        tempest.api.compute.servers.test_create_server.ServersTestJSON.test_list_servers_with_detail
        tempest.api.compute.servers.test_create_server.ServersTestJSON.test_verify_server_details

        # volume
        tempest.api.compute.volumes.test_attach_volume.AttachVolumeTestJSON.test_attach_detach_volume
        tempest.api.compute.volumes.test_attach_volume.AttachVolumeTestJSON.test_list_get_volume_attachments
        tempest.api.compute.volumes.test_volumes_get.VolumesGetTestJSON.test_volume_create_get_delete
        tempest.api.compute.volumes.test_volume_snapshots.VolumesSnapshotsTestJSON.test_volume_snapshot_create_get_list_delete

1. To run the whitelist tests, do

        ostestr --serial -w etc/whitelist.txt`

1. To run 1 test, you can do

        ostestr --serial --pdb tempest.api.compute.volumes.test_attach_volume.AttachVolumeTestJSON.test_list_get_volume_attachments`

NOTE TO SELF: Please tidy up the commits and put the configuration on Github!
