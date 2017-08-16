---
title: SQL 函数
tags:
  - sql
categories:
  - 数据库
date: 2017-08-16 14:03:36
---



## NULL 函数

> NULL 值不能用等于 `=` 或 `!=` 来判断

```mysql
IFNULL(col, val) # 如果列的值为 NULL 则取值 val

COALESCE(col, val)
```

## DATETIME 类型与 TIMESTAMP 类型的区别

- 两种格式都会返回相同的格式 **YYYY-MM-DD HH:MM:SS**

> Timestamps in MySQL generally used to track changes to records, and are often updated every time the record is changed. If you want to store a specific value you should use a datetime field.
>
> If you meant that you want to decide between using a UNIX timestamp or a native MySQL datetime field, go with the **native format**.
>
> You can do calculations within MySQL that way
>
> `("SELECT DATE_ADD(my_datetime, INTERVAL 1 DAY)")`
>
> and it is simple to change the format of the value to a UNIX timestamp
>
> `("SELECT UNIX_TIMESTAMP(my_datetime)")`
>
> when you query the record if you want to operate on it with PHP.
>
> [参考](https://stackoverflow.com/a/409305)

## Aggregate functions

操作一系列的值，返回单一值，通常结合 `GROUP BY` 语句。

可以对一个以上的列应用 `GROUP BY` 语句

### AVG

### COUNT

返回指定列的值的数目，** NULL 不计入**

`COUNT(DISTINCT col)` 函数返回指定列的**不同值**的数目

```mysql
SELECT COUNT(DISTINCT col) FROM table_name;
```

### FIRST

```mysql
SELECT FIRST(col) FROM table_name;
```

### LAST

```mysql
SELECT LAST(col) FROM table_name;
```

### MAX

MAX 函数返回一列中的最大值，** NULL 值不包括在计算中**

> MIN MAX 用于文本列中，以获得按字母顺序排列的最低或最高值

### MIN

### SUM

### HAVING 子句

在 SQL 中增加 HAVING 子句的原因是：WHERE 关键字无法与 Aggregate function 一起使用。

```mysql
SELECT col1, SUM(col2) FROM table_name
GROUP BY col1
HAVING SUM(col2) > 1000;
```

## Scalar functions

操作单一的值，返回单一的值。

### UCASE

UCASE 函数把字段值转为大写

### LCASE

LCASE 函数把字段值转为小写

### MID

用于从文本字段中提取字符

```mysql
SELECT MID(col, start[, length]) FROM table_name;
```

- col 必需，要提取字符的字段
- start 必需，开始的位置，从1开始
- length 可选，返回字符数

### LENGTH

返回文本字段值的长度

### ROUND

把数值字段舍入指定的小数位数

```mysql
SELECT ROUND(col, decimals) FROM table_name;
```

- col 必需
- decimals 可选，保留的小数位数

## 日期计算

### TIMESTAMPDIFF

```mysql
SELECT TIMESTAMPDIFF(DAY, updatedAt, CURDATE()) as days from admins;
```

- unit 比较的单位 SECOND / MINUTE / HOUR / DAY / MONTH / YEAR
- from_time 比较的开始时间
- to_time 比较的结束时间

### DATEDIFF

DATEDIFF(expr1, expr2) returns `expr1 - expr2` expressed as a value in days.

### YEAR / MONTH / DAY / DATE / TIME

exrtract specific part of the column.

**注意 DATE 函数是提取日期部分**

```mysql
# 找出下个月生日的宠物
SELECT name, birth FROM pet
WHERE MONTH(birth) = MONTH(DATE_ADD(CURDATE(), INTERVAL 1 MONTH));

# 另一方法，使用 MOD 函数
SELECT name, birth FROM pet
WHERE MONTH(birth) = MOD(MONTH(CURDATE()), 12) + 1;
```

### CONVERT_TZ

时区转换

```mysql
SELECT CONVERT_TZ('2017-08-15 01:00:00', '+00:00', '+08:00');
```

### ADDDATE / SUBDATE

语法：

- `ADDDATE(date, INTERVAL n unit)`
- `ADDDATE(expr, days)`

```mysql
SELECT DATE_ADD('2008-01-02', INTERVAL 31 DAY);
 -> '2008-02-02'
SELECT ADDDATE('2008-01-02', INTERVAL 31 DAY);
 -> '2008-02-02'
SELECT ADDDATE('2008-01-02', 31);
 -> '2008-02-02'
```

同义词：

- `ADDDATE` / `DATE_ADD`
- `SUBDATE` / `DATE_SUB`

### ADDTIME

```mysql
SELECT ADDTIME('2007-12-31 23:59:59.999999', '1 1:1:1.000002');
 -> '2008-01-02 01:01:01.000001'
SELECT ADDTIME('01:00:00.999999', '02:00:00.999998');
 -> '03:00:01.999997'
```

### NOW / CURDATE / CURTIME /

同义词：

- `CURRENT_DATE` / `CURDATE`
- `CURRENT_TIME` / `CURTIME`
- `CURRENT_TIMESTAMP` / `NOW`

```mysql
SELECT NOW();
SELECT CURDATE();
SELECT CURTIME();
```

### DATE_FORMAT

格式化函数

```mysql
SELECT DATE_FORMAT(NOW(), '%Y-%m-%d %H:%i:%S');
```
