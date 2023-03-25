## 2023-03-25 特斯拉公开笔试



### 第1题

在数`num`的任意位置（包括开头和结尾），加入数字5，在出现的所有数字中，找出差距最大的两个数（最大数和最小数），返回他们的差距。

- $-10^5 \le num \le 10^5$

```shell
2	# possible: 25 52
27 	# return 52 - 25
```

``` shell
23	# possible: 523 253 235
288	# return 523 -235
```

#### 思路

> **首先明确正负数其实没有任何意义，最大值最小值在取绝对值后互换。**

所以我们不妨只考虑正数，什么时候最大呢？我们考虑这样的情况：我在第`i`位插入5，数字就变成了`digits[:i)-5-digits[i:)`，我要他最大，一定就是`digits[:)`这一部分都得至少是大于5的数，为什么呢，我们做一个反推，如果中间有一个数小于了5，那么我只要讲5插入这个数之前，那么整个数字就比5在后面要小，换言之，我构造了一个比最大值还大的数，明显矛盾了。

最小值也是类似的思路，`digits[:)`中所有的数字都小于等于5，这样一做，答案就很明显了。

```go
func maxDifference(num int) int {
    if num == 0 {
        return 50 - 5
    }
    if num < 0 {
        num = -num
    }
    numStr := strconv.Itoa(num)
    isMaxSet, isMinSet := false, false
    maxNum, minNum := 0, 0
    
    for i:=0; i<len(numStr); i++ {
        d := int(numStr-'0')
        if !isMaxSet {
            if d < 5 {
                maxNum = maxNum*10 + 5
                isMaxSet = true
            }
        }
        maxNum = maxNum*10 + d
        
        if !isMinSet {
            if d > 5 {
                minNum = minNum*10 + 5
                isMinSet = true
            }
        }
        minNum = minNum*10 + d
    }
    
    if !isMaxSet {
        maxNum = maxNum*10 + 5
    }
    if !isMinSet {
        minNum = minNum*10 + 5
    }
    return maxNum - minNum
}
```



### 第2题

输入一个字符串，对字符串可以做一次操作，每次操作会删掉一个指定的字符，经过若干次操作后，字符串中的字符出现的次数唯一，即不存在两个字符出现在字符串中，且他们出现的次数相等。求最小的操作次数。

```shell
"aabbbcc"
1			# "aabbbc" or "abbbcc"
```

#### 思路

暴力就完事了：

```go
func minOperations(str string) int {
    ans := 0
    m := map[byte]int{}
    max := 0
    for i:=0; i<len(str); i++ {
        c := str[i]
        m[c]++
        if m[c]>max {
            max = m[i]
        }
    }
    counts := make([]int, max+1)
    for i:=max; i>0; i-- {
        if counts[i] <= 1 {
            continue
        }
        ans += counts[i]-1
        counts[i-1] += counts[i]-1
    }
    return ans
}
```



### 第3题

（反人类表述啊~）

有一张`transactions`表长这样：

| Name   | Type     |
| ------ | -------- |
| amount | int      |
| date   | datetime |

这表存了一个人的消费记录，当然，消费记录的总和可以为负数（就权当信用卡了），同时你也可以向账户里充值，充值金额为正数，消费金额为负数。我们要统计2022全年你的消费总额。

但是，没那么简单，你每次去消费要停车，每次停车要5元，但是的但是，停车费是可以免的，规则如下：

- 单次消费满50元，免停车费；
- 本月消费总额达1000元（$\ge 1000$），本月所有的停车费全免；
- 本月消费次数达5次（$\ge 5$），本月所有的停车费全免；

#### 思路

稍稍考虑以下，感觉很简单，只要判断本次要不要交停车费就修行了，怎么判断呢，我们有函数`IF`：

```sql
IF(<condition>, <if-true-return>, <if-false-return>)
```

剩下的就很简单咯，直接看代码：

```sql
SELECT sum(IF(
    t1.amount>0 or
    t1.amount<=-50 or
    (
        SELECT count(t2.amount) 
	    FROM transactions as t2
    	WHERE t2.amount<0 and YEAR(t2.date)=YEAR(t1.date) and MONTH(t2.date)=MONTH(t1.date)
    )>=5 or
    (
        SELECT sum(t2.amount) 
	    FROM transactions as t3
    	WHERE t3.amount<0 and YEAR(t3.date)=YEAR(t1.date) and MONTH(t3.date)=MONTH(t1.date)
    )<=-1000
    , t1.amount, t1.amount-5))
FROM transactions as t1
WHERE t1.date>='2022-01-01' and t1.date <'2023-01-01';
```

