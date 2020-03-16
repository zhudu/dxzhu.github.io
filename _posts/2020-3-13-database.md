---
layout: post
title: 关系数据库标准语言
categories: database
description: 关系数据库标准语言
keywords: database
---

## SQL概述

SQL(Structured Query Language结构化查询语言)：不仅仅具有查询功能。它是一个通用的、功能极强的关系数据库操作语言。

#### SQL的特点

- 面向集合化、高度非过程化
- 集数据定义、数据查询和数据控制功能于一体
- 统一语法结构的两种使用方式，简单易学

#### SQL的基本组成

- SQL由数据定义语言(DDL)、交互式数据操纵语言(DML)、事务控制、完整性及权限管理等部分组成。

------

SQL语言支持关系数据库的三级模式结构，其中视图对应外模式，基本表对应模式，存储文件对应内模式。

![](\images\posts\database\202003135.png)

## 数据定义

在SQL server中有两类数据库：系统数据库和用户数据库。

系统数据库存储关于SQL server的系统信息，它们是SQL server管理数据库的依据。如果系统数据库遭到破坏，SQL server将不能正常启动。在安装SQL server后，系统将创建4个可见的系统数据库：`master`、`model`、`msdb`、`tempdb`。

1. master数据库包含两了SQL server诸如登录账号、系统配置、数据库位置及数据库错误信息等，用于控制用户数据库和SQL server的运行。
2. model数据库为新创建的数据库提供模板。
3. msdb数据库为“SQL server代理”调度信息和作业记录提供存储空间。
4. tempdb数据库为临时表和临时存储过程提供存储空间，所有与系统连接的用户的临时表和临时存储过程都存储于该数据库中。

每个系统数据库都包含主数据文件和主日志文件。扩展名分别为`.mdf`和`.ldf`，例如master数据库的两个文件分别为`master.mdf`和`master.ldf`。

**以下为创建数据库的语法：**

```sql
create database 数据库名
on [primary]
(
<数据文件参数>[1,…,n][<文件组参数>]
)
[LOG ON]
(
<日志文件参数>[1,…,n]
)

-- 例如：

create database studb
on primary -- 主文件组，可省略
(
	name='studb_data',	-- 主数据文件的逻辑名
    filename='D:\studb_data.mdf',	--主数据文件的物理名
    size=5MB,	--主数据文件初始大小
    maxsize=100MB,	--主数据文件增长的最大值，数据文件的具体描述
    filegrowth=15%	--主数据文件的增长率，数据文件的具体描述
)
log on
(
	name='studb_log',
    filename='D:\stubd_log.ldf',
    size=2MB,	--日志文件的具体描述
    filegrowth=1MB	--日志文件的具体描述
)
GO
```

**SQL的数据定义语句：**

| 操作对象 | 创建         | 删除       | 修改        |
| -------- | ------------ | ---------- | ----------- |
| 表       | CREATE TABLE | DROP TABLE | ALTER TABLE |
| 视图     | CREATE VIEW  | DROP VIEW  |             |
| 索引     | CREATE INDEX | DROP INDEX |             |

##### 创建表

```sql
CREATE TABLE <表名>
	(<列名><数据类型>[<列级完整性约束条件>]
    [,<列名><数据类型>[<列级完整性约束条件>]]…
    [,<表级完整性约束条件>])
    
    /*
    常用完整性约束：
    主码约束：PRIMARY KEY
    唯一性约束：UNIQUE
    非空值约束：NOT NULL
    参照完整性约束：foreign key,references
    */
    
--例如：

create table Student(
   Stu_no       char(8) ,
   Stu_name     varchar(10) not null,
   Stu_age       int,
   Stu_sex       char(2),
   Stu_dept      varchar(20),
   CONSTRAINT pk_sno PRIMARY KEY (Stu_no));

```

###### 用户表属性及建表代码：

| 属性名称   | 类型     | 长度             | 备注           |
| ---------- | -------- | ---------------- | -------------- |
| UID        | int      | 自动编码，标识列 | 用户ID         |
| Uname      | nvarchar | 20               | 用户名         |
| Upassword  | varchar  | 10               | 用户密码       |
| Usex       | bit      | 1                | 用户性别       |
| Ubirthday  | datetime |                  | 生日           |
| Uclass     | int      |                  | 级别(几星级)   |
| Uremark    | nvarchar | 50               | 备注           |
| Ustate     | int      |                  | 状态(是否禁言) |
| CreateTime | datetime |                  | 注册时间       |
| Upoint     | int      |                  | 积分(点数)     |

```sql
CREATE TABLE Users
(
	UID INT IDENTITY (1,1),	--自动编码，标识列
    Uname VARCHAR(15) NOT NULL,	--昵称
    Upassword VARCHAR(10),	--密码
    Uemail VARCHAR(20),	--邮箱
    Ubirthday DATETIME,	--生日
    Usex BIT NOT NULL,	--性别
    Uclass INT,	--级别(几星级)
    Uremark VARCHAR(20),	--备注
    UregDate DATETIME NOT NULL,	--注册日期
    Ustate INT NULL,	--状态(是否禁言等)
    Upoint INT NULL	--积分(点数)
)
```

##### 对表增加约束：

```sql
ALTER TABLE 表名
	ADD CONSTRAINT 约束名 约束类型 具体约束说明

--例如：

--增加主键约束：
ALTER TABLE Users
	ADD CONSTRAINT PK_UID PRIMARY KEY(UID)
--设置初始密码为8888
ALTER TABLE Users
	ADD CONSTRAINT DF_Upassword
		DEFAULT('8888') FOR Upassword
--注册日期默认为当前日期
ALTER TABLE Users
	ADD CONSTRAINT DF_UregDate
		DEFAULT(getDate()) FOR UregDate		
--邮件必须包含‘@’字符
ALTER TABLE Users
	ADD CONSTRAINT CK_Uemail
		CHECK (Uemail LIKE '%@%')
--密码至少六位
ALTER TABLE Users
	ADD CONSTRAINT CK_Upassword
		CHECK (LEN(Upassword)>=6)
```

- 约束名的取名规则推荐采用：约束类型_约束字段

  主键(Primary Key)约束：如PK_stuNo

  唯一(Unique Key)约束：如UQ_stu_ID

  默认(Default Key)约束：如DF_stuAddress

  检查(Check Key)约束：如CK_stuAge

  外键(Foreign Key)约束：如FK_stuNo

##### 修改基本表

```sql
ALTER TABLE <表名>
	[ADD<新列名><数据类型>[完整性约束]]
	[DROP<完整性约束名>]
	[ALTER COLUMN<列名><数据类型>]
```

1. 插入数据

   ```sql
   INSERT INTO Student VALUES('20186101','李勇','男',18,'计算机科学与技术','15359028970')
   ```

2. 修改数据

   ```sql
   UPDATE TABLENAME	--指定要更新的表
   SET COLUMN = VALUE[,COLUMN = VALUE]	--指定所有要改变数值的列和指定新值
   [WHERE CONDITION]	--用于确定哪些行要进行修改
   --例如：
   update student
   set stu_age=18
   where stu_no='20186101';
   ```

3. 删除数据

   1. 删除某一记录

      ```sql
      delete from sc
      where cou_no='a01';
      ```

   2. 删除所有记录并释放表所占空间

      ```sql
      truncate table sc;
      ```

##### 查询

语句格式：

```sql
SELECT [ALL|DISTINCT]<目标列表达式>
					[,<目标列表达式>]…
FROM <表名或视图名>[,<表名或视图名>]…
[WHERE <条件表达式>]
[GROUP BY<列名1>
[HAVING <条件表达式>]]
[ORDER BY <列名2> [ASC|DESC]]
```

1. 单表查询
   - 查询仅涉及一个表，是一种最简单的查询操作
     1. 选择表中的若干列(select…)
     2. 选择表中的若干元组(where…)
     3. 对查询结果排序(order by …)
     4. 使用聚合函数(avg(),max(),min(),count(),sum())
     5. 对查询结果分组(group by)
2. 连接查询
3. 嵌套查询
4. 集合查询

#### SQL server中的函数

字符串函数

| 函数名    | 描述                                               | 举例                                                |
| --------- | -------------------------------------------------- | --------------------------------------------------- |
| CHARINDEX | 用来寻找一个指定的字符串在另一个字符串中的起始位置 | SELECT CHARINDEX(‘ACCP’,’MY AccpCourse’,1);返回：4  |
| LEN       | 返回传递给它的字符串长度                           | SELECT LEN(‘SQL Server课程’);返回：12               |
| LOWER     | 把传递给它的字符串转换为小写                       | SELECT LOWER(‘SQL Server课程’);返回：sql server课程 |
| UPPER     | 把传递给它的字符串转换为大写                       | SELECT UPPER(‘SQL Server课程’);返回：SQL SERVER课程 |
| LTRIM     | 清除字符左边的空格                                 | SELECET LTRIM(‘ a ’)；返回：a (后面的空格保留)      |

日期函数

| 函数名   | 描述                                     | 举例                                                         |
| -------- | ---------------------------------------- | ------------------------------------------------------------ |
| GETDATE  | 取得当前的系统日期                       | SELECT GETDATE()返回今天的日期                               |
| DATEADD  | 将指定的数值添加到指定的日期部分后的日期 | SELECT DATEADD(mm,4,’2019-3-27’)返回2019-03-27 00:00:00.0000 |
| DATEDIFF | 两个日期之间的指定日期部分的区别         | SELECT DATEDIFF(mm,’2019-3-27’,’2019-1-1’) 返回-2            |
| DATENAME | 日期中指定日期部分的字符串形式           | SELECT DATENAME(dw,’2019-3-27’)返回：星期五                  |
| DATEPART | 日其中指定日期部分的整数形式             | SELECT DATEPART(day,’2019-3-27)返回：27                      |

| 日期简写     | 简写 | 值                   |
| ------------ | ---- | -------------------- |
| year         | yy   | 1753-9999            |
| quarter      | qq   | 1-4                  |
| month        | mm   | 1-12                 |
| day for year | dy   | 1-366                |
| day          | dd   | 1-31                 |
| week         | wk   | 1-53                 |
| weekday      | dw   | 1-7(Sunday-Saturday) |
| hour         | hh   | 0-23                 |
| minute       | mi   | 0-59                 |
| second       | ss   | 0-59                 |
| milisecond   | ms   | 0-999                |

数学函数

| 函数名  | 描述                                            | 举例                                     |
| ------- | ----------------------------------------------- | ---------------------------------------- |
| ABS     | 取数值表达式的绝对值                            | SELECT ABS(-43)返回：43                  |
| FLOOR   | 取小于或等于指定表达式的最大整数                | SELECT FLOOR(43.5)返回：43               |
| CEILING | 返回大于或等于所给数字表达式的最小整数          | SELECT CEILING(43.5)返回：44             |
| POWER   | 取数值表达式的幂值                              | SELECT POWER(5,2)返回：25                |
| ROUND   | 将数值表达式四舍五入为指定精度                  | SELECT ROUND(43.543,1)返回：43.5         |
| RAND    | 返回一个介于0到1(不包括0和1)之间的伪随机float值 | SELECT FLOOR(RAND()*100)返回：0-99的整数 |
| SQRT    | 取浮点表达式的平方根                            | SELECT SQRT(9)返回：3                    |

系统函数

| 函数名       | 描述                           | 举例                                              |
| ------------ | ------------------------------ | ------------------------------------------------- |
| CONVERT      | 用来转变数据类型               | SELECT CONVERT(VARCHAR(5),12345)返回：字符串12345 |
| CURRENT_USER | 返回当前用户的名字             | SELECT CURRENT_USER返回：你登录的用户名           |
| DATALENGTH   | 返回用于指定表达式的字节数     | SELECT DATALENGTH返回：4                          |
| HOST_NAME    | 返回当前用户所登录的计算机名字 | SELECT HOST_NAME()返回：你登录的计算机的名字      |
| SYSTEM_USER  | 返回当前所登录的用户名称       | SELECT SYSTEM_USER返回：你当前所登录的用户名      |
| USER_NAME    | 从给定的用户ID返回用户名       | SELECT USER_NAME(1)返回：从任意数据库中返回“dbo”  |

常用的查询条件

| 查询条件T | 谓词                        |
| --------- | --------------------------- |
| 比较      | >,=,<,>=,!=,!>,!<;NOT       |
| 确定范围  | BETWEEN AND,NOT BETWEEN AND |
| 确定集合  | IN,NOT IN                   |
| 字符匹配  | LIKE,NOT LIKE               |
| 空值      | IS NULL,IS NOT NULL         |
| 多重条件  | AND,OR                      |

