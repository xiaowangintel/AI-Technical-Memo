# utils.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/opinfo/utils.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for utils, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 utils 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
# mypy: ignore-errors

import collections
import warnings
from collections.abc import Sequence
from functools import partial, wraps

```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: none; external imports: `collections`, `warnings`, `collections.abc`, `functools`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：无；外部导入：`collections`, `warnings`, `collections.abc`, `functools`。

### Lines 8-21
```python
import numpy as np
import numpy.typing as npt

import torch
from torch.testing._internal.common_cuda import TEST_CUDA
from torch.testing._internal.common_dtype import (
    _dispatch_dtypes,
    all_types,
    all_types_and,
    all_types_and_complex,
    all_types_and_complex_and,
    all_types_and_half,
    complex_types,
    floating_and_complex_types,
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 22-29
```python
    floating_and_complex_types_and,
    floating_types,
    floating_types_and,
    floating_types_and_half,
    integral_types,
    integral_types_and,
)
from torch.testing._internal.common_utils import torch_to_numpy_dtype_dict
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 30-42
```python


COMPLETE_DTYPES_DISPATCH = (
    all_types,
    all_types_and_complex,
    all_types_and_half,
    floating_types,
    floating_and_complex_types,
    floating_types_and_half,
    integral_types,
    complex_types,
)

```
- EN: This block handles tensor metadata or sample values. Key symbols: `COMPLETE_DTYPES_DISPATCH`.
- CN: 该代码块处理张量元数据或示例值。关键符号：`COMPLETE_DTYPES_DISPATCH`。

### Lines 43-50
```python
EXTENSIBLE_DTYPE_DISPATCH = (
    all_types_and_complex_and,
    floating_types_and,
    floating_and_complex_types_and,
    integral_types_and,
    all_types_and,
)

```
- EN: This block handles tensor metadata or sample values. Key symbols: `EXTENSIBLE_DTYPE_DISPATCH`.
- CN: 该代码块处理张量元数据或示例值。关键符号：`EXTENSIBLE_DTYPE_DISPATCH`。

### Lines 51-58
```python
# Better way to acquire devices?
DEVICES = ["cpu"] + (["cuda"] if TEST_CUDA else [])


class _dynamic_dispatch_dtypes(_dispatch_dtypes):
    # Class to tag the dynamically generated types.
    pass

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `DEVICES`, `_dynamic_dispatch_dtypes`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`DEVICES`, `_dynamic_dispatch_dtypes`。

### Lines 59-66
```python

def get_supported_dtypes(op, sample_inputs_fn, device_type):
    # Returns the supported dtypes for the given operator and device_type pair.
    if device_type not in ["cpu", "cuda"]:
        raise AssertionError(
            f"Expected device_type in ['cpu', 'cuda'], got {device_type!r}"
        )
    if not TEST_CUDA and device_type == "cuda":
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_supported_dtypes`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_supported_dtypes`。

### Lines 67-74
```python
        warnings.warn(
            "WARNING: CUDA is not available, empty_dtypes dispatch will be returned!",
            stacklevel=2,
        )
        return _dynamic_dispatch_dtypes(())

    supported_dtypes = set()
    for dtype in all_types_and_complex_and(torch.bool, torch.bfloat16, torch.half):
```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 75-87
```python
        try:
            samples = sample_inputs_fn(op, device_type, dtype, False)
        except RuntimeError:
            # If `sample_inputs_fn` doesn't support sampling for a given
            # `dtype`, we assume that the `dtype` is not supported.
            # We raise a warning, so that user knows that this was the case
            # and can investigate if there was an issue with the `sample_inputs_fn`.
            warnings.warn(
                f"WARNING: Unable to generate sample for device:{device_type} and dtype:{dtype}",
                stacklevel=2,
            )
            continue

```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 88-98
```python
        # We assume the dtype is supported
        # only if all samples pass for the given dtype.
        supported = True
        for sample in samples:
            try:
                op(sample.input, *sample.args, **sample.kwargs)
            except RuntimeError:
                # dtype is not supported
                supported = False
                break

```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 99-105
```python
        if supported:
            supported_dtypes.add(dtype)

    return _dynamic_dispatch_dtypes(supported_dtypes)


def dtypes_dispatch_hint(dtypes):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `dtypes_dispatch_hint`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`dtypes_dispatch_hint`。

### Lines 106-112
```python
    # Function returns the appropriate dispatch function (from COMPLETE_DTYPES_DISPATCH and EXTENSIBLE_DTYPE_DISPATCH)
    # and its string representation for the passed `dtypes`.
    return_type = collections.namedtuple("return_type", "dispatch_fn dispatch_fn_str")

    # CUDA is not available, dtypes will be empty.
    if len(dtypes) == 0:
        return return_type((), "()")
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 113-119
```python

    set_dtypes = set(dtypes)
    for dispatch in COMPLETE_DTYPES_DISPATCH:
        # Short circuit if we get an exact match.
        if set(dispatch()) == set_dtypes:
            return return_type(dispatch, dispatch.__name__ + "()")

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 120-126
```python
    chosen_dispatch = None
    chosen_dispatch_score = 0.0
    for dispatch in EXTENSIBLE_DTYPE_DISPATCH:
        dispatch_dtypes = set(dispatch())
        if not dispatch_dtypes.issubset(set_dtypes):
            continue

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 127-134
```python
        score = len(dispatch_dtypes)
        if score > chosen_dispatch_score:
            chosen_dispatch_score = score
            chosen_dispatch = dispatch

    # If user passed dtypes which are lower than the lowest
    # dispatch type available (not likely but possible in code path).
    if chosen_dispatch is None:
```
- EN: This block handles tensor metadata or sample values; bridges to backend-specific execution artifacts; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；桥接到特定后端的执行产物；处理条件控制流。关键符号：无明显局部符号。

### Lines 135-141
```python
        return return_type((), str(dtypes))

    return return_type(
        partial(dispatch, *tuple(set(dtypes) - set(dispatch()))),
        dispatch.__name__ + str(tuple(set(dtypes) - set(dispatch()))),
    )

```
- EN: This block handles tensor metadata or sample values; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 142-148
```python

def is_dynamic_dtype_set(op):
    # Detect if the OpInfo entry acquired dtypes dynamically
    # using `get_supported_dtypes`.
    return op.dynamic_dtypes


```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `is_dynamic_dtype_set`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`is_dynamic_dtype_set`。

### Lines 149-156
```python
def str_format_dynamic_dtype(op):
    fmt_str = f"""
        OpInfo({op.name},
               dtypes={dtypes_dispatch_hint(op.dtypes).dispatch_fn_str},
               dtypesIfCUDA={dtypes_dispatch_hint(op.dtypesIfCUDA).dispatch_fn_str},
        )
        """

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `str_format_dynamic_dtype`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`str_format_dynamic_dtype`。

### Lines 157-170
```python
    return fmt_str


def np_unary_ufunc_integer_promotion_wrapper(fn):
    # Wrapper that passes PyTorch's default scalar
    #   type as an argument to the wrapped NumPy
    #   unary ufunc when given an integer input.
    #   This mimics PyTorch's integer->floating point
    #   type promotion.
    #
    # This is necessary when NumPy promotes
    #   integer types to double, since PyTorch promotes
    #   integer types to the default scalar type.

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `np_unary_ufunc_integer_promotion_wrapper`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`np_unary_ufunc_integer_promotion_wrapper`。

### Lines 171-182
```python
    # Helper to determine if promotion is needed
    def is_integral(dtype):
        return dtype in [
            np.bool_,
            bool,
            np.uint8,
            np.int8,
            np.int16,
            np.int32,
            np.int64,
        ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `is_integral`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`is_integral`。

### Lines 183-189
```python
    @wraps(fn)
    def wrapped_fn(x):
        # As the default dtype can change, acquire it when function is called.
        # NOTE: Promotion in PyTorch is from integer types to the default dtype
        np_dtype = torch_to_numpy_dtype_dict[torch.get_default_dtype()]

        if is_integral(x.dtype):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `wrapped_fn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`wrapped_fn`。

### Lines 190-196
```python
            return fn(x.astype(np_dtype))
        return fn(x)

    return wrapped_fn


def reference_reduction_numpy(f, supports_keepdims=True):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_reduction_numpy`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_reduction_numpy`。

### Lines 197-208
```python
    """Wraps a NumPy reduction operator.

    The wrapper function will forward dim, keepdim, mask, and identity
    kwargs to the wrapped function as the NumPy equivalent axis,
    keepdims, where, and initiak kwargs, respectively.

    Args:
        f: NumPy reduction operator to wrap
        supports_keepdims (bool, optional): Whether the NumPy operator accepts
            keepdims parameter. If it does not, the wrapper will manually unsqueeze
            the reduced dimensions if it was called with keepdim=True. Defaults to True.

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 209-215
```python
    Returns:
        Wrapped function

    """

    @wraps(f)
    def wrapper(x: npt.NDArray, *args, **kwargs):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `wrapper`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`wrapper`。

### Lines 216-222
```python
        # Copy keys into a set
        keys = set(kwargs.keys())

        dim = kwargs.pop("dim", None)
        keepdim = kwargs.pop("keepdim", False)

        if "dim" in keys:
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 223-231
```python
            dim = tuple(dim) if isinstance(dim, Sequence) else dim

            # NumPy reductions don't accept dim=0 for scalar inputs
            # so we convert it to None if and only if dim is equivalent
            if x.ndim == 0 and dim in {0, -1, (0,), (-1,)}:
                kwargs["axis"] = None
            else:
                kwargs["axis"] = dim

```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 232-238
```python
        if "keepdim" in keys and supports_keepdims:
            kwargs["keepdims"] = keepdim

        if "mask" in keys:
            mask = kwargs.pop("mask")
            if mask is not None:
                if mask.layout != torch.strided:
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 239-246
```python
                    raise AssertionError(
                        f"Expected mask.layout == torch.strided, got {mask.layout}"
                    )
                kwargs["where"] = mask.cpu().numpy()

        if "identity" in keys:
            identity = kwargs.pop("identity")
            if identity is not None:
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 247-254
```python
                if identity.dtype is torch.bfloat16:
                    identity = identity.cpu().to(torch.float32)
                else:
                    identity = identity.cpu()
                kwargs["initial"] = identity.numpy()

        result = f(x, *args, **kwargs)

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 255-261
```python
        # Unsqueeze reduced dimensions if NumPy does not support keepdims
        if keepdim and not supports_keepdims and x.ndim > 0:
            dim = list(range(x.ndim)) if dim is None else dim
            result = np.expand_dims(result, dim)

        return result

```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 262-271
```python
    return wrapper


def prod_numpy(a, *args, **kwargs):
    """
    The function will call np.prod with type as np.int64 if the input type
    is int or uint64 if is uint. This is necessary because windows np.prod uses by default
    int32 while on linux it uses int64.
    This is for fixing integer overflow https://github.com/pytorch/pytorch/issues/77320

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `prod_numpy`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`prod_numpy`。

### Lines 272-280
```python
    Returns:
        np.prod of input
    """
    if "dtype" not in kwargs:
        if np.issubdtype(a.dtype, np.signedinteger):
            a = a.astype(np.int64)
        elif np.issubdtype(a.dtype, np.unsignedinteger):
            a = a.astype(np.uint64)

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 281-282
```python
    fn = reference_reduction_numpy(np.prod)
    return fn(a, *args, **kwargs)
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Backend coverage — the file contains backend-specific behavior or validation paths.
  CN: 后端覆盖——该文件包含特定后端的行为或校验路径。
- EN: Backend lowering — the code bridges higher-level programs to executable backend artifacts.
  CN: 后端降级——代码把更高层程序桥接到可执行的后端产物。
- EN: Test maintainability — the file helps standardize expectations, fixtures, or generated coverage across PyTorch tests.
  CN: 测试可维护性——该文件有助于在 PyTorch 测试中统一期望、夹具或生成式覆盖。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.testing._internal.common_cuda`, `torch.testing._internal.common_dtype`, `torch.testing._internal.common_utils`
- External imports / 外部导入: `collections`, `warnings`, `collections.abc`, `functools`, `numpy`, `numpy.typing`
- Representative symbols / 代表性符号: `COMPLETE_DTYPES_DISPATCH`, `EXTENSIBLE_DTYPE_DISPATCH`, `DEVICES`, `_dynamic_dispatch_dtypes`, `get_supported_dtypes`, `dtypes_dispatch_hint`, `is_dynamic_dtype_set`, `str_format_dynamic_dtype`, `np_unary_ufunc_integer_promotion_wrapper`, `reference_reduction_numpy`, `...`
