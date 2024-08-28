### Step1

- 本当はFalseが来た時にすぐ返したい。(いちいちmin, maxを計算するのも大変)nonlocalの方がよかったか
    - でもnonlocalはnonlocalで実装がちょっと見にくいかな
- returnする値も多い気もする
    - 引数が多いのと、returnする値が多いのと、どちらの方がいいのか、選択の基準を持てていない
- calculate_min_max_and_check_validは一つの関数で2つの動作、役割があり、微妙な気もするが、しょうがない？

```python

class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def calculate_min_max_and_check_valid(node):
            if not node:
                return inf, -inf, True

            def is_current_node_valid(node):
                return left_max < node.val < right_min

            left_min, left_max, left_valid = calculate_min_max_and_check_valid(
                node.left
            )
            right_min, right_max, right_valid = calculate_min_max_and_check_valid(
                node.right
            )
            min_so_far = min(left_min, right_min, node.val)
            max_so_far = max(left_max, right_max, node.val)
            valid_so_far = left_valid and right_valid and is_current_node_valid(node)
            return min_so_far, max_so_far, valid_so_far

        _, _, is_valid = calculate_min_max_and_check_valid(root)
        return is_valid
```

## Step2

- [探索順自体をin-orderにする方法もあった](https://github.com/TORUS0818/leetcode/pull/30/files)
    - あと、math.infがマジックナンバーぽいという意見もあった。すぐわかるものだと思っていた。
- https://github.com/kazukiii/leetcode/pull/29/files
    - Noneも入れちゃう実装が好きだったが、入れない方が早くなるかもという
    - &の不要(?)
- https://github.com/sakupan102/arai60-practice/pull/29/files
    - float(”inf”)とmath.infがあるの知らなかった(実装を読んでみる)
        - float(”inf”)は[PyFloatFromString](https://github.com/python/cpython/blob/ce4b9c8464706a58d0c98c2b0deeec07e7496ccc/Objects/floatobject.c#L178)から[_Py_string_to_number_with_underscores](https://github.com/python/cpython/blob/main/Python/pystrtod.c#L345)を呼び出される。その後結局、innerfuncである[float_from_string_inner](https://github.com/python/cpython/blob/ce4b9c8464706a58d0c98c2b0deeec07e7496ccc/Objects/floatobject.c#L138) が呼び出されて、[PyOS_string_to_double](https://github.com/python/cpython/blob/main/Python/pystrtod.c#L299)が来て、[_PyOS_ascii_strtod](https://github.com/python/cpython/blob/main/Python/pystrtod.c#L93)が来て、[_Py_parse_inf_or_nan](https://github.com/python/cpython/blob/main/Python/pystrtod.c#L28)がくる(やっと見つかった、、、)ここで頑張って+-符号を含めて文字列をparseしている、なぜかPy_HUGE_VALになっている
        - math.infはどこで定義されているのかよくわからない
    - sys.maxsizeについて
        - intっぽいのでこっちの方がいいかも([参考](https://docs.python.org/ja/3/whatsnew/3.0.html#integers))　厳密な最大ではないっぽいが
        
        > • 整数の上限がなくなったため、`sys.maxint` 定数は削除されました。しかしながら、通常のリストや文字列の添え字よりも大きい整数として [`sys.maxsize`](https://docs.python.org/ja/3/library/sys.html#sys.maxsize) を使うことができます。 [`sys.maxsize`](https://docs.python.org/ja/3/library/sys.html#sys.maxsize) は実装の "自然な" 整数の大きさに一致し、同じプラットフォームでは (同じビルドオプションなら) 過去のリリースの `sys.maxint` と普通は同じです。
        > 
- https://github.com/fhiyo/leetcode/pull/30#discussion_r1660235486
    - iterativeでやる方法
    - inorderもiterativeでできる(スタック領域におけるリターンアドレスのようなものを入れればできるの賢い)
    - よく考えれば前やった行きと帰りをstackに入れるやつの応用か
- https://discord.com/channels/1084280443945353267/1200089668901937312/1213043194242015252
    - inorderのいろんな書き方

1, preorder, 上からvalidateする方法、再帰、nodeがNoneの時は別で実装してinfを避ける

```python
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def is_valid_val(node, left_min, right_max):
            if not node:
                return True
            if left_min is not None and left_min >= node.val:
                return False
            if right_max is not None and right_max <= node.val:
                return False
            return is_valid_val(node.left, left_min, node.val) \
            and is_valid_val(node.right, node.val, right_max)

        return is_valid_val(root, None, None)
```

2, inorderでの再帰　yield fromの利用

- generatorにすればfor文で回せるので、単純なreturnより前との比較が楽

```python
class Solution:
    def generate_nodes_inorder(self, node):
        if not node:
            return
        yield from self.generate_nodes_inorder(node.left)
        yield node
        yield from self.generate_nodes_inorder(node.right)

    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        prev_val = None
        for node in self.generate_nodes_inorder(root):
            if prev_val is not None and node.val <= prev_val:
                return False
            prev_val = node.val
        return True
```

3, inorderでのiterative, 見ている方向をstackにつめる

```python
class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def sort_node_val_inorder(node):
            nodes_stack = [(root, ["left"])]
            inordered_val = []
            while nodes_stack:
                node, next_direction_ref = nodes_stack[-1]
                if not node or next_direction_ref[0] == "parent":
                    nodes_stack.pop()
                    continue
                if next_direction_ref[0] == "left":
                    nodes_stack.append((node.left, ["left"]))
                    next_direction_ref[0] = "right"
                    continue
                if next_direction_ref[0] == "right":
                    inordered_val.append(node.val)
                    nodes_stack.append((node.right, ["left"]))
                    next_direction_ref[0] = "parent"
            return inordered_val

        inordered_val = sort_node_val_inorder(root)
        unique_count = len(set(inordered_val)) 
        return inordered_val == sorted(inordered_val) and len(inordered_val) == unique_count

```

4, inorderでのiterative その2

- なんかスッキリしないのでこの後さらにリファクタ

```python

class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def check_valid(node):
            left_childs_stack = []
            prev_val = None
            while node or left_childs_stack:
                while node:
                    left_childs_stack.append(node)
                    node = node.left
                node = left_childs_stack.pop()
                if prev_val is not None and prev_val >= node.val:
                    return False
                prev_val = node.val
                node = node.right
            return True

        return check_valid(root)
```

5, 上のやつはこの方がいいかも

- 処理として素直な気がする

```python

class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def is_valid(node):
            def append_left_children_to_stack(node):
                left_node = node
                while left_node:
                    left_children_stack.append(left_node)
                    left_node = left_node.left
            
            left_children_stack = []
            append_left_children_to_stack(node)
            prev_node = TreeNode(-sys.maxsize)
            while True:
                if not left_children_stack:
                    return True
                current_node = left_children_stack.pop()
                if prev_node.val >= current_node.val:
                    return False
                if current_node.right:
                    append_left_children_to_stack(current_node.right)
                prev_node = current_node
        
        return is_valid(root)
```

6, Morris in-order([参考](https://discord.com/channels/1084280443945353267/1200089668901937312/1213356258103525407))

- 理解に苦労
- 基本的にはrightに行ってinorder順にたどろうとするが、まだ左が見れてないならばそっちを先に見る
    - その時左にたどるついでに、今のnodeが後で.rightで戻れるようにつなげておく
    - くらいの理解(むずい)

```python

class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def generate_node_inorder(node):
            def search_previous_node_inorder(node):
                if not node.left:
                    return None, True
                prev = node.left
                while prev.right:
                    if prev.right == node:
                        return prev, True
                    prev = prev.right
                return prev, False

            while node:
                prev, left_searched = search_previous_node_inorder(node)
                if not prev:
                    yield node
                    node = node.right
                    continue
                if not left_searched:
                    prev.right = node
                    node = node.left
                    continue
                yield node
                node = node.right
                prev.right = None
                    
        prev_val = None
        for node in generate_node_inorder(root):
            if prev_val is not None and prev_val >= node.val:
                return False
            prev_val = node.val
        return True
```

### Step3

```python

class Solution:
    def isValidBST(self, root: Optional[TreeNode]) -> bool:
        def generate_node_inorder(node):
            if not node:
                return
            yield from generate_node_inorder(node.left)
            yield node
            yield from generate_node_inorder(node.right)
          
        prev_val = None
        for node in generate_node_inorder(root):
            if prev_val is not None and prev_val >= node.val:
                return False
            prev_val = node.val
        return True
```
