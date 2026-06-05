# _aiter_ops.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/_aiter_ops.py`
- **Repository**: vllm-project/vllm
- **Purpose**: EN: Bridge ROCm AITER kernels into vLLM with feature gating, fake implementations, and high-level wrapper methods. / CN: 将 ROCm AITER 内核接入 vLLM，并提供特性开关、fake 实现以及高层包装接口。

## Line-by-Line Analysis / 逐行分析

### Capability discovery and cached globals
```python
try:
    import pandas as pd
except ImportError:
    pd = PlaceholderModule("pandas")

# fp8_dtype is not cached.
# on ROCm the fp8_dtype always calls is_fp8_fnuz
# which is a host op, so we cache it once here.
FP8_DTYPE = current_platform.fp8_dtype()


def is_aiter_found() -> bool:
    from importlib.util import find_spec

    return find_spec("aiter") is not None


# `find_spec` is not torch.compile compatible.
# In cases where aiter availability might have
# been checked in forward passes that are torch compiled.
# we keep this global outside to not cause torch compile breaks.
IS_AITER_FOUND = is_aiter_found()
```
**EN:** This bootstrap path keeps optional AITER support compile-safe. `find_spec("aiter")` is executed once and cached in `IS_AITER_FOUND`, while `FP8_DTYPE` is also cached because querying the ROCm FP8 dtype can itself trigger host-side logic. The `pandas` import is wrapped in `PlaceholderModule`, so autotune CSV support degrades gracefully instead of making the whole module unimportable.
**CN:** 这段启动逻辑把可选的 AITER 支持做成了 `torch.compile` 友好的形式。`find_spec("aiter")` 只在导入时执行一次并缓存到 `IS_AITER_FOUND`；`FP8_DTYPE` 也提前缓存，因为在 ROCm 上查询 FP8 dtype 本身可能触发主机侧逻辑。`pandas` 则通过 `PlaceholderModule` 包装，因此调优 CSV 缺失时只会降级，不会让整个模块无法导入。

### Support gating and tuned GEMM lookup
```python
def is_aiter_found_and_supported() -> bool:
    """Check if AITER library is available and platform supports it.

    Checks: platform (ROCm), device arch (gfx9), and library existence.
    Does NOT check environment variables - that's handled by rocm_aiter_ops.is_enabled().

    This function determines if aiter CAN be used, not if it SHOULD be used.

    Separation of concerns:
    - This function: Can aiter work on this system? (platform + library availability)
    - rocm_aiter_ops.is_enabled(): Should aiter be used by default? (adds env var check)
    - Backend selection: Can explicitly request aiter regardless of env var

    This allows explicit backend selection via attention_config to work even when
    VLLM_ROCM_USE_AITER=0, while preventing unwanted JIT warnings for auto-discovery.
    """
    if current_platform.is_rocm() and IS_AITER_FOUND:
        from vllm.platforms.rocm import on_mi3xx

        return on_mi3xx()
    return False


@functools.cache
def _load_gemm_tuned_configs(
    q_dtype_w: torch.dtype, csv_path: str
) -> set[tuple[int, int, int]]:
    try:
        df = pd.read_csv(csv_path).drop_duplicates()
        df = df[df["q_dtype_w"] == str(q_dtype_w)]
        return set(zip(df["N"].astype(int), df["K"].astype(int), df["M"].astype(int)))
    except Exception:
        return set()


def _check_kernel_tuned(N: int, K: int, q_dtype_w: torch.dtype, csv_path: str) -> bool:
    configs = _load_gemm_tuned_configs(q_dtype_w, csv_path)
    l_m = (
        [1, 2, 4]
        + list(range(8, 513, 8))
        + [1024, 1536]
        + [2**i for i in range(11, 19)]
    )
    return any((N, K, M) in configs for M in l_m)


def if_aiter_supported(func: Callable) -> Callable:
    """Decorator that only executes the function if
    ROCm AITER package is supported and enabled on gfx9 archs.
    """

    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        if is_aiter_found_and_supported():
            return func(*args, **kwargs)

        return None

    return wrapper
```
**EN:** `is_aiter_found_and_supported()` answers whether AITER can run on this machine (ROCm + MI3xx + library present) and deliberately does not look at env toggles; those belong to later policy checks. `_load_gemm_tuned_configs()` / `_check_kernel_tuned()` read AITER autotune CSVs, filter by weight-quant dtype, and answer whether a given `(N, K)` has any tuned `M` candidate. The decorator short-circuits unsupported systems by returning `None` instead of executing the wrapped function.
**CN:** `is_aiter_found_and_supported()` 判断的是“这台机器能不能跑 AITER”（ROCm + MI3xx + 库存在），而不是“当前策略要不要启用”，因此它故意不看环境变量。`_load_gemm_tuned_configs()` / `_check_kernel_tuned()` 会读取 AITER 的 autotune CSV，按权重量化 dtype 过滤，并判断某个 `(N, K)` 是否存在任意已调优的 `M`。装饰器则在不支持的平台上直接返回 `None`，避免误执行。

### Fused MoE bridge
```python
def _rocm_aiter_fused_moe_impl(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_weight: torch.Tensor,
    topk_ids: torch.Tensor,
    expert_mask: torch.Tensor | None = None,
    activation_method: int = 0,
    quant_method: int = 0,
    doweight_stage1: bool = False,
    w1_scale: torch.Tensor | None = None,
    w2_scale: torch.Tensor | None = None,
    a1_scale: torch.Tensor | None = None,
    a2_scale: torch.Tensor | None = None,
    num_local_tokens: torch.Tensor | None = None,
    output_dtype: torch.dtype | None = None,
    hidden_pad: int = 0,
    intermediate_pad: int = 0,
    bias1: torch.Tensor | None = None,
    bias2: torch.Tensor | None = None,
) -> torch.Tensor:
    from aiter import ActivationType, QuantType
    from aiter.fused_moe import fused_moe

    activation = ActivationType(activation_method)
    quant_type = QuantType(quant_method)

    return fused_moe(
        hidden_states,
        w1,
        w2,
        topk_weight,
        topk_ids,
        expert_mask,
        activation,
        quant_type,
        doweight_stage1,
        w1_scale,
        w2_scale,
        a1_scale,
        a2_scale,
        num_local_tokens=num_local_tokens,
        dtype=output_dtype,
        hidden_pad=hidden_pad,
        intermediate_pad=intermediate_pad,
        bias1=bias1,
        bias2=bias2,
    )


def _rocm_aiter_fused_moe_fake(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_weight: torch.Tensor,
    topk_ids: torch.Tensor,
    expert_mask: torch.Tensor | None = None,
    activation_method: int = 0,
    quant_method: int = 0,
    doweight_stage1: bool = False,
    w1_scale: torch.Tensor | None = None,
    w2_scale: torch.Tensor | None = None,
    a1_scale: torch.Tensor | None = None,
    a2_scale: torch.Tensor | None = None,
    num_local_tokens: torch.Tensor | None = None,
    output_dtype: torch.dtype | None = None,
    hidden_pad: int = 0,
    intermediate_pad: int = 0,
    bias1: torch.Tensor | None = None,
    bias2: torch.Tensor | None = None,
) -> torch.Tensor:
    if output_dtype is not None:
        return torch.empty_like(hidden_states, dtype=output_dtype)
    return torch.empty_like(hidden_states)
```
**EN:** This wrapper converts integer `activation_method` / `quant_method` ids into AITER enums and forwards the full set of optional scales, paddings, token-count hints, and biases into `aiter.fused_moe.fused_moe`. The fake path mirrors the output allocation contract by preserving shape and honoring `output_dtype` when supplied, which is crucial for tracing and graph compilation without the real ROCm extension.
**CN:** 这里把整数形式的 `activation_method` / `quant_method` 转成 AITER 枚举，并把各种可选 scale、padding、token 数提示和 bias 原样传给 `aiter.fused_moe.fused_moe`。fake 实现则严格复刻输出分配约定：保持形状不变，并在给出 `output_dtype` 时返回对应 dtype，这对没有真实 ROCm 扩展时的 tracing 与图编译非常关键。

### Routing kernels
```python
def _rocm_aiter_topk_softmax_impl(
    topk_weights: torch.Tensor,
    topk_indices: torch.Tensor,
    token_expert_indices: torch.Tensor,
    gating_output: torch.Tensor,
    renormalize: bool,
    num_shared_experts: int = 0,
    shared_expert_scoring_func: str = "",
) -> None:
    from aiter import topk_softmax

    topk_softmax(
        topk_weights,
        topk_indices,
        token_expert_indices,
        gating_output,
        renormalize,
        num_shared_experts,
        shared_expert_scoring_func,
    )


def _rocm_aiter_topk_softmax_fake(
    topk_weights: torch.Tensor,
    topk_indices: torch.Tensor,
    token_expert_indices: torch.Tensor,
    gating_output: torch.Tensor,
    renormalize: bool,
    num_shared_experts: int = 0,
    shared_expert_scoring_func: str = "",
) -> None:
    pass


def _rocm_aiter_topk_sigmoid_impl(
    topk_weights: torch.Tensor,
    topk_indices: torch.Tensor,
    gating_output: torch.Tensor,
) -> None:
    from aiter import topk_sigmoid

    topk_sigmoid(topk_weights, topk_indices, gating_output)


def _rocm_aiter_topk_sigmoid_fake(
    topk_weights: torch.Tensor,
    topk_indices: torch.Tensor,
    gating_output: torch.Tensor,
) -> None:
    pass


def _rocm_aiter_biased_grouped_topk_impl(
    gating_output: torch.Tensor,
    correction_bias: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    num_expert_group: int,
    topk_group: int,
    need_renorm: bool,
    routed_scaling_factor: float = 1.0,  # mul to topk_weights
) -> None:
    from aiter import biased_grouped_topk

    biased_grouped_topk(
        gating_output,
        correction_bias,
        topk_weights,
        topk_ids,
        num_expert_group,
        topk_group,
        need_renorm,
        routed_scaling_factor,
    )


def _rocm_aiter_biased_grouped_topk_fake(
    gating_output: torch.Tensor,
    correction_bias: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    num_expert_group: int,
    topk_group: int,
    need_renorm: bool,
    routed_scaling_factor: float = 1.0,  # mul to topk_weights
) -> None:
    pass


def _rocm_aiter_grouped_topk_impl(
    gating_output: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    num_expert_group: int,
    topk_group: int,
    need_renorm: bool,
    scoring_func: str = "softmax",
    routed_scaling_factor: float = 1.0,  # mul to topk_weights
) -> None:
    is_softmax = scoring_func == "softmax"
    from aiter import grouped_topk

    grouped_topk(
        gating_output,
        topk_weights,
        topk_ids,
        num_expert_group,
        topk_group,
        need_renorm,
        is_softmax,
        routed_scaling_factor,
    )


def _rocm_aiter_grouped_topk_fake(
    gating_output: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    num_expert_group: int,
    topk_group: int,
    need_renorm: bool,
    scoring_func: str = "softmax",
    routed_scaling_factor: float = 1.0,  # mul to topk_weights
) -> None:
    pass


def _rocm_aiter_fused_topk_impl(
    x: torch.Tensor,
    router_logits: torch.Tensor,
    top_k: int,
    gate_up: bool,
) -> tuple[torch.Tensor, torch.Tensor]:
    from aiter.fused_moe import fused_topk

    # fused_topk returns (topk_weights, topk_indices)
    return fused_topk(x, router_logits, top_k, gate_up)


def _rocm_aiter_fused_topk_fake(
    x: torch.Tensor,
    router_logits: torch.Tensor,
    top_k: int,
    gate_up: bool,
) -> tuple[torch.Tensor, torch.Tensor]:
    num_tokens = x.shape[0]
    topk_weights = torch.empty(
        (num_tokens, top_k), dtype=torch.float32, device=x.device
    )
    topk_indices = torch.empty((num_tokens, top_k), dtype=torch.int32, device=x.device)
    return topk_weights, topk_indices
```
**EN:** This block exposes multiple router kernels with different scoring semantics: softmax, sigmoid, grouped top-k, biased grouped top-k, and a fused top-k API that allocates its own outputs. Several of these operators mutate preallocated `topk_weights` / `topk_ids` buffers in-place, matching Torch custom-op schemas that minimize temporary allocations in MoE routing hot paths.
**CN:** 这一段暴露了多种路由内核：softmax、sigmoid、grouped top-k、带校正 bias 的 grouped top-k，以及会自己分配输出的 fused top-k。这里有不少算子是原地写入预分配好的 `topk_weights` / `topk_ids`，这与 Torch 自定义算子在 MoE 热路径里尽量减少临时分配的设计一致。

### MLA capability probing and decode
```python
# Cache whether aiter supports FP8 MLA parameters
_AITER_MLA_SUPPORTS_FP8: bool | None = None
_AITER_HAS_FUSED_QK_RMSNORM: bool | None = None


def check_aiter_fused_qk_rmsnorm() -> bool:
    """Check if aiter provides fused_qk_rmsnorm.

    Supports both the new private name ``_fused_qk_rmsnorm``
    (AITER >= PR #2958) and the old public name ``fused_qk_rmsnorm``
    (AITER >= PR #2442).

    TODO(rbrugaro-amd): remove the legacy fused_qk_rmsnorm path once
    AITER stabilizes the API (https://github.com/ROCm/aiter/issues/3207).
    """
    global _AITER_HAS_FUSED_QK_RMSNORM
    if _AITER_HAS_FUSED_QK_RMSNORM is None:
        try:
            from aiter.ops.fused_qk_norm_rope_cache_quant import (  # noqa: F401
                _fused_qk_rmsnorm,
            )

            _AITER_HAS_FUSED_QK_RMSNORM = True
        except (ImportError, ModuleNotFoundError, AttributeError):
            try:
                from aiter.ops.fused_qk_norm_rope_cache_quant import (  # noqa: F401
                    fused_qk_rmsnorm,
                )

                _AITER_HAS_FUSED_QK_RMSNORM = True
            except (ImportError, ModuleNotFoundError, AttributeError):
                _AITER_HAS_FUSED_QK_RMSNORM = False
    return _AITER_HAS_FUSED_QK_RMSNORM


def _check_aiter_mla_fp8_support() -> bool:
    """Check if aiter.mla.mla_decode_fwd supports q_scale and kv_scale parameters."""
    global _AITER_MLA_SUPPORTS_FP8
    if _AITER_MLA_SUPPORTS_FP8 is None:
        try:
            import inspect

            from aiter.mla import mla_decode_fwd

            sig = inspect.signature(mla_decode_fwd)
            _AITER_MLA_SUPPORTS_FP8 = (
                "q_scale" in sig.parameters and "kv_scale" in sig.parameters
            )
        except (
            ImportError,
            ModuleNotFoundError,
            AttributeError,
            ValueError,
            TypeError,
        ):
            # ImportError/ModuleNotFoundError: aiter.mla module not available
            # AttributeError: mla_decode_fwd doesn't exist
            # ValueError: mla_decode_fwd has no signature (e.g., built-in)
            # TypeError: mla_decode_fwd is not a callable
            _AITER_MLA_SUPPORTS_FP8 = False
    return _AITER_MLA_SUPPORTS_FP8


def _rocm_aiter_mla_decode_fwd_impl(
    q: torch.Tensor,
    kv_buffer: torch.Tensor,
    o: torch.Tensor,
    qo_indptr: torch.Tensor,
    max_seqlen_qo: int,
    kv_indptr: torch.Tensor | None = None,
    kv_indices: torch.Tensor | None = None,
    kv_last_page_lens: torch.Tensor | None = None,
    sm_scale: float = 1.0,
    logit_cap: float = 0.0,
    q_scale: torch.Tensor | None = None,
    kv_scale: torch.Tensor | None = None,
    work_meta_data: torch.Tensor | None = None,
    work_indptr: torch.Tensor | None = None,
    work_info_set: torch.Tensor | None = None,
    reduce_indptr: torch.Tensor | None = None,
    reduce_final_map: torch.Tensor | None = None,
    reduce_partial_map: torch.Tensor | None = None,
) -> None:
    from aiter.mla import mla_decode_fwd

    kwargs: dict[str, float | torch.Tensor | None] = {
        "sm_scale": sm_scale,
        "logit_cap": logit_cap,
    }

    # Only pass q_scale and kv_scale if the aiter library supports them
    if _check_aiter_mla_fp8_support():
        kwargs["q_scale"] = q_scale
        kwargs["kv_scale"] = kv_scale

    if work_meta_data is not None:
        assert work_indptr is not None, (
            "work_indptr must be provided with work_meta_data"
        )
        assert work_info_set is not None, (
            "work_info_set must be provided with work_meta_data"
        )
        assert reduce_indptr is not None, (
            "reduce_indptr must be provided with work_meta_data"
        )
        assert reduce_final_map is not None, (
            "reduce_final_map must be provided with work_meta_data"
        )
        assert reduce_partial_map is not None, (
            "reduce_partial_map must be provided with work_meta_data"
        )
        kwargs["work_meta_data"] = work_meta_data
        kwargs["work_indptr"] = work_indptr
        kwargs["work_info_set"] = work_info_set
        kwargs["reduce_indptr"] = reduce_indptr
        kwargs["reduce_final_map"] = reduce_final_map
        kwargs["reduce_partial_map"] = reduce_partial_map

    mla_decode_fwd(
        q,
        kv_buffer.view(-1, 1, 1, q.shape[-1]),
        o,
        qo_indptr,
        kv_indptr,
        kv_indices,
        kv_last_page_lens,
        max_seqlen_qo,
        **kwargs,
    )
```
**EN:** AITER's Python API changed over time, so the module lazily probes whether `mla_decode_fwd` accepts `q_scale` and `kv_scale` before passing FP8 metadata. The implementation also conditionally appends work-partition / reduction tensors only when sparse MLA metadata is present, and reshapes `kv_buffer` to the `[pages, 1, 1, head_dim]` layout expected by the downstream decoder without changing storage.
**CN:** 由于 AITER 的 Python API 有过演进，这里会延迟探测 `mla_decode_fwd` 是否支持 `q_scale` 和 `kv_scale`，只有支持时才传 FP8 元数据。实现还会在存在稀疏 MLA 工作划分信息时才附加 work/reduce 张量，并把 `kv_buffer` 视图重排为下游解码器要求的 `[pages, 1, 1, head_dim]` 形状，而不改变底层存储。

### Quantized RMSNorm and custom allreduce
```python
def _rocm_aiter_rmsnorm_fused_add_dynamic_quant_impl(
    x: torch.Tensor,
    residual: torch.Tensor,
    weight: torch.Tensor,
    epsilon: float,
    quant_dtype: torch.dtype,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    import aiter as rocm_aiter

    assert quant_dtype in [torch.int8, FP8_DTYPE]

    y_scale = torch.empty(x.shape[0], 1, dtype=torch.float32, device=x.device)
    out = torch.empty(x.shape, dtype=quant_dtype, device=x.device)
    residual_out = torch.empty_like(x)

    rocm_aiter.rmsnorm2d_fwd_with_add_dynamicquant(
        out,
        x,
        residual,
        residual_out,
        y_scale,
        weight,
        epsilon,
        use_model_sensitive_rmsnorm=0,
    )

    return out, residual_out, y_scale


def _rocm_aiter_rmsnorm_fused_add_dynamic_quant_fake(
    x: torch.Tensor,
    residual: torch.Tensor,
    weight: torch.Tensor,
    epsilon: float,
    quant_dtype: torch.dtype,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    y_scale = torch.empty(x.shape[0], 1, dtype=torch.float32, device=x.device)
    out = torch.empty(x.shape, dtype=quant_dtype, device=x.device)
    residual_out = torch.empty_like(x)

    return out, residual_out, y_scale


def _rocm_aiter_rmsnorm_fused_dynamic_quant_impl(
    x: torch.Tensor,
    weight: torch.Tensor,
    epsilon: float,
    quant_dtype: torch.dtype,
) -> tuple[torch.Tensor, torch.Tensor]:
    import aiter as rocm_aiter

    assert quant_dtype in [torch.int8, FP8_DTYPE]

    y_scale = torch.empty(x.shape[0], 1, dtype=torch.float32, device=x.device)
    out = torch.empty(x.shape, dtype=quant_dtype, device=x.device)

    rocm_aiter.rmsnorm2d_fwd_with_dynamicquant(
        out, x, y_scale, weight, epsilon, use_model_sensitive_rmsnorm=0
    )

    return out, y_scale


def _rocm_aiter_rmsnorm_fused_dynamic_quant_fake(
    x: torch.Tensor,
    weight: torch.Tensor,
    epsilon: float,
    quant_dtype: torch.dtype,
) -> tuple[torch.Tensor, torch.Tensor]:
    y_scale = torch.empty(x.shape[0], 1, dtype=torch.float32, device=x.device)
    out = torch.empty(x.shape, dtype=quant_dtype, device=x.device)

    return out, y_scale


def _rocm_aiter_fused_allreduce_rmsnorm_impl(
    input_: torch.Tensor,
    residual: torch.Tensor,
    weight: torch.Tensor,
    epsilon: float,
) -> tuple[torch.Tensor, torch.Tensor]:
    aiter_ar = rocm_aiter_ops.get_aiter_allreduce()
    assert aiter_ar is not None, "aiter allreduce must be initialized"

    total_bytes = input_.numel() * input_.element_size()
    hidden_dim = input_.shape[-1]
    token_num = input_.shape[0]
    if input_.dtype in (torch.bfloat16, torch.float16):
        pack_size = 16 // input_.element_size()
        hidden_ok = hidden_dim % pack_size == 0 and hidden_dim // pack_size <= 1024
    else:
        hidden_ok = False
    token_ok = token_num <= 80
    world_size = aiter_ar.world_size
    full_nvlink = aiter_ar.fully_connected

    if world_size == 2:
        size_ok = True
    elif full_nvlink and world_size <= 4:
        size_ok = total_bytes < 256 * 1024
    elif full_nvlink and world_size <= 8:
        size_ok = total_bytes < 128 * 1024
    else:
        size_ok = False

    use_1stage = hidden_ok and token_ok and size_ok

    result = aiter_ar.fused_ar_rms(
        input_,
        residual,
        w=weight,
        eps=epsilon,
        registered=torch.cuda.is_current_stream_capturing(),
        use_1stage=use_1stage,
    )
    assert result is not None
    return result[0], result[1]
```
**EN:** These helpers allocate the exact output / residual / scale buffers expected by AITER's fused RMSNorm+quant kernels, then hand execution to ROCm kernels that emit quantized activations and dynamic scales together. `_rocm_aiter_fused_allreduce_rmsnorm_impl()` adds transport heuristics on top: it enables the 1-stage fused allreduce path only when datatype packing, hidden width, token count, world size, and NVLink connectivity suggest the smaller latency-optimized algorithm will be beneficial.
**CN:** 这些辅助函数先按 AITER 融合 RMSNorm+量化内核要求分配好输出、残差和 scale 缓冲区，再交给 ROCm 内核同时产出量化后的激活与动态 scale。`_rocm_aiter_fused_allreduce_rmsnorm_impl()` 还叠加了通信启发式：只有当 dtype 打包方式、hidden 宽度、token 数、world size 和 NVLink 拓扑都合适时，才启用低延迟的 1-stage fused allreduce 路径。

### FP8 group-quant fusion
```python
def _rocm_aiter_rmsnorm_with_add_fp8_group_quant_impl(
    x: torch.Tensor,
    residual: torch.Tensor,
    weight: torch.Tensor,
    variance_epsilon: float,
    group_size: int,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    from aiter.ops.triton.fused_fp8_quant import fused_rms_fp8_group_quant

    (x_quant, x_quant_scales), _, _, res = fused_rms_fp8_group_quant(
        x,
        weight,
        variance_epsilon,
        None,
        None,
        None,
        group_size=group_size,
        dtype_quant=FP8_DTYPE,
        res1=residual,
    )
    return (
        x_quant,
        res,
        x_quant_scales,
    )


def _rocm_aiter_rmsnorm_with_add_fp8_group_quant_fake(
    x: torch.Tensor,
    residual: torch.Tensor,
    weight: torch.Tensor,
    variance_epsilon: float,
    group_size: int,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    M, N = x.shape
    scale_shape = (M, (N + group_size - 1) // group_size)
    return (
        torch.empty_like(x, dtype=FP8_DTYPE, device=x.device),
        torch.empty_like(residual, device=residual.device),
        torch.empty(scale_shape, dtype=torch.float32, device=x.device),
    )


def _rocm_aiter_rmsnorm_fp8_group_quant_impl(
    x: torch.Tensor,
    weight: torch.Tensor,
    variance_epsilon: float,
    group_size: int,
) -> tuple[torch.Tensor, torch.Tensor]:
    from aiter.ops.triton.fused_fp8_quant import fused_rms_fp8_group_quant

    (x_quant, x_quant_scales), _, _, res = fused_rms_fp8_group_quant(
        x,
        weight,
        variance_epsilon,
        None,
        None,
        None,
        group_size=group_size,
        dtype_quant=FP8_DTYPE,
        res1=None,
    )
    return (x_quant, x_quant_scales)


def _rocm_aiter_rmsnorm_fp8_group_quant_fake(
    x: torch.Tensor,
    weight: torch.Tensor,
    variance_epsilon: float,
    group_size: int,
) -> tuple[torch.Tensor, torch.Tensor]:
    M, N = x.shape
    scale_shape = (M, (N + group_size - 1) // group_size)
    return (
        torch.empty_like(x, dtype=FP8_DTYPE, device=x.device),
        torch.empty(scale_shape, dtype=torch.float32, device=x.device),
    )


def _rocm_aiter_fused_rms_gated_fp8_group_quant_impl(
    x: torch.Tensor,
    weight: torch.Tensor,
    bias: torch.Tensor | None,
    z: torch.Tensor,
    eps: float,
    norm_before_gate: bool,
    activation: str,
    group_size: int,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Fused gated-RMSNorm + FP8 group quantization via aiter Triton kernel."""
    from aiter.ops.triton.quant import fused_rms_gated_fp8_group_quant

    return fused_rms_gated_fp8_group_quant(
        x,
        weight,
        bias,
        z,
        eps,
        norm_before_gate=norm_before_gate,
        activation=activation,
        out_dtype=FP8_DTYPE,
        group_size=group_size,
    )
```
**EN:** This family wraps Triton kernels from AITER that fuse RMSNorm, optional residual addition, optional gated activation, and groupwise FP8 quantization into a single launch. The returned values are not only quantized activations: the code also materializes per-group float32 scale tensors whose shapes are derived from `group_size`, which later GEMM kernels need to dequantize correctly.
**CN:** 这一组接口包装了 AITER 的 Triton 内核，把 RMSNorm、可选 residual add、可选门控激活以及按组 FP8 量化融合到一次 launch 里。返回值不只是量化后的激活，还会显式产出按 `group_size` 推导出来的每组 float32 scale，这些 scale 是后续 GEMM 正确反量化所必需的。

### Dual RMSNorm, FP4 GEMM, and rotary
```python
def _fused_mla_dual_rms_norm_impl(
    x1: torch.Tensor,
    x1_weight: torch.Tensor,
    x2: torch.Tensor,
    x2_weight: torch.Tensor,
    x1_epsilon: float,
    x2_epsilon: float,
) -> tuple[torch.Tensor, torch.Tensor]:
    try:
        import aiter.ops.fused_qk_norm_rope_cache_quant as aiter_ops
    except (ImportError, ModuleNotFoundError, AttributeError) as exc:
        raise ImportError(
            "fused_qk_rmsnorm requires AITer >= PR #2442. "
            "Please upgrade aiter or disable the "
            "fuse_mla_dual_rms_norm pass."
        ) from exc

    if hasattr(aiter_ops, "_fused_qk_rmsnorm"):
        return aiter_ops._fused_qk_rmsnorm(
            q_out=None,
            q=x1,
            q_weight=x1_weight,
            q_eps=x1_epsilon,
            k_out=None,
            k=x2,
            k_weight=x2_weight,
            k_eps=x2_epsilon,
        )

    # TODO(rbrugaro-amd): remove the legacy fused_qk_rmsnorm path once
    # AITER stabilizes the API (https://github.com/ROCm/aiter/issues/3207).
    if hasattr(aiter_ops, "fused_qk_rmsnorm"):
        return aiter_ops.fused_qk_rmsnorm(
            q=x1,
            q_weight=x1_weight,
            q_eps=x1_epsilon,
            k=x2,
            k_weight=x2_weight,
            k_eps=x2_epsilon,
        )

    raise ImportError(
        "fused_qk_rmsnorm requires AITer >= PR #2442. "
        "Please upgrade aiter or disable the "
        "fuse_mla_dual_rms_norm pass."
    )


def _fused_mla_dual_rms_norm_fake(
    x1: torch.Tensor,
    x1_weight: torch.Tensor,
    x2: torch.Tensor,
    x2_weight: torch.Tensor,
    x1_epsilon: float,
    x2_epsilon: float,
) -> tuple[torch.Tensor, torch.Tensor]:
    return (torch.empty_like(x1), torch.empty_like(x2))


def _rocm_aiter_gemm_a8wfp4_impl(
    x: torch.Tensor,
    w: torch.Tensor,
    x_scales: torch.Tensor,
    w_scales: torch.Tensor,
    out_dtype: torch.dtype,
) -> torch.Tensor:
    from aiter.ops.triton.gemm_a8wfp4 import gemm_a8wfp4

    M, N = x.shape[0], w.shape[0]
    y = torch.empty(M, N, dtype=out_dtype, device=x.device)
    gemm_a8wfp4(
        x=x,
        w=w,
        y=y,
        x_scales=x_scales,
        w_scales=w_scales,
        dtype=out_dtype,
        config=None,
    )
    return y


def _rocm_aiter_gemm_a8wfp4_fake(
    x: torch.Tensor,
    w: torch.Tensor,
    x_scales: torch.Tensor,
    w_scales: torch.Tensor,
    out_dtype: torch.dtype,
) -> torch.Tensor:
    return torch.empty(x.shape[0], w.shape[0], dtype=out_dtype, device=x.device)


def _triton_rotary_embedding_impl(
    positions: torch.Tensor,
    query: torch.Tensor,
    key: torch.Tensor,
    head_size: int,
    cos_sin_cache: torch.Tensor,
    is_neox: bool,
    offsets: torch.Tensor | None = None,
) -> None:
    # Modifies query and key in-place
    from aiter.ops.triton.rope.rope import (
        rope_cached_thd_positions_offsets_2c_fwd_inplace,
    )

    num_tokens = positions.numel()
    cos, sin = cos_sin_cache.chunk(2, dim=-1)
    query_shape = query.shape
    key_shape = key.shape
    rotate_style = 0 if is_neox else 1
    rotary_dim = head_size

    query = query.view(num_tokens, -1, head_size)
    key = key.view(num_tokens, -1, head_size)
    query_ = query[..., :rotary_dim]
    key_ = key[..., :rotary_dim]
    positions = positions.view(*query.shape[:1])
    rope_cached_thd_positions_offsets_2c_fwd_inplace(
        query_,
        key_,
        cos,
        sin,
        positions,
        offsets,
        rotate_style,
        reuse_freqs_front_part=True,
        nope_first=False,
    )
    query = query.view(query_shape)
    key = key.view(key_shape)
```
**EN:** `_fused_mla_dual_rms_norm_impl()` is a compatibility shim for an unstable AITER API: it accepts both the newer private `_fused_qk_rmsnorm` symbol and the older public `fused_qk_rmsnorm` symbol. The following helpers then bridge into FP4 GEMM and Triton rotary embedding, reshaping `query` / `key` to `[tokens, heads, head_size]` views and rotating only the prefix slice that belongs to the RoPE dimension.
**CN:** `_fused_mla_dual_rms_norm_impl()` 是对 AITER 不稳定 API 的兼容层：它同时兼容较新的私有符号 `_fused_qk_rmsnorm` 和较旧的公共符号 `fused_qk_rmsnorm`。后面的辅助函数再进一步桥接到 FP4 GEMM 与 Triton rotary embedding，通过把 `query` / `key` 重解释为 `[tokens, heads, head_size]` 视图，只旋转属于 RoPE 维度的前缀部分。

### Environment flags and feature predicates
```python
    # Check if the env variable is set
    _AITER_ENABLED = envs.VLLM_ROCM_USE_AITER
    _LINEAR_ENABLED = envs.VLLM_ROCM_USE_AITER_LINEAR
    _FMOE_ENABLED = envs.VLLM_ROCM_USE_AITER_MOE
    _MLA_ENABLED = envs.VLLM_ROCM_USE_AITER_MLA
    _MHA_ENABLED = envs.VLLM_ROCM_USE_AITER_MHA
    _SHUFFLE_KV_CACHE_ENABLED = envs.VLLM_ROCM_SHUFFLE_KV_CACHE_LAYOUT
    _TRITON_UNIFIED_ATTN_ENABLED = envs.VLLM_ROCM_USE_AITER_UNIFIED_ATTENTION
    # TODO: Consolidate under _LINEAR_ENABLED
    _FP8BMM_ENABLED = envs.VLLM_ROCM_USE_AITER_FP8BMM
    _FP4BMM_ENABLED = envs.VLLM_ROCM_USE_AITER_FP4BMM
    # TODO: Consolidate under _LINEAR_ENABLED
    _FP4_GEMM_DYNAMIC_QUANT_ASM = envs.VLLM_ROCM_USE_AITER_FP4_ASM_GEMM
    # TODO: Consolidate under VLLM_ROCM_USE_AITER_ROPE
    _TRITON_ROTARY_EMBED = envs.VLLM_ROCM_USE_AITER_TRITON_ROPE
    _MOE_SHARED_EXPERTS_ENABLED = envs.VLLM_ROCM_USE_AITER_FUSION_SHARED_EXPERTS
    # TODO: Consolidate under _LINEAR_ENABLED
    _TRITON_UNQUANT_GEMM = envs.VLLM_ROCM_USE_AITER_TRITON_GEMM
    # Lazily probed: whether aiter.topk_softmax supports the
    # num_shared_experts / shared_expert_scoring_func args (7-arg form).
    _TOPK_SOFTMAX_FUSED_SIGMOID: bool | None = None

    _ALL_REDUCE_MAX_SIZE: int = 8192 * 1024 * 8 * 2
    _CUSTOM_ALL_REDUCE: AiterCustomAllreduceProto | None = None

    @classmethod
    def refresh_env_variables(cls):
        """
        Since the environment variables are assigned when the module is imported,
        This is a helper function to reload all the env variables from
        the environment variables.
        for example, after monkey patching the env variables in the unit test,
        you can call this function to reload the env variables.
        """
        cls._AITER_ENABLED = envs.VLLM_ROCM_USE_AITER
        cls._LINEAR_ENABLED = envs.VLLM_ROCM_USE_AITER_LINEAR
        cls._FMOE_ENABLED = envs.VLLM_ROCM_USE_AITER_MOE
        cls._MLA_ENABLED = envs.VLLM_ROCM_USE_AITER_MLA
        cls._MHA_ENABLED = envs.VLLM_ROCM_USE_AITER_MHA
        cls._SHUFFLE_KV_CACHE_ENABLED = envs.VLLM_ROCM_SHUFFLE_KV_CACHE_LAYOUT
        cls._TRITON_UNIFIED_ATTN_ENABLED = envs.VLLM_ROCM_USE_AITER_UNIFIED_ATTENTION
        cls._FP8BMM_ENABLED = envs.VLLM_ROCM_USE_AITER_FP8BMM
        cls._FP4BMM_ENABLED = envs.VLLM_ROCM_USE_AITER_FP4BMM
        cls._FP4_GEMM_DYNAMIC_QUANT_ASM = envs.VLLM_ROCM_USE_AITER_FP4_ASM_GEMM
        cls._TRITON_ROTARY_EMBED = envs.VLLM_ROCM_USE_AITER_TRITON_ROPE
        cls._MOE_SHARED_EXPERTS_ENABLED = envs.VLLM_ROCM_USE_AITER_FUSION_SHARED_EXPERTS
        cls._TRITON_UNQUANT_GEMM = envs.VLLM_ROCM_USE_AITER_TRITON_GEMM

    @staticmethod
    def get_aiter_activation_type(activation_str: str):
        """
        Given an activation type as a string, returns the corresponding aiter ActivationType enum.
        Supported activation types: "no", "none", "silu", "gelu", "swiglu".
        Returns None if the mapping fails.

        Args:
            activation_str (str): Activation type as string.

        Returns:
            Aiter ActivationType enum value, or None if not found.
        """
        # Import only locally, since aiter may not always be available.
        try:
            from aiter import ActivationType
        except ImportError:
            return None

        if not isinstance(activation_str, str):
            return None

        name = activation_str.strip().lower()
        mapping = {
            "none": ActivationType.No,
            "no": ActivationType.No,
            "silu": ActivationType.Silu,
            "gelu": ActivationType.Gelu,
            "swiglu": ActivationType.Swiglu,
        }
        return mapping.get(name)

    @staticmethod
    def get_aiter_quant_type(quant_type_str: str):
        """
        Given a quantization type as a string, returns the corresponding aiter QuantType enum.
        Supported quantization types: "no", "per_tensor", "per_token", "per_1x32", "per_1x128", "per_128x128".
        Returns None if the mapping fails.

        Args:
            quant_type_str (str): Quantization type as string.

        Returns:
            Aiter QuantType enum value, or None if not found.
        """
        try:
            from aiter import QuantType
        except ImportError:
            return None

        if not isinstance(quant_type_str, str):
            return None

        name = quant_type_str.strip().lower()
        mapping = {
            "no": QuantType.No,
            "per_tensor": QuantType.per_Tensor,
            "per_token": QuantType.per_Token,
            "per_1x32": QuantType.per_1x32,
            "per_1x128": QuantType.per_1x128,
            "per_128x128": QuantType.per_128x128,
        }
        return mapping.get(name)

    @classmethod
    @if_aiter_supported
    def is_enabled(cls) -> bool:
        return cls._AITER_ENABLED

    @classmethod
    @if_aiter_supported
    def is_linear_enabled(cls) -> bool:
        return cls._AITER_ENABLED and cls._LINEAR_ENABLED

    @classmethod
    @if_aiter_supported
    def is_linear_fp8_enabled(cls) -> bool:
        return cls.is_linear_enabled()

    @classmethod
    @if_aiter_supported
    def is_fused_moe_enabled(cls) -> bool:
        return cls._AITER_ENABLED and cls._FMOE_ENABLED

    @classmethod
    @if_aiter_supported
    def is_fusion_moe_shared_experts_enabled(cls) -> bool:
        return cls.is_fused_moe_enabled() and cls._MOE_SHARED_EXPERTS_ENABLED

    @classmethod
    @if_aiter_supported
    def topk_softmax_supports_fused_sigmoid(cls) -> bool:
        """Check if topk_softmax supports fused shared expert activation."""
        if cls._TOPK_SOFTMAX_FUSED_SIGMOID is None:
            try:
                import inspect

                from aiter import topk_softmax

                params = inspect.signature(topk_softmax).parameters
                if "num_shared_experts" in params:
                    cls._TOPK_SOFTMAX_FUSED_SIGMOID = True
                else:
                    # @compile_ops wrapper loses the original signature.
                    # Fall back to the torch custom op schema.
                    import torch

                    schema = getattr(
                        getattr(torch.ops.aiter, "topk_softmax", None), "default", None
                    )
                    schema_str = str(getattr(schema, "_schema", ""))
                    cls._TOPK_SOFTMAX_FUSED_SIGMOID = "num_shared_experts" in schema_str
            except (ImportError, ValueError):
                cls._TOPK_SOFTMAX_FUSED_SIGMOID = False
        return cls._TOPK_SOFTMAX_FUSED_SIGMOID

    @classmethod
    @if_aiter_supported
    def fuse_sigmoid_in_kernel(cls, aiter_topK_meta_data: object) -> bool:
        """Whether fused shared-expert sigmoid in the topk kernel is usable.

        Combines the cached static capability checks (FSE enabled, fused-moe
        enabled, topk_softmax supports fused sigmoid) with the runtime
        readiness check (topK meta-data buffer initialized).

        ``aiter_topK_meta_data`` is accepted as a parameter rather than
        imported internally so callers cannot hit initialization-order
        issues where the module-level global has not been set yet.
        """
        return (
            cls.is_fusion_moe_shared_experts_enabled()
            and cls.topk_softmax_supports_fused_sigmoid()
            and aiter_topK_meta_data is not None
        )
```
**EN:** The class snapshots a large set of ROCm environment variables at import time, turning policy checks into cheap boolean reads on the hot path. It also exposes string-to-enum helpers plus cached feature predicates, including a signature/schema probe that detects whether `aiter.topk_softmax` supports fused shared-expert sigmoid arguments even when decorators hide the original Python signature.
**CN:** 这个类在导入时一次性快照了大量 ROCm 环境变量，从而把热路径中的策略判断变成廉价的布尔读取。它还提供字符串到枚举的映射函数，以及带缓存的特性判定逻辑；其中 `topk_softmax` 的判定甚至会在 Python 签名被装饰器遮蔽时退回到 Torch schema 字符串进行探测。

### Allreduce lifecycle and GDN kernel probing
```python
    @classmethod
    def initialize_aiter_allreduce(
        cls, group: ProcessGroup, device: torch.device
    ) -> None:
        try:
            from aiter.dist.device_communicators.custom_all_reduce import (
                CustomAllreduce as AiterCustomAllreduce,
            )

            cls._CUSTOM_ALL_REDUCE = AiterCustomAllreduce(group, device)
        except Exception:
            cls._CUSTOM_ALL_REDUCE = None

    @classmethod
    def get_aiter_allreduce(cls) -> AiterCustomAllreduceProto | None:
        return cls._CUSTOM_ALL_REDUCE

    @classmethod
    def destroy_aiter_allreduce(cls) -> None:
        if cls._CUSTOM_ALL_REDUCE is not None:
            cls._CUSTOM_ALL_REDUCE.close()
            cls._CUSTOM_ALL_REDUCE = None

    @classmethod
    def get_aiter_allreduce_max_size(cls) -> int | None:
        # effective max input size (based on upstream aiter version: v0.1.10.post3)
        # https://github.com/ROCm/aiter/blob/6a0e7b26ccf33164785531212cc2ec2cde0b9243/aiter/dist/device_communicators/custom_all_reduce.py#L272-L273
        return int(cls._ALL_REDUCE_MAX_SIZE / 2)

    @classmethod
    @if_aiter_supported
    def are_gdn_triton_kernels_available(cls) -> bool:
        """Check if AITER Triton kernels for GDN attention are importable.

        These are optional Triton kernels (conv1d fast-path, gated delta net)
        used by GatedDeltaNetAttention's decode fast-path.  They may be absent
        in older aiter builds.
        """
        if not cls._AITER_ENABLED:
            return False
        try:
            import aiter.ops.triton.causal_conv1d_update_single_token  # noqa: F401
            import aiter.ops.triton.gated_delta_net  # noqa: F401
            from aiter.ops.triton.quant import (  # noqa: F401
                fused_rms_gated_fp8_group_quant,
            )

            return True
        except (ImportError, ModuleNotFoundError):
            return False
```
**EN:** Custom allreduce is managed as class-level state: initialize from a `ProcessGroup`, expose the handle, close it on teardown, and derive an effective max-size limit from upstream AITER behavior. `are_gdn_triton_kernels_available()` separately checks whether optional Triton pieces for Gated DeltaNet decode—causal conv1d, gated delta net, and fused gated FP8 quant—can actually be imported on the current install.
**CN:** 自定义 allreduce 以类级状态管理：从 `ProcessGroup` 初始化、对外暴露句柄、在销毁时关闭，并根据上游 AITER 的实现细节推导有效的最大输入大小。`are_gdn_triton_kernels_available()` 则独立检查 Gated DeltaNet 解码快路径所需的可选 Triton 组件——例如 causal conv1d、gated delta net 和融合式 gated FP8 量化——当前环境是否真的可导入。

### Custom-op registration
```python
    @staticmethod
    @if_aiter_supported
    def register_ops_once() -> None:
        global _OPS_REGISTERED
        if not _OPS_REGISTERED:
            # register all the custom ops here
            direct_register_custom_op(
                op_name="rocm_aiter_asm_moe_tkw1",
                op_func=_rocm_aiter_asm_moe_tkw1_impl,
                mutates_args=[],
                fake_impl=_rocm_aiter_asm_moe_tkw1_fake,
                dispatch_key=current_platform.dispatch_key,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_fused_moe",
                op_func=_rocm_aiter_fused_moe_impl,
                mutates_args=[],
                fake_impl=_rocm_aiter_fused_moe_fake,
                dispatch_key=current_platform.dispatch_key,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_topk_softmax",
                op_func=_rocm_aiter_topk_softmax_impl,
                mutates_args=["topk_weights", "topk_indices", "token_expert_indices"],
                fake_impl=_rocm_aiter_topk_softmax_fake,
                dispatch_key=current_platform.dispatch_key,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_topk_sigmoid",
                op_func=_rocm_aiter_topk_sigmoid_impl,
                mutates_args=["topk_weights", "topk_indices"],
                fake_impl=_rocm_aiter_topk_sigmoid_fake,
                dispatch_key=current_platform.dispatch_key,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_biased_grouped_topk",
                op_func=_rocm_aiter_biased_grouped_topk_impl,
                mutates_args=["topk_weights", "topk_ids"],
                fake_impl=_rocm_aiter_biased_grouped_topk_fake,
                dispatch_key=current_platform.dispatch_key,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_grouped_topk",
                op_func=_rocm_aiter_grouped_topk_impl,
                mutates_args=["topk_weights", "topk_ids"],
                fake_impl=_rocm_aiter_grouped_topk_fake,
                dispatch_key=current_platform.dispatch_key,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_fused_topk",
                op_func=_rocm_aiter_fused_topk_impl,
                mutates_args=[],
                fake_impl=_rocm_aiter_fused_topk_fake,
                dispatch_key=current_platform.dispatch_key,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_mla_decode_fwd",
                op_func=_rocm_aiter_mla_decode_fwd_impl,
                mutates_args=["o"],
                fake_impl=_rocm_aiter_mla_decode_fwd_fake,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_w8a8_gemm",
                op_func=_rocm_aiter_w8a8_gemm_impl,
                fake_impl=_rocm_aiter_w8a8_gemm_fake,
            )

            direct_register_custom_op(
                op_name="_rocm_aiter_preshuffled_per_token_w8a8_gemm",
                op_func=_rocm_aiter_preshuffled_per_token_w8a8_gemm_impl,
                fake_impl=_rocm_aiter_preshuffled_per_token_w8a8_gemm_fake,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_triton_gemm_a8w8_blockscale",
                op_func=_rocm_aiter_triton_gemm_a8w8_blockscale_impl,
                fake_impl=_rocm_aiter_triton_gemm_a8w8_blockscale_fake,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_gemm_a8w8_blockscale",
                op_func=_rocm_aiter_gemm_a8w8_blockscale_impl,
                fake_impl=_rocm_aiter_gemm_a8w8_blockscale_fake,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_rmsnorm_fused_dynamic_quant",
                op_func=_rocm_aiter_rmsnorm_fused_dynamic_quant_impl,
                fake_impl=_rocm_aiter_rmsnorm_fused_dynamic_quant_fake,
                dispatch_key=current_platform.dispatch_key,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_rmsnorm_fused_add_dynamic_quant",
                op_func=_rocm_aiter_rmsnorm_fused_add_dynamic_quant_impl,
                fake_impl=_rocm_aiter_rmsnorm_fused_add_dynamic_quant_fake,
                dispatch_key=current_platform.dispatch_key,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_rmsnorm_fp8_group_quant",
                op_func=_rocm_aiter_rmsnorm_fp8_group_quant_impl,
                fake_impl=_rocm_aiter_rmsnorm_fp8_group_quant_fake,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_fused_rms_gated_fp8_group_quant",
                op_func=_rocm_aiter_fused_rms_gated_fp8_group_quant_impl,
                fake_impl=_rocm_aiter_fused_rms_gated_fp8_group_quant_fake,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_rmsnorm_with_add_fp8_group_quant",
                op_func=_rocm_aiter_rmsnorm_with_add_fp8_group_quant_impl,
                fake_impl=_rocm_aiter_rmsnorm_with_add_fp8_group_quant_fake,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_act_mul_and_fp8_group_quant",
                op_func=_rocm_aiter_act_mul_and_fp8_group_quant_impl,
                fake_impl=_rocm_aiter_act_mul_and_fp8_group_quant_fake,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_triton_add_rmsnorm_pad",
                op_func=_rocm_aiter_triton_add_rmsnorm_pad_impl,
                fake_impl=_rocm_aiter_triton_add_rmsnorm_pad_fake,
                dispatch_key=current_platform.dispatch_key,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_group_fp8_quant",
                op_func=_rocm_aiter_group_fp8_quant_impl,
                fake_impl=_rocm_aiter_group_fp8_quant_fake,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_per_tensor_quant",
                op_func=_rocm_aiter_per_tensor_quant_impl,
                mutates_args=[],
                fake_impl=_rocm_aiter_per_tensor_quant_fake,
                dispatch_key=current_platform.dispatch_key,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_per_token_quant",
                op_func=_rocm_aiter_per_token_quant_impl,
                fake_impl=_rocm_aiter_per_token_quant_fake,
                dispatch_key=current_platform.dispatch_key,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_sparse_attn_indexer",
                op_func=rocm_aiter_sparse_attn_indexer,
                mutates_args=["topk_indices_buffer"],
                fake_impl=rocm_aiter_sparse_attn_indexer_fake,
                dispatch_key=current_platform.dispatch_key,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_gemm_a8wfp4",
                op_func=_rocm_aiter_gemm_a8wfp4_impl,
                mutates_args=[],
                fake_impl=_rocm_aiter_gemm_a8wfp4_fake,
                dispatch_key=current_platform.dispatch_key,
            )

            # Register rocm aiter rotary embedding custom op
            direct_register_custom_op(
                op_name="rocm_aiter_triton_rotary_embedding",
                op_func=_triton_rotary_embedding_impl,
                mutates_args=["query", "key"],  # These tensors are modified in-place
                fake_impl=_triton_rotary_embedding_fake,
            )

            direct_register_custom_op(
                op_name="rocm_aiter_fused_allreduce_rmsnorm",
                op_func=_rocm_aiter_fused_allreduce_rmsnorm_impl,
                fake_impl=_rocm_aiter_fused_allreduce_rmsnorm_fake,
            )

            direct_register_custom_op(
                op_name="fused_mla_dual_rms_norm",
                op_func=_fused_mla_dual_rms_norm_impl,
                mutates_args=[],
                fake_impl=_fused_mla_dual_rms_norm_fake,
            )

            _OPS_REGISTERED = True
```
**EN:** `register_ops_once()` is the Torch bridge that makes the rest of vLLM see AITER implementations as first-class custom ops. Each registration spells out the op name, mutability contract, fake implementation, and sometimes a ROCm dispatch key; the set covers MoE, routing, MLA decode, multiple quantization forms, sparse attention indexing, rotary embedding, and fused allreduce/RMSNorm.
**CN:** `register_ops_once()` 是把 AITER 实现接入 Torch 自定义算子体系的核心桥梁，使 vLLM 其它部分可以像调用普通 `torch.ops` 一样调用它们。每个注册项都显式声明了算子名、参数可变性、fake 实现以及在需要时的 ROCm dispatch key；覆盖范围包括 MoE、路由、MLA decode、多种量化、稀疏注意力索引、rotary embedding 和融合 allreduce/RMSNorm。

### MHC wrapper utilities
```python
    @staticmethod
    def mhc_pre(
        residual: torch.Tensor,
        fn: torch.Tensor,
        hc_scale: torch.Tensor,
        hc_base: torch.Tensor,
        rms_eps: float,
        hc_pre_eps: float,
        hc_sinkhorn_eps: float,
        hc_post_mult_value: float,
        sinkhorn_repeat: int,
    ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
        """
        Forward pass for mHC pre block.

        Args:
            residual: shape (..., hc_mult, hidden_size), dtype torch.bfloat16
            fn: shape (hc_mult3, hc_mult * hidden_size), dtype torch.float32
            hc_scale: shape (3,), dtype torch.float32
            hc_base: shape (hc_mult3,), dtype torch.float32
            rms_eps: RMS normalization epsilon
            hc_pre_eps: pre-mix epsilon
            hc_sinkhorn_eps: sinkhorn epsilon
            hc_post_mult_value: post-mix multiplier value
            sinkhorn_repeat: number of sinkhorn iterations
            n_splits: split-k factor;

        Returns:
            post_mix: shape (..., hc_mult), dtype torch.float32
            comb_mix: shape (..., hc_mult, hc_mult), dtype torch.float32
            layer_input: shape (..., hidden_size), dtype torch.bfloat16
        """
        from aiter.ops.mhc import mhc_pre

        # Validate shapes
        assert residual.dtype == torch.bfloat16
        assert fn.dtype == torch.float32
        assert hc_scale.dtype == torch.float32
        assert hc_base.dtype == torch.float32

        hc_mult = residual.shape[-2]
        hidden_size = residual.shape[-1]
        hc_mult2 = hc_mult * hc_mult
        hc_mult3 = hc_mult * 2 + hc_mult2

        hc_hidden_size = hc_mult * hidden_size
        assert fn.shape[0] == hc_mult3
        assert fn.shape[1] == hc_hidden_size
        assert hc_scale.shape == (3,)
        assert hc_base.shape == (hc_mult3,)

        outer_shape = residual.shape[:-2]

        residual_flat = residual.view(-1, hc_mult, hidden_size)

        num_tokens = residual_flat.shape[0]
        if num_tokens == 0:
            return (
                torch.empty(
                    num_tokens,
                    hc_mult,
                    1,
                    dtype=torch.float32,
                    device=residual_flat.device,
                ),
                torch.empty(
                    num_tokens,
                    hc_mult,
                    hc_mult,
                    dtype=torch.float32,
                    device=residual_flat.device,
                ),
                torch.empty(
                    num_tokens,
                    hidden_size,
                    dtype=torch.bfloat16,
                    device=residual_flat.device,
                ),
            )

        # AITER's Python wrapper allocates intermediate/output tensors without
        # explicit device arguments, so run it under the residual tensor's device.
        with torch.device(residual_flat.device):
            post_mix, comb_mix, layer_input = mhc_pre(
                residual_flat,
                fn,
                hc_scale,
                hc_base,
                rms_eps,
                hc_pre_eps,
                hc_sinkhorn_eps,
                hc_post_mult_value,
                sinkhorn_repeat,
            )
        return (
            post_mix.view(*outer_shape, hc_mult, 1),
            comb_mix.view(*outer_shape, hc_mult, hc_mult),
            layer_input.view(*outer_shape, hidden_size),
        )

    @staticmethod
    def hc_head(
        hs_flat: torch.Tensor,
        fn: torch.Tensor,
        hc_scale: torch.Tensor,
        hc_base: torch.Tensor,
        out: torch.Tensor,
        hidden_size: int,
        rms_eps: float,
        hc_eps: float,
        hc_mult: int,
    ) -> None:
        """Run hc_head through AITER mhc_pre and write the result to out."""
        assert hs_flat.dtype == torch.bfloat16
        assert fn.dtype == torch.float32
        assert hc_scale.dtype == torch.float32
        assert hc_base.dtype == torch.float32
        assert hs_flat.shape[-2:] == (hc_mult, hidden_size)
        assert fn.shape == (hc_mult, hc_mult * hidden_size)
        assert hc_scale.shape == (1,)
        assert hc_base.shape == (hc_mult,)

        num_tokens = hs_flat.shape[0]
        if num_tokens == 0:
            return

        hc_mult3 = hc_mult * 2 + hc_mult * hc_mult

        full_fn = torch.zeros(
            hc_mult3,
            hc_mult * hidden_size,
            dtype=fn.dtype,
            device=fn.device,
        )
        full_fn[:hc_mult] = fn

        full_base = torch.zeros(hc_mult3, dtype=hc_base.dtype, device=hc_base.device)
        full_base[:hc_mult] = hc_base

        full_scale = torch.zeros(3, dtype=hc_scale.dtype, device=hc_scale.device)
        full_scale[0] = hc_scale[0]

        _, _, layer_input = rocm_aiter_ops.mhc_pre(
            hs_flat,
            full_fn,
            full_scale,
            full_base,
            rms_eps,
            hc_eps,
            0.0,
            1.0,
            0,
        )
        out.copy_(layer_input)

    @staticmethod
    def mhc_post(
        x: torch.Tensor,
        residual: torch.Tensor,
        post_layer_mix: torch.Tensor,
        comb_res_mix: torch.Tensor,
    ) -> torch.Tensor:
        from aiter.ops.mhc import mhc_post

        hc_mult = residual.shape[-2]
        hidden_size = residual.shape[-1]
        residual_flat = residual.view(-1, hc_mult, hidden_size)
        num_tokens = residual_flat.shape[0]
        out = torch.empty_like(residual_flat)
        mhc_post(
            out,
            x.view(num_tokens, hidden_size),
            residual_flat,
            post_layer_mix.view(num_tokens, hc_mult, 1),
            comb_res_mix.view(num_tokens, hc_mult, hc_mult),
        )
        return out.view_as(residual)
```
**EN:** These are the higher-level wrappers that the model code actually consumes for mHC-style blocks. `mhc_pre()` validates BF16/FP32 layouts, flattens outer dimensions, and works around AITER's device-allocation behavior by temporarily entering the residual tensor's device context; `hc_head()` repacks a smaller head-only formulation into the larger `mhc_pre()` parameterization; `mhc_post()` delegates the residual recombination step back to AITER while restoring the original outer shape.
**CN:** 这些是模型代码真正会调用的 mHC 高层包装接口。`mhc_pre()` 会校验 BF16/FP32 布局、展平外层维度，并通过暂时切换到 residual 所在设备来规避 AITER Python 包装器的设备分配习惯；`hc_head()` 把更小的 head-only 公式重新打包成 `mhc_pre()` 所需的大参数化形式；`mhc_post()` 则把残差重组步骤交回 AITER，并最终恢复原始外层形状。

## Key Concepts / 关键概念
- EN: `IS_AITER_FOUND` and `FP8_DTYPE` are cached at import time so optional-backend discovery does not leak host-side checks into compiled graphs.
- CN: `IS_AITER_FOUND` 与 `FP8_DTYPE` 在导入时就被缓存，从而避免把可选后端探测和主机侧检查带进编译图。
- EN: Most real kernels have matching fake implementations so `torch.compile` can keep shape/dtype propagation even when ROCm AITER is absent.
- CN: 大多数真实内核都配有对应的 fake 实现，因此即使没有 ROCm AITER，`torch.compile` 仍然可以完成形状和 dtype 传播。
- EN: `rocm_aiter_ops` separates hardware feasibility, env-policy toggles, and explicit user backend selection into different layers of logic.
- CN: `rocm_aiter_ops` 把硬件可行性、环境变量策略开关和用户显式后端选择拆成了不同层次的判断逻辑。
- EN: The module is both a low-level custom-op registrar and a high-level façade that normalizes AITER API drift for the rest of vLLM.
- CN: 这个模块既是底层自定义算子注册器，也是一个高层外观层，用来为 vLLM 其余部分屏蔽 AITER API 演进带来的差异。

## Dependencies / 依赖关系
- EN: Depends on `torch`, `torch.ops`, `direct_register_custom_op`, and ROCm platform helpers from `vllm.platforms`.
- CN: 依赖 `torch`、`torch.ops`、`direct_register_custom_op`，以及来自 `vllm.platforms` 的 ROCm 平台辅助函数。
- EN: Optionally depends on external `aiter` Python packages/modules (`aiter.mla`, `aiter.ops.triton.*`, `aiter.dist.*`) and on `pandas` for tuned-config CSV parsing.
- CN: 可选依赖外部 `aiter` Python 包/模块（如 `aiter.mla`、`aiter.ops.triton.*`、`aiter.dist.*`），以及用于读取调优 CSV 的 `pandas`。
- EN: Interacts with vLLM attention utilities such as `rocm_aiter_sparse_attn_indexer` and mHC helpers that expect specific BF16/FP32 layouts.
- CN: 会与 vLLM 的注意力辅助模块交互，例如 `rocm_aiter_sparse_attn_indexer` 以及要求特定 BF16/FP32 布局的 mHC 辅助逻辑。
