---
title: PHP 与 MySQL数据库
date: 2017-07-24
categories:
- PHP
---
>Web程序中的各类数据需要靠数据库存储，只有在数据库配合的基础上，PHP才能发挥其最大功效。

>MySQL本身并不是数据库，他只是用来创建、维护、管理数据库的计算机软件。

#### MySQL数据库的基本操作
```basicOperations
//windous 登录MySQL命令
mysql -u user_name -p password -h host
//退出MySQL
quit
exit

//创建数据库
create database database_name;

//删除数据库
drop database database_name;

//表的建立
create table Users(
UserId int unsignd not null,
UserName varchar(50) not null,
Gender char(6) not null defaultale',
RegTime date not null;
);
// varchar是变长字符串，在1-255之间
// char指定长度，不足空格补

//建立索引：INDEX
create table books(
id int not null,
name varchar(40) not null,
price = decimal(3,2),
index idx(id)
);

//特殊索引，主键
create table users(
id int not null primary key,
name varchar(30),
create_time date
);

//查看某个数据库中所有表的命令
show tables

//查看某个表的结构
use database
describe table_name

//插入数据
insert into users(id,name,created_time) values (1,'MrLiuQ','2017-07-22 10:18:00');
insert into users values(1,'MrLiuQ','2017-07-22 10:18:00');//省略字段名
insert into users set id=2,name='David',create_name='2017-07-22 10:18:00';

//查询数据
SELECT name,created_time AS ct FROM users;

//条件查询
SELECT *FROM users WHERE name='MrLiuQ';
SELECT *FROM users WHERE created_time>'2017-07-22 10:18:00';
SELECT *FROM users WHERE name='MrLiuQ'->AND created_time>'2017-07-22 10:18:00';
SELECT *FROM users WHERE name='MrLiuQ'->OR created_time>'2017-07-22 10:18:00';

//更新数据
UPDATE users SET name'Lily Cameron' WHERE id=3;//成功后，表users中id为3的用户名被修改为Lily Cameron

//删除数据
DELETE FROM users WHERE name='Lily Cameron';

//对查询结果做排序
SELECT *FROM users WHERE created_time >'2017-07-22 10:18:00'->ORDER BY id DESC;//ASC表示升序，DESC表示降序。

//对查询结果做分组
SELECT city FROM users GROUP BY city;

//对查询结果做限定
SELECT name,city FROM users ORDER BY name LIMIT 4;//返回前4条信息
```

#### MySQL的数据类型
>INT：整数值。UNSIGNED INT指定无符号整数值
DECIMAL：指定数字值的精度和范围
REAL：浮点数值
CHAR：定长字符类型
VARCHAR：变长字符串，在1~255之间
TEXT：文本类型
DATE：日期值
TIME：时间值
DATETIME：MySQL支持，存放日期和日期类型

#### MySQL函数
>COUNT()：统计表中记录个数或这列中值的个数
MAX()：返回某列中的最大值
MIN()：返回某列中的最小值
SUM()：指定列的值求和
AVG()：计算指定列的平均值
LENGTH()：字符串的长度
SUBSTRING()：截取子字符串
YEAR()：返回指定日期的年份
UNIX_TIMESTAMP()：返回一个UNIX时间戳
DATE_FORMAT()：将一个日期格式化
NOW()：返回MySQL服务器系统的当前日期和时间

```function
//计算表中总共有多少条记录
SELECT COUNT(*) FROM users;
//计算特定值的个数
SELECT COUNT(*) FROM users WHERE name LIKE 'J%';

//计算列最大值
SELECT MAX(salary) AS max_salary FROM emp_salary;

//计算列最小值
SELECT MIN(salary) AS max_salary FROM emp_salary;

//求和
SELECT SUM(salary) AS total_salary FROM emp_salary;

//求平均
SELECT AVG(salary) AS avg_salary FROM emp_salary;

//字符串长度
SELECT LENGTH('string in MySQL');

//截取指定长度字符串
SUBSTRING(string,pos,len);

//获取一个UNIX时间戳
SELECT UNIX_TIMESTAMP() AS timestamp;

//格式化输出日期
SELECT DATE_FORMAT<created_time,'%Y年%m月%d日'> AS DATE-> FROM users WHERE id=3;
```
#### 用PHP操作MySQL数据库
>步骤：
=> 建立数据库的链接
=> 选择要使用的数据库
=> 创建SQL语句
=> 执行SQL语句
=> 获取SQL执行结果
=> 处理数据结果集
=> 关闭与数据库的链接

```Operations
//连接数据库
mysql_connect(string $server,string $user_name,string $password,[bool $new_link,int $client_type]);
mysql_pconnect(...)  //长久链接
$conn = mysql_connect('locolhost','root','user_pass');

//关闭数据库
bool mysql_close([resource $link]);

//执行SQL语句
mysql_query(string $sql);

//处理查询结果集的函数
mysql_affected_rows();      //取前一次MySQL操作记录行数。
mysql_fetch_row($result);  //从查询结果集中返回一行数据。$result是执行mysql_query()之后返回的资源标识符
mysql_fetch_array($result,$type);  //从结果集中返回一行作为关联数组
mysql_fetch_assoc($result);        //该函数只将结果集作为关联数组返回

//获取字段信息
object mysql_fetch_field($result [, int field_offset]);

//选择一个数据库
mysql_select_db($database);
//取得结果集的行目数
mysql_num_rows($result);
//返回最近一次MySQL操作产生的错误文本信息
mysql_error();


//PHP程序
<?php
$host = 'localhost';    //定义服务器
$user_name = 'root';  //定义用户名
$password = 'admin';  //定义密码

$conn = mysql_connect($host,$user_name,$password);  //连接MySQL
if(!$conn){
      die('数据库连接失败：'.mysql_error());
}
mysql_select_db('test');  //连接数据库

$sql = 'select id,name,city from users';
$result = mysql_query($sql) OR die("<br/>ERROR:<b>".mysql_error()."</b><br/><br/><br/>产生问题的SQL<br/>".$sql);  //获取查询结果


if($result){
    echo 'SQL语句：'.$sql.'<br/>已经成功执行！';
    $num = mysql_num_rows($result);  //调用mysql_num_row()获取SELECT语句查询
    echo '<br/>该SQL语句查询到<b>'.$num.'</b>行数据';
}

if($num = mysql_num_rows($result))  //判断SELECT语句查找到的行数
{
    $row = mysql_fetch_array($result);  //mysql_fetch_array()将结果集中的一行作为数组返回
    echo '<pre>';   //格式化输出
    while($row = mysql_fetch_array($result,MYSQL_ASSOC))
    {
      print_r($row);    //输出每行数据
    }
}

mysql_close($conn);
?>
```
