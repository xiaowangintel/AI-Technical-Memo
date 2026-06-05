# test_minimax_m25_eval_amd.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/accuracy/mi30x/test_minimax_m25_eval_amd.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on mi30x minimax m25 eval amd in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 mi30x minimax m25 eval amd 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Document the module
```python
"""AMD MiniMax-M2.5 GSM8K Completion Evaluation Test (8-GPU)

Tests MiniMax-M2.5 with TP=8 + EP=8 configuration using few-shot completion
benchmark on MI325/MI300X.

Registry: nightly-amd-accuracy-8-gpu-minimax-m25 suite
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 9-28: Import dependencies
```python
import ast
import os
import re
import time
import unittest
from dataclasses import dataclass
from typing import List, Optional, Tuple

import numpy as np

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    is_in_ci,
    popen_launch_server,
    write_github_step_summary,
)
from sglang.utils import download_and_cache_file, read_jsonl
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 30-34: Register CI metadata
```python
register_amd_ci(
    est_time=3600,
    suite="nightly-amd-accuracy-8-gpu-minimax-m25",
    nightly=True,
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 36-36: Define module constants
```python
INVALID = -9999999
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 39-40: Define class ModelConfig
```python
@dataclass
class ModelConfig:
```
**EN:** This declaration introduces the `ModelConfig` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `ModelConfig` 测试类，并说明它通过继承承担的职责。

### Lines 41-41: Document the class `ModelConfig`
```python
    """Configuration for a model to test."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `ModelConfig`.
**CN:** 该代码块保存说明性文本，用于解释周围class `ModelConfig`的设计意图。

### Lines 43-49: Declare ModelConfig configuration
```python
    model_path: str
    tp_size: int = 8
    accuracy_threshold: float = 0.50
    other_args: Optional[List[str]] = None
    env_vars: Optional[dict] = None
    timeout: Optional[int] = None
    variant: Optional[str] = None
```
**EN:** This block defines class-level settings that are shared across the `ModelConfig` test methods.
**CN:** 该代码块定义了 `ModelConfig` 各测试方法共享的类级配置。

### Lines 51-55: Define helper: post init
```python
    def __post_init__(self):
        if self.other_args is None:
            self.other_args = []
        if self.env_vars is None:
            self.env_vars = {}
```
**EN:** This helper function encapsulates reusable logic inside `ModelConfig` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `ModelConfig` 内部调用，从而让场景结构更清晰。

### Lines 57-60: Define helper: get display name
```python
    def get_display_name(self) -> str:
        if self.variant:
            return f"{self.model_path} ({self.variant})"
        return self.model_path
```
**EN:** This helper function encapsulates reusable logic inside `ModelConfig` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `ModelConfig` 内部调用，从而让场景结构更清晰。

### Lines 63-85: Define module constants
```python
MINIMAX_M25_MODELS = [
    ModelConfig(
        model_path="MiniMaxAI/MiniMax-M2.5",
        tp_size=8,
        accuracy_threshold=0.93,
        timeout=3600,
        variant="TP8+EP8",
        other_args=[
            "--ep-size",
            "8",
            "--trust-remote-code",
            "--attention-backend",
            "aiter",
            "--mem-fraction-static",
            "0.85",
            "--model-loader-extra-config",
            '{"enable_multithread_load": true}',
            "--watchdog-timeout",
            "1200",
        ],
        env_vars={"SGLANG_USE_AITER": "1"},
    ),
]
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 88-93: Define helper: get one example
```python
def get_one_example(lines, i, include_answer):
    """Format a single GSM8K example."""
    ret = "Question: " + lines[i]["question"] + "\nAnswer:"
    if include_answer:
        ret += " " + lines[i]["answer"]
    return ret
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 96-101: Define helper: get few shot examples
```python
def get_few_shot_examples(lines, k):
    """Get k few-shot examples for prompting."""
    ret = ""
    for i in range(k):
        ret += get_one_example(lines, i, True) + "\n\n"
    return ret
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 104-113: Define helper: get answer value
```python
def get_answer_value(answer_str):
    """Extract numerical answer from response."""
    answer_str = answer_str.replace(",", "")
    numbers = re.findall(r"\d+", answer_str)
    if len(numbers) < 1:
        return INVALID
    try:
        return ast.literal_eval(numbers[-1])
    except SyntaxError:
        return INVALID
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 116-160: Define helper: run gsm8k benchmark
```python
def run_gsm8k_benchmark(
    base_url: str,
    num_questions: int = 200,
    num_shots: int = 5,
    parallel: int = 64,
) -> Tuple[float, float, float]:
    """Run GSM8K few-shot completion benchmark."""
    import sglang as sgl
    from sglang.lang.backend.runtime_endpoint import RuntimeEndpoint

    url = "https://raw.githubusercontent.com/openai/grade-school-math/master/grade_school_math/data/test.jsonl"
    data_path = download_and_cache_file(url)
    lines = list(read_jsonl(data_path))

    few_shot_examples = get_few_shot_examples(lines, num_shots)

    questions = []
    labels = []
    for i in range(len(lines[:num_questions])):
        questions.append(get_one_example(lines, i, False))
        labels.append(get_answer_value(lines[i]["answer"]))
    assert all(l != INVALID for l in labels)
    arguments = [{"question": q} for q in questions]

    @sgl.function
    def few_shot_gsm8k(s, question):
        s += few_shot_examples + question
        s += sgl.gen(
            "answer", max_tokens=512, stop=["Question", "Assistant:", "<|separator|>"]
        )

    backend = RuntimeEndpoint(base_url)
    sgl.set_default_backend(backend)

    tic = time.perf_counter()
    states = few_shot_gsm8k.run_batch(
        arguments, temperature=0, num_threads=parallel, progress_bar=True
    )
    latency = time.perf_counter() - tic

    preds = [get_answer_value(states[i]["answer"]) for i in range(len(states))]
    acc = np.mean(np.array(preds) == np.array(labels))
    invalid = np.mean(np.array(preds) == INVALID)

    return float(acc), float(invalid), float(latency)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 163-163: Define class TestMiniMaxM25EvalAMD
```python
class TestMiniMaxM25EvalAMD(unittest.TestCase):
```
**EN:** This declaration introduces the `TestMiniMaxM25EvalAMD` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMiniMaxM25EvalAMD` 测试类，并说明它通过继承承担的职责。

### Lines 164-164: Document the class `TestMiniMaxM25EvalAMD`
```python
    """MiniMax-M2.5 GSM8K Completion Evaluation Test for AMD MI325/MI300X."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestMiniMaxM25EvalAMD`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestMiniMaxM25EvalAMD`的设计意图。

### Lines 166-170: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.models = MINIMAX_M25_MODELS
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.num_questions = int(os.environ.get("GSM8K_NUM_QUESTIONS", "200"))
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 172-223: Run test: minimax m25 accuracy (part 1)
```python
    def test_minimax_m25_accuracy(self):
        """Test MiniMax-M2.5 with GSM8K completion benchmark."""
        all_results = []
        summary = "### MiniMax-M2.5 Models (MI325)\n\n"
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
                        acc, invalid, latency = run_gsm8k_benchmark(
                            self.base_url, num_questions=self.num_questions
                        )
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
```
**EN:** This test method exercises minimax m25 accuracy and verifies that the observed behavior matches the expected contract. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该测试方法会执行 minimax m25 accuracy 场景，并验证观测到的行为是否符合预期契约。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 225-237: Run test: minimax m25 accuracy (part 2)
```python
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
**EN:** This test method exercises minimax m25 accuracy and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 minimax m25 accuracy 场景，并验证观测到的行为是否符合预期契约。

### Lines 239-241: Run test: minimax m25 accuracy (part 3)
```python
        failed = [r for r in all_results if not r["passed"]]
        if failed:
            raise AssertionError(f"Failed models: {[r['model'] for r in failed]}")
```
**EN:** This test method exercises minimax m25 accuracy and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 minimax m25 accuracy 场景，并验证观测到的行为是否符合预期契约。

### Lines 244-245: Expose unittest entrypoint
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
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang`, `sglang.lang.backend.runtime_endpoint`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`, `sglang.utils`
- External and stdlib modules / 外部与标准库模块: `ast`, `dataclasses`, `numpy`, `os`, `re`, `time`, `typing`, `unittest`
- Notable symbols / 关键符号: `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `unittest.main`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
