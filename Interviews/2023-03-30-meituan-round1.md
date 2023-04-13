## 美团一面（凉经）

> 凉经，投的Go，上来就要求转Java，凉了并不意外。
>
> （2023-04-11更新）又被捞起来了...



### 项目

直接跳过了我头两个篇幅最大的，直奔最水的那个，可能和语言有关吧。



### 八股

1. HTTP发送URL，到服务器响应，发生了什么？
   - 协议栈：HTTP（TCP） + DNS（UDP） + ARP
   - 四层结构，层层封装，关键帧，层层解封
   - URL的识别，参数提取
2. MySQL为什么用InnoDB？
   - B+Tree的节点特点（范围查询，减少I/O，层数较低）
   - 事务（MVCC）
3. TCP和UDP区别
   - 可靠交付
   - 字节流 v.s. 数据包
4. 三次握手，为什么需要三次，而不是两次或者四次？
   - 三次流程 + 双方确认通信畅通 + 效率
5. 死锁产生的条件？如何避免？
   - 互斥条件（多个资源，读写锁）
   - 持有并等待（一次性获取，无法获取全部锁则放弃全部）
   - 不可剥夺（锁可剥夺）
   - 循环等待（有序获取）
6. 进程调度有什么调度算法？
   - RR、FCFS、多级优先队列、优先级、短时间优先...
7. 数组的中位数如何获取？
   - 数组个数比较少（排序）
   - 数组数据个数多/流式到达（双堆维护）
8. Top10问题：找出请求中最频繁的10个
   - 维护大小为10的小顶堆存请求中较大的
9. 判断一个数字和40亿个不同的数字判断是否重复？
   - Bitmap（32位大概要1GB）
   - Bloom Filter（假阳性的额可能）

10. Spring的两大特性？
    - （开摆）



### 算法题

[链表相加（二）](https://www.nowcoder.com/practice/c56f6c70fb3f4849bc56e33ff2a50b6b?tpId=295&tqId=1008772&ru=/exam/company&qru=/ta/format-top101/question-ranking&sourceUrl=%2Fexam%2Fcompany)

假设链表中每一个节点的值都在 0 - 9 之间，那么链表整体就可以代表一个整数。

给定两个这种链表，请生成代表两个整数相加值的结果链表。

- 数据范围：$0 \le n$，$m \le 1000000$，链表任意值 $0 \le val \le 9$
- 要求：空间复杂度 $O(n)$，时间复杂度$O(n)$

> 例如：链表 1 为 9->3->7，链表 2 为 6->3，最后生成新的结果链表为 1->0->0->0。

```shell
示例1
输入： [9,3,7],[6,3]
输出： {1,0,0,0}

示例2
输入： [0],[6,3]
输出： {6,3}
```

我的思路是翻转加数的两个链表，然后直接带进位相加，由于使用头插法，结果是不需要翻转的。习惯在链表头上加上空节点代表表头，这样能简化一些边界条件，可以看作是“哨兵”。

时间复杂度$O(n)$，空间复杂度$O(1)$（输出认为没有复杂度，中间只有常数的额外空间使用）

```go
package main

type ListNode struct {
	Val  int
	Next *ListNode
}

/**
 *
 * @param head1 ListNode类
 * @param head2 ListNode类
 * @return ListNode类
 */
func addInList(head1 *ListNode, head2 *ListNode) *ListNode {
	// write code here
	emptyHead1 := &ListNode{Val: -1, Next: nil}
	emptyHead2 := &ListNode{Val: -1, Next: nil}
	tmp := &ListNode{Val: -1, Next: head1}
	for tmp.Next != nil {
		tmp1 := tmp.Next
		tmp.Next = tmp1.Next
		tmp1.Next = emptyHead1.Next
		emptyHead1.Next = tmp1
	}
	tmp = &ListNode{Val: -1, Next: head2}
	for tmp.Next != nil {
		tmp2 := tmp.Next
		tmp.Next = tmp2.Next
		tmp2.Next = emptyHead2.Next
		emptyHead2.Next = tmp2
	}

	ans := &ListNode{Val: -1, Next: nil}
	num := 0
	add := 0
	for ptr1, ptr2 := emptyHead1, emptyHead2; ptr1.Next != nil || ptr2.Next != nil; {
		num += add
		if ptr1.Next != nil {
			num += ptr1.Next.Val
			ptr1 = ptr1.Next
		}
		if ptr2.Next != nil {
			num += ptr2.Next.Val
			ptr2 = ptr2.Next
		}
		add = num / 10
		num %= 10
		node := &ListNode{Val: num, Next: ans.Next}
		ans.Next = node
		num = 0
	}
	if add > 0 {
		node := &ListNode{Val: add, Next: ans.Next}
		ans.Next = node
	}
	return ans.Next
}
```

