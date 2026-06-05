# test_pooling_chunked_prefill.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/e2e/general/test_pooling_chunked_prefill.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises end-to-end `pooling chunked prefill` scenarios in the v1 test suite. / 在 v1 测试套件中覆盖端到端的 `pooling chunked prefill` 场景。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-6)
```python
import pytest
import torch.nn as nn

from vllm.platforms import current_platform
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch.nn`. vLLM modules under test include `vllm.platforms`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch.nn`。 被测试的 vLLM 模块包括 `vllm.platforms`。

### Module state / 模块级状态 (lines 8-37)
```python
prompt = """
Generals gathered in their masses
Just like witches at black masses
Evil minds that plot destruction
Sorcerer of death's construction
In the fields, the bodies burning
As the war machine keeps turning
Death and hatred to mankind
Poisoning their brainwashed minds
Oh, Lord, yeah

Politicians hide themselves away
They only started the war
Why should they go out to fight?
They leave that all to the poor, yeah
Time will tell on their power minds
Making war just for fun
Treating people just like pawns in chess
Wait till their judgment day comes, yeah

Now, in darkness, world stops turning
Ashes where their bodies burning
No more war pigs have the power
Hand of God has struck the hour
Day of Judgment, God is calling
On their knees, the war pigs crawling
Begging mercies for their sins
Satan, laughing, spreads his wings
Oh, Lord, yeah
"""
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `prompt`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`prompt`。

### WrapperPooler (lines 40-55)
```python
class WrapperPooler(nn.Module):
    def __init__(self, pooler):
        super().__init__()
        self.pooler = pooler
        self.chunks = []

    def get_pooling_updates(self, task):
        return self.pooler.get_pooling_updates(task)

    def forward(
        self,
        hidden_states,
        pooling_metadata,
    ):
        self.chunks.append(hidden_states.shape[0])
        return self.pooler(hidden_states, pooling_metadata)
```
**EN:** Class `WrapperPooler` groups 0 test method(s) and 3 helper/fixture method(s). Bases: `nn.Module`.
**CN:** 类 `WrapperPooler` 组织了 0 个测试方法，以及 3 个辅助或 fixture 方法。 基类：`nn.Module`。

### inject_pooler (lines 58-61)
```python
def inject_pooler(self):
    model = self.get_model()
    wrapper = WrapperPooler(model.pooler)
    model.pooler = wrapper
```
**EN:** Helper function `inject_pooler` encapsulates reusable logic for `inject pooler`. Key calls include `self.get_model, WrapperPooler`.
**CN:** 辅助函数 `inject_pooler` 封装了与 `inject pooler` 相关的可复用逻辑。 关键调用包括 `self.get_model, WrapperPooler`。

### retrieve_chunks (lines 64-68)
```python
def retrieve_chunks(self):
    model = self.get_model()
    chunks = model.pooler.chunks
    model.pooler.chunks = []
    return chunks
```
**EN:** Helper function `retrieve_chunks` encapsulates reusable logic for `retrieve chunks`. Key calls include `self.get_model`.
**CN:** 辅助函数 `retrieve_chunks` 封装了与 `retrieve chunks` 相关的可复用逻辑。 关键调用包括 `self.get_model`。

### test_pooling_chunked_prefill (lines 72-121)
```python
def test_pooling_chunked_prefill(vllm_runner, monkeypatch):
    """Test chunked prefill for pooling models with LastPool."""

    with monkeypatch.context() as m:
        m.setenv("VLLM_ALLOW_INSECURE_SERIALIZATION", "1")
        model_id = "Qwen/Qwen3-Embedding-0.6B"
        chunk_size = 10
        # Set chunking parameters to force chunked prefill
        # Note: Chunked prefill is automatically handled by vLLM
        # internally based on the model size and prompt
        with vllm_runner(
            model_id,
            runner="pooling",
            long_prefill_token_threshold=chunk_size,
            tensor_parallel_size=1,
            enforce_eager=True,
    # ... excerpt omitted for brevity ...
            expected_chunks = [chunk_size] * full_chunks
                expected_chunks.append(last_chunk)
        assert len(chunks) > 1
        assert chunks == expected_chunks
            llm.embed([prompt])
            chunks = llm.get_llm().llm_engine.collective_rpc(retrieve_chunks)[0]
        # Check that PoolerWrapper was called and no chunks were received
        assert len(chunks) == 1
        assert chunks[0] == prompt_len
```
**EN:** Test case covering `pooling chunked prefill`. Inputs/fixtures: `vllm_runner, monkeypatch`. It exercises `mark.skipif, monkeypatch.context, m.setenv, current_platform.is_cuda, vllm_runner, llm_engine.collective_rpc`. The body contains 4 explicit assertion(s).
**CN:** 该代码块是覆盖 `pooling chunked prefill` 的测试用例。 输入或 fixture：`vllm_runner, monkeypatch`。 该测试会调用 `mark.skipif, monkeypatch.context, m.setenv, current_platform.is_cuda, vllm_runner, llm_engine.collective_rpc`。 代码主体包含 4 个显式断言。

### test_pooling_prefix_cache (lines 125-168)
```python
def test_pooling_prefix_cache(vllm_runner, monkeypatch):
    """Test chunked prefill for pooling models with LastPool."""

    verses = prompt.split("\n\n")
    with monkeypatch.context() as m:
        m.setenv("VLLM_ALLOW_INSECURE_SERIALIZATION", "1")
        model_id = "Qwen/Qwen3-Embedding-0.6B"
        with vllm_runner(
            model_id,
            runner="pooling",
            enable_prefix_caching=True,
            tensor_parallel_size=1,
            enforce_eager=True,
        ) as llm:
            llm.get_llm().llm_engine.collective_rpc(inject_pooler)
            tokenizer = llm.get_llm().get_tokenizer()
    # ... excerpt omitted for brevity ...
            assert len(chunks) == 1
            assert chunks[0] == prompt1_len
            assert chunks[0] <= prompt1_len
            assert chunks[0] < prompt2_len
            vllm_config = llm.get_llm().llm_engine.vllm_config
            cache_config = vllm_config.cache_config
            print(f"{cache_config=}")
            # Prefixes are cached in blocks
            assert (prompt2_len - chunks[0]) % cache_config.block_size == 0
```
**EN:** Test case covering `pooling prefix cache`. Inputs/fixtures: `vllm_runner, monkeypatch`. It exercises `mark.skipif, prompt.split, monkeypatch.context, m.setenv, current_platform.is_cuda, vllm_runner`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `pooling prefix 缓存` 的测试用例。 输入或 fixture：`vllm_runner, monkeypatch`。 该测试会调用 `mark.skipif, prompt.split, monkeypatch.context, m.setenv, current_platform.is_cuda, vllm_runner`。 代码主体包含 6 个显式断言。

## Key Concepts / 关键概念
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Cache allocation, reuse, and invalidation behavior
- **CN:** 缓存分配、复用与失效行为
- **EN:** End-to-end integration rather than isolated units
- **CN:** 端到端集成而非孤立单元

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch.nn`.
- **CN:** 外部库：`pytest, torch.nn`。
- **EN:** vLLM modules under test: `vllm.platforms`.
- **CN:** 被测试的 vLLM 模块：`vllm.platforms`。
