# LeetCode-28 实现strStr()
title: 'LeetCode-28 实现strStr()'
date: 2018/6/29 16:39:20
tag: [LeetCode,双指针,字符串]
categories: 面试题
---

# 实现 strStr() 函数。

给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回  -1。

**示例 1:**

> 输入: haystack = "hello", needle = "ll" 
输出: 2

示例 2:

> 输入: haystack = "aaaaa", needle = "bba" 
输出: -1

说明:

当 needle 是空字符串时，我们应当返回什么值呢？这是一个在面试中很好的问题。

对于本题而言，当 needle 是空字符串时我们应当返回 0 。这与C语言的 strstr() 以及 Java的 indexOf() 定义相符。


# 思路
显然，要用两个引用（指针）i,j，从两个字符串头部开始判断。
相同则同时后移（i++，j++），不同则i++。
终止条件：i，j其中之一达到底端。

第一次尝试：
```Java
class Solution {
    public int strStr(String haystack, String needle) {
        if(needle == null) return 0;
        int i = 0, j = 0, n1 = haystack.length(), n2 = needle.length();
        while(i < n1 && j < n2){
            if(haystack.charAt(i) == needle.charAt(j)){
                i++;j++;
                continue;
            }
            i++;
            j = 0;
        }
        if(j == n2) return i-j;
        return -1;
    }
}
```

> 输入： "mississippi" "issip"
> 输出： -1
> 预期：4

踩坑错误：忽视了i这个指针，在与j相遇时，应进入一个内循环，终止条件是j到底或i到底，若中途有不相等的，要回到进入位置，故后修改，用一新指针head表示头部。
```Java
class Solution {
    public int strStr(String haystack, String needle) {
        if(needle == null) return 0;
        int i = 0, j = 0, n1 = haystack.length(), n2 = needle.length();
        if(n1 < n2) return -1;
        int head = 0;
        while(i < n1 && j < n2){
            if(haystack.charAt(i) == needle.charAt(j)){
                head = i;
                while(haystack.charAt(i) == needle.charAt(j)){
                    if(j == n2-1) return head;
                    if(i == n1-1) return -1;//这里没写好，处理边界防止i到底移除
                    i++;j++;                    
                    }
                i = head;
                }
            i++;
            j = 0;
        }
        if(j == n2) return head;
        return -1;
    }
}
```

# 看看别人的代码思路。
```Java
class Solution {
    public int strStr(String haystack, String needle) {
        if(needle.length()==0)return 0;
        char first = needle.charAt(0);
        int max = haystack.length()-needle.length();//差值
        for (int i = 0; i <= max; i++) {
            if (haystack.charAt(i) != first) {
                while (++i <= max && haystack.charAt(i) != first);//若i与needle第一个不同，++i直到相同
            }
            if (i <= max) {//所在位置没超过差值，有戏
                int j = i + 1;
                int end = j + needle.length() - 1;
                for (int k = 1; j < end && haystack.charAt(j) == needle.charAt(k); j++, k++);
                if (j == end) {
                    return i;
                }
            }
        }
        return -1;//长度超了，则-1
    }
}
```






狗屁无赖的答案如下：
```Java
class Solution {
    public int strStr(String haystack, String needle) {
        return haystack.indexOf(needle);
    }
}
```




