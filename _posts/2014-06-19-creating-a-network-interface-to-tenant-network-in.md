---
layout: post
title: Creating a network interface to tenant network in baremetal node (neutron)
date: '2014-06-19T02:18:00+02:00'
tags:
- openstack
- neutron
tumblr_url: http://www.ajo.es/post/89207996034/creating-a-network-interface-to-tenant-network-in
---
The next example illustrates how to create a test port on a bare-metal node connected to an specific tenant network, this can be useful for testing, or connecting specific bare-metal services to tenant networks.

The bare-metal node $HOST_ID needs to run the neutron-openvswitch-agent, which will wire the port into the right tag-id, tell neutron-server that our port is ACTIVE, and setup proper L2 connectivity (ext VLAN, tunnels, etc..)

Setup our host id and tenant network

```bash
NETWORK_NAME=private
NETWORK_ID=$(openstack network show -f value -c id $NETWORK_NAME)
HOST_ID=$(hostname)
```

Create the port in neutron


```bash
PORT_ID=$(neutron port-create --device-owner compute:container \
          --name test_interf0 $NETWORK_ID | awk '/ id / { print $4 }')

PORT_MAC=$(neutron port-show $PORT_ID -f value -c mac_address)
```

The port is not bound yet, so it will be in DOWN status


```bash
neutron port-show $PORT_ID -f value -c status
DOWN
```

Create the test_interf0 interface, wired to our new port

```bash
ovs-vsctl -- --may-exist add-port br-int test_interf0 \
  -- set Interface test_interf0 type=internal \
  -- set Interface test_interf0 external-ids:iface-status=active \
  -- set Interface test_interf0 \
  -- set Interface test_interf0 \
```

We can now see how neutron marked this port as ACTIVE


```bash
neutron port-show $PORT_ID -f value -c status
ACTIVE
```

Set MAC address and move the interface into a namespace
(namespace is important if you're using dhclient, otherwise the host-wide routes
and DNS configuration of the host would be changed, you can omit the netns if
you're setting the IP address manually)


```bash
ip link set dev test_interf0 address $PORT_MAC
ip netns add test-ns
ip link set test_interf0 netns test-ns
ip netns exec test-ns ip link set dev test_interf0 up

```
Get IP configuration via DHCP

```bash
ip netns exec test-ns dhclient -I test_interf0 --no-pid test_interf0 -v
Internet Systems Consortium DHCP Client 4.2.5
Copyright 2004-2013 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/test_interf0/fa:16:3e:6f:64:46
Sending on   LPF/test_interf0/fa:16:3e:6f:64:46
Sending on   Socket/fallback
DHCPREQUEST on test_interf0 to 255.255.255.255 port 67 (xid=0x5b6ddebc)
DHCPACK from 192.168.125.14 (xid=0x5b6ddebc)
```

Test connectivity (assuming we have DNS and a router for this subnet)

```bash
ip netns exec test-ns ping www.google.com
PING www.google.com (173.194.70.99) 56(84) bytes of data.
64 bytes from fa-in-f99.1e100.net (173.194.70.99): icmp_seq=1 ttl=36 time=115 ms
64 bytes from fa-in-f99.1e100.net (173.194.70.99): icmp_seq=2 ttl=36 time=114 ms
...
```

