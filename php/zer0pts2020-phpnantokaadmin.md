# \[Zer0pts2020]phpNantokaAdmin

## \[Zer0pts2020]phpNantokaAdmin

## 考点

## wp

题目功能流程如下

![](<../.gitbook/assets/image (30).png>)

填参数，点下一步，最后的请求是向/?page=create发送 POST请求，请求内容如下

```
table_name=test&columns[0][name]=testa&columns[0][type]=INTEGER
```

然后请求/?page=index，来到插入数据的页面

![](<../.gitbook/assets/image (4).png>)

向/?page=insert发送POST请求，内容如下

```
values[]=1&values[]=2&values[]=3
```

然后请求/?page=index

![](<../.gitbook/assets/image (33).png>)

那么后台的语句可能如下

```sql
CREATE TABLE test(
   dummy1 TEXT,
   dummy2 TEXT,
   testa INTEGER,
);

INSERT INTO test (dummy1,dummy2,testa) 
          VALUES ('1', '2', 3);
```

但是`testa`那个字段输入字母也会成功插入

![](<../.gitbook/assets/image (1).png>)

然后找信息，啥都没找到，直接用BUU给的代码吧

三个文件，config.php定义flag位置，flag\_bf1811da表的flag\_2a2d04c3字段

util.php存放自定义函数，index.php实现路由和视图功能

