---
layout: post
title: How to split a git commit
date: '2015-06-19T16:12:43+02:00'
tags:
- openstack git patch split
tumblr_url: http://www.ajo.es/post/121920518309/how-to-split-a-git-commit
---
Sometimes you write a piece of code within a context, and such context grows wider and wider, or you simple need all the pieces in one place to make sure it works.

Then, for reviewing, or to work in parallel, it makes sense to split your patch in more logical patchlets. I always need to ask google. So let’s write it down here:



Let's assume $COMMIT is the commit you want to split (set the commit for edit with the edit action):

```bash
git rebase -i $COMMIT^
```


And this will leave your commit changes in the working tree, but you will be back in the previous commit.

```bash
git reset HEAD^
```

loop:

```bash
git add -p # the pieces of code you want to
git commit
git rebase --continue
```

If you were working with gerrit, make sure that only one of your patches (probably the biggest one) keeps the original change ID, so the change can still be tracked, and old comments will be available.

<figure>
<img src="/images/banana_split.png" alt="banana split git joke"/>
</figure>
(image credits go to: http://www.nicartoons.com/wallpapers/?id=1)

More interesting git stuff (fixup and autosquash): http://fle.github.io/git-tip-keep-your-branch-clean-with-fixup-and-autosquash.html (thanks to Jakub Libosvar!)
