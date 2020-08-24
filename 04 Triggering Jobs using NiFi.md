As part of this session we will talk about how to trigger jobs using NiFi. We will also talk about **Wait** and **Notify**.

* Creating Database and Table
* Executing Commands
* Execute Shell Command - Process output
* Execute Shell Script - Using output destination attribute
* Execute Spark SQL Query
* Wait and Notify

## Creating Database and Table

Let us create a database and table for trips data. We will create partitioned table where the table is partitioned by year.
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
## Executing Commands
Here are the processors that can be leveraged to execute the commands.
* ExecuteProcessor - Does not take any input
* ExecuteStreamingCommand - Need to provide input

We can pass the command and all the relevant arguments with appropriate delimiter. We can also pass the environment variables as additional custom properties.

## Execute Shell Command - Process output
Let us understand how to run simple linux commands using NiFi and process the output.
* Get files related to a given year using find command.
```
find /data/citibike/trips -name "2013*" -type f
```
* Output will be captured as part of the flowfile.
* We can split the files using **SplitText**. By passing **Line Split Count** as 1 the flow file will be divided into multiple flowfiles.
* Each flowfile will contain one record and we can convert that record into attributes using **ExtractText**.
* We can pass regular expressions to **ExtractText**. In our case we will use `.*` for custom property called as `filename` to capture the output of `find` command (one file at a time).
* Once we get the names, we can pass it to processors like **FetchFile** to get the files from file system for further processing or ingesting into our Data Lake.

## Execute Shell Script - Using output destination attribute
Let us understand how we can execute shell script and process output as part of the attribute.
* Here is the shell script to get number of files based up on the pattern.
```
FILE_COUNT=0
for i in /data/citibike/trips/${1}*
do
    FILE_COUNT=`expr $FILE_COUNT + 1`
done
echo $FILE_COUNT
```
* With ExecuteStreamingCommand, we can capture the output as part of the attribute.

## Execute Spark SQL Query
Let us understand how to execute Spark SQL queries.
* We can use `spark-sql` command to run Spark SQl queries.
* To run single query we can use `-e` option. We typically pass the query using double quotes after `-e`.
* While running queries using `spark-sql` we might have to pass control arguments such as `--master`, `--num-executors` etc to control the run time behavior.
* Let us use **GenerateFlowFile**, **ReplaceText** and **ExecuteStreamingCommand** to run a simple **ALTER TABLE** command to add partition to our table.

## Wait and Notify
Let us understand how to use Wait and Notify to develop conditional flows. Here is the design of the flow.
* Get number of files for a given year.
* Wait until all the files are copied using **Wait** so that the command to add partition runs only once irrespective of number of files.
* Once all the files are copies, we will notify using **Notify**.
* Attribute for actual file count: **file.count**
* Release Signal Identifier: **wait.filename**. It is created using **filename** from **GenerateFlowfile**.
* Signal Counter Name: **files.copied**. It will be automatically incremented by **Notify** whenever the files are copied.
* When **files.copied** is equal to **file.count**, then the partition will be added.
* We need to configure **DistributedMapCacheClientService** for **Wait** and **Notify**.
