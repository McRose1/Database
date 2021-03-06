# 数据库事务
事务（Transaction）是作为单个逻辑工作单元执行的一系列操作，这些操作作为一个整体一起向系统提交，要么都执行、要么都不执行。

事务是一个不可分割的工作逻辑单元。

## 事务的四大特性
ACID：
- A：Atomicity（原子性）：指事务包含的所有操作要么全部成功，要么全部失败回滚，如果失败则不能对数据库有任何影响
- C：Consistency（一致性）：指事务必须使数据库从一个一致性状态变换到另一个一致性状态，也就是说一个事务执行之前和执行之后都必须处于一致性状态
- I：Isolation（隔离性）：当多个用户并发访问数据库时，比如操作同一张表时，数据库为每一个用户开启的事务，不能被其他事务的操作所干扰，多个并发事务之间要相互隔离
- D：Durabilty（持久性）：指一个事务一旦被提交了，那么对数据库中的数据的改变就是永久性的，即便是在数据库系统遇到故障的情况下也不会丢失提交事务的操作

# MySQL 的四种隔离级别
1. Serializable（串行化）：可避免脏读、不可重复读、幻读的发生
2. Repeatable read（可重复读）：可避免脏读、不可重复读的发生
3. Read commited（读已提交）：可避免脏读的发生
4. Read uncommitted（读未提交）：最低级别，任何情况都无法保证

