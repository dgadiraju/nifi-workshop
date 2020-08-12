# Converting File Formats

As part of data ingestion process, we might have to convert file format with out performing any transformations over data. In this module we will explore options in NiFi about File Format conversion.

* Pre-requisites
* General Guidelines
* Setp and Analyze Data
* Design NiFi Flow
* Configuring Reader and Writer
* Converting CSV to JSON
* Overview of Schema Registry
* Passing Avro Schema

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
mkdir -p /data/citibike && cd /data/citibike
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
  * Once you analyze, you can cleanup the uncompressed files by saying `rm *.csv`.
* Here are the details about our data.
  * Data contains header.
  * Underlying file names after unzipping following different naming convention.
  * There are some system files generated and they are supposed to be ignored before ingestion of data into the Data Lake or Data Hub.

## Design NiFi Flow
Let us come up with all the processors that are required to get the data from CSV to JSON using citibike data.
* ListFile - list the files in the local file system on the server where NiFi is running.
* FetchFile - fetch the files so that the files are converted downstream to JSON and placed in HDFS.
* UnpackContent - as files are zipped, we need to unzip before we convert the file format.
* ConvertRecord - processor to convert each and every record in input file to target format (JSON)
* UpdateAttribute - to set the filename with appropriate file name and right extension.
* PutHDFS - to place the files in HDFS.

## Configuring Reader and Writer
Let us understand how to configure the reader and writer while converting the file formats before ingesting data into the data lake.
* We can use CSVReader controller service to read the data from CSV Files.
* We need to configure the controller service based up on the observations.
  * Schema Access Strategy - Infer Schema
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
## Overview of Schema Registry
## Passing Avro Schema
