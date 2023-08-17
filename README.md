# Installation of Elastic Stack with Docker Compose

The Elastic Stack family, also known as ELK Stack, combines the tools Elasticsearch, Logstash, and Kibana to provide us with a toolkit for log collection, searching, and analysis within our environment.

In brief, Elasticsearch is an open-source software built on top of the Lucene infrastructure, offering fast and scalable capabilities for querying content within a specific dataset, allowing for detailed analysis. It takes on the role of a search engine.

Logstash, on the other hand, is a server-side process management tool used for log management. It is used for indexing received logs and utilizes Elasticsearch as the backend solution.

Kibana is a data visualization platform that focuses on presenting data in visual formats. It allows us to transform JSON data into various charts or visual analyses, a process referred to as data visualization.

## Setup

1. Create a directory named 'elasticsearch' to house Elasticsearch configuration:

    ```sh
    mkdir elasticsearch; nano elasticsearch/elasticsearch.yml
    ```

    Add the following configuration to `elasticsearch/elasticsearch.yml`:

    ```yaml
    cluster.name: "docker-cluster"
    network.host: 0.0.0.0
    discovery.zen.minimum_master_nodes: 1
    discovery.type: single-node
    xpack.security.enabled: false
    ```

2. Create a directory named 'kibana' and configure Kibana:

    ```sh
    mkdir kibana; nano kibana/kibana.yml
    ```

    Add the following configuration to `kibana/kibana.yml`:

    ```yaml
    server.name: kibana
    server.host: "0"
    elasticsearch.url: http://elasticsearch:9200
    ```

3. Create a directory named 'logstash' and configure Logstash:

    ```sh
    mkdir logstash; nano logstash/logstash.conf
    ```

    Add the following configuration to `logstash/logstash.conf`:

    ```conf
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
    ```

4. Create a `docker-compose.yml` file:

    ```sh
    nano docker-compose.yml
    ```

    Add the following configuration to `docker-compose.yml`:

    ```yaml
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
    ```

## Running the Services

To start the services, use the following command:

```sh
docker-compose up -d

To view the terminal outputs of each service, omit the '-d' parameter.

Accessing Kibana
The services will communicate over the configured ports. Access the Kibana web interface by visiting:

http://localhost:5601

As no data or application is present in the stack, Kibana will prompt you to set an index pattern for filtering.

The Elastic Stack (formerly known as the ELK Stack) is now ready for use. To interact with your desired application and perform filtering, define the 'index pattern' variable.
