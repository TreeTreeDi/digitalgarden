---
{"dg-publish":true,"permalink":"/react_核心概念_声明式_为什么要使用声明式 React 替代命令式 Dom/"}
---

## LiveKit 远程媒体流渲染：命令式 DOM vs 声明式 React



## 1. 引言

*   背景：在 React 应用中集成 LiveKit 并实时显示远程参与者的视频/音频流。
*   挑战：LiveKit SDK 通过异步事件通知流的订阅/取消订阅，如何将其与 React 的渲染机制结合？
*   两种方法概述：
    *   **命令式方法：** ==直接在事件回调中操作浏览器 DOM==。
    *   **声明式方法：** ==利用 React 状态和组件生命周期管理==。
*   本文目的：对比分析这两种方法，解释为何声明式方法更适合 React 应用。

#review 
## 2. 方法一：命令式 DOM 操作 (旧方法)

### 2.1 原理

*   监听 LiveKit 的 `RoomEvent.TrackSubscribed` 和 `RoomEvent.TrackUnsubscribed` 事件。
*   在事件回调函数中：
    *   使用 `document.getElementById` 查找预先渲染好的、用于容纳媒体元素的父容器 (`div`)。
    *   调用 `track.attach()` 创建媒体元素 (`<video>` 或 `<audio>`)。
    *   使用 `appendChild` 将媒体元素添加到父容器中。
    *   在取消订阅时，使用 `track.detach()` 分离媒体元素，并用 `removeChild` 将其从 DOM 中移除。

[[浏览器_API_video_liveKit 与 HTML 集成\|浏览器_API_video_liveKit 与 HTML 集成]]

### 2.2 代码示例 (关键片段)

```javascript
// 旧 handleTrackSubscribed 核心逻辑 (示意)
function handleTrackSubscribed(track, publication, participant) {
  const containerId = `participant-${participant.sid}`;
  // 1. 查找容器 (可能在 React 渲染完成前执行)
  const container = document.getElementById(containerId);
  if (container) {
    // 2. 创建并附加媒体元素
    const element = track.attach();
    element.id = `track-${track.sid}`;
    // 3. 添加到 DOM
    container.appendChild(element);
  } else {
    console.error(`Container ${containerId} not found!`); // <--- 问题点
  }
}

// 旧 handleTrackUnsubscribed 核心逻辑 (示意)
function handleTrackUnsubscribed(track, publication, participant) {
  const elementId = `track-${track.sid}`;
  const element = document.getElementById(elementId);
  if (element) {
    track.detach(element);
    element.remove();
  }
}
```

### 2.3 问题分析：为何与 React 冲突？

*   **命令式 vs 声明式:** React 的核心是声明式编程，开发者描述 UI *应该是什么样子* (基于 state)，==React 负责更新 DOM。而直接操作 DOM 是命令式的，开发者直接指挥浏览器 *如何去做*==。这绕过了 React 的控制，破坏了 React 的抽象。
*   **React 的虚拟 DOM 与协调 (Reconciliation):**
    *   简述 React 使用 Virtual DOM 来提高渲染性能。
    *   当状态改变时，React 会计算出新旧 Virtual DOM 的差异 (Diffing)。
    *   然后，React 将这些差异以最优化的方式应用到真实 DOM 上 (Reconciliation/协调)。
*   **时序竞态 (Race Condition):**
    *   LiveKit 事件是异步的。`TrackSubscribed` 事件可能在 `setRemoteParticipants` 状态更新之后、但在 React 完成协调过程并将新的参与者 `div` 渲染到真实 DOM **之前**触发。
    *   此时 `getElementById` 就会因为找不到容器而返回 `null`。

    ```mermaid
    sequenceDiagram
        participant User as 用户操作
        participant LK as LiveKit SDK
        participant React as React App
        participant DOM as 浏览器 DOM

        User->>React: 加入房间/参与者加入
        React->>React: setRemoteParticipants(newState)
        React->>React: 开始协调 (Reconciliation)
        LK-->>React: 触发 TrackSubscribed 事件
        React->>DOM: (旧方法) getElementById('participant-XYZ')
        Note right of DOM: 此时协调未完成, div 不存在
        DOM-->>React: 返回 null
        React->>React: 附加失败!
        React->>DOM: (稍后) 完成协调, 渲染 div
    ```
*   **维护性与风险:**
    *   代码分散，状态不明确（真实 DOM 成了隐式状态来源）。
    *   手动管理 DOM 可能导致忘记清理（如 `detach` 或 `removeChild`），引发内存泄漏。
    *   与 React 的生命周期和状态管理脱节，难以调试。

## 3. 方法二：声明式 React 状态与组件 (新方法)

### 3.1 原理

*   **状态驱动 UI:** 将远程轨道的存在视为 React 组件的状态 (`remoteTracks`)。
*   **事件更新状态:** LiveKit 事件回调仅负责更新此状态 (`setRemoteTracks`)。
*   **React 负责渲染:** React 检测到状态变化后，自动重新渲染 UI。
*   **组件封装:** 将单个轨道的渲染逻辑（包括 `attach`/`detach`）封装在独立的 `ParticipantTrack` 组件中。
*   **Hooks 管理副作用:** 在 `ParticipantTrack` 组件内部，使用 `useEffect` Hook 来处理 `attach` 和 `detach` 这种与外部系统交互的副作用 (Side Effect)，并利用 `useRef` 获取对 DOM 节点的引用。

### 3.2 核心要素

*   **状态管理 (`useState`)**

    ```javascript
    // Map<ParticipantSID, RemoteTrack[]>
    const [remoteTracks, setRemoteTracks] = useState<Map<string, RemoteTrack[]>>(new Map());
    ```
*   **事件处理 (`onTrackSubscribed`/`Unsubscribed`)**

    ```javascript
    // 新 handleTrackSubscribed (只更新状态)
    const handleTrackSubscribed = (track, publication, participant) => {
      setRemoteTracks((prevTracks) => {
        const newTracksMap = new Map(prevTracks);
        const participantTracks = newTracksMap.get(participant.sid) || [];
        if (!participantTracks.some(t => t.sid === track.sid)) {
          newTracksMap.set(participant.sid, [...participantTracks, track]);
        }
        return newTracksMap;
      });
    };

    // 新 handleTrackUnsubscribed (只更新状态)
    const handleTrackUnsubscribed = (track, publication, participant) => {
      setRemoteTracks((prevTracks) => {
        const newTracksMap = new Map(prevTracks);
        const participantTracks = newTracksMap.get(participant.sid) || [];
        const filteredTracks = participantTracks.filter(t => t.sid !== track.sid);
        if (filteredTracks.length === 0) {
            newTracksMap.delete(participant.sid);
        } else {
            newTracksMap.set(participant.sid, filteredTracks);
        }
        return newTracksMap;
      });
    };
    ```
*   **轨道渲染组件 (`ParticipantTrack`)**

    ```javascript
    function ParticipantTrack({ track }: { track: RemoteTrack }) {
      const containerRef = useRef<HTMLDivElement>(null);

      useEffect(() => {
        const container = containerRef.current;
        if (!container) return; // 等待容器渲染完成

        // 副作用：附加媒体元素
        const element = track.attach();
        container.appendChild(element);

        // 清理函数：在组件卸载或 track 变化时分离
        return () => {
          track.detach(element);
          if (container.contains(element)) {
              container.removeChild(element);
          }
        };
      }, [track]); // 依赖项数组，确保 track 变化时重新执行

      // 渲染一个空的 div 作为容器，React 负责渲染它
      return <div ref={containerRef}></div>;
    }
    ```
    *   **`useRef` 的作用:** 提供一个持久化的引用，指向由 React 渲染的真实 DOM 节点 (`div`)。
    *   **`useEffect` 的作用:**
        *   将包含副作用（与 LiveKit SDK 交互，操作 DOM）的代码与 React 的渲染过程分离。
        *   保证副作用代码在组件**挂载到 DOM 后**执行，此时 `containerRef.current` 肯定有效。
        *   提供清理机制，确保在不再需要时（组件卸载或 `track` prop 改变）能正确 `detach`。
*   **父组件渲染逻辑**

    ```javascript
    // 在 LiveKitTestPage 组件的 return 语句中
    {Array.from(remoteParticipants.values()).map(participant => (
      <div key={participant.sid}>
        <p>{participant.identity}</p>
        {/* 根据 remoteTracks 状态渲染 ParticipantTrack */}
        {(remoteTracks.get(participant.sid) || []).map(track => (
          <ParticipantTrack key={track.sid} track={track} />
        ))}
      </div>
    ))}
    ```

### 3.3 优势分析

*   **遵循 React 范式:** 代码完全符合 React 的声明式、组件化和状态驱动思想。
*   **解决时序问题:** `useEffect` 确保了 DOM 操作发生在 React 渲染之后，从根本上避免了竞态条件。

    ```mermaid
    sequenceDiagram
        participant User as 用户操作
        participant LK as LiveKit SDK
        participant React as React App
        participant DOM as 浏览器 DOM

        User->>React: 加入房间/参与者加入
        React->>React: setRemoteParticipants(newState)
        React->>React: 开始协调 (Reconciliation)
        LK-->>React: 触发 TrackSubscribed 事件
        React->>React: setRemoteTracks(newState)
        React->>React: (可能再次) 开始协调
        React->>DOM: 完成协调, 渲染参与者 div 和 ParticipantTrack 的 div (ref 指向它)
        React->>React: ParticipantTrack 的 useEffect 执行
        React->>LK: track.attach()
        LK-->>React: 返回 media element
        React->>DOM: containerRef.current.appendChild(element)
        Note right of DOM: 此时 containerRef.current 有效
        DOM-->>React: 附加成功
    ```
*   **提升可维护性与可读性:** 逻辑清晰，状态管理集中，`ParticipantTrack` 组件职责单一。
*   **自动资源管理:** 利用 `useEffect` 的清理函数，React 会自动帮助我们管理 `detach` 操作，减少内存泄漏风险。

## 4. 总结

*   直接在 React 应用中命令式地操作 DOM 通常是反模式的，尤其是在处理异步事件时，容易引发难以调试的时序问题。
*   利用 React 的状态管理 (`useState`) 和副作用处理机制 (`useEffect`, `useRef`)，将外部 SDK 的事件转化为状态变更，让 React 来驱动 UI 更新，是更健壮、更符合 React 思想的解决方案。
*   这种声明式的方法提高了代码的可维护性、可读性，并能更好地利用 React 的性能优化和生命周期管理能力。
