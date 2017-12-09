---
layout:     post
title:      "LeetCode 04 中位数问题"
subtitle:   "限定复杂度求解两个数组的中位数"
date:       2017-12-9 17:00:00
author:     "Huper"
tags:
    算法
---

此提是LeetCode里比较著名的一道题，同时也是比较难的一道题。难是难在题目给出了一个比较小的时间复杂度要求，需要仔细钻研一下这个问题才能达到要求。当然，事实上我发现你用比较low的方法其实也可以AC，先看题：

>There are two sorted arrays **nums1** and **nums2** of size m and n respectively. Find the median of the two sorted arrays. The overall run time complexity should be (O(log (m+n)).

一起就是说给你两个排好序的数组，大小分别为m和n，让你在\\(O(log (m+n))\\)的时间内求出这两组数据的中位数，中位数就是一组数据排序后最中间的一个或者两个数据的平均值。

第一眼看到这个题直接想到了无脑合并，然后直接返回中位数，虽然还是过了，但是这样复杂度很明显是\\(O(m+n)\\)。

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int [] merged = merge(nums1, nums2);
        return check(merged);
    }
    
    int [] merge(int[] nums1, int[] nums2){
        int [] temp = new int[nums1.length + nums2.length];
        int cur1 = 0;
        int cur2 = 0;
        int cur = 0;
        while(cur1 < nums1.length && cur2 < nums2.length){
            if(nums1[cur1] < nums2[cur2]){
                temp[cur++] = nums1[cur1++];
            }
            else{
                temp[cur++] = nums2[cur2++];
            }
        }
        
        if(cur1 == nums1.length){
            while(cur2 < nums2.length){
                temp[cur++] = nums2[cur2++];
            }
        }
        
        if(cur2 == nums2.length){
            while(cur1 < nums1.length){
                temp[cur++] = nums1[cur1++];
            }
        }
        
        return temp;
    }
    
    double check(int[] in){
        if(in.length % 2 == 0){
            return ((double)(in[in.length / 2] + in[in.length / 2 - 1]))/2;
        }
        else
            return in[in.length / 2];
    }
    
}
```

仔细观察题目，要求复杂度降到对数级，一般会直接想到二分或者树这样的解决方式，每次判定后减少一般搜索量。聪明的小朋友可能会想用堆来解决这个问题，因为这个问题可以转化为求第k大的数，但是这样的话没有利用到两个数组分别有序的条件，在建树过程还需要\\(m+n\\)的复杂度，事实上还是不能达到要求。愚笨的我想了很久都没有得到一个比较好的方法，感觉这个题是纯粹在考你能不能发现一个不错的规律，所以它被定位Hard绝对是有原因的。最后还是屈服去看了Solution，看完后这样想：这TM能想到个Hammer。

还记得上次[回文子串](https://prohuper.github.io/2017/12/06/LeetCode-5-longestPalindrome/)马拉车算法的解题思路吗？马拉车只能适用于奇数长度的字符串，所以当时用了“插空法”，在每个实际元素之前个之后都插入虚拟符号“#”来把数据 总长度变成奇数，并且这样不影响数据原有的特性，源数据下标可以直接通过下标/2得到。由于中位数的求解与奇偶性有关，这里使用“插空法”可以帮我们省掉很多麻烦。下面再来介绍另外一个概念“cut”，由于大体思路是使用二分法进行分割，那么我们要有东西来描述这个分割的位置在哪里。这个cut的分割位置是可以在两个数之间也可以是在某个数之上的，如下：

>1 2 | 3 4 5
>
>1 2 (3|3) 4 5

然后我们将两个数组描述成如下形式：

|         left          | cut  |             right             |
| :-------------------: | :--: | :---------------------------: |
| \\(a_1,a_2,...,a_i\\) |  \|  | \\(a_{i+1},a_{i+2},...,a_m\\) |
| \\(b_1,b_2,...,b_j\\) |  \|  | \\(b_{j+1},b_{j+2},...,b_n\\) |

我们发现如果$a_i<=b_{j+1}\&\&b_j<=a_{i+1}$，就会有两个数组左半边的值全部小于右半边？？？由于我们要找第K个大的数，所以左半边总数为k就是我们的停止条件？？？然后问题就迎刃而解了呀，我们只要不断调整两个数组里cut的位置就行了。如果 $a_i>b_{j+1}$，需要把$C_1$左移，把$C_2$，增大。$b_j>a_{i+1}$同理，把$C_1$增大，$C_2$减小。大体思路就是这样，另外使用“插空法“之后也可以很轻松地根据$C_1,C_2$的值确定两个数组被cut之后左右边界的值，就看你观察得够不够仔细了（以数组一为例，数组二同理）：

>$a_i = (C_i-1)/2$
>$a_{i+1} = C_i/2$

找到两个cut的位置之后，需要参与计算的元素无非就是在两个cut附近的四个元素里了，从左边两个选出最小的一个，从右边两个选取最大的一个，平均一下就行了。

>$(Max(a_i+b_i) + Min(a_{i+1}+b_{i+1}) )/2$

当然还有一些边界控制条件就不用详细介绍了。准备工作比较多，代码还是比较精简的，以下：

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int total1 = nums1.length;
        int total2 = nums2.length;
    
   /*
   这里是另一个细节处理，因为只要确定一个cut的位置，另一个也就确定了，
   为了减少复杂度，选取最短的数组操作。
   */
        if(total1 > total2)
            return findMedianSortedArrays(nums2, nums1);

        int L1 = 0, L2 = 0, R1 = 0 ,R2 = 0, C1, C2;
        int low = 0, high = 2*total1;  //确定二分的上下界

        while(low <= high){

            C1 = (low + high) / 2;
            C2 = total1 + total2 - C1;
            L1 = (C1 == 0)?Integer.MIN_VALUE:nums1[(C1-1)/2];   //C1无法继续左移，L1取最小值。
            R1 = (C1 == 2*total1)?Integer.MAX_VALUE:nums1[C1/2]; //C1无法继续右移，R1取最大值。
            L2 = (C2 == 0)?Integer.MIN_VALUE:nums2[(C2-1)/2];
            R2 = (C2 == 2*total2)?Integer.MAX_VALUE:nums2[C2/2];

            //搜索长度减半，达到二分的效果,这三个条件只能满足一个。
            if(L1 > R2)
                high = C1 - 1;
            else if(L2 > R1)
                low = C1 + 1;
            else break;
        }
        return (Math.max(L1,L2) + Math.min(R1,R2))/2.0;
    }
}

```

恩，就是这样，最后吐槽一下，java内部变量的初始化机制，就不能支持自动初始化吗？或者要是出于安全原因考虑的话能不能全部要求初始化？

> int L1 = 0, L2 = 0, R1 = 0 ,R2 = 0, C1, C2;

比如刚才上面这句，这也太不整齐了吧。