---
layout: post
title: Neutron QoS service plugin
date: '2015-08-18T14:51:22+02:00'
tags:
- openstack
- qos
- neutron
tumblr_url: http://www.ajo.es/post/126667247769/neutron-qos-service-plugin
---
Finally, I've been able to record a video showing how the QoS service plugin works.If you want to deploy this follow the instructions under the video. (open in vimeo for better quality: https://vimeo.com/136295066)

<iframe
src="https://player.vimeo.com/video/136295066?title=0&amp;byline=0&amp;portrait=0"
title="Neutron QoS service plugin" width="500" height="313"
frameborder="0"></iframe>

Deployment instructions

Add to your devstack/local.conf

```bash
enable_plugin neutron git://git.openstack.org/openstack/neutron
enable_service q-qos
```

Let stack!

```bash
~/devstack/stack.sh
```

now create rules to allow traffic to the VM port 22 & ICMP

```bash
source ~/devstack/accrc/demo/demo

neutron security-group-rule-create  --direction ingress \
                                --port-range-min 22 \
                                --port-range-max 22 \
                                default

neutron security-group-rule-create --protocol icmp \
                                   --direction ingress \
                                   default

nova net-list
nova boot --image cirros-0.3.4-x86_64-uec --flavor m1.tiny \
          --nic net-id=*your-net-id* qos-cirros
#wait....

nova show qos-cirros  # look for the IP
neutron port-list # look for the IP and find your *port id*
```

In another console, run the packet pusher

```bash
ssh cirros@$THE_IP_ADDRESS \
     'dd if=/dev/zero  bs=1M count=1000000000'
```

In yet another console, look for the port and monitor it

```bash
# given a port id 49d4a680-4236-4d0c-9feb-8b4990ac35b9
# look for the ovs port:
$ sudo ovs-vsctl show | grep qvo49d4a680-42
       Port "qvo49d4a680-42"
           Interface "qvo49d4a680-42"
```

finally, try the QoS rules

```bash
source ~/devstack/accrc/admin/admin

neutron qos-policy-create bw-limiter
neutron qos-bandwidth-limit-rule-create *rule-id* bw-limiter \
                        --max-kbps 3000 --max-burst-kbps 300

# after next command, the port will quickly go down to 3Mbps
neutron port-update *your-port-id* --qos-policy bw-limiter
```

You can change rules in runtime, and ports will be updated

```bash
neutron qos-bandwidth-limit-rule-update *rule-id* bw-limiter \
                        --max-kbps 5000 --max-burst-kbps 500
```

Or you can remove the policy from the port, and traffic will
spike up fast to the original maximum.

```bash
neutron port-update *your-port-id* --no-qos-policy
```
