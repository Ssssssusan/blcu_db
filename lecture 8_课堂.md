# 第八次课堂作业   
   * 练习存储过程、函数与触发器的定义与调用（不要完全照抄课件内容，照抄分数低）；
   * 存储过程、函数与触发器在真实业务系统中都适用于具体哪些场景，用自己的话说明一下原因。
 
## 存储过程与函数
### 定义
存储过程是在大型数据库系统中，一组为了完成特定功能的SQL语句集，存储在数据库中，经过第一次编译后再次调用不需要再次编译，
用户通过指定存储过程的名字并给出参数（如果该存储过程带有参数）来执行它。
### 适用场景
存储过程适用于处理较为复杂的业务场景：


 
## 触发器  
### 定义调用
```
mysql> use blcu_db_demo;
Database changed

mysql> SELECT * FROM GOODS;
+------+------+------+
| gid  | name | num  |
+------+------+------+
|    1 | cat  |   40 |
|    2 | dog  |   63 |
|    3 | pig  |   87 |
+------+------+------+
3 rows in set (0.00 sec)

mysql> show triggers;
+---------+--------+-------+---------------------------------------------------------+--------+---------+----------+----------------+----------------------+----------------------+--------------------+
| Trigger | Event  | Table | Statement                                               | Timing | Created | sql_mode | Definer        | character_set_client | collation_connection | Database Collation |
+---------+--------+-------+---------------------------------------------------------+--------+---------+----------+----------------+----------------------+----------------------+--------------------+
| t1      | INSERT | ord   | begin
  update goods set num=num-2 where gid = 1;
end | AFTER  | NULL    |          | root@localhost | utf8                 | utf8_general_ci      | utf8_general_ci    |
+---------+--------+-------+---------------------------------------------------------+--------+---------+----------+----------------+----------------------+----------------------+--------------------+
1 row in set (0.03 sec)








mysql> SELECT * FROM GOODS;
+------+------+------+
| gid  | name | num  |
+------+------+------+
|    1 | cat  |   40 |
|    2 | dog  |   63 |
|    3 | pig  |   87 |
+------+------+------+
3 rows in set (0.00 sec)

mysql> show triggers;
+---------+--------+-------+---------------------------------------------------------+--------+---------+----------+----------------+----------------------+----------------------+--------------------+
| Trigger | Event  | Table | Statement                                               | Timing | Created | sql_mode | Definer        | character_set_client | collation_connection | Database Collation |
+---------+--------+-------+---------------------------------------------------------+--------+---------+----------+----------------+----------------------+----------------------+--------------------+
| t1      | INSERT | ord   | begin
  update goods set num=num-2 where gid = 1;
end | AFTER  | NULL    |          | root@localhost | utf8                 | utf8_general_ci      | utf8_general_ci    |
+---------+--------+-------+---------------------------------------------------------+--------+---------+----------+----------------+----------------------+----------------------+--------------------+
1 row in set (0.03 sec)

mysql> drop trigger triggerName;
ERROR 1360 (HY000): Trigger does not exist
mysql> drop trigger t1;
Query OK, 0 rows affected (0.00 sec)

mysql> delimiter $
mysql> create trigger t2
    -> after insert on ord
    -> for each row
    -> begin
    ->   update goods set num=num-new.much where gid=new.gid;
    -> end$
Query OK, 0 rows affected (0.03 sec)

mysql> show triggers;
    -> show triggers;
    -> end$
+---------+--------+-------+------------------------------------------------------------------+--------+---------+----------+----------------+----------------------+----------------------+--------------------+
| Trigger | Event  | Table | Statement                                                        | Timing | Created | sql_mode | Definer        | character_set_client | collation_connection | Database Collation |
+---------+--------+-------+------------------------------------------------------------------+--------+---------+----------+----------------+----------------------+----------------------+--------------------+
| t2      | INSERT | ord   | begin
  update goods set num=num-new.much where gid=new.gid;
end | AFTER  | NULL    |          | root@localhost | gbk                  | gbk_chinese_ci       | utf8_general_ci    |
+---------+--------+-------+------------------------------------------------------------------+--------+---------+----------+----------------+----------------------+----------------------+--------------------+
1 row in set (0.03 sec)

+---------+--------+-------+------------------------------------------------------------------+--------+---------+----------+----------------+----------------------+----------------------+--------------------+
| Trigger | Event  | Table | Statement                                                        | Timing | Created | sql_mode | Definer        | character_set_client | collation_connection | Database Collation |
+---------+--------+-------+------------------------------------------------------------------+--------+---------+----------+----------------+----------------------+----------------------+--------------------+
| t2      | INSERT | ord   | begin
  update goods set num=num-new.much where gid=new.gid;
end | AFTER  | NULL    |          | root@localhost | gbk                  | gbk_chinese_ci       | utf8_general_ci    |
+---------+--------+-------+------------------------------------------------------------------+--------+---------+----------+----------------+----------------------+----------------------+--------------------+
1 row in set (0.05 sec)

ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'end' at line 1
mysql> delimiter $
mysql> create trigger t3 after delete on ord
    -> for each row
    -> begin
    ->  update goods set num=num+old.much where gid=old.gid;
    -> end$
Query OK, 0 rows affected (0.02 sec)

mysql> delimiter $
mysql> create trigger t4 before  update on ord
    -> for each row
    -> begin
    ->  update goods set num=num+old.much-new.much where gid = new.gid;
    -> end$
Query OK, 0 rows affected (0.01 sec)

mysql> show triggers
    ->
    -> show triggers
    -> show triggers;
    -> show triggers;
    -> end $
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'show triggers
show triggers;
show triggers;
end' at line 3
mysql> show triggers;
    -> end$
+---------+--------+-------+----------------------------------------------------------------------------+--------+---------+----------+----------------+----------------------+----------------------+--------------------+
| Trigger | Event  | Table | Statement                                                                  | Timing | Created | sql_mode | Definer        | character_set_client | collation_connection | Database Collation |
+---------+--------+-------+----------------------------------------------------------------------------+--------+---------+----------+----------------+----------------------+----------------------+--------------------+
| t2      | INSERT | ord   | begin
  update goods set num=num-new.much where gid=new.gid;
end           | AFTER  | NULL    |          | root@localhost | gbk                  | gbk_chinese_ci       | utf8_general_ci    |
| t4      | UPDATE | ord   | begin
 update goods set num=num+old.much-new.much where gid = new.gid;
end | BEFORE | NULL    |          | root@localhost | gbk                  | gbk_chinese_ci       | utf8_general_ci    |
| t3      | DELETE | ord   | begin
 update goods set num=num+old.much where gid=old.gid;
end            | AFTER  | NULL    |          | root@localhost | gbk                  | gbk_chinese_ci       | utf8_general_ci    |
+---------+--------+-------+----------------------------------------------------------------------------+--------+---------+----------+----------------+----------------------+----------------------+--------------------+
3 rows in set (0.03 sec)
```




### 
 
 
### 适用场景：
- 维护约束，如防止在选定一门课后删除该课程
- 商业规则，如果客户进行国际货币转账，则向其发送e-mail消息 
- 监控，如传感器感知到一氧化碳浓度级别提高，则开启通风系统 
- 辅助缓存数据的维护，如当基础表发生改变时，更新物化视图 
- 简化应用设计，如将核心编程逻辑从异常处理中分离出来
