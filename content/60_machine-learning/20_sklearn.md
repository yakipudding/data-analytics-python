---
title: 'Sklearn'
---

機械学習ライブラリ
clfで宣言するモデルを切り替えるだけで機械学習アルゴリズムを差し替えられて便利

## model
- パラメーター
  - learning_rate：学習率
  - n_jobs：並列実行数。コア数を設定

### ロジスティック回帰
```py
from sklearn.linear_model import LogisticRegression
clf = LogisticRegression(penalty='l2', solver='sag', random_state=0)
```

### ランダムフォレスト
```py
from sklearn.ensemble import RandomForestClassifier
clf = RandomForestClassifier(n_estimators=100, max_depth=2, random_state=0, n_estimators=epoch)
```

### XGBoost(勾配ブースティング)
```py
from xgboost import XGBRegressor
clf = XGBRegressor(n_estimators=epoch)
```

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

## sklearn.impute.SimpleImputer
```py
from sklearn.impute import SimpleImputer
my_imputer = SimpleImputer()
# storategy: mean(default) / median / most_frequent / constant: fill_value(default=None)をセット

# fit_transform: trainのデータでfit(mean計算) + transform
imputed_X_train = pd.DataFrame(my_imputer.fit_transform(X_train))
# validはtrainで計算したmeanを使用してtransformする
imputed_X_valid = pd.DataFrame(my_imputer.transform(X_valid))

# Imputation removed column names
imputed_X_train.columns = X_train.columns
imputed_X_valid.columns = X_valid.columns
```

## Pipeline
一連の処理(欠損値補完・エンコーディングなど）をまとめて定義・実行できる
  - 例えば前処理をtrainとtestそれぞれに変換をかけなくて良いので楽
  - 参考：https://www.kaggle.com/alexisbcook/pipelines

## 定義
```py
# preprocessor
from sklearn.compose import ColumnTransformer
from sklearn.pipeline import Pipeline
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import OneHotEncoder

# 数値型：欠損値補完はNone
numerical_transformer = SimpleImputer(strategy='constant')

# Preprocessing for categorical data
categorical_transformer = Pipeline(steps=[
    ('imputer', SimpleImputer(strategy='most_frequent')),
    ('onehot', OneHotEncoder(handle_unknown='ignore'))
])

# Bundle preprocessing for numerical and categorical data
preprocessor = ColumnTransformer(
    transformers=[
        ('num', numerical_transformer, numerical_cols),
        ('cat', categorical_transformer, categorical_cols)
    ])

# model定義
from sklearn.ensemble import RandomForestRegressor
model = RandomForestRegressor(n_estimators=100, random_state=0)

# pipeline
from sklearn.metrics import mean_absolute_error
my_pipeline = Pipeline(steps=[('preprocessor', preprocessor),
                              ('model', model)
                             ])
```

## 1パターンで実行
```py
# 前処理・モデル作成
my_pipeline.fit(X_train, y_train)

# 推論
preds = my_pipeline.predict(X_valid)

# スコア
score = mean_absolute_error(y_valid, preds)
print('MAE:', score)
```

## クロスバリデーションで実行
```py
from sklearn.model_selection import cross_val_score
# Multiply by -1 since sklearn calculates *negative* MAE
scores = -1 * cross_val_score(my_pipeline, X, y,
                              cv=5,
                              scoring='neg_mean_absolute_error')
print("MAE scores:\n", scores)
print("Average MAE score (across experiments):")
print(scores.mean())
```