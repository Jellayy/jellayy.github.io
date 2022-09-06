---
title: "Proxmox Setup Tips"
header:
  image: /assets/images/2022-09-02-ultimate-truenas-guide/banner.png
toc: true
toc_sticky: true
categories:
  - Guides
tags:
  - Homelab
---

Whether it be by wanting to start from scratch again, completely breaking everything, or having no backups when a drive dies, I install Proxmox quite frequently and I always seem to miss something. This writeup is mostly for myself to use for when I inevitably have to reinstall Proxmox again, but would also be a good guide to follow along to optimise your setup.

Of course the first step is to install the iso onto your boot drive, but I don't think I need to explain that here.

# Configuring The Network Interface

Since most of your management of Proxmox is going to be remote, getting your network set up correctly is usually my first step.

First for me is setting up a static IP. Personally, I also give this IP a custom DNS entry in pfSense as well. See my pfSense guide for instructions on how to do that.

![Static-ip]({{ site.url }}{{ site.baseurl }}/assets/images/2022-02-15-proxmox-setup/static-ip.jpg)

After this, don't forget to change the address in `/etc/hosts` as well, or else the webUI will open on the wrong address and be unaccessible. You also won't be able to authenticate with SSH if this happens.

# Remove Subscription Message
![Annoying Message]({{ site.url }}{{ site.baseurl }}/assets/images/2022-02-15-proxmox-setup/annoying-message.png)

1. Make a backup of `proxmoxlib.js`
{% highlight bash %}
cd /usr/share/javascript/proxmox-widget-toolkit
cp proxmoxlib.js proxmoxlib.js.bak
{% endhighlight %}

2. Edit `proxmoxlib.js` to replace the following function call:
{% highlight javascript %}
Ext.Msg.show({
  title: gettext('No valid subscription'),
{% endhighlight %}
{% highlight javascript %}
void({ //Ext.Msg.show({
  title: gettext('No valid subscription'),
{% endhighlight %}

3. Restart the proxmox web service
{% highlight bash %}
systemctl restart pveproxy.service
{% endhighlight %}

# PVEDiscordDark

I love the Proxmox web interface, but it's a little too bright. Thankfully, Weilbyte maintains a great and easy to install dark theme for the UI.

Github: [Weilbyte/PVEDiscordDark](https://github.com/Weilbyte/PVEDiscordDark)
{% highlight bash %}
wget https://raw.githubusercontent.com/Weilbyte/PVEDiscordDark/master/PVEDiscordDark.sh
bash PVEDiscordDark.sh install
{% endhighlight %}

# Enable Updates

Now, if you're like me, you don't pay for proxmox. However, it is stil pretty easy to enable updates without a subscription.

First, you'll want to edit your `/etc/apt/sources.list` with the latest no subscription sources from [here.](https://pve.proxmox.com/wiki/Package_Repositories)

{% highlight bash %}
deb http://ftp.debian.org/debian bullseye main contrib
deb http://ftp.debian.org/debian bullseye-updates main contrib

# PVE pve-no-subscription repository provided by proxmox.com,
# NOT recommended for production use
deb http://download.proxmox.com/debian/pve bullseye pve-no-subscription

# security updates
deb http://security.debian.org/debian-security bullseye-security main contrib
{% endhighlight %}

If you don't have a subscription, you'll also have to comment out this line in `/etc/apt/sources.list.d/pve-enterprise.list`

{% highlight bash %}
# deb https://enterprise.proxmox.com/debian/pve bullseye pve-enterprise
{% endhighlight %}

After that, you can test your changes with:

{% highlight bash %}
apt update
apt dist-upgrade
{% endhighlight %}

# Install TrueNAS Core

Many people will simply setup a ZFS array for the disks on their server. However, I really like the featureset and management that the TrueNAS Core OS provides, so I virtualize it and pass all my drives to it. (Note: These drives are for shared storage and backups, my VM storage is on an SSD)

The TrueNAS installation is pretty straightforward; however, the one part you need to pay attention to is passing your disks. The UI only allows passing disks by sd#, which can change depending on how the drives initialize at boot. The proper practice is to add the disks by ID, which can only be done through the terminal.

To see disk IDs run:

{% highlight bash %}
ls -l /dev/disk/by-id
{% endhighlight %}

Then add a disk to your VM by running the following command, incramenting the scsi ID as you go.

{% highlight bash %}
qm set VM_ID -scsiX /dev/disk/by-id/DRIVE_ID
{% endhighlight %}

If you've done everything right, your hardware tab should look something like this:

![Drives]({{ site.url }}{{ site.baseurl }}/assets/images/2022-02-15-proxmox-setup/drive-by-id.jpg)