---
title: LeetCode-5 最长回文子串
date: 2018-07-11 15:13:51
tags: [LeetCode,字符串,动态规划]
categories: 面试题

---

# LeetCode-5 最长回文子串
给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为1000。

**示例 1：**

> 输入: "babad" > 输出: "bab" > 注意: "aba"也是一个有效答案。

示例 2：

> 输入: "cbbd"
> 
> 输出: "bb"


# 解法一：暴力破解

遍历每一个子串的方法要O（n^2），判断每一个子串是不是回文的时间复杂度是O(n)

时间复杂度： O（n^3)

# 解法二：带剪枝的，中心扩展

中心扩展就是把给定的字符串的每一个字母当做中心，向两边扩展，这样来找最长的子回文串。算法复杂度为O(N^2)。
但是要考虑两种情况：

 1. 像aba，这样长度为奇数。
 2. 想abba，这样长度为偶数。
 
```Java
    private static int low;
    private static int maxLen;
    private static String sub;

    /**
     * 剪枝的中心扩散法
     * @param s
     * @return
     */
    public static String longestPalindrome(String s) {
        if(s.length() < 2){
            return s;
        }

        for(int i = 0; i < s.length()-1; i++){
            findLongestSub(s, i, i);
            findLongestSub(s, i, i+1);
        }
        sub = s.substring(low, low+maxLen);
        return sub;
    }

    private static void findLongestSub(String s, int j, int k){
        while(j >=0 && k <= s.length()-1 && s.charAt(j)==s.charAt(k)){
            j--;
            k++;
        }
        if(maxLen < k-j-1){
            low = j+1;
            maxLen = k-j-1;
        }
        if((s.length()- 1 - k) < (maxLen / 2)){
            return;//最后部分可以剪枝
        }
    }
```

# Manacher's ALGORITHM:马拉车算法

O(n)时间求字符串的最长回文子串

记忆细节：

 1. 插入#，奇偶数组都能解决, p[]数组存每个位置最长回文长度
 2. id表示最长回文中心点，max表示最长回文半径，用于监控
 3. i from 0 to n，i若处在max中，p[i]取min(p[2*id - i], max-i)
 4.  再看看p[i]能否左右扩张
 5.  更新最长回文串，更新ip和max，保存p[]中对应maxC，maxP

[详细介绍][1]


```Java
    /**
     * 骚气的马拉车算法求解
     * @param s
     * @return
     */
    public static String longestPalindrome2(String s) {

        //先给新数组插入#
        String t = "#";
        for (int i = 0; i < s.length(); i++){
            t += s.charAt(i) + "#";
        }
        int[] p = new int[t.length()];  //定义数组p[]，代表对应数组上i位置的回文半径

        int id = 0;
        int max = id + p[id];//定义id，为最长回文中心。定义max，为最长回文右端。
        int maxC = 0; int maxP = 0;//定义maxC，maxP，为数组p中最大值和对应的中心位置

        //循环，分别求得p[i]
        for (int i = 0; i < t.length(); i++){
            //判断：i是否在max的范围内
            if (i < max){
                p[i] = Math.min(p[2*id - i], max-i);
            }
            while( i-p[i] >= 0 && i+p[i] < t.length() && t.charAt(i - p[i]) == t.charAt(i + p[i])){
                p[i]++;
            }
            //更新最长回文串，更新ip和max
            if (i + p[i] > max){
                id = i;//中心转移
                max = id + p[id]; //最右影响位置转移
            }
            //保存数组p中位置
            if (p[i] > maxP){
                maxP = p[i];
                maxC = i;
            }
        }

        //还原数组,在中心为是不是#
        if (t.charAt(maxC) == '#') { // 还原长度为偶数的回文串
            return s.substring((maxC - 2) / 2 - (maxP - 2) / 2, (maxC - 2) / 2 + 1 + (maxP - 2) / 2 + 1);
        } else {                     // 还原长度为奇数的回文串
            return s.substring((maxC - 1) / 2 - (maxP - 1) / 2, (maxC - 1) / 2 + (maxP - 1) / 2 + 1);
        }
    }
```


参考：https://www.jianshu.com/p/7c3f074b380b

  [1]: https://www.felix021.com/blog/read.php?2040