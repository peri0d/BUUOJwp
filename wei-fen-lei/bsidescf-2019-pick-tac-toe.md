# \[BSidesCF 2019]Pick Tac Toe

## \[BSidesCF 2019]Pick Tac Toe

## 考点

* 逻辑漏洞

## wp

是个井字棋，先手没办法必赢。

F12看下代码，九个格子用了不同的变量定义

![](<../.gitbook/assets/image (25) (1) (1).png>)

抓包看看，是传入不同的value实现下棋的功能

![](<../.gitbook/assets/image (9).png>)

ul就是第一个格子，电脑下在了中心就是c这个变量\


![](<../.gitbook/assets/image (22) (1) (1) (1).png>)



如果在第二次下棋，传入电脑下过的位置，就会覆盖掉电脑的棋子

![](<../.gitbook/assets/image (33) (1) (1) (1).png>)

![](<../.gitbook/assets/image (29) (1).png>)

再下最后一个就可以了
