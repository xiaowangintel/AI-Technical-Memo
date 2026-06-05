# request_queue.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/core/sched/request_queue.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `SchedulingPolicy`, `RequestQueue`, `FCFSRequestQueue` for the V1 `core/sched` subsystem. / 为 V1 的 `core/sched` 子系统实现 `SchedulingPolicy`, `RequestQueue`, `FCFSRequestQueue`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import heapq
from abc import ABC, abstractmethod
from collections import deque
from collections.abc import Iterable, Iterator
from enum import Enum

from vllm.v1.request import Request
```
**EN:** Sets up the module namespace with standard-library helpers, and internal vLLM modules such as `vllm.v1.request`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具，以及 `vllm.v1.request` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `SchedulingPolicy` class / `SchedulingPolicy` 类
```python
class SchedulingPolicy(Enum):
    """Enum for scheduling policies."""

    FCFS = "fcfs"
    PRIORITY = "priority"
```
**EN:** Defines the `SchedulingPolicy` enum used to normalize modes or options across the subsystem. Representative members: `FCFS`, `PRIORITY`.
**CN:** `SchedulingPolicy` 定义了一个枚举，用于统一子系统中的模式或选项。代表性成员包括：`FCFS`, `PRIORITY`。

### `RequestQueue` class / `RequestQueue` 类
```python
class RequestQueue(ABC):
    """Abstract base class for request queues."""
```
**EN:** Declares the `RequestQueue` interface. Downstream implementations are expected to provide methods such as `add_request`, `pop_request`, `peek_request`, `prepend_request`, `prepend_requests`, `remove_request`.
**CN:** `RequestQueue` 声明了一组接口约定。下游实现需要提供 `add_request`, `pop_request`, `peek_request`, `prepend_request`, `prepend_requests`, `remove_request` 等方法。

### `RequestQueue.add_request` method / `RequestQueue.add_request` 方法
```python
    @abstractmethod
    def add_request(self, request: Request) -> None:
        """Add a request to the queue according to the policy."""
        pass
```
**EN:** This method implements `add_request` within `RequestQueue`. The docstring frames it as: Add a request to the queue according to the policy.
**CN:** 该方法会实现 `add_request`，其作用域位于`RequestQueue`。

### `RequestQueue.pop_request` method / `RequestQueue.pop_request` 方法
```python
    @abstractmethod
    def pop_request(self) -> Request:
        """Pop a request from the queue according to the policy."""
        pass
```
**EN:** This method implements `pop_request` within `RequestQueue`. The docstring frames it as: Pop a request from the queue according to the policy.
**CN:** 该方法会实现 `pop_request`，其作用域位于`RequestQueue`。

### `RequestQueue.peek_request` method / `RequestQueue.peek_request` 方法
```python
    @abstractmethod
    def peek_request(self) -> Request:
        """Peek at the request at the front of the queue without removing it."""
        pass
```
**EN:** This method implements `peek_request` within `RequestQueue`. The docstring frames it as: Peek at the request at the front of the queue without removing it.
**CN:** 该方法会实现 `peek_request`，其作用域位于`RequestQueue`。

### `RequestQueue.prepend_request` method / `RequestQueue.prepend_request` 方法
```python
    @abstractmethod
    def prepend_request(self, request: Request) -> None:
        """Prepend a request to the front of the queue."""
        pass
```
**EN:** This method implements `prepend_request` within `RequestQueue`. The docstring frames it as: Prepend a request to the front of the queue.
**CN:** 该方法会实现 `prepend_request`，其作用域位于`RequestQueue`。

### `RequestQueue.prepend_requests` method / `RequestQueue.prepend_requests` 方法
```python
    @abstractmethod
    def prepend_requests(self, requests: "RequestQueue") -> None:
        """Prepend all requests from another queue to the front of this
        queue."""
        pass
```
**EN:** This method implements `prepend_requests` within `RequestQueue`. The docstring frames it as: Prepend all requests from another queue to the front of this queue.
**CN:** 该方法会实现 `prepend_requests`，其作用域位于`RequestQueue`。

### `RequestQueue.remove_request` method / `RequestQueue.remove_request` 方法
```python
    @abstractmethod
    def remove_request(self, request: Request) -> None:
        """Remove a specific request from the queue."""
        pass
```
**EN:** This method implements `remove_request` within `RequestQueue`. The docstring frames it as: Remove a specific request from the queue.
**CN:** 该方法会实现 `remove_request`，其作用域位于`RequestQueue`。

### `RequestQueue.remove_requests` method / `RequestQueue.remove_requests` 方法
```python
    @abstractmethod
    def remove_requests(self, requests: Iterable[Request]) -> None:
        """Remove multiple specific requests from the queue."""
        pass
```
**EN:** This method implements `remove_requests` within `RequestQueue`. The docstring frames it as: Remove multiple specific requests from the queue.
**CN:** 该方法会实现 `remove_requests`，其作用域位于`RequestQueue`。

### `RequestQueue.__bool__` method / `RequestQueue.__bool__` 方法
```python
    @abstractmethod
    def __bool__(self) -> bool:
        """Check if queue has any requests."""
        pass
```
**EN:** This method implements `__bool__` within `RequestQueue`. The docstring frames it as: Check if queue has any requests.
**CN:** 该方法会实现 `__bool__`，其作用域位于`RequestQueue`。

### `FCFSRequestQueue` class / `FCFSRequestQueue` 类
```python
class FCFSRequestQueue(deque[Request], RequestQueue):
    """A first-come-first-served queue that supports deque operations."""
```
**EN:** Introduces the `FCFSRequestQueue` class on top of `deque[Request]`, `RequestQueue`. Core methods include `add_request`, `pop_request`, `peek_request`, `prepend_request`, `prepend_requests`, `remove_request`. Docstring signal: A first-come-first-served queue that supports deque operations.
**CN:** 这里定义 `FCFSRequestQueue` 类，其基类包括 `deque[Request]`, `RequestQueue`。核心方法包括 `add_request`, `pop_request`, `peek_request`, `prepend_request`, `prepend_requests`, `remove_request`。

### `FCFSRequestQueue.add_request` method / `FCFSRequestQueue.add_request` 方法
```python
    def add_request(self, request: Request) -> None:
        """Add a request to the queue according to FCFS policy."""
        self.append(request)
```
**EN:** This method implements `add_request` within `FCFSRequestQueue`. The docstring frames it as: Add a request to the queue according to FCFS policy. Key calls include `append`.
**CN:** 该方法会实现 `add_request`，其作用域位于`FCFSRequestQueue`。 关键调用包括 `append`。

### `FCFSRequestQueue.pop_request` method / `FCFSRequestQueue.pop_request` 方法
```python
    def pop_request(self) -> Request:
        """Pop a request from the queue according to FCFS policy."""
        return self.popleft()
```
**EN:** This method implements `pop_request` within `FCFSRequestQueue`. The docstring frames it as: Pop a request from the queue according to FCFS policy. Key calls include `popleft`.
**CN:** 该方法会实现 `pop_request`，其作用域位于`FCFSRequestQueue`。 关键调用包括 `popleft`。

### `FCFSRequestQueue.peek_request` method / `FCFSRequestQueue.peek_request` 方法
```python
    def peek_request(self) -> Request:
        """Peek at the next request in the queue without removing it."""
        if not self:
            raise IndexError("peek from an empty queue")
        return self[0]
```
**EN:** This method implements `peek_request` within `FCFSRequestQueue`. The docstring frames it as: Peek at the next request in the queue without removing it. Key calls include `IndexError`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `peek_request`，其作用域位于`FCFSRequestQueue`。 关键调用包括 `IndexError`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FCFSRequestQueue.prepend_request` method / `FCFSRequestQueue.prepend_request` 方法
```python
    def prepend_request(self, request: Request) -> None:
        """Prepend a request to the front of the queue."""
        self.appendleft(request)
```
**EN:** This method implements `prepend_request` within `FCFSRequestQueue`. The docstring frames it as: Prepend a request to the front of the queue. Key calls include `appendleft`.
**CN:** 该方法会实现 `prepend_request`，其作用域位于`FCFSRequestQueue`。 关键调用包括 `appendleft`。

### `FCFSRequestQueue.prepend_requests` method / `FCFSRequestQueue.prepend_requests` 方法
```python
    def prepend_requests(self, requests: RequestQueue) -> None:
        """Prepend all requests from another queue to the front of this
        queue.

        Note: The requests will be prepended in reverse order of their
        appearance in the `requests` queue.
        """
        self.extendleft(requests)
```
**EN:** This method implements `prepend_requests` within `FCFSRequestQueue`. The docstring frames it as: Prepend all requests from another queue to the front of this queue. Key calls include `extendleft`.
**CN:** 该方法会实现 `prepend_requests`，其作用域位于`FCFSRequestQueue`。 关键调用包括 `extendleft`。

### `FCFSRequestQueue.remove_request` method / `FCFSRequestQueue.remove_request` 方法
```python
    def remove_request(self, request: Request) -> None:
        """Remove a specific request from the queue."""
        self.remove(request)
```
**EN:** This method implements `remove_request` within `FCFSRequestQueue`. The docstring frames it as: Remove a specific request from the queue. Key calls include `remove`.
**CN:** 该方法会实现 `remove_request`，其作用域位于`FCFSRequestQueue`。 关键调用包括 `remove`。

### `FCFSRequestQueue.remove_requests` method / `FCFSRequestQueue.remove_requests` 方法
```python
    def remove_requests(self, requests: Iterable[Request]) -> None:
        """Remove multiple specific requests from the queue."""
        requests_to_remove = set(requests)
        filtered_requests = [req for req in self if req not in requests_to_remove]
        # deque does not support in-place filtering, so we need to clear
        # and extend
        self.clear()
        self.extend(filtered_requests)
```
**EN:** This method implements `remove_requests` within `FCFSRequestQueue`. The docstring frames it as: Remove multiple specific requests from the queue. Key calls include `set`, `clear`, `extend`.
**CN:** 该方法会实现 `remove_requests`，其作用域位于`FCFSRequestQueue`。 关键调用包括 `set`, `clear`, `extend`。

### `FCFSRequestQueue.__bool__` method / `FCFSRequestQueue.__bool__` 方法
```python
    def __bool__(self) -> bool:
        """Check if queue has any requests."""
        return len(self) > 0
```
**EN:** This method implements `__bool__` within `FCFSRequestQueue`. The docstring frames it as: Check if queue has any requests. Key calls include `len`.
**CN:** 该方法会实现 `__bool__`，其作用域位于`FCFSRequestQueue`。 关键调用包括 `len`。

### `PriorityRequestQueue` class / `PriorityRequestQueue` 类
```python
class PriorityRequestQueue(RequestQueue):
    """
    A priority queue that supports heap operations.

    Respects the ordering defined in the Request class, where
    requests with a smaller value of `priority` are processed first.
    If multiple requests have the same priority, the one with the earlier
    `arrival_time` is processed first.
    """
```
**EN:** Introduces the `PriorityRequestQueue` class on top of `RequestQueue`. Core methods include `__init__`, `add_request`, `pop_request`, `peek_request`, `prepend_request`, `prepend_requests`. Docstring signal: A priority queue that supports heap operations.
**CN:** 这里定义 `PriorityRequestQueue` 类，其基类包括 `RequestQueue`。核心方法包括 `__init__`, `add_request`, `pop_request`, `peek_request`, `prepend_request`, `prepend_requests`。

### `PriorityRequestQueue.__init__` method / `PriorityRequestQueue.__init__` 方法
```python
    def __init__(self) -> None:
        self._heap: list[Request] = []
```
**EN:** This method initializes the object state within `PriorityRequestQueue`. It touches state such as `_heap`.
**CN:** 该方法会初始化对象状态，其作用域位于`PriorityRequestQueue`。 它会读写 `_heap` 等状态。

### `PriorityRequestQueue.add_request` method / `PriorityRequestQueue.add_request` 方法
```python
    def add_request(self, request: Request) -> None:
        """Add a request to the queue according to priority policy."""
        heapq.heappush(self._heap, request)
```
**EN:** This method implements `add_request` within `PriorityRequestQueue`. The docstring frames it as: Add a request to the queue according to priority policy. Key calls include `heappush`.
**CN:** 该方法会实现 `add_request`，其作用域位于`PriorityRequestQueue`。 关键调用包括 `heappush`。

### `PriorityRequestQueue.pop_request` method / `PriorityRequestQueue.pop_request` 方法
```python
    def pop_request(self) -> Request:
        """Pop a request from the queue according to priority policy."""
        if not self._heap:
            raise IndexError("pop from empty heap")
        return heapq.heappop(self._heap)
```
**EN:** This method implements `pop_request` within `PriorityRequestQueue`. The docstring frames it as: Pop a request from the queue according to priority policy. Key calls include `heappop`, `IndexError`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `pop_request`，其作用域位于`PriorityRequestQueue`。 关键调用包括 `heappop`, `IndexError`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `PriorityRequestQueue.peek_request` method / `PriorityRequestQueue.peek_request` 方法
```python
    def peek_request(self) -> Request:
        """Peek at the next request in the queue without removing it."""
        if not self._heap:
            raise IndexError("peek from empty heap")
        return self._heap[0]
```
**EN:** This method implements `peek_request` within `PriorityRequestQueue`. The docstring frames it as: Peek at the next request in the queue without removing it. Key calls include `IndexError`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `peek_request`，其作用域位于`PriorityRequestQueue`。 关键调用包括 `IndexError`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `PriorityRequestQueue.prepend_request` method / `PriorityRequestQueue.prepend_request` 方法
```python
    def prepend_request(self, request: Request) -> None:
        """Add a request to the queue according to priority policy.

        Note: In a priority queue, there is no concept of prepending to the
        front. Requests are ordered by (priority, arrival_time)."""
        self.add_request(request)
```
**EN:** This method implements `prepend_request` within `PriorityRequestQueue`. The docstring frames it as: Add a request to the queue according to priority policy. Key calls include `add_request`.
**CN:** 该方法会实现 `prepend_request`，其作用域位于`PriorityRequestQueue`。 关键调用包括 `add_request`。

### `PriorityRequestQueue.prepend_requests` method / `PriorityRequestQueue.prepend_requests` 方法
```python
    def prepend_requests(self, requests: RequestQueue) -> None:
        """Add all requests from another queue according to priority policy.

        Note: In a priority queue, there is no concept of prepending to the
        front. Requests are ordered by (priority, arrival_time)."""
        for request in requests:
            self.add_request(request)
```
**EN:** This method implements `prepend_requests` within `PriorityRequestQueue`. The docstring frames it as: Add all requests from another queue according to priority policy. Key calls include `add_request`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `prepend_requests`，其作用域位于`PriorityRequestQueue`。 关键调用包括 `add_request`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `PriorityRequestQueue.remove_request` method / `PriorityRequestQueue.remove_request` 方法
```python
    def remove_request(self, request: Request) -> None:
        """Remove a specific request from the queue."""
        self._heap.remove(request)
        heapq.heapify(self._heap)
```
**EN:** This method implements `remove_request` within `PriorityRequestQueue`. The docstring frames it as: Remove a specific request from the queue. Key calls include `remove`, `heapify`.
**CN:** 该方法会实现 `remove_request`，其作用域位于`PriorityRequestQueue`。 关键调用包括 `remove`, `heapify`。

### `PriorityRequestQueue.remove_requests` method / `PriorityRequestQueue.remove_requests` 方法
```python
    def remove_requests(self, requests: Iterable[Request]) -> None:
        """Remove multiple specific requests from the queue."""
        requests_to_remove = requests if isinstance(requests, set) else set(requests)
        self._heap = [r for r in self._heap if r not in requests_to_remove]
        heapq.heapify(self._heap)
```
**EN:** This method implements `remove_requests` within `PriorityRequestQueue`. The docstring frames it as: Remove multiple specific requests from the queue. Key calls include `heapify`, `isinstance`, `set`. It touches state such as `_heap`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `remove_requests`，其作用域位于`PriorityRequestQueue`。 关键调用包括 `heapify`, `isinstance`, `set`。 它会读写 `_heap` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `create_request_queue` function / `create_request_queue` 函数
```python
def create_request_queue(policy: SchedulingPolicy) -> RequestQueue:
    """Create request queue based on scheduling policy."""
    if policy == SchedulingPolicy.PRIORITY:
        return PriorityRequestQueue()
    elif policy == SchedulingPolicy.FCFS:
        return FCFSRequestQueue()
    else:
        raise ValueError(f"Unknown scheduling policy: {policy}")
```
**EN:** This function creates a new object or plan within the module. The docstring frames it as: Create request queue based on scheduling policy. Key calls include `PriorityRequestQueue`, `FCFSRequestQueue`, `ValueError`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会创建新的对象或计划，其作用域位于the module。 关键调用包括 `PriorityRequestQueue`, `FCFSRequestQueue`, `ValueError`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `SchedulingPolicy`: central class or interface in this module. / `SchedulingPolicy`：本模块中的核心类或接口。
- `RequestQueue`: central class or interface in this module. / `RequestQueue`：本模块中的核心类或接口。
- `FCFSRequestQueue`: central class or interface in this module. / `FCFSRequestQueue`：本模块中的核心类或接口。
- `PriorityRequestQueue`: central class or interface in this module. / `PriorityRequestQueue`：本模块中的核心类或接口。
- `create_request_queue`: top-level helper or orchestration entry point. / `create_request_queue`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `heapq`, `abc`, `collections`, `enum`
- Internal vLLM / 内部依赖: `vllm.v1.request`
