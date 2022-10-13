## 排序（Sort）



#### 归并排序（Merge Sort）

- **稳定**的排序方法，时间复杂度**最优**、**最坏**和**平均**都是 $O(n\,log\,n)$ ，时间复杂度为 $O(n)$ ；

- 归并排序可以只使用 $O(1)$ 的辅助空间。

  ```go
  //  [l,r)
  func merge(l, r int) {
  	if r-l <= 1 {
  		return
  	}
  	mid := l + (l+r)>>1
  	merge(l, mid)
  	merge(mid, r)
  	for i, j, k := l, mid, l; k < r; k++ {
  		if j == r || (i < mid && a[i] <= a[j]) {
  			tmp[k] = a[i]
  			i++
  		} else {
  			tmp[k] = a[j]
  			j++
  		}
  	}
  	for i := l; i < r; i++ {
  		a[i] = tmp[i]
  	}
  }
  ```

- **典型例题**：[315. Count of Smaller Numbers After Self](https://leetcode.cn/problems/count-of-smaller-numbers-after-self/)，**判断后序数列中有几个比当前数少**  ：

  - 通过归并排序部分有序的特性，每次会**合并前后两部分**，那么我只要在合并的时候关注后一部分移到前一部分的个数即可：

    ![merge-sort](images/merge-sort.svg)



#### 桶排序（Bucket Sort）

- 桶排序（`Bucket sort`）是排序算法的一种，适用于**待排序数据值域较大但分布比较均匀**的情况。一般情况下认为**桶排序**是一种**稳定**的排序算法。
- 桶排序按下列步骤进行：
  1. 设置一个定量的数组当作空桶；
  2. 遍历序列，并将元素一个个放到对应的桶中；
  3. 对每个不是空的桶进行排序；
  4. 从不是空的桶里把元素再放回原来的序列中。
- 桶排序的平均时间复杂度为 $O(n+n^2/k+k)$ （将值域平均分成 $k$ 块 + 排序 + 重新合并元素），当 $k \approx n$ 时为 $O(n)$ 。



#### 排序例题

| ID   | LeetCode 题号                                                | 描述                                                         | 思路                                                        |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------------------- |
| 1    | [315. Count of Smaller Numbers After Self](https://leetcode.cn/problems/count-of-smaller-numbers-after-self/) | 判断后序数列中有几个比当前数少                               | 归并排序中计算前半个数组中数据在排序中的偏移                |
| 2    | [剑指 Offer 51. 数组中的逆序对](https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof/) | 前面一个数字大于后面的数字即组成一个逆序对，求数组中逆序对个数 | 转移为 `ID-1`，归并排序中计算前半个数组中数据在排序中的偏移 |
| 3    | [973. K Closest Points to Origin](https://leetcode.cn/problems/k-closest-points-to-origin/) | 离原点最近的 `K` 个点                                        | 暴力排序                                                    |

