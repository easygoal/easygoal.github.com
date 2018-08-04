---
layout: post
category: tutorial
tags: [intro, tutorial]
ref: glances-influxdb-grafana
lang: en
---

# Glances Influxdb Grafana

## Influxdb

```sh
# start
sudo /etc/init.d/influxdb start

# check port 8086
curl -i 'http://127.0.0.1:8086'
```

## Grafana

```sh
# start
sudo /etc/init.d/grafana-server start

# check port 3000
curl -i 'http://127.0.0.1:3000'
```
Use the provided [dashboard](https://github.com/nicolargo/glances/blob/master/conf/glances-grafana.json) as the Grafana dashboard. 
