## Step1

- splitメソッドを知らなかった
- $が終わりマークなので、「〇〇以降のものを取り除く」という処理ができる
- 変数名が、中身が変化していってもずっとlocal_nameと同じ変数名だが、これでいいのかは正直迷った。こういうのは専門家にとって忌避感があるのか？（が、個々に対して適切な変数名が思いつかない）
- setの初期化方法を忘れていた

```python

import re

class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        email_set = set()
        for email in emails:
            local_name, domain_name = email.split('@', 1)
            local_name = re.sub('\.', '', local_name)
            local_name = re.sub('\+.*$', '', local_name)
            email_set.add(str.join(local_name, domain_name))
        return len(email_set)
```

## Step2

- [ayashiさんのやつ](https://github.com/hayashi-ay/leetcode/blob/hayashi-ay-patch-14/929.%20Unique%20Email%20Addresses.md)が色々参考になった
`_ignore_alias` とか`_without_dots` とかなるほど
    - 文字列の処理と、setに加えるので関数を分けるのもいいと思った。
        - canonicalizeという表現が参考になった、正規化
    - 入力値に@マークが必ず1つだけ入ることに強く依存している。
        - 確かに
        - エラーの処理もうまく実装
    - split以外にも、replace関数を知らなかった。
        - reに囚われていたが、を使わなくてもこれぐらいのことはできるとわかった。
    - [以下のリファレンス](https://docs.python.org/3/library/stdtypes.html#string-methods)をチラ見。文字列のメソッドを確認。思ったより多かった。全部覚えなくてもちまちま確認していくのが良さそう？
    - 正規表現にgroupがあることがわかった。うまく分割するのにこの方が楽かも。
    - re.matchはドキュメント見たら、matchしない場合Noneを返すのでif not matchができる。
    - f文字列は確かにわかりやすいかも
- ２種類の方法で実装　まずはreを使わないバージョン

```python

class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def canonicalize(email):
            if len(email.split('@')) != 2:
                raise Exception('Invalid Inputs')
            local_name = email.split('@')[0]
            domain_name = email.split('@')[1]
            canonicalized_local = local_name.split('+')[0].replace('.', '')
            return f'{canonicalized_local}@{domain_name}'
        
        email_set = set()
        for email in emails:
            print(canonicalize(email))
            email_set.add(canonicalize(email))
        return len(email_set)
```

- reを使うバージョン

```python

import re

class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def canonicalize(email):
            match = re.match(r'([\w\.]*)\+?.*@([\w\.\+]*)', email)
            if not match:
                raise Exception('Invalid Inputs')
            canonicalized_local = match.group(1).replace('.', '')
            domain = match.group(2)
            return f'{canonicalized_local}@{domain}'
        email_set = set()
        for email in emails:
            email_set.add(canonicalize(email))
        return len(email_set)
```

## Step3

- canonicalized_localにするときの、[0]のつけ忘れ

```python

class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def canonicalize(email):
            if len(email.split('@')) != 2:
                raise Exception('invalid input')
            local_name, domain_name = email.split('@')
            canonicalized_local = local_name.split('+')[0].replace('.', '')
            return f'{canonicalized_local}@{domain_name}'
        
        email_set = set()
        for email in emails:
            email_set.add(canonicalize(email))
        return len(email_set)
```

- 正規表現バージョン

```python

import re
class Solution:
    def numUniqueEmails(self, emails: List[str]) -> int:
        def canonicalize(email):
            match = re.match(r'^([\.\w]*)\+?[\.\w\+]*@([\.\w\+]*)$', email)
            if not match:
                raise Exception('invalid input')
            canonicalized_local = re.sub(r'\.', '',match.group(1))
            domain = match.group(2)
            return f'{canonicalized_local}@{domain}'
        email_set = set()
        for email in emails:
            email_set.add(canonicalize(email))
        return len(email_set)
```

参考

- [ayashiさんのやつ](https://github.com/hayashi-ay/leetcode/blob/hayashi-ay-patch-14/929.%20Unique%20Email%20Addresses.md)
- [re.match](https://docs.python.org/ja/3/library/re.html#:~:text=re.match(pattern%2C%20string%2C%20flags%3D0)%C2%B6)
- [文字列メソッドのリファレンス](https://docs.python.org/3/library/stdtypes.html#string-methods)（チラ見）
