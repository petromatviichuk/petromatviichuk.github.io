---
title: "Nested json in Logstash"
layout: post
date: 2016-07-08 23:00
tag:
- logstash
- nested json
- geoip
blog: true
star: false
author: pmatv
---


Recently I experienced problem in logstash with handling nested json data. The document was separated by sub-documents and looked like:

{% highlight json %}
{
"date": "2016-07-08T23:00:05.000Z",
    "request": {
      "id": "57",
      "url": "http://localhost/jsondata",
      "clientIp": "8.8.8.8",
      "userAgent": "curl/7.43.0",
      "page": {
        "id": "575",
        "name": "Test data"
      }
     }
}
{% endhighlight %}

After spending some time, I resolved my problem and noticed few useful cases.

Let's take simple logstash configuration for demonstrating them.

### Case 1: Process field value in logstash

For accessing field values in logstash [sprintf](https://www.elastic.co/guide/en/logstash/current/event-dependent-configuration.html#sprintf) format is used.

According to configuration, value of field **request.id**'s should be assigned to custom field **user-request-id**.

{% highlight shell %}
$ cat json-test.conf
input { stdin { codec => "json" } }
filter {
  mutate {
    add_field => {
      "user-request-id" => "%{[request][id]}"
    }
  }
output { stdout { codec => "rubydebug" } }
}

$ echo '{"date": "2016-07-08T23:00:05.000Z","request": {"id": "57","url": "http://localhost/jsondata","clientIp": "8.8.8.8","userAgent": "curl/7.43.0","page": {"id": "575","name":"Test data"}}}' | /opt/logstash/bin/logstash -f json-test.conf
Settings: Default pipeline workers: 1
Pipeline main started
{
               "date" => "2016-07-08T23:00:05.000Z",
       "request" => {
               "id" => "57",
              "url" => "http://localhost/jsondata",
         "clientIp" => "8.8.8.8",
        "userAgent" => "curl/7.43.0",
             "page" => {
              "id" => "57",
            "name" => "Test data"
        }
        }
    },
           "@version" => "1",
         "@timestamp" => "2016-07-08T23:07:39.064Z",
               "host" => "5522dbfa5532",
    "user-request-id" => "57"
}
Pipeline main has been shutdown
stopping pipeline {:id=>"main"}
{% endhighlight %}

As result, **user-request-id** now has "57" as new value.

### Case 2: Process field in logstash

Let's take previous example and try to add location information about client IP addresses. In our case it's **"clientIp"** field.

To handle our nested field, the full path to that field should be specified: **"[request][clientIp]"**.

Let's slightly modify configuration and see results:

{% highlight shell %}
$ cat json-test.conf
input { stdin { codec => "json" } }
filter {
  mutate {
    add_field => {
      "user-request-id" => "%{[request][id]}"
    }
  }
  geoip {
    source => "[request][clientIp]"
    target => "geoip"
  }
}
output { stdout { codec => "rubydebug" } }

$ echo '{"date": "2016-07-08T23:00:05.000Z","request": {"id": "57","url": "http://localhost/jsondata","clientIp": "8.8.8.8","userAgent": "curl/7.43.0","page": {"id": "575","name":"Test data"}}}' | /opt/logstash/bin/logstash -f json-test.conf
Settings: Default pipeline workers: 1
Pipeline main started
{
          "date" => "2016-07-08T23:00:05.000Z",
       "request" => {
               "id" => "57",
              "url" => "http://localhost/jsondata",
         "clientIp" => "8.8.8.8",
        "userAgent" => "curl/7.43.0",
             "page" => {
              "id" => "57",
            "name" => "Test data"
        }
    },
           "@version" => "1",
         "@timestamp" => "2016-07-08T23:09:23.014Z",
               "host" => "5522dbfa5532",
    "user-request-id" => "57"
         "geoip" => {
                      "ip" => "8.8.8.8",
           "country_code2" => "US",
           "country_code3" => "USA",
            "country_name" => "United States",
          "continent_code" => "NA",
             "region_name" => "CA",
               "city_name" => "Mountain View",
             "postal_code" => "94043",
                "latitude" => 37.41919999999999,
               "longitude" => -122.0574,
                "dma_code" => 807,
               "area_code" => 650,
                "timezone" => "America/Los_Angeles",
        "real_region_name" => "California",
                "location" => [
            [0] -122.0574,
            [1] 37.41919999999999
        ]
    }
}
Pipeline main has been shutdown
stopping pipeline {:id=>"main"}
{% endhighlight %}

Afterwards the client IP address has been successfully processed by geoip module and we received information about the geographical location.





