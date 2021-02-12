---
title: LeetCode(一)
tags: [LeetCode]
categories: [算法]
date: 2020-09-18 17:30:00
---

1. 两数之和
   给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出`和`为`目标值`的那 两个 整数，并返回他们的`数组下标`。

   你可以假设每种输入只会对应一个答案。但是，数组中同一个元素`不能`使用两遍。
   
   ```java
      /**
       * 给定 nums = [2, 7, 11, 15], target = 9 
       * 因为 nums[0] + nums[1] = 2 + 7 = 9 
       * 所以返回 [0, 1]
       */
      class Solution {
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
      }
   题解：遍历nums，根据输入的target，算出应该目标值target - nums[i]，将算出的值存到HashMap（值为key,下标为value）中，存之前先去map中取，存在则结束，否则将计算的目标值存起来。由于HashMap查询复杂度为O(1)，所以用HashMap来做容器。
   ```
2. 两数相加

   给出两个 非空 的链表用来表示两个`非负`的整数。其中，它们各自的位数是按照 `逆序` 的方式存储的，并且它们的`每个节点只能存储 一位 数字`。

   如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。

   您可以假设`除了数字 0 `之外，这两个数都`不会以 0 开头`。

   ```java
    /**
       * 示例：
       * 输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
       * 输出：7 -> 0 -> 8
       * 原因：342 + 465 = 807
    *
       * Definition for singly-linked list.
       * public class ListNode {
       *     int val;
       *     ListNode next;
       *     ListNode(int x) { val = x; }
       * }
       */
      class Solution {
          public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
              ListNode header = new ListNode(0);
              int carry = 0;
              ListNode p = l1, q = l2, curr = header;
              while(p != null || q != null){
                  int x = (p != null) ? p.val : 0;
                  int y = (q != null) ? q.val : 0;
                  int sum = (x + y + carry); // 两数相同位数相加，carry为是否有进位情况 上一轮循环也就是低位产生的进位值
                  carry = sum / 10;// 满十进一 进位的值 
                  curr.next = new ListNode(sum % 10);
                  curr = curr.next;
                  p = (p != null) ? p.next : null;
                  q = (q != null) ? q.next : null;
              }
              if (carry > 0) {
                  curr.next = new ListNode(carry);
              }
              return header.next;
          }
      }
   题解：两数相加，注意满十进一，以及ListNode遍历即可。此方法为循环求解。
   ```

   ```java
   /**
       * 示例：
       * 输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
       * 输出：7 -> 0 -> 8
       * 原因：342 + 465 = 807
       *
       * Definition for singly-linked list.
       * public class ListNode {
       *     int val;
       *     ListNode next;
       *     ListNode(int x) { val = x; }
       * }
       */
      class Solution {
          public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
              return addTowNumber2(l1,l2,new ListNode(0));
          }
           public ListNode addTowNumber2(ListNode l1, ListNode l2, ListNode res){
            ListNode curr = res;
              l1 = (l1 != null) ? l1 : new ListNode(0); // l1为null 下一次递归判断 l1.next 空指针异常
              l2 = (l2 != null) ? l2 : new ListNode(0);
              int sum = l1.val + l2.val + curr.val;
              curr.val = sum % 10;
              int carry = sum / 10;
              if (l1.next != null || l2.next != null){
                  curr.next = new ListNode(carry);
                  curr = curr.next;
                  addTowNumber2(l1.next,l2.next,curr);
              }else{
                  if (carry > 0){
                      curr.next = new ListNode(carry); // (5) + (5) = (1 -> 0) 
                  }
              }
              return res;
          }
      }  
   
   ```

3. 无重复字符的最长子串

   给定一个字符串，请你找出其中不含有重复字符的 `最长子串` 的长度。

   ```java
   	/**
        * 给定一个字符串，请你找出其中不含有重复字符的 最长子串 的长度。
        * 输入: "abcabcbb"
        * 输出: 3
        * 解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
        *
        * 输入: "bbbbb"
        * 输出: 1
        * 解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
        *
        * 输入: "pwwkew"
        * 输出: 3
        * 解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
        *      请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。
        *
        */
   	public int lengthOfLongestSubstring(String s) {
        char[] chars = s.toCharArray();
           int res = 0,curr = 0,index = 0, strLength = chars.length;
           for (int i = index; i < strLength; i++) {
               for (int j = index; j < i; j++) {
                   if (chars[i] == chars[j]) {
                       index++;
                       i = index;
                       j = index;
                       res = Math.max(res,curr);
                       curr = 0;
                   }
               }
               curr++;
           }
           res = Math.max(res,curr);
           return res;
       }
   题解：从左到右遍历，每拿到一个char都与之前的遍历过的char进行比较，如果有重复，那么从重复的元素，重新开始遍历。
   ```
   
   



​	