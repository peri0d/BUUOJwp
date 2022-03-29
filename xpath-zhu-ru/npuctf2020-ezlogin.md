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

![](../.gitbook/assets/图片.png)
