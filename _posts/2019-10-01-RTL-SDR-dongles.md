---
layout: post
title: RTL-SDR dongles
---

In a [previous post](https://danspanner.github.io/2019/09/16/Stuff-to-do-with-a-raspberry-pi.html) I mentioned that something nifty to do with my Raspberry Pi boards could be SDR scanning. So, I thought I'd take a dive into the world of RTL-SDR dongles.

I bought a cheap USB DVB-T stick from [Ozhack](https://ozhack.com/products/rtl-sdr-r820t2-rtl2832u) and plugged it in. The package shipped with a tiny whip antenna with an MCX connector and magnetic base, which I also plugged in. 

![dongle](/img/dongle.jpg)

I did a quick google around to see what people were using for listening in to traffic, and it seems [gqrx](http://gqrx.dk/) is pretty popular on the linux side, and [SDR-sharp](https://airspy.com/download/) is the go-to for Windows. After an `apt-get install gqrx` I was up and running (although I did need a reboot before it would see the dongle for some reason).

Within minutes I had my headphones in and could hear FM transmissions. Well that was simple! Surprised at my own success, I decided to go a bit deeper to see what else the dongle was capable of.

I found a package called `dump1090-mutability` and installed it. I was admiring the lovely screenshots people had posted on various forums, fired up my browser...

Nothing. I doublechecked my install- it was all fine. Configuration existed for it. The binary existed. But no web service. Everything I had read implied the package would spin up its own webserver and start pumping data to it. However localhost was sadly unreachable.

I scratched my head for a while, checked `netstat -plant` and `ps auxf` but still nothing.

Then I noticed something strange- port 80 was already spoken for. There was something listening on it. Also, according to `ps auxf` lighttpd was running.

So I started poking around in its configuration, and realised there was a dump1090 site enabled.

And that's when I tried looking at the [documentation](https://salsa.debian.org/debian-hamradio-team/dump1090-mutability).

Pro tip: start with reading the documentation rather than the hearsay from old forum posts (or as the Simpsons put it- Don't do what Donny Don't does). As it turns out, I could reach the service on localhost/dump1090.

![dump1090](img/dump1090.png)

Now I have a webserver running, and all is well with the world. But, if I can see planes, does that mean I can listen to the pilot chatter too? Apparently so!

I found a [youtube video](https://www.youtube.com/watch?v=9QzklSyKqQM&) that yes, I can do precisely that, so I set about scanning through some frequencies. After ten or fifteen minutes of static, I was less than impressed.

![gqrx](/img/gqrx.png)

A bigger antenna would probably help, the little whip that came with the dongle is certainly portable but not exactly useful. So my next post will (hopefully) centre around what I can see in the spectrum with a decent antenna. I'm of two minds to either purchase sometehing like a big whip antenna, or build my own out of a long length of wire (probably the latter, purely for financial reasons).
