[返回首页](../../README.md)

* <a href="#rand7()">rand5() 求出 rand7()</a>
* <a href="#best-time-to-buy-and-sell-stock">买卖股票最佳时期</a>


# <a id="rand7()">rand5() 求出 rand7()</a>

描述：一个函数：rand5() 可以等概率求出 1-5 中任意一个随机数，要求使用 rand5() 求出 rand7()。rand7是求1-7中任意一个随机数。

思路：rand5() 可以等概率求出 1-5，5*rand5() + (rand5() - 1) 可以等概率求出 5-29 的数

```
function rand7 () {
    let md
    do {
        md = 5 * rand5() + (rand5() - 1)
    } while (md >= 8 && md <= 14)
    return md - 7
}

```

# <a id="best-time-to-buy-and-sell-stock">买卖股票最佳时期</a>

LeetCode：https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/

### 描述：

给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。

如果你最多只允许完成一笔交易（即买入和卖出一支股票），设计一个算法来计算你所能获取的最大利润。

示例 1:

输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格。
示例 2:

输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。

### 思路：

```
var maxProfit = function(prices) {
    if (!prices || prices.length <= 1) {
        return 0
    }
    let profit = 0, max = 0, len = prices.length
    for (let i = 1; i < len; i++) {
        profit = Math.max(0, profit + prices[i] - prices[i - 1])
        max = Math.max(profit, max)
    }
    return max
}
```