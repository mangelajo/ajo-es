---
layout: post
title: Neutron security_group_rules_for_devices RPC rewrite
date: '2014-08-20T11:14:00+02:00'
tags:
- neutron
- rpc
- security_group_rules_for_devices
- openstack
tumblr_url: http://www.ajo.es/post/95269040924/neutron-securitygrouprulesfordevices-rpc
---
We found during scalability tests, that the security_group_rules_for_devices RPC, which is transmitted from neutron-server to the neutron L2 agents during port changes, grew exponentially.

So we filled a spec for juno-3, the effort leaded by shihanzhang and me can be tracked here:

* https://review.openstack.org/#/c/111876/
* https://review.openstack.org/#/c/115575/

I have written a test and a little -dirty- benchmark (https://review.openstack.org/#/c/115575/1/neutron/tests/unit/test_security_groups_rpc.py line 418) to check the results and make sure the new RPC actually performs better.

# Here are the results:

Message size (Y) vs. number of ports (X) graph:

<figure><img src="/images/security_groups_rpc_0.png"/></figure>

RPC execution time in seconds (Y) vs. number of ports (X):

<figure><img src="/images/security_groups_rpc_1.png"/></figure>
