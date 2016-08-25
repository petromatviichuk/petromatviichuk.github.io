---
title: "Zabbix compilation dependencies"
layout: post
date: 2016-08-25 00:00
tag:
- zabbix server
- zabbix proxy
- zabbix installation
- compilation dependencies
blog: true
star: false
author: pmatv
---

I often compile Zabbix from sources for work and for testing purposes. 

In my opinion this is the best way of installation, because you can configure and install Zabbix Server or Proxy only with features that you exactly need and without relying on package options.

For each configuration, additional libraries will be needed, otherwise you'll receive a dependency error. 

So I've decided to find out which packages should be installed before compilation.

The installation has been performed in local lab on following environments:

* Operation systems: **Ubuntu 16.04.1 LTS / CentOS 7.2.1511 (Core)**
* Software: **Zabbix 3.0.4 LTS** 

After testing all cases, I've created a list with detailed information for each configuration flag:

{% highlight raw %}
--with-ibm-db2
configure: error: IBM DB2 library not found
{% endhighlight %}
**Solution:** According to Zabbix official documentation, CLI location should be set. If DB2 is already installed locally, you shouldn't receive such error. Otherwise download [IBM Data Server Driver for ODBC and CLI (CLI Driver)](http://www-01.ibm.com/support/docview.wss?rs=4020&uid=swg21385217)
<div class="breaker"></div>

{% highlight raw %}
--with-oracle
configure: error: Oracle OCI library not found
{% endhighlight %}
**Resolution:** Download and install [Oracle Instant Client](http://www.oracle.com/technetwork/database/features/instant-client)
<div class="breaker"></div>

{% highlight raw %}
--with-mysql
configure: error: MySQL library not found
{% endhighlight %}
**Resolution:** sudo apt-get install **libmysqld-dev** / sudo yum install **mysql-devel**

**Note:** in CentOS 7 mysql replaced with MariaDB so mariadb-devel will be installed
<div class="breaker"></div>

{% highlight raw %}
--with-postgresql
configure: error: PostgreSQL library not found
{% endhighlight %}
**Resolution:** sudo apt-get install **libpq-dev** / sudo yum install **postgresql-devel**
<div class="breaker"></div>

{% highlight raw %}
--with-sqlite3
configure: error: SQLite3 library not found
{% endhighlight %}
**Resolution:** sudo apt-get install **libsqlite3-dev** / sudo yum install **sqlite-devel**
<div class="breaker"></div>

{% highlight raw %}
--with-jabber
configure: error: Jabber library not found
{% endhighlight %}
**Resolution:** sudo apt-get install **libiksemel-dev** then try **--with-jabber=/usr** / for CentOS install RPM's from [zabbix repos](https://repo.zabbix.com/non-supported/rhel/7/x86_64/)
<div class="breaker"></div>

{% highlight raw %}
--with-libxml2
configure: error: LIBXML2 library not found
{% endhighlight %}
**Resolution:** sudo apt-get install **libxml2-dev** / sudo yum install **libxml2-devel**
<div class="breaker"></div>

{% highlight raw %}
--with-unixodbc
configure: error: unixODBC library not found
{% endhighlight %}
**Resolution:** sudo apt-get install **unixodbc-dev** / sudo yum install **unixODBC-devel**
<div class="breaker"></div>

{% highlight raw %}
--with-net-snmp
configure: error: Invalid Net-SNMP directory - unable to find net-snmp-config
{% endhighlight %}
**Resolution:** sudo apt-get install **libsnmp-dev snmp** / sudo yum install **net-snmp-devel net-snmp**

**Note:** I'd recommend install snmp/net-snmp also with devel packages just to avoid error of loading MIBs during Zabbix Server/Proxy first start.
<div class="breaker"></div>

{% highlight raw %}
--with-ssh2
configure: error: SSH2 library not found
{% endhighlight %}
**Resolution:** sudo apt-get install **libssh2-1-dev** / sudo yum install **libssh2-devel**
<div class="breaker"></div>

{% highlight raw %}
--with-openipmi
configure: error: Invalid OPENIPMI directory - unable to find ipmiif.h
{% endhighlight %}
**Resolution:** sudo apt-get install **libopenipmi-dev** / sudo yum install **OpenIPMI-devel**
<div class="breaker"></div>

{% highlight raw %}
--with-ldap
configure: error: Invalid LDAP directory - unable to find ldap.h
{% endhighlight %}
**Resolution:** sudo apt-get install **libldap2-dev** / sudo yum install **openldap-devel**
<div class="breaker"></div>

{% highlight raw %}
--with-libcurl
configure: error: Curl library not found
{% endhighlight %}
**Resolution:** sudo apt-get install **libcurl4-openssl-dev** / sudo yum install **libcurl-devel**
<div class="breaker"></div>

{% highlight raw %}
--with-mbedtls
configure: error: mbed TLS (PolarSSL) library libpolarssl not found
{% endhighlight %}
**Resolution:** Best options is to install mbed TLS from [sources](https://tls.mbed.org/download-archive)

**Note:** Currently Zabbix supports 1.3.X version only. See more [here](https://www.zabbix.com/documentation/3.0/manual/encryption#compiling_zabbix_with_encryption_support).
<div class="breaker"></div>

{% highlight raw %}
--with-gnutls
configure: error: GnuTLS library libgnutls not found
{% endhighlight %}
**Resolution:** sudo apt-get install **libgnutls-dev** / sudo yum install **gnutls-devel**


