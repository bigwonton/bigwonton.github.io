---
title: 《SQL学习指南》笔记
date: 2018-12-15 19:14:45
tags: SQL
toc: true
---


### 创建和使用数据库

- 主键约束
`constraint pk_person primary key (person_id)`

- 检查约束
`gender char(1) check (gender in ('M','F')),`

或用`gender ENUM('M','F'),`代替检查约束

<!--more-->


```sql
 create table person
 (person_id smallint unsigned,
 fname varchar(20),
 lname varchar(20),
 gender enum('M','F'),
 birth_date date,
 street varchar(30),
 city varchar(20),
 state varchar(20),
 country varchar(20),
 postal_code varchar(20),
 constraint pk_person primary key (person_id)
 );
```



```sql
create table favorite_food
(person_id smallint unsigned,
food varchar(20),
constraint pk_favorite_food primary key (person_id, food),
constraint fk_fav_food_person_id foreign key (person_id)
references person (person_id)
);
```

**此时想要给person表主键增加auto_incremnt**
```
alter table person modify person_id smallint unsigned auto_increment;
```
报错，
```
Error Code: 1833. Cannot change column 'person_id': used in a foreign key constraint 'fk_fav_food_person_id' of table 'table.favorite_food'
```
，因为被用在favorite_food表中作为外键了，不允许修改，

==Solution 1==：

```
lock tables 
favorite_food write,
person write;
```
```
alter table favorite_food 
drop foreign key fk_fav_food_person_id,
modify person_id smallint unsigned;
```
然后再修改person主键
```
alter table person modify person_id smallint unsigned auto_increment;
```

创建外键
```
alter table favorite_food 
add constraint fk_fav_food_person_id foreign key (person_id) 
references person (person_id);
```
```
unlock tables;
```

==solution 2==

You can turn off foreign key checks:


```
SET FOREIGN_KEY_CHECKS = 0;

/* DO WHAT YOU NEED HERE */

SET FOREIGN_KEY_CHECKS = 1;
```

插入一条数据
```
insert into person 
(person_id, fname, lname,gender,birth_date) 
values (null, 'William', 'Turner','M','1972-05-27');
```

```
insert into favorite_food (person_id,food) 
values (1, 'pizza');
insert into favorite_food (person_id,food) 
values (1, 'cookies');
insert into favorite_food (person_id,food) 
values (1, 'nachos');
```

更新数据
```
update person 
set street='1225 Tremont St.',
city='Boston',
state='MA',
country='USA',
postal_code='02138' 
where person_id=1;
```

删除数据
```
delete from person 
where person_id=2;
```


### 查询

```
select emp_id 
'ACTIVE',
emp_id * 3.14159,
UPPER(lname) 
from employee;
```

执行内建函数或对简单的表达式求值可以完全省略from子句：
```
select version(),
user(),
database();
```

#### 列的别名：

```
select emp_id,
'ACTIVE' status,
emp_id * 3.14159 empid_x_pi,
UPPER(lname) last_name_upper 
from employee;
```

(可以在别名前加上AS关键字)


#### 去除重复的行

```
select distinct cust_id 
from account;
```

==注：产生无重复的结果集需要首先对数据排序，因此，不要为了确保去重二使用DiSTINCT，而应先了解所使用的数据是否可能包含重复的行。==


#### from子句

from子句定义了查询中所使用的表，以及连接这些表的方式。

- 表
    - 永久表（create table语句创建的表）
    - 临时表（子查询返回的表）
    - 虚拟表（create view）

通过别名e来引用子查询：
```
select e.emp_id, e.fname, e.lname 
from (select emp_id,fname,lname,start_date,title 
from employee) e;
```

视图是存储在数据字典中的查询，它的行为表现的像一个表，但实际上并不拥有任何数据（因此称为虚拟表）。当发出一个对视图的查询时，该查询会被绑定到视图定义上，以产生最终被执行的查询。

1. 定义一个查询employee表的视图

```
create view employee_vw AS 
select emp_id, fname, lname, 
YEAE(start_date) start_year 
from employee;
```

2. 查询视图

```
select emp_id, start_year 
from employee_vw;
```

##### 表连接

```
select employee.emp_id,employee.fname,employee.lname,
department.name dept_name 
from employee inner join department 
on employee.dept_id=department.dept_id;
```

##### 定义表别名

```
select e.emp_id,e.fname,e.lname,
d.name dept_name 
from employee e inner join department d 
on e.dept_id=d.dept_id;
```

#### where子句

```
select emp_id,fname,lname,start_date,title 
from employee 
where title='Head Teller' 
AND start_date > '2006-01-01';
```


#### group by和having子句

```
select d.name,count(e.emp_id) num_employees 
from department d inner join employee e 
on d.dept_id=e.dept_id 
group by d.name 
having count(e.emp_id) > 2;
```

#### order by

```
select open_emp_id, product_id 
from account 
order by open_emp_id;
```

- 升序（ASC、默认）
- 降序DESC

##### 根据表达式排序

使用right()内建函数提取fed_id列的最后三个字符排序：

```
select cust_id,cust_type_cd,city,state,fed_id 
from customer 
order by right(fed_id, 3);
```

##### 根据数字占位符排序

根据查询返回的第2，5列排序： 
```
select emp_id,title,start_date,fname.lname 
from employee 
order by 2,5;
```

### 过滤

#### 条件类型


```
select pt.name product_type,p.name product 
from product p inner join product_type pt 
on p.product_type_cd = pt.product_type_cd 
where pt.name = 'Customer Accounts';
```

不等条件：
```
select pt.name product_type,p.name product 
from product p inner join product_type pt 
on p.product_type_cd = pt.product_type_cd 
where pt.name <> 'Customer Accounts';
```

描述不等条件时，可以用`<>`也可以用`!=`


##### 范围条件

```
select emp_id,fname,lname,start_date 
from employee 
where start_date<'2007-01-01';
```

```
select emp_id,fname,lname,start_date 
from employee 
where start_date <'2007-01-01' 
and start_date >= '2005-01-01';
```

between(两端都是包含)

```
select emp_id,fname,lname,start_date 
from employee 
where start_date between '2005-01-01' and '2006-12-31';
```

```
select cust_id,fed_id 
from customer 
where cust_type_cd='I'
and fed_id between '500-00-0000' and '999-99-9999';
```

in操作符

```
select account_id,product_id,cust_id,avail_balance 
from account 
where product_cd in ('CHK','SAV','CD,'MM');
```

```
select account_id,product_id,cust_id,avail_balance 
from account 
where product_cd in (select product_cd from product 
where product_type_cd = 'ACCOUNT');
```


not in
```
select account_id,product_cd,cust_id,avail_balance 
from account 
where product_cd not in ('CHK','SAV','CD','MM');
```

##### 匹配条件

```
select emp_id,fname,lname 
from employee 
where left(lname,1)='T';
```

可以使用内建函数left()，但更好的做法是使用通配符：
```
select lname 
from employee 
where lname like '_a%e%';
```

```
select cust_id,fed_id 
from customer 
where fed_id like '___-__-____';

```

```
select emp_id,fname,lname 
from employee
where lname like 'F%' or lname like 'G%';
```

##### 正则表达式

```
select emp_id,fname,lname 
from employee 
where lname regexp '^[FG]';
```

#### NULL

```
select emp_id,fname,lname,superior_emp_id 
from employee 
where superior_emp_id is NULL;
```

### 多表查询

##### 笛卡尔积

交叉连接

```
select e.fname,e.lname,d.name 
from employee e join department d;
```

##### 内连接

如果在一个表中的dept_id列中存在某个值，但该值在另一张表的dept_id列中不存在，那么相关行的连接会失败，在结果集中将会排除包含该值的行。

```
select e.fname,e.lname,d.name 
from employee e join department d 
on e.dept_id = d.dept_id;
```

- 上面没加inner关键字，默认使用inner，但最好加上。
- 如果连接两个表的列名是相同的，可以使用using子句代替on子句：

```
select e.fname,e.lname,d.name 
from employee e inner join department d 
using (dept_id);
```


#### 连接3个或更多的表

```
select a.account_id,c.fed_id,e.fname,e.lname 
from account a inner join customer c 
on a.cust_id=c.cust_id 
inner join employee e 
on a.open_emp_id = e.emp_id 
where c.cust_type_cd = 'B';
```



##### 将子查询结果作为查询表

```
select a.account_id,a.cust_id,a.open_date,a.product_cd 
from account a INNER JOIN 
    (select emp_id,assigned_branch_id 
    from employee 
    where start_date < '2007-01-01' 
    and (title = 'Teller' OR title = 'Head Teller')) e
    ON a.open_emp_id = e.emp_id 
    INNER JOIN
        (select branch_id 
        from branch 
        where name = 'Woburn Branch') b 
        ON e.assigned_branch_id = b.branch_id;
```

##### 连续两次使用同一个表

需要给每个branch表的实例定义不用的别名

```
select a.account_id,e.emp_id,
b_a.name open_branch,b_e.name emp_branch 
from account a inner join branch b_a 
on a.open_branch_id=b_a.branch_id 
inner join employee e 
on a.open_emp_id=e.emp_id 
inner join branch b_e 
on e.assigned_branch_id=b_e.branch_id 
where a.product_cd='CHK';
```

#### 自连接

```
select e.fname,e.lname 
e_mgr.fname mgr_fname,e_mgr.lname mgr_lname 
from employee e inner join employee e_mgr 
on e.superior_emp_id = e_mgr.emp_id;
```

上面语句列出了雇员姓名和主管姓名，但是employee表中一共有18行，但此查询只返回17行，这是由于银行的总经理并没有自己的主管（superior_emp_id列为null），因此在该行连接失败了。为了在结果集中包含这个数据，应使用外连接。


```
select e1.fname, e1.lname, 'VS' vs, e2.fname, e2.lname 
from employee e1 inner join employee e2 
on e1.emp_id < e2.emp_id 
where e1.title='Teller' and e2.title = 'Teller';
```

#### 连接条件和过滤条件

```
select a.account_id, a.product_cd, c.fed_id 
from account a inner join customer c 
on a.cust_id = c.cust_id 
where c.cust_type_cd = 'B';
```


### 使用集合

- 当对两个数据集合执行集合操作时，必须满足：
    - 两个数据集合必须具有同样数目的列
    - 两个数据集中对应列的数据类型必须是一样的（或者服务器能偶从一种转换成另一种）

```
select 1 num, 'abc' str 
union 
select 9 num, 'xyz' str;
```


##### union操作符

union对连接后的集合排序并去除重复项，而union all保留重复项。

```
select 'IND' type_cd, cust_id, lname name 
from individual 
union all 
select 'BUS' type_cd, cust_id, name 
from business;
```


返回重复数据：

```
select emp_id 
from employee 
where assigned_branch_id=2 
and (title='Teller' or title='Head Teller') 
union all 
select distinct open_emp_id 
from account 
where open_branch_id = 2;
```

查询结果含有重复项，如果要排除重复项，需要把union all改成union


##### intersect 操作符


intersect 查找交集，并且去除了交集区域中所有重复的行，不删除重复的的是intersect all操作符

```
select emp_id 
from employee 
where assigned_branch_id=2 
and (title='Teller' or title='Head Teller') 
intersect  
select distinct open_emp_id 
from account 
where open_branch_id = 2;
```

##### except操作符

except操作符返回第一个表减去第二个表重合的元素后剩下的部分。
except在集合A中去除所有的重复数据，而except all则根据重复数据在集合B中出现的次数进行删除。

```
select emp_id 
from employee 
where assigned_branch_id=2 
and (title='Teller' or title='Head Teller') 
except   
select distinct open_emp_id 
from account 
where open_branch_id = 2;
```


##### 集合操作符优先级

```
select cust_id 
from account 
where product_cd in ('SAV','MM') 
union all 
select a.cust_id 
from account a inner join branch b 
on a.open_branch_id = b.branch_id 
where b.name = 'Woburn Branch' 
union 
select cust_id 
from account 
where avail_balance between 500 and 2500;
```

复合查询包含3个或3个以上的查询语句，他们是自顶向下的顺序解析和执行的，但要注意：

- 在调用集合操作时，intersect操作符比其他操作符具有更高的优先级
- 可以用圆括号对多个查询进行封装，以明确指定他们的指定次序

MySQL 没有实现intersect操作符

### 数据生成、转换和操作

#### 使用字符串数据

- `CHAR`
    - 固定长度，不足部分使用空格填充的字符串
    - MySQL的CHAR类型的长度为255字节。
    - Oracle为2000
    - SQL Server 8000
- `varchar`
    - 变长字符串。
    - MySQL允许的varchar列最多包含65536个字符
    - Oracle为4000
    - SQL Server为8000
- `text`(MySQL和SQL Server) 或 `CLOB`(Character Large Object; Oracle)
    - MySQL有多种text类型，最多4GB
    - SQL只有text，最长2GB
    - Oracle的CLOB，最大128TB


```
create table string_tbl
(char_fld CHAR(30),
vchar_fld varchar(30),
text_fld text);
```

##### 生成字符串

MySQL 6.0中，默认行为是“strict”模式，发生问题时抛出异常，而在早先的服务器版本中，默认方式是截断字符串并发出一个警告。如果希望数据库引擎采取后一种方式，可以将之修改为ANSI模式。

- 查看数据库当前模式

```
select @@session.sql_mode;
```

- 改变当前模式

```
set sql_mode='ansi';
```

==由于服务器是在纯粹字符串时按需分配空间，因此不会因为将varchar列的上限值设的比较大而浪费资源。==

- 单引号转义符

```
update string_tb1 set text_fld='This string didn''t work, but it does now';
```

可以使用单引号作为转义符

```
update string_tb1 set text_fld='This string didn\'t work, but it does now';
```

- 使用内建函数quote()，他用单引号将整个字符串包含起来，并为字符串本身的单引号/撇号增加转义符：
```
select quote(text_fld) 
from string_tbl;
```

- 包含特殊字符 
```
select 'abcdefg', CHAR(97,98,99,100,101,102,103);
```

```
select CHAR(138,139,140,141,142,143,144,145,146,147);
```

```
select concat('danke sch', char(148), 'n');
```

字符串长度：
```
select length(char_fld) char_length,
length(vchar_fld) varchar_length,
length(text_fld) text_length 
from string_tbl;
```

字符串位置：
```
select position('characters' in vchar_fld) 
from string_tbl;
```

如果找不到该子字符串，那么position()函数将返回0.


从任意位置开始查找字符串位置：

```
select locate('is', vchar_fld, 5) 
froom string_tbl;
```

字符串比较（只有mysql）

mysql的strcmp()大小写不敏感

```
select strcmp('12345', 'abc') 12345_abc,
strcmp('abcd', 'xyz') abcd_xyz,
strcmp('qrstuv', 'QRSTUV') qrstuv_QRSTUV;
```

- like
```
select name,name like '%ns' ends_in_ns 
from department;
```

```
select cust_id, cust_type_cd, fed_id, 
fed_id regexp '_{3}-_{2}-_{4}' is_ss_no_format 
from customer;
```

- concat()
```
update string_tbl 
set text_fld = concat(text_fld, ', but now it is longer');
```

Oracle下，可以使用连接操作符`||`代替concat()进行字符串连接
SQL Server并么有concat()函数，使用的是连接操作符`+`

- 插入字符串 insert()
```
select insert('goodbye world', 9, 0, 'cruel ') string;
```

如果第三个参数大于0，那么相应数目的字符将会被替换字符串所取代：

```
select insert('goodbye world', 1, 7, 'hello') string;
```

Oracle数据没有提供MySQL中的insert()类似函数，但他的replace()函数也可以用于替换字符串：

```
select replace('goodbye world', 'goodbye', 'hello') from dual;
```

SQL Server 同样包含replace()函数，除此之外，还包含stuff()函数：
```
select stuff('hello world', 1, 5, 'goodbye cruel');
```

- 子串 substring()
```
select substring('goodbye cruel world', 9, 5);
```

#### 使用数值数据

```
select (37 * 59) / (78 - (8 * 6));
```

```
select mod(10,4);

select pow(2, 8);


```

##### 控制数字经度

- ceil()
- floor()
- round() 四舍五入，保留n位小数
- truncate() 直接舍掉后面的
```
select ceil(72.445),floor(72.445);
```

结果:`73    72`

```
select round(72.4999),round(72.5);

select round(72.0909, 1), round(72.0909,2), round(72.0909, 3);
```

```
select truncate(72.0909,1), truncate(72.0909,2), truncate(72.0909,3);
```

truncate()和round()第二个参数都可以指定为负数：
```
select round(17, -1), truncate(17, -1);
```

结果：
`20  10`

##### 处理有符号数

```
select account_id, sign(avail_balance), abs(avail_balance) 
from account;
```

- sign()函数取符号
- abs()函数取绝对值

#### 使用时间数据

- 时区设置：全局时区和会话时区

```
select @@global.time_zone,@@session.time_zone;
```

服务器自动将字符串适配到datetime列：
```
update transaction 
set txn_date = '2008-09-17 15:30:00' 
where txn_id = 99999;
```

使用cast()函数转换

```
select cast('2008-09-17 15:30:00' as datetime);

select cast('2009-09-17' as date) date_field 
select cast('108:17:57' as time) time_field;
```



- `str_to_date()`函数  将字符串格式化为日期
```
update individual 
set birth_date = str_to_date('September 17, 2008', '%M %d, %Y') 
where cust_id = 9999;
```

- 当前日期
```
select current_date(), current_time(), current_timestamp();
```

- 当前日期加上5天 `date_add()`
```
select date_add(current_date(), interval 5 day);
```

```
update transaction 
set txn_date = date_add(txn_date, interval '3:27:11' hour_second) 
where txn_id = 9999;
```

- 当月最后一天  `last_day()`
```
select last_day('2008-09-17');
```


- 返回某一天是星期几 `dayname()`
```
select dayname('2008-09-18');
```
- 提取日期值中的信息 `extract()`函数
```
select extract(year from '2008-09-18 22:19:05');
```

- 时间间隔 `datediff()`
```
select datediff('2009-09-03', '2009-06-24');
```


#### 转换函数

- `cast()`

```
select cast('14567' as signed integer);
```

遇到非数字的字符，转换将中止并不返回错误：
```
selct cast('999ABC111' as unsigned integer);
```

### 分组与聚集

- group by

- 分组
```
select open_emp_id 
from account 
group by open_emp_id;
```

- 分组，并查看每一个分组的数量
```
select open_emp_id, count(*) how_many 
from account 
group by open_emp_id;
```

- 过滤出条数大于4的分组，需要用having，不能用where，因为用where过滤的时候，还未分组

```
select open_emp_id, count(*) how_many 
from account 
group by open_emp_id 
having count(*) > 4;
```
#### 聚集函数

- Max()
- Min()
- Avg()
- Sum()
- Count()

##### 隐式或显式分组

```
select max(avail_balance) max_balance,
min(avail_balance) min_balance,
avg(avail_balance) avg_balance,
sum(avail_balance) sum_balance,
count(*) num_accounts 
from account 
where product_cd = 'CHK';
```

上个例子中，查询返回的每个值都是由聚集函数产生的，这些聚集函数作用于使用过滤条件product_cd='CHK'指定的分组上的所有行。这里没有使用group by子句，因此它是一个隐式分组（即包含查询返回的所有行）。

但如果同时需要提取product_cd列，如下：
```
select product_cd,
max(avail_balance) max_balance,
min(avail_balance) min_balance,
avg(avail_balance) avg_balance,
sum(avail_balance) sum_balance,
count(*) num_accounts 
from account;
```

将会报错，因为没有显式地指定如何对数据分组。改为：
```
select product_cd,
max(avail_balance) max_balance,
min(avail_balance) min_balance,
avg(avail_balance) avg_balance,
sum(avail_balance) sum_balance,
count(*) num_accounts 
from account 
group by product_cd;
```


##### 对独立值计数

```
select count(open_emp_id)
from account;
```

去除重复的行统计：
```
select count(distinct open_emp_id) 
from account;
```

##### 对null的计算

count(*) 计算所有行
count(val) 会忽略null

#### 产生分组
##### 单列分组

```
select product_cd, sum(avail_balance) prod_balance 
from account 
group by product_cd;
```

##### 对多列的分组
```
select product_cd,open_branch_id, 
sum(avail_balance) tot_balance 
from account 
group by product_cd, open_branch_id;
```

##### 利用表达式分组

```
select extract(year form start_date) year,
count(*) how_many 
from employee 
group by extract(year from start_date);
```

##### 产生合计数 with rollup

- 独立产品也进行合计，最后还会产生总的合计

```
select product_cd, open_branch_id,
sum(avail_balance) tot_balance 
from account 
group by product_cd, open_branch_id with rollup;
```

#### 分组过滤条件

- 分组前——用where过滤
- 分组后——用having过滤
 
```
select product_cd, sum(avail_balance) prod_balance 
from account 
where status = 'ACTIVE' 
group by product_cd 
having sum(avail_balance) >=10000;
```

### 子查询

```
select account_id,product_id,cust_id,avail_balance 
from account 
where open_emp_id <> (select e.emp_id 
from employee e inner join branch b 
on e.assigned_branch_id = b.branch_id 
where e.title = 'Head Teller' and b.city = 'Woburn');
```

上述子查询只有一个结果，否则会报错。

#####  多行单列子查询
- in 和 not in运算分
```
select branch_id, name, city 
from branch 
where name in ('Headquarters', 'Quincy Branch');
```

查询所有主管：
```
select emp_id, fname, lname, title 
from employee 
where emp_id in (select superior_emp_id 
from employee);
```

查询所有不管理别人的雇员：

```
select emp_id, fname, lname, title 
from employee 
where emp_id not in (select superior_emp_id 
from employee 
where superior_emp_id is not null);
```

- all运算符 
将某单值与集合中的每个值进行比较：

 查询非主管雇员：
```
select emp_id, fname, lname, title 
from employee 
where emp_id <> all (select superior_emp_id 
from employee 
where superior_emp_id is not null);
```

==当使用not in或<>运算符比较一个值和一个值集时，必须确保值集中不包含null值，因为服务器将表达式左边的值与null比较时，都将产生未知的结果。==



- any运算符

any运算符只要有一个比较成立，则条件为真。all需要都成立才为真。

```
select account_id,cust_id,product_cd, avail_balance 
from account 
where avail_balance > any (select a.avail_balance 
from account a inner join individual i 
on a.cust_id = i.cust_id 
where i.fname = 'Frank' and i.lname = 'Tucker');
```


##### 多列子查询

- 多重单列子查询：
```
select account_id, product_id, cust_id 
from account 
where open_branch_id = (select branch_id 
from branch 
where name = 'Woburn Branch') 
and open_emp_id in (select emp_id 
from employee 
where title = 'Teller' or title = 'Head Teller');
```

等同于下面的 多列子查询：
```
select account_id, product_id, cust_id 
from account 
where (open_branch_id, open_emp_id) in 
(select b.branch_id, e.emp_id 
from branch b inner join employee e 
on b.branch_id = e.assigned_branch_id 
where b.name = 'Woburn Branch' 
and (e.title = 'Teller' or e.title = 'Head Teller'));
```

#### 关联子查询

子查询最后引用的c.cust_id使之具有关联性，这样他的执行必须依赖于包含查询提供的c.cust_id。
```
select c.cust_id, c.cust_type_cd, c.city 
from customer c 
where 2 = (select count(*) 
from account a 
where a.cust_id = c.cust_id);
```

```
select c.cust_id, c.cust_type_cd, c.city
from customer c 
where (select sum(a.avail_balance) 
from account a 
where a.cust_id = c.cust_id)
between 5000 and 10000;
```


```
select 'ALTER! : Account #1 Hs Incorrect Balance!' 
from account 
where (avail_balance, pending_balance) <> ¡™
(select sum(<expression to generate available balance>), 
sum(<expression to generate pending balance>) 
from transaction 
where account_id = 1) 
and account_id = 1;
```

对于上面的例子，用关联子查询代替非关联子查询，升级为：
```
select 'ALTER! : Account #1 Hs Incorrect Balance!' 
from account t 
where (avail_balance, pending_balance) <> 
(select sum(<expression to generate available balance>), 
sum(<expression to generate pending balance>) 
from transaction t 
where t.account_id = a.account_id);
```

##### exists 运算符

exists运算符是构造包含关联子查询条件最常用的运算符。

```
select a.account_id, a.product_cd, a.cust_id, a.avail_balance 
from account a 
where exists (select 1 
from transaction t 
where t.account_id = a.account_id 
and t.txn_date = '2008-09-22');
```

- not exists

```
select a.account_id, a.product_cd, a.cust_id 
from account a 
where not exists (select 1 
from business b 
where b.cust_id = a.cust_id);
```
##### 关联子查询操作数据

```
update account a 
set a.last_activity_date = 
(select max(t.txn_date) 
from transaction t 
where t.account_id = a.account_id);
```

但是上面的子查询语句没有检查每个账户是否发生过交易，否则有可能为null，因此修改为：
```
update account a 
set a.last_activity_date = 
(select max(t.txn_date) 
from transaction t 
where t.account_id = a.account_id) 
where exists (select 1 
from transaction t 
where t.account_id = a.account_id);
```

删除没有员工的部门：

```
delete from department 
where not exists (select 1 
from employee 
where employee.dept_id = department.dept_id);
```

==注：在MySQL中delete语句使用关联子查询时，不能使用表别名。==


查找开户数最多的雇员：

```
select open_emp_id, count(*) how_many 
from account 
group by open_emp_id 
having count(*) = (select max(emp_cnt.how_many) 
from (select count(*) how_many 
from account 
group by open_emp_id) emp_cnt);
```



检索雇员数据，排序第一准则是老板姓氏，第二是雇员的姓氏：
```
select emp.emp_id, concat(emp.fname, ' ', emp.lname) emp_name, 
(select concat(boss.fname, ' ', boss.lname) 
from employee boss 
where boss.emp_id = emp.superior_emp_id) boss_name 
from employee emp 
where emp.superior_emp_id is not null 
order by (select boss.lname from employee boss 
where boss.emp_id = emp.superior_emp_id), emp.lname;
```


### 再谈连接

#### 外连接

```
select a.account_id, a.cust_id, b.name 
from account a left outer join business b 
on a.cust_id = b.cust_id;
```


##### 左外连接与右外连接

left指连接左边的表决定结果集的行数，而右边只负责提供与之匹配的列值：
```
select c.cust_id,b.name 
from customer c left outer join business b 
on c.cust_id = b.cust_id;
```

如果是右链接：
```
select c.cust_id,b.name 
from customer c right outer join busines b 
where c.cust_id = b.cust_id;
```

##### 三路外连接

```
select a.account_id, a.product_cd, 
concat(i.fname, ' ', i.lname) person_name,
b.name business_name 
from account a left outer join individual i 
on a.cust_id = i.cust_id 
left outer join business b 
on a.cust_id = b.cust_id;
```

如果mysql对于外连接到同一个表的其他表的树木有限制，那么可以这么重写：
```
select account_ind.account_id, account_ind.product_cd,
account_ind.person_name,
b.name business_name 
from 
(select a.account_id, a.product_cd, a.cust_id, 
concat(i.fname, ' ', i.lname) person name 
from account a left outer join individual i 
on a.cust_id = i.cust_id) account_ind 
left outer join business b 
on account_ind.cust_id = b.cust_id;
```


##### 自外连接

```
select e.fname, e.lname,
e_mgr.fname mgr_fname, e_mgr.lname mgr_lname 
from employee e left outer join employee e_mgr 
on e.superior_emp_id = e_mgr.emp_id;
```
#### 交叉连接

```
select pt.name, p.product_cd, p.name 
from product p cross join product_type pt;
```

- 生成2008年全年日期
```
select date_add('2008-01-01', interval (ones.num + tens.num + hundreds.num) day) dt  
from 
(select 0 num union all 
select 1 num union all 
select 2 num union all 
select 3 num union all 
select 4 num union all 
select 5 num union all 
select 6 num union all 
select 7 num union all 
select 8 num union all 
select 9 num) ones 
cross join 
(select 0 num union all 
select 10 num union all 
select 20 num union all 
select 30 num union all 
select 40 num union all 
select 50 num union all 
select 60 num union all 
select 70 num union all 
select 80 num union all 
select 90 num) tens  
cross join 
(select 0 num union all 
select 100 num union all 
select 200 num union all 
select 300 num) hundreds 
where date_add('2008-01-01', 
interval(ones.num+tens.num+hundreds.num) day) < '2009-01-01' 
order by 1;
```

==order by 1表示根据第一列的值自然排序。==


#### 自然连接

要避免不要使用自然连接。

### 条件逻辑

- 根据cust_type_cd字段判断决定使用indiv_name还是business_name列的值：
```
select c.cust_id, c.fed_id,
case 
when c.cust_type_cd = 'I' 
then concat(i.fname, ' ', i.lname) 
when c.cust_type_cd = 'B' 
then b.name 
else 'Unknown' 
end name 
from customer c left outer join individual i 
on c.cust_id = i.cust_id 
left outer join business b 
on c.cust_id = b.cust_id;
```

#### case表达式

##### 查找型case表达式

```
CASE 
    WHEN C1 THEN E1 
    WHEN C2 THEN E2 
    ...
    WHEN CN THEN CN 
    [ELSE ED] 
END
```

上面的语句可以改为：
```
select c.cust_id, c.fed_id,
case
    when c.cust_type_cd = 'I' then 
    (select concat(i.fname, ' ', i.lname) 
    from individual i 
    where i.cust_id = c.cust_id) 
    when c.cust_type_cd = 'B' then 
    (select b.name 
    from business b 
    where b.cust_id = c.cust_id) 
    else 'Unknown' 
    end name 
from customer c;
```

##### 结果集变换
```
select year(open_date) year, count(*) how_many 
from account 
where open_date > '1999-12-31' 
and open_date < '2006-01-01' 
group by year(open_date);
```

结果是6行，如果想要变换成单行6列：
```
select 
    sum(case 
            when extract(year from open_date) = 2000 then 1 
            else 0 
        end) year_2000,
    sum(case 
            when extract(year from open_date) = 2001 then 1 
            else 0 
        end) year_2001,
    sum(case 
            when extract(year from open_date) = 2002 then 1 
            else 0 
        end) year_2002,
    sum(case 
            when extract(year from open_date) = 2003 then 1 
            else 0 
        end) year_2003,
    sum(case 
            when extract(year from open_date) = 2004 then 1 
            else 0 
        end) year_2004,
    sum(case 
            when extract(year from open_date) = 2005 then 1 
            else 0 
        end) year_2005
from account 
where open_date > '1999-12-31' and open_date < '2006-01-01';
```
        
        
##### 存在性检查

```
select c.cust_id, c.fed_id, c.cust_type_cd,
    case 
        when exists (select 1 from account a 
        where a.cust_id = c.cust_id 
        and a.product_cd = 'CHK') then 'Y' 
        else 'N' 
    end has_checking,
    case
        when exists (select 1 froom account a 
        where a.cust_id = c.cust_id 
        and a.product_cd = 'SAV') then 'Y' 
        else 'N' 
    end has_savings 
from customer c;
```

##### 除零错误

```
select a.cust_id, a.product_cd, a.avail_balance / 
 case
  when prod_tots.tot_balance = 0 then 1 
  else prod_tots.tot_balance 
 end percent_of_total 
from account a inner join 
(select a.product_cd, sum(a.avail_balance) tot_balance 
from account a 
group by a.product_cd) prod_tots 
on a.product_cd = prod_tots.product_cd;
```

### 事务

#### 多用户数据库

##### 锁

两种锁……
- ==写锁+读锁==
- 写锁，读取不需要锁。服务器要保证从查询开始到结束读操作看到一个一致的数据视图，这个方法被称为==版本控制==。

##### 锁的粒度

- 表锁
- 页锁（阻止多用户同时修改某表中同一页（一页通常是一段2～16KB的内存空间）的数据）
- 行锁


**显式启动事务**：
- SQL Server
```
begin transaction
```
- MySQL
```
start transaction
```

**关闭自动提交：**
- SQL Server

```
set implicit_transactions on
```
- MySQL
```
set autocommit=0
```


对于SQL Server和MySQL，默认自动提交模式，直到显示启动一个事务。

一旦离开了自动提交模式，所有的SQL命令都会发生在同一个事务的范围，并且必须显式地对事务进行提交或回滚。

##### 结束事务

- commit
- rollback

除了提交commit或rollback指令，结束事务还可以有其他情景触发：
- 服务器宕机
- 提交一个SQL模式语句，比如alter table，这将会引起当前事务提交和一个新事务启动
- 提交另一个start transaction命令，将会引起前一个事务提交
- 因为服务器监测到一个死锁，并确定当前事务是罪魁祸首，服务器将会结束当前事务。这种情况下，事务将会被回滚，同时释放错误消息（大多数情况喜爱，终止的事务可以重启，如果没有再次遇到另一个死锁情况他将会成功）


##### 事务保存点

6.0版的MySQL包括以下存储引擎：
- MyISAM    表级锁定的非事务引擎
- MEMORY    内存表使用的非事务引擎
- BDB       页级锁定的事务引擎
- InnoDB    行级锁定的事务引擎
- Merge     使多个相同的MyISAM看起来像一个单表（也叫表分割）的专用引擎
- Maria     6.0.6版本中MyISAM的替代品，他添加了充分的恢复功能
- Falcon    6.0.4版本起引入的采用行级锁定的高性能事务引擎
- Archive   用于存储大量未检索数据的专用引擎，主要用来存档

查看引擎：
```
show table status like 'transaction' \G
```

设置引擎：
```
alter table transaction engine = innodb;
```


- **创建保存点**

```
savepoint my_savepoint;
```

- **回滚到保存点**

```
rollback to savepoint my_savepoint;
```

### 索引和约束


#### 索引
索引是一种以特定顺序保存的专用表。


##### 创建索引

```
alter table department 
add index dept_name_idx (name);
```

- 查看索引
```
show index from department \G
```

创建表时的包含一个约束，该约束将primary key dept_id列作为主关键字。

- 删除索引
```
alter table department 
drop index dept_name_idx;
```

- **唯一索引**


设计数据库时，需要考虑哪些列能包含重复数据，哪些列不能。
例如department表中不能出现两个相同名字的部门。可以通过department.name创建唯一索引限制出现重复部门名字。

```
alter table department 
add unique dept_name_idx (name);
```



- **多列索引**

```
alter table employee 
add index emp_names_idx (lname, fname);
```


创建多列索引时，必须考虑哪一列作为第一列，哪一列作为第二列，这样索引才会尽可能的有用。

##### 索引类型

- **B树索引**

平衡树索引

MySQL、Oracle、SQL Server默认都是B树索引。

当向表中插入、更新和删除行时，服务器会尽力保持树的平衡。


- **位图索引**

位图索引通常应用于数据仓库环境，那里有大量数据被索引，那些列却只包含相对少的值（比如销售岗位、地理环境、产品、销售员）

- **文本索引**

MySQL、SQL Server   全文索引
Oracle              Oracle Text

##### 如何使用索引

```
explain select cust_id, sum(avail_balance) tot_bal 
from account 
where cust_id in (1,5,9,11) 
group by cust_id \G
```

结果：
```
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: account
   partitions: NULL
         type: index
possible_keys: fk_a_cust_id
          key: fk_a_cust_id
      key_len: 4
          ref: NULL
         rows: 24
     filtered: 33.33
        Extra: Using where
1 row in set, 1 warning (0.01 sec)
```


优化，给cust_id和avail_balance两列添加新索引：
```
alter table account 
add index acc_bal_idx (cust_id, avail_balance);
```

再次查询优化器是如何处理的：
```
explain select cust_id, sum(avail_balance) tot_bal 
from account 
where cust_id in (1,5,9,11) 
group by cust_id \G
```
结果如下：
```
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: account
   partitions: NULL
         type: range
possible_keys: acc_bal_idx
          key: acc_bal_idx
      key_len: 4
          ref: NULL
         rows: 8
     filtered: 100.00
        Extra: Using where; Using index
1 row in set, 1 warning (0.01 sec)
```

- ==优化器使用了新索引，而不是fk_a_cust_id==
- ==优化器预期只需要8行，而不是24行==
- ==不需要account表即可满足查询结果（使用附加列的索引指定）==

##### 索引的不足


- 每个索引其实 都是一个表
- 每次在对表添加或删除行时，表中的所有索引必须被修改，当更新行时，受到影响的列的任何索引也必须被修改。
- 索引越多，服务器就需要做越多的工作来保持所有模式对象最新。


默认策略：
- ==确保所有的主键列被索引==（大部分服务器会在创建主键约束时自动生成唯一索引），针对多列主键，考虑为主键列的子集构建附加索引，或者以与主键约束定义不同的顺序为所有主键列另外生成索引
- ==为所有被外键约束引用的列创建索引。== 服务器在准备删除父行时会查找以确保没有子行存在，为此他必须发出一个查询搜索列中的特定值，如果该列没有索引，那么服务器必须扫描整个表
- ==索引那些被频繁检索的列。除了短字符串（3～50个字符）列，大多数日期列也是不错的候选==


#### 约束

- 主键约束
- 外键约束
- 唯一约束
- 检查约束

**创建约束**
```sql
create table product 
(product_cd varchar(10) not null,
name varchar(50) not null,
product_type_cd varchar(10) not null,
date_offered date,
date_retired date,
constraint fk_product_type_cd foreign key (product_type_cd) 
references product_type (product_type_cd) ,
constraint pk_product primary key (product_cd)
);

```


##### 约束与索引

MySQL 主键约束、外键约束和唯一约束时，都会生成索引。

##### 级联约束


- on updae cascade

```sql
alter table product 
drop foreign key fk_product_type_cd;

alter table product 
add constraint fk_product_type_cd foreign key (product_type_cd) 
references product_type (product_type_cd) 
on update cascade;
```

- on delete cascade

```
alter table product 
add constraint fk_product_type_cd foreign key (product_type_cd) 
references product_type (product_type_cd) 
on update cascade 
on delete cascade;
```


### 视图

创建视图
```
create view customer_vw 
(cust_id,
fed_id,
cust_type_cd,
address,
city,
state,
zipcode
)
as
select cust_id,
concat('ends in ', substr(fed_id,8,4)) fed_id, 
cust_type_cd,
address,
city,
state,
postal_code
from customer;
```
使用视图
```
select cust_id, fed_id, cust_type_cd 
from customer_vw;
```

服务器真正执行的查询是将两者合并创建的一个新查询：
```
select cust_id,
concat("ends in ", substr(fed_id,8,4)) fed_id,
cust_type_cd 
from customer;
```

查看视图有哪些列：
```
describe customer_vw;
```


```
select cst.cust_id, cst.fed_id, bus.name 
from customer_vw cst inner join business bus 
on cst.cust_id = bus.cust_id;
```

#### 为什么使用视图

##### 数据安全

只能查询企业用户：
```
create view business_customer_vw 
(cust_id,
fed_id,
cust_type_cd,
address,
city,
state,
zipcode
) 
as 
select cust_id,
concat('ends in ', substr(fed_id, 8, 4)) fed_id,
cust_type_cd,
address,
city,
state,
postal_code 
from customer 
where cust_type_cd = 'B';
```


##### 数据聚合

生成报表展示账户数据和每个客户的储蓄总额：
```
create view customer_totals_vw 
(cust_id,
cust_type_cd,
cust_name,
num_accounts,
tot_deposits) 
as 
select cst.cust_id, cst.cust_type_cd,
 case 
  when cst.cust_type_cd = 'B' then 
  (select bus.name from business bus where bus.cust_id = cst.cust_id) 
  else 
  (select concat(ind.fname, ' ', ind.lname) 
  from individual ind 
  where ind.cust_id = cst.cust_id) 
 end cust_name,
 sum(case when act.status = 'ACTIVE' then 1 else 0 end) tot_active_accounts,
 sum(case when act.status = 'ACTIVE' then act.avail_balance else 0 end) tot_balance 
from customer cst inner join account act 
on act.cust_id = cst.cust_id 
group by cst.cust_id, cst.cust_type_cd;
```

决定将数据预聚合到一个表中而不是利用视图总计，那么可以先创建customer_totals表，然后修改视图customer_totals_vw定义，再从该表中检索数据。
```
create table customer_totals  
as 
select * from customer_totals_vw;
```

```
create or replace view customer_totals_vw 
(cust_id,
cust_type_cd,
cust_name,
num_accounts,
tot_deposits
) 
as 
select cust_id, cust_type_cd, cust_name, num_accounts, tot_deposits 
from customer_totals;
```

##### 隐藏复杂性

##### 连接分区数据

transaction变的越来越大，那么设计者可以将其分为两个表：transaction_current,transaction_historic。然后可以通过视图将两个表联合起来：
```
create view transaction_vw 
(txn_date,
account_id,
txn_type_cd,
amount,
teller_emp_id,
execution_branch_id,
funds_avail_date
) 
as 
select txn_date, account_id, txn_type_cd, amount, teller_emp_id, 
execution_branch_id, funds_avail_date 
from transaction_historic 
union all 
select txn_date, account_id, txn_type_cd, amount, teller_emp_id, 
execution_branch_id, funds_avail_date 
from transaction_current;
```


#### 可更新的视图

对于MySQL，如果以下条件能满足，那么视图就是可更新的：
- 没有使用聚合函数，例如max()、min()、avg() 等
- 视图没有使用group by或having子句
- select或from子句中不存在子查询，并且where子句中的任何子查询都不引用from子句中的表
- 视图没有使用union、union all和distinct
- from子句包括不止一个表或可更新视图
- 如果有不止一个表或视图，那么from子句只使用内连接

修改视图：
```
update customer_vw 
set city = 'Woooburn' 
where city =  'Woburn';
```


### 元数据

- 元数据——数据的数据
- 数据字典或者系统目录


#### 信息模式

- ==information_schema数据库==里所有可用对象是视图

检索bank数据库里所有表的名字：
```
select table_name, table_type 
from information_schema.tables 
where table_schema = 'bank' 
order by 1;
```

在结果中排除视图：
```
select table_name, table_type 
from information_schema.tables 
where table_schema = 'bank' and table_type = 'BASE TABLE' 
order by 1;
```

只检索视图：
```
select table_name,is_updatable 
from information_schema.views 
where table_schema='bank' 
order by 1;
```


查看表的信息：
```
select column_name,data_type,character_maximum_length char_max_len,
numeric_precision num_prcsn, numeric_scale num_scale 
from infromation_schema.columns 
where table_schema='bank' and table_name='account' 
order by ordinal_position;
```


查询表中的索引信息：
```
select index_name,non_unique,seq_in_index,column_name 
from information_schema.statistics 
where table_schema = 'bank' and table_name = 'account' 
order by 1,3;
```

查询bank模式中的所有视图：
```
select constraint_name,table_name,constraint_type 
from information_schema.table_constraints 
where table_schema = 'bank' 
order by 3,1;
```

#### 使用元数据

##### 模式生成脚本

##### 生成动态SQL

MySQL为动态SQL执行提供了：
- prepare
- execute
- deallocate
语句


```
--将sql语句赋给变量qry
set @qry = 'select cust_id, cust_type_cd, fed_id from customer';

-- qry被prepare语句提交给数据库引擎（解析、安全检查、优化）
prepare dynsql1 from @qry;

-- 执行
execute dynsql1;

-- deallocate prepare关闭语句，释放执行中使用的所有数据库资源（如游标）
deallocate prepare dynsql1;

```



```
-- 查询包含占位符
set @qry = 'select product_cd, name, product_type_cd, date_offered, date_retired from product where product_cd = ?';

prepare dynsql2 from @qry;

set @prodcd = 'CKH';

execute dynsql2 using @prodcd;

set @prodcd = 'SAV';

execute dynsql2 usging @prodcd;

deallocate prepare dynsql2;

```

可以通过使用元数据生成动态SQL查询语句，而不是使用上面的硬编码。


### MySQL对SQL语言的扩展

#### limit子句


```
select open_emp_id, count(*) how_many 
from account 
group by open_emp_id 
limit 3;
```


- 组合limit子句和order by子句
```
select open_emp_id, count(*) how_many 
from account 
group by open_emp_id 
order by how_many desc 
limit 3;
```


```
select open_emp_id, count(*) how_many 
from account 
group by open_emp_id 
order by how_many desc 
limit 2,1;
```

```
select open_emp_id, count(*) how_many 
from account 
group by open_emp_id 
order by how_many desc 
limit 2,999999999;
```


##### into outfile子句

```
select emp_id, fname, lname, start_date 
into outfile 'C:\\TEMP||emp_list.txt' 
from employee;
```
默认列间用制表符（'\t'）隔开，记录间用换行符（'\n'）隔开。

- 改成用字符 '|'隔开
```
select emp_id, fname, lname, start_date 
into outfile 'C:\\TEMP||emp_list.txt' 
fields terminated by '|' 
lines terminated by '@' 
from employee;
```


##### 组合insert/update语句

更新插入（upsert）

```
insert into branch_usage (branch_id, cust_id, last_visited_on) 
values (1,5, current_timestamp()) 
on duplicate key update last_visited_on = current_timestamp();
```


##### 多表更新与删除

同时删除account、customer、individual表中的某一个客户数据

```
delete account2, customer2, individual2 
from account2 inner join customer2 
on account2.cust_id=customer2.cust_id 
inner join individual2 
on customer2.cust_id = individual2.cust_id 
where individual2.cust_id = 1;
```








