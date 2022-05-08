---
title: 'クラス'
---

## 基本的な記法
```py
class CustomClass:
  def __init__(self, x):
    # インスタンス変数
    self.x = x

  # メソッド
  def get_x(self):
    return self.x
```

## 継承
```py
class CustomSubClass(CustomClass):
  # ...
```