---
title: 'データ加工'
---

DataFrameのデータ加工

## 欠損値

- 欠損値が多い場合、列ごと削除したほうが良い
- 欠損値が少ない場合、平均値などで補完したほうが良い

項目 | 内容 | 備考
--- | --- | --- 
欠損値(Null,NaN)かどうか | `df.列名.isnull()` |
欠損値の数 | `df.列名.isnull().sum()` | 
欠損値を補完する（固定値）| `df.列名.fillna('固定値', inplace=True)` |
欠損値を補完する（平均値）| `df.列名.fillna(np.mean(df.列名), inplace=True)` |

## 型変換・型操作

項目 | 内容 | 備考
--- | --- | --- 
Date→文字列 | `pd.to_datetime(df.列名).dt.strftime('%Y%m%d')` |
文字列→Date | `pd.concat([df.列名, pd.to_datetime(df.列名)], axis=1)` |
数値→文字列 | `df.列名.astype('str')` |
unix秒→Date | `pd.to_datetime(df.列名, unit='s')` |
Dateの年,月,日 | `df.列名.dt.year` | year, month, day
Dateの年,月,日(0埋め) | `df.列名.dt.strftime('%m')` |

## 置換・変換

項目 | 内容 | 備考
--- | --- | --- 
固定値→固定値 | `df.列名.replace(['A','B'], [0,1], inplace=True)` |
固定値→固定値2 | `df['列名'] = df['列名'].map({'A': 0, 'B': 1} ).astype(int)` |
文字列加工 | `df['name'].str[0:3]` |前3文字
関数を使用した変換 | `df['列名'] = df.apply(lambda x:replace_fc_sample(x),axis=1)` |
関数を使用した変換2 | `df['列名'] = df.apply(replace_fc_sample,axis=1)` |

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
