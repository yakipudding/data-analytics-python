---
title: 'Seaborn'
---

Matplotlibをベースに、設定やデザインをいい感じにできる可視化のライブラリ

参考：[seabornの細かい見た目調整をあきらめない](https://qiita.com/skotaro/items/7fee4dd35c6d42e0ebae)

## imports
```py
pd.plotting.register_matplotlib_converters()
import matplotlib.pyplot as plt
%matplotlib inline
import seaborn as sns
```

- `pd.plotting.register_matplotlib_converters()`：pandas型変換
- `%matplotlib inline`：Notebookでインラインに画像表示

## レイアウト

項目 | 内容 | 備考
--- | --- | ---
デフォルトスタイル適用 | `sns.set()` |
細かいレイアウト設定 | `seaborn.set_style(style=None, rc=None)` | 詳しくは[チュートリアル](http://seaborn.pydata.org/tutorial/aesthetics.html#overriding-elements-of-the-seaborn-styles)
大きさ | `seaborn.set_context(context=None, font_scale=1, rc=None)` | context: paper < notebook（デフォルト） < talk < poster
色 | `sns.set_palette()`

## グラフ設定の基本
seabornはグラフの種類ごとに返すmatplotlibのオブジェクトが異なる

項目 | 内容 | 返却オブジェクト | 返却オブジェクトが持つmatplotlibのオブジェクト
--- | --- | --- | ---
relplot | 散布図をグループごとに色分けしたりできる | FacetGrid | figure
catplot | カテゴリごとの分布とかをいろいろなグラフで表現できる | FacetGrid | figure
jointplot | 2変数間の分布を可視化できる。散布図の上右にヒストグラム表示したり | JointGrid | figure
pairplot | 散布図の行列 | PairGrid | figure
lmplot | 散布図と回帰モデルを重ねられる | FacetGrid | figure
そのほか | | | axes

- figureレベル関数
  - figureとaxは自動生成されるため、事前にはアクセスできない
  - 戻り値の`grid.fig`や`grid.fig.axes`にアクセスして設定する
- axesレベル関数
  - figureはデフォルトで現在のfigureが採用されるため、事前に設定できる
  - axもデフォルトで現在のaxが採用されるため、事前に設定できる

### Grid系グラフの設定例
```py
grid = sns.lmplot(x="x", y="y", ... )
fig = grid.fig
axes = grid.axes

# grid.figでfigureオブジェクトにアクセスできる
grid.fig.suptitle('suptitle via grid.fig', y=1.02)

# grid.axesはnumpy arrayなのでravelかflatで一次元化します
for ax, title in zip(grid.axes.ravel(), ['a', 'b', 'c', 'd']):
    ax.set_title(f'changed title {title}')

```

### 非Grid系グラフの設定例
```py
# axesレベル関数の例
sns.set(style="darkgrid")
fmri = sns.load_dataset("fmri")
plt.figure(figsize=(4, 3)) # デフォルトだと大きいので小さめに
ax = sns.lineplot(x="timepoint", y="signal",
                  hue="region", style="event",
                  data=fmri)
ax.set_title('title test')
ax.set_xlabel('time (msec)')

# lineplot実行後ならax = plt.gca()でも同じAxesを取得できる
print(id(ax) == id(plt.gca()))
# True
```

## 画像保存
```py
line_plot = sns.lineplot()
figure = line_plot.get_figure()
figure.savefig(save_path)
```