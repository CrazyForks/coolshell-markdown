# 6个有用的MySQL语句
>date: 2010-12-31T08:29:35+08:00


以前本站给大家介绍过《[MySQL性能优化的最佳20+条经验](/2009/MySQL%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%E7%9A%84%E6%9C%80%E4%BD%B320%2B%E6%9D%A1%E7%BB%8F%E9%AA%8C.md)》，今天给大家介绍六条比较有用的MySQL的SQL语句，可能很多人都通过PHP来实现这些功能。




目录



* [1. 计算年数](#1_%E8%AE%A1%E7%AE%97%E5%B9%B4%E6%95%B0 "1. 计算年数")
* [2. 两个时间的差](#2_%E4%B8%A4%E4%B8%AA%E6%97%B6%E9%97%B4%E7%9A%84%E5%B7%AE "2. 两个时间的差")
* [3. 显示某一列出现过N次的值](#3_%E6%98%BE%E7%A4%BA%E6%9F%90%E4%B8%80%E5%88%97%E5%87%BA%E7%8E%B0%E8%BF%87N%E6%AC%A1%E7%9A%84%E5%80%BC "3. 显示某一列出现过N次的值")
* [4. 计算两个日子间的工作日](#4_%E8%AE%A1%E7%AE%97%E4%B8%A4%E4%B8%AA%E6%97%A5%E5%AD%90%E9%97%B4%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%97%A5 "4. 计算两个日子间的工作日")
* [5. 查找表中的主键](#5_%E6%9F%A5%E6%89%BE%E8%A1%A8%E4%B8%AD%E7%9A%84%E4%B8%BB%E9%94%AE "5. 查找表中的主键")
* [6. 查看你的数库有多大](#6_%E6%9F%A5%E7%9C%8B%E4%BD%A0%E7%9A%84%E6%95%B0%E5%BA%93%E6%9C%89%E5%A4%9A%E5%A4%A7 "6. 查看你的数库有多大")

#### 1. 计算年数


你想通过生日来计算这个人有几岁了。



```

SELECT DATE_FORMAT(FROM_DAYS(TO_DAYS(now()) - TO_DAYS(@dateofbirth)), '%Y') + 0;

```

#### 2. 两个时间的差


取得两个 datetime 值的差。假设 dt1 和 dt2 是 datetime 类型，其格式为 ‘yyyy-mm-dd hh:mm:ss’，那么它们之间所差的秒数为：



```

UNIX_TIMESTAMP( dt2 ) - UNIX_TIMESTAMP( dt1 )

```

除以60就是所差的分钟数，除以3600就是所差的小时数，再除以24就是所差的天数。


#### 3. 显示某一列出现过N次的值



```

SELECT id
FROM tbl
GROUP BY id
HAVING COUNT(*) = N;

```


#### 4. 计算两个日子间的工作日


所谓工作日就是除出周六周日和节假日。



```

SELECT COUNT(*)
FROM calendar
WHERE d BETWEEN Start AND Stop
  AND DAYOFWEEK(d) NOT IN(1,7)
  AND holiday=0;

```

#### 5. 查找表中的主键



```

SELECT k.column_name
FROM information_schema.table_constraints t
JOIN information_schema.key_column_usage k
USING (constraint_name,table_schema,table_name)
WHERE t.constraint_type='PRIMARY KEY'
  AND t.table_schema='db'
  AND t.table_name=tbl'

```

#### 6. 查看你的数库有多大



```

SELECT
  table_schema AS 'Db Name',
  Round( Sum( data_length + index_length ) / 1024 / 1024, 3 ) AS 'Db Size (MB)',
  Round( Sum( data_free ) / 1024 / 1024, 3 ) AS 'Free Space (MB)'
FROM information_schema.tables
GROUP BY table_schema ;

```

希望对你有帮助。


文章：[来源](http://www.codeforest.net/6-useful-mysql-queries)


