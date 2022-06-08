## [GettingStarted](https://cwiki.apache.org/confluence/display/Hive/GettingStarted)

### 1、安装和配置

##### 需要：

- **Java**：推荐使用java 8
- **Hadoop**：推荐使用Hadoop 2.x
- 一般Linux或者Mac系统，Windows会有些不同

##### 安装：

- 设置环境变量`HIVE_HOME`指向安装目录

  ```
  export HIVE_HOME=<hive-install-dir>
  ```

- 添加`$HIVE_HOME/bin`到`PATH`

  ```
  export PATH=$HIVE_HOME/bin:$PATH
  ```

#### 1.1、运行

Hive依赖Hadoop：

- `PATH`中必须有Hadoop，或者
- `export HADOOP_HOME=<hadoop-install-dir>`

此外，创建Hive表之前必须用HDFS命令创建`/tmp`和`/user/hive/warehouse`（即，`hive.metastore.warehouse.dir`）并且用`chmod g+w`为它们设置权限。

```
  $ $HADOOP_HOME/bin/hadoop fs -mkdir       /tmp
  $ $HADOOP_HOME/bin/hadoop fs -mkdir       /user/hive/warehouse
  $ $HADOOP_HOME/bin/hadoop fs -chmod g+w   /tmp
  $ $HADOOP_HOME/bin/hadoop fs -chmod g+w   /user/hive/warehouse
```

设置HIVE_HOME可能不必要，但是很有用：

```
export HIVE_HOME=<hive-install-dir>
```

##### 运行Hive CLI

```
 $ $HIVE_HOME/bin/hive
```

##### 运行HiveServer2（HS2）和Beeline

从Hive 2.1开始，需要运行*schematool*命令作为初始步骤。比如，可以使用”derby“作为db类型：

```
  $ $HIVE_HOME/bin/schematool -dbType <db type> -initSchema
```

HiveServer2是Hive的一个服务接口，远程客户端通过HS2可以对Hive进行查询并获得结果。HS2是HiveServer1（已经废弃）的接替者。HS2支持多客户端并发和认证（authentication），它被用于为JDBC和ODBC这类开放API提供更好的支持。它的核心是基于Thrift（跨平台的RPC框架）的Hive服务。运行HiveServer2和Beeline：

```
  $ $HIVE_HOME/bin/hiveserver2

  $ $HIVE_HOME/bin/beeline -u jdbc:hive2://$HS2_HOST:$HS2_PORT
```

Beeline是HS2的CLI（命令行接口）。为了支持Beeline，HiveCLI（不支持多用户，缺乏安全性）已经被废弃了。运行Beeline的指令通过HS2的JDBC URL启动Beeline，需要依赖HS2启动的地址和端口（默认`localhost:10000`）。

为了测试，可以在同一个进程中启动Beeline和HiveServer2，用户体验与HiveCLI类似：

```
 $ $HIVE_HOME/bin/beeline -u jdbc:hive2://
```

##### 运行HCatalog

##### 运行WebHCat

#### 1.2、配置管理概览

- Hive默认从`<install-dir>/conf/hive-default.xml`获取配置

- 通过设置环境变量`HIVE_CONF_DIR`可以改变Hive配置目录的位置

- 通过`<install-dir>/conf/hive-default.xml`改变配置变量

- Log4j配置在 `<install-dir>/conf/hive-log4j.properties`中

- Hive配置基于Hadoop——它默认继承了Hadoop的配置变量

- 通过如下途径可以修改Hive配置：

  - 修改`hive-site.xml`，可以定义任何变量（包括Hadoop变量）

  - 使用`set`命令

  - 使用Hive（废弃的），Beeline或者HiveServer2的配置语法：

    ```
    //this sets the variables x1 and x2 to y1 and y2 respectively
    $ bin/hive --hiveconf x1=y1 --hiveconf x2=y2  
    
    //this sets server-side variables x1 and x2 to y1 and y2 respectively
    $ bin/hiveserver2 --hiveconf x1=y1 --hiveconf x2=y2  
    
    //this sets client-side variables x1 and x2 to y1 and y2 respectively.
    $ bin/beeline --hiveconf x1=y1 --hiveconf x2=y2  
    ```

  - 设置`HIVE_OPTS`环境变量为`--hiveconf x1=y1 --hiveconf x2=y2 `

#### 1.3、运行时配置

- Hive查询使用map-reduce查询执行，因而这些查询行为可以通过Hadoop配置变量控制

- HiveCLI（废弃的）和Beeline的`SET`命令可以用来设置任何Hadoop（或Hive）配置变量。比如：

  ```
      beeline> SET mapred.job.tracker=myhost.mycompany.com:50030;
      beeline> SET -v;
  ```

  `SET -v`命令显示所有的当前设置。如果不使用`-v`选项，则只有与基本的Hadoop配置不同的变量才会被展示

#### 1.4、Hive、Map-Reduce和Local-Mode

Hive编译器为大多数的查询生成map-reduce jobs。这些jobs会被提交到变量`mapred.job.tracker`所指的Map-Reduce集群。

Hadoop也提供了本地运行map-reduce jobs的选项，对于基于少量数据进行测试非常有用——这种情况下，本地模式执行通常比把jobs提交到大型集群更快。从版本0.7开始，Hive全面地支持**本地模式**运行，使用如下选项开启本地模式：

```
  hive> SET mapreduce.framework.name=local;
```

此外，`mapred.local.dir`应该指向本机的有效路径（比如，`tmp/<username>/mapred/local`），否则用户会得到一个分配本地磁盘空间的异常。

从版本0.7开始，Hive还支持一种**自动地以本地模式运行**map-reduce jobs的模式。相关的选项是 `hive.exec.mode.local.auto`， `hive.exec.mode.local.auto.inputbytes.max`，和 `hive.exec.mode.local.auto.tasks.max`：

```
hive> SET hive.exec.mode.local.auto=false;
```

注意，这种模式默认情况下是未开启的。如果开启自动地本地模式，Hive会分析查询的map-reduce job中数据的大小，如果满足如下条件，就会本地运行：

- job的全部输入数据大小小于`hive.exec.mode.local.auto.inputbytes.max`（默认128M）
- map-tasks的总数量小于`hive.exec.mode.local.auto.tasks.max` （默认4）
- reduce tasks的总数量是0或1

因此，对于少量数据的查询、或者对于多个map-reduce jobs查询中输入数据比较少（因为前面job的过滤或者reduction）的后续jobs，可能会本地的运行。

需要注意，Hadoop服务器节点运行时环境和运行Hive客户端的机器可能不同（因为不同的JVM版本或者不同的软件库）。本地模式运行时可能会导致意想不到的行为或错误。还要注意，本地模式执行是在（Hive客户端的）子JVM独立执行的。如果用户想要，可以通过选项`hive.mapred.local.mem`来控制子JVM的最大内存。它的默认值是0，即Hive让Hadoop决定子JVM的默认内存限制。

#### 1.5、Hive日志

Hive使用log4j记录日志，默认日志不会被CLI输出到控制台。版本0.13.0之前默认的日志级别是`WARN`。从Hive 0.13.0开始默认日志级别是`INFO`。

logs保存在*/tmp/&lt;user.name>/hive.log*中。通过设置*$HIVE_HOME/conf/hive-log4j.properties*中的`hive.log.dir`可以改变日志目录位置。要确保目录有适当的权限（`chmod 1777 <dir>`）。

- `hive.log.dir=<other_location>`

通过添加如下参数可以将Hive日志输出到控制台：

- `bin/hive --hiveconf hive.root.logger=INFO,console  ` // 用于HiveCLI (已经废弃)
- `bin/hiveserver2 --hiveconf hive.root.logger=INFO,console`

或者使用如下命令改变日志级别：

- `bin/hive --hiveconf hive.root.logger=INFO,DRFA ` 
- `bin/hiveserver2 --hiveconf hive.root.logger=INFO,DRFA`

另一个日志选项是通过`DAILY`选项提供`TimeBasedRollingPolicy`：

- `bin/hive --hiveconf hive.root.logger=INFO,DAILY` // 用于HiveCLI (已经废弃)
- `bin/hiveserver2 --hiveconf hive.root.logger=INFO,DAILY`

注意，通过`set`命令设置`hive.root.logger`不能改变日志属性，因为日志属性是在初始化时决定的。

Hive还基于Hive会话把查询日志保存在*/tmp/&lt;user.name>/*目录，通过修改*hive-site.xml*的`hive.querylog.location`属性可以更改这个目录。从Hive 1.1.0开始，将`hive.log.explain.output`属性设置为`true`可以用`INFO`级别记录`EXPLAIN EXTENDED`查询的输出。

Hive在Hadoop集群上执行期间的日志由Hadoop配置控制。通常Hadoop为每个map或reduce task在task执行的机器上保存一个日志文件。从Hadoop JobTracker Web UI的Task Details也可以查看这些日志文件。

使用本地模式时（`mapreduce.framework.name=local`），Hadoop/Hive执行日志会生成在客户端机器上。从版本0.6开始，Hive默认使用`hive-exec-log4j.properties`（如果丢失，则退回到使用`hive-log4j.properties`）来决定在哪儿保存这些日志。默认配置是，为每次本地模式查询产生一个日志文件并保存在*/tmp/&lt;user.name>/*目录。提供独立配置文件的目的是，让管理员可以集中保存（比如，保存到NFS文件服务器）执行日志。执行日志对于运行时错误的debug非常有用。

从Hive 2.1.0开始，Hive默认使用Log4j2的异步logger。设置`hive.async.log.enabled`为false可以关闭异步日志记录并退回到使用同步日志记录。异步日志记录使用的是用LMAX disruptor队列缓存日志信息的一个独立线程来记录日志，可以带来显著的性能提升。

##### HiveServer2 Logs

##### Audit Logs

Audit logs来自Hive metastore server每次metastore API的调用。

audit logs包含方法和相关方法的参数。以log4j的`INFO`级别记录，要确保`INFO`基本日志记录的开启。日志条目的名字是”HiveMetaStore.audit“。

##### Perf Logger

为了可以通过PerfLogger获取性能信息，需要将`PerfLogger`类的日志基本设置为`DEBUG`。在log4j属性文件中进行如下设置即可：

`log4j.logger.org.apache.hadoop.hive.ql.log.PerfLogger=DEBUG`

如果在根部将`hive.root.logger`设置为了`DEBUG`，上面的设置则没有必要了。

#### 1.6、DDL操作

创建表：

```sql
CREATE TABLE pokes (foo INT, bar STRING);
```

创建分区表：

```sql
CREATE TABLE invites (foo INT, bar STRING) PARTITIONED BY (ds STRING);
```

分区字段是一个虚拟字段（virtual column），它不是数据的一部分。

默认情况下，表的数据格式是text，定界符是^A（ctrl + a）。

查看所有的表：

```sql
SHOW TABLES;
```

查看表名以”s“结尾的表：

```sql
SHOW TABLES '.*s';
```

匹配样式（pattern）与java正则表达式一致。

查看表的字段：

```sql
DESCRIBE invites;
```

修改表：

```sql
--修改表名
ALTER TABLE events RENAME TO 3koobecaf;
--新增字段
ALTER TABLE pokes ADD COLUMNS (new_col INT);
ALTER TABLE invites ADD COLUMNS (new_col2 INT COMMENT 'a comment');
--替换字段
ALTER TABLE invites REPLACE COLUMNS (foo INT, bar STRING, baz INT COMMENT 'baz replaces new_col2');
```

`REPLACE COLUMNS`会替换所有的字段，并且只改变表的schema，不改变数据，表必须使用原生的（native）SerDe。`REPLACE COLUMNS`也可以用来删除表的字段：

```sql
ALTER TABLE invites REPLACE COLUMNS (foo INT COMMENT 'only keep the first column');
```

删除表：

```sql
DROP TABLE pokes;
```

##### Metadata Store

Metadata保存在一个嵌入式的Derby数据库，数据库的磁盘存储位置由Hive的配置变量`javax.jdo.option.ConnectionURL`决定，默认只是`./metastore_db`（查看`conf/hive-default.xml`）。使用默认配置，同时只能有一个用户访问metadata。

Metastore（metadata store）可以保存在JPOX支持的任意数据库中。RDBMS的位置和类型由变量`javax.jdo.option.ConnectionURL` 和`javax.jdo.option.ConnectionDriverName`控制。

metastore可以是一个独立的服务器。如果metastore运行为一个独立的网络服务器，那么可以同时被多个节点访问。

#### 1.7、DML

从一般文件加载数据：

```sql
LOAD DATA LOCAL INPATH './examples/files/kv1.txt' OVERWRITE INTO TABLE pokes;
```

`LOCAL`表示文件在本地文件系统中，如果没有`LOCAL`则在HDFS中查找指定的文件。

`OVERWRITE`表示表中已经存在的数据会被覆盖；如果没有`OVERWRITE`则是往表中追加数据。

- 加载数据时不会对数据进行schema验证；
- 如果加载的文件在HDFS中，则会把它移动到Hive控制的文件系统命名空间中，这时操作会非常快几乎是瞬时的。Hive的根目录由`hive-default.xml`文件中的`hive.metastore.warehouse.dir` 属性控制，建议在通过Hive创建表之前，创建这个目录。

往表的指定分区加载数据：

```sql
--加载本地文件到表分区
LOAD DATA LOCAL INPATH './examples/files/kv2.txt' OVERWRITE INTO TABLE invites PARTITION (ds='2008-08-15');
LOAD DATA LOCAL INPATH './examples/files/kv3.txt' OVERWRITE INTO TABLE invites PARTITION (ds='2008-08-08');
--加载HDFS文件到表分区
LOAD DATA INPATH '/user/myname/kv2.txt' OVERWRITE INTO TABLE invites PARTITION (ds='2008-08-15');
```

前提，当然是表的shcema包含分区。

#### 1.8、SQL操作

查询和过滤：

```sql
SELECT a.foo FROM invites a WHERE a.ds='2008-08-15';
```

从指定分区查询；查询的结果不会在任何地方保存，只会在控制台展示。

把查询结果保存到HDFS目录：

```sql
INSERT OVERWRITE DIRECTORY '/tmp/hdfs_out' SELECT a.* FROM invites a WHERE a.ds='2008-08-15';
```

Select子句中的”\*“**包含分区字段**。

把查询结果保存到本地目录：

```sql
INSERT OVERWRITE LOCAL DIRECTORY '/tmp/local_out' SELECT a.* FROM pokes a;
```

其它：

```sql
--保存查询结果到其它表
INSERT OVERWRITE TABLE events SELECT a.* FROM profiles a;
INSERT OVERWRITE TABLE events SELECT a.* FROM profiles a WHERE a.key < 100;
INSERT OVERWRITE LOCAL DIRECTORY '/tmp/reg_3' SELECT a.* FROM events a;
INSERT OVERWRITE DIRECTORY '/tmp/reg_4' select a.invites, a.pokes FROM profiles a;
-- 使用聚合函数
INSERT OVERWRITE DIRECTORY '/tmp/reg_5' SELECT COUNT(*) FROM invites a WHERE a.ds='2008-08-15';
INSERT OVERWRITE DIRECTORY '/tmp/reg_5' SELECT a.foo, a.bar FROM invites a;
INSERT OVERWRITE LOCAL DIRECTORY '/tmp/sum' SELECT SUM(a.pc) FROM pc1 a;
```

某些Hive版本可能不支持`count(*)`，那么可以用`count(1)`替代。

Group By：

```sql
FROM invites a INSERT OVERWRITE TABLE events SELECT a.bar, count(*) WHERE a.foo > 0 GROUP BY a.bar;

INSERT OVERWRITE TABLE events SELECT a.bar, count(*) FROM invites a WHERE a.foo > 0 GROUP BY a.bar;
```

这两个group by语句是等价的。

Join：

```sql
FROM pokes t1 JOIN invites t2 ON (t1.bar = t2.bar) INSERT OVERWRITE TABLE events SELECT t1.bar, t1.foo, t2.foo;
```

多表Insert：

```sql
FROM src
INSERT OVERWRITE TABLE dest1 SELECT src.* WHERE src.key < 100
INSERT OVERWRITE TABLE dest2 SELECT src.key, src.value WHERE src.key >= 100 and src.key < 200
--插入到指定分区
INSERT OVERWRITE TABLE dest3 PARTITION(ds='2008-04-08', hr='12') SELECT src.key WHERE src.key >= 200 and src.key < 300
INSERT OVERWRITE LOCAL DIRECTORY '/tmp/dest4.out' SELECT src.value WHERE src.key >= 300;
```

Streaming：

```sql
FROM invites a INSERT OVERWRITE TABLE events SELECT TRANSFORM(a.foo, a.bar) AS (oof, rab) USING '/bin/cat' WHERE a.ds > '2008-08-09';
```

`TRANSFORM`把查询结果转换为stream，经过脚本`/bin/cat`处理后执行插入操作。

此处是在map阶段使用了流，在reduce阶段也可以使用流。

- ##### 例子：

创建tab分隔得文本格式表：

```sql
CREATE TABLE u_data (
  userid INT,
  movieid INT,
  rating INT,
  unixtime STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE;
```

下载文件：`wget http://files.grouplens.org/datasets/movielens/ml-100k.zip`

解压：`unzip ml-100k.zip`

将解压后的`u.data`文件加载进创建的表：

```sql
LOAD DATA LOCAL INPATH '<path>/u.data'
OVERWRITE INTO TABLE u_data;
```

查看数据量：

```sql
SELECT COUNT(*) FROM u_data;
```

对`u_data`表进行一些复杂的数据分析：

创建`weekday_mapper.py`：

```python
import sys
import datetime

for line in sys.stdin:
  line = line.strip()
  userid, movieid, rating, unixtime = line.split('\t')
  weekday = datetime.datetime.fromtimestamp(float(unixtime)).isoweekday()
  print '\t'.join([userid, movieid, rating, str(weekday)])
```

mapper脚本使用：

使用脚本处理数据流：

```sql
CREATE TABLE u_data_new (
  userid INT,
  movieid INT,
  rating INT,
  weekday INT)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t';

--把文件上传到hive
add FILE weekday_mapper.py;

INSERT OVERWRITE TABLE u_data_new
SELECT
  TRANSFORM (userid, movieid, rating, unixtime)
  USING 'python weekday_mapper.py'
  AS (userid, movieid, rating, weekday)
FROM u_data;
```

##### Apache Weblog Data

Apache weblog的格式是可以自定义的，但是大多数的webmaster都使用默认的。对于默认的Apache weblog，可以使用如下的命令创建一个表：

```sql
CREATE TABLE apachelog (
  host STRING,
  identity STRING,
  user STRING,
  time STRING,
  request STRING,
  status STRING,
  size STRING,
  referer STRING,
  agent STRING)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
WITH SERDEPROPERTIES (
  "input.regex" = "([^]*) ([^]*) ([^]*) (-|\\[^\\]*\\]) ([^ \"]*|\"[^\"]*\") (-|[0-9]*) (-|[0-9]*)(?: ([^ \"]*|\".*\") ([^ \"]*|\".*\"))?"
)
STORED AS TEXTFILE;
```

*RegexSerDe* ？？？

