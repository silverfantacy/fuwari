---
title: JavaScript 陣列搜尋：indexOf() 之外的方法 O(log n) 
published: 2024-05-23
description: '探索 JavaScript 中高效的二分搜尋法，了解其原理、步驟與在陣列搜尋中的應用。'
image: 'https://media.virtualxnews.com/2024/05/5023bafc87390133866bfd17f083b3a2.png'
tags: [O(log n), 二分搜尋, 陣列, 搜尋, JavaScript, 時間複雜度, 演算法]
category: '演算法'
draft: false 
---

近期面試中，我遇到了一道關於 JavaScript 陣列搜尋的題目。雖然題目本身看似簡單，但在面試的緊張氣氛下加上沒有刷 LeetCode，要快速給出最佳解法並不容易。

## 面試題目：

給定一個升序的整數陣列和一個目標值，找出目標值在陣列中的索引位置（從 0 開始）。如果目標值不在陣列中，則返回 -1。

### 一開始的想法：

**indexOf()：**  
 - 優點：簡單直接，程式碼簡潔。
 - 首先想到的是 JavaScript 內建的 `indexOf()` 方法，它可以直接查找目標值並返回索引。
 - 缺點：對於大型陣列，效率較低（時間複雜度為 O(n)）。

```javascript
// indexOf()
const numbers = [1, 4, 6, 7, 9, 10];
const index = numbers.indexOf(9);
console.log(index); // 輸出：4
```

### 面試需要的答案：
**二分搜尋法：**
 - 由於陣列已排序，可以利用二分搜尋法來提高效率。
 - 優點：在已排序陣列中極為高效（時間複雜度為 O(log n)）。
 - 缺點：實作稍為複雜，需要仔細處理邊界條件。

```javascript
// 二分搜尋法
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (arr[mid] === target) {
      return mid; // 找到目標，返回索引
    } else if (arr[mid] < target) {
      left = mid + 1; // 目標在右半邊
    } else {
      right = mid - 1; // 目標在左半邊
    }
  }

  return -1; // 找不到目標
}

const numbers = [1, 4, 6, 7, 9, 10];
const index = binarySearch(numbers, 9);

console.log(index); // 輸出：4
```

> 本文由 gpt-4o 輔助撰寫。