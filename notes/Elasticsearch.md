---
title: Elasticsearch
created: '2020-05-13T15:12:00.028Z'
modified: '2020-05-17T13:45:24.575Z'
---

# Elasticsearch


[Installare e configurare ES con Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)

[Swappiness (per regolare la frequenza di swap in Linux)](https://www.howtoforge.com/tutorial/linux-swappiness/): Cambiare da 60 (default) a 1. Per fare ciò in maniera permanente, crea un file di configurazione in `/etc/sysctl.d/swappinessConfig.conf` con all'interno `vm.swappiness=1`, riavviare e verificare il valore attuale con `cat /proc/sys/vm/swappiness`

#### [Installare l'indice shakespeare:](https://sundog-education.com/elasticsearch/)

```
wget http://media.sundog-soft.com/es7/shakes-mapping.json

curl -H 'Content-Type: application/json' -XPUT 127.0.0.1:9200/shakespeare 
--data-binary @shakes-mapping.json

wget http://media.sundog-soft.com/es7/shakespeare_7.0.json

curl -H 'Content-Type: application/json' -XPOST
'127.0.0.1:9200/shakespeare/_bulk?pretty' --data-binary
@shakespeare_7.0.json

curl -H 'Content-Type: application/json' -XGET
'127.0.0.1:9200/shakespeare/_search?pretty' -d '
{
"query" : {
"match_phrase" : {
"text_entry" : "to be or not to be"
}}}
```

!



