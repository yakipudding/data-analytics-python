---
title: 'BoW'
---

Bag of Words with SpaCy

```py
import spacy

spam = pd.read_csv('../input/nlp-course/spam.csv')
spam.head(10)
# spamは label/text のdf

# 空のSpaCyモデル作成
nlp = spacy.blank("en")
## jaも一応ある。nlp = spacy.load("ja_core_news_sm")など

# Bowのテキスト分類器を定義
## 未定義のラベル(ham/spam以外)は出現しない場合はexclusive_classesをTrueに設定
textcat = nlp.create_pipe(
              "textcat", #text categolyze
              config={
                "exclusive_classes": True,
                "architecture": "bow"})

# テキスト分類器をモデルに追加
nlp.add_pipe(textcat)

# 分類するラベルを追加
textcat.add_label("ham")
textcat.add_label("spam")
```

```py
# 教師データ作成
def load_data(csv_file, split=0.9):
    data = pd.read_csv(csv_file)
    
    # Shuffle data
    train_data = data.sample(frac=1, random_state=7)
    
    texts = train_data.text.values
    labels = [{"ham": bool(y), "spam": not bool(y)}
              for y in train_data.sentiment.values]
    split = int(len(train_data) * split)
    
    # ラベルは{'cats': {'label1': True, 'label2': False, ...} }の形にする
    train_labels = [{"cats": labels} for labels in labels[:split]]
    val_labels = [{"cats": labels} for labels in labels[split:]]
    
    return texts[:split], train_labels, texts[split:], val_labels

train_texts, train_labels, val_texts, val_labels = load_data('../input/nlp-course/yelp_ratings.csv')


```

# 学習（epoch1）
```py
from spacy.util import minibatch

spacy.util.fix_random_seed(1)
optimizer = nlp.begin_training()

batches = minibatch(train_data, size=8)
for batch in batches:
    # updateにはtextsとlabelsを分ける必要がある
    texts, labels = zip(*batch)
    nlp.update(texts, labels, sgd=optimizer)
```

# 学習（epoch10）
```py
from spacy.util import minibatch
import random

def train(model, train_data, optimizer, batch_size=8):
    losses = {}
    random.seed(1)
    random.shuffle(train_data)
    
    batches = minibatch(train_data, size=batch_size)
    for batch in batches:
        # train_data is a list of tuples [(text0, label0), (text1, label1), ...]
        # Split batch into texts and labels
        texts, labels = zip(*batch)
        
        # Update model with texts and labels
        model.update(texts, labels, sgd=optimizer, losses=losses)
        
    return losses

# Fix seed for reproducibility
spacy.util.fix_random_seed(1)
random.seed(1)

# This may take a while to run!
optimizer = nlp.begin_training()
train_data = list(zip(train_texts, train_labels))
losses = train(nlp, train_data, optimizer)
print(losses['textcat'])
```

# 推論
```py
texts = ["Are you ready for the tea party????? It's gonna be wild",
         "URGENT Reply to this message for GUARANTEED FREE TEA" ]
docs = [nlp.tokenizer(text) for text in texts]
    
# Use textcat to get the scores for each doc
textcat = nlp.get_pipe('textcat')
scores, _ = textcat.predict(docs)

print(scores)

# From the scores, find the label with the highest score/probability
predicted_labels = scores.argmax(axis=1)
print([textcat.labels[label] for label in predicted_labels])
```