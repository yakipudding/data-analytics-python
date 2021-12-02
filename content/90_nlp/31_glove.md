---
title: 'GloVe'
---

参考：https://www.kaggle.com/shahules/basic-eda-cleaning-and-glove（ほぼ丸コピ）

GloVe：Word2Vecのあとに出てきた、単語のベクトル変換モデル
- カウントベース手法と周辺語から予測する手法のいいとこどりで効率よく学習できる

GloVeの事前学習済みモデルを使用
 - そのままでは重すぎるため、出現した単語のみ辞書を抽出して作成
 - RNN(LSTM)で推論

```py
from tqdm import tqdm # 進捗ログ表示用
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords
stop=set(stopwords.words('english'))
from keras.preprocessing.text import Tokenizer
from keras.preprocessing.sequence import pad_sequences
from keras.models import Sequential
from keras.layers import Embedding,LSTM,Dense,SpatialDropout1D

df=pd.concat([tweet,test])

def create_corpus(df):
    corpus=[]
    for tweet in tqdm(df['text']):
        words=[word.lower() for word in word_tokenize(tweet) if((word.isalpha()==1) & (word not in stop))]
        corpus.append(words)
    return corpus

corpus=create_corpus(df)

embedding_dict={}
# 学習済みコーパスモデル(dim100)
with open('../input/glove-global-vectors-for-word-representation/glove.6B.100d.txt','r') as f:
    for line in f:
        values=line.split()
        word=values[0]
        vectors=np.asarray(values[1:],'float32')
        embedding_dict[word]=vectors
f.close()

MAX_LEN=50
tokenizer_obj=Tokenizer()
tokenizer_obj.fit_on_texts(corpus)
# コーパス(単語配列)から単語のシーケンス番号(単語index)配列に変換
sequences=tokenizer_obj.texts_to_sequences(corpus)
# 各文の長さが異なり扱いにくいため、固定長に変換
tweet_pad=pad_sequences(sequences,maxlen=MAX_LEN,truncating='post',padding='post')

word_index=tokenizer_obj.word_index
print('Number of unique words:',len(word_index))
# Number of unique words: 20342

num_words=len(word_index)+1
# モデルを0初期化
embedding_matrix=np.zeros((num_words,100))

# 出現した単語のコーパスモデルを作成（embedding_matrix）
for word,i in tqdm(word_index.items()):
    if i > num_words:
        continue
    
    emb_vec=embedding_dict.get(word)
    if emb_vec is not None:
        embedding_matrix[i]=emb_vec

# 学習
model=Sequential()

# 1.Embedding：入力層：文章のベクトル変換（辞書を利用）
embedding=Embedding(num_words,100,embeddings_initializer=Constant(embedding_matrix),
                   input_length=MAX_LEN,trainable=False)

model.add(embedding)
# 2.Dropout
model.add(SpatialDropout1D(0.2))
# 3.RNN(LSTM)
model.add(LSTM(64, dropout=0.2, recurrent_dropout=0.2))
# 4.Dense
model.add(Dense(1, activation='sigmoid'))
# 最適化：Adam
optimzer=Adam(learning_rate=1e-5)

model.compile(loss='binary_crossentropy',optimizer=optimzer,metrics=['accuracy'])

model.summary()
# Model: "sequential_1"
# _________________________________________________________________
# Layer (type)                 Output Shape              Param #   
# =================================================================
# embedding_1 (Embedding)      (None, 50, 100)           2034300   
# _________________________________________________________________
# spatial_dropout1d_1 (Spatial (None, 50, 100)           0         
# _________________________________________________________________
# lstm_1 (LSTM)                (None, 64)                42240     
# _________________________________________________________________
# dense_1 (Dense)              (None, 1)                 65        
# =================================================================
# Total params: 2,076,605
# Trainable params: 42,305
# Non-trainable params: 2,034,300
# _________________________________________________________________


train=tweet_pad[:tweet.shape[0]]
test=tweet_pad[tweet.shape[0]:]

X_train,X_test,y_train,y_test=train_test_split(train,tweet['target'].values,test_size=0.15)
print('Shape of train',X_train.shape)
print("Shape of Validation ",X_test.shape)
# Shape of train (6471, 50)
# Shape of Validation  (1142, 50)

history=model.fit(X_train,y_train,batch_size=4,epochs=15,validation_data=(X_test,y_test),verbose=2)

# predict
sample_sub=pd.read_csv('../input/nlp-getting-started/sample_submission.csv')

y_pre=model.predict(test)
y_pre=np.round(y_pre).astype(int).reshape(3263)
sub=pd.DataFrame({'id':sample_sub['id'].values.tolist(),'target':y_pre})
sub.to_csv('submission.csv',index=False)

sub.head()
```