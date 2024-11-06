### Step1

- search_parent_of_right_childは入力のcandidatesの要素を変更してしまうが、これはまあしょうがない気がする(実際候補は変更するし、、、)
- ちょっと遅めだったが、nodeからindexの連想配列とか作ると速くなるかなあと何となく思う
    - メモリは余分にかかるけどね
- search_parent_of_right_childのprev_nodeというのはちょっとスッキリしないが代案はない

```python
python
class Solution:
    def is_inorder(self, node, next_node, inorder):
        for i in range(len(inorder)):
            if inorder[i] == node.val:
                node_index = i
            if inorder[i] == next_node.val:
                next_node_index = i
        return node_index < next_node_index

    def search_parent_of_right_child(self, child_node, candidates, inorder):
        prev_node = None
        while True:
            if not candidates:
                return prev_node
            node_searched = candidates.pop()
            if not self.is_inorder(node_searched, child_node, inorder):
                candidates.append(node_searched)
                return prev_node
            prev_node = node_searched
            
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        assert len(preorder) == len(inorder)
        assert len(preorder) == len(set(preorder))
        assert len(inorder) == len(set(inorder))
        root = TreeNode(preorder[0])
        parent_candidates = [root]
        for i in range(1, len(preorder)):
            child_node = TreeNode(preorder[i])
            if not self.is_inorder(parent_candidates[-1], child_node, inorder):
                parent_node = parent_candidates[-1]
                parent_node.left = child_node
                parent_candidates.append(child_node)
                continue
            parent_node = self.search_parent_of_right_child(child_node, parent_candidates, inorder)
            parent_node.right = child_node
            parent_candidates.append(child_node)
        return root
```

## Step2

https://github.com/TORUS0818/leetcode/pull/31/files

[https://github.com/seal-azarashi/leetcode/pull/29](https://github.com/seal-azarashi/leetcode/pull/29#pullrequestreview-2344592484)

https://github.com/goto-untrapped/Arai60/pull/53

https://github.com/Ryotaro25/leetcode_first60/pull/31

https://github.com/hayashi-ay/leetcode/pull/43

- 再帰でやる方法は思いつかなかった

- 再帰で、子を呼び出す前に全部の処理を終える方法(ややこしいのでこうは書かないが練習のために)、さらにスライスでコピーをしないためにindexで管理
- .right, .leftの後処理さえしないために、引き継ぎに必要な情報は以下3つ
    - 今のtree size
    - 今のtreeで、preorderの最初のところ
    - 今のtreeで、inorderの最初のところ
- 子を呼び出す前に処理するには、必要な情報は引数として渡す必要がある(returnでは渡せない)
- どこまで引数で渡し、どこまでreturnで渡すか、その選択の基準を自分の中でまだ持てていない
- listの[indexメソッド](https://docs.python.org/3/tutorial/datastructures.html)知らなかったのでドキュメント読んだ
    - 複数ある場合は最初の値
    - ない場合はValueError
    - start, endのオプションがあり、その範囲内での相対indexを返す

```python
from dataclasses import dataclass

@dataclass
class SubtreeIndexRange:
    subtree_size: int
    preorder_min_index: int
    inorder_min_index: int

class Solution:
    def calculate_child_index_range(self, parent_subtree: SubtreeIndexRange, parent_index_inorder: int) -> tuple:
        left_tree_size = parent_index_inorder - parent_subtree.inorder_min_index
        left_preorder_min_index = parent_subtree.preorder_min_index + 1
        left_inorder_min_index = parent_subtree.inorder_min_index
        left_subtree = SubtreeIndexRange(left_tree_size, left_preorder_min_index, left_inorder_min_index)
        right_tree_size = parent_subtree.subtree_size - left_tree_size - 1
        right_preorder_min_index = parent_subtree.preorder_min_index + left_tree_size + 1
        right_inorder_min_index = parent_subtree.inorder_min_index + left_tree_size + 1
        right_subtree = SubtreeIndexRange(right_tree_size, right_preorder_min_index, right_inorder_min_index)
        return left_subtree, right_subtree
     
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        assert len(preorder) == len(inorder)
        assert len(preorder) == len(set(preorder))
        assert len(inorder) == len(set(inorder))
        preorder_index_to_node = []
        for val in preorder:
            preorder_index_to_node.append(TreeNode(val))
        node_val_to_inorder_index = {val: i for i, val in enumerate(inorder)}
        def build_tree_helper(subtree: SubtreeIndexRange):
            if subtree.subtree_size <= 1:
                return None
            parent_node = preorder_index_to_node[subtree.preorder_min_index]
            parent_index_inorder = node_val_to_inorder_index[parent_node.val]
            left_subtree, right_subtree = self.calculate_child_index_range(subtree, parent_index_inorder)
            if left_subtree.subtree_size:
                parent_node.left = preorder_index_to_node[left_subtree.preorder_min_index]
            if right_subtree.subtree_size:
                parent_node.right = preorder_index_to_node[right_subtree.preorder_min_index]
            build_tree_helper(left_subtree)
            build_tree_helper(right_subtree)
        build_tree_helper(SubtreeIndexRange(len(preorder), 0, 0))
        return preorder_index_to_node[0]

```

- dataclassのドキュメントと、ソースコード(https://github.com/python/cpython/blob/main/Lib/dataclasses.py)を読んだ
- process_class(L930)で__init__とか__repr__みたいな特殊メソッドを定義して、クラスを作っているみたい。主な関数はこんな感じ？
    - def init_fn(L614)
        - 関数の初期化。init_paramとfield_initが呼ばれている。前者はx:int=3のような__init__関数のパラメータ文字列、後者はself.x = 1のような__init__関数の中身の文字列が作られる。その後、add_fnを__init__関数で実行
    - FuncBuilder.add_fn
        - self.namesに関数名を、self.srcに関数のコードの文字列を入れる
    - *FuncBuilder.*add_fns_to_class
        - dataclassで作る、クラスの中の関数の一覧を返す関数(create_fn)を、文字列からexecで実行することで、name属性に関数を結びつける
        - add_fnでsrcに関数一覧の中身のコードが入っているのでそれを中に置いて、return self.namesで関数名を返すことで、関数create_fnを作っている
    

https://github.com/fhiyo/leetcode/pull/31

- 再帰で呼び出す関数の引数に必要な計算を、あらかじめ別変数に入れてその変数を引数に、とやらずに、関数の引数の中で計算式を入れる選択肢がある。いつも忘れがち。
    - 引数が何を示すかわかりにくくなるデメリットはあるかも？
- Step1のように、preorder順に見ていくが、rootからいちいち親を探す方法を実装
    - いちいちrootに戻って探すのは手間な感じもするが、実装としては素直なのかもしれない
    - 実行時間は結構かかる
- うーん、search_parent関数の中身ネストが深いが、代案が思いつかない

```python

class Solution:
    def search_parent(self, child, root, node_val_to_inorder_index):
        result = root
        while True:
            if node_val_to_inorder_index[child.val] == node_val_to_inorder_index[result.val]:
                raise ValueError("The node is already in tree")
            if node_val_to_inorder_index[child.val] < node_val_to_inorder_index[result.val]:
                if not result.left:
                    return result, "left"
                result = result.left
            else:
                if not result.right:
                    return result, "right"
                result = result.right

    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        assert len(preorder) == len(inorder)
        assert len(preorder) == len(set(preorder))
        assert len(inorder) == len(set(inorder))
        root = TreeNode(preorder[0])
        node_val_to_inorder_index = {val: i for i, val in enumerate(inorder)}
        for i in range(1, len(preorder)):
            node = TreeNode(preorder[i])
            parent, direction = self.search_parent(node, root, node_val_to_inorder_index)
            if direction == "left":
                parent.left = node
            else:
                parent.right = node
        return root
```

- 再帰をstackに直す
    - nullのnodeも突っ込んで後で取り出す時にチェックするより、どうせ.left, .rightで繋げる時に本当にあるかチェックしなきゃいけないので、stackにpushするかもその中に入れれば、取り出す時のチェックはいらない
    
    ```python
    
    class Solution:
        def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
            assert len(preorder) == len(inorder)
            assert len(preorder) == len(set(preorder))
            assert len(inorder) == len(set(inorder))
            root = TreeNode(preorder[0])
            stack = [(root, preorder, inorder)]
            while stack:
                parent, node_vals_preorder, node_vals_inorder = stack.pop()
                left_count = node_vals_inorder.index(parent.val)
                right_count = len(node_vals_preorder) - left_count - 1
                if left_count:
                    parent.left = TreeNode(node_vals_preorder[1])
                    stack.append(
                        (
                        parent.left,
                        node_vals_preorder[1 : left_count + 1],
                        node_vals_inorder[:left_count],
                        )
                    )
                if right_count:
                    parent.right = TreeNode(node_vals_preorder[-right_count])
                    stack.append(
                        (
                        parent.right,
                        node_vals_preorder[-right_count:],
                        node_vals_inorder[-right_count:],
                        )
                    )    
            return root
    ```
    
- 上と同じロジックの再帰

```python

class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        def build_tree_helper(parent, node_val_preorder, node_val_inorder):
            left_node_count = node_val_inorder.index(parent.val)
            right_node_count = len(node_val_preorder) - left_node_count - 1
            if left_node_count:
                parent.left = TreeNode(node_val_preorder[1])
                build_tree_helper(parent.left, node_val_preorder[1:left_node_count + 1], node_val_inorder[:left_node_count])
            if right_node_count:
                parent.right = TreeNode(node_val_preorder[-right_node_count])
                build_tree_helper(parent.right, node_val_preorder[-right_node_count:], node_val_inorder[-right_node_count:])
        root = TreeNode(preorder[0])
        build_tree_helper(root, preorder, inorder)
        return root
```

- nodeを返して、nodeを繋げるのは親がやることにした再帰
    - これが簡潔で一番好き
    - if left_node_count: の条件はつけず、一番上でif not preorder: returnをやるのは、stackのノードに空のも突っ込んで後でチェックするのに対応するのか(今更気づいた)

```python

class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        root = TreeNode(preorder[0])
        left_node_count = inorder.index(root.val)
        right_node_count = len(preorder) - left_node_count - 1
        if left_node_count:
            root.left = self.buildTree(preorder[1:left_node_count + 1], inorder[:left_node_count])
        if right_node_count:
            root.right = self.buildTree(preorder[-right_node_count:], inorder[-right_node_count:])
        return root
```

## Step4(追加のやり方)
- [この辺](https://discord.com/channels/1084280443945353267/1247673286503039020/1300957769477918791)　を参照
- inorder順に見ていく方法
- 最初わけわかんなくなった(下が訳わかんなくなってエラーが出たコード)
    - 各ループの時点で、stackにどこのnodeがどの順序で入っていれば良いのか混乱していた

```python
# 訳わかんなくなってエラーが出たコード

# inorer順に見てく書き方
class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        # 入力値のチェック 今回は省略
        stack = []
        
        # inorderで今までのnodeの中で、.rightがまだのものを繋げる関数
        def connect_right_child_nodes_so_far(node):
            child = None
            while stack:
                parent = stack[-1]
                if parent == node:
                    return child
                parent.right = child
                child = stack.pop()
        
        node_val_to_preorder_index = {val: i for i, val in enumerate(preorder)}
        prev_val = None
        dummy = TreeNode()
        root = None
        for val in inorder:
            node = TreeNode(val)
            stack.append(node)
            if prev_val is None:
                prev_val = val
                continue
            if node_val_to_preorder_index[prev_val] < node_val_to_preorder_index[val]:
                prev_val = val
                continue
            if node_val_to_preorder_index[val] == 0:
                root = node
                dummy.left = root
            node.left = connect_right_child_nodes_so_far(node)
            prev_val = val
        return connect_right_child_nodes_so_far(dummy)
```

- かなり時間がかかったが、なんとか書けた、苦労した
    - stackに入っているデータの不変条件をよく意識
    - う〜ん、スッキリするような、しないような？？？

```python
class Solution:
    dummy_root_preorder_index = -1
    dummy_root_val = 0
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        stack = []
        node_val_to_preorder_index = {val: i for i, val in enumerate(preorder)}
        dummy_root = None
        inorder.append(self.dummy_root_val)
        node_val_to_preorder_index[None] = self.dummy_root_preorder_index

        def construct_left_child_tree(node):
            child = None
            while stack:
                parent = stack.pop()
                parent.right = child
                if node_val_to_preorder_index[parent.val] == node_val_to_preorder_index[node.val] + 1:
                    return parent
                child = parent

        prev_val = None
        for val in inorder:
            node = TreeNode(val)
            if node_val_to_preorder_index[node.val] == -1:
                dummy_root = node
            if prev_val is not None and node_val_to_preorder_index[node.val] < node_val_to_preorder_index[prev_val]:
                node.left = construct_left_child_tree(node)
            stack.append(node)
            prev_val = val
        return dummy_root.left
```

- odaさんのコードを見た。construct_left_treeの中でstackの後ろを見て、ダメだったら終わりにすれば、いちいちpreorder順をinorderのforループの中で調べなくていいのか

```python

class Solution:
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        node_val_to_preorder_index = {val: i for i, val in enumerate(preorder)}
        stack = []
        inorder_plus_dummy = inorder + [None]
        node_val_to_preorder_index[None] = -inf
        
        def construct_left_tree(node):
            child = None
            while True:
                parent = stack[-1] if stack else None
                if not parent or node_val_to_preorder_index[parent.val] < node_val_to_preorder_index[node.val]:
                    return child
                parent.right = child
                stack.pop()
                child = parent
              
        for val in inorder_plus_dummy:
            node = TreeNode(val)
            node.left = construct_left_tree(node)
            stack.append(node)
        dummy = stack[-1]
        return dummy.left
```

- preorder順に見てくやつ(範囲情報も入れる)も実装してみた。
    - 最初どういうことか分かってなかったが、自分のpositionと、stackに入っている自分の先祖の中で、自分より右にある中で一番左のpositionを入れると判定できるという案か
    - 後者が、Step1の自分の実装では、結果的に一個上のnodeのpositionになってる
    - この考えの方が自然なのか？？
    - odaさんの実装とは違い、右につながるやつの親はstackに入れっぱ

```python

from dataclasses import dataclass

@dataclass
class InorderPosition:
    node: TreeNode
    left_limit: int
    right_limit: int

class Solution:
    def search_parent_of_right_child(self, node: TreeNode, current_inorder_index: int, stack: List[InorderPosition]):
        while stack:
            if current_inorder_index < stack[-1].right_limit:
                return stack[-1]
            stack.pop()
                
    def buildTree(self, preorder: List[int], inorder: List[int]) -> Optional[TreeNode]:
        dummy = TreeNode()
        stack = [InorderPosition(dummy, inf, inf)]
        node_val_to_inorder_index = {val: i for i, val in enumerate(inorder)}
        for val in preorder:
            node = TreeNode(val)
            current_inorder_index = node_val_to_inorder_index[val]
            back = stack[-1]
            if current_inorder_index < back.left_limit:
                back.node.left = node
                stack.append(InorderPosition(node, current_inorder_index, back.left_limit))
                continue
            back = self.search_parent_of_right_child(node, current_inorder_index, stack)
            back.node.right = node
            stack.append(InorderPosition(node, current_inorder_index, back.right_limit))
        return dummy.left
```
