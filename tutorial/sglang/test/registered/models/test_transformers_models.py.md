# test_transformers_models.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/models/test_transformers_models.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates transformers models behavior in SGLang's models area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 models 领域中与 transformers models 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: supporting source context / 辅助源码上下文
```python
# Transformers fallback model tests

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 3-22: module imports and dependencies / 模块导入与依赖
```python
import dataclasses
import multiprocessing as mp
import unittest
from types import SimpleNamespace
from typing import List

import torch

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.runners import DEFAULT_PROMPTS, SRTRunner, check_close_model_outputs
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `dataclasses`, `multiprocessing`, `unittest`, `types`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `dataclasses`, `multiprocessing`, `unittest`, `types`。

### Lines 24-25: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=177, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=320, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 28-28: class TestTransformersFallbackEndpoint declaration / 类 TestTransformersFallbackEndpoint 声明
```python
class TestTransformersFallbackEndpoint(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 29-40: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=["--model-impl", "transformers"],
        )
        cls.mmlu_lower_bound = 0.63
        cls.gsm8k_lower_bound = 0.65
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 42-44: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 46-55: test case mmlu / 测试用例 mmlu
```python
    def test_mmlu(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="mmlu",
            num_examples=64,
            num_threads=32,
        )
        metrics = run_eval(args)
        self.assertGreaterEqual(metrics["score"], self.mmlu_lower_bound)
```
**EN:** This test exercises `test_mmlu` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mmlu`。

### Lines 57-69: test case gsm8k / 测试用例 gsm8k
```python
    def test_gsm8k(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=200,
            num_threads=128,
        )
        metrics = run_eval(args)
        print(f"{metrics=}")
        self.assertGreater(metrics["score"], self.gsm8k_lower_bound)
```
**EN:** This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 72-73: class ModelCase declaration / 类 ModelCase 声明
```python
@dataclasses.dataclass
class ModelCase:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 74-81: class-level constants and configuration for `ModelCase` / 类级常量与配置
```python
    model_path: str
    tp_size: int = 1
    prefill_tolerance: float = 5e-2
    decode_tolerance: float = 5e-2
    rouge_l_tolerance: float = 1
    skip_long_prompt: bool = False
    trust_remote_code: bool = False
    torch_dtype: torch.dtype = torch.float16
```
**EN:** This block defines shared names such as `model_path`, `tp_size`, `prefill_tolerance`, `decode_tolerance`, `rouge_l_tolerance`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model_path`, `tp_size`, `prefill_tolerance`, `decode_tolerance`, `rouge_l_tolerance` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 82-84: supporting source context / 辅助源码上下文
```python


# Popular models that run on the CI
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 85-91: module-level constants and configuration / 模块级常量与配置
```python
CI_MODELS = [
    ModelCase(DEFAULT_MODEL_NAME_FOR_TEST),
]

ALL_OTHER_MODELS = [
    ModelCase(DEFAULT_MODEL_NAME_FOR_TEST, tp_size=2),
]
```
**EN:** This block defines shared names such as `CI_MODELS`, `ALL_OTHER_MODELS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `CI_MODELS`, `ALL_OTHER_MODELS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 94-94: class TestTransformersFallbackEngine declaration / 类 TestTransformersFallbackEngine 声明
```python
class TestTransformersFallbackEngine(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 95-97: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        mp.set_start_method("spawn", force=True)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 99-135: helper routine assert close logits and output strs / 辅助流程 assert close logits and output strs
```python
    def assert_close_logits_and_output_strs(
        self,
        prompts: List[str],
        model_case: ModelCase,
    ) -> None:
        model_path = model_case.model_path
        max_new_tokens = 32
        # force to use transformers impl
        with SRTRunner(
            model_path,
            tp_size=model_case.tp_size,
            torch_dtype=model_case.torch_dtype,
            model_type="generation",
            model_impl="transformers",
            trust_remote_code=model_case.trust_remote_code,
        ) as srt_runner:
            srt_outputs = srt_runner.forward(prompts, max_new_tokens=max_new_tokens)

        with SRTRunner(
            model_path,
            tp_size=model_case.tp_size,
            torch_dtype=model_case.torch_dtype,
            model_type="generation",
            trust_remote_code=model_case.trust_remote_code,
        ) as srt_runner:
            srt_transformers_outputs = srt_runner.forward(
                prompts, max_new_tokens=max_new_tokens
            )

        check_close_model_outputs(
            hf_outputs=srt_transformers_outputs,
            srt_outputs=srt_outputs,
            prefill_tolerance=model_case.prefill_tolerance,
            decode_tolerance=model_case.decode_tolerance,
            rouge_l_tolerance=model_case.rouge_l_tolerance,
            debug_text=f"model_path={model_path} prompts={prompts}",
        )
```
**EN:** This helper encapsulates `assert_close_logits_and_output_strs` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `assert_close_logits_and_output_strs`，以便周围测试复用准备、执行或校验逻辑。

### Lines 137-144: test case ci models / 测试用例 ci models
```python
    def test_ci_models(self):
        for model_case in CI_MODELS:
            # Skip long prompts for models that do not have a long context
            prompts = DEFAULT_PROMPTS
            if model_case.skip_long_prompt:
                prompts = [p for p in DEFAULT_PROMPTS if len(p) < 1000]
            # Assert the logits and output strs are close
            self.assert_close_logits_and_output_strs(prompts, model_case)
```
**EN:** This test exercises `test_ci_models` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ci_models`。

### Lines 146-157: test case others / 测试用例 others
```python
    def test_others(self):
        if is_in_ci():
            return

        # Skip long prompts for models that do not have a long context
        prompts = DEFAULT_PROMPTS
        for model_case in ALL_OTHER_MODELS:
            if model_case.skip_long_prompt:
                prompts = [p for p in DEFAULT_PROMPTS if len(p) < 1000]

            # Assert the logits and output strs are close
            self.assert_close_logits_and_output_strs(prompts, model_case)
```
**EN:** This test exercises `test_others` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_others`。

### Lines 160-161: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestTransformersFallbackEndpoint`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `ModelCase`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestTransformersFallbackEngine`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestTransformersFallbackEndpoint.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestTransformersFallbackEndpoint.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestTransformersFallbackEndpoint.test_mmlu`: This test exercises `test_mmlu` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mmlu`。
- `TestTransformersFallbackEndpoint.test_gsm8k`: This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。
- `TestTransformersFallbackEngine.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestTransformersFallbackEngine.assert_close_logits_and_output_strs`: This helper encapsulates `assert_close_logits_and_output_strs` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `assert_close_logits_and_output_strs`，以便周围测试复用准备、执行或校验逻辑。
- `TestTransformersFallbackEngine.test_ci_models`: This test exercises `test_ci_models` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ci_models`。
- `TestTransformersFallbackEngine.test_others`: This test exercises `test_others` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_others`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `multiprocessing`, `unittest`, `types`, `typing`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.runners`, `sglang.test.test_utils`

- **Total lines / 总行数**: 161
