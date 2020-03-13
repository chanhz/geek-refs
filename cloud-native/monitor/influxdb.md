InfluxDB 是一个用于存储和分析时间序列数据的开源数据库。

主要特性有：
- 内置HTTP接口，使用方便
- 数据可以打标记（tag），这样查询可以很灵活
- 类 SQL 的查询语句
- 安装管理很简单，并且读写数据很高效（LSM 结构）。
- 能够实时查询，数据在写入时被索引后就能够被立即查出


## InfluxDB 数据模型
1. Measurement：从原理上讲更像SQL中表的概念。这和其他很多时序数据库有些不同，其他时序数据库中Measurement可能与Metric等同，类似于下文讲到的Field，这点需要注意。


2. Tags：维度列

（1）上图中location和scientist分别是表中的两个Tag Key，其中location对应的维度值Tag Values为｛1, 2｝，scientist对应的维度值Tag Values为{langstroth, perpetual}，两者的组合TagSet有四种：
```
location = 1 , scientist = langstroth
location = 1 , scientist = perpetual
location = 2 , scientist = langstroth
location = 2 , scientist = perpetual
```
（2）在InfluxDB中，表中Tags组合会被作为记录的主键，因此主键并不唯一，比如上表中第一行和第三行记录的主键都为'location=1,scientist=langstroth'。所有时序查询最终都会基于主键查询之后再经过时间戳过滤完成。


3. Fields：数值列。数值列存放用户的时序数据。


4. Point：类似SQL中一行记录，而并不是一个点。


## Series =  Measurement + Tags
InfluxDB 中使用 Series 表示数据源，Series 由 Measurement 和 Tags 组合而成，Tags 组合用来唯一标识Measurement。Series 是 InfluxDB 中最重要的概念。

InfluxDB 使用 HTTP 接口查询数据。
```bash
curl -G 'http://localhost:8086/query?pretty=true' --data-urlencode "db=mydb" --data-urlencode "q=SELECT \"value\" FROM \"cpu_load_short\" WHERE \"region\"='us-west'"
```
查询返回的结果如下所示：
```json
{
    "results": [
        {
            "statement_id": 0,
            "series": [
                {
                    "name": "cpu_load_short",
                    "columns": [
                        "time",
                        "value"
                    ],
                    "values": [
                        [
                            "2015-01-29T21:55:43.702900257Z",
                            2
                        ],
                        [
                            "2015-01-29T21:55:43.702900257Z",
                            0.55
                        ],
                        [
                            "2015-06-11T20:46:02Z",
                            0.64
                        ]
                    ]
                }
            ]
        }
    ]
}
```

## Retention Policy（RP）-- 保留策略
指定数据的过期时间，指定数据副本数量以及指定 **ShardGroup Duration**。

InfluxDB中Retention Policy有这么几个性质和用法：

1. RP是数据库级别而不是表级别的属性。这和很多数据库都不同。

2. 每个数据库可以有多个数据保留策略，但只能有一个默认策略。

3. 不同表可以根据保留策略规划在写入数据的时候指定RP进行写入。


## Shard Group -- 时间段组
Shard Group 是InfluxDB中一个重要的逻辑概念，从字面意思来看 Shard Group 会包含多个Shard，每个Shard Group只存储指定时间段的数据，不同 Shard Group 对应的时间段不会重合。


为什么需要将数据按照时间分成一个一个Shard Group？个人认为有两个原因：

1. 将数据按照时间分割成小的粒度会使得数据过期实现非常简单，InfluxDB中数据过期删除的执行粒度就是Shard Group，系统会对每一个Shard Group判断是否过期，而不是一条一条记录判断。

2. 实现了将数据按照时间分区的特性。将时序数据按照时间分区是时序数据库一个非常重要的特性，基本上所有时序数据查询操作都会带有时间的过滤条件，比如查询最近一小时或最近一天，数据分区可以有效根据时间维度选择部分目标分区，淘汰部分分区。

## Shard
Shard Group实现了数据分区，但是Shard Group只是一个逻辑概念，在它里面包含了大量Shard，Shard才是InfluxDB中真正存储数据以及提供读写服务的概念，类似于HBase中Region，Kudu中Tablet的概念。关于Shard，需要弄清楚两个方面：

1. Shard是InfluxDB的存储引擎实现，具体称之为TSM(Time Sort Merge Tree) Engine，负责数据的编码存储、读写服务等。TSM类似于LSM，因此Shard和HBase Region一样包含Cache、WAL以及Data File等各个组件，也会有flush、compaction等这类数据操作。

2. Shard Group对数据按时间进行了分区，那落在一个Shard Group中的数据又是如何映射到哪个Shard上呢？

InfluxDB 采用了 Hash 分区的方法将落到同一个 Shard Group 中的数据再次进行了一次分区。这里特别需要注意的是，InfluxDB是根据hash(Series)将时序数据映射到不同的Shard，而不是根据Measurement进行hash映射，这样会使得相同Series的数据肯定会存在同一个Shard中，但这样的映射策略会使得一个Shard中包含多个Measurement的数据，不像HBase中一个Region的数据肯定都属于同一张表。


## 总结

本篇文章重点介绍InfluxDB中一些基本概念，为后面分析InfluxDB内核实现奠定一个基础。文章主要介绍了三个重要模块：

1. 首先介绍了InfluxDB中一些基本概念，包括Measurement、Tags、Fields以及Point。

2. 接着介绍了Series这个非常非常重要的概念。

3. 最后重点介绍了InfluxDB中数据的组织形式，总结起来就是：先按照RP划分，不同过期时间的数据划分到不同的RP，同一个RP下的数据再按照时间Range分区形成ShardGroup，同一个ShardGroup中的数据再按照Series进行Hash分区，将数据划分成更小粒度的管理单元。Shard是InfluxDB中实际工作者，是InfluxDB的存储引擎。下文会重点介绍Shard的工作原理。

## ref
- [时序数据库技术体系 – 初识InfluxDB by 范欣欣](http://hbasefly.com/2017/12/08/influxdb-1/)
- [More](http://hbasefly.com/category/%e6%97%b6%e5%ba%8f%e6%95%b0%e6%8d%ae%e5%ba%93/)


# InfluxDB TSM存储引擎之 TSMFile
在计算机科学中，预写式日志（Write-ahead logging，缩写 WAL）是关系数据库系统中用于提供原子性和持久性（ACID属性中的两个）的一系列技术。在使用WAL的系统中，**所有的修改在提交之前都要先写入log文件中**。

log文件中通常包括redo和undo信息。这样做的目的可以通过一个例子来说明。假设一个程序在执行某些操作的过程中机器掉电了。在重新启动时，程序可能需要知道当时执行的操作是成功了还是部分成功或者是失败了。如果使用了WAL，程序就可以检查log文件，并对突然掉电时计划执行的操作内容跟实际上执行的操作内容进行比较。在这个比较的基础上，程序就可以决定是撤销已做的操作还是继续完成已做的操作，或者是保持原样。

WAL允许用in-place方式更新数据库。另一种用来实现原子更新的方法是shadow paging，它并不是in-place方式。用in-place方式做更新的主要优点是减少索引和块列表的修改。ARIES是WAL系列技术常用的算法。在文件系统中，WAL通常称为journaling。PostgreSQL也是用WAL来提供point-in-time恢复和数据库复制特性。