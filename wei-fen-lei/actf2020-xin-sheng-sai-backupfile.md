# \[ACTF2020 新生赛]BackupFile

## \[ACTF2020 新生赛]BackupFile

## wp

提示找到备份文件，找到`index.php.bak`

```php
<?php
include_once "flag.php";

if(isset($_GET['key'])) {
    $key = $_GET['key'];
    if(!is_numeric($key)) {
        exit("Just num!");
    }
    $key = intval($key);
    $str = "123ffwsfwefwf24r2f32ir23jrw923rskfjwtsw54w3";
    if($key == $str) {
        echo $flag;
    }
}
else {
    echo "Try to find out source file!";
}
```

`key`是数字，然后使用`intval()`获取`key`的十进制值，与`str`弱相等

GET `?key=123`
