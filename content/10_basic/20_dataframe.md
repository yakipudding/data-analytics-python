---
title: 'DataFrame'
---
pandasのDataFrameの扱い方

## 作成
```
list = [
  ['A','aaaa'],
  ['B','bbbb'],
]
df = pd.DataFrame(list)
```

## FileIO
### 読み込み

```py
df = pd.read_csv('../../data/test.csv', header=None, names=['head1','head2',...], encoding='utf_8')
```

項目名 | パラメータ | 備考
--- | --- | --- 
エンコード | `encoding='utf_8'` | sjisは`cp932`
ヘッダなし | `header=None` | 
ヘッダなし・任意の名前に設定 | `names=['head1','head2',...]` | `header=None`は書かなくてもよい
区切り文字 | `sep=','` | 

### 書き込み

```py
df.to_csv(filename, mode='w', header=True, index=False, encoding='utf_8')
```

項目名 | パラメータ | 備考
--- | --- | --- 
上書きモード | `mode='w'` | 追記モードは`a`
index出力なし | `index=False` | 
ヘッダ出力なし | `header=False` | 

## 行数や列名・列数の情報
項目 | 内容
--- | --- 
列情報 | `df.info()`
列名一覧 | `df.columns`
行数 | `len(df)`
行数・列数 | `df.shape`
列の型一覧 | `df.dtypes`

## 特定の型の列を抽出する
- `df.select_dtypes(include=object)`
- `df.select_dtypes(exclude='number')`
  - 数値型は`number`で指定可能

## 一部のデータを見る
項目 | 内容
--- | --- 
頭からN行 | `df.head(N)`
後ろからN行 | `df.tail(N)`
ランダム1%のデータ | `df.sample(frac=0.01).head(N)`
特定の列のデータ | `df[['列名1','列名2','列名3','列名4']]`

## クエリ
条件に一致した行を見る

項目 | 内容 | 備考
--- | --- | --- 
完全一致 | `df.query('列名=="値"')` |
先頭文字指定 | `df.query('列名.str.startswith("A")', engine='python')` | Nullがあるとエラー
末尾文字指定 | `df.query('列名.str.endswith("A")', engine='python')` | Nullがあるとエラー
正規表現 | `df.query('列名.str.contains("[1-9]$", regex=True)',engine='python')` |

## ソート

項目 | 内容 | 備考
--- | --- | --- 
基本 | `df.sort_values('列名', ascending=False)` | ascendingは昇順/降順
複数列ソート | `df.sort_values(['列名1','列名2'])` |

集計したデータをソートしたいときはreset_indexしてから行う

## 行単位の計算
項目 | 内容
--- | --- 
前行からの差分を取る | `df.diff()`
前行までの値を使って計算 | `df.expanding().mean()`

## df操作

項目 | 内容 | 備考
--- | --- | --- 
列名変更 | `df.rename(columns={'A': 'a'})` |
列削除 | `df.drop('削除列', axis = 1)` |
df縦にくっつける | `pd.concat([df1,df2], axis=0)` |
df結合 | `pd.merge(df_left, df_right, on='key', how='left')` | howはleft/right/inner/outer
dfコピー | `df_product.copy()` |
Numpy配列(narray)として取得 | `df列名.values` |
