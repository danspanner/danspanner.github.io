---
layout: post
title: Stuff to with a Raspberry Pi
---

## Stuff to do with a Raspberry Pi

So I have a couple of Raspberry Pis. I'm not 100% certain why I have them, which is probably why they have been gathering dust on a shelf in my office for so long. There was some conversation in the SecTalks slack about mobile computing and the Rpi in general, and I felt guilty about leaving my little computers to languish so long. So I got to thinking- what could I use my Rpi for?

Then I read about the wonders of [Pi-hole](https://pi-hole.net/). Seems neat!

And then I realised this was available in [docker](https://github.com/pi-hole/docker-pi-hole).

Which lead to the shocking realisation that [docker was available on Rpi](https://www.raspberrypi.org/blog/docker-comes-to-raspberry-pi/).

I know, I know. It's been a while. But I haven't exactly been keeping up with the lilliputing scene at all. (I say this despite having two Raspberry Pis doing nothing...)

So I got Ubuntu written to a micro SD card, plugged my Rpi into the TV, and set myself up a user. I reserved an IP in my DHCP (which, for the moment, is running on the router) and ssh'd to it.

After and `apt-get update` and `apt-get upgrade` I finally spun up pi-hole.

`docker run -d --name pihole -p 53:53/tcp -p 53:53/udp -p 80:80 -p 443:443 -e TZ="Australia/Brisbane" -v "/data/docker-volumes/etc-pihole/:/etc/pihole/" -v "/data/docker-volumes/etc-dnsmasq.d/:/etc/dnsmasq.d/" --dns=127.0.0.1 --dns=1.1.1.1 --restart=unless-stopped pihole/pihole:latest`

I ran into some issues here though. Turns out, port 53 was spoken for. A little googling lead me to [this page](https://askubuntu.com/questions/907246/how-to-disable-systemd-resolved-in-ubuntu) which helpfully explained how to disable resolved in systemd. With that done, pi-hole was happy to act as a DNS server for me!

[pihole]: img/pihole.png

In fact, I liked it so much I added it to the router as my primary DNS. I have noticed since using it that from time to time, I get resolution failures on my phone after connecting, and oddly only on my phone (a Samsung Note 5 running Android 7).
 
So, what else can we do here? I have a whole Raspberry Pi to play with!

A few ideas I've had are:

- mysql server
- [zabbix monitoring](https://www.zabbix.com/)
- logging with greylog or [HELK](https://github.com/Cyb3rWard0g/HELK/wiki/Installation)

And for the second Pi, I'd love to have something mobile. I was thinking:

- [war walking for wifi](https://ozhack.com/products/awus036nha-802-11b-g-n-wireless-usb-adapter?variant=43245778375)
- [SDR scanning](https://ozhack.com/collections/software-defined-radio/products/rtl-sdr-r820t2-rtl2832u)

I'll have to source a decent battery for the Pi though, and I'm not sure how resilient it will be in a backpack. Hopefully my next Pi themed post will be about wacky wireless shenanigans, only time will tell.
