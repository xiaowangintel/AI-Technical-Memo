# mteb_score_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling_mteb_test/mteb_score_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared test utilities for pooling or embedding behavior and language-model behavior. The file exposes 15 helper/class block(s) used by nearby tests. / [CN] 为池化或嵌入行为与语言模型行为提供共享测试工具。该文件暴露了 15 个辅助函数/类块，供附近测试复用。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L19)
```python
import tempfile
from pathlib import Path
from typing import Any

import mteb
import numpy as np
import requests
import torch
from mteb.models import ModelMeta
from torch.utils.data import DataLoader

from tests.conftest import HfRunner
from tests.models.utils import (
    RerankModelInfo,
    get_vllm_extra_kwargs,
)
```
**EN:** Imports standard-library modules such as `pathlib.Path`, `tempfile`, `typing.Any`, third-party packages like `mteb`, `mteb.models.ModelMeta`, `numpy`, project helpers such as `tests.conftest.HfRunner`, `tests.models.utils.RerankModelInfo`, `tests.models.utils.get_vllm_extra_kwargs`.
**CN:** 导入标准库模块（如 `pathlib.Path`、`tempfile`、`typing.Any`）、第三方包（如 `mteb`、`mteb.models.ModelMeta`、`numpy`）、项目内辅助模块（如 `tests.conftest.HfRunner`、`tests.models.utils.RerankModelInfo`、`tests.models.utils.get_vllm_extra_kwargs`）。

### Module setup / 模块级配置: MTEB_RERANK_TASKS, MTEB_RERANK_LANGS, MTEB_RERANK_TOL (L22-L50)
```python
MTEB_RERANK_TASKS = ["NFCorpus"]
MTEB_RERANK_LANGS = ["eng"]
MTEB_RERANK_TOL = 2e-3

template_home = (
    Path(__file__).parent.parent.parent.parent.parent
    / "examples/pooling/score/template"
)

_empty_model_meta = ModelMeta(
    loader=None,
    name="vllm/model",
    revision="1",
    release_date=None,
    languages=None,
    framework=[],
# ... 7 lines omitted for brevity ...
    public_training_code=None,
    public_training_data=None,
    use_instructions=None,
    training_datasets=None,
    modalities=["text"],  # 'image' can be added to evaluate multimodal models
)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MTEB_RERANK_TASKS`, `MTEB_RERANK_LANGS`, `MTEB_RERANK_TOL`, `template_home`, `_empty_model_meta`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MTEB_RERANK_TASKS`、`MTEB_RERANK_LANGS`、`MTEB_RERANK_TOL`、`template_home`、`_empty_model_meta`。后续辅助函数和测试会复用这些值，以减少重复。

### Class / 类: MtebCrossEncoderMixin (L53-L54)
```python
class MtebCrossEncoderMixin(mteb.CrossEncoderProtocol):
    mteb_model_meta = _empty_model_meta
```
**EN:** This class groups related scenarios in `MtebCrossEncoderMixin`. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `MtebCrossEncoderMixin` 相关的场景组织在一起。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: VllmMtebCrossEncoder (L57-L88)
```python
class VllmMtebCrossEncoder(MtebCrossEncoderMixin):
    def __init__(self, vllm_model):
        self.llm = vllm_model
        self.rng = np.random.default_rng(seed=42)
        self.chat_template: str | None = getattr(vllm_model, "chat_template", None)

    def predict(
        self,
        inputs1: DataLoader[mteb.types.BatchedInput],
        inputs2: DataLoader[mteb.types.BatchedInput],
        *args,
        **kwargs,
    ) -> np.ndarray:
        queries = [text for batch in inputs1 for text in batch["text"]]
# ... 10 lines omitted for brevity ...
            corpus,
            use_tqdm=False,
            chat_template=self.chat_template,
            tokenization_kwargs={"truncate_prompt_tokens": -1},
        )
        scores = np.array(outputs)
        scores = scores[np.argsort(r)]
        return scores
```
**EN:** This class groups related scenarios in `VllmMtebCrossEncoder`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `__init__`, `predict`.
**CN:** 该类将与 `VllmMtebCrossEncoder` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `__init__`、`predict`。

### Helper method / 辅助方法: VllmMtebCrossEncoder.__init__ (L58-L61)
```python
    def __init__(self, vllm_model):
        self.llm = vllm_model
        self.rng = np.random.default_rng(seed=42)
        self.chat_template: str | None = getattr(vllm_model, "chat_template", None)
```
**EN:** This helper encapsulates reusable logic in `VllmMtebCrossEncoder.__init__`. Key inputs are `vllm_model`.
**CN:** 这个辅助函数将可复用逻辑封装在 `VllmMtebCrossEncoder.__init__` 中。 关键输入包括 `vllm_model`。

### Helper method / 辅助方法: VllmMtebCrossEncoder.predict (L63-L88)
```python
    def predict(
        self,
        inputs1: DataLoader[mteb.types.BatchedInput],
        inputs2: DataLoader[mteb.types.BatchedInput],
        *args,
        **kwargs,
    ) -> np.ndarray:
        queries = [text for batch in inputs1 for text in batch["text"]]
        corpus = [text for batch in inputs2 for text in batch["text"]]

        # Hoping to discover potential scheduling
        # issues by randomizing the order.
        r = self.rng.permutation(len(queries))
        queries = [queries[i] for i in r]
        corpus = [corpus[i] for i in r]

        outputs = self.llm.score(
            queries,
            corpus,
            use_tqdm=False,
            chat_template=self.chat_template,
            tokenization_kwargs={"truncate_prompt_tokens": -1},
        )
        scores = np.array(outputs)
        scores = scores[np.argsort(r)]
        return scores
```
**EN:** This helper encapsulates reusable logic in `VllmMtebCrossEncoder.predict`. Key inputs are `inputs1`, `inputs2`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `VllmMtebCrossEncoder.predict` 中。 关键输入包括 `inputs1`、`inputs2`。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: ScoreClientMtebEncoder (L91-L125)
```python
class ScoreClientMtebEncoder(MtebCrossEncoderMixin):
    mteb_model_meta = _empty_model_meta

    def __init__(self, model_name: str, url):
        self.model_name = model_name
        self.url = url

    def predict(
        self,
        inputs1: DataLoader[mteb.types.BatchedInput],
        inputs2: DataLoader[mteb.types.BatchedInput],
        *args,
        **kwargs,
    ) -> np.ndarray:
# ... 13 lines omitted for brevity ...
            json={
                "model": self.model_name,
                "queries": query,
                "documents": corpus,
                "truncate_prompt_tokens": -1,
            },
        ).json()
        return response["data"][0]["score"]
```
**EN:** This class groups related scenarios in `ScoreClientMtebEncoder`. It contains 0 test method(s) and 3 supporting method(s). Representative methods include `__init__`, `predict`.
**CN:** 该类将与 `ScoreClientMtebEncoder` 相关的场景组织在一起。 它包含 0 个测试方法和 3 个辅助方法。 代表性方法包括 `__init__`、`predict`。

### Helper method / 辅助方法: ScoreClientMtebEncoder.__init__ (L94-L96)
```python
    def __init__(self, model_name: str, url):
        self.model_name = model_name
        self.url = url
```
**EN:** This helper encapsulates reusable logic in `ScoreClientMtebEncoder.__init__`. Key inputs are `model_name`, `url`.
**CN:** 这个辅助函数将可复用逻辑封装在 `ScoreClientMtebEncoder.__init__` 中。 关键输入包括 `model_name`、`url`。

### Helper method / 辅助方法: ScoreClientMtebEncoder.predict (L98-L113)
```python
    def predict(
        self,
        inputs1: DataLoader[mteb.types.BatchedInput],
        inputs2: DataLoader[mteb.types.BatchedInput],
        *args,
        **kwargs,
    ) -> np.ndarray:
        queries = [text for batch in inputs1 for text in batch["text"]]
        full_corpus = [text for batch in inputs2 for text in batch["text"]]

        outputs = []
        for query, corpus in zip(queries, full_corpus):
            outputs.append(self.get_score(query, corpus))

        scores = np.array(outputs)
        return scores
```
**EN:** This helper encapsulates reusable logic in `ScoreClientMtebEncoder.predict`. Key inputs are `inputs1`, `inputs2`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `ScoreClientMtebEncoder.predict` 中。 关键输入包括 `inputs1`、`inputs2`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper method / 辅助方法: ScoreClientMtebEncoder.get_score (L115-L125)
```python
    def get_score(self, query, corpus):
        response = requests.post(
            self.url,
            json={
                "model": self.model_name,
                "queries": query,
                "documents": corpus,
                "truncate_prompt_tokens": -1,
            },
        ).json()
        return response["data"][0]["score"]
```
**EN:** This helper encapsulates reusable logic in `ScoreClientMtebEncoder.get_score`. Key inputs are `query`, `corpus`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `ScoreClientMtebEncoder.get_score` 中。 关键输入包括 `query`、`corpus`。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: RerankClientMtebEncoder (L128-L139)
```python
class RerankClientMtebEncoder(ScoreClientMtebEncoder):
    def get_score(self, query, corpus):
        response = requests.post(
            self.url,
            json={
                "model": self.model_name,
                "query": query,
                "documents": [corpus],
                "truncate_prompt_tokens": -1,
            },
        ).json()
        return response["results"][0]["relevance_score"]
```
**EN:** This class groups related scenarios in `RerankClientMtebEncoder`. It contains 0 test method(s) and 1 supporting method(s). Representative methods include `get_score`.
**CN:** 该类将与 `RerankClientMtebEncoder` 相关的场景组织在一起。 它包含 0 个测试方法和 1 个辅助方法。 代表性方法包括 `get_score`。

### Helper method / 辅助方法: RerankClientMtebEncoder.get_score (L129-L139)
```python
    def get_score(self, query, corpus):
        response = requests.post(
            self.url,
            json={
                "model": self.model_name,
                "query": query,
                "documents": [corpus],
                "truncate_prompt_tokens": -1,
            },
        ).json()
        return response["results"][0]["relevance_score"]
```
**EN:** This helper encapsulates reusable logic in `RerankClientMtebEncoder.get_score`. Key inputs are `query`, `corpus`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `RerankClientMtebEncoder.get_score` 中。 关键输入包括 `query`、`corpus`。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: HFMtebCrossEncoder (L142-L183)
```python
class HFMtebCrossEncoder(MtebCrossEncoderMixin, HfRunner):
    chat_template: str | None = None

    def __init__(self, model_name: str, dtype: str = "auto", **kwargs: Any) -> None:
        HfRunner.__init__(
            self, model_name=model_name, is_cross_encoder=True, dtype=dtype, **kwargs
        )

    @torch.no_grad
    def predict(
        self,
        inputs1: DataLoader[mteb.types.BatchedInput],
        inputs2: DataLoader[mteb.types.BatchedInput],
        *args,
# ... 20 lines omitted for brevity ...
                prompts.append(prompt)
            outputs_list = HfRunner.classify(self, prompts)
            scores = np.array(outputs_list).squeeze(-1)
            return scores
        else:
            prompts = list(zip(queries, corpus))
            outputs_tensor = HfRunner.predict(self, prompts, show_progress_bar=False)
            return outputs_tensor.cpu().numpy()
```
**EN:** This class groups related scenarios in `HFMtebCrossEncoder`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `__init__`, `predict`.
**CN:** 该类将与 `HFMtebCrossEncoder` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `__init__`、`predict`。

### Helper method / 辅助方法: HFMtebCrossEncoder.__init__ (L145-L148)
```python
    def __init__(self, model_name: str, dtype: str = "auto", **kwargs: Any) -> None:
        HfRunner.__init__(
            self, model_name=model_name, is_cross_encoder=True, dtype=dtype, **kwargs
        )
```
**EN:** This helper encapsulates reusable logic in `HFMtebCrossEncoder.__init__`. Key inputs are `model_name`, `dtype`.
**CN:** 这个辅助函数将可复用逻辑封装在 `HFMtebCrossEncoder.__init__` 中。 关键输入包括 `model_name`、`dtype`。

### Helper method / 辅助方法: HFMtebCrossEncoder.predict (L150-L183)
```python
    @torch.no_grad
    def predict(
        self,
        inputs1: DataLoader[mteb.types.BatchedInput],
        inputs2: DataLoader[mteb.types.BatchedInput],
        *args,
        **kwargs,
    ) -> np.ndarray:
        queries = [text for batch in inputs1 for text in batch["text"]]
        corpus = [text for batch in inputs2 for text in batch["text"]]

        if self.chat_template is not None:
            tokenizer = self.model.tokenizer
            prompts = []
            for query, document in zip(queries, corpus):
                conversation = [
                    {"role": "query", "content": query},
                    {"role": "document", "content": document},
# ... 8 lines omitted for brevity ...
                prompts.append(prompt)
            outputs_list = HfRunner.classify(self, prompts)
            scores = np.array(outputs_list).squeeze(-1)
            return scores
        else:
            prompts = list(zip(queries, corpus))
            outputs_tensor = HfRunner.predict(self, prompts, show_progress_bar=False)
            return outputs_tensor.cpu().numpy()
```
**EN:** This helper encapsulates reusable logic in `HFMtebCrossEncoder.predict`. Key inputs are `inputs1`, `inputs2`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `HFMtebCrossEncoder.predict` 中。 关键输入包括 `inputs1`、`inputs2`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: run_mteb_rerank (L186-L224)
```python
def run_mteb_rerank(cross_encoder: mteb.CrossEncoderProtocol, tasks, languages):
    with tempfile.TemporaryDirectory() as prediction_folder:
        bm25s = mteb.get_model("bm25s")
        eval_splits = ["test"]

        mteb_tasks: list[mteb.abstasks.AbsTaskRetrieval] = mteb.get_tasks(
            tasks=tasks, languages=languages, eval_splits=eval_splits
        )
        for task in mteb_tasks:
            if not task.data_loaded:
                task.load_data()

        mteb.evaluate(
            bm25s,
            mteb_tasks,
            prediction_folder=prediction_folder,
            show_progress_bar=False,
            # don't save results for test runs
# ... 13 lines omitted for brevity ...
        results = mteb.evaluate(
            cross_encoder,
            second_stage_tasks,
            show_progress_bar=False,
            cache=None,
        )
        main_score = results[0].scores["test"][0]["main_score"]
    return main_score
```
**EN:** This helper encapsulates reusable logic in `run_mteb_rerank`. Key inputs are `cross_encoder`, `tasks`, `languages`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_mteb_rerank` 中。 关键输入包括 `cross_encoder`、`tasks`、`languages`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: mteb_test_rerank_models (L227-L311)
```python
def mteb_test_rerank_models(
    vllm_runner,
    model_info: RerankModelInfo,
    hf_runner=HFMtebCrossEncoder,
    vllm_extra_kwargs=None,
    vllm_mteb_encoder=VllmMtebCrossEncoder,
    atol=MTEB_RERANK_TOL,
):
    vllm_extra_kwargs = get_vllm_extra_kwargs(model_info, vllm_extra_kwargs)

    # Maybe load chat_template.
    chat_template: str | None = None
    if model_info.chat_template_name is not None:
        chat_template = (template_home / model_info.chat_template_name).read_text()

    with vllm_runner(
        model_info.name,
        revision=model_info.revision,
# ... 59 lines omitted for brevity ...
    print("Model:", model_info.name)
    print("VLLM:", f"dtype:{vllm_dtype}", f"head_dtype:{head_dtype}", vllm_main_score)
    print("SentenceTransformers:", st_dtype, st_main_score)
    print("Difference:", st_main_score - vllm_main_score)

    # We are not concerned that the vllm mteb results are better
    # than SentenceTransformers, so we only perform one-sided testing.
    assert st_main_score - vllm_main_score < atol
```
**EN:** This helper encapsulates reusable logic in `mteb_test_rerank_models`. Key inputs are `vllm_runner`, `model_info`, `hf_runner`, `vllm_extra_kwargs`, `vllm_mteb_encoder`, `atol`. The main assertion is `st_main_score - vllm_main_score < atol` and `model_config.hf_config.num_labels == 1`.
**CN:** 这个辅助函数将可复用逻辑封装在 `mteb_test_rerank_models` 中。 关键输入包括 `vllm_runner`、`model_info`、`hf_runner`、`vllm_extra_kwargs`、`vllm_mteb_encoder`、`atol`。 核心断言是 `st_main_score - vllm_main_score < atol` and `model_config.hf_config.num_labels == 1`。

## Key Concepts / 关键概念
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。
- **EN:** Shared metadata and helper utilities keep nearby tests focused on behavior.
  **CN:** 共享元数据与辅助工具让相邻测试更专注于行为本身。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `pathlib.Path`, `tempfile`, `typing.Any`
- **Third-party / 第三方**: `mteb`, `mteb.models.ModelMeta`, `numpy`, `requests`, `torch`, `torch.utils.data.DataLoader`
- **Project / 项目内**: `tests.conftest.HfRunner`, `tests.models.utils.RerankModelInfo`, `tests.models.utils.get_vllm_extra_kwargs`
