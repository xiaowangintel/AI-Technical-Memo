# _leakage_detection_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/export/_leakage_detection_utils.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements the public-facing `torch.export` API, data models, and helper utilities around exported programs. The file mainly revolves around `_is_globals_or_locals`.
- **Purpose (CN)**: 实现面向用户的 `torch.export` API、数据模型以及导出程序相关辅助工具。 该文件主要围绕 `_is_globals_or_locals` 展开。

## Line-by-Line Analysis / 逐行分析

### Lines 1-21 / 第 1-21 行

````python
0001: import gc
0002: import types
0003: import typing
0004: import weakref
0005: from typing_extensions import TypeIs
0006: 
0007: from torch.fx.experimental.symbolic_shapes import TrackedFake
0008: 
0009: 
0010: """
0011: These functions are used to detect potential fake tensor leakage when using PT2 export.
0012: See NOTE [export non-strict fake tensor leak detection]
0013: 
0014: There are some complications that made this logic overly complicated:
0015: 1) Python 3.10 and Python 3.12 have different ways of implementing referrer so
0016:    we need to account for whether it is ref.__dict__ or the real ref object
0017: 
0018: 2) There are some internal PT2 references to fake tensors like `TrackedFake`
0019: 3) closures, generators, and bound methods can hold fake tensors.
0020: 4) global object can hold onto a fake tensor
0021: 
````

- **L1** EN: Imports module dependencies: `gc`. | CN: 导入模块依赖：`gc`。
- **L2** EN: Imports module dependencies: `types`. | CN: 导入模块依赖：`types`。
- **L3** EN: Imports module dependencies: `typing`. | CN: 导入模块依赖：`typing`。
- **L4** EN: Imports module dependencies: `weakref`. | CN: 导入模块依赖：`weakref`。
- **L5** EN: Imports `TypeIs` from `typing_extensions` so later code can reuse those definitions. | CN: 从 `typing_extensions` 导入 `TypeIs`，供后续代码复用这些定义。
- **L6** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L7** EN: Imports `TrackedFake` from `torch.fx.experimental.symbolic_shapes` so later code can reuse those definitions. | CN: 从 `torch.fx.experimental.symbolic_shapes` 导入 `TrackedFake`，供后续代码复用这些定义。
- **L8** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L9** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L10** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L11** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L12** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L13** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L14** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L15** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L16** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L17** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L18** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L19** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L20** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L21** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 22-43 / 第 22-43 行

````python
0022: In general, these utils are our last resort to detect fake tensors. if the leak happens
0023: within the model attributes, we have a separate mechanism to detect. This tool relies a bit
0024: on garbage collector internal details, so I think it is unsafe to turn on by default, hence
0025: this tool should be used as debugging tool.
0026: """
0027: 
0028: 
0029: # Things we never want to flag as leaks
0030: _SKIP_TYPES = (
0031:     types.FrameType,
0032:     types.ModuleType,
0033: )
0034: 
0035: 
0036: def _is_globals_or_locals(obj: typing.Any) -> bool:
0037:     # These comparisons only make sense within this frame; still cheap to check.
0038:     return obj is globals() or obj is locals()
0039: 
0040: 
0041: def _is_tracked_fake(obj: typing.Any) -> TypeIs[TrackedFake]:
0042:     return isinstance(obj, TrackedFake)
0043: 
````

- **L22** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L23** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L24** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L25** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L26** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L27** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L28** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L29** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L30** EN: Assigns module-level configuration or cached state to `_SKIP_TYPES`. | CN: 为 `_SKIP_TYPES` 赋予模块级配置或缓存状态。
- **L31** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L32** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L33** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L34** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L35** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L36** EN: Defines function `_is_globals_or_locals`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `_is_globals_or_locals`，其作用是实现导出流水线或其元数据处理的一部分。
- **L37** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L38** EN: Returns from `_is_globals_or_locals` with the computed result or updated state. | CN: 从 `_is_globals_or_locals` 返回计算结果或更新后的状态。
- **L39** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L40** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L41** EN: Defines function `_is_tracked_fake`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `_is_tracked_fake`，其作用是实现导出流水线或其元数据处理的一部分。
- **L42** EN: Returns from `_is_tracked_fake` with the computed result or updated state. | CN: 从 `_is_tracked_fake` 返回计算结果或更新后的状态。
- **L43** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 44-64 / 第 44-64 行

````python
0044: 
0045: def _is_gm_meta_like_dict(d: dict, o: typing.Any) -> bool:
0046:     # Hope gm.meta was a custom dict we can assert on
0047:     return d.get("val") is o
0048: 
0049: 
0050: def _dict_is_attr_of_tracked_fake(d: dict) -> bool:
0051:     """
0052:     Python 3.10 quirk: sometimes the referrer is obj.__dict__ instead of obj.
0053:     Check if this dict is exactly the __dict__ of a TrackedFake.
0054:     """
0055:     for parent in gc.get_referrers(d):
0056:         if (
0057:             hasattr(parent, "__dict__")
0058:             and parent.__dict__ is d
0059:             and _is_tracked_fake(parent)
0060:         ):
0061:             return True
0062:     return False
0063: 
0064: 
````

- **L44** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L45** EN: Defines function `_is_gm_meta_like_dict`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `_is_gm_meta_like_dict`，其作用是实现导出流水线或其元数据处理的一部分。
- **L46** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L47** EN: Returns from `_is_gm_meta_like_dict` with the computed result or updated state. | CN: 从 `_is_gm_meta_like_dict` 返回计算结果或更新后的状态。
- **L48** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L49** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L50** EN: Defines function `_dict_is_attr_of_tracked_fake`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `_dict_is_attr_of_tracked_fake`，其作用是实现导出流水线或其元数据处理的一部分。
- **L51** EN: Starts the docstring for function `_dict_is_attr_of_tracked_fake`. | CN: 开始为 function `_dict_is_attr_of_tracked_fake` 编写文档字符串。
- **L52** EN: Continues the docstring for function `_dict_is_attr_of_tracked_fake`. | CN: 继续补充 function `_dict_is_attr_of_tracked_fake` 的文档字符串。
- **L53** EN: Continues the docstring for function `_dict_is_attr_of_tracked_fake`. | CN: 继续补充 function `_dict_is_attr_of_tracked_fake` 的文档字符串。
- **L54** EN: Ends the docstring for function `_dict_is_attr_of_tracked_fake`. | CN: 结束 function `_dict_is_attr_of_tracked_fake` 的文档字符串。
- **L55** EN: Starts an iteration that applies the same rule across elements or steps. | CN: 开始一个迭代过程，把同一规则应用到多个元素或步骤上。
- **L56** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L57** EN: Invokes `hasattr` to advance the surrounding implementation. | CN: 调用 `hasattr` 来推进周围的实现逻辑。
- **L58** EN: Continues `_dict_is_attr_of_tracked_fake`, which implements part of the export pipeline or its metadata handling. | CN: 继续 `_dict_is_attr_of_tracked_fake` 的实现，其作用是实现导出流水线或其元数据处理的一部分。
- **L59** EN: Invokes `_is_tracked_fake` to advance the surrounding implementation. | CN: 调用 `_is_tracked_fake` 来推进周围的实现逻辑。
- **L60** EN: Continues `_dict_is_attr_of_tracked_fake`, which implements part of the export pipeline or its metadata handling. | CN: 继续 `_dict_is_attr_of_tracked_fake` 的实现，其作用是实现导出流水线或其元数据处理的一部分。
- **L61** EN: Returns from `_dict_is_attr_of_tracked_fake` with the computed result or updated state. | CN: 从 `_dict_is_attr_of_tracked_fake` 返回计算结果或更新后的状态。
- **L62** EN: Returns from `_dict_is_attr_of_tracked_fake` with the computed result or updated state. | CN: 从 `_dict_is_attr_of_tracked_fake` 返回计算结果或更新后的状态。
- **L63** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L64** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 65-83 / 第 65-83 行

````python
0065: def find_legit_leaks_from_referrers(active_fakes: weakref.WeakSet) -> weakref.WeakSet:
0066:     legit_leak: weakref.WeakSet = weakref.WeakSet()
0067: 
0068:     # This is so that we don't falsely flag generator to be holding fake tensor
0069:     fake_list = list(active_fakes)
0070:     fake_list_id = id(fake_list)
0071: 
0072:     for act in fake_list:
0073:         # Track by id to avoid processing duplicate referrers
0074:         seen = set()
0075:         # Assume it's a leak unless we find only ignorable referrers
0076:         flagged = False
0077: 
0078:         for r in gc.get_referrers(act):
0079:             rid = id(r)
0080:             if rid in seen:
0081:                 continue
0082:             seen.add(rid)
0083: 
````

- **L65** EN: Defines function `find_legit_leaks_from_referrers`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `find_legit_leaks_from_referrers`，其作用是实现导出流水线或其元数据处理的一部分。
- **L66** EN: Invokes `weakref.WeakSet` to advance the surrounding implementation. | CN: 调用 `weakref.WeakSet` 来推进周围的实现逻辑。
- **L67** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L68** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L69** EN: Assigns or updates `fake_list`. | CN: 对 `fake_list` 进行赋值或更新。
- **L70** EN: Assigns or updates `fake_list_id`. | CN: 对 `fake_list_id` 进行赋值或更新。
- **L71** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L72** EN: Starts an iteration that applies the same rule across elements or steps. | CN: 开始一个迭代过程，把同一规则应用到多个元素或步骤上。
- **L73** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L74** EN: Assigns or updates `seen`. | CN: 对 `seen` 进行赋值或更新。
- **L75** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L76** EN: Assigns or updates `flagged`. | CN: 对 `flagged` 进行赋值或更新。
- **L77** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L78** EN: Starts an iteration that applies the same rule across elements or steps. | CN: 开始一个迭代过程，把同一规则应用到多个元素或步骤上。
- **L79** EN: Assigns or updates `rid`. | CN: 对 `rid` 进行赋值或更新。
- **L80** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L81** EN: Continues `find_legit_leaks_from_referrers`, which implements part of the export pipeline or its metadata handling. | CN: 继续 `find_legit_leaks_from_referrers` 的实现，其作用是实现导出流水线或其元数据处理的一部分。
- **L82** EN: Invokes `seen.add` to advance the surrounding implementation. | CN: 调用 `seen.add` 来推进周围的实现逻辑。
- **L83** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 84-105 / 第 84-105 行

````python
0084:             # Skip our own fake_list
0085:             if rid == fake_list_id:
0086:                 continue
0087: 
0088:             # Fast-path: skip obvious non-owners
0089:             if _is_globals_or_locals(r):
0090:                 continue
0091:             if isinstance(r, _SKIP_TYPES):
0092:                 continue
0093:             if _is_tracked_fake(r):
0094:                 # TrackedFake should be ignored
0095:                 continue
0096: 
0097:             # Handle dicts carefully (Python 3.10 sometimes shows __dict__)
0098:             if isinstance(r, dict):
0099:                 if _is_gm_meta_like_dict(r, act):
0100:                     continue
0101:                 if _dict_is_attr_of_tracked_fake(r):
0102:                     continue
0103:                 flagged = True
0104:                 break
0105: 
````

- **L84** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L85** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L86** EN: Continues `find_legit_leaks_from_referrers`, which implements part of the export pipeline or its metadata handling. | CN: 继续 `find_legit_leaks_from_referrers` 的实现，其作用是实现导出流水线或其元数据处理的一部分。
- **L87** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L88** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L89** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L90** EN: Continues `find_legit_leaks_from_referrers`, which implements part of the export pipeline or its metadata handling. | CN: 继续 `find_legit_leaks_from_referrers` 的实现，其作用是实现导出流水线或其元数据处理的一部分。
- **L91** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L92** EN: Continues `find_legit_leaks_from_referrers`, which implements part of the export pipeline or its metadata handling. | CN: 继续 `find_legit_leaks_from_referrers` 的实现，其作用是实现导出流水线或其元数据处理的一部分。
- **L93** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L94** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L95** EN: Continues `find_legit_leaks_from_referrers`, which implements part of the export pipeline or its metadata handling. | CN: 继续 `find_legit_leaks_from_referrers` 的实现，其作用是实现导出流水线或其元数据处理的一部分。
- **L96** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L97** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L98** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L99** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L100** EN: Continues `find_legit_leaks_from_referrers`, which implements part of the export pipeline or its metadata handling. | CN: 继续 `find_legit_leaks_from_referrers` 的实现，其作用是实现导出流水线或其元数据处理的一部分。
- **L101** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L102** EN: Continues `find_legit_leaks_from_referrers`, which implements part of the export pipeline or its metadata handling. | CN: 继续 `find_legit_leaks_from_referrers` 的实现，其作用是实现导出流水线或其元数据处理的一部分。
- **L103** EN: Assigns or updates `flagged`. | CN: 对 `flagged` 进行赋值或更新。
- **L104** EN: Continues `find_legit_leaks_from_referrers`, which implements part of the export pipeline or its metadata handling. | CN: 继续 `find_legit_leaks_from_referrers` 的实现，其作用是实现导出流水线或其元数据处理的一部分。
- **L105** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 106-113 / 第 106-113 行

````python
0106:             # Any other referrer we don't explicitly whitelist counts as a leak
0107:             flagged = True
0108:             break
0109: 
0110:         if flagged:
0111:             legit_leak.add(act)
0112: 
0113:     return legit_leak
````

- **L106** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L107** EN: Assigns or updates `flagged`. | CN: 对 `flagged` 进行赋值或更新。
- **L108** EN: Continues `find_legit_leaks_from_referrers`, which implements part of the export pipeline or its metadata handling. | CN: 继续 `find_legit_leaks_from_referrers` 的实现，其作用是实现导出流水线或其元数据处理的一部分。
- **L109** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L110** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L111** EN: Invokes `legit_leak.add` to advance the surrounding implementation. | CN: 调用 `legit_leak.add` 来推进周围的实现逻辑。
- **L112** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L113** EN: Returns from `find_legit_leaks_from_referrers` with the computed result or updated state. | CN: 从 `find_legit_leaks_from_referrers` 返回计算结果或更新后的状态。

## Key Concepts / 关键概念

- **EN**: Public export API — The module shapes the user-facing API for exporting and manipulating exported programs.
  **CN**: Public export API——该模块塑造了导出与操作导出程序的用户侧 API。
- **EN**: Exported program model — It often defines the data structures that carry graphs, signatures, state, and constraints.
  **CN**: Exported program model——它常常定义承载图、签名、状态与约束的数据结构。
- **EN**: Shape/spec reasoning — Dynamic-shape specifications and argument metadata are important for correctness.
  **CN**: Shape/spec reasoning——动态形状规格与参数元数据对正确性非常重要。
- **EN**: FX graphs — FX graph modules or nodes are central to the implementation.
  **CN**: FX graphs——FX 图模块或节点是实现的核心。
- **EN**: Primary callable `_is_globals_or_locals` — this routine is one of the main entry points in the module.
  **CN**: 核心可调用对象 `_is_globals_or_locals`——该例程是本模块的主要入口之一。
- **EN**: Primary callable `_is_tracked_fake` — this routine is one of the main entry points in the module.
  **CN**: 核心可调用对象 `_is_tracked_fake`——该例程是本模块的主要入口之一。

## Dependencies / 依赖关系

- **Torch imports / Torch 导入**: `torch.fx.experimental.symbolic_shapes:TrackedFake`
- **Other imports / 其他导入**: `gc`、`types`、`typing`、`weakref`、`typing_extensions:TypeIs`
- **Top-level classes / 顶层类**: 无
- **Top-level functions / 顶层函数**: `_is_globals_or_locals`、`_is_tracked_fake`、`_is_gm_meta_like_dict`、`_dict_is_attr_of_tracked_fake`、`find_legit_leaks_from_referrers`
- **Base classes / 基类**: 无
- **Decorators / 装饰器**: 无
- **Module assignments / 模块级赋值**: `_SKIP_TYPES`
