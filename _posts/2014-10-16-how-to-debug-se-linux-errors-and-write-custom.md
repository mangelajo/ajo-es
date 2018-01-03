---
layout: post
title: How to debug SE linux errors, and write custom policies
date: '2014-10-16T10:30:00+02:00'
tags:
- openstack
- selinux
tumblr_url: http://www.ajo.es/post/100147773734/how-to-debug-se-linux-errors-and-write-custom
---
Sometimes, you find yourself trying to debug a problem with SE linux, specially during software development, or packaging new software features.
I have found this with neutron agents to happen quite often, as new system interactions are developed.
Disabling selinux during development is generally a bad idea, because you’ll discover such problems later in time and under higher pressure (release deadlines).
Here we show a recipe, from Kashyap Chamarthy, to find out what rules are missing, and generate a possible SELinux policy:

Make sure selinux is enabled

```bash
sudo su -
setenforce 1
```

Clear your audit log, and supposing the problem was in neutron-dhcp-agent,
restart it.

```bash
 > /var/log/audit/audit.log
systemctl restart neutron-dhcp-agent
```

Wait for the problem to be reproduced..

Find what you got, and create a reference policy

```bash
cat /var/log/audit/audit.log
cat /var/log/audit/audit.log | audit2allow -R
```


At that point, report a bug so you get those policies incorporated in advance.
Give a good description of what's blocked by the policies, and why does it need to be unblocked.
Now you can generate a policy, and install it locally:

You can generate a SELinux loadable module to move on without
disabling the whole SELinux:

```bash
cat /var/log/audit/audit.log | audit2allow -a -M neutron
```

And you can also install it in runtime

```bash
semodule -i neutron.pp
```

Restart neutron-dhcp-agent (or re-trigger the problem to make sure it's fixed)

```bash
systemctl restart neutron-dhcp-agent
```

