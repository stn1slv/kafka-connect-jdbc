# kafka-connect-jdbc

## Start infrastructure

Clean old containers:
```
docker rm zookeeper kafka postgres connect
```
Start components:
```bash
 docker-compose -f compose.yml -f postgres.yml -f kafka.yml up
```

## Start connectors
### Source connector
```
curl --location --request POST 'http://localhost:8083/connectors' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "jdbc-source-1",
    "config": {
        "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
        "connection.url": "jdbc:postgresql://postgres:5432/postgres",
        "connection.user": "postgres",
        "connection.password": "postgres",
        "topic.prefix": "input1",
        "mode": "bulk",
        "query": "SELECT id,name,status FROM public.invoices where status='\''R'\'';",
        "poll.interval.ms": 5000,
        "value.converter": "org.apache.kafka.connect.json.JsonConverter",
        "value.converter.schemas.enable": "true"
    }
}'
```
### Sink connector

```
curl --location --request POST 'http://localhost:8083/connectors' \
--header 'Content-Type: application/json' \
--data-raw '{
    "name": "jdbc-sink-1",
    "config": {
        "connector.class": "io.confluent.connect.jdbc.JdbcSinkConnector",
        "connection.url": "jdbc:postgresql://postgres:5432/postgres",
        "connection.user": "postgres",
        "connection.password": "postgres",
        "table.name.format": "invoices",
        "topics": "output",
        "insert.mode": "update",
        "pk.mode": "record_value",
        "pk.fields": "id",
        "poll.interval.ms": 5000,
        "value.converter": "org.apache.kafka.connect.json.JsonConverter",
        "value.converter.schemas.enable": "true",
        "auto.create": "true"
    }
}'
```

## Stop connectors
### Source connector
```
curl --location --request DELETE 'http://localhost:8083/connectors/jdbc-source-1'
```
### Sink connector
```
curl --location --request DELETE 'http://localhost:8083/connectors/jdbc-sink-1'
```
