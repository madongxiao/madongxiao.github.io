---
title: JavaScript数组排序详解
layout: post
categories: JavaScript
tags: 数组 排序 乱序
excerpt: 关于JavaScript中数组排序函数的介绍和应用
---
提到 JavaScript 中对数组进行排序操作，可能首先想到的就是 `Array.prototype.sort()` 这个函数，比如以下场景就比较常见：

```js
var arr = [3, 1, 2];

console.log(arr.sort());
// [1, 2, 3]
console.log(arr); // sort() 函数会修改原数组
// [1, 2, 3]

arr = ['c', 'b','B', 'a','A'];
arr.sort();
console.log(arr);
// ["A", "B", "a", "b", "c"]
```

和预想的一样，`sort()` 函数默认将数组元素升序排列，但是不要被上面的数字数组的排序结果迷惑，该函数并不是按照数字递增的方式排列的，而是按照元素的 **ASCII** 码或者 **Unicode** 码进行排序，比如字符 `a` 对应的 **ASCII** 码要比字符 `b` 的小，所以 `a` 排在 `b` 前面，同样字符 `A` 的比字符 `a` 的小，所以大写字母 `A` 会排在小写字母 `a` 前面；考虑以下情景：
```js
var arr = [1, 2, 11, 12];
arr.sort();
console.log(arr);
// [1, 11, 12, 2]
```

是不是有些和预想的不一样，这也验证了之前所说，并不是按照数字递增在排序，而是把数组中的数字类型的元素转换成字符，在拆分字符比较单个字符对应的字符码的大小；

## 比较函数

那么问题就来了，要按照数字递增方式排序，该怎么操作呢？其实这种情况早就被 `.sort()` 函数考虑到了，只是可能被大家忽略了，就是 `.sort()` 函数还能接受一个参数，叫做 `compareFunction`，顾名思义，就是 **比较函数**，由于该参数是一个函数，所以该函数又能接受两个参数，即比较的值，所以最终就是 `.sort(compareFunction(a, b))`；

关于这个 **比较函数**，存在如下规则：

- 如果 `compareFunction(a, b)` 返回值小于 0 ，那么 `a` 会被排列到 `b` 之前；
- 如果 `compareFunction(a, b)` 返回值等于 0 ，那么 `a` 和 `b` 的相对位置不变；
    - 备注： ECMAScript 标准并不保证这一行为，而且也不是所有浏览器都会遵守（例如 Mozilla 在 2003 年之前的版本）；
- 如果 `compareFunction(a, b)` 返回值大于 0 ，那么 `b` 会被排列到 `a` 之前；

> `compareFunction(a, b)` 必须总是对相同的输入返回相同的比较结果，否则排序的结果将是不确定的。

在使用它之前，先来看看函数里面的参数 `a, b` 是如何对应数组元素的：
```js
var arr = [2, 1, 4, 3];
arr.sort(function(a, b) {
    console.log(a, b);
})

// 1 2
// 4 1
// 3 4
```

可以发现，由于这里的比较函数没有返回值，所以对数组就没有排序操作，而每一次遍历中，第二个参数 `b` 对应前一个元素，第一个参数 `a` 对应后一个元素；当然该函数的具体排序方法就不得而知并且因 JS 引擎而异了；

## 升序

对数组按照升序方式排序，即小的元素排在前面，大的元素排在后面，假设比较函数当前遍历的元素对为 `(2, 1)`，则 `a = 1, b = 2`，要想升序就要 `a` 排到 `b` 的前面，对应上面的规则，就是需要比较函数的返回值小于 0，由于当前 `a - b < 0`；所以直接返回一个 `a - b` 就行了，代码如下：
```js
var arr = [2, 1, 3, 11, 12, 11]
arr.sort(function(a, b) {
    return a - b;
})

console.log(arr);
// [1, 2, 3, 11, 11, 12]
```

针对上面的代码再来分析下，在每一次遍历比较的两个元素中：

- 如果后一个元素比前一个元素小，即 `a - b < 0`，按照规则就是 `a` 要排到 `b` 的前面，也就是这两个元素会交换，小的在前，大的在后；
- 如果后一个元素比前一个元素大，即 `a - b > 0`，按照规则就是 `b` 要排到 `a` 的前面，由于 `b` 本来就在 `a` 的前面，所以两元素位置不变；
- 如果后一个元素与前一个元素相同，即 `a - b = 0`，按照规则就是 `a` 和 `b` 的位置不变，两元素位置同样不变；

最后，数组就变成升序的了；

## 降序

原理和升序类似，只是思路反过来了，代码如下：
```js
var arr = [2, 1, 3, 11, 12, 11]
arr.sort(function(a, b) {
    return b - a;
})

console.log(arr);
// [12, 11, 11, 3, 2, 1]
```

同样来分析一下，每一次遍历中：

- 如果前一个元素比后一个元素小，即 `b - a < 0`，按照规则就是 `a` 要排到 `b` 的前面，也就是这两个元素会交换，大的在前，小的在后；
- 如果前一个元素比后一个元素大，即 `b - a > 0`，按照规则就是 `b` 要排到 `a` 的前面，由于 `b` 本来就在 `a` 的前面，所以两元素位置不变；
- 如果前一个元素与后一个元素相同，即 `b - a = 0`，按照规则就是 `a` 和 `b` 的位置不变，两元素位置同样不变；

最后，数组也就变成降序了；

## 反序

这是排序函数一个另一个应用，作用相当于 `.reverse()` 函数，即让数组中的元素顺序颠倒，实现也很简单，就是利用规则，让每次比较函数的返回值小于 0 就行了，例如：
```js
var arr = [2, 1, 4, 3];
arr.sort(function(a, b) {
    return -1
});

console.log(arr);
// [3, 4, 1, 2]
```

## 乱序

这也算是一个比较实用的用途了，即将数组中元素的位置和顺序打乱，增加随机性，实现也简单，即利用规则，让比较函数的返回值随机为 `> 0, < 0, = 0` 这三种情况之一，使得元素是否交换位置具有随机性，也就实现了顺序的打乱，实现代码如下：
```js
var arr = [1, 2, 3, 4, 5];
arr.sort((a, b) => {
    return 0.5 - Math.random();
});

console.log(arr);
// [4, 3, 2, 1, 5]

arr.sort((a, b) => {
    return 0.5 - Math.random();
});

console.log(arr);
// [5, 3, 1, 2, 4]
```