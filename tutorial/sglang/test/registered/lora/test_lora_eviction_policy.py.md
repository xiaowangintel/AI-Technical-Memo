# test_lora_eviction_policy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/lora/test_lora_eviction_policy.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates lora eviction policy behavior in SGLang's lora area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 lora 领域中与 lora eviction policy 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: supporting source context / 辅助源码上下文
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 15-18: supporting statements / 辅助语句
```python
"""
Unit tests for LoRA eviction policies.
Tests LRU and FIFO eviction behavior.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 20-23: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.lora.eviction_policy import get_eviction_policy
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.lora.eviction_policy`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.lora.eviction_policy`, `sglang.test.ci.ci_register`。

### Lines 25-26: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=200, suite="nightly-1-gpu", nightly=True)
register_amd_ci(est_time=200, suite="nightly-amd-1-gpu", nightly=True)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 29-29: class TestLoRAEvictionPolicy declaration / 类 TestLoRAEvictionPolicy 声明
```python
class TestLoRAEvictionPolicy(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 30-30: supporting statements / 辅助语句
```python
    """Unit tests for LoRA eviction policies."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 32-56: method test eviction policy / 方法 test eviction policy
```python
    def _test_eviction_policy(
        self, policy_name, access_sequence, candidates, expected_victim
    ):
        """
        Helper to test eviction policy with given access pattern.

        Args:
            policy_name: Name of eviction policy ("lru" or "fifo")
            access_sequence: List of adapter IDs in access order
            candidates: Set of adapter IDs that can be evicted
            expected_victim: Expected adapter ID to be evicted
        """
        policy = get_eviction_policy(policy_name)

        # Simulate access pattern
        for adapter_id in access_sequence:
            policy.mark_used(adapter_id)

        # Select victim from candidates
        victim = policy.select_victim(candidates)
        self.assertEqual(
            victim,
            expected_victim,
            f"{policy_name.upper()}: Expected {expected_victim}, got {victim}",
        )
```
**EN:** Helper to test eviction policy with given access pattern. This block implements `_test_eviction_policy` and captures one focused piece of the module's behavior.
**CN:** Helper to test eviction policy with given access pattern. 该代码块实现 `_test_eviction_policy`，承担模块行为中的一个聚焦逻辑片段。

### Lines 58-65: test case lru basic / 测试用例 lru basic
```python
    def test_lru_basic(self):
        """Test LRU selects least recently used adapter."""
        self._test_eviction_policy(
            "lru",
            access_sequence=["lora1", "lora2", "lora3", "lora4"],
            candidates={"lora1", "lora2", "lora3", "lora4"},
            expected_victim="lora1",
        )
```
**EN:** Test LRU selects least recently used adapter. This test exercises `test_lru_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test LRU selects least recently used adapter. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lru_basic`。

### Lines 67-74: test case lru with reuse / 测试用例 lru with reuse
```python
    def test_lru_with_reuse(self):
        """Test LRU updates order on reuse."""
        self._test_eviction_policy(
            "lru",
            access_sequence=["lora1", "lora2", "lora3", "lora4", "lora1"],
            candidates={"lora1", "lora2", "lora3", "lora4"},
            expected_victim="lora2",
        )
```
**EN:** Test LRU updates order on reuse. This test exercises `test_lru_with_reuse` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test LRU updates order on reuse. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lru_with_reuse`。

### Lines 76-83: test case lru multiple reuse / 测试用例 lru multiple reuse
```python
    def test_lru_multiple_reuse(self):
        """Test LRU with multiple reuses."""
        self._test_eviction_policy(
            "lru",
            access_sequence=["lora1", "lora2", "lora3", "lora1", "lora2"],
            candidates={"lora1", "lora2", "lora3"},
            expected_victim="lora3",
        )
```
**EN:** Test LRU with multiple reuses. This test exercises `test_lru_multiple_reuse` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test LRU with multiple reuses. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lru_multiple_reuse`。

### Lines 85-92: test case lru with subset candidates / 测试用例 lru with subset candidates
```python
    def test_lru_with_subset_candidates(self):
        """Test LRU with subset of candidates."""
        self._test_eviction_policy(
            "lru",
            access_sequence=["lora1", "lora2", "lora3", "lora4"],
            candidates={"lora2", "lora3", "lora4"},
            expected_victim="lora2",
        )
```
**EN:** Test LRU with subset of candidates. This test exercises `test_lru_with_subset_candidates` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test LRU with subset of candidates. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lru_with_subset_candidates`。

### Lines 94-101: test case lru base model evicted last / 测试用例 lru base model evicted last
```python
    def test_lru_base_model_evicted_last(self):
        """Test LRU evicts LoRA adapters before base model (None)."""
        self._test_eviction_policy(
            "lru",
            access_sequence=["lora1", "lora2", "lora3"],
            candidates={None, "lora1", "lora2", "lora3"},
            expected_victim="lora1",
        )
```
**EN:** Test LRU evicts LoRA adapters before base model (None). This test exercises `test_lru_base_model_evicted_last` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test LRU evicts LoRA adapters before base model (None). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lru_base_model_evicted_last`。

### Lines 103-110: test case fifo basic / 测试用例 fifo basic
```python
    def test_fifo_basic(self):
        """Test FIFO selects first inserted adapter."""
        self._test_eviction_policy(
            "fifo",
            access_sequence=["lora1", "lora2", "lora3", "lora4"],
            candidates={"lora1", "lora2", "lora3", "lora4"},
            expected_victim="lora1",
        )
```
**EN:** Test FIFO selects first inserted adapter. This test exercises `test_fifo_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test FIFO selects first inserted adapter. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fifo_basic`。

### Lines 112-128: test case fifo ignores reuse / 测试用例 fifo ignores reuse
```python
    def test_fifo_ignores_reuse(self):
        """Test FIFO ignores reuse."""
        self._test_eviction_policy(
            "fifo",
            access_sequence=[
                "lora1",
                "lora2",
                "lora3",
                "lora4",
                "lora4",
                "lora3",
                "lora2",
                "lora1",
            ],
            candidates={"lora1", "lora2", "lora3", "lora4"},
            expected_victim="lora1",
        )
```
**EN:** Test FIFO ignores reuse. This test exercises `test_fifo_ignores_reuse` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test FIFO ignores reuse. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fifo_ignores_reuse`。

### Lines 130-137: test case fifo with subset candidates / 测试用例 fifo with subset candidates
```python
    def test_fifo_with_subset_candidates(self):
        """Test FIFO with subset of candidates."""
        self._test_eviction_policy(
            "fifo",
            access_sequence=["lora1", "lora2", "lora3", "lora4"],
            candidates={"lora2", "lora3", "lora4"},
            expected_victim="lora2",
        )
```
**EN:** Test FIFO with subset of candidates. This test exercises `test_fifo_with_subset_candidates` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test FIFO with subset of candidates. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fifo_with_subset_candidates`。

### Lines 139-146: test case fifo base model evicted last / 测试用例 fifo base model evicted last
```python
    def test_fifo_base_model_evicted_last(self):
        """Test FIFO evicts LoRA adapters before base model (None)."""
        self._test_eviction_policy(
            "fifo",
            access_sequence=["lora1", "lora2", "lora3"],
            candidates={None, "lora1", "lora2", "lora3"},
            expected_victim="lora1",
        )
```
**EN:** Test FIFO evicts LoRA adapters before base model (None). This test exercises `test_fifo_base_model_evicted_last` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test FIFO evicts LoRA adapters before base model (None). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fifo_base_model_evicted_last`。

### Lines 148-158: test case policy remove / 测试用例 policy remove
```python
    def test_policy_remove(self):
        """Test that remove() correctly updates internal state."""
        lru = get_eviction_policy("lru")
        lru.mark_used("lora1")
        lru.mark_used("lora2")
        lru.mark_used("lora3")

        # Remove lora1, so lora2 becomes LRU
        lru.remove("lora1")
        victim = lru.select_victim({"lora1", "lora2", "lora3"})
        self.assertEqual(victim, "lora2")
```
**EN:** Test that remove() correctly updates internal state. This test exercises `test_policy_remove` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that remove() correctly updates internal state. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_policy_remove`。

### Lines 160-171: test case eviction policy factory / 测试用例 eviction policy factory
```python
    def test_eviction_policy_factory(self):
        """Test eviction policy factory function."""
        # Test valid policies
        lru = get_eviction_policy("lru")
        fifo = get_eviction_policy("fifo")

        self.assertIsNotNone(lru)
        self.assertIsNotNone(fifo)

        # Test invalid policy
        with self.assertRaises(ValueError):
            get_eviction_policy("invalid_policy")
```
**EN:** Test eviction policy factory function. This test exercises `test_eviction_policy_factory` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test eviction policy factory function. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_eviction_policy_factory`。

### Lines 173-190: test case lru vs fifo behavior / 测试用例 lru vs fifo behavior
```python
    def test_lru_vs_fifo_behavior(self):
        """Test that LRU and FIFO behave differently."""
        access_sequence = ["lora1", "lora2", "lora3", "lora1"]
        candidates = {"lora1", "lora2", "lora3"}

        lru = get_eviction_policy("lru")
        for adapter_id in access_sequence:
            lru.mark_used(adapter_id)
        lru_victim = lru.select_victim(candidates)

        fifo = get_eviction_policy("fifo")
        for adapter_id in access_sequence:
            fifo.mark_used(adapter_id)
        fifo_victim = fifo.select_victim(candidates)

        self.assertNotEqual(lru_victim, fifo_victim)
        self.assertEqual(lru_victim, "lora2")
        self.assertEqual(fifo_victim, "lora1")
```
**EN:** Test that LRU and FIFO behave differently. This test exercises `test_lru_vs_fifo_behavior` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that LRU and FIFO behave differently. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lru_vs_fifo_behavior`。

### Lines 193-194: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestLoRAEvictionPolicy`: Unit tests for LoRA eviction policies. / 用于组织相关测试、夹具或辅助方法。
- `TestLoRAEvictionPolicy._test_eviction_policy`: Helper to test eviction policy with given access pattern. / 该代码块实现 `_test_eviction_policy`，承担模块行为中的一个聚焦逻辑片段。
- `TestLoRAEvictionPolicy.test_lru_basic`: Test LRU selects least recently used adapter. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lru_basic`。
- `TestLoRAEvictionPolicy.test_lru_with_reuse`: Test LRU updates order on reuse. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lru_with_reuse`。
- `TestLoRAEvictionPolicy.test_lru_multiple_reuse`: Test LRU with multiple reuses. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lru_multiple_reuse`。
- `TestLoRAEvictionPolicy.test_lru_with_subset_candidates`: Test LRU with subset of candidates. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lru_with_subset_candidates`。
- `TestLoRAEvictionPolicy.test_lru_base_model_evicted_last`: Test LRU evicts LoRA adapters before base model (None). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lru_base_model_evicted_last`。
- `TestLoRAEvictionPolicy.test_fifo_basic`: Test FIFO selects first inserted adapter. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fifo_basic`。
- `TestLoRAEvictionPolicy.test_fifo_ignores_reuse`: Test FIFO ignores reuse. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fifo_ignores_reuse`。
- `TestLoRAEvictionPolicy.test_fifo_with_subset_candidates`: Test FIFO with subset of candidates. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fifo_with_subset_candidates`。
- `TestLoRAEvictionPolicy.test_fifo_base_model_evicted_last`: Test FIFO evicts LoRA adapters before base model (None). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fifo_base_model_evicted_last`。
- `TestLoRAEvictionPolicy.test_policy_remove`: Test that remove() correctly updates internal state. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_policy_remove`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.lora.eviction_policy`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 194
