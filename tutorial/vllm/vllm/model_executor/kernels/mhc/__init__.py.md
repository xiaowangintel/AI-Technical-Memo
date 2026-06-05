# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/mhc/__init__.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Re-exports the public symbols for the mhc package. / 重新导出 mhc 包的公共符号。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-6)
```python
from .aiter import *
from .tilelang import *
from .torch import *
from .triton import *
```
**EN:** This import block loads `.aiter`, `.tilelang`, `.torch`, `.triton`, establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `__init__.py`.
**CN:** 该导入代码块加载了 `.aiter`, `.tilelang`, `.torch`, `.triton`，为 `__init__.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Constants / assignments (lines 8-29)
```python
__all__ = [
    "mhc_pre_cuda",
    "mhc_post_cuda",
    "mhc_fused_post_pre_cuda",
    "hc_head_fused_kernel_cuda",
    "mhc_pre_aiter",
    "mhc_post_aiter",
    "mhc_fused_post_pre_aiter",
    "hc_head_fused_aiter",
    "mhc_pre_tilelang",
    "mhc_post_tilelang",
    "mhc_fused_post_pre_tilelang",
    "hc_head_fused_tilelang",
    "mhc_pre_torch",
    "mhc_post_torch",
    "mhc_fused_post_pre_torch",
    "hc_head_fused_torch",
    "mhc_pre_triton",
    "mhc_post_triton",
    "mhc_fused_post_pre_triton",
    "hc_head_fused_triton",
]
```
**EN:** This block defines the public export surface so callers can import a stable package API without depending on internal file layout.
**CN:** 该代码块定义了公共导出集合，使调用方可以在不依赖内部文件布局的情况下使用稳定的包级 API。

## Key Concepts / 关键概念
- **Triton kernels / Triton 内核**
  - **EN:** The implementation relies on Triton-generated GPU kernels for low-level tensor work.
  - **CN:** 该实现依赖 Triton 生成的 GPU 内核完成底层张量计算。
- **AITER backend / AITER 后端**
  - **EN:** The module routes execution through AITER custom operators or kernel helpers.
  - **CN:** 该模块通过 AITER 自定义算子或内核辅助逻辑完成执行。
- **MHC kernels / MHC 内核**
  - **EN:** The implementation provides backend-specific MHC kernels and wrappers.
  - **CN:** 该实现提供面向不同后端的 MHC 内核与包装层。
- **Package exports / 包导出**
  - **EN:** The file stabilizes the package API by re-exporting selected symbols.
  - **CN:** 该文件通过重新导出符号来稳定包级 API。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `.aiter`, `.tilelang`, `.torch`, `.triton`
- **External / 外部依赖**: None
