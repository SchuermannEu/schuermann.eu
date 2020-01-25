---
layout: post
title: Remapping Caps Lock to Windows (Super) key
author: Dominik Schürmann
date: 2011-04-22
slug: remap-caps-lock
aliases:
    - "/2011/04/22/remap-caps-lock.html"
---

I was using ``xmodmap ~/.Xmodmap`` in my **.Xsession** to remap caps lock to the super key utilizing the following configuration in **.Xmodmap**:

```
! disable caps lock and map to super
clear Lock
remove lock = Caps_Lock
add mod4 = Caps_Lock
```

After some updates in Debian testing, this configuration did not work anymore.
I found some information on [http://lists.debian.org/debian-x/2011/04/msg00291.html](http://lists.debian.org/debian-x/2011/04/msg00291.html).

After reading **/usr/share/X11/xkb/rules/base.lst**, a working way seems to be using ``setxkbmap`` with the option ``caps:super``:

```
setxkbmap -option caps:super
```

At the moment I am too lazy to read the documentation on XKB, so I am using xmodmap for the remaining keymappings. The right way should involve some configuration files documented on [http://madduck.net/docs/extending-xkb/](http://madduck.net/docs/extending-xkb/) and [http://www.charvolant.org/~doug/xkb/](http://www.charvolant.org/~doug/xkb/)

An introduction to the evolution of X.org can be found on [http://wiki.debian.org/XStrikeForce/InputHotplugGuide](http://wiki.debian.org/XStrikeForce/InputHotplugGuide)
