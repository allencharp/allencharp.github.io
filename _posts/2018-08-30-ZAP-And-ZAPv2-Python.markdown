---
layout: post
title:  "ZAP and ZAPv2 Python Library"
date:   2018-08-30 21:31:34 +0800
---
# Setup ZAP
Some configuration of OWASP ZAP

#### Setup proxy
![set the zap proxy]({{site.baseurl}}/assets/images/zap1.jpg)

#### Setup certification
Generate the ZAP certification from Tools->Options->Dynamic SSL Certification<br>
Import the certification file into browser

#### Browser setup the proxy
The proxy should be the same as ZAP proxy

#### Other ZAP common setting
* Download community script from : https://github.com/zaproxy/zap-extensions/releases, then we could use Python/Javascript to write our own rules (the details will be talked later)
* pip install python-owasp-zap-v2.4 to download the OWASP ZAP API (ZAPv2 library)
* Copy the API Key from Tools->Options->API, it will be used in ZAPv2 script
![API KEY]({{site.baseurl}}/assets/images/zap2.jpg) 

# Use ZAPv2 API
#### Init ZAPv2 object and set the scan target:
{% highlight python %}
from zapv2 import ZAPv2

target = 'http://127.0.0.1' # your scan target
apikey = 'apikey' # the api key from ZAP->Tools->Options-API 

zap = ZAPv2(apikey=apikey, proxies={'http': 'http://127.0.0.1:8090', 'https': 'http://127.0.0.1:8090'})

zap.urlopen(target)
{% endhighlight %}

#### Use selenium to launch firefox, 
#### make the ZAP to listen the firefox traffic and scan the vlun issue:
#### In this way, we could automate the login/auth scan
{% highlight python %}
capabilities = webdriver.DesiredCapabilities.FIREFOX
capabilities['proxy'] = {
    'proxyType': "manual",
    'httpProxy': "127.0.0.1:8080",
    'ftpProxy': "127.0.0.1:8080",
    'sslProxy': "127.0.0.1:8080"
    }
capabilities['acceptInsecureCerts'] = True

capabilities['acceptSslCerts'] = False

self.driver = webdriver.Firefox(capabilities=capabilities)
self.driver.maximize_window()
{% endhighlight %}

# User ZAP Docker Container
