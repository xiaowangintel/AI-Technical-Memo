# gemma4_vision.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/gemma4_vision.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the gemma4 Vision architecture into SGLang's serving runtime, typically defining layers, model wrappers, and weight-loading behavior for inference. / 该模块将 gemma4 Vision 架构接入 SGLang 服务运行时，通常定义推理所需的层、模型封装以及权重加载逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13: Module header and imports / 模块头与导入
```python
# Copyright 2025 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 40-43: Function: _rotate_half() / 函数：_rotate_half()
```python
def _rotate_half(x: torch.Tensor) -> torch.Tensor:
    x1 = x[..., : x.shape[-1] // 2]
    x2 = x[..., x.shape[-1] // 2 :]
    return torch.cat((-x2, x1), dim=-1)
```
**EN:** This function implements rotate half for the surrounding model/runtime logic. Key parameters include x.
**CN:** 该函数实现了 rotate half 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 46-49: Function: _apply_rotary() / 函数：_apply_rotary()
```python
def _apply_rotary(
    x: torch.Tensor, cos: torch.Tensor, sin: torch.Tensor
) -> torch.Tensor:
    return (x * cos) + (_rotate_half(x) * sin)
```
**EN:** This function implements apply rotary for the surrounding model/runtime logic. Key parameters include x, cos, sin.
**CN:** 该函数实现了 apply rotary 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 52-98: Class: Gemma4VisionRotaryEmbedding / 类：Gemma4VisionRotaryEmbedding
```python
class Gemma4VisionRotaryEmbedding(nn.Module):
    """Compute 2-D multidimensional RoPE cos/sin for patch positions."""

    def __init__(self, config: Gemma4VisionConfig):
        super().__init__()
        self.head_dim = config.head_dim
        self.rope_theta: float = config.rope_parameters["rope_theta"]

    @torch.no_grad()
    def forward(
        self, x: torch.Tensor, patch_positions: torch.Tensor
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        """
        Args:
            x: [batch, seq, hidden] – only used for device/dtype.
            patch_positions: [batch, num_patches, 2] – (x, y) coordinates.
        Returns:
            (cos, sin) each of shape [batch, num_patches, head_dim].
        """
        ndim = patch_positions.shape[-1]  # 2
        head_dim_per_dim = self.head_dim // ndim

        all_embs = []
        for d in range(ndim):
            dim_inv_freq = 1.0 / (
                self.rope_theta
                ** (
                    torch.arange(
                        0, head_dim_per_dim, 2, device=x.device, dtype=torch.float
                    )
                    / head_dim_per_dim
                )
            )
            dim_inv_freq_expanded = dim_inv_freq[None, :, None].expand(
                patch_positions.shape[0], -1, 1
            )
            dim_positions = patch_positions[:, :, d].float()
            dim_positions_expanded = dim_positions[:, None, :]

            dim_freqs = (dim_inv_freq_expanded @ dim_positions_expanded).transpose(1, 2)
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Vision Rotary Embedding inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Compute 2-D multidimensional RoPE cos/sin for patch positions..
**CN:** 该类定义了 Gemma4 Vision Rotary Embedding，用于封装该模型组件的状态与方法。 文档字符串摘要：Compute 2-D multidimensional RoPE cos/sin for patch positions.。

### Lines 101-119: Function: _apply_multidimensional_rope() / 函数：_apply_multidimensional_rope()
```python
def _apply_multidimensional_rope(
    x: torch.Tensor,
    cos: torch.Tensor,
    sin: torch.Tensor,
) -> torch.Tensor:
    """Apply 2-D RoPE to x of shape [batch*seq, heads, head_dim].

    cos/sin have shape [batch, seq, head_dim]. We split along head_dim into
    ndim=2 parts and apply standard rotary to each independently.
    """
    ndim = 2
    chunk_size = x.shape[-1] // ndim
    x_parts = x.split(chunk_size, dim=-1)
    cos_parts = cos.split(chunk_size, dim=-1)
    sin_parts = sin.split(chunk_size, dim=-1)
    y_parts = [
        _apply_rotary(x_parts[k], cos_parts[k], sin_parts[k]) for k in range(ndim)
    ]
    return torch.cat(y_parts, dim=-1)
```
**EN:** This function implements apply multidimensional rope for the surrounding model/runtime logic. Key parameters include x, cos, sin.
**CN:** 该函数实现了 apply multidimensional rope 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 127-248: Class: Gemma4VisionAttention / 类：Gemma4VisionAttention
```python
class Gemma4VisionAttention(nn.Module):
    """Multi-head attention for the Gemma 4 vision encoder.

    QKV uses a fused ``ClippableQKVParallelLinear`` for efficient matmul with
    per-projection clip bounds.  Output projection uses ``ClippableLinear``.
    """

    def __init__(
        self,
        config: Gemma4VisionConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.head_dim = config.head_dim

        tp_size = get_attention_tp_size()
        self.num_heads_per_partition = config.num_attention_heads // tp_size
        self.num_kv_heads_per_partition = config.num_key_value_heads // tp_size

        self.qkv = ClippableQKVParallelLinear(
            hidden_size=config.hidden_size,
            head_size=config.head_dim,
            total_num_heads=config.num_attention_heads,
            total_num_kv_heads=config.num_key_value_heads,
            bias=config.attention_bias,
            quant_config=quant_config,
            prefix=prefix,
        )
        self.o_proj = ClippableRowParallelLinear(
            input_size=config.num_attention_heads * config.head_dim,
            output_size=config.hidden_size,
            bias=config.attention_bias,
            quant_config=quant_config,
            prefix=add_prefix("o_proj", prefix),
        )

        self.q_norm = Gemma4RMSNorm(self.head_dim, eps=config.rms_norm_eps)
        self.k_norm = Gemma4RMSNorm(self.head_dim, eps=config.rms_norm_eps)
        self.v_norm = Gemma4RMSNorm(
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Vision Attention inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Multi-head attention for the Gemma 4 vision encoder..
**CN:** 该类定义了 Gemma4 Vision Attention，用于封装该模型组件的状态与方法。 文档字符串摘要：Multi-head attention for the Gemma 4 vision encoder.。

### Lines 256-288: Class: Gemma4VisionMLP / 类：Gemma4VisionMLP
```python
class Gemma4VisionMLP(nn.Module):
    def __init__(
        self,
        config: Gemma4VisionConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        if config.hidden_activation != "gelu_pytorch_tanh":
            raise ValueError(
                f"Gemma4VisionMLP expects hidden_activation='gelu_pytorch_tanh', "
                f"got {config.hidden_activation!r}"
            )
        self.gate_up = ClippableGateUpParallelLinear(
            input_size=config.hidden_size,
            intermediate_size=config.intermediate_size,
            bias=False,
            quant_config=quant_config,
            prefix=prefix,
        )
        self.down_proj = ClippableRowParallelLinear(
            input_size=config.intermediate_size,
            output_size=config.hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("down_proj", prefix),
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        gate, up = self.gate_up(x)
        x = F.gelu(gate, approximate="tanh") * up
        x = self.down_proj(x)
        return x
```
**EN:** This class defines Gemma4 Vision M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Vision M L P，用于封装该模型组件的状态与方法。

### Lines 296-344: Class: Gemma4VisionEncoderLayer / 类：Gemma4VisionEncoderLayer
```python
class Gemma4VisionEncoderLayer(nn.Module):
    def __init__(
        self,
        config: Gemma4VisionConfig,
        layer_idx: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.self_attn = Gemma4VisionAttention(
            config,
            quant_config=quant_config,
            prefix=add_prefix("self_attn", prefix),
        )
        self.mlp = Gemma4VisionMLP(
            config,
            quant_config=quant_config,
            prefix=add_prefix("mlp", prefix),
        )
        eps = config.rms_norm_eps
        hs = config.hidden_size
        self.input_layernorm = Gemma4RMSNorm(hs, eps=eps)
        self.post_attention_layernorm = Gemma4RMSNorm(hs, eps=eps)
        self.pre_feedforward_layernorm = Gemma4RMSNorm(hs, eps=eps)
        self.post_feedforward_layernorm = Gemma4RMSNorm(hs, eps=eps)

        self.register_buffer("layer_scalar", torch.ones(()))

    def forward(
        self,
        hidden_states: torch.Tensor,
        cos: torch.Tensor,
        sin: torch.Tensor,
        attention_mask: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        residual = hidden_states
        hidden_states = self.input_layernorm(hidden_states)
        hidden_states = self.self_attn(hidden_states, cos, sin, attention_mask)
        hidden_states = self.post_attention_layernorm(hidden_states)
        hidden_states = residual + hidden_states
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Vision Encoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Vision Encoder Layer，用于封装该模型组件的状态与方法。

### Lines 352-392: Class: Gemma4VisionTransformer / 类：Gemma4VisionTransformer
```python
class Gemma4VisionTransformer(nn.Module):
    def __init__(
        self,
        config: Gemma4VisionConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.rotary_emb = Gemma4VisionRotaryEmbedding(config)
        self.layers = nn.ModuleList(
            [
                Gemma4VisionEncoderLayer(
                    config,
                    layer_idx=i,
                    quant_config=quant_config,
                    prefix=add_prefix(f"layers.{i}", prefix),
                )
                for i in range(config.num_hidden_layers)
            ]
        )

    def forward(
        self,
        inputs_embeds: torch.Tensor,
        attention_mask: torch.Tensor,
        patch_positions: torch.Tensor,
    ) -> torch.Tensor:
        """
        Args:
            inputs_embeds: [batch, seq, hidden_size]
            attention_mask: [batch, seq] — True = valid token
            patch_positions: [batch, seq, 2]
        Returns:
            last_hidden_state: [batch, seq, hidden_size]
        """
        cos, sin = self.rotary_emb(inputs_embeds, patch_positions)
        hidden_states = inputs_embeds
        for layer in self.layers:
            hidden_states = layer(hidden_states, cos, sin, attention_mask)
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Vision Transformer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Vision Transformer，用于封装该模型组件的状态与方法。

### Lines 400-455: Class: Gemma4VisionPatchEmbedder / 类：Gemma4VisionPatchEmbedder
```python
class Gemma4VisionPatchEmbedder(nn.Module):
    def __init__(self, config: Gemma4VisionConfig):
        super().__init__()
        self.patch_size = config.patch_size
        self.hidden_size = config.hidden_size
        self.position_embedding_size = config.position_embedding_size

        self.input_proj = nn.Linear(
            3 * self.patch_size**2, self.hidden_size, bias=False
        )
        self.position_embedding_table = nn.Parameter(
            torch.ones(2, self.position_embedding_size, self.hidden_size)
        )

    def _position_embeddings(
        self, patch_positions: torch.Tensor, padding_positions: torch.Tensor
    ) -> torch.Tensor:
        clamped_positions = patch_positions.clamp(min=0)
        one_hot = F.one_hot(clamped_positions, num_classes=self.position_embedding_size)
        one_hot = one_hot.permute(0, 2, 1, 3).to(self.position_embedding_table)
        position_embeddings = one_hot @ self.position_embedding_table
        position_embeddings = position_embeddings.sum(dim=1)
        position_embeddings = torch.where(
            padding_positions.unsqueeze(-1), 0.0, position_embeddings
        )
        return position_embeddings

    def _patch_projection(self, pixel_values: torch.Tensor) -> torch.Tensor:
        """Project pre-patchified pixels into model space.

        Args:
            pixel_values: [batch, num_patches, patch_pixels] — already patchified
                          by the image processor, values in [0, 1].
        """
        patches = 2 * (pixel_values - 0.5)
        return self.input_proj(patches.to(self.input_proj.weight.dtype))

    def forward(
        self,
        pixel_values: torch.Tensor,
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Vision Patch Embedder inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Vision Patch Embedder，用于封装该模型组件的状态与方法。

### Lines 463-517: Class: Gemma4VisionPooler / 类：Gemma4VisionPooler
```python
class Gemma4VisionPooler(nn.Module):
    def __init__(self, config: Gemma4VisionConfig):
        super().__init__()
        self.hidden_size = config.hidden_size
        self.root_hidden_size = self.hidden_size**0.5

    def _avg_pool_by_positions(
        self, x: torch.Tensor, patch_positions: torch.Tensor, length: int
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        input_seq_len = x.shape[1]
        k = int((input_seq_len // length) ** 0.5)
        k_squared = k**2
        if k_squared * length != input_seq_len:
            raise ValueError(
                f"Cannot pool {x.shape} to {length}: {k=}^2 times {length=} must be {input_seq_len}."
            )
        clamped_positions = patch_positions.clamp(min=0)
        max_x = clamped_positions[..., 0].max(dim=-1, keepdim=True)[0] + 1
        kernel_idxs = torch.div(clamped_positions, k, rounding_mode="floor")
        kernel_idxs = kernel_idxs[..., 0] + (max_x // k) * kernel_idxs[..., 1]

        weights = F.one_hot(kernel_idxs.long(), length).float() / k_squared
        output = weights.transpose(1, 2).to(x.dtype) @ x
        mask = torch.logical_not((weights == 0).all(dim=1))
        return output, mask

    def forward(
        self,
        hidden_states: torch.Tensor,
        patch_positions: torch.Tensor,
        padding_positions: torch.Tensor,
        output_length: Optional[int] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        """
        Returns:
            (pooled_hidden_states, mask) where mask is True for valid tokens.
        """
        if output_length is None:
            raise ValueError("output_length is required for Gemma4VisionPooler")
        if output_length > hidden_states.shape[1]:
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Vision Pooler inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma4 Vision Pooler，用于封装该模型组件的状态与方法。

### Lines 525-599: Class: Gemma4VisionEncoder / 类：Gemma4VisionEncoder
```python
class Gemma4VisionEncoder(nn.Module):
    """Drop-in replacement for HF ``Gemma4VisionEncoder`` with TP support."""

    def __init__(
        self,
        config: Gemma4VisionConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.patch_size = config.patch_size
        self.pooling_kernel_size = config.pooling_kernel_size

        self.patch_embedder = Gemma4VisionPatchEmbedder(config)
        self.encoder = Gemma4VisionTransformer(
            config,
            quant_config=quant_config,
            prefix=add_prefix("encoder", prefix),
        )
        self.pooler = Gemma4VisionPooler(config)

        # Post-pooling standardization (normalizes vision tokens before projection)
        self.standardize = getattr(config, "standardize", False)
        if self.standardize:
            self.register_buffer("std_bias", torch.zeros(config.hidden_size))
            self.register_buffer("std_scale", torch.ones(config.hidden_size))

    @property
    def device(self) -> torch.device:
        return self.patch_embedder.input_proj.weight.device

    def forward(
        self,
        pixel_values: torch.Tensor,
        pixel_position_ids: torch.Tensor,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        """Encode pre-patchified pixel_values into soft tokens.

        Args:
# ... truncated for brevity ...
```
**EN:** This class defines Gemma4 Vision Encoder inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Drop-in replacement for HF ``Gemma4VisionEncoder`` with TP support..
**CN:** 该类定义了 Gemma4 Vision Encoder，用于封装该模型组件的状态与方法。 文档字符串摘要：Drop-in replacement for HF ``Gemma4VisionEncoder`` with TP support.。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `__future__: annotations`
- `typing: Optional, Tuple`
- `torch`
- `torch.nn`
- `torch.nn.functional`
- `einops: rearrange`
- `transformers: Gemma4VisionConfig`
- `sglang.srt.layers.attention.vision: QKV_BACKEND_IMPL`
- `sglang.srt.layers.clippable_linear: ClippableGateUpParallelLinear, ClippableQKVParallelLinear, ClippableRowParallelLinear`
- `sglang.srt.layers.dp_attention: get_attention_tp_size`
- `sglang.srt.layers.layernorm: Gemma4RMSNorm`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.utils: add_prefix, get_device_capability, is_cuda, is_hip`
