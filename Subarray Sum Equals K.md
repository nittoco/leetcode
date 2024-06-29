## Step1

- ツーパスを直せないか考えたが思いつかない
- 手でやるならこのようにやるな、という手順でやったつもり
- before_sums、current_sumあたり変数名はいいのかよくわからない

```python

from collections import defaultdict

class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        current_sum = 0
        number_of_sums = defaultdict(int)
        before_sums = []
        for num in nums:
            before_sums.append(current_sum)
            number_of_sums[current_sum] += 1
            current_sum += num
        before_sums.append(current_sum)
        number_of_sums[current_sum] += 1
        subarray_num = 0
        for before_sum in before_sums:
            number_of_sums[before_sum] -= 1
            subarray_num += number_of_sums[before_sum + k]
        return subarray_num
```

## Step2

https://github.com/Mike0121/LeetCode/pull/33/files

https://github.com/Yoshiki-Iwasa/Arai60/pull/15/files

https://github.com/TORUS0818/leetcode/pull/18/files

https://github.com/fhiyo/leetcode/pull/19/files

https://github.com/hayashi-ay/leetcode/pull/31/files

- 左から右ではなく、右から左を見れば単によかった(ワンパスの方法) 、なぜか思いつかなかった
- prefix_sumという単語は知らなかった
- 辞書の時にkey_to_valueという命名が分かりやすく便利なことに気づく
- 累積和は既にわかっていた。[この辺](https://discord.com/channels/1084280443945353267/1233603535862628432/1252232545056063548)をあらためて読んで分かりやすいなと思った

```python

from collections import defaultdict

class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        prefix_sum_to_count = defaultdict(int)
        current_prefix_sum = 0
        subarray_count = 0
        prefix_sum_to_count[0] = 1
        for num in nums:
            current_prefix_sum += num
            subarray_count += prefix_sum_to_count[current_prefix_sum - k]
            prefix_sum_to_count[current_prefix_sum] += 1
        return subarray_count
```

- defaultdictを使わないバージョンも書いてみた。defaultdictがあるのにあえてgetを選択するのはどのような場面がいいのか、自分の中でしっくりきていない。

```python

class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        prefix_sum_to_count = {}
        prefix_sum_to_count[0] = 1
        prefix_sum = 0
        subarray_count = 0
        for num in nums:
            prefix_sum += num
            subarray_count += prefix_sum_to_count.get(prefix_sum - k, 0)
            prefix_sum_to_count[prefix_sum] = prefix_sum_to_count.get(prefix_sum, 0) + 1
        return subarray_count
```

## Step3

```python

from collections import defaultdict

class Solution:
    def subarraySum(self, nums: List[int], k: int) -> int:
        prefix_sum = 0
        prefix_sum_to_count = defaultdict(int)
        prefix_sum_to_count[0] = 1
        subarray_count = 0
        for num in nums:
            prefix_sum += num
            subarray_count += prefix_sum_to_count[prefix_sum - k]
            prefix_sum_to_count[prefix_sum] += 1
        return subarray_count
```
