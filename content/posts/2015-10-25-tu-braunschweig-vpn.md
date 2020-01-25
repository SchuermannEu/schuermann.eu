---
layout: post
title: TU Braunschweig - VPN on Debian without Cisco AnyConnect
author: Dominik Schürmann
date: 2015-10-25
slug: tu-braunschweig-vpn
aliases:
    - "/2015/10/25/tu-braunschweig-vpn.html"
---

My employer TU Braunschweig supports VPN connections via the [Cisco AnyConnect client](https://doku.rz.tu-bs.de/doku.php?id=netz:vpn:vpn_einrichten).
Unfortunately, the client is closed source and the installation on Debian stable fails with ``Failed to start vpnagentd.service: Unit vpnagentd.service failed to load: No such file or directory.``.
The following methods have been proven to work using open source packages from Debian.

## VPN over SSL
To tunnel all traffic over SSL, install AnyConnect support with ``apt-get network-manager-openconnect-gnome``, then add a new VPN connection to the Network Manager using "Cisco AnyConnect Compatible VPN (openconnect)".
Use the following configuration:

```
Gateway: vpngate.tu-bs.de
```
The connect to it and enter your credentials to authenticate.

## VPN over IPSec
To tunnel all traffic over IPSec, install vpnc support with ``apt-get network-manager-vpnc-gnome``, then add a new VPN connection to the Network Manager using "Cisco Compatible VPN (vpnc)".
Use the following configuration:

```
Gateway: vpngate.tu-bs.de
User name: $your-y-number
User password: $your-password
Group name: tubsipsec
Group password: tuipsec
```
