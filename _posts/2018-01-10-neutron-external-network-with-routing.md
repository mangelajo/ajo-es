---
layout: post
title: Neutron external network with routing (no NAT)
date: '2018-01-10'
tags:
- openstack
- neutron
---

TO-DO, fix this blog post to explain "--disable-snat" that does the
same, and probably also supports FIP (I need to check)

```bash
usage: neutron router-gateway-set [-h] [--request-format {json}]
                                  [--disable-snat]
                                  [--fixed-ip
subnet_id=SUBNET,ip_address=IP_ADDR]
                                  ROUTER EXTERNAL-NETWORK

Set the external network gateway for a router.

positional arguments:
  ROUTER                ID or name of the router.
  EXTERNAL-NETWORK      ID or name of the external network for the gateway.

optional arguments:
  -h, --help            show this help message and exit
  --request-format {json}
                        DEPRECATED! Only JSON request format is supported.
  --disable-snat        Disable source NAT on the router gateway.
  --fixed-ip subnet_id=SUBNET,ip_address=IP_ADDR
                        Desired IP and/or subnet on external network:
                        subnet_id=<name_or_id>,ip_address=<ip>. You can
                        specify both of subnet_id and ip_address or specify
                        one of them as well. You can repeat this option.
```

...

In this blog post I will explain how to connect private tenant networks to
an external network without the use of NAT or DNAT (floating ips) via
a neutron router.

> With the following configuration you will have routers that don't do NAT
> on ingress or egress connections. You won't be able to use floating ips
> too, at least in the explained configuration, you could add a second external
> network and a gateway port, plus some static routes to let the router steer
> traffic over each network.

This can be done, with just one limitation, tenant networks connected in
such topology cannot have overlapping ips. And also, the upstream router(s)
must be informed about the internal networks so they can add routes themselves
to those networks. That can be done manually, or automatically by periodically
talking to the openstack API (checking the router interfaces, the subnets etc..)
but I'll skip that part in this blog post.

> We're going to assume that our external provider network is "public", with the
> subnet "public_subnet" and that the CIDR of such network is 192.168.1.0/24 with
> a gateway on *192.168.1.1*. Please excuse me because I'm not using the new
> openstack commands, I can make an updated post later if somebody is interested
> in that.

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

nova boot --flavor m1.tiny --image cirros --nic net-id=$PRIV_NET \
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

Extra note, if you want to avoid issues with overlapping tenant network IPs, I
recommend you to have a look at the [subnet pool functionality of
neutron](https://docs.openstack.org/mitaka/networking-guide/config-subnet-pools.html)
which started in Kilo.

I hope you enjoyed it, and that this is helpful, let me know in the comments :)
