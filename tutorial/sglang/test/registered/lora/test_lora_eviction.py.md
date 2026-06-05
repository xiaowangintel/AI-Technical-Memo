# test_lora_eviction.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/lora/test_lora_eviction.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates lora eviction behavior in SGLang's lora area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 lora 领域中与 lora eviction 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: supporting source context / 辅助源码上下文
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 15-24: module imports and dependencies / 模块导入与依赖
```python
import contextlib
import multiprocessing as mp
import unittest
from typing import Dict, List, Tuple

import torch

from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.runners import SRTRunner
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `contextlib`, `multiprocessing`, `unittest`, `typing`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `contextlib`, `multiprocessing`, `unittest`, `typing`。

### Lines 26-44: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=263, stage="extra-a", runner_config="1-gpu-small")
register_amd_ci(est_time=224, suite="stage-b-test-1-gpu-small-amd")

PROMPTS = [
    "AI is a field of computer science focused on",
    """
    ### Instruction:
    Compose a SQL query that uses the following table: users, and returns the user_id and name of all users whose name that does not have a duplicate in the table.
    ### Response:
    SELECT user_id, name FROM users WHERE name LIKE 'A%';
    """,
]

ADAPTERS = [
    "faridlazuarda/valadapt-llama-3.1-8B-it-chinese",  # target_modules = q, v
    "philschmid/code-llama-3-1-8b-text-to-sql-lora",  # target_modules = q, k, v, o, gate, up, down
]

BASE_MODEL = "meta-llama/Meta-Llama-3.1-8B-Instruct"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 47-54: function dynamically loaded adapter / 函数 dynamically loaded adapter
```python
@contextlib.contextmanager
def dynamically_loaded_adapter(runner, lora_path: str, lora_name: str):
    """A context manager to load and automatically unload a LoRA adapter."""
    try:
        runner.load_lora_adapter(lora_name=lora_name, lora_path=lora_path)
        yield
    finally:
        runner.unload_lora_adapter(lora_name=lora_name)
```
**EN:** A context manager to load and automatically unload a LoRA adapter. This block implements `dynamically_loaded_adapter` and captures one focused piece of the module's behavior.
**CN:** A context manager to load and automatically unload a LoRA adapter. 该代码块实现 `dynamically_loaded_adapter`，承担模块行为中的一个聚焦逻辑片段。

### Lines 57-57: class TestLoRAEviction declaration / 类 TestLoRAEviction 声明
```python
class TestLoRAEviction(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 58-67: test case lora eviction with different target modules / 测试用例 lora eviction with different target modules
```python
    def test_lora_eviction_with_different_target_modules(self):
        """
        Test LoRA eviction with different target modules.

        This test runs inference against two LoRA adapters in different orders to force eviction behavior, and ensures
        that the outputs of the same (adapter, prompt) pair are consistent across runs.
        """
        output_history = {}
        self._run_test(ADAPTERS, output_history, reverse=False)
        self._run_test(ADAPTERS, output_history, reverse=True)
```
**EN:** Test LoRA eviction with different target modules. This test exercises `test_lora_eviction_with_different_target_modules` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test LoRA eviction with different target modules. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_eviction_with_different_target_modules`。

### Lines 69-78: test case lora eviction with reused lora name / 测试用例 lora eviction with reused lora name
```python
    def test_lora_eviction_with_reused_lora_name(self):
        """
        Test LoRA eviction with reused LoRA names.

        This test runs inference against two LoRA adapters with the same name to ensure that the eviction behavior
        works correctly when reusing LoRA names.
        """
        output_history = {}
        self._run_test(ADAPTERS, output_history, reuse_lora_name=True, repeat=1)
        self._run_test(ADAPTERS, output_history, reuse_lora_name=False, repeat=1)
```
**EN:** Test LoRA eviction with reused LoRA names. This test exercises `test_lora_eviction_with_reused_lora_name` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test LoRA eviction with reused LoRA names. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_eviction_with_reused_lora_name`。

### Lines 80-139: method run test / 方法 run test
```python
    def _run_test(
        self,
        lora_paths: List[str],
        output_history: Dict[Tuple[str, str], str],
        reverse: bool = False,
        repeat: int = 2,
        reuse_lora_name: bool = False,
    ):
        REUSED_LORA_NAME = "lora"
        max_new_tokens = 256
        torch_dtype = torch.float16
        base_path = BASE_MODEL
        assert len(lora_paths) >= 2

        initial_lora_paths = lora_paths if not reuse_lora_name else None
        # Initialize runners
        with SRTRunner(
            base_path,
            torch_dtype=torch_dtype,
            model_type="generation",
            lora_paths=initial_lora_paths,
            max_loras_per_batch=1,
            enable_lora=True,
            max_lora_rank=256,
            lora_target_modules=["all"],
        ) as srt_runner:
            adapter_sequence = lora_paths if not reverse else lora_paths[::-1]

            for i in range(repeat):
                for j, lora_path in enumerate(adapter_sequence):
                    print(
                        f"\n========== Testing LoRA eviction with adapter '{lora_path}' (#{j + 1}/{len(adapter_sequence)}), reuse_lora_name: {reuse_lora_name}, reversed: {reverse}, repeat: {i + 1}/{repeat} ---"
                    )

                    lora_name = REUSED_LORA_NAME if reuse_lora_name else lora_path
                    context = (
                        dynamically_loaded_adapter(srt_runner, lora_path, lora_name)
                        if reuse_lora_name
                        else contextlib.nullcontext()
                    )
                    with context:
                        for prompt in PROMPTS:
                            print("\nprompt:\n", prompt)
                            srt_outputs = srt_runner.forward(
                                [prompt],
                                max_new_tokens=max_new_tokens,
                                lora_paths=[lora_name],
                            )
                            output = srt_outputs.output_strs[0].strip()
                            print("\noutput:\n", output)

                            prev_output = output_history.get((lora_path, prompt))
                            if prev_output is not None:
                                self.assertEqual(
                                    prev_output,
                                    output,
                                    f"Output mismatch for adapter {lora_path} and prompt '{prompt}' on repeat {j + 1}, previous: '{prev_output}', current: '{output}'.",
                                )
                            else:
                                output_history[(lora_path, prompt)] = output
```
**EN:** This block implements `_run_test` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_test`，承担模块行为中的一个聚焦逻辑片段。

### Lines 142-148: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    try:
        mp.set_start_method("spawn")
    except RuntimeError:
        pass

    unittest.main(warnings="ignore")
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `dynamically_loaded_adapter`: A context manager to load and automatically unload a LoRA adapter. / 该代码块实现 `dynamically_loaded_adapter`，承担模块行为中的一个聚焦逻辑片段。
- `TestLoRAEviction`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLoRAEviction.test_lora_eviction_with_different_target_modules`: Test LoRA eviction with different target modules. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_eviction_with_different_target_modules`。
- `TestLoRAEviction.test_lora_eviction_with_reused_lora_name`: Test LoRA eviction with reused LoRA names. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_eviction_with_reused_lora_name`。
- `TestLoRAEviction._run_test`: This block implements `_run_test` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_test`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `contextlib`, `multiprocessing`, `unittest`, `typing`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.runners`, `sglang.test.test_utils`

- **Total lines / 总行数**: 148
