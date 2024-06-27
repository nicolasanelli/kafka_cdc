# Kafka environment

Trying to setup an kafka environment using:
- Kafka (without zookeper)
- Kafka Connect
- Some UI (Kafka UI/Control Center/Lenses)


# Connect

Some boilerplate request do create a connector:
````sh
curl --location 'http://localhost:8083/connectors' \
--header 'Content-Type: application/json' \
--data '{
  "name": "mysql-source-connector",  
  "config": {  
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "tasks.max": "1",  
    "database.hostname": "mysql",  
    "database.port": "3306",
    "database.user": "debezium",
    "database.password": "password",
    "database.server.id": "297001", 
    "database.include.list": "exampledb",
    "topic.prefix": "dbserver1",  
    "schema.history.internal.kafka.bootstrap.servers": "kafka:29092",  
    "schema.history.internal.kafka.topic": "schema-changes.inventory",
    "snapshot.mode": "when_needed",
    "include.schema.changes": "true"
  }
}'
````

# Mysql

After creating user, is necessary grant some permitions:

````mysql
GRANT RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'debezium'@'%';
GRANT USAGE ON *.* TO 'debezium'@'%';
GRANT LOCK TABLES ON *.* TO 'debezium'@'%';
GRANT SELECT, INSERT, UPDATE, DELETE ON *database*.* TO 'debezium'@'%';
````

Its also necessary assert that binlog is correct configured in mysql server.
- log_bin | mysql-bin
- binlog_format | ROW
- binlog_row_image | FULL
- binlog_expire_logs_seconds | 864000 (greater than 0)

Could use statement below to check:
````
SELECT variable_name, variable_value FROM performance_schema.global_variables WHERE variable_name in ('server_id', 'binlog_format', 'binlog_row_image', 'binlog_expire_logs_seconds');
````

Can also use SET if some of those not match (ie):
````
set global binlog_format='ROW';
````


Some copy-and-paste commands:
````mysql
use exampledb;
create table ppl (ID int, name varchar(255));
insert into ppl (1, 'name');
insert into ppl ((select max(x.id) + 1 from ppl x), 'another');
````


**Important:** Debezium connector executes a "SHOW MASTER STATUS" command at start of snapshot, and this command has been removed in mysql 8.4. We need an update version to the connector which uses the equivalent command "SHOW BINARY LOG STATUS";

## Refs

- Kafka: https://kafka.apache.org/documentation.html
- Kafka UI: https://docs.kafka-ui.provectus.io/
- Connect: https://docs.confluent.io/platform/current/installation/docker/development.html#extending-images
- Debezium: https://debezium.io/documentation/reference/2.6/connectors/mysql.html#setting-up-mysql
- Debezium Mysql Connector: https://www.confluent.io/hub/debezium/debezium-connector-mysql
- Mysql 8.4 Changelog: https://dev.mysql.com/doc/relnotes/mysql/8.4/en/news-8-4-0.html
- Debezium + Mysql throubleshootinh: https://sylhare.github.io/2023/11/07/Debezium-configuration.html

