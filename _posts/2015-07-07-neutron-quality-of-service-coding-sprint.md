---
layout: post
title: Neutron Quality of Service coding sprint
date: '2015-07-07T15:24:53+02:00'
tags:
  - openstack
  - neutron
  - midcycle
  - qualityofservice
  - qos
  - ovs
  - sriov
tumblr_url: http://www.ajo.es/post/123458887419/neutron-quality-of-service-coding-sprint
---
Last week we had the Openstack Neutron Quality of
Service coding sprint in Ra''anana, Israel to work on [1].

It's been an amazing experience, we've acomplished a lot, but we
still have a lot ahead.We gathered together at Red Hat office for three days [2],
delivering almost (sigh!) the full stack for the QoS service
with bandwidth limiting.The first day we had a short meeting where we went over
the whole picture of blocks and dependencies that we had
to complete.

<figure><img src="/images/qos_coding_sprint1.jpg"/></figure>

The people from Huawei India (hi Vikram Choudhary & Ramanjaneya Reddy) helped us
remotely by bootstraping the DB models and the neutron client.

Eran Gampel (Huawei), Irena Berezovsky (Midokura) and Mike Kolesnik (Red Hat)
revised the API for REST consistency during the first day, provided an
amendment to the original spec [12], the API extension and
the service plugin [13] Concurrently John Schwarz (Red Hat) was working on the API tests
which acted as validation of the work they were doing.

<figure><img src="/images/qos_coding_sprint2.jpg"/></figure>

Ihar Hrachyshka (Red Hat) finished the DB models and submited the first neutron
versioned objects *ever* on top of the DB models, I
recomend reading those patches, they are like nirvana
of coding ;).

Mike Kolesnik plugged the missing callbacks for extending
networks and ports. Some of those, extending object reads
will be moved to a new neutron.callbacks interface.I mostly worked on coordination and writing some code
for the generic RPC callbacks [5] to be used with versioned objects,
where I had lots of help from Eran and Moshe Levi (Mellanox), the current
version is very basic, not supporting object updates but initial
retrieval of the resources, hence not a real callback ;) (yet!).

<figure><img src="/images/qos_coding_sprint3.jpg"/></figure>

Eran wrote a pluggable driver backend interface for the service,
[6] with a default rpc/messaging backend which fitted very nicely.

Gal Sagie (Huawei) and Moshe Levi worked at the agent level, Gal created
the QoS OvS library with the ability to manipulate queues, configure
the limits, and attach those queues to ports [7], Moshe leaded
the agent design, providing an interface for dynamic agent extensions [8],
a QoS agent extension interface [9], and the example for SRIOV [10],
Gal then coded the OvS QoS extension driver [11].

<figure><img src="/images/qos_coding_sprint4.jpg"/></figure>

During the last day, we tried to put all the pieces together, John
was debugging API->SVC->vo->DB (you'd be amazed if you saw him
going through vim or ipdb at high speed). Ihar was polishing the models
and versioned objects, Mike was polishing the callbacks, and I was
tying together the agent side. We were not able to fully assemble
a POC in the end, but we were able to interact with neutron client
to the server across all the layers. And the agent side was looking
good but I managed to destroy the environment I was using, so I will
be working on it next week.The plan aheadWe need to assemble the basic POC, make a checklist for missing tests and TODO(QoS), and start enforcing full testing for any other non-poc-essential patch.Doing it as I write: https://etherpad.openstack.org/p/neutron-qos-testing-gapsOnce that's done we may be ready to merge back during the end of liberty-2, or the very start of next one: liberty-3. Since QoS is designed as a separate service, most of the pieces won't be activated unless explicitly installed, which makes it very low risk of breaking anything for anyone not using QoS.

# What can be done better

Better coordination (in general), I'm not awesome at that,
but I guess I had the whole picture of the service, so that's
what I did.Better coordination with remotes: It's hard when you have a lot
of ongoing local discussions, and very limited time to sprint,
I'm looking forward to find formulas to enhance that part.

# Notes

In my opinion, the mid-cycle coding sprint was very positive, the ability to meet every day, do fast cross-reviews, and very quickly loop in specific people to specific topics was very productive.I guess remote coding sprints should be very productive too, as long as companies guarantee the ability of people to focus on the specific topic, said that, the face to face part is always very valuable.I was able to learn a lot from all the other participants on specific parts of neutron I wasn't fully aware of, and by building a service plugin we all got the understanding of a *fullstack* development, from API request, to database, messaging (or not), agents and how all fits together.

Special thanks Gary Kotton for joining us the first day to understand our plan, and help us later with reviews towards merging patches on the branch.To Livnat Peer, for organizing the event within Red Hat, and making sure we prioritized everything correctly.To Doug Wiegley and Kyle Mestery for helping us with rebases from master to the feature branch to cleanup gate bugs on time.

# References:

[1] http://specs.openstack.org/openstack/neutron-specs/specs/liberty/qos-api-extension.html#rest-api-impact

[2] https://www.dropbox.com/sh/0ixsqk4dz092ppv/AAAd2hVFP-vXErKacjAdc90La?dl=0

[3] Versioned objects 1/2: https://review.openstack.org/#/c/197047

[4] Versioned objects 2/2: https://review.openstack.org/#/c/197876/

[5] Generic RPC callbacks: https://review.openstack.org/#/c/190635/

[6] Pluggable driver backend: https://review.openstack.org/#/c/197631/

[7] OVS Low level (ovsdb): https://review.openstack.org/196373

[8] Agent extensions: https://review.openstack.org/#/c/195439/

[9] QoS agent extension : https://review.openstack.org/#/c/195440/

[10] SRIOV agent extension https://review.openstack.org/#/c/195441/

[11] OvS QoS extension: https://review.openstack.org/#/c/197557/

[12] API amendment: https://review.openstack.org/#/c/197004/

[13] SVC and extension amendment: https://review.openstack.org/#/c/197078/
