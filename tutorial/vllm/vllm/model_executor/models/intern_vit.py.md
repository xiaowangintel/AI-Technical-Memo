# intern_vit.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/intern_vit.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for intern_vit, including architecture wrappers and weight loading logic. / 面向推理的 intern_vit vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 10-41)
```python
from collections.abc import Iterable
from functools import partial

import torch
import torch.nn as nn
import torch.nn.functional as F
from transformers import PretrainedConfig

from vllm.compilation.decorators import (
    should_torch_compile_mm_encoder,
    support_torch_compile,
)
from vllm.distributed import (
    divide,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    split_tensor_along_last_dim,
    tensor_model_parallel_all_gather,
)
from vllm.model_executor.layers.activation import get_act_fn
from vllm.model_executor.layers.attention import MMEncoderAttention
from vllm.model_executor.layers.conv import Conv2dLayer
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.model_loader.weight_utils import default_weight_loader

from .vision import is_vit_use_data_parallel, run_dp_sharded_vision_model
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 43-46)
```python
NORM2FN = {
    "rms_norm": RMSNorm,
    "layer_norm": nn.LayerNorm,
}
```
**EN:** This block defines NORM2FN, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 NORM2FN，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Class `InternVisionEmbeddings` (lines 49-114)
```python
class InternVisionEmbeddings(nn.Module):
    def __init__(self, config: PretrainedConfig):
        super().__init__()
        self.config = config
        self.embed_dim = config.hidden_size
        self.image_size = config.image_size
        self.patch_size = config.patch_size

        self.class_embedding = nn.Parameter(torch.randn(1, 1, self.embed_dim))

        self.patch_embedding = Conv2dLayer(
            in_channels=3,
            out_channels=self.embed_dim,
            kernel_size=self.patch_size,
            stride=self.patch_size,
        )

        self.num_patches = (self.image_size // self.patch_size) ** 2
        self.num_positions = self.num_patches + 1

        self.position_embedding = nn.Parameter(
            torch.randn(1, self.num_positions, self.embed_dim)
        )

    def _get_pos_embed(self, pos_embed: torch.Tensor, H: int, W: int):
```
**EN:** Class `InternVisionEmbeddings` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, _get_pos_embed, _get_position_embedding, forward.
**CN:** 类 `InternVisionEmbeddings` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, _get_pos_embed, _get_position_embedding, forward。

### Method `InternVisionEmbeddings.__init__` (lines 50-71)
```python
    def __init__(self, config: PretrainedConfig):
        super().__init__()
        self.config = config
        self.embed_dim = config.hidden_size
        self.image_size = config.image_size
        self.patch_size = config.patch_size

        self.class_embedding = nn.Parameter(torch.randn(1, 1, self.embed_dim))

        self.patch_embedding = Conv2dLayer(
            in_channels=3,
            out_channels=self.embed_dim,
            kernel_size=self.patch_size,
            stride=self.patch_size,
        )

        self.num_patches = (self.image_size // self.patch_size) ** 2
        self.num_positions = self.num_patches + 1

        self.position_embedding = nn.Parameter(
            torch.randn(1, self.num_positions, self.embed_dim)
        )
```
**EN:** Method `InternVisionEmbeddings.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternVisionEmbeddings.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternVisionEmbeddings.forward` (lines 103-114)
```python
    def forward(self, pixel_values: torch.FloatTensor) -> torch.Tensor:
        target_dtype = self.patch_embedding.weight.dtype
        patch_embeds = self.patch_embedding(
            pixel_values.to(target_dtype)
        )  # shape = [*, channel, width, height]
        batch_size, _, height, width = patch_embeds.shape
        patch_embeds = patch_embeds.flatten(2).transpose(1, 2)
        class_embeds = self.class_embedding.expand(batch_size, 1, -1).to(target_dtype)
        embeddings = torch.cat([class_embeds, patch_embeds], dim=1)
        position_embedding = self._get_position_embedding(height, width)
        embeddings = embeddings + position_embedding.to(target_dtype)
        return embeddings
```
**EN:** Method `InternVisionEmbeddings.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternVisionEmbeddings.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `InternVisionPatchModel` (lines 117-142)
```python
class InternVisionPatchModel(nn.Module):
    def __init__(self, config: PretrainedConfig):
        super().__init__()
        self.config = config
        self.embeddings = InternVisionEmbeddings(config)

    def get_input_embeddings(self):
        return self.embeddings

    def forward(
        self,
        pixel_values: torch.Tensor | None = None,
        pixel_embeds: torch.Tensor | None = None,
    ) -> torch.FloatTensor:
        if pixel_values is None and pixel_embeds is None:
            raise ValueError("You have to specify pixel_values or pixel_embeds")

        if pixel_embeds is not None:
            hidden_states = pixel_embeds
        elif pixel_values is not None:
            if pixel_values.ndim == 4:
                hidden_states = self.embeddings(pixel_values)
            else:
                raise ValueError(f"wrong pixel_values size: {pixel_values.shape}")
```
**EN:** Class `InternVisionPatchModel` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, get_input_embeddings, forward.
**CN:** 类 `InternVisionPatchModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, get_input_embeddings, forward。

### Method `InternVisionPatchModel.__init__` (lines 118-121)
```python
    def __init__(self, config: PretrainedConfig):
        super().__init__()
        self.config = config
        self.embeddings = InternVisionEmbeddings(config)
```
**EN:** Method `InternVisionPatchModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternVisionPatchModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternVisionPatchModel.get_input_embeddings` (lines 123-124)
```python
    def get_input_embeddings(self):
        return self.embeddings
```
**EN:** Method `InternVisionPatchModel.get_input_embeddings` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `InternVisionPatchModel.get_input_embeddings` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `InternVisionPatchModel.forward` (lines 126-142)
```python
    def forward(
        self,
        pixel_values: torch.Tensor | None = None,
        pixel_embeds: torch.Tensor | None = None,
    ) -> torch.FloatTensor:
        if pixel_values is None and pixel_embeds is None:
            raise ValueError("You have to specify pixel_values or pixel_embeds")

        if pixel_embeds is not None:
            hidden_states = pixel_embeds
        elif pixel_values is not None:
            if pixel_values.ndim == 4:
                hidden_states = self.embeddings(pixel_values)
            else:
                raise ValueError(f"wrong pixel_values size: {pixel_values.shape}")

        return hidden_states
```
**EN:** Method `InternVisionPatchModel.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternVisionPatchModel.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `InternParallelAttention` (lines 145-247)
```python
class InternParallelAttention(nn.Module):
    """Multi-headed attention from 'Attention Is All You Need' paper"""

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        num_dummy_heads: int = 0,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.config = config
        self.embed_dim = config.hidden_size
        self.num_heads = config.num_attention_heads
        self.head_dim = self.embed_dim // self.num_heads
        if self.head_dim * self.num_heads != self.embed_dim:
            raise ValueError(
                f"embed_dim must be divisible by num_heads "
                f"(got `embed_dim`: {self.embed_dim} and `num_heads`:"
                f" {self.num_heads})."
            )

        use_data_parallel = is_vit_use_data_parallel()
```
**EN:** Class `InternParallelAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, _apply_qk_norm, forward.
**CN:** 类 `InternParallelAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, _apply_qk_norm, forward。

### Method `InternParallelAttention.__init__` (lines 148-223)
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        num_dummy_heads: int = 0,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.config = config
        self.embed_dim = config.hidden_size
        self.num_heads = config.num_attention_heads
        self.head_dim = self.embed_dim // self.num_heads
        if self.head_dim * self.num_heads != self.embed_dim:
            raise ValueError(
                f"embed_dim must be divisible by num_heads "
                f"(got `embed_dim`: {self.embed_dim} and `num_heads`:"
                f" {self.num_heads})."
            )

        use_data_parallel = is_vit_use_data_parallel()
        # if the number of heads is not divisible by tp_size,
        # we also disable Attention's TP
        tp_size = 1 if use_data_parallel else get_tensor_model_parallel_world_size()
        use_data_parallel = (
            use_data_parallel or (self.num_heads + num_dummy_heads) % tp_size != 0
        )
        self.tp_size = 1 if use_data_parallel else tp_size
        self.tp_rank = 0 if use_data_parallel else get_tensor_model_parallel_rank()

        # Additional dummy heads are used to enable TP for common GPU counts.
        self.dummy_dim = (num_dummy_heads + self.num_heads) * self.head_dim
        self.num_heads_per_partition = divide(
            num_dummy_heads + self.num_heads, self.tp_size
        )

        self.scale = self.head_dim**-0.5
        self.qkv = QKVParallelLinear(
            self.embed_dim,
            self.head_dim,
            num_dummy_heads + self.num_heads,
            bias=config.qkv_bias,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv",
            disable_tp=use_data_parallel,
        )

        self.qk_normalization = config.qk_normalization

        if self.qk_normalization:
            self.q_norm = RMSNorm(
                self.dummy_dim,
                eps=config.layer_norm_eps,
                var_hidden_size=self.embed_dim,
            )
            self.k_norm = RMSNorm(
                self.dummy_dim,
                eps=config.layer_norm_eps,
                var_hidden_size=self.embed_dim,
# ... truncated for analysis ...
```
**EN:** Method `InternParallelAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternParallelAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternParallelAttention.forward` (lines 237-247)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        B, N, _ = x.shape
        qkv, _ = self.qkv(x)
        q, k, v = qkv.chunk(3, dim=-1)

        if self.qk_normalization:
            q, k = self._apply_qk_norm(q, k)

        out = self.attn(q, k, v)
        out, _ = self.proj(out)
        return out
```
**EN:** Method `InternParallelAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternParallelAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `InternMLP` (lines 250-284)
```python
class InternMLP(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.config = config
        self.activation_fn = get_act_fn(config.hidden_act)
        use_data_parallel = is_vit_use_data_parallel()
        self.fc1 = ColumnParallelLinear(
            config.hidden_size,
            config.intermediate_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.fc1",
            disable_tp=use_data_parallel,
        )
        self.fc2 = RowParallelLinear(
            config.intermediate_size,
            config.hidden_size,
            bias=True,
            quant_config=quant_config,
```
**EN:** Class `InternMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `InternMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `InternMLP.__init__` (lines 251-277)
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.config = config
        self.activation_fn = get_act_fn(config.hidden_act)
        use_data_parallel = is_vit_use_data_parallel()
        self.fc1 = ColumnParallelLinear(
            config.hidden_size,
            config.intermediate_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.fc1",
            disable_tp=use_data_parallel,
        )
        self.fc2 = RowParallelLinear(
            config.intermediate_size,
            config.hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.fc2",
            disable_tp=use_data_parallel,
        )
```
**EN:** Method `InternMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternMLP.forward` (lines 279-284)
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states, _ = self.fc1(hidden_states)
        hidden_states = self.activation_fn(hidden_states)
        hidden_states, _ = self.fc2(hidden_states)

        return hidden_states
```
**EN:** Method `InternMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `InternVisionEncoderLayer` (lines 292-349)
```python
@support_torch_compile(
    dynamic_arg_dims={"hidden_states": 0},
    enable_if=should_torch_compile_mm_encoder,
    is_encoder=True,
)
class InternVisionEncoderLayer(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        num_dummy_heads: int = 0,
        prefix: str = "",
        attn_cls: type[InternParallelAttention] = InternParallelAttention,
    ) -> None:
        super().__init__()
        self.embed_dim = config.hidden_size
        self.intermediate_size = config.intermediate_size
        self.norm_type = config.norm_type
        self.attn_cls = attn_cls

        self.attn = self._init_attn(
            config,
            quant_config,
            num_dummy_heads=num_dummy_heads,
```
**EN:** Class `InternVisionEncoderLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, _init_attn, forward.
**CN:** 类 `InternVisionEncoderLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, _init_attn, forward。

### Method `InternVisionEncoderLayer.__init__` (lines 293-324)
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        num_dummy_heads: int = 0,
        prefix: str = "",
        attn_cls: type[InternParallelAttention] = InternParallelAttention,
    ) -> None:
        super().__init__()
        self.embed_dim = config.hidden_size
        self.intermediate_size = config.intermediate_size
        self.norm_type = config.norm_type
        self.attn_cls = attn_cls

        self.attn = self._init_attn(
            config,
            quant_config,
            num_dummy_heads=num_dummy_heads,
            prefix=f"{prefix}.attn",
        )

        self.mlp = InternMLP(
            config,
            quant_config=quant_config,
            prefix=f"{prefix}.mlp",
        )
        self.norm1 = NORM2FN[self.norm_type](self.embed_dim, eps=config.layer_norm_eps)
        self.norm2 = NORM2FN[self.norm_type](self.embed_dim, eps=config.layer_norm_eps)

        self.ls1 = nn.Parameter(config.initializer_factor * torch.ones(self.embed_dim))
        self.ls2 = nn.Parameter(config.initializer_factor * torch.ones(self.embed_dim))
```
**EN:** Method `InternVisionEncoderLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternVisionEncoderLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternVisionEncoderLayer.forward` (lines 341-349)
```python
    def forward(
        self,
        hidden_states: torch.Tensor,
    ):
        hidden_states = hidden_states + self.attn(self.norm1(hidden_states)) * self.ls1

        hidden_states = hidden_states + self.mlp(self.norm2(hidden_states)) * self.ls2

        return hidden_states
```
**EN:** Method `InternVisionEncoderLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternVisionEncoderLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `InternVisionEncoder` (lines 352-390)
```python
class InternVisionEncoder(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        num_hidden_layers_override: int | None = None,
        num_dummy_heads: int = 0,
        prefix: str = "",
        layer_cls: type[InternVisionEncoderLayer] = InternVisionEncoderLayer,
    ):
        super().__init__()

        self.config = config
        self.layer_cls = layer_cls

        if num_hidden_layers_override is None:
            num_hidden_layers = config.num_hidden_layers
        else:
            num_hidden_layers = num_hidden_layers_override

        self.layers = nn.ModuleList(
            [
                self.layer_cls(
                    config=config,
```
**EN:** Class `InternVisionEncoder` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `InternVisionEncoder` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `InternVisionEncoder.__init__` (lines 353-383)
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        num_hidden_layers_override: int | None = None,
        num_dummy_heads: int = 0,
        prefix: str = "",
        layer_cls: type[InternVisionEncoderLayer] = InternVisionEncoderLayer,
    ):
        super().__init__()

        self.config = config
        self.layer_cls = layer_cls

        if num_hidden_layers_override is None:
            num_hidden_layers = config.num_hidden_layers
        else:
            num_hidden_layers = num_hidden_layers_override

        self.layers = nn.ModuleList(
            [
                self.layer_cls(
                    config=config,
                    quant_config=quant_config,
                    num_dummy_heads=num_dummy_heads,
                    prefix=f"{prefix}.layers.{layer_idx}",
                )
                for layer_idx in range(num_hidden_layers)
            ]
        )
```
**EN:** Method `InternVisionEncoder.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternVisionEncoder.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternVisionEncoder.forward` (lines 385-390)
```python
    def forward(self, inputs_embeds: torch.Tensor):
        hidden_states = inputs_embeds
        for encoder_layer in self.layers:
            hidden_states = encoder_layer(hidden_states)

        return hidden_states
```
**EN:** Method `InternVisionEncoder.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternVisionEncoder.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `InternVisionModel` (lines 393-455)
```python
class InternVisionModel(nn.Module):
    packed_modules_mapping = {
        "qkv": ["qkv"],
    }

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        num_hidden_layers_override: int | None = None,
        num_dummy_heads: int = 0,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.config = config
        self.use_data_parallel = is_vit_use_data_parallel()

        self.embeddings = InternVisionEmbeddings(config)
        self.encoder = InternVisionEncoder(
            config=config,
            quant_config=quant_config,
            num_hidden_layers_override=num_hidden_layers_override,
            num_dummy_heads=num_dummy_heads,
```
**EN:** Class `InternVisionModel` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, get_input_embeddings, forward, load_weights.
**CN:** 类 `InternVisionModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, get_input_embeddings, forward, load_weights。

### Method `InternVisionModel.__init__` (lines 398-419)
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        *,
        num_hidden_layers_override: int | None = None,
        num_dummy_heads: int = 0,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.config = config
        self.use_data_parallel = is_vit_use_data_parallel()

        self.embeddings = InternVisionEmbeddings(config)
        self.encoder = InternVisionEncoder(
            config=config,
            quant_config=quant_config,
            num_hidden_layers_override=num_hidden_layers_override,
            num_dummy_heads=num_dummy_heads,
            prefix=f"{prefix}.encoder",
        )
```
**EN:** Method `InternVisionModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `InternVisionModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `InternVisionModel.get_input_embeddings` (lines 421-422)
```python
    def get_input_embeddings(self):
        return self.embeddings
```
**EN:** Method `InternVisionModel.get_input_embeddings` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `InternVisionModel.get_input_embeddings` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `InternVisionModel.forward` (lines 424-445)
```python
    def forward(
        self,
        pixel_values: torch.Tensor | None = None,
        pixel_embeds: torch.Tensor | None = None,
    ) -> torch.FloatTensor:
        if pixel_values is None and pixel_embeds is None:
            raise ValueError("You have to specify pixel_values or pixel_embeds")

        if pixel_embeds is not None:
            hidden_states = pixel_embeds
        elif pixel_values is not None:
            if pixel_values.ndim == 4:
                hidden_states = self.embeddings(pixel_values)
            else:
                raise ValueError(f"wrong pixel_values size: {pixel_values.shape}")

        if self.use_data_parallel:
            encoder_outputs = run_dp_sharded_vision_model(hidden_states, self.encoder)
        else:
            encoder_outputs = self.encoder(inputs_embeds=hidden_states)

        return encoder_outputs
```
**EN:** Method `InternVisionModel.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `InternVisionModel.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `InternVisionModel.load_weights` (lines 447-455)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()
        for name, loaded_weight in weights:
            param = params_dict[name]
            weight_loader = getattr(param, "weight_loader", default_weight_loader)
            weight_loader(param, loaded_weight)
            loaded_params.add(name)
        return loaded_params
```
**EN:** Method `InternVisionModel.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `InternVisionModel.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

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
- **Standard library / 标准库**: `from collections.abc import Iterable`, `from functools import partial`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`, `import torch.nn.functional as F`, `from transformers import PretrainedConfig`
- **vLLM internal / vLLM 内部依赖**: `from vllm.compilation.decorators import (`, `from vllm.distributed import (`, `from vllm.model_executor.layers.activation import get_act_fn`, `from vllm.model_executor.layers.attention import MMEncoderAttention`, `from vllm.model_executor.layers.conv import Conv2dLayer`, `from vllm.model_executor.layers.layernorm import RMSNorm`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.model_loader.weight_utils import default_weight_loader`, `from .vision import is_vit_use_data_parallel, run_dp_sharded_vision_model`
