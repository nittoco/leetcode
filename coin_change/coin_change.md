### Step1

- 作れない時、infは何を表しているのかわからなくなりそうなのでNoneで実装
    - ロジックは結構複雑になる
    - num_current_coin += 1をまとめるために、continueではなく試しにelifを使ってみたが、elifがこういう使い方が自然なのかよくわからない
    - TLE。無駄なことをやっていたことに気づく。()

```python
class Solution(object):
    def count_min_needed_coin_kind_so_far_to_make_amount(self, val_current_coin, min_needed_coin_for_amount, num_coin_kind, amount):
        num_current_coin = 0
        while amount - val_current_coin * num_current_coin >= 0:
            num_prev_coin_so_far = min_needed_coin_for_amount[num_coin_kind - 1][amount - val_current_coin * num_current_coin]
            if num_prev_coin_so_far is None:
                pass
            elif min_needed_coin_for_amount[num_coin_kind][amount] is None:
                min_needed_coin_for_amount[num_coin_kind][amount] = num_prev_coin_so_far + num_current_coin
            else:
                min_needed_coin_for_amount[num_coin_kind][amount] = min(min_needed_coin_for_amount[num_coin_kind][amount], \
                 num_prev_coin_so_far + num_current_coin)
            num_current_coin += 1
        return None

    def coinChange(self, coins, target_amount):
        """
        :type coins: List[int]
        :type amount: int
        :rtype: int
        """
        # min_needed_coin_count_for_amount[i][j] := もしi種類目までだったときの、j円作る時のコイン必要枚数の最小値
        min_needed_coin_for_amount = [[None] * (target_amount + 1) for _ in range(len(coins) + 1)]
        min_needed_coin_for_amount[0][0] = 0
        for num_coin_kind in range(1, len(coins) + 1):
            for amount in range(target_amount + 1):
                self.count_min_needed_coin_kind_so_far_to_make_amount(coins[num_coin_kind - 1], min_needed_coin_for_amount, num_coin_kind, amount)
        if min_needed_coin_for_amount[-1][-1] is None:
            return -1
        return min_needed_coin_for_amount[-1][-1]
```

- このように1次元がシンプルだね
    - min_needed_coin_for_amount[amount - coin_val]は変数定義しても良かったかも
    - amount < coin_val よりcoin_val > amountが良かったかも

```python
class Solution:
    def coinChange(self, coins, target_amount):
        """
        :type coins: List[int]
        :type amount: int
        :rtype: int
        """
        min_needed_coin_for_amount = [None] * (target_amount + 1)
        min_needed_coin_for_amount[0] = 0
        for coin_val in coins:
            for amount in range(target_amount + 1):
                if amount < coin_val:
                    continue
                if min_needed_coin_for_amount[amount - coin_val] is None:
                    continue
                if min_needed_coin_for_amount[amount] is None:
                    min_needed_coin_for_amount[amount] = min_needed_coin_for_amount[amount - coin_val] + 1
                min_needed_coin_for_amount[amount] = min(min_needed_coin_for_amount[amount], min_needed_coin_for_amount[amount - coin_val] + 1)
        if min_needed_coin_for_amount[target_amount] is None:
            return -1
        return min_needed_coin_for_amount[target_amount]
```

### Step2

- Noneが作れないというの、マジックナンバーよりはマシかもだが、どこかで明示しないと辛い気がする。
    - [seal_azarashi](https://github.com/seal-azarashi/leetcode/pull/37/files)さんのStep2-1。変数名で明示しているので自分のよりはわかりやすいが、まずminをとる(このためにinfを用意)→さっきminをとった時のみDP配列に代入、は、ちょっと周りくどい気もする
- for amount in range(target_amount + 1)より、for amount in range(len(min_needed_coin_for_amount))が良いか？でも、各amountごとにtarget_amountまで計算やってるということを明示したいのでここは今のままで良いかも
- a = min(a, b)とやると1行が長くなりがちなので、if(b >a) continue a = bの方が良かったかも
- [fhiyo](https://github.com/fhiyo/leetcode/pull/41/files) さんのやつ、最初に絶対使わないcoin弾くかは確かに迷ったなあ。どうせif amount < coin_val:　continueでまとめて弾かれるが、for amount in range(target_amount + 1)ごとにチェックしないので多少は早いかも。ただこのコードみたいに入力を変更するのはあんまりな気がして、そうするとコピーしなきゃいけないからなあ。
- iwasaさん
    - i32: Max を直接使う方がわかりやすいという[コメント](https://github.com/Yoshiki-Iwasa/Arai60/pull/54/files#r1736036985).
        - 確かに本当にminのアルゴリズムが正しいかの一手間はいるかも。ただDPの値がinfの時何を表しているのかわからなくなるので良し悪し。
    - 変数を分けるのはいいかもね、dataclassとかでもいいかも
- [gotoさん](https://github.com/goto-untrapped/Arai60/pull/34/files)
    - 再帰は引数多くなってちょっと見にくいかもね
    - -1がマジックナンバーがちかもと思った。LeetCodeの設定ではあるけど、仕様は変わるかも
- dataclassでの実装
- BFSでの実装
    - そもそもDPを、各配列の要素をnodeだと思った最短距離問題だと思っていたので、自分としては割と考えは自然だった
        - これだと配るDPみたいになるのと、探索順序は違ってくる
    - dequeでやるのは同じdequeに色々なものが入って嫌なので、今の配列と次の配列で分けて実装
    - 個人的にはseenというsetを作るのが、意図が伝わりやすい気がする

```python

class Solution:
    NOT_EXIST = -1
    def coinChange(self, coins, target_amount):
        """
        :type coins: List[int]
        :type amount: int
        :rtype: int
        """
        amounts_can_make_current = [0]
        amounts_can_make_one_more_coin = []
        coin_count = 0
        seen = set()
        while True:
            if not amounts_can_make_current:
                return self.NOT_EXIST
            amounts_can_make_one_more_coin = []
            for amount in amounts_can_make_current:
                if amount == target_amount:
                    return coin_count
                if amount > target_amount:
                    continue
                for coin_val in coins:
                    next_val = amount + coin_val
                    if next_val in seen:
                        continue
                    amounts_can_make_one_more_coin.append(next_val)
                    seen.add(next_val)
            amounts_can_make_current = amounts_can_make_one_more_coin
            coin_count += 1
```

- DFS(非再帰)での実装
- stackに入れるnum_coinsとsum_valのタプルにdataclass使うか迷ったが、2個くらいならいいかという気持ちに(甘い?)
- 最初、以下のように間違えた
    - targetが10の例で、(5, 1, 1, 1, 1, 1)の方が(4, 2, 2)より先に探索されてしまう
```python
class Solution:
    NOT_EXIST = -1
    def coinChange(self, coins, target_amount):
        """
        :type coins: List[int]
        :type amount: int
        :rtype: int
        """
        stack = [(0, 0)]
        seen = set()
        sorted_coins = sorted(coins)   
        while stack:
            num_coins, sum_val = stack.pop()
            if sum_val == target_amount:
                return num_coins
            if sum_val in seen or sum_val > target_amount:
                continue
            seen.add(sum_val)
            for coin_val in sorted_coins:
                stack.append((num_coins + 1, sum_val + coin_val))
        return self.NOT_EXIST
```
- 直して書いたやつTime Limit Exceeded、効率が悪い
- 細かいが、この場合すでに見ている時にもseenに毎回Trueを記入するのは気になるかも？(良い代案はない)
 - infにすればこの問題はないが、見たかどうかは別変数で管理したい
```python
class Solution:
    NOT_EXIST = -1
    def coinChange(self, coins, target_amount):
        """
        :type coins: List[int]
        :type amount: int
        :rtype: int
        """
        stack = [(0, 0)]
        amount_to_needed_coin = [self.NOT_EXIST] * (target_amount + 1)
        seen = [False] * (target_amount + 1)
        sorted_coins = sorted(coins)   
        while stack:
            num_coins, sum_val = stack.pop()
            if sum_val > target_amount:
                continue
            if seen[sum_val] and num_coins > amount_to_needed_coin[sum_val]:
                continue
            amount_to_needed_coin[sum_val] = num_coins
            seen[sum_val] = True
            for coin_val in sorted_coins:
                stack.append((num_coins + 1, sum_val + coin_val))
        return amount_to_needed_coin[target_amount]
```
- 見たかどうかを別変数seenで管理したDP
- 以下の問題はどうしても
    - not seenも何かしらの値でDP配列は埋めなきゃなので、どうしてもinfが出てしまう
    - 上と同様、すでに見ている時にもseenに毎回Trueを記入

```python
class Solution:
    NOT_EXIST = -1
    def coinChange(self, coins, target_amount):
        """
        :type coins: List[int]
        :type amount: int
        :rtype: int
        """
        min_needed_coin = [inf] * (target_amount + 1)
        min_needed_coin[0] = 0
        seen = [False] * (target_amount + 1)
        seen[0] = True
        for amount in range(target_amount + 1):
            for coin_val in coins:
                amount_current_coin_excluded = amount - coin_val
                if amount_current_coin_excluded < 0:
                    continue
                if not seen[amount_current_coin_excluded]:
                    continue
                if min_needed_coin[amount_current_coin_excluded] + 1 > min_needed_coin[amount]:
                    continue
                min_needed_coin[amount] = min_needed_coin[amount_current_coin_excluded] + 1
                seen[amount] = True
        if not seen[target_amount]:
            return self.NOT_EXIST
        return min_needed_coin[target_amount]
```

### Step3
- infよりNoneの方が初期値としていいかなと思い結局こうした。この問題、悩みどころが多い。
```python
class Solution:
    NOT_EXIST = -1
    def coinChange(self, coins, target_amount):
        """
        :type coins: List[int]
        :type amount: int
        :rtype: int
        """  
        min_needed_coins = [None] * (target_amount + 1)
        seen = [False] * (target_amount + 1)
        min_needed_coins[0] = 0
        seen[0] = True
        for amount in range(target_amount + 1):
            for coin_val in coins:
                amount_current_coin_excluded = amount - coin_val
                if amount_current_coin_excluded < 0:
                    continue
                if not seen[amount_current_coin_excluded]:
                    continue
                if seen[amount] and min_needed_coins[amount_current_coin_excluded] + 1 > min_needed_coins[amount]:
                    continue
                min_needed_coins[amount] = min_needed_coins[amount_current_coin_excluded] + 1
                seen[amount] = True
        if not seen[target_amount]:
            return self.NOT_EXIST
        return min_needed_coins[target_amount]
```