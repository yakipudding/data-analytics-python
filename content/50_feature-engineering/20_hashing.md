---
title: 'ハッシュ化'
---

潜在的に無限に広がる値を有限のm種類の値に割り当てること。

- 例えば文字列を一定の幅の数値に変換する。
- ハッシュ化してしまうと元の値が何だったかわからなくなるため、考察が難しくなるため注意

```py
from sklearn.feature_extraction import FeatureHasher

# ハッシュ化
h = FeatureHasher(n_features=m, input_type='string')
f = h.transform(review_df['business_id'])
f.toarray()
```