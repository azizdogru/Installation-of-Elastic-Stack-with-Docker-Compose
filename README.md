*** Installation of Elastic Stack with Docker Compose


The Elastic Stack family, also known as ELK Stack, combines the tools Elasticsearch, Logstash, and Kibana to provide us with a toolkit for log collection, searching, and analysis within our environment.

In brief, Elasticsearch is an open-source software built on top of the Lucene infrastructure, offering fast and scalable capabilities for querying content within a specific dataset, allowing for detailed analysis. It takes on the role of a search engine.

Logstash, on the other hand, is a server-side process management tool used for log management. It is used for indexing received logs and utilizes Elasticsearch as the backend solution.

Kibana is a data visualization platform that focuses on presenting data in visual formats. It allows us to transform JSON data into various charts or visual analyses, a process referred to as data visualization.

Let's examine how Elastic Stack is installed with Docker. 

We will raise the Elastic Stack we will install using a Docker Compose file and observe how we can achieve results.

First, we add a directory named 'elasticsearch' to the working directory and create an 'elasticsearch.yml' file under it.

- mkdir elasticsearch; nano elasticsearch/elasticsearch.yml

cluster.name: "docker-cluster"
network.host: 0.0.0.0
discovery.zen.minimum_master_nodes: 1
discovery.type: single-node
xpack.security.enabled: false


As can be easily understood, the first two definitions are set as the name of the cluster we will listen to and 0.0.0.0 to obtain information from any IP. We include the single-node configuration to avoid bootstrap checks. We turn off the X-Pack security option to enable access to Elasticsearch without logging in.

- mkdir kibana; nano kibana/kibana.yml

server.name: kibana
server.host: "0"
elasticsearch.url: http://elasticsearch:9200


After specifying the server's name, its host, and the URL for Kibana to access Elasticsearch in the Stack we will create, we proceed to create a small configuration file for Logstash as well.

- mkdir logstash; nano logstash/logstash.conf

input {
        tcp {
                port => 5000
        }
}
output {
        elasticsearch {
                hosts => "elasticsearch:9200"
        }
}

In this file, we add the TCP port that Logstash will communicate on and the search engine it will perform searches on as a URL to the output statement. After preparing these configurations, we can start creating our docker-compose.yml file.

- nano docker-compose.yml

version: '3'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.0.1
    volumes:
      - ./elasticsearch/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      ES_JAVA_OPTS: "-Xmx256m -Xms256m"
  logstash:
    image: docker.elastic.co/logstash/logstash:6.0.1
    command: -f /etc/logstash/conf.d/
    volumes:
      - ./logstash/:/etc/logstash/conf.d/
    ports:
      - "5000:5000"
    environment:
      LS_JAVA_OPTS: "-Xmx256m -Xms256m"
    depends_on:
      - elasticsearch
  kibana:
    image: docker.elastic.co/kibana/kibana:6.0.1
    volumes:
      - ./kibana/:/usr/share/kibana/config/
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch


The docker-compose.yml file we have prepared consists of three services: elasticsearch, logstash, and kibana. Since the elasticsearch service will be the first to run, we added 'depend_on' statements with elasticsearch under the kibana and logstash services. This way, the other services won't start before the elasticsearch service. We connect the configuration files we created for each service as docker volumes associated with the services.

The 'ES_JAVA_OPTS' variable represents the portion of Java dedicated to the heap when the services are ready. By default, if we don't use this variable, 1 GB of heap size is allocated for Java. For exposing the ports to be used, we arrange the ports for elasticsearch as 9200 and 9300 on the host side. Port 9200 is used to communicate with the RESTful API in another language, while port 9300 is necessary for communication among nodes in the cluster.

After exposing ports 5000 and 5061 for Logstash and Kibana, respectively, our docker-compose.yml file is ready.

- docker-compose up -d

To run the services we've configured, we use the command docker-compose up. If we want to see the individual terminal outputs of each service along with the docker compose command we're running, we simply remove the '-d' parameter. Additionally, to have a clearer view of which services are running, you can use the command:

- docker ps

Our services will be communicating with each other over the configured ports. To access the Kibana web interface, use:

http://localhost:5601

Since there is no data set or application in the stack we've set up, Kibana will ask for an index pattern variable to filter.

The Elastic Stack (formerly known as the ELK Stack) is now ready for use. To interact with the desired application and perform filtering, it's sufficient to define the 'index pattern' variable.