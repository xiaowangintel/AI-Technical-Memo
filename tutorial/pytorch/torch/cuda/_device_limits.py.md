# _device_limits.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/cuda/_device_limits.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements CUDA-facing Python wrappers for streams, memory, random state, and runtime management.
- **Purpose (CN)**: 实现面向 CUDA 的 Python 包装层，用于流、内存、随机状态以及运行时管理。
## Line-by-Line Analysis / 逐行分析

### Lines 1-13 / 第 1-13 行
````python
import torch
from torch._C import dtype


__all__ = ["GPULimits"]


class GPULimits:
    r"""Utility class that provides the theoretical limits of Nvidia GPU devices. The
    limits don't take into account thermal throttling (assume that the GPU run at its
    peak rated frequency). This is because user hardware configuration may influence
    power behavior.
    """
````
- **EN**: This block assembles module dependencies, pulling in internal torch modules such as torch, torch._C. `__all__` defines the public symbols that this module chooses to export. It introduces or extends `GPULimits`, which hold the main object-oriented state for this portion of the file.
- **CN**: 这一段组织模块依赖，引入了内部 torch 模块，如 torch、torch._C。 `__all__` 定义了本模块选择导出的公共符号。 它引入或扩展了 `GPULimits`，这些类承载了本段涉及的主要面向对象状态。

### Lines 15-27 / 第 15-27 行
````python
    def __init__(self, target_device: torch.device):
        # The device properties object is obtained by calling 'cudaGetDeviceProperties' CUDA
        # runtime function. We need the total memory bus width and the memory clock rate to
        # calculate the memory bandwidth.
        self.device_properties = torch.cuda.get_device_properties(target_device)

        # The compute capability is needed to determine the number of FLOPs per cycle per SM
        self.compute_capability = int(
            f"{self.device_properties.major}{self.device_properties.minor}"
        )

    # FLOPs per cycle information derived from Table 2 in:
    # https://resources.nvidia.com/en-us-hopper-architecture/nvidia-h100-tensor-c
````
- **EN**: This chunk defines `__init__`, which initializes object state and binds construction-time configuration.
- **CN**: 这一段定义了 `__init__`，其作用是初始化对象状态并绑定构造期配置。

### Lines 29-42 / 第 29-42 行
````python
    # Returns the number of FMA instructions retired per cycle per SM for a given
    # data type, when tensor cores are NOT used
    def get_fma_per_cycle_per_sm_cuda_cores(self, data_type: dtype) -> int:
        hardcoded_device_values = {
            # Ampere Architecture
            "fp16_80": 256,
            "fp32_80": 64,
            "fp64_80": 32,
            # Hopper Architecture
            "fp16_90": 64,
            "fp32_90": 128,
            "fp64_90": 64,
            # Blackwell Architecture
            "fp16_100": 256,
````
- **EN**: This chunk defines `get_fma_per_cycle_per_sm_cuda_cores`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries.
- **CN**: 这一段定义了 `get_fma_per_cycle_per_sm_cuda_cores`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。

### Lines 43-54 / 第 43-54 行
````python
            "fp32_100": 128,
            "fp64_100": 64,
        }
        dict_key = ""
        if data_type is torch.float16:
            dict_key = f"fp16_{self.compute_capability}"
        elif data_type is torch.float32:
            dict_key = f"fp32_{self.compute_capability}"
        elif data_type is torch.float64:
            dict_key = f"fp64_{self.compute_capability}"
        else:
            dict_key = "unknown"
````
- **EN**: This chunk continues `get_fma_per_cycle_per_sm_cuda_cores` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `get_fma_per_cycle_per_sm_cuda_cores`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 56-69 / 第 56-69 行
````python
        if dict_key not in hardcoded_device_values:
            raise RuntimeError(
                f"No data for sm_{self.compute_capability} and {data_type}."
            )

        return hardcoded_device_values[dict_key]

    # Returns the number of FMA instructions retired per cycle per SM for a given
    # data type, when tensor cores ARE used
    def get_fma_per_cycle_per_sm_tensor_cores(self, data_type: dtype) -> int:
        hardcoded_device_values = {
            # Ampere Architecture
            "int8_80": 2048,
            "fp16_80": 1024,
````
- **EN**: This chunk defines `get_fma_per_cycle_per_sm_tensor_cores`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_fma_per_cycle_per_sm_tensor_cores`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 70-83 / 第 70-83 行
````python
            "fp32_80": 512,
            "fp64_80": 64,
            # Hopper Architecture
            "int8_90": 4096,
            "fp8_90": 4096,
            "fp16_90": 2048,
            "fp32_90": 1024,
            "fp64_90": 128,
            # Blackwell Architecture
            "int8_100": 8192,
            "fp8_100": 8192,
            "fp16_100": 4096,
            "fp32_100": 2048,
        }
````
- **EN**: This chunk continues `get_fma_per_cycle_per_sm_tensor_cores` and expands its internal control flow or state updates.
- **CN**: 这一段延续了 `get_fma_per_cycle_per_sm_tensor_cores`，进一步展开其内部控制流或状态更新。

### Lines 84-97 / 第 84-97 行
````python
        dict_key = ""
        if data_type is torch.float16:
            dict_key = f"fp16_{self.compute_capability}"
        elif data_type is torch.bfloat16:
            # FP16 and BF16 are equivalent in terms of FLOPs per cycle per SM
            dict_key = f"fp16_{self.compute_capability}"
        elif data_type is torch.float32:
            dict_key = f"fp32_{self.compute_capability}"
        elif data_type is torch.int8:
            dict_key = f"int8_{self.compute_capability}"
        elif data_type is torch.float64:
            dict_key = f"fp64_{self.compute_capability}"
        else:
            dict_key = "unknown"
````
- **EN**: This chunk continues `get_fma_per_cycle_per_sm_tensor_cores` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants.
- **CN**: 这一段延续了 `get_fma_per_cycle_per_sm_tensor_cores`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。

### Lines 99-110 / 第 99-110 行
````python
        if dict_key not in hardcoded_device_values:
            raise RuntimeError(
                f"No data for sm_{self.compute_capability} and {data_type}."
            )

        return hardcoded_device_values[dict_key]

    def get_tflops_per_second(
        self, data_type: dtype, use_tensor_cores: bool = True
    ) -> float:
        num_sms = self.device_properties.multi_processor_count
        clock_rate = self.device_properties.clock_rate  # KHz
````
- **EN**: This chunk defines `get_tflops_per_second`, which retrieves runtime state and exposes it through a Python-friendly accessor. Looping logic applies the same validation or transformation across tensors, parameters, or registry entries. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. Assertions or explicit exceptions make unsupported states fail early instead of silently producing bad results. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_tflops_per_second`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 循环逻辑会在张量、参数或注册表条目上重复应用相同的校验或变换。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 断言或显式异常会让不支持的状态尽早失败，而不是悄悄产生错误结果。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 112-121 / 第 112-121 行
````python
        fma_per_cycle = 0
        if use_tensor_cores:
            fma_per_cycle = self.get_fma_per_cycle_per_sm_tensor_cores(data_type)
        else:
            fma_per_cycle = self.get_fma_per_cycle_per_sm_cuda_cores(data_type)

        # 1 FMA counts as 2 floating point operations
        # Clock rate is in KHz
        tflops_per_second = num_sms * fma_per_cycle * 2 * clock_rate / 1e9
        return tflops_per_second
````
- **EN**: This chunk continues `get_tflops_per_second` and expands its internal control flow or state updates. Conditional branches separate fast paths, backend-specific cases, and user-facing invariants. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `get_tflops_per_second`，进一步展开其内部控制流或状态更新。 条件分支用于区分快速路径、后端特定情况以及面向用户的不变量。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 123-136 / 第 123-136 行
````python
    def get_memory_bandwidth_Bps(self) -> int:
        # DRAM devices are Double-Data which means they provide an output at both fronts of
        # a clock beat
        bus_bytes_per_cycle = int(2 * self.device_properties.memory_bus_width / 8)
        mem_clock_rate_Hz = self.device_properties.memory_clock_rate * 1000
        bytes_per_second = bus_bytes_per_cycle * mem_clock_rate_Hz
        return bytes_per_second

    def get_shared_memory_bandwidth_Bps(self) -> int:
        # Each warp can LD or ST 32 x 4 bytes per cycle. To calculate the
        # device's throughput we need to multiply with frequency and number of SMs.
        num_sms = self.device_properties.multi_processor_count
        bytes_per_cycle_per_sm = 128
        bytes_per_cycle_per_device = num_sms * bytes_per_cycle_per_sm
````
- **EN**: This chunk defines `get_shared_memory_bandwidth_Bps`, which retrieves runtime state and exposes it through a Python-friendly accessor. Context-manager style flow scopes temporary state such as streams, autocast modes, or tracing/capture boundaries. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段定义了 `get_shared_memory_bandwidth_Bps`，其作用是获取运行时状态，并通过 Python 友好的访问器暴露出去。 类似上下文管理器的流程会限定临时状态的作用域，例如流、autocast 模式或 tracing/捕获边界。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

### Lines 137-140 / 第 137-140 行
````python
        bytes_per_second = (
            bytes_per_cycle_per_device * self.device_properties.clock_rate * 1000
        )
        return bytes_per_second
````
- **EN**: This chunk continues `get_shared_memory_bandwidth_Bps` and expands its internal control flow or state updates. The tail returns the assembled value or hands updated state back to the caller.
- **CN**: 这一段延续了 `get_shared_memory_bandwidth_Bps`，进一步展开其内部控制流或状态更新。 末尾会返回组装好的值，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **CUDA runtime**
  - EN: Wraps streams, devices, memory state, and CUDA-specific coordination points.
  - CN: 封装流、设备、内存状态以及 CUDA 专用协同点。
- **__all__**
  - EN: `__all__` is one of the main symbols declared or implemented in this file.
  - CN: `__all__` 是本文件声明或实现的主要符号之一。
- **GPULimits**
  - EN: `GPULimits` is one of the main symbols declared or implemented in this file.
  - CN: `GPULimits` 是本文件声明或实现的主要符号之一。
- **Memory management**
  - EN: The file manages caches, pools, or allocation-related state exposed to Python.
  - CN: 该文件管理暴露给 Python 的缓存、内存池或分配相关状态。
- **Type surface**
  - EN: The code uses typing metadata or scalar/layout enums to make APIs safer and clearer.
  - CN: 代码使用类型元数据或标量/布局枚举，让 API 更安全、更清晰。
## Dependencies / 依赖关系

- **Internal torch modules / torch 内部模块**: `torch`, `torch._C`
- **Primary symbols in this file / 本文件核心符号**: `__all__`, `GPULimits`
