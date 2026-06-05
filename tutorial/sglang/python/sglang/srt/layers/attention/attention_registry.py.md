# attention_registry.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/attention_registry.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module maintains the registry of available attention backends and the factory helpers that create them for runtime dispatch. / 该模块维护可用注意力后端的注册表，以及供运行时分发使用的后端工厂辅助函数。
## Line-by-Line Analysis / 逐行分析
### Lines 1-8: imports
```python
import logging
from typing import TYPE_CHECKING

from sglang.srt.configs.linear_attn_model_registry import (
    get_linear_attn_config,
    import_backend_class,
)
from sglang.srt.utils import get_device_capability, is_musa
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 10-12: module constants
```python
_is_musa = is_musa()

logger = logging.getLogger(__name__)
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 15-18: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    # evade circular imports
    from sglang.srt.layers.attention.base_attn_backend import AttentionBackend
    from sglang.srt.model_executor.model_runner import ModelRunner
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 20-20: module constants
```python
ATTENTION_BACKENDS = {}
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 23-28: function register_attention_backend
```python
def register_attention_backend(name):
    def decorator(fn):
        ATTENTION_BACKENDS[name] = fn
        return fn

    return decorator
```
**EN:** Registers register attention backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 register attention backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 31-53: function create_flashinfer_backend
```python
@register_attention_backend("flashinfer")
def create_flashinfer_backend(runner):
    import torch

    if not runner.use_mla_backend:
        from sglang.srt.layers.attention.flashinfer_backend import FlashInferAttnBackend

        # Init streams
        if runner.server_args.speculative_algorithm == "EAGLE":
            if (
                not hasattr(runner, "plan_stream_for_flashinfer")
                or not runner.plan_stream_for_flashinfer
            ):
                runner.plan_stream_for_flashinfer = torch.cuda.Stream()
        return FlashInferAttnBackend(
            runner, init_new_workspace=runner.init_new_workspace
        )
    else:
        from sglang.srt.layers.attention.flashinfer_mla_backend import (
            FlashInferMLAAttnBackend,
        )

        return FlashInferMLAAttnBackend(runner)
```
**EN:** Registers create flashinfer backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create flashinfer backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 56-62: function create_trtllm_mla_backend
```python
@register_attention_backend("trtllm_mla")
def create_trtllm_mla_backend(runner):
    if not runner.use_mla_backend:
        raise ValueError("trtllm_mla backend can only be used with MLA models.")
    from sglang.srt.layers.attention.trtllm_mla_backend import TRTLLMMLABackend

    return TRTLLMMLABackend(runner)
```
**EN:** Registers create trtllm mla backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create trtllm mla backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 65-73: function create_tokenspeed_mla_backend
```python
@register_attention_backend("tokenspeed_mla")
def create_tokenspeed_mla_backend(runner):
    if not runner.use_mla_backend:
        raise ValueError("tokenspeed_mla backend can only be used with MLA models.")
    from sglang.srt.layers.attention.tokenspeed_mla_backend import (
        TokenspeedMLABackend,
    )

    return TokenspeedMLABackend(runner)
```
**EN:** Registers create tokenspeed mla backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create tokenspeed mla backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 76-80: function create_aiter_backend
```python
@register_attention_backend("aiter")
def create_aiter_backend(runner):
    from sglang.srt.layers.attention.aiter_backend import AiterAttnBackend

    return AiterAttnBackend(runner)
```
**EN:** Registers create aiter backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create aiter backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 83-87: function create_wave_backend
```python
@register_attention_backend("wave")
def create_wave_backend(runner):
    from sglang.srt.layers.attention.wave_backend import WaveAttnBackend

    return WaveAttnBackend(runner)
```
**EN:** Registers create wave backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create wave backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 90-96: function create_ascend_backend
```python
@register_attention_backend("ascend")
def create_ascend_backend(runner):
    from sglang.srt.hardware_backend.npu.attention.ascend_backend import (
        AscendAttnBackend,
    )

    return AscendAttnBackend(runner)
```
**EN:** Registers create ascend backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create ascend backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 99-103: function create_nsa_backend
```python
@register_attention_backend("nsa")
def create_nsa_backend(runner):
    from sglang.srt.layers.attention.nsa_backend import NativeSparseAttnBackend

    return NativeSparseAttnBackend(runner)
```
**EN:** Registers create nsa backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create nsa backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 106-112: function create_dsv4_backend
```python
@register_attention_backend("dsv4")
def create_dsv4_backend(runner):
    from sglang.srt.layers.attention.deepseek_v4_backend import (
        DeepseekV4AttnBackend,
    )

    return DeepseekV4AttnBackend(runner)
```
**EN:** Registers create dsv4 backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create dsv4 backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 115-123: function create_triton_backend
```python
@register_attention_backend("triton")
def create_triton_backend(runner):
    assert not runner.model_config.is_encoder_decoder, (
        "Cross attention is not supported in the triton attention backend. "
        "Please use `--attention-backend flashinfer`."
    )
    from sglang.srt.layers.attention.triton_backend import TritonAttnBackend

    return TritonAttnBackend(runner)
```
**EN:** Registers create triton backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create triton backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 126-130: function create_torch_native_backend
```python
@register_attention_backend("torch_native")
def create_torch_native_backend(runner):
    from sglang.srt.layers.attention.torch_native_backend import TorchNativeAttnBackend

    return TorchNativeAttnBackend(runner)
```
**EN:** Registers create torch native backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create torch native backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 133-137: function create_flex_attention_backend
```python
@register_attention_backend("flex_attention")
def create_flex_attention_backend(runner):
    from sglang.srt.layers.attention.torch_flex_backend import TorchFlexAttnBackend

    return TorchFlexAttnBackend(runner)
```
**EN:** Registers create flex attention backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create flex attention backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 140-144: function create_flashmla_backend
```python
@register_attention_backend("flashmla")
def create_flashmla_backend(runner):
    from sglang.srt.layers.attention.flashmla_backend import FlashMLABackend

    return FlashMLABackend(runner)
```
**EN:** Registers create flashmla backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create flashmla backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 147-170: function create_flashattention_v3_backend
```python
@register_attention_backend("fa3")
def create_flashattention_v3_backend(runner):

    major, minor = get_device_capability()
    if not _is_musa:
        assert (major == 8 and not runner.use_mla_backend) or major == 9, (
            "FlashAttention v3 Backend requires SM>=80 and SM<=90. "
            "Please use `--attention-backend flashinfer`."
        )
        from sglang.srt.layers.attention.flashattention_backend import (
            FlashAttentionBackend,
        )

        return FlashAttentionBackend(runner)
    else:
        assert major == 3 and minor >= 1, (
            "FlashAttention v3 Backend requires MP>=31. "
            "Please use `--attention-backend triton`."
        )
        from sglang.srt.hardware_backend.musa.attention import (
            MusaFlashAttentionBackend,
        )

        return MusaFlashAttentionBackend(runner)
```
**EN:** Registers create flashattention v3 backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create flashattention v3 backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 173-177: function create_flashattention_v4_backend
```python
@register_attention_backend("fa4")
def create_flashattention_v4_backend(runner):
    from sglang.srt.layers.attention.flashattention_backend import FlashAttentionBackend

    return FlashAttentionBackend(runner, fa_impl_ver=4)
```
**EN:** Registers create flashattention v4 backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create flashattention v4 backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 180-184: function create_cutlass_mla_backend
```python
@register_attention_backend("cutlass_mla")
def create_cutlass_mla_backend(runner):
    from sglang.srt.layers.attention.cutlass_mla_backend import CutlassMLABackend

    return CutlassMLABackend(runner)
```
**EN:** Registers create cutlass mla backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create cutlass mla backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 187-193: function create_trtllm_mha_backend
```python
@register_attention_backend("trtllm_mha")
def create_trtllm_mha_backend(runner):
    if runner.use_mla_backend:
        raise ValueError("trtllm_mha backend can only be used with non-MLA models.")
    from sglang.srt.layers.attention.trtllm_mha_backend import TRTLLMHAAttnBackend

    return TRTLLMHAAttnBackend(runner)
```
**EN:** Registers create trtllm mha backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create trtllm mha backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 196-200: function create_intel_amx_backend
```python
@register_attention_backend("intel_amx")
def create_intel_amx_backend(runner):
    from sglang.srt.layers.attention.intel_amx_backend import IntelAMXAttnBackend

    return IntelAMXAttnBackend(runner)
```
**EN:** Registers create intel amx backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create intel amx backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 203-209: function create_dual_chunk_flash_attn_backend
```python
@register_attention_backend("dual_chunk_flash_attn")
def create_dual_chunk_flash_attn_backend(runner):
    from sglang.srt.layers.attention.dual_chunk_flashattention_backend import (
        DualChunkFlashAttentionBackend,
    )

    return DualChunkFlashAttentionBackend(runner)
```
**EN:** Registers create dual chunk flash attn backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create dual chunk flash attn backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

### Lines 212-288: function attn_backend_wrapper
```python
def attn_backend_wrapper(runner: "ModelRunner", full_attn_backend: "AttentionBackend"):
    """
    Wrapper for special models like hybrid GDN, so we don't
    need to change the code of the original attention backend.
    """
    assert not (
        runner.hybrid_gdn_config is not None and runner.use_mla_backend
    ), "hybrid_gdn can only be used with non-MLA models."

    if cfg := runner.mambaish_config:
        from sglang.srt.layers.attention.fla.utils import check_environments
        from sglang.srt.layers.attention.linear.kda_backend import KDAAttnBackend
        from sglang.srt.layers.attention.linear.lightning_backend import (
            LightningAttentionBackend,
        )
        from sglang.srt.layers.attention.linear.utils import (
            initialize_linear_attn_config,
        )
        from sglang.srt.utils import is_blackwell, is_npu

        if not is_npu():
            from sglang.srt.layers.attention.hybrid_linear_attn_backend import (
                HybridLinearAttnBackend,
                Mamba2AttnBackend,
            )
            from sglang.srt.layers.attention.linear.gdn_backend import GDNAttnBackend
        else:
            from sglang.srt.hardware_backend.npu.attention.ascend_gdn_backend import (
                AscendGDNAttnBackend as GDNAttnBackend,
            )
            from sglang.srt.hardware_backend.npu.attention.ascend_hybrid_linear_attn_backend import (
                AscendHybridLinearAttnBackend as HybridLinearAttnBackend,
# ... omitted 33 lines ...
            else:
                raise ValueError(
                    "Expected hybrid GDN or NemotronH models, but got unknown model. "
                    "If this is a custom hybrid model, use register_linear_attn_model() "
                    "from sglang.srt.configs.linear_attn_model_registry."
                )
        full_attn_layers = cfg.full_attention_layer_ids
        return HybridLinearAttnBackend(
            full_attn_backend, linear_attn_backend, full_attn_layers
        )

    return full_attn_backend
```
**EN:** Implements the attn backend wrapper routine used by this attention module.
**CN:** 实现该注意力模块使用的 attn backend wrapper 例程。

### Lines 291-295: function create_intel_xpu_backend
```python
@register_attention_backend("intel_xpu")
def create_intel_xpu_backend(runner):
    from sglang.srt.layers.attention.xpu_backend import XPUAttentionBackend

    return XPUAttentionBackend(runner)
```
**EN:** Registers create intel xpu backend in a dispatch table or plugin registry so the runtime can discover it by name.
**CN:** 将 create intel xpu backend 注册到分发表或插件注册表中，使运行时可以按名称发现它。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** FlashInfer execution path / **CN:** FlashInfer 执行路径
- **EN:** Flash-style fused attention kernels / **CN:** Flash 风格融合注意力内核
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Speculative decoding support / **CN:** 投机解码支持

## Dependencies / 依赖关系
- `logging`
- `typing.TYPE_CHECKING`
- `sglang.srt.configs.linear_attn_model_registry.get_linear_attn_config`
- `sglang.srt.configs.linear_attn_model_registry.import_backend_class`
- `sglang.srt.utils.get_device_capability`
- `sglang.srt.utils.is_musa`
- `sglang.srt.layers.attention.base_attn_backend.AttentionBackend`
- `sglang.srt.model_executor.model_runner.ModelRunner`
- `torch`
- `sglang.srt.layers.attention.trtllm_mla_backend.TRTLLMMLABackend`
- `sglang.srt.layers.attention.tokenspeed_mla_backend.TokenspeedMLABackend`
- `sglang.srt.layers.attention.aiter_backend.AiterAttnBackend`
- `sglang.srt.layers.attention.wave_backend.WaveAttnBackend`
- `sglang.srt.hardware_backend.npu.attention.ascend_backend.AscendAttnBackend`
- `sglang.srt.layers.attention.nsa_backend.NativeSparseAttnBackend`
- `sglang.srt.layers.attention.deepseek_v4_backend.DeepseekV4AttnBackend`
- `sglang.srt.layers.attention.triton_backend.TritonAttnBackend`
- `sglang.srt.layers.attention.torch_native_backend.TorchNativeAttnBackend`
- `sglang.srt.layers.attention.torch_flex_backend.TorchFlexAttnBackend`
- `sglang.srt.layers.attention.flashmla_backend.FlashMLABackend`
- `sglang.srt.layers.attention.flashattention_backend.FlashAttentionBackend`
- `sglang.srt.layers.attention.cutlass_mla_backend.CutlassMLABackend`
- `sglang.srt.layers.attention.trtllm_mha_backend.TRTLLMHAAttnBackend`
- `sglang.srt.layers.attention.intel_amx_backend.IntelAMXAttnBackend`
- `sglang.srt.layers.attention.dual_chunk_flashattention_backend.DualChunkFlashAttentionBackend`
- `sglang.srt.layers.attention.xpu_backend.XPUAttentionBackend`
- `sglang.srt.layers.attention.flashinfer_backend.FlashInferAttnBackend`
- `sglang.srt.layers.attention.flashinfer_mla_backend.FlashInferMLAAttnBackend`
- `sglang.srt.hardware_backend.musa.attention.MusaFlashAttentionBackend`
- `sglang.srt.layers.attention.fla.utils.check_environments`
- `sglang.srt.layers.attention.linear.kda_backend.KDAAttnBackend`
- `sglang.srt.layers.attention.linear.lightning_backend.LightningAttentionBackend`
- `sglang.srt.layers.attention.linear.utils.initialize_linear_attn_config`
- `sglang.srt.utils.is_blackwell`
- `sglang.srt.utils.is_npu`
- `sglang.srt.layers.attention.hybrid_linear_attn_backend.HybridLinearAttnBackend`
- `sglang.srt.layers.attention.hybrid_linear_attn_backend.Mamba2AttnBackend`
- `sglang.srt.layers.attention.linear.gdn_backend.GDNAttnBackend`
- `sglang.srt.hardware_backend.npu.attention.ascend_gdn_backend.AscendGDNAttnBackend`
- `sglang.srt.hardware_backend.npu.attention.ascend_hybrid_linear_attn_backend.AscendHybridLinearAttnBackend`
- `sglang.srt.hardware_backend.npu.attention.ascend_hybrid_linear_attn_backend.AscendMamba2AttnBackend`
