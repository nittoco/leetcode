## Step1

- 最初にパッと思いついたやつ、時間計算量O(len(nums)^2)なので、ギリギリ
- if nums[i] == 0: continue とするかは迷った
    - インデントが結構今だと深いので
- 左への移動をわざわざ関数として切り分けるかも迷った、これくらいならいらないかも（むしろややこしい？）
- これを書いた後もっといいやり方を思いついた（下に書く）

```python

class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        def move_nonzero_left(nonzero_index):
            nums[nonzero_index - 1], nums[nonzero_index] = nums[nonzero_index], nums[nonzero_index-1]
            return nums[nonzero_index - 1], nums[nonzero_index]

        for i in range(len(nums)):
            if nums[i]:
                nonzero_index = i
                while nonzero_index-1 >= 0 and nums[nonzero_index - 1] == 0:
                    nums[nonzero_index - 1], nums[nonzero_index] = move_nonzero_left(nonzero_index)
                    nonzero_index -= 1
```

- 次に思いついたやつ、計算量は改善できた
- ツーパスになっているが、解消方法は思いつかない

```python

class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        nonzero_index = 0
        zero_count = 0
        for num in nums:
            if num == 0:
                zero_count += 1
                continue
            nums[nonzero_index] = num
            nonzero_index += 1
        for i in range(zero_count):
            zero_index = -i - 1 
            nums[zero_index] = 0
```

## Step2

[https://github.com/Mike0121/LeetCode/pull/2](https://github.com/Mike0121/LeetCode/pull/24) </br>
https://github.com/hayashi-ay/leetcode/pull/58/files

- 最後にゼロ埋めするとき、for i in range(zero_count)でインデックスをマイナスするよりfor i in range(nonzero_count, len(nums))の方がわかりやすいかもと思う
    - 変数もnonzeroを数えるやつ、zeroを数えるやつ2つもいらない
- 変数名を考える。nonzero_indexよりnonzero_countの方がわかりやすいかも

```python

class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        nonzero_count = 0
        for num in nums:
            if num == 0:
                continue
            nums[nonzero_count] = num
            nonzero_count += 1
        for i in range(nonzero_count, len(nums)):
            nums[i] = 0
```

- swapをするなら、left_most_zeroを管理すればO(len(nums)^2)のswap操作をしなくていいのか、書いてみたけどこれが一番スッキリするかも

```python

class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        nonzero_count = 0
        for i in range(len(nums)):
            if nums[i] == 0:
                continue
            nums[nonzero_count], nums[i] = nums[i], nums[nonzero_count]
            nonzero_count += 1
```

https://discord.com/channels/1084280443945353267/1192736784354918470/1199759433543200819

- これは自分には読みにくく逆に勉強になった(すみません)　素直に処理を書く
- 「行き先」としてswapする先をdestinationと管理するのは一つアリかな

[https://en.wikipedia.org/wiki/Erase–remove_idiom](https://en.wikipedia.org/wiki/Erase%E2%80%93remove_idiom)

- 上記は連想する資料として勉強
- C++のremoveの方が、後ろの要素は未定義でいいので実装が楽だと思う

## Step3

```python
class Solution:
    def moveZeroes(self, nums: List[int]) -> None:
        """
        Do not return anything, modify nums in-place instead.
        """
        nonzero_count = 0
        for i in range(len(nums)):
            if nums[i] == 0:
                continue
            nums[nonzero_count], nums[i] = nums[i], nums[nonzero_count]
            nonzero_count += 1
```
