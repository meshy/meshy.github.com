---
layout: post
title: "Ubuntu on a ReadyNAS Pro 6"
description: "Steps and mis-steps taken to install linux on my NAS."
---

A little while ago I was looking to build a home server, and was very grateful
to receive a [Netgear ReadyNAS Pro 6][readynas] as a gift. The software on
it was rather out of date, so rather than update it to the latest (but
unsupported on this hardware) Netgear OS 6, I decided to install Ubuntu Server.

I've installed Ubuntu on desktops and servers loads of times, so I'm pretty
familiar with using it in a normal setting, but this presented a couple of
unusual challenges.

Firstly, the box doesn't have anywhere to plug in a monitor. Now, it's not
needed once the NAS is up and running, but it _is_ important for the
installation. It does have a suspiciously-VGA-port-shaped plate on the back,
however...

[Jaroslaw Zachwieja's related blog post][replace-os-post] very helpfully
explains that there are some nvidia graphics card breakout cables that can be
installed to get this working. He lists them as "XFX part: MA-BK01-LP1K". It
proved very hard to find that item in stock anywhere. I finally found a listing
on ebay under the name "[VGA SVGA Video Graphics Adapter Card Slot Bracket
Header Cable 15 Pin 16 Hole][breakout-cable]". Catchy. I snapped one up.
I've just checked: now they're sold out too.

If you're trying to do this, and are unable to get hold of (or make) the
required cable, then you might have luck [using the serial port on the back
instead][serial-port].

With the VGA cable installed I was able to plug in a monitor, watch the default
OS boot, and do a little victory jig.

After that, a had some false starts. I discovered that:

- To boot to the live USB, one must hold down the "Backup" button while turning
  on the machine.
- The NAS is picky about USB sticks. A Kingston DataTraveller (100 G3) 8GB did
  not work, but my Sandisk Ultra Fit 128GB did.
- On boot, Ubuntu showed an error and a prompt:
    ```
    graphics initialization failed
    Error setting up gfxboot
    boot:
    ```
    The advice online was to [type `help` at this prompt][type-help], but that
    failed for me on Ubuntu 16.04. It turns out that [`gfxboot` has improved
    since 16.04 came out][gfxboot-bug], so 17.10 got me beyond this point,
    despite displaying the same error.
- One of the USB ports was faulty.

I had intended to install from one USB stick to another, using the internal
128MB drive as a boot partition. The faulty USB port meant I could only plug in
one USB stick and a keyboard.

I found out that I can [install to the USB I booted from][install-to-boot-usb]
if I boot with the `toram` boot option. The way to do this was to type `install
toram` at the `boot:` prompt _instead_ of typing `help`.

The standard Ubuntu Server image failed to format the USB it was running from
with this method, but it worked a treat on the [Ubuntu 17.10 Network
installer][ubuntu-network-installer]. (Confusingly, the file was called
`mini.iso`, unlike the other ubuntu downloads, which, on the whole, are named
descriptively.)

The installation went swimmingly from there on. I thought things might have
gone wrong a couple of times, but nothing turned out to be an issue. Notably:

- The installer seemed stuck at 20% of "Creating swap file", but it did
  eventually get moving again.
- My concern that [automatic updates would fill up the `/boot/` partition with
  old kernels][boot-old-kernels], was based on out of date information, so I
  was able to turn them on.
- I was asked what software I wished to install. I selected "OpenSSH Server"
  (which is required to SSH into the machine over the LAN) and "Basic Ubuntu
  Server" (which, in retrospect, [I probably didn't need][basic-ubuntu-server]).
- Installing Grub to the Master Boot Record didn't seem to cause the chaos I
  thought it might.
- Not that it's installed, nothing after the BIOS appears on the monitor. I
  just get a blank screen. I suspect that there is an incorrect video setting
  somwhere. That doesn't cause me any issues, however. I'm able to SSH in, and
  I do not plan on having a monitor plugged in.

So now I have Ubuntu installed! [Yatta!][hiro-nakamura]

I have a small list of things to do next:

- This thing is _loud_. I'd like to [pop some new fans in it][new-fans].
- I'll be installing [Pi Hole][pi-hole] and [NextCloud][nextcloud].
- There's a tiny screen on the front. I'd like to see if can get it to display
  anything.
- It's a NAS... Maybe I should put some hard drives in it...


[readynas]: https://www.netgear.com/support/product/RNDP6000-200_(ReadyNAS_Pro_6).aspx
[replace-os-post]: https://warwick.ac.uk/fac/sci/csc/people/computingstaff/jaroslaw_zachwieja/readynaspro-jailfix/
[gfxboot-bug]: https://bugzilla.suse.com/show_bug.cgi?id=980570
[breakout-cable]: https://www.ebay.co.uk/itm//261646911396
[serial-port]: https://nerdyness2012.wordpress.com/2015/03/31/installing-ubuntu-14-10-server-on-a-netgear-readynas-ultra-duo-v2/
[type-help]: https://askubuntu.com/a/417966/30904
[install-to-boot-usb]: https://askubuntu.com/a/855298/30904
[ubuntu-network-installer]: https://www.ubuntu.com/download/alternative-downloads
[boot-old-kernels]: https://bugs.launchpad.net/ubuntu/+source/unattended-upgrades/+bug/1357093
[basic-ubuntu-server]: https://askubuntu.com/a/153292/30904
[pi-hole]: https://pi-hole.net/
[nextcloud]: https://nextcloud.com/
[hiro-nakamura]: https://media0.giphy.com/media/6GxdekV8GmwhO/giphy.webp
[new-fans]: https://community.netgear.com/t5/ReadyNAS-Hardware-Compatibility/ReadyNAS-Pro-6-Power-Supply-FAN-replacement/td-p/1072881
