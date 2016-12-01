# What is the Elastic Stack?
By combining the massively popular Elasticsearch, Logstash, and Kibana, Elastic has created an end-to-end stack that delivers actionable insights in real time from almost any type of structured and unstructured data source. Built and supported by the engineers behind each of these open source products, the Elastic Stack makes searching and analyzing data easier than ever before.

[<img src="https://www.elastic.co/assets/blte1ccb52ef00ca60e/icon-elastic-stack-bb.svg" width="144" height="144">](https://www.elastic.co/)


# Based on:

* [<img src="https://static-www.elastic.co/fr/assets/blt282ae2420e32fc38/icon-kibana-bb.svg?q=802" width="50" height="50">](https://www.elastic.co/fr/products/kibana) [![](https://images.microbadger.com/badges/image/khezen/kibana.svg)](https://hub.docker.com/r/khezen/kibana/) [khezen/kibana](https://github.com/Khezen/docker-kibana)
* [<img src="https://static-www.elastic.co/fr/assets/blt9a26f88bfbd20eb5/icon-elasticsearch-bb.svg?q=802" width="50" height="50">](https://www.elastic.co/fr/products/elasticsearch) [![](https://images.microbadger.com/badges/image/khezen/elasticsearch.svg)](https://hub.docker.com/r/khezen/elasticsearch/) [khezen/elasticsearch](https://github.com/Khezen/docker-elasticsearch)
* [<img src="https://static-www.elastic.co/fr/assets/blt946bc636d34a70eb/icon-logstash-bb.svg?q=600" width="50" height="50">](https://www.elastic.co/fr/products/logstash) [![](https://images.microbadger.com/badges/image/khezen/logstash.svg)](https://hub.docker.com/r/khezen/logstash/) [khezen/logstash](https://github.com/Khezen/docker-logstash)
* [<img src="https://static-www.elastic.co/assets/blt121ead33d4ed1f55/icon-beats-bb.svg?q=600" width="50" height="50">](https://www.elastic.co/products/beats) [Beats](https://www.elastic.co/guide/en/beats/libbeat/current/installing-beats.html)
  * [metricbeat](https://www.elastic.co/guide/en/beats/metricbeat/current/index.html)
  * [filebeat](https://www.elastic.co/guide/en/beats/filebeat/current/index.html) 
  * [packetbeat](https://www.elastic.co/guide/en/beats/packetbeat/current/index.html)
  * [![](https://images.microbadger.com/badges/image/khezen/httpbeat.svg) khezen/httpbeat](https://hub.docker.com/r/khezen/httpbeat/)
  * [![](https://images.microbadger.com/badges/image/khezen/execbeat.svg) khezen/execbeat](https://hub.docker.com/r/khezen/execbeat/)
* [![](https://images.microbadger.com/badges/image/khezen/elastalert.svg) khezen/elastalert](https://hub.docker.com/r/khezen/elastalert/)


# Setup

## Install Docker

1. [Docker engine](https://docs.docker.com/engine/installation/)
2. [Docker compose](https://docs.docker.com/compose/install/)
3. Clone this repository: `git clone https://github.com/khezen/docker-elk`

## [File Descriptors and MMap](https://www.elastic.co/guide/en/elasticsearch/guide/current/_file_descriptors_and_mmap.html) (Linux Only)

run the following command on your host:
```
sysctl -w vm.max_map_count=262144
```
You can set it permanently by modifying `vm.max_map_count` setting in your `/etc/sysctl.conf`.

# Usage

Start the Elastic Stack using *docker-compose*:

```bash
$ docker-compose up
```

You can also choose to run it in background:

```bash
$ docker-compose up -d
```

Now that the stack is running, you'll want to inject logs in it. The shipped logstash configuration allows you to send content via tcp or udp:

```bash
$ nc localhost 5000 < ./logstash-init.log
```

And then access Kibana by hitting [http://localhost:5601](http://localhost:5601) with a web browser.

*WARNING*: If you're using [*boot2docker*](http://boot2docker.io/), or [*Docker Toolbox*](https://www.docker.com/products/docker-toolbox) you must access it via the *boot2docker* IP address instead of *localhost*.

*NOTE*: You need to inject data into logstash before being able to create a logstash index in Kibana. Then all you should have to do is to hit the create button.

By Default, The Elastic Stack exposes the following ports:
* 5000: Logstash TCP input.
* 5001: Logstash UDP input.
* 9200: Elasticsearch HTTP
* 9300: Elasticsearch TCP transport
* 5601: Kibana


# Elasticsearch

Configuration file is located in `/etc/elasticsearch/elasticsearch.yml`.

You can find default config [there](https://github.com/Khezen/docker-elasticsearch/blob/master/config/elasticsearch.yml).

You can find help with elasticsearch configuration [there](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html).

You can edit `docker-compose.yml` to set [khezen/elasticsearch](https://github.com/Khezen/docker-elasticsearch) environment variables yourself.
```
elasticsearch:
    image: khezen/elasticsearch
    environment:
        HEAP_SIZE: 1g
        ELASTIC_PWD: changeme
        KIBANA_PWD: changeme
        LOGSTASH_PWD: changeme
        BEATS_PWD: changeme
        ELASTALERT_PWD: changeme
    volumes:
        - /data/elasticsearch:/usr/share/elasticsearch/data
        - /etc/elasticsearch:/usr/share/elasticsearch/config
    ports:
          - "9200:9200"
          - "9300:9300"
    networks:
        - elk
    restart: unless-stopped
```

# Kibana

* [Discover](https://www.elastic.co/guide/en/kibana/current/discover.html) - explore your data,

* [Visualize](https://www.elastic.co/guide/en/kibana/current/visualize.html) - create visualizations of your data,
  * You can find exported visualizations under `./visualizations` folder,
  * To import them in Kibana, go to `Managment->Saved Objects` panel,

* [Dashboard](https://www.elastic.co/guide/en/kibana/current/dashboard.html) - displays a collection of saved visualizations,
  * You can find exported dashboards under `./dashboards` folder,
  * To import them in Kibana, go to `Managment->Saved Objects` panel,

* [Timelion](https://www.elastic.co/guide/en/kibana/current/timelion.html) - combine totally independent data sources within a single visualization.

Configuration file is located in `/etc/kibana/kibana.yml`.

You can find default config [there](https://github.com/Khezen/docker-kibana/blob/master/config/default.yml).

You can find help with kibana configuration [there](https://www.elastic.co/guide/en/kibana/current/settings.html).

You can edit `docker-compose.yml` to set [khezen/kibana](https://github.com/Khezen/docker-kibana) environment variables yourself.
```
kibana:
    links:
        - elasticsearch
    image: khezen/kibana
    environment:
        KIBANA_PWD: changeme
        ELASTICSEARCH_HOST: elasticsearch
        ELASTICSEARCH_PORT: 9200
    volumes:
        - /etc/kibana:/etc/kibana
    ports:
          - "5601:5601"
    networks:
        - elk
    restart: unless-stopped
```

# logstash

Configuration file is located in `/etc/logstash/logstash.conf`.

You can find default config [there](https://github.com/Khezen/docker-logstash/blob/master/config/logstash.conf).

*NOTE*: It is possible to use [environment variables in logstash.conf](https://www.elastic.co/guide/en/logstash/current/environment-variables.html).

You can find help with logstash configuration [there](https://www.elastic.co/guide/en/logstash/current/configuration.html).

You can edit `docker-compose.yml` to set [khezen/logstash](https://github.com/Khezen/docker-logstash) environment variables yourself.
```
logstash:
    links:
        - elasticsearch
    image: khezen/logstash
    environment:
        HEAP_SIZE: 1g
        LOGSTASH_PWD: changeme
        ELASTICSEARCH_HOST: elasticsearch
        ELASTICSEARCH_PORT: 9200    
    volumes:
        - /etc/logstash:/etc/logstash/conf.d
    ports:
          - "5000:5000"
          - "5001:5001"
    networks:
        - elk
    restart: unless-stopped
```

# Beats

 The [Beats](https://www.elastic.co/guide/en/beats/libbeat/current/beats-reference.html) are open source data shippers that you install as agents on your servers to send different types of operational data to Elasticsearch


## any beat

You need to provide elasticsearch `host:port` and credentials for `beats` user in the configuration file:
```
output.elasticsearch:
  hosts: ["<ELASTICSEARCH_HOST>:<ELASTICSEARCH_PORT>"]
  index: "packetbeat"
  user: beats
  password: <BEATS_PWD>

```

## metricbeat

You can find help with metricbeat installation [here](https://www.elastic.co/guide/en/beats/metricbeat/5.0/metricbeat-installation.html). 

Configuration file is located in `/etc/metricbeat/metricbeat.yml`.

You can find help with metricbeat configuration [here](https://www.elastic.co/guide/en/beats/metricbeat/5.0/metricbeat-configuration.html).

## filebeat

You can find help with filebeat installation [here](https://www.elastic.co/guide/en/beats/filebeat/5.0/filebeat-installation.html).

Configuration file is located in `/etc/filebeat/filebeat.yml`.

You can find help with filebeat configuration [here](https://www.elastic.co/guide/en/beats/filebeat/5.0/filebeat-configuration.html).

## packetbeat

You can find help with packetbeat installation [here](https://www.elastic.co/guide/en/beats/packetbeat/5.0/packetbeat-installation.html).

Configuration file is located in `/etc/packetbeat/packetbeat.yml`.

You can find help with packetbeat configuration [here](https://www.elastic.co/guide/en/beats/packetbeat/5.0/filebeat-configuration.html).

## httpbeat

installation:

0. make sure you already started the elastic stack once using `./docker-compose.yml`
    * This compose define a network declared as external network in `./beats/httpbeat/docker-compose.yml`
1. go to `./beats/httpbeat/`
2. execute `docker-compose up -d`

Configuration file is located in `/etc/httpbeat/httpbeat.yml`.

You can find help with httpbeat configuration [here](https://github.com/christiangalsterer/httpbeat/blob/master/docs/configuration.asciidoc).

## execbeat

installation:

0. make sure you already started the elastic stack once using `./docker-compose.yml`
    * This compose define a network declared as external network in `./beats/execbeat/docker-compose.yml`
1. go to `./beats/execbeat/`
2. execute `docker-compose up -d`

Configuration file is located in `/etc/execbeat/execbeat.yml`.

You can find help with execbeat configuration [here](https://github.com/christiangalsterer/execbeat/blob/master/docs/configuration.asciidoc).

You can share scripts from your host to the container by adding them to `/usr/share/execbeat/scripts`.

## Other beats

The open source community has been hard at work developing new Beats. You can check out some of them [here](https://www.elastic.co/guide/en/beats/libbeat/current/community-beats.html).

# Elastalert

### What is Elastalert?
[ElastAlert](https://github.com/Yelp/elastalert) is a simple framework for alerting on anomalies, spikes, or other patterns of interest from data in Elasticsearch.
It is a nice replacement of the [Watcher](https://www.elastic.co/guide/en/x-pack/current/xpack-alerting.html#xpack-alerting) module if your are not willing to pay the x-pack subscription and still needs some alerting features.

## Installation
0. make sure you already started the elastic stack once using `./docker-compose.yml`
    * This compose define a network declared as external network in `./alerts/docker-compose.yml`
1. go to `./alerts/`
2. execute `docker-compose up -d`

## Configuration
Configuration file is located in `/etc/elastalert/elastalert.yml`.

You can find help with elastalert configuration [here](https://elastalert.readthedocs.io/en/latest/index.html).

You can share rules from host to the container by adding them to `/usr/share/elastalert/rules`


# User Feedback
## Issues
If you have any problems with or questions about this project, please ask for help through a [GitHub issue](https://github.com/Khezen/docker-elk/issues).