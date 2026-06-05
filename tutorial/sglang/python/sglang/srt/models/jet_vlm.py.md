# jet_vlm.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/jet_vlm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Jet Vlm architecture into SGLang's serving runtime, typically defining layers, model wrappers, and weight-loading behavior for inference. / 该模块将 Jet Vlm 架构接入 SGLang 服务运行时，通常定义推理所需的层、模型封装以及权重加载逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module header and imports / 模块头与导入
```python
import math
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 27-27: Assignment: MM_HIDDEN_SIZE / 赋值：MM_HIDDEN_SIZE
```python
MM_HIDDEN_SIZE = 1152
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 30-45: Class: JetVLMDownSample2x2BlockFix / 类：JetVLMDownSample2x2BlockFix
```python
class JetVLMDownSample2x2BlockFix(nn.Module):
    def forward(self, x: Tensor) -> Tensor:
        _, seq_len, _ = x.shape

        feat_size = math.isqrt(seq_len)

        features = einops.rearrange(x, "b (h w) d -> b h w d", h=feat_size, w=feat_size)

        if feat_size % 2 == 1:
            features = F.pad(features, (0, 0, 0, 1, 0, 1))

        features = einops.rearrange(
            features, "b (h p1) (w p2) d -> b (h w) (p1 p2 d)", p1=2, p2=2
        )

        return features
```
**EN:** This class defines Jet V L M Down Sample2x2 Block Fix inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Jet V L M Down Sample2x2 Block Fix，用于封装该模型组件的状态与方法。

### Lines 48-61: Class: JetVLMMultiModalProjector / 类：JetVLMMultiModalProjector
```python
class JetVLMMultiModalProjector(nn.Module):
    def __init__(self, config: JetVLMConfig) -> None:
        super().__init__()

        self.layers = nn.Sequential(
            JetVLMDownSample2x2BlockFix(),
            nn.LayerNorm(MM_HIDDEN_SIZE * 4),
            nn.Linear(MM_HIDDEN_SIZE * 4, config.text_config.hidden_size),
            nn.GELU(),
            nn.Linear(config.text_config.hidden_size, config.text_config.hidden_size),
        )

    def forward(self, x: Tensor) -> Tensor:
        return self.layers(x)
```
**EN:** This class defines Jet V L M Multi Modal Projector inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Jet V L M Multi Modal Projector，用于封装该模型组件的状态与方法。

### Lines 64-140: Class: JetVLMForConditionalGeneration / 类：JetVLMForConditionalGeneration
```python
class JetVLMForConditionalGeneration(nn.Module):
    def __init__(
        self,
        config: JetVLMConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()

        self.config = config

        self.vision_tower = SiglipVisionModel(config.vision_config)
        self.mm_projector = JetVLMMultiModalProjector(config)
        self.llm = JetNemotronForCausalLM(
            config=config.text_config,
            quant_config=quant_config,
            prefix=utils.add_prefix("llm", prefix),
        )

    def forward(
        self,
        input_ids: Tensor,
        positions: Tensor,
        forward_batch: ForwardBatch,
        get_embedding: bool = False,
    ) -> LogitsProcessorOutput:
        output = mm_utils.general_mm_embed_routine(
            input_ids=input_ids,
            forward_batch=forward_batch,
            language_model=self.llm,
            data_embedding_funcs={
                Modality.IMAGE: self.get_image_feature,
                Modality.VIDEO: self.get_image_feature,
            },
            get_embedding=get_embedding,
            positions=positions,
        )

        assert isinstance(output, LogitsProcessorOutput)

# ... truncated for brevity ...
```
**EN:** This class defines Jet V L M For Conditional Generation inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Jet V L M For Conditional Generation，用于封装该模型组件的状态与方法。

### Lines 143-143: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [JetVLMForConditionalGeneration]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `math`
- `collections.abc: Iterable`
- `einops`
- `torch`
- `torch.nn`
- `torch.nn.functional`
- `torch: Tensor`
- `transformers.modeling_outputs: BaseModelOutputWithPooling`
- `transformers.models.siglip: SiglipVisionModel`
- `sglang.srt.managers.mm_utils`
- `sglang.srt.model_loader.weight_utils`
- `sglang.srt.utils`
- `sglang.srt.configs.jet_vlm: JetVLMConfig`
- `sglang.srt.layers.logits_processor: LogitsProcessorOutput`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.managers.mm_utils: MultiModalityDataPaddingPatternMultimodalTokens`
- `sglang.srt.managers.schedule_batch: Modality, MultimodalDataItem, MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.models.jet_nemotron: JetNemotronForCausalLM`
