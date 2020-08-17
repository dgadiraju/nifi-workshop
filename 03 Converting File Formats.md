# Converting File Formats

As part of data ingestion process, we might have to convert file format with out performing any transformations over data. In this module we will explore options in NiFi about File Format conversion.

* Pre-requisites
* General Guidelines
* Setup and Analyze Data
* Design NiFi Flow
* Configuring Reader and Writer
* Converting CSV to JSON
* Validate Ingested Data
* Overview of Schema Registry
* Passing Avro Schema
* CSV to Parquet with Schema

The content will be free, however if you want to watch videos to understand key concepts feel free to [join our YouTube Channel as a member](https://www.youtube.com/channel/UCakdSIPsJqiOLqylgoYmwQg/join).

[![NiFi - Convert File Formats](http://img.youtube.com/vi/N2iggpbaw90/0.jpg)](http://www.youtube.com/watch?v=N2iggpbaw90 "NiFi - Convert File Formats")

## Pre-requisites
Here are the pre-requisites before we get into the session. We will be using single node development server to explore NiFi in-depth.
* We are supposed to have an environment where Hadoop and NiFi are setup.
* Run `jps` to ensure whether HDFS Components and NiFi are running or not.
* In case if HDFS and NiFi processes are not running, run following commands to get the environment ready.
```
start-dfs.sh
start-yarn.sh
nifi.sh start
# You can run jps and see if all the Java processes related to HDFS and NiFi are running or not.
```

## General Guidelines
We might get files from different source systems using different File Formats into the landing zone on File Server or HDFS. However we tend to standardize the file formats in the data lake.
* Typical File Formats from sources - Delimited Text Files with Header
* Here are some of the file formats we use in Data Lake.
  * JSON
  * Avro
  * Parquet
* As part of the data ingestion process we typically store the data as is. We do not manipulate the data before ingesting data into the data lake.
* NiFi provides robust mechanism to get the data from different file formats to industry standard Data Lake file formats.
* We can use processors such as **ConvertRecord** to convert data from one file format to another with out manipulating the data.
* If we get the data in text file format with header, we can leverage the header to infer the column names while converting the file formats.
  
## Setup and Analyze Data
We will use citibike data set on top of retail_db data set for this session. You can get citibike data set from this [location](https://s3.amazonaws.com/tripdata/index.html).
* We can use `wget` to download the data.
* You can find that there are 2 patterns to the data set. We can build wget command using range of months rather than downloading one file at a time.
  * yyyyMM-citibike-tripdata.zip -> 20{13..16}{01..12}-citibike-tripdata.zip
  * yyyyMM-citibike-tripdata.csv.zip -> 20{17..20}{01..12}-citibike-tripdata.csv.zip
* Here are the commands to download the data for all the months. We will be downloading the data into `/data/citibike/trips`.
```
mkdir -p /data/citibike/trips && cd /data/citibike/trips
wget https://s3.amazonaws.com/tripdata/20{13..16}{01..12}-citibike-tripdata.zip
wget https://s3.amazonaws.com/tripdata/20{17..20}{01..12}-citibike-tripdata.csv.zip
```
* Now let us unzip the data and understand how the data look like. We will be reviewing 2 files (201306 and 202001}
  * Make sure to install unzip using `sudo yum -y install unzip` on Centos or RedHat based machines. You can use `apt` on Ubuntu based machines.
  * Run `unzip 201306-citibike-tripdata.zip` and review the characteristics of the data.
  * Run `unzip 202001-citibike-tripdata.csv.zip` and review the characteristics of the data.
  * We can also run below script to uncompress all the files to understand the details of the files that are compressed.
```
ls -1 *.zip|while read LINE
do
  unzip -u $LINE
done
```
  * Once you analyze, you can cleanup the uncompressed files by saying `rm *.csv && rm -rf __MACOSX`.
* Here are the details about our data.
  * Data contains header. However header is not uniform across all files.
    * Examples: 201703 and 201704
  * **birthyear** have nulls and nulls are represented as **NULL** or **\N**
    * Examples: 201306 and 201401
  * Underlying file names after unzipping following different naming convention.
  * There are some system files generated and they are supposed to be ignored before ingestion of data into the Data Lake or Data Hub.
* Here is the **avro** schema for our data. We will use it at a later point in time.
```
{
  "name": "CitibikeTrip",
  "type": "record",
  "namespace": "com.acme.avro",
  "fields": [
    {
      "name": "tripduration",
      "type": ["int", "null"]
    },
    {
      "name": "starttime",
      "type": ["string", "null"]
    },
    {
      "name": "stoptime",
      "type": ["string", "null"]
    },
    {
      "name": "start_station_id",
      "type": ["string", "null"]
    },
    {
      "name": "start_station_name",
      "type": ["string", "null"]
    },
    {
      "name": "start_station_latitude",
      "type": ["float", "null"]
    },
    {
      "name": "start_station_longitude",
      "type": ["float", "null"]
    },
    {
      "name": "end_station_id",
      "type": ["string", "null"]
    },
    {
      "name": "end_station_name",
      "type": ["string", "null"]
    },
    {
      "name": "end_station_latitude",
      "type": ["float", "null"]
    },
    {
      "name": "end_station_longitude",
      "type": ["float", "null"]
    },
    {
      "name": "bikeid",
      "type": ["int", "null"]
    },
    {
      "name": "usertype",
      "type": ["string", "null"]
    },
    {
      "name": "birth_year",
      "type": ["int", "null"]
    },
    {
      "name": "gender",
      "type": ["int", "null"]
    }
  ]
}
```
## Design NiFi Flow
Let us come up with all the processors that are required to get the data from CSV to JSON using citibike data. We will validate using 2019 data set.
* ListFile - list the files in the local file system on the server where NiFi is running. Add filter to process the files belonging to **2019**.
* UpdateAttribute - capture the original path and file name (without extension) so that we can standardize the naming convention.
* FetchFile - fetch the files so that the files are converted downstream to JSON and placed in HDFS.
* UnpackContent - as files are zipped, we need to unzip before we convert the file format.
* RouteOnAttribute - to ignore system folders or files (folders with name **__MACOSX**)
* ConvertRecord - processor to convert each and every record in input file to target format (JSON)
* UpdateAttribute - to set the filename with appropriate file name and right extension.
* PutHDFS - to place the files in HDFS.

## Configuring Reader and Writer
Let us understand how to configure the reader and writer while converting the file formats before ingesting data into the data lake.
* We can use CSVReader controller service to read the data from CSV Files.
* We need to configure the controller service based up on the observations.
  * Schema Access Strategy - **Infer Schema** or 
  * Treat first line as header - True
* Make sure to enable the controller service
* We can use JSONRecordSetWriter controller service to convert each record into JSON document.
* The new flowfile generated will contain records in the form of JSON documents.
* Most of the properties can be default.
  * Output Grouping - One Line Per Object
  * Schema Access Strategy - Inherit Record Schema

## Converting CSV to JSON
As we are done with ConvertRecord with Reader and Writer, we can take care of rest of the stuff and start the flow.
* Update Attribute such as filename to provide meaningful names to the files.
* Make sure all the properties are set in PutHDFS to write the data into HDFS.
* As the data size is in GBs, we can consider compression while placing the files in HDFS.
* Once we make the changes let us see data flowing from input path to the target folder in the form of JSON files.

## Validate Ingested Data
Let us validated by reading the data using Pyspark.
```
trips = spark. \
    read. \
    json('/user/training/json/citibike/trips/2019*')
trips.printSchema()
trips.count()
trips.show()
```

Here is the code snippet using Scala.
```
trips = spark.
    read.
    json("/user/training/json/citibike/trips/2019*')
trips.printSchema
trips.count
trips.show
```
## Overview of Schema Registry
Here are the details about Schema Registry.
* Even though we can hard code the schema, we should avoid it to make our flows generic.
* We can integrate several Schema Registries.
  * Kafka Schema Registry
  * Hortonworks Schema Registry
  * and more
As we need to have additional services, for this demo we will be passing schema hard coded instead of leveraging schema registry.

## Passing Avro Schema
Let us understand how to pass hard coded schema for controller services related to file formats.
* We can directly place the schema as Schema Text.
* We can also add as variable.
* As we configure schema with processors like ConvertRecord, the schema will be applied for each and every record.
* For ConvertRecord, we can apply schema both while reading as well as writing.
* In case of CSV, we can pass the schema as header.

## CSV to Parquet with Schema
Parquet is one of the industry standard file format that is used in Data Lakes or Big Data Clusters. Let us understand how we can convert our CSV data to Parquet with Schema.
* Why Schema when we have header?
  * The column names are not same in all the files.
  * By using Schema we can make sure that all the records have same column names when we save in Parquet format.
* Null values are not represented uniformly across all the files as highlighted before. We need to ensure it is taken care while converting our data into Parquet file format.
* Here are the details about the processors.
  * ListFile - list the files in the local file system on the server where NiFi is running. Add filter to process the files belonging to **2019**.
  * UpdateAttribute - capture the original path and file name (without extension) so that we can standardize the naming convention.
  * FetchFile - fetch the files so that the files are converted downstream to JSON and placed in HDFS.
  * UnpackContent - as files are zipped, we need to unzip before we convert the file format.
  * RouteOnAttribute - to ignore system folders or files (folders with name **__MACOSX**)
  * ConvertRecord - processor to convert each and every record in input file to target format (Parquet)
    * We will use the Avro Schema instead of using Header for Schema. Avro Schema will be defined as a variable with in the processor group.
    * We will have 2 ConvertRecord Processors to deal with 2 types of Null values.
  * UpdateAttribute - to set the filename with appropriate file name and right extension.
  * PutHDFS - to place the files in HDFS.
