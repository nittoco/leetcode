## Intersection of Two Arrays

## Step1

- setで積集合を取るときなんだったか忘れたのでドキュメントを読んだ
- 出力の形式に今回は気をつけてAC

```python

class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        unique_nums1 = set(nums1)
        unique_nums2 = set(nums2)
        return list(unique_nums1 & unique_nums2)
```

## Step2

https://discord.com/channels/1084280443945353267/1196472827457589338/1196541121912909824 の49を読んで色々な解放があるなあと

- 双方を sorted にする解法
    - [後で見た](https://github.com/shining-ai/leetcode/pull/13/files)(step3のahayashiさんのコメント)が、確かに例外ではないのでcontinueじゃなくてelif, elseでいいかも？
    - と思ったが、intersectionに入れない処理はスルーしてるイメージなので、やはりcontinueでいい気もしてきた
        - 最近、前とは逆になんでもcontinue全部つかいがちな癖もあるので、選択肢という意味では見てよかった
    - 最初自分は共通のsetをintersectionという変数名にしていたが、関数名と同じなのは微妙な気がしてやめた。([これ](https://github.com/hayashi-ay/leetcode/pull/21/files)みてcommonという変数名がよいと思いこれに）

```python

class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        index_1 = 0
        index_2 = 0
        nums1 = sorted(nums1)
        nums2 = sorted(nums2)
        common = set()
        while index_1 < len(nums1) and index_2 < len(nums2):
            if nums1[index_1] < nums2[index_2]:
                index_1 += 1
                continue
            if nums1[index_1] > nums2[index_2]:
                index_2 += 1
                continue
            common.add(nums1[index_1])
            index_1 += 1
            index_2 += 1
        return list(common)
```

- 片方を、dict か set にする
    - 変数名、elementとか回りくどく説明しなくても、これくらいならnumでもいいかも、となんとなく思う

```python

class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        nums2 = set(nums2)
        common = set()
        for element_1 in nums1:
            if element_1 in nums2:
                common.add(element_1)
        return list(common)
```

- 両方 set にして、set intersection。

```python

class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        nums1 = set(nums1)
        nums2 = set(nums2)
        return list(nums1.intersection(nums2))
```

[setのドキュメント](https://docs.python.org/ja/3/library/stdtypes.html#set-types-set-frozenset)を1通り流し読み

- frozensetというのもあるのね
- 差集合とか、部分集合かの判定とかもできるのね
- setの要素はhashableじゃなきゃいけないので、listとかはダメなのね

## Step3

- これでやってみた。elementという変数名を変えた。

```python
class Solution:
    def intersection(self, nums1: List[int], nums2: List[int]) -> List[int]:
        nums2 = set(nums2)
        common = set()
        for num in nums1:
            if num in nums2:
                common.add(num)
        return list(common)
```
