# test_qwen35_eval_mi35x.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/accuracy/mi35x/test_qwen35_eval_mi35x.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on qwen35 eval mi35x in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 qwen35 eval mi35x 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Document the module
```python
"""MI35x Qwen 3.5 GSM8K lm-eval Evaluation Test (8-GPU)

Tests Qwen/Qwen3.5-397B-A17B (MoE, Hybrid Attention with Gated Delta Networks)
with lm-eval GSM8K benchmark on MI35x, matching the AMD Day 0 article.

Registry: nightly-amd-accuracy-8-gpu-mi35x-qwen35 suite
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 9-26: Import dependencies
```python
import os
import unittest
from pathlib import Path

import numpy as np
import requests
import yaml

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.kits.lm_eval_kit import LMEvalMixin
from sglang.test.test_utils import (
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    popen_launch_server,
    write_github_step_summary,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 28-30: Register CI metadata
```python
register_amd_ci(
    est_time=3600, suite="nightly-amd-accuracy-8-gpu-mi35x-qwen35", nightly=True
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 32-34: Define module constants
```python
QWEN35_MODEL_PATH = "Qwen/Qwen3.5-397B-A17B"
SERVER_LAUNCH_TIMEOUT = 3600
TP_SIZE = 8
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 37-37: Define class TestQwen35EvalMI35x
```python
class TestQwen35EvalMI35x(LMEvalMixin, CustomTestCase):
```
**EN:** This declaration introduces the `TestQwen35EvalMI35x` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestQwen35EvalMI35x` 测试类，并说明它通过继承承担的职责。

### Lines 38-38: Document the class `TestQwen35EvalMI35x`
```python
    """Qwen 3.5 GSM8K lm-eval Test for AMD MI35x."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestQwen35EvalMI35x`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestQwen35EvalMI35x`的设计意图。

### Lines 40-40: Declare TestQwen35EvalMI35x configuration
```python
    model_config_name = "lm_eval_configs/Qwen3.5-397B-A17B.yaml"
```
**EN:** This block defines class-level settings that are shared across the `TestQwen35EvalMI35x` test methods.
**CN:** 该代码块定义了 `TestQwen35EvalMI35x` 各测试方法共享的类级配置。

### Lines 42-45: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.model = QWEN35_MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 47-103: Run test: lm eval
```python
    def test_lm_eval(self):
        """Override to handle server lifecycle and write results to summary."""
        other_args = [
            "--tp",
            str(TP_SIZE),
            "--attention-backend",
            "aiter",
            "--trust-remote-code",
            "--model-loader-extra-config",
            '{"enable_multithread_load": true}',
            "--watchdog-timeout",
            "1200",
        ]
        env = os.environ.copy()
        env["SGLANG_USE_AITER"] = "1"

        process = popen_launch_server(
            QWEN35_MODEL_PATH,
            self.base_url,
            timeout=SERVER_LAUNCH_TIMEOUT,
            other_args=other_args,
            env=env,
        )

        try:
            requests.get(self.base_url + "/flush_cache")

            eval_config = yaml.safe_load(
                Path(self.model_config_name).read_text(encoding="utf-8")
            )
            results = self.launch_lm_eval(eval_config)
            rtol = eval_config.get("rtol", self.default_rtol)
            model_name = eval_config.get("model_name", self.model)

            success = True
            summary = f"### lm-eval accuracy ({model_name})\n"
            summary += "| task | metric | expected | measured | status |\n"
            summary += "| ---- | ------ | -------- | -------- | ------ |\n"
            for task in eval_config["tasks"]:
                for metric in task["metrics"]:
                    expected = metric["value"]
                    measured = results["results"][task["name"]][metric["name"]]
                    passed = bool(np.isclose(expected, measured, rtol=rtol))
                    status = "✅" if passed else "❌"
                    summary += f"| {task['name']} | {metric['name']} | {expected:.4f} | {measured:.4f} | {status} |\n"
                    print(
                        f"{task['name']} | {metric['name']}: "
                        f"expected={expected:.3f} | measured={measured:.3f} | rtol={rtol}"
                    )
                    success = success and passed

            if is_in_ci():
                write_github_step_summary(summary)

            self.assertTrue(success, "lm-eval validation failed")
        finally:
            kill_process_tree(process.pid)
```
**EN:** This test method exercises lm eval and verifies that the observed behavior matches the expected contract. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该测试方法会执行 lm eval 场景，并验证观测到的行为是否符合预期契约。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 106-107: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Server lifecycle helpers start the target runtime before assertions begin. / 服务生命周期辅助函数会在断言开始前启动目标运行时。
- Explicit teardown avoids leaked child processes between runs. / 显式清理可以避免多次运行之间遗留子进程。
- HTTP requests validate the externally visible API contract rather than only internal helpers. / HTTP 请求验证的是对外暴露的 API 契约，而不只是内部辅助函数。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.lm_eval_kit`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `numpy`, `os`, `pathlib`, `requests`, `unittest`, `yaml`
- Notable symbols / 关键符号: `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `CustomTestCase`, `requests.get`, `unittest.main`
