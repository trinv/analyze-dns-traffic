# Install Elastic + Kibana + Packetbeat
## 1. Elastic + Kibana on the same server
## Install Elasticsearch 7.x on Ubuntu 20.04 | Debian 9/10
```
apt update
```
Install requirement package JDK-8
```
apt install openjdk-8-jdk
```
```
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list

```
Install Elasticsearch
```
sudo apt update
sudo apt install elasticsearch
```
File config: `/etc/elasticsearch/elasticsearch.yml`

Template file config: [elasticsearch.yml.template](https://github.com/trinv/elk-dns/blob/master/elasticsearch.yml.template)

Start & Enable Elasticsearch
```
systemctl start elasticsearch
systemctl enable elasticsearch
systemctl status elasticsearch
```
## Install Kibana 7.x on Ubuntu 20.04 | Debian 9/10
```
apt update
sudo apt install kibana
```
File config: `/etc/kibana/kibana.yml`

Template file config: [kibana.yml.template](https://github.com/trinv/elk-dns/blob/master/kibana.yml.template)

Start & Enable Kibana
```
systemctl start kibana
systemctl enable kibana
systemctl status kibana
```

Generate password auto for users Elastic + Kibana:

```
/usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
```
Output:
```
Changed password for user apm_system
PASSWORD apm_system = wbzklsCm7lI5EbGXmC6O

Changed password for user kibana_system
PASSWORD kibana_system = Aw0Wkdxpai7gZcEXxmgz

Changed password for user kibana
PASSWORD kibana = Aw0Wkdxpai7gZcEXxmgz

Changed password for user logstash_system
PASSWORD logstash_system = frGpzd4W5yuWitbELwA7

Changed password for user beats_system
PASSWORD beats_system = yn7m0TA8KJevyuMSzPa0

Changed password for user remote_monitoring_user
PASSWORD remote_monitoring_user = Zl7bbH9cpu5DeX5iWn0E

Changed password for user elastic
PASSWORD elastic = eHGIdPnw0x3VeNy9Z3Ua
```
Update those information into `kibana.yml` file config:
```
elasticsearch.username: "kibana_system"
elasticsearch.password: "Aw0Wkdxpai7gZcEXxmgz"
```
## Enable X-Pack security for elastic & kibana
Change the configuration in [elasticsearch.yml.template](https://github.com/trinv/elk-dns/blob/master/elasticsearch.yml.template) file:
```
xpack.monitoring.collection.enabled: true
xpack.security.enabled: true
```
Generate Xpack Encrytion Key:
```
/usr/share/kibana/bin/kibana-encryption-keys generate
```
Update those information into [kibana.yml.template](https://github.com/trinv/elk-dns/blob/master/kibana.yml.template) file config:
```
  #Settings:
xpack.encryptedSavedObjects.encryptionKey: ca1072cad8df9051338308968c9cbcfb
xpack.reporting.encryptionKey: f850893b440fc2511b85683d98a29c6e
xpack.security.encryptionKey: f99f7ddd65d7f17e0340ed00eb8a0402
```
Restart Elastic & Kibana
```
systemctl restart elasticsearch
systemctl restart kibana
```
## 2. Install Packetbeat on the Agent server
```
apt update
curl -L -O https://artifacts.elastic.co/downloads/beats/packetbeat/packetbeat-7.14.1-amd64.deb
sudo dpkg -i packetbeat-7.14.1-amd64.deb
```
Start & Enable Packetbeat
```
systemctl start packetbeat
systemctl enable packetbeat
systemctl status packetbeat
```
Edit config file `/etc/packetbeat/packetbeat.yml` to get DNS traffic on DNS servers  & push into Elastic & Kibana server
* DNS traffic
```
packetbeat.protocols:

  - type: dns
  # Configure the ports where to listen for DNS traffic. You can disable
  # the DNS protocol by commenting out the list of ports.
    ports: [53]
    include_authorities: true
    include_additionals: true
```
* Connect to Elastic & Kibana server:

```
# =================================== Kibana ===================================

# Starting with Beats version 6.0.0, the dashboards are loaded via the Kibana API.
# This requires a Kibana endpoint configuration.
setup.kibana:

  # Kibana Host
  # Scheme and port can be left out and will be set to the default (http and 5601)
  # In case you specify and additional path, the scheme is required: http://localhost:5601/path
  # IPv6 addresses should always be defined as: https://[2001:db8::1]:5601
  host: "192.168.5.5:5601"

  # Kibana Space ID
  # ID of the Kibana Space into which the dashboards should be loaded. By default,
  # the Default Space will be used.
  #space.id:

# ---------------------------- Elasticsearch Output ----------------------------
output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["192.168.5.5:9200"]
  pipeline: geoip-info

  # Protocol - either `http` (default) or `https`.
  #protocol: "https"

  # Authentication credentials - either API key or username/password.
  #api_key: "id:api_key"
  username: "elastic"
  password: "eHGIdPnw0x3VeNy9Z3Ua"
```
* User & password for elastic that we generated before on Elastic server

Reference in config file template [packetbeat.yml.template](https://github.com/trinv/elk-dns/blob/master/packetbeat.yml.template)

Restart Packetbeat
```
systemctl restart packetbeat
systemctl status packetbeat
```
## 3. GeoIP Kibana:

Configure the geoip processoredit
To configure Filebeat and the geoip processor:

Define an ingest node pipeline that uses one or more geoip processors to add location information to the event. For example, you can use the Console in Kibana `Dev Tool` to create the following pipeline:
```
PUT _ingest/pipeline/geoip-info
{
  "description": "Add geoip info",
  "processors": [
    {
      "geoip": {
        "field": "client.ip",
        "target_field": "client.geo",
        "ignore_missing": true
      }
    },
    {
      "geoip": {
        "field": "source.ip",
        "target_field": "source.geo",
        "ignore_missing": true
      }
    },
    {
      "geoip": {
        "field": "destination.ip",
        "target_field": "destination.geo",
        "ignore_missing": true
      }
    },
    {
      "geoip": {
        "field": "server.ip",
        "target_field": "server.geo",
        "ignore_missing": true
      }
    },
    {
      "geoip": {
        "field": "host.ip",
        "target_field": "host.geo",
        "ignore_missing": true
      }
    }
  ]
}
```
 
In this example, the pipeline ID is `geoip-info`. field specifies the field that contains the IP address to use for the geographical lookup, and `target_field` is the field that will hold the geographical information. `"ignore_missing":` true configures the pipeline to continue processing when it encounters an event that doesn’t have the specified field.

See GeoIP Processor for more options.

To learn more about adding host information to an event, see `add_host_metadata`.

In the `Packetbeat` config file, configure the Elasticsearch output to use the `pipeline`. Specify the pipeline ID in the pipeline option under `output.elasticsearch`. For example:
```
output.elasticsearch:
  hosts: ["localhost:9200"]
  pipeline: geoip-info
```
Run `Packetbeat`. Remember to use sudo if the config file is owned by root.
```
systemctl restart packetbeat
```
If the lookups succeed, the events are enriched with `geo_point` fields, such as `client.geo.location` and `host.geo.location`, that you can use to populate visualizations in Kibana.

If you add a field that’s not already defined as a `geo_point` in the index template, add a mapping so the field gets indexed correctly.

## Notice
It not working, You can add on Kibana Console:
Define an ingest pipeline that uses one or more `geoip` processors to add location information to the event. For example, you can use the Console in Kibana to create the following pipeline:
```
PUT _ingest/pipeline/geoip-info
{
  "description": "Add geoip info",
  "processors": [
    {
      "geoip": {
        "field": "client.ip",
        "target_field": "client.geo",
        "ignore_missing": true
      }
    },
    {
      "geoip": {
        "database_file": "GeoLite2-ASN.mmdb",
        "field": "client.ip",
        "target_field": "client.as",
        "properties": [
          "asn",
          "organization_name"
        ],
        "ignore_missing": true
      }
    },
    {
      "geoip": {
        "field": "source.ip",
        "target_field": "source.geo",
        "ignore_missing": true
      }
    },
    {
      "geoip": {
        "database_file": "GeoLite2-ASN.mmdb",
        "field": "source.ip",
        "target_field": "source.as",
        "properties": [
          "asn",
          "organization_name"
        ],
        "ignore_missing": true
      }
    },
    {
      "geoip": {
        "field": "destination.ip",
        "target_field": "destination.geo",
        "ignore_missing": true
      }
    },
    {
      "geoip": {
        "database_file": "GeoLite2-ASN.mmdb",
        "field": "destination.ip",
        "target_field": "destination.as",
        "properties": [
          "asn",
          "organization_name"
        ],
        "ignore_missing": true
      }
    },
    {
      "geoip": {
        "field": "server.ip",
        "target_field": "server.geo",
        "ignore_missing": true
      }
    },
    {
      "geoip": {
        "database_file": "GeoLite2-ASN.mmdb",
        "field": "server.ip",
        "target_field": "server.as",
        "properties": [
          "asn",
          "organization_name"
        ],
        "ignore_missing": true
      }
    },
    {
      "geoip": {
        "field": "host.ip",
        "target_field": "host.geo",
        "ignore_missing": true
      }
    },
    {
      "rename": {
        "field": "server.as.asn",
        "target_field": "server.as.number",
        "ignore_missing": true
      }
    },
    {
      "rename": {
        "field": "server.as.organization_name",
        "target_field": "server.as.organization.name",
        "ignore_missing": true
      }
    },
    {
      "rename": {
        "field": "client.as.asn",
        "target_field": "client.as.number",
        "ignore_missing": true
      }
    },
    {
      "rename": {
        "field": "client.as.organization_name",
        "target_field": "client.as.organization.name",
        "ignore_missing": true
      }
    },
    {
      "rename": {
        "field": "source.as.asn",
        "target_field": "source.as.number",
        "ignore_missing": true
      }
    },
    {
      "rename": {
        "field": "source.as.organization_name",
        "target_field": "source.as.organization.name",
        "ignore_missing": true
      }
    },
    {
      "rename": {
        "field": "destination.as.asn",
        "target_field": "destination.as.number",
        "ignore_missing": true
      }
    },
    {
      "rename": {
        "field": "destination.as.organization_name",
        "target_field": "destination.as.organization.name",
        "ignore_missing": true
      }
    }
  ]
}
```
https://www.elastic.co/guide/en/elasticsearch/reference/current/geoip-processor.html#ingest-geoip-downloader-endpoint
