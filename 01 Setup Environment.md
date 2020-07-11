# Setup Environment

As part of this session we will see how to setup environment to practice NiFi in a single node cluster with Hadoop and Spark.

* Provision Server from AWS
* Setup Pre-Requisites
* Setup and Configure Hadoop
* Start and Validate Hadoop Components
* Setup and Validate Spark
* Setup and Start NiFi

## Provision Server from AWS

## Setup Pre-Requisites

## Setup and Configure Hadoop

## Start and Validate Hadoop Components

## Setup and Validate Spark

## Setup and Start NiFi

Here are the instructions to setup and start NiFi.

* Go to [NiFi Official Downloads Page](https://nifi.apache.org/download.html) and choose the version of your choice.
* Go to the mirror and copy the link.
* Login into the server provisioned and run `wget` command to download.
* Untar and unzip using `tar xzf` command.
* Copy the untarred folder to /opt and change the ownership to user such as centos or non root user you are using.
* Create softlink **/opt/nifi** for the versioned nifi folder.
* Export environment variables in the session as well as update **.bash_profile**.
* Here are the commands we have used for your reference. Make sure you update as per your environment.
```
wget https://mirrors.sonic.net/apache/nifi/1.11.4/nifi-1.11.4-bin.tar.gz
tar xzf nifi-1.11.4-bin.tar.gz
sudo mv -f nifi-1.11.4 /opt
sudo chown centos:centos -R /opt/nifi-1.11.4
sudo ln -s /opt/nifi-1.11.4 /opt/nifi
```

## Setup Datasets

Let us go ahead and setup retail_db dataset to get started with NiFi.

Please go to this [page](https://www.github.com/dgadiraju/retail_db.git) and take care of setting up retail_db data set.
