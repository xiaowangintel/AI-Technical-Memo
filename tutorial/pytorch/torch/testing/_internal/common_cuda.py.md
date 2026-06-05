# common_cuda.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/common_cuda.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for common cuda, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 common cuda 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7
```python
# mypy: ignore-errors

r"""This file is allowed to initialize CUDA context when imported."""

import functools
import torch
import torch.cuda
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.cuda`; external imports: `functools`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.cuda`；外部导入：`functools`。

### Lines 8-14
```python
from torch.testing._internal.common_utils import LazyVal, TEST_NUMBA, TEST_WITH_ROCM, TEST_CUDA, IS_WINDOWS, IS_MACOS, TEST_XPU
import inspect
import contextlib
import os
import unittest


```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.testing._internal.common_utils`; external imports: `inspect`, `contextlib`, `os`, `unittest`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.testing._internal.common_utils`；外部导入：`inspect`, `contextlib`, `os`, `unittest`。

### Lines 15-21
```python
CUDA_ALREADY_INITIALIZED_ON_IMPORT = torch.cuda.is_initialized()


TEST_MULTIGPU = TEST_CUDA and torch.cuda.device_count() >= 2
CUDA_DEVICE = torch.device("cuda:0") if TEST_CUDA else None
# note: if ROCm is targeted, TEST_CUDNN is code for TEST_MIOPEN
if TEST_WITH_ROCM:
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 22-28
```python
    TEST_CUDNN = LazyVal(lambda: TEST_CUDA)
else:
    TEST_CUDNN = LazyVal(lambda: TEST_CUDA and torch.backends.cudnn.is_acceptable(torch.tensor(1., device=CUDA_DEVICE)))

TEST_CUDNN_VERSION = LazyVal(lambda: torch.backends.cudnn.version() if TEST_CUDNN else 0)
ROCM_VERSION = LazyVal(lambda : tuple(int(v) for v in torch.version.hip.split('.')[:2]) if torch.version.hip else (0, 0))

```
- EN: This block handles tensor metadata or sample values; bridges to backend-specific execution artifacts; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；桥接到特定后端的执行产物；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 29-38
```python
SM53OrLater = LazyVal(lambda: torch.cuda.is_available() and torch.cuda.get_device_capability() >= (5, 3))
SM60OrLater = LazyVal(lambda: torch.cuda.is_available() and torch.cuda.get_device_capability() >= (6, 0))
SM70OrLater = LazyVal(lambda: torch.cuda.is_available() and torch.cuda.get_device_capability() >= (7, 0))
SM75OrLater = LazyVal(lambda: torch.cuda.is_available() and torch.cuda.get_device_capability() >= (7, 5))
SM80OrLater = LazyVal(lambda: torch.cuda.is_available() and torch.cuda.get_device_capability() >= (8, 0))
SM89OrLater = LazyVal(lambda: torch.cuda.is_available() and torch.cuda.get_device_capability() >= (8, 9))
SM90OrLater = LazyVal(lambda: torch.cuda.is_available() and torch.cuda.get_device_capability() >= (9, 0))
SM100OrLater = LazyVal(lambda: torch.cuda.is_available() and torch.cuda.get_device_capability() >= (10, 0))
SM120OrLater = LazyVal(lambda: torch.cuda.is_available() and torch.cuda.get_device_capability() >= (12, 0))

```
- EN: This block handles tensor metadata or sample values. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值。关键符号：无明显局部符号。

### Lines 39-47
```python
IS_THOR = LazyVal(lambda: torch.cuda.is_available() and torch.version.cuda is not None and
                  ((torch.cuda.get_device_capability() == (11, 0) and int(torch.version.cuda[:2]) >= 13) or
                   (torch.cuda.get_device_capability() == (10, 1) and int(torch.version.cuda[:2]) < 13)))
IS_JETSON = LazyVal(lambda: torch.cuda.is_available() and (torch.cuda.get_device_capability() in [(7, 2), (8, 7)] or IS_THOR))
IS_SM89 = LazyVal(lambda: torch.cuda.is_available() and torch.cuda.get_device_capability() == (8, 9))
IS_SM90 = LazyVal(lambda: torch.cuda.is_available() and torch.cuda.get_device_capability() == (9, 0))
IS_SM100 = LazyVal(lambda: torch.cuda.is_available() and torch.cuda.get_device_capability() == (10, 0))
IS_SM12X = LazyVal(lambda: torch.cuda.is_available() and torch.cuda.get_device_capability()[0] == 12)

```
- EN: This block handles tensor metadata or sample values. Key symbols: `IS_THOR`, `IS_JETSON`, `IS_SM89`, `IS_SM90`, `IS_SM100`, `IS_SM12X`.
- CN: 该代码块处理张量元数据或示例值。关键符号：`IS_THOR`, `IS_JETSON`, `IS_SM89`, `IS_SM90`, `IS_SM100`, `IS_SM12X`。

### Lines 48-56
```python
@contextlib.contextmanager
def blas_library_context(backend):
    prev_backend = torch.backends.cuda.preferred_blas_library()
    torch.backends.cuda.preferred_blas_library(backend)
    try:
        yield
    finally:
        torch.backends.cuda.preferred_blas_library(prev_backend)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `blas_library_context`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`blas_library_context`。

### Lines 57-64
```python
def evaluate_gfx_arch_within(arch_list):
    if not torch.cuda.is_available():
        return False
    gcn_arch_name = torch.cuda.get_device_properties('cuda').gcnArchName
    effective_arch = os.environ.get('PYTORCH_DEBUG_FLASH_ATTENTION_GCN_ARCH_OVERRIDE', gcn_arch_name)
    # gcnArchName can be complicated strings like gfx90a:sramecc+:xnack-
    # Hence the matching should be done reversely
    return any(arch in effective_arch for arch in arch_list)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `evaluate_gfx_arch_within`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`evaluate_gfx_arch_within`。

### Lines 65-71
```python

def CDNA3OrLater():
    return evaluate_gfx_arch_within(["gfx942", "gfx950"])

def CDNA2OrLater():
    return evaluate_gfx_arch_within(["gfx90a", "gfx942"])

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `CDNA3OrLater`, `CDNA2OrLater`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`CDNA3OrLater`, `CDNA2OrLater`。

### Lines 72-78
```python
def evaluate_platform_supports_flash_attention():
    if TEST_WITH_ROCM:
        arch_list = ["gfx90a", "gfx942", "gfx1100", "gfx1201", "gfx950"]
        if os.environ.get("TORCH_ROCM_AOTRITON_ENABLE_EXPERIMENTAL", "0") != "0":
            arch_list += ["gfx1101", "gfx1102", "gfx1150", "gfx1151", "gfx1200"]
        return evaluate_gfx_arch_within(arch_list)
    if TEST_CUDA:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `evaluate_platform_supports_flash_attention`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`evaluate_platform_supports_flash_attention`。

### Lines 79-85
```python
        return not IS_WINDOWS and SM80OrLater
    if TEST_XPU:
        return True
    return False

def evaluate_platform_supports_ck_sdpa():
    if TEST_WITH_ROCM:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `evaluate_platform_supports_ck_sdpa`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`evaluate_platform_supports_ck_sdpa`。

### Lines 86-93
```python
        return torch.backends.cuda.is_ck_sdpa_available()
    else:
        return False

def evaluate_platform_supports_efficient_attention():
    if TEST_WITH_ROCM:
        arch_list = ["gfx90a", "gfx942", "gfx1100", "gfx1201", "gfx950"]
        if os.environ.get("TORCH_ROCM_AOTRITON_ENABLE_EXPERIMENTAL", "0") != "0":
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `evaluate_platform_supports_efficient_attention`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`evaluate_platform_supports_efficient_attention`。

### Lines 94-100
```python
            arch_list += ["gfx1101", "gfx1102", "gfx1150", "gfx1151", "gfx1200"]
        return evaluate_gfx_arch_within(arch_list)
    if TEST_CUDA:
        return True
    if TEST_XPU:
        return True
    return False
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 101-107
```python

def evaluate_platform_supports_cudnn_attention():
    return (not TEST_WITH_ROCM) and SM80OrLater and (TEST_CUDNN_VERSION >= 90000)

def evaluate_platform_supports_green_context():
    if IS_WINDOWS:
        return False
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `evaluate_platform_supports_cudnn_attention`, `evaluate_platform_supports_green_context`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`evaluate_platform_supports_cudnn_attention`, `evaluate_platform_supports_green_context`。

### Lines 108-114
```python
    if not _get_torch_cuda_version() >= (12, 8):
        return False
    driver_version = torch.utils.collect_env.get_nvidia_driver_version(torch.utils.collect_env.run)
    if driver_version is None:
        return False
    return int(driver_version.split('.')[0]) >= 570

```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 115-122
```python
PLATFORM_SUPPORTS_FLASH_ATTENTION: bool = LazyVal(lambda: evaluate_platform_supports_flash_attention())
PLATFORM_SUPPORTS_MEM_EFF_ATTENTION: bool = LazyVal(lambda: evaluate_platform_supports_efficient_attention())
PLATFORM_SUPPORTS_CUDNN_ATTENTION: bool = LazyVal(lambda: evaluate_platform_supports_cudnn_attention())
# This condition always evaluates to PLATFORM_SUPPORTS_MEM_EFF_ATTENTION but for logical clarity we keep it separate
PLATFORM_SUPPORTS_FUSED_ATTENTION: bool = LazyVal(lambda: PLATFORM_SUPPORTS_FLASH_ATTENTION or
                                                  PLATFORM_SUPPORTS_CUDNN_ATTENTION or
                                                  PLATFORM_SUPPORTS_MEM_EFF_ATTENTION)

```
- EN: This block iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 123-129
```python
PLATFORM_SUPPORTS_FUSED_SDPA: bool = TEST_CUDA and not TEST_WITH_ROCM

PLATFORM_SUPPORTS_CK_SDPA: bool = LazyVal(lambda: evaluate_platform_supports_ck_sdpa())


def evaluate_platform_supports_bf16():
    if torch.version.cuda:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `evaluate_platform_supports_bf16`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`evaluate_platform_supports_bf16`。

### Lines 130-136
```python
        return SM80OrLater
    elif torch.version.hip:
        return True
    elif TEST_XPU:
        return True
    return False

```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 137-143
```python

def evaluate_platform_supports_bf16_atomics():
    if torch.version.cuda:
        return SM80OrLater
    elif torch.version.hip:
        return ROCM_VERSION >= (8, 0)
    return False
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `evaluate_platform_supports_bf16_atomics`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`evaluate_platform_supports_bf16_atomics`。

### Lines 144-150
```python


def evaluate_platform_supports_half_atomics():
    if torch.version.hip:
        return ROCM_VERSION >= (8, 0)
    return True

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `evaluate_platform_supports_half_atomics`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`evaluate_platform_supports_half_atomics`。

### Lines 151-157
```python

PLATFORM_SUPPORTS_BF16: bool = LazyVal(lambda: evaluate_platform_supports_bf16())
PLATFORM_SUPPORTS_BF16_ATOMICS: bool = LazyVal(lambda: evaluate_platform_supports_bf16_atomics())
PLATFORM_SUPPORTS_HALF_ATOMICS: bool = LazyVal(lambda: evaluate_platform_supports_half_atomics())

PLATFORM_SUPPORTS_GREEN_CONTEXT: bool = LazyVal(lambda: evaluate_platform_supports_green_context())

```
- EN: This block protects shared state or ordering guarantees. Key symbols: no dominant local symbols.
- CN: 该代码块保护共享状态或执行顺序保证。关键符号：无明显局部符号。

### Lines 158-164
```python
def evaluate_platform_supports_workqueue_config():
    if IS_WINDOWS:
        return False
    if not _get_torch_cuda_version() >= (13, 1):
        return False
    driver_version = torch.utils.collect_env.get_nvidia_driver_version(torch.utils.collect_env.run)
    if driver_version is None:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `evaluate_platform_supports_workqueue_config`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`evaluate_platform_supports_workqueue_config`。

### Lines 165-171
```python
        return False
    return int(driver_version.split('.')[0]) >= 590

PLATFORM_SUPPORTS_WORKQUEUE_CONFIG: bool = LazyVal(lambda: evaluate_platform_supports_workqueue_config())

def evaluate_platform_supports_fp8():
    if torch.cuda.is_available():
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `evaluate_platform_supports_fp8`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`evaluate_platform_supports_fp8`。

### Lines 172-178
```python
        if torch.version.hip:
            archs = ['gfx94']
            if ROCM_VERSION >= (6, 3):
                archs.extend(['gfx120'])
            if ROCM_VERSION >= (6, 5):
                archs.append('gfx95')
            for arch in archs:
```
- EN: This block manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块操作图状程序结构；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 179-185
```python
                if arch in torch.cuda.get_device_properties(0).gcnArchName:
                    return True
            return False
        else:
            return SM90OrLater or torch.cuda.get_device_capability() == (8, 9)
    if torch.xpu.is_available():
        return True
```
- EN: This block handles tensor metadata or sample values; handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 186-192
```python
    # As CPU supports FP8 and is always available, return True.
    return True

def evaluate_platform_supports_fp8_grouped_gemm():
    if torch.cuda.is_available():
        if torch.version.hip:
            if "USE_MSLK" not in torch.__config__.show():
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `evaluate_platform_supports_fp8_grouped_gemm`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`evaluate_platform_supports_fp8_grouped_gemm`。

### Lines 193-199
```python
                return False
            archs = ['gfx942', 'gfx950']
            for arch in archs:
                if arch in torch.cuda.get_device_properties(0).gcnArchName:
                    return True
        else:
            return SM90OrLater and not SM100OrLater
```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures; handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构；处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 200-206
```python
    return False

def evaluate_platform_supports_mx_gemm():
    if torch.cuda.is_available():
        if torch.version.hip:
            if ROCM_VERSION >= (7, 0):
                return 'gfx950' in torch.cuda.get_device_properties(0).gcnArchName
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `evaluate_platform_supports_mx_gemm`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`evaluate_platform_supports_mx_gemm`。

### Lines 207-214
```python
        else:
            return SM100OrLater
    return False

def evaluate_platform_supports_mxfp8_grouped_gemm():
    if torch.cuda.is_available() and not torch.version.hip:
        built_with_mslk = "USE_MSLK" in torch.__config__.show()
        return built_with_mslk and IS_SM100
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `evaluate_platform_supports_mxfp8_grouped_gemm`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`evaluate_platform_supports_mxfp8_grouped_gemm`。

### Lines 215-222
```python
    return False

def evaluate_platform_supports_fp8_sparse():
    if torch.cuda.is_available():
        if torch.version.hip:
            return 'gfx950' in torch.cuda.get_device_properties(0).gcnArchName
        else:
            return (
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `evaluate_platform_supports_fp8_sparse`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`evaluate_platform_supports_fp8_sparse`。

### Lines 223-234
```python
                (SM90OrLater or torch.cuda.get_device_capability() == (8, 9))
                and torch.backends.cusparselt.is_available()
                and torch.backends.cusparselt.version() >= 602
            )
    return False

PLATFORM_SUPPORTS_MX_GEMM: bool = LazyVal(lambda: evaluate_platform_supports_mx_gemm())
PLATFORM_SUPPORTS_FP8: bool = LazyVal(lambda: evaluate_platform_supports_fp8())
PLATFORM_SUPPORTS_FP8_SPARSE: bool = LazyVal(lambda: evaluate_platform_supports_fp8_sparse())
PLATFORM_SUPPORTS_FP8_GROUPED_GEMM: bool = LazyVal(lambda: evaluate_platform_supports_fp8_grouped_gemm())
PLATFORM_SUPPORTS_MXFP8_GROUPED_GEMM: bool = LazyVal(lambda: evaluate_platform_supports_mxfp8_grouped_gemm())

```
- EN: This block handles tensor metadata or sample values; bridges to backend-specific execution artifacts; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；桥接到特定后端的执行产物；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 235-244
```python
if TEST_NUMBA:
    try:
        import numba.cuda
        TEST_NUMBA_CUDA = numba.cuda.is_available()
    except (ImportError, RuntimeError, OSError):
        TEST_NUMBA_CUDA = False
        TEST_NUMBA = False
else:
    TEST_NUMBA_CUDA = False

```
- EN: This block reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。

### Lines 245-251
```python
# Used below in `initialize_cuda_context_rng` to ensure that CUDA context and
# RNG have been initialized.
__cuda_ctx_rng_initialized = False


# after this call, CUDA context and RNG must have been initialized on each GPU
def initialize_cuda_context_rng():
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `initialize_cuda_context_rng`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`initialize_cuda_context_rng`。

### Lines 252-260
```python
    global __cuda_ctx_rng_initialized
    if not TEST_CUDA:
        raise AssertionError('CUDA must be available when calling initialize_cuda_context_rng')
    if not __cuda_ctx_rng_initialized:
        # initialize cuda context and rng for memory tests
        for i in range(torch.cuda.device_count()):
            torch.randn(1, device=f"cuda:{i}")
        __cuda_ctx_rng_initialized = True

```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; reports or normalizes error conditions; reuses computed state to reduce repeated work; handles conditional control flow; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；报告或规范化错误情况；复用已计算状态以减少重复工作；处理条件控制流；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 261-271
```python

@contextlib.contextmanager
def tf32_off():
    old_allow_tf32_matmul = torch.backends.cuda.matmul.allow_tf32
    try:
        torch.backends.cuda.matmul.allow_tf32 = False
        with torch.backends.cudnn.flags(enabled=None, benchmark=None, deterministic=None, allow_tf32=False):
            yield
    finally:
        torch.backends.cuda.matmul.allow_tf32 = old_allow_tf32_matmul

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `tf32_off`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`tf32_off`。

### Lines 272-285
```python

@contextlib.contextmanager
def tf32_on(self, tf32_precision=1e-5):
    old_allow_tf32_matmul = torch.backends.cuda.matmul.allow_tf32
    old_precision = self.precision
    try:
        torch.backends.cuda.matmul.allow_tf32 = True
        self.precision = tf32_precision
        with torch.backends.cudnn.flags(enabled=None, benchmark=None, deterministic=None, allow_tf32=True):
            yield
    finally:
        torch.backends.cuda.matmul.allow_tf32 = old_allow_tf32_matmul
        self.precision = old_precision

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `tf32_on`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`tf32_on`。

### Lines 286-299
```python

@contextlib.contextmanager
def tf32_enabled():
    """
    Context manager to temporarily enable TF32 for CUDA operations.
    Restores the previous TF32 state after exiting the context.
    """
    old_allow_tf32_matmul = torch.backends.cuda.matmul.allow_tf32
    try:
        torch.backends.cuda.matmul.allow_tf32 = True
        with torch.backends.cudnn.flags(
            enabled=None, benchmark=None, deterministic=None, allow_tf32=True
        ):
            yield
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `tf32_enabled`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`tf32_enabled`。

### Lines 300-313
```python
    finally:
        torch.backends.cuda.matmul.allow_tf32 = old_allow_tf32_matmul


# This is a wrapper that wraps a test to run this test twice, one with
# allow_tf32=True, another with allow_tf32=False. When running with
# allow_tf32=True, it will use reduced precision as specified by the
# argument. For example:
#    @dtypes(torch.float32, torch.float64, torch.complex64, torch.complex128)
#    @tf32_on_and_off(0.005)
#    def test_matmul(self, device, dtype):
#        a = ...; b = ...;
#        c = torch.matmul(a, b)
#        self.assertEqual(c, expected)
```
- EN: This block checks invariants or expected outcomes; handles tensor metadata or sample values; bridges to backend-specific execution artifacts; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；处理张量元数据或示例值；桥接到特定后端的执行产物；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 314-327
```python
# In the above example, when testing torch.float32 and torch.complex64 on CUDA
# on a CUDA >= 11 build on an >=Ampere architecture, the matmul will be running at
# TF32 mode and TF32 mode off, and on TF32 mode, the assertEqual will use reduced
# precision to check values.
#
# This decorator can be used for function with or without device/dtype, such as
# @tf32_on_and_off(0.005)
# def test_my_op(self)
# @tf32_on_and_off(0.005)
# def test_my_op(self, device)
# @tf32_on_and_off(0.005)
# def test_my_op(self, device, dtype)
# @tf32_on_and_off(0.005)
# def test_my_op(self, dtype)
```
- EN: Documents intent, assumptions, or compatibility notes that guide the surrounding implementation.
- CN: 说明周围实现所遵循的目标、假设或兼容性约束。

### Lines 328-336
```python
# if neither device nor dtype is specified, it will check if the system has ampere device
# if device is specified, it will check if device is cuda
# if dtype is specified, it will check if dtype is float32 or complex64
# tf32 and fp32 are different only when all the three checks pass
def tf32_on_and_off(tf32_precision=1e-5, *, only_if=True):
    def with_tf32_disabled(self, function_call):
        with tf32_off():
            function_call()

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `tf32_on_and_off`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`tf32_on_and_off`。

### Lines 337-344
```python
    def with_tf32_enabled(self, function_call):
        with tf32_on(self, tf32_precision):
            function_call()

    def wrapper(f):
        params = inspect.signature(f).parameters
        arg_names = tuple(params.keys())

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `with_tf32_enabled`, `wrapper`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`with_tf32_enabled`, `wrapper`。

### Lines 345-351
```python
        @functools.wraps(f)
        def wrapped(*args, **kwargs):
            kwargs.update(zip(arg_names, args, strict=False))
            cond = torch.cuda.is_tf32_supported() and only_if
            if 'device' in kwargs:
                cond = cond and (torch.device(kwargs['device']).type == 'cuda')
            if 'dtype' in kwargs:
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `wrapped`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`wrapped`。

### Lines 352-358
```python
                cond = cond and (kwargs['dtype'] in {torch.float32, torch.complex64})
            if cond:
                with_tf32_disabled(kwargs['self'], lambda: f(**kwargs))
                with_tf32_enabled(kwargs['self'], lambda: f(**kwargs))
            else:
                f(**kwargs)

```
- EN: This block handles tensor metadata or sample values; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；处理条件控制流。关键符号：无明显局部符号。

### Lines 359-367
```python
        return wrapped
    return wrapper

# This is a wrapper that wraps a test to run it with TF32 turned off.
# This wrapper is designed to be used when a test uses matmul or convolutions
# but the purpose of that test is not testing matmul or convolutions.
# Disabling TF32 will enforce torch.float tensors to be always computed
# at full precision.
def with_tf32_off(f):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `with_tf32_off`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`with_tf32_off`。

### Lines 368-374
```python
    @functools.wraps(f)
    def wrapped(*args, **kwargs):
        with tf32_off():
            return f(*args, **kwargs)

    return wrapped

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `wrapped`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`wrapped`。

### Lines 375-381
```python
def _get_magma_version():
    if 'Magma' not in torch.__config__.show():
        return (0, 0)
    position = torch.__config__.show().find('Magma ')
    version_str = torch.__config__.show()[position + len('Magma '):].split('\n')[0]
    return tuple(int(x) for x in version_str.split("."))

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_get_magma_version`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_get_magma_version`。

### Lines 382-388
```python
def _get_torch_cuda_version():
    if torch.version.cuda is None:
        return (0, 0)
    cuda_version = str(torch.version.cuda)
    return tuple(int(x) for x in cuda_version.split("."))

def _get_torch_rocm_version():
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_get_torch_cuda_version`, `_get_torch_rocm_version`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_get_torch_cuda_version`, `_get_torch_rocm_version`。

### Lines 389-395
```python
    if not TEST_WITH_ROCM or torch.version.hip is None:
        return (0, 0)
    rocm_version = str(torch.version.hip)
    rocm_version = rocm_version.split("-", maxsplit=1)[0]    # ignore git sha
    return tuple(int(x) for x in rocm_version.split("."))

def _get_torch_hipblaslt_version():
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_get_torch_hipblaslt_version`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_get_torch_hipblaslt_version`。

### Lines 396-402
```python
    if not TEST_WITH_ROCM:
        return None
    try:
        # Access through direct C binding
        # versionHipBLASLt returns: MAJOR * 10000 + MINOR * 100 + PATCH
        version_int = torch._C._cuda_getHipblasltVersion()
        if version_int is None or version_int == 0:
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 403-409
```python
            return None
        major = version_int // 10000
        minor = (version_int % 10000) // 100
        patch = version_int % 100
        return (major, minor, patch)
    except (AttributeError, RuntimeError):
        return None
```
- EN: This block reports or normalizes error conditions; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块报告或规范化错误情况；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 410-416
```python

def _check_cusparse_generic_available():
    return not TEST_WITH_ROCM

def _check_hipsparse_generic_available():
    if not TEST_WITH_ROCM:
        return False
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_check_cusparse_generic_available`, `_check_hipsparse_generic_available`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_check_cusparse_generic_available`, `_check_hipsparse_generic_available`。

### Lines 417-423
```python
    if not torch.version.hip:
        return False

    rocm_version = str(torch.version.hip)
    rocm_version = rocm_version.split("-", maxsplit=1)[0]    # ignore git sha
    rocm_version_tuple = tuple(int(x) for x in rocm_version.split("."))
    return not (rocm_version_tuple is None or rocm_version_tuple < (5, 1))
```
- EN: This block handles conditional control flow; iterates over collections or execution units; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；遍历集合或执行单元；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 424-430
```python


TEST_CUSPARSE_GENERIC = _check_cusparse_generic_available()
TEST_HIPSPARSE_GENERIC = _check_hipsparse_generic_available()

# Shared by test_torch.py and test_multigpu.py
def _create_scaling_models_optimizers(device="cuda", optimizer_ctor=torch.optim.SGD, optimizer_kwargs=None):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_create_scaling_models_optimizers`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_create_scaling_models_optimizers`。

### Lines 431-438
```python
    # Create a module+optimizer that will use scaling, and a control module+optimizer
    # that will not use scaling, against which the scaling-enabled module+optimizer can be compared.
    mod_control = torch.nn.Sequential(torch.nn.Linear(8, 8), torch.nn.Linear(8, 8)).to(device=device)
    mod_scaling = torch.nn.Sequential(torch.nn.Linear(8, 8), torch.nn.Linear(8, 8)).to(device=device)
    with torch.no_grad():
        for c, s in zip(mod_control.parameters(), mod_scaling.parameters(), strict=True):
            s.copy_(c)

```
- EN: This block handles tensor metadata or sample values; iterates over collections or execution units. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；遍历集合或执行单元。关键符号：无明显局部符号。

### Lines 439-445
```python
    kwargs = {"lr": 1.0}
    if optimizer_kwargs is not None:
        kwargs.update(optimizer_kwargs)
    opt_control = optimizer_ctor(mod_control.parameters(), **kwargs)
    opt_scaling = optimizer_ctor(mod_scaling.parameters(), **kwargs)

    return mod_control, mod_scaling, opt_control, opt_scaling
```
- EN: This block handles conditional control flow; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理条件控制流；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 446-453
```python

# Shared by test_torch.py, test_cuda.py and test_multigpu.py
def _create_scaling_case(device="cuda", dtype=torch.float, optimizer_ctor=torch.optim.SGD, optimizer_kwargs=None):
    data = [(torch.randn((8, 8), dtype=dtype, device=device), torch.randn((8, 8), dtype=dtype, device=device)),
            (torch.randn((8, 8), dtype=dtype, device=device), torch.randn((8, 8), dtype=dtype, device=device)),
            (torch.randn((8, 8), dtype=dtype, device=device), torch.randn((8, 8), dtype=dtype, device=device)),
            (torch.randn((8, 8), dtype=dtype, device=device), torch.randn((8, 8), dtype=dtype, device=device))]

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `_create_scaling_case`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`_create_scaling_case`。

### Lines 454-461
```python
    loss_fn = torch.nn.MSELoss().to(device)

    skip_iter = 2

    return _create_scaling_models_optimizers(
        device=device, optimizer_ctor=optimizer_ctor, optimizer_kwargs=optimizer_kwargs,
    ) + (data, loss_fn, skip_iter)

```
- EN: This block handles tensor metadata or sample values; returns results to callers or downstream stages. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；向调用方或后续阶段返回结果。关键符号：无明显局部符号。

### Lines 462-468
```python

def xfailIfSM89(func):
    return func if not IS_SM89 else unittest.expectedFailure(func)

def xfailIfSM90(func):
    return func if not IS_SM90 else unittest.expectedFailure(func)

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `xfailIfSM89`, `xfailIfSM90`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`xfailIfSM89`, `xfailIfSM90`。

### Lines 469-475
```python
def xfailIfSM89PreCUDA13(func):
    """xfail on SM89 only for CUDA < 13. On CUDA 13+, test should pass on all architectures."""
    if IS_SM89 and _get_torch_cuda_version() < (13, 0):
        return unittest.expectedFailure(func)
    return func

def xfailIfSM100OrLater(func):
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `xfailIfSM89PreCUDA13`, `xfailIfSM100OrLater`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`xfailIfSM89PreCUDA13`, `xfailIfSM100OrLater`。

### Lines 476-482
```python
    return func if not SM100OrLater else unittest.expectedFailure(func)

def xfailIfSM120OrLater(func):
    return func if not SM120OrLater else unittest.expectedFailure(func)

def xfailIfSM12X(func):
    return func if not IS_SM12X else unittest.expectedFailure(func)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `xfailIfSM120OrLater`, `xfailIfSM12X`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`xfailIfSM120OrLater`, `xfailIfSM12X`。

### Lines 483-493
```python

def xfailIfDistributedNotSupported(func):
    return func if not (IS_MACOS or IS_JETSON) else unittest.expectedFailure(func)

# When using nvcc from the CUDA toolkit its versuib must be at least the one from ptxas bundled with Triton
TRITON_PTXAS_VERSION = (12, 8)
requires_triton_ptxas_compat = unittest.skipIf(not torch.version.xpu
                                               and torch.version.hip is None
                                               and _get_torch_cuda_version() < TRITON_PTXAS_VERSION,
                                               "Requires CUDA {}.{} to match Tritons ptxas version".format(*TRITON_PTXAS_VERSION))

```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `xfailIfDistributedNotSupported`, `TRITON_PTXAS_VERSION`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`xfailIfDistributedNotSupported`, `TRITON_PTXAS_VERSION`。

### Lines 494-497
```python
# Importing this module should NOT eagerly initialize CUDA
if not CUDA_ALREADY_INITIALIZED_ON_IMPORT:
    if torch.cuda.is_initialized():
        raise AssertionError("CUDA should not be initialized on import")
```
- EN: This block checks invariants or expected outcomes; reports or normalizes error conditions; handles conditional control flow. Key symbols: no dominant local symbols.
- CN: 该代码块检查不变量或预期结果；报告或规范化错误情况；处理条件控制流。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- EN: Testing assertions — the file focuses on checking values, errors, or textual expectations.
  CN: 测试断言——该文件重点检查数值、异常或文本期望是否满足要求。
- EN: Tensor construction and metadata — the code manages sample tensors, dtypes, layouts, or device placement.
  CN: 张量构造与元数据——代码会管理示例张量、数据类型、布局或设备放置。
- EN: Reusable test infrastructure — the module provides shared helpers, fixtures, or data for many test suites.
  CN: 可复用测试基础设施——该模块为多个测试套件提供共享辅助函数、夹具或数据。
- EN: Distributed testing — the implementation coordinates multi-process or sharded training checks.
  CN: 分布式测试——该实现会协调多进程或分片训练相关的检查逻辑。
- EN: Backend coverage — the file contains backend-specific behavior or validation paths.
  CN: 后端覆盖——该文件包含特定后端的行为或校验路径。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.cuda`, `torch.testing._internal.common_utils`
- External imports / 外部导入: `functools`, `inspect`, `contextlib`, `os`, `unittest`, `numba.cuda`
- Representative symbols / 代表性符号: `CUDA_ALREADY_INITIALIZED_ON_IMPORT`, `TEST_MULTIGPU`, `CUDA_DEVICE`, `TEST_CUDNN_VERSION`, `ROCM_VERSION`, `IS_THOR`, `IS_JETSON`, `IS_SM89`, `IS_SM90`, `IS_SM100`, `...`
