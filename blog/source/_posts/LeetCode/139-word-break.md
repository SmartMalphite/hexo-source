---
title: LeetCode.139 | 单词拆分
date: 2022-05-20 00:25:49
categories: 
    - 刷题
tags: 
    - 动态规划
---

## [题目描述](https://leetcode.cn/problems/word-break/)

> 给你一个字符串 s 和一个字符串列表 wordDict 作为字典。请你判断是否可以利用字典中出现的单词拼接出 s 。
注意：不要求字典中出现的单词全部都使用，并且字典中的单词可以重复使用。

示例 ：

```
输入: s = "leetcode", wordDict = ["leet", "code"]
输出: true
解释: 返回 true 因为 "leetcode" 可以由 "leet" 和 "code" 拼接成。
```

## 解题思路
假如要判断字符串"catsanddog"是否满足条件，我们可以采取`分而治之`的思路去思考，比如将字符串切割成两部分去分别判断，当前后两部分都满足条件时则可以证明这个字符串整体也是满足条件的；

这种思路的本质是将字符串分为两部分去看，后半部分为`本次待判断`的字符串，前半部分为不包括最后一个字符串的其余部分。那么如何求前半部分是否满足条件呢? 也可以通过相同的方式在进行一次分割判断。

此时已经将此问题翻译成了一个标准的动态规划题目

## 实现思路
代码实现上可以采取两层循环的方式: `外层循环`设置变量i作为每个`待判断子串`(即s[0,i])的结尾。从1开始，循环至字符串最后一个字符，逐步判断每个子串是否满足条件，并将结果记录下来方便对下个子串进行判断时的搜索。即逐步判断'c', 'ca', 'cat', 'cats' ... 'catsanddog'是否满足条件。 

`内层循环`设置变量j在每个由外层循环切割出的子串内进行`遍历切割`，根据记忆化搜索的结果判断当前子串是否满足条件。比如待判断子串为'catsanddog'时，将其切割为['c', 'atsanddog']、['ca', 'tsanddog']等等，最终发现切割为['catsand', 'dog']时可以满足条件('catsand'子串命中记忆化搜索，'dog'命中字典)

## 代码实现

```go
func wordBreak(s string, wordDict []string) bool {
    wordDictSet := map[string]bool{}
    for _,v := range wordDict{
        wordDictSet[v] = true
    }
    dp := make([]bool, len(s)+1)
    dp[0] = true
    for i:=1;i<=len(s);i++{
        for j:=0;j<i;j++{
            if dp[j] && wordDictSet[s[j:i]]{
                dp[i] = true
            }
        }
    }
    return dp[len(s)]
}
```
