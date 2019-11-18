[返回首页](../../README.md)

* <a href="#reorder">重新排序：奇数序号降序，偶数序号升序</a>
* <a href="#2-keys-keyboard">只有2个键的键盘</a>
* <a href="#best-time-to-buy-and-sell-stock">买卖股票最佳时机</a>
* <a href="#max-diff">数组最大差值：给定一个整数数组，找出两个下标，要求：后面下标所指的数减前面下标所指的数之差最大</a>
* <a href="#max-points-on-a-line">直线上最多的点数</a>（待补充）
* <a href="#generate-parentheses">括号生成</a>
* <a href="#find-first-and-last-position-of-element-in-sorted-array">在排序数组中查找元素的第一个和最后一个位置</a>
* <a href="#find-first-large-num-in-array">找出这个数组中每一个数右边的第一个比它大的数</a>（待补充）
* <a href="#powx-n">Pow(x,n)</a>

# <a id="reorder">重新排序</a>

描述：给出一个数组，奇数序号降序，偶数序号升序，要求重排成重小到大的数组，时间复杂度为O(n)

思路一：定义一个新的数组 result，定义 i、j，i 表示从小到大偶数序号，j表示从大到小奇数序号，比较 nums[i] 与 nums[j] 的值，将较小的值放到 result。

复杂度：时间复杂度 O(n)，空间复杂度 O(n)

```
function reOrder (nums) {
    if (!nums || nums.length <= 1) {
        return nums
    }
    let len = nums.length, j = (len >> 1) * 2 - 1, i = 0, result = []
    while (j > 0 && i < len) {
        while (i < len && nums[i] <= nums[j]) {
            result.push(nums[i])
            i += 2
        }
        while (j > 0 && nums[j] <= nums[i]) {
            result.push(nums[j])
            j -= 2
        }
    }
    while (i < len) {
        result.push(nums[i])
        i += 2
    }
    while (j > 0) {
        result.push(nums[j])
        j -= 2
    }

    return result
}
```


思路二：先将奇数序号排序，使得奇数序号增序。相邻两个元素进行比较，较大的往后放（冒泡排序方法）

复杂度：时间 O(n)，空间复杂度 O(1)


```
function reOrder (nums) {
    if (!nums || nums.length <= 1) {
        return nums
    }
    let len = nums.length, j = (len >> 1) * 2 - 1, i = 1
    while (j > i) {
        let tmp = nums[j]
        nums[j] = nums[i]
        nums[i] = tmp
        j -= 2
        i += 2
    }
    for (let k = 0; k <len - 1; k++) {
        if (nums[k] > nums[k + 1]) {
            let tmp = nums[k]
            nums[k] = nums[k + 1]
            nums[k + 1] = tmp
        }
    }
    return nums
}
```

# <a id="2-keys-keyboard">只有2个键盘</a>

### 描述：

最初在一个记事本上只有一个字符 'A'。你每次可以对这个记事本进行两种操作：

Copy All (复制全部) : 你可以复制这个记事本中的所有字符(部分的复制是不允许的)。
Paste (粘贴) : 你可以粘贴你上一次复制的字符。
给定一个数字 n 。你需要使用最少的操作次数，在记事本中打印出恰好 n 个 'A'。输出能够打印出 n 个 'A' 的最少操作次数。

示例 1:

输入: 3
输出: 3
解释:
最初, 我们只有一个字符 'A'。
第 1 步, 我们使用 Copy All 操作。
第 2 步, 我们使用 Paste 操作来获得 'AA'。
第 3 步, 我们使用 Paste 操作来获得 'AAA'。
说明:

n 的取值范围是 [1, 1000] 。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/2-keys-keyboard

### 思路
求最小因数分解之和。由于只能复制全部，每一次复制可以粘贴多次，所以结果是：1*(a1+1)(a2+1)(a3+1)......其中，+1 表示复制全部一次,a1/a2/a3代表第一次、二次、三次粘贴的次数，n = (a1+1)(a2+1)(a3+1)...(ak+1)，所以我们应当对总数做因式分解。

比如，30 = 2*3*5

分解后的步骤：

* 复制-粘贴(1次)（AA）
* 复制-粘贴(2次)（AAAAAA）
* 复制-粘贴(4次)（AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA）

就算交换顺序 3*5*2 得到的结果也是一样的

```
var minSteps = function(n) {
    if (n <= 1) {
        return 0
    }
    let result = 0, i = 2
    while (n > 1 && i <= n) {
        while (n % i === 0) {
            result += i
            n = n / i
        }
        i++
    }
    return result
}
```

# <a id="best-time-to-buy-and-sell-stock">买卖股票最佳时机</a>

### 描述：

给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。

如果你最多只允许完成一笔交易（即买入和卖出一支股票），设计一个算法来计算你所能获取的最大利润。

注意你不能在买入股票前卖出股票。

示例 1:

输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格。
示例 2:

输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock

### 思路：

动态规划：

* 找到状态转移方程：res = max(res, prices[i] - min_val)。res 为前 i 天最大收益，min_val 为前 i 天的最小值
* 初始化量：res = 0, min_val = prices[0]

```
var maxProfit = function(prices) {
    if (!prices || prices.length < 2) {
        return 0
    }
    let min_val = prices[0], res = 0, len = prices.length
    for (let i = 1; i < len; i++) {
        min_val = Math.min(min_val, prices[i])
        res = Math.max(res, prices[i] - min_val)
    }
    return res
}
```

# <a id="max-diff">数组最大差值</a>

### 描述：给定一个整数数组，找出两个下标，要求：后面下标所指的数减前面下标所指的数之差最大

### 思路：

与上一题思路一样，只不过需要保存最小和最大指标，定义 minCurIndex = 0、maxCurIndex = 0 表示当前差值最大的下标

```
var maxProfit = function(prices) {
    if (!prices || prices.length < 2) {
        return []
    }

    let minCurIndex = 0, minIndex = 0, maxCurIndex = 0, len = prices.length, res = 0
    for (let i = 0; i < len; i++) {
        minIndex = prices[minIndex] < prices[i] ? minIndex : i
        if (prices[i] - prices[minIndex] > res) {
            console.log(i, minIndex)
            minCurIndex = minIndex
            maxCurIndex = i
            res = prices[i] - prices[minIndex]
        }
    }
    return [minCurIndex, maxCurIndex]
}
```


# <a id="max-points-on-a-line">直线上最多的点数</a>

### 描述：
给定一个二维平面，平面上有 n 个点，求最多有多少个点在同一条直线上。

示例 1:

输入: [[1,1],[2,2],[3,3]]
输出: 3
解释:
^
|
|        o
|     o
|  o  
+------------->
0  1  2  3  4
示例 2:

输入: [[1,1],[3,2],[5,3],[4,1],[2,3],[1,4]]
输出: 4
解释:
^
|
|  o
|     o        o
|        o
|  o        o
+------------------->
0  1  2  3  4  5  6

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/max-points-on-a-line


# <a id="generate-parentheses">括号生成</a>

### 描述：

给出 n 代表生成括号的对数，请你写出一个函数，使其能够生成所有可能的并且有效的括号组合。

例如，给出 n = 3，生成结果为：

[
  "((()))",
  "(()())",
  "(())()",
  "()(())",
  "()()()"
]

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/generate-parentheses

### 思路：

#### 思路一：

暴力解法：缓存第 n-1 结果，遍历第 n-1 结果，再对每个字符串进行遍历，设置下标 m1 和 m2（m1<=m2），在 m1 之前加 '(' 在 m2 之后加 ')'，再对结果去重。

```
var generateParenthesis = function(n) {
    if (n < 1) {
        return []
    }
    let res = ['()']
    for (let i = 1; i < n; i++) {
        let len = res.length, l = []
        for (let j = 0; j < len; j++) {
            let val = res[j], tmpLen = val.length
            for (let m1 = 0; m1 < tmpLen; m1++) {
                for (let m2 = 0; m2 < tmpLen && m1 <= m2; m2++) {
                    let valTmp = `${val.slice(0, m1)}(${val.slice(m1, m2)})${val.slice(m2)}`
                    l.push(valTmp)
                }
            }
        }
        res = Array.from(new Set(l))
    }
    return res
}
```

#### 思路二：

递归回调：openP 表示剩余 '(' 的数量，closeP 表示剩余 ')' 的数量

* openP > 0 时，加 '('
* openP < closeP 时，加 ')'

```
var generateParenthesis = function (n) {
    let result = []
    function bfs (tmp, openP, closeP) {
        console.log('tmp', tmp, openP, closeP)
        if (openP == 0 && closeP == 0) {
            return result.push(tmp)
        }
        if (openP > 0) {
            bfs(tmp + '(', openP - 1, closeP)
        }
        if (openP < closeP) {
            bfs(tmp + ')', openP, closeP - 1)
        }
    }
    bfs('', n, n)
    return result
}
```

# <a id="find-first-and-last-position-of-element-in-sorted-array">在排序数组中查找元素的第一个和最后一个位置</a>

### 描述：

给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。

你的算法时间复杂度必须是 O(log n) 级别。

如果数组中不存在目标值，返回 [-1, -1]。

示例 1:

输入: nums = [5,7,7,8,8,10], target = 8
输出: [3,4]
示例 2:

输入: nums = [5,7,7,8,8,10], target = 6
输出: [-1,-1]

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array


### 思路：

二分查找法：

```
var searchRange = function(nums, target) {
    if (!nums || nums.length <= 0) {
        return [-1, -1]
    }
    let len = nums.length, s = 0, e = len - 1, start = -1, end = -1
    while (s >= 0 && e < len && s <= e) {
        let mid = Math.floor((e - s + 1) / 2) + s
        if (nums[mid] === target) {
            start = mid
            end = mid
            while (start > 0 && nums[start - 1] === target) {
                start--
            }
            while (end < len + 1 && nums[end + 1] === target) {
                end++
            }
            return [start, end]
        } else if (nums[mid] > target) {
            e = mid - 1
        } else {
            s = mid + 1
        }
    }
    return [start, end]
}
```


# <a id="find-first-large-num-in-array">找出这个数组中每一个数右边的第一个比它大的数</a>

### 思路：


# <a id="powx-n">Pow(x,n)</a>

### 描述：
实现 pow(x, n) ，即计算 x 的 n 次幂函数。

示例 1:

输入: 2.00000, 10
输出: 1024.00000
示例 2:

输入: 2.10000, 3
输出: 9.26100
示例 3:

输入: 2.00000, -2
输出: 0.25000
解释: 2-2 = 1/22 = 1/4 = 0.25
说明:

-100.0 < x < 100.0
n 是 32 位有符号整数，其数值范围是 [−231, 231 − 1] 。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/powx-n

