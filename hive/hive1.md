# 第一章：hive简介和核心概念



## 引子：存在的疑问和解决

- [ ] 结构化数据的分隔符在Hive数据表中如何设置？
- [ ] 灵活性高：自定义用户函数（UDF）和存储格式？(自定义存储格式是什么意思)
- [ ] 统一的元数据管理：可与presto   /     impala     /    sparksql    **共享元数据**，元数据（metastore ）的含义？
- [ ] 隐式转换需要理解去记忆









## 一、Hive简介

### 1.1是什么

hive类似数仓管理工具：Hive是建立在Hadoop之上的数据仓库，由Facebook开发，在某种程度上可以看成是用户编程接口，本身并不存储和处理数据，依赖于HDFS存储数据，依赖MR处理数据。有类SQL语言，它可以将结构化的数据文件映射成表，并提供类 SQL 查询功能，用于查询的 SQL 语句会被转化为 MapReduce 作业，然后提交到 Hadoop 上运行。

建立原因：传统数据仓库建立在关系型数据仓库之上，计算和处理能力不足，当数据量达到TB级后基本无法获得好的性能。





### 1.2hive与传统的数据库的对比

|    对比    | Hive                                                         |                          传统数据库                          |
| :--------: | :----------------------------------------------------------- | :----------------------------------------------------------: |
|  数据更新  | **不支持**                                                   |                             支持                             |
|  数据插入  | 支持批量插入                                                 |                             支持                             |
|    索引    | 有索引功能，不像RDBMS有键的概念，<br />可在某些列上建立索引，<br />实现枷锁一些查询效果。<br />创建的索数据会被保存到存在的另外的表中。 |                             支持                             |
|   分区列   | 支持                                                         |                             支持                             |
| 执行的延迟 | 高，hive构建在HDFS和MR上，<br />比传统数据库延迟要高         | 低，传统SQL语句执行延迟一般少于1秒，<br />而HSQL语句延迟可达分钟级 |
|   扩展性   | 好，基于Hadoop集群，有很好的**横向**扩展性                   | 有限，RDBMS（关系型数据库）非分布式，<br />横向扩展（分布式添加节点）难实现，<br />纵向（扩展内存、CPU等）也很有限 |

[RDBMS：Relational Database Management System]: https://zhuanlan.zhihu.com/p/61627651	"关系型数据库"



### 1.3为什么

传统数据仓库建立在关系型数据仓库之上，计算和处理能力不足，当数据量达到TB级后基本无法获得好的性能。

Hive可以：

- 结构化的数据文件映射成表（X.csv 文件映射成表文件---> shoolTable）
- 提供类SQL查询功能
- 查询的SQL语句会被转换成MapReduce作业，最后提交到Hadoop上运行

### 1.4Hive的特点

- **简单**：提供类SQL语言Hsql，让大家不使用javaAPI开发，而只是学习sql就行了
- 灵活性高：**自定义用户函数（UDF）和存储格式**？(自定义存储格式是什么意思)
- 为**超大的数据集**设计的计算（MapReduce）和存储能力（HDFS），集群扩展容易（Hadoop生态的横向扩展）
- 统一的元数据管理：可与presto   /     impala     /    sparksql    **共享元数据**
- **执行延迟高**：不适合数据实时处理，适合海量数据离线处理 



## 二、Hive的体系架构

### 2.1自顶向下的体系架构图

![img](https://camo.githubusercontent.com/d6fa9a13f7bd454285c88ad315dde1c8cb8d1a4840695cb1e726da8b8a863bbf/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f68697665e4bd93e7b3bbe69eb6e69e842e706e67)





### 2.2   command-line shell（命令行shell） &    thrift/jdbc   （JDBC连接）



启动hive：

```shell
cd /export/server/hive-2.1.0/bin

expect beeline.exp


# 您在 /var/spool/mail/root 中有新邮件
[root@node1 onekey]# cd /export/server/hive-2.1.0/bin
# 您在 /var/spool/mail/root 中有新邮件
[root@node1 bin]# expect beeline.exp
```

可以用 command-line shell 和 thrift／jdbc 两种方式来操作数据：

- **command-line shell**：通过 hive 命令行的的方式来操作数据；

```shell
[root@node1 onekey]# hive
which: no hbase in (:/export/server/sqoop-1.4.6/bin::/export/server/hive-2.1.0/bin::/export/server/hadoop-2.7.5/bin:/export/server/hadoop-2.7.5/sbin:/export/server/jdk1.8.0_241/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin)
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/export/server/hive-2.1.0/lib/hive-jdbc-2.1.0-standalone.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/export/server/hive-2.1.0/lib/log4j-slf4j-impl-2.4.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/export/server/hadoop-2.7.5/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]

Logging initialized using configuration in jar:file:/export/server/hive-2.1.0/lib/hive-common-2.1.0.jar!/hive-log4j2.properties Async: true
Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
hive> 
```

- **thrift／jdbc**：通过 thrift 协议按照标准的 JDBC 的方式操作数据。

```shell
[root@node1 bin]# expect beeline.exp
spawn beeline
which: no hbase in (:/export/server/sqoop-1.4.6/bin::/export/server/hive-2.1.0/bin::/export/server/hadoop-2.7.5/bin:/export/server/hadoop-2.7.5/sbin:/export/server/jdk1.8.0_241/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin)
Beeline version 2.1.0 by Apache Hive
beeline> !connect jdbc:hive2://node1:10000
Connecting to jdbc:hive2://node1:10000
Enter username for jdbc:hive2://node1:10000: root
Enter password for jdbc:hive2://node1:10000: ******
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/export/server/hive-2.1.0/lib/hive-jdbc-2.1.0-standalone.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/export/server/hive-2.1.0/lib/log4j-slf4j-impl-2.4.1.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/export/server/hadoop-2.7.5/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Connected to: Apache Hive (version 2.1.0)
Driver: Hive JDBC (version 2.1.0)
23/05/01 16:09:23 [main]: WARN jdbc.HiveConnection: Request to set autoCommit to false; Hive does not support autoCommit=false.
Transaction isolation: TRANSACTION_REPEATABLE_READ
0: jdbc:hive2://node1:10000> 
0: jdbc:hive2://node1:10000> 
0: jdbc:hive2://node1:10000> 
0: jdbc:hive2://node1:10000> 
0: jdbc:hive2://node1:10000> 
0: jdbc:hive2://node1:10000> 
0: jdbc:hive2://node1:10000> 
```





### 2.3Metastore 元数据

在Hive中，表名称、表结构、字段名、字段类型、表的分隔符统一称为元数据。所有的元数据（mateStore）





### 2.4HSQL的执行过程







## 三、Hive的数据类型

### 3.1基本数据类型

| 数据类型   大类                         | 类型                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| Integers（**整型**）                    | TINYINT（tinyint）1字节的**有符号整数**<br />SMALLINT（smallint）2字节的有符号整数<br />INT（int） 4个字节的有符号整数 |
| Boolean（**布尔类型**）                 | BOOLEAN——TRUE、FALSE**真值**                                 |
| Floating Point Numbers（**浮点型**）    | FLOAT单精度浮点数，DOUBLE双精度浮点数                        |
| Fixed point numbers（**定点数**）       | DECIMAL（decimal）用户自己定义的精度定点数，如：     DECIMAL(7,2) |
| String types（**字符串**）              | STRING指定字符集的字符序列<br />VARCHAR具有最大**长度**的字符序列<br />CHAR**固定长度**的字符序列 |
| Data abd time types（日期**时间**类型） | TIMESTAMP**时间戳**<br />TIMESTAMP WITH LOCAL TIME ZONE时间戳，纳秒精度<br />DATE日期类型 |
| Binary types（**二进制**类型）          | BINARY字节序列                                               |

### 3.2复杂类型（类似对象数据类型）

 

| 类型             | 描述                                                         | 示例                                                         |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| STRUCT结构体     | 字段的集合，类似与对象[小明(12岁，男，小学生)]，<br />字段的类型可以不同，可以使用`名称.字段名`方式进行访问 | `STRUCT('李明',12,'小学生','2023-4-8')`                      |
| MAP              | 键值对的集合，使用`名称[key]`的方式访问对应的VALUE           | `map('a',1,'b',2)`<br />name=a的值为1，<br />name=b的值为2   |
| ARRAY(array)数组 | 数组是一组具有相同数据类型和名称的变量组，<br />用名称加索引访问对应的VALUE：`名称[index]` | `ARRAY(11290,1243124,345,242,3432)`<br />`ARRAY('jone','live','valueFda','data')` |

### 3.3隐式转换

Hive 中基本数据类型遵循以下的层次结构，按照这个层次结构，子类型到祖先类型允许隐式转换。例如 INT 类型的数据允许隐式转换为 BIGINT 类型。额外注意的是：按照类型层次结构允许将 STRING 类型隐式转换为 DOUBLE 类型。

![img](https://camo.githubusercontent.com/2b9ac2eccdbb05b9ff8635126e77cc5c9cdf683658ef7bc340f22be75a69f57c/68747470733a2f2f67697465652e636f6d2f68656962616979696e672f426967446174612d4e6f7465732f7261772f6d61737465722f70696374757265732f686976652d646174612d747970652e706e67)

### 3.4示例

```sql
CREATE TABLE students(
  name      STRING,   -- 姓名
  age       INT,      -- 年龄
  subject   ARRAY<STRING>,   --学科
  score     MAP<STRING,FLOAT>,  --各个学科考试成绩
  address   STRUCT<houseNumber:int, street:STRING, city:STRING, province：STRING>  --家庭居住地址
) ROW FORMAT DELIMITED FIELDS TERMINATED BY "\t";

-- 其中文件分隔符是： "\t"制表符

```

## 四、文件分割符的设置





































###  





 