'''
python3　　<br>
class Solution:<br>
    def twoSum(self, nums: List[int], target: int) -> List[int]:　　<br>
        for i in range(len(nums)):　　<br>
            for j in range(len(nums)):　　<br>
                if i != j and nums[i] + nums[j] == target:　　<br>
                    return [i, j]　　<br>
'''
