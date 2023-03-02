# Task06 详读第7章Mapreduce内容

## 1 数据仓库
数据仓库是一个面向主题的、集成的、非易失的、随时间变化的，用来支持管理人员决策的数据集合，数据仓库中包含了粒度化的企业数据。

数据仓库的主要特征是：**主题性、集成性、非易失性、时变性**。

## 2 Hive
![Alt text](https://datawhalechina.github.io/juicy-bigdata/images/ch06/ch6.1.png)
Hive是建立在Hadoop之上的一种数仓工具。该工具的功能是将结构化、半结构化的数据文件映射为一张数据库表，基于数据库表，提供了一种类似SQL的查询模型（HQL），用于访问和分析存储在Hadoop文件中的大型数据集。

Hive本身并不具备存储功能，其核心是将HQL转换为MapReduce程序，然后将程序提交到Hadoop集群中执行。
![Alt text](https://datawhalechina.github.io/juicy-bigdata/images/ch06/ch6.1.1.png)

特点：

1. 简单、容易上手（提供了类似SQL的查询语言HiveQL），使得精通SQL却不了解Java 编程的人也能很好地进行大数据分析；
1. 灵活性高，可以自定义用户函数（UDF）和存储格式；
1. 为超大的数据集设计的计算和存储能力，集群扩展容易;
1. 统一的元数据管理，可与presto/impala/sparksql等共享数据；
1. 执行延迟高，不适合做数据的实时处理，但适合做海量数据的离线处理。

### 2.1 Hive与HBase的区别
HBase是一个面向列式存储、分布式、可伸缩的数据库，它可以提供数据的实时访问功能，而Hive只能处理静态数据，主要是BI报表数据。就设计初衷而言，在Hadoop上设计Hive，是为了减少复杂MapReduce应用程序的编写工作，在Hadoop上设计HBase是为了实现对数据的实时访问。所以，HBase与Hive的功能是互补的，它实现了Hive不能提供的功能。

### 2.2 Hive与传统数据库的对比

`Hive`在很多方面和传统数据库类似，但是，它的底层依赖的是`HDFS`和`MapReduce`（或`Tez`、`Spark`）。以下将从各个方面，对`Hive`和传统数据库进行对比分析。

| 对比内容 | Hive | 传统数据库 |
| :---: | :---: | :---: |
| 数据存储 | HDFS | 本地文件系统 |
| 索引 | 支持有限索引 | 支持复杂索引 |
| 分区 | 支持 | 支持 |
| 执行引擎 | MapReduce、Tez、Spark | 自身的执行引擎 |
| 执行延迟 | 高 | 低 |
| 扩展性 | 好 | 有限 |
| 数据规模 | 大 | 小 |


### 2.3 Hive数据类型

- **基本数据类型**

&emsp;&emsp;`Hive`表中的列支持以下基本数据类型：

| 大类 | 类型 |
| :--- | :--- |
| Integers（整型） | TINYINT：1字节的有符号整数；<br>SMALLINT：2字节的有符号整数；<br>INT：4字节的有符号整数；<br>BIGINT：8字节的有符号整数 |
| Boolean（布尔型）| BOOLEAN：TRUE/FALSE |
| Floating point numbers（浮点型）| FLOAT：单精度浮点型；<br>DOUBLE：双精度浮点型 |
| Fixed point numbers（定点数）| DECIMAL：用户自定义精度定点数，比如 DECIMAL(7,2) |
| String types（字符串）| STRING：指定字符集的字符序列；<br>VARCHAR：具有最大长度限制的字符序列；<br>CHAR：固定长度的字符序列 |
| Date and time types（日期时间类型） | TIMESTAMP：时间戳；<br>TIMESTAMP WITH LOCAL TIME ZONE：时间戳，纳秒精度；<br>DATE：日期类型 |
| Binary types（二进制类型）| BINARY：字节序列 |

> 注：`TIMESTAMP`和`TIMESTAMP WITH LOCAL TIME ZONE`的区别如下：  
> - **TIMESTAMP WITH LOCAL TIME ZONE**：用户提交`TIMESTAMP`给数据库时，会被转换成数据库所在的时区来保存。查询时，则按照查询客户端的不同，转换为查询客户端所在时区的时间。  
> - **TIMESTAMP** ：提交的时间按照原始时间保存，查询时，也不做任何转换。

- **隐式转换**

&emsp;&emsp;`Hive`中基本数据类型遵循以下的层次结构，按照这个层次结构，子类型到祖先类型允许隐式转换。例如`INT`类型的数据允许隐式转换为`BIGINT`类型。额外注意的是：按照类型层次结构，允许将`STRING`类型隐式转换为`DOUBLE`类型。

<center><img src="images/ch06/ch6.2.1.png" style="zoom:33%;" /></center>

- **复杂类型**

| 类型 | 描述 | 示例 |
| --- | --- | --- |
| STRUCT | 类似于对象，是字段的集合，字段的类型可以不同，可以使用`名称.字段名`方式进行访问 | STRUCT('xiaoming', 12 , '2018-12-12') |
| MAP | 键值对的集合，可以使用`名称[key]`的方式访问对应的值 | map('a', 1, 'b', 2) |
| ARRAY | 数组是一组具有相同类型和名称的变量的集合，可以使用`名称[index]`访问对应的值 | ARRAY('a', 'b', 'c', 'd') |

- **示例**

&emsp;&emsp;下面是一个基本数据类型和复杂数据类型的使用示例：

```sql
CREATE TABLE students(
  name      STRING,   -- 姓名
  age       INT,      -- 年龄
  subject   ARRAY<STRING>,   -- 学科
  score     MAP<STRING,FLOAT>,  -- 各个学科考试成绩
  address   STRUCT<houseNumber:int, street:STRING, city:STRING, province:STRING>  -- 家庭居住地址
) ROW FORMAT DELIMITED FIELDS TERMINATED BY "\t";
```

### 2.4 Hive数据模型

&emsp;&emsp;`Hive`的数据都是存储在`HDFS`上的，默认有一个根目录，在`hive-site.xml`中可以进行配置数据的存储路径。`Hive`数据模型的含义是，描述`Hive`组织、管理和操作数据的方式。`Hive`包含如下4种数据模型：

1. **库**  
&emsp;&emsp;`MySQL`中默认数据库是`default`，用户可以创建不同的`database`，在`database`下也可以创建不同的表。`Hive`也可以分为不同的数据（仓）库，和传统数据库保持一致。在传统数仓中创建`database`。默认的数据库也是`default`。`Hive`中的库相当于关系数据库中的命名空间，它的作用是将用户和数据库的表进行隔离。  

2. **表**  
&emsp;&emsp;`Hive`中的表所对应的数据是存储在`HDFS`中，而表相关的元数据是存储在关系数据库中。Hive中的表分为内部表和外部表两种类型，两者的区别在于数据的访问和删除：  
- 内部表的加载数据和创建表的过程是分开的，在加载数据时，实际数据会被移动到数仓目录中，之后对数据的访问是在数仓目录实现。而外部表加载数据和创建表是同一个过程，对数据的访问是读取`HDFS`中的数据；
- 内部表删除时，因为数据移动到了数仓目录中，因此删除表时，表中数据和元数据会被同时删除。外部表因为数据还在`HDFS`中，删除表时并不影响数据。
- 创建表时不做任何指定，默认创建的就是内部表。想要创建外部表，则需要使用`External`进行修饰

| 对比内容 | 内部表 | 外部表 |
| :--- | :--- | :--- |
| 数据存储位置 | 内部表数据存储的位置由`hive.Metastore.warehouse.dir`参数指定，<br>默认情况下，表的数据存储在`HDFS`的`/user/hive/warehouse/数据库名.db/表名/`目录下 | 外部表数据的存储位置创建表时由`Location`参数指定 |
| 导入数据 | 在导入数据到内部表，内部表将数据移动到自己的数据仓库目录下，<br>数据的生命周期由`Hive`来进行管理 | 外部表不会将数据移动到自己的数据仓库目录下，<br>只是在元数据中存储了数据的位置 |
| 删除表 | 删除元数据（metadata）和文件 | 只删除元数据（metadata） |

3. **分区**  
&emsp;&emsp;分区是一个优化的手段，目的是**减少全表扫描**，提高查询效率。在`Hive`中存储的方式就是表的主目录文件夹下的子文件夹，子文件夹的名字表示所定义的分区列名字。
4. **分桶**  
&emsp;&emsp;分桶和分区的区别在于：分桶是针对数据文件本身进行拆分，根据表中字段（例如，编号ID）的值，经过`hash`计算规则，将数据文件划分成指定的若干个小文件。分桶后，`HDFS`中的数据文件会变为多个小文件。分桶的优点是**优化join查询**和**方便抽样查询**。
