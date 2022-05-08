---
title: '形態素解析'
---

- 日本語の文章データを形態素（名詞、動詞など品詞）で解析する手法
    - 英語は単語が半角スペースで区切られているので形態素解析はいらないが、日本語は区切り文字がないため必要
- 形態素解析することを「分かち書き」という
    - 「今日は東京タワーで遊んだ」→「[今日] [は] [東京タワー] [で] [遊んだ]」みたいに、文章を品詞単位で分割できる

## MeCab
オープンソースの形態素解析エンジン

- [公式サイト](https://taku910.github.io/mecab/#download)
- [mecab-ipadic-NEologd](https://github.com/neologd/mecab-ipadic-neologd/blob/master/README.ja.md)
    - MeCab用の辞書

```py
import MeCab

mecab = MeCab.Tagger()

text = '今日は東京タワーで遊んだ'
mecab.parse('')#文字列がGCされるのを防ぐ
node = mecab.parseToNode(text)

while node:
    #単語を取得
    word = node.surface
    
    # 辞書の特徴（品詞分類などが表示される）
    print(node.feature)

    node = node.next
```