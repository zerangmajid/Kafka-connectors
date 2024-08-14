# Building a Real-Time Data Pipeline with Kafka Connect, PostgreSQL, and Change Data Capture (CDC)
Stream rows from DataBase table to topic Kafka via Kafka Connect (used JdbcSourceConnector)
 In MSSQL
 
CREATE TABLE test
(
    id    BIGINT IDENTITY (1,1) PRIMARY KEY NOT NULL,
    value VARCHAR(255)                      NOT NULL,
    date  DATETIME2                         NOT NULL
);

--don't forget create index for best performance on big data
CREATE INDEX inx_test ON test (date);

DECLARE @i int = 0
WHILE @i < 100000
    BEGIN
        SET @i = @i + 1
        INSERT INTO test (value, date) VALUES ('Testing ' + CAST(@i AS VARCHAR), SYSDATETIME());
    END

--
select count(*) from test;
