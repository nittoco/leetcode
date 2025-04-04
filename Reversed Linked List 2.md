### Step1

- 1 passで解いてみた。
- ループの最初にnext_nodeをおかないといけないのがうーん(最初気づかずバグった)

```python
class Solution:
    def reverseBetween(self, head: Optional[ListNode], left: int, right: int) -> Optional[ListNode]:
        dummy_head = ListNode()
        dummy_head.next = head
        reversed_node = None
        reversing_node = dummy_head
        reversing_index = 0
        while reversing_node:
            next_node = reversing_node.next
            if left < reversing_index <= right:
                reversing_node.next = reversed_node
            if reversing_index == left - 1:
                reversing_node.next = None
                prev_leftmost_in_range_node = reversing_node
            if reversing_index == right:
                prev_leftmost_in_range_node.next = reversing_node
            if reversing_index == left:
                reversing_node.next = None
                leftmost_in_range_node = reversing_node
            if reversing_index == right + 1:
                leftmost_in_range_node.next = reversing_node
            reversed_node = reversing_node
            reversing_node = next_node
            reversing_index += 1
        return dummy_head.next
```

### Step2

- https://discord.com/channels/1084280443945353267/1196498607977799853/1354841302826619021
    - torusさんの。なるほど、1 passじゃないとそう解くのか
    - pre_leftやleftのnodeを見つける処理も関数したい
    - 同じアルゴリズムで、個人的に気にいるように書き直してみた。
    - find_boundary_nodes_from_linked_list関数のwhileループの中のindex += 1を行うタイミングは迷う
        - この辺のタイミング、まだよく分からないなあ

```python
from dataclasses import dataclass

@dataclass
class BoundaryNodes:
    pre_left_node: ListNode
    left_node: ListNode
    right_node: ListNode
    next_right_node: ListNode

class Solution:
    def find_boundary_nodes_from_linked_list(self, head: ListNode, left: int, right: int) -> ListNode:
        node = head
        pre_node = None
        index = 0
        while node:
            index += 1
            if index == left:
                left_node = node
                pre_left_node = pre_node
            if index == right:
                right_node = node
                next_right_node = node.next
            pre_node = node
            node = node.next
        return BoundaryNodes(pre_left_node = pre_left_node, left_node = left_node, right_node = right_node, next_right_node = next_right_node)
            
    def reverse_all_linked_nodes(self, head):
        node = head
        reversed_head = None
        while node:
            next_node = node.next
            node.next = reversed_head
            reversed_head = node
            node = next_node
        return reversed_head

    def reverseBetween(self, head: Optional[ListNode], left: int, right: int) -> Optional[ListNode]:
        boundary_nodes = self.find_boundary_nodes_from_linked_list(head, left, right)
        if boundary_nodes.pre_left_node:
            boundary_nodes.pre_left_node.next = None
        boundary_nodes.right_node.next = None
        self.reverse_all_linked_nodes(boundary_nodes.left_node)
        if boundary_nodes.pre_left_node:
            boundary_nodes.pre_left_node.next = boundary_nodes.right_node
        boundary_nodes.left_node.next = boundary_nodes.next_right_node
        if not boundary_nodes.pre_left_node:
            return boundary_nodes.right_node
        return head

```

- https://discord.com/channels/1084280443945353267/1355246975309844550/1355246980200399062
    - odaさんの。for文でいいの確かに
    - ループを分けた方が見やすいね

```python
class Solution:
    def reverse_linked_nodes_partically(self, head, n):
        if not head:
            return None, None
        reversed_node = head
        reversing_node = head.next
        next_to_reverse = head.next
        for _ in range(n - 1):
            next_to_reverse = reversing_node.next
            reversing_node.next = reversed_node
            reversed_node = reversing_node
            reversing_node = next_to_reverse   
        return reversed_node, next_to_reverse

    def reverseBetween(self, head: Optional[ListNode], left: int, right: int) -> Optional[ListNode]:
        dummy_head = ListNode()
        dummy_head.next = head
        node = dummy_head
        for _ in range(left - 1):
            node = node.next
        prev_leftmost_reversed = node
        leftmost_reversed = node.next
        prev_leftmost_reversed.next = None
        rightmost_reversed, next_rightmost_reversed= self.reverse_linked_nodes_partically(leftmost_reversed, right - left + 1)
        prev_leftmost_reversed.next = rightmost_reversed
        leftmost_reversed.next = next_rightmost_reversed
        return dummy_head.next

```

- 上のコードと、[odaさんの3番目のコード](https://discord.com/channels/1084280443945353267/1355246975309844550/1355247225394958407)との比較
    - reverse_segmentの特別な場合の処理。
        - 自分の場合は、nが0の場合ループがrange(-1)になって結果的に変更はなし、だが、わかりにくいだろうか。ちょっと行儀が悪そう。
        - また、自分はnが0の場合は、引数のnodeと次のnodeを返しているが、odaさんの場合はNoneと引数のnodeを返している。自分の場合、n = 0と1で出力が一緒になってしまうのでわかりにくいかも？
        - あと、nがデカすぎた時の処理も、自分の場合だと弱かったかも。(nextがありません、ってエラーが出ると思うけど、やや不親切な気がする。)
        - nが1の場合の処理。自分は何も起こらないが、odaさんの場合、引数のnodeの次のnodeがNoneになってしまう。今回の場合if left == rightで弾いているから良いけど、汎用的な関数にした場合どうだろう。自然かなあ。
    - leftがデカすぎた場合の処理
        - エラーを返さず、元のまま返すのはなるほど。Pythonのリストのスライス操作とかの処理を考えても、確かに
- https://discord.com/channels/1084280443945353267/1196498607977799853/1354851603630522671
    - やっぱループ分けた方が良いなあ
    - current, next, prevなどの多用は、操作上の問題で、あまり意図が見えないので好みではない(と言いつつ自分もちょこちょこ使っているが)
- odaさんの2番目のコードで書いてみた。区間の左端と、その1個まえのnodeがどこにつながっているかというのだけ変わっているが、実質そんな変わらない気がする
    - まあ左端の直前のnodeが、左端のnodeからどんどん奪っていく、という見方をすればちょっと気持ちは違うか
    - odaさんのコード見たらinsert_afterだけreturn NoneのNoneがないけど、なんか意図があるのかな

```python

class Solution:
    def skip_nodes(self, node, n):
        for _ in range(n):
            if not node:
                return None
            node = node.next
        return node

    def remove_after(self, node):
        if not node or not node.next:
            return None
        removed = node.next
        node.next = removed.next
        removed.next = None
        return removed

    def insert_after(self, node, inserted):
        if not node or not inserted:
            return None
        inserted.next = node.next
        node.next = inserted

    def reverseBetween(self, head: Optional[ListNode], left: int, right: int) -> Optional[ListNode]:
        dummy_head = ListNode()
        dummy_head.next = head
        before_reversed = self.skip_nodes(dummy_head, left - 1)
        reversed_head = before_reversed.next
        for i in range(right - left):
            removed = self.remove_after(reversed_head)
            self.insert_after(before_reversed, removed)
        return dummy_head.next
```
