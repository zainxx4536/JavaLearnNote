## MySQL_note

### 1. 认知MySQL

```mysql
-- 在 cmd 中启动 MySQL
-- mysql [-h 127.0.0.1] [-P 3306] -u root -p
-- 这里 root 是用户名
-- -h : MySQL服务所在的主机IP,默认是本机
-- -P : MySQL服务端口号， 默认3306
-- -u : MySQL数据库用户名,初始用户是 root
-- -p ： MySQL数据库用户名对应的密码

-- 关系型数据库：用表保存数据，表由字段和记录组成，表之间可建立关系。

-- MySQL 对象层级通常表示为：实例 -> 数据库(schema) -> 表 -> 列/行。
-- 用户与权限由实例独立管理，再通过 GRANT 授予对数据库或对象的访问权，不是数据库的上级对象。

-- SQL语句可以单行或多行书写，以分号结尾
-- SQL语句可以使用空格/缩进来增强语句的可读性
-- SQL 关键字通常不区分大小写；表名大小写受操作系统和 lower_case_table_names 影响，字符串比较受字符集与排序规则影响。
-- 单行注释：-- 后必须有空白字符，也可使用 #
-- 多行注释：/* 注释内容 */


-- 数据类型
-- 数值类型：保存整数或小数。TINYINT、INT、BIGINT、FLOAT、DOUBLE、DECIMAL
-- 字符串类型：保存文本。CHAR、VARCHAR、TEXT
-- 日期时间类型：保存日期和时间。DATE、TIME、DATETIME、TIMESTAMP
-- unsigned：无符号数。扩大非负整数可表示范围。
-- varchar(n)：可变长度字符串。按实际内容占用空间，n表示最大长度。
```

### 2. SQL语句

SQL 语句分为 : DDL / DML / DQL / DCL 语句,每种语句对应不同部分的操作

#### 2.1 DDL 语句

```mysql
-- DDL-数据定义语句
-- 对数据库的操作 :
-- 展示登录用户权限内的所有数据库
show databases;

-- 查询当前操作的数据库(函数)
select database();

-- 创建一个itcast数据库, 使用数据库默认的字符集(utf8mb4)
create database itcast;

create database if not exists itcast;

-- 创建一个itheima数据库，并且指定字符集
create database itheima default charset utf8mb4;

create database if not exists itheima default charset utf8mb4;

-- 删除数据库
drop database itcast;

drop database if exists itcast;

-- 切换数据库
use itheima;


-- 对数据库中的表的操作 :
-- 展示数据库中的所有表
show tables;

-- 创建表
create table tb_user
(
    id     int comment '编号',
    name   varchar(50) comment '姓名',
    age    int comment '年龄',
    gender varchar(1) comment '性别'
) comment '用户表';

-- 查看指定表的结构(指定表的字段，字段的类型、是否可以为NULL，是否存在默认值等信息)
desc tb_user;

-- 展示指定表的建表语句
show create table tb_user;

-- 修改表名
alter table tb_user rename to emp;

rename table tb_user to emp;

-- 删除表
drop table emp;

drop table if exists emp;

-- 删除并重新创建指定表,相当于清空表中数据
truncate  table emp;

-- 表操作-修改字段
-- 添加字段
-- 为tb_uesr表增加一个新的字段'昵称'为nickname，类型为varchar(20)
alter table tb_user add nickname varchar(20) comment '昵称';

-- 修改字段数据类型
-- 将tb_user表的nickname数据类型修改为varchar(30)
alter table tb_user modify nickname varchar(30);

-- 修改字段名以及字段数据类型
-- 将tb_user表的nickname字段修改为username，类型为varchar(30)
alter table tb_user change nickname username varchar(30) comment '用户名';

-- 删除字段
-- 将tb_user表的字段username删除
alter table tb_user drop username;

-- 以上操作均可在 MySQL 图形化界面 DataGrip 中使用图形操作完成
```

#### 2.2 DML 语句

```mysql
-- DML-数据操作语言
-- 给表中字段添加数据
insert into employee(id, workno, name, gender, age, idcard, entrydate)
values (1, '1', '张三', '男', 23, '123456789012345678', '2026-04-23');

-- 查询建表语句,即认知表的具体结构
desc employee;

select *
from employee;

insert into employee
values (2, '2', '李四', '男', 24, '123456789012345679', '2026-04-22');

insert into employee
values (3, '3', '王五', '男', 25, '123456789012345679', '2026-04-22'),
       (4, '4', '赵六', '男', 26, '123456789012345679', '2026-04-22');


-- 修改字段数据
update employee
set name = '张大三'
where id = 1;

update employee
set name = '李小四'
where id = 2;

update employee
set entrydate = '2008-01-01';

update employee
set gender = '女'
where id = 1;

-- 删除数据
delete
from employee
where gender = '女';

delete
from employee;


create table employee
(
    id          int comment '编号',
    workno      varchar(10) comment '工号',
    name        varchar(10) comment '姓名',
    gender      varchar(1) comment '性别',
    age         tinyint unsigned comment '年龄',
    idcard      varchar(18) comment '身份证',
    workaddress varchar(50) comment '工作地址',
    entrydate   date comment '入职时间'
) comment '员工表';

insert into employee
values (1, '1', '柳岩666', '女', 20, '123456789012345678', '北京', '2000-01-01');

INSERT INTO employee (id, workno, name, gender, age, idcard, workaddress, entrydate)
VALUES (2, '2', '张无忌', '男', 18, '123456789012345670', '北京', '2005-09-01');

INSERT INTO employee (id, workno, name, gender, age, idcard, workaddress, entrydate)
VALUES (3, '3', '韦一笑', '男', 38, '123456789712345670', '上海', '2005-08-01');
```

#### 2.3 DQL 语句

```mysql
-- DQL-数据查询语句
-- 查询指定字段
select name, workno, age
from employee;

-- 查询所有字段,可以用通配符 *,但尽量不用,写清楚具体查询的内容,方便理解
select *
from employee;

select id,
       workno,
       name,
       gender,
       age,
       idcard,
       workaddress,
       entrydate
from employee;


-- 查询所有员工的工作地址,同时起个别名,方便展示
select workaddress
from employee;

select workaddress as '工作地址'
from employee;


-- 查询员工的上班地址,同时去重
select distinct workaddress as '工作地址'
from employee;

-- DQL-条件查询
-- 查询年龄等于 88 的员工
select *
from employee
where age = 88;

-- 查询年龄小于 20 的员工信息
select *
from employee
where age < 20;

-- 查询年龄小于等于 20 的员工信息
select *
from employee
where age <= 20;

-- 查询没有身份证号的员工信息
select *
from employee
where idcard is null;

-- 查询有身份证号的员工信息
select *
from employee
where idcard is not null;

-- 查询年龄不等于 88 的员工信息
select *
from employee
where age != 88;

select *
from employee
where age <> 88;

-- 查询年龄在15岁(包含) 到 20岁(包含)之间的员工信息
select *
from employee
where age >= 15
  and age <= 20;

select *
from employee
where age >= 15 && age <= 20;

select *
from employee
where age between 15 and 20;

-- 查询性别为 女 且年龄小于 25岁的员工信息
select *
from employee
where gender = '女'
  and age < 25;

-- 查询年龄等于18 或 20 或 40 的员工信息
select *
from employee
where age = 18
   or age = 20
   or age = 40;

select *
from employee
where age in (18, 20, 40);

-- 查询姓名为两个字的员工信息 _匹配一个字符, %匹配任意个数字符
select *
from employee
where name like '__';

-- 查询身份证号最后一位是X的员工信息
select *
from employee
where idcard like '%X';


-- DQL-聚合函数
-- 统计该企业员工数量,null 值不参与统计
select count(*)
from employee;

select count(id)
from employee;

-- 统计该企业员工的平均年龄
select avg(age)
from employee;

-- 统计该企业员工的最大年龄
select max(age)
from employee;

-- 统计该企业员工的最小年龄
select min(age)
from employee;

-- 统计西安地区员工的年龄之和
select sum(age)
from employee
where workaddress = '西安';



-- DQL-分组查询
-- 根据性别分组 , 统计男性员工 和 女性员工的数量
select gender, count(*)
from employee
group by gender;

-- 根据性别分组 , 统计男性员工 和 女性员工的平均年龄
select gender, avg(age)
from employee
group by gender;

-- 查询年龄小于45的员工 , 并根据工作地址分组 , 获取员工数量大于等于3的工作地址
--     执行顺序 : where -> 聚合函数 -> having
--     分组之后,查询的字段一般为聚合函数和分组字段,查询其他字段无意义
select workaddress, count(*) addresscount
from employee
where age < 45
group by workaddress
having addresscount >= 3;

-- 统计各个工作地址上班的男性及女性员工的数量
--     分成两组
select workaddress, gender, count(*)
from employee
group by workaddress, gender;


-- DQL-排序查询
-- 根据年龄对公司的员工进行升序排序,升序是默认的, asc 可以省略
select *
from employee
order by age asc;

-- 根据入职时间, 对员工进行降序排序
select *
from employee
order by entrydate desc;

-- 根据年龄对公司的员工进行升序排序 , 年龄相同 , 再按照入职时间进行降序排序
select *
from employee
order by age asc, entrydate desc;


-- DQL-分页查询
-- 查询第1页员工数据, 每页展示10条记录
--      第一页的起始索引 0 可以省略
select *
from employee
limit 0,10;

select *
from employee
limit 10;

-- 查询第2页员工数据, 每页展示10条记录 --------> (页码-1)*页展示记录数
select *
from employee
limit 10,10;


-- 练习
-- 查询年龄为20,21,22,23岁的员工信息
select *
from employee
where age in (20, 21, 22, 23);

-- 查询性别为 男 ，并且年龄在 20-40 岁(含)以内的姓名为三个字的员工
select *
from employee
where gender = '男'
  and age >= 20
  and age <= 40
  and name like '___';

-- 统计员工表中, 年龄小于60岁的 , 男性员工和女性员工的人数
select gender, count(*)
from employee
where age < 60
group by gender;

-- 查询所有年龄小于等于35岁员工的姓名和年龄，并对查询结果按年龄升序排序，如果年龄相同按入职时间降序排序
select name, age
from employee
where age <= 35
order by age asc, entrydate desc;

-- 查询性别为男，且年龄在20-40 岁(含)以内的前5个员工信息，对查询的结果按年龄升序排序，年龄相同按入职时间升序排序
select *
from employee
where gender = '男'
  and age between 20 and 40
order by age asc, entrydate asc
limit 5;

-- DQL-编写顺序
-- select -> from -> where -> group by -> having -> order by -> limit

-- DQL-执行顺序
-- from -> where -> group by -> having -> select -> order by -> limit
```

#### 2.4 DCL 语句

```mysql
-- DCL-数据控制语言
-- 用户管理 :
-- 创建用户itcast, 只能够在当前主机localhost访问, 密码123456;
create user 'itcast'@'localhost' identified by '123456';

-- 创建用户itheima, 可以在任意主机访问该数据库, 密码123456;
create user 'itheima'@'%' identified by '123456';

-- 修改用户 itheima 的访问密码。MySQL 8.4 默认认证插件已不是 mysql_native_password，除兼容旧客户端外不要显式指定它。
alter user 'itheima'@'%' identified by 'Replace-With-A-Strong-Password!';

-- 删除 itcast@localhost 用户
drop user 'itcast'@'localhost';


-- 权限控制 :
-- 查询 'heima'@'%' 用户的权限
show grants for 'itheima'@'%';
-- GRANT USAGE ON *.* TO `itheima`@`%` 即无权限,仅仅只能连接和登录该用户


-- 授予 'itheima'@'%' 用户对数据库xxdatabase的所有操作权限
grant all on xxdatabase.* to 'itheima'@'%';
-- 超级管理员,授予itheima所有数据库的所有操作权限
grant all on *.* to 'itheima'@'%';


-- 撤销 'itheima'@'%' 用户的xxdatabase数据库的所有权限
revoke all on xxdatabase.* from 'itheima'@'%';
```

### 3. 函数

#### 3.1 字符串函数

```mysql
-- 字符串函数
-- select 函数(参数)

-- concat 拼接字符串
select concat('Hello', 'MySQL');

-- lower 将字符串变为全小写
select lower('Hello MySQL');

-- upper 将字符串变为全大写
select upper('Hello MySQL');

-- lpad(str,n,pad) 左填充，用字符串pad对str的左边进行填充，达到n个字符串长度
select lpad('MySQL', 12, 'Hello');

-- rpad(str,n,pad) 右填充，用字符串pad对str的右边进行填充，达到n个字符串长度
select rpad('Hello', 13, 'MySQL');

-- trim(str) 去掉字符串头部和尾部的空格
select trim('     Hello MySQL             ');

-- substring(str,start,len) 返回从字符串str从start位置起的len个长度的字符串
select substring('Hello MySQL', 1, 5);

-- char_length(str) 返回字符数；length(str) 返回字节数
select char_length('hello world!');

-- 练习,使用xxdatabase数据库下的employee表
-- 由于业务需求变更，企业员工的工号，统一为5位数，目前不足5位数的全部在前面补0
-- 比如： 1号员工的工号应该为00001
use xxdatabase;

select *
from employee;

desc employee;

update employee
set workno = lpad(workno, 5, '0');
```

#### 3.2 数值函数

```mysql
-- 数值函数

-- ceil 向上取整
select ceil(1.1);

-- floor 向下取整
select floor(1.9);

-- mod(x,y) 取 x/y 的模(余数)
select mod(2, 3);

-- rand 获得 0~1 之间随机数
select rand();

-- round(x,y) 对 x 进行四舍五入,留 y 个小数位
select round(1.23541, 2);


-- 练习：生成固定六位数字。该写法不适合安全验证码；安全场景应由密码学安全随机源生成，并设置有效期和尝试次数。
select lpad(floor(rand() * 1000000), 6, '0');
```

#### 3.3 日期函数

```mysql
-- 日期函数
-- curdate 当前日期
select curdate();

-- curtime 当前时间
select curtime();

-- now 当前日期和时间
select now();

-- year , month , day
select year(now());

select month(now());

select day(now());

-- date_add 给某一时间加上指定时间
select date_add(now(), interval 20 year);

select date_add(now(), interval 20 month);

select date_add(now(), interval 20 day);

-- datediff 计算两个时间的间隔时间,前面的时间减后面的时间
select datediff(now(), '2002-12-07');


-- 练习,使用xxdatabase数据库下的employee表
-- 查询所有员工的入职天数,并根据入职天数倒序排列
select id, name, datediff(now(), entrydate) as 'entrydays'
from employee
order by entrydays desc;
```

#### 3.4 流程函数

```mysql
-- 流程函数
-- if(value , t , f) 如果value为true，则返回t，否则返回f
select if(true, 'OK', 'NOT OK');

select if(false, 'OK', 'NOT OK');

-- ifnull(value1 , value2) 如果value1不为空，返回value1，否则返回value2
select ifnull('OK', 'Default');

select ifnull('', 'Default'); -- 空的字符串不等于 null

select ifnull(null, 'Default');

-- case when (value1) then (result1) ... else (default) end
-- 如果val1为true，返回res1，... 否则返回default默认值

-- case (expr) when (value1) then (result1) ... else (default) end
-- 如果expr的值等于val1，返回res1，... 否则返回default默认值

-- 练习1,查询员工姓名和工作地址 (北京/上海 ----> 一线城市 , 其他 ----> 二线城市)
select name,
       case when workaddress in ('北京', '上海') then '一线城市' else '二线城市' end as '工作地址'
from employee;

select name,
       case workaddress when '北京' then '一线城市' when '上海' then '一线城市' else '二线城市' end as '工作地址'
from employee;

select name,
       case when workaddress = '北京' or workaddress = '上海' then '一线城市' else '二线城市' end as '工作地址'
from employee;

-- 练习2,统计score表中的学生成绩,>=85,显示优秀;>=60,显示及格;否则,显示不及格.
select name,
       case when math >= 85 then '优秀' when math >= 60 then '及格' else '不及格' end       as '数学成绩',
       case when english >= 85 then '优秀' when english >= 60 then '及格' else '不及格' end as '英语成绩',
       case when chinese >= 85 then '优秀' when chinese >= 60 then '及格' else '不及格' end as '语文成绩'
from score;



create table score
(
    id      int comment 'ID',
    name    varchar(20) comment '名字',
    math    int comment '数学',
    english int comment '英语',
    chinese int comment '语文'
) comment '学生成绩表';

insert into score
values (1, 'Tom', 67, 88, 95),
       (2, 'Rose', 23, 66, 90),
       (3, 'Jack', 56, 98, 76);
```

### 4. 约束

```mysql
-- 约束
-- 概念：约束是作用于表中字段上的规则，用于限制存储在表中的数据
-- 目的：保证数据库中数据的正确、有效性和完整性

-- 多个约束之间使用空格隔开即可

-- not null 非空约束
-- unique 唯一约束(不重复)
-- primary key 主键约束,一行数据的唯一标识,要求非空且唯一
-- default 默认约束,指定默认值
-- check 检查约束，保证字段满足条件；MySQL 从 8.0.16 起才真正强制执行 CHECK。
-- auto_increment 自增约束,让数据自动依次增加(从 1 开始)

use xxdatabase;

create table user
(
    id     int primary key auto_increment comment '主键',
    name   varchar(10) not null unique comment '姓名',
    age    int check ( age > 0 and age <= 120 ) comment '年龄',
    status char(1) default '1' comment '状态',
    gender char(1) comment '性别'
) comment '用户表';

-- 插入数据时,不满足约束条件会插入失败,但是会浪费一个主键名额

insert into user(name, age, status, gender)
values ('Tom1', 19, '1', '男'),
       ('Tom2', 25, '0', '男');

insert into user(name, age, status, gender)
values ('Tom3', 19, '1', '男');

insert into user(name, age, status, gender)
values (null, 19, '1', '男');

insert into user(name, age, status, gender)
values ('Tom3', 19, '1', '男');

insert into user(name, age, status, gender)
values ('Tom4', 80, '1', '男');

insert into user(name, age, status, gender)
values ('Tom5', -1, '1', '男');

insert into user(name, age, status, gender)
values ('Tom5', 121, '1', '男');

insert into user(name, age, gender)
values ('Tom5', 120, '男');



create table emp
(
    id        int auto_increment comment 'ID' primary key,
    name      varchar(50) not null comment '姓名',
    age       int comment '年龄',
    job       varchar(20) comment '职位',
    salary    int comment '薪资',
    entrydate date comment '入职时间',
    managerid int comment '直属领导ID',
    dept_id   int comment '部门ID'
) comment '员工表';

create table dept
(
    id   int auto_increment comment 'ID' primary key,
    name varchar(50) not null comment '部门名称'
) comment '部门表';

INSERT INTO dept (id, name)
VALUES (1, '研发部'),
       (2, '市场部'),
       (3, '财务部'),
       (4, '销售部'),
       (5, '总经办');

INSERT INTO emp (id, name, age, job, salary, entrydate, managerid, dept_id)
VALUES (1, '金庸', 66, '总裁', 20000, '2000-01-01', null, 5),
       (2, '张无忌', 20, '项目经理', 12500, '2005-12-05', 1, 1),
       (3, '杨逍', 33, '开发', 8400, '2000-11-03', 2, 1),
       (4, '韦一笑', 48, '开发', 11000, '2002-02-05', 2, 1),
       (5, '常遇春', 43, '开发', 10500, '2004-09-07', 3, 1),
       (6, '小昭', 19, '程序员鼓励师', 6600, '2004-10-12', 2, 1);



-- foreign key 外键约束,用来让两张表的数据之间建立连接，保证数据的一致性和完整性
-- 为 emp 表的 dept_id 字段添加外键约束,关联 dept 表的主键 id
-- 这里属于多表查询的一对多关系,在多的一方建立外键(子表),去关联一的一方的主键(父表)
-- 可以在创建表时规定表结构,也可以另外增加或修改约束,一般额外添加外键约束
alter table emp
    add constraint fk_emp_dept_id foreign key (dept_id) references dept (id);

-- 删除外键 fk_emp_dept_id
alter table emp
    drop foreign key fk_emp_dept_id;


-- 外键删除更新行为
-- 在添加外键后,在去删除父表数据时产生的约束行为，我们就称为删除/更新行为
-- no action 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，
-- 如果有,则不允许删除/更新。 (与 RESTRICT 一致) 这是默认行为,无需设置
-- restrict 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，
-- 如果有,则不允许删除/更新。 (与 NO ACTION 一致) 这是默认行为,无需设置
-- cascade 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，
-- 如果有，则也删除/更新外键在子表中的记录。
-- set null 当在父表中删除对应记录时，首先检查该记录是否有对应外键，
-- 如果有,则设置子表中该外键值为null（这要求该外键允许取null）。
-- set default 父表有变更时，子表将外键列设置成一个默认的值 (Innodb不支持,所以这里不用)

-- 以下两种外键行为是互斥示例，同一个 dept_id 不要同时添加这两个约束。
-- 方案一：级联更新、级联删除
alter table emp
    add constraint fk_emp_dept_id_cascade foreign key (dept_id) references dept (id) on update cascade on delete cascade;

-- 方案二：更新或删除父记录时将外键设为 NULL（dept_id 必须允许 NULL）。使用本方案前先删除方案一的约束。
alter table emp
    add constraint fk_emp_dept_id_set_null foreign key (dept_id) references dept (id) on update set null on delete set null;
```

### 5.多表查询

```mysql
-- 多表查询

-- 多表关系 : 项目开发中，在进行数据库表结构设计时，会根据业务需求及业务模块之间的关系，
-- 		分析并设计表结构，由于业务之间相互关联，所以各个表结构之间也存在着各种联系，基本上分为三种：
-- 		一对多(多对一) ; 多对多 ; 一对一

-- 一对多 :   一个部门对应多个员工，一个员工对应一个部门
-- 		实现 : 在多的一方建立外键，指向一的一方的主键(见 console_4)
-- 多对多 :  一个学生可以选修多门课程，一门课程也可以供多个学生选择
-- 		实现 : 建立第三张中间表，中间表至少包含两个外键，分别关联两方主键
-- 一对一 : 多用于单表拆分，将一张表的基础字段放在一张表中，其他详情字段放在另一张表中，以提升操作效率
-- 		实现: 在任意一方加入外键，关联另外一方的主键，并且设置外键为唯一的(UNIQUE)


-- 多对多实现演示
create table student
(
    id   int auto_increment primary key comment '主键ID',
    name varchar(10) comment '姓名',
    no   varchar(10) comment '学号'
) comment '学生表';

insert into student
values (null, '黛绮丝', '2000100101'),
       (null, '谢逊', '2000100102'),
       (null, '殷天正', '2000100103'),
       (null, '韦一笑', '2000100104');

create table course
(
    id   int auto_increment primary key comment '主键ID',
    name varchar(10) comment '课程名称'
) comment '课程表';

insert into course
values (null, 'Java'),
       (null, 'PHP'),
       (null, 'MySQL'),
       (null, 'Hadoop');


-- 建立中间表,管理多对多的关系
create table student_course
(
    id        int auto_increment primary key comment '主键',
    studentID int not null comment '学生ID',
    courseID  int not null comment '课程ID',
    constraint fk_studentID foreign key (studentID) references student (id),
    constraint fk_courseID foreign key (courseID) references course (id)
) comment '学生课程中间表';

insert into student_course
values (null, 1, 1),
       (null, 1, 2),
       (null, 1, 3),
       (null, 2, 2),
       (null, 2, 3),
       (null, 3, 4);


-- 一对一实现演示
create table tb_user
(
    id     int auto_increment primary key comment '主键ID',
    name   varchar(10) comment '姓名',
    age    int comment '年龄',
    gender char(1) comment '1: 男 , 2: 女',
    phone  char(11) comment '手机号'
) comment '用户基本信息表';

create table tb_user_edu
(
    id            int auto_increment primary key comment '主键ID',
    degree        varchar(20) comment '学历',
    major         varchar(50) comment '专业',
    primaryschool varchar(50) comment '小学',
    middleschool  varchar(50) comment '中学',
    university    varchar(50) comment '大学',
    userid        int unique comment '用户ID',
    -- 这个外键放在哪张表中都可以
    constraint fk_userid foreign key (userid) references tb_user (id)
) comment '用户教育信息表';

insert into tb_user(id, name, age, gender, phone)
values (null, '黄渤', 45, '1', '18800001111'),
       (null, '冰冰', 35, '2', '18800002222'),
       (null, '码云', 55, '1', '18800008888'),
       (null, '李彦宏', 50, '1', '18800009999');

insert into tb_user_edu(id, degree, major, primaryschool, middleschool, university, userid)
values (null, '本科', '舞蹈', '静安区第一小学', '静安区第一中学', '北京舞蹈学院', 1),
       (null, '硕士', '表演', '朝阳区第一小学', '朝阳区第一中学', '北京电影学院', 2),
       (null, '本科', '英语', '杭州市第一小学', '杭州市第一中学', '杭州师范大学', 3),
       (null, '本科', '应用数学', '阳泉第一小学', '阳泉区第一中学', '清华大学', 4);


-- 多表查询演示
create table dept
(
    id   int auto_increment comment 'ID' primary key,
    name varchar(50) not null comment '部门名称'
) comment '部门表';
INSERT INTO dept (id, name)
VALUES (1, '研发部'),
       (2, '市场部'),
       (3, '财务部'),
       (4, '销售部'),
       (5, '总经办'),
       (6, '人事部');

create table emp
(
    id        int auto_increment comment 'ID' primary key,
    name      varchar(50) not null comment '姓名',
    age       int comment '年龄',
    job       varchar(20) comment '职位',
    salary    int comment '薪资',
    entrydate date comment '入职时间',
    managerid int comment '直属领导ID',
    dept_id   int comment '部门ID'
) comment '员工表';

-- 添加外键
alter table emp
    add constraint fk_emp_dept_id foreign key (dept_id) references dept (id);

INSERT INTO emp (id, name, age, job, salary, entrydate, managerid, dept_id)
VALUES (1, '金庸', 66, '总裁', 20000, '2000-01-01', null, 5),
       (2, '张无忌', 20, '项目经理', 12500, '2005-12-05', 1, 1),
       (3, '杨逍', 33, '开发', 8400, '2000-11-03', 2, 1),
       (4, '韦一笑', 48, '开发', 11000, '2002-02-05', 2, 1),
       (5, '常遇春', 43, '开发', 10500, '2004-09-07', 3, 1),
       (6, '小昭', 19, '程序员鼓励师', 6600, '2004-10-12', 2, 1),
       (7, '灭绝', 60, '财务总监', 8500, '2002-09-12', 1, 3),
       (8, '周芷若', 19, '会计', 48000, '2006-06-02', 7, 3),
       (9, '丁敏君', 23, '出纳', 5250, '2009-05-13', 7, 3),
       (10, '赵敏', 20, '市场部总监', 12500, '2004-10-12', 1, 2),
       (11, '鹿杖客', 56, '职员', 3750, '2006-10-03', 10, 2),
       (12, '鹤笔翁', 19, '职员', 3750, '2007-05-09', 10, 2),
       (13, '方东白', 19, '职员', 5500, '2009-02-12', 10, 2),
       (14, '张三丰', 88, '销售总监', 14000, '2004-10-12', 1, 4),
       (15, '俞莲舟', 38, '销售', 4600, '2004-10-12', 14, 4),
       (16, '宋远桥', 40, '销售', 4600, '2004-10-12', 14, 4),
       (17, '陈友谅', 42, null, 2000, '2011-10-12', 1, null);

-- 笛卡尔积 : 指在数学中，两个集合 A 集合和 B 集合的所有组合情况
-- 多表查询首先要消除笛卡尔积 : 添加条件,说明外键对应关系
select * from emp, dept where emp.dept_id = dept.id;


-- 多表查询分类 : 连接查询 ; 联合查询 ; 子查询
```

#### 5.1 连接查询

```mysql
-- 连接查询 :
-- 		内连接 : 相当于查询A、B两表交集部分数据
-- 		外连接 : 左外连接,查询左表所有数据，以及两张表交集部分数据
-- 				右外连接,查询右表所有数据，以及两张表交集部分数据
-- 		自连接 : 当前表与自身的连接查询，自连接必须使用表别名(将一张表当做两张表)

-- 内连接 : 隐式内连接 ; 显式内连接
-- 查询每一个员工的姓名,及关联的部门名称
-- 使用 隐式内连接
select emp.name, dept.name from emp , dept where emp.dept_id = dept.id;

select e.name , d.name from emp e , dept d where e.dept_id = d.id;

-- 使用 显式内连接
select e.name , d.name from emp e inner join dept d on e.dept_id = d.id;

select e.name , d.name from emp e join dept d on e.dept_id = d.id;


-- 外连接
-- 查询emp表的所有数据,以及对应的部门信息
-- 左外连接
select e.* , d.name from emp e left outer join dept d on e.dept_id = d.id;

select e.* , d.name from emp e left join dept d on e.dept_id = d.id;

-- 查询dept表的所有数据,以及对应的员工信息
-- 右外连接
select d.* , e.* from emp e right outer join dept d on d.id = e.dept_id;

select d.* , e.* from emp e right join dept d on d.id = e.dept_id;

-- 左连接和右连接可以相互转化,所以一般只使用左连接

-- 自连接
-- 查询员工及其所属领导的名字
-- 在 emp 表中,每一个人既是员工也可能是领导
select a.name , b.name from emp a , emp b where a.managerid = b.id;

-- 外连接和自连接的配合使用
-- 查询所有员工及其所属领导的名字,若员工没有领导,也将其查询出来
select a.name '员工' , b.name '领导' from emp a left join emp b on a.managerid = b.id;
```

#### 5.2 联合查询

```mysql
-- 联合查询 : 把多次查询的结果合并起来，形成一个新的查询结果集
-- UNION 各查询的列数必须一致，对应列的数据类型要能够兼容转换；列名由第一个 SELECT 决定。
-- union / union all
-- union all 会将全部的数据直接合并在一起，union 会对合并之后的数据去重

-- 将薪资低于 5000 的员工 , 和年龄大于 50 岁的员工全部查询出来
select * from emp where salary < 5000
union all
select * from emp where age > 50;

select * from emp where salary < 5000
union
select * from emp where age > 50;
```

#### 5.3 子查询

```mysql
-- 子查询 : SQL 语句中嵌套 SELECT 语句，称为嵌套查询，又称子查询
-- 子查询外部的语句可以是 INSERT / UPDATE / DELETE / SELECT 的任何一个
/*
根据子查询结果不同，分为：
1.标量子查询（子查询结果为单个值）
2.列子查询(子查询结果为一列)
3.行子查询(子查询结果为一行)
4.表子查询(子查询结果为多行多列)

根据子查询位置，分为：
1.WHERE之后
2.FROM之后
3.SELECT之后
*/

-- 标量子查询（子查询结果为单个值）
-- 查询销售部的所有员工信息
-- a. 查询"销售部"的id
-- b. 根据传到的id查找其中的员工信息
select * from emp where dept_id = (select id from dept where dept.name = '销售部');

-- 查询在'方东白'之后入职的员工信息
select * from emp where entrydate > (select entrydate from emp where name = '方东白');


-- 列子查询(子查询结果为一列) : in / not in / any = some / all
-- 查询销售部和市场部的所有员工信息
select * from emp where dept_id in (select id from dept where name in ('市场部','销售部'));

-- 查询比所有财务部所有人工资都高的员工信息
select * from emp where salary > all (select salary from emp where dept_id = (select id from dept where name = '财务部'));

-- 查询比研发部任意一人工资高的员工信息
select * from emp where salary > any (select salary from emp where dept_id = (select id from dept where name = '研发部'));


-- 行子查询(子查询结果为一行) : = / <> / in / not in
-- 查询与张无忌的薪资及直属领导相同的员工信息
select * from emp where (salary,managerid) = (12500,1);

select * from emp where (salary,managerid) = (select salary,managerid from emp where name = '张无忌');


-- 表子查询(子查询结果为多行多列) : in
-- 查询与'鹿杖客'或'宋远桥'的职位和薪资相同的员工的信息
select salary,job from emp where name = '鹿杖客' or name = '宋远桥';

select * from emp where (salary,job) in (select salary,job from emp where name = '鹿杖客' or name = '宋远桥');

-- 查询入职日期是'2006-1-1'之后的员工信息,及其部门信息
select * from (select * from emp where entrydate > '2006-01-01') e left join dept d on e.dept_id = d.id;
```

#### 5.4 多表查询练习

```mysql
-- 多表查询练习
create table salgrade
(
    grade int,
    losal int,
    hisal int
) comment '薪资等级表';

insert into salgrade
values (1, 0, 3000);
insert into salgrade
values (2, 3001, 5000);
insert into salgrade
values (3, 5001, 8000);
insert into salgrade
values (4, 8001, 10000);
insert into salgrade
values (5, 10001, 15000);
insert into salgrade
values (6, 15001, 20000);
insert into salgrade
values (7, 20001, 25000);
insert into salgrade
values (8, 25001, 30000);

-- 1)查询员工的姓名、年龄、职位、部门信息 （隐式内连接）
select e.name, e.age, e.job, d.*
from emp e,
     dept d
where e.dept_id = d.id;

-- 2)查询年龄小于30岁的员工的姓名、年龄、职位、部门信息（显式内连接）
select e.name, e.age, e.job, d.*
from emp e
         join dept d on e.dept_id = d.id
where e.age < 30;

-- 3)查询 拥有员工 的部门ID、部门名称(实际上就是 emp 和 dept 交集的部分,内连接)
select d.id, d.name
from dept d
         join emp e on d.id = e.dept_id; -- 这样查询出来的数据存在重复部分

select distinct d.id, d.name
from dept d
         join emp e on d.id = e.dept_id;
-- 去重

-- 4)查询所有年龄大于40岁的员工, 及其归属的部门名称; 如果员工没有分配部门, 也需要展示出来(外连接)
select e.*, d.name
from emp e
         left join dept d on e.dept_id = d.id
where e.age > 40;

-- 5)查询所有员工的工资等级(两表的连接条件)
select e.name, s.grade
from emp e
         join salgrade s on e.salary >= s.losal and e.salary <= s.hisal;

-- 6)查询 "研发部" 所有员工的信息及 工资等级
select e.*, s.grade
from emp e
         left join salgrade s on e.salary >= s.losal and e.salary <= s.hisal
where e.dept_id = (select id from dept where name = '研发部');

-- 7)查询 "研发部" 员工的平均工资
select avg(e.salary)
from emp e
where e.dept_id = (select id from dept where name = '研发部');

-- 8)查询工资比 "灭绝" 高的员工信息
select *
from emp e
where e.salary > (select salary from emp where name = '灭绝');

-- 9)查询比平均薪资高的员工信息
select *
from emp e
where e.salary > (select avg(salary) from emp);

-- 10)查询低于本部门平均工资的员工信息
select *
from emp e1
where e1.salary < (select avg(salary) from emp e2 where e2.dept_id = e1.dept_id);

-- 11)查询所有的部门信息, 并统计部门的员工人数
select d.id, d.name, (select count(*) from emp e where e.dept_id = d.id) '人数'
from dept d;

-- 12)查询所有学生的选课情况, 展示出学生名称, 学号, 课程名称(多对多的表连接)
select s.name, s.no, c.name
from student s,
     course c,
     student_course sc
where s.id = sc.studentID
  and c.id = sc.courseID;
```

### 6. 事务

```mysql
-- 事务
-- 事务 是一组操作的集合，它是一个不可分割的工作单位，
-- 事务会把所有的操作作为一个整体一起向系统提交或撤销操作请求，即这些操作要么同时成功，要么同时失败。
-- 我们只需要在业务逻辑执行之前开启事务，执行完毕后提交事务。
-- 如果执行过程中报错，则回滚事务，把数据恢复到事务开始之前的状态。

drop table if exists account;

create table account
(
    id    int primary key AUTO_INCREMENT comment 'ID',
    name  varchar(10) comment '姓名',
    money double(10, 2) comment '余额'
) comment '账户表';

insert into account(name, money)
VALUES ('张三', 2000),
       ('李四', 2000);

-- 避免出错的方式一 : 修改 @@autocommit 参数,进行手动提交/回滚事务

-- @@autocommit 自动提交指令,默认为 1 ,即自动提交
select @@autocommit;

-- 将 @@autocommit 设置为 0,即手动提交
set @@autocommit = 0;

-- 转账操作
-- 查询张三余额
select money from account where name = '张三';

-- 扣除张三余额
update account set money = money - 1000 where name = '张三';

-- 增加李四余额
update account set money = money + 1000 where name = '李四';


-- 手动提交事务
commit ;

-- 程序出错时 回滚事务
rollback ;


-- 方式二 : 不去修改 @@autocommit 指令,使用 begin / start transaction 控制事务

-- 开启事务,代表我们要手动控制事务
start transaction ;

-- 转账操作
-- 查询张三余额
select money from account where name = '张三';

-- 扣除张三余额
update account set money = money - 1000 where name = '张三';

-- 增加李四余额
update account set money = money + 1000 where name = '李四';

-- 提交事务
commit ;

-- 程序出错时,回滚事务
rollback ;


-- set @@autocommit = 0 在未写 GLOBAL 时只影响当前会话；该会话后续事务需要显式 COMMIT/ROLLBACK。
-- BEGIN / START TRANSACTION 只显式开启当前事务，事务结束后会话仍恢复其 autocommit 设置。


/*
事务四大特性(ACID) :
    原子性（Atomicity）：事务是不可分割的最小操作单元，要么全部成功，要么全部失败
    一致性（Consistency）：事务完成时，必须使所有的数据都保持一致状态
    隔离性（Isolation）：数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行
    持久性（Durability）：事务一旦提交，已提交的结果应在故障恢复后仍然保留；回滚表示撤销未提交修改，不是“永久保存回滚造成的改变”。
*/

/*
并发事务问题 :
    脏读：一个事务读到另外一个事务还没有提交的数据
    不可重复读：一个事务先后读取同一条记录，但两次读取的数据不同，称之为不可重复读
    幻读：一个事务按照条件查询数据时，没有对应的数据行，但是在插入数据时，又发现这行数据已经存在，好像出现了 "幻影"
*/


-- 事务隔离级别 : 用来解决上述并发事务问题
/*
    隔离级别                脏读      不可重复读       幻读
Read uncommitted            √           √             √
Read committed              ×           √             √
Repeatable Read(default)    ×           ×             √
Serializable                ×           ×             ×
*/

-- 隔离级别越高通常会增加并发冲突或等待，但性能不能只按隔离级别绝对排序，还取决于 SQL、索引、读写比例和竞争情况。
-- InnoDB 的 REPEATABLE READ 结合 MVCC 与 next-key lock 能避免许多幻读场景，但当前读、快照读和具体加锁行为仍需区分。

-- 查看事务的隔离级别
select @@transaction_isolation;

-- 设置事务的隔离级别(session / global 分别针对当前客户端和所有客户端)
set session transaction isolation level read committed ;

set session transaction isolation level repeatable read ;
```

### 7. MySQL 体系结构

```mysql
-- MySQL 体系结构

/*
1). 连接层
    最上层是一些客户端和链接服务，包含本地 sock 通信和大多数基于客户端/服务端工具实现的类似于
    TCP/IP 的通信。主要完成一些类似于连接处理、授权认证、及相关的安全方案。在该层上引入了线程
    池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于 SSL 的安全链接。服务
    器也会为安全接入的每个客户端验证它所具有的操作权限。
2). 服务层
    第二层完成 SQL 解析、权限检查、优化与执行等核心服务，也实现存储过程、函数等跨存储引擎能力。
    优化器会选择连接顺序、访问路径和索引，再把操作交给存储引擎。旧版 MySQL 的 Query Cache 已在
    MySQL 8.0 删除，不能再把“查询缓存”作为 MySQL 8.x 的执行层组成部分。
3). 引擎层(存储引擎)
    存储引擎层， 存储引擎真正的负责了 MySQL 中数据的存储和提取，服务器通过 API 和存储引擎进行通
    信。不同的存储引擎具有不同的功能，这样我们可以根据自己的需要，来选取合适的存储引擎。数据库
    中的索引是在存储引擎层实现的。(不同的引擎,索引结构是不同的)
4). 存储层
    数据存储层， 主要是将数据(如: redolog、undolog、数据、索引、二进制日志、错误日志、查询
    日志、慢查询日志等)存储在文件系统之上，并完成与存储引擎的交互


    和其他数据库相比，MySQL有点与众不同，它的架构可以在多种不同场景中应用并发挥良好作用。
    主要体现在存储引擎上，插件式的存储引擎架构，将查询处理和其他的系统任务以及数据的存储提取分离。
    这种架构可以根据业务的需求和实际需要选择合适的存储引擎。
*/
```

#### 7.1 存储引擎

```mysql
-- 存储引擎
/*
    存储引擎就是存储数据、建立索引、更新/查询数据等技术的实现方式 。存储引擎是基于表的，而不是
    基于库的，所以存储引擎也可被称为表类型。我们可以在创建表的时候，来指定选择的存储引擎，如果
    没有指定将自动选择默认的存储引擎(InnoDB)。
*/

-- 查询建表语句
show create table account;
-- 展示所有引擎
show engines ;

-- 创建表 my_myisam , 并指定MyISAM存储引擎
create table my_myisam(
                          id int,
                          name varchar(10)
) engine = MyISAM ;

-- 创建表 my_memory , 指定Memory存储引擎
create table my_memory (
                           id int,
                           name varchar(10)
) engine = memory ;



-- InnoDB 认识
/*
InnoDB 是一种兼顾高可靠性和高性能的通用存储引擎，在 MySQL 5.5 之后，InnoDB 是默认的 MySQL 存储引擎。

特点 :
    DML 操作遵循 ACID 模型，支持事务;
    行级锁，提高并发访问性能;
    支持外键 FOREIGN KEY 约束，保证数据的完整性和正确性

xxx.ibd ：xxx 代表的是表名，innoDB 引擎的每张表都会对应这样一个表空间文件，
存储该表的表结构（frm-早期的 、sdi-新版的）、数据和索引。
参数：innodb_file_per_table ,如果该参数开启，代表对于 InnoDB 引擎的表，每一张表都对应一个 ibd 文件。
 */
show variables like 'innodb_file_per_table';


-- InnoDB 逻辑存储结构
/*
表空间 : InnoDB 存储引擎逻辑结构的最高层，ibd 文件其实就是表空间文件，在表空间中可以包含多个 Segment 段。
段 : 表空间是由各个段组成的， 常见的段有数据段、索引段、回滚段等。InnoDB 中对于段的管理，都是引擎自身完成，不需要人为对其控制，一个段中包含多个区。
区 : 区是表空间的单元结构，每个区的大小为 1M。 默认情况下， InnoDB 存储引擎页大小为 16K， 即一个区中一共有64个连续的页。
页 : 页是组成区的最小单元，页也是 InnoDB 存储引擎磁盘管理的最小单元，每个页的大小默认为 16KB。为了保证页的连续性，InnoDB 存储引擎每次从磁盘申请 4-5 个区。
行 : InnoDB 按行组织记录。聚簇索引记录通常还包含事务 ID（DB_TRX_ID）和回滚指针（DB_ROLL_PTR）；如果表没有合适的主键，还会生成隐藏行 ID（DB_ROW_ID），因此不能固定说所有行都只有两个隐藏字段。
*/


-- MyISAM 认识
/*
MyISAM是MySQL早期的默认存储引擎。
特点 :
    不支持事务，不支持外键
    支持表锁，不支持行锁
    某些只读或顺序访问场景可能较快，但不能笼统认定始终快于 InnoDB；选择应以事务、并发、恢复需求和基准测试为依据。
文件 :
    xxx.sdi：存储表结构信息
    xxx.MYD: 存储数据
    xxx.MYI: 存储索引
*/


-- Memory 认识
/*
Memory 引擎的表数据时存储在内存中的，由于受到硬件问题、或断电问题的影响，只能将这些表作为临时表或缓存使用。
特点 :
    内存存放
    hash 索引（默认）
文件 :
    xxx.sdi：存储表结构信息
*/

/*
InnoDB引擎与MyISAM引擎的区别 ?
    1.InnoDB 引擎, 支持事务, 而 MyISAM 不支持。
    2.InnoDB 引擎, 支持行锁和表锁, 而 MyISAM 仅支持表锁, 不支持行锁。
    3.InnoDB 引擎, 支持外键, 而 MyISAM 是不支持的。
主要是上述三点区别，当然也可以从索引结构、存储限制等方面.
*/

-- 存储引擎的选择
/*
在选择存储引擎时，应该根据应用系统的特点选择合适的存储引擎。
对于复杂的应用系统，还可以根据实际情况选择多种存储引擎进行组合。

InnoDB : 是 Mysql 的默认存储引擎，支持事务、外键。如果应用对事务的完整性有比较高的要求，
    在并发条件下要求数据的一致性，数据操作除了插入和查询之外，还包含很多的更新、删除操作，那么 InnoDB 存储引擎是比较合适的选择。

MyISAM：仅在明确不需要事务、崩溃恢复、外键和行级并发，并经过基准验证的特殊场景考虑；常规业务默认优先 InnoDB。

MEMORY：将所有数据保存在内存中，访问速度快，通常用于临时表及缓存。
    MEMORY的缺陷就是对表的大小有限制，太大的表无法缓存在内存中，而且无法保障数据的安全性。
*/
```

### 8. 索引

#### 8.1 索引认识

```mysql
-- 索引

/*
索引（index）是帮助 MySQL 高效获取数据的数据结构(有序)。在数据之外，数据库系统还维护着满足
特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据， 这样就可以在这些数据结构
上实现高级查找算法，这种数据结构就是索引。

优势：
    1.提高数据检索的效率，降低数据库的IO成本
    2.通过索引列对数据进行排序，降低数据排序的成本，降低CPU的消耗。

劣势：
    1.索引列也是要占用空间的。
    2.索引大大提高了查询效率，同时却也降低更新表的速度，如对表进行INSERT、UPDATE、DELETE时，效率降低。
*/

/*
MySQL的索引是在存储引擎层实现的，不同的存储引擎有不同的索引结构，主要包含以下几种：
索引结构                        描述
B+Tree索引               最常见的索引类型，大部分引擎都支持 B+ 树索引
Hash索引                 底层数据结构是用哈希表实现的, 只有精确匹配索引列的查询才有效, 不支持范围查询
R-tree(空间索引）         空间索引用于地理空间数据类型；现代 MySQL 的 InnoDB 和 MyISAM 都支持 SPATIAL 索引，具体限制取决于版本、SRID 和数据类型
Full-text(全文索引)       是一种通过建立倒排索引,快速匹配文档的方式。类似于Lucene,Solr,ES
*/

/*
B-Tree
B-Tree，B 树是一种多路平衡查找树，相对于二叉树，每个节点可以有多个分支。
以一颗最大度数（max-degree）为5(5阶)的 b-tree 为例，那这个B树每个节点最多存储4个key，5个指针

特点:
    1.5阶的B树，每一个节点最多存储4个key，对应5个指针。
    2.一旦节点存储的key数量到达5，就会裂变，中间元素向上分裂。
    3.在B树中，非叶子节点和叶子节点都会存放数据。
*/

/*
标准的 B+Tree
特点:
    1.所有的数据都会出现在叶子节点。
    2.叶子节点形成一个单向链表。
    3.非叶子节点仅仅起到索引数据作用，具体的数据都是在叶子节点存放的。

MySQL 索引数据结构对经典的 B+Tree 进行了优化。在原 B+Tree 的基础上，增加一个指向相邻叶子节点
的双向链表指针，就形成了带有顺序指针的 B+Tree，提高区间访问的性能，利于排序
*/

/*
Hash
哈希索引就是采用一定的 hash 算法，将键值换算成新的 hash 值，映射到对应的槽位上，然后存储在 hash 表中

如果两个(或多个)键值，映射到一个相同的槽位上，他们就产生了hash冲突（也称为hash碰撞），可以通过链表来解决。

特点:
1.Hash 索引只能用于对等比较(=，in)，不支持范围查询（between，>，< ，...）
2.无法利用索引完成排序操作
3.查询效率高，通常(不存在 hash 冲突的情况)只需要一次检索就可以了，效率通常要高于 B+tree 索引

在 MySQL 中，支持 hash 索引的是 Memory 存储引擎。 而 InnoDB 中具有自适应 hash 功能，hash 索引是
InnoDB 存储引擎根据 B+Tree 索引在指定条件下自动构建的。
*/

/*
为什么 InnoDB 存储引擎选择使用 B+tree 索引结构?
1.相对于二叉树，层级更少，搜索效率高；
2.相对于 B-tree，无论是叶子节点还是非叶子节点，都会保存数据，
    这样导致一页中存储的键值减少，指针跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低；
3.相对于 Hash 索引，B+tree 支持范围匹配及排序操作
*/
```

#### 8.2 索引分类

```mysql
/*
索引分类
	主键索引(PRIMARY) : 针对于表中主键创建的索引;默认自动创建, 只能有一个
	唯一索引(UNIQUE) : 避免同一个表中某数据列中的值重复;可以有多个
	常规索引 : 快速定位特定数据 可以有多个
	全文索引(FULLTEXT) : 全文索引查找的是文本中的关键词，而不是比较索引中的值;可以有多个
*/

/*
在 InnoDB 存储引擎中，根据索引的存储形式，又可以分为:
	聚集索引(ClusteredIndex) : 将数据存储与索引放到了一块，索引结构的叶子节点保存了行数据(row);必须有,而且只有一个
	二级索引(SecondaryIndex) : 将数据与索引分开存储，索引结构的叶子节点关联的是对应的主键;可以存在多个

聚集索引选取规则:
    1.如果存在主键，主键索引就是聚集索引。
    2.如果不存在主键，将使用第一个唯一（UNIQUE）索引作为聚集索引。
    3.如果表没有主键，或没有合适的唯一索引，则 InnoDB 会自动生成一个 rowid 作为隐藏的聚集索引。

回表查询：先到二级索引中查找数据，找到主键值，然后再到聚集索引中根据主键值，获取数据的方式
    例如 select * from user where name = 'Arm'
    具体过程如下:
        ①. 由于是根据name字段进行查询，所以先根据 name = 'Arm' 到 name 字段的二级索引中进行匹配查
            找。但是在二级索引中只能查找到 Arm 对应的主键值 10。
        ②. 由于查询返回的数据是 * ，所以此时，还需要根据主键值 10，到聚集索引中查找 10 对应的记录，最
            终找到 10 对应的行 row。
        ③. 最终拿到这一行的数据，直接返回即可。
*/

/*
思考题：以下两条SQL语句，那个执行效率高? 为什么?
    A. select * from user where id = 10 ;
    B. select * from user where name = 'Arm' ;
    备注: id为主键，name字段创建的有索引；
解答：
A 语句的执行性能要高于 B 语句。
因为 A 语句直接走聚集索引，直接返回数据.
而 B 语句需要先查询 name 字段的二级索引，然后再查询聚集索引，也就是需要进行回表查询。

思考题：
InnoDB 主键索引的 B+tree 高度为多高呢?
假设:一行数据大小为 1k，一页中可以存储 16 行这样的数据。InnoDB 的指针占用 6 个字节的空间，主键即使为 bigint，占用字节数为 8。

高度为 2：
    n * 8 + (n + 1) * 6 = 16*1024 , 算出n约为 1170
    1171 * 16 = 18736
也就是说，如果树的高度为2，则可以存储 18000 多条记录

高度为 3：
    1171 * 1171 * 16 = 21939856
也就是说，如果树的高度为3，则可以存储 2200w 左右的记录
 */
```

#### 8.3 索引语法

```mysql
-- 索引语法

create database itzain;

use itzain;

create table tb_user
(
    id         int primary key auto_increment comment '主键',
    name       varchar(50) not null comment '用户名',
    phone      varchar(11) not null comment '手机号',
    email      varchar(100) comment '邮箱',
    profession varchar(11) comment '专业',
    age        tinyint unsigned comment '年龄',
    gender     char(1) comment '性别 , 1: 男, 2: 女',
    status     char(1) comment '状态',
    createtime datetime comment '创建时间'
) comment '系统用户表';

-- name 字段为姓名字段，该字段的值可能会重复，为该字段创建索引
create index idx_user_name on tb_user(name);
-- phone 手机号字段的值，是非空，且唯一的，为该字段创建唯一索引
create unique index idx_user_phone on tb_user(phone);
-- 为 profession、age、status 创建联合索引
create index idx_user_pro_age_sta on tb_user(profession,age,status);
-- 为 email 建立合适的索引来提升查询效率
create index idx_user_email on tb_user(email);

-- 查看索引数据。
show index from tb_user;

-- 删除索引
drop index idx_user_email on tb_user;
```

#### 8.4 SQL 性能分析

```mysql
-- 索引 - SQL 性能分析

-- 1.查看 SQL 执行频率，进行针对性优化
/*
MySQL 客户端连接成功后，通过 show [session|global] status 命令可以提供服务器状态信息。
通过如下指令，可以查看当前数据库的 INSERT、UPDATE、DELETE、SELECT 的访问频次：
*/
show global status like 'com_______';  -- 七个下划线

/*
Com_delete: 删除次数
Com_insert: 插入次数
Com_select: 查询次数
Com_update: 更新次数
*/
/*
如果是以增删改为主，我们可以考虑不对其进行索引的优化。 如果是以查询为主，那么就要考虑对数据库的索引进行优化了
*/


-- 2.慢日志查询
/*
慢查询日志记录了所有执行时间超过指定参数（long_query_time，单位：秒，默认10秒）的所有 SQL 语句的日志。
MySQL 的慢查询日志默认没有开启，我们可以查看一下系统变量 slow_query_log
*/
show variables like 'slow_query_log';

/*
如果要开启慢查询日志，需要在 MySQL 的配置文件（/etc/my.cnf）中配置如下信息:
	# 开启MySQL慢日志查询开关
	slow_query_log=1
	# 设置慢日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询，记录慢查询日志
	long_query_time=2

查看慢日志文件中记录的信息 /var/lib/mysql/localhost-slow.log

在慢查询日志中，只会记录执行时间超过我们预设时间（2s）的 SQL，执行较快的 SQL 是不会记录的
这样，通过慢查询日志，就可以定位出执行效率比较低的 SQL，从而有针对性的进行优化
*/


-- 3.历史命令：SHOW PROFILE（已弃用）
/*
SHOW PROFILE/SHOW PROFILES 已被弃用，不应作为 MySQL 8.4 的主要诊断方案。
新代码优先使用 Performance Schema、sys schema、慢查询日志和 EXPLAIN ANALYZE。
以下命令仅用于阅读旧资料或兼容旧环境：
*/
select @@have_profiling;

-- MySQL 支持 profile 操作，但开关默认是关闭的
select @@profiling; -- 0 代表关闭

-- 可以通过 set 语句在 session/global 级别开启 profiling：
set profiling = 1;

-- 开关打开后，我们所执行的 SQL 语句，都会被 MySQL 记录，并记录执行时间
select * from tb_user;
select * from tb_user where id = 1;
select * from tb_user where name = '白起';

-- 查看每一条 SQL 的耗时基本情况
show profiles;

-- 查看指定 query_id 的 SQL 语句各个阶段的耗时情况
show profile all for query 125;

-- 查看指定 query_id 的 SQL 语句 CPU 的使用情况
show profile cpu for query 78;


-- 4.explain 执行计划(常用)
/*
EXPLAIN 或者 DESC 命令获取 MySQL 如何执行 SELECT 语句的信息，
包括在 SELECT 语句执行过程中表如何连接和连接的顺序
*/

-- 直接在 select 语句之前加上关键字 explain / desc
explain select * from tb_user where id = 1 ;

/*
Explain 执行计划中各个字段的含义:
*id : select 查询的序列号，表示查询中执行 select 子句或者是操作表的顺序(id 相同，执行顺序从上到下；id 不同，值越大，越先执行)

select_type : 表示 SELECT 的类型，常见的取值有 SIMPLE（简单表，即不使用表连接或者子查询）、PRIMARY（主查询，即外层的查询）、UNION（UNION 中的第二个或者后面的查询语句）、SUBQUERY（SELECT/WHERE之后包含了子查询）等

*type : 表示连接类型，性能由好到差的连接类型为 NULL、system、const、eq_ref、ref、range、index、all

*possible_keys : 显示可能应用在这张表上的索引，一个或多个

*key : 实际使用的索引，如果为 NULL，则没有使用索引

*key_len : 表示优化器计划使用的索引键长度，可帮助判断联合索引使用到哪些列；不能简单认定越短越好

rows : MySQL 认为必须要执行查询的行数，在 innodb 引擎的表中，是一个估计值，可能并不总是准确的

filtered ：优化器估算经过条件过滤后剩余行的百分比，要与 rows、访问类型和实际执行结果一起分析，不是越大越好
*/
```

#### 8.5 索引 - 使用原则

```mysql
-- 索引 - 使用规则

-- 最左前缀法则
/*
如果索引了多列（联合索引），要遵守最左前缀法则。
最左前缀法则指的是查询从索引的最左列开始，并且不跳过索引中的列。如果跳跃某一列，索引将会部分失效(后面的字段索引失效)。

最左前缀法则中指的最左边的列，是指在查询时，联合索引的最左边的字段(即是第一个字段)必须存在，
与我们编写 SQL 时，条件编写的先后顺序无关


范围查询
联合索引遇到范围条件后，后续列能否继续用于访问或下推过滤取决于 MySQL 版本、条件形式和优化器选择。
`>`/`<` 与 `>=`/`<=` 都可能使用索引，不能仅为“让索引生效”而改写业务边界；应以 EXPLAIN ANALYZE 验证。
*/


-- 索引失效情况
/*
1.索引列运算
不要在索引列上进行运算操作， 否则索引将失效

2.字符串不加引号
字符串字面量应加引号。不匹配的数据类型可能触发隐式转换并影响索引使用，甚至改变查询语义，但是否失效取决于转换发生在哪一侧和执行计划。

3.模糊查询
如果仅仅是尾部模糊匹配，索引不会失效。如果是头部模糊匹配，索引失效
在 like 模糊查询中，在关键字后面加 %，索引可以生效。而如果在关键字前面加了 %，索引将会失效

4.or 连接条件
OR 条件可能使用 index_merge、单个索引或全表扫描；某一侧无索引会降低使用索引的收益，但不能写成绝对规则，应检查执行计划和数据分布。

5.数据分布影响
如果 MySQL 评估使用索引比全表更慢，则不使用索引
MySQL 在查询时，会评估使用索引的效率与走全表扫描的效率，如果走全表扫描更快，则放弃索引，走全表扫描
因为索引是用来索引少量数据的，如果通过索引查询返回大批量的数据，则还不如走全表扫描来的快，此时索引就会失效
*/


-- SQL 提示
/*
SQL 提示，是优化数据库的一个重要手段，简单来说，就是在SQL语句中加入一些人为的提示来达到优化操作的目的

1. use index ： 建议 MySQL 使用哪一个索引完成此次查询（仅仅是建议，mysql 内部还会再次进行评估）
explain select * from tb_user use index(idx_user_pro) where profession = '软件工程';

2. ignore index ： 忽略指定的索引
explain select * from tb_user ignore index(idx_user_pro) where profession = '软件工程';

3. force index ： 强制使用索引
explain select * from tb_user force index(idx_user_pro) where profession = '软件工程';
*/


-- 覆盖索引
/*
尽量使用覆盖索引，减少 select *，否则极易出现回表查询，性能降低
覆盖索引是指查询使用了索引，并且需要返回的列在该索引中已经全部能够找到

Extra：`Using where; Using index` 表示在使用覆盖索引的同时还要应用 WHERE 条件，通常无需回表。

Extra：`Using index condition` 表示使用了 Index Condition Pushdown（ICP），部分条件先在存储引擎的索引层判断；它不等同于一句“必然回表”，是否回表还取决于所需列和记录是否通过条件。

因为，在 tb_user 表中有一个联合索引 idx_user_pro_age_sta，该索引关联了三个字段 profession、age、status ，而这个索引也是一个二级索引，所以叶子节点下面挂的是这一行的主键 id 。
所以当我们查询返回的数据在 id、profession、age、status 之中，则直接走二级索引返回数据了。
如果超出这个范围，就需要拿到主键 id，再去扫描聚集索引，再获取额外的数据了，这个过程就是回表。
而我们如果一直使用 select * 查询返回所有字段值，很容易就会造成回表查询（除非是根据主键查询，此时只会扫描聚集索引）
*/

/*
思考题：
一张表有 id、username、password、status 四个字段。下列 SQL 是否需要优化，应先结合查询频率、选择性、表大小和执行计划判断：
select id,username,password from tb_user where username = 'itcast';

可选方案之一是在安全与空间成本允许时建立 `(username, password)` 覆盖索引，但不能称为无条件“最优”。密码摘要通常较长且敏感，把它放入二级索引会增加空间与写放大；也可只索引 username 并接受一次回表。必须用真实数据基准和 EXPLAIN ANALYZE 决策。
*/


-- 前缀索引
/*
当字段类型为字符串（varchar，text，longtext等）时，有时候需要索引很长的字符串，这会让索引变得很大，查询时浪费大量的磁盘 IO，影响查询效率。
此时可以只将字符串的一部分前缀，建立索引，这样可以大大节约索引空间，从而提高索引效率。

create index idx_xxxx on table_name(column(n)) ;

create index idx_email_5 on tb_user(email(5));

前缀长度，可以根据索引的选择性来决定，而选择性是指不重复的索引值（基数）和数据表的记录总数的比值，
索引选择性越高则查询效率越高， 唯一索引的选择性是 1，这是最好的索引选择性，性能也是最好的。
select count(distinct email) / count(*) from tb_user ;
select count(distinct substring(email,1,5)) / count(*) from tb_user ;
这里截取的长度可以试出来，如果只考虑查询选择性，则选择性为 1 时最好，
如果考虑选择性以及体积，选择性可以稍降一些，找到平衡点
*/


-- 单列索引与联合索引的选择
/*
单列索引：即一个索引只包含单个列。
联合索引：即一个索引包含了多个列。

在业务场景中，如果存在多个查询条件，考虑针对于查询字段建立索引时，建议建立联合索引，而非单列索引。
*/


-- 索引使用原则（总结）
/*
1). 针对于数据量较大，且查询比较频繁的表建立索引。
2). 针对于常作为查询条件（where）、排序（order by）、分组（group by）操作的字段建立索引。
3). 尽量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高。
4). 如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点，建立前缀索引。
5). 尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率。
6). 要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大，会影响增删改的效率。
7). 如果索引列不能存储 NULL 值，请在创建表时使用 NOT NULL 约束它。
    当优化器知道每列是否包含 NULL 值时，它可以更好地确定哪个索引最有效地用于查询。
*/
```

### 9. SQL 优化

#### 9.1 insert 优化

```mysql
/*
insert 优化
1). 优化方案一 : 批量插入数据
2). 优化方案二 : 手动控制事务
3). 优化方案三 : 主键顺序插入，性能要高于乱序插入

如果一次性需要插入大批量数据(比如: 几百万的记录)，使用 insert 语句插入性能较低，
此时可以使用 MySQL 数据库提供的 load 指令进行插入。
可以执行如下指令，将数据脚本文件中的数据加载到表结构中:

-- 客户端连接服务端时，加上参数 --local-infile=1（注意是两个 ASCII 连字符）
mysql --local-infile=1 -u root -p

-- 设置全局参数 local_infile 为 1 ，开启从本地加载文件导入数据的开关
set global local_infile = 1;

-- 执行 load 指令将准备好的数据，加载到表结构中
load data local infile '/root/sql1.log' into table tb_user fields terminated by ',' lines terminated by '\n' ;
*/
```

#### 9.2 主键优化

```mysql
/*
主键优化

在 InnoDB 存储引擎中，表数据都是根据主键顺序组织存放的，这种存储方式的表称为索引组织表(index organized table IOT)
在 InnoDB 引擎中，数据行是记录在逻辑结构 page 页中的，而每一个页的大小是固定的，默认16K。
那也就意味着， 一个页中所存储的行也是有限的，如果插入的数据行（row）在该页存储不小，将会存储到下一个页中，页与页之间会通过指针连接

页分裂
页可以为空，也可以填充一半，也可以填充 100%。每个页包含了 2-N 行数据(如果一行数据过大，会行溢出)，根据主键排列

主键顺序插入效果 :
	从磁盘中申请页， 主键顺序插入
	第一个页没有满，继续往第一页插入
	当第一个也写满之后，再写入第二个页，页与页之间会通过指针连接
	当第二页写满了，再往第三页写入

主键乱序插入效果 :
	加入1#,2#页都已经写满了，存放了如图所示的数据
	此时再插入id为50的记录，我们来看看会发生什么现象会再次开启一个页，写入新的页中吗？
	不会。因为，索引结构的叶子节点是有顺序的。按照顺序，应该存储在 47 之后。但是 47 所在的1#页，已经写满了，存储不了 50 对应的数据了。 那么此时会开辟一个新的页 3#。
	但是并不会直接将 50 存入 3# 页，而是会将 1# 页后一半的数据，移动到 3# 页，然后在 3# 页，插入50。
	移动数据，并插入id为50的数据之后，那么此时，这三个页之间的数据顺序是有问题的。 1# 的下一个页，应该是3#，3#的下一个页是2#。
	所以，此时需要重新设置链表指针。上述的这种现象，称之为 "页分裂"，是比较耗费性能的操作。


页合并
当我们对已有数据进行删除时，具体的效果如下:
	当删除一行记录时，实际上记录并没有被物理删除，只是记录被标记（flaged）为删除并且它的空间变得允许被其他记录声明使用。
	当我们继续删除2#的数据记录，当页中删除的记录达到 MERGE_THRESHOLD（默认为页的50%），InnoDB 会开始寻找最靠近的页（前或后）看看是否可以将两个页合并以优化空间使用。删除数据，并将页合并之后，再次插入新的数据21，则直接插入3#页
	这个里面所发生的合并页的这个现象，就称之为 "页合并"。


索引设计原则 ：
    1.满足业务需求的情况下，尽量降低主键的长度。
    2.插入数据时，尽量选择顺序插入，选择使用 AUTO_INCREMENT 自增主键。
    3.尽量不要使用 UUID 做主键或者是其他自然主键，如身份证号。
    4.业务操作时，避免对主键的修改。
*/
```

#### 9.3 order by 优化

```mysql
/*
order by优化

MySQL 的排序，有两种方式 ：

Using filesort : 通过表的索引或全表扫描，读取满足条件的数据行，然后在排序缓冲区 sort buffer 中完成排序操作，所有不是通过索引直接返回排序结果的排序都叫 FileSort 排序。

Using index : 通过有序索引顺序扫描直接返回有序数据，这种情况即为 using index，不需要额外排序，操作效率高。

对于以上的两种排序方式，Using index 的性能高，而 Using filesort 的性能低，
我们在优化排序操作时，尽量要优化为 Using index

Backward index scan，这个代表反向扫描索引，
因为在 MySQL 中我们创建的索引，默认索引的叶子节点是从小到大排序的，而此时我们查询排序时，是从大到小，
所以，在扫描时，就是反向扫描，就会出现 Backward index scan。
在 MySQL8 版本中，支持降序索引，我们也可以创建降序索引 :
create index idx_user_age_phone_ad on tb_user(age asc ,phone desc);

排序时,也需要满足最左前缀法则,否则也会出现 filesort

order by 优化原则:
	1.根据排序字段建立合适的索引，多字段排序时，也遵循最左前缀法则。
	2.尽量使用覆盖索引。
	3.多字段排序, 一个升序一个降序，此时需要注意联合索引在创建时的规则（ASC/DESC）。
	4.如果不可避免的出现 filesort，大数据量排序时，可以适当增大排序缓冲区大小 sort_buffer_size(默认256k)。
*/
```

#### 9.4 group by 优化

```mysql
/*
group by 优化

Using temporary : 效率不高
尽量要优化为 Using index

对于分组操作，在联合索引中，也是符合最左前缀法则的。

在分组操作中，我们需要通过以下两点进行优化，以提升性能：
    1.在分组操作时，可以通过索引来提高效率。
    2.分组操作时，索引的使用也是满足最左前缀法则的。
*/
```

#### 9.5 limit 优化

```mysql
/*
limit 优化

在数据量比较大时，如果进行 limit 分页查询，在查询时，越往后，分页查询效率越低
因为，当在进行分页查询时，如果执行 limit 2000000,10 ，此时需要 MySQL 排序前 2000010 记录，
仅仅返回 2000000 - 2000010 的记录，其他记录丢弃，查询排序的代价非常大

MySQL 版本不支持 select 嵌套 select limit 语句

优化思路: 一般分页查询时，通过创建覆盖索引能够比较好地提高性能，可以通过覆盖索引加子查询形式进行优化。
explain select t.*
from tb_sku t
join (select id from tb_sku order by id limit 2000000, 10) a on t.id = a.id;
*/
```

#### 9.6 count 优化

```mysql
/*
count 优化

MyISAM 会维护无条件总行数，因此无 WHERE 的 `count(*)` 可直接返回；带条件仍需访问数据。InnoDB 不维护精确总行数，具体性能取决于索引、条件、缓存和并发，不能把某一种 `count` 写成对所有场景固定最快。
InnoDB 的精确 `count(*)` 通常需要扫描一个可用索引并累计可见记录，但优化器会选择成本较低的访问路径，并非读取所有列。

若业务必须频繁读取大表总数，可维护汇总表或异步统计，但必须设计事务一致性、重算和故障补偿；仅放入 Redis 会引入数据库与缓存不一致风险。

count() 是一个聚合函数，对于返回的结果集，一行行地判断，如果 count 函数的参数不是 NULL，累计值就加 1，否则不加，最后返回累计值。
用法：count（*）、count（主键）、count（字段）、count（数字）

count(*) : InnoDB 引擎并不会把全部字段取出来，而是专门做了优化，不取值，服务层直接按行进行累加。
count(主键) : InnoDB 引擎会遍历整张表，把每一行的 主键id 值都取出来，返回给服务层。服务层拿到主键后，直接按行进行累加(主键不可能为null)
count(字段) :
    没有 not null 约束 : InnoDB 引擎会遍历整张表把每一行的字段值都取出来，返回给服务层，服务层判断是否为null，不为null，计数累加。
    有 not null 约束：InnoDB 引擎会遍历整张表把每一行的字段值都取出来，返回给服务层，直接按行进行累加。
count(数字) : InnoDB 引擎遍历整张表，但不取值。服务层对于返回的每一行，放一个数字“1”进去，直接按行进行累加。

不存在对所有版本、表结构和执行计划都成立的固定性能排序。统计行数且不排除 NULL 时优先使用语义清晰、优化器可专门处理的 `count(*)`；`count(column)` 的语义是只统计非 NULL 值。
*/
```

#### 9.7 update 优化

```mysql
/*
update 优化

当我们在执行带索引的 update 语句时，会锁定这一行的数据，然后事务提交之后，行锁释放。
InnoDB 不会因为条件未命中索引就把行锁“升级”为表锁。没有合适索引时，执行计划可能扫描并锁住大量索引记录或间隙，效果上会阻塞大量并发，但锁机制仍应按记录锁、间隙锁、next-key lock 等分析。

因此 UPDATE/DELETE 必须使用可选择的条件和合适索引，并通过 EXPLAIN、事务隔离级别与锁监控验证实际锁定范围。
即我们在更新数据时最好是根据带索引的字段进行更新
*/
```
