# \[ASIS 2019]Unicorn shop

## \[ASIS 2019]Unicorn shop

## 考点

* Unicode欺骗

## wp

是个购物的功能，输入id=4\&price=2222，提示`Only one char(?) allowed!`

也就是说要找到一个字符，它的值大于1337，到[Unicode Table](https://unicode-table.com/en/)上找一个即可

比如`U+137C`，到最下面找UTF-8编码即可

![](<../../.gitbook/assets/image (25) (1).png>)

payload：`id=4&price=%E1%8D%BC`

## 小结

1. [\[HCTF 2018\]admin](../../python/flask/hctf-2018-admin.md)也是类似的思路
