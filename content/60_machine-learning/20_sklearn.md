---
title: 'Sklearn'
---

機械学習ライブラリ
clfで宣言するモデルを切り替えるだけで機械学習アルゴリズムを差し替えられて便利

## model
```py
# ロジスティック回帰
from sklearn.linear_model import LogisticRegression
clf = LogisticRegression(penalty='l2', solver='sag', random_state=0)

# ランダムフォレスト
from sklearn.ensemble import RandomForestClassifier
clf = RandomForestClassifier(n_estimators=100, max_depth=2, random_state=0, n_estimators=epoch)

# XGBoost(勾配ブースティング)
## LightGBMの方が後発
from xgboost import XGBRegressor
clf = XGBRegressor(n_estimators=epoch)

```

## modelのパラメーター
- learning_rate：学習率
- n_jobs：並列実行数。コア数を設定

# 学習・予測
```py
clf.fit(X_train, y_train)
predictions = clf.predict(X_test)
```

## fitのパタメーター
- n_estimators: epoch
- early_stopping_rounds
    - 最適なepochを見つけるための設定
    - 低スコアが何回続いたら停止するかを設定できる
    - epochを高くしてearly_stopping_roundsを5とかに設定しておくとオートチューニングしてくれる
    - validation用のデータはeval_set=[(X_valid, y_valid)]で設定

# 効果測定
```py
## 平均絶対誤差
from sklearn.metrics import mean_absolute_error
print("Mean Absolute Error: " + str(mean_absolute_error(predictions, y_valid)))
```