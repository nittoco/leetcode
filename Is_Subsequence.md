## Step1

- 一発目のコード。間違い。
- 普通の処理の時のt_indexの進め忘れ。ただ、t_indexを進めると今度はインデックスの数字がオーバーするので調整が必要になり、ちょっと時間を食った。

```python
python
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        t_index = 0
        for s_index in range(len(s)):
            while t[t_index] != s[s_index]:
                t_index += 1
                if t_index >= len(t):
                    return False
        return True

```

- 上の調整を終えた後のコード、t_indexがlenを超えたらFalse、の判定を2箇所に分けたくなかったので、ifのインデントなど工夫した

```python
class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        t_index = 0
        for s_index in range(len(s)):
            while t_index < len(t) and t[t_index] != s[s_index]:
                t_index += 1
            if t_index >= len(t): 
                return False 
            t_index += 1 
        return True

```

## Step2

- まずこれ(https://discord.com/channels/1084280443945353267/1225849404037009609/1243290893671465080) が目に付く。すごい選択肢の量。
    - 関数型はよく知らないが、再帰でループを書くのね。最終的な条件がシンプルになるので、綺麗な感じはします。
    - for文にするかwhile文にするかは考えていたが、以下の選択肢はあんまり考えてなかった
        
        
        - while Trueにする
            - 最初は空文字列の処理を忘れ一敗（下のは直した後）
            - 関数型に近いと感じる。これ結構好きかも
        
        ```python
        python
        class Solution:
            def isSubsequence(self, s: str, t: str) -> bool:
                if len(s) == 0:
                    return True
                if len(t) == 0:
                    return False
                si = 0
                ti = 0
                while True:
                    if s[si] == t[ti]:
                        si += 1
                    ti += 1
                    if si == len(s):
                        return True
                    if ti == len(t):
                        return False
        ```
        
        - 上のやつ、odaさんの見たらうまく条件変えて0の判定せずに済んでる（自分で気づかなかった） 一般に、indexを参照する直前に判定した方が簡潔になりやすいのかな？
        
        ```python
        
        class Solution:
            def isSubsequence(self, s: str, t: str) -> bool:
                si = 0
                ti = 0
                while True:
                    if si == len(s):
                        return True
                    if ti == len(t):
                        return False
                    if s[si] == t[ti]:
                        si += 1
                    ti += 1
        ```
        
        - 1万外側のループをtにする、今度は下のように書いて1敗
            - tの最後の文字==sの最後の文字 の処理が走る場合に部分文字列があるのに検出できず（すぐ気づいた）
            - 今回は残念ながら空文字のTrue返す処理を分けるしかなさそう。綺麗じゃないけど。
            
            ```python
            class Solution:
                def isSubsequence(self, s: str, t: str) -> bool:
                    si = 0
                    for ti in range(len(t)):
                        if si == len(s):
                            return True
                        if s[si] == t[ti]:
                            si += 1
                    return False
            
            ```
            
            - 訂正後のコード
            - 自分は、「sの文字を1個ずつ追っていってt側にあるかな？と順に確かめる」という気持ちなので、あんまりtを外側に出す気持ちではないかも
            
            ```python
            python
            class Solution:
                def isSubsequence(self, s: str, t: str) -> bool:
                    if len(s) == 0:
                        return True
                    si = 0
                    ti = 0
                    for ti in range(len(t)):
                        if s[si] == t[ti]:
                            si += 1
                        if si == len(s):
                            return True
                    return False
            ```
            
        
        - while elseを使う（なんとなく聞いたことあるくらいで、よく把握していないので調べた）　これは全然慣れない。結局コードを参照しながら書いて時間がかかった。慣れといた方がいいのか？
            - breakした時の処理(今回でいうsi+=1 ti+=1の処理)が長い時には手段の1つではあるが、他の手段もあるし…(https://discord.com/channels/1084280443945353267/1221030192609493053/1225674901445283860)
        
        ```python
        
        class Solution:
            def isSubsequence(self, s: str, t: str) -> bool:
                si = 0
                ti = 0
                while si < len(s):
                    while ti < len(t):
                        if s[si] == t[ti]:
                            break
                        ti += 1
                    else:
                        return False
                    si += 1
                    ti += 1
                return True
        ```
        
        - 正規表現を使う
            - 何回か計測したが、かなり遅いみたい
            - わかりやすくはある
        
        ```python
        
        import re
        
        class Solution:
            def isSubsequence(self, s: str, t: str) -> bool:
                regrex = []
                for sc in s:
                    regrex.append(f'.*{sc}')
                regrex = ''.join(regrex)
                return re.match(regrex, t)
        ```
        
    - これくらいなら、変数名はsi, tiでいいね
    - sを外にあるコードもDiscordにあるodaさんのやつ(下のやつ、([https://discord.com/channels/1084280443945353267/1225849404037009609/1243290893671465080](https://discord.com/channels/1084280443945353267/1225849404037009609/1243304635998011615))の1番目のコード)の方が簡潔にかける。これもでも結局、tの方を必ず1進める、sを進めるかは場合によって、という処理
    - for文でsをイテレートするという気持ちになってしまうとこの選択肢は取れないので、難しい
    
    ```python
    
    class Solution:
        def isSubsequence(self, s: str, t: str) -> bool:
            si = 0
            ti = 0
            while si < len(s):
                if ti >= len(t):
                    return False
                if s[si] == t[ti]:
                    si += 1
                ti += 1
            return True
    ```
    
- この辺も一通り見た
    - return s[i]==len(s) みたいにする選択肢もあると気づいた
    - https://github.com/hayashi-ay/leetcode/pull/64/files
    - https://github.com/shining-ai/leetcode/pull/57/files

## Step3

- これで書いた

```python

class Solution:
    def isSubsequence(self, s: str, t: str) -> bool:
        si = 0
        ti = 0
        while si < len(s):
            if ti == len(t):
                return False
            if s[si] == t[ti]:
                si += 1
            ti += 1
        return True
```
