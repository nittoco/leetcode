## Step1

- 最初に思いついた解法
- depthを中でも参照するために、returnでdepth, max_depthの両方を返してしまったが、そもそもclassなのでself.で再帰関数を定義すればよかった
- 最初はreturn depth_search(depth, root, max_depth)[1]だったが見にくい気がして直す
    - なぜか実行速度も45ms→33msに

```python

class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        def depth_search(depth, current_node, max_depth):
            depth += 1
            if not current_node:
                return depth, max_depth
            max_depth = max(depth, max_depth)
            depth, max_depth = depth_search(depth, current_node.left, max_depth)
            depth -= 1
            depth, max_depth = depth_search(depth, current_node.right, max_depth)
            depth -= 1
            return depth, max_depth
        
        depth = 0
        max_depth = 0
        depth, max_depth = depth_search(depth, root, max_depth)
        return max_depth
```

## Step2

- [ahayashiさん](https://github.com/hayashi-ay/leetcode/pull/22/files)のをみた。よく考えたら、再帰ならdepthは持たなくても良い

```python
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0
        max_depth = max(self.maxDepth(root.left), self.maxDepth(root.right)) + 1
        return max_depth
```

- 上のは簡潔だけど、left_depthとright_depthで一旦分けといた方がいいのか？迷う
    - Step3では分けることに
- BFS順に見ていくやつも実装してみた
    - 1回目（失敗作）が上、直したのが下
    
    ```python
    python
    class Solution:
        def maxDepth(self, root: Optional[TreeNode]) -> int:
            if not root:
                return 0
            depth = 1
            current_node = [root]
            while current_node:
                depth += 1
                next_node = []
                if current_node.left:
                    next_node.append(current_node.left)
                if current_node.right:
                    next_node.append(curent_node.right)
                current_node = next_node
            return depth
    ```
    

```python

class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0
        current_nodes = [root]
        depth = 0
        while current_nodes:
            depth += 1
            next_nodes = []
            for node in current_nodes:
                if node.left:
                    next_nodes.append(node.left) 
                if node.right:
                    next_nodes.append(node.right) 
            current_nodes = next_nodes
        return depth
```

- DFS順のやつも実装、StackでDFSを実装するのは初めてなので練習
    - 割とすぐできた、うっかりappend(node.right, depth+1)とやってしまい一瞬あれ、となってたけど
    - [これ](https://github.com/shining-ai/leetcode/pull/21/files)を見る限り、node_depthという変数名よりstack_node_depthとstackであることを明示した方がいいのかな
        - 迷う

```python
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0
        depth = 1
        max_depth = 1
        node_depth = [(root, depth)]
        while node_depth:
            node, depth = node_depth.pop()
            max_depth = max(max_depth, depth)
            if node.right:
                node_depth.append((node.right, depth+1))
                max_depth = max(max_depth, depth)
            if node.left:
                node_depth.append((node.left, depth+1))
                max_depth = max(max_depth, depth)
        return max_depth
```

## Step3

- BFS（これが一番詰まる）

```python
def maxDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0
        depth = 0
        current_nodes = [root]
        while current_nodes:
            depth += 1
            next_nodes = []
            for node in current_nodes:
                if node.left:
                    next_nodes.append(node.left)
                if node.right:
                    next_nodes.append(node.right)
                current_nodes = next_nodes
        return depth
```

- DFS

よく考えたら、maxを取るのはpop()する時だけでOK

```python

class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0
        node = root
        depth = 1
        stack_node_depth = [(node, depth)]
        max_depth = 1
        while stack_node_depth:
            node, depth = stack_node_depth.pop()
            max_depth = max(max_depth, depth)
            if node.right:
                stack_node_depth.append((node.right, depth+1))
            if node.left:
                stack_node_depth.append((node.left, depth+1))
        return max_depth
```

- 再帰（これは簡単）
    - なぜかmax_depth = max(self.maxDepth(root.left), self.maxDepth(root.right)) + 1と一気に書くよりだいぶ遅い(一気に書くのは27ms、下のは40ms)

```python

class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if not root:
            return 0
        left_depth = self.maxDepth(root.left)
        right_depth = self.maxDepth(root.right)
        return max(left_depth, right_depth) + 1
```
