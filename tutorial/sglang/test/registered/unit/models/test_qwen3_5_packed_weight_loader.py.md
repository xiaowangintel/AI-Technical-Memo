# test_qwen3_5_packed_weight_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/models/test_qwen3_5_packed_weight_loader.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates qwen3 5 packed weight loader behavior in SGLang's unit / models area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / models 领域中与 qwen3 5 packed weight loader 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: supporting statements / 辅助语句
```python
"""
Unit tests for Qwen3_5GatedDeltaNet._make_packed_weight_loader.

Validates that per-tensor FP8 scales (scalar or single-element tensors)
are broadcast to every logical shard, while normal multi-element weights
are split correctly.

Regression test for https://github.com/sgl-project/sglang/issues/23051
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 11-11: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 13-13: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=4, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 15-22: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace
from unittest.mock import MagicMock

import torch

from sglang.srt.layers.parameter import PerTensorScaleParameter
from sglang.srt.models.qwen3_5 import Qwen3_5GatedDeltaNet
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `unittest.mock`, `torch`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `unittest.mock`, `torch`。

### Lines 25-27: function make mock module / 函数 make mock module
```python
def _make_mock_module(output_sizes):
    """Create a lightweight mock module with the attributes needed by the loader."""
    return SimpleNamespace(output_sizes=output_sizes)
```
**EN:** Create a lightweight mock module with the attributes needed by the loader. This block implements `_make_mock_module` and captures one focused piece of the module's behavior.
**CN:** Create a lightweight mock module with the attributes needed by the loader. 该代码块实现 `_make_mock_module`，承担模块行为中的一个聚焦逻辑片段。

### Lines 30-39: function make per tensor scale param / 函数 make per tensor scale param
```python
def _make_per_tensor_scale_param(num_shards):
    """Create a PerTensorScaleParameter pre-allocated for `num_shards` scales.

    PerTensorScaleParameter requires a weight_loader callable;
    we supply a no-op since the packed loader wraps it anyway.
    """
    return PerTensorScaleParameter(
        data=torch.zeros(num_shards),
        weight_loader=lambda *args, **kwargs: None,
    )
```
**EN:** Create a PerTensorScaleParameter pre-allocated for `num_shards` scales. This block implements `_make_per_tensor_scale_param` and captures one focused piece of the module's behavior.
**CN:** Create a PerTensorScaleParameter pre-allocated for `num_shards` scales. 该代码块实现 `_make_per_tensor_scale_param`，承担模块行为中的一个聚焦逻辑片段。

### Lines 42-42: class TestMakePackedWeightLoader declaration / 类 TestMakePackedWeightLoader 声明
```python
class TestMakePackedWeightLoader(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 43-43: supporting statements / 辅助语句
```python
    """Tests for _make_packed_weight_loader broadcast / split logic."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 44-48: supporting source context / 辅助源码上下文
```python

    # ------------------------------------------------------------------ #
    #  Per-tensor scale broadcast                                         #
    # ------------------------------------------------------------------ #

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 49-69: test case scalar weight broadcast / 测试用例 scalar weight broadcast
```python
    def test_scalar_weight_broadcast(self):
        """A 0-d scalar should be broadcast (via .view(-1)) to every shard."""
        module = _make_mock_module(output_sizes=[128, 128, 64, 64])
        param = _make_per_tensor_scale_param(num_shards=4)

        calls = []

        def original_loader(p, chunk, shard_id):
            calls.append((shard_id, chunk.clone()))

        loader = Qwen3_5GatedDeltaNet._make_packed_weight_loader(
            module, original_loader
        )

        scalar = torch.tensor(0.5)  # shape=[]
        loader(param, scalar, loaded_shard_id=(0, 1, 2))

        self.assertEqual(len(calls), 3)
        for shard_id, chunk in calls:
            self.assertEqual(chunk.shape, torch.Size([1]))
            self.assertAlmostEqual(chunk.item(), 0.5, places=5)
```
**EN:** A 0-d scalar should be broadcast (via .view(-1)) to every shard. This test exercises `test_scalar_weight_broadcast` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** A 0-d scalar should be broadcast (via .view(-1)) to every shard. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_scalar_weight_broadcast`。

### Lines 71-93: test case single element tensor broadcast / 测试用例 single element tensor broadcast
```python
    def test_single_element_tensor_broadcast(self):
        """A [1]-shaped tensor (e.g. per-tensor weight_scale) should be
        broadcast to every logical shard."""
        module = _make_mock_module(output_sizes=[128, 128, 64, 64])
        param = _make_per_tensor_scale_param(num_shards=4)

        calls = []

        def original_loader(p, chunk, shard_id):
            calls.append((shard_id, chunk.clone()))

        loader = Qwen3_5GatedDeltaNet._make_packed_weight_loader(
            module, original_loader
        )

        scale = torch.tensor([0.25])  # shape=[1]
        loader(param, scale, loaded_shard_id=(0, 1, 2))

        self.assertEqual(len(calls), 3)
        for idx, (shard_id, chunk) in enumerate(calls):
            self.assertEqual(shard_id, idx)
            self.assertEqual(chunk.shape, torch.Size([1]))
            self.assertAlmostEqual(chunk.item(), 0.25, places=5)
```
**EN:** A [1]-shaped tensor (e.g. This test exercises `test_single_element_tensor_broadcast` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** A [1]-shaped tensor (e.g. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_element_tensor_broadcast`。

### Lines 95-115: test case broadcast with two shards / 测试用例 broadcast with two shards
```python
    def test_broadcast_with_two_shards(self):
        """Broadcast for in_proj_ba style (2 shards: b, a)."""
        module = _make_mock_module(output_sizes=[16, 16])
        param = _make_per_tensor_scale_param(num_shards=2)

        calls = []

        def original_loader(p, chunk, shard_id):
            calls.append((shard_id, chunk.clone()))

        loader = Qwen3_5GatedDeltaNet._make_packed_weight_loader(
            module, original_loader
        )

        scale = torch.tensor([0.1])
        loader(param, scale, loaded_shard_id=(0, 1))

        self.assertEqual(len(calls), 2)
        for shard_id, chunk in calls:
            self.assertEqual(chunk.shape, torch.Size([1]))
            self.assertAlmostEqual(chunk.item(), 0.1, places=5)
```
**EN:** Broadcast for in_proj_ba style (2 shards: b, a). This test exercises `test_broadcast_with_two_shards` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Broadcast for in_proj_ba style (2 shards: b, a). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_broadcast_with_two_shards`。

### Lines 116-120: supporting source context / 辅助源码上下文
```python

    # ------------------------------------------------------------------ #
    #  Normal weight split                                                #
    # ------------------------------------------------------------------ #

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 121-143: test case normal weight split / 测试用例 normal weight split
```python
    def test_normal_weight_split(self):
        """Multi-element weights should be split by output_sizes, not broadcast."""
        module = _make_mock_module(output_sizes=[128, 128, 64])
        param = MagicMock()
        param.output_dim = 0

        calls = []

        def original_loader(p, chunk, shard_id):
            calls.append((shard_id, chunk.clone()))

        loader = Qwen3_5GatedDeltaNet._make_packed_weight_loader(
            module, original_loader
        )

        # Simulate a checkpoint weight that covers shard 0, 1, 2
        weight = torch.randn(128 + 128 + 64, 256)
        loader(param, weight, loaded_shard_id=(0, 1, 2))

        self.assertEqual(len(calls), 3)
        self.assertEqual(calls[0][1].shape[0], 128)
        self.assertEqual(calls[1][1].shape[0], 128)
        self.assertEqual(calls[2][1].shape[0], 64)
```
**EN:** Multi-element weights should be split by output_sizes, not broadcast. This test exercises `test_normal_weight_split` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Multi-element weights should be split by output_sizes, not broadcast. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_normal_weight_split`。

### Lines 144-148: supporting source context / 辅助源码上下文
```python

    # ------------------------------------------------------------------ #
    #  Passthrough for non-tuple shard_id                                 #
    # ------------------------------------------------------------------ #

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 149-166: test case int shard id passthrough / 测试用例 int shard id passthrough
```python
    def test_int_shard_id_passthrough(self):
        """An int shard_id should bypass the tuple logic entirely."""
        module = _make_mock_module(output_sizes=[128, 128, 64, 64])

        calls = []

        def original_loader(p, loaded_weight, shard_id):
            calls.append(("original", shard_id))

        loader = Qwen3_5GatedDeltaNet._make_packed_weight_loader(
            module, original_loader
        )

        weight = torch.randn(128, 256)
        loader(MagicMock(), weight, loaded_shard_id=2)

        self.assertEqual(len(calls), 1)
        self.assertEqual(calls[0], ("original", 2))
```
**EN:** An int shard_id should bypass the tuple logic entirely. This test exercises `test_int_shard_id_passthrough` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** An int shard_id should bypass the tuple logic entirely. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_int_shard_id_passthrough`。

### Lines 168-185: test case none shard id passthrough / 测试用例 none shard id passthrough
```python
    def test_none_shard_id_passthrough(self):
        """None shard_id should pass through to the original loader."""
        module = _make_mock_module(output_sizes=[128])

        calls = []

        def original_loader(p, loaded_weight, shard_id):
            calls.append(("original", shard_id))

        loader = Qwen3_5GatedDeltaNet._make_packed_weight_loader(
            module, original_loader
        )

        weight = torch.randn(128, 256)
        loader(MagicMock(), weight, loaded_shard_id=None)

        self.assertEqual(len(calls), 1)
        self.assertEqual(calls[0], ("original", None))
```
**EN:** None shard_id should pass through to the original loader. This test exercises `test_none_shard_id_passthrough` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** None shard_id should pass through to the original loader. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_shard_id_passthrough`。

### Lines 186-190: supporting source context / 辅助源码上下文
```python

    # ------------------------------------------------------------------ #
    #  Edge case: nested single-element tensors                           #
    # ------------------------------------------------------------------ #

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 191-212: test case nested single element tensor broadcast / 测试用例 nested single element tensor broadcast
```python
    def test_nested_single_element_tensor_broadcast(self):
        """A [[value]] shaped tensor (numel==1, ndim==2) should also broadcast."""
        module = _make_mock_module(output_sizes=[128, 128, 64])
        param = _make_per_tensor_scale_param(num_shards=3)

        calls = []

        def original_loader(p, chunk, shard_id):
            calls.append((shard_id, chunk.clone()))

        loader = Qwen3_5GatedDeltaNet._make_packed_weight_loader(
            module, original_loader
        )

        scale = torch.tensor([[0.75]])  # shape=[1,1], numel==1
        loader(param, scale, loaded_shard_id=(0, 1, 2))

        self.assertEqual(len(calls), 3)
        for shard_id, chunk in calls:
            # .view(-1) should flatten to [1]
            self.assertEqual(chunk.shape, torch.Size([1]))
            self.assertAlmostEqual(chunk.item(), 0.75, places=5)
```
**EN:** A [[value]] shaped tensor (numel==1, ndim==2) should also broadcast. This test exercises `test_nested_single_element_tensor_broadcast` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** A [[value]] shaped tensor (numel==1, ndim==2) should also broadcast. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nested_single_element_tensor_broadcast`。

### Lines 215-216: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_mock_module`: Create a lightweight mock module with the attributes needed by the loader. / 该代码块实现 `_make_mock_module`，承担模块行为中的一个聚焦逻辑片段。
- `_make_per_tensor_scale_param`: Create a PerTensorScaleParameter pre-allocated for `num_shards` scales. / 该代码块实现 `_make_per_tensor_scale_param`，承担模块行为中的一个聚焦逻辑片段。
- `TestMakePackedWeightLoader`: Tests for _make_packed_weight_loader broadcast / split logic. / 用于组织相关测试、夹具或辅助方法。
- `TestMakePackedWeightLoader.test_scalar_weight_broadcast`: A 0-d scalar should be broadcast (via .view(-1)) to every shard. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_scalar_weight_broadcast`。
- `TestMakePackedWeightLoader.test_single_element_tensor_broadcast`: A [1]-shaped tensor (e.g. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_element_tensor_broadcast`。
- `TestMakePackedWeightLoader.test_broadcast_with_two_shards`: Broadcast for in_proj_ba style (2 shards: b, a). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_broadcast_with_two_shards`。
- `TestMakePackedWeightLoader.test_normal_weight_split`: Multi-element weights should be split by output_sizes, not broadcast. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_normal_weight_split`。
- `TestMakePackedWeightLoader.test_int_shard_id_passthrough`: An int shard_id should bypass the tuple logic entirely. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_int_shard_id_passthrough`。
- `TestMakePackedWeightLoader.test_none_shard_id_passthrough`: None shard_id should pass through to the original loader. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_shard_id_passthrough`。
- `TestMakePackedWeightLoader.test_nested_single_element_tensor_broadcast`: A [[value]] shaped tensor (numel==1, ndim==2) should also broadcast. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nested_single_element_tensor_broadcast`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`, `unittest.mock`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.layers.parameter`, `sglang.srt.models.qwen3_5`

- **Total lines / 总行数**: 216
