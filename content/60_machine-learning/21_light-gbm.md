---
title: 'LightGBM'
---

大量の決定木を作成しながら学習を進める勾配ブースティング

## 特徴
- モデル訓練に掛かる時間が短い 
- メモリ効率が高い 
  - 計量値をヒストグラムとして扱うのでメモリを抑えることが可能
- 推測精度が高い 
  - 同じデータセットに対して他のブースティングの機械学習アルゴリズムと比較した場合、Leaf-Wiseのため推測精度が改善する傾向がある
  - これはLeaf-Wiseの方がLevel-Wiseと比較して、より複雑な決定木となるため
- 過学習しやすい 
  - Leaf-wiseは決定木が複雑になる
  - つまり、決定木の構造をハイパーパラメータで適切に調整しないと過学習（Overfitting）となる可能性が高いです。
  - 学習に利用しない検証用のデータに対する性能を見ながら学習を打ち切る「early stopping」を利用する
- 大規模なデータセットも訓練可能 

```py
# 学習用・検証用にデータセットを分割する
from sklearn.model_selection import train_test_split
X_train, X_valid, y_train, y_valid = train_test_split(X_train, y_train, test_size=0.3, random_state=0, stratify=y_train)
# LightGBMでは、カテゴリ変数に対して特別な処理28を自動的に実行してくれる
## カテゴリ変数をリスト形式で宣言する
categorical_features = ['Embarked', 'Pclass', 'Sex']

# 学習・予測
import lightgbm as lgb
lgb_train = lgb.Dataset(X_train, y_train, categorical_feature=categorical_features)
lgb_eval = lgb.Dataset(X_valid, y_valid, reference=lgb_train, categorical_feature=categorical_features)

params = {
    'objective': 'binary'
}

model = lgb.train(params, lgb_train,
                  valid_sets=[lgb_train, lgb_eval],
                  verbose_eval=10,
                  num_boost_round=1000,
                  early_stopping_rounds=10)

y_pred = model.predict(X_test, num_iteration=model.best_iteration)
```