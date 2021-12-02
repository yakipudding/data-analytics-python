---
title: 'カテゴリ値のエンコーディング'
---

カテゴリ値を機械学習の特徴量として扱うためにエンコーディングする手法

## 方法1. object列をdrop
カーディナリティが高かったり（ユニーク値の数が多い）、欠損値が多いなど、精度に貢献しないようであれば消しちゃうのもあり

## 方法2. Label Encoding
カテゴリ値を数字に変換するだけ
  - 事前にラベルの種類が決まっている場合に用いる
  - 学習で出現していないラベルが検証で出現すると例外が発生するので注意

```py
from sklearn.preprocessing import LabelEncoder

# Make copy to avoid changing original data 
label_X_train = X_train.copy()
label_X_valid = X_valid.copy()

# Apply label encoder to each column with categorical data
label_encoder = LabelEncoder()
for col in object_cols:
    label_X_train[col] = label_encoder.fit_transform(X_train[col])
    label_X_valid[col] = label_encoder.transform(X_valid[col])

print("MAE from Approach 2 (Label Encoding):") 
print(score_dataset(label_X_train, label_X_valid, y_train, y_valid))
```

## 方法3. one-hot Encoding
カーディナリティが低い列(10以下など)に対して行うのが一般的
未知のラベルにも対応可能

```py
from sklearn.preprocessing import OneHotEncoder

# object_cols: one-hotエンコーディング対象列

# Apply one-hot encoder to each column with categorical data
OH_encoder = OneHotEncoder(handle_unknown='ignore', sparse=False)
OH_cols_train = pd.DataFrame(OH_encoder.fit_transform(X_train[object_cols]))
OH_cols_valid = pd.DataFrame(OH_encoder.transform(X_valid[object_cols]))

# One-hot encoding removed index; put it back
OH_cols_train.index = X_train.index
OH_cols_valid.index = X_valid.index

# Remove categorical columns (will replace with one-hot encoding)
num_X_train = X_train.drop(object_cols, axis=1)
num_X_valid = X_valid.drop(object_cols, axis=1)

# Add one-hot encoded columns to numerical features
OH_X_train = pd.concat([num_X_train, OH_cols_train], axis=1)
OH_X_valid = pd.concat([num_X_valid, OH_cols_valid], axis=1)

print("MAE from Approach 3 (One-Hot Encoding):") 
print(score_dataset(OH_X_train, OH_X_valid, y_train, y_valid))
```

## 方法4. CountEncoding
出現回数でエンコーディング（1回出たカテゴリ値→1, 100回出たカテゴリ値→100など）
- 出現回数の低いカテゴリ値を1や2にまとめることができる
- 出現回数の多いカテゴリ値はたいていバラバラに分類される
- 共通・重要な値は独自のグループ化を取得できる
```py
import category_encoders as ce
cat_features = ['category', 'currency', 'country']

# Create the encoder
count_enc = ce.CountEncoder(cols=cat_features)

# Transform the features, rename the columns with the _count suffix, and join to dataframe
count_encoded = count_enc.fit_transform(ks[cat_features])
data = data.join(count_encoded.add_suffix("_count"))

# Train a model 
train, valid, test = get_data_splits(data)
train_model(train, valid)
```

## 方法5. TargetEncoding
出現確率でエンコーディング
- 出現回数の少ない値の分散を減らします
```py
# Create the encoder
target_enc = ce.TargetEncoder(cols=cat_features)
target_enc.fit(train[cat_features], train['outcome'])

# Transform the features, rename the columns with _target suffix, and join to dataframe
train_TE = train.join(target_enc.transform(train[cat_features]).add_suffix('_target'))
valid_TE = valid.join(target_enc.transform(valid[cat_features]).add_suffix('_target'))

# Train a model
train_model(train_TE, valid_TE)
```

## 方法6. CatBoostEncoding
行ごとに、ターゲット確率はその前の行からのみ計算される。
```py
# Create the encoder
target_enc = ce.CatBoostEncoder(cols=cat_features)
target_enc.fit(train[cat_features], train['outcome'])

# Transform the features, rename columns with _cb suffix, and join to dataframe
train_CBE = train.join(target_enc.transform(train[cat_features]).add_suffix('_cb'))
valid_CBE = valid.join(target_enc.transform(valid[cat_features]).add_suffix('_cb'))

# Train a model
train_model(train_CBE, valid_CBE)
```