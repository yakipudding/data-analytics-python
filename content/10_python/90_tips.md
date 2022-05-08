---
title: 'その他Tips'
---

## 名前付きタプル
```py
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p1 = Point(0,1)
# Point(x=0, y=1)
```

## boolの判定
- オブジェクトに__bool__()があればそれを使用 `bool(obj)`
- なければ__len__()を呼び出す `len(obj)`
  - 0ならFalse
  - それ以外ならTrue
- 例
  - False
    - 0
    - None
    - 空文字 `''`
    - 空リスト `[]`
  - True
    - 0以外の数値
    - `len(obj)!=0` のobj

## match
**※3.10で追加。かなり新しいので注意**
いわゆるswitch文

```py
point = 1
match point:
  case 0:
    print('zero')
  case 1:
    print('one')
```

## 三項演算子
```py
ret = true_val if (条件) else false_val
```

## 演算Tips
### int同士の除算は**float**になる
cだとintになるから…と思ってるとはまる
```py
4/2
# 2.0(float)
```

## 演算子Tips
### 累乗、商の演算子
- 累乗： `**`
- 商： `//`



## インポート元のパスを増やしたい時
```py
sys.path.append(絶対パス)
```

## pass
ブロックの中で何もしないときに使う

```py
def test():
  pass
```

