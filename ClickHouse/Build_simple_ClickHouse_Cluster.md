# Build simple ClickHouse Cluster on Fyre for development

## Background 



## applied 3 Fyre machines:

machines and software to be installed:

| machine | ip | software installed |
|---|---|---|
|ch-001|IP1|jdk, zookeeper, clickhouse|
|ch-002|IP2|jdk, zookeeper |
|ch-003|IP3|jdk, zookeeper, clickhouse|

vi /etc/hosts

IP1 ch-001   
IP2 ch-002  
IP3 ch-003

## 1. Install java: (on all 3 machines):
  ```
  sudo apt install openjdk-8-jdk
  ```

## 2. Install zookeeper

- Download zookeeper (on all 3 machines):

    ```
    sudo wget https://dlcdn.apache.org/zookeeper/zookeeper-3.7.0/apache-zookeeper-3.7.0-bin.tar.gz

    mkdir /data

    sudo tar -zxvf ./apache-zookeeper-3.7.0-bin.tar.gz -C /data

    cd /data

    ln -s apache-zookeeper-3.7.0-bin zookeeper
    ```

- Create data and log folder and make a copy of zoo.fg (on all 3 machines):
    ```
    cd /data/zookeeper

    mkdir data
    mkdir logs

    cp conf/zoo_sample.cfg conf/zoo.cfg
    ```

- Edit zoo.fg to config zookeeper (on all 3 machines):  

    ```
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataDir=/data/zookeeper/data
    clientPort=2181
    server.1=IP1:2888:3888
    server.2=IP2:2888:3888
    server.3=IP3:2888:3888
    ```
- Set the id for each machine:   

    on the machine IP1, run:    
    ```
    /data/zookeeper # echo 1 > /data/zookeeper/data/myid   
    ```
    on the machine IP2, run:   
    ```
    /data/zookeeper # echo 2 > /data/zookeeper/data/myid  
    ```
    on the machine IP3, run:   
    ```
    /data/zookeeper # echo 3 > /data/zookeeper/data/myid  
    ```

- Start zookeeper (on all 3 machines):
    ```
    ./bin/zkServer.sh start
    ```

- Check zookeeper status:  
    ```
    /data/zookeeper # ./bin/zkServer.sh status                                                                                                                                    root@squibber1
    /usr/bin/java
    ZooKeeper JMX enabled by default
    Using config: /data/zookeeper/bin/../conf/zoo.cfg
    Client port found: 2181. Client address: localhost. Client SSL: false.
    Mode: leader
    ```  
   

## 3. Setup ClickHouse cluster  
run on machine IP1 and IP3
- Install  ClickHouse 
    ```
    sudo apt-get install -y apt-transport-https ca-certificates dirmngr
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754

    echo "deb https://packages.clickhouse.com/deb stable main" | sudo tee \
        /etc/apt/sources.list.d/clickhouse.list
    sudo apt-get update

    sudo apt-get install -y clickhouse-server clickhouse-client
    ```
use `/etc/clickhouse-server/config.xml`  
- Add listen host for accepting all  
    
    ```
    <!-- Default values - try listen localhost on IPv4 and IPv6. -->
    <!--
    <listen_host>::1</listen_host>
    <listen_host>127.0.0.1</listen_host>
    -->
    <listen_host>0.0.0.0</listen_host>          <---------------------------------- add 
    <!-- Don't exit if IPv6 or IPv4 networks are unavailable while trying to listen. -->
    <!-- <listen_try>0</listen_try> -->

    <!-- Allow multiple servers to listen on the same address:port. This is not recommended.
      -->
    <!-- <listen_reuse_port>0</listen_reuse_port> -->

    <!-- <listen_backlog>4096</listen_backlog> -->    
    ```

- Setup my data path :
    ```
    <!-- Path to data directory, with trailing slash. -->
    <path>/data/ClickHouse/</path>

    <!-- Path to temporary data for processing hard queries. -->
    <tmp_path>/data/ClickHouse/tmp/</tmp_path>    
    ```

- Add config for cluster:      
    set a cluster `groupby_test` with 1 shard 2 replica   
    
    ```
    <remote_servers>
        <groupby_test> <!-- cluster name self defined -->
            <shard> 
                <!-- Optional. Shard weight when writing data. Default: 1. -->
                <weight>1</weight>
                <!-- Optional. Whether to write data to just one of the replicas. Default: false (write data to all replicas). -->
                <internal_replication>true</internal_replication>
                <replica>  
                    <host>IP1</host>
                    <port>9000</port>
                </replica>
                <replica>
                    <host>IP3</host>
                    <port>9000</port>
                </replica>
            </shard>
            <shard>   <!-- add extra shard is needed>
            ...
            </shard> 
        </groupby_test>        
    ```
- Add zookeeper config:
    ```
        <zookeeper>
            <node index="1">
                <host>IP1</host>
                <port>2181</port>
            </node>
            <node index="3">
                <host>IP2</host>
                <port>2181</port>
            </node>
            <node index="3">
                <host>IP3</host>
                <port>2181</port>
            </node>
        </zookeeper>
    ```
- Change macros for IP1  
    ```
        <macros>
            <shard>1</shard>
            <replica>IP1</replica>
        </macros>
    ```
- Change macro for IP3 
    ```
        <macros>
            <shard>1</shard>
            <replica>IP3</replica>
        </macros>
    ```
- Start server on both machine

    ```
    sudo service clickhouse-server start

    ```
- Run client on one machine and check cluster:
    ```
    clickhouse-client
    
    SELECT *
    FROM system.clusters

    Query id: c015f717-c871-4061-b9b3-dd0d00bd51e0

    ┌─cluster────────┬─shard_num─┬─shard_weight─┬─replica_num─┬─host_name────┬─host_address─┬─port─┬─is_local─┬─user────┬─default_database─┬─errors_count─┬─slowdowns_count─┬─estimated_recovery_time─┐
    │ groupby_test   │         1 │            1 │           1 │ IP1 │ IP1 │ 9000 │        1 │ default │                  │            0 │               0 │                       0 │
    │ groupby_test   │         1 │            1 │           2 │ IP3 │ IP3 │ 9000 │        0 │ default │                  │            0 │               0 │                       0 │
    | ...            |           |              |             |              |              |      |          |         |                  |              |                 |                         |
    └────────────────┴───────────┴──────────────┴─────────────┴──────────────┴──────────────┴──────┴──────────┴─────────┴──────────────────┴──────────────┴─────────────────┴─────────────────────────┘

    ```


