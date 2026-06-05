# test_unified_radix_cache_kl_hicache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/radix_cache/test_unified_radix_cache_kl_hicache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates unified radix cache kl hicache behavior in SGLang's radix cache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 Radix 缓存 领域中与 unified radix cache kl hicache 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: module imports and dependencies / 模块导入与依赖
```python
import unittest

from test_unified_radix_cache_kl import UnifiedRadixTreeTestMixin

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.kl_multiturn_utils import (
    get_input_ids,
    make_mamba_decode_assert,
    make_mamba_prefill_assert,
)
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `test_unified_radix_cache_kl`, `sglang.srt.utils`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `test_unified_radix_cache_kl`, `sglang.srt.utils`, `sglang.test.ci.ci_register`。

### Lines 20-27: CI registration and metadata / CI 注册与元数据
```python
MAMBA_MODEL = "Qwen/Qwen3-Next-80B-A3B-Instruct"
MAMBA_CHUNK_SIZE = 64
MAMBA_TRACK_INTERVAL = 128

DSV4_FLASH_MODEL = "sgl-project/DeepSeek-V4-Flash-FP8"
DSV4_FLASH_LAUNCH_TIMEOUT = 3600

register_cuda_ci(est_time=768, stage="base-c", runner_config="8-gpu-h200")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 30-30: class TestUnifiedMambaHiCache declaration / 类 TestUnifiedMambaHiCache 声明
```python
class TestUnifiedMambaHiCache(UnifiedRadixTreeTestMixin, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `UnifiedRadixTreeTestMixin`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `UnifiedRadixTreeTestMixin`, `CustomTestCase`。

### Lines 31-39: class-level constants and configuration for `TestUnifiedMambaHiCache` / 类级常量与配置
```python
    """Mamba hybrid + HiCache + UnifiedRadixCache."""

    kl_threshold = 0.005
    prefill_cache_assert = staticmethod(
        make_mamba_prefill_assert(chunk_size=MAMBA_CHUNK_SIZE)
    )
    decode_cache_assert = staticmethod(
        make_mamba_decode_assert(track_interval=MAMBA_TRACK_INTERVAL)
    )
```
**EN:** This block defines shared names such as `kl_threshold`, `prefill_cache_assert`, `decode_cache_assert`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `kl_threshold`, `prefill_cache_assert`, `decode_cache_assert` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 41-78: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = MAMBA_MODEL
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--tp-size",
                "4",
                "--chunked-prefill-size",
                "2048",
                "--mem-fraction-static",
                "0.85",
                "--mamba-scheduler-strategy",
                "extra_buffer",
                "--mamba-track-interval",
                str(MAMBA_TRACK_INTERVAL),
                "--enable-hierarchical-cache",
                "--hicache-ratio",
                "4",
                "--hicache-write-policy",
                "write_through",
                "--hicache-io-backend",
                "direct",
                "--hicache-mem-layout",
                "page_first_direct",
                "--max-total-tokens",
                "12000",
                "--max-mamba-cache-size",
                "500",
                "--max-running-requests",
                "4",
            ],
            env={"SGLANG_ENABLE_UNIFIED_RADIX_TREE": "1"},
        )
        cls.input_ids = get_input_ids(cls.model, num_samples=18)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 80-82: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 85-89: function assert dsv4 decode cached tokens / 函数 assert dsv4 decode cached tokens
```python
def _assert_dsv4_decode_cached_tokens(result, history_len, output_len, label):
    expected = history_len + output_len
    actual = result["meta_info"]["cached_tokens"]
    lower = max(0, expected - 256)
    assert actual >= lower, f"{label}: expected cached_tokens>={lower}, got {actual}"
```
**EN:** This block implements `_assert_dsv4_decode_cached_tokens` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_assert_dsv4_decode_cached_tokens`，承担模块行为中的一个聚焦逻辑片段。

### Lines 92-92: class TestUnifiedDeepSeekV4FlashHiCache declaration / 类 TestUnifiedDeepSeekV4FlashHiCache 声明
```python
class TestUnifiedDeepSeekV4FlashHiCache(UnifiedRadixTreeTestMixin, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `UnifiedRadixTreeTestMixin`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `UnifiedRadixTreeTestMixin`, `CustomTestCase`。

### Lines 93-99: class-level constants and configuration for `TestUnifiedDeepSeekV4FlashHiCache` / 类级常量与配置
```python
    """DeepSeek V4 Flash FP8 + HiCache + UnifiedRadixCache."""

    kl_threshold = 0.005
    sampling_temperature = 0
    decode_cache_assert = staticmethod(_assert_dsv4_decode_cached_tokens)
    gsm8k_threshold = 0.90
    num_gsm8k_questions = 100
```
**EN:** This block defines shared names such as `kl_threshold`, `sampling_temperature`, `decode_cache_assert`, `gsm8k_threshold`, `num_gsm8k_questions`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `kl_threshold`, `sampling_temperature`, `decode_cache_assert`, `gsm8k_threshold`, `num_gsm8k_questions` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 101-103: test case multiturn logprobs match / 测试用例 multiturn logprobs match
```python
    @unittest.skipIf(is_in_ci(), "To reduce the CI execution time.")
    def test_multiturn_logprobs_match(self):
        pass
```
**EN:** This test exercises `test_multiturn_logprobs_match` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiturn_logprobs_match`。

### Lines 105-147: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DSV4_FLASH_MODEL
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DSV4_FLASH_LAUNCH_TIMEOUT,
            other_args=[
                "--trust-remote-code",
                "--tp-size",
                "4",
                "--attention-backend",
                "compressed",
                "--page-size",
                "256",
                "--chunked-prefill-size",
                "8192",
                "--mem-fraction-static",
                "0.9",
                "--disable-shared-experts-fusion",
                "--enable-hierarchical-cache",
                "--hicache-ratio",
                "4",
                "--hicache-write-policy",
                "write_through",
                "--hicache-io-backend",
                "direct",
                "--hicache-mem-layout",
                "page_first_direct",
                "--swa-full-tokens-ratio",
                "0.25",
                "--max-total-tokens",
                "20000",
                "--max-running-requests",
                "2",
            ],
            env={
                "SGLANG_DSV4_FP4_EXPERTS": "0",
                "SGLANG_ENABLE_UNIFIED_RADIX_TREE": "1",
            },
        )
        cls.input_ids = get_input_ids(cls.model, num_samples=18)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 149-151: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 154-155: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestUnifiedMambaHiCache`: Mamba hybrid + HiCache + UnifiedRadixCache. / 用于组织相关测试、夹具或辅助方法。
- `_assert_dsv4_decode_cached_tokens`: This block implements `_assert_dsv4_decode_cached_tokens` and captures one focused piece of the module's behavior. / 该代码块实现 `_assert_dsv4_decode_cached_tokens`，承担模块行为中的一个聚焦逻辑片段。
- `TestUnifiedDeepSeekV4FlashHiCache`: DeepSeek V4 Flash FP8 + HiCache + UnifiedRadixCache. / 用于组织相关测试、夹具或辅助方法。
- `TestUnifiedMambaHiCache.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestUnifiedMambaHiCache.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestUnifiedDeepSeekV4FlashHiCache.test_multiturn_logprobs_match`: This test exercises `test_multiturn_logprobs_match` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiturn_logprobs_match`。
- `TestUnifiedDeepSeekV4FlashHiCache.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestUnifiedDeepSeekV4FlashHiCache.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `test_unified_radix_cache_kl`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kl_multiturn_utils`, `sglang.test.test_utils`

- **Total lines / 总行数**: 155
