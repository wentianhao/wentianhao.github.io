---
title: 动态规划算法详解
katex: false
date: 2021-04-03 19:12:19
tags: DP
categories: 
            - LeetCode
            - DP
---
动态规划DP(Dynamic programming):通过把原问题分解为相对简单的子问题的方式求解复杂问题的方法

DP 常适用于 有 **重叠子问题** 和 **最优子结构性质** 的问题，动态规划方法所耗时间往往低于朴素解法

## 基本思想
若要解一个给定的问题，需解其不同部分(即子问题)，再根据子问题的解以得到原问题的解。

<!-- more -->

## DP 需要注意的要素

1. 明确dp二维数组表示的含义
2. base case
3. 状态的转移： 对于回文/LCS(最长公共子序列)之类的问题则是考虑当前字串与已经计算过的字串之间的关系
4. 由状态的转移来确定 loop的边界
5. 由loop的边界打出表格，可得出最后一个dp的状态值，即结果。

## LeetCode 1143.最长公共子序列
>>给定两个字符串 text1 和 text2，返回这两个字符串的最长 公共子序列 的长度。如果不存在 公共子序列 ，返回 0

1. 对于s[1,...,i] t[1,...,j] LCS 长度为dp[i][j]
2. base case 一个字符串和自身没有子序列 dp[0][j] = dp[i][0] = 0  
3.  ```java
        s.charAt(i) = t.charAt(j) :   dp[i][j] = dp[i-1][j-1] + 1 
        s.charAt(i) != t.charAt(j):   dp[i][j] = max(dp[i-1][j],dp[i][j-1])
    ```
4.  ```python
        for i in range(n+1):
            for j in range(m+1):
    ```
5. dp[n][m]

状态矩阵
```java
/*              j
       *  a  b  c  b  a
    *  0  0  0  0  0  0
    a  0  1  1  1  1  1
    b  0  1  2  2  2  2
i   c  0  1  2  3  3  3
    b  0  1  2  3  4  4
    c  0  1  2  3  4  4
    b  0  1  2  3  4  4
    a  0  1  2  3  4  5
*/
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int n = text1.length();
        int m = text2.length();
        int [][]dp = new int[n+1][m+1];
        for(int i=0;i<=n;i++){
            for(int j=0;j<=m;j++){
                dp[i][0] = dp[0][j] = 0;
            }
        }
        for(int i=1;i<=n;i++){
            for(int j=1;j<=m;j++){
                if(text1.charAt(i-1) == text2.charAt(j-1)){
                    dp[i][j] = dp[i-1][j-1] + 1;
                }else{
                    dp[i][j] = Integer.Max(dp[i-1][j],dp[i][j-1]);
                }
            }
        }
        return dp[n][m];
    }
}
```

## LeetCode 115.不同的子序列
>>给定一个字符串 s 和一个字符串 t ，计算在 s 的子序列中 t 出现的个数。

1. 对于s[1,...,j] t[1,...,i] 在s的子序列中t出现的个数为dp[i][j]
2. base case t为空串时，dp[0][j] = 1; 
3.  ```java
        s.charAt(i) = t.charAt(j) :   dp[i][j] = dp[i-1][j-1] + dp[i][j-1];
        s.charAt(i) != t.charAt(j):   dp[i][j] = dp[i][j-1]
    ```
4.  ```python
        for i in range(m+1):
            for j in range(n+1):
    ```
5. dp[m][n]

```java
/**
*
*    *  b  a  b  g  b  a  g
* *  1  1  1  1  1  1  1  1
* b  0  1  1  2  2  3  3  3
* a  0  0  1  1  1  1  4  4
* g  0  0  0  0  1  1  1  5
*/
    public int numDistinct(String s, String t) {
        int n = s.length();
        int m = t.length();
        int[][] dp = new int[m + 1][n + 1];
        //初始化第一行
        for(int j = 0; j <= n; j++){
            dp[0][j] = 1;
        }

        for(int i = 1; i <= m; i++){
            for(int j = 1; j <= n; j++){
                if(t.charAt(i-1) == s.charAt(j-1)){
                    dp[i][j] = dp[i-1][j-1] + dp[i][j-1];
                }else {
                    dp[i][j] = dp[i][j-1];
                }
            }
        }
        return dp[m][n];
    }
```
