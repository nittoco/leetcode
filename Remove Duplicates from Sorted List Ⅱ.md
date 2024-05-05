## Step1

- Iの方で参考にしたコードのように基本的にはnextで入れ替える、ただしheadだけは別で処理して進める、というコードにした
    - headだけ別になってしまうのがどうしようもないのか、アイデアは思い浮かばない
- whileやifの中に、何がNoneでないという条件を書くべきかでこんがらがった。なるべく時間がかからないように処理したい。
- 前半のheadの部分と後半のcurrentの部分で同じような処理はしているので関数化はできそう。

```python

class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        while head and head.next and head.val == head.next.val:
            head = head.next
            if not head.next or head.val != head.next.val:
                head = head.next
        current = head
        while current:
            while current.next and current.next.next and current.next.val == current.next.next.val:
                current.next = current.next.next
                if not current.next.next or current.next.val != current.next.next.val:
                    current.next = current.next.next
            current = current.next
        return head

```

## Step2

- 番兵、以前一回やったのに気づかなかった
- 以下のodaさんのやつは、かなり読むのに苦労し相当時間がかかった。連続する数の末尾の部分でどうしてうまくいくか謎だったが、日本語で１行ずつコメント書いてみたらわかった。
    - もっとロジックを把握するのを早くしたい。あと、わからなくなったら、日本語でコメントをつけてみようと思う。

```python

class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        dummy = ListNode(-1000)
        dummy.next = head
        last_unique_node = dummy
        curr = head

        while curr:
            # 今と後の数字が違うなら、lastに入　このままだと3, 3, 4, 4の2要素目みたいな時にやばい
            if curr.next and curr.val != curr.next.val:
                last_unique_node = curr
                curr = curr.next
                continue
            # 今と後の数字が同じなら、まずは違うところになるまで送る
            while curr.next and curr.val == curr.next.val:
                curr.next = curr.next.next
            # 今と違う数字の最初の位置にcurrを映す　これで、3, 3, 4, 4の2要素目みたいな時に、最初の条件に引っ掛からなくてすむ
            curr = curr.next
            # 上記の位置に暫定でlastをつなげる この後のループで今と後が違うなら、正式に繋げられる
            last_unique_node.next = curr
        return dummy.nex
```

- 以下のコードもコメントをつけて読んでみた
- 電車をつなぐ仕事の例が分かりやすかった
    - for文とかが仕事の引き継ぎというのが腑に落ちた

```python
dummy = ListNode()
dymmy.next = head
previous = dummy  # 本線のお尻
is_deleting = False
deleting_number = 0
while current:
  # 昨日の続きで外す。
  remove_car = False
  if is_deleting and deleting_number == current.val: # 前消した、かつ前と同じnumber
    remove_car = True
  elif not current.next or current.val != current.next.val: # 最後か、今と次が違う
    remove_car = False
  else:　# 今と次が同じ
    remove_car = True
  if remove_car: 
    is_deleting = True
    deleting_number = current.val
    previous.next = current.next　# 今のやつを飛ばす previousが最後にListを通過するので、これにより接続先が変更される
  previous = current # とばされるやつがpreviousになってもOK（どっちも次のに接続するようになり結果的に飛ばされる）
  current = current.next
```

```python

while current and current.next:
  if current.val != current.next.val: # 今と後で違うなら、そのまま送る
    previous = current
    current = current.next
    continue
  value_to_remove = current.val
  while current.val == value_to_remove:　# 前と一緒の限り、次に送ってスルー
    current = current.next
  previous.next = current　# とりあえず違うところの先頭につなげる
  
```

- Haskellのconcat, filter, groupも見てみた、すごい簡潔に書けそう

https://hackage.haskell.org/package/base-4.19.0.0/docs/Data-List.html#v:group

## Step3

- 結局、1つずつ見て情報を引き継ぐバージョンは自分なりには以下のようなコードに落ち着いた

```python

class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        sentinel = ListNode()
        sentinel.next = head
        unique_node = sentinel
        is_delete = False
        prev_val = -1000
        curr = head
        while curr:
            if curr.next and curr.val == curr.next.val: # 後と同じ
                is_delete = True
            elif prev_val == curr.val: # 前と同じ
                is_delete = True
            else:
                is_delete = False
            if is_delete:
                unique_node.next = curr.next # とりあえずスルー
            else:
                unique_node = curr # 今のノードをつなげる
            prev_val = curr.val
            curr = curr.next
        return sentinel.next
```

- 「値が同じ場合は、連続して仕事をする」バージョンは以下に落ち着いた

```python

def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        sentinel = ListNode()
        sentinel.next = head
        curr = head
        unique = sentinel
        while curr:
            if not curr.next or curr.val != curr.next.val:
                unique.next = curr
                unique = curr
                curr = curr.next
                continue
            while curr.next and curr.val == curr.next.val:
                unique.next = curr.next # 飛ばす
                curr = curr.next
            unique.next = curr.next # 同じ値の塊の最終のもう1個先に
            curr = curr.next
        return sentinel.next
```

- 「削除すべき場合は続ける」は以下

```python

class Solution:
    def deleteDuplicates(self, head: Optional[ListNode]) -> Optional[ListNode]:
        sentinel = ListNode()
        sentinel.next = head
        curr = head
        unique = sentinel
        value_to_remove = -10000
        while curr:
            if not curr.next or curr.val != curr.next.val:
                unique.next = curr
                unique = curr
                curr = curr.next
            elif curr.next and curr.val == curr.next.val:
                value_to_remove = curr.val
            while curr and curr.val == value_to_remove:
                curr = curr.next  
            unique.next = curr
        return sentinel.next
```

## 参考資料

主にahayashiさん、hardjuiceさんのものを参考に

- https://discord.com/channels/1084280443945353267/1195700948786491403/1196681161498447982
- https://discord.com/channels/1084280443945353267/1221030192609493053/1225144371121361016
- https://discord.com/channels/1084280443945353267/1195700948786491403/1196701558382018590
- https://discord.com/channels/1084280443945353267/1183683738635346001/1183691370683174973
- https://hackage.haskell.org/package/base-4.19.0.0/docs/Data-List.html#v:group
