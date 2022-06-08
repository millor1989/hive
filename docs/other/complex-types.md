### 复杂类型的使用

以 Hive 的 `array` 和 `map` 类型为例。

`array` 字面量：

```sql
hive> SELECT array(1,2,3);

OK
_c0
[1,2,3]
```

`map` 字面量：

```sql
hive> select array(map('123',1),map('234',2));

OK
_c0
[{"123":1},{"234":2}]
```

创建一个表：

```sql
CREATE TABLE `just_a_test`(
  `id` int,
  `am` string)
```

表中插入一些数据，之后，查看：

```sql
hive> select * from just_a_test;
OK
just_a_test.id  just_a_test.am
2       123
1       hello moto
```

插入复杂类型值时，报错：

```sql
hive> insert into just_a_test(id,am)values(2,array(map('hello',0)));

FAILED: SemanticException [Error 10293]: Unable to create temp file for insert values Expression of type TOK_FUNCTION not supported in insert/values
```

其实，这里不是 Hive 表字段格式的原因，Hive 是读时验证格式的（read on schema）。应该是因为执行复杂类型插入时，用到了 `TOK_FUNCTION` 函数，而`insert values` 表达式不支持使用这个函数。

该用 `insert ... select ... from ...` 的插入方式：

```
insert into just_a_test(id,am) select 2,array(map('hello',0)) from (select 'a') a;

OK
```

插入成功，查看结果：

```sql
hive> select * from just_a_test;

OK
just_a_test.id  just_a_test.am
2       123
2       hello0
1       hello moto
```

插入的复杂类型值，展示也只是一个字符串。

更改 `am` 字段的类型：

```sql
hive> ALTER TABLE just_a_test REPLACE COLUMNS(id int, am ARRAY<MAP<string, INT>>);

OK
```

查看结果：

```
hive> select * from just_a_test;

OK
just_a_test.id  just_a_test.am
2       [{"123":null}]
2       [{"hello":0}]
1       [{"hello moto":null}]
```

其中，值 `hello0` 变成了应有的格式 `[{"hello":0}]`。

```
hive> SELECT id,am[0] from just_a_test;

2       {"123":null}
2       {"hello":0}
1       {"hello moto":null}
```

Impala 操作复杂类型，好像与 Hive 差别比较大。更改了 `am` 字段类型为复杂类型之后，不能在 Impala 中读取，报如下错误：

```
NotImplementedException: Scan of table 'bigdata_motor.just_a_test' in format 'TEXT' is not supported because the table has a column 'am' with a complex type 'ARRAY<MAP<STRING,INT>>'. Complex types are supported for these file formats: PARQUET.
```

大概是 Impala 在表存储格式为 `TEXT` 时不支持复杂类型，`PARQUET` 格式时才支持复杂类型。

但是，创建了 `PARQUET` 格式表后，又报如下错：

```sql
impala> SELECT id,am from bigdata_motor.just_a_test_1;

AnalysisException: Expr 'am' in select list returns a complex type 'ARRAY<MAP<STRING,INT>>'. Only scalar types are allowed in the select list.
```

