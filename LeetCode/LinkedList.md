## 链表（Linked List）

- **链表**采用链状结构，指针访问相邻的节点。

- **单向链表**：

  ![img](images/list.svg)

  ```go
  type node struct {
      val 	interface{}
      next 	*node
  }
  ```

- **双向链表**：

  ![img](images/double-list.svg)

  ```go
  type node struct {
      val 	interface{}
      next 	*node
      prev 	*node
  }
  ```



**常用策略**

1. **链表的倒置**：（例题：[206. Reverse Linked List](https://leetcode.cn/problems/reverse-linked-list/)）

   - 我个人喜欢**头插法**配合**额外的头指针**（`Dummy Head`）使用，对于边界条件比较友好：（代码例题：[25. Reverse Nodes in k-Group](https://leetcode.cn/problems/reverse-nodes-in-k-group/)）

   - 同样也是**双指针**的小trick：

     ```go
     func reverseKGroup(head *ListNode, k int) *ListNode {
     	if head == nil {
     		return head
     	}
     	emptyHead := &ListNode{Next: head}
     	ptr := emptyHead
     	stopped := emptyHead.Next
     	for {
     		for i := 0; i < k; i++ {
     			if stopped != nil {
     				stopped = stopped.Next
     			} else {
     				return emptyHead.Next
     			}
     		}
             // insert from head
     		tmp := ptr.Next
     		t := tmp
     		for tmp != stopped {
     			t := tmp.Next
     			tmp.Next = ptr.Next
     			ptr.Next = tmp
     			tmp = t
     		}
     		t.Next = stopped
     		ptr = t
     	}
     }
     ```

2.   **`Floyd` 判圈法（龟兔赛跑算法）**：（例题：[141. Linked List Cycle](https://leetcode.cn/problems/linked-list-cycle/)）

   - 通过**双指针（快慢指针）**的办法，**跑得快**的 “兔子” 和**跑得慢**的 “乌龟” 终究会在环上相遇；

     ```go
     type ListNode struct {
     	Val  int
     	Next *ListNode
     }
     
     func hasCycle(head *ListNode) bool {
     	if head == nil {
     		return false
     	}
     	slow, fast := head, head.Next
     	for slow != nil && fast != nil {
     		if fast.Next == nil || fast.Next.Next == nil {
     			return false
     		}
     		fast = fast.Next.Next
     		slow = slow.Next
     		if fast == slow {
     			return true
     		}
     	}
     	return false
     }
     ```

3. 



#### 链表例题

| ID   | LeetCode 题号                                                | 描述                  | 思路                           |
| ---- | ------------------------------------------------------------ | --------------------- | ------------------------------ |
| 1    | [141. Linked List Cycle](https://leetcode.cn/problems/linked-list-cycle/) | 找链表里的环          | `Floyd` 判圈法（龟兔赛跑算法） |
| 2    | [19. Remove Nth Node From End of List](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/) | 删除倒数第 `N` 个元素 | 前后两指针 + 空表头            |
| 3    | [206. Reverse Linked List](https://leetcode.cn/problems/reverse-linked-list/) | 链表倒置              | 头插法                         |
| 4    | [25. Reverse Nodes in k-Group](https://leetcode.cn/problems/reverse-nodes-in-k-group/) | 分组链表倒置          | 头插法                         |
| 5    | [146. LRU Cache](https://leetcode.cn/problems/lru-cache/)    | LRU                   | 哈希表 + 双向链表              |
| 6    | [面试题 16.25. LRU Cache LCCI](https://leetcode.cn/problems/lru-cache-lcci/) | LRU（就是上面一道题） | 哈希表 + 双向链表              |

| 题                                     | 题                                            |      |      |
| -------------------------------------- | --------------------------------------------- | ---- | ---- |
| 2. Add Two Numbers  (Merge LinkedList) | 138 Copy List with Random Pointer (Deep copy) |      |      |

