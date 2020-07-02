---
layout: post
title: 算法与数据结构 - 哈希表总结
category: algorithm
tags: [algorithm]
excerpt: Algorithm and Data structure - HashTable  
---


本文总结下哈希表相关的常见题目，还有一些心得体会:  

第一点，一定要树立这样一种观念：  

整数数组也可以充当哈希表。  

举个例子，给你一个字符串，其中的字母都是小写字母，请统计每个字母出现的次数。  

如果，你用`HashTable`，就像这样： `HashMap<Character, Integer>`。  

其中`key`代表字母，`value`代表字母出现的次数。  

但是，由于上面的限制： 所有字母都是小写字母。  

所以，我们完全可以使用一个长度为`26`的整数数组替代`HashMap`。  

``` java
int[] arr = new int[26];
for (char c: s.toCharArray()){
    arr[c - 'a']++;
}
```

第二点，通常做到这类题时，可以选择`HashMap`或者`HashSet`，有的时候都要用到。  

那么，我们就要熟悉其中一些重要的方法：  

- `HashMap.getOrDefault(x, 0)`  
- `HashMap.containsKey(x)`  
- `HashSet.add(x)`  



## 哈希表    

- *1、[[1] Two Sum](http://yaoyichen.cn/algorithm/2020/04/19/leetcode-1.html){:target="_blank"}  
- 2、[[217] Contains Duplicate](http://yaoyichen.cn/algorithm/2020/02/15/leetcode-217.html){:target="_blank"}  
- *3、[[594] Longest Harmonious Subsequence](http://yaoyichen.cn/algorithm/2020/03/16/leetcode-594.html){:target="_blank"}  
- *4、[[128] Longest Consecutive Sequence](http://yaoyichen.cn/algorithm/2020/04/19/leetcode-128.html){:target="_blank"}  
- 5、[[771] Jewels and Stones](http://yaoyichen.cn/algorithm/2020/06/28/leetcode-771.html){:target="_blank"}  
- *6、[[1365] How Many Numbers Are Smaller Than the Current Number](http://yaoyichen.cn/algorithm/2020/03/11/leetcode-1365.html){:target="_blank"}  
- *7、[[1207] Unique Number of Occurrences](http://yaoyichen.cn/algorithm/2020/03/12/leetcode-1207.html){:target="_blank"}  
- *8、[[1002] Find Common Characters](http://yaoyichen.cn/algorithm/2020/03/13/leetcode-1002.html){:target="_blank"}  
- 9、[[1160] Find Words That Can Be Formed by Characters](http://yaoyichen.cn/algorithm/2020/03/12/leetcode-1160.html){:target="_blank"}  
- *10、[[463] Island Perimeter](http://yaoyichen.cn/algorithm/2020/03/13/leetcode-463.html){:target="_blank"}  
- 11、[[884] Uncommon Words from Two Sentences](http://yaoyichen.cn/algorithm/2020/03/14/leetcode-884.html){:target="_blank"}  
- 12、[[1189] Maximum Number of Balloons](http://yaoyichen.cn/algorithm/2020/03/14/leetcode-1189.html){:target="_blank"}  
- 13、[[389] Find the Difference](http://yaoyichen.cn/algorithm/2020/03/13/leetcode-389.html){:target="_blank"}  
- 14、[[387] First Unique Character in a String](http://yaoyichen.cn/algorithm/2020/06/28/leetcode-387.html){:target="_blank"}  
- 15、[[350] Intersection of Two Arrays II](http://yaoyichen.cn/algorithm/2020/06/28/leetcode-350.html){:target="_blank"}  
- 16、[[599] Minimum Index Sum of Two Lists](http://yaoyichen.cn/algorithm/2020/03/15/leetcode-599.html){:target="_blank"}  
- *17、[[202] Happy Number](http://yaoyichen.cn/algorithm/2020/03/15/leetcode-202.html){:target="_blank"}  
- 18、[[409] Longest Palindrome](http://yaoyichen.cn/algorithm/2020/03/15/leetcode-409.html){:target="_blank"}  
- *19、[[645] Set Mismatch](http://yaoyichen.cn/algorithm/2020/03/16/leetcode-645.html){:target="_blank"}  
- *20、[[204] Count Primes](http://yaoyichen.cn/algorithm/2020/03/17/leetcode-204.html){:target="_blank"}  
- *21、[[36] Valid Sudoku](http://yaoyichen.cn/algorithm/2020/05/24/leetcode-36.html){:target="_blank"}  
- *22、[[3] Longest Substring Without Repeating Characters](http://yaoyichen.cn/algorithm/2020/03/10/leetcode-3.html){:target="_blank"}  




## 同形异构 
- *1、[[242] Valid Anagram](http://yaoyichen.cn/algorithm/2020/03/14/leetcode-242.html){:target="_blank"}  
- *2、[[205] Isomorphic Strings](http://yaoyichen.cn/algorithm/2020/04/21/leetcode-205.html){:target="_blank"}  
- *3、[[290] Word Pattern](http://yaoyichen.cn/algorithm/2020/06/29/leetcode-290.html){:target="_blank"}  




