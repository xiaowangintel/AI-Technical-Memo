# test_unified_radix_cache_kl_hicache_nightly.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/radix_cache/test_unified_radix_cache_kl_hicache_nightly.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates unified radix cache kl hicache nightly behavior in SGLang's radix cache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 Radix 缓存 领域中与 unified radix cache kl hicache nightly 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: supporting statements / 辅助语句
```python
"""UnifiedRadixTree + HiCache KL divergence tests.

Tests Mamba hybrid, DeepSeek V4 Flash, and GLM-5 models with HiCache L2
offloading under UnifiedRadixTree, verifying multi-turn cache correctness
via KL divergence.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 8-23: module imports and dependencies / 模块导入与依赖
```python
import os
import shutil
import tempfile
import unittest
from types import SimpleNamespace
from urllib.parse import urlparse

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `os`, `shutil`, `tempfile`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`, `shutil`, `tempfile`, `unittest`。

### Lines 25-28: CI registration and metadata / CI 注册与元数据
```python
GLM5_MODEL = "zai-org/GLM-5.1-FP8"
GLM5_LAUNCH_TIMEOUT = 3600

register_cuda_ci(est_time=900, suite="nightly-8-gpu-h200", nightly=True)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 31-31: class GSM8KTwoPassMixin declaration / 类 GSM8KTwoPassMixin 声明
```python
class GSM8KTwoPassMixin:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 32-42: class-level constants and configuration for `GSM8KTwoPassMixin` / 类级常量与配置
```python
    """Mixin: run GSM8K twice with flush in between, verify accuracy diff.

    Subclass must provide:
      - self.base_url
      - self.model (for logging)
    """

    gsm8k_threshold: float = 0.90
    num_gsm8k_questions: int = 200
    max_accuracy_diff: float = 0.02
    gsm8k_parallel: int = 40
```
**EN:** This block defines shared names such as `gsm8k_threshold`, `num_gsm8k_questions`, `max_accuracy_diff`, `gsm8k_parallel`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `gsm8k_threshold`, `num_gsm8k_questions`, `max_accuracy_diff`, `gsm8k_parallel` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 44-58: method run gsm8k / 方法 run gsm8k
```python
    def _run_gsm8k(self):
        from sglang.test.few_shot_gsm8k import run_eval as run_few_shot_gsm8k

        url = urlparse(self.base_url)
        args = SimpleNamespace(
            num_shots=10,
            data_path=None,
            num_questions=self.num_gsm8k_questions,
            max_new_tokens=16000,
            parallel=self.gsm8k_parallel,
            host=f"http://{url.hostname}",
            port=int(url.port),
        )
        metrics = run_few_shot_gsm8k(args)
        return metrics["accuracy"]
```
**EN:** This block implements `_run_gsm8k` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_gsm8k`，承担模块行为中的一个聚焦逻辑片段。

### Lines 60-66: method flush cache / 方法 flush cache
```python
    def _flush_cache(self):
        response = requests.post(
            self.base_url + "/flush_cache",
            params={"timeout": 30},
            timeout=40,
        )
        response.raise_for_status()
```
**EN:** This block implements `_flush_cache` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_flush_cache`，承担模块行为中的一个聚焦逻辑片段。

### Lines 68-103: test case gsm8k two passes / 测试用例 gsm8k two passes
```python
    def test_gsm8k_two_passes(self):
        """Run GSM8K twice with flush in between, verify accuracy diff <= max_accuracy_diff."""
        # First pass
        acc1 = self._run_gsm8k()
        print(f"[{self.__class__.__name__}] GSM8K pass 1 accuracy: {acc1:.3f}")
        self.assertGreaterEqual(
            acc1,
            self.gsm8k_threshold,
            f"Pass 1 accuracy {acc1:.3f} < threshold {self.gsm8k_threshold}",
        )

        # Flush cache
        self._flush_cache()

        # Second pass
        acc2 = self._run_gsm8k()
        print(f"[{self.__class__.__name__}] GSM8K pass 2 accuracy: {acc2:.3f}")
        self.assertGreaterEqual(
            acc2,
            self.gsm8k_threshold,
            f"Pass 2 accuracy {acc2:.3f} < threshold {self.gsm8k_threshold}",
        )

        # Verify diff
        if acc1 > acc2:
            diff = abs(acc1 - acc2)
            print(
                f"[{self.__class__.__name__}] Accuracy diff: {diff:.3f} "
                f"(max allowed: {self.max_accuracy_diff})"
            )
            self.assertLessEqual(
                diff,
                self.max_accuracy_diff,
                f"Accuracy diff {diff:.3f} exceeds max {self.max_accuracy_diff} "
                f"(pass1={acc1:.3f}, pass2={acc2:.3f})",
            )
```
**EN:** Run GSM8K twice with flush in between, verify accuracy diff <= max_accuracy_diff. This test exercises `test_gsm8k_two_passes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Run GSM8K twice with flush in between, verify accuracy diff <= max_accuracy_diff. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k_two_passes`。

### Lines 106-106: class TestGLM5HiCacheL3GSM8K declaration / 类 TestGLM5HiCacheL3GSM8K 声明
```python
class TestGLM5HiCacheL3GSM8K(GSM8KTwoPassMixin, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `GSM8KTwoPassMixin`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `GSM8KTwoPassMixin`, `CustomTestCase`。

### Lines 107-107: supporting statements / 辅助语句
```python
    """GLM-5.1-FP8 + HiCache L3 (file backend), with HiRadixTree."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 109-143: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = GLM5_MODEL
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.hicache_dir = tempfile.mkdtemp(prefix="hicache_l3_")
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=GLM5_LAUNCH_TIMEOUT,
            other_args=[
                "--trust-remote-code",
                "--tp-size",
                "8",
                "--page-size",
                "64",
                "--mem-fraction-static",
                "0.85",
                "--model-loader-extra-config",
                '{"enable_multithread_load": true, "num_threads": 64}',
                "--enable-hierarchical-cache",
                "--hicache-ratio",
                "2",
                "--hicache-write-policy",
                "write_through",
                "--hicache-storage-prefetch-policy",
                "wait_complete",
                "--hicache-io-backend",
                "direct",
                "--hicache-mem-layout",
                "page_first_direct",
                "--hicache-storage-backend",
                "file",
            ],
            env={"SGLANG_HICACHE_FILE_BACKEND_STORAGE_DIR": cls.hicache_dir},
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 145-149: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
        if os.path.isdir(cls.hicache_dir):
            shutil.rmtree(cls.hicache_dir, ignore_errors=True)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 152-153: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `GSM8KTwoPassMixin`: Mixin: run GSM8K twice with flush in between, verify accuracy diff. / 用于组织相关测试、夹具或辅助方法。
- `TestGLM5HiCacheL3GSM8K`: GLM-5.1-FP8 + HiCache L3 (file backend), with HiRadixTree. / 用于组织相关测试、夹具或辅助方法。
- `GSM8KTwoPassMixin._run_gsm8k`: This block implements `_run_gsm8k` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_gsm8k`，承担模块行为中的一个聚焦逻辑片段。
- `GSM8KTwoPassMixin._flush_cache`: This block implements `_flush_cache` and captures one focused piece of the module's behavior. / 该代码块实现 `_flush_cache`，承担模块行为中的一个聚焦逻辑片段。
- `GSM8KTwoPassMixin.test_gsm8k_two_passes`: Run GSM8K twice with flush in between, verify accuracy diff <= max_accuracy_diff. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k_two_passes`。
- `TestGLM5HiCacheL3GSM8K.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestGLM5HiCacheL3GSM8K.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `shutil`, `tempfile`, `unittest`, `types`, `urllib.parse`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 153
