---
title: "Helpful VM Commands"
header:
  image: /assets/images/2022-09-02-ultimate-truenas-guide/banner.png
toc: true
toc_sticky: true
categories:
  - Guides
tags:
  - Homelab
  - Networking
---

## Setting Static IPs

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

## Mounting SMB Shares

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

## Installing Docker and Portainer

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
