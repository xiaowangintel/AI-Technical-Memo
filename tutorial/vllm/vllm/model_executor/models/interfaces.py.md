# interfaces.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/interfaces.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Interface definition module that describes model capabilities and helper predicates. / 接口定义模块，描述模型能力标记及辅助判断函数。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-41)
```python
import asyncio
from collections.abc import (
    AsyncGenerator,
    Callable,
    Iterable,
    Mapping,
    MutableSequence,
    Sequence,
)
from contextlib import ExitStack, contextmanager, nullcontext
from typing import (
    TYPE_CHECKING,
    Any,
    ClassVar,
    Literal,
    Protocol,
    TypeAlias,
    overload,
    runtime_checkable,
)

import numpy as np
import torch
import torch.nn as nn
from torch import Tensor
from transformers.models.whisper.tokenization_whisper import LANGUAGES
from typing_extensions import Self, TypeIs

from vllm.config import ModelConfig, SpeechToTextConfig, SpeechToTextParams
from vllm.inputs import PromptType, TokensPrompt
from vllm.logger import init_logger
from vllm.model_executor.layers.mamba.mamba_utils import MambaStateCopyFunc
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.tasks import ScoreType
from vllm.utils.collection_utils import common_prefix
from vllm.utils.func_utils import supports_kw

from .interfaces_base import VllmModel
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Constants / assignments (lines 90-90)
```python
_language_model_by_module = dict[nn.Module, VllmModel]()
```
**EN:** This block defines _language_model_by_module, a shared constant or lookup table that later code uses to make branching, mapping, or numerical decisions.
**CN:** 该代码块定义了 _language_model_by_module，这是后续逻辑进行分支选择、映射查找或数值控制时会复用的常量/查找表。

### Function `_require_is_multimodal` (lines 73-86)
```python
def _require_is_multimodal(is_multimodal: Tensor | None) -> Tensor:
    """
    A helper function to be used in the context of
    [vllm.model_executor.models.interfaces.SupportsMultiModal.embed_input_ids][]
    to provide a better error message.
    """
    if is_multimodal is None:
        raise ValueError(
            "`embed_input_ids` now requires `is_multimodal` arg, "
            "please update your model runner according to "
            "https://github.com/vllm-project/vllm/pull/16229."
        )

    return is_multimodal
```
**EN:** Function `_require_is_multimodal` encapsulates a focused piece of reusable logic inside this module. The docstring says: A helper function to be used in the context of [vllm.model_executor.models.interfaces.SupportsMultiModal.embed_input_ids][] to provide a better error message.
**CN:** Function `_require_is_multimodal` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：A helper function to be used in the context of [vllm.model_executor.models.interfaces.SupportsMultiModal.embed_input_ids][] to provide a better error message。

### Function `supports_multimodal` (lines 451-451)
```python
@overload
def supports_multimodal(model: type[object]) -> TypeIs[type[SupportsMultiModal]]: ...
```
**EN:** Function `supports_multimodal` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `supports_multimodal` 封装了该模块中的一段可复用核心逻辑。

### Function `supports_multimodal` (lines 455-455)
```python
@overload
def supports_multimodal(model: object) -> TypeIs[SupportsMultiModal]: ...
```
**EN:** Function `supports_multimodal` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `supports_multimodal` 封装了该模块中的一段可复用核心逻辑。

### Function `supports_multimodal` (lines 458-461)
```python
def supports_multimodal(
    model: type[object] | object,
) -> TypeIs[type[SupportsMultiModal]] | TypeIs[SupportsMultiModal]:
    return getattr(model, "supports_multimodal", False)
```
**EN:** Function `supports_multimodal` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `supports_multimodal` 封装了该模块中的一段可复用核心逻辑。

### Function `supports_multimodal_raw_input_only` (lines 464-465)
```python
def supports_multimodal_raw_input_only(model: type[object] | object) -> bool:
    return getattr(model, "supports_multimodal_raw_input_only", False)
```
**EN:** Function `supports_multimodal_raw_input_only` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `supports_multimodal_raw_input_only` 封装了该模块中的一段可复用核心逻辑。

### Function `requires_raw_input_tokens` (lines 468-469)
```python
def requires_raw_input_tokens(model: type[object] | object) -> bool:
    return getattr(model, "requires_raw_input_tokens", False)
```
**EN:** Function `requires_raw_input_tokens` encapsulates a focused piece of reusable logic inside this module.
**CN:** Function `requires_raw_input_tokens` 封装了该模块中的一段可复用核心逻辑。

### Class `SupportsMultiModal` (lines 94-408)
```python
@runtime_checkable
class SupportsMultiModal(Protocol):
    """The interface required for all multi-modal models."""

    supports_multimodal: ClassVar[Literal[True]] = True
    """
    A flag that indicates this model supports multi-modal inputs.

    Note:
        There is no need to redefine this flag if this class is in the
        MRO of your model class.
    """

    supports_multimodal_raw_input_only: ClassVar[bool] = False
    """
    A flag that indicates this model supports multi-modal inputs and processes
    them in their raw form and not embeddings.
    """

    supports_encoder_tp_data: ClassVar[bool] = False
    """
    A flag that indicates whether this model supports
    `multimodal_config.mm_encoder_tp_mode="data"`.
    """
```
**EN:** Class `SupportsMultiModal` organizes related behavior for this model family or helper component. It inherits from Protocol. Key methods include get_placeholder_str, embed_multimodal, configure_mm_token_handling, get_language_model, _mark_language_model, _mark_tower_model.
**CN:** 类 `SupportsMultiModal` 用于组织该模型族或辅助组件的相关行为。 它继承自 Protocol。 关键方法包括 get_placeholder_str, embed_multimodal, configure_mm_token_handling, get_language_model, _mark_language_model, _mark_tower_model。

### Method `SupportsMultiModal.get_placeholder_str` (lines 147-151)
```python
    @classmethod
    def get_placeholder_str(cls, modality: str, i: int) -> str | None:
        """
        Get the placeholder text for the `i`th `modality` item in the prompt.
        """
        ...
```
**EN:** Method `SupportsMultiModal.get_placeholder_str` encapsulates a focused piece of reusable logic inside this module. The docstring says: Get the placeholder text for the `i`th `modality` item in the prompt.
**CN:** Method `SupportsMultiModal.get_placeholder_str` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Get the placeholder text for the `i`th `modality` item in the prompt。

### Method `SupportsMultiModal.embed_multimodal` (lines 153-163)
```python
    def embed_multimodal(self, **kwargs: object) -> MultiModalEmbeddings:
        """
        Returns multimodal embeddings generated from multimodal kwargs
        to be merged with text embeddings.

        Note:
            The returned multimodal embeddings must be in the same order as
            the appearances of their corresponding multimodal data item in the
            input prompt.
        """
        ...
```
**EN:** Method `SupportsMultiModal.embed_multimodal` prepares embedding representations so tokens or multimodal features can enter the shared model pipeline. The docstring says: Returns multimodal embeddings generated from multimodal kwargs to be merged with text embeddings.
**CN:** Method `SupportsMultiModal.embed_multimodal` 用于准备嵌入表示，使 token 或多模态特征能够进入统一的模型流水线。 文档字符串表达的核心意思是：Returns multimodal embeddings generated from multimodal kwargs to be merged with text embeddings。

### Class `SupportsMultiModalPruning` (lines 412-447)
```python
@runtime_checkable
class SupportsMultiModalPruning(Protocol):
    """The interface required for models that support returning both input
    embeddings and positions. Model may require custom positions for dynamic
    pruning of multimodal embeddings.
    """

    supports_multimodal_pruning: ClassVar[Literal[True]] = True

    def recompute_mrope_positions(
        self,
        input_ids: list[int],
        multimodal_embeddings: MultiModalEmbeddings,
        mrope_positions: torch.LongTensor,
        num_computed_tokens: int,
    ) -> tuple[MultiModalEmbeddings, Tensor, int]:
        """
        Update part of input mrope positions (starting with
        num_computed_tokens index). Original mrope_positions are computed
        for unpruned sequence and becomes incorrect once pruning occurs,
        so once we prune media tokens we should reflect this in the
        mrope_positions before we feed it to LLM.

        Args:
            input_ids: (N,) All input tokens of the prompt containing
```
**EN:** Class `SupportsMultiModalPruning` organizes related behavior for this model family or helper component. It inherits from Protocol. Key methods include recompute_mrope_positions.
**CN:** 类 `SupportsMultiModalPruning` 用于组织该模型族或辅助组件的相关行为。 它继承自 Protocol。 关键方法包括 recompute_mrope_positions。

### Method `SupportsMultiModalPruning.recompute_mrope_positions` (lines 420-447)
```python
    def recompute_mrope_positions(
        self,
        input_ids: list[int],
        multimodal_embeddings: MultiModalEmbeddings,
        mrope_positions: torch.LongTensor,
        num_computed_tokens: int,
    ) -> tuple[MultiModalEmbeddings, Tensor, int]:
        """
        Update part of input mrope positions (starting with
        num_computed_tokens index). Original mrope_positions are computed
        for unpruned sequence and becomes incorrect once pruning occurs,
        so once we prune media tokens we should reflect this in the
        mrope_positions before we feed it to LLM.

        Args:
            input_ids: (N,) All input tokens of the prompt containing
                entire sequence.
            multimodal_embeddings: Tuple of multimodal embeddings that
                fits into the prefill chunk that is being processed.
            mrope_positions: Existing mrope positions (3, N) for entire
                sequence
            num_computed_tokens: A number of computed tokens so far.

        Returns:
            Tuple of (multimodal_embeddings, mrope_positions,
                mrope_position_delta).
        """
        ...
```
**EN:** Method `SupportsMultiModalPruning.recompute_mrope_positions` encapsulates a focused piece of reusable logic inside this module. The docstring says: Update part of input mrope positions (starting with num_computed_tokens index).
**CN:** Method `SupportsMultiModalPruning.recompute_mrope_positions` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Update part of input mrope positions (starting with num_computed_tokens index)。

### Class `SupportsScoreTemplate` (lines 493-517)
```python
@runtime_checkable
class SupportsScoreTemplate(Protocol):
    """The interface required for all models that support score template."""

    supports_score_template: ClassVar[Literal[True]] = True
    """
    A flag that indicates this model supports score template.

    Note:
        There is no need to redefine this flag if this class is in the
        MRO of your model class.
    """

    @classmethod
    def get_score_template(cls, query: str, document: str) -> str | None:
        """
        Generate a full prompt by populating the score template with query and document content.
        """  # noqa: E501
        ...

    @classmethod
    def post_process_tokens(cls, prompt: TokensPrompt) -> None:
        """
        Perform architecture-specific manipulations on the input tokens.
        """
```
**EN:** Class `SupportsScoreTemplate` organizes related behavior for this model family or helper component. It inherits from Protocol. Key methods include get_score_template, post_process_tokens.
**CN:** 类 `SupportsScoreTemplate` 用于组织该模型族或辅助组件的相关行为。 它继承自 Protocol。 关键方法包括 get_score_template, post_process_tokens。

### Method `SupportsScoreTemplate.get_score_template` (lines 506-510)
```python
    @classmethod
    def get_score_template(cls, query: str, document: str) -> str | None:
        """
        Generate a full prompt by populating the score template with query and document content.
        """  # noqa: E501
        ...
```
**EN:** Method `SupportsScoreTemplate.get_score_template` encapsulates a focused piece of reusable logic inside this module. The docstring says: Generate a full prompt by populating the score template with query and document content.
**CN:** Method `SupportsScoreTemplate.get_score_template` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Generate a full prompt by populating the score template with query and document content。

### Method `SupportsScoreTemplate.post_process_tokens` (lines 513-517)
```python
    @classmethod
    def post_process_tokens(cls, prompt: TokensPrompt) -> None:
        """
        Perform architecture-specific manipulations on the input tokens.
        """
        ...
```
**EN:** Method `SupportsScoreTemplate.post_process_tokens` encapsulates a focused piece of reusable logic inside this module. The docstring says: Perform architecture-specific manipulations on the input tokens.
**CN:** Method `SupportsScoreTemplate.post_process_tokens` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Perform architecture-specific manipulations on the input tokens。

### Class `SupportsLoRA` (lines 537-555)
```python
@runtime_checkable
class SupportsLoRA(Protocol):
    """The interface required for all models that support LoRA."""

    supports_lora: ClassVar[Literal[True]] = True
    """
    A flag that indicates this model supports LoRA.

    Note:
        There is no need to redefine this flag if this class is in the
        MRO of your model class.
    """
    is_3d_moe_weight: ClassVar[bool] = False
    is_non_gated_moe: ClassVar[bool] = False
    # The `embedding_module` and `embedding_padding_modules`
    # are empty by default.
    embedding_modules: ClassVar[dict[str, str]] = {}
    packed_modules_mapping: dict[str, list[str]] = {}
    # Module prefixes to skip during LoRA loading (e.g., ["mtp."] for MTP layers)
    lora_skip_prefixes: ClassVar[list[str]] = []
```
**EN:** Class `SupportsLoRA` organizes related behavior for this model family or helper component. It inherits from Protocol.
**CN:** 类 `SupportsLoRA` 用于组织该模型族或辅助组件的相关行为。 它继承自 Protocol。

### Class `_SupportsLoRAType` (lines 561-565)
```python
@runtime_checkable
class _SupportsLoRAType(Protocol):
    supports_lora: Literal[True]

    packed_modules_mapping: dict[str, list[str]]
    embedding_modules: dict[str, str]
```
**EN:** Class `_SupportsLoRAType` organizes related behavior for this model family or helper component. It inherits from Protocol.
**CN:** 类 `_SupportsLoRAType` 用于组织该模型族或辅助组件的相关行为。 它继承自 Protocol。

### Class `SupportsPP` (lines 615-650)
```python
@runtime_checkable
class SupportsPP(Protocol):
    """The interface required for all models that support pipeline parallel."""

    supports_pp: ClassVar[Literal[True]] = True
    """
    A flag that indicates this model supports pipeline parallel.

    Note:
        There is no need to redefine this flag if this class is in the
        MRO of your model class.
    """

    def make_empty_intermediate_tensors(
        self,
        batch_size: int,
        dtype: torch.dtype,
        device: torch.device,
    ) -> IntermediateTensors:
        """Called when PP rank > 0 for profiling purposes."""
        ...

    def forward(
        self,
        input_ids: Tensor | None,
```
**EN:** Class `SupportsPP` organizes related behavior for this model family or helper component. It inherits from Protocol. Key methods include make_empty_intermediate_tensors, forward.
**CN:** 类 `SupportsPP` 用于组织该模型族或辅助组件的相关行为。 它继承自 Protocol。 关键方法包括 make_empty_intermediate_tensors, forward。

### Method `SupportsPP.make_empty_intermediate_tensors` (lines 627-634)
```python
    def make_empty_intermediate_tensors(
        self,
        batch_size: int,
        dtype: torch.dtype,
        device: torch.device,
    ) -> IntermediateTensors:
        """Called when PP rank > 0 for profiling purposes."""
        ...
```
**EN:** Method `SupportsPP.make_empty_intermediate_tensors` encapsulates a focused piece of reusable logic inside this module. The docstring says: Called when PP rank > 0 for profiling purposes.
**CN:** Method `SupportsPP.make_empty_intermediate_tensors` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Called when PP rank > 0 for profiling purposes。

### Method `SupportsPP.forward` (lines 636-650)
```python
    def forward(
        self,
        input_ids: Tensor | None,
        positions: Tensor,
        *,
        intermediate_tensors: IntermediateTensors | None,
    ) -> IntermediateTensors | None:
        """
        Accept [`IntermediateTensors`][vllm.sequence.IntermediateTensors] when
        PP rank > 0.

        Return [`IntermediateTensors`][vllm.sequence.IntermediateTensors] only
        for the last PP rank.
        """
        ...
```
**EN:** Method `SupportsPP.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream. The docstring says: Accept [`IntermediateTensors`][vllm.sequence.IntermediateTensors] when PP rank > 0.
**CN:** Method `SupportsPP.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。 文档字符串表达的核心意思是：Accept [`IntermediateTensors`][vllm.sequence.IntermediateTensors] when PP rank > 0。

### Class `_SupportsPPType` (lines 656-672)
```python
@runtime_checkable
class _SupportsPPType(Protocol):
    supports_pp: Literal[True]

    def make_empty_intermediate_tensors(
        self,
        batch_size: int,
        dtype: torch.dtype,
        device: torch.device,
    ) -> IntermediateTensors: ...

    def forward(
        self,
        input_ids: Tensor | None,
        positions: Tensor,
        *,
        intermediate_tensors: IntermediateTensors | None,
    ) -> Tensor | IntermediateTensors: ...
```
**EN:** Class `_SupportsPPType` organizes related behavior for this model family or helper component. It inherits from Protocol. Key methods include make_empty_intermediate_tensors, forward.
**CN:** 类 `_SupportsPPType` 用于组织该模型族或辅助组件的相关行为。 它继承自 Protocol。 关键方法包括 make_empty_intermediate_tensors, forward。

### Method `_SupportsPPType.make_empty_intermediate_tensors` (lines 659-664)
```python
    def make_empty_intermediate_tensors(
        self,
        batch_size: int,
        dtype: torch.dtype,
        device: torch.device,
    ) -> IntermediateTensors: ...
```
**EN:** Method `_SupportsPPType.make_empty_intermediate_tensors` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `_SupportsPPType.make_empty_intermediate_tensors` 封装了该模块中的一段可复用核心逻辑。

### Method `_SupportsPPType.forward` (lines 666-672)
```python
    def forward(
        self,
        input_ids: Tensor | None,
        positions: Tensor,
        *,
        intermediate_tensors: IntermediateTensors | None,
    ) -> Tensor | IntermediateTensors: ...
```
**EN:** Method `_SupportsPPType.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `_SupportsPPType.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `HasInnerState` (lines 735-743)
```python
@runtime_checkable
class HasInnerState(Protocol):
    """The interface required for all models that has inner state."""

    has_inner_state: ClassVar[Literal[True]] = True
    """
        A flag that indicates this model has inner state.
        Models that has inner state usually need access to the scheduler_config
        for max_num_seqs, etc. True for e.g. both Mamba and Jamba.
    """
```
**EN:** Class `HasInnerState` organizes related behavior for this model family or helper component. It inherits from Protocol.
**CN:** 类 `HasInnerState` 用于组织该模型族或辅助组件的相关行为。 它继承自 Protocol。

### Class `IsAttentionFree` (lines 761-770)
```python
@runtime_checkable
class IsAttentionFree(Protocol):
    """The interface required for all models like Mamba that lack attention,
    but do have state whose size is constant wrt the number of tokens."""

    is_attention_free: ClassVar[Literal[True]] = True
    """
        A flag that indicates this model has no attention.
        Used for block manager and attention backend selection.
        True for Mamba but not Jamba.
    """
```
**EN:** Class `IsAttentionFree` organizes related behavior for this model family or helper component. It inherits from Protocol.
**CN:** 类 `IsAttentionFree` 用于组织该模型族或辅助组件的相关行为。 它继承自 Protocol。

### Class `IsHybrid` (lines 788-827)
```python
@runtime_checkable
class IsHybrid(Protocol):
    """The interface required for all models like Jamba that have both
    attention and mamba blocks, indicates that
    hf_config has 'layers_block_type'"""

    is_hybrid: ClassVar[Literal[True]] = True
    """
        A flag that indicates this model has both mamba and attention blocks
        , also indicates that the model's hf_config has 
        'layers_block_type' """

    @classmethod
    def get_mamba_state_shape_from_config(
        cls,
        vllm_config: VllmConfig,
    ) -> tuple[tuple[int, int], tuple[int, int, int]]:
        """Calculate shapes for Mamba's convolutional and state caches.

        Args:
            vllm_config: vLLM config

        Returns:
            Tuple containing:
            - conv_state_shape: Shape for convolutional state cache
```
**EN:** Class `IsHybrid` organizes related behavior for this model family or helper component. It inherits from Protocol. Key methods include get_mamba_state_shape_from_config, get_mamba_state_copy_func.
**CN:** 类 `IsHybrid` 用于组织该模型族或辅助组件的相关行为。 它继承自 Protocol。 关键方法包括 get_mamba_state_shape_from_config, get_mamba_state_copy_func。

### Method `IsHybrid.get_mamba_state_shape_from_config` (lines 800-814)
```python
    @classmethod
    def get_mamba_state_shape_from_config(
        cls,
        vllm_config: VllmConfig,
    ) -> tuple[tuple[int, int], tuple[int, int, int]]:
        """Calculate shapes for Mamba's convolutional and state caches.

        Args:
            vllm_config: vLLM config

        Returns:
            Tuple containing:
            - conv_state_shape: Shape for convolutional state cache
            - temporal_state_shape: Shape for state space model cache
        """
        ...
```
**EN:** Method `IsHybrid.get_mamba_state_shape_from_config` encapsulates a focused piece of reusable logic inside this module. The docstring says: Calculate shapes for Mamba's convolutional and state caches.
**CN:** Method `IsHybrid.get_mamba_state_shape_from_config` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Calculate shapes for Mamba's convolutional and state caches。

### Method `IsHybrid.get_mamba_state_copy_func` (lines 817-827)
```python
    @classmethod
    def get_mamba_state_copy_func(cls) -> tuple[MambaStateCopyFunc, ...]:
        """Calculate copy-function callables for each Mamba state.

        Returns:
            A tuple of MambaStateCopyFunc callables that correspond, in order,
            to the Mamba states produced by the model. Each callable accepts
            (state, block_ids, cur_block_idx, num_accepted_tokens) and returns
            a MambaCopySpec describing the memory-copy parameters for prefix
            caching in align mode.
        """
        ...
```
**EN:** Method `IsHybrid.get_mamba_state_copy_func` encapsulates a focused piece of reusable logic inside this module. The docstring says: Calculate copy-function callables for each Mamba state.
**CN:** Method `IsHybrid.get_mamba_state_copy_func` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Calculate copy-function callables for each Mamba state。

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
- **Standard library / 标准库**: `import asyncio`, `from collections.abc import (`, `from contextlib import ExitStack, contextmanager, nullcontext`, `from typing import (`, `from typing_extensions import Self, TypeIs`
- **Third-party / 第三方**: `import numpy as np`, `import torch`, `import torch.nn as nn`, `from torch import Tensor`, `from transformers.models.whisper.tokenization_whisper import LANGUAGES`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import ModelConfig, SpeechToTextConfig, SpeechToTextParams`, `from vllm.inputs import PromptType, TokensPrompt`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.mamba.mamba_utils import MambaStateCopyFunc`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.tasks import ScoreType`, `from vllm.utils.collection_utils import common_prefix`, `from vllm.utils.func_utils import supports_kw`, `from .interfaces_base import VllmModel`, `from vllm.config import VllmConfig`, `from vllm.model_executor.models.utils import WeightsMapper`, `from vllm.multimodal.inputs import MultiModalFeatureSpec`
