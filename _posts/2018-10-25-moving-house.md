---
title: Moving house
categories: tech future
---

We moved out of our flat at the start of the month. Our new place has an
additional room and therefore allows me to keep an office without denying my kid
the right to an own room. My new office is located between my kid's room and the
living room.

Our new landlord agreed to my plan of putting some cables into the walls of this
office space and so I went to work. We (my friend Glenn and I) drilled holes
into the walls, cut slits and put conduit in. Afterwards I installed several
Cat6a outlets in the office, the living room, the kid's room and the hallway so
I could use Gigabit speeds via wired connnections and put the wireless AP in the
hallway again.

So far, most connections have been working great but unfortunately it seems like
the hallway cable is broken and has to be replaced. After this is done I'll
probably update this post and maybe even put some pictures in.

The cables are terminated on a [patch
panel](https://www.amazon.de/gp/product/B0089V44UY/) on one side and
[outlets](https://www.hornbach.de/shop/Cat-6-Ethernet-Netzwerkdose-2-fach-Unterputz-35287/7954998/artikel.html)
on the other side. (Don't buy the cat6a outlets from Obi, they're robbing you
blind.)

I made sure to announce the move early to my [ISP](https://easybell.de) and the
switch has been smooth sailing from day one. They terminated the connection in
my old flat on the last day of September and when the tech showed up as ordered
on Oct 4th, everything worked just fine.

Well, everything except the transmission speed.

I'm currently on a 50 Mbps plan with Easybell and when I only got ~ 21 Mbps, I
opened a ticket with them to fix this. They have been nothing but helpful and
helped me troubleshoot as much as possible on their side. They even asked their
carrier (DTAG) to reset my DSLAM port to make sure it wasn't an issue outside my
flat.

Since this still didn't work out, I asked some friends from [THW
Stuttgart](https://thw-stuttgart.de) to help me debug my connection and we
managed to pinpoint the source of the problem: my Draytek modem. One firmware
upgrade later (now on
[3.8.4_modem_7](http://www.draytek.com/download_de/Vigor130/Vigor130_v3.8.4_modem_7%20-%20englisch.zip)),
according to [BelWue speedtest](https://speedtest.belwue.net), I now receive 50
Mbps and transmit 10 Mbps -- exactly what was agreed to.

If I ever need more speed, I can easily upgrade the connection to 100 Mbps down
and 40 Mbps up for additional 5 Euros per month.

(I just love this ISP and I wholeheartedly recommend them time and time again.
They're just as great as my [electricity provider](https://naturstrom.de),
another company where I've been a loyal customer for years and which I recommend
again and again. The move there was just as easy and painless.)

All in all I'm really happy about our new home and we hope to stay here for
quite some time. Maybe I'll even be able to blog somewhat more frequently but
currently I'm not holding my breath. My priorities aren't really shifting
anytime soon and I'm rather busy and probably will stay so for a while. Sorry
about that..
