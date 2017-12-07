---
layout:     post
title:      "LeetCode 05 最长回文子串"
subtitle:   "四种方法求解回文子串"
date:       2017-12-6 12:00:00
author:     "Huper"
tags:
    - 算法
---

>我就是笨，就是不能放弃刷题。

最长回文子串，本来是很经典的DP问题，在刷题的时候无意间看到题解区另一个比较tricky的方法，所以记录一下。题目就不用介绍了，求给定字符串的最长回文子串，解题思路有很多方式，这里介绍四种比较常用的。

# 暴力法

这个不用详细介绍了，两个循环用来固定起始和终止的指针，一个循环用来检验匹配，复杂度为\\(O(n^3)\\)，直接上码：

```java
class Solution {
    public String longestPalindrome(String s) {
    	int total = s.length();
      	String result = "";
      	if(total <= 1)
          return s;
      	for(int i = 0; i < total; i++)
          for(int j = i+1; j < total; j++){
            String matched = match(s,i,j);
          	result = result.length() < matched.length() ? matched : result; 
          }
      	return result;
    }
  
    public String match(String s, int left, int right){
      	int from = left, to = right;
        while(from <= to){
            if(s.charAt(from) == s.charAt(to)){
		from++;
              	to--;
            }
          	else return s.substring(from-1,to);
        }
      	return s.substring(left, right+1);
    }
}
```

# 动态规划

本题的动规划分子问题是：

$$
\begin{equation}
dp[i,j]=
   \begin{cases}
   dp[i+1,j-1] && str[i]=str[j]\\
   0 && str[i] \not= str[j] 
   \end{cases}
  \end{equation}
$$

其中\\(dp[i][j]\\)保存的是当前字符串是否为回文串其成立条件是\\(dp[i+1][j-1]==1\\)和\\(str[i]==str[j]\\)，否则当前下标对应的字符串不是回文串。

当然还有一种直接保存回文串长度的dp思想：

$$
\begin{equation}
dp[i,j]=max
   \begin{cases}
   dp[i+1,j-1]+2  && str[i] == str[j] \&\&dp[i+1, j-1] == j-i-1\\
   dp[i-1,j]\\ 
   dp[i,j-1]
   \end{cases}
  \end{equation}
$$

***这里安利一款MD的编辑器--[Typora](https://www.typora.io/)，优雅纯净轻量（看起来就是个TXT编辑器），使用编辑后渲染模式（不同于一般MD编辑器的左编辑右渲染模式），支持Tax公式语法。趁免费赶紧下一波。***

DP的时间复杂度是\\(O(n^2)\\)，只减少了一层循环（也就是暴力法里用来检验匹配的循环，这是时候是直接从dp数组里fetch），由于前后需要前后两个指针，dp数组自然是二维的。代码如下：

```java
class Solution {
    public static String longestPalindrome(String s){
        int total = s.length();
        String result = "";
        int[][] dp = new int[total][total];
        if(total == 0 || total == 1)
            return s;

        for(int i = 0; i < total; i++){
            dp[i][i] = 1;
        }

        for(int i = 0; i < total; i++){
            for(int j = i + 1; j < total; j++){
                if(s.charAt(i) == s.charAt(j)){
                    if(j == i+1)
                        dp[i][j] = 1;
                    else
                        dp[i][j] = dp[i+1][j-1];
                }
                else dp[i][j] = 0;
                if(dp[i][j] == 1 && result.length() < j-i+1)
                    result = s.substring(i,j+1);
            }
        }

        return result;
    }
}
```

# 中心扩展法

从暴力法和DP的解决思路来看，是每次先将待检验字符串的起始和终止下标都确定以后再缩小范围进行检验的，这样很明显太麻烦了，因为要保存最近的匹配信息，一旦匹配失败返回上一次成功的状态。有种思想就是先固定最小范围（1），然后向外扩散检验，这样只要在匹配失败的时候返回当前状态就行了，复杂度还是\\(O(n^2)\\)。另外，中心扩展法需要判断一种特殊情况，就是有相同字符连着的时候。

```java
class Solution {
    public String longestPalindrome(String s){
        int total = s.length();
        String result = "";
        if(total <= 1)
            return s;
        for(int i = 0; i < total; i++){
            String matched = match(s,i);
            if(result.length() < matched.length())
                result = matched;
        }
        return result;
    }

    public String match(String s, int mid){
        int left = mid - 1;
        int right = mid + 1;
      	//检验连续的重复元素,只用照顾右边就行了，因为外部循环是从左扫到右的。
        while(s.charAt(mid) == s.charAt(right++));
        while(left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)){
            left--;
            right++;
        }

        return s.substring(left+1, right);
    }
}
```

# Manacher算法

这是个比较著名的算法，之前有一道中位数的题也是用到了这个算法，后面我会讲解到，基本思想还是中心扩展法，但是用了一个很Tricky的插空思想。

按照之前中间扩散法的思想，遍历字符串的每个元素，左右同时向外扩散检验。但是判断连续出现的某个字符。有没有办法避免这个操作呢？其实仔细想想这个问题，如果有某种方法可以把两个字符之间的空格也当成扩散的中心，就可以很好地结局重复字符的问题了，插空法的思想就是假设往两个字符之间都插入一个虚拟的‘#’字符，比如说之前的序列是：

>a b c d e

插入虚拟字符后的序列就是：

>a # b # c # d # e

我们来观察一下这个新的序列，其长度是2x-1，一定是奇数，并且每个实际字符的新序号除以二等于其对应在旧序列里的序号。那么就好说了，遍历新的序列，如果是实际字符就以其为中心进行扩散，如果是虚拟字符就抽出两个字符进行扩散，代码如下：

```java
class Solution {
    public static String longestPalindrome(String s) {
        int total = s.length();
        int left;
        int right;
        String result = "";
        String temp = "";
        for(int index = 0; index < total*2-1; index++){
            left = index / 2;
            right = index / 2;
            if(index % 2 == 1){
                right++;
            }
            temp = sideCheck(s, left, right);
            if(temp.length() > result.length()){
                result = temp;
            }
        }
        return result;
    }
    
    public static String sideCheck(String s, int left, int right){
        while(left >=0 && right < s.length()){
            if(s.charAt(left) == s.charAt(right)){
                left--;
                right++;
            }
            else break;
        }
        return s.substring(left+1, right);
    }
}
```

乍一看的确是简单了很多，但是很不幸，它的复杂度还是\\(O(n^2)\\)，于是就要用到最后的升级算法---马拉车算法了。其实我懒得画图介绍了，大家可以自己去查一下，主要还是用到了回文串的对称性，并用数组保存已经查得的回文串半径长度。由于马拉车算法中每次只用匹配未检验的字符串部分，最终的复杂度可以减小到\\(O(n)\\)。。。

```java
class Solution {
    public static String longestPalindrome(String s) {
        String origin = "";
        int total = s.length();
        origin += "+";
        for(int i = 0; i < total; i++){
            origin += "#";
            origin += s.charAt(i);
        }
        origin += "#-";
        int mx = 0, po = 0;
        int rlen = 0, rp = 0;
        int[] len = new int[2*total+2];
        for(int i = 1; i <= 2*total+1 ; i++){
            if(mx > i)
                len[i] = Math.min(mx-i, len[2*po - i]);
            else len[i] = 1;
            while(origin.charAt(i-len[i]) == origin.charAt(i+len[i]))
                len[i]++;
            if(len[i]+i > mx){
                mx = len[i]+i;
                po = i;
            }
            if(len[i] > rlen){
                rlen = len[i];
                rp = i;
            }
        }
        return s.substring((rp-rlen)/2, (rp+rlen)/2-1);
    }
}
```
