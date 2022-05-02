# \[极客大挑战 2019]Http

## \[极客大挑战 2019]Http

## 考点

* http请求头

## wp

抓包发现隐藏链接Secret.php

![](<../../.gitbook/assets/image (25) (1) (1) (1).png>)

直接访问，提示`It doesn't come from 'https://Sycsecret.buuoj.cn'`

修改HTTP头`Referer: https://Sycsecret.buuoj.cn`，然后提示Please use "Syclover" browser

修改`User-Agent: Syclover`，提示`No!!! you can only read this locally!!!`

修改`X-Forwarded-For: 127.0.0.1`即可

## 小结

1. [部分HTTP头解释](https://itbilu.com/other/relate/EJ3fKUwUx.html#http-request-headers)
2. [HTTP头参数详解及其中的危险](https://www.cnblogs.com/builder4ever/p/11797358.html)
