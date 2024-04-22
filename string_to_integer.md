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
