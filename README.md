# elk-nginx-docker
Run a Nginx reverse proxy integrated with ELK stack all running with Docker and Docker Compose. 

It allows you to analyse any log data sent by nginx to Elastic Search using the visualization tools of Kibana.

The diagram below shows all components in this example.
![alt text](./images/diagram.png)
- User admin can access kibana by port locahost:8020. Nginx will redirect from 9020 to kibana port 5601.
- Nginx will send logs to logstabh by udp at port 1025.

## Elastic Search
![alt text](./images/elasticsearch.png)
