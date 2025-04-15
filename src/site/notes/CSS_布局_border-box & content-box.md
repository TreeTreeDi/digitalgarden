---
{"dg-publish":true,"permalink":"/CSS_布局_border-box & content-box/"}
---

#review  



想象一下你正在开发一个电商网站的商品列表页面，类似淘宝或京东。你需要并排展示多个商品卡片，每个卡片宽度固定，比如 200px，并且有内边距（`padding`）和边框（`border`）来美化样式。

**`content-box` (默认值):**

*   **思考角度/业务场景:** 如果你使用默认的 `content-box`，你设置的 `width: 200px` **仅仅是内容区域（content area）的宽度**。如果你给这个卡片添加了 `padding: 10px` 和 `border: 1px solid black`，那么这个卡片在屏幕上实际占据的总宽度会变成：
    `内容宽度 (200px) + 左内边距 (10px) + 右内边距 (10px) + 左边框 (1px) + 右边框 (1px) = 222px`。
*   **问题:** 这就有点反直觉了。你明明想让它正好是 200px 宽，方便计算一行能放几个卡片（比如容器宽度 800px，理论上能放 4 个 200px 的卡片），但加上内边距和边框后，它就“胖”了，变成了 222px，导致一行可能放不下 4 个，布局就乱了。你需要反过来计算，如果你希望最终宽度是 200px，那么内容宽度就得设置成 `200px - 10px*2 - 1px*2 = 178px`。这太麻烦了，每次调整内边距或边框都要重新计算。
*   **具体网站例子:** 早期的网页或者没有明确设置 `box-sizing` 的元素就遵循这个规则。你可以用浏览器开发者工具检查一些旧网站的元素，或者自己写个简单的例子，不加 `box-sizing` 声明，然后设置 `width`, `padding`, `border`，观察Computed（计算样式）里的总宽度。

**`border-box`:**

*   **思考角度/业务场景:** 这个模型就直观多了！==当你设置 `box-sizing: border-box;` 后，你设置的 `width: 200px` **就是这个元素最终在屏幕上占据的总宽度，它包含了内容（content）、内边距（padding）和边框（border）**==。
*   **好处:** 还是刚才那个商品卡片的例子，设置了 `box-sizing: border-box;` 和 `width: 200px;`。现在，即使你再加上 `padding: 10px` 和 `border: 1px solid black`，这个卡片在屏幕上的总宽度**仍然是 200px**！浏览器会自动帮你调整内容区域的大小，让内容、内边距、边框加起来正好等于你设定的 200px 宽度。内容区域实际宽度会变成 `200px - 10px*2 - 1px*2 = 178px`，但这通常是你不太关心的，你关心的是它占据的布局空间是精确的 200px。
*   **具体网站例子:** 现在几乎所有的现代网站和 CSS 框架（如 Bootstrap, Tailwind CSS）都会在全局设置 `*, *::before, *::after { box-sizing: border-box; }`。你去检查这些网站（比如 Bootstrap 的官方文档页面），会发现大部分元素都应用了这个规则。这使得它们的栅格系统、组件布局等都非常稳定和可预测。你设置一个 `col-md-4`（假设它意味着 33.33% 宽度），它就真的是占据那么多宽度，无论你给它加多少 padding 或 border。

**总结:**

*   `content-box`: `width` 和 `height` 只包含内容区。==总尺寸 = 内容 + padding + border==。
*   `border-box`: `width` 和 `height` 包含内容、padding 和 border。总尺寸 = `width`/`height` 值。

==对于我们前端开发来说，`border-box` 通常是更优的选择，因为它让布局计算更简单、直观、可预测。这就是为什么现代 CSS 实践中，通常会把它设置为全局默认值==。

---

**联想相关知识点:**

理解了 `box-sizing`，自然会联想到以下几个 CSS 核心概念：

1.  **CSS 盒模型 (CSS Box Model):** `box-sizing` 直接影响盒模型的计算方式。盒模型描述了 HTML 元素如何被渲染成一个矩形的盒子，这个盒子由四个区域组成：内容（Content）、内边距（Padding）、边框（Border）、外边距（Margin）。`box-sizing` 决定了 `width` 和 `height` 属性具体应用到哪个区域的总和上。注意，`margin` 始终在 `width`/`height` 定义的范围之外，不受 `box-sizing` 影响。
2.  **布局 (Layout):** `box-sizing` 对基于宽度的布局（如传统的 `float` 布局，以及现代的 `Flexbox` 和 `Grid` 布局）有重大影响。使用 `border-box` 可以更容易地创建等宽列、精确控制元素尺寸，避免因子元素添加 `padding` 或 `border` 导致父容器被撑开或布局错乱的问题。
3.  **CSS 重置 (CSS Reset) / Normalize.css:** 正是因为 `border-box` 的好处，很多 CSS Reset 或 Normalize 方案（用于统一不同浏览器的默认样式）会包含一条全局规则，如：
    ```css
    html {
      box-sizing: border-box;
    }
    *, *::before, *::after {
      box-sizing: inherit; /* 让所有元素继承 html 的 box-sizing 设置 */
    }
    ```
    或者更直接地：
    ```css
    *, *::before, *::after {
      box-sizing: border-box;
    }
    ```
    这确保了整个项目中的元素尺寸计算方式一致，提高了开发效率和样式稳定性。
4.  **`width` 和 `height` 属性:** `box-sizing` 重新定义了这两个基本属性的计算范围。

