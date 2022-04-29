# Proof of concept on postgres event handling with Kafka. 

The objective of this proof of concept is to be able to replicate information from certain tables of a postgres database to other repositories. The following technology stack will be used : 

| Tool     | Version               | 
|:----------|:-------------|
| Zookeeper| 3.5.6         |
| kafka    | 5.4.0-ccs (Commit:9401111f8c4bb0ec)|
| schema-registry | 5.4.0 |
| kafka-connect | 5.4.0 |
| ksqldb-server| 0.6.0 |
| ksqldb-cli|0.6.0 |
| postgres|11 |.


# About licensing 
This laboratory was made with confluent enterprise images, but according to the list of licenses below, our laboratory so far has functionalities of the community version. 

## Apache 2.0 License

| Tool       | 
|:----------|
| Apache Kafka® (with Connect & Streams)| 
| Apache ZooKeeper™ |
| Confluent Clients|
| Ansible Playbooks | 
| Community Connectors |

## Confluent Community License

| Tool       | 
|:----------|
| Pre-built Connectors| 
| REST Proxy|
| Schema Registry|
| ksqlDB| 

## Confluent Enterprise License

| Tool       | 
|:----------|
| Pre-built Connectors| 
| Control Center|
| Operator (k8s)|
| Auto Data Balancer | 
| Role-Based Access Control|
| Structured Audit Logs |
| Secret Protection|
| Multi-Region Clusters|
| Replicator|    
| MQTT Proxy|    
| Tiered Storage (preview)

--------------------------

For more information you should check the page: [Enterprise VS Comunnity](https://www.confluent.io/confluent-community-license-faq/) 


# To run the POC 

The proof of concept is about containers, which are described in a docker compose file, so it must be run with the command 

`docker-compose up -d`

Once we have the containers running we can proceed to create connectors as follows: 

## Source Connector: Send data from Postgres to Kafka
~~~
curl -X POST http://localhost:8083/connectors -H "Content-Type: application/json" -d '{
        "name": "jdbc_source_postgres_11",
        "config": {
                "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
                "connection.url": "jdbc:postgresql://postgres:5432/postgres",
                "connection.user": "connect_user",
                "connection.password": "asgard",
                "topic.prefix": "postgres-11-",
                "mode":"timestamp",
                "table.whitelist" : "demo.accounts",
                "timestamp.column.name": "update_ts",
                "validate.non.null": false,
                "transforms"  :    "copyFieldToKey,extractKeyFromStruct,removeKeyFromValue",
                "transforms.copyFieldToKey.type": "org.apache.kafka.connect.transforms.ValueToKey",
                "transforms.copyFieldToKey.fields": "id",
                "transforms.extractKeyFromStruct.type": "org.apache.kafka.connect.transforms.ExtractField$Key",
                "transforms.extractKeyFromStruct.field" : "id",
                "transforms.removeKeyFromValue.type" : "org.apache.kafka.connect.transforms.ReplaceField$Value",
                "transforms.removeKeyFromValue.blacklist": "id",
                "key.converter" : "org.apache.kafka.connect.converters.IntegerConverter"
                }
        }'
~~~

## Sink Connector: Send data from Kafka(KSQL) to Postgres

~~~
curl -X POST http://localhost:8083/connectors -H "Content-Type: application/json" -d '{
        "name": "jdbc_source_postgres_11",
        "config": {
                "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
                "connection.url": "jdbc:postgresql://postgres:5432/postgres",
                "connection.user": "connect_user",
                "connection.password": "asgard",
                "topic.prefix": "postgres-11-",
                "mode":"timestamp",
                "table.whitelist" : "demo.accounts",
                "timestamp.column.name": "update_ts",
                "validate.non.null": false,
                "transforms"  :    "copyFieldToKey,extractKeyFromStruct,removeKeyFromValue",
                "transforms.copyFieldToKey.type": "org.apache.kafka.connect.transforms.ValueToKey",
                "transforms.copyFieldToKey.fields": "id",
                "transforms.extractKeyFromStruct.type": "org.apache.kafka.connect.transforms.ExtractField$Key",
                "transforms.extractKeyFromStruct.field" : "id",
                "transforms.removeKeyFromValue.type" : "org.apache.kafka.connect.transforms.ReplaceField$Value",
                "transforms.removeKeyFromValue.blacklist": "id",
                "key.converter" : "org.apache.kafka.connect.converters.IntegerConverter"
                }
        }'
~~~


for more information about kafka connect visit: [Kafka Connect](https://docs.confluent.io/platform/current/connect/index.html) 

To verify if the connector was created correctly we execute: 
~~~
curl localhost:8083/connectors
~~~

To verify status from connecto:
~~~
curl -s "http://localhost:8083/connectors/jdbc_source_postgres_01/status"|jq '.'
~~~

To check the operation of the POC, we enter postgres and execute an update on the accounts table. For that we can use a GUI for postgres or we enter the container in the following way: 

~~~
docker exec --tty --interactive postgres bash -c 'psql -U $POSTGRES_USER $POSTGRES_DB'
~~~
