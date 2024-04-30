## Step1
- LinkedListを初めてみたので実装方法がよくわからず。editorialの実装を見てから理解し、自力で再現
- for文で書く実装しかあまり経験がなく、while文が思いつきにくかった
- if l1 else 0の表現は、eritorialの実装を先に見てしまったので思いついた。NoneはFalseで評価されることを知る。見てなかったら細かく場合わけしていただろう。
- 途中中断したが、それを抜いてもアクセプトまでに要した時間は1時間くらいかかってしまった。

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def addTwoNumbers(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        res = ListNode(0)
        carry = 0
        tmp = res
        while l1 != None or l2 != None or carry != 0:
            l1Val = l1.val if l1 else 0
            l2Val = l2.val if l2 else 0
            tmpVal = l1Val + l2Val + carry
            tmp.next = ListNode(tmpVal%10)
            tmp = tmp.next
            l1 = l1.next if l1 else None
            l2 = l2.next if l2 else None
            carry = tmpVal//10
        return res.next
```

## Step2
- t0d4さんの実装を見てdivmod関数を知る。ただ今回は、コードの読者に関数を知らない人がいるリスクと、コードが簡潔になる方を天秤にかけて選ばなかった
- tayzarnwさんへのnodchipさんのコメントを見て、resと言う変数名は微妙だと思う。
- YukiMichishitaさんの実装を見て、sentinelと言う変数名が良さそうと思う。
- コードを整える過程で、pythonのオブジェクトと名前空間の定義が曖昧だったことに気づく。教えていただきありがとうございます。
- while文の中身で各桁で計算しているのはtmp.nextではなく今の値(tmp)ということにして、前回(prev)にくっつける、という実装にしたほうがわかりやすくなると感じたのでそうする
- 再帰があることをmike0121さんへのコメントを見て学ぶ。
```python
class Solution:
    def addTwoNumbers(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        sentinel = ListNode(0)
        carry = 0
        prev = sentinel
        while l1 != None or l2 != None or carry != 0:
            l1Val = l1.val if l1 else 0
            l2Val = l2.val if l2 else 0
            tmpVal = l1Val + l2Val + carry
            tmp = ListNode(tmpVal%10)
            prev.next = tmp
            prev = tmp
            l1 = l1.next if l1 else None
            l2 = l2.next if l2 else None
            carry = tmpVal//10
        return sentinel.next
```

## Step3

- 自力でもう一度書くと、if l1 else 0と書くのをうっかり忘れていた
- while文の中身をandにしてしまっていた
- l1!=Noneではなくてl1.val!=0にしてしまった(l1がNoneの時にはそもそも取れないので、l1.valの値が不定になりコードが永遠に続く）
- l1, l2を次に送り忘れた

```python
class Solution:
    def addTwoNumbers(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        sentinental = ListNode(0)
        prev = sentinental
        carry = 0
        while(l1!=None or l2!=None or carry!=0):
            l1Val = l1.val if l1 else 0
            l2Val = l2.val if l2 else 0
            digit_sum = l1Val + l2Val + carry
            tmp = ListNode(digit_sum%10)
            carry = digit_sum//10
            prev.next = tmp
            prev = tmp
            l1 = l1.next if l1 else None
            l2 = l2.next if l2 else None
        return sentinental.next
```






## 参考にした資料
[Python divmod関数](https://docs.python.org/ja/3/library/functions.html#divmod) <br>
[t0d4さんの実装](https://github.com/t0d4/leetcode/blob/main/arai60/2-add-two-numbers/solve1.py) <br>
[tayzarnwさんへのnodchipさんのコメント](https://github.com/tayzarnw/LeetCode/pull/6/files/1691259aebf5cc9d2e1a04650992f84b6fe476bd#diff-97f26f151cc8b48dcfbfd106cebae3916d4b16a4081bde5799e7b05de5068959) <br>
[YukiMichishitaさんの実装](https://github.com/YukiMichishita/LeetCode/blob/main/2_add_two_numbers/step2.py) <br>
[hardjuiceさんの実装](https://discord.com/channels/1084280443945353267/1195700948786491403/1197114717001502822)
[mike0121さんへのコメント](https://discord.com/channels/1084280443945353267/1196472827457589338/1197166381146329208)

## Step4
- 久しぶりに1から書くとやや時間はかかった
- 変数名、whileの条件のレビューを反映
  ```python
  class Solution:
    def addTwoNumbers(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        sentinel = ListNode(0)
        prev_node = sentinel
        carry = 0
        while l1 or l2 or carry:
            l1_val = l1.val if l1 else 0
            l2_val = l2.val if l2 else 0
            current_sum = l1_val + l2_val + carry
            current_digit = current_sum % 10
            prev_node.next = ListNode(current_digit)
            carry = current_sum//10
            l1 = l1.next if l1 else None
            l2 = l2.next if l2 else None
            prev_node = prev_node.next 
        return sentinel.next
  ```
