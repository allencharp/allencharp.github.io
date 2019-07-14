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
{% enghighlight %}