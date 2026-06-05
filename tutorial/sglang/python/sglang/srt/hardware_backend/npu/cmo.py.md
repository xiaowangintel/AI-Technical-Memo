# cmo.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/cmo.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for backend-specific runtime support inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的后端运行时支持支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Module setup and shared state / 模块设置与共享状态
```python
import torch

cmo_stream = None
share_stream = None
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `torch`. It also defines symbols such as `cmo_stream`, `share_stream` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `torch`。 同时定义了 `cmo_stream`, `share_stream` 等符号，供后续逻辑使用。

### Lines 7-14: Function `get_cmo_stream` / 函数 `get_cmo_stream`
```python
def get_cmo_stream():
    """
    Cache Management Operation(CMO).
    Launch a new stream to prefetch the weight of matmul when running other
    AIV or communication kernels, aiming to overlap the memory access time.
    """
    global cmo_stream
    return cmo_stream
```
**EN:** This function implements `get_cmo_stream`.
**CN:** 该函数实现了 `get_cmo_stream`。

### Lines 17-19: Function `set_cmo_stream` / 函数 `set_cmo_stream`
```python
def set_cmo_stream(stream):
    global cmo_stream
    cmo_stream = stream
```
**EN:** This function implements `set_cmo_stream`. State updates are written into `cmo_stream`.
**CN:** 该函数实现了 `set_cmo_stream`。 状态更新主要写入 `cmo_stream`。

### Lines 22-48: Function `prepare_weight_cache` / 函数 `prepare_weight_cache`
```python
def prepare_weight_cache(handle, cache, PREFETCH_MAX_SIZE=1000000000):
    """
    PREFETCH_MAX_SIZE: maximum size (bytes) for each prefetch operation.
    This affects the time spent in prefetch:
        time ≈ PREFETCH_MAX_SIZE / system_bandwidth
    """
    import torch_npu

    stream = get_cmo_stream()
    if stream is None:
        stream = torch.npu.Stream()
        set_cmo_stream(stream)
    stream.wait_stream(torch.npu.current_stream())
    with torch.npu.stream(stream):
        if isinstance(cache, list):
            for weight in cache:
                torch_npu.npu_prefetch(
                    weight,
                    handle,
                    PREFETCH_MAX_SIZE,
                )
        else:
            torch_npu.npu_prefetch(
                cache,
                handle,
                PREFETCH_MAX_SIZE,
            )
```
**EN:** This function implements `prepare_weight_cache`. It primarily calls `get_cmo_stream`, `stream.wait_stream`, `torch.npu.Stream`, `set_cmo_stream`, `torch.npu.current_stream`, `torch.npu.stream` to complete its work. State updates are written into `stream`. The implementation relies on conditional branches, iteration, context-managed resources.
**CN:** 该函数实现了 `prepare_weight_cache`。 它主要通过调用 `get_cmo_stream`, `stream.wait_stream`, `torch.npu.Stream`, `set_cmo_stream`, `torch.npu.current_stream`, `torch.npu.stream` 来完成任务。 状态更新主要写入 `stream`。 实现中使用了条件分支、迭代逻辑、上下文管理资源。

### Lines 51-55: Function `wait_cmo_stream` / 函数 `wait_cmo_stream`
```python
def wait_cmo_stream():
    stream = get_cmo_stream()
    if stream is not None:
        cur_stream = torch.npu.current_stream()
        cur_stream.wait_stream(stream)
```
**EN:** This function implements `wait_cmo_stream`. It primarily calls `get_cmo_stream`, `torch.npu.current_stream`, `cur_stream.wait_stream` to complete its work. State updates are written into `stream`, `cur_stream`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `wait_cmo_stream`。 它主要通过调用 `get_cmo_stream`, `torch.npu.current_stream`, `cur_stream.wait_stream` 来完成任务。 状态更新主要写入 `stream`, `cur_stream`。 实现中使用了条件分支。

### Lines 58-60: Function `get_share_stream` / 函数 `get_share_stream`
```python
def get_share_stream():
    global share_stream
    return share_stream
```
**EN:** This function implements `get_share_stream`.
**CN:** 该函数实现了 `get_share_stream`。

### Lines 63-65: Function `set_share_stream` / 函数 `set_share_stream`
```python
def set_share_stream(stream):
    global share_stream
    share_stream = stream
```
**EN:** This function implements `set_share_stream`. State updates are written into `share_stream`.
**CN:** 该函数实现了 `set_share_stream`。 状态更新主要写入 `share_stream`。

### Lines 68-72: Function `wait_share_stream` / 函数 `wait_share_stream`
```python
def wait_share_stream():
    stream = get_share_stream()
    if stream is not None:
        cur_stream = torch.npu.current_stream()
        cur_stream.wait_stream(stream)
```
**EN:** This function implements `wait_share_stream`. It primarily calls `get_share_stream`, `torch.npu.current_stream`, `cur_stream.wait_stream` to complete its work. State updates are written into `stream`, `cur_stream`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `wait_share_stream`。 它主要通过调用 `get_share_stream`, `torch.npu.current_stream`, `cur_stream.wait_stream` 来完成任务。 状态更新主要写入 `stream`, `cur_stream`。 实现中使用了条件分支。

### Lines 75-83: Function `shared_expert_on_independent_stream` / 函数 `shared_expert_on_independent_stream`
```python
def shared_expert_on_independent_stream(hidden_states, forward_func):
    stream = get_share_stream()
    if stream is None:
        stream = torch.npu.Stream()
        set_share_stream(stream)
    stream.wait_stream(torch.npu.current_stream())
    with torch.npu.stream(stream):
        shared_output = forward_func(hidden_states)
        return shared_output
```
**EN:** This function implements `shared_expert_on_independent_stream`. It primarily calls `get_share_stream`, `stream.wait_stream`, `torch.npu.Stream`, `set_share_stream`, `torch.npu.current_stream`, `torch.npu.stream` to complete its work. State updates are written into `stream`, `shared_output`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该函数实现了 `shared_expert_on_independent_stream`。 它主要通过调用 `get_share_stream`, `stream.wait_stream`, `torch.npu.Stream`, `set_share_stream`, `torch.npu.current_stream`, `torch.npu.stream` 来完成任务。 状态更新主要写入 `stream`, `shared_output`。 实现中使用了条件分支、上下文管理资源。

## Key Concepts / 关键概念
- **Functions / 函数**: `get_cmo_stream`, `set_cmo_stream`, `prepare_weight_cache`, `wait_cmo_stream`, `get_share_stream`, `set_share_stream`, `wait_share_stream`, `shared_expert_on_independent_stream`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: None / 无
- **External / 外部依赖**: `torch`, `torch_npu`
- **Standard library / 标准库**: None / 无
