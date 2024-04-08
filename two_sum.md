1, まずは愚直な方法で実施。mapとか使えば高速になるかなと思ったが、Pythonでの書き方を忘れてしまった
```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:　
        for i in range(len(nums)):　　
            for j in range(len(nums)):　　
                if i != j and nums[i] + nums[j] == target:　　
                    return [i, j]　　
```

2, 以下の[GitHub](https://github.com/Kitaken0107/GrindEasy/pull/4)　を参考に
ハッシュテーブルで実装した<br>
該当のindexがなかった場合の処理も実装
```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        nums_dict = {value: idx for idx, value in enumerate(nums)}
        for idx, value in enumerate(nums):
            other_value = target - value
            if(idx!=nums_dict[other_value]):
                return [idx, nums_dict[other_value]]
        return [0, 0]
```
