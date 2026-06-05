# test_encode.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/basic/test_encode.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior. The file defines 2 test(s), 1 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为。它定义了 2 个测试、1 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L10)
```python
import weakref

import pytest

from vllm import LLM, PoolingParams
from vllm.distributed import cleanup_dist_env_and_memory
from vllm.platforms import current_platform
```
**EN:** Imports standard-library modules such as `weakref`, third-party packages like `pytest`, project helpers such as `vllm.LLM`, `vllm.PoolingParams`, `vllm.distributed.cleanup_dist_env_and_memory`.
**CN:** 导入标准库模块（如 `weakref`）、第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.LLM`、`vllm.PoolingParams`、`vllm.distributed.cleanup_dist_env_and_memory`）。

### Module setup / 模块级配置: MODEL_NAME, PROMPTS, TOKEN_IDS (L12-L28)
```python
MODEL_NAME = "intfloat/multilingual-e5-small"

PROMPTS = [
    "Hello, my name is",
    "The president of the United States is",
    "The capital of France is",
    "The future of AI is",
]

TOKEN_IDS = [
    # Using ID={0, 1, 2, 3} results in NaN values,
    # so we add this offset of 1000
    [1000],
    [1000, 1001],
    [1000, 1002, 1001],
    [1000, 1003, 1001, 1002],
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `PROMPTS`, `TOKEN_IDS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`PROMPTS`、`TOKEN_IDS`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: llm (L31-L55)
```python
@pytest.fixture(scope="module")
def llm():
    # ROCm: Use FLEX_ATTENTION backend as it's the only attention backend
    # that supports encoder-only models on ROCm.
    attention_config = None
    if current_platform.is_rocm():
        attention_config = {"backend": "FLEX_ATTENTION"}

    # pytest caches the fixture so we use weakref.proxy to
    # enable garbage collection
    llm = LLM(
        model=MODEL_NAME,
        max_num_batched_tokens=32768,
        tensor_parallel_size=1,
        gpu_memory_utilization=0.75,
        enforce_eager=True,
        seed=0,
        attention_config=attention_config,
    )

    yield weakref.proxy(llm)

    del llm

    cleanup_dist_env_and_memory()
```
**EN:** This fixture prepares `llm` for dependent tests. It touches the core vLLM initialization or engine path directly. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `llm`。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_multiple_pooling_params (L58-L86)
```python
@pytest.mark.skip_global_cleanup
def test_multiple_pooling_params(llm: LLM):
    pooling_params = [
        PoolingParams(),
        PoolingParams(),
        PoolingParams(),
        PoolingParams(),
    ]

    # Multiple PoolingParams should be matched with each prompt
    outputs = llm.encode(PROMPTS, pooling_params=pooling_params, pooling_task="embed")
    assert len(PROMPTS) == len(outputs)

    # Exception raised, if the size of params does not match the size of prompts
    with pytest.raises(ValueError):
        outputs = llm.encode(
            PROMPTS, pooling_params=pooling_params[:3], pooling_task="embed"
        )

    # Single PoolingParams should be applied to every prompt
    single_pooling_params = PoolingParams()
    outputs = llm.encode(
        PROMPTS, pooling_params=single_pooling_params, pooling_task="embed"
    )
    assert len(PROMPTS) == len(outputs)

    # pooling_params is None, default params should be applied
    outputs = llm.encode(PROMPTS, pooling_params=None, pooling_task="embed")
    assert len(PROMPTS) == len(outputs)
```
**EN:** This test validates `test_multiple_pooling_params`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. It checks an expected failure path with `pytest.raises`. The main assertion is `len(PROMPTS) == len(outputs)` and `len(PROMPTS) == len(outputs)`.
**CN:** 这个测试验证 `test_multiple_pooling_params`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 它使用 `pytest.raises` 检查预期失败路径。 核心断言是 `len(PROMPTS) == len(outputs)` and `len(PROMPTS) == len(outputs)`。

### Test / 测试: test_right_side_truncation (L89-L92)
```python
def test_right_side_truncation(llm: LLM):
    # Embeddings models should truncate the end of the prompt
    tokenizer = llm.get_tokenizer()
    assert tokenizer.truncation_side == "right"
```
**EN:** This test validates `test_right_side_truncation`. Key inputs are `llm`. The main assertion is `tokenizer.truncation_side == 'right'`.
**CN:** 这个测试验证 `test_right_side_truncation`。 关键输入包括 `llm`。 核心断言是 `tokenizer.truncation_side == 'right'`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `weakref`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `vllm.LLM`, `vllm.PoolingParams`, `vllm.distributed.cleanup_dist_env_and_memory`, `vllm.platforms.current_platform`
