## Reversed Link List

- 最初あまり頭使わずに書いた以下のコードでエラー
    - Linked Listはサイクルが出るとダメらしい
- そういえばこれツーパス

```python

class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        curr = head
        prev = {}
        while curr.next:
            prev[curr.next] = curr
            curr = curr.next
        reverse_node = curr
        tail = curr
        while prev[reverse_node]:
            reverse_node.next = prev[reverse_node]
            reverse_node = prev[reverse_node]
        return tail.next
```

- こう書いて通した

```python
class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        curr = head
        prev = None
        while curr: 
            ordered_curr_next = curr.next
            curr.next = prev
            prev = curr
            curr = ordered_curr_next
        return prev
```

## Step2

- return prevってまあ答えは合うんだけど、Linked Listを逆順にしたもの、とは伝わりづらい気がしないでもない
    - となんとなく思ってDiscordを漁ったら、[これ](https://github.com/sakupan102/arai60-practice/pull/8/commits/cc2eb808a76995f48b89d457dca0ce9578abbbb7)はreversed_listという変数名。この方がいいかも。
- スタックを使う、再帰を使う
    - こちらは全く選択肢になかった
- 上記が何も見ずに書けるか
    - 再帰
        - 「サイクルを作らない」ためには再帰の関数の前にprevのノードを見つけてつなげる必要だが、これをうまく実装する方法が思い浮かばない
        - ahayashiさんのを見て、関数の引数にしてしまうという方法を思いつく
            - なるほど
        - [head.next](http://head.next) = None とすれば一応できなくもないのか
            - しかし読みにくい気がする
        - 関数呼び出しのインライン化
            - [Python.jpなどの資料](https://www.python.jp/news/wnpython311/inline-function.html)を見る。Cのスタックを使わずにPythonの方で、gotoで一直線的に呼び出しているという理解
            - 実際に[CPython](https://github.com/python/cpython/blob/1a6e2138773b94fdae449b658a9983cd1fc0f08a/Lib/functools.py)のコードを読むのに挑戦。Pythonファイルの方で連続して呼び出しているというのはなんとなくわかるが、関数や変数が何を指しているものなのかがよくわからないので詳細な処理はわからない。
                - あと、C言語のポインタを雰囲気でしか理解していないのでよくわかっていない
            - [CPythonのドキュメント](https://devguide.python.org/internals/interpreter/)で、どういう変数がどういう役割なのかほんのちょっとわかった
        - Pythonの変数のスコープについてわかってなくて一回はまった
            - 内側の関数で変数を変更しても外側には反映されない
            - リファレンス読んでみた、globalとか、classでunbindの場合は外側が参照されるとか、知らなかった(https://docs.python.org/3/reference/executionmodel.html)
        
        ```python
        
        class Solution:
            def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
                def helper(reversed_node, current):
                    if not current:
                        return reversed_node
                    next_node = current.next 
                    current.next = reversed_node
                    return helper(current, next_node)
                return helper(None, head)
        ```
        
    - スタックを使う
        
        ```python
        
        class Solution:
            def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
                if not head:
                    return None
                node_stack = []
                current = head
                while current:
                    node_stack.append(current)
                    current = current.next
                reversed_head = node_stack.pop()
                current = reversed_head
                while node_stack:
                    reversed_next = node_stack.pop()
                    current.next = reversed_next
                    current = current.next
                current.next = None
                return reversed_head
        ```
        

## Step3

- 同じようなコードになった
- whileやforが引き継ぎだと思ってから結構意識が変わった（気がする）

```python

class Solution:
    def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
        reversed_list = None
        current = head
        while current:
            next_node = current.next
            current.next = reversed_list
            reversed_list = current
            current = next_node
        return reversed_list
```
