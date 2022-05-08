---
title: 'クエリ'
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


## string
- 文字列型の列の場合、`str`でstringオブジェクトとして扱える
  - Dtypeはobject, stringに対して有効
  - `startswith` `endswith` `contains` 等のメソッドが使用できる
  - スライスも使用できる
  - ただし文字列型以外の値（Noneなど）があると例外が発生するので注意すること

スライスによる文字列加工

```py
df.text.str[:2]
```

```
0    aa
1    bb
2    cc
Name: text, dtype: object
```

startswithの例

```py
df.query('text.str.startswith("aa")', engine='python')
```

index | type | text | id | length
 --- | --- | --- | --- | ---
 0 | A | aaaa | 1 | 0.1

containsで正規表現を使うこともできる

```py
df.query('text.str.contains("[a-b]$", regex=True)',engine='python')
```

index | type | text | id | length
 --- | --- | --- | --- | ---
 0 | A | aaaa | 1 | 0.1
 1 | B | bbbb | 2 | 0.2

## datetime
- 日付系の型の列の場合、`dt`でdatetime系オブジェクトとして扱える
  - リファレンス：https://pandas.pydata.org/docs/user_guide/timeseries.html#
  - 例えばdatetime型なら標準モジュールのdatetime.datetimeと同じメソッドが扱える

```py
df.date.dt.year
```

日付の書式

```py
df.date.dt.strftime('%Y/%m/%d')
```

## 行指定

### 頭からN行
```py
df.head(1)
```
index | type | text | id | length
 --- | --- | --- | --- | ---
 0 | A | aaaa | 1 | 0.1

### 後ろから1行
```py
df.tail(1)
```

index | type | text | id | length
 --- | --- | --- | --- | ---
 2 | C | cccc | 3 | 0.3

### ランダムサンプリング（30%）
```py
df.sample(frac=0.3)
```

index | type | text | id | length
 --- | --- | --- | --- | ---
 1 | B | bbbb | 2 | 0.2

### 欠損値
欠損値(Null,NaN)の抽出

```py
df[df.type.isnull()]
```

index | type | text | id | length
 --- | --- | --- | --- | ---

欠損値の数

```py
df.type.isnull().sum()
```

## 行指定（クエリ）
```py
df.query('クエリ文字列')
```

### クエリ条件（比較演算子）
`==` `!=` `<` `>` 等は普通に使える

```py
df.query('type=="A"')
```

index | type | text | id | length
 --- | --- | --- | --- | ---
 0 | A | aaaa | 1 | 0.1

### 条件に変数を用いる
`@`を使用すると変数が使える

```py
id_a = 1
df.query('id==@id_a')
```

index | type | text | id | length
 --- | --- | --- | --- | ---
 0 | A | aaaa | 1 | 0.1

### Noneの扱い
- Noneを取り除く方法
  - クエリとは別に、先に`notnull()`で抽出しておく
  - もしくは、クエリ文字列で`列名==列名`をする
    - Falseになるため、Noneの行は含まれなくなる

## 列指定
```py
cols = ['type', 'id']
df[cols]
```

index | type | id |
 --- | --- | --- |
 0 | A | 1 |
 1 | B | 2 |
 2 | C | 3 |

### object列の抽出
```py
df.select_dtypes(include=object)
```

index | type | text |
 --- | --- | --- |
 0 | A | aaaa |
 1 | B | bbbb |
 2 | C | cccc |

### 数値系以外の列の抽出
```py
# numberだと数値系をまとめて指定できる
df.select_dtypes(exclude='number')
```

index | type | text |
 --- | --- | --- |
 0 | A | aaaa |
 1 | B | bbbb |
 2 | C | cccc |

## 行列指定
### loc
- 行列の指定（indexの値, 列名）
  - 固定値
  - 複数値（リスト）
  - スライス

```py
df.loc[0, 'text']
# aaaa
```

```py
df.loc[[0, 1], ['text','id']]
```

index | text | id
 --- | --- | ---
 0 | aaaa | 1
 1 | bbbb | 2

```py
df.loc[:, 'text':'length']
```

index | text | id | length
 --- | --- | --- | ---
 0 | aaaa | 1 | 0.1
 1 | bbbb | 2 | 0.2
 2 | cccc | 3 | 0.3

### iloc
- 行列の指定（行番号, 列番号）
  - 固定値
  - 複数値（リスト）
  - スライス

```py
df.iloc[0, 1]
# aaaa
```

```py
df.loc[[0, 1], ['text','id']]
```

## ソート
※集計したデータをソートしたいときはreset_indexしてから行う

```py
df.sort_values('type', ascending=False)
```

index | type | text |
 --- | --- | --- |
 2 | C | cccc |
 1 | B | bbbb |
 0 | A | aaaa |

### 複数列ソート
```py
df.sort_values(['type','id'])
```

index | type | text |
 --- | --- | --- |
 0 | A | aaaa |
 1 | B | bbbb |
 2 | C | cccc |

