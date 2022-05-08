---
title: '置換・型変換'
---

## 型変換・型操作

### 基本

```py
df.列名.astype('str')
```

### 日付型
pandasの日付型は標準モジュールのdatetimeではなくpandasのdatetime
datetime列のdtオブジェクトは標準モジュールのdatetimeとして扱える
参考：https://pandas.pydata.org/docs/user_guide/timeseries.html

日付(string)列をpandasのDatetimeに変換

```py
pd.to_datetime(df.date_str)
```

日付列をstringに変換（フォーマット）

```py
pd.date_column.dt.strftime('%Y%m%d')
```

## 欠損値の補完
 
固定値
```py
df.列名.fillna('固定値', inplace=True)
```

平均値で補完
```py
df.列名.fillna(np.mean(df.列名), inplace=True)
```

## 置換・変換

### 置換
replace、mapが使える

```py
df.列名.replace(['A','B'], [0,1])
```

```py
df.列名.map({'A': 0, 'B': 1})
```

### 変換
applyで任意の関数が使用できる

```py
df.apply(lambda x:replace_fc_sample(x),axis=1)
```

```py
df.apply(replace_fc_sample,axis=1)
```

変換用の関数サンプル

```py
def replace_fc_sample(x):
    if x.open < x.close:
        return 1
    elif x.open > x.close:
        return 2
    else:
        return 0
```

## ダミー変数化
カテゴリ変数をダミー変数化する

```py
df_dummy = pd.get_dummies(df)
```
