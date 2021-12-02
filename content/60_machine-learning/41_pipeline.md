---
title: ' Pipeline'
---

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