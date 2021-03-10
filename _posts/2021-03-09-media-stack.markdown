---
layout: post
title:  "Working title - My Media Stack"
date:   2021-03-09
image:  pfsense.png
tags:   [Homelab]
---

## Disclaimer:

This post in no way supports or promotes the act of piracy. Piracy is illegal and you should only ever acquire media you own or have legal access to. The intent of this post is purely educational in nature and serves to build IT knowledge of media distribution servers.

## Preface:

My media stack is hosted across two different servers. The first server is a Dell Poweredge R720 with 2 Xeon E5-2640 CPUs that acts as my NAS. The other is a custom built virtulization server with two Xeon E5-2658 v2 CPUs. If anyone is following along with only one server, you can combine all of these services onto one; however, for my purposes, I will be hosting everything related to acquisition and downloading on my NAS with all distribution and monitoring services on my virtulization server.

## Initial Setup:

On both of my servers I will be setting up dedicated Ubuntu Live Server VMs in Proxmox. You could use any other distribution or even any other OS with docker support; however, the rest of this post will be centered around this setup.

## Setting Static IPs:

For this sort of setup it is a very good idea to have everything running on static IP addresses. The easiest way to do this on Ubuntu Server is thorugh netplan. Use whatever addressing scheme you have set up in your network.

{% highlight bash %}
$ sudo nano /etc/netplan/01-netcfg.yaml
{% endhighlight %}

{% highlight yaml %}
network:
 version: 2
 renderer: networkd
 ethernets:
  ens5:
   dhcp4: no
   addresses: [192.168.1.6/24]
   gateway4: 192.168.1.1
   nameservers:
    addresses: [192.168.1.1]
{% endhighlight %}
