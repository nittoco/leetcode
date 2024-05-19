## Step1

- heapqから自前実装してみた。ドキュメントを見てアルゴリズムを理解。スケジューリングにも役立つらしい。また、ディスクソートに効率が良いというのを書いてあり、ディスクソートというのを初めて知った。
    - ドキュメント見ると、その周辺の常識知識がアップデートされるのが良い
    - 参考資料(https://docs.python.org/3/library/heapq.html)(https://github.com/python/cpython/blob/3.12/Lib/heapq.py)

```python
class heapq:
    def heappush(heapq: list[int], new_item: int) -> list[int]:
        heapq.append(new_item)
        child_idx = len(heapq) - 1
        if child_idx == 0:
            return heapq
        parent_idx = (child_idx - 1) >> 1
        while child_idx != 0 and heapq[parent_idx] > heapq[child_idx]:
            bigger_value = heapq[parent_idx]
            heapq[parent_idx] = heapq[child_idx]
            heapq[child_idx] = bigger_value
            child_idx = parent_idx
            parent_idx = (child_idx - 1) >> 1
        return heapq

    def heappop(heapq: list[int]) -> list[int]:
        pop_item = heapq[0]
        heapq[0] = last_item
        last_item = heapq.pop()
        last_idx = len(heapq) - 1
        parent_idx = 0
        left_child_idx = 2 * parent_idx + 1
        right_child_idx = 2 * parent_idx + 2
        # 自分より小さい子供がいるならば、子供のうち小さい方を上にあげる
        while left_child_idx <= last_idx:
            if (
                right_child_idx <= last_idx
                and heapq[left_child_idx] > heapq[right_child_idx]
            ):
                min_child_idx = right_child_idx
            else:
                min_child_idx = left_child_idx
            if heapq[min_child_idx] > heapq[parent_idx]:
                break
            min_child_value = heapq[min_child_idx]
            heapq[min_child_idx] = heapq[parent_idx]
            heapq[parent_idx] = min_child_value
            parent_idx = min_child_idx
            left_child_idx = 2 * parent_idx + 1
            right_child_idx = 2 * parent_idx + 2
        return pop_item

```

- フォーマッターにかけたらif文内が改行されて、カッコがついた（長い時はこうするのが良い？）

## Step2

- ここで1回他の人の実装を見てみる

https://github.com/shining-ai/leetcode/pull/8/files

- 上記のkandaさんのやつを見るとかなりクラスの内部関数を作っている。こんなに分ける選択肢がなかった。
    - こういうある程度規模のある実装が初めてで、**どれくらいで関数を分けるかの感覚がない**（もちろん最終的には好みだが、その好みの判断基準の感覚すらない、他の皆さんはどう考えてるんだろう）
- たまたま見つけた。確かにswapするとき一回変数保存するの厄介だなと思ってたのでa, b = c, dの方が良さそう
    
    https://discord.com/channels/1084280443945353267/1196472827457589338/1196541234169266387
    
    - そして、このswap処理を分けるのが良さそう
- classの名称はCapWordであることを思い出す
- [この辺](https://github.com/sakupan102/arai60-practice/pull/9/files#diff-fa666eb35a92f951b1928e29857ece2144f24557eeee7eda85275639cd6e3693)とか見てみた。parentの値に関してコメントしてみた。
- [これ](https://github.com/TORUS0818/leetcode/pull/10/files#diff-834e73e8238c79fda22f0cf168ae9c2d4b99e7e7db94424dfa8e4f594f2a9ed2)も見た
- ところで、初期化でheapにする実装、みんな_sift_upを1個ずつの要素にしていて、自分みたいにsortしてない。sortしたら結果的に二分木になるはず
    - なんか避けた理由がある？
- len()の組み込み関数がどうclassで実装されてるかを初めて知った(https://docs.python.org/ja/3/reference/datamodel.html#object.__len__)
- swapのところでPythonの関数の引数のローカル代入の知識がなく引っかかる
    - liquo_riceさんからの記事を[まとめた](https://discord.com/channels/1084280443945353267/1226508154833993788/1241432108937908326)
    - nonlocalは代入の時だけでappendとかは違う by odaさん（[参考](https://discord.com/channels/1084280443945353267/1226508154833993788/1241494615941709977)）
- たまたま発見 childだけ引き継ぎparentの計算は次の日の人に任せたほうが綺麗という感覚もあるらしい(https://discord.com/channels/1084280443945353267/1227073733844406343/1230564682851553310)

```python

class BinaryHeapTree:
  
  def __init__(self, heap: list[int] = None):
    if not heap:
      heap = []
    self.heap = sorted(heap)
  
  def __str__(self):
    return str(self.heap)
  
  def push(self, element: int) -> int:
    self.heap.append(element)
    self._siftdown(len(self.heap) - 1)
    return self.heap

  def pop(self) -> int | str:
    if not self.heap:
      return 'No Elements in BinaryHeapTree'
    poped_element = self.heap[0]
    last_element = self.heap[-1]
    self.heap[0] = last_element
    self.heap.pop()
    self._siftup(0)
    return poped_element
    
  def top(self):
    return self.heap[0]

  def _siftup(self, pos: int):
    parent = pos
    min_child = self._get_min_child(parent)
    while min_child < len(self.heap) and self.heap[parent] > self.heap[min_child]:
      self._swap_element(parent, min_child)
      parent = min_child
      min_child = self._get_min_child(parent)

  def _siftdown(self, pos: int):
    child = pos
    parent = self._get_parent(child)
    while child > 0 and self.heap[parent] > self.heap[child]:
      self._swap_element(parent, child)
      child = parent
      parent = self._get_parent(child)

  def _swap_element(self, idx_1: int, idx_2: int):
    self.heap[idx_2], self.heap[idx_1] = self.heap[idx_1], self.heap[idx_2]
    
  def _get_min_child(self, parent: int) -> int:
    min_child = 2*parent + 1
    right_child = 2*parent + 2
    if right_child < len(self.heap) and self.heap[right_child] < self.heap[min_child]:
      min_child = right_child
    return min_child

  def _get_parent(self, child: int) -> int:
    parent = (child - 1) >> 1
    return parent
  
  def __len__(self):
    return len(self.heap)

class KthLargest:

    def __init__(self, k: int, nums: List[int]):
        self.k = k
        self.nums = BinaryHeapTree(nums) 

    def add(self, val: int) -> int:
        self.nums.push(val)
        while len(self.nums) > self.k:
            self.nums.pop()
        return self.nums.top()
```

## Step3

- 初期化の段階で最初のk個だけにした方が早いかもという意見も発見、確かに。それをもとにKthLargestを再実装し、3連accept

```python

class KthLargest:

    def __init__(self, k: int, nums: List[int]):
        self.nums = BinaryHeapTree(nums[:k])
        self.k = k

    def add(self, val: int) -> int:
        self.nums.push(val)
        while len(self.nums) > self.k:
            self.nums.pop()
        return self.nums.top()
```
