---
layout: post
title: Neutron external network with routing (no NAT)
date: '2018-01-10'
tags:
- openstack
- neutron
---
In this blog post I will explain how to connect private tenant networks to
an external network without the use of NAT or DNAT (floating ips) via
a neutron router.

This can be done, with just one limitation, tenant networks connected in
such topology cannot have overlapping ips. And also, the upstream router(s)
must be informed about the internal networks so they can add router themselves
to those networks. That can be done manually, or automatically by periodically
talking to the openstack API (checking the router interfaces, the subnets etc..)
but I'll skip that part in this blog post.

We're going to asume that our external provider network is "public", with the
subnet "public_subnet" and that the CIDR of such network is 192.168.1.0/24 with
a gateway on *192.168.1.1*. Please excuse me because I'm not using the new
openstack commands, I can make an updated post later if somebody is interested
in that.

# Step by step

Create the virtual router

```bash
source keystonerc_admin

neutron router-create router_test

# please note that you can manipulate static routes on that router if you need:
# neutron router-update --help
#  [--route destination=CIDR,nexthop=IP_ADDR | --no-routes]
```

Create the private network for our test, and add it to the router

```bash
neutron net-create private_test
$PRIV_NET=$(neutron net-list | awk ' /private_test/ { print $2 }')

neutron subnet-create --name private_subnet $PRIV_NET 10.222.0.0/16
neutron router-interface-add router_test private_subnet
```

Create a security group, and add an instance

```bash
neutron security-group-create test
neutron security-group-rule-create test --direction ingress

nova boot --flavor m1.tiny --image cirros --nic net-id=$PRIV_NET
--security-group test cirros
sleep 10
nova console-log cirros
```

Please note that in the console-log you may find that everything went well...
and that the instance has an address through DHCP.

Create a port for the router on the external network, do it manually
so we can specify the exact address we want.

```bash
SUBNET_ID=$(neutron subnet-list | awk ' /public_subnet/ { print $2 }')
neutron port-create public --fixed-ip \
    subnet_id=$SUBNET_ID,ip_address=192.168.1.102 ext-router-port
```

Add it to the router

```bash
PORT_ID=$(neutron port-show ext-router-port | awk '/ id / { print $4 } ')
neutron router-interface-add router_test port=$PORT_ID
````

We can test it locally (assuming our host has access to 192.168.1.102)

```bash
# test local...
ip route add 10.222.0.0/16 via 192.168.1.102

# and assuming that our instance was created with 10.222.0.3

[root@server ~(keystone_admin)]# ping 10.222.0.3
PING 10.222.0.3 (10.222.0.3) 56(84) bytes of data.
64 bytes from 10.222.0.3: icmp_seq=1 ttl=63 time=0.621 ms
64 bytes from 10.222.0.3: icmp_seq=2 ttl=63 time=0.298 ms
```

I hope you enjoyed it, and that this is helpful, let me know in the comments :)
