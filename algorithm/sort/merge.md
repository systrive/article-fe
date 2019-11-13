[返回首页](../../README.md)

# 合并区间

LeetCode：https://leetcode-cn.com/problems/merge-intervals/

描述：
给出一个区间的集合，请合并所有重叠的区间。

示例 1:

输入: [[1,3],[2,6],[8,10],[15,18]]
输出: [[1,6],[8,10],[15,18]]
解释: 区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].
示例 2:

输入: [[1,4],[4,5]]
输出: [[1,5]]
解释: 区间 [1,4] 和 [4,5] 可被视为重叠区间。

思路：将 intervals 按第一个元素大小进行排序

复杂度：时间复杂度 O(nlogn)，空间复杂度 O(1)

```
var merge = function(intervals) {
    if (!intervals || intervals.length <= 1) {
        return intervals
    }
    intervals.sort((a, b) => {
        return a[0] - b[0]
    })
    let len = intervals.length
    let start = intervals[0][0], end = intervals[0][1], result = []
    for (let i = 1; i < len; i++) {
        console.log(intervals[i][0] > end)
        if (intervals[i][0] > end) {
            result.push([start, end])
            start = intervals[i][0]
            end = intervals[i][1]
        } else {
            end = intervals[i][1] > end ? intervals[i][1] : end
        }
    }
    result.push([start, end])
    return result
}
```