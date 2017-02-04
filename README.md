# What is the Elastic Stack?
By combining the massively popular Elasticsearch, Logstash, and Kibana, Elastic has created an end-to-end stack that delivers actionable insights in real time from almost any type of structured and unstructured data source. Built and supported by the engineers behind each of these open source products, the Elastic Stack makes searching and analyzing data easier than ever before.

[<img src="https://www.elastic.co/assets/blte1ccb52ef00ca60e/icon-elastic-stack-bb.svg" width="144" height="144">](https://www.elastic.co/)


# Based on:

* [<img src="https://static-www.elastic.co/fr/assets/blt282ae2420e32fc38/icon-kibana-bb.svg?q=802" width="50" height="50">](https://www.elastic.co/fr/products/kibana) [![](https://images.microbadger.com/badges/image/khezen/kibana.svg)](https://hub.docker.com/r/khezen/kibana/) [khezen/kibana](https://github.com/Khezen/docker-kibana)
* [<img src="https://static-www.elastic.co/fr/assets/blt9a26f88bfbd20eb5/icon-elasticsearch-bb.svg?q=802" width="50" height="50">](https://www.elastic.co/fr/products/elasticsearch) [![](https://images.microbadger.com/badges/image/khezen/elasticsearch.svg)](https://hub.docker.com/r/khezen/elasticsearch/) [khezen/elasticsearch](https://github.com/Khezen/docker-elasticsearch)
* [<img src="https://static-www.elastic.co/fr/assets/blt946bc636d34a70eb/icon-logstash-bb.svg?q=600" width="50" height="50">](https://www.elastic.co/fr/products/logstash) [![](https://images.microbadger.com/badges/image/khezen/logstash.svg)](https://hub.docker.com/r/khezen/logstash/) [khezen/logstash](https://github.com/Khezen/docker-logstash)
* [<img src="https://static-www.elastic.co/assets/blt121ead33d4ed1f55/icon-beats-bb.svg?q=600" width="50" height="50">](https://www.elastic.co/products/beats) [Beats](https://www.elastic.co/guide/en/beats/libbeat/current/installing-beats.html)
  * [topbeat](https://www.elastic.co/guide/en/beats/topbeat/current/index.html)
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
* 9200: Elasticsearch HTTP
* 9300: Elasticsearch TCP transport
* 5601: Kibana

# Docker Swarm

Deploy the Elastic Stack on your cluster using docker swarm:

1. Connect to a manager node of the swarm
2. `git clone https://github.com/khezen/docker-elk`
3. `cd docker-elk`
4. `git checkout 2`
5. `docker stack deploy -c swarm-stack.yml elk`

The number of replicas for each services can be edited from `swarm-stack.yml`:
```
...
deploy:
      mode: replicated
      replicas: 2
...
```

Services are load balanced using **HAProxy**.


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
  * To import them in Kibana, go to `Settings->Objects` panel,

* [Dashboard](https://www.elastic.co/guide/en/kibana/current/dashboard.html) - displays a collection of saved visualizations,
  * You can find exported dashboards under `./dashboards` folder,
  * To import them in Kibana, go to `Settings->Objects` panel,

Configuration file is located in `/etc/libana/kibana.yml`.

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
        - /etc/elasticsearch/searchguard/ssl:/etc/elasticsearch/searchguard/ssl
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
        - /etc/elasticsearch/searchguard/ssl:/etc/elasticsearch/searchguard/ssl
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
  #user: beats
  #password: <BEATS_PWD>

```

## topbeat

You can find help with topbeat installation [here](https://www.elastic.co/guide/en/beats/topbeat/current/topbeat-installation.html).

Configuration file is located in `/etc/topbeat/topbeat.yml`.

You can find help with topbeat configuration [here](https://www.elastic.co/guide/en/beats/topbeat/current/topbeat-configuration.html).

start with `sudo /etc/init.d/topbeat start`


## packetbeat

You can find help with packetbeat installation [here](https://www.elastic.co/guide/en/beats/packetbeat/1.3/packetbeat-installation.html).

Configuration file is located in `/etc/packetbeat/packetbeat.yml`.

You can find help with packetbeat configuration [here](https://www.elastic.co/guide/en/beats/packetbeat/1.3/configuring-packetbeat.html).

start with `sudo /etc/init.d/packetbeat start`

# Elastalert

### What is Elastalert?
[ElastAlert](https://github.com/Yelp/elastalert) is a simple framework for alerting on anomalies, spikes, or other patterns of interest from data in Elasticsearch.
It is a nice replacement of the [Watcher](https://www.elastic.co/guide/en/x-pack/current/xpack-alerting.html#xpack-alerting) module if your are not willing to pay the x-pack subscription and still needs some alerting features.

## Configuration
Configuration file is located in `/etc/elastalert/elastalert.yml`.

You can find help with elastalert configuration [here](https://elastalert.readthedocs.io/en/latest/index.html).

You can share rules from host to the container by adding them to `/var/lib/docker/volumes/elastalert_rules/`


# User Feedback
## Issues
If you have any problems with or questions about this project, please ask for help through a [GitHub issue](https://github.com/Khezen/docker-elk/issues).
