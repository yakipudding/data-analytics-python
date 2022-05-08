---
title: '基本操作'
---

## リストから作成
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

## CSV読み込み

```py
path = '../../data/test.csv'
header = None # ヘッダなし（namesで指定すること）
names = ['type','text']
encoding='utf_8' # sjisはcp932
sep = ','
df = pd.read_csv(path, header=None, names=names, encoding=encoding, sep=sep)
```

## CSV書き込み

```py
mode = 'w' # 追記ならa
header = True # ヘッダを出力するかどうか
index = False # index列を出力するかどうか
encoding='utf_8' # sjisはcp932
df.to_csv(filename, mode=mode, header=header, index=index, encoding=encoding)
```

## 行数や列名・列数の情報
### dfの情報
```py
df.info()
```

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 3 entries, 0 to 2
Data columns (total 4 columns):
 #   Column  Non-Null Count  Dtype  
---  ------  --------------  -----  
 0   type    3 non-null      object 
 1   text    3 non-null      object 
 2   id      3 non-null      int64  
 3   length  3 non-null      float64
dtypes: float64(1), int64(1), object(2)
memory usage: 224.0+ bytes
```

### 列名
```py
df.columns
# Index(['type', 'text', 'id', 'length'], dtype='object')
```

### 行数
```py
len(df)
# 3
```

### 行数・列数
```py
df.shape
# (3, 4)
```

### 列の型一覧
```py
df.dtypes
```

```
type       object
text       object
id          int64
length    float64
dtype: object
```

## 列操作

### 列名変更
```py
df.rename(columns={'type': 'type_a'})
```

index | type_a | text | id | length
 --- | --- | --- | --- | ---
 0 | A | aaaa | 1 | 0.1
 1 | B | bbbb | 2 | 0.2
 2 | C | cccc | 3 | 0.3

### 列削除
```py
df.drop('length', axis = 1)
```

index | type | text | id |
 --- | --- | --- | --- |
 0 | A | aaaa | 1 |
 1 | B | bbbb | 2 |
 2 | C | cccc | 3 |

## 結合
### UNION
```py
pd.concat([df,df], axis=0)
```

index | type | text | id | length
 --- | --- | --- | --- | ---
 0 | A | aaaa | 1 | 0.1
 1 | B | bbbb | 2 | 0.2
 2 | C | cccc | 3 | 0.3
 0 | A | aaaa | 1 | 0.1
 1 | B | bbbb | 2 | 0.2
 2 | C | cccc | 3 | 0.3

### JOIN
how: left/right/inner/outer
```py
pd.merge(df, df, on='type', how='left')
```

index | type | text_x | id_x | length_x | text_y | id_y | length_y
 --- | --- | --- | --- | --- | --- | --- | ---
 0 | A | aaaa | 1 | 0.1 | aaaa | 1 | 0.1
 1 | B | bbbb | 2 | 0.2 | bbbb | 2 | 0.2
 2 | C | cccc | 3 | 0.3 | cccc | 3 | 0.3

## コピー
```py
df.copy()
```

## numpy.ndarrayとして取得
```py
df.values
```

```
array([['A', 'aaaa', 1, 0.1],
       ['B', 'bbbb', 2, 0.2],
       ['C', 'cccc', 3, 0.3]], dtype=object)
```
