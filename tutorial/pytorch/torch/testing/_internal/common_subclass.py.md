# common_subclass.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/common_subclass.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for common subclass, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 common subclass 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
# mypy: ignore-errors

import torch
from copy import deepcopy
from torch.utils._pytree import tree_map
import torch.utils._pytree as pytree

```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.utils._pytree`; external imports: `copy`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.utils._pytree`；外部导入：`copy`。

### Lines 8-14
```python

# TODO: Move LoggingTensor here.
from torch.testing._internal.logging_tensor import LoggingTensor


# Base class for wrapper-style tensors.
class WrapperTensor(torch.Tensor):
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `WrapperTensor`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`WrapperTensor`。

### Lines 15-23
```python
    @staticmethod
    def __new__(cls, *args, **kwargs):
        t, kwargs = cls.get_wrapper_properties(*args, **kwargs)
        if "size" not in kwargs:
            size = t.size()
        else:
            size = kwargs["size"]
            del kwargs["size"]
        if "dtype" not in kwargs:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__new__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__new__`。

### Lines 24-33
```python
            kwargs["dtype"] = t.dtype
        if "layout" not in kwargs:
            kwargs["layout"] = t.layout
        if "device" not in kwargs:
            kwargs["device"] = t.device
        if "requires_grad" not in kwargs:
            kwargs["requires_grad"] = False
        # Ignore memory_format and pin memory for now as I don't know how to
        # safely access them on a Tensor (if possible??)

```
- EN: This block handles tensor metadata or sample values; reuses computed state to reduce repeated work; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；复用已计算状态以减少重复工作；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 34-44
```python
        wrapper = torch.Tensor._make_wrapper_subclass(cls, size, **kwargs)
        wrapper._validate_methods()
        return wrapper

    @classmethod
    def get_wrapper_properties(cls, *args, **kwargs):
        # Should return both an example Tensor and a dictionary of kwargs
        # to override any of that example Tensor's properly.
        # This is very similar to the `t.new_*(args)` API
        raise NotImplementedError("You need to implement get_wrapper_properties")

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_wrapper_properties`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_wrapper_properties`。

### Lines 45-51
```python
    def _validate_methods(self):
        # Skip this if not in debug mode?
        # Changing these on the python side is wrong as it would not be properly reflected
        # on the c++ side
        # This doesn't catch attributes set in the __init__
        forbidden_overrides = ["size", "stride", "dtype", "layout", "device", "requires_grad"]
        for el in forbidden_overrides:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_validate_methods`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_validate_methods`。

### Lines 52-58
```python
            if getattr(self.__class__, el) is not getattr(torch.Tensor, el):
                raise RuntimeError(f"Subclass {self.__class__.__name__} is overwriting the "
                                   f"property {el} but this is not allowed as such change would "
                                   "not be reflected to c++ callers.")


class WrapperTensorWithCustomSizes(WrapperTensor):
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `WrapperTensorWithCustomSizes`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`WrapperTensorWithCustomSizes`。

### Lines 59-65
```python
    @classmethod
    def get_wrapper_properties(cls, t, requires_grad=False):
        return t, {"requires_grad": requires_grad, "dispatch_sizes_strides_policy": "sizes"}

    def __init__(self, t, requires_grad=False):
        self.t = t

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_wrapper_properties`, `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_wrapper_properties`, `__init__`。

### Lines 66-73
```python
    @classmethod
    def __torch_dispatch__(cls, func, types, args=(), kwargs=None):
        if not all(issubclass(cls, t) for t in types):
            return NotImplemented

        if kwargs is None:
            kwargs = {}

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__torch_dispatch__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__torch_dispatch__`。

### Lines 74-81
```python
        def unwrap(e):
            return e.t if isinstance(e, WrapperTensorWithCustomSizes) else e

        def wrap(e):
            return WrapperTensorWithCustomSizes(e) if isinstance(e, torch.Tensor) else e

        rs = tree_map(wrap, func(*tree_map(unwrap, args), **tree_map(unwrap, kwargs or {})))
        return rs
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `unwrap`, `wrap`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`unwrap`, `wrap`。

### Lines 82-88
```python

    def __repr__(self):
        return super().__repr__(tensor_contents=f"t={self.t}")


class WrapperTensorWithCustomStrides(WrapperTensor):
    @classmethod
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `__repr__`, `WrapperTensorWithCustomStrides`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`__repr__`, `WrapperTensorWithCustomStrides`。

### Lines 89-95
```python
    def get_wrapper_properties(cls, t, requires_grad=False):
        return t, {"requires_grad": requires_grad, "dispatch_sizes_strides_policy": "strides"}

    def __init__(self, t, requires_grad=False):
        self.t = t

    @classmethod
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_wrapper_properties`, `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_wrapper_properties`, `__init__`。

### Lines 96-102
```python
    def __torch_dispatch__(cls, func, types, args=(), kwargs=None):
        if not all(issubclass(cls, t) for t in types):
            return NotImplemented

        if kwargs is None:
            kwargs = {}

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__torch_dispatch__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__torch_dispatch__`。

### Lines 103-110
```python
        def unwrap(e):
            return e.t if isinstance(e, WrapperTensorWithCustomStrides) else e

        def wrap(e):
            return WrapperTensorWithCustomStrides(e) if isinstance(e, torch.Tensor) else e

        rs = tree_map(wrap, func(*tree_map(unwrap, args), **tree_map(unwrap, kwargs or {})))
        return rs
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `unwrap`, `wrap`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`unwrap`, `wrap`。

### Lines 111-117
```python

    def __repr__(self):
        return super().__repr__(tensor_contents=f"t={self.t}")


class DiagTensorBelow(WrapperTensor):
    @classmethod
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `__repr__`, `DiagTensorBelow`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`__repr__`, `DiagTensorBelow`。

### Lines 118-125
```python
    def get_wrapper_properties(cls, diag, requires_grad=False):
        if diag.ndim != 1:
            raise AssertionError(f"Expected diag.ndim == 1, got {diag.ndim}")
        return diag, {"size": diag.size() + diag.size(), "requires_grad": requires_grad}

    def __init__(self, diag, requires_grad=False):
        self.diag = diag

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_wrapper_properties`, `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_wrapper_properties`, `__init__`。

### Lines 126-132
```python
    handled_ops = {}

    @classmethod
    def __torch_dispatch__(cls, func, types, args=(), kwargs=None):
        if not all(issubclass(cls, t) for t in types):
            return NotImplemented

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__torch_dispatch__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__torch_dispatch__`。

### Lines 133-141
```python
        # For everything else, call the handler:
        fn = cls.handled_ops.get(func.__name__, None)
        if fn:
            return fn(*args, **(kwargs or {}))
        else:
            # Note that here, because we don't need to provide the autograd formulas
            # we can have a default "fallback" that creates a plain Tensor based
            # on the diag elements and calls the func again.

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 142-148
```python
            def unwrap(e):
                return e.diag.diag() if isinstance(e, DiagTensorBelow) else e

            def wrap(e):
                if isinstance(e, torch.Tensor) and e.ndim == 1:
                    return DiagTensorBelow(e)
                if isinstance(e, torch.Tensor) and e.ndim == 2 and e.count_nonzero() == e.diag().count_nonzero():
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `unwrap`, `wrap`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`unwrap`, `wrap`。

### Lines 149-155
```python
                    return DiagTensorBelow(e.diag())
                return e

            rs = tree_map(wrap, func(*tree_map(unwrap, args), **tree_map(unwrap, kwargs or {})))
            return rs

    def __repr__(self):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__repr__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__repr__`。

### Lines 156-162
```python
        return super().__repr__(tensor_contents=f"diag={self.diag}")


class SparseTensor(WrapperTensor):
    @classmethod
    def get_wrapper_properties(cls, size, values, indices, requires_grad=False):
        if values.device != indices.device:
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `SparseTensor`, `get_wrapper_properties`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`SparseTensor`, `get_wrapper_properties`。

### Lines 163-169
```python
            raise AssertionError(f"Expected values.device == indices.device, got {values.device} != {indices.device}")
        return values, {"size": size, "requires_grad": requires_grad}

    def __init__(self, size, values, indices, requires_grad=False):
        self.values = values
        self.indices = indices

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`。

### Lines 170-176
```python
    def __repr__(self):
        return super().__repr__(tensor_contents=f"values={self.values}, indices={self.indices}")

    def sparse_to_dense(self):
        res = torch.zeros(self.size(), dtype=self.values.dtype)
        res[self.indices.unbind(1)] = self.values
        return res
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__repr__`, `sparse_to_dense`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__repr__`, `sparse_to_dense`。

### Lines 177-183
```python

    @staticmethod
    def from_dense(t):
        indices = t.nonzero()
        values = t[indices.unbind(1)]
        return SparseTensor(t.size(), values, indices)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `from_dense`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`from_dense`。

### Lines 184-190
```python
    @classmethod
    def __torch_dispatch__(cls, func, types, args=(), kwargs=None):
        func_name = f"{func.__module__}.{func.__name__}"

        res = cls._try_call_special_impl(func_name, args, kwargs)
        if res is not NotImplemented:
            return res
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__torch_dispatch__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__torch_dispatch__`。

### Lines 191-198
```python

        # Otherwise, use a default implementation that construct dense
        # tensors and use that to compute values
        def unwrap(e):
            return e.sparse_to_dense() if isinstance(e, SparseTensor) else e

        # Wrap back all Tensors into our custom class
        def wrap(e):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `unwrap`, `wrap`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`unwrap`, `wrap`。

### Lines 199-205
```python
            # Check for zeros and use that to get indices
            return SparseTensor.from_dense(e) if isinstance(e, torch.Tensor) else e

        rs = tree_map(wrap, func(*tree_map(unwrap, args), **tree_map(unwrap, kwargs or {})))
        return rs


```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 206-212
```python
    _SPECIAL_IMPLS = {}

    @classmethod
    def _try_call_special_impl(cls, func, args, kwargs):
        if func not in cls._SPECIAL_IMPLS:
            return NotImplemented
        return cls._SPECIAL_IMPLS[func](args, kwargs)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_try_call_special_impl`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_try_call_special_impl`。

### Lines 213-222
```python


# Example non-wrapper subclass that stores extra state.
class NonWrapperTensor(torch.Tensor):
    def __new__(cls, data):
        t = torch.Tensor._make_subclass(cls, data)
        t.extra_state = {
            'last_func_called': None
        }
        return t
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `NonWrapperTensor`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`NonWrapperTensor`。

### Lines 223-231
```python

    @classmethod
    def __torch_function__(cls, func, types, args=(), kwargs=None):
        result = super().__torch_function__(func, types, args, kwargs)

        if isinstance(result, cls):
            # Do something with the extra state. For the example here, just store the name of the
            # last function called (skip for deepcopy so the copy has the same extra state).
            if func is torch.Tensor.__deepcopy__:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__torch_function__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__torch_function__`。

### Lines 232-238
```python
                result.extra_state = deepcopy(args[0].extra_state)
            else:
                result.extra_state = {
                    'last_func_called': func.__name__,
                }

        return result
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 239-246
```python

    # new_empty() must be defined for deepcopy to work
    def new_empty(self, shape):
        return type(self)(torch.empty(shape))


# Class used to store info about subclass tensors used in testing.
class SubclassInfo:
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `new_empty`, `SubclassInfo`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`new_empty`, `SubclassInfo`。

### Lines 247-254
```python

    __slots__ = ['name', 'create_fn', 'closed_under_ops']

    def __init__(self, name, create_fn, closed_under_ops=True):
        self.name = name
        self.create_fn = create_fn  # create_fn(shape) -> tensor instance
        self.closed_under_ops = closed_under_ops

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`。

### Lines 255-265
```python

# Helper function to create a subclass of the given class and possibly cache sizes / strides.
def _create_and_access_shape(cls, shape):
    sub = cls(torch.randn(shape))
    # NB: Wrapper subclasses with custom dispatched sizes / strides cache this info
    # on the first call via non-serializable PyCapsules. We purposefully trigger cache
    # population here for serialization / deepcopy tests to verify that the presence of this
    # cache info doesn't cause problems.
    sub.size()
    sub.stride()
    return sub
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_create_and_access_shape`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_create_and_access_shape`。

### Lines 266-279
```python


subclass_db = {
    torch.Tensor: SubclassInfo(
        'base_tensor', create_fn=torch.randn
    ),
    NonWrapperTensor: SubclassInfo(
        'non_wrapper_tensor',
        create_fn=lambda shape: NonWrapperTensor(torch.randn(shape))
    ),
    LoggingTensor: SubclassInfo(
        'logging_tensor',
        create_fn=lambda shape: LoggingTensor(torch.randn(shape))
    ),
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 280-293
```python
    SparseTensor: SubclassInfo(
        'sparse_tensor',
        create_fn=lambda shape: SparseTensor.from_dense(torch.randn(shape).relu())
    ),
    DiagTensorBelow: SubclassInfo(
        'diag_tensor_below',
        create_fn=lambda shape: DiagTensorBelow(torch.randn(shape)),
        closed_under_ops=False  # sparse semantics
    ),
    WrapperTensorWithCustomSizes: SubclassInfo(
        'wrapper_with_custom_sizes',
        create_fn=lambda shape: _create_and_access_shape(WrapperTensorWithCustomSizes, shape),
        closed_under_ops=False,
    ),
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 294-300
```python
    WrapperTensorWithCustomStrides: SubclassInfo(
        'wrapper_with_custom_strides',
        create_fn=lambda shape: _create_and_access_shape(WrapperTensorWithCustomStrides, shape),
        closed_under_ops=False,
    ),
}

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 301-313
```python
class SubclassWithTensorFactory(torch.Tensor):
    @staticmethod
    def __new__(cls, src):
        shape = src.shape
        kwargs = {}
        kwargs["strides"] = src.stride()
        kwargs["storage_offset"] = src.storage_offset()
        kwargs["device"] = src.device
        kwargs["layout"] = src.layout
        kwargs["requires_grad"] = src.requires_grad
        kwargs["dtype"] = src.dtype
        out = torch.Tensor._make_wrapper_subclass(cls, shape, **kwargs)
        return out
```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `SubclassWithTensorFactory`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`SubclassWithTensorFactory`。

### Lines 314-320
```python

    def __init__(self, src):
        self.src = src

    def __repr__(self):
        return f"{self.__class__.__name__}"

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`, `__repr__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`, `__repr__`。

### Lines 321-327
```python
    def __tensor_flatten__(self):
        return ["src"], None

    @classmethod
    def __tensor_unflatten__(cls, inner_tensors, meta, outer_size, outer_stride):
        src = inner_tensors["src"]
        return cls(src)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__tensor_flatten__`, `__tensor_unflatten__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__tensor_flatten__`, `__tensor_unflatten__`。

### Lines 328-334
```python

    @classmethod
    def __torch_dispatch__(cls, func, types, args, kwargs):
        if kwargs is None:
            kwargs = {}

        def _fn(x):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__torch_dispatch__`, `_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__torch_dispatch__`, `_fn`。

### Lines 335-341
```python
            return x.src * torch.ones(x.src.shape) if x.src.dtype == torch.float32 else x.src

        _args = pytree.tree_map_only(cls, _fn, args)
        _kwargs = pytree.tree_map_only(cls, _fn, kwargs)

        _out = func(*_args, **_kwargs)

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 342-345
```python
        _out_flat, _out_spec = pytree.tree_flatten(_out)

        out_flat = [cls(o) if isinstance(o, torch.Tensor) else o for o in _out_flat]
        return pytree.tree_unflatten(out_flat, _out_spec)
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Test maintainability — the file helps standardize expectations, fixtures, or generated coverage across PyTorch tests.
  CN: 测试可维护性——该文件有助于在 PyTorch 测试中统一期望、夹具或生成式覆盖。
- EN: Module integration — the implementation coordinates neighboring Python modules and utility layers.
  CN: 模块集成——该实现会协调相邻的 Python 模块与工具层。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.utils._pytree`, `torch.testing._internal.logging_tensor`
- External imports / 外部导入: `copy`
- Representative symbols / 代表性符号: `WrapperTensor`, `WrapperTensorWithCustomSizes`, `WrapperTensorWithCustomStrides`, `DiagTensorBelow`, `SparseTensor`, `NonWrapperTensor`, `SubclassInfo`, `_create_and_access_shape`, `SubclassWithTensorFactory`
