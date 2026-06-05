# colqwen3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/colqwen3.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for colqwen3, including architecture wrappers and weight loading logic. / 面向推理的 colqwen3 vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 22-42)
```python
from collections.abc import Iterable, Mapping

import torch
import torch.nn as nn
from transformers.models.qwen3_vl import Qwen3VLProcessor

from vllm.config import VllmConfig
from vllm.model_executor.layers.pooler.tokwise import pooler_for_token_embed
from vllm.model_executor.model_loader.weight_utils import default_weight_loader
from vllm.multimodal import MULTIMODAL_REGISTRY

from .interfaces import SupportsLateInteraction
from .interfaces_base import default_pooling_type
from .qwen2_vl import Qwen2VLMultiModalDataParser
from .qwen3_vl import (
    Qwen3VLDummyInputsBuilder,
    Qwen3VLForConditionalGeneration,
    Qwen3VLMultiModalProcessor,
    Qwen3VLProcessingInfo,
)
from .utils import AutoWeightsLoader, WeightsMapper
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `ColQwen3ProcessingInfo` (lines 45-107)
```python
class ColQwen3ProcessingInfo(Qwen3VLProcessingInfo):
    """Processing info for ColQwen3 models.

    ColQwen3 models (TomoroAI, OpenSearch-AI, etc.) use custom HuggingFace
    configs (e.g. ColQwen3Config, OpsColQwen3Config) that are not instances
    of Qwen3VLConfig. We override get_hf_config() and get_hf_processor()
    to skip the strict type check, similar to OpenCUAProcessingInfo.
    """

    def get_hf_config(self):
        return self.ctx.get_hf_config()

    def get_hf_processor(self, **kwargs: object) -> Qwen3VLProcessor:
        # Force standard Qwen3VLProcessor even when trust_remote_code=True.
        # ColQwen3 custom processors (e.g. ColQwen3Processor) have
        # incompatible interfaces with vLLM's Qwen3VLMultiModalProcessor.
        # The standard Qwen3VLProcessor handles both text and image inputs
        # correctly for the Qwen3-VL backbone.
        return self.ctx.get_hf_processor(
            Qwen3VLProcessor,
            use_fast=kwargs.pop("use_fast", True),
            **kwargs,
        )

    @property
```
**EN:** Class `ColQwen3ProcessingInfo` organizes related behavior for this model family or helper component. It inherits from Qwen3VLProcessingInfo. Key methods include get_hf_config, get_hf_processor, _supports_video, get_video_processor, get_supported_mm_limits, get_mm_max_tokens_per_item.
**CN:** 类 `ColQwen3ProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 Qwen3VLProcessingInfo。 关键方法包括 get_hf_config, get_hf_processor, _supports_video, get_video_processor, get_supported_mm_limits, get_mm_max_tokens_per_item。

### Method `ColQwen3ProcessingInfo.get_hf_config` (lines 54-55)
```python
    def get_hf_config(self):
        return self.ctx.get_hf_config()
```
**EN:** Method `ColQwen3ProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `ColQwen3ProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `ColQwen3ProcessingInfo.get_hf_processor` (lines 57-67)
```python
    def get_hf_processor(self, **kwargs: object) -> Qwen3VLProcessor:
        # Force standard Qwen3VLProcessor even when trust_remote_code=True.
        # ColQwen3 custom processors (e.g. ColQwen3Processor) have
        # incompatible interfaces with vLLM's Qwen3VLMultiModalProcessor.
        # The standard Qwen3VLProcessor handles both text and image inputs
        # correctly for the Qwen3-VL backbone.
        return self.ctx.get_hf_processor(
            Qwen3VLProcessor,
            use_fast=kwargs.pop("use_fast", True),
            **kwargs,
        )
```
**EN:** Method `ColQwen3ProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `ColQwen3ProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `ColQwen3ProcessingInfo.get_video_processor` (lines 74-80)
```python
    def get_video_processor(self, **kwargs: object):
        if not self._supports_video:
            raise AttributeError(
                f"The processor for {self.ctx.model_config.model} does not "
                "support video inputs (no video_processor attribute)."
            )
        return self.get_hf_processor(**kwargs).video_processor  # type: ignore[attr-defined]
```
**EN:** Method `ColQwen3ProcessingInfo.get_video_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `ColQwen3ProcessingInfo.get_video_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `ColQwen3ProcessingInfo.get_supported_mm_limits` (lines 82-86)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        limits: dict[str, int | None] = {"image": None}
        if self._supports_video:
            limits["video"] = None
        return limits
```
**EN:** Method `ColQwen3ProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `ColQwen3ProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Class `ColQwen3Model` (lines 116-301)
```python
@default_pooling_type(seq_pooling_type="CLS", tok_pooling_type="ALL")
@MULTIMODAL_REGISTRY.register_processor(
    Qwen3VLMultiModalProcessor,
    info=ColQwen3ProcessingInfo,
    dummy_inputs=Qwen3VLDummyInputsBuilder,
)
class ColQwen3Model(Qwen3VLForConditionalGeneration, SupportsLateInteraction):
    """ColQwen3 late interaction model for multi-modal retrieval/reranking.

    This model extends Qwen3VLForConditionalGeneration with a ColBERT-style
    linear projection layer for per-token embeddings. It supports:
    - "token_embed" task: Per-token embeddings for late interaction scoring

    The model produces L2-normalized per-token embeddings by:
    1. Running the Qwen3-VL backbone (vision + language) to get hidden states
    2. Projecting hidden states through a linear layer (hidden_size -> embed_dim)
    3. L2-normalizing the projected embeddings

    ColBERT-style MaxSim scoring is computed externally, either client-side
    or via the late interaction scoring path in ServingScores.

    Attributes:
        custom_text_proj: Linear projection from hidden_size to embed_dim
    """
```
**EN:** Class `ColQwen3Model` organizes related behavior for this model family or helper component. It inherits from Qwen3VLForConditionalGeneration, SupportsLateInteraction. Key methods include __init__, forward, _is_proj_weight, load_weights.
**CN:** 类 `ColQwen3Model` 用于组织该模型族或辅助组件的相关行为。 它继承自 Qwen3VLForConditionalGeneration、SupportsLateInteraction。 关键方法包括 __init__, forward, _is_proj_weight, load_weights。

### Method `ColQwen3Model.__init__` (lines 162-204)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = "model"):
        super().__init__(vllm_config=vllm_config, prefix=prefix)

        config = vllm_config.model_config.hf_config
        head_dtype = vllm_config.model_config.head_dtype

        hidden_size = getattr(config, "hidden_size", None)
        if hidden_size is None and hasattr(config, "text_config"):
            hidden_size = config.text_config.hidden_size
        if hidden_size is None:
            raise ValueError(
                "Unable to determine text hidden size from config. "
                "Expected 'hidden_size' or 'text_config.hidden_size'."
            )
        self._proj_hidden_size = hidden_size

        # (TomoroAI: embed_dim, OpenSearch: dims, ColPali: dim)
        self.embed_dim: int | None = (
            getattr(config, "embed_dim", None)
            or getattr(config, "dims", None)
            or getattr(config, "dim", None)
            or getattr(config, "projection_dim", None)
            or getattr(config, "colbert_dim", None)
        )

        # Build the projection layer if embed_dim is known
        if self.embed_dim is not None:
            self.custom_text_proj = nn.Linear(
                hidden_size,
                self.embed_dim,
                bias=False,
                dtype=head_dtype,
            )
        else:
            # Will be created during load_weights when dim is inferred
            self.custom_text_proj = None

        pooler_config = vllm_config.model_config.pooler_config
        assert pooler_config is not None
        self.pooler = pooler_for_token_embed(
            pooler_config,
            projector=None,
        )
```
**EN:** Method `ColQwen3Model.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `ColQwen3Model.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `ColQwen3Model.forward` (lines 206-233)
```python
    def forward(
        self,
        input_ids: torch.Tensor | None,
        positions: torch.Tensor,
        intermediate_tensors=None,
        inputs_embeds: torch.Tensor | None = None,
        **kwargs: object,
    ) -> torch.Tensor:
        """Run forward pass producing per-token embeddings."""
        hidden_states = super().forward(
            input_ids=input_ids,
            positions=positions,
            intermediate_tensors=intermediate_tensors,
            inputs_embeds=inputs_embeds,
            **kwargs,
        )

        if not isinstance(hidden_states, torch.Tensor):
            return hidden_states  # type: ignore

        if self.custom_text_proj is not None:
            proj_dtype = self.custom_text_proj.weight.dtype
            if hidden_states.dtype != proj_dtype:
                hidden_states = hidden_states.to(proj_dtype)
            hidden_states = self.custom_text_proj(hidden_states)

        # L2 normalize
        return torch.nn.functional.normalize(hidden_states, p=2, dim=-1)
```
**EN:** Method `ColQwen3Model.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Run forward pass producing per-token embeddings.
**CN:** Method `ColQwen3Model.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Run forward pass producing per-token embeddings。

### Method `ColQwen3Model.load_weights` (lines 245-301)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        """Load weights with special handling for ColQwen3 projection layer."""
        weights_list = list(weights)
        proj_weights: list[tuple[str, torch.Tensor]] = []
        model_weights: list[tuple[str, torch.Tensor]] = []

        # Scan all weight names to determine if re-prefixing is needed.
        # OpenSearch-AI models have unprefixed weights ("language_model.*",
        # "visual.*") that need "model." added so hf_to_vllm_mapper can
        # process them. Only re-prefix if ALL backbone weights are
        # unprefixed (no "vlm." or "model." prefix found).
        has_unprefixed = any(
            name.startswith("language_model.") or name.startswith("visual.")
            for name, _ in weights_list
        )
        has_prefixed = any(
            name.startswith("vlm.") or name.startswith("model.")
            for name, _ in weights_list
        )
        needs_reprefix = has_unprefixed and not has_prefixed

        for name, weight in weights_list:
            if self._is_proj_weight(name):
                proj_weights.append((name, weight))
            else:
                if needs_reprefix and not self._is_proj_weight(name):
                    name = "model." + name
                model_weights.append((name, weight))

        loader = AutoWeightsLoader(self)
        loaded = loader.load_weights(model_weights, mapper=self.hf_to_vllm_mapper)

        if proj_weights:
            model_dtype = next(self.language_model.parameters()).dtype
            model_device = next(self.language_model.parameters()).device

            for name, weight in proj_weights:
                if self.embed_dim is None and "weight" in name:
                    self.embed_dim = weight.shape[0]
                    has_bias = any("bias" in n for n, _ in proj_weights)
                    self.custom_text_proj = nn.Linear(
                        self._proj_hidden_size,
                        self.embed_dim,
                        bias=has_bias,
                        dtype=model_dtype,
                    )
                    self.custom_text_proj.to(model_device)

                if self.custom_text_proj is not None:
                    param_name = name.split(".")[-1]
                    param = getattr(self.custom_text_proj, param_name, None)
                    if param is not None:
                        weight = weight.to(device=param.device, dtype=param.dtype)
                        default_weight_loader(param, weight)
                        loaded.add(f"custom_text_proj.{param_name}")

        return loaded
```
**EN:** Method `ColQwen3Model.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders. The docstring says: Load weights with special handling for ColQwen3 projection layer.
**CN:** Method `ColQwen3Model.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。 文档字符串表达的核心意思是：Load weights with special handling for ColQwen3 projection layer。

## Key Concepts / 关键概念
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from collections.abc import Iterable, Mapping`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`, `from transformers.models.qwen3_vl import Qwen3VLProcessor`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.model_executor.layers.pooler.tokwise import pooler_for_token_embed`, `from vllm.model_executor.model_loader.weight_utils import default_weight_loader`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from .interfaces import SupportsLateInteraction`, `from .interfaces_base import default_pooling_type`, `from .qwen2_vl import Qwen2VLMultiModalDataParser`, `from .qwen3_vl import (`, `from .utils import AutoWeightsLoader, WeightsMapper`
- **Module note / 模块说明**: **EN:** ColQwen3 late interaction model for multi-modal retrieval and reranking.  ColQwen3 extends Qwen3-VL with a ColBERT-style late interaction head, producing per-token embeddings for both text and image inputs. It uses MaxSim scoring for retrieval/reranking tasks.  This model supports the "token_embed" pooling task and is designed for multi-vector retrieval of documents containing both text and images.  Reference: https://arxiv.org/abs/2407.01449 (ColPali) Based on: Qwen3-VL backbone with custom text projection  Target models: - TomoroAI/tomoro-colqwen3-embed-8b - OpenSearch-AI/Ops-Colqwen3-4B - nvidia/nemotron-colembed-vl-4b-v2 **CN:** 模块文档字符串给出的原始说明是：ColQwen3 late interaction model for multi-modal retrieval and reranking.  ColQwen3 extends Qwen3-VL with a ColBERT-style late interaction head, producing per-token embeddings for both text and image inputs. It uses MaxSim scoring for retrieval/reranking tasks.  This model supports the "token_embed" pooling task and is designed for multi-vector retrieval of documents containing both text and images.  Reference: https://arxiv.org/abs/2407.01449 (ColPali) Based on: Qwen3-VL backbone with custom text projection  Target models: - TomoroAI/tomoro-colqwen3-embed-8b - OpenSearch-AI/Ops-Colqwen3-4B - nvidia/nemotron-colembed-vl-4b-v2。
