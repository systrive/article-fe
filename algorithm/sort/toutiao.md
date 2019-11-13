[返回首页](../../README.md)

* <a href="#reorder">重新排序：奇数序号降序，偶数序号升序</a>
* <a href="#2-keys-keyboard">只有2个键的键盘</a>

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