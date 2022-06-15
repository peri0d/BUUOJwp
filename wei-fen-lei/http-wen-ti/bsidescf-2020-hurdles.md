# \[BSidesCF 2020]Hurdles

## \[BSidesCF 2020]Hurdles

## 考点

* HTTP请求头
* Burp修改HTTP请求头
* curl使用

## wp

只给了一句话

```
You'll be rewarded with a flag if you can make it over some /hurdles.
```

访问`/hurdles`

```
I'm sorry, I was expecting the PUT Method.
```

使用curl进行PUT请求，`curl -X PUT http://node4.buuoj.cn:25154/hurdles`

```
I'm sorry, Your path would be more exciting if it ended in !
```

要以!结尾，`curl -X PUT http://node4.buuoj.cn:25154/hurdles/!`&#x20;

```
I'm sorry, Your URL did not ask to `get` the `flag` in its query string.
```

要传参get=flag，`curl -X PUT http://node4.buuoj.cn:25154/hurdles/!?get=flag`

```
I'm sorry, I was looking for a parameter named &=&=&
```

要传入参数`&=&=&`，先URL编码再传，`curl -X PUT "http://node4.buuoj.cn:25154/hurdles/!?get=flag&%26%3d%26%3d%26=1"`

```
I'm sorry, I expected '&=&=&' to equal '%00
'
```

`&=&=&`要和`%00回车`相同，`curl -X PUT "http://node4.buuoj.cn:25154/hurdles/!?get=flag&%26%3d%26%3d%26=%2500%0a"`

```
I'm sorry, Basically, I was expecting the username player.
```

需要player用户，`curl -X PUT "http://node4.buuoj.cn:25154/hurdles/!?get=flag&%26%3d%26%3d%26=%2500%0a" -u player:1`

```
I'm sorry, Basically, I was expecting the password of the hex representation of the md5 of the string 'open sesame'
```

密码是`open sesame`的md5值，`curl -X PUT "http://node4.buuoj.cn:25154/hurdles/!?get=flag&%26%3d%26%3d%26=%2500%0a" -u player:54ef36ec71201fdf9d1423fd26f97f6b`

```
I'm sorry, I was expecting you to be using a 1337 Browser.
```

UA要是`1337 Browser`，`curl -X PUT "http://node4.buuoj.cn:25154/hurdles/!?get=flag&%26%3d%26%3d%26=%2500%0a" -u player:54ef36ec71201fdf9d1423fd26f97f6b -A "1337 Browser"`

```
I'm sorry, I was expecting your browser version (v.XXXX) to be over 9000!
```

UA版本大于9000，`curl -X PUT "http://node4.buuoj.cn:25154/hurdles/!?get=flag&%26%3d%26%3d%26=%2500%0a" -u player:54ef36ec71201fdf9d1423fd26f97f6b -A "1337 Browser v.9001"`

```
I'm sorry, I was eXpecting this to be Forwarded-For someone!
```

改XFF为127.0.0.1，`curl -X PUT "http://node4.buuoj.cn:25154/hurdles/!?get=flag&%26%3d%26%3d%26=%2500%0a" -u player:54ef36ec71201fdf9d1423fd26f97f6b -A "1337 Browser v.9001" -H "X-Forwarded-For:127.0.0.1"`

```
I'm sorry, I was eXpecting this to be Forwarded For someone through another proxy!
```

要用代理和额外的代理转发，随便输一个，`curl -X PUT "http://node4.buuoj.cn:25154/hurdles/!?get=flag&%26%3d%26%3d%26=%2500%0a" -u player:54ef36ec71201fdf9d1423fd26f97f6b -A "1337 Browser v.9001" -H "X-Forwarded-For:0.0.0.0,127.0.0.1"`

```
I'm sorry, I was expecting the forwarding client to be 13.37.13.37
```

改成`13.37.13.37`，`curl -X PUT "http://node4.buuoj.cn:25154/hurdles/!?get=flag&%26%3d%26%3d%26=%2500%0a" -u player:54ef36ec71201fdf9d1423fd26f97f6b -A "1337 Browser v.9001" -H "X-Forwarded-For:13.37.13.37,127.0.0.1"`

```
I'm sorry, I was expecting a Fortune Cookie
```

再加个cookie，`curl -X PUT "http://node4.buuoj.cn:25154/hurdles/!?get=flag&%26%3d%26%3d%26=%2500%0a" -u player:54ef36ec71201fdf9d1423fd26f97f6b -A "1337 Browser v.9001" -H "X-Forwarded-For:13.37.13.37,127.0.0.1" --cookie "Fortune=1"`

```
I'm sorry, I was expecting the cookie to contain the number of the HTTP Cookie (State Management Mechanism) RFC from 2011.
```

cookie要包含2011年State Management Mechanism的RFC编号，这个编号是6265，`curl -X PUT "http://node4.buuoj.cn:25154/hurdles/!?get=flag&%26%3d%26%3d%26=%2500%0a" -u player:54ef36ec71201fdf9d1423fd26f97f6b -A "1337 Browser v.9001" -H "X-Forwarded-For:13.37.13.37,127.0.0.1" --cookie "Fortune=6265"`

```
I'm sorry, I expect you to accept only plain text media (MIME) type.
```

改Accept为text/plain，`curl -X PUT "http://node4.buuoj.cn:25154/hurdles/!?get=flag&%26%3d%26%3d%26=%2500%0a" -u player:54ef36ec71201fdf9d1423fd26f97f6b -A "1337 Browser v.9001" -H "X-Forwarded-For:13.37.13.37,127.0.0.1" --cookie "Fortune=6265" -H "Accept:text/plain"`

```shell
I'm sorry, Я ожидал, что вы говорите по-русски.
```

翻译一下，意思是`对不起，我希望你会说俄语。`，`curl -X PUT "http://node4.buuoj.cn:25154/hurdles/!?get=flag&%26%3d%26%3d%26=%2500%0a" -u player:54ef36ec71201fdf9d1423fd26f97f6b -A "1337 Browser v.9001" -H "X-Forwarded-For:13.37.13.37,127.0.0.1" --cookie "Fortune=6265" -H "Accept:text/plain" -H "Accept-Language:ru"`

```
I'm sorry, I was expecting to share resources with the origin https://ctf.bsidessf.net
```

修改Origin，`curl -X PUT "http://node4.buuoj.cn:25154/hurdles/!?get=flag&%26%3d%26%3d%26=%2500%0a" -u player:54ef36ec71201fdf9d1423fd26f97f6b -A "1337 Browser v.9001" -H "X-Forwarded-For:13.37.13.37,127.0.0.1" --cookie "Fortune=6265" -H "Accept:text/plain" -H "Accept-Language:ru" -H "Origin:https://ctf.bsidessf.net"`

```
I'm sorry, I was expecting you would be refered by https://ctf.bsidessf.net/challenges?
```

修改Referer，`curl -X PUT "http://node4.buuoj.cn:25154/hurdles/!?get=flag&%26%3d%26%3d%26=%2500%0a" -u player:54ef36ec71201fdf9d1423fd26f97f6b -A "1337 Browser v.9001" -H "X-Forwarded-For:13.37.13.37,127.0.0.1" --cookie "Fortune=6265" -H "Accept:text/plain" -H "Accept-Language:ru" -H "Origin:https://ctf.bsidessf.net" -H "Referr:https://ctf.bsidessf.net/challenges"`

```
Congratulations!
```

再加个`-i`参数即可

![](<../../.gitbook/assets/image (20).png>)

使用burp如下

![](<../../.gitbook/assets/image (10) (1).png>)
