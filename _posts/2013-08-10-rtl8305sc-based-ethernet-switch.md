---
layout: post
title: RTL8305SC based ethernet switch
date: '2013-08-10T14:38:00+02:00'
tags: []
tumblr_url: http://www.ajo.es/post/59860747489/rtl8305sc-based-ethernet-switch
---
A few years ago, I built a little embedded switch for one of my clients, they needed something tiny, with VLAN support, which coupled to their hardware and flexible enough for their needs.

They didn't need any management interface, as they used their own hardware to reconfigure the switch EEPROM.
After some search, we found a tiny jewel, impressive for just $3, the RTL8305SC from the Realtek family, it allowed up to 16 VLANs (more than enough for their application) and it also had nice features like loop detection and broadcast storm control.
VLAN switching was handled by a simple 16 entry table: VID + a bitmask to map which switch ethernet ports were connected to that VLAN.
In 802.1Q aware mode, ports could receive untagged and tagged frames, the untagged ones were connected to the switch port configured table index.

<figure> <img src="/images/RTL8305SC_01.png"/></figure>

For tagged ones, the table was searched, and portmask checked:

<figure> <img src="/images/RTL8305SC_02.png"/></figure>

Of course, before forwarding out any packet, the packet dst mac is checked in a 1024 entries lookup table, to check the dst mac is in the out port, or that the dst mac it's a broadcast address.
For inbound packets, the switch was able to pre filter in 4 different ways:

1. Do nothing, just pass the packets to the switching engine. (default)
2. Insert input port VLAN tags for non-tagged packets, and keep the tagged ones VID.
3. Remove VLAN tags from all packets.
4. Remove incoming VLAN tag, and add the port configured VLAN tag.

Talking about loop protection mechanisms:

<figure> <img src="/images/RTL8305SC_03.png"/></figure>

Although it didn't have loop removal capabilities (via STP protocol or friends).. the loop detection was enough for the project, it was rather simple: the switch sends a broadcast message every 3-5 minutes, with Ethertype 0x8899 (RRCP protocol), payload = 0x03,  with src mac = switch's mac. If the packet comes back into any port, with the same src macs then there is a loop detection signal triggered out (for a led or controller to detect).
The switch features weren't awesome, but it was quite a lot for the cost, and only 0.3 to 1.2W power consumption.
This switch chip has been replaced by the RTL8306SD, more powerful, and yet cheap.
