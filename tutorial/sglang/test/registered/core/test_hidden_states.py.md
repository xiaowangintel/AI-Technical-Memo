# test_hidden_states.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/core/test_hidden_states.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on core hidden states in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 core hidden states 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Import dependencies
```python
import unittest

import torch
from transformers import AutoModelForCausalLM, AutoTokenizer

import sglang as sgl
from sglang.srt.utils import get_device, is_hip
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import DEFAULT_SMALL_MODEL_NAME_FOR_TEST, CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 11-12: Register CI metadata
```python
register_cuda_ci(est_time=45, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=55, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 14-14: Define module constants
```python
_is_hip = is_hip()
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 15-18: Implement control flow
```python
if _is_hip:
    import os

    os.environ["SGLANG_USE_AITER"] = "0"
```
**EN:** This block adds supporting control flow around the main test or helper logic.
**CN:** 该代码块围绕主要测试或辅助逻辑补充了控制流。

### Lines 21-21: Define class TestHiddenState
```python
class TestHiddenState(CustomTestCase):
```
**EN:** This declaration introduces the `TestHiddenState` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestHiddenState` 测试类，并说明它通过继承承担的职责。

### Lines 22-80: Run test: return hidden states (part 1)
```python
    def test_return_hidden_states(self):
        prompts = ["Today is", "Today is a sunny day and I like"]
        model_path = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        tokenizer = AutoTokenizer.from_pretrained(model_path)
        input_ids = tokenizer(prompts).input_ids

        sampling_params = {
            "temperature": 0,
            "max_new_tokens": 8,
        }

        engine = sgl.Engine(
            model_path=model_path,
            random_seed=42,
            skip_tokenizer_init=True,
            enable_return_hidden_states=True,
        )
        outputs = engine.generate(
            input_ids=input_ids,
            sampling_params=sampling_params,
            return_hidden_states=True,
        )
        engine.shutdown()

        for output in outputs:
            self.assertEqual(len(output["meta_info"]["hidden_states"]), 8)
            for i in range(len(output["meta_info"]["hidden_states"])):
                assert isinstance(output["meta_info"]["hidden_states"][i], list)
                output["meta_info"]["hidden_states"][i] = torch.tensor(
                    output["meta_info"]["hidden_states"][i], dtype=torch.bfloat16
                )
        # Checks that splicing of the batch was done correctly
        self.assertGreater(
            outputs[1]["meta_info"]["hidden_states"][0].shape[0],
            outputs[0]["meta_info"]["hidden_states"][0].shape[0],
        )

        model = AutoModelForCausalLM.from_pretrained(
            model_path, torch_dtype=torch.bfloat16, device_map=get_device()
        )

        for input_id, output in zip(input_ids, outputs):
            with torch.inference_mode():
                hf_out = model(
                    torch.tensor(
                        [input_id + output["output_ids"][:-1]], device=model.device
                    ),
                    output_hidden_states=True,
                )
            print("=== HF Hiddens ===")
            print(hf_out["hidden_states"][-1][0])
            sg_hidden_states = torch.cat(
                [
                    i.unsqueeze(0) if len(i.shape) == 1 else i
                    for i in output["meta_info"]["hidden_states"]
                ]
            ).to(get_device())
            print("=== SRT Hiddens ===")
            print(sg_hidden_states)
```
**EN:** This test method exercises return hidden states and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 return hidden states 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 82-84: Run test: return hidden states (part 2)
```python
            print(
                f"Max diff: {torch.max(torch.abs(hf_out['hidden_states'][-1][0] - sg_hidden_states))}"
            )
```
**EN:** This test method exercises return hidden states and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 return hidden states 场景，并验证观测到的行为是否符合预期契约。

### Lines 86-94: Run test: return hidden states (part 3)
```python
            atol = 0.8
            self.assertTrue(
                torch.allclose(
                    hf_out["hidden_states"][-1][0],
                    sg_hidden_states,
                    atol=atol,
                    rtol=0,
                )
            )
```
**EN:** This test method exercises return hidden states and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 return hidden states 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 96-146: Run test: repeatedly changes hidden states
```python
    def test_repeatedly_changes_hidden_states(self):
        prompts = ["Today is", "Today is a sunny day and I like"]
        model_path = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        tokenizer = AutoTokenizer.from_pretrained(model_path)
        input_ids = tokenizer(prompts).input_ids

        sampling_params = {
            "temperature": 0,
            "max_new_tokens": 8,
        }

        engine = sgl.Engine(
            model_path=model_path,
            random_seed=42,
            skip_tokenizer_init=True,
            enable_return_hidden_states=True,
        )
        outputs_completion_first_round = engine.generate(
            input_ids=input_ids,
            sampling_params=sampling_params,
            return_hidden_states=True,
        )
        outputs_hidden_state = engine.generate(
            input_ids=input_ids,
            sampling_params=sampling_params,
            return_hidden_states=False,
        )

        outputs_completion_last_round = engine.generate(
            input_ids=input_ids,
            sampling_params=sampling_params,
            return_hidden_states=True,
        )
        engine.shutdown()

        for (
            output_completion_first_round,
            output_hidden_state,
            output_completion_last_round,
        ) in zip(
            outputs_completion_first_round,
            outputs_hidden_state,
            outputs_completion_last_round,
        ):
            self.assertEqual(
                len(output_completion_first_round["meta_info"]["hidden_states"]), 8
            )
            self.assertNotIn("hidden_states", output_hidden_state["meta_info"])
            self.assertEqual(
                len(output_completion_last_round["meta_info"]["hidden_states"]), 8
            )
```
**EN:** This test method exercises repeatedly changes hidden states and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 repeatedly changes hidden states 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 149-150: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `os`, `torch`, `transformers`, `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `register_amd_ci`, `CustomTestCase`, `unittest.main`
