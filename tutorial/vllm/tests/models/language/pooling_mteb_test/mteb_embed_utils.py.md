# mteb_embed_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling_mteb_test/mteb_embed_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared test utilities for pooling or embedding behavior and language-model behavior. The file exposes 14 helper/class block(s) used by nearby tests. / [CN] 为池化或嵌入行为与语言模型行为提供共享测试工具。该文件暴露了 14 个辅助函数/类块，供附近测试复用。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L16)
```python
import mteb
import numpy as np
import torch
from mteb.models import ModelMeta
from mteb.types import Array
from torch.utils.data import DataLoader

import tests.ci_envs as ci_envs
from tests.models.utils import (
    EmbedModelInfo,
    check_embeddings_close,
    get_vllm_extra_kwargs,
)
```
**EN:** Imports third-party packages like `mteb`, `mteb.models.ModelMeta`, `mteb.types.Array`, project helpers such as `tests.ci_envs`, `tests.models.utils.EmbedModelInfo`, `tests.models.utils.check_embeddings_close`.
**CN:** 导入第三方包（如 `mteb`、`mteb.models.ModelMeta`、`mteb.types.Array`）、项目内辅助模块（如 `tests.ci_envs`、`tests.models.utils.EmbedModelInfo`、`tests.models.utils.check_embeddings_close`）。

### Module setup / 模块级配置: MTEB_EMBED_TASKS, MTEB_EMBED_TOL, _empty_model_meta (L23-L46)
```python
MTEB_EMBED_TASKS = ["STS12"]
MTEB_EMBED_TOL = 5e-4


_empty_model_meta = ModelMeta(
    loader=None,
    name="vllm/model",
    revision="1",
    release_date=None,
    languages=None,
    framework=[],
    similarity_fn_name=None,
    n_parameters=None,
    memory_usage_mb=None,
    max_tokens=None,
    embed_dim=None,
    license=None,
    open_weights=None,
    public_training_code=None,
    public_training_data=None,
    use_instructions=None,
    training_datasets=None,
    modalities=["text"],  # 'image' can be added to evaluate multimodal models
)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MTEB_EMBED_TASKS`, `MTEB_EMBED_TOL`, `_empty_model_meta`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MTEB_EMBED_TASKS`、`MTEB_EMBED_TOL`、`_empty_model_meta`。后续辅助函数和测试会复用这些值，以减少重复。

### Class / 类: MtebEmbedMixin (L49-L74)
```python
class MtebEmbedMixin(mteb.EncoderProtocol):
    mteb_model_meta = _empty_model_meta

    def similarity(
        self,
        embeddings1: np.ndarray,
        embeddings2: np.ndarray,
    ) -> np.ndarray:
        # Cosine similarity
        norm1 = np.linalg.norm(embeddings1, axis=1, keepdims=True)
        norm2 = np.linalg.norm(embeddings2, axis=1, keepdims=True)
        sim = np.dot(embeddings1, embeddings2.T) / (norm1 * norm2.T)
        return sim

    def similarity_pairwise(
        self,
        embeddings1: Array,
        embeddings2: Array,
    ) -> Array:
        # Cosine similarity
        norm1 = np.linalg.norm(embeddings1, axis=1, keepdims=True)
        norm2 = np.linalg.norm(embeddings2, axis=1, keepdims=True)
        sim = np.sum(embeddings1 * embeddings2, axis=1) / (
            norm1.flatten() * norm2.flatten()
        )
        return sim
```
**EN:** This class groups related scenarios in `MtebEmbedMixin`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `similarity`, `similarity_pairwise`.
**CN:** 该类将与 `MtebEmbedMixin` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `similarity`、`similarity_pairwise`。

### Helper method / 辅助方法: MtebEmbedMixin.similarity (L52-L61)
```python
    def similarity(
        self,
        embeddings1: np.ndarray,
        embeddings2: np.ndarray,
    ) -> np.ndarray:
        # Cosine similarity
        norm1 = np.linalg.norm(embeddings1, axis=1, keepdims=True)
        norm2 = np.linalg.norm(embeddings2, axis=1, keepdims=True)
        sim = np.dot(embeddings1, embeddings2.T) / (norm1 * norm2.T)
        return sim
```
**EN:** This helper encapsulates reusable logic in `MtebEmbedMixin.similarity`. Key inputs are `embeddings1`, `embeddings2`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `MtebEmbedMixin.similarity` 中。 关键输入包括 `embeddings1`、`embeddings2`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper method / 辅助方法: MtebEmbedMixin.similarity_pairwise (L63-L74)
```python
    def similarity_pairwise(
        self,
        embeddings1: Array,
        embeddings2: Array,
    ) -> Array:
        # Cosine similarity
        norm1 = np.linalg.norm(embeddings1, axis=1, keepdims=True)
        norm2 = np.linalg.norm(embeddings2, axis=1, keepdims=True)
        sim = np.sum(embeddings1 * embeddings2, axis=1) / (
            norm1.flatten() * norm2.flatten()
        )
        return sim
```
**EN:** This helper encapsulates reusable logic in `MtebEmbedMixin.similarity_pairwise`. Key inputs are `embeddings1`, `embeddings2`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `MtebEmbedMixin.similarity_pairwise` 中。 关键输入包括 `embeddings1`、`embeddings2`。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: HfMtebEncoder (L77-L88)
```python
class HfMtebEncoder(MtebEmbedMixin):
    def __init__(self, model):
        self.model = model

    def encode(
        self,
        inputs: DataLoader[mteb.types.BatchedInput],
        *args,
        **kwargs,
    ) -> np.ndarray:
        sentences = [text for batch in inputs for text in batch["text"]]
        return self.model.encode(sentences)
```
**EN:** This class groups related scenarios in `HfMtebEncoder`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `__init__`, `encode`.
**CN:** 该类将与 `HfMtebEncoder` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `__init__`、`encode`。

### Helper method / 辅助方法: HfMtebEncoder.__init__ (L78-L79)
```python
    def __init__(self, model):
        self.model = model
```
**EN:** This helper encapsulates reusable logic in `HfMtebEncoder.__init__`. Key inputs are `model`.
**CN:** 这个辅助函数将可复用逻辑封装在 `HfMtebEncoder.__init__` 中。 关键输入包括 `model`。

### Helper method / 辅助方法: HfMtebEncoder.encode (L81-L88)
```python
    def encode(
        self,
        inputs: DataLoader[mteb.types.BatchedInput],
        *args,
        **kwargs,
    ) -> np.ndarray:
        sentences = [text for batch in inputs for text in batch["text"]]
        return self.model.encode(sentences)
```
**EN:** This helper encapsulates reusable logic in `HfMtebEncoder.encode`. Key inputs are `inputs`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `HfMtebEncoder.encode` 中。 关键输入包括 `inputs`。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: VllmMtebEncoder (L91-L115)
```python
class VllmMtebEncoder(MtebEmbedMixin):
    def __init__(self, vllm_model, prompt_prefix: str | None = None):
        self.llm = vllm_model
        self.rng = np.random.default_rng(seed=42)
        self.prompt_prefix = prompt_prefix

    def encode(
        self,
        inputs: DataLoader[mteb.types.BatchedInput],
        *args,
        **kwargs,
    ) -> np.ndarray:
        # Hoping to discover potential scheduling
        # issues by randomizing the order.
        sentences = [
            self.prompt_prefix + text if self.prompt_prefix else text
            for batch in inputs
            for text in batch["text"]
        ]
        r = self.rng.permutation(len(sentences))
        sentences = [sentences[i] for i in r]
        outputs = self.llm.embed(sentences, use_tqdm=False)
        embeds = np.array(outputs)
        embeds = embeds[np.argsort(r)]
        return embeds
```
**EN:** This class groups related scenarios in `VllmMtebEncoder`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `__init__`, `encode`.
**CN:** 该类将与 `VllmMtebEncoder` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `__init__`、`encode`。

### Helper method / 辅助方法: VllmMtebEncoder.__init__ (L92-L95)
```python
    def __init__(self, vllm_model, prompt_prefix: str | None = None):
        self.llm = vllm_model
        self.rng = np.random.default_rng(seed=42)
        self.prompt_prefix = prompt_prefix
```
**EN:** This helper encapsulates reusable logic in `VllmMtebEncoder.__init__`. Key inputs are `vllm_model`, `prompt_prefix`.
**CN:** 这个辅助函数将可复用逻辑封装在 `VllmMtebEncoder.__init__` 中。 关键输入包括 `vllm_model`、`prompt_prefix`。

### Helper method / 辅助方法: VllmMtebEncoder.encode (L97-L115)
```python
    def encode(
        self,
        inputs: DataLoader[mteb.types.BatchedInput],
        *args,
        **kwargs,
    ) -> np.ndarray:
        # Hoping to discover potential scheduling
        # issues by randomizing the order.
        sentences = [
            self.prompt_prefix + text if self.prompt_prefix else text
            for batch in inputs
            for text in batch["text"]
        ]
        r = self.rng.permutation(len(sentences))
        sentences = [sentences[i] for i in r]
        outputs = self.llm.embed(sentences, use_tqdm=False)
        embeds = np.array(outputs)
        embeds = embeds[np.argsort(r)]
        return embeds
```
**EN:** This helper encapsulates reusable logic in `VllmMtebEncoder.encode`. Key inputs are `inputs`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `VllmMtebEncoder.encode` 中。 关键输入包括 `inputs`。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: OpenAIClientMtebEncoder (L118-L142)
```python
class OpenAIClientMtebEncoder(MtebEmbedMixin):
    def __init__(self, model_name: str, client):
        self.model_name = model_name
        self.client = client
        self.rng = np.random.default_rng(seed=42)

    def encode(
        self,
        inputs: DataLoader[mteb.types.BatchedInput],
        *args,
        **kwargs,
    ) -> np.ndarray:
        # Hoping to discover potential scheduling
        # issues by randomizing the order.
        sentences = [text for batch in inputs for text in batch["text"]]
        r = self.rng.permutation(len(sentences))
        sentences = [sentences[i] for i in r]

        embeddings = self.client.embeddings.create(
            model=self.model_name, input=sentences
        )
        outputs = [d.embedding for d in embeddings.data]
        embeds = np.array(outputs)
        embeds = embeds[np.argsort(r)]
        return embeds
```
**EN:** This class groups related scenarios in `OpenAIClientMtebEncoder`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `__init__`, `encode`.
**CN:** 该类将与 `OpenAIClientMtebEncoder` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `__init__`、`encode`。

### Helper method / 辅助方法: OpenAIClientMtebEncoder.__init__ (L119-L122)
```python
    def __init__(self, model_name: str, client):
        self.model_name = model_name
        self.client = client
        self.rng = np.random.default_rng(seed=42)
```
**EN:** This helper encapsulates reusable logic in `OpenAIClientMtebEncoder.__init__`. Key inputs are `model_name`, `client`.
**CN:** 这个辅助函数将可复用逻辑封装在 `OpenAIClientMtebEncoder.__init__` 中。 关键输入包括 `model_name`、`client`。

### Helper method / 辅助方法: OpenAIClientMtebEncoder.encode (L124-L142)
```python
    def encode(
        self,
        inputs: DataLoader[mteb.types.BatchedInput],
        *args,
        **kwargs,
    ) -> np.ndarray:
        # Hoping to discover potential scheduling
        # issues by randomizing the order.
        sentences = [text for batch in inputs for text in batch["text"]]
        r = self.rng.permutation(len(sentences))
        sentences = [sentences[i] for i in r]

        embeddings = self.client.embeddings.create(
            model=self.model_name, input=sentences
        )
        outputs = [d.embedding for d in embeddings.data]
        embeds = np.array(outputs)
        embeds = embeds[np.argsort(r)]
        return embeds
```
**EN:** This helper encapsulates reusable logic in `OpenAIClientMtebEncoder.encode`. Key inputs are `inputs`. It drives client-facing request creation through the API surface under test. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `OpenAIClientMtebEncoder.encode` 中。 关键输入包括 `inputs`。 它通过被测 API 表面触发面向客户端的请求创建流程。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: run_mteb_embed_task (L145-L155)
```python
def run_mteb_embed_task(encoder: mteb.EncoderProtocol, tasks):
    tasks = mteb.get_tasks(tasks=tasks)
    results = mteb.evaluate(
        encoder,
        tasks,
        cache=None,
        show_progress_bar=False,
    )

    main_score = results[0].scores["test"][0]["main_score"]
    return main_score
```
**EN:** This helper encapsulates reusable logic in `run_mteb_embed_task`. Key inputs are `encoder`, `tasks`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_mteb_embed_task` 中。 关键输入包括 `encoder`、`tasks`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: mteb_test_embed_models (L158-L258)
```python
def mteb_test_embed_models(
    hf_runner,
    vllm_runner,
    model_info: EmbedModelInfo,
    vllm_extra_kwargs=None,
    hf_model_callback=None,
    atol=MTEB_EMBED_TOL,
    prompt_prefix: str | None = None,
):
    vllm_extra_kwargs = get_vllm_extra_kwargs(model_info, vllm_extra_kwargs)

    # Test embed_dims, isnan and whether to use normalize
    example_prompts = ["The chef prepared a delicious meal." * 1000]

    with vllm_runner(
        model_info.name,
        revision=model_info.revision,
        runner="pooling",
# ... 75 lines omitted for brevity ...
    print("Model:", model_info.name)
    print("VLLM:", f"dtype:{vllm_dtype}", f"head_dtype:{head_dtype}", vllm_main_score)
    print("SentenceTransformers:", st_dtype, st_main_score)
    print("Difference:", st_main_score - vllm_main_score)

    # We are not concerned that the vllm mteb results are better
    # than SentenceTransformers, so we only perform one-sided testing.
    assert st_main_score - vllm_main_score < atol
```
**EN:** This helper encapsulates reusable logic in `mteb_test_embed_models`. Key inputs are `hf_runner`, `vllm_runner`, `model_info`, `vllm_extra_kwargs`, `hf_model_callback`, `atol`. The main assertion is `st_main_score - vllm_main_score < atol` and `not torch.any(torch.isnan(outputs_tensor))`.
**CN:** 这个辅助函数将可复用逻辑封装在 `mteb_test_embed_models` 中。 关键输入包括 `hf_runner`、`vllm_runner`、`model_info`、`vllm_extra_kwargs`、`hf_model_callback`、`atol`。 核心断言是 `st_main_score - vllm_main_score < atol` and `not torch.any(torch.isnan(outputs_tensor))`。

## Key Concepts / 关键概念
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。
- **EN:** Shared metadata and helper utilities keep nearby tests focused on behavior.
  **CN:** 共享元数据与辅助工具让相邻测试更专注于行为本身。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `mteb`, `mteb.models.ModelMeta`, `mteb.types.Array`, `numpy`, `torch`, `torch.utils.data.DataLoader`
- **Project / 项目内**: `tests.ci_envs`, `tests.models.utils.EmbedModelInfo`, `tests.models.utils.check_embeddings_close`, `tests.models.utils.get_vllm_extra_kwargs`
