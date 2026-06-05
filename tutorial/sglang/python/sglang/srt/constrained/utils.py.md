# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/constrained/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the constrained decoding and grammar control part of the SRT runtime and implements logic centered on `utils`. It exposes primary entry points such as `is_legacy_structural_tag`. / 该模块属于 SRT 运行时的约束解码与语法控制部分，主要实现围绕 `utils` 的逻辑。 它对外提供的主要入口包括 `is_legacy_structural_tag`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: Module imports, constants, and setup
```python
from typing import Dict


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 4-12: Function is_legacy_structural_tag
```python
def is_legacy_structural_tag(obj: Dict) -> bool:
    # test whether an object is a legacy structural tag
    # see `StructuralTagResponseFormat` at `sglang.srt.entrypoints.openai.protocol`
    if obj.get("structures", None) is not None:
        assert obj.get("triggers", None) is not None
        return True
    else:
        assert obj.get("format", None) is not None
        return False
```
**EN:** This callable implements `is_legacy_structural_tag`. It takes `obj` and mainly implements is legacy structural tag. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `is_legacy_structural_tag`。它接收 `obj`，主要用于实现 is legacy structural tag 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

## Key Concepts / 关键概念
- `is_legacy_structural_tag`: implements is legacy structural tag / 实现 is legacy structural tag 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
