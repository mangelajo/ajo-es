---
layout: post
title: Simplest openstack with networking-ovn deployment
date: '2018-04-20'
tags:
- openstack
---

One of the simplest and less memory hungry ways to deploy a tiny
networking-ovn all in one it's still to use packstack.


<b>Note</b> This is an unsupported way of deployment (for the product version), but it
must be just fine to give it a try. Afterwards, if you want to get serious and try
something closer to production, please have a look af the <b>[TripleO deployment guide](https://docs.openstack.org/networking-ovn/latest/install/tripleo.html)</b>

On a fresh Centos 7, login and write:

```bash
      sudo yum install -y "*-queens"
      sudo yum update -y
      sudo yum install -y openstack-packstack
      sudo yum install -y python-setuptools
      sudo packstack --allinone \
          --cinder-volume-name="aVolume" \
          --debug \
          --service-workers=2 \
          --default-password="packstack" \
          --os-aodh-install=n \
          --os-ceilometer-install=n \
          --os-swift-install=n \
          --os-manila-install=n \
          --os-horizon-ssl=y \
          --amqp-enable-ssl=y \
          --glance-backend=file \
          --os-neutron-l2-agent=ovn \
          --os-neutron-ml2-type-drivers="geneve,flat" \
          --os-neutron-ml2-tenant-network-types="geneve" \
          --provision-demo=y \
          --provision-tempest=y \
          --run-tempest=y \
          --run-tempest-tests="smoke dashboard"
```

When that has finished, you will have as root user, a set of keystonerc_admin,
and keystonerc_demo files. An example public and private network, a router, and
a cirros image.

If you want to see how objects are mapped into OVN, give it a try:

```bash

$ sudo ovn-nbctl show
switch 3b31aaa0-eea0-462e-9cdc-866f2bd8171d (neutron-097345d1-3299-43d4-aeda-8af03516b92e) (aka public)
    port provnet-097345d1-3299-43d4-aeda-8af03516b92e
        type: localnet
        addresses: ["unknown"]
    port 38eda4cc-4931-4c48-bc83-6e1d72ccb90b
        type: router
        router-port: lrp-38eda4cc-4931-4c48-bc83-6e1d72ccb90b
switch f6555e37-305f-4342-87f1-f21de5adadc2 (neutron-c3959f48-caa2-4eb9-b217-19e70c2380cb) (aka private)
    port ee519f75-08a8-4559-bf44-8acb9b2cec1b
        type: router
        router-port: lrp-ee519f75-08a8-4559-bf44-8acb9b2cec1b
router 20efb278-0394-4330-a18d-9e40a56b3ae5 (neutron-e70a02f3-6758-4f78-bc50-133b6c6b0584) (aka router1)
    port lrp-ee519f75-08a8-4559-bf44-8acb9b2cec1b
        mac: "fa:16:3e:f6:c6:d3"
        networks: ["10.0.0.1/24"]
    port lrp-38eda4cc-4931-4c48-bc83-6e1d72ccb90b
        mac: "fa:16:3e:5f:41:cf"
        networks: ["172.24.4.7/24"]
        gateway chassis: [7a42645d-70b3-4286-8ddc-6240ccdd131c]
    nat a5a6bec0-7167-41af-8514-2764502fef21
        external ip: "172.24.4.7"
        logical ip: "10.0.0.0/24"
        type: "snat"

```



