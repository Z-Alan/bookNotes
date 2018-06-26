# 使用explain命令进行sql语句性能优化（一）

## 前提条件

* 使用的数据库： Mysql 5.0+
* 使用的可视化工具：sqlyog

## 认识Explain

> 在工作中，我们用于捕捉性能问题最常用的就是打开慢查询，定位执行效率差的SQL，那么当我们定位到一个SQL以后还不算完事，我们还需要知道该SQL的执行计划，比如是全表扫描，还是索引扫描，这些都需要通过EXPLAIN去完成。EXPLAIN命令是查看优化器如何决定执行查询的主要方法。可以帮助我们深入了解MySQL的基于开销的优化器，还可以获得很多可能被优化器考虑到的访问策略的细节，以及当运行SQL语句时哪种策略预计会被优化器采用。需要注意的是，生成的QEP并不确定，它可能会根据很多因素发生改变。MySQL不会将一个QEP和某个给定查询绑定，QEP将由SQL语句每次执行时的实际情况确定，即便使用存储过程也是如此。尽管在存储过程中SQL语句都是预先解析过的，但QEP仍然会在每次调用存储过程的时候才被确定

## 如何使用Explain

> 只需要在你的sql语句前加Explain就可以了，然后执行查询就可以看到Explain视角下的sql语句

### 举个栗子

```SQL
EXPLAIN
SELECT * 
FROM db_m_s 
```

![](C:\Users\Administrator\Desktop\文档\images\1525756801132.png)

### 术语表

| **id**            | SELECT识别符。这是SELECT的查询序列号                         |
| ----------------- | ------------------------------------------------------------ |
| **select_type**   | SELECT类型,可以为以下任何一种:                                                                                          **SIMPLE**:简单SELECT(不使用UNION或子查询)                                                                               **PRIMARY**:最外面的SELECT                                                                                       **UNION**:UNION中的第二个或后面的SELECT语句                                                                                              **DEPENDENT UNION**:UNION中的第二个或后面的SELECT语句,取决于外面的查询                         **UNION RESULT**:UNION 的结果                                                                                     **SUBQUERY**:子查询中的第一个SELECT                                                                       **DEPENDENT SUBQUERY**:子查询中的第一个SELECT,取决于外面的查询                             **DERIVED**:导出表的SELECT(FROM子句的子查询) |
| **table**         | 输出的行所引用的表                                           |
| **type**          | 联接类型。下面给出各种联接类型,按照从最佳类型到最坏类型进行排序:                              **system**:表仅有一行(=系统表)。这是const联接类型的一个特例。                                            **const**:表最多有一个匹配行,它将在查询开始时被读取。因为仅有一行,在这行的列值可被优化器剩余部分认为是常数。const表很快,因为它们只读取一次!                                                           **eq_ref**:对于每个来自于前面的表的行组合,从该表中读取一行。这可能是最好的联接类型,除了const类型。                                                                                                                                             **ref**:对于每个来自于前面的表的行组合,所有有匹配索引值的行将从这张表中读取。               **ref_or_null**:该联接类型如同ref,但是添加了MySQL可以专门搜索包含NULL值的行。   **index_merge**:该联接类型表示使用了索引合并优化方法。                                         **unique_subquery**:该类型替换了下面形式的IN子查询的ref: value IN (SELECT primary_key FROM single_table WHERE some_expr) unique_subquery是一个索引查找函数,可以完全替换子查询,效率更高。                                                                                                                **index_subquery**:该联接类型类似于unique_subquery。可以替换IN子查询,但只适合下列形式的子查询中的非唯一索引: value IN (SELECT key_column FROM single_table WHERE some_expr)   **range**:只检索给定范围的行,使用一个索引来选择行。                                                                    **index**:该联接类型与ALL相同,除了只有索引树被扫描。这通常比ALL快,因为索引文件通常比数据文件小。                                                                                                                                           **ALL**:对于每个来自于先前的表的行组合,进行完整的表扫描。 |
| **possible_keys** | 指出MySQL能使用哪个索引在该表中找到行                        |
| **key**           | 显示MySQL实际决定使用的键(索引)。如果没有选择索引,键是NULL。 |
| **key_len**       | 显示MySQL决定使用的键长度。如果键是NULL,则长度为NULL。       |
| **ref**           | 显示使用哪个列或常数与key一起从表中选择行。                  |
| **rows**          | 显示MySQL认为它执行查询时必须检查的行数。多行之间的数据相乘可以估算要处理的行数。 |
| **filtered**      | 显示了通过条件过滤出的行数的百分比估计值。                   |

## 解释

### id

包含一组数字，白哦是查询中执行select字句或操作表的顺序

- id相同，执行顺序由上至下
- 如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
- id 如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行。

### select_type

表示查询中每个select子句的类型（简单or复杂）

- **SIMPLE**：查询中不包含子查询或者UNION
- 查询中若包含任何复杂的子部分，最外层查询则被标记为：**PRIMARY**
- 在SELECT或WHERE列表中包含了子查询，该子查询被标记为：**SUBQUERY**
- 在FROM列表中包含的子查询被标记为：**DERIVED**（衍生）用来表示包含在from子句中的子查询的select，mysql会递归执行并将结果放到一个临时表中。服务器内部称为"派生表"，因为该临时表是从子查询中派生出来的
- 若第二个SELECT出现在UNION之后，则被标记为UNION；若UNION包含在FROM子句的子查询中，外层SELECT将被标记为：**DERIVED**
- 从UNION表获取结果的SELECT被标记为：**UNION RESULT**
- **SUBQUERY**和**UNION**还可以被标记为**DEPENDENT**和**UNCACHEABLE**。
- **DEPENDENT**意味着select依赖于外层查询中发现的数据。
- **UNCACHEABLE**意味着select中的某些 特性阻止结果被缓存于一个item_cache中。

### type

表示MySQL在表中找到所需行的方式，又称“访问类型”，常见类型如下:

 **ALL, index,  range, ref, eq_ref, const, system, NULL**

- **ALL：Full Table Scan， MySQL将遍历全表以找到匹配的行**
-  **index：Full Index Scan，index与ALL区别为index类型只遍历索引树**
-  **range:索引范围扫描，对索引的扫描开始于某一点，返回匹配值域的行。显而易见的索引范围扫描是带有between或者where子句里带有<, >查询。当mysql使用索引去查找一系列值时，例如IN()和OR列表，也会显示range（范围扫描）,当然性能上面是有差异的。**
- **ref：使用非唯一索引扫描或者唯一索引的前缀扫描，返回匹配某个单独值的记录行**
- **eq_ref：类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件**
- **const、system：当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量**,**system是const类型的特例，当查询的表只有一行的情况下，使用system**
- **NULL：MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。**

### possible_keys

指出MySQL能使用哪个索引在表中找到记录，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用

### key

显示MySQL在查询中实际使用的索引，若没有使用索引，显示为NULL

### key_len

表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度（key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的）

### ref

表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值

### rows

表示MySQL根据表统计信息及索引选用情况，估算的找到所需的记录所需要读取的行数

### Extra

包含不适合在其他列中显示但十分重要的额外信息

- **Using index**   该值表示相应的select操作中使用了覆盖索引（Covering Index）

> **覆盖索引（Covering Index）**
> MySQL可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件
> 包含所有满足查询需要的数据的索引称为覆盖索引（Covering Index）
> 注意：如果要使用覆盖索引，一定要注意select列表中只取出需要的列，不可select \*，因为如果将所有字段一起做索引会导致索引文件过大，查询性能下降

- **Using where**

> 表示mysql服务器将在存储引擎检索行后再进行过滤。许多where条件里涉及索引中的列，当（并且如果）它读取索引时，就能被存储引擎检验，因此不是所有带where字句的查询都会显示"Using where"。有时"Using where"的出现就是一个暗示：查询可受益与不同的索引。

-  **Using temporary**

> 表示MySQL需要使用临时表来存储结果集，常见于排序和分组查询这个值表示使用了内部临时(基于内存的)表。一个查询可能用到多个临时表。有很多原因都会导致MySQL在执行查询期间创建临时表。两个常见的原因是在来自不同表的上使用了DISTINCT,或者使用了不同的ORDER BY和GROUP BY列。可以强制指定一个临时表使用基于磁盘的MyISAM存储引擎。这样做的原因主要有两个：
>
> 1)内部临时表占用的空间超过min(tmp_table_size，max_heap_table_size)系统变量的限制
>
> 2)使用了TEXT/BLOB 列

- **Using filesort**

> MySQL中无法利用索引完成的排序操作称为“文件排序”

- **Using join buffer**

> 该值强调了在获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果。如果出现了这个值，那应该注意，根据查询的具体情况可能需要添加索引来改进能。

- **Impossible where**

> 这个值强调了where语句会导致没有符合条件的行。

- **Impossible where**

> 这个值强调了where语句会导致没有符合条件的行。

- **Index merges**

> 当MySQL 决定要在一个给定的表上使用超过一个索引的时候，就会出现以下格式中的一个，详细说明使用的索引以及合并的类型。

## EXPLAIN 能干什么

- EXPLAIN不会告诉你关于触发器、存储过程的信息或用户自定义函数对查询的影响情况
- EXPLAIN不考虑各种Cache
- EXPLAIN不能显示MySQL在执行查询时所作的优化工作
- 部分统计信息是估算的，并非精确值
- EXPALIN只能解释SELECT操作，其他操作要重写为SELECT后查看执行计划。

## 

参考博客：

https://blog.csdn.net/u010061060/article/details/52473244

https://www.cnblogs.com/gomysql/p/3720123.html

https://blog.csdn.net/longwoniu/article/details/52815979

