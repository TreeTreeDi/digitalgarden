---
{"dg-publish":true,"permalink":"/JS_场景_深拷贝/"}
---

好的，我们来彻底搞懂**深拷贝 (Deep Copy)**。这在 JavaScript 中是个非常重要的概念，尤其是在处理复杂数据结构（特别是嵌套的对象和数组）以及在 React 等框架中管理状态时。

#review 

**理解问题的根源：赋值 vs. 浅拷贝 vs. 深拷贝**



首先，我们要区分 JavaScript 中处理数据的几种方式：

1.  **赋值 (`=`)**
    *   **基本类型 (Primitives):** 如 `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`。赋值是**按值复制**。
        ```javascript
        let a = 10;
        let b = a; // b 得到 a 的值 10
        b = 20;    // 修改 b
        console.log(a); // 输出 10 (a 不受影响)
        ```
    *   **引用类型 (Objects, Arrays):** 赋值是**按引用复制**。这意味着变量存储的是对象在内存中的地址（引用），赋值只是复制了这个地址。两个变量指向同一个对象。
        ```javascript
        let objA = { name: 'Alice', address: { city: 'Wonderland' } };
        let objB = objA; // objB 和 objA 指向同一个对象

        objB.name = 'Bob'; // 修改 objB 的属性
        console.log(objA.name); // 输出 'Bob' (objA 也被修改了!)

        objB.address.city = 'New York'; // 修改嵌套对象的属性
        console.log(objA.address.city); // 输出 'New York' (objA 的嵌套对象也被修改了!)
        ```
    *   **结论：** ==赋值对于引用类型来说，根本不算“拷贝”，只是创建了一个指向相同对象的别名==。

2.  **浅拷贝 (Shallow Copy)**
    *   **目标：** 创建一个新的对象或数组，并将原始对象/数组的**顶层**属性/元素复制到新对象/数组中。
    *   **行为：**
        *   如果顶层属性值是**基本类型**，则复制该值。
        *   如果顶层属性值是**引用类型**（嵌套的对象或数组），则只复制其**引用（内存地址）**。
    * 
    *   **常见方法：**
        *   Spread syntax (`{...obj}`, `[...arr]`)
        *   `Object.assign({}, obj)`
        *   `Array.prototype.slice()`
        *   `Array.from(arr)`
    *   **例子：**
        ```javascript
        let objA = { name: 'Alice', address: { city: 'Wonderland' } };
        let objShallow = { ...objA }; // 使用 spread syntax 进行浅拷贝

        console.log(objShallow === objA); // 输出 false (objShallow 是一个新对象)

        objShallow.name = 'Bob'; // 修改顶层基本类型属性
        console.log(objA.name); // 输出 'Alice' (原始对象的顶层基本类型不受影响)

        // !!! 关键点 !!!
        objShallow.address.city = 'New York'; // 修改嵌套对象的属性
        console.log(objA.address.city); // 输出 'New York' (原始对象的嵌套对象 *也* 被修改了!)
        ```
    *   **结论：** ==浅拷贝只复制了第一层。如果对象内部还有嵌套的对象或数组，拷贝后的对象和原始对象将共享这些嵌套的引用类型数据==。修改任意一方的嵌套内容，另一方也会受到影响。

3.  **深拷贝 (Deep Copy)**
    *   **目标：** 创建一个**完全独立**的新对象或数组。不仅复制顶层属性/元素，还会**递归地**复制所有嵌套的对象和数组，直到所有层级都是基本类型为止。
    *   **行为：** 拷贝后的对象/数组及其所有嵌套内容，与原始对象/数组在内存中是完全分离的，没有任何共享的引用（除了不可变的基本类型值）。
    *   **结果：** 修改拷贝后的对象/数组（无论多深层级），==**绝对不会**影响到原始对象/数组==。
    *   **例子 (假设 `deepClone` 是一个实现深拷贝的函数):**
        ```javascript
        let objA = { name: 'Alice', address: { city: 'Wonderland' }, hobbies: ['reading'] };
        let objDeep = deepClone(objA); // 执行深拷贝

        console.log(objDeep === objA); // 输出 false (objDeep 是一个新对象)
        console.log(objDeep.address === objA.address); // 输出 false (嵌套对象也是新的)
        console.log(objDeep.hobbies === objA.hobbies); // 输出 false (嵌套数组也是新的)

        objDeep.name = 'Bob';
        console.log(objA.name); // 输出 'Alice' (不受影响)

        objDeep.address.city = 'New York';
        console.log(objA.address.city); // 输出 'Wonderland' (不受影响!)

        objDeep.hobbies.push('coding');
        console.log(objA.hobbies); // 输出 ['reading'] (不受影响!)
        ```
    *   **结论：** 深拷贝创建了一个彻底的副本，两者完全解耦。

**如何实现深拷贝？**

有几种常见的方法，各有优缺点：

1.  **`JSON.parse(JSON.stringify(object))` (最常用，但有局限)**
    *   **原理：** 先将 JavaScript 对象序列化（转换）为 JSON 字符串，然后再将该字符串解析（转换）回新的 JavaScript 对象。这个过程天然地创建了所有层级的新对象和数组。
    *   **优点：** 非常简单，易于理解和使用，浏览器原生支持。
    *   **缺点 (非常重要！):**
        *   会丢失 `undefined`、`function`、`symbol` 类型的值（属性会直接消失或变为 `null`）。
        *   不能正确处理 `Date` 对象（会变成字符串）。
        *   不能处理 `RegExp`、`Error` 对象。
        *   不能处理 `Map`、`Set` 等 ES6+ 的数据结构。
        *   不能处理循环引用（对象内部有属性直接或间接指向自身），会导致错误。
        *   会忽略对象的原型链。
    *   **适用场景：** 当你的数据结构只包含 JSON 安全的类型（字符串、数字、布尔值、`null`、以及只包含这些类型的嵌套对象和数组）时，这是一个快速简便的方法。例如，处理来自 API 的纯数据。

    ```javascript
    let objA = {
      name: 'Alice',
      birthday: new Date(),
      action: function() {},
      id: Symbol('id'),
      metadata: undefined,
      scores: new Set([1, 2]),
      address: { city: 'Wonderland' }
    };
    let objDeepJSON = JSON.parse(JSON.stringify(objA));
    console.log(objDeepJSON);
    /* 输出类似（具体看环境）:
       {
         name: 'Alice',
         birthday: '2023-10-27T...', // Date 变成了字符串
         // action, id, metadata 丢失了!
         scores: {}, // Set 变成了空对象!
         address: { city: 'Wonderland' }
       }
    */
    ```

2.  **`structuredClone(object)` (现代浏览器的推荐方式)**
    *   **原理：** 这是浏览器和 Node.js (v17+) 提供的**原生 API**，专门用于执行深拷贝。它使用了**结构化克隆算法 (Structured Cloning Algorithm)**，这个算法比 JSON 序列化强大得多。
    *   **优点：**
        *   原生、高效。
        *   支持多种数据类型，包括 `Date`, `RegExp`, `Map`, `Set`, `ArrayBuffer`, `Blob`, `File`, `ImageData` 等。
        *   能正确处理**循环引用**。
        *   比 `JSON.parse(JSON.stringify())` 更可靠、功能更强。
    *   **缺点：**
        *   **不复制函数 (Functions)**：函数会被直接忽略。
        *   **不复制原型链 (Prototype Chain)**：只复制对象自身的属性。
        *   浏览器和 Node.js 版本兼容性问题（需要较新的环境）。
    *   **适用场景：** **目前最佳的内置深拷贝方法**，==只要你的目标环境支持并且你不需要拷贝函数或原型链==。

    ```javascript
    let objA = {
        name: 'Alice',
        birthday: new Date(),
        action: function() {}, // 会被忽略
        scores: new Set([1, 2]),
        address: { city: 'Wonderland' },
        ref: null
    };
    objA.ref = objA; // 创建循环引用

    try {
        let objDeepStructured = structuredClone(objA);
        console.log(objDeepStructured);
        console.log(objDeepStructured.birthday instanceof Date); // true (Date 类型被保留)
        console.log(objDeepStructured.scores instanceof Set);    // true (Set 类型被保留)
        console.log(objDeepStructured.ref === objDeepStructured); // true (循环引用被正确处理)
        console.log('action' in objDeepStructured);             // false (函数被忽略)
    } catch (e) {
        console.error("structuredClone failed:", e);
    }
    ```

3.  **递归手动实现**
    *   **原理：** 编写一个递归函数，遍历对象的所有属性。如果是基本类型，直接赋值；如果是对象或数组，就创建一个新的空对象或数组，并递归调用该函数来复制其内部属性或元素。需要处理 `null`、数组、对象等不同情况，以及考虑循环引用（通常通过一个 `Map` 或 `Set` 来跟踪已访问的对象）。
    *   **优点：** 完全控制拷贝过程，可以定制处理特定类型或逻辑。
    *   **缺点：** 实现起来复杂，容易出错，需要仔细处理各种边界情况（如原型链、特殊对象类型、循环引用）。性能可能不如原生实现。
    *   **适用场景：** 学习目的，或者在没有合适库且 `structuredClone` 不可用/不满足特定需求（如需要拷贝函数或原型）时的备选方案。一般不推荐在生产中手写。

    ```javascript
   /**
 * 手动实现深拷贝函数
 * @param {any} obj 需要深拷贝的对象
 * @param {Map} [memo=new Map()] 用于处理循环引用的缓存，默认为新 Map
 * @returns {any} 深拷贝后的新对象
 */
function deepClone(obj, memo = new Map()) {
  // 1. 处理基本类型和 null，它们直接按值返回
  if (typeof obj !== 'object' || obj === null) {
    return obj;
  }

  // 2. 处理 Date 对象：创建一个新的 Date 实例
  if (obj instanceof Date) {
    return new Date(obj.getTime());
  }

  // 3. 处理 RegExp 对象：创建一个新的 RegExp 实例
  if (obj instanceof RegExp) {
    return new RegExp(obj.source, obj.flags);
  }

  // 4. 处理循环引用：如果对象已经在缓存中，直接返回缓存的克隆对象
  if (memo.has(obj)) {
    return memo.get(obj);
  }

  // 5. 创建新的容器（对象或数组）
  //    使用 obj.constructor 可以尝试保留原型链上的类型信息，
  //    但对于纯粹的数据克隆，Array.isArray 判断通常足够区分核心类型。
  //    这里我们用更通用的方式，但也需注意 constructor 可能被修改。
  //    更安全的基础方式是： let clone = Array.isArray(obj) ? [] : {};
  let clone = new obj.constructor();

  // 6. 将新创建的克隆对象放入缓存中【在遍历其属性之前】
  //    这对于处理循环引用至关重要。
  memo.set(obj, clone);

  // 7. 遍历对象的 *所有* 自有属性（包括 Symbol 类型的 key）
  //    Reflect.ownKeys() 可以获取包括 Symbol 在内的所有自身属性键
  Reflect.ownKeys(obj).forEach(key => {
    // 递归地深拷贝属性值，并将 memo 传递下去
    clone[key] = deepClone(obj[key], memo);
  });

  // 8. 返回克隆后的对象
  return clone;
}

// --- 测试用例 ---

// 包含基本类型、嵌套对象、数组、Date、RegExp
let objA = {
  name: 'Alice',
  age: 30,
  isActive: true,
  metadata: null,
  scores: [10, 20, { subject: 'Math', score: 95 }],
  address: {
    street: '123 Main St',
    city: 'Wonderland'
  },
  birthday: new Date('1993-05-15'),
  pattern: /abc/gi,
  ref: null, // 用于测试循环引用
  symKey: Symbol('desc')
};
objA.ref = objA; // 创建循环引用
objA[objA.symKey] = 'Symbol value'; // 添加 Symbol 键

// 执行深拷贝
let objB = deepClone(objA);
    ```

知识点
* 获取对象属性的方法 [[JS_API_获取对象属性的方法\|JS_API_获取对象属性的方法]]
* 

4.  **使用第三方库 (如 Lodash 的 `_.cloneDeep`)**
    *   **原理：** 利用成熟、经过充分测试的库提供的深拷贝函数。
    *   **优点：** 功能强大、稳定可靠，通常能处理各种复杂数据类型和边界情况（包括函数、原型链、循环引用等，具体看库的实现）。使用简单。
    *   **缺点：** 需要引入外部库，增加项目依赖和打包体积（虽然可以通过按需引入或 Tree Shaking 优化）。
    *   **适用场景：** 在需要健壮、能处理各种复杂情况的深拷贝，且不介意引入库依赖时，这是非常常用和推荐的选择。

    ```javascript
    // 需要先安装 lodash: npm install lodash
    import _ from 'lodash'; // 或者 import { cloneDeep } from 'lodash';

    let objA = { /* ... 包含各种复杂类型和循环引用 ... */ };
    let objDeepLodash = _.cloneDeep(objA);
    // objDeepLodash 是一个完全独立的、包含所有类型（通常包括函数）的副本
    ```

**React/前端中的应用场景**

深拷贝在 React (及其他类似框架) 中至关重要，主要用于：

*   **不可变地更新状态 (Immutability):** React 的状态更新依赖于引用比较来检测变化。直接修改（mutate）现有状态对象/数组，其引用不变，React 可能无法检测到变化，导致 UI 不更新。你需要创建一个*新的*状态对象/数组。==对于嵌套状态，浅拷贝往往不够，因为嵌套部分仍然共享引用。深拷贝确保你得到一个完全独立的新状态副本，可以安全地修改它，然后用 `setState` 更新==。

    ```jsx
    import React, { useState } from 'react';
    import _ from 'lodash'; // 或者使用 structuredClone

    function ProfileEditor() {
      const [profile, setProfile] = useState({
        name: 'Alice',
        address: { city: 'Wonderland', zip: '12345' },
        hobbies: ['reading']
      });

      const handleCityChange = (event) => {
        const newCity = event.target.value;

        // !!! 错误的方式：直接修改状态 !!!
        // profile.address.city = newCity;
        // setProfile(profile); // React 可能不会检测到变化!

        // 正确的方式：深拷贝 -> 修改副本 -> 更新状态
        const newProfile = _.cloneDeep(profile); // 使用 Lodash
        // const newProfile = structuredClone(profile); // 或者使用 structuredClone (如果环境支持且不需要函数等)
        newProfile.address.city = newCity;
        setProfile(newProfile);
      };

      const addHobby = () => {
          const newProfile = _.cloneDeep(profile);
          newProfile.hobbies.push('coding');
          setProfile(newProfile);
      }

      return (
        <div>
          <input
            type="text"
            value={profile.address.city}
            onChange={handleCityChange}
          />
          <button onClick={addHobby}>Add Hobby</button>
          <pre>{JSON.stringify(profile, null, 2)}</pre>
        </div>
      );
    }
    ```

*   **避免副作用：** 当你把一个复杂对象传递给子组件或某个函数时，如果它可能会修改这个对象，而你不希望原始对象被改变，传递一个深拷贝可以防止意外的副作用。

**总结**

*   **浅拷贝**只复制顶层，嵌套引用共享。
*   **深拷贝**递归复制所有层级，完全独立。
*   **`JSON.parse(JSON.stringify())`**简单但局限多，适用于纯 JSON 数据。
*   **`structuredClone()`**是现代首选内置方法，功能强大但有环境和类型限制（不拷函数/原型）。
*   **库 (Lodash `_.cloneDeep`)** 功能全面、可靠，但需引入依赖。
*   在 React 等场景中，为了保证状态更新的**不可变性**，处理嵌套数据时通常需要**深拷贝**。