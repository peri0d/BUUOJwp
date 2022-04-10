# \[BUUCTF 2018]Online Tool

## \[BUUCTF 2018]Online Tool

## 考点

* Nmap使用
* escapeshellarg()+escapeshellcmd() 之殇

## wp

```php
<?php

if (isset($_SERVER['HTTP_X_FORWARDED_FOR'])) {
    $_SERVER['REMOTE_ADDR'] = $_SERVER['HTTP_X_FORWARDED_FOR'];
}

if(!isset($_GET['host'])) {
    highlight_file(__FILE__);
} else {
    $host = $_GET['host'];
    $host = escapeshellarg($host);
    $host = escapeshellcmd($host);
    $sandbox = md5("glzjin". $_SERVER['REMOTE_ADDR']);
    echo 'you are in sandbox '.$sandbox;
    @mkdir($sandbox);
    chdir($sandbox);
    echo system("nmap -T5 -sT -Pn --host-timeout 2 -F ".$host);
}
```

GET参数host，然后经过`escapeshellarg()`和`escapeshellcmd()`两个函数的处理，再进行命令拼接

> **`escapeshellarg()`** 将给字符串增加一个单引号并且能引用或者转码任何已经存在的单引号，这样以确保能够直接将一个字符串传入 shell 函数，并且还是确保安全的。对于用户输入的部分参数就应该使用这个函数。
>
> **`escapeshellcmd()`** 对字符串中可能会欺骗 shell 命令执行任意命令的字符进行转义。反斜线（\）会在以下字符之前插入： <mark style="color:red;">``&#;`|*?~<>^()[]{}$\``</mark>, <mark style="color:red;">`\x0A`</mark> 和 <mark style="color:red;">`\xFF`</mark>。 <mark style="color:red;">`'`</mark> <mark style="color:red;"></mark><mark style="color:red;">和</mark> <mark style="color:red;"></mark><mark style="color:red;">`"`</mark> <mark style="color:red;"></mark><mark style="color:red;">仅在不配对儿的时候被转义</mark>。 在 Windows 平台上，<mark style="color:red;">所有这些字符以及</mark> <mark style="color:red;"></mark><mark style="color:red;">`%`</mark> <mark style="color:red;"></mark><mark style="color:red;">和</mark> <mark style="color:red;"></mark><mark style="color:red;">`!`</mark> <mark style="color:red;"></mark><mark style="color:red;">字符都会被空格代替</mark>。

命令拼接是在考nmap的参数，



## 小结

