---
title: 'モデル作成・テスト用のデータ分割'
---

## train_test_split
```py
from sklearn.model_selection import train_test_split
## stratifyはtrainとtestで割合を同じにしたい列。正解ラベルなど。
## random_stateはシード値
df_train, df_test = train_test_split(df_customer, test_size=0.1, stratify=df_customer['gender'])

## X,yそれぞれの場合
train_X, val_X, train_y, val_y = train_test_split(X,y,test_size=0.1, random_state=1)
```

## アンダーサンプリング
数が少ない方にあわせてサンプリング

```py
#unbalancedのubUnderを使った例
df_tmp = df_receipt.groupby('customer_id').agg({'amount':'sum'}).reset_index()
df_tmp = pd.merge(df_customer, df_tmp, how='left', on='customer_id')
df_tmp['buy_flg'] = df_tmp['amount'].apply(lambda x: 0 if np.isnan(x) else 1)

print('0の件数', len(df_tmp.query('buy_flg == 0')))
print('1の件数', len(df_tmp.query('buy_flg == 1')))

positive_count = len(df_tmp.query('buy_flg == 1'))

rs = RandomUnderSampler(random_state=71)

df_sample, _ = rs.fit_sample(df_tmp, df_tmp.buy_flg)

print('0の件数', len(df_sample.query('buy_flg == 0')))
print('1の件数', len(df_sample.query('buy_flg == 1')))
```

## KFold
K-分割交差検証

```py
from sklearn.model_selection import KFold
import lightgbm as lgb

y_preds = []
models = []
oof_train = np.zeros((len(X_train),))
# データ分割
cv = KFold(n_splits=5, shuffle=True, random_state=0)

categorical_features = ['Embarked', 'Pclass', 'Sex']

params = {
    'objective': 'binary',
    'max_bin': 300,
    'learning_rate': 0.05,
    'num_leaves': 40
}

for fold_id, (train_index, valid_index) in enumerate(cv.split(X_train)):
    X_tr = X_train.loc[train_index, :]
    X_val = X_train.loc[valid_index, :]
    y_tr = y_train[train_index]
    y_val = y_train[valid_index]

    lgb_train = lgb.Dataset(X_tr, y_tr,
                            categorical_feature=categorical_features)
    lgb_eval = lgb.Dataset(X_val, y_val,
                           reference=lgb_train,
                           categorical_feature=categorical_features)

    model = lgb.train(params, lgb_train,
                      valid_sets=[lgb_train, lgb_eval],
                      verbose_eval=10,
                      num_boost_round=1000,
                      early_stopping_rounds=10)

    oof_train[valid_index] = model.predict(X_val, num_iteration=model.best_iteration)
    y_pred = model.predict(X_test, num_iteration=model.best_iteration)

    y_preds.append(y_pred)
    models.append(model)

scores = [
m.best_score['valid_1']['binary_logloss'] for m in models
]
score = sum(scores) / len(scores)
print('===CV scores===')
print(scores)
print(score)
```

## ラベルの分布を維持したまま分割する場合はStratifiedKFoldを用いる
```py
K = 2 # 2セット
skf = StratifiedKFold(n_splits=K, random_state=SEED, shuffle=True)

DISASTER = df_train['target'] == 1
print('Whole Training Set Shape = {}'.format(df_train.shape))
print('Whole Training Set Unique keyword Count = {}'.format(df_train['keyword'].nunique()))
print('Whole Training Set Target Rate (Disaster) {}/{} (Not Disaster)'.format(df_train[DISASTER]['target_relabeled'].count(), df_train[~DISASTER]['target_relabeled'].count()))

for fold, (trn_idx, val_idx) in enumerate(skf.split(df_train['text_cleaned'], df_train['target']), 1):
    print('\nFold {} Training Set Shape = {} - Validation Set Shape = {}'.format(fold, df_train.loc[trn_idx, 'text_cleaned'].shape, df_train.loc[val_idx, 'text_cleaned'].shape))
    print('Fold {} Training Set Unique keyword Count = {} - Validation Set Unique keyword Count = {}'.format(fold, df_train.loc[trn_idx, 'keyword'].nunique(), df_train.loc[val_idx, 'keyword'].nunique()))    

```

## 時系列に並べたい場合(データリーク対応)
```py
feature_cols = ['day', 'hour', 'minute', 'second', 
                'ip_labels', 'app_labels', 'device_labels',
                'os_labels', 'channel_labels']

valid_fraction = 0.1
# 時刻列でソート
df = df_org.sort_values('click_time')
valid_rows = int(len(df) * valid_fraction)
train = df[:-valid_rows * 2]
# valid size == test size, last two sections of the data
valid = df[-valid_rows * 2:-valid_rows]
test = df[-valid_rows:]
```