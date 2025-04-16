---
{"dg-publish":true,"permalink":"/算法_链表_Hash表在两次遍历中的应用_随机链表的复制/"}
---

*   **随机链表的复制 (Copy List with Random Pointer)**
        *   难度：中等
        *   核心：处理 `random` 指针是难点。常用方法有==哈希表==或原地复制（将新节点插入旧节点之后）。
        *   链接：[https://leetcode.cn/problems/copy-list-with-random-pointer/](https://leetcode.cn/problems/copy-list-with-random-pointer/)、



**问题理解 (138. 随机链表的复制)**

我们需要创建一个给定链表的**深拷贝 (Deep Copy)**。这个链表特殊之处在于，每个节点除了有 `next` 指针指向下一个节点外，还有一个 `random` 指针，它可以指向链表中的任意一个节点，或者指向 `null`。

深拷贝意味着：

1.  要创建**全新的**节点，数量与原链表相同。
2.  新节点的值 (`val`) 应与原对应节点的值相同。
3.  新节点的 `next` 指针应指向新链表中对应的下一个节点。
4.  新节点的 `random` 指针应指向新链表中对应的随机目标节点。
5.  **关键**：新链表中的所有指针（`next` 和 `random`）都必须指向**新链表内部的节点**，绝不能指向旧链表中的节点。

**思路：遍历两遍**

*   **第一遍遍历**：
    1.  遍历原始链表。
    2.  对于原始链表中的每一个节点 `originalNode`，创建一个新的节点 `newNode`，并将 `newNode.val` 设置为 `originalNode.val`。
    3.  将 `newNode` 的 `next` 指针连接起来，形成新的链表骨架。
    4.  **核心问题**：在这一步，我们还不能设置 `newNode.random`，因为 `originalNode.random` 指向的那个原始节点，它对应的*新*节点可能还没有被创建，或者即使创建了，我们也不知道它的引用。
    5.  **解决方案**：我==们需要一种方法，在创建新节点时，就能够方便地在之后找到**原始节点**和**新创建节点**之间的对应关系。一个**哈希表 (HashMap / Map)** 是完美的选择，它的键是原始节点，值是对应的新创建节点。 `Map<OriginalNode, NewNode>`==。

*   **第二遍遍历**：
    1.  再次遍历原始链表。
    2.  对于每一个原始节点 `originalNode`：
        *   找到它对应的新节点 `newNode` (通过哈希表查找：`newNode = map.get(originalNode)`)。
        *   查看 `originalNode.random` 指向哪个原始节点 `randomTargetOriginalNode` (如果不是 `null`)。
        *   如果 `randomTargetOriginalNode` 存在，那么通过哈希表找到它对应的新节点 `randomTargetNewNode = map.get(randomTargetOriginalNode)`。
        *   设置 `newNode.random = randomTargetNewNode`。
        *   如果 `originalNode.random` 是 `null`，则设置 `newNode.random = null`。

这种两遍遍历 + 哈希表的思路是解决此问题的标准方法之一，它清晰地分离了节点创建/`next`链接和`random`指针链接两个步骤。

**代码实现 (两遍遍历 + 哈希表)**

```javascript
/**
 * // Definition for a Node.
 * function Node(val, next, random) {
 *    this.val = val;
 *    this.next = next;
 *    this.random = random;
 * };
 */

/**
 * LeetCode 138: 随机链表的复制 (两遍遍历 + 哈希表)
 * @param {Node} head - 原链表的头结点
 * @return {Node} - 深拷贝链表的头结点
 */
var copyRandomList_twoPass = function(head) {
    if (!head) {
        return null; // 处理空链表情况
    }

    // 使用 Map 存储 <原始节点, 新节点> 的映射关系
    const map = new Map();

    // --- 第一遍遍历：创建新节点并连接 next 指针 ---
    let current = head;
    while (current !== null) {
        // 1. 创建新节点，值与原节点相同
        const newNode = new Node(current.val, null, null);
        // 2. 存储映射关系
        map.set(current, newNode);
        // 3. 移动到下一个原始节点
        current = current.next;
    }

    // --- 第二遍遍历：连接新节点的 next 和 random 指针 ---
    current = head;
    while (current !== null) {
        // 1. 获取当前原始节点对应的新节点
        const newNode = map.get(current);

        // 2. 设置新节点的 next 指针
        // 找到原始节点的下一个节点 (originalNext)，然后从 map 中获取它对应的新节点 (newNext)
        const originalNext = current.next;
        const newNext = originalNext ? map.get(originalNext) : null;
        newNode.next = newNext;

        // 3. 设置新节点的 random 指针
        // 找到原始节点的随机指向节点 (originalRandom)，然后从 map 中获取它对应的新节点 (newRandom)
        const originalRandom = current.random;
        const newRandom = originalRandom ? map.get(originalRandom) : null;
        newNode.random = newRandom;

        // 4. 移动到下一个原始节点
        current = current.next;
    }

    // 返回新链表的头节点 (即原始头节点 head 对应的新节点)
    return map.get(head);
};

// --- 辅助 Node 定义 (根据题目描述) ---
function Node(val, next, random) {
   this.val = val;
   this.next = next === undefined ? null : next;
   this.random = random === undefined ? null : random;
}

// --- 辅助函数：创建带随机指针的链表 (用于测试) ---
// 输入是 [[val1, randomIndex1], [val2, randomIndex2], ...]
function createRandomList(arr) {
    if (!arr || arr.length === 0) return null;
    const nodes = [];
    // 创建所有节点
    for (let i = 0; i < arr.length; i++) {
        nodes.push(new Node(arr[i][0]));
    }
    // 连接 next 和 random 指针
    for (let i = 0; i < arr.length; i++) {
        if (i + 1 < arr.length) {
            nodes[i].next = nodes[i+1];
        }
        const randomIndex = arr[i][1];
        if (randomIndex !== null && randomIndex >= 0 && randomIndex < arr.length) {
            nodes[i].random = nodes[randomIndex];
        } else {
            nodes[i].random = null;
        }
    }
    return nodes[0];
}

// --- 辅助函数：打印链表验证 (可选) ---
function printRandomList(head) {
    let curr = head;
    const result = [];
    const nodeToIndex = new Map(); // 用于查找random指向的索引
    let index = 0;
    while(curr) {
        nodeToIndex.set(curr, index++);
        curr = curr.next;
    }

    curr = head;
    index = 0;
     while(curr) {
        const randomIndex = curr.random ? nodeToIndex.get(curr.random) : null;
        result.push([curr.val, randomIndex]);
        curr = curr.next;
    }
    console.log(JSON.stringify(result));

}


// --- 示例测试 ---
console.log("测试用例 1:");
let head1_arr = [[7,null],[13,0],[11,4],[10,2],[1,0]];
let head1 = createRandomList(head1_arr);
console.log("原始链表:");
printRandomList(head1);
let copiedHead1 = copyRandomList_twoPass(head1);
console.log("复制后链表:");
printRandomList(copiedHead1); // 预期输出: [[7,null],[13,0],[11,4],[10,2],[1,0]]

console.log("\n测试用例 2:");
let head2_arr = [[1,1],[2,1]];
let head2 = createRandomList(head2_arr);
console.log("原始链表:");
printRandomList(head2);
let copiedHead2 = copyRandomList_twoPass(head2);
console.log("复制后链表:");
printRandomList(copiedHead2); // 预期输出: [[1,1],[2,1]]

console.log("\n测试用例 3:");
let head3_arr = [[3,null],[3,0],[3,null]];
let head3 = createRandomList(head3_arr);
console.log("原始链表:");
printRandomList(head3);
let copiedHead3 = copyRandomList_twoPass(head3);
console.log("复制后链表:");
printRandomList(copiedHead3); // 预期输出: [[3,null],[3,0],[3,null]]

console.log("\n测试用例 4 (空链表):");
let head4 = createRandomList([]);
console.log("原始链表: []");
let copiedHead4 = copyRandomList_twoPass(head4);
console.log("复制后链表: []"); // 预期输出: []
```

**
**复杂度分析 (两遍遍历 + 哈希表)**

*   **时间复杂度**: O(N)，其中 N 是链表节点的数量。我们需要遍历链表两次，每次遍历都是 O(N)。哈希表的插入和查找操作平均时间复杂度是 O(1)。所以总时间复杂度是 O(N) + O(N) = O(N)。
*   **空间复杂度**: O(N)。我们需要一个哈希表来存储原始节点到新节点的映射，最坏情况下需要存储 N 个映射关系，所以空间复杂度是 O(N)。

**优化思路：O(1) 空间复杂度的方法 (原地修改 + 三遍遍历)**

存在一种更巧妙的方法，可以避免使用额外的哈希表，从而达到 O(1) 的空间复杂度（如果我们不把创建新链表本身所需的空间算作额外空间的话）。

1.  **第一遍遍历：复制并交织节点**
    *   遍历原始链表。
    *   对于每个节点 `originalNode`，创建一个新节点 `newNode` (`newNode.val = originalNode.val`)。
    *   将 `newNode` 插入到 `originalNode` 和 `originalNode.next` 之间。
    *   即：`newNode.next = originalNode.next;`
    *   `originalNode.next = newNode;`
    *   完成后，链表变成 `O1 -> N1 -> O2 -> N2 -> O3 -> N3 -> ...` 的形式（O 代表原始，N 代表新）。

2.  **第二遍遍历：复制 `random` 指针**
    *   再次遍历这个交织的链表，但只关注原始节点 (`current = head`)。
    *   对于每个原始节点 `current`，它对应的新节点就是 `current.next`。
    *   如果 `current.random` 存在，那么 `current.random` 指向的原始节点 `targetOriginal`，它对应的新节点就是 `targetOriginal.next`。
    *   所以，设置新节点的 `random` 指针：`current.next.random = current.random ? current.random.next : null;`
    *   移动 `current` 到下一个原始节点：`current = current.next.next;`

3.  **第三遍遍历：拆分链表**
    *   将交织的链表拆分成原始链表和新链表。
    *   维护原始链表的指针 `originalPtr` 和新链表的指针 `newPtr`（以及新链表的头 `newHead`）。
    *   遍历交织链表，将 `originalNode.next` 指回它本来的下一个原始节点，将 `newNode.next` 指向它本来的下一个新节点。
    *   `originalPtr.next = newPtr.next;`
    *   `newPtr.next = (newPtr.next != null) ? newPtr.next.next : null;`
    *   移动指针 `originalPtr = originalPtr.next;` 和 `newPtr = newPtr.next;`

**代码实现 (O(1) 空间优化)**

```javascript
/**
 * LeetCode 138: 随机链表的复制 (O(1) 空间优化)
 * @param {Node} head - 原链表的头结点
 * @return {Node} - 深拷贝链表的头结点
 */
var copyRandomList_optimized = function(head) {
    if (!head) {
        return null;
    }

    // --- 第一遍遍历：复制节点并插入到原节点后面 ---
    let current = head;
    while (current !== null) {
        const newNode = new Node(current.val);
        newNode.next = current.next; // 新节点的 next 指向原节点的下一个
        current.next = newNode;      // 原节点的 next 指向新节点
        current = newNode.next;      // 移动到下一个原节点
    }

    // --- 第二遍遍历：复制 random 指针 ---
    current = head;
    while (current !== null) {
        // current 是原节点, current.next 是对应的新节点
        if (current.random !== null) {
            // 新节点的 random 指向 原节点 random 指向的节点的下一个节点(即对应的新节点)
            current.next.random = current.random.next;
        } else {
            current.next.random = null;
        }
        // 移动到下一个原节点 (跳过新节点)
        current = current.next.next;
    }

    // --- 第三遍遍历：拆分两个链表 ---
    let originalCurrent = head;        // 指向原链表的当前节点
    let newHead = head.next;         // 新链表的头节点
    let newCurrent = newHead;          // 指向新链表的当前节点

    while (originalCurrent !== null) {
        // 1. 恢复原链表的 next 指针
        originalCurrent.next = newCurrent.next;

        // 2. 设置新链表的 next 指针 (需要检查 newCurrent.next 是否为空)
        newCurrent.next = (newCurrent.next !== null) ? newCurrent.next.next : null;

        // 3. 移动指针
        originalCurrent = originalCurrent.next; // 移动到下一个原节点
        newCurrent = newCurrent.next;         // 移动到下一个新节点
    }

    return newHead;
};

// --- 测试 O(1) 空间方法 ---
console.log("\n--- O(1) 空间复杂度方法测试 ---");
console.log("测试用例 1:");
head1 = createRandomList([[7,null],[13,0],[11,4],[10,2],[1,0]]);
copiedHead1_opt = copyRandomList_optimized(head1);
console.log("原始链表 (应未被永久改变):");
printRandomList(head1); // 检查原链表是否恢复
console.log("复制后链表:");
printRandomList(copiedHead1_opt); // 预期输出: [[7,null],[13,0],[11,4],[10,2],[1,0]]

console.log("\n测试用例 2:");
head2 = createRandomList([[1,1],[2,1]]);
copiedHead2_opt = copyRandomList_optimized(head2);
console.log("原始链表 (应未被永久改变):");
printRandomList(head2);
console.log("复制后链表:");
printRandomList(copiedHead2_opt); // 预期输出: [[1,1],[2,1]]
```

**
**相关知识点扩展**

1.  **深拷贝 (Deep Copy) vs. 浅拷贝 (Shallow Copy)**:
    *   浅拷贝只复制对象的引用（指针），不复制对象本身。修改拷贝后的对象会影响原始对象（如果它们指向同一块内存）。
    *   深拷贝会创建全新的对象，并递归地复制所有嵌套的对象，使得拷贝后的对象与原始对象完全独立。对于链表，这意味着要创建全新的节点。
2.  **哈希表/Map**: 非常重要的数据结构，提供键值对存储，支持快速（平均 O(1)）的插入、删除和查找。在需要快速查找对应关系时非常有用。
3.  **链表原地操作 (In-place Manipulation)**: 像 O(1) 空间复杂度的解法那样，直接在原始链表结构上进行修改和操作，以避免使用额外的存储空间。这通常需要更精巧的指针操作。
4.  **指针操作**: 链表问题的核心就是指针的理解和操作，包括如何遍历、如何断开连接、如何建立新连接、如何处理边界条件（头节点、尾节点、空链表）。

