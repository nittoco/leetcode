## Step1

- 全体に論理が複雑になってしまった。
- pop()の使い方を調べた

```python
class Solution:
    def isValid(self, s: str) -> bool:
        parent_stack = []
        for bracket in s:
            if bracket == '(' or bracket=='{' or bracket=='[':
                parent_stack.append(bracket)
            elif bracket == ')':
                if parent_stack[-1] == '(':
                    parent_stack.pop()
                else:
                    return False
            elif bracket == '}':
                if parent_stack[-1] == '{':
                    parent_stack.pop()
                else:
                    return False
            elif bracket == ']':
                if parent_stack[-1] == '[':
                    parent_stack.pop()
                else:
                    return False
        if len(parent_stack) != 0:
            return False
        return True
        

                    

```

## Step2

- [nodchipさんのコメント](https://github.com/rossy0213/leetcode/pull/7)で、左カッコの判定は入力データの条件があるので要らない
- [このコード](https://github.com/rm3as/code_interview/pull/4/commits/890c44853a54448c19c9c8801b6867c191fdd9f1)で、if文のinの存在を知る setで使うという以外を知らなかったので、公式Docを確認

https://docs.python.org/3/reference/expressions.html#membership-test-operations

- 対応関係があるロジックの場合、辞書が良い（それも上のコードで知る）
- for文の中では、elseを使うより、continueを使う方が認知負荷が低い事も多い
- ラストの0かどうかの判定は、returnの中に条件を含めれば簡潔に書ける
- [このコメント](https://github.com/cheeseNA/leetcode/pull/10#pullrequestreview-1957466328)でnot stackで判定できるとわかる

```python

class Solution:
    def isValid(self, s: str) -> bool:
        bracket_stack = []
        for character in s:
            open_bracket = ['(', '[', '{']
            if character in open_bracket:
                bracket_stack.append(character)
                continue
            pair = {'(':')', '{':'}', '[':']'}
            if not bracket_stack:
                return False
            if pair[bracket_stack[-1]] == character:
                bracket_stack.pop()
            else:
                return False
        return not bracket_stack

        

                    

```

## Step3

- 良く考えたらopen_bracketいらない。全部pairで管理できる
- if character in ~ を、if s in ~ と書いてしまった

```python

class Solution:
    def isValid(self, s: str) -> bool:
        bracket_stack = []
        pair = {'(':')', '{':'}', '[':']'}
        for character in s:
            if character in pair:
                bracket_stack.append(character)
                continue
            if not bracket_stack:
                return False
            if character != pair[bracket_stack[-1]]:
                return False
            bracket_stack.pop()
        return not bracket_stack
```

## 参考文献

- [rossyさんのコード](https://github.com/rossy0213/leetcode/pull/7)
- [rm3as](https://github.com/rm3as/code_interview/pull/4/commits/890c44853a54448c19c9c8801b6867c191fdd9f1)さんのコード
- [inの仕様](https://docs.python.org/3/reference/expressions.html#membership-test-operations)
- [cheeseNAさんへのコメント](https://github.com/cheeseNA/leetcode/pull/10#pullrequestreview-1957466328)
