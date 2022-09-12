---
title: "pfSense, The Humble Beginning"
header:
  image: /assets/images/2020-08-30-pfsense-setup/pfsense.png
toc: true
toc_sticky: true
categories:
  - Guides
tags:
  - Homelab
  - Networking
---

With my first virtulization server project in the works, there is still one major hurdle I have to get past before I can *really* get to work on that. My college dorm internet. This thread will be all about my findings and experience with pfSense, the first addition to my first homelab.


## Problems to Solve:

1. My dorm only gives me a single ethernnet jack with a single connection, I'd like to be able to manage and switch that connection.
2. There is no way for me to port forward on my dorm connection, so even though I have 400 up and down, I can't really host anything.
3. I'd like to be able to remote into my network without making everything public to the entire dorm. (I can see everyone's game consoles when I open spotify)


## The Hardware:

![Hardware Image]({{ site.url }}{{ site.baseurl }}/assets/images/2020-08-30-pfsense-setup/pfsense-hardware.png)
Originally serving time in use in an ASU linux development class, this box eventually made it into a lot of "computer parts" in a surplus auction. A friend and I got a couple of these along with a bunch of other disregarded university hardware for basically nothing (ASU also did not bother clearing off the drive). This box ended up being a really good canidate for pfSense.

CPU: Intel Atom D510 (2c4t@1.66GHz, 13W)

RAM: 1GB DDR2

HDD: WD Scorpio Blue 160GB

MOBO: 1 SODIMM Slot, 2 1GIG Ethernet, VGA, Lots of Serial

The CPU actually ended up being a lot more than I thought it would be. I was expecting 1c2t honestly. I could get some mildly serious stuff running on here with just a simple RAM upgrade. That plus the dual 1GIG ports made this almost perfect for my first pfSense box.



## Setup:

Setup was actually pretty painless all things considered. The only issue I faced was some wonky stuff with the bootable USB drive I had with the [pfSense iso](https://www.pfsense.org/download/) on it constantly formatting itself for no reason. After that, I followed the guided setup, plugged into the LAN port to finish things up, and that was it. For open-source router software, that was way more plug-and-play than I expected.

![Blending In...]({{ site.url }}{{ site.baseurl }}/assets/images/2020-08-30-pfsense-setup/pfsense-blending-in.png)

Hopefully this will allow me to not raise too much suspicion if someone were to check the dorm network. A device named "xbox 360" on ethernet pumping out a lot of encrypted traffic looks normal right?

![Switch]({{ site.url }}{{ site.baseurl }}/assets/images/2020-08-30-pfsense-setup/pfsense-switch.png)

Currently, I'm simply using one of the ports on the pfSense box for my WAN interface and the other for my LAN interface going straight into a tiny, USB powered, unmanaged switch. As my homelab scales up, this will definitely be a canidate for an upgrade. This tiny thing doesn't like it when running local iso transfers to my server and other downloads at the same time. I'd like to get a rack-mountable managed cisco switch as I am very familiar with the Cisco iOS and could easily set up VLANs and ACLs on it.


## DNS Resolving:
A cool added bonus I'm now able to do with pfSense is DNS Resolving along with static IPs. The process for doing this was fairly simple as well.

The first thing you want to do is set up your host with a static IP. First create a DHCP static mapping in pfSense by going to Services > DHCP Server > LAN and scrolling to the bottom of the page to make a new entry. I'll be using my proxmox container for plex for this example.

![DHCP Entry]({{ site.url }}{{ site.baseurl }}/assets/images/2020-08-30-pfsense-setup/pfsense-dhcp-entry.png)

Simply replace the respective values with your devices MAC address, any identifier you want, the static address you're assigning (make sure it is outside the range of your DHCP server), a hostname and description, then make sure to create an ARP table entry so you can refrence you entries at any time
