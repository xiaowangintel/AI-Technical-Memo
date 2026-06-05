# two_tensor.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/two_tensor.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for two tensor, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 two tensor 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```python
# mypy: ignore-errors

import torch
import torch.utils._pytree as pytree
from torch._export.wrappers import mark_subclass_constructor_exportable_experimental
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.utils._pytree`, `torch._export.wrappers`; external imports: none.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.utils._pytree`, `torch._export.wrappers`；外部导入：无。

### Lines 6-10
```python
from torch.utils._python_dispatch import return_and_correct_aliasing


# A simple tensor subclass that holds two tensors internally, and runs every op on both tensors.
class TwoTensor(torch.Tensor):
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.utils._python_dispatch`; external imports: none.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.utils._python_dispatch`；外部导入：无。

### Lines 11-15
```python
    @staticmethod
    def __new__(cls, a, b, outer_size=None, outer_stride=None, *, requires_grad=None):
        if outer_size is None:
            outer_size = a.size()
        if outer_stride is None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__new__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__new__`。

### Lines 16-25
```python
            outer_stride = a.stride()

        if not (
            a.device == b.device
            and a.layout == b.layout
            and a.requires_grad == b.requires_grad
            and a.dtype == b.dtype
        ):
            raise AssertionError(
                "Expected a and b to have same device, layout, requires_grad, and dtype"
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 26-35
```python
            )
        # I guess it would be more accurate to represent the shape as torch.cat(a, b).shape
        shape = outer_size
        kwargs = {}
        kwargs["strides"] = outer_stride
        kwargs["storage_offset"] = a.storage_offset()
        kwargs["device"] = a.device
        kwargs["layout"] = a.layout
        kwargs["requires_grad"] = requires_grad or a.requires_grad
        kwargs["dtype"] = a.dtype
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 36-42
```python
        out = torch.Tensor._make_wrapper_subclass(cls, shape, **kwargs)

        if a.shape != b.shape:
            raise AssertionError(
                f"Expected a.shape == b.shape, got {a.shape} != {b.shape}"
            )
        if a.stride() != b.stride():
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 43-51
```python
            raise AssertionError(
                f"Expected a.stride() == b.stride(), got {a.stride()} != {b.stride()}"
            )
        if a.storage_offset() != b.storage_offset():
            raise AssertionError(
                f"Expected a.storage_offset() == b.storage_offset(), "
                f"got {a.storage_offset()} != {b.storage_offset()}"
            )
        return out
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 52-58
```python

    @torch._disable_dynamo
    @mark_subclass_constructor_exportable_experimental
    def __init__(self, a, b, outer_size=None, outer_stride=None, *, requires_grad=None):
        self.a = a
        self.b = b

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`。

### Lines 59-63
```python
    def __repr__(self):
        a_repr = repr(self.a)
        b_repr = repr(self.b)
        return f"TwoTensor({a_repr}, {b_repr})"

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__repr__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__repr__`。

### Lines 64-68
```python
    def __tensor_flatten__(self):
        return ["a", "b"], None

    @staticmethod
    def __tensor_unflatten__(inner_tensors, meta, outer_size, outer_stride):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__tensor_flatten__`, `__tensor_unflatten__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__tensor_flatten__`, `__tensor_unflatten__`。

### Lines 69-73
```python
        if meta is not None:
            raise AssertionError("Expected meta to be None")
        a, b = inner_tensors["a"], inner_tensors["b"]
        if type(a) is torch.Tensor:
            if outer_size is None:
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 74-78
```python
                raise AssertionError("Expected outer_size to not be None")
            if outer_stride is None:
                raise AssertionError("Expected outer_stride to not be None")
        return TwoTensor(a, b, outer_size, outer_stride)

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 79-85
```python
    @classmethod
    def __torch_dispatch__(cls, func, types, args, kwargs):
        if kwargs is None:
            kwargs = {}
        args_a = pytree.tree_map_only(TwoTensor, lambda x: x.a, args)
        args_b = pytree.tree_map_only(TwoTensor, lambda x: x.b, args)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__torch_dispatch__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__torch_dispatch__`。

### Lines 86-95
```python
        kwargs_a = pytree.tree_map_only(TwoTensor, lambda x: x.a, kwargs)
        kwargs_b = pytree.tree_map_only(TwoTensor, lambda x: x.b, kwargs)

        out_a = func(*args_a, **kwargs_a)
        out_b = func(*args_b, **kwargs_b)
        out_a_flat, spec = pytree.tree_flatten(out_a)
        out_b_flat = pytree.tree_leaves(out_b)
        # for aten ops that return non-tensors, just assume that
        # our two inner tensors return the same value
        out_flat = [
```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 96-100
```python
            cls(o_a, o_b) if isinstance(o_a, torch.Tensor) else o_a
            for o_a, o_b in zip(out_a_flat, out_b_flat, strict=True)
        ]
        out = pytree.tree_unflatten(out_flat, spec)
        from torch._higher_order_ops.cond import cond_op
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 101-105
```python

        if func is cond_op:
            return out
        else:
            return return_and_correct_aliasing(func, args, kwargs, out)
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 106-110
```python

    def get_elem_a(self):
        return self.a


```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_elem_a`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_elem_a`。

### Lines 111-116
```python
class TwoTensorMode(torch.utils._python_dispatch.TorchDispatchMode):
    def __torch_dispatch__(self, func, types, args=(), kwargs=None):
        out = func(*args, **kwargs)
        if torch._subclasses.fake_tensor._is_tensor_constructor(func):
            out = TwoTensor(out, out.clone())
        return out
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `TwoTensorMode`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`TwoTensorMode`。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: ONNX export pipeline — the code translates PyTorch programs into ONNX-friendly representations.
  CN: ONNX 导出流水线——代码把 PyTorch 程序转换为适合 ONNX 的表示。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Test maintainability — the file helps standardize expectations, fixtures, or generated coverage across PyTorch tests.
  CN: 测试可维护性——该文件有助于在 PyTorch 测试中统一期望、夹具或生成式覆盖。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.utils._pytree`, `torch._export.wrappers`, `torch.utils._python_dispatch`, `torch._higher_order_ops.cond`
- External imports / 外部导入: 无
- Representative symbols / 代表性符号: `TwoTensor`, `TwoTensorMode`
