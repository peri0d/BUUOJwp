# \[BSidesCF 2019]Mixer

## \[BSidesCF 2019]Mixer

## 考点

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

在修改了一部分密文后不影响后续的密文解密，
