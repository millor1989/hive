### 操作符和函数（Operators and Functions）

Hive的所有关键字都是大小写不敏感的。

如下命令可以查看Hive的内置函数，或者内置函数的描述：

```
SHOW FUNCTIONS;
DESCRIBE FUNCTION <function_name>;
DESCRIBE FUNCTION EXTENDED <function_name>;
```

#### 1、内置运算符

##### 1.1、运算符优先级

- 元素选择器：`[]`、`.`
- 一元前缀操作符：`+`，`-`，`~`
- 一元后缀：`IS [NOT] (NULL|TRUE|FALSE)`
- 按位异或：A `^` B
- 乘法操作符：`*`、`/`、`%`、`DIV`
- 加法操作符：`+`、`-`
- 字符串连结：`||`
- 按位与：`&`
- 按位或：`|`

##### 1.2、关系运算符

除了`<=>`和`IS [NOT] (TRUE|FALSE)`操作符，其它关系操作符与`NULL`比较的结果都是`NULL`。

- `=`、`==`：相等
- `<=>`：相等比较，如果两端都为`NULL`返回TRUE
- `<>`、`!=`：不等
- `<`、`<=`、`>`、`>=`
- `[NOT] BETWEEN AND`

以上类型适用于所有的基本类型。

- `IS [NOT] NULL`：适用于所有类型；
- `IS [NOT] (TRUE|FALSE)`：使用于BOOLEAN类型，如果为`NULL`（`NULL`是`UNKNOWN`）则返回FALSE；
- `[NOT] LIKE`：SQL简单正则表达式匹配，只支持`%`和`_`两种通配符。
- `RLIKE`、`REGEXP`：Java正则表达式匹配

##### 1.3、算术运算符

除了`DIV`运算符，其它的可以用于所有的数值类型。

- `+`、`-`：加、减，结果的类型是操作符两端类型的共同父类型。
- `*`：乘，结果的类型是操作符两端类型的共同父类型，但是，如果乘的结果移除，就需要把其中的某一个操作数强转为类型层级中更高层级的类型。
- `/`：除法，大多数情况下，返回结果都是double。如果`hive.compat`配置参数设置为’0.13‘或’latest‘，则返回结果是decimal类型。
- `DIV`：除法，取整，仅适用于整数类型。
- `%`：取余，结果的类型是操作符两端类型的共同父类型。
- `&`、`|`、`^`、`~`：位运算，与、或、异或、取反；

##### 1.4、逻辑运算符

`AND`、`OR`、`NOT`、`!`、`[NOT] IN`、`[NOT] EXISTS(subquery)`

##### 1.5、字符串操作符

`||`：字符串连结，`concat(A,B)`的快捷方式

##### 1.6、复杂类型构造器

`map(key1, value1, key2, value2, ...)`

`struct(val1, val2, val3, ...)`：Struct属性名是col1、col2...

`named_struct(name1, val1, name2, val2, ...)`

`array(val1,val2,...)`

`create_union(tag,val1,val2,...)`

#### 2、内置函数

