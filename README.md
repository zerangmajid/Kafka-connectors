
# Building a Real-Time Data Pipeline with Kafka Connect, PostgreSQL, and Change Data Capture (CDC)

## Overview
This project demonstrates setting up a real-time data pipeline that streams rows from a database table in MSSQL to a Kafka topic via Kafka Connect using the JdbcSourceConnector, and then sinks this data into PostgreSQL.

## 1. MSSQL Database Setup
Create a table and populate it with sample data:

```sql
CREATE TABLE test
(
    id    BIGINT IDENTITY (1,1) PRIMARY KEY NOT NULL,
    value VARCHAR(255)                      NOT NULL,
    date  DATETIME2                         NOT NULL
);

-- Create an index for better performance on large datasets
CREATE INDEX inx_test ON test (date);

DECLARE @i int = 0
WHILE @i < 100000
    BEGIN
        SET @i = @i + 1
        INSERT INTO test (value, date) VALUES ('Testing ' + CAST(@i AS VARCHAR), SYSDATETIME());
    END;

SELECT count(*) FROM test;
```

## 2. Kafka Connect Configuration
Ensure the following configurations are correctly set up:

- **Plugin Path**: 
  - `config/kafka_2.13-2.6.0/config/connect-standalone-jdbc.properties` (Check `plugin.path`)
- **Connector Properties**:
  - `config/kafka_2.13-2.6.0/config/connect-mssql-source.properties`
  - `config/kafka_2.13-2.6.0/config/connect-postgres-source.properties`
- **JDBC Connector Plugin Folder**:
  - `/kafka_2.13-2.6.0/libs/jdbc-plugin`
- **MS SQL Server JDBC-driver**:
  - `/kafka_2.13-2.6.0/libs/jdbc-plugin/mssql-jdbc-8.4.1.jre8.jar`
- **PostgreSQL JDBC-driver**:
  - `/kafka_2.13-2.6.0/libs/jdbc-plugin/postgresql-42.2.10.jar`
- **Logging**:
  - Modify `config/kafka_2.13-2.6.0/config/connect-log4j.properties` to set DEBUG level for detailed logs.

## 3. Running the Pipeline

### Step 1: Start Zookeeper
```bash
bin/zookeeper-server-start.sh config/zookeeper.properties
```

### Step 2: Start Kafka Broker
```bash
bin/kafka-server-start.sh config/server.properties
```

### Step 3: Run MSSQL Connector
```bash
CLASSPATH=/mnt/.../kafka-connect-jdbc/kafka_2.13-2.6.0/libs/jdbc-plugin/* ./bin/connect-standalone.sh config/connect-standalone-jdbc.properties ./config/connect-mssql-source.properties
```

### Step 4: Run PostgreSQL in Docker
```bash
docker run --name postgres-container -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=test_db -p 5432:5432 -d postgres:11
```

### Step 5: Create Database, Schema, and Table in PostgreSQL
```bash
docker exec -it postgres-container psql -U postgres -d test_db
psql -h localhost -p 5432  -U postgres -d test_db

CREATE SCHEMA calculator;
SET search_path = calculator;
CREATE TABLE test
(
    id    SERIAL PRIMARY KEY          NOT NULL,
    value VARCHAR(255)                NOT NULL,
    date  TIMESTAMP(3) WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX inx_test ON test (date);
```

### Step 6: Start Kafka Connect for PostgreSQL
```bash
CLASSPATH=/mnt/.../kafka-connect-jdbc/kafka_2.13-2.6.0/libs/jdbc-plugin/postgresql-42.2.10.jar ./bin/connect-standalone.sh config/connect-standalone-postgres-sink.properties ./config/connect-postgres-sink.properties
```

### Step 7: Verify Data in PostgreSQL
```sql
SELECT * FROM calculator.test;
```
