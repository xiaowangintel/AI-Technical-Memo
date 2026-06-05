# test_spec_ngram_extra.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/spec/test_spec_ngram_extra.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates spec ngram extra behavior in SGLang's spec area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 推测解码 领域中与 spec ngram extra 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: module imports and dependencies / 模块导入与依赖
```python
import unittest

import requests

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.kits.eval_accuracy_kit import GSM8KMixin
from sglang.test.server_fixtures.ngram_fixture import NgramServerBase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `requests`, `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `requests`, `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`。

### Lines 8-10: supporting source context / 辅助源码上下文
```python

# Extra: Triton + Flashinfer NGRAM backends. Sibling per-commit file
# (test_spec_ngram.py) keeps the Paged variant.
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 11-11: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=254, stage="extra-a", runner_config="1-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 14-14: class TestNgramSpeculativeDecodingTriton declaration / 类 TestNgramSpeculativeDecodingTriton 声明
```python
class TestNgramSpeculativeDecodingTriton(NgramServerBase, GSM8KMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `NgramServerBase`, `GSM8KMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `NgramServerBase`, `GSM8KMixin`。

### Lines 15-15: class-level constants and configuration for `TestNgramSpeculativeDecodingTriton` / 类级常量与配置
```python
    attention_backend = "triton"
```
**EN:** This block defines shared names such as `attention_backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `attention_backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 18-18: class TestNgramSpeculativeDecodingFlashinfer declaration / 类 TestNgramSpeculativeDecodingFlashinfer 声明
```python
class TestNgramSpeculativeDecodingFlashinfer(NgramServerBase, GSM8KMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `NgramServerBase`, `GSM8KMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `NgramServerBase`, `GSM8KMixin`。

### Lines 19-20: class-level constants and configuration for `TestNgramSpeculativeDecodingFlashinfer` / 类级常量与配置
```python
    attention_backend = "flashinfer"
    extra_args = ["--speculative-ngram-external-sam-budget", "8"]
```
**EN:** This block defines shared names such as `attention_backend`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `attention_backend`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 22-86: test case output as corpus boosts accept length / 测试用例 output as corpus boosts accept length
```python
    def test_output_as_corpus_boosts_accept_length(self):
        """Baseline → HTTP add corpus → verify accept length boost."""
        prompts = [
            "The capital of France is",
            "In mathematics, the Pythagorean theorem states that",
            "The speed of light in a vacuum is approximately",
            "Water boils at a temperature of",
            "The largest planet in our solar system is",
        ]
        max_new_tokens = 128
        num_rounds = 3

        def generate_batch():
            outputs = []
            for prompt in prompts:
                resp = requests.post(
                    self.base_url + "/generate",
                    json={
                        "text": prompt,
                        "sampling_params": {
                            "temperature": 0,
                            "max_new_tokens": max_new_tokens,
                        },
                    },
                    timeout=120,
                )
                self.assertEqual(resp.status_code, 200, resp.text)
                outputs.append(resp.json()["text"])
            return outputs

        def get_accept_length():
            info = requests.get(self.base_url + "/server_info").json()
            return info["internal_states"][0]["avg_spec_accept_length"]

        # Phase 1: baseline — no SAM corpus loaded, only trie
        generated_outputs = []
        for _ in range(num_rounds):
            generated_outputs = generate_batch()
        baseline_accept_len = get_accept_length()
        print(f"\n  Baseline accept length (no SAM): {baseline_accept_len:.2f}")

        # Flush cache so phase 2 starts clean
        requests.post(self.base_url + "/flush_cache", timeout=30)

        # Phase 2: add generated outputs as corpus via HTTP API
        resp = requests.post(
            self.base_url + "/add_external_corpus",
            json={"corpus_id": "bench", "documents": generated_outputs},
            timeout=120,
        )
        self.assertEqual(resp.status_code, 200, resp.text)
        self.assertTrue(resp.json()["success"], resp.json().get("message"))

        for _ in range(num_rounds):
            generate_batch()
        sam_accept_len = get_accept_length()
        print(f"  SAM accept length (output as corpus): {sam_accept_len:.2f}")
        print(f"  Speedup: {sam_accept_len / baseline_accept_len:.2f}x")

        self.assertGreater(
            sam_accept_len,
            baseline_accept_len * 2.0,
            f"SAM accept length ({sam_accept_len:.2f}) should be at least 2x "
            f"baseline ({baseline_accept_len:.2f}) when corpus matches output",
        )
```
**EN:** Baseline → HTTP add corpus → verify accept length boost. This test exercises `test_output_as_corpus_boosts_accept_length` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Baseline → HTTP add corpus → verify accept length boost. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_output_as_corpus_boosts_accept_length`。

### Lines 89-90: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestNgramSpeculativeDecodingTriton`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNgramSpeculativeDecodingFlashinfer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestNgramSpeculativeDecodingFlashinfer.test_output_as_corpus_boosts_accept_length`: Baseline → HTTP add corpus → verify accept length boost. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_output_as_corpus_boosts_accept_length`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`, `sglang.test.server_fixtures.ngram_fixture`

- **Total lines / 总行数**: 90
