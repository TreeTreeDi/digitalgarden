---
{"dg-publish":true,"permalink":"/算法_数据结构_hash_单位时间内插入删除/"}
---

#review  
```ad-note
![Pasted image 20250312154648.png](/img/user/%E9%99%84%E4%BB%B6/Pasted%20image%2020250312154648.png)
思路 1. 覆盖掉原先的 val 添加 lastVal 索引 2. 删除lastVal 去掉val的索引 
```



### 第三步：动手实现
以下是用 JavaScript 实现的完整代码，我会边写边解释。

```javascript
class RandomizedSet {
  constructor() {
    this.arr = []; // 存储元素
    this.map = new Map(); // 记录元素到索引的映射
  }

  // 插入元素，返回是否成功
  insert(val) {
    // 如果元素已存在，返回 false
    if (this.map.has(val)) {
      return false;
    }
    // 加到数组尾部
    this.arr.push(val);
    // 记录索引（尾部索引是长度-1）
    this.map.set(val, this.arr.length - 1);
    return true;
  }

  // 删除元素，返回是否成功
  remove(val) {
    // 如果元素不存在，返回 false
    if (!this.map.has(val)) {
      return false;
    }
    // 获取目标元素的索引
    const index = this.map.get(val);
    // 获取最后一个元素
    const lastVal = this.arr[this.arr.length - 1];
    
    // 如果目标不是最后一个，将最后一个元素移到目标位置
    if (index !== this.arr.length - 1) {
      this.arr[index] = lastVal; // 覆盖目标位置
      this.map.set(lastVal, index); // 更新最后一个元素的索引
    }
    
    // 删除尾部元素
    this.arr.pop();
    this.map.delete(val); // 从哈希表移除目标元素
    return true;
  }

  // 随机获取一个元素
  getRandom() {
    // 用 Math.random() 生成 0 到 1 的随机数，乘以数组长度并取整
    const randomIndex = Math.floor(Math.random() * this.arr.length);
    return this.arr[randomIndex];
  }
}
```

#### 逐步解释
1. **初始化**：
   - `arr` 是动态数组，存元素。
   - `map` 是哈希表，存 `{元素: 索引}` 键值对。

2. **insert(val)**：
   - 用 `map.has(val)` 检查是否重复，O(1)。
   - 如果不重复，`arr.push(val)` 是 O(1)，`map.set(val, index)` 也是 O(1)。
   - 返回 `true` 表示插入成功。

3. **remove(val)**：
   - 用 `map.has(val)` 检查元素是否存在，O(1)。
   - 找到 `val` 的索引，将最后一个元素移到这个位置（覆盖它），更新 `map` 中最后一个元素的索引。
   - 删除尾部（`arr.pop()` 是 O(1)），从 `map` 中移除 `val`（O(1)）。
   - 为什么交换？直接删除中间元素需要移动后续元素（O(n)），交换后只删尾部就行了。

4. **getRandom()**：
   - `Math.random()` 生成 0 到 1 的随机数，乘以数组长度并取整，得到随机索引。
   - 直接返回 `arr[randomIndex]`，O(1)。