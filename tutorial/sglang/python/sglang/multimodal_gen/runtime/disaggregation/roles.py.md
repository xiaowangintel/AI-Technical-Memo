# roles.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/disaggregation/roles.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the disaggregation runtime layer. It centers on `RoleType`, `get_module_role`, and `filter_modules_for_role`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Role definitions for diffusion pipeline disaggregation. / 该文件属于解耦运行时层。它围绕 `RoleType`、`get_module_role` 和 `filter_modules_for_role` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-4: module setup and imports / 模块初始化与导入
```python
"""Role definitions for diffusion pipeline disaggregation."""

from enum import Enum
```
**EN:** This block establishes the module context and imports `enum`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `enum`。这些依赖为后续实现提供所需符号。

### Lines 6-6: supporting statements / 辅助语句
```python
_ROLE_ALIASES = {"denoising": "denoiser"}
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_ROLE_ALIASES`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_ROLE_ALIASES` 等名称。

### Lines 9-9: `RoleType` class overview / `RoleType` 类概览
```python
class RoleType(str, Enum):
```
**EN:** This block defines class `RoleType`. It encapsulates role type behavior. It inherits from `str`, and `Enum`.
**CN:** 该代码块定义了类 `RoleType`。 它用于封装 role type 相关行为。 它继承自 `str` 和 `Enum`。

### Lines 10-14: supporting statements / 辅助语句
```python
    MONOLITHIC = "monolithic"
    ENCODER = "encoder"
    DENOISER = "denoiser"
    DECODER = "decoder"
    SERVER = "server"  # Head node (no GPU, routes requests)
```
**EN:** This block gathers supporting statements inside `RoleType`. It updates names such as `MONOLITHIC`, `ENCODER`, `DENOISER`, `DECODER`, and `SERVER`.
**CN:** 该代码块汇集了位于 `RoleType` 内部的辅助语句。 它会更新 `MONOLITHIC`、`ENCODER`、`DENOISER`、`DECODER` 和 `SERVER` 等名称。

### Lines 16-24: `from_string` implementation / `from_string` 实现
```python
    @classmethod
    def from_string(cls, value: str) -> "RoleType":
        v = _ROLE_ALIASES.get(value.lower(), value.lower())
        try:
            return cls(v)
        except ValueError:
            raise ValueError(
                f"Invalid role: {value}. Must be one of: {', '.join([r.value for r in cls])}"
            ) from None
```
**EN:** This block defines method `from_string` on `RoleType`. It constructs from string. Key calls include `_ROLE_ALIASES.get`, `value.lower`, `cls`, `ValueError`, and `join`. The implementation handles exceptional paths. Parameters such as `value` drive the behavior in this section.
**CN:** 该代码块定义了 `RoleType` 的方法 `from_string`。 它用于从…构造string。 关键调用包括 `_ROLE_ALIASES.get`、`value.lower`、`cls`、`ValueError` 和 `join`。 实现中处理异常路径。 本段逻辑主要由 `value` 等参数驱动。

### Lines 26-28: `choices` implementation / `choices` 实现
```python
    @classmethod
    def choices(cls) -> list[str]:
        return [role.value for role in cls]
```
**EN:** This block defines method `choices` on `RoleType`. It handles choices logic.
**CN:** 该代码块定义了 `RoleType` 的方法 `choices`。 它用于处理 choices 相关逻辑。

### Lines 31-58: `get_module_role` implementation / `get_module_role` 实现
```python
def get_module_role(module_name: str) -> "RoleType | None":
    """Classify a module name to its primary role. Returns None for shared modules."""
    encoder_prefixes = (
        "text_encoder",
        "tokenizer",
        "image_encoder",
        "image_processor",
        "processor",
        "connectors",
    )
    if any(
        module_name == p or module_name.startswith(p + "_") for p in encoder_prefixes
    ):
        return RoleType.ENCODER

    denoising_prefixes = ("transformer",)
    if any(
        module_name == p or module_name.startswith(p + "_") for p in denoising_prefixes
    ):
        return RoleType.DENOISER

    decoder_prefixes = ("vae", "audio_vae", "video_vae", "vocoder")
    if any(
        module_name == p or module_name.startswith(p + "_") for p in decoder_prefixes
    ):
        return RoleType.DECODER

    return None
```
**EN:** This block defines function `get_module_role`. Classify a module name to its primary role. Returns None for shared modules. Key calls include `any`, and `module_name.startswith`. The implementation branches on conditions. Parameters such as `module_name` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_module_role`。 它用于获取module role。 关键调用包括 `any` 和 `module_name.startswith`。 实现中包含条件分支。 本段逻辑主要由 `module_name` 等参数驱动。

### Lines 61-78: `filter_modules_for_role` implementation / `filter_modules_for_role` 实现
```python
def filter_modules_for_role(module_names: list[str], role: "RoleType") -> list[str]:
    """Filter module names to only those needed by the given role."""
    if role in (RoleType.MONOLITHIC, RoleType.SERVER):
        return module_names

    filtered = []
    for name in module_names:
        module_role = get_module_role(name)

        if module_role is None:
            filtered.append(name)
        elif module_role == role:
            filtered.append(name)
        elif role == RoleType.ENCODER and module_role == RoleType.DECODER:
            # Encoder also needs VAE for ImageVAEEncoding stages
            filtered.append(name)

    return filtered
```
**EN:** This block defines function `filter_modules_for_role`. Filter module names to only those needed by the given role. Key calls include `get_module_role`, and `filtered.append`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `module_names`, and `role` drive the behavior in this section.
**CN:** 该代码块定义了函数 `filter_modules_for_role`。 它用于处理 filter modules for role 相关逻辑。 关键调用包括 `get_module_role` 和 `filtered.append`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `module_names` 和 `role` 等参数驱动。

## Key Concepts / 关键概念
- `RoleType`: Primary class that encapsulates role type behavior. / 核心类，用于封装 role type 相关行为。
- `get_module_role`: Classify a module name to its primary role. / 顶层函数，用于获取module role。
- `filter_modules_for_role`: Filter module names to only those needed by the given role. / 顶层函数，用于处理 filter modules for role 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `enum`

- **Total lines / 总行数**: 78
