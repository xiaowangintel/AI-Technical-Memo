# _creation.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_creation.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Provides public testing utilities for creation, such as tensor construction, comparison, and test-facing helper APIs.
- 用途 (CN): 为 creation 提供公开测试工具，例如张量构造、结果比较以及面向测试的辅助 API。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
"""
This module contains tensor creation utilities.
"""

import collections.abc
import functools
import math
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 8-21
```python
import warnings
from typing import cast

import torch


_INTEGRAL_TYPES = [
    torch.uint8,
    torch.int8,
    torch.int16,
    torch.int32,
    torch.int64,
    torch.uint16,
    torch.uint32,
```
- EN: This block reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 22-34
```python
    torch.uint64,
]
_FLOATING_TYPES = [torch.float16, torch.bfloat16, torch.float32, torch.float64]
_FLOATING_8BIT_TYPES = [
    torch.float8_e4m3fn,
    torch.float8_e5m2,
    torch.float8_e4m3fnuz,
    torch.float8_e5m2fnuz,
]
_COMPLEX_TYPES = [torch.complex32, torch.complex64, torch.complex128]
_BOOLEAN_OR_INTEGRAL_TYPES = [torch.bool, *_INTEGRAL_TYPES]
_FLOATING_OR_COMPLEX_TYPES = [*_FLOATING_TYPES, *_COMPLEX_TYPES]

```
- EN: This block implements local helper logic for creation. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 creation 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 35-42
```python

def _uniform_random_(t: torch.Tensor, low: float, high: float) -> torch.Tensor:
    # uniform_ requires to-from <= std::numeric_limits<scalar_t>::max()
    # Work around this by scaling the range before and after the PRNG
    if high - low >= torch.finfo(t.dtype).max:
        return t.uniform_(low / 2, high / 2).mul_(2)
    else:
        return t.uniform_(low, high)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_uniform_random_`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_uniform_random_`。

### Lines 43-56
```python


def make_tensor(
    *shape: int | torch.Size | list[int] | tuple[int, ...],
    dtype: torch.dtype,
    device: str | torch.device,
    low: float | None = None,
    high: float | None = None,
    requires_grad: bool = False,
    noncontiguous: bool = False,
    exclude_zero: bool = False,
    memory_format: torch.memory_format | None = None,
) -> torch.Tensor:
    r"""Creates a tensor with the given :attr:`shape`, :attr:`device`, and :attr:`dtype`, and filled with
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `make_tensor`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`make_tensor`。

### Lines 57-63
```python
    values uniformly drawn from ``[low, high)``.

    If :attr:`low` or :attr:`high` are specified and are outside the range of the :attr:`dtype`'s representable
    finite values then they are clamped to the lowest or highest representable finite value, respectively.
    If ``None``, then the following table describes the default values for :attr:`low` and :attr:`high`,
    which depend on :attr:`dtype`.

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 64-77
```python
    +---------------------------+------------+----------+
    | ``dtype``                 | ``low``    | ``high`` |
    +===========================+============+==========+
    | boolean type              | ``0``      | ``2``    |
    +---------------------------+------------+----------+
    | unsigned integral type    | ``0``      | ``10``   |
    +---------------------------+------------+----------+
    | signed integral types     | ``-9``     | ``10``   |
    +---------------------------+------------+----------+
    | floating types            | ``-9``     | ``9``    |
    +---------------------------+------------+----------+
    | complex types             | ``-9``     | ``9``    |
    +---------------------------+------------+----------+

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 78-88
```python
    Args:
        shape (Tuple[int, ...]): Single integer or a sequence of integers defining the shape of the output tensor.
        dtype (:class:`torch.dtype`): The data type of the returned tensor.
        device (Union[str, torch.device]): The device of the returned tensor.
        low (Optional[Number]): Sets the lower limit (inclusive) of the given range. If a number is provided it is
            clamped to the least representable finite value of the given dtype. When ``None`` (default),
            this value is determined based on the :attr:`dtype` (see the table above). Default: ``None``.
        high (Optional[Number]): Sets the upper limit (exclusive) of the given range. If a number is provided it is
            clamped to the greatest representable finite value of the given dtype. When ``None`` (default) this value
            is determined based on the :attr:`dtype` (see the table above). Default: ``None``.

```
- EN: Declares C++ types or containers needed in this file. Representative symbols: no dominant local symbols.
- CN: 声明该文件需要的 C++ 类型或容器。代表性符号：无明显局部符号。

### Lines 89-102
```python
            .. deprecated:: 2.1

                Passing ``low==high`` to :func:`~torch.testing.make_tensor` for floating or complex types is deprecated
                since 2.1 and will be removed in 2.3. Use :func:`torch.full` instead.

        requires_grad (Optional[bool]): If autograd should record operations on the returned tensor. Default: ``False``.
        noncontiguous (Optional[bool]): If `True`, the returned tensor will be noncontiguous. This argument is
            ignored if the constructed tensor has fewer than two elements. Mutually exclusive with ``memory_format``.
        exclude_zero (Optional[bool]): If ``True`` then zeros are replaced with the dtype's small positive value
            depending on the :attr:`dtype`. For bool and integer types zero is replaced with one. For floating
            point types it is replaced with the dtype's smallest positive normal number (the "tiny" value of the
            :attr:`dtype`'s :func:`~torch.finfo` object), and for complex types it is replaced with a complex number
            whose real and imaginary parts are both the smallest positive normal number representable by the complex
            type. Default ``False``.
```
- EN: This block handles tensor metadata or sample values; reuses computed state to reduce repeated work; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；复用已计算状态以减少重复工作；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 103-112
```python
        memory_format (Optional[torch.memory_format]): The memory format of the returned tensor. Mutually exclusive
            with ``noncontiguous``.

    Raises:
        ValueError: If ``requires_grad=True`` is passed for integral `dtype`
        ValueError: If ``low >= high``.
        ValueError: If either :attr:`low` or :attr:`high` is ``nan``.
        ValueError: If both :attr:`noncontiguous` and :attr:`memory_format` are passed.
        TypeError: If :attr:`dtype` isn't supported by this function.

```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; reuses computed state to reduce repeated work; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；复用已计算状态以减少重复工作；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 113-126
```python
    Examples:
        >>> # xdoctest: +SKIP
        >>> # xdoctest: +REQUIRES(env:TORCH_DOCTEST_CUDA)
        >>> from torch.testing import make_tensor
        >>> # Creates a float tensor with values in [-1, 1)
        >>> make_tensor((3,), device="cpu", dtype=torch.float32, low=-1, high=1)
        >>> # xdoctest: +SKIP
        tensor([ 0.1205, 0.2282, -0.6380])
        >>> # Creates a bool tensor on CUDA
        >>> make_tensor((2, 2), device="cuda", dtype=torch.bool)
        tensor([[False, False],
                [False, True]], device='cuda:0')
    """

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 127-138
```python
    def modify_low_high(
        low: float | None,
        high: float | None,
        *,
        lowest_inclusive: float,
        highest_exclusive: float,
        default_low: float,
        default_high: float,
    ) -> tuple[float, float]:
        """
        Modifies (and raises ValueError when appropriate) low and high values given by the user (input_low, input_high)
        if required.
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `modify_low_high`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`modify_low_high`。

### Lines 139-146
```python
        """

        def clamp(a: float, l: float, h: float) -> float:
            return min(max(a, l), h)

        low = low if low is not None else default_low
        high = high if high is not None else default_high

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `clamp`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`clamp`。

### Lines 147-160
```python
        if any(isinstance(value, float) and math.isnan(value) for value in [low, high]):
            raise ValueError(
                f"`low` and `high` cannot be NaN, but got {low=} and {high=}"
            )
        elif low == high and dtype in _FLOATING_OR_COMPLEX_TYPES:
            warnings.warn(
                "Passing `low==high` to `torch.testing.make_tensor` for floating or complex types "
                "is deprecated since 2.1 and will be removed in 2.3. "
                "Use `torch.full(...)` instead.",
                FutureWarning,
                stacklevel=3,
            )
        elif low >= high:
            raise ValueError(f"`low` must be less than `high`, but got {low} >= {high}")
```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 161-169
```python
        elif high < lowest_inclusive or low >= highest_exclusive:
            raise ValueError(
                f"The value interval specified by `low` and `high` is [{low}, {high}), "
                f"but {dtype} only supports [{lowest_inclusive}, {highest_exclusive})"
            )

        low = clamp(low, lowest_inclusive, highest_exclusive)
        high = clamp(high, lowest_inclusive, highest_exclusive)

```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 170-176
```python
        if dtype in _BOOLEAN_OR_INTEGRAL_TYPES:
            # 1. `low` is ceiled to avoid creating values smaller than `low` and thus outside the specified interval
            # 2. Following the same reasoning as for 1., `high` should be floored. However, the higher bound of
            #    `torch.randint` is exclusive, and thus we need to ceil here as well.
            return math.ceil(low), math.ceil(high)

        return low, high
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 177-187
```python

    if len(shape) == 1 and isinstance(shape[0], collections.abc.Sequence):
        shape = shape[0]  # type: ignore[assignment]
    shape = cast(tuple[int, ...], tuple(shape))

    if noncontiguous and memory_format is not None:
        raise ValueError(
            f"The parameters `noncontiguous` and `memory_format` are mutually exclusive, "
            f"but got {noncontiguous=} and {memory_format=}"
        )

```
- EN: This block reports or normalizes error conditions; reuses computed state to reduce repeated work; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；复用已计算状态以减少重复工作；处理条件控制流。关键符号：无明显局部符号。

### Lines 188-194
```python
    if requires_grad and dtype in _BOOLEAN_OR_INTEGRAL_TYPES:
        raise ValueError(
            f"`requires_grad=True` is not supported for boolean and integral dtypes, but got {dtype=}"
        )

    noncontiguous = noncontiguous and functools.reduce(lambda x, y: x * y, shape, 1) > 1
    if noncontiguous:
```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 195-208
```python
        # Double the size of the shape in the last dimension, so that we have
        # non-identical values when we make the non-contiguous operation.
        shape = cast(tuple[int, ...], (*shape[:-1], 2 * shape[-1]))

    if dtype is torch.bool:
        low, high = cast(
            tuple[int, int],
            modify_low_high(
                low,
                high,
                lowest_inclusive=0,
                highest_exclusive=2,
                default_low=0,
                default_high=2,
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 209-222
```python
            ),
        )
        result = torch.randint(low, high, shape, device=device, dtype=dtype)
    elif dtype in _BOOLEAN_OR_INTEGRAL_TYPES:
        low, high = cast(
            tuple[int, int],
            modify_low_high(
                low,
                high,
                lowest_inclusive=torch.iinfo(dtype).min,
                highest_exclusive=torch.iinfo(dtype).max
                # In theory, `highest_exclusive` should always be the maximum value + 1. However, `torch.randint`
                # internally converts the bounds to an int64 and would overflow. In other words: `torch.randint` cannot
                # sample 2**63 - 1, i.e. the maximum value of `torch.int64` and we need to account for that here.
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 223-236
```python
                + (1 if dtype is not torch.int64 else 0),
                # This is incorrect for `torch.uint8`, but since we clamp to `lowest`, i.e. 0 for `torch.uint8`,
                # _after_ we use the default value, we don't need to special case it here
                default_low=-9,
                default_high=10,
            ),
        )
        result = torch.randint(low, high, shape, device=device, dtype=dtype)
    elif dtype in _FLOATING_OR_COMPLEX_TYPES:
        low, high = modify_low_high(
            low,
            high,
            lowest_inclusive=torch.finfo(dtype).min,
            highest_exclusive=torch.finfo(dtype).max,
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 237-250
```python
            default_low=-9,
            default_high=9,
        )
        result = torch.empty(shape, device=device, dtype=dtype)
        _uniform_random_(
            torch.view_as_real(result) if dtype in _COMPLEX_TYPES else result, low, high
        )
    elif dtype in _FLOATING_8BIT_TYPES:
        low, high = modify_low_high(
            low,
            high,
            lowest_inclusive=torch.finfo(dtype).min,
            highest_exclusive=torch.finfo(dtype).max,
            default_low=-9,
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 251-261
```python
            default_high=9,
        )
        result = torch.empty(shape, device=device, dtype=torch.float32)
        _uniform_random_(result, low, high)
        result = result.to(dtype)
    else:
        raise TypeError(
            f"The requested dtype '{dtype}' is not supported by torch.testing.make_tensor()."
            " To request support, file an issue at: https://github.com/pytorch/pytorch/issues"
        )

```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 262-268
```python
    if noncontiguous:
        # Offset by 1 to also catch offsetting issues
        result = result[..., 1::2]
    elif memory_format is not None:
        result = result.clone(memory_format=memory_format)

    if exclude_zero:
```
- EN: This block reuses computed state to reduce repeated work; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块复用已计算状态以减少重复工作；处理条件控制流。关键符号：无明显局部符号。

### Lines 269-275
```python
        result[result == 0] = (
            1 if dtype in _BOOLEAN_OR_INTEGRAL_TYPES else torch.finfo(dtype).tiny
        )

    if dtype in _FLOATING_OR_COMPLEX_TYPES:
        result.requires_grad = requires_grad

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 276-276
```python
    return result
```
- EN: This block returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块向调用方或后续阶段返回结果。关键符号：无明显局部符号。


## Key Concepts / 关键概念
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
- EN: Module integration — the implementation coordinates neighboring Python modules and utility layers.
  CN: 模块集成——该实现会协调相邻的 Python 模块与工具层。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`
- External imports / 外部导入: `collections.abc`, `functools`, `math`, `warnings`, `typing`
- Representative symbols / 代表性符号: `_INTEGRAL_TYPES`, `_FLOATING_TYPES`, `_FLOATING_8BIT_TYPES`, `_COMPLEX_TYPES`, `_BOOLEAN_OR_INTEGRAL_TYPES`, `_FLOATING_OR_COMPLEX_TYPES`, `_uniform_random_`, `make_tensor`
