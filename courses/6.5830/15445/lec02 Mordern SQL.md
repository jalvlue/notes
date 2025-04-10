- DML: data management language
- DDL: data definition language
- DCL: data control language


# cross joins: 笛卡尔积
将多个表连接成一个, 用于需要查询数据分布在多个表的记录
```SQL
SELECT s.name
FROM student AS s, enrolled AS e
WHERE e.grade = ... AND ...
```
除了使用 FROM student, enrolled 这种简单方式之外
还可以使用 CROSS JOIN 关键字得到笛卡尔积

# aggregates: 聚合函数
将输入映射为一个标量输出, 常用的聚合函数:
- AVG (COL)
- MIN
- MAX
- SUM
- COUNT

![[Pasted image 20230907151336.png]]
![[Pasted image 20230907151420.png]]

### distinct
DISTINCT 修饰 SELECT 可以删除重复记录, 让查询结果中每个记录唯一
![[Pasted image 20230907151602.png]]

### group by
SELECT 中同时选择了聚合值和非聚合值, 此时想要做的事情应该是在非聚合值内部做聚合, 然后输出对应的非聚合值-内部聚合值
非聚合值要用 GROUP BY 子句包括
GROUP BY 就相当是为该列的每一个值创造一个桶, 然后在桶内做聚合
![[Pasted image 20230907152032.png]]

error: 没有使用 GROUP BY, 因此聚合的是所有查询出来的结果, 此时再选择 `e.cid` 在这个背景下没有任何意义
![[Pasted image 20230907152044.png]]

因此结论就是, 如果一个非聚合列要跟聚合函数一起同时查询的话, 必须使用 GROUP BY 包括起来

如果需要在分桶聚合之后, 加入一些过滤条件:
error: 在聚合计算的过程中, 是无法知道最终的结果的, 因此直接使用 `AND avg_gpa > 3.9` 是错误的
![[Pasted image 20240608095737.png]]

此时可以使用 HAVING 子句
HAVING 子句就像是 GROUP BY 的条件语句, 可以根据设置的条件对经过了 GROUP BY 的输出再次过滤
![[Pasted image 20230907152244.png]]

# string operations
SQL 标准是只能使用单引号 `''` 来创建字符常量, 同时大小写是敏感的(字符串大小写敏感, 关键字不敏感), 实际上不同的 DBMS 有不同的规则, 例如 MYSQL 就是字符串(varchar)大小写不敏感

### 通配符: 在 like 关键字进行字符串匹配的时候

1. % 匹配任何子串, 包括空
2. _ 匹配单个字符

![[Pasted image 20240608101109.png]]

### 字符串函数
字符串也有内置函数进行处理, 当然不同的 DBMS 有不同的函数和处理逻辑

- SUBSTRING
- UPPER
- LOWER
- CONCAT

![[Pasted image 20240608101222.png]]
![[Pasted image 20240608101300.png]]

# output redirection
除了直接 SELECT 将查询结果输出到终端之外, 还可以重定向到某个表(或作为新表存储)

新表:
```SQL
SELECT ... INTO newtable ...
```

插入到某个表
```SQL
INSERT INTO existing_table (SELECT ...)
```

![[Pasted image 20230907153021.png]]


# output control
SQL 的输出是无序的, 可以使用 ORDER BY 子句按照某些属性的值排序

ORDER BY 的默认顺序是升序 `ASC`, 降序是 `DESC` 
![[Pasted image 20230907153405.png]]

多属性会按照层次排序
![[Pasted image 20230907153507.png]]


SQL 的输出可能有很多记录, 可以用 LIMIT 子句做限制
![[Pasted image 20230907153653.png]]

使用 OFFSET 子句可以跳过最开始的几个记录, 返回一个范围内的记录
![[Pasted image 20230907160510.png]]
将返回第 11 到第 31 共 20 个记录


# nested Queries
总结: 嵌套查询实际上是非常慢的, 大部分智能的数据库编译器实际上会在执行计划上选择使用 `JOIN` 进行操作

![[Pasted image 20230907154708.png]]

在子查询获得结果集后, 使用这些关键字跟结果集进行交互
- ALL
- ANY
- IN
- EXIST

![[Pasted image 20230907154927.png]]


# Lateral joins
![[Pasted image 20240608114144.png]]
![[Pasted image 20240608113543.png]]

在原本的嵌套子查询中, 查询都是独立的, 不能引用其他查询的东西, 但是加了 `LATERAL` 关键字之后, 将多个表侧连起来, 可以在子查询内部引用其他子查询的内容


# common table expression: CTE
总结: CTE 实际上就是一个临时表(虚拟表, 临时结果集), 可以在后续使用, 实际上可以看作是嵌套查询的一种形式, 
使用 `WITH...AS...` 语句创建

![[Pasted image 20240608114632.png]]
![[Pasted image 20240608114729.png]]
![[Pasted image 20240608114819.png]]

Common Table Expression（CTE）是一种在 SQL 查询中使用的临时命名结果集的技术。它允许您创建命名的临时表达式，将其视为虚拟表，并在查询中引用它们。CTE 提供了一种清晰、可读性强且可重用的方式来组织和处理复杂的查询逻辑。

以下是 Common Table Expression 的一般语法：

```sql
WITH cte_name (column1, column2, ...)
AS (
  SELECT column1, column2, ...
  FROM table_name
  WHERE condition
)
SELECT *
FROM cte_name
WHERE condition;
```

在上述语法中，CTE 由 `WITH` 关键字引入，后面跟着一个可选的 CTE 名称和列列表。然后，在 `AS` 关键字后面定义 CTE 的查询语句，该查询语句可以引用其他表或 CTE，并且可以包含条件和其他逻辑。最后，查询语句可以使用 CTE 名称作为一个虚拟表来检索结果。

CTE 的主要优点和用途包括：

1. **提高查询可读性和可维护性**：通过将复杂的查询逻辑拆分成命名的临时表达式，CTE 可以提高查询的可读性和可维护性。它使得查询语句更加模块化，易于理解和调试。

2. **避免重复查询逻辑**：使用 CTE，您可以定义一次复杂的查询逻辑，并在后续的查询中多次引用它，而不需要重复编写相同的逻辑。这可以减少代码的冗余并提高效率。

3. **支持递归查询**：CTE 还提供了递归查询的功能。递归查询是一种自引用的查询，可以通过在 CTE 中定义递归部分来构建层次结构和树状结构的查询。

以下是一个示例，演示如何使用 Common Table Expression：

```sql
WITH cte_customers AS (
  SELECT *
  FROM customers
  WHERE country = 'USA'
)
SELECT *
FROM cte_customers
WHERE age > 30;
```

在上述示例中，我们创建了一个名为 `cte_customers` 的 CTE，它选择了 customers 表中国家为 'USA' 的客户。然后，在后续的查询中，我们从该 CTE 中选择了年龄大于 30 的客户。

通过使用 CTE，您可以更清晰地组织查询逻辑，并提高查询的可读性和可维护性。

# window function
![[Pasted image 20240608103407.png]]

![[Pasted image 20240608110652.png]]


总结: window function, `ROW_NUMBER(), RANK()` 提供了一种窗口内追加信息的能力, 它类似聚合函数, 可以基于其他行做计算(例如 `RANK()`), 但是不会像聚合函数一样将许多行聚合成一个标量, 而是在该行的基础上进行追加, 然后还是返回该行, 最终查询结果的行数不变


在 SQL 中，窗口函数（Window Function）是一种用于在查询结果集内执行计算的功能。它与传统的聚合函数（如 SUM、AVG、COUNT）不同，它可以在结果集的指定窗口或分区内进行计算，而不会导致数据的聚合。

窗口函数的特点是它们可以在每一行上计算结果，同时可以访问和处理其他行的数据，而不仅仅是当前行的数据。此外，窗口函数不会改变结果集的行数，而是为每一行返回一个计算结果。

窗口函数通常与 OVER 子句一起使用，该子句用于定义窗口的范围。OVER 子句可以指定窗口的分区方式、排序方式和窗口的边界。以下是一些常用的窗口函数：
1. **ROW_NUMBER ()**：为结果集中的每一行分配一个唯一的行号。
2. **RANK ()**：为结果集中的每一行分配一个排名，相同值的行将获得相同的排名，下一个排名将被跳过。
3. **DENSE_RANK ()**：类似于 RANK ()，但是相同值的行将获得相同的排名，下一个排名将不会被跳过。
4. **SUM ()、AVG ()、MIN ()、MAX ()**：这些聚合函数可以与窗口函数一起使用，用于计算窗口内的汇总、平均、最小值和最大值。
5. **LEAD ()、LAG ()**：这些函数用于获取当前行之前或之后的行的值。LEAD () 返回后续行的值，LAG () 返回前面行的值。

窗口函数在 SQL 中非常有用，可以用于解决许多复杂的查询和分析问题。它们提供了更灵活的方式来计算和处理结果集，并且可以减少对子查询和临时表的需求。但是，窗口函数的支持程度可能因不同的数据库管理系统而有所不同，因此在使用时需要参考特定数据库的文档和语法规范。

当使用 MySQL 数据库时，以下是一些使用窗口函数的示例：
1. **计算行号**：
```sql
SELECT name, age, ROW_NUMBER() OVER (ORDER BY age) AS row_number
FROM students;
```
这个示例中，对于表 `students` 中的每一行，计算出按照年龄排序的行号。

2. **计算排名**：
```sql
SELECT name, score, RANK() OVER (ORDER BY score DESC) AS ranking
FROM exam_results;
```
这个示例中，对于表 `exam_results` 中的每一行，计算按照分数降序排列的排名。

3. **计算每个类别的平均值**：
```sql
SELECT category, amount, AVG(amount) OVER (PARTITION BY category) AS avg_amount
FROM sales;
```
这个示例中，对于表 `sales` 中的每一行，计算出每个类别的销售额平均值。

4. **获取前一行的值**：
```sql
SELECT name, score, LAG(score) OVER (ORDER BY score) AS previous_score
FROM exam_results;
```
这个示例中，对于表 `exam_results` 中的每一行，获取前一行的分数值。

这些示例只是窗口函数在 MySQL 中的一小部分应用场景。MySQL 支持的窗口函数还包括 SUM、MIN、MAX、LEAD、LAG 等，您可以根据具体的需求参考 MySQL 的官方文档以及特定版本的语法规范，以了解更多窗口函数的用法和选项。

在 MySQL 中，`OVER` 关键字用于指定窗口函数的窗口范围。它出现在窗口函数之后，并用于定义窗口的分区方式、排序方式和边界。

`OVER` 关键字的作用如下：

1. **定义窗口分区（Partitioning）**：使用 `PARTITION BY` 子句可以将结果集划分为多个分区。每个分区都将独立地进行窗口函数的计算。例如，可以按照类别将数据分区，然后在每个类别内计算窗口函数。

2. **指定排序方式（Ordering）**：使用 `ORDER BY` 子句可以对每个分区内的行进行排序。窗口函数将按照指定的排序方式对行进行处理。例如，可以按照特定的列对数据进行排序，然后在每个分区内计算窗口函数。

3. **定义窗口边界（Window Frame）**：使用窗口边界可以限定窗口函数的计算范围。可以使用 `ROWS` 或 `RANGE` 关键字定义边界类型，并使用 `BETWEEN` 关键字指定边界的起始和结束位置。窗口边界可以根据当前行的位置或值来确定。例如，可以定义一个窗口，包括当前行及其前面的三行。

以下是一个示例，展示了如何使用 `OVER` 关键字定义窗口函数的窗口范围：

```sql
SELECT category, amount, 
       SUM(amount) OVER (PARTITION BY category ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS cumulative_sum
FROM sales;
```

在上述示例中，`OVER` 关键字后的 `PARTITION BY category ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` 定义了一个窗口，它根据 `category` 列进行分区，按照 `date` 列进行排序，并包括当前行及其之前的所有行。在该窗口范围内，使用 `SUM(amount)` 计算累积销售额。

通过使用 `OVER` 关键字并结合其他子句，可以灵活地定义窗口函数的计算范围和条件，以满足特定的分析需求。

