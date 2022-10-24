## 并查集（Disjoint-Set Data Structure）

#### 介绍

![img](images/disjoint-set.svg)

- **并查集**是一种用于管理元素所属集合的数据结构，实现为一个森林，其中每棵树表示一个集合，树中的节点表示对应集合中的元素，本质上是一棵棵父亲树。

  ```go
  type dsu struct {
      pa []int
  }
  ```

- **并查集**支持两种操作：

  - **合并**（`union`）：合并两个元素所属集合（合并对应的树）：

    ![img](images/disjoint-set-merge.svg)

    ```go
    func (d *dsu) unite(x int, y int) {
        d.pa[d.find(x)] = d.find(y)
    }
    ```

  - **查询**（`find`）：查询某个元素所属集合（查询对应的树的根节点），这可以用于判断两个元素是否属于同一集合：

    ![img](images/disjoint-set-find.svg)

    ```go
    func (d *dsu) find(x int) int {
        if d.pa[x] == x {
            return x
        }
        return d.find(d.pa[x])
    }
    ```

  - **查询时路径压缩**：

    ![img](images/disjoint-set-compress.svg)

    ```go
    func (d *dsu) find(x int) int {
        if d.pa[x] == x {
            return x
        }
        d.pa[x] = d.find(d.pa[x])
        return d.pa[x]
    }
    ```



- 快速合并图的不同的独立部分
- 判断两个部分是否连通



#### 并查集的例题

| ID   | LeetCode 题号 | 描述 | 思路 |
| ---- | ------------- | ---- | ---- |
|      |               |      |      |
|      |               |      |      |
|      |               |      |      |

| 题                        | 题   | 题   | 题   |
| ------------------------- | ---- | ---- | ---- |
| 305. Number of Islands II |      |      |      |
|                           |      |      |      |
|                           |      |      |      |

