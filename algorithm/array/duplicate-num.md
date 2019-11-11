[返回首页](../../README.md)

数组是在程序设计中，为了处理方便，把具有相同类型的若干元素按有序的形式组织起来的一种形式。抽象地讲，数组即是有限个类型相同的元素的有序序列。若将此序列命名，那么这个名称即为数组名。组成数组的各个变量称为数组的分量，也称为数组的元素。而用于区分数组的各个元素的数字编号则被称为下标，若为此定义一个变量，即为下标变量。

## 题目一：判断是否存在重复元素

描述：在一个长度为 n 的数组里的所有数字都在 0~n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。例如，如果长度为 7 的数组 [2, 3, 1, 0, 2, 5, 3]，那么对应的输出是重复的数字 2 或者 3。

思路：重排这个数组。从头到尾依次扫描每个数字。当下标为 i 时，首先比较这个数字（m = number[i]）是不是等于 i，如果是，接着扫描下一个数字；如果不是，拿它和第m个数字进行比较。如果它和第m个数字相等，就找到了一个重复的数字（该数字在下标为 i 和 m 的位置出现了）；如果它和 m 不等，就把第 i 个数字和第 m 个数字交换，把 m 放到属于它的位置。接下来再重复比较、交换，指导我们发现一个重复的数字。

时间复杂度：O(n)，空间复杂度 O(1)

以 [2, 3, 1, 0, 2, 5, 3] 为例

第0次：[1, 3, 2, 0, 2, 5, 3]
第1次：[3, 1, 2, 0, 2, 5, 3]
第2次：[0, 1, 2, 3, 2, 5, 3]
第3次：nums[i] === nums[nums[i]] 返回 true

```
function duplicate (nums) {
    if (!nums || nums.length <= 0) {
        return false, undefined
    }
    let len = nums.length
    for (let i = 0; i < len; i++) {
        if (nums[i] < 0 || nums[i] >= len) {
            return false, undefined
        }
    }
    for (let i = 0; i < len; i++) {
        while (nums[i] !== i) {
            if (nums[i] === nums[nums[i]]) {
                return true, nums[i]
            }
            let tmp = nums[i]
            nums[i] = nums[tmp]
            nums[tmp] = tmp
        }
    }
    return false, undefined
}
```



## 题目二：不修改数组找出重复的数字

描述：在一个长度为 n 的数组里的所有数字都在 0~n-1 的范围内。所以数组中至少有一个数字是重复的。请找出数组中任意一个重复的数字，但不能修改数组。例如，如果长度为 7 的数组 [2, 3, 1, 0, 2, 5, 3]，那么对应的输出是重复的数字 2 或者 3。

思路一：看起来跟题目一类似。不能修改输入的数组，我们可以创建一个 n+1 的辅助数组，然后逐一把原数组每个数字复制到辅助数组中，这样很容易发现哪些数字重复，时间复杂度 O(n)、空间复杂度 O(n)。

思路二：把从 1~n 从直接的数字 m 分为两部分，前半部分 1~m，后面 m+1~n。如果 1~m 的数字超过 m，那么这一半的区间肯定包含重复的数字；否则，m+1~n 包含重复数字。继续把包含重复数字的区间一分为二，执照找到一个重复的数字。这个过程和二分查找法很相似，只是多了一步统计区间里的数字的数目。时间复杂度 O(nlogn) 、空间复杂度 O(1)。

```
function getDuplication (nums) {
    if (!nums || nums.length <= 0) {
        return -1
    }
    let len = nums.length, start = 0, end = len - 1
    for (let i = 0; i < len; i++) {
        if (nums[i] < 0 || nums[i] >= len) {
            return -1
        }
    }
    while (start <= end) {
        let mid = ((end - start) >> 1) + start

        let count = 0
        for (let i = 0; i < len; i++) {
            if (nums[i] >= start && nums[i] <= mid) {
                count++
            }
        }

        if (start === end) {
            if (count > 1) {
                return start
            } else {
                break
            }
        }
        if (count > mid - start + 1) {
            end = mid
        } else {
            start = mid + 1
        }
    }

    return -1
}
```