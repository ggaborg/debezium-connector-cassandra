[![License](http://img.shields.io/:license-apache%202.0-brightgreen.svg)](http://www.apache.org/licenses/LICENSE-2.0.html)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/io.debezium/debezium-connector-reactor-cassandra/badge.svg)](https://search.maven.org/search?q=debezium%20connector%20cassandra)
[![Build Status](https://github.com/debezium/debezium-connector-cassandra/workflows/Maven%20CI/badge.svg?branch=main)](https://github.com/debezium/debezium-connector-cassandra/actions)
[![User chat](https://img.shields.io/badge/chat-users-brightgreen.svg)](https://gitter.im/debezium/user)
[![Developer chat](https://img.shields.io/badge/chat-devs-brightgreen.svg)](https://gitter.im/debezium/dev)
[![Google Group](https://img.shields.io/:mailing%20list-debezium-brightgreen.svg)](https://groups.google.com/forum/#!forum/debezium)
[![Stack Overflow](http://img.shields.io/:stack%20overflow-debezium-brightgreen.svg)](http://stackoverflow.com/questions/tagged/debezium)

Copyright Debezium Authors.
Licensed under the [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0).

# Debezium Connector for Datastax Enterprise

Debezium is an open source project that provides a low latency data streaming platform for change data capture (CDC).

This project contains connector for Datastax Enterprise.

Please see the [README.md](https://github.com/debezium/debezium#building-debezium) in the main repository for general instructions on building Debezium from source (prerequisites, usage of Docker etc).

## Datastax Enterprise compatibility

The following table lists the supported versions of Datastax Enterprise:

| Connector version | Datastax Enterprise | Datastax Connector class (`debezium.source.connector.class`) |
|--|---|---|
| 2.2+ | 6.8.0 - 6.8.15 | io.debezium.connector.dse.Dse680Connector |
| 2.2+ | 6.8.16+ | io.debezium.connector.dse.Dse6816Connector |

## Building the Debezium Datastax Enterprise connector

Please follow the steps described in the main repository to build the connector: [building-debezium](https://github.com/debezium/debezium#building-debezium)

### Prerequisites for building:
- Cassandra Connector Core
- few Datastax libraries (from the Datastax 6.8.16 distribution) need to be installed manually in the local maven repository:
```
mvn install:install-file -DgroupId=com.datastax -DartifactId=dse-db-all -Dversion=6.8.16 -Dpackaging=jar -Dfile=dse-db-all-6.8.16.jar

mvn install:install-file -DgroupId=com.datastax -DartifactId=dse-commons -Dversion=6.8.16 -Dpackaging=jar -Dfile=dse-commons-6.8.16.jar

mvn install:install-file -DgroupId=io.netty -DartifactId=netty-all -Dversion=4.1.25.7.dse -Dpackaging=jar -Dfile=netty-all-4.1.25.7.dse.jar
```

## Running under Debezium Server

Please check also the Debezium documentation: [Debezium Server](https://debezium.io/documentation/reference/stable/operations/debezium-server.html) and [Cassandra Connector](https://debezium.io/documentation/reference/stable/operations/debezium-server.html#_cassandra_connector).

### 1. Prerequisites:

- Debezium Server distribution must be deployed on the same instance running Datastax Enterprise.
- DSE_HOME environment variable should be properly defined (usually it is already defined, but if not, it should be something like `DSE_HOME=/opt/dse`). It is used in the `run.sh` startup script.
- Running with java 11+ requires setting the following java options at startup trough the JDK_JAVA_OPTIONS environment variable or equivalent:
```
JDK_JAVA_OPTIONS="--add-exports java.base/jdk.internal.misc=ALL-UNNAMED --add-exports java.base/jdk.internal.ref=ALL-UNNAMED --add-exports java.base/sun.nio.ch=ALL-UNNAMED --add-exports java.management.rmi/com.sun.jmx.remote.internal.rmi=ALL-UNNAMED --add-exports java.rmi/sun.rmi.registry=ALL-UNNAMED --add-exports java.rmi/sun.rmi.server=ALL-UNNAMED --add-exports java.sql/java.sql=ALL-UNNAMED  --add-opens java.base/java.lang.module=ALL-UNNAMED --add-opens java.base/jdk.internal.loader=ALL-UNNAMED --add-opens java.base/jdk.internal.ref=ALL-UNNAMED --add-opens java.base/jdk.internal.reflect=ALL-UNNAMED --add-opens java.base/jdk.internal.math=ALL-UNNAMED --add-opens java.base/jdk.internal.module=ALL-UNNAMED --add-opens java.base/jdk.internal.util.jar=ALL-UNNAMED --add-opens jdk.management/com.sun.management.internal=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens java.base/sun.nio.ch=ALL-UNNAMED"
```

### 2. Deploy Datastax connector:

Create a `dse/lib` folder (it is used in the run.sh script bellow) under Debezium Server deployment and extract there and flatten the contents of the Datastax connector distribution archive (e.g `debezium-connector-dse-{version}-plugin.zip`).

### 3. Changes to Debezium Server `run.sh`:

Replace last line:
```
exec "$JAVA_BINARY" $DEBEZIUM_OPTS $JAVA_OPTS -cp "$RUNNER"$PATH_SEP"conf"$PATH_SEP$LIB_PATH io.debezium.server.Main
```
with line:
```
exec "$JAVA_BINARY" $DEBEZIUM_OPTS $JAVA_OPTS -cp "$RUNNER"$PATH_SEP"conf"$PATH_SEP"lib/slf4j-jboss-logmanager-1.2.0.Final.jar"$PATH_SEP"$DSE_HOME/resources/cassandra/lib/*"$PATH_SEP"dse/lib/*"$PATH_SEP"$DSE_HOME/resources/cassandra/conf"$PATH_SEP"$LIB_PATH"$PATH_SEP"$DSE_HOME/lib/*"$PATH_SEP"$DSE_HOME/resources/solr/lib/*" io.debezium.server.Main
```

### 4. Configure Debezium Server - Sample of basic `application.properties` for running Datastax connector with the Redis sink:

```
# Sink
debezium.sink.type=redis
debezium.sink.redis.address=host.docker.internal:6379
# Dse Connector
debezium.source.connector.class=io.debezium.connector.dse.Dse6816Connector
debezium.source.topic.prefix=dse
debezium.source.cassandra.node.id=dse_node_01
debezium.source.cassandra.hosts=127.0.0.1
debezium.source.cassandra.port=9042
debezium.source.cassandra.config=${DSE_HOME}/resources/cassandra/conf/cassandra.yaml
debezium.source.commit.log.relocation.dir=dse/relocdir
debezium.source.commit.log.real.time.processing.enabled=true
debezium.source.commit.log.marked.complete.poll.interval.ms=2000
debezium.source.offset.storage=io.debezium.server.redis.RedisOffsetBackingStore
debezium.source.http.port=8940
# Transforms
debezium.transforms=AddPrefix,EnvelopeTransformation
debezium.transforms.AddPrefix.type=org.apache.kafka.connect.transforms.RegexRouter
debezium.transforms.AddPrefix.regex=.*
debezium.transforms.AddPrefix.replacement=data:\$0
debezium.transforms.EnvelopeTransformation.type=io.debezium.connector.cassandra.transforms.EnvelopeTransformation
# Quarkus
quarkus.log.level=INFO
quarkus.log.console.json=false
quarkus.http.port=8980
```

### 5. Transformation for Operation Code:

By default, Datastax connector has it's own Operation Codes which are not entirely compatible with Debezium Operation Codes.

If needed, a specific transform can be defined in Debezium Server's `application.properties` to enable the conversion (see point 4. above):
```
debezium.transforms=EnvelopeTransformation
debezium.transforms.EnvelopeTransformation.type=io.debezium.connector.cassandra.transforms.EnvelopeTransformation
```

This will convert Operation Codes as follows:
```
INSERT "i"          -> CREATE "c"
UPDATE "u"          -> UPDATE "u"
DELETE "d"          -> DELETE "d"
RANGE_TOMBSTONE "r" -> TRUNCATE "t"
```

## Contributing

The Debezium community welcomes anyone that wants to help out in any way, whether that includes reporting problems, helping with documentation, or contributing code changes to fix bugs, add tests, or implement new features. See [this document](https://github.com/debezium/debezium/blob/master/CONTRIBUTE.md) for details.
