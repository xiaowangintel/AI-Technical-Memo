# test_balanced_packing.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/eplb/test_balanced_packing.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates balanced packing behavior in SGLang's unit / eplb area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / eplb 领域中与 balanced packing 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for balanced_packing — no server, no model loading."""
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

### Lines 7-12: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch

from sglang.srt.eplb.eplb_algorithms.deepseek import balanced_packing
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `sglang.srt.eplb.eplb_algorithms.deepseek`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `sglang.srt.eplb.eplb_algorithms.deepseek`, `sglang.test.test_utils`。

### Lines 15-15: class TestBalancedPacking declaration / 类 TestBalancedPacking 声明
```python
class TestBalancedPacking(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 16-25: supporting statements / 辅助语句
```python
    """Tests for balanced_packing(weight, num_packs).

    Invariants:
    - Output shapes match input: both [X, n].
    - pack_index values are in [0, num_packs).
    - Each pack receives exactly n // num_packs items per layer.
    - rank_in_pack values are in [0, groups_per_pack).
    - Each (pack, rank) slot is used exactly once per layer.
    - Packs are as weight-balanced as possible (greedy optimality).
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 26-28: supporting source context / 辅助源码上下文
```python

    # ------------------------------------------------------------------ helpers

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 29-31: method check shapes / 方法 check shapes
```python
    def _check_shapes(self, weight, pack_index, rank_in_pack):
        self.assertEqual(pack_index.shape, weight.shape)
        self.assertEqual(rank_in_pack.shape, weight.shape)
```
**EN:** This block implements `_check_shapes` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_check_shapes`，承担模块行为中的一个聚焦逻辑片段。

### Lines 33-35: method check pack index range / 方法 check pack index range
```python
    def _check_pack_index_range(self, pack_index, num_packs):
        self.assertTrue(torch.all(pack_index >= 0))
        self.assertTrue(torch.all(pack_index < num_packs))
```
**EN:** This block implements `_check_pack_index_range` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_check_pack_index_range`，承担模块行为中的一个聚焦逻辑片段。

### Lines 37-44: method check items per pack / 方法 check items per pack
```python
    def _check_items_per_pack(self, pack_index, num_packs, groups_per_pack):
        """Every pack must hold exactly groups_per_pack items in every layer."""
        for layer in range(pack_index.shape[0]):
            counts = torch.bincount(pack_index[layer], minlength=num_packs)
            self.assertTrue(
                torch.all(counts == groups_per_pack),
                f"layer {layer}: pack counts {counts.tolist()} != {groups_per_pack}",
            )
```
**EN:** Every pack must hold exactly groups_per_pack items in every layer. This block implements `_check_items_per_pack` and captures one focused piece of the module's behavior.
**CN:** Every pack must hold exactly groups_per_pack items in every layer. 该代码块实现 `_check_items_per_pack`，承担模块行为中的一个聚焦逻辑片段。

### Lines 46-48: method check rank in pack range / 方法 check rank in pack range
```python
    def _check_rank_in_pack_range(self, rank_in_pack, groups_per_pack):
        self.assertTrue(torch.all(rank_in_pack >= 0))
        self.assertTrue(torch.all(rank_in_pack < groups_per_pack))
```
**EN:** This block implements `_check_rank_in_pack_range` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_check_rank_in_pack_range`，承担模块行为中的一个聚焦逻辑片段。

### Lines 50-55: method check unique slots / 方法 check unique slots
```python
    def _check_unique_slots(self, pack_index, rank_in_pack, num_packs, groups_per_pack):
        """Each (pack, rank) slot is occupied exactly once per layer."""
        num_layers = pack_index.shape[0]
        for layer in range(num_layers):
            slots = set(zip(pack_index[layer].tolist(), rank_in_pack[layer].tolist()))
            self.assertEqual(len(slots), num_packs * groups_per_pack)
```
**EN:** Each (pack, rank) slot is occupied exactly once per layer. This block implements `_check_unique_slots` and captures one focused piece of the module's behavior.
**CN:** Each (pack, rank) slot is occupied exactly once per layer. 该代码块实现 `_check_unique_slots`，承担模块行为中的一个聚焦逻辑片段。

### Lines 56-58: supporting source context / 辅助源码上下文
```python

    # ------------------------------------------------------------------ tests

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 59-63: test case output shapes / 测试用例 output shapes
```python
    def test_output_shapes(self):
        """pack_index and rank_in_pack have the same shape as weight."""
        weight = torch.rand(3, 8)
        pack_index, rank_in_pack = balanced_packing(weight, num_packs=4)
        self._check_shapes(weight, pack_index, rank_in_pack)
```
**EN:** pack_index and rank_in_pack have the same shape as weight. This test exercises `test_output_shapes` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** pack_index and rank_in_pack have the same shape as weight. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_output_shapes`。

### Lines 65-69: test case pack index range / 测试用例 pack index range
```python
    def test_pack_index_range(self):
        """All pack indices are in [0, num_packs)."""
        weight = torch.rand(2, 6)
        pack_index, _ = balanced_packing(weight, num_packs=3)
        self._check_pack_index_range(pack_index, num_packs=3)
```
**EN:** All pack indices are in [0, num_packs). This test exercises `test_pack_index_range` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** All pack indices are in [0, num_packs). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pack_index_range`。

### Lines 71-76: test case each pack receives equal items / 测试用例 each pack receives equal items
```python
    def test_each_pack_receives_equal_items(self):
        """Each pack receives exactly n // num_packs items per layer."""
        weight = torch.rand(4, 8)
        num_packs = 4
        pack_index, _ = balanced_packing(weight, num_packs=num_packs)
        self._check_items_per_pack(pack_index, num_packs, groups_per_pack=2)
```
**EN:** Each pack receives exactly n // num_packs items per layer. This test exercises `test_each_pack_receives_equal_items` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Each pack receives exactly n // num_packs items per layer. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_each_pack_receives_equal_items`。

### Lines 78-84: test case rank in pack range / 测试用例 rank in pack range
```python
    def test_rank_in_pack_range(self):
        """rank_in_pack values are in [0, groups_per_pack)."""
        weight = torch.rand(2, 8)
        num_packs = 4
        groups_per_pack = 8 // num_packs
        _, rank_in_pack = balanced_packing(weight, num_packs=num_packs)
        self._check_rank_in_pack_range(rank_in_pack, groups_per_pack)
```
**EN:** rank_in_pack values are in [0, groups_per_pack). This test exercises `test_rank_in_pack_range` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** rank_in_pack values are in [0, groups_per_pack). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rank_in_pack_range`。

### Lines 86-91: test case unique pack rank slots / 测试用例 unique pack rank slots
```python
    def test_unique_pack_rank_slots(self):
        """Each (pack, rank) slot is used exactly once per layer."""
        weight = torch.rand(3, 8)
        num_packs = 4
        pack_index, rank_in_pack = balanced_packing(weight, num_packs=num_packs)
        self._check_unique_slots(pack_index, rank_in_pack, num_packs, groups_per_pack=2)
```
**EN:** Each (pack, rank) slot is used exactly once per layer. This test exercises `test_unique_pack_rank_slots` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Each (pack, rank) slot is used exactly once per layer. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unique_pack_rank_slots`。

### Lines 93-102: test case groups per pack one special case / 测试用例 groups per pack one special case
```python
    def test_groups_per_pack_one_special_case(self):
        """When groups_per_pack == 1 (num_packs == n), each item gets its own pack."""
        n = 6
        weight = torch.rand(2, n)
        pack_index, rank_in_pack = balanced_packing(weight, num_packs=n)
        # pack_index[layer] should be a permutation of [0, n)
        for layer in range(weight.shape[0]):
            self.assertEqual(sorted(pack_index[layer].tolist()), list(range(n)))
        # rank_in_pack is all zeros
        self.assertTrue(torch.all(rank_in_pack == 0))
```
**EN:** When groups_per_pack == 1 (num_packs == n), each item gets its own pack. This test exercises `test_groups_per_pack_one_special_case` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When groups_per_pack == 1 (num_packs == n), each item gets its own pack. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_groups_per_pack_one_special_case`。

### Lines 104-109: test case single layer / 测试用例 single layer
```python
    def test_single_layer(self):
        """Works correctly with a single layer."""
        weight = torch.tensor([[3.0, 1.0, 4.0, 1.0]])
        pack_index, rank_in_pack = balanced_packing(weight, num_packs=2)
        self._check_shapes(weight, pack_index, rank_in_pack)
        self._check_items_per_pack(pack_index, num_packs=2, groups_per_pack=2)
```
**EN:** Works correctly with a single layer. This test exercises `test_single_layer` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Works correctly with a single layer. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_layer`。

### Lines 111-120: test case uniform weights all invariants / 测试用例 uniform weights all invariants
```python
    def test_uniform_weights_all_invariants(self):
        """Uniform weights: all invariants hold regardless of assignment."""
        weight = torch.ones(3, 8)
        num_packs = 4
        pack_index, rank_in_pack = balanced_packing(weight, num_packs=num_packs)
        self._check_shapes(weight, pack_index, rank_in_pack)
        self._check_pack_index_range(pack_index, num_packs)
        self._check_items_per_pack(pack_index, num_packs, groups_per_pack=2)
        self._check_rank_in_pack_range(rank_in_pack, groups_per_pack=2)
        self._check_unique_slots(pack_index, rank_in_pack, num_packs, groups_per_pack=2)
```
**EN:** Uniform weights: all invariants hold regardless of assignment. This test exercises `test_uniform_weights_all_invariants` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Uniform weights: all invariants hold regardless of assignment. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_uniform_weights_all_invariants`。

### Lines 122-131: test case balance property / 测试用例 balance property
```python
    def test_balance_property(self):
        """Heavier items are spread across packs to minimize max pack weight."""
        # Weights: [9, 1, 1, 1] with 2 packs → optimal: {9,1} and {1,1}, not {9,1,1} and {1}
        weight = torch.tensor([[9.0, 1.0, 1.0, 1.0]])
        pack_index, _ = balanced_packing(weight, num_packs=2)
        pack_weights = torch.zeros(2)
        for i, p in enumerate(pack_index[0].tolist()):
            pack_weights[p] += weight[0, i]
        # Max pack weight should be 10 (9+1), not 11 (9+1+1)
        self.assertEqual(pack_weights.max().item(), 10.0)
```
**EN:** Heavier items are spread across packs to minimize max pack weight. This test exercises `test_balance_property` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Heavier items are spread across packs to minimize max pack weight. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_balance_property`。

### Lines 133-139: test case deterministic / 测试用例 deterministic
```python
    def test_deterministic(self):
        """Same input always produces the same output."""
        weight = torch.rand(3, 8)
        result1 = balanced_packing(weight.clone(), num_packs=4)
        result2 = balanced_packing(weight.clone(), num_packs=4)
        self.assertTrue(torch.equal(result1[0], result2[0]))
        self.assertTrue(torch.equal(result1[1], result2[1]))
```
**EN:** Same input always produces the same output. This test exercises `test_deterministic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Same input always produces the same output. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deterministic`。

### Lines 141-149: test case many layers / 测试用例 many layers
```python
    def test_many_layers(self):
        """All invariants hold across many layers."""
        weight = torch.rand(16, 8)
        num_packs = 4
        pack_index, rank_in_pack = balanced_packing(weight, num_packs=num_packs)
        self._check_shapes(weight, pack_index, rank_in_pack)
        self._check_pack_index_range(pack_index, num_packs)
        self._check_items_per_pack(pack_index, num_packs, groups_per_pack=2)
        self._check_unique_slots(pack_index, rank_in_pack, num_packs, groups_per_pack=2)
```
**EN:** All invariants hold across many layers. This test exercises `test_many_layers` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** All invariants hold across many layers. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_many_layers`。

### Lines 152-153: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestBalancedPacking`: Tests for balanced_packing(weight, num_packs). / 用于组织相关测试、夹具或辅助方法。
- `TestBalancedPacking._check_shapes`: This block implements `_check_shapes` and captures one focused piece of the module's behavior. / 该代码块实现 `_check_shapes`，承担模块行为中的一个聚焦逻辑片段。
- `TestBalancedPacking._check_pack_index_range`: This block implements `_check_pack_index_range` and captures one focused piece of the module's behavior. / 该代码块实现 `_check_pack_index_range`，承担模块行为中的一个聚焦逻辑片段。
- `TestBalancedPacking._check_items_per_pack`: Every pack must hold exactly groups_per_pack items in every layer. / 该代码块实现 `_check_items_per_pack`，承担模块行为中的一个聚焦逻辑片段。
- `TestBalancedPacking._check_rank_in_pack_range`: This block implements `_check_rank_in_pack_range` and captures one focused piece of the module's behavior. / 该代码块实现 `_check_rank_in_pack_range`，承担模块行为中的一个聚焦逻辑片段。
- `TestBalancedPacking._check_unique_slots`: Each (pack, rank) slot is occupied exactly once per layer. / 该代码块实现 `_check_unique_slots`，承担模块行为中的一个聚焦逻辑片段。
- `TestBalancedPacking.test_output_shapes`: pack_index and rank_in_pack have the same shape as weight. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_output_shapes`。
- `TestBalancedPacking.test_pack_index_range`: All pack indices are in [0, num_packs). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pack_index_range`。
- `TestBalancedPacking.test_each_pack_receives_equal_items`: Each pack receives exactly n // num_packs items per layer. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_each_pack_receives_equal_items`。
- `TestBalancedPacking.test_rank_in_pack_range`: rank_in_pack values are in [0, groups_per_pack). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rank_in_pack_range`。
- `TestBalancedPacking.test_unique_pack_rank_slots`: Each (pack, rank) slot is used exactly once per layer. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unique_pack_rank_slots`。
- `TestBalancedPacking.test_groups_per_pack_one_special_case`: When groups_per_pack == 1 (num_packs == n), each item gets its own pack. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_groups_per_pack_one_special_case`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.eplb.eplb_algorithms.deepseek`, `sglang.test.test_utils`

- **Total lines / 总行数**: 153
