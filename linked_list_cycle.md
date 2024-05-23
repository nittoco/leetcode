### Step1

- まず思い浮かんだのはsetを使う解法
- setでNodeクラスが等しいかの判定ができるかは自信がないが、よくわからないが通った

```python

class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        visited_node = set()
        current = head
        while current:
            if current in visited_node:
                return True
            visited_node.add(current)
            current = current.next
        return False
```

## Step2

- 2つ動かしてって、ぶつかったらサイクルがあると検出、という方法があるらしい、それを実装してみた。
    - これくらいの規模なら選択肢を持って、自力で気に入る処理を選んで書けるようになってきた
        - fast, slowという変数名については書く前に見てしまった。自力で思いつく自信は微妙。変数名については、まだいいものを自力で選べる能力はまだ身に付いてないと思う。
    - フロイトのウサギとカメという有名なものらしい、知らなかった　Atcoder界隈で似たようなことを議論しているのを見たことがあるかも？（うろ覚え）

```python

class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        slow = head
        fast = head
        while fast and fast.next:
            fast = fast.next.next
            slow = slow.next
            if slow == fast:
                return True
        return False
```

- https://discord.com/channels/1084280443945353267/1192728121644945439/1194203372115464272　青いランプの説明のたとえ
    - 上のdiscordのodaさんのコードの、下から2つ目と一番下でちょうど迷っていて、一番下に最終的にした
- fastをhead.nextにして例外処理をしていて、その後やはり最初は両方headにするか…と言っている方が多かった
    - 逆にfastをhead.nextにする選択肢が自分になかった
- https://github.com/cheeseNA/leetcode/pull/6/files
    - なぜsetでNodeの判定ができるかの参考になった
    - CPythonはまだ読める気配がない
- https://discord.com/channels/1084280443945353267/1195700948786491403/1195727801626665000 代入の議論、自分も以前意見をいただいたことがある
- [空間計算量](https://discord.com/channels/1084280443945353267/1200089668901937312/1205459874708717628　)　概念自体を何となくでしか知らず
    - 参考になった　https://discord.com/channels/1084280443945353267/1237649827240742942/1239801311642517555

## Step3

- Step2の方のコードでやってみた。

```python
python
class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        fast = head
        slow = head
        while fast and fast.next:
            fast = fast.next.next
            slow = slow.next
            if slow == fast:
                return True
        return False
```
