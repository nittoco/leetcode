- 再帰
    - またcopy()をし忘れ最初戸惑った
    - indexという変数名とiという変数名があるのは微妙な気がする
    - 「in_placeで変更せずにlistに挿入する」というのの記述方法がStep2まで思いつかなかったので、無理矢理swapを使ってなんかややこしいコードになってしまった

```python

class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        def helper(index):
            if(index == len(nums)):
                return [[]]
            all_permutations = []
            part_of_permutations = helper(index + 1)
            for permutation in part_of_permutations:
                permutation.append(nums[index])
                all_permutations.append(permutation)
                for i in reversed(range(len(permutation) - 1)):
                    permutation = permutation.copy()
                    permutation[i], permutation[i+1] = permutation[i+1], permutation[i]
                    all_permutations.append(permutation)
            return all_permutations

        return helper(0)
```

- ループ 最初all_permutationsに直接appendしてループを無限にしてしまった

```python
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        copied_nums = nums.copy()
        all_permutations = [[]]
        while copied_nums:
            num = copied_nums.pop()
            permutations_add = []
            for permutation in all_permutations:
                permutation.append(num)
                inserted_permutation = permutation.copy()
                for i in reversed(range(len(permutation) - 1)):
                    inserted_permutation[i], inserted_permutation[i+1] = inserted_permutation[i+1], inserted_permutation[i]
                    permutations_add.append(inserted_permutation.copy())
            all_permutations += permutations_add
        return all_permutations

```

## Step2

https://github.com/hayashi-ay/leetcode/pull/57/files

https://github.com/shining-ai/leetcode/pull/50/files

https://github.com/Mike0121/LeetCode/pull/14/files

https://github.com/Exzrgs/LeetCode/pull/19/files

https://github.com/SuperHotDogCat/coding-interview/pull/12/files

- Step1の再帰でやる方法の書き直し。無理矢理swapせずに、[:i] + [nums] + [i:]をappendすればよかったね
    - いちいちcopy()したりめちゃまどろこしかった
    - 新しいlistを、in-placeでなく既存のを少し変更したのを作るなら、＋を使うと覚えておく

```python

class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        all_permutations = []
        if len(nums) == 0:
            return [[]]
        added_permutations = self.permute(nums[1:])
        added_number = nums[0]
        for permutation in added_permutations:
            for i in range(len(permutation) + 1):
                all_permutations.append(permutation[:i] + [added_number] + permutation[i:])
        return all_permutations
```

- 再帰でも、swapしてから再帰する方法
- next_permutationを順に求める方法
    - 「自分より右側の大きいのとswapできるもの」の中で一番右側を探す
    - この辺を見ている限り、関数化した方が良いのかな、処理も長いし
        - https://github.com/Mike0121/LeetCode/pull/15/files
    - どういう時にNoneを返すかも分かりにくいかもしれない？？
    - iという変数名、indexという変数名があり、混乱しないようなんとかする必要がありそう
    - 入力が破壊されるとマルチスレッドの時にどうなるかということを一応考慮し、入力をcopyした

```python
# 変数名、関数の分け方に改善の余地あり
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        def next_permutation(nums):
            next_permute_nums = nums.copy()
            right_max = -inf
            for i in reversed(range(len(next_permute_nums))):
                if next_permute_nums[i] > right_max:
                    right_max = next_permute_nums[i] 
                    continue
                bigger_min_value = inf
                for index in range(i+1, len(next_permute_nums)):
                    if next_permute_nums[index] < next_permute_nums[i] or next_permute_nums[index] > bigger_min_value:
                        continue
                    bigger_min_value = next_permute_nums[index]
                    bigger_min_index = index
                next_permute_nums[i], next_permute_nums[bigger_min_index] = next_permute_nums[bigger_min_index], next_permute_nums[i]
                next_permute_nums[i+1:] = reversed(next_permute_nums[i+1:])
                return next_permute_nums
            return None

        permutation = sorted(nums).copy()
        all_permutations = []
        while permutation:
            all_permutations.append(permutation)
            permutation = next_permutation(permutation)
        return all_permutations
```

- 上のを改善。pivotというindexの名前はshining_aiさんのを参考に、自力では思いつかなかった
- 例外処理は選択肢があったが、find_swap_indexは(-1, -1)を返した方が後続の楽だと思いこうした
    - ただ-1だと一応indexとして代入できちゃうので、例外を出してもよかったかも、、、この辺判断難しい
- [関数名は動詞を使うと良い](https://github.com/hayashi-ay/leetcode/pull/67/files)と書いてあったのでそうした

```python

class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        def make_next_permutation(permutation):
            def find_decreasing_index_from_last(permutation):
                max_num = -inf
                for i in reversed(range(len(permutation))):
                    if permutation[i] < max_num:
                        return i
                    max_num = max(permutation[i], max_num)
                return None
            
            def find_swap_index(permutation):
                pivot = find_decreasing_index_from_last(permutation)
                if pivot is None:
                    return -1, -1
                min_of_greater = inf
                for i in range(pivot+1, len(permutation)):
                    if permutation[pivot] < permutation[i] < min_of_greater:
                        min_of_greater = permutation[i]
                        swap = i
                return pivot, swap

            pivot, swap = find_swap_index(permutation)
            if pivot == -1 and swap == -1:
                return None
            permutation[pivot], permutation[swap] = permutation[swap], permutation[pivot]
            permutation[pivot+1:] = list(reversed(permutation[pivot+1:]))
            return permutation

        all_permutations = []
        current_permutation = sorted(nums).copy()
        while current_permutation:
            all_permutations.append(current_permutation)
            current_permutation = make_next_permutation(current_permutation.copy())
        return all_permutations
```

- バックトラッキング
    - 意外と楽にさっと実装できた

```python

class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        def helper(permutation):
            if len(permutation) == len(nums):
                all_permutations.append(permutation.copy())
                return
            for num in nums:
                if num in permutation:
                    continue
                permutation.append(num)
                helper(permutation)
                permutation.pop()
                
                
        all_permutations = []
        helper([])
        return all_permutations
```

- Stackを利用したDFS。慣れてないからあまりスッキリしない

```python
python
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        permutation_and_remaining = [([], nums)]
        all_permutations = []
        while permutation_and_remaining:
            permutation, remaining = permutation_and_remaining.pop()
            if len(remaining) == 0:
                all_permutations.append(permutation.copy())
            for i in range(len(remaining)):
                new_permutation = permutation + [remaining[i]]
                new_remaining = remaining[:i] + remaining[i+1:]
                permutation_and_remaining.append((new_permutation , new_remaining))
        return all_permutations
```

- permutation_and_remaningを用いるなら、以下のBFSでの実装の方が引き継ぎの感覚としてしっくりくるかも

```python

from collections import deque

class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        permutations_and_remaining = deque()
        permutations_and_remaining.append(([], nums))
        all_permutations = []
        while permutations_and_remaining:
            permutation, remaining = permutations_and_remaining.popleft()
            if len(permutation) == len(nums):
                all_permutations.append(permutation)
                continue
            for num in remaining:
                added_permutation = permutation + [num]
                added_remaining = [x for x in remaining if x != num]
                permutations_and_remaining.append((added_permutation, added_remaining))
        return all_permutations
```

- [Pythonのitertoolsの実装](https://docs.python.org/3/library/itertools.html#itertools.permutations)を見てみた
    - i とcycle[i]　の値の推移をprintしてみると、[0, 1, 2, 3]のpermutationの場合
        - (3, 0)→(2, 1)  [0, 1, 3, 2]
        - (3, 0)→(2, 0)→(1, 2) [0, 2, 1, 3]
        - (3, 0)→(2, 1) [0, 2, 3, 1]
        - (3, 0)→(2, 0)→(1, 1) [0, 3, 1, 2]
        - (3, 0)→(2, 1) [0, 3, 2, 1]
        - (3, 0)→(2, 0)→(1, 0)→(0, 3) [1, 0, 2, 3]
        - 以下つづく
    - cycle[i]は0の時、例えば(3, 0)→(2, 0)→(1, 0)なら1から3のindexの値を昇順ソート(ソート前は降順になってるのでうまくいく)
    - 左から見て最初に変更するindexの右側をソートした後に、その左のやつを適当なところに挿入する感じ？

## Step3

- これが一番簡潔かも

```python
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        if len(nums) == 0:
            return [[]]
        all_permutations = []
        added_permutations = self.permute(nums[1:])
        added_num = nums[0]
        for permutation in added_permutations:
            for i in range(len(nums)):
                all_permutations.append(permutation[:i] + [added_num] + permutation[i:])
        return all_permutations
```
