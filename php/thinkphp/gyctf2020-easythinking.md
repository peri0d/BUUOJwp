# \[GYCTF2020]EasyThinking

## \[GYCTF2020]EasyThinking

## 考点

* ThinkPHP V6.0.0 任意文件写的漏洞

## wp

打开靶机后有很多功能，登录、注册、搜索、登出等等。随便注册账号测试测试功能，都是可以使用的。

目录扫描可以扫到www.zip

尝试访问`robots.txt`提示系统错误，并且是ThinkPHP V6.0.0

```
控制器不存在:app\home\controller\robots\Txt
ThinkPHP V6.0.0 { 十年磨一剑-为API开发设计的高性能框架 } - 官方手册 
```

ThinkPHP V6.0.0有个任意文件写的漏洞，可以直接写shell。

这个漏洞的利用条件是

1、session开启

2、要能写入session

回归到这一题，用了注册登录肯定开了session，而搜索那里在登出后就清空了，也就是说可能是搜索内容是写入session的。

接着拿到了源码，发现session位置是`/runtime/session/`，搜索内容也确实会写入session

那就修改PHPSESSID为`aaaaaaaaaaaaaaaaaaaaaaaaaaaa.php`，然后在搜索框写shell`<?php @eval($_POST['a']);?>`，最后访问`runtime/session/sess_aaaaaaaaaaaaaaaaaaaaaaaaaaaa.php`进行getshell

![](<../../.gitbook/assets/image (3).png>)
