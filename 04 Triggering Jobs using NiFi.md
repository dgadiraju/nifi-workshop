```
CREATE DATABASE IF NOT EXISTS training_citibike;
CREATE EXTERNAL TABLE IF NOT EXISTS training_citibike.trips (
    tripduration INT,
    starttime STRING,
    stoptime STRING,
    start_station_id STRING,
    start_station_name STRING,
    start_station_latitude FLOAT,
    start_station_longitude FLOAT,
    end_station_id STRING,
    end_station_name STRING,
    end_station_latitude FLOAT,
    end_station_longitude FLOAT,
    bikeid INT,
    usertype STRING,
    birth_year INT,
    gender INT
) PARTITIONED BY (tripyear INT)
LOCATION '/user/training/parquet/citibike.db/trips'
STORED AS parquet;
```
