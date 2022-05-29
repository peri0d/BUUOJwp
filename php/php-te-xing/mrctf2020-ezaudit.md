# \[MRCTF2020]Ezaudit

## \[MRCTF2020]Ezaudit

## 考点

* 使用`php_mt_seed`破解`mt_rand`伪随机
* 万能密码

## wphp

目录扫描存在www.zip，解压出index.php，给了公钥，要知道私钥才能登录

```php
// genarate public_key 
function public_key($length = 16) {
    $strings1 = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
    $public_key = '';
    for ( $i = 0; $i < $length; $i++ )
    $public_key .= substr($strings1, mt_rand(0, strlen($strings1) - 1), 1);
    return $public_key;
  }

//genarate private_key
  function private_key($length = 12) {
    $strings2 = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
    $private_key = '';
    for ( $i = 0; $i < $length; $i++ )
    $private_key .= substr($strings2, mt_rand(0, strlen($strings2) - 1), 1);
    return $private_key;
  }
  $Public_key = public_key();
  //$Public_key = KVQP0LdJKRaV3n9D
```

使用了mt\_rand获取随机字符，而它是伪随机的，可以用php\_mt\_seed破解。

先把公钥转换

```python
str1 = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'
str2 = 'KVQP0LdJKRaV3n9D'
str3 = str1[::t-1]
res = ''
for i in range(len(str2)):
    for j in range(len(str1)):
        if str2[i] == str1[j]:
            res += str(j) + ' ' + str(j) + ' ' + '0' + ' ' + str(len(str1) - 1) + ' '
            break
print(res)

# 36 36 0 61 47 47 0 61 42 42 0 61 41 41 0 61 52 52 0 61 37 37 0 61 3 3 0 61 35 35 0 61 36 36 0 61 43 43 0 61 0 0 0 61 47 47 0 61 55 55 0 61 13 13 0 61 61 61 0 61 29 29 0 61
```

用php\_mt\_seed计算随机数种子，得到结果`1775196155`

![](<../../.gitbook/assets/image (22) (1) (1) (1) (1) (1) (1) (1).png>)

在返回头可以看到PHP版本为5.6.40，在该版本下用得到的随机数种子运行private\_key()即可

```php
<?php
mt_srand(1775196155);
// genarate public_key 
function public_key($length = 16) {
    $strings1 = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
    $public_key = '';
    for ( $i = 0; $i < $length; $i++ )
    $public_key .= substr($strings1, mt_rand(0, strlen($strings1) - 1), 1);
    return $public_key;
  }

  //genarate private_key
  function private_key($length = 12) {
    $strings2 = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
    $private_key = '';
    for ( $i = 0; $i < $length; $i++ )
    $private_key .= substr($strings2, mt_rand(0, strlen($strings2) - 1), 1);
    return $private_key;
  }
echo public_key()."<br>";
echo private_key();
// KVQP0LdJKRaV3n9D
// XuNhoueCDCGc
?>
```

再用万能密码登录即可

![](<../../.gitbook/assets/image (24) (1) (1) (1) (1) (1) (1) (1).png>)
