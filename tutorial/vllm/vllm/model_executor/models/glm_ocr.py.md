# glm_ocr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/glm_ocr.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Multimodal model integration for glm_ocr, including encoder/decoder glue and vLLM runtime adaptation. / 面向 glm_ocr 的多模态模型集成模块，包含编解码胶水层与 vLLM 运行时适配逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 29-80)
```python
from collections.abc import Callable
from functools import partial
from typing import TYPE_CHECKING

import torch
import torch.nn as nn
from einops import rearrange

if TYPE_CHECKING:
    from transformers.models.glm_ocr.configuration_glm_ocr import (
        GlmOcrTextConfig,
        GlmOcrVisionConfig,
    )

from vllm.config import VllmConfig
from vllm.distributed import get_tensor_model_parallel_world_size, parallel_state
from vllm.distributed import utils as dist_utils
from vllm.logger import init_logger
from vllm.model_executor.layers.attention.mm_encoder_attention import (
    MMEncoderAttention,
)
from vllm.model_executor.layers.conv import Conv2dLayer
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.linear import (
    QKVParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.rotary_embedding import get_rope
from vllm.model_executor.layers.rotary_embedding.common import (
    ApplyRotaryEmb,
)
from vllm.model_executor.models.glm4_1v import (
    Glm4vDummyInputsBuilder,
    Glm4vForConditionalGeneration,
    Glm4vMultiModalProcessor,
    Glm4vPatchMerger,
    Glm4vProcessingInfo,
    Glm4vVisionBlock,
    Glm4vVisionMLP,
    Glm4vVisionPatchEmbed,
    Glm4vVisionTransformer,
)
from vllm.multimodal import MULTIMODAL_REGISTRY

from .utils import (
    maybe_prefix,
)
from .vision import (
    get_vit_attn_backend,
    is_vit_use_data_parallel,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `GlmOcrVisionMLP` (lines 85-86)
```python
class GlmOcrVisionMLP(Glm4vVisionMLP):
    pass
```
**EN:** Class `GlmOcrVisionMLP` organizes related behavior for this model family or helper component. It inherits from Glm4vVisionMLP.
**CN:** 类 `GlmOcrVisionMLP` 用于组织该模型族或辅助组件的相关行为。 它继承自 Glm4vVisionMLP。

### Class `GlmOcrVisionAttention` (lines 89-204)
```python
class GlmOcrVisionAttention(nn.Module):
    def __init__(
        self,
        embed_dim: int,
        num_heads: int,
        projection_size: int,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        # Per attention head and per partition values.
        use_data_parallel = is_vit_use_data_parallel()
        self.tp_size = (
            1 if use_data_parallel else get_tensor_model_parallel_world_size()
        )
        self.tp_rank = (
            0 if use_data_parallel else parallel_state.get_tensor_model_parallel_rank()
        )
        self.hidden_size_per_attention_head = dist_utils.divide(
            projection_size, num_heads
        )
        self.num_attention_heads_per_partition = dist_utils.divide(
            num_heads, self.tp_size
        )
```
**EN:** Class `GlmOcrVisionAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, split_qkv, forward.
**CN:** 类 `GlmOcrVisionAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, split_qkv, forward。

### Method `GlmOcrVisionAttention.__init__` (lines 90-145)
```python
    def __init__(
        self,
        embed_dim: int,
        num_heads: int,
        projection_size: int,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        # Per attention head and per partition values.
        use_data_parallel = is_vit_use_data_parallel()
        self.tp_size = (
            1 if use_data_parallel else get_tensor_model_parallel_world_size()
        )
        self.tp_rank = (
            0 if use_data_parallel else parallel_state.get_tensor_model_parallel_rank()
        )
        self.hidden_size_per_attention_head = dist_utils.divide(
            projection_size, num_heads
        )
        self.num_attention_heads_per_partition = dist_utils.divide(
            num_heads, self.tp_size
        )

        self.head_dim = embed_dim // num_heads

        self.q_norm = RMSNorm(self.head_dim, eps=1e-5)
        self.k_norm = RMSNorm(self.head_dim, eps=1e-5)

        self.qkv = QKVParallelLinear(
            hidden_size=embed_dim,
            head_size=self.hidden_size_per_attention_head,
            total_num_heads=num_heads,
            total_num_kv_heads=num_heads,
            bias=True,
            quant_config=quant_config,
            # Change qkv prefix to align with GLM-4.5V-FP8 quantization cfg
            prefix=f"{prefix}.qkv_proj" if quant_config else f"{prefix}.qkv",
            disable_tp=use_data_parallel,
        )
        self.proj = RowParallelLinear(
            input_size=projection_size,
            output_size=embed_dim,
            quant_config=quant_config,
            prefix=f"{prefix}.proj",
            bias=True,
            disable_tp=use_data_parallel,
        )

        self.attn = MMEncoderAttention(
            num_heads=self.num_attention_heads_per_partition,
            head_size=self.hidden_size_per_attention_head,
            scale=self.hidden_size_per_attention_head**-0.5,
            prefix=f"{prefix}.attn",
        )
        self.apply_rotary_emb = ApplyRotaryEmb(enforce_enable=True)
```
**EN:** Method `GlmOcrVisionAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GlmOcrVisionAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GlmOcrVisionAttention.split_qkv` (lines 147-162)
```python
    def split_qkv(self, qkv: torch.Tensor) -> tuple[torch.Tensor, ...]:
        # [s, b, 3 * head * head_dim]
        seq_len, bs, _ = qkv.shape

        # [s, b, 3 * head * head_dim] -> 3 * [s, b, head * head_dim]
        q, k, v = qkv.chunk(3, dim=2)

        # 3 * [s, b, head * head_dim] -> 3 * [s, b, head, head_dim]
        new_shape = (
            seq_len,
            bs,
            self.num_attention_heads_per_partition,
            self.hidden_size_per_attention_head,
        )
        q, k, v = (x.view(*new_shape) for x in (q, k, v))
        return q, k, v
```
**EN:** Method `GlmOcrVisionAttention.split_qkv` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `GlmOcrVisionAttention.split_qkv` 封装了该模块中的一段可复用核心逻辑。

### Method `GlmOcrVisionAttention.forward` (lines 164-204)
```python
    def forward(
        self,
        x: torch.Tensor,
        cu_seqlens: torch.Tensor,
        rotary_pos_emb_cos: torch.Tensor,
        rotary_pos_emb_sin: torch.Tensor,
        max_seqlen: torch.Tensor | None = None,  # Only used for Flash Attention
    ) -> torch.Tensor:
        # [s, b, c] --> [s, b, head * 3 * head_dim]
        x, _ = self.qkv(x)

        # [s, b, 3 * head * head_dim] -> 3 * [s, b, head, head_dim]
        q, k, v = self.split_qkv(x)

        # RMSNorm on q, k
        q_shape, k_shape = q.shape, k.shape
        q = self.q_norm(q.reshape(-1, self.head_dim)).view(q_shape)
        k = self.k_norm(k.reshape(-1, self.head_dim)).view(k_shape)

        q, k, v = (rearrange(x, "s b ... -> b s ...").contiguous() for x in (q, k, v))
        if rotary_pos_emb_cos is not None and rotary_pos_emb_sin is not None:
            # [2 * b, s, heads, head_dim]
            qk_concat = torch.cat([q, k], dim=0)
            qk_rotated = self.apply_rotary_emb(
                qk_concat,
                rotary_pos_emb_cos,
                rotary_pos_emb_sin,
            )
            q, k = torch.chunk(qk_rotated, 2, dim=0)

        context_layer = self.attn(
            query=q,
            key=k,
            value=v,
            cu_seqlens=cu_seqlens,
            max_seqlen=max_seqlen,
        )
        context_layer = rearrange(context_layer, "b s h d -> s b (h d)").contiguous()

        output, _ = self.proj(context_layer)
        return output
```
**EN:** Method `GlmOcrVisionAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GlmOcrVisionAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `GlmOcrVisionBlock` (lines 207-242)
```python
class GlmOcrVisionBlock(Glm4vVisionBlock):
    def __init__(
        self,
        dim: int,
        num_heads: int,
        mlp_hidden_dim: int,
        norm_layer: Callable[[int], nn.Module] | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__(
            dim,
            num_heads,
            mlp_hidden_dim,
            norm_layer,
            quant_config,
            prefix,
        )
        if norm_layer is None:
            norm_layer = partial(nn.LayerNorm, eps=1e-6)
        self.norm1 = norm_layer(dim)
        self.norm2 = norm_layer(dim)
        self.attn = GlmOcrVisionAttention(
            embed_dim=dim,
            num_heads=num_heads,
```
**EN:** Class `GlmOcrVisionBlock` is a structural model block in the vLLM execution graph. It inherits from Glm4vVisionBlock. Key methods include __init__.
**CN:** 类 `GlmOcrVisionBlock` 是 vLLM 执行图中的结构化模型模块。 它继承自 Glm4vVisionBlock。 关键方法包括 __init__。

### Method `GlmOcrVisionBlock.__init__` (lines 208-242)
```python
    def __init__(
        self,
        dim: int,
        num_heads: int,
        mlp_hidden_dim: int,
        norm_layer: Callable[[int], nn.Module] | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__(
            dim,
            num_heads,
            mlp_hidden_dim,
            norm_layer,
            quant_config,
            prefix,
        )
        if norm_layer is None:
            norm_layer = partial(nn.LayerNorm, eps=1e-6)
        self.norm1 = norm_layer(dim)
        self.norm2 = norm_layer(dim)
        self.attn = GlmOcrVisionAttention(
            embed_dim=dim,
            num_heads=num_heads,
            projection_size=dim,
            quant_config=quant_config,
            prefix=f"{prefix}.attn",
        )
        self.mlp = GlmOcrVisionMLP(
            dim,
            mlp_hidden_dim,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
        )
```
**EN:** Method `GlmOcrVisionBlock.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GlmOcrVisionBlock.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Class `GlmOcrVisionPatchEmbed` (lines 245-246)
```python
class GlmOcrVisionPatchEmbed(Glm4vVisionPatchEmbed):
    pass
```
**EN:** Class `GlmOcrVisionPatchEmbed` organizes related behavior for this model family or helper component. It inherits from Glm4vVisionPatchEmbed.
**CN:** 类 `GlmOcrVisionPatchEmbed` 用于组织该模型族或辅助组件的相关行为。 它继承自 Glm4vVisionPatchEmbed。

### Class `GlmOcrPatchMerger` (lines 249-250)
```python
class GlmOcrPatchMerger(Glm4vPatchMerger):
    pass
```
**EN:** Class `GlmOcrPatchMerger` organizes related behavior for this model family or helper component. It inherits from Glm4vPatchMerger.
**CN:** 类 `GlmOcrPatchMerger` 用于组织该模型族或辅助组件的相关行为。 它继承自 Glm4vPatchMerger。

### Class `GlmOcrVisionTransformer` (lines 253-374)
```python
class GlmOcrVisionTransformer(Glm4vVisionTransformer):
    def __init__(
        self,
        text_config: "GlmOcrTextConfig",
        vision_config: "GlmOcrVisionConfig",
        norm_eps: float = 1e-5,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__(text_config, vision_config, norm_eps, quant_config, prefix)

        del self.post_conv_layernorm
        del self.embeddings

        patch_size = vision_config.patch_size
        temporal_patch_size = vision_config.temporal_patch_size
        in_channels = vision_config.in_channels
        depth = vision_config.depth
        self.hidden_size = vision_config.hidden_size
        self.num_heads = vision_config.num_heads

        self.patch_size = vision_config.patch_size
        self.spatial_merge_size = vision_config.spatial_merge_size
        self.out_hidden_size = vision_config.out_hidden_size
```
**EN:** Class `GlmOcrVisionTransformer` organizes related behavior for this model family or helper component. It inherits from Glm4vVisionTransformer. Key methods include __init__, forward.
**CN:** 类 `GlmOcrVisionTransformer` 用于组织该模型族或辅助组件的相关行为。 它继承自 Glm4vVisionTransformer。 关键方法包括 __init__, forward。

### Method `GlmOcrVisionTransformer.__init__` (lines 254-327)
```python
    def __init__(
        self,
        text_config: "GlmOcrTextConfig",
        vision_config: "GlmOcrVisionConfig",
        norm_eps: float = 1e-5,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__(text_config, vision_config, norm_eps, quant_config, prefix)

        del self.post_conv_layernorm
        del self.embeddings

        patch_size = vision_config.patch_size
        temporal_patch_size = vision_config.temporal_patch_size
        in_channels = vision_config.in_channels
        depth = vision_config.depth
        self.hidden_size = vision_config.hidden_size
        self.num_heads = vision_config.num_heads

        self.patch_size = vision_config.patch_size
        self.spatial_merge_size = vision_config.spatial_merge_size
        self.out_hidden_size = vision_config.out_hidden_size

        self.patch_embed = Glm4vVisionPatchEmbed(
            patch_size=patch_size,
            temporal_patch_size=temporal_patch_size,
            in_channels=in_channels,
            hidden_size=self.hidden_size,
        )

        norm_layer = partial(RMSNorm, eps=norm_eps)
        head_dim = self.hidden_size // self.num_heads
        self.rotary_pos_emb = get_rope(
            head_size=head_dim,
            max_position=8192,
            is_neox_style=True,
            rope_parameters={"partial_rotary_factor": 0.5},
        )
        self.blocks = nn.ModuleList(
            [
                GlmOcrVisionBlock(
                    dim=self.hidden_size,
                    num_heads=self.num_heads,
                    mlp_hidden_dim=vision_config.intermediate_size,
                    norm_layer=norm_layer,
                    quant_config=quant_config,
                    prefix=f"{prefix}.blocks.{layer_idx}",
                )
                for layer_idx in range(depth)
            ]
        )
        self.merger = GlmOcrPatchMerger(
            d_model=vision_config.out_hidden_size,
            context_dim=text_config.intermediate_size,
            quant_config=quant_config,
            bias=False,
            prefix=f"{prefix}.merger",
        )

# ... truncated for analysis ...
```
**EN:** Method `GlmOcrVisionTransformer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GlmOcrVisionTransformer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GlmOcrVisionTransformer.forward` (lines 329-374)
```python
    def forward(
        self,
        x: torch.Tensor,
        grid_thw: torch.Tensor | list[list[int]],
    ) -> torch.Tensor:
        if isinstance(grid_thw, list):
            grid_thw = torch.tensor(grid_thw, dtype=torch.int32)

        # patchify
        x = x.to(device=self.device, dtype=self.dtype)
        x = self.patch_embed(x)

        # compute position embedding
        rotary_pos_emb_cos, rotary_pos_emb_sin, image_type_ids = self.rot_pos_emb(
            grid_thw
        )
        # compute cu_seqlens
        cu_seqlens = torch.repeat_interleave(
            grid_thw[:, 1] * grid_thw[:, 2], grid_thw[:, 0]
        ).cumsum(dim=0, dtype=torch.int32)
        cu_seqlens = torch.cat([cu_seqlens.new_zeros(1), cu_seqlens])
        cu_seqlens = cu_seqlens.to(self.device, non_blocking=True)

        # pre-compute max_seqlen for attn mask to reduce cuMemcpy operations
        max_seqlen = self.compute_attn_mask_seqlen(cu_seqlens)

        # transformers
        x = x.unsqueeze(1)
        for blk in self.blocks:
            x = blk(
                x,
                cu_seqlens=cu_seqlens,
                rotary_pos_emb_cos=rotary_pos_emb_cos,
                rotary_pos_emb_sin=rotary_pos_emb_sin,
                max_seqlen=max_seqlen,
            )

        # adapter
        x = self.post_layernorm(x)

        x = x.view(-1, self.spatial_merge_size, self.spatial_merge_size, x.shape[-1])
        x = x.permute(0, 3, 1, 2)
        x = self.downsample(x).view(-1, self.out_hidden_size)
        x = self.merger(x)

        return x
```
**EN:** Method `GlmOcrVisionTransformer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `GlmOcrVisionTransformer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `GlmOcrForConditionalGeneration` (lines 382-395)
```python
@MULTIMODAL_REGISTRY.register_processor(
    Glm4vMultiModalProcessor,
    info=Glm4vProcessingInfo,
    dummy_inputs=Glm4vDummyInputsBuilder,
)
class GlmOcrForConditionalGeneration(Glm4vForConditionalGeneration):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__(vllm_config=vllm_config, prefix=prefix)
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        with self._mark_tower_model(vllm_config, {"image", "video"}):
            self.visual = GlmOcrVisionTransformer(
                config.text_config,
                config.vision_config,
                norm_eps=getattr(config, "rms_norm_eps", 1e-5),
                quant_config=quant_config,
                prefix=maybe_prefix(prefix, "visual"),
            )
```
**EN:** Class `GlmOcrForConditionalGeneration` organizes related behavior for this model family or helper component. It inherits from Glm4vForConditionalGeneration. Key methods include __init__.
**CN:** 类 `GlmOcrForConditionalGeneration` 用于组织该模型族或辅助组件的相关行为。 它继承自 Glm4vForConditionalGeneration。 关键方法包括 __init__。

### Method `GlmOcrForConditionalGeneration.__init__` (lines 383-395)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__(vllm_config=vllm_config, prefix=prefix)
        config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config

        with self._mark_tower_model(vllm_config, {"image", "video"}):
            self.visual = GlmOcrVisionTransformer(
                config.text_config,
                config.vision_config,
                norm_eps=getattr(config, "rms_norm_eps", 1e-5),
                quant_config=quant_config,
                prefix=maybe_prefix(prefix, "visual"),
            )
```
**EN:** Method `GlmOcrForConditionalGeneration.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GlmOcrForConditionalGeneration.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

## Key Concepts / 关键概念
- **Attention / 注意力**
  - **EN:** Attention layers model token-to-token or token-to-feature interactions inside the architecture.
  - **CN:** 注意力层负责建模 token 与 token，或 token 与特征之间的交互。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from collections.abc import Callable`, `from functools import partial`, `from typing import TYPE_CHECKING`, `from einops import rearrange`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`, `from transformers.models.glm_ocr.configuration_glm_ocr import (`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.distributed import get_tensor_model_parallel_world_size, parallel_state`, `from vllm.distributed import utils as dist_utils`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.attention.mm_encoder_attention import (`, `from vllm.model_executor.layers.conv import Conv2dLayer`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.layers.rotary_embedding import get_rope`, `from vllm.model_executor.layers.rotary_embedding.common import (`, `from vllm.model_executor.models.glm4_1v import (`
- **Module note / 模块说明**: **EN:** Inference-only GLM-OCR model compatible with HuggingFace weights. **CN:** 模块文档字符串给出的原始说明是：Inference-only GLM-OCR model compatible with HuggingFace weights.。
