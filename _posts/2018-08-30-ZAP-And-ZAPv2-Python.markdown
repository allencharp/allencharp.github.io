---
layout: post
title:  "ZAP and ZAPv2 Python Library"
date:   2018-08-30 21:31:34 +0800
author: allencharp
tags: [pentest, zap, python]
---

# Setup ZAP

Some configuration notes for OWASP ZAP.

## Setup proxy
![set the zap proxy]({{site.baseurl}}/assets/images/zap1.jpg)

## Setup certification
Generate the ZAP certificate from **Tools -> Options -> Dynamic SSL Certification**, then import the certificate file into the browser.

## Browser proxy setup
The browser proxy should be the same as the ZAP proxy.

## Other common ZAP settings
* Download community scripts from [zap-extensions releases](https://github.com/zaproxy/zap-extensions/releases), then you can use Python/JavaScript to write your own rules (details will be covered later).
* `pip install python-owasp-zap-v2.4` to install the OWASP ZAP API (ZAPv2 library).
* Copy the API Key from **Tools -> Options -> API**; it will be used in ZAPv2 scripts.
![API KEY]({{site.baseurl}}/assets/images/zap2.jpg)

# Use the ZAPv2 API

Initialize the ZAPv2 object and set the scan target:

{% highlight python %}
from zapv2 import ZAPv2

target = 'http://127.0.0.1'   # your scan target
apikey = '<your-api-key>'     # replace with the key from Tools -> Options -> API

zap = ZAPv2(apikey=apikey,
            proxies={'http': 'http://127.0.0.1:8090',
                     'https': 'http://127.0.0.1:8090'})

zap.urlopen(target)
{% endhighlight %}

## Use Selenium to drive Firefox

Let ZAP listen to the Firefox traffic and scan for vulnerabilities. In this way, login/auth flows can be automated:

{% highlight python %}
from selenium import webdriver

capabilities = webdriver.DesiredCapabilities.FIREFOX
capabilities['proxy'] = {
    'proxyType': "manual",
    'httpProxy': "127.0.0.1:8080",
    'ftpProxy': "127.0.0.1:8080",
    'sslProxy': "127.0.0.1:8080"
}
capabilities['acceptInsecureCerts'] = True

driver = webdriver.Firefox(capabilities=capabilities)
driver.maximize_window()
{% endhighlight %}

# Run ZAP in Docker

ZAP also ships official Docker images (e.g. `owasp/zap2docker-stable`), which are handy for CI/CD pipelines:

{% highlight bash %}
docker pull owasp/zap2docker-stable
docker run -t owasp/zap2docker-stable zap-baseline.py -t https://example.com
{% endhighlight %}
