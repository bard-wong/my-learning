## 程序结构

```plsql
declare 
  -- 声明部分
  i integer;
begin
  -- 执行部分
exception
  -- 异常部分
end;
```

> 程序块是可以嵌套的

## 变量

### 数据类型

#### 基本类型

数字型：NUMBER
字符型: VARCHAR2, CHAR, LONG, CLOB
日期型：DATE
二进制类型：RAW：存储多媒体数据，如图象、声音、视频等
      				  BLOB：大对象如图象、声音、视频等

#### 记录类型

 TYPE   xxxxx（类型名称，自定义）   IS RECORD（
		……
记录类型中的字段变量声明
	）； 

### 声明变量

```plsql
v_name NUMBER :=1; --声明并赋值
v_name E.name%TYPE; --声明与E表中name字段同类型变量
v_i	CONSTANT NUMBER := 8; --声明常量，只能被赋值一次
v_xxx BOOLEAN NOT NULL := TRUE; --布尔类型，不为空

v_emp emp%ROWTYPE; --记录型变量，声明为emp表所有字段的变量
--记录类型，自定义字段变量
type v_student is record (
  v_name varchar2(30),
  v_age number,
);
newstudent v_student;
```

### 变量赋值

除声明时直接赋值外，可在执行部分赋值

```plsql
v_name := 'zhang'; -- 直接赋值
SELECT 'zhang' INTO v_name FROM xxx; -- 语句赋值
```

## 流程控制

### 条件分支

#### IF

```plsql
begin
  if 条件1 then 执行1 --条件判断=相当于java中的==
    elsif 条件2 then 执行2
    else 执行3
  end if;

end;
```

#### DECODE

> decode(条件，值1，返回值1，值2，返回值2，...值n,返回值n，缺省值)

#### CASE WHEN 

```plsql
--简单Case函数  

CASE sex  
WHEN '1' THEN '男'  
WHEN '2' THEN '女'  
ELSE '其他' END  

--Case搜索函数  

CASE
WHEN sex = '1' THEN '男'  
WHEN sex = '2' THEN '女'  
ELSE '其他' END 
```



### 循环

#### LOOP循环

```plsql
begin
  loop
    执行语句
    exit when 退出循环条件
  end loop;
    
end;
```

#### WHILE循环

```plsql
begin
  while 条件判断 loop
    执行语句
  end loop;
    
end;
```

#### FOR循环

```plsql
begin
  for 计数器变量 in 低界值..高界值 loop
    执行语句
  end loop;
    
end;
```

> CONTINUE，可退出本次循环（Oracle 11g 新增）

## 游标

### 隐式游标

增删改查语句都会有隐式游标，可以通过隐式游标来分析受到增删改查影响的数据

> 隐式游标一般使用SQL前缀，例如SQL%ROWCOUNT；

```plsql
DECLARE 
  V_COUNT NUMBER(3);
BEGIN
  DELETE FROM EMP WHERE DEPTNO = 10;
  V_COUNT := SQL%ROWCOUNT;
  DBMS_OUTPUT.put_line('被删除的数据条数是：'||V_COUNT);
END;
```



### 显式游标

用于临时存储一个查询返回的多行数据（结果集，类似于java的JDBC连接返回的ResultSet集合），通过遍历游标，可以逐行返回处理该结果集的数据。
游标的使用方式：声明---打开---读取---关闭

#### 语法

游标声明：

> CURSOR 游标名[(参数列表)] IS 查询语句;

游标打开：

> OPEN 游标名;

游标的取值：

> FETCH 游标名 INTO 变量列表;

游标的关闭：

> CLOSE 游标名;

#### 游标的属性

| 游标的属性 | 返回值类型 | 说明                                      |
| ---------- | ---------- | ----------------------------------------- |
| %ROWCOUNT  | 整型       | 获得FETCH语句返回的数据行数               |
| %FOUND     | 布尔型     | 最近的FETCH语句返回一行数据则为真，否则假 |
| %NOTFOUND  | 布尔型     | 与%FOUND属性相反                          |
| %ISOPEN    | 布尔型     | 游标打开时为真，关闭为假                  |

例：无参游标打印姓名和工资

```plsql
DECLARE
  CURSOR c_emp IS
    SELECT EMP_NAME, BASE_SALARY FROM SIE_EMP order by EMP_NO;
  ename SIE_EMP.EMP_NAME%TYPE;
  sal   SIE_EMP.BASE_SALARY%TYPE;
BEGIN
  OPEN c_emp;

  loop
    fetch c_emp
      into ename, sal;
    exit when c_emp%notfound;
    DBMS_OUTPUT.PUT_LINE(ename || ' - ' || sal);
  end loop;

  CLOSE c_emp;
END;
```

例：有参游标根据输入ID打印姓名和工资

```plsql
DECLARE
  CURSOR c_emp(v_id SIE_EMP.EMP_ID%TYPE) IS
    SELECT EMP_NAME, BASE_SALARY
      FROM SIE_EMP
     WHERE EMP_ID = v_id
     order by EMP_NO;
  ename SIE_EMP.EMP_NAME%TYPE;
  sal   SIE_EMP.BASE_SALARY%TYPE;
BEGIN
  OPEN c_emp(1); --传入游标值

  loop
    fetch c_emp
      into ename, sal;
    exit when c_emp%notfound;
    DBMS_OUTPUT.PUT_LINE(ename || ' - ' || sal);
  end loop;

  CLOSE c_emp;
END;
```

> FETCH应放在EXIT WHEN之前，因为%NOTFOUND默认为false

## 存储过程

```plsql
CREATE OR REPLACE PROCEDURE ROUTINE_NAME(参数列表) IS
--声明变量部分
BEGIN
--执行语句部分
END [过程名称];
```

### 无参存储

```plsql
CREATE OR REPLACE PROCEDURE P_HELLO IS  --IS和AS互用，用哪个都可以
BEGIN
    DBMS_OUTPUT.PUT_LINE('HELLO WORLD');
END P_HELLO;
```

调用

```plsql
BEGIN
    P_HELLO();
END;
```

### 带输入参数存储

```plsql
CREATE OR REPLACE PROCEDURE P_QUERYNAME(v_id IN SIE_EMP.EMP_ID%TYPE) AS --定义存储过程参数用IN/OUT区分输入输出参数
  v_name SIE_EMP.EMP_NAME%TYPE;
  v_sal  SIE_EMP.BASE_SALARY%TYPE;
BEGIN
  SELECT EMP_NAME, BASE_SALARY
    INTO v_name, v_sal
    FROM SIE_EMP
   WHERE EMP_ID = v_id;
  DBMS_OUTPUT.PUT_LINE(v_name || '_' || v_sal);
END P_QUERYNAME;
```

调用

```plsql
BEGIN
  P_QUERYNAME(1);
END;
```

### 带输出参数存储

```plsql
CREATE OR REPLACE PROCEDURE P_QUERYSAL(v_name IN SIE_EMP.EMP_NAME%TYPE,
                                       v_sal  OUT SIE_EMP.BASE_SALARY%TYPE) AS
BEGIN
  SELECT BASE_SALARY INTO v_sal FROM SIE_EMP WHERE EMP_NAME = v_name;
END P_QUERYSAL;
```

调用

```plsql
DECLARE
  v_sal SIE_EMP.BASE_SALARY%TYPE; --定义变量接收存储过程返回值
BEGIN
  P_QUERYSAL('张三', v_sal);
  DBMS_OUTPUT.PUT_LINE(v_sal);
END;
```

## JAVA程序调用存储过程