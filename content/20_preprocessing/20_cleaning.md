---
title: '文字列のクリーニング'
---

Webでスクレイピングした文章データなどのクリーニング  
参考: https://www.kaggle.com/shahules/basic-eda-cleaning-and-glove

## Removing URL
```py
example="New competition launched :https://www.kaggle.com/c/nlp-getting-started"
def remove_URL(text):
    url = re.compile(r'https?://\S+|www\.\S+')
    return url.sub(r'',text)
remove_URL(example)
# 'New competition launched :'
df['text']=df['text'].apply(lambda x : remove_URL(x))
```

## Removing HTML tags
```py
example = """<div>
<h1>Real or Fake</h1>
<p>Kaggle </p>
<a href="https://www.kaggle.com/c/nlp-getting-started">getting started</a>
</div>"""
def remove_html(text):
    html=re.compile(r'<.*?>')
    return html.sub(r'',text)
print(remove_html(example))
# Real or Fake
# Kaggle 
# getting started
df['text']=df['text'].apply(lambda x : remove_html(x))
```

## Romoving Emojis
```py
# Reference : https://gist.github.com/slowkow/7a7f61f495e3dbb7e3d767f97bd7304b
def remove_emoji(text):
    emoji_pattern = re.compile("["
                           u"\U0001F600-\U0001F64F"  # emoticons
                           u"\U0001F300-\U0001F5FF"  # symbols & pictographs
                           u"\U0001F680-\U0001F6FF"  # transport & map symbols
                           u"\U0001F1E0-\U0001F1FF"  # flags (iOS)
                           u"\U00002702-\U000027B0"
                           u"\U000024C2-\U0001F251"
                           "]+", flags=re.UNICODE)
    return emoji_pattern.sub(r'', text)

remove_emoji("Omg another Earthquake 😔😔")
# 'Omg another Earthquake '
df['text']=df['text'].apply(lambda x: remove_emoji(x))
```

## Removing punctuations
```py
def remove_punct(text):
    table=str.maketrans('','',string.punctuation)
    return text.translate(table)

example="I am a #king"
print(remove_punct(example))
# I am a king
df['text']=df['text'].apply(lambda x : remove_punct(x))
```

## Spelling Correction(English)
```
!pip install pyspellchecker
```

```py
from spellchecker import SpellChecker

spell = SpellChecker()
def correct_spellings(text):
    corrected_text = []
    misspelled_words = spell.unknown(text.split())
    for word in text.split():
        if word in misspelled_words:
            corrected_text.append(spell.correction(word))
        else:
            corrected_text.append(word)
    return " ".join(corrected_text)
        
text = "corect me plese"
correct_spellings(text)
#'correct me please'
df['text']=df['text'].apply(lambda x : correct_spellings(x))
```