---
layout: post
title: "MySQL Transcation"
date: 2025-05-12 17:00:00 +0800
categories: [mysql]
tags: [java]
---


#### 事务并发的三大问题
    1. **脏读：**一个事务读取到了其他事务未提交的数据，造成读不一致
    2. **不可重复度：**一个事务读取到其他事务已经提交的数据，导致前后两次读取的数据不一致的情况
    3. **幻读：**一个事务前后两次读取的数据不一致，是由于其他事务插入数据造成。

#### 不可重复度和幻读的区别：
    4. 修改或者删除造成的读不一致叫做不可重复度，插入造成的读不一致叫幻读

#### 事务四种隔离级别
    1. **Read Uncommitted（未提交读）**
        1. 一个事务读取到其他事务未提交的数据，<font style="color:#DF2A3F;">会出现脏读，没有解决任何问题</font>
    2. **Read Committed（已提交读）**
        1. 一个事务只能读取到其他已经提交的数据，不能读取到其他事务未提交的数据，它解决了脏读。但是会出现不可重复读的问题
    3. **Repeatable Read（可重复读）**
        1. 解决了不可重复读的问题，也就是在一个事务里面多次读取同样的数据结果是一样的。但是没有解决幻读
    4. **Serializable（串行化）**
        1. 这个隔离级别里面，所有事务都是串行执行的，对数据操作都需要排队，不存在事务的并发操作。解决了所有的问题

#### MySQL InnoDB对隔离级别的支持
    1. 略
    2. **<font style="color:#DF2A3F;">InnoDB在RR的级别解决了幻读的问题，InnoDB默认使用RR作为事务隔离级别。</font>**

#### 事务一致性的两大类方案
    3. **LBCC（Lock Based Concurrency Control）**
        1. 基于锁的并发控制。即为了保证前后两次读取数据一致，那么我们读取的时候锁定我们要操作的数据，不允许其他事务修改。意味着不支持并发操作，影响数据的操作效率
    4. **MVCC（多版本的并发控制，Multi Version Concurrency Control）**
        1. 为了保证一个事务前后两次读取的数据保持一致，可以在修改数据之前给他建立一个备份或者叫快照
        2. MVCC的规则：
            1. 一个事务**<font style="color:#DF2A3F;">能够</font>**看到的数据版本
                1. 第一次查询之前已经提交的事务的修改
                2. 本事务的修改
            2. 一个事务**<font style="color:#DF2A3F;">不能够</font>**看到的数据版本
                1. 在本事务第一次查询之后创建的事务（事务ID比我的事务ID大）
                2. 活跃的（未提交的）事务的修改
        3. MVCC的原理
            1. InnoDB的所有事务都是有编号的，而且会不断累加递增。InnoDB为每行记录都实现了两个隐藏字段：
                1. DB_TRX_ID，6字节，事务ID，数据是在哪个事务插入或者修改为新数据的，就记录为当前事务ID
                2. DB_ROLL_PTR，7字节，回滚指针（<font style="color:#DF2A3F;">可以理解为删除版本号</font>），数据被删除或记录为旧数据时，记录当前事务ID；没有修改或者删除的时候为空
            2. MVCC的查找规则：只能查找创建时间<font style="color:#DF2A3F;">小于等于</font>当前事务ID的数据，<font style="color:#DF2A3F;">和删除时间大于当前事务ID的行</font>（或未删除）
            3. 为了进行上面的判断，我们必须维护一个数据结构，叫Read View（可见性视图）。每个事务都维护一个自己的Read View。
        4. 寻找过程如下：略
        5. **<font style="color:#DF2A3F;">RR中Read View是事务第一次查询的时候建立的（因此RR隔离级别下，MVCC能解决部分幻读问题！特别是在处理范围查询时，仍然存在发生幻读的可能性！！！！所以，InnoDB还使用了一种叫做Next-Key Locks的机制来防止幻读</font>**
        6. **<font style="color:#DF2A3F;">RC的Read View是事务每次查询的时候建立。</font>**
        7. **<font style="color:#DF2A3F;"></font>**

#### Mysql InnoDB 锁的基本类型
    1. MYISAM 支持表锁，InnoDB支持表锁和行锁
        1. 表锁的使用方式：lock tables xxx read; lock tables xxx write; unlock tables;
    2. 表锁和行锁的效率：
        1. 锁定粒度：表锁 > 行锁
        2. 加锁效率：表锁 > 行锁
        3. 冲突概率：表锁 > 行锁
        4. 并发性能：表锁 < 行锁
    3. 锁的类型：
        1. <font style="color:#DF2A3F;">共享锁：Shared Locks</font>
            1. 我们获取一行数据的读锁后，就可以用来读取数据，所以它也叫**<font style="color:#DF2A3F;">读锁</font>**。
            2. 共享锁的作用：因为共享锁会阻塞其他事务的修改，所以可以用在不允许其他事务修改数据的情况下
            3. 用法：select ... lock in share mode;
        2. <font style="color:#DF2A3F;">排它锁：Exclusive Locks</font> 
            1. 是用来操作数据的，所以又叫**<font style="color:#DF2A3F;">写锁</font>**
            2. 一个事务只要获取了一行数据的排它锁，其他事务就不能再获取这一行数据的共享锁和排它锁
            3. 加锁方式：
                1. 自动加排它锁，数据增删改都会默认加上一个排它锁
                2. 手动加排它锁，FOR UPDATE给一行数据加上排它锁
        3. 意向锁：
            1. 意向锁是由数据库自己维护的，这两个是表级别的锁
            2. 当给一行数据加上共享锁之前，数据库会自动在这张表上面加上一个意向共享锁
            3. 当给一行数据加上排他锁之前，数据库会自动在这张表上面加上一个意向排它锁
            4. <font style="color:#DF2A3F;">作用</font>：一张表准备加表锁时，需要先判断有没有其他事务锁定了其中某行，如果一行一行数据区检查的话效率会非常低。因此，引进意向锁之后，就可以直接判断是否存在意向锁，如果有，直接返回失败即可。如果没有，就可以加锁成功。

##### 锁的原理
    4. **<font style="color:#DF2A3F;">InnoDB的锁是通过锁住索引实现的</font>**
    5. 为什么会锁表？**<font style="color:#DF2A3F;">一般因为查询没有使用索引，会进行全表扫描，然后会把每一个隐藏的聚集索引都锁住</font>**

##### 锁的算法
    6. 三种行锁的算法：
    7.略
    8. 记录锁
        1. 对于唯一性的索引（包括唯一索引和主键索引）使用等值查询，精准匹配到一条记录的时候，使用的就是记录锁
    9. 间隙锁
        1. <font style="color:#DF2A3F;">当我们查询的记录不存在，没有命中任何一个record，无论是用等值查询或者范围查询的时候，都是使用间隙锁。</font>
        2. 略
        3. 当查询记录不存在的时候，使用间隙锁。<font style="color:#DF2A3F;">间隙锁主要是阻止插入insert，相同的间隙锁之间不会冲突（相同的间隙锁可以重入）</font>
    10. 临键锁
        1. 当使用了范围查询，不仅仅命中Record记录，还包含Gap间隙，这种情况使用临键值（**<font style="color:#DF2A3F;">是一个左开右闭的区间</font>**）。它是MySQL里面默认的行锁算法。
        2.略
        3. 临键锁，**<font style="color:#DF2A3F;">锁住最后一个Key的下一个左开右闭的区间</font>**<font style="color:#DF2A3F;">，</font>
        4. <font style="color:#DF2A3F;">InnoDB的RR级别就能解决幻读问题就是使用临键锁实现的</font>

##### 四种事务隔离级别的实现
    11. Read Uncommited：不加锁
    12. Read Commited：普通的select都是快照读，使用MVCC实现。**<font style="color:#DF2A3F;">加锁的select都是使用记录锁，因为没有Gap Lock ，所以RC会出现幻读</font>**
    13. Repeatable Read：普通的select使用快照读，底层使用MVCC实现。加锁的select使用记录锁、间隙锁或者临键锁。所以，能解决幻读问题
    14. Serializable：所有的select语句都会被隐式转化为select ... in share mode，会和update、delete互斥

### 四、SQL优化
数据库性能优化思路：

#### 配置优化
    1. 最大连接数max_connections
    2. 客户端默认超时时间wait_timeout，默认28800秒，8小时
    3. 使用连接池，C3P0、阿里的Druid、SpringBoot默认连接池：Hikari。
        1. Druid默认最大连接池大小是8。Hikari默认是10.
        2. 数据库连接池大小建议：机器核数*2+1，例如4核的机器连接池维持9个连接

#### 架构优化
    4. 使用缓存技术，减少数据库的查询压力
    5. 集群，主从复制，一般写入Master节点，读请求可以分担到slave节点。对于读多写少的项目，读写分离对减轻主服务器的访问压力有很大帮助，但是，无法解决单表过大的问题
    6. 分库分表，一般分为：
        1. 垂直分库，减少并发压力，把一个数据库按照业务拆分成不同的数据库
            1. 略
        2. 水平分表，解决存储瓶颈
            1. 把单张表的数据按照一定的规则分布到多个数据库。
    7. 

#### SQL优化思路
##### 慢查询日志
        1. show_query_log，慢查询日志开关
        2. long_query_time，执行超过多少秒才记录日志，默认10s

##### 监控数据库状态
        3. show processlist，查看运行线程
        4. show status，查看数据库运行状态
        5. show engine，查看事务持有表锁、行锁情况

##### <font style="color:#DF2A3F;">分析EXPLAIN 执行计划</font>
        1. id字段，查询序列编号，一个select就会有一个id序号。一个查询包含多个select会有多个id
            1. id值不同的时候，先查询id值大的（**<font style="color:#DF2A3F;">先大后小</font>**）
            2. id值相同，查询顺序是从上到下顺序执行
        2. select_type，查询类型
            1. SIMPLE，简单查询，不包含子查询，不包含关联查询
            2. PRIMARY，子查询中的主查询，也就是最外面的那层查询
            3. SUBQUERY，子查询中所有内层的查询都是SUBQUERY类型
            4. DERIVED，衍生查询，表示得到最终结果之前会用到的临时表
            5. 在连接查询中，先查询的表叫做驱动表，后查询的表叫被驱动表
        3. type连接类型，常见的连接类型：system>const>eq_ref>ref>range>index>all，以上除了all都能用上索引
            1. <font style="color:#DF2A3F;">const</font>，主键索引或者唯一索引，<font style="color:#DF2A3F;">只能查询到一条数据</font>
            2. system，是const的一种特例，只有一行满足条件
            3. eq_ref，通常出现在多表的join查询，被驱动表通过唯一性索引（UNIQUE或PRIMARY KEY）进行访问，此时被驱动表的访问方式就是eq_ref。eq_ref是除了const之外最好的访问类型，
            4. ref，查询用到了非唯一索引，或者关联操作只使用到了索引的最左前缀。
            5. range，索引范围扫描，where后面是between and、<、>、<=、>=、in这些，type类型就是range
            6. index，Full Index Scan，<font style="color:#DF2A3F;">查询全部索引中的数据</font>（比不走索引快）
            7. <font style="color:#DF2A3F;">all，Full Table Scan，全表扫描，如果没有索引或者没用到索引，type就是all</font>
            8. null，不用访问表或者索引就能得到结果
            9. **<font style="color:#DF2A3F;">结论：一般保证查询的type至少能达到range级别，最好能达到ref</font>**
        4. possible_key：可能用到的索引，如果是NULL就表示没有用到索引，possible_key可以有一个或者多个，可能用到索引并不代表一定用到索引
        5. key：实际用到索引，如果是NULL就表示没有用到索引。如果possible_key为空，key可能会有值（覆盖索引的情况）
        6. key_len
            1. 索引的长度（使用的字节数）。跟索引字段的类型、长度有关。
            2. utf8mb4编码1个字符4个字节。
        7. rows
            1. MySQL认为扫描多少行才能返回请求数据，是一个预估值。一般来说行数越少越好。
        8. filtered
            1. 表示存储引擎返回的数据在Server层过滤后，剩下多少满足查询的记录数量的比例，是一个比例。
            2. 比例低，说明存储引擎返回的数据需要经过大量过滤，这个是会消耗性能的
        9. ref
            1. 使用哪个列或者常数和索引一起从列表中筛选数据
        10. extra
            1. 执行计划的额外的信息说明
            2. using index：使用了覆盖索引，不需要回表
            3. using where：where过滤，存储引擎返回的记录不满足查询条件，需要再Server层过滤
            4. using index condition（索引下推）
            5. using temporary（使用临时表）

##### SQL与索引优化
1. 创建索引或者联合索引
2. 改写SQL，这里需要平时积累经验，例如：
    1. 使用小表驱动大表
    2. 用join来代替子查询
    3. or 改为union
    4. 大偏移量的limit，先过滤再排序
3. 如果SQL本身解决不了问题，就要上升到表架构和架构

### 五、其他
#### 1、MySQL中Order By 排序实现原理
<font style="color:rgb(64, 64, 64);">MySQL 中的 </font>`<font style="color:rgb(64, 64, 64);">ORDER BY</font>`<font style="color:rgb(64, 64, 64);"> 排序实现主要依赖于两种机制：</font>**<font style="color:rgb(64, 64, 64);">索引排序</font>**<font style="color:rgb(64, 64, 64);">和</font>**<font style="color:rgb(64, 64, 64);">文件排序（Filesort）。</font>**<font style="color:rgb(64, 64, 64);">具体实现方式取决于查询的索引设计和数据量大小。</font>

+ **<font style="color:rgb(64, 64, 64);">索引排序（利用索引直接返回有序数据）</font>**
    - <font style="color:rgb(64, 64, 64);">如果</font>`<font style="color:rgb(64, 64, 64);">ORDER BY</font>`<font style="color:rgb(64, 64, 64);">的列已经存在有序索引，</font><font style="color:rgb(64, 64, 64);">MySQL 可以直接通过索引的顺序读取数据，避免额外排序。</font>
    - <font style="color:rgb(64, 64, 64);">例如</font><font style="color:rgb(255, 255, 255);background-color:rgb(80, 80, 90);"></font>

```sql
-- 假设表 `users` 有索引 `idx_age` (age)
SELECT * FROM users ORDER BY age;
```

    - **<font style="color:rgb(64, 64, 64);">执行过程</font>**<font style="color:rgb(64, 64, 64);">：</font>
        1. <font style="color:rgb(64, 64, 64);">通过索引</font><font style="color:rgb(64, 64, 64);"> </font>`<font style="color:rgb(64, 64, 64);">idx_age</font>`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">的 B+Tree 结构，按</font><font style="color:rgb(64, 64, 64);"> </font>`<font style="color:rgb(64, 64, 64);">age</font>`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">顺序遍历叶子节点。</font>
        2. <font style="color:rgb(64, 64, 64);">直接从索引中读取数据（如果索引是覆盖索引）或通过主键回表查询完整行数据。</font>
+ **<font style="color:rgb(64, 64, 64);">文件排序</font>**
    - <font style="color:rgb(64, 64, 64);">当无法利用索引排序时，MySQL 会使用 </font>**<font style="color:rgb(64, 64, 64);">文件排序（Filesort）</font>**<font style="color:rgb(64, 64, 64);">，将符合条件的记录加载到内存或临时磁盘文件中排序。具体分为两种模式：</font>
    - **<font style="color:rgb(64, 64, 64);">单路排序（Single-Pass）</font>**
        * **<font style="color:rgb(64, 64, 64);">适用场景</font>**<font style="color:rgb(64, 64, 64);">：  
</font><font style="color:rgb(64, 64, 64);">当查询的字段总长度较小（由</font><font style="color:rgb(64, 64, 64);"> </font>`<font style="color:rgb(64, 64, 64);">max_length_for_sort_data</font>`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">参数控制）时使用。</font>
        * **<font style="color:rgb(64, 64, 64);">流程</font>**<font style="color:rgb(64, 64, 64);">：</font>
            1. <font style="color:rgb(64, 64, 64);">读取所有满足条件的行，将</font><font style="color:rgb(64, 64, 64);"> </font>`<font style="color:rgb(64, 64, 64);">ORDER BY</font>`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">列和查询所需的字段存入</font><font style="color:rgb(64, 64, 64);"> </font>**<font style="color:rgb(64, 64, 64);">排序缓冲区（Sort Buffer）</font>**<font style="color:rgb(64, 64, 64);">。</font>
            2. <font style="color:rgb(64, 64, 64);">在内存中对排序缓冲区中的数据进行快速排序。</font>
            3. <font style="color:rgb(64, 64, 64);">直接返回排序后的结果。</font>
    - **<font style="color:rgb(64, 64, 64);">双路排序（Two-Pass）</font>**
        * **<font style="color:rgb(64, 64, 64);">适用场景</font>**<font style="color:rgb(64, 64, 64);">：  
</font><font style="color:rgb(64, 64, 64);">当查询的字段总长度较大时（超出</font><font style="color:rgb(64, 64, 64);"> </font>`<font style="color:rgb(64, 64, 64);">max_length_for_sort_data</font>`<font style="color:rgb(64, 64, 64);">）。</font>
        * **<font style="color:rgb(64, 64, 64);">流程</font>**<font style="color:rgb(64, 64, 64);">：</font>
            1. <font style="color:rgb(64, 64, 64);">仅读取</font><font style="color:rgb(64, 64, 64);"> </font>`<font style="color:rgb(64, 64, 64);">ORDER BY</font>`<font style="color:rgb(64, 64, 64);"> </font><font style="color:rgb(64, 64, 64);">列和主键（或行指针），存入排序缓冲区。</font>
            2. <font style="color:rgb(64, 64, 64);">在内存中对这些列排序。</font>
            3. <font style="color:rgb(64, 64, 64);">根据排序后的主键（或行指针）回表查询完整数据。</font>

#### 2、增删改操作自动开启事务
#### 3、不走索引，一定是全表扫描
