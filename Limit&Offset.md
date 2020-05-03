## limit [起始条目索引], 条目数
起始条目索引从 0 开始，起始条目索引可省略

## limit 条目数 offset 起始条目索引
起始条目索引从 0 开始，起始条目索引可省略

```sql
select * from table limit 3,1;              # 从第 4 条数据开始取数，取 1 条数据，即只取第 4 条
select * from table limit 1 offset 3;       # 从第 4 条数据开始取数，取 1 条数据，即只取第 4 条
select * from table limit 3,2;              # 从第 4 条数据开始取数，取 2 条数据，即取第 4 条、第 5 条
select * from table limit 2 offset 3;       # 从第 4 条数据开始取数，取 2 条数据，即只取第 4 条、第 5 条
```
