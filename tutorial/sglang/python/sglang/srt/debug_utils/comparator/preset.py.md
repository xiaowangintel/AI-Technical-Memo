# preset.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/preset.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on preset debug configurations. It mainly packages common settings for repeatable debug sessions. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于预设调试配置。它主要用于封装常用设置以支持可重复的调试会话。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 3-20: Declare module-level symbols such as `PRESETS`, `DEFAULT_PRESET` / 声明模块级符号，例如 `PRESETS`, `DEFAULT_PRESET`
```python
PRESETS: dict[str, list[str]] = {
    "raw": [
        "--grouping-skip-keys",
    ],
    "sglang_dev": [
        "--grouping-skip-keys",
        "rank",
    ],
    "sglang_megatron": [
        "--grouping-skip-keys",
        "rank",
        "step",
        "--token-aligner",
        "concat_steps",
    ],
}

DEFAULT_PRESET: str = "sglang_dev"
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 23-35: Implement function `expand_preset` / 实现函数 `expand_preset`
```python
def expand_preset(argv: list[str], presets: dict[str, list[str]]) -> list[str]:
    """Expand ``--preset <name>`` into the corresponding argv fragment.

    If ``--preset`` is absent **and** ``--grouping-skip-keys`` is also absent,
    the DEFAULT_PRESET is applied automatically.
    """
    if (expanded := _expand_flag(argv, "--preset", presets)) is not None:
        return expanded

    if "--grouping-skip-keys" not in argv:
        return presets[DEFAULT_PRESET] + argv

    return argv
```
**EN:** Function `expand_preset` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `expand_preset` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 38-52: Implement helper `_expand_flag` / 实现辅助函数 `_expand_flag`
```python
def _expand_flag(
    argv: list[str], flag: str, mapping: dict[str, list[str]]
) -> list[str] | None:
    """Replace ``flag <name>`` in *argv* with the corresponding argv fragment from *mapping*."""
    if flag not in argv:
        return None

    idx: int = argv.index(flag)
    name: str = argv[idx + 1]
    if name not in mapping:
        raise ValueError(
            f"Unknown value for {flag}: {name}. Available: {list(mapping.keys())}"
        )

    return argv[:idx] + mapping[name] + argv[idx + 2 :]
```
**EN:** Function `_expand_flag` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_expand_flag` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `expand_preset`, `_expand_flag`
- **Module role / 模块角色**: Preset debug configurations / 预设调试配置
- **Implementation focus / 实现重点**: Packages common settings for repeatable debug sessions / 封装常用设置以支持可重复的调试会话

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: None / 无
