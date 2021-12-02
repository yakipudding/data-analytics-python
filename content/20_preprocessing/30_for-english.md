---
title: '英語向けの文字列処理'
---

## corpus作成
```py
def create_corpus(target):
    corpus=[]
    
    # textを空白文字で分割してcorpusに挿入
    for x in tweet[tweet['target']==target]['text'].str.split():
        for i in x:
            corpus.append(i)
    return corpus
```

## stopword（英語）：the as など
```py
from collections import defaultdict
from nltk.corpus import stopwords
stop=set(stopwords.words('english'))

corpus=create_corpus(0)
dic=defaultdict(int)
for word in corpus:
    if word in stop:
        dic[word]+=1

# TOP10を描画
top=sorted(dic.items(), key=lambda x:x[1],reverse=True)[:10] 
#  top:[('the', 1524),('a', 1115), ...] タプルの配列
x,y=zip(*top) # zip:複数リストの要素を取得
# *top:('the', 1524) ('a', 1115)... ※要素がアンパックされる
# x:('the','a',...) y:(1524,1115,...)
plt.bar(x,y)
```

## punctuation（英語）：句読点、記号
```py
import string
special = string.punctuation

corpus=create_corpus(1)
dic=defaultdict(int)
for i in (corpus):
    if i in special:
        dic[i]+=1
```

## common word（英語）
```py
from collections import  Counter
counter=Counter(corpus) # copusの単語をカウントする
# Counter({"What's": 12,'up': 167, ... })

most=counter.most_common()
# [('the', 1524), ('a', 1115),... ]
x=[]
y=[]
for word,count in most[:40]:
    if (word not in stop) :
        x.append(word)
        y.append(count)
```