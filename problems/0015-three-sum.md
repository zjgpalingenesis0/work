# 15. 三数之和

- LeetCode：[15. 三数之和](https://leetcode.cn/problems/3sum/)
- 难度：中等

## 先理解题意

从数组中找出所有满足下面条件的三元组：

```text
nums[i] + nums[j] + nums[k] = 0
```

三个元素必须来自三个不同位置，并且答案中不能包含重复的三元组。

例如：

```text
nums = [-1, 0, 1, 2, -1, -4]
```

符合条件的不同三元组是：

```text
[-1, -1, 2]
[-1, 0, 1]
```

本题返回的是数字组成的三元组，不是数字下标。

## 解题思路

先对数组进行升序排序：

```text
[-4, -1, -1, 0, 1, 2]
```

然后依次固定第一个数字 `nums[i]`。剩余问题变成：

> 在 `i` 右边寻找两个数字，使它们的和等于 `-nums[i]`。

使用两个指针：

- `left = i + 1`，指向剩余部分最小的数字。
- `right = nums.length - 1`，指向剩余部分最大的数字。

计算：

```text
sum = nums[i] + nums[left] + nums[right]
```

根据结果移动指针：

- `sum < 0`：总和太小，需要让 `left` 向右移动，使总和变大。
- `sum > 0`：总和太大，需要让 `right` 向左移动，使总和变小。
- `sum === 0`：找到一个答案，记录后同时移动两个指针，并跳过重复数字。

## 用到的数据结构及原因

### 排序后的输入数组

直接对 `nums` 进行升序排序。

排序的作用：

1. 双指针可以根据总和大小决定移动方向。
2. 相同数字会排列在一起，便于跳过重复元素。
3. 当固定数字已经大于 `0` 时，可以提前结束搜索。

### 结果数组 `result`

保存所有不重复的三元组：

```text
result.push([nums[i], nums[left], nums[right]])
```

### 三个下标

- `i`：当前固定的第一个数字。
- `left`：从固定位置右侧向右移动。
- `right`：从数组末尾向左移动。

## 用到的方法及原因

### 数字升序排序

JavaScript 的 `sort()` 默认按照字符串规则排序，因此必须传入数字比较函数：

```js
nums.sort((a, b) => a - b);
```

### 双指针

排序后：

- 左指针右移，数字不会变小，所以总和会趋向增大。
- 右指针左移，数字不会变大，所以总和会趋向减小。

这样可以在线性时间内寻找固定 `nums[i]` 对应的所有两数之和。

### 去重

本题最容易出错的是重复答案，需要在三个位置去重。

#### 固定数字 `i` 去重

如果当前固定数字与上一个相同，就跳过：

```text
i > 0 并且 nums[i] === nums[i - 1]
```

否则相同的第一个数字会产生相同三元组。

#### `left` 去重

找到答案并让 `left` 右移后，继续跳过与前一个相同的数字。

#### `right` 去重

找到答案并让 `right` 左移后，继续跳过与后一个相同的数字。

## 过程示例

排序后：

```text
[-4, -1, -1, 0, 1, 2]
```

固定第一个 `-1`：

```text
i = 1，nums[i] = -1
left = 2，nums[left] = -1
right = 5，nums[right] = 2
```

总和：

```text
-1 + (-1) + 2 = 0
```

得到：

```text
[-1, -1, 2]
```

移动并去重后：

```text
left 指向 0
right 指向 1
```

总和：

```text
-1 + 0 + 1 = 0
```

得到：

```text
[-1, 0, 1]
```

下一次外层循环遇到第二个 `-1` 时，因为它与上一个固定数字相同，所以直接跳过，避免重复答案。

## 伪代码

```text
将 nums 升序排序
创建结果数组 result

从左到右枚举固定位置 i：
    如果 nums[i] > 0：
        结束循环

    如果 nums[i] 与前一个固定数字相同：
        跳过当前 i

    left = i + 1
    right = 数组末尾

    当 left < right：
        sum = nums[i] + nums[left] + nums[right]

        如果 sum < 0：
            left 向右移动
        否则如果 sum > 0：
            right 向左移动
        否则：
            将三元组加入 result
            left 向右移动
            right 向左移动
            跳过 left 和 right 遇到的重复数字

返回 result
```

## JavaScript 代码

```js
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
var threeSum = function (nums) {
  nums.sort((a, b) => a - b);

  const result = [];

  for (let i = 0; i < nums.length - 2; i++) {
    if (nums[i] > 0) {
      break;
    }

    if (i > 0 && nums[i] === nums[i - 1]) {
      continue;
    }

    let left = i + 1;
    let right = nums.length - 1;

    while (left < right) {
      const sum = nums[i] + nums[left] + nums[right];

      if (sum < 0) {
        left++;
      } else if (sum > 0) {
        right--;
      } else {
        result.push([nums[i], nums[left], nums[right]]);

        left++;
        right--;

        while (left < right && nums[left] === nums[left - 1]) {
          left++;
        }

        while (left < right && nums[right] === nums[right + 1]) {
          right--;
        }
      }
    }
  }

  return result;
};
```

## 复杂度分析

设数组长度为 `n`：

- 时间复杂度：`O(n²)`。排序需要 `O(n log n)`；外层固定一个数字，内层双指针总共线性移动，因此主要复杂度是 `O(n²)`。
- 额外空间复杂度：不计算返回结果时通常记为 `O(1)`，但 JavaScript 排序实现本身可能使用额外空间。

## 为什么 `nums[i] > 0` 时可以结束

数组已经升序排列。如果当前固定数字大于 `0`，那么它右边的两个数字也都大于或等于它，三个正数不可能相加得到 `0`。

注意只能在：

```text
nums[i] > 0
```

时结束，不能在 `nums[i] === 0` 时结束，因为 `[0, 0, 0]` 是合法答案。

## 容易出错的地方

### 1. 忘记数字排序比较函数

不能只写 `nums.sort()`，应该写：

```js
nums.sort((a, b) => a - b);
```

### 2. 固定数字去重方向错误

固定 `i` 时要与前一个数字比较：

```text
nums[i] === nums[i - 1]
```

不能因为它与后一个数字相同就直接跳过，否则可能漏掉 `[-1, -1, 2]`。

### 3. 找到答案后没有移动两个指针

记录答案后必须让 `left++`、`right--`，否则会一直处理同一组三个位置，形成死循环。

### 4. 只对固定数字去重

`left` 和 `right` 也可能产生重复组合。找到答案后要跳过两侧重复值。

### 5. 返回下标

第 1 题“两数之和”返回下标，但本题要求返回数字组成的三元组。

### 6. 使用 `left <= right`

三个位置必须不同，因此两个指针不能相遇，循环条件应该是：

```text
left < right
```

## 与第 1 题的关系

两数之和使用哈希表快速寻找补数。三数之和则先固定一个数字，把问题转换成：

```text
在有序数组中寻找两个数，使它们的和等于 -nums[i]
```

排序后的双指针既能降低复杂度，也更方便消除重复三元组。
