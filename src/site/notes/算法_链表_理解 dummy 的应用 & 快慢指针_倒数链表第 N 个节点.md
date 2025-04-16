---
{"dg-publish":true,"permalink":"/算法_链表_理解 dummy 的应用 & 快慢指针_倒数链表第 N 个节点/"}
---

*   **删除链表的倒数第 N 个结点 (Remove Nth Node From End of List)**
        *   难度：中等
        *   核心：快慢指针技巧的入门应用。理解相对运动。
        *   链接：[https://leetcode.cn/problems/remove-nth-node-from-end-of-list/](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)

[[算法_链表_dummy节点的应用\|算法_链表_dummy节点的应用]]


```javascript
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} head
 * @param {number} n
 * @return {ListNode}
 */
var removeNthFromEnd = function(head, n) {
    // 创建一个虚拟头节点，指向实际的头节点
    // 这使得删除头节点的操作和删除其他节点的操作逻辑一致
    let dummy = new ListNode(0, head);

    // 初始化快慢指针都指向虚拟头节点
    // slow 最终需要停在待删除节点的前一个节点
    let slow = dummy;
    let fast = dummy; // 让 fast 也从 dummy 开始，方便计算差距

    // 快指针先向前移动 n+1 步
    // 这样快慢指针之间就相差 n 个节点
    // 当 fast 到达链表末尾 null 时，slow 正好在倒数第 n+1 个节点（即倒数第 n 个节点的前一个）
    for (let i = 0; i <= n; i++) {
        // 如果 n 大于链表长度（虽然题目约束通常不会，但健壮性考虑），fast 可能提前变 null
        if (!fast) {
            // 根据题目说明，n 始终有效，这里理论上不会执行
            // 但可以加个判断以防万一或处理更宽泛的约束
            return head; // 或者根据具体要求处理无效 n
        }
        fast = fast.next;
    }

    // 快慢指针同步向后移动，直到快指针到达链表末尾 (null)
    while (fast) {
        fast = fast.next;
        slow = slow.next;
    }

    // 此时 slow 指向的是待删除节点的前一个节点
    // 执行删除操作：跳过待删除节点
    slow.next = slow.next.next;

    // 返回虚拟头节点的下一个节点，即为新链表的头节点
    return dummy.next;
};

// --- 辅助代码：用于测试 ---
function ListNode(val, next) {
    this.val = (val === undefined ? 0 : val);
    this.next = (next === undefined ? null : next);
}

function createLinkedList(arr) {
    if (!arr || arr.length === 0) return null;
    let head = new ListNode(arr[0]);
    let current = head;
    for (let i = 1; i < arr.length; i++) {
        current.next = new ListNode(arr[i]);
        current = current.next;
    }
    return head;
}

function printLinkedList(head) {
    let current = head;
    let result = [];
    while (current) {
        result.push(current.val);
        current = current.next;
    }
    console.log(result.join(" -> "));
}

// 测试用例 1: 删除中间节点
let head1 = createLinkedList([1, 2, 3, 4, 5]);
console.log("Original 1:");
printLinkedList(head1);
let newHead1 = removeNthFromEnd(head1, 2); // 删除倒数第 2 个节点 (4)
console.log("After removing Nth from end (n=2):");
printLinkedList(newHead1); // 输出: 1 -> 2 -> 3 -> 5

console.log("---");

// 测试用例 2: 删除头节点
let head2 = createLinkedList([1]);
console.log("Original 2:");
printLinkedList(head2);
let newHead2 = removeNthFromEnd(head2, 1); // 删除倒数第 1 个节点 (1)
console.log("After removing Nth from end (n=1):");
printLinkedList(newHead2); // 输出: (空)

console.log("---");

// 测试用例 3: 删除头节点（多节点）
let head3 = createLinkedList([1, 2]);
console.log("Original 3:");
printLinkedList(head3);
let newHead3 = removeNthFromEnd(head3, 2); // 删除倒数第 2 个节点 (1)
console.log("After removing Nth from end (n=2):");
printLinkedList(newHead3); // 输出: 2
```

**总结关键修正点：**

1.  **引入 `dummy` 节点**：`let dummy = new ListNode(0, head);`
2.  **`slow` 指针从 `dummy` 开始**：`let slow = dummy;`
3.  **`fast` 指针也从 `dummy` 开始（或调整步数）**：让 `fast` 指针比 `slow` 指针领先 `n+1` 步。一种方法是 `fast = dummy` 然后循环 `n+1` 次 `fast = fast.next`；另一种是 `fast = head` 然后循环 `n` 次 `fast = fast.next`。修改后的代码采用了前者（稍作调整，循环 `n+1` 次）。
    *   **修正版代码的逻辑**: `fast` 从 `dummy` 开始，先走 `n+1` 步。这样，当 `fast` 最终走到 `null` 时，`slow` 正好停在待删除节点的前一个节点。
4.  **循环条件**： `while (fast)` ，因为当 `fast` 为 `null` 时，它已经到达了链表末尾的下一个位置。
5.  **返回结果**：返回 `dummy.next` 而不是 `head`，因为原始的 `head` 可能已经被删除了。

