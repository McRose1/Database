# Index（索引）
索引是一种单独的、物理的对数据库表中一列或多列的值进行排序的一种存储结构，它是某个表中一列或若干列值的集合和相应的指向表中物理标识这些值的数据也的逻辑指针清单。

索引的作用相当于图书的目录，可以根据目录中的页码快速找到所需的内容。

索引提供指向存储在表的指定列中的数据值的指针，然后根据我们指定的排序顺序对这些指针排序。

数据库使用索引以找到特定值，然后顺指针找到包含该值的行。

这样可以使对应于表的 SQL 语句执行得更快，可快速访问数据库表中的特定信息。

索引大大减少了服务器需要扫描的数据量、可以帮助服务器避免排序和临时表、可以将随机 IO 变成顺序 IO。

但索引并不总是最好的工具，**对于非常小的表，大部分情况下会采用全表扫描**。

对于中到大型的表，索引就非常有效。

但对于特大型的表，**建立和使用索引的代价也随之增长**，这种情况下应该使用**分区技术**

## 常见问题

### 为什么要使用索引
全表扫描非常慢，除了只有少量数据的情况比索引快

灵感来源于字典，能快速查询数据


### 什么样的信息能成为索引
主键、唯一键以及普通键等


### 索引的数据结构
#### 二叉查找树
优点：查询时间复杂度从 O(n) -> O(logn)
存在问题：
- 增加删除可能会让平衡二叉树变成线性树
- 假设索引都在磁盘，需要 IO，数据很多树就会很深，IO 次数很多

如何优化？既保证查询时间少，又减少 IO 次数 -> 让每个节点存的信息增加（B Tree）

#### B-Tree
![B-Tree](/images/B-Tree.png)
定义（约束条件）：
1. 根节点至少包括 2 个孩子
2. 树中每个节点最多包含 m 个孩子（m >= 2）-> m 阶树
3. 除根节点和叶节点外，其他每个节点至少有 ceil(m/2) 个孩子
4. 所有叶子节点都处于同一层

前 4 条主要是限定每个节点的孩子数以及 B 树的深度，目的是为了让每个节点尽可能存储更多的信息，让树的高度尽可能减小以减少 IO 次数

5. 假设每个非终端节点中包含有 n 个关键字信息，其中：
  - Ki(i=1...n)为关键字，且关键字按顺序升序排序 K(i-1)<Ki
  - 关键字的个数 n 必须满足：[ceil(m/2)-1]<= n <= m-1
  - 非叶子结点的指针：P[1], P[2],..., P[m]；其中 P[1] 指向关键字小于 K[1] 的子树，P[M] 指向关键字大于 K[M-1] 的子树，其他 P[i] 指向关键字属于(K[i-1], K[i]) 的子树

某节点最左边的孩子节点的关键字的值均小于该节点最左边的关键字的值

某节点最右边的孩子节点的关键字的值均大于该节点最右边的关键字的值

其他节点的孩子节点的关键字的值均位于离该节点最近的两个关键字的值的中间

第 5 条限定了 B 树节点关键字数量以及大小

B-Tree 索引的限制：
- 如果不是按照索引的最左列开始查找，则无法使用索引
- 不能跳过索引中的列，例如索引为（id,name,sex），不能只使用 id 和 sex 而跳过 name
- 如果查询中有某个列的范围查询，则其右边的所有列都无法使用索引

#### B+ Tree（InnoDB）
![B+Tree](/images/B+Tree.png)
B+树是 B 树的变体，其定义基本与 B 树相同，除了：
- 非叶子节点的子树指针与关键字个数相同（B+树能存储更多的关键字）
- 非叶子节点的子树指针 P[i]，指向关键字值 [K[i], K[i+1]) 的子树
- 非叶子节点仅用来索引，数据都保存在叶子节点中（非叶子结点不存储数据代表其可以存储更多的关键字，使得 B+树的深度会比 B 树来的更低）
- 所有叶子节点均有一个链指针指向下一个叶子节点（方便我们直接在叶子节点做范围统计：需要查找>=10的数据，不用回到上一层节点）

**在 B+ Tree 中，所有数据记录节点都是按照键值大小顺序存放在同一层的叶子节点上，而非叶子结点上只存储 key 值信息，这样可以大大加大每个节点存储的 key 值数量，降低 B+ Tree 的高度。**

**B+树更适合用来做存储索引**：
- B+树的磁盘读写代价更低：内部结构并没有指向关键字具体信息的指针（不存放数据，只存放索引信息），内部节点相比于 B 树更小，可以容纳更多的关键字数量，一次性读入内存中需要查找的关键字也就越多，IO 读写次数也就相应降低了
- B+树的查询效率更加稳定：任何关键字查找必须从根节点到叶节点，所有关键字查找的长度是相同的，稳定的 O(logn)
- B+树更有利于对数据库的扫描：只需要遍历叶子节点就能完成对全部关键信息的扫描（数据库中频繁使用的范围查询）

#### Hash
缺点：
- 仅仅能满足“=”，“IN”，不能使用范围查询
- 无法被用来避免数据的排序操作
- 不能利用部分索引键查询（组合索引只能哈希整体，而 B+树是支持利用组合索引的部分索引的）
- 不能避免表扫描：不同数据可能有相同的哈希值，还是需要访问 bucket 里的所有数据进行比较查找
- 遇到大量 Hash 值相等的情况后性能并不一定就会比B树索引高（存在同一个 bucket，变成线性存储结构，最极端例子：都放在同一个 bucket）：不稳定

BitMap 位图索引（Oracle 支持，很少支持，不是主流索引）

缺点是锁的力度非常大，不适合高并发

### 密集索引和稀疏索引的区别
- 密集索引文件中的每个搜索码值都对应一个索引值
- 稀疏索引文件只为索引码的某些值建立索引项

![](images/密集索引%26稀疏索引.png)

密集索引：叶子节点保存的不只是键值，还保存了位于同一行记录里的其他列的消息，由于密集索引决定了表的物理排列顺序，一个表只有一个物理排列顺序，所以一个表只能创建一个密集索引

稀疏索引：叶子节点仅保存了键位信息以及该行数据的地址，有的稀疏索引只保存了键位信息机器主键

MyISAM：

不管是主键索引，唯一键值索引或者普通索引，其索引均属于稀疏索引

InnoDB:

有且仅有一个密集索引

密集索引选取规则：
- 若一个主键被定义，则该主键作为密集索引
- 若没有主键被定义，该表的第一个唯一非空索引则作为密集索引
- 若不满足以上条件，innodb 内部会生成一个隐藏主键（密集索引）
- 非主键索引存储相关键位和其对应的主键值，包含两次查找

## 常见的索引原则
1. 选择唯一性索引
2. 为经常需要排序、分组和联合操作的字段建立索引
3. 为常作为查询条件的字段建立索引
4. 限制索引的数目（越多的索引，会使更新表变得很浪费时间）
5. 尽量使用数据量少的索引
6. 如果索引字段的值很长，最好使用值的前缀来索引
7. 删除不再使用或者很少使用的索引
8. 最左前缀匹配原则
9. 尽量选择分区度高的列作为索引
10. 索引列不能参与计算，保持列“干净”：带函数的查询不参与索引
11. 尽量的扩展索引，不要新建索引

## 衍生出来的问题，以 MySQL 为例

### 如何定位并优化慢查询 SQL

#### 根据慢日志定位慢查询 SQL
show variables like '%quer%'; ->
- slow_query_log：on/off
- slow_query_log_file：日志存储的位置
- long_query_time：设置超过多少时间算作慢日志

```sql
show status like '%slow_queries%';

set global slow_query_log = on;（打开慢查询日志）
set global long_query_tie = 1;（设置慢查询基准时间）需要重启数据库

select count(id) from person_info_large;
select name from person_info_large order by name desc;
```
#### 使用 explain 等工具分析 SQL
```sql
explain select name from person_info_large order by name desc;
```
- type（MySQL 找到需要数据行的方式）；最优->最差：system>const>eq_ref>ref>fulltext>ref_or_null>index_merge>unique_subquery>index_subquery>range>**index>all（全表扫描）**
- extra：extra 中出现以下 2 项意味着 MySQL 根本不能使用索引，效率会收到重大影响。应尽可能对此进行优化。
  - Using filesort：表示 MySQL 会对结果使用一个外部索引排序，而不是从表里按索引次序读到相关内容。可能在内存或者磁盘上进行排序。MySQL 中无法利用索引完成的排序操作称为“文件排序”。
  - Using temporary：表示 MySQL 在对查询结果排序时使用临时表。常见于排序 order by 和分组查询 group by。

#### 修改 SQL 或者尽量让 SQL 走索引
- 修改 SQL
```sql
explain select account from person_info_large order by account desc;    // DML
```
type: all -> index

extra: Using filesort -> Using index

如果业务要求就是要按 name 排序怎么办？
- 让 SQL 走索引
```sql
alter table person_info_large add index idx_name(name);     // DDL
```

```sql
explain select count(id) from person_info_large;
```
- type: index
- key: account 
- extra: Using index
我们可以发现这里虽然主键是 id，但是 key 用的却是唯一键；

因为 MySQL 的查询优化器主要目标是尽可能的使用索引，并且使用最严格的索引来消除尽可能多的数据行，它会根据分析和判断的标准决定走哪一个索引

不走主键索引主要可能是考虑到了密集索引的叶子节点把其他列的数据也存放进去了，效率比稀疏索引低，因为稀疏索引只存储了关键字和主键的值

```sql
explain select count(id) from person_info_large force index(primary);
```
会发现这个 SQL 语句执行的时间要比上面那个多

MySQL 优化器并不是永远最优的，需要自己具体事情具体分析

### 联合索引的最左匹配原则的成因
KEY 'index_area_title'('area', 'title') -> area 位于最左
```sql
explain select * from person_info_large where area = '' and title = '';
```
走联合索引：
- possible_keys: index_area_title 
- key: index_area_title 
```sql
explain select * from person_info_large where area = '';
```  
依然走的是联合索引：index_area_title
```sql
explain select * from person_info_large where title = '';
```
不走联合索引而走全表扫描 -> 如果不走最左索引，就无法通过联合索引查找（无法进入 B+树）

#### 最左前缀匹配原则
1. MySQL 会一直向右匹配直到遇到范围查询（&gt;, &lt;, between, like）就停止匹配，比如 a = 3 and b = 4 and c &gt; 5 and d = 6 如果建立（a,b,c,d）顺序的索引，d 是用不到索引的，如果建立（a,b,d,c）的索引则都可以用到，a,b,d 的顺序可以任意调整。
2. = 和 in 可以乱序，比如 a = 1 and b = 2 and c = 3 建立（a,b,c）索引可以任意顺序，MySQL 的查询优化器会帮我们优化成索引可以识别的形式。

### 索引是建立得越多越好吗？
- 数据量小的表不需要建立索引，建立会增加额外的索引开销
- 数据变更需要维护索引，因此更多的索引意味着更多的维护成本
- 更多的索引意味着也需要更多的空间















