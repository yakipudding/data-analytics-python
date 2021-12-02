---
title: 'SimpleImputer'
---

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
