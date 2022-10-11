## 栈（Stack）

- **常规栈**：就是简单的栈操作，一般直接 `pop` 和 `push` 操作就行了。（例题：`ID-1` `ID-2` `ID-3` `ID-4`）

- **单调栈**：栈中的元素按照一定的规则呈递增或者递减顺序排列，当出现失序时出栈。（例题：`ID-5` `ID-6` `ID-7` `ID-8` ）

  

#### 栈的例题

| ID   | LeetCode 题号                                                | 描述                                                     | 思路                                    |
| ---- | ------------------------------------------------------------ | -------------------------------------------------------- | --------------------------------------- |
| 1    | [946. Validate Stack Sequences](https://leetcode.cn/problems/validate-stack-sequences/) | 判断数组是否可以通过栈操作得到重排序的数组               | 用栈模拟                                |
| 2    | [735. Asteroid Collision](https://leetcode.cn/problems/asteroid-collision/) | 行星碰撞问题                                             | 用栈模拟，注意规则细节                  |
| 3    | [20. Valid Parentheses](https://leetcode.cn/problems/valid-parentheses/) | 判断括号是否合规                                         | 栈模拟                                  |
| 4    | [301. Remove Invalid Parentheses](https://leetcode.cn/problems/remove-invalid-parentheses/) | 移除非法括号                                             | 栈模拟统计 + `DFS`，竟然没有优化方法... |
| 5    | [496. Next Greater Element I](https://leetcode.cn/problems/next-greater-element-i/) | 找到数组里对应数字比他大的下一个数字                     | 严格单调递增栈                          |
| 6    | [503. Next Greater Element II](https://leetcode.cn/problems/next-greater-element-ii/) | 找到循环数组里对应数字比他大的下一个数字                 | 严格单调递增栈 + 遍历两次               |
| 7    | [556. Next Greater Element III](https://leetcode.cn/problems/next-greater-element-iii/) | 将一个数字变成比他大的最小的数字，且每位上的数字个数相同 | 模拟单调栈                              |
| 8    | [402. Remove K Digits](https://leetcode.cn/problems/remove-k-digits/) | 删掉 K 个数字后的最小数字                                | 单调栈                                  |
| 9    | [853. Car Fleet](https://leetcode.cn/problems/car-fleet/)    | 不同车速的车是否能组成车队                               | 按出发先后排序 + 单调栈                 |
| 10   | [739. Daily Temperatures](https://leetcode.cn/problems/daily-temperatures/) | 几天后会比今天更温暖                                     | 单调栈                                  |

| 题                    | 题                        | 题            | 题              |
| --------------------- | ------------------------- | ------------- | --------------- |
| Basic Calculator 系列 | Nested List Iterator 系列 | Decode String | Number of Atoms |

