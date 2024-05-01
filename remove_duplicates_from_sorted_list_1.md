## Step1

- ListNodeの扱いに慣れてきた
- 初めの時の処理をどうするか迷った。
    - beginという変数はあんまり使いたくない（状態を持つので）
    - 暫定で、prev_valを-10000とか入力制約にない値にして、prev_val == head.valにはならないようにしたが、変な入力が来たらやばいかもしれない

```python
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        sentinel = ListNode(0)
        prev_node = sentinel
        prev_val = -10000
        while head:
            if prev_val == head.val:
                head = head.next
                continue
            prev_node.next = ListNode(head.val)
            prev_node = prev_node.next
            prev_val = head.val
            head = head.next 
        return sentinel.next

```

## Step2

- 他の方のコードを見る限り、prevとcurrentの比較なら、最初にダミーを用意してやる必要がなさそう(https://github.com/TORUS0818/leetcode/pull/5/files)
- よく考えたら、headって頭という意味なので、それをwhile文で後ろに動かしていくのは変かもしれない。ということで訂正。
    - currentがないのにprevが出てきたり、prevとheadを両方動かす必要があったり、上のコードはごちゃごちゃしている
    - などと思ってたらちょうど同じようなこと書いてあるコメントを見つけた。コメントの予測が少しできて嬉しい。(https://discord.com/channels/1084280443945353267/1228896007279083653/1231823840355815475)
- このコードすっきりしてる(https://discord.com/channels/1084280443945353267/1195700948786491403/1196399353116499970)
- 上記のコードを見た後、しばらくして書いたコード。最後を考慮してないのでWrong Answerだが、どう直すか迷った

```python

class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        current = head
        while current.next:
            while current.next.next and current.val == current.next.val:
                current.next = current.next.next
            current = current.next
        return head
```

- よく考えたら、current.nextがNoneでも、current = current.nextという操作は有効だった。やっぱりまだLinekd Listに慣れない

```python
class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        current = head
        while current:
            while current.next and current.val == current.next.val:
                current.next = current.next.next
            current = current.next
        return head
```

## Step3

- 同じコードを再現できた

```python

class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        current = head
        while current:
            while current.next and current.val == current.next.val:
                current.next = current.next.next
            current = current.next
        return head
```
