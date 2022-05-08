---
title: 'クラスタリング'
---

複数次元の数値データを、ベクトルが近いもの同士をまとめてクラスタリングすること。

- 複数次元の数値データの次元を削減することができる
- クラスタリングした値を特徴量として扱ったりする
- k-meansクラスタリングがよく用いられる

## k-meansクラスタリング
3次元のデータを5つのクラスターに分類する例

```py
from sklearn.cluster import KMeans
clusters = KMeans(n_clusters=5, random_state=1).fit_predict(df[['数値列1','数値列2','数値列3']])
df['clusters'] = clusters

# プロット
fig2 = plt.figure()
ax = fig2.add_subplot(111, projection='3d')
ax.scatter(df['数値列1'], df['数値列2'], df['数値列3'], c=clusters, cmap='Spectral')
```
