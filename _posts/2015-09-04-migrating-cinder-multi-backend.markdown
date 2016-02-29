---
layout: post
title:  Migrating Cinder to multiple backend
categories: NeCTAR
---

OpenStack cinder has the ability to do multiple backends, which is _quite_
useful if you are running out of space on one type of storage and you need to
put in additional/replacement storage.

It is relatively simple to migrate to multiple backends, just enclose your
current backend under a section (e.g. `[netapp]`) and add in the
`enabled_backends` options.

Before:
    
    netapp_login = <login>
    netapp_password = <password>
    volume_driver = cinder.volume.drivers.netapp.common.NetAppDriver
    ...

After:

    enabled_backends = netapp
    [netapp]
    netapp_login = <login>
    netapp_password = <password>
    volume_driver = cinder.volume.drivers.netapp.common.NetAppDriver
    ...

# Gotcha

One important thing to note is that with multiple backends, the `host` column
in cinder DB will have the backend appended. You need to run the
following on your cinder controller:

    cinder-manage volume update_host --currenthost cinder-qh2-test --newhost 'cinder-qh2-test@netapp'

This updates the `host` colume of existing entries so that existing volumes can
still be managed under the new format.
