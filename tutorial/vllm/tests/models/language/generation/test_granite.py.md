# test_granite.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/generation/test_granite.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers language-model behavior and model-facing behavior. The file defines 1 test(s), 0 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖语言模型行为与面向模型的行为。它定义了 1 个测试、0 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L5)
```python
import pytest

from ...utils import check_logprobs_close
```
**EN:** Imports third-party packages like `pytest`, project helpers such as `...utils.check_logprobs_close`.
**CN:** 导入第三方包（如 `pytest`）、项目内辅助模块（如 `...utils.check_logprobs_close`）。

### Module setup / 模块级配置: MODELS (L7-L11)
```python
MODELS = [
    # TODO(sang): Sliding window should be tested separately.
    "ibm/PowerLM-3b",
    "ibm/PowerMoE-3b",
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODELS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODELS`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_models (L14-L42)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["bfloat16"])
@pytest.mark.parametrize("max_tokens", [64])
@pytest.mark.parametrize("num_logprobs", [5])
@pytest.mark.cpu_model
def test_models(
    hf_runner,
    vllm_runner,
    example_prompts,
    model: str,
    dtype: str,
    max_tokens: int,
    num_logprobs: int,
) -> None:
    with hf_runner(model, dtype=dtype) as hf_model:
        hf_outputs = hf_model.generate_greedy_logprobs_limit(
            example_prompts, max_tokens, num_logprobs
        )

    with vllm_runner(model, dtype=dtype) as vllm_model:
        vllm_outputs = vllm_model.generate_greedy_logprobs(
            example_prompts, max_tokens, num_logprobs
        )
    check_logprobs_close(
        outputs_0_lst=hf_outputs,
        outputs_1_lst=vllm_outputs,
        name_0="hf",
        name_1="vllm",
    )
```
**EN:** This test validates `test_models`. It uses parameterization over `model`. Relevant pytest markers include `cpu_model`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `model`, `dtype`, `max_tokens`.
**CN:** 这个测试验证 `test_models`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `cpu_model`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`model`、`dtype`、`max_tokens`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `pytest`
- **Local relative imports / 本地相对导入**: `...utils.check_logprobs_close`
