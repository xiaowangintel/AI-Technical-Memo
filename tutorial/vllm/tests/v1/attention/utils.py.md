# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/attention/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Utility functions for attention-related v1 tests. / 该文件的文档字符串表明其用途：`utility functions for attention-related v1 tests`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (line 3)
```python
"""Utility functions for attention-related v1 tests."""
```
**EN:** Module docstring that declares the scope of the file: Utility functions for attention-related v1 tests.
**CN:** 模块文档字符串直接说明了文件范围：`utility functions for attention-related v1 tests`。

### Imports and setup / 导入与设置 (lines 5-28)
```python
from dataclasses import dataclass

import pytest
import torch

from vllm.config import (
    CacheConfig,
    CompilationConfig,
    DeviceConfig,
    LoadConfig,
    ModelConfig,
    ParallelConfig,
    SchedulerConfig,
    VllmConfig,
)
from vllm.config.model import ModelDType
from vllm.v1.attention.backend import (
    AttentionImpl,
    AttentionMetadataBuilder,
    AttentionType,
    CommonAttentionMetadata,
)
from vllm.v1.attention.backends.registry import AttentionBackendEnum
from vllm.v1.kv_cache_interface import EncoderOnlyAttentionSpec, FullAttentionSpec
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.config, vllm.config.model, vllm.v1.attention.backend, vllm.v1.attention.backends.registry, vllm.v1.kv_cache_interface`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.config, vllm.config.model, vllm.v1.attention.backend, vllm.v1.attention.backends.registry, vllm.v1.kv_cache_interface`。

### BatchSpec (lines 32-48)
```python
class BatchSpec:
    """Specification for a batch configuration (workload shape only)."""

    seq_lens: list[int]
    query_lens: list[int]

    name: str = "unnamed"

    @property
    def batch_size(self):
        return len(self.seq_lens)

    def __post_init__(self):
        assert len(self.seq_lens) == len(self.query_lens)

    def compute_num_tokens(self):
        return sum(self.query_lens)
```
**EN:** Class `BatchSpec` groups 0 test method(s) and 3 helper/fixture method(s).
**CN:** 类 `BatchSpec` 组织了 0 个测试方法，以及 3 个辅助或 fixture 方法。

### create_common_attn_metadata (lines 51-120)
```python
def create_common_attn_metadata(
    batch_spec: BatchSpec,
    block_size: int,
    device: torch.device,
    max_block_idx: int = 1000,
    arange_block_indices: bool = False,
) -> CommonAttentionMetadata:
    """Create CommonAttentionMetadata from a BatchSpec and ModelParams."""
    # Create query start locations
    query_start_loc = torch.zeros(
        batch_spec.batch_size + 1, dtype=torch.int32, device=device
    )
    query_start_loc[1:] = torch.tensor(
        batch_spec.query_lens, dtype=torch.int32, device=device
    ).cumsum(0)
    query_start_loc_cpu = query_start_loc.cpu()
    num_tokens = batch_spec.compute_num_tokens()

    # ... excerpt omitted for brevity ...
    return CommonAttentionMetadata(
        max_query_len=max_query_len,
        max_seq_len=max_seq_len,
        block_table_tensor=block_table_tensor,
        slot_mapping=slot_mapping,
        causal=True,
```
**EN:** Helper function `create_common_attn_metadata` encapsulates reusable logic for `common attn metadata`. Inputs: `batch_spec, block_size, device, max_block_idx, arange_block_indices`. Key calls include `torch.zeros, tensor.cumsum, query_start_loc.cpu, batch_spec.compute_num_tokens, torch.tensor, seq_lens.cpu`.
**CN:** 辅助函数 `create_common_attn_metadata` 封装了与 `common attn metadata` 相关的可复用逻辑。 输入参数：`batch_spec, block_size, device, max_block_idx, arange_block_indices`。 关键调用包括 `torch.zeros, tensor.cumsum, query_start_loc.cpu, batch_spec.compute_num_tokens, torch.tensor, seq_lens.cpu`。

### try_get_attention_backend (lines 123-132)
```python
def try_get_attention_backend(
    backend: AttentionBackendEnum,
) -> tuple[type[AttentionMetadataBuilder], type[AttentionImpl]]:
    """Try to get the attention backend class, skipping test if not found."""
    try:
        backend_class = backend.get_class()
        return backend_class.get_builder_cls(), backend_class.get_impl_cls()
    except ImportError as e:
        pytest.skip(f"{backend.name} not available: {e}")
        raise AssertionError("unreachable") from None
```
**EN:** Helper function `try_get_attention_backend` encapsulates reusable logic for `try get attention backend`. Inputs: `backend`. Key calls include `backend.get_class, backend_class.get_builder_cls, backend_class.get_impl_cls, pytest.skip, AssertionError`.
**CN:** 辅助函数 `try_get_attention_backend` 封装了与 `try get 注意力 后端` 相关的可复用逻辑。 输入参数：`backend`。 关键调用包括 `backend.get_class, backend_class.get_builder_cls, backend_class.get_impl_cls, pytest.skip, AssertionError`。

### try_backend_includes_kv_cache_update (lines 135-144)
```python
def try_backend_includes_kv_cache_update(
    backend: AttentionBackendEnum,
) -> bool:
    """Try to get the attention backend class, skipping test if not found."""
    try:
        backend_class = backend.get_class()
        return backend_class.forward_includes_kv_cache_update
    except ImportError as e:
        pytest.skip(f"{backend.name} not available: {e}")
        raise AssertionError("unreachable") from None
```
**EN:** Helper function `try_backend_includes_kv_cache_update` encapsulates reusable logic for `try backend includes KV cache update`. Inputs: `backend`. Key calls include `backend.get_class, pytest.skip, AssertionError`.
**CN:** 辅助函数 `try_backend_includes_kv_cache_update` 封装了与 `try 后端 includes KV 缓存 update` 相关的可复用逻辑。 输入参数：`backend`。 关键调用包括 `backend.get_class, pytest.skip, AssertionError`。

### create_standard_kv_cache_spec (lines 147-173)
```python
def create_standard_kv_cache_spec(
    vllm_config: VllmConfig,
    attn_type: AttentionType = AttentionType.DECODER,
) -> FullAttentionSpec | EncoderOnlyAttentionSpec:
    """Create an AttentionSpec from VllmConfig.

    Returns an EncoderOnlyAttentionSpec for encoder-only attention (no KV
    cache), and a FullAttentionSpec otherwise.
    """
    if attn_type == AttentionType.ENCODER_ONLY:
        return EncoderOnlyAttentionSpec(
            block_size=vllm_config.cache_config.block_size,
            num_kv_heads=vllm_config.model_config.get_num_kv_heads(
                vllm_config.parallel_config
            ),
            head_size=vllm_config.model_config.get_head_size(),
            dtype=vllm_config.model_config.dtype,
        )
    return FullAttentionSpec(
        block_size=vllm_config.cache_config.block_size,
        num_kv_heads=vllm_config.model_config.get_num_kv_heads(
            vllm_config.parallel_config
        ),
        head_size=vllm_config.model_config.get_head_size(),
        dtype=vllm_config.model_config.dtype,
        sliding_window=vllm_config.model_config.get_sliding_window(),
    )
```
**EN:** Helper function `create_standard_kv_cache_spec` encapsulates reusable logic for `standard KV cache spec`. Inputs: `vllm_config, attn_type`. Key calls include `FullAttentionSpec, EncoderOnlyAttentionSpec, model_config.get_num_kv_heads, model_config.get_head_size, model_config.get_sliding_window`.
**CN:** 辅助函数 `create_standard_kv_cache_spec` 封装了与 `standard KV 缓存 spec` 相关的可复用逻辑。 输入参数：`vllm_config, attn_type`。 关键调用包括 `FullAttentionSpec, EncoderOnlyAttentionSpec, model_config.get_num_kv_heads, model_config.get_head_size, model_config.get_sliding_window`。

### create_vllm_config (lines 176-254)
```python
def create_vllm_config(
    model_name: str = "meta-llama/Meta-Llama-3-8B",
    tensor_parallel_size: int = 1,
    max_model_len: int = 1024,
    dtype: ModelDType | torch.dtype = "auto",
    num_gpu_blocks: int = 1000,
    block_size: int = 16,
    max_num_seqs: int = 256,
    max_num_batched_tokens: int = 8192,
    enable_chunked_prefill: bool = True,
    add_mock_model_methods: bool = True,
    hf_config_override: dict | None = None,
) -> VllmConfig:
    """Create a VllmConfig for testing with reasonable defaults."""

    model_config = ModelConfig(
        model=model_name,
        tokenizer=model_name,
    # ... excerpt omitted for brevity ...
    return VllmConfig(
        parallel_config=parallel_config,
        scheduler_config=scheduler_config,
        device_config=device_config,
        load_config=load_config,
        compilation_config=compilation_config,
    )
```
**EN:** Helper function `create_vllm_config` encapsulates reusable logic for `vllm config`. Inputs: `model_name, tensor_parallel_size, max_model_len, dtype, num_gpu_blocks, block_size, max_num_seqs, max_num_batched_tokens, ...`. Key calls include `ModelConfig, CacheConfig, ParallelConfig, SchedulerConfig, DeviceConfig, LoadConfig`.
**CN:** 辅助函数 `create_vllm_config` 封装了与 `vllm config` 相关的可复用逻辑。 输入参数：`model_name, tensor_parallel_size, max_model_len, dtype, num_gpu_blocks, block_size, max_num_seqs, max_num_batched_tokens, ...`。 关键调用包括 `ModelConfig, CacheConfig, ParallelConfig, SchedulerConfig, DeviceConfig, LoadConfig`。

### create_dummy_kv_cache (lines 257-275)
```python
def create_dummy_kv_cache(
    block_size: int,
    num_kv_heads: int,
    head_size: int,
    dtype: torch.dtype,
    device: torch.device,
    num_blocks: int = 100,
) -> torch.Tensor:
    """Create a dummy KV cache tensor for testing."""
    kv_cache = torch.randn(
        num_blocks,
        2,  # K and V
        block_size,
        num_kv_heads,
        head_size,
        dtype=dtype,
        device=device,
    )
    return kv_cache
```
**EN:** Helper function `create_dummy_kv_cache` encapsulates reusable logic for `dummy KV cache`. Inputs: `block_size, num_kv_heads, head_size, dtype, device, num_blocks`. Key calls include `torch.randn`.
**CN:** 辅助函数 `create_dummy_kv_cache` 封装了与 `dummy KV 缓存` 相关的可复用逻辑。 输入参数：`block_size, num_kv_heads, head_size, dtype, device, num_blocks`。 关键调用包括 `torch.randn`。

### BackendConfig (lines 279-283)
```python
class BackendConfig:
    name: str
    attention_config: dict
    comp_config: dict
    specific_gpu_arch: tuple | None = None
```
**EN:** Class `BackendConfig` groups 0 test method(s).
**CN:** 类 `BackendConfig` 组织了 0 个测试方法。

### Module state / 模块级状态 (lines 287-378)
```python
full_cg_backend_configs = {
    # FA3 on Hopper
    "FA3": BackendConfig(
        name="FA3",
        attention_config={
            "backend": "FLASH_ATTN",
            "flash_attn_version": 3,
            "flash_attn_max_num_splits_for_cuda_graph": 16,
        },
        comp_config={
            "cudagraph_mode": "FULL",
        specific_gpu_arch=(9, 0),
    ),
    # FlashMLA on Hopper
    "FlashMLA": BackendConfig(
        name="FlashMLA",
        attention_config={"backend": "FLASHMLA"},
    # ... excerpt omitted for brevity ...
}
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `full_cg_backend_configs`. Shared setup calls include `BackendConfig`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`full_cg_backend_configs`。 共享初始化调用包括 `BackendConfig`。

## Key Concepts / 关键概念
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** Speculative execution or decoding paths
- **CN:** 推测执行或解码路径

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm.config, vllm.config.model, vllm.v1.attention.backend, vllm.v1.attention.backends.registry, vllm.v1.kv_cache_interface`.
- **CN:** 被测试的 vLLM 模块：`vllm.config, vllm.config.model, vllm.v1.attention.backend, vllm.v1.attention.backends.registry, vllm.v1.kv_cache_interface`。
- **EN:** Standard-library support: `dataclasses, types`.
- **CN:** 标准库支持：`dataclasses, types`。
