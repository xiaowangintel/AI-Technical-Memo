# test_phimoe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/generation/test_phimoe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers language-model behavior and model-facing behavior. The file defines 2 test(s), 0 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖语言模型行为与面向模型的行为。它定义了 2 个测试、0 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L9)
```python
import pytest
import torch

from vllm.platforms import current_platform

from ....utils import large_gpu_test
from ...utils import check_logprobs_close
```
**EN:** Imports third-party packages like `pytest`, `torch`, project helpers such as `vllm.model_executor.models.phimoe.phimoe_routing_function`, `vllm.platforms.current_platform`, `....utils.large_gpu_test`.
**CN:** 导入第三方包（如 `pytest`、`torch`）、项目内辅助模块（如 `vllm.model_executor.models.phimoe.phimoe_routing_function`、`vllm.platforms.current_platform`、`....utils.large_gpu_test`）。

### Module setup / 模块级配置: MODELS (L11-L13)
```python
MODELS = [
    "microsoft/Phi-3.5-MoE-instruct",
]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODELS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODELS`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_phimoe_routing_function (L16-L60)
```python
def test_phimoe_routing_function():
    from vllm.model_executor.models.phimoe import phimoe_routing_function

    test_case = {
        0: {
            "hidden_states": torch.tensor(
                [1, 2, 3, 4, 5, 6, 7, 8], dtype=torch.float32, requires_grad=False
            ).view(4, 2),
            "gating_output": torch.tensor(
                [0.1, 0.2, 0.3, 0.4], dtype=torch.float32, requires_grad=False
            ),
            "topk": 2,
            "renormalize": False,
        },
        1: {
            "hidden_states": torch.tensor(
                [1, 2, 3, 4, 5, 6, 7, 8], dtype=torch.float32, requires_grad=False
            ).view(4, 2),
# ... 19 lines omitted for brevity ...
            "topk_ids": torch.tensor([0, 3], dtype=torch.long, requires_grad=False),
        },
    }

    for test_id in test_case:
        topk_weights, topk_ids = phimoe_routing_function(**test_case[test_id])
        assert torch.allclose(topk_weights, ground_truth[test_id]["topk_weights"])
        assert torch.equal(topk_ids, ground_truth[test_id]["topk_ids"])
```
**EN:** This test validates `test_phimoe_routing_function`. The main assertion is `torch.allclose(topk_weights, ground_truth[test_id]['topk_weights'])` and `torch.equal(topk_ids, ground_truth[test_id]['topk_ids'])`.
**CN:** 这个测试验证 `test_phimoe_routing_function`。 核心断言是 `torch.allclose(topk_weights, ground_truth[test_id]['topk_weights'])` and `torch.equal(topk_ids, ground_truth[test_id]['topk_ids'])`。

### Test / 测试: test_models (L70-L109)
```python
@pytest.mark.skip(
    reason="Skipping due to known issue: "
    "'DynamicCache' object has no attribute 'seen_tokens'. See: "
    "https://huggingface.co/microsoft/Phi-3.5-MoE-instruct/discussions/58 "
    "for details.",
)
@pytest.mark.skipif(
    condition=current_platform.is_cpu(),
    reason="This test takes a lot time to run on CPU, "
    "and vllm CI's disk space is not enough for this model.",
)
@large_gpu_test(min_gb=80)
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["bfloat16"])
@pytest.mark.parametrize("max_tokens", [64])
@pytest.mark.parametrize("num_logprobs", [5])
def test_models(
    hf_runner,
# ... 14 lines omitted for brevity ...
            example_prompts, max_tokens, num_logprobs
        )
    check_logprobs_close(
        outputs_0_lst=hf_outputs,
        outputs_1_lst=vllm_outputs,
        name_0="hf",
        name_1="vllm",
    )
```
**EN:** This test validates `test_models`. It uses parameterization over `model`. Relevant pytest markers include `skip`, `skipif`. Key inputs are `hf_runner`, `vllm_runner`, `example_prompts`, `model`, `dtype`, `max_tokens`.
**CN:** 这个测试验证 `test_models`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `skip`、`skipif`。 关键输入包括 `hf_runner`、`vllm_runner`、`example_prompts`、`model`、`dtype`、`max_tokens`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `pytest`, `torch`
- **Project / 项目内**: `vllm.model_executor.models.phimoe.phimoe_routing_function`, `vllm.platforms.current_platform`
- **Local relative imports / 本地相对导入**: `....utils.large_gpu_test`, `...utils.check_logprobs_close`
