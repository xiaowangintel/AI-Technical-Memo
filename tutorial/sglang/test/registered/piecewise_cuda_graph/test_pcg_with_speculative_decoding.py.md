# test_pcg_with_speculative_decoding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/piecewise_cuda_graph/test_pcg_with_speculative_decoding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates pcg with speculative decoding behavior in SGLang's piecewise cuda graph area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 piecewise cuda graph 领域中与 pcg with speculative decoding 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: supporting statements / 辅助语句
```python
"""Test piecewise CUDA graph coexisting with speculative decoding (EAGLE3).

PCG handles prefill/extend path while speculative decoding (EAGLE3) uses
decode CUDA graphs. This test verifies they don't interfere with each
other. MTP / STANDALONE / NGRAM variants moved to the sibling file
test_pcg_with_speculative_decoding_extra.py.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 9-12: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.server_fixtures.pcg_spec_fixture import PCGSpecBase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.test.ci.ci_register`, `sglang.test.server_fixtures.pcg_spec_fixture`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.test.ci.ci_register`, `sglang.test.server_fixtures.pcg_spec_fixture`。

### Lines 14-14: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=531, stage="base-b", runner_config="2-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 17-17: class TestPCGWithEAGLE3 declaration / 类 TestPCGWithEAGLE3 声明
```python
class TestPCGWithEAGLE3(PCGSpecBase, unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `PCGSpecBase`, `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `PCGSpecBase`, `unittest.TestCase`。

### Lines 18-41: class-level constants and configuration for `TestPCGWithEAGLE3` / 类级常量与配置
```python
    """PCG + EAGLE3 on Qwen3-30B-A3B-Instruct-2507."""

    model = "Qwen/Qwen3-30B-A3B-Instruct-2507"
    server_args = [
        "--tp",
        "2",
        "--trust-remote-code",
        "--enforce-piecewise-cuda-graph",
        "--mem-fraction-static",
        "0.6",
        "--speculative-algorithm",
        "EAGLE3",
        "--speculative-draft-model-path",
        "lmsys/SGLang-EAGLE3-Qwen3-30B-A3B-Instruct-2507-SpecForge-Nex",
        "--speculative-num-steps",
        "5",
        "--speculative-eagle-topk",
        "4",
        "--speculative-num-draft-tokens",
        "8",
    ]
    timeout_mult = 3
    server_env = {"SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN": "1"}
    accuracy_threshold = 0.75
```
**EN:** This block defines shared names such as `model`, `server_args`, `timeout_mult`, `server_env`, `accuracy_threshold`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `server_args`, `timeout_mult`, `server_env`, `accuracy_threshold` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 44-45: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestPCGWithEAGLE3`: PCG + EAGLE3 on Qwen3-30B-A3B-Instruct-2507. / 用于组织相关测试、夹具或辅助方法。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.server_fixtures.pcg_spec_fixture`

- **Total lines / 总行数**: 45
