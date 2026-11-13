# 存储引擎
## 存储引擎简介
### 概念：存储引擎就是存储数据、建立索引、更新/查询数据等技术的实现方式。存储引擎是基于表的，而不是基于库的，所以存储引擎也可被称为表类型。
### 在创建表时，指定存储引擎
```sql
create table 表名(
	字段1 字段1类型[commit 字段1注释]
	...
	字段n 字段n类型[commit 字段n注释]
)engine = innodb [commit 表注释];
```
### 查看当前数据库支持的存储引擎
```sql
show engines;
```
## 存储引擎特点
### InnoDB
#### 介绍：InnoDB是一种兼顾高可靠性和高性能的通用存储引擎，在MySQL5.5之后，InnoDB是默认的MySQL存储引擎。
#### 特点：
1. DML操作遵循ACID模型，支持事务
2. 行级锁，提高并发访问性能
3. 支持外键foreign key 约束，保证数据的完整性和正确性
#### 文件
xxx.ibd：xxx代表的是表名，innoDB引擎的每张表都会哦对应这样一个表空间文件，存储该表的表结构（frm、sdi）、数据和索引。
参数：innodb_file_per_table
#### 逻辑存储结构
| Tablespace | Segment | Extent | Page |
| :--------: | :-----: | :----- | ---- |
|  Segment   | Extent  | Page   | Row  |
|  Segment   | Extent  | Page   | Row  |
|  Segment   | Extent  | Page   | Row  |
|  Segment   | Extent  | Page   | Row  |
### MyISAM
#### 介绍
MyISAM是MySQL早期的默认存储引擎
#### 特点
1. 不支持事务，不支持外键
2. 支持表锁，不支持行锁
3. 访问速度快
#### 文件
xxx.myd：存储表结构信息
xxx.myi：存储数据
xxx.sdi：存储索引
### Memory
#### 介绍
Memory引擎的表数据存储在内存中的，由于受到硬件问题、或断电问题的影响，只能将这些表作为临时表或缓存使用
#### 特点
1. 内存存放
2. hash索引（默认）
#### 文件
xxx.sdi：存储表结构信息
### 总表
|    特点    |   InnoDB    | MyISAM | Memory |
| :------: | :---------: | :----: | :----: |
|   存储限制   |    64TB     |   有    |   有    |
|   事务安全   |     支持      |   -    |   -    |
|   锁机制    |     行锁      |   表锁   |   表锁   |
| B+tree索引 |     支持      |   支持   |   支持   |
|  Hash索引  |      -      |   -    |   支持   |
|   全文索引   | 支持（5.6版本之后） |   支持   |   -    |
|   空间使用   |      高      |   低    |  N/A   |
|   内存使用   |      高      |   低    |   中等   |
|  批量插入速度  |      低      |   高    |   高    |
|   支持外键   |     支持      |   -    |   -    |
## 存储引擎选择
### InnoDB：是MySQL的默认存储引擎，支持事务、外键。如果应用对事务的完整性有比较高的要求，在并发条件下要求数据的一致性，数据操作除了插入和查询之外，还包含很屌的更新、删除操作，那么InnpDB存储引擎是比较合适的选择。
### MyISAM：如果应用是以读操作和插入炒作位置，只有很少的更新和删除操作，并且对事务的完整性，并发性要求不是很高，那么选择这个存储引擎是非常合适的。
### Memory：将所有数据存储在内存中，访问速度快，通常用于临时表及缓存。Memory的缺陷就是对标的大小有限制，太大的表无法缓存在内存中，而且无法保障数据的安全性。
# 索引
## 索引概述
### 介绍
==索引（index）是帮助MySQL高效获取数据的数据结构（有序）。==在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种引用（指向）数据，这样就可以在这些数据结构上实现高级查找算法，这种数据结构就是索引。
### 优缺点
|               优点               | 缺点                                                       |
| :----------------------------: | :------------------------------------------------------- |
|      提高数据检索的效率，降低数据库的IO成本      | 索引列也是要占用空间的                                              |
| 通过索引对数据进行排序，降低数据排序的成本，降低CPU的消耗 | 索引大大提高了查询效率，同时却也降低更新表的速度，如对表进行insert、update、delete时，效率降低 |
## 索引结构（一般没有特别指定索引，都指的是B+树索引）

|      索引结构       | 描述                                         | LnnoDB    | MyISAM | Memory |
| :-------------: | :----------------------------------------- | --------- | ------ | ------ |
|    B+Tree索引     | 最常见的索引类型，大部分引擎都支持B+树索引                     | 支持        | 支持     | 支持     |
|     Hash索引      | 底层数据结构是用哈希表实现的，只有精确匹配索引列的查询才能有效，不支持范围查询    | 不支持       | 不支持    | 支持     |
|  R-Tree（空间索引）   | 空间索引是MyISAM引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少 | 不支持       | 支持     | 不支持    |
| Full-text（全文索引） | 是一种通过建立倒排索引，快速匹配文档的方式。类似于Lucene，Solr，ES    | 5.6版本之后支持 | 支持     | 不支持    |

## 索引分类
### 常规分法：

|  分类  |             含义             |      特点      | 关键字      |
| :--: | :------------------------: | :----------: | :------- |
| 主键索引 |        正对于表中主键创建的索引        | 某人自动创建，只能有一个 | primary  |
| 唯一索引 |      避免同一个表中某数据列中的值重复      |    可以有多个     | unique   |
| 常规索引 |          快速定位特定数据          |    可以有多个     |          |
| 全文索引 | 全文索引查询的是文本中的关键词，而不是比较索引中的值 |    可以有多个     | fulltext |
### 按照存储形式分：
|          分类           | 含义                            | 特点         |
| :-------------------: | :---------------------------- | ---------- |
| 聚集索引（Clustered Index） | 将数据存储于索引结构的叶子节点保存了行数据         | 必须有，而且只有一个 |
| 二级索引（Secondary Index） | 将数据于索引分开存储，索引结构的叶子节点关联的是对应的主键 | 可以存在多个     |
#### 聚集索引选取规则：
1. 如果存在主键，主键就是聚集索引
2. 如果不存在主键，将使用第一个唯一（unique）索引作为聚集索引
3. 如果表没有主键，或没有合适的唯一索引，则InnoDB会自动生成一个rowid作为隐藏的聚集索引
## 索引语法
### 创建索引
```mysql
create [unique | fulltext] index index_name on table_name (inddex_col_name,…);
```
### 查看索引
```mysql
show index from table_name;
```
### 删除索引
```mysql
drop index index_name on table_name;
```
## SQL性能分析
### SQL执行频率
MySQL客户端连接成功后，通过```show [session|global] status```命令可以提供服务器状态信息。通过以下指令，可以查看当前数据库的insert、update、delete、select的访问频次：
```mysql
   show global status like 'com_______';
```
### 慢查询日志
慢查询日志记录了所有执行时间超过指定参数（long_query_time，单位：秒，默认10秒）的所有SQL语句的日志。MySQL的慢查询日志默认没有开启，需要在MySQL的配置文件（/etc/my.cnf）中配置如下信息：
```
#开启MySQL慢日志查询开关
slow_query_log=1
#设置慢日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询，机理慢查询日志
long_query_time=2
```
配置完毕后，通过以下指令重新启动MySQL服务器进行测试，查看慢日志中记录的信息/var/lib/mysql/localhost-slow.log
```
#重启指令
systemctl resart mysqlb
```
### profile详情
`show profiles`能够在做SQL优化时帮助我们了解时间都耗费到哪里去了。通过have_profiling参数，能够看到当前MySQL是否支持profile操作：`select @@have_profiling;`
默认profiling是关闭的，可以通过set语句在session/global级别开启profiling：`set profiling=1;`
#### 语法
1. 查看每一条SQL的耗时基本情况
```mysql
show profiles;
```
2. 查看指定query_id的SQL语句各个阶段的耗时情况
```mysql
show profile for query query_id;
```
3. 查看指定query_id的SQL语句CPU的使用情况
```mysql
show profile cpu for query query_id;
```
### explain执行计划
explain或者desc命令获取MySQL如何执行select语句的信息，包括在select语句执行过程中表如何连接和连接的顺序
#### 语法：
```mysql
# 直接在select语句前加上关键字explain / desc
explain select 字段列表 from 表名 where 条件;
desc select 字段列表 from 表名 where 条件;
```
#### 各字段含义：
|      字段      | 含义                                                                                                                       |
| :----------: | :----------------------------------------------------------------------------------------------------------------------- |
|      id      | select查询的序列号，表示查询中执行select子句或是操作表的顺序（id相同，执行顺序从上到下；id不同，值越大，越先执行）                                                        |
| select_type  | 表示select的类型，常见的取值有simple（简单表，即不用表连接或者子查询）、primary（主查询，即外层的查询）、union（union中的第二个或者后面的查询语句）、subquery（select/where之后包含了子查询）等 |
|     type     | 表示连接类型，性能由好到差的连接类型为null、system、const、eq_ref、ref、range、index、all                                                          |
| possible_key | 显示可能应用在这张表上的索引，一个或多个                                                                                                     |
|     key      | 实际使用的索引，如果为null，则没有使用索引                                                                                                  |
|   ken_len    | 表示索引中使用的字节数，该值为索引字段最大可能长度，并非实际使用长度，在不损耗精确性的前提下，长度越短越好                                                                    |
|     rows     | MySQL认为必须要执行查询的行数，在InnoDB引擎的表中，是一个估计只，可能并不总是准确的                                                                          |
|   filtered   | 表示返回结果的行数占需读取行数的百分比，filtered的值越大越好                                                                                       |
## 索引使用
### 验证索引效率
1. 在未建立索引之前，执行如下语句，查询SQL的耗时(sn没有主键，效率极低)
   ```mysql
   select * from tb_sku where sn = '100000003145001';
   ```
2. 针对字段创建索引
   ```mysql
   create index idx_sku_sn on tb_sku(sn);
   ```
### 最左前缀法则
1. 如果索引了多列（联合索引），要遵守最左前缀法则。最左前缀法则指的是查询从索引的最左列开始，并且不跳过索引中的列
2. 如果跳跃某一列，索引将部分失效（后面的字段索引失效）。
### 范围查询
1. 联合索引中，出现范围查询（>,<），范围查询右侧的列索引失效
### 索引列运算
1. 不要在索引列上进行运算操作，索引将失效
### 字符串不加引号
1. 字符串类型字段使用时，不加引号，索引将失效
### 模糊查询
1. 如果仅仅是尾部模糊匹配，索引不会失效。如果头部模糊匹配，索引失效
### or连接的条件
1. 用or分割的条件，如果or前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到
2. 简单来说，两个都要有索引，不然系统默认查全表all
### 数据分布影响
1. 如果MySQL评估使用索引比全表更慢，则不使用索引
### SQL提示
1. SQL提示，是优化数据库的一个重要手段，简单来说，就是在SQL语句中加入一些人为的提示来达到优化操作的目的
2. 写法：
   - use index(建议使用我推荐的索引）：
     ```mysql
     explain select * from tb_user use index(idx_user_pro) where profession = '软件工程';
     ```
   - ignore index（忽略这个索引）：
    ```mysql
     explain select * from tb_user ignore index(idx_user_pro) where profession = '软件工程';
    ```
   - force index（强制使用我推荐的索引）：
     ```mysql
     explain select * from tb_user force index(idx_user_pro) where profession = '软件工程';
     ```
### 覆盖索引
1. 尽量使用覆盖索引（查询使用了索引，并且需要返回的列，在该索引中已经全部能够找到），减少`select *`
2. 注意：
   - `using index condition`：查询使用了索引，但是需要回表查询数据
   - `using where;using index`：查询使用了索引，但是需要的数据都在索引列中能找到，索引不需要回表查询数据
### 前缀索引
#### 定义：
当字段类型为字符串（varchar，text等）时，有时候需要索引很长的字符串，这会让索引变得很大，查询时，浪费大量的磁盘IO，影响查询效率。此时可以只将字符串的一部分前缀，建立索引，这样可以大大节约索引空间，从而提高索引效率
#### 语法
```mysql
create index idx_xxxx on table_name(column(n));
```
### 前缀长度
   可以根据索引的选择性来决定，而选择性是指不重复的索引值（基数）和数据表的记录总数的比值（如下是计算公式），索引选择性越高则查询效率越高，唯一索引的选择性是1，这是最好的索引选择性，性能也是最好的
```mysql
select count(distinct email) / count(*) from tb_user;
select count(distinct substring(email,1,5)) / count(*) from tb_user;
```
### 单列索引与联合索引
#### 定义：
1. 单列索引：即一个索引只包含单个列
2. 联合索引：即一个索引包含了多个列
3. 在业务场景中，如果存在多个查询条件，考虑针对字段建立索引时，建议建立联合索引，而非单列索引
4. 多条件联合查询时，MySQL优化器会评估哪个字段的索引效率更高，会选择该索引完成本次查询
## 索引设计原则
1. 针对于数据量较大（百万级），且查询比较频繁的表建立索引
2. 针对于常作为查询条件（where）、排序（order by）、分组（group by）操作的字段建立索引
3. 尽量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高
4. 如果是字符串类型的字段，字段的长度较长，可以针对字段的特点，建立前缀索引
5. 尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率
6. 要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大，会影响增删改的效率
7. 如果索引列不能存储null值，请在创建表时使用not null约束它。当优化器知道每列是否包含null值时，它可以更好地确定哪个索引最有效地用于查询
# SQL优化
## 插入数据
### insert优化（小批量插入）（为了减少SQL的开销，频繁的数据传输，消耗网络）
```mysql
#常规的插入方式
insert into tb_test values(1,'Tom');
insert into tb_test values(2,'Cat');
insert into tb_test values(3,'Jerry');
……
```
#### 批量插入
```mysql
insert into tb_test values(1,'Tom'),(2,'Cat'),(3,'jerry');
```
#### 手动提交事务
```mysql
start transaction;
insert into tb_test values(1,'Tom'),(2,'Cat'),(3,'Jerry');
insert into tb_test values(4,'Tom'),(5,'Cat'),(6,'Jerry');
insert into tb_test values(7,'Tom'),(8,'Cat'),(9,'Jerry');
commit;
```
#### 主键顺序插入（在主键优化里拓展）
1. 主键顺序插入性能高于乱序插入
### 大批量插入数据（load指令）

1. 客户端连接服务端时，加上参数 --local-infile
   ```mysql
   mysql --local-infile -u root -p
   ```
2. 设置全局参数local_infile为1，开启从本地加载文件导入数据的开关
   ```mysql
   set global local_infile = 1`
   ```
3. 执行load指令将准备好的数据，加载到表结构中
   ```mysql
   load data local infile '/root/sql1.log' into table 'tb_user' fields terminated by ',' lines terminated by '\n';
   ```
## 主键优化
### 数据组织方式
1. 在InnoDB存储引擎中，表数据都是根据主键顺序组织存放的，这种存储方式的表称为索引组织表（index organized table IOT）
### 页分裂
1. 页可以为空，也可以填方一半，也可以填充100%。每个页包含2-N行数据（如果一行数据多大，会行溢出），根据主键排列
### 页合并
1. 当删除一行记录时，实际上记录并没有被物理删除，只是记录被标记（flaged）为删除并且它的空间变得允许被其他记录声明使用。
2. 当页中删除的记录达到MERGE_THRESHOLD（默认为页的50%），InnoDB会开始寻找最近的页（前或后）看看是否可以将两个页合并以优化空间使用
3. MERGE_THRESHOLD：合并页的阈值，可以自己设置，在创建表或者创建索引时指定
### 主键设计原则
1. 满足业务需求的情况下，尽量降低主键的长度
2. 插入数据时，尽量选择顺序插入，选择AUTO_INCREMENT自增主键
3. 尽量不要使用UUID做左键或者其他自然主键，如身份证号
4. 业务操作时，避免对主键的修改
## order by优化
1. Using filesort：通过表的索引或全表扫描，读取满足条件的数据行，然后在排序缓冲区sort buffer中完成排序操作，所有不是通过索引直接返回排序结果的排序都叫FileSort排序
2. Using index：通过有序索引顺序扫描直接返回有序数据，这种情况即为using index，不需要额外排序，操作效率高
3. 注意：
   - 根据排序字段建立合适的索引，多字段排序时，也遵循最左前缀法则
   - 尽量使用覆盖索引
   - 多字段排序，一个升序一个降序，此时需要注意联合索引在创建时的规则（ASC/DESC）
   - 如果不可避免的出现filesort，大数据量排序时，可以适当真大排序缓冲区大小sort_buffer_size（默认256k）
## group by优化
### 为分组字段建索引
1. 给`group by`后面的字段建单列/复合索引
2. 遵守最左前缀法则
3. 最好做到覆盖索引（Using index）
4. 注意：出现Using temporary或Using filesort意味着太慢了需要优化
## limit优化
### 原因：
比如说有个需求是`limit 2000000,10`，此时需要MySQL排序前2000010记录，仅仅返回2000000 ~ 2000010的记录，其他记录丢弃，查询排序的代价非常大
### 优化思路：
一般分页查询时，通过创建覆盖索引能够比较好地提高性能，可以通过所有加子查询形式进行优化
```mysql
explain select * from tb_sku t,(select id from tb_sku order by id limit 2000000,10) a where t.id = a.id;
```
## count优化
```mysql
explain select count(*) from tb_user;
```
1. MyISAM引擎把一个表的总行书存储在了磁盘上，因此执行`count(*)`的时候会直接返回这个数，效率很高
2. InnoDB引擎执行`count(*)`的时候，需要把数据一行一行地从引擎里面读出来，然后累积计数
### 优化思路：自己计数
### count的几种用法
1. `count()`是一个聚合函数，队友返回的结果集，一行行地判断，如果count函数的参数不是NULL，累计值就加1，否则不加，最后返回累计值
2. 用法：`count(*)`、`count(字段)`、`count(1)`
3. `count(主键)`
   InnoDB引擎会遍历整张表，把每一行的主键id值都取出来，返回给服务层。服务层拿到主键后，直接按行进行累加（主键不可能为null）
4. `count(字段)`
   没有not null约束：InnoDB引擎会遍历整张表把每一行的字段值都取出来，返回给服务层，服务层判断是否为null，不为null，计数累加
   有not null约束：InnoDB引擎会遍历整张表把每一行的字段值都取出来，返回给服务层，直接按行进行累加
5. `count(1)`
   InnoDB引擎遍历整张表，但不取值。服务层对一返回的每一行，翻一个数字“1”进去，按照行进行累加
6. `count(*)`
   InnoDB引擎并不会把全部字段取出来，而是专门做了优化，不取值，服务层直接按行进行累加
7. 按照效率排序，`count(字段)`<`count(主键id)`<`count(1)`≈`count(*)`，所以尽量使用`count(*)`
## update优化
```mysql
update student set no='2000100100' where id = 1;
```
```mysql
update student set '2000100105' where name = '韦一笑';
```
InnoDB的行锁是针对索引加的锁，不是针对记录加的锁，并且该索引不能失效，狗子会从行锁升级成表锁
# 视图/存储过程/触发器
## 视图
### 介绍
视图（view）是一种虚拟存在的表。视图中的数据并不在数据库中实际存在，行业累数据来自定义视图的查询中使用的表，并且是在使用视图时动态生成的
通俗的讲，视图只保存了查询的SQL逻辑，不保存查询结果。所以我们在创建视图的时候，主要的工作就落在创建这条SQL查询语句上
### 创建
```mysql
create [or replace] view 视图名称[(列名列表)] as select语句 [with [cascaded | local] check option]
```
### 查询
```mysql
#查看创建视图语句：
show create view 视图名称;
#查看视图数据：
select * from 视图名称……;
```
### 修改
```mysql
#方式一：
create [or replace] view 视图名称[(列名列表)] as select语句 [with [cascaded | local] check option]
#方式二：
alter view 视图名称[(列名列表)] as select语句[with[cascaded | local] check option]
```
### 删除
```mysql
drop view [if exists] 视图名称[视图名称]……;
```
### 视图的检查选项
当使用`with check option`子句创建视图时，MySQL会通过视图检查正在更改的每一行，例如：插入、更改、删除，以使其符合视图的定义。MySQL允许基于另一个视图创建视图，它还会检查一来视图中的规则以保持一致性。为了确认检查的范围，mysql提供了两个选项：cascaded和local，默认值为cascaded
### 视图的更新
要使视图可更新，视图中的行与基础表中的行之间必须存在一对一的关系。如果视图包含一下任何一项，则该视图不可更新：
1. 聚合函数或者窗口函数（`sum()`、`min()`、`max()`、`count()`等
2. distinct
3. group by
4. having
5. union或者union all
### 作用
1. 简单
   视图不仅可以简化用户对数据的理解，也可以简化他们的操作。那些被经常使用的查询可以被定义为视图，从而使得用户不必为以后的操作每次指定全部的条件
2. 安全
   数据库可以授权，但不能授权到数据库特定行和特定的列上。通过视图用户只能查询和修改他们所能见到的数据
3. 数据独立
   视图可帮助用户屏蔽真实表结构变化带来的影响



















## 存储过程
### 介绍
存储过程是实现经过编译并存储在数据库中的一段SQL语句的集合，调用存储过程可以简化应用开发人员的很多工作，减少数据在数据库和应用服务器之间的传输，对于提高数据处理的效率是有好处的。
存储过程思想上很简单，就是数据库SQL语言层面的代码封装与重用
### 特点
1. 封装，复用
2. 可以接受参数，也可以返回数据
3. 减少网络交换，效率提升
### 创建
```mysql
create procedure 存储过程名称([参数列表])
begin
	--SQL语句
end;
```
### 调用
```mysql
call 名称([参数]);
```
### 查看
```mysql
#查询指定数据库的存储过程及状态信息
select * from information_schema.ROUTINES where routine_schema = 'xxx';
#查询某个存储过程的定义
show create procedure 存储过程名称;
```
### 删除
```mysql
drop procedure [if exists] 存储过程名称;
```
### 注意：在命令行中，执行创建存储过程的SQL时，需要通过关键字delimiter指定SQL语句的结束符
```mysql
#将结束符改为$$，输入完创建指令记得改回来
delimiter $$
```
### 变量
#### 系统变量
系统变量是MySQL服务器提供，不是用户定义的，属于服务器层面。分为全局遍历（global）、会话变量（session）
##### 查看系统变量
```mysql
#查看所有系统变量
show [session | global] variables;
#可以通过like模糊匹配方式查找变量
show [session | global] variables like '……';
#查看指定变量的值
```
##### 设置系统变量
```mysql
set [session | global] 系统变量名 = 值;
set @@[session | global] 系统变量名 = 值;
```
##### 注意：
如果没有指定session/global，默认是session，会话变量
MySQL服务重新启动之后，所设置的全局参数会失效，要想不失效，可以在/etc/my.cnf中配置
#### 用户定义变量
用户定义变量是用户根据需要自己定义的变量，用户变量不用提前声明，在用的时候直接用“@变量名”使用就可以。其作用域为当前连接
##### 赋值
```mysql
set @var_name = expr [,@var_name = expr]……;

set @var_name := expr [,@var_name := expr]……;

select @var_name := expr [,@var_name := expr]……;

select 字段名 into @var_name from 表名;
```
##### 使用
```mysql
select @var_name;
```
##### 注意：
用户定义的变量无需对其进行声明或初始化，只不过获取到的值为null
#### 局部变量
局部变量是根据需要定义的在局部生效的变量，访问之前，需要declare声明。可用作存储过程内的局部变量和输入参数，局部变量的范围是在其内声明的begin……end块
##### 声明
```mysql
declare 变量名 变量类型[default ……];
```
变量类型就是数据库字段类型：int、bigint、char、varchar、date、time等
##### 赋值
```mysql
set 变量名 = 值;
set 变量名 := 值;
select 字段名 into 变量名 from 表名……;
```
### if
#### 语法
```mysql
if 条件1 then
……
elseif 条件2 then
……
else
……
end if;
```
### 参数
|  类型   |          含义           | 备注  |
| :---: | :-------------------: | :-- |
|  in   | 该类参数作为输入，也就是需要调用时传入值  | 默认  |
|  out  | 该类参数作为输出，也就是参数可以作为返回值 |     |
| inout |  既可以作为输入参数，也可以作为输出参数  |     |
#### 用法
```mysql
create procedure 存储过程名称([in/out/inout 参数名 参数类型])
begin
	sql语句
end;
```
### case
#### 语法
##### 语法一
```mysql
case case_value
	when when_value1 then statement_list1
	[when when_value2 then statement_list2]……
	[else statement_list]
end case;
```
##### 语法二
```mysql
case
	when search_condition1 then statement_list1
	[when search_condition2 then statement_list2]
	[else statement_list]
end case;
```
### while
while循环是有条件的循环控制语句。满足条件后，再执行循环体中的sqk语句。
#### 语法
```mysql
#先判定条件，如果条件为true，则执行逻辑，否则，不执行逻辑
while 条件 do
	sql逻辑……
end while;
```
### repeat
repeat是有条件的循环控制语句，当满足条件的时候退出循环
```mysql
#先执行一次逻辑，然后判定逻辑是否满足，如果满足，则退出。如果不满足，则继续下一次循环
repeat
	sql逻辑……
	until 条件
end repeat;
```
### loop
loop实现简单的循环，如果不在sql逻辑中增加退出循环的条件，可以用其来实现简单的死循环。loop可以配合以下两个语句使用：
1. leave：配合循环使用，退出循环
2. iterate：必须用在循环中，作用是跳过当前循环剩下的语句，直接进入下一次循环
#### 语法：
```mysql
[begin_label:] loop
	sql逻辑……
end loop [end_label];
```
```mysql
#退出指定标记的循环体
leave label;
#直接进入下一次循环
iterate label;
```
### 游标
游标（cursor）是用来存储查询结果集的数据类型，在存储过程和函数中可以使用游标对结果集进行循环的处理。游标的使用保括游标的声明、open、fetch和close
#### 语法
##### 声明游标
```mysql
declare 游标名称 cursor for 查询语句;
```
##### 打开游标
```mysql
open 游标名称;
```
##### 获取游标记录
```mysql
fetch 游标名称 into 变量[,变量];
```
##### 关闭游标
```mysql
close 游标名称;
```
### 条件处理程序
条件处理程序（handler）可以用来定义在流程控制结构中遇到问题是相应的处理步骤
#### 语法：
```mysql
declare handler_action handler for condition_value[,condition_value]…statement;
handler_action
	continue:继续执行当前程序
	exit:终止执行当前程序
condition_value
	sqlstate sqlstate_value:状态码，如02000
	sqlwarning:所有以01开头的sqlstate代码的简写
	not found:所有以02开头的sqlstate代码的简写
	sqlexception:所有没有被sqlwarning或not found捕获的sqlstate代码的简写
```
## 存储函数
存储函数是有返回值的存储过程，存储函数的参数只能是in类型
### 语法
```mysql
create function 存储函数名称([参数列表])
returns type [characteristic…]
begin
	sql语句
	return…;
end;
characteristic说明:
1.determinstic：相同的输入参数总是产生相同的结果
2.no sql：不包含sql语句
3.reads sql data：包含读取数据的语句，但不包含写入数据的语句
```
## 触发器
1. 触发器是与表有关的数据库对象，指在insert/update/delete之前或之后，触发并执行触发器中定义的sql语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性，日志记录，数据校验等操作
2. 使用别名OLD和NEW来引用触发器中发生变化的记录内容，这与其他的数据库是相似的。现在触发器还只支持行级触发，不支持语句级触发

|   触发器类型    |            NEW和OLD            |
| :--------: | :---------------------------: |
| insert型触发器 |       new表示将要或者已经新增的数据        |
| update型触发器 | old表示修改之前的数据，new表示将要或已经修改后的数据 |
| delete型触发器 |       old表示将要或者已经删除的数据        |
### 语法
#### 创建
```mysql
create trigger trigger_name
before/after insert/update/delete
on tn;_name for each row -- 行级触发器
begin
	trigger_stmt;
end;
```
#### 查看
```mysql
show triggers;
```
#### 删除
```mysql
drop trigger [schema_name.]trigger_name; -- 如果没有指定schema_name，默认为当前数据库
```
# 锁
## 概述
### 介绍
锁是计算机协调多个进程或线程并发访问某一资源的机制。在数据库中，除传统的计算资源（CPU、RAM、I/O）的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂
### 分类
按照锁的粒度分：
- 全局锁：锁定数据库中的所有表
- 表级锁：每次操作锁住整张表
- 行级锁：每次操作锁住对应的行数据
## 全局锁
### 介绍
全局锁就是对整个数据库实例加锁，加锁后整个实例就处于制度状态，后续的DML语句，DDL语句，以及更新操作的失误提交语句都将被阻塞
其典型的使用场景是做全库的逻辑备份，对所有的表进行锁定，从而获取一致性视图，保证数据的完整性
### 语法
```mysql
#上全局锁
flush tables with read lock;
#备份到另一个数据库
mysqldump -uroot -p1234 itcast>itcast.sql
#备份完成，解除全局锁
unlock tables;
```
### 特点
数据库中加全局锁，是一个比较重的操作，存在一下问题：
1. 如果在主库上备份，那么在备份期间都不能执行更新，业务基本就得停摆
2. 如果在从库上备份，那么在备份期间从库不能执行主库同步过来的二进制日志（binlog），会导致主从延迟
在InnoDB引擎中，我们可以在备份时加上参数--single-transaction参数来完成不加锁的一致性数据备份
```mysql
mysqldump --single-transaction -uroot -p123456 itcast>itcast.sql
```
## 表级锁
### 介绍
表级锁，每次操作锁住整张表。锁定粒度大，发生锁冲突的概率最高，并发度最低。应用在MyISAM、InnoDB、BDB等存储引擎中。
对于表级锁，主要分为以下三类：
1. 表锁
2. 元数据锁（meta data lock，MDL）
3. 意向锁
### 表锁
#### 对于表锁，分为两类：
1. 表共享读锁（read lock）
2. 表独占写锁（write lock）
#### 语法：
1. 加锁：`lock tables 表名…read/write`
2. 释放锁：`unlock tables / 客户贷断开连接`
### 元数据锁（meta data lock，MDL）
MDL加锁过程是系统自动控制，无需显式使用，在访问同一张表的时候会自动加上。MDL锁主要作用是维护表元数据的数据一致性，在表上有活动失误的时候，不可以对元数据进行写入操作。==为了避免DML与DDL冲突，保证读写的正确性==
在MySQL5.5中引入了MDL，当对一张表进行增删改查的时候，加MDL读锁（共享）；当对表结构进行变更操作的时候，加MDL写锁（排他）

|                  对应SQL                   |                   锁类型                   |                    说明                    |
| :--------------------------------------: | :-------------------------------------: | :--------------------------------------: |
|       `lock tables xxx read/write`       | `shared_read_only/shared_no_read_write` |                                          |
|  `select 、select … lock in share mode`   |              `shared_read`              | 与shared_read、shared_write兼容，与exclusive互斥 |
| `insert、update、delete、select…for update` |             `shared_write`              | 与shared_read、shared_write兼容，与exclusive互斥 |
|              `alter table…`              |               `exclusive`               |                与其他的MDL都互斥                |
##### 插件元数据锁：
```mysql
select object_tybe,object_schema,object_name,lock_type_duration from performance_schema.metadata_locks;
```
#### 意向锁
为了避免DML在执行时，加的行锁与表锁的冲突，在InnoDB中引入了意向锁，使得表锁不用检查每行数据是否加锁，使用意向锁来减少锁的检查
1. 意向共享锁（IS）：由语句select…lock in share mode添加
2. 意向排他锁（IX）：由insert、update、delete、select…for update添加
可以通过以下SQL，查看意向锁及行锁的加锁情况：
```mysql
select object_schema,object_name,index_name,lock_type,lock_mode,lock_data from performance_schema.data_locks;
```
## 行级锁
### 介绍
行级锁，每次操作锁住对应的行数据。锁定粒度最小，发生锁冲突的概率最小，并发度最高。应用在InnoDB存储引擎中
InnoDB的数据是基于索引组织的，行锁是通过对索引上的索引项加锁来实现的，而不是对记录加的锁。对于行级锁，主要分为以下三类：
1. 行锁（Record Lock）：锁定单个行记录的锁，防止其他事务对此行进行update和delete。在RC（read commit）、RR（repeatable read）隔离级别下都支持
2. 间隙锁（Gap Lock）：锁定索引记录间隙（不含该记录），确保索引记录间隙不变，防止其他事务在这个间隙进行insert，产生幻读。在RR隔离级别下都支持
3. 临键锁（Next-Key-Lock）：行锁和间隙锁组合，同时锁住数据，并锁住数据前面的间隙Gap。在RR隔离级别下支持
### 行锁
InnoDB实现了一下两种类型的行锁：
1. 共享锁（S）：运行一个事务去读一行，阻止其他事务获取相同数据集的排他锁
2. 排他锁（X）：运行获取排他锁的事务更新数据，阻止其他事务获取相同数据集的共享锁和排他锁。

| 当前锁类型\请求锁类型 | S（共享锁） | X（排他锁） |
| :---------: | :----: | :----: |
|   S（共享锁）    |   兼容   |   冲突   |
|   X（排他锁）    |   冲突   |   冲突   |

|            SQL            | 行锁类型  | 说明                               |
| :-----------------------: | :---: | :------------------------------- |
|          insert…          |  排他锁  | 自动加锁                             |
|          update…          |  排他锁  | 自动加锁                             |
|          delete…          |  排他锁  | 自动加锁                             |
|          select…          | 不加任何锁 |                                  |
| select…lock in share mode |  排他锁  | 需要手动在select之后加lock in share mode |
|     select…for update     |  排他锁  | 需要手动在select之后加for update         |
#### 行锁-演示
默认情况下，InnoDB在repeatable read事务隔离级别运行，InnoDB使用next-key锁进行搜索和索引扫描，以防止幻读
1. 针对唯一索引进行检索时，对已存在的记录进行等值匹配时，将会自动优化为行锁
2. InnoDB的行锁是针对于索引加的锁，不通过索引条件检索数据，那么InnoDB将对表中的所有记录加锁，此时就会升级为表锁
可以通过以下SQL，插件意向锁及行锁的加锁情况：
```mysql
select object_schema,object_name,index_name,lock_type,lock_mode,lock_data from performance_schema.data_locks;
```
### 间隙锁/临键锁-演示
默认情况下，InnoDB在repeatable read事务隔离级别运行，InnoDB使用next-key锁进行搜索和索引扫描，以防止幻读
1. 索引上的等值查询（唯一索引），给不存在的记录加锁时，优化为间隙锁
2. 索引上的等值查询（普通索引），向右遍历时最后一个值不满足查询需求时，next-key lock退化为间隙锁
3. 索引上的范围查询（唯一索引）--会访问到不满足条件的第一个值为止
注意：间隙锁唯一目的是防止其他事务插入间隙。间隙锁可以共存，一个事务采用的间隙锁不会阻止另一个事务在同一间隙上采用间隙锁
# InnoDB引擎
## 逻辑存储结构
### 表空间：逻辑结构的顶层容器
表空间是 InnoDB 逻辑结构的最高层。当开启 `innodb_file_per_table` 参数时（MySQL 8.0 默认开启），每张表都对应一个独立的表空间文件（`xxx.ibd`），用于存放该表的数据与索引
### 段：按功能对区进行归类
段是表空间的组成部分，主要分为三类：
- 数据段：对应 B+ 树的叶子节点，存放实际的行数据
- 索引段：对应 B+ 树的非叶子节点，存放索引键值
- 回滚段：用于支撑事务的回滚操作
一个段会管理多个“区”
### 区：保证页连续性的空间单元
区是表空间的单元结构，大小固定为 1MB。由于默认页大小为 16KB，因此一个区包含 64 个连续的页。为了保证页的连续性，InnoDB 在分配空间时通常会一次性申请 4~5 个区
### 页：磁盘管理的最小单元
页是 InnoDB 磁盘管理的最小单元，默认大小为 16KB。数据库与磁盘之间的 I/O 交互以页为单位进行，即使只读取一行，也会将整页加载到内存中
### 行：数据的最终存放形式
数据最终按行存放。除了用户定义的列，InnoDB 的行记录中还包含两个关键的隐藏字段：
- **`Trx_id`**：记录最后一次改动该行的事务 ID
- **`Roll_pointer`**：指向 undo 日志的指针，用于查找该行修改前的历史版本
## 架构
MySQL5.5版本开始，默认使用InnoDB存储引擎，它擅长事务处理，具有崩溃回复特性，在日常开发中使用非常广泛
### 内存架构
#### Buffer Pool
缓冲池是主内存中的一个区域，里面可以缓存磁盘上经常操作的真实数据，在执行增删改查操作时，先操作缓冲池中的数据（若缓冲池没有数据，则从磁盘加载并缓存），然后再以一定频率刷新到磁盘，从而减少磁盘IO，加快处理速度
缓冲池以page页为单位，底层采用链表数据结构管理page。根据状态，将page分为三种类型：
- free page：空闲page，未被使用
- clean page：被使用page，数据没有被修改过
- dirty page：脏页，被使用page，数据被修改过，页中数据与磁盘的数据产生了不一致
#### Change Buffer
更改缓冲区（针对于非唯一的二级索引页），在执行DML语句时，如果这些数据Page没有在Buffer Pool中，不会直接操作磁盘，而会将数据变更存在更改缓冲区Change Buffer中，在未来数据被读取时，再将数据合并恢复到Buffer Pool中，再将合并后的数据刷新到磁盘中
##### Change Buffer的意义是什么？
与聚集索引不同，二级索引通常是非唯一的，并且以相对随机的顺序插入二级索引。同样，删除和更新可能会影响索引树中不相邻的二级索引页，如果每一次都操作磁盘，会造成大量的磁盘IO。有了Change Buffer之后，我们可以在缓冲池中进行合并处理，减少磁盘IO
#### Adaptive Hash index
自适应hash索引，用于优化对Buffer Pool数据的查询。InnoDB存储引擎会监控对表上各索引页的查询，如果观察到hash索引可以提升速度，则建立hash索引，称之为自适应hash索引
##### 注意
自适应哈西索引，无需人工干预，是系统根据情况自动完成
##### 参数
adaptive_hash_index
#### Log Buffer
日志缓冲区，用来保存要写入到磁盘中的log日志数据（redo log、undo log），默认大小为16MB，日志缓冲区的日志会定期刷新到磁盘中。如果需要更新、插入或删除许多行的事务，增加日志缓冲区的大小可以节省磁盘I/O
##### 参数
###### InnoDB_log_buffer_size：缓冲区大小
###### InnoDB_flush_log_at_trx_commit：日志刷新到磁盘时机
- 1：日志在每次事务提交时写入并刷新到磁盘
- 0：每秒将日志写入并刷新到磁盘一次
- 2：日志在每次事务提交后写入，并每秒刷新到磁盘一次
### 磁盘结构
#### System Tablespace
系统表空间是更改缓冲区的存储区域。如果表是在系统表空间而不是每个表文件或通用表空间中创建的，它也可能包含表和索引数据。（在MySQL5.x版本中还包含InnoDB数据字典、undolog等
##### 参数
InnoDB_data_file_path
#### General Tablespaces
通用表空间，需要通过`create tablespace`语法创建通用表空间，在创建表时，可以指定该表空间
##### 语法
```mysql
#创建表空间
create tablespace xxxx add
datafile 'file_name'
engine = engine_name;
#创建表的时候把表放到指定表空间
create table xxx …… tablespace ts_name;
```
#### Undo Tablespaces
撤销表空间，MySQL实例在初始化时会自动创建两个默认的undo表空间（初始大小16M），用于存储undo log日志
#### Temporary Tablespaces
InnoDB使用会话临时表空间和全局临时表空间，存储用户创建的临时表等数据
#### Doublewrite Buffer Files
双写缓冲区，InnoDB引擎将数据页从Buffer Pool刷新到磁盘前，先将数据页写入双写缓冲区文件中，便于系统异常时恢复数据
#### Redo Log
重做日志，是用来实现事务的持久性。该日志文件由两部分组成：重做日志缓冲（redo log buffer）以及重做日志文件（redo log），前者是在内存中，后者在磁盘中。当事务提交之后会把所有修改信息都存到该日志中，用于在刷新脏页到磁盘中，发生错误时，进行数据恢复使用
以循环方式写入重做日志文件，设计两个文件ib_logfile0和ib_logfile1
### 后台线程
#### Master Thread
核心后台线程，负责调度其他线程，还负责将缓冲池中的数据异步刷新到磁盘中，保持数据的一致性，还包括脏页到刷新，合并插入缓存，undo页的回收
#### IO Thread
在InnoDB存储引擎中大量使用了AIO来处理IO请求，这样可以极大地提高数据库的性能，而IO Thread主要负责这些IO请求的回调

|         线程类型         | 默认个数 | 职责             |
| :------------------: | :--: | :------------- |
|     Read thread      |  4   | 负责读操作          |
|     Write thread     |  4   | 负责写操作          |
|      Log thread      |  1   | 负责将日志缓冲区刷新到磁盘  |
| Insert buffer thread |  1   | 负责将写缓冲区内容刷新到磁盘 |
#### Purge Thread
主要用于回收事务已经提交了的undo log，在事务提交之后，undo log可能不用了，就用它来回收
#### Page Cleaner Thread
协助Master Thread刷新脏页到磁盘的线程，它可以减轻Master Thread的工作压力，减少阻塞
## 事务原理
### 事务
事务是一组操作的集合，它是一个不可分割的工作单位，事务会把所有的操作作为一个整体一起向系统提交或撤销操作请求，即这些操作要么同时成功，要么同时失败
### 特性
- 原子性（Atomicity）：事务是不可分割的最小操作单元，要么全部成功，要么全部失败
- 一致性（Consistency）：事务完成时，必须使所有数据都保持一致状态
- 隔离性（Isolation）：数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行
- 持久性（Durability）：事务一旦提交或回滚，它对数据的改变就是永久的
原子性、一致性、持久性是由InnoDB存储引擎底层的两份日志来保障的，分别是redo log和undo log
隔离性是由锁和MVCC（多版本并发控制）来实现的
### redo log（持久性）
重做日志，记录的是事务提交时数据页的物理修改，是用来实现事务的持久性
该日志文件由两部分组成：重做日志缓冲（redo log buffer）以及重做日志文件（redo log file），前者是在内存中，后者是在磁盘中。当事务提交之后会把所有修改信息都存到该日志文件中，用于在刷新脏页到磁盘，发生错误时，进行数据恢复使用
### undo log（原子性）
回滚日志，用于记录数据被修改前的信息，作用包含两个：提供回滚和MVCC（多版本并发控制）
undo log和redo log记录物理日志不一样，它是逻辑日志。可以认为当delete一条记录时，undo log中会记录一条对应的insert记录，反之亦然，当update一条记录时，它记录一条对应相反的update记录。当执行rollback时，就可以从undo log中的逻辑记录读取到相应的内容并进行回滚
#### Undo log销毁
undo log在事务执行时产生，事务提交时，并不会立即删除undo log，因为这些日志可能还用于MVCC
#### Undo log存储
undo log采用段的方式进行管理和记录，存放在前面介绍的rollback segment回滚段中，内部包含1024个undo logsegment
## MVCC
### 基本概念
#### 当前读
读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。对于我们日常的操作，如：`select…lock in share mode（共享锁）`，`select…for update`、`update`、`insert`、`delete（排他锁）`都是一种当前读
#### 快照读
简单的select（不加锁）就是快照读。快照读，读取的是记录数据的可见版本，有可能是历史数据，不加锁，是非阻断读
- Read Committed：每次select，都生成一个快照读
- Repeatable Read：开启事务后第一个select语句才是快照读的地方
- Serialzable：快照读会退化为当前读
#### MVCC
全程Multi-Version Concurrency Control，多版本并发控制。指维护一个数据的多个版本，使得读写操作没有冲突，快照读为MySQL实现MVCC提供了一个非阻塞读功能。MVCC的具体实现，还需要依赖于数据库记录中的三个隐式字段、undo log日志、readView
### 实现原理
#### 记录中的隐藏字段
例：

| id  | age | name | ==DB_TRXID== | ==DB_ROLL_PTR== | ==DB_ROW_ID== |
| :-: | :-: | :--- | ------------ | --------------- | ------------- |
|  1  |  1  | tom  |              |                 |               |
|  3  |  3  | cat  |              |                 |               |

|    隐藏字段     | 含义                                     |
| :---------: | :------------------------------------- |
|  DB_TRX_ID  | 最近修改事务ID，旧料插入这条记录或最后一次修改该记录的           |
| DB_ROLL_PTR | 回滚指针，指向这条记录的上一个版本，用于配合undo log，指向上一个版本 |
|  DB_ROW_ID  | 隐藏主键，如果表结构没有指定主键，将会身材该隐藏字段             |
#### undo log
回滚日志，在`insert`、`update`、`delete`的时候产生的便于数据回滚的日志
当`insert`的时候，产生的undo log日志只在回滚时需要，在事务提交后，可被立即删除
而`update`、`delete`的时候，产生的undo log日志不仅在回滚时需要，在快照读时也需要，不会立即被删除
#### undo log版本链
不同事务或相同事务对同一条记录进行修改，会导致该记录的undolog生成一条记录版本链表，链表的头部是最新的旧记录，链表尾部是最早的旧记录
#### readview
ReadView（读视图）是快照读SQL执行时MVCC提取数据的依据，记录并维护系统当前活跃的事务（未提交的）id
ReadView中包含了四个核心字段：

|       字段       | 含义                               |
| :------------: | :------------------------------- |
|     m_ids      | 当前活跃的事务ID集合                      |
|   min_trx_id   | 最小活跃事务ID                         |
|   max_trx_id   | 预分配事务ID，当前最大事务ID + 1（因为事务ID是自增的） |
| creator_trx_id | ReadView创建者的事务ID                 |
trx_id：代表是当前事务ID
版本链数据访问规则：
1. `trx_id == create_trx_id?`可以访问该版本->成立，说明数据是当前这个事务更改的
2. `trx_id < min _trx_id?`可以访问该版本->成立，说明数据已经提交了
3. `trx_id > max_trx_id?`不可以访问该版本->成立，说明该事务是在ReadView生成后才开启
4. `min_trx_id <= trx_id <= max_trx_id?`如果trx_id不在m_ids中是可以访问该版本的->成立，说明数据已经提交
不同的隔离级别，生成ReadView的时机不同：
- Read Committed：在事务中每一次执行快照时生成ReadView
- Repeatable Read：仅在事务中第一次执行快照读时生成ReadView，后续复用该ReadView
# MySQL管理
## 系统数据库
MySQL数据库安装后，自带了以下四个数据库，具体作用如下：

|        数据库         | 含义                                                    |
| :----------------: | :---------------------------------------------------- |
|       mysql        | 存储MySQL服务器正常运行所需要的各种信息（时区、主从、用户、权限等）                  |
| information_schema | 提供了访问数据库元数据的各种表和视图，包含数据库、表、字段类型及访问权限等                 |
| performance_schema | 为MySQL服务器运行时提供了一个底层监控功能，主要用于收集数据库服务器性能参数              |
|        sys         | 包含了一系列方便DBA和开发人员利用performance_schema性能数据库进行性能调优和诊断的视图 |
## 常用工具
### mysql
该mysql不是指mysql服务，而是指mysql的客户端工具
#### 语法
```mysql
mysql [options] [database]
```
#### 选项
```mysql
#指定用户名
-u,--user=name
#指定密码
-p,--password[=name]
#指定服务器IP或域名
-h,--host=name
#指定链接端口
-p,--port=port
#执行SQL语句并退出
-e,--execute=name
```
-e选项可以在MySQL客户端执行SQL语句，而不用连接到MySQL数据库再执行，对于一些批处理脚本，这种方式尤其方便
```mysql
mysql -u root -p1234 -e"select * from stu";
```
### mysqladmin
mysqladmin是一个执行管理操作的客户端程序。可以用它来检查服务器的配置和当前状态，创建并删除数据库等
通过帮助文档查看选项：
```mysql
mysqladmin --help

```
示例：
```mysql
mysqladmin -u root -p123456 drop 'test01';
mysqladmin -u root -p123456 version;
```
### mysqlbinlog
由于服务器生成的二进制日志文件以二进制格式保存，所以如果想要检查这些文本的文本格式，就会使用到mysqlbinlog日志管理工具
#### 语法
```mysql
mysqlbinlog[options] log-files1 log-files2…
```
#### 选项
```mysql
#指定数据库名称，只列出指定的数据库相关操作
-d,--database=name
#忽略掉日志中的前n行命令
-o,--offset=#
#将输出的文本格式日志输出到指定文件
-r，--result-file=name
#显示简单格式，忽略掉一些信息
-s,--short-form
#指定日期间隔内的所有日志
--start-datatime=data1 --stop-datatime=data2
#指定位置间隔内的所有日志
--start-position=pos1 --stop-positon=pos2
```
### mysqlshow
mysql客户端对象查找工具，用来很快地查找存在哪些数据库、数据库中的表、表中的列或者索引
#### 语法
```mysql
mysqlshow[options][db_name[col_name]]
```
#### 选项
```mysql
#显示数据库及表的统计信息（数据库、表均可以不指定）
--count
#显示指定数据库或者指定表的状态信息
--i
```
#### 示例
```mysql
#查询每个数据库的表的数量及表中记录的数量
mysqlshow -u root -p2143 --count
#查询test库中每个表中的字段数，及行数
mysqlshow -u root -p2143 test --count
#查询test库中book表的详细情况
mysqlshow -u root -p2143 test book --count
```
### mysqldump
mysqldump客户端工具用来备份数据库或在不同数据库之间进行数据迁移。备份内容包含创建表，及插入表的SQL语句
#### 语法
```mysql
mysqldump[options]db_name[tables]
mysqldump[options]--database/-B db1[db2 db3…]
mysqldump[options]--all-databases/-A
```
#### 连接选项
```mysql
#指定用户名
-u，--user=name
#指定密码
-p,--password[=name]
#指定服务器ip或域名
-h,--host=name
#指定连接端口
-p,--port=#
```
#### 输出选项
```mysql
#在每个数据库创建语句前加上`drop database`语句
--add-drop-database
#在每个表创建语句前加上`drop table`语句，默认开启；不开启（--skip-add-drop-table)
--add-drop-table
#不包含数据库的创建语句
-n,--no-create-db
#不包含数据表的创建语句
-t,--no-create-info
#不包含数据
-d,--no-data
#自动生成两个文件：一个.sql文件，创建表结构的语句；一个.txt文件，数据文件
-T,--tab=name
```
### mysqlimport/source
mysqlimport是客户端数据导入工具，用来导入mysqldump加-T参数后导出的文本文件
#### 语法
```mysql
mysqlimport [options] db_name textfile1 [textfile2…]
```
#### 示例
```mysql
mysqlimport -u root -p2143 test/tmp/city.txt
```
#### 注意
如果需要导入sql文件，可以使用mysql中的source指令
```mysql
scource/root/xxxxx.sql
```


