# test_pooler_activations.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/model_executor/layers/test_pooler_activations.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Unit tests for vllm.model_executor.layers.pooler.activations. / 该文件主要围绕 Pooler Activations 组织测试，并验证关键行为与边界场景。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-19)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""Unit tests for vllm.model_executor.layers.pooler.activations."""

from types import SimpleNamespace

import pytest
import torch
import torch.nn as nn

from vllm.model_executor.layers.pooler.activations import (
    LambdaPoolerActivation,
    PoolerClassify,
    PoolerIdentity,
    PoolerMultiLabelClassify,
    PoolerNormalize,
    get_act_fn,
    resolve_classifier_act_fn,
)
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `types`, `pytest`, `torch`, `vllm.model_executor.layers.pooler.activations`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Class: TestPoolerIdentity (lines 25-38)
```python
class TestPoolerIdentity:
    def test_returns_input_unchanged(self):
        pooler = PoolerIdentity()
        x = torch.randn(4, 128)
        out = pooler(x)
        assert torch.equal(out, x)

    def test_forward_list(self):
        pooler = PoolerIdentity()
        tensors = [torch.randn(128), torch.randn(256)]
        out = pooler(tensors)
        assert len(out) == 2
        for orig, result in zip(tensors, out):
            assert torch.equal(orig, result)
```
**EN:** Groups related scenarios for Testpooleridentity. The class contains 2 test method(s).
**CN:** 该类把与 Testpooleridentity 相关的场景组织在一起。 其中包含 2 个测试方法。

### Class: TestPoolerNormalize (lines 44-65)
```python
class TestPoolerNormalize:
    def test_output_has_unit_norm(self):
        pooler = PoolerNormalize()
        x = torch.randn(4, 128)
        out = pooler(x)
        norms = torch.linalg.norm(out, dim=-1)
        assert torch.allclose(norms, torch.ones(4), atol=1e-5)

    def test_single_vector(self):
        pooler = PoolerNormalize()
        x = torch.randn(1, 64)
        out = pooler(x)
        norm = torch.linalg.norm(out, dim=-1)
        assert torch.allclose(norm, torch.ones(1), atol=1e-5)

    def test_forward_list(self):
        pooler = PoolerNormalize()
        tensors = [torch.randn(1, 64), torch.randn(1, 128)]
        out = pooler(tensors)
        for t in out:
            norm = torch.linalg.norm(t, dim=-1)
            assert torch.allclose(norm, torch.ones(1), atol=1e-5)
```
**EN:** Groups related scenarios for Testpoolernormalize. The class contains 3 test method(s).
**CN:** 该类把与 Testpoolernormalize 相关的场景组织在一起。 其中包含 3 个测试方法。

### Class: TestPoolerMultiLabelClassify (lines 71-88)
```python
class TestPoolerMultiLabelClassify:
    def test_output_in_zero_one(self):
        pooler = PoolerMultiLabelClassify()
        x = torch.randn(4, 10)
        out = pooler(x)
        assert (out >= 0).all() and (out <= 1).all()

    def test_large_positive_maps_near_one(self):
        pooler = PoolerMultiLabelClassify()
        x = torch.full((1, 3), 100.0)
        out = pooler(x)
        assert torch.allclose(out, torch.ones(1, 3), atol=1e-4)

    def test_large_negative_maps_near_zero(self):
        pooler = PoolerMultiLabelClassify()
        x = torch.full((1, 3), -100.0)
        out = pooler(x)
        assert torch.allclose(out, torch.zeros(1, 3), atol=1e-4)
```
**EN:** Groups related scenarios for Testpoolermultilabelclassify. The class contains 3 test method(s).
**CN:** 该类把与 Testpoolermultilabelclassify 相关的场景组织在一起。 其中包含 3 个测试方法。

### Class: TestPoolerClassify (lines 94-126)
```python
class TestPoolerClassify:
    def test_infers_from_shape_when_num_labels_none(self):
        pooler = PoolerClassify(num_labels=None)
        assert pooler.num_labels is None
        x = torch.randn(2, 5)
        out = pooler(x)
        sums = out.sum(dim=-1)
        assert torch.allclose(sums, torch.ones(2), atol=1e-5)

    def test_sigmoid_when_num_labels_lt_2(self):
        pooler = PoolerClassify(num_labels=1)
        x = torch.zeros(1, 1)
        out = pooler(x)
        assert torch.allclose(out, torch.tensor([[0.5]]), atol=1e-5)

    def test_num_labels_zero_uses_sigmoid(self):
        pooler = PoolerClassify(num_labels=0)
        assert pooler.num_labels == 0
        x = torch.zeros(1, 3)
        out = pooler(x)
        assert torch.allclose(out, torch.full((1, 3), 0.5), atol=1e-5)

    def test_num_labels_ge_2_uses_softmax(self):
        pooler = PoolerClassify(num_labels=4)
        assert pooler.num_labels == 4
        x = torch.randn(2, 4)
        out = pooler(x)
        sums = out.sum(dim=-1)
        assert torch.allclose(sums, torch.ones(2), atol=1e-5)

    def test_default_num_labels_is_none(self):
        pooler = PoolerClassify()
        assert pooler.num_labels is None
```
**EN:** Groups related scenarios for Testpoolerclassify. The class contains 5 test method(s).
**CN:** 该类把与 Testpoolerclassify 相关的场景组织在一起。 其中包含 5 个测试方法。

### Class: TestLambdaPoolerActivation (lines 132-145)
```python
class TestLambdaPoolerActivation:
    def test_applies_custom_fn(self):
        pooler = LambdaPoolerActivation(nn.ReLU())
        x = torch.tensor([[-1.0, 2.0, -3.0]])
        out = pooler(x)
        expected = torch.tensor([[0.0, 2.0, 0.0]])
        assert torch.equal(out, expected)

    def test_forward_list(self):
        pooler = LambdaPoolerActivation(nn.ReLU())
        tensors = [torch.tensor([-1.0, 2.0]), torch.tensor([3.0, -4.0])]
        out = pooler(tensors)
        assert torch.equal(out[0], torch.tensor([0.0, 2.0]))
        assert torch.equal(out[1], torch.tensor([3.0, 0.0]))
```
**EN:** Groups related scenarios for Testlambdapooleractivation. The class contains 2 test method(s).
**CN:** 该类把与 Testlambdapooleractivation 相关的场景组织在一起。 其中包含 2 个测试方法。

### Class: TestGetActFn (lines 151-216)
```python
class TestGetActFn:
    @staticmethod
    def _make_config(**kwargs):
        return SimpleNamespace(**kwargs)

    def test_regression(self):
        cfg = self._make_config(problem_type="regression")
        result = get_act_fn(cfg)
        assert isinstance(result, PoolerIdentity)

    def test_single_label_classification(self):
        cfg = self._make_config(
            problem_type="single_label_classification", num_labels=3
        )
        result = get_act_fn(cfg)
        assert isinstance(result, PoolerClassify)
        assert result.num_labels == 3

    def test_multi_label_classification(self):
# ... omitted for brevity ...
            sbert_ce_default_activation_function=(
                "torch.nn.modules.activation.Sigmoid"
            ),
        )
        result = get_act_fn(cfg)
        assert isinstance(result, PoolerIdentity)

    def test_rejects_non_torch_activation(self):
        cfg = self._make_config(
            problem_type="",
            sentence_transformers={"activation_fn": "os.system"},
        )
        with pytest.raises(AssertionError, match="restricted"):
            get_act_fn(cfg)
```
**EN:** Groups related scenarios for Testgetactfn. The class contains 8 test method(s) and 1 helper/setup method(s).
**CN:** 该类把与 Testgetactfn 相关的场景组织在一起。 其中包含 8 个测试方法，以及 1 个辅助或初始化方法。

### Class: TestResolveClassifierActFn (lines 222-234)
```python
class TestResolveClassifierActFn:
    def test_delegates_to_get_act_fn_when_none(self):
        model_config = SimpleNamespace(
            hf_config=SimpleNamespace(num_labels=3, problem_type="")
        )
        result = resolve_classifier_act_fn(model_config, act_fn=None)
        assert isinstance(result, PoolerClassify)
        assert result.num_labels == 3

    def test_passes_through_provided_act_fn(self):
        custom = PoolerIdentity()
        result = resolve_classifier_act_fn(None, act_fn=custom)
        assert result is custom
```
**EN:** Groups related scenarios for Testresolveclassifieractfn. The class contains 2 test method(s).
**CN:** 该类把与 Testresolveclassifieractfn 相关的场景组织在一起。 其中包含 2 个测试方法。

## Key Concepts / 关键概念
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。
- **Assertions / 断言**
  - **EN:** The file relies on explicit assertions to pin down expected state transitions and outputs.
  - **CN:** 该文件通过显式断言固定预期的状态变化与输出结果。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `types`
- **Third-party / 第三方依赖**: `pytest`, `torch`, `torch.nn`
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.layers.pooler.activations`
