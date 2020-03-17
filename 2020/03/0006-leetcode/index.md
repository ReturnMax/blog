# 日常！ Leetcode


<!--more-->
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

