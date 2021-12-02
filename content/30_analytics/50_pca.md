---
title: '主成分分析'
---

複数次元の数値データを、相関のある列同士をまとめて主な成分としてまとめること

- 複数次元の数値データの次元を削減することができる
- 主成分分析した値を、次元削減した特徴量として扱うこともできる

```py
from sklearn import datasets
from sklearn.decomposition import PCA

# データセットに対してPCAを適用する
# 主成分の数は、少なくとも全体の分散の80%が説明できる水準となるよう自動的に選ばれる
pca = PCA(n_components=0.8)
pca.fit(df)
# データを主成分空間に写像
feature = pca.transform(df)

#  寄与率（第N主成分まででどのくらいの情報を説明できているか）
pd.DataFrame(pca.explained_variance_ratio_, index=["PC{}".format(x + 1) for x in range(len(dfs.columns))])

```

このあたりが参考になるかも
[https://qiita.com/maskot1977/items/082557fcda78c4cdb41f](https://qiita.com/maskot1977/items/082557fcda78c4cdb41f)