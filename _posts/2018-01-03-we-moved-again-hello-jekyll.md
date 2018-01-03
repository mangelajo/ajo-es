---
layout: post
title: We moved again! hello jekyll, bye tumblr
date: '2018-01-03T10:28:53+01:00'
tags: []
---
After 5 years of using tumblr, and tumblr getting to mess with
all my bash and code scripts in undescriptible ways I have decided
to move into <a href="https://jekyllrb.com/">jekyll</a> based blogging.

The theme is a *very nice* MIT licensed theme (<a
  href="https://mademistakes.com/work/hpstr-jekyll-theme/">HPSTR</a>).

And you can find the source code to this blog here:
<a href="https://github.com/mangelajo/ajo-es/">https://github.com/mangelajo/ajo-es/</a>

I went back from hosted at tumblr.com to hosted *at home*, produly on an RDO
<a href="https://rdoproject.org">OpenStack</a> instance I have which also
serves other projects, the projects are managed in Ansible thank's to the
openstack ansible modules.

A front instance based on my pet project
<a href="https://github.com/mangelajo/nginx-master">nginx-master</a> takes care
of: reverse proxying each domain/project into it's VM/container/whatever,
DKIM/SMTP configuration on DNS records and IP address changes.

<figure>
<img src="/images/we_moved.png"/>
</figure>

