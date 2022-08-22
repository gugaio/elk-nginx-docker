# Nginx monitored by ELK stack. All running as Docker containers.
This small project contains all you need to run a **docker compose up** command and have a Nginx ready monitored by a ELK stack. 
It allows you to analyse any log data sent by nginx to Elastic Search using the visualization tools of Kibana.
You DONT need this README to run the project, only **docker compose up** is enough. 
But here I'll show you the key points of solution if you need understand each item.

Below you see the project diagram.
![alt text](./images/diagram.png)
Shortly explanation:
- User access kibana by http://localhost:8020. NGINX will redirect it from 9020 to kibana default port 5601.
- Nginx will send logs to logstabh by udp at port 1025.
- Logstach will transform and send the logs to Elastic Search at API port 9200. Kibana also will call Elastic Search at port 9200 to get data.

## This README will follow these steps:
- 1. Elastic Search configuration YAML and Dockerfile
- 2. Logstah configuration YAML and Dockerfile
- 3. Kibana configuration YAML and Dockerfile
- 4. Nginx configuration files and Dockerfile
- 4. Docker Compose

## Elastic Search
The *./elasticsearch/* directory contains the follow files:
### *./elasticsearch/Dockerfile*
This file only specify a Docker image to build the container.
```dockerfile
FROM docker.elastic.co/elasticsearch/elasticsearch-oss:6.6.0
```

### *./elasticsearch/config/elasticsearch.yml*
```dockerfile
#Define a name for the cluster. Its not important to a development enviroment.
cluster.name: "docker-cluster"

#Specify the network interface which elasticsearch will bind to. Set 0.0.0.0 to bind to any one. 
network.host: 0.0.0.0

#Single-node as we dont need a multiple cluster at development enviroment
discovery.zen.minimum_master_nodes: 1
discovery.type: single-node
```


## Logstach
The *./logstash/* directory contains the follow files:
### *./logstash/Dockerfile*
This file only specify a Docker image to build the container.
```dockerfile
FROM docker.elastic.co/logstash/logstash-oss:6.6.0
```

### *./logstash/config/logstash.yml*
```yaml
#The bind address for the HTTP API endpoint.
http.host: "0.0.0.0"
#Path to Logstash main pipeline conf
path.config: /usr/share/logstash/pipeline
```

### *./logstash/pipeline/logstash.config*
This file specify the input port to receive Nginx log messages, all the filter to transform the raw log message, and the output destination (elastic search url). Details you can see inside the file.

## Kibana
Kibana directory has a similiar Dockerfile as showed at Elastc and Logstash topics. We also have a YAML where we configure the elastic search url.
### *./kibana/config/kibana.yml*
```yaml
## Default Kibana configuration from kibana-docker.
server.name: kibana
server.host: "0"
# Elastic Search API port
elasticsearch.url: http://elasticsearch:9200
```


![alt text](./images/docker_elastic.png)

The key points of Elastic Search configuration here are:
- "build/context" set the Elastic Dockerfile directory path.
- "volumes" maps the elastic.yml (to configure Elastic Search) and an writable directory to "data". 
- "ports" maps port 9200 (API Elastic default port). Also maps 9300 but it is only used for clusterization.
- "enviroments/ES_JAVA_OPTS" contains JVM flags to incresate memory size.
- "network" indicates the private network which all components to comunicate each other.


