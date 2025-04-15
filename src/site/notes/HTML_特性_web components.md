---
{"dg-publish":true,"permalink":"/HTML_特性_web components/"}
---

#review



Web Components 是一套 **浏览器原生提供的技术集合**，旨在让你能够创建可复用、封装良好的自定义 HTML 组件。  这意味着你可以像使用 `<div>` 或 `<span>` 这样的原生 HTML 标签一样，创建并使用自己的标签，并赋予它们特定的功能和样式。

让我们从业务场景的角度思考一下为什么需要 Web Components，以及它解决了什么问题。

**业务场景：组件复用和隔离的痛点**

假设你正在开发一个大型电商网站，需要构建各种各样的 UI 组件，例如：

* **商品卡片组件 (product-card)**：展示商品图片、价格、标题等信息，可能在首页、搜索结果页、商品详情页等多个地方使用。
* **轮播图组件 (image-slider)**：用于展示商品图片轮播，可能在首页、商品详情页等使用。
* **表单验证组件 (input-validator)**：用于各种表单输入框的实时验证，例如注册表单、登录表单、地址填写表单等。
* **地图组件 (store-locator)**：展示店铺位置，可能在店铺列表页、联系我们页面等使用。

在没有 Web Components 之前，你可能使用以下方式来构建和复用这些组件：

1. **JavaScript + HTML 模板字符串 + CSS 类名约定:**  你需要手动编写 JavaScript 代码来创建 DOM 结构，使用模板字符串来组织 HTML，并依赖 CSS 类名约定来应用样式。  **问题：**  组件的 HTML 结构、JavaScript 行为和 CSS 样式都散落在代码各处，缺乏真正的封装，容易造成命名冲突和样式污染。如果你使用不同的框架（比如一部分页面用 React，一部分用 Vue），组件的复用就更加困难。

2. **使用前端框架的组件系统 (如 React 组件, Vue 组件):**  框架的组件系统提供了更好的封装和复用能力，例如 React 的 JSX 和组件生命周期，Vue 的模板和组件选项。 **问题：** ==组件和特定的框架深度绑定，跨框架复用非常困难。如果你的项目逐渐演变成多框架架构（例如微前端架构），框架组件的复用性就大打折扣==。

**Web Components 的解决方案**

Web Components  旨在解决这些问题，它提供了一套标准化的、浏览器原生的方式来构建真正的**可复用、可封装、跨框架**的组件。

它主要由以下四个核心技术组成，就像构建一个积木房子一样，每个技术都是一个关键的积木：

让我们逐个击破这些核心技术，并结合业务场景来理解：

### 1. Custom Elements (自定义元素)

**核心思想：**  ==允许你定义自己的 HTML 标签==。

**业务场景应用：**  你想要创建一个商品卡片组件，在 HTML 中像这样使用：

```html
<product-card
  image="product1.jpg"
  title="精选商品 A"
  price="$99"
></product-card>

<product-card
  image="product2.jpg"
  title="热销商品 B"
  price="$49"
></product-card>
```

**如何实现：**  你需要使用 JavaScript  API `customElements.define()` 来注册你的自定义元素。  你需要创建一个 JavaScript 类，继承自 `HTMLElement` (或其子类，例如 `HTMLDivElement`，如果你想扩展 `div` 的功能)，并在类中定义组件的行为和生命周期。

**代码示例 (简化版):**

```javascript
class ProductCard extends HTMLElement {
  constructor() {
    super(); // 必须调用 super()
    // 组件初始化逻辑
    this.innerHTML = `
      <div class="product-card">
        <img src="${this.getAttribute('image')}" alt="${this.getAttribute('title')}">
        <h3>${this.getAttribute('title')}</h3>
        <p class="price">${this.getAttribute('price')}</p>
      </div>
    `;
  }

  connectedCallback() {
    // 组件被添加到 DOM 时触发
    console.log('ProductCard 组件被添加到页面了');
  }

  disconnectedCallback() {
    // 组件从 DOM 中移除时触发
    console.log('ProductCard 组件被从页面移除了');
  }

  attributeChangedCallback(name, oldValue, newValue) {
    // 组件的 attribute 发生变化时触发 (例如 <product-card title="..."> 中的 title 属性改变)
    console.log(`ProductCard 组件的 ${name} 属性从 ${oldValue} 变更为 ${newValue}`);
    // 你可以在这里根据 attribute 的变化更新组件的 UI
  }

  static get observedAttributes() {
    // 声明需要监听的 attribute 列表
    return ['image', 'title', 'price'];
  }
}

customElements.define('product-card', ProductCard); // 注册自定义元素
```

**关键点：**

* **生命周期回调：**  `connectedCallback`, `disconnectedCallback`, `attributeChangedCallback` 等方法允许你在组件的不同生命周期阶段执行代码，例如初始化、更新、销毁等。这和 React 组件的 `componentDidMount`, `componentWillUnmount`, `componentDidUpdate` 等生命周期方法类似，但更加原生和底层。
* **`observedAttributes`：**  你需要使用 `static get observedAttributes()` 方法声明你希望监听的 attribute 列表。只有声明过的 attribute 发生变化时，`attributeChangedCallback` 才会触发。
* **`this.getAttribute()`：**  在组件内部可以通过 `this.getAttribute('attributeName')` 获取 HTML 标签上的 attribute 值。

### 2. Shadow DOM (影子 DOM)

**核心思想：** 为组件创建一个独立的、隔离的 DOM 树，称为 Shadow DOM。

**业务场景应用：**  你希望 `product-card` 组件的样式和行为完全封装在组件内部，**不受到全局 CSS 样式的影响，也不影响全局 CSS 样式**。 例如，你希望组件内部使用的 `.product-card` 类名不会与页面上其他地方的 `.product-card` 类名冲突。

**如何实现：**  在你的自定义元素类中，使用 `this.attachShadow({ mode: 'open' })` 方法来创建一个 Shadow DOM 根节点，并将组件的 HTML 结构添加到这个 Shadow DOM 中。

**代码示例 (在上面的 `ProductCard` 代码基础上修改):**

```javascript
class ProductCard extends HTMLElement {
  constructor() {
    super();
    // 创建 Shadow DOM
    const shadowRoot = this.attachShadow({ mode: 'open' }); // 'open' 模式允许外部 JS 访问 Shadow DOM

    // 组件 HTML 结构放在 Shadow DOM 中
    shadowRoot.innerHTML = `
      <style>
        /* 组件内部的 CSS 样式，只作用于 Shadow DOM 内部 */
        .product-card {
          border: 1px solid #ccc;
          padding: 10px;
          border-radius: 5px;
          width: 250px;
          text-align: center;
        }
        .product-card img {
          max-width: 100%;
          height: auto;
        }
        .price {
          color: red;
          font-weight: bold;
        }
      </style>
      <div class="product-card">
        <img src="${this.getAttribute('image')}" alt="${this.getAttribute('title')}">
        <h3>${this.getAttribute('title')}</h3>
        <p class="price">${this.getAttribute('price')}</p>
      </div>
    `;
  }
  // ... (其他生命周期方法和 observedAttributes 同上)
}
```

**关键点：**

* **封装性：**  Shadow DOM 实现了真正的样式和行为封装。 组件内部的 CSS 样式只会作用于 Shadow DOM 内部的元素，不会泄露到外部。外部的 CSS 样式也默认不会影响 Shadow DOM 内部的元素。  JavaScript 也可以限制在 Shadow DOM 内部的作用域。
* **`attachShadow({ mode: 'open' })`：** `mode` 可以设置为 `'open'` 或 `'closed'`。 `'open'` 模式允许你通过 JavaScript (例如 `element.shadowRoot`) 从外部访问 Shadow DOM 的内容。 `'closed'` 模式则不允许外部访问，提供更强的封装性，但也会限制组件的灵活性。
* **DOM 查询范围：**  在 Shadow DOM 内部使用 `querySelector` 或 `querySelectorAll` 等 DOM 查询方法时，默认只会在 Shadow DOM 内部查找，不会穿透到外部文档 DOM。
* **事件冒泡：**  Shadow DOM 内发生的事件会冒泡到 Shadow DOM 的宿主元素 (即 `product-card` 标签)，但事件目标会被重定向到宿主元素本身，而不是 Shadow DOM 内部的元素。 这也是为了保证封装性。

**Shadow DOM 就像给你的组件创建了一个 “影子世界”，组件内部的一切都生活在这个影子世界里，与外部世界保持一定的隔离。**  这对于构建大型应用和组件库非常重要，可以避免样式和行为的冲突，提高代码的可维护性。

### 3. HTML Templates (&lt;template&gt; 和 &lt;slot&gt;)

**核心思想：**  使用 `<template>` 标签声明可复用的 HTML 结构模板，结合 `<slot>` 标签定义内容插槽，允许外部内容插入到组件的特定位置。

**业务场景应用：**  你希望 `product-card` 组件的 HTML 结构更加清晰和易于维护，并且希望组件的内容可以灵活定制。 例如，你希望在不同的场景下，`product-card` 组件的卡片底部可能展示不同的内容，例如 “加入购物车” 按钮、 “查看详情” 链接等等。

**如何实现：**

1. **使用 `<template>` 定义模板：**  将组件的 HTML 结构放在 `<template>` 标签内部。 `<template>` 标签的内容在页面加载时不会被渲染，直到被 JavaScript 激活。
2. **使用 `<slot>` 定义插槽：** 在 `<template>` 内部，使用 `<slot>` 标签定义内容插槽。  `<slot>` 标签就像组件中的占位符，外部 HTML 可以通过在自定义元素标签内部放置内容，来填充这些插槽。

**代码示例 (在上面的 `ProductCard` 代码基础上修改):**

```javascript
class ProductCard extends HTMLElement {
  constructor() {
    super();
    const shadowRoot = this.attachShadow({ mode: 'open' });

    // 创建 <template> 元素
    const template = document.createElement('template');
    template.innerHTML = `
      <style> /* ... 样式同上 ... */ </style>
      <div class="product-card">
        <img src="" alt="">
        <h3></h3>
        <p class="price"></p>
        <div class="actions">
          <slot name="actions"></slot>  <!-- 定义一个名为 "actions" 的插槽 -->
        </div>
      </div>
    `;

    // 将 template 的内容克隆到 Shadow DOM
    shadowRoot.appendChild(template.content.cloneNode(true));

    // ... (后续逻辑，例如在 connectedCallback 中设置图片、标题、价格等)
  }

  connectedCallback() {
    // 获取 Shadow DOM 内部的元素
    const shadowRoot = this.shadowRoot;
    const imgElement = shadowRoot.querySelector('img');
    const titleElement = shadowRoot.querySelector('h3');
    const priceElement = shadowRoot.querySelector('.price');

    // 设置元素的内容和属性
    imgElement.src = this.getAttribute('image');
    imgElement.alt = this.getAttribute('title');
    titleElement.textContent = this.getAttribute('title');
    priceElement.textContent = this.getAttribute('price');
  }

  // ... (其他生命周期方法和 observedAttributes 同上)
}
```

**HTML 使用示例：**

```html
<product-card
  image="product3.jpg"
  title="特色商品 C"
  price="$129"
>
  <button slot="actions">加入购物车</button>  <!--  使用 slot="actions"  填充 actions 插槽 -->
</product-card>

<product-card
  image="product4.jpg"
  title="新品商品 D"
  price="$79"
>
  <a slot="actions" href="/product/D">查看详情</a> <!--  使用 slot="actions"  填充 actions 插槽 -->
</product-card>
```

**关键点：**

* **`<template>` 的好处：**  将组件的 HTML 结构放在 `<template>` 中，可以提高代码的可读性和可维护性，并避免直接在 JavaScript 字符串中拼接 HTML 带来的问题。  浏览器会对 `<template>` 内部的内容进行解析，但不会立即渲染，这提高了性能。
* **`<slot>` 的灵活性：**  `<slot>` 提供了内容分发机制，允许你在组件的不同使用场景下，灵活地定制组件的内容，而无需修改组件的内部代码。  你可以定义**匿名插槽** (不带 `name` 属性) 和 **命名插槽** (带有 `name` 属性)。  外部内容通过 `slot` attribute 来指定要填充哪个插槽。
* **`template.content.cloneNode(true)`：**  你需要使用 `template.content.cloneNode(true)` 来克隆 `<template>` 的内容，并将其添加到 Shadow DOM 中。  直接使用 `template.innerHTML` 会导致每次组件实例都解析 HTML 字符串，效率较低。

### 4. ES Modules (模块化组织代码)

**核心思想：**  使用 ES Modules  的 `import` 和 `export` 语法来组织和管理你的 Web Components 代码，实现真正的模块化开发。

**业务场景应用：**  当你的项目包含多个 Web Components 组件时，你需要一种方式来组织这些组件的代码，避免全局命名冲突，并实现按需加载。

**如何实现：**  将每个 Web Component 的代码放在单独的 JavaScript 文件中，使用 `export default` 导出组件类，然后在需要使用组件的地方使用 `import` 导入。

**文件结构示例：**

```
components/
├── product-card.js  // ProductCard 组件代码
├── image-slider.js  // ImageSlider 组件代码
├── input-validator.js // InputValidator 组件代码
└── ...
```

**`product-card.js` 文件 (简化版):**

```javascript
class ProductCard extends HTMLElement {
  //