# kimi_k25.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/kimi_k25.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Kimi k25 architecture into SGLang's serving runtime, typically defining layers, model wrappers, and weight-loading behavior for inference. / 该模块将 Kimi k25 架构接入 SGLang 服务运行时，通常定义推理所需的层、模型封装以及权重加载逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module header and imports / 模块头与导入
```python
import logging
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 20-26: Error handling block: optional dependency guard / 异常处理块：optional dependency guard
```python
try:
    from transformers.activations import PytorchGELUTanh
except ImportError:
    from transformers.activations import GELUTanh

    activations.PytorchGELUTanh = GELUTanh
    PytorchGELUTanh = GELUTanh
```
**EN:** This block guards optional functionality and keeps the module resilient when dependencies are unavailable.
**CN:** 该代码块通过异常保护可选功能，以便在依赖缺失时保持模块可用。

### Lines 46-46: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 50-50: Assignment: _is_npu / 赋值：_is_npu
```python
_is_npu = is_npu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 53-71: Function: apply_rope() / 函数：apply_rope()
```python
def apply_rope(
    xq: torch.Tensor, xk: torch.Tensor, freqs_cis: torch.Tensor, x_shape=None
) -> tuple[torch.Tensor, torch.Tensor]:
    """
    Args: (The leading dimensions of all inputs should be the same)
        xq: query, tensor of shape (..., num_heads, head_dim)
        xk: key, tensor of shape (..., num_heads, head_dim)
        freqs_cis: tensor of shape (..., head_dim/2), dtype=torch.complex64. It contains the precomputed cis(freqs) for each position in the 2D grid.
    Returns:
        xq_out, xk_out: tensors of shape (..., num_heads, head_dim)
    """

    freqs_cis = freqs_cis.unsqueeze(-2)  # ..., 1, head_dim/2
    # ..., num_heads, head_dim/2
    xq_ = torch.view_as_complex(xq.float().view(*xq.shape[:-1], -1, 2))
    xk_ = torch.view_as_complex(xk.float().view(*xq.shape[:-1], -1, 2))
    xq_out = torch.view_as_real(xq_ * freqs_cis).flatten(-2)  # ..., num_heads, head_dim
    xk_out = torch.view_as_real(xk_ * freqs_cis).flatten(-2)  # ..., num_heads, head_dim
    return xq_out.type_as(xq), xk_out.type_as(xk)
```
**EN:** This function implements apply rope for the surrounding model/runtime logic. Key parameters include xq, xk, freqs_cis, x_shape.
**CN:** 该函数实现了 apply rope 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 74-101: Function: tpool_patch_merger() / 函数：tpool_patch_merger()
```python
def tpool_patch_merger(
    x: torch.Tensor,
    grid_thws: torch.Tensor,
    merge_kernel_size: tuple[int, int] = (2, 2),
) -> list[torch.Tensor]:
    d_model = x.size(-1)

    outputs = []
    pre_sum = 0
    for t, h, w in grid_thws.tolist():
        # Get the current sequence
        seq = x[pre_sum : pre_sum + t * h * w]
        # Reshape along self.merge_kernel_size and concat to the last dimension
        kernel_height, kernel_width = merge_kernel_size
        new_height, new_width = h // kernel_height, w // kernel_width
        reshaped_seq = seq.view(
            t, new_height, kernel_height, new_width, kernel_width, d_model
        )
        reshaped_seq = (
            reshaped_seq.permute(0, 1, 3, 2, 4, 5).contiguous().mean(dim=0)
        )  # temporal pooling
        padded_seq = reshaped_seq.view(
            new_height * new_width, kernel_height * kernel_width, -1
        )
        outputs.append(padded_seq)
        pre_sum += t * h * w

    return outputs
```
**EN:** This function implements tpool patch merger for the surrounding model/runtime logic. Key parameters include x, grid_thws, merge_kernel_size.
**CN:** 该函数实现了 tpool patch merger 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 104-171: Class: MoonViTEncoderLayer / 类：MoonViTEncoderLayer
```python
class MoonViTEncoderLayer(nn.Module):

    def __init__(
        self,
        num_heads: int,
        hidden_dim: int,
        mlp_dim: int,
        *,
        activation=F.gelu,
        attn_bias: bool = False,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        use_data_parallel: bool = False,
    ):
        super().__init__()
        self.num_heads = num_heads
        self.hidden_dim = hidden_dim
        self.hidden_size_per_attention_head = self.hidden_dim // self.num_heads

        self.norm0 = nn.LayerNorm(hidden_dim)
        self.norm1 = nn.LayerNorm(hidden_dim)

        self.mlp = MLP2(
            [hidden_dim, mlp_dim, hidden_dim],
            activation,
            quant_config=quant_config,
            prefix=add_prefix("mlp", prefix),
        )

        self.attn = VisionAttention(
            embed_dim=hidden_dim,
            num_heads=num_heads,
            projection_size=hidden_dim,
            use_qkv_parallel=True,
            qkv_bias=attn_bias,
            proj_bias=attn_bias,
            flatten_batch=True,
            quant_config=quant_config,
            prefix=add_prefix("attn", prefix),
            use_data_parallel=use_data_parallel,
# ... truncated for brevity ...
```
**EN:** This class defines Moon Vi T Encoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Moon Vi T Encoder Layer，用于封装该模型组件的状态与方法。

### Lines 174-184: Function: get_rope_shape_decorate() / 函数：get_rope_shape_decorate()
```python
def get_rope_shape_decorate(func):
    _get_rope_shape_first_call_flag = set()

    def wrapper(org, interpolation_mode, shape):
        key = (org.requires_grad, torch.is_grad_enabled(), interpolation_mode)
        if key not in _get_rope_shape_first_call_flag:
            _get_rope_shape_first_call_flag.add(key)
            _ = func(org, interpolation_mode, shape=(64, 64))
        return func(org, interpolation_mode, shape)

    return wrapper
```
**EN:** This function implements get rope shape decorate for the surrounding model/runtime logic. Key parameters include func.
**CN:** 该函数实现了 get rope shape decorate 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 187-207: Function: get_1d_sincos_pos_embed_from_grid() / 函数：get_1d_sincos_pos_embed_from_grid()
```python
def get_1d_sincos_pos_embed_from_grid(embed_dim, pos):
    """
    From:
    https://github.com/OpenGVLab/InternVideo/blob/421f6d2361fc8f61a3394244571f2601a4e99e29/InternVideo2/multi_modality/models/backbones/internvideo2/pos_embed.py#L86
    embed_dim: output dimension for each position
    pos: a list of positions to be encoded: size (M,)
    out: (M, D)
    """
    assert embed_dim % 2 == 0
    omega = np.arange(embed_dim // 2, dtype=np.float32)
    omega /= embed_dim / 2.0
    omega = 1.0 / 10000**omega  # (D/2,)

    pos = pos.reshape(-1)  # (M,)
    out = np.einsum("m,d->md", pos, omega)  # (M, D/2), outer product

    emb_sin = np.sin(out)  # (M, D/2)
    emb_cos = np.cos(out)  # (M, D/2)

    emb = np.concatenate([emb_sin, emb_cos], axis=1)  # (M, D)
    return emb
```
**EN:** This function implements get 1d sincos pos embed from grid for the surrounding model/runtime logic. Key parameters include embed_dim, pos.
**CN:** 该函数实现了 get 1d sincos pos embed from grid 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 212-222: Function: get_rope_shape() / 函数：get_rope_shape()
```python
def get_rope_shape(org, interpolation_mode, shape):
    return (
        F.interpolate(
            org.permute((2, 0, 1)).unsqueeze(0),
            size=shape,
            mode=interpolation_mode,
        )
        .squeeze(0)
        .permute((1, 2, 0))
        .flatten(end_dim=1)
    )
```
**EN:** This function implements get rope shape for the surrounding model/runtime logic. Key parameters include org, interpolation_mode, shape.
**CN:** 该函数实现了 get rope shape 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 225-235: Function: get_1d_sincos_pos_embed() / 函数：get_1d_sincos_pos_embed()
```python
def get_1d_sincos_pos_embed(embed_dim, t_size, cls_token=False):
    """
    t_size: int of the temporal size
    return:
    pos_embed: [t_size, embed_dim] or [1+t_size, embed_dim] (w/ or w/o cls_token)
    """
    grid_t = np.arange(t_size, dtype=np.float32)
    pos_embed = get_1d_sincos_pos_embed_from_grid(embed_dim, grid_t)
    if cls_token:
        pos_embed = np.concatenate([np.zeros([1, embed_dim]), pos_embed], axis=0)
    return pos_embed
```
**EN:** This function implements get 1d sincos pos embed for the surrounding model/runtime logic. Key parameters include embed_dim, t_size, cls_token.
**CN:** 该函数实现了 get 1d sincos pos embed 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 238-291: Class: Learnable2DInterpPosEmbDivided_fixed / 类：Learnable2DInterpPosEmbDivided_fixed
```python
class Learnable2DInterpPosEmbDivided_fixed(nn.Module):

    def __init__(
        self,
        height: int,
        width: int,
        num_frames: int,
        dim: int,
        interpolation_mode: str = "bicubic",
    ) -> None:
        super().__init__()
        self.height = height
        self.width = width
        self.num_frames = num_frames
        self.dim = dim
        self.interpolation_mode = interpolation_mode
        self.weight = nn.Parameter(torch.empty(height, width, dim))
        self.register_buffer(
            "time_weight",
            torch.from_numpy(get_1d_sincos_pos_embed(self.dim, self.num_frames))
            .float()
            .unsqueeze(1),
            persistent=False,
        )

        self.reset_parameters()

    def reset_parameters(self):
        nn.init.normal_(self.weight)

    def forward(self, x: torch.Tensor, grid_thws: torch.Tensor) -> torch.Tensor:
        pos_embs = []
        for t, h, w in grid_thws.tolist():
            assert t <= self.num_frames, f"t:{t} > self.num_frames:{self.num_frames}"
            if (h, w) == self.weight.shape[:-1]:
                pos_emb_2d = self.weight.flatten(end_dim=1)
            else:
                pos_emb_2d = get_rope_shape(
                    self.weight,
                    interpolation_mode=self.interpolation_mode,
# ... truncated for brevity ...
```
**EN:** This class defines Learnable2 D Interp Pos Emb Divided fixed inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Learnable2 D Interp Pos Emb Divided fixed，用于封装该模型组件的状态与方法。

### Lines 294-379: Class: Rope2DPosEmbRepeated / 类：Rope2DPosEmbRepeated
```python
class Rope2DPosEmbRepeated(nn.Module):
    """2D rotary position embedding with multi-resolution support.
    This class is intended to be used in the following way:
    1. Before training, create an instance of Rope2DPosEmb. This instance will hold the precomputed cis.
    2. Before each forward pass, call `get_freqs_cis_by_*` to get the `freqs_cis` tensor for this iteration.
    3. During the forward pass, pass the `freqs_cis` tensor to each attention layer, and call `apply` just before each attention operation.
        The rope is shared across all attention layers and all heads.
    Refs:
    - RoFormer: https://arxiv.org/abs/2104.09864
    - VisionLLaMA: https://arxiv.org/abs/2403.00522
    - https://github.com/Meituan-AutoML/VisionLLaMA/blob/main/dit/models.py
    Args:
        dim (int): usually the multi-head attention dimension, should be divisible by 4 (TODO: relax this constraint if needed)
        max_height (int): the maximum height of the 2D grid
        max_width (int): the maximum width of the 2D grid
        theta_base (float): the base of the theta
    """

    def __init__(self, dim: int, max_height: int, max_width: int, theta_base=10000):
        super().__init__()
        self.dim = dim
        assert self.dim % 4 == 0, "dim must be divisible by 4"
        self.max_height = max_height
        self.max_width = max_width
        self.theta_base = theta_base

    def extra_repr(self):
        return f"dim={self.dim}, max_height={self.max_height}, max_width={self.max_width}, theta_base={self.theta_base}"

    def _precompute_freqs_cis(self, device: torch.device) -> torch.Tensor:
        """Calculate the cis(freqs) for each position in the 2D grid.
        Return: complex tensor of shape (max_height, max_width, dim//2) and value:
            height axis: ret[h, w, 2*i] = cis(h * theta_base**(-4*i/dim))
            weight axis: ret[h, w, 2*i+1] = cis(w * theta_base**(-4*i/dim))   with (i in [0, dim//4))
            note: `cis` is a mathematical notation defined by cis x = cos x + i sin x,
        """
        N = self.max_height * self.max_width
        flat_pos = torch.arange(0, N).float().to(device)
        x_pos = flat_pos % self.max_width
        y_pos = flat_pos // self.max_width
# ... truncated for brevity ...
```
**EN:** This class defines Rope2 D Pos Emb Repeated inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: 2D rotary position embedding with multi-resolution support..
**CN:** 该类定义了 Rope2 D Pos Emb Repeated，用于封装该模型组件的状态与方法。 文档字符串摘要：2D rotary position embedding with multi-resolution support.。

### Lines 382-430: Class: MoonVision3dPatchEmbed / 类：MoonVision3dPatchEmbed
```python
class MoonVision3dPatchEmbed(nn.Module):

    def __init__(
        self,
        out_dim: int,
        in_dim: int = 3,
        patch_size: int | tuple[int, int] = (14, 14),
        pos_emb_height: int = 14,
        pos_emb_width: int = 14,
        pos_emb_time: int = 4,
        pos_emb_type: str = "divided_fixed",
    ):
        super().__init__()
        assert isinstance(
            patch_size, int | Sequence
        ), f"Invalid patch_size type: {type(patch_size)}"
        if isinstance(patch_size, int):
            patch_size = (patch_size, patch_size)
        assert (
            len(patch_size) == 2
        ), f"Expected patch_size to be a tuple of 2, got {patch_size}"
        self.patch_size = patch_size

        self.proj = Conv2dLayer(
            in_dim, out_dim, kernel_size=patch_size, stride=patch_size
        )

        if pos_emb_type == "divided_fixed":
            self.pos_emb = Learnable2DInterpPosEmbDivided_fixed(
                height=pos_emb_height,
                width=pos_emb_width,
                num_frames=pos_emb_time,
                dim=out_dim,
            )
        else:
            raise NotImplementedError(f"Not support pos_emb_type: {pos_emb_type}")

    def forward(self, x: torch.Tensor, grid_thws: torch.Tensor) -> torch.Tensor:
        """
        Args:
# ... truncated for brevity ...
```
**EN:** This class defines Moon Vision3d Patch Embed inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Moon Vision3d Patch Embed，用于封装该模型组件的状态与方法。

### Lines 433-491: Class: MoonViT3dEncoder / 类：MoonViT3dEncoder
```python
class MoonViT3dEncoder(nn.Module):

    def __init__(
        self,
        hidden_dim: int,
        num_layers: int,
        block_cfg: dict,
        video_attn_type: str = "spatial_temporal",
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        assert (
            video_attn_type == "spatial_temporal"
        ), f'video_attn_type must be "spatial_temporal", got {video_attn_type}'
        self.video_attn_type = video_attn_type
        self.rope_2d = Rope2DPosEmbRepeated(
            block_cfg["hidden_dim"] // block_cfg["num_heads"], 512, 512
        )
        self.blocks = nn.ModuleList(
            [
                MoonViTEncoderLayer(
                    **block_cfg,
                    quant_config=quant_config,
                    prefix=add_prefix(f"blocks.{layer_idx}", prefix),
                )
                for layer_idx in range(num_layers)
            ]
        )
        self.final_layernorm = nn.LayerNorm(hidden_dim)

    def forward(
        self,
        hidden_states: torch.Tensor,
        grid_thws: torch.Tensor,
    ) -> torch.Tensor:
        rope_freqs_cis = self.rope_2d.get_freqs_cis(
            grid_thws=grid_thws, device=hidden_states.device
        )
# ... truncated for brevity ...
```
**EN:** This class defines Moon Vi T3d Encoder inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Moon Vi T3d Encoder，用于封装该模型组件的状态与方法。

### Lines 494-569: Class: MoonViT3dPretrainedModel / 类：MoonViT3dPretrainedModel
```python
class MoonViT3dPretrainedModel(nn.Module):
    model_type = "moonvit3d"
    _no_split_modules = ["PackingTransformer"]
    _supports_flash_attn_2 = True
    _supports_sdpa = True

    def __init__(
        self,
        config,
        *inputs,
        use_data_parallel: bool = False,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        **kwargs,
    ):
        super().__init__()
        config = deepcopy(config)
        self.config = config
        self.merge_kernel_size = config.merge_kernel_size
        self.patch_size = config.patch_size
        self.merge_type = config.merge_type

        self.patch_embed = MoonVision3dPatchEmbed(
            out_dim=config.hidden_size,
            patch_size=config.patch_size,
            pos_emb_height=config.init_pos_emb_height,
            pos_emb_width=config.init_pos_emb_width,
            pos_emb_time=config.init_pos_emb_time,
            pos_emb_type=config.pos_emb_type,
        )

        self.encoder = MoonViT3dEncoder(
            hidden_dim=config.hidden_size,
            num_layers=config.num_hidden_layers,
            block_cfg={
                "num_heads": config.num_attention_heads,
                "hidden_dim": config.hidden_size,
                "mlp_dim": config.intermediate_size,
                "activation": PytorchGELUTanh(),
                "attn_bias": True,
# ... truncated for brevity ...
```
**EN:** This class defines Moon Vi T3d Pretrained Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Moon Vi T3d Pretrained Model，用于封装该模型组件的状态与方法。

### Lines 572-606: Class: K2VLMultiModalProjector / 类：K2VLMultiModalProjector
```python
class K2VLMultiModalProjector(nn.Module):
    """Multi-modal projector with patch merging for K2-VL."""

    def __init__(
        self,
        config: KimiK25VisionConfig,
        prefix: str = "",
    ):
        super().__init__()

        # Hidden size after patch merging
        merge_h, merge_w = config.merge_kernel_size
        self.hidden_size = config.vt_hidden_size * merge_h * merge_w

        self.pre_norm = torch.nn.LayerNorm(config.vt_hidden_size, eps=1e-5)
        self.linear_1 = ReplicatedLinear(
            self.hidden_size,
            self.hidden_size,
            bias=True,
            prefix=add_prefix(prefix, "linear_1"),
        )
        self.linear_2 = ReplicatedLinear(
            self.hidden_size,
            config.text_hidden_size,
            bias=True,
            prefix=add_prefix(prefix, "linear_2"),
        )
        self.act = nn.GELU()

    def forward(self, image_features: torch.Tensor) -> torch.Tensor:
        hidden_states = self.pre_norm(image_features).view(-1, self.hidden_size)
        hidden_states, _ = self.linear_1(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states, _ = self.linear_2(hidden_states)
        return hidden_states
```
**EN:** This class defines K2 V L Multi Modal Projector inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Multi-modal projector with patch merging for K2-VL..
**CN:** 该类定义了 K2 V L Multi Modal Projector，用于封装该模型组件的状态与方法。 文档字符串摘要：Multi-modal projector with patch merging for K2-VL.。

### Lines 610-622: Function: mm_projection_auto() / 函数：mm_projection_auto()
```python
def mm_projection_auto(
    mm_projector: torch.nn.Module | None, vt_output: list[torch.Tensor]
):
    """Apply MM projector to vision tower outputs."""
    if mm_projector is None:
        return vt_output

    num_embedding_list = [x.shape[0] for x in vt_output]
    batched = torch.cat(vt_output, dim=0)
    proj_out = mm_projector(batched) if mm_projector else batched
    proj_out = proj_out.reshape(-1, proj_out.shape[-1])
    proj_out = torch.split(proj_out, num_embedding_list)
    return proj_out
```
**EN:** This function implements mm projection auto for the surrounding model/runtime logic. Key parameters include mm_projector, vt_output.
**CN:** 该函数实现了 mm projection auto 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 625-856: Class: KimiK25ForConditionalGeneration / 类：KimiK25ForConditionalGeneration
```python
class KimiK25ForConditionalGeneration(nn.Module):
    # Support nvidia/Kimi-K2.5-NVFP4 naming: language_model.layers.*.
    # Ref: HF config.json for nvidia/Kimi-K2.5-NVFP4
    # https://huggingface.co/nvidia/Kimi-K2.5-NVFP4/blob/main/config.json
    hf_to_sglang_mapper = WeightsMapper(
        orig_to_new_prefix={
            "language_model.layers.": "language_model.model.layers.",
        }
    )

    def __init__(
        self,
        config: KimiK25Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        **kwargs,  # fix init_tts argument error
    ) -> None:
        super().__init__()
        self.config = config
        self.quant_config = quant_config
        self.use_data_parallel = get_global_server_args().mm_enable_dp_encoder
        # Create vision tower
        self.vision_tower = MoonViT3dPretrainedModel(
            config.vision_config,
            use_data_parallel=self.use_data_parallel,
            quant_config=(
                quant_config if isinstance(quant_config, ModelSlimConfig) else None
            ),
            prefix="vision_tower",
        )
        # Create mm projector
        self.mm_projector = K2VLMultiModalProjector(config.vision_config)

        self.language_model = None
        if not config.encoder_only:
            self.language_model = DeepseekV3ForCausalLM(
                config.text_config,
                quant_config,
                prefix=(
                    "language_model"
# ... truncated for brevity ...
```
**EN:** This class defines Kimi K25 For Conditional Generation inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Kimi K25 For Conditional Generation，用于封装该模型组件的状态与方法。

### Lines 859-859: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [KimiK25ForConditionalGeneration]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `logging`
- `copy: deepcopy`
- `typing: Iterable, List, Optional, Sequence, Tuple`
- `numpy`
- `torch`
- `torch.nn.functional`
- `torch: nn`
- `transformers: activations`
- `sglang.srt.configs.kimi_k25: KimiK25Config, KimiK25VisionConfig`
- `sglang.srt.eplb.expert_location: ModelConfigForExpertLocation`
- `sglang.srt.layers.conv: Conv2dLayer`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.managers.mm_utils: MultiModalityDataPaddingPatternMultimodalTokens, general_mm_embed_routine`
- `sglang.srt.layers.attention.vision: VisionAttention`
- `sglang.srt.layers.linear: ReplicatedLinear`
- `sglang.srt.layers.quantization.modelslim.modelslim: ModelSlimConfig`
- `sglang.srt.layers.quantization.quark.quark: QuarkConfig`
- `sglang.srt.managers.schedule_batch: Modality, MultimodalDataItem, MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch, PPProxyTensors`
- `sglang.srt.model_loader.weight_utils: default_weight_loader`
