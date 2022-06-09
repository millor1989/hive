## [Hive Tutorial](https://cwiki.apache.org/confluence/display/Hive/Tutorial)

Hive是基于Apache Hadoop的数据仓库基础架构。Hive用于基于普通商用设备的数据存储和处理，并支持大规模的扩展和容错。

Hive支持对大规模的数据进行汇总（summarization），即席（ad-hoc）查询和分析。Hive提供了SQL，让用户可以很简单的对数据进行汇总（summarization），即席查询和分析。同时，Hive SQL还支持集成用户自定的一些分析功能，比如 UDFs（User Defined Functions）。

Hive 不是用来进行在线事务处理的，它的最佳用途是传统的数据建仓任务。

### 1、数据单元（Data Units）

Hive 的数据按照粒度大小被组织为：

* **Databases**：数据库，用于避免表（tables）、视图（views）、分区（partitions）、字段（columns）等命名冲突的命名空间（namespaces）。数据库也可以用来确保一个或一组用户的安全性。
* **Tables**：具有相同schema的同质（homogeneous）的数据单元。
* **Partitions**：每个表可以有一个或者多个分区键，用来决定如何保存数据。分区——除了是数据存储单元——还能让用户高效地定位到满足特定条件（specified criteria）的记录。使用分区可以显著地提高分析查询的速度。分区字段是虚拟字段，分区不是数据的一部分，而是数据加载时派生的。分区不是Hive根据数据划分的，而是用户决定的，分区和数据的对应意义要由用户来确保。
* **Buckets**（**Clusters**）：每个分区的数据可以基于某些字段的哈希函数值划分为桶（buckets）。可以用来高效的对数据进行抽样。

对表进行分区或者分桶不是必须的，但是，分区或者分桶能够对大量数据进行精简从而提高查询效率。

### 2、类型（Types）

Hive支持基本（primitive）类型和复杂（complex）数据类型。

#### 2.1、基本类型

* 类型与表的字段相关。如下是Hive支持的基本类型
* 整型
  * TINYINT—1 字节整数
  * SMALLINT—2 字节整数
  * INT—4 字节整数
  * BIGINT—8 字节整数
* 布尔类型
  * BOOLEAN—TRUE/FALSE
* 浮点类型数值
  * FLOAT—单精度
  * DOUBLE—双精度
* 定点数（Fixed point numbers）
  * DECIMAL—用户定义权重（scale）和精度（precision）的定点值
* 字符串类型
  * STRING—字符串
  * VARCHAR—有最大长度的字符串
  * CHAR—有固定长度的字符串
* 日期和时间类型
  * TIMESTAMP — 不带时区的日期和时间（Java "LocalDateTime" 语义）
  * TIMESTAMP WITH LOCAL TIME ZONE — 精确到纳秒的时间点 （Java "Instant" 语义）
  * DATE—日期
* 二进制类型
  * BINARY—字节序列

类型的层级如下：

```
Primitive Type
    Number

        DOUBLE

            FLOAT

                BIGINT

                    INT

                        SMALLINT

                            TINYINT

            STRING

    BOOLEAN
```

低层级的类型可以隐式地转换为高层几的类型；比如String类型可以隐式地转换为Double类型。

#### 2.2、Complex类型

复杂类型，可以基于基本类型和其它复合类型，使用如下方式构建：

* Structs：使用点“.”号可以访问Struct内部的元素。比如，类型为`STRUCT{a INT;b INT}`的字段c，可以使用`c.a`访问它的属性`a`；
* Maps\(key-value tuples\)：使用\['element name'\]访问Map内部的元素。比如，由映射'group'-&gt;gid构成的Map M，可以使用M\['group'\]访问gid的值；
* Arrays\(indexable lists\)：数组中的元素必须是相同类型。可以使用从0开始的索引访问数组中的元素。比如，有数组A，`['a','b','c']`，A\[1\]返回'b'。

使用基本类型和用于创建复杂类型的构造，可以创建任意嵌套级别的复杂类型。

#### 2.3、时间戳

##### 2.3.1、TimeStamp（“LocalDateTime”语义）

Java的`LocalDateTime`是一个记录日期和时间的时间戳，包含年、月、日、时、分、秒，不带时区。这些时间戳不考虑当地时区，总是有相同的值。（※MySQL的`TimeStamp`是包含时区的）

##### 2.3.2、TimeStamp with local time zone（“Instant”语义）

Java的`Instant`时间代表了一个UTC时的时间点。读取的时候加上当地的时区。

### 3、内置运算符和函数

在Beeline或者Hive CLI中运行如下命令可以显示支持的函数和运算符：

```
SHOW FUNCTIONS;
DESCRIBE FUNCTION &lt;function_name&gt;;
DESCRIBE FUNCTION EXTENDED &lt;function_name&gt;;
```

**※** Hive的所有关键字都是大小写不敏感的，包括Hive运算符和函名。

#### 3.1、内置运算符

- **关系运算符**：基于比较结果返回true或false

  

  | 关系运算符      | 操作数类型   | 描述                                                         |
  | --------------- | ------------ | ------------------------------------------------------------ |
  | A `!=` B        | 所有基本类型 | 不等                                                         |
  | A `<` B         | 所有基本类型 | 小于                                                         |
  | A `<=` B        | 所有基本类型 | 小于等于                                                     |
  | A `=` B         | 所有基本类型 | 等于                                                         |
  | A `>` B         | 所有基本类型 | 大于                                                         |
  | A `>=` B        | 所有基本类型 | 大于等于                                                     |
  | A `IS NOT NULL` | 所有类型     | 不是NULL                                                     |
  | A `IS NULL`     | 所有类型     | 是NULL                                                       |
  | A `LIKE` B      | 字符串类型   | 字符串A是否匹配SQL简单正则表达式B。比较是逐字符进行的“\_”匹配一个任意字符，“%”匹配任意数量的字符。如果要匹配字符“%”或者“\\”，可以使用“\\”进行转义。 |
  | A `REGEXP` B    | 字符串类型   | A或B，如果有一个是NULL，则返回NULL；A的任意子字符串（可以是空串）是否匹配Java正则表达式B。 |
  | A `RLIKE` B     | 字符串类型   | 与`REGEXP`相同                                               |

- **算术运算符**：返回数值类型

  

  | 算术运算符 | 操作数类型   | 描述                                                         |
  | ---------- | ------------ | ------------------------------------------------------------ |
  | A `%` B    | 所有数值类型 | 取余。结果类型是A、B类型的父类型                             |
  | A `+` B    | 所有数值类型 | 加法。结果类型是A、B类型的父类型                             |
  | A `&` B    | 所有数值类型 | 位与。结果类型是A、B类型的父类型                             |
  | `~`A       | 所有数值类型 | 按位取反。结果类型是A的类型                                  |
  | A &#124;&#124; B | 所有数值类型 | 位或。结果类型是A、B类型的父类型                             |
  | A `^` B    | 所有数值类型 | 按位异或。结果类型是A、B类型的父类型                         |
  | A `/` B    | 所有数值类型 | 除法。结果类型是A、B类型的父类型，如果操作数都是整数，结果是除法的商。 |
  | A `*` B    | 所有数值类型 | 乘法。结果类型是A、B类型的父类型，但是如果乘积会导致溢出，必须把其中一个的类型转换为更高层级的类型。 |
  | A `-` B    | 所有数值类型 | 减法。结果类型是A、B类型的父类型                             |

- **逻辑运算符**：返回true或false

  

  | 逻辑运算符       | 操作数类型 | 描述 |
  | ---------------- | ---------- | ---- |
  | A `AND` B        | boolean    | 与   |
  | A `&&` B         | boolean    | 与   |
  | A `OR` B         | boolean    | 或   |
  | A &#124;&#124; B | boolean    | 或   |
  | `NOT` A          | boolean    | 非   |
  | `!` A            | boolean    | 非   |

- **复杂类型运算符**：用于访问复杂类型的元素

  
  
  | 运算符   | 操作数类型                     | 描述                                         |
  | -------- | ------------------------------ | -------------------------------------------- |
  | A`[n]`   | A为`Array`，n为整数            | 返回数组A的第n个元素，n为0表示数组的第一个值 |
  | M`[key]` | M为`Map<K, V>`，key的类型是`K` | 返回map中与key对应的值                       |
  | s`.x`    | S是一个`Struct`                | 返回S的x属性                                 |

除了`IS NULL`和`IS NOT NULL`，值`NULL`参与任何运算，返回值都是`NULL`。

#### 3.2、内置函数

- Hive支持的内置函数（源码`FunctionRegistry.java`中的函数）：

  

  | 函数名                                              | 返回值类型 | 描述                                                         |
  | --------------------------------------------------- | ---------- | ------------------------------------------------------------ |
  | round(double a)                                     | BIGINT     | 返回a小数点后一位四舍五入取整的结果                          |
  | floor(double a)                                     | BIGINT     | 返回比小于等于a的最大整数                                    |
  | ceil(double a)                                      | BIGINT     | 返回大于等于a的最小整数                                      |
  | rand()，rand(int seed)                              | double     | 返回一个随机数（行与行不同）。指定`seed`能够确保生成的随机数序列是确定的。 |
  | concat(string A, String B, ...)                     | string     | 返回将B连接到A后的字符串，可以接受任意数量的字符串，并依次连接。 |
  | substr(string A, int start)                         | string     | 从指定的位置开始截取字符串                                   |
  | substring(string A, int start, int length)          | string     | 从指定位置开始截取指定长度的字符串                           |
  | upper(string A)                                     | string     | 将字符串转为大写                                             |
  | ucase(string A)                                     | string     | 同`upper`                                                    |
  | lower(string A)                                     | string     | 将字符串转为小写                                             |
  | lcase(string A)                                     | string     | 同`lower`                                                    |
  | trim(string A)                                      | string     | 去除字符串两端的空格                                         |
  | ltrim(string A)                                     | string     | 去除字符串左边空格                                           |
  | rtrim(string A)                                     | string     | 去除字符串右边空格                                           |
  | regexp_replace(string A, string B, string C)        | string     | 用字符串C替换字符串A中与Java正则表达式B匹配的所有子字符串。  |
  | size(Map&lt;K, V>)                                    | int        | 返回Map类型的元素个数                                        |
  | size(Array&lt;T>)                                     | int        | 返回数组类型的元素个数                                       |
  | cast(&lt;expr> as &lt;type>)                            | &lt;type>    | 将表达式expr转换为type类型；转换失败则返回`null`             |
  | from_unixtime(int unixtime)                         | string     | 将从UNIX epoch（1970-01-01 00:00:00 UTC）开始的秒数转换为当前系统时区的时间戳字符串表示形式 |
  | to_date(string timestamp)                           | string     | 返回时间戳字符串的日期部分                                   |
  | year(string date)                                   | int        | 返回日期或时间戳字符串的年                                   |
  | month(string date)                                  | int        | 返回日期或时间戳字符串的月                                   |
  | day(string date)                                    | int        | 返回日期或时间戳字符串的日                                   |
  | get\_json\_object(string json\_string, string path) | string     | 从json_string中提取指定json path的json对象字符串             |

- 内置聚合函数

  
  
  | 聚合函数名                        | 返回值类型 | 描述                                           |
  | --------------------------------- | ---------- | ---------------------------------------------- |
  | count(\*)，count(1)               | BIGINT     | 返回获取的所有行的数量，包括值为`NULL`的行     |
  | count(expr)                       | BIGINT     | 返回使表达式`expr`不为`NULL`的行的数量         |
  | count(DISTINCT expr\[,expr\_\.\]) | BIGINT     | 返回使表达式`expr`唯一并且不为`NULL`的行的数量 |
  | sum(col)                          | double     | 分组中所有元素的累加和                         |
  | sum(DISTINCT col)                 | double     | 分组中所有不同元素的累加和                     |
  | avg(col)                          | double     | 分组中所有元素的平均值                         |
  | avg(DISTINCT col)                 | double     | 分组中所有不同元素的平均值                     |
  | min(col)                          | double     | 分组中的最小值                                 |
  | max(col)                          | double     | 分组中的最大值                                 |

### 4、Hive SQL

Hive SQL的基本操作：

- WHERE子句过滤
- SELECT子句筛选某些字段
- 两个表的等值连接（equi-join）
- 与“group by”组合使用聚合
- 把查询结果保存到另一个表
- 把表的内容下载到本地（比如，nfs）目录
- 把查询结果保存到一个hadoop dfs目录
- 管理表和分区（create，drop，alter）
- 在SQL中引入自定义的脚本用于map/reduce jobs

