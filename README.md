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
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d
```
Install Elasticsearch
```
sudo apt update
sudo apt install elasticsearch
```
File config: `/etc/elasticsearch/elasticsearch.yml`

Template file config: `elasticsearch.yml.template`

Start & Enable Elasticsearch
```
systemctl start elasticsearh
systemctl enable elasticsearh
systemctl status elasticsearh
```
## Install Kibana 7.x on Ubuntu 20.04 | Debian 9/10
```
apt update
sudo apt install kibana
```
File config: `/etc/kibana/kibana.yml`

Template file config: `kibana.yml.template`

Start & Enable Kibana
```
systemctl start kibana
systemctl enable kibana
systemctl status kibana
```

Generate password auto for users Elastic + Kibana:

```
/usr/share/elasticsearch/bin/ /elasticsearch-setup-passwords auto
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

