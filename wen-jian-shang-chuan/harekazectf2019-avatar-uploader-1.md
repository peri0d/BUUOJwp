# \[HarekazeCTF2019]Avatar Uploader 1

## \[HarekazeCTF2019]Avatar Uploader 1

## 考点



## wp

源码是给出的，登录之后，有一个文件上传功能

先看代码，在 `config.php` 中定义了一些常量，客户端cookie中使用session字段表示PHPSESSID，定义上传目录`uploads`

```php
define('CLIENT_SESSION_ID', 'session');
define('SECRET_KEY', getenv('SECRET_KEY'));
define('UPLOAD_DIR', __DIR__ . '/uploads');
```

然后在 `upload.php` 中判断文件大小，并使用 `FILEINFO` 判断上传图片类型，上传图片只能是 png 类型

```php
// check file size
if ($_FILES['file']['size'] > 256000) {
  error('Uploaded file is too large.');
}

// check file type
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$type = finfo_file($finfo, $_FILES['file']['tmp_name']);
finfo_close($finfo);
if (!in_array($type, ['image/png'])) {
  error('Uploaded file is not PNG format.');
}
```

后面再用 `getimagesize` 判断文件像素大小，并且再进行一次类型判断，如果不是 png 类型就给出 flag

```php
// check file width/height
$size = getimagesize($_FILES['file']['tmp_name']);
if ($size[0] > 256 || $size[1] > 256) {
  error('Uploaded image is too large.');
}
if ($size[2] !== IMAGETYPE_PNG) {
  // I hope this never happens...
  error('What happened...? OK, the flag for part 1 is: <code>' . getenv('FLAG1') . '</code>');
}
```

在这两种判断上传图片类型的函数中，有一个很有趣的现象， `FILEINFO` 可以识别 png 图片( 十六进制下 )的第一行，而 `getimagesize` 不可以，代码如下

```php
<?php
$file = finfo_open(FILEINFO_MIME_TYPE);
  
var_dump(finfo_file($file, "test"));
  
$f = getimagesize("test"); 
var_dump($f[2] === IMAGETYPE_PNG);

```

结果如下

![](<../.gitbook/assets/image (10) (1) (1).png>)

![](<../.gitbook/assets/image (21) (1) (1).png>)

直接上传这个文件就可以获取 flag 了
