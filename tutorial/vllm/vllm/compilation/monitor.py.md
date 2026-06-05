# monitor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/compilation/monitor.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Runtime monitors and toggles for torch.compile and CUDA-graph capture. / 用于 torch.compile 与 CUDA 图捕获的运行时监控与开关。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-9)
```python
import contextlib
import time
from collections.abc import Generator

from vllm.config import CompilationMode, VllmConfig
from vllm.logger import init_logger
```
**EN:** This import block assembles the PyTorch/FX, Inductor, and vLLM modules that the rest of the file relies on for graph rewriting, backend integration, caching, or runtime control in vLLM's compilation path.
**CN:** 这一组导入语句组织了 PyTorch/FX、Inductor 与 vLLM 模块，供文件后续代码在vLLM 的编译路径中完成图改写、后端集成、缓存或运行时控制。

### Constants / assignments (lines 11-87)
```python
logger = init_logger(__name__)
torch_compile_start_time: float = 0.0
cudagraph_capturing_enabled: bool = True
```
**EN:** These assignments establish shared module state such as loggers, global flags, dtype/op handles, or counters that later logic reuses throughout vLLM's compilation path.
**CN:** 这些赋值语句建立了共享模块状态，例如日志器、全局开关、dtype/算子句柄或计数器，供后续代码在vLLM 的编译路径中复用。

### Function `monitor_torch_compile` (lines 18-59)
```python
def monitor_torch_compile(
    vllm_config: VllmConfig,
    message: str = "torch.compile took %.2f s in total",
    is_encoder: bool = False,
) -> Generator[None, None, None]:
    """Context manager that times torch.compile and manages depyf debugging.

    On normal exit: logs the compile time and exits depyf.
    On exception: cleans up depyf without logging (compilation failed).
    """
    global torch_compile_start_time
    torch_compile_start_time = time.perf_counter()

    compilation_config = vllm_config.compilation_config
    depyf_cm = None
    path = vllm_config.compile_debug_dump_path()
...
```
**EN:** Function `monitor_torch_compile` checks runtime state and emits guardrails or diagnostics. It fits into vLLM's compilation path. Context manager that times torch.compile and manages depyf debugging.
**CN:** 函数 `monitor_torch_compile` 检查运行时状态并给出保护或诊断信息，并嵌入vLLM 的编译路径。 文档字符串说明：Context manager that times torch.compile and manages depyf debugging.

### Function `monitor_profiling_run` (lines 63-84)
```python
def monitor_profiling_run() -> Generator[None, None, None]:
    """Context manager that times the initial profiling run.

    Asserts that no backend compilation occurs during the profiling run
    (all compilation should have completed before this point).
    """
    from vllm.compilation.counter import compilation_counter

    backend_compilations_before = compilation_counter.num_backend_compilations
    start = time.perf_counter()
    yield
    elapsed = time.perf_counter() - start
    assert (
        compilation_counter.num_backend_compilations == backend_compilations_before
    ), (
        "backend compilation occurred during the initial profiling run; "
...
```
**EN:** Function `monitor_profiling_run` checks runtime state and emits guardrails or diagnostics. It fits into vLLM's compilation path. Context manager that times the initial profiling run.
**CN:** 函数 `monitor_profiling_run` 检查运行时状态并给出保护或诊断信息，并嵌入vLLM 的编译路径。 文档字符串说明：Context manager that times the initial profiling run.

### Function `validate_cudagraph_capturing_enabled` (lines 90-99)
```python
def validate_cudagraph_capturing_enabled() -> None:
    # used to monitor whether a cudagraph capturing is legal at runtime.
    # should be called before any cudagraph capturing.
    # if an illegal cudagraph capturing happens, raise an error.
    global cudagraph_capturing_enabled
    if not cudagraph_capturing_enabled:
        raise RuntimeError(
            "CUDA graph capturing detected at an inappropriate "
            "time. This operation is currently disabled."
        )
```
**EN:** Function `validate_cudagraph_capturing_enabled` checks runtime state and emits guardrails or diagnostics. It fits into vLLM's compilation path.
**CN:** 函数 `validate_cudagraph_capturing_enabled` 检查运行时状态并给出保护或诊断信息，并嵌入vLLM 的编译路径。

### Function `set_cudagraph_capturing_enabled` (lines 102-104)
```python
def set_cudagraph_capturing_enabled(enabled: bool) -> None:
    global cudagraph_capturing_enabled
    cudagraph_capturing_enabled = enabled
```
**EN:** Function `set_cudagraph_capturing_enabled` mutates configuration or runtime flags that affect subsequent execution. It fits into vLLM's compilation path.
**CN:** 函数 `set_cudagraph_capturing_enabled` 会修改影响后续执行的配置或运行时标志，并嵌入vLLM 的编译路径。

## Key Concepts / 关键概念
- **Compilation pipeline / 编译流水线**
  - **EN:** The file participates in turning FX graphs into optimized executable code or runtime wrappers.
  - **CN:** 该文件参与将 FX 图转换为优化后的可执行代码或运行时封装。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import CompilationMode, VllmConfig`, `from vllm.logger import init_logger`
- **Stdlib / 标准库**: `import contextlib`, `import time`, `from collections.abc import Generator`
