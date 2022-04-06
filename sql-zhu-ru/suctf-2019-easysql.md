# \[SUCTF 2019]EasySQL

## \[SUCTF 2019]EasySQL

## 考点

* MySQL sql\_mode
* SQL注入fuzz

## wp

* [easy\_sql github](https://github.com/team-su/SUCTF-2019/tree/master/Web/easy\_sql)
*   打开靶机，是这样的界面

    ![](https://wcgimages.oss-cn-shenzhen.aliyuncs.com/buuoj/su\_ezsql\_1.png)
*   直接用字典 fuzz 看一下过滤了哪些字符，如图

    ![](https://wcgimages.oss-cn-shenzhen.aliyuncs.com/buuoj/su\_ezsql\_2.png)
* 这个和强网杯相似，都是堆叠注入，在公开的源码中可以看到，传入的 `query` 长度不超过 40
* 关键的查询代码是 `select $post['query']||flag from Flag`
*   输入 1 或 0 查询结果如图，要想办法让 `||` 不是逻辑或

    ![](https://wcgimages.oss-cn-shenzhen.aliyuncs.com/buuoj/su\_ezsql\_3.png)
* 官方给的 payload 是 `1;set sql_mode=PIPES_AS_CONCAT;select 1`
* 拼接一下就是 `select 1;set sql_mode=PIPES_AS_CONCAT;select 1||flag from Flag`
* 关于 `sql_mode` : 它定义了 MySQL 应支持的 SQL 语法，以及应该在数据上执行何种确认检查，其中的 `PIPES_AS_CONCAT` 将 `||` 视为字符串的连接操作符而非 "或" 运算符
* 关于 `sql_mode` 更多可以查看这个链接 : [MySQL sql\_mode 说明](https://www.cnblogs.com/piperck/p/9835695.html)
*   还有就是这个模式下进行查询的时候，使用字母连接会报错，使用数字连接才会查询出数据，因为这个 `||` 相当于是将 `select 1` 和 `select flag from flag` 的结果拼接在一起

    ![](https://wcgimages.oss-cn-shenzhen.aliyuncs.com/buuoj/su\_ezsql\_4.png)
* 关于非预期解 : `*,1`
* 拼接一下，不难理解 : `select *,1||flag from Flag`
*   等同于 `select *,1 from Flag`

    ![](https://wcgimages.oss-cn-shenzhen.aliyuncs.com/buuoj/su\_ezsql\_5.png)

## 小结

1. sql\_mode的 PIPES\_AS\_CONCAT 将 || 视为字符串的连接操作符而非 "或" 运算符
2. 在fuzz时需要注意的页面有<mark style="background-color:orange;">正常页面</mark>、<mark style="background-color:orange;">语句执行错误页面</mark>、<mark style="background-color:orange;">waf页面</mark>
