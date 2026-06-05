# BuiltinsARM.def — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/BuiltinsARM.def`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: ARM Builtin function database *- C++.
- **Purpose (CN)**: 声明与 `BuiltinsARM` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 346

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===--- BuiltinsARM.def - ARM Builtin function database ----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the ARM-specific builtin function database.  Users of
// this file must define the BUILTIN macro to make use of this information.
//
//===----------------------------------------------------------------------===//

// The format of this database matches clang/Basic/Builtins.def.

#if defined(BUILTIN) && !defined(LANGBUILTIN)
#   define LANGBUILTIN(ID, TYPE, ATTRS, BUILTIN_LANG) BUILTIN(ID, TYPE, ATTRS)
#endif

#if defined(BUILTIN) && !defined(TARGET_BUILTIN)
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, constraints, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, constraints, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释解释附近代码的逻辑、约束或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, constraints, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the ARM-specific builtin function database. Users of`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the ARM-specific builtin function database. Users of`。
- **L10 EN**: Comment explains nearby logic, constraints, or intent: `this file must define the BUILTIN macro to make use of this information.`.
  **L10 CN**: 注释解释附近代码的逻辑、约束或设计意图：`this file must define the BUILTIN macro to make use of this information.`。
- **L11 EN**: Separator comment used for visual grouping.
  **L11 CN**: 用于视觉分组的分隔注释。
- **L12 EN**: Banner comment marking a file or section boundary.
  **L12 CN**: 横幅注释，用于标记文件或章节边界。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L14 EN**: Comment explains nearby logic, constraints, or intent: `The format of this database matches clang/Basic/Builtins.def.`.
  **L14 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The format of this database matches clang/Basic/Builtins.def.`。
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L16 EN**: Starts a preprocessor conditional block: `#if defined(BUILTIN) && !defined(LANGBUILTIN)`.
  **L16 CN**: 开始一个预处理条件块：`#if defined(BUILTIN) && !defined(LANGBUILTIN)`。
- **L17 EN**: Continues logic associated with callable symbol `LANGBUILTIN`.
  **L17 CN**: 继续与可调用符号 `LANGBUILTIN` 相关的逻辑。
- **L18 EN**: Closes the current preprocessor conditional block.
  **L18 CN**: 结束当前预处理条件块。
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L20 EN**: Starts a preprocessor conditional block: `#if defined(BUILTIN) && !defined(TARGET_BUILTIN)`.
  **L20 CN**: 开始一个预处理条件块：`#if defined(BUILTIN) && !defined(TARGET_BUILTIN)`。

### Lines 21-40

````cpp
#  define TARGET_BUILTIN(ID, TYPE, ATTRS, FEATURE) BUILTIN(ID, TYPE, ATTRS)
#endif

#if defined(BUILTIN) && !defined(TARGET_HEADER_BUILTIN)
#  define TARGET_HEADER_BUILTIN(ID, TYPE, ATTRS, HEADER, LANG, FEATURE) BUILTIN(ID, TYPE, ATTRS)
#endif

// In libgcc
BUILTIN(__clear_cache, "vv*v*", "i")

// 16-bit multiplications
BUILTIN(__builtin_arm_smulbb, "iii", "nc")
BUILTIN(__builtin_arm_smulbt, "iii", "nc")
BUILTIN(__builtin_arm_smultb, "iii", "nc")
BUILTIN(__builtin_arm_smultt, "iii", "nc")
BUILTIN(__builtin_arm_smulwb, "iii", "nc")
BUILTIN(__builtin_arm_smulwt, "iii", "nc")

// Saturating arithmetic
BUILTIN(__builtin_arm_qadd, "iii", "nc")
````
- **L21 EN**: Continues logic associated with callable symbol `TARGET_BUILTIN`.
  **L21 CN**: 继续与可调用符号 `TARGET_BUILTIN` 相关的逻辑。
- **L22 EN**: Closes the current preprocessor conditional block.
  **L22 CN**: 结束当前预处理条件块。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L24 EN**: Starts a preprocessor conditional block: `#if defined(BUILTIN) && !defined(TARGET_HEADER_BUILTIN)`.
  **L24 CN**: 开始一个预处理条件块：`#if defined(BUILTIN) && !defined(TARGET_HEADER_BUILTIN)`。
- **L25 EN**: Continues logic associated with callable symbol `TARGET_HEADER_BUILTIN`.
  **L25 CN**: 继续与可调用符号 `TARGET_HEADER_BUILTIN` 相关的逻辑。
- **L26 EN**: Closes the current preprocessor conditional block.
  **L26 CN**: 结束当前预处理条件块。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L28 EN**: Comment explains nearby logic, constraints, or intent: `In libgcc`.
  **L28 CN**: 注释解释附近代码的逻辑、约束或设计意图：`In libgcc`。
- **L29 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L29 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L31 EN**: Comment explains nearby logic, constraints, or intent: `16-bit multiplications`.
  **L31 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit multiplications`。
- **L32 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L32 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L33 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L33 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L34 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L34 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L35 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L35 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L36 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L36 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L37 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L37 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L38 EN**: Blank line separating nearby declarations or logic blocks.
  **L38 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L39 EN**: Comment explains nearby logic, constraints, or intent: `Saturating arithmetic`.
  **L39 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Saturating arithmetic`。
- **L40 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L40 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 41-60

````cpp
BUILTIN(__builtin_arm_qsub, "iii", "nc")
BUILTIN(__builtin_arm_qdbl, "ii", "nc")
BUILTIN(__builtin_arm_ssat, "iiUi", "nc")
BUILTIN(__builtin_arm_usat, "UiiUi", "nc")

BUILTIN(__builtin_arm_smlabb, "iiii", "nc")
BUILTIN(__builtin_arm_smlabt, "iiii", "nc")
BUILTIN(__builtin_arm_smlatb, "iiii", "nc")
BUILTIN(__builtin_arm_smlatt, "iiii", "nc")
BUILTIN(__builtin_arm_smlawb, "iiii", "nc")
BUILTIN(__builtin_arm_smlawt, "iiii", "nc")

BUILTIN(__builtin_arm_ssat16, "iii", "nc")
BUILTIN(__builtin_arm_usat16, "iii", "nc")

BUILTIN(__builtin_arm_sxtab16, "iii", "nc")
BUILTIN(__builtin_arm_sxtb16, "ii", "nc")
BUILTIN(__builtin_arm_uxtab16, "iii", "nc")
BUILTIN(__builtin_arm_uxtb16, "ii", "nc")

````
- **L41 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L41 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L42 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L42 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L43 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L43 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L44 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L44 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L45 EN**: Blank line separating nearby declarations or logic blocks.
  **L45 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L46 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L46 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L47 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L47 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L48 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L48 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L49 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L49 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L50 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L50 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L51 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L51 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L52 EN**: Blank line separating nearby declarations or logic blocks.
  **L52 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L53 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L53 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L54 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L54 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L55 EN**: Blank line separating nearby declarations or logic blocks.
  **L55 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L56 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L56 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L57 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L57 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L58 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L58 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L59 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L59 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L60 EN**: Blank line separating nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 61-80

````cpp
BUILTIN(__builtin_arm_sel, "iii", "nc")

BUILTIN(__builtin_arm_qadd8, "iii", "nc")
BUILTIN(__builtin_arm_qsub8, "iii", "nc")
BUILTIN(__builtin_arm_sadd8, "iii", "nc")
BUILTIN(__builtin_arm_shadd8, "iii", "nc")
BUILTIN(__builtin_arm_shsub8, "iii", "nc")
BUILTIN(__builtin_arm_ssub8, "iii", "nc")
BUILTIN(__builtin_arm_uadd8, "UiUiUi", "nc")
BUILTIN(__builtin_arm_uhadd8, "UiUiUi", "nc")
BUILTIN(__builtin_arm_uhsub8, "UiUiUi", "nc")
BUILTIN(__builtin_arm_uqadd8, "UiUiUi", "nc")
BUILTIN(__builtin_arm_uqsub8, "UiUiUi", "nc")
BUILTIN(__builtin_arm_usub8, "UiUiUi", "nc")

// Sum of 8-bit absolute differences
BUILTIN(__builtin_arm_usad8, "UiUiUi", "nc")
BUILTIN(__builtin_arm_usada8, "UiUiUiUi", "nc")

// Parallel 16-bit addition and subtraction
````
- **L61 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L61 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L63 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L63 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L64 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L64 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L65 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L65 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L66 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L66 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L67 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L67 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L68 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L68 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L69 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L69 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L70 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L70 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L71 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L71 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L72 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L72 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L73 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L73 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L74 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L74 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L75 EN**: Blank line separating nearby declarations or logic blocks.
  **L75 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L76 EN**: Comment explains nearby logic, constraints, or intent: `Sum of 8-bit absolute differences`.
  **L76 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sum of 8-bit absolute differences`。
- **L77 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L77 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L78 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L78 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L79 EN**: Blank line separating nearby declarations or logic blocks.
  **L79 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L80 EN**: Comment explains nearby logic, constraints, or intent: `Parallel 16-bit addition and subtraction`.
  **L80 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Parallel 16-bit addition and subtraction`。

### Lines 81-100

````cpp
BUILTIN(__builtin_arm_qadd16, "iii", "nc")
BUILTIN(__builtin_arm_qasx, "iii", "nc")
BUILTIN(__builtin_arm_qsax, "iii", "nc")
BUILTIN(__builtin_arm_qsub16, "iii", "nc")
BUILTIN(__builtin_arm_sadd16, "iii", "nc")
BUILTIN(__builtin_arm_sasx, "iii", "nc")
BUILTIN(__builtin_arm_shadd16, "iii", "nc")
BUILTIN(__builtin_arm_shasx, "iii", "nc")
BUILTIN(__builtin_arm_shsax, "iii", "nc")
BUILTIN(__builtin_arm_shsub16, "iii", "nc")
BUILTIN(__builtin_arm_ssax, "iii", "nc")
BUILTIN(__builtin_arm_ssub16, "iii", "nc")
BUILTIN(__builtin_arm_uadd16, "UiUiUi", "nc")
BUILTIN(__builtin_arm_uasx, "UiUiUi", "nc")
BUILTIN(__builtin_arm_uhadd16, "UiUiUi", "nc")
BUILTIN(__builtin_arm_uhasx, "UiUiUi", "nc")
BUILTIN(__builtin_arm_uhsax, "UiUiUi", "nc")
BUILTIN(__builtin_arm_uhsub16, "UiUiUi", "nc")
BUILTIN(__builtin_arm_uqadd16, "UiUiUi", "nc")
BUILTIN(__builtin_arm_uqasx, "UiUiUi", "nc")
````
- **L81 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L81 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L82 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L82 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L83 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L83 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L84 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L84 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L85 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L85 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L86 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L86 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L87 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L87 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L88 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L88 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L89 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L89 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L90 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L90 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L91 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L91 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L92 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L92 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L93 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L93 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L94 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L94 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L95 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L95 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L96 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L96 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L97 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L97 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L98 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L98 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L99 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L99 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L100 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L100 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 101-120

````cpp
BUILTIN(__builtin_arm_uqsax, "UiUiUi", "nc")
BUILTIN(__builtin_arm_uqsub16, "UiUiUi", "nc")
BUILTIN(__builtin_arm_usax, "UiUiUi", "nc")
BUILTIN(__builtin_arm_usub16, "UiUiUi", "nc")

// Parallel 16-bit multiplication
BUILTIN(__builtin_arm_smlad, "iiii", "nc")
BUILTIN(__builtin_arm_smladx, "iiii", "nc")
BUILTIN(__builtin_arm_smlald, "LLiiiLLi", "nc")
BUILTIN(__builtin_arm_smlaldx, "LLiiiLLi", "nc")
BUILTIN(__builtin_arm_smlsd, "iiii", "nc")
BUILTIN(__builtin_arm_smlsdx, "iiii", "nc")
BUILTIN(__builtin_arm_smlsld, "LLiiiLLi", "nc")
BUILTIN(__builtin_arm_smlsldx, "LLiiiLLi", "nc")
BUILTIN(__builtin_arm_smuad, "iii", "nc")
BUILTIN(__builtin_arm_smuadx, "iii", "nc")
BUILTIN(__builtin_arm_smusd, "iii", "nc")
BUILTIN(__builtin_arm_smusdx, "iii", "nc")

// Bit manipulation
````
- **L101 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L101 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L102 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L102 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L103 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L103 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L104 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L104 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L105 EN**: Blank line separating nearby declarations or logic blocks.
  **L105 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L106 EN**: Comment explains nearby logic, constraints, or intent: `Parallel 16-bit multiplication`.
  **L106 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Parallel 16-bit multiplication`。
- **L107 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L107 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L108 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L108 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L109 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L109 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L110 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L110 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L111 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L111 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L112 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L112 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L113 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L113 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L114 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L114 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L115 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L115 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L116 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L116 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L117 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L117 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L118 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L118 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L119 EN**: Blank line separating nearby declarations or logic blocks.
  **L119 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L120 EN**: Comment explains nearby logic, constraints, or intent: `Bit manipulation`.
  **L120 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Bit manipulation`。

### Lines 121-140

````cpp
BUILTIN(__builtin_arm_rbit, "UiUi", "nc")
BUILTIN(__builtin_arm_clz, "UiZUi", "nc")
BUILTIN(__builtin_arm_clz64, "UiWUi", "nc")
BUILTIN(__builtin_arm_cls, "UiZUi", "nc")
BUILTIN(__builtin_arm_cls64, "UiWUi", "nc")

// Store and load exclusive
BUILTIN(__builtin_arm_ldrexd, "v.", "t")
BUILTIN(__builtin_arm_strexd, "i.", "t")

BUILTIN(__builtin_arm_ldrex, "v.", "t")
BUILTIN(__builtin_arm_ldaex, "v.", "t")
BUILTIN(__builtin_arm_strex, "i.", "t")
BUILTIN(__builtin_arm_stlex, "i.", "t")
BUILTIN(__builtin_arm_clrex, "v", "")

// VFP
BUILTIN(__builtin_arm_get_fpscr, "Ui", "nc")
BUILTIN(__builtin_arm_set_fpscr, "vUi", "nc")
BUILTIN(__builtin_arm_vcvtr_f, "ffi", "nc")
````
- **L121 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L121 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L122 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L122 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L123 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L123 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L124 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L124 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L125 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L125 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L126 EN**: Blank line separating nearby declarations or logic blocks.
  **L126 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L127 EN**: Comment explains nearby logic, constraints, or intent: `Store and load exclusive`.
  **L127 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Store and load exclusive`。
- **L128 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L128 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L129 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L129 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L130 EN**: Blank line separating nearby declarations or logic blocks.
  **L130 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L131 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L131 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L132 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L132 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L133 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L133 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L134 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L134 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L135 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L135 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L136 EN**: Blank line separating nearby declarations or logic blocks.
  **L136 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L137 EN**: Comment explains nearby logic, constraints, or intent: `VFP`.
  **L137 CN**: 注释解释附近代码的逻辑、约束或设计意图：`VFP`。
- **L138 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L138 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L139 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L139 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L140 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L140 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 141-160

````cpp
BUILTIN(__builtin_arm_vcvtr_d, "fdi", "nc")

// Coprocessor
BUILTIN(__builtin_arm_ldc, "vUIiUIivC*", "")
BUILTIN(__builtin_arm_ldcl, "vUIiUIivC*", "")
BUILTIN(__builtin_arm_ldc2, "vUIiUIivC*", "")
BUILTIN(__builtin_arm_ldc2l, "vUIiUIivC*", "")

BUILTIN(__builtin_arm_stc, "vUIiUIiv*", "")
BUILTIN(__builtin_arm_stcl, "vUIiUIiv*", "")
BUILTIN(__builtin_arm_stc2, "vUIiUIiv*", "")
BUILTIN(__builtin_arm_stc2l, "vUIiUIiv*", "")

BUILTIN(__builtin_arm_cdp, "vUIiUIiUIiUIiUIiUIi", "")
BUILTIN(__builtin_arm_cdp2, "vUIiUIiUIiUIiUIiUIi", "")
BUILTIN(__builtin_arm_mcr, "vUIiUIiUiUIiUIiUIi", "")
BUILTIN(__builtin_arm_mcr2, "vUIiUIiUiUIiUIiUIi", "")
BUILTIN(__builtin_arm_mrc, "UiUIiUIiUIiUIiUIi", "")
BUILTIN(__builtin_arm_mrc2, "UiUIiUIiUIiUIiUIi", "")
BUILTIN(__builtin_arm_mcrr, "vUIiUIiLLUiUIi", "")
````
- **L141 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L141 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L142 EN**: Blank line separating nearby declarations or logic blocks.
  **L142 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L143 EN**: Comment explains nearby logic, constraints, or intent: `Coprocessor`.
  **L143 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Coprocessor`。
- **L144 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L144 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L145 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L145 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L146 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L146 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L147 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L147 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L148 EN**: Blank line separating nearby declarations or logic blocks.
  **L148 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L149 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L149 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L150 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L150 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L151 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L151 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L152 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L152 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L153 EN**: Blank line separating nearby declarations or logic blocks.
  **L153 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L154 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L154 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L155 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L155 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L156 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L156 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L157 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L157 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L158 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L158 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L159 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L159 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L160 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L160 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 161-180

````cpp
BUILTIN(__builtin_arm_mcrr2, "vUIiUIiLLUiUIi", "")
BUILTIN(__builtin_arm_mrrc, "LLUiUIiUIiUIi", "")
BUILTIN(__builtin_arm_mrrc2, "LLUiUIiUIiUIi", "")

// CRC32
TARGET_BUILTIN(__builtin_arm_crc32b, "UiUiUc", "nc", "crc")
TARGET_BUILTIN(__builtin_arm_crc32cb, "UiUiUc", "nc", "crc")
TARGET_BUILTIN(__builtin_arm_crc32h, "UiUiUs", "nc", "crc")
TARGET_BUILTIN(__builtin_arm_crc32ch, "UiUiUs", "nc", "crc")
TARGET_BUILTIN(__builtin_arm_crc32w, "UiUiUi", "nc", "crc")
TARGET_BUILTIN(__builtin_arm_crc32cw, "UiUiUi", "nc", "crc")
TARGET_BUILTIN(__builtin_arm_crc32d, "UiUiLLUi", "nc", "crc")
TARGET_BUILTIN(__builtin_arm_crc32cd, "UiUiLLUi", "nc", "crc")

// ARMv8-M Security Extensions a.k.a CMSE
BUILTIN(__builtin_arm_cmse_TT, "Uiv*", "n")
BUILTIN(__builtin_arm_cmse_TTT, "Uiv*", "n")
BUILTIN(__builtin_arm_cmse_TTA, "Uiv*", "n")
BUILTIN(__builtin_arm_cmse_TTAT, "Uiv*", "n")

````
- **L161 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L161 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L162 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L162 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L163 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L163 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L165 EN**: Comment explains nearby logic, constraints, or intent: `CRC32`.
  **L165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`CRC32`。
- **L166 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L166 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L167 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L167 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L168 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L168 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L169 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L169 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L170 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L170 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L171 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L171 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L172 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L172 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L173 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L173 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L174 EN**: Blank line separating nearby declarations or logic blocks.
  **L174 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L175 EN**: Comment explains nearby logic, constraints, or intent: `ARMv8-M Security Extensions a.k.a CMSE`.
  **L175 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ARMv8-M Security Extensions a.k.a CMSE`。
- **L176 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L176 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L177 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L177 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L178 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L178 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L179 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L179 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L180 EN**: Blank line separating nearby declarations or logic blocks.
  **L180 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 181-200

````cpp
// HINT
BUILTIN(__builtin_arm_nop, "v", "")
BUILTIN(__builtin_arm_yield, "v", "")
BUILTIN(__builtin_arm_wfe, "v", "")
BUILTIN(__builtin_arm_wfi, "v", "")
BUILTIN(__builtin_arm_sev, "v", "")
BUILTIN(__builtin_arm_sevl, "v", "")
BUILTIN(__builtin_arm_dbg, "vUi", "")
TARGET_HEADER_BUILTIN(__yield, "v", "h", ARM_ACLE_H, ALL_LANGUAGES, "")
TARGET_HEADER_BUILTIN(__wfe, "v", "h", ARM_ACLE_H, ALL_LANGUAGES, "")
TARGET_HEADER_BUILTIN(__wfi, "v", "h", ARM_ACLE_H, ALL_LANGUAGES, "")
TARGET_HEADER_BUILTIN(__sev, "v", "h", ARM_ACLE_H, ALL_LANGUAGES, "")
TARGET_HEADER_BUILTIN(__sevl, "v", "h", ARM_ACLE_H, ALL_LANGUAGES, "")

// Data barrier
BUILTIN(__builtin_arm_dmb, "vUi", "nc")
BUILTIN(__builtin_arm_dsb, "vUi", "nc")
BUILTIN(__builtin_arm_isb, "vUi", "nc")
TARGET_HEADER_BUILTIN(__dmb, "vUi", "nch", ARM_ACLE_H, ALL_LANGUAGES, "")
TARGET_HEADER_BUILTIN(__dsb, "vUi", "nch", ARM_ACLE_H, ALL_LANGUAGES, "")
````
- **L181 EN**: Comment explains nearby logic, constraints, or intent: `HINT`.
  **L181 CN**: 注释解释附近代码的逻辑、约束或设计意图：`HINT`。
- **L182 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L182 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L183 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L183 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L184 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L184 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L185 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L185 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L186 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L186 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L187 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L187 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L188 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L188 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L189 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L189 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L190 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L190 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L191 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L191 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L192 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L192 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L193 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L193 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L194 EN**: Blank line separating nearby declarations or logic blocks.
  **L194 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L195 EN**: Comment explains nearby logic, constraints, or intent: `Data barrier`.
  **L195 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Data barrier`。
- **L196 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L196 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L197 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L197 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L198 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L198 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L199 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L199 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L200 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L200 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 201-220

````cpp
TARGET_HEADER_BUILTIN(__isb, "vUi", "nch", ARM_ACLE_H, ALL_LANGUAGES, "")

// Prefetch
BUILTIN(__builtin_arm_prefetch, "vvC*UiUi", "nc")

// System registers (ACLE)
BUILTIN(__builtin_arm_rsr, "UicC*", "nc")
BUILTIN(__builtin_arm_rsr64, "LLUicC*", "nc")
BUILTIN(__builtin_arm_rsrp, "v*cC*", "nc")
BUILTIN(__builtin_arm_wsr, "vcC*Ui", "nc")
BUILTIN(__builtin_arm_wsr64, "vcC*LLUi", "nc")
BUILTIN(__builtin_arm_wsrp, "vcC*vC*", "nc")

// Misc
BUILTIN(__builtin_sponentry, "v*", "c")

// MSVC
LANGBUILTIN(__emit, "vIUiC", "", ALL_MS_LANGUAGES)

LANGBUILTIN(__ldrexd, "WiWiCD*", "", ALL_MS_LANGUAGES)
````
- **L201 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L201 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L202 EN**: Blank line separating nearby declarations or logic blocks.
  **L202 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L203 EN**: Comment explains nearby logic, constraints, or intent: `Prefetch`.
  **L203 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Prefetch`。
- **L204 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L204 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L205 EN**: Blank line separating nearby declarations or logic blocks.
  **L205 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L206 EN**: Comment explains nearby logic, constraints, or intent: `System registers (ACLE)`.
  **L206 CN**: 注释解释附近代码的逻辑、约束或设计意图：`System registers (ACLE)`。
- **L207 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L207 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L208 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L208 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L209 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L209 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L210 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L210 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L211 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L211 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L212 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L212 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L213 EN**: Blank line separating nearby declarations or logic blocks.
  **L213 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L214 EN**: Comment explains nearby logic, constraints, or intent: `Misc`.
  **L214 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Misc`。
- **L215 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L215 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L216 EN**: Blank line separating nearby declarations or logic blocks.
  **L216 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L217 EN**: Comment explains nearby logic, constraints, or intent: `MSVC`.
  **L217 CN**: 注释解释附近代码的逻辑、约束或设计意图：`MSVC`。
- **L218 EN**: Invokes macro `LANGBUILTIN` to contribute one entry to a table-driven definition list.
  **L218 CN**: 调用宏 `LANGBUILTIN`，向表驱动定义列表贡献一个条目。
- **L219 EN**: Blank line separating nearby declarations or logic blocks.
  **L219 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L220 EN**: Invokes macro `LANGBUILTIN` to contribute one entry to a table-driven definition list.
  **L220 CN**: 调用宏 `LANGBUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 221-240

````cpp
LANGBUILTIN(_MoveFromCoprocessor, "UiIUiIUiIUiIUiIUi", "", ALL_MS_LANGUAGES)
LANGBUILTIN(_MoveFromCoprocessor2, "UiIUiIUiIUiIUiIUi", "", ALL_MS_LANGUAGES)
LANGBUILTIN(_MoveToCoprocessor, "vUiIUiIUiIUiIUiIUi", "", ALL_MS_LANGUAGES)
LANGBUILTIN(_MoveToCoprocessor2, "vUiIUiIUiIUiIUiIUi", "", ALL_MS_LANGUAGES)

TARGET_HEADER_BUILTIN(_BitScanForward, "UcUNi*UNi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_BitScanReverse, "UcUNi*UNi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_BitScanForward64, "UcUNi*ULLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_BitScanReverse64, "UcUNi*ULLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")

TARGET_HEADER_BUILTIN(_InterlockedAnd64,         "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedDecrement64,   "LLiLLiD*",    "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchange64,    "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchangeAdd64, "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchangeSub64, "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedIncrement64,   "LLiLLiD*",    "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedOr64,          "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedXor64,         "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")

TARGET_HEADER_BUILTIN(_InterlockedExchangeAdd_acq, "NiNiD*Ni", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
````
- **L221 EN**: Invokes macro `LANGBUILTIN` to contribute one entry to a table-driven definition list.
  **L221 CN**: 调用宏 `LANGBUILTIN`，向表驱动定义列表贡献一个条目。
- **L222 EN**: Invokes macro `LANGBUILTIN` to contribute one entry to a table-driven definition list.
  **L222 CN**: 调用宏 `LANGBUILTIN`，向表驱动定义列表贡献一个条目。
- **L223 EN**: Invokes macro `LANGBUILTIN` to contribute one entry to a table-driven definition list.
  **L223 CN**: 调用宏 `LANGBUILTIN`，向表驱动定义列表贡献一个条目。
- **L224 EN**: Invokes macro `LANGBUILTIN` to contribute one entry to a table-driven definition list.
  **L224 CN**: 调用宏 `LANGBUILTIN`，向表驱动定义列表贡献一个条目。
- **L225 EN**: Blank line separating nearby declarations or logic blocks.
  **L225 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L226 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L226 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L227 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L227 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L228 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L228 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L229 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L229 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L230 EN**: Blank line separating nearby declarations or logic blocks.
  **L230 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L231 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L231 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L232 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L232 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L233 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L233 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L234 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L234 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L235 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L235 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L236 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L236 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L237 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L237 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L238 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L238 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L239 EN**: Blank line separating nearby declarations or logic blocks.
  **L239 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L240 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L240 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 241-260

````cpp
TARGET_HEADER_BUILTIN(_InterlockedExchangeAdd_rel, "NiNiD*Ni", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchangeAdd_nf, "NiNiD*Ni", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchangeAdd8_acq, "ccD*c", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchangeAdd8_rel, "ccD*c", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchangeAdd8_nf, "ccD*c", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchangeAdd16_acq, "ssD*s", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchangeAdd16_rel, "ssD*s", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchangeAdd16_nf, "ssD*s", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchangeAdd64_acq, "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchangeAdd64_rel, "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchangeAdd64_nf, "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")

TARGET_HEADER_BUILTIN(_InterlockedExchange8_acq,  "ccD*c",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchange8_nf,   "ccD*c",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchange8_rel,  "ccD*c",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchange16_acq, "ssD*s",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchange16_nf,  "ssD*s",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchange16_rel, "ssD*s",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchange_acq,   "NiNiD*Ni",    "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchange_nf,    "NiNiD*Ni",    "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
````
- **L241 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L241 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L242 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L242 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L243 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L243 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L244 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L244 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L245 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L245 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L246 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L246 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L247 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L247 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L248 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L248 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L249 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L249 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L250 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L250 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L251 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L251 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L252 EN**: Blank line separating nearby declarations or logic blocks.
  **L252 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L253 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L253 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L254 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L254 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L255 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L255 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L256 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L256 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L257 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L257 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L258 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L258 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L259 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L259 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L260 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L260 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 261-280

````cpp
TARGET_HEADER_BUILTIN(_InterlockedExchange_rel,   "NiNiD*Ni",    "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchange64_acq, "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchange64_nf,  "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchange64_rel, "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchangePointer_acq, "v*v*D*v*","nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchangePointer_nf,  "v*v*D*v*","nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedExchangePointer_rel, "v*v*D*v*","nh", INTRIN_H, ALL_MS_LANGUAGES, "")

TARGET_HEADER_BUILTIN(_InterlockedCompareExchange8_acq,  "ccD*cc",         "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedCompareExchange8_nf,   "ccD*cc",         "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedCompareExchange8_rel,  "ccD*cc",         "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedCompareExchange16_acq, "ssD*ss",         "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedCompareExchange16_nf,  "ssD*ss",         "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedCompareExchange16_rel, "ssD*ss",         "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedCompareExchange_acq,   "NiNiD*NiNi",     "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedCompareExchange_nf,    "NiNiD*NiNi",     "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedCompareExchange_rel,   "NiNiD*NiNi",     "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedCompareExchange64_acq, "LLiLLiD*LLiLLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedCompareExchange64_nf,  "LLiLLiD*LLiLLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedCompareExchange64_rel, "LLiLLiD*LLiLLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
````
- **L261 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L261 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L262 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L262 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L263 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L263 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L264 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L264 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L265 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L265 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L266 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L266 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L267 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L267 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L268 EN**: Blank line separating nearby declarations or logic blocks.
  **L268 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L269 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L269 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L270 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L270 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L271 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L271 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L272 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L272 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L273 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L273 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L274 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L274 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L275 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L275 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L276 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L276 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L277 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L277 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L278 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L278 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L279 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L279 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L280 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L280 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 281-300

````cpp
TARGET_HEADER_BUILTIN(_InterlockedCompareExchangePointer_acq, "v*v*D*v*v*","nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedCompareExchangePointer_rel, "v*v*D*v*v*","nh", INTRIN_H, ALL_MS_LANGUAGES, "")

TARGET_HEADER_BUILTIN(_InterlockedOr8_acq,  "ccD*c",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedOr8_nf,   "ccD*c",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedOr8_rel,  "ccD*c",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedOr16_acq, "ssD*s",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedOr16_nf,  "ssD*s",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedOr16_rel, "ssD*s",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedOr_acq,   "NiNiD*Ni",    "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedOr_nf,    "NiNiD*Ni",    "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedOr_rel,   "NiNiD*Ni",    "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedOr64_acq, "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedOr64_nf,  "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedOr64_rel, "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")

TARGET_HEADER_BUILTIN(_InterlockedXor8_acq,  "ccD*c",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedXor8_nf,   "ccD*c",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedXor8_rel,  "ccD*c",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedXor16_acq, "ssD*s",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
````
- **L281 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L281 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L282 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L282 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L283 EN**: Blank line separating nearby declarations or logic blocks.
  **L283 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L284 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L284 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L285 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L285 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L286 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L286 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L287 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L287 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L288 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L288 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L289 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L289 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L290 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L290 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L291 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L291 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L292 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L292 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L293 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L293 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L294 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L294 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L295 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L295 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L296 EN**: Blank line separating nearby declarations or logic blocks.
  **L296 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L297 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L297 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L298 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L298 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L299 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L299 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L300 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L300 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 301-320

````cpp
TARGET_HEADER_BUILTIN(_InterlockedXor16_nf,  "ssD*s",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedXor16_rel, "ssD*s",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedXor_acq,   "NiNiD*Ni",    "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedXor_nf,    "NiNiD*Ni",    "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedXor_rel,   "NiNiD*Ni",    "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedXor64_acq, "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedXor64_nf,  "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedXor64_rel, "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")

TARGET_HEADER_BUILTIN(_InterlockedAnd8_acq,  "ccD*c",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedAnd8_nf,   "ccD*c",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedAnd8_rel,  "ccD*c",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedAnd16_acq, "ssD*s",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedAnd16_nf,  "ssD*s",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedAnd16_rel, "ssD*s",       "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedAnd_acq,   "NiNiD*Ni",    "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedAnd_nf,    "NiNiD*Ni",    "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedAnd_rel,   "NiNiD*Ni",    "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedAnd64_acq, "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedAnd64_nf,  "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
````
- **L301 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L301 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L302 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L302 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L303 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L303 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L304 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L304 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L305 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L305 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L306 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L306 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L307 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L307 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L308 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L308 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L310 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L310 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L311 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L311 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L312 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L312 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L313 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L313 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L314 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L314 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L315 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L315 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L316 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L316 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L317 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L317 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L318 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L318 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L319 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L319 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L320 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L320 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 321-340

````cpp
TARGET_HEADER_BUILTIN(_InterlockedAnd64_rel, "LLiLLiD*LLi", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")

TARGET_HEADER_BUILTIN(_InterlockedIncrement16_acq, "ssD*",     "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedIncrement16_nf,  "ssD*",     "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedIncrement16_rel, "ssD*",     "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedIncrement_acq,   "NiNiD*",   "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedIncrement_nf,    "NiNiD*",   "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedIncrement_rel,   "NiNiD*",   "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedIncrement64_acq, "LLiLLiD*", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedIncrement64_nf,  "LLiLLiD*", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedIncrement64_rel, "LLiLLiD*", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")

TARGET_HEADER_BUILTIN(_InterlockedDecrement16_acq, "ssD*",     "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedDecrement16_nf,  "ssD*",     "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedDecrement16_rel, "ssD*",     "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedDecrement_acq,   "NiNiD*",   "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedDecrement_nf,    "NiNiD*",   "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedDecrement_rel,   "NiNiD*",   "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedDecrement64_acq, "LLiLLiD*", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
TARGET_HEADER_BUILTIN(_InterlockedDecrement64_nf,  "LLiLLiD*", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")
````
- **L321 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L321 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L322 EN**: Blank line separating nearby declarations or logic blocks.
  **L322 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L323 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L323 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L324 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L324 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L325 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L325 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L326 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L326 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L327 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L327 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L328 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L328 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L329 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L329 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L330 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L330 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L331 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L331 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L332 EN**: Blank line separating nearby declarations or logic blocks.
  **L332 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L333 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L333 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L334 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L334 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L335 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L335 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L336 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L336 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L337 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L337 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L338 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L338 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L339 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L339 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L340 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L340 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 341-346

````cpp
TARGET_HEADER_BUILTIN(_InterlockedDecrement64_rel, "LLiLLiD*", "nh", INTRIN_H, ALL_MS_LANGUAGES, "")

#undef BUILTIN
#undef LANGBUILTIN
#undef TARGET_BUILTIN
#undef TARGET_HEADER_BUILTIN
````
- **L341 EN**: Invokes macro `TARGET_HEADER_BUILTIN` to contribute one entry to a table-driven definition list.
  **L341 CN**: 调用宏 `TARGET_HEADER_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L342 EN**: Blank line separating nearby declarations or logic blocks.
  **L342 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L343 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef BUILTIN`.
  **L343 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef BUILTIN`。
- **L344 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef LANGBUILTIN`.
  **L344 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef LANGBUILTIN`。
- **L345 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef TARGET_BUILTIN`.
  **L345 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef TARGET_BUILTIN`。
- **L346 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef TARGET_HEADER_BUILTIN`.
  **L346 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef TARGET_HEADER_BUILTIN`。

## Key Concepts / 关键概念

- **Compiler configuration / 编译器配置**
  - **EN**: Holds low-level enums, constants, feature switches, and descriptive metadata used across Clang.
  - **CN**: 保存 Clang 各处复用的底层枚举、常量、特性开关与描述性元数据。
- **Target and language metadata / 目标与语言元数据**
  - **EN**: Models ABI, address-space, builtin, and target-feature information shared by many subsystems.
  - **CN**: 建模多个子系统共享的 ABI、地址空间、builtin 与目标特性信息。
- **Generated definition tables / 生成式定义表**
  - **EN**: Many Basic headers use `.td` or `.def` files to describe families of diagnostics, attributes, or builtins declaratively.
  - **CN**: 许多 Basic 头文件使用 `.td` 或 `.def` 文件以声明式方式描述诊断、属性或 builtin 家族。
- **Macro-driven definition lists / 宏驱动定义列表**
  - **EN**: Provides reusable entry lists that different includers expand under different macro definitions.
  - **CN**: 提供可复用的条目列表，不同包含者可在不同宏定义下对其展开。
- **Builtin descriptions / Builtin 描述**
  - **EN**: Describes compiler builtins, their signatures, properties, or target-specific availability.
  - **CN**: 描述编译器 builtin 的签名、属性或目标相关可用性。
- **Attribute metadata / 属性元数据**
  - **EN**: Captures attribute spellings, subjects, semantic flags, and generated helper data.
  - **CN**: 刻画属性的拼写、适用对象、语义标志与生成的辅助数据。
- **Type-system support / 类型系统支撑**
  - **EN**: Provides metadata or helpers used to model Clang types and related dispatch logic.
  - **CN**: 提供建模 Clang 类型及相关分发逻辑所需的元数据或辅助工具。
- **Arm target support / Arm 目标支持**
  - **EN**: Encodes Arm-specific language extensions, intrinsics, or target metadata.
  - **CN**: 编码 Arm 专用语言扩展、intrinsic 或目标元数据。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: No prominent macros detected. / 未检测到明显宏。
- **Types / 类型**: No obvious type declarations detected by the generator. / 生成器未检测到明显类型声明。
- **Functions or callables / 函数或可调用对象**: `BUILTIN`, `TARGET_BUILTIN`, `TARGET_HEADER_BUILTIN`, `registers`, `LANGBUILTIN`
- **TableGen records / TableGen 记录**: No prominent TableGen record names detected. / 未检测到明显的 TableGen 记录名称。
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
