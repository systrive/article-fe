[返回首页](../../README.md)

# 贪心算法

>> 贪心法又称贪心算法、贪婪算法，在对问题求解时，总是做出在当前看来最好的选择。

## 算法题一：买卖股票的最佳时机

LeetCode：https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/

描述：给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。

设计一个算法来计算你所能获取的最大利润。你可以尽可能地完成更多的交易（多次买卖一支股票）。

注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

示例 1:

输入: [7,1,5,3,6,4]
输出: 7
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
     随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6-3 = 3 。
示例 2:

输入: [1,2,3,4,5]
输出: 4
解释: 在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
     注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。
     因为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。
示例 3:

输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。

思路：贪心算法，只要明天比今天股价高，就今天买进，明天卖出

```
var maxProfit = function(prices) {
    if (!prices || prices.length < 2) {
        return 0
    }
    let result = 0, len = prices.length
    for (let i = 1; i < len; i++) {
        if (prices[i - 1] < prices[i]) {
            result += prices[i] - prices[i - 1]
        }
    }
    return result
}
```


## 算法二：买股票最佳时机

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

思路：最大子和问题

复杂度：时间复杂度 O(n)

```
var maxProfit = function(prices) {
    if (!prices || prices.length < 2) {
        return 0
    }
    let last = 0, profit = 0, len = prices.length
    for (let i = 0; i < len - 1; i++) {
        last = Math.max(0, last + prices[i + 1] - prices[i])
        profit = Math.max(profit, last)
    }
    return profit
}
```