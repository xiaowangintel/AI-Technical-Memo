# test_schedule_policy.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_schedule_policy.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `schedule policy` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual` 中的 `schedule policy` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and shared helpers / 导入与共享辅助项
```python
import unittest

from sglang.srt.managers.schedule_batch import Req, ScheduleBatch
from sglang.srt.managers.schedule_policy import (
    CacheAgnosticPolicy,
    CacheAwarePolicy,
    SchedulePolicy,
)
from sglang.srt.mem_cache.radix_cache import RadixCache
from sglang.srt.sampling.sampling_params import SamplingParams
from sglang.test.test_utils import CustomTestCase
```
**EN:** This range imports `unittest`, `sglang.srt.managers.schedule_batch`, `sglang.srt.managers.schedule_policy` and `sglang.srt.mem_cache.radix_cache`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 14-15: Class definition for TestSchedulePolicy / 类定义
```python
class TestSchedulePolicy(CustomTestCase):
```
**EN:** This range declares `TestSchedulePolicy`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 16-17: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUp(self):
        self.tree_cache = RadixCache.create_simulated()
```
**EN:** This range implements lifecycle helper(s) `setUp` to prepare or clean up shared resources across the test flow. Representative call sites include `create_simulated`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 19-27: Test routines around test_init_with_cache_aware_policy / 测试例程
```python
    def test_init_with_cache_aware_policy(self):
        policy = SchedulePolicy(
            policy="lpm",
            tree_cache=self.tree_cache,
            enable_hierarchical_cache=True,
            enable_priority_scheduling=False,
            schedule_low_priority_values_first=False,
        )
        self.assertEqual(policy.policy, CacheAwarePolicy.LPM)
```
**EN:** This range defines concrete test routine(s) `test_init_with_cache_aware_policy`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `SchedulePolicy` and `assertEqual`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 29-37: Test routines around test_init_with_cache_agnostic_policy / 测试例程
```python
    def test_init_with_cache_agnostic_policy(self):
        policy = SchedulePolicy(
            policy="fcfs",
            tree_cache=self.tree_cache,
            enable_hierarchical_cache=True,
            enable_priority_scheduling=False,
            schedule_low_priority_values_first=False,
        )
        self.assertEqual(policy.policy, CacheAgnosticPolicy.FCFS)
```
**EN:** This range defines concrete test routine(s) `test_init_with_cache_agnostic_policy`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `SchedulePolicy` and `assertEqual`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 39-47: Test routines around test_init_with_unknown_policy / 测试例程
```python
    def test_init_with_unknown_policy(self):
        with self.assertRaises(ValueError):
            SchedulePolicy(
                policy="invalid",
                tree_cache=self.tree_cache,
                enable_hierarchical_cache=True,
                enable_priority_scheduling=False,
                schedule_low_priority_values_first=False,
            )
```
**EN:** This range defines concrete test routine(s) `test_init_with_unknown_policy`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `assertRaises` and `SchedulePolicy`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 49-58: Test routines around test_init_with_disabled_cache / 测试例程
```python
    def test_init_with_disabled_cache(self):
        tree_cache = RadixCache.create_simulated(disable=True)
        policy = SchedulePolicy(
            policy="lpm",
            tree_cache=tree_cache,
            enable_hierarchical_cache=True,
            enable_priority_scheduling=False,
            schedule_low_priority_values_first=False,
        )
        self.assertEqual(policy.policy, CacheAgnosticPolicy.FCFS)
```
**EN:** This range defines concrete test routine(s) `test_init_with_disabled_cache`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `create_simulated`, `SchedulePolicy` and `assertEqual`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 60-63: Test routines around test_calc_priority_fcfs / 测试例程
```python
    def test_calc_priority_fcfs(self):
        tree_cache = RadixCache.create_simulated()
        waiting_queue = [
            Req(1, "a b", [1, 2], SamplingParams()),
```
**EN:** This range defines concrete test routine(s) `test_calc_priority_fcfs`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `create_simulated`, `Req` and `SamplingParams`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 64-79: Assertions and result checks / 断言与结果检查
```python
            Req(3, "a b c", [1, 2, 3], SamplingParams()),
            Req(2, "a", [1], SamplingParams()),
        ]

        policy = SchedulePolicy(
            policy="fcfs",
            tree_cache=tree_cache,
            enable_hierarchical_cache=True,
            enable_priority_scheduling=False,
            schedule_low_priority_values_first=False,
        )
        policy.calc_priority(waiting_queue)
        # Check if FCFS keeps the original order
        self.assertEqual(waiting_queue[0].rid, 1)
        self.assertEqual(waiting_queue[1].rid, 3)
        self.assertEqual(waiting_queue[2].rid, 2)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `Req`, `SamplingParams`, `SchedulePolicy` and `calc_priority`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 81-88: Test routines around test_calc_priority_priority_enabled_fcfs_scheduling / 测试例程
```python
    def test_calc_priority_priority_enabled_fcfs_scheduling(self):
        tree_cache = RadixCache.create_simulated()
        r1 = Req(1, "a b", [1, 2], SamplingParams())
        r2 = Req(3, "a b c", [1, 2, 3], SamplingParams())
        r3 = Req(2, "a", [1], SamplingParams())
        r1.priority, r1.time_stats.wait_queue_entry_time = 1, 1
        r2.priority, r2.time_stats.wait_queue_entry_time = 0, 1
        r3.priority, r3.time_stats.wait_queue_entry_time = 0, 0
```
**EN:** This range defines concrete test routine(s) `test_calc_priority_priority_enabled_fcfs_scheduling`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `create_simulated`, `Req` and `SamplingParams`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 89-104: Assertions and result checks / 断言与结果检查
```python

        waiting_queue = [r1, r2, r3]

        policy = SchedulePolicy(
            policy="fcfs",
            tree_cache=tree_cache,
            enable_hierarchical_cache=True,
            enable_priority_scheduling=True,
            schedule_low_priority_values_first=False,
        )
        policy.calc_priority(waiting_queue)

        # Check if priority enabled fcfs ordering is applied.
        self.assertEqual(waiting_queue[0].rid, 1)
        self.assertEqual(waiting_queue[1].rid, 2)
        self.assertEqual(waiting_queue[2].rid, 3)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `SchedulePolicy`, `calc_priority` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 106-113: Test routines around test_calc_priority_priority_enabled_fcfs_scheduling_with_low_priority_values_first / 测试例程
```python
    def test_calc_priority_priority_enabled_fcfs_scheduling_with_low_priority_values_first(
        self,
    ):
        tree_cache = RadixCache.create_simulated()
        r1 = Req(1, "a b", [1, 2], SamplingParams())
        r2 = Req(3, "a b c", [1, 2, 3], SamplingParams())
        r3 = Req(2, "a", [1], SamplingParams())
        r1.priority, r1.time_stats.wait_queue_entry_time = -1, 1
```
**EN:** This range defines concrete test routine(s) `test_calc_priority_priority_enabled_fcfs_scheduling_with_low_priority_values_first`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `create_simulated`, `Req` and `SamplingParams`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 114-130: Assertions and result checks / 断言与结果检查
```python
        r2.priority, r2.time_stats.wait_queue_entry_time = 0, 1
        r3.priority, r3.time_stats.wait_queue_entry_time = 0, 0

        waiting_queue = [r1, r2, r3]

        policy = SchedulePolicy(
            policy="fcfs",
            tree_cache=tree_cache,
            enable_hierarchical_cache=True,
            enable_priority_scheduling=True,
            schedule_low_priority_values_first=True,
        )
        policy.calc_priority(waiting_queue)
        # Check if priority enabled fcfs ordering is applied.
        self.assertEqual(waiting_queue[0].rid, 1)
        self.assertEqual(waiting_queue[1].rid, 2)
        self.assertEqual(waiting_queue[2].rid, 3)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `SchedulePolicy`, `calc_priority` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 132-138: Test routines around test_calc_priority_longest_output_first_scheduling / 测试例程
```python
    def test_calc_priority_longest_output_first_scheduling(self):
        tree_cache = RadixCache.create_simulated()

        waiting_queue = [
            Req(1, "a b", [1, 2], SamplingParams(max_new_tokens=1000)),
            Req(3, "a b c", [1, 2, 3], SamplingParams(max_new_tokens=10)),
            Req(2, "a", [1], SamplingParams(max_new_tokens=100)),
```
**EN:** This range defines concrete test routine(s) `test_calc_priority_longest_output_first_scheduling`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `create_simulated`, `Req` and `SamplingParams`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 139-152: Assertions and result checks / 断言与结果检查
```python
        ]

        policy = SchedulePolicy(
            policy="lof",
            tree_cache=tree_cache,
            enable_hierarchical_cache=True,
            enable_priority_scheduling=False,
            schedule_low_priority_values_first=False,
        )
        policy.calc_priority(waiting_queue)
        # Check if priority enabled fcfs ordering is applied.
        self.assertEqual(waiting_queue[0].rid, 1)
        self.assertEqual(waiting_queue[1].rid, 2)
        self.assertEqual(waiting_queue[2].rid, 3)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `SchedulePolicy`, `calc_priority` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 154-163: Test routines around test_calc_priority_priority_enabled_longest_output_first_scheduling / 测试例程
```python
    def test_calc_priority_priority_enabled_longest_output_first_scheduling(self):
        tree_cache = RadixCache.create_simulated()

        waiting_queue = [
            Req(1, "a b", [1, 2], SamplingParams(max_new_tokens=1), priority=1),
            Req(3, "a b c", [1, 2, 3], SamplingParams(max_new_tokens=10), priority=0),
            Req(2, "a", [1], SamplingParams(max_new_tokens=100), priority=0),
        ]

        policy = SchedulePolicy(
```
**EN:** This range defines concrete test routine(s) `test_calc_priority_priority_enabled_longest_output_first_scheduling`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `create_simulated`, `Req`, `SamplingParams` and `SchedulePolicy`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 164-174: Assertions and result checks / 断言与结果检查
```python
            policy="lof",
            tree_cache=tree_cache,
            enable_hierarchical_cache=True,
            enable_priority_scheduling=True,
            schedule_low_priority_values_first=False,
        )
        policy.calc_priority(waiting_queue)
        # Check if priority enabled fcfs ordering is applied.
        self.assertEqual(waiting_queue[0].rid, 1)
        self.assertEqual(waiting_queue[1].rid, 2)
        self.assertEqual(waiting_queue[2].rid, 3)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `calc_priority` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 176-188: Test routines around test_calc_priority_priority_enabled_longest_output_first_scheduling_with_low_priority_values_first / 测试例程
```python
    def test_calc_priority_priority_enabled_longest_output_first_scheduling_with_low_priority_values_first(
        self,
    ):
        tree_cache = RadixCache.create_simulated()

        waiting_queue = [
            Req(1, "a b", [1, 2], SamplingParams(max_new_tokens=1), priority=0),
            Req(3, "a b c", [1, 2, 3], SamplingParams(max_new_tokens=10), priority=1),
            Req(2, "a", [1], SamplingParams(max_new_tokens=100), priority=1),
        ]

        policy = SchedulePolicy(
            policy="lof",
```
**EN:** This range defines concrete test routine(s) `test_calc_priority_priority_enabled_longest_output_first_scheduling_with_low_priority_values_first`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `create_simulated`, `Req`, `SamplingParams` and `SchedulePolicy`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 189-198: Assertions and result checks / 断言与结果检查
```python
            tree_cache=tree_cache,
            enable_hierarchical_cache=True,
            enable_priority_scheduling=True,
            schedule_low_priority_values_first=True,
        )
        policy.calc_priority(waiting_queue)
        # Check if priority enabled fcfs ordering is applied.
        self.assertEqual(waiting_queue[0].rid, 1)
        self.assertEqual(waiting_queue[1].rid, 2)
        self.assertEqual(waiting_queue[2].rid, 3)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `calc_priority` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 200-213: Test routines around test_calc_priority_routing_key_scheduling / 测试例程
```python
    def test_calc_priority_routing_key_scheduling(self):
        """Test routing-key policy: prioritize by routing key frequency in running batch."""
        tree_cache = RadixCache.create_simulated()

        running_reqs = [
            Req("r1", "a", [1], SamplingParams(), routing_key="key_a"),
            Req("r2", "b", [2], SamplingParams(), routing_key="key_a"),
            Req("r3", "c", [3], SamplingParams(), routing_key="key_b"),
        ]
        running_batch = ScheduleBatch(reqs=running_reqs)

        waiting_queue = [
            Req("w1", "d", [4], SamplingParams(), routing_key="key_b"),
            Req("w2", "e", [5], SamplingParams(), routing_key="key_a"),
```
**EN:** This range defines concrete test routine(s) `test_calc_priority_routing_key_scheduling`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `create_simulated`, `Req`, `SamplingParams` and `ScheduleBatch`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 214-224: Scenario logic / 场景逻辑
```python
            Req("w3", "f", [6], SamplingParams(), routing_key="key_c"),
        ]

        policy = SchedulePolicy(
            policy="routing-key",
            tree_cache=tree_cache,
            enable_hierarchical_cache=False,
            enable_priority_scheduling=False,
            schedule_low_priority_values_first=False,
        )
        policy.calc_priority(waiting_queue, running_batch)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `Req`, `SamplingParams`, `SchedulePolicy` and `calc_priority`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 225-228: Assertions and result checks / 断言与结果检查
```python

        self.assertEqual(waiting_queue[0].rid, "w2")
        self.assertEqual(waiting_queue[1].rid, "w1")
        self.assertEqual(waiting_queue[2].rid, "w3")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 230-238: Test routines around test_calc_priority_routing_key_tie_break_by_lexicographic_order / 测试例程
```python
    def test_calc_priority_routing_key_tie_break_by_lexicographic_order(self):
        """Test routing-key policy: tie-break by lexicographic order."""
        tree_cache = RadixCache.create_simulated()

        running_reqs = [
            Req("r1", "a", [1], SamplingParams(), routing_key="key_b"),
            Req("r2", "b", [2], SamplingParams(), routing_key="key_a"),
        ]
        running_batch = ScheduleBatch(reqs=running_reqs)
```
**EN:** This range defines concrete test routine(s) `test_calc_priority_routing_key_tie_break_by_lexicographic_order`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `create_simulated`, `Req`, `SamplingParams` and `ScheduleBatch`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 239-254: Assertions and result checks / 断言与结果检查
```python

        waiting_queue = [
            Req("w1", "d", [4], SamplingParams(), routing_key="key_b"),
            Req("w2", "e", [5], SamplingParams(), routing_key="key_a"),
        ]

        policy = SchedulePolicy(
            policy="routing-key",
            tree_cache=tree_cache,
            enable_hierarchical_cache=False,
            enable_priority_scheduling=False,
            schedule_low_priority_values_first=False,
        )
        policy.calc_priority(waiting_queue, running_batch)

        self.assertEqual(waiting_queue[0].rid, "w2")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `Req`, `SamplingParams`, `SchedulePolicy` and `calc_priority`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 255-255: Assertions and result checks / 断言与结果检查
```python
        self.assertEqual(waiting_queue[1].rid, "w1")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 257-263: Test routines around test_calc_priority_routing_key_no_match_deprioritized / 测试例程
```python
    def test_calc_priority_routing_key_no_match_deprioritized(self):
        """Test routing-key policy: requests without matching routing keys are deprioritized."""
        tree_cache = RadixCache.create_simulated()

        running_reqs = [
            Req("r1", "a", [1], SamplingParams(), routing_key="key_a"),
            Req("r2", "b", [2], SamplingParams(), routing_key="key_b"),
```
**EN:** This range defines concrete test routine(s) `test_calc_priority_routing_key_no_match_deprioritized`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `create_simulated`, `Req` and `SamplingParams`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 264-281: Scenario logic / 场景逻辑
```python
            Req("r3", "c", [3], SamplingParams(), routing_key="key_c"),
        ]
        running_batch = ScheduleBatch(reqs=running_reqs)

        waiting_queue = [
            Req("w1", "d", [4], SamplingParams(), routing_key="key_d"),
            Req("w2", "e", [5], SamplingParams(), routing_key="key_e"),
            Req("w3", "f", [6], SamplingParams(), routing_key="key_c"),
        ]

        policy = SchedulePolicy(
            policy="routing-key",
            tree_cache=tree_cache,
            enable_hierarchical_cache=False,
            enable_priority_scheduling=False,
            schedule_low_priority_values_first=False,
        )
        policy.calc_priority(waiting_queue, running_batch)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `Req`, `SamplingParams`, `ScheduleBatch` and `SchedulePolicy`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 282-285: Assertions and result checks / 断言与结果检查
```python

        self.assertEqual(waiting_queue[0].rid, "w3")
        self.assertEqual(waiting_queue[1].rid, "w1")
        self.assertEqual(waiting_queue[2].rid, "w2")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 287-288: Test routines around test_calc_priority_routing_key_empty_running_batch / 测试例程
```python
    def test_calc_priority_routing_key_empty_running_batch(self):
        """Test routing-key policy: empty running batch keeps original order."""
```
**EN:** This range defines concrete test routine(s) `test_calc_priority_routing_key_empty_running_batch`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 289-310: Assertions and result checks / 断言与结果检查
```python
        tree_cache = RadixCache.create_simulated()

        running_batch = ScheduleBatch(reqs=[])

        waiting_queue = [
            Req("w1", "d", [4], SamplingParams(), routing_key="key_a"),
            Req("w2", "e", [5], SamplingParams(), routing_key="key_b"),
            Req("w3", "f", [6], SamplingParams(), routing_key="key_c"),
        ]

        policy = SchedulePolicy(
            policy="routing-key",
            tree_cache=tree_cache,
            enable_hierarchical_cache=False,
            enable_priority_scheduling=False,
            schedule_low_priority_values_first=False,
        )
        policy.calc_priority(waiting_queue, running_batch)

        self.assertEqual(waiting_queue[0].rid, "w1")
        self.assertEqual(waiting_queue[1].rid, "w2")
        self.assertEqual(waiting_queue[2].rid, "w3")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `create_simulated`, `ScheduleBatch`, `Req` and `SamplingParams`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 311-314: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Token-level inspection / Token 级分析
- Prefix caching / 前缀缓存

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.managers.schedule_policy`, `sglang.srt.mem_cache.radix_cache`, `sglang.srt.sampling.sampling_params`, `sglang.test.test_utils`
