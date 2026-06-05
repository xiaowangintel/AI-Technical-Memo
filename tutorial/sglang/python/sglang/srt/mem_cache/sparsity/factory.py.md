# factory.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/mem_cache/sparsity/factory.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements the factory logic used by SGLang's memory-cache subsystem. / 该模块实现 SGLang 内存缓存子系统中的工厂相关逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Imports and setup / 导入与初始化
```python
import json
import logging
from typing import Optional

import torch

from sglang.srt.mem_cache.sparsity.algorithms.base_algorithm import BaseSparseAlgorithm
from sglang.srt.mem_cache.sparsity.algorithms.deepseek_nsa import DeepSeekNSAAlgorithm
from sglang.srt.mem_cache.sparsity.algorithms.quest_algorithm import QuestAlgorithm
from sglang.srt.mem_cache.sparsity.backend.backend_adaptor import (
    FlashAttentionAdaptor,
    NSABackendAdaptor,
)
from sglang.srt.mem_cache.sparsity.core.sparse_coordinator import (
    SparseConfig,
    SparseCoordinator,
)
```
**EN:** Imports `json`, `logging`, `typing`, `torch`, `sglang.srt.mem_cache.sparsity.algorithms.base_algorithm`, `sglang.srt.mem_cache.sparsity.algorithms.deepseek_nsa` and other helpers used by the surrounding scope.
**CN:** 导入 `json`, `logging`, `typing`, `torch`, `sglang.srt.mem_cache.sparsity.algorithms.base_algorithm`, `sglang.srt.mem_cache.sparsity.algorithms.deepseek_nsa` 等依赖，为当前作用域提供类型、工具或实现支撑。

### Lines 19-28: Shared state definitions / 共享状态定义
```python
logger = logging.getLogger(__name__)

_global_sparse_coordinator: Optional[SparseCoordinator] = None

_ALGORITHM_REGISTRY = {
    "quest": lambda config, device, **kw: QuestAlgorithm(config, device, **kw),
    "deepseek_nsa": lambda config, device, **kw: DeepSeekNSAAlgorithm(
        config, device, **kw
    ),
}
```
**EN:** Defines module-level variables such as `logger`, `_global_sparse_coordinator`, `_ALGORITHM_REGISTRY`.
**CN:** 定义模块级变量，例如 `logger`, `_global_sparse_coordinator`, `_ALGORITHM_REGISTRY`。

### Lines 31-42: _create_sparse_algorithm implementation / _create_sparse_algorithm 实现
```python
def _create_sparse_algorithm(
    config: SparseConfig,
    device: torch.device,
    **kwargs,
) -> BaseSparseAlgorithm:
    algorithm_name = config.algorithm.lower()
    factory = _ALGORITHM_REGISTRY.get(algorithm_name)

    if factory is None:
        raise ValueError(f"Unknown sparse algorithm: {algorithm_name}")

    return factory(config, device, **kwargs)
```
**EN:** Implements the create sparse algorithm routine for this scope. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 实现当前作用域中的create稀疏算法例程。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 45-58: _create_backend_adaptor implementation / _create_backend_adaptor 实现
```python
def _create_backend_adaptor(
    backend: str,
    device: torch.device,
    sparse_algorithm: BaseSparseAlgorithm,
    req_to_token_pool,
):
    """Create backend adaptor."""
    if isinstance(sparse_algorithm, DeepSeekNSAAlgorithm):
        return NSABackendAdaptor(device, req_to_token_pool)

    if backend in ["fa3", "flashattention"]:
        return FlashAttentionAdaptor(device)

    raise ValueError(f"Unknown attention backend: {backend}")
```
**EN:** Create backend adaptor. Implements the create backend adaptor routine for this scope. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 实现当前作用域中的create后端适配器例程。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 61-101: _parse_sparse_config implementation / _parse_sparse_config 实现
```python
def _parse_sparse_config(server_args) -> SparseConfig:
    """Parse hierarchical sparse config from JSON string.

    Required fields with defaults: top_k (2048), device_buffer_size (2*top_k),
    host_to_device_ratio (2).
    Optional fields (default None): algorithm, backend, min_sparse_prompt_len,
    page_size. All remaining fields go to sparse_extra_config.
    """
    extra_config_str = server_args.hisparse_config
    if extra_config_str is not None:
        try:
            extra_config = json.loads(extra_config_str)
# ... omitted for brevity ...
        page_size=page_size,
        min_sparse_prompt_len=min_sparse_prompt_len,
        sparse_extra_config=extra_config,
    )
```
**EN:** Parse hierarchical sparse config from JSON string. Implements the parse sparse config routine for this scope. It validates error cases explicitly. It returns a computed result to its caller.
**CN:** 实现当前作用域中的parse稀疏config例程。它会显式处理错误场景。它会向调用方返回计算结果。

### Lines 104-106: parse_hisparse_config implementation / parse_hisparse_config 实现
```python
def parse_hisparse_config(server_args) -> SparseConfig:
    """Parse hisparse config from server_args, returning defaults if no config provided."""
    return _parse_sparse_config(server_args)
```
**EN:** Parse hisparse config from server_args, returning defaults if no config provided. Implements the parse hisparse config routine for this scope. It returns a computed result to its caller.
**CN:** 实现当前作用域中的parse hisparse config例程。它会向调用方返回计算结果。

### Lines 109-135: create_sparse_coordinator implementation / create_sparse_coordinator 实现
```python
def create_sparse_coordinator(
    device: torch.device,
    req_to_token_pool,
    token_to_kv_pool,
    start_layer: int,
    end_layer: int,
    server_args,
    **kwargs,
) -> SparseCoordinator:
    config = _parse_sparse_config(server_args)
    algorithm = _create_sparse_algorithm(config, device, **kwargs)
    backend_adaptor = _create_backend_adaptor(
# ... omitted for brevity ...
        device=device,
    )
    register_sparse_coordinator(coordinator)
    return coordinator
```
**EN:** Constructs a new object or resource with the requested configuration. It returns a computed result to its caller.
**CN:** 按给定配置构造新的对象或资源。它会向调用方返回计算结果。

### Lines 138-140: register_sparse_coordinator implementation / register_sparse_coordinator 实现
```python
def register_sparse_coordinator(coordinator: SparseCoordinator) -> None:
    global _global_sparse_coordinator
    _global_sparse_coordinator = coordinator
```
**EN:** Registers metadata so other components can discover this object.
**CN:** 注册元数据，使其他组件能够发现该对象。

### Lines 143-144: get_sparse_coordinator implementation / get_sparse_coordinator 实现
```python
def get_sparse_coordinator() -> Optional[SparseCoordinator]:
    return _global_sparse_coordinator
```
**EN:** Retrieves the requested data or state from the current object. It returns a computed result to its caller.
**CN:** 获取当前对象中的目标数据或状态。它会向调用方返回计算结果。

## Key Concepts / 关键概念
- **`_create_sparse_algorithm`**: Provides the `_create_sparse_algorithm` entry point for module-level behavior. / 提供模块级行为的 `_create_sparse_algorithm` 入口。
- **`_create_backend_adaptor`**: Provides the `_create_backend_adaptor` entry point for module-level behavior. / 提供模块级行为的 `_create_backend_adaptor` 入口。
- **`_parse_sparse_config`**: Provides the `_parse_sparse_config` entry point for module-level behavior. / 提供模块级行为的 `_parse_sparse_config` 入口。
- **`parse_hisparse_config`**: Provides the `parse_hisparse_config` entry point for module-level behavior. / 提供模块级行为的 `parse_hisparse_config` 入口。
- **`create_sparse_coordinator`**: Provides the `create_sparse_coordinator` entry point for module-level behavior. / 提供模块级行为的 `create_sparse_coordinator` 入口。
- **`register_sparse_coordinator`**: Provides the `register_sparse_coordinator` entry point for module-level behavior. / 提供模块级行为的 `register_sparse_coordinator` 入口。
- **`get_sparse_coordinator`**: Provides the `get_sparse_coordinator` entry point for module-level behavior. / 提供模块级行为的 `get_sparse_coordinator` 入口。

## Dependencies / 依赖关系
- **External / 外部**: `json`, `logging`, `typing`, `torch`
- **Internal / 内部**: `sglang.srt.mem_cache.sparsity.algorithms.base_algorithm`, `sglang.srt.mem_cache.sparsity.algorithms.deepseek_nsa`, `sglang.srt.mem_cache.sparsity.algorithms.quest_algorithm`, `sglang.srt.mem_cache.sparsity.backend.backend_adaptor`, `sglang.srt.mem_cache.sparsity.core.sparse_coordinator`
