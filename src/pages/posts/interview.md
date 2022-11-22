<!-- ---
title: 面试题专栏
date: 2022-11-11T00:00:00.000+00:00
lang: zh
duration: 10min
author: 沈佳棋
--- -->

## 1. == 隐式转换的规则

```javascript
// 阿里、百度、腾讯面试题
// ? 位置应该怎么写才能输出 true
var a = ?;
console.log(
  a == 1 &&
  a == 2 &&
  a == 3
);
```

考点：`=` 运算符的运算规则 & 隐式转换规则

1. `=` 运算符的运算规则
<img src="/public/img1.png" />  
   
这里重点看一下：一端是原始值，一端是对象这种情况：  
**在 == 比较的时候对象会先转换成原始值。怎么转换呢？对象会先调用 `valueOf`，如果还是无法转换成原始值，再调用 `toString`**。  
  
举例如下：
```javascript
const obj = {};
console.log(obj == 1); // false

// 会经过的步骤如下：
// 1. 调用 valueOf
console.log(obj.valueOf()); // {}
// 2. 调用 toString
console.log(obj.toString()); // '[object Object]'
```

那么怎么让 `obj == 1` 的结果为 `true` 呢？  
很简单，只需要重写 `obj` 内部的 `valueOf` 或者是 `toString` 方法即可。  
  
```javascript
const obj = {
  valueOf: function() {
    return 1;
  }
};
console.log(obj == 1); // true
```

再回到这道题本身，现在看就很简单了。a 必然是个对象，而且会经过三次隐式转换，分别转换成 `1`、`2`、`3`。  
  
答案：
```javascript
const a = {
  n: 1;
  valueOf: function() {
    return this.n++;
  }
};
console.log(
  a == 1 &&
  a == 2 &&
  a == 3
);
```

**题目变体：**
```javascript
var a = ?;
console.log(
  a == 1 &&
  a == 2 &&
  a == 4
); // true
```

答案：
```javascript
var a = {
  n: 0,
  valueOf() {
    var res = Math.pow(2, n);
    n++;
    return res;
  }
};
console.log(
  a == 1 &&
  a == 2 &&
  a == 4
); // true
```

## 2. 可以重试的请求方法

```javascript
// 美团
/**
 * 发出请求，返回Promise
 * @param {string} url 请求地址
 * @param {number} maxCount 最大重试次数
 */
function request(url, maxCount = 5) {}

request('https://github.com/', 6)
  .then(res => res.json())
  .then(data => {
    console.log(data);
  });
  .catch(err => {
    console.log(err);
  })
```

这里的请求选择用 `fetch`。请求正常的时候我们直接将其返回即可；在请求失败的时候如果重试次数 `<=0` 则用 `Project.reject` 返回失败结果，否则递归调用 `request` 方法进行重试。  
  
代码如下：
```javascript
function request(url, maxCount = 5) {
  return fetch(url).catch(err => {
    maxCount <= 0 ? Promise.reject(err) : request(url, maxCount-1);
  });
}
```

<img src="/public/img2.png" />