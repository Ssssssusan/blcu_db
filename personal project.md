# 个人项目：SQL 性能与查询优化

```
写一篇实践小论文，并用编写实验过程的实验代理。

要求：
需要完成可运行的实验代码
需要有实验结果截图
需要分享心得
```
## 前言  
在课程开发项目中，由于数据库数据较少、查询逻辑较为简单等原因，我们很难体会到数据库查询相关的性能优劣。但在实际应用的系统中，随着数据量的增加，系统的响应速度将成为主要问题之一。SQL 语句各种写法可导致性能的优劣之分，系统优化中一项重要方法即 SQL 查询语句的优化。

优化查询，首先要理解查询语句背后的**执行逻辑**。数据库根据 SQL 语句和相关数据表的统计信息形成查询方案，该方案由查询优化器自动分析产生。多数情况下，索引被应用以更快地遍历数据表，优化器主要根据定义的索引来提高性能。然而，如果 SQL 语句不合理，优化器将选择删去索引而使用全表扫描，导致性能降低。所以，在编写 SQL 语句时应清楚其执行原则，提高性能。我们可以从以下两个角度考虑这一问题：

- SQL 语句清晰地告诉查询优化器它想干什么；
- 查询优化器得到的数据库统计信息是最新的、正确的。

---

## 问题定位
SQL 优化一般步骤为：
- 查询数据库运行状况
- 定位慢查询
- 分析 SQL 执行过程
- 优化

1.使用 show status 查询数据库运行状况
```sql
//显示数据库运行状态
SHOW STATUS
//显示数据库运行总时间
SHOW STATUS LIKE 'uptime'
//显示连接的次数
SHOW STATUS LIKE 'connections'
//显示执行CRUD的次数
SHOW STATUS LIKE 'com_select'
SHOW STATUS LIKE 'com_insert'
SHOW STATUS LIKE 'com_update'
SHOW STATUS LIKE 'com_delete'
```
2.定位慢查询 SQL  
一旦有 SQL 执行时间超过了设置的慢查询时间，就会被记录到日志中。慢查询默认是关闭的，可以通过修改配置文件或通过命令来开启。
```sql
//显示慢查询次数
SHOW STATUS LIKE 'slow_queries'
//显示慢查询时间，默认为10s
SHOW VARIABLES LIKE 'long_query_time'
```

3.分析 SQL 执行过程
1. 使用 explain  
    ```
    explain [要分析的sql]
    ```
2. 使用 show profile  
    ```
    show profile for query [query id]
    ```
3. 通过 trace 分析优化器如何选择执行计划 

---

## 优化措施
优化措施主要从两个方面考虑：
- 规范数据库或数据表结构
- 优化查询语句

### 规范数据库或数据表结构
1. 尽量不使用 TEXT 数据类型 
    除非处理很大的数据，否则不要使用 TEXT，TEXT 不易于查询且速度慢，用的不好还会浪费大量的空间。一般 VARCHAR 可以更好的处理数据。
2. 选择最有效率的表名顺序 
    ORACLE 的解析器按照从右到左的顺序处理 FROM 子句中的表名，FROM 子句中写在最后的表将被最先处理。在 FROM 子句中包含多个表的情况下,须选择记录条数最少的表作为基础表，如果有3个以上的表连接查询, 那就需要选择交叉表作为基础表。

### 优化查询语句
1. 统一 SQL 语句的写法  
    对于以下语句，数据库查询优化器认为是不同的，必须进行两次解析,编写代码时应注意大小写等语句写法的统一。 
    ```sql
    select * from todo
    select * From todo
    ```
2. 优化过于复杂的 SQL 语句
    一般，将一个 Select 语句的结果作为子集，然后从该子集中再进行查询，这种1层嵌套较为常见。超过3层嵌套，查询优化器就很容易给出错误的执行计划。另外，执行计划可以被重用，越简单的 SQL 语句被重用的可能性越高。而复杂的 SQL 语句只要有一个字符发生变化就要重新解析，导致数据库效率低下。

3. 尽量不使用 SELECT *  
    利用索引，而非全文扫描。
    ```sql
    SELECT todo.tid, todo.content
    FROM todo 
    WHERE todo.uid ="'.$_SESSION['userid'].'"
    ```
4. 使用 LIKE 关键字的查询
    进行模糊查询时，可能应用以下语句。关键词“%$content%”，由于用到了“%”，因此该查询为全表扫描，除非必要，否则不要在关键词前加%。
    ```sql
    SELECT * FROM todo WHERE todo.content LIKE '%$content%'’
    ```

5. 使用 Order by 语句的查询
    任何在 Order by 语句的非索引项或者有计算表达式都将降低查询速度,如不必要，应避免使用。
    ```sql
    SELECT todo.tid, todo.content, todo.ddl, todo.importance, todo.status, list.list_name 
    FROM todo 
    LEFT JOIN list 
    ON todo.lid=list.lid 
    WHERE todo.uid ="'.$_SESSION['userid'].'" and todo.ddl >= CURDATE() 
    ORDER BY todo.status asc, todo.ddl asc'
    ```
6. 用 WHERE 替换 HAVING 
    避免使用 HAVING 子句, HAVING 会在检索出所有记录之后才对结果集进行过滤。这个处理需要排序、总计等操作。如果能通过 WHERE 子句限制记录数目,就能减少开销。

7. 减少对表的查询，通过内部函数提高效率 
    在含有子查询的 SQL 语句中,要注意减少对表的查询. 
    ```sql    
    SELECT  TAB_NAME FROM TABLES WHERE (TAB_NAME,DB_VER) = ( SELECT 
    TAB_NAME,DB_VER FROM  TAB_COLUMNS  WHERE  VERSION = 604) 
    ```

8. 使用表的别名
    当语句中连接多个表时, 使用表的别名并把别名前缀于每一列上，减少解析的时间以及由歧义引起的语法错误。
    ```sql
    SELECT todo.tid, todo.content, list.list_name 
    FROM todo 
    LEFT JOIN list 
    ON todo.lid=list.lid 
    ```
9. 用 EXISTS 替代 IN 
    在许多基于基础表的查询中,为了满足一个条件,往往需要对另一个表进行联接，在这种情况下, 使用 EXISTS 将提高查询的效率。在子查询中,NOT IN 子句将执行一个内部的排序和合并，是低效的。




4. 使用联合索引的查询
MySQL可以为多个字段创建索引，一个索引可以包括16个字段。对于联合索引，只有查询条件中使用了这些字段中第一个字段时，索引才会生效。

3. 使用 OR 关键字的查询
查询语句的查询条件中只有 OR 关键字，且 OR 前后的两个条件中的列都是索引时，索引才会生效，否则，索引不生效。

1. IS NULL 与 IS NOT NULL
    任何在where子句中使用is null或is not null的语句优化器是不允许使用索引的。







（2）      WHERE子句中的连接顺序．： 
ORACLE采用自下而上的顺序解析WHERE子句,根据这个原理,表之间的连接必须写在其他WHERE条件之前, 那些可以过滤掉最大数量记录的条件必须写在WHERE子句的末尾. 


（6）      使用DECODE函数来减少处理时间： 
使用DECODE函数可以避免重复扫描相同记录或重复连接相同的表. 
（7）      整合简单,无关联的数据库访问： 
如果你有几个简单的数据库查询语句,你可以把它们整合到一个查询中(即使它们之间没有关系) 
（8）      删除重复记录： 
最高效的删除重复记录方法 ( 因为使用了ROWID)例子： 
DELETE  FROM  EMP E  WHERE  E.ROWID > (SELECT MIN(X.ROWID) 
FROM  EMP X  WHERE  X.EMP_NO = E.EMP_NO); 
（9）      用TRUNCATE替代DELETE： 
当删除表中的记录时,在通常情况下, 回滚段(rollback segments ) 用来存放可以被恢复的信息. 如果你没有COMMIT事务,ORACLE会将数据恢复到删除之前的状态(准确地说是恢复到执行删除命令之前的状况) 而当运用TRUNCATE时, 回滚段不再存放任何可被恢复的信息.当命令运行后,数据不能被恢复.因此很少的资源被调用,执行时间也会很短. (译者按: TRUNCATE只在删除全表适用,TRUNCATE是DDL不是DML) 





（21） 避免在索引列上使用NOT 通常，　 
我们要避免在索引列上使用NOT, NOT会产生在和在索引列上使用函数相同的影响. 当ORACLE”遇到”NOT,他就会停止使用索引转而执行全表扫描. 
（22） 避免在索引列上使用计算． 
WHERE子句中，如果索引列是函数的一部分．优化器将不使用索引而使用全表扫描． 
举例: 
低效： 
SELECT … FROM  DEPT  WHERE SAL * 12 > 25000; 
高效: 
SELECT … FROM DEPT WHERE SAL > 25000/12; 
（23） 用>=替代> 
高效: 
SELECT * FROM  EMP  WHERE  DEPTNO >=4 
低效: 
SELECT * FROM EMP WHERE DEPTNO >3 
两者的区别在于, 前者DBMS将直接跳到第一个DEPT等于4的记录而后者将首先定位到DEPTNO=3的记录并且向前扫描到第一个DEPT大于3的记录. 
（24） 用UNION替换OR (适用于索引列) 
通常情况下, 用UNION替换WHERE子句中的OR将会起到较好的效果. 对索引列使用OR将造成全表扫描. 注意, 以上规则只针对多个索引列有效. 如果有column没有被索引, 查询效率可能会因为你没有选择OR而降低. 在下面的例子中, LOC_ID 和REGION上都建有索引. 
高效: 
SELECT LOC_ID , LOC_DESC , REGION 
FROM LOCATION 
WHERE LOC_ID = 10 
UNION 
SELECT LOC_ID , LOC_DESC , REGION 
FROM LOCATION 
WHERE REGION = “MELBOURNE” 
低效: 
SELECT LOC_ID , LOC_DESC , REGION 
FROM LOCATION 
WHERE LOC_ID = 10 OR REGION = “MELBOURNE” 
如果你坚持要用OR, 那就需要返回记录最少的索引列写在最前面. 
（25） 用IN来替换OR  
这是一条简单易记的规则，但是实际的执行效果还须检验，在ORACLE8i下，两者的执行路径似乎是相同的．　 
低效: 
SELECT…. FROM LOCATION WHERE LOC_ID = 10 OR LOC_ID = 20 OR LOC_ID = 30 
高效 
SELECT… FROM LOCATION WHERE LOC_IN  IN (10,20,30); 
（26） 避免在索引列上使用IS NULL和IS NOT NULL 
避免在索引中使用任何可以为空的列，ORACLE将无法使用该索引．对于单列索引，如果列包含空值，索引中将不存在此记录. 对于复合索引，如果每个列都为空，索引中同样不存在此记录.　如果至少有一个列不为空，则记录存在于索引中．举例: 如果唯一性索引建立在表的A列和B列上, 并且表中存在一条记录的A,B值为(123,null) , ORACLE将不接受下一条具有相同A,B值（123,null）的记录(插入). 然而如果所有的索引列都为空，ORACLE将认为整个键值为空而空不等于空. 因此你可以插入1000 条具有相同键值的记录,当然它们都是空! 因为空值不存在于索引列中,所以WHERE子句中对索引列进行空值比较将使ORACLE停用该索引. 
低效: (索引失效) 
SELECT … FROM  DEPARTMENT  WHERE  DEPT_CODE IS NOT NULL; 
高效: (索引有效) 
SELECT … FROM  DEPARTMENT  WHERE  DEPT_CODE >=0; 
（27） 总是使用索引的第一个列： 
如果索引是建立在多个列上, 只有在它的第一个列(leading column)被where子句引用时,优化器才会选择使用该索引. 这也是一条简单而重要的规则，当仅引用索引的第二个列时,优化器使用了全表扫描而忽略了索引 
28） 用UNION-ALL 替换UNION ( 如果有可能的话)： 
当SQL 语句需要UNION两个查询结果集合时,这两个结果集合会以UNION-ALL的方式被合并, 然后在输出最终结果前进行排序. 如果用UNION ALL替代UNION, 这样排序就不是必要了. 效率就会因此得到提高. 需要注意的是，UNION ALL 将重复输出两个结果集合中相同记录. 因此各位还是要从业务需求分析使用UNION ALL的可行性. UNION 将对结果集合排序,这个操作会使用到SORT_AREA_SIZE这块内存. 对于这块内存的优化也是相当重要的. 下面的SQL可以用来查询排序的消耗量 
低效： 
SELECT  ACCT_NUM, BALANCE_AMT 
FROM  DEBIT_TRANSACTIONS 
WHERE TRAN_DATE = ’31-DEC-95′ 
UNION 
SELECT ACCT_NUM, BALANCE_AMT 
FROM DEBIT_TRANSACTIONS 
WHERE TRAN_DATE = ’31-DEC-95′ 
高效: 
SELECT ACCT_NUM, BALANCE_AMT 
FROM DEBIT_TRANSACTIONS 
WHERE TRAN_DATE = ’31-DEC-95′ 
UNION ALL 
SELECT ACCT_NUM, BALANCE_AMT 
FROM DEBIT_TRANSACTIONS 
WHERE TRAN_DATE = ’31-DEC-95′ 
（29） 用WHERE替代ORDER BY： 
ORDER BY 子句只在两种严格的条件下使用索引. 
ORDER BY中所有的列必须包含在相同的索引中并保持在索引中的排列顺序. 
ORDER BY中所有的列必须定义为非空. 
WHERE子句使用的索引和ORDER BY子句中所使用的索引不能并列. 
例如: 
表DEPT包含以下列: 
DEPT_CODE PK NOT NULL 
DEPT_DESC NOT NULL 
DEPT_TYPE NULL 
低效: (索引不被使用) 
SELECT DEPT_CODE FROM  DEPT  ORDER BY  DEPT_TYPE 
高效: (使用索引) 
SELECT DEPT_CODE  FROM  DEPT  WHERE  DEPT_TYPE > 0 
（30） 避免改变索引列的类型.: 
当比较不同数据类型的数据时, ORACLE自动对列进行简单的类型转换. 
假设 EMPNO是一个数值类型的索引列. 
SELECT …  FROM EMP  WHERE  EMPNO = ‘123′ 
实际上,经过ORACLE类型转换, 语句转化为: 
SELECT …  FROM EMP  WHERE  EMPNO = TO_NUMBER(‘123′) 
幸运的是,类型转换没有发生在索引列上,索引的用途没有被改变. 
现在,假设EMP_TYPE是一个字符类型的索引列. 
SELECT …  FROM EMP  WHERE EMP_TYPE = 123 
这个语句被ORACLE转换为: 
SELECT …  FROM EMP  WHERETO_NUMBER(EMP_TYPE)=123 
因为内部发生的类型转换, 这个索引将不会被用到! 为了避免ORACLE对你的SQL进行隐式的类型转换, 最好把类型转换用显式表现出来. 注意当字符和数值比较时, ORACLE会优先转换数值类型到字符类型 
（31） 需要当心的WHERE子句: 
某些SELECT 语句中的WHERE子句不使用索引. 这里有一些例子. 
在下面的例子里, (1)‘!=’ 将不使用索引. 记住, 索引只能告诉你什么存在于表中, 而不能告诉你什么不存在于表中. (2) ‘ ¦ ¦’是字符连接函数. 就象其他函数那样, 停用了索引. (3) ‘+’是数学函数. 就象其他数学函数那样, 停用了索引. (4)相同的索引列不能互相比较,这将会启用全表扫描. 
（32） a. 如果检索数据量超过30%的表中记录数.使用索引将没有显著的效率提高. 
b. 在特定情况下, 使用索引也许会比全表扫描慢, 但这是同一个数量级上的区别. 而通常情况下,使用索引比全表扫描要块几倍乃至几千倍! 




====================================


五、了解你将要对数据进行的操作 
为你的数据库创建一个健壮的索引，那可是功德一件。可要做到这一点简直就是一门艺术。每当你为一个表添加一个索引，SELECT会更快了，可INSERT 和DELETE却大大的变慢了，因为创建了维护索引需要许多额外的工作。显然，这里问题的关键是：你要对这张表进行什么样的操作。这个问题不太好把握，特别是涉及DELETE和UPDATE时，因为这些语句经常在WHERE部分包含SELECT命令。 
六、不要给“性别”列创建索引 
首先，我们必须了解索引是如何加速对表的访问的。你可以将索引理解为基于一定的标准上对表进行划分的一种方式。如果你给类似于“性别”这样的列创建了一个 索引，你仅仅是将表划分为两部分：男和女。你在处理一个有1,000,000条记录的表，这样的划分有什么意义？记住：维护索引是比较费时的。当你设计索 引时，请遵循这样的规则：根据列可能包含不同内容的数目从多到少排列，比如：姓名+省份+性别。 


十一、使用参数查询 
有时，我在CSDN技术论坛看到类似这样的问题：“SELECT * FROM a WHERE a.id=’A’B，因为单引号查询发生异常，我该怎么办？”，而普遍的回答是：用两个单引号代替单引号。这是错误的。这样治标不治本，因为你还会在其他 一些字符上遇到这样的问题，更何况这样会导致严重的bug，除此以外，这样做还会使SQL Server的缓冲系统无法发挥应有的作用。使用参数查询，釜底抽薪，这些问题统统不存在了。 
十二、在程序编码时使用大数据量的数据库 
程序员在开发中使用的测试数据库一般数据量都不大，可经常的是最终用户的数据量都很大。我们通常的做法是不对的，原因很简单：现在硬盘不是很贵，可为什么性能问题却要等到已经无可挽回的时候才被注意呢？ 


十六、在细节表中插入纪录时，不要在主表执行SELECT MAX(ID) 
这是一个普遍的错误，当两个用户在同一时间插入数据时，这会导致错误。你可以使用SCOPE_IDENTITY，IDENT_CURRENT和IDENTITY。如果可能，不要使用IDENTITY，因为在有触发器的情况下，它会引起一些问题（详见这里的讨论）。 





　　2. 尽量建好和使用好索引:建索引也是有讲究的，在建索引时，也不是索引越多越好，当一个表的索引达到4个以上时，ORACLE的性能可能还是改善不了，因为 OLTP系统每表超过5个索引即会降低性能，而且在一个sql 中， Oracle 从不能使用超过 5个索引;当我们用到GROUP BY和ORDER BY时,ORACLE就会自动对数据进行排序,而ORACLE在INIT.ORA中决定了sort_area_size区的大小,当排序不能在我们给定的 排序区完成时,ORACLE就会在磁盘中进行排序,也就是我们讲的临时表空间中排序, 过多的磁盘排序将会令 free buffer waits 的值变高,而这个区间并不只是用于排序的,对于开发人员我提出如下忠告:

　　1)、select,update,delete 语句中的子查询应当有规律地查找少于20%的表行.如果一个语句查找的行数超过总行数的20%,它将不能通过使用索引获得性能上的提高.

　　2)、索引可能产生碎片,因为记录从表中删除时,相应也从表的索引中删除.表释放的空间可以再用,而索引释放的空间却不能再用.频繁进行删除操 作的被索引的表,应当阶段性地重建索引,以避免在索引中造成空间碎片,影响性能.在许可的条件下,也可以阶段性地truncate表,truncate命 令删除表中所有记录,也删除索引碎片.

　　3)、在使用索引时一定要按索引对应字段的顺序进行引用。
　　4)、用(+)比用NOT IN更有效率。

　　

　

　　6. IN和EXISTS

　　有时候会将一列和一系列值相比较。最简单的办法就是在where子句中使用子查询。在where子句中可以使用两种格式的子查询。

　　第一种格式是使用IN操作符:

… where column in(select * from … where …);

第二种格式是使用EXIST操作符:

… where exists (select ‘X’ from …where …);

 

 

4、使用“临时表”暂存中间结果
简化SQL语句的重要方法就是采用临时表暂存中间结果，但是，临时表的好处远远不止这些，将临时结果暂存在临时表，后面的查询就在tempdb中了，这可以避免程序中多次扫描主表，也大大减少了程序执行中“共享锁”阻塞“更新锁”，减少了阻塞，提高了并发性能。


查询条件语句中减少使用函数，避免全表扫描
减少不必要的表连接
不能循环执行查询
用 exists 代替 in 
inner关联的表可以先查出来，再去关联leftjoin的表
可以进行表关联数据拆分，即先查出核心数据，再通过核心数据查其他数据
表关联时取别名，也能提高效率












## 参考资料
- MySQL 性能调优和优化资源 
https://www.mysql.com/cn/why-mysql/performance/
- MySQL 性能优化实战，MySQL 性能调优和系统资源优化解决方案
https://blog.csdn.net/Hello_World_QWP/article/details/104900695
- SQL查询优化的步骤
https://www.cnblogs.com/rouqinglangzi/p/11144960.html