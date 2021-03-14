---
layout: post
title: 'Using Elasticsearch with Docker'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Hosting software with Docker is a great way to setup your development environment. Using Docker to host a local instance of Elasticsearch is no different. Here are my notes on how to set things started.

<!--more-->

**NOTE**: A _HUGE_ thank you to the instructions here:

[https://dev.to/mandeepm91/setting-up-elasticsearch-and-kibana-on-docker-with-x-pack-security-enabled-48dm](https://dev.to/mandeepm91/setting-up-elasticsearch-and-kibana-on-docker-with-x-pack-security-enabled-48dm)

This saved me hours of work.

### Using Docker Compose

I decided to use a docker compose file to run Elasticsearch. This is not required, but it made things easier. Here is the file I started with:

```yaml
version: '2.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.9.3
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - esnet
volumes:
  esdata:
    driver: local
networks:
  esnet:
```

Store this in a `docker-compose.yml` file and then run the command:

```powershell
C:\es-docker> docker-compose up
```

You can confirm things are working correctly by opening a web browser to `http://localhost:9200`. You should get a JSON document that looks something like this:

```json
{
    "name": "eb62a4930732",
    "cluster_name": "docker-cluster",
    "cluster_uuid": "YU1k8piTS068nMXA9I7NHg",
    "version": {
        "number": "7.9.3",
        "build_flavor": "default",
        "build_type": "docker",
        "build_hash": "c4138e51121ef06a6404866cddc601906fe5c868",
        "build_date": "2020-10-16T10:36:16.141335Z",
        "build_snapshot": false,
        "lucene_version": "8.6.2",
        "minimum_wire_compatibility_version": "6.8.0",
        "minimum_index_compatibility_version": "6.0.0-beta1"
    },
    "tagline": "You Know, for Search"
}
```

But there are two additional things I want to achieve. First, I want to enable authentication in order to pass a username/password with each request. This will more closely simulate the shared hosted environment used in our testing and production environments. And second, I want to place the data files in a folder outside of Docker. The files are currently stored in the Docker volumes which is fine, but I want to easily check the number of files and size on disk for performance reasons. 

### Enabling Security

So this turned out to be relatively straightforward. But it took a long time to find any instructions on how to actually achieve it. 

To enable security with the Docker image you need to set an environment variable in the `elasticsearch.yml` file. This file is found within the container. To view it's contents, open a bash session within the container:

```powershell
docker-compose exec elasticsearch bash
```

Then view the contents of the file:

```bash
[root@f1da5d0599a2 elasticsearch]# cat config/elasticsearch.yml

cluster.name: "docker-cluster"
network.host: 0.0.0.0
```

To enable security, you need to add a new line with the following: `xpack.security.enabled: true`. But editing this file within the container is not a good idea, since it will be destroyed when the container is destroyed. To use a file persisted outside of the container, create a `elasticsearch.yml` on your local file system. First create a folder to store the file:

```powershell
mkdir docker-volume
mkdir docker-volume\config
```

Save the following contents to the `docker-volume\config\elasticsearch.yml` file:

```yaml
cluster.name: "docker-cluster"
network.host: 0.0.0.0
xpack.security.enabled: true
```

Modify the `docker-compose.yml` file to use the new file by adding the following line:

```yaml
version: '2.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.9.3
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata:/usr/share/elasticsearch/data
      - ./docker-volume/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    ports:
      - "9200:9200"
    networks:
      - esnet
volumes:
  esdata:
    driver: local
networks:
  esnet:
```

With security now enabled, go ahead and run `docker-compose up` again. When attempting to access Elasticsearch, you will encounter an error:

```json
{
  "error": {
    "root_cause": [
      {
        "type": "security_exception",
        "reason": "missing authentication credentials for REST request [/]",
        "header": {
          "WWW-Authenticate": "..."
        }
      }
    ],
    "status": 401
  }
}
```

Now we need to set the passwords.

### Setting Passwords

**NOTE:** Don't forget to turn Elasticsearch off: `docker-compose down`. 

Now create a new local folder to store the data files:

```powershell
mkdir docker-volume\data
```

And modify the `docker-compose.yml` to use the new volume:

```yml
version: '2.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.9.3
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./docker-volume/data:/usr/share/elasticsearch/data
      - ./docker-volume/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    ports:
      - "9200:9200"
    networks:
      - esnet
volumes:
  esdata:
    driver: local
networks:
  esnet:
```

And you should be able to run it without errors: `docker-compose up`

Now we need to set the passwords. For that, we need to run a command from within the container.

```powershell
docker-compose exec elasticsearch bash

[root@a7861f7ea9b2 elasticsearch]# bin/elasticsearch-setup-passwords auto
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,kibana_system,logstash_system,beats_system,remote_monitoring_user.
The passwords will be randomly generated and printed to the console.
Please confirm that you would like to continue [y/N]y

...

Changed password for user elastic
PASSWORD elastic = {new password here}

[root@a7861f7ea9b2 elasticsearch]#
```

Write down the value for `{new password here}`. It will be the new password for you to access Elasticsearch (the username is `elastic`).

You can provide the credentials interactively in a web browser or via `curl`:

```powershell
curl -XGET -u elastic:{new password here} http://localhost:9200/
```

### Summary

Thankfully I found an article to follow that explained how to get past some of the issues I was facing. Setting up Docker with Elasticsearch was not that difficult, but you had to know a few tricks to make it work. 
