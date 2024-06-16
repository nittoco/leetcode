## Step1

- enumerateを利用するなどの選択肢は浮かんでいる
    - そうするとでもsに対して2回for文を回す必要があると思いやめた
    - 2回目にfor文回すときに英小文字に対して回したほうが節約できるのではという考え
- char_to_countを、辞書ではなく単なる配列にするなどの選択肢も
    - ただアルファベット以外も入力にもし来るなら辞書のほうがいいと思いこれにした
- 本当は1パスにしたいが浮かばない
    - 1回目の出現で文字とindexをindex順に入れて、2回目に出現したらO(n)未満でその文字のを抜けさせる、イテレートが終わったら最初に入れた文字を取り出せる、ができるようなデータ構造があれば便利だが、までは思う

```python

from string import ascii_lowercase

class Solution:
    def firstUniqChar(self, s: str) -> int:
        char_count = defaultdict(int)
        char_to_index = {}
        for i in range(len(s)):
            char_count[s[i]] += 1
            char_to_index[s[i]] = i
        first_unique_index = inf
        for c in ascii_lowercase:
            if char_count[c] == 1:
                first_unique_index = min(first_unique_index, char_to_index[c])
        if first_unique_index == inf:
            return -1
        return first_unique_index

```

## Step2

- https://github.com/shining-ai/leetcode/pull/15
    - LinkedHashMapが、多分Step1の最後で言ったようなことができるデータ構造。後で、LinkedHashMapの実装をしてみる

https://github.com/TORUS0818/leetcode/pull/17/files

https://github.com/hayashi-ay/leetcode/pull/28/files

https://github.com/fhiyo/leetcode/pull/18/files

- 他にも1パスにする方法あった
    - dictにpop()があるのを知らなかった。これなら2回目出た時の削除に時間が掛からずにできるのか。
        - 削除するだけならdelでも良かったかも？
    - [これ](https://docs.python.org/ja/3/library/stdtypes.html#dict)も知らなかった
        
        > 辞書は挿入順序を保存するようになりました。 キーの更新は順序には影響が無いことに注意してください。 いったん削除されてから再度追加されたキーは末尾に挿入されます。:
        > 
    - [minの計算量](https://wiki.python.org/moin/TimeComplexity)　minを使うと結局もう1回文字列を探索する？
        - torusさんへのodaさんのコメントを見る限り、十分高速ではあるらしい
        - [コード](https://github.com/python/cpython/blob/main/Python/bltinmodule.c#L1796C1-L1796C8)　も流し読み　やはり全要素をなめてる
    - next iterを使えば2回ループは避けれる（無理矢理感）
    - secoundは微妙な気がしてきた。seenとかduplicatedの方が良かったかも。
    - dictが順序を保存するようになったので今ではあまり意味ないが、[OrderedDict](https://docs.python.org/ja/3/library/collections.html#collections.OrderedDict)というのもある。(LinkedHashMapとほぼ同じらしい）
    
    ```python
    # 最初のアイデア　下で直す
    class Solution:
        def firstUniqChar(self, s: str) -> int:
            unique_char_to_index = {}
            secound = set()
            for i, c in enumerate(s):
                if c in secound:
                    continue
                if c not in unique_char_to_index:
                    unique_char_to_index[c] = i
                    continue
                unique_char_to_index.pop(c)
                secound.add(c)
            if not unique_char_to_index:
                return -1
            return next(iter(unique_char_to_index.values()))
    ```
    
- もう1回書き直し

```python

class Solution:
    def firstUniqChar(self, s: str) -> int:
        unique_char_to_index = {}
        duplicated = set()
        for i, c in enumerate(s):
            if c in duplicated:
                continue
            if c not in unique_char_to_index:
                unique_char_to_index[c] = i
                continue
            del unique_char_to_index[c]
            duplicated.add(c)
        if not unique_char_to_index:
            return -1
        return next(iter(unique_char_to_index.values()))
```

- [Counter](https://docs.python.org/ja/3/library/collections.html#counter-objects)というのがあるらしい
    - iterableなものの要素数を数えて辞書にしてくれる。mappingを入れたら初期化してくれる。
    - 実はマイナスも取れる
- 2回for文を使うバージョンも書いた、簡潔に感じる

```python

from collections import defaultdict

class Solution:
    def firstUniqChar(self, s: str) -> int:
        char_to_freq = defaultdict(int)
        for c in s:
            char_to_freq[c] += 1
        for i, c in enumerate(s):
            if char_to_freq[c] == 1:
                return i
        return -1
```

## Step3

結局これが一番簡潔に感じた

```python

from collections import defaultdict

class Solution:
    def firstUniqChar(self, s: str) -> int:
        char_to_freq = defaultdict(int)
        for c in s:
            char_to_freq[c] += 1
        for i, c in enumerate(s):
            if char_to_freq[c] == 1:
                return i
        return -1
```
