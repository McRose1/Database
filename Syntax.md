# 关键语法

## GROUP BY
- 满足“SELECT 子句中的列名必须为分组列或列函数”（只针对同一张表成立）：如果用 group by，那么你的 select 语句中选出的列要么是你 group by 里用到的列，要么就是带有如 sum、min等列函数的列
- 列函数对于 group by 子句定义的每个组各返回一个结果

查询所有同学的学号、选课数、总成绩
- group by student_id：按学号对数据进行分组
- select sutdent_id, count(course_id), sum(score)
- from score 
```sql
select sutdent_id, count(course_id), sum(score)
from score 
group by student_id
```
- explain: extra -> Using temporary
列函数对于 group by 子句定义的每个组各返回一个结果 -> 会生成一个临时表存储

查询所有同学的学号、姓名、选课数、总成绩
```sql
select s.student_id, stu.name, count(s.course_id), sum(score)
from 
  score s,
  student stu
where
s.student_id = stu.student_id
group by s.student_id
```

## HAVING
- 通常与 GROUP BY 子句一起使用（没有 GROUP BY，作用和 WHERE 一样）
- WHERE 过滤行，HAVING 过滤组
- 出现在同一 SQL 的顺序：WHERE > GRUOP BY > HAVING（不可调换）

查询平均成绩大于 60 分的同学的学号和平均成绩
```sql
select student_id, avg(score)
from score
group by student_id
having avg(score)>60
```

查询没有学全所有课的同学的学号、姓名
```sql
select stu.student_id, stu.name 
from 
  student stu,
  socre s
where stu.student_id = s.student_id
group by s.student_id
having count(*) < 
(
select count(*) from course     # 课程总数
)
```

## 统计相关：COUNT，SUM，MAX，MIN，AVG

## LIKE

SQL 提供了 4 种匹配模式：
1. % 表示任意 0 个或多个字符
  - %三：左匹配
  - 三%：右匹配
  - %三%：模糊查询
2. - 表示任意单个字符
3. [ ] 表示括号内所列字符中的一个：[0-9], [a-z]
4. [^ ] 表示不在括号所列之内的单个字符：[^0-9]















