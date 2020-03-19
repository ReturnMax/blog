# Leetcode！ 日常


第 N+1 次立下决心刷题，现已加入日常任务。  

<!--more-->

## 225. 用队列实现栈

{{< admonition question >}}
使用队列实现栈的下列操作：

push(x) -- 元素 x 入栈
pop() -- 移除栈顶元素
top() -- 获取栈顶元素
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
