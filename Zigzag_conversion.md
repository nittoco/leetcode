## Step1
- string_indexの初期化忘れ（必要な変数の定義忘れ）
- 配列の空きをNoneで定義するか、スペースでやるか迷ってしまった
- string_indexがどれだけ進むかループによってバラバラなのと、column_indexの最後がわからない→string_indexのwhileが良い？
- charのlistをstringにどうやって変換するか
- for文と配列で、rowとcolumn、どっちが先か、迷った
- 周期の間違い　cycle=numRows-1だった

```python
def get_element(matrix, row, col):
    try:
        return matrix[row][col]
    except IndexError:
        return None

class Solution:
    def convert(self, s: str, numRows: int) -> str:
        zigzag_array = []
        zigzag_list = []
        column = 0
        column_index = 0
        string_index = 0
        cycle = numRows-1 if numRows!=1 else 1
        while string_index < len(s):
            if(column_index%cycle == 0):
                current_column = list(s[string_index:(string_index+numRows)])
                zigzag_array.append(current_column)
                string_index = string_index + numRows
            else:
                current_column = [None]*numRows
                char_index = -(column_index%cycle + 1)
                current_column[char_index] = s[string_index]
                string_index = string_index + 1
                zigzag_array.append(current_column)
            column_index = column_index + 1
        #print(zigzag_array)
        for current_column in range(numRows):
            for current_row in range(column_index):
                ordered_char = get_element(zigzag_array, current_row, current_column)
                if ordered_char:
                    zigzag_list.append(ordered_char)
        return ''.join(zigzag_list)

        

                

                

 
```

## Step2

- 二次元配列の空のやつの定義の仕方、書き方
    - チュートリアルの5.1.4を[参考にした](https://docs.python.org/3/tutorial/datastructures.html#more-on-lists)
    - ついでに「イテレータとは」とかも知ったhttps://docs.python.org/3/glossary.html#term-iterable
- whileの中の処理が素直じゃないかも
- 配列に「条件を満たす限り」appendしていく、の方がindexの管理をcolumn側でしなくて良いので楽
    - whileの中身のifとelseで処理の流れを統一できる。appendならどちらも1文字ずつで統一できる。
    - スペースについて考えなくて良い。左に詰めるだけ。
    - 最終的な出力も、順番に素直に出力するだけなので、楽
- 最終的な出力から考えて、どういう形でデータを持ったら楽かよく考えるのもいいのかも
- joinについても調べた
    - https://docs.python.org/3/library/stdtypes.html#str.join
- 最後に結合するときのfor文の中身の変数名に迷った。ayahashiさんのを参考にした。
- 行が1行の場合のコーナーケースを忘れていた
- ちなみにwhileの中身のwhileをforでやる選択肢もある

```python
class Solution:
    def convert(self, s: str, numRows: int) -> str:
        num_rows = numRows
        if(num_rows==1):
            return s
        zigzag_list = [ [] for _ in range(num_rows)]
        row_index = 0
        str_index = 0
        while str_index < len(s):
            while row_index < num_rows - 1 and str_index < len(s):
                zigzag_list[row_index].append(s[str_index])
                row_index = row_index + 1
                str_index = str_index + 1
            while row_index > 0 and str_index < len(s):
                zigzag_list[row_index].append(s[str_index])
                row_index = row_index - 1
                str_index = str_index + 1
        zigzag_string = ''
        for zigzag_row in zigzag_list:
            zigzag_string += ''.join(zigzag_row)
        return zigzag_string
```

## Step3

- num_rows - 1の-1 を忘れていて1ミス
- Step1ではいつもすごい苦労するがこちらではあまり苦労しないなあ

```python
class Solution:
    def convert(self, s: str, numRows: int) -> str:
        num_rows = numRows
        if(num_rows == 1):
            return s
        zigzag_list = [[] for _ in range(num_rows)]
        str_index = 0
        row_index = 0
        while str_index < len(s):
            while row_index < num_rows-1 and str_index < len(s):
                zigzag_list[row_index].append(s[str_index])
                str_index = str_index + 1
                row_index = row_index + 1
            while row_index > 0 and str_index < len(s):
                zigzag_list[row_index].append(s[str_index])
                str_index = str_index + 1
                row_index = row_index - 1
        zigzag_string = ''
        for zigzag_row in zigzag_list:
            zigzag_string += ''.join(zigzag_row)
        return zigzag_string
```

## 参考資料
- [Pythonチュートリアル　配列](https://docs.python.org/3/tutorial/datastructures.html#more-on-lists)
- [イテレータ　用語解説](https://docs.python.org/3/glossary.html#term-iterable)
- [joinについて](https://docs.python.org/3/library/stdtypes.html#str.join)
- [ahayashiさんの実装](https://github.com/hayashi-ay/leetcode/pull/71/files/4e3f19427ec7b3f63883aefe18bf640fee7bc7c9)
- [小田さんのdiscordコメント](https://discord.com/channels/1084280443945353267/1196472827457589338/1196472926862577745)
