# test_lora_tp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/lora/test_lora_tp.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates lora tp behavior in SGLang's lora area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 lora 领域中与 lora tp 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

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

### Lines 15-30: module imports and dependencies / 模块导入与依赖
```python
import multiprocessing as mp
import os
import unittest
from typing import List, Optional

from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.lora_utils import (
    ALL_OTHER_LORA_MODELS,
    CI_LORA_MODELS,
    CI_MULTI_LORA_MODELS,
    DEFAULT_PROMPTS,
    TORCH_DTYPES,
    LoRAModelCase,
    run_lora_test_one_by_one,
)
from sglang.test.test_utils import CustomTestCase, is_in_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `multiprocessing`, `os`, `unittest`, `typing`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `multiprocessing`, `os`, `unittest`, `typing`。

### Lines 32-41: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(
    est_time=230,
    stage="base-c",
    runner_config="8-gpu-h200",
)
register_amd_ci(
    est_time=116,
    suite="stage-b-test-2-gpu-large-amd",
    disabled="see https://github.com/sgl-project/sglang/issues/13107",
)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 44-45: class TestLoRATP declaration / 类 TestLoRATP 声明
```python
class TestLoRATP(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 46-70: method run tp on model cases / 方法 run tp on model cases
```python
    def _run_tp_on_model_cases(
        self,
        model_cases: List[LoRAModelCase],
        enable_lora_overlap_loading: Optional[bool] = None,
    ):
        tp_list = [2]  # Define TP sizes to iterate over
        for model_case in model_cases:
            # If skip_long_prompt is True, filter out prompts longer than 1000 characters
            prompts = (
                DEFAULT_PROMPTS
                if not model_case.skip_long_prompt
                else [p for p in DEFAULT_PROMPTS if len(p) < 1000]
            )
            for tp_size in tp_list:
                model_case.tp_size = tp_size
                for torch_dtype in TORCH_DTYPES:
                    run_lora_test_one_by_one(
                        prompts,
                        model_case,
                        torch_dtype,
                        max_new_tokens=32,
                        enable_lora_overlap_loading=enable_lora_overlap_loading,
                        test_tag=f"tp={tp_size}, enable_lora_overlap_loading={enable_lora_overlap_loading}",
                        attention_backend="fa3",
                    )
```
**EN:** This block implements `_run_tp_on_model_cases` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_tp_on_model_cases`，承担模块行为中的一个聚焦逻辑片段。

### Lines 72-73: test case ci lora models / 测试用例 ci lora models
```python
    def test_ci_lora_models(self):
        self._run_tp_on_model_cases(CI_LORA_MODELS)
```
**EN:** This test exercises `test_ci_lora_models` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ci_lora_models`。

### Lines 75-78: test case lora overlap loading ci lora models / 测试用例 lora overlap loading ci lora models
```python
    def test_lora_overlap_loading_ci_lora_models(self):
        self._run_tp_on_model_cases(
            CI_MULTI_LORA_MODELS, enable_lora_overlap_loading=True
        )
```
**EN:** This test exercises `test_lora_overlap_loading_ci_lora_models` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_overlap_loading_ci_lora_models`。

### Lines 80-91: test case all lora models / 测试用例 all lora models
```python
    def test_all_lora_models(self):
        if is_in_ci():
            return

        # Retain ONLY_RUN check here
        filtered_models = []
        for model_case in ALL_OTHER_LORA_MODELS:
            if "ONLY_RUN" in os.environ and os.environ["ONLY_RUN"] != model_case.base:
                continue
            filtered_models.append(model_case)

        self._run_tp_on_model_cases(filtered_models)
```
**EN:** This test exercises `test_all_lora_models` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_lora_models`。

### Lines 94-100: direct execution entry point / 直接执行入口
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
- `TestLoRATP`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLoRATP._run_tp_on_model_cases`: This block implements `_run_tp_on_model_cases` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_tp_on_model_cases`，承担模块行为中的一个聚焦逻辑片段。
- `TestLoRATP.test_ci_lora_models`: This test exercises `test_ci_lora_models` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ci_lora_models`。
- `TestLoRATP.test_lora_overlap_loading_ci_lora_models`: This test exercises `test_lora_overlap_loading_ci_lora_models` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_overlap_loading_ci_lora_models`。
- `TestLoRATP.test_all_lora_models`: This test exercises `test_all_lora_models` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_lora_models`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `multiprocessing`, `os`, `unittest`, `typing`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.lora_utils`, `sglang.test.test_utils`

- **Total lines / 总行数**: 100
