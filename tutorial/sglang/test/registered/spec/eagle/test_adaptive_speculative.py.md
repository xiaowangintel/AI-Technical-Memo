# test_adaptive_speculative.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/spec/eagle/test_adaptive_speculative.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates adaptive speculative behavior in SGLang's spec / eagle area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 推测解码 / eagle 领域中与 adaptive speculative 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: module imports and dependencies / 模块导入与依赖
```python
import json
import os
import tempfile
import unittest
from types import SimpleNamespace

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_DRAFT_MODEL_EAGLE,
    DEFAULT_TARGET_MODEL_EAGLE,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `os`, `tempfile`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `os`, `tempfile`, `unittest`。

### Lines 21-35: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=76, stage="base-b", runner_config="1-gpu-large")

HIGH_ACCEPT_PROMPT = (
    "Output exactly 128 new lines. "
    "Every line must be READY. "
    "Do not add numbering, punctuation, or commentary."
)

LOW_ACCEPT_PROMPT = (
    "Compose a poem in the style of Emily Dickinson about quantum entanglement. "
    "Make it emotionally resonant and at least 100 words."
)

MAX_UPSHIFT_ATTEMPTS = 4
MAX_DOWNSHIFT_ATTEMPTS = 6
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 38-38: class TestAdaptiveSpeculativeServer declaration / 类 TestAdaptiveSpeculativeServer 声明
```python
class TestAdaptiveSpeculativeServer(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 39-43: class-level constants and configuration for `TestAdaptiveSpeculativeServer` / 类级常量与配置
```python
    """Test adaptive speculative decoding with state switching and GSM8K accuracy."""

    model = DEFAULT_TARGET_MODEL_EAGLE
    draft_model = DEFAULT_DRAFT_MODEL_EAGLE
    base_url = DEFAULT_URL_FOR_TEST
```
**EN:** This block defines shared names such as `model`, `draft_model`, `base_url`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `draft_model`, `base_url` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 45-89: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        with tempfile.NamedTemporaryFile("w", suffix=".json", delete=False) as f:
            json.dump(
                {
                    "candidate_steps": [1, 3],
                    "ema_alpha": 1.0,
                    "warmup_batches": 1,
                    "update_interval": 1,
                    "up_hysteresis": 0.0,
                },
                f,
            )
            cls.adaptive_config_path = f.name

        try:
            cls.process = popen_launch_server(
                cls.model,
                cls.base_url,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=[
                    "--trust-remote-code",
                    "--attention-backend",
                    "triton",
                    "--speculative-algorithm",
                    "EAGLE",
                    "--speculative-draft-model-path",
                    cls.draft_model,
                    "--speculative-num-steps",
                    "1",
                    "--speculative-eagle-topk",
                    "1",
                    "--speculative-num-draft-tokens",
                    "2",
                    "--speculative-adaptive",
                    "--speculative-adaptive-config",
                    cls.adaptive_config_path,
                    "--skip-server-warmup",
                    "--mem-fraction-static",
                    "0.7",
                ],
            )
        except Exception:
            os.unlink(cls.adaptive_config_path)
            raise
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 91-96: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if hasattr(cls, "process"):
            kill_process_tree(cls.process.pid)
        if os.path.exists(cls.adaptive_config_path):
            os.unlink(cls.adaptive_config_path)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 98-101: method get internal state / 方法 get internal state
```python
    def _get_internal_state(self) -> dict:
        response = requests.get(self.base_url + "/server_info", timeout=30)
        self.assertEqual(response.status_code, 200, response.text)
        return response.json()["internal_states"][0]
```
**EN:** This block implements `_get_internal_state` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_get_internal_state`，承担模块行为中的一个聚焦逻辑片段。

### Lines 103-117: method generate / 方法 generate
```python
    def _generate(self, prompt: str, max_new_tokens: int = 64) -> dict:
        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": prompt,
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": max_new_tokens,
                    "ignore_eos": True,
                },
            },
            timeout=180,
        )
        self.assertEqual(response.status_code, 200, response.text)
        return response.json()
```
**EN:** This block implements `_generate` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_generate`，承担模块行为中的一个聚焦逻辑片段。

### Lines 119-127: method drive upshift / 方法 drive upshift
```python
    def _drive_upshift(self) -> dict:
        """Send high-acceptance prompts until steps upshift to 3."""
        state = self._get_internal_state()
        for _ in range(MAX_UPSHIFT_ATTEMPTS):
            self._generate(HIGH_ACCEPT_PROMPT)
            state = self._get_internal_state()
            if state["speculative_num_steps"] == 3:
                return state
        return state
```
**EN:** Send high-acceptance prompts until steps upshift to 3. This block implements `_drive_upshift` and captures one focused piece of the module's behavior.
**CN:** Send high-acceptance prompts until steps upshift to 3. 该代码块实现 `_drive_upshift`，承担模块行为中的一个聚焦逻辑片段。

### Lines 129-137: method drive downshift / 方法 drive downshift
```python
    def _drive_downshift(self) -> dict:
        """Send low-acceptance prompts until steps downshift to 1."""
        state = self._get_internal_state()
        for _ in range(MAX_DOWNSHIFT_ATTEMPTS):
            self._generate(LOW_ACCEPT_PROMPT)
            state = self._get_internal_state()
            if state["speculative_num_steps"] == 1:
                return state
        return state
```
**EN:** Send low-acceptance prompts until steps downshift to 1. This block implements `_drive_downshift` and captures one focused piece of the module's behavior.
**CN:** Send low-acceptance prompts until steps downshift to 1. 该代码块实现 `_drive_downshift`，承担模块行为中的一个聚焦逻辑片段。

### Lines 139-166: test case gsm8k after adaptive switches / 测试用例 gsm8k after adaptive switches
```python
    def test_gsm8k_after_adaptive_switches(self):
        """Exercise up/down/up adaptive switches, then verify GSM8K accuracy."""
        state = self._drive_upshift()
        self.assertEqual(state["speculative_num_steps"], 3, f"Never upshifted: {state}")

        state = self._drive_downshift()
        self.assertEqual(
            state["speculative_num_steps"], 1, f"Never downshifted: {state}"
        )

        self._drive_upshift()

        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            api="completion",
            max_tokens=512,
            num_examples=100,
            num_threads=64,
        )
        metrics = run_eval(args)
        print(f"GSM8K after adaptive switches: {metrics}")
        self.assertGreater(metrics["score"], 0.20)

        server_info = requests.get(self.base_url + "/server_info").json()
        avg_accept_len = server_info["internal_states"][0]["avg_spec_accept_length"]
        print(f"avg_spec_accept_length={avg_accept_len:.4f}")
```
**EN:** Exercise up/down/up adaptive switches, then verify GSM8K accuracy. This test exercises `test_gsm8k_after_adaptive_switches` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Exercise up/down/up adaptive switches, then verify GSM8K accuracy. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k_after_adaptive_switches`。

### Lines 169-170: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestAdaptiveSpeculativeServer`: Test adaptive speculative decoding with state switching and GSM8K accuracy. / 用于组织相关测试、夹具或辅助方法。
- `TestAdaptiveSpeculativeServer.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestAdaptiveSpeculativeServer.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestAdaptiveSpeculativeServer._get_internal_state`: This block implements `_get_internal_state` and captures one focused piece of the module's behavior. / 该代码块实现 `_get_internal_state`，承担模块行为中的一个聚焦逻辑片段。
- `TestAdaptiveSpeculativeServer._generate`: This block implements `_generate` and captures one focused piece of the module's behavior. / 该代码块实现 `_generate`，承担模块行为中的一个聚焦逻辑片段。
- `TestAdaptiveSpeculativeServer._drive_upshift`: Send high-acceptance prompts until steps upshift to 3. / 该代码块实现 `_drive_upshift`，承担模块行为中的一个聚焦逻辑片段。
- `TestAdaptiveSpeculativeServer._drive_downshift`: Send low-acceptance prompts until steps downshift to 1. / 该代码块实现 `_drive_downshift`，承担模块行为中的一个聚焦逻辑片段。
- `TestAdaptiveSpeculativeServer.test_gsm8k_after_adaptive_switches`: Exercise up/down/up adaptive switches, then verify GSM8K accuracy. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k_after_adaptive_switches`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `tempfile`, `unittest`, `types`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`

- **Total lines / 总行数**: 170
