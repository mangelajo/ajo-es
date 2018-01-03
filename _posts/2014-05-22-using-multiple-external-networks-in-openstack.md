---
layout: post
title: Using multiple external networks in OpenStack Neutron (reference implementation)
date: '2014-05-22T14:45:00+02:00'
tags: []
tumblr_url: http://www.ajo.es/post/86497974174/using-multiple-external-networks-in-openstack
---
Starting on Icehouse release, a single neutron network node using ML2+ovs or OVS, can handle several external networks.
I haven’t found a lot of documentation about it, but basically, here’s how to do it, assuming this:
you start from a single external network, which is connected to ‘br-ex'' 
you want to attach the new external network to ''eth1’.
In the network node (were neutron-l3-agent, neutron-dhcp-agent, etc.. run):
Create a second OVS bridge, which will provide connectivity to the new external network:

```bash
ovs-vsctl add-br br-eth1
ovs-vsctl add-port br-eth1 eth1
ip link set eth1 up
```

(Optionally) If you want to plug a virtual interface into this bridge and add a local IP on the node to this network for testing:

```bash
ovs-vsctl add-port br-eth1 vi1 -- set Interface vi1 type=internal
ip addr add 192.168.1.253/24 dev vi1
```

Edit your /etc/neutron/l3_agent.ini , and set/change:

```bash
gateway_external_network_id =
external_network_bridge =
```

This change tells the l3 agent that it must relay on the physnet<->bridge mappings at /etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini it will automatically patch those bridges and router interfaces around. For example, in tunneling mode, it will patch br-int to the external bridges, and set the external ''q''router interfaces on br-int.
Edit your /etc/neutron/plugins/openvswitch/ovs_neutron_plugin.ini to map ''logical physical nets’ to ''external bridges’

```bash
bridge_mappings = physnet1:br-ex,physnet2:br-eth1
```

Restart your neutron-l3-agent and your neutron-openvswitch-agent

```bash
service neutron-l3-agent restart
service neutron-openvswitch-agent restart
```

At this point, you can create two external networks (please note, if you don’t make the l3_agent.ini changes, the l3 agent will start complaining and will refuse to work)

```bash
neutron net-create ext_net --provider:network_type flat \
                           --provider:physical_network physnet1 \
                           --router:external=True

neutron net-create ext_net2 --provider:network_type flat \
                            --provider:physical_network physnet2 \
                            --router:external=True
```

And for example create a couple of internal subnets and routers:

```bash
# for the first external net
neutron subnet-create ext_net --gateway 172.16.0.1 172.16.0.0/24 \
                      --enable_dhcp=False

# here the allocation pool goes explicit. all the IPs available..
neutron router-create router1
neutron router-gateway-set router1 ext_net
neutron net-create privnet
neutron subnet-create privnet --gateway 192.168.123.1 192.168.123.0/24 \
                 --name privnet_subnet
neutron router-interface-add router1 privnet_subnet

# for the second external net
neutron subnet-create ext_net2 --allocation-pool start=192.168.1.200,end=192.168.1.222 \
         --gateway=192.168.1.1 --enable_dhcp=False 192.168.1.0/24
neutron router-create router2
neutron router-gateway-set router2 ext_net2
neutron net-create privnet2
neutron subnet-create privnet2 --gateway 192.168.125.1 192.168.125.0/24 --name privnet2_subnet
neutron router-interface-add router2 privnet2_subnet
```

