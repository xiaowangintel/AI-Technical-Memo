# test_nvidia_nemotron_nano_v2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/models/test_nvidia_nemotron_nano_v2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `nvidia nemotron nano v2` scenario in `test/manual/models`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/models` 中的 `nvidia nemotron nano v2` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports and shared helpers / 导入与共享辅助项
```python
import unittest

from sglang.srt.utils import is_blackwell
from sglang.test.kits.eval_accuracy_kit import GSM8KMixin
from sglang.test.server_fixtures.default_fixture import DefaultServerBase
```
**EN:** This range imports `unittest`, `sglang.srt.utils`, `sglang.test.kits.eval_accuracy_kit` and `sglang.test.server_fixtures.default_fixture`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 8-11: Class definition for TestNvidiaNemotronNanoV2BF16 / 类定义
```python
class TestNvidiaNemotronNanoV2BF16(GSM8KMixin, DefaultServerBase):
    model = "nvidia/NVIDIA-Nemotron-Nano-9B-v2"
    gsm8k_accuracy_thres = 0.87
    other_args = ["--max-mamba-cache-size", "256"]
```
**EN:** This range declares `TestNvidiaNemotronNanoV2BF16`, which organizes the scenario as a reusable test-oriented class. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 14-17: Class definition for TestNvidiaNemotronNanoV2BF16PP / 类定义
```python
class TestNvidiaNemotronNanoV2BF16PP(GSM8KMixin, DefaultServerBase):
    model = "nvidia/NVIDIA-Nemotron-Nano-9B-v2"
    gsm8k_accuracy_thres = 0.87
    other_args = ["--max-mamba-cache-size", "256", "--pp-size", "2"]
```
**EN:** This range declares `TestNvidiaNemotronNanoV2BF16PP`, which organizes the scenario as a reusable test-oriented class. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 20-23: Class definition for TestNvidiaNemotronNanoV2FP8 / 类定义
```python
class TestNvidiaNemotronNanoV2FP8(GSM8KMixin, DefaultServerBase):
    gsm8k_accuracy_thres = 0.87
    model = "nvidia/NVIDIA-Nemotron-Nano-9B-v2-FP8"
    other_args = ["--max-mamba-cache-size", "256"]
```
**EN:** This range declares `TestNvidiaNemotronNanoV2FP8`, which organizes the scenario as a reusable test-oriented class. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 24-26: Scenario logic / 场景逻辑
```python


@unittest.skipIf(not is_blackwell(), "NVFP4 only supported on blackwell")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skipIf` and `is_blackwell`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 27-30: Class definition for TestNvidiaNemotronNanoV2NVFP4 / 类定义
```python
class TestNvidiaNemotronNanoV2NVFP4(GSM8KMixin, DefaultServerBase):
    gsm8k_accuracy_thres = 0.855
    model = "nvidia/NVIDIA-Nemotron-Nano-9B-v2-NVFP4"
    other_args = ["--max-mamba-cache-size", "256"]
```
**EN:** This range declares `TestNvidiaNemotronNanoV2NVFP4`, which organizes the scenario as a reusable test-oriented class. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 31-36: Scenario logic / 场景逻辑
```python


@unittest.skip(
    "STANDALONE speculative decoding does not yet support target and draft models "
    "with different hidden sizes (Nemotron-9B: 4480, Llama-3.2-1B: 2048)"
)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skip` and `sizes`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 37-59: Class definition for TestNvidiaNemotronNanoV2SpeculativeDecoding / 类定义
```python
class TestNvidiaNemotronNanoV2SpeculativeDecoding(GSM8KMixin, DefaultServerBase):
    gsm8k_accuracy_thres = 0.87
    model = "nvidia/NVIDIA-Nemotron-Nano-9B-v2"
    other_args = [
        "--speculative-algorithm",
        "STANDALONE",
        "--speculative-num-steps",
        "2",
        "--speculative-eagle-topk",
        "3",
        "--speculative-num-draft-tokens",
        "5",
        "--speculative-draft-model-path",
        "meta-llama/Llama-3.2-1B",
        "--speculative-draft-load-format",
        "dummy",
        "--max-running-requests",
        "8",
        "--max-total-tokens",
        "2048",
        "--json-model-override-args",
        '{"vocab_size": 131072}',
    ]
```
**EN:** This range declares `TestNvidiaNemotronNanoV2SpeculativeDecoding`, which organizes the scenario as a reusable test-oriented class. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 60-65: Scenario logic / 场景逻辑
```python


@unittest.skip(
    "STANDALONE speculative decoding does not yet support target and draft models "
    "with different hidden sizes (Nemotron-9B: 4480, Llama-3.2-1B: 2048)"
)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `skip` and `sizes`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 66-90: Class definition for TestNvidiaNemotronNanoV2SpeculativeDecodingBF16Cache / 类定义
```python
class TestNvidiaNemotronNanoV2SpeculativeDecodingBF16Cache(
    GSM8KMixin, DefaultServerBase
):
    gsm8k_accuracy_thres = 0.87
    model = "nvidia/NVIDIA-Nemotron-Nano-9B-v2"
    other_args = [
        "--speculative-algorithm",
        "STANDALONE",
        "--speculative-num-steps",
        "2",
        "--speculative-eagle-topk",
        "3",
        "--speculative-num-draft-tokens",
        "5",
        "--speculative-draft-model-path",
        "meta-llama/Llama-3.2-1B",
        "--speculative-draft-load-format",
        "dummy",
        "--max-running-requests",
        "8",
        "--max-total-tokens",
        "2048",
        "--json-model-override-args",
        '{"vocab_size": 131072}',
        "--mamba-ssm-dtype",
```
**EN:** This range declares `TestNvidiaNemotronNanoV2SpeculativeDecodingBF16Cache`, which organizes the scenario as a reusable test-oriented class. The code also ties this block to evaluation or benchmark execution.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。 这段代码还把当前逻辑与评测或基准执行联系起来。

### Lines 91-92: Scenario logic / 场景逻辑
```python
        "bfloat16",
    ]
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 93-96: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Quantization configuration / 量化配置
- Token-level inspection / Token 级分析
- Prefix caching / 前缀缓存
- Structured outputs / 结构化输出

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.kits.eval_accuracy_kit`, `sglang.test.server_fixtures.default_fixture`
