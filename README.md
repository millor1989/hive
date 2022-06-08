### [Apache Hive](https://cwiki.apache.org/confluence/display/Hive/Home)

Apache Hive是用来读、写、管理保存在分布式存储中的大型数据集，并且使用SQL语法进行查询。

基于Apache Hadoop构建，Hive有如下特性：

* 使用SQL进行数据访问的工具，可以满足数据建仓的任务，包括ETL（extract/transform/load），报告和数据分析；
* 将结构强加于多种数据格式的机制；
* 访问Apache HDFS中，或者其它数据存储系统（比如Apache HBase）保存的数据；
* 通过Apache Tez，Apache Spark或者MapReduce执行查询；
* 使用HPL-SQL的（procedural language，存储过程处理语言？）；
* 通过Hive LLAP，Apache YARN和Apache Slider进行亚秒级（sub-second）查询。

Hive提供了标准的SQL支持，包括随后用于分析的SQL：2003，SQL：2011，和SQL：2016特性。

Hive的SQL可以通过UDFs（user defined functions）、UDAFs（user defined aggregates functions）、UDTFs（user defined table functions）使用用户代码进行扩展。

Hive支持多种数据格式。Hive内置了CSV、TSV、Apache Parquet、Apache ORC和其它格式的连接器（connectors）。用户可以使用其它格式的连接器对Hive进行扩展。

Hive不是为联机事务处理（OLTP）工作而设计的，最佳的用途是传统的数据建仓任务。

Hive是可扩展、高性能、容错、与输入格式低耦合的

Hive的组件包括HCatalog和WebHCat：

* HCatalog：是Hadoop的一个表和存储管理层，让用户可以使用不同的数据处理工具——包括Pig和MapReduce——更简单地进行数据读写。
* WebHCat：提供可以用来运行Hadoop MapReduce（或者YARN）、Pig、Hive jobs的服务。也可以使用HTTP接口进行Hive metadata操作。



