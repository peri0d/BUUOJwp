# \[GWCTF 2019]枯燥的抽奖

## \[GWCTF 2019]枯燥的抽奖

## 考点

* 使用`php_mt_seed`破解`mt_rand`伪随机

## wp

F12在js文件发现check.php

```php
x3zUcxWZKA
<?php
#这不是抽奖程序的源代码！不许看！
header("Content-Type: text/html;charset=utf-8");
session_start();
if(!isset($_SESSION['seed'])){
$_SESSION['seed']=rand(0,999999999);
}

mt_srand($_SESSION['seed']);
$str_long1 = "abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ";
$str='';
$len1=20;
for ( $i = 0; $i < $len1; $i++ ){
    $str.=substr($str_long1, mt_rand(0, strlen($str_long1) - 1), 1);       
}
$str_show = substr($str, 0, 10);
echo "<p id='p1'>".$str_show."</p>";


if(isset($_POST['num'])){
    if($_POST['num']===$str){x
        echo "<p id=flag>抽奖，就是那么枯燥且无味，给你flag{xxxxxxxxx}</p>";
    }
    else{
        echo "<p id=flag>没抽中哦，再试试吧</p>";
    }
}
show_source("check.php"); 
```

使用了`mt_srand`给随机数生成器播种，然后使用`mt_rand`生成`20`个随机数，然后根据这20个随机数读取编码表（`str_long1`）中对应的字符，返回前`10`个字符。

使用脚本转换，得到前10个随机数对应的格式

```python
str1 = 'abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ'
str2 = 'znXCVCNqS5'
str3 = str1[::-1]
length = len(str2)
res = ''
for i in range(len(str2)):
    for j in range(len(str1)):
        if str2[i] == str1[j]:
            res += str(j) + ' ' + str(j) + ' ' + '0' + ' ' + str(len(str1) - 1) + ' '
            break
print(res)
# 27 27 0 61 34 34 0 61 30 30 0 61 25 25 0 61 56 56 0 61 36 36 0 61 17 17 0 61 18 18 0 61 15 15 0 61 15 15 0 61
```

然后使用[php\_mt\_seed](https://www.openwall.com/php\_mt\_seed/)破解

```
./php_mt_seed 27 27 0 61 34 34 0 61 30 30 0 61 25 25 0 61 56 56 0 61 36 36 0 61 17 17 0 61 18 18 0 61 15 15 0 61 15 15 0 61
```

![](<../../.gitbook/assets/image (18) (1).png>)

得到结果999490007，根据返回的版本（7.1+），源码放到对应版本重新执行一遍即可

```php
<?php
mt_srand(999490007);

$str_long1 = "abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ";
$str='';
$len1=20;
for ( $i = 0; $i < $len1; $i++ ){
    $str.=substr($str_long1, mt_rand(0, strlen($str_long1) - 1), 1);       
}
echo $str;
?>
```

## 小结

1. [\[MRCTF2020\]Ezaudit](mrctf2020-ezaudit.md)也是考伪随机数
