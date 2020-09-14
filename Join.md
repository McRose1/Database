# 表连接
FROM table1 INNER|LEFT|RIGHT JOIN table2 ON condition

JOIN 按照功能大致分为如下三类：
- INNER JOIN（内连接，或等值连接）：取得两个表中存在连接匹配关系的记录
- LEFT JOIN（左连接）：取得左边（table1）完全记录，即是右表（table2）并无对应匹配记录
- RIGHT JOIN（右连接）：与 LEFT JOIN 相反，取得右表（table2）完全记录，即是左表（table1）并无匹配对应记录

注意：MySQL 不支持 Full join，不过可以通过 UNION 关键字来合并 LEFT JOIN 与 RIGHT JOIN 来模拟 FULL join

table1:
| id | name |
| ----------- | ----------- |
| 1 | Alice |
| 2 | Bob |
| 3 | Chris |
| 4 | Dick |

table2:
| id | name |
| ----------- | ----------- |
| 1 | Dick |
| 2 | Chris |
| 3 | Brown |
| 4 | Amy |

## Inner join
内连接，也叫等值连接，inner join 产生同时符合 A 和 B 的一组数据
```sql 
select * from A inner join B on A.name = B.name;
```
Result:
| id | name | id | name
| -- | --- | ---- | ----|
|    4 | Dick  |    1 | Dick  |
|    3 | Chris |    2 | Chris |

![inner join](/images/innerJoin.png)

## Left join
左连接从左表产生一套完整的记录，与匹配的记录（右表）；如果没有匹配，右侧将包含 NULL
```sql
select * from A left join B on A.name = B.name;
```
Result:
| id | name | id | name
| -- | --- | ---- | ----|
|    4 | Dick  |    1 | Dick  |
|    3 | Chris |    2 | Chris |
|    1 | Alice | NULL | NULL  |
|    2 | Bob   | NULL | NULL  |

![left join](/images/leftJoin.png)

如果只想从左表中产生一套记录，但不包含右表的记录，可以通过设置 where 语句来执行：
```sql
select * from A left join B on A.name = B.name where A.id is null or B.id is null;
```
Result:
| id | name | id | name
| -- | --- | ---- | ----|
| 1 | Alice | NULL | NULL|
| 2 | Bob | NULL | NULL |

![A 包含但 B 不包含](/images/leftJoin1.png)

还可以用 left join 模拟 inner join：
```sql
select * from A left join B on A.name = B.name where A.id is not null and B.id is not null;
```
Result:
| id | name | id | name
| -- | --- | ---- | ----|
|    4 | Dick  |    1 | Dick  |
|    3 | Chris |    2 | Chris |

也可以让 left join、right join 和 union 一起使用求差集：
```sql
select * from A left join B on A.name = B.name
where B.id is null 
union 
select * from A right join B on A.name = B.name
where A.id is null
```
Result:
| id | name | id | name
| -- | --- | ---- | ----|
| 1 | Alice | NULL | NULL|
| 2 | Bob | NULL | NULL |
| NULL | NULL | 3 | Brown |
| NULL | NULL | 4 | Amy |

![差集](/images/差集.png)

## Right join
同 left join
```sql
select * from A right join B on A.name = B.name;
```
Result:
| id | name | id | name
| -- | --- | ---- | ----|
|    3 | Chris |    2 | Chris |
|    4 | Dick  |    1 | Dick  |
| NULL | NULL  |    3 | Brown |
| NULL | NULL  |    4 | Amy   |

## Cross join
交叉连接，得到的结果是两个表的乘积，即笛卡尔积
```sql
select * from A cross join B
```
Result:
| id | name | id | name
| -- | --- | ---- | ----|
| 1 | Alice | 1 | Dick |
| 2 | Bob | 1 | Dick |
| 3 | Chiris | 1 | Dick |
| 4 | Dick | 1 | Dick |
| 1 | Alice | 2 | Chris |
| 2 | Bob | 2 | Chris |
| 3 | Chiris | 2 | Chris |
| 4 | Dick | 2 | Chris |
| 1 | Alice | 3 | Brown |
| 2 | Bob | 3 | Brown |
| 3 | Chiris | 3 | Brown |
| 4 | Dick | 3 | Brown |
| 1 | Alice | 4 | Amy |
| 2 | Bob | 4 | Amy |
| 3 | Chiris | 4 | Amy |
| 4 | Dick | 4 | Amy |

再执行：
```sql
select * from A inner join B;
```
Result:
| id   | name  | id   | name  |
|------|-------|------|-------|
|    1 | Alice |    1 | Dick  |
|    2 | Bob   |    1 | Dick  |
|    3 | Chris |    1 | Dick  |
|    4 | Dick  |    1 | Dick  |
|    1 | Alice |    2 | Chris |
|    2 | Bob   |    2 | Chris |
|    3 | Chris |    2 | Chris |
|    4 | Dick  |    2 | Chris |
|    1 | Alice |    3 | Brown |
|    2 | Bob   |    3 | Brown |
|    3 | Chris |    3 | Brown |
|    4 | Dick  |    3 | Brown |
|    1 | Alice |    4 | Amy   |
|    2 | Bob   |    4 | Amy   |
|    3 | Chris |    4 | Amy   |
|    4 | Dick  |    4 | Amy   |

顺序互换了

再执行 
```sql
select * from A cross join B on A.name = B.name;
```
Result:
| id | name | id | name
| -- | --- | ---- | ----|
|    4 | Dick  |    1 | Dick  |
|    3 | Chris |    2 | Chris |

和 inner join 的结果相同 

实际上，在 MySQL 中（仅限于 MySQL），cross join 与 inner join 的表现是一样的，在不指定 on 条件得到的结果都是笛卡尔积，反之取得两个表完全匹配的结果；

inner join 与 cross join 可以省略 inner 或 cross 关键字，因此下面的 SQL 效果是一样的：
```sql
... from table1 inner join table2
... from table1 cross join table2
... from table1 join table2
```

## Full join 
全连接产生的所有记录（双方匹配记录）在表 A 和表 B；如果没有匹配，则对面将包含 NULL

注意：MySQL 不支持 Full join，不过可以通过 UNION 关键字来合并 LEFT JOIN 与 RIGHT JOIN 来模拟 FULL join

```sql
select * from A left join B on B.name = A.name
union
select * from A right join B on B.name = A.name;
```
Result:
| id | name | id | name
| -- | --- | ---- | ----|
|    4 | Dick  |    1 | Dick  |
|    3 | Chris |    2 | Chris |
|    1 | Alice | NULL | NULL  |
|    2 | Bob   | NULL | NULL  |
| NULL | NULL  |    3 | Brown |
| NULL | NULL  |    4 | Amy   |

![full join](/images/fullJoin.png)

## 性能优化
未完待续。。。

[csdn](https://blog.csdn.net/w348399060/article/details/70158125)




