---
{"dg-publish":true,"permalink":"/浏览器_API_性能调度 API & 性能优化 API/"}
---

#review  



**一、 性能测量 API (Performance Measurement APIs)**

这类 API 让你能够精确地获取代码执行、资源加载、页面渲染等各个环节的时间信息和指标。

1.  **`performance.now()`**
    *   **目的:** ==提供一个高精度的时间戳==（通常精确到微秒级别），==表示自页面导航开始（或某个特定时间原点）以来经过的毫秒数==。
    *   **与 `Date.now()` 的区别:** `Date.now()` 基于系统时钟，可能受系统时间调整影响，精度也较低（毫秒级）。`performance.now()` 是单调递增的，不受系统时钟影响，精度更高，更适合用来测量代码执行耗时。
    *   **使用场景:** 精确测量某段 JavaScript 代码的执行时间。
        ```javascript
        const startTime = performance.now();
        // --- 执行一些耗时操作 ---
        for (let i = 0; i < 1000000; i++) { /* ... */ }
        const endTime = performance.now();
        const duration = endTime - startTime;
        console.log(`操作耗时: ${duration} 毫秒`);
        ```
    *   **React 场景:** 测量某个复杂计算函数、组件渲染（结合 `useEffect` 或生命周期钩子）的耗时，帮助定位性能瓶颈。

2.  **Performance Timeline API (`performance.getEntries*`, `PerformanceObserver`)**
    *   **目的:** 访问浏览器记录的各种性能指标条目（Performance Entries），如资源加载时间、导航时间、绘制时间、自定义标记等。
    *   **核心方法:**
        *   `performance.getEntries()`: 获取所有性能条目。
        *   `performance.getEntriesByType(type)`: 获取特定类型的条目 (如 'navigation', 'resource', 'paint', 'mark', 'measure', 'largest-contentful-paint', 'layout-shift', 'first-input')。
        *   `performance.getEntriesByName(name, type)`: 获取指定名称和类型的条目。
    *   **`PerformanceObserver`:** 一个更高效的方式，用于**异步**订阅性能条目事件。当新的符合条件的性能条目产生时，你的回调函数会被（通常作为微任务）调用，避免了反复轮询 `getEntries*`。
    *   **使用场景:**
        *   分析页面加载性能（Navigation Timing）。
        *   监控图片、脚本、CSS 等资源的加载时间（Resource Timing）。
        *   获取首次绘制 (FP)、首次内容绘制 (FCP) 时间（Paint Timing）。
        *   获取最大内容绘制 (LCP)、累积布局偏移 (CLS)、首次输入延迟 (FID) 或交互到下一次绘制(INP) 等 **Core Web Vitals** 指标。
        *   配合 User Timing API 分析自定义业务逻辑耗时。
    *   **React 场景:** 在应用加载后，使用 `PerformanceObserver` 收集 LCP、CLS、FID/INP 指标，上报给监控系统，了解真实用户体验。或者观察特定资源的加载情况。

3.  **User Timing API (`performance.mark`, `performance.measure`)**
    *   **目的:** 允许开发者在代码中设置自定义的时间戳（标记）和测量区间。
    *   **核心方法:**
        *   `performance.mark(name)`: 在代码执行到此处时创建一个名为 `name` 的性能标记条目。
        *   `performance.measure(measureName, startMarkName, endMarkName)`: 计算从 `startMarkName` 到 `endMarkName` 之间的时间差，并创建一个名为 `measureName` 的性能测量条目。也可以省略标记名，直接测量从导航开始或某个标记到当前的时间。
    *   **使用场景:** 测量特定业务逻辑、组件生命周期阶段、异步操作等的耗时。
        ```javascript
        performance.mark('complex-calculation-start');
        // --- 执行复杂计算 ---
        await performCalculation();
        performance.mark('complex-calculation-end');

        performance.measure(
          'complex-calculation-duration',
          'complex-calculation-start',
          'complex-calculation-end'
        );

        // 之后可以通过 PerformanceObserver 或 getEntriesByType('measure') 获取这个测量结果
        const measures = performance.getEntriesByName('complex-calculation-duration', 'measure');
        if (measures.length > 0) {
          console.log(`复杂计算耗时: ${measures[0].duration} ms`);
        }
        ```
    *   **React 场景:** 测量某个特定组件（如大型列表或图表）从开始渲染到完成渲染（可能结合 `useEffect` 和 `performance.mark`）的总耗时，或者测量 Redux action 处理及后续渲染的端到端时间。

4.  **`navigator.connection` (Network Information API)**
    *   **目的:** 提供有关用户设备当前网络连接的信息（如类型 `type` - 'wifi', 'cellular'，有效类型 `effectiveType` - '4g', '3g', 'slow-2g'，下行链路 `downlink` 带宽估计等）。
    *   **使用场景:** ==根据用户的网络状况调整应用行为以优化性能==。例如，在慢速网络下，可以：
        *   加载低质量图片或延迟加载非关键资源。
        *   减少数据预取。
        *   提供更简单的 UI 体验。
    *   **React 场景:** 创建一个 `useNetworkStatus` Hook，根据网络状态动态调整传递给图片组件的 `src` 或数据请求的参数。

**二、 性能优化/调度 API (Performance Optimization/Scheduling APIs)**

这些 API 帮助你更智能地安排代码执行时机，避免阻塞主线程，或者更有效地加载资源。

*(以下部分 API 在之前的讨论中已涉及，这里做整合和补充)*

[[浏览器_API_性能调度 API & 性能优化 API_requestAnimationFrame\|浏览器_API_性能调度 API & 性能优化 API_requestAnimationFrame]]

[[浏览器_API_性能调度 API & 性能优化 API_requestIdleCallback\|浏览器_API_性能调度 API & 性能优化 API_requestIdleCallback]]
6.  **`setTimeout(callback, 0)` / `queueMicrotask(callback)`**
    *   **目的:** 将任务推迟到**未来的事件循环**执行。`setTimeout(fn, 0)` 将任务放入**宏任务**队列，`queueMicrotask(fn)` 将任务放入**微任务**队列。
    *   **核心区别与用途:**
        *   `queueMicrotask`: 优先级极高，在当前同步代码块结束后、任何渲染或下一个宏任务前立即执行。用于需要尽快异步执行但又晚于同步代码的场景。
        *   `setTimeout(fn, 0)`: 优先级较低，用于将长任务拆分成小块，释放主线程，允许渲染和用户交互介入。
    *   **React 场景:** 如前所述，`setTimeout` 用于分批渲染大数据列表；`queueMicrotask` 可能用于在多个状态同步更新后，安排一个最终的校验或副作用操作，确保它在渲染前执行。
[[浏览器_API_性能调度 API & 性能优化 API_Web Worker\|浏览器_API_性能调度 API & 性能优化 API_Web Worker]]

8.  **Resource Hints (`<link rel="preload|preconnect|prefetch">`)**
    *   **目的:** 提前告知浏览器可能需要哪些资源或需要与哪些服务器建立连接，让浏览器可以优化加载过程。这不是 JS API，而是 HTML 标签，但与性能密切相关。
    *   **`preload`:** 告诉浏览器当前页面**肯定**需要某个资源（如关键脚本、CSS、字体），应尽早以较高优先级获取，但不执行/应用它，等待代码实际使用。
    *   **`preconnect`:** 告诉浏览器当前页面**很可能**需要与某个域名建立连接（DNS 查询、TCP 握手、TLS 协商），可以提前完成这些耗时操作。
    *   **`prefetch`:** 告诉浏览器某个资源**可能在未来**的导航中用到（如下一个页面），可以在浏览器空闲时以较低优先级下载。
    *   **使用场景:** 优化关键渲染路径，减少关键资源的加载延迟，加速后续页面加载。
    *   **React 场景:** 对于 Code Splitting 产生的关键 JS chunk，可以使用 `preload` 提前加载；对于 API 服务器或 CDN，可以使用 `preconnect`。

9. **Fetch API `priority` hint (较新)**
    *   **目的:** 在使用 `fetch` API 时，向浏览器提供请求优先级建议。
    *   **`fetch(url, { priority: 'high' | 'low' | 'auto' })`**
    *   **使用场景:** 明确告知浏览器哪些 API 请求更重要（如用户操作触发的数据请求设为 `high`），哪些可以推迟（如后台日志上报设为 `low`）。浏览器会据此调整网络请求的处理顺序。
    *   **React 场景:** 在数据获取逻辑中，根据请求的重要性设置不同的 `priority`。

**总结**

掌握这些性能相关的 API 是高级前端工程师必备的技能。核心思路是：

1.  **测量 (Measure):** 使用 `performance.now()`, Performance Timeline API, User Timing API 等工具精确了解性能现状和瓶颈。关注 Core Web Vitals。
2.  **优化 (Optimize):**
    *   **调度 (Scheduling):** 使用 `requestAnimationFrame` 处理动画，`requestIdleCallback` 处理低优任务，`setTimeout(0)` 分割长任务，`queueMicrotask` 处理高优异步，`Web Workers` 处理重计算。
    *   **资源 (Resources):** 使用 Resource Hints 优化加载，考虑 Fetch priority。
    *   **感知 (Perception):** 利用 Network Information API 适应不同网络环境。

通过结合使用这些 API，你可以更科学地进行前端性能优化，提升用户体验。