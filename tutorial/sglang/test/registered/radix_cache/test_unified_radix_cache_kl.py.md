# test_unified_radix_cache_kl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/radix_cache/test_unified_radix_cache_kl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates unified radix cache kl behavior in SGLang's radix cache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 Radix 缓存 领域中与 unified radix cache kl 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-22: module imports and dependencies / 模块导入与依赖
```python
import random
import unittest
from types import SimpleNamespace
from urllib.parse import urlparse

from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.kl_multiturn_utils import (
    get_input_ids,
    make_mamba_decode_assert,
    make_mamba_prefill_assert,
    test_input_output_logprobs_match_decode_cache_hit_helper,
    test_input_output_logprobs_match_helper,
    test_input_output_logprobs_match_prefill_cache_hit_helper,
)
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    is_in_ci,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `random`, `unittest`, `types`, `urllib.parse`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `random`, `unittest`, `types`, `urllib.parse`。

### Lines 25-28: function random suffixes / 函数 random suffixes
```python
def _random_suffixes(n, length, seed):
    """Generate n random token-id lists of the given length."""
    rng = random.Random(seed)
    return [[rng.randint(1, 30000) for _ in range(length)] for _ in range(n)]
```
**EN:** Generate n random token-id lists of the given length. This block implements `_random_suffixes` and captures one focused piece of the module's behavior.
**CN:** Generate n random token-id lists of the given length. 该代码块实现 `_random_suffixes`，承担模块行为中的一个聚焦逻辑片段。

### Lines 31-38: CI registration and metadata / CI 注册与元数据
```python
MAMBA_MODEL = "Qwen/Qwen3-Next-80B-A3B-Instruct"
MAMBA_CHUNK_SIZE = 64
MAMBA_TRACK_INTERVAL = 128

SWA_MODEL = "openai/gpt-oss-20b"
FULL_MODEL = "Qwen/Qwen3-32B"

register_cuda_ci(est_time=760, stage="base-c", runner_config="4-gpu-h100")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 41-41: class UnifiedRadixTreeTestMixin declaration / 类 UnifiedRadixTreeTestMixin 声明
```python
class UnifiedRadixTreeTestMixin:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 42-55: class-level constants and configuration for `UnifiedRadixTreeTestMixin` / 类级常量与配置
```python
    """Mixin: gsm8k、mmlu and multi-turn KL tests with multi-branch interleaving."""

    kl_threshold: float = 0.003
    max_new_tokens: int = 512
    num_groups: int = 3
    branches_per_group: int = 3
    prefix_len: int = 512
    prefill_cache_assert = None
    decode_cache_assert = None
    sampling_temperature: float = 1

    gsm8k_threshold: float = 0.93
    mmlu_threshold: float = 0.8
    num_gsm8k_questions: int = 200
```
**EN:** This block defines shared names such as `kl_threshold`, `max_new_tokens`, `num_groups`, `branches_per_group`, `prefix_len`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `kl_threshold`, `max_new_tokens`, `num_groups`, `branches_per_group`, `prefix_len` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 57-76: test case gsm8k / 测试用例 gsm8k
```python
    def test_gsm8k(self):
        """Few-shot GSM8K math reasoning accuracy."""
        from sglang.test.few_shot_gsm8k import run_eval as run_few_shot_gsm8k

        url = urlparse(self.base_url)
        args = SimpleNamespace(
            num_shots=10,
            data_path=None,
            num_questions=self.num_gsm8k_questions,
            max_new_tokens=16000,
            parallel=128,
            host=f"http://{url.hostname}",
            port=int(url.port),
        )
        metrics = run_few_shot_gsm8k(args)
        print(
            f"[{self.__class__.__name__}] GSM8K accuracy: {metrics['accuracy']:.3f} "
            f"(threshold: {self.gsm8k_threshold})"
        )
        self.assertGreaterEqual(metrics["accuracy"], self.gsm8k_threshold)
```
**EN:** Few-shot GSM8K math reasoning accuracy. This test exercises `test_gsm8k` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Few-shot GSM8K math reasoning accuracy. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。

### Lines 78-94: test case mmlu / 测试用例 mmlu
```python
    def test_mmlu(self):
        """Simple-evals MMLU multi-task accuracy."""
        from sglang.test.run_eval import run_eval as run_simple_eval

        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="mmlu",
            num_examples=64,
            num_threads=32,
        )
        metrics = run_simple_eval(args)
        print(
            f"[{self.__class__.__name__}] MMLU score: {metrics['score']:.3f} "
            f"(threshold: {self.mmlu_threshold})"
        )
        self.assertGreaterEqual(metrics["score"], self.mmlu_threshold)
```
**EN:** Simple-evals MMLU multi-task accuracy. This test exercises `test_mmlu` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Simple-evals MMLU multi-task accuracy. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mmlu`。

### Lines 96-111: test case multiturn logprobs match / 测试用例 multiturn logprobs match
```python
    def test_multiturn_logprobs_match(self):
        """Helper 1: 3-turn, no explicit cache seeding."""
        ids = self.input_ids[:4]
        n = len(ids)
        t2 = _random_suffixes(n, 512, seed=100)
        t3 = _random_suffixes(n, 256, seed=200)
        test_input_output_logprobs_match_helper(
            self.base_url,
            self.model,
            self.kl_threshold,
            ids,
            turn_suffixes=[t2, t3],
            assert_decode_cached_tokens=self.decode_cache_assert,
            max_new_tokens=self.max_new_tokens,
            sampling_temperature=self.sampling_temperature,
        )
```
**EN:** Helper 1: 3-turn, no explicit cache seeding. This test exercises `test_multiturn_logprobs_match` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Helper 1: 3-turn, no explicit cache seeding. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiturn_logprobs_match`。

### Lines 113-141: test case multiturn prefill cache hit branching / 测试用例 multiturn prefill cache hit branching
```python
    def test_multiturn_prefill_cache_hit_branching(self):
        """Helper 2: prefill hit + 2 decode-hit turns, multi-branch interleaved."""
        num_groups = self.num_groups
        branches = self.branches_per_group
        n = num_groups * branches
        rng = random.Random(456)
        prefix_ids, full_ids = [], []
        for g in range(num_groups):
            prefix = self.input_ids[g][: self.prefix_len]
            for b in range(branches):
                suffix = [rng.randint(1, 30000) for _ in range(256 + b * 64)]
                prefix_ids.append(list(prefix))
                full_ids.append(prefix + suffix)

        t2 = _random_suffixes(n, 512, seed=789)
        t3 = _random_suffixes(n, 256, seed=890)
        test_input_output_logprobs_match_prefill_cache_hit_helper(
            self.base_url,
            self.model,
            self.kl_threshold,
            prefix_input_ids=prefix_ids,
            full_input_ids=full_ids,
            turn_suffixes=[t2, t3],
            assert_prefill_cached_tokens=self.prefill_cache_assert,
            assert_decode_cached_tokens=self.decode_cache_assert,
            branches_per_group=branches,
            max_new_tokens=self.max_new_tokens,
            sampling_temperature=self.sampling_temperature,
        )
```
**EN:** Helper 2: prefill hit + 2 decode-hit turns, multi-branch interleaved. This test exercises `test_multiturn_prefill_cache_hit_branching` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Helper 2: prefill hit + 2 decode-hit turns, multi-branch interleaved. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiturn_prefill_cache_hit_branching`。

### Lines 143-166: test case multiturn decode cache hit branching / 测试用例 multiturn decode cache hit branching
```python
    def test_multiturn_decode_cache_hit_branching(self):
        """Helper 3: 3-turn decode hit, multi-branch interleaved."""
        num_groups = self.num_groups
        branches = self.branches_per_group
        n = num_groups * branches
        first_turn = []
        for g in range(num_groups):
            base = self.input_ids[g][: self.prefix_len]
            for _ in range(branches):
                first_turn.append(list(base))

        t2 = _random_suffixes(n, 512, seed=300)
        t3 = _random_suffixes(n, 256, seed=400)
        test_input_output_logprobs_match_decode_cache_hit_helper(
            self.base_url,
            self.model,
            self.kl_threshold,
            first_turn,
            turn_suffixes=[t2, t3],
            assert_decode_cached_tokens=self.decode_cache_assert,
            branches_per_group=branches,
            max_new_tokens=self.max_new_tokens,
            sampling_temperature=self.sampling_temperature,
        )
```
**EN:** Helper 3: 3-turn decode hit, multi-branch interleaved. This test exercises `test_multiturn_decode_cache_hit_branching` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Helper 3: 3-turn decode hit, multi-branch interleaved. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiturn_decode_cache_hit_branching`。

### Lines 169-169: class TestUnifiedFullRadixCache declaration / 类 TestUnifiedFullRadixCache 声明
```python
class TestUnifiedFullRadixCache(UnifiedRadixTreeTestMixin, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `UnifiedRadixTreeTestMixin`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `UnifiedRadixTreeTestMixin`, `CustomTestCase`。

### Lines 170-172: class-level constants and configuration for `TestUnifiedFullRadixCache` / 类级常量与配置
```python
    """Full attention."""

    kl_threshold = 0.0025
```
**EN:** This block defines shared names such as `kl_threshold`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `kl_threshold` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 174-192: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = FULL_MODEL
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--tp-size",
                "4",
                "--mem-fraction-static",
                "0.80",
                "--page-size",
                "64",
            ],
            env={"SGLANG_ENABLE_UNIFIED_RADIX_TREE": "1"},
        )
        cls.input_ids = get_input_ids(cls.model, num_samples=18)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 194-196: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 199-199: class TestUnifiedMambaRadixCache declaration / 类 TestUnifiedMambaRadixCache 声明
```python
class TestUnifiedMambaRadixCache(UnifiedRadixTreeTestMixin, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `UnifiedRadixTreeTestMixin`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `UnifiedRadixTreeTestMixin`, `CustomTestCase`。

### Lines 200-208: class-level constants and configuration for `TestUnifiedMambaRadixCache` / 类级常量与配置
```python
    """Mamba hybrid + UnifiedRadixCache."""

    kl_threshold = 0.003
    prefill_cache_assert = staticmethod(
        make_mamba_prefill_assert(chunk_size=MAMBA_CHUNK_SIZE)
    )
    decode_cache_assert = staticmethod(
        make_mamba_decode_assert(track_interval=MAMBA_TRACK_INTERVAL)
    )
```
**EN:** This block defines shared names such as `kl_threshold`, `prefill_cache_assert`, `decode_cache_assert`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `kl_threshold`, `prefill_cache_assert`, `decode_cache_assert` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 210-232: setUpClass setup routine / setUpClass 初始化流程
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
            ],
            env={"SGLANG_ENABLE_UNIFIED_RADIX_TREE": "1"},
        )
        cls.input_ids = get_input_ids(cls.model, num_samples=18)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 234-236: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 239-239: class TestUnifiedSWARadixCache declaration / 类 TestUnifiedSWARadixCache 声明
```python
class TestUnifiedSWARadixCache(UnifiedRadixTreeTestMixin, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `UnifiedRadixTreeTestMixin`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `UnifiedRadixTreeTestMixin`, `CustomTestCase`。

### Lines 240-244: class-level constants and configuration for `TestUnifiedSWARadixCache` / 类级常量与配置
```python
    """SWA hybrid + UnifiedRadixCache."""

    kl_threshold = 0.03
    gsm8k_threshold = 0.7
    mmlu_threshold = 0.7
```
**EN:** This block defines shared names such as `kl_threshold`, `gsm8k_threshold`, `mmlu_threshold`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `kl_threshold`, `gsm8k_threshold`, `mmlu_threshold` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 246-248: test case mmlu / 测试用例 mmlu
```python
    @unittest.skipIf(is_in_ci(), "SWA model mmlu eval not stable enough")
    def test_mmlu(self):
        super().test_mmlu()
```
**EN:** This test exercises `test_mmlu` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mmlu`。

### Lines 250-267: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = SWA_MODEL
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--tp-size",
                "4",
                "--mem-fraction-static",
                "0.7",
                "--disable-piecewise-cuda-graph",
            ],
            env={"SGLANG_ENABLE_UNIFIED_RADIX_TREE": "1"},
        )
        cls.input_ids = get_input_ids(cls.model, num_samples=18)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 269-271: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 274-275: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_random_suffixes`: Generate n random token-id lists of the given length. / 该代码块实现 `_random_suffixes`，承担模块行为中的一个聚焦逻辑片段。
- `UnifiedRadixTreeTestMixin`: Mixin: gsm8k、mmlu and multi-turn KL tests with multi-branch interleaving. / 用于组织相关测试、夹具或辅助方法。
- `TestUnifiedFullRadixCache`: Full attention. / 用于组织相关测试、夹具或辅助方法。
- `TestUnifiedMambaRadixCache`: Mamba hybrid + UnifiedRadixCache. / 用于组织相关测试、夹具或辅助方法。
- `TestUnifiedSWARadixCache`: SWA hybrid + UnifiedRadixCache. / 用于组织相关测试、夹具或辅助方法。
- `UnifiedRadixTreeTestMixin.test_gsm8k`: Few-shot GSM8K math reasoning accuracy. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k`。
- `UnifiedRadixTreeTestMixin.test_mmlu`: Simple-evals MMLU multi-task accuracy. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mmlu`。
- `UnifiedRadixTreeTestMixin.test_multiturn_logprobs_match`: Helper 1: 3-turn, no explicit cache seeding. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiturn_logprobs_match`。
- `UnifiedRadixTreeTestMixin.test_multiturn_prefill_cache_hit_branching`: Helper 2: prefill hit + 2 decode-hit turns, multi-branch interleaved. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiturn_prefill_cache_hit_branching`。
- `UnifiedRadixTreeTestMixin.test_multiturn_decode_cache_hit_branching`: Helper 3: 3-turn decode hit, multi-branch interleaved. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiturn_decode_cache_hit_branching`。
- `TestUnifiedFullRadixCache.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestUnifiedFullRadixCache.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `random`, `unittest`, `types`, `urllib.parse`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.kl_multiturn_utils`, `sglang.test.test_utils`

- **Total lines / 总行数**: 275
