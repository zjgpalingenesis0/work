# 54. 螺旋矩阵

- LeetCode：[54. 螺旋矩阵](https://leetcode.cn/problems/spiral-matrix/)
- 难度：中等

## 解题思路

按照顺时针方向一圈一圈读取矩阵：

```text
向右读取上边
向下读取右边
向左读取下边
向上读取左边
```

每读取完一条边，就把这条边从尚未处理的矩阵范围中排除。

使用四个变量表示当前还没有读取的矩形边界：

```text
top：上边界
bottom：下边界
left：左边界
right：右边界
```

初始状态：

```text
top = 0
bottom = matrix.length - 1
left = 0
right = matrix[0].length - 1
```

## 用到的数据结构及原因

### 结果数组 `result`

按照螺旋顺序依次保存访问到的元素。

题目要求返回所有元素组成的数组，因此必须使用一个结果数组。除最终结果外，不需要额外记录每个位置是否访问过。

### 四个边界变量

- `top`：当前未访问区域最上面一行。
- `bottom`：当前未访问区域最下面一行。
- `left`：当前未访问区域最左边一列。
- `right`：当前未访问区域最右边一列。

使用边界变量的原因：

1. 可以直接确定每个方向应该遍历到哪里。
2. 访问完一条边后，只需要收缩对应边界。
3. 不需要额外创建与矩阵同样大小的 `visited` 数组。

## 用到的方法及原因

### 边界模拟

按照题目要求的顺时针方向，模拟每一圈的四段移动。

每一圈依次执行：

```text
1. 遍历上边：从 left 到 right
2. 遍历右边：从 top 到 bottom
3. 遍历下边：从 right 到 left
4. 遍历左边：从 bottom 到 top
```

选择边界模拟的原因：

1. 移动方向固定，适合直接模拟。
2. 每个元素只访问一次。
3. 四个边界可以避免重复访问。

## 每走完一条边如何收缩

### 走完上边

```text
top++
```

表示最上面一行已经处理完。

### 走完右边

```text
right--
```

表示最右边一列已经处理完。

### 走完下边

```text
bottom--
```

表示最下面一行已经处理完。

### 走完左边

```text
left++
```

表示最左边一列已经处理完。

## 过程示例

矩阵：

```text
1  2  3
4  5  6
7  8  9
```

第一圈：

```text
向右：1、2、3
向下：6、9
向左：8、7
向上：4
```

此时外圈处理完毕，只剩中心：

```text
5
```

最终结果：

```text
[1, 2, 3, 6, 9, 8, 7, 4, 5]
```

## 为什么需要重复检查边界

不是每一圈都一定有完整的四条边。

例如只剩一行：

```text
1  2  3  4
```

走完上边并执行 `top++` 后：

```text
top > bottom
```

说明没有下边可以继续遍历。如果仍然遍历下边，就会把同一行重复加入结果。

同样，如果只剩一列，走完右边并执行 `right--` 后，可能出现：

```text
left > right
```

这时不能再遍历左边。

因此：

- 遍历下边前，要检查 `top <= bottom`。
- 遍历左边前，要检查 `left <= right`。

## 伪代码

```text
创建空结果数组
初始化 top、bottom、left、right

当 top <= bottom 并且 left <= right：
    从 left 到 right 遍历 top 行
    top 向下移动

    从 top 到 bottom 遍历 right 列
    right 向左移动

    如果 top <= bottom：
        从 right 到 left 遍历 bottom 行
        bottom 向上移动

    如果 left <= right：
        从 bottom 到 top 遍历 left 列
        left 向右移动

返回结果数组
```

## JavaScript 代码

```js
/**
 * @param {number[][]} matrix
 * @return {number[]}
 */
var spiralOrder = function (matrix) {
  const result = [];

  let top = 0;
  let bottom = matrix.length - 1;
  let left = 0;
  let right = matrix[0].length - 1;

  while (top <= bottom && left <= right) {
    // 从左到右遍历上边
    for (let column = left; column <= right; column++) {
      result.push(matrix[top][column]);
    }
    top++;

    // 从上到下遍历右边
    for (let row = top; row <= bottom; row++) {
      result.push(matrix[row][right]);
    }
    right--;

    // 从右到左遍历下边
    if (top <= bottom) {
      for (let column = right; column >= left; column--) {
        result.push(matrix[bottom][column]);
      }
      bottom--;
    }

    // 从下到上遍历左边
    if (left <= right) {
      for (let row = bottom; row >= top; row--) {
        result.push(matrix[row][left]);
      }
      left++;
    }
  }

  return result;
};
```

## 复杂度分析

假设矩阵有 `m` 行、`n` 列：

- 时间复杂度：`O(m × n)`。每个元素恰好访问一次。
- 额外空间复杂度：`O(1)`。除返回结果外，只使用四个边界变量。
- 返回结果需要 `O(m × n)` 空间，但通常不计入额外空间复杂度。

## 容易出错的地方

### 1. 下边和左边没有进行边界检查

每完成上边和右边后，未访问区域可能已经为空。遍历下边和左边前必须重新判断边界，否则单行或单列矩阵会出现重复元素。

### 2. 边界更新顺序错误

必须先遍历当前边，再收缩对应边界。例如遍历上边后才能执行 `top++`。

### 3. 方向中的循环条件写错

- 向右、向下使用 `<=`。
- 向左、向上使用 `>=`。

### 4. 行下标和列下标混淆

访问矩阵使用：

```text
matrix[row][column]
```

- 上边和下边固定 `row`，改变 `column`。
- 左边和右边固定 `column`，改变 `row`。

### 5. 主循环只判断一组边界

主循环应该同时保证行和列范围都有效：

```text
top <= bottom && left <= right
```

### 6. 使用访问标记增加不必要空间

使用方向数组和 `visited` 矩阵也可以完成，但需要 `O(m × n)` 额外空间。四个边界变量更直接。
