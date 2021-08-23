- DQL（数据查询语言）：查询语句，凡是SELECT语句都是DQL

- DML（数据操作语言）：INSERT、DELETE、UPDATE 对表当中的数据进行增删改

- DDL（数据定义语言）：CREATE、DROP、ALTER对表结构的增删改

- TCL（事务控制语言）：COMMIT提交事务，ROLLBACK回滚事务

- DCL（数据控制语言）：GRANT授权，REVOKE撤销权限等




source xxx.sql 运行SQL文件

> 

条件查询 语法格式：

```sql
SELECT					//5
	字段,字段
FROM					//1
	表名
WHERE					//2
	条件
GROUP BY				//3
	...
HAVING					//4
	...
ORDER BY				//6
	...
LIMIT					//7
	...					
```

执行顺序：先FROM，然后WHERE，再SELECT，最后ORDER BY



BETWEEN...AND... 闭区间            1、使用时必须左小右大  2、应用在字符上时左闭右开



分组函数： 

1. COUNT 计数
2. SUM 求和
3. AVG平均值
4. MAX最大值
5. MIN最小值

IFNULL 空处理函数：IFNULL(可能为null的数据, 被当做什么处理)

DISTINCT 去除重复数据

内连接

```sql
SELECT
	ENAME 姓名,EMP.DEPTNO 部门号,DNAME 部门名
FROM
	EMP
INNER JOIN				//内连接
	DEPT
ON
	EMP.DEPTNO = DEPT.DEPTNO;
```

自连接

```sql
SELECT
	A.ENAME 员工名,
	A.MGR 领导编号,
	B.ENAME 领导名 
FROM
	EMP A
	INNER JOIN EMP B ON B.EMPNO = A.MGR;
```

外连接

```sql
SELECT
	A.ENAME 员工名,
	A.MGR 领导编号,
	B.ENAME 领导名 
FROM
	EMP A
	LEFT JOIN EMP B ON B.EMPNO = A.MGR;
```

外连接特点：主表的数据无条件的全部查询

三表连接查询

```sql
SELECT
	empA.ENAME 员工名,
	empB.ENAME 领导名,
	empA.MGR 领导编号,
	dept.DNAME 部门名,
	salgrade.GRADE 工资等级 
FROM
	emp empA
	JOIN dept ON dept.DEPTNO = empA.DEPTNO
	JOIN salgrade ON empA.SAL BETWEEN salgrade.LOSAL 
	AND salgrade.HISAL
	LEFT JOIN emp empB ON empA.MGR = empB.EMPNO
```

子查询

1、WHERE后

```sql
SELECT
	emp.ENAME 员工姓名,
	emp.SAL 员工工资 
FROM
	emp 
WHERE
	emp.SAL > (
	SELECT
		AVG( emp.SAL ) 
FROM
	emp)
```

2、FROM后

```sql
SELECT
	salgrade.GRADE 工资等级,
	t.DEPTNO 
FROM
	( SELECT AVG( emp.SAL ) avgsal, emp.DEPTNO FROM emp GROUP BY emp.DEPTNO ) t
	JOIN salgrade ON t.avgsal BETWEEN salgrade.LOSAL 
	AND salgrade.HISAL;
```

3、SELECT后

```sql
SELECT
	emp.ENAME,
	( SELECT dept.DNAME FROM dept WHERE emp.DEPTNO = dept.DEPTNO ) DNAME 
FROM
	emp;
```

 union 连接不同表中的数据

LIMIT startIndex,length		startIndex表示起始位置，length表示取几个



# 建表

建表语法

```sql
create table 表名（
	字段名1 数据类型;
	字段名2 数据类型;
	字段名3 数据类型;
	...
);
```

数据类型：

- int			整数型（java中的int）
- bigint       长整型（java中的long）
- float         浮点型（java中的float  double）
- char         定义字符串（String）
- varchar    可变长字符串（StringBuffer/StringBuilder）
- date         日期类型（对应java中的java.sql.Date类型）
- BLOB       二进制大对象（存储图片、视频等流媒体信息）Binary Large OBject（对应java中的Object）
- CLOB       字符大对象（存储较大文本，可以存储4G的字符串）Character Large OBject（对应java中的Object）



# DML

INSERT INTO 表名（字段名1,字段名2,字段名3.....)values(值1,值2,值3,....)    要求字段数量和值数量要对应，数据类型也要对应

表的复制：CREATE TABLE 表名 AS SELECT * FROM 表名   （将查询结果复制到新表）

将查询结果插入进表：INSERT INTO 表名 SELECT * FROM 表名

修改数据：UPDATE

UPDATE 表名 SET 字段名1=值1,字段名2=值2...WHERE 条件；

注意：没有条件整张表数据全部更新

删除数据：

DELETE FROM 表名 WHERE 条件;

注意：没有条件全部删除。

删除大表：TRUNCATE TABLE 表名: （表被截断，数据丢失，不可回滚）



增删改查 CRUD

Create （增）Retrueve（检索）Upadate（修改）Delete（删除）



# 约束

- 非空约束（NOT NULL）：约束的字段不能为NULL
- 唯一约束（UNIQUE）：约束的字段不能重复
- 主键约束（PRINARY KEY）：约束的字段既不能为NULL，也不能重复（简称PK）
- 外键约束（FOREIGN KEY）：约束的字段既不能为NULL，也不能重复（简称FK）
- 检查约束（CHECK）注意Oracle数据库有CHECK约束，MySQL不支持该约束



# 事务

COMMIT提交事务，ROLLBACK回滚事务

一个事务是一个完整的是无逻辑单元，不可再分

一个事物通常包含多条DML语句，事务机制保证多条DML语句同时失败或同时成功，和事务相关的语句只有：DML语句。事务的存在是为了保证数据的完整性，安全性。

事务的四大特性：ACID

- A：原子性：事务是最小的工作单位，不可再分
- C：一致性：事务必须保证多条DML语句同时成功或者同时失败
- I：隔离性：事务A与事务B之间具有隔离。
- D：持久性：最终数据必须持久化到硬盘文件中，事务才算成功的结束

事务之间的隔离性：事务隔离性存在隔离级别，理论上隔离级别包括4个：

第一级别：读未提交（read uncmmitted）：对方事务还没有提交，我们当前事务可以读取到对方未提交的数据。读未提交存在脏读（Dirty Read）现象：表示读到了脏数据

第二级别：读已提交（read committed）：对方事务提交之后的数据我方可以读取到。读已提交存在的问题是：不可重复读

第三级别：可重复读（repeatable read）：这种隔离级别解决了：不可重复读问题。这种隔离级别存在的问题是：读取到的数据是幻象

第四级别：序列化读/串行化读：解决了所有问题。效率低，需要事务排队

Oracle数据库默认：读已提交。mysql数据库默认：可重复读

设置事务隔离级别：set global transaction isolation level read unconmmitted




