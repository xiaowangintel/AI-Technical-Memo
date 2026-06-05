# test_pcg_with_speculative_decoding_extra.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/piecewise_cuda_graph/test_pcg_with_speculative_decoding_extra.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates pcg with speculative decoding extra behavior in SGLang's piecewise cuda graph area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 piecewise cuda graph 领域中与 pcg with speculative decoding extra 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: supporting statements / 辅助语句
```python
"""Extra: PCG coexistence with non-EAGLE3 speculative decoding variants.

EAGLE3 stays per-commit in the sibling file
test_pcg_with_speculative_decoding.py.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 7-10: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.server_fixtures.pcg_spec_fixture import PCGSpecBase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.test.ci.ci_register`, `sglang.test.server_fixtures.pcg_spec_fixture`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.test.ci.ci_register`, `sglang.test.server_fixtures.pcg_spec_fixture`。

### Lines 12-12: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=531, stage="extra-a", runner_config="2-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 15-15: class TestPCGWithMTP declaration / 类 TestPCGWithMTP 声明
```python
class TestPCGWithMTP(PCGSpecBase, unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `PCGSpecBase`, `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `PCGSpecBase`, `unittest.TestCase`。

### Lines 16-36: class-level constants and configuration for `TestPCGWithMTP` / 类级常量与配置
```python
    """PCG + MTP (NEXTN) on Qwen3.5-35B-A3B with FP8."""

    model = "Qwen/Qwen3.5-35B-A3B"
    server_args = [
        "--tp",
        "2",
        "--trust-remote-code",
        "--quantization",
        "fp8",
        "--mamba-scheduler-strategy",
        "extra_buffer",
        "--enable-piecewise-cuda-graph",
        "--speculative-algorithm",
        "NEXTN",
        "--reasoning-parser",
        "qwen3",
    ]
    timeout_mult = 3
    max_tokens = 8192
    thinking_mode = "qwen3"
    accuracy_threshold = 0.75
```
**EN:** This block defines shared names such as `model`, `server_args`, `timeout_mult`, `max_tokens`, `thinking_mode`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `server_args`, `timeout_mult`, `max_tokens`, `thinking_mode` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 39-39: class TestPCGWithSTANDALONE declaration / 类 TestPCGWithSTANDALONE 声明
```python
class TestPCGWithSTANDALONE(PCGSpecBase, unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `PCGSpecBase`, `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `PCGSpecBase`, `unittest.TestCase`。

### Lines 40-59: class-level constants and configuration for `TestPCGWithSTANDALONE` / 类级常量与配置
```python
    """PCG + STANDALONE on Llama-3.1-8B-Instruct + Llama-3.2-1B-Instruct."""

    model = "meta-llama/Llama-3.1-8B-Instruct"
    server_args = [
        "--trust-remote-code",
        "--enforce-piecewise-cuda-graph",
        "--mem-fraction-static",
        "0.5",
        "--speculative-algorithm",
        "STANDALONE",
        "--speculative-draft-model-path",
        "meta-llama/Llama-3.2-1B-Instruct",
        "--speculative-num-steps",
        "3",
        "--speculative-eagle-topk",
        "1",
        "--speculative-num-draft-tokens",
        "4",
    ]
    accuracy_threshold = 0.50
```
**EN:** This block defines shared names such as `model`, `server_args`, `accuracy_threshold`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `server_args`, `accuracy_threshold` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 62-62: class TestPCGWithNGRAM declaration / 类 TestPCGWithNGRAM 声明
```python
class TestPCGWithNGRAM(PCGSpecBase, unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `PCGSpecBase`, `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `PCGSpecBase`, `unittest.TestCase`。

### Lines 63-77: class-level constants and configuration for `TestPCGWithNGRAM` / 类级常量与配置
```python
    """PCG + NGRAM on Qwen2.5-Coder-7B-Instruct."""

    model = "Qwen/Qwen2.5-Coder-7B-Instruct"
    server_args = [
        "--trust-remote-code",
        "--enforce-piecewise-cuda-graph",
        "--speculative-algorithm",
        "NGRAM",
        "--speculative-num-draft-tokens",
        "16",
        "--cuda-graph-max-bs",
        "8",
        "--mem-fraction-static",
        "0.8",
    ]
```
**EN:** This block defines shared names such as `model`, `server_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `server_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 80-81: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestPCGWithMTP`: PCG + MTP (NEXTN) on Qwen3.5-35B-A3B with FP8. / 用于组织相关测试、夹具或辅助方法。
- `TestPCGWithSTANDALONE`: PCG + STANDALONE on Llama-3.1-8B-Instruct + Llama-3.2-1B-Instruct. / 用于组织相关测试、夹具或辅助方法。
- `TestPCGWithNGRAM`: PCG + NGRAM on Qwen2.5-Coder-7B-Instruct. / 用于组织相关测试、夹具或辅助方法。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.server_fixtures.pcg_spec_fixture`

- **Total lines / 总行数**: 81
