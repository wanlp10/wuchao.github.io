---
layout: post
title: com.microsoft.sqlserver.jdbc.SQLServerException: The "variant" data type is not supported.
category : [学习问题记录]
tagline: "Supporting tagline"
tags : [SQL Server]
---
{% include JB/setup %}
# com.microsoft.sqlserver.jdbc.SQLServerException: The "variant" data type is not supported.
---

<!--break-->

查询 SQL SERVER 中某张表结构，sql 语句如下：
```
SELECT
	tb.name AS tableName,
	col.name AS columnName,
	col.max_length AS length,
	col.is_nullable AS isNullable,
	t.name AS type,
	(
	SELECT
		TOP 1 ind.is_primary_key
	FROM
		sys.index_columns ic
		LEFT JOIN sys.indexes ind ON ic.object_id = ind.object_id AND ic.index_id= ind.index_id AND ind.name LIKE 'PK_%'
	WHERE
		ic.object_id = tb.object_id AND ic.column_id= col.column_id
	) AS isPrimaryKey,
	com.value AS comment
FROM
	sys.TABLES tb
	INNER JOIN sys.columns col ON col.object_id = tb.object_id
	LEFT JOIN sys.types t ON t.user_type_id = col.user_type_id
	LEFT JOIN sys.extended_properties com ON com.major_id = col.object_id
	AND com.minor_id = col.column_id
WHERE
	tb.name = '表名'
```

该 sql 可以正常执行，但是当把 sql 放到 jdbcTemplate 中执行时报一下错误：
```
Caused by: com.microsoft.sqlserver.jdbc.SQLServerException: The "variant" data type is not supported.
```

原因是 sql 语句 select 后面有 `sql_variant` 类型的属性，在 JDBC 中不支持它。使用 `sp_columns` 命令最终查出 `sys.extended_properties` 表的 `value` 属性的 `TYPE_NAME` 是 `sql_variant` 类型的，sql 如下：
```
sp_columns extended_properties
```

解决方法是使用 `CONVERT` 函数将该属性转成 `varchar` 类型。
> CONVERT 函数的用法参考：[SQL Server 中 CONVERT() 函数的使用](https://blog.csdn.net/jacksonary/article/details/78800591)。

修改后的 sql 语句为：
```
SELECT
	tb.name AS tableName,
	col.name AS columnName,
	col.max_length AS length,
	col.is_nullable AS isNullable,
	t.name AS type,
	(
	SELECT
		TOP 1 ind.is_primary_key
	FROM
		sys.index_columns ic
		LEFT JOIN sys.indexes ind ON ic.object_id = ind.object_id AND ic.index_id= ind.index_id AND ind.name LIKE 'PK_%'
	WHERE
		ic.object_id = tb.object_id AND ic.column_id= col.column_id
	) AS isPrimaryKey,
	CONVERT(varchar(200), com.value) AS comment
FROM
	sys.TABLES tb
	INNER JOIN sys.columns col ON col.object_id = tb.object_id
	LEFT JOIN sys.types t ON t.user_type_id = col.user_type_id
	LEFT JOIN sys.extended_properties com ON com.major_id = col.object_id
	AND com.minor_id = col.column_id
WHERE
	tb.name = '表名'
```

> 参考：
>
> [com.microsoft.sqlserver.jdbc.SQLServerException: The "variant" data type is not supported.](https://social.msdn.microsoft.com/Forums/office/en-US/c4868d02-cf39-4cc8-8500-99359ca24632/commicrosoftsqlserverjdbcsqlserverexception-the-quotvariantquot-data-type-is-not-supported?forum=sqldataaccess)
