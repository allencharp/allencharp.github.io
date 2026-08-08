---
layout: post
title:  "Raspberry and Kali Linux"
date:   2019-07-04 22:31:34 +0800
author: allencharp
tags: [kali, raspberry-pi, pentest]
---

# Install Kali Linux on Raspberry Pi 3

A Raspberry Pi running Kali makes a cheap, portable pentesting box. See the YouTube video [here](https://www.youtube.com/watch?v=yCD4x38yOSg).  
Useful tool: [Etcher](https://www.balena.io/etcher/) (burn the image to an SD card).

# Enable SSH in Kali

By default SSH is disabled. Enable and start it:

{% highlight bash %}
sudo systemctl enable ssh
sudo systemctl start ssh
{% endhighlight %}

## SSH hardening tip

For a lab or field device, prefer key-based authentication: copy your public key to `~/.ssh/authorized_keys`, then disable password logins in `/etc/ssh/sshd_config` (`PasswordAuthentication no`). This keeps brute force off your box.

# (Optional) Create a Home NAS with Raspberry Pi

See the YouTube video [here](https://www.youtube.com/watch?v=EH6P6v3lxsE&t=388s).  
Useful tool: Samba.

{% highlight bash %}
sudo mount /dev/sda1 /External/
sudo /etc/init.d/smbd restart
{% endhighlight %}

# Nmap Scripts

All scripts are located in `/usr/share/nmap/scripts`.  
Read the introduction [here](https://null-byte.wonderhowto.com/how-to/easily-detect-cves-with-nmap-scripts-0181925/).

{% highlight bash %}
# go to nmap script folder
cd /usr/share/nmap/scripts/

# get the nmap-vulners nse
git clone https://github.com/vulnersCom/nmap-vulners.git

# get the vulscan nse
git clone https://github.com/scipag/vulscan.git

# update
cd vulscan/utilities/updater/
chmod +x updateFiles.sh
./updateFiles.sh
{% endhighlight %}

The nmap command will be:

{% highlight bash %}
nmap --script vulscan -sV www.baidu.com
{% endhighlight %}

Notes:

* `nmap-vulners` queries the Vulners database and reports CVEs for the detected service versions.
* `vulscan` maintains its own local vulnerability database (updated by `updateFiles.sh`).
* Both need `-sV` (version detection) to know which software versions are running.

# Wireshark

Wireshark is a packet analyzer that comes pre-installed on Kali; use it to inspect HTTP, DNS and other traffic while testing. For headless usage on the Pi, `tshark` (the CLI version) is a good alternative.

# Summary

A Kali Pi is a compact, always-ready platform for network tests: burn the image, enable SSH with key auth, load nmap vulnerability scripts, and you have a field device that can scan, sniff and pivot — plus a Samba NAS when you need storage.
