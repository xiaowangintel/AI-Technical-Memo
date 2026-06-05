# runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/attention_benchmarks/runner.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, attention benchmark flows, centered around `_get_backend_config`, `log_warnings_and_errors_only`, `_build_common_attn_metadata`, `_create_vllm_config`. / 实现与基准测试编排、注意力基准流程相关的逻辑，核心符号包括 `_get_backend_config`, `log_warnings_and_errors_only`, `_build_common_attn_metadata`, `_create_vllm_config`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 4-9)
```python
"""
Standard attention benchmark runner - shared utilities for non-MLA benchmarks.

This module provides helpers for running standard attention backends
(FlashAttention, Triton, FlashInfer) with real vLLM integration.
"""
```
**EN:** The module docstring introduces Standard attention benchmark runner - shared utilities for non-MLA benchmarks. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Standard attention benchmark runner - shared utilities for non-MLA benchmarks 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 11-36)
```python
import logging
import types
from contextlib import contextmanager

import numpy as np
import torch
from batch_spec import parse_batch_spec, reorder_for_flashinfer
from common import BenchmarkConfig, BenchmarkResult, MockLayer, get_attention_scale

from vllm.config import (
    CacheConfig,
    CompilationConfig,
    DeviceConfig,
    LoadConfig,
    ModelConfig,
    ParallelConfig,
    SchedulerConfig,
    VllmConfig,
    set_current_vllm_config,
)
from vllm.v1.attention.backends.utils import (
    CommonAttentionMetadata,
    get_kv_cache_layout,
    set_kv_cache_layout,
)
from vllm.v1.kv_cache_interface import FullAttentionSpec
```
**EN:** This block gathers standard-library helpers such as `logging`, `types`, `contextlib`; third-party packages such as `numpy`, `torch`; project-local modules such as `batch_spec`, `common`, `vllm.config`, `vllm.v1.attention.backends.utils`, `vllm.v1.kv_cache_interface`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `logging`, `types`, `contextlib`；第三方依赖，如 `numpy`, `torch`；项目内部模块，如 `batch_spec`, `common`, `vllm.config`, `vllm.v1.attention.backends.utils`, `vllm.v1.kv_cache_interface`。这些依赖构成了后续基准测试逻辑的基础。

### Function `_get_backend_config` (lines 43-65)
```python
def _get_backend_config(backend: str) -> dict:
    """
    Get backend configuration from AttentionBackendEnum.

    Args:
        backend: Backend name matching AttentionBackendEnum exactly
                 (e.g., "FLASH_ATTN", "TRITON_ATTN", "FLASHINFER")

    Returns:
        Dict with backend_class
    """
    from vllm.v1.attention.backends.registry import AttentionBackendEnum

    try:
        backend_enum = AttentionBackendEnum[backend]
        backend_class = backend_enum.get_class()
    except (KeyError, ValueError) as e:
        valid_backends = [b.name for b in AttentionBackendEnum if b.name != "CUSTOM"]
        raise ValueError(
            f"Unknown backend: {backend}. Valid backends: {valid_backends}"
        ) from e

    return {"backend_class": backend_class}
```
**EN:** `_get_backend_config` Get backend configuration from AttentionBackendEnum. It mainly works with `backend` and relies on `backend_enum.get_class`, `ValueError` plus error handling to move data through this part of the benchmark pipeline.
**CN:** `_get_backend_config` 的职责是：Get backend configuration from AttentionBackendEnum。 它主要处理 `backend`，并结合 `backend_enum.get_class`, `ValueError` 以及 异常处理 来完成这一段基准测试流程。

### Function `log_warnings_and_errors_only` (lines 69-77)
```python
def log_warnings_and_errors_only():
    """Temporarily set vLLM logger to WARNING level."""
    logger = logging.getLogger("vllm")
    old_level = logger.level
    logger.setLevel(logging.WARNING)
    try:
        yield
    finally:
        logger.setLevel(old_level)
```
**EN:** `log_warnings_and_errors_only` Temporarily set vLLM logger to WARNING level. It mainly works with no explicit parameters and relies on `logging.getLogger`, `logger.setLevel` plus error handling to move data through this part of the benchmark pipeline.
**CN:** `log_warnings_and_errors_only` 的职责是：Temporarily set vLLM logger to WARNING level。 它主要处理 无显式参数，并结合 `logging.getLogger`, `logger.setLevel` 以及 异常处理 来完成这一段基准测试流程。

### Function `_build_common_attn_metadata` (lines 85-124)
```python
def _build_common_attn_metadata(
    q_lens: list[int],
    kv_lens: list[int],
    block_size: int,
    device: torch.device,
) -> CommonAttentionMetadata:
    """Build CommonAttentionMetadata from query/kv lengths."""
    batch_size = len(q_lens)
    total_tokens = sum(q_lens)

    query_start_loc = torch.zeros(batch_size + 1, dtype=torch.int32, device=device)
    query_start_loc[1:] = torch.tensor(q_lens, dtype=torch.int32, device=device).cumsum(
        0
    )
    query_start_loc_cpu = query_start_loc.cpu()

    seq_lens = torch.tensor(kv_lens, dtype=torch.int32, device=device)
    max_seq_len = int(seq_lens.max().item())

    max_blocks = (max(kv_lens) + block_size - 1) // block_size
    num_blocks = batch_size * max_blocks
    block_table_tensor = torch.arange(
        num_blocks, dtype=torch.int32, device=device
    ).view(batch_size, max_blocks)
    slot_mapping = torch.arange(total_tokens, dtype=torch.int64, device=device)

    max_query_len = max(q_lens)

    return CommonAttentionMetadata(
        query_start_loc=query_start_loc,
        query_start_loc_cpu=query_start_loc_cpu,
        seq_lens=seq_lens,
        num_reqs=batch_size,
        num_actual_tokens=total_tokens,
        max_query_len=max_query_len,
        max_seq_len=max_seq_len,
        block_table_tensor=block_table_tensor,
        slot_mapping=slot_mapping,
        causal=True,
    )
```
**EN:** `_build_common_attn_metadata` Build CommonAttentionMetadata from query/kv lengths. It mainly works with `q_lens`, `kv_lens`, `block_size`, `device` and relies on `len`, `sum`, `torch.zeros`, `torch.tensor.cumsum`, `torch.tensor`, `query_start_loc.cpu` plus value production to move data through this part of the benchmark pipeline.
**CN:** `_build_common_attn_metadata` 的职责是：Build CommonAttentionMetadata from query/kv lengths。 它主要处理 `q_lens`, `kv_lens`, `block_size`, `device`，并结合 `len`, `sum`, `torch.zeros`, `torch.tensor.cumsum`, `torch.tensor`, `query_start_loc.cpu` 以及 结果返回 来完成这一段基准测试流程。

### Function `_create_vllm_config` (lines 127-192)
```python
def _create_vllm_config(
    config: BenchmarkConfig,
    max_num_blocks: int,
) -> VllmConfig:
    """Create a VllmConfig for benchmarking with mock model methods."""
    model_config = ModelConfig(
        model="meta-llama/Meta-Llama-3-8B",
        tokenizer="meta-llama/Meta-Llama-3-8B",
        trust_remote_code=False,
        dtype="auto",  # Use model's native dtype
        seed=0,
        max_model_len=1024,
    )

    cache_config = CacheConfig(
        block_size=config.block_size,
        cache_dtype=config.kv_cache_dtype,
    )
    cache_config.num_gpu_blocks = max_num_blocks
    cache_config.num_cpu_blocks = 0

    parallel_config = ParallelConfig(tensor_parallel_size=1)
    scheduler_config = SchedulerConfig(
        max_num_seqs=256,
        max_num_batched_tokens=8192,
        max_model_len=8192,
        is_encoder_decoder=False,
        enable_chunked_prefill=True,
    )
    device_config = DeviceConfig()
    load_config = LoadConfig()
    compilation_config = CompilationConfig()

    # Add mock methods for benchmark config values
    # ... omitted for brevity ...
        model_config=model_config,
        cache_config=cache_config,
        parallel_config=parallel_config,
        scheduler_config=scheduler_config,
        device_config=device_config,
        load_config=load_config,
        compilation_config=compilation_config,
    )
```
**EN:** `_create_vllm_config` Create a VllmConfig for benchmarking with mock model methods. It mainly works with `config`, `max_num_blocks` and relies on `ModelConfig`, `CacheConfig`, `ParallelConfig`, `SchedulerConfig`, `DeviceConfig`, `LoadConfig` plus value production to move data through this part of the benchmark pipeline.
**CN:** `_create_vllm_config` 的职责是：Create a VllmConfig for benchmarking with mock model methods。 它主要处理 `config`, `max_num_blocks`，并结合 `ModelConfig`, `CacheConfig`, `ParallelConfig`, `SchedulerConfig`, `DeviceConfig`, `LoadConfig` 以及 结果返回 来完成这一段基准测试流程。

### Function `_create_backend_impl` (lines 200-230)
```python
def _create_backend_impl(
    backend_cfg: dict,
    config: BenchmarkConfig,
    device: torch.device,
    dtype: torch.dtype,
):
    """Create backend implementation instance."""
    backend_class = backend_cfg["backend_class"]

    scale = get_attention_scale(config.head_dim)

    impl = backend_class.get_impl_cls()(
        num_heads=config.num_q_heads,
        head_size=config.head_dim,
        scale=scale,
        num_kv_heads=config.num_kv_heads,
        alibi_slopes=None,
        sliding_window=None,
        kv_cache_dtype=config.kv_cache_dtype,
    )

    kv_cache_spec = FullAttentionSpec(
        block_size=config.block_size,
        num_kv_heads=config.num_kv_heads,
        head_size=config.head_dim,
        dtype=dtype,
    )

    layer = MockLayer(device, kv_cache_spec=kv_cache_spec)

    return backend_class, impl, layer
```
**EN:** `_create_backend_impl` Create backend implementation instance. It mainly works with `backend_cfg`, `config`, `device`, `dtype` and relies on `get_attention_scale`, `backend_class.get_impl_cls`, `FullAttentionSpec`, `MockLayer` plus value production to move data through this part of the benchmark pipeline.
**CN:** `_create_backend_impl` 的职责是：Create backend implementation instance。 它主要处理 `backend_cfg`, `config`, `device`, `dtype`，并结合 `get_attention_scale`, `backend_class.get_impl_cls`, `FullAttentionSpec`, `MockLayer` 以及 结果返回 来完成这一段基准测试流程。

### Function `_create_metadata_builder` (lines 233-278)
```python
def _create_metadata_builder(
    backend_class,
    kv_cache_spec: FullAttentionSpec,
    vllm_config: VllmConfig,
    device: torch.device,
    backend_name: str = "",
):
    """Create metadata builder instance."""
    layer_names = ["layer_0"]
    builder_cls = backend_class.get_builder_cls()

    # Flashinfer needs get_per_layer_parameters mocked since we don't have
    # real model layers registered
    if backend_name == "FLASHINFER":
        import unittest.mock

        from vllm.v1.attention.backends.utils import PerLayerParameters

        def mock_get_per_layer_parameters(vllm_config, layer_names, impl_cls):
            head_size = vllm_config.model_config.get_head_size()
            return {
                layer_name: PerLayerParameters(
                    window_left=-1,  # No sliding window
                    logits_soft_cap=0.0,  # No soft cap
                    sm_scale=1.0 / (head_size**0.5),  # Standard scale
                )
                for layer_name in layer_names
            }

        with unittest.mock.patch(
            "vllm.v1.attention.backends.flashinfer.get_per_layer_parameters",
            mock_get_per_layer_parameters,
        ):
            return builder_cls(
                kv_cache_spec=kv_cache_spec,
                layer_names=layer_names,
                vllm_config=vllm_config,
                device=device,
            )

    return builder_cls(
        kv_cache_spec=kv_cache_spec,
        layer_names=layer_names,
        vllm_config=vllm_config,
        device=device,
    )
```
**EN:** `_create_metadata_builder` Create metadata builder instance. It mainly works with `backend_class`, `kv_cache_spec`, `vllm_config`, `device`, `backend_name` and relies on `backend_class.get_builder_cls`, `vllm_config.model_config.get_head_size`, `PerLayerParameters`, `unittest.mock.patch`, `builder_cls` plus branching and context management to move data through this part of the benchmark pipeline.
**CN:** `_create_metadata_builder` 的职责是：Create metadata builder instance。 它主要处理 `backend_class`, `kv_cache_spec`, `vllm_config`, `device`, `backend_name`，并结合 `backend_class.get_builder_cls`, `vllm_config.model_config.get_head_size`, `PerLayerParameters`, `unittest.mock.patch`, `builder_cls` 以及 条件分支、上下文管理 来完成这一段基准测试流程。

### Function `_create_input_tensors` (lines 286-321)
```python
def _create_input_tensors(
    config: BenchmarkConfig,
    total_q: int,
    device: torch.device,
    dtype: torch.dtype,
    quantize_query: bool = False,
) -> tuple:
    """Create Q, K, V input tensors for all layers.

    When quantize_query is True, queries are cast to fp8 to match backends
    that require query/key/value dtype consistency.
    """
    q_dtype = dtype
    if quantize_query:
        from vllm.platforms import current_platform

        q_dtype = current_platform.fp8_dtype()
    q_list = [
        torch.randn(
            total_q, config.num_q_heads, config.head_dim, device=device, dtype=dtype
        ).to(q_dtype)
        for _ in range(config.num_layers)
    ]
    k_list = [
        torch.randn(
            total_q, config.num_kv_heads, config.head_dim, device=device, dtype=dtype
        )
        for _ in range(config.num_layers)
    ]
    v_list = [
        torch.randn(
            total_q, config.num_kv_heads, config.head_dim, device=device, dtype=dtype
        )
        for _ in range(config.num_layers)
    ]
    return q_list, k_list, v_list
```
**EN:** `_create_input_tensors` Create Q, K, V input tensors for all layers. It mainly works with `config`, `total_q`, `device`, `dtype`, `quantize_query` and relies on `current_platform.fp8_dtype`, `torch.randn.to`, `torch.randn`, `range` plus branching to move data through this part of the benchmark pipeline.
**CN:** `_create_input_tensors` 的职责是：Create Q, K, V input tensors for all layers。 它主要处理 `config`, `total_q`, `device`, `dtype`, `quantize_query`，并结合 `current_platform.fp8_dtype`, `torch.randn.to`, `torch.randn`, `range` 以及 条件分支 来完成这一段基准测试流程。

### Function `_create_kv_cache` (lines 324-372)
```python
def _create_kv_cache(
    config: BenchmarkConfig,
    max_num_blocks: int,
    backend_class,
    device: torch.device,
    dtype: torch.dtype,
) -> list:
    """Create KV cache tensors for all layers using the backend's methods.

    Uses the backend's get_kv_cache_shape() and get_kv_cache_stride_order()
    to create the cache with the correct shape and memory layout.
    """
    # Get the logical shape from the backend
    cache_shape = backend_class.get_kv_cache_shape(
        num_blocks=max_num_blocks,
        block_size=config.block_size,
        num_kv_heads=config.num_kv_heads,
        head_size=config.head_dim,
    )

    # Get the stride order for custom memory layout
    try:
        stride_order = backend_class.get_kv_cache_stride_order()
        assert len(stride_order) == len(cache_shape)
    except (AttributeError, NotImplementedError):
        stride_order = tuple(range(len(cache_shape)))

    # Permute shape to physical layout order
    physical_shape = tuple(cache_shape[i] for i in stride_order)

    # Compute inverse permutation to get back to logical view
    inv_order = [stride_order.index(i) for i in range(len(stride_order))]

    # Use fp8 dtype for cache when requested.
    # ... omitted for brevity ...
    for _ in range(config.num_layers):
        # Allocate in physical layout order (contiguous in memory)
        cache = torch.zeros(*physical_shape, device=device, dtype=cache_dtype)
        # Permute to logical view
        cache = cache.permute(*inv_order)
        cache_list.append(cache)

    return cache_list
```
**EN:** `_create_kv_cache` Create KV cache tensors for all layers using the backend's methods. It mainly works with `config`, `max_num_blocks`, `backend_class`, `device`, `dtype` and relies on `backend_class.get_kv_cache_shape`, `backend_class.get_kv_cache_stride_order`, `len`, `tuple`, `range`, `stride_order.index` plus iteration, branching and error handling to move data through this part of the benchmark pipeline.
**CN:** `_create_kv_cache` 的职责是：Create KV cache tensors for all layers using the backend's methods。 它主要处理 `config`, `max_num_blocks`, `backend_class`, `device`, `dtype`，并结合 `backend_class.get_kv_cache_shape`, `backend_class.get_kv_cache_stride_order`, `len`, `tuple`, `range`, `stride_order.index` 以及 循环迭代、条件分支、异常处理 来完成这一段基准测试流程。

### Function `_run_single_benchmark` (lines 380-464)
```python
def _run_single_benchmark(
    config: BenchmarkConfig,
    impl,
    layer,
    q_list: list,
    k_list: list,
    v_list: list,
    cache_list: list,
    attn_metadata,
    device: torch.device,
    dtype: torch.dtype,
) -> tuple:
    """Run single benchmark iteration with warmup and timing loop."""
    total_q = q_list[0].shape[0]
    out = torch.empty(
        total_q, config.num_q_heads, config.head_dim, device=device, dtype=dtype
    )

    # Warmup
    for _ in range(config.warmup_iters):
        for i in range(config.num_layers):
            impl.forward(
                layer,
                q_list[i],
                k_list[i],
                v_list[i],
                cache_list[i],
                attn_metadata,
                output=out,
            )
    torch.accelerator.synchronize()

    # Optionally capture a CUDA graph after warmup.
    # Graph replay eliminates CPU launch overhead so timings reflect pure
    # ... omitted for brevity ...
    mem_stats = {}
    if config.profile_memory:
        mem_stats = {
            "allocated_mb": torch.accelerator.memory_allocated(device) / 1024**2,
            "reserved_mb": torch.accelerator.memory_reserved(device) / 1024**2,
        }

    return times, mem_stats
```
**EN:** `_run_single_benchmark` Run single benchmark iteration with warmup and timing loop. It mainly works with `config`, `impl`, `layer`, `q_list`, `k_list`, ... and relies on `torch.empty`, `range`, `impl.forward`, `torch.accelerator.synchronize`, `torch.cuda.CUDAGraph`, `torch.cuda.graph` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `_run_single_benchmark` 的职责是：Run single benchmark iteration with warmup and timing loop。 它主要处理 `config`, `impl`, `layer`, `q_list`, `k_list`, ...，并结合 `torch.empty`, `range`, `impl.forward`, `torch.accelerator.synchronize`, `torch.cuda.CUDAGraph`, `torch.cuda.graph` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Function `run_attention_benchmark` (lines 472-581)
```python
def run_attention_benchmark(config: BenchmarkConfig) -> BenchmarkResult:
    """
    Run standard attention benchmark with real kernels.

    Supports: FLASH_ATTN, TRITON_ATTN, FLASHINFER

    Args:
        config: Benchmark configuration

    Returns:
        BenchmarkResult with timing and memory statistics
    """
    device = torch.device(config.device)
    torch.accelerator.set_device_index(device)

    backend_cfg = _get_backend_config(config.backend)

    requests = parse_batch_spec(config.batch_spec)

    if config.backend == "FLASHINFER":
        requests = reorder_for_flashinfer(requests)

    q_lens = [r.q_len for r in requests]
    kv_lens = [r.kv_len for r in requests]
    total_q = sum(q_lens)
    max_kv = max(kv_lens)
    batch_size = len(q_lens)

    # Calculate total blocks needed: batch_size * max_blocks_per_request
    max_blocks_per_request = (max_kv + config.block_size - 1) // config.block_size
    max_num_blocks = batch_size * max_blocks_per_request

    # Suppress vLLM logs during setup to reduce spam
    with log_warnings_and_errors_only():
    # ... omitted for brevity ...
        mean_time=mean_time,
        std_time=np.std(times),
        min_time=np.min(times),
        max_time=np.max(times),
        throughput_tokens_per_sec=throughput,
        memory_allocated_mb=mem_stats.get("allocated_mb"),
        memory_reserved_mb=mem_stats.get("reserved_mb"),
    )
```
**EN:** `run_attention_benchmark` Run standard attention benchmark with real kernels. It mainly works with `config` and relies on `torch.device`, `torch.accelerator.set_device_index`, `_get_backend_config`, `parse_batch_spec`, `reorder_for_flashinfer`, `sum` plus branching and context management to move data through this part of the benchmark pipeline.
**CN:** `run_attention_benchmark` 的职责是：Run standard attention benchmark with real kernels。 它主要处理 `config`，并结合 `torch.device`, `torch.accelerator.set_device_index`, `_get_backend_config`, `parse_batch_spec`, `reorder_for_flashinfer`, `sum` 以及 条件分支、上下文管理 来完成这一段基准测试流程。

## Key Concepts / 关键概念
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Serving path benchmarking: the script interacts with HTTP services, servers, or request proxies.
- **CN:** 服务路径基准：脚本会与 HTTP 服务、服务器或请求代理交互。
- **EN:** Dataset handling: benchmark inputs are loaded, normalized, or synthesized before execution.
- **CN:** 数据集处理：基准输入会在执行前被加载、规范化或合成。
- **EN:** Attention specialization: the code studies attention backends, cache layouts, or MLA variants.
- **CN:** 注意力专项：代码研究注意力后端、缓存布局或 MLA 变体。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `logging`, `types`, `contextlib`.
- **CN:** 标准库依赖：`logging`, `types`, `contextlib`。
- **EN:** Third-party packages: `numpy`, `torch`.
- **CN:** 第三方依赖：`numpy`, `torch`。
- **EN:** Internal modules: `batch_spec`, `common`, `vllm.config`, `vllm.v1.attention.backends.utils`, `vllm.v1.kv_cache_interface`.
- **CN:** 内部模块：`batch_spec`, `common`, `vllm.config`, `vllm.v1.attention.backends.utils`, `vllm.v1.kv_cache_interface`。
