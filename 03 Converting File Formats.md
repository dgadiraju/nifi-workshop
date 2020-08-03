# Converting File Formats

As part of data ingestion process, we might have to convert file format with out performing any transformations over data. In this module we will explore options in NiFi about File Format conversion.

* General Guidelines
* Analyzing Data
* Design NiFi Flow
* Configuring Reader and Writer
* Converting CSV to JSON
* Overview of Schema Registry
* Passing Avro Schema

The content will be free, however if you want to watch videos to understand key concepts feel free to [join our YouTube Channel as a member](https://www.youtube.com/channel/UCakdSIPsJqiOLqylgoYmwQg/join).

[![NiFi - Convert File Formats](http://img.youtube.com/vi/N2iggpbaw90/0.jpg)](http://www.youtube.com/watch?v=N2iggpbaw90 "NiFi - Convert File Formats")


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
  
## Analyzing Data
Let us analyze and understand characteristics of data before converting and loading data into HDFS.
* We will be using **/data/citibike/tripdata**
* Files are downloaded from [s3 bucket](https://s3.amazonaws.com/tripdata) provided by [official citibike](https://www.citibikenyc.com/system-data) website.
* Data is in the form of CSV and also compressed using Zip.
* Each file have header and we should be able to extract column names from the header.
* You can observe that there is one file per month.

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
