# \[HarekazeCTF2019]encode\_and\_encode

## \[HarekazeCTF2019]encode\_and\_encode

## 考点

* PHP在json\_decode时会自动解码传入的Unicode编码

## wp

Source Code给了源码

```php
<?php
error_reporting(0);
  
if (isset($_GET['source'])) {
	show_source(__FILE__);
	exit();
}
  
function is_valid($str) {
	$banword = [
      // no path traversal
      '\.\.',
      // no stream wrapper
      '(php|file|glob|data|tp|zip|zlib|phar):',
      // no data exfiltration
      'flag'
    ];
    $regexp = '/' . implode('|', $banword) . '/i';
    if (preg_match($regexp, $str)) {
      return false;
    }
	return true;
}
  
$body = file_get_contents('php://input');
$json = json_decode($body, true);
  
if (is_valid($body) && isset($json) && isset($json['page'])) {
    $page = $json['page'];
    $content = file_get_contents($page);
    if (!$content || !is_valid($content)) {
      $content = "<p>not found</p>\n";
    }
  } else {
	$content = '<p>invalid request</p>';
  }
  
// no data exfiltration!!!
$content = preg_replace('/HarekazeCTF\{.+\}/i', 'HarekazeCTF{&lt;censored&gt;}', $content);
echo json_encode(['content' => $content]); 
```

`file_get_contents('php://input')` 获取 post 的数据，`json_decode($body, true)` 用 json 格式解码 post 的数据并返回数组，然后 `is_valid($body)` 对 post 数据检验，大概输入的格式如下

![](../../.gitbook/assets/Harekaze2019\_encode\_2.png)



`is_valid($body)` 对 post 数据检验，导致无法传输 `$banword` 中的关键词，也就无法传输 `flag`，这里在 json 中，可以使用 Unicode 编码绕过，`flag` 就等于 `\u0066\u006c\u0061\u0067`

通过检验后，获取 `page` 对应的文件，并且页面里的内容也要通过 `is_valid` 检验，然后将文件中 `HarekazeCTF{}` 替换为 `HarekazeCTF{&lt;censored&gt;}` ，这样就无法明文读取 flag

这里传入 `/\u0066\u006c\u0061\u0067` 后，由于 `flag` 文件中也包含 flag 关键字，所以返回 `not found` ，这也无法使用 `file://`

![](../../.gitbook/assets/Harekaze2019\_encode\_3.png)

`file_get_contents` 是可以触发 `php://filter` 的，所以考虑使用伪协议读取，对 `php` 的过滤使用 `Unicode` 绕过即可

![](../../.gitbook/assets/Harekaze2019\_encode\_4.png)
