# test_hicache_variants.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/hicache/test_hicache_variants.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates hicache variants behavior in SGLang's hicache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 hicache 领域中与 hicache variants 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 3-8: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=450, stage="base-b", runner_config="1-gpu-large")
register_amd_ci(est_time=524, suite="stage-b-test-1-gpu-small-amd")
"""
Consolidated HiCache variant tests.
Tests HiCache with different configurations: standard, MLA, EAGLE, and page size variants.
"""
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 10-24: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.benchmark.utils import get_tokenizer
from sglang.srt.utils import is_hip, kill_process_tree
from sglang.test.kits.eval_accuracy_kit import MGSMEnMixin, MMLUMixin
from sglang.test.test_utils import (
    DEFAULT_DRAFT_MODEL_EAGLE3,
    DEFAULT_MLA_MODEL_NAME_FOR_TEST,
    DEFAULT_MODEL_NAME_FOR_TEST,
    DEFAULT_TARGET_MODEL_EAGLE3,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.benchmark.utils`, `sglang.srt.utils`, `sglang.test.kits.eval_accuracy_kit`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.benchmark.utils`, `sglang.srt.utils`, `sglang.test.kits.eval_accuracy_kit`。

### Lines 26-26: module-level constants and configuration / 模块级常量与配置
```python
_is_hip = is_hip()
```
**EN:** This block defines shared names such as `_is_hip`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `_is_hip` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 29-29: class HiCacheBaseServer declaration / 类 HiCacheBaseServer 声明
```python
class HiCacheBaseServer(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 30-33: class-level constants and configuration for `HiCacheBaseServer` / 类级常量与配置
```python
    """Base class for HiCache tests with configurable server setup"""

    model_name = DEFAULT_MODEL_NAME_FOR_TEST
    hicache_args = []
```
**EN:** This block defines shared names such as `model_name`, `hicache_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model_name`, `hicache_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 35-49: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = cls.model_name
        cls.base_url = DEFAULT_URL_FOR_TEST

        # Setup tokenizer if needed by subclass
        if hasattr(cls, "needs_tokenizer") and cls.needs_tokenizer:
            cls.tokenizer = get_tokenizer(cls.model)

        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=cls.hicache_args,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 51-53: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 56-56: class TestHiCacheStandard declaration / 类 TestHiCacheStandard 声明
```python
class TestHiCacheStandard(HiCacheBaseServer, MMLUMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `HiCacheBaseServer`, `MMLUMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `HiCacheBaseServer`, `MMLUMixin`。

### Lines 57-69: class-level constants and configuration for `TestHiCacheStandard` / 类级常量与配置
```python
    """Standard HiCache configuration tests"""

    model_name = DEFAULT_MODEL_NAME_FOR_TEST
    hicache_args = [
        "--enable-hierarchical-cache",
        "--mem-fraction-static",
        0.7,
        "--hicache-size",
        100 if not _is_hip else 200,
    ]
    mmlu_score_threshold = 0.65
    mmlu_num_examples = 64
    mmlu_num_threads = 32
```
**EN:** This block defines shared names such as `model_name`, `hicache_args`, `mmlu_score_threshold`, `mmlu_num_examples`, `mmlu_num_threads`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model_name`, `hicache_args`, `mmlu_score_threshold`, `mmlu_num_examples`, `mmlu_num_threads` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 72-72: class TestHiCacheMLA declaration / 类 TestHiCacheMLA 声明
```python
class TestHiCacheMLA(HiCacheBaseServer, MMLUMixin, MGSMEnMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `HiCacheBaseServer`, `MMLUMixin`, `MGSMEnMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `HiCacheBaseServer`, `MMLUMixin`, `MGSMEnMixin`。

### Lines 73-83: class-level constants and configuration for `TestHiCacheMLA` / 类级常量与配置
```python
    """HiCache with MLA model tests"""

    model_name = DEFAULT_MLA_MODEL_NAME_FOR_TEST
    hicache_args = [
        "--trust-remote-code",
        "--enable-hierarchical-cache",
    ] + (["--hicache-size", 200] if _is_hip else ["--hicache-ratio", 2])
    mmlu_score_threshold = 0.5
    mmlu_num_examples = 64
    mmlu_num_threads = 32
    mgsm_en_score_threshold = 0.8
```
**EN:** This block defines shared names such as `model_name`, `hicache_args`, `mmlu_score_threshold`, `mmlu_num_examples`, `mmlu_num_threads`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model_name`, `hicache_args`, `mmlu_score_threshold`, `mmlu_num_examples`, `mmlu_num_threads` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 86-87: class TestHiCacheEagle declaration / 类 TestHiCacheEagle 声明
```python
@unittest.skipIf(is_hip(), "Disabled for AMD-aiter")
class TestHiCacheEagle(HiCacheBaseServer, MMLUMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `HiCacheBaseServer`, `MMLUMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `HiCacheBaseServer`, `MMLUMixin`。

### Lines 88-116: class-level constants and configuration for `TestHiCacheEagle` / 类级常量与配置
```python
    """HiCache with EAGLE speculative decoding tests"""

    model_name = DEFAULT_TARGET_MODEL_EAGLE3
    needs_tokenizer = True
    hicache_args = [
        "--enable-hierarchical-cache",
        "--hicache-ratio",
        1.2,
        "--mem-fraction-static",
        0.7,
        "--speculative-algorithm",
        "EAGLE3",
        "--speculative-draft-model-path",
        DEFAULT_DRAFT_MODEL_EAGLE3,
        "--speculative-num-steps",
        2,
        "--speculative-eagle-topk",
        1,
        "--speculative-num-draft-tokens",
        3,
        "--dtype",
        "float16",
        "--chunked-prefill-size",
        1024,
    ]
    mmlu_score_threshold = 0.72
    mmlu_num_examples = 64
    mmlu_num_threads = 32
    mmlu_accept_length_thres = 2.26
```
**EN:** This block defines shared names such as `model_name`, `needs_tokenizer`, `hicache_args`, `mmlu_score_threshold`, `mmlu_num_examples`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model_name`, `needs_tokenizer`, `hicache_args`, `mmlu_score_threshold`, `mmlu_num_examples` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 119-119: class TestHiCachePage declaration / 类 TestHiCachePage 声明
```python
class TestHiCachePage(HiCacheBaseServer, MMLUMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `HiCacheBaseServer`, `MMLUMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `HiCacheBaseServer`, `MMLUMixin`。

### Lines 120-132: class-level constants and configuration for `TestHiCachePage` / 类级常量与配置
```python
    """HiCache with custom page size tests"""

    model_name = DEFAULT_MODEL_NAME_FOR_TEST
    hicache_args = [
        "--enable-hierarchical-cache",
        "--page-size",
        32,
        "--hicache-write-policy",
        "write_back",
    ]
    mmlu_score_threshold = 0.65
    mmlu_num_examples = 64
    mmlu_num_threads = 32
```
**EN:** This block defines shared names such as `model_name`, `hicache_args`, `mmlu_score_threshold`, `mmlu_num_examples`, `mmlu_num_threads`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model_name`, `hicache_args`, `mmlu_score_threshold`, `mmlu_num_examples`, `mmlu_num_threads` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 135-136: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `HiCacheBaseServer`: Base class for HiCache tests with configurable server setup / 用于组织相关测试、夹具或辅助方法。
- `TestHiCacheStandard`: Standard HiCache configuration tests / 用于组织相关测试、夹具或辅助方法。
- `TestHiCacheMLA`: HiCache with MLA model tests / 用于组织相关测试、夹具或辅助方法。
- `TestHiCacheEagle`: HiCache with EAGLE speculative decoding tests / 用于组织相关测试、夹具或辅助方法。
- `TestHiCachePage`: HiCache with custom page size tests / 用于组织相关测试、夹具或辅助方法。
- `HiCacheBaseServer.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `HiCacheBaseServer.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.benchmark.utils`, `sglang.srt.utils`, `sglang.test.kits.eval_accuracy_kit`, `sglang.test.test_utils`

- **Total lines / 总行数**: 136
