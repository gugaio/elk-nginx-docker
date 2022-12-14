# Nginx monitored by ELK stack

## TL;DR;
```bash
git clone https://github.com/gugaio/elk-nginx-docker.git
cd elk-nginx-docker
docker-compose up
//Kibana will be available at http://localhost:8020
./create-nginx-index-at-kibana.sh
```


# Intro
This small project contains all you need to execute a **docker compose up** command and get a Nginx up, running and monitored by a ELK stack. 
It allows you to analyse any log data sent by nginx using Kibana.
You DONT need this README to run the project, only **docker compose up** is enough. 
But here I'll show you the key points of solution if you need understand each item.

# Diagram
![alt text](./images/diagram.png)

# Follow I will provide a shortly explanation about each component of project:

- 1. Elastic Search
- 2. Logstah
- 3. Kibana
- 4. Nginx
- 5. Docker

## 1. Elastic Search
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


## 2. Logstach
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

## 3. Kibana
### *./kibana/Dockerfile*
This file only specify a Docker image to build the container.
```dockerfile
FROM docker.elastic.co/kibana/kibana-oss:6.6.0
```
### *./kibana/config/kibana.yml*
```yaml
## Default Kibana configuration from kibana-docker.
server.name: kibana
server.host: "0"
# Elastic Search API port
elasticsearch.url: http://elasticsearch:9200
```

## 4. Nginx
### *./nginx/Dockerfile*
Nginx Dockerfile copy two config files.
* nginx.conf: where I configure to send access log data to logstash.
* reverse-proxy.conf: where I configure nginx as a reverse proxy forwarding requests from 8020 port to kibana address.
```dockerfile
FROM nginx:latest
COPY nginx.conf /etc/nginx/nginx.conf
COPY reverse-proxy.conf /etc/nginx/conf.d/reverse-proxy.conf
EXPOSE 8020
EXPOSE 80
STOPSIGNAL SIGTERM
CMD ["nginx", "-g", "daemon off;"]
```

### *./nginx/nginx.config*
Main section of this file is where I define the access log format and how to send to logstash.
Below a highlight it.
```yaml
#################
#### Here is where we define the Log format and how to send to LogStach
#################
log_format custom '$remote_addr - $remote_user [$time_local]'
              '"$request" $status $body_bytes_sent'
              '"$http_referer" "$http_user_agent"'
              '"$request_time" "$upstream_connect_time"';
access_log /var/log/nginx/access.log custom;
access_log syslog:server=logstash:1025 custom;
error_log /var/log/nginx/error.log;
```

### *./nginx/reverse-proxy.config*
```yaml
server {
    listen 8020; #listem from 8020
    server_name elk;

    location / {
        proxy_pass http://kibana:5601; #and redirect to kibana:5601
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

![alt text](./images/docker_elastic.png)

The key points of Elastic Search configuration here are:
- "build/context" set the Elastic Dockerfile directory path.
- "volumes" maps the elastic.yml (to configure Elastic Search) and an writable directory to "data". 
- "ports" maps port 9200 (API Elastic default port). Also maps 9300 but it is only used for clusterization.
- "enviroments/ES_JAVA_OPTS" contains JVM flags to incresate memory size.
- "network" indicates the private network which all components to comunicate each other.


