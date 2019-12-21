https://ksqldb.io/quickstart.html

1、安装kafka本身
0.11的即可；

===================================================================

2、参考quickstart
https://ksqldb.io/quickstart.html

===================================================================
3、建立docker-compose.yml

因为我的本机有kafka和zookeeper在运行，所以我修改了官网的例子，将zk的端口号修改到了22181上去；


---
version: '2'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:5.3.1
    hostname: zookeeper
    container_name: zookeeper
    ports:
      - "22181:22181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 22181
      ZOOKEEPER_TICK_TIME: 2000

  broker:
    image: confluentinc/cp-enterprise-kafka:5.3.1
    hostname: broker
    container_name: broker
    depends_on:
      - zookeeper
    ports:
      - "29092:29092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: 'zookeeper:22181'
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://broker:9092,PLAINTEXT_HOST://localhost:29092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0

  ksqldb-server:
    image: confluentinc/ksqldb-server:0.6.0
    hostname: ksqldb-server
    container_name: ksqldb-server
    depends_on:
      - broker
    ports:
      - "8088:8088"
    environment:
      KSQL_LISTENERS: http://0.0.0.0:8088
      KSQL_BOOTSTRAP_SERVERS: broker:9092
      KSQL_KSQL_LOGGING_PROCESSING_STREAM_AUTO_CREATE: "true"
      KSQL_KSQL_LOGGING_PROCESSING_TOPIC_AUTO_CREATE: "true"

  ksqldb-cli:
    image: confluentinc/ksqldb-cli:0.6.0
    container_name: ksqldb-cli
    depends_on:
      - broker
      - ksqldb-server
    entrypoint: /bin/sh
    tty: true

===================================================================

4、安装好docker，略过
mac还是比较好装的

===================================================================
5、连上vpn

===================================================================
6、运行
docker-compose up


qldb-server    | 	ksql.udf.collect.metrics = false
ksqldb-server    | 	ksql.udf.enable.security.manager = true
ksqldb-server    | 	ksql.udfs.enabled = true
ksqldb-server    | 	ksql.windowed.session.key.legacy = false
ksqldb-server    | 	ssl.cipher.suites = null
ksqldb-server    | 	ssl.enabled.protocols = [TLSv1.2, TLSv1.1, TLSv1]
ksqldb-server    | 	ssl.endpoint.identification.algorithm = https
ksqldb-server    | 	ssl.key.password = null
ksqldb-server    | 	ssl.keymanager.algorithm = SunX509
ksqldb-server    | 	ssl.keystore.location = null
ksqldb-server    | 	ssl.keystore.password = null
ksqldb-server    | 	ssl.keystore.type = JKS
ksqldb-server    | 	ssl.protocol = TLS
ksqldb-server    | 	ssl.provider = null
ksqldb-server    | 	ssl.secure.random.implementation = null
ksqldb-server    | 	ssl.trustmanager.algorithm = PKIX
ksqldb-server    | 	ssl.truststore.location = null
ksqldb-server    | 	ssl.truststore.password = null
ksqldb-server    | 	ssl.truststore.type = JKS
ksqldb-server    |  (io.confluent.ksql.util.KsqlConfig:347)
ksqldb-server    | [2019-12-21 03:02:51,933] WARN Writing to metrics Kafka topic will be disabled (io.confluent.support.metrics.PhoneHomeConfig:61)
ksqldb-server    | [2019-12-21 03:02:51,934] INFO No customer ID configured -- falling back to id 'anonymous' (io.confluent.support.metrics.BaseSupportConfig:336)
ksqldb-server    | [2019-12-21 03:02:51,934] WARN Enforcing customer ID 'anonymous' (io.confluent.support.metrics.PhoneHomeConfig:66)
ksqldb-server    | [2019-12-21 03:02:51,944] WARN Please note that the version check feature of KSQL is enabled.  With this enabled, this instance is configured to collect and report anonymously the version information to Confluent, Inc. ("Confluent") or its parent, subsidiaries, affiliates or service providers every 24hours.  This Metadata may be transferred to any country in which Confluent maintains facilities.  For a more in depth discussion of how Confluent processes such information, please read our Privacy Policy located at http://www.confluent.io/privacy. By proceeding with `confluent.support.metrics.enable=true`, you agree to all such collection, transfer and use of Version information by Confluent. You can turn the version check  feature off by setting `confluent.support.metrics.enable=false` in the KSQL configuration and restarting the KSQL.  See the Confluent Platform documentation for further information. (io.confluent.ksql.version.metrics.KsqlVersionCheckerAgent:90)
ksqldb-server    | [2019-12-21 03:02:51,944] INFO Waiting until monitored service is ready for metrics collection (io.confluent.support.metrics.BaseMetricsReporter:189)
ksqldb-server    | [2019-12-21 03:02:51,945] INFO Monitored service is now ready (io.confluent.support.metrics.BaseMetricsReporter:200)
ksqldb-server    | [2019-12-21 03:02:51,945] INFO Attempting to collect and submit metrics (io.confluent.support.metrics.BaseMetricsReporter:160)
ksqldb-server    | [2019-12-21 03:02:51,946] INFO Server up and running (io.confluent.ksql.rest.server.KsqlServerMain:76)
ksqldb-server    | [2019-12-21 03:02:53,200] INFO Successfully submitted metrics to Confluent via secure endpoint (io.confluent.support.metrics.submitters.ConfluentSubmitter:143)


===================================================================

7、再启动一个termail
在同一目录下启动一个docker，启动ksqldb-cli端，连接上server端；

docker exec -it ksqldb-cli ksql http://ksqldb-server:8088

lemonhall@yunings-Mac-mini ksql % docker exec -it ksqldb-cli ksql http://ksqldb-server:8088

                  ===========================================
                  =       _              _ ____  ____       =
                  =      | | _____  __ _| |  _ \| __ )      =
                  =      | |/ / __|/ _` | | | | |  _ \      =
                  =      |   <\__ \ (_| | | |_| | |_) |     =
                  =      |_|\_\___/\__, |_|____/|____/      =
                  =                   |_|                   =
                  =  Event Streaming Database purpose-built =
                  =        for stream processing apps       =
                  ===========================================

Copyright 2017-2019 Confluent Inc.

CLI v0.6.0, Server v0.6.0 located at http://ksqldb-server:8088

Having trouble? Type 'help' (case-insensitive) for a rundown of how things work!

ksql>


--------------------------------------------------------
===================================================================


8、建立一个流：

ksql> CREATE STREAM riderLocations (profileId VARCHAR, latitude DOUBLE, longitude DOUBLE)
>  WITH (kafka_topic='locations', key='profileId', value_format='json', partitions=1);

 Message
----------------
 Stream created
----------------
ksql>

===================================================================

9、跑一个持续查询：

----------------
ksql> -- Mountain View lat, long: 37.4133, -122.1162
>SELECT * FROM riderLocations
>  WHERE GEO_DISTANCE(latitude, longitude, 37.4133, -122.1162) <= 5 EMIT CHANGES;


Press CTRL-C to interrupt


你会注意到，这个语句，需要Press CTRL-C to interrupt来停止

===================================================================

10、再开一个终端
lemonhall@yunings-Mac-mini ~ % cd development/ksql
lemonhall@yunings-Mac-mini ksql % ls
docker-compose.yml  reademe.md
lemonhall@yunings-Mac-mini ksql % docker exec -it ksqldb-cli ksql http://ksqldb-server:8088

                  ===========================================
                  =       _              _ ____  ____       =
                  =      | | _____  __ _| |  _ \| __ )      =
                  =      | |/ / __|/ _` | | | | |  _ \      =
                  =      |   <\__ \ (_| | | |_| | |_) |     =
                  =      |_|\_\___/\__, |_|____/|____/      =
                  =                   |_|                   =
                  =  Event Streaming Database purpose-built =
                  =        for stream processing apps       =
                  ===========================================

Copyright 2017-2019 Confluent Inc.

CLI v0.6.0, Server v0.6.0 located at http://ksqldb-server:8088

Having trouble? Type 'help' (case-insensitive) for a rundown of how things work!

ksql>

===================================================================

11、插入数据：


ksql> INSERT INTO riderLocations (profileId, latitude, longitude) VALUES ('c2309eec', 37.7877, -122.4205);
>INSERT INTO riderLocations (profileId, latitude, longitude) VALUES ('18f4ea86', 37.3903, -122.0643);
>INSERT INTO riderLocations (profileId, latitude, longitude) VALUES ('4ab5cbad', 37.3952, -122.0813);
>INSERT INTO riderLocations (profileId, latitude, longitude) VALUES ('8b6eae59', 37.3944, -122.0813);
>INSERT INTO riderLocations (profileId, latitude, longitude) VALUES ('4a7c7b41', 37.4049, -122.0822);
>INSERT INTO riderLocations (profileId, latitude, longitude) VALUES ('4ddad000', 37.7857, -122.4011);
ksql>


===================================================================

12、查看结果：

+--------------------------------------------------+--------------------------------------------------+--------------------------------------------------+--------------------------------------------------+--------------------------------------------------+
|ROWTIME                                           |ROWKEY                                            |PROFILEID                                         |LATITUDE                                          |LONGITUDE                                         |
+--------------------------------------------------+--------------------------------------------------+--------------------------------------------------+--------------------------------------------------+--------------------------------------------------+
|1576898186440                                     |4ab5cbad                                          |4ab5cbad                                          |37.3952                                           |-122.0813                                         |
|1576898186493                                     |8b6eae59                                          |8b6eae59                                          |37.3944                                           |-122.0813                                         |
|1576898186519                                     |4a7c7b41                                          |4a7c7b41                                          |37.4049                                           |-122.0822                                         |


===================================================================




