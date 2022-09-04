**DQL**(Data QueryLanguage): 数据查询语言,SELECT;
**DML**(Data Manipulation Language)：数据操纵语言，INSERT,UPDATE,DELETE;
**DDL**(Data Definition Language):数据定义语言，创建表、视图、索引等；
**DCL**(Data Control Language):数据控制语言，GRANT授权等操作；

**新增**
INSERT INTO <表名> (字段1, 字段2, ...) VALUES (值1, 值2, ...);

**更新**
UPDATE <表名> SET 字段1=值1, 字段2=值2, ... WHERE ...;

**删除**
DELETE FROM <表名> WHERE ...;

**插入或替换**
如果我们希望插入一条新记录（INSERT），但如果记录已经存在，就先删除原记录，再插入新记录。此时，可以使用REPLACE语句，这样就不必先查询，再决定是否先删除再插入：
REPLACE INTO students (id, class_id, name, gender, score) VALUES (1, 1, '小明', 'F', 99);

**插入或更新**
如果我们希望插入一条新记录（INSERT），但如果记录已经存在，就更新该记录
INSERT INTO students (id, class_id, name, gender, score) VALUES (1, 1, '小明', 'F', 99) ON DUPLICATE KEY UPDATE name='小明', gender='F', score=99;
返回结果（影响的行数）说明：返回1代表插入成功、返回2代表更新成功、返回0代表已存在并且更新的值与原值相同

**插入或忽略**
如果我们希望插入一条新记录（INSERT），但如果记录已经存在，就啥事也不干直接忽略
INSERT IGNORE INTO students (id, class_id, name, gender, score) VALUES (1, 1, '小明', 'F', 99);

**快照**
如果想要对一个表进行快照，即复制一份当前表的数据到一个新表，可以结合CREATE TABLE和SELECT
CREATE TABLE students_of_class1 SELECT * FROM students WHERE class_id=1;

**写入查询结果集**
如果查询结果集需要写入到表中，可以结合INSERT和SELECT，将SELECT语句的结果集直接插入到指定表中
INSERT INTO statistics (class_id, average) SELECT class_id, AVG(score) FROM students GROUP BY class_id;

**sql权限**

```mysql
# 创建新用户(zdk/123),授权数据库及表(zdk库的所有表),刷新权限使权限生效
create user 'zdk'@'%' identified by '123';
grant all privileges on zdk.* to 'zdk'@'%';
flush privileges ;

# 切换mysql数据库,更新用户密码
use mysql;
update user set authentication_string = password('123') where user='zdk';
```



参考：[廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/1177760294764384/1246617682185952)