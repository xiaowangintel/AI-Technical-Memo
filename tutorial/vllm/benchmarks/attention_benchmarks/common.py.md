# common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/attention_benchmarks/common.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements multimodal processing benchmarks, benchmark orchestration, attention benchmark flows, centered around `batch_spec_sort_key`, `MockHfConfig`, `MockKVBProj`, `MockIndexer`. / 实现与多模态处理基准、基准测试编排、注意力基准流程相关的逻辑，核心符号包括 `batch_spec_sort_key`, `MockHfConfig`, `MockKVBProj`, `MockIndexer`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 4-4)
```python
"""Common utilities for attention benchmarking."""
```
**EN:** The module docstring introduces Common utilities for attention benchmarking. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Common utilities for attention benchmarking 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 6-16)
```python
import csv
import json
import math
from dataclasses import asdict, dataclass
from pathlib import Path
from typing import Any

import torch
from batch_spec import get_batch_type, parse_batch_spec
from rich.console import Console
from rich.table import Table
```
**EN:** This block gathers standard-library helpers such as `csv`, `json`, `math`, `dataclasses`, `pathlib`, `typing`; third-party packages such as `torch`, `rich.console`, `rich.table`; project-local modules such as `batch_spec`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `csv`, `json`, `math`, `dataclasses`, `pathlib`, `typing`；第三方依赖，如 `torch`, `rich.console`, `rich.table`；项目内部模块，如 `batch_spec`。这些依赖构成了后续基准测试逻辑的基础。

### Function `batch_spec_sort_key` (lines 19-34)
```python
def batch_spec_sort_key(spec: str) -> tuple[int, int, int]:
    """
    Extract sorting key from batch spec: (batch_size, max_q_len, max_kv_len).

    This ensures results are sorted by batch size first, then query length,
    then sequence length, rather than alphabetically.
    """
    try:
        requests = parse_batch_spec(spec)
        batch_size = len(requests)
        max_q_len = max(r.q_len for r in requests) if requests else 0
        max_kv_len = max(r.kv_len for r in requests) if requests else 0
        return (batch_size, max_q_len, max_kv_len)
    except Exception:
        # Fallback for unparsable specs
        return (0, 0, 0)
```
**EN:** `batch_spec_sort_key` Extract sorting key from batch spec: (batch_size, max_q_len, max_kv_len). It mainly works with `spec` and relies on `parse_batch_spec`, `len`, `max` plus error handling to move data through this part of the benchmark pipeline.
**CN:** `batch_spec_sort_key` 的职责是：Extract sorting key from batch spec: (batch_size, max_q_len, max_kv_len)。 它主要处理 `spec`，并结合 `parse_batch_spec`, `len`, `max` 以及 异常处理 来完成这一段基准测试流程。

### Class `MockHfConfig` (lines 40-58)
```python
class MockHfConfig:
    """Mock HuggingFace config that satisfies vLLM's requirements."""

    def __init__(self, mla_dims: dict, index_topk: int | None = None):
        self.num_attention_heads = mla_dims["num_q_heads"]
        self.num_key_value_heads = mla_dims["num_kv_heads"]
        self.hidden_size = mla_dims["head_dim"] * mla_dims["num_q_heads"]
        self.model_type = "deepseek_v2"
        self.is_encoder_decoder = False
        self.kv_lora_rank = mla_dims["kv_lora_rank"]
        self.qk_nope_head_dim = mla_dims["qk_nope_head_dim"]
        self.qk_rope_head_dim = mla_dims["qk_rope_head_dim"]
        self.v_head_dim = mla_dims["v_head_dim"]
        self.qk_head_dim = mla_dims["qk_nope_head_dim"] + mla_dims["qk_rope_head_dim"]
        if index_topk is not None:
            self.index_topk = index_topk

    def get_text_config(self):
        return self
```
**EN:** Class `MockHfConfig` is the main object-oriented wrapper for this module. Its docstring says: Mock HuggingFace config that satisfies vLLM's requirements. It extends `object` and organizes behavior through `__init__`, `get_text_config`.
**CN:** 类 `MockHfConfig` 是该模块中的主要面向对象封装。文档字符串指出：Mock HuggingFace config that satisfies vLLM's requirements。它继承自 `object`，并通过 `__init__`, `get_text_config` 组织行为。

### Method `MockHfConfig.__init__` (lines 43-55)
```python
    def __init__(self, mla_dims: dict, index_topk: int | None = None):
        self.num_attention_heads = mla_dims["num_q_heads"]
        self.num_key_value_heads = mla_dims["num_kv_heads"]
        self.hidden_size = mla_dims["head_dim"] * mla_dims["num_q_heads"]
        self.model_type = "deepseek_v2"
        self.is_encoder_decoder = False
        self.kv_lora_rank = mla_dims["kv_lora_rank"]
        self.qk_nope_head_dim = mla_dims["qk_nope_head_dim"]
        self.qk_rope_head_dim = mla_dims["qk_rope_head_dim"]
        self.v_head_dim = mla_dims["v_head_dim"]
        self.qk_head_dim = mla_dims["qk_nope_head_dim"] + mla_dims["qk_rope_head_dim"]
        if index_topk is not None:
            self.index_topk = index_topk
```
**EN:** `__init__` implements a helper used by `common.py`. It mainly works with `mla_dims`, `index_topk` and relies on direct statements plus branching to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `common.py` 使用的辅助逻辑。 它主要处理 `mla_dims`, `index_topk`，并结合 direct statements 以及 条件分支 来完成这一段基准测试流程。

### Method `MockHfConfig.get_text_config` (lines 57-58)
```python
    def get_text_config(self):
        return self
```
**EN:** `get_text_config` implements a helper used by `common.py`. It mainly works with no explicit parameters and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_text_config` 负责实现 `common.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Top-level setup (lines 62-65)
```python
try:
    from vllm.model_executor.layers.attention_layer_base import AttentionLayerBase
except ImportError:
    AttentionLayerBase = object  # Fallback
```
**EN:** This top-level block prepares shared state such as module-level state. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 module-level state。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Class `MockKVBProj` (lines 68-101)
```python
class MockKVBProj:
    """Mock KV projection layer for MLA prefill mode.

    Mimics ColumnParallelLinear behavior for kv_b_proj in MLA backends.
    Projects kv_c_normed to [qk_nope_head_dim + v_head_dim] per head.
    """

    def __init__(self, num_heads: int, qk_nope_head_dim: int, v_head_dim: int):
        self.num_heads = num_heads
        self.qk_nope_head_dim = qk_nope_head_dim
        self.v_head_dim = v_head_dim
        self.out_dim = qk_nope_head_dim + v_head_dim
        self.weight = torch.empty(0, dtype=torch.bfloat16)

    def __call__(self, x: torch.Tensor) -> tuple[torch.Tensor]:
        """
        Project kv_c_normed to output space.

        Args:
            x: Input tensor [num_tokens, kv_lora_rank]

        Returns:
            Tuple containing output tensor
                [num_tokens, num_heads, qk_nope_head_dim + v_head_dim]
        """
        num_tokens = x.shape[0]
        result = torch.randn(
            num_tokens,
            self.num_heads,
            self.out_dim,
            device=x.device,
            dtype=x.dtype,
        )
        return (result,)  # Return as tuple to match ColumnParallelLinear API
```
**EN:** Class `MockKVBProj` is the main object-oriented wrapper for this module. Its docstring says: Mock KV projection layer for MLA prefill mode. It extends `object` and organizes behavior through `__init__`, `__call__`.
**CN:** 类 `MockKVBProj` 是该模块中的主要面向对象封装。文档字符串指出：Mock KV projection layer for MLA prefill mode。它继承自 `object`，并通过 `__init__`, `__call__` 组织行为。

### Method `MockKVBProj.__init__` (lines 75-80)
```python
    def __init__(self, num_heads: int, qk_nope_head_dim: int, v_head_dim: int):
        self.num_heads = num_heads
        self.qk_nope_head_dim = qk_nope_head_dim
        self.v_head_dim = v_head_dim
        self.out_dim = qk_nope_head_dim + v_head_dim
        self.weight = torch.empty(0, dtype=torch.bfloat16)
```
**EN:** `__init__` implements a helper used by `common.py`. It mainly works with `num_heads`, `qk_nope_head_dim`, `v_head_dim` and relies on `torch.empty` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `common.py` 使用的辅助逻辑。 它主要处理 `num_heads`, `qk_nope_head_dim`, `v_head_dim`，并结合 `torch.empty` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `MockKVBProj.__call__` (lines 82-101)
```python
    def __call__(self, x: torch.Tensor) -> tuple[torch.Tensor]:
        """
        Project kv_c_normed to output space.

        Args:
            x: Input tensor [num_tokens, kv_lora_rank]

        Returns:
            Tuple containing output tensor
                [num_tokens, num_heads, qk_nope_head_dim + v_head_dim]
        """
        num_tokens = x.shape[0]
        result = torch.randn(
            num_tokens,
            self.num_heads,
            self.out_dim,
            device=x.device,
            dtype=x.dtype,
        )
        return (result,)  # Return as tuple to match ColumnParallelLinear API
```
**EN:** `__call__` Project kv_c_normed to output space. It mainly works with `x` and relies on `torch.randn` plus value production to move data through this part of the benchmark pipeline.
**CN:** `__call__` 的职责是：Project kv_c_normed to output space。 它主要处理 `x`，并结合 `torch.randn` 以及 结果返回 来完成这一段基准测试流程。

### Class `MockIndexer` (lines 104-133)
```python
class MockIndexer:
    """Mock Indexer for sparse MLA backends.

    Provides topk_indices_buffer that sparse MLA backends use to determine
    which KV cache slots to attend to for each token.
    """

    def __init__(
        self,
        max_num_tokens: int,
        topk_tokens: int,
        device: torch.device,
    ):
        self.topk_tokens = topk_tokens
        self.topk_indices_buffer = torch.zeros(
            (max_num_tokens, topk_tokens),
            dtype=torch.int32,
            device=device,
        )

    def fill_random_indices(self, num_tokens: int, max_kv_len: int):
        """Fill topk_indices_buffer with random valid indices for benchmarking."""
        indices = torch.randint(
            0,
            max_kv_len,
            (num_tokens, self.topk_tokens),
            dtype=torch.int32,
            device=self.topk_indices_buffer.device,
        )
        self.topk_indices_buffer[:num_tokens] = indices
```
**EN:** Class `MockIndexer` is the main object-oriented wrapper for this module. Its docstring says: Mock Indexer for sparse MLA backends. It extends `object` and organizes behavior through `__init__`, `fill_random_indices`.
**CN:** 类 `MockIndexer` 是该模块中的主要面向对象封装。文档字符串指出：Mock Indexer for sparse MLA backends。它继承自 `object`，并通过 `__init__`, `fill_random_indices` 组织行为。

### Method `MockIndexer.__init__` (lines 111-122)
```python
    def __init__(
        self,
        max_num_tokens: int,
        topk_tokens: int,
        device: torch.device,
    ):
        self.topk_tokens = topk_tokens
        self.topk_indices_buffer = torch.zeros(
            (max_num_tokens, topk_tokens),
            dtype=torch.int32,
            device=device,
        )
```
**EN:** `__init__` implements a helper used by `common.py`. It mainly works with `max_num_tokens`, `topk_tokens`, `device` and relies on `torch.zeros` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `common.py` 使用的辅助逻辑。 它主要处理 `max_num_tokens`, `topk_tokens`, `device`，并结合 `torch.zeros` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `MockIndexer.fill_random_indices` (lines 124-133)
```python
    def fill_random_indices(self, num_tokens: int, max_kv_len: int):
        """Fill topk_indices_buffer with random valid indices for benchmarking."""
        indices = torch.randint(
            0,
            max_kv_len,
            (num_tokens, self.topk_tokens),
            dtype=torch.int32,
            device=self.topk_indices_buffer.device,
        )
        self.topk_indices_buffer[:num_tokens] = indices
```
**EN:** `fill_random_indices` Fill topk_indices_buffer with random valid indices for benchmarking. It mainly works with `num_tokens`, `max_kv_len` and relies on `torch.randint` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `fill_random_indices` 的职责是：Fill topk_indices_buffer with random valid indices for benchmarking。 它主要处理 `num_tokens`, `max_kv_len`，并结合 `torch.randint` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Class `MockLayer` (lines 136-164)
```python
class MockLayer(AttentionLayerBase):
    """Mock attention layer with scale parameters and impl.

    Inherits from AttentionLayerBase so it passes isinstance checks
    in get_layers_from_vllm_config when FlashInfer prefill is enabled.
    """

    def __init__(self, device: torch.device, impl=None, kv_cache_spec=None):
        # Don't call super().__init__() as AttentionLayerBase doesn't have __init__
        self._k_scale = torch.tensor(1.0, device=device)
        self._v_scale = torch.tensor(1.0, device=device)
        self._q_scale = torch.tensor(1.0, device=device)
        # Scalar floats for kernels that need them
        self._k_scale_float = float(self._k_scale.item())
        self._v_scale_float = float(self._v_scale.item())
        self._q_scale_float = float(self._q_scale.item())
        # AttentionImpl for metadata builders to query
        self.impl = impl
        # KV cache spec for get_kv_cache_spec
        self._kv_cache_spec = kv_cache_spec

    def get_attn_backend(self):
        """Get the attention backend class (required by AttentionLayerBase)."""
        # Return None as this is just a mock layer for benchmarking
        return None

    def get_kv_cache_spec(self):
        """Get the KV cache spec (required by AttentionLayerBase)."""
        return self._kv_cache_spec
```
**EN:** Class `MockLayer` is the main object-oriented wrapper for this module. Its docstring says: Mock attention layer with scale parameters and impl. It extends `AttentionLayerBase` and organizes behavior through `__init__`, `get_attn_backend`, `get_kv_cache_spec`.
**CN:** 类 `MockLayer` 是该模块中的主要面向对象封装。文档字符串指出：Mock attention layer with scale parameters and impl。它继承自 `AttentionLayerBase`，并通过 `__init__`, `get_attn_backend`, `get_kv_cache_spec` 组织行为。

### Method `MockLayer.__init__` (lines 143-155)
```python
    def __init__(self, device: torch.device, impl=None, kv_cache_spec=None):
        # Don't call super().__init__() as AttentionLayerBase doesn't have __init__
        self._k_scale = torch.tensor(1.0, device=device)
        self._v_scale = torch.tensor(1.0, device=device)
        self._q_scale = torch.tensor(1.0, device=device)
        # Scalar floats for kernels that need them
        self._k_scale_float = float(self._k_scale.item())
        self._v_scale_float = float(self._v_scale.item())
        self._q_scale_float = float(self._q_scale.item())
        # AttentionImpl for metadata builders to query
        self.impl = impl
        # KV cache spec for get_kv_cache_spec
        self._kv_cache_spec = kv_cache_spec
```
**EN:** `__init__` implements a helper used by `common.py`. It mainly works with `device`, `impl`, `kv_cache_spec` and relies on `torch.tensor`, `float`, `self._k_scale.item`, `self._v_scale.item`, `self._q_scale.item` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `common.py` 使用的辅助逻辑。 它主要处理 `device`, `impl`, `kv_cache_spec`，并结合 `torch.tensor`, `float`, `self._k_scale.item`, `self._v_scale.item`, `self._q_scale.item` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `MockLayer.get_attn_backend` (lines 157-160)
```python
    def get_attn_backend(self):
        """Get the attention backend class (required by AttentionLayerBase)."""
        # Return None as this is just a mock layer for benchmarking
        return None
```
**EN:** `get_attn_backend` Get the attention backend class (required by AttentionLayerBase). It mainly works with no explicit parameters and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_attn_backend` 的职责是：Get the attention backend class (required by AttentionLayerBase)。 它主要处理 无显式参数，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Method `MockLayer.get_kv_cache_spec` (lines 162-164)
```python
    def get_kv_cache_spec(self):
        """Get the KV cache spec (required by AttentionLayerBase)."""
        return self._kv_cache_spec
```
**EN:** `get_kv_cache_spec` Get the KV cache spec (required by AttentionLayerBase). It mainly works with no explicit parameters and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_kv_cache_spec` 的职责是：Get the KV cache spec (required by AttentionLayerBase)。 它主要处理 无显式参数，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Class `ParameterSweep` (lines 168-180)
```python
class ParameterSweep:
    """Configuration for sweeping a backend parameter."""

    param_name: str  # Name of the backend parameter to sweep
    values: list[Any]  # List of values to test
    include_auto: bool = False  # Also test with param unset (auto mode)
    label_format: str = "{backend}_{param_name}_{value}"  # Result label template

    def get_label(self, backend: str, value: Any) -> str:
        """Generate a label for a specific parameter value."""
        return self.label_format.format(
            backend=backend, param_name=self.param_name, value=value
        )
```
**EN:** Class `ParameterSweep` is the main object-oriented wrapper for this module. Its docstring says: Configuration for sweeping a backend parameter. It extends `object` and organizes behavior through `get_label`.
**CN:** 类 `ParameterSweep` 是该模块中的主要面向对象封装。文档字符串指出：Configuration for sweeping a backend parameter。它继承自 `object`，并通过 `get_label` 组织行为。

### Method `ParameterSweep.get_label` (lines 176-180)
```python
    def get_label(self, backend: str, value: Any) -> str:
        """Generate a label for a specific parameter value."""
        return self.label_format.format(
            backend=backend, param_name=self.param_name, value=value
        )
```
**EN:** `get_label` Generate a label for a specific parameter value. It mainly works with `backend`, `value` and relies on `self.label_format.format` plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_label` 的职责是：Generate a label for a specific parameter value。 它主要处理 `backend`, `value`，并结合 `self.label_format.format` 以及 结果返回 来完成这一段基准测试流程。

### Class `ModelParameterSweep` (lines 184-195)
```python
class ModelParameterSweep:
    """Configuration for sweeping a model configuration parameter."""

    param_name: str  # Name of the model config parameter to sweep (e.g., "num_q_heads")
    values: list[Any]  # List of values to test
    label_format: str = "{backend}_{param_name}_{value}"  # Result label template

    def get_label(self, backend: str, value: Any) -> str:
        """Generate a label for a specific parameter value."""
        return self.label_format.format(
            backend=backend, param_name=self.param_name, value=value
        )
```
**EN:** Class `ModelParameterSweep` is the main object-oriented wrapper for this module. Its docstring says: Configuration for sweeping a model configuration parameter. It extends `object` and organizes behavior through `get_label`.
**CN:** 类 `ModelParameterSweep` 是该模块中的主要面向对象封装。文档字符串指出：Configuration for sweeping a model configuration parameter。它继承自 `object`，并通过 `get_label` 组织行为。

### Method `ModelParameterSweep.get_label` (lines 191-195)
```python
    def get_label(self, backend: str, value: Any) -> str:
        """Generate a label for a specific parameter value."""
        return self.label_format.format(
            backend=backend, param_name=self.param_name, value=value
        )
```
**EN:** `get_label` Generate a label for a specific parameter value. It mainly works with `backend`, `value` and relies on `self.label_format.format` plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_label` 的职责是：Generate a label for a specific parameter value。 它主要处理 `backend`, `value`，并结合 `self.label_format.format` 以及 结果返回 来完成这一段基准测试流程。

### Class `BenchmarkConfig` (lines 199-228)
```python
class BenchmarkConfig:
    """Configuration for a single benchmark run."""

    backend: str
    batch_spec: str
    num_layers: int
    head_dim: int
    num_q_heads: int
    num_kv_heads: int
    block_size: int
    device: str
    dtype: torch.dtype = torch.float16
    repeats: int = 1
    warmup_iters: int = 3
    profile_memory: bool = False
    use_cuda_graphs: bool = False

    # "auto" or "fp8"
    kv_cache_dtype: str = "auto"

    # MLA-specific
    prefill_backend: str | None = None
    kv_lora_rank: int | None = None
    qk_nope_head_dim: int | None = None
    qk_rope_head_dim: int | None = None
    v_head_dim: int | None = None

    # Backend-specific tuning
    num_kv_splits: int | None = None  # CUTLASS MLA
    reorder_batch_threshold: int | None = None  # FlashAttn MLA, FlashMLA
```
**EN:** Class `BenchmarkConfig` is the main object-oriented wrapper for this module. Its docstring says: Configuration for a single benchmark run. It extends `object` and organizes behavior through its methods.
**CN:** 类 `BenchmarkConfig` 是该模块中的主要面向对象封装。文档字符串指出：Configuration for a single benchmark run。它继承自 `object`，并通过 its methods 组织行为。

### Class `BenchmarkResult` (lines 232-262)
```python
class BenchmarkResult:
    """Results from a single benchmark run."""

    config: BenchmarkConfig
    mean_time: float  # seconds
    std_time: float  # seconds
    min_time: float  # seconds
    max_time: float  # seconds
    throughput_tokens_per_sec: float | None = None
    memory_allocated_mb: float | None = None
    memory_reserved_mb: float | None = None
    error: str | None = None

    @property
    def success(self) -> bool:
        """Whether benchmark completed successfully."""
        return self.error is None

    def to_dict(self) -> dict[str, Any]:
        """Convert to dictionary for serialization."""
        return {
            "config": asdict(self.config),
            "mean_time": self.mean_time,
            "std_time": self.std_time,
            "min_time": self.min_time,
            "max_time": self.max_time,
            "throughput_tokens_per_sec": self.throughput_tokens_per_sec,
            "memory_allocated_mb": self.memory_allocated_mb,
            "memory_reserved_mb": self.memory_reserved_mb,
            "error": self.error,
        }
```
**EN:** Class `BenchmarkResult` is the main object-oriented wrapper for this module. Its docstring says: Results from a single benchmark run. It extends `object` and organizes behavior through `success`, `to_dict`.
**CN:** 类 `BenchmarkResult` 是该模块中的主要面向对象封装。文档字符串指出：Results from a single benchmark run。它继承自 `object`，并通过 `success`, `to_dict` 组织行为。

### Method `BenchmarkResult.success` (lines 246-248)
```python
    def success(self) -> bool:
        """Whether benchmark completed successfully."""
        return self.error is None
```
**EN:** `success` Whether benchmark completed successfully. It mainly works with no explicit parameters and relies on direct statements plus value production to move data through this part of the benchmark pipeline.
**CN:** `success` 的职责是：Whether benchmark completed successfully。 它主要处理 无显式参数，并结合 direct statements 以及 结果返回 来完成这一段基准测试流程。

### Method `BenchmarkResult.to_dict` (lines 250-262)
```python
    def to_dict(self) -> dict[str, Any]:
        """Convert to dictionary for serialization."""
        return {
            "config": asdict(self.config),
            "mean_time": self.mean_time,
            "std_time": self.std_time,
            "min_time": self.min_time,
            "max_time": self.max_time,
            "throughput_tokens_per_sec": self.throughput_tokens_per_sec,
            "memory_allocated_mb": self.memory_allocated_mb,
            "memory_reserved_mb": self.memory_reserved_mb,
            "error": self.error,
        }
```
**EN:** `to_dict` Convert to dictionary for serialization. It mainly works with no explicit parameters and relies on `asdict` plus value production to move data through this part of the benchmark pipeline.
**CN:** `to_dict` 的职责是：Convert to dictionary for serialization。 它主要处理 无显式参数，并结合 `asdict` 以及 结果返回 来完成这一段基准测试流程。

### Class `ResultsFormatter` (lines 265-408)
```python
class ResultsFormatter:
    """Format and display benchmark results."""

    def __init__(self, console: Console | None = None):
        self.console = console or Console()

    def print_table(
        self,
        results: list[BenchmarkResult],
        backends: list[str],
        compare_to_fastest: bool = True,
    ):
        """
        Print results as a rich table.

        Args:
            results: List of BenchmarkResult
            backends: List of backend names being compared
            compare_to_fastest: Show percentage comparison to fastest
        """
        # Group by batch spec, preserving first-occurrence order
        by_spec = {}
        specs_order = []
        for r in results:
            spec = r.config.batch_spec
            if spec not in by_spec:
                by_spec[spec] = {}
                specs_order.append(spec)
            by_spec[spec][r.config.backend] = r

        # Sort specs by (batch_size, q_len, kv_len) instead of alphabetically
        specs_order = sorted(by_spec.keys(), key=batch_spec_sort_key)

        # Create shortened backend names for display
    # ... omitted for brevity ...
        path_obj = Path(path)
        path_obj.parent.mkdir(parents=True, exist_ok=True)

        data = [r.to_dict() for r in results]
        with open(path, "w") as f:
            json.dump(data, f, indent=2, default=str)

        self.console.print(f"[green]Saved JSON results to {path}[/]")
```
**EN:** Class `ResultsFormatter` is the main object-oriented wrapper for this module. Its docstring says: Format and display benchmark results. It extends `object` and organizes behavior through `__init__`, `print_table`, `save_csv`, `save_json`.
**CN:** 类 `ResultsFormatter` 是该模块中的主要面向对象封装。文档字符串指出：Format and display benchmark results。它继承自 `object`，并通过 `__init__`, `print_table`, `save_csv`, `save_json` 组织行为。

### Method `ResultsFormatter.__init__` (lines 268-269)
```python
    def __init__(self, console: Console | None = None):
        self.console = console or Console()
```
**EN:** `__init__` implements a helper used by `common.py`. It mainly works with `console` and relies on `Console` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `__init__` 负责实现 `common.py` 使用的辅助逻辑。 它主要处理 `console`，并结合 `Console` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Method `ResultsFormatter.print_table` (lines 271-358)
```python
    def print_table(
        self,
        results: list[BenchmarkResult],
        backends: list[str],
        compare_to_fastest: bool = True,
    ):
        """
        Print results as a rich table.

        Args:
            results: List of BenchmarkResult
            backends: List of backend names being compared
            compare_to_fastest: Show percentage comparison to fastest
        """
        # Group by batch spec, preserving first-occurrence order
        by_spec = {}
        specs_order = []
        for r in results:
            spec = r.config.batch_spec
            if spec not in by_spec:
                by_spec[spec] = {}
                specs_order.append(spec)
            by_spec[spec][r.config.backend] = r

        # Sort specs by (batch_size, q_len, kv_len) instead of alphabetically
        specs_order = sorted(by_spec.keys(), key=batch_spec_sort_key)

        # Create shortened backend names for display
        def shorten_backend_name(name: str) -> str:
            """Shorten long backend names for table display."""
            # Remove common prefixes
            name = name.replace("flashattn_mla", "famla")
            name = name.replace("flashinfer_mla", "fimla")
            name = name.replace("flashmla", "fmla")
        # ... omitted for brevity ...
                else:
                    row.append("-")
                    if multi and compare_to_fastest:
                        row.append("-")

            table.add_row(*row)

        self.console.print(table)
```
**EN:** `print_table` Print results as a rich table. It mainly works with `results`, `backends`, `compare_to_fastest` and relies on `specs_order.append`, `sorted`, `by_spec.keys`, `name.replace`, `Table`, `table.add_column` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `print_table` 的职责是：Print results as a rich table。 它主要处理 `results`, `backends`, `compare_to_fastest`，并结合 `specs_order.append`, `sorted`, `by_spec.keys`, `name.replace`, `Table`, `table.add_column` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Method `ResultsFormatter.save_csv` (lines 360-397)
```python
    def save_csv(self, results: list[BenchmarkResult], path: str):
        """Save results to CSV file."""
        if not results:
            return

        path_obj = Path(path)
        path_obj.parent.mkdir(parents=True, exist_ok=True)

        with open(path, "w", newline="") as f:
            writer = csv.DictWriter(
                f,
                fieldnames=[
                    "backend",
                    "batch_spec",
                    "num_layers",
                    "kv_cache_dtype",
                    "mean_time",
                    "std_time",
                    "throughput",
                    "memory_mb",
                ],
            )
            writer.writeheader()
            for r in results:
                writer.writerow(
                    {
                        "backend": r.config.backend,
                        "batch_spec": r.config.batch_spec,
                        "num_layers": r.config.num_layers,
                        "kv_cache_dtype": r.config.kv_cache_dtype,
                        "mean_time": r.mean_time,
                        "std_time": r.std_time,
                        "throughput": r.throughput_tokens_per_sec or 0,
                        "memory_mb": r.memory_allocated_mb or 0,
                    }
                )

        self.console.print(f"[green]Saved CSV results to {path}[/]")
```
**EN:** `save_csv` Save results to CSV file. It mainly works with `results`, `path` and relies on `Path`, `path_obj.parent.mkdir`, `open`, `csv.DictWriter`, `writer.writeheader`, `writer.writerow` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `save_csv` 的职责是：Save results to CSV file。 它主要处理 `results`, `path`，并结合 `Path`, `path_obj.parent.mkdir`, `open`, `csv.DictWriter`, `writer.writeheader`, `writer.writerow` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Method `ResultsFormatter.save_json` (lines 399-408)
```python
    def save_json(self, results: list[BenchmarkResult], path: str):
        """Save results to JSON file."""
        path_obj = Path(path)
        path_obj.parent.mkdir(parents=True, exist_ok=True)

        data = [r.to_dict() for r in results]
        with open(path, "w") as f:
            json.dump(data, f, indent=2, default=str)

        self.console.print(f"[green]Saved JSON results to {path}[/]")
```
**EN:** `save_json` Save results to JSON file. It mainly works with `results`, `path` and relies on `Path`, `path_obj.parent.mkdir`, `r.to_dict`, `open`, `json.dump`, `self.console.print` plus context management to move data through this part of the benchmark pipeline.
**CN:** `save_json` 的职责是：Save results to JSON file。 它主要处理 `results`, `path`，并结合 `Path`, `path_obj.parent.mkdir`, `r.to_dict`, `open`, `json.dump`, `self.console.print` 以及 上下文管理 来完成这一段基准测试流程。

### Function `setup_mla_dims` (lines 411-456)
```python
def setup_mla_dims(model_name: str = "deepseek-v3") -> dict:
    """
    Get MLA dimensions for known models.

    Args:
        model_name: Model identifier

    Returns:
        Dict with MLA dimension configuration
    """
    configs = {
        "deepseek-v2": {
            "kv_lora_rank": 512,
            "qk_nope_head_dim": 128,
            "qk_rope_head_dim": 64,
            "v_head_dim": 128,
            "num_q_heads": 128,
            "num_kv_heads": 1,
            "head_dim": 576,
        },
        "deepseek-v3": {
            "kv_lora_rank": 512,
            "qk_nope_head_dim": 128,
            "qk_rope_head_dim": 64,
            "v_head_dim": 128,
            "num_q_heads": 128,
            "num_kv_heads": 1,
            "head_dim": 576,
        },
        "deepseek-v2-lite": {
            "kv_lora_rank": 512,
            "qk_nope_head_dim": 128,
            "qk_rope_head_dim": 64,
            "v_head_dim": 128,
            "num_q_heads": 16,
            "num_kv_heads": 1,
            "head_dim": 576,
        },
    }

    if model_name not in configs:
        raise ValueError(
            f"Unknown model '{model_name}'. Known models: {list(configs.keys())}"
        )

    return configs[model_name]
```
**EN:** `setup_mla_dims` Get MLA dimensions for known models. It mainly works with `model_name` and relies on `ValueError`, `list`, `configs.keys` plus branching to move data through this part of the benchmark pipeline.
**CN:** `setup_mla_dims` 的职责是：Get MLA dimensions for known models。 它主要处理 `model_name`，并结合 `ValueError`, `list`, `configs.keys` 以及 条件分支 来完成这一段基准测试流程。

### Function `get_attention_scale` (lines 459-461)
```python
def get_attention_scale(head_dim: int) -> float:
    """Compute attention scale factor (1/sqrt(d))."""
    return 1.0 / math.sqrt(head_dim)
```
**EN:** `get_attention_scale` Compute attention scale factor (1/sqrt(d)). It mainly works with `head_dim` and relies on `math.sqrt` plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_attention_scale` 的职责是：Compute attention scale factor (1/sqrt(d))。 它主要处理 `head_dim`，并结合 `math.sqrt` 以及 结果返回 来完成这一段基准测试流程。

### Function `is_mla_backend` (lines 464-482)
```python
def is_mla_backend(backend: str) -> bool:
    """
    Check if backend is an MLA backend using the AttentionBackendEnum.

    Args:
        backend: Backend name matching AttentionBackendEnum exactly
        (e.g., "FLASHMLA_SPARSE")

    Returns:
        True if the backend is an MLA backend, False otherwise
    """
    from vllm.v1.attention.backends.registry import AttentionBackendEnum

    try:
        backend_enum = AttentionBackendEnum[backend]
        backend_class = backend_enum.get_class()
        return backend_class.is_mla()
    except (KeyError, ValueError, ImportError, AttributeError):
        return False
```
**EN:** `is_mla_backend` Check if backend is an MLA backend using the AttentionBackendEnum. It mainly works with `backend` and relies on `backend_enum.get_class`, `backend_class.is_mla` plus error handling to move data through this part of the benchmark pipeline.
**CN:** `is_mla_backend` 的职责是：Check if backend is an MLA backend using the AttentionBackendEnum。 它主要处理 `backend`，并结合 `backend_enum.get_class`, `backend_class.is_mla` 以及 异常处理 来完成这一段基准测试流程。

## Key Concepts / 关键概念
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Serving path benchmarking: the script interacts with HTTP services, servers, or request proxies.
- **CN:** 服务路径基准：脚本会与 HTTP 服务、服务器或请求代理交互。
- **EN:** Attention specialization: the code studies attention backends, cache layouts, or MLA variants.
- **CN:** 注意力专项：代码研究注意力后端、缓存布局或 MLA 变体。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。
- **EN:** Performance metrics: the script emphasizes latency/throughput measurement rather than model quality.
- **CN:** 性能指标：该脚本更强调延迟/吞吐量测量，而不是模型质量。

## Dependencies / 依赖关系
- **EN:** Standard library: `csv`, `json`, `math`, `dataclasses`, `pathlib`, `typing`.
- **CN:** 标准库依赖：`csv`, `json`, `math`, `dataclasses`, `pathlib`, `typing`。
- **EN:** Third-party packages: `torch`, `rich.console`, `rich.table`.
- **CN:** 第三方依赖：`torch`, `rich.console`, `rich.table`。
- **EN:** Internal modules: `batch_spec`.
- **CN:** 内部模块：`batch_spec`。
