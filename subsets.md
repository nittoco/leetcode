## Step1

- bitで確認する方法。過去に同じロジックを競プロで実装したことがあった
- if i & 1 == 0: continueなどの選択肢は浮かんでいる

```python

class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        subset_index_bit = [i for i in range(1 << len(nums))] # 優先順序
        subsets = []
        for i in subset_index_bit:
            subset = []
            nums_index = 0
            while i:
                if i & 1:
                    subset.append(nums[nums_index])
                nums_index += 1
                i >>= 1
            subsets.append(subset)
        return subsets
```

https://github.com/hayashi-ay/leetcode/pull/63

- 色々選択肢がありますね。DFSぽくやる方法、extendを使う方法
- 結果を入れる変数名、all_subsetsはいいね
- DFSでやる方法、最初上のように書いて間違えた、直すのに苦労
    - 再帰を書くときは、どのようなことを行う(入力、出力、変更するデータ)関数かをしっかり頭で設計しないとだめ
    - 下のコードでもまだ冗長な気がするので、さらに直す

```python
# 間違いコード
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        def part_of_subsets(nums, index):
            if index == len(nums):
                return [[]]
            all_subsets = part_of_subsets(nums, index + 1)
            for subset in all_subsets: # all_subsetsが増えるので無限追加される
                all_subsets.append([nums[index]] + subset)
            return all_subsets
        
        return part_of_subsets(nums, 0)
```

```python
# 直したコード
class Solution:
    
    def subsets(self, nums: List[int]) -> List[List[int]]:
        def include_and_make_subset(include, index):
            if index == len(nums):
                if include is None:
                    return [[]]
                return [[include]]
            not_current_inclusive = include_and_make_subset(None, index+1)
            current_inclusive = include_and_make_subset(nums[index], index+1)
            all_subsets = not_current_inclusive + current_inclusive
            if include is not None:
                for subset in all_subsets:
                    subset.append(include)
            return all_subsets
        
        return include_and_make_subset(None, 0)
```

- 上のコード、さらにこの方が自然かつ簡潔だと思い直す

```python
# さらに直したコード
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        def part_of_subset(index):
            if index == len(nums):
                return [[]]
            not_inclusive = part_of_subset(index + 1)
            inclusive = part_of_subset(index + 1)
            for subset in inclusive:
                subset.append(nums[index])
            return not_inclusive + inclusive
        
        return part_of_subset(0)
```

- [ahayashiさんの再帰のやつ](https://github.com/hayashi-ay/leetcode/pull/63/files)(Step4のbacktracking)は、かなり読むのに時間かかった。
    - [Discord](https://discord.com/channels/1084280443945353267/1226508154833993788/1250316040257404929)で理解できました。ありがとうございます。
    - [実装参考](https://drken1215.hatenablog.com/entry/2020/05/04/190252)
    - 練習してみる、慣れてないので書きにくく感じた
    - copy()を最初忘れてた(subset自体をappendすると、subsetがのちに変化すると同じオブジェクトを挿してるのでappendしたものも変化してやばい)
    
    ```python
    
    class Solution:
        def subsets(self, nums: List[int]) -> List[List[int]]:
            all_subsets = []
            subset = []
            def subsets_after(index):
                if index == len(nums):
                    all_subsets.append(subset.copy())
                    return
                subset.append(nums[index])
                subsets_after(index + 1)
                subset.pop()
                subsets_after(index + 1)
            subsets_after(0)
            return all_subsets
    ```
    
- Step１のbit検査をする場合、bitmaskという変数名の方が良いかも
- extendはよく知らないので[調べる](https://docs.python.org/3/tutorial/datastructures.html)
    - iterableのやつを全部入れることができるらしい。知らなかった。
    - でも+=と一緒か
    - 下の解法は最初一瞬わかりにくいと思ったが、分かればシンプル　再帰の引き継ぎをfor文とextendでやってるイメージ

```python
class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        all_subsets = [[]]
        for num in nums:
            all_subsets += [subset + [num] for subset in all_subsets]
        return all_subsets
```

- https://github.com/SuperHotDogCat/coding-interview/pull/18/files
- https://discord.com/channels/1084280443945353267/1200089668901937312/1222078986759311410
    - 命名について参考

## Step3

- 結局これが一番好き

```python

class Solution:
    def subsets(self, nums: List[int]) -> List[List[int]]:
        def subset_after(index):
            if index == len(nums):
                return [[]]
            inclusive = subset_after(index + 1)
            not_inclusive = subset_after(index + 1)
            for subset in not_inclusive:
                subset.append(nums[index])
            return inclusive + not_inclusive
        return subset_after(0)
```

## Step4

Step3と同じロジックをループで実装
[deepcopyを知らなかった](https://docs.python.org/3/library/copy.html)ので勉強になった

```python

class Solution:
  def subsets(self, nums: List[int]) -> List[List[int]]:
    copied_nums = nums.copy()
    all_subsets = [[]]
    while copied_nums:
      num = copied_nums.pop()
      num_include_subsets = copy.deepcopy(all_subsets)
      for subset in num_include_subsets:
        subset.append(num)
      all_subsets += num_include_subsets
    return all_subsets
