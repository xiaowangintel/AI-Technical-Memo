# test_reward.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/language/pooling/test_reward.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and language-model behavior. The file defines 2 test(s), 1 fixture(s), and 5 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与语言模型行为。它定义了 2 个测试、1 个 fixture，以及 5 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L15)
```python
import json
from typing import TYPE_CHECKING

import pytest
import torch
import torch.nn.functional as F
from transformers import AutoModel

from vllm.platforms import current_platform

from ....conftest import HfRunner
from ....utils import VLLM_PATH
from ...registry import HF_EXAMPLE_MODELS
```
**EN:** Imports standard-library modules such as `json`, `typing.TYPE_CHECKING`, third-party packages like `_typeshed.StrPath`, `pytest`, `torch`, project helpers such as `vllm.platforms.current_platform`, `....conftest.HfRunner`, `....utils.VLLM_PATH`.
**CN:** 导入标准库模块（如 `json`、`typing.TYPE_CHECKING`）、第三方包（如 `_typeshed.StrPath`、`pytest`、`torch`）、项目内辅助模块（如 `vllm.platforms.current_platform`、`....conftest.HfRunner`、`....utils.VLLM_PATH`）。

### Module setup / 模块级配置: FIXTURES_PATH, FIXTURE_REWARD_RESULT (L17-L25)
```python
if TYPE_CHECKING:
    from _typeshed import StrPath


FIXTURES_PATH = VLLM_PATH / "tests/models/fixtures"
assert FIXTURES_PATH.exists()
FIXTURE_REWARD_RESULT = {
    "Qwen/Qwen2.5-Math-PRM-7B": FIXTURES_PATH / "qwen2_5_math_prm_reward_step.json",
}
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `FIXTURES_PATH`, `FIXTURE_REWARD_RESULT`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `FIXTURES_PATH`、`FIXTURE_REWARD_RESULT`。后续辅助函数和测试会复用这些值，以减少重复。

### Fixture / 夹具: math_step_prompts (L28-L43)
```python
@pytest.fixture
def math_step_prompts():
    # ruff: noqa: E501
    data = {
        "system": "Please reason step by step, and put your final answer within \\boxed{}. ",
        "query": "Sue lives in a fun neighborhood.  One weekend, the neighbors decided to play a prank on Sue.  On Friday morning, the neighbors placed 18 pink plastic flamingos out on Sue's front yard.  On Saturday morning, the neighbors took back one third of the flamingos, painted them white, and put these newly painted white flamingos back out on Sue's front yard.  Then, on Sunday morning, they added another 18 pink plastic flamingos to the collection. At noon on Sunday, how many more pink plastic flamingos were out than white plastic flamingos?",
        "response": [
            "To find out how many more pink plastic flamingos were out than white plastic flamingos at noon on Sunday, we can break down the problem into steps. First, on Friday, the neighbors start with 18 pink plastic flamingos.",
            "On Saturday, they take back one third of the flamingos. Since there were 18 flamingos, (1/3 \\times 18 = 6) flamingos are taken back. So, they have (18 - 6 = 12) flamingos left in their possession. Then, they paint these 6 flamingos white and put them back out on Sue's front yard. Now, Sue has the original 12 pink flamingos plus the 6 new white ones. Thus, by the end of Saturday, Sue has (12 + 6 = 18) pink flamingos and 6 white flamingos.",
            "On Sunday, the neighbors add another 18 pink plastic flamingos to Sue's front yard. By the end of Sunday morning, Sue has (18 + 18 = 36) pink flamingos and still 6 white flamingos.",
            "To find the difference, subtract the number of white flamingos from the number of pink flamingos: (36 - 6 = 30). Therefore, at noon on Sunday, there were 30 more pink plastic flamingos out than white plastic flamingos. The answer is (\\boxed{30}).",
        ],
    }
    answer = "<extra_0>".join(data["response"]) + "<extra_0>"
    prompt = f"<im_start>system\n{data['system']}<im_end>\n<im_start>user\n{data['query']}<im_end>\n<im_start>assistant\n{answer}<im_end><|endoftext|>"
    return [prompt]
```
**EN:** This fixture prepares `math_step_prompts` for dependent tests.
**CN:** 这个 fixture 为依赖它的测试准备 `math_step_prompts`。

### Helper / 辅助函数: step_reward_patch_hf_model (L46-L73)
```python
def step_reward_patch_hf_model(hf_model: HfRunner):
    # Patch the hf_runner to use the step reward function
    def make_step_rewards(
        logits: torch.Tensor, token_masks: torch.Tensor
    ) -> list[list[float]]:
        probabilities = F.softmax(logits, dim=-1)
        probabilities = probabilities * token_masks.unsqueeze(-1)

        all_scores_res: list[list[float]] = []
        for i in range(probabilities.size(0)):
            sample = probabilities[i]  # seq_len, num_labels
            positive_probs = sample[sample != 0].view(-1, 2)
            non_zero_elements_list = positive_probs.cpu().tolist()
            all_scores_res.append(non_zero_elements_list)
        return all_scores_res

    def reward(prompts: list[str]) -> list[list[float]]:
        input_ids = hf_model.tokenizer(prompts, return_tensors="pt").input_ids
        input_ids = hf_model.wrap_device(input_ids)
        outputs = hf_model.model(input_ids=input_ids)

        step_sep_id = hf_model.tokenizer.encode("<extra_0>")[0]
        token_masks = input_ids == step_sep_id
        return make_step_rewards(outputs[0], token_masks)

    hf_model.reward = reward  # type: ignore[attr-defined]

    return hf_model
```
**EN:** This helper encapsulates reusable logic in `step_reward_patch_hf_model`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `step_reward_patch_hf_model` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: dump_reward_outputs (L76-L78)
```python
def dump_reward_outputs(outputs: list[list[float]], filename: "StrPath"):
    with open(filename, "w", encoding="utf-8") as f:
        json.dump(outputs, f)
```
**EN:** This helper encapsulates reusable logic in `dump_reward_outputs`. Key inputs are `outputs`, `filename`.
**CN:** 这个辅助函数将可复用逻辑封装在 `dump_reward_outputs` 中。 关键输入包括 `outputs`、`filename`。

### Helper / 辅助函数: load_reward_outputs (L81-L83)
```python
def load_reward_outputs(filename: "StrPath") -> list[list[float]]:
    with open(filename, encoding="utf-8") as f:
        return json.load(f)
```
**EN:** This helper encapsulates reusable logic in `load_reward_outputs`. Key inputs are `filename`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `load_reward_outputs` 中。 关键输入包括 `filename`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_prm_models (L86-L126)
```python
@pytest.mark.parametrize(
    "model",
    [
        pytest.param(
            "Qwen/Qwen2.5-Math-PRM-7B",
            marks=[pytest.mark.core_model, pytest.mark.cpu_model],
        ),
    ],
)
@pytest.mark.parametrize("dtype", ["half"])
def test_prm_models(
    hf_runner,
    vllm_runner,
    math_step_prompts,
    model: str,
    dtype: str,
) -> None:
    model_info = HF_EXAMPLE_MODELS.find_hf_info(model)
# ... 15 lines omitted for brevity ...
    )

    # check logits difference
    for hf_output, vllm_output in zip(hf_outputs, vllm_outputs):
        hf_output = torch.tensor(hf_output).float()
        vllm_output = torch.tensor(vllm_output).float()

        assert torch.allclose(hf_output, vllm_output, 1.5e-2)
```
**EN:** This test validates `test_prm_models`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `math_step_prompts`, `model`, `dtype`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `torch.allclose(hf_output, vllm_output, 0.015)`.
**CN:** 这个测试验证 `test_prm_models`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`math_step_prompts`、`model`、`dtype`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `torch.allclose(hf_output, vllm_output, 0.015)`。

### Test / 测试: test_prm_models_with_golden_outputs (L129-L158)
```python
@pytest.mark.parametrize(
    "model",
    [
        pytest.param(
            "Qwen/Qwen2.5-Math-PRM-7B",
            marks=[pytest.mark.core_model, pytest.mark.cpu_model],
        ),
    ],
)
@pytest.mark.parametrize("dtype", ["half"])
def test_prm_models_with_golden_outputs(
    vllm_runner,
    math_step_prompts,
    model: str,
    dtype: str,
) -> None:
    if not FIXTURE_REWARD_RESULT.get(model):
        pytest.skip(f"No available golden outputs for {model}.")

    with vllm_runner(model, max_model_len=1024, dtype=dtype) as vllm_model:
        vllm_outputs = vllm_model.reward(math_step_prompts)

    golden_outputs = load_reward_outputs(FIXTURE_REWARD_RESULT[model])

    # check logits difference
    for golden_output, vllm_output in zip(golden_outputs, vllm_outputs):
        golden_output = torch.tensor(golden_output).float()
        vllm_output = torch.tensor(vllm_output).float()

        assert torch.allclose(golden_output, vllm_output, 1.5e-2)
```
**EN:** This test validates `test_prm_models_with_golden_outputs`. It uses parameterization over `model`. Key inputs are `vllm_runner`, `math_step_prompts`, `model`, `dtype`. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `torch.allclose(golden_output, vllm_output, 0.015)`.
**CN:** 这个测试验证 `test_prm_models_with_golden_outputs`。 它通过参数化组合 `model`。 关键输入包括 `vllm_runner`、`math_step_prompts`、`model`、`dtype`。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `torch.allclose(golden_output, vllm_output, 0.015)`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `json`, `typing.TYPE_CHECKING`
- **Third-party / 第三方**: `_typeshed.StrPath`, `pytest`, `torch`, `torch.nn.functional`, `transformers.AutoModel`
- **Project / 项目内**: `vllm.platforms.current_platform`
- **Local relative imports / 本地相对导入**: `....conftest.HfRunner`, `....utils.VLLM_PATH`, `...registry.HF_EXAMPLE_MODELS`
