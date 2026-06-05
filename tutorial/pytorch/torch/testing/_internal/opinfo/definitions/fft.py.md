# fft.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/opinfo/definitions/fft.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for fft, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 fft 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9
```python
# mypy: ignore-errors

import unittest
from functools import partial

import numpy as np

import torch
from torch.testing import make_tensor
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.testing`; external imports: `unittest`, `functools`, `numpy`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.testing`；外部导入：`unittest`, `functools`, `numpy`。

### Lines 10-25
```python
from torch.testing._internal.common_device_type import precisionOverride
from torch.testing._internal.common_dtype import (
    all_types_and,
    all_types_and_complex_and,
)
from torch.testing._internal.common_utils import TEST_SCIPY, TEST_WITH_ROCM
from torch.testing._internal.opinfo.core import (
    DecorateInfo,
    ErrorInput,
    OpInfo,
    sample_inputs_spectral_ops,
    SampleInput,
    SpectralFuncInfo,
    SpectralFuncType,
)
from torch.testing._internal.opinfo.refs import (
```
- EN: This block handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 26-35
```python
    _find_referenced_opinfo,
    _inherit_constructor_args,
    PythonRefInfo,
)


has_scipy_fft = False
if TEST_SCIPY:
    try:
        import scipy.fft
```
- EN: This block handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流。关键符号：无明显局部符号。

### Lines 36-46
```python

        has_scipy_fft = True
    except ModuleNotFoundError:
        pass


class SpectralFuncPythonRefInfo(SpectralFuncInfo):
    """
    An OpInfo for a Python reference of an elementwise unary operation.
    """

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `SpectralFuncPythonRefInfo`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`SpectralFuncPythonRefInfo`。

### Lines 47-60
```python
    def __init__(
        self,
        name,  # the stringname of the callable Python reference
        *,
        op=None,  # the function variant of the operation, populated as torch.<name> if None
        torch_opinfo_name,  # the string name of the corresponding torch opinfo
        torch_opinfo_variant="",
        **kwargs,
    ):  # additional kwargs override kwargs inherited from the torch opinfo
        self.torch_opinfo_name = torch_opinfo_name
        self.torch_opinfo = _find_referenced_opinfo(
            torch_opinfo_name, torch_opinfo_variant, op_db=op_db
        )
        if not isinstance(self.torch_opinfo, SpectralFuncInfo):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `__init__`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`__init__`。

### Lines 61-69
```python
            raise AssertionError(
                f"Expected torch_opinfo to be SpectralFuncInfo, got {type(self.torch_opinfo)}"
            )

        inherited = self.torch_opinfo._original_spectral_func_args
        ukwargs = _inherit_constructor_args(name, op, inherited, kwargs)

        super().__init__(**ukwargs)

```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 70-79
```python

def error_inputs_fft(op_info, device, **kwargs):
    make_arg = partial(make_tensor, device=device, dtype=torch.float32)
    # Zero-dimensional tensor has no dimension to take FFT of
    yield ErrorInput(
        SampleInput(make_arg()),
        error_type=IndexError,
        error_regex="Dimension specified as -1 but tensor has no dimensions",
    )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `error_inputs_fft`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`error_inputs_fft`。

### Lines 80-89
```python

def error_inputs_fftn(op_info, device, **kwargs):
    make_arg = partial(make_tensor, device=device, dtype=torch.float32)
    # Specifying a dimension on a zero-dimensional tensor
    yield ErrorInput(
        SampleInput(make_arg(), dim=(0,)),
        error_type=IndexError,
        error_regex="Dimension specified as 0 but tensor has no dimensions",
    )

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `error_inputs_fftn`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`error_inputs_fftn`。

### Lines 90-100
```python

def sample_inputs_fft_with_min(
    op_info, device, dtype, requires_grad=False, *, min_size, **kwargs
):
    yield from sample_inputs_spectral_ops(
        op_info, device, dtype, requires_grad, **kwargs
    )
    if TEST_WITH_ROCM:
        # FIXME: Causes floating point exception on ROCm
        return

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_fft_with_min`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_fft_with_min`。

### Lines 101-109
```python
    # Check the "Invalid number of data points" error isn't too strict
    # https://github.com/pytorch/pytorch/pull/109083
    a = make_tensor(min_size, dtype=dtype, device=device, requires_grad=requires_grad)
    yield SampleInput(a)


def sample_inputs_fftshift(op_info, device, dtype, requires_grad, **kwargs):
    def mt(shape, **kwargs):
        return make_tensor(
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `sample_inputs_fftshift`, `mt`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`sample_inputs_fftshift`, `mt`。

### Lines 110-118
```python
            shape, device=device, dtype=dtype, requires_grad=requires_grad, **kwargs
        )

    yield SampleInput(mt((9, 10)))
    yield SampleInput(mt((50,)), kwargs=dict(dim=0))
    yield SampleInput(mt((5, 11)), kwargs=dict(dim=(1,)))
    yield SampleInput(mt((5, 6)), kwargs=dict(dim=(0, 1)))
    yield SampleInput(mt((5, 6, 2)), kwargs=dict(dim=(0, 2)))

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 119-136
```python

# Operator database
op_db: list[OpInfo] = [
    SpectralFuncInfo(
        "fft.fft",
        aten_name="fft_fft",
        decomp_aten_name="_fft_c2c",
        ref=np.fft.fft,
        ndimensional=SpectralFuncType.OneD,
        dtypes=all_types_and_complex_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
        dtypesIfCUDA=all_types_and_complex_and(torch.bool, torch.half, torch.complex32),
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=1),
        error_inputs_func=error_inputs_fft,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 137-154
```python
        # See https://github.com/pytorch/pytorch/pull/78358
        check_batched_forward_grad=False,
    ),
    SpectralFuncInfo(
        "fft.fft2",
        aten_name="fft_fft2",
        ref=np.fft.fft2,
        decomp_aten_name="_fft_c2c",
        ndimensional=SpectralFuncType.TwoD,
        dtypes=all_types_and_complex_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
        dtypesIfCUDA=all_types_and_complex_and(torch.bool, torch.half, torch.complex32),
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=(1, 1)),
        error_inputs_func=error_inputs_fftn,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 155-172
```python
        # See https://github.com/pytorch/pytorch/pull/78358
        check_batched_forward_grad=False,
        decorators=[precisionOverride({torch.float: 1e-4, torch.cfloat: 1e-4})],
        skips=(
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestCommon",
                "test_complex_half_reference_testing",
                device_type="cuda",
                dtypes=[torch.complex32],
                active_if=TEST_WITH_ROCM,
            ),
            # RuntimeError: [srcBuf length] > 0 INTERNAL ASSERT FAILED
            DecorateInfo(
                unittest.expectedFailure,
                "TestCommon",
                "test_out",
                device_type="mps",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 173-190
```python
            ),
            # AssertionError: The values for attribute 'shape' do not match: torch.Size([5, 3, 10]) != torch.Size([5, 3, 11]).
            DecorateInfo(
                unittest.expectedFailure,
                "TestCommon",
                "test_out_warning",
                device_type="mps",
            ),
        ),
    ),
    SpectralFuncInfo(
        "fft.fftn",
        aten_name="fft_fftn",
        decomp_aten_name="_fft_c2c",
        ref=np.fft.fftn,
        ndimensional=SpectralFuncType.ND,
        dtypes=all_types_and_complex_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 191-208
```python
        dtypesIfCUDA=all_types_and_complex_and(
            torch.bool,
            torch.half,
            torch.complex32,
        ),
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=(1, 1)),
        error_inputs_func=error_inputs_fftn,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        # See https://github.com/pytorch/pytorch/pull/78358
        check_batched_forward_grad=False,
        decorators=[precisionOverride({torch.float: 1e-4, torch.cfloat: 1e-4})],
        skips=(
            # RuntimeError: [srcBuf length] > 0 INTERNAL ASSERT FAILED
            DecorateInfo(
                unittest.expectedFailure,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 209-226
```python
                "TestCommon",
                "test_out",
                device_type="mps",
            ),
            # AssertionError: The values for attribute 'shape' do not match: torch.Size([5, 3, 10]) != torch.Size([5, 3, 11]).
            DecorateInfo(
                unittest.expectedFailure,
                "TestCommon",
                "test_out_warning",
                device_type="mps",
            ),
        ),
    ),
    SpectralFuncInfo(
        "fft.hfft",
        aten_name="fft_hfft",
        decomp_aten_name="_fft_c2r",
        ref=np.fft.hfft,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 227-244
```python
        ndimensional=SpectralFuncType.OneD,
        dtypes=all_types_and_complex_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
        dtypesIfCUDA=all_types_and_complex_and(
            torch.bool,
            torch.half,
            torch.complex32,
        ),
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=2),
        error_inputs_func=error_inputs_fft,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        # See https://github.com/pytorch/pytorch/pull/78358
        check_batched_forward_grad=False,
        check_batched_gradgrad=False,
        skips=(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 245-262
```python
            # Issue with conj and torch dispatch, see https://github.com/pytorch/pytorch/issues/82479
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestSchemaCheckModeOpInfo",
                "test_schema_correctness",
                dtypes=(torch.complex64, torch.complex128),
            ),
        ),
    ),
    SpectralFuncInfo(
        "fft.hfft2",
        aten_name="fft_hfft2",
        decomp_aten_name="_fft_c2r",
        ref=scipy.fft.hfft2 if has_scipy_fft else None,
        ndimensional=SpectralFuncType.TwoD,
        dtypes=all_types_and_complex_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
        dtypesIfCUDA=all_types_and_complex_and(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 263-280
```python
            torch.bool,
            torch.half,
            torch.complex32,
        ),
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=(2, 2)),
        error_inputs_func=error_inputs_fftn,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        check_batched_gradgrad=False,
        # See https://github.com/pytorch/pytorch/pull/78358
        check_batched_forward_grad=False,
        decorators=[
            DecorateInfo(
                precisionOverride({torch.float: 2e-4, torch.cfloat: 2e-4}),
                "TestFFT",
                "test_reference_nd",
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 281-298
```python
            ),
        ],
        skips=(
            # Issue with conj and torch dispatch, see https://github.com/pytorch/pytorch/issues/82479
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestSchemaCheckModeOpInfo",
                "test_schema_correctness",
            ),
            # FIXME: errors are too large; needs investigation
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestCommon",
                "test_complex_half_reference_testing",
                device_type="cuda",
            ),
        ),
    ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 299-316
```python
    SpectralFuncInfo(
        "fft.hfftn",
        aten_name="fft_hfftn",
        decomp_aten_name="_fft_c2r",
        ref=scipy.fft.hfftn if has_scipy_fft else None,
        ndimensional=SpectralFuncType.ND,
        dtypes=all_types_and_complex_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
        dtypesIfCUDA=all_types_and_complex_and(torch.bool, torch.half, torch.complex32),
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=(2, 2)),
        error_inputs_func=error_inputs_fftn,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        check_batched_gradgrad=False,
        # See https://github.com/pytorch/pytorch/pull/78358
        check_batched_forward_grad=False,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 317-334
```python
        decorators=[
            DecorateInfo(
                precisionOverride({torch.float: 2e-4, torch.cfloat: 2e-4}),
                "TestFFT",
                "test_reference_nd",
            ),
        ],
        skips=(
            # Issue with conj and torch dispatch, see https://github.com/pytorch/pytorch/issues/82479
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestSchemaCheckModeOpInfo",
                "test_schema_correctness",
            ),
        ),
    ),
    SpectralFuncInfo(
        "fft.rfft",
```
- EN: This block checks invariants or expected outcomes. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果。关键符号：无明显局部符号。

### Lines 335-352
```python
        aten_name="fft_rfft",
        decomp_aten_name="_fft_r2c",
        ref=np.fft.rfft,
        ndimensional=SpectralFuncType.OneD,
        dtypes=all_types_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
        dtypesIfCUDA=all_types_and(torch.bool, torch.half),
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=1),
        error_inputs_func=error_inputs_fft,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        check_batched_grad=False,
        check_batched_gradgrad=False,
    ),
    SpectralFuncInfo(
        "fft.rfft2",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 353-370
```python
        aten_name="fft_rfft2",
        decomp_aten_name="_fft_r2c",
        ref=np.fft.rfft2,
        ndimensional=SpectralFuncType.TwoD,
        dtypes=all_types_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
        dtypesIfCUDA=all_types_and(torch.bool, torch.half),
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=(1, 1)),
        error_inputs_func=error_inputs_fftn,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        check_batched_grad=False,
        check_batched_gradgrad=False,
        decorators=[
            precisionOverride({torch.float: 1e-4}),
        ],
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 371-388
```python
    ),
    SpectralFuncInfo(
        "fft.rfftn",
        aten_name="fft_rfftn",
        decomp_aten_name="_fft_r2c",
        ref=np.fft.rfftn,
        ndimensional=SpectralFuncType.ND,
        dtypes=all_types_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
        dtypesIfCUDA=all_types_and(torch.bool, torch.half),
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=(1, 1)),
        error_inputs_func=error_inputs_fftn,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        check_batched_grad=False,
        check_batched_gradgrad=False,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 389-406
```python
        decorators=[
            precisionOverride({torch.float: 1e-4}),
        ],
    ),
    SpectralFuncInfo(
        "fft.ifft",
        aten_name="fft_ifft",
        decomp_aten_name="_fft_c2c",
        ref=np.fft.ifft,
        ndimensional=SpectralFuncType.OneD,
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=1),
        error_inputs_func=error_inputs_fft,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        # See https://github.com/pytorch/pytorch/pull/78358
        check_batched_forward_grad=False,
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 407-424
```python
        dtypes=all_types_and_complex_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
        dtypesIfCUDA=all_types_and_complex_and(torch.bool, torch.half, torch.complex32),
        skips=(
            # AssertionError: Resizing an out= argument with no elements threw a resize warning!
            DecorateInfo(
                unittest.expectedFailure,
                "TestCommon",
                "test_out",
                device_type="mps",
            ),
        ),
    ),
    SpectralFuncInfo(
        "fft.ifft2",
        aten_name="fft_ifft2",
        decomp_aten_name="_fft_c2c",
        ref=np.fft.ifft2,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 425-442
```python
        ndimensional=SpectralFuncType.TwoD,
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=(1, 1)),
        error_inputs_func=error_inputs_fftn,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        # See https://github.com/pytorch/pytorch/pull/78358
        check_batched_forward_grad=False,
        dtypes=all_types_and_complex_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
        dtypesIfCUDA=all_types_and_complex_and(torch.bool, torch.half, torch.complex32),
        decorators=[
            DecorateInfo(
                precisionOverride({torch.float: 1e-4, torch.cfloat: 1e-4}),
                "TestFFT",
                "test_reference_nd",
            )
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 443-460
```python
        ],
        skips=(
            # RuntimeError: [srcBuf length] > 0 INTERNAL ASSERT FAILED
            DecorateInfo(
                unittest.expectedFailure,
                "TestCommon",
                "test_out",
                device_type="mps",
            ),
            # AssertionError: The values for attribute 'shape' do not match: torch.Size([5, 3, 10]) != torch.Size([5, 3, 11]).
            DecorateInfo(
                unittest.expectedFailure,
                "TestCommon",
                "test_out_warning",
                device_type="mps",
            ),
        ),
    ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 461-478
```python
    SpectralFuncInfo(
        "fft.ifftn",
        aten_name="fft_ifftn",
        decomp_aten_name="_fft_c2c",
        ref=np.fft.ifftn,
        ndimensional=SpectralFuncType.ND,
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=(1, 1)),
        error_inputs_func=error_inputs_fftn,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        # See https://github.com/pytorch/pytorch/pull/78358
        check_batched_forward_grad=False,
        dtypes=all_types_and_complex_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
        dtypesIfCUDA=all_types_and_complex_and(
            torch.bool,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 479-496
```python
            torch.half,
            torch.complex32,
        ),
        decorators=[
            DecorateInfo(
                precisionOverride({torch.float: 1e-4, torch.cfloat: 1e-4}),
                "TestFFT",
                "test_reference_nd",
            )
        ],
        skips=(
            # RuntimeError: [srcBuf length] > 0 INTERNAL ASSERT FAILED
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
            # AssertionError: The values for attribute 'shape' do not match: torch.Size([5, 3, 10]) != torch.Size([5, 3, 11]).
            DecorateInfo(
                unittest.expectedFailure,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 497-514
```python
                "TestCommon",
                "test_out_warning",
                device_type="mps",
            ),
        ),
    ),
    SpectralFuncInfo(
        "fft.ihfft",
        aten_name="fft_ihfft",
        decomp_aten_name="_fft_r2c",
        ref=np.fft.ihfft,
        ndimensional=SpectralFuncType.OneD,
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=(1, 1)),
        error_inputs_func=error_inputs_fft,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        # See https://github.com/pytorch/pytorch/pull/78358
        check_batched_forward_grad=False,
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 515-532
```python
        dtypes=all_types_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
        dtypesIfCUDA=all_types_and(torch.bool, torch.half),
        skips=(
            # AssertionError: Resizing an out= argument with no elements threw a resize warning!
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
        ),
        check_batched_grad=False,
    ),
    SpectralFuncInfo(
        "fft.ihfft2",
        aten_name="fft_ihfft2",
        decomp_aten_name="_fft_r2c",
        ref=scipy.fft.ihfftn if has_scipy_fft else None,
        ndimensional=SpectralFuncType.TwoD,
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=(1, 1)),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 533-550
```python
        error_inputs_func=error_inputs_fftn,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        # See https://github.com/pytorch/pytorch/pull/78358
        check_batched_forward_grad=False,
        dtypes=all_types_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
        dtypesIfCUDA=all_types_and(torch.bool, torch.half),
        check_batched_grad=False,
        check_batched_gradgrad=False,
        decorators=(
            # The values for attribute 'shape' do not match: torch.Size([5, 6, 5]) != torch.Size([5, 6, 6]).
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_out_warning"),
            DecorateInfo(
                precisionOverride({torch.float: 2e-4}), "TestFFT", "test_reference_nd"
            ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 551-568
```python
            # Mismatched elements!
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_out"),
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_out_warnings"),
        ),
    ),
    SpectralFuncInfo(
        "fft.ihfftn",
        aten_name="fft_ihfftn",
        decomp_aten_name="_fft_r2c",
        ref=scipy.fft.ihfftn if has_scipy_fft else None,
        ndimensional=SpectralFuncType.ND,
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=(1, 1)),
        error_inputs_func=error_inputs_fftn,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        # See https://github.com/pytorch/pytorch/pull/78358
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 569-586
```python
        check_batched_forward_grad=False,
        dtypes=all_types_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archss
        dtypesIfCUDA=all_types_and(torch.bool, torch.half),
        check_batched_grad=False,
        check_batched_gradgrad=False,
        decorators=[
            # The values for attribute 'shape' do not match: torch.Size([5, 6, 5]) != torch.Size([5, 6, 6]).
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_out_warning"),
            # Mismatched elements!
            DecorateInfo(unittest.expectedFailure, "TestCommon", "test_out"),
            DecorateInfo(
                precisionOverride({torch.float: 2e-4}), "TestFFT", "test_reference_nd"
            ),
        ],
    ),
    SpectralFuncInfo(
        "fft.irfft",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 587-604
```python
        aten_name="fft_irfft",
        decomp_aten_name="_fft_c2r",
        ref=np.fft.irfft,
        ndimensional=SpectralFuncType.OneD,
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=(1, 2)),
        error_inputs_func=error_inputs_fft,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        # See https://github.com/pytorch/pytorch/pull/78358
        check_batched_forward_grad=False,
        dtypes=all_types_and_complex_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
        dtypesIfCUDA=all_types_and_complex_and(torch.bool, torch.half, torch.complex32),
        check_batched_gradgrad=False,
    ),
    SpectralFuncInfo(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 605-622
```python
        "fft.irfft2",
        aten_name="fft_irfft2",
        decomp_aten_name="_fft_c2r",
        ref=np.fft.irfft2,
        ndimensional=SpectralFuncType.TwoD,
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=(1, 2)),
        error_inputs_func=error_inputs_fftn,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
        # See https://github.com/pytorch/pytorch/pull/78358
        check_batched_forward_grad=False,
        dtypes=all_types_and_complex_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
        dtypesIfCUDA=all_types_and_complex_and(torch.bool, torch.half, torch.complex32),
        check_batched_gradgrad=False,
        decorators=[
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 623-640
```python
            DecorateInfo(
                precisionOverride({torch.float: 1e-4, torch.cfloat: 1e-4}),
                "TestFFT",
                "test_reference_nd",
            )
        ],
    ),
    SpectralFuncInfo(
        "fft.irfftn",
        aten_name="fft_irfftn",
        decomp_aten_name="_fft_c2r",
        ref=np.fft.irfftn,
        ndimensional=SpectralFuncType.ND,
        sample_inputs_func=partial(sample_inputs_fft_with_min, min_size=(1, 2)),
        error_inputs_func=error_inputs_fftn,
        # https://github.com/pytorch/pytorch/issues/80411
        gradcheck_fast_mode=True,
        supports_forward_ad=True,
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 641-658
```python
        supports_fwgrad_bwgrad=True,
        # See https://github.com/pytorch/pytorch/pull/78358
        check_batched_forward_grad=False,
        dtypes=all_types_and_complex_and(torch.bool),
        # CUDA supports Half/ComplexHalf Precision FFT only on SM53 or later archs
        dtypesIfCUDA=all_types_and_complex_and(torch.bool, torch.half, torch.complex32),
        check_batched_gradgrad=False,
        decorators=[
            DecorateInfo(
                precisionOverride({torch.float: 1e-4, torch.cfloat: 1e-4}),
                "TestFFT",
                "test_reference_nd",
            )
        ],
    ),
    OpInfo(
        "fft.fftshift",
        dtypes=all_types_and_complex_and(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 659-676
```python
            torch.bool, torch.bfloat16, torch.half, torch.chalf
        ),
        sample_inputs_func=sample_inputs_fftshift,
        supports_out=False,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
    ),
    OpInfo(
        "fft.ifftshift",
        dtypes=all_types_and_complex_and(
            torch.bool, torch.bfloat16, torch.half, torch.chalf
        ),
        sample_inputs_func=sample_inputs_fftshift,
        supports_out=False,
        supports_forward_ad=True,
        supports_fwgrad_bwgrad=True,
    ),
]
```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 677-694
```python

python_ref_db: list[OpInfo] = [
    SpectralFuncPythonRefInfo(
        "_refs.fft.fft",
        torch_opinfo_name="fft.fft",
        skips=(
            # AssertionError: Resizing an out= argument with no elements threw a resize warning!
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
        ),
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.ifft",
        torch_opinfo_name="fft.ifft",
        skips=(
            # AssertionError: Resizing an out= argument with no elements threw a resize warning!
            DecorateInfo(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 695-712
```python
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
        ),
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.rfft",
        torch_opinfo_name="fft.rfft",
        skips=(
            # AssertionError: Resizing an out= argument with no elements threw a resize warning!
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
        ),
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.irfft",
        torch_opinfo_name="fft.irfft",
        skips=(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 713-730
```python
            # AssertionError: Resizing an out= argument with no elements threw a resize warning!
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
        ),
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.hfft",
        torch_opinfo_name="fft.hfft",
        skips=(
            # AssertionError: Resizing an out= argument with no elements threw a resize warning!
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
        ),
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.ihfft",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 731-748
```python
        torch_opinfo_name="fft.ihfft",
        skips=(
            # AssertionError: Resizing an out= argument with no elements threw a resize warning!
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
        ),
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.fftn",
        torch_opinfo_name="fft.fftn",
        decorators=[
            DecorateInfo(
                precisionOverride({torch.float: 1e-4, torch.cfloat: 1e-4}),
                "TestFFT",
                "test_reference_nd",
            )
        ],
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 749-766
```python
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.ifftn",
        torch_opinfo_name="fft.ifftn",
        decorators=[
            DecorateInfo(
                precisionOverride({torch.float: 1e-4, torch.cfloat: 1e-4}),
                "TestFFT",
                "test_reference_nd",
            )
        ],
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.rfftn",
        torch_opinfo_name="fft.rfftn",
        skips=(
            # AssertionError: Resizing an out= argument with no elements threw a resize warning!
            DecorateInfo(
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 767-784
```python
                unittest.expectedFailure,
                "TestCommon",
                "test_out",
                device_type="mps",
            ),
        ),
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.irfftn",
        torch_opinfo_name="fft.irfftn",
        decorators=[
            DecorateInfo(
                precisionOverride({torch.float: 1e-4, torch.cfloat: 1e-4}),
                "TestFFT",
                "test_reference_nd",
            )
        ],
        skips=(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 785-802
```python
            # AssertionError: Resizing an out= argument with no elements threw a resize warning!
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
        ),
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.hfftn",
        torch_opinfo_name="fft.hfftn",
        decorators=[
            DecorateInfo(
                precisionOverride({torch.float: 2e-4, torch.cfloat: 2e-4}),
                "TestFFT",
                "test_reference_nd",
            )
        ],
        skips=(
            # AssertionError: Resizing an out= argument with no elements threw a resize warning!
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 803-820
```python
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
        ),
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.ihfftn",
        torch_opinfo_name="fft.ihfftn",
        decorators=[
            DecorateInfo(
                precisionOverride({torch.float: 2e-4}),
                "TestFFT",
                "test_reference_nd",
            ),
            # AssertionError: Reference result was farther (0.09746177145360499) from the precise
            # computation than the torch result was (0.09111555632069855)
            # See https://github.com/pytorch/pytorch/pull/170856 for more details.
            DecorateInfo(
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 821-838
```python
                unittest.skip("Skipped!"),
                "TestCommon",
                "test_python_ref_torch_fallback",
                dtypes=(torch.float16,),
                device_type="cuda",
            ),
            # AssertionError: Reference result was farther (0.0953431016138116) from the precise
            # computation than the torch result was (0.09305490684430734)
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestCommon",
                "test_python_ref_executor",
                dtypes=(torch.float16,),
                device_type="cuda",
            ),
        ],
        skips=(
            # AssertionError: Resizing an out= argument with no elements threw a resize warning!
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; coordinates runtime execution state; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；协调运行时执行状态；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 839-856
```python
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
        ),
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.fft2",
        torch_opinfo_name="fft.fft2",
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.ifft2",
        torch_opinfo_name="fft.ifft2",
        decorators=[
            DecorateInfo(
                precisionOverride({torch.float: 1e-4, torch.cfloat: 1e-4}),
                "TestFFT",
                "test_reference_nd",
            )
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 857-874
```python
        ],
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.rfft2",
        torch_opinfo_name="fft.rfft2",
        skips=(
            # AssertionError: Resizing an out= argument with no elements threw a resize warning!
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
        ),
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.irfft2",
        torch_opinfo_name="fft.irfft2",
        decorators=[
            DecorateInfo(
                precisionOverride({torch.float: 1e-4, torch.cfloat: 1e-4}),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 875-892
```python
                "TestFFT",
                "test_reference_nd",
            )
        ],
        skips=(
            # AssertionError: Resizing an out= argument with no elements threw a resize warning!
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
        ),
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.hfft2",
        torch_opinfo_name="fft.hfft2",
        decorators=[
            DecorateInfo(
                precisionOverride({torch.float: 2e-4, torch.cfloat: 2e-4}),
                "TestFFT",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 893-910
```python
                "test_reference_nd",
            )
        ],
        skips=(
            # AssertionError: Resizing an out= argument with no elements threw a resize warning!
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
        ),
    ),
    SpectralFuncPythonRefInfo(
        "_refs.fft.ihfft2",
        torch_opinfo_name="fft.ihfft2",
        decorators=[
            DecorateInfo(
                precisionOverride({torch.float: 2e-4}),
                "TestFFT",
                "test_reference_nd",
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 911-928
```python
            ),
            # FIXME:
            # Reference result was farther (0.0953431016138116) from the precise computation
            # than the torch result was (0.09305490684430734)!
            DecorateInfo(
                unittest.skip("Skipped!"),
                "TestCommon",
                "test_python_ref_executor",
                device_type="cuda",
            ),
        ],
        skips=(
            # AssertionError: Resizing an out= argument with no elements threw a resize warning!
            DecorateInfo(
                unittest.expectedFailure, "TestCommon", "test_out", device_type="mps"
            ),
        ),
    ),
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; coordinates runtime execution state; reports or normalizes error conditions. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；协调运行时执行状态；报告或规范化错误情况。关键符号：无明显局部符号。

### Lines 929-939
```python
    PythonRefInfo(
        "_refs.fft.fftshift",
        op_db=op_db,
        torch_opinfo_name="fft.fftshift",
    ),
    PythonRefInfo(
        "_refs.fft.ifftshift",
        op_db=op_db,
        torch_opinfo_name="fft.ifftshift",
    ),
]
```
- EN: This block implements local helper logic for fft. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 fft 相关的局部辅助逻辑。关键符号：无明显局部符号。


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
- Internal imports / 内部导入: `torch`, `torch.testing`, `torch.testing._internal.common_device_type`, `torch.testing._internal.common_dtype`, `torch.testing._internal.common_utils`, `torch.testing._internal.opinfo.core`, `torch.testing._internal.opinfo.refs`
- External imports / 外部导入: `unittest`, `functools`, `numpy`, `scipy.fft`
- Representative symbols / 代表性符号: `SpectralFuncPythonRefInfo`, `error_inputs_fft`, `error_inputs_fftn`, `sample_inputs_fft_with_min`, `sample_inputs_fftshift`
