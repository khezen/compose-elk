# What is the Elastic Stack?
By combining the massively popular Elasticsearch, Logstash, and Kibana, Elastic has created an end-to-end stack that delivers actionable insights in real time from almost any type of structured and unstructured data source. Built and supported by the engineers behind each of these open source products, the Elastic Stack makes searching and analyzing data easier than ever before.

[<img src="https://www.elastic.co/assets/blte1ccb52ef00ca60e/icon-elastic-stack-bb.svg" width="144" height="144">](https://www.elastic.co/)


# Based on:

* [<img src="https://static-www.elastic.co/fr/assets/blt282ae2420e32fc38/icon-kibana-bb.svg?q=802" width="50" height="50">](https://www.elastic.co/fr/products/kibana) [![](https://images.microbadger.com/badges/image/khezen/kibana.svg)](https://hub.docker.com/r/khezen/kibana/) [khezen/kibana](https://github.com/Khezen/docker-kibana)
* [<img src="https://static-www.elastic.co/fr/assets/blt9a26f88bfbd20eb5/icon-elasticsearch-bb.svg?q=802" width="50" height="50">](https://www.elastic.co/fr/products/elasticsearch) [![](https://images.microbadger.com/badges/image/khezen/elasticsearch.svg)](https://hub.docker.com/r/khezen/elasticsearch/) [khezen/elasticsearch](https://github.com/Khezen/docker-elasticsearch)
* [<img src="https://static-www.elastic.co/fr/assets/blt946bc636d34a70eb/icon-logstash-bb.svg?q=600" width="50" height="50">](https://www.elastic.co/fr/products/logstash) [![](https://images.microbadger.com/badges/image/khezen/logstash.svg)](https://hub.docker.com/r/khezen/logstash/) [khezen/logstash](https://github.com/Khezen/docker-logstash)
* [<img src="https://static-www.elastic.co/assets/blt121ead33d4ed1f55/icon-beats-bb.svg?q=600" width="50" height="50">](https://www.elastic.co/products/beats) [Beats](https://www.elastic.co/guide/en/beats/libbeat/current/installing-beats.html)


# Setup

## Install Docker

1. [Docker engine](https://docs.docker.com/engine/installation/)
2. [Docker compose](https://docs.docker.com/compose/install/)
3. Clone this repository: `git clone https://github.com/khezen/docker-elk`

## [File Descriptors and MMap](https://www.elastic.co/guide/en/elasticsearch/guide/current/_file_descriptors_and_mmap.html)

run the following command on your host:
```
sysctl -w vm.max_map_count=262144
```
You can set it permanently by modifying `vm.max_map_count` setting in your `/etc/sysctl.conf`.

# Usage

Start the ELK stack using *docker-compose*:

```bash
$ docker-compose up
```

You can also choose to run it in background:

```bash
$ docker-compose up -d
```

Now that the stack is running, you'll want to inject logs in it. The shipped logstash configuration allows you to send content via tcp or udp:

```bash
$ nc localhost 5000 < ./init.log
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

# Configuration

## Elasticsearch

Configuration file is editable from `/srv/elasticsearch/conf.d/elasticsearch.yml`.

You can find default config [there](https://github.com/Khezen/docker-elasticsearch/blob/master/config/elasticsearch.yml).

You can find help with elasticsearch configuration [there](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html).

You can edit `docker-compose.yml` to set [khezen/elasticsearch](https://github.com/Khezen/docker-elasticsearch) environment variables yourself.
```
elasticsearch:
    image: khezen/elasticsearch
    environment:
        heap_size: 1g
        elastic_pwd: changeme
        kibana_pwd: changeme
        logstash_pwd: changeme
        beats_pwd: changeme
    volumes:
        - /srv/elasticsearch/data:/data
        - /srv/elasticsearch/config:/etc/elasticsearch/config/
    ports:
          - "9200:9200"
          - "9300:9300"
    networks:
        - elk
    restart: always
```

## Kibana

* [Discover](https://www.elastic.co/guide/en/kibana/current/discover.html) - explore your data,
* [Visualize](https://www.elastic.co/guide/en/kibana/current/visualize.html) - create visualizations of your data,
* [Dashboard](https://www.elastic.co/guide/en/kibana/current/dashboard.html) - displays a collection of saved visualizations,
* [Timelion](https://www.elastic.co/guide/en/kibana/current/timelion.html) - combine totally independent data sources within a single visualization.

Configuration file is editable from `/srv/kibana/kibana.yml`.

You can find default config [there](https://github.com/Khezen/docker-kibana/blob/master/config/default.yml).

You can find help with kibana configuration [there](https://www.elastic.co/guide/en/kibana/current/settings.html).

You can edit `docker-compose.yml` to set [khezen/kibana](https://github.com/Khezen/docker-kibana) environment variables yourself.
```
kibana:
    links:
        - elasticsearch
    image: khezen/kibana
    environment:
        kibana_pwd: changeme
        elasticsearch_host: elasticsearch
        elasticsearch_port: 9200
    volumes:
        - /srv/kibana:/etc/kibana
    ports:
          - "5601:5601"
    networks:
        - elk
    restart: always
```

## logstash

Configuration file is editable from `/srv/logstash/conf.d/logstash.conf`.

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
        heap_size: 1g
        logstash_pwd: changeme
        elasticsearch_host: elasticsearch
        elasticsearch_port: 9200    
    volumes:
        - /srv/logstash/conf.d:/etc/logstash/conf.d
    ports:
          - "5000:5000"
          - "5001:5001"
    networks:
        - elk
    restart: always
```

## Beats

* [Overview](https://www.elastic.co/guide/en/beats/libbeat/current/beats-reference.html),
* [Non offical beats](https://www.elastic.co/guide/en/beats/libbeat/current/community-beats.html),
* [installation](https://www.elastic.co/guide/en/beats/libbeat/current/installing-beats.html).

### Configuration

You need to provide elasticsearch `host:port` and credentials for `beats` user:
```
output.elasticsearch:
  hosts: ["<elasticsearch_host>:<elasticsearch_port>"]
  index: "packetbeat"
  user: beats
  password: <beats_pwd>

```
Configuration file is located in `/etc/packetbeat/packetbeat.yml` if your are dealing with `packetbeat` for example.

## X-pack

### Watcher
[Alerting on Cluster and Index Events](https://www.elastic.co/guide/en/x-pack/current/xpack-alerting.html#xpack-alerting).

[This getting started guide](https://www.elastic.co/guide/en/x-pack/current/watcher-getting-started.html) walks you through setting up alerting and creating your first watches, and introduces the building blocks you’ll use to create custom watches.

### Reporter

[Reporting from Kibana](https://www.elastic.co/guide/en/x-pack/current/xpack-reporting.html).

[This getting started guide](https://www.elastic.co/guide/en/x-pack/current/reporting-getting-started.html) walks you through setting up reporting and creating your first automated reports.

### Graph

[Graphing connections in your data](https://www.elastic.co/guide/en/x-pack/current/xpack-graph.html).