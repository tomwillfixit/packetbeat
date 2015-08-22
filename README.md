### Log container data packets to Elasticsearch/Kibana

Based on article : http://agonzalezro.github.io/log-your-docker-containers-from-a-container-with-packetbeat.html

The Elasticsearch and Kibana containers should run in a central location and treated as production containers.
The packetbeat containers would be run on all docker agents/build agents.  The packetbeat containers log directly to
Elasticsearch and store logs locally under /tmp/packetbeat.
 
### Cheat Sheet :

#### Start ElasticSearch Container
```
docker run -d -p 9200:9200 -p 9300:9300 --name elasticsearch elasticsearch:latest
```

#### Start Kibana Container
```
docker run -d -p 5601:5601 --name kibana --link elasticsearch:elasticsearch -e ELASTICSEARCH_URL=http://elasticsearch:9200 kibana:latest
```

#### Build Packetbeat image 

The configuration for packetbeat is in the packetbeat.yml file.

```
docker build --pull -t logging/packetbeat .
```

#### Start Packetbeat container
```
docker run -d --restart=always --net=host --name pb logging/packetbeat
```

#### Check everything is running as expected :
```
docker ps

Sample output

CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                                            NAMES
7bf1b8d6676e        logging/packetbeat     "./packetbeat -e -c=p"   10 seconds ago      Up 9 seconds                                                         pb
4e47f7fe8fd7        kibana:latest          "/docker-entrypoint.s"   35 seconds ago      Up 34 seconds       0.0.0.0:5601->5601/tcp                           kibana
41160a58deab        elasticsearch:latest   "/docker-entrypoint.s"   46 seconds ago      Up 45 seconds       0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   elasticsearch

```

Open Browser : http://localhost:5601

Change index pattern from :

logstash-*

to

packetbeat-*

***Note if you get a message regarding no values just wait a minute or start a container and run some sort of command like apt-get install python to add some values to the packetbeat index


