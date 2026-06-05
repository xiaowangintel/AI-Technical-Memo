# test_fp32_lm_head.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/rl/test_fp32_lm_head.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates fp32 lm head behavior in SGLang's rl area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 rl 领域中与 fp32 lm head 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace
from unittest.mock import patch

import torch
import torch.nn as nn
import torch.nn.functional as F

from sglang.srt.layers.logits_processor import LogitsProcessor
from sglang.srt.server_args import (
    ServerArgs,
    get_global_server_args,
    set_global_server_args_for_scheduler,
)
from sglang.srt.utils import get_device
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `unittest.mock`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `unittest.mock`, `torch`。

### Lines 18-19: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=9, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=15, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 22-22: class LMHeadStub declaration / 类 LMHeadStub 声明
```python
class LMHeadStub(nn.Module):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `nn.Module`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `nn.Module`。

### Lines 23-27: method init / 方法 init
```python
    def __init__(self, vocab, hidden, dtype, device=get_device()):
        super().__init__()
        self.weight = nn.Parameter(
            torch.randn(vocab, hidden, dtype=dtype, device=device)
        )
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 30-30: class DummyMeta declaration / 类 DummyMeta 声明
```python
class DummyMeta:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 31-32: class-level constants and configuration for `DummyMeta` / 类级常量与配置
```python
    gathered_buffer = None
    next_token_logits_buffer = None
```
**EN:** This block defines shared names such as `gathered_buffer`, `next_token_logits_buffer`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `gathered_buffer`, `next_token_logits_buffer` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 34-34: method compute dp attention metadata / 方法 compute dp attention metadata
```python
    def compute_dp_attention_metadata(self): ...
```
**EN:** This block implements `compute_dp_attention_metadata` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `compute_dp_attention_metadata`，承担模块行为中的一个聚焦逻辑片段。

### Lines 37-37: class TestLMHeadFP32 declaration / 类 TestLMHeadFP32 声明
```python
class TestLMHeadFP32(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 38-43: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        if not torch.cuda.is_available() and not (
            hasattr(torch, "xpu") and torch.xpu.is_available()
        ):
            raise unittest.SkipTest("needs CUDA GPU or XPU")
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 45-50: method make logprocessor / 方法 make logprocessor
```python
    def _make_logprocessor(self, vocab_size, enable_fp32):
        set_global_server_args_for_scheduler(ServerArgs(model_path="dummy"))
        get_global_server_args().enable_dp_lm_head = False
        get_global_server_args().enable_fp32_lm_head = enable_fp32
        cfg = SimpleNamespace(vocab_size=vocab_size, final_logit_softcapping=None)
        return LogitsProcessor(cfg, skip_all_gather=True, logit_scale=None)
```
**EN:** This block implements `_make_logprocessor` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_logprocessor`，承担模块行为中的一个聚焦逻辑片段。

### Lines 52-97: method run case / 方法 run case
```python
    def _run_case(
        self,
        hidden_state_dtype,
        enable_fp32,
        weights_dtype,
        expected_a_dtype,
        expected_b_dtype,
    ):
        device = get_device()
        BATCH_SIZE, HIDDEN_SIZE, VOCAB_SIZE = 2, 64, 128
        hidden_state = torch.randn(
            BATCH_SIZE, HIDDEN_SIZE, dtype=hidden_state_dtype, device=device
        )
        head = LMHeadStub(VOCAB_SIZE, HIDDEN_SIZE, dtype=weights_dtype, device=device)
        meta = DummyMeta()
        logprocessor = self._make_logprocessor(VOCAB_SIZE, enable_fp32)

        original_matmul = torch.matmul
        original_linear = F.linear

        state = {
            "called": False,  # Whether a matmul/linear call has been intercepted yet
            "operation": None,  # Which operation was captured ("matmul" or "linear")
            "a": None,  # The dtype of the first input tensor to the operation
            "b": None,  # The dtype of the second input tensor to the operation
        }

        def probe_matmul(a, b, *args, **kw):
            if not state["called"]:
                state.update(called=True, operation="matmul", a=a.dtype, b=b.dtype)
            return original_matmul(a, b, *args, **kw)

        def probe_linear(x, w, bias=None):
            if not state["called"]:
                state.update(called=True, ooperationp="linear", a=x.dtype, b=w.dtype)
            return original_linear(x, w, bias)

        with (
            patch("torch.matmul", new=probe_matmul),
            patch("torch.nn.functional.linear", new=probe_linear),
        ):
            logits = logprocessor._get_logits(hidden_state, head, meta)
        self.assertEqual(hidden_state.dtype, hidden_state_dtype)
        self.assertTrue(state["called"], "no call lm head matlmul/linear")
        self.assertEqual(state["a"], expected_a_dtype)
        self.assertEqual(state["b"], expected_b_dtype)
```
**EN:** This block implements `_run_case` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_case`，承担模块行为中的一个聚焦逻辑片段。

### Lines 99-100: test case flag true fp16 activations / 测试用例 flag true fp16 activations
```python
    def test_flag_true_fp16_activations(self):
        self._run_case(torch.float16, True, torch.float16, torch.float32, torch.float32)
```
**EN:** This test exercises `test_flag_true_fp16_activations` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_flag_true_fp16_activations`。

### Lines 102-105: test case flag true bf16 activations / 测试用例 flag true bf16 activations
```python
    def test_flag_true_bf16_activations(self):
        self._run_case(
            torch.bfloat16, True, torch.bfloat16, torch.float32, torch.float32
        )
```
**EN:** This test exercises `test_flag_true_bf16_activations` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_flag_true_bf16_activations`。

### Lines 107-110: test case flag false fp16 path / 测试用例 flag false fp16 path
```python
    def test_flag_false_fp16_path(self):
        self._run_case(
            torch.float16, False, torch.float16, torch.float16, torch.float16
        )
```
**EN:** This test exercises `test_flag_false_fp16_path` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_flag_false_fp16_path`。

### Lines 112-115: test case flag false bf16 path / 测试用例 flag false bf16 path
```python
    def test_flag_false_bf16_path(self):
        self._run_case(
            torch.bfloat16, False, torch.bfloat16, torch.bfloat16, torch.bfloat16
        )
```
**EN:** This test exercises `test_flag_false_bf16_path` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_flag_false_bf16_path`。

### Lines 118-119: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `LMHeadStub`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `DummyMeta`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLMHeadFP32`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `LMHeadStub.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `DummyMeta.compute_dp_attention_metadata`: This block implements `compute_dp_attention_metadata` and captures one focused piece of the module's behavior. / 该代码块实现 `compute_dp_attention_metadata`，承担模块行为中的一个聚焦逻辑片段。
- `TestLMHeadFP32.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestLMHeadFP32._make_logprocessor`: This block implements `_make_logprocessor` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_logprocessor`，承担模块行为中的一个聚焦逻辑片段。
- `TestLMHeadFP32._run_case`: This block implements `_run_case` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_case`，承担模块行为中的一个聚焦逻辑片段。
- `TestLMHeadFP32.test_flag_true_fp16_activations`: This test exercises `test_flag_true_fp16_activations` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_flag_true_fp16_activations`。
- `TestLMHeadFP32.test_flag_true_bf16_activations`: This test exercises `test_flag_true_bf16_activations` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_flag_true_bf16_activations`。
- `TestLMHeadFP32.test_flag_false_fp16_path`: This test exercises `test_flag_false_fp16_path` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_flag_false_fp16_path`。
- `TestLMHeadFP32.test_flag_false_bf16_path`: This test exercises `test_flag_false_bf16_path` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_flag_false_bf16_path`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`, `unittest.mock`
- **Third-party modules / 第三方模块**: `torch`, `torch.nn`, `torch.nn.functional`
- **Internal modules / 内部模块**: `sglang.srt.layers.logits_processor`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 119
