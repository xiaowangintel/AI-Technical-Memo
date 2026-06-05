# test_npu_hierarchical_cache_ttft_mha.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/ascend/basic_function/HiCache/test_npu_hierarchical_cache_ttft_mha.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on Hi Cache npu hierarchical cache ttft mha in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 Hi Cache npu hierarchical cache ttft mha 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Import dependencies
```python
import unittest

from sglang.test.ascend.test_ascend_utils import (
    QWEN3_32B_WEIGHTS_PATH,
    run_bench_serving,
)
from sglang.test.ci.ci_register import register_npu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 10-15: Register CI metadata
```python
register_npu_ci(
    est_time=400,
    suite="nightly-2-npu-a3",
    nightly=True,
    disabled="run failed",
)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 18-18: Define class TestNpuHierarchicalCacheTTFT
```python
class TestNpuHierarchicalCacheTTFT(CustomTestCase):
```
**EN:** This declaration introduces the `TestNpuHierarchicalCacheTTFT` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNpuHierarchicalCacheTTFT` 测试类，并说明它通过继承承担的职责。

### Lines 19-23: Document the class `TestNpuHierarchicalCacheTTFT`
```python
    """The test used the Qwen3-32B model, with hierarchical cache enabled, and TTFT improved by 40%.

    [Test Category] HiCache
    [Test Target] --enable-hierarchical-cache
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNpuHierarchicalCacheTTFT`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNpuHierarchicalCacheTTFT`的设计意图。

### Lines 25-83: Run test: no chunked prefill without radix cache (part 1)
```python
    def test_no_chunked_prefill_without_radix_cache(self):
        TTFTS = []
        model = QWEN3_32B_WEIGHTS_PATH
        common_args = [
            [
                "--trust-remote-code",
                "--tp-size",
                2,
                "--mem-fraction-static",
                0.8,
                "--max-running-requests",
                16,
                "--disable-radix-cache",
                "--chunked-prefill-size",
                "-1",
                "--disable-cuda-graph",
            ],
            [
                "--trust-remote-code",
                "--tp-size",
                2,
                "--mem-fraction-static",
                0.8,
                "--max-running-requests",
                16,
                "--chunked-prefill-size",
                "-1",
                "--disable-cuda-graph",
                "--enable-hierarchical-cache",
                "--hicache-ratio",
                5,
                "--hicache-write-policy",
                "write_back",
            ],
        ]
        for common_arg in common_args:
            other_args = common_arg + (
                [
                    "--attention-backend",
                    "ascend",
                ]
            )
            res = run_bench_serving(
                model=model,
                dataset_name="generated-shared-prefix",
                num_prompts=128,
                random_input_len=3584,
                random_output_len=1,
                request_rate=float("inf"),
                max_concurrency=16,
                gsp_num_groups=1,
                gsp_prompts_per_group=128,
                gsp_system_prompt_len=1792,
                gsp_question_len=1792,
                gsp_output_len=1,
                other_server_args=other_args,
            )
            TTFT = res["mean_ttft_ms"]
            TTFTS.append(TTFT)
```
**EN:** This test method exercises no chunked prefill without radix cache and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 no chunked prefill without radix cache 场景，并验证观测到的行为是否符合预期契约。

### Lines 85-85: Run test: no chunked prefill without radix cache (part 2)
```python
        assert float(TTFTS[1]) <= 0.6 * float(TTFTS[0])
```
**EN:** This test method exercises no chunked prefill without radix cache and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no chunked prefill without radix cache 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 88-89: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.test.ascend.test_ascend_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `unittest`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
