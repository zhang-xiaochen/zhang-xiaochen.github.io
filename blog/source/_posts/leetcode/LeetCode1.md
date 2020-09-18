---
title: LeetCode(一)
tags: [LeetCode]
categories: [算法]
date: 2020-09-18 17:30:00
---

### 两数之和

给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中`同一个元素不能使用两遍`。

示例:
    给定 nums = [2, 7, 11, 15], target = 9
    因为 nums[0] + nums[1] = 2 + 7 = 9
    所以返回 [0, 1]
    Related Topics 数组 哈希表
    👍 8865 👎 0

```java
 public int[] twoSum(int[] nums, int target) {
        HashMap<Integer, Integer> temp = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            if (temp.get(nums[i]) != null) {
                return new int[]{temp.get(nums[i]),i};
            }
            temp.put(target - nums[i], i);
        }
        return new int[2];
    }
```