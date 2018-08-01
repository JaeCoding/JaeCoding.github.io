---
title: LeetCode-15 三数之和
date: 2018-07-19 09:24:25
tags: [LeetCode,字符串]
categories: 面试题

---

# LeetCode-6 三数之和
难度：middle

给定一个包含 n 个整数的数组 `nums`，判断 `nums` 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？找出所有满足条件且不重复的三元组。

**注意：**答案中不可以包含重复的三元组。

> 例如, 给定数组 nums = [-1, 0, 1, 2, -1, -4]，
> 
> 满足要求的三元组集合为： [   [-1, 0, 1],   [-1, -1, 2] ]

前面做过一道TwoSum的题，思路是对于每一个数a，在HashMap中查找是否有n-a这个数，有则找到，没有则把a添加进Map，a为key，索引为value。

这道题，一开始思路也是想这样，但是存在两个问题：1.数有重复，所以不能以数为索引。2.可能会有-1，-1，2这样的情况，无法解决。

这一题大思路肯定是找通过a，查找两数之和-a，可以肯定的是，三个数中肯定有正有负。定下一个数后，就可以通过双指针来变量。

`双指针解法往往对有序数组格外生效`，所以我们将数组排序。

一些剪纸优化：假如第一个数是4，我们要寻找的twoSum是-4，第二个确定的数是-1，那么我们在遍历到正数的时候就break，

然后我们还要加上重复就跳过的处理，处理方法是从第二个数开始，如果和前面的数字相等，就跳过（-1，-1，2如何解决呢？），因为我们不想把相同的数字fix两次。对于遍历到的数，用0减去这个fix的数得到一个target，然后只需要再之后找到两个数之和等于target即可。

用两个指针分别指向fix数字之后 开始的数组**首尾两个数**，如果两个数**和正好为target**，则将这两个数和fix的数一起存入结果中。
如果**和小于target**，说明需要一个更大的数，所以左指针右移。
如果**和大于target**，说明需要一个更小的数，所以右指针左移。


然后就是跳过重复数字的步骤了，两个指针都需要检测重复数字

```Java
package leetcode;


import java.util.*;

public class Leetcode_15 {

    public static List<List<Integer>> threeSum(int[] nums) {

        if(nums == null || nums.length < 3) {
            return new ArrayList<>();
        }

        Arrays.sort(nums);
        int i = 0;
        int fix = nums[0], target = - fix;
        List<List<Integer>> list = new ArrayList<>();
        while (i < nums.length - 2) {
            if(nums[i] > 0) break; //右边都是比nums[i]大的数，也就是都大于0 不可能找到了
            if(i > 0 && nums[i] == nums[i-1]) { //第二个数开始，遇到一个重复的数，不想重复找两次，跳过
                i++;
                continue;
            }
            target = 0 - nums[i];
            int p1 = i + 1, p2 = nums.length - 1;
            while (p1 < p2) {
                if (nums[p1] + nums[p2] == target ) {
                    if(p2 != nums.length -1 && nums[p1] == nums[p1 -1] && nums[p2] == nums[p2+1]){ //避免-2 0 0 2 2 重复的问题
                        ++p1; --p2;
                        continue;
                    }
                    List listThree = new ArrayList();
                    listThree.add(nums[i]);
                    listThree.add(nums[p1]);
                    listThree.add(nums[p2]);
                    list.add(listThree);
                    ++p1; --p2;
                } else if (nums[p1] + nums[p2] < target) {
                    p1++;
                } else {
                    p2--;
                }
            }
            i++;
           }
        return list;

    }




    public static void main(String[] args) {
        int[] a = {-1,0,1,2,-1,-4};

        for (List list1 : threeSum(a))
            for (Object i : list1) {
                System.out.println(""+ i);
            }

    }


}

```