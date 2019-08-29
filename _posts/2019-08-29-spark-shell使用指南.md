---
layout:     post
title:      spark-shell使用指南.
subtitle:   博客文章
date:       2019-08-29
author:     Hybot
header-img: img/post-md.jpg
catalog: true
tags:
    - spark
---

在2.0版本之前，Spark的主要编程接口是RDD（弹性分布式数据集），在2.0之后，则主推Dataset，他与RDD一样是强类型，但更加优化。RDD接口仍然支持，但为了更优性能考虑还是用Dataset的好。

在spark目录中运行bin/spark-shell，或将spark安装目录设为SPARK_HOME环境变量且将其$SPARK_HOME/bin加到PATH中，则以后可在任意目录执行spark-shell即可启动。

RDD可以从Hadoop的InputFormats文件（如hdfs文件）创建，也可读写本地文件，也可由其他RDD经转换而来。Dataset也具有这些性质。以读取文件为例，RDD时代可以在shell中通过sc.textFile(filename)直接读取，在Dataset则需要通过spark.read.textFile(filename)读取。

## 1. 读取Dataset方式

`val dataset = spark.read.textFile(source_path)`

其中`spark.read`返回的是一个`DataFrameReader`，所以上述方法其加载文本文件并返回一个string的Dataset，这个dataset仅包含单个名为”value”的列。
若文本文件的目录结构包含分区信息，在读到的dataset中也将被忽略，要想将这些分区信息作为schema列信息的话，需要用text API, 看textFile的实现，
其也是用的text的特殊参数。

### 1.1 查看内容

`dataset.collect().foreach(println)` 或者 `dataset.take(10).foreach(println)`

其中collect返回所有记录，take(n)返回n条记录。


## 2. 读取Json为dataset并进行select操作

`val dataset = spark.read.json(source_path)`

spark.read.json可以返回DataFrame形式的数据

`val data = dataset.select($"content", $"id", $"time").filter($"id"===01 && $"time"="2019-01-01")`

返回 `org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [content: string, gid: bigint ... 1 more field]`

`val dataC = data.select(unbase64($"content")).map(s => new String(s.getAs[Array[Byte]](0), "gb2312"))`

将content中的base64的内容解码为gb2312

`val sample = dataC.take(10).foreach(println)`

输出

