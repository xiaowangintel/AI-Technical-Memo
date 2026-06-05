# test_sampling_params.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/sampling/test_sampling_params.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates sampling params behavior in SGLang's unit / sampling area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / sampling 领域中与 sampling params 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for srt/sampling/sampling_params.py — no server, no model loading."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-3: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 5-5: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=7, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 7-16: module imports and dependencies / 模块导入与依赖
```python
import unittest
from unittest.mock import MagicMock

from sglang.srt.sampling.sampling_params import (
    MAX_LEN,
    TOP_K_ALL,
    SamplingParams,
    get_max_seq_length,
)
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `unittest.mock`, `sglang.srt.sampling.sampling_params`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `unittest.mock`, `sglang.srt.sampling.sampling_params`, `sglang.test.test_utils`。

### Lines 19-20: class TestSamplingParamsInit declaration / 类 TestSamplingParamsInit 声明
```python
class TestSamplingParamsInit(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 21-25: test case zero temperature becomes greedy / 测试用例 zero temperature becomes greedy
```python
    def test_zero_temperature_becomes_greedy(self):
        """Test greedy conversion when temperature is 0."""
        sp = SamplingParams(temperature=0.0)
        self.assertEqual(sp.top_k, 1)
        self.assertEqual(sp.temperature, 1.0)
```
**EN:** Test greedy conversion when temperature is 0. This test exercises `test_zero_temperature_becomes_greedy` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test greedy conversion when temperature is 0. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_zero_temperature_becomes_greedy`。

### Lines 27-31: test case near zero temperature becomes greedy / 测试用例 near zero temperature becomes greedy
```python
    def test_near_zero_temperature_becomes_greedy(self):
        """Test greedy conversion when temperature is near zero (1e-7)."""
        sp = SamplingParams(temperature=1e-7)
        self.assertEqual(sp.top_k, 1)
        self.assertEqual(sp.temperature, 1.0)
```
**EN:** Test greedy conversion when temperature is near zero (1e-7). This test exercises `test_near_zero_temperature_becomes_greedy` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test greedy conversion when temperature is near zero (1e-7). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_near_zero_temperature_becomes_greedy`。

### Lines 33-38: test case temperature at eps boundary not greedy / 测试用例 temperature at eps boundary not greedy
```python
    def test_temperature_at_eps_boundary_not_greedy(self):
        """Test that temperature exactly at 1e-6 does not trigger greedy (strict <)."""
        sp = SamplingParams(temperature=1e-6)
        self.assertEqual(sp.temperature, 1e-6)
        # top_k should remain at TOP_K_ALL (from -1 default)
        self.assertEqual(sp.top_k, TOP_K_ALL)
```
**EN:** Test that temperature exactly at 1e-6 does not trigger greedy (strict <). This test exercises `test_temperature_at_eps_boundary_not_greedy` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that temperature exactly at 1e-6 does not trigger greedy (strict <). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_temperature_at_eps_boundary_not_greedy`。

### Lines 40-43: test case negative temperature not modified / 测试用例 negative temperature not modified
```python
    def test_negative_temperature_not_modified(self):
        """Test that __init__ preserves negative temperature (rejected by verify instead)."""
        sp = SamplingParams(temperature=-1.0)
        self.assertEqual(sp.temperature, -1.0)
```
**EN:** Test that __init__ preserves negative temperature (rejected by verify instead). This test exercises `test_negative_temperature_not_modified` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that __init__ preserves negative temperature (rejected by verify instead). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_negative_temperature_not_modified`。

### Lines 45-48: test case top k minus one becomes top k all / 测试用例 top k minus one becomes top k all
```python
    def test_top_k_minus_one_becomes_top_k_all(self):
        """Test that top_k=-1 is converted to TOP_K_ALL (whole vocabulary)."""
        sp = SamplingParams(top_k=-1)
        self.assertEqual(sp.top_k, TOP_K_ALL)
```
**EN:** Test that top_k=-1 is converted to TOP_K_ALL (whole vocabulary). This test exercises `test_top_k_minus_one_becomes_top_k_all` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that top_k=-1 is converted to TOP_K_ALL (whole vocabulary). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_top_k_minus_one_becomes_top_k_all`。

### Lines 50-53: test case positive top k preserved / 测试用例 positive top k preserved
```python
    def test_positive_top_k_preserved(self):
        """Test that explicit positive top_k is kept as-is."""
        sp = SamplingParams(top_k=50)
        self.assertEqual(sp.top_k, 50)
```
**EN:** Test that explicit positive top_k is kept as-is. This test exercises `test_positive_top_k_preserved` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that explicit positive top_k is kept as-is. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_positive_top_k_preserved`。

### Lines 55-59: test case stop token ids stored as set / 测试用例 stop token ids stored as set
```python
    def test_stop_token_ids_stored_as_set(self):
        """Test that stop_token_ids list is converted to set."""
        sp = SamplingParams(stop_token_ids=[1, 2, 3])
        self.assertIsInstance(sp.stop_token_ids, set)
        self.assertEqual(sp.stop_token_ids, {1, 2, 3})
```
**EN:** Test that stop_token_ids list is converted to set. This test exercises `test_stop_token_ids_stored_as_set` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that stop_token_ids list is converted to set. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stop_token_ids_stored_as_set`。

### Lines 61-64: test case stop token ids none stays none / 测试用例 stop token ids none stays none
```python
    def test_stop_token_ids_none_stays_none(self):
        """Test that None stop_token_ids stays None."""
        sp = SamplingParams(stop_token_ids=None)
        self.assertIsNone(sp.stop_token_ids)
```
**EN:** Test that None stop_token_ids stays None. This test exercises `test_stop_token_ids_none_stays_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that None stop_token_ids stays None. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stop_token_ids_none_stays_none`。

### Lines 66-69: test case empty stop token ids becomes none / 测试用例 empty stop token ids becomes none
```python
    def test_empty_stop_token_ids_becomes_none(self):
        """Test that empty list is treated as None (falsy in Python)."""
        sp = SamplingParams(stop_token_ids=[])
        self.assertIsNone(sp.stop_token_ids)
```
**EN:** Test that empty list is treated as None (falsy in Python). This test exercises `test_empty_stop_token_ids_becomes_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that empty list is treated as None (falsy in Python). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_stop_token_ids_becomes_none`。

### Lines 72-73: class TestSamplingParamsVerify declaration / 类 TestSamplingParamsVerify 声明
```python
class TestSamplingParamsVerify(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 74-74: class-level constants and configuration for `TestSamplingParamsVerify` / 类级常量与配置
```python
    VOCAB_SIZE = 32000
```
**EN:** This block defines shared names such as `VOCAB_SIZE`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `VOCAB_SIZE` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 76-80: method make / 方法 make
```python
    def _make(self, **kwargs):
        """Helper: create SamplingParams with safe defaults, override with kwargs."""
        defaults = dict(temperature=1.0, top_p=1.0, top_k=10, min_p=0.0)
        defaults.update(kwargs)
        return SamplingParams(**defaults)
```
**EN:** Helper: create SamplingParams with safe defaults, override with kwargs. This block implements `_make` and captures one focused piece of the module's behavior.
**CN:** Helper: create SamplingParams with safe defaults, override with kwargs. 该代码块实现 `_make`，承担模块行为中的一个聚焦逻辑片段。

### Lines 82-85: test case valid params pass / 测试用例 valid params pass
```python
    def test_valid_params_pass(self):
        """Default valid params should pass verify() without raising."""
        sp = self._make()
        sp.verify(self.VOCAB_SIZE)
```
**EN:** Default valid params should pass verify() without raising. This test exercises `test_valid_params_pass` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Default valid params should pass verify() without raising. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_valid_params_pass`。

### Lines 87-91: test case negative temperature raises / 测试用例 negative temperature raises
```python
    def test_negative_temperature_raises(self):
        """Test that verify() rejects negative temperature (must be >= 0)."""
        sp = self._make(temperature=-0.5)
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects negative temperature (must be >= 0). This test exercises `test_negative_temperature_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects negative temperature (must be >= 0). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_negative_temperature_raises`。

### Lines 92-93: supporting source context / 辅助源码上下文
```python

    # --- top_p ---
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 94-98: test case top p negative raises / 测试用例 top p negative raises
```python
    def test_top_p_negative_raises(self):
        """Test that verify() rejects negative top_p (valid range is (0, 1])."""
        sp = self._make(top_p=-0.5)
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects negative top_p (valid range is (0, 1]). This test exercises `test_top_p_negative_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects negative top_p (valid range is (0, 1]). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_top_p_negative_raises`。

### Lines 100-104: test case top p zero raises / 测试用例 top p zero raises
```python
    def test_top_p_zero_raises(self):
        """Test that verify() rejects top_p=0 (not in (0, 1])."""
        sp = self._make(top_p=0.0)
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects top_p=0 (not in (0, 1]). This test exercises `test_top_p_zero_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects top_p=0 (not in (0, 1]). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_top_p_zero_raises`。

### Lines 106-110: test case top p above one raises / 测试用例 top p above one raises
```python
    def test_top_p_above_one_raises(self):
        """Test that verify() rejects top_p > 1.0."""
        sp = self._make(top_p=1.1)
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects top_p > 1.0. This test exercises `test_top_p_above_one_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects top_p > 1.0. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_top_p_above_one_raises`。

### Lines 112-115: test case top p exactly one is valid / 测试用例 top p exactly one is valid
```python
    def test_top_p_exactly_one_is_valid(self):
        """Test that top_p=1.0 is accepted (inclusive upper bound)."""
        sp = self._make(top_p=1.0)
        sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that top_p=1.0 is accepted (inclusive upper bound). This test exercises `test_top_p_exactly_one_is_valid` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that top_p=1.0 is accepted (inclusive upper bound). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_top_p_exactly_one_is_valid`。

### Lines 117-120: test case top p small positive is valid / 测试用例 top p small positive is valid
```python
    def test_top_p_small_positive_is_valid(self):
        """Test that a small positive top_p (0.01) is accepted."""
        sp = self._make(top_p=0.01)
        sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that a small positive top_p (0.01) is accepted. This test exercises `test_top_p_small_positive_is_valid` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that a small positive top_p (0.01) is accepted. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_top_p_small_positive_is_valid`。

### Lines 121-122: supporting source context / 辅助源码上下文
```python

    # --- min_p ---
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 123-127: test case min p negative raises / 测试用例 min p negative raises
```python
    def test_min_p_negative_raises(self):
        """Test that verify() rejects negative min_p (valid range is [0, 1])."""
        sp = self._make(min_p=-0.1)
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects negative min_p (valid range is [0, 1]). This test exercises `test_min_p_negative_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects negative min_p (valid range is [0, 1]). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_min_p_negative_raises`。

### Lines 129-133: test case min p above one raises / 测试用例 min p above one raises
```python
    def test_min_p_above_one_raises(self):
        """Test that verify() rejects min_p > 1.0."""
        sp = self._make(min_p=1.01)
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects min_p > 1.0. This test exercises `test_min_p_above_one_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects min_p > 1.0. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_min_p_above_one_raises`。

### Lines 135-138: test case min p boundaries valid / 测试用例 min p boundaries valid
```python
    def test_min_p_boundaries_valid(self):
        """Test that both 0.0 and 1.0 are accepted."""
        self._make(min_p=0.0).verify(self.VOCAB_SIZE)
        self._make(min_p=1.0).verify(self.VOCAB_SIZE)
```
**EN:** Test that both 0.0 and 1.0 are accepted. This test exercises `test_min_p_boundaries_valid` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that both 0.0 and 1.0 are accepted. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_min_p_boundaries_valid`。

### Lines 140-145: test case top k zero raises / 测试用例 top k zero raises
```python
    def test_top_k_zero_raises(self):
        """Test that verify() rejects top_k=0 (must be >=1 or -1 for all)."""
        sp = self._make()
        sp.top_k = 0  # bypass __init__ conversion
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects top_k=0 (must be >=1 or -1 for all). This test exercises `test_top_k_zero_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects top_k=0 (must be >=1 or -1 for all). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_top_k_zero_raises`。

### Lines 147-152: test case top k negative raises / 测试用例 top k negative raises
```python
    def test_top_k_negative_raises(self):
        """Test that top_k=-2 is rejected (__init__ only converts -1)."""
        sp = self._make()
        sp.top_k = -2  # bypass __init__ conversion
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that top_k=-2 is rejected (__init__ only converts -1). This test exercises `test_top_k_negative_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that top_k=-2 is rejected (__init__ only converts -1). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_top_k_negative_raises`。

### Lines 153-154: supporting source context / 辅助源码上下文
```python

    # --- frequency_penalty ---
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 155-159: test case frequency penalty below minus two raises / 测试用例 frequency penalty below minus two raises
```python
    def test_frequency_penalty_below_minus_two_raises(self):
        """Test that verify() rejects frequency_penalty < -2.0."""
        sp = self._make(frequency_penalty=-2.1)
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects frequency_penalty < -2.0. This test exercises `test_frequency_penalty_below_minus_two_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects frequency_penalty < -2.0. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_frequency_penalty_below_minus_two_raises`。

### Lines 161-165: test case frequency penalty above two raises / 测试用例 frequency penalty above two raises
```python
    def test_frequency_penalty_above_two_raises(self):
        """Test that verify() rejects frequency_penalty > 2.0."""
        sp = self._make(frequency_penalty=2.1)
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects frequency_penalty > 2.0. This test exercises `test_frequency_penalty_above_two_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects frequency_penalty > 2.0. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_frequency_penalty_above_two_raises`。

### Lines 167-170: test case frequency penalty boundaries valid / 测试用例 frequency penalty boundaries valid
```python
    def test_frequency_penalty_boundaries_valid(self):
        """Test that both -2.0 and 2.0 are accepted."""
        self._make(frequency_penalty=-2.0).verify(self.VOCAB_SIZE)
        self._make(frequency_penalty=2.0).verify(self.VOCAB_SIZE)
```
**EN:** Test that both -2.0 and 2.0 are accepted. This test exercises `test_frequency_penalty_boundaries_valid` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that both -2.0 and 2.0 are accepted. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_frequency_penalty_boundaries_valid`。

### Lines 171-172: supporting source context / 辅助源码上下文
```python

    # --- presence_penalty ---
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 173-177: test case presence penalty out of range raises / 测试用例 presence penalty out of range raises
```python
    def test_presence_penalty_out_of_range_raises(self):
        """Test that verify() rejects presence_penalty outside [-2, 2]."""
        sp = self._make(presence_penalty=2.5)
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects presence_penalty outside [-2, 2]. This test exercises `test_presence_penalty_out_of_range_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects presence_penalty outside [-2, 2]. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_presence_penalty_out_of_range_raises`。

### Lines 178-179: supporting source context / 辅助源码上下文
```python

    # --- repetition_penalty ---
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 180-184: test case repetition penalty negative raises / 测试用例 repetition penalty negative raises
```python
    def test_repetition_penalty_negative_raises(self):
        """Test that verify() rejects negative repetition_penalty (valid range is (0, 2])."""
        sp = self._make(repetition_penalty=-0.1)
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects negative repetition_penalty (valid range is (0, 2]). This test exercises `test_repetition_penalty_negative_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects negative repetition_penalty (valid range is (0, 2]). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_repetition_penalty_negative_raises`。

### Lines 186-195: test case repetition penalty zero raises / 测试用例 repetition penalty zero raises
```python
    def test_repetition_penalty_zero_raises(self):
        """Test that verify() rejects repetition_penalty=0.

        A value of 0 makes the sampling kernel divide logits by 0, producing
        inf/NaN in the probability tensor and crashing every TP rank with a
        device-side assert.
        """
        sp = self._make(repetition_penalty=0.0)
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects repetition_penalty=0. This test exercises `test_repetition_penalty_zero_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects repetition_penalty=0. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_repetition_penalty_zero_raises`。

### Lines 197-201: test case repetition penalty above two raises / 测试用例 repetition penalty above two raises
```python
    def test_repetition_penalty_above_two_raises(self):
        """Test that verify() rejects repetition_penalty > 2.0."""
        sp = self._make(repetition_penalty=2.1)
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects repetition_penalty > 2.0. This test exercises `test_repetition_penalty_above_two_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects repetition_penalty > 2.0. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_repetition_penalty_above_two_raises`。

### Lines 203-205: test case repetition penalty boundary two valid / 测试用例 repetition penalty boundary two valid
```python
    def test_repetition_penalty_boundary_two_valid(self):
        """Test that the upper boundary value 2.0 is accepted."""
        self._make(repetition_penalty=2.0).verify(self.VOCAB_SIZE)
```
**EN:** Test that the upper boundary value 2.0 is accepted. This test exercises `test_repetition_penalty_boundary_two_valid` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that the upper boundary value 2.0 is accepted. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_repetition_penalty_boundary_two_valid`。

### Lines 207-209: test case repetition penalty small positive valid / 测试用例 repetition penalty small positive valid
```python
    def test_repetition_penalty_small_positive_valid(self):
        """Test that a small positive repetition_penalty (e.g. 1e-3) is accepted."""
        self._make(repetition_penalty=1e-3).verify(self.VOCAB_SIZE)
```
**EN:** Test that a small positive repetition_penalty (e.g. This test exercises `test_repetition_penalty_small_positive_valid` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that a small positive repetition_penalty (e.g. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_repetition_penalty_small_positive_valid`。

### Lines 210-211: supporting source context / 辅助源码上下文
```python

    # --- min_new_tokens / max_new_tokens ---
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 212-216: test case negative min new tokens raises / 测试用例 negative min new tokens raises
```python
    def test_negative_min_new_tokens_raises(self):
        """Test that verify() rejects negative min_new_tokens."""
        sp = self._make(min_new_tokens=-1)
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects negative min_new_tokens. This test exercises `test_negative_min_new_tokens_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects negative min_new_tokens. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_negative_min_new_tokens_raises`。

### Lines 218-222: test case negative max new tokens raises / 测试用例 negative max new tokens raises
```python
    def test_negative_max_new_tokens_raises(self):
        """Test that verify() rejects negative max_new_tokens."""
        sp = self._make(max_new_tokens=-1)
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects negative max_new_tokens. This test exercises `test_negative_max_new_tokens_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects negative max_new_tokens. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_negative_max_new_tokens_raises`。

### Lines 224-228: test case min exceeds max new tokens raises / 测试用例 min exceeds max new tokens raises
```python
    def test_min_exceeds_max_new_tokens_raises(self):
        """Test that verify() rejects min_new_tokens > max_new_tokens."""
        sp = self._make(min_new_tokens=100, max_new_tokens=50)
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects min_new_tokens > max_new_tokens. This test exercises `test_min_exceeds_max_new_tokens_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects min_new_tokens > max_new_tokens. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_min_exceeds_max_new_tokens_raises`。

### Lines 230-233: test case min equals max new tokens valid / 测试用例 min equals max new tokens valid
```python
    def test_min_equals_max_new_tokens_valid(self):
        """Test that min_new_tokens == max_new_tokens is accepted."""
        sp = self._make(min_new_tokens=10, max_new_tokens=10)
        sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that min_new_tokens == max_new_tokens is accepted. This test exercises `test_min_equals_max_new_tokens_valid` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that min_new_tokens == max_new_tokens is accepted. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_min_equals_max_new_tokens_valid`。

### Lines 235-238: test case max new tokens none skips validation / 测试用例 max new tokens none skips validation
```python
    def test_max_new_tokens_none_skips_validation(self):
        """Test that max_new_tokens=None skips the min<=max check."""
        sp = self._make(min_new_tokens=9999, max_new_tokens=None)
        sp.verify(self.VOCAB_SIZE)  # should not raise
```
**EN:** Test that max_new_tokens=None skips the min<=max check. This test exercises `test_max_new_tokens_none_skips_validation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that max_new_tokens=None skips the min<=max check. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_max_new_tokens_none_skips_validation`。

### Lines 239-240: supporting source context / 辅助源码上下文
```python

    # --- logit_bias ---
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 241-245: test case logit bias token exceeds vocab raises / 测试用例 logit bias token exceeds vocab raises
```python
    def test_logit_bias_token_exceeds_vocab_raises(self):
        """Test that verify() rejects logit_bias with token_id >= vocab_size."""
        sp = self._make(logit_bias={"99999": 1.0})
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects logit_bias with token_id >= vocab_size. This test exercises `test_logit_bias_token_exceeds_vocab_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects logit_bias with token_id >= vocab_size. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_logit_bias_token_exceeds_vocab_raises`。

### Lines 247-251: test case logit bias negative token raises / 测试用例 logit bias negative token raises
```python
    def test_logit_bias_negative_token_raises(self):
        """Test that verify() rejects logit_bias with negative token_id."""
        sp = self._make(logit_bias={"-1": 1.0})
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects logit_bias with negative token_id. This test exercises `test_logit_bias_negative_token_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects logit_bias with negative token_id. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_logit_bias_negative_token_raises`。

### Lines 253-256: test case logit bias valid tokens / 测试用例 logit bias valid tokens
```python
    def test_logit_bias_valid_tokens(self):
        """Test that logit_bias with token_ids within [0, vocab_size) is accepted."""
        sp = self._make(logit_bias={"0": 1.0, "31999": -0.5})
        sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that logit_bias with token_ids within [0, vocab_size) is accepted. This test exercises `test_logit_bias_valid_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that logit_bias with token_ids within [0, vocab_size) is accepted. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_logit_bias_valid_tokens`。

### Lines 258-262: test case multiple grammars raises / 测试用例 multiple grammars raises
```python
    def test_multiple_grammars_raises(self):
        """Test that verify() rejects setting both json_schema and regex (mutually exclusive)."""
        sp = self._make(json_schema='{"type":"object"}', regex="abc")
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects setting both json_schema and regex (mutually exclusive). This test exercises `test_multiple_grammars_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects setting both json_schema and regex (mutually exclusive). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_grammars_raises`。

### Lines 264-267: test case single grammar valid / 测试用例 single grammar valid
```python
    def test_single_grammar_valid(self):
        """Test that setting only one grammar type is accepted."""
        sp = self._make(json_schema='{"type":"object"}')
        sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that setting only one grammar type is accepted. This test exercises `test_single_grammar_valid` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that setting only one grammar type is accepted. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_grammar_valid`。

### Lines 269-273: test case all three grammars set raises / 测试用例 all three grammars set raises
```python
    def test_all_three_grammars_set_raises(self):
        """Test that verify() rejects setting json_schema, regex, and ebnf together."""
        sp = self._make(json_schema="{}", regex="a", ebnf="rule")
        with self.assertRaises(ValueError):
            sp.verify(self.VOCAB_SIZE)
```
**EN:** Test that verify() rejects setting json_schema, regex, and ebnf together. This test exercises `test_all_three_grammars_set_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that verify() rejects setting json_schema, regex, and ebnf together. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_three_grammars_set_raises`。

### Lines 276-277: class TestSamplingParamsNormalize declaration / 类 TestSamplingParamsNormalize 声明
```python
class TestSamplingParamsNormalize(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 278-283: test case none stop strs becomes empty list / 测试用例 none stop strs becomes empty list
```python
    def test_none_stop_strs_becomes_empty_list(self):
        """Test that normalize() converts None stop to empty list with max_len=0."""
        sp = SamplingParams(stop=None)
        sp.normalize(tokenizer=None)
        self.assertEqual(sp.stop_strs, [])
        self.assertEqual(sp.stop_str_max_len, 0)
```
**EN:** Test that normalize() converts None stop to empty list with max_len=0. This test exercises `test_none_stop_strs_becomes_empty_list` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that normalize() converts None stop to empty list with max_len=0. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_stop_strs_becomes_empty_list`。

### Lines 285-289: test case string stop str wrapped in list / 测试用例 string stop str wrapped in list
```python
    def test_string_stop_str_wrapped_in_list(self):
        """Test that normalize() wraps a single stop string into a list."""
        sp = SamplingParams(stop="<|end|>")
        sp.normalize(tokenizer=None)
        self.assertEqual(sp.stop_strs, ["<|end|>"])
```
**EN:** Test that normalize() wraps a single stop string into a list. This test exercises `test_string_stop_str_wrapped_in_list` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that normalize() wraps a single stop string into a list. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_string_stop_str_wrapped_in_list`。

### Lines 291-295: test case list stop strs unchanged / 测试用例 list stop strs unchanged
```python
    def test_list_stop_strs_unchanged(self):
        """Test that normalize() preserves a list of stop strings as-is."""
        sp = SamplingParams(stop=["stop1", "stop2"])
        sp.normalize(tokenizer=None)
        self.assertEqual(sp.stop_strs, ["stop1", "stop2"])
```
**EN:** Test that normalize() preserves a list of stop strings as-is. This test exercises `test_list_stop_strs_unchanged` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that normalize() preserves a list of stop strings as-is. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_list_stop_strs_unchanged`。

### Lines 297-301: test case stop str max len without tokenizer / 测试用例 stop str max len without tokenizer
```python
    def test_stop_str_max_len_without_tokenizer(self):
        """Test that without a tokenizer, max_len is the raw string character count."""
        sp = SamplingParams(stop=["ab", "cdef"])
        sp.normalize(tokenizer=None)
        self.assertEqual(sp.stop_str_max_len, 4)  # len("cdef")
```
**EN:** Test that without a tokenizer, max_len is the raw string character count. This test exercises `test_stop_str_max_len_without_tokenizer` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that without a tokenizer, max_len is the raw string character count. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stop_str_max_len_without_tokenizer`。

### Lines 303-313: test case stop str max len with tokenizer / 测试用例 stop str max len with tokenizer
```python
    def test_stop_str_max_len_with_tokenizer(self):
        """Test that with a tokenizer, max_len counts encoded token IDs."""
        tokenizer = MagicMock()
        # "hello" encodes to 2 tokens, "world!!" to 3 tokens
        tokenizer.encode.side_effect = lambda s, add_special_tokens=False: {
            "hello": [101, 102],
            "world!!": [201, 202, 203],
        }[s]
        sp = SamplingParams(stop=["hello", "world!!"])
        sp.normalize(tokenizer=tokenizer)
        self.assertEqual(sp.stop_str_max_len, 3)
```
**EN:** Test that with a tokenizer, max_len counts encoded token IDs. This test exercises `test_stop_str_max_len_with_tokenizer` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that with a tokenizer, max_len counts encoded token IDs. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stop_str_max_len_with_tokenizer`。

### Lines 315-320: test case none stop regex becomes empty list / 测试用例 none stop regex becomes empty list
```python
    def test_none_stop_regex_becomes_empty_list(self):
        """Test that normalize() converts None stop_regex to empty list with max_len=0."""
        sp = SamplingParams(stop_regex=None)
        sp.normalize(tokenizer=None)
        self.assertEqual(sp.stop_regex_strs, [])
        self.assertEqual(sp.stop_regex_max_len, 0)
```
**EN:** Test that normalize() converts None stop_regex to empty list with max_len=0. This test exercises `test_none_stop_regex_becomes_empty_list` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that normalize() converts None stop_regex to empty list with max_len=0. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_stop_regex_becomes_empty_list`。

### Lines 322-326: test case string stop regex wrapped in list / 测试用例 string stop regex wrapped in list
```python
    def test_string_stop_regex_wrapped_in_list(self):
        """Test that normalize() wraps a single stop_regex string into a list."""
        sp = SamplingParams(stop_regex=r"\d+")
        sp.normalize(tokenizer=None)
        self.assertEqual(sp.stop_regex_strs, [r"\d+"])
```
**EN:** Test that normalize() wraps a single stop_regex string into a list. This test exercises `test_string_stop_regex_wrapped_in_list` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that normalize() wraps a single stop_regex string into a list. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_string_stop_regex_wrapped_in_list`。

### Lines 328-332: test case stop regex max len computed / 测试用例 stop regex max len computed
```python
    def test_stop_regex_max_len_computed(self):
        """Test that bounded regex computes a finite max length."""
        sp = SamplingParams(stop_regex=r"[a-z]{3}")
        sp.normalize(tokenizer=None)
        self.assertEqual(sp.stop_regex_max_len, 3)
```
**EN:** Test that bounded regex computes a finite max length. This test exercises `test_stop_regex_max_len_computed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that bounded regex computes a finite max length. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stop_regex_max_len_computed`。

### Lines 335-336: class TestRegexMaxLength declaration / 类 TestRegexMaxLength 声明
```python
class TestRegexMaxLength(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 337-339: test case literal string / 测试用例 literal string
```python
    def test_literal_string(self):
        """Test that plain string 'abc' gives max length 3."""
        self.assertEqual(get_max_seq_length("abc"), 3)
```
**EN:** Test that plain string 'abc' gives max length 3. This test exercises `test_literal_string` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that plain string 'abc' gives max length 3. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_literal_string`。

### Lines 341-343: test case character class / 测试用例 character class
```python
    def test_character_class(self):
        """Test that character class '[a-z]' gives max length 1."""
        self.assertEqual(get_max_seq_length("[a-z]"), 1)
```
**EN:** Test that character class '[a-z]' gives max length 1. This test exercises `test_character_class` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that character class '[a-z]' gives max length 1. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_character_class`。

### Lines 345-347: test case dot any / 测试用例 dot any
```python
    def test_dot_any(self):
        """Test that dot wildcard '.' gives max length 1."""
        self.assertEqual(get_max_seq_length("."), 1)
```
**EN:** Test that dot wildcard '.' gives max length 1. This test exercises `test_dot_any` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that dot wildcard '.' gives max length 1. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dot_any`。

### Lines 349-352: test case unbounded star / 测试用例 unbounded star
```python
    def test_unbounded_star(self):
        """Test that 'a*' (zero or more, no upper bound) returns MAX_LEN."""
        result = get_max_seq_length("a*")
        self.assertEqual(result, MAX_LEN)
```
**EN:** Test that 'a*' (zero or more, no upper bound) returns MAX_LEN. This test exercises `test_unbounded_star` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that 'a*' (zero or more, no upper bound) returns MAX_LEN. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unbounded_star`。

### Lines 354-357: test case unbounded plus / 测试用例 unbounded plus
```python
    def test_unbounded_plus(self):
        """Test that 'a+' (one or more, no upper bound) returns MAX_LEN."""
        result = get_max_seq_length("a+")
        self.assertEqual(result, MAX_LEN)
```
**EN:** Test that 'a+' (one or more, no upper bound) returns MAX_LEN. This test exercises `test_unbounded_plus` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that 'a+' (one or more, no upper bound) returns MAX_LEN. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unbounded_plus`。

### Lines 359-361: test case bounded repeat / 测试用例 bounded repeat
```python
    def test_bounded_repeat(self):
        """Test that exact repeat 'a{5}' gives max length 5."""
        self.assertEqual(get_max_seq_length("a{5}"), 5)
```
**EN:** Test that exact repeat 'a{5}' gives max length 5. This test exercises `test_bounded_repeat` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that exact repeat 'a{5}' gives max length 5. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bounded_repeat`。

### Lines 363-365: test case bounded range repeat / 测试用例 bounded range repeat
```python
    def test_bounded_range_repeat(self):
        """Test that range repeat 'a{2,4}' uses upper bound, giving max length 4."""
        self.assertEqual(get_max_seq_length("a{2,4}"), 4)
```
**EN:** Test that range repeat 'a{2,4}' uses upper bound, giving max length 4. This test exercises `test_bounded_range_repeat` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that range repeat 'a{2,4}' uses upper bound, giving max length 4. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bounded_range_repeat`。

### Lines 367-369: test case branch takes max / 测试用例 branch takes max
```python
    def test_branch_takes_max(self):
        """Test that alternation 'abc|de' takes the longer branch: max(3, 2) = 3."""
        self.assertEqual(get_max_seq_length("abc|de"), 3)
```
**EN:** Test that alternation 'abc|de' takes the longer branch: max(3, 2) = 3. This test exercises `test_branch_takes_max` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that alternation 'abc|de' takes the longer branch: max(3, 2) = 3. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_branch_takes_max`。

### Lines 371-373: test case subpattern group / 测试用例 subpattern group
```python
    def test_subpattern_group(self):
        """Test that capturing group '(abc)' gives max length 3 from inner content."""
        self.assertEqual(get_max_seq_length("(abc)"), 3)
```
**EN:** Test that capturing group '(abc)' gives max length 3 from inner content. This test exercises `test_subpattern_group` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that capturing group '(abc)' gives max length 3 from inner content. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_subpattern_group`。

### Lines 375-377: test case zero width assertions ignored / 测试用例 zero width assertions ignored
```python
    def test_zero_width_assertions_ignored(self):
        """Test that anchors ^ and $ in '^abc$' add 0, giving max length 3."""
        self.assertEqual(get_max_seq_length("^abc$"), 3)
```
**EN:** Test that anchors ^ and $ in '^abc$' add 0, giving max length 3. This test exercises `test_zero_width_assertions_ignored` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that anchors ^ and $ in '^abc$' add 0, giving max length 3. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_zero_width_assertions_ignored`。

### Lines 379-381: test case complex pattern / 测试用例 complex pattern
```python
    def test_complex_pattern(self):
        """Test combined pattern '(foo|bar)\\d{2}': branch(3) + repeat(2) = 5."""
        self.assertEqual(get_max_seq_length(r"(foo|bar)\d{2}"), 5)
```
**EN:** Test combined pattern '(foo|bar)\d{2}': branch(3) + repeat(2) = 5. This test exercises `test_complex_pattern` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test combined pattern '(foo|bar)\d{2}': branch(3) + repeat(2) = 5. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_complex_pattern`。

### Lines 383-385: test case nested groups / 测试用例 nested groups
```python
    def test_nested_groups(self):
        """Test that nested groups '((ab))' correctly recurse to give max length 2."""
        self.assertEqual(get_max_seq_length("((ab))"), 2)
```
**EN:** Test that nested groups '((ab))' correctly recurse to give max length 2. This test exercises `test_nested_groups` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that nested groups '((ab))' correctly recurse to give max length 2. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nested_groups`。

### Lines 387-389: test case question mark optional / 测试用例 question mark optional
```python
    def test_question_mark_optional(self):
        """Test that optional 'a?' (equivalent to a{0,1}) gives max length 1."""
        self.assertEqual(get_max_seq_length("a?"), 1)
```
**EN:** Test that optional 'a?' (equivalent to a{0,1}) gives max length 1. This test exercises `test_question_mark_optional` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that optional 'a?' (equivalent to a{0,1}) gives max length 1. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_question_mark_optional`。

### Lines 391-394: test case mixed unbounded and bounded / 测试用例 mixed unbounded and bounded
```python
    def test_mixed_unbounded_and_bounded(self):
        """Test that 'ab+c{3}' gives >= MAX_LEN because b+ is unbounded."""
        result = get_max_seq_length("ab+c{3}")
        self.assertGreaterEqual(result, MAX_LEN)
```
**EN:** Test that 'ab+c{3}' gives >= MAX_LEN because b+ is unbounded. This test exercises `test_mixed_unbounded_and_bounded` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that 'ab+c{3}' gives >= MAX_LEN because b+ is unbounded. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_unbounded_and_bounded`。

### Lines 396-398: test case empty regex / 测试用例 empty regex
```python
    def test_empty_regex(self):
        """Test that empty regex gives max length 0 (no tokens to match)."""
        self.assertEqual(get_max_seq_length(""), 0)
```
**EN:** Test that empty regex gives max length 0 (no tokens to match). This test exercises `test_empty_regex` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that empty regex gives max length 0 (no tokens to match). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_regex`。

### Lines 400-403: test case lookahead triggers unhandled token / 测试用例 lookahead triggers unhandled token
```python
    def test_lookahead_triggers_unhandled_token(self):
        """Test that lookahead (?=a) hits the unhandled-token fallback (MAX_LEN)."""
        result = get_max_seq_length("(?=a)b")
        self.assertGreaterEqual(result, MAX_LEN)
```
**EN:** Test that lookahead (?=a) hits the unhandled-token fallback (MAX_LEN). This test exercises `test_lookahead_triggers_unhandled_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that lookahead (?=a) hits the unhandled-token fallback (MAX_LEN). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lookahead_triggers_unhandled_token`。

### Lines 405-408: test case lookbehind triggers unhandled token / 测试用例 lookbehind triggers unhandled token
```python
    def test_lookbehind_triggers_unhandled_token(self):
        """Test that lookbehind (?<=x) hits the unhandled-token fallback (MAX_LEN)."""
        result = get_max_seq_length("(?<=x)y")
        self.assertGreaterEqual(result, MAX_LEN)
```
**EN:** Test that lookbehind (?<=x) hits the unhandled-token fallback (MAX_LEN). This test exercises `test_lookbehind_triggers_unhandled_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that lookbehind (?<=x) hits the unhandled-token fallback (MAX_LEN). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lookbehind_triggers_unhandled_token`。

### Lines 411-412: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestSamplingParamsInit`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSamplingParamsVerify`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSamplingParamsNormalize`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestRegexMaxLength`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSamplingParamsInit.test_zero_temperature_becomes_greedy`: Test greedy conversion when temperature is 0. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_zero_temperature_becomes_greedy`。
- `TestSamplingParamsInit.test_near_zero_temperature_becomes_greedy`: Test greedy conversion when temperature is near zero (1e-7). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_near_zero_temperature_becomes_greedy`。
- `TestSamplingParamsInit.test_temperature_at_eps_boundary_not_greedy`: Test that temperature exactly at 1e-6 does not trigger greedy (strict <). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_temperature_at_eps_boundary_not_greedy`。
- `TestSamplingParamsInit.test_negative_temperature_not_modified`: Test that __init__ preserves negative temperature (rejected by verify instead). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_negative_temperature_not_modified`。
- `TestSamplingParamsInit.test_top_k_minus_one_becomes_top_k_all`: Test that top_k=-1 is converted to TOP_K_ALL (whole vocabulary). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_top_k_minus_one_becomes_top_k_all`。
- `TestSamplingParamsInit.test_positive_top_k_preserved`: Test that explicit positive top_k is kept as-is. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_positive_top_k_preserved`。
- `TestSamplingParamsInit.test_stop_token_ids_stored_as_set`: Test that stop_token_ids list is converted to set. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stop_token_ids_stored_as_set`。
- `TestSamplingParamsInit.test_stop_token_ids_none_stays_none`: Test that None stop_token_ids stays None. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_stop_token_ids_none_stays_none`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.sampling.sampling_params`, `sglang.test.test_utils`

- **Total lines / 总行数**: 412
