# test_adapters.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/test_adapters.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers model-facing behavior. The file defines 2 test(s), 0 fixture(s), and 17 helper/class block(s) to validate this area. / [CN] 该文件覆盖面向模型的行为。它定义了 2 个测试、0 个 fixture，以及 17 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L9)
```python
import pytest
import torch

from vllm.model_executor.models.adapters import _create_pooling_model_cls
from vllm.model_executor.models.utils import AutoWeightsLoader, StageMissingLayer
```
**EN:** Imports third-party packages like `pytest`, `torch`, project helpers such as `vllm.model_executor.models.adapters._create_pooling_model_cls`, `vllm.model_executor.models.utils.AutoWeightsLoader`, `vllm.model_executor.models.utils.StageMissingLayer`.
**CN:** 导入第三方包（如 `pytest`、`torch`）、项目内辅助模块（如 `vllm.model_executor.models.adapters._create_pooling_model_cls`、`vllm.model_executor.models.utils.AutoWeightsLoader`、`vllm.model_executor.models.utils.StageMissingLayer`）。

### Module setup / 模块级配置: pytestmark (L11-L11)
```python
pytestmark = pytest.mark.cpu_test
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `pytestmark`. Later helpers and tests reuse these values to keep scenarios concise. It also applies module-wide pytest markers.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `pytestmark`。后续辅助函数和测试会复用这些值，以减少重复。 它还会应用模块级的 pytest 标记。

### Class / 类: SimpleInnerModel (L14-L29)
```python
class SimpleInnerModel(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.embed = torch.nn.Linear(4, 8, bias=False)
        self.layer0 = torch.nn.Linear(8, 8, bias=False)
        self.layer1 = torch.nn.Linear(8, 8, bias=False)
        self.norm = torch.nn.Linear(8, 4, bias=False)

    def load_weights(self, weights):
        params = dict(self.named_parameters())
        loaded = set()
        for name, tensor in weights:
            if name in params:
                params[name].data.copy_(tensor)
                loaded.add(name)
        return loaded
```
**EN:** This class groups related scenarios in `SimpleInnerModel`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `__init__`, `load_weights`.
**CN:** 该类将与 `SimpleInnerModel` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `__init__`、`load_weights`。

### Helper method / 辅助方法: SimpleInnerModel.__init__ (L15-L20)
```python
    def __init__(self):
        super().__init__()
        self.embed = torch.nn.Linear(4, 8, bias=False)
        self.layer0 = torch.nn.Linear(8, 8, bias=False)
        self.layer1 = torch.nn.Linear(8, 8, bias=False)
        self.norm = torch.nn.Linear(8, 4, bias=False)
```
**EN:** This helper encapsulates reusable logic in `SimpleInnerModel.__init__`.
**CN:** 这个辅助函数将可复用逻辑封装在 `SimpleInnerModel.__init__` 中。

### Helper method / 辅助方法: SimpleInnerModel.load_weights (L22-L29)
```python
    def load_weights(self, weights):
        params = dict(self.named_parameters())
        loaded = set()
        for name, tensor in weights:
            if name in params:
                params[name].data.copy_(tensor)
                loaded.add(name)
        return loaded
```
**EN:** This helper encapsulates reusable logic in `SimpleInnerModel.load_weights`. Key inputs are `weights`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `SimpleInnerModel.load_weights` 中。 关键输入包括 `weights`。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: SimpleModel (L32-L40)
```python
class SimpleModel(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.model = SimpleInnerModel()
        self.lm_head = torch.nn.Linear(8, 16, bias=False)

    def load_weights(self, weights):
        loader = AutoWeightsLoader(self)
        return loader.load_weights(weights)
```
**EN:** This class groups related scenarios in `SimpleModel`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `__init__`, `load_weights`.
**CN:** 该类将与 `SimpleModel` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `__init__`、`load_weights`。

### Helper method / 辅助方法: SimpleModel.__init__ (L33-L36)
```python
    def __init__(self):
        super().__init__()
        self.model = SimpleInnerModel()
        self.lm_head = torch.nn.Linear(8, 16, bias=False)
```
**EN:** This helper encapsulates reusable logic in `SimpleModel.__init__`.
**CN:** 这个辅助函数将可复用逻辑封装在 `SimpleModel.__init__` 中。

### Helper method / 辅助方法: SimpleModel.load_weights (L38-L40)
```python
    def load_weights(self, weights):
        loader = AutoWeightsLoader(self)
        return loader.load_weights(weights)
```
**EN:** This helper encapsulates reusable logic in `SimpleModel.load_weights`. Key inputs are `weights`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `SimpleModel.load_weights` 中。 关键输入包括 `weights`。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: PackedWeightInnerModel (L43-L64)
```python
class PackedWeightInnerModel(torch.nn.Module):
    """Remaps q_proj/k_proj into a fused qkv_proj (Qwen2/Llama pattern)."""

    def __init__(self):
        super().__init__()
        self.qkv_proj = torch.nn.Linear(4, 16, bias=False)
        self.out = torch.nn.Linear(8, 4, bias=False)

    def load_weights(self, weights):
        params = dict(self.named_parameters())
        loaded = set()
        for name, tensor in weights:
            if name == "q_proj.weight":
                params["qkv_proj.weight"].data[:8].copy_(tensor)
                loaded.add("qkv_proj.weight")
            elif name == "k_proj.weight":
                params["qkv_proj.weight"].data[8:].copy_(tensor)
                loaded.add("qkv_proj.weight")
            elif name in params:
                params[name].data.copy_(tensor)
                loaded.add(name)
        return loaded
```
**EN:** This class groups related scenarios in `PackedWeightInnerModel`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `__init__`, `load_weights`.
**CN:** 该类将与 `PackedWeightInnerModel` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `__init__`、`load_weights`。

### Helper method / 辅助方法: PackedWeightInnerModel.__init__ (L46-L49)
```python
    def __init__(self):
        super().__init__()
        self.qkv_proj = torch.nn.Linear(4, 16, bias=False)
        self.out = torch.nn.Linear(8, 4, bias=False)
```
**EN:** This helper encapsulates reusable logic in `PackedWeightInnerModel.__init__`.
**CN:** 这个辅助函数将可复用逻辑封装在 `PackedWeightInnerModel.__init__` 中。

### Helper method / 辅助方法: PackedWeightInnerModel.load_weights (L51-L64)
```python
    def load_weights(self, weights):
        params = dict(self.named_parameters())
        loaded = set()
        for name, tensor in weights:
            if name == "q_proj.weight":
                params["qkv_proj.weight"].data[:8].copy_(tensor)
                loaded.add("qkv_proj.weight")
            elif name == "k_proj.weight":
                params["qkv_proj.weight"].data[8:].copy_(tensor)
                loaded.add("qkv_proj.weight")
            elif name in params:
                params[name].data.copy_(tensor)
                loaded.add(name)
        return loaded
```
**EN:** This helper encapsulates reusable logic in `PackedWeightInnerModel.load_weights`. Key inputs are `weights`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `PackedWeightInnerModel.load_weights` 中。 关键输入包括 `weights`。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: PackedWeightModel (L67-L75)
```python
class PackedWeightModel(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.model = PackedWeightInnerModel()
        self.lm_head = torch.nn.Linear(4, 8, bias=False)

    def load_weights(self, weights):
        loader = AutoWeightsLoader(self)
        return loader.load_weights(weights)
```
**EN:** This class groups related scenarios in `PackedWeightModel`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `__init__`, `load_weights`.
**CN:** 该类将与 `PackedWeightModel` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `__init__`、`load_weights`。

### Helper method / 辅助方法: PackedWeightModel.__init__ (L68-L71)
```python
    def __init__(self):
        super().__init__()
        self.model = PackedWeightInnerModel()
        self.lm_head = torch.nn.Linear(4, 8, bias=False)
```
**EN:** This helper encapsulates reusable logic in `PackedWeightModel.__init__`.
**CN:** 这个辅助函数将可复用逻辑封装在 `PackedWeightModel.__init__` 中。

### Helper method / 辅助方法: PackedWeightModel.load_weights (L73-L75)
```python
    def load_weights(self, weights):
        loader = AutoWeightsLoader(self)
        return loader.load_weights(weights)
```
**EN:** This helper encapsulates reusable logic in `PackedWeightModel.load_weights`. Key inputs are `weights`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `PackedWeightModel.load_weights` 中。 关键输入包括 `weights`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _buffer_reusing_iterator (L78-L89)
```python
def _buffer_reusing_iterator(weight_dict):
    """Yield weights through a shared buffer overwritten each step.

    Mimics ``runai_model_streamer`` with ``RUNAI_STREAMER_MEMORY_LIMIT=0``.
    """
    buf = None
    for name, tensor in weight_dict.items():
        if buf is None or buf.numel() < tensor.numel():
            buf = torch.empty(tensor.numel(), dtype=tensor.dtype)
        view = buf[: tensor.numel()].view(tensor.shape)
        view.copy_(tensor)
        yield name, view
```
**EN:** This helper encapsulates reusable logic in `_buffer_reusing_iterator`. Key inputs are `weight_dict`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_buffer_reusing_iterator` 中。 关键输入包括 `weight_dict`。

### Helper / 辅助函数: _make_pooling_model (L92-L97)
```python
def _make_pooling_model(base_cls=SimpleModel):
    PoolingModel = _create_pooling_model_cls(base_cls)
    model = base_cls()
    model.__class__ = PoolingModel
    model.lm_head = StageMissingLayer("output", model.lm_head)
    return model
```
**EN:** This helper encapsulates reusable logic in `_make_pooling_model`. Key inputs are `base_cls`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_make_pooling_model` 中。 关键输入包括 `base_cls`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _make_reference_weights (L100-L108)
```python
def _make_reference_weights():
    torch.manual_seed(42)
    return {
        "model.embed.weight": torch.randn(8, 4),
        "model.layer0.weight": torch.randn(8, 8),
        "model.layer1.weight": torch.randn(8, 8),
        "model.norm.weight": torch.randn(4, 8),
        "lm_head.weight": torch.randn(16, 8),
    }
```
**EN:** This helper encapsulates reusable logic in `_make_reference_weights`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_make_reference_weights` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _make_packed_reference_weights (L111-L118)
```python
def _make_packed_reference_weights():
    torch.manual_seed(42)
    return {
        "model.q_proj.weight": torch.randn(8, 4),
        "model.k_proj.weight": torch.randn(8, 4),
        "model.out.weight": torch.randn(4, 8),
        "lm_head.weight": torch.randn(8, 4),
    }
```
**EN:** This helper encapsulates reusable logic in `_make_packed_reference_weights`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_make_packed_reference_weights` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _load_and_compare (L121-L126)
```python
def _load_and_compare(model, ref, expected):
    for p in model.parameters():
        p.data.zero_()
    model.load_weights(_buffer_reusing_iterator(ref))
    for name, param in model.named_parameters():
        assert torch.equal(param.data, expected[name]), name
```
**EN:** This helper encapsulates reusable logic in `_load_and_compare`. Key inputs are `model`, `ref`, `expected`. The main assertion is `torch.equal(param.data, expected[name])`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_load_and_compare` 中。 关键输入包括 `model`、`ref`、`expected`。 核心断言是 `torch.equal(param.data, expected[name])`。

### Test / 测试: test_pooling_load_weights_with_buffer_reuse (L129-L137)
```python
def test_pooling_load_weights_with_buffer_reuse():
    """Ensure ModelForPooling.load_weights works with buffer-reusing iterators."""
    ref = _make_reference_weights()

    ground_truth = SimpleModel()
    ground_truth.load_weights(ref.items())
    expected = {n: p.data.clone() for n, p in ground_truth.named_parameters()}

    _load_and_compare(_make_pooling_model(), ref, expected)
```
**EN:** This test validates `test_pooling_load_weights_with_buffer_reuse`.
**CN:** 这个测试验证 `test_pooling_load_weights_with_buffer_reuse`。

### Test / 测试: test_pooling_load_weights_clones_probed_weights (L140-L148)
```python
def test_pooling_load_weights_clones_probed_weights():
    """Ensure probed weights survive buffer reuse during packed remapping."""
    ref = _make_packed_reference_weights()

    ground_truth = PackedWeightModel()
    ground_truth.load_weights(ref.items())
    expected = {n: p.data.clone() for n, p in ground_truth.named_parameters()}

    _load_and_compare(_make_pooling_model(PackedWeightModel), ref, expected)
```
**EN:** This test validates `test_pooling_load_weights_clones_probed_weights`.
**CN:** 这个测试验证 `test_pooling_load_weights_clones_probed_weights`。

## Key Concepts / 关键概念
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `pytest`, `torch`
- **Project / 项目内**: `vllm.model_executor.models.adapters._create_pooling_model_cls`, `vllm.model_executor.models.utils.AutoWeightsLoader`, `vllm.model_executor.models.utils.StageMissingLayer`
