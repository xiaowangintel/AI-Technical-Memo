# test_multi_tokenizer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/tokenizer/test_multi_tokenizer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates multi tokenizer behavior in SGLang's tokenizer area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 tokenizer 领域中与 multi tokenizer 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.kits.eval_accuracy_kit import MMLUMixin
from sglang.test.test_utils import (
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    auto_config_device,
    get_benchmark_args,
    is_in_amd_ci,
    is_in_ci,
    popen_launch_server,
    run_benchmark,
    write_github_step_summary,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`。

### Lines 20-21: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=211, stage="base-b", runner_config="1-gpu-large")
register_amd_ci(est_time=345, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 24-24: class TestMultiTokenizer declaration / 类 TestMultiTokenizer 声明
```python
class TestMultiTokenizer(CustomTestCase, MMLUMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`, `MMLUMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`, `MMLUMixin`。

### Lines 25-27: class-level constants and configuration for `TestMultiTokenizer` / 类级常量与配置
```python
    mmlu_score_threshold = 0.65
    mmlu_num_examples = 64
    mmlu_num_threads = 32
```
**EN:** This block defines shared names such as `mmlu_score_threshold`, `mmlu_num_examples`, `mmlu_num_threads`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `mmlu_score_threshold`, `mmlu_num_examples`, `mmlu_num_threads` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 29-43: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--tokenizer-worker-num",
                8,
                "--mem-fraction-static",
                0.7,
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 45-47: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 49-76: test case multi tokenizer ttft / 测试用例 multi tokenizer ttft
```python
    def test_multi_tokenizer_ttft(self):
        # from test_bench_serving.py run_bench_serving
        args = get_benchmark_args(
            base_url=self.base_url,
            dataset_name="random",
            dataset_path="",
            tokenizer=None,
            num_prompts=100,
            random_input_len=4096,
            random_output_len=2048,
            sharegpt_context_len=None,
            request_rate=1,
            disable_stream=False,
            disable_ignore_eos=False,
            seed=0,
            device=auto_config_device(),
            lora_name=None,
        )
        res = run_benchmark(args)
        if is_in_ci():
            write_github_step_summary(
                f"### test_multi_tokenizer_ttft\n"
                f"median_e2e_latency_ms: {res['median_e2e_latency_ms']:.2f} ms\n"
            )
            self.assertLess(res["median_e2e_latency_ms"], 11000)
            # relax for mi300x
            self.assertLess(res["median_ttft_ms"], 130 if is_in_amd_ci() else 86)
            self.assertLess(res["median_itl_ms"], 10)
```
**EN:** This test exercises `test_multi_tokenizer_ttft` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multi_tokenizer_ttft`。

### Lines 79-80: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestMultiTokenizer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMultiTokenizer.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestMultiTokenizer.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestMultiTokenizer.test_multi_tokenizer_ttft`: This test exercises `test_multi_tokenizer_ttft` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multi_tokenizer_ttft`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kits.eval_accuracy_kit`, `sglang.test.test_utils`

- **Total lines / 总行数**: 80
