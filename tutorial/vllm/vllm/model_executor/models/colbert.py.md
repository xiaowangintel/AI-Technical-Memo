# colbert.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/colbert.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Inference-oriented vLLM model implementation for colbert, including architecture wrappers and weight loading logic. / 面向推理的 colbert vLLM 模型实现，包含架构封装与权重加载逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 20-32)
```python
from collections.abc import Iterable

import torch
from torch import nn

from vllm.config import PoolerConfig, VllmConfig
from vllm.model_executor.layers.pooler import Pooler
from vllm.model_executor.layers.pooler.tokwise import pooler_for_token_embed

from .bert import BertEmbeddingModel, BertModel
from .interfaces import HasInnerState, IsHybrid, SupportsLateInteraction
from .interfaces_base import default_pooling_type
from .lfm2 import Lfm2ForCausalLM, Lfm2Model
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Class `ColBERTMixin` (lines 35-184)
```python
class ColBERTMixin(nn.Module, SupportsLateInteraction):
    """Mixin that adds ColBERT late interaction support to any embedding model.

    ColBERT (Contextualized Late Interaction over BERT) uses per-token
    embeddings with a linear projection layer.  This mixin provides:

    - ColBERT linear projection initialisation / lazy creation
    - Weight loading helpers for the projection layer
    - A builder for the token-embedding pooler

    **Integration:**

    1. Inherit from both ``ColBERTMixin`` and ``nn.Module``.
    2. In ``__init__``: call ``super().__init__()``, then
       :meth:`_init_colbert_components`, then create ``self.model``
       (the backbone) and ``self.pooler`` via :meth:`_build_colbert_pooler`.
    3. In ``load_weights``: use :meth:`_load_colbert_weights` to separate
       the ColBERT projection weight, then delegate the rest to the backbone.
    """

    # Set during _init_colbert_components
    colbert_dim: int | None
    colbert_linear: nn.Linear | None
    hidden_size: int
    head_dtype: torch.dtype
```
**EN:** Class `ColBERTMixin` is a structural model block in the vLLM execution graph. It inherits from nn.Module, SupportsLateInteraction. Key methods include _init_colbert_components, _build_colbert_linear, _build_colbert_pooler, get_colbert_dim_from_config, _load_colbert_weights.
**CN:** 类 `ColBERTMixin` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module、SupportsLateInteraction。 关键方法包括 _init_colbert_components, _build_colbert_linear, _build_colbert_pooler, get_colbert_dim_from_config, _load_colbert_weights。

### Method `ColBERTMixin.get_colbert_dim_from_config` (lines 115-124)
```python
    @classmethod
    def get_colbert_dim_from_config(cls, hf_config) -> int | None:
        """Extract ColBERT dimension from a HuggingFace config.

        Checks ``colbert_dim``, ``dim`` and ``projection_dim`` in that order.
        """
        return (
            getattr(hf_config, "colbert_dim", None)
            or getattr(hf_config, "dim", None)
            or getattr(hf_config, "projection_dim", None)
        )
```
**EN:** Method `ColBERTMixin.get_colbert_dim_from_config` encapsulates a focused piece of reusable logic inside this module. The docstring says: Extract ColBERT dimension from a HuggingFace config.
**CN:** Method `ColBERTMixin.get_colbert_dim_from_config` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Extract ColBERT dimension from a HuggingFace config。

### Class `ColBERTModel` (lines 193-251)
```python
@default_pooling_type(seq_pooling_type="CLS", tok_pooling_type="ALL")
class ColBERTModel(ColBERTMixin, BertEmbeddingModel):
    """ColBERT late interaction model with BERT backbone.

    Supports the ``token_embed`` task (per-token embeddings for late
    interaction).  MaxSim scoring is computed externally.
    """

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        config = vllm_config.model_config.hf_config

        # Must run before super().__init__ because _build_pooler reads these.
        colbert_dim = self.get_colbert_dim_from_config(config)
        self._init_colbert_components(
            hidden_size=config.hidden_size,
            colbert_dim=colbert_dim,
            head_dtype=vllm_config.model_config.head_dtype,
        )

        super().__init__(vllm_config=vllm_config, prefix=prefix)

    def _build_model(self, vllm_config: VllmConfig, prefix: str = "") -> BertModel:
        return BertModel(vllm_config=vllm_config, prefix=prefix)

    def _build_pooler(self, pooler_config: PoolerConfig) -> Pooler:
```
**EN:** Class `ColBERTModel` organizes related behavior for this model family or helper component. It inherits from ColBERTMixin, BertEmbeddingModel. Key methods include __init__, _build_model, _build_pooler, load_weights.
**CN:** 类 `ColBERTModel` 用于组织该模型族或辅助组件的相关行为。 它继承自 ColBERTMixin、BertEmbeddingModel。 关键方法包括 __init__, _build_model, _build_pooler, load_weights。

### Method `ColBERTModel.__init__` (lines 200-211)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        config = vllm_config.model_config.hf_config

        # Must run before super().__init__ because _build_pooler reads these.
        colbert_dim = self.get_colbert_dim_from_config(config)
        self._init_colbert_components(
            hidden_size=config.hidden_size,
            colbert_dim=colbert_dim,
            head_dtype=vllm_config.model_config.head_dtype,
        )

        super().__init__(vllm_config=vllm_config, prefix=prefix)
```
**EN:** Method `ColBERTModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `ColBERTModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `ColBERTModel.load_weights` (lines 219-251)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]):
        def _strip(name: str) -> str:
            for p in ("model.", "bert."):
                if name.startswith(p):
                    name = name[len(p) :]
            return name

        weights_list = list(weights)
        model_side: list[tuple[str, torch.Tensor]] = []
        colbert_side: list[tuple[str, torch.Tensor]] = []

        for name, weight in weights_list:
            stripped = _strip(name)
            # Handle different checkpoint naming conventions
            if stripped in ("linear.weight", "colbert_linear.weight"):
                colbert_side.append(("colbert_linear.weight", weight))
            elif stripped.startswith("linear.") or stripped.startswith(
                "colbert_linear."
            ):
                new_name = stripped.replace("linear.", "colbert_linear.")
                colbert_side.append((new_name, weight))
            else:
                model_side.append((stripped, weight))

        loaded: set[str] = set()
        loaded_model = self.model.load_weights(model_side)
        loaded.update({"model." + n for n in loaded_model})

        if colbert_side:
            _, colbert_loaded = self._load_colbert_weights(colbert_side)
            loaded.update(colbert_loaded)

        return loaded
```
**EN:** Method `ColBERTModel.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `ColBERTModel.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `ColBERTModernBertModel` (lines 262-331)
```python
@default_pooling_type(seq_pooling_type="CLS", tok_pooling_type="ALL")
class ColBERTModernBertModel(ColBERTMixin, nn.Module):
    """ColBERT late interaction model with ModernBERT backbone.

    For ``lightonai/GTE-ModernColBERT-v1`` and similar models.
    The projection is auto-loaded from sentence-transformers ``1_Dense/``
    when not present in the main checkpoint.
    """

    is_pooling_model = True

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config

        colbert_dim = self.get_colbert_dim_from_config(config)
        self._init_colbert_components(
            hidden_size=config.hidden_size,
            colbert_dim=colbert_dim,
            head_dtype=vllm_config.model_config.head_dtype,
        )

        self.model = ModernBertModel(
            vllm_config=vllm_config,
            prefix=prefix,
```
**EN:** Class `ColBERTModernBertModel` is a structural model block in the vLLM execution graph. It inherits from ColBERTMixin, nn.Module. Key methods include __init__, embed_input_ids, forward, load_weights.
**CN:** 类 `ColBERTModernBertModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 ColBERTMixin、nn.Module。 关键方法包括 __init__, embed_input_ids, forward, load_weights。

### Method `ColBERTModernBertModel.__init__` (lines 272-290)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config

        colbert_dim = self.get_colbert_dim_from_config(config)
        self._init_colbert_components(
            hidden_size=config.hidden_size,
            colbert_dim=colbert_dim,
            head_dtype=vllm_config.model_config.head_dtype,
        )

        self.model = ModernBertModel(
            vllm_config=vllm_config,
            prefix=prefix,
        )

        pooler_config = vllm_config.model_config.pooler_config
        assert pooler_config is not None
        self.pooler = self._build_colbert_pooler(pooler_config)
```
**EN:** Method `ColBERTModernBertModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `ColBERTModernBertModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `ColBERTModernBertModel.embed_input_ids` (lines 292-293)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.embed_input_ids(input_ids)
```
**EN:** Method `ColBERTModernBertModel.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `ColBERTModernBertModel.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `ColBERTModernBertModel.forward` (lines 295-307)
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        intermediate_tensors=None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor:
        return self.model(
            input_ids=input_ids,
            positions=positions,
            inputs_embeds=inputs_embeds,
            intermediate_tensors=intermediate_tensors,
        )
```
**EN:** Method `ColBERTModernBertModel.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `ColBERTModernBertModel.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `ColBERTModernBertModel.load_weights` (lines 309-331)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]):
        other_weights, colbert_loaded = self._load_colbert_weights(weights)

        # Strip "model." prefix added by the embedding adapter
        model_weights = [
            (n[len("model.") :] if n.startswith("model.") else n, w)
            for n, w in other_weights
        ]

        loaded_model = self.model.load_weights(model_weights)
        loaded = {"model." + n for n in loaded_model} | colbert_loaded

        # When the ST projector was auto-loaded during init
        # (not from the main checkpoint), mark its params as loaded
        # so the weight validator doesn't complain.
        if hasattr(self.pooler, "head"):
            head = self.pooler.head
            projector = getattr(head, "projector", None)
            if projector is not None and isinstance(projector, nn.Module):
                for name, _ in projector.named_parameters():
                    loaded.add(f"pooler.head.projector.{name}")

        return loaded
```
**EN:** Method `ColBERTModernBertModel.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `ColBERTModernBertModel.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `ColBERTJinaRobertaModel` (lines 342-417)
```python
@default_pooling_type(seq_pooling_type="CLS", tok_pooling_type="ALL")
class ColBERTJinaRobertaModel(ColBERTMixin, nn.Module):
    """ColBERT late interaction model with Jina XLM-RoBERTa backbone.

    For ``jinaai/jina-colbert-v2`` and similar models.
    """

    is_pooling_model = True

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config

        colbert_dim = self.get_colbert_dim_from_config(config)
        self._init_colbert_components(
            hidden_size=config.hidden_size,
            colbert_dim=colbert_dim,
            head_dtype=vllm_config.model_config.head_dtype,
        )

        self.model = JinaRobertaModel(
            vllm_config=vllm_config,
            prefix=prefix,
        )
```
**EN:** Class `ColBERTJinaRobertaModel` is a structural model block in the vLLM execution graph. It inherits from ColBERTMixin, nn.Module. Key methods include __init__, embed_input_ids, forward, load_weights.
**CN:** 类 `ColBERTJinaRobertaModel` 是 vLLM 执行图中的结构化模型模块。 它继承自 ColBERTMixin、nn.Module。 关键方法包括 __init__, embed_input_ids, forward, load_weights。

### Method `ColBERTJinaRobertaModel.__init__` (lines 350-368)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config

        colbert_dim = self.get_colbert_dim_from_config(config)
        self._init_colbert_components(
            hidden_size=config.hidden_size,
            colbert_dim=colbert_dim,
            head_dtype=vllm_config.model_config.head_dtype,
        )

        self.model = JinaRobertaModel(
            vllm_config=vllm_config,
            prefix=prefix,
        )

        pooler_config = vllm_config.model_config.pooler_config
        assert pooler_config is not None
        self.pooler = self._build_colbert_pooler(pooler_config)
```
**EN:** Method `ColBERTJinaRobertaModel.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `ColBERTJinaRobertaModel.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `ColBERTJinaRobertaModel.embed_input_ids` (lines 370-371)
```python
    def embed_input_ids(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.model.embed_input_ids(input_ids)
```
**EN:** Method `ColBERTJinaRobertaModel.embed_input_ids` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline.
**CN:** Method `ColBERTJinaRobertaModel.embed_input_ids` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。

### Method `ColBERTJinaRobertaModel.forward` (lines 373-385)
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        intermediate_tensors=None,
        inputs_embeds: torch.Tensor | None = None,
    ) -> torch.Tensor:
        return self.model(
            input_ids=input_ids,
            positions=positions,
            inputs_embeds=inputs_embeds,
            intermediate_tensors=intermediate_tensors,
        )
```
**EN:** Method `ColBERTJinaRobertaModel.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `ColBERTJinaRobertaModel.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Method `ColBERTJinaRobertaModel.load_weights` (lines 387-417)
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]):
        weights_list = list(weights)
        model_side: list[tuple[str, torch.Tensor]] = []
        colbert_side: list[tuple[str, torch.Tensor]] = []

        for name, weight in weights_list:
            stripped = name
            # Strip "model." prefix added by the embedding adapter
            if stripped.startswith("model."):
                stripped = stripped[len("model.") :]
            # Strip "roberta." prefix from checkpoint
            if stripped.startswith("roberta."):
                stripped = stripped[len("roberta.") :]

            if stripped in ("linear.weight", "colbert_linear.weight"):
                colbert_side.append(("colbert_linear.weight", weight))
            elif stripped.startswith("pooler."):
                # Skip HF pooler weights (not used in ColBERT)
                continue
            else:
                model_side.append((stripped, weight))

        loaded: set[str] = set()
        loaded_model = self.model.load_weights(model_side)
        loaded.update({"model." + n for n in loaded_model})

        if colbert_side:
            _, colbert_loaded = self._load_colbert_weights(colbert_side)
            loaded.update(colbert_loaded)

        return loaded
```
**EN:** Method `ColBERTJinaRobertaModel.load_weights` maps checkpoint tensors into vLLM parameters, usually handling naming differences, tensor slicing, or custom loaders.
**CN:** Method `ColBERTJinaRobertaModel.load_weights` 将检查点张量映射到 vLLM 参数中，通常会处理命名差异、张量切分或自定义加载器。

### Class `ColBERTLfm2Model` (lines 426-512)
```python
@default_pooling_type(seq_pooling_type="CLS", tok_pooling_type="ALL")
class ColBERTLfm2Model(ColBERTMixin, nn.Module, HasInnerState, IsHybrid):
    """ColBERT late interaction model with LFM2 backbone.

    For ``LiquidAI/LFM2-ColBERT-350M`` and similar models.

    The projection is auto-loaded from sentence-transformers ``1_Dense/``
    when not present in the main checkpoint.
    """

    is_pooling_model = True
    # LFM2 is a hybrid model (attention + SSM layers); these flags ensure
    # HybridAttentionMambaModelConfig.verify_and_update_config runs so that
    # mamba_block_size and related cache settings are correctly initialised.
    is_hybrid = True
    has_inner_state = True

    @classmethod
    def get_mamba_state_shape_from_config(cls, vllm_config: VllmConfig):
        return Lfm2ForCausalLM.get_mamba_state_shape_from_config(vllm_config)

    @classmethod
    def get_mamba_state_dtype_from_config(cls, vllm_config: VllmConfig):
        return Lfm2ForCausalLM.get_mamba_state_dtype_from_config(vllm_config)
```
**EN:** Class `ColBERTLfm2Model` is a structural model block in the vLLM execution graph. It inherits from ColBERTMixin, nn.Module, HasInnerState, IsHybrid. Key methods include get_mamba_state_shape_from_config, get_mamba_state_dtype_from_config, get_mamba_state_copy_func, __init__, embed_input_ids, forward.
**CN:** 类 `ColBERTLfm2Model` 是 vLLM 执行图中的结构化模型模块。 它继承自 ColBERTMixin、nn.Module、HasInnerState、IsHybrid。 关键方法包括 get_mamba_state_shape_from_config, get_mamba_state_dtype_from_config, get_mamba_state_copy_func, __init__, embed_input_ids, forward。

### Method `ColBERTLfm2Model.get_mamba_state_shape_from_config` (lines 443-444)
```python
    @classmethod
    def get_mamba_state_shape_from_config(cls, vllm_config: VllmConfig):
        return Lfm2ForCausalLM.get_mamba_state_shape_from_config(vllm_config)
```
**EN:** Method `ColBERTLfm2Model.get_mamba_state_shape_from_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `ColBERTLfm2Model.get_mamba_state_shape_from_config` 封装了该模块中的一段可复用核心逻辑。

### Method `ColBERTLfm2Model.get_mamba_state_dtype_from_config` (lines 447-448)
```python
    @classmethod
    def get_mamba_state_dtype_from_config(cls, vllm_config: VllmConfig):
        return Lfm2ForCausalLM.get_mamba_state_dtype_from_config(vllm_config)
```
**EN:** Method `ColBERTLfm2Model.get_mamba_state_dtype_from_config` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `ColBERTLfm2Model.get_mamba_state_dtype_from_config` 封装了该模块中的一段可复用核心逻辑。

### Method `ColBERTLfm2Model.get_mamba_state_copy_func` (lines 451-452)
```python
    @classmethod
    def get_mamba_state_copy_func(cls):
        return Lfm2ForCausalLM.get_mamba_state_copy_func()
```
**EN:** Method `ColBERTLfm2Model.get_mamba_state_copy_func` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `ColBERTLfm2Model.get_mamba_state_copy_func` 封装了该模块中的一段可复用核心逻辑。

### Method `ColBERTLfm2Model.__init__` (lines 454-472)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        super().__init__()
        config = vllm_config.model_config.hf_config

        colbert_dim = self.get_colbert_dim_from_config(config)
        self._init_colbert_components(
            hidden_size=config.hidden_size,
            colbert_dim=colbert_dim,
            head_dtype=vllm_config.model_config.head_dtype,
        )

        self.model = Lfm2Model(
            vllm_config=vllm_config,
            prefix=prefix,
        )

        pooler_config = vllm_config.model_config.pooler_config
        assert pooler_config is not None
        self.pooler = self._build_colbert_pooler(pooler_config)
```
**EN:** Method `ColBERTLfm2Model.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `ColBERTLfm2Model.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

## Key Concepts / 关键概念
- **Attention / 注意力**
  - **EN:** Attention layers model token-to-token or token-to-feature interactions inside the architecture.
  - **CN:** 注意力层负责建模 token 与 token，或 token 与特征之间的交互。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **KV cache / KV 缓存**
  - **EN:** The runtime stores key/value state so decoding can reuse prior computation efficiently.
  - **CN:** 运行时会保存 key/value 状态，以便解码阶段高效复用历史计算。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from collections.abc import Iterable`
- **Third-party / 第三方**: `import torch`, `from torch import nn`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import PoolerConfig, VllmConfig`, `from vllm.model_executor.layers.pooler import Pooler`, `from vllm.model_executor.layers.pooler.tokwise import pooler_for_token_embed`, `from .bert import BertEmbeddingModel, BertModel`, `from .interfaces import HasInnerState, IsHybrid, SupportsLateInteraction`, `from .interfaces_base import default_pooling_type`, `from .lfm2 import Lfm2ForCausalLM, Lfm2Model`, `from .modernbert import ModernBertModel  # noqa: E402`, `from .bert_with_rope import JinaRobertaModel  # noqa: E402`
- **Module note / 模块说明**: **EN:** ColBERT late interaction model for retrieval and reranking.  ColBERT uses per-token embeddings and late interaction (MaxSim) scoring instead of single-vector representations or cross-encoder concatenation.  This module provides:  - :class:`ColBERTMixin` — mixin that adds ColBERT late-interaction support to any embedding model. - :class:`ColBERTModel` — ColBERT with BERT backbone (original architecture). - :class:`ColBERTModernBertModel` — ColBERT with ModernBERT backbone. - :class:`ColBERTJinaRobertaModel` — ColBERT with Jina XLM-RoBERTa backbone.  Reference: https://arxiv.org/abs/2004.12832 **CN:** 模块文档字符串给出的原始说明是：ColBERT late interaction model for retrieval and reranking.  ColBERT uses per-token embeddings and late interaction (MaxSim) scoring instead of single-vector representations or cross-encoder concatenation.  This module provides:  - :class:`ColBERTMixin` — mixin that adds ColBERT late-interaction support to any embedding model. - :class:`ColBERTModel` — ColBERT with BERT backbone (original architecture). - :class:`ColBERTModernBertModel` — ColBERT with ModernBERT backbone. - :class:`ColBERTJinaRobertaModel` — ColBERT with Jina XLM-RoBERTa backbone.  Reference: https://arxiv.org/abs/2004.12832。
