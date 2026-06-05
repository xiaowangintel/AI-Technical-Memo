# test_evict_policy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/mem_cache/test_evict_policy.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates evict policy behavior in SGLang's unit / mem cache area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 内存缓存 领域中与 evict policy 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for evict_policy.py"""
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
register_cpu_ci(est_time=6, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 7-18: module imports and dependencies / 模块导入与依赖
```python
import unittest
from unittest.mock import MagicMock

from sglang.srt.mem_cache.evict_policy import (
    FIFOStrategy,
    FILOStrategy,
    LFUStrategy,
    LRUStrategy,
    MRUStrategy,
    PriorityStrategy,
    SLRUStrategy,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `unittest.mock`, `sglang.srt.mem_cache.evict_policy`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `unittest.mock`, `sglang.srt.mem_cache.evict_policy`。

### Lines 21-27: function make node / 函数 make node
```python
def _make_node(**kwargs):
    node = MagicMock()
    node.last_access_time = kwargs.get("last_access_time", 0.0)
    node.hit_count = kwargs.get("hit_count", 0)
    node.creation_time = kwargs.get("creation_time", 0.0)
    node.priority = kwargs.get("priority", 0)
    return node
```
**EN:** This block implements `_make_node` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_node`，承担模块行为中的一个聚焦逻辑片段。

### Lines 30-30: class TestLRUStrategy declaration / 类 TestLRUStrategy 声明
```python
class TestLRUStrategy(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 31-32: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.strategy = LRUStrategy()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 34-36: test case priority is last access time / 测试用例 priority is last access time
```python
    def test_priority_is_last_access_time(self):
        node = _make_node(last_access_time=42.0)
        self.assertEqual(self.strategy.get_priority(node), 42.0)
```
**EN:** This test exercises `test_priority_is_last_access_time` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_is_last_access_time`。

### Lines 38-43: test case older access evicted first / 测试用例 older access evicted first
```python
    def test_older_access_evicted_first(self):
        old = _make_node(last_access_time=1.0)
        new = _make_node(last_access_time=10.0)
        self.assertLess(
            self.strategy.get_priority(old), self.strategy.get_priority(new)
        )
```
**EN:** This test exercises `test_older_access_evicted_first` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_older_access_evicted_first`。

### Lines 46-46: class TestLFUStrategy declaration / 类 TestLFUStrategy 声明
```python
class TestLFUStrategy(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 47-48: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.strategy = LFUStrategy()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 50-52: test case priority is hit count and time / 测试用例 priority is hit count and time
```python
    def test_priority_is_hit_count_and_time(self):
        node = _make_node(hit_count=5, last_access_time=3.0)
        self.assertEqual(self.strategy.get_priority(node), (5, 3.0))
```
**EN:** This test exercises `test_priority_is_hit_count_and_time` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_is_hit_count_and_time`。

### Lines 54-59: test case lower hit count evicted first / 测试用例 lower hit count evicted first
```python
    def test_lower_hit_count_evicted_first(self):
        cold = _make_node(hit_count=1, last_access_time=10.0)
        hot = _make_node(hit_count=100, last_access_time=1.0)
        self.assertLess(
            self.strategy.get_priority(cold), self.strategy.get_priority(hot)
        )
```
**EN:** This test exercises `test_lower_hit_count_evicted_first` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lower_hit_count_evicted_first`。

### Lines 61-66: test case same hit count older access evicted first / 测试用例 same hit count older access evicted first
```python
    def test_same_hit_count_older_access_evicted_first(self):
        old = _make_node(hit_count=3, last_access_time=1.0)
        new = _make_node(hit_count=3, last_access_time=10.0)
        self.assertLess(
            self.strategy.get_priority(old), self.strategy.get_priority(new)
        )
```
**EN:** This test exercises `test_same_hit_count_older_access_evicted_first` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_same_hit_count_older_access_evicted_first`。

### Lines 69-69: class TestFIFOStrategy declaration / 类 TestFIFOStrategy 声明
```python
class TestFIFOStrategy(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 70-71: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.strategy = FIFOStrategy()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 73-75: test case priority is creation time / 测试用例 priority is creation time
```python
    def test_priority_is_creation_time(self):
        node = _make_node(creation_time=7.0)
        self.assertEqual(self.strategy.get_priority(node), 7.0)
```
**EN:** This test exercises `test_priority_is_creation_time` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_is_creation_time`。

### Lines 77-82: test case earlier created evicted first / 测试用例 earlier created evicted first
```python
    def test_earlier_created_evicted_first(self):
        first = _make_node(creation_time=1.0)
        second = _make_node(creation_time=5.0)
        self.assertLess(
            self.strategy.get_priority(first), self.strategy.get_priority(second)
        )
```
**EN:** This test exercises `test_earlier_created_evicted_first` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_earlier_created_evicted_first`。

### Lines 85-85: class TestMRUStrategy declaration / 类 TestMRUStrategy 声明
```python
class TestMRUStrategy(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 86-87: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.strategy = MRUStrategy()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 89-91: test case priority is negated access time / 测试用例 priority is negated access time
```python
    def test_priority_is_negated_access_time(self):
        node = _make_node(last_access_time=5.0)
        self.assertEqual(self.strategy.get_priority(node), -5.0)
```
**EN:** This test exercises `test_priority_is_negated_access_time` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_is_negated_access_time`。

### Lines 93-99: test case most recently used evicted first / 测试用例 most recently used evicted first
```python
    def test_most_recently_used_evicted_first(self):
        """MRU evicts the most recently accessed node first (lowest priority value)."""
        old = _make_node(last_access_time=1.0)
        new = _make_node(last_access_time=10.0)
        self.assertLess(
            self.strategy.get_priority(new), self.strategy.get_priority(old)
        )
```
**EN:** MRU evicts the most recently accessed node first (lowest priority value). This test exercises `test_most_recently_used_evicted_first` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** MRU evicts the most recently accessed node first (lowest priority value). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_most_recently_used_evicted_first`。

### Lines 102-102: class TestFILOStrategy declaration / 类 TestFILOStrategy 声明
```python
class TestFILOStrategy(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 103-104: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.strategy = FILOStrategy()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 106-108: test case priority is negated creation time / 测试用例 priority is negated creation time
```python
    def test_priority_is_negated_creation_time(self):
        node = _make_node(creation_time=3.0)
        self.assertEqual(self.strategy.get_priority(node), -3.0)
```
**EN:** This test exercises `test_priority_is_negated_creation_time` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_is_negated_creation_time`。

### Lines 110-116: test case last created evicted first / 测试用例 last created evicted first
```python
    def test_last_created_evicted_first(self):
        """FILO evicts the most recently created node first."""
        first = _make_node(creation_time=1.0)
        second = _make_node(creation_time=5.0)
        self.assertLess(
            self.strategy.get_priority(second), self.strategy.get_priority(first)
        )
```
**EN:** FILO evicts the most recently created node first. This test exercises `test_last_created_evicted_first` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** FILO evicts the most recently created node first. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_last_created_evicted_first`。

### Lines 119-119: class TestPriorityStrategy declaration / 类 TestPriorityStrategy 声明
```python
class TestPriorityStrategy(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 120-121: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.strategy = PriorityStrategy()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 123-125: test case priority is tuple / 测试用例 priority is tuple
```python
    def test_priority_is_tuple(self):
        node = _make_node(priority=2, last_access_time=4.0)
        self.assertEqual(self.strategy.get_priority(node), (2, 4.0))
```
**EN:** This test exercises `test_priority_is_tuple` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_is_tuple`。

### Lines 127-132: test case lower priority evicted first / 测试用例 lower priority evicted first
```python
    def test_lower_priority_evicted_first(self):
        low = _make_node(priority=1, last_access_time=10.0)
        high = _make_node(priority=5, last_access_time=1.0)
        self.assertLess(
            self.strategy.get_priority(low), self.strategy.get_priority(high)
        )
```
**EN:** This test exercises `test_lower_priority_evicted_first` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lower_priority_evicted_first`。

### Lines 134-139: test case same priority older access evicted first / 测试用例 same priority older access evicted first
```python
    def test_same_priority_older_access_evicted_first(self):
        old = _make_node(priority=3, last_access_time=1.0)
        new = _make_node(priority=3, last_access_time=10.0)
        self.assertLess(
            self.strategy.get_priority(old), self.strategy.get_priority(new)
        )
```
**EN:** This test exercises `test_same_priority_older_access_evicted_first` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_same_priority_older_access_evicted_first`。

### Lines 142-142: class TestSLRUStrategy declaration / 类 TestSLRUStrategy 声明
```python
class TestSLRUStrategy(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 143-144: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.strategy = SLRUStrategy(protected_threshold=2)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 146-148: test case probationary segment / 测试用例 probationary segment
```python
    def test_probationary_segment(self):
        node = _make_node(hit_count=1, last_access_time=5.0)
        self.assertEqual(self.strategy.get_priority(node), (0, 5.0))
```
**EN:** This test exercises `test_probationary_segment` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_probationary_segment`。

### Lines 150-152: test case protected segment / 测试用例 protected segment
```python
    def test_protected_segment(self):
        node = _make_node(hit_count=2, last_access_time=5.0)
        self.assertEqual(self.strategy.get_priority(node), (1, 5.0))
```
**EN:** This test exercises `test_protected_segment` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_protected_segment`。

### Lines 154-156: test case highly accessed is protected / 测试用例 highly accessed is protected
```python
    def test_highly_accessed_is_protected(self):
        node = _make_node(hit_count=100, last_access_time=5.0)
        self.assertEqual(self.strategy.get_priority(node), (1, 5.0))
```
**EN:** This test exercises `test_highly_accessed_is_protected` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_highly_accessed_is_protected`。

### Lines 158-163: test case probationary evicted before protected / 测试用例 probationary evicted before protected
```python
    def test_probationary_evicted_before_protected(self):
        prob = _make_node(hit_count=1, last_access_time=10.0)
        prot = _make_node(hit_count=5, last_access_time=1.0)
        self.assertLess(
            self.strategy.get_priority(prob), self.strategy.get_priority(prot)
        )
```
**EN:** This test exercises `test_probationary_evicted_before_protected` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_probationary_evicted_before_protected`。

### Lines 165-170: test case same segment older access evicted first / 测试用例 same segment older access evicted first
```python
    def test_same_segment_older_access_evicted_first(self):
        old = _make_node(hit_count=0, last_access_time=1.0)
        new = _make_node(hit_count=0, last_access_time=10.0)
        self.assertLess(
            self.strategy.get_priority(old), self.strategy.get_priority(new)
        )
```
**EN:** This test exercises `test_same_segment_older_access_evicted_first` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_same_segment_older_access_evicted_first`。

### Lines 172-177: test case custom threshold / 测试用例 custom threshold
```python
    def test_custom_threshold(self):
        strategy = SLRUStrategy(protected_threshold=5)
        below = _make_node(hit_count=4, last_access_time=1.0)
        at = _make_node(hit_count=5, last_access_time=1.0)
        self.assertEqual(strategy.get_priority(below), (0, 1.0))
        self.assertEqual(strategy.get_priority(at), (1, 1.0))
```
**EN:** This test exercises `test_custom_threshold` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_threshold`。

### Lines 179-181: test case default threshold is 2 / 测试用例 default threshold is 2
```python
    def test_default_threshold_is_2(self):
        default = SLRUStrategy()
        self.assertEqual(default.protected_threshold, 2)
```
**EN:** This test exercises `test_default_threshold_is_2` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_threshold_is_2`。

### Lines 184-184: class TestEvictionOrdering declaration / 类 TestEvictionOrdering 声明
```python
class TestEvictionOrdering(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 185-185: supporting statements / 辅助语句
```python
    """Integration-style test: sort a list of nodes by eviction priority."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 187-196: test case lru ordering / 测试用例 lru ordering
```python
    def test_lru_ordering(self):
        strategy = LRUStrategy()
        nodes = [
            _make_node(last_access_time=5.0),
            _make_node(last_access_time=1.0),
            _make_node(last_access_time=3.0),
        ]
        eviction_order = sorted(nodes, key=strategy.get_priority)
        times = [n.last_access_time for n in eviction_order]
        self.assertEqual(times, [1.0, 3.0, 5.0])
```
**EN:** This test exercises `test_lru_ordering` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lru_ordering`。

### Lines 198-214: test case slru ordering / 测试用例 slru ordering
```python
    def test_slru_ordering(self):
        strategy = SLRUStrategy(protected_threshold=2)
        nodes = [
            _make_node(hit_count=5, last_access_time=1.0),  # protected, old
            _make_node(hit_count=0, last_access_time=10.0),  # probationary, new
            _make_node(hit_count=0, last_access_time=2.0),  # probationary, old
            _make_node(hit_count=3, last_access_time=8.0),  # protected, new
        ]
        eviction_order = sorted(nodes, key=strategy.get_priority)
        expected = [
            (0, 2.0),  # probationary old
            (0, 10.0),  # probationary new
            (1, 1.0),  # protected old
            (1, 8.0),  # protected new
        ]
        actual = [strategy.get_priority(n) for n in eviction_order]
        self.assertEqual(actual, expected)
```
**EN:** This test exercises `test_slru_ordering` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_slru_ordering`。

### Lines 217-218: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_node`: This block implements `_make_node` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_node`，承担模块行为中的一个聚焦逻辑片段。
- `TestLRUStrategy`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLFUStrategy`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFIFOStrategy`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMRUStrategy`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFILOStrategy`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestPriorityStrategy`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSLRUStrategy`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLRUStrategy.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestLRUStrategy.test_priority_is_last_access_time`: This test exercises `test_priority_is_last_access_time` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_priority_is_last_access_time`。
- `TestLRUStrategy.test_older_access_evicted_first`: This test exercises `test_older_access_evicted_first` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_older_access_evicted_first`。
- `TestLFUStrategy.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `unittest.mock`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.mem_cache.evict_policy`

- **Total lines / 总行数**: 218
