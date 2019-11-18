[返回首页](../../README.md)

# 插入区间

LeetCode：https://leetcode-cn.com/problems/insert-interval/

```
var insert = function(intervals, newInterval) {
    if (!intervals || intervals.length <= 0) {
        return [newInterval]
    }
    if (!newInterval || newInterval.length <= 0) {
        return intervals
    }
    let i = 0, start = newInterval[0], end = newInterval[1], result = [], len = intervals.length
    while (i < len && intervals[i][1] < start) {
        result.push(intervals[i++])
    }
    while (i < len && intervals[i][0] <= end) {
        start = intervals[i][0] < start ? intervals[i][0] : start
        end = intervals[i][1] > end ? intervals[i][1] : end
        i++
    }
    result.push([start, end])
    while (i < len) {
        result.push(intervals[i++])
    }

    return result
};
```