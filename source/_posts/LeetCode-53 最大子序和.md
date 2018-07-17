# LeetCode-53 最大子序和
title: LeetCode-53 最大子序和
date: 2018-07-03 16:10:01
tags: [LeetCode,分治法,动态规划]
categories: 面试题

---


给定一个整数数组 `nums`，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

示例:

> 输入: [-2,1,-3,4,-1,2,1,-5,4], 
输出: 6 
解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。

进阶:如果你已经实现复杂度为 O(n)的解法，尝试使用更为精妙的分治法求解。

# 解决方法（一）：暴力破解
笨蛋的JAE第一想法当然是暴力求解。从第一个元素开始进行遍历，时间复杂度是n+(n-1)+......+1 = (n^2+n)/2  。但是这样写，肯定会。

面试官：好的，我知道了，回家等消息去吧。
JAE： QAQ

# 解决方法（二）:  分治法

分治法当然是分治递归的思想。一般是二分。

将数组分为两部分，它的最大子序列要么在左半边，要么在右半边，要么在中间。

左边和右边的最大值，递归调用此方法即可求出，边界条件是当数组只剩下一个元素。
而求中间位置的最大值较麻烦，可用处于中间位置的数，先后向左向右累加，并保存下最大值，即可获得中部的最大子序列。

最后我们只要比较，这三部分那部分的最大子序列的和最大即可。
复杂度：因为是二分，且包含一次循环。故为 `O(nlogn)`



代码如下：
```Java
class Solution {

    public int maxSubArray(int[] nums) {
        return maxSubArray(nums, 0, nums.length - 1);
    }


    private static int maxSubArray(int[] nums, int left, int right) {

        if (left >= right) {
            return nums[left];
        }

        int mid = (left + right) / 2;
        int lmax = maxSubArray(nums, left, mid - 1);
        int rmax = maxSubArray(nums, mid +1, right);
        int mmax = nums[mid];
        int temp = mmax;

        for (int i = mid - 1; i >= left; i--) {
            temp += nums[i];
            mmax = Math.max(temp, mmax);
        }

        /**
         * {-2,1,-3,4,-1,2,1,-5,400}  temp=3,是从4，-1开始的，并不是从-1开始，所以让temp = mmax而不是nums[mid]
         */
        temp = mmax;
        for (int i = mid + 1; i <= right; i++) {
            temp += nums[i];
            mmax = Math.max(temp, mmax);
        }

        int max = Math.max(lmax, Math.max(rmax, mmax));
        return max;
    }
}
```

# 解决方法（三）:  kadane算法，动态规划

kadane算法利用了数学归纳法和动态规划的思想。随意给你一个现成的数组，比如说−2, 1, −3, 4, −1, 2, 1, −5, 4，如果我们能从第一个数开始，随着数组的扩充，可以轻易的求出任意一个数组的最大子列和，先求1，再求2，最后根据i求到i+1.
就像走10阶楼梯，能走一步，能走两步，有多少种走法。你可能不知道，但是1阶你肯定知道，2阶你也肯定知道，3阶就是一阶走法加二阶走法。推广到10阶。

## 思路一：
 maxSum 必然是以`nums[i]`结尾的某段构成的，也就是说maxSum的candidate必然是以nums[i]结果的。如果遍历每个candidate，然后进行比较，那么就能找到最大的maxSum了。

假设把`nums[i]之前`的连续段叫做`sum`。可以很容易想到:

 1. 如果`sum>=0`，就可以和nums[i]拼接在一起构成新的sum。因为不管nums[i]多大，`加上一个正数(sum)`总会更大，这样形成一个新的candidate。
 2. 反之，如果`sum<0`，就没必要和nums[i]拼接在一起了。因为不管nums[i]多小，`加上一个负数(sum)`总会更小。此时由于题目要求数组连续，所以没法保留原sum，所以只能让sum等于`从nums[i]重新开始`的新的一段数了，这一段数字形成新的candidate。
 3. 如果每次得到新的candidate都和全局的maxSum进行比较，那么必然能找到最大的max sum subarray.

总结：用两个数。sum为单纯的累加和，逐数添加。若sum>0，则让其加上num[i],若sum<0,则舍弃，让新的sum等于num[i]。
第二个数maxSum用于保存最大，当sum > maxSum时maxSum = sum
在循环过程中，用maxSum记录历史最大的值。从nums[0]到nums[n-1]一步一步地进行。

```Java
//复杂度为O(n)
public static int maxSum(int[] nums) {
        int thisSum = 0;
        int maxSum = nums[0];

        for (int i = 0; i < nums.length; i++) {
            if (thisSum > 0) {
                thisSum += nums[i];
            }else {
                thisSum = nums[i];
            }
            if (thisSum > maxSum) {
                maxSum = thisSum;
            }
        }
        return maxSum;
    }
```

## 思路二：动态规划

遍历array，对于，我们判断（此位置之前的maxSum + nums[i]） 和 nums[i]比大小，如果nums[i]>（sum + nums[i]） 大的话，那么说明不需要再继续加了，直接让这个数字作为新的nums[i]，开始继续，因为nums[i]已经比之前的sum都大了。

反过来，如果 （之前的sum + 这个数字）大于 （这个数字）就继续加下去。

>   利用动态规划做题。 只遍历数组一遍，当从头到尾部遍历数组A， 遇到一个数有两种选择 
> （1）加入之前subArray
> （2）自己另起一个subArray

设状态S[i], 表示以A[i]结尾的最大连续子序列和，状态转移方程如下:

> S[i] = max(S[i-1] + A[i],A[i])

从状态转移方程上S[i]只与S[i-1]有关，与其他都无关，因此可以用一个变量来记住前一个的最大连续数组和就可以了。这样就可以节省空间了。
```Java
//时间复杂度：O(n)     空间复杂度：O(1)

    public static int maxSubArray_2(int[] nums) {
        int sum = 0; //或者初始化为  sum = INT_MIN 也OK。
        int maxSum = nums[0];
        //动态规划
        for (int i = 0; i < nums.length; i++) {
        
        //S[i] = max(S[i-1] + A[i],A[i])
        //等号左边sum是最新的i，等号右边的sum是i-1的
        
            sum = Math.max(sum + nums[i], nums[i]); //当前最大值
            maxSum = Math.max(sum, maxSum);//用于保存总体最大值
        }
        return maxSum;
    }
```

