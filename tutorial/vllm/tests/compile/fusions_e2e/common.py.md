# common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/fusions_e2e/common.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Support module for compile tests in compile / fusions_e2e / common; it defines reusable helpers, reference utilities, or backend wrappers shared by multiple test cases. / compile / fusions_e2e / common 对应的编译测试支撑模块；它定义了可复用的辅助函数、参考工具或后端包装器，供多个测试用例共享。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-11)
```python
import itertools
from collections.abc import Callable, Iterable
from typing import Any, NamedTuple

import pytest
import regex as re

from vllm.platforms import current_platform
from vllm.v1.attention.backends.registry import AttentionBackendEnum
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as itertools, collections.abc, typing, pytest; and vLLM components like vllm.platforms, vllm.v1.attention.backends.registry.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 itertools、collections.abc、typing、pytest；vLLM 内部组件，例如 vllm.platforms、vllm.v1.attention.backends.registry。

### Class `Matches` (lines 14-24)
```python
class Matches(NamedTuple):
    # simple pointwise
    aiter_rms_quant_fusion: int = 0
    rms_quant_fusion: int = 0
    act_quant_fusion: int = 0
    norm_rope_fusion: int = 0
    attn_quant_fusion: int = 0
    # distributed
    ar_rms_fusion: int = 0
    sequence_parallel: int = 0
    async_tp: int = 0
```
**EN:** This class defines a container for Matches.
**CN:** 该类定义了 Matches 对应的容器。

### Class `ModelFusionInfo` (lines 27-32)
```python
class ModelFusionInfo(NamedTuple):
    model_name: str
    matches: Callable[[int], Matches]
    """Given number of hidden layers, produces the matches object"""
    model_kwargs: dict[str, Any] = {}
    hf_overrides: Callable[[int], dict] = lambda n: {"num_hidden_layers": n}
```
**EN:** This class defines a container for ModelFusionInfo.
**CN:** 该类定义了 ModelFusionInfo 对应的容器。

### Class `AttentionBackendCase` (lines 35-38)
```python
class AttentionBackendCase(NamedTuple):
    backend: AttentionBackendEnum
    model_kwargs: dict[str, Any] = {}
    """Additional args required for attn+quant fusion"""
```
**EN:** This class defines a container for AttentionBackendCase.
**CN:** 该类定义了 AttentionBackendCase 对应的容器。

### Constants and module state (lines 41-41)
```python
is_blackwell = lambda: current_platform.is_device_capability_family(100)
```
**EN:** This block prepares module-level state that later helpers and tests reuse.
**CN:** 该代码块准备了后续辅助函数和测试会复用的模块级状态。

### Top-level block starting at line 42 (lines 42-42)
```python
"""Are we running on Blackwell, a lot of tests depend on it"""
```
**EN:** This top-level `Expr` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Expr` 代码块执行周边测试所依赖的辅助逻辑。

### Function `custom_ops_combos` (lines 45-49)
```python
def custom_ops_combos(*custom_ops: str) -> Iterable[str]:
    """Generate all combinations of custom ops for parametrization."""
    custom_ops_lists = [[f"-{op}", f"+{op}"] for op in custom_ops]
    for op_list in itertools.product(*custom_ops_lists):
        yield ",".join(op_list)
```
**EN:** This helper function implements the shared logic for custom ops combos. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 custom ops combos 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Top-level block starting at line 53 (lines 53-53)
```python
assert list(custom_ops_combos("silu_and_mul")) == ["-silu_and_mul", "+silu_and_mul"]
```
**EN:** This top-level `Assert` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Assert` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 54 (lines 54-59)
```python
assert list(custom_ops_combos("quant_fp8", "rms_norm")) == [
    "-quant_fp8,-rms_norm",
    "-quant_fp8,+rms_norm",
    "+quant_fp8,-rms_norm",
    "+quant_fp8,+rms_norm",
]
```
**EN:** This top-level `Assert` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Assert` 代码块执行周边测试所依赖的辅助逻辑。

### Function `has_cuda_graph_wrapper_metadata` (lines 62-70)
```python
def has_cuda_graph_wrapper_metadata() -> bool:
    from importlib import import_module

    try:
        module = import_module("torch._inductor.utils")
        module.CUDAGraphWrapperMetadata  # noqa B018
    except AttributeError:
        return False
    return True
```
**EN:** This helper function implements the shared logic for has CUDA graph wrapper metadata. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 has CUDA graph wrapper metadata 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Constants and module state (lines 73-104)
```python
INDUCTOR_GRAPH_PARTITION = [
    pytest.param(
        True,
        marks=pytest.mark.skipif(
            not has_cuda_graph_wrapper_metadata(),
            reason="torch version does not support Inductor partition",
        ),
        id="inductor_partition",
    ),
    pytest.param(False, id="dynamo_partition"),
]

FUSION_LOG_PATTERNS: dict[str, re.Pattern] = {
    "aiter_rms_quant_fusion": re.compile(
        r"RocmAiterRMSNormQuantFusionPass Replaced (\d+) patterns"
    ),
    "rms_quant_fusion": re.compile(r"rms_quant_fusion.py:\d+] Replaced (\d+) patterns"),
    "act_quant_fusion": re.compile(r"act_quant_fusion.py:\d+] Replaced (\d+) patterns"),
    "norm_rope_fusion": re.compile(
        r"qk_norm_rope_fusion.py:\d+] Fused QK Norm\+RoPE on (\d+) sites"
    ),
    "attn_quant_fusion": re.compile(
        r"attn_quant_fusion.py:\d+] Fused quant onto (\d+) attention nodes"
    ),
    "ar_rms_fusion": re.compile(
        r"allreduce_rms_fusion.py:\d+] Replaced (\d+) patterns"
    ),
    "sequence_parallel": re.compile(
        r"sequence_parallelism.py:\d+] Replaced (\d+) patterns"
    ),
    "async_tp": re.compile(r"collective_fusion.py:\d+] Replaced (\d+) patterns"),
}
```
**EN:** This block centralizes shared constants and parameter grids, including INDUCTOR_GRAPH_PARTITION, FUSION_LOG_PATTERNS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 INDUCTOR_GRAPH_PARTITION、FUSION_LOG_PATTERNS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。
- **Shared test utilities / 共享测试工具:** The module factors repeated setup or reference logic out of the individual test files. / 该模块把重复的初始化或参考逻辑从单个测试文件中抽离出来复用。

## Dependencies / 依赖关系
- `itertools`
- `collections.abc -> Callable, Iterable`
- `typing -> Any, NamedTuple`
- `pytest`
- `regex`
- `vllm.platforms -> current_platform`
- `vllm.v1.attention.backends.registry -> AttentionBackendEnum`
