## Step1

- 空文字列の処理は別だった。right=1から始めてるので。
- スペースの処理は迷ったが、普通の文字列と同じように扱われるらしい。
- len()の書き方を忘れていた。
- while True:をleftに関するfor文にしようとしていた。left = left+1の処理をしているのでいらない。
- if(right ≥ length)　ではない。最後にright = right+1をするので。

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        if(s==''):
            return 0
        left = 0
        right = 1
        longest = 0
        length = len(s)
        while True:
            current_length = right - left
            current_kind = len(set(s[left:right]))
            if(current_length != current_kind):
                left = left + 1
            else:
                longest = max(right - left, longest)
                right = right + 1
            if(right > length):
                return(longest)

```

## Step2

- なんか嫌な予感がしたのでたまたま見たら、配列のスライスはO(k)であることを知る。https://wiki.python.org/moin/TimeComplexity
- 他の方の実装を見る限り、素直にsetの中で同じ文字があるかの判定が良さそう。
- 上のTime Complexityのページ曰く、setのx in sはなぜかAverageではO(1)らしい　WorstではO(n)
- lenもO(1)らしい
- left, right = 0, 0の方がまとまりあるかも
    - 別々に定義するより良いし、空文字列の場合に場合分けしなくて良い
- whileの代わりにfor文を使うことも考えたが、同じ文字がある場合はleft、ない場合はrightを進める、という論理の方がわかりやすいと思った
- while True: if(条件) returnよりも、while 条件 return の方がいいかも(ahayashiさんの実装)
    - https://github.com/hayashi-ay/leetcode/pull/47/files
- 登場文字の位置を管理するコードも見た、比較のために管理するものが増えるのでやめたhttps://github.com/usatie/leetcode/blob/main/Blind75/02. Longest Substring Without Repeating Characters/ans_01_20240125.cpp
- ついでにASCII などについて知る　https://discordapp.com/channels/1084280443945353267/1198621745565937764/1211683153668866139

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        left, right = 0, 0
        longest = 0
        length = len(s)
        current_set = set()
        while right<length:
        　　if s[right] in current_set:
                current_set.remove(s[left])
                left = left + 1
            else:
                current_set.add(s[right])
                longest = max(longest, right - left + 1)
                right = right + 1
        return longest

```

## Step3

- left, right = 0としてしまった
  
  ```python
  class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        left, right = 0, 0
        length = len(s)
        longest = 0
        unique_char_set = set()
        while right < length:
            if s[right] in unique_char_set:
                unique_char_set.remove(s[left])
                left = left + 1
            else:
                unique_char_set.add(s[right])
                longest = max(longest, right-left+1)
                right = right + 1
        return longest
  ```

## 参考にした資料
[Pythonの計算量のドキュメント](<https://wiki.python.org/moin/TimeComplexity>)
[ASCIIについて](<https://discordapp.com/channels/1084280443945353267/1198621745565937764/1211683153668866139>)
[ahayashiさんの実装](<https://github.com/hayashi-ay/leetcode/pull/47/files>)
[usatieさんの実装](<https://github.com/usatie/leetcode/blob/main/Blind75/02.%20Longest%20Substring%20Without%20Repeating%20Characters/ans_01_20240125.cpp>)
[hashについて](https://discordapp.com/channels/1084280443945353267/1199984201521430588/1200295973050650664)
