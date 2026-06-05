# tokenspeed_mla.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/mla/tokenspeed_mla.py`
- **Repository**: vllm-project/vllm
- **Purpose**: TokenSpeed CuTe DSL MLA decode backend (Blackwell, FP8 KV cache only). / 该模块位于 `attention/backends/mla` 子系统，主要围绕 `_get_workspace`, `TokenspeedMLAMetadataBuilder`, `TokenspeedMLABackend` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""TokenSpeed CuTe DSL MLA decode backend (Blackwell, FP8 KV cache only)."""

from typing import ClassVar

import torch

from vllm.config.cache import CacheDType
from vllm.logger import init_logger
from vllm.model_executor.layers.attention.mla_attention import (
    MLACommonBackend,
    MLACommonImpl,
    MLACommonMetadata,
    MLACommonMetadataBuilder,
    QueryLenSupport,
)
from vllm.platforms.interface import DeviceCapability
from vllm.utils.torch_utils import is_quantized_kv_cache
from vllm.v1.attention.backend import (
    AttentionCGSupport,
    AttentionLayer,
    AttentionType,
    MultipleOf,
)
from vllm.v1.attention.backends.utils import KVCacheLayoutType

logger = init_logger(__name__)

# Workspace upper bound for tokenspeed_mla_decode (per-device, lazy):
#   num_sms * num_heads * MAX_Q_LEN * (kv_lora_rank + 1) * sizeof(float32)
# Matches the kernel's `get_workspace_size` formula. MAX_Q_LEN=8 covers up to
# EAGLE3 / MTP-2 spec decoding query lengths; larger q_len fails the kernel's
# own buffer check.
_TOKENSPEED_MAX_Q_LEN = 8

_g_workspace: dict[torch.device, torch.Tensor] = {}
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `_TOKENSPEED_MAX_Q_LEN`, `_g_workspace`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `_TOKENSPEED_MAX_Q_LEN`, `_g_workspace`。

### `_get_workspace` function / `_get_workspace` 函数
```python
def _get_workspace(
    device: torch.device, num_heads: int, kv_lora_rank: int
) -> torch.Tensor:
    from tokenspeed_mla import get_num_sm

    needed = (
        get_num_sm(device) * num_heads * _TOKENSPEED_MAX_Q_LEN * (kv_lora_rank + 1) * 4
    )
    existing = _g_workspace.get(device)
    if existing is None or existing.numel() < needed:
        _g_workspace[device] = torch.empty(needed, dtype=torch.int8, device=device)
    return _g_workspace[device]
```
**EN:** This function implements `_get_workspace` within the module. Key calls include `get`, `empty`, `numel`, `get_num_sm`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_get_workspace`，其作用域位于the module。 关键调用包括 `get`, `empty`, `numel`, `get_num_sm`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TokenspeedMLAMetadataBuilder` class / `TokenspeedMLAMetadataBuilder` 类
```python
class TokenspeedMLAMetadataBuilder(MLACommonMetadataBuilder[MLACommonMetadata]):
    _cudagraph_support: ClassVar[AttentionCGSupport] = AttentionCGSupport.UNIFORM_BATCH
    query_len_support: ClassVar[QueryLenSupport] = QueryLenSupport.UNIFORM
```
**EN:** Introduces the `TokenspeedMLAMetadataBuilder` class on top of `MLACommonMetadataBuilder[MLACommonMetadata]`. Core methods include its methods defined below.
**CN:** 这里定义 `TokenspeedMLAMetadataBuilder` 类，其基类包括 `MLACommonMetadataBuilder[MLACommonMetadata]`。核心方法包括 下方定义的方法。

### `TokenspeedMLABackend` class / `TokenspeedMLABackend` 类
```python
class TokenspeedMLABackend(MLACommonBackend):
    supported_dtypes: ClassVar[list[torch.dtype]] = [torch.float16, torch.bfloat16]
    supported_kv_cache_dtypes: ClassVar[list[CacheDType]] = [
        "fp8",
        "fp8_e4m3",
    ]
```
**EN:** Introduces the `TokenspeedMLABackend` class on top of `MLACommonBackend`. Core methods include `get_supported_kernel_block_sizes`, `get_name`, `get_impl_cls`, `get_builder_cls`, `supports_compute_capability`, `supports_combination`.
**CN:** 这里定义 `TokenspeedMLABackend` 类，其基类包括 `MLACommonBackend`。核心方法包括 `get_supported_kernel_block_sizes`, `get_name`, `get_impl_cls`, `get_builder_cls`, `supports_compute_capability`, `supports_combination`。

### `TokenspeedMLABackend.get_supported_kernel_block_sizes` method / `TokenspeedMLABackend.get_supported_kernel_block_sizes` 方法
```python
    @staticmethod
    def get_supported_kernel_block_sizes() -> list[int | MultipleOf]:
        return [32, 64]
```
**EN:** This method returns or derives a value within `TokenspeedMLABackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`TokenspeedMLABackend`。

### `TokenspeedMLABackend.get_name` method / `TokenspeedMLABackend.get_name` 方法
```python
    @staticmethod
    def get_name() -> str:
        return "TOKENSPEED_MLA"
```
**EN:** This method returns or derives a value within `TokenspeedMLABackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`TokenspeedMLABackend`。

### `TokenspeedMLABackend.get_impl_cls` method / `TokenspeedMLABackend.get_impl_cls` 方法
```python
    @staticmethod
    def get_impl_cls() -> type["TokenspeedMLAImpl"]:
        return TokenspeedMLAImpl
```
**EN:** This method returns or derives a value within `TokenspeedMLABackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`TokenspeedMLABackend`。

### `TokenspeedMLABackend.get_builder_cls` method / `TokenspeedMLABackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["TokenspeedMLAMetadataBuilder"]:
        return TokenspeedMLAMetadataBuilder
```
**EN:** This method returns or derives a value within `TokenspeedMLABackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`TokenspeedMLABackend`。

### `TokenspeedMLABackend.supports_compute_capability` method / `TokenspeedMLABackend.supports_compute_capability` 方法
```python
    @classmethod
    def supports_compute_capability(cls, capability: DeviceCapability) -> bool:
        return capability.major == 10
```
**EN:** This method implements `supports_compute_capability` within `TokenspeedMLABackend`.
**CN:** 该方法会实现 `supports_compute_capability`，其作用域位于`TokenspeedMLABackend`。

### `TokenspeedMLABackend.supports_combination` method / `TokenspeedMLABackend.supports_combination` 方法
```python
    @classmethod
    def supports_combination(
        cls,
        head_size: int,
        dtype: torch.dtype,
        kv_cache_dtype: CacheDType | None,
        block_size: int | None,
        use_mla: bool,
        has_sink: bool,
        use_sparse: bool,
        device_capability: DeviceCapability,
    ) -> str | None:
        # Surface a clear install hint up front rather than letting a raw
        # ModuleNotFoundError fire deep inside `forward_mqa` at first request.
        try:
            import tokenspeed_mla  # noqa: F401
        except ImportError:
            return (
                "tokenspeed_mla package is not installed. "
                "Install it with: `uv pip install tokenspeed-mla`"
            )

        # tokenspeed_mla CuTe DSL kernel is shape-specialized for DeepSeek R1
        # MLA dimensions (qk_nope=128, qk_rope=64, v=128). Reject anything else.
        from vllm.config import get_current_vllm_config

        vllm_config = get_current_vllm_config()
        if vllm_config.model_config is not None:
            hf_text_config = vllm_config.model_config.hf_text_config
            qk_nope_head_dim = getattr(hf_text_config, "qk_nope_head_dim", 0)
            qk_rope_head_dim = getattr(hf_text_config, "qk_rope_head_dim", 0)
            v_head_dim = getattr(hf_text_config, "v_head_dim", 0)
            if qk_nope_head_dim != 128 or qk_rope_head_dim != 64 or v_head_dim != 128:
                return (
                    "tokenspeed_mla requires DeepSeek R1 MLA dimensions "
                    "(qk_nope_head_dim=128, qk_rope_head_dim=64, v_head_dim=128), "
                    f"got ({qk_nope_head_dim}, {qk_rope_head_dim}, {v_head_dim})"
                )
        return None
```
**EN:** This method implements `supports_combination` within `TokenspeedMLABackend`. Key calls include `get_current_vllm_config`, `getattr`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `supports_combination`，其作用域位于`TokenspeedMLABackend`。 关键调用包括 `get_current_vllm_config`, `getattr`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TokenspeedMLABackend.get_required_kv_cache_layout` method / `TokenspeedMLABackend.get_required_kv_cache_layout` 方法
```python
    @classmethod
    def get_required_kv_cache_layout(cls) -> "KVCacheLayoutType | None":
        return "HND"
```
**EN:** This method returns or derives a value within `TokenspeedMLABackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`TokenspeedMLABackend`。

### `TokenspeedMLAImpl` class / `TokenspeedMLAImpl` 类
```python
class TokenspeedMLAImpl(MLACommonImpl[MLACommonMetadata]):
```
**EN:** Introduces the `TokenspeedMLAImpl` class on top of `MLACommonImpl[MLACommonMetadata]`. Core methods include `__init__`, `forward_mqa`.
**CN:** 这里定义 `TokenspeedMLAImpl` 类，其基类包括 `MLACommonImpl[MLACommonMetadata]`。核心方法包括 `__init__`, `forward_mqa`。

### `TokenspeedMLAImpl.__init__` method / `TokenspeedMLAImpl.__init__` 方法
```python
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        scale: float,
        num_kv_heads: int,
        alibi_slopes: list[float] | None,
        sliding_window: int | None,
        kv_cache_dtype: str,
        logits_soft_cap: float | None,
        attn_type: str,
        kv_sharing_target_layer_name: str | None,
        # MLA Specific Arguments
        **mla_args,
    ) -> None:
        super().__init__(
            num_heads,
            head_size,
            scale,
            num_kv_heads,
            alibi_slopes,
            sliding_window,
            kv_cache_dtype,
            logits_soft_cap,
            attn_type,
            kv_sharing_target_layer_name,
            **mla_args,
        )

        unsupported_features = [alibi_slopes, sliding_window, logits_soft_cap]
        if any(unsupported_features):
            raise NotImplementedError(
                "TokenspeedMLAImpl does not support one of the following: "
                "alibi_slopes, sliding_window, logits_soft_cap"
            )

        if attn_type != AttentionType.DECODER:
            raise NotImplementedError(
                "Encoder self-attention and "
                "encoder/decoder cross-attention "
                "are not implemented for "
                "TokenspeedMLAImpl"
            )

        if not is_quantized_kv_cache(self.kv_cache_dtype):
            raise NotImplementedError(
                "TokenspeedMLAImpl requires an FP8 KV cache "
                "(--kv-cache-dtype fp8 or fp8_e4m3); "
                f"got kv_cache_dtype={self.kv_cache_dtype!r}."
            )

        # Allocate (or fetch the cached) workspace lazily on first forward —
        # __init__ runs before the device is necessarily set on the worker;
        # we know it for sure at forward time when we see the input tensor.
        self._workspace_buffer: torch.Tensor | None = None
        self.softmax_scale: float | None = None
        self.output_scale: float | None = None

        # Pre-JIT BF16 and FP8 prefill kernels here too — decode impl always
        # runs when tokenspeed is selected, prefill backend may not (user can
        # pair with flash_attn / trtllm). Idempotent.
        from tokenspeed_mla import warmup_compile_prefill

        for q_dtype in (torch.bfloat16, torch.float8_e4m3fn):
            warmup_compile_prefill(
                q_dtype=q_dtype,
                d_qk=self.qk_nope_head_dim + self.qk_rope_head_dim,
                d_v=self.v_head_dim,
                enable_pdl=False,
            )
```
**EN:** This method initializes the object state within `TokenspeedMLAImpl`. Key calls include `__init__`, `any`, `NotImplementedError`, `is_quantized_kv_cache`, `warmup_compile_prefill`, `super`. It touches state such as `_workspace_buffer`, `softmax_scale`, `output_scale`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`TokenspeedMLAImpl`。 关键调用包括 `__init__`, `any`, `NotImplementedError`, `is_quantized_kv_cache`, `warmup_compile_prefill`, `super`。 它会读写 `_workspace_buffer`, `softmax_scale`, `output_scale` 等状态。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `TokenspeedMLAImpl.forward_mqa` method / `TokenspeedMLAImpl.forward_mqa` 方法
```python
    def forward_mqa(
        self,
        q: torch.Tensor | tuple[torch.Tensor, torch.Tensor],
        kv_c_and_k_pe_cache: torch.Tensor,
        attn_metadata: MLACommonMetadata,
        layer: AttentionLayer,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        from tokenspeed_mla import tokenspeed_mla_decode

        assert kv_c_and_k_pe_cache.numel() > 0
        assert attn_metadata.decode is not None

        if isinstance(q, tuple):
            q_nope, q_pe = q
            q = torch.cat([q_nope, q_pe], dim=-1)

        # supports_quant_query_input=True (set in MLACommonImpl) tells the
        # pipeline to concat+FP8-quantize Q upstream via _decode_concat_quant_fp8_op.
        # The kernel is shape-specialized for FP8 Q + FP8 KV, so anything else
        # here means the upstream quant didn't run and the kernel will produce
        # garbage.
        assert q.dtype == torch.float8_e4m3fn, (
            f"TokenspeedMLAImpl expected FP8 query (supports_quant_query_input=True), "
            f"got {q.dtype}. Pipeline isinstance(q, tuple)={isinstance(q, tuple)}, "
            f"q_scale={layer._q_scale_float}, k_scale={layer._k_scale_float}."
        )

        # tokenspeed_mla_decode expects query shape
        # (num_decodes, q_len_per_request, num_heads, head_dim).
        if attn_metadata.num_decode_tokens % attn_metadata.num_decodes != 0:
            logger.warning_once(
                """TokenspeedMLAImpl got a query of uneven length.
                This usually indicates an issue in batch reordering
                or incorrect setup in dummy_run."""
            )
            q = q.unsqueeze(1)
        else:
            q = q.view(attn_metadata.num_decodes, -1, q.shape[-2], q.shape[-1])

        if self.softmax_scale is None:
            # FP8 KV cache is mandatory for this backend, so q_scale/k_scale
            # always apply. softmax_scale is bmm1; output_scale is bmm2 — both
            # required to recover the correct attention output from the FP8
            # KV cache (V is stored as V_real/k_scale).
            self.softmax_scale = (
                self.scale * layer._q_scale_float * layer._k_scale_float
            )
            self.output_scale = layer._k_scale_float

        if self._workspace_buffer is None:
            self._workspace_buffer = _get_workspace(
                q.device, self.num_heads, self.kv_lora_rank
            )

        # vLLM kv_c_and_k_pe_cache is already (num_blocks, block_size, head_size).
        # tokenspeed_mla_decode wants 3D — pass as-is (no unsqueeze, unlike trtllm).
        o = tokenspeed_mla_decode(
            query=q,
            kv_cache=kv_c_and_k_pe_cache,
            workspace_buffer=self._workspace_buffer,
            kv_lora_rank=self.kv_lora_rank,
            qk_rope_head_dim=self.qk_rope_head_dim,
            block_tables=attn_metadata.decode.block_table,
            seq_lens=attn_metadata.decode.seq_lens,
            max_seq_len=attn_metadata.max_seq_len,
            softmax_scale=self.softmax_scale,
            output_scale=self.output_scale,
            enable_pdl=False,
        )

        # Flatten the output for consistent shape
        o = o.view(-1, o.shape[-2], o.shape[-1])

        # tokenspeed_mla_decode does not return LSE.
        return o, None
```
**EN:** This method drives the forward-pass computation within `TokenspeedMLAImpl`. Key calls include `isinstance`, `tokenspeed_mla_decode`, `view`, `numel`, `cat`, `warning_once`. It touches state such as `softmax_scale`, `output_scale`, `_workspace_buffer`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会驱动前向计算流程，其作用域位于`TokenspeedMLAImpl`。 关键调用包括 `isinstance`, `tokenspeed_mla_decode`, `view`, `numel`, `cat`, `warning_once`。 它会读写 `softmax_scale`, `output_scale`, `_workspace_buffer` 等状态。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_get_workspace`: top-level helper or orchestration entry point. / `_get_workspace`：顶层辅助函数或编排入口。
- `TokenspeedMLAMetadataBuilder`: central class or interface in this module. / `TokenspeedMLAMetadataBuilder`：本模块中的核心类或接口。
- `TokenspeedMLABackend`: central class or interface in this module. / `TokenspeedMLABackend`：本模块中的核心类或接口。
- `TokenspeedMLAImpl`: central class or interface in this module. / `TokenspeedMLAImpl`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `typing`
- External / 外部依赖: `torch`, `tokenspeed_mla`
- Internal vLLM / 内部依赖: `vllm.config.cache`, `vllm.logger`, `vllm.model_executor.layers.attention.mla_attention`, `vllm.platforms.interface`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.utils`, `vllm.config`
