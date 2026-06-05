# colqwen3_5.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/colqwen3_5.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for colqwen3_5, including architecture wrappers and weight loading logic. / 面向推理的 colqwen3_5 vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 20-42)
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
from .qwen3_5 import (
    Qwen3_5ForConditionalGeneration,
    Qwen3_5ProcessingInfo,
)
from .qwen3_vl import (
    Qwen3VLDummyInputsBuilder,
    Qwen3VLMultiModalProcessor,
)
from .utils import AutoWeightsLoader, WeightsMapper
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `ColQwen3_5ProcessingInfo` (lines 45-103)
```python
class ColQwen3_5ProcessingInfo(Qwen3_5ProcessingInfo):
    """Processing info for ColQwen3.5 models.

    ColQwen3.5 models use custom HuggingFace processors (e.g.
    ColQwen3_5Processor) that are incompatible with vLLM's
    Qwen3VLMultiModalProcessor. We override get_hf_config() and
    get_hf_processor() to skip the strict type check and force the
    standard Qwen3VLProcessor.
    """

    def get_hf_config(self):
        return self.ctx.get_hf_config()

    def get_hf_processor(self, **kwargs: object) -> Qwen3VLProcessor:
        return self.ctx.get_hf_processor(
            Qwen3VLProcessor,
            use_fast=kwargs.pop("use_fast", True),
            **kwargs,
        )

    @property
    def _supports_video(self) -> bool:
        """Check if the HF processor supports video inputs."""
        return hasattr(self.get_hf_processor(), "video_processor")
```
**EN:** Class `ColQwen3_5ProcessingInfo` organizes related behavior for this model family or helper component. It inherits from Qwen3_5ProcessingInfo. Key methods include get_hf_config, get_hf_processor, _supports_video, get_video_processor, get_supported_mm_limits, get_mm_max_tokens_per_item.
**CN:** 类 `ColQwen3_5ProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 Qwen3_5ProcessingInfo。 关键方法包括 get_hf_config, get_hf_processor, _supports_video, get_video_processor, get_supported_mm_limits, get_mm_max_tokens_per_item。

### Method `ColQwen3_5ProcessingInfo.get_hf_config` (lines 55-56)
```python
    def get_hf_config(self):
        return self.ctx.get_hf_config()
```
**EN:** Method `ColQwen3_5ProcessingInfo.get_hf_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `ColQwen3_5ProcessingInfo.get_hf_config` 封装了该模块中的一段可复用核心逻辑。

### Method `ColQwen3_5ProcessingInfo.get_hf_processor` (lines 58-63)
```python
    def get_hf_processor(self, **kwargs: object) -> Qwen3VLProcessor:
        return self.ctx.get_hf_processor(
            Qwen3VLProcessor,
            use_fast=kwargs.pop("use_fast", True),
            **kwargs,
        )
```
**EN:** Method `ColQwen3_5ProcessingInfo.get_hf_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `ColQwen3_5ProcessingInfo.get_hf_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `ColQwen3_5ProcessingInfo.get_video_processor` (lines 70-76)
```python
    def get_video_processor(self, **kwargs: object):
        if not self._supports_video:
            raise AttributeError(
                f"The processor for {self.ctx.model_config.model} does not "
                "support video inputs (no video_processor attribute)."
            )
        return self.get_hf_processor(**kwargs).video_processor  # type: ignore[attr-defined]
```
**EN:** Method `ColQwen3_5ProcessingInfo.get_video_processor` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `ColQwen3_5ProcessingInfo.get_video_processor` 封装了该模块中的一段可复用核心逻辑。

### Method `ColQwen3_5ProcessingInfo.get_supported_mm_limits` (lines 78-82)
```python
    def get_supported_mm_limits(self) -> Mapping[str, int | None]:
        limits: dict[str, int | None] = {"image": None}
        if self._supports_video:
            limits["video"] = None
        return limits
```
**EN:** Method `ColQwen3_5ProcessingInfo.get_supported_mm_limits` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `ColQwen3_5ProcessingInfo.get_supported_mm_limits` 封装了该模块中的一段可复用核心逻辑。

### Class `ColQwen3_5Model` (lines 112-246)
```python
@default_pooling_type(seq_pooling_type="CLS", tok_pooling_type="ALL")
@MULTIMODAL_REGISTRY.register_processor(
    Qwen3VLMultiModalProcessor,
    info=ColQwen3_5ProcessingInfo,
    dummy_inputs=Qwen3VLDummyInputsBuilder,
)
class ColQwen3_5Model(
    Qwen3_5ForConditionalGeneration,
    SupportsLateInteraction,
):
    """ColQwen3.5 late interaction model for multi-modal retrieval/reranking.

    This model extends Qwen3_5ForConditionalGeneration with a ColBERT-style
    linear projection layer for per-token embeddings. It supports:
    - "token_embed" task: Per-token embeddings for late interaction scoring

    The model produces per-token embeddings by:
    1. Running the Qwen3.5 backbone (vision + language) to get hidden states
    2. Projecting hidden states through a linear layer (hidden_size -> embed_dim)
    3. L2 normalization is handled by the pooler via PoolerNormalize

    Attributes:
        custom_text_proj: Linear projection from hidden_size to embed_dim
    """
```
**EN:** Class `ColQwen3_5Model` organizes related behavior for this model family or helper component. It inherits from Qwen3_5ForConditionalGeneration, SupportsLateInteraction. Key methods include __init__, forward, _is_proj_weight, load_weights.
**CN:** 类 `ColQwen3_5Model` 用于组织该模型族或辅助组件的相关行为。 它继承自 Qwen3_5ForConditionalGeneration、SupportsLateInteraction。 关键方法包括 __init__, forward, _is_proj_weight, load_weights。

### Method `ColQwen3_5Model.__init__` (lines 144-181)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
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

        # (ColPali: dim, projection_dim, colbert_dim)
        self.embed_dim: int = (
            getattr(config, "embed_dim", None)
            or getattr(config, "dims", None)
            or getattr(config, "dim", None)
            or getattr(config, "projection_dim", None)
            or getattr(config, "colbert_dim", None)
            or 128  # default from reference implementation
        )

        self.custom_text_proj = nn.Linear(
            hidden_size,
            self.embed_dim,
            bias=False,
            dtype=head_dtype,
        )

        pooler_config = vllm_config.model_config.pooler_config
        assert pooler_config is not None
        self.pooler = pooler_for_token_embed(
            pooler_config,
            projector=None,
        )
```
**EN:** Method `ColQwen3_5Model.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `ColQwen3_5Model.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `ColQwen3_5Model.forward` (lines 183-208)
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

        proj_dtype = self.custom_text_proj.weight.dtype
        if hidden_states.dtype != proj_dtype:
            hidden_states = hidden_states.to(proj_dtype)

        # Project to embedding dimension (normalization handled by pooler)
        return self.custom_text_proj(hidden_states)
```
**EN:** Method `ColQwen3_5Model.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Run forward pass producing per-token embeddings.
**CN:** Method `ColQwen3_5Model.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Run forward pass producing per-token embeddings。

### Method `ColQwen3_5Model.load_weights` (lines 220-246)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]) -> set[str]:
        """Load weights with special handling for projection layer."""
        weights_list = list(weights)
        proj_weights: list[tuple[str, torch.Tensor]] = []
        model_weights: list[tuple[str, torch.Tensor]] = []

        for name, weight in weights_list:
            if self._is_proj_weight(name):
                proj_weights.append((name, weight))
            else:
                model_weights.append((name, weight))

        loader = AutoWeightsLoader(
            self,
            skip_prefixes=["mtp."],
        )
        loaded = loader.load_weights(model_weights, mapper=self.hf_to_vllm_mapper)

        for name, weight in proj_weights:
            param_name = name.split(".")[-1]
            param = getattr(self.custom_text_proj, param_name, None)
            if param is not None:
                weight = weight.to(device=param.device, dtype=param.dtype)
                default_weight_loader(param, weight)
                loaded.add(f"custom_text_proj.{param_name}")

        return loaded
```
**EN:** Method `ColQwen3_5Model.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders. The docstring says: Load weights with special handling for projection layer.
**CN:** Method `ColQwen3_5Model.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。 文档字符串表达的核心意思是：Load weights with special handling for projection layer。

## Key Concepts / 关键概念
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from collections.abc import Iterable, Mapping`
- **Third-party / 第三方**: `import torch`, `import torch.nn as nn`, `from transformers.models.qwen3_vl import Qwen3VLProcessor`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.model_executor.layers.pooler.tokwise import pooler_for_token_embed`, `from vllm.model_executor.model_loader.weight_utils import default_weight_loader`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from .interfaces import SupportsLateInteraction`, `from .interfaces_base import default_pooling_type`, `from .qwen2_vl import Qwen2VLMultiModalDataParser`, `from .qwen3_5 import (`, `from .qwen3_vl import (`, `from .utils import AutoWeightsLoader, WeightsMapper`
- **Module note / 模块说明**: **EN:** ColQwen3.5 late interaction model for multi-modal retrieval and reranking.  ColQwen3.5 extends Qwen3.5 with a ColBERT-style late interaction head, producing per-token embeddings for both text and image inputs. It uses MaxSim scoring for retrieval/reranking tasks.  This model supports the "token_embed" pooling task and is designed for multi-vector retrieval of documents containing both text and images.  Reference: https://arxiv.org/abs/2407.01449 (ColPali) Based on: Qwen3.5 backbone with custom text projection  Target models: - athrael-soju/colqwen3.5-4.5B-v3 **CN:** 模块文档字符串给出的原始说明是：ColQwen3.5 late interaction model for multi-modal retrieval and reranking.  ColQwen3.5 extends Qwen3.5 with a ColBERT-style late interaction head, producing per-token embeddings for both text and image inputs. It uses MaxSim scoring for retrieval/reranking tasks.  This model supports the "token_embed" pooling task and is designed for multi-vector retrieval of documents containing both text and images.  Reference: https://arxiv.org/abs/2407.01449 (ColPali) Based on: Qwen3.5 backbone with custom text projection  Target models: - athrael-soju/colqwen3.5-4.5B-v3。
