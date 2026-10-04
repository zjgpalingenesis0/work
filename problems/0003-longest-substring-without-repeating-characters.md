# 3. 无重复字符的最长子串

- LeetCode：[3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)
- 难度：中等

## 解题思路

使用滑动窗口维护一个“没有重复字符的连续子串”。

- `left` 表示窗口左边界。
- `right` 表示窗口右边界，并从左向右扫描字符串。
- 遇到重复字符时，将 `left` 移到该字符上一次出现位置的后一位。
- 每轮扫描后，用当前窗口长度更新最大值。

左边界只能向右移动，不能后退。因此更新左边界时，需要取当前 `left` 和“重复字符上次位置加一”中的较大值。

## 用到的数据结构及原因

### `Map`（哈希表）

使用 `Map` 保存：

```text
字符 -> 该字符最近一次出现的下标
```

选择 `Map` 的原因：

1. 可以快速判断一个字符之前是否出现过。
2. 可以快速取得字符最近一次出现的位置。
3. 查询和更新的平均时间复杂度都是 `O(1)`。
4. 遇到重复字符时，左边界可以直接跳到正确位置，不需要逐个删除窗口内的字符。

## 用到的方法及原因

### `map.has(key)`

判断当前字符是否已经出现过。

### `map.get(key)`

取得当前字符最近一次出现的下标，用来计算窗口新的左边界。

### `map.set(key, value)`

记录或更新当前字符最近一次出现的下标。

### `Math.max(a, b)`

有两个用途：

1. 更新 `left` 时，保证窗口左边界不会后退。
2. 更新答案时，保留目前找到的最大窗口长度。

## 伪代码

```text
创建 Map，记录字符最近一次出现的位置
left = 0
maxLength = 0

让 right 从 0 遍历到字符串末尾：
    char = 当前字符

    如果 char 曾经出现过：
        left = left 与 char 上次位置加一中的较大值

    更新 char 最近一次出现的位置为 right
    当前窗口长度 = right - left + 1
    更新最大长度

返回最大长度
```

## JavaScript 代码

```js
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLongestSubstring = function (s) {
  const lastIndex = new Map();
  let left = 0;
  let maxLength = 0;
  
  for (let right = 0; right < s.length; right++) {
    const char = s[right];
    // 如果出现过
    if (lastIndex.has(char)) {
      left = Math.max(left, lastIndex.get(char) + 1);
    }

    lastIndex.set(char, right);
    maxLength = Math.max(maxLength, right - left + 1);
  }

  return maxLength;
};
```

## 复杂度分析

- 时间复杂度：`O(n)`。右指针只遍历字符串一次。
- 空间复杂度：`O(k)`。`k` 是字符串中不同字符的数量。

## 容易出错的地方

更新左边界时不能直接写成“字符上次出现的位置加一”。旧字符可能已经不在当前窗口中，这样会导致 `left` 后退。

例如字符串 `abba`，扫描到最后一个 `a` 时，第一个 `a` 已经在窗口外，所以必须使用：

```js
left = Math.max(left, lastIndex.get(char) + 1);
```
