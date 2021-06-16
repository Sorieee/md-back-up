---
title: manacher算法笔记
date: 2020-10-15 21:38:47
tags: [manacher]
---

https://leetcode-cn.com/problems/longest-palindromic-substring/submissions/

视频讲解：

https://www.bilibili.com/video/av4829276

其他推荐题：

POJ 1159 Palindrome
HDU 3068 最长回文
POJ 3974 Palindrome
HYSBZ 2342 双倍回文
HYSBZ 2565 最长双回文串

```java
import java.util.Arrays;
//ac
public class Solution {
//    public static void main(String[] args) {
////        System.out.println(new Solution().longestPalindrome("babad"));
////        System.out.println(new Solution().longestPalindrome("cbbd"));
////        System.out.println(new Solution().longestPalindrome("2"));
//
//    }
    public char[] manacherString(String str) {
        if (str == null) {
            return null;
        }
        char[] charArr = str.toCharArray();
        char[] res = new char[str.length() * 2 + 3];
        int index = 0;
        res[0] = '$';
        res[1] = '#';
        int j = 2;
        for (int i = 0; i != str.length(); i++) {
            res[j++] = charArr[i];
            res[j++] = '#';
        }
        res[j] = '\0';
        return res;
    }

    public String longestPalindrome(String s) {
        if (s == null) {
            return null;
        }
        char[] manacherCharArr = this.manacherString(s);
        int maxLen = -1; //最长回文长度
        int c = 0;
        int r = 0;
        int[] p = new int[s.length() * 2 + 3];
        Arrays.fill(p, 0);
        int len = s.length() * 2 + 2;
        int ansMid = -1;
        for (int i = 1; i < len; i++) {
            if (i < r) {
                p[i] = Math.min(p[2 * c - i], r - i);
            } else {
                p[i] = 1;
            }
            while (manacherCharArr[i - p[i]] == manacherCharArr[i + p[i]]) {
                p[i]++;
            }
            if (r < i + p[i]) {
                c = i;
                r = i + p[i];
            }
            if (maxLen < p[i] - 1) {
                maxLen = p[i] - 1;
                ansMid = i;
            }
        }
        StringBuilder ans = new StringBuilder();
        int left = ansMid - maxLen;
        int right = ansMid + maxLen - 1;
        for (int i = left; i <= right; i++) {
            if (manacherCharArr[i] != '#') {
                ans.append(manacherCharArr[i]);
            }
        }
        return ans.toString();
    }
}
```