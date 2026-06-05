# memref.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/python/mlir/dialects/transform/memref.py`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Provides Python bindings and helper APIs for the MLIR transform dialect.
  - **CN**: 提供 MLIR Transform Dialect 的 Python 绑定与辅助 API。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12 / 第 1-12 行

````python
   1 | #  Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   2 | #  See https://llvm.org/LICENSE.txt for license information.
   3 | #  SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   4 | 
   5 | from .._memref_transform_ops_gen import *
   6 | from .._memref_transform_ops_gen import _Dialect
   7 | 
   8 | try:
   9 |     from ...ir import *
  10 |     from ...dialects import transform
  11 |     from .._ods_common import _cext as _ods_cext
  12 | except ImportError as e:
````
- **L1 EN**: Comment documents nearby Python logic: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L1 CN**: 注释说明附近的 Python 逻辑：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L2 EN**: Comment documents nearby Python logic: `See https://llvm.org/LICENSE.txt for license information.`.
  **L2 CN**: 注释说明附近的 Python 逻辑：`See https://llvm.org/LICENSE.txt for license information.`。
- **L3 EN**: Comment documents nearby Python logic: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L3 CN**: 注释说明附近的 Python 逻辑：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L4 EN**: Blank line separating nearby declarations or logic blocks.
  **L4 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L5 EN**: Imports selected names from module `.._memref_transform_ops_gen`.
  **L5 CN**: 从模块 `.._memref_transform_ops_gen` 中导入指定名称。
- **L6 EN**: Imports selected names from module `.._memref_transform_ops_gen`.
  **L6 CN**: 从模块 `.._memref_transform_ops_gen` 中导入指定名称。
- **L7 EN**: Blank line separating nearby declarations or logic blocks.
  **L7 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L8 EN**: Starts a Python control-flow or context-management clause: `try:`.
  **L8 CN**: 开始一条 Python 控制流或上下文管理子句：`try:`。
- **L9 EN**: Imports selected names from module `...ir`.
  **L9 CN**: 从模块 `...ir` 中导入指定名称。
- **L10 EN**: Imports selected names from module `...dialects`.
  **L10 CN**: 从模块 `...dialects` 中导入指定名称。
- **L11 EN**: Imports selected names from module `.._ods_common`.
  **L11 CN**: 从模块 `.._ods_common` 中导入指定名称。
- **L12 EN**: Starts a Python control-flow or context-management clause: `except ImportError as e:`.
  **L12 CN**: 开始一条 Python 控制流或上下文管理子句：`except ImportError as e:`。

### Lines 13-24 / 第 13-24 行

````python
  13 |     raise RuntimeError("Error loading imports from extension module") from e
  14 | 
  15 | from typing import Optional, overload, Union
  16 | 
  17 | 
  18 | @_ods_cext.register_operation(_Dialect, replace=True)
  19 | class MemRefAllocaToGlobalOp(MemRefAllocaToGlobalOp):
  20 |     """Specialization for MemRefAllocaToGlobalOp class."""
  21 | 
  22 |     @overload
  23 |     def __init__(
  24 |         self,
````
- **L13 EN**: Executes a Python control statement: `raise RuntimeError("Error loading imports from extension module") from e`.
  **L13 CN**: 执行一条 Python 控制语句：`raise RuntimeError("Error loading imports from extension module") from e`。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L15 EN**: Imports selected names from module `typing`.
  **L15 CN**: 从模块 `typing` 中导入指定名称。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L17 EN**: Blank line separating nearby declarations or logic blocks.
  **L17 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L18 EN**: Applies decorator `@_ods_cext.register_operation(_Dialect, replace=True)` to the next definition.
  **L18 CN**: 将装饰器 `@_ods_cext.register_operation(_Dialect, replace=True)` 应用于后续定义。
- **L19 EN**: Declares Python class `MemRefAllocaToGlobalOp`.
  **L19 CN**: 声明 Python 类 `MemRefAllocaToGlobalOp`。
- **L20 EN**: Participates in a module, class, or function docstring: `"""Specialization for MemRefAllocaToGlobalOp class."""`.
  **L20 CN**: 参与模块、类或函数的 docstring：`"""Specialization for MemRefAllocaToGlobalOp class."""`。
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L22 EN**: Applies decorator `@overload` to the next definition.
  **L22 CN**: 将装饰器 `@overload` 应用于后续定义。
- **L23 EN**: Defines function `__init__`.
  **L23 CN**: 定义函数 `__init__`。
- **L24 EN**: Executes Python statement `self,`.
  **L24 CN**: 执行 Python 语句 `self,`。

### Lines 25-36 / 第 25-36 行

````python
  25 |         get_global_type: Type,
  26 |         global_type: Type,
  27 |         alloca: Union[Operation, OpView, Value],
  28 |         *,
  29 |         loc=None,
  30 |         ip=None,
  31 |     ):
  32 |         ...
  33 | 
  34 |     @overload
  35 |     def __init__(self, alloca: Union[Operation, OpView, Value], *, loc=None, ip=None):
  36 |         ...
````
- **L25 EN**: Executes Python statement `get_global_type: Type,`.
  **L25 CN**: 执行 Python 语句 `get_global_type: Type,`。
- **L26 EN**: Executes Python statement `global_type: Type,`.
  **L26 CN**: 执行 Python 语句 `global_type: Type,`。
- **L27 EN**: Executes Python statement `alloca: Union[Operation, OpView, Value],`.
  **L27 CN**: 执行 Python 语句 `alloca: Union[Operation, OpView, Value],`。
- **L28 EN**: Executes Python statement `*,`.
  **L28 CN**: 执行 Python 语句 `*,`。
- **L29 EN**: Assigns or updates `loc`.
  **L29 CN**: 对 `loc` 进行赋值或更新。
- **L30 EN**: Assigns or updates `ip`.
  **L30 CN**: 对 `ip` 进行赋值或更新。
- **L31 EN**: Executes Python statement `):`.
  **L31 CN**: 执行 Python 语句 `):`。
- **L32 EN**: Executes Python statement `...`.
  **L32 CN**: 执行 Python 语句 `...`。
- **L33 EN**: Blank line separating nearby declarations or logic blocks.
  **L33 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L34 EN**: Applies decorator `@overload` to the next definition.
  **L34 CN**: 将装饰器 `@overload` 应用于后续定义。
- **L35 EN**: Defines function `__init__`.
  **L35 CN**: 定义函数 `__init__`。
- **L36 EN**: Executes Python statement `...`.
  **L36 CN**: 执行 Python 语句 `...`。

### Lines 37-48 / 第 37-48 行

````python
  37 | 
  38 |     def __init__(
  39 |         self,
  40 |         get_global_type_or_alloca: Union[Operation, OpView, Type, Value],
  41 |         global_type_or_none: Optional[Type] = None,
  42 |         alloca_or_none: Optional[Union[Operation, OpView, Value]] = None,
  43 |         *,
  44 |         loc=None,
  45 |         ip=None,
  46 |     ):
  47 |         if isinstance(get_global_type_or_alloca, Type):
  48 |             get_global_type = get_global_type_or_alloca
````
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L38 EN**: Defines function `__init__`.
  **L38 CN**: 定义函数 `__init__`。
- **L39 EN**: Executes Python statement `self,`.
  **L39 CN**: 执行 Python 语句 `self,`。
- **L40 EN**: Executes Python statement `get_global_type_or_alloca: Union[Operation, OpView, Type, Value],`.
  **L40 CN**: 执行 Python 语句 `get_global_type_or_alloca: Union[Operation, OpView, Type, Value],`。
- **L41 EN**: Executes Python statement `global_type_or_none: Optional[Type] = None,`.
  **L41 CN**: 执行 Python 语句 `global_type_or_none: Optional[Type] = None,`。
- **L42 EN**: Executes Python statement `alloca_or_none: Optional[Union[Operation, OpView, Value]] = None,`.
  **L42 CN**: 执行 Python 语句 `alloca_or_none: Optional[Union[Operation, OpView, Value]] = None,`。
- **L43 EN**: Executes Python statement `*,`.
  **L43 CN**: 执行 Python 语句 `*,`。
- **L44 EN**: Assigns or updates `loc`.
  **L44 CN**: 对 `loc` 进行赋值或更新。
- **L45 EN**: Assigns or updates `ip`.
  **L45 CN**: 对 `ip` 进行赋值或更新。
- **L46 EN**: Executes Python statement `):`.
  **L46 CN**: 执行 Python 语句 `):`。
- **L47 EN**: Starts a Python control-flow or context-management clause: `if isinstance(get_global_type_or_alloca, Type):`.
  **L47 CN**: 开始一条 Python 控制流或上下文管理子句：`if isinstance(get_global_type_or_alloca, Type):`。
- **L48 EN**: Assigns or updates `get_global_type`.
  **L48 CN**: 对 `get_global_type` 进行赋值或更新。

### Lines 49-60 / 第 49-60 行

````python
  49 |             global_type = global_type_or_none
  50 |             alloca = alloca_or_none
  51 |         else:
  52 |             get_global_type = transform.AnyOpType.get()
  53 |             global_type = transform.AnyOpType.get()
  54 |             alloca = get_global_type_or_alloca
  55 | 
  56 |         super().__init__(
  57 |             get_global_type,
  58 |             global_type,
  59 |             alloca,
  60 |             loc=loc,
````
- **L49 EN**: Assigns or updates `global_type`.
  **L49 CN**: 对 `global_type` 进行赋值或更新。
- **L50 EN**: Assigns or updates `alloca`.
  **L50 CN**: 对 `alloca` 进行赋值或更新。
- **L51 EN**: Starts the fallback branch for the preceding conditional.
  **L51 CN**: 开始前一个条件结构的兜底分支。
- **L52 EN**: Assigns or updates `get_global_type`.
  **L52 CN**: 对 `get_global_type` 进行赋值或更新。
- **L53 EN**: Assigns or updates `global_type`.
  **L53 CN**: 对 `global_type` 进行赋值或更新。
- **L54 EN**: Assigns or updates `alloca`.
  **L54 CN**: 对 `alloca` 进行赋值或更新。
- **L55 EN**: Blank line separating nearby declarations or logic blocks.
  **L55 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L56 EN**: Executes Python statement `super().__init__(`.
  **L56 CN**: 执行 Python 语句 `super().__init__(`。
- **L57 EN**: Executes Python statement `get_global_type,`.
  **L57 CN**: 执行 Python 语句 `get_global_type,`。
- **L58 EN**: Executes Python statement `global_type,`.
  **L58 CN**: 执行 Python 语句 `global_type,`。
- **L59 EN**: Executes Python statement `alloca,`.
  **L59 CN**: 执行 Python 语句 `alloca,`。
- **L60 EN**: Assigns or updates `loc`.
  **L60 CN**: 对 `loc` 进行赋值或更新。

### Lines 61-72 / 第 61-72 行

````python
  61 |             ip=ip,
  62 |         )
  63 | 
  64 | 
  65 | @_ods_cext.register_operation(_Dialect, replace=True)
  66 | class MemRefMultiBufferOp(MemRefMultiBufferOp):
  67 |     """Specialization for MemRefMultiBufferOp class."""
  68 | 
  69 |     @overload
  70 |     def __init__(
  71 |         self,
  72 |         transformed_type: Type,
````
- **L61 EN**: Assigns or updates `ip`.
  **L61 CN**: 对 `ip` 进行赋值或更新。
- **L62 EN**: Executes Python statement `)`.
  **L62 CN**: 执行 Python 语句 `)`。
- **L63 EN**: Blank line separating nearby declarations or logic blocks.
  **L63 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L64 EN**: Blank line separating nearby declarations or logic blocks.
  **L64 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L65 EN**: Applies decorator `@_ods_cext.register_operation(_Dialect, replace=True)` to the next definition.
  **L65 CN**: 将装饰器 `@_ods_cext.register_operation(_Dialect, replace=True)` 应用于后续定义。
- **L66 EN**: Declares Python class `MemRefMultiBufferOp`.
  **L66 CN**: 声明 Python 类 `MemRefMultiBufferOp`。
- **L67 EN**: Participates in a module, class, or function docstring: `"""Specialization for MemRefMultiBufferOp class."""`.
  **L67 CN**: 参与模块、类或函数的 docstring：`"""Specialization for MemRefMultiBufferOp class."""`。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L69 EN**: Applies decorator `@overload` to the next definition.
  **L69 CN**: 将装饰器 `@overload` 应用于后续定义。
- **L70 EN**: Defines function `__init__`.
  **L70 CN**: 定义函数 `__init__`。
- **L71 EN**: Executes Python statement `self,`.
  **L71 CN**: 执行 Python 语句 `self,`。
- **L72 EN**: Executes Python statement `transformed_type: Type,`.
  **L72 CN**: 执行 Python 语句 `transformed_type: Type,`。

### Lines 73-84 / 第 73-84 行

````python
  73 |         target: Union[Operation, OpView, Value],
  74 |         factor: Union[int, IntegerAttr],
  75 |         *,
  76 |         skip_analysis: Optional[bool] = None,
  77 |         loc=None,
  78 |         ip=None,
  79 |     ):
  80 |         ...
  81 | 
  82 |     @overload
  83 |     def __init__(
  84 |         self,
````
- **L73 EN**: Executes Python statement `target: Union[Operation, OpView, Value],`.
  **L73 CN**: 执行 Python 语句 `target: Union[Operation, OpView, Value],`。
- **L74 EN**: Executes Python statement `factor: Union[int, IntegerAttr],`.
  **L74 CN**: 执行 Python 语句 `factor: Union[int, IntegerAttr],`。
- **L75 EN**: Executes Python statement `*,`.
  **L75 CN**: 执行 Python 语句 `*,`。
- **L76 EN**: Executes Python statement `skip_analysis: Optional[bool] = None,`.
  **L76 CN**: 执行 Python 语句 `skip_analysis: Optional[bool] = None,`。
- **L77 EN**: Assigns or updates `loc`.
  **L77 CN**: 对 `loc` 进行赋值或更新。
- **L78 EN**: Assigns or updates `ip`.
  **L78 CN**: 对 `ip` 进行赋值或更新。
- **L79 EN**: Executes Python statement `):`.
  **L79 CN**: 执行 Python 语句 `):`。
- **L80 EN**: Executes Python statement `...`.
  **L80 CN**: 执行 Python 语句 `...`。
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L82 EN**: Applies decorator `@overload` to the next definition.
  **L82 CN**: 将装饰器 `@overload` 应用于后续定义。
- **L83 EN**: Defines function `__init__`.
  **L83 CN**: 定义函数 `__init__`。
- **L84 EN**: Executes Python statement `self,`.
  **L84 CN**: 执行 Python 语句 `self,`。

### Lines 85-96 / 第 85-96 行

````python
  85 |         target: Union[Operation, OpView, Value],
  86 |         factor: Union[int, IntegerAttr],
  87 |         *,
  88 |         skip_analysis: Optional[bool] = None,
  89 |         loc=None,
  90 |         ip=None,
  91 |     ):
  92 |         ...
  93 | 
  94 |     def __init__(
  95 |         self,
  96 |         transformed_type_or_target: Type,
````
- **L85 EN**: Executes Python statement `target: Union[Operation, OpView, Value],`.
  **L85 CN**: 执行 Python 语句 `target: Union[Operation, OpView, Value],`。
- **L86 EN**: Executes Python statement `factor: Union[int, IntegerAttr],`.
  **L86 CN**: 执行 Python 语句 `factor: Union[int, IntegerAttr],`。
- **L87 EN**: Executes Python statement `*,`.
  **L87 CN**: 执行 Python 语句 `*,`。
- **L88 EN**: Executes Python statement `skip_analysis: Optional[bool] = None,`.
  **L88 CN**: 执行 Python 语句 `skip_analysis: Optional[bool] = None,`。
- **L89 EN**: Assigns or updates `loc`.
  **L89 CN**: 对 `loc` 进行赋值或更新。
- **L90 EN**: Assigns or updates `ip`.
  **L90 CN**: 对 `ip` 进行赋值或更新。
- **L91 EN**: Executes Python statement `):`.
  **L91 CN**: 执行 Python 语句 `):`。
- **L92 EN**: Executes Python statement `...`.
  **L92 CN**: 执行 Python 语句 `...`。
- **L93 EN**: Blank line separating nearby declarations or logic blocks.
  **L93 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L94 EN**: Defines function `__init__`.
  **L94 CN**: 定义函数 `__init__`。
- **L95 EN**: Executes Python statement `self,`.
  **L95 CN**: 执行 Python 语句 `self,`。
- **L96 EN**: Executes Python statement `transformed_type_or_target: Type,`.
  **L96 CN**: 执行 Python 语句 `transformed_type_or_target: Type,`。

### Lines 97-108 / 第 97-108 行

````python
  97 |         target_or_factor: Union[int, IntegerAttr, Operation, OpView, Value] = None,
  98 |         factor_or_none: Optional[Union[int, IntegerAttr]] = None,
  99 |         *,
 100 |         skip_analysis: Optional[bool] = None,
 101 |         loc=None,
 102 |         ip=None,
 103 |     ):
 104 |         if isinstance(transformed_type_or_target, Type):
 105 |             transformed_type = transformed_type_or_target
 106 |             target = target_or_factor
 107 |             factor = factor_or_none
 108 |         else:
````
- **L97 EN**: Executes Python statement `target_or_factor: Union[int, IntegerAttr, Operation, OpView, Value] = None,`.
  **L97 CN**: 执行 Python 语句 `target_or_factor: Union[int, IntegerAttr, Operation, OpView, Value] = None,`。
- **L98 EN**: Executes Python statement `factor_or_none: Optional[Union[int, IntegerAttr]] = None,`.
  **L98 CN**: 执行 Python 语句 `factor_or_none: Optional[Union[int, IntegerAttr]] = None,`。
- **L99 EN**: Executes Python statement `*,`.
  **L99 CN**: 执行 Python 语句 `*,`。
- **L100 EN**: Executes Python statement `skip_analysis: Optional[bool] = None,`.
  **L100 CN**: 执行 Python 语句 `skip_analysis: Optional[bool] = None,`。
- **L101 EN**: Assigns or updates `loc`.
  **L101 CN**: 对 `loc` 进行赋值或更新。
- **L102 EN**: Assigns or updates `ip`.
  **L102 CN**: 对 `ip` 进行赋值或更新。
- **L103 EN**: Executes Python statement `):`.
  **L103 CN**: 执行 Python 语句 `):`。
- **L104 EN**: Starts a Python control-flow or context-management clause: `if isinstance(transformed_type_or_target, Type):`.
  **L104 CN**: 开始一条 Python 控制流或上下文管理子句：`if isinstance(transformed_type_or_target, Type):`。
- **L105 EN**: Assigns or updates `transformed_type`.
  **L105 CN**: 对 `transformed_type` 进行赋值或更新。
- **L106 EN**: Assigns or updates `target`.
  **L106 CN**: 对 `target` 进行赋值或更新。
- **L107 EN**: Assigns or updates `factor`.
  **L107 CN**: 对 `factor` 进行赋值或更新。
- **L108 EN**: Starts the fallback branch for the preceding conditional.
  **L108 CN**: 开始前一个条件结构的兜底分支。

### Lines 109-120 / 第 109-120 行

````python
 109 |             transformed_type = transform.AnyOpType.get()
 110 |             target = transformed_type_or_target
 111 |             factor = target_or_factor
 112 | 
 113 |         super().__init__(
 114 |             transformed_type,
 115 |             target,
 116 |             factor,
 117 |             skip_analysis=skip_analysis,
 118 |             loc=loc,
 119 |             ip=ip,
 120 |         )
````
- **L109 EN**: Assigns or updates `transformed_type`.
  **L109 CN**: 对 `transformed_type` 进行赋值或更新。
- **L110 EN**: Assigns or updates `target`.
  **L110 CN**: 对 `target` 进行赋值或更新。
- **L111 EN**: Assigns or updates `factor`.
  **L111 CN**: 对 `factor` 进行赋值或更新。
- **L112 EN**: Blank line separating nearby declarations or logic blocks.
  **L112 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L113 EN**: Executes Python statement `super().__init__(`.
  **L113 CN**: 执行 Python 语句 `super().__init__(`。
- **L114 EN**: Executes Python statement `transformed_type,`.
  **L114 CN**: 执行 Python 语句 `transformed_type,`。
- **L115 EN**: Executes Python statement `target,`.
  **L115 CN**: 执行 Python 语句 `target,`。
- **L116 EN**: Executes Python statement `factor,`.
  **L116 CN**: 执行 Python 语句 `factor,`。
- **L117 EN**: Assigns or updates `skip_analysis`.
  **L117 CN**: 对 `skip_analysis` 进行赋值或更新。
- **L118 EN**: Assigns or updates `loc`.
  **L118 CN**: 对 `loc` 进行赋值或更新。
- **L119 EN**: Assigns or updates `ip`.
  **L119 CN**: 对 `ip` 进行赋值或更新。
- **L120 EN**: Executes Python statement `)`.
  **L120 CN**: 执行 Python 语句 `)`。

## Key Concepts / 关键概念

- **Python bindings / Python 绑定**:
  - **EN**: Bridges MLIR concepts into Python classes, helpers, and user-facing APIs.
  - **CN**: 将 MLIR 概念桥接为 Python 类、辅助逻辑与面向用户的 API。
- **Python dialect bindings / Python 方言绑定**:
  - **EN**: Exposes MLIR dialect operations, attributes, or enums through Python-friendly wrapper classes.
  - **CN**: 通过 Python 友好的包装类暴露 MLIR 方言操作、属性或枚举。
- **Operation specialization / 操作特化**:
  - **EN**: Replaces or extends generated operation classes with handwritten Python behavior.
  - **CN**: 使用手写 Python 行为替换或扩展生成的操作类。
- **ODS helper integration / ODS 辅助逻辑集成**:
  - **EN**: Relies on generated ODS support helpers for operand/result conversion and registration.
  - **CN**: 依赖生成的 ODS 辅助逻辑来完成操作数/结果转换与注册。
- **Python automation / Python 自动化**:
  - **EN**: Uses Python to orchestrate MLIR construction, registration, execution, or developer utilities.
  - **CN**: 使用 Python 编排 MLIR 的构造、注册、执行或开发者辅助流程。
- **Library composition / 库组合**:
  - **EN**: Builds behavior by composing generated artifacts, MLIR libraries, or Python modules.
  - **CN**: 通过组合生成工件、MLIR 库或 Python 模块来构建行为。

## Dependencies / 依赖关系

- **Imported modules / 导入模块**: `.._memref_transform_ops_gen`, `...ir`, `...dialects`, `.._ods_common`, `typing`
- **Generated/local binding modules / 生成或本地绑定模块**: `.._memref_transform_ops_gen`, `...ir`, `...dialects`, `.._ods_common`
