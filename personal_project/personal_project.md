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
    任何在 Order by 语句的非索引项或者有计算表达式都将降低查询速度,如不必要，应避免使用,可尝试以 WHERE 替代。
    ```sql
    SELECT todo.tid, todo.content, todo.ddl, todo.importance, todo.status, list.list_name 
    FROM todo 
    LEFT JOIN list 
    ON todo.lid=list.lid 
    WHERE todo.uid ="'.$_SESSION['userid'].'" and todo.ddl >= CURDATE() 
    ORDER BY todo.status asc, todo.ddl asc'
    ```

6. 使用 OR 关键字的查询  
    查询语句的查询条件中只有 OR 关键字，且 OR 前后的两个条件中的列都是索引时，索引才会生效，否则，索引不生效。通常情况下, 用 UNION 替换 OR 将会起到较好的效果, 以上规则只针对多个索引列有效。
    ```sql    
    //高效: 
    SELECT LOC_ID , LOC_DESC , REGION 
    FROM todo 
    WHERE LOC_ID = 10 
    UNION 
    SELECT LOC_ID , LOC_DESC , REGION 
    FROM todo 
    WHERE REGION = “MELBOURNE” 
    //低效: 
    SELECT LOC_ID , LOC_DESC , REGION 
    FROM todo 
    WHERE LOC_ID = 10 OR REGION = “MELBOURNE” 
    ```

6. 用 WHERE 替换 HAVING   
    避免使用 HAVING 子句, HAVING 会在检索出所有记录之后才对结果集进行过滤。这个处理需要排序、总计等操作。如果能通过 WHERE 子句限制记录数目,就能减少开销。

7. 用 EXISTS 替换 IN   
    在许多基于基础表的查询中,为了满足一个条件,往往需要对另一个表进行联接，在这种情况下, 使用 EXISTS 将提高查询的效率。在子查询中,NOT IN 子句将执行一个内部的排序和合并，是低效的。

8. 减少对表的查询，通过内部函数提高效率   
    在含有子查询的 SQL 语句中,要注意减少对表的查询。可以通过内部函数提高效率，甚至可以整合简单、无关联的数据库访问。 
    ```sql    
    SELECT  TAB_NAME FROM TABLES WHERE (TAB_NAME,DB_VER) = ( SELECT 
    TAB_NAME,DB_VER FROM  TAB_COLUMNS  WHERE  VERSION = 604) 
    ```

9. 使用表的别名  
    当语句中连接多个表时, 使用表的别名并把别名前缀于每一列上，减少解析的时间以及由歧义引起的语法错误。
    ```sql
    SELECT todo.tid, todo.content, list.list_name 
    FROM todo 
    LEFT JOIN list 
    ON todo.lid=list.lid 
    ```
10. 避免使用 IS NULL 与 IS NOT NULL  
    使用 IS NULL 与 IS NOT NULL 时，不允许使用索引。
    ```sql    
    //低效: (索引失效) 
    SELECT … FROM  …  WHERE  A IS NOT NULL; 
    //高效: (索引有效) 
    SELECT … FROM  …  WHERE  A >=0;
    ``` 

## 心得体会
经过这一主题的自学后，反思本组项目开发流程及成果，我发现了很多现有代码中急待优化或仍有改进空间的地方。很多方面是非常细节的，也是我原来完全意识不到的内容。

在以后的编程学习和项目开发中，应注意养成运用效率高的 SQL 代码的编程习惯，并且学会设计规范化的数据库。除此之外，还应意识到不是所有优化方式都适用于各种环境，在实际项目中，应首先明确开发环境与开发工具的各种特点，结合自己的实际需求（有时需要理清需求），再选择开发及优化的原则与方法。在此基础上，还也要养成问题定位与解决的思维模式。

## 参考资料
- MySQL 性能调优和优化资源 
https://www.mysql.com/cn/why-mysql/performance/
- MySQL 性能优化实战，MySQL 性能调优和系统资源优化解决方案
https://blog.csdn.net/Hello_World_QWP/article/details/104900695
- SQL查询优化的步骤
https://www.cnblogs.com/rouqinglangzi/p/11144960.html
