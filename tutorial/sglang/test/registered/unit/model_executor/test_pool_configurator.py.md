# test_pool_configurator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/model_executor/test_pool_configurator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates pool configurator behavior in SGLang's unit / model executor area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / model executor 领域中与 pool configurator 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: supporting statements / 辅助语句
```python
"""Unit tests for pool_configurator.py -- CPU only, no GPU required.

Tests the end-to-end computation: available_bytes -> MemoryPoolConfig,
verifying tokens are correct, constraints are respected, and memory
invariants hold (tokens * per_token_cost <= available_bytes).
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 8-13: module imports and dependencies / 模块导入与依赖
```python
import contextlib
import unittest
from types import SimpleNamespace
from unittest.mock import MagicMock, patch

from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `contextlib`, `unittest`, `types`, `unittest.mock`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `contextlib`, `unittest`, `types`, `unittest.mock`。

### Lines 15-15: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 18-28: function mock cpu env / 函数 mock cpu env
```python
@contextlib.contextmanager
def mock_cpu_env(kv_size=2, tp_size=1):
    """Mock GPU-dependent functions for CPU-only testing."""
    with (
        patch("torch._utils._element_size", return_value=kv_size),
        patch(
            "sglang.srt.model_executor.pool_configurator.get_attention_tp_size",
            return_value=tp_size,
        ),
    ):
        yield
```
**EN:** Mock GPU-dependent functions for CPU-only testing. This block implements `mock_cpu_env` and captures one focused piece of the module's behavior.
**CN:** Mock GPU-dependent functions for CPU-only testing. 该代码块实现 `mock_cpu_env`，承担模块行为中的一个聚焦逻辑片段。

### Lines 31-90: function make model runner / 函数 make model runner
```python
def _make_model_runner(
    *,
    num_kv_heads=4,
    head_dim=64,
    v_head_dim=64,
    num_layers=32,
    use_mla_backend=False,
    is_hybrid_swa=False,
    full_attention_layer_ids=None,
    swa_attention_layer_ids=None,
    swa_num_kv_heads=None,
    swa_head_dim=None,
    swa_v_head_dim=None,
    swa_full_tokens_ratio=0.5,
    page_size=1,
    mambaish_config=None,
):
    """Create a mock ModelRunner with the fields configurators need."""
    mr = MagicMock()

    mr.use_mla_backend = use_mla_backend
    mr.is_draft_worker = False
    mr.num_effective_layers = num_layers
    mr.start_layer = 0
    mr.end_layer = num_layers
    mr.mambaish_config = mambaish_config
    mr.is_hybrid_swa = is_hybrid_swa

    mc = SimpleNamespace()
    mc.head_dim = head_dim
    mc.v_head_dim = v_head_dim
    mc.is_hybrid_swa = is_hybrid_swa
    mc.full_attention_layer_ids = (
        full_attention_layer_ids
        if full_attention_layer_ids is not None
        else list(range(num_layers))
    )
    mc.swa_attention_layer_ids = (
        swa_attention_layer_ids if swa_attention_layer_ids is not None else []
    )
    mc.swa_head_dim = swa_head_dim or head_dim
    mc.swa_v_head_dim = swa_v_head_dim or v_head_dim
    mc.get_num_kv_heads = lambda tp_size: num_kv_heads
    mc.get_swa_num_kv_heads = lambda tp_size: swa_num_kv_heads or num_kv_heads
    mc.hf_config = SimpleNamespace(architectures=["LlamaForCausalLM"])
    mr.model_config = mc

    mr.kv_cache_dtype = "fake_bf16"

    sa = SimpleNamespace()
    sa.swa_full_tokens_ratio = swa_full_tokens_ratio
    sa.page_size = page_size
    mr.server_args = sa

    spec = MagicMock()
    spec.is_dflash.return_value = False
    spec.is_none.return_value = True
    mr.spec_algorithm = spec

    return mr
```
**EN:** Create a mock ModelRunner with the fields configurators need. This block implements `_make_model_runner` and captures one focused piece of the module's behavior.
**CN:** Create a mock ModelRunner with the fields configurators need. 该代码块实现 `_make_model_runner`，承担模块行为中的一个聚焦逻辑片段。

### Lines 93-93: module-level constants and configuration / 模块级常量与配置
```python
KV_SIZE = 2  # bf16
```
**EN:** This block defines shared names such as `KV_SIZE`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `KV_SIZE` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 96-98: function full per token / 函数 full per token
```python
def _full_per_token(mr):
    mc = mr.model_config
    return mc.get_num_kv_heads(1) * (mc.head_dim + mc.v_head_dim) * KV_SIZE
```
**EN:** This block implements `_full_per_token` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_full_per_token`，承担模块行为中的一个聚焦逻辑片段。

### Lines 101-103: function swa per token / 函数 swa per token
```python
def _swa_per_token(mr):
    mc = mr.model_config
    return mc.get_swa_num_kv_heads(1) * (mc.swa_head_dim + mc.swa_v_head_dim) * KV_SIZE
```
**EN:** This block implements `_swa_per_token` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_swa_per_token`，承担模块行为中的一个聚焦逻辑片段。

### Lines 106-119: function actual memory used / 函数 actual memory used
```python
def _actual_memory_used(mr, config):
    """Compute actual memory consumed by the pool sizes in config."""
    mc = mr.model_config
    full_pt = _full_per_token(mr)
    swa_pt = _swa_per_token(mr)
    nf = len(mc.full_attention_layer_ids)
    ns = len(mc.swa_attention_layer_ids)

    if mr.is_hybrid_swa:
        full = config.full_max_total_num_tokens or 0
        swa = config.swa_max_total_num_tokens or 0
        return full * full_pt * nf + swa * swa_pt * ns
    else:
        return config.max_total_num_tokens * full_pt * (nf + ns)
```
**EN:** Compute actual memory consumed by the pool sizes in config. This block implements `_actual_memory_used` and captures one focused piece of the module's behavior.
**CN:** Compute actual memory consumed by the pool sizes in config. 该代码块实现 `_actual_memory_used`，承担模块行为中的一个聚焦逻辑片段。

### Lines 122-122: class TestDefaultConfigurator declaration / 类 TestDefaultConfigurator 声明
```python
class TestDefaultConfigurator(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 123-123: supporting statements / 辅助语句
```python
    """Default (MHA): available_bytes -> tokens, memory invariant holds."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 125-134: method run / 方法 run
```python
    def _run(self, available_bytes, page_size=1, **kwargs):
        mr = _make_model_runner(page_size=page_size, **kwargs)
        with mock_cpu_env():
            from sglang.srt.model_executor.pool_configurator import (
                create_memory_pool_configurator,
            )

            cfg = create_memory_pool_configurator(mr)
            config = cfg.calculate_pool_sizes(available_bytes, page_size)
        return mr, cfg, config
```
**EN:** This block implements `_run` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run`，承担模块行为中的一个聚焦逻辑片段。

### Lines 136-142: test case memory utilization / 测试用例 memory utilization
```python
    def test_memory_utilization(self):
        """Memory used should be <= available and within 1% of available."""
        available = 10_000_000
        mr, cfg, config = self._run(available)
        used = _actual_memory_used(mr, config)
        self.assertLessEqual(used, available)
        self.assertGreater(used, available * 0.99)
```
**EN:** Memory used should be <= available and within 1% of available. This test exercises `test_memory_utilization` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Memory used should be <= available and within 1% of available. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_memory_utilization`。

### Lines 144-147: test case page alignment / 测试用例 page alignment
```python
    def test_page_alignment(self):
        available = 10_000_000
        _, _, config = self._run(available, page_size=128)
        self.assertEqual(config.max_total_num_tokens % 128, 0)
```
**EN:** This test exercises `test_page_alignment` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_page_alignment`。

### Lines 149-154: test case constraint respected / 测试用例 constraint respected
```python
    def test_constraint_respected(self):
        """calculate_pool_sizes_from_max_tokens respects the limit."""
        mr, cfg, config = self._run(10_000_000)
        with mock_cpu_env():
            constrained = cfg.calculate_pool_sizes_from_max_tokens(100, page_size=1)
        self.assertEqual(constrained.max_total_num_tokens, 100)
```
**EN:** calculate_pool_sizes_from_max_tokens respects the limit. This test exercises `test_constraint_respected` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** calculate_pool_sizes_from_max_tokens respects the limit. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_constraint_respected`。

### Lines 156-160: test case constraint page aligned / 测试用例 constraint page aligned
```python
    def test_constraint_page_aligned(self):
        mr, cfg, _ = self._run(10_000_000, page_size=128)
        with mock_cpu_env():
            constrained = cfg.calculate_pool_sizes_from_max_tokens(1000, page_size=128)
        self.assertEqual(constrained.max_total_num_tokens, 896)  # 1000 // 128 * 128
```
**EN:** This test exercises `test_constraint_page_aligned` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_constraint_page_aligned`。

### Lines 162-165: test case no swa fields / 测试用例 no swa fields
```python
    def test_no_swa_fields(self):
        _, _, config = self._run(10_000_000)
        self.assertIsNone(config.full_max_total_num_tokens)
        self.assertIsNone(config.swa_max_total_num_tokens)
```
**EN:** This test exercises `test_no_swa_fields` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_swa_fields`。

### Lines 168-168: class TestHybridSWAConfigurator declaration / 类 TestHybridSWAConfigurator 声明
```python
class TestHybridSWAConfigurator(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 169-169: supporting statements / 辅助语句
```python
    """Hybrid SWA: full/swa split, ratio, memory invariant."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 171-179: method make swa runner / 方法 make swa runner
```python
    def _make_swa_runner(self, full_layers=16, swa_layers=16, ratio=0.5, page_size=1):
        return _make_model_runner(
            is_hybrid_swa=True,
            full_attention_layer_ids=list(range(full_layers)),
            swa_attention_layer_ids=list(range(full_layers, full_layers + swa_layers)),
            swa_num_kv_heads=4,
            page_size=page_size,
            swa_full_tokens_ratio=ratio,
        )
```
**EN:** This block implements `_make_swa_runner` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_swa_runner`，承担模块行为中的一个聚焦逻辑片段。

### Lines 181-190: method run / 方法 run
```python
    def _run(self, available_bytes, **kwargs):
        mr = self._make_swa_runner(**kwargs)
        with mock_cpu_env():
            from sglang.srt.model_executor.pool_configurator import (
                create_memory_pool_configurator,
            )

            cfg = create_memory_pool_configurator(mr)
            config = cfg.calculate_pool_sizes(available_bytes, mr.server_args.page_size)
        return mr, cfg, config
```
**EN:** This block implements `_run` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run`，承担模块行为中的一个聚焦逻辑片段。

### Lines 192-198: test case memory utilization / 测试用例 memory utilization
```python
    def test_memory_utilization(self):
        """Memory used should be <= available and within 1% of available."""
        available = 10_000_000
        mr, _, config = self._run(available)
        used = _actual_memory_used(mr, config)
        self.assertLessEqual(used, available)
        self.assertGreater(used, available * 0.99)
```
**EN:** Memory used should be <= available and within 1% of available. This test exercises `test_memory_utilization` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Memory used should be <= available and within 1% of available. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_memory_utilization`。

### Lines 200-207: test case ratio respected / 测试用例 ratio respected
```python
    def test_ratio_respected(self):
        """swa_tokens ~= full_tokens * ratio (within page alignment)"""
        available = 10_000_000
        for ratio in [0.25, 0.5, 0.75, 1.0]:
            mr, _, config = self._run(available, ratio=ratio, page_size=1)
            full = config.full_max_total_num_tokens
            swa = config.swa_max_total_num_tokens
            self.assertEqual(swa, int(full * ratio), f"ratio={ratio}")
```
**EN:** swa_tokens ~= full_tokens * ratio (within page alignment) This test exercises `test_ratio_respected` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** swa_tokens ~= full_tokens * ratio (within page alignment) 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ratio_respected`。

### Lines 209-217: test case ratio with page alignment / 测试用例 ratio with page alignment
```python
    def test_ratio_with_page_alignment(self):
        """With page alignment, swa_tokens = align(full_tokens * ratio)"""
        available = 10_000_000
        mr, _, config = self._run(available, ratio=0.5, page_size=128)
        full = config.full_max_total_num_tokens
        swa = config.swa_max_total_num_tokens
        self.assertEqual(full % 128, 0)
        self.assertEqual(swa % 128, 0)
        self.assertEqual(swa, (int(full * 0.5) // 128) * 128)
```
**EN:** With page alignment, swa_tokens = align(full_tokens * ratio) This test exercises `test_ratio_with_page_alignment` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** With page alignment, swa_tokens = align(full_tokens * ratio) 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ratio_with_page_alignment`。

### Lines 219-222: test case max total equals full / 测试用例 max total equals full
```python
    def test_max_total_equals_full(self):
        """For hybrid, max_total_num_tokens = full_max_total_num_tokens"""
        _, _, config = self._run(10_000_000)
        self.assertEqual(config.max_total_num_tokens, config.full_max_total_num_tokens)
```
**EN:** For hybrid, max_total_num_tokens = full_max_total_num_tokens This test exercises `test_max_total_equals_full` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** For hybrid, max_total_num_tokens = full_max_total_num_tokens 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_max_total_equals_full`。

### Lines 224-230: test case constraint respected / 测试用例 constraint respected
```python
    def test_constraint_respected(self):
        """full_tokens = constrained value after re-run"""
        mr, cfg, _ = self._run(10_000_000, page_size=1)
        with mock_cpu_env():
            config = cfg.calculate_pool_sizes_from_max_tokens(200, page_size=1)
        self.assertEqual(config.full_max_total_num_tokens, 200)
        self.assertEqual(config.swa_max_total_num_tokens, 100)
```
**EN:** full_tokens = constrained value after re-run This test exercises `test_constraint_respected` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** full_tokens = constrained value after re-run 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_constraint_respected`。

### Lines 232-245: test case constraint memory within budget / 测试用例 constraint memory within budget
```python
    def test_constraint_memory_within_budget(self):
        """After constraint, memory <= original budget (but less than profiled due to constraint)."""
        available = 10_000_000
        mr, cfg, original = self._run(available, page_size=1)
        user_limit = original.full_max_total_num_tokens // 2
        with mock_cpu_env():
            config = cfg.calculate_pool_sizes_from_max_tokens(
                user_limit, mr.server_args.page_size
            )
        used = _actual_memory_used(mr, config)
        self.assertLessEqual(used, available)
        # constrained should use roughly half the memory
        original_used = _actual_memory_used(mr, original)
        self.assertAlmostEqual(used / original_used, 0.5, delta=0.01)
```
**EN:** After constraint, memory <= original budget (but less than profiled due to constraint). This test exercises `test_constraint_memory_within_budget` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** After constraint, memory <= original budget (but less than profiled due to constraint). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_constraint_memory_within_budget`。

### Lines 247-256: test case different layer counts / 测试用例 different layer counts
```python
    def test_different_layer_counts(self):
        """Asymmetric full/swa layer counts"""
        available = 10_000_000
        mr, _, config = self._run(available, full_layers=24, swa_layers=8, ratio=0.5)
        used = _actual_memory_used(mr, config)
        self.assertLessEqual(used, available)
        self.assertEqual(
            config.swa_max_total_num_tokens,
            int(config.full_max_total_num_tokens * 0.5),
        )
```
**EN:** Asymmetric full/swa layer counts This test exercises `test_different_layer_counts` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Asymmetric full/swa layer counts 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_different_layer_counts`。

### Lines 259-259: class TestAllSWAConfigurator declaration / 类 TestAllSWAConfigurator 声明
```python
class TestAllSWAConfigurator(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 260-260: supporting statements / 辅助语句
```python
    """All-SWA (full_layers=0): special case."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 262-278: method run / 方法 run
```python
    def _run(self, available_bytes, ratio=0.5, page_size=1):
        mr = _make_model_runner(
            is_hybrid_swa=True,
            full_attention_layer_ids=[],
            swa_attention_layer_ids=list(range(32)),
            swa_num_kv_heads=4,
            swa_full_tokens_ratio=ratio,
            page_size=page_size,
        )
        with mock_cpu_env():
            from sglang.srt.model_executor.pool_configurator import (
                create_memory_pool_configurator,
            )

            cfg = create_memory_pool_configurator(mr)
            config = cfg.calculate_pool_sizes(available_bytes, page_size)
        return mr, cfg, config
```
**EN:** This block implements `_run` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run`，承担模块行为中的一个聚焦逻辑片段。

### Lines 280-282: test case full max is zero / 测试用例 full max is zero
```python
    def test_full_max_is_zero(self):
        _, _, config = self._run(10_000_000)
        self.assertEqual(config.full_max_total_num_tokens, 0)
```
**EN:** This test exercises `test_full_max_is_zero` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_full_max_is_zero`。

### Lines 284-286: test case max total equals swa / 测试用例 max total equals swa
```python
    def test_max_total_equals_swa(self):
        _, _, config = self._run(10_000_000)
        self.assertEqual(config.max_total_num_tokens, config.swa_max_total_num_tokens)
```
**EN:** This test exercises `test_max_total_equals_swa` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_max_total_equals_swa`。

### Lines 288-296: test case memory utilization / 测试用例 memory utilization
```python
    def test_memory_utilization(self):
        """Memory used should be <= available and within 1% of available."""
        available = 10_000_000
        mr, _, config = self._run(available)
        swa_pt = _swa_per_token(mr)
        ns = len(mr.model_config.swa_attention_layer_ids)
        used = config.swa_max_total_num_tokens * swa_pt * ns
        self.assertLessEqual(used, available)
        self.assertGreater(used, available * 0.99)
```
**EN:** Memory used should be <= available and within 1% of available. This test exercises `test_memory_utilization` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Memory used should be <= available and within 1% of available. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_memory_utilization`。

### Lines 298-303: test case constraint respected / 测试用例 constraint respected
```python
    def test_constraint_respected(self):
        mr, cfg, _ = self._run(10_000_000, page_size=1)
        with mock_cpu_env():
            config = cfg.calculate_pool_sizes_from_max_tokens(500, page_size=1)
        self.assertEqual(config.max_total_num_tokens, 500)
        self.assertEqual(config.swa_max_total_num_tokens, 500)
```
**EN:** This test exercises `test_constraint_respected` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_constraint_respected`。

### Lines 306-306: class TestFactory declaration / 类 TestFactory 声明
```python
class TestFactory(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 307-316: test case default for non swa / 测试用例 default for non swa
```python
    def test_default_for_non_swa(self):
        mr = _make_model_runner(is_hybrid_swa=False)
        with mock_cpu_env():
            from sglang.srt.model_executor.pool_configurator import (
                DefaultPoolConfigurator,
                create_memory_pool_configurator,
            )

            cfg = create_memory_pool_configurator(mr)
        self.assertIsInstance(cfg, DefaultPoolConfigurator)
```
**EN:** This test exercises `test_default_for_non_swa` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_for_non_swa`。

### Lines 318-332: test case swa for hybrid / 测试用例 swa for hybrid
```python
    def test_swa_for_hybrid(self):
        mr = _make_model_runner(
            is_hybrid_swa=True,
            full_attention_layer_ids=list(range(16)),
            swa_attention_layer_ids=list(range(16, 32)),
            swa_num_kv_heads=4,
        )
        with mock_cpu_env():
            from sglang.srt.model_executor.pool_configurator import (
                HybridSWAPoolConfigurator,
                create_memory_pool_configurator,
            )

            cfg = create_memory_pool_configurator(mr)
        self.assertIsInstance(cfg, HybridSWAPoolConfigurator)
```
**EN:** This test exercises `test_swa_for_hybrid` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_swa_for_hybrid`。

### Lines 335-336: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `mock_cpu_env`: Mock GPU-dependent functions for CPU-only testing. / 该代码块实现 `mock_cpu_env`，承担模块行为中的一个聚焦逻辑片段。
- `_make_model_runner`: Create a mock ModelRunner with the fields configurators need. / 该代码块实现 `_make_model_runner`，承担模块行为中的一个聚焦逻辑片段。
- `_full_per_token`: This block implements `_full_per_token` and captures one focused piece of the module's behavior. / 该代码块实现 `_full_per_token`，承担模块行为中的一个聚焦逻辑片段。
- `_swa_per_token`: This block implements `_swa_per_token` and captures one focused piece of the module's behavior. / 该代码块实现 `_swa_per_token`，承担模块行为中的一个聚焦逻辑片段。
- `_actual_memory_used`: Compute actual memory consumed by the pool sizes in config. / 该代码块实现 `_actual_memory_used`，承担模块行为中的一个聚焦逻辑片段。
- `TestDefaultConfigurator`: Default (MHA): available_bytes -> tokens, memory invariant holds. / 用于组织相关测试、夹具或辅助方法。
- `TestHybridSWAConfigurator`: Hybrid SWA: full/swa split, ratio, memory invariant. / 用于组织相关测试、夹具或辅助方法。
- `TestAllSWAConfigurator`: All-SWA (full_layers=0): special case. / 用于组织相关测试、夹具或辅助方法。
- `TestDefaultConfigurator._run`: This block implements `_run` and captures one focused piece of the module's behavior. / 该代码块实现 `_run`，承担模块行为中的一个聚焦逻辑片段。
- `TestDefaultConfigurator.test_memory_utilization`: Memory used should be <= available and within 1% of available. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_memory_utilization`。
- `TestDefaultConfigurator.test_page_alignment`: This test exercises `test_page_alignment` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_page_alignment`。
- `TestDefaultConfigurator.test_constraint_respected`: calculate_pool_sizes_from_max_tokens respects the limit. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_constraint_respected`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `contextlib`, `unittest`, `types`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 336
