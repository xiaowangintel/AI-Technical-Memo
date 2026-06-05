# utils.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/deepseek_common/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module provides shared utility helpers used by the surrounding model implementation. / 该模块提供周边模型实现复用的共享工具函数。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13: Module header and imports / 模块头与导入
```python
# Copyright 2026 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 38-38: Assignment: _is_hip / 赋值：_is_hip
```python
_is_hip = is_hip()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 39-39: Assignment: _is_cuda / 赋值：_is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 40-40: Assignment: _is_npu / 赋值：_is_npu
```python
_is_npu = is_npu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 41-41: Assignment: _is_musa / 赋值：_is_musa
```python
_is_musa = is_musa()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 42-42: Assignment: _is_fp8_fnuz / 赋值：_is_fp8_fnuz
```python
_is_fp8_fnuz = is_fp8_fnuz()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 43-43: Assignment: _use_aiter / 赋值：_use_aiter
```python
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 44-44: Assignment: _is_cpu_amx_available / 赋值：_is_cpu_amx_available
```python
_is_cpu_amx_available = cpu_has_amx_support()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 45-45: Assignment: _is_cpu / 赋值：_is_cpu
```python
_is_cpu = is_cpu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 46-46: Assignment: _is_xpu / 赋值：_is_xpu
```python
_is_xpu = is_xpu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 47-47: Assignment: _device_sm / 赋值：_device_sm
```python
_device_sm = get_device_sm()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 48-48: Assignment: _is_gfx95_supported / 赋值：_is_gfx95_supported
```python
_is_gfx95_supported = is_gfx95_supported()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 49-49: Assignment: _use_aiter_gfx95 / 赋值：_use_aiter_gfx95
```python
_use_aiter_gfx95 = _use_aiter and _is_gfx95_supported
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 52-52: Assignment: _is_cublas_ge_129 / 赋值：_is_cublas_ge_129
```python
_is_cublas_ge_129 = is_nvidia_cublas_version_ge_12_9()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 54-54: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 56-56: Assignment: NVFP4_CKPT_FP8_ATTN_QUANT_MODULES / 赋值：NVFP4_CKPT_FP8_ATTN_QUANT_MODULES
```python
NVFP4_CKPT_FP8_ATTN_QUANT_MODULES = ["q_b_proj"]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 58-67: Assignment: FORWARD_ABSORB_CORE_ATTENTION_BACKENDS / 赋值：FORWARD_ABSORB_CORE_ATTENTION_BACKENDS
```python
FORWARD_ABSORB_CORE_ATTENTION_BACKENDS = [
    "fa3",
    "nsa",
    "flashinfer",
    "cutlass_mla",
    "trtllm_mla",
    "tokenspeed_mla",
    "ascend",
    "intel_xpu",
]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 70-97: Function: awq_dequantize_func() / 函数：awq_dequantize_func()
```python
def awq_dequantize_func():
    """
    Get the AWQ dequantize function for the current device

    Return:
        - The AWQ dequantize function for the current device.
        - None if the current device is not supported.
    """
    if _is_cuda:
        from sgl_kernel import awq_dequantize

        return awq_dequantize
    elif _is_hip:
        from sglang.kernel_api_logging import debug_kernel_api
        from sglang.srt.layers.quantization.awq.awq_triton import (
            awq_dequantize_triton as awq_dequantize,
        )

        return debug_kernel_api(awq_dequantize, op_name="DeepseekCommon.awq_dequantize")
    elif _is_npu:
        from sglang.kernel_api_logging import debug_kernel_api
        from sglang.srt.layers.quantization.awq.awq_triton import (
            awq_dequantize_decomposition as awq_dequantize,
        )

        return debug_kernel_api(awq_dequantize, op_name="DeepseekCommon.awq_dequantize")
    else:
        return None
```
**EN:** This function implements awq dequantize func for the surrounding model/runtime logic.
**CN:** 该函数实现了 awq dequantize func 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 100-108: Function: enable_nextn_moe_bf16_cast_to_fp8() / 函数：enable_nextn_moe_bf16_cast_to_fp8()
```python
def enable_nextn_moe_bf16_cast_to_fp8(
    quant_config: Optional[QuantizationConfig],
) -> bool:
    return (
        envs.SGLANG_NVFP4_CKPT_FP8_NEXTN_MOE.get()
        and quant_config is not None
        and quant_config.get_name() == "modelopt_fp4"
        and get_moe_runner_backend().is_deep_gemm()
    )
```
**EN:** This function implements enable nextn moe bf16 cast to fp8 for the surrounding model/runtime logic. Key parameters include quant_config.
**CN:** 该函数实现了 enable nextn moe bf16 cast to fp8 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 111-114: Function: yarn_get_mscale() / 函数：yarn_get_mscale()
```python
def yarn_get_mscale(scale: float = 1, mscale: float = 1) -> float:
    if scale <= 1:
        return 1.0
    return 0.1 * mscale * math.log(scale) + 1.0
```
**EN:** This function implements yarn get mscale for the surrounding model/runtime logic. Key parameters include scale, mscale.
**CN:** 该函数实现了 yarn get mscale 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 117-123: Function: _get_llama_4_scaling() / 函数：_get_llama_4_scaling()
```python
def _get_llama_4_scaling(
    original_max_position_embeddings: int, scaling_beta: float, positions: torch.Tensor
) -> torch.Tensor:
    scaling = 1 + scaling_beta * torch.log(
        1 + torch.floor(positions / original_max_position_embeddings)
    )
    return scaling[..., None, None]
```
**EN:** This function implements get llama 4 scaling for the surrounding model/runtime logic. Key parameters include original_max_position_embeddings, scaling_beta, positions.
**CN:** 该函数实现了 get llama 4 scaling 相关逻辑，用于支撑周边模型或运行时流程。

## Key Concepts / 关键概念
- **EN:** Backend-aware dispatch that selects specialized attention implementations for different hardware and execution modes. / **CN:** 具备后端感知能力的分派机制，可针对不同硬件与执行模式选择特化注意力实现。
- **EN:** Shared utilities that reduce duplication across DeepSeek-family model integrations. / **CN:** 在 DeepSeek 系列模型集成中复用的共享工具，以减少重复实现。

## Dependencies / 依赖关系
- `logging`
- `math`
- `typing: Optional`
- `torch`
- `sglang.srt.environ: envs`
- `sglang.srt.layers.moe.fused_moe_triton.layer: get_moe_runner_backend`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.quantization.fp8_kernel: is_fp8_fnuz`
- `sglang.srt.utils: cpu_has_amx_support, get_bool_env_var, get_device_sm, is_cpu, is_cuda, is_gfx95_supported, is_hip, is_musa, is_npu, is_nvidia_cublas_version_ge_12_9, is_xpu`
