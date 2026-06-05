# mla_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/attention_benchmarks/mla_runner.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements MLA attention experiments, benchmark orchestration, attention benchmark flows, centered around `_add_mock_methods_to_model_config`, `create_minimal_vllm_config`, `_PREFILL_BACKEND_CONFIG`, `get_prefill_backend_config`. / 实现与MLA 注意力实验、基准测试编排、注意力基准流程相关的逻辑，核心符号包括 `_add_mock_methods_to_model_config`, `create_minimal_vllm_config`, `_PREFILL_BACKEND_CONFIG`, `get_prefill_backend_config`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 4-9)
```python
"""
MLA benchmark runner - shared utilities for MLA benchmarks.

This module provides helpers for running MLA backends without
needing full VllmConfig integration.
"""
```
**EN:** The module docstring introduces MLA benchmark runner - shared utilities for MLA benchmarks. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 MLA benchmark runner - shared utilities for MLA benchmarks 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 11-32)
```python
import numpy as np
import torch
from batch_spec import parse_batch_spec
from common import (
    BenchmarkResult,
    MockHfConfig,
    MockIndexer,
    MockKVBProj,
    MockLayer,
    setup_mla_dims,
)

from vllm.config import (
    CacheConfig,
    CompilationConfig,
    ModelConfig,
    ParallelConfig,
    SchedulerConfig,
    VllmConfig,
    set_current_vllm_config,
)
from vllm.v1.attention.backends.mla.prefill.registry import MLAPrefillBackendEnum
```
**EN:** This block gathers third-party packages such as `numpy`, `torch`; project-local modules such as `batch_spec`, `common`, `vllm.config`, `vllm.v1.attention.backends.mla.prefill.registry`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了第三方依赖，如 `numpy`, `torch`；项目内部模块，如 `batch_spec`, `common`, `vllm.config`, `vllm.v1.attention.backends.mla.prefill.registry`。这些依赖构成了后续基准测试逻辑的基础。

### Function `_add_mock_methods_to_model_config` (lines 39-57)
```python
def _add_mock_methods_to_model_config(model_config: ModelConfig) -> None:
    """
    Add mock methods for layer-specific queries to ModelConfig.

    These methods are needed by metadata builders but aren't normally
    present on ModelConfig when used in benchmark contexts.
    """
    import types

    model_config.get_num_layers = types.MethodType(lambda self: 1, model_config)
    model_config.get_sliding_window_for_layer = types.MethodType(
        lambda self, _i: None, model_config
    )
    model_config.get_logits_soft_cap_for_layer = types.MethodType(
        lambda self, _i: None, model_config
    )
    model_config.get_sm_scale_for_layer = types.MethodType(
        lambda self, _i: 1.0 / model_config.get_head_size() ** 0.5, model_config
    )
```
**EN:** `_add_mock_methods_to_model_config` Add mock methods for layer-specific queries to ModelConfig. It mainly works with `model_config` and relies on `types.MethodType`, `model_config.get_head_size` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `_add_mock_methods_to_model_config` 的职责是：Add mock methods for layer-specific queries to ModelConfig。 它主要处理 `model_config`，并结合 `types.MethodType`, `model_config.get_head_size` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `create_minimal_vllm_config` (lines 60-191)
```python
def create_minimal_vllm_config(
    model_name: str = "deepseek-v3",
    block_size: int = 128,
    max_num_seqs: int = 256,
    max_num_batched_tokens: int = 8192,
    mla_dims: dict | None = None,
    index_topk: int | None = None,
    prefill_backend: str | None = None,
    kv_cache_dtype: str = "auto",
) -> VllmConfig:
    """
    Create minimal VllmConfig for MLA benchmarks.

    Args:
        model_name: Model name (deepseek-v2, deepseek-v3, etc.) - used if mla_dims not
                    provided
        block_size: KV cache block size
        max_num_seqs: Maximum number of sequences
        mla_dims: Optional custom MLA dimensions dict. If not provided, uses
                  setup_mla_dims(model_name)
        index_topk: Optional topk value for sparse MLA backends. If provided,
                    the config will include index_topk for sparse attention.
        prefill_backend: Prefill backend name (e.g., "fa3", "fa4", "flashinfer",
                        "trtllm"). Configures the attention config to force
                        the specified prefill backend.

    Returns:
        VllmConfig for benchmarking
    """
    # Get MLA dimensions - use provided or load from model name
    if mla_dims is None:
        mla_dims = setup_mla_dims(model_name)

    # Create mock HF config first (avoids downloading from HuggingFace)
    # ... omitted for brevity ...
            "mla_prefill_backend"
        ]
        if prefill_cfg["flash_attn_version"] is not None:
            vllm_config.attention_config.flash_attn_version = prefill_cfg[
                "flash_attn_version"
            ]

    return vllm_config
```
**EN:** `create_minimal_vllm_config` Create minimal VllmConfig for MLA benchmarks. It mainly works with `model_name`, `block_size`, `max_num_seqs`, `max_num_batched_tokens`, `mla_dims`, ... and relies on `setup_mla_dims`, `MockHfConfig`, `tempfile.mkdtemp`, `os.path.join`, `open`, `json.dump` plus branching, error handling and context management to move data through this part of the benchmark pipeline.
**CN:** `create_minimal_vllm_config` 的职责是：Create minimal VllmConfig for MLA benchmarks。 它主要处理 `model_name`, `block_size`, `max_num_seqs`, `max_num_batched_tokens`, `mla_dims`, ...，并结合 `setup_mla_dims`, `MockHfConfig`, `tempfile.mkdtemp`, `os.path.join`, `open`, `json.dump` 以及 条件分支、异常处理、上下文管理 来完成这一段基准测试流程。

### Top-level setup (lines 201-226)
```python
_PREFILL_BACKEND_CONFIG: dict[str, dict] = {
    "fa2": {
        "flash_attn_version": 2,
        "mla_prefill_backend": MLAPrefillBackendEnum.FLASH_ATTN,
    },
    "fa3": {
        "flash_attn_version": 3,
        "mla_prefill_backend": MLAPrefillBackendEnum.FLASH_ATTN,
    },
    "fa4": {
        "flash_attn_version": 4,
        "mla_prefill_backend": MLAPrefillBackendEnum.FLASH_ATTN,
    },
    "flashinfer": {
        "flash_attn_version": None,
        "mla_prefill_backend": MLAPrefillBackendEnum.FLASHINFER,
    },
    "trtllm": {
        "flash_attn_version": None,
        "mla_prefill_backend": MLAPrefillBackendEnum.TRTLLM_RAGGED,
    },
    "tokenspeed": {
        "flash_attn_version": None,
        "mla_prefill_backend": MLAPrefillBackendEnum.TOKENSPEED_MLA,
    },
}
```
**EN:** This top-level block prepares shared state such as `_PREFILL_BACKEND_CONFIG`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `_PREFILL_BACKEND_CONFIG`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `get_prefill_backend_config` (lines 229-236)
```python
def get_prefill_backend_config(prefill_backend: str) -> dict:
    """Get attention config overrides for a prefill backend."""
    if prefill_backend not in _PREFILL_BACKEND_CONFIG:
        raise ValueError(
            f"Unknown prefill backend: {prefill_backend!r}. "
            f"Available: {list(_PREFILL_BACKEND_CONFIG.keys())}"
        )
    return _PREFILL_BACKEND_CONFIG[prefill_backend]
```
**EN:** `get_prefill_backend_config` Get attention config overrides for a prefill backend. It mainly works with `prefill_backend` and relies on `ValueError`, `list`, `_PREFILL_BACKEND_CONFIG.keys` plus branching to move data through this part of the benchmark pipeline.
**CN:** `get_prefill_backend_config` 的职责是：Get attention config overrides for a prefill backend。 它主要处理 `prefill_backend`，并结合 `ValueError`, `list`, `_PREFILL_BACKEND_CONFIG.keys` 以及 条件分支 来完成这一段基准测试流程。

### Top-level setup (lines 246-253)
```python
_BACKEND_PROPERTIES = {
    "FLASHMLA": {
        "query_format": "concat",  # Single concatenated tensor (vs tuple)
    },
    "FLASHMLA_SPARSE": {
        "query_format": "concat",  # Single concatenated tensor (vs tuple)
    },
}
```
**EN:** This top-level block prepares shared state such as `_BACKEND_PROPERTIES`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `_BACKEND_PROPERTIES`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `_get_backend_config` (lines 256-304)
```python
def _get_backend_config(backend: str) -> dict:
    """
    Get backend configuration from AttentionBackendEnum.

    Uses the registry to get the backend class and extract configuration
    from its methods (get_impl_cls, get_builder_cls, is_sparse, etc.).

    Args:
        backend: Backend name matching AttentionBackendEnum exactly
        (e.g., "FLASHMLA_SPARSE")

    Returns:
        Dict with backend configuration
    """
    from vllm.v1.attention.backend import MultipleOf
    from vllm.v1.attention.backends.registry import AttentionBackendEnum

    try:
        backend_enum = AttentionBackendEnum[backend]
        backend_class = backend_enum.get_class()
    except (KeyError, ValueError) as e:
        valid_backends = [e.name for e in AttentionBackendEnum if e.name != "CUSTOM"]
        raise ValueError(
            f"Unknown backend: {backend}. "
            f"Valid MLA backends: {[b for b in valid_backends if 'MLA' in b]}"
        ) from e

    # Get block size from backend class
    block_sizes = backend_class.get_supported_kernel_block_sizes()
    # Use first supported block size (backends typically support one for MLA)
    block_size = block_sizes[0] if block_sizes else None
    if isinstance(block_size, MultipleOf):
        # No fixed block size; fall back to config value
        block_size = None
    # ... omitted for brevity ...
    return {
        "backend_class": backend_class,
        "impl_class": backend_class.get_impl_cls(),
        "builder_class": backend_class.get_builder_cls(),
        "query_format": props.get("query_format", "tuple"),
        "block_size": block_size,
        "is_sparse": is_sparse,
    }
```
**EN:** `_get_backend_config` Get backend configuration from AttentionBackendEnum. It mainly works with `backend` and relies on `backend_enum.get_class`, `ValueError`, `backend_class.get_supported_kernel_block_sizes`, `isinstance`, `getattr`, `_BACKEND_PROPERTIES.get` plus branching and error handling to move data through this part of the benchmark pipeline.
**CN:** `_get_backend_config` 的职责是：Get backend configuration from AttentionBackendEnum。 它主要处理 `backend`，并结合 `backend_enum.get_class`, `ValueError`, `backend_class.get_supported_kernel_block_sizes`, `isinstance`, `getattr`, `_BACKEND_PROPERTIES.get` 以及 条件分支、异常处理 来完成这一段基准测试流程。

### Function `_build_attention_metadata` (lines 312-405)
```python
def _build_attention_metadata(
    requests: list,
    block_size: int,
    device: torch.device,
    builder_instance,
) -> tuple:
    """
    Build attention metadata from batch requests.

    Args:
        requests: List of BatchRequest objects
        block_size: KV cache block size
        device: Target device
        builder_instance: Metadata builder instance

    Returns:
        Tuple of (metadata, kv_cache_num_blocks)
    """
    q_lens = [r.q_len for r in requests]
    kv_lens = [r.kv_len for r in requests]
    total_q = sum(q_lens)
    max_kv = max(kv_lens)

    # Build query start locations
    q_start_cpu = torch.tensor(
        [0] + [sum(q_lens[: i + 1]) for i in range(len(q_lens))],
        dtype=torch.int32,
    )
    q_start_gpu = q_start_cpu.to(device)

    # Build sequence lengths
    seq_lens_cpu = torch.tensor(kv_lens, dtype=torch.int32)
    seq_lens_gpu = seq_lens_cpu.to(device)

    # ... omitted for brevity ...
    # Use the production build() method
    metadata = builder_instance.build(
        common_prefix_len=0,
        common_attn_metadata=common_attn_metadata,
        fast_build=False,
    )

    return metadata, current_block
```
**EN:** `_build_attention_metadata` Build attention metadata from batch requests. It mainly works with `requests`, `block_size`, `device`, `builder_instance` and relies on `sum`, `max`, `torch.tensor`, `range`, `len`, `q_start_cpu.to` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `_build_attention_metadata` 的职责是：Build attention metadata from batch requests。 它主要处理 `requests`, `block_size`, `device`, `builder_instance`，并结合 `sum`, `max`, `torch.tensor`, `range`, `len`, `q_start_cpu.to` 以及 循环迭代 来完成这一段基准测试流程。

### Function `_create_input_tensors` (lines 408-509)
```python
def _create_input_tensors(
    total_q: int,
    mla_dims: dict,
    query_format: str,
    device: torch.device,
    dtype: torch.dtype,
):
    """
    Create input tensors for both decode and prefill modes.

    MLA requires different tensor formats for decode vs prefill:
    - Decode: Uses kv_lora_rank (512) dimension
    - Prefill: Uses qk_nope_head_dim (128) to stay under FlashAttention's 256 limit

    Args:
        total_q: Total number of query tokens
        mla_dims: MLA dimension configuration
        query_format: Either "tuple" or "concat"
        device: Target device
        dtype: Tensor dtype

    Returns:
        Tuple of (decode_inputs, prefill_inputs)
        - decode_inputs: Query tensor(s) for decode mode
        - prefill_inputs: Dict with 'q', 'k_c_normed', 'k_pe', 'k_scale' for prefill
    """
    if query_format == "tuple":
        # Decode mode format: (q_nope, q_pe) where q_nope has kv_lora_rank dim
        q_nope_decode = torch.randn(
            total_q,
            mla_dims["num_q_heads"],
            mla_dims["kv_lora_rank"],
            device=device,
            dtype=dtype,
    # ... omitted for brevity ...
        "q": prefill_q,
        "k_c_normed": k_c_normed,
        "k_pe": k_pe,
        "k_scale": k_scale,
        "output": output,
    }

    return decode_inputs, prefill_inputs
```
**EN:** `_create_input_tensors` Create input tensors for both decode and prefill modes. It mainly works with `total_q`, `mla_dims`, `query_format`, `device`, `dtype` and relies on `torch.randn`, `torch.cat`, `torch.ones`, `torch.zeros` plus branching to move data through this part of the benchmark pipeline.
**CN:** `_create_input_tensors` 的职责是：Create input tensors for both decode and prefill modes。 它主要处理 `total_q`, `mla_dims`, `query_format`, `device`, `dtype`，并结合 `torch.randn`, `torch.cat`, `torch.ones`, `torch.zeros` 以及 条件分支 来完成这一段基准测试流程。

### Function `_create_backend_impl` (lines 517-640)
```python
def _create_backend_impl(
    backend_cfg: dict,
    mla_dims: dict,
    vllm_config: VllmConfig,
    device: torch.device,
    max_num_tokens: int = 8192,
    index_topk: int | None = None,
    kv_cache_dtype: str = "auto",
):
    """
    Create backend implementation instance.

    Args:
        backend_cfg: Backend configuration dict from _get_backend_config()
        mla_dims: MLA dimension configuration
        vllm_config: VllmConfig instance
        device: Target device
        max_num_tokens: Maximum number of tokens for sparse indexer buffer
        index_topk: Topk value for sparse MLA backends

    Returns:
        Tuple of (impl, layer, builder_instance, indexer)
    """
    # Get classes from backend config (already resolved by _get_backend_config)
    impl_class = backend_cfg["impl_class"]
    builder_class = backend_cfg["builder_class"]

    # Calculate scale
    scale = 1.0 / np.sqrt(mla_dims["qk_nope_head_dim"] + mla_dims["qk_rope_head_dim"])

    # Create mock kv_b_proj layer for prefill mode
    mock_kv_b_proj = MockKVBProj(
        num_heads=mla_dims["num_q_heads"],
        qk_nope_head_dim=mla_dims["qk_nope_head_dim"],
    # ... omitted for brevity ...
        builder_instance = builder_class(
            kv_cache_spec=kv_cache_spec,
            layer_names=["placeholder"],
            vllm_config=vllm_config,
            device=device,
        )

    return impl, layer, builder_instance, indexer
```
**EN:** `_create_backend_impl` Create backend implementation instance. It mainly works with `backend_cfg`, `mla_dims`, `vllm_config`, `device`, `max_num_tokens`, ... and relies on `np.sqrt`, `MockKVBProj`, `backend_cfg.get`, `MockIndexer`, `impl_class`, `hasattr` plus branching to move data through this part of the benchmark pipeline.
**CN:** `_create_backend_impl` 的职责是：Create backend implementation instance。 它主要处理 `backend_cfg`, `mla_dims`, `vllm_config`, `device`, `max_num_tokens`, ...，并结合 `np.sqrt`, `MockKVBProj`, `backend_cfg.get`, `MockIndexer`, `impl_class`, `hasattr` 以及 条件分支 来完成这一段基准测试流程。

### Function `_extract_mla_dims_from_config` (lines 648-689)
```python
def _extract_mla_dims_from_config(config) -> dict | None:
    """
    Extract MLA dimensions from BenchmarkConfig if all required fields are present.

    Args:
        config: BenchmarkConfig instance

    Returns:
        Dict with MLA dimensions if all fields are provided, None otherwise
    """
    # Check if all MLA-specific fields are provided
    if all(
        [
            config.kv_lora_rank is not None,
            config.qk_nope_head_dim is not None,
            config.qk_rope_head_dim is not None,
            config.v_head_dim is not None,
        ]
    ):
        return {
            "kv_lora_rank": config.kv_lora_rank,
            "qk_nope_head_dim": config.qk_nope_head_dim,
            "qk_rope_head_dim": config.qk_rope_head_dim,
            "v_head_dim": config.v_head_dim,
            "num_q_heads": config.num_q_heads,
            "num_kv_heads": config.num_kv_heads,
            "head_dim": config.head_dim,
        }
    # Fallback: if MLA fields not fully specified, try to construct from basic fields
    elif config.head_dim == 576:
        # This looks like a DeepSeek MLA config, use standard dimensions with custom
        # head count
        return {
            "kv_lora_rank": 512,
            "qk_nope_head_dim": 128,
            "qk_rope_head_dim": 64,
            "v_head_dim": 128,
            "num_q_heads": config.num_q_heads,
            "num_kv_heads": config.num_kv_heads,
            "head_dim": config.head_dim,
        }
    return None
```
**EN:** `_extract_mla_dims_from_config` Extract MLA dimensions from BenchmarkConfig if all required fields are present. It mainly works with `config` and relies on `all` plus branching to move data through this part of the benchmark pipeline.
**CN:** `_extract_mla_dims_from_config` 的职责是：Extract MLA dimensions from BenchmarkConfig if all required fields are present。 它主要处理 `config`，并结合 `all` 以及 条件分支 来完成这一段基准测试流程。

### Function `_run_single_benchmark` (lines 697-881)
```python
def _run_single_benchmark(
    config,
    impl,
    layer,
    builder_instance,
    backend_cfg: dict,
    mla_dims: dict,
    device: torch.device,
    indexer=None,
    kv_cache_dtype: str | None = None,
) -> BenchmarkResult:
    """
    Run a single benchmark iteration.

    Args:
        config: BenchmarkConfig instance
        impl: Backend implementation instance
        layer: MockLayer instance
        builder_instance: Metadata builder instance
        backend_cfg: Backend configuration dict
        mla_dims: MLA dimension configuration
        device: Target device
        indexer: Optional MockIndexer for sparse backends

    Returns:
        BenchmarkResult with timing statistics
    """
    # Parse batch spec
    requests = parse_batch_spec(config.batch_spec)
    q_lens = [r.q_len for r in requests]
    kv_lens = [r.kv_len for r in requests]
    total_q = sum(q_lens)
    max_kv_len = max(kv_lens)

    # ... omitted for brevity ...
    return BenchmarkResult(
        config=config,
        mean_time=mean_time,
        std_time=float(np.std(times)),
        min_time=float(np.min(times)),
        max_time=float(np.max(times)),
        throughput_tokens_per_sec=total_q / mean_time if mean_time > 0 else 0,
    )
```
**EN:** `_run_single_benchmark` Run a single benchmark iteration. It mainly works with `config`, `impl`, `layer`, `builder_instance`, `backend_cfg`, ... and relies on `parse_batch_spec`, `sum`, `max`, `_build_attention_metadata`, `getattr`, `torch.zeros` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `_run_single_benchmark` 的职责是：Run a single benchmark iteration。 它主要处理 `config`, `impl`, `layer`, `builder_instance`, `backend_cfg`, ...，并结合 `parse_batch_spec`, `sum`, `max`, `_build_attention_metadata`, `getattr`, `torch.zeros` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Function `_run_mla_benchmark_batched` (lines 884-1040)
```python
def _run_mla_benchmark_batched(
    backend: str,
    configs_with_params: list[tuple],  # [(config, threshold, num_splits), ...]
    index_topk: int = 2048,
    prefill_backend: str | None = None,
) -> list[BenchmarkResult]:
    """
    Unified batched MLA benchmark runner for all backends.

    Works for: flashattn_mla, flashmla, flashinfer_mla, cutlass_mla,
               flashinfer_mla_sparse, flashmla_sparse

    This function reuses backend initialization across multiple benchmarks
    to avoid setup/teardown overhead.

    Args:
        backend: Backend name (decode backend used for impl construction)
        configs_with_params: List of (config, threshold, num_splits) tuples
            - threshold: reorder_batch_threshold (FlashAttn/FlashMLA only)
            - num_splits: num_kv_splits (CUTLASS only)
        index_topk: Topk value for sparse MLA backends (default 2048)
        prefill_backend: Prefill backend name (e.g., "fa3", "fa4").
            When set, forces the specified FlashAttention version for prefill.

    Returns:
        List of BenchmarkResult objects
    """
    if not configs_with_params:
        return []

    backend_cfg = _get_backend_config(backend)
    device = torch.device(configs_with_params[0][0].device)
    torch.accelerator.set_device_index(device)

    # ... omitted for brevity ...
                if original_threshold is not None:
                    builder_instance.reorder_batch_threshold = original_threshold

                # Restore original num_splits
                if original_num_splits is not None:
                    impl._num_kv_splits = original_num_splits

    return results
```
**EN:** `_run_mla_benchmark_batched` Unified batched MLA benchmark runner for all backends. It mainly works with `backend`, `configs_with_params`, `index_topk`, `prefill_backend` and relies on `_get_backend_config`, `torch.device`, `torch.accelerator.set_device_index`, `_extract_mla_dims_from_config`, `setup_mla_dims`, `backend_cfg.get` plus iteration, branching, error handling and context management to move data through this part of the benchmark pipeline.
**CN:** `_run_mla_benchmark_batched` 的职责是：Unified batched MLA benchmark runner for all backends。 它主要处理 `backend`, `configs_with_params`, `index_topk`, `prefill_backend`，并结合 `_get_backend_config`, `torch.device`, `torch.accelerator.set_device_index`, `_extract_mla_dims_from_config`, `setup_mla_dims`, `backend_cfg.get` 以及 循环迭代、条件分支、异常处理、上下文管理 来完成这一段基准测试流程。

### Function `run_mla_benchmark` (lines 1048-1102)
```python
def run_mla_benchmark(
    backend: str,
    config,
    reorder_batch_threshold: int | None = None,
    num_kv_splits: int | None = None,
    index_topk: int = 2048,
    prefill_backend: str | None = None,
) -> BenchmarkResult | list[BenchmarkResult]:
    """
    Unified MLA benchmark runner for all backends.

    Works for: flashattn_mla, flashmla, flashinfer_mla, cutlass_mla,
               flashinfer_mla_sparse, flashmla_sparse

    Always uses batched execution internally for optimal performance.

    Args:
        backend: Backend name (flashattn_mla, flashmla, flashinfer_mla, cutlass_mla,
                 flashinfer_mla_sparse, flashmla_sparse)
        config: BenchmarkConfig or list of (BenchmarkConfig, param) tuples
        reorder_batch_threshold: Threshold override for FlashAttn/FlashMLA
                                 (single config mode only)
        num_kv_splits: Number of KV splits for CUTLASS (single config mode only)
        index_topk: Topk value for sparse MLA backends (default 2048)
        prefill_backend: Prefill backend name (e.g., "fa3", "fa4").
            When set, forces the specified FlashAttention version for prefill.

    Returns:
        BenchmarkResult (single mode) or list of BenchmarkResult (batched mode)
    """
    # Normalize to batched mode: (config, threshold, num_splits)
    if isinstance(config, list):
        # Already in batched format
        if len(config) > 0 and isinstance(config[0], tuple):
    # ... omitted for brevity ...

    # Use unified batched execution
    results = _run_mla_benchmark_batched(
        backend, configs_with_params, index_topk, prefill_backend=prefill_backend
    )

    # Return single result or list based on input
    return results[0] if return_single else results
```
**EN:** `run_mla_benchmark` Unified MLA benchmark runner for all backends. It mainly works with `backend`, `config`, `reorder_batch_threshold`, `num_kv_splits`, `index_topk`, ... and relies on `isinstance`, `len`, `_run_mla_benchmark_batched` plus branching to move data through this part of the benchmark pipeline.
**CN:** `run_mla_benchmark` 的职责是：Unified MLA benchmark runner for all backends。 它主要处理 `backend`, `config`, `reorder_batch_threshold`, `num_kv_splits`, `index_topk`, ...，并结合 `isinstance`, `len`, `_run_mla_benchmark_batched` 以及 条件分支 来完成这一段基准测试流程。

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
- **EN:** Standard library: none.
- **CN:** 标准库依赖：无。
- **EN:** Third-party packages: `numpy`, `torch`.
- **CN:** 第三方依赖：`numpy`, `torch`。
- **EN:** Internal modules: `batch_spec`, `common`, `vllm.config`, `vllm.v1.attention.backends.mla.prefill.registry`.
- **CN:** 内部模块：`batch_spec`, `common`, `vllm.config`, `vllm.v1.attention.backends.mla.prefill.registry`。
