## Golang Map

- https://golang.design/go-questions/map/extend/



#### 0. 什么是map

简单说明一下：在计算机科学里，被称为相关数组、map、符号表或者字典，是由一组 `<key, value>` 对组成的抽象数据结构，，并且同一个 key 只会出现一次。

有两个关键点：map 是由 `key-value` 对组成的；`key` 只会出现一次。

和 map 相关的操作主要是：

1. 增加一个 k-v 对 —— Add or insert；
2. 删除一个 k-v 对 —— Remove or delete；
3. 修改某个 k 对应的 v —— Reassign；
4. 查询某个 k 对应的 v —— Lookup；

简单说就是最基本的 `增删查改`。

map 的设计也被称为 “The dictionary problem”，它的任务是设计一种数据结构用来维护一个集合的数据，并且可以同时对集合进行增删查改的操作。最主要的数据结构有两种：`哈希查找表（Hash table）`、`搜索树（Search tree）`。

哈希查找表用一个哈希函数将 key 分配到不同的桶（bucket，也就是数组的不同 index）。这样，开销主要在哈希函数的计算以及数组的常数访问时间。在很多场景下，哈希查找表的性能很高。

哈希查找表一般会存在“碰撞”的问题，就是说不同的 key 被哈希到了同一个 bucket。一般有两种应对方法：`链表法`和`开放地址法`。`链表法`将一个 bucket 实现成一个链表，落在同一个 bucket 中的 key 都会插入这个链表。`开放地址法`则是碰撞发生后，通过一定的规律，在数组的后面挑选“空位”，用来放置新的 key。

搜索树法一般采用自平衡搜索树，包括：AVL 树，红黑树。面试时经常会被问到，甚至被要求手写红黑树代码，很多时候，面试官自己都写不上来，非常过分。

自平衡搜索树法的最差搜索效率是 O(logN)，而哈希查找表最差是 O(N)。当然，哈希查找表的平均查找效率是 O(1)，如果哈希函数设计的很好，最坏的情况基本不会出现。还有一点，遍历自平衡搜索树，返回的 key 序列，一般会按照从小到大的顺序；而哈希查找表则是乱序的。



#### 1. map底层的实现原理

Go 语言采用的是哈希查找表，并且使用链表解决哈希冲突。在源码中，表示 `map` 的底层数据结构是 `hmap` ，是 `hashmap` 的缩写，数据存储在 `buckets` 数组中，数组每一个元素即一个桶 `bucket` ，一个桶内可以容纳8个 `key`，这些 `key` 是通过哈希函数计算出来属于一类的，同时又通过hash值得高8位来决定落在桶内的第几个位置，而落在哪个桶中则依靠hash值的低 **B** 位，因此，`hmap` 的容量会维持在 $B^2$ 个。如果超过了8个 `key` ，那么将会有额外的桶被创建，通过指针挂在原有的 `bucket` 之上。

> 在程序启动时，检测CPU是否支持AES（aes hash），如果不支持则用memhash。

`makemap` 初始化 `map` 时返回的是 `*hmap`，即指针引用，所以 `map` 永远是引用类型，与Slice有所区别，后者虽然也是复制了一份引用，但是属于结构体拷贝，在函数内操作有可能改变函数外的Slice（比如修改数据），也可能不会影响外面（比如扩容）。



#### 2. 如何实现两种 get 操作

Go 语言中读取 `map` 有两种语法：带 comma 和 不带 comma。当要查询的 `key` 不在 `map` 里，带 comma 的用法会返回一个 `bool` 型变量提示 `key` 是否在 `map` 中；而不带 comma 的语句则会返回一个 `key` 对应 `value` 类型的零值。如果 `value` 是 `int` 型就会返回 0，如果 `value` 是 `string` 类型，就会返回空字符串。总而言之，就是返回对应类型的**零值**。

```go
// src/runtime/hashmap.go
func mapaccess1(t *maptype, h *hmap, key unsafe.Pointer) unsafe.Pointer
func mapaccess2(t *maptype, h *hmap, key unsafe.Pointer) (unsafe.Pointer, bool)
```

另外，根据 key 的不同类型，编译器还会将查找、插入、删除的函数用更具体的函数替换，以优化效率：

| key 类型 | 查找                                                         |
| -------- | ------------------------------------------------------------ |
| uint32   | mapaccess1_fast32(t *maptype, h *hmap, key uint32) unsafe.Pointer |
| uint32   | mapaccess2_fast32(t *maptype, h *hmap, key uint32) (unsafe.Pointer, bool) |
| uint64   | mapaccess1_fast64(t *maptype, h *hmap, key uint64) unsafe.Pointer |
| uint64   | mapaccess2_fast64(t *maptype, h *hmap, key uint64) (unsafe.Pointer, bool) |
| string   | mapaccess1_faststr(t *maptype, h *hmap, ky string) unsafe.Pointer |
| string   | mapaccess2_faststr(t *maptype, h *hmap, ky string) (unsafe.Pointer, bool) |

这些函数的参数类型直接是具体的 `uint32`、`uint64`、`string`，在函数内部由于提前知晓了 `key` 的类型，所以内存布局是很清楚的，因此能节省很多操作，提高效率。



#### 3. map的遍历过程

**简单过程**：遍历所有的 bucket 以及它后面挂的 overflow bucket，然后挨个遍历 bucket 中的所有 cell。每个 bucket 中包含 8 个 cell，从有 key 的 cell 中取出 key 和 value，这个过程就完成了。由于Golang的骚操作，其每次遍历的起点都是加上了随机数的，等到再次回到起点就行了。
**复杂过程**：

> 扩容过程不是一个原子的操作，它每次最多只搬运 2 个 bucket，所以如果触发了扩容操作，那么在很长时间里，map 的状态都是处于一个中间态：有些 bucket 已经搬迁到新家，而有些 bucket 还待在老地方。

假设我们有下图所示的一个 map，起始时 B = 1，有两个 bucket，后来触发了扩容（这里不要深究扩容条件，只是一个设定），B 变成 2。并且， 1 号 bucket 中的内容搬迁到了新的 bucket，`1 号`裂变成 `1 号`和 `3 号`；`0 号` bucket 暂未搬迁。老的 bucket 挂在在 `*oldbuckets` 指针上面，新的 bucket 则挂在 `*buckets` 指针上面。map 遍历的核心在于理解 2 倍扩容时，老 bucket 会分裂到 2 个新 bucket 中去。而遍历操作，会按照新 bucket 的序号顺序进行，碰到老 bucket 未搬迁的情况时，要在老 bucket 中找到将来要搬迁到新 bucket 来的 key。



#### 4. map的赋值过程

通过汇编语言可以看到，向 map 中插入或者修改 key，最终调用的是 `mapassign` 函数。实际上插入或修改 key 的语法是一样的，只不过前者操作的 key 在 map 中不存在，而后者操作的 key 存在 map 中。`mapassign` 有一个系列的函数，根据 key 类型的不同，编译器会将其优化为相应的“快速函数”。

| key 类型 | 插入                                                         |
| -------- | ------------------------------------------------------------ |
| uint32   | mapassign_fast32(t *maptype, h *hmap, key uint32) unsafe.Pointer |
| uint64   | mapassign_fast64(t *maptype, h *hmap, key uint64) unsafe.Pointer |
| string   | mapassign_faststr(t *maptype, h *hmap, ky string) unsafe.Pointer |

我们只用研究最一般的赋值函数 `mapassign`。整体来看，流程非常得简单：对 `key` 计算 hash 值，根据 hash 值按照之前的流程，找到要赋值的位置（可能是插入新 `key`，也可能是更新老 `key`），对相应位置进行赋值。源码大体和之前讲的类似，核心还是一个双层循环，外层遍历 bucket 和它的 overflow bucket，内层遍历整个 bucket 的各个 cell。

在正式安置 key 之前，还要检查 map 的状态，看它是否需要进行扩容。如果满足扩容的条件，就主动触发一次扩容操作。这之后，整个之前的查找定位 key 的过程，还得再重新走一次。因为扩容之后，key 的分布都发生了变化。最后，会更新 map 相关的值，如果是插入新 key，map 的元素数量字段 count 值会加 1。

最后根据 `mapassign` 函数返回的 `key` 对应的 value 的地址更新 value ，而不是直接通过 `mapassign` 修改。



#### 5.  map的删除过程

计算 `key` 的哈希，找到落入的 bucket。检查此 `map` 如果正在扩容的过程中，直接触发一次搬迁操作。删除操作同样是两层循环，核心还是找到 key 的具体位置。寻找过程都是类似的，在 bucket 中挨个 cell 寻找。找到对应位置后，对 `key` 或者 value 进行“清零”操作。最后，将 count 值减 1，将对应位置的 tophash 值置成 `Empty`。



#### 6. map的扩容过程

> 使用哈希表的目的就是要快速查找到目标 key，然而，随着向 map 中添加的 key 越来越多，key 发生碰撞的概率也越来越大。bucket 中的 8 个 cell 会被逐渐塞满，查找、插入、删除 key 的效率也会越来越低。最理想的情况是一个 bucket 只装一个 key，这样，就能达到 $O(1)$ 的效率，但这样空间消耗太大，用空间换时间的代价太高。Go 语言采用一个 bucket 里装载 8 个 key，定位到某个 bucket 后，还需要再定位到具体的 key，这实际上又用了时间换空间。当然，这样做，要有一个度，不然所有的 key 都落在了同一个 bucket 里，直接退化成了链表，各种操作的效率直接降为 $O(n)$，是不行的。

触发 `map` 扩容的时机：在向 map 插入新 key 的时候，会进行条件检测，符合下面这 2 个条件，就会触发扩容：

1. 装载因子超过阈值，源码里定义的阈值是 6.5。
2. overflow 的 bucket 数量过多：当 B 小于 15，也就是 bucket 总数 $2^B$ 小于 $2^{15}$ 时，如果 overflow 的 bucket 数量超过 $2^B$；当 B >= 15，也就是 bucket 总数 $2^B$ 大于等于 $2^{15}$，如果 overflow 的 bucket 数量超过 $2^{15}$。



#### 7. key 为什么是无序的

`map` 在扩容后，会发生 `key` 的搬迁，原来落在同一个 bucket 中的 `key`，搬迁后，有些 `key` 就要远走高飞了（bucket 序号加上了 $2^B$）。而遍历的过程，就是按顺序遍历 bucket，同时按顺序遍历 bucket 中的 `key`。搬迁后，`key` 的位置发生了重大的变化，有些 `key` 飞上高枝，有些 `key` 则原地不动。这样，遍历 `map` 的结果就不可能按原来的顺序了。

当然，如果我就一个 hard code 的 `map`，我也不会向 `map` 进行插入删除的操作，按理说每次遍历这样的 map 都会返回一个固定顺序的 key/value 序列吧。的确是这样，但是 Go 杜绝了这种做法，因为这样会给新手程序员带来误解，以为这是一定会发生的事情，在某些情况下，可能会酿成大错。

当然，Go 做得更绝，当我们在遍历 `map` 时，并不是固定地从 0 号 bucket 开始遍历，每次都是从一个随机值序号的 bucket 开始遍历，并且是从这个 bucket 的一个随机序号的 cell 开始遍历。这样，即使你是一个写死的 `map`，仅仅只是遍历它，也不太可能会返回一个固定序列的 key/value 对了。

多说一句，“迭代 `map` 的结果是无序的”这个特性是从 go 1.0 开始加入的。



#### 8. float类型可以作为map的key吗

可以，但不建议。首先float类型由于其表示原因（IEEE754），带来了奇怪的舍入问题，比如经典的 $1.1 \neq 1.1_{float}$ ，因为无论64位还是32位都不能表示 0.1 ，但是，原理上 `map` 是可以用float类型当 `key` 的，因为 `key` 实现了比较函数 `==` 和 `!=` 。



#### 9. 可以边遍历边删除吗

`map` 并不是一个线程安全的数据结构。同时读写一个 `map` 是未定义的行为，如果被检测到，会直接 `panic`。

上面说的是发生在多个协程同时读写同一个 `map` 的情况下。 如果在同一个协程内边遍历边删除，并不会检测到同时读写，理论上是可以这样做的。但是，遍历的结果就可能不会是相同的了，有可能结果遍历结果集中包含了删除的 `key`，也有可能不包含，这取决于删除 `key` 的时间：是在遍历到 `key` 所在的 bucket 时刻前或者后。

一般而言，这可以通过读写锁来解决：`sync.RWMutex`。读之前调用 `RLock()` 函数，读完之后调用 `RUnlock()` 函数解锁；写之前调用 `Lock()` 函数，写完之后，调用 `Unlock()` 解锁。另外，`sync.Map` 是线程安全的 `map`，也可以使用。



#### 10. 可以对map的元素取地址吗

无法对 `map` 的 `key` 或 `value` 进行取址。以下代码不能通过编译：

```go
package main

import "fmt"

func main() {
	m := make(map[string]int)

	fmt.Println(&m["qcrao"])
}
```

编译报错：

```bash
./main.go:8:14: cannot take the address of m["qcrao"]
```

如果通过其他 hack 的方式，例如 `unsafe.Pointer` 等获取到了 `key` 或 `value` 的地址，也不能长期持有，因为一旦发生扩容，`key` 和 `value` 的位置就会改变，之前保存的地址也就失效了。



#### 11. 如何比较两个map是否相等

map 深度相等的条件：

1. 都为 `nil`
2. 非空、长度相等，指向同一个 `map` 实体对象
3. 相应的 `key` 指向的 `value` “深度”相等

但是呢，直接将使用 `map1 == map2` 是错误的。这种写法只能比较 `map` 是否为 `nil`。

```go
package main

import "fmt"

func main() {
	var m map[string]int
	var n map[string]int

	fmt.Println(m == nil)
	fmt.Println(n == nil)

	// 不能通过编译
	//fmt.Println(m == n)
}
```

因此只能是遍历 `map` 的每个元素，比较元素是否都是深度相等。



#### 12. map是线程安全的吗

`map` 不是线程安全的。在查找、赋值、遍历、删除的过程中都会检测写标志，一旦发现写标志置位（等于1），则直接 `panic`。赋值和删除函数在检测完写标志是复位之后，先将写标志位置位，才会进行之后的操作。

检测写标志：

```go
if h.flags&hashWriting == 0 {
    throw("concurrent map writes")
}
```

设置写标志：

```go
h.flags |= hashWriting
```

