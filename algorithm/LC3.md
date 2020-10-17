**LeetCode&nbsp;&nbsp;3. Longest Substring Without Repeating Characters**

## 题目描述

给定一个字符串，请你找出其中不含有重复字符的**最长子串**的长度。

```
输入: "abcabcbb"
输出: 3 
解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
```

## 解题思路

1. 使用start指针指向无重复字串起始位置
2. 使用Map记录字符的索引, 遇到存在的字符更新索引, 移动start

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        char[] chs = s.toCharArray();
        Map<Character, Integer> map = new HashMap<>();
        int start = 0, ans = 0;
        for (int i = 0; i < chs.length; i++) {
            if (map.containsKey(chs[i])) {
                // 如果当前字符的索引在start之前, 那么就不需要移动了
                start = Math.max(map.get(chs[i]) + 1, start);
            }
            map.put(chs[i], i);
            
            ans = Math.max(ans, i - start + 1);
        }
        return ans;
    }
}
```