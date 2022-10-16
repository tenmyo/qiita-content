<!--
title:   賭け戦略シミュレーション2（定額・定率・マーチンゲール法）
tags:    Jupyter,matplotlib,pandas,シミュレーション,マーチンゲール法
id:      b7ab18876c1c1d032ca2
private: false
-->
Jupyter Notebookの練習として、各種賭け戦略のシミュレーションを行います。[前回](/tenmyo/items/a3bd89ff020503095ac2)はシミュレーション結果を折れ線グラフで見ました。今回は、100ラウンドの結果をヒストグラムと統計値で見てみます。

## 賭けのルール

- サイコロの奇数・偶数を当てる
- 好きな額を事前に賭けられる
- 当たれば、賭けた額×2が払い戻される
- 外れたら、賭けた額は没収
- 借金禁止
- 開始資金は10000コイン
- 1000回賭けられる
- 各回で、全参加者は同じ側に賭ける（当たるタイミングや回数が同一）

本記事では、勝率を約50％、52％、48％の3パターンでシミュレーションします。勝率にブレが生じないよう前回からコードを改良しています。

## 戦略の説明

- 定額：必ず一定額を賭けます
- 定率：持ち金の一定割合を賭けます

### マーチンゲール法

ある額から賭け始め、負けるたび倍に増やします（負けを取り戻せるように賭けます）。勝って負けが取り戻せたら、額に戻します。
必勝法として有名です。負けが続いた場合、たちまち賭け金が増えていくため、実際は次に示すような形で打ち切りとなることが多いでしょう。

- 手持ち資金が尽きて、賭けられなくなる
- 賭場の賭け上限にかかり、倍額を賭けられなくなる

## 結果（勝率約50%）

![50-plot.png](https://qiita-image-store.s3.amazonaws.com/0/142637/81cddf1e-4c50-54c3-99ff-822dca699f8f.png)
![50-hist.png](https://qiita-image-store.s3.amazonaws.com/0/142637/7144b9c7-636a-7758-5b0f-b29f3bb06add.png)

||マーチンゲール(10開始)|マーチンゲール(2開始)|定率(1%)|定率(3%)|定額(100)|定額(300)|
|:----|------:|------:|-----:|-----:|------:|------:|
|mean | 9890.8| 9564.4|9512.6|6380.3|10000.0| 8697.0|
|std  | 7134.8| 3715.9|   9.0|   9.3|    0.0| 3378.9|
|min  |    0.0|    0.0|9490.0|6355.0|10000.0|    0.0|
|25%  |    0.0|10994.0|9506.8|6375.0|10000.0|10000.0|
|50%  |14970.0|10998.0|9512.0|6380.0|10000.0|10000.0|
|75%  |15000.0|11000.0|9517.0|6386.0|10000.0|10000.0|
|max  |15000.0|11000.0|9544.0|6402.0|10000.0|10000.0|

## 結果（勝率約48%）

![48-plot.png](https://qiita-image-store.s3.amazonaws.com/0/142637/775ac0d6-a3d6-8d4b-43af-6d59d868ebce.png)
![48-hist.png](https://qiita-image-store.s3.amazonaws.com/0/142637/f568f01f-72e0-68cb-4bc6-d0145730b091.png)

||マーチンゲール(10開始)|マーチンゲール(2開始)|定率(1%)|定率(3%)|定額(100)|定額(300)|
|:----|------:|------:|-----:|-----:|-----:|--:|
|mean | 7039.8| 8981.7|6394.8|1933.8|6000.0|0.0|
|std  | 7385.7| 4229.4|   7.7|   6.5|   0.0|0.0|
|min  |    0.0|    0.0|6376.0|1919.0|6000.0|0.0|
|25%  |    0.0|10946.0|6390.0|1931.0|6000.0|0.0|
|50%  |    0.0|10958.0|6394.0|1934.0|6000.0|0.0|
|75%  |14790.0|10960.0|6400.0|1937.0|6000.0|0.0|
|max  |14800.0|10960.0|6417.0|1955.0|6000.0|0.0|

## 結果（勝率約52%）

![52-plot.png](https://qiita-image-store.s3.amazonaws.com/0/142637/c6ce0298-457e-2814-443d-b44d7b0fd541.png)
![52-hist.png](https://qiita-image-store.s3.amazonaws.com/0/142637/cbb7bc33-b8b1-ee9b-8adb-40f0c9f41b89.png)

||マーチンゲール(10開始)|マーチンゲール(2開始)|定率(1%)|定率(3%)|定額(100)|定額(300)|
|:---|------:|------:|------:|------:|------:|------:|
|mean|11390.4|10151.3|14172.2|21152.8|14000.0|22000.0|
|std | 6609.4| 3008.7|   12.5|   17.0|    0.0|    0.0|
|min |    0.0|    0.0|14145.0|21106.0|14000.0|22000.0|
|25% |11347.5|11034.0|14163.0|21141.0|14000.0|22000.0|
|50% |15190.0|11038.0|14171.0|21151.5|14000.0|22000.0|
|75% |15200.0|11040.0|14180.2|21163.5|14000.0|22000.0|
|max |15200.0|11040.0|14205.0|21206.0|14000.0|22000.0|

## 実験コード

```python3:賭け戦略シミュレーション2.py
# %%
# 定義
import math
import typing  # noqa: F401

import matplotlib.pyplot as plt
import numpy as np
import pandas as pd


# 戦略スーパークラス
class Storategy:
    def __init__(self, name, coin):  # type: (str, int) -> None
        self.name = name
        self.coin = coin
        self.next_bet = 0

    def bet(self):  # type: () -> int
        ret = self.next_bet
        self.coin -= ret
        return ret

    def refound(self, coin):  # type: (int) -> int
        self.coin += coin
        self.update(coin)
        return self.coin

    def update(self, coin):  # type: (int) -> None
        pass


# 定額
class FixedAmount(Storategy):
    def __init__(self, name, coin, first_bet):  # type: (str, int, int) -> None
        super().__init__(name, coin)
        self.first_bet = first_bet
        self.update(coin)

    def update(self, coin):  # type: (int) -> None
        self.next_bet = min(self.first_bet, self.coin)


# 定率
class FixedRate(Storategy):
    def __init__(self, name, coin, rate):  # type: (str, int, float) -> None
        super().__init__(name, coin)
        self.rate = rate
        self.update(coin)

    def update(self, coin):  # type: (int) -> None
        self.next_bet = math.floor(self.coin * self.rate)


# マーチンゲール法（負けたら倍賭け）
class Martingale(Storategy):
    def __init__(self, name, coin, first_bet):  # type: (str, int, int) -> None
        super().__init__(name, coin)
        self.first_bet = first_bet
        self.debt = 0
        self.update(coin)

    def update(self, coin):  # type: (int) -> None
        if coin == 0:
            self.next_bet *= 2
        else:
            self.next_bet = self.first_bet + self.debt
            self.debt = 0
        if self.next_bet > self.coin:
            self.debt = self.next_bet - self.coin
            self.next_bet = self.coin


# 勝敗（払い戻し率）リスト生成
def winning_list(winning_persentage, odds,
                 k):  # type: (float, float, int) -> np.ndarray[float]
    win = int(k * winning_persentage)
    ary = np.concatenate((np.full(win, odds),
                          np.zeros(k - win)))  # type: np.ndarray[float]
    np.random.shuffle(ary)
    return ary


# シミュレーション実施
def simulate(start_coin, num_rounds, winning_persentage, odds,
             k):  # type: (int, int, float, float, int) -> pd.DataFrame
    np.random.seed(1)  # 再現可能にする
    df = pd.DataFrame(columns=['name', 'round', 'count', 'coin'])
    for round in range(num_rounds):
        strategies = (
            FixedAmount('定額(300)', start_coin, 300),
            FixedAmount('定額(100)', start_coin, 100),
            FixedRate('定率(3%)', start_coin, 0.03),
            FixedRate('定率(1%)', start_coin, 0.01),
            Martingale('マーチンゲール(10開始)', start_coin, 10),
            Martingale('マーチンゲール(2開始)', start_coin, 2),
        )  # type: typing.Sequence[Storategy]
        wl = winning_list(winning_persentage, odds, k)

        for strategy in strategies:
            history = [strategy.coin]
            for result_odds in wl:
                bet_coin = strategy.bet()
                refound_coin = bet_coin * result_odds
                strategy.refound(refound_coin)
                history.append(strategy.coin)
            df_one = pd.DataFrame([
                np.full(len(history), strategy.name),
                np.full(len(history), round),
                np.arange(len(history)), history
            ]).T
            df_one.columns = df.columns
            df = df.append(df_one)
    return df.astype({'round': int, 'count': int, 'coin': int})


def save_plot(df, fname,
              save_rounds):  # type: (pd.DataFrame, str, int) -> None
    plt.style.use('ggplot')
    fig = plt.figure(figsize=(16, 6 * math.ceil(save_rounds / 2)))
    for round in range(save_rounds):
        ax = fig.add_subplot(math.ceil(save_rounds / 2), 2, round + 1)
        for name in df['name'].unique():
            df_one = df[(df['name'] == name) & (df['round'] == round)]
            ax.plot(df_one['count'], df_one['coin'], label=name)
        ax.legend()
        ax.set_title(f'Round {round + 1}')
    fig.savefig(fname, bbox_inches='tight')
    fig.show()


def save_hist(df, fname):  # type: (pd.DataFrame, str) -> None
    plt.style.use('ggplot')
    storategy_names = df['name'].unique()
    num_charts = len(storategy_names)
    k = df['count'].max()
    fig = plt.figure(figsize=(16, 6 * math.ceil(num_charts / 2)))
    for i, storategy_name in enumerate(storategy_names, 1):
        ax = fig.add_subplot(math.ceil(num_charts / 2), 2, i)
        df_one = df[(df['name'] == storategy_name) & (df['count'] == k)]
        ax.hist(df_one['coin'], bins=20)
        ax.set_title(storategy_name)
    fig.savefig(fname, bbox_inches='tight')
    fig.show()


# %%
# 50%
# シミュレート
num_rounds = 100
k = 1000
per = 50
df = simulate(
    start_coin=10000,
    num_rounds=num_rounds,
    winning_persentage=per/100,
    odds=2,
    k=k)
# 保存
save_plot(df, f'{per}-plot.png', 4)
save_hist(df, f'{per}-hist.png')
df[(df['count'] == k)].groupby('name')[(
    'name', 'coin')].describe().round(1).T.to_csv(f'{per}.csv')


# %%
# 48%
# シミュレート
num_rounds = 100
k = 1000
per = 48
df = simulate(
    start_coin=10000,
    num_rounds=num_rounds,
    winning_persentage=per/100,
    odds=2,
    k=k)
# 保存
save_plot(df, f'{per}-plot.png', 4)
save_hist(df, f'{per}-hist.png')
df[(df['count'] == k)].groupby('name')[(
    'name', 'coin')].describe().round(1).T.to_csv(f'{per}.csv')


# %%
# 52%
# シミュレート
num_rounds = 100
k = 1000
per = 52
df = simulate(
    start_coin=10000,
    num_rounds=num_rounds,
    winning_persentage=per/100,
    odds=2,
    k=k)
# 保存
save_plot(df, f'{per}-plot.png', 4)
save_hist(df, f'{per}-hist.png')
df[(df['count'] == k)].groupby('name')[(
    'name', 'coin')].describe().round(1).T.to_csv(f'{per}.csv')
```

## ToDo（今後のアイデアなど）

- ケリー基準と比較してみる
- トレンド＆ランダムウォークの推移シミュレーション
  - ポートフォリオ式/ドルコスト平均法(定額購入)/定数購入での比較
- ヒストグラムのスケールを統一したい
- シミュレーションの高速化