---
layout: post
title: KiCad Win32 full scripting support, just arrived
date: '2013-08-05T09:38:22+02:00'
tags: []
tumblr_url: http://www.ajo.es/post/57410394419/kicad-win32-full-scripting-support-just-arrived
---
Brian Sidebotham and and Dick Hollenbeck just did it!,
<figure>
<a href="http://kicad-pcb.org">
<img src="/images/kicadpython.png" />
</a>
</figure>

*Long story short:*
A few months ago we suffered, as soon as we discovered that the standard python binaries for win32, were MSVC built, and they had runtime discrepancies with some modules or executables built from mingw (wxpython support), that always ended execution with a segmentation fault.

At that time I was getting "a little" busy, but dick & brian kept working. They discovered that python has no support to build outside MSVC on windows(really, it doesn't), so dick started python-a-mingw-us project, which cmake-compiles python 2.7.4 and builds a set of win32 binary installers or python 2.7.4, fully mingw runtime compatible.
After that, Brian kept working on his kicad winbuilder, yesterday he announced that he got it working in full : wxpython, scripting, everything.
