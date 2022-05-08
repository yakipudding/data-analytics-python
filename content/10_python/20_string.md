---
title: '文字列操作'
---

## 文字数
```py
len('aaa')
# 3
```

## 検索
存在するかどうかはin

```py
'aa' in 'aa_bb_cc'
```

indexはfind

```py
'aa_bb_cc'.find('bb')
# 3
```

## フォーマット
### f文字列
```py
name = 'Taro'
f'Hello, {name}!'
# 'Hello, Taro!'
```

### 書式指定
[書式指定ミニ言語](https://docs.python.org/ja/3/library/string.html#format-specification-mini-language)を使用できる

```py
# . 小数桁数1 f(float)
f'{1.234:.1f}'
# '1.2'

# カンマ
f'{12345:,}'
# '12,345'

# 左詰め5桁
f'{"aa":<5}'
# 'aa   '
```

## 特殊文字の処理
- Windowsのパスなどで`\`を扱いたいとき、そのままだと特殊構文になるため`\\`にエスケープする必要がある
- 文字列をそのまま扱いたいときは`r`で生文字列として扱う

```py
# c:\\...のパスを指定したい場合
'c:\\\\...'

# 生文字
r'c:\\...'
# c:\\\\...
```

## replace
`文字列`の`対象文字列`を`置換文字列`に置換する

```py
'aa_bb_cc'.replace('a', 'z')
# zz_bb_cc
```

## split
`文字列`を`区切り文字`で分割する

```py
'aa_bb_cc'.split('_')
# [aa, bb, cc]
```
