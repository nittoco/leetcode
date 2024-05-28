## Step1

- そんなに時間かからず
- sorted(word)がmutableなのでhashできず、どうするか迷ったがtupleにした

```python

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        grouped_words = []
        word_to_list_index = {}
        for word in strs:
            sorted_word = tuple(sorted(word))
            if sorted_word in word_to_list_index:
                current_index = word_to_list_index[sorted_word]
                grouped_words[current_index].append(word)
                continue
            grouped_words.append([word])
            word_to_list_index[sorted_word] = len(grouped_words) - 1
        return grouped_words
```

## Step2

- defaultdict、聞いたことはあるがよく知らない
- https://github.com/hayashi-ay/leetcode/pull/19/files
    - ランレングス圧縮の解法を知る
    - values() を知らなかった
        - items()は知ってたけど、どっちもview objectというのを返しているらしい
        - ドキュメント曰く、setの演算を受け付けるけどiterableで、keyとvaluesがどちらも同じ順番（挿入順）で格納されている
        - これをうまく使えば、grouped_wordsとword_to_list_index両方はいらない。特に後者があるのが今ややこしい
- [sorted()、はlistを返す]https://docs.python.org/3/library/functions.html#sorted))、listはmutableなのでhashのkeyにできない

https://github.com/YukiMichishita/LeetCode/pull/5/files

- Step1のようにtupleは分かりにくいかも。str(sorted())でもいいね

https://github.com/SuperHotDogCat/coding-interview/pull/13/files

https://github.com/sakupan102/arai60-practice/pull/13/files

- “”.joinとstr変換で迷う。
- defaultdict、__getitem__を読んだ時の__missing__をoverrideしているっぽい
    - 他の人の実装も見てみる
    - 参考　https://github.com/Mike0121/LeetCode/pull/3/files
    - これも見たhttps://github.com/sakupan102/arai60-practice/pull/10/files#diff-5cd4e95016aaca2d8bd5031dc91d4cc73ac3baad3b6c002619c9ceb3f0a9fd5fR82
    - dictを継承してmissingを実装するのでもいけるか？ちょっと両方試してみる
    - 自力でここまでやったがエラー。dictクラスそのものに対してはsubscriptableではないらしい（[ドキュメント](https://docs.python.org/3/reference/datamodel.html#object.__setitem__)で`object.__setitem__()`とかいてあったのでおそらく__setitem__()はclassじゃなくてobjectにbindしないといけないってこと)
    
    ```python
    
    class DefaultDict(dict):
      def __init__(self, default_factory = None):
        super().__init__()
        self.default_factory = default_factory
    
      def __missing__(self, key):
        if self.default_factory is None:
          raise KeyError
        dict[key] = self.default_factory
        
    ```
    
- この後はこのコードでエラー
    - ここで質問。お手数をおかけしました🙇コミュニケーションに気をつけます(https://discord.com/channels/1084280443945353267/1226508154833993788/1244125329560305774)
    - `__missing__` でreturnされていない、returnしないときはPythonはNoneを返す
    - default_factory()を__init__でやるのは、__missing__が複数回呼ばれた時に、同じlistを参照してしまうのでやばい　by odaさん
        - 複数回呼ばれた時のシミュレーション、というのをあまり考えたことなかったので勉強になった
    - super(）使った継承は初めての実装なのでベストプラクティスが載ってる[これ](https://rhettinger.wordpress.com/2011/05/26/super-considered-super/)　を読んだ。公式docからこれ見ましょうってリンクだから大丈夫と信じてるけど、古いからやや心配

```python
class DefaultDict(dict):
  def __init__(self, default_factory = None):
    super().__init__()
    self.default_factory = default_factory()

  def __missing__(self, key):
    if self.default_factory is None:
      raise KeyError
    self[key] = self.default_factory
```

- 以下のコードで最終的に実装
    - 関数部分の実装が楽になりdefaultdict便利

```python

class DafaultDict(dict):
    def __init__(self, default_factory = None):
        self.default_factory = default_factory

    def __missing__(self, key, value):
        if default_factory is None:
            raise KeyError
        value = default_factory
        self[key] = value
        return value

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        to_anaglam = DefaultDict(list)
        for word in strs:
            sorted_word = str(sorted(word))
            to_anaglam[sorted_word].append(word)
        return to_anaglam.values()
```

- この実装も試した

```python

class DafaultDict():
    def __init__(self, default_factory = None):
        self.default_factory = default_factory
        self._dict = dict()

    def __getitem__(self, key):
        if key not in _dict:
            value = default_factory()
		        self._dict[key] = value
            return value
        return self._dict[key]

    def __setitem__(self, key, value):
        self._dict[key] = value
```

- [これ](https://github.com/python/cpython/blob/v3.10.0/Modules/_collectionsmodule.c　)の1994行目からCPythonの実装、何となく雰囲気はわかった
- defaultdictがないときの実装は、この実装の方が統一的で整理されている。

https://discord.com/channels/1084280443945353267/1227102763222171648/1237899825903833111

- 思いつかなかった…
- list[index].append(word)と、list.append(word)という、見た目が似てるけどやってる処理が微妙に異なるものが出てくるのは見にくいなあと思っていたが、いい代案が思いついてなかった
- 新しく作るときに、「空の」リストを作ってappendするという発想がなかった
- そもそもdefaultdictの実装もそうなっている

## Step3

- ここでは練習のため、defaultdictをないものとしてやった
- str(sorted(word)) のsortedをつけ忘れ1敗

```python

class Solution:
    def groupAnagrams(self, strs: List[str]) -> List[List[str]]:
        to_anagram = dict()
        for word in strs:
            sorted_word = str(sorted(word))
            if sorted_word not in to_anagram:
                to_anagram[sorted_word] = []
            to_anagram[sorted_word].append(word)
        return to_anagram.values()
```
