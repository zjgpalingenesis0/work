# 33. 搜索旋转排序数组

- LeetCode：[33. 搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/)
- 难度：中等

## 解题思路

普通升序数组可以直接使用二分查找，但旋转后的数组整体不再有序，例如：

```text
[4, 5, 6, 7, 0, 1, 2]
```

不过，它仍然保留一个重要性质：

> 对于任意中间位置 `mid`，`left...mid` 和 `mid...right` 中至少有一边是有序的。

因此，每轮二分可以分为两步：

1. 判断左半边还是右半边有序。
2. 判断 `target` 是否位于有序的那一边。

如果目标值在有序区间中，就保留这一半；否则保留另一半。这样每轮仍然能够排除一半搜索范围。

## 用到的数据结构及原因

### 数组和下标

这道题不需要额外的 `Map`、`Set` 或辅助数组，只需要保存三个下标：

- `left`：当前搜索范围的左边界。
- `right`：当前搜索范围的右边界。
- `mid`：当前搜索范围的中间位置。

使用下标的原因：

1. 输入数组支持通过下标直接访问元素。
2. 二分查找只需要不断缩小左右边界。
3. 不复制数组，也不需要寻找旋转点，空间复杂度为 `O(1)`。

## 用到的方法及原因

### 修改后的二分查找

普通二分查找直接根据 `nums[mid]` 与 `target` 的大小决定方向，但这里不能只比较这两个值，因为数组整体并不有序。

本题需要先判断哪一半有序：

```text
如果 nums[left] <= nums[mid]：
    左半边有序
否则：
    右半边有序
```

然后判断目标值是否落在有序半边的数值范围内。

选择二分查找的原因：

1. 题目要求 `O(log n)` 的时间复杂度。
2. 每轮都能利用有序的一半排除另一半搜索空间。
3. 不需要先查找旋转点，也不需要进行两次二分。

### `Math.floor()`

用于将左右边界的中间位置取整：

```text
mid = left + floor((right - left) / 2)
```

这种写法也避免了其他语言中直接计算 `left + right` 可能产生的整数溢出问题。

## 如何判断搜索方向

### 情况一：左半边有序

判断条件：

```text
nums[left] <= nums[mid]
```

如果目标值满足：

```text
nums[left] <= target < nums[mid]
```

说明目标值位于有序的左半边，应当令 `right = mid - 1`；否则搜索右半边。

### 情况二：右半边有序

如果左半边不是有序区间，那么右半边一定有序。

如果目标值满足：

```text
nums[mid] < target <= nums[right]
```

说明目标值位于有序的右半边，应当令 `left = mid + 1`；否则搜索左半边。

## 过程示例

在下面的数组中搜索 `0`：

```text
[4, 5, 6, 7, 0, 1, 2]
```

第一次：

```text
left = 0，mid = 3，right = 6
nums[mid] = 7
左半边 [4, 5, 6, 7] 有序
0 不在 [4, 7) 中，所以搜索右半边
```

第二次：

```text
left = 4，mid = 5，right = 6
nums[mid] = 1
左半边 [0, 1] 有序
0 在 [0, 1) 中，所以搜索左半边
```

第三次：

```text
left = 4，mid = 4，right = 4
nums[mid] = 0
找到目标值，下标为 4
```

## 伪代码

```text
left = 0
right = 数组长度 - 1

当 left <= right：
    计算 mid

    如果 nums[mid] 等于 target：
        返回 mid

    如果 nums[left] <= nums[mid]：
        左半边有序

        如果 target 位于 nums[left] 到 nums[mid] 之间：
            搜索左半边
        否则：
            搜索右半边
    否则：
        右半边有序

        如果 target 位于 nums[mid] 到 nums[right] 之间：
            搜索右半边
        否则：
            搜索左半边

没有找到时返回 -1
```

## JavaScript 代码

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number}
 */
var search = function (nums, target) {
  let left = 0;
  let right = nums.length - 1;

  while (left <= right) {
    const mid = left + Math.floor((right - left) / 2);

    if (nums[mid] === target) {
      return mid;
    }

    if (nums[left] <= nums[mid]) {
      // 左半边有序
      if (nums[left] <= target && target < nums[mid]) {
        right = mid - 1;
      } else {
        left = mid + 1;
      }
    } else {
      // 右半边有序
      if (nums[mid] < target && target <= nums[right]) {
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
  }

  return -1;
};
```

## 复杂度分析

- 时间复杂度：`O(log n)`。每轮都将搜索范围缩小一半。
- 空间复杂度：`O(1)`。只使用了有限个下标变量。

## 容易出错的地方

### 1. 只比较 `nums[mid]` 和 `target`

旋转数组整体无序，不能照搬普通二分查找。必须先确定哪一半有序。

### 2. 判断左半边有序时漏掉等号

应该使用：

```text
nums[left] <= nums[mid]
```

当搜索范围只剩一个元素时，`left` 和 `mid` 会相等。加上等号可以正确处理这种情况。

### 3. 区间条件两端的等号写错

因为前面已经判断过 `nums[mid] !== target`，所以左半边的目标范围是：

```text
nums[left] <= target < nums[mid]
```

右半边的目标范围是：

```text
nums[mid] < target <= nums[right]
```

### 4. 边界没有越过 `mid`

排除中间元素时必须使用：

```text
left = mid + 1
right = mid - 1
```

如果只赋值为 `mid`，可能导致搜索范围无法继续缩小，从而出现死循环。

### 5. 这套判断依赖元素互不相同

本题规定数组中的值互不相同。如果允许大量重复值，仅通过 `nums[left] <= nums[mid]` 有时无法确定哪一边真正有序，需要额外处理重复边界。
