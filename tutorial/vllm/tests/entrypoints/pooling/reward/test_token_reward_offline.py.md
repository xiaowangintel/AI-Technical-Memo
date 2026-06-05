# test_token_reward_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/pooling/reward/test_token_reward_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior. The file defines 2 test(s), 1 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为。它定义了 2 个测试、1 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L11)
```python
import weakref

import pytest
import torch

from tests.models.utils import softmax
from vllm import LLM, PoolingParams
from vllm.distributed import cleanup_dist_env_and_memory
```
**EN:** Imports standard-library modules such as `weakref`, third-party packages like `pytest`, `torch`, project helpers such as `tests.models.utils.softmax`, `vllm.LLM`, `vllm.PoolingParams`.
**CN:** 导入标准库模块（如 `weakref`）、第三方包（如 `pytest`、`torch`）、项目内辅助模块（如 `tests.models.utils.softmax`、`vllm.LLM`、`vllm.PoolingParams`）。

### Module setup / 模块级配置: MODEL_NAME, prompts (L13-L15)
```python
MODEL_NAME = "internlm/internlm2-1_8b-reward"

prompts = ["The chef prepared a delicious meal."]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`, `prompts`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`、`prompts`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: llm (L18-L36)
```python
@pytest.fixture(scope="module")
def llm():
    # pytest caches the fixture so we use weakref.proxy to
    # enable garbage collection
    llm = LLM(
        model=MODEL_NAME,
        max_num_batched_tokens=32768,
        tensor_parallel_size=1,
        gpu_memory_utilization=0.75,
        enforce_eager=True,
        trust_remote_code=True,
        seed=0,
    )

    yield weakref.proxy(llm)

    del llm

    cleanup_dist_env_and_memory()
```
**EN:** This fixture prepares `llm` for dependent tests. It touches the core vLLM initialization or engine path directly. The fixture yields its resource so teardown can happen after the test completes.
**CN:** 这个 fixture 为依赖它的测试准备 `llm`。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 该 fixture 通过 `yield` 暴露资源，使清理逻辑可以在测试完成后执行。

### Test / 测试: test_config (L39-L43)
```python
@pytest.mark.skip_global_cleanup
def test_config(llm: LLM):
    vllm_config = llm.llm_engine.vllm_config
    assert vllm_config.cache_config.enable_prefix_caching
    assert vllm_config.scheduler_config.enable_chunked_prefill
```
**EN:** This test validates `test_config`. Relevant pytest markers include `skip_global_cleanup`. Key inputs are `llm`. The main assertion is `vllm_config.cache_config.enable_prefix_caching` and `vllm_config.scheduler_config.enable_chunked_prefill`.
**CN:** 这个测试验证 `test_config`。 相关的 pytest 标记包括 `skip_global_cleanup`。 关键输入包括 `llm`。 核心断言是 `vllm_config.cache_config.enable_prefix_caching` and `vllm_config.scheduler_config.enable_chunked_prefill`。

### Test / 测试: test_pooling_params (L46-L67)
```python
def test_pooling_params(llm: LLM):
    def get_outputs(use_activation):
        outputs = llm.reward(
            prompts,
            pooling_params=PoolingParams(use_activation=use_activation),
            use_tqdm=False,
        )
        return torch.cat([x.outputs.data for x in outputs])

    default = get_outputs(use_activation=None)
    w_activation = get_outputs(use_activation=True)
    wo_activation = get_outputs(use_activation=False)

    assert torch.allclose(default, w_activation, atol=1e-2), (
        "Default should use activation."
    )
    assert not torch.allclose(w_activation, wo_activation, atol=1e-2), (
        "wo_activation should not use activation."
    )
    assert torch.allclose(softmax(wo_activation), w_activation, atol=1e-2), (
        "w_activation should be close to activation(wo_activation)."
    )
```
**EN:** This test validates `test_pooling_params`. Key inputs are `llm`. The main assertion is `torch.allclose(default, w_activation, atol=0.01)` and `not torch.allclose(w_activation, wo_activation, atol=0.01)`.
**CN:** 这个测试验证 `test_pooling_params`。 关键输入包括 `llm`。 核心断言是 `torch.allclose(default, w_activation, atol=0.01)` and `not torch.allclose(w_activation, wo_activation, atol=0.01)`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `weakref`
- **Third-party / 第三方**: `pytest`, `torch`
- **Project / 项目内**: `tests.models.utils.softmax`, `vllm.LLM`, `vllm.PoolingParams`, `vllm.distributed.cleanup_dist_env_and_memory`
