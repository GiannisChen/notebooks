# LeetCode

![img](images/hello_leetcode.png)

### 题目列表

- https://www.1point3acres.com/bbs/thread-840864-1-1.html

### 参考

- https://books.halfrost.com/leetcode/
- https://algorithm-essentials.soulmachine.me/



并查集Union Find  准备模板
用于快速合并图的不同components （305. Number of Islands II）
用于快速判断两个nodes是不是连通
回溯法 Backtracking 本质就是想象成图，然后递归的DFS（有时可以剪枝)  526. Beautiful Arrangement, 22. Generate Parentheses

binarysearch+BFS： 用binary search 查找答案，然后在限制条件下做BFS。类似的用binary search 查找答案的思路见【7. 搜索和查询 中的binary search部分】
1102 Path With Maximum Minimum Value
778  Swim in Rising Water
1631 Path With Minimum Effort 

## TODO

| Status       | FileName                | Description        |
| ------------ | ----------------------- | ------------------ |
| `unfinished` | `DynamicProgramming.md` | 树上背包 `Go` 版本 |
| `unfinished` | `Graph.md`              | 并查集部分？       |
|              |                         |                    |



队 Queue, Deque
BFS related
239 Sliding Window Maximum  ---> 2D sliding window maximum ( 转化成两次1D的问题）
Moving Average from Data Stream

1. 链表 LinkedList
  Fast and Slow pointer  (detect cycle, get middle,  get kth element)
  141 Linked List Cycle
  19 Remove Nth Node From End of List
  Reverse Linked List (trick: dummy head)  206 Reverse Linked List, 25 Reverse Nodes in k-Group
  LRU cache
  Deep copy  (138 Copy List with Random Pointer)
  Merge LinkedList  (2. Add Two Numbers)
2. 排序 Sort
  Merge sort
  非常规高频题 315 Count of Smaller Numbers After Self  --> (google题： 一堆点, 对每个点(x,y)数【严格大 (x,y)<(u,v)】的点的个数. 思路：先排序，x增序，y减序，然后把y单独拿出来看，对每个点数右边有多少大的元素，变成问题315 with bigger numbers after self)
  Quick Sort --> QuickSelect O(n) time on average) 973. K Closest Points to Origin
  Bucket Sort O(n) 通常是整体数据量可能很大，但是unique元素有限
  Cycle Sort O(n) 通常是用于把0到n-1在array中排序 （不断交换的想法）
  Python built-in sort
  OrderedDict (linked list + hash) --> 自己实现： 用 hashtable 存double linkedlist 的 node
  sorted containers (sorted list, sorted dict, sorted set)
3. 搜索和查询 Search and Query
  hash (python: dictionary, set):
  O(1)查找，
  记录unique element的frequency
  binary search  左开右闭模板
  data是有顺序的，每次可以缩小搜索范围。 经典题：
  33 Search in Rotated Sorted Array，
  153 Find Minimum in Rotated Sorted Array，
  162 Find Peak Element. check 1point3acres for more.
  解的范围是一个区间可以二分搜索
  Binary search + greedy: 1231 Divide Chocolate, 1011 Capacity To Ship Packages Within D Days, 410. Split Array Largest Sum
  378 Kth Smallest Element in a Sorted Matrix
  经典题 Search a 2D Matrix 系列
  字典树 Trie 模板 （单词相关的查找）： 642. Design Search Autocomplete System， 472. Concatenated Words， 212. Word Search II
  Range Query (Segment Tree 模板)  307. Range Sum Query - Mutable. check 1point3acres for more.
4. 数组和字符串相关 （array & string）
  括号相关题 （另外见【栈】）921. Minimum Add to Make Parentheses Valid， 1249. Minimum Remove to Make Valid Parentheses
  排列（组合） Permutation
  区间题 Intervals [left, right, val]
  按左右端点排序的思想 252. Meeting Rooms
  插入，合并，删除区间： 56. Merge Intervals， 57. Insert Interval， 1272. Remove Interval， 435. Non-overlapping Intervals
  安排会议/任务  253. Meeting Rooms II， 1235 Maximum Profit in Job Scheduling， 2054 Two Best Non-Overlapping Events
  区间更新 1094. Car Pooling. 1point 3acres
  常规双指针
  15 3Sum，
  75 Sort Colors（Dutch national flag problem 经典题
  1229 Meeting Scheduler，
  680 Valid Palindrome II，
  408 Valid Word Abbreviation
  滑动窗口 Sliding window 模板. 1point 3 acres
  3 Longest Substring Without Repeating Characters
  76 Minimum Window Substring
  1004 Max Consecutive Ones III
  209  Minimum Size Subarray Sum
  1438 Longest Continuous Subarray With Absolute Diff Less Than or Equal to Limit
  Subsequence.
  Greedy drop idea: 392 Is Subsequence， 792 Number of Matching Subsequences
  经典题 727. Minimum Window Subsequence
  940 Distinct Subsequences II
  Subarray、substring （连续的）. From 1point 3acres bbs
  Rolling hash （Rabin-Karp） 1062. Longest Repeating Substring， 1044. Longest Duplicate Substring
  排列组合
  46 Permutations, 31. Next Permutation
  77 Combinations
  Subset
  78 Subsets
  368 Largest Divisible Subset
  数据流相关的问题
  top k 问题 --> heap; buck sort -->  distributed system:
  1146 Snapshot Array  (打version tag + binary search）
  359 Logger Rate Limiter
  LRU, LFU
  Median 295. Find Median from Data Stream
  Iterator  284. Peeking Iterator, 900. RLE Iterator. 1point3acres.com
  Bitmask 用bit来表示状态 847. Shortest Path Visiting All Nodes
  补充内容 (2022-01-20 13:20 +8:00):
  最近转码上岸了，总结了一下我做过的一些有代表的力扣题，回馈地里，希望对大家有帮助！
  另外还有一篇【转码找工作的资料总结】还在审核中
  链接： https://www.1point3acres.com/bbs/thread-840857-1-1.html
