---
title: Data Lake — Insights about Data Fidelity
date: 2019-06-04 11:59:48
tags:
- DATA
- 
- Security
- VPC
categories:
- Data
cover_index: /assets/Data-lake-insights-about-data-fidelity.png
---

It’s been a few weeks since I was thinking about writing this article about data fidelity in a data lake. These insights result of some constraints I have encountered in my one-year-mission in a project of building a Data Lake of a Moroccan Banking Group.

In a Datalake, it is trivial to make difference between raw data and clean data. Raw data is not always ready to be used in decision making. That’s why RDBMS or NoSQL tables are used to extract efficient information from raw data.

However, a Data Lake should be able to ingest landing data in its full fidelity as much as possible. This is one of the most powerful features of Hadoop. In addition storing raw data regardless of its format is advantageous to play new processes and for upcoming Data Science use cases. Also, unprocessed data can be loaded with the imposed structure at processing time based on your job. It’s **Schema-on-Read**.

Hadoop has the advantage of processing structured and unstructured data of many formats (text, video, image ..) But we maybe using structured and flexibly structured data more than other formats. In my mission, flexibly structured data to which we refer using the term unstructured data, is data with fields that may change in time (add, remove …).

### Data in Hadoop

Although being able to store all of your raw data is a powerful feature, there are still many factors that you should take into consideration before dumping your data into Hadoop :	

#### File Format — to avoid

Hadoop supports many formats : Plain Text, SequenceFile, Avro, Parquet. Data format is important because choosing an improper format may produce a huge disk consumption in no time. Generally, some formats are to avoid. For example, XML and JSON formats because of single document. In addition, in Hadoop, there’s no in built Input format to handle them. In our case, we deal with Text Files. They’re often used to transport data between Hadoop and external systems. We have conventions with provider systems about the Text File’s Structure, its name and the frequency of reception. Yet, this format has limited support for schema evolution. That said, if a file’s structure has to change, existing fields should never be deleted and new ones can only be appended at the end of columns. Other formats are more complex but have more options, different strengths. At the end, it depends on the application and data provider systems to decide if a File Format is more or less suitable to use.

#### Use the right compressor
Large files in Hadoop can consume a lot f disk. Compression is one solution to consider while storing data in Hadoop. Choosing the compression type and tool often depends on the application’s specification :

#### Fast compression but not aggressive
Long compression storing smaller files (more CPU).
It is best for compressed files to be splittable because Hadoop splits files into blocks while compressing. It is important for parallel processing that means creating multiple parallelly running tasks which is one of Hadoop biggest advantages. Yet, compression format depends on how the data will be used. Also, if your file is smaller than the size of HDFS block, compression won’t have a great advantage. Still, Hadoop is more efficient when working with large files and small one can cause performance issues.

#### Data storage system
To my point of view, this point refers to the best way to store data : Raw or using Hbase/Hive/Impala. It is recommended to store data in its raw format. It will better represent the truth of your application’s sources.

### Data Modeling — Use case driven modeling

By Data Modeling, I mean the way files will be structured in HDFS.
Again, It depends on the convention made with data sources. For example, in my mission, we receive TXT files having this name pattern : **<TOPIC>_<DATE>.txt**
One way to midel data was to archive these files with this arborescence : **/data/raw/topic/filename.txt**.

![](https://cdn-images-1.medium.com/max/800/1*VXC8rvXBeUEM6kvE2rK4TQ.png)

Use cases should be taken into consideration when setting-up the Data Lake Layers. Ignoring this aspect or putting down a theoretical Model for Data risks of impacting other Use cases that required less or more Layers (Use cases that need Raw Data only …). That said, it is recommended to build the Model in an iterative way and keep the use case a first entry.

**Thanks for reading ❤ **


##### Author : 

[Rania ZYANE - Data Consultant @ Octo Technology.](https://github.com/Raniazy/)



