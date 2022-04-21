# \[0CTF 2016]piapiapia

## \[0CTF 2016]piapiapia

## 考点

* phar反序列化

## wp

www.zip泄露源码，下载审计。

index.php有登录功能，先看index.php。包含class.php，使用session判断是否登录，登录了就跳转profile.php

```php
<?php
require_once('class.php');
if($_SESSION['username']) {
    header('Location: profile.php');
    exit;
}
```

然后到登录功能，获取的username和password长度必须在\[3,16]这个区间内，然后使用`$user->login`判断账号密码是否正确，这个函数在class.php中。

```php
if($_POST['username'] && $_POST['password']) {
	$username = $_POST['username'];
	$password = $_POST['password'];
	if(strlen($username) < 3 or strlen($username) > 16) 
		die('Invalid user name');
	if(strlen($password) < 3 or strlen($password) > 16) 
		die('Invalid password');
	if($user->login($username, $password)) {
		$_SESSION['username'] = $username;
		header('Location: profile.php');
		exit;}
	else {  die('Invalid user name or password');  }
}else{}
```

到class.php找$user的定义。

```
// Some code
```
