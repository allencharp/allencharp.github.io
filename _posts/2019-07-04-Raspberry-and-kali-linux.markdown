---
layout: post
title:  "Raspberry and Kali Linux"
date:   2019-07-04 22:31:34 +0800
---

# Install Kali Linux in Raspberry 3
See the youtube video [here](https://www.youtube.com/watch?v=yCD4x38yOSg) <br>
Useful tool: [etcher](https://www.balena.io/etcher/) (burn the image into SD card)

# Enable SSH in Kail
{% highlight bash %}
sudo systemctl enable ssh
sudo systemctl start ssh
{% endhighlight %}

# (optional) Create Home NAS with Raspberry
See the youtube video [here](https://www.youtube.com/watch?v=EH6P6v3lxsE&t=388s)<br>
Useful tool: Samba <br>

{% highlight bash %}
sudo mount /dev/sda1 /External/
sudo /etc/init.d/smbd restart
{% endhighlight %}

# nmap script
All scirpts locate in /usr/share/nmap/scripts
{% highlight bash %}
cd /usr/share/nmap/scripts/
git clone https://github.com/vulnersCom/nmap-vulners.git
git clone https://github.com/scipag/vulscan.git
cd vulscan/utilities/updater/
chmod +x updateFiles.sh
./updateFiles.sh
{% endhighlight %}