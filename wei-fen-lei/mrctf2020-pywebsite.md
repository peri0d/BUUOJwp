# \[MRCTF2020]PYWebsite

## \[MRCTF2020]PYWebsite

## wp

F12可以看到JS代码

```javascript
    function enc(code){
      hash = hex_md5(code);
      return hash;
    }
    function validate(){
      var code = document.getElementById("vcode").value;
      if (code != ""){
        if(hex_md5(code) == "0cd4da0223c0b280829dc3ea458d655c"){
          alert("您通过了验证！");
          window.location = "./flag.php"
        }else{
          alert("你的授权码不正确！");
        }
      }else{
        alert("请输入授权码");
      }
      
    }
```

直接访问flag.php提示`我已经把购买者的IP保存了，显然你没有购买`

改XFF再访问

payload：`X-Forwarded-For: 127.0.0.1`
