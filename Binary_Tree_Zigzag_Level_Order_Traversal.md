### Step1

```python

from collections import deque

class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        nodes_and_depthes = deque([(root, 0)])
        zigzag_ordered = []
        prev_depth = None
        from_left = False
        while nodes_and_depthes:
            node, depth = nodes_and_depthes.popleft()
            if not node:
                continue
            if prev_depth != depth:
                if prev_depth and not from_left:
                    same_depth_nodes.reverse()
                same_depth_nodes = []
                from_left = not from_left
                zigzag_ordered.append(same_depth_nodes)
            same_depth_nodes.append(node.val)
            nodes_and_depthes.append((node.left, depth + 1))
            nodes_and_depthes.append((node.right, depth + 1))
            prev_depth = depth
        if prev_depth and not from_left:
            same_depth_nodes.reverse()
        return zigzag_ordered
```

### Step2

https://discord.com/channels/1084280443945353267/1201211204547383386/1219179255615717399 

- Zigzagじゃない方のStep2の最後で、current_levelとnext_levelを2つ持たず、要素数で管理するやつを自分も書いたが、確かに同じ変数で2種類のものを管理するのでややこしいかも
    - 一般に、同じ変数に複数の役割を持たすのはあまり好ましくないかもと思った

https://github.com/fhiyo/leetcode/pull/29

- left_to_rightという変数名でも良い
- 突っ込む順番を逆にするより、フラグを立てて後からreverse()の方が遅くなるものの意図が明確で良い

https://github.com/TORUS0818/leetcode/pull/29

- next_levelを持つ実装
    - 引き継ぎはcurrent_depth_nodesとfrom_leftで、current_depth_nodes_valとnext_depth_nodesの初期化は次の人に任せた
    - 後で気づいたが、next_depth_nodesをwhileの外で持つ必要はない

```python
class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []
        current_depth_nodes = [root]
        from_left = True
        next_depth_nodes = []
        zigzag_ordered = []
        while current_depth_nodes:
            current_depth_nodes_val = []
            next_depth_nodes = []
            for node in current_depth_nodes:
                current_depth_nodes_val.append(node.val)
                if node.left:
                    next_depth_nodes.append(node.left)
                if node.right:
                    next_depth_nodes.append(node.right)
            if not from_left:
                current_depth_nodes_val.reverse()
            zigzag_ordered.append(current_depth_nodes_val)
            from_left = not from_left
            current_depth_nodes = next_depth_nodes
        return zigzag_ordered
```

- DFSでの実装
    - 反転を探索ついでにやるのはかなりややこしくなりそう(可読性も微妙になりそう)なのでやめといた

```python

class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []
        zigzag_ordered = []
        nodes_stack = [(root, 0)]
        while nodes_stack:
            node, depth = nodes_stack.pop()
            while depth >= len(zigzag_ordered):
                zigzag_ordered.append([])
            zigzag_ordered[depth].append(node.val)
            if node.right:
                nodes_stack.append((node.right, depth + 1))
            if node.left:
                nodes_stack.append((node.left, depth + 1))
        for i in range(len(zigzag_ordered)):
            if i % 2:
                zigzag_ordered[i].reverse()
        return zigzag_ordered
```

## Step3

```python

class Solution:
    def zigzagLevelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:
        if not root:
            return []
        result = []
        current_depth_nodes = [root]
        left_to_right = True
        while current_depth_nodes:
            next_depth_nodes = []
            values_of_current_depth = []
            for node in current_depth_nodes:
                values_of_current_depth.append(node.val)
                if node.left:
                    next_depth_nodes.append(node.left)
                if node.right:
                    next_depth_nodes.append(node.right)
            if not left_to_right:
                values_of_current_depth.reverse()
            result.append(values_of_current_depth)
            current_depth_nodes = next_depth_nodes
            left_to_right = not left_to_right
        return result
```
