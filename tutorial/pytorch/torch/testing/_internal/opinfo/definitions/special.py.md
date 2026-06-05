# special.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/opinfo/definitions/special.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for special, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 special 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9
```python
# mypy: ignore-errors

import unittest
from functools import partial
from itertools import product

import numpy as np

import torch
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`; external imports: `unittest`, `functools`, `itertools`, `numpy`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`；外部导入：`unittest`, `functools`, `itertools`, `numpy`。

### Lines 10-18
```python
from torch.testing import make_tensor
from torch.testing._internal.common_device_type import (
    precisionOverride,
    tol,
    toleranceOverride,
)
from torch.testing._internal.common_dtype import all_types_and, floating_types
from torch.testing._internal.common_utils import TEST_SCIPY, torch_to_numpy_dtype_dict
from torch.testing._internal.opinfo.core import (
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 19-28
```python
    BinaryUfuncInfo,
    DecorateInfo,
    L,
    NumericsFilter,
    OpInfo,
    S,
    SampleInput,
    UnaryUfuncInfo,
)
from torch.testing._internal.opinfo.refs import (
```
- EN: This block implements local helper logic for special. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 special 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 29-37
```python
    ElementwiseBinaryPythonRefInfo,
    ElementwiseUnaryPythonRefInfo,
)
from torch.testing._internal.opinfo.utils import (
    np_unary_ufunc_integer_promotion_wrapper,
)


if TEST_SCIPY:
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 38-55
```python
    import scipy.special


# TODO: Consolidate `i0e` with sample_inputs_unary when `make_tensor`,
#       supports `exclude` argument.
#       For more context: https://github.com/pytorch/pytorch/pull/56352#discussion_r633277617
def sample_inputs_i0_i1(op_info, device, dtype, requires_grad, **kwargs):
    exclude_zero = requires_grad and op_info.op is torch.special.i0e
    make_arg = partial(
        make_tensor,
        dtype=dtype,
        device=device,
        requires_grad=requires_grad,
        exclude_zero=exclude_zero,
    )
    yield SampleInput(make_arg((S,)))
    yield SampleInput(make_arg(()))

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_i0_i1`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_i0_i1`。

### Lines 56-64
```python
    if requires_grad and not exclude_zero:
        # Special Case for gradient
        # Sample with `0` in the input
        t = make_arg((S,))
        t[0] = 0

        yield SampleInput(t)


```
- EN: This block handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 65-76
```python
def sample_inputs_polygamma(op_info, device, dtype, requires_grad, **kwargs):
    make_arg = partial(
        make_tensor,
        device=device,
        # TODO: eliminate low after gh-106692 is fixed:
        low=(1 if dtype in {torch.int32, torch.int64} else None),
        dtype=dtype,
        requires_grad=requires_grad,
    )
    tensor_shapes = ((S, S), ())
    ns = (1, 2, 3, 4, 5)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_polygamma`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_polygamma`。

### Lines 77-90
```python
    for shape, n in product(tensor_shapes, ns):
        yield SampleInput(make_arg(shape), args=(n,))


def reference_polygamma(x, n):
    # WEIRD `scipy.special.polygamma` behavior
    # >>> scipy.special.polygamma(0, np.array(501, dtype=np.float32)).dtype
    # dtype('float64')
    # >>> scipy.special.polygamma(0, np.array([501], dtype=np.float32)).dtype
    # dtype('float32')
    #
    # Thus we cast output to the default torch dtype or preserve double
    result_dtype = torch_to_numpy_dtype_dict[torch.get_default_dtype()]
    if x.dtype == np.double:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `reference_polygamma`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`reference_polygamma`。

### Lines 91-100
```python
        result_dtype = np.double
    return scipy.special.polygamma(n, x).astype(result_dtype)


def sample_inputs_entr(op_info, device, dtype, requires_grad, **kwargs):
    low, _ = op_info.domain

    if requires_grad:
        low = 0 + op_info._domain_eps

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_entr`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_entr`。

### Lines 101-109
```python
    make_arg = partial(
        make_tensor, dtype=dtype, device=device, low=low, requires_grad=requires_grad
    )
    yield SampleInput(make_arg((L,)))
    yield SampleInput(make_arg(()))


def sample_inputs_erfcx(op_info, device, dtype, requires_grad, **kwargs):
    for shape in ((L,), (1, 0, 3), ()):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_erfcx`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_erfcx`。

### Lines 110-119
```python
        yield SampleInput(
            make_tensor(
                shape,
                device=device,
                dtype=dtype,
                low=-5,
                requires_grad=requires_grad,
            ),
        )

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 120-137
```python

_unsigned_int_types = (torch.uint16, torch.uint32, torch.uint64)


op_db: list[OpInfo] = [
    UnaryUfuncInfo(
        "special.i0e",
        aten_name="special_i0e",
        ref=scipy.special.i0e if TEST_SCIPY else None,
        decorators=(precisionOverride({torch.bfloat16: 3e-1, torch.float16: 3e-1}),),
        dtypes=all_types_and(
            torch.bool, torch.half, torch.bfloat16, *_unsigned_int_types
        ),
        dtypesIfMPS=all_types_and(torch.bool, torch.half, torch.bfloat16),
        sample_inputs_func=sample_inputs_i0_i1,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
    ),
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 138-155
```python
    UnaryUfuncInfo(
        "special.i1",
        aten_name="special_i1",
        ref=np_unary_ufunc_integer_promotion_wrapper(scipy.special.i1)
        if TEST_SCIPY
        else None,
        dtypes=all_types_and(
            torch.bool, torch.half, torch.bfloat16, *_unsigned_int_types
        ),
        dtypesIfMPS=all_types_and(torch.bool, torch.half, torch.bfloat16),
        backward_dtypes=floating_types(),
        sample_inputs_func=sample_inputs_i0_i1,
        decorators=(
            DecorateInfo(
                toleranceOverride(
                    {
                        torch.float32: tol(atol=1e-4, rtol=0),
                        torch.bool: tol(atol=1e-4, rtol=0),
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 156-173
```python
                    }
                )
            ),
        ),
        skips=(
            DecorateInfo(
                unittest.skip("Incorrect result!"),
                "TestUnaryUfuncs",
                "test_reference_numerics_large",
                dtypes=(torch.int8,),
            ),
        ),
        supports_fwgrad_bwgrad=True,
        supports_forward_ad=True,
    ),
    UnaryUfuncInfo(
        "special.i1e",
        aten_name="special_i1e",
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 174-191
```python
        ref=scipy.special.i1e if TEST_SCIPY else None,
        dtypes=all_types_and(
            torch.bool, torch.half, torch.bfloat16, *_unsigned_int_types
        ),
        dtypesIfMPS=all_types_and(torch.bool, torch.half, torch.bfloat16),
        backward_dtypes=floating_types(),
        sample_inputs_func=sample_inputs_i0_i1,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
    ),
    UnaryUfuncInfo(
        "special.ndtr",
        aten_name="special_ndtr",
        decorators=(precisionOverride({torch.bfloat16: 5e-3, torch.float16: 5e-4}),),
        ref=scipy.special.ndtr if TEST_SCIPY else None,
        dtypes=all_types_and(
            torch.bool, torch.half, torch.bfloat16, *_unsigned_int_types
        ),
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 192-209
```python
        dtypesIfMPS=all_types_and(torch.bool, torch.half, torch.bfloat16),
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        skips=(
            # Dispatch stub: unsupported device typemeta
            DecorateInfo(
                unittest.expectedFailure,
                "TestFwdGradients",
                "test_fn_fwgrad_bwgrad",
                device_type="meta",
            ),
        ),
    ),
    # A separate OpInfo entry for special.polygamma is needed to reorder the arguments
    # for the alias. See the discussion here: https://github.com/pytorch/pytorch/pull/59691#discussion_r650261939
    UnaryUfuncInfo(
        "special.polygamma",
        op=lambda x, n, **kwargs: torch.special.polygamma(n, x, **kwargs),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 210-227
```python
        variant_test_name="special_polygamma_n_0",
        ref=reference_polygamma if TEST_SCIPY else None,
        dtypes=all_types_and(torch.bool, torch.half, torch.bfloat16),
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        sample_inputs_func=sample_inputs_polygamma,
        skips=(
            # lambda impl
            DecorateInfo(
                unittest.expectedFailure, "TestJit", "test_variant_consistency_jit"
            ),
            DecorateInfo(
                unittest.expectedFailure,
                "TestNormalizeOperators",
                "test_normalize_operator_exhaustive",
            ),
        ),
        sample_kwargs=lambda device, dtype, input: ({"n": 0}, {"n": 0}),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 228-245
```python
        # polygamma functions have multiple singularities at x having non-positive integer value
        reference_numerics_filter=NumericsFilter(
            condition=lambda x: (x < 0.1) & ((x - x.round()).abs() < 1e-4), safe_val=1
        ),
    ),
    BinaryUfuncInfo(
        "special.xlog1py",
        aten_name="special_xlog1py",
        dtypes=all_types_and(
            torch.bool, torch.half, torch.bfloat16, *_unsigned_int_types
        ),
        dtypesIfMPS=all_types_and(torch.bool, torch.half, torch.bfloat16),
        promotes_int_to_float=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        supports_one_python_scalar=True,
        # We don't test -1 as the gradient will be NaN and it'll break
        rhs_make_tensor_kwargs=dict(low=-0.99),
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 246-263
```python
    ),
    BinaryUfuncInfo(
        "special.zeta",
        aten_name="special_zeta",
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        promotes_int_to_float=True,
        supports_autograd=False,
        supports_one_python_scalar=True,
        skips=(
            # Reference reference_inputs nans and infs on cuda and nan, inf, 0., -inf for cpu
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_compare_cpu"),
        ),
    ),
    # TODO: FIXME
    # OpInfo entry to verify the gradient formula of `other`/`q`
    # BinaryUfuncInfo('special.zeta',
    #                 op=lambda q, x, **kwargs: torch.special.zeta(x, q, **kwargs),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 264-281
```python
    #                 aten_name='special_zeta',
    #                 variant_test_name='grad',
    #                 dtypes=all_types_and(torch.bool),
    #                 promotes_int_to_float=True,
    #                 supports_autograd=True,
    #                 supports_rhs_python_scalar=False,
    #                 decorators=[
    #                     # Derivative wrt first tensor not implemented
    #                     DecorateInfo(unittest.expectedFailure, "TestCommon",
    #                                  "test_floating_inputs_are_differentiable")
    #                 ],
    #                 skips=(
    #                     # Lambda doesn't work in JIT test
    #                     # AssertionError: JIT Test does not execute any logic
    #                     DecorateInfo(unittest.skip("Skipped!"), "TestJit", "test_variant_consistency_jit"),
    #                 )),
    UnaryUfuncInfo(
        "special.entr",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 282-299
```python
        ref=scipy.special.entr if TEST_SCIPY else None,
        aten_name="special_entr",
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        decorators=(precisionOverride({torch.float16: 1e-1, torch.bfloat16: 1e-1}),),
        dtypes=all_types_and(
            torch.bool, torch.half, torch.bfloat16, *_unsigned_int_types
        ),
        dtypesIfMPS=all_types_and(torch.bool, torch.half, torch.bfloat16),
        skips=(
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestUnaryUfuncs",
                "test_reference_numerics_large",
                dtypes=[torch.bfloat16, torch.float16],
            ),
        ),
        supports_inplace_autograd=False,
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 300-317
```python
        sample_inputs_func=sample_inputs_entr,
    ),
    UnaryUfuncInfo(
        "special.ndtri",
        ref=scipy.special.ndtri if TEST_SCIPY else None,
        domain=(0, 1),
        aten_name="special_ndtri",
        dtypes=all_types_and(torch.bool),
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        skips=(
            # The operator 'aten::special_ndtri.out' is not currently implemented for the MPS device
            DecorateInfo(unittest.expectedFailure, "TestCommon", device_type="mps"),
        ),
    ),
    UnaryUfuncInfo(
        "special.log_ndtr",
        aten_name="special_log_ndtr",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 318-335
```python
        ref=scipy.special.log_ndtr if TEST_SCIPY else None,
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool),
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        skips=(
            # The operator 'aten::special_log_ndtr.out' is not currently implemented for the MPS device
            DecorateInfo(unittest.expectedFailure, "TestCommon", device_type="mps"),
        ),
    ),
    UnaryUfuncInfo(
        "special.erfcx",
        ref=scipy.special.erfcx if TEST_SCIPY else None,
        aten_name="special_erfcx",
        decorators=(
            toleranceOverride(
                {
                    torch.float32: tol(atol=0, rtol=4e-6),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 336-353
```python
                }
            ),
        ),
        skips=(
            # Exception: Tensor-likes are not close!
            # Greatest absolute difference: inf at index (10,) (up to 1e-05 allowed)
            # Greatest relative difference: nan at index (10,) (up to 0.001 allowed)
            DecorateInfo(
                unittest.expectedFailure,
                "TestConsistency",
                "test_output_grad_match",
                device_type="mps",
                dtypes=(torch.float16,),
            ),
        ),
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        supports_forward_ad=True,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 354-371
```python
        supports_fwgrad_bwgrad=True,
        sample_inputs_func=sample_inputs_erfcx,
    ),
    UnaryUfuncInfo(
        "special.airy_ai",
        decorators=(
            precisionOverride(
                {
                    torch.float32: 1e-03,
                    torch.float64: 1e-05,
                },
            ),
        ),
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool),
        ref=lambda x: scipy.special.airy(x)[0] if TEST_SCIPY else None,
        skips=(
            DecorateInfo(
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 372-389
```python
                unittest.skip("Skipped!"),
                "TestUnaryUfuncs",
                "test_reference_numerics_large",
            ),
            # NotImplementedError: The operator 'aten::special_airy_ai.out' is not currently implemented for the MPS device
            DecorateInfo(unittest.expectedFailure, "TestCommon", device_type="mps"),
        ),
        supports_autograd=False,
    ),
    UnaryUfuncInfo(
        "special.bessel_j0",
        decorators=(
            precisionOverride(
                {
                    torch.float32: 1e-04,
                    torch.float64: 1e-05,
                },
            ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 390-407
```python
            # Comparing fp32 CPU grads with fp16 MPS ones leads to high errors
            DecorateInfo(
                toleranceOverride(
                    {
                        torch.float16: tol(atol=1e-2, rtol=1e-4),
                        torch.bfloat16: tol(atol=1e-2, rtol=1e-4),
                    }
                ),
                "TestConsistency",
                "test_output_match",
                device_type="mps",
            ),
            DecorateInfo(
                toleranceOverride(
                    {
                        torch.float16: tol(atol=1e-2, rtol=1e-4),
                        torch.bfloat16: tol(atol=1e-2, rtol=1e-4),
                    }
```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 408-425
```python
                ),
                "TestConsistency",
                "test_output_grad_match",
                device_type="mps",
            ),
        ),
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        ref=scipy.special.j0 if TEST_SCIPY else None,
        supports_autograd=False,
    ),
    UnaryUfuncInfo(
        "special.bessel_j1",
        decorators=(
            precisionOverride(
                {
                    torch.float32: 1e-04,
                    torch.float64: 1e-05,
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 426-443
```python
                },
            ),
            # Comparing fp32 CPU grads with fp16 MPS ones leads to high errors
            DecorateInfo(
                toleranceOverride(
                    {
                        torch.float16: tol(atol=1e-2, rtol=1e-4),
                        torch.bfloat16: tol(atol=1e-2, rtol=1e-4),
                    }
                ),
                "TestConsistency",
                "test_output_match",
                device_type="mps",
            ),
            DecorateInfo(
                toleranceOverride({torch.float16: tol(atol=0.0004, rtol=0.009)}),
                "TestConsistency",
                "test_output_grad_match",
```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 444-461
```python
                device_type="mps",
            ),
        ),
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        ref=scipy.special.j1 if TEST_SCIPY else None,
        supports_autograd=False,
    ),
    UnaryUfuncInfo(
        "special.bessel_y0",
        decorators=(
            precisionOverride(
                {
                    torch.float32: 1e-04,
                    torch.float64: 1e-05,
                },
            ),
            # Comparing fp32 CPU grads with fp16 MPS ones leads to high errors
```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 462-479
```python
            DecorateInfo(
                toleranceOverride(
                    {
                        torch.float16: tol(atol=1e-2, rtol=1e-4),
                        torch.bfloat16: tol(atol=1e-2, rtol=1e-4),
                    }
                ),
                "TestConsistency",
                "test_output_match",
                device_type="mps",
            ),
        ),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        ref=scipy.special.y0 if TEST_SCIPY else None,
        supports_autograd=False,
    ),
    UnaryUfuncInfo(
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 480-497
```python
        "special.bessel_y1",
        decorators=(
            precisionOverride(
                {
                    torch.float32: 1e-04,
                    torch.float64: 1e-05,
                },
            ),
            # Comparing fp32 CPU grads with fp16 MPS ones leads to high errors
            DecorateInfo(
                toleranceOverride(
                    {
                        torch.float16: tol(atol=1e-2, rtol=1e-4),
                        torch.bfloat16: tol(atol=1e-2, rtol=1e-4),
                    }
                ),
                "TestConsistency",
                "test_output_match",
```
- EN: This block reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 498-515
```python
                device_type="mps",
            ),
        ),
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        ref=scipy.special.y1 if TEST_SCIPY else None,
        supports_autograd=False,
    ),
    BinaryUfuncInfo(
        "special.chebyshev_polynomial_t",
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        promotes_int_to_float=True,
        skips=(
            DecorateInfo(unittest.skip("Skipped!"), "TestCudaFuserOpInfo"),
            DecorateInfo(unittest.skip("Skipped!"), "TestNNCOpInfo"),
            # Greatest absolute difference: nan
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_compare_cpu"),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 516-533
```python
        ),
        supports_one_python_scalar=True,
        supports_autograd=False,
    ),
    BinaryUfuncInfo(
        "special.chebyshev_polynomial_u",
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        promotes_int_to_float=True,
        skips=(
            DecorateInfo(unittest.skip("Skipped!"), "TestCudaFuserOpInfo"),
            DecorateInfo(unittest.skip("Skipped!"), "TestNNCOpInfo"),
            # Greatest absolute difference: nan
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_compare_cpu"),
        ),
        supports_one_python_scalar=True,
        supports_autograd=False,
    ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 534-551
```python
    BinaryUfuncInfo(
        "special.chebyshev_polynomial_v",
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        promotes_int_to_float=True,
        skips=(
            DecorateInfo(unittest.skip("Skipped!"), "TestCudaFuserOpInfo"),
            DecorateInfo(unittest.skip("Skipped!"), "TestNNCOpInfo"),
            # Greatest absolute difference: nan
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_compare_cpu"),
        ),
        supports_one_python_scalar=True,
        supports_autograd=False,
    ),
    BinaryUfuncInfo(
        "special.chebyshev_polynomial_w",
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 552-569
```python
        promotes_int_to_float=True,
        skips=(
            DecorateInfo(unittest.skip("Skipped!"), "TestCudaFuserOpInfo"),
            DecorateInfo(unittest.skip("Skipped!"), "TestNNCOpInfo"),
            # Greatest absolute difference: nan
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_compare_cpu"),
        ),
        supports_one_python_scalar=True,
        supports_autograd=False,
    ),
    BinaryUfuncInfo(
        "special.hermite_polynomial_h",
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        promotes_int_to_float=True,
        skips=(
            DecorateInfo(unittest.skip("Skipped!"), "TestCudaFuserOpInfo"),
            DecorateInfo(unittest.skip("Skipped!"), "TestNNCOpInfo"),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 570-587
```python
            # Greatest absolute difference: inf
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_compare_cpu"),
            # Too slow
            DecorateInfo(
                unittest.skip, "TestCommon", "test_compare_cpu", device_type="xpu"
            ),
        ),
        supports_one_python_scalar=True,
        supports_autograd=False,
    ),
    BinaryUfuncInfo(
        "special.hermite_polynomial_he",
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        promotes_int_to_float=True,
        skips=(
            DecorateInfo(unittest.skip("Skipped!"), "TestCudaFuserOpInfo"),
            DecorateInfo(unittest.skip("Skipped!"), "TestNNCOpInfo"),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 588-605
```python
            # Greatest absolute difference: inf
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_compare_cpu"),
        ),
        supports_one_python_scalar=True,
        supports_autograd=False,
    ),
    BinaryUfuncInfo(
        "special.laguerre_polynomial_l",
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool),
        promotes_int_to_float=True,
        skips=(
            DecorateInfo(unittest.skip("Skipped!"), "TestCudaFuserOpInfo"),
            DecorateInfo(unittest.skip("Skipped!"), "TestNNCOpInfo"),
            # Greatest absolute difference: nan
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_compare_cpu"),
            # Too slow
            DecorateInfo(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 606-623
```python
                unittest.skip, "TestCommon", "test_compare_cpu", device_type="xpu"
            ),
            # NotImplementedError: The operator 'aten::special_laguerre_polynomial_l.out'
            # is not currently implemented for the MPS device
            DecorateInfo(
                unittest.expectedFailure,
                "TestCommon",
                "test_variant_consistency_eager",
                device_type="mps",
            ),
            DecorateInfo(
                unittest.expectedFailure,
                "TestCommon",
                "test_promotes_int_to_float",
                device_type="mps",
            ),
            DecorateInfo(
                unittest.expectedFailure,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 624-641
```python
                "TestCommon",
                "test_out_warning",
                device_type="mps",
            ),
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
            DecorateInfo(
                unittest.expectedFailure,
                "TestCommon",
                "test_noncontiguous_samples",
                device_type="mps",
            ),
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_dtypes", device_type="mps"
            ),
        ),
        supports_one_python_scalar=True,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 642-659
```python
        supports_autograd=False,
    ),
    BinaryUfuncInfo(
        "special.legendre_polynomial_p",
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool),
        promotes_int_to_float=True,
        skips=(
            DecorateInfo(unittest.skip("Skipped!"), "TestCudaFuserOpInfo"),
            DecorateInfo(unittest.skip("Skipped!"), "TestNNCOpInfo"),
            # Greatest absolute difference: nan
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_compare_cpu"),
            # NotImplementedError: The operator 'aten::special_legendre_polynomial_p.out'
            # is not currently implemented for the MPS device
            DecorateInfo(
                unittest.expectedFailure,
                "TestCommon",
                "test_variant_consistency_eager",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 660-677
```python
                device_type="mps",
            ),
            DecorateInfo(
                unittest.expectedFailure,
                "TestCommon",
                "test_promotes_int_to_float",
                device_type="mps",
            ),
            DecorateInfo(
                unittest.expectedFailure,
                "TestCommon",
                "test_out_warning",
                device_type="mps",
            ),
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
            DecorateInfo(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 678-695
```python
                unittest.expectedFailure,
                "TestCommon",
                "test_noncontiguous_samples",
                device_type="mps",
            ),
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_dtypes", device_type="mps"
            ),
        ),
        supports_one_python_scalar=True,
        supports_autograd=False,
    ),
    UnaryUfuncInfo(
        "special.modified_bessel_i0",
        decorators=(
            precisionOverride(
                {
                    torch.float32: 1e-03,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 696-713
```python
                    torch.float64: 1e-05,
                },
            ),
        ),
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        ref=scipy.special.i0 if TEST_SCIPY else None,
        supports_autograd=False,
    ),
    UnaryUfuncInfo(
        "special.modified_bessel_i1",
        decorators=(
            precisionOverride(
                {
                    torch.float32: 1e-03,
                    torch.float64: 1e-05,
                },
            ),
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 714-731
```python
        ),
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        ref=scipy.special.i1 if TEST_SCIPY else None,
        supports_autograd=False,
    ),
    UnaryUfuncInfo(
        "special.modified_bessel_k0",
        decorators=(
            precisionOverride(
                {
                    torch.float32: 1e-03,
                    torch.float64: 1e-05,
                },
            ),
        ),
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 732-749
```python
        ref=scipy.special.k0 if TEST_SCIPY else None,
        supports_autograd=False,
    ),
    UnaryUfuncInfo(
        "special.modified_bessel_k1",
        decorators=(
            precisionOverride(
                {
                    torch.float32: 1e-03,
                    torch.float64: 1e-05,
                },
            ),
        ),
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        ref=scipy.special.k1 if TEST_SCIPY else None,
        supports_autograd=False,
    ),
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 750-767
```python
    UnaryUfuncInfo(
        "special.scaled_modified_bessel_k0",
        decorators=(
            toleranceOverride(
                {
                    torch.float32: tol(atol=1e-03, rtol=1e-03),
                    torch.float64: tol(atol=1e-05, rtol=1e-03),
                }
            ),
        ),
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        ref=scipy.special.k0e if TEST_SCIPY else None,
        supports_autograd=False,
    ),
    UnaryUfuncInfo(
        "special.scaled_modified_bessel_k1",
        decorators=(
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 768-785
```python
            toleranceOverride(
                {
                    torch.float32: tol(atol=1e-03, rtol=1e-03),
                    torch.float64: tol(atol=1e-05, rtol=1e-03),
                }
            ),
        ),
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        ref=scipy.special.k1e if TEST_SCIPY else None,
        supports_autograd=False,
    ),
    BinaryUfuncInfo(
        "special.shifted_chebyshev_polynomial_t",
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        promotes_int_to_float=True,
        skips=(
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 786-803
```python
            DecorateInfo(unittest.skip("Skipped!"), "TestCudaFuserOpInfo"),
            DecorateInfo(unittest.skip("Skipped!"), "TestNNCOpInfo"),
            # Greatest absolute difference: nan
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_compare_cpu"),
        ),
        supports_one_python_scalar=True,
        supports_autograd=False,
    ),
    BinaryUfuncInfo(
        "special.shifted_chebyshev_polynomial_u",
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        promotes_int_to_float=True,
        skips=(
            DecorateInfo(unittest.skip("Skipped!"), "TestCudaFuserOpInfo"),
            DecorateInfo(unittest.skip("Skipped!"), "TestNNCOpInfo"),
            # Greatest absolute difference: nan
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_compare_cpu"),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 804-821
```python
        ),
        supports_one_python_scalar=True,
        supports_autograd=False,
    ),
    BinaryUfuncInfo(
        "special.shifted_chebyshev_polynomial_v",
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        promotes_int_to_float=True,
        skips=(
            DecorateInfo(unittest.skip("Skipped!"), "TestCudaFuserOpInfo"),
            DecorateInfo(unittest.skip("Skipped!"), "TestNNCOpInfo"),
            # Greatest absolute difference: nan
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_compare_cpu"),
        ),
        supports_one_python_scalar=True,
        supports_autograd=False,
    ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 822-839
```python
    BinaryUfuncInfo(
        "special.shifted_chebyshev_polynomial_w",
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        promotes_int_to_float=True,
        skips=(
            DecorateInfo(unittest.skip("Skipped!"), "TestCudaFuserOpInfo"),
            DecorateInfo(unittest.skip("Skipped!"), "TestNNCOpInfo"),
            # Greatest absolute difference: nan
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_compare_cpu"),
        ),
        supports_one_python_scalar=True,
        supports_autograd=False,
    ),
    UnaryUfuncInfo(
        "special.spherical_bessel_j0",
        decorators=(
            toleranceOverride(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 840-857
```python
                {
                    torch.float16: tol(atol=1e-02, rtol=1e-02),
                    torch.bfloat16: tol(atol=1e-02, rtol=1e-02),
                    torch.float32: tol(atol=1e-03, rtol=1e-03),
                    torch.float64: tol(atol=1e-05, rtol=1e-03),
                }
            ),
        ),
        dtypes=all_types_and(torch.bool, *_unsigned_int_types),
        dtypesIfMPS=all_types_and(torch.bool, torch.float16, torch.bfloat16),
        ref=lambda x: scipy.special.spherical_jn(0, x) if TEST_SCIPY else None,
        supports_autograd=False,
        skips=(
            DecorateInfo(
                unittest.skip(
                    "Scipy doesn't support bool inputs to spherical_bessel_j0"
                ),
                "TestUnaryUfuncs",
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 858-875
```python
                "test_reference_numerics_normal",
                dtypes=(torch.bool,),
            ),
        ),
    ),
]

python_ref_db: list[OpInfo] = [
    #
    # Elementwise Unary Special OpInfos
    #
    ElementwiseUnaryPythonRefInfo(
        "_refs.special.bessel_j0",
        torch_opinfo_name="special.bessel_j0",
        op_db=op_db,
        decorators=(
            precisionOverride(
                {
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 876-893
```python
                    torch.float32: 1e-04,
                    torch.float64: 1e-05,
                },
            ),
        ),
    ),
    ElementwiseUnaryPythonRefInfo(
        "_refs.special.bessel_j1",
        torch_opinfo_name="special.bessel_j1",
        op_db=op_db,
        decorators=(
            precisionOverride(
                {
                    torch.float32: 1e-04,
                    torch.float64: 1e-05,
                },
            ),
        ),
```
- EN: This block implements local helper logic for special. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 special 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 894-911
```python
    ),
    ElementwiseUnaryPythonRefInfo(
        "_refs.special.entr",
        torch_opinfo_name="special.entr",
        op_db=op_db,
        decorators=(precisionOverride({torch.float16: 1e-1, torch.bfloat16: 1e-1}),),
        skips=(
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestUnaryUfuncs",
                "test_reference_numerics_large",
                dtypes=[torch.bfloat16, torch.float16],
            ),
        ),
    ),
    ElementwiseUnaryPythonRefInfo(
        "_refs.special.erfcx",
        torch_opinfo_name="special.erfcx",
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 912-929
```python
        op_db=op_db,
        decorators=(
            toleranceOverride(
                {
                    torch.float32: tol(atol=0, rtol=4e-6),
                }
            ),
        ),
        skips=(
            DecorateInfo(
                unittest.expectedFailure,
                "TestCommon",
                "test_python_ref",
                device_type="mps",
                dtypes=(torch.float16,),
            ),
            DecorateInfo(
                unittest.expectedFailure,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 930-947
```python
                "TestCommon",
                "test_python_ref_torch_fallback",
                device_type="mps",
                dtypes=(torch.float16,),
            ),
        ),
    ),
    ElementwiseUnaryPythonRefInfo(
        "_refs.special.i0e",
        torch_opinfo_name="special.i0e",
        op_db=op_db,
        decorators=(precisionOverride({torch.bfloat16: 3e-1, torch.float16: 3e-1}),),
    ),
    ElementwiseUnaryPythonRefInfo(
        "_refs.special.i1",
        torch_opinfo_name="special.i1",
        op_db=op_db,
        decorators=(
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 948-965
```python
            DecorateInfo(
                toleranceOverride(
                    {
                        torch.float32: tol(atol=1e-4, rtol=0),
                        torch.bool: tol(atol=1e-4, rtol=0),
                    }
                )
            ),
        ),
        skips=(
            DecorateInfo(
                unittest.skip("Incorrect result!"),
                "TestUnaryUfuncs",
                "test_reference_numerics_large",
                dtypes=(torch.int8,),
            ),
        ),
    ),
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 966-983
```python
    ElementwiseUnaryPythonRefInfo(
        "_refs.special.i1e",
        torch_opinfo_name="special.i1e",
        op_db=op_db,
    ),
    ElementwiseUnaryPythonRefInfo(
        "_refs.special.log_ndtr",
        torch_opinfo_name="special.log_ndtr",
        op_db=op_db,
        skips=(
            # The operator 'aten::special_log_ndtr.out' is not currently implemented for the MPS device
            DecorateInfo(
                unittest.expectedFailure,
                "TestCommon",
                "test_python_ref",
                device_type="mps",
            ),
            DecorateInfo(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 984-1001
```python
                unittest.expectedFailure,
                "TestCommon",
                "test_python_ref_torch_fallback",
                device_type="mps",
            ),
        ),
    ),
    ElementwiseUnaryPythonRefInfo(
        "_refs.special.ndtr",
        torch_opinfo_name="special.ndtr",
        op_db=op_db,
    ),
    ElementwiseUnaryPythonRefInfo(
        "_refs.special.ndtri",
        torch_opinfo_name="special.ndtri",
        op_db=op_db,
        skips=(
            # The operator 'aten::special_ndtri.out' is not currently implemented for the MPS device
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1002-1019
```python
            DecorateInfo(unittest.expectedFailure, "TestCommon", device_type="mps"),
        ),
    ),
    ElementwiseUnaryPythonRefInfo(
        "_refs.special.spherical_bessel_j0",
        torch_opinfo_name="special.spherical_bessel_j0",
        op_db=op_db,
        decorators=(
            toleranceOverride(
                {
                    torch.float32: tol(atol=1e-03, rtol=1e-03),
                    torch.float64: tol(atol=1e-05, rtol=1e-03),
                }
            ),
        ),
        skips=(
            DecorateInfo(
                unittest.skip(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 1020-1037
```python
                    "Scipy doesn't support bool inputs to spherical_bessel_j0"
                ),
                "TestUnaryUfuncs",
                "test_reference_numerics_normal",
                dtypes=(torch.bool,),
            ),
            # Seems to fail on M2 but not M4:
            # Exception: Cannot convert a MPS Tensor to float64 dtype as the MPS framework doesn't support float64
            DecorateInfo(
                unittest.skip("Platform-specific error"),
                "TestCommon",
                "test_python_ref",
                device_type="mps",
                dtypes=(torch.float16,),
            ),
            DecorateInfo(
                unittest.skip("Platform-specific error"),
                "TestCommon",
```
- EN: This block handles tensor metadata or sample values; coordinates runtime execution state; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；协调运行时执行状态；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 1038-1055
```python
                "test_python_ref_torch_fallback",
                device_type="mps",
                dtypes=(torch.float16,),
            ),
        ),
    ),
    #
    # Elementwise Binary Special OpInfos
    #
    ElementwiseBinaryPythonRefInfo(
        "_refs.special.zeta",
        torch_opinfo_name="special.zeta",
        supports_one_python_scalar=True,
        op_db=op_db,
        skips=(
            # Reference reference_inputs nans and infs on cuda and nan, inf, 0., -inf for cpu
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_compare_cpu"),
        ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 1056-1057
```python
    ),
]
```
- EN: This block implements local helper logic for special. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 special 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Backend coverage — the file contains backend-specific behavior or validation paths.
  CN: 后端覆盖——该文件包含特定后端的行为或校验路径。
- EN: Runtime execution — the file schedules or executes native runtime programs and delegates.
  CN: 运行时执行——该文件会调度或执行原生运行时程序及其 delegate。
- EN: Test maintainability — the file helps standardize expectations, fixtures, or generated coverage across PyTorch tests.
  CN: 测试可维护性——该文件有助于在 PyTorch 测试中统一期望、夹具或生成式覆盖。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.testing`, `torch.testing._internal.common_device_type`, `torch.testing._internal.common_dtype`, `torch.testing._internal.common_utils`, `torch.testing._internal.opinfo.core`, `torch.testing._internal.opinfo.refs`, `torch.testing._internal.opinfo.utils`
- External imports / 外部导入: `unittest`, `functools`, `itertools`, `numpy`, `scipy.special`
- Representative symbols / 代表性符号: `sample_inputs_i0_i1`, `sample_inputs_polygamma`, `reference_polygamma`, `sample_inputs_entr`, `sample_inputs_erfcx`
