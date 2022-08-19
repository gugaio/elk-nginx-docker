# elk-nginx-docker
Run a Nginx reverse proxy integrated with ELK stack all running with Docker and Docker Compose. 
It allows you to analyse any log data sent by nginx to Elastic Search using the visualization tools of Kibana.
You can check all components in this example at diagram below.
![alt text](./images/diagram.png)
- User admin can access kibana by port locahost:8020. Nginx will redirect from 9020 to kibana port 5601.
- Nginx will send logs to logstabh by udp at port 1025.

## Elastic Search
![alt text](./images/elastic_search.png)

The key points of Elastic Search configuration here are:
- "build/context" set the Elastic Dockerfile directory path.
- "volumes" maps the elastic.yml (to configure Elastic Search) and an writable directory to "data". 
- "ports" maps port 9200 (API Elastic default port). Also maps 9300 but it is only used for clusterization.
- "enviroments/ES_JAVA_OPTS" contains JVM flags to incresate memory size.
- "network" indicates the private network which all components to comunicate each other.
