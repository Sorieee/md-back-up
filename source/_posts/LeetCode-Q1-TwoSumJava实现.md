---
title: LeetCode_Q1_TwoSumJava实现
date: 2020-04-04 22:17:50
tags: leetcode
---

```java
/**
 * 思路:
 *  边循环边构建一个差值对index的map
 *  如果该map包含cur,那么直接返回 [leftNumToIndex.get(cur), i]即可
 *  结果: 通过， 时间消耗2ms, 内存消耗39.8 MB
 * @param nums   :
 * @param target
 * @return
 * @author soriee
 * @date 2020/4/4 22:01
 */
public int[] twoSum(int[] nums, int target) {
    //初始化为length避免扩容机制造成的效率问题和内存问题 不写length时间消耗2ms
    Map<Integer, Integer> leftNumToIndex = new HashMap<>(nums.length);
    for (int i = 0; i < nums.length; i ++){
        int cur = nums[i];
        if (leftNumToIndex.containsKey(cur)) {
            return new int[]{leftNumToIndex.get(cur), i};
        }
        int left = target - nums[i];
        leftNumToIndex.put(left, i);
    }

    return null;
}
```