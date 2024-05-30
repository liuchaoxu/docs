# 使用
## 字符集
### 1、默认字符集
MySQL 8版本之前，默认字符集为 latin1（ISO-8859-1） ，不支持中文，使用前必须设置字符集为utf8（utf8mb3）或utf8mb4。从MySQL 8开始，数据库的默认字符集为 utf8mb4 ，从而避免中文乱码的问题。
`**SHOW VARIABLES LIKE '%char%';**`
### 2、utf8与utf8mb4
utf8 字符集表示一个字符需要使用1～4个字节，但是我们常用的一些字符使用1～3个字节就可以表示了。而字符集表示一个字符所用的最大字节长度，在某些方面会影响系统的存储和性能，所以设计MySQL的设计者偷偷的定义了两个概念：
**utf8mb3 ：**阉割过的 utf8 字符集，只使用1～3个字节表示字符。（无法存储emoji表情）
MySQL5.7中的utf8是utf8mb3字符集
**utf8mb4 ：**正宗的 utf8 字符集，使用1～4个字节表示字符。
MySQL8.0中的utf8是utf8mb4字符集



 
##  SQL大小写规范
### 1、Windows和Linux的区别 
**Windows环境：**
全部不区分大小写
**Linux环境：**
1、数据库名、表名、表的别名、变量名严格区分大小写；
2、列名与列的别名不区分大小写。
3、关键字、函数名称不区分大小写；
### 2、Linux下大小写规则设置（了解）
在MySQL 8中设置的具体步骤为：
```
1、停止MySQL服务 
2、删除数据目录，即删除 /var/lib/mysql 目录 
3、在MySQL配置文件（/etc/my.cnf ）的 [mysqld] 中添加 lower_case_table_names=1 
4、初始化数据目录 mysqld --initialize --user=mysql
5、启动MySQL服务 systemctl start mysqld
```
注意：不建议在开发过程中修改此参数，将会丢失所有数据
## sql_mode
### 1、宽松模式 vs 严格模式
**宽松模式：**
执行错误的SQL或插入不规范的数据，也会被接受，并且不报错。
**严格模式：**
执行错误的SQL或插入不规范的数据，会报错。
MySQL5.7版本开始就将sql_mode默认值设置为了严格模式。
### 2、查看和设置sql_mode
**查询sql_mode的值：**
```
SELECT @@session.sql_mode; 
SELECT @@global.sql_mode; 
-- 或者 
SHOW VARIABLES LIKE 'sql_mode';
--session级别
```
![image-20220627032313938.png](https://cdn.nlark.com/yuque/0/2023/png/35187126/1687947582143-053017ce-6dd6-4fc0-a9eb-cb108333910e.png#averageHue=%23333130&clientId=u1f7496e5-e6a7-4&from=paste&height=113&id=udeb4453c&originHeight=153&originWidth=1004&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=9939&status=done&style=none&taskId=u14277cf8-c107-44c6-b8d5-81a053c0fbc&title=&width=741.3333740234375)
**临时设置sql_mode的值：**
```
SET GLOBAL sql_mode = 'mode1,model2,...';
--全局，要重新启动客户端生效，重启MySQL服务后失效
SET SESSION sql_mode = 'mode1,model2,...';
--当前会话生效效，关闭当前会话就不生效了。可以省略SESSION关键字
```
**在mysql配置文件中配置，永久生效：**在宿主机上执行以下命令，创建配置文件：
`vim /atguigu/mysql/mysql8/conf/my.cnf`
编辑配置文件
```
[mysqld]
sql-mode = "mode1,model2,..."
```
重启mysql容器
`docker restart atguigu-mysql8`
### 3、错误开发演示 
**建表并插入数据：**
```
CREATE DATABASE atguigudb;
USE atguigudb;
CREATE TABLE employee(id INT, `name` VARCHAR(16),age INT,dept INT);
INSERT INTO employee VALUES(1,'zhang3',33,101);
INSERT INTO employee VALUES(2,'li4',34,101);
INSERT INTO employee VALUES(3,'wang5',34,102);
INSERT INTO employee VALUES(4,'zhao6',34,102);
INSERT INTO employee VALUES(5,'tian7',36,102);
```
**需求：查询每个部门年龄最大的人**
```
-- 错误演示
SELECT `name`, dept, MAX(age) FROM employee GROUP BY dept;
```
以上查询语句在 “`**ONLY_FULL_GROUP_BY**`” 模式下查询出错，因为select子句中的name列并没有出现在group by子句中，也没有出现在函数中：
![image-20220627033533410.png](https://cdn.nlark.com/yuque/0/2023/png/35187126/1687947611478-6f128868-6e74-47d7-8a29-1d53b2cc4dd6.png#averageHue=%23464442&clientId=u1f7496e5-e6a7-4&from=paste&height=84&id=u4db71824&originHeight=112&originWidth=1004&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=12110&status=done&style=none&taskId=ua6656f12-fbda-4803-8cb6-e754af57ff9&title=&width=753.3333740234375)
在非 “`**ONLY_FULL_GROUP_BY**`” 模式下可以正常执行，
但是得到的是错误的结果：`SET SESSION sql_mode = ''; `
![image-20220627033754883.png](https://cdn.nlark.com/yuque/0/2023/png/35187126/1687947630749-67943265-57a2-4407-b321-175e483cc557.png#averageHue=%23232322&clientId=u1f7496e5-e6a7-4&from=paste&height=158&id=ub0435ae9&originHeight=210&originWidth=1004&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=13181&status=done&style=none&taskId=u69be3521-063c-4e83-9e38-d02fe38198e&title=&width=755.3333740234375)
**正确的查询方式：查询应该分两个步骤**
1、查询每个部门最大的年龄
2、查询人
正确的语句：
```
SELECT e.* 
FROM employee e
INNER JOIN (SELECT dept, MAX(age) age FROM employee GROUP BY dept) AS maxage 
ON e.dept = maxage.dept AND e.age = maxage.age;
```
**测试完成后再将sql_mode设置回来：**
`**SET SESSION sql_mode = 'ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';**`
### 4、sql_mode常用值（了解）

- `ONLY_FULL_GROUP_BY：`对于GROUP BY聚合操作，SELECT子句中只能包含函数和 GROUP BY 中出现的字段。
- STRICT_TRANS_TABLES： 
   - 对于支持事务的表，如果发现某个值缺失或非法，MySQL将抛出错误，语句会停止运行并回滚。
   - 对于不支持事务的表，不做限制，提高性能。
- NO_ZERO_IN_DATE：不允许`日期`和`月份`为零。
- NO_ZERO_DATE：MySQL数据库不允许插入零日期，插入零日期会抛出错误而不是警告。
- ERROR_FOR_DIVISION_BY_ZERO：在INSERT或UPDATE过程中，如果数据被零除，则产生错误而非警告。如果未给出该模式，那么数据被零除时MySQL返回NULL。
- NO_ENGINE_SUBSTITUTION：如果需要的存储引擎被禁用或不存在，那么抛出错误。不设置此值时，用默认的存储引擎替代。
# 逻辑架构
## 1、逻辑架构剖析
### 1.1、服务器处理客户端请求
![b7a8eb46-1168-434c-b63b-ec775affab7b.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/35187126/1687948147313-35325380-26c4-40fd-b9bf-cac5dd3f592e.jpeg#averageHue=%23e0e5e0&clientId=u13f8adc5-bd46-4&from=paste&height=484&id=u96d03f05&originHeight=476&originWidth=730&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=91178&status=done&style=stroke&taskId=u671ee6b2-c058-4c43-b1cd-9ae2ef8159d&title=&width=742.6666870117188)
### 1.2、Connectors（客户端）
MySQL服务器之外的客户端程序，与具体的语言相关，例如Java中的JDBC，图形用户界面SQLyog等。本质上都是在TCP连接上通过MySQL协议和MySQL服务器进行通信。
### 1.3、MySQL Server（服务器）
#### **第1层：连接层**

- 客户端访问 MySQL 服务器前，做的第一件事就是建立 TCP 连接。
- 经过三次握手建立连接成功后， MySQL 服务器对 TCP 传输过来的账号密码做身份认证、权限获取。
   - 用户名或密码不对，会收到一个Access denied for user错误，客户端程序结束执行
   - 用户名密码认证通过，会从权限表查出账号拥有的权限与连接关联，之后的权限判断逻辑，都将依赖于此时读到的权限
- TCP 连接收到请求后，必须要分配给一个线程专门与这个客户端的交互。所以还会有个线程池，去走后面的流程。每一个连接从线程池中获取线程，省去了创建和销毁线程的开销。
#### **第2层：服务层**
**Management Serveices & Utilities： 系统管理和控制工具**
**SQL Interface：SQL接口：**

- 接收用户的SQL命令，并且返回用户需要查询的结果。比如SELECT ... FROM就是调用SQL Interface 
- MySQL支持DML（数据操作语言）、DDL（数据定义语言）、存储过程、视图、触发器、自定义函数等多种SQL语言接口

**Parser：解析器：**
在SQL命令传递到解析器的时候会被解析器验证和解析。解析器中SQL 语句进行词法分析、语法分析、语义分析，并为其创建语法树。

- 词法分析：将整个语句拆分成一个个字段
- 语法分析：将词法分析拆分出的字段，按照MySQl语法规则，生成解析树
- 语义分析：检查解析树是否合法，比如查看表是否存在，列是否存在

典型的解析树如下：
![image-20220702002430362.png](https://cdn.nlark.com/yuque/0/2023/png/35187126/1687949445216-3da578b6-a504-4c97-862b-d08ae0cb6d0f.png#averageHue=%23f5f5f5&clientId=u13f8adc5-bd46-4&from=paste&height=309&id=u1072a335&originHeight=463&originWidth=909&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=63539&status=done&style=stroke&taskId=u59e66a50-b18d-446c-b433-7dd8214d5c4&title=&width=606)
**Optimizer：查询优化器：**

- SQL语句在语法解析后、查询前会使用查询优化器对查询进行优化，确定SQL语句的执行路径，生成一个执行计划。

**Caches & Buffers： 查询缓存组件：**

- MySQL内部维持着一些Cache和Buffer，比如Query Cache用来缓存一条SELECT语句的执行结果，如果能够在其中找到对应的查询结果，那么就不必再进行查询解析、查询优化和执行的整个过程了，直接将结果反馈给客户端。
- 这个缓存机制是由一系列小缓存组成的。比如表缓存，记录缓存，key缓存，权限缓存等 。
- 这个查询缓存可以在不同客户端之间共享 。 
- **问：大多数情况查询缓存就是个鸡肋，为什么呢？**
   - 只有相同的SQL语句才会命中查询缓存。两个查询请求在任何字符上的不同（例如：空格、注释、大小写），都会导致缓存不会命中。
   - 在两条查询之间 有 INSERT 、 UPDATE 、 DELETE 、 TRUNCATE TABLE 、 ALTER TABLE 、 DROP TABLE 或 DROP DATABASE 语句也会导致缓存失效
   - 因此 MySQL的查询缓存命中率不高。所以在MySQL 8之后就抛弃了这个功能。
#### **第3层：引擎层**
存储引擎层（ Storage Engines），负责MySQL中数据的存储和提取，对物理服务器级别维护的底层数据执行操作，服务器通过API与存储引擎进行通信。不同的存储引擎具有的功能不同，管理的表有不同的存储结构，采用的存取算法也不同，这样我们可以根据自己的实际需要进行选取。例如MyISAM引擎和InnoDB引擎。
### 1.4、存储层
所有的数据、数据库、表的定义、表的每一行的内容、索引，都是存在文件系统 上，以文件的方式存在，并完成与存储引擎的交互。
### 1.5、查询流程说明
![image-20220914161040788.png](https://cdn.nlark.com/yuque/0/2023/png/35187126/1687949463146-f4b72c13-7476-4d29-b7c3-6cf7888e4daa.png#averageHue=%23c5c5c2&clientId=u13f8adc5-bd46-4&from=paste&height=375&id=ud1df9cd1&originHeight=391&originWidth=655&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=165797&status=done&style=stroke&taskId=u1c3ec79c-e0dc-4fc6-9be4-43f2ebed816&title=&width=628.6666870117188)
**首先，**MySQL客户端通过协议与MySQL服务器建连接，通过SQL接口发送SQL语句，先检查查询缓存，如果命中，直接返回结果，否则进行语句解析。也就是说，在解析查询之前，服务器会先访问查询缓存，如果某个查询结果已经位于缓存中，服务器就不会再对查询进行解析、优化、以及执行。它仅仅将缓存中的结果返回给用户即可，这将大大提高系统的性能。
**接下来是解析过程，**MySQL解析器通过关键字将SQL语句进行解析，并生成一棵对应的解析树，解析器使用MySQL语法规则验证和解析SQL语句。例如，它将验证是否使用了错误的关键字，或者使用关键字的顺序是否正确，引号能否前后匹配等；预处理器则根据MySQL规则进一步检查解析树是否合法，例如，这里将检查数据表和数据列是否存在，还会解析名字和别名，看是否有歧义等，并生成一棵新解析树，新解析树可能和旧解析树结构一致。
**然后是优化过程，**MySQL优化程序会对我们的语句做一些优化，将查询的IO成本和CPU成本降到最低。优化的结果就是生成一个执行计划。这个执行计划表明了应该使用哪些索引执行查询，以及表之间的连接顺序是啥样，必要时将子查询转换为连接、表达式简化等等。我们可以使用EXPLAIN语句来查看某个语句的执行计划。
**最后，**进入执行阶段。完成查询优化后，查询执行引擎会按照生成的执行计划调用存储引擎提供的接口执行SQL查询并将结果返回给客户端。在MySQL8以下的版本，如果设置了查询缓存，这时会将查询结果进行缓存，再返回给客户端。
MySQL 8取消了查询缓存功能。在之前的MySQL版本中，查询缓存允许数据库将查询语句的结果缓存起来，以便在相同的查询再次执行时，可以直接返回缓存的结果，从而提高查询性能。然而，由于查询缓存在某些情况下会导致性能问题和不稳定性，MySQL在版本8中决定移除了这个功能。
以下是一些导致查询缓存被取消的原因：

1. **性能问题**：查询缓存的管理和维护需要消耗额外的资源，而且在高并发环境下，可能会导致锁争用和性能下降。
2. **内存开销**：查询缓存需要占用内存存储缓存的查询结果，如果缓存结果的开销过大，可能导致内存不足。
3. **失效问题**：当有表发生写操作（如INSERT、UPDATE、DELETE）时，与这个表相关的缓存会被自动失效，这可能导致缓存频繁失效，影响效率。
4. **不一致问题**：查询缓存会导致数据库返回缓存的旧数据，当数据发生变化时，可能会导致不一致性问题。
5. **复杂性和维护成本**：维护查询缓存的复杂性，包括缓存的失效、更新等操作，使得数据库引擎的开发和维护更加困难。
## **口述：**
连接通过tcp协议与连接池连接，sql接口发送对应的语句先检查缓存中是否存在，如果命中直接返回结果，如果不存在，进行解析器解析。根据关键字解析成对应的解析树，预处理器会检查列名别名等是否合法，然后生成新的解析树，可能与旧的解析树结构一样。然后进行sql优化器进行优化，会将sql的io成本和CPU成本降到最低，结果就是会得出一个执行计划。表之间的查询的顺序，使用的索引是什么，有时会把子查询转为连接查询等。
使用explain加查询语句可以查看执行计划
执行阶段会根据表对应的存储引擎进行查询如果是8一下版本那么如果开启了缓存还会将结果入缓存，再返回客户端
MySQL的查询流程可以概括为客户端-服务器模型，涉及到多个组件和步骤。以下是MySQL查询的典型流程：

1. **客户端发送查询请求**： 客户端应用程序（如Web应用、命令行工具）通过网络连接向MySQL服务器发送SQL查询请求。
2. **连接管理和身份验证**： MySQL服务器收到查询请求后，会处理连接管理和身份验证，验证客户端的身份，并确保有合法的权限执行查询。
3. **查询解析与优化**： 服务器会解析客户端发送的SQL查询，进行语法和语义分析，然后生成查询解析树。接下来，优化器会根据查询解析树生成最佳的执行计划，以提高查询性能。
4. **执行计划生成与执行**： 优化器生成的执行计划描述了如何执行查询。MySQL服务器根据执行计划从数据库中获取所需的数据，应用WHERE、GROUP BY、HAVING等操作，并根据执行计划的步骤执行查询。
5. **锁管理与事务处理**： 在执行查询的过程中，MySQL服务器会根据需要获取锁来保证事务的隔离性和数据的完整性。锁管理和事务处理确保多个并发事务不会相互干扰。
6. **结果返回给客户端**： 执行完查询后，MySQL服务器会将结果集或执行信息返回给客户端应用程序，通过网络连接传输到客户端。
7. **客户端处理结果**： 客户端应用程序接收到查询结果后，可以对结果进行处理，如显示、存储或进一步的数据处理。
8. **连接关闭**： 一旦查询执行完毕，客户端可以选择关闭与MySQL服务器的连接，释放资源。

需要注意的是，这只是MySQL查询的高级流程。实际执行可能会受到多种因素的影响，如查询复杂度、索引、并发连接数、数据库配置等。理解MySQL查询流程有助于优化查询性能、调试问题以及更好地设计数据库应用。
# ===以下内容仅做目录使用===
# SQL执行流程
MySQL语句的执行流程涉及多个阶段，从查询的提交到结果的返回。以下是MySQL语句的典型执行流程：

1. **查询解析**： 当用户提交一个SQL查询语句，MySQL首先会对查询语句进行解析，以识别语法错误和语义错误。在这个阶段，MySQL会检查查询语句是否遵循正确的SQL语法规则。
2. **查询优化**： 在查询解析完成后，MySQL会进行查询优化，目的是找到最有效的执行计划，以在性能和资源利用方面达到最佳平衡。优化器会考虑选择适当的索引、连接顺序、连接方法等。
3. **执行计划生成**： 在查询优化阶段，MySQL的优化器会生成一个执行计划，描述了执行查询所需的操作序列。执行计划包括了访问哪些表、使用哪些索引、执行哪些操作等细节。
4. **执行计划执行**： 一旦生成了执行计划，MySQL会根据执行计划开始执行查询。这涉及从磁盘读取数据、应用过滤条件、进行连接操作、执行聚合函数等。MySQL会根据需要获取合适的锁，以确保事务隔离和数据完整性。
5. **结果返回**： 在执行计划执行完毕后，MySQL将结果返回给客户端。结果可以包括查询结果行、更新、插入或删除的行数，以及任何错误信息。结果通过网络协议发送到客户端，供应用程序使用。

需要注意的是，这个流程是一个高级概述，实际执行过程可能会受到多种因素的影响，如查询复杂度、表结构、索引、数据库配置、并发访问等。此外，MySQL还具有缓存机制，如查询缓存和缓冲池，这些也会影响执行过程。
理解MySQL语句的执行流程对于优化查询性能、诊断性能问题以及设计数据库架构都非常重要。
## sql执行顺序
MySQL查询的执行顺序可以用以下步骤来描述：

1. **FROM 子句**： 在查询开始时，MySQL会根据FROM子句中指定的表获取所需的数据。
2. **WHERE 子句**： 如果查询包含WHERE子句，MySQL会应用WHERE条件，过滤掉不满足条件的行。
3. **GROUP BY 子句**： 如果查询包含GROUP BY子句，MySQL会按照指定的列对数据进行分组。
4. **HAVING 子句**： 如果查询包含HAVING子句，MySQL会在分组后应用HAVING条件，过滤掉不满足条件的分组。
5. **SELECT 子句**： 在SELECT子句中，MySQL会根据查询的列选择需要返回的数据。
6. **ORDER BY 子句**： 如果查询包含ORDER BY子句，MySQL会对结果集进行排序，按照指定的顺序排列数据。
7. **LIMIT 子句**： 如果查询包含LIMIT子句，MySQL会根据LIMIT限制返回的结果行数。

这些步骤描述了典型的SELECT查询的执行顺序。需要注意的是，MySQL的查询执行可能会受到索引、数据量、查询复杂度、数据库配置和并发等因素的影响。MySQL的查询优化器会尝试选择最佳的执行计划，以提高查询性能。
此外，如果查询涉及多个表的连接、子查询、联合查询等，执行顺序可能会更加复杂。了解查询执行的顺序有助于理解查询优化和性能调优的方向。
# 存储引擎
#  索引
## 1、索引简介
### 1.1、什么是索引**☆**
MySQL官方对索引的定义为：**索引（Index）是帮助MySQL高效获取数据的数据结构。索引的本质：**索引是数据结构。你可以简单理解为“排好序的快速查找数据结构”。这些数据结构以某种方式指向数据， 可以在这些数据结构的基础上实现高级查找算法 。
### 1.2、索引的优缺点**☆**
**优点：**
（1）提高数据检索的效率，降低数据库的IO成本
（2）保证表中每条记录的唯一性 。
**缺点：**
（1）创建索引和维护索引要耗费时间 。
（2）索引是存储在磁盘上的，因此需要占用磁盘空间 。 

### 1.3、索引分类**☆**

## 2、树
### 2.1、二叉树
**二叉树**
**二叉搜索树BST**
**BST的问题**
**平衡二叉树（AVL）**
**AVL的问题**
### 2.2、B树
## 3、MySQL的索引结构：B+tree
### 3.1、InnoDB中的索引
#### 3.1.1、设计索引
#### 3.1.2、InnoDB中的索引方案
### 3.2、B树和B+树对比**☆**
**B+树和B树的差异：**
**B+树为什么IO的次数会更少：**
### 3.3、聚簇索引**☆**
**特点：**
**优点：**
**缺点：**
**限制：**
### 3.4、非聚簇索引**☆**
**（二级索引、辅助索引）**
**这个B+树与聚簇索引有几处不同**
**概念：回表**
**问题：**
为什么我们还需要一次回表操作呢？直接把完整的用户记录放到叶子节点不OK吗？
**回答：**如果把完整的用户记录放到叶子节点是可以不用回表。但是太占地方了，相当于每建立一棵B+树都需要把所有的用户记录再都拷贝一遍，这就有点太浪费存储空间了。
**一张表可以有多个非聚簇索引：**
### 3.5、联合索引
### 3.6、覆盖索引**☆**
### 3.7、MyISAM中的索引**☆**
### 3.8、MyISAM与InnoDB对比**☆**
## 4、索引操作
### 4.1、创建索引

- 随表一起创建索引：
- 单独建创索引：
- 使用ALTER命令：
### 4.2、查看索引
SHOW INDEX FROM customer;
### 4.3、删除索引
```
DROP INDEX idx_name ON customer; -- 删除单值、唯一、复合索引

ALTER TABLE customer MODIFY id INT UNSIGNED, DROP PRIMARY KEY; -- 删除主键索引(有主键自增)
ALTER TABLE customer1 DROP PRIMARY KEY;  -- 删除主键索引(没有主键自增)
```
## 5、索引的使用场景**☆**
**哪些情况适合创建索引：**

- 频繁作为WHERE查询条件的字段
- 经常GROUP BY 和 ORDER BY的列
- 字段的值有唯一性的限制
- DISTINCT字段需要创建索引
- 多表JOIN时，对连接字段创建索引
- 使用字符串前缀创建索引
   - 例如一个字段 address varchar（120），我们可以创建索引的长度为（12）个字符，节省索引空间
- 区分度高的列（重复的数据少）适合作为索引
- 使用频繁的列，放到联合索引的左侧

**哪些情况不要创建索引：**

- WHERE、GROUP BY 、ORDER BY里用不到的字段不创建索引
- 表的数据记录太少
- 有大量重复数据的列上
- 避免对经常增删改的表创建索引
- 不要定义冗余或重复的索引
# 索引优化
## 1、数据库优化方案
**问题：**
哪些方法可以进行数据库调优？
**解决方案：**

- 索引失效，没有充分利用到索引：索引建立
- 关联查询太多JOIN（设计缺陷或不得已的需求）：SQL优化
- 数据过多：分库分表
- 服务器调优及各个参数设置（缓冲、线程数等）：调整my.cnf
## 2、性能分析（EXPLAIN）
### 2.1、EXPLAIN是什么
查看SQL执行计划：使用EXPLAIN关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理你的SQL语句的。分析你的查询语句或是表结构的性能瓶颈。
**用法：**
EXPLAIN + SQL语句
### 2.2、数据准备
### 2.3、各字段解释
#### 2.3.1、table

- **单表：**
- **多表：**
#### 2.3.2、id

- **id相同：**
- **id不同：**

**注意：**
**注意：**

- **id为NULL**

**小结：**
#### 2.3.3、select_type
查询的类型，主要是用于区别普通查询、联合查询、子查询等的复杂查询。

- **SIMPLE：**简单查询。查询中不包含子查询或者UNION。

EXPLAIN SELECT * FROM t1;

- **PRIMARY：**主查询。查询中若包含子查询，则最外层查询被标记为PRIMARY。
- **SUBQUERY：**子查询。在SELECT或WHERE列表中包含了子查询。

EXPLAIN SELECT * FROM t3 WHERE id = ( SELECT id FROM t2 WHERE content= 'a');

- **DEPENDENT SUBQUREY：**如果包含了子查询，并且查询语句不能被优化器转换为连接查询，并且子查询是相关子查询（子查询基于外部数据列），则子查询就是DEPENDENT SUBQUREY。

EXPLAIN SELECT * FROM t3 WHERE id = ( SELECT id FROM t2 WHERE content = t3.content);

- **UNCACHEABLE SUBQUREY：**表示这个subquery的查询要受到外部系统变量的影响
```
EXPLAIN SELECT * FROM t3 
WHERE id = ( SELECT id FROM t2 WHERE content = @@character_set_server);
```

- **UNION：**对于包含UNION或者UNION ALL的查询语句，除了最左边的查询是PRIMARY，其余的查询都是UNION。
- **UNION RESULT：**UNION会对查询结果进行查询去重，MYSQL会使用临时表来完成UNION查询的去重工作，针对这个临时表的查询就是"UNION RESULT"。
```
EXPLAIN 
SELECT * FROM t3 WHERE id = 1 
UNION  
SELECT * FROM t2 WHERE id = 1;
```

- **DEPENDENT UNION：**子查询中的UNION或者UNION ALL，除了最左边的查询是DEPENDENT SUBQUREY，其余的查询都是DEPENDENT UNION。
```
EXPLAIN SELECT * FROM t1 WHERE content IN
 (
 SELECT content FROM t2 
 UNION 
 SELECT content FROM t3
 );
```

- **DERIVED：**在包含派生表（子查询在from子句中）的查询中，MySQL会递归执行这些子查询，把结果放在临时表里。
```
EXPLAIN SELECT * FROM (
   SELECT content, COUNT(*) AS c FROM t1 GROUP BY content
) AS derived_t1 WHERE c > 1;
```
这里的<derived2>就是在id为2的查询中产生的派生表。
**补充：**MySQL在处理带有派生表的语句时，优先尝试把派生表和外层查询进行合并，如果不行，再把派生表物化掉，然后执行查询。下面的例子就是就是将派生表和外层查询进行合并的例子：
EXPLAIN SELECT * FROM (SELECT * FROM t1 WHERE content = 't1_832') AS derived_t1;
物化：执行子查询，将结果放入临时表的过程称为物化，默认情况下会建立基于内存的物化表，并建立哈希索引，如果子查询的结果非常大，超过了系统变量tmp_table_size的设置，会建立基于磁盘的物化表，并建立B+树索引。（MySQl5.7及之后）

- **MATERIALIZED：**优化器对于包含子查询的语句，如果选择将子查询物化后再与外层查询连接查询，该子查询的类型就是MATERIALIZED。如下的例子中，查询优化器先将子查询转换成物化表，然后将t1和物化表进行连接查询。

 EXPLAIN SELECT * FROM t1 WHERE content IN (SELECT content FROM t2);
#### 2.3.4、partitions
代表分区表中的命中情况，非分区表，该项为NULL
#### 2.3.5、type **☆**
**说明：**
结果值从最好到最坏依次是： 
system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL
比较重要的包含：system > const > eq_ref > ref > range > index > ALL
SQL 性能优化的目标：至少要达到 range 级别，要求是 ref 级别，最好是 const级别。（阿里巴巴开发手册要求）

- **ALL：**全表扫描。Full Table Scan，将遍历全表以找到匹配的行

EXPLAIN SELECT * FROM t1;

- **index：**全索引扫描。当使用覆盖索引，但需要扫描全部的索引记录时

EXPLAIN SELECT content1 FROM t4;EXPLAIN SELECT id FROM t1;

- **range：**只检索给定范围的行，使用一个索引来选择行。key 列显示使用了哪个索引，一般就是在你的where语句中出现了between、<、>、in等的查询。这种范围扫描索引扫描比全表扫描要好，因为它只需要开始于索引的某一点，而结束于另一点，不用扫描全部索引。

EXPLAIN SELECT * FROM t1 WHERE id > 2;

- **ref：**通过普通二级索引列与常量进行等值匹配时

EXPLAIN SELECT * FROM t4 WHERE content1 = 'a';

- **eq_ref：**连接查询时通过主键或不允许NULL值的唯一二级索引列进行等值匹配时

EXPLAIN SELECT * FROM t1, t2 WHERE t1.id = t2.id;

- **const：**根据主键或者唯一二级索引列与常数进行匹配时

EXPLAIN SELECT * FROM t1 WHERE id = 1;

- **system：**MyISAM引擎中，当表中只有一条记录时。（这是所有type的值中性能最高的场景）
```
CREATE TABLE t(i int) Engine=MyISAM;
INSERT INTO t VALUES(1);
EXPLAIN SELECT * FROM t;
```
#### 2.3.6、possible_keys 和 key **☆**

- possible_keys表示执行查询时可能用到的索引，一个或多个。 查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用。
- keys表示实际使用的索引。如果为NULL，则没有使用索引。

EXPLAIN SELECT id FROM t1 WHERE id = 1;
#### 2.3.7、key_len **☆**
表示索引使用的字节数，根据这个值可以判断索引的使用情况，检查是否充分利用了索引，针对联合索引值越大越好。
**如何计算：**

1. 先看索引上字段的类型+长度。比如：int=4 ; varchar(20) =20 ; char(20) =20 
2. 如果是varchar或者char这种字符串字段，视字符集要乘不同的值，比如utf8要乘 3(MySQL5.7)，如果是utf8mb4要乘4，GBK要乘2
3. varchar这种动态字符串要加2个字节
4. 允许为空的字段要加1个字节 
```
-- 创建索引
CREATE INDEX idx_age_name ON t_emp(age, `name`);
-- 测试1
EXPLAIN SELECT * FROM t_emp WHERE age = 30 AND `name` = 'ab%';
-- 测试2
EXPLAIN SELECT * FROM t_emp WHERE age = 30;
```
#### 2.3.8、ref
显示与key中的索引进行比较的列或常量。

- **const：** 与索引列进行等值比较的东西是啥，const表示一个常数

EXPLAIN SELECT * FROM t4 WHERE content1 = 'a';

- **ref=atguigudb.t1.id** 关联查询时出现，t2表和t1表的哪一列进行关联

EXPLAIN SELECT * FROM t1, t2 WHERE t1.id = t2.id;
#### 2.3.9、rows **☆**
MySQL认为它执行查询时实际从索引树中查找到的行数。值越小越好。
```
-- 如果是全表扫描，rows的值就是表中数据的估计行数
EXPLAIN SELECT * FROM t_emp WHERE empno = '100001';

-- 如果是使用索引查询，rows的值就是预计扫描索引记录行数
EXPLAIN SELECT * FROM t_emp WHERE deptId = 1;
```
#### 2.3.10、filtered
最后查询出来的数据占所有服务器端（server）检查行数（rows）的百分比。值越大越好。
例如上一个例子。
#### 2.3.11、Extra **☆**
包含不适合在其他列中显示但十分重要的额外信息。通过这些额外信息来理解MySQL到底将如何执行当前的查询语句。MySQL提供的额外信息有好几十个，这里只挑比较重要的介绍。

- **Impossible WHERE**：where子句的值总是false

EXPLAIN SELECT * FROM t_emp WHERE 1 != 1;

- **Using where：**使用了where，但在where上有字段没有创建索引。也可以理解为如果数据从引擎层被返回到server层进行过滤，那么就是Using where。

EXPLAIN SELECT * FROM t_emp WHERE `name` = '风清扬';

- **Using filesort：**

在对查询结果中的记录进行排序时，是可以使用索引的，如下所示：
EXPLAIN SELECT * FROM t1 ORDER BY id;
如果排序操作无法使用到索引，只能在内存中（记录较少时）或者磁盘中（记录较多时）进行排序（filesort），如下所示：
EXPLAIN SELECT * FROM t1 ORDER BY content;

- **Using index：**使用了覆盖索引，表示直接访问索引就足够获取到所需要的数据，不需要通过索引回表

 EXPLAIN SELECT id, content1 FROM t4;

- **Using index condition：**叫作 Index Condition Pushdown Optimization （索引下推优化）
   - 如果没有索引下推（ICP），那么MySQL在存储引擎层找到满足content1 > 'z'条件的第一条二级索引记录。主键值进行回表，返回完整的记录给server层，server层再判断其他的搜索条件是否成立。如果成立则保留该记录，否则跳过该记录，然后向存储引擎层要下一条记录。
   - 如果使用了索引下推（ICP），那么MySQL在存储引擎层找到满足content1 > 'z'条件的第一条二级索引记录。不着急执行回表，而是在这条记录上先判断一下所有关于idx_content1索引中包含的条件是否成立，也就是content1 > 'z' AND content1 LIKE '%a'是否成立。如果这些条件不成立，则直接跳过该二级索引记录，去找下一条二级索引记录；如果这些条件成立，则执行回表操作，返回完整的记录给server层。
```
-- content1列上有索引idx_content1
EXPLAIN SELECT * FROM t4 WHERE content1 > 'z' AND content1 LIKE '%a';
```
**注意：**如果这里的查询条件只有content1 > 'z'，那么找到满足条件的索引后也会进行一次索引下推的操作，判断content1 > 'z'是否成立（这是源码中为了编程方便做的冗余判断）

- **Using join buffer：**在连接查询时，当被驱动表（t2）不能有效的利用索引时，MySQL会提前申请一块内存空间（join buffer）存储驱动表的数据，来加快查询速度

EXPLAIN  SELECT * FROM t1, t2 WHERE t1.content = t2.content;
下面这个例子就是被驱动表使用了索引，此时Extra中就没有Using join buffer了：
EXPLAIN SELECT * FROM t_emp, t_dept WHERE t_dept.id = t_emp.deptId;
课外阅读：在没有索引的情况下，为了优化多表连接，减少磁盘IO读取次数和数据遍历次数，MySQL为我们提供了很多不同的连接缓存的优化算法，可参考[https://blog.csdn.net/qq_35423190/article/details/120504960](https://blog.csdn.net/qq_35423190/article/details/120504960)

- Using join buffer (hash join)**8.0新增：**连接缓存（hash连接） 速度更快
- Using join buffer (Block Nested Loop)**从5.7开始**：连接缓存（块嵌套循环）
## 3、准备数据
在做优化之前，要准备大量数据。接下来创建两张表，并往员工表里插入50W数据，部门表中插入1W条数据。
怎么快速插入50w条数据呢？ 存储过程
怎么保证插入的数据不重复？函数
**部门表：**

- id：自增长
- deptName：随机字符串，允许重复
- address：随机字符串，允许重复
- CEO：1-50w之间的任意数字

**员工表：**

- id：自增长
- empno：可以使用随机数字，或者从1开始的自增数字，不允许重复
- name：随机生成，允许姓名重复
- age：区间随机数
- deptId：1-1w之间随机数

**总结：**需要产生随机字符串和区间随机数的函数。
### 3.1、创建表
```
CREATE TABLE `dept` (
    `id` INT(11) NOT NULL AUTO_INCREMENT,
    `deptName` VARCHAR(30) DEFAULT NULL,
    `address` VARCHAR(40) DEFAULT NULL,
    ceo INT NULL ,
    PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=1;

CREATE TABLE `emp` (
    `id` INT(11) NOT NULL AUTO_INCREMENT,
    `empno` INT NOT NULL ,
    `name` VARCHAR(20) DEFAULT NULL,
    `age` INT(3) DEFAULT NULL,
    `deptId` INT(11) DEFAULT NULL,
    PRIMARY KEY (`id`)
    #CONSTRAINT `fk_dept_id` FOREIGN KEY (`deptId`) REFERENCES `t_dept` (`id`)
) ENGINE=INNODB AUTO_INCREMENT=1;
```
### 3.2、创建函数
```
-- 查看mysql是否允许创建函数：
SHOW VARIABLES LIKE 'log_bin_trust_function_creators';
-- 命令开启：允许创建函数设置：（global-所有session都生效）
SET GLOBAL log_bin_trust_function_creators=1;
```
```
-- 随机产生字符串
DELIMITER $$
CREATE FUNCTION rand_string(n INT) RETURNS VARCHAR(255)
BEGIN    
    DECLARE chars_str VARCHAR(100) DEFAULT 'abcdefghijklmnopqrstuvwxyzABCDEFJHIJKLMNOPQRSTUVWXYZ';
    DECLARE return_str VARCHAR(255) DEFAULT '';
    DECLARE i INT DEFAULT 0;
    WHILE i < n DO  
        SET return_str =CONCAT(return_str,SUBSTRING(chars_str,FLOOR(1+RAND()*52),1));  
        SET i = i + 1;
    END WHILE;
    RETURN return_str;
END $$

-- 假如要删除
-- drop function rand_string;
```
```
-- 用于随机产生区间数字
DELIMITER $$
CREATE FUNCTION rand_num (from_num INT ,to_num INT) RETURNS INT(11)
BEGIN   
 DECLARE i INT DEFAULT 0;  
 SET i = FLOOR(from_num +RAND()*(to_num -from_num+1));
RETURN i;  
END$$

-- 假如要删除
-- drop function rand_num;
```
### 3.3、创建存储过程
```
-- 插入员工数据
DELIMITER $$
CREATE PROCEDURE  insert_emp(START INT, max_num INT)
BEGIN  
    DECLARE i INT DEFAULT 0;   
    #set autocommit =0 把autocommit设置成0  
    SET autocommit = 0;    
    REPEAT  
        SET i = i + 1;  
        INSERT INTO emp (empno, NAME, age, deptid ) VALUES ((START+i) ,rand_string(6), rand_num(30,50), rand_num(1,10000));  
        UNTIL i = max_num  
    END REPEAT;  
    COMMIT;  
END$$
 
-- 删除
-- DELIMITER ;
-- drop PROCEDURE insert_emp;
```
```
-- 插入部门数据
DELIMITER $$
CREATE PROCEDURE insert_dept(max_num INT)
BEGIN  
    DECLARE i INT DEFAULT 0;   
    SET autocommit = 0;    
    REPEAT  
        SET i = i + 1;  
        INSERT INTO dept ( deptname,address,ceo ) VALUES (rand_string(8),rand_string(10),rand_num(1,500000));  
        UNTIL i = max_num  
    END REPEAT;  
    COMMIT;  
END$$
 
-- 删除
-- DELIMITER ;
-- drop PROCEDURE insert_dept;
```
### 3.4、调用存储过程
```
-- 执行存储过程，往dept表添加1万条数据
CALL insert_dept(10000); 

-- 执行存储过程，往emp表添加50万条数据，编号从100000开始
CALL insert_emp(100000,500000);
```
### 3.5、批量删除表索引
```
-- 批量删除某个表上的所有索引
DELIMITER $$
CREATE PROCEDURE `proc_drop_index`(dbname VARCHAR(200),tablename VARCHAR(200))
BEGIN
    DECLARE done INT DEFAULT 0;
    DECLARE ct INT DEFAULT 0;
    DECLARE _index VARCHAR(200) DEFAULT '';
    DECLARE _cur CURSOR FOR SELECT index_name FROM information_schema.STATISTICS WHERE table_schema=dbname AND table_name=tablename AND seq_in_index=1 AND index_name <>'PRIMARY'  ;
    DECLARE CONTINUE HANDLER FOR NOT FOUND set done=2 ;      
    OPEN _cur;
        FETCH _cur INTO _index;
        WHILE  _index<>'' DO 
            SET @str = CONCAT("drop index ",_index," on ",tablename ); 
            PREPARE sql_str FROM @str ;
            EXECUTE sql_str;
            DEALLOCATE PREPARE sql_str;
            SET _index=''; 
            FETCH _cur INTO _index; 
        END WHILE;
    CLOSE _cur;
END$$
```
```
-- 执行批量删除：dbname 库名称, tablename 表名称
CALL proc_drop_index("dbname","tablename");
```
### 3.6、开启SQL执行时间的显示
为了方便后面的测试中随时查看SQL运行的时间，测试索引优化后的效果，我们开启profiling
```
-- 显示sql语句执行时间
SET profiling = 1;
SHOW VARIABLES  LIKE '%profiling%';
SHOW PROFILES;
```
## 4、单表索引失效案例
MySQL中提高性能的一个最有效的方式是对数据表设计合理的索引。索引提供了高效访问数据的方法，并且加快查询的速度，因此索引对查询的速度有着至关重要的影响。
**我们创建索引后，用不用索引，最终是优化器说了算。优化器会基于开销选择索引，怎么开销小就怎么来。不是基于规则，也不是基于语义。**
**另外SQL语句是否使用索引，和数据库的版本、数据量、数据选择度（查询中选择的列数）运行环境都有关系。**
```
-- 创建索引
CREATE INDEX idx_name ON emp(`name`);
```
### 4.1、计算、函数导致索引失效
```
-- 显示查询分析
EXPLAIN SELECT * FROM emp WHERE emp.name  LIKE 'abc%';
EXPLAIN SELECT * FROM emp WHERE LEFT(emp.name,3) = 'abc'; --索引失效
```
### 4.2、LIKE以%开头索引失效
EXPLAIN SELECT * FROM emp WHERE name LIKE '%ab%'; --索引失效
**拓展：Alibaba《Java开发手册》**
【强制】页面搜索严禁左模糊或者全模糊，如果需要请走搜索引擎来解决。
### 4.3、不等于(!= 或者<>)索引失效
```
EXPLAIN SELECT * FROM emp WHERE emp.name = 'abc' ;
EXPLAIN SELECT * FROM emp WHERE emp.name <> 'abc' ; --索引失效
```
### 4.4、IS NOT NULL 和 IS NULL
```
EXPLAIN SELECT * FROM emp WHERE emp.name IS NULL;
EXPLAIN SELECT * FROM emp WHERE emp.name IS NOT NULL; --索引失效
```
**注意：**当数据库中的数据的索引列的NULL值达到比较高的比例的时候，即使在IS NOT NULL 的情况下 MySQL的查询优化器会选择使用索引，此时type的值是range（范围查询）
```
-- 将 id>20000 的数据的 name 值改为 NULL
UPDATE emp SET `name` = NULL WHERE `id` > 20000;

-- 执行查询分析，可以发现 IS NOT NULL 使用了索引
-- 具体多少条记录的值为NULL可以使索引在IS NOT NULL的情况下生效，由查询优化器的算法决定
EXPLAIN SELECT * FROM emp WHERE emp.name IS NOT NULL;
```
**测试完将name的值改回来**
UPDATE emp SET `name` = rand_string(6) WHERE `id` > 20000;
### 4.5、类型转换导致索引失效
```
EXPLAIN SELECT * FROM emp WHERE name='123'; 
EXPLAIN SELECT * FROM emp WHERE name= 123; --索引失效
```
### 4.6、全值匹配我最爱
**准备：**
```
-- 首先删除之前创建的索引
CALL proc_drop_index("atguigudb","emp");
```
**问题：**为以下查询语句创建哪种索引效率最高
```
-- 查询分析
EXPLAIN SELECT * FROM emp WHERE emp.age = 30 and deptid = 4 AND emp.name = 'abcd';
-- 执行SQL
SELECT * FROM emp WHERE emp.age = 30 and deptid = 4 AND emp.name = 'abcd';
-- 查看执行时间
SHOW PROFILES;
```
**创建索引并重新执行以上测试：**
```
-- 创建索引：分别创建以下三种索引的一种，并分别进行以上查询分析
CREATE INDEX idx_age ON emp(age);
CREATE INDEX idx_age_deptid ON emp(age,deptid);
CREATE INDEX idx_age_deptid_name ON emp(age,deptid,`name`);
```
**结论：**可以发现最高效的查询应用了联合索引 idx_age_deptid_name
### 4.7、最佳左前缀法则
**准备：**
```
-- 首先删除之前创建的索引
CALL proc_drop_index("atguigudb","emp");
-- 创建索引
CREATE INDEX idx_age_deptid_name ON emp(age,deptid,`name`);
```
**问题：**以下这些SQL语句能否命中 idx_age_deptid_name 索引，可以匹配多少个索引字段
**测试：**

- 如果索引了多列，要遵守最左前缀法则。即查询从索引的最左前列开始并且不跳过索引中的列。
- 过滤条件要使用索引，必须按照索引建立时的顺序，依次满足，一旦跳过某个字段，索引后面的字段都无法被使用。
```
EXPLAIN SELECT * FROM emp WHERE emp.age=30 AND emp.name = 'abcd' ;
-- EXPLAIN结果：
-- key_len：5 只使用了age索引
-- 索引查找的顺序为 age、deptid、name，查询条件中不包含deptid，无法使用deptid和name索引

EXPLAIN SELECT * FROM emp WHERE emp.deptid=1 AND emp.name = 'abcd';
-- EXPLAIN结果：
-- type： ALL， 执行了全表扫描
-- key_len： NULL， 索引失效
-- 索引查找的顺序为 age、deptid、name，查询条件中不包含age，无法使用整个索引

EXPLAIN SELECT * FROM emp WHERE emp.age = 30 AND emp.deptid=1 AND emp.name = 'abcd';
-- EXPLAIN结果：
-- 索引查找的顺序为 age、deptid、name，匹配所有索引字段

EXPLAIN SELECT * FROM emp WHERE emp.deptid=1 AND emp.name = 'abcd' AND emp.age = 30;
-- EXPLAIN结果：
-- 索引查找的顺序为 age、deptid、name，匹配所有索引字段
```
### 4.8、索引中范围条件右边的列失效 
**准备：**
```
-- 首先删除之前创建的索引
CALL proc_drop_index("atguigudb","emp");
```
**问题：**为以下查询语句创建哪种索引效率最高
EXPLAIN SELECT * FROM emp WHERE emp.age=30 AND emp.deptId>1000 AND emp.name = 'abc'; 
**测试1：**
```
-- 创建索引并执行以上SQL语句的EXPLAIN
CREATE INDEX idx_age_deptid_name ON emp(age,deptid,`name`);
-- key_len：10， 只是用了 age 和 deptid索引，name失效
```
**注意：**当我们修改deptId的范围条件的时候，例如deptId>100，那么整个索引失效，MySQL的优化器基于成本计算后认为没必要使用索引了，所以就进行了全表扫描。（注意：因为表中的数据是随机生成的，因此实际测试中根据具体数据的不同测试的结果也会不一样，最终是否使用索引由优化器决定）
**测试2：**
```
-- 创建索引并执行以上SQL语句的EXPLAIN（将deptid索引的放在最后）
CREATE INDEX idx_age_name_deptid ON emp(age,`name`,deptid);
-- 使用了完整的索引
```
**补充：**以上两个索引都存在的时候，MySQL优化器会自动选择最好的方案
## 5、关联查询优化
### 5.1、数据准备
创建两张表，并分插入16条和20条数据：
```
-- 分类
CREATE TABLE IF NOT EXISTS `class` (
`id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
`card` INT(10) UNSIGNED NOT NULL,
PRIMARY KEY (`id`)
);
-- 图书
CREATE TABLE IF NOT EXISTS `book` (
`bookid` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
`card` INT(10) UNSIGNED NOT NULL,
PRIMARY KEY (`bookid`)
);
 
-- 插入16条记录
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO class(card) VALUES(FLOOR(1 + (RAND() * 20)));
 
-- 插入20条记录
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
INSERT INTO book(card) VALUES(FLOOR(1 + (RAND() * 20)));
```
### 5.2、左外连接
**没有创建索引前的测试：**进行了全表扫描，查询次数为16*20
```
EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card;
-- 左表class：驱动表、右表book：被驱动表
```
**测试1：**在驱动表上创建索引：进行了全索引扫描，查询次数是16*20
```
-- 创建索引
CREATE INDEX idx_class_card ON class(card);
```
**测试2：**在被驱动表上创建索引：可以避免全表扫描，查询次数是16*1
```
-- 首先删除之前创建的索引
CALL proc_drop_index("atguigudb","class");
-- 创建索引
CREATE INDEX idx_book_card ON book(card);
```
**测试3：**同时给两张表添加索引：充分利用了索引，查询次数是16*1
```
-- 已经有了book索引
CREATE INDEX idx_class_card ON class(card);
```
**结论：**
针对两张表的连接条件涉及的列，索引要创建在被驱动表上，驱动表尽量是小表

- 如果驱动表上没有where过滤条件
   - 当驱动表的连接条件没有索引时，驱动表是全表扫描
   - 当针对驱动表的连接条件建立索引时，驱动表依然要进行全索引扫描
   - 因此，此时建立在驱动表上的连接条件上的索引是没有太大意义的
- 如果驱动表上有where过滤条件，那么针对过滤条件创建的索引是有必要的
### 5.3、内连接
**测试：**将前面外连接中的LEFT JOIN 变成 INNER JOIN
```
-- 换成inner join
EXPLAIN SELECT * FROM class INNER JOIN book ON class.card=book.card;
-- 交换class和book的位置
EXPLAIN SELECT * FROM book INNER JOIN class ON class.card=book.card;
```
**都有索引的情况下：**查询优化器自动选择数据量小的表做为驱动表
**class表有索引的情况下：**book表是驱动表
**book表有索引的情况下：**class表是驱动表
**都没有索引的情况下：**选择数据量小的表做为驱动表
**结论：**发现即使交换表的位置，MySQL优化器也会自动选择驱动表，自动选择驱动表的原则是：索引创建在被驱动表上，驱动表是小表。
### 5.4、扩展掌门人的练习
```
-- 首先删除之前创建的索引
CALL proc_drop_index("atguigudb","emp");
CALL proc_drop_index("atguigudb","dept");
```
**1.三表左连接方式:**
```
-- 员工表(t_emp)、部门表(t_dept)、ceo(t_emp)表 关联查询
EXPLAIN SELECT emp.name, ceo.name AS ceoname 
FROM emp
LEFT JOIN dept ON emp.deptid = dept.id 
LEFT JOIN emp ceo ON dept.ceo = ceo.id;
```
一趟查询，用到了主键索引，效果最佳
**2.子查询方式：**
```
EXPLAIN SELECT emp.name,
(SELECT emp.name FROM emp WHERE emp.id = dept.ceo) AS ceoname
FROM emp 
LEFT JOIN dept ON emp.deptid = dept.id;
```
两趟查询，用到了主键索引，跟第一种比，效果稍微差点。
**3.临时表连接方式**
```
EXPLAIN SELECT emp_with_ceo_id.name, emp.name AS ceoname 
FROM 
( 
SELECT emp.name, dept.ceo 
FROM emp 
LEFT JOIN dept ON emp.deptid = dept.id 
) emp_with_ceo_id
LEFT JOIN emp ON emp_with_ceo_id.ceo = emp.id;
```
查询一趟，MySQL查询优化器将衍生表查询转换成了连接表查询，速度堪比第一种方式
**MySQL5.5查询结果：**两趟查询，先查询a,b产生衍生表ab,衍生表作为驱动表，c作为被驱动表，使用到c表主键。效果比后面一种要好一点。
**4、临时表连接方式2**
```
EXPLAIN SELECT emp.name, ceo.ceoname FROM emp LEFT JOIN
( 
SELECT emp.deptId AS deptId, emp.name AS ceoname 
FROM emp 
INNER JOIN dept ON emp.id = dept.ceo 
) ceo
ON emp.deptId = ceo.deptId;
```
查询一趟，MySQL查询优化器将衍生表查询转换成了连接表查询，但是只有一个表使用了索引，数据检索的次数稍多，性能最差。
**MySQL5.5查询结果：**两趟查询，先查询b, a产生衍生表ab,衍生表作为被驱动表，衍生表无法建立索引，也就无法优化; 所以，这种语句是性能最差的。
### 5.5、总结

- 保证被驱动表的JOIN字段已经创建了索引
- 需要JOIN 的字段，数据类型保持绝对一致。
- LEFT JOIN 时，选择小表作为驱动表，大表作为被驱动表 。减少外层循环的次数。
- INNER JOIN 时，MySQL会自动将小结果集的表选为驱动表 。选择相信MySQL优化策略。
- 能够直接多表关联的尽量直接关联，不用子查询。(减少查询的趟数)
- 衍生表建不了索引（MySQL5.5）
## 6、子查询优化
**查询非掌门人**
```
-- 不推荐
-- 查询员工，这些员工的id没在（掌门人id列表中）
-- 【查询不是CEO的员工】
SELECT * FROM t_emp emp WHERE emp.id NOT IN 
(SELECT dept.ceo FROM t_dept dept WHERE dept.ceo IS NOT NULL);
```
**注意：**使用大表（emp、dept表）测试更加直观
```
-- 推荐
-- 按照集合查询
SELECT emp.* FROM t_emp emp 
LEFT JOIN t_dept dept ON emp.id = dept.ceo WHERE dept.id IS NULL;
```
也可以为ceo添加一个索引字段
**结论：**尽量不要使用NOT IN 或者 NOT EXISTS，用LEFT JOIN xxx ON xx = xx WHERE xx IS NULL替代
## 7、排序优化
### 7.1、索引失效的情况
**以下三种情况不走索引：**

1. 无过滤，不索引
2. 顺序错，不索引
3. 方向反，不索引

**准备：**
```
-- 删除现有索引
CALL proc_drop_index("atguigudb","emp");
-- 创建索引
CREATE INDEX idx_age_deptid_name ON emp (age,deptid,`name`);
```
**无过滤，不索引：**
```
-- 没有使用索引：
EXPLAIN SELECT * FROM emp ORDER BY age,deptid;

-- 使用了索引：order by想使用索引，必须有过滤条件，索引才能生效，limit也可以看作是过滤条件
EXPLAIN SELECT * FROM emp ORDER BY age,deptid LIMIT 10;
```
**顺序错，不索引：**
```
-- 排序使用了索引：
-- 注意：key_len = 5是where语句使用age索引的标记，order by语句使用索引不在key_len中体现。
--      order by语句如果没有使用索引，在extra中会出现using filesort。                  
EXPLAIN SELECT * FROM emp WHERE age=45 ORDER BY deptid;

-- 排序使用了索引：
EXPLAIN SELECT * FROM emp WHERE age=45 ORDER BY deptid, `name`; 

-- 排序没有使用索引：因为索引列中不存在empno
EXPLAIN SELECT * FROM emp WHERE age=45 ORDER BY deptid, empno;

-- 排序没有使用索引：order by 后的排序条件的顺序，与索引顺序不一致
EXPLAIN SELECT * FROM emp WHERE age=45 ORDER BY `name`, deptid;

-- 排序没有使用索引：出现的顺序要和复合索引中的列的顺序一致！
EXPLAIN SELECT * FROM emp WHERE deptid=45 ORDER BY age;
```
**方向反，不索引：**
```
-- 排序使用了索引：排序条件和索引一致，并方向相同，可以使用索引
EXPLAIN SELECT * FROM emp WHERE age=45 ORDER BY deptid DESC, `name` DESC;

-- 没有使用索引：两个排序条件方向相反
EXPLAIN SELECT * FROM emp WHERE age=45 ORDER BY deptid ASC, `name` DESC;
```
### 7.2、索引优化案例
排序优化的目的是，去掉 Extra 中的 using filesort（手工排序）
**准备：**
```
-- 删除现有索引
CALL proc_drop_index("atguigudb","emp");
-- 这个例子结合 show profiles; 查看运行时间
SET profiling = 1;
```
**需求：**查询 年龄为30岁的，且员工编号小于101000的用户，按用户名称排序 
**测试1：**很显然，type 是 ALL，即最坏的情况。Extra 里还出现了 Using filesort,也是最坏的情况。优化是必须的。
```
EXPLAIN SELECT * FROM emp WHERE age =30 AND empno <101000 ORDER BY `name`;
-- 然后查看一下SQL的执行时间
```
性能测试：
**优化思路：** 尽量让where的过滤条件和排序使用上索引
**step1：** 我们建一个三个字段的组合索引可否？ 
CREATE INDEX idx_age_empno_name ON emp (age,empno,`name`);
最后的name索引没有用到，出现了Using filesort。原因是，因为empno是一个范围过滤，所以索引后面的字段不会再使用索引了。所以我们建一个3值索引是没有意义的 
性能测试：
**step2：**那么我们先删掉这个索引：
DROP INDEX idx_age_empno_name ON emp;
为了去掉filesort我们可以把索引建成，也就是说empno 和name这个两个字段我只能二选其一。
CREATE INDEX idx_age_name ON emp(age,`name`);
这样我们优化掉了 using filesort。但是经过测试，性能反而下降
性能测试：
**step3：**如果我们选择那个范围过滤，而放弃排序上的索引呢?
```
DROP INDEX idx_age_name ON emp;
CREATE INDEX idx_age_empno ON emp(age,empno);
```
执行原始的sql语句，查看性能，结果竟然有 filesort的 sql 运行速度，超过了已经优化掉 filesort的 sql，而且快了好多倍。何故？
性能测试：
**原因：**所有的排序都是在条件过滤之后才执行的，所以，如果条件过滤掉大部分数据的话，剩下几百几千条数据进行排序其实并不是很消耗性能，即使索引优化了排序，但实际提升性能很有限。 相对的 empno<101000 这个条件，如果没有用到索引的话，要对几万条的数据进行扫描，这是非常消耗性能的，所以索引放在这个字段上性价比最高，是最优选择。
**结论：**当【范围条件】和【group by 或者 order by】的字段出现二选一时，优先观察条件字段的过滤数量，如果过滤的数据足够多，而需要排序的数据并不多时，优先把索引放在范围字段上。反之，亦然。
也可以将选择权交给MySQL：索引同时存在，mysql自动选择最优的方案：（对于这个例子，mysql选择idx_age_empno），但是，随着数据量的变化，选择的索引也会随之变化的。
### 7.3、双路排序和单路排序
如果排序没有使用索引，引起了filesort（手工排序），那么filesort有两种算法

- 双路排序
- 单路排序

**双路排序（慢）**
MySQL 4.1之前是使用双路排序，字面意思就是两次扫描磁盘，最终得到数据。

- 首先，根据行指针从磁盘取排序字段，在buffer进行排序。
- 再按照排序字段的顺序从磁盘取其他字段。

取一批数据，要对磁盘进行两次扫描。众所周知，IO是很耗时的，所以在mysql4.1之后，出现了第二种改进的算法，就是单路排序。
**单路排序（快）**

- 从磁盘读取查询需要的所有字段，按照order by列在buffer对它们进行排序。
- 然后扫描排序后的列表进行输出。

它的效率更快一些，因为只读取一次磁盘，避免了第二次读取数据。并且把随机IO变成了顺序IO。但是它会使用更多的空间， 因为它把每一行都保存在内存中了。
**结论及引申出的问题**

- 单路比多路要多占用更多内存空间
- 因为单路是把所有字段都取出，所以有可能取出的数据的总大小超出了sort_buffer_size的容量，导致每次只能取sort_buffer_size容量大小的数据，进行排序（创建tmp文件，多路合并），排完再取sort_buffer容量大小，再排……从而多次I/O。
- 单路本来想省一次I/O操作，反而导致了大量的I/O操作，反而得不偿失。

**优化策略**

- 减少select 后面的查询的字段：Order by时select * 是一个大忌。查询字段过多会占用sort_buffer_size的容量。
- 增大sort_buffer_size参数的设置：当然，要根据系统的能力去提高，因为这个参数是针对每个进程（connection）的 1M-8M之间调整。 MySQL8.0，InnoDB存储引擎默认值是1048576字节，1MB。

SHOW VARIABLES LIKE '%sort_buffer_size%'; --默认1MB

- 增大max_length_for_sort_data参数的设置：MySQL根据max_length_for_sort_data变量来确定使用哪种算法，默认值是4096字节，如果需要返回的列的总长度大于max_length_for_sort_data，使用双路排序算法，否则使用单路排序算法。但是如果设的太高，数据总容量超出sort_buffer_size的概率就增大，明显症状是高的磁盘I/O活动和低的处理器使用率。1024-8192之间调整。

SHOW VARIABLES LIKE '%max_length_for_sort_data%'; --默认4K
**举例：**
1、如果数据总量很小（单路一次就可以读取所有数据），单条记录大小很大（大于4K，默认会使用双路排序），此时，可以增加max_length_for_sort_data的值，增加sort_buffer_size的值，让服务器默认使用单路排序。
2、如果数据总量很大（单路很多次IO才可以），单条记录大小很小（小于4K，默认会使用单路排序），此时，可以减小max_length_for_sort_data的值，让服务器默认使用双路排序。
## 8、分组优化

- group by 使用索引的原则几乎跟order by一致。但是group by 即使没有过滤条件用到索引，也可以直接使用索引（Order By 必须有过滤条件才能使用上索引）
- 包含了order by、group by、distinct这些查询的语句，where条件过滤出来的结果集请保持在1000行以内，否则SQL会很慢。
## 9、覆盖索引优化
**总结**

- 禁止使用select *，禁止查询与业务无关字段
- 尽量利用覆盖索引
# 慢查询日志
## 1、是什么
一种日志记录，查看哪些SQL超出了我们的最大忍耐时间值。
## 2、开启慢查询日志参数
### 2.1、开启slow_query_log
默认情况下，MySQL数据库没有开启慢查询日志，需要我们手动来设置这个参数。当然，如果不是调优需要的话，一般不建议启动该参数，因为开启慢查询日志会或多或少带来一定的性能影响。
SET GLOBAL slow_query_log=1; 
然后我们再来查看下慢查询日志是否开启，以及慢查询日志文件的位置：
SHOW VARIABLES LIKE '%slow_query_log%'; 
### 2.2、修改long_query_time阈值
```
SHOW VARIABLES LIKE '%long_query_time%'; -- 查看值：默认10秒
SET GLOBAL long_query_time=0.1; -- 设置一个比较短的时间，便于测试
```
**注意：**

- **需要重新登录客户端**使上面的设置生效
- 假如运行时间正好等于long_query_time的情况，并不会被记录下来。
- 也就是说，在mysql源码里是判断大于long_query_time，而非大于等于。
### 2.3、日志分析工具
**执行耗时sql：**
```
SELECT * from emp;
SELECT * FROM emp;
SELECT * FROM emp WHERE deptid > 1;
```
**查询慢查询记录数：**
SHOW GLOBAL STATUS LIKE '%Slow_queries%'; 
**查询日志：**
```
#退出mysql容器
quit
#查看慢查询日志
cat -n /var/lib/mysql/d7b5e8f7503e-slow.log
```
**mysqldumpslow：**
在生产环境中，如果要手工分析日志，查找、分析SQL，显然是个体力活，MySQL提供了日志分析工具mysqldumpslow。
**注意：**默认情况下，传统rpm方式安装的MySQL环境自带mysqldumpslow工具，直接使用即可。docker下安装的MySQL环境没有mysqldumpslow工具。
退出mysql命令行，执行以下命令：
```
-- 查看mysqldumpslow的帮助信息
mysqldumpslow --help

-- 工作常用参考
-- 1.得到返回记录集最多的10个SQL
mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow.log
-- 2.得到访问次数最多的10个SQL
mysqldumpslow -s c -t 10 /var/lib/mysql/atguigu-slow.log
-- 3.得到按照时间排序的前10条里面含有左连接的查询语句
mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/atguigu-slow.log
-- 4.另外建议在使用这些命令时结合 | 和more 使用 ，否则语句过多有可能出现爆屏情况
mysqldumpslow -s r -t 10 /var/lib/mysql/atguigu-slow.log | more
```

- -a: 不将数字抽象成N，字符串抽象成S
- -s: 是表示按照何种方式排序；
   - c: sql语句的访问次数
   - l: 锁定时间
   - r: 返回数据记录集的总数量
   - t: 查询时间
   - al:平均锁定时间
   - ar:平均返回记录数
   - at:平均查询时间
- -t: 即为返回前面多少条的数据；
- -g: 后边搭配一个正则匹配模式，大小写不敏感的；
# View视图
## 1、是什么

- 将一段查询sql封装为一个虚拟的表。 
- 这个虚拟表只保存了sql逻辑，不会保存任何查询结果。
## 2、作用

- 封装复杂sql语句，提高复用性
- 逻辑放在数据库上面，更新不需要发布程序，面对频繁的需求变更更灵活
## 3、适用场景

- 共用查询结果
- 报表
## 4、语法
### 4.1、创建
```
-- 语法
CREATE VIEW view_name 
AS SELECT column_name(s) FROM table_name WHERE condition;

-- 例如：求所有人物对应的掌门名称
CREATE VIEW v_ceo AS
SELECT emp.name, ceo.name AS ceoname 
FROM t_emp emp
LEFT JOIN t_dept dept ON emp.deptid = dept.id 
LEFT JOIN t_emp ceo ON dept.ceo = ceo.id;
```
### 4.2、使用
**查询**
```
-- 语法
SELECT * FROM view_name; 

-- 例如：
SELECT * FROM v_ceo;
```
**更新**
```
-- 语法
CREATE OR REPLACE VIEW view_name 
AS SELECT column_name(s) FROM table_name WHERE condition

-- 建议直接删除重新创建
```
**删除**
```
DROP VIEW view_name;

-- 例如：
DROP VIEW v_ceo;
```

 
# MVCC
MVCC（Multi-Version Concurrency Control，多版本并发控制）是一种用于数据库管理系统的并发控制技术，用于在多个事务并发执行时保持数据的一致性和隔离性。MVCC的核心思想是在事务执行过程中，数据库会为每个事务创建一个独立的数据版本，从而避免了读写冲突，提高了并发性能。
以下是MVCC的一些关键概念和工作原理：

1. **版本号**： 在MVCC中，每个数据行都有一个版本号或时间戳，表示数据的创建或修改时间。
2. **读操作**： 当一个事务要读取数据时，它会看到在其开始时间之前已提交的事务所做的修改。换句话说，事务只会看到在它开始之前已经存在的数据版本。
3. **写操作**： 当一个事务要修改数据时，它会创建一个新的数据版本，并将修改写入这个版本。其他正在执行的事务不会受到这个修改的影响，因为它们会继续访问旧的数据版本。
4. **事务隔离级别**： MVCC支持不同的事务隔离级别，如读未提交、读已提交、可重复读和串行化。每个隔离级别对于数据版本的可见性和事务并发性都有不同的要求。
5. **垃圾回收**： 随着时间的推移，旧的数据版本可能会变得无用。数据库系统需要定期进行垃圾回收，删除已经不再需要的旧版本，以释放存储空间。

MVCC的优势在于它可以提供高度的并发性能，因为不同事务可以并行地读取和写入数据，而不会导致锁冲突。然而，实现MVCC需要一些复杂的数据结构和管理机制，对于数据库引擎的设计和开发来说是一个挑战。主流的数据库系统，如MySQL、PostgreSQL等，都使用了MVCC来支持并发事务。


MySQL中的MVCC（Multi-Version Concurrency Control，多版本并发控制）是通过以下方式来实现的：

1. **版本号和数据存储**：
   - 每一行数据都会存储一个或多个版本号或时间戳，用来表示数据的创建或修改时间。
   - 数据行的存储通常包括多个版本，每个版本对应一个事务的修改或插入。这些版本被称为快照。
2. **系统版本号**：
   - MySQL会维护一个系统版本号，表示当前系统的全局时间戳。这个版本号会不断递增，用于判断事务的可见性。
3. **事务版本号**：
   - 每个事务都会分配一个唯一的事务版本号或ID。这个版本号会与事务的开始时间相关联。
4. **读操作**：
   - 当一个事务开始时，它会记录下当前的系统版本号作为自己的事务版本号。
   - 在读取数据时，事务会检查数据行的版本号，只读取在事务版本号之前创建或修改的数据版本。这确保了事务只能看到在它开始之前已经存在的数据快照。
5. **写操作**：
   - 当一个事务要修改数据时，它会创建一个新的数据版本，修改并存储在其中。
   - 这个新版本的数据会与事务的版本号相关联，以及与数据行原有的版本一起存储。
6. **事务隔离级别**：
   - MySQL支持不同的事务隔离级别，如读未提交、读已提交、可重复读和串行化。这些隔离级别会影响事务的可见性和并发性。
7. **垃圾回收**：
   - 随着时间的推移，旧的数据版本可能会变得无用。MySQL会定期进行垃圾回收，删除已经不再需要的旧版本，以释放存储空间。

通过这种方式，MySQL实现了MVCC，允许多个事务并发执行，而不会相互干扰或导致锁冲突。每个事务可以独立地读取和修改数据，而不会直接影响其他事务，从而提高了数据库的并发性能。这种机制还使得MySQL能够支持不同的事务隔离级别，以满足不同应用场景的需求。

# 
 
