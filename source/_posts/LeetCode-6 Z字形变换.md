---
title: LeetCode-6 Z字形变换
date: 2018-06-21 09:35:53
tags: [LeetCode,字符串]
categories: 面试题

---

# LeetCode-6 Z字形变换
难度：middle

将字符串 `PAYPALISHIRING` 以Z字形排列成给定的行数：

P&nbsp;&nbsp;&nbsp;&nbsp;A  &nbsp;&nbsp;&nbsp; H &nbsp;&nbsp;&nbsp;  N
A P&nbsp; L &nbsp;S&nbsp;I&nbsp;&nbsp; I&nbsp; G
Y  &nbsp;&nbsp;&nbsp; I &nbsp;&nbsp;&nbsp;  R
之后从左往右，逐行读取字符："PAHNAPLSIIGYIR"

实现一个将字符串进行指定行数变换的函数:

string convert(string s, int numRows);

> 示例 1:
> 
> 输入: s = "PAYPALISHIRING", numRows = 3 
输出: "PAHNAPLSIIGYIR"
> 
> 示例 2:
> 
> 输入: s = "PAYPALISHIRING", numRows = 4 
输出: "PINALSIGYAHRPI"

解释:

P  &nbsp;&nbsp;  &nbsp; I  &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; N
A  &nbsp; L&nbsp; S &nbsp; &nbsp;&nbsp;I G
Y A &nbsp;  H &nbsp;R
P   &nbsp;&nbsp; &nbsp; I


# 我的解法————考虑首尾行，中间行分前后跳跃

```Java
class Solution {
public  String convert(String s, int numRows) {
        if (numRows == 1) {
            return s;
        }
        int len = s.length();
        int i = 0;//外层大循环，表示每一行的循环
        int j = 0;
        int gap1 = 2 * numRows - 3;//前半段gap
        int gap2; //后半段gap
        int gap = 0; //真实的gap
        int flag = 1; //1表示在前半段，-1表示在后半段
        StringBuffer sb = new StringBuffer();
        while (i < numRows) {
            if (i == 0 || i == numRows - 1) {
                gap = 2 * numRows - 3;
                while (j < len) {
                    sb.append(s.charAt(j));
                    j += (gap + 1);
                }
            }else {//处于中间，要判断下在前面还是后面
                gap1 = gap;//左
                gap2 = 2 * i - 1;//右
                while (j < len) {
                    sb.append(s.charAt(j));
                    j = flag == 1 ? (j + gap1 + 1) : (j + gap2 + 1);
                    flag *= -1;
                }
            }
            gap = gap - 2 > 0 ? gap - 2 : (2 * numRows - 3);
            i++;
            j = i;
            flag = 1;
        }
        return String.valueOf(sb);
    }
}
```
# 解法一简化整理版本

对于所有整数 k，

行0中的字符位于索引k(2⋅numRows−2) 处;
行numRows−1 中的字符位于索引k(2⋅numRows−2)+numRows−1 处;
内部的 行 i中的字符位于索引k(2⋅numRows−2)+i 以及(k+1)(2⋅numRows−2)−i 处;


```Java
class Solution {
    public String convert(String s, int numRows) {

        if (numRows == 1) return s;

        StringBuilder ret = new StringBuilder();
        int n = s.length();
        int cycleLen = 2 * numRows - 2;//循环的长度，也就是gap+1

        for (int i = 0; i < numRows; i++) {
            for (int j = 0; j + i < n; j += cycleLen) {
                ret.append(s.charAt(j + i));
                if (i != 0 && i != numRows - 1 && j + cycleLen - i < n)//不是头尾，且没有超过s长度
                    ret.append(s.charAt(j + cycleLen - i));//j + cycleLen - i
            }
        }
        return ret.toString();
    }
}
```


# 解法二 ：按行排序，每行有一个数组。
思路

通过从左向右迭代字符串，我们可以轻松地确定字符位于 Z 字形图案中的哪一行。

算法

我们可以使用 min( numRows, len(s)) 个列表来表示 Z 字形图案中的**非空行**。

从左到右迭代 s，将每个字符添加到合适的行。
可以**使用当前行**和**当前方向**这两个变量对合适的行进行跟踪。

只有当我们向上移动到最上面的行或向下移动到最下面的行时，**当前方向**才会发生改变。
```Java
class Solution {
    public String convert(String s, int numRows) {

        if (numRows == 1) return s;

        List<StringBuilder> rows = new ArrayList<>();
        for (int i = 0; i < Math.min(numRows, s.length()); i++){
            rows.add(new StringBuilder());//确定list中有若干个个sb
            }

        int curRow = 0;
        boolean goingDown = false;//方向

        for (char c : s.toCharArray()) {
            rows.get(curRow).append(c);//获取正确的sb，添加c
            if (curRow == 0 || curRow == numRows - 1) goingDown = !goingDown;
            curRow += goingDown ? 1 : -1;//是向上还是向下
        }

        StringBuilder ret = new StringBuilder();
        for (StringBuilder row : rows) ret.append(row);
        return ret.toString();
    }
}
```