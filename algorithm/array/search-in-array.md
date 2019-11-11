[返回首页](../../README.md)

# 二维数组中的查找

题目：在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

思路：每次寻找范围内右上角数字。如果等于要查找的数字，则查找结束；如果大于，则提出所在列；如果小于，则剔除所在行

```
function find (matrix, num) {
    if (!matrix || matrix.length <= 0 || matrix[0].length <= 0) {
        return false
    }
    let rows = matrix.length, cols = matrix[0].length
    let row = 0, col = cols - 1
    while (row < rows && col >= 0) {
        if (matrix[row][col] === num) {
            return true
        } else if (matrix[row][col] > num) {
            col--
        } else {
            row++
        }
    }
    return false
}
```