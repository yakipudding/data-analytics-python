---
title: '統計指標値・集計'
---

## 元のDF
```py
import pandas as pd
li = [
  ['A','aaaa', 1, 0.1],
  ['B','bbbb', 2, 0.2],
  ['C','cccc', 3, 0.3],
]
columns = ['type', 'text', 'id', 'length']
df = pd.DataFrame(li, columns=columns)
```

index | type | text | id | length
 --- | --- | --- | --- | ---
 0 | A | aaaa | 1 | 0.1
 1 | B | bbbb | 2 | 0.2
 2 | C | cccc | 3 | 0.3

## 統計値まとめ
```py
df.describe(include='all')
```

- | type | text | id | length
--- | --- | --- | --- | ---
count | 3 | 3 | 3.0 | 3.00
unique | 3 | 3 | NaN | NaN
top | A | aaaa | NaN | NaN
freq | 1 | 1 | NaN | NaN
mean | NaN | NaN | 2.0 | 0.20
std | NaN | NaN | 1.0 | 0.10
min | NaN | NaN | 1.0 | 0.10
25% | NaN | NaN | 1.5 | 0.15
50% | NaN | NaN | 2.0 | 0.20
75% | NaN | NaN | 2.5 | 0.25
max | NaN | NaN | 3.0 | 0.30

## 行数
### 欠損値を含めた行数
```py
len(df)
```

### 欠損値を除いた行数
```py
df.count()
```

## 統計関数
- df全体に対して統計関数を使用すると列ごとの結果を出力する
- 列名を指定したSeriesに対しても統計関数をかけられる

### 数値列向け
最大
```py
df.max()
```

最小
```py
df.min()
```

合計
```py
df.sum()
```

平均
```py
# NaNを除外する場合はskipna=True
df.mean()
```

中央
```py
df.median()
```

標準偏差(母数=標本の時)
```py
df.std(ddof=0)
```

不偏標準偏差(母数=標本の一部の時)
```py
df.std()
```

### object列向け

ユニーク値一覧
```py
df.type.unique()
# array(['A', 'B', 'C'], dtype=object)
```

ユニーク値の個数
```py
df.type.nunique()
# 3
```

ユニーク値ごとの個数
```py
df.type.value_counts()
```

```
A    1
B    1
C    1
Name: type, dtype: int64
```

## 差分計算
前行からの差分を取る
```py
df.diff()
```

前行までの値を使って計算
```py
df.expanding().mean()
```

## 集計

集計(単一キー)
```py
df.groupby('type')
```

集計(複数キー)
```py
df.groupby(['type','id'])
```

集計＋統計関数
```py
df.groupby('type').sum()
```

集計＋統計関数(複数列に異なる統計関数)
```py
df.groupby('type').agg({'id':'sum', 'length':'mean'})
```

集計＋統計関数(異なる統計関数)
```py
df.groupby('type').agg(['sum','count'])
```

各行に統計値をセットしたい時はtransformを使う
```py
df['target_mean'] = df.groupby('type').id.transform('mean')
```

groupbyを行うと列名がMultiIndex(タプル)になるため、必要に応じて列名変更を行う
```py
df_g.columns = ["_".join(pair) for pair in df_g.columns]
```

## ピボット（クロス集計）
```py
# 集計：列
grouping_rows_index = ['type']
# 集計：行
grouping_rows_columns = ['length']
## 集計：値
pivot_values = 'id'
## 統計関数：sum/count/max/min/mean
pivot_value_fc = 'count' 
## 集計行を表示するか
view_sum = True
pivot_df = pd.pivot_table(df, index=grouping_rows_index, columns=grouping_rows_columns, values=pivot_values, aggfunc=pivot_value_fc, margins=view_sum)
```

length | 0.1 | 0.2 | 0.3 | All
-- | -- | -- | -- | --
type |  |  |  | 
A | 1.0 | NaN | NaN | 1
B | NaN | 1.0 | NaN | 1
C | NaN | NaN | 1.0 | 1
All | 1.0 | 1.0 | 1.0 | 3

## ランク

```py
df.列名.rank(ascending=False, method='first')
```

## パーセンタイル

1/4分位数
```py
df.列名.quantile(0.25)
```

0,1/4,1/2,3/4,1分位数
```py
df.列名.quantile([0, 0.25, 0.5, 0.75, 1])
```

別パターン
```py
df.列名.quantile(q=np.arange(5)/4)
```

パーセンタイルで分類する例
```py
df_quantile = df.列名.quantile(q=np.arange(5)/4)
df['category'] = df.列名.apply(lambda x: 1 if x < df_quantile[0.25] else 2 if x < df_quantile[0.50] else 3 if df_quantile[0.75] else 4)
```