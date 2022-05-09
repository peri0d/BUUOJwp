# \[pasecactf\_2019]flask\_ssti

## \[pasecactf\_2019]flask\_ssti <a href="#2671280054" id="2671280054"></a>

## 考点

* Flask SSTI 读取config
* 根据加密代码写解密代码

## wp

输入一个name会回显，这里就是SSTI了

![](<../../.gitbook/assets/image (29) (1).png>)

看给的代码，只是一部分代码

```python
def encode(line, key, key2):
    return ''.join(chr(x ^ ord(line[x]) ^ ord(key[::-1][x]) ^ ord(key2[x])) for x in range(len(line)))
app.config['flag'] = encode('', 'GQIS5EmzfZA1Ci8NslaoMxPXqrvFB7hYOkbg9y20W3', 'xwdFqMck1vA0pl7B8WO3DrGLma4sZ2Y6ouCPEHSQVT')
```

把flag用key和key2异或再写入到config中，那就先读取config

![](<../../.gitbook/assets/image (17) (1).png>)

在最后面看到flag

```
'flag': '-M7\x10wH57dU*`.\x0e\x1a\x02j\x02(DN\x12\x08\x17 mSd\x02^E\\ \n.zF`\x15Y\x16G'
```

密文是flag经过一系列异或运算得到的，用相同的代码就可以得到flag， encode 脚本即为 decode 脚本

```python
def encode(line, key, key2):
    return ''.join(chr(x ^ ord(line[x]) ^ ord(key[::-1][x]) ^ ord(key2[x])) for x in range(len(line)))

enc = "-M7\x10wH57dU*`.\x0e\x1a\x02j\x02(DN\x12\x08\x17 mSd\x02^E\\ \n.zF`\x15Y\x16G"
flag = encode(enc, 'GQIS5EmzfZA1Ci8NslaoMxPXqrvFB7hYOkbg9y20W3', 'xwdFqMck1vA0pl7B8WO3DrGLma4sZ2Y6ouCPEHSQVT')

print(flag)
```

另外，buu 上题目描述的代码有误。
