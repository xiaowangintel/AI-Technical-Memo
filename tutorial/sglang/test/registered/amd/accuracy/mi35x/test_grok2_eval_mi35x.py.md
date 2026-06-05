# test_grok2_eval_mi35x.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/accuracy/mi35x/test_grok2_eval_mi35x.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on grok2 eval mi35x in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 grok2 eval mi35x 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Document the module
```python
"""MI35x GROK2 GSM8K Completion Evaluation Test (8-GPU)

Tests Grok-2 model using few-shot completion benchmark on MI35x.

Registry: nightly-amd-accuracy-8-gpu-mi35x-grok2 suite
"""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 8-24: Import dependencies
```python
import ast
import os
import re
import time
import unittest

import numpy as np

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci
from sglang.test.test_utils import (
    DEFAULT_URL_FOR_TEST,
    is_in_ci,
    popen_launch_server,
    write_github_step_summary,
)
from sglang.utils import download_and_cache_file, read_jsonl
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 27-29: Register CI metadata
```python
register_amd_ci(
    est_time=1500, suite="nightly-amd-accuracy-8-gpu-mi35x-grok2", nightly=True
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 31-36: Define module constants
```python
INVALID = -9999999

GROK2_MODEL_PATH = os.environ.get("GROK2_MODEL_PATH", "xai-org/grok-2")
GROK2_TOKENIZER_PATH = os.environ.get(
    "GROK2_TOKENIZER_PATH", "alvarobartt/grok-2-tokenizer"
)
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 39-43: Define helper: get one example
```python
def get_one_example(lines, i, include_answer):
    ret = "Question: " + lines[i]["question"] + "\nAnswer:"
    if include_answer:
        ret += " " + lines[i]["answer"]
    return ret
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 46-50: Define helper: get few shot examples
```python
def get_few_shot_examples(lines, k):
    ret = ""
    for i in range(k):
        ret += get_one_example(lines, i, True) + "\n\n"
    return ret
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 53-61: Define helper: get answer value
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

### Lines 64-98: Define helper: run gsm8k benchmark
```python
def run_gsm8k_benchmark(base_url, num_questions=200, num_shots=5, parallel=64):
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
    return float(acc), float(latency)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 101-101: Define class TestGrok2EvalMI35x
```python
class TestGrok2EvalMI35x(unittest.TestCase):
```
**EN:** This declaration introduces the `TestGrok2EvalMI35x` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGrok2EvalMI35x` 测试类，并说明它通过继承承担的职责。

### Lines 102-102: Document the class `TestGrok2EvalMI35x`
```python
    """GROK2 GSM8K Completion Evaluation Test for MI35x."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestGrok2EvalMI35x`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestGrok2EvalMI35x`的设计意图。

### Lines 104-108: Initialize shared test fixture
```python
    @classmethod
    def setUpClass(cls):
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.num_questions = int(os.environ.get("GSM8K_NUM_QUESTIONS", "200"))
        cls.accuracy_threshold = 0.90
```
**EN:** This class-level setup prepares shared state and launches the resources needed by the suite.
**CN:** 该类级初始化会准备共享状态，并启动测试套件所需的资源。

### Lines 110-161: Run test: grok2 accuracy
```python
    def test_grok2_accuracy(self):
        """Test Grok-2 with GSM8K completion benchmark."""
        env = os.environ.copy()
        env["RCCL_MSCCL_ENABLE"] = "0"
        env["SGLANG_USE_AITER"] = "1"
        env["SGLANG_INT4_WEIGHT"] = "0"

        other_args = [
            "--tp",
            "8",
            "--quantization",
            "fp8",
            "--attention-backend",
            "aiter",
            "--mem-fraction-static",
            "0.85",
            "--tokenizer-path",
            GROK2_TOKENIZER_PATH,
            "--trust-remote-code",
        ]

        process = popen_launch_server(
            model=GROK2_MODEL_PATH,
            base_url=self.base_url,
            timeout=3600,
            other_args=other_args,
            env=env,
        )

        try:
            acc, latency = run_gsm8k_benchmark(
                self.base_url, num_questions=self.num_questions
            )
            passed = acc >= self.accuracy_threshold
            status = "✅ PASS" if passed else "❌ FAIL"
            print(f"  accuracy={acc:.3f} threshold={self.accuracy_threshold} {status}")

            summary = f"### GROK2 (MI35x)\n\n"
            summary += f"| Model | Accuracy | Threshold | Status |\n"
            summary += f"| ----- | -------- | --------- | ------ |\n"
            summary += f"| {GROK2_MODEL_PATH} | {acc:.3f} | {self.accuracy_threshold} | {status} |\n"

            if is_in_ci():
                write_github_step_summary(summary)

            self.assertGreaterEqual(
                acc,
                self.accuracy_threshold,
                f"Accuracy {acc:.3f} below threshold {self.accuracy_threshold}",
            )
        finally:
            kill_process_tree(process.pid)
```
**EN:** This test method exercises grok2 accuracy and verifies that the observed behavior matches the expected contract. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该测试方法会执行 grok2 accuracy 场景，并验证观测到的行为是否符合预期契约。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 164-165: Expose unittest entrypoint
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
- External and stdlib modules / 外部与标准库模块: `ast`, `numpy`, `os`, `re`, `time`, `unittest`
- Notable symbols / 关键符号: `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `unittest.main`
