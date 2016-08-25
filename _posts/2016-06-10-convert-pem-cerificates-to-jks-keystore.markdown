---
title: "Convert PEM certificates to JKS keystore"
layout: post
date: 2016-06-10 22:44
tag:
- jks
- pem
blog: true
star: false
author: pmatv
---

Technically, main idea is simple: export PEM certificate and private key to **PKCS#12 container**, import container to [Java keystore](https://en.wikipedia.org/wiki/Keystore).

For generating PKCS#12 container we will use openssl:

{% highlight shell %}
pmatv@work-pro:~$ sudo openssl pkcs12 -export -name myalias -in ca.crt -inkey ca.key -out ca.p12
Enter Export Password: mystrongexportpass
Verifying - Enter Export Password: mystrongexportpass
{% endhighlight %}

where:

* **-name** is a unique alias for future keystore
* **-in** is a path and a file name of PEM certificate
* **-inkey** is a path and a name of the private key
* **-out** is a path where PKCS#12 container must be saved

It's mandatory to set **Export Password** as it will be used later.

After successfull result of openssl command, new container **ca.p12** can be imported to Java keystore.

For creating Java keystore, we have to use keytool program. This tool is included in the bin directory of the Java SDK.

{% highlight shell %}
pmatv@work-pro:~$ keytool -importkeystore -destkeystore ca.jks -srckeystore ca.p12 -srcstoretype pkcs12 -alias myalias
Enter destination keystore password: mystrongkeystorepass
Re-enter new password: mystrongkeystorepass
Enter source keystore password: mystrongexportpass
{% endhighlight %}
where:

* **-destkeystore** is a path and a name of future keystore
* **-srckeystore** is a path and a name of our PKCS#12 container
* **-alias** should be same unique name that we've used for openssl command

So at the end you should have file **ca.jks** which is Java keystore.

Also remember about two passwords:

* **mystrongkeystorepass** - private key password (**keyPass** SSL connector attribute option in tomcat configuration)
* **mystrongexportpass** - keystore password (**keystorePass** SSL connector attribute in tomcat configuration)


If you'd like to check content of keystore, you can run following command:

{% highlight shell %}
pmatv@work-pro:~$ keytool -list -keystore ca.jks -rfc
Enter keystore password: mystrongkeystorepass

Keystore type: JKS
Keystore provider: SUN

Your keystore contains 1 entry

Alias name: myalias
Creation date: Jun 15, 2016
Entry type: PrivateKeyEntry
Certificate chain length: 2
Certificate[1]:
-----BEGIN CERTIFICATE-----
MIIETTCCAzWgAwIBAgIDAjpxMA0GCSqGSIb3DQEBCwUAMEIxCzAJBgNVBAYTAlVTMRYwFAYDVQQK
Ew1HZW9UcnVzdCBJbmMuMRswGQYDVQQDExJHZW9UcnVzdCBHbG9iYWwgQ0EwHhcNMTMxMjExMjM0
NTUxWhcNMjIwNTIwMjM0NTUxWjBCMQswCQYDVQQGEwJVUzEWMBQGA1UEChMNR2VvVHJ1c3QgSW5j
....
-----END CERTIFICATE-----
{% endhighlight %}
