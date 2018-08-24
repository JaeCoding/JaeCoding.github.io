---
title: shuffle()打乱数组的实现
date: 2018-08-04 11:23:56
tags: [数据结构]
categories: 原理实现

---


# shuffle()打乱数组的实现

在做快排时，为了保证快排nlogn，会打乱数组。
然后想着怎么实现，用随机数需要一点技巧。

注意：Collection.shuffle()只能打乱集合，没法打乱数组

思路：
在数组长度n内生成一个随机数r，复制arr1的r到arr2上。
arr1的最后一个数组替换到r上，n--。重复进行

```Java
import java.util.Random;

/**
 * @author: 彭文杰
 * @create: 2018-08-05 23:45
 **/
public class MyShuffle {

    public static int[] myShuffle(int[] arr) {
        int[] arr2 = new int[arr.length];
        Random rand = new Random();
        int lengthNow = arr.length;
        int arrIndex = rand.nextInt(lengthNow);
        int arr2Index = 0;
        while (lengthNow > 0) {
            arr2[arr2Index++] = arr[arrIndex];
            arr[arrIndex] = arr[lengthNow - 1];
            lengthNow--;
            if(lengthNow == 0) break;
            arrIndex = rand.nextInt(lengthNow);
        }
        arr = arr2;
        return arr;
    }

    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 4, 5, 6};
        arr = myShuffle(arr);
        for (int i = 0; i < arr.length; i++) {
            System.out.println(arr[i]);
        }
    }
}
```

## 小坑：
之前采用的void 传参arr进去修改，arr = arr2 。后来输出的时候，发现出现重复，问题在于传的参是对象的拷贝。
在其中能修改，但是**赋值却只是赋值给了形参。**

可以用全局变量解决（static）
