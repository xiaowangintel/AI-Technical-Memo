# dist_utils.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/dist_utils.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for dist utils, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 dist utils 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```python
# mypy: ignore-errors

import re
import sys
import time
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `re`, `sys`, `time`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`re`, `sys`, `time`。

### Lines 6-10
```python
from functools import partial, wraps

import torch.distributed as dist
import torch.distributed.rpc as rpc
from torch.distributed.rpc import _rref_context_get_debug_info
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.distributed`, `torch.distributed.rpc`; external imports: `functools`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.distributed`, `torch.distributed.rpc`；外部导入：`functools`。

### Lines 11-17
```python
from torch.testing._internal.common_utils import FILE_SCHEMA, TEST_WITH_TSAN


if not dist.is_available():
    print("c10d not available, skipping tests", file=sys.stderr)
    sys.exit(0)

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 18-27
```python

INIT_METHOD_TEMPLATE = FILE_SCHEMA + "{file_name}"

def dist_init(
    old_test_method=None,
    setup_rpc: bool = True,
    clean_shutdown: bool = True,
    faulty_messages=None,
    messages_to_delay=None,
):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `dist_init`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`dist_init`。

### Lines 28-33
```python
    """
    We use this decorator for setting up and tearing down state since
    MultiProcessTestCase runs each `test*` method in a separate process and
    each process just runs the `test*` method without actually calling
    'setUp' and 'tearDown' methods of unittest.

```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 34-43
```python
    Note: pass the string representation of MessageTypes that should be used
    with the faulty agent's send function. By default, all retriable messages
    ("RREF_FORK_REQUEST", "RREF_CHILD_ACCEPT", "RREF_USER_DELETE",
    "CLEANUP_AUTOGRAD_CONTEXT_REQ") will use the faulty send (this default is
    set from faulty_rpc_agent_test_fixture.py).
    """
    # If we use dist_init without arguments (ex: @dist_init), old_test_method is
    # appropriately set and we return the wrapper appropriately. On the other
    # hand if dist_init has arguments (ex: @dist_init(clean_shutdown=False)),
    # old_test_method is None and we return a functools.partial which is the real
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 44-53
```python
    # decorator that is used and as a result we recursively call dist_init with
    # old_test_method and the rest of the arguments appropriately set.
    if old_test_method is None:
        return partial(
            dist_init,
            setup_rpc=setup_rpc,
            clean_shutdown=clean_shutdown,
            faulty_messages=faulty_messages,
            messages_to_delay=messages_to_delay,
        )
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 54-59
```python

    @wraps(old_test_method)
    def new_test_method(self, *arg, **kwargs):
        # Setting _ignore_rref_leak to make sure OwnerRRefs are properly deleted
        # in tests.
        import torch.distributed.rpc.api as api
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `new_test_method`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`new_test_method`。

### Lines 60-64
```python

        api._ignore_rref_leak = False
        self.worker_id = self.rank
        self.setup_fault_injection(faulty_messages, messages_to_delay)

```
- EN: This block implements local helper logic for dist utils. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 dist utils 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 65-71
```python
        rpc_backend_options = self.rpc_backend_options
        if setup_rpc:
            if TEST_WITH_TSAN:
                # TSAN runs much slower.
                rpc_backend_options.rpc_timeout = rpc.constants.DEFAULT_RPC_TIMEOUT_SEC * 5
                rpc.constants.DEFAULT_SHUTDOWN_TIMEOUT = 60

```
- EN: This block bridges to backend-specific execution artifacts; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块桥接到特定后端的执行产物；处理条件控制流。关键符号：无明显局部符号。

### Lines 72-79
```python
            rpc.init_rpc(
                name=f"worker{self.rank:d}",
                backend=self.rpc_backend,
                rank=self.rank,
                world_size=self.world_size,
                rpc_backend_options=rpc_backend_options,
            )

```
- EN: This block bridges to backend-specific execution artifacts. Key symbols: no dominant local symbols.
- CN: 该代码块桥接到特定后端的执行产物。关键符号：无明显局部符号。

### Lines 80-84
```python
        return_value = old_test_method(self, *arg, **kwargs)

        if setup_rpc:
            rpc.shutdown(graceful=clean_shutdown)

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 85-89
```python
        return return_value

    return new_test_method


```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 90-94
```python
def noop() -> None:
    pass


def wait_until_node_failure(rank: int, expected_error_regex: str = ".*") -> str:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `noop`, `wait_until_node_failure`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`noop`, `wait_until_node_failure`。

### Lines 95-103
```python
    """
    Loops until an RPC to the given rank fails. This is used to
    indicate that the node has failed in unit tests.
    Args:
    rank (int): Rank of the node expected to fail
    expected_error_regex (optional, str): Regex of exception message expected. Useful to ensure a specific failure
    occurs, not just any.
    """
    while True:
```
- EN: This block checks invariants or expected outcomes; manipulates graph-like program structures; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；操作图状程序结构；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 104-108
```python
        try:
            rpc.rpc_sync(f"worker{rank}", noop, args=())
            time.sleep(0.1)
        except Exception as e:
            if re.search(pattern=expected_error_regex, string=str(e)):
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 109-118
```python
                return str(e)


def wait_until_pending_futures_and_users_flushed(timeout: int = 20) -> None:
    """
    The RRef protocol holds forkIds of rrefs in a map until those forks are
    confirmed by the owner. The message confirming the fork may arrive after
    our tests check whether this map is empty, which leads to failures and
    flaky tests. to_here also does not guarantee that we have finished
    processind the owner's confirmation message for the RRef. This function
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `wait_until_pending_futures_and_users_flushed`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`wait_until_pending_futures_and_users_flushed`。

### Lines 119-124
```python
    loops until the map is empty, which means the messages have been received
    as processed. Call this function before asserting the map returned by
    _get_debug_info is empty.
    """
    start = time.time()
    while True:
```
- EN: This block checks invariants or expected outcomes; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 125-131
```python
        debug_info = _rref_context_get_debug_info()
        num_pending_futures = int(debug_info["num_pending_futures"])
        num_pending_users = int(debug_info["num_pending_users"])
        if num_pending_futures == 0 and num_pending_users == 0:
            break
        time.sleep(0.1)
        if time.time() - start > timeout:
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 132-136
```python
            raise ValueError(
                f"Timed out waiting to flush pending futures and users, "
                f"had {num_pending_futures} pending futures and {num_pending_users} pending users"
            )

```
- EN: This block reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 137-146
```python

def get_num_owners_and_forks() -> tuple[str, str]:
    """
    Retrieves number of OwnerRRefs and forks on this node from
    _rref_context_get_debug_info.
    """
    rref_dbg_info = _rref_context_get_debug_info()
    num_owners = rref_dbg_info["num_owner_rrefs"]
    num_forks = rref_dbg_info["num_forks"]
    return num_owners, num_forks
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_num_owners_and_forks`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_num_owners_and_forks`。

### Lines 147-156
```python


def wait_until_owners_and_forks_on_rank(
    num_owners: int, num_forks: int, rank: int, timeout: int = 20
) -> None:
    """
    Waits until timeout for num_forks and num_owners to exist on the rank. Used
    to ensure proper deletion of RRefs in tests.
    """
    start = time.time()
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `wait_until_owners_and_forks_on_rank`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`wait_until_owners_and_forks_on_rank`。

### Lines 157-163
```python
    while True:
        num_owners_on_rank, num_forks_on_rank = rpc.rpc_sync(
            worker_name(rank), get_num_owners_and_forks, args=(), timeout=5
        )
        num_owners_on_rank = int(num_owners_on_rank)
        num_forks_on_rank = int(num_forks_on_rank)
        if num_owners_on_rank == num_owners and num_forks_on_rank == num_forks:
```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 164-171
```python
            return
        time.sleep(1)
        if time.time() - start > timeout:
            raise ValueError(
                f"Timed out waiting {timeout} sec for {num_owners} owners and {num_forks} forks on rank,"
                f" had {num_owners_on_rank} owners and {num_forks_on_rank} forks"
            )

```
- EN: This block reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 172-181
```python

def initialize_pg(init_method, rank: int, world_size: int) -> None:
    # This is for tests using `dist.barrier`.
    if not dist.is_initialized():
        dist.init_process_group(
            backend="gloo",
            init_method=init_method,
            rank=rank,
            world_size=world_size,
        )
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `initialize_pg`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`initialize_pg`。

### Lines 182-186
```python


def worker_name(rank: int) -> str:
    return f"worker{rank}"

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `worker_name`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`worker_name`。

### Lines 187-193
```python

def get_function_event(function_events, partial_event_name):
    """
    Returns the first event that matches partial_event_name in the provided
    function_events. These function_events should be the output of
    torch.autograd.profiler.function_events().

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_function_event`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_function_event`。

### Lines 194-199
```python
    Args:
    function_events: function_events returned by the profiler.
    event_name (str): partial key that the event was profiled with.
    """
    event = [event for event in function_events if partial_event_name in event.name][0]  # noqa: RUF015
    return event
```
- EN: This block handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Distributed testing — the implementation coordinates multi-process or sharded training checks.
  CN: 分布式测试——该实现会协调多进程或分片训练相关的检查逻辑。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Backend lowering — the code bridges higher-level programs to executable backend artifacts.
  CN: 后端降级——代码把更高层程序桥接到可执行的后端产物。
- EN: Test maintainability — the file helps standardize expectations, fixtures, or generated coverage across PyTorch tests.
  CN: 测试可维护性——该文件有助于在 PyTorch 测试中统一期望、夹具或生成式覆盖。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch.distributed`, `torch.distributed.rpc`, `torch.testing._internal.common_utils`, `torch.distributed.rpc.api`
- External imports / 外部导入: `re`, `sys`, `time`, `functools`
- Representative symbols / 代表性符号: `INIT_METHOD_TEMPLATE`, `dist_init`, `noop`, `wait_until_node_failure`, `wait_until_pending_futures_and_users_flushed`, `get_num_owners_and_forks`, `wait_until_owners_and_forks_on_rank`, `initialize_pg`, `worker_name`, `get_function_event`
