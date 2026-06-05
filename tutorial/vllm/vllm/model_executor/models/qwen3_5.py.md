# qwen3_5.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/qwen3_5.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Qwen3 5 multimodal model adapter used for inference in vLLM. The module docstring summarizes it as: "Inference-only Qwen3.5 Series compatible with HuggingFace weights." / 实现 Qwen3 5 在 vLLM 中用于推理的多模态模型适配器。 模块文档字符串还将其概括为：“Inference-only Qwen3.5 Series compatible with HuggingFace weights。”

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-104)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

# Copyright 2025 The vLLM team.
# Copyright 2025 The Qwen Team.
# Copyright 2025 The HuggingFace Inc. team.
# All rights reserved.
#
# This code is based on EleutherAI's GPT-NeoX library and the GPT-NeoX
# and OPT implementations in this library. It has been modified from its
# original forms to accommodate minor architectural differences compared
# to GPT-NeoX and OPT used by the Meta AI team that trained the model.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
# ... omitted for brevity ...
    Qwen3VLDummyInputsBuilder,
    Qwen3VLForConditionalGeneration,
    Qwen3VLMultiModalProcessor,
    Qwen3VLProcessingInfo,
)
from .utils import (
    AutoWeightsLoader,
    PPMissingLayer,
    _merge_multimodal_embeddings,
    extract_layer_index,
    is_pp_missing_parameter,
    make_empty_intermediate_tensors_factory,
    make_layers,
    maybe_prefix,
)
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as torch supply framework primitives, while internal modules like vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.logger connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 torch 这样的外部依赖提供基础框架能力，而 vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.logger 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Top-level mapping `logger` (lines 105-105)
```python
logger = init_logger(__name__)
```
**EN:** This assignment initializes the module-level logger used for diagnostics, backend selection messages, and compatibility warnings.
**CN:** 这个赋值语句初始化了模块级日志记录器，用于输出诊断信息、后端选择消息和兼容性警告。

### Class `Qwen3_5ProcessingInfo` (lines 108-110)
```python
class Qwen3_5ProcessingInfo(Qwen3VLProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config(Qwen3_5Config)
```
**EN:** Defines `Qwen3_5ProcessingInfo`, a supporting module used by the surrounding model implementation. It inherits from Qwen3VLProcessingInfo. Key methods such as `get_hf_config` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen3_5ProcessingInfo`，它是一个被周边模型实现复用的支撑模块。 它继承自 Qwen3VLProcessingInfo。 `get_hf_config` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Qwen3_5MoeProcessingInfo` (lines 113-115)
```python
class Qwen3_5MoeProcessingInfo(Qwen3VLProcessingInfo):
    def get_hf_config(self):
        return self.ctx.get_hf_config(Qwen3_5MoeConfig)
```
**EN:** Defines `Qwen3_5MoeProcessingInfo`, a supporting module used by the surrounding model implementation. It inherits from Qwen3VLProcessingInfo. Key methods such as `get_hf_config` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen3_5MoeProcessingInfo`，它是一个被周边模型实现复用的支撑模块。 它继承自 Qwen3VLProcessingInfo。 `get_hf_config` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Qwen3_5DecoderLayer` (lines 118-193)
```python
class Qwen3_5DecoderLayer(Qwen3NextDecoderLayer):
    def __init__(
        self,
        vllm_config: VllmConfig,
        layer_type: str,
        prefix: str = "",
    ) -> None:
        super(Qwen3NextDecoderLayer, self).__init__()

        config = vllm_config.model_config.hf_text_config
        model_config = vllm_config.model_config
        cache_config = vllm_config.cache_config
        quant_config = vllm_config.quant_config

        self.layer_type = layer_type
        self.layer_idx = extract_layer_index(prefix)

        if self.layer_type == "linear_attention":
            self.linear_attn = GatedDeltaNetAttention(
                config=config,
```
**EN:** Defines `Qwen3_5DecoderLayer`, a transformer layer that stitches normalization, attention, and projection submodules together. It inherits from Qwen3NextDecoderLayer. Key methods such as `__init__` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen3_5DecoderLayer`，它是一个将归一化、注意力和投影子模块拼接起来的 Transformer 层。 它继承自 Qwen3NextDecoderLayer。 `__init__` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Qwen3_5Model` (lines 196-428)
```python
@support_torch_compile(
    dynamic_arg_dims={
        "input_ids": 0,
        # positions is of shape (3, seq_len) if mrope is enabled for qwen2-vl,
        # otherwise (seq_len, ).
        "positions": -1,
        "intermediate_tensors": 0,
        "inputs_embeds": 0,
    }
)
class Qwen3_5Model(Qwen3NextModel):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super(Qwen3NextModel, self).__init__()

        config: Qwen3_5TextConfig | Qwen3_5MoeTextConfig = (
            vllm_config.model_config.hf_text_config
        )
        parallel_config = vllm_config.parallel_config

        eplb_config = parallel_config.eplb_config
        self.num_redundant_experts = eplb_config.num_redundant_experts

        self.config = config

        self.vocab_size = config.vocab_size

        self.embed_tokens = VocabParallelEmbedding(
            self.vocab_size,
            config.hidden_size,
        )
# ... omitted for brevity ...
    def load_fused_expert_weights(
        self,
        name: str,
        params_dict: dict,
        loaded_weight: torch.Tensor,
        shard_id: str,
        num_experts: int,
    ) -> bool:
        param = params_dict[name]
        weight_loader = typing.cast(Callable[..., bool], param.weight_loader)
        loaded_local_expert = False
        for expert_id in range(num_experts):
            curr_expert_weight = loaded_weight[expert_id]
            success = weight_loader(
                param,
                curr_expert_weight,
                name,
                shard_id=shard_id,
                expert_id=expert_id,
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            # GDN
            ("in_proj_qkvz", "in_proj_qkv", (0, 1, 2)),
            ("in_proj_qkvz", "in_proj_z", 3),
            # self attention
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            # mlp
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
            ("in_proj_ba", "in_proj_b", 0),
            ("in_proj_ba", "in_proj_a", 1),
        ]

        params_dict = dict(self.named_parameters())
        loaded_params: set[str] = set()
```
**EN:** Defines `Qwen3_5Model`, a backbone model container that orchestrates embeddings, layers, and output heads. It inherits from Qwen3NextModel. Key methods such as `__init__`, `load_fused_expert_weights`, `load_weights` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen3_5Model`，它是一个协调嵌入、层堆叠与输出头的主干模型容器。 它继承自 Qwen3NextModel。 `__init__`, `load_fused_expert_weights`, `load_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Qwen3_5ForCausalLMBase` (lines 431-523)
```python
class Qwen3_5ForCausalLMBase(
    nn.Module,
    HasInnerState,
    SupportsEagle3,
    SupportsLoRA,
    SupportsPP,
):
    packed_modules_mapping = {
        "qkv_proj": [
# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        config = vllm_config.model_config.hf_text_config
        self.vllm_config = vllm_config
        self.model_config = vllm_config.model_config
        cache_config = vllm_config.cache_config

        scheduler_config = vllm_config.scheduler_config
        if cache_config.mamba_cache_mode == "all":
            raise NotImplementedError(
                "Qwen3.5 currently does not support 'all' prefix caching, "
                "please use '--mamba-cache-mode=align' instead"
            )
        self.quant_config = vllm_config.quant_config

        super().__init__()
        self.config = config
        self.scheduler_config = scheduler_config
        self.model = Qwen3_5Model(
            vllm_config=vllm_config, prefix=maybe_prefix(prefix, "model")
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ):
        hidden_states = self.model(
            input_ids, positions, intermediate_tensors, inputs_embeds
        )

        return hidden_states
# ... omitted for brevity ...
    def compute_logits(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor | None:
        return self.logits_processor(self.lm_head, hidden_states)
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loader = AutoWeightsLoader(
            self,
            skip_prefixes=["mtp."],
        )
        return loader.load_weights(weights)
```
**EN:** Defines `Qwen3_5ForCausalLMBase`, a top-level language-model wrapper that exposes logits and weight-loading behavior. It inherits from nn.Module, HasInnerState, SupportsEagle3. Key methods such as `__init__`, `embed_input_ids`, `set_aux_hidden_state_layers`, `get_eagle3_aux_hidden_state_layers`, `forward` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen3_5ForCausalLMBase`，它是一个暴露 logits 计算与权重加载行为的顶层语言模型封装类。 它继承自 nn.Module、HasInnerState、SupportsEagle3。 `__init__`, `embed_input_ids`, `set_aux_hidden_state_layers`, `get_eagle3_aux_hidden_state_layers`, `forward` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Qwen3_5ForCausalLM` (lines 526-527)
```python
class Qwen3_5ForCausalLM(Qwen3_5ForCausalLMBase):
    pass
```
**EN:** Defines `Qwen3_5ForCausalLM`, a top-level language-model wrapper that exposes logits and weight-loading behavior. It inherits from Qwen3_5ForCausalLMBase.
**CN:** 定义 `Qwen3_5ForCausalLM`，它是一个暴露 logits 计算与权重加载行为的顶层语言模型封装类。 它继承自 Qwen3_5ForCausalLMBase。

### Class `Qwen3_5MoeForCausalLM` (lines 530-538)
```python
class Qwen3_5MoeForCausalLM(Qwen3_5ForCausalLMBase, QwenNextMixtureOfExperts):
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__(vllm_config=vllm_config, prefix=prefix)

        # set MoE hyperparameters
        self.set_moe_parameters()

    def get_expert_mapping(self) -> list[tuple[str, str, int, str]]:
        return self.model.get_expert_mapping()
```
**EN:** Defines `Qwen3_5MoeForCausalLM`, a top-level language-model wrapper that exposes logits and weight-loading behavior. It inherits from Qwen3_5ForCausalLMBase, QwenNextMixtureOfExperts. Key methods such as `__init__`, `get_expert_mapping` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen3_5MoeForCausalLM`，它是一个暴露 logits 计算与权重加载行为的顶层语言模型封装类。 它继承自 Qwen3_5ForCausalLMBase、QwenNextMixtureOfExperts。 `__init__`, `get_expert_mapping` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Qwen3_5ForConditionalGeneration` (lines 546-709)
```python
@MULTIMODAL_REGISTRY.register_processor(
    Qwen3VLMultiModalProcessor,
    info=Qwen3_5ProcessingInfo,
    dummy_inputs=Qwen3VLDummyInputsBuilder,
)
class Qwen3_5ForConditionalGeneration(Qwen3VLForConditionalGeneration, IsHybrid):
    # Qwen3.5 does not support multimodal pruning (EVS).
    supports_multimodal_pruning = False

# ... omitted for brevity ...
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "model"):
        # protocols have not __init__ method, so we need to use nn.Module.__init__
        nn.Module.__init__(self)
        config: Qwen3_5Config = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config

        self.config = config
        self.model_config = vllm_config.model_config
        self.multimodal_config = multimodal_config
        self.use_data_parallel = multimodal_config.mm_encoder_tp_mode == "data"
        # Qwen3.5 does not support multimodal pruning (EVS).
        self.is_multimodal_pruning_enabled = False

        with self._mark_tower_model(vllm_config, {"image", "video"}):
            self.visual = Qwen3_VisionTransformer(
                config.vision_config,
                norm_eps=getattr(config, "rms_norm_eps", 1e-6),
                quant_config=quant_config,
# ... omitted for brevity ...
    def embed_input_ids(
        self,
        input_ids: torch.Tensor,
        multimodal_embeddings: MultiModalEmbeddings | None = None,
        *,
# ... omitted for brevity ...

        if multimodal_embeddings is None or len(multimodal_embeddings) == 0:
            return inputs_embeds

        is_multimodal = _require_is_multimodal(is_multimodal)

        inputs_embeds = _merge_multimodal_embeddings(
# ... omitted for brevity ...
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        intermediate_tensors: IntermediateTensors | None = None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ) -> torch.Tensor | IntermediateTensors:
        """Run forward pass for Qwen3.5.

        Args:
            input_ids: Flattened (concatenated) input_ids corresponding to a
                batch.
            positions: Flattened (concatenated) position ids corresponding to a
                batch.
                **NOTE**: If mrope is enabled (default setting for Qwen3VL
                opensource models), the shape will be `(3, seq_len)`,
                otherwise it will be `(seq_len,).
            intermediate_tensors: Intermediate tensors from previous pipeline
# ... omitted for brevity ...
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        loader = AutoWeightsLoader(
            self,
            skip_prefixes=["mtp."],
        )
        return loader.load_weights(weights, mapper=self.hf_to_vllm_mapper)
```
**EN:** Defines `Qwen3_5ForConditionalGeneration`, a supporting module used by the surrounding model implementation. It inherits from Qwen3VLForConditionalGeneration, IsHybrid. Key methods such as `__init__`, `embed_input_ids`, `recompute_mrope_positions`, `forward`, `load_weights` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen3_5ForConditionalGeneration`，它是一个被周边模型实现复用的支撑模块。 它继承自 Qwen3VLForConditionalGeneration、IsHybrid。 `__init__`, `embed_input_ids`, `recompute_mrope_positions`, `forward`, `load_weights` 等关键方法展示了构造、前向执行或权重适配发生的位置。

### Class `Qwen3_5MoeForConditionalGeneration` (lines 763-806)
```python
@MULTIMODAL_REGISTRY.register_processor(
    Qwen3VLMultiModalProcessor,
    info=Qwen3_5MoeProcessingInfo,
    dummy_inputs=Qwen3VLDummyInputsBuilder,
)
class Qwen3_5MoeForConditionalGeneration(
    Qwen3_5ForConditionalGeneration, Qwen3_5_MoeMixtureOfExperts
):
    # For MoE LoRA weights loading
    is_3d_moe_weight: bool = True

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "model"):
        # protocols have not __init__ method, so we need to use nn.Module.__init__
        nn.Module.__init__(self)
        config: Qwen3_5MoeConfig = vllm_config.model_config.hf_config
        quant_config = vllm_config.quant_config
        multimodal_config = vllm_config.model_config.multimodal_config

        self.config = config
        self.model_config = vllm_config.model_config
        self.multimodal_config = multimodal_config
        self.use_data_parallel = multimodal_config.mm_encoder_tp_mode == "data"
        # Qwen3.5 does not support multimodal pruning (EVS).
        self.is_multimodal_pruning_enabled = False

        with self._mark_tower_model(vllm_config, {"image", "video"}):
            self.visual = Qwen3_VisionTransformer(
                config.vision_config,
                norm_eps=getattr(config, "rms_norm_eps", 1e-6),
                quant_config=quant_config,
                prefix=maybe_prefix(prefix, "visual"),
            )

        with self._mark_language_model(vllm_config):
            self.language_model = Qwen3_5MoeForCausalLM(
                vllm_config=vllm_config, prefix=maybe_prefix(prefix, "language_model")
            )

        self.make_empty_intermediate_tensors = (
            self.language_model.make_empty_intermediate_tensors
        )

        # set MoE hyperparameters
        self.set_moe_parameters()
```
**EN:** Defines `Qwen3_5MoeForConditionalGeneration`, a supporting module used by the surrounding model implementation. It inherits from Qwen3_5ForConditionalGeneration, Qwen3_5_MoeMixtureOfExperts. Key methods such as `__init__` show where construction, forward execution, or weight adaptation happens.
**CN:** 定义 `Qwen3_5MoeForConditionalGeneration`，它是一个被周边模型实现复用的支撑模块。 它继承自 Qwen3_5ForConditionalGeneration、Qwen3_5_MoeMixtureOfExperts。 `__init__` 等关键方法展示了构造、前向执行或权重适配发生的位置。

## Key Concepts / 关键概念
- **Multimodal bridging**
  - **EN:** The file combines text decoding with image, video, audio, or OCR features before tokens are generated.
  - **CN:** 该文件在生成 token 之前把文本解码与图像、视频、音频或 OCR 特征连接起来。
- **Attention and KV cache**
  - **EN:** Core logic manages query/key/value projection and integrates with vLLM attention backends and cache layout.
  - **CN:** 核心逻辑管理 Q/K/V 投影，并与 vLLM 的注意力后端及 KV Cache 布局集成。
- **Distributed execution**
  - **EN:** The implementation is aware of tensor parallelism or pipeline parallelism, so layers and weights can be sharded across devices.
  - **CN:** 实现考虑了张量并行或流水线并行，因此层和权重可以在多设备间切分。
- **Quantized weights**
  - **EN:** Quantization hooks allow the model to load compressed checkpoints or specialized linear layers.
  - **CN:** 量化钩子使模型能够加载压缩检查点或专用线性层。
- **Checkpoint remapping**
  - **EN:** Weight-loading helpers translate Hugging Face checkpoints into the parameter names and tensor layout expected by vLLM.
  - **CN:** 权重加载辅助逻辑会把 Hugging Face 检查点转换成 vLLM 期望的参数名和张量布局。

## Dependencies / 依赖关系
- **Standard library**: typing, collections.abc
  - **EN:** Standard modules provide typing, iteration, and compatibility helpers.
  - **CN:** 标准库模块提供类型、迭代和兼容性辅助能力。
- **External libraries**: torch
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.compilation.decorators, vllm.config, vllm.distributed, vllm.logger, vllm.model_executor.layers.layernorm, vllm.model_executor.layers.logits_processor, vllm.model_executor.layers.mamba.gdn_linear_attn, vllm.model_executor.layers.mamba.mamba_utils
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .interfaces, .qwen2_moe, .qwen3_next, .qwen3_vl, .utils
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
