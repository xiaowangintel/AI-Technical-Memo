# tokenspeed_mla_backend.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/tokenspeed_mla_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the tokenspeed mla backend attention backend used by SGLang. It combines runtime checks, metadata handling, and kernel dispatch helpers for the attention path. / 该模块实现 SGLang 使用的 tokenspeed mla backend 注意力后端，组合了注意力路径所需的运行时检查、元数据处理和内核分发辅助逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 21-21: imports
```python
from __future__ import annotations
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 23-29: docstring
```python
"""Attention backend for the tokenspeed-mla CuTe DSL kernels on Blackwell.

Subclasses :class:`TRTLLMMLABackend` and overrides only ``_run_decode_kernel``
and ``_run_prefill_kernel``. All metadata, KV-cache layout, CUDA-graph
plumbing, FP8 quantize/rope, draft-extend padding, and chunked-prefix
dispatch are inherited unchanged from the parent.
"""
```
**EN:** Provides inline documentation that explains the scope of the surrounding module or class.
**CN:** 提供内联文档，用于说明周围模块或类的职责范围。

### Lines 31-42: imports
```python
import logging
from typing import TYPE_CHECKING, Optional

import torch

from sglang.jit_kernel.utils import is_arch_support_pdl
from sglang.srt.layers.attention.trtllm_mla_backend import (
    TRTLLMMLABackend,
    TRTLLMMLAMultiStepDraftBackend,
    _quantize_fp8_qkv,
)
from sglang.srt.utils import is_tokenspeed_mla_available
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 44-45: conditional branch
```python
if is_tokenspeed_mla_available():
    import tokenspeed_mla
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 47-49: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.layers.radix_attention import RadixAttention
    from sglang.srt.model_executor.model_runner import ModelRunner
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 51-59: module constants
```python
logger = logging.getLogger(__name__)


# Workspace upper bound for tokenspeed_mla_decode:
#   num_sms * num_heads * max_q_len * (kv_lora_rank + 1) * sizeof(float32)
# MAX_Q_LEN=8 covers EAGLE3 num_draft_tokens=4 plus headroom.
_TOKENSPEED_MAX_Q_LEN = 8

_g_tokenspeed_workspace: dict[torch.device, torch.Tensor] = {}
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 62-77: function _get_tokenspeed_workspace
```python
def _get_tokenspeed_workspace(
    device: torch.device, num_heads: int, kv_lora_rank: int
) -> torch.Tensor:
    needed = (
        tokenspeed_mla.get_num_sm(device)
        * num_heads
        * _TOKENSPEED_MAX_Q_LEN
        * (kv_lora_rank + 1)
        * 4
    )
    existing = _g_tokenspeed_workspace.get(device)
    if existing is None or existing.numel() < needed:
        _g_tokenspeed_workspace[device] = torch.empty(
            needed, dtype=torch.int8, device=device
        )
    return _g_tokenspeed_workspace[device]
```
**EN:** Implements the get tokenspeed workspace routine used by this attention module.
**CN:** 实现该注意力模块使用的 get tokenspeed workspace 例程。

### Lines 80-82: class TokenspeedMLABackend
```python
class TokenspeedMLABackend(TRTLLMMLABackend):
    """tokenspeed-mla CuTe DSL attention backend (Blackwell SM100, FP8 KV)."""
```
**EN:** Concrete attention backend that connects tokenspeed mlabackend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 tokenspeed mlabackend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 83-143: method TokenspeedMLABackend.__init__
```python
    def __init__(
        self,
        model_runner: "ModelRunner",
        skip_prefill: bool = False,
        kv_indptr_buf: Optional[torch.Tensor] = None,
        q_indptr_decode_buf: Optional[torch.Tensor] = None,
    ):
        super().__init__(
            model_runner,
            skip_prefill,
            kv_indptr_buf,
            q_indptr_decode_buf,
        )

        if self.data_type != torch.float8_e4m3fn:
            raise ValueError(
                "tokenspeed_mla backend requires --kv-cache-dtype fp8_e4m3, "
                f"got data_type={self.data_type}."
            )
        if self.page_size not in (32, 64):
            raise ValueError(
                "tokenspeed_mla backend requires page_size in {32, 64}, "
                f"got page_size={self.page_size}."
            )

        self._tokenspeed_workspace: Optional[torch.Tensor] = None

        # Pre-JIT the prefill kernel variants. Each cute.compile takes 1-2 min;
        # without warm-up the first request trips the 300 s scheduler watchdog.
        if is_tokenspeed_mla_available():
            _compile_prefill_kernel = tokenspeed_mla.mla_prefill._compile_prefill_kernel
            _compiled_kernels = tokenspeed_mla.mla_prefill._compiled_kernels
# ... omitted 17 lines ...
                    )
                    if config in _compiled_kernels:
                        continue
                    _compiled_kernels[config] = _compile_prefill_kernel(
                        torch.bfloat16,
                        head_dim_qk,
                        self.v_head_dim,
                        is_causal,
                        return_lse,
                        use_pdl=use_pdl,
                        enable_ex2_emulation=enable_ex2_emulation,
                    )
```
**EN:** Initializes the TokenspeedMLABackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 TokenspeedMLABackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 145-153: method TokenspeedMLABackend._ensure_workspace
```python
    def _ensure_workspace(self, device: torch.device) -> torch.Tensor:
        if (
            self._tokenspeed_workspace is None
            or self._tokenspeed_workspace.device != device
        ):
            self._tokenspeed_workspace = _get_tokenspeed_workspace(
                device, self.num_q_heads, self.kv_lora_rank
            )
        return self._tokenspeed_workspace
```
**EN:** Implements the ensure workspace routine used by this attention module.
**CN:** 实现该注意力模块使用的 ensure workspace 例程。

### Lines 155-185: method TokenspeedMLABackend._run_decode_kernel
```python
    def _run_decode_kernel(
        self,
        query: torch.Tensor,
        kv_cache: torch.Tensor,
        block_tables: torch.Tensor,
        seq_lens: torch.Tensor,
        max_seq_len: int,
        layer: "RadixAttention",
    ) -> torch.Tensor:
        k_scale = getattr(layer, "k_scale_float", None)
        if k_scale is None:
            k_scale = 1.0
        softmax_scale = float(layer.scaling) * float(k_scale)
        output_scale = float(k_scale)

        seq_lens_i32 = (
            seq_lens if seq_lens.dtype == torch.int32 else seq_lens.to(torch.int32)
        )
        return tokenspeed_mla.tokenspeed_mla_decode(
            query=query,
            kv_cache=kv_cache,
            workspace_buffer=self._ensure_workspace(query.device),
            kv_lora_rank=self.kv_lora_rank,
            qk_rope_head_dim=self.qk_rope_head_dim,
            block_tables=block_tables,
            seq_lens=seq_lens_i32,
            max_seq_len=int(max_seq_len),
            softmax_scale=softmax_scale,
            output_scale=output_scale,
            enable_pdl=is_arch_support_pdl(),
        )
```
**EN:** Implements the run decode kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 run decode kernel 例程。

### Lines 187-229: method TokenspeedMLABackend._run_prefill_kernel
```python
    def _run_prefill_kernel(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: "RadixAttention",
        batch_size: int,
        cum_seq_lens_q: torch.Tensor,
        max_q_len: int,
        seq_lens_kv: torch.Tensor,
        cum_seq_lens_kv: torch.Tensor,
        max_kv_len: int,
        is_causal: bool,
        return_lse: bool,
        out_buffer: torch.Tensor,
        o_sf_scale: float = 1.0,
    ):
        # Quantize to FP8 for the Blackwell FP8 GEMM speedup (mirrors trtllm-gen).
        # The kernel has no per-tensor scale knob for either K or V, so we
        # require both ``k_scale_float`` and ``v_scale_float`` to be 1.0.
        if self.data_type == torch.float8_e4m3fn:
            q, k, v, k_scale, v_scale = _quantize_fp8_qkv(q, k, v, layer)
            assert k_scale == 1.0 and v_scale == 1.0, (
                "tokenspeed_mla prefill kernel has no per-tensor K/V scale "
                "knob; both k_scale_float and v_scale_float must be 1.0, got "
                f"k_scale={k_scale}, v_scale={v_scale}."
            )

        return tokenspeed_mla.tokenspeed_mla_prefill(
            query=q,
            key=k,
            value=v,
            seq_lens=seq_lens_kv,
            cum_seq_lens=cum_seq_lens_kv,
            max_seq_len=int(max_kv_len),
            batch_size=int(batch_size),
            softmax_scale=float(layer.scaling),
            is_causal=is_causal,
            return_lse=return_lse,
            cum_seq_lens_q=cum_seq_lens_q,
            max_seq_len_q=int(max_q_len),
            enable_pdl=is_arch_support_pdl(),
        )
```
**EN:** Implements the run prefill kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 run prefill kernel 例程。

### Lines 232-234: class TokenspeedMLAMultiStepDraftBackend
```python
class TokenspeedMLAMultiStepDraftBackend(TRTLLMMLAMultiStepDraftBackend):
    """Multi-step draft backend for tokenspeed_mla used by EAGLE."""
```
**EN:** Concrete attention backend that connects tokenspeed mlamulti step draft backend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 tokenspeed mlamulti step draft backend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 235-247: method TokenspeedMLAMultiStepDraftBackend.__init__
```python
    def __init__(
        self, model_runner: "ModelRunner", topk: int, speculative_num_steps: int
    ):
        super().__init__(model_runner, topk, speculative_num_steps)
        # Parent populates self.attn_backends with TRT-LLM instances; replace
        # them with tokenspeed instances sharing the parent's index buffers.
        for i in range(self.speculative_num_steps - 1):
            self.attn_backends[i] = TokenspeedMLABackend(
                model_runner,
                skip_prefill=True,
                kv_indptr_buf=self.kv_indptr[i],
                q_indptr_decode_buf=self.q_indptr_decode,
            )
```
**EN:** Initializes the TokenspeedMLAMultiStepDraftBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 TokenspeedMLAMultiStepDraftBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** Execution metadata planning / **CN:** 执行元数据规划
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Paged attention layouts / **CN:** 分页注意力布局
- **EN:** Quantized cache and layout handling / **CN:** 量化缓存与布局处理

## Dependencies / 依赖关系
- `__future__.annotations`
- `logging`
- `typing.TYPE_CHECKING`
- `typing.Optional`
- `torch`
- `sglang.jit_kernel.utils.is_arch_support_pdl`
- `sglang.srt.layers.attention.trtllm_mla_backend.TRTLLMMLABackend`
- `sglang.srt.layers.attention.trtllm_mla_backend.TRTLLMMLAMultiStepDraftBackend`
- `sglang.srt.layers.attention.trtllm_mla_backend._quantize_fp8_qkv`
- `sglang.srt.utils.is_tokenspeed_mla_available`
- `tokenspeed_mla`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.model_executor.model_runner.ModelRunner`
