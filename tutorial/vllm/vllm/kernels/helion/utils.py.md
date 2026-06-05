# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/kernels/helion/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides Helion helper utilities for argument handling and runtime checks. / 提供用于参数处理与运行时检查的 Helion 辅助工具。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 3-3)
```python
"""Utility functions for Helion kernel management."""
```
**EN:** The opening docstring explains the file's role and design intent, giving readers context before the concrete kernel code starts.
**CN:** 开头的文档字符串说明了文件职责与设计意图，让读者在进入具体内核实现前先获得整体上下文。

### Imports (lines 5-6)
```python
from vllm.logger import init_logger
from vllm.platforms import current_platform
```
**EN:** This import block loads `vllm.logger`, `vllm.platforms`, establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `utils.py`.
**CN:** 该导入代码块加载了 `vllm.logger`, `vllm.platforms`，为 `utils.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Constants / assignments (lines 8-46)
```python
logger = init_logger(__name__)

# Maps known variant GPU names (after lowercase/underscore normalization)
# to their canonical form.
#
# Names that are already canonical after normalization are NOT listed here.
# For example, "NVIDIA H200" normalizes to "nvidia_h200" which needs no
# further mapping, and AMD ROCm names like "AMD_Instinct_MI300X" come from
# a controlled lookup table in rocm.py and normalize cleanly to
# "amd_instinct_mi300x". Only names with variant suffixes (form factor,
# memory size, memory type, etc.) that should be stripped need entries.
#
# To add a new GPU variant: run `canonicalize_gpu_name()` without the alias
# to see the normalized name, then add a mapping here if it contains variant
# suffixes that should be stripped (e.g. Blackwell/Rubin variants).
_GPU_NAME_ALIASES: dict[str, str] = {
    # H100 variants
    "nvidia_h100_pcie": "nvidia_h100",
    "nvidia_h100_sxm5": "nvidia_h100",
    "nvidia_h100_80gb_hbm3": "nvidia_h100",
    "nvidia_h100_nvl": "nvidia_h100",
    # H200 variants
    "nvidia_h200_nvl": "nvidia_h200",
    "nvidia_h200_141gb_hbm3e": "nvidia_h200",
    # A100 variants
    "nvidia_a100_sxm4_80gb": "nvidia_a100",
    "nvidia_a100_sxm4_40gb": "nvidia_a100",
    "nvidia_a100_pcie_80gb": "nvidia_a100",
    "nvidia_a100_pcie_40gb": "nvidia_a100",
    "nvidia_a100_80gb_pcie": "nvidia_a100",
    # V100 variants (Tesla-branded)
    "tesla_v100_sxm2_32gb": "tesla_v100",
    "tesla_v100_sxm2_16gb": "tesla_v100",
    "tesla_v100_pcie_32gb": "tesla_v100",
    "tesla_v100_pcie_16gb": "tesla_v100",
    # AMD ROCm variants (from _ROCM_DEVICE_ID_NAME_MAP in rocm.py)
    "amd_instinct_mi300x_hf": "amd_instinct_mi300x",
    # ADD MORE HERE
}
```
**EN:** This block defines module constants (`logger`, `_GPU_NAME_ALIASES`) that encode defaults, feature flags, or operator metadata used later in the file.
**CN:** 该代码块定义了模块常量（`logger`, `_GPU_NAME_ALIASES`），用于表示默认值、特性开关或后续逻辑要使用的算子元数据。

### Function `get_gpu_name` (lines 49-56)
```python
def get_gpu_name(device_id: int | None = None) -> str:
    if device_id is None:
        logger.warning_once(
            "get_gpu_name() called without device_id, defaulting to 0. "
            "This may return the wrong device name in multi-node setups."
        )
        device_id = 0
    return current_platform.get_device_name(device_id)
```
**EN:** This lookup helper implements `get_gpu_name`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `get_device_name`, `warning_once`.
**CN:** 该函数 `get_gpu_name` 封装了此模块中的一段关键运行时逻辑，重点处理 get gpu name 相关工作。 它内部会调用 `get_device_name`, `warning_once` 等例程。

### Function `canonicalize_gpu_name` (lines 59-76)
```python
def canonicalize_gpu_name(name: str) -> str:
    """
    Canonicalize GPU name for use as a platform identifier.

    Converts to lowercase, replaces spaces and hyphens with underscores,
    and maps known variant names to their canonical form via _GPU_NAME_ALIASES.
    e.g., "NVIDIA H100 80GB HBM3" -> "nvidia_h100"
          "NVIDIA A100-SXM4-80GB" -> "nvidia_a100"
          "AMD Instinct MI300X"   -> "amd_instinct_mi300x"
    """
    if not name or not name.strip():
        raise ValueError("GPU name cannot be empty")
    name = name.lower()
    name = name.replace(" ", "_")
    name = name.replace("-", "_")
    if name in _GPU_NAME_ALIASES:
        return _GPU_NAME_ALIASES[name]
    return name
```
**EN:** This helper implements `canonicalize_gpu_name`. Canonicalize GPU name for use as a platform identifier. Internally it relies on calls such as `lower`, `replace`, `ValueError`, `strip`.
**CN:** 该函数 `canonicalize_gpu_name` 封装了此模块中的一段关键运行时逻辑，重点处理 canonicalize gpu name 相关工作。 它内部会调用 `lower`, `replace`, `ValueError`, `strip` 等例程。

### Function `get_canonical_gpu_name` (lines 79-80)
```python
def get_canonical_gpu_name(device_id: int | None = None) -> str:
    return canonicalize_gpu_name(get_gpu_name(device_id))
```
**EN:** This lookup helper implements `get_canonical_gpu_name`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `canonicalize_gpu_name`, `get_gpu_name`.
**CN:** 该函数 `get_canonical_gpu_name` 封装了此模块中的一段关键运行时逻辑，重点处理 get canonical gpu name 相关工作。 它内部会调用 `canonicalize_gpu_name`, `get_gpu_name` 等例程。

## Key Concepts / 关键概念
- **Helion integration / Helion 集成**
  - **EN:** The file plugs vLLM into Helion compilation, registration, or preset configuration flows.
  - **CN:** 该文件把 vLLM 接入 Helion 的编译、注册或预设配置流程。
- **ROCm support / ROCm 支持**
  - **EN:** The implementation targets ROCm-capable devices and their kernel constraints.
  - **CN:** 该实现面向 ROCm 设备及其内核约束。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `vllm.logger`, `vllm.platforms`
- **External / 外部依赖**: None
