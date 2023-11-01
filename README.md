# docker-spark-hive
This repo is aim to provide a so called Spark on Hive environment for develop purpose.
> Spark on Hive is that Spark SQL integrated with Hive tables

This docker-compose is based on [big-data-europe/docker-spack](https://github.com/big-data-europe/docker-spark) with intergreted [apache/hive](https://hub.docker.com/r/apache/hive).

```
version: '3'
services:
  spark-master:
    image: bde2020/spark-master:3.3.0-hadoop3.3
    container_name: spark-master
    depends_on:
      - hive
    ports:
      - "8080:8080"
      - "7077:7077"
    networks:
      spark_net:
        ipv4_address: 172.28.1.1
    environment:
      - INIT_DAEMON_STEP=setup_spark
    volumes:
      - ./tmp/spark-events-local:/tmp/spark-events
      - ./spark/conf/hive-site.xml:/spark/conf/hive-site.xml
    extra_hosts:
      - "spark-master:172.28.1.1"
      - "spark-worker:172.28.1.2"
      - "spark-history-server:172.28.1.3"
      - "metastore:172.28.1.4"
      - "hive:172.28.1.5"
  spark-worker:
    image: bde2020/spark-worker:3.3.0-hadoop3.3
    container_name: spark-worker
    depends_on:
      - spark-master
    ports:
      - "8081:8081"
    networks:
      spark_net:
        ipv4_address: 172.28.1.2
    environment:
      - "SPARK_MASTER=spark://spark-master:7077"
    volumes:
      - ./tmp/spark-events-local:/tmp/spark-events
      - ./spark/conf/hive-site.xml:/spark/conf/hive-site.xml
    extra_hosts:
      - "spark-master:172.28.1.1"
      - "spark-worker:172.28.1.2"
      - "spark-history-server:172.28.1.3"
      - "metastore:172.28.1.4"
      - "hive:172.28.1.5"
  spark-history-server:
      image: bde2020/spark-history-server:3.3.0-hadoop3.3
      container_name: spark-history-server
      depends_on:
        - spark-master
      ports:
        - "18081:18081"
      networks:
        spark_net:
          ipv4_address: 172.28.1.3
      volumes:
        - ./tmp/spark-events-local:/tmp/spark-events
        - ./spark/conf/hive-site.xml:/spark/conf/hive-site.xml
      extra_hosts:
        - "spark-master:172.28.1.1"
        - "spark-worker:172.28.1.2"
        - "spark-history-server:172.28.1.3"
        - "metastore:172.28.1.4"
        - "hive:172.28.1.5"
  metastore:
      image: apache/hive:3.1.3
      container_name: metastore
      environment:
        - SERVICE_NAME=metastore
      networks:
        spark_net:
          ipv4_address: 172.28.1.4
      extra_hosts:
        - "spark-master:172.28.1.1"
        - "spark-worker:172.28.1.2"
        - "spark-history-server:172.28.1.3"
        - "metastore:172.28.1.4"
        - "hive:172.28.1.5"
  hive:
      image: apache/hive:3.1.3
      container_name: hive
      depends_on: 
        - metastore
      ports:
        - "10002:10002"
      networks:
        spark_net:
          ipv4_address: 172.28.1.5
      environment:
        - IS_RESUME="true"
        - SERVICE_NAME=hiveserver2
        - SERVICE_OPTS=-Dhive.metastore.uris=thrift://metastore:9083
      extra_hosts:
        - "spark-master:172.28.1.1"
        - "spark-worker:172.28.1.2"
        - "spark-history-server:172.28.1.3"
        - "metastore:172.28.1.4"
        - "hive:172.28.1.5"
networks:
  spark_net:
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16
```
## How to use
Just run the command below in the root of the repo, otherwise you will have to provide a hive-site.yaml to make it work.
```
docker-compose up -d
```
