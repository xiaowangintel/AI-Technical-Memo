# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/linear/scaled_mm/__init__.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Re-exports the public symbols for the scaled mm package. / 重新导出 scaled mm 包的公共符号。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-41)
```python
from vllm.model_executor.kernels.linear.scaled_mm.aiter import (
    AiterInt8ScaledMMLinearKernel,
)
from vllm.model_executor.kernels.linear.scaled_mm.BlockScaledMMLinearKernel import (
    Fp8BlockScaledMMLinearKernel,
)
from vllm.model_executor.kernels.linear.scaled_mm.cpu import (
    CPUFp8BlockScaledMMKernel,
    CPUInt8ScaledMMLinearKernel,
)
from vllm.model_executor.kernels.linear.scaled_mm.cutlass import (
    CutlassFP8ScaledMMLinearKernel,
    CutlassInt8ScaledMMLinearKernel,
)
from vllm.model_executor.kernels.linear.scaled_mm.flashinfer import (
    FlashInferFP8ScaledMMLinearKernel,
)
from vllm.model_executor.kernels.linear.scaled_mm.marlin import (
    MarlinFP8ScaledMMLinearKernel,
)
from vllm.model_executor.kernels.linear.scaled_mm.pytorch import (
    ChannelWiseTorchFP8ScaledMMLinearKernel,
    PerTensorTorchFP8ScaledMMLinearKernel,
    RowWiseTorchFP8ScaledMMLinearKernel,
)
from vllm.model_executor.kernels.linear.scaled_mm.rocm import (
    ROCmFP8ScaledMMLinearKernel,
)
from vllm.model_executor.kernels.linear.scaled_mm.ScaledMMLinearKernel import (
    FP8ScaledMMLinearKernel,
    FP8ScaledMMLinearLayerConfig,
    Int8ScaledMMLinearKernel,
    Int8ScaledMMLinearLayerConfig,
    ScaledMMLinearKernel,
)
from vllm.model_executor.kernels.linear.scaled_mm.triton import (
    TritonInt8ScaledMMLinearKernel,
)
```
**EN:** This import block loads `vllm.model_executor.kernels.linear.scaled_mm.aiter`, `vllm.model_executor.kernels.linear.scaled_mm.BlockScaledMMLinearKernel`, `vllm.model_executor.kernels.linear.scaled_mm.cpu`, `vllm.model_executor.kernels.linear.scaled_mm.cutlass`, `vllm.model_executor.kernels.linear.scaled_mm.flashinfer`, `vllm.model_executor.kernels.linear.scaled_mm.marlin`, ..., establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `__init__.py`.
**CN:** 该导入代码块加载了 `vllm.model_executor.kernels.linear.scaled_mm.aiter`, `vllm.model_executor.kernels.linear.scaled_mm.BlockScaledMMLinearKernel`, `vllm.model_executor.kernels.linear.scaled_mm.cpu`, `vllm.model_executor.kernels.linear.scaled_mm.cutlass`, `vllm.model_executor.kernels.linear.scaled_mm.flashinfer`, `vllm.model_executor.kernels.linear.scaled_mm.marlin`, ...，为 `__init__.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Constants / assignments (lines 43-63)
```python
__all__ = [
    "FP8ScaledMMLinearKernel",
    "FP8ScaledMMLinearLayerConfig",
    "Int8ScaledMMLinearKernel",
    "Int8ScaledMMLinearLayerConfig",
    "ScaledMMLinearKernel",
    "ScaledMMLinearLayerConfig",
    "AiterInt8ScaledMMLinearKernel",
    "CPUInt8ScaledMMLinearKernel",
    "CutlassFP8ScaledMMLinearKernel",
    "CutlassInt8ScaledMMLinearKernel",
    "FlashInferFP8ScaledMMLinearKernel",
    "MarlinFP8ScaledMMLinearKernel",
    "ChannelWiseTorchFP8ScaledMMLinearKernel",
    "PerTensorTorchFP8ScaledMMLinearKernel",
    "RowWiseTorchFP8ScaledMMLinearKernel",
    "ROCmFP8ScaledMMLinearKernel",
    "TritonInt8ScaledMMLinearKernel",
    "Fp8BlockScaledMMLinearKernel",
    "CPUFp8BlockScaledMMKernel",
]
```
**EN:** This block defines the public export surface so callers can import a stable package API without depending on internal file layout.
**CN:** 该代码块定义了公共导出集合，使调用方可以在不依赖内部文件布局的情况下使用稳定的包级 API。

## Key Concepts / 关键概念
- **Triton kernels / Triton 内核**
  - **EN:** The implementation relies on Triton-generated GPU kernels for low-level tensor work.
  - **CN:** 该实现依赖 Triton 生成的 GPU 内核完成底层张量计算。
- **CUTLASS backend / CUTLASS 后端**
  - **EN:** The module selects or wraps CUTLASS kernels for NVIDIA-oriented matrix multiplication.
  - **CN:** 该模块选择或封装 CUTLASS 内核，以支持面向 NVIDIA 的矩阵乘计算。
- **FlashInfer backend / FlashInfer 后端**
  - **EN:** The code integrates FlashInfer primitives for fast low-precision inference paths.
  - **CN:** 代码集成 FlashInfer 原语，以支持高效低精度推理路径。
- **Marlin backend / Marlin 后端**
  - **EN:** The file exposes Marlin-specific kernels or selection logic for quantized linear layers.
  - **CN:** 该文件暴露 Marlin 专用内核或量化线性层的选择逻辑。
- **AITER backend / AITER 后端**
  - **EN:** The module routes execution through AITER custom operators or kernel helpers.
  - **CN:** 该模块通过 AITER 自定义算子或内核辅助逻辑完成执行。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm.model_executor.kernels.linear.scaled_mm.aiter`, `vllm.model_executor.kernels.linear.scaled_mm.BlockScaledMMLinearKernel`, `vllm.model_executor.kernels.linear.scaled_mm.cpu`, `vllm.model_executor.kernels.linear.scaled_mm.cutlass`, `vllm.model_executor.kernels.linear.scaled_mm.flashinfer`, `vllm.model_executor.kernels.linear.scaled_mm.marlin`, `vllm.model_executor.kernels.linear.scaled_mm.pytorch`, `vllm.model_executor.kernels.linear.scaled_mm.rocm`, `vllm.model_executor.kernels.linear.scaled_mm.ScaledMMLinearKernel`, `vllm.model_executor.kernels.linear.scaled_mm.triton`
- **External / 外部依赖**: None
