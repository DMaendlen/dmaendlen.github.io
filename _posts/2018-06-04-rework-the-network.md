---
title: Reworking the network
categories: tech firewall experiment future
---

Due to some unforeseen circumstances, I was unable to proceed with my little
project as planned. Until last week, when I took my family on an out-of-state
trip to my parents.

My kid was cared for, my wife was able to relax despite her condition so I was
free to get some work done.

In this post I'll detail

 * my physical network (with diagram!)
 * my logical network (with diagram!)
 * the [Zyxel GS1900-10HP](https://www.amazon.de/Zyxel-8-Port-Gigabit-Managed-Switch/dp/B0189ZRSMK) setup (bought to replace the [Netgear GS108E](https://www.amazon.de/Netgear-GS108E-300PES-Managed-konfigurierbar-deutscher/dp/B00MYYTP3S))
 * the [Ubiquiti UAP Pro](https://www.amazon.de/Ubiquiti-UAP-AC-PRO-Networks-wei%C3%9F/dp/B016XYQ3WK) setup
 * the PFSense configuration
 * the Raspberry Pi 3 configuration

Some advice up front: While it seems like a daunting task, the project can be
broken into smaller pieces. If you do so, however, please ensure to document
each and every single step. Also, have a backup of just about everything, I
almost crippled the network several times and in retrospect should have had
better backups of critical stuff. Looking at you, Zyxel TLS certificates...

## Network design
During this project I got to play around with designing a network. I know at
least two of my readers are now shaking their head in disgust, mumbling about
this not being a real network design and/or this not even being a real network.

While I get their point, this project covered all essential basics. The WAF is
also considerably higher for a nice, working network setup for home use instead
of a server capable of hosting several VMs to achieve basically the same stuff.

So, I designed a network. The diagrams were done online with
[draw.io](https://draw.io) after I did some pen-and-paper-based raw designs.

The idea was to have one physical network with separate devices for

 * uplink (the Draytek modem)
 * firewall (the PFSense box)
 * switching (the Zyxel switch)
 * wireless access (the UAP AC Pro)

I need wired access for my backup server, an HP printer, two RPis and my work
notebook. Furthermore I have to connect the switch to the PFSense box and the
wireless access point (WAP) to the switch. That leaves me with one remaining
network port, currently used as the access port for management, that could be
used for further extension of the network. Although I'd probably just [buy a
bigger switch](https://www.youtube.com/watch?v=QT9BeGNnCqw).

<img src='images/phys.nrd.png' alt='physical network layout' class='inline'>

Why would I use one physical network? Well, because having redundant devices
would be a little bit overkill for a simple home network, wouldn't it?

The logical network, however, should have some separation of concerns to ensure
at least a tiny bit of privilege separation. Again, since this is a private home
network it's more about the concept in general and less about a specific threat
I wanted to rule out. Therefore I have some cross connections that shouldn't be
present in a corporate network. I'll probably have to rethink some settings as
well.

I grouped all management interfaces into VLAN 10, all "regular" surfing devices
into VLAN 20, the mobile devices into VLAN 30 and work-related devices
(currently two notebooks) into VLAN 40. The VLAN split is done at the access
port level on the switch for wired connections and by the RADIUS server for
wireless ones.

<img src='images/lgcl.nrd.png' alt='logical network layout' class='inline'>

## Setup

To configure my devices, I relied heavily on help from people walking that route
before me and I don't see any benefit in reiterating their work, so I'll just
link those posts. If you're one of those people I'd like to buy you a drink if
we ever meet in person.

### PFSense VLAN
The PFSEnse VLAN interfaces were configured by using [this
guide](https://www.highlnk.com/2014/06/configuring-vlans-on-pfsense/). It's
really easy to set up a working VLAN config if you think about what you want
first and follow the guide. Pro tip: Do *NOT* under any circumstances disable
the LAN interface when you're done setting up the VLANs. You still need that for
your AP and Switch to be able to communicate with the PFSense. I made that
mistake and had to configure a VLAN tag on my Gentoo box to communicate to the
PFSense via the switch's uplink and it was an all-around pain. Yes, I lerned
something from that but I wouldn't recommend that experience for a production
setup.

### Zyxel GS1900-10HP
The Zyxel configuration is a little less easy. For one, Zyxel uses a weird
terminology in their [user
guide](ftp://ftp.zyxel.com/GS1900-8/user_guide/GS1900-8_V2.1_Ed1.pdf) and I was
a bit confused what that meant so I took to google once more. [This blog
post](https://codebeta.com/vlans-and-the-zyxel-gs1900-16-6b1eecd0632?gi=777555a90c72)
helped a lot, so thank you for that, Nicole.

I also configured the switch to use custom TLS certificates by using [this blog
post](https://hansmi.ch/articles/zyxel-gs1900-tls-cert). Switching on Telnet and
getting a root shell via an OS command injection sounds about right. Since it's
not mentioned in the blog post, please make sure to have a backup of the
certificates and make sure that the private key of your new certificate does
*NOT* require a password. Otherwise you'll get a confusing error message about
something called 'no_cyphers_overlap' in firefox and will be searching for a
solution to a non-existant problem. Don't ask me how I know that...

### Unifi AC AP PRO
My AP automatically tags users' VLANs via RADIUS. So, dmaendlen_laptop
automatically connects to srf.nrd, dmaendlen_phone automatically connects to
mbl.nrd and so on. 

This is achieved by using a combination of Ubiquiti's Controller software to
configure my AP and a FreeRADIUS instance. Both run on the RPi3 in mgmt.nrd.

The controller configuration can be seen in [this german blog
post](https://blog.andreseck.de/2017/05/unifi-radius-authentifizierung-mit-vlan-zuordnung/)
which has enough pictures to actually make it useful for non-german-speaking
people as well. Keep in mind that Henning uses a RADIUS server on Ubiquiti gear
while I used a custom one since I only own one Ubiquiti device (the AP).

### RADIUS
Getting FreeRADIUS 3.0 to work was something else and cost me several days of
trial and error until I finally stumbled upon [Tony Maro's really helpful
post](https://www.ossramblings.com/RADIUS-3.X-Server-on-Ubuntu-14.04-for-WIFI-Auth)
which cleared my path to success.

Somewhat down at the end he mentions freeradius being unable to create a /tmp/
directory. I solved that by making use of tmpfiles.

```
echo 'd /tmp/radiusd 0700 freerad freerad' > /etc/tmpfiles.d/freeradius.conf
```

Now FreeRADIUS should start without trouble.

Currently my implementation does not check if the certificate actually matches
the user so any user can use any certificate from my CA to authenticate against
the RADIUS server. That's not very nice but I didn't manage to make that work in
3.0 and I had to stop by then because we were packing up.

### Connectivity
To connect my Gentoo box to the WiFi I created, I just set the connection up in
the wpa_supplicant.conf file and told wpa_cli to enable the connection. This was
straightforward and no big deal. Getting network-manager on my wife's box to
play nice was a bit more difficult and included several threats of migrating
that box from a systemd-polluted Debian to Gentoo as well.

Lineage 15.1 and Android 5.0 work out of the box as soon as you install the CA
certificate and the user certificate. I just pushed the certs onto our devices
using adb and took it from there.

## Going forward
This will probably not be the last time I tinker with my network. I'll probably
have to implement a kid's network with corresponding filters someday and I still
need to get my printer working again but right now I'm rather content with my
achievement. I actually learned a ton about VLANs in general and my gear's
implementations in particular and I also got to play with my expensive toys
again so that's nice.

I hope this post helps somebody and I'm open to questions if you have any. I'll
probably even edit the post if I get asked questions.
