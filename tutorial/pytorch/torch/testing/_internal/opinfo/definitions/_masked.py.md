# _masked.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/opinfo/definitions/_masked.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for masked, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 masked 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9
```python
# mypy: ignore-errors

import unittest
from collections.abc import Sequence
from functools import partial

import numpy as np

import torch
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`; external imports: `unittest`, `collections.abc`, `functools`, `numpy`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`；外部导入：`unittest`, `collections.abc`, `functools`, `numpy`。

### Lines 10-20
```python
from torch.testing import make_tensor
from torch.testing._internal.common_device_type import tol, toleranceOverride
from torch.testing._internal.common_dtype import (
    all_types_and,
    all_types_and_complex_and,
    complex_types,
    floating_and_complex_types_and,
    floating_types_and,
    integral_types,
)
from torch.testing._internal.opinfo.core import (
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 21-31
```python
    DecorateInfo,
    gradcheck_wrapper_masked_operation,
    gradcheck_wrapper_masked_pointwise_operation,
    M,
    OpInfo,
    ReductionOpInfo,
    S,
    sample_inputs_reduction,
    SampleInput,
)
from torch.testing._internal.opinfo.utils import prod_numpy, reference_reduction_numpy
```
- EN: This block checks invariants or expected outcomes. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果。关键符号：无明显局部符号。

### Lines 32-49
```python


# Used for log_softmax, softmax, softmin
def sample_inputs_softmax_variant(
    op_info,
    device,
    dtype,
    requires_grad,
    with_dtype=False,
    use_zero_dimensions=True,
    **kwargs,
):
    make_arg = partial(
        make_tensor, device=device, dtype=dtype, requires_grad=requires_grad
    )
    cases = [
        ((S,), (0,)),
        ((S, S), (0,)),
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_softmax_variant`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_softmax_variant`。

### Lines 50-61
```python
        ((S, S), (1,)),
        ((S, S), (-1,)),
        ((S, M, S), (2,)),
        *([((S, 0, 0), (-1,))] if use_zero_dimensions else []),
    ]
    kwargs = (
        dict(
            dtype=(
                torch.bfloat16 if torch.device(device).type == "mps" else torch.float64
            )
        )
        if with_dtype
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 62-70
```python
        else None
    )

    # PyTorch on XLA throws an error when passed with dim argument for 0d tensor.
    # See https://github.com/pytorch/xla/issues/3061 for more details.
    if torch.device(device).type != "xla":
        cases.append(((), (0,)))

    return (
```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 71-81
```python
        SampleInput(make_arg(shape), args=dim, kwargs=kwargs) for shape, dim in cases
    )


def _generate_masked_op_mask(input_shape, device, **kwargs):
    make_arg = partial(
        make_tensor, dtype=torch.bool, device=device, requires_grad=False
    )
    yield None
    yield make_arg(input_shape)
    if len(input_shape) > 2:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_generate_masked_op_mask`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_generate_masked_op_mask`。

### Lines 82-96
```python
        # broadcast last mask dimension:
        yield make_arg(input_shape[:-1] + (1,))
        # broadcast middle mask dimension:
        yield make_arg(input_shape[:1] + (1,) + input_shape[2:])
        # broadcast first mask dimension:
        yield make_arg((1,) + input_shape[1:])
        # mask.ndim < input.ndim
        yield make_arg(input_shape[1:])
        # mask.ndim == 1
        yield make_arg(input_shape[-1:])
        # masks that require broadcasting of inputs (mask.ndim >
        # input.ndim) will not be supported, however, we may
        # reconsider this if there will be demand on this kind of
        # degenerate cases.

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 97-106
```python

def sample_inputs_masked_reduction(op_info, device, dtype, requires_grad, **kwargs):
    """Sample inputs for masked reduction operators.

    Masked reduction operator is a reduction operator with trailing
    mask optional argument. A mask is a bool tensor with the same
    shape as input or a shape that is broadcastable to input shape.
    """
    kwargs["supports_multiple_dims"] = op_info.supports_multiple_dims

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_masked_reduction`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_masked_reduction`。

### Lines 107-122
```python
    for sample_input in sample_inputs_reduction(
        op_info, device, dtype, requires_grad, **kwargs
    ):
        for mask in _generate_masked_op_mask(
            sample_input.input.shape, device, **kwargs
        ):
            sample_input_args, sample_input_kwargs = (
                sample_input.args,
                dict(mask=mask, **sample_input.kwargs),
            )
            yield SampleInput(
                sample_input.input.detach().requires_grad_(requires_grad),
                args=sample_input_args,
                kwargs=sample_input_kwargs,
            )
            if (
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 123-137
```python
                not requires_grad
                and dtype.is_floating_point
                and sample_input.input.ndim == 2
                and mask is not None
                and mask.shape == sample_input.input.shape
            ):
                for v in [torch.inf, -torch.inf, torch.nan]:
                    t = sample_input.input.detach()
                    t.diagonal(0, -2, -1).fill_(v)
                    yield SampleInput(
                        t.requires_grad_(requires_grad),
                        args=sample_input_args,
                        kwargs=sample_input_kwargs,
                    )

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 138-147
```python

def sample_inputs_sparse_coo_masked_reduction(
    op_info, device, dtype, requires_grad, **kwargs
):
    """Sample inputs for masked reduction operators that support inputs
    with sparse coo layouts.
    """
    if op_info.supports_sparse:
        op_name = op_info.name.replace("masked.", "")
        for sample_input in sample_inputs_masked_reduction(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_sparse_coo_masked_reduction`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_sparse_coo_masked_reduction`。

### Lines 148-160
```python
            op_info, device, dtype, requires_grad, **kwargs
        ):
            mask = sample_input.kwargs.get("mask")
            if mask is not None:
                sample_input_kwargs = sample_input.kwargs.copy()
                sample_input_kwargs.update(mask=mask.to_sparse())
                yield SampleInput(
                    sample_input.input.to_sparse(),
                    args=sample_input.args,
                    kwargs=sample_input_kwargs,
                )
            else:
                if op_name in {"prod", "amax", "amin"}:
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 161-171
```python
                    # FIXME: for now reductions with non-zero reduction identity and
                    # unspecified mask are not supported for sparse COO
                    # tensors, see torch.masked.prod implementation
                    # for details.
                    continue
                yield SampleInput(
                    sample_input.input.to_sparse(),
                    args=sample_input.args,
                    kwargs=sample_input.kwargs,
                )

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 172-181
```python

def sample_inputs_sparse_csr_masked_reduction(
    op_info, device, dtype, requires_grad, **kwargs
):
    """Sample inputs for masked reduction operators that support inputs
    with sparse csr layouts.
    """
    if op_info.supports_sparse_csr:
        op_name = op_info.name.replace("masked.", "")
        for sample_input in sample_inputs_masked_reduction(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_sparse_csr_masked_reduction`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_sparse_csr_masked_reduction`。

### Lines 182-191
```python
            op_info, device, dtype, requires_grad, **kwargs
        ):
            if not (
                sample_input.input.ndim == 2 and sample_input.kwargs.get("keepdim")
            ):
                # - sparse CSR tensors are always 2-D tensors
                # - masked reduction on CSR tensors are defined only if keepdim is True.
                continue
            mask = sample_input.kwargs.get("mask")
            if mask is not None:
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 192-200
```python
                sample_input_kwargs = sample_input.kwargs.copy()
                sample_input_kwargs.update(mask=mask.to_sparse_csr())
                new_sample = SampleInput(
                    sample_input.input.to_sparse_csr(),
                    args=sample_input.args,
                    kwargs=sample_input_kwargs,
                )
            else:
                if op_name in ["prod", "amax", "amin", "mean"]:
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 201-212
```python
                    # reductions with non-zero reduction identity and
                    # unspecified mask is not supported for sparse CSR
                    # tensors, see torch.masked.prod implementation
                    # for details.
                    continue
                new_sample = SampleInput(
                    sample_input.input.to_sparse_csr(),
                    args=sample_input.args,
                    kwargs=sample_input.kwargs,
                )
            yield new_sample
            if sample_input.kwargs["dim"] == 0:
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 213-227
```python
                # Reductions of CSR tensors use different implementations for
                # inner and/or outer dimensions. So, as a minimum of testing CSR
                # implementations the following kwargs must be generated:
                #   dict(dim=0, keepdim=True)
                #   dict(dim=1, keepdim=True)
                #   dict(dim=(0, 1), keepdim=True)
                # Here we generate the dim=1 case from the dim=0 case.
                sample_input_kwargs = new_sample.kwargs.copy()
                sample_input_kwargs.update(dim=1)
                yield SampleInput(
                    new_sample.input.clone(),
                    args=sample_input.args,
                    kwargs=sample_input_kwargs,
                )

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 228-244
```python

def sample_inputs_masked_norm(op_info, device, dtype, requires_grad, **kwargs):
    """Sample inputs for masked norm."""
    for ord in [2.0, 1, float("inf"), float("-inf"), 0]:
        for sample_input in sample_inputs_masked_reduction(
            op_info, device, dtype, requires_grad, **kwargs
        ):
            sample_input_args, sample_input_kwargs = (
                (ord,) + sample_input.args,
                sample_input.kwargs.copy(),
            )
            yield SampleInput(
                sample_input.input.clone().requires_grad_(requires_grad),
                args=sample_input_args,
                kwargs=sample_input_kwargs,
            )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_masked_norm`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_masked_norm`。

### Lines 245-261
```python

def reference_masked_std_var(
    numpy_fn,
):
    ref = reference_reduction_numpy(numpy_fn)

    # Translate unbiased or correction arguments into ddof
    def func(
        input,
        dim=None,
        unbiased=None,
        *,
        correction=None,
        **kwargs,
    ):
        ddof = 1
        if unbiased is not None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_masked_std_var`, `func`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_masked_std_var`, `func`。

### Lines 262-270
```python
            ddof = 1 if unbiased else 0
        if correction is not None:
            ddof = correction

        if isinstance(dim, Sequence):
            dim = tuple(dim)

        return ref(input, dim, ddof=ddof, **kwargs)

```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 271-279
```python
    return func


def sample_inputs_masked_std_var(op_info, device, dtype, requires_grad, **kwargs):
    """Sample inputs for masked std/var."""
    kwargs["supports_multiple_dims"] = op_info.supports_multiple_dims
    from torch.testing._internal.common_methods_invocations import sample_inputs_std_var

    def masked_samples():
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_masked_std_var`, `masked_samples`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_masked_std_var`, `masked_samples`。

### Lines 280-297
```python
        for sample_input in sample_inputs_std_var(
            op_info, device, dtype, requires_grad, **kwargs
        ):
            if len(sample_input.args) and isinstance(sample_input.args[0], bool):
                continue  # masked.{std, var} doesn't support `.var(unbiased)`

            for mask in _generate_masked_op_mask(
                sample_input.input.shape, device, **kwargs
            ):
                sample_input_args, sample_input_kwargs = (
                    sample_input.args,
                    dict(mask=mask, **sample_input.kwargs),
                )
                yield SampleInput(
                    sample_input.input.detach().requires_grad_(requires_grad),
                    args=sample_input_args,
                    kwargs=sample_input_kwargs,
                )
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 298-313
```python
                if (
                    not requires_grad
                    and dtype.is_floating_point
                    and sample_input.input.ndim == 2
                    and mask is not None
                    and mask.shape == sample_input.input.shape
                ):
                    for v in [torch.inf, -torch.inf, torch.nan]:
                        t = sample_input.input.detach()
                        t.diagonal(0, -2, -1).fill_(v)
                        yield SampleInput(
                            t.requires_grad_(requires_grad),
                            args=sample_input_args,
                            kwargs=sample_input_kwargs,
                        )

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 314-331
```python
    for sample_input in masked_samples():
        correction = sample_input.kwargs.get("correction")
        if correction is None:
            correction = int(sample_input.kwargs.get("unbiased", True))

        dim = sample_input.kwargs.get("dim", None)

        if sample_input.kwargs.get("mask") is None:
            orig_count = torch.masked.sum(
                torch.ones(sample_input.input.shape, dtype=torch.int64),
                dim,
                keepdim=True,
            )
        else:
            inmask = torch.masked._input_mask(
                sample_input.input, *sample_input.args, **sample_input.kwargs
            )
            orig_count = torch.masked.sum(
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 332-340
```python
                inmask.new_ones(sample_input.input.shape, dtype=torch.int64),
                dim,
                keepdim=True,
                mask=inmask,
            )
        if orig_count.min() <= correction + 1:
            # Skip samples that lead to nans in var computation
            continue

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 341-354
```python
        yield sample_input


def sample_inputs_masked_softmax(
    op_info, device, dtype, requires_grad, with_dtype=False, **kwargs
):
    """Sample inputs for masked softmax, log_softmax, and softmin.

    Masked normalization operator is a reduction operator with
    trailing mask optional argument. A mask is a bool tensor with the
    same shape as input or a shape that is broadcastable to input
    shape.
    """
    for sample_input in sample_inputs_softmax_variant(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_masked_softmax`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_masked_softmax`。

### Lines 355-366
```python
        op_info, device, dtype, requires_grad, with_dtype=with_dtype, **kwargs
    ):
        for mask in _generate_masked_op_mask(
            sample_input.input.shape, device, **kwargs
        ):
            yield SampleInput(
                sample_input.input.clone().requires_grad_(requires_grad),
                *sample_input.args,
                mask=mask,
                **sample_input.kwargs,
            )

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 367-376
```python

def sample_inputs_masked_cumops(op_info, device, dtype, requires_grad, **kwargs):
    """Sample inputs for masked cumsum and cumprod."""
    for sample_input in sample_inputs_softmax_variant(
        op_info, device, dtype, requires_grad, **kwargs
    ):
        for mask in _generate_masked_op_mask(
            sample_input.input.shape, device, **kwargs
        ):
            if type(mask) is not torch.Tensor:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_masked_cumops`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_masked_cumops`。

### Lines 377-385
```python
                continue
            sample_input_args, sample_input_kwargs = (
                sample_input.args,
                dict(mask=mask, **sample_input.kwargs),
            )
            if "keepdim" in sample_input_kwargs:
                sample_input_kwargs.pop("keepdim")
            # dimension is required
            if sample_input_args:
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 386-397
```python
                dim = sample_input.args[0]
            else:
                if "dim" not in sample_input_kwargs:
                    continue
                dim = sample_input_kwargs.pop("dim")
                sample_input_args = (dim,)
            yield SampleInput(
                sample_input.input.clone().requires_grad_(requires_grad),
                *sample_input_args,
                **sample_input_kwargs,
            )

```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 398-408
```python

def sample_inputs_masked_logaddexp(op_info, device, dtype, requires_grad, **kwargs):
    """Sample inputs for masked logaddexp."""
    shapes = [(S,), (S, S), (S, M, S)]
    input_mask_lists = [
        list(_generate_masked_op_mask(shape, device, **kwargs)) for shape in shapes
    ]
    other_mask_lists = [
        list(_generate_masked_op_mask(shape, device, **kwargs)) for shape in shapes
    ]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_masked_logaddexp`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_masked_logaddexp`。

### Lines 409-422
```python
    make_arg = partial(
        make_tensor, dtype=dtype, device=device, requires_grad=requires_grad
    )
    for shape, input_masks, other_masks in zip(
        shapes, input_mask_lists, other_mask_lists, strict=True
    ):
        for input_mask, other_mask in zip(input_masks, other_masks, strict=True):
            yield SampleInput(
                make_arg(shape),
                make_arg(shape),
                input_mask=input_mask,
                other_mask=other_mask,
            )

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 423-436
```python

def sample_inputs_masked_normalize(op_info, device, dtype, requires_grad, **kwargs):
    """Sample inputs for masked normalize."""
    for ord in [2.0, 1, float("inf"), float("-inf"), 0]:
        for sample_input in sample_inputs_softmax_variant(
            op_info, device, dtype, requires_grad, use_zero_dimensions=False, **kwargs
        ):
            yield SampleInput(
                sample_input.input.clone().requires_grad_(requires_grad),
                ord,
                *sample_input.args,
                **sample_input.kwargs,
            )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_masked_normalize`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_masked_normalize`。

### Lines 437-454
```python

op_db: list[OpInfo] = [
    ReductionOpInfo(
        "masked.sum",
        ref=reference_reduction_numpy(np.sum),
        method_variant=None,
        identity=0,
        nan_policy="propagate",
        supports_out=False,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        supports_sparse=True,
        supports_sparse_csr=True,
        promotes_int_to_int64=True,
        dtypes=all_types_and_complex_and(torch.bool, torch.float16, torch.bfloat16),
        skips=(
            DecorateInfo(
                unittest.skip("Failing on some jobs"),
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 455-472
```python
                "TestReductions",
                "test_reference_masked",
                dtypes=(torch.bool, torch.int8, torch.int16, torch.int32),
            ),
            # FIXME: improve precision
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
                "test_reference_masked",
                dtypes=(torch.float16,),
            ),
            DecorateInfo(
                unittest.expectedFailure,
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            # FIXME: sum reduces all dimensions when dim=[]
            DecorateInfo(unittest.expectedFailure, "TestReductions", "test_dim_empty"),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 473-490
```python
            DecorateInfo(
                unittest.expectedFailure, "TestReductions", "test_dim_empty_keepdim"
            ),
            # RuntimeError: undefined value tensor
            DecorateInfo(
                unittest.expectedFailure, "TestJit", "test_variant_consistency_jit"
            ),
            # Driver issue of XPU, see https://github.com/intel/torch-xpu-ops/issues/2295
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
                "test_ref_small_input",
                device_type="xpu",
                dtypes=[torch.complex128],
            ),
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 491-508
```python
                "test_reference_masked",
                device_type="xpu",
                dtypes=[torch.complex128],
            ),
        ),
        decorators=[
            DecorateInfo(
                toleranceOverride(
                    {
                        torch.bfloat16: tol(atol=1e-03, rtol=5e-2),
                        torch.float16: tol(atol=1e-03, rtol=5e-3),
                    }
                ),
                "TestReductions",
                "test_reference_masked",
            ),
            DecorateInfo(
                toleranceOverride({torch.float16: tol(atol=1e-02, rtol=1e-03)}),
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 509-526
```python
                "TestReductions",
                "test_ref_small_input",
            ),
            DecorateInfo(
                toleranceOverride(
                    {
                        torch.bfloat16: tol(atol=0.1, rtol=0.1),
                        torch.float16: tol(atol=5e-3, rtol=5e-3),
                    }
                ),
                "TestMasked",
                "test_mask_layout",
            ),
        ],
        sample_inputs_func=sample_inputs_masked_reduction,
        sample_inputs_sparse_coo_func=sample_inputs_sparse_coo_masked_reduction,
        sample_inputs_sparse_csr_func=sample_inputs_sparse_csr_masked_reduction,
    ),
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 527-544
```python
    ReductionOpInfo(
        "masked.prod",
        ref=prod_numpy,
        method_variant=None,
        identity=1,
        nan_policy="propagate",
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_out=False,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        supports_sparse=True,
        supports_sparse_csr=True,
        promotes_int_to_int64=True,
        dtypes=all_types_and_complex_and(torch.bool, torch.float16, torch.bfloat16),
        skips=(
            DecorateInfo(
                unittest.expectedFailure,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 545-562
```python
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            DecorateInfo(
                unittest.expectedFailure, "TestJit", "test_variant_consistency_jit"
            ),
            DecorateInfo(
                unittest.skip("Failing on some jobs"),
                "TestReductions",
                "test_reference_masked",
                dtypes=(torch.bool, torch.int8, torch.int16, torch.int32),
            ),
            DecorateInfo(
                "TestReductions",
                "test_ref_small_input",
                dtypes=(torch.int8, torch.int16, torch.int32),
            ),
            # FIXME: "cuda_scatter_gather_base_kernel_func" not implemented for ... (used for sparse_coo inputs)
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 563-580
```python
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestMasked",
                "test_mask_layout",
                device_type="cuda",
                dtypes=(torch.bool, *integral_types(), *complex_types()),
            ),
            # Driver issue of XPU, see https://github.com/intel/torch-xpu-ops/issues/2295
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
                "test_reference_masked",
                device_type="xpu",
                dtypes=(torch.complex128, torch.int64, torch.uint8),
            ),
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 581-598
```python
                "test_ref_small_input",
                device_type="xpu",
                dtypes=(
                    torch.int64,
                    torch.int32,
                    torch.int16,
                    torch.int8,
                    torch.complex128,
                ),
            ),
        ),
        decorators=[
            DecorateInfo(
                toleranceOverride({torch.float16: tol(atol=1e-03, rtol=1e-02)}),
                "TestReductions",
                "test_reference_masked",
            ),
            DecorateInfo(
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 599-616
```python
                toleranceOverride({torch.float16: tol(atol=1e-03, rtol=1e-03)}),
                "TestReductions",
                "test_ref_duplicate_values",
            ),
            DecorateInfo(
                toleranceOverride({torch.float16: tol(atol=1e-03, rtol=1e-03)}),
                "TestReductions",
                "test_ref_small_input",
            ),
            DecorateInfo(
                toleranceOverride({torch.float16: tol(atol=1e-02, rtol=1.5e-03)}),
                "TestMasked",
                "test_mask_layout",
                device_type="cpu",
            ),
            DecorateInfo(
                toleranceOverride({torch.float32: tol(atol=1e-05, rtol=1e-05)}),
                "TestOperators",
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 617-634
```python
                "test_jvp",
                device_type="cuda",
            ),
        ],
        sample_inputs_func=sample_inputs_masked_reduction,
        sample_inputs_sparse_coo_func=sample_inputs_sparse_coo_masked_reduction,
        sample_inputs_sparse_csr_func=sample_inputs_sparse_csr_masked_reduction,
    ),
    OpInfo(
        "masked.cumsum",
        dtypes=all_types_and_complex_and(torch.float16, torch.bfloat16),
        method_variant=None,
        # Runs very slowly on slow gradcheck - alternatively reduce input sizes
        gradcheck_fast_mode=True,
        supports_out=False,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        skips=(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 635-652
```python
            DecorateInfo(
                unittest.expectedFailure,
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            # NotSupportedError: Compiled functions can't ... use keyword-only arguments with defaults
            DecorateInfo(
                unittest.skip("Skipped!"), "TestJit", "test_variant_consistency_jit"
            ),
            # The following dtypes worked in forward but are not listed by the OpInfo: {torch.bool}.
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_dtypes", device_type="mps"
            ),
        ),
        # Can reuse the same inputs; dim is required in both
        sample_inputs_func=sample_inputs_masked_cumops,
        gradcheck_wrapper=gradcheck_wrapper_masked_operation,
    ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; reuses computed state to reduce repeated work. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；复用已计算状态以减少重复工作。关键符号：无明显局部符号。

### Lines 653-670
```python
    OpInfo(
        "masked.cumprod",
        dtypes=all_types_and_complex_and(torch.float16, torch.bfloat16),
        method_variant=None,
        # Runs very slowly on slow gradcheck - alternatively reduce input sizes
        gradcheck_fast_mode=True,
        supports_out=False,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        skips=(
            # NotSupportedError: Compiled functions can't ... use keyword-only arguments with defaults
            DecorateInfo(
                unittest.expectedFailure,
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            # NotSupportedError: Compiled functions can't ... use keyword-only arguments with defaults
            DecorateInfo(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 671-688
```python
                unittest.skip("Skipped!"), "TestJit", "test_variant_consistency_jit"
            ),
            DecorateInfo(
                toleranceOverride({torch.float32: tol(atol=1e-5, rtol=1e-5)}),
                "TestCompositeCompliance",
                "test_backward",
                device_type="cuda",
            ),
            DecorateInfo(
                toleranceOverride({torch.float16: tol(atol=1e-2, rtol=2.6e-3)}),
                "TestInductorOpInfo",
                "test_comprehensive",
                device_type="cuda",
            ),
            # The following dtypes worked in forward but are not listed by the OpInfo: {torch.bool}.
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_dtypes", device_type="mps"
            ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; bridges to backend-specific execution artifacts. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；桥接到特定后端的执行产物。关键符号：无明显局部符号。

### Lines 689-706
```python
        ),
        # Can reuse the same inputs; dim is required in both
        sample_inputs_func=sample_inputs_masked_cumops,
        gradcheck_wrapper=gradcheck_wrapper_masked_operation,
    ),
    ReductionOpInfo(
        "masked.amax",
        nan_policy="propagate",
        supports_out=False,
        dtypes=all_types_and(torch.float16, torch.bfloat16),
        supports_sparse=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        supports_sparse_csr=True,
        ref=reference_reduction_numpy(np.amax),
        skips=(
            DecorateInfo(
                unittest.expectedFailure,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reuses computed state to reduce repeated work. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；复用已计算状态以减少重复工作。关键符号：无明显局部符号。

### Lines 707-724
```python
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            # FIXME: amax reduces all dimensions when dim=[]
            DecorateInfo(unittest.expectedFailure, "TestReductions", "test_dim_empty"),
            DecorateInfo(
                unittest.expectedFailure, "TestReductions", "test_dim_empty_keepdim"
            ),
            # RuntimeError: Unknown builtin op: aten::iinfo
            DecorateInfo(
                unittest.skip("Skipped!"), "TestJit", "test_variant_consistency_jit"
            ),
            # FIXME: "cuda_scatter_gather_base_kernel_func" not implemented for ... (used for sparse_coo inputs)
            # FIXME: "_segment_reduce_lengths_cpu/cuda" not implemented for ... (used for sparse_csr inputs)
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestMasked",
                "test_mask_layout",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 725-742
```python
                dtypes=(torch.bool, *integral_types(), *complex_types()),
            ),
            # Driver issue of XPU, see https://github.com/intel/torch-xpu-ops/issues/2295
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
                "test_reference_masked",
                device_type="xpu",
                dtypes=[torch.int64],
            ),
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
                "test_ref_small_input",
                device_type="xpu",
                dtypes=[torch.int64],
            ),
        ),
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 743-760
```python
        sample_inputs_func=sample_inputs_masked_reduction,
        sample_inputs_sparse_coo_func=sample_inputs_sparse_coo_masked_reduction,
        sample_inputs_sparse_csr_func=sample_inputs_sparse_csr_masked_reduction,
        gradcheck_wrapper=gradcheck_wrapper_masked_operation,
    ),
    ReductionOpInfo(
        "masked.amin",
        nan_policy="propagate",
        supports_out=False,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        dtypes=all_types_and(torch.float16, torch.bfloat16),
        supports_sparse=True,
        supports_sparse_csr=True,
        ref=reference_reduction_numpy(np.amin),
        skips=(
            DecorateInfo(
                unittest.expectedFailure,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 761-778
```python
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            # FIXME: amax reduces all dimensions when dim=[]
            DecorateInfo(unittest.expectedFailure, "TestReductions", "test_dim_empty"),
            DecorateInfo(
                unittest.expectedFailure, "TestReductions", "test_dim_empty_keepdim"
            ),
            # RuntimeError: Unknown builtin op: aten::iinfo
            DecorateInfo(
                unittest.expectedFailure, "TestJit", "test_variant_consistency_jit"
            ),
            # FIXME: "cuda_scatter_gather_base_kernel_func" not implemented for ... (used for sparse_coo inputs)
            # FIXME: "_segment_reduce_lengths_cpu/cuda" not implemented for ... (used for sparse_csr inputs)
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestMasked",
                "test_mask_layout",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 779-796
```python
                dtypes=(torch.bool, *integral_types(), *complex_types()),
            ),
            # Driver issue of XPU, see https://github.com/intel/torch-xpu-ops/issues/2295
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
                "test_reference_masked",
                device_type="xpu",
                dtypes=[torch.int64],
            ),
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
                "test_ref_small_input",
                device_type="xpu",
                dtypes=[torch.int64],
            ),
        ),
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 797-814
```python
        sample_inputs_func=sample_inputs_masked_reduction,
        sample_inputs_sparse_coo_func=sample_inputs_sparse_coo_masked_reduction,
        sample_inputs_sparse_csr_func=sample_inputs_sparse_csr_masked_reduction,
        gradcheck_wrapper=gradcheck_wrapper_masked_operation,
    ),
    ReductionOpInfo(
        "masked.argmax",
        supports_out=False,
        supports_multiple_dims=False,
        supports_autograd=False,
        dtypes=all_types_and(torch.float16, torch.bfloat16),
        ref=reference_reduction_numpy(np.argmax, supports_keepdims=False),
        skips=(
            DecorateInfo(
                unittest.expectedFailure,
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 815-832
```python
            # initial is not a keyword for argmax
            DecorateInfo(
                unittest.expectedFailure, "TestReductions", "test_reference_masked"
            ),
            # NotSupportedError: Compiled functions can't ... use keyword-only arguments with defaults
            DecorateInfo(
                unittest.expectedFailure, "TestJit", "test_variant_consistency_jit"
            ),
            # Driver issue of XPU, see https://github.com/intel/torch-xpu-ops/issues/2295
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
                "test_ref_small_input",
                device_type="xpu",
                dtypes=floating_types_and(
                    torch.int8, torch.int16, torch.int32, torch.int64, torch.float16
                ),
            ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 833-850
```python
        ),
        sample_inputs_func=sample_inputs_masked_reduction,
        gradcheck_wrapper=gradcheck_wrapper_masked_operation,
    ),
    ReductionOpInfo(
        "masked.argmin",
        supports_out=False,
        supports_multiple_dims=False,
        supports_autograd=False,
        dtypes=all_types_and(torch.float16, torch.bfloat16),
        ref=reference_reduction_numpy(np.argmin, supports_keepdims=False),
        skips=(
            DecorateInfo(
                unittest.expectedFailure,
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            # initial is not a keyword for argmin
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 851-868
```python
            DecorateInfo(
                unittest.expectedFailure, "TestReductions", "test_reference_masked"
            ),
            # NotSupportedError: Compiled functions can't ... use keyword-only arguments with defaults
            DecorateInfo(
                unittest.expectedFailure, "TestJit", "test_variant_consistency_jit"
            ),
            # Driver issue of XPU, see https://github.com/intel/torch-xpu-ops/issues/2295
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
                "test_ref_small_input",
                device_type="xpu",
                dtypes=floating_types_and(
                    torch.int8, torch.int16, torch.int32, torch.int64, torch.float16
                ),
            ),
        ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 869-886
```python
        sample_inputs_func=sample_inputs_masked_reduction,
        gradcheck_wrapper=gradcheck_wrapper_masked_operation,
    ),
    ReductionOpInfo(
        "masked.mean",
        ref=reference_reduction_numpy(np.mean)
        if np.lib.NumpyVersion(np.__version__) >= "1.20.2"
        else None,
        method_variant=None,
        nan_policy="propagate",
        supports_out=False,
        supports_sparse_csr=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        promotes_int_to_float=True,
        dtypes=floating_and_complex_types_and(torch.float16, torch.bfloat16),
        skips=(
            DecorateInfo(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 887-904
```python
                unittest.expectedFailure,
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            # FIXME: sum reduces all dimensions when dim=[]
            DecorateInfo(unittest.expectedFailure, "TestReductions", "test_dim_empty"),
            DecorateInfo(
                unittest.expectedFailure, "TestReductions", "test_dim_empty_keepdim"
            ),
            # RuntimeError: undefined value tensor
            DecorateInfo(
                unittest.expectedFailure, "TestJit", "test_variant_consistency_jit"
            ),
            # FIXME: "_segment_reduce_lengths_cpu/cuda" not implemented for ... (used for sparse_csr inputs)
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestMasked",
                "test_mask_layout",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 905-922
```python
                dtypes=(torch.bool, *integral_types(), *complex_types()),
            ),
            # Driver issue of XPU, see https://github.com/intel/torch-xpu-ops/issues/2295
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
                "test_ref_small_input",
                device_type="xpu",
                dtypes=[torch.complex128],
            ),
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
                "test_reference_masked",
                device_type="xpu",
                dtypes=[torch.complex128],
            ),
        ),
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 923-940
```python
        decorators=[
            DecorateInfo(
                toleranceOverride(
                    {
                        torch.bfloat16: tol(atol=1e-03, rtol=0.05),
                        torch.float16: tol(atol=1e-03, rtol=1e-03),
                    }
                ),
                "TestReductions",
                "test_reference_masked",
            ),
            DecorateInfo(
                toleranceOverride({torch.float16: tol(atol=1e-03, rtol=1e-03)}),
                "TestReductions",
                "test_ref_small_input",
            ),
            DecorateInfo(
                toleranceOverride({torch.float16: tol(atol=1e-03, rtol=2e-03)}),
```
- EN: This block implements local helper logic for masked. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 masked 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 941-958
```python
                "TestSparseCompressed",
                "test_consistency",
                device_type="cuda",
            ),
        ],
        sample_inputs_func=sample_inputs_masked_reduction,
        sample_inputs_sparse_csr_func=sample_inputs_sparse_csr_masked_reduction,
        gradcheck_wrapper=gradcheck_wrapper_masked_operation,
    ),
    OpInfo(
        "masked.median",
        dtypes=floating_types_and(torch.bfloat16, torch.float16),
        method_variant=None,
        supports_out=False,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        skips=(
            DecorateInfo(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 959-976
```python
                unittest.expectedFailure,
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            # NotSupportedError: Compiled functions can't ... use keyword-only arguments with defaults
            DecorateInfo(
                unittest.skip("Skipped!"), "TestJit", "test_variant_consistency_jit"
            ),
        ),
        sample_inputs_func=partial(
            sample_inputs_masked_softmax, use_zero_dimensions=False
        ),
        gradcheck_wrapper=gradcheck_wrapper_masked_operation,
    ),
    ReductionOpInfo(
        "masked.norm",
        identity=0,
        method_variant=None,
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 977-994
```python
        nan_policy="propagate",
        supports_out=False,
        promotes_int_to_float=True,
        dtypes=floating_types_and(torch.float16, torch.bfloat16),
        skips=(
            DecorateInfo(
                unittest.expectedFailure,
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            # FIXME: sum reduces all dimensions when dim=[]
            DecorateInfo(unittest.expectedFailure, "TestReductions", "test_dim_empty"),
            DecorateInfo(
                unittest.expectedFailure, "TestReductions", "test_dim_empty_keepdim"
            ),
            # torch.jit.frontend.NotSupportedError: Compiled functions
            # can't take variable number of arguments or use
            # keyword-only arguments with defaults
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 995-1007
```python
            DecorateInfo(
                unittest.expectedFailure, "TestJit", "test_variant_consistency_jit"
            ),
        ),
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        sample_inputs_func=sample_inputs_masked_norm,
        gradcheck_wrapper=gradcheck_wrapper_masked_operation,
    ),
    ReductionOpInfo(
        "masked.var",
        ref=reference_masked_std_var(np.var)
        if np.lib.NumpyVersion(np.__version__) >= "1.20.2"
```
- EN: This block checks invariants or expected outcomes; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理条件控制流。关键符号：无明显局部符号。

### Lines 1008-1025
```python
        else None,
        method_variant=None,
        nan_policy="propagate",
        supports_out=False,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        # See https://github.com/pytorch/pytorch/pull/78358
        check_batched_forward_grad=False,
        promotes_int_to_float=True,
        dtypes=all_types_and_complex_and(torch.float16, torch.bfloat16),
        skips=(
            # Issue with conj and torch dispatch, see https://github.com/pytorch/pytorch/issues/82479
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestSchemaCheckModeOpInfo",
                "test_schema_correctness",
                dtypes=(torch.complex64, torch.complex128),
            ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 1026-1043
```python
            DecorateInfo(
                unittest.expectedFailure,
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            # FIXME: sum reduces all dimensions when dim=[]
            DecorateInfo(unittest.expectedFailure, "TestReductions", "test_dim_empty"),
            DecorateInfo(
                unittest.expectedFailure, "TestReductions", "test_dim_empty_keepdim"
            ),
            # RuntimeError: undefined value tensor
            DecorateInfo(
                unittest.expectedFailure, "TestJit", "test_variant_consistency_jit"
            ),
            # Driver issue of XPU, see https://github.com/intel/torch-xpu-ops/issues/2295
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 1044-1061
```python
                "test_ref_small_input",
                device_type="xpu",
                dtypes=[torch.complex128],
            ),
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
                "test_reference_masked",
                device_type="xpu",
                dtypes=[torch.complex128],
            ),
        ),
        decorators=[
            DecorateInfo(
                toleranceOverride(
                    {
                        torch.float16: tol(atol=1e-02, rtol=1e-02),
                        torch.bfloat16: tol(atol=1e-03, rtol=1e-03),
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 1062-1079
```python
                    }
                ),
                "TestReductions",
                "test_reference_masked",
            ),
            DecorateInfo(
                toleranceOverride({torch.float16: tol(atol=1e-02, rtol=1e-02)}),
                "TestReductions",
                "test_ref_small_input",
            ),
            DecorateInfo(
                toleranceOverride({torch.float16: tol(atol=1e-02, rtol=1e-02)}),
                "TestMasked",
                "test_reference_masked",
            ),
            DecorateInfo(
                toleranceOverride(
                    {
```
- EN: This block implements local helper logic for masked. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 masked 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 1080-1097
```python
                        torch.float16: tol(atol=1e-02, rtol=1e-02),
                        torch.bfloat16: tol(atol=1e-03, rtol=1e-03),
                    }
                ),
                "TestMasked",
                "test_reference_masked",
            ),
            DecorateInfo(
                toleranceOverride(
                    {
                        torch.float16: tol(atol=4e-5, rtol=2e-2),
                    }
                ),
                "TestInductorOpInfo",
                "test_comprehensive",
                device_type="cuda",
            ),
        ],
```
- EN: This block handles tensor metadata or sample values; bridges to backend-specific execution artifacts. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；桥接到特定后端的执行产物。关键符号：无明显局部符号。

### Lines 1098-1115
```python
        sample_inputs_func=sample_inputs_masked_std_var,
        gradcheck_wrapper=gradcheck_wrapper_masked_operation,
        check_batched_grad=True,
    ),
    ReductionOpInfo(
        "masked.std",
        ref=reference_masked_std_var(np.std)
        if np.lib.NumpyVersion(np.__version__) >= "1.20.2"
        else None,
        method_variant=None,
        nan_policy="propagate",
        # Runs very slowly on slow gradcheck - alternatively reduce input sizes
        gradcheck_fast_mode=True,
        supports_out=False,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        # See https://github.com/pytorch/pytorch/pull/78358
        check_batched_forward_grad=False,
```
- EN: This block checks invariants or expected outcomes; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理条件控制流。关键符号：无明显局部符号。

### Lines 1116-1133
```python
        promotes_int_to_float=True,
        dtypes=all_types_and_complex_and(torch.half, torch.bfloat16),
        skips=(
            # Issue with conj and torch dispatch, see https://github.com/pytorch/pytorch/issues/82479
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestSchemaCheckModeOpInfo",
                "test_schema_correctness",
                dtypes=(torch.complex64, torch.complex128),
            ),
            DecorateInfo(
                unittest.expectedFailure,
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            # FIXME: sum reduces all dimensions when dim=[]
            DecorateInfo(unittest.expectedFailure, "TestReductions", "test_dim_empty"),
            DecorateInfo(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 1134-1151
```python
                unittest.expectedFailure, "TestReductions", "test_dim_empty_keepdim"
            ),
            # RuntimeError: undefined value tensor
            DecorateInfo(
                unittest.expectedFailure, "TestJit", "test_variant_consistency_jit"
            ),
            # Driver issue of XPU, see https://github.com/intel/torch-xpu-ops/issues/2295
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
                "test_ref_small_input",
                device_type="xpu",
                dtypes=[torch.complex128],
            ),
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
                "test_reference_masked",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 1152-1169
```python
                device_type="xpu",
                dtypes=[torch.complex128],
            ),
        ),
        decorators=[
            DecorateInfo(
                toleranceOverride(
                    {
                        torch.bfloat16: tol(atol=1e-02, rtol=1e-02),
                        torch.float16: tol(atol=1e-02, rtol=1e-02),
                    }
                ),
                "TestReductions",
                "test_reference_masked",
            ),
            DecorateInfo(
                toleranceOverride({torch.float16: tol(atol=1e-02, rtol=1e-02)}),
                "TestReductions",
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 1170-1187
```python
                "test_ref_small_input",
            ),
            DecorateInfo(
                toleranceOverride(
                    {
                        torch.float16: tol(atol=1e-02, rtol=1e-02),
                        torch.bfloat16: tol(atol=5e-03, rtol=5e-04),
                    }
                ),
                "TestMasked",
                "test_reference_masked",
            ),
        ],
        sample_inputs_func=sample_inputs_masked_std_var,
        gradcheck_wrapper=gradcheck_wrapper_masked_operation,
        check_batched_grad=True,
    ),
    OpInfo(
```
- EN: This block checks invariants or expected outcomes. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果。关键符号：无明显局部符号。

### Lines 1188-1205
```python
        "masked.softmax",
        method_variant=None,
        dtypes=floating_types_and(torch.half, torch.bfloat16),
        sample_inputs_func=sample_inputs_masked_softmax,
        skips=(
            DecorateInfo(
                unittest.expectedFailure,
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            DecorateInfo(
                unittest.expectedFailure, "TestJit", "test_variant_consistency_jit"
            ),
        ),
        gradcheck_wrapper=gradcheck_wrapper_masked_operation,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        supports_out=False,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 1206-1223
```python
    ),
    OpInfo(
        "masked.log_softmax",
        method_variant=None,
        dtypes=floating_types_and(torch.half, torch.bfloat16),
        dtypesIfMPS=floating_types_and(
            torch.half,
            torch.bfloat16,
            torch.uint8,
            torch.int32,
            torch.int16,
            torch.int8,
        ),
        sample_inputs_func=sample_inputs_masked_softmax,
        skips=(
            DecorateInfo(
                unittest.expectedFailure,
                "TestNormalizeOperators",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 1224-1241
```python
                "test_normalize_operator_exhaustive",
            ),
            DecorateInfo(
                unittest.expectedFailure, "TestJit", "test_variant_consistency_jit"
            ),
            # NotImplementedError: "log_softmax_lastdim_kernel_impl" not implemented for *
            DecorateInfo(
                unittest.expectedFailure,
                "TestConsistency",
                device_type="mps",
                dtypes=(
                    torch.uint8,
                    torch.int32,
                    torch.int16,
                    torch.int8,
                ),
            ),
        ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1242-1259
```python
        decorators=[
            DecorateInfo(
                toleranceOverride({torch.bfloat16: tol(atol=1e-02, rtol=1e-02)}),
                "TestMasked",
                "test_reference_masked",
            ),
        ],
        gradcheck_wrapper=gradcheck_wrapper_masked_operation,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        supports_out=False,
    ),
    OpInfo(
        "masked.softmin",
        method_variant=None,
        dtypes=floating_types_and(torch.half, torch.bfloat16),
        sample_inputs_func=sample_inputs_masked_softmax,
        skips=(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 1260-1277
```python
            DecorateInfo(
                unittest.expectedFailure,
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            DecorateInfo(
                unittest.expectedFailure, "TestJit", "test_variant_consistency_jit"
            ),
            # FIXME:
            # Mismatched elements: 2 / 2 (100.0%)
            # Greatest absolute difference: nan at index (0,) (up to 0.0001 allowed)
            # Greatest relative difference: nan at index (0,) (up to 0.0001 allowed
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestOperators",
                "test_vmapvjpvjp",
                device_type="cpu",
            ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 1278-1295
```python
        ),
        gradcheck_wrapper=gradcheck_wrapper_masked_operation,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        supports_out=False,
    ),
    OpInfo(
        "masked.normalize",
        method_variant=None,
        dtypes=floating_and_complex_types_and(torch.half, torch.bfloat16),
        sample_inputs_func=sample_inputs_masked_normalize,
        decorators=[
            DecorateInfo(
                toleranceOverride({torch.float16: tol(atol=2e-5, rtol=6e-3)}),
                "TestInductorOpInfo",
                "test_comprehensive",
                device_type="cuda",
            ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; bridges to backend-specific execution artifacts. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；桥接到特定后端的执行产物。关键符号：无明显局部符号。

### Lines 1296-1313
```python
        ],
        skips=(
            DecorateInfo(
                unittest.expectedFailure,
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            DecorateInfo(
                unittest.expectedFailure, "TestJit", "test_variant_consistency_jit"
            ),
        ),
        gradcheck_wrapper=gradcheck_wrapper_masked_operation,
        # Runs very slowly on slow gradcheck - alternatively reduce input sizes
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        supports_out=False,
    ),
```
- EN: This block checks invariants or expected outcomes. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果。关键符号：无明显局部符号。

### Lines 1314-1331
```python
    OpInfo(
        "masked.logaddexp",
        dtypes=floating_types_and(torch.float16, torch.bfloat16),
        supports_out=False,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        check_batched_forward_grad=False,
        skips=(
            DecorateInfo(
                unittest.expectedFailure,
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            # NotSupportedError: Compiled functions can't ... use keyword-only arguments with defaults
            DecorateInfo(
                unittest.skip("Skipped!"), "TestJit", "test_variant_consistency_jit"
            ),
            DecorateInfo(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 1332-1349
```python
                unittest.skip("Skipped!"), "TestFwdGradients", "test_fn_gradgrad"
            ),
            DecorateInfo(
                unittest.skip("Skipped!"), "TestBwdGradients", "test_fn_gradgrad"
            ),
        ),
        sample_inputs_func=sample_inputs_masked_logaddexp,
        gradcheck_wrapper=gradcheck_wrapper_masked_pointwise_operation,
    ),
    ReductionOpInfo(
        "masked.logsumexp",
        dtypes=all_types_and_complex_and(torch.half, torch.bfloat16),
        method_variant=None,
        nan_policy="propagate",
        supports_out=False,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        skips=(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 1350-1367
```python
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
            # FIXME: reduces all dimensions when dim=[]
            DecorateInfo(unittest.skip("Skipped!"), "TestReductions", "test_dim_empty"),
            DecorateInfo(
                unittest.skip("Skipped!"), "TestReductions", "test_dim_empty_keepdim"
            ),
            # Identity can't be -torch.inf without overflow
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestReductions",
                "test_empty_tensor_empty_slice",
            ),
            # NotSupportedError: Compiled functions can't ... use keyword-only arguments with defaults
            DecorateInfo(
```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 1368-1385
```python
                unittest.skip("Skipped!"), "TestJit", "test_variant_consistency_jit"
            ),
            # all the values are the same except for -inf vs nan
            DecorateInfo(unittest.skip("Skipped!"), "TestDecomp", "test_comprehensive"),
            # FIXME:
            # Mismatched elements: 2 / 12 (16.7%)
            # Greatest absolute difference: 9223372034707292160 at index (0, 0, 0, 0)
            # Greatest relative difference: 0.0 at index (0, 0, 0, 1)
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestInductorOpInfo",
                "test_comprehensive",
                device_type="cpu",
            ),
        ),
        sample_inputs_func=sample_inputs_masked_reduction,
        gradcheck_wrapper=gradcheck_wrapper_masked_operation,
    ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; bridges to backend-specific execution artifacts; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；桥接到特定后端的执行产物；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1386-1386
```python
]
```
- EN: This block implements local helper logic for masked. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 masked 相关的局部辅助逻辑。关键符号：无明显局部符号。


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
- Internal imports / 内部导入: `torch`, `torch.testing`, `torch.testing._internal.common_device_type`, `torch.testing._internal.common_dtype`, `torch.testing._internal.opinfo.core`, `torch.testing._internal.opinfo.utils`, `torch.testing._internal.common_methods_invocations`
- External imports / 外部导入: `unittest`, `collections.abc`, `functools`, `numpy`
- Representative symbols / 代表性符号: `sample_inputs_softmax_variant`, `_generate_masked_op_mask`, `sample_inputs_masked_reduction`, `sample_inputs_sparse_coo_masked_reduction`, `sample_inputs_sparse_csr_masked_reduction`, `sample_inputs_masked_norm`, `reference_masked_std_var`, `sample_inputs_masked_std_var`, `sample_inputs_masked_softmax`, `sample_inputs_masked_cumops`, `...`
