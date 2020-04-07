# Leetcode！ 日常


第 N+1 次立下决心刷题，现已加入日常任务。  

<!--more-->

{{< admonition warning >}}

2020/ 04/ 01 Update  近期比较忙，疯狂打脸ing  
{{< /admonition >}}
{{< admonition warning >}}
2020/ 04/ 02 Update 抽空看完了 Python 实现的常见数据结构，又信心满满的回来刷题了。
{{< /admonition >}}


## 面试题 01.07. 旋转矩阵

{{< admonition question >}}
给你一幅由 N × N 矩阵表示的图像，其中每个像素的大小为 4 字节。请你设计一种算法，将图像顺时针旋转 90 度。

不占用额外内存空间能否做到？
{{< /admonition >}}

```python
class Solution:
    def rotate(self, matrix: List[List[int]]) -> None:
        """
        Do not return anything, modify matrix in-place instead.
        """
        n = len(matrix)
        # 水平翻转
        for i in range(n // 2):
            for j in range(n):
                matrix[i][j], matrix[n - i - 1][j] = matrix[n - i - 1][j], matrix[i][j]
        # 主对角线翻转
        for i in range(n):
            for j in range(i):
                matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
```
  
要求不占用额外空间，首先想到的就是矩阵内元素两两互换，类似于快排。但是没想到具体怎么实现，刚好题解中有这种解法，使用了对角线旋转。  

## 121. 买卖股票的最佳时机

{{< admonition question >}}
给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。

如果你最多只允许完成一笔交易（即买入和卖出一支股票一次），设计一个算法来计算你所能获取的最大利润。

注意：你不能在买入股票前卖出股票。
{{< /admonition >}}

  
```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        if len(prices) == 0:
            return 0
        min_price = prices[0]
        profit = 0
        for cur_price in prices:
            if cur_price > min_price:
                max_profit = cur_price - min_price
                profit = max(max_profit, profit)
            else:
                min_price = cur_price
        
        return profit
```

关键是 `max()` 的使用，开始没有想到。另外没事还得看看谷歌开源项目风格指南，经常忘。  

## `Mark` 8. 字符串转换整数 (atoi)
{{< admonition question >}}
请你来实现一个 atoi 函数，使其能将字符串转换成整数。

首先，该函数会根据需要丢弃无用的开头空格字符，直到寻找到第一个非空格的字符为止。接下来的转化规则如下：
{{< /admonition >}}
{{< admonition question >}}
如果第一个非空字符为正或者负号时，则将该符号与之后面尽可能多的连续数字字符组合起来，形成一个有符号整数。
{{< /admonition >}}
{{< admonition question >}}
假如第一个非空字符是数字，则直接将其与之后连续的数字字符组合起来，形成一个整数。
{{< /admonition >}}
{{< admonition question >}}
该字符串在有效的整数部分之后也可能会存在多余的字符，那么这些字符可以被忽略，它们对函数不应该造成影响。
{{< /admonition >}}
{{< admonition question >}}
注意：假如该字符串中的第一个非空格字符不是一个有效整数字符、字符串为空或字符串仅包含空白字符时，则你的函数不需要进行转换，即无法进行有效转换。

在任何情况下，若函数不能进行有效的转换时，请返回 0 。
{{< /admonition >}} 

```python
INT_MAX = 2 ** 31 - 1
INT_MIN = -2 ** 31

class Automaton:
    def __init__(self):
        self.state = 'start'
        self.sign = 1
        self.ans = 0
        self.table = {
            'start': ['start', 'signed', 'in_number', 'end'],
            'signed': ['end', 'end', 'in_number', 'end'],
            'in_number': ['end', 'end', 'in_number', 'end'],
            'end': ['end', 'end', 'end', 'end'],
        }
        
    def get_col(self, c):
        if c.isspace():
            return 0
        if c == '+' or c == '-':
            return 1
        if c.isdigit():
            return 2
        return 3

    def get(self, c):
        self.state = self.table[self.state][self.get_col(c)]
        if self.state == 'in_number':
            self.ans = self.ans * 10 + int(c)
            self.ans = min(self.ans, INT_MAX) if self.sign == 1 else min(self.ans, -INT_MIN)
        elif self.state == 'signed':
            self.sign = 1 if c == '+' else -1

class Solution:
    def myAtoi(self, str: str) -> int:
        automaton = Automaton()
        for c in str:
            automaton.get(c)
        return automaton.sign * automaton.ans
```
  
题解中使用了 [确定有限状态机 (DFA)]^(deterministic finite automaton)，类似于 `TCP` 协议中的状态机。有点意思，先mark一下回头再仔细看看


## 289. 生命游戏

{{< admonition question >}}
给定一个包含 m × n 个格子的面板，每一个格子都可以看成是一个细胞。每个细胞都具有一个初始状态：1 即为活细胞（live），或 0 即为死细胞（dead）。每个细胞与其八个相邻位置（水平，垂直，对角线）的细胞都遵循以下四条生存定律：
{{< /admonition >}}
{{< admonition question >}}
1. 如果活细胞周围八个位置的活细胞数少于两个，则该位置活细胞死亡；
{{< /admonition >}}
{{< admonition question >}}
2. 如果活细胞周围八个位置有两个或三个活细胞，则该位置活细胞仍然存活；
{{< /admonition >}}
{{< admonition question >}}
3. 如果活细胞周围八个位置有超过三个活细胞，则该位置活细胞死亡；
{{< /admonition >}}
{{< admonition question >}}
4. 如果死细胞周围正好有三个活细胞，则该位置死细胞复活；
{{< /admonition >}}
{{< admonition question >}}
根据当前状态，写一个函数来计算面板上所有细胞的下一个（一次更新后的）状态。下一个状态是通过将上述规则同时应用于当前状态下的每个细胞所形成的，其中细胞的出生和死亡是同时发生的。
{{< /admonition >}}  

```python
class Solution:
    def gameOfLife(self, board: List[List[int]]) -> None:
        """
        Do not return anything, modify board in-place instead.
        """

        neighbors = [(1,0), (1,-1), (0,-1), (-1,-1), (-1,0), (-1,1), (0,1), (1,1)]

        rows = len(board)
        cols = len(board[0])

        copy_board = [[board[row][col] for col in range(cols)] for row in range(rows)]

        for row in range(rows):
            for col in range(cols):

                live_neighbors = 0
                for neighbor in neighbors:

                    r = (row + neighbor[0])
                    c = (col + neighbor[1])

                    if (r < rows and r >= 0) and (c < cols and c >= 0) and (copy_board[r][c] == 1):
                        live_neighbors += 1
    
                if copy_board[row][col] == 1 and (live_neighbors < 2 or live_neighbors > 3):
                    board[row][col] = 0

                if copy_board[row][col] == 0 and live_neighbors == 3:
                    board[row][col] = 1

```

注意 Python 迭代器 [] 和生成器 () 的使用

有个大佬给出了使用 CNN 卷积来解决问题的思路，真的是服气。

## 225. 用队列实现栈

{{< admonition question >}}
使用队列实现栈的下列操作：
{{< /admonition >}}
{{< admonition question >}}
push(x) -- 元素 x 入栈
{{< /admonition >}}
{{< admonition question >}}
pop() -- 移除栈顶元素
{{< /admonition >}}
{{< admonition question >}}
top() -- 获取栈顶元素
{{< /admonition >}}
{{< admonition question >}}
empty() -- 返回栈是否为空
{{< /admonition >}}

```python
class MyStack:

    def __init__(self):
        """
        Initialize your data structure here.
        """
        self.queue = []

    def push(self, x: int) -> None:
        """
        Push element x onto stack.
        """
        self.queue.append(x)
        for i in range(len(self.queue)-1):
            self.queue.append(self.queue.pop(0))        

    def pop(self) -> int:
        """
        Removes the element on top of the stack and returns that element.
        """
        if len(self.queue) > 0:
            return self.queue.pop(0)

    def top(self) -> int:
        """
        Get the top element.
        """
        if len(self.queue) > 0:
            return self.queue[0]

    def empty(self) -> bool:
        """
        Returns whether the stack is empty.
        """
        return len(self.queue) == 0

# Your MyStack object will be instantiated and called as such:
# obj = MyStack()
# obj.push(x)
# param_2 = obj.pop()
# param_3 = obj.top()
# param_4 = obj.empty()
```

用队列的先进先出实现栈的后进先出。

## 409. 最长回文串

{{< admonition question >}}
给定一个包含大写字母和小写字母的字符串，找到通过这些字母构造成的最长的回文串。

在构造过程中，请注意区分大小写。比如 "Aa" 不能当做一个回文字符串。
{{< /admonition >}}

我的答案：  
```python
class Solution:
    def longestPalindrome(self, s: str) -> int:
        ch_cnt = collections.Counter(s)
        length = 0
        flag = 0
        for ch in ch_cnt:
            if ch_cnt[ch] % 2 == 1:
                length += ch_cnt[ch] - 1
                flag = 1
            else:
                length += ch_cnt[ch]
        
        return length + 1 if flag == 1 else length
```

推荐答案：  
```python
class Solution:
    def longestPalindrome(self, s):
        ans = 0
        count = collections.Counter(s)
        for v in count.values():
            ans += v // 2 * 2
            if ans % 2 == 0 and v % 2 == 1:
                ans += 1
        return ans
```

思路一致，但是写法不优雅。  

时间复杂度 `O(N)`，其中 `N` 为字符串 `s` 的长度。我们需要遍历每个字符一次。

## 1160. 拼写单词  

{{< admonition question >}}
给你一份『词汇表』（字符串数组） words 和一张『字母表』（字符串） chars。  
假如你可以用 chars 中的『字母』（字符）拼写出 words 中的某个『单词』（字符串），那么我们就认为你掌握了这个单词。  
注意：每次拼写时，chars 中的每个字母都只能用一次。  
返回词汇表 words 中你掌握的所有单词的 长度之和。 
{{< /admonition >}}

```python
class Solution:
    def countCharacters(self, words: List[str], chars: str) -> int:
        chars_cnt = collections.Counter(chars)
        ans = 0
        for word in words:
            word_cnt = collections.Counter(word)
            for c in word_cnt:
                if chars_cnt[c] < word_cnt[c]:
                    break
            else:
                ans += len(word)
        return ans

```

时间复杂度 `O(n)`，`n` 为所有字符串的总长度，包括 `chars` 和 `words`。  

精髓是使用哈希表存储 `chars` 中每个字母的数量，没有接触过 `collection.Counter()` 这种用法 orz。  

另外注意 `for...else` 的使用。

## 面试题 01.06 字符串压缩  

{{< admonition question >}}
字符串压缩。利用字符重复出现的次数，编写一种方法，实现基本的字符串压缩功能。比如，字符串aabcccccaaa会变为a2b1c5a3。若“压缩”后的字符串没有变短，则返回原先的字符串。你可以假设字符串中只包含大小写英文字母（a至z）。
{{< /admonition >}}

```python  
class Solution:
    def compressString(self, S: str) -> str:
        if not S:
            return ""
        ch = S[0]
        ans = ''
        cnt = 0
        for c in S:
            if c == ch:
                cnt += 1
            else:
                ans += ch + str(cnt)
                ch = c
                cnt = 1
        ans += ch + str(cnt)
        return ans if len(ans) < len(S) else S
```
