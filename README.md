# Data analysis system layout

## Launch the project

Recommended memory dedicated for docker while running this progect is 5 GB, otherwise you might have some glitches.

### Launching services

``` bash
git pull git@github.com:Agitolyev/kafka-elastic-integration.git
cd kafka-elastic-integration
docker-compose up -d
```

You may verify that es is up and running executing requests:

``` bash
curl -XGET https://localhost:9200 -u admin:admin --insecure | jq
curl -XGET https://localhost:9200/_cat/nodes?v -u admin:admin --insecure
curl -XGET https://localhost:9200/_cat/plugins?v -u admin:admin --insecure
```

### Starting the simulation

Might requier about a minute for data-generator to boot.
``` bash
curl -X POST -H "Content-Type: application/json" --data @kafka-connect/connector_users.config http://localhost:8083/connectors | jq
```

### Check the simulated data

#### Using kafka-cat

``` bash
docker-compose run kafka-cat kafkacat -b kafka:9092 -t customer_data -C
```

#### Using kafkadb

Go to the ksqldb-cli:
``` bash
docker-compose exec ksqldb-cli ksql http://primary-ksqldb-server:8088
```
Create a stream:
``` 
CREATE STREAM customer_data_stream (
    userid VARCHAR KEY,
    regionid VARCHAR,
    gender VARCHAR
) WITH (
    kafka_topic = 'customer_data',
    value_format = 'json',
    partitions=1,
    replicas=1
);
```

List stream:
```
select * from  customer_data_stream EMIT CHANGES;
```