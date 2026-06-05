# glm4v.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/glm4v.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for glm4v, including architecture wrappers and weight loading logic. / 面向推理的 glm4v vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 8-63)
```python
from argparse import Namespace
from collections.abc import Iterator, Mapping, Sequence
from typing import Annotated, Literal

import numpy as np
import torch
from torch import nn
from torch.nn import LayerNorm
from transformers import BatchFeature

from vllm.config import VllmConfig
from vllm.config.multimodal import BaseDummyOptions
from vllm.distributed import get_tensor_model_parallel_world_size
from vllm.inputs import MultiModalDataDict
from vllm.model_executor.layers.activation import SiluAndMul, get_act_fn
from vllm.model_executor.layers.attention import MMEncoderAttention
from vllm.model_executor.layers.conv import Conv2dLayer
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
    MergedColumnParallelLinear,
    QKVParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
)
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.models.module_mapping import MultiModelKeys
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    MultiModalFeatureSpec,
    MultiModalFieldConfig,
    MultiModalKwargsItems,
)
from vllm.multimodal.parse import MultiModalDataItems
from vllm.multimodal.processing import (
    BaseDummyInputsBuilder,
    BaseMultiModalProcessor,
    BaseProcessingInfo,
    PromptReplacement,
    PromptUpdate,
)
from vllm.sequence import IntermediateTensors
from vllm.transformers_utils.configs.chatglm import ChatGLMConfig
from vllm.transformers_utils.processors.glm4v import (
    GLM4VImageProcessorFast,
    GLM4VProcessor,
)
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .chatglm import ChatGLMBaseModel, ChatGLMModel, GLMTransformer
from .interfaces import (
    MultiModalEmbeddings,
    SupportsLoRA,
    SupportsMRoPE,
    SupportsMultiModal,
    SupportsPP,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `GLMVImagePixelInputs` (lines 66-76)
```python
class GLMVImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - b: Batch size
        - c: Number of channels (3)
        - h: Height of image
        - w: Width of image
    """

    type: Literal["pixel_values"] = "pixel_values"
    data: Annotated[torch.Tensor, TensorShape("b", 3, "h", "w")]
```
**EN:** Class `GLMVImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `GLMVImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `EVA2CLIPPatchEmbedding` (lines 79-107)
```python
class EVA2CLIPPatchEmbedding(nn.Module):
    def __init__(self, config):
        super().__init__()
        self.proj = Conv2dLayer(
            config.in_channels,
            config.hidden_size,
            kernel_size=config.patch_size,
            stride=config.patch_size,
        )
        self.cls_embedding = nn.Parameter(torch.zeros(1, config.hidden_size))
        self.position_embedding = nn.Embedding(config.num_positions, config.hidden_size)

    def forward(self, images: torch.Tensor) -> torch.Tensor:
        """
        Parameters:
        images : torch.Tensor
            Input image tensor with shape (B, C, H, W)

        Returns:
        torch.Tensor
            Transformed tensor with shape (B, L, D)
        """
        images = images.to(device=self.proj.weight.device, dtype=self.proj.weight.dtype)
        x = self.proj(images)
        x = x.flatten(2).transpose(1, 2)
```
**EN:** Class `EVA2CLIPPatchEmbedding` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `EVA2CLIPPatchEmbedding` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `EVA2CLIPPatchEmbedding.__init__` (lines 80-89)
```python
    def __init__(self, config):
        super().__init__()
        self.proj = Conv2dLayer(
            config.in_channels,
            config.hidden_size,
            kernel_size=config.patch_size,
            stride=config.patch_size,
        )
        self.cls_embedding = nn.Parameter(torch.zeros(1, config.hidden_size))
        self.position_embedding = nn.Embedding(config.num_positions, config.hidden_size)
```
**EN:** Method `EVA2CLIPPatchEmbedding.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `EVA2CLIPPatchEmbedding.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `EVA2CLIPPatchEmbedding.forward` (lines 91-107)
```python
    def forward(self, images: torch.Tensor) -> torch.Tensor:
        """
        Parameters:
        images : torch.Tensor
            Input image tensor with shape (B, C, H, W)

        Returns:
        torch.Tensor
            Transformed tensor with shape (B, L, D)
        """
        images = images.to(device=self.proj.weight.device, dtype=self.proj.weight.dtype)
        x = self.proj(images)
        x = x.flatten(2).transpose(1, 2)
        cls_token = self.cls_embedding.expand(x.shape[0], -1, -1)
        x = torch.cat((cls_token, x), dim=1)
        x += self.position_embedding.weight.unsqueeze(0)
        return x
```
**EN:** Method `EVA2CLIPPatchEmbedding.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Parameters: images : torch.Tensor Input image tensor with shape (B, C, H, W)  Returns: torch.Tensor Transformed tensor with shape (B, L, D).
**CN:** Method `EVA2CLIPPatchEmbedding.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Parameters: images : torch.Tensor Input image tensor with shape (B, C, H, W)  Returns: torch.Tensor Transformed tensor with shape (B, L, D)。

### Class `EVA2CLIPAttention` (lines 110-153)
```python
class EVA2CLIPAttention(nn.Module):
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        self.tp_size = get_tensor_model_parallel_world_size()
        self.num_heads_per_rank = config.num_heads // self.tp_size
        self.head_dim = config.hidden_size // config.num_heads
        self.scale = self.head_dim**-0.5

        self.query_key_value = QKVParallelLinear(
            config.hidden_size,
            self.head_dim,
            config.num_heads,
            quant_config=quant_config,
            prefix=f"{prefix}.query_key_value",
        )
        self.dense = RowParallelLinear(
            config.hidden_size,
            config.hidden_size,
            quant_config=quant_config,
```
**EN:** Class `EVA2CLIPAttention` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `EVA2CLIPAttention` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `EVA2CLIPAttention.__init__` (lines 111-144)
```python
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        self.tp_size = get_tensor_model_parallel_world_size()
        self.num_heads_per_rank = config.num_heads // self.tp_size
        self.head_dim = config.hidden_size // config.num_heads
        self.scale = self.head_dim**-0.5

        self.query_key_value = QKVParallelLinear(
            config.hidden_size,
            self.head_dim,
            config.num_heads,
            quant_config=quant_config,
            prefix=f"{prefix}.query_key_value",
        )
        self.dense = RowParallelLinear(
            config.hidden_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=f"{prefix}.dense",
        )

        self.attn = MMEncoderAttention(
            self.num_heads_per_rank,
            self.head_dim,
            self.scale,
            prefix=f"{prefix}.attn",
        )
        self.output_dropout = torch.nn.Dropout(config.dropout_prob)
```
**EN:** Method `EVA2CLIPAttention.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `EVA2CLIPAttention.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `EVA2CLIPAttention.forward` (lines 146-153)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        qkv, _ = self.query_key_value(x)  # B, L, 3 * H * D
        q, k, v = qkv.chunk(3, dim=-1)

        out = self.attn(q, k, v)
        output, _ = self.dense(out)
        output = self.output_dropout(output)
        return output
```
**EN:** Method `EVA2CLIPAttention.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `EVA2CLIPAttention.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `EVA2CLIPMLP` (lines 156-183)
```python
class EVA2CLIPMLP(nn.Module):
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.activation_fn = get_act_fn(config.hidden_act)
        self.fc1 = ColumnParallelLinear(
            config.hidden_size,
            config.intermediate_size,
            quant_config=quant_config,
            prefix=f"{prefix}.fc1",
        )
        self.fc2 = RowParallelLinear(
            config.intermediate_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=f"{prefix}.fc2",
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x, _ = self.fc1(x)
```
**EN:** Class `EVA2CLIPMLP` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `EVA2CLIPMLP` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `EVA2CLIPMLP.__init__` (lines 157-177)
```python
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.activation_fn = get_act_fn(config.hidden_act)
        self.fc1 = ColumnParallelLinear(
            config.hidden_size,
            config.intermediate_size,
            quant_config=quant_config,
            prefix=f"{prefix}.fc1",
        )
        self.fc2 = RowParallelLinear(
            config.intermediate_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=f"{prefix}.fc2",
        )
```
**EN:** Method `EVA2CLIPMLP.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `EVA2CLIPMLP.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `EVA2CLIPMLP.forward` (lines 179-183)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x, _ = self.fc1(x)
        x = self.activation_fn(x)
        x, _ = self.fc2(x)
        return x
```
**EN:** Method `EVA2CLIPMLP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `EVA2CLIPMLP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `EVA2CLIPTransformerLayer` (lines 186-212)
```python
class EVA2CLIPTransformerLayer(nn.Module):
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.input_layernorm = LayerNorm(config.hidden_size, eps=config.layer_norm_eps)
        self.attention = EVA2CLIPAttention(
            config, quant_config=quant_config, prefix=f"{prefix}.attention"
        )
        self.mlp = EVA2CLIPMLP(
            config, quant_config=quant_config, prefix=f"{prefix}.mlp"
        )
        self.post_attention_layernorm = LayerNorm(
            config.hidden_size, eps=config.layer_norm_eps
        )

    def forward(self, hidden_states):
        attention_input = hidden_states
        attention_output = self.input_layernorm(self.attention(attention_input))
        hidden_states = attention_input + attention_output
        mlp_input = hidden_states
        mlp_output = self.post_attention_layernorm(self.mlp(mlp_input))
```
**EN:** Class `EVA2CLIPTransformerLayer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `EVA2CLIPTransformerLayer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `EVA2CLIPTransformerLayer.__init__` (lines 187-203)
```python
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.input_layernorm = LayerNorm(config.hidden_size, eps=config.layer_norm_eps)
        self.attention = EVA2CLIPAttention(
            config, quant_config=quant_config, prefix=f"{prefix}.attention"
        )
        self.mlp = EVA2CLIPMLP(
            config, quant_config=quant_config, prefix=f"{prefix}.mlp"
        )
        self.post_attention_layernorm = LayerNorm(
            config.hidden_size, eps=config.layer_norm_eps
        )
```
**EN:** Method `EVA2CLIPTransformerLayer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `EVA2CLIPTransformerLayer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `EVA2CLIPTransformerLayer.forward` (lines 205-212)
```python
    def forward(self, hidden_states):
        attention_input = hidden_states
        attention_output = self.input_layernorm(self.attention(attention_input))
        hidden_states = attention_input + attention_output
        mlp_input = hidden_states
        mlp_output = self.post_attention_layernorm(self.mlp(mlp_input))
        output = mlp_input + mlp_output
        return output
```
**EN:** Method `EVA2CLIPTransformerLayer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `EVA2CLIPTransformerLayer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `EVA2CLIPTransformer` (lines 215-237)
```python
class EVA2CLIPTransformer(nn.Module):
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.layers = nn.ModuleList(
            [
                EVA2CLIPTransformerLayer(
                    config,
                    quant_config=quant_config,
                    prefix=f"{prefix}.layers.{layer_idx}",
                )
                for layer_idx in range(config.num_hidden_layers)
            ]
        )

    def forward(self, hidden_states):
        for layer_module in self.layers:
            hidden_states = layer_module(hidden_states)
        return hidden_states
```
**EN:** Class `EVA2CLIPTransformer` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `EVA2CLIPTransformer` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `EVA2CLIPTransformer.__init__` (lines 216-232)
```python
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.layers = nn.ModuleList(
            [
                EVA2CLIPTransformerLayer(
                    config,
                    quant_config=quant_config,
                    prefix=f"{prefix}.layers.{layer_idx}",
                )
                for layer_idx in range(config.num_hidden_layers)
            ]
        )
```
**EN:** Method `EVA2CLIPTransformer.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `EVA2CLIPTransformer.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `EVA2CLIPTransformer.forward` (lines 234-237)
```python
    def forward(self, hidden_states):
        for layer_module in self.layers:
            hidden_states = layer_module(hidden_states)
        return hidden_states
```
**EN:** Method `EVA2CLIPTransformer.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `EVA2CLIPTransformer.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `EVA2CLIPGLU` (lines 240-318)
```python
class EVA2CLIPGLU(nn.Module):
    def __init__(
        self,
        config,
        in_features,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        """
        The original implementation is the same as:
        ```python
        self.dense_h_to_4h = ColumnParallelLinear(
            config.hidden_size,
            config.ffn_hidden_size,
            bias=False,
            quant_config=quant_config,
        )

        self.gate_proj = ColumnParallelLinear(
            config.hidden_size,
            config.ffn_hidden_size,
            bias=False,
            quant_config=quant_config,
        )
        ```
```
**EN:** Class `EVA2CLIPGLU` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `EVA2CLIPGLU` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `EVA2CLIPGLU.__init__` (lines 241-310)
```python
    def __init__(
        self,
        config,
        in_features,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        """
        The original implementation is the same as:
        ```python
        self.dense_h_to_4h = ColumnParallelLinear(
            config.hidden_size,
            config.ffn_hidden_size,
            bias=False,
            quant_config=quant_config,
        )

        self.gate_proj = ColumnParallelLinear(
            config.hidden_size,
            config.ffn_hidden_size,
            bias=False,
            quant_config=quant_config,
        )
        ```
        ```
        gate_proj_output, _ = self.gate_proj(x)
        dense_h_to_4h_output, _ = self.dense_h_to_4h(x)
        x = torch.cat([gate_proj_output, dense_h_to_4h_output], dim=-1)
        ```

        We merge two ColumnParallelLinear into one MergedColumnParallelLinear:
        ```
        self.merged_proj = MergedColumnParallelLinear(
            config.hidden_size,
            [config.ffn_hidden_size] * 2,
            bias=False,
            quant_config=quant_config,
        )
        ```
        ```
        x, _ = self.merged_proj(x)
        ```
        """
        super().__init__()
        self.linear_proj = ReplicatedLinear(
            in_features,
            config.hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.linear_proj",
        )
        self.norm1 = nn.LayerNorm(config.hidden_size)
        self.act1 = nn.GELU()
        self.act2 = SiluAndMul()

        self.merged_proj = MergedColumnParallelLinear(
            config.hidden_size,
            [config.ffn_hidden_size] * 2,
            bias=False,
            quant_config=quant_config,
# ... truncated for analysis ...
```
**EN:** Method `EVA2CLIPGLU.__init__` initializes the object, creates submodules, and captures the configuration required for later execution. The docstring says: The original implementation is the same as: ```python self.dense_h_to_4h = ColumnParallelLinear( config.hidden_size, config.ffn_hidden_size, bias=False, quant_config=quant_config, )  self.gate_proj = ColumnParallelLinear( config.hidden_size, config.ffn_hidden_size, bias=False, quant_config=quant_config, ) ``` ``` gate_proj_output, _ = self.gate_proj(x) dense_h_to_4h_output, _ = self.dense_h_to_4h(x) x = torch.cat([gate_proj_output, dense_h_to_4h_output], dim=-1) ```  We merge two ColumnParallelLinear into one MergedColumnParallelLinear: ``` self.merged_proj = MergedColumnParallelLinear( config.hidden_size, [config.ffn_hidden_size] * 2, bias=False, quant_config=quant_config, ) ``` ``` x, _ = self.merged_proj(x) ```.
**CN:** Method `EVA2CLIPGLU.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。 文档字符串表达的核心意思是：The original implementation is the same as: ```python self.dense_h_to_4h = ColumnParallelLinear( config.hidden_size, config.ffn_hidden_size, bias=False, quant_config=quant_config, )  self.gate_proj = ColumnParallelLinear( config.hidden_size, config.ffn_hidden_size, bias=False, quant_config=quant_config, ) ``` ``` gate_proj_output, _ = self.gate_proj(x) dense_h_to_4h_output, _ = self.dense_h_to_4h(x) x = torch.cat([gate_proj_output, dense_h_to_4h_output], dim=-1) ```  We merge two ColumnParallelLinear into one MergedColumnParallelLinear: ``` self.merged_proj = MergedColumnParallelLinear( config.hidden_size, [config.ffn_hidden_size] * 2, bias=False, quant_config=quant_config, ) ``` ``` x, _ = self.merged_proj(x) ```。

### Method `EVA2CLIPGLU.forward` (lines 312-318)
```python
    def forward(self, x):
        x, _ = self.linear_proj(x)
        x = self.act1(self.norm1(x))
        x, _ = self.merged_proj(x)
        x = self.act2(x)
        x, _ = self.dense_4h_to_h(x)
        return x
```
**EN:** Method `EVA2CLIPGLU.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `EVA2CLIPGLU.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `EVA2CLIPModel` (lines 321-375)
```python
class EVA2CLIPModel(nn.Module):
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        vision_config = Namespace(**config.vision_config)
        self.patch_embedding = EVA2CLIPPatchEmbedding(vision_config)
        self.transformer = EVA2CLIPTransformer(
            vision_config, quant_config=quant_config, prefix=f"{prefix}.transformer"
        )
        self.linear_proj = EVA2CLIPGLU(
            config,
            in_features=config.hidden_size,
            quant_config=quant_config,
            prefix=f"{prefix}.linear_proj",
        )
        self.conv = Conv2dLayer(
            in_channels=vision_config.hidden_size,
            out_channels=config.hidden_size,
            kernel_size=2,
            stride=2,
        )
```
**EN:** Class `EVA2CLIPModel` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `EVA2CLIPModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `EVA2CLIPModel.__init__` (lines 322-348)
```python
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        vision_config = Namespace(**config.vision_config)
        self.patch_embedding = EVA2CLIPPatchEmbedding(vision_config)
        self.transformer = EVA2CLIPTransformer(
            vision_config, quant_config=quant_config, prefix=f"{prefix}.transformer"
        )
        self.linear_proj = EVA2CLIPGLU(
            config,
            in_features=config.hidden_size,
            quant_config=quant_config,
            prefix=f"{prefix}.linear_proj",
        )
        self.conv = Conv2dLayer(
            in_channels=vision_config.hidden_size,
            out_channels=config.hidden_size,
            kernel_size=2,
            stride=2,
        )
        self.boi = nn.Parameter(torch.zeros(1, 1, config.hidden_size))
        self.eoi = nn.Parameter(torch.zeros(1, 1, config.hidden_size))
        self.scaling_factor = vision_config.scaling_factor
```
**EN:** Method `EVA2CLIPModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `EVA2CLIPModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `EVA2CLIPModel.forward` (lines 350-375)
```python
    def forward(self, images: torch.Tensor) -> torch.Tensor:
        """
        Parameters:
        images : torch.Tensor
            Input image tensor with shape (B, C, H, W)

        Returns:
        torch.Tensor
            Transformed tensor with shape (B, L, D)
        """
        x = self.patch_embedding(images)
        x = self.transformer(x)
        x = x[:, 1:]

        b, s, h = x.shape
        grid_size = int(s**0.5)
        x = x.view(b, grid_size, grid_size, h).permute(0, 3, 1, 2)
        x = self.conv(x)

        x = x.flatten(2).transpose(1, 2)
        x = self.linear_proj(x)
        boi = self.boi.expand(x.shape[0], -1, -1)
        eoi = self.eoi.expand(x.shape[0], -1, -1)
        x = torch.cat((boi, x, eoi), dim=1)
        x = x / self.scaling_factor
        return x
```
**EN:** Method `EVA2CLIPModel.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Parameters: images : torch.Tensor Input image tensor with shape (B, C, H, W)  Returns: torch.Tensor Transformed tensor with shape (B, L, D).
**CN:** Method `EVA2CLIPModel.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Parameters: images : torch.Tensor Input image tensor with shape (B, C, H, W)  Returns: torch.Tensor Transformed tensor with shape (B, L, D)。

### Class `GLM4VModel` (lines 378-386)
```python
class GLM4VModel(ChatGLMModel):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__(vllm_config=vllm_config, prefix=prefix)

        quant_config = vllm_config.quant_config

        self.vision = EVA2CLIPModel(
            self.config, quant_config, prefix=f"{prefix}.vision"
        )
```
**EN:** Class `GLM4VModel` organizes related behavior for this model family or helper component. It inherits from ChatGLMModel. Key methods include __init__.
**CN:** 类 `GLM4VModel` 用于组织该模型族或辅助组件的相关行为。 它继承自 ChatGLMModel。 关键方法包括 __init__。

### Method `GLM4VModel.__init__` (lines 379-386)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__(vllm_config=vllm_config, prefix=prefix)

        quant_config = vllm_config.quant_config

        self.vision = EVA2CLIPModel(
            self.config, quant_config, prefix=f"{prefix}.vision"
        )
```
**EN:** Method `GLM4VModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GLM4VModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Class `GLM4VProcessingInfo` (lines 389-423)
```python
class GLM4VProcessingInfo(BaseProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config(ChatGLMConfig)

    def get_image_processor(self, **kwargs):
        config = self.get_hf_config()
        vision_config = config.vision_config

        image_size = vision_config["image_size"]
        kwargs = self.ctx.get_merged_mm_kwargs(kwargs)
        kwargs.setdefault("size", {"width": image_size, "height": image_size})

        return GLM4VImageProcessorFast(**kwargs)

    def get_hf_processor(self, **kwargs: object) -> GLM4VProcessor:
        return GLM4VProcessor(
            tokenizer=self.get_tokenizer(),
            image_processor=self.get_image_processor(**kwargs),
        )

    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": 1}

    def get_num_image_tokens(self) -> int:
        hf_config = self.get_hf_config()
```
**EN:** Class `GLM4VProcessingInfo` organizes related behavior for this model family or helper component. It inherits from BaseProcessingInfo. Key methods include get_hf_config, get_image_processor, get_hf_processor, get_supported_mm_limits, get_num_image_tokens, get_num_image_feature_tokens.
**CN:** 类 `GLM4VProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseProcessingInfo。 关键方法包括 get_hf_config, get_image_processor, get_hf_processor, get_supported_mm_limits, get_num_image_tokens, get_num_image_feature_tokens。

### Method `GLM4VProcessingInfo.get_hf_config` (lines 390-391)
```python
    def get_hf_config(self):
        return self.ctx.get_hf_config(ChatGLMConfig)
```
**EN:** Method `GLM4VProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `GLM4VProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `GLM4VProcessingInfo.get_image_processor` (lines 393-401)
```python
    def get_image_processor(self, **kwargs):
        config = self.get_hf_config()
        vision_config = config.vision_config

        image_size = vision_config["image_size"]
        kwargs = self.ctx.get_merged_mm_kwargs(kwargs)
        kwargs.setdefault("size", {"width": image_size, "height": image_size})

        return GLM4VImageProcessorFast(**kwargs)
```
**EN:** Method `GLM4VProcessingInfo.get_image_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `GLM4VProcessingInfo.get_image_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `GLM4VProcessingInfo.get_hf_processor` (lines 403-407)
```python
    def get_hf_processor(self, **kwargs: object) -> GLM4VProcessor:
        return GLM4VProcessor(
            tokenizer=self.get_tokenizer(),
            image_processor=self.get_image_processor(**kwargs),
        )
```
**EN:** Method `GLM4VProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `GLM4VProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `GLM4VProcessingInfo.get_supported_mm_limits` (lines 409-410)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        return {"image": 1}
```
**EN:** Method `GLM4VProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `GLM4VProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Class `GLM4VDummyInputsBuilder` (lines 426-455)
```python
class GLM4VDummyInputsBuilder(BaseDummyInputsBuilder[GLM4VProcessingInfo]):
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        base_text = "<|begin_of_image|><|endoftext|><|end_of_image|>"

        return base_text * num_images

    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        hf_config = self.info.get_hf_config()
        vision_config = hf_config.vision_config

        target_width = target_height = vision_config["image_size"]
        num_images = mm_counts.get("image", 0)

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=target_width,
```
**EN:** Class `GLM4VDummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from BaseDummyInputsBuilder[GLM4VProcessingInfo]. Key methods include get_dummy_text, get_dummy_mm_data.
**CN:** 类 `GLM4VDummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseDummyInputsBuilder[GLM4VProcessingInfo]。 关键方法包括 get_dummy_text, get_dummy_mm_data。

### Method `GLM4VDummyInputsBuilder.get_dummy_text` (lines 427-432)
```python
    def get_dummy_text(self, mm_counts: Mapping[str, int]) -> str:
        num_images = mm_counts.get("image", 0)

        base_text = "<|begin_of_image|><|endoftext|><|end_of_image|>"

        return base_text * num_images
```
**EN:** Method `GLM4VDummyInputsBuilder.get_dummy_text` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `GLM4VDummyInputsBuilder.get_dummy_text` 封装了该模块中的一段可复用核心逻辑。

### Method `GLM4VDummyInputsBuilder.get_dummy_mm_data` (lines 434-455)
```python
    def get_dummy_mm_data(
        self,
        seq_len: int,
        mm_counts: Mapping[str, int],
        mm_options: Mapping[str, BaseDummyOptions],
    ) -> MultiModalDataDict:
        hf_config = self.info.get_hf_config()
        vision_config = hf_config.vision_config

        target_width = target_height = vision_config["image_size"]
        num_images = mm_counts.get("image", 0)

        image_overrides = mm_options.get("image")

        return {
            "image": self._get_dummy_images(
                width=target_width,
                height=target_height,
                num_images=num_images,
                overrides=image_overrides,
            )
        }
```
**EN:** Method `GLM4VDummyInputsBuilder.get_dummy_mm_data` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `GLM4VDummyInputsBuilder.get_dummy_mm_data` 封装了该模块中的一段可复用核心逻辑。

### Class `GLM4VMultiModalProcessor` (lines 458-499)
```python
class GLM4VMultiModalProcessor(BaseMultiModalProcessor[GLM4VProcessingInfo]):
    def _hf_processor_applies_updates(
        self,
        prompt_text: str,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, object],
        tokenization_kwargs: Mapping[str, object],
    ) -> bool:
        return False

    def _get_mm_fields_config(
        self,
        hf_inputs: BatchFeature,
        hf_processor_mm_kwargs: Mapping[str, object],
    ) -> Mapping[str, MultiModalFieldConfig]:
        return dict(pixel_values=MultiModalFieldConfig.batched("image"))

    def _get_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, object],
        out_mm_kwargs: MultiModalKwargsItems,
    ) -> Sequence[PromptUpdate]:
        hf_config = self.info.get_hf_config()
```
**EN:** Class `GLM4VMultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[GLM4VProcessingInfo]. Key methods include _hf_processor_applies_updates, _get_mm_fields_config, _get_prompt_updates.
**CN:** 类 `GLM4VMultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[GLM4VProcessingInfo]。 关键方法包括 _hf_processor_applies_updates, _get_mm_fields_config, _get_prompt_updates。

### Class `GLM4VForCausalLM` (lines 507-649)
```python
@MULTIMODAL_REGISTRY.register_processor(
    GLM4VMultiModalProcessor,
    info=GLM4VProcessingInfo,
    dummy_inputs=GLM4VDummyInputsBuilder,
)
class GLM4VForCausalLM(
    ChatGLMBaseModel, SupportsMultiModal, SupportsLoRA, SupportsPP, SupportsMRoPE
):
    packed_modules_mapping = {
        "query_key_value": ["query_key_value"],
        "dense_h_to_4h": ["dense_h_to_4h"],
        "merged_proj": ["gate_proj", "dense_h_to_4h"],
    }

    def get_mm_mapping(self) -> MultiModelKeys:
        """
        Get the module prefix in multimodal models
        """
        return MultiModelKeys.from_string_field(
            language_model="transformer.encoder",
            connector="transformer.vision.linear_proj",
            tower_model="transformer.vision.transformer",
        )

    @classmethod
```
**EN:** Class `GLM4VForCausalLM` organizes related behavior for this model family or helper component. It inherits from ChatGLMBaseModel, SupportsMultiModal, SupportsLoRA, SupportsPP, SupportsMRoPE. Key methods include get_mm_mapping, get_placeholder_str, __init__, _parse_and_validate_image_input, _process_image_input, iter_mm_grid_thw.
**CN:** 类 `GLM4VForCausalLM` 用于组织该模型族或辅助组件的相关行为。 它继承自 ChatGLMBaseModel、SupportsMultiModal、SupportsLoRA、SupportsPP、SupportsMRoPE。 关键方法包括 get_mm_mapping, get_placeholder_str, __init__, _parse_and_validate_image_input, _process_image_input, iter_mm_grid_thw。

### Method `GLM4VForCausalLM.get_mm_mapping` (lines 516-524)
```python
    def get_mm_mapping(self) -> MultiModelKeys:
        """
        Get the module prefix in multimodal models
        """
        return MultiModelKeys.from_string_field(
            language_model="transformer.encoder",
            connector="transformer.vision.linear_proj",
            tower_model="transformer.vision.transformer",
        )
```
**EN:** Method `GLM4VForCausalLM.get_mm_mapping` encapsulates a focused piece of reusable logic inside this module. The docstring says: Get the module prefix in multimodal models.
**CN:** Method `GLM4VForCausalLM.get_mm_mapping` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Get the module prefix in multimodal models。

### Method `GLM4VForCausalLM.get_placeholder_str` (lines 527-531)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        if modality.startswith("image"):
            return "<|begin_of_image|><|endoftext|><|end_of_image|>"

        raise ValueError("Only image modality is supported")
```
**EN:** Method `GLM4VForCausalLM.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `GLM4VForCausalLM.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。

### Method `GLM4VForCausalLM.__init__` (lines 533-551)
```python
    def __init__(
        self,
        *,
        vllm_config: VllmConfig,
        prefix: str = "",
        transformer_type: type[GLM4VModel] = GLM4VModel,
    ) -> None:
        with self._mark_composite_model(
            vllm_config,
            language_targets=GLMTransformer,
            tower_targets={"image": EVA2CLIPModel},
        ):
            super().__init__(
                vllm_config=vllm_config,
                prefix=prefix,
                transformer_type=transformer_type,
            )

        self.transformer: GLM4VModel
```
**EN:** Method `GLM4VForCausalLM.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `GLM4VForCausalLM.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `GLM4VForCausalLM.iter_mm_grid_thw` (lines 573-586)
```python
    def iter_mm_grid_thw(
        self, mm_features: list[MultiModalFeatureSpec]
    ) -> Iterator[tuple[int, int, int, int]]:
        hf_config = self.config
        spatial_merge_size = hf_config.vision_config.spatial_merge_size
        for mm_feature in sorted(mm_features, key=lambda f: f.mm_position.offset):
            offset = mm_feature.mm_position.offset
            if mm_feature.modality == "image":
                t, h, w = mm_feature.data["image_grid_thw"].data.tolist()
                assert t == 1, f"Image must have 1 frame, got {t}"
                yield offset, t, h // spatial_merge_size, w // spatial_merge_size
            else:
                # glm4v only supports image modality
                raise ValueError(f"Unsupported modality: {mm_feature.modality}")
```
**EN:** Method `GLM4VForCausalLM.iter_mm_grid_thw` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `GLM4VForCausalLM.iter_mm_grid_thw` 封装了该模块中的一段可复用核心逻辑。

## Key Concepts / 关键概念
- **Attention / 注意力**
  - **EN:** Attention layers model token-to-token or token-to-feature interactions inside the architecture.
  - **CN:** 注意力层负责建模 token 与 token，或 token 与特征之间的交互。
- **LoRA support / LoRA 支持**
  - **EN:** The file is aware of low-rank adaptation interfaces or compatible parameter layouts.
  - **CN:** 该文件考虑了 LoRA 接口或兼容的低秩参数布局。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from argparse import Namespace`, `from collections.abc import Iterator, Mapping, Sequence`, `from typing import Annotated, Literal`
- **Third-party / 第三方**: `import numpy as np`, `import torch`, `from torch import nn`, `from torch.nn import LayerNorm`, `from transformers import BatchFeature`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.multimodal import BaseDummyOptions`, `from vllm.distributed import get_tensor_model_parallel_world_size`, `from vllm.inputs import MultiModalDataDict`, `from vllm.model_executor.layers.activation import SiluAndMul, get_act_fn`, `from vllm.model_executor.layers.attention import MMEncoderAttention`, `from vllm.model_executor.layers.conv import Conv2dLayer`, `from vllm.model_executor.layers.linear import (`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.model_executor.models.module_mapping import MultiModelKeys`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`
- **Module note / 模块说明**: **EN:** Inference-only CogAgent model compatible with THUDM weights. **CN:** 模块文档字符串给出的原始说明是：Inference-only CogAgent model compatible with THUDM weights.。
