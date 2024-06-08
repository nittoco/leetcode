## Step1

- 最初の間違えたコード　問題点は以下
    - //は〇〇「以下」の最大の整数を返すので、nが負の奇数の時欲しい値より1小さい値が帰ってくる
    - そもそも、n == -1の時の処理を実装していないので、負の時無限ループしてしまう
    - self.myPowを2回呼んでるので、binary treeみたいになって(いい言葉が思い浮かばない)関数がnの線形オーダーだけ呼ばれてしまう

```python

class Solution:
    def myPow(self, x: float, n: int) -> float:
        if n == 0:
            return 1.0
        if n == 1:
            return x
        if n%2 == 0:
            return self.myPow(x, n//2)*self.myPow(x, n//2)
        return self.myPow(x, n//2)*self.myPow(x, n//2)*x
```

- 以下のように直した

```python

class Solution:
    def myPow(self, x: float, n: int) -> float:
        if n == 0:
            return 1.0
        if n == 1:
            return x
        if n < 0:
            return 1/self.myPow(x, -n)
        half_pow = self.myPow(x, n//2)
        if n%2 == 1:
            return half_pow*half_pow*x
        return half_pow*half_pow
```

## Step2

- https://github.com/Exzrgs/LeetCode/pull/28/files
- https://github.com/hayashi-ay/leetcode/pull/41/files
- のあたりまずは見た
- 1/self.myPow(x, -n)は読解負荷が高いという意見もあるらしい
    - in-placeでxとnを置き換えてみますか
- 再帰を使わずwhileループを使った実装もある
- whileの実装、まず何も見ず書いてみた、変数名の選択むずい！
    - 書いた後、lowest_bitは微妙な気がしてくる、lowestがその時々で変わるのでcurrent_bitとかの方が良い？
        - そもそもif n&1だけでいいかもね
    - hayashiさんへのnodchipさんのコメントを見て、result_powよりpoweredが良さそうと思った
    - n%2とn&1で迷ったが、n&1の方がbitを見ているということを明示的に言っている感覚があり、わかりやすいと思ってこうした
    - 「bit_pow」は「power_two_pow」とかでもいいかも
    - 代入し直さなくても、n+=1みたいな書き方、n >> = 1というのもあるのね
    - resultはfloatにするため1.0を初期値の方がいいかも

```python
# whileもあると言われ何も見ずに書いたやつ
class Solution:
    def myPow(self, x: float, n: int) -> float:
        if n < 0:
            x = 1/x
            n = -n
        bit_pow = x
        result_pow = 1
        while n:
            lowest_bit = n & 1
            if lowest_bit:
                result_pow *= bit_pow
            bit_pow *= bit_pow
            n = n >> 1
        return result_pow
```

```python
# 上記の修正後
class Solution:
    def myPow(self, x: float, n: int) -> float:
        if n < 0:
            n = -n
            x = 1.0/x
        powered = 1.0
        while n:
            if n & 1:
                powered *= x
            x *= x
            n >>= 1
        return powered

```

- 
- powのdocument(https://docs.python.org/ja/3/library/functions.html#pow) みてみた
    - モジュラ逆数や複素数までサポートしてるらしい
- [powのCPythonの実装](https://github.com/python/cpython/blob/d5ba4fc9bc9b2d9eff2a90893e8d500e0c367237/Objects/longobject.c#L4849)みてみた(以下読解メモ、間違ってるかも)
    - a**bをcで割ったあまり　zが多分最終格納する文字
    - cが空白でbが負→float出力用の別処置
    - cが0→エラー
    - cが負→別処理
    - cが1→0
    - cが空白でなく、bが負→aのcをmodとするinvmodを計算してbの符号を反転
    - aが負→cであまりをとって負でなくする
    - aがcより大きい→一定のケースでmodをとる(modは重いらしい)
    - MULT(x, y, z)を定義　これはx*yをcで割ったあまりをzに格納する
    - ob_digit配列は、2^30進法で下の桁から順に整数を格納([参考URL](https://zenn.dev/yukinarit/articles/afb263bf68fff2))
    - bが3以下の場合→愚直に計算
    - bが大きくない場合(2^60以下?)→上の自分が書いた処理を、下からでなく上のbitかたみる感じで累乗を実装
    - bが大きい場合→上のbitから5桁ずつにまとめて実装？（ここいまいち読めなかったので自信なし）
- 上からbit検査する方法で実装してみる これはこれで意外と好きかも
    - n == 0の別実装が必要な点で、一回ミスをしてしまった

```python

class Solution:
    def myPow(self, x: float, n: int) -> float:
        if n < 0:
            n = -n
            x = 1.0/x
        if n == 0:
            return 1.0
        bit = 1 << (n.bit_length() - 1)
        powered = 1.0
        while bit:
            powered *= powered
            if n & bit:
                powered *= x
            bit >>= 1
        return powered
```

- https://discord.com/channels/1084280443945353267/1196472827457589338/1242419019558948904 で、「負の数の余り」は見たほうがいいとあるので見てみる
    - [ドキュメント](https://docs.python.org/ja/3/reference/expressions.html#binary-arithmetic-operations)の感じ、a%bでbが負なら結果も負、知らなかった
- https://github.com/hayashi-ay/leetcode/pull/41/files
    - 偶奇の分岐はif elseでもいいね
    - よく考えたら、self.myPow(x*x, n//2)でいいのか。再帰の引数が増えるとだめだと思っちゃったが、最終の判定は第二引数なので、そっちさえ減ればOKでした。

```python
class Solution:
    def myPow(self, x: float, n: int) -> float:
        if n < 0:
            n = -n
            x = 1/x
        if n == 0:
            return 1.0
        if n%2:
            return x*self.myPow(x*x, n//2)
        else:
            return self.myPow(x*x, n//2)

```

- https://github.com/SuperHotDogCat/coding-interview/pull/15/files
    - C++よくわからない、、、
- https://github.com/shining-ai/leetcode/pull/45/files
    - これもみた、やっぱbitで回すならn%2よりn&1の方が好きかも

## Step3

- CPythonのやつで実装

```python

class Solution:
    def myPow(self, x: float, n: int) -> float:
        if n < 0:
            x = 1.0/x
            n = -n
        if n == 0:
            return 1.0
        bit = 1 << (n.bit_length() - 1)
        powered = 1.0
        while bit:
            powered *= powered
            if n & bit:
                powered *= x
            bit >>= 1
        return powered
```
