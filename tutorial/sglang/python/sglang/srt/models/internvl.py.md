# internvl.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/internvl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Internvl architecture into SGLang's serving runtime, typically defining layers, model wrappers, and weight-loading behavior for inference. / 该模块将 Internvl 架构接入 SGLang 服务运行时，通常定义推理所需的层、模型封装以及权重加载逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2: Module header and imports / 模块头与导入
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 51-51: Assignment: _is_cuda / 赋值：_is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 54-96: Class: InternAttention / 类：InternAttention
```python
class InternAttention(nn.Module):
    def __init__(
        self,
        config,
        quant_config: QuantizationConfig = None,
        use_data_parallel: bool = False,
        aux_stream: Optional[torch.cuda.Stream] = None,
    ):
        super().__init__()
        self.config = config
        self.embed_dim = config.hidden_size
        self.num_heads = config.num_attention_heads
        self.head_dim = self.embed_dim // self.num_heads
        self.scale = self.head_dim**-0.5

        self.attn = VisionAttention(
            embed_dim=self.embed_dim,
            num_heads=self.num_heads,
            projection_size=self.embed_dim,
            use_qkv_parallel=True,
            quant_config=quant_config,
            dropout=getattr(config, "dropout", 0.0),
            qkv_bias=getattr(config, "qkv_bias", False)
            or getattr(config, "attention_bias", False),
            num_dummy_heads=getattr(config, "num_dummy_heads", 0),
            qk_normalization=getattr(config, "qk_normalization", False)
            or getattr(config, "use_qk_norm", False),
            flatten_batch=False,
            use_data_parallel=use_data_parallel,
            aux_stream=aux_stream,
        )

        self.proj_drop = nn.Dropout(config.dropout)

    def forward(
        self,
        hidden_states: torch.Tensor,
        cu_seqlens: torch.Tensor,
        output_ws: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
# ... truncated for brevity ...
```
**EN:** This class defines Intern Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Intern Attention，用于封装该模型组件的状态与方法。

### Lines 99-170: Class: InternVisionEmbeddings / 类：InternVisionEmbeddings
```python
class InternVisionEmbeddings(nn.Module):
    def __init__(self, config: PretrainedConfig):
        super().__init__()
        self.config = config
        self.embed_dim = config.hidden_size
        self.image_size = (
            config.image_size
            if isinstance(config.image_size, int)
            else config.image_size[0]
        )
        self.patch_size = (
            config.patch_size
            if isinstance(config.patch_size, int)
            else config.patch_size[0]
        )

        self.class_embedding = nn.Parameter(
            torch.randn(1, 1, self.embed_dim),
        )

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

    def _get_pos_embed(self, pos_embed, H, W):
        target_dtype = pos_embed.dtype
        pos_embed = (
            pos_embed.float()
            .reshape(
                1,
# ... truncated for brevity ...
```
**EN:** This class defines Intern Vision Embeddings inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Intern Vision Embeddings，用于封装该模型组件的状态与方法。

### Lines 173-184: Class: InternRMSNorm / 类：InternRMSNorm
```python
class InternRMSNorm(nn.Module):
    def __init__(self, hidden_size, eps=1e-6):
        super().__init__()
        self.weight = nn.Parameter(torch.ones(hidden_size))
        self.variance_epsilon = eps

    def forward(self, hidden_states):
        input_dtype = hidden_states.dtype
        hidden_states = hidden_states.to(torch.float32)
        variance = hidden_states.pow(2).mean(-1, keepdim=True)
        hidden_states = hidden_states * torch.rsqrt(variance + self.variance_epsilon)
        return self.weight * hidden_states.to(input_dtype)
```
**EN:** This class defines Intern R M S Norm inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Intern R M S Norm，用于封装该模型组件的状态与方法。

### Lines 187-221: Class: InternMLP / 类：InternMLP
```python
class InternMLP(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        use_data_parallel: bool = False,
    ):
        super().__init__()
        self.tp_size = (
            1 if use_data_parallel else get_tensor_model_parallel_world_size()
        )
        self.tp_rank = 0 if use_data_parallel else get_tensor_model_parallel_rank()
        self.config = config
        self.act = get_act_fn(config.hidden_act)
        self.fc1 = ColumnParallelLinear(
            config.hidden_size,
            config.intermediate_size,
            bias=True,
            quant_config=None,
            tp_size=self.tp_size,
            tp_rank=self.tp_rank,
        )
        self.fc2 = RowParallelLinear(
            config.intermediate_size,
            config.hidden_size,
            bias=True,
            quant_config=None,
            tp_size=self.tp_size,
            tp_rank=self.tp_rank,
        )

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states, _ = self.fc1(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states, _ = self.fc2(hidden_states)
        return hidden_states
```
**EN:** This class defines Intern M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Intern M L P，用于封装该模型组件的状态与方法。

### Lines 224-227: Assignment: NORM2FN / 赋值：NORM2FN
```python
NORM2FN = {
    "rms_norm": InternRMSNorm,
    "layer_norm": nn.LayerNorm,
}
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 230-291: Class: InternVisionEncoderLayer / 类：InternVisionEncoderLayer
```python
class InternVisionEncoderLayer(nn.Module):

    def __init__(
        self,
        config: PretrainedConfig,
        drop_path_rate: float,
        quant_config: QuantizationConfig = None,
        use_data_parallel: bool = False,
        aux_stream: Optional[torch.cuda.Stream] = None,
    ):
        super().__init__()
        self.embed_dim = config.hidden_size
        self.intermediate_size = config.intermediate_size
        self.norm_type = config.norm_type
        self.attn = InternAttention(
            config=config,
            quant_config=quant_config,
            use_data_parallel=use_data_parallel,
            aux_stream=aux_stream,
        )
        self.mlp = InternMLP(config, use_data_parallel)
        self.norm1 = NORM2FN[self.norm_type](self.embed_dim, eps=config.layer_norm_eps)
        self.norm2 = NORM2FN[self.norm_type](self.embed_dim, eps=config.layer_norm_eps)

        self.ls1 = nn.Parameter(config.initializer_factor * torch.ones(self.embed_dim))
        self.ls2 = nn.Parameter(config.initializer_factor * torch.ones(self.embed_dim))
        self.drop_path1 = (
            DropPath(drop_path_rate) if drop_path_rate > 0.0 else nn.Identity()
        )
        self.drop_path2 = (
            DropPath(drop_path_rate) if drop_path_rate > 0.0 else nn.Identity()
        )

    def forward(
        self,
        hidden_states: torch.Tensor,
        cu_seqlens: torch.Tensor,
        output_ws: Optional[torch.Tensor] = None,
    ) -> Tuple[
        torch.FloatTensor,
# ... truncated for brevity ...
```
**EN:** This class defines Intern Vision Encoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Intern Vision Encoder Layer，用于封装该模型组件的状态与方法。

### Lines 294-388: Class: InternVisionEncoder / 类：InternVisionEncoder
```python
class InternVisionEncoder(nn.Module):
    """
    Transformer encoder consisting of `config.num_hidden_layers` self attention layers. Each layer is a
    [`InternEncoderLayer`].

    Args:
        config (`InternConfig`):
            The corresponding vision configuration for the `InternEncoder`.
    """

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        use_data_parallel: bool = False,
    ):
        super().__init__()
        self.config = config
        # stochastic depth decay rule
        dpr = [
            x.item()
            for x in torch.linspace(0, config.drop_path_rate, config.num_hidden_layers)
        ]

        self.enable_cg = _is_cuda and envs.SGLANG_VIT_ENABLE_CUDA_GRAPH.get()
        aux_stream = (
            None if self.enable_cg else (torch.cuda.Stream() if _is_cuda else None)
        )
        self.layers = nn.ModuleList(
            [
                InternVisionEncoderLayer(
                    config, dpr[idx], quant_config, use_data_parallel, aux_stream
                )
                for idx in range(config.num_hidden_layers)
            ]
        )

        self.cuda_graph_runner: Optional[InternViTCudaGraphRunner] = None
        if self.enable_cg:
            self.cuda_graph_runner = InternViTCudaGraphRunner(self)
# ... truncated for brevity ...
```
**EN:** This class defines Intern Vision Encoder inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Transformer encoder consisting of `config.num_hidden_layers` self attention layers. Each layer is a.
**CN:** 该类定义了 Intern Vision Encoder，用于封装该模型组件的状态与方法。 文档字符串摘要：Transformer encoder consisting of `config.num_hidden_layers` self attention layers. Each layer is a。

### Lines 391-494: Class: InternVisionModel / 类：InternVisionModel
```python
class InternVisionModel(PreTrainedModel):
    main_input_name = "pixel_values"
    _supports_flash_attn_2 = True
    config_class = PretrainedConfig
    _no_split_modules = ["InternVisionEncoderLayer"]

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        use_data_parallel: bool = False,
    ):
        super().__init__(config)

        self.config = config
        self.use_data_parallel = use_data_parallel
        self.embeddings = InternVisionEmbeddings(
            config,
        )
        self.encoder = InternVisionEncoder(config, quant_config, use_data_parallel)

    def resize_pos_embeddings(self, old_size, new_size, patch_size):
        pos_emb = self.embeddings.position_embedding
        _, num_positions, embed_dim = pos_emb.shape
        cls_emb = pos_emb[:, :1, :]
        pos_emb = (
            pos_emb[:, 1:, :]
            .reshape(1, old_size // patch_size, old_size // patch_size, -1)
            .permute(0, 3, 1, 2)
        )
        pos_emb = F.interpolate(
            pos_emb.float(),
            size=new_size // patch_size,
            mode="bicubic",
            align_corners=False,
        )
        pos_emb = pos_emb.to(cls_emb.dtype).reshape(1, embed_dim, -1).permute(0, 2, 1)
        pos_emb = torch.cat([cls_emb, pos_emb], dim=1)
        self.embeddings.position_embedding = nn.Parameter(pos_emb)
        self.embeddings.image_size = new_size
# ... truncated for brevity ...
```
**EN:** This class defines Intern Vision Model inheriting from PreTrainedModel, grouping state and methods for this model component.
**CN:** 该类定义了 Intern Vision Model，用于封装该模型组件的状态与方法。

### Lines 497-792: Class: InternVLChatModel / 类：InternVLChatModel
```python
class InternVLChatModel(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        use_flash_attn=True,
    ) -> None:
        super().__init__()
        self.config = config
        self.use_data_parallel = get_global_server_args().mm_enable_dp_encoder
        self.quant_config = quant_config
        vision_utils.update_vit_attn_dummy_heads_config(self.config)
        image_size = config.force_image_size or config.vision_config.image_size
        patch_size = config.vision_config.patch_size
        self.patch_size = patch_size
        self.select_layer = config.select_layer
        self.template = config.template
        self.num_image_token = int(
            (image_size // patch_size) ** 2 * (config.downsample_ratio**2)
        )
        self.downsample_ratio = config.downsample_ratio
        self.ps_version = config.ps_version

        config.vision_config.use_flash_attn = True if use_flash_attn else False
        config.llm_config._attn_implementation = (
            "flash_attention_2" if use_flash_attn else "eager"
        )

        logger.info(f"num_image_token: {self.num_image_token}")
        logger.info(f"ps_version: {self.ps_version}")

        self.vision_model = InternVisionModel(
            config.vision_config,
            use_data_parallel=self.use_data_parallel,
        )
        if config.llm_config.architectures[0] == "Qwen2ForCausalLM":
            self.language_model = Qwen2ForCausalLM(
                config=config.llm_config, quant_config=quant_config
            )
        elif config.llm_config.architectures[0] == "InternLM2ForCausalLM":
# ... truncated for brevity ...
```
**EN:** This class defines Intern V L Chat Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Intern V L Chat Model，用于封装该模型组件的状态与方法。

### Lines 795-795: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = InternVLChatModel
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `typing: Iterable, List, Optional, Tuple, Union`
- `torch`
- `torch.nn.functional`
- `torch: nn`
- `transformers: PretrainedConfig, PreTrainedModel`
- `transformers.modeling_outputs: BaseModelOutput, BaseModelOutputWithPooling`
- `sglang.srt.distributed: get_tensor_model_parallel_rank, get_tensor_model_parallel_world_size`
- `sglang.srt.environ: envs`
- `sglang.srt.layers.activation: get_act_fn`
- `sglang.srt.layers.attention: vision_utils`
- `sglang.srt.layers.attention.vision: SingletonCache, VisionAttention`
- `sglang.srt.layers.conv: Conv2dLayer`
- `sglang.srt.layers.linear: ColumnParallelLinear, RowParallelLinear`
- `sglang.srt.layers.moe.fused_moe_triton.layer: FusedMoE`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.managers.mm_utils: MultiModalityDataPaddingPatternTokenPairs, general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch: Modality, MultimodalDataItem, MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.model_loader.weight_utils: default_weight_loader`
- `sglang.srt.models.deepseek_janus_pro: DropPath`
