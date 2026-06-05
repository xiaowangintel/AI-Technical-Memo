# amx_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/amx_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements low-precision quantization data paths for the SGLang SRT runtime. It exposes symbols such as `CPUQuantMethod`, `CPUQuantAlgo`, `amx_process_weight_after_loading`, and `dim_is_supported` and connects them to backend-specific paths such as `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了低精度量化数据路径。它提供了 `CPUQuantMethod`、`CPUQuantAlgo`、`amx_process_weight_after_loading` 以及 `dim_is_supported` 等符号，并把这些符号连接到 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Imports, constants, and runtime setup
```python
import logging

import torch

from sglang.srt.utils import cpu_has_amx_support

logger = logging.getLogger(__name__)

from enum import IntEnum
```
**EN:** This section prepares the module namespace. It imports `logging`, `torch`, `sglang.srt.utils.cpu_has_amx_support`, and `enum.IntEnum`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `logger` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `logging`、`torch`、`sglang.srt.utils.cpu_has_amx_support` 以及 `enum.IntEnum`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `logger` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 12-18: Class `CPUQuantMethod` declaration and shared state
```python
class CPUQuantMethod(IntEnum):
    UNQUANT = 0
    INT8_W8A8 = 1
    FP8_W8A16 = 2
    INT4_W4A8 = 3
```
**EN:** This block introduces class `CPUQuantMethod` and the state shared by its methods. It inherits from `IntEnum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `CPUQuantMethod`，并定义其方法共享的状态。 它继承自 `IntEnum`，说明了它在 SRT 层栈中的接入方式。

### Lines 19-23: Class `CPUQuantAlgo` declaration and shared state
```python
class CPUQuantAlgo(IntEnum):
    AWQ = 0
    GPTQ = 1
```
**EN:** This block introduces class `CPUQuantAlgo` and the state shared by its methods. It inherits from `IntEnum`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `CPUQuantAlgo`，并定义其方法共享的状态。 它继承自 `IntEnum`，说明了它在 SRT 层栈中的接入方式。

### Lines 24-39: Function `amx_process_weight_after_loading` and its core logic
```python
def amx_process_weight_after_loading(weight, is_conv=False):
    if weight.device != torch.device("cpu"):
        return weight
    if not cpu_has_amx_support():
        return weight
    if is_conv:
        return torch.ops.sgl_kernel.causal_conv1d_weight_pack(
            weight.view(-1, weight.size(-1))
        )
    else:
        return torch.ops.sgl_kernel.convert_weight_packed(weight)


# TODO: currently gemm kernel has the below requirements:
# OC: OC % TILE_N == 0 or OC < TILE_N, where TILE_N = 16
# IC: IC % TILE_K == 0, where TILE_K = 32
```
**EN:** This block defines `amx_process_weight_after_loading` and contains the main logic for this step. It mainly invokes `torch.device`, `cpu_has_amx_support`, `torch.ops.sgl_kernel.causal_conv1d_weight_pack`, `torch.ops.sgl_kernel.convert_weight_packed`, and `weight.view`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `amx_process_weight_after_loading`，并承载这一阶段的核心逻辑。 它主要调用 `torch.device`、`cpu_has_amx_support`、`torch.ops.sgl_kernel.causal_conv1d_weight_pack`、`torch.ops.sgl_kernel.convert_weight_packed` 以及 `weight.view`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 40-50: Function `dim_is_supported` and its core logic
```python
def dim_is_supported(weight):
    TILE_N = 16
    TILE_K = 32
    ndim = weight.ndim
    OC = weight.size(1) if ndim == 3 else weight.size(0)
    IC = weight.size(2) if ndim == 3 else weight.size(1)
    is_oc_support = OC < TILE_N or OC % TILE_N == 0
    is_ic_support = IC % TILE_K == 0
    return is_oc_support and is_ic_support
```
**EN:** This block defines `dim_is_supported` and contains the main logic for this step. It mainly invokes `weight.size`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `TILE_N`, `TILE_K`, `ndim`, `OC`, and `IC` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `dim_is_supported`，并承载这一阶段的核心逻辑。 它主要调用 `weight.size`，说明该流程会编排底层辅助函数或计算内核。 像 `TILE_N`、`TILE_K`、`ndim`、`OC` 以及 `IC` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 51-59: Function `dtype_is_supported` and its core logic
```python
def dtype_is_supported(weight):
    return weight.dtype in [
        torch.float16,
        torch.bfloat16,
        torch.int8,
        torch.float8_e4m3fn,
    ]
```
**EN:** This block defines `dtype_is_supported` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `dtype_is_supported`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 60-63: `is_dim_conv_weight` predicate for is dim convolution weight
```python
def is_dim_conv_weight(weight):
    return weight.dim() == 3 and weight.size(1) == 1
```
**EN:** This block defines `is_dim_conv_weight` and contains the main logic for this step. It mainly invokes `weight.dim` and `weight.size`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `is_dim_conv_weight`，并承载这一阶段的核心逻辑。 它主要调用 `weight.dim` 和 `weight.size`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 64-80: Internal helper `_init_amx_conv_state`
```python
def _init_amx_conv_state(conv_state):
    # CPU AMX layout for conv_state kernel optimization
    conv_state_cpu = []
    for conv_shape_t in conv_state:
        conv_shape_new = conv_shape_t.as_strided_(
            conv_shape_t.size(),
            (
                conv_shape_t.stride(0),
                conv_shape_t.stride(1),
                1,
                conv_shape_t.size(2),
            ),
        )
        conv_state_cpu.append(conv_shape_new)
    return conv_state_cpu
```
**EN:** This block defines `_init_amx_conv_state` and contains the main logic for this step. It mainly invokes `conv_shape_t.as_strided_`, `conv_state_cpu.append`, `conv_shape_t.size`, and `conv_shape_t.stride`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `conv_state_cpu` and `conv_shape_new` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `_init_amx_conv_state`，并承载这一阶段的核心逻辑。 它主要调用 `conv_shape_t.as_strided_`、`conv_state_cpu.append`、`conv_shape_t.size` 以及 `conv_shape_t.stride`，说明该流程会编排底层辅助函数或计算内核。 像 `conv_state_cpu` 和 `conv_shape_new` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 81-164: Internal helper `_amx_process_weight_after_loading`
```python
def _amx_process_weight_after_loading(
    module, weight_names, transpose_dims=None, qweight_packed_method=None
) -> None:
    # Pack weight for get better performance on CPU
    devices = {getattr(module, weight_name).device for weight_name in weight_names}
    assert len(devices) == 1, f"Expects all weights to be on the same device"
    device = devices.pop()

    if transpose_dims:
        assert len(weight_names) == len(
            transpose_dims
        ), "len(weight_names) should be equal to len(transpose_dims)"

    module.use_intel_amx_backend = (
        device == torch.device("cpu") and cpu_has_amx_support()
    )

    if qweight_packed_method is None:
        for i, weight_name in enumerate(weight_names):
            weight_tensor = getattr(module, weight_name)

            if transpose_dims and transpose_dims[i]:
                weight_tensor = weight_tensor.transpose(*transpose_dims[i])
            is_conv_weight = is_dim_conv_weight(weight_tensor)
            # We don't pack weight or use intel amx backend if any weight of this module has unsupported dim.
            if (
                (not dim_is_supported(weight_tensor))
                or not dtype_is_supported(weight_tensor)
            ) and (not is_conv_weight):
                logger.warning(
                    f"Unsupported dimension or dtype for prepacking for weight '{weight_name}' with shape {weight_tensor.shape} and dtype {weight_tensor.dtype} in {module}. "
                    f"The derived (OC, IC) dimensions must be divisible by (16, 32). "
                )
                module.use_intel_amx_backend = False
                return

            packed_weight = torch.nn.Parameter(
                amx_process_weight_after_loading(weight_tensor, is_conv_weight),
                requires_grad=False,
            )
            packed_weight.__dict__ = weight_tensor.__dict__
            setattr(module, weight_name, packed_weight)
            if is_conv_weight:
                # need to use inplace copy for conv weight amx packing,
                # as its usage in radix_linear_attention will use the original conv weight.
                weight_tensor = weight_tensor.view(-1, weight_tensor.size(-1))
                weight_tensor.copy_(packed_weight)
    else:
        assert qweight_packed_method in ["awq", "gptq"]
        qweight_tensor = getattr(module, weight_names[0])
        qzeros_tensor = getattr(module, weight_names[1])
        scales_tensor = getattr(module, weight_names[2])
        qweight, qzeros, scales = torch.ops.sgl_kernel.convert_weight_packed_scale_zp(
            qweight_tensor,
            qzeros_tensor,
            scales_tensor,
            CPUQuantAlgo.AWQ if qweight_packed_method == "awq" else CPUQuantAlgo.GPTQ,
        )
        packed_qweight = torch.nn.Parameter(
            qweight.detach(),
            requires_grad=False,
        )
        packed_qzeros = torch.nn.Parameter(
            qzeros.detach(),
            requires_grad=False,
        )
        packed_scales = torch.nn.Parameter(
            scales.detach(),
            requires_grad=False,
        )
        packed_qweight.__dict__ = qweight_tensor.__dict__
        packed_qzeros.__dict__ = qzeros_tensor.__dict__
        packed_scales.__dict__ = scales_tensor.__dict__
        setattr(module, weight_names[0], packed_qweight)
        setattr(module, weight_names[1], packed_qzeros)
        setattr(module, weight_names[2], packed_scales)
    if (
        module.use_intel_amx_backend
        and hasattr(module, "bias")
        and module.bias is not None
    ):
        module.bias = torch.nn.Parameter(module.bias.data.float(), requires_grad=False)
```
**EN:** This block defines `_amx_process_weight_after_loading` and contains the main logic for this step. It mainly invokes `devices.pop`, `len`, `cpu_has_amx_support`, `enumerate`, and `getattr`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `devices`, `device`, `qweight_tensor`, `qzeros_tensor`, and `scales_tensor` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `_amx_process_weight_after_loading`，并承载这一阶段的核心逻辑。 它主要调用 `devices.pop`、`len`、`cpu_has_amx_support`、`enumerate` 以及 `getattr`，说明该流程会编排底层辅助函数或计算内核。 像 `devices`、`device`、`qweight_tensor`、`qzeros_tensor` 以及 `scales_tensor` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 165-165: Class `PackWeightMethod` declaration and shared state
```python
class PackWeightMethod:
```
**EN:** This block introduces class `PackWeightMethod` and the state shared by its methods.
**CN:** 该代码块引入类 `PackWeightMethod`，并定义其方法共享的状态。

### Lines 166-169: `PackWeightMethod` initialization and state setup
```python
    def __init__(self, weight_names, transpose_dims=None):
        self.weight_names = weight_names
        self.transpose_dims = transpose_dims
```
**EN:** This block defines `PackWeightMethod.__init__` and contains the main logic for this step. Intermediate names such as `self.weight_names` and `self.transpose_dims` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `PackWeightMethod.__init__`，并承载这一阶段的核心逻辑。 像 `self.weight_names` 和 `self.transpose_dims` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 170-173: Function `PackWeightMethod.process_weights_after_loading` and its core logic
```python
    def process_weights_after_loading(self, module) -> None:
        _amx_process_weight_after_loading(
            module, self.weight_names, self.transpose_dims
        )
```
**EN:** This block defines `PackWeightMethod.process_weights_after_loading` and contains the main logic for this step. It mainly invokes `_amx_process_weight_after_loading`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `PackWeightMethod.process_weights_after_loading`，并承载这一阶段的核心逻辑。 它主要调用 `_amx_process_weight_after_loading`，说明该流程会编排底层辅助函数或计算内核。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `CPUQuantMethod`, `CPUQuantAlgo`, `amx_process_weight_after_loading`, `dim_is_supported`, and `dtype_is_supported`. / **主要符号**：核心入口包括 `CPUQuantMethod`、`CPUQuantAlgo`、`amx_process_weight_after_loading`、`dim_is_supported` 以及 `dtype_is_supported`。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Projection layers**: Focuses on matrix multiplication, weight layout, and projection-oriented wrappers. / **投影层**：关注矩阵乘法、权重布局与面向投影的包装层。

## Dependencies / 依赖关系
- **Standard library**: `logging` and `enum.IntEnum` / **标准库**：`logging` 和 `enum.IntEnum`
- **Third-party**: `torch` / **第三方依赖**：`torch`
- **Internal SGLang modules**: `sglang.srt.utils.cpu_has_amx_support` / **SGLang 内部模块**：`sglang.srt.utils.cpu_has_amx_support`
