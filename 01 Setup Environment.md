# Setup Environment

As part of this session we will see how to setup environment to practice NiFi in a single node cluster with Hadoop and Spark.

* Provision Server from AWS
* Setup Pre-Requisites
* Setup and Configure Hadoop
* Start and Validate Hadoop Components
* Setup and Validate Spark
* Setup and Start NiFi

The content will be free, however if you want to watch videos to understand key concepts feel free to [join our YouTube Channel as a member](https://www.youtube.com/channel/UCakdSIPsJqiOLqylgoYmwQg/join).

All the instructions that are related to provisionsing server from AWS, Setup Hadoop, Spark etc are covered as part of [Setup Development Environment for Spark](https://github.com/dgadiraju/itversity-books/blob/master/Data%20Engineering%20Bootcamp/46%20Apache%20Spark%20using%20Python/00%20Setup%20Development%20Environment.md).

Here are the videos as well for instructions to setup Hadoop and Spark on Centos 7 based machine as a single node cluster.

[![Data Engineering - Setup Development Environment](http://img.youtube.com/vi/Z4pDhBaWB64/0.jpg)](http://www.youtube.com/watch?v=Z4pDhBaWB64 "Data Engineering - Setup Development Environment")

[![Data Engineering - Setup Development Environment](http://img.youtube.com/vi/jcdAO90yY2U/0.jpg)](http://www.youtube.com/watch?v=jcdAO90yY2U "Data Engineering - Setup Development Environment (Contd...)")

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

* In my case I have setup retail_db dataset under /data folder on the server where we have setup NiFi.
```
sudo mkdir /data
sudo chown -R centos:centos /data
git clone https://www.github.com/dgadiraju/retail_db.git /data/retail_db
```
