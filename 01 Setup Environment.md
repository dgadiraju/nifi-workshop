# Setup Environment

As part of this session we will see how to setup environment to practice NiFi in a single node cluster with Hadoop and Spark.

* Provision Server from AWS
* Setup Pre-Requisites
* Setup and Configure Hadoop
* Start and Validate Hadoop Components
* Setup and Validate Spark
* Setup and Start NiFi

The content will be free, however if you want to watch videos to understand key concepts feel free to [join our YouTube Channel as a member](https://www.youtube.com/channel/UCakdSIPsJqiOLqylgoYmwQg/join).

## Provision Server from AWS
Let us understand how to provision server from AWS quickly so that we can setup single node Big Data Cluster to practice key technologies such as Spark, NiFi, Kafka etc.

* Operating System - Centos 7
* Instance Type - **t2x.large** (4 vCores and 16 GB RAM).
* Root File System size - 60 GB
* Make sure elastic ip is configured as we will be stopping EC2 instance frequently.
* Make sure to understand the pricing.

Even though demo is given on AWS, you can use server with similar configuration from any cloud platform or bare metal servers.

## Setup Pre-Requisites
Let us setup some of the pre-requisites required to setup Hadoop, Spark, NiFi etc on a single node.
* Use `yum` to install JDK 1.8, git, wget, telnet etc.
```
sudo yum -y install python3 wget telnet java-1.8.0-openjdk-devel git
```
* JDK 1.8 is required by most of the Big Data technologies.
* **telnet** comes handy to troubleshoot whether the services are started using specific port number or not.
* **wget** will be used to download required softwares.
* **git** will be used to clone GitHub repositories of the data and code that is used as part of this course.

## Setup and Configure Hadoop
Let us understand how to download, setup and configure Hadoop on single node.
* Download from Hadoop.
* Configure HDFS.
* Here are the contents of **core-site.xml**.
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```
* Here are the contents of **hdfs-site.xml**.
```
<configuration>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/opt/hadoop/dfs/name</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/opt/hadoop/dfs/name</value>
    </property>
    <property>
        <name>dfs.namenode.checkpoint.dir</name>
        <value>/opt/hadoop/dfs/namesecondary</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/opt/hadoop/dfs/data</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```
* Configure YARN.
* Here are the contents of **yarn-site.xml**.
```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```
* Here are the contents of **mapred-site.xml**.
```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>
```
* Format HDFS so that directories for Namenode, Secondary Namenode as well as Datanode are created.
```
hdfs namenode -format
ls -ltr /opt/hadoop/dfs/
```

## Start and Validate Hadoop Components
Let us start and validate hadoop components.
* Ensure environment variables such as PATH are updated with bin folder of Hadooop.
* We can start HDFS components using `start-dfs.sh`.
* Run `jps` command to ensure **Namenode**, **Secondary Namenode** and **Datanode** are running.
* We can start YARN components using `start-yarn.sh`.
* Run `jps` command to ensure **Resource Manager** and **Node Mnager** are running.
* Create **/user/centos** folder structure or folder structure of your choice.
```
hdfs dfs -mkdir -p /user/centos
hdfs dfs -put /opt/hadoop/etc/hadoop /user/centos
hdfs dfs -cat /user/centos/core-site.xml
```

## Setup and Validate Spark
Let us understand how to setup and validate Spark. We will be using YARN to submit Spark Jobs for our practice.

* Download Spark 2.4.6 Version.
* Update Spark Configuration files.
* Validate Spark using Scala by running `spark-shell --master yarn --conf spark.ui.port=0`.
* Validate Spark using Python by running `pyspark --master yarn --conf spark.ui.port=0`. Make sure to export PYSPARK_PYTHON to point to Python 3 if the default version is 2.7.
```
export PYSPARK_PYTHON=python3
pyspark --master yarn --conf spark.ui.port=0
```
* Validate Spark SQL by running `spark-sql --master yarn --conf spark.ui.port=0`.

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
