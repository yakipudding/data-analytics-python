---
title: 'N-Gram'
---

```py
import pandas as pd
import json
from sklearn.feature_extraction.text import CountVectorizer

# 最初の 10,000 件のレビューを読み込む
with open('data/yelp/yelp_academic_dataset_review.json') as f:
    js = []
    for i in range(10000):
        js.append(json.loads(f.readline()))
review_df = pd.DataFrame(js)

# scikit-learn の CountVectorizer を使ってユニグラム（BoW）、
# バイグラム、トライグラムの特徴量変換器を作成する。
# CountVectorizer はデフォルトでは1文字の単語を無視するが、
# これは意味のない単語を除外するため実用的である。
# ただしここでは全ての単語を含むように設定している。
bow_converter = CountVectorizer(token_pattern='(?u)\\b\\w+\\b')
bigram_converter = CountVectorizer(ngram_range=(2,2), token_pattern='(?u)\\b\\w+\\b')
trigram_converter = CountVectorizer(ngram_range=(3,3), token_pattern='(?u)\\b\\w+\\b')

# 変換器を適用し、語彙数を確認する
bow_converter.fit(review_df['text'])
words = bow_converter.get_feature_names()

bigram_converter.fit(review_df['text'])
bigrams = bigram_converter.get_feature_names()

trigram_converter.fit(review_df['text'])
trigrams = trigram_converter.get_feature_names()

print (len(words), len(bigrams), len(trigrams))

# n-グラムを確認する
words[:10]