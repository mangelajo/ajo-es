---
layout: post
title: 'Using Openstack neutron SDN in OVH or Hetzner commodity datacenters '
date: '2014-01-24T07:41:00+01:00'
tags:
- openstack
- neutron
- sdn
- ovh
- hetzner
tumblr_url: http://www.ajo.es/post/74363654373/using-openstack-neutron-sdn-in-ovh-or-hetzner
---
There are commodity data center operators out there like OVH or Hetzner, who provide you with single floating IPs or RIPE blocks directly routed to your machine.
That means that, your machine usually is  AAA.BBB.CCC.DDD/24 (the primary address), with the default router in AAA.BBB.CCC.254 , but this router also will handle traffic for WWX.XXX.YYY.ZZZ/29 or EEE.FFF.GGG.HHH/32 (floating RIPE blocks or addresses) to your network interface to your eth MAC directly, and you can
if you have a virtual machine inside your host, you need to setup routing like this:

ifconfig eth0 EEE.FFF.GGG.HHH netmask 255.255.255.248 up
route add AAA.BBB.CCC.254 dev eth0route add default via AAA.BBB.CCC.254 dev eth0

(if you use RHEL or CENTOS in an VM, you will need to use the route-eth0 script to setup this)
As an extra (at least for ovh) they provide an API to setup your .254 router connect your floating IPs to virtual mac adresses, but… you cannot chose the MAC address, they will provide you a random one.
When trying to use this together with openstack / neutron server, it won’t work out of the box.
Loïc Dachary describes a solution here Fragmented floating IP pools and multiple AS hack
His solution solves 2 problems: disperse floating IP pools + the MAC translation. But adds complexity (you have an extra layer of virtual IPs that you see within neutron, and then you have to manually keep a correlation table in the network node external IP:internal IP … etc)
Another solution

warning: this is not tested, but an idea for implementation

He sparked an idea in my mind about how to handle it without double IP translation (but I’m not fixing the multiple floating IP blocks problem):
My solution (and I yet have to try it) is adding another bridge in the middle (classic linux bridge) before the external eth…and then use MAC DNAT:
for every virtual MAC address in neutron:

ebtables -t nat -A PREROUTING -d 00:11:22:33:44:55  MAC)  -i eth0 -j dnat –to-destination 54:44:33:22:11:00 -ip-dst 172.16.1.1+ the output counterpart rule (TBD)
considering:00:11:22:33:44:55 : the external interface general MAC address54:44:33:22:11:00 : the internal, randomly generated neutron address
172.16.1.1 : the floating IP address connected to 54:44:33:22:11:00 in neutron

of course, we need a little tool that might be learning from neutron-server which virtual mac addresses are we using…

Yet another solution (sounds way better):
Adding openflow rules to br-ex (or an extra br-ovh bridge), to handle incoming and outgoing traffic and do this MAC translation based on openflow tables This would allow us to learn from any outgoing traffic the MAC DNAT counterpath in the input table.
See Table 10 & Table 20 for the openflow configuration in openvswitch/neutron in Assaf Muller’s blog: GRE Tunnels in OpenStack Neutron

Cheers,
Miguel Ángel Ajo
