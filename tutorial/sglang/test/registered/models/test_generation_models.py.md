# test_generation_models.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/models/test_generation_models.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates generation models behavior in SGLang's models area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 models 领域中与 generation models 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 2-3: supporting source context / 辅助源码上下文
```python

# Generation model tests (CUDA only)
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 4-26: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=150, stage="extra-a", runner_config="1-gpu-large")
register_amd_ci(est_time=106, suite="stage-b-test-1-gpu-small-amd")

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
"""
Usage:

To test a specific model locally:
1. Add it to ALL_MODELS, for example, `ModelCase("Qwen/Qwen2-1.5B")`
2. Run `ONLY_RUN=Qwen/Qwen2-1.5B python3 -m unittest test_generation_models.TestGenerationModels`
"""
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 28-45: module imports and dependencies / 模块导入与依赖
```python
import dataclasses
import multiprocessing as mp
import os
import unittest
from contextlib import nullcontext
from typing import List, Optional

import torch

from sglang.srt.environ import envs
from sglang.srt.utils import is_hip
from sglang.test.runners import (
    DEFAULT_PROMPTS,
    HFRunner,
    SRTRunner,
    check_close_model_outputs,
)
from sglang.test.test_utils import CustomTestCase, is_in_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `dataclasses`, `multiprocessing`, `os`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `dataclasses`, `multiprocessing`, `os`, `unittest`。

### Lines 48-49: class ModelCase declaration / 类 ModelCase 声明
```python
@dataclasses.dataclass
class ModelCase:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 50-57: class-level constants and configuration for `ModelCase` / 类级常量与配置
```python
    model_path: str
    tp_size: int = 1
    prefill_tolerance: float = 5e-2
    decode_tolerance: float = 6e-2  # Increased to fix numerical error in issue #8614.
    rouge_l_tolerance: float = 1
    skip_long_prompt: bool = False
    trust_remote_code: bool = False
    attention_backend: Optional[str] = None
```
**EN:** This block defines shared names such as `model_path`, `tp_size`, `prefill_tolerance`, `decode_tolerance`, `rouge_l_tolerance`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model_path`, `tp_size`, `prefill_tolerance`, `decode_tolerance`, `rouge_l_tolerance` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 58-60: supporting source context / 辅助源码上下文
```python


# Popular models that run on the CI
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 61-128: module-level constants and configuration / 模块级常量与配置
```python
CI_MODELS = [
    ModelCase("meta-llama/Llama-3.1-8B-Instruct"),
    ModelCase("google/gemma-2-2b", attention_backend="triton" if is_hip() else None),
]

# the complete set of models to test sglang's generation model
ALL_MODELS = [
    *CI_MODELS,
    ModelCase("Qwen/Qwen2-1.5B", decode_tolerance=7e-2),
    ModelCase("Qwen/Qwen2.5-14B-Instruct"),
    ModelCase("HuggingFaceTB/SmolLM-135M-Instruct", skip_long_prompt=True),
    ModelCase("allenai/OLMo-1B-0724-hf", decode_tolerance=8e-2, skip_long_prompt=True),
    ModelCase("shanearora/2025-sep-a-base-model"),
    ModelCase(
        "THUDM/glm-4-9b-chat", tp_size=2, trust_remote_code=True, skip_long_prompt=True
    ),
    ModelCase("openai-community/gpt2"),
    ModelCase("microsoft/phi-1_5", trust_remote_code=True),
    ModelCase("adept/persimmon-8b-chat"),
    ModelCase("upstage/SOLAR-10.7B-Instruct-v1.0"),
    ModelCase("inclusionAI/Ling-lite", trust_remote_code=True),
    ModelCase("microsoft/Phi-3-small-8k-instruct", trust_remote_code=True),
    ModelCase("allenai/OLMo-2-1124-7B-Instruct", skip_long_prompt=True),
    ModelCase("ibm-granite/granite-3.0-2b-instruct", skip_long_prompt=True),
    ModelCase(
        "microsoft/Phi-3.5-MoE-instruct",
        tp_size=2,
        trust_remote_code=True,
        skip_long_prompt=True,
    ),
    ModelCase("facebook/opt-125m", skip_long_prompt=True),
    ModelCase(
        "nvidia/Llama-3_3-Nemotron-Super-49B-v1_5",
        tp_size=2,
        trust_remote_code=True,
        skip_long_prompt=True,
    ),
    ModelCase(
        "nvidia/Llama-3_1-Nemotron-Ultra-253B-v1",
        tp_size=8,
        trust_remote_code=True,
        skip_long_prompt=True,
    ),
    ModelCase(
        "nvidia/NVIDIA-Nemotron-Nano-9B-v2",
        trust_remote_code=True,
        skip_long_prompt=True,
    ),
    ModelCase(
        "swiss-ai/Apertus-8B",
        trust_remote_code=True,
        skip_long_prompt=True,
    ),
    ModelCase(
        "LiquidAI/LFM2.5-1.2B-Instruct",
        trust_remote_code=True,
    ),
    ModelCase(
        "ibm-granite/granite-4.0-h-micro",
        trust_remote_code=True,
    ),
]

MAMBA_MODEL_PATHS = [
    "LiquidAI/LFM2.5-1.2B-Instruct",
]

TORCH_DTYPES = [torch.float16]
```
**EN:** This block defines shared names such as `CI_MODELS`, `ALL_MODELS`, `MAMBA_MODEL_PATHS`, `TORCH_DTYPES`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `CI_MODELS`, `ALL_MODELS`, `MAMBA_MODEL_PATHS`, `TORCH_DTYPES` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 131-132: class TestGenerationModels declaration / 类 TestGenerationModels 声明
```python
class TestGenerationModels(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 133-135: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        mp.set_start_method("spawn", force=True)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 137-184: helper routine assert close logits and output strs / 辅助流程 assert close logits and output strs
```python
    def assert_close_logits_and_output_strs(
        self,
        prompts: List[str],
        model_case: ModelCase,
        torch_dtype: torch.dtype,
    ) -> None:
        model_path = model_case.model_path
        max_new_tokens = 32

        # Set conv dtype for hybrid models to match inference dtype
        dtype_str = {torch.float16: "float16", torch.bfloat16: "bfloat16"}.get(
            torch_dtype, "bfloat16"
        )

        if model_case.model_path in MAMBA_MODEL_PATHS:
            env_ctx = envs.SGLANG_MAMBA_CONV_DTYPE.override(dtype_str)
        else:
            env_ctx = nullcontext()

        with HFRunner(
            model_path,
            torch_dtype=torch_dtype,
            model_type="generation",
            trust_remote_code=model_case.trust_remote_code,
        ) as hf_runner:
            hf_outputs = hf_runner.forward(prompts, max_new_tokens=max_new_tokens)

        with (
            env_ctx,
            SRTRunner(
                model_path,
                tp_size=model_case.tp_size,
                torch_dtype=torch_dtype,
                model_type="generation",
                trust_remote_code=model_case.trust_remote_code,
                attention_backend=model_case.attention_backend,
            ) as srt_runner,
        ):
            srt_outputs = srt_runner.forward(prompts, max_new_tokens=max_new_tokens)

        check_close_model_outputs(
            hf_outputs=hf_outputs,
            srt_outputs=srt_outputs,
            prefill_tolerance=model_case.prefill_tolerance,
            decode_tolerance=model_case.decode_tolerance,
            rouge_l_tolerance=model_case.rouge_l_tolerance,
            debug_text=f"model_path={model_path} prompts={prompts}",
        )
```
**EN:** This helper encapsulates `assert_close_logits_and_output_strs` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `assert_close_logits_and_output_strs`，以便周围测试复用准备、执行或校验逻辑。

### Lines 186-199: test case ci models / 测试用例 ci models
```python
    @unittest.skipIf(not is_in_ci(), "Local test should run all models")
    def test_ci_models(self):
        for model_case in CI_MODELS:
            for torch_dtype in TORCH_DTYPES:
                prompts = DEFAULT_PROMPTS

                # Skip long prompts for models that do not have a long context
                if model_case.skip_long_prompt:
                    prompts = [p for p in DEFAULT_PROMPTS if len(p) < 1000]

                # Assert the logits and output strs are close
                self.assert_close_logits_and_output_strs(
                    prompts, model_case, torch_dtype
                )
```
**EN:** This test exercises `test_ci_models` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ci_models`。

### Lines 201-219: test case all models / 测试用例 all models
```python
    @unittest.skipIf(is_in_ci(), "CI only runs selected models for simplicity")
    def test_all_models(self):
        for model_case in ALL_MODELS:
            for torch_dtype in TORCH_DTYPES:
                if (
                    "ONLY_RUN" in os.environ
                    and os.environ["ONLY_RUN"] != model_case.model_path
                ):
                    continue

                # Skip long prompts for models that do not have a long context
                prompts = DEFAULT_PROMPTS
                if model_case.skip_long_prompt:
                    prompts = [p for p in DEFAULT_PROMPTS if len(p) < 1000]

                # Assert the logits and output strs are close
                self.assert_close_logits_and_output_strs(
                    prompts, model_case, torch_dtype
                )
```
**EN:** This test exercises `test_all_models` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_models`。

### Lines 222-223: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `ModelCase`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestGenerationModels`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestGenerationModels.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestGenerationModels.assert_close_logits_and_output_strs`: This helper encapsulates `assert_close_logits_and_output_strs` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `assert_close_logits_and_output_strs`，以便周围测试复用准备、执行或校验逻辑。
- `TestGenerationModels.test_ci_models`: This test exercises `test_ci_models` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ci_models`。
- `TestGenerationModels.test_all_models`: This test exercises `test_all_models` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_models`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `multiprocessing`, `os`, `unittest`, `contextlib`, `typing`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.runners`, `sglang.test.test_utils`

- **Total lines / 总行数**: 223
