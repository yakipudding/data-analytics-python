---
title: '関数'
---

## 基本的な記法
```py
def hello_world():
  print('hello world')
```

## 引数リストのアンパック
```py
def get_args(*args):
  print(args)

get_args(1,2,3)
# (1, 2, 3)
```

## キーワード引数をdictで受け取る
```py
def get_keys(**keys):
  print(keys)

get_keys(a=0, b=1, c=2)
# {'a': 0, 'b': 1, 'c': 2}
```

## lambda
```py
lambda x: x + 1
# def f(x): return x + 1
```