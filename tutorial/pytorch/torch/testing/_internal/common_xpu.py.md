# common_xpu.py — Code Analysis / 代码分析

## Source / 来源
- File: `torch/testing/_internal/common_xpu.py`
- Repository: `pytorch/pytorch`
- Purpose (EN): Implements internal PyTorch testing infrastructure for common xpu, including reusable fixtures, comparison helpers, and backend-specific test support.
- 用途 (CN): 实现与 common xpu 相关的 PyTorch 内部测试基础设施，包括可复用夹具、比较辅助函数以及特定后端的测试支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5
```python
import enum
import functools

import torch
import torch.xpu
```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch`, `torch.xpu`; external imports: `enum`, `functools`.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch`, `torch.xpu`；外部导入：`enum`, `functools`。

### Lines 6-10
```python
from torch.testing._internal.common_utils import IS_WINDOWS, LazyVal, TEST_XPU


XPU_ALREADY_INITIALIZED_ON_IMPORT = torch.xpu.is_initialized()

```
- EN: Imports Python modules needed by this part of the subsystem. Internal imports: `torch.testing._internal.common_utils`; external imports: none.
- CN: 引入该子系统这一部分所需的 Python 模块。内部导入：`torch.testing._internal.common_utils`；外部导入：无。

### Lines 11-15
```python

class XPUCodename(enum.Enum):
    PVC = "PVC"  # Intel® Data Center GPU Max Series
    BMG = "BMG"  # Intel® Arc™ Pro Battlemage Graphics

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `XPUCodename`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`XPUCodename`。

### Lines 16-21
```python

class XPUArch(enum.IntEnum):
    Unknown = 0
    Xe = 1  # Xe HPC
    Xe2 = 2

```
- EN: Defines a class or structured abstraction used by the file. Representative symbols: `XPUArch`.
- CN: 定义该文件使用的类或结构化抽象。代表性符号：`XPUArch`。

### Lines 22-31
```python

# device_id -> GPU codename
# From https://github.com/intel/intel-graphics-compiler/blob/master/inc/common/igfxfmid.h
_DEVICE_ID_TO_CODENAME = {
    0x0BD0: XPUCodename.PVC,
    0x0BD4: XPUCodename.PVC,
    0x0BD5: XPUCodename.PVC,
    0x0BD6: XPUCodename.PVC,
    0x0BD7: XPUCodename.PVC,
    0x0BD8: XPUCodename.PVC,
```
- EN: This block handles tensor metadata or sample values; manipulates graph-like program structures. Key symbols: no dominant local symbols.
- CN: 该代码块处理张量元数据或示例值；操作图状程序结构。关键符号：无明显局部符号。

### Lines 32-41
```python
    0x0BD9: XPUCodename.PVC,
    0x0BDA: XPUCodename.PVC,
    0x0BDB: XPUCodename.PVC,
    0x0B69: XPUCodename.PVC,
    0x0B6E: XPUCodename.PVC,
    0xE202: XPUCodename.BMG,
    0xE20B: XPUCodename.BMG,
    0xE20C: XPUCodename.BMG,
    0xE20D: XPUCodename.BMG,
    0xE210: XPUCodename.BMG,
```
- EN: This block implements local helper logic for common xpu. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common xpu 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 42-50
```python
    0xE212: XPUCodename.BMG,
    0xE215: XPUCodename.BMG,
    0xE216: XPUCodename.BMG,
    0xE220: XPUCodename.BMG,
    0xE221: XPUCodename.BMG,
    0xE222: XPUCodename.BMG,
    0xE223: XPUCodename.BMG,
}

```
- EN: This block implements local helper logic for common xpu. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common xpu 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 51-56
```python
# GPU codename -> architecture
_CODENAME_TO_ARCH = {
    XPUCodename.PVC: XPUArch.Xe,
    XPUCodename.BMG: XPUArch.Xe2,
}

```
- EN: This block implements local helper logic for common xpu. Key symbols: `_CODENAME_TO_ARCH`.
- CN: 该代码块实现与 common xpu 相关的局部辅助逻辑。关键符号：`_CODENAME_TO_ARCH`。

### Lines 57-61
```python

@functools.lru_cache(1)
def get_xpu_codename() -> XPUCodename | None:
    device_id = torch.xpu.get_device_capability()["device_id"]
    return _DEVICE_ID_TO_CODENAME.get(device_id)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_xpu_codename`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_xpu_codename`。

### Lines 62-67
```python


@functools.lru_cache(1)
def get_xpu_arch() -> XPUArch | None:
    codename = get_xpu_codename()
    return _CODENAME_TO_ARCH.get(codename, XPUArch.Unknown)
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `get_xpu_arch`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`get_xpu_arch`。

### Lines 68-73
```python


Xe2_Or_Later = LazyVal(
    lambda: torch.xpu.is_available() and get_xpu_arch() >= XPUArch.Xe2
)

```
- EN: This block implements local helper logic for common xpu. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common xpu 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 74-78
```python

def evaluate_platform_supports_flash_attention():
    if TEST_XPU:
        return not IS_WINDOWS and Xe2_Or_Later
    return False
```
- EN: Defines Python callable logic for this module, including API behavior or helper routines. Representative symbols: `evaluate_platform_supports_flash_attention`.
- CN: 定义该模块的 Python 可调用逻辑，包括 API 行为或辅助例程。代表性符号：`evaluate_platform_supports_flash_attention`。

### Lines 79-84
```python


PLATFORM_SUPPORTS_FLASH_ATTENTION_XPU: bool = LazyVal(
    lambda: evaluate_platform_supports_flash_attention()
)

```
- EN: This block implements local helper logic for common xpu. Key symbols: no dominant local symbols.
- CN: 该代码块实现与 common xpu 相关的局部辅助逻辑。关键符号：无明显局部符号。

### Lines 85-88
```python
# Importing this module should NOT eagerly initialize XPU
if not XPU_ALREADY_INITIALIZED_ON_IMPORT:
    if torch.xpu.is_initialized():
        raise AssertionError("XPU should not be initialized on import")
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
- EN: Backend coverage — the file contains backend-specific behavior or validation paths.
  CN: 后端覆盖——该文件包含特定后端的行为或校验路径。
- EN: Graph capture and transformation — the implementation works with program graphs and rewrite passes.
  CN: 图捕获与变换——实现会处理程序图及其改写 pass。
- EN: Test maintainability — the file helps standardize expectations, fixtures, or generated coverage across PyTorch tests.
  CN: 测试可维护性——该文件有助于在 PyTorch 测试中统一期望、夹具或生成式覆盖。

## Dependencies / 依赖关系
- Internal imports / 内部导入: `torch`, `torch.xpu`, `torch.testing._internal.common_utils`
- External imports / 外部导入: `enum`, `functools`
- Representative symbols / 代表性符号: `XPU_ALREADY_INITIALIZED_ON_IMPORT`, `XPUCodename`, `XPUArch`, `_DEVICE_ID_TO_CODENAME`, `_CODENAME_TO_ARCH`, `get_xpu_codename`, `get_xpu_arch`, `evaluate_platform_supports_flash_attention`
