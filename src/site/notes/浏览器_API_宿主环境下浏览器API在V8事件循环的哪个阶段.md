---
{"dg-publish":true,"permalink":"/浏览器_API_宿主环境下浏览器API在V8事件循环的哪个阶段/"}
---

#review  


**事件循环基本流程（简化版）：**

1.  执行当前的同步代码（宏任务的一部分，或初始脚本）。
2.  执行所有**微任务 (Microtasks)**。
3.  **可能**进行渲染更新（Style -> Layout -> Paint）。
4.  从**宏任务 (Macrotasks / Tasks)** 队列中取出一个任务执行。
5.  回到第 2 步。

**API 回调执行阶段：**

1.  **同步执行 (直接在当前任务中):**
    *   大部分函数调用，如 `performance.now()`, `console.log()`, `element.setAttribute()` 等。
    *   `performance.mark()`, `performance.measure()`, `performance.getEntries*()`。
    *   读取 DOM 属性 (`element.clientWidth`)。

2.  **微任务队列 (Microtask Queue - 在同步代码后、渲染前、下一个宏任务前立即执行):**
    *   `Promise.then()`, `.catch()`, `.finally()` 的回调。
    *   `queueMicrotask()` 的回调。
    *   `MutationObserver` 的回调。
    *   `PerformanceObserver` 的回调 (通常是微任务，为了及时响应)。

==3.  **渲染前 (与渲染紧密绑定，非标准队列):**==
    *   `requestAnimationFrame(callback)` 的回调：在浏览器执行下一次绘制（Style, Layout, Paint）**之前**被调用。

4.  **宏任务队列 (Macrotask Queue / Task Queue - 每个循环执行一个):**
    *   `setTimeout(callback)` 的回调。
    *   `setInterval(callback)` 的回调。
    *   用户交互事件的回调 (如 `click`, `scroll`, `keydown` 的 `addEventListener` 回调)。
    *   I/O 操作的回调 (如 `XMLHttpRequest` 的 `onload`/`onerror`, `fetch` *完成网络请求后触发 Promise resolve/reject 的内部任务*)。
    *   `postMessage` (来自 Worker 或跨窗口) 的 `onmessage` 事件回调。
    *   `IndexedDB` 操作的回调 (`onsuccess`, `onerror`)。

5.  **空闲时段 (机会性执行，非标准队列):**
    *   `requestIdleCallback(callback)` 的回调：在浏览器一帧的末尾（渲染完成后），如果**有空闲时间**，则执行。优先级较低。
