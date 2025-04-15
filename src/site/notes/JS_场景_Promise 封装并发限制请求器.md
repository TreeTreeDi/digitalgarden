---
{"dg-publish":true,"permalink":"/JS_场景_Promise 封装并发限制请求器/"}
---

#review  

#### 完整代码
```javascript
class LimitedRequester {
  constructor(maxConcurrency) {
    this.maxConcurrency = maxConcurrency;
    this.runningCount = 0;
    this.queue = [];
  }

  add(requestFn) {
  // 返回 Promise 调用者知道什么情况下可以返回 Req
    return new Promise((resolve, reject) => {
      const task = { fn: requestFn, resolve, reject };
      if (this.runningCount < this.maxConcurrency) {
        this.runTask(task);
      } else {
        this.queue.push(task);
      }
    });
  }

  runTask(task) {
    this.runningCount++;
    task.fn()
      .then(result => task.resolve(result))
      .catch(error => task.reject(error))
      .finally(() => {
        this.runningCount--;
        this.scheduleNext();
      });
  }

  scheduleNext() {
    if (this.queue.length > 0 && this.runningCount < this.maxConcurrency) {
      const nextTask = this.queue.shift();
      this.runTask(nextTask);
    }
  }
}
```

#### 测试一下
```javascript
// 模拟请求函数
const mockRequest = (id, delay) => {
  return new Promise(resolve => {
    setTimeout(() => resolve(`请求 ${id} 完成`), delay);
  });
};

// 创建请求器，最大并发数为 2
const requester = new LimitedRequester(2);

// 添加 5 个请求
for (let i = 1; i <= 5; i++) {
  requester.add(() => mockRequest(i, 1000)).then(result => console.log(result));
}
```
**输出（大致每秒 2 个完成）：**  
```
请求 1 完成
请求 2 完成
请求 3 完成
请求 4 完成
请求 5 完成
```

---

### 第四步：总结与巩固
#### 核心逻辑回顾
1. 用 `maxConcurrency` 控制并发上限。
2. `runningCount` 跟踪当前任务数。
3. `queue` 存等待任务。
4. `add` 判断是立即执行还是排队，`runTask` 执行任务，`scheduleNext` 调度队列。



#### 检查理解漏洞
- 如果所有请求同时到达，会发生什么？（答：前 `maxConcurrency` 个立即执行，其余排队）
- `finally` 为什么重要？（答：确保任务完成后更新状态并调度）

#### 扩展练习
1. **加日志**：在 `runTask` 中打印当前运行数和队列长度。
2. **优先级队列**：让任务带优先级，高优先级的先执行。

---

### 第五步：互动练习
请完成以下任务，并告诉我结果：
1. 用这个 `LimitedRequester` 模拟 4 个请求，最大并发数为 2，每个请求延时不同（比如 500ms, 1000ms, 1500ms, 2000ms），观察输出顺序。
2. 解释为什么 `runTask` 用 `finally` 而不是只在 `then` 或 `catch` 中减 `runningCount`。

期待你的代码和回答！通过这个过程，你不仅学会了并发限制请求器，还能用图表和语言清晰表达逻辑！