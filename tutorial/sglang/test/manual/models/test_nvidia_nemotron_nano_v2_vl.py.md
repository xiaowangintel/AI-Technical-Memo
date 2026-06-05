# test_nvidia_nemotron_nano_v2_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/models/test_nvidia_nemotron_nano_v2_vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `nvidia nemotron nano v2 vl` scenario in `test/manual/models`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/models` 中的 `nvidia nemotron nano v2 vl` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Imports and shared helpers / 导入与共享辅助项
```python
import unittest

from sglang.test.kits.eval_accuracy_kit import GSM8KMixin
from sglang.test.kits.mmmu_vlm_kit import MMMUMixin
from sglang.test.server_fixtures.default_fixture import DefaultServerBase
from sglang.test.server_fixtures.mmmu_fixture import MMMUServerBase

# NVIDIA Nemotron Nano V2 VL model tests (CUDA only)
# GSM8k + MMMU evaluation


MODEL = "nvidia/NVIDIA-Nemotron-Nano-12B-v2-VL-BF16"
```
**EN:** This range imports `unittest`, `sglang.test.kits.eval_accuracy_kit`, `sglang.test.kits.mmmu_vlm_kit` and `sglang.test.server_fixtures.default_fixture`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. The code also ties this block to evaluation or benchmark execution. Representative call sites include `tests`.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 15-18: Class definition for TestNvidiaNemotronNanoV2VLTextOnly / 类定义
```python
class TestNvidiaNemotronNanoV2VLTextOnly(GSM8KMixin, DefaultServerBase):
    gsm8k_accuracy_thres = 0.85
    model = MODEL
    other_args = ["--max-mamba-cache-size", "256", "--trust-remote-code"]
```
**EN:** This range declares `TestNvidiaNemotronNanoV2VLTextOnly`, which organizes the scenario as a reusable test-oriented class. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 21-26: Class definition for TestNvidiaNemotronNanoV2VLMMMU / 类定义
```python
class TestNvidiaNemotronNanoV2VLMMMU(MMMUMixin, MMMUServerBase):
    accuracy = 0.444
    model = MODEL
    other_args = ["--max-mamba-cache-size", "128", "--trust-remote-code"]
    mmmu_args = ["--limit=0.1"]
    """`--limit=0.1`: 10 percent of each task - this is fine for testing since the nominal result isn't interesting - this run is just to prevent relative regressions."""
```
**EN:** This range declares `TestNvidiaNemotronNanoV2VLMMMU`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 27-30: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Multi-GPU orchestration / 多 GPU 编排
- Prefix caching / 前缀缓存

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.test.kits.eval_accuracy_kit`, `sglang.test.kits.mmmu_vlm_kit`, `sglang.test.server_fixtures.default_fixture`, `sglang.test.server_fixtures.mmmu_fixture`
