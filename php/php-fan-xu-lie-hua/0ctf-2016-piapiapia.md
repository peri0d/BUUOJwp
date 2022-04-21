# \[0CTF 2016]piapiapia

## \[0CTF 2016]piapiapia

## 考点

* phar反序列化
* 反序列化逃逸

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

到class.php找`$user`的定义。在config.php中定义了`$config`数组用于保存MySQL连接的相关参数。

```php
require('config.php');
session_start();

class mysql {
	private $link = null;
	public function filter($string) {
		$escape = array('\'', '\\\\');
		$escape = '/' . implode('|', $escape) . '/';
		$string = preg_replace($escape, '_', $string);

		$safe = array('select', 'insert', 'update', 'delete', 'where');
		$safe = '/' . implode('|', $safe) . '/i';
		return preg_replace($safe, 'hacker', $string);
	}
	public function select($table, $where, $ret = '*') {
		$sql = "SELECT $ret FROM $table WHERE $where";
		$result = mysql_query($sql, $this->link);
		return mysql_fetch_object($result);
	}
}
class user extends mysql{
	private $table = 'users';
	public function login($username, $password) {
		$username = parent::filter($username);
		$password = parent::filter($password);

		$where = "username = '$username'";
		$object = parent::select($this->table, $where);
		if ($object && $object->password === md5($password)) {
			return true;
		} else {
			return false;
		}
	}
}
$user = new user();
$user->connect($config);
```

在`mysql::filter`处把`'`和`\\`替换成`_`，把`select`，`insert`，`update`，`where`和`delete`替换成`hacker`，可能会有反序列化逃逸。

再看注册，和登录类似

```php
require_once('class.php');
if($_POST['username'] && $_POST['password']) {
	$username = $_POST['username'];
	$password = $_POST['password'];
	if(strlen($username) < 3 or strlen($username) > 16) 
		die('Invalid user name');
	if(strlen($password) < 3 or strlen($password) > 16) 
		die('Invalid password');
	if(!$user->is_exists($username)) {
		$user->register($username, $password);
		echo 'Register OK!<a href="index.php">Please Login</a>';		
	}else {  die('User name Already Exists');  }}
```

调用了is\_exists和register函数，

```php
class user extends mysql{
	private $table = 'users';
	public function is_exists($username) {
		$username = parent::filter($username);
		$where = "username = '$username'";
		return parent::select($this->table, $where);
	}
	public function register($username, $password) {
		$username = parent::filter($username);
		$password = parent::filter($password);
		$key_list = Array('username', 'password');
		$value_list = Array($username, md5($password));
		return parent::insert($this->table, $key_list, $value_list);
	}}
class mysql {
	private $link = null;
	public function select($table, $where, $ret = '*') {
		$sql = "SELECT $ret FROM $table WHERE $where";
		$result = mysql_query($sql, $this->link);
		return mysql_fetch_object($result);
	}
	public function insert($table, $key_list, $value_list) {
		$key = implode(',', $key_list);
		$value = '\'' . implode('\',\'', $value_list) . '\''; 
		$sql = "INSERT INTO $table ($key) VALUES ($value)";
		return mysql_query($sql);
	}}
```

然后看profile.php，调用user::show\_profile()函数，然后反序列化profile和文件读取。

```php
<?php
	require_once('class.php');
	if($_SESSION['username'] == null) {
		die('Login First');	
	}
	$username = $_SESSION['username'];
	$profile=$user->show_profile($username);
	if($profile  == null) {
		header('Location: update.php');
	}
	else {
		$profile = unserialize($profile);
		$phone = $profile['phone'];
		$email = $profile['email'];
		$nickname = $profile['nickname'];
		$photo = base64_encode(file_get_contents($profile['photo']));
?>
```

user::show\_profile()

```
// Some code
```
