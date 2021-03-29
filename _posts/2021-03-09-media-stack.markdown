---
layout: post
title:  "Working title - My Media Stack"
date:   2021-03-09
image:  plex.jpg
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

{% highlight bash %}
$ sudo netplan apply
{% endhighlight %}

## Mounting SMB Shares:

My setup will be utilizing an SMB share of my array from a TrueNAS Core VM. I mount through fstab on both servers like so:

{% highlight bash %}
$ sudo apt install cifs-utils
{% endhighlight %}

{% highlight bash %}
$ sudo nano /etc/fstab
{% endhighlight %}

{% highlight bash %}
//truenas.jellayy.com/Data /Data cifs credentials=/etc/share-credentials,uid=1000,gid=1000 0 0
{% endhighlight %}

Create credentials file:

{% highlight bash %}
$ sudo nano /etc/share-credentials
{% endhighlight %}

{% highlight bash %}
username=user
password=password
{% endhighlight %}

Secure credentials file:

{% highlight bash %}
$ sudo chown root: /etc/share-credentials
$ sudo chmod 600 /etc/share-credentials
{% endhighlight %}

Mount share:

{% highlight bash %}
$ sudo mount -a
{% endhighlight %}

## Installing Docker and Portainer:

You can follow Docker's official guide for installing on Ubuntu [here](https://docs.docker.com/engine/install/ubuntu/). For those of you (including me in the future) that are spamming copy paste to follow along, here's some lines:

{% highlight bash %}
$ sudo apt install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
    
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

$ echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

$ sudo apt update

$ sudo apt install docker-ce docker-ce-cli containerd.io

$ sudo docker run hello-world
{% endhighlight %}

If hello world displayed its output that means everything is working great. For the future we're going to make things easier by adding your user to the docker group. Make sure to log out and back in after doing this.

{% highlight bash %}
$ sudo usermod -aG docker <YOUR-USER>
{% endhighlight %}
 
Then getting portainer running is as easy as:

{% highlight bash %}
$ docker volume create portainer_data

$ docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
{% endhighlight %}
