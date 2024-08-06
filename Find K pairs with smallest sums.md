### Step1

- どっかで見たことあったので解けた
- 最初、すでに突っ込んだペアかのチェックを忘れてた

```python

class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        num_pairs = 0
        pairs_heap = [(nums1[0] + nums2[0], 0, 0)]
        result_pairs = []
        checked = set((0, 0))
        while len(result_pairs) < k:
            pair_sum, index1, index2 = heappop(pairs_heap)
            if (index1, index2) in checked:
                continue
            checked.add((index1, index2))
            result_pairs.append([nums1[index1], nums2[index2]])
            if index1 + 1 < len(nums1):
                heappush(pairs_heap, (nums1[index1 + 1] + nums2[index2], index1 + 1, index2))
            if index2 + 1 < len(nums2):
                heappush(pairs_heap, (nums1[index1] + nums2[index2 + 1], index1, index2 + 1))
        return result_pairs
```

## Step2

- 参考資料
    - https://github.com/sendahuang14/leetcode/pull/9/files
    - https://github.com/TORUS0818/leetcode/pull/12
    - https://github.com/seal-azarashi/leetcode/pull/10/files
    - https://github.com/ryoooooory/LeetCode/pull/17
    - [https://github.com/kazukiii/leetcode/pull/11](https://github.com/kazukiii/leetcode/pull/11#discussion_r1643725895)
    - https://discord.com/channels/1084280443945353267/1235829049511903273/1246118347863621652
- 論点
    - heapqに入れられるかどうかは、関数化してもいいかも。関数化する選択肢、自分あまり持てないがちなので注意
        - 関数化したらearly returnも使える
    - set()を使わずにheapqに入れるものを制限する方法もある
        - 左と上の両方が揃ってから初めて入れる
        - 各行、各列で、どこまで入れたかの配列を持つ
        - set()を使うのが好ましくないというシチュエーションがいまいちわからない

```python
# 各行、各列でどこまで入れたかの配列をもつ
# この下でこのコードをリファクタリング
class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        index1_to_appended_index2 = [-1] * len(nums1)
        index2_to_appended_index1 = [-1] * len(nums2)
        sum_and_index_heap = []
        heapq.heappush(sum_and_index_heap, (nums1[0] + nums2[0], 0, 0))
        
        def add_heap_if_needed(index1, index2):
            if index1 >= len(nums1) or index2 >= len(nums2):
                return
            if not index1_to_appended_index2[index1] == index2 - 1 or not index2_to_appended_index1[index2] == index1 - 1:
                return
            heapq.heappush(sum_and_index_heap, (nums1[index1] + nums2[index2], index1, index2))

        result_pairs = []
        while len(result_pairs) < k:
            pair_sum, index1, index2 = heapq.heappop(sum_and_index_heap)
            result_pairs.append([nums1[index1], nums2[index2]])
            index1_to_appended_index2[index1] = index2
            index2_to_appended_index1[index2] = index1
            add_heap_if_needed(index1 + 1, index2)
            add_heap_if_needed(index1, index2 + 1)
        return result_pairs
```

- 上のコードについて
    - 変数名が微妙かも。index1_to_appended_index2は、index1_to_index2_in_resultとか？
    - この辺(https://github.com/TORUS0818/leetcode/pull/12)見る感じ
        - 関数化が意図を伝えられるというのなるほど。もうちょっと関数化して、意図を伝えたほうが良さそう？(今までのやつあんまりしてこなかったけど）
        - sum_and_index_heapはデータの中身をまあ表してはいるけど、読む立場の時にcandidatesの方が意図が分かりやすそう
        - len(nums1)*len(nums2)がkを超えない場合の処理も考えてみる
- 上記に気を付けて、結構関数化してみた（やり過ぎ？）

```python

class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        def is_inner_index(index1, index2):
            return 0 <= index1 < len(nums1) and 0 <= index2 < len(nums2)

        def is_last_pairs(index1, index2):
            return index1 == len(nums1) - 1 and index2 == len(nums2) - 1

        def is_before_index1_in_result(index1, index2):
            return index2_to_index1_in_result[index2] == index1 - 1

        def is_before_index2_in_result(index1, index2):
            return index1_to_index2_in_result[index1] == index2 - 1

        def add_candidates_if_needed(index1, index2):
            if not is_inner_index(index1, index2):
                return
            if not is_before_index1_in_result(index1, index2) or not is_before_index2_in_result(index1, index2):
                return
            heappush(candidates, (nums1[index1] + nums2[index2], index1, index2))

        index1_to_index2_in_result = [-1] * len(nums1)
        index2_to_index1_in_result = [-1] * len(nums2)
        candidates = [(nums1[0] + nums2[0], 0, 0)]
        result_pairs = []
        while len(result_pairs) < k:
            pair_sum, index1, index2 = heappop(candidates)
            result_pairs.append([nums1[index1], nums2[index2]])
            index1_to_index2_in_result[index1] = index2
            index2_to_index1_in_result[index2] = index1
            if is_last_pairs(index1, index2):
                break
            add_candidates_if_needed(index1 + 1, index2)
            add_candidates_if_needed(index1, index2 + 1)
        return result_pairs
```

[ここ](https://discord.com/channels/1084280443945353267/1235829049511903273/1246118347863621652)　にあった変わったやつも実装

- ジェネレータの再帰をどうするか最初わからなかったが、イテレータになるのでfor文を使えばよかった
- TLEになった。時間計算量を考えたけどわからない、、、
- 再帰の計算量を考えるのが苦手

```python

class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        def generate_smallest_pairs(index2):
            if index2 == len(nums2):
                yield [inf, inf]
                return
            index1 = 0
            for next_pair in generate_smallest_pairs(index2 + 1):
                while index1 < len(nums1) and nums1[index1] + nums2[index2] < sum(next_pair):
                    yield [nums1[index1], nums2[index2]]
                    index1 += 1
                yield next_pair
            
        return list(islice(generate_smallest_pairs(0), k))
```

## Step3

```python

class Solution:
    def kSmallestPairs(self, nums1: List[int], nums2: List[int], k: int) -> List[List[int]]:
        def is_valid_index(index1, index2):
            return 0 <= index1 < len(nums1) and 0 <= index2 < len(nums2)
        
        def is_possible_candidate(index1, index2):
            if index1 == 0 or index2 == 0:
                return True
            return (index1 - 1, index2) in index_pairs_in_result\
            and (index1, index2 - 1) in index_pairs_in_result

        def is_last_pairs(index1, index2):
            return index1 == len(nums1) - 1 and index2 == len(nums2) - 1
        
        candidates = [(nums1[0] + nums2[0], 0, 0)]
        result_pairs = []
        index_pairs_in_result = set()
        while len(result_pairs) < k:
            pair_sum, index1, index2 = heapq.heappop(candidates)
            result_pairs.append([nums1[index1], nums2[index2]])
            if is_last_pairs(index1, index2):
                return result_pairs
            index_pairs_in_result.add((index1, index2))
            diffs = [(0, 1), (1, 0)]
            for diff in diffs:
                diff1, diff2 = diff
                new_index1 = index1 + diff1
                new_index2 = index2 + diff2
                if not is_valid_index(new_index1, new_index2):
                    continue
                if not is_possible_candidate(new_index1, new_index2):
                    continue
                new_sum = nums1[new_index1] + nums2[new_index2]
                heapq.heappush(candidates, (new_sum, new_index1, new_index2))
        return result_pairs
```
