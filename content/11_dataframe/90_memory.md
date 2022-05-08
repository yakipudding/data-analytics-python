---
title: 'メモリ削減'
---
読み込んだままだとめっちゃメモリ使うのでメモリ削減のtips

```py
# DFの数値型の範囲から適切な型に設定しなおす
import pandas as pd
import numpy as np

def set_numtype(df, verbose=True):
    numerics = ['int16', 'int32', 'int64', 'float16', 'float32', 'float64']
    start_mem = df.memory_usage().sum() / 1024**2    
    for col in df.columns:
        col_type = df[col].dtypes
        if col_type in numerics:
            c_min = df[col].min()
            c_max = df[col].max()
            if str(col_type)[:3] == 'int':
                if c_min &gt; np.iinfo(np.int8).min and c_max &lt; np.iinfo(np.int8).max:
                    df[col] = df[col].astype(np.int8)
                elif c_min &gt; np.iinfo(np.int16).min and c_max &lt; np.iinfo(np.int16).max:
                    df[col] = df[col].astype(np.int16)
                elif c_min &gt; np.iinfo(np.int32).min and c_max &lt; np.iinfo(np.int32).max:
                    df[col] = df[col].astype(np.int32)
                elif c_min &gt; np.iinfo(np.int64).min and c_max &lt; np.iinfo(np.int64).max:
                    df[col] = df[col].astype(np.int64)  
            else:
                if c_min &gt; np.finfo(np.float16).min and c_max &lt; np.finfo(np.float16).max:
                    df[col] = df[col].astype(np.float16)
                elif c_min &gt; np.finfo(np.float32).min and c_max &lt; np.finfo(np.float32).max:
                    df[col] = df[col].astype(np.float32)
                else:
                    df[col] = df[col].astype(np.float64)
    end_mem = df.memory_usage().sum() / 1024**2
    if verbose: print('Mem. usage decreased to {:5.2f} Mb ({:.1f}% reduction)'.format(end_mem, 100 * (start_mem - end_mem) / start_mem))
    return df
```

CSVを読み込む際に使うと良い
```py
df = reduce_mem_usage(pd.read_csv('train.csv'))
```