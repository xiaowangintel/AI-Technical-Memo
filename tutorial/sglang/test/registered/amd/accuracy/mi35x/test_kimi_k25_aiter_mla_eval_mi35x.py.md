# test_kimi_k25_aiter_mla_eval_mi35x.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/accuracy/mi35x/test_kimi_k25_aiter_mla_eval_mi35x.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on kimi k25 aiter mla eval mi35x in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 kimi k25 aiter mla eval mi35x 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: Document the module
```python
"""MI35x Kimi-K2.5 aiter MLA backend accuracy tests (4-GPU)

Tests moonshotai/Kimi-K2.5 with the aiter unified attention backend on MI35x,
covering both default and FP8 KV cache configurations.

The FP8 KV cache variant validates the fix for assertion failure
`q_scale.has_value() && kv_scale.has_value()` in aiter ASM MLA decode
when layer.k_scale is None (the RadixAttention default).

NOTE: TP must be <= 4 for Kimi-K2.5 with the aiter MLA kernel.
Kimi-K2.5 has num_attention_heads=64; with tp_size=8 that gives
64/8 = 8 heads per GPU, but the aiter ASM MLA kernel requires
heads_per_gpu % 16 == 0. With tp_size=4: 64/4 = 16 heads, which
satisfies the constraint. (DeepSeek-R1/V3 has 128 heads so TP=8
yields 128/8 = 16 heads and works fine.)

Registry: nightly-amd-8-gpu-mi35x-kimi-k25-aiter-mla suite
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module. It also checks concrete expectations with assertions.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。 其中还会通过断言检查明确的预期。

### Lines 20-20: Import dependencies
```python
import os
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 22-23: Implement expr logic
```python
os.environ.setdefault("HF_HOME", "/data2/models/huggingface")
os.environ.setdefault("HF_HUB_CACHE", "/data2/models/huggingface/hub")
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 25-37: Import dependencies
```python
import unittest
from dataclasses import dataclass
from typing import List, Optional

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    is_in_ci,
    popen_launch_server,
    write_github_step_summary,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 39-41: Register CI metadata
```python
register_amd_ci(
    est_time=7200, suite="nightly-amd-8-gpu-mi35x-kimi-k25-aiter-mla", nightly=True
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 43-44: Define module constants
```python
KIMI_K25_LOCAL_PATH = "/data/models/amd/Kimi-K2.5"
KIMI_K25_HF_MODEL_ID = "moonshotai/Kimi-K2.5"
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 47-54: Define helper: get model path
```python
def get_model_path() -> str:
    """Get effective model path: env var > local path > HF model ID."""
    env_path = os.environ.get("KIMI_K25_MODEL_PATH")
    if env_path:
        return env_path
    if os.path.exists(KIMI_K25_LOCAL_PATH):
        return KIMI_K25_LOCAL_PATH
    return KIMI_K25_HF_MODEL_ID
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 57-58: Define class ModelConfig
```python
@dataclass
class ModelConfig:
```
**EN:** This declaration introduces the `ModelConfig` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `ModelConfig` 测试类，并说明它通过继承承担的职责。

### Lines 59-59: Document the class `ModelConfig`
```python
    """Configuration for a model variant to test."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `ModelConfig`.
**CN:** 该代码块保存说明性文本，用于解释周围class `ModelConfig`的设计意图。

### Lines 61-67: Declare ModelConfig configuration
```python
    model_path: str
    tp_size: int = 4
    accuracy_threshold: float = 0.92
    other_args: Optional[List[str]] = None
    env_vars: Optional[dict] = None
    timeout: Optional[int] = None
    variant: Optional[str] = None
```
**EN:** This block defines class-level settings that are shared across the `ModelConfig` test methods.
**CN:** 该代码块定义了 `ModelConfig` 各测试方法共享的类级配置。

### Lines 69-73: Define helper: post init
```python
    def __post_init__(self):
        if self.other_args is None:
            self.other_args = []
        if self.env_vars is None:
            self.env_vars = {}
```
**EN:** This helper function encapsulates reusable logic inside `ModelConfig` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `ModelConfig` 内部调用，从而让场景结构更清晰。

### Lines 75-78: Define helper: get display name
```python
    def get_display_name(self) -> str:
        if self.variant:
            return f"{self.model_path} ({self.variant})"
        return self.model_path
```
**EN:** This helper function encapsulates reusable logic inside `ModelConfig` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `ModelConfig` 内部调用，从而让场景结构更清晰。

### Lines 81-127: Define helper: get kimi k25 models
```python
def get_kimi_k25_models() -> List[ModelConfig]:
    """Get Kimi-K2.5 model configurations for MI35x."""
    model_path = get_model_path()
    common_kwargs = {
        "model_path": model_path,
        # TP=4 required: Kimi-K2.5 has 64 attn heads; aiter ASM MLA needs
        # heads_per_gpu % 16 == 0 → 64/4=16 works, 64/8=8 does not.
        "tp_size": 4,
        "accuracy_threshold": 0.92,
        "timeout": 3600,
    }
    common_args = [
        "--attention-backend",
        "aiter",
        "--chunked-prefill-size",
        "131072",
        "--disable-radix-cache",
        "--mem-fraction-static",
        "0.8",
        "--max-running-requests",
        "64",
        "--trust-remote-code",
        "--watchdog-timeout",
        "1200",
    ]
    common_env = {"SGLANG_AITER_MLA_PERSIST": "1"}

    return [
        ModelConfig(
            **common_kwargs,
            variant="default",
            other_args=common_args,
            env_vars=common_env,
        ),
        # FP8 KV cache — validates the k_scale None fallback fix in
        # aiter ASM MLA decode (all 4 mla_decode_fwd call sites).
        ModelConfig(
            **common_kwargs,
            variant="fp8kv",
            other_args=common_args
            + [
                "--kv-cache-dtype",
                "fp8_e4m3",
            ],
            env_vars=common_env,
        ),
    ]
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 130-130: Define class TestKimiK25AiterMlaEvalMI35x
```python
class TestKimiK25AiterMlaEvalMI35x(unittest.TestCase):
```
**EN:** This declaration introduces the `TestKimiK25AiterMlaEvalMI35x` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestKimiK25AiterMlaEvalMI35x` 测试类，并说明它通过继承承担的职责。

### Lines 131-131: Document the class `TestKimiK25AiterMlaEvalMI35x`
```python
    """Kimi-K2.5 aiter MLA backend accuracy tests on MI35x."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestKimiK25AiterMlaEvalMI35x`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestKimiK25AiterMlaEvalMI35x`的设计意图。

### Lines 133-137: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.models = get_kimi_k25_models()
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.num_questions = int(os.environ.get("GSM8K_NUM_QUESTIONS", "1319"))
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 139-197: Run test: kimi k25 accuracy (part 1)
```python
    def test_kimi_k25_accuracy(self):
        """Test Kimi-K2.5 with GSM8K completion benchmark (default & fp8kv)."""
        model_path = get_model_path()
        is_local_path = model_path.startswith("/")
        if is_local_path and not os.path.exists(model_path):
            print(f"\nSKIPPING: Local model not found at {model_path}")
            self.skipTest(f"Local model not found at {model_path}")
            return

        if is_local_path:
            print(f"Using local model: {model_path}")
        else:
            print(f"Using HuggingFace model: {model_path}")

        from types import SimpleNamespace

        from sglang.test.few_shot_gsm8k import run_eval as run_eval_few_shot_gsm8k

        all_results = []
        summary = "### Kimi-K2.5 aiter MLA (MI35x)\n\n"
        summary += "| Model | Variant | TP | Accuracy | Threshold | Status |\n"
        summary += "| ----- | ------- | -- | -------- | --------- | ------ |\n"

        for config in self.models:
            display_name = config.get_display_name()
            with self.subTest(model=display_name):
                print(f"\n{'='*60}")
                print(f"Testing: {display_name}")
                print(f"{'='*60}")

                env = os.environ.copy()
                for key, value in config.env_vars.items():
                    env[key] = value

                other_args = list(config.other_args)
                other_args.extend(["--tp", str(config.tp_size)])
                timeout = config.timeout or DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH

                try:
                    process = popen_launch_server(
                        model=config.model_path,
                        base_url=self.base_url,
                        timeout=timeout,
                        other_args=other_args,
                        env=env,
                    )

                    try:
                        args = SimpleNamespace(
                            num_shots=8,
                            data_path=None,
                            num_questions=self.num_questions,
                            parallel=self.num_questions,
                            max_new_tokens=512,
                            host="http://127.0.0.1",
                            port=int(self.base_url.split(":")[-1]),
                        )
                        metrics = run_eval_few_shot_gsm8k(args)
                        acc = metrics["accuracy"]
```
**EN:** This test method exercises kimi k25 accuracy and verifies that the observed behavior matches the expected contract. It also launches a model server for the scenario and runs an evaluation workflow to measure model quality.
**CN:** 该测试方法会执行 kimi k25 accuracy 场景，并验证观测到的行为是否符合预期契约。 其中还会启动场景所需的模型服务，并运行评测流程以衡量模型质量。

### Lines 199-229: Run test: kimi k25 accuracy (part 2)
```python
                        passed = acc >= config.accuracy_threshold
                        status = "PASS" if passed else "FAIL"
                        print(
                            f"  accuracy={acc:.3f} threshold={config.accuracy_threshold} {status}"
                        )

                        all_results.append(
                            {
                                "model": display_name,
                                "accuracy": acc,
                                "passed": passed,
                            }
                        )
                        summary += f"| {config.model_path} | {config.variant or 'N/A'} | {config.tp_size} | {acc:.3f} | {config.accuracy_threshold} | {status} |\n"

                    finally:
                        kill_process_tree(process.pid)

                except Exception as e:
                    summary += f"| {config.model_path} | {config.variant or 'N/A'} | {config.tp_size} | N/A | {config.accuracy_threshold} | ERROR |\n"
                    all_results.append(
                        {
                            "model": display_name,
                            "accuracy": None,
                            "passed": False,
                            "error": str(e),
                        }
                    )

        if is_in_ci():
            write_github_step_summary(summary)
```
**EN:** This test method exercises kimi k25 accuracy and verifies that the observed behavior matches the expected contract. It also releases spawned processes after the checks finish.
**CN:** 该测试方法会执行 kimi k25 accuracy 场景，并验证观测到的行为是否符合预期契约。 其中还会在检查完成后释放已启动的进程。

### Lines 231-233: Run test: kimi k25 accuracy (part 3)
```python
        failed = [r for r in all_results if not r["passed"]]
        if failed:
            raise AssertionError(f"Failed models: {[r['model'] for r in failed]}")
```
**EN:** This test method exercises kimi k25 accuracy and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 kimi k25 accuracy 场景，并验证观测到的行为是否符合预期契约。

### Lines 236-237: Expose unittest entrypoint
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
- Evaluation helpers convert model behavior into reproducible benchmark-style metrics. / 评测辅助工具会把模型行为转化为可复现的基准指标。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.few_shot_gsm8k`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `dataclasses`, `os`, `types`, `typing`, `unittest`
- Notable symbols / 关键符号: `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `run_eval`, `SimpleNamespace`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
