# Join
FROM table1 INNER|LEFT|RIGHT JOIN table2 ON contiona

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
| 2 | Bob | 4 | Bob | 
| 4 | Dick | 2 | Dick |










