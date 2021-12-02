---
title: '統計指標値・集計'
---

## 統計指標

項目 | 内容 | 備考
--- | --- | --- 
統計情報まとめ | `df.describe(include='all')` |
最大 | `df.列名.max()` |
最小 | `df.列名.min()` |
合計 | `df.列名.sum()` |
平均 | `df.列名.mean()` | Nullを除外する場合はskipna=True
中央 | `df.列名.median()` |
最頻値 | `df.列名.apply(lambda x: x.mode())` |
標準偏差 | `df.列名.std(ddof=0)` | 母数=標本の時
不偏標準偏差 | `df.列名.std()` | 母数=標本の一部の時
ユニーク値一覧 | `df.列名.unique()` | ['A','B','C']
ユニーク値の個数 | `df.列.nunique()` | 3
ユニーク値ごとの個数 | `df.列名.value_counts()` | [{'A':1}, {'B':2}, {'C':3} ]
欠損値以外の個数 | `df.列名.count()` |

## 集計

項目 | 内容 | 備考 
--- | --- | --- 
集計キー1つ | `df.groupby('key')` |
集計キー複数 | `df.groupby(['key1','key2'])` |
集計後に計算(複数列) | `df.groupby('key').agg({'列名1':'計算1', '列名2':'計算1'})` | 計算は`sum/count/max/min/mean/median/std/nunique`
集計後に計算(同じ列で複数計算) | `df.groupby('key1').列名.agg(['計算1','計算2'])` | ※2

- 数値
  - sum / max / min / mean / median / std
- カテゴリ
  - count
  - nunique

※1 groupbyしたあとはreset_index()しないとソートとかはできない
※2 集計後に計算(同じ列で複数計算)した後は、列名がpairになっているため列名を編集する

```py
df_g.columns = ["_".join(pair) for pair in df_g.columns]
```

GroupByで各行に合計値をセットしたい時はtransformを使う

```py
df['target_mean'] = df.groupby('key')['target'].transform('mean')
```

## ピボット（クロス集計）

```py
# 集計：列
grouping_rows_index = ['sex']
# 集計：行
grouping_rows_columns = ['学年']
## 集計：値
pivot_values = 'id' 
## 値関数：sum/count/max/min/mean
pivot_value_fc = 'count' 
## 集計行を表示するか
view_sum = True
pivot_df = pd.pivot_table(df, index=grouping_rows_index, columns=grouping_rows_columns, values=pivot_values, aggfunc=pivot_value_fc, margins=view_sum)
```

ピボットテーブルを展開する場合はstackを使う
→行名がlevel_1とか0になるので修正する

```py
pivot_df2 = pivot_df2.set_index('age_rank').stack().reset_index().rename(columns={'level_1':'gender_cd', 0: 'amount'})
```

## ランク

```py
df_rank = pd.concat([df, df["列名"].rank(ascending=False, method='first')], axis=1).rename(columns={'列名': 'rank'})
df_rank.sort_values('rank', ascending=False)
```

## パーセンタイル

```py
df.列名.quantile(0.25)
# 引数(1/4)分位数が出力される

df.列名.quantile([0, 0.25, 0.5, 0.75, 1])
# 1/4分位数が出力される
df.列名.quantile(q=np.arange(5)/4)
# ↑と同じ
```

パーセンタイルで分類する
```py
df_quantile = df.列名.quantile(q=np.arange(5)/4)
df['category'] = df.列名.apply(lambda x: 1 if x < df_quantile[0.25] else 2 if x < df_quantile[0.50] else 3 if df_quantile[0.75] else 4)
```