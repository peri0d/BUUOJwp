# \[CISCN2021 Quals]upload

### \[CISCN2021 Quals]upload

## 考点



## wp

### index.php

给了源码

```php
<?php
$ctf = $_GET["ctf"];

if($ctf=="upload") {
    if ($_FILES['postedFile']['size'] > 1024*512) {
        die("这么大个的东西你是想d我吗？");
    }
    $imageinfo = getimagesize($_FILES['postedFile']['tmp_name']);
    if ($imageinfo === FALSE) {
        die("如果不能好好传图片的话就还是不要来打扰我了");
    }
    if ($imageinfo[0] !== 1 && $imageinfo[1] !== 1) {
        die("东西不能方方正正的话就很讨厌");
    }
    $fileName=urldecode($_FILES['postedFile']['name']);
    if(stristr($fileName,"c") || stristr($fileName,"i") || stristr($fileName,"h") || stristr($fileName,"ph")) {
        die("有些东西让你传上去的话那可不得了");
    }
    $imagePath = "image/" . mb_strtolower($fileName);
    if(move_uploaded_file($_FILES["postedFile"]["tmp_name"], $imagePath)) {
        echo "upload success, image at $imagePath";
    } else {
        die("传都没有传上去");
}}
```

先看index.php内容，GET传ctf为upload，文件上传的参数名要是postedFile

可以用Python传

```python
from urllib3 import encode_multipart_formdata
import requests

def sendFile(filePath):
    url = "http://aa97a970-180f-4260-b3e3-fd92f6fd02bc.node4.buuoj.cn:81/?ctf=upload"
    f = open(filePath, "rb")
    file = {
        "postedFile": ("123.png", f.read()),
    }
    
    encodeData = encode_multipart_formdata(file)
    fileData = encodeData[0]
    
    headersFromData = {
        "Content-Type": encodeData[1],
    }
    
    res = requests.post(url=url, headers=headersFromData, data=fileData)
    return res
    

if __name__=='__main__':
    res = sendFile('123.png')
    print(res.text)
```

可以用`#define %s %d`的方式绕过`getimagesize`对宽高的限制。它的底层实现是，如果某一行格式满足`#define %s %d`，那么取出其中的字符串和数字，再从字符串中取出`width`或`height`，将数字作为图片的长和宽。

```
#define width 1
#define height 1
```

然后文件名不能包含`i h p ph`这四个字符串，最后把文件名所有字符转成小写存储

### example.php

```php
 <?php
$ctf = $_GET["ctf"];
if($ctf=="poc") {
    $zip = new \ZipArchive();
    $name_for_zip = "example/" . $_POST["file"];
    if(explode(".",$name_for_zip)[count(explode(".",$name_for_zip))-1]!=="zip") {
        die("要不咱们再看看？");
    }
    if ($zip->open($name_for_zip) !== TRUE) {
        die ("都不能解压呢");
    }
    echo "可以解压，我想想存哪里";
    $pos_for_zip = "/tmp/example/" . md5($_SERVER["REMOTE_ADDR"]);
    $zip->extractTo($pos_for_zip);
    $zip->close();
    unlink($name_for_zip);
    $files = glob("$pos_for_zip/*");
    foreach($files as $file){
        if (is_dir($file)) {
            continue;
        }
        $first = imagecreatefrompng($file);
        $size = min(imagesx($first), imagesy($first));
        $second = imagecrop($first, ['x' => 0, 'y' => 0, 'width' => $size, 'height' => $size]);
        if ($second !== FALSE) {
            $final_name = pathinfo($file)["basename"];
            imagepng($second, 'example/'.$final_name);
            imagedestroy($second);
        }
        imagedestroy($first);
        unlink($file);
}}
```



## 小结

