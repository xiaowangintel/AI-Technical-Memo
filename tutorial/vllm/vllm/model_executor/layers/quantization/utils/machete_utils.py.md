# machete_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/quantization/utils/machete_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Collects utility helpers for quantization backends, schemes, and utilities. / 汇总量化后端、方案与工具的工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 5-7)
```python
import torch

from vllm.scalar_type import ScalarType, scalar_types
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as `vllm.scalar_type`. That import mix shows the file is part of the quantization backends, schemes, and utilities stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 `vllm.scalar_type`）。这些导入关系表明该文件属于量化后端、方案与工具栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 9-9)
```python
MACHETE_PREPACKED_BLOCK_SHAPE = [64, 128]
```
**EN:** This block defines module-level metadata or constants such as `MACHETE_PREPACKED_BLOCK_SHAPE`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the quantization backends, schemes, and utilities pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `MACHETE_PREPACKED_BLOCK_SHAPE`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在量化后端、方案与工具流程中复用。

### Function `query_machete_supported_quant_types` (lines 12-16)
```python
def query_machete_supported_quant_types(zero_points: bool) -> list[ScalarType]:
    if zero_points:
        return [scalar_types.uint4, scalar_types.uint8]
    else:
        return [scalar_types.uint4b8, scalar_types.uint8b128]
```
**EN:** Defines function `query_machete_supported_quant_types` with signature `query_machete_supported_quant_types(zero_points: bool) -> list[ScalarType]`. It mainly works with `zero_points`; handles quantization-related transformation logic. The body uses branching. Key calls include no notable helper calls.
**CN:** 定义函数 `query_machete_supported_quant_types`，其签名为 `query_machete_supported_quant_types(zero_points: bool) -> list[ScalarType]`。它主要围绕 `zero_points` 展开；处理量化相关的变换逻辑。函数体包含分支判断。关键调用包括 无显著辅助调用。

### Function `query_machete_supported_act_types` (lines 19-20)
```python
def query_machete_supported_act_types(zero_points: bool) -> list[ScalarType]:
    return [torch.float16, torch.bfloat16]
```
**EN:** Defines function `query_machete_supported_act_types` with signature `query_machete_supported_act_types(zero_points: bool) -> list[ScalarType]`. It mainly works with `zero_points`; implements one step in the quantized-weight execution flow. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `query_machete_supported_act_types`，其签名为 `query_machete_supported_act_types(zero_points: bool) -> list[ScalarType]`。它主要围绕 `zero_points` 展开；实现量化权重执行流程中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Function `query_machete_supported_group_sizes` (lines 23-38)
```python
def query_machete_supported_group_sizes(act_type: torch.dtype) -> list[int]:
    """
    Queries the supported group sizes for Machete based on the activation type.

    Args:
        act_type: The activation data type (torch.float16, torch.bfloat16).

    Returns:
        A list of supported group sizes. The group size must
        be divisible by `TileShapeK = 128 * 8 // num_bits(act_type)`.
        -1 indicates per-channel quantization.
    """
    if act_type in [torch.float16, torch.bfloat16]:
        return [-1, 64, 128]
    else:
        return [-1, 128]
```
**EN:** Defines function `query_machete_supported_group_sizes` with signature `query_machete_supported_group_sizes(act_type: torch.dtype) -> list[int]`. It mainly works with `act_type`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include no notable helper calls.
**CN:** 定义函数 `query_machete_supported_group_sizes`，其签名为 `query_machete_supported_group_sizes(act_type: torch.dtype) -> list[int]`。它主要围绕 `act_type` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 无显著辅助调用。

### Function `check_machete_supports_shape` (lines 41-56)
```python
def check_machete_supports_shape(
    in_features: int, out_features: int
) -> tuple[bool, str | None]:
    if in_features % MACHETE_PREPACKED_BLOCK_SHAPE[0] != 0:
        return (
            False,
            "Input features size must be divisible by "
            f"{MACHETE_PREPACKED_BLOCK_SHAPE[0]}",
        )
    if out_features % MACHETE_PREPACKED_BLOCK_SHAPE[1] != 0:
        return (
            False,
            "Output features size must be divisible by "
            f"{MACHETE_PREPACKED_BLOCK_SHAPE[1]}",
        )
    return True, None
```
**EN:** Defines function `check_machete_supports_shape` with signature `check_machete_supports_shape(in_features: int, out_features: int) -> tuple[bool, str | None]`. It mainly works with `in_features`, `out_features`; implements one step in the quantized-weight execution flow. The body uses branching. Key calls include no notable helper calls.
**CN:** 定义函数 `check_machete_supports_shape`，其签名为 `check_machete_supports_shape(in_features: int, out_features: int) -> tuple[bool, str | None]`。它主要围绕 `in_features`, `out_features` 展开；实现量化权重执行流程中的一个步骤。函数体包含分支判断。关键调用包括 无显著辅助调用。

## Key Concepts / 关键概念
- **EN:** The file lives in the quantization stack, so weight packing, backend dispatch, and post-load processing are core concerns.
  **CN:** 该文件位于量化栈中，因此权重打包、后端分发和加载后处理是核心关注点。
- **EN:** Top-level helpers such as `query_machete_supported_quant_types`, `query_machete_supported_act_types`, `query_machete_supported_group_sizes`, `check_machete_supports_shape` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `query_machete_supported_quant_types`, `query_machete_supported_act_types`, `query_machete_supported_group_sizes`, `check_machete_supports_shape` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。
- **EN:** Internal vLLM imports show that this module is tightly integrated with the broader runtime, parallelism, or custom-layer stack.
  **CN:** vLLM 内部导入说明该模块与更大的运行时、并行机制或自定义层栈紧密集成。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: `vllm.scalar_type`
