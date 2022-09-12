---
title: "Setting up Grafana - Simplified"
header:
  image: /assets/images/2021-07-31-grafana-setup/grafana.png
toc: true
toc_sticky: true
categories:
  - Guides
tags:
  - Homelab
  - Grafana
---

Grafana is a great tool for getting insights into your homelab or just to look at some pretty graphs; however, it is notorious for being quite a chore to set up. Thankfully, in my endless toiling of setting it up for my Homelab, I think I've come up with a pretty simple solution for getting set up, regardless of how many brain cells you may have.

## Preface

Grafana can be run on just about anything. For this guide, I'll be running it on my 'overseer' server, a custom x86-based 8-core Avoton C2750 low power 1u server running Ubuntu Server 20.04 LTS. While this could be easily recreated on other hardware/software, the rest of this guide will be assuming you have a similar debian-based setup.

## Installing Docker

Like 90% of my guides, this one starts with installing Docker, I just think it's neat.

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
 
This isn't strictly necessary for this guide, but I will be using portainer as well. The rest of this guide can also be completed via exclusively command line.

{% highlight bash %}
$ docker volume create portainer_data

$ docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
{% endhighlight %}

## Installing Grafana and InfluxDB

Grafana needs a database to pull data from. While Grafana has official support for basically every database under the moon, I chose InfluxDB here for its ease of use with Telegraf for collecting data as well as its web UI which lets you cheat and have it write queries for you.

If you're running portainer, you can deploy docker compose as a stack. Otherwise, go about installing and running docker compose via cli.

Github: [Jellayy/grafana/docker-compose.yml](https://github.com/Jellayy/grafana/blob/master/docker-compose.yml)
{% highlight yml %}
version: "3.3"

services:
  grafana:
    container_name: grafana_test
    image: grafana/grafana:latest
    ports:
      - 3000:3000
    restart: unless-stopped

  influxdb:
    container_name: influxdb_test
    image: influxdb:latest
    ports:
      - 8086:8086
    volumes:
      - influxdb2:/var/lib/influxdb2
    restart: unless-stopped

volumes:
  influxdb2:
{% endhighlight %}

You can then visit these services' web UIs at port 3000 and 8086 for inital setup (Grafana's default credentials are admin:admin). Go ahead and make your default accounts. When configuring InfluxDB, keep in mind what you call your organization and bucket, you'll need to give those to Grafana. You can then go to your grafana sources and point it to InfluxDB like so. Don't forget to grab your account's token from the InfluxDB web UI.

![Grafana DB Setup]({{ site.url }}{{ site.baseurl }}/assets/images/2021-07-31-grafana-setup/grafana-db-setup.png)

And that's it! You now technically have Grafana setup with its database. You can now go about setting up reporting to the InfluxDB database on your homelab devices. Below, I'll provide an example of setting up IPMITool with Telegraf to monitor stats on a Dell Poweredge r720's iDRAC.

## IPMITool + Telegraf iDRAC Monitoring

To start reporting to InfluxDB, we're going to have to install Telegraf. Since IPMI is inherently remote, you can install and run this on any server/VM that you want. For my purposes, I'm going to install to the same server I have Grafana and my DB running on.

{% highlight bash %}
$ wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -

$ source /etc/lsb-release

$ echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

$ sudo apt update

$ sudo apt install telegraf
{% endhighlight %}

Telegraf is entirely config based. The service will read from /etc/telegraf/telegraf.conf. There is a default config placed there on install that you can look at, but here's one that will fit our purpose.

Github: [Jellayy/grafana/iDRAC-IPMI Monitoring/telegraf.conf](https://github.com/Jellayy/grafana/blob/master/iDRAC-IPMI%20Monitoring/telegraf.conf)

Be sure to replace the respective values to those of your database and server, as well as any parameters you'd like to change for your setup. After replaceing telegraf.conf with this we can simply run the following to start the telegraf service and it will start reporting to your database.

{% highlight bash %}
$ systemctl start telegraf
{% endhighlight %}

If you're ever making your own configs for telegraf and you're running into issues that you'd like more debug output for, you can test your configs verbosely outside of the main service using this command.

{% highlight bash %}
$ telegraf --config ./PATH-TO-CONF
{% endhighlight %}

To see the stats being reported to your database, you can go into the InfluxDB web ui and use the Data Explorer.

![InfluxDB Data Explorer]({{ site.url }}{{ site.baseurl }}/assets/images/2021-07-31-grafana-setup/grafana-data-explorer.png)

## Making your first Grafana Dashboard

Now that we're reporting some data to our database and that database is connected to Grafana, we can make our first dashboard. If you create an empty panel in Grafana, you'll see that it's asking for a database query. Now learning flux to make some fancy graphs sounds pretty lame, thankfully we can cheat.

Remember InfluxDB's Data Explorer that we used to check our data with a nice GUI to select the data we want to view? Well we can use that to export queries by simply going to Save As > Variable. This gives us a query we can copy paste into Grafana, easy as pie.

![InfluxDB Query Export]({{ site.url }}{{ site.baseurl }}/assets/images/2021-07-31-grafana-setup/grafana-query-export.png)

Now you can mess around with the panel's settings and make plenty more wherever your imagination will take you! Be sure to save your dashboard when you're done so you don't lose it after a bunch of hard work like I did.
