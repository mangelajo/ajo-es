---
layout: post
title: Cirros Image mirror
date: '2018-01-11'
tags:
- openstack
---

Every time I try to download a cirros image from
[cirros cloud](http://download.cirros-cloud.net/0.4.0/) for use in my
OpenStack devel environment it's super slow (100-200KB/s), so I'm making a local
mirror of it here.

You can grab the files from here:

* [cirros-0.4.0-x86_64-disk.img](https://ajo.es/binaries/cirros-0.4.0-x86_64-disk.img)
* [cirros-0.4.0-i386-disk.img](https://ajo.es/binaries/cirros-0.4.0-i386-disk.img)

Or into your cloud with:

```bash
source ~/keystonerc_admin
pushd /tmp
curl https://ajo.es/binaries/cirros-0.4.0-x86_64-disk.img  > cirros-0.4.0-x86_64-disk.img
openstack image create "cirros"   --file cirros-0.4.0-x86_64-disk.img \
                                  --disk-format qcow2 --container-format bare \
                                  --public
popd
```

Or for i386:

```bash
source ~/keystonerc_admin
pushd /tmp
curl https://ajo.es/binaries/cirros-0.4.0-i386-disk.img  > cirros-0.4.0-i386-disk.img
openstack image create "cirros"   --file cirros-0.4.0-i386-disk.img \
                                  --disk-format qcow2 --container-format bare \
                                  --public
popd
```

