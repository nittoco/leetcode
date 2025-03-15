## Step1

- stringをintegerにする必要に気づいてなかった
- 「最初の」空白とマイナスを除く、どう処理する？
- integerという変数名のマイナス反転の打ち間違い　注意！
- +-45みたいな入力が入るとは思わなかった。

```python

class Solution:
    def myAtoi(self, s: str) -> int:
        minus = False
        integer_from_string = 0
        begin = True
        for character in s:
            if character in  (' ') and begin:
                continue
            if character in ('-', '+') and begin:
                if character == '-':
                    minus = True
                begin = False
                continue
            begin = False
            try:
                character_to_int = int(character)
            except ValueError:
                if minus:
                    integer_from_string = -(integer_from_string)
                return integer_from_string
            integer_from_string *= 10
            integer_from_string += character_to_int
            if minus and integer_from_string > 2**31:
                return -2**31
            if integer_from_string > 2**31 - 1 and not minus:
                return 2**31-1
        if minus:
            integer_from_string = -(integer_from_string)
        return integer_from_string
```

## Step2

- [これ](https://github.com/hayashi-ay/leetcode/pull/69/commits/6e33c001dfbd4b5e91d1c4ba798004bc515ccd10)(ahayashiさん)でも[これ](https://github.com/SuperHotDogCat/coding-interview/pull/5/commits/aa2084ad8de307b5d5054ec1a07bbb74f155b888)(SuperHotDogCatさん）でもsign = -1として、後でかけてる。この方がスマート。
- ordという関数を知る(https://docs.python.org/3/library/functions.html#ord)
    - これでエラー判定とか読みにくくことをしなくて良さそう
- みんなwhileで書いてindexを進めてるけど、forとcontinueを使って書くと、違和感があるのだろうか
    - beginというのを管理しなくていいのは良さそう？
    - forが仕事の引き継ぎだという感覚を持つと、確かにいちいち回すのは不自然？
- abs_valueの命名は確かにいい(https://github.com/cheeseNA/leetcode/pull/5/files)
- どこまで関数化するか迷う。数値判定くらいはしてもいいかも
- MIN_VALUEとかも定数にしていいかも
- andの仕様として、current_index < len(s)を最初に書かなきゃいけない、そっちだけまず評価されるので（index out of rangeが出て、[ちょうど最近読んだこと](https://docs.python.org/ja/3/reference/expressions.html#index-84)を思い出した

```python

class Solution:
    def myAtoi(self, s: str) -> int:
        MAX_VALUE = 2**31 - 1
        MIN_VALUE = -2**31
        def _is_digit(char):
            if ord('0') <= ord(char) <= ord('9'):
                return True
            return False
        current_index = 0
        abs_value = 0
        sign = 1
        # whileかfor, continueか迷ったが、「文字列のこの部分まで見ました」と区切りをつけれる感覚でwhileが自然な気もしてきた
        while current_index < len(s) and s[current_index] == ' ':
            current_index += 1
        if current_index == len(s):
            return 0
        if s[current_index] in ('-', '+'):
            if s[current_index] == '-':
                sign = -1
            current_index += 1
        if current_index == len(s):
            return 0
        while current_index < len(s) and _is_digit(s[current_index]):
            abs_value *= 10
            abs_value += int(s[current_index])
            if abs_value * sign >= MAX_VALUE:
                return MAX_VALUE
            if abs_value * sign <= MIN_VALUE:
                return MIN_VALUE
            current_index += 1
        return sign * abs_value

        
        
                
                
                
            

```

## Step3

- スペースの時の、current_index < len(s)のつけ忘れ
    - whileで走査する時はつけると覚えてもいいかも
- MIN_VALUE, MAX_VALUEが超えたかどうかの、signのつけ忘れ

```python

class Solution:
    def myAtoi(self, s: str) -> int:
        abs_value = 0
        sign = 1
        MAX_VALUE = 2**31 - 1
        MIN_VALUE = -2**31
        current_index = 0
        def _is_integer(char):
            if ord('0') <= ord(char) <= ord('9'):
                return True
            return False
        while current_index < len(s) and s[current_index] == ' ':
            current_index += 1
        if current_index == len(s):
            return 0
        if s[current_index] in ('+', '-'):
            if(s[current_index] == '-'):
                sign = -1
            current_index += 1
        while current_index < len(s) and _is_integer(s[current_index]):
            abs_value *= 10
            abs_value += int(s[current_index])
            current_index += 1
            if sign * abs_value > MAX_VALUE:
                return MAX_VALUE
            if sign * abs_value < MIN_VALUE:
                return MIN_VALUE
        return sign * abs_value
```

## 参考資料

- [ahayashiさん](https://github.com/hayashi-ay/leetcode/pull/69/commits/6e33c001dfbd4b5e91d1c4ba798004bc515ccd10)
- [ordの仕様](https://docs.python.org/3/library/functions.html#ord)
- [superHotDogさん](https://github.com/SuperHotDogCat/coding-interview/pull/5/commits/aa2084ad8de307b5d5054ec1a07bbb74f155b888)
- [cheeseNAさん](https://github.com/cheeseNA/leetcode/pull/5/files)
- [andの仕様](https://docs.python.org/ja/3/reference/expressions.html#index-84)

### Step4

- なるべく関数化
- パースするとき、indexを進める(入力が変更される)想定になっているが、まあしょうがないかなあ
- is_number_to_make_next_overflowはちょっと設計が悪いかも？(一つの関数で色々やりすぎ?)

```python
class Solution:
    MAX_INT = 2**31 - 1
    MIN_INT = -2**31
    valid_digits = {str(i) for i in range(10)}

    def skip_whitespaces(self, s, index):
        while index < len(s) and s[index] == " ":
            index += 1
        return index

    def search_sign(self, s, index):
        sign = 1
        if index < len(s) and s[index] in ("+", "-"):
            if s[index] == "-":
                sign = -1
            index += 1
        return sign, index

    def is_number_to_make_next_overflow(self, current_num, next_digit, sign):
        if sign == 1:
            max_abs = abs(self.MAX_INT)
            result = self.MAX_INT
        else:
            max_abs = abs(self.MIN_INT)
            result = self.MIN_INT
        if current_num < (max_abs - next_digit)//10:
            return False, None
        if current_num == (max_abs - next_digit)//10 and next_digit <= max_abs % 10:
            return False, None
        return True, result
    
    def myAtoi(self, s: str) -> int:
        index = 0
        index = self.skip_whitespaces(s, index)
        sign, index = self.search_sign(s, index)
        number_start_index = index
        absolute_num = 0
        for i in range(number_start_index, len(s)): # whileかちょい迷う
            if s[i] not in self.valid_digits:
                return sign * absolute_num
            next_digit = int(s[i])
            is_overflow, result_if_overflow = self.is_number_to_make_next_overflow(absolute_num, next_digit, sign)
            if is_overflow:
                return result_if_overflow
            absolute_num *= 10
            absolute_num += next_digit
        return sign * absolute_num
```

- 他の人のもみてみる
    - https://github.com/Exzrgs/LeetCode/pull/4/files
        - ‘0’ ≤ s[i] ≤ ‘9’ともかける
            - 有効なstringの仕様が変わらないなら、クラス変数にせずにこれでいいかも
            - そうそう変わることはないか？いやあでも、急に2進数のみを受け取りますとかにはなるかも
        - 空白が終わった後にすぐにreturn 0する選択肢もある
            - が、それなら+-のcheckが終わった後もreturn 0しないと整合性はない？
    - https://github.com/shining-ai/leetcode/pull/59/files
        - digit = ord(s[index]) - ord("0")という書き方もある
            - 一応ordの[ドキュメント](https://docs.python.org/3/library/functions.html#ord)を読む
            - chrというodの逆操作をするがあるらしい
        - 確かに、最後のnumberに変換するところも関数化しちゃってもいいかも
            - 再利用できそうだし
        - ループを一つにまとめたバージョンは、読みにくく感じた
    - https://github.com/fhiyo/leetcode/pull/57/files
        - Step1の最後のC++のコードが読みやすかった。
        - C++だと、parseする関数で、indexを参照渡しすれば変更しやすいの良いね
            - Pythonだとnonlocalや、mutableなオブジェクトを渡すのはちょっとわかりにくい気がして、あとは一緒にreturnするしかない
        - MINの絶対値がMAXより大きいので、overflowの可能性(Pythonは関係ないが)
            - 一応それを考慮して実装してみるか
        - 移植性考えるなら確かに、*= 10の10も変数として定義してもいいかも
            - てゆうかvalid_digitsよりこっちをクラス変数にすればよかった
            - まあそもそもそこまでする必要があるかというのもある
        - lstrip、こんな便利なメソッドがあるのね(コピーコストはかかるけど)
            
            (https://docs.python.org/ja/3/library/stdtypes.html#str.lstrip)
            
        - isdigit()とかもあるのね
            - タイ数字でもTrueになるらしいが、流石にそこまでは考えなくていい気もする
- MIN_VALUEのオーバーフローを厳格に処理した場合。思ったより大変だった

```python
class Solution:
    MAX_INT = 2**31 - 1
    MIN_INT = -2**31
    def detect_sign(self, s, index):
        sign = 1
        if index < len(s) and s[index] in ("+", "-"):
            if s[index] == "-":
                sign = -1
            index += 1
        return sign, index

    def is_overflow_too_big(self, abs_value, next_digit, sign):
        if sign != 1:
            return False
        if self.MAX_INT//10 < abs_value:
            return True
        if self.MAX_INT//10 == abs_value and self.MAX_INT % 10 < next_digit:
            return True
        return False

    def is_overflow_too_small(self, abs_value, next_digit, sign):
        if sign != -1:
            return False
        value = -abs_value
        min_last_digit = 10 - self.MIN_INT % 10
        min_except_last = self.MIN_INT//10 + 1
        if value < min_except_last:
            return True
        if value == min_except_last and min_last_digit < next_digit:
            return True
        return False

    def myAtoi(self, s: str) -> int:
        space_skipped = s.lstrip()
        sign, index = self.detect_sign(space_skipped, 0)
        abs_value = 0
        for i in range(index, len(space_skipped)):
            if not space_skipped[i].isdigit():
                return sign * abs_value
            digit = int(space_skipped[i])
            if self.is_overflow_too_big(abs_value, digit, sign):
                return self.MAX_INT
            if self.is_overflow_too_small(abs_value, digit, sign):
                return self.MIN_INT
            abs_value *= 10
            abs_value += digit
        return sign * abs_value
```
