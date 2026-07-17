---
title: 深入理解 JavaScript 异步编程
date: 2026-07-03
tags: 技术, JavaScript
excerpt: 从回调到 Promise 再到 async/await，梳理 JavaScript 异步编程的演进脉络和最佳实践。
cover: linear-gradient(135deg, #f6d365 0%, #fda085 100%)
readingTime: 12
---

# 深入理解 JavaScript 异步编程

JavaScript 是单线程语言，异步编程是它的核心能力之一。

## 为什么需要异步？

JavaScript 运行在单线程中，如果所有操作都是同步的，一个耗时操作会阻塞整个页面。

```javascript
// 同步：阻塞
const data = syncFetchData(); // 页面卡住直到数据返回
console.log(data);

// 异步：非阻塞
asyncFetchData().then(data => {
    console.log(data);
});
console.log("这行会先执行");
```

## 异步的三个时代

### 时代一：回调（Callback）

最原始的异步方式，通过传入回调函数处理结果。

```javascript
fetchData(url, function(err, data) {
    if (err) {
        handleError(err);
        return;
    }
    processData(data, function(err, result) {
        if (err) {
            handleError(err);
            return;
        }
        renderResult(result);
    });
});
```

**问题**：回调地狱（Callback Hell）— 嵌套层层叠加，难以维护。

### 时代二：Promise

Promise 代表一个最终会完成的异步操作的结果。

```javascript
fetchData(url)
    .then(data => processData(data))
    .then(result => renderResult(result))
    .catch(err => handleError(err));
```

**优点**：
- 链式调用，避免嵌套
- 统一的错误处理
- 状态明确：pending → fulfilled / rejected

### 时代三：async/await

让异步代码看起来像同步代码。

```javascript
async function main() {
    try {
        const data = await fetchData(url);
        const result = await processData(data);
        renderResult(result);
    } catch (err) {
        handleError(err);
    }
}
```

## 实用模式

### 并行执行

```javascript
// 等待所有完成
const [users, posts] = await Promise.all([
    fetchUsers(),
    fetchPosts()
]);
```

### 错误重试

```javascript
async function retry(fn, times = 3) {
    for (let i = 0; i < times; i++) {
        try {
            return await fn();
        } catch (err) {
            if (i === times - 1) throw err;
            await new Promise(r => setTimeout(r, 1000 * (i + 1)));
        }
    }
}
```

## 最佳实践

1. **始终处理错误** — 不要让 Promise 静默失败
2. **合理使用并行** — 无依赖的异步操作应该并行
3. **避免 async 构造函数** — 构造函数不能是 async
4. **注意 await 的位置** — `const a = await x(); const b = await y();` 是串行的

---

异步编程是 JavaScript 的灵魂。理解它，你就掌握了现代前端开发的核心。
