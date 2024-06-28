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

## Step3(Step4?) 後日復習
- 初めてやった問題の復習
- try exceptで実装してみた。
- エラーハンドリングは実装したことがないしよく知らなかったため、この辺の文章を読んだ。(https://docs.python.org/3/tutorial/errors.html)
- 確かに処理が追いにくいし、tryが長いとどこのエラーだかわからなくなるかも
-　built-in exceptionは、今まで何となくみてたエラーメッセージの意味が明確にわかり、勉強になった(https://docs.python.org/3/library/exceptions.html)
- [Google Style Code Python](https://google.github.io/styleguide/pyguide.html)　では、finallyを実装して必ず終了してくださいとあるが、ペアが存在しない場合とする場合で明らかに処理が違うので、finallyで共通の処理を実装するのは難しい(必ずファイルを閉じるとかはしないので不要？）
```python
def twoSum(nums: list[int], target: int) -> list[int]:
        try:
            num_to_idx = {}
            for idx, num in enumerate(nums):
                matched_num = target - num
                if matched_num in num_to_idx:
                  return num_to_idx[matched_num], idx   
                num_to_idx[num] = idx
            raise ValueError("There are no pairs that meet the conditions in the list")
        except ValueError as e:
            raise 
