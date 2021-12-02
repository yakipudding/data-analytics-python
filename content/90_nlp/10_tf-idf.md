---
title: 'TF-IDF'
---

```py
# Pythonで、Yelpのレビューデータを読み込み、クリーニングを行う
import json
import pandas as pd

# Yelpのビジネスデータを読み込み
with open('data/yelp/yelp_academic_dataset_business.json') as biz_f:
    biz_df = pd.DataFrame([json.loads(x) for x in biz_f.readlines()])

# Yelpのレビューデータを読み込み
with open('data/yelp/yelp_academic_dataset_review.json') as review_file:
    review_df = pd.DataFrame([json.loads(x) for x in review_file.readlines()])

# YelpのビジネスデータからcategoriesがNightlife（ナイトライフ@<fn>{nightlife}）またはRestaurants(レストラン)のデータを取り出し
filter_func = lambda x:  len(set(x) &  set(['Nightlife', 'Restaurants'])) > 0
twobiz = biz_df[biz_df['categories'].apply(filter_func)]

# 取り出した2つのカテゴリのYelpのビジネスデータとYelpのレビューデータを結合する
twobiz_reviews = twobiz.merge(review_df, on='business_id', how='inner')

# 必要ない特徴量を排除
twobiz_reviews = twobiz_reviews[['business_id', 'name', 'stars_y', 'text', 'categories']]

# target列を作成。categoriesがNightlifeの時はTrue、それ以外の場合はFalse
twobiz_reviews['target'] =  twobiz_reviews['categories'].apply(set(['Nightlife']).issubset)

twobiz

# クラス均衡の取れた分類用データセットの作成 ###################################
## レストランとナイトライフでデータ数が異なるため、少ないほうに合わせる

from sklearn.model_selection import train_test_split

# サンプリングしてクラス均衡を是正したデータセットを作成
nightlife = twobiz_reviews[twobiz_reviews['categories'].apply(set(['Nightlife']).issubset)]
restaurants = twobiz_reviews[twobiz_reviews['categories'].apply(set(['Restaurants']).issubset)]

nightlife_subset = nightlife.sample(frac=0.1, random_state=123)
restaurant_subset = restaurants.sample(frac=0.021, random_state=123)
combined = pd.concat([nightlife_subset, restaurant_subset])

# 学習データとテストデータに分割
training_data, test_data = train_test_split(combined, test_size=0.3, random_state=123)
training_data.shape ## 行数、列数

# 特徴量変換 ###################################

from sklearn.feature_extraction import text

# レビューをBag-of-Wordsで表す
## CountVectorizer:テキストをBoWに変換
bow_transform = text.CountVectorizer()
X_tr_bow = bow_transform.fit_transform(training_data['text'])
X_te_bow = bow_transform.transform(test_data['text'])
len(bow_transform.vocabulary_)

from sklearn.preprocessing import normalize

y_tr = training_data['target']
y_te = test_data['target']

# Bag-of-Words行列からTF-IDFを作成
tfidf_trfm = text.TfidfTransformer(norm=None)
X_tr_tfidf = tfidf_trfm.fit_transform(X_tr_bow)
X_te_tfidf = tfidf_trfm.transform(X_te_bow)

#  Bag-of-WordsのL2正規化
X_tr_l2 = normalize(X_tr_bow, norm='l2', axis=0)
X_te_l2 = normalize(X_te_bow, norm='l2', axis=0)

# デフォルトのパラメータでロジスティック回帰を学習 #############################

from sklearn.linear_model import LogisticRegression

def simple_logistic_classify(X_tr, y_tr, X_test, y_test, description, _C=1.0):
### ロジスティック回帰で学習し、テストデータでの予測結果を得る関数
    m = LogisticRegression(solver='liblinear', C=_C).fit(X_tr, y_tr)
    s = m.score(X_test, y_test)
    print ('Test score with', description, 'features:', s)
    return m

m1 = simple_logistic_classify(X_tr_bow, y_tr, X_te_bow, y_te, 'bow')
m2 = simple_logistic_classify(X_tr_l2, y_tr, X_te_l2, y_te, 'l2-normalized')
m3 = simple_logistic_classify(X_tr_tfidf, y_tr, X_te_tfidf, y_te, 'tf-idf')

# ロジスティック回帰のハイパーパラメータをグリッドサーチでチューニング ############

from sklearn.model_selection import GridSearchCV

# 探索範囲を指定して、5分割でグリッドサーチを実行します
param_grid_ = {'C': [1e-5, 1e-3, 1e-1, 1e0, 1e1, 1e2]}

# Bag-of-Wordsでの分類器をチューニング
## GriidSearchCV：クロスバリデーションしながらグリッドサーチ
bow_search = GridSearchCV(LogisticRegression(solver='liblinear'), cv=5, param_grid=param_grid_, return_train_score=True)
bow_search.fit(X_tr_bow, y_tr)

# L2正規化単語ベクトルでの分類器をチューニング
l2_search = GridSearchCV(LogisticRegression(solver='liblinear'), cv=5, param_grid=param_grid_, return_train_score=True)
l2_search.fit(X_tr_l2, y_tr)

# TF-IDFでの分類器をチューニング
tfidf_search = GridSearchCV(LogisticRegression(solver='liblinear'), cv=5, param_grid=param_grid_, return_train_score=True)
tfidf_search.fit(X_tr_tfidf, y_tr)

# グリッドサーチにおける出力を見て、挙動を確認します
bow_search.cv_results_

# クロスバリデーションの結果を箱ひげ図でプロットする
# 分類器のパフォーマンスを可視化して比較する
search_results = pd.DataFrame.from_dict({
    'bow': bow_search.cv_results_['mean_test_score'],
    'tfidf': tfidf_search.cv_results_['mean_test_score'],
    'l2': l2_search.cv_results_['mean_test_score']
})

# matplotlibでグラフを描く。ここでSeabornはグラフの見た目を整える為に用いている。
# %matplotlib inline
import matplotlib.pyplot as plt
import seaborn as sns
sns.set_style("whitegrid")
ax = sns.boxplot(data=search_results, width=0.4)
ax.set_ylabel('Accuracy', size=14)
ax.tick_params(labelsize=14)

# 異なる特徴量で比較するための最終的な学習と検証 ############################

# クロスバリデーションで得られた最適なハイパーパラメータと学習用データ全てを用いて最終的なモデルを学習し、そのモデルを用いて検証用データにおける精度を算出する
m1 = simple_logistic_classify(X_tr_bow, y_tr, X_te_bow, y_te, 'bow',  _C=bow_search.best_params_['C'])
m2 = simple_logistic_classify(X_tr_l2, y_tr, X_te_l2, y_te, 'l2-normalized',  _C=l2_search.best_params_['C'])
m3 = simple_logistic_classify(X_tr_tfidf, y_tr, X_te_tfidf, y_te, 'tf-idf',  _C=tfidf_search.best_params_['C'])
```