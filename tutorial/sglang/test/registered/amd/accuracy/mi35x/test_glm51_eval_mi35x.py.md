# test_glm51_eval_mi35x.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/accuracy/mi35x/test_glm51_eval_mi35x.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on glm51 eval mi35x in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 glm51 eval mi35x 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Document the module
```python
"""MI35x GLM-5.1 GSM8K Completion Evaluation Test (8-GPU)

Tests GLM-5.1-FP8 with NSA attention backend using few-shot
completion benchmark on MI35x.

Registry: nightly-amd-8-gpu-mi35x-glm51 suite
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 9-10: Import dependencies
```python
import ast
import os
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 12-13: Implement expr logic
```python
os.environ.setdefault("HF_HOME", "/data2/models/huggingface")
os.environ.setdefault("HF_HUB_CACHE", "/data2/models/huggingface/hub")
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 15-32: Import dependencies
```python
import re
import time
import unittest
from dataclasses import dataclass, field
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

### Lines 34-38: Register CI metadata
```python
register_amd_ci(
    est_time=5400,
    suite="nightly-amd-8-gpu-mi35x-glm51",
    nightly=True,
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 40-40: Define module constants
```python
INVALID = -9999999
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 43-44: Define class ModelConfig
```python
@dataclass
class ModelConfig:
```
**EN:** This declaration introduces the `ModelConfig` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `ModelConfig` 测试类，并说明它通过继承承担的职责。

### Lines 45-51: Declare ModelConfig configuration
```python
    model_path: str
    tp_size: int = 8
    accuracy_threshold: float = 0.50
    other_args: List[str] = field(default_factory=list)
    env_vars: dict = field(default_factory=dict)
    timeout: Optional[int] = None
    variant: Optional[str] = None
```
**EN:** This block defines class-level settings that are shared across the `ModelConfig` test methods.
**CN:** 该代码块定义了 `ModelConfig` 各测试方法共享的类级配置。

### Lines 53-56: Define helper: get display name
```python
    def get_display_name(self) -> str:
        if self.variant:
            return f"{self.model_path} ({self.variant})"
        return self.model_path
```
**EN:** This helper function encapsulates reusable logic inside `ModelConfig` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `ModelConfig` 内部调用，从而让场景结构更清晰。

### Lines 59-87: Define module constants
```python
MI35X_GLM51_MODELS = [
    ModelConfig(
        model_path="zai-org/GLM-5.1-FP8",
        tp_size=8,
        accuracy_threshold=0.93,
        timeout=5400,
        variant="nsa",
        other_args=[
            "--trust-remote-code",
            "--reasoning-parser",
            "glm45",
            "--tool-call-parser",
            "glm47",
            "--nsa-prefill-backend",
            "tilelang",
            "--nsa-decode-backend",
            "tilelang",
            "--chunked-prefill-size",
            "131072",
            "--mem-fraction-static",
            "0.80",
            "--model-loader-extra-config",
            '{"enable_multithread_load": true}',
            "--watchdog-timeout",
            "1200",
        ],
        env_vars={},
    ),
]
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 90-94: Define helper: get one example
```python
def get_one_example(lines, i, include_answer):
    ret = "Question: " + lines[i]["question"] + "\nAnswer:"
    if include_answer:
        ret += " " + lines[i]["answer"]
    return ret
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 97-101: Define helper: get few shot examples
```python
def get_few_shot_examples(lines, k):
    ret = ""
    for i in range(k):
        ret += get_one_example(lines, i, True) + "\n\n"
    return ret
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 104-112: Define helper: get answer value
```python
def get_answer_value(answer_str):
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

### Lines 115-158: Define helper: run gsm8k benchmark
```python
def run_gsm8k_benchmark(
    base_url: str,
    num_questions: int = 200,
    num_shots: int = 5,
    parallel: int = 64,
) -> Tuple[float, float, float]:
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

### Lines 161-161: Define class TestGLM51EvalMI35x
```python
class TestGLM51EvalMI35x(unittest.TestCase):
```
**EN:** This declaration introduces the `TestGLM51EvalMI35x` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGLM51EvalMI35x` 测试类，并说明它通过继承承担的职责。

### Lines 162-162: Document the class `TestGLM51EvalMI35x`
```python
    """GLM-5.1 GSM8K Completion Evaluation Test for AMD MI35x."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestGLM51EvalMI35x`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestGLM51EvalMI35x`的设计意图。

### Lines 164-168: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.models = MI35X_GLM51_MODELS
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.num_questions = int(os.environ.get("GSM8K_NUM_QUESTIONS", "200"))
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 170-220: Run test: glm51 accuracy (part 1)
```python
    def test_glm51_accuracy(self):
        all_results = []
        summary = "### GLM-5.1 Models (MI35x)\n\n"
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
                        status = "✅ PASS" if passed else "❌ FAIL"
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
**EN:** This test method exercises glm51 accuracy and verifies that the observed behavior matches the expected contract. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该测试方法会执行 glm51 accuracy 场景，并验证观测到的行为是否符合预期契约。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 222-234: Run test: glm51 accuracy (part 2)
```python
                except Exception as e:
                    summary += f"| {config.model_path} | {config.variant or 'N/A'} | {config.tp_size} | N/A | {config.accuracy_threshold} | ❌ ERROR |\n"
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
**EN:** This test method exercises glm51 accuracy and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 glm51 accuracy 场景，并验证观测到的行为是否符合预期契约。

### Lines 236-238: Run test: glm51 accuracy (part 3)
```python
        failed = [r for r in all_results if not r["passed"]]
        if failed:
            raise AssertionError(f"Failed models: {[r['model'] for r in failed]}")
```
**EN:** This test method exercises glm51 accuracy and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 glm51 accuracy 场景，并验证观测到的行为是否符合预期契约。

### Lines 241-242: Expose unittest entrypoint
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
