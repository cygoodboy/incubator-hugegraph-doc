---
title: "HugeGraph-Server Quick Start"
linkTitle: "Install HugeGraph-Server"
weight: 1
---

### 1 HugeGraph-Server Overview

HugeGraph-Server is the core part of the HugeGraph Project, contains submodules such as Core、Backend、API.

The Core Module is an implementation of the Tinkerpop interface; The Backend module is used to save the graph data to the data store, currently supported backends include：Memory、Cassandra、ScyllaDB、RocksDB; The API Module provides HTTP Server, which converts Client's HTTP request into a call to Core Module.

> There will be two spellings HugeGraph-Server and HugeGraphServer in the document, and other modules are similar. There is no big difference in the meaning of these two ways of writing, which can be distinguished as follows: `HugeGraph-Server` represents the code of server-related components, `HugeGraphServer` represents the service process.

### 2 Dependency

#### 2.1 Install Java11 (JDK 11)

Consider use Java 11 to run `HugeGraph-Server`(also compatible with Java 8), and configure by yourself.

**Be sure to execute the `java -version` command to check the jdk version before reading**

```bash
java -version
```

#### 2.2 Install GCC-4.3.0(GLIBCXX_3.4.10) or update version (optional)

If you are using the RocksDB backend, be sure to execute the `gcc --version` command to check the gcc version; if you are using other backends, this is not required.

```bash
gcc --version
```

### 3 Deploy

There are three ways to deploy HugeGraph-Server components:

- Method 1: One-click deployment
- Method 2: Download the tarball
- Method 3: Source code compilation

#### 3.1 One-click deployment

HugeGraph-Tools provides a command-line tool for one-click deployment, users can use this tool to quickly download、decompress、configure and start HugeGraphServer and HugeGraphStudio with one click.
of course, you still have to download the tarball of HugeGraph-Tools first.

```bash
wget https://github.com/hugegraph/hugegraph-tools/releases/download/v${version}/hugegraph-tools-${version}.tar.gz
tar -zxvf hugegraph-tools-${version}.tar.gz
cd hugegraph-tools-${version}
```

> note：${version} is the version, The latest version can refer to [Download Page](/docs/download/download), Or click the link to download directly from the Download page

The general entry script for HugeGraph-Tools is `bin/hugegraph`, Users can use the `help` command to view its usage, here only the commands for one-click deployment are introduced.

```bash
bin/hugegraph deploy -v {hugegraph-version} -p {install-path} [-u {download-path-prefix}]
```

`{hugegraph-version}` indicates the version of HugeGraphServer and HugeGraphStudio to be deployed, users can view the `conf/version-mapping.yaml` file for version information, `{install-path}` specify the installation directory of HugeGraphServer and HugeGraphStudio, `{download-path-prefix}` optional, specify the download address of HugeGraphServer and HugeGraphStudio tarball, use default download URL if not provided, for example, to start HugeGraph-Server and HugeGraphStudio version 0.6, write the above command as `bin/hugegraph deploy -v 0.6 -p services`.

#### 3.2 Download the tar tarball

```bash
wget https://github.com/hugegraph/hugegraph/releases/download/v${version}/hugegraph-${version}.tar.gz
tar -zxvf hugegraph-${version}.tar.gz
```

#### 3.3 Source code compilation

Download HugeGraph source code

```bash
git clone https://github.com/hugegraph/hugegraph.git
```

Compile and generate tarball

```bash
cd hugegraph
mvn package -DskipTests
```

The execution log is as follows:

```bash
......
[INFO] Reactor Summary:
[INFO] 
[INFO] hugegraph .......................................... SUCCESS [  0.003 s]
[INFO] hugegraph-core ..................................... SUCCESS [ 15.335 s]
[INFO] hugegraph-api ...................................... SUCCESS [  0.829 s]
[INFO] hugegraph-cassandra ................................ SUCCESS [  1.095 s]
[INFO] hugegraph-scylladb ................................. SUCCESS [  0.313 s]
[INFO] hugegraph-rocksdb .................................. SUCCESS [  0.506 s]
[INFO] hugegraph-mysql .................................... SUCCESS [  0.412 s]
[INFO] hugegraph-palo ..................................... SUCCESS [  0.359 s]
[INFO] hugegraph-dist ..................................... SUCCESS [  7.470 s]
[INFO] hugegraph-example .................................. SUCCESS [  0.403 s]
[INFO] hugegraph-test ..................................... SUCCESS [  1.509 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
......
```

After successful execution, hugegraph-*.tar.gz files will be generated in the hugegraph directory, which is the tarball generated by compilation.

### 4 Config

If you need to quickly start HugeGraph just for testing, then you only need to modify a few configuration items (see next section).
for detailed configuration introduction, please refer to [configuration document](/docs/config/config-guide) and [introduction to configuration items](/docs/config/config-option)

### 5 Startup

The startup is divided into "first startup" and "non-first startup". This distinction is because the back-end database needs to be initialized before the first startup, and then the service is started.
after the service is stopped artificially, or when the service needs to be started again for other reasons, because the backend database is persistent, you can start the service directly.

When HugeGraphServer starts, it will connect to the backend storage and try to check the version number of the backend storage. If the backend is not initialized or the backend has been initialized but the version does not match (old version data), HugeGraphServer will fail to start and give an error message.

If you need to access HugeGraphServer externally, please modify the `restserver.url` configuration item of `rest-server.properties`
（default is `http://127.0.0.1:8080`）, change to machine name or IP address.

Since the configuration (hugegraph.properties) and startup steps required by various backends are slightly different, the following will introduce the configuration and startup of each backend one by one.

#### 5.1 Memory

Update hugegraph.properties

```properties
backend=memory
serializer=text
```

> The data of the Memory backend is stored in memory and cannot be persisted. It does not need to initialize the backend. This is the only backend that does not require initialization.

Start server

```bash
bin/start-hugegraph.sh
Starting HugeGraphServer...
Connecting to HugeGraphServer (http://127.0.0.1:8080/graphs)....OK
```

The prompted url is the same as the restserver.url configured in rest-server.properties

#### 5.2 RocksDB

> RocksDB is an embedded database that does not require manual installation and deployment. GCC version >= 4.3.0 (GLIBCXX_3.4.10) is required. If not, GCC needs to be upgraded in advance

Update hugegraph.properties

```properties
backend=rocksdb
serializer=binary
rocksdb.data_path=.
rocksdb.wal_path=.
```

Initialize the database (required only on first startup)

```bash
cd hugegraph-${version}
bin/init-store.sh
```

Start server

```bash
bin/start-hugegraph.sh
Starting HugeGraphServer...
Connecting to HugeGraphServer (http://127.0.0.1:8080/graphs)....OK
```

#### 5.3 Cassandra

> users need to install Cassandra by themselves, requiring version 3.0 or above, [download link](http://cassandra.apache.org/download/)

Update hugegraph.properties

```properties
backend=cassandra
serializer=cassandra

# cassandra backend config
cassandra.host=localhost
cassandra.port=9042
cassandra.username=
cassandra.password=
#cassandra.connect_timeout=5
#cassandra.read_timeout=20

#cassandra.keyspace.strategy=SimpleStrategy
#cassandra.keyspace.replication=3
```

Initialize the database (required only on first startup)


```bash
cd hugegraph-${version}
bin/init-store.sh
Initing HugeGraph Store...
2017-12-01 11:26:51 1424  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Opening backend store: 'cassandra'
2017-12-01 11:26:52 2389  [main] [INFO ] com.baidu.hugegraph.backend.store.cassandra.CassandraStore [] - Failed to connect keyspace: hugegraph, try init keyspace later
2017-12-01 11:26:52 2472  [main] [INFO ] com.baidu.hugegraph.backend.store.cassandra.CassandraStore [] - Failed to connect keyspace: hugegraph, try init keyspace later
2017-12-01 11:26:52 2557  [main] [INFO ] com.baidu.hugegraph.backend.store.cassandra.CassandraStore [] - Failed to connect keyspace: hugegraph, try init keyspace later
2017-12-01 11:26:53 2797  [main] [INFO ] com.baidu.hugegraph.backend.store.cassandra.CassandraStore [] - Store initialized: huge_graph
2017-12-01 11:26:53 2945  [main] [INFO ] com.baidu.hugegraph.backend.store.cassandra.CassandraStore [] - Store initialized: huge_schema
2017-12-01 11:26:53 3044  [main] [INFO ] com.baidu.hugegraph.backend.store.cassandra.CassandraStore [] - Store initialized: huge_index
2017-12-01 11:26:53 3046  [pool-3-thread-1] [INFO ] com.baidu.hugegraph.backend.Transaction [] - Clear cache on event 'store.init'
2017-12-01 11:26:59 9720  [main] [INFO ] com.baidu.hugegraph.HugeGraph [] - Opening backend store: 'cassandra'
2017-12-01 11:27:00 9805  [main] [INFO ] com.baidu.hugegraph.backend.store.cassandra.CassandraStore [] - Failed to connect keyspace: hugegraph1, try init keyspace later
2017-12-01 11:27:00 9886  [main] [INFO ] com.baidu.hugegraph.backend.store.cassandra.CassandraStore [] - Failed to connect keyspace: hugegraph1, try init keyspace later
2017-12-01 11:27:00 9955  [main] [INFO ] com.baidu.hugegraph.backend.store.cassandra.CassandraStore [] - Failed to connect keyspace: hugegraph1, try init keyspace later
2017-12-01 11:27:00 10175 [main] [INFO ] com.baidu.hugegraph.backend.store.cassandra.CassandraStore [] - Store initialized: huge_graph
2017-12-01 11:27:00 10321 [main] [INFO ] com.baidu.hugegraph.backend.store.cassandra.CassandraStore [] - Store initialized: huge_schema
2017-12-01 11:27:00 10413 [main] [INFO ] com.baidu.hugegraph.backend.store.cassandra.CassandraStore [] - Store initialized: huge_index
2017-12-01 11:27:00 10413 [pool-3-thread-1] [INFO ] com.baidu.hugegraph.backend.Transaction [] - Clear cache on event 'store.init'
```

Start server

```bash
bin/start-hugegraph.sh
Starting HugeGraphServer...
Connecting to HugeGraphServer (http://127.0.0.1:8080/graphs)....OK
```

#### 5.4 ScyllaDB

> users need to install ScyllaDB by themselves, version 2.1 or above is recommended, [download link](https://docs.scylladb.com/getting-started/)

Update hugegraph.properties

```properties
backend=scylladb
serializer=scylladb

# cassandra backend config
cassandra.host=localhost
cassandra.port=9042
cassandra.username=
cassandra.password=
#cassandra.connect_timeout=5
#cassandra.read_timeout=20

#cassandra.keyspace.strategy=SimpleStrategy
#cassandra.keyspace.replication=3
```

Since the scylladb database itself is an "optimized version" based on cassandra, if the user does not have scylladb installed, they can also use cassandra as the backend storage directly. They only need to change the backend and serializer to scylladb, and the host and post point to the seeds and port of the cassandra cluster. Yes, but it is not recommended to do so, it will not take advantage of scylladb itself.

Initialize the database (required only on first startup)

```bash
cd hugegraph-${version}
bin/init-store.sh
```

Start server

```bash
bin/start-hugegraph.sh
Starting HugeGraphServer...
Connecting to HugeGraphServer (http://127.0.0.1:8080/graphs)....OK
```

#### 5.5 HBase

> users need to install HBase by themselves, requiring version 2.0 or above,[download link](https://hbase.apache.org/downloads.html)

Update hugegraph.properties

```properties
backend=hbase
serializer=hbase

# hbase backend config
hbase.hosts=localhost
hbase.port=2181
# Note: recommend to modify the HBase partition number by the actual/env data amount & RS amount before init store
# it may influence the loading speed a lot
#hbase.enable_partition=true
#hbase.vertex_partitions=10
#hbase.edge_partitions=30
```

Initialize the database (required only on first startup)

```bash
cd hugegraph-${version}
bin/init-store.sh
```

Start server

```bash
bin/start-hugegraph.sh
Starting HugeGraphServer...
Connecting to HugeGraphServer (http://127.0.0.1:8080/graphs)....OK
```

> for more other backend configurations, please refer to[introduction to configuration items](/docs/config/config-option)

### 6 Access server

#### 6.1 Service startup status check

Use `jps` to see service process

```bash
jps
6475 HugeGraphServer
```

`curl` request `RESTfulAPI`

```bash
echo `curl -o /dev/null -s -w %{http_code} "http://localhost:8080/graphs/hugegraph/graph/vertices"`
```

Return 200, which means the server starts normally.

#### 6.2 Request Server

The RESTful API of HugeGraphServer includes various types of resources, typically including graph, schema, gremlin, traverser and task.

- `graph` contains `vertices`、`edges`
- `schema`  contains `vertexlabels`、 `propertykeys`、 `edgelabels`、`indexlabels`
- `gremlin` contains various `Gremlin` statements, such as `g.v()`, which can be executed synchronously or asynchronously
- `traverser` contains various advanced queries including shortest paths, intersections, N-step reachable neighbors, etc.
- `task` contains query and delete with asynchronous tasks

##### 6.2.1 Get vertices and its related properties in `hugegraph`

```bash
curl http://localhost:8080/graphs/hugegraph/graph/vertices 
```

_explanation_

1. Since there are many vertices and edges in the graph, for list-type requests, such as getting all vertices, getting all edges, etc., the server will compress the data and return it, so when use curl, you get a bunch of garbled characters, you can redirect to gunzip for decompression. It is recommended to use Chrome browser + Restlet plugin to send HTTP requests for testing.

    ```
    curl "http://localhost:8080/graphs/hugegraph/graph/vertices" | gunzip
    ```

2. The current default configuration of HugeGraphServer can only be accessed locally, and the configuration can be modified so that it can be accessed on other machines.

    ```
    vim conf/rest-server.properties
    
    restserver.url=http://0.0.0.0:8080
    ```

response body：

```javasript
{
    "vertices": [
        {
            "id": "2lop",
            "label": "software",
            "type": "vertex",
            "properties": {
                "price": [
                    {
                        "id": "price",
                        "value": 328
                    }
                ],
                "name": [
                    {
                        "id": "name",
                        "value": "lop"
                    }
                ],
                "lang": [
                    {
                        "id": "lang",
                        "value": "java"
                    }
                ]
            }
        },
        {
            "id": "1josh",
            "label": "person",
            "type": "vertex",
            "properties": {
                "name": [
                    {
                        "id": "name",
                        "value": "josh"
                    }
                ],
                "age": [
                    {
                        "id": "age",
                        "value": 32
                    }
                ]
            }
        },
        ...
    ]
}
```

For detailed API, please refer to[RESTful-API](/dcos/clients/restful-api)

### 7 Stop Server

```bash
$cd hugegraph-${version}
$bin/stop-hugegraph.sh
```
