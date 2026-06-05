# device_info.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/analysis/device_info.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module analyzes compiler, runtime, or profiling state for TorchInductor. It defines classes such as `DeviceInfo`. It exposes functions such as `lookup_device_info`, and `datasheet_tops`.
- **用途（中文）**: 该模块分析 TorchInductor 的编译、运行期或性能画像状态。其中定义了 `DeviceInfo` 等类。同时提供 `lookup_device_info`、`datasheet_tops` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行
````python
import logging
from dataclasses import dataclass

import torch


log = logging.getLogger(__name__)


@dataclass(frozen=True)
class DeviceInfo:
    """
    Theoretical Numbers from data sheet. If two numbers are given, Tensor/Matrix Core vs not,
    then the higher number is reported. Sparsity is not considered.
````
- **EN**: Imports dependencies such as `logging`, `dataclasses`, and `torch` for the logic in this range. Introduces class `DeviceInfo`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields.
- **CN**: 这里导入了 `logging`、`dataclasses`、`torch` 等依赖，为后续逻辑提供基础能力。这里定义了类`DeviceInfo`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。

### Lines 15-28 / 第 15-28 行
````python


    Bandwidth numbers are tricky, because there are platform differences that may not show up in the profiler trace.
    For example,
    """

    tops: dict[torch.dtype | str, float]
    dram_bw_gbs: float
    dram_gb: float


# Indexing is based on `torch.cuda.get_device_name()`
# TODO investigate profiler support for tf32 and allow device to report correct number when it's turned on.
_device_mapping: dict[str, DeviceInfo] = {
````
- **EN**: Processes profiling, FLOP, or benchmark information to characterize execution cost. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 处理性能画像、FLOP 或基准测试信息，用于刻画执行代价。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 29-42 / 第 29-42 行
````python
    # Source:
    # @lint-ignore https://www.nvidia.com/en-us/data-center/h100/
    "NVIDIA H100": DeviceInfo(
        tops={
            torch.float64: 67.0,
            torch.float32: 67.5,
            "torch.tf32": 156.0,
            torch.bfloat16: 1979.0,
            torch.float16: 1979.0,
            torch.float8_e8m0fnu: 3958.0,
            torch.float8_e8m0fnu: 3958.0,
            torch.float8_e4m3fnuz: 3958.0,
            torch.float8_e5m2: 3958.0,
            torch.float8_e5m2fnuz: 3958.0,
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `tops`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `tops` 等值。

### Lines 43-56 / 第 43-56 行
````python
            torch.float8_e8m0fnu: 3958.0,
            torch.int8: 3958.0,
        },
        dram_bw_gbs=3350,
        dram_gb=80,
    ),
    # Source:
    # @lint-ignore https://www.nvidia.com/content/dam/en-zz/Solutions/Data-Center/a100/pdf/
    # nvidia-a100-datasheet-us-nvidia-1758950-r4-web.pdf
    "NVIDIA A100": DeviceInfo(
        tops={
            torch.float64: 19.5,
            torch.float32: 19.5,
            torch.bfloat16: 312.5,
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `dram_bw_gbs`, `dram_gb`, and `tops`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `dram_bw_gbs`、`dram_gb`、`tops` 等值。

### Lines 57-70 / 第 57-70 行
````python
            torch.float16: 312.5,
            # Not in datasheet: float8
            torch.int8: 624.0,
            "torch.tf32": 156.0,
        },
        dram_bw_gbs=2039.0,
        dram_gb=80.0,
    ),
    # Source:
    # @lint-ignore https://resources.nvidia.com/en-us-gpu-resources/l4-tensor-datasheet
    "NVIDIA L4": DeviceInfo(
        tops={
            # This is a guess, not in datasheet
            torch.float64: 15.1,
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `dram_bw_gbs`, `dram_gb`, and `tops`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `dram_bw_gbs`、`dram_gb`、`tops` 等值。

### Lines 71-84 / 第 71-84 行
````python
            torch.float32: 30.3,
            "torch.tf32": 120.0,
            torch.bfloat16: 242.0,
            torch.float16: 242.0,
            torch.float8_e8m0fnu: 485.0,
            torch.float8_e8m0fnu: 485.0,
            torch.float8_e4m3fnuz: 485.0,
            torch.float8_e5m2: 485.0,
            torch.float8_e5m2fnuz: 485.0,
            torch.float8_e8m0fnu: 485.0,
            torch.int8: 485.0,
        },
        dram_bw_gbs=3350,
        dram_gb=24,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `dram_bw_gbs`, and `dram_gb`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `dram_bw_gbs`、`dram_gb` 等值。

### Lines 85-98 / 第 85-98 行
````python
    ),
    # Source:
    # @lint-ignore https://www.amd.com/content/dam/amd/en/documents\
    # /instinct-tech-docs/product-briefs/amd-instinct-mi350x-gpu-brochure.pdf
    "AMD MI350X": DeviceInfo(
        tops={
            torch.float64: 72.1,
            torch.float32: 144.2,
            # not specified, fall back to float32 numbers
            "torch.tf32": 144.2,
            torch.bfloat16: 2309.6,
            torch.float16: 2309.6,
            torch.float8_e8m0fnu: 4614.0,
            torch.float8_e8m0fnu: 4614.0,
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `tops`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `tops` 等值。

### Lines 99-112 / 第 99-112 行
````python
            torch.float8_e4m3fnuz: 4614.0,
            torch.float8_e5m2: 4614.0,
            torch.float8_e5m2fnuz: 4614.0,
            torch.float8_e8m0fnu: 4614.0,
            torch.int8: 4614.0,
        },
        dram_bw_gbs=8000.0,
        dram_gb=288.0,
    ),
    # Source:
    # @lint-ignore https://www.amd.com/content/dam/amd/en/documents\
    # /instinct-tech-docs/data-sheets/amd-instinct-mi300a-data-sheet.pdf
    "AMD MI300A": DeviceInfo(
        tops={
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `dram_bw_gbs`, `dram_gb`, and `tops`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `dram_bw_gbs`、`dram_gb`、`tops` 等值。

### Lines 113-126 / 第 113-126 行
````python
            torch.float64: 122.6,
            torch.float32: 122.6,
            "torch.tf32": 490.3,
            torch.bfloat16: 980.6,
            torch.float16: 980.6,
            torch.float8_e8m0fnu: 1961.2,
            torch.float8_e8m0fnu: 1961.2,
            torch.float8_e4m3fnuz: 1961.2,
            torch.float8_e5m2: 1961.2,
            torch.float8_e5m2fnuz: 1961.2,
            torch.float8_e8m0fnu: 1961.2,
            torch.int8: 1961.2,
        },
        dram_bw_gbs=5300.0,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `dram_bw_gbs`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `dram_bw_gbs` 等值。

### Lines 127-140 / 第 127-140 行
````python
        dram_gb=128.0,
    ),
    # Source:
    # @lint-ignore https://www.amd.com/content/dam/amd/en/documents/\
    # instinct-tech-docs/data-sheets/amd-instinct-mi300x-data-sheet.pdf
    "AMD MI300X": DeviceInfo(
        tops={
            torch.float64: 163.4,
            torch.float32: 163.4,
            "torch.tf32": 653.7,
            torch.bfloat16: 1307.4,
            torch.float16: 1307.4,
            torch.float8_e8m0fnu: 2614.9,
            torch.float8_e8m0fnu: 2614.9,
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `dram_gb`, and `tops`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `dram_gb`、`tops` 等值。

### Lines 141-154 / 第 141-154 行
````python
            torch.float8_e4m3fnuz: 2614.9,
            torch.float8_e5m2: 2614.9,
            torch.float8_e5m2fnuz: 2614.9,
            torch.float8_e8m0fnu: 2614.9,
            torch.int8: 2614.9,
        },
        dram_bw_gbs=5300.0,
        dram_gb=192.0,
    ),
    # Source:
    # @lint-ignore https://www.amd.com/content/dam/amd/\
    # en/documents/instinct-business-docs/product-briefs/instinct-mi210-brochure.pdf
    "AMD MI210X": DeviceInfo(
        tops={
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `dram_bw_gbs`, `dram_gb`, and `tops`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `dram_bw_gbs`、`dram_gb`、`tops` 等值。

### Lines 155-168 / 第 155-168 行
````python
            torch.float64: 45.3,
            torch.float32: 45.3,
            # not specified, fall back to float32 numbers
            "torch.tf32": 45.3,
            torch.bfloat16: 181.0,
            torch.float16: 181.0,
            # not specified, fall back to float16 numbers
            torch.float8_e8m0fnu: 181.0,
            torch.float8_e8m0fnu: 181.0,
            torch.float8_e4m3fnuz: 181.0,
            torch.float8_e5m2: 181.0,
            torch.float8_e5m2fnuz: 181.0,
            torch.float8_e8m0fnu: 181.0,
            torch.int8: 181.0,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 169-182 / 第 169-182 行
````python
        },
        # pcie4.0x16
        dram_bw_gbs=1600.0,
        dram_gb=64.0,
    ),
}
_device_mapping["AMD INSTINCT MI350X"] = _device_mapping["AMD MI350X"]
_device_mapping["AMD INSTINCT MI300X"] = _device_mapping["AMD MI300X"]
_device_mapping["AMD INSTINCT MI210X"] = _device_mapping["AMD MI210X"]


def lookup_device_info(name: str) -> DeviceInfo | None:
    """
    Problem: when diffing profiles between amd and nvidia, we don't have access to the device information
````
- **EN**: Introduces function `lookup_device_info`. Handles device-specific behavior, capabilities, or dispatch decisions. Builds or updates a registry/mapping so later code can dispatch by name or capability. Initializes or updates values such as `dram_bw_gbs`, `dram_gb`, and `Problem`.
- **CN**: 这里定义了函数`lookup_device_info`。处理与设备相关的行为、能力信息或派发决策。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。初始化或更新了 `dram_bw_gbs`、`dram_gb`、`Problem` 等值。

### Lines 183-196 / 第 183-196 行
````python
    of the other one. Also, since the analysis is static, we should be able to do it on another device unrelated
    to the recorded device. Therefore, _device_mapping statically contains the information for lots of devices.
    If one is missing, please run DeviceInfo.get_device_info() and add it to _device_mapping.
      name (str): name of the device to lookup. Should map onto torch.cuda.get_device_name().
    """
    return _device_mapping.get(name)


def datasheet_tops(dtype: torch.dtype, is_tf32: bool = False) -> float | None:
    """
    Get the theoretical TFLOPS of the device for a given dtype. This can throw an exception if the device
    is not in the datasheet list above.
    """
    name: str | None = torch.cuda.get_device_name()
````
- **EN**: Introduces function `datasheet_tops`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`datasheet_tops`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 197-210 / 第 197-210 行
````python
    if name is None:
        log.info("No device found, returning None")
        return None
    device_info = lookup_device_info(name)
    if device_info is None:
        log_str = f"Device {name} not in datasheet, returning None"
        log.info(log_str)
        return None
    if dtype not in device_info.tops:
        log.info(
            "Device %s does not have a datasheet entry for %s, returning None",
            name,
            dtype,
        )
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 211-215 / 第 211-215 行
````python
        return None

    return device_info.tops[
        "torch.tf32" if dtype == torch.float32 and is_tf32 else dtype
    ]
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. This range continues the implementation of function `datasheet_tops`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`datasheet_tops` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Analyzes compiler, runtime, or profiling state for TorchInductor  
  **CN**: 分析 TorchInductor 的编译、运行期或性能画像状态
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Profiling data handling  
  **CN**: 性能画像数据处理
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary classes: `DeviceInfo`  
  **CN**: 主要类：`DeviceInfo`
- **EN**: Primary functions: `lookup_device_info`, and `datasheet_tops`  
  **CN**: 主要函数：`lookup_device_info`、`datasheet_tops`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `dataclasses`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`
