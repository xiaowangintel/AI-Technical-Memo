# model_runner_stub.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/mlx/model_runner_stub.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements MLX backend support for model execution orchestration inside the SGLang runtime. / 为 SGLang 运行时提供面向 MLX 后端的模型执行编排支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Module setup and shared state / 模块设置与共享状态
```python
"""Lightweight ModelRunner stub for MLX on Apple Silicon.

Skips PyTorch weight loading.  Creates only the CPU-side bookkeeping
(req_to_token_pool, token_to_kv_pool_allocator) the scheduler needs.
"""

import logging
from typing import Tuple

import torch

from sglang.srt.mem_cache.allocator import TokenToKVPoolAllocator
from sglang.srt.mem_cache.memory_pool import KVCache, ReqToTokenPool
from sglang.srt.model_executor.model_runner import ModelRunner

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `logging`, `typing`, `torch`, `sglang.srt.mem_cache.allocator`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.model_executor.model_runner`. It also defines symbols such as `logger` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `logging`, `typing`, `torch`, `sglang.srt.mem_cache.allocator`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.model_executor.model_runner`。 同时定义了 `logger` 等符号，供后续逻辑使用。

### Lines 19-25: Class `_DummyKVCache` declaration / 类 `_DummyKVCache` 声明
```python
class _DummyKVCache(KVCache):
    """A KV cache that allocates no GPU memory.

    Satisfies the KVCache interface so that TokenToKVPoolAllocator can be
    constructed, but every buffer access raises — the MLX backend manages
    its own KV cache internally.
    """
```
**EN:** This class establishes `_DummyKVCache` as the main container/coordinator for the surrounding logic. It inherits from `KVCache`. Its core interface includes methods such as `__init__`, `get_key_buffer`, `get_value_buffer`, `get_kv_buffer`, `set_kv_buffer`, `get_kv_size_bytes`.
**CN:** 该类将 `_DummyKVCache` 定义为周边逻辑的主要封装体或协调者。 它继承自 `KVCache`。 其核心接口包括 `__init__`, `get_key_buffer`, `get_value_buffer`, `get_kv_buffer`, `set_kv_buffer`, `get_kv_size_bytes` 等方法。

### Lines 27-42: Method `_DummyKVCache.__init__` / 方法 `_DummyKVCache.__init__`
```python
    def __init__(self, size: int, dtype: torch.dtype, device: str):
        # Bypass KVCache.__init__ to avoid custom_mem_pool / memory_saver
        # initialization that may touch CUDA APIs.
        self.size = size
        self.page_size = 1
        self.dtype = dtype
        self.store_dtype = dtype
        self.device = device
        self.layer_num = 0
        self.start_layer = 0
        self.end_layer = 0
        self.mem_usage = 0
        self.cpu_offloading_chunk_size = 8192
        self.layer_transfer_counter = None
        self.enable_custom_mem_pool = False
        self.custom_mem_pool = None
```
**EN:** This method implements `__init__` on `_DummyKVCache`. State updates are written into `self.size`, `self.page_size`, `self.dtype`, `self.store_dtype`, `self.device`, `self.layer_num`.
**CN:** 该方法（属于 `_DummyKVCache`）实现了 `__init__`。 状态更新主要写入 `self.size`, `self.page_size`, `self.dtype`, `self.store_dtype`, `self.device`, `self.layer_num`。

### Lines 44-45: Method `_DummyKVCache.get_key_buffer` / 方法 `_DummyKVCache.get_key_buffer`
```python
    def get_key_buffer(self, layer_id: int) -> torch.Tensor:
        raise RuntimeError("_DummyKVCache has no key buffer (MLX manages KV cache)")
```
**EN:** This method implements `get_key_buffer` on `_DummyKVCache`. It primarily calls `RuntimeError` to complete its work.
**CN:** 该方法（属于 `_DummyKVCache`）实现了 `get_key_buffer`。 它主要通过调用 `RuntimeError` 来完成任务。

### Lines 47-48: Method `_DummyKVCache.get_value_buffer` / 方法 `_DummyKVCache.get_value_buffer`
```python
    def get_value_buffer(self, layer_id: int) -> torch.Tensor:
        raise RuntimeError("_DummyKVCache has no value buffer (MLX manages KV cache)")
```
**EN:** This method implements `get_value_buffer` on `_DummyKVCache`. It primarily calls `RuntimeError` to complete its work.
**CN:** 该方法（属于 `_DummyKVCache`）实现了 `get_value_buffer`。 它主要通过调用 `RuntimeError` 来完成任务。

### Lines 50-51: Method `_DummyKVCache.get_kv_buffer` / 方法 `_DummyKVCache.get_kv_buffer`
```python
    def get_kv_buffer(self, layer_id: int) -> Tuple[torch.Tensor, torch.Tensor]:
        raise RuntimeError("_DummyKVCache has no kv buffer (MLX manages KV cache)")
```
**EN:** This method implements `get_kv_buffer` on `_DummyKVCache`. It primarily calls `RuntimeError` to complete its work.
**CN:** 该方法（属于 `_DummyKVCache`）实现了 `get_kv_buffer`。 它主要通过调用 `RuntimeError` 来完成任务。

### Lines 53-54: Method `_DummyKVCache.set_kv_buffer` / 方法 `_DummyKVCache.set_kv_buffer`
```python
    def set_kv_buffer(self, layer, loc, cache_k, cache_v) -> None:
        raise RuntimeError("_DummyKVCache cannot set kv buffer (MLX manages KV cache)")
```
**EN:** This method implements `set_kv_buffer` on `_DummyKVCache`. It primarily calls `RuntimeError` to complete its work.
**CN:** 该方法（属于 `_DummyKVCache`）实现了 `set_kv_buffer`。 它主要通过调用 `RuntimeError` 来完成任务。

### Lines 56-57: Method `_DummyKVCache.get_kv_size_bytes` / 方法 `_DummyKVCache.get_kv_size_bytes`
```python
    def get_kv_size_bytes(self):
        return 0, 0
```
**EN:** This method implements `get_kv_size_bytes` on `_DummyKVCache`.
**CN:** 该方法（属于 `_DummyKVCache`）实现了 `get_kv_size_bytes`。

### Lines 60-62: Class `_DummyModel` declaration / 类 `_DummyModel` 声明
```python
class _DummyModel:
    """Minimal stand-in so that `inspect.signature(model.forward)` and
    `getattr(model, ...)` calls in ModelRunner.__init__ don't crash."""
```
**EN:** This class establishes `_DummyModel` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `forward`.
**CN:** 该类将 `_DummyModel` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `forward` 等方法。

### Lines 64-66: Method `_DummyModel.forward` / 方法 `_DummyModel.forward`
```python
    @staticmethod
    def forward():
        pass
```
**EN:** This method implements `forward` on `_DummyModel`.
**CN:** 该方法（属于 `_DummyModel`）实现了 `forward`。

### Lines 69-75: Class `MlxModelRunnerStub` declaration / 类 `MlxModelRunnerStub` 声明
```python
class MlxModelRunnerStub(ModelRunner):
    """ModelRunner that skips PyTorch weight loading and KV cache allocation.

    Overrides both load_model() and initialize() so that no PyTorch model
    weights are loaded and no large KV cache tensors are allocated.  Only
    the minimal bookkeeping pools needed by the scheduler are created.
    """
```
**EN:** This class establishes `MlxModelRunnerStub` as the main container/coordinator for the surrounding logic. It inherits from `ModelRunner`. Its core interface includes methods such as `__init__`, `load_model`, `initialize`.
**CN:** 该类将 `MlxModelRunnerStub` 定义为周边逻辑的主要封装体或协调者。 它继承自 `ModelRunner`。 其核心接口包括 `__init__`, `load_model`, `initialize` 等方法。

### Lines 77-79: Method `MlxModelRunnerStub.__init__` / 方法 `MlxModelRunnerStub.__init__`
```python
    def __init__(self, *args, mlx_pool_size: int | None = None, **kwargs):
        self._mlx_pool_size = mlx_pool_size
        super().__init__(*args, **kwargs)
```
**EN:** This method implements `__init__` on `MlxModelRunnerStub`. It primarily calls `super.__init__`, `super` to complete its work. State updates are written into `self._mlx_pool_size`.
**CN:** 该方法（属于 `MlxModelRunnerStub`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `super` 来完成任务。 状态更新主要写入 `self._mlx_pool_size`。

### Lines 81-101: Method `MlxModelRunnerStub.load_model` / 方法 `MlxModelRunnerStub.load_model`
```python
    def load_model(self):
        """Set only the metadata that downstream code needs, without
        loading any PyTorch model weights."""
        logger.info(
            "MLX stub: skipping PyTorch model weight loading "
            "(inference runs through MLX)"
        )

        self.model = _DummyModel()

        self.sliding_window_size = None
        if (
            self.model_config.is_hybrid_swa
            and self.model_config.sliding_window_size is not None
        ):
            self.sliding_window_size = self.model_config.sliding_window_size
        elif self.model_config.attention_chunk_size is not None:
            self.sliding_window_size = self.model_config.attention_chunk_size

        self.dtype = self.model_config.dtype
        self.weight_load_mem_usage = 0
```
**EN:** This method implements `load_model` on `MlxModelRunnerStub`. It primarily calls `logger.info`, `_DummyModel` to complete its work. State updates are written into `self.model`, `self.sliding_window_size`, `self.dtype`, `self.weight_load_mem_usage`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxModelRunnerStub`）实现了 `load_model`。 它主要通过调用 `logger.info`, `_DummyModel` 来完成任务。 状态更新主要写入 `self.model`, `self.sliding_window_size`, `self.dtype`, `self.weight_load_mem_usage`。 实现中使用了条件分支。

### Lines 103-175: Method `MlxModelRunnerStub.initialize` / 方法 `MlxModelRunnerStub.initialize`
```python
    def initialize(self, pre_model_load_memory: float):
        """Lightweight initialize that skips heavy PyTorch setup.

        Creates minimal req_to_token_pool and token_to_kv_pool_allocator
        with a dummy KV cache (zero GPU memory) so the scheduler works.
        """
        from sglang.srt.utils.torch_memory_saver_adapter import TorchMemorySaverAdapter

        self.memory_saver_adapter = TorchMemorySaverAdapter.create(
            enable=self.server_args.enable_memory_saver
        )

        # Load model (sets metadata only)
        self.sampler = None
        self.load_model()

        # Layer metadata
        model_num_layers = max(
            self.model_config.num_hidden_layers,
            self.model_config.num_attention_layers,
        )
        self.start_layer = 0
        self.end_layer = model_num_layers
        self.num_effective_layers = model_num_layers

        # KV cache dtype
        self.kv_cache_dtype = self.dtype

# ... omitted for brevity ...
        logger.info(
            f"MLX stub: initialized minimal pools "
            f"(max_total_num_tokens={self.max_total_num_tokens}, "
            f"max_running_requests={self.max_running_requests}, "
            f"zero GPU KV cache allocation)"
        )
```
**EN:** This method implements `initialize` on `MlxModelRunnerStub`. It primarily calls `TorchMemorySaverAdapter.create`, `self.load_model`, `max`, `min`, `ReqToTokenPool`, `_DummyKVCache` to complete its work. State updates are written into `self.memory_saver_adapter`, `self.sampler`, `model_num_layers`, `self.start_layer`, `self.end_layer`, `self.num_effective_layers`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxModelRunnerStub`）实现了 `initialize`。 它主要通过调用 `TorchMemorySaverAdapter.create`, `self.load_model`, `max`, `min`, `ReqToTokenPool`, `_DummyKVCache` 来完成任务。 状态更新主要写入 `self.memory_saver_adapter`, `self.sampler`, `model_num_layers`, `self.start_layer`, `self.end_layer`, `self.num_effective_layers`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Classes / 类**: `_DummyKVCache`, `_DummyModel`, `MlxModelRunnerStub`
- **Functions / 函数**: `__init__`, `get_key_buffer`, `get_value_buffer`, `get_kv_buffer`, `set_kv_buffer`, `get_kv_size_bytes`, `forward`, `__init__`
- **Themes / 主题**: `runner`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.mem_cache.allocator`, `sglang.srt.mem_cache.memory_pool`, `sglang.srt.model_executor.model_runner`, `sglang.srt.utils.torch_memory_saver_adapter`
- **External / 外部依赖**: `torch`
- **Standard library / 标准库**: `logging`, `typing`
