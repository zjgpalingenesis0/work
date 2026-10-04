# 5. 最长回文子串

- LeetCode：[5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)
- 难度：中等

## 解题思路

回文串从左向右读和从右向左读相同，例如 `aba`、`abba`。

每一个回文串都有一个中心，因此可以从中心向左右两边扩展：

- 如果左右字符相同，就继续向外扩展。
- 如果左右字符不同，或者指针越界，就停止扩展。
- 每次扩展结束后，记录当前找到的回文串范围。

回文串有两种中心：

1. 奇数长度回文串只有一个中心字符，例如 `aba`，中心是 `b`。
2. 偶数长度回文串的中心位于两个字符之间，例如 `abba`，中心位于两个 `b` 之间。

所以遍历每个位置时，都要尝试两种扩展方式：

```text
奇数长度：left = i，right = i
偶数长度：left = i，right = i + 1
```

## 用到的数据结构及原因

### 字符串和下标

这道题不需要 `Map`、`Set` 或数组等额外数据结构，只需要：

- 原字符串 `s`。
- `start`：目前最长回文子串的起始下标。
- `end`：目前最长回文子串的结束下标。
- `left`、`right`：从中心向两侧扩展的指针。

使用下标记录范围的原因：

1. 扩展过程中只需要比较字符，不需要反复创建新字符串。
2. 只记录最长结果的起点和终点，额外空间是常数级别。
3. 全部检查完成后，只需要截取一次字符串就能得到答案。

## 用到的方法及原因

### 中心扩展法

把每个字符以及每两个相邻字符之间的位置当作回文中心，然后向两边比较。

选择中心扩展法的原因：

1. 回文串天然具有左右对称的特点，这种方法与题目的性质直接对应。
2. 不需要复杂的数据结构。
3. 相比动态规划，空间复杂度可以从 `O(n²)` 降到 `O(1)`。
4. 实现直观，适合先掌握这道题的核心规律。

### `s.slice(start, end)`

用于截取最终的最长回文子串。

`slice` 的结束位置不包含在结果中，所以如果最长回文串范围是闭区间 `[start, end]`，需要写成：

```text
s.slice(start, end + 1)
```

## 伪代码

```text
如果字符串长度小于 2：
    直接返回字符串

start = 0
end = 0

定义 expand(left, right)：
    当 left 和 right 没有越界，并且两边字符相同：
        left 向左移动
        right 向右移动

    此时指针已经多移动了一步
    当前回文范围是 left + 1 到 right - 1

    如果当前回文比已记录的回文更长：
        更新 start 和 end

遍历字符串中的每个下标 i：
    expand(i, i)       // 检查奇数长度回文串
    expand(i, i + 1)   // 检查偶数长度回文串

返回 start 到 end 范围内的字符串
```

## JavaScript 代码

```js
/**
 * @param {string} s
 * @return {string}
 */
var longestPalindrome = function (s) {
  if (s.length < 2) {
    return s;
  }

  let start = 0;
  let end = 0;

  const expand = (left, right) => {
    while (
      left >= 0 &&
      right < s.length &&
      s[left] === s[right]
    ) {
      left--;
      right++;
    }

    const currentStart = left + 1;
    const currentEnd = right - 1;

    if (currentEnd - currentStart > end - start) {
      start = currentStart;
      end = currentEnd;
    }
  };

  for (let i = 0; i < s.length; i++) {
    expand(i, i);
    expand(i, i + 1);
  }

  return s.slice(start, end + 1);
};
```

## 复杂度分析

- 时间复杂度：`O(n²)`。一共有 `O(n)` 个中心，每个中心最坏需要向两边扩展 `O(n)` 次。
- 空间复杂度：`O(1)`。只使用了有限个下标变量，不计算最终返回的字符串。

## 容易出错的地方

### 1. 只检查一种中心

只调用 `expand(i, i)` 会漏掉 `abba` 这种偶数长度的回文串。奇数和偶数两种中心都必须检查。

### 2. 扩展结束后的边界多走了一步

循环结束时，`left` 和 `right` 已经指向回文串外侧，因此真正的范围是：

```text
[left + 1, right - 1]
```

### 3. `slice` 不包含结束下标

如果记录的 `end` 是回文串最后一个字符的下标，截取时必须传入 `end + 1`。

### 4. 忽略偶数长度为零的情况

调用 `expand(i, i + 1)` 时，相邻字符可能一开始就不同。此时不会形成更长回文串，不应错误更新结果。
