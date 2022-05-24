# \[BSidesCF 2019]Mixer

## \[BSidesCF 2019]Mixer

## 考点

* 攻击ECB加密模式

## wp

随便输入，登录，提示signature和rack.session不是考察点，并且要把is\_admin改成1

![](<../.gitbook/assets/image (22).png>)

登录时请求的URL为`/?action=login&first_name=aaaa&last_name=bbbb`

登录后的cookie如下，去掉rack.session

```
user=a90e6740e375b9984967c3b8e56ab0b94cad614ff047478f947f9ac1faa36a84194c8b0116104663ca1187c77513d7b89208e22d7e596f516e88cef6b397fc27;
```

修改user的值为user=aaaaaaaaaa，提示data not multiple of block length

在原来的基础上修改为`aaaaaaaae375b9984967c3b8e56ab0b94cad614ff047478f947f9ac1faa36a84194c8b0116104663ca1187c77513d7b89208e22d7e596f516e88cef6b397fc27`

提示内容`Error parsing JSON: 765: unexpected token at '-?????>??~???]PIaaa","last_name":"bbbb","is_admin":0}'`

说明原来的密文解密后是`{"first_name":"aaaa","last_name":"bbbb","is_admin":0}`这种形式

在修改了一部分密文后不影响后续的密文解密，典型的ECB模式，分组长度一般是8字节，16字节，32字节。

假设本题分组长度为8字节，通过调整first\_name，可以得到如下分组

```
{"first_
name":"a
aaaaaaaa
","last_
name":"b
bbb","is
_admin":
0}
```

修改first\_name为a1.00000}，分组如下

```
{"first_
name":"a
1.00000}
","last_
name":"b
bbb","is
_admin":
0}
```

再将密文分成8组，第三组和最后一组对调，解密得到的明文如下，用`*`代表padding

```
{"first_
name":"a
0}******
","last_
name":"b
bbb","is
_admin":
1.00000}
```

这样就满足题目要求了，测试一下，发现不对

再试试分组长度为16的情况

```
{"first_name":"a
1.0000000000000}
","last_name":"b
bbb","is_admin":
0}
```

得到密文`6c7de5751ce3ab56d3961b695c269643936bbdf01255242e236e2d1d99602ea408f95041eca831ff2b2cfd77210d8fd6a164192494b6804a83ab8daa9bc1503849d11304091af7bdb1de04be0c27ea8a`

将密文分成5组，第二组和最后一组对调

```
6c7de5751ce3ab56d3961b695c269643
400473caf782906ced101bc4cf86e782
baf68ee937fc060a8574b7bdbfdbc7be
186fbf3e093dc9d0c4fb70e8ce2d3bb6
f199d6b133223ed30b735584b65c2003
```

![](<../.gitbook/assets/image (37).png>)
