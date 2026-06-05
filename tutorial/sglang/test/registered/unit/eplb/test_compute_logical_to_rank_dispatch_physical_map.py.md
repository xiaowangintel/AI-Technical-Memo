# test_compute_logical_to_rank_dispatch_physical_map.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/eplb/test_compute_logical_to_rank_dispatch_physical_map.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates compute logical to rank dispatch physical map behavior in SGLang's unit / eplb area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / eplb 领域中与 compute logical to rank dispatch physical map 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for compute_logical_to_rank_dispatch_physical_map — no server, no model loading."""
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

### Lines 7-15: module imports and dependencies / 模块导入与依赖
```python
import types
import unittest

import torch

from sglang.srt.eplb.expert_location import (
    compute_logical_to_rank_dispatch_physical_map,
)
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `types`, `unittest`, `torch`, `sglang.srt.eplb.expert_location`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `types`, `unittest`, `torch`, `sglang.srt.eplb.expert_location`。

### Lines 18-20: function make server args / 函数 make server args
```python
def _make_server_args(ep_size: int, nnodes: int):
    """Minimal server_args stub — only ep_size and nnodes are used."""
    return types.SimpleNamespace(ep_size=ep_size, nnodes=nnodes)
```
**EN:** Minimal server_args stub — only ep_size and nnodes are used. This block implements `_make_server_args` and captures one focused piece of the module's behavior.
**CN:** Minimal server_args stub — only ep_size and nnodes are used. 该代码块实现 `_make_server_args`，承担模块行为中的一个聚焦逻辑片段。

### Lines 23-40: function make logical to all physical map / 函数 make logical to all physical map
```python
def _make_logical_to_all_physical_map(
    num_layers: int,
    num_logical_experts: int,
    num_physical_experts: int,
    replicas_per_logical: int,
) -> torch.Tensor:
    """Build a simple [num_layers, num_logical_experts, replicas_per_logical] map.

    Physical expert assignment: logical i → physical [i*R, i*R+1, ..., i*R+R-1]
    where R = replicas_per_logical.
    """
    mapping = torch.full(
        (num_layers, num_logical_experts, replicas_per_logical), -1, dtype=torch.int64
    )
    for logical_id in range(num_logical_experts):
        for r in range(replicas_per_logical):
            mapping[:, logical_id, r] = logical_id * replicas_per_logical + r
    return mapping
```
**EN:** Build a simple [num_layers, num_logical_experts, replicas_per_logical] map. This block implements `_make_logical_to_all_physical_map` and captures one focused piece of the module's behavior.
**CN:** Build a simple [num_layers, num_logical_experts, replicas_per_logical] map. 该代码块实现 `_make_logical_to_all_physical_map`，承担模块行为中的一个聚焦逻辑片段。

### Lines 43-43: class TestComputeLogicalToRankDispatchPhysicalMap declaration / 类 TestComputeLogicalToRankDispatchPhysicalMap 声明
```python
class TestComputeLogicalToRankDispatchPhysicalMap(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 44-63: class-level constants and configuration for `TestComputeLogicalToRankDispatchPhysicalMap` / 类级常量与配置
```python
    """Tests for compute_logical_to_rank_dispatch_physical_map.

    Setup used in most tests:
      - 4 GPUs (ep_size=4), 2 nodes (nnodes=2) → 2 GPUs/node
      - 8 physical experts (2 per GPU), 4 logical experts (each replicated ×2)
      - physical expert layout:
          GPU 0 (node 0): experts 0, 1
          GPU 1 (node 0): experts 2, 3
          GPU 2 (node 1): experts 4, 5
          GPU 3 (node 1): experts 6, 7
      - logical→physical:
          logical 0 → [0, 1],  logical 1 → [2, 3]
          logical 2 → [4, 5],  logical 3 → [6, 7]
    """

    EP_SIZE = 4
    NNODES = 2
    NUM_PHYSICAL = 8
    NUM_LOGICAL = 4
    NUM_LAYERS = 2
```
**EN:** This block defines shared names such as `EP_SIZE`, `NNODES`, `NUM_PHYSICAL`, `NUM_LOGICAL`, `NUM_LAYERS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `EP_SIZE`, `NNODES`, `NUM_PHYSICAL`, `NUM_LOGICAL`, `NUM_LAYERS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 65-72: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self.server_args = _make_server_args(self.EP_SIZE, self.NNODES)
        self.logical_to_all_physical = _make_logical_to_all_physical_map(
            num_layers=self.NUM_LAYERS,
            num_logical_experts=self.NUM_LOGICAL,
            num_physical_experts=self.NUM_PHYSICAL,
            replicas_per_logical=2,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 74-82: method call / 方法 call
```python
    def _call(self, ep_rank, seed=42):
        return compute_logical_to_rank_dispatch_physical_map(
            server_args=self.server_args,
            logical_to_all_physical_map=self.logical_to_all_physical.clone(),
            ep_size=self.EP_SIZE,
            num_physical_experts=self.NUM_PHYSICAL,
            ep_rank=ep_rank,
            seed=seed,
        )
```
**EN:** This block implements `_call` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_call`，承担模块行为中的一个聚焦逻辑片段。

### Lines 83-85: supporting source context / 辅助源码上下文
```python

    # ------------------------------------------------------------------ shape & range

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 86-89: test case output shape / 测试用例 output shape
```python
    def test_output_shape(self):
        """Output is [num_layers, num_logical_experts]."""
        result = self._call(ep_rank=0)
        self.assertEqual(result.shape, (self.NUM_LAYERS, self.NUM_LOGICAL))
```
**EN:** Output is [num_layers, num_logical_experts]. This test exercises `test_output_shape` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Output is [num_layers, num_logical_experts]. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_output_shape`。

### Lines 91-101: test case all values are valid physical expert ids / 测试用例 all values are valid physical expert ids
```python
    def test_all_values_are_valid_physical_expert_ids(self):
        """Every entry is a valid physical expert ID in [0, num_physical_experts)."""
        for ep_rank in range(self.EP_SIZE):
            result = self._call(ep_rank=ep_rank)
            self.assertTrue(
                torch.all(result >= 0), f"ep_rank={ep_rank} has negative values"
            )
            self.assertTrue(
                torch.all(result < self.NUM_PHYSICAL),
                f"ep_rank={ep_rank} has out-of-range values",
            )
```
**EN:** Every entry is a valid physical expert ID in [0, num_physical_experts). This test exercises `test_all_values_are_valid_physical_expert_ids` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Every entry is a valid physical expert ID in [0, num_physical_experts). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_values_are_valid_physical_expert_ids`。

### Lines 103-110: test case no minus one in output / 测试用例 no minus one in output
```python
    def test_no_minus_one_in_output(self):
        """No -1 sentinel values remain in the output (all ranks are assigned)."""
        for ep_rank in range(self.EP_SIZE):
            result = self._call(ep_rank=ep_rank)
            self.assertFalse(
                torch.any(result == -1),
                f"ep_rank={ep_rank} still has unassigned entries",
            )
```
**EN:** No -1 sentinel values remain in the output (all ranks are assigned). This test exercises `test_no_minus_one_in_output` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** No -1 sentinel values remain in the output (all ranks are assigned). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_minus_one_in_output`。

### Lines 111-113: supporting source context / 辅助源码上下文
```python

    # ------------------------------------------------------------------ correctness

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 114-119: test case gpu0 prefers local experts / 测试用例 gpu0 prefers local experts
```python
    def test_gpu0_prefers_local_experts(self):
        """GPU 0 (node 0) should be assigned its local physical experts (0 or 1)."""
        result = self._call(ep_rank=0)
        # Logical 0 has candidates [0,1] — both on GPU 0 → nearest is 0
        for layer in range(self.NUM_LAYERS):
            self.assertIn(result[layer, 0].item(), [0, 1])
```
**EN:** GPU 0 (node 0) should be assigned its local physical experts (0 or 1). This test exercises `test_gpu0_prefers_local_experts` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** GPU 0 (node 0) should be assigned its local physical experts (0 or 1). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gpu0_prefers_local_experts`。

### Lines 121-126: test case same node fallback / 测试用例 same node fallback
```python
    def test_same_node_fallback(self):
        """GPU 0 (node 0) should get a node-0 expert for logical 1 (experts 2,3 on GPU 1)."""
        result = self._call(ep_rank=0)
        # Logical 1 → candidates [2, 3], GPU 1 (node 0) → same-node match
        for layer in range(self.NUM_LAYERS):
            self.assertIn(result[layer, 1].item(), [2, 3])
```
**EN:** GPU 0 (node 0) should get a node-0 expert for logical 1 (experts 2,3 on GPU 1). This test exercises `test_same_node_fallback` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** GPU 0 (node 0) should get a node-0 expert for logical 1 (experts 2,3 on GPU 1). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_same_node_fallback`。

### Lines 128-137: test case each rank gets different assignment / 测试用例 each rank gets different assignment
```python
    def test_each_rank_gets_different_assignment(self):
        """Different ep_ranks should in general get different physical experts."""
        results = [self._call(ep_rank=r) for r in range(self.EP_SIZE)]
        # At least two ranks should differ for at least one entry
        any_diff = any(
            not torch.equal(results[i], results[j])
            for i in range(self.EP_SIZE)
            for j in range(i + 1, self.EP_SIZE)
        )
        self.assertTrue(any_diff, "All ranks produced identical mappings")
```
**EN:** Different ep_ranks should in general get different physical experts. This test exercises `test_each_rank_gets_different_assignment` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Different ep_ranks should in general get different physical experts. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_each_rank_gets_different_assignment`。

### Lines 138-140: supporting source context / 辅助源码上下文
```python

    # ------------------------------------------------------------------ determinism & seed

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 141-145: test case deterministic same seed / 测试用例 deterministic same seed
```python
    def test_deterministic_same_seed(self):
        """Same seed always produces the same result."""
        r1 = self._call(ep_rank=0, seed=7)
        r2 = self._call(ep_rank=0, seed=7)
        self.assertTrue(torch.equal(r1, r2))
```
**EN:** Same seed always produces the same result. This test exercises `test_deterministic_same_seed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Same seed always produces the same result. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deterministic_same_seed`。

### Lines 147-153: test case different seeds may differ / 测试用例 different seeds may differ
```python
    def test_different_seeds_may_differ(self):
        """Different seeds can produce different assignments for remote experts."""
        results = {
            tuple(self._call(ep_rank=2, seed=s).flatten().tolist()) for s in range(20)
        }
        # GPU 2 has some remote experts → seed affects _fair_choices → results can vary
        self.assertGreater(len(results), 1)
```
**EN:** Different seeds can produce different assignments for remote experts. This test exercises `test_different_seeds_may_differ` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Different seeds can produce different assignments for remote experts. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_different_seeds_may_differ`。

### Lines 154-156: supporting source context / 辅助源码上下文
```python

    # ------------------------------------------------------------------ edge cases

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 157-173: test case single layer / 测试用例 single layer
```python
    def test_single_layer(self):
        """Works correctly with a single MoE layer."""
        logical_to_all_physical = _make_logical_to_all_physical_map(
            num_layers=1,
            num_logical_experts=self.NUM_LOGICAL,
            num_physical_experts=self.NUM_PHYSICAL,
            replicas_per_logical=2,
        )
        result = compute_logical_to_rank_dispatch_physical_map(
            server_args=self.server_args,
            logical_to_all_physical_map=logical_to_all_physical,
            ep_size=self.EP_SIZE,
            num_physical_experts=self.NUM_PHYSICAL,
            ep_rank=0,
        )
        self.assertEqual(result.shape, (1, self.NUM_LOGICAL))
        self.assertTrue(torch.all(result >= 0))
```
**EN:** Works correctly with a single MoE layer. This test exercises `test_single_layer` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Works correctly with a single MoE layer. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_layer`。

### Lines 175-187: test case single node / 测试用例 single node
```python
    def test_single_node(self):
        """With nnodes=1, all GPUs are on the same node."""
        server_args = _make_server_args(ep_size=4, nnodes=1)
        result = compute_logical_to_rank_dispatch_physical_map(
            server_args=server_args,
            logical_to_all_physical_map=self.logical_to_all_physical.clone(),
            ep_size=self.EP_SIZE,
            num_physical_experts=self.NUM_PHYSICAL,
            ep_rank=0,
        )
        self.assertEqual(result.shape, (self.NUM_LAYERS, self.NUM_LOGICAL))
        self.assertTrue(torch.all(result >= 0))
        self.assertTrue(torch.all(result < self.NUM_PHYSICAL))
```
**EN:** With nnodes=1, all GPUs are on the same node. This test exercises `test_single_node` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** With nnodes=1, all GPUs are on the same node. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_node`。

### Lines 189-204: test case all experts replicated to all gpus / 测试用例 all experts replicated to all gpus
```python
    def test_all_experts_replicated_to_all_gpus(self):
        """When every physical expert maps to the same logical expert, all ranks get valid IDs."""
        # All physical experts are replicas of a single logical expert
        mapping = (
            torch.arange(self.NUM_PHYSICAL, dtype=torch.int64).unsqueeze(0).unsqueeze(0)
        )
        mapping = mapping.expand(self.NUM_LAYERS, 1, self.NUM_PHYSICAL).clone()
        result = compute_logical_to_rank_dispatch_physical_map(
            server_args=self.server_args,
            logical_to_all_physical_map=mapping,
            ep_size=self.EP_SIZE,
            num_physical_experts=self.NUM_PHYSICAL,
            ep_rank=0,
        )
        self.assertEqual(result.shape, (self.NUM_LAYERS, 1))
        self.assertTrue(torch.all(result >= 0))
```
**EN:** When every physical expert maps to the same logical expert, all ranks get valid IDs. This test exercises `test_all_experts_replicated_to_all_gpus` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When every physical expert maps to the same logical expert, all ranks get valid IDs. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_experts_replicated_to_all_gpus`。

### Lines 207-208: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_server_args`: Minimal server_args stub — only ep_size and nnodes are used. / 该代码块实现 `_make_server_args`，承担模块行为中的一个聚焦逻辑片段。
- `_make_logical_to_all_physical_map`: Build a simple [num_layers, num_logical_experts, replicas_per_logical] map. / 该代码块实现 `_make_logical_to_all_physical_map`，承担模块行为中的一个聚焦逻辑片段。
- `TestComputeLogicalToRankDispatchPhysicalMap`: Tests for compute_logical_to_rank_dispatch_physical_map. / 用于组织相关测试、夹具或辅助方法。
- `TestComputeLogicalToRankDispatchPhysicalMap.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestComputeLogicalToRankDispatchPhysicalMap._call`: This block implements `_call` and captures one focused piece of the module's behavior. / 该代码块实现 `_call`，承担模块行为中的一个聚焦逻辑片段。
- `TestComputeLogicalToRankDispatchPhysicalMap.test_output_shape`: Output is [num_layers, num_logical_experts]. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_output_shape`。
- `TestComputeLogicalToRankDispatchPhysicalMap.test_all_values_are_valid_physical_expert_ids`: Every entry is a valid physical expert ID in [0, num_physical_experts). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_values_are_valid_physical_expert_ids`。
- `TestComputeLogicalToRankDispatchPhysicalMap.test_no_minus_one_in_output`: No -1 sentinel values remain in the output (all ranks are assigned). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_minus_one_in_output`。
- `TestComputeLogicalToRankDispatchPhysicalMap.test_gpu0_prefers_local_experts`: GPU 0 (node 0) should be assigned its local physical experts (0 or 1). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gpu0_prefers_local_experts`。
- `TestComputeLogicalToRankDispatchPhysicalMap.test_same_node_fallback`: GPU 0 (node 0) should get a node-0 expert for logical 1 (experts 2,3 on GPU 1). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_same_node_fallback`。
- `TestComputeLogicalToRankDispatchPhysicalMap.test_each_rank_gets_different_assignment`: Different ep_ranks should in general get different physical experts. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_each_rank_gets_different_assignment`。
- `TestComputeLogicalToRankDispatchPhysicalMap.test_deterministic_same_seed`: Same seed always produces the same result. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deterministic_same_seed`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `types`, `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.eplb.expert_location`, `sglang.test.test_utils`

- **Total lines / 总行数**: 208
