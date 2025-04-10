`COALESCE` 是一个在许多关系型数据库中常见的函数，用于选择一组值中的第一个非空值。

`COALESCE` 函数接受多个参数，并按顺序返回第一个非空参数的值。如果所有参数都为 NULL，则 `COALESCE` 函数返回 NULL。

语法如下：

```sql
COALESCE(value1, value2, ..., valueN)
```

- `value1, value2, ..., valueN`：要评估的参数列表。

使用 `COALESCE` 函数的主要目的是处理可能包含 NULL 值的列或表达式。它可以在查询中提供默认值或选择一个可用的非空值。

下面是一些示例：

```sql
SELECT COALESCE(null, 'default'); -- 返回 'default'

SELECT COALESCE(null, null, 'value', null); -- 返回 'value'

SELECT COALESCE(column_name, 'N/A') FROM table_name; -- 如果 column_name 为 NULL，则返回 'N/A'
```

在这些示例中，`COALESCE` 函数用于选择非空值作为结果。如果所有参数都为 NULL，则返回指定的默认值或 NULL。

在上述代码示例中，`COALESCE` 函数用于处理更新语句中的新值。如果新值不为空，则使用新值，否则保持原来的值。这样可以避免在更新操作中意外地将列的值设置为 NULL。