# Wallbreaker\_Easy

## Wallbreaker\_Easy

## 考点

## wp

TCTF2019 WallBreaker-Easy

提示

> Imagick is a awesome library for hackers to break `disable_functions`.&#x20;
>
> So I installed php-imagick in the server, opened a `backdoor` for you.
>
> Let's try to execute `/readflag` to get the flag.&#x20;
>
> Open basedir: /var/www/html:/tmp/ebe1d01ac021c4a3e68eb5bbdc842e69
>
> Hint: eval($\_POST\["backdoor"]);

给了shell，要命令执行，但是有disable\_functions。这题就是bypass disable\_functions

![](<../.gitbook/assets/image (7).png>)







## 小结

1. [TCTF2019 WallBreaker-Easy 解题分析](https://xz.aliyun.com/t/4688)
