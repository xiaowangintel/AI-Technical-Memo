# BuiltinsPPC.def — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/BuiltinsPPC.def`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: PowerPC Builtin function database *- C++.
- **Purpose (CN)**: 声明与 `BuiltinsPPC` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 1284

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===--- BuiltinsPPC.def - PowerPC Builtin function database ----*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the PowerPC-specific builtin function database.  Users of
// this file must define the BUILTIN macro or the CUSTOM_BUILTIN macro to
// make use of this information. The latter is used for builtins requiring
// custom code generation and checking.
//
//===----------------------------------------------------------------------===//

// FIXME: this needs to be the full list supported by GCC.  Right now, I'm just
// adding stuff on demand.

// The format of this database matches clang/Basic/Builtins.def except for the
// MMA builtins that are using their own format documented below.

#ifndef BUILTIN
#define BUILTIN(ID, TYPE, ATTRS)
#endif
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
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the PowerPC-specific builtin function database. Users of`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the PowerPC-specific builtin function database. Users of`。
- **L10 EN**: Comment explains nearby logic, constraints, or intent: `this file must define the BUILTIN macro or the CUSTOM_BUILTIN macro to`.
  **L10 CN**: 注释解释附近代码的逻辑、约束或设计意图：`this file must define the BUILTIN macro or the CUSTOM_BUILTIN macro to`。
- **L11 EN**: Comment explains nearby logic, constraints, or intent: `make use of this information. The latter is used for builtins requiring`.
  **L11 CN**: 注释解释附近代码的逻辑、约束或设计意图：`make use of this information. The latter is used for builtins requiring`。
- **L12 EN**: Comment explains nearby logic, constraints, or intent: `custom code generation and checking.`.
  **L12 CN**: 注释解释附近代码的逻辑、约束或设计意图：`custom code generation and checking.`。
- **L13 EN**: Separator comment used for visual grouping.
  **L13 CN**: 用于视觉分组的分隔注释。
- **L14 EN**: Banner comment marking a file or section boundary.
  **L14 CN**: 横幅注释，用于标记文件或章节边界。
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L16 EN**: Comment records a pending task or caution: `FIXME: this needs to be the full list supported by GCC. Right now, I'm just`.
  **L16 CN**: 注释记录待办事项或注意点：`FIXME: this needs to be the full list supported by GCC. Right now, I'm just`。
- **L17 EN**: Comment explains nearby logic, constraints, or intent: `adding stuff on demand.`.
  **L17 CN**: 注释解释附近代码的逻辑、约束或设计意图：`adding stuff on demand.`。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L19 EN**: Comment explains nearby logic, constraints, or intent: `The format of this database matches clang/Basic/Builtins.def except for the`.
  **L19 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The format of this database matches clang/Basic/Builtins.def except for the`。
- **L20 EN**: Comment explains nearby logic, constraints, or intent: `MMA builtins that are using their own format documented below.`.
  **L20 CN**: 注释解释附近代码的逻辑、约束或设计意图：`MMA builtins that are using their own format documented below.`。
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L22 EN**: Starts a preprocessor conditional block: `#ifndef BUILTIN`.
  **L22 CN**: 开始一个预处理条件块：`#ifndef BUILTIN`。
- **L23 EN**: Defines macro `BUILTIN(ID,` for conditional compilation, shorthand, or table-driven expansion.
  **L23 CN**: 定义宏 `BUILTIN(ID,`，用于条件编译、简写或表驱动展开。
- **L24 EN**: Closes the current preprocessor conditional block.
  **L24 CN**: 结束当前预处理条件块。

### Lines 25-48

````cpp

#if defined(BUILTIN) && !defined(TARGET_BUILTIN)
#define TARGET_BUILTIN(ID, TYPE, ATTRS, FEATURE) BUILTIN(ID, TYPE, ATTRS)
#endif

// Built-ins requiring custom code generation.
// Because these built-ins rely on target-dependent types and to avoid pervasive
// change, they are type checked manually in Sema using custom type descriptors.
// The first argument of the CUSTOM_BUILTIN macro is the name of the built-in
// with its prefix, the second argument is the name of the intrinsic this
// built-in generates, the third argument specifies the type of the function
// (result value, then each argument) as follows:
//  i -> Unsigned integer followed by the greatest possible value for that
//       argument or 0 if no constraint on the value.
//       (e.g. i15 for a 4-bits value)
//  V -> Vector type used with MMA built-ins (vector unsigned char)
//  W -> PPC Vector type followed by the size of the vector type.
//       (e.g. W512 for __vector_quad)
//  any other descriptor -> Fall back to generic type descriptor decoding.
// The 'C' suffix can be used as a suffix to specify the const type.
// The '*' suffix can be used as a suffix to specify a pointer to a type.
// The fourth argument is set to true if the built-in accumulates its result
// into its given accumulator.

````
- **L25 EN**: Blank line separating nearby declarations or logic blocks.
  **L25 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L26 EN**: Starts a preprocessor conditional block: `#if defined(BUILTIN) && !defined(TARGET_BUILTIN)`.
  **L26 CN**: 开始一个预处理条件块：`#if defined(BUILTIN) && !defined(TARGET_BUILTIN)`。
- **L27 EN**: Defines macro `TARGET_BUILTIN(ID,` for conditional compilation, shorthand, or table-driven expansion.
  **L27 CN**: 定义宏 `TARGET_BUILTIN(ID,`，用于条件编译、简写或表驱动展开。
- **L28 EN**: Closes the current preprocessor conditional block.
  **L28 CN**: 结束当前预处理条件块。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L30 EN**: Comment explains nearby logic, constraints, or intent: `Built-ins requiring custom code generation.`.
  **L30 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Built-ins requiring custom code generation.`。
- **L31 EN**: Comment explains nearby logic, constraints, or intent: `Because these built-ins rely on target-dependent types and to avoid pervasive`.
  **L31 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Because these built-ins rely on target-dependent types and to avoid pervasive`。
- **L32 EN**: Comment explains nearby logic, constraints, or intent: `change, they are type checked manually in Sema using custom type descriptors.`.
  **L32 CN**: 注释解释附近代码的逻辑、约束或设计意图：`change, they are type checked manually in Sema using custom type descriptors.`。
- **L33 EN**: Comment explains nearby logic, constraints, or intent: `The first argument of the CUSTOM_BUILTIN macro is the name of the built-in`.
  **L33 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The first argument of the CUSTOM_BUILTIN macro is the name of the built-in`。
- **L34 EN**: Comment explains nearby logic, constraints, or intent: `with its prefix, the second argument is the name of the intrinsic this`.
  **L34 CN**: 注释解释附近代码的逻辑、约束或设计意图：`with its prefix, the second argument is the name of the intrinsic this`。
- **L35 EN**: Comment explains nearby logic, constraints, or intent: `built-in generates, the third argument specifies the type of the function`.
  **L35 CN**: 注释解释附近代码的逻辑、约束或设计意图：`built-in generates, the third argument specifies the type of the function`。
- **L36 EN**: Comment explains nearby logic, constraints, or intent: `(result value, then each argument) as follows:`.
  **L36 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(result value, then each argument) as follows:`。
- **L37 EN**: Comment explains nearby logic, constraints, or intent: `i -> Unsigned integer followed by the greatest possible value for that`.
  **L37 CN**: 注释解释附近代码的逻辑、约束或设计意图：`i -> Unsigned integer followed by the greatest possible value for that`。
- **L38 EN**: Comment explains nearby logic, constraints, or intent: `argument or 0 if no constraint on the value.`.
  **L38 CN**: 注释解释附近代码的逻辑、约束或设计意图：`argument or 0 if no constraint on the value.`。
- **L39 EN**: Comment explains nearby logic, constraints, or intent: `(e.g. i15 for a 4-bits value)`.
  **L39 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(e.g. i15 for a 4-bits value)`。
- **L40 EN**: Comment explains nearby logic, constraints, or intent: `V -> Vector type used with MMA built-ins (vector unsigned char)`.
  **L40 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V -> Vector type used with MMA built-ins (vector unsigned char)`。
- **L41 EN**: Comment explains nearby logic, constraints, or intent: `W -> PPC Vector type followed by the size of the vector type.`.
  **L41 CN**: 注释解释附近代码的逻辑、约束或设计意图：`W -> PPC Vector type followed by the size of the vector type.`。
- **L42 EN**: Comment explains nearby logic, constraints, or intent: `(e.g. W512 for __vector_quad)`.
  **L42 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(e.g. W512 for __vector_quad)`。
- **L43 EN**: Comment explains nearby logic, constraints, or intent: `any other descriptor -> Fall back to generic type descriptor decoding.`.
  **L43 CN**: 注释解释附近代码的逻辑、约束或设计意图：`any other descriptor -> Fall back to generic type descriptor decoding.`。
- **L44 EN**: Comment explains nearby logic, constraints, or intent: `The 'C' suffix can be used as a suffix to specify the const type.`.
  **L44 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The 'C' suffix can be used as a suffix to specify the const type.`。
- **L45 EN**: Comment explains nearby logic, constraints, or intent: `The '*' suffix can be used as a suffix to specify a pointer to a type.`.
  **L45 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The '*' suffix can be used as a suffix to specify a pointer to a type.`。
- **L46 EN**: Comment explains nearby logic, constraints, or intent: `The fourth argument is set to true if the built-in accumulates its result`.
  **L46 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The fourth argument is set to true if the built-in accumulates its result`。
- **L47 EN**: Comment explains nearby logic, constraints, or intent: `into its given accumulator.`.
  **L47 CN**: 注释解释附近代码的逻辑、约束或设计意图：`into its given accumulator.`。
- **L48 EN**: Blank line separating nearby declarations or logic blocks.
  **L48 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 49-72

````cpp
#ifndef CUSTOM_BUILTIN
#define CUSTOM_BUILTIN(ID, INTR, TYPES, ACCUMULATE, FEATURE)                   \
  TARGET_BUILTIN(__builtin_##ID, "i.", "t", FEATURE)
#endif

// UNALIASED_CUSTOM_BUILTIN macro is used for built-ins that have
// the same name as that of the intrinsic they generate, i.e. the
// ID and INTR are the same.
// This avoids repeating the ID and INTR in the macro expression.
#define UNALIASED_CUSTOM_BUILTIN(ID, TYPES, ACCUMULATE, FEATURE)               \
  CUSTOM_BUILTIN(ID, ID, TYPES, ACCUMULATE, FEATURE)

// UNALIASED_CUSTOM_MMA_BUILTIN macro is used for MMA built-ins and its
// corresponding 4 positive/negative multiply and positive/negative accumulate
// built-in with the same ID concated with posfix [nn|np|pn|pp].
// This avoids repeating the TYPES and FEATURE in the macro expression.
// eg.
//   UNALIASED_CUSTOM_MMA_BUILTIN(mma_xvf16ger2, "vW512*VV",
//                                "mma,paired-vector-memops")
// Will generate built-ins:
//    * __builtin_mma_xvf16ger2()
//    * __builtin_mma_xvf16ger2nn()
//    * __builtin_mma_xvf16ger2np()
//    * __builtin_mma_xvf16ger2pn()
````
- **L49 EN**: Starts a preprocessor conditional block: `#ifndef CUSTOM_BUILTIN`.
  **L49 CN**: 开始一个预处理条件块：`#ifndef CUSTOM_BUILTIN`。
- **L50 EN**: Defines macro `CUSTOM_BUILTIN(ID,` for conditional compilation, shorthand, or table-driven expansion.
  **L50 CN**: 定义宏 `CUSTOM_BUILTIN(ID,`，用于条件编译、简写或表驱动展开。
- **L51 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L51 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L52 EN**: Closes the current preprocessor conditional block.
  **L52 CN**: 结束当前预处理条件块。
- **L53 EN**: Blank line separating nearby declarations or logic blocks.
  **L53 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L54 EN**: Comment explains nearby logic, constraints, or intent: `UNALIASED_CUSTOM_BUILTIN macro is used for built-ins that have`.
  **L54 CN**: 注释解释附近代码的逻辑、约束或设计意图：`UNALIASED_CUSTOM_BUILTIN macro is used for built-ins that have`。
- **L55 EN**: Comment explains nearby logic, constraints, or intent: `the same name as that of the intrinsic they generate, i.e. the`.
  **L55 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the same name as that of the intrinsic they generate, i.e. the`。
- **L56 EN**: Comment explains nearby logic, constraints, or intent: `ID and INTR are the same.`.
  **L56 CN**: 注释解释附近代码的逻辑、约束或设计意图：`ID and INTR are the same.`。
- **L57 EN**: Comment explains nearby logic, constraints, or intent: `This avoids repeating the ID and INTR in the macro expression.`.
  **L57 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This avoids repeating the ID and INTR in the macro expression.`。
- **L58 EN**: Defines macro `UNALIASED_CUSTOM_BUILTIN(ID,` for conditional compilation, shorthand, or table-driven expansion.
  **L58 CN**: 定义宏 `UNALIASED_CUSTOM_BUILTIN(ID,`，用于条件编译、简写或表驱动展开。
- **L59 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L59 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L60 EN**: Blank line separating nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L61 EN**: Comment explains nearby logic, constraints, or intent: `UNALIASED_CUSTOM_MMA_BUILTIN macro is used for MMA built-ins and its`.
  **L61 CN**: 注释解释附近代码的逻辑、约束或设计意图：`UNALIASED_CUSTOM_MMA_BUILTIN macro is used for MMA built-ins and its`。
- **L62 EN**: Comment explains nearby logic, constraints, or intent: `corresponding 4 positive/negative multiply and positive/negative accumulate`.
  **L62 CN**: 注释解释附近代码的逻辑、约束或设计意图：`corresponding 4 positive/negative multiply and positive/negative accumulate`。
- **L63 EN**: Comment explains nearby logic, constraints, or intent: `built-in with the same ID concated with posfix [nn|np|pn|pp].`.
  **L63 CN**: 注释解释附近代码的逻辑、约束或设计意图：`built-in with the same ID concated with posfix [nn|np|pn|pp].`。
- **L64 EN**: Comment explains nearby logic, constraints, or intent: `This avoids repeating the TYPES and FEATURE in the macro expression.`.
  **L64 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This avoids repeating the TYPES and FEATURE in the macro expression.`。
- **L65 EN**: Comment explains nearby logic, constraints, or intent: `eg.`.
  **L65 CN**: 注释解释附近代码的逻辑、约束或设计意图：`eg.`。
- **L66 EN**: Comment explains nearby logic, constraints, or intent: `UNALIASED_CUSTOM_MMA_BUILTIN(mma_xvf16ger2, "vW512*VV",`.
  **L66 CN**: 注释解释附近代码的逻辑、约束或设计意图：`UNALIASED_CUSTOM_MMA_BUILTIN(mma_xvf16ger2, "vW512*VV",`。
- **L67 EN**: Comment explains nearby logic, constraints, or intent: `"mma,paired-vector-memops")`.
  **L67 CN**: 注释解释附近代码的逻辑、约束或设计意图：`"mma,paired-vector-memops")`。
- **L68 EN**: Comment explains nearby logic, constraints, or intent: `Will generate built-ins:`.
  **L68 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Will generate built-ins:`。
- **L69 EN**: Comment explains nearby logic, constraints, or intent: `__builtin_mma_xvf16ger2()`.
  **L69 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__builtin_mma_xvf16ger2()`。
- **L70 EN**: Comment explains nearby logic, constraints, or intent: `__builtin_mma_xvf16ger2nn()`.
  **L70 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__builtin_mma_xvf16ger2nn()`。
- **L71 EN**: Comment explains nearby logic, constraints, or intent: `__builtin_mma_xvf16ger2np()`.
  **L71 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__builtin_mma_xvf16ger2np()`。
- **L72 EN**: Comment explains nearby logic, constraints, or intent: `__builtin_mma_xvf16ger2pn()`.
  **L72 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__builtin_mma_xvf16ger2pn()`。

### Lines 73-96

````cpp
//    * __builtin_mma_xvf16ger2nn()
// All with the same TYPES and FEATURE.
#define UNALIASED_CUSTOM_MMA_BUILTIN(ID, TYPES, FEATURE)                       \
  UNALIASED_CUSTOM_BUILTIN(ID, TYPES, false, FEATURE)                          \
  UNALIASED_CUSTOM_BUILTIN(ID##nn, TYPES, true, FEATURE)                       \
  UNALIASED_CUSTOM_BUILTIN(ID##np, TYPES, true, FEATURE)                       \
  UNALIASED_CUSTOM_BUILTIN(ID##pn, TYPES, true, FEATURE)                       \
  UNALIASED_CUSTOM_BUILTIN(ID##pp, TYPES, true, FEATURE)

// GCC predefined macros to rename builtins, undef them to keep original names.
#if defined(__GNUC__) && !defined(__clang__)
#undef __builtin_vsx_xvnmaddadp
#undef __builtin_vsx_xvnmaddasp
#undef __builtin_vsx_xvmsubasp
#undef __builtin_vsx_xvmsubadp
#undef __builtin_vsx_xvmaddadp
#undef __builtin_vsx_xvnmsubasp
#undef __builtin_vsx_xvnmsubadp
#undef __builtin_vsx_xvmaddasp
#endif

// XL Compatibility built-ins
BUILTIN(__builtin_ppc_popcntb, "ULiULi", "")
BUILTIN(__builtin_ppc_poppar4, "iUi", "")
````
- **L73 EN**: Comment explains nearby logic, constraints, or intent: `__builtin_mma_xvf16ger2nn()`.
  **L73 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__builtin_mma_xvf16ger2nn()`。
- **L74 EN**: Comment explains nearby logic, constraints, or intent: `All with the same TYPES and FEATURE.`.
  **L74 CN**: 注释解释附近代码的逻辑、约束或设计意图：`All with the same TYPES and FEATURE.`。
- **L75 EN**: Defines macro `UNALIASED_CUSTOM_MMA_BUILTIN(ID,` for conditional compilation, shorthand, or table-driven expansion.
  **L75 CN**: 定义宏 `UNALIASED_CUSTOM_MMA_BUILTIN(ID,`，用于条件编译、简写或表驱动展开。
- **L76 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L76 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L77 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L77 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L78 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L78 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L79 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L79 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L80 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L80 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L82 EN**: Comment explains nearby logic, constraints, or intent: `GCC predefined macros to rename builtins, undef them to keep original names.`.
  **L82 CN**: 注释解释附近代码的逻辑、约束或设计意图：`GCC predefined macros to rename builtins, undef them to keep original names.`。
- **L83 EN**: Starts a preprocessor conditional block: `#if defined(__GNUC__) && !defined(__clang__)`.
  **L83 CN**: 开始一个预处理条件块：`#if defined(__GNUC__) && !defined(__clang__)`。
- **L84 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __builtin_vsx_xvnmaddadp`.
  **L84 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __builtin_vsx_xvnmaddadp`。
- **L85 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __builtin_vsx_xvnmaddasp`.
  **L85 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __builtin_vsx_xvnmaddasp`。
- **L86 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __builtin_vsx_xvmsubasp`.
  **L86 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __builtin_vsx_xvmsubasp`。
- **L87 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __builtin_vsx_xvmsubadp`.
  **L87 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __builtin_vsx_xvmsubadp`。
- **L88 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __builtin_vsx_xvmaddadp`.
  **L88 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __builtin_vsx_xvmaddadp`。
- **L89 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __builtin_vsx_xvnmsubasp`.
  **L89 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __builtin_vsx_xvnmsubasp`。
- **L90 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __builtin_vsx_xvnmsubadp`.
  **L90 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __builtin_vsx_xvnmsubadp`。
- **L91 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __builtin_vsx_xvmaddasp`.
  **L91 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __builtin_vsx_xvmaddasp`。
- **L92 EN**: Closes the current preprocessor conditional block.
  **L92 CN**: 结束当前预处理条件块。
- **L93 EN**: Blank line separating nearby declarations or logic blocks.
  **L93 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L94 EN**: Comment explains nearby logic, constraints, or intent: `XL Compatibility built-ins`.
  **L94 CN**: 注释解释附近代码的逻辑、约束或设计意图：`XL Compatibility built-ins`。
- **L95 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L95 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L96 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L96 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 97-120

````cpp
BUILTIN(__builtin_ppc_poppar8, "iULLi", "")
BUILTIN(__builtin_ppc_eieio, "v", "")
BUILTIN(__builtin_ppc_iospace_eieio, "v", "")
BUILTIN(__builtin_ppc_isync, "v", "")
BUILTIN(__builtin_ppc_lwsync, "v", "")
BUILTIN(__builtin_ppc_iospace_lwsync, "v", "")
BUILTIN(__builtin_ppc_sync, "v", "")
BUILTIN(__builtin_ppc_iospace_sync, "v", "")
BUILTIN(__builtin_ppc_dcbfl, "vvC*", "")
BUILTIN(__builtin_ppc_dcbflp, "vvC*", "")
BUILTIN(__builtin_ppc_dcbst, "vvC*", "")
BUILTIN(__builtin_ppc_dcbt, "vv*", "")
BUILTIN(__builtin_ppc_dcbtst, "vv*", "")
BUILTIN(__builtin_ppc_dcbz, "vv*", "")
TARGET_BUILTIN(__builtin_ppc_icbt, "vv*", "", "isa-v207-instructions")
BUILTIN(__builtin_ppc_fric, "dd", "")
BUILTIN(__builtin_ppc_frim, "dd", "")
BUILTIN(__builtin_ppc_frims, "ff", "")
BUILTIN(__builtin_ppc_frin, "dd", "")
BUILTIN(__builtin_ppc_frins, "ff", "")
BUILTIN(__builtin_ppc_frip, "dd", "")
BUILTIN(__builtin_ppc_frips, "ff", "")
BUILTIN(__builtin_ppc_friz, "dd", "")
BUILTIN(__builtin_ppc_frizs, "ff", "")
````
- **L97 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L97 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L98 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L98 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L99 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L99 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L100 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L100 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L101 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L101 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L102 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L102 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L103 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L103 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L104 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L104 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L105 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L105 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L106 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L106 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L107 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L107 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L108 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L108 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L109 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L109 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L110 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L110 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L111 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L111 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
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
- **L119 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L119 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L120 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L120 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 121-144

````cpp
BUILTIN(__builtin_ppc_fsel, "dddd", "")
BUILTIN(__builtin_ppc_fsels, "ffff", "")
BUILTIN(__builtin_ppc_frsqrte, "dd", "")
BUILTIN(__builtin_ppc_frsqrtes, "ff", "")
BUILTIN(__builtin_ppc_fsqrt, "dd", "")
BUILTIN(__builtin_ppc_fsqrts, "ff", "")
BUILTIN(__builtin_ppc_compare_and_swap, "iiD*i*i", "")
BUILTIN(__builtin_ppc_compare_and_swaplp, "iLiD*Li*Li", "")
BUILTIN(__builtin_ppc_fetch_and_add, "iiD*i", "")
BUILTIN(__builtin_ppc_fetch_and_addlp, "LiLiD*Li", "")
BUILTIN(__builtin_ppc_fetch_and_and, "UiUiD*Ui", "")
BUILTIN(__builtin_ppc_fetch_and_andlp, "ULiULiD*ULi", "")
BUILTIN(__builtin_ppc_fetch_and_or, "UiUiD*Ui", "")
BUILTIN(__builtin_ppc_fetch_and_orlp, "ULiULiD*ULi", "")
BUILTIN(__builtin_ppc_fetch_and_swap, "UiUiD*Ui", "")
BUILTIN(__builtin_ppc_fetch_and_swaplp, "ULiULiD*ULi", "")
BUILTIN(__builtin_ppc_ldarx, "LiLiD*", "")
BUILTIN(__builtin_ppc_lwarx, "iiD*", "")
TARGET_BUILTIN(__builtin_ppc_lharx, "ssD*", "", "isa-v207-instructions")
TARGET_BUILTIN(__builtin_ppc_lbarx, "ccD*", "", "isa-v207-instructions")
BUILTIN(__builtin_ppc_stdcx, "iLiD*Li", "")
BUILTIN(__builtin_ppc_stwcx, "iiD*i", "")
TARGET_BUILTIN(__builtin_ppc_sthcx, "isD*s", "", "isa-v207-instructions")
TARGET_BUILTIN(__builtin_ppc_stbcx, "icD*i", "", "isa-v207-instructions")
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
- **L126 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L126 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L127 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L127 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L128 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L128 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L129 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L129 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L130 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L130 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
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
- **L136 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L136 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L137 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L137 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L138 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L138 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L139 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L139 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L140 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L140 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L141 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L141 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L142 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L142 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L143 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L143 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L144 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L144 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 145-168

````cpp
BUILTIN(__builtin_ppc_tdw, "vLLiLLiIUi", "")
BUILTIN(__builtin_ppc_tw, "viiIUi", "")
BUILTIN(__builtin_ppc_trap, "vi", "")
BUILTIN(__builtin_ppc_trapd, "vLi", "")
BUILTIN(__builtin_ppc_fcfid, "dd", "")
BUILTIN(__builtin_ppc_fcfud, "dd", "")
BUILTIN(__builtin_ppc_fctid, "dd", "")
BUILTIN(__builtin_ppc_fctidz, "dd", "")
BUILTIN(__builtin_ppc_fctiw, "dd", "")
BUILTIN(__builtin_ppc_fctiwz, "dd", "")
BUILTIN(__builtin_ppc_fctudz, "dd", "")
BUILTIN(__builtin_ppc_fctuwz, "dd", "")

// fence builtin prevents all instructions moved across it
BUILTIN(__builtin_ppc_fence, "v", "")

BUILTIN(__builtin_ppc_swdiv_nochk, "ddd", "")
BUILTIN(__builtin_ppc_swdivs_nochk, "fff", "")
BUILTIN(__builtin_ppc_alignx, "vIivC*", "nc")
BUILTIN(__builtin_ppc_rdlam, "UWiUWiUWiUWIi", "nc")
TARGET_BUILTIN(__builtin_ppc_compare_exp_uo, "idd", "", "isa-v30-instructions,vsx")
TARGET_BUILTIN(__builtin_ppc_compare_exp_lt, "idd", "", "isa-v30-instructions,vsx")
TARGET_BUILTIN(__builtin_ppc_compare_exp_gt, "idd", "", "isa-v30-instructions,vsx")
TARGET_BUILTIN(__builtin_ppc_compare_exp_eq, "idd", "", "isa-v30-instructions,vsx")
````
- **L145 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L145 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L146 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L146 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L147 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L147 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L148 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L148 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L149 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L149 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L150 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L150 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L151 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L151 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L152 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L152 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L153 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L153 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L154 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L154 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L155 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L155 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L156 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L156 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L157 EN**: Blank line separating nearby declarations or logic blocks.
  **L157 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L158 EN**: Comment explains nearby logic, constraints, or intent: `fence builtin prevents all instructions moved across it`.
  **L158 CN**: 注释解释附近代码的逻辑、约束或设计意图：`fence builtin prevents all instructions moved across it`。
- **L159 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L159 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L160 EN**: Blank line separating nearby declarations or logic blocks.
  **L160 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L161 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L161 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L162 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L162 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L163 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L163 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L164 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L164 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L165 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L165 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L166 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L166 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L167 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L167 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L168 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L168 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 169-192

````cpp
TARGET_BUILTIN(__builtin_ppc_test_data_class, "idIi", "t", "isa-v30-instructions,vsx")
BUILTIN(__builtin_ppc_swdiv, "ddd", "")
BUILTIN(__builtin_ppc_swdivs, "fff", "")
// Compare
TARGET_BUILTIN(__builtin_ppc_cmpeqb, "LLiLLiLLi", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_ppc_cmprb, "iCIiii", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_ppc_setb, "LLiLLiLLi", "", "isa-v30-instructions")
BUILTIN(__builtin_ppc_cmpb, "LLiLLiLLi", "")
// Multiply
BUILTIN(__builtin_ppc_mulhd, "LLiLiLi", "")
BUILTIN(__builtin_ppc_mulhdu, "ULLiULiULi", "")
BUILTIN(__builtin_ppc_mulhw, "iii", "")
BUILTIN(__builtin_ppc_mulhwu, "UiUiUi", "")
TARGET_BUILTIN(__builtin_ppc_maddhd, "LLiLLiLLiLLi", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_ppc_maddhdu, "ULLiULLiULLiULLi", "",
               "isa-v30-instructions")
TARGET_BUILTIN(__builtin_ppc_maddld, "LLiLLiLLiLLi", "", "isa-v30-instructions")
// Rotate
BUILTIN(__builtin_ppc_rlwnm, "UiUiUiIUi", "")
BUILTIN(__builtin_ppc_rlwimi, "UiUiUiIUiIUi", "")
BUILTIN(__builtin_ppc_rldimi, "ULLiULLiULLiIUiIULLi", "")
// load
BUILTIN(__builtin_ppc_load2r, "UsUs*", "")
BUILTIN(__builtin_ppc_load4r, "UiUi*", "")
````
- **L169 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L169 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L170 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L170 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L171 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L171 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L172 EN**: Comment explains nearby logic, constraints, or intent: `Compare`.
  **L172 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compare`。
- **L173 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L173 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L174 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L174 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L175 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L175 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L176 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L176 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L177 EN**: Comment explains nearby logic, constraints, or intent: `Multiply`.
  **L177 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multiply`。
- **L178 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L178 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L179 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L179 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L180 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L180 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L181 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L181 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L182 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L182 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L183 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L183 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L184 EN**: Continues the surrounding expression or declaration: `"isa-v30-instructions")`.
  **L184 CN**: 继续构造周围的表达式或声明：`"isa-v30-instructions")`。
- **L185 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L185 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L186 EN**: Comment explains nearby logic, constraints, or intent: `Rotate`.
  **L186 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Rotate`。
- **L187 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L187 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L188 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L188 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L189 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L189 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L190 EN**: Comment explains nearby logic, constraints, or intent: `load`.
  **L190 CN**: 注释解释附近代码的逻辑、约束或设计意图：`load`。
- **L191 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L191 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L192 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L192 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 193-216

````cpp
TARGET_BUILTIN(__builtin_ppc_load8r, "ULLiULLi*", "", "isa-v206-instructions")
// store
BUILTIN(__builtin_ppc_store2r, "vUiUs*", "")
BUILTIN(__builtin_ppc_store4r, "vUiUi*", "")
TARGET_BUILTIN(__builtin_ppc_store8r, "vULLiULLi*", "", "isa-v206-instructions")
TARGET_BUILTIN(__builtin_ppc_extract_exp, "Uid", "", "power9-vector")
TARGET_BUILTIN(__builtin_ppc_extract_sig, "ULLid", "", "power9-vector")
BUILTIN(__builtin_ppc_mtfsb0, "vUIi", "")
BUILTIN(__builtin_ppc_mtfsb1, "vUIi", "")
BUILTIN(__builtin_ppc_mffs, "d", "")
TARGET_BUILTIN(__builtin_ppc_mffsl, "d", "", "isa-v30-instructions")
BUILTIN(__builtin_ppc_mtfsf, "vUIiUi", "")
BUILTIN(__builtin_ppc_mtfsfi, "vUIiUIi", "")
BUILTIN(__builtin_ppc_set_fpscr_rn, "di", "")
TARGET_BUILTIN(__builtin_ppc_insert_exp, "ddULLi", "", "power9-vector")
BUILTIN(__builtin_ppc_fmsub, "dddd", "")
BUILTIN(__builtin_ppc_fmsubs, "ffff", "")
BUILTIN(__builtin_ppc_fnmadd, "dddd", "")
BUILTIN(__builtin_ppc_fnmadds, "ffff", "")
BUILTIN(__builtin_ppc_fnmsub, "dddd", "")
BUILTIN(__builtin_ppc_fnmsubs, "ffff", "")
BUILTIN(__builtin_ppc_fre, "dd", "")
BUILTIN(__builtin_ppc_fres, "ff", "")
BUILTIN(__builtin_ppc_dcbtstt, "vv*", "")
````
- **L193 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L193 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L194 EN**: Comment explains nearby logic, constraints, or intent: `store`.
  **L194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`store`。
- **L195 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L195 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L196 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L196 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L197 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L197 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L198 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L198 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L199 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L199 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L200 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L200 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L201 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L201 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L202 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L202 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L203 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L203 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L204 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L204 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L205 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L205 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L206 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L206 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L207 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L207 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
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
- **L213 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L213 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L214 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L214 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L215 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L215 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L216 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L216 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 217-240

````cpp
BUILTIN(__builtin_ppc_dcbtt, "vv*", "")
BUILTIN(__builtin_ppc_mftbu, "Ui", "")
BUILTIN(__builtin_ppc_mfmsr, "Ui", "")
BUILTIN(__builtin_ppc_mfspr, "ULiIi", "")
BUILTIN(__builtin_ppc_mtmsr, "vUi", "")
BUILTIN(__builtin_ppc_mtspr, "vIiULi", "")
BUILTIN(__builtin_ppc_stfiw, "viC*d", "")
TARGET_BUILTIN(__builtin_ppc_addex, "LLiLLiLLiCIi", "", "isa-v30-instructions")
// select
BUILTIN(__builtin_ppc_maxfe, "LdLdLdLd.", "t")
BUILTIN(__builtin_ppc_maxfl, "dddd.", "t")
BUILTIN(__builtin_ppc_maxfs, "ffff.", "t")
BUILTIN(__builtin_ppc_minfe, "LdLdLdLd.", "t")
BUILTIN(__builtin_ppc_minfl, "dddd.", "t")
BUILTIN(__builtin_ppc_minfs, "ffff.", "t")
// Floating Negative Absolute Value
BUILTIN(__builtin_ppc_fnabs, "dd", "")
BUILTIN(__builtin_ppc_fnabss, "ff", "")

BUILTIN(__builtin_ppc_get_timebase, "ULLi", "n")

// This is just a placeholder, the types and attributes are wrong.
TARGET_BUILTIN(__builtin_altivec_vaddcuw, "V4UiV4UiV4Ui", "", "altivec")

````
- **L217 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L217 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L218 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L218 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L219 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L219 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L220 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L220 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L221 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L221 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L222 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L222 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L223 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L223 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L224 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L224 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L225 EN**: Comment explains nearby logic, constraints, or intent: `select`.
  **L225 CN**: 注释解释附近代码的逻辑、约束或设计意图：`select`。
- **L226 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L226 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L227 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L227 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L228 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L228 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L229 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L229 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L230 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L230 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L231 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L231 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L232 EN**: Comment explains nearby logic, constraints, or intent: `Floating Negative Absolute Value`.
  **L232 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Floating Negative Absolute Value`。
- **L233 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L233 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L234 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L234 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L235 EN**: Blank line separating nearby declarations or logic blocks.
  **L235 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L236 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L236 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L237 EN**: Blank line separating nearby declarations or logic blocks.
  **L237 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L238 EN**: Comment explains nearby logic, constraints, or intent: `This is just a placeholder, the types and attributes are wrong.`.
  **L238 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This is just a placeholder, the types and attributes are wrong.`。
- **L239 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L239 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L240 EN**: Blank line separating nearby declarations or logic blocks.
  **L240 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 241-264

````cpp
TARGET_BUILTIN(__builtin_altivec_vaddsbs, "V16ScV16ScV16Sc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vaddubs, "V16UcV16UcV16Uc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vaddshs, "V8SsV8SsV8Ss", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vadduhs, "V8UsV8UsV8Us", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vaddsws, "V4SiV4SiV4Si", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vadduws, "V4UiV4UiV4Ui", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vaddeuqm, "V1ULLLiV1ULLLiV1ULLLiV1ULLLi", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vaddcuq, "V1ULLLiV1ULLLiV1ULLLi", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vaddecuq, "V1ULLLiV1ULLLiV1ULLLiV1ULLLi", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vadduqm, "V1ULLLiV16UcV16Uc", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vaddeuqm_c, "V16UcV16UcV16UcV16Uc", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vaddcuq_c, "V16UcV16UcV16Uc", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vaddecuq_c, "V16UcV16UcV16UcV16Uc", "",
               "power8-vector")

TARGET_BUILTIN(__builtin_altivec_vsubsbs, "V16ScV16ScV16Sc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vsububs, "V16UcV16UcV16Uc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vsubshs, "V8SsV8SsV8Ss", "", "altivec")
````
- **L241 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L241 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L242 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L242 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L243 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L243 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L244 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L244 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L245 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L245 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L246 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L246 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L247 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L247 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L248 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L248 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L249 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L249 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L250 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L250 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L251 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L251 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L252 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L252 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L253 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L253 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L254 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L254 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L255 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L255 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L256 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L256 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L257 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L257 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L258 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L258 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L259 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L259 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L260 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L260 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L261 EN**: Blank line separating nearby declarations or logic blocks.
  **L261 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L262 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L262 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L263 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L263 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L264 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L264 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 265-288

````cpp
TARGET_BUILTIN(__builtin_altivec_vsubuhs, "V8UsV8UsV8Us", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vsubsws, "V4SiV4SiV4Si", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vsubuws, "V4UiV4UiV4Ui", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vsubeuqm, "V1ULLLiV1ULLLiV1ULLLiV1ULLLi", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vsubcuq, "V1ULLLiV1ULLLiV1ULLLi", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vsubecuq, "V1ULLLiV1ULLLiV1ULLLiV1ULLLi", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vsubuqm, "V1ULLLiV16UcV16Uc", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vsubeuqm_c, "V16UcV16UcV16UcV16Uc", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vsubcuq_c, "V16UcV16UcV16Uc", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vsubecuq_c, "V16UcV16UcV16UcV16Uc", "",
               "power8-vector")

TARGET_BUILTIN(__builtin_altivec_vavgsb, "V16ScV16ScV16Sc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vavgub, "V16UcV16UcV16Uc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vavgsh, "V8SsV8SsV8Ss", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vavguh, "V8UsV8UsV8Us", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vavgsw, "V4SiV4SiV4Si", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vavguw, "V4UiV4UiV4Ui", "", "altivec")
````
- **L265 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L265 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L266 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L266 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L267 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L267 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L268 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L268 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L269 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L269 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L270 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L270 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L271 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L271 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L272 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L272 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L273 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L273 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L274 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L274 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L275 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L275 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L276 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L276 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L277 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L277 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L278 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L278 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L279 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L279 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L280 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L280 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L281 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L281 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L282 EN**: Blank line separating nearby declarations or logic blocks.
  **L282 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L283 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L283 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L284 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L284 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L285 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L285 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L286 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L286 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L287 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L287 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L288 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L288 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 289-312

````cpp

TARGET_BUILTIN(__builtin_altivec_vrfip, "V4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vcfsx, "V4fV4SiIi", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcfux, "V4fV4UiIi", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vctsxs, "V4SiV4fIi", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vctuxs, "V4UiV4fIi", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_dss, "vUIi", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_dssall, "v", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_dst, "vvC*iUIi", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_dstt, "vvC*iUIi", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_dstst, "vvC*iUIi", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_dststt, "vvC*iUIi", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vexptefp, "V4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vrfim, "V4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_lvx, "V4iLivC*", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_lvxl, "V4iLivC*", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_lvebx, "V16cLivC*", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_lvehx, "V8sLivC*", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_lvewx, "V4iLivC*", "", "altivec")
````
- **L289 EN**: Blank line separating nearby declarations or logic blocks.
  **L289 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L290 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L290 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L291 EN**: Blank line separating nearby declarations or logic blocks.
  **L291 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L292 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L292 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L293 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L293 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L294 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L294 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L295 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L295 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L296 EN**: Blank line separating nearby declarations or logic blocks.
  **L296 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L297 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L297 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L298 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L298 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L299 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L299 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L300 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L300 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L301 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L301 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L302 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L302 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L303 EN**: Blank line separating nearby declarations or logic blocks.
  **L303 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L304 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L304 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L305 EN**: Blank line separating nearby declarations or logic blocks.
  **L305 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L306 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L306 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L307 EN**: Blank line separating nearby declarations or logic blocks.
  **L307 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L308 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L308 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L309 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L309 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L310 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L310 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L311 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L311 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L312 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L312 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 313-336

````cpp

TARGET_BUILTIN(__builtin_altivec_vlogefp, "V4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_lvsl, "V16cUcvC*", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_lvsr, "V16cUcvC*", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vmaddfp, "V4fV4fV4fV4f", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmhaddshs, "V8sV8sV8sV8s", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmhraddshs, "V8sV8sV8sV8s", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vmsumubm, "V4UiV16UcV16UcV4Ui", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmsummbm, "V4SiV16ScV16UcV4Si", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmsumuhm, "V4UiV8UsV8UsV4Ui", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmsumshm, "V4SiV8SsV8SsV4Si", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmsumuhs, "V4UiV8UsV8UsV4Ui", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmsumshs, "V4SiV8SsV8SsV4Si", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vmuleub, "V8UsV16UcV16Uc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmulesb, "V8SsV16ScV16Sc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmuleuh, "V4UiV8UsV8Us", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmulesh, "V4SiV8SsV8Ss", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmuleuw, "V2ULLiV4UiV4Ui", "", "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vmulesw, "V2SLLiV4SiV4Si", "", "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vmuloub, "V8UsV16UcV16Uc", "", "altivec")
````
- **L313 EN**: Blank line separating nearby declarations or logic blocks.
  **L313 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L314 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L314 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L315 EN**: Blank line separating nearby declarations or logic blocks.
  **L315 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L316 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L316 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L317 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L317 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L318 EN**: Blank line separating nearby declarations or logic blocks.
  **L318 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L319 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L319 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L320 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L320 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L321 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L321 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L322 EN**: Blank line separating nearby declarations or logic blocks.
  **L322 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L323 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L323 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L324 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L324 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L325 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L325 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L326 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L326 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L327 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L327 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L328 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L328 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L329 EN**: Blank line separating nearby declarations or logic blocks.
  **L329 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L330 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L330 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L331 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L331 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L332 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L332 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L333 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L333 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L334 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L334 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L335 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L335 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L336 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L336 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 337-360

````cpp
TARGET_BUILTIN(__builtin_altivec_vmulosb, "V8SsV16ScV16Sc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmulouh, "V4UiV8UsV8Us", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmulosh, "V4SiV8SsV8Ss", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmulouw, "V2ULLiV4UiV4Ui", "", "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vmulosw, "V2SLLiV4SiV4Si", "", "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vmuleud, "V1ULLLiV2ULLiV2ULLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vmulesd, "V1SLLLiV2SLLiV2SLLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vmuloud, "V1ULLLiV2ULLiV2ULLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vmulosd, "V1SLLLiV2SLLiV2SLLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vmsumcud, "V1ULLLiV2ULLiV2ULLiV1ULLLi", "",
               "power10-vector")

TARGET_BUILTIN(__builtin_altivec_vnmsubfp, "V4fV4fV4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vpkpx, "V8sV4UiV4Ui", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vpkuhus, "V16UcV8UsV8Us", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vpkshss, "V16ScV8SsV8Ss", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vpkuwus, "V8UsV4UiV4Ui", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vpkswss, "V8SsV4SiV4Si", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vpkshus, "V16UcV8SsV8Ss", "", "altivec")
````
- **L337 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L337 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L338 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L338 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L339 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L339 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L340 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L340 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L341 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L341 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L342 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L342 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L343 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L343 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L344 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L344 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L345 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L345 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L346 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L346 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L347 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L347 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L348 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L348 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L349 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L349 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L350 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L350 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L351 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L351 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L352 EN**: Blank line separating nearby declarations or logic blocks.
  **L352 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L353 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L353 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L354 EN**: Blank line separating nearby declarations or logic blocks.
  **L354 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L355 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L355 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L356 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L356 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L357 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L357 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L358 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L358 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L359 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L359 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L360 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L360 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 361-384

````cpp
TARGET_BUILTIN(__builtin_altivec_vpkswus, "V8UsV4SiV4Si", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vpksdss, "V4SiV2SLLiV2SLLi", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vpksdus, "V4UiV2SLLiV2SLLi", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vpkudus, "V4UiV2ULLiV2ULLi", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vpkudum, "V4UiV2ULLiV2ULLi", "",
               "power8-vector")

TARGET_BUILTIN(__builtin_altivec_vperm_4si, "V4iV4iV4iV16Uc", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_stvx, "vV4iLiv*", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_stvxl, "vV4iLiv*", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_stvebx, "vV16cLiv*", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_stvehx, "vV8sLiv*", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_stvewx, "vV4iLiv*", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vcmpbfp, "V4iV4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vcmpgefp, "V4iV4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vcmpequb, "V16cV16cV16c", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpequh, "V8sV8sV8s", "", "altivec")
````
- **L361 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L361 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L362 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L362 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L363 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L363 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L364 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L364 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L365 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L365 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L366 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L366 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L367 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L367 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L368 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L368 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L369 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L369 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L370 EN**: Blank line separating nearby declarations or logic blocks.
  **L370 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L371 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L371 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L372 EN**: Blank line separating nearby declarations or logic blocks.
  **L372 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L373 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L373 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L374 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L374 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L375 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L375 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L376 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L376 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L377 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L377 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L378 EN**: Blank line separating nearby declarations or logic blocks.
  **L378 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L379 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L379 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L380 EN**: Blank line separating nearby declarations or logic blocks.
  **L380 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L381 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L381 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L382 EN**: Blank line separating nearby declarations or logic blocks.
  **L382 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L383 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L383 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L384 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L384 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 385-408

````cpp
TARGET_BUILTIN(__builtin_altivec_vcmpequw, "V4iV4iV4i", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpequd, "V2LLiV2LLiV2LLi", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vcmpeqfp, "V4iV4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vcmpneb, "V16cV16cV16c", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vcmpneh, "V8sV8sV8s", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vcmpnew, "V4iV4iV4i", "", "power9-vector")

TARGET_BUILTIN(__builtin_altivec_vcmpnezb, "V16cV16cV16c", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vcmpnezh, "V8sV8sV8s", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vcmpnezw, "V4iV4iV4i", "", "power9-vector")

TARGET_BUILTIN(__builtin_altivec_vcmpgtsb, "V16cV16ScV16Sc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpgtub, "V16cV16UcV16Uc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpgtsh, "V8sV8SsV8Ss", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpgtuh, "V8sV8UsV8Us", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpgtsw, "V4iV4SiV4Si", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpgtuw, "V4iV4UiV4Ui", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpgtsd, "V2LLiV2LLiV2LLi", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vcmpgtud, "V2LLiV2ULLiV2ULLi", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vcmpgtfp, "V4iV4fV4f", "", "altivec")
````
- **L385 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L385 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L386 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L386 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L387 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L387 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L388 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L388 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L389 EN**: Blank line separating nearby declarations or logic blocks.
  **L389 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L390 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L390 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L391 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L391 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L392 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L392 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L393 EN**: Blank line separating nearby declarations or logic blocks.
  **L393 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L394 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L394 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L395 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L395 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L396 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L396 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L397 EN**: Blank line separating nearby declarations or logic blocks.
  **L397 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L398 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L398 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L399 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L399 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L400 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L400 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L401 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L401 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L402 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L402 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L403 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L403 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L404 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L404 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L405 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L405 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L406 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L406 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L407 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L407 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L408 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L408 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 409-432

````cpp

// P10 Vector compare builtins.
TARGET_BUILTIN(__builtin_altivec_vcmpequq, "V1LLLiV1ULLLiV1ULLLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vcmpgtsq, "V1LLLiV1SLLLiV1SLLLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vcmpgtuq, "V1LLLiV1ULLLiV1ULLLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vcmpequq_p, "iiV1ULLLiV1LLLi", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpgtsq_p, "iiV1SLLLiV1SLLLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vcmpgtuq_p, "iiV1ULLLiV1ULLLi", "",
               "power10-vector")

TARGET_BUILTIN(__builtin_altivec_vmaxsb, "V16ScV16ScV16Sc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmaxub, "V16UcV16UcV16Uc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmaxsh, "V8SsV8SsV8Ss", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmaxuh, "V8UsV8UsV8Us", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmaxsw, "V4SiV4SiV4Si", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmaxuw, "V4UiV4UiV4Ui", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vmaxsd, "V2LLiV2LLiV2LLi", "", "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vmaxud, "V2ULLiV2ULLiV2ULLi", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vmaxfp, "V4fV4fV4f", "", "altivec")
````
- **L409 EN**: Blank line separating nearby declarations or logic blocks.
  **L409 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L410 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector compare builtins.`.
  **L410 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector compare builtins.`。
- **L411 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L411 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L412 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L412 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L413 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L413 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L414 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L414 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L415 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L415 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L416 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L416 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L417 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L417 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L418 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L418 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L419 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L419 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L420 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L420 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L421 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L421 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L422 EN**: Blank line separating nearby declarations or logic blocks.
  **L422 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L423 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L423 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L424 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L424 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L425 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L425 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L426 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L426 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L427 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L427 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L428 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L428 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L429 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L429 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L430 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L430 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L431 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L431 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L432 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L432 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 433-456

````cpp

TARGET_BUILTIN(__builtin_altivec_mfvscr, "V8Us", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vminsb, "V16ScV16ScV16Sc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vminub, "V16UcV16UcV16Uc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vminsh, "V8SsV8SsV8Ss", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vminuh, "V8UsV8UsV8Us", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vminsw, "V4SiV4SiV4Si", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vminuw, "V4UiV4UiV4Ui", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vminsd, "V2LLiV2LLiV2LLi", "", "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vminud, "V2ULLiV2ULLiV2ULLi", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vminfp, "V4fV4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_mtvscr, "vV4i", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vrefp, "V4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vrlb, "V16cV16cV16Uc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vrlh, "V8sV8sV8Us", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vrlw, "V4iV4iV4Ui", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vrld, "V2LLiV2LLiV2ULLi", "", "power8-vector")

TARGET_BUILTIN(__builtin_altivec_vsel_4si, "V4iV4iV4iV4Ui", "", "altivec")
````
- **L433 EN**: Blank line separating nearby declarations or logic blocks.
  **L433 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L434 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L434 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L435 EN**: Blank line separating nearby declarations or logic blocks.
  **L435 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L436 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L436 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L437 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L437 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L438 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L438 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L439 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L439 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L440 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L440 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L441 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L441 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L442 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L442 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L443 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L443 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L444 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L444 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L445 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L445 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L446 EN**: Blank line separating nearby declarations or logic blocks.
  **L446 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L447 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L447 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L448 EN**: Blank line separating nearby declarations or logic blocks.
  **L448 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L449 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L449 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L450 EN**: Blank line separating nearby declarations or logic blocks.
  **L450 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L451 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L451 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L452 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L452 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L453 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L453 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L454 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L454 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L455 EN**: Blank line separating nearby declarations or logic blocks.
  **L455 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L456 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L456 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 457-480

````cpp

TARGET_BUILTIN(__builtin_altivec_vsl, "V4iV4iV4i", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vslo, "V4iV4iV4i", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vsrab, "V16cV16cV16Uc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vsrah, "V8sV8sV8Us", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vsraw, "V4iV4iV4Ui", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vsr, "V4iV4iV4i", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vsro, "V4iV4iV4i", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vrfin, "V4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vrsqrtefp, "V4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vsubcuw, "V4UiV4UiV4Ui", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vsum4sbs, "V4SiV16ScV4Si", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vsum4ubs, "V4UiV16UcV4Ui", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vsum4shs, "V4SiV8SsV4Si", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vsum2sws, "V4SiV4SiV4Si", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vsumsws, "V4SiV4SiV4Si", "", "altivec")
````
- **L457 EN**: Blank line separating nearby declarations or logic blocks.
  **L457 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L458 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L458 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L459 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L459 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L460 EN**: Blank line separating nearby declarations or logic blocks.
  **L460 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L461 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L461 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L462 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L462 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L463 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L463 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L464 EN**: Blank line separating nearby declarations or logic blocks.
  **L464 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L465 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L465 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L466 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L466 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L467 EN**: Blank line separating nearby declarations or logic blocks.
  **L467 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L468 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L468 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L469 EN**: Blank line separating nearby declarations or logic blocks.
  **L469 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L470 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L470 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L471 EN**: Blank line separating nearby declarations or logic blocks.
  **L471 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L472 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L472 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L473 EN**: Blank line separating nearby declarations or logic blocks.
  **L473 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L474 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L474 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L475 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L475 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L476 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L476 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L477 EN**: Blank line separating nearby declarations or logic blocks.
  **L477 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L478 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L478 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L479 EN**: Blank line separating nearby declarations or logic blocks.
  **L479 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L480 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L480 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 481-504

````cpp

TARGET_BUILTIN(__builtin_altivec_vrfiz, "V4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vupkhsb, "V8sV16c", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vupkhpx, "V4UiV8s", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vupkhsh, "V4iV8s", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vupkhsw, "V2LLiV4i", "", "power8-vector")

TARGET_BUILTIN(__builtin_altivec_vupklsb, "V8sV16c", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vupklpx, "V4UiV8s", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vupklsh, "V4iV8s", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vupklsw, "V2LLiV4i", "", "power8-vector")

TARGET_BUILTIN(__builtin_altivec_vcmpbfp_p, "iiV4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vcmpgefp_p, "iiV4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vcmpequb_p, "iiV16cV16c", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpequh_p, "iiV8sV8s", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpequw_p, "iiV4iV4i", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpequd_p, "iiV2LLiV2LLi", "", "vsx")
TARGET_BUILTIN(__builtin_altivec_vcmpeqfp_p, "iiV4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vcmpneb_p, "iiV16cV16c", "", "power9-vector")
````
- **L481 EN**: Blank line separating nearby declarations or logic blocks.
  **L481 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L482 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L482 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L483 EN**: Blank line separating nearby declarations or logic blocks.
  **L483 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L484 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L484 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L485 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L485 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L486 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L486 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L487 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L487 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L488 EN**: Blank line separating nearby declarations or logic blocks.
  **L488 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L489 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L489 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L490 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L490 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L491 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L491 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L492 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L492 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L493 EN**: Blank line separating nearby declarations or logic blocks.
  **L493 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L494 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L494 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L495 EN**: Blank line separating nearby declarations or logic blocks.
  **L495 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L496 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L496 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L497 EN**: Blank line separating nearby declarations or logic blocks.
  **L497 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L498 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L498 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L499 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L499 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L500 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L500 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L501 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L501 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L502 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L502 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L503 EN**: Blank line separating nearby declarations or logic blocks.
  **L503 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L504 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L504 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 505-528

````cpp
TARGET_BUILTIN(__builtin_altivec_vcmpneh_p, "iiV8sV8s", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vcmpnew_p, "iiV4iV4i", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vcmpned_p, "iiV2LLiV2LLi", "", "vsx")

TARGET_BUILTIN(__builtin_altivec_vcmpgtsb_p, "iiV16ScV16Sc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpgtub_p, "iiV16UcV16Uc", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpgtsh_p, "iiV8SsV8Ss", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpgtuh_p, "iiV8UsV8Us", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpgtsw_p, "iiV4SiV4Si", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpgtuw_p, "iiV4UiV4Ui", "", "altivec")
TARGET_BUILTIN(__builtin_altivec_vcmpgtsd_p, "iiV2LLiV2LLi", "", "vsx")
TARGET_BUILTIN(__builtin_altivec_vcmpgtud_p, "iiV2ULLiV2ULLi", "", "vsx")
TARGET_BUILTIN(__builtin_altivec_vcmpgtfp_p, "iiV4fV4f", "", "altivec")

TARGET_BUILTIN(__builtin_altivec_vgbbd, "V16UcV16Uc", "", "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vbpermq, "V2ULLiV16UcV16Uc", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vbpermd, "V2ULLiV2ULLiV16Uc", "",
               "power9-vector")

// P8 Crypto built-ins.
TARGET_BUILTIN(__builtin_altivec_crypto_vsbox, "V16UcV16Uc", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_crypto_vpermxor, "V16UcV16UcV16UcV16Uc", "",
````
- **L505 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L505 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L506 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L506 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L507 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L507 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L508 EN**: Blank line separating nearby declarations or logic blocks.
  **L508 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L509 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L509 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L510 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L510 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L511 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L511 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L512 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L512 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L513 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L513 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L514 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L514 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L515 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L515 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L516 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L516 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L517 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L517 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L518 EN**: Blank line separating nearby declarations or logic blocks.
  **L518 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L519 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L519 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L520 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L520 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L521 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L521 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L522 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L522 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L523 EN**: Continues the surrounding expression or declaration: `"power9-vector")`.
  **L523 CN**: 继续构造周围的表达式或声明：`"power9-vector")`。
- **L524 EN**: Blank line separating nearby declarations or logic blocks.
  **L524 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L525 EN**: Comment explains nearby logic, constraints, or intent: `P8 Crypto built-ins.`.
  **L525 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P8 Crypto built-ins.`。
- **L526 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L526 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L527 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L527 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L528 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L528 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 529-552

````cpp
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_crypto_vpermxor_be, "V16UcV16UcV16UcV16Uc", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_crypto_vshasigmaw, "V4UiV4UiIiIi", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_crypto_vshasigmad, "V2ULLiV2ULLiIiIi", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_crypto_vcipher, "V16UcV16UcV16Uc", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_crypto_vcipherlast, "V16UcV16UcV16Uc", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_crypto_vncipher, "V16UcV16UcV16Uc", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_crypto_vncipherlast, "V16UcV16UcV16Uc", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_crypto_vpmsumb, "V16UcV16UcV16Uc", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_crypto_vpmsumh, "V8UsV8UsV8Us", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_crypto_vpmsumw, "V4UiV4UiV4Ui", "",
               "power8-vector")
TARGET_BUILTIN(__builtin_altivec_crypto_vpmsumd, "V2ULLiV2ULLiV2ULLi", "",
               "power8-vector")

````
- **L529 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L529 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L530 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L530 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L531 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L531 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L532 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L532 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L533 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L533 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L534 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L534 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L535 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L535 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L536 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L536 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L537 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L537 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L538 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L538 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L539 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L539 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L540 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L540 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L541 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L541 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L542 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L542 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L543 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L543 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L544 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L544 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L545 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L545 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L546 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L546 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L547 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L547 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L548 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L548 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L549 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L549 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L550 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L550 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L551 EN**: Continues the surrounding expression or declaration: `"power8-vector")`.
  **L551 CN**: 继续构造周围的表达式或声明：`"power8-vector")`。
- **L552 EN**: Blank line separating nearby declarations or logic blocks.
  **L552 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 553-576

````cpp
TARGET_BUILTIN(__builtin_altivec_vclzb, "V16UcV16Uc", "", "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vclzh, "V8UsV8Us", "", "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vclzw, "V4UiV4Ui", "", "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vclzd, "V2ULLiV2ULLi", "", "power8-vector")
TARGET_BUILTIN(__builtin_altivec_vctzb, "V16UcV16Uc", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vctzh, "V8UsV8Us", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vctzw, "V4UiV4Ui", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vctzd, "V2ULLiV2ULLi", "", "power9-vector")

// P7 BCD builtins.
TARGET_BUILTIN(__builtin_cdtbcd, "UiUi", "", "isa-v206-instructions")
TARGET_BUILTIN(__builtin_cbcdtd, "UiUi", "", "isa-v206-instructions")
TARGET_BUILTIN(__builtin_addg6s, "UiUiUi", "", "isa-v206-instructions")

// P7 XL Compat BCD builtins.
TARGET_BUILTIN(__builtin_ppc_cdtbcd, "LLiLLi", "", "isa-v206-instructions")
TARGET_BUILTIN(__builtin_ppc_cbcdtd, "LLiLLi", "", "isa-v206-instructions")
TARGET_BUILTIN(__builtin_ppc_addg6s, "LLiLLiLLi", "", "isa-v206-instructions")

// P8 BCD builtins.
TARGET_BUILTIN(__builtin_ppc_bcdadd, "V16UcV16UcV16UcIi", "",
               "isa-v207-instructions")
TARGET_BUILTIN(__builtin_ppc_bcdsub, "V16UcV16UcV16UcIi", "",
               "isa-v207-instructions")
````
- **L553 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L553 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L554 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L554 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L555 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L555 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L556 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L556 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L557 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L557 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L558 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L558 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L559 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L559 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L560 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L560 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L561 EN**: Blank line separating nearby declarations or logic blocks.
  **L561 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L562 EN**: Comment explains nearby logic, constraints, or intent: `P7 BCD builtins.`.
  **L562 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P7 BCD builtins.`。
- **L563 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L563 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L564 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L564 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L565 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L565 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L566 EN**: Blank line separating nearby declarations or logic blocks.
  **L566 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L567 EN**: Comment explains nearby logic, constraints, or intent: `P7 XL Compat BCD builtins.`.
  **L567 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P7 XL Compat BCD builtins.`。
- **L568 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L568 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L569 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L569 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L570 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L570 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L571 EN**: Blank line separating nearby declarations or logic blocks.
  **L571 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L572 EN**: Comment explains nearby logic, constraints, or intent: `P8 BCD builtins.`.
  **L572 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P8 BCD builtins.`。
- **L573 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L573 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L574 EN**: Continues the surrounding expression or declaration: `"isa-v207-instructions")`.
  **L574 CN**: 继续构造周围的表达式或声明：`"isa-v207-instructions")`。
- **L575 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L575 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L576 EN**: Continues the surrounding expression or declaration: `"isa-v207-instructions")`.
  **L576 CN**: 继续构造周围的表达式或声明：`"isa-v207-instructions")`。

### Lines 577-600

````cpp
TARGET_BUILTIN(__builtin_ppc_bcdadd_p, "iiV16UcV16Uc", "",
               "isa-v207-instructions")
TARGET_BUILTIN(__builtin_ppc_bcdsub_p, "iiV16UcV16Uc", "",
               "isa-v207-instructions")

// P9 Binary-coded decimal (BCD) builtins.                                                
TARGET_BUILTIN(__builtin_ppc_bcdcopysign, "V16UcV16UcV16Uc", "", "power9-vector")
TARGET_BUILTIN(__builtin_ppc_bcdsetsign, "V16UcV16UcUc", "t", "power9-vector")
TARGET_BUILTIN(__builtin_ppc_bcdshift, "V16UcV16UciUc", "t", "power9-vector")
TARGET_BUILTIN(__builtin_ppc_bcdshiftround, "V16UcV16UciUc", "t", "power9-vector")
TARGET_BUILTIN(__builtin_ppc_bcdtruncate, "V16UcV16UciUc", "t", "power9-vector")
TARGET_BUILTIN(__builtin_ppc_bcdunsignedtruncate, "V16UcV16Uci", "", "power9-vector")
TARGET_BUILTIN(__builtin_ppc_bcdunsignedshift, "V16UcV16Uci", "", "power9-vector")
TARGET_BUILTIN(__builtin_ppc_national2packed, "V16UcV16UcUc", "t", "power9-vector")
TARGET_BUILTIN(__builtin_ppc_packed2national, "V16UcV16Uc", "", "power9-vector")
TARGET_BUILTIN(__builtin_ppc_packed2zoned, "V16UcV16UcUc", "t", "power9-vector")
TARGET_BUILTIN(__builtin_ppc_zoned2packed, "V16UcV16UcUc", "t", "power9-vector")

TARGET_BUILTIN(__builtin_altivec_vclzlsbb, "SiV16Uc", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vctzlsbb, "SiV16Uc", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vprtybw, "V4UiV4Ui", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vprtybd, "V2ULLiV2ULLi", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vprtybq, "V1ULLLiV1ULLLi", "", "power9-vector")

````
- **L577 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L577 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L578 EN**: Continues the surrounding expression or declaration: `"isa-v207-instructions")`.
  **L578 CN**: 继续构造周围的表达式或声明：`"isa-v207-instructions")`。
- **L579 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L579 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L580 EN**: Continues the surrounding expression or declaration: `"isa-v207-instructions")`.
  **L580 CN**: 继续构造周围的表达式或声明：`"isa-v207-instructions")`。
- **L581 EN**: Blank line separating nearby declarations or logic blocks.
  **L581 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L582 EN**: Comment explains nearby logic, constraints, or intent: `P9 Binary-coded decimal (BCD) builtins.`.
  **L582 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P9 Binary-coded decimal (BCD) builtins.`。
- **L583 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L583 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L584 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L584 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L585 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L585 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L586 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L586 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L587 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L587 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L588 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L588 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L589 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L589 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L590 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L590 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L591 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L591 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L592 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L592 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L593 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L593 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L594 EN**: Blank line separating nearby declarations or logic blocks.
  **L594 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L595 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L595 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L596 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L596 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L597 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L597 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L598 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L598 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L599 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L599 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L600 EN**: Blank line separating nearby declarations or logic blocks.
  **L600 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 601-624

````cpp
// Absolute difference built-ins
TARGET_BUILTIN(__builtin_altivec_vabsdub, "V16UcV16UcV16Uc", "",
               "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vabsduh, "V8UsV8UsV8Us", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vabsduw, "V4UiV4UiV4Ui", "", "power9-vector")

// P9 Shift built-ins.
TARGET_BUILTIN(__builtin_altivec_vslv, "V16UcV16UcV16Uc", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vsrv, "V16UcV16UcV16Uc", "", "power9-vector")

// P9 Vector rotate built-ins
TARGET_BUILTIN(__builtin_altivec_vrlwmi, "V4UiV4UiV4UiV4Ui", "",
               "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vrldmi, "V2ULLiV2ULLiV2ULLiV2ULLi", "",
               "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vrlwnm, "V4UiV4UiV4Ui", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vrldnm, "V2ULLiV2ULLiV2ULLi", "",
               "power9-vector")

// P9 Vector extend sign builtins.
TARGET_BUILTIN(__builtin_altivec_vextsb2w, "V4SiV16Sc", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vextsb2d, "V2SLLiV16Sc", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vextsh2w, "V4SiV8Ss", "", "power9-vector")
TARGET_BUILTIN(__builtin_altivec_vextsh2d, "V2SLLiV8Ss", "", "power9-vector")
````
- **L601 EN**: Comment explains nearby logic, constraints, or intent: `Absolute difference built-ins`.
  **L601 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Absolute difference built-ins`。
- **L602 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L602 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L603 EN**: Continues the surrounding expression or declaration: `"power9-vector")`.
  **L603 CN**: 继续构造周围的表达式或声明：`"power9-vector")`。
- **L604 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L604 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L605 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L605 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L606 EN**: Blank line separating nearby declarations or logic blocks.
  **L606 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L607 EN**: Comment explains nearby logic, constraints, or intent: `P9 Shift built-ins.`.
  **L607 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P9 Shift built-ins.`。
- **L608 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L608 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L609 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L609 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L610 EN**: Blank line separating nearby declarations or logic blocks.
  **L610 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L611 EN**: Comment explains nearby logic, constraints, or intent: `P9 Vector rotate built-ins`.
  **L611 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P9 Vector rotate built-ins`。
- **L612 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L612 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L613 EN**: Continues the surrounding expression or declaration: `"power9-vector")`.
  **L613 CN**: 继续构造周围的表达式或声明：`"power9-vector")`。
- **L614 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L614 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L615 EN**: Continues the surrounding expression or declaration: `"power9-vector")`.
  **L615 CN**: 继续构造周围的表达式或声明：`"power9-vector")`。
- **L616 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L616 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L617 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L617 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L618 EN**: Continues the surrounding expression or declaration: `"power9-vector")`.
  **L618 CN**: 继续构造周围的表达式或声明：`"power9-vector")`。
- **L619 EN**: Blank line separating nearby declarations or logic blocks.
  **L619 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L620 EN**: Comment explains nearby logic, constraints, or intent: `P9 Vector extend sign builtins.`.
  **L620 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P9 Vector extend sign builtins.`。
- **L621 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L621 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L622 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L622 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L623 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L623 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L624 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L624 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 625-648

````cpp
TARGET_BUILTIN(__builtin_altivec_vextsw2d, "V2SLLiV4Si", "", "power9-vector")

// P10 Vector extend sign builtins.
TARGET_BUILTIN(__builtin_altivec_vextsd2q, "V1SLLLiV2SLLi", "",
               "power10-vector")

// P10 Vector Extract with Mask built-ins.
TARGET_BUILTIN(__builtin_altivec_vextractbm, "UiV16Uc", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vextracthm, "UiV8Us", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vextractwm, "UiV4Ui", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vextractdm, "UiV2ULLi", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vextractqm, "UiV1ULLLi", "", "power10-vector")

// P10 Vector Divide Extended built-ins.
TARGET_BUILTIN(__builtin_altivec_vdivesw, "V4SiV4SiV4Si", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vdiveuw, "V4UiV4UiV4Ui", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vdivesd, "V2LLiV2LLiV2LLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vdiveud, "V2ULLiV2ULLiV2ULLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vdivesq, "V1SLLLiV1SLLLiV1SLLLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vdiveuq, "V1ULLLiV1ULLLiV1ULLLi", "",
               "power10-vector")
````
- **L625 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L625 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L626 EN**: Blank line separating nearby declarations or logic blocks.
  **L626 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L627 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector extend sign builtins.`.
  **L627 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector extend sign builtins.`。
- **L628 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L628 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L629 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L629 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L630 EN**: Blank line separating nearby declarations or logic blocks.
  **L630 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L631 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector Extract with Mask built-ins.`.
  **L631 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector Extract with Mask built-ins.`。
- **L632 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L632 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L633 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L633 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L634 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L634 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L635 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L635 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L636 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L636 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L637 EN**: Blank line separating nearby declarations or logic blocks.
  **L637 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L638 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector Divide Extended built-ins.`.
  **L638 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector Divide Extended built-ins.`。
- **L639 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L639 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L640 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L640 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L641 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L641 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L642 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L642 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L643 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L643 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L644 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L644 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L645 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L645 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L646 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L646 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L647 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L647 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L648 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L648 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。

### Lines 649-672

````cpp

// P10 Vector Multiply High built-ins.
TARGET_BUILTIN(__builtin_altivec_vmulhsw, "V4SiV4SiV4Si", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vmulhuw, "V4UiV4UiV4Ui", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vmulhsd, "V2LLiV2LLiV2LLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vmulhud, "V2ULLiV2ULLiV2ULLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vmulhsh, "V8SsV8SsV8Ss", "", "future-vector")
TARGET_BUILTIN(__builtin_altivec_vmulhuh, "V8UsV8UsV8Us", "", "future-vector")

// P10 Vector Expand with Mask built-ins.
TARGET_BUILTIN(__builtin_altivec_vexpandbm, "V16UcV16Uc", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vexpandhm, "V8UsV8Us", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vexpandwm, "V4UiV4Ui", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vexpanddm, "V2ULLiV2ULLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vexpandqm, "V1ULLLiV1ULLLi", "",
               "power10-vector")

// P10 Vector Count with Mask built-ins.
TARGET_BUILTIN(__builtin_altivec_vcntmbb, "ULLiV16UcUi", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vcntmbh, "ULLiV8UsUi", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vcntmbw, "ULLiV4UiUi", "", "power10-vector")
````
- **L649 EN**: Blank line separating nearby declarations or logic blocks.
  **L649 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L650 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector Multiply High built-ins.`.
  **L650 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector Multiply High built-ins.`。
- **L651 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L651 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L652 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L652 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L653 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L653 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L654 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L654 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L655 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L655 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L656 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L656 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L657 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L657 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L658 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L658 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L659 EN**: Blank line separating nearby declarations or logic blocks.
  **L659 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L660 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector Expand with Mask built-ins.`.
  **L660 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector Expand with Mask built-ins.`。
- **L661 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L661 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L662 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L662 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L663 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L663 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L664 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L664 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L665 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L665 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L666 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L666 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L667 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L667 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L668 EN**: Blank line separating nearby declarations or logic blocks.
  **L668 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L669 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector Count with Mask built-ins.`.
  **L669 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector Count with Mask built-ins.`。
- **L670 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L670 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L671 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L671 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L672 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L672 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 673-696

````cpp
TARGET_BUILTIN(__builtin_altivec_vcntmbd, "ULLiV2ULLiUi", "", "power10-vector")

// P10 Move to VSR with Mask built-ins.
TARGET_BUILTIN(__builtin_altivec_mtvsrbm, "V16UcULLi", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_mtvsrhm, "V8UsULLi", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_mtvsrwm, "V4UiULLi", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_mtvsrdm, "V2ULLiULLi", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_mtvsrqm, "V1ULLLiULLi", "", "power10-vector")

// P10 Vector Parallel Bits built-ins.
TARGET_BUILTIN(__builtin_altivec_vpdepd, "V2ULLiV2ULLiV2ULLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vpextd, "V2ULLiV2ULLiV2ULLi", "",
               "power10-vector")

// P10 Vector String Isolate Built-ins.
TARGET_BUILTIN(__builtin_altivec_vstribr, "V16UcV16Uc", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vstribl, "V16UcV16Uc", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vstrihr, "V8sV8s", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vstrihl, "V8sV8s", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vstribr_p, "iiV16Uc", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vstribl_p, "iiV16Uc", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vstrihr_p, "iiV8s", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vstrihl_p, "iiV8s", "", "power10-vector")
````
- **L673 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L673 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L674 EN**: Blank line separating nearby declarations or logic blocks.
  **L674 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L675 EN**: Comment explains nearby logic, constraints, or intent: `P10 Move to VSR with Mask built-ins.`.
  **L675 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Move to VSR with Mask built-ins.`。
- **L676 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L676 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L677 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L677 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L678 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L678 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L679 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L679 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L680 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L680 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L681 EN**: Blank line separating nearby declarations or logic blocks.
  **L681 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L682 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector Parallel Bits built-ins.`.
  **L682 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector Parallel Bits built-ins.`。
- **L683 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L683 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L684 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L684 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L685 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L685 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L686 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L686 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L687 EN**: Blank line separating nearby declarations or logic blocks.
  **L687 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L688 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector String Isolate Built-ins.`.
  **L688 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector String Isolate Built-ins.`。
- **L689 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L689 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L690 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L690 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L691 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L691 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L692 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L692 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L693 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L693 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L694 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L694 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L695 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L695 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L696 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L696 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 697-720

````cpp

// P10 Vector Centrifuge built-in.
TARGET_BUILTIN(__builtin_altivec_vcfuged, "V2ULLiV2ULLiV2ULLi", "",
               "power10-vector")

// P10 Vector Gather Every N-th Bit built-in.
TARGET_BUILTIN(__builtin_altivec_vgnb, "ULLiV1ULLLiIi", "", "power10-vector")

// P10 Vector Clear Bytes built-ins.
TARGET_BUILTIN(__builtin_altivec_vclrlb, "V16UcV16UcUi", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vclrrb, "V16UcV16UcUi", "", "power10-vector")

// P10 Vector Count Leading / Trailing Zeroes under bit Mask built-ins.
TARGET_BUILTIN(__builtin_altivec_vclzdm, "V2ULLiV2ULLiV2ULLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vctzdm, "V2ULLiV2ULLiV2ULLi", "",
               "power10-vector")

// P10 Vector Shift built-ins.
TARGET_BUILTIN(__builtin_altivec_vsldbi, "V16UcV16UcV16UcIi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vsrdbi, "V16UcV16UcV16UcIi", "",
               "power10-vector")

````
- **L697 EN**: Blank line separating nearby declarations or logic blocks.
  **L697 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L698 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector Centrifuge built-in.`.
  **L698 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector Centrifuge built-in.`。
- **L699 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L699 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L700 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L700 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L701 EN**: Blank line separating nearby declarations or logic blocks.
  **L701 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L702 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector Gather Every N-th Bit built-in.`.
  **L702 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector Gather Every N-th Bit built-in.`。
- **L703 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L703 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L704 EN**: Blank line separating nearby declarations or logic blocks.
  **L704 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L705 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector Clear Bytes built-ins.`.
  **L705 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector Clear Bytes built-ins.`。
- **L706 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L706 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L707 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L707 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L708 EN**: Blank line separating nearby declarations or logic blocks.
  **L708 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L709 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector Count Leading / Trailing Zeroes under bit Mask built-ins.`.
  **L709 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector Count Leading / Trailing Zeroes under bit Mask built-ins.`。
- **L710 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L710 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L711 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L711 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L712 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L712 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L713 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L713 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L714 EN**: Blank line separating nearby declarations or logic blocks.
  **L714 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L715 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector Shift built-ins.`.
  **L715 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector Shift built-ins.`。
- **L716 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L716 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L717 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L717 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L718 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L718 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L719 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L719 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L720 EN**: Blank line separating nearby declarations or logic blocks.
  **L720 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 721-744

````cpp
// P10 Vector Insert built-ins.
TARGET_BUILTIN(__builtin_altivec_vinsblx, "V16UcV16UcUiUi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinsbrx, "V16UcV16UcUiUi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinshlx, "V8UsV8UsUiUi", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinshrx, "V8UsV8UsUiUi", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinswlx, "V4UiV4UiUiUi", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinswrx, "V4UiV4UiUiUi", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinsdlx, "V2ULLiV2ULLiULLiULLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinsdrx, "V2ULLiV2ULLiULLiULLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinsbvlx, "V16UcV16UcUiV16Uc", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinsbvrx, "V16UcV16UcUiV16Uc", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinshvlx, "V8UsV8UsUiV8Us", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinshvrx, "V8UsV8UsUiV8Us", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinswvlx, "V4UiV4UiUiV4Ui", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinswvrx, "V4UiV4UiUiV4Ui", "",
````
- **L721 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector Insert built-ins.`.
  **L721 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector Insert built-ins.`。
- **L722 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L722 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L723 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L723 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L724 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L724 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L725 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L725 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L726 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L726 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L727 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L727 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L728 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L728 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L729 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L729 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L730 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L730 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L731 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L731 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L732 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L732 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L733 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L733 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L734 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L734 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L735 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L735 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L736 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L736 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L737 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L737 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L738 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L738 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L739 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L739 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L740 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L740 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L741 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L741 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L742 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L742 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L743 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L743 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L744 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L744 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 745-768

````cpp
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinsw, "V16UcV16UcUiIi", "", "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinsd, "V16UcV16UcULLiIi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinsw_elt, "V16UcV16UcUiiC", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vinsd_elt, "V16UcV16UcULLiiC", "",
               "power10-vector")

// P10 Vector Extract built-ins.
TARGET_BUILTIN(__builtin_altivec_vextdubvlx, "V2ULLiV16UcV16UcUi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vextdubvrx, "V2ULLiV16UcV16UcUi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vextduhvlx, "V2ULLiV8UsV8UsUi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vextduhvrx, "V2ULLiV8UsV8UsUi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vextduwvlx, "V2ULLiV4UiV4UiUi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vextduwvrx, "V2ULLiV4UiV4UiUi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vextddvlx, "V2ULLiV2ULLiV2ULLiUi", "",
               "power10-vector")
````
- **L745 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L745 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L746 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L746 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L747 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L747 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L748 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L748 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L749 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L749 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L750 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L750 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L751 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L751 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L752 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L752 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L753 EN**: Blank line separating nearby declarations or logic blocks.
  **L753 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L754 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector Extract built-ins.`.
  **L754 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector Extract built-ins.`。
- **L755 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L755 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L756 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L756 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L757 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L757 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L758 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L758 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L759 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L759 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L760 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L760 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L761 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L761 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L762 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L762 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L763 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L763 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L764 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L764 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L765 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L765 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L766 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L766 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L767 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L767 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L768 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L768 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。

### Lines 769-792

````cpp
TARGET_BUILTIN(__builtin_altivec_vextddvrx, "V2ULLiV2ULLiV2ULLiUi", "",
               "power10-vector")

// P10 Vector rotate built-ins.
TARGET_BUILTIN(__builtin_altivec_vrlqmi, "V1ULLLiV1ULLLiV1ULLLiV1ULLLi", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_altivec_vrlqnm, "V1ULLLiV1ULLLiV1ULLLi", "",
               "power10-vector")

// VSX built-ins.

TARGET_BUILTIN(__builtin_vsx_lxvd2x, "V2dLivC*", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_lxvw4x, "V4iLivC*", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_lxvd2x_be, "V2dSLLivC*", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_lxvw4x_be, "V4iSLLivC*", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_stxvd2x, "vV2dLiv*", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_stxvw4x, "vV4iLiv*", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_stxvd2x_be, "vV2dSLLivC*", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_stxvw4x_be, "vV4iSLLivC*", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_lxvl, "V4ivC*ULLi", "", "power9-vector")
TARGET_BUILTIN(__builtin_vsx_lxvll, "V4ivC*ULLi", "", "power9-vector")
TARGET_BUILTIN(__builtin_vsx_stxvl, "vV4iv*ULLi", "", "power9-vector")
````
- **L769 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L769 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L770 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L770 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L771 EN**: Blank line separating nearby declarations or logic blocks.
  **L771 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L772 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector rotate built-ins.`.
  **L772 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector rotate built-ins.`。
- **L773 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L773 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L774 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L774 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L775 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L775 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L776 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L776 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L777 EN**: Blank line separating nearby declarations or logic blocks.
  **L777 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L778 EN**: Comment explains nearby logic, constraints, or intent: `VSX built-ins.`.
  **L778 CN**: 注释解释附近代码的逻辑、约束或设计意图：`VSX built-ins.`。
- **L779 EN**: Blank line separating nearby declarations or logic blocks.
  **L779 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L780 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L780 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L781 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L781 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L782 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L782 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L783 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L783 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L784 EN**: Blank line separating nearby declarations or logic blocks.
  **L784 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L785 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L785 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L786 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L786 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L787 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L787 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L788 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L788 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L789 EN**: Blank line separating nearby declarations or logic blocks.
  **L789 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L790 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L790 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L791 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L791 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L792 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L792 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 793-816

````cpp
TARGET_BUILTIN(__builtin_vsx_stxvll, "vV4iv*ULLi", "", "power9-vector")
TARGET_BUILTIN(__builtin_vsx_ldrmb, "V16UcCc*Ii", "", "isa-v207-instructions")
TARGET_BUILTIN(__builtin_vsx_strmb, "vCc*IiV16Uc", "", "isa-v207-instructions")

TARGET_BUILTIN(__builtin_vsx_xvmaxdp, "V2dV2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvmaxsp, "V4fV4fV4f", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xsmaxdp, "ddd", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvmindp, "V2dV2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvminsp, "V4fV4fV4f", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xsmindp, "ddd", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvdivdp, "V2dV2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvdivsp, "V4fV4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvrdpip, "V2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvrspip, "V4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvcmpeqdp, "V2ULLiV2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvcmpeqsp, "V4UiV4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvcmpeqdp_p, "iiV2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvcmpeqsp_p, "iiV4fV4f", "", "vsx")

````
- **L793 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L793 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L794 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L794 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L795 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L795 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L796 EN**: Blank line separating nearby declarations or logic blocks.
  **L796 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L797 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L797 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L798 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L798 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L799 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L799 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L800 EN**: Blank line separating nearby declarations or logic blocks.
  **L800 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L801 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L801 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L802 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L802 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L803 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L803 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L804 EN**: Blank line separating nearby declarations or logic blocks.
  **L804 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L805 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L805 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L806 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L806 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L807 EN**: Blank line separating nearby declarations or logic blocks.
  **L807 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L808 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L808 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L809 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L809 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L810 EN**: Blank line separating nearby declarations or logic blocks.
  **L810 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L811 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L811 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L812 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L812 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L813 EN**: Blank line separating nearby declarations or logic blocks.
  **L813 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L814 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L814 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L815 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L815 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L816 EN**: Blank line separating nearby declarations or logic blocks.
  **L816 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 817-840

````cpp
TARGET_BUILTIN(__builtin_vsx_xvcmpgedp, "V2ULLiV2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvcmpgesp, "V4UiV4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvcmpgedp_p, "iiV2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvcmpgesp_p, "iiV4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvcmpgtdp, "V2ULLiV2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvcmpgtsp, "V4UiV4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvcmpgtdp_p, "iiV2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvcmpgtsp_p, "iiV4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvrdpim, "V2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvrspim, "V4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvrdpi, "V2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvrspi, "V4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvrdpic, "V2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvrspic, "V4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvrdpiz, "V2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvrspiz, "V4fV4f", "", "vsx")

````
- **L817 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L817 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L818 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L818 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L819 EN**: Blank line separating nearby declarations or logic blocks.
  **L819 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L820 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L820 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L821 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L821 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L822 EN**: Blank line separating nearby declarations or logic blocks.
  **L822 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L823 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L823 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L824 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L824 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L825 EN**: Blank line separating nearby declarations or logic blocks.
  **L825 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L826 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L826 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L827 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L827 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L828 EN**: Blank line separating nearby declarations or logic blocks.
  **L828 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L829 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L829 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L830 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L830 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L831 EN**: Blank line separating nearby declarations or logic blocks.
  **L831 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L832 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L832 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L833 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L833 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L834 EN**: Blank line separating nearby declarations or logic blocks.
  **L834 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L835 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L835 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L836 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L836 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L837 EN**: Blank line separating nearby declarations or logic blocks.
  **L837 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L838 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L838 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L839 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L839 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L840 EN**: Blank line separating nearby declarations or logic blocks.
  **L840 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 841-864

````cpp
TARGET_BUILTIN(__builtin_vsx_xvmaddadp, "V2dV2dV2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvmaddasp, "V4fV4fV4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvmsubadp, "V2dV2dV2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvmsubasp, "V4fV4fV4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvmuldp, "V2dV2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvmulsp, "V4fV4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvnmaddadp, "V2dV2dV2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvnmaddasp, "V4fV4fV4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvnmsubadp, "V2dV2dV2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvnmsubasp, "V4fV4fV4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvredp, "V2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvresp, "V4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvrsqrtedp, "V2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvrsqrtesp, "V4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvsqrtdp, "V2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvsqrtsp, "V4fV4f", "", "vsx")

````
- **L841 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L841 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L842 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L842 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L843 EN**: Blank line separating nearby declarations or logic blocks.
  **L843 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L844 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L844 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L845 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L845 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L846 EN**: Blank line separating nearby declarations or logic blocks.
  **L846 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L847 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L847 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L848 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L848 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L849 EN**: Blank line separating nearby declarations or logic blocks.
  **L849 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L850 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L850 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L851 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L851 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L852 EN**: Blank line separating nearby declarations or logic blocks.
  **L852 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L853 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L853 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L854 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L854 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L855 EN**: Blank line separating nearby declarations or logic blocks.
  **L855 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L856 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L856 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L857 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L857 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L858 EN**: Blank line separating nearby declarations or logic blocks.
  **L858 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L859 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L859 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L860 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L860 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L861 EN**: Blank line separating nearby declarations or logic blocks.
  **L861 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L862 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L862 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L863 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L863 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L864 EN**: Blank line separating nearby declarations or logic blocks.
  **L864 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 865-888

````cpp
TARGET_BUILTIN(__builtin_vsx_xxleqv, "V4UiV4UiV4Ui", "", "power8-vector")

TARGET_BUILTIN(__builtin_vsx_xvcpsgndp, "V2dV2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvcpsgnsp, "V4fV4fV4f", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvabssp, "V4fV4f", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvabsdp, "V2dV2d", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xxgenpcvbm, "V16UcV16Uci", "", "power10-vector")
TARGET_BUILTIN(__builtin_vsx_xxgenpcvhm, "V8UsV8Usi", "", "power10-vector")
TARGET_BUILTIN(__builtin_vsx_xxgenpcvwm, "V4UiV4Uii", "", "power10-vector")
TARGET_BUILTIN(__builtin_vsx_xxgenpcvdm, "V2ULLiV2ULLii", "", "power10-vector")

// vector Insert/Extract exponent/significand builtins
TARGET_BUILTIN(__builtin_vsx_xviexpdp, "V2dV2ULLiV2ULLi", "", "power9-vector")
TARGET_BUILTIN(__builtin_vsx_xviexpsp, "V4fV4UiV4Ui", "", "power9-vector")
TARGET_BUILTIN(__builtin_vsx_xvxexpdp, "V2ULLiV2d", "", "power9-vector")
TARGET_BUILTIN(__builtin_vsx_xvxexpsp, "V4UiV4f", "", "power9-vector")
TARGET_BUILTIN(__builtin_vsx_xvxsigdp, "V2ULLiV2d", "", "power9-vector")
TARGET_BUILTIN(__builtin_vsx_xvxsigsp, "V4UiV4f", "", "power9-vector")

// Conversion builtins
TARGET_BUILTIN(__builtin_vsx_xvcvdpsxws, "V4SiV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvcvdpuxws, "V4UiV2d", "", "vsx")
````
- **L865 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L865 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L866 EN**: Blank line separating nearby declarations or logic blocks.
  **L866 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L867 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L867 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L868 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L868 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L869 EN**: Blank line separating nearby declarations or logic blocks.
  **L869 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L870 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L870 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L871 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L871 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L872 EN**: Blank line separating nearby declarations or logic blocks.
  **L872 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L873 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L873 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L874 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L874 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L875 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L875 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L876 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L876 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L877 EN**: Blank line separating nearby declarations or logic blocks.
  **L877 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L878 EN**: Comment explains nearby logic, constraints, or intent: `vector Insert/Extract exponent/significand builtins`.
  **L878 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector Insert/Extract exponent/significand builtins`。
- **L879 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L879 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L880 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L880 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L881 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L881 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L882 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L882 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L883 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L883 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L884 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L884 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L885 EN**: Blank line separating nearby declarations or logic blocks.
  **L885 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L886 EN**: Comment explains nearby logic, constraints, or intent: `Conversion builtins`.
  **L886 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Conversion builtins`。
- **L887 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L887 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L888 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L888 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 889-912

````cpp
TARGET_BUILTIN(__builtin_vsx_xvcvspsxds, "V2SLLiV4f", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvcvspuxds, "V2ULLiV4f", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvcvsxwdp, "V2dV4Si", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvcvuxwdp, "V2dV4Ui", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvcvspdp, "V2dV4f", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvcvsxdsp, "V4fV2SLLi", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvcvuxdsp, "V4fV2ULLi", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvcvdpsp, "V4fV2d", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xvcvsphp, "V4fV4f", "", "power9-vector")
TARGET_BUILTIN(__builtin_vsx_xvcvhpsp, "V4fV8Us", "", "power9-vector")

TARGET_BUILTIN(__builtin_vsx_xvcvspbf16, "V16UcV16Uc", "", "power10-vector")
TARGET_BUILTIN(__builtin_vsx_xvcvbf16spn, "V16UcV16Uc", "", "power10-vector")

// Vector Test Data Class builtins
TARGET_BUILTIN(__builtin_vsx_xvtstdcdp, "V2ULLiV2dIi", "", "power9-vector")
TARGET_BUILTIN(__builtin_vsx_xvtstdcsp, "V4UiV4fIi", "", "power9-vector")

TARGET_BUILTIN(__builtin_vsx_insertword, "V16UcV4UiV16UcIi", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_extractuword, "V2ULLiV16UcIi", "", "vsx")

TARGET_BUILTIN(__builtin_vsx_xxpermdi, "v.", "t", "vsx")
TARGET_BUILTIN(__builtin_vsx_xxsldwi, "v.", "t", "vsx")
````
- **L889 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L889 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L890 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L890 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L891 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L891 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L892 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L892 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L893 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L893 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L894 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L894 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L895 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L895 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L896 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L896 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L897 EN**: Blank line separating nearby declarations or logic blocks.
  **L897 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L898 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L898 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L899 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L899 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L900 EN**: Blank line separating nearby declarations or logic blocks.
  **L900 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L901 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L901 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L902 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L902 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L903 EN**: Blank line separating nearby declarations or logic blocks.
  **L903 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L904 EN**: Comment explains nearby logic, constraints, or intent: `Vector Test Data Class builtins`.
  **L904 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector Test Data Class builtins`。
- **L905 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L905 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L906 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L906 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L907 EN**: Blank line separating nearby declarations or logic blocks.
  **L907 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L908 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L908 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L909 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L909 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L910 EN**: Blank line separating nearby declarations or logic blocks.
  **L910 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L911 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L911 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L912 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L912 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 913-936

````cpp

TARGET_BUILTIN(__builtin_vsx_xxeval, "V2ULLiV2ULLiV2ULLiV2ULLiIi", "",
               "power10-vector")

TARGET_BUILTIN(__builtin_vsx_xvtlsbb, "iV16UcUi", "", "power10-vector")

TARGET_BUILTIN(__builtin_vsx_xvtdivdp, "iV2dV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvtdivsp, "iV4fV4f", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvtsqrtdp, "iV2d", "", "vsx")
TARGET_BUILTIN(__builtin_vsx_xvtsqrtsp, "iV4f", "", "vsx")

// P10 Vector Permute Extended built-in.
TARGET_BUILTIN(__builtin_vsx_xxpermx, "V16UcV16UcV16UcV16UcIi", "",
               "power10-vector")

// P10 Vector Blend built-ins.
TARGET_BUILTIN(__builtin_vsx_xxblendvb, "V16UcV16UcV16UcV16Uc", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_vsx_xxblendvh, "V8UsV8UsV8UsV8Us", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_vsx_xxblendvw, "V4UiV4UiV4UiV4Ui", "",
               "power10-vector")
TARGET_BUILTIN(__builtin_vsx_xxblendvd, "V2ULLiV2ULLiV2ULLiV2ULLi", "",
               "power10-vector")
````
- **L913 EN**: Blank line separating nearby declarations or logic blocks.
  **L913 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L914 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L914 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L915 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L915 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L916 EN**: Blank line separating nearby declarations or logic blocks.
  **L916 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L917 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L917 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L918 EN**: Blank line separating nearby declarations or logic blocks.
  **L918 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L919 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L919 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L920 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L920 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L921 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L921 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L922 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L922 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L923 EN**: Blank line separating nearby declarations or logic blocks.
  **L923 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L924 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector Permute Extended built-in.`.
  **L924 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector Permute Extended built-in.`。
- **L925 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L925 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L926 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L926 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L927 EN**: Blank line separating nearby declarations or logic blocks.
  **L927 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L928 EN**: Comment explains nearby logic, constraints, or intent: `P10 Vector Blend built-ins.`.
  **L928 CN**: 注释解释附近代码的逻辑、约束或设计意图：`P10 Vector Blend built-ins.`。
- **L929 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L929 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L930 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L930 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L931 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L931 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L932 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L932 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L933 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L933 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L934 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L934 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。
- **L935 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L935 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L936 EN**: Continues the surrounding expression or declaration: `"power10-vector")`.
  **L936 CN**: 继续构造周围的表达式或声明：`"power10-vector")`。

### Lines 937-960

````cpp

// Float 128 built-ins
TARGET_BUILTIN(__builtin_sqrtf128_round_to_odd, "LLdLLd", "", "float128")
TARGET_BUILTIN(__builtin_addf128_round_to_odd, "LLdLLdLLd", "", "float128")
TARGET_BUILTIN(__builtin_subf128_round_to_odd, "LLdLLdLLd", "", "float128")
TARGET_BUILTIN(__builtin_mulf128_round_to_odd, "LLdLLdLLd", "", "float128")
TARGET_BUILTIN(__builtin_divf128_round_to_odd, "LLdLLdLLd", "", "float128")
TARGET_BUILTIN(__builtin_fmaf128_round_to_odd, "LLdLLdLLdLLd", "", "float128")
TARGET_BUILTIN(__builtin_truncf128_round_to_odd, "dLLd", "", "float128")
TARGET_BUILTIN(__builtin_vsx_scalar_extract_expq, "ULLiLLd", "", "float128")
TARGET_BUILTIN(__builtin_vsx_scalar_insert_exp_qp, "LLdLLdULLi", "", "float128")

// Fastmath by default builtins
BUILTIN(__builtin_ppc_rsqrtf, "V4fV4f", "")
BUILTIN(__builtin_ppc_rsqrtd, "V2dV2d", "")
BUILTIN(__builtin_ppc_recipdivf, "V4fV4fV4f", "")
BUILTIN(__builtin_ppc_recipdivd, "V2dV2dV2d", "")

// HTM builtins
TARGET_BUILTIN(__builtin_tbegin, "UiUIi", "", "htm")
TARGET_BUILTIN(__builtin_tend, "UiUIi", "", "htm")

TARGET_BUILTIN(__builtin_tabort, "UiUi", "", "htm")
TARGET_BUILTIN(__builtin_tabortdc, "UiUiUiUi", "", "htm")
````
- **L937 EN**: Blank line separating nearby declarations or logic blocks.
  **L937 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L938 EN**: Comment explains nearby logic, constraints, or intent: `Float 128 built-ins`.
  **L938 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Float 128 built-ins`。
- **L939 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L939 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L940 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L940 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L941 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L941 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L942 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L942 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L943 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L943 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L944 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L944 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L945 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L945 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L946 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L946 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L947 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L947 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L948 EN**: Blank line separating nearby declarations or logic blocks.
  **L948 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L949 EN**: Comment explains nearby logic, constraints, or intent: `Fastmath by default builtins`.
  **L949 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Fastmath by default builtins`。
- **L950 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L950 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L951 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L951 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L952 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L952 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L953 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L953 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L954 EN**: Blank line separating nearby declarations or logic blocks.
  **L954 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L955 EN**: Comment explains nearby logic, constraints, or intent: `HTM builtins`.
  **L955 CN**: 注释解释附近代码的逻辑、约束或设计意图：`HTM builtins`。
- **L956 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L956 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L957 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L957 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L958 EN**: Blank line separating nearby declarations or logic blocks.
  **L958 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L959 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L959 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L960 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L960 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 961-984

````cpp
TARGET_BUILTIN(__builtin_tabortdci, "UiUiUii", "", "htm")
TARGET_BUILTIN(__builtin_tabortwc, "UiUiUiUi", "", "htm")
TARGET_BUILTIN(__builtin_tabortwci, "UiUiUii", "", "htm")

TARGET_BUILTIN(__builtin_tcheck, "Ui", "", "htm")
TARGET_BUILTIN(__builtin_treclaim, "UiUi", "", "htm")
TARGET_BUILTIN(__builtin_trechkpt, "Ui", "", "htm")
TARGET_BUILTIN(__builtin_tsr, "UiUi", "", "htm")

TARGET_BUILTIN(__builtin_tendall, "Ui", "", "htm")
TARGET_BUILTIN(__builtin_tresume, "Ui", "", "htm")
TARGET_BUILTIN(__builtin_tsuspend, "Ui", "", "htm")

TARGET_BUILTIN(__builtin_get_texasr, "LUi", "c", "htm")
TARGET_BUILTIN(__builtin_get_texasru, "LUi", "c", "htm")
TARGET_BUILTIN(__builtin_get_tfhar, "LUi", "c", "htm")
TARGET_BUILTIN(__builtin_get_tfiar, "LUi", "c", "htm")

TARGET_BUILTIN(__builtin_set_texasr, "vLUi", "c", "htm")
TARGET_BUILTIN(__builtin_set_texasru, "vLUi", "c", "htm")
TARGET_BUILTIN(__builtin_set_tfhar, "vLUi", "c", "htm")
TARGET_BUILTIN(__builtin_set_tfiar, "vLUi", "c", "htm")

TARGET_BUILTIN(__builtin_ttest, "LUi", "", "htm")
````
- **L961 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L961 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L962 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L962 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L963 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L963 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L964 EN**: Blank line separating nearby declarations or logic blocks.
  **L964 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L965 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L965 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L966 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L966 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L967 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L967 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L968 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L968 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L969 EN**: Blank line separating nearby declarations or logic blocks.
  **L969 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L970 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L970 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L971 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L971 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L972 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L972 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L973 EN**: Blank line separating nearby declarations or logic blocks.
  **L973 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L974 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L974 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L975 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L975 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L976 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L976 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L977 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L977 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L978 EN**: Blank line separating nearby declarations or logic blocks.
  **L978 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L979 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L979 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L980 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L980 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L981 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L981 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L982 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L982 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L983 EN**: Blank line separating nearby declarations or logic blocks.
  **L983 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L984 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L984 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 985-1008

````cpp

// Scalar built-ins
TARGET_BUILTIN(__builtin_divwe, "SiSiSi", "", "extdiv")
TARGET_BUILTIN(__builtin_divweu, "UiUiUi", "", "extdiv")
TARGET_BUILTIN(__builtin_divde, "SLLiSLLiSLLi", "", "extdiv")
TARGET_BUILTIN(__builtin_divdeu, "ULLiULLiULLi", "", "extdiv")
TARGET_BUILTIN(__builtin_bpermd, "SLLiSLLiSLLi", "", "bpermd")
TARGET_BUILTIN(__builtin_pdepd, "ULLiULLiULLi", "", "isa-v31-instructions")
TARGET_BUILTIN(__builtin_pextd, "ULLiULLiULLi", "", "isa-v31-instructions")
TARGET_BUILTIN(__builtin_cfuged, "ULLiULLiULLi", "", "isa-v31-instructions")
TARGET_BUILTIN(__builtin_cntlzdm, "ULLiULLiULLi", "", "isa-v31-instructions")
TARGET_BUILTIN(__builtin_cnttzdm, "ULLiULLiULLi", "", "isa-v31-instructions")

// Double-double (un)pack
BUILTIN(__builtin_unpack_longdouble, "dLdIi", "")
BUILTIN(__builtin_pack_longdouble, "Lddd", "")

// Generate random number
TARGET_BUILTIN(__builtin_darn, "LLi", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_darn_raw, "LLi", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_darn_32, "i", "", "isa-v30-instructions")

// Vector int128 (un)pack
TARGET_BUILTIN(__builtin_unpack_vector_int128, "ULLiV1LLLii", "", "vsx")
````
- **L985 EN**: Blank line separating nearby declarations or logic blocks.
  **L985 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L986 EN**: Comment explains nearby logic, constraints, or intent: `Scalar built-ins`.
  **L986 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Scalar built-ins`。
- **L987 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L987 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L988 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L988 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L989 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L989 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L990 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L990 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L991 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L991 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L992 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L992 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L993 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L993 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L994 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L994 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L995 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L995 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L996 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L996 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L997 EN**: Blank line separating nearby declarations or logic blocks.
  **L997 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L998 EN**: Comment explains nearby logic, constraints, or intent: `Double-double (un)pack`.
  **L998 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Double-double (un)pack`。
- **L999 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L999 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1000 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L1000 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1001 EN**: Blank line separating nearby declarations or logic blocks.
  **L1001 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1002 EN**: Comment explains nearby logic, constraints, or intent: `Generate random number`.
  **L1002 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Generate random number`。
- **L1003 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1003 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1004 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1004 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1005 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1005 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1006 EN**: Blank line separating nearby declarations or logic blocks.
  **L1006 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1007 EN**: Comment explains nearby logic, constraints, or intent: `Vector int128 (un)pack`.
  **L1007 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector int128 (un)pack`。
- **L1008 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1008 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 1009-1032

````cpp
TARGET_BUILTIN(__builtin_pack_vector_int128, "V1LLLiULLiULLi", "", "vsx")

// AMO builtins
TARGET_BUILTIN(__builtin_amo_lwat, "UiUi*UiIi", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_amo_ldat, "ULiULi*ULiIi", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_amo_lwat_s, "SiSi*SiIi", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_amo_ldat_s, "SLiSLi*SLiIi", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_amo_lwat_cond, "UiUi*Ii", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_amo_ldat_cond, "ULiULi*Ii", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_amo_lwat_cond_s, "SiSi*Ii", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_amo_ldat_cond_s, "SLiSLi*Ii", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_amo_stwat, "vUi*UiIi", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_amo_stdat, "vULi*ULiIi", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_amo_stwat_s, "vSi*SiIi", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_amo_stdat_s, "vSLi*SLiIi", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_amo_lwat_csne, "UiUi*UiUi", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_amo_ldat_csne, "ULiULi*ULiULi", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_amo_lwat_csne_s, "SiSi*SiSi", "", "isa-v30-instructions")
TARGET_BUILTIN(__builtin_amo_ldat_csne_s, "SLiSLi*SLiSLi", "", "isa-v30-instructions")


// Set the floating point rounding mode
BUILTIN(__builtin_setrnd, "di", "")

````
- **L1009 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1009 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1010 EN**: Blank line separating nearby declarations or logic blocks.
  **L1010 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1011 EN**: Comment explains nearby logic, constraints, or intent: `AMO builtins`.
  **L1011 CN**: 注释解释附近代码的逻辑、约束或设计意图：`AMO builtins`。
- **L1012 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1012 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1013 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1013 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1014 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1014 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1015 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1015 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1016 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1016 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1017 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1017 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1018 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1018 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1019 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1019 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1020 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1020 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1021 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1021 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1022 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1022 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1023 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1023 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1024 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1024 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1025 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1025 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1026 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1026 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1027 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1027 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1028 EN**: Blank line separating nearby declarations or logic blocks.
  **L1028 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1029 EN**: Blank line separating nearby declarations or logic blocks.
  **L1029 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1030 EN**: Comment explains nearby logic, constraints, or intent: `Set the floating point rounding mode`.
  **L1030 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set the floating point rounding mode`。
- **L1031 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L1031 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1032 EN**: Blank line separating nearby declarations or logic blocks.
  **L1032 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1033-1056

````cpp
// Get content from current FPSCR
BUILTIN(__builtin_readflm, "d", "")

// Set content of FPSCR, and return its content before update
BUILTIN(__builtin_setflm, "dd", "")

// Cache built-ins
BUILTIN(__builtin_dcbf, "vvC*", "")

// Provided builtins with _mma_ prefix for compatibility.
CUSTOM_BUILTIN(mma_lxvp, vsx_lxvp, "W256SLiW256C*", false,
               "paired-vector-memops")
CUSTOM_BUILTIN(mma_stxvp, vsx_stxvp, "vW256SLiW256*", false,
               "paired-vector-memops")
CUSTOM_BUILTIN(mma_assemble_pair, vsx_assemble_pair, "vW256*VV", false,
               "paired-vector-memops")
CUSTOM_BUILTIN(mma_disassemble_pair, vsx_disassemble_pair, "vv*W256*", false,
               "paired-vector-memops")
CUSTOM_BUILTIN(vsx_build_pair, vsx_assemble_pair, "vW256*VV", false,
               "paired-vector-memops")
CUSTOM_BUILTIN(mma_build_acc, mma_assemble_acc, "vW512*VVVV", false, "mma")

UNALIASED_CUSTOM_BUILTIN(vsx_lxvp, "W256SLiW256C*", false,
                         "paired-vector-memops")
````
- **L1033 EN**: Comment explains nearby logic, constraints, or intent: `Get content from current FPSCR`.
  **L1033 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Get content from current FPSCR`。
- **L1034 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L1034 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1035 EN**: Blank line separating nearby declarations or logic blocks.
  **L1035 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1036 EN**: Comment explains nearby logic, constraints, or intent: `Set content of FPSCR, and return its content before update`.
  **L1036 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set content of FPSCR, and return its content before update`。
- **L1037 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L1037 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1038 EN**: Blank line separating nearby declarations or logic blocks.
  **L1038 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1039 EN**: Comment explains nearby logic, constraints, or intent: `Cache built-ins`.
  **L1039 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Cache built-ins`。
- **L1040 EN**: Invokes macro `BUILTIN` to contribute one entry to a table-driven definition list.
  **L1040 CN**: 调用宏 `BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1041 EN**: Blank line separating nearby declarations or logic blocks.
  **L1041 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1042 EN**: Comment explains nearby logic, constraints, or intent: `Provided builtins with _mma_ prefix for compatibility.`.
  **L1042 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Provided builtins with _mma_ prefix for compatibility.`。
- **L1043 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1043 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1044 EN**: Continues the surrounding expression or declaration: `"paired-vector-memops")`.
  **L1044 CN**: 继续构造周围的表达式或声明：`"paired-vector-memops")`。
- **L1045 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1045 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1046 EN**: Continues the surrounding expression or declaration: `"paired-vector-memops")`.
  **L1046 CN**: 继续构造周围的表达式或声明：`"paired-vector-memops")`。
- **L1047 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1047 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1048 EN**: Continues the surrounding expression or declaration: `"paired-vector-memops")`.
  **L1048 CN**: 继续构造周围的表达式或声明：`"paired-vector-memops")`。
- **L1049 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1049 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1050 EN**: Continues the surrounding expression or declaration: `"paired-vector-memops")`.
  **L1050 CN**: 继续构造周围的表达式或声明：`"paired-vector-memops")`。
- **L1051 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1051 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1052 EN**: Continues the surrounding expression or declaration: `"paired-vector-memops")`.
  **L1052 CN**: 继续构造周围的表达式或声明：`"paired-vector-memops")`。
- **L1053 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1053 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1054 EN**: Blank line separating nearby declarations or logic blocks.
  **L1054 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1055 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1055 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1056 EN**: Continues the surrounding expression or declaration: `"paired-vector-memops")`.
  **L1056 CN**: 继续构造周围的表达式或声明：`"paired-vector-memops")`。

### Lines 1057-1080

````cpp
UNALIASED_CUSTOM_BUILTIN(vsx_stxvp, "vW256SLiW256*", false,
                         "paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(vsx_assemble_pair, "vW256*VV", false,
                         "paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(vsx_disassemble_pair, "vv*W256*", false,
                         "paired-vector-memops")

// TODO: Require only mma after backend supports these without paired memops
UNALIASED_CUSTOM_BUILTIN(mma_assemble_acc, "vW512*VVVV", false,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_disassemble_acc, "vv*W512*", false,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_xxmtacc, "vW512*", true,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_xxmfacc, "vW512*", true,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_xxsetaccz, "vW512*", false,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_xvi4ger8, "vW512*VV", false,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_xvi8ger4, "vW512*VV", false,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_xvi16ger2, "vW512*VV", false,
                         "mma,paired-vector-memops")
````
- **L1057 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1057 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1058 EN**: Continues the surrounding expression or declaration: `"paired-vector-memops")`.
  **L1058 CN**: 继续构造周围的表达式或声明：`"paired-vector-memops")`。
- **L1059 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1059 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1060 EN**: Continues the surrounding expression or declaration: `"paired-vector-memops")`.
  **L1060 CN**: 继续构造周围的表达式或声明：`"paired-vector-memops")`。
- **L1061 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1061 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1062 EN**: Continues the surrounding expression or declaration: `"paired-vector-memops")`.
  **L1062 CN**: 继续构造周围的表达式或声明：`"paired-vector-memops")`。
- **L1063 EN**: Blank line separating nearby declarations or logic blocks.
  **L1063 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1064 EN**: Comment records a pending task or caution: `TODO: Require only mma after backend supports these without paired memops`.
  **L1064 CN**: 注释记录待办事项或注意点：`TODO: Require only mma after backend supports these without paired memops`。
- **L1065 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1065 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1066 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1066 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1067 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1067 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1068 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1068 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1069 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1069 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1070 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1070 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1071 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1071 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1072 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1072 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1073 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1073 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1074 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1074 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1075 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1075 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1076 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1076 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1077 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1077 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1078 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1078 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1079 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1079 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1080 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1080 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。

### Lines 1081-1104

````cpp
UNALIASED_CUSTOM_BUILTIN(mma_xvi16ger2s, "vW512*VV", false,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_pmxvi4ger8, "vW512*VVi15i15i255", false,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_pmxvi8ger4, "vW512*VVi15i15i15", false,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_pmxvi16ger2, "vW512*VVi15i15i3", false,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_pmxvi16ger2s, "vW512*VVi15i15i3", false,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_xvi4ger8pp, "vW512*VV", true,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_xvi8ger4pp, "vW512*VV", true,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_xvi8ger4spp, "vW512*VV", true,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_xvi16ger2pp, "vW512*VV", true,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_xvi16ger2spp, "vW512*VV", true,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_pmxvi4ger8pp, "vW512*VVi15i15i255", true,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_pmxvi8ger4pp, "vW512*VVi15i15i15", true,
                         "mma,paired-vector-memops")
````
- **L1081 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1081 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1082 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1082 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1083 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1083 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1084 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1084 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1085 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1085 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1086 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1086 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1087 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1087 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1088 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1088 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1089 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1089 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1090 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1090 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1091 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1091 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1092 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1092 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1093 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1093 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1094 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1094 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1095 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1095 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1096 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1096 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1097 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1097 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1098 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1098 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1099 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1099 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1100 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1100 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1101 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1101 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1102 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1102 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1103 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1103 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1104 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1104 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。

### Lines 1105-1128

````cpp
UNALIASED_CUSTOM_BUILTIN(mma_pmxvi8ger4spp, "vW512*VVi15i15i15", true,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_pmxvi16ger2pp, "vW512*VVi15i15i3", true,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_pmxvi16ger2spp, "vW512*VVi15i15i3", true,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_dmxvi8gerx4, "vW1024*W256V", false,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_pmdmxvi8gerx4, "vW1024*W256Vi255i15i15", false,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_dmxvi8gerx4pp, "vW1024*W256V", true,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_pmdmxvi8gerx4pp, "vW1024*W256Vi255i15i15", true,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_dmxvi8gerx4spp, "vW1024*W256V", true,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(mma_pmdmxvi8gerx4spp, "vW1024*W256Vi255i15i15", true,
                         "mma,paired-vector-memops")
UNALIASED_CUSTOM_BUILTIN(dmsetdmrz, "vW1024*", false,
                         "mma,isa-future-instructions")
UNALIASED_CUSTOM_BUILTIN(dmmr, "vW1024*W1024*", false,
                         "mma,isa-future-instructions")
UNALIASED_CUSTOM_BUILTIN(dmxor, "vW1024*W1024*", true,
                         "mma,isa-future-instructions")
````
- **L1105 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1105 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1106 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1106 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1107 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1107 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1108 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1108 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1109 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1109 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1110 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1110 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1111 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1111 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1112 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1112 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1113 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1113 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1114 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1114 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1115 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1115 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1116 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1116 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1117 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1117 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1118 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1118 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1119 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1119 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1120 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1120 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1121 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1121 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1122 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1122 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1123 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1123 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1124 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1124 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1125 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1125 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1126 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1126 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1127 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1127 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1128 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1128 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。

### Lines 1129-1152

````cpp
UNALIASED_CUSTOM_BUILTIN(disassemble_dmr, "vv*W1024*", false,
                         "mma,isa-future-instructions")
UNALIASED_CUSTOM_BUILTIN(build_dmr, "vW1024*VVVVVVVV", false,
                         "mma,isa-future-instructions")

UNALIASED_CUSTOM_BUILTIN(dmsha2hash, "vW1024*W1024*i1", true,
                         "mma,isa-future-instructions")
UNALIASED_CUSTOM_BUILTIN(dmsha3hash, "vW2048*i31", true,
                         "mma,isa-future-instructions")
UNALIASED_CUSTOM_BUILTIN(dmxxshapad, "vW1024*Vi3i1i3", true,
                         "mma,isa-future-instructions")
CUSTOM_BUILTIN(dmsha256hash, dmsha2hash, "vW1024*W1024*", true,
                         "mma,isa-future-instructions")
CUSTOM_BUILTIN(dmsha512hash, dmsha2hash, "vW1024*W1024*", true,
                         "mma,isa-future-instructions")
CUSTOM_BUILTIN(dmsha3dw, dmsha3hash, "vW2048*", true,
                         "mma,isa-future-instructions")
CUSTOM_BUILTIN(dmcryshash, dmsha3hash, "vW2048*", true,
                         "mma,isa-future-instructions")
CUSTOM_BUILTIN(dmxxsha3512pad, dmxxshapad, "vW1024*Vi1", true,
                         "mma,isa-future-instructions")
CUSTOM_BUILTIN(dmxxsha3384pad, dmxxshapad, "vW1024*Vi1", true,
                         "mma,isa-future-instructions")
CUSTOM_BUILTIN(dmxxsha3256pad, dmxxshapad, "vW1024*Vi1", true,
````
- **L1129 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1129 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1130 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1130 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1131 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1131 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1132 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1132 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1133 EN**: Blank line separating nearby declarations or logic blocks.
  **L1133 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1134 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1134 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1135 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1135 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1136 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1136 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1137 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1137 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1138 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1138 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1139 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1139 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1140 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1140 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1141 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1141 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1142 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1142 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1143 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1143 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1144 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1144 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1145 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1145 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1146 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1146 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1147 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1147 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1148 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1148 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1149 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1149 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1150 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1150 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1151 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1151 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1152 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1152 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 1153-1176

````cpp
                         "mma,isa-future-instructions")
CUSTOM_BUILTIN(dmxxsha3224pad, dmxxshapad, "vW1024*Vi1", true,
                         "mma,isa-future-instructions")
CUSTOM_BUILTIN(dmxxshake256pad, dmxxshapad, "vW1024*Vi1", true,
                         "mma,isa-future-instructions")
CUSTOM_BUILTIN(dmxxshake128pad, dmxxshapad, "vW1024*Vi1", true,
                         "mma,isa-future-instructions")
CUSTOM_BUILTIN(dmxxsha384512pad, dmxxshapad, "vW1024*V", true,
                         "mma,isa-future-instructions")
CUSTOM_BUILTIN(dmxxsha224256pad, dmxxshapad, "vW1024*V", true,
                         "mma,isa-future-instructions")

// MMA builtins with positive/negative multiply/accumulate.
UNALIASED_CUSTOM_MMA_BUILTIN(mma_xvf16ger2, "vW512*VV",
                             "mma,paired-vector-memops")
UNALIASED_CUSTOM_MMA_BUILTIN(mma_xvf32ger, "vW512*VV",
                             "mma,paired-vector-memops")
UNALIASED_CUSTOM_MMA_BUILTIN(mma_xvf64ger, "vW512*W256V",
                             "mma,paired-vector-memops")
UNALIASED_CUSTOM_MMA_BUILTIN(mma_pmxvf16ger2, "vW512*VVi15i15i3",
                             "mma,paired-vector-memops")
UNALIASED_CUSTOM_MMA_BUILTIN(mma_pmxvf32ger, "vW512*VVi15i15",
                             "mma,paired-vector-memops")
UNALIASED_CUSTOM_MMA_BUILTIN(mma_pmxvf64ger, "vW512*W256Vi15i3",
````
- **L1153 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1153 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1154 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1154 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1155 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1155 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1156 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1156 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1157 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1157 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1158 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1158 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1159 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1159 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1160 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1160 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1161 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1161 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1162 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1162 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1163 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1163 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1164 EN**: Blank line separating nearby declarations or logic blocks.
  **L1164 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1165 EN**: Comment explains nearby logic, constraints, or intent: `MMA builtins with positive/negative multiply/accumulate.`.
  **L1165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`MMA builtins with positive/negative multiply/accumulate.`。
- **L1166 EN**: Invokes macro `UNALIASED_CUSTOM_MMA_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1166 CN**: 调用宏 `UNALIASED_CUSTOM_MMA_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1167 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1167 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1168 EN**: Invokes macro `UNALIASED_CUSTOM_MMA_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1168 CN**: 调用宏 `UNALIASED_CUSTOM_MMA_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1169 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1169 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1170 EN**: Invokes macro `UNALIASED_CUSTOM_MMA_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1170 CN**: 调用宏 `UNALIASED_CUSTOM_MMA_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1171 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1171 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1172 EN**: Invokes macro `UNALIASED_CUSTOM_MMA_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1172 CN**: 调用宏 `UNALIASED_CUSTOM_MMA_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1173 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1173 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1174 EN**: Invokes macro `UNALIASED_CUSTOM_MMA_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1174 CN**: 调用宏 `UNALIASED_CUSTOM_MMA_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1175 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1175 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1176 EN**: Invokes macro `UNALIASED_CUSTOM_MMA_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1176 CN**: 调用宏 `UNALIASED_CUSTOM_MMA_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 1177-1200

````cpp
                             "mma,paired-vector-memops")
UNALIASED_CUSTOM_MMA_BUILTIN(mma_xvbf16ger2, "vW512*VV",
                             "mma,paired-vector-memops")
UNALIASED_CUSTOM_MMA_BUILTIN(mma_pmxvbf16ger2, "vW512*VVi15i15i3",
                             "mma,paired-vector-memops")
UNALIASED_CUSTOM_MMA_BUILTIN(mma_dmxvbf16gerx2, "vW1024*W256V",
                             "mma,isa-future-instructions")
UNALIASED_CUSTOM_MMA_BUILTIN(mma_pmdmxvbf16gerx2, "vW1024*W256Vi255i15i3",
                             "mma,isa-future-instructions")
UNALIASED_CUSTOM_MMA_BUILTIN(mma_dmxvf16gerx2, "vW1024*W256V",
                             "mma,isa-future-instructions")
UNALIASED_CUSTOM_MMA_BUILTIN(mma_pmdmxvf16gerx2, "vW1024*W256Vi255i15i3",
                             "mma,isa-future-instructions")

// Elliptic Curve Cryptography Builtins
UNALIASED_CUSTOM_BUILTIN(xxmulmul, "VVVi7", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xxmulmulhiadd, "VVVi1i1i1", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xxmulmulloadd, "VVVi1i1", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xxssumudm, "VVVi1", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xxssumudmc, "VVVi1", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xxssumudmcext, "VVVVi1", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xsaddadduqm, "VVV", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xsaddaddsuqm, "VVV", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xsaddsubuqm, "VVV", false, "future-vector")
````
- **L1177 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1177 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1178 EN**: Invokes macro `UNALIASED_CUSTOM_MMA_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1178 CN**: 调用宏 `UNALIASED_CUSTOM_MMA_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1179 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1179 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1180 EN**: Invokes macro `UNALIASED_CUSTOM_MMA_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1180 CN**: 调用宏 `UNALIASED_CUSTOM_MMA_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1181 EN**: Continues the surrounding expression or declaration: `"mma,paired-vector-memops")`.
  **L1181 CN**: 继续构造周围的表达式或声明：`"mma,paired-vector-memops")`。
- **L1182 EN**: Invokes macro `UNALIASED_CUSTOM_MMA_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1182 CN**: 调用宏 `UNALIASED_CUSTOM_MMA_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1183 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1183 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1184 EN**: Invokes macro `UNALIASED_CUSTOM_MMA_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1184 CN**: 调用宏 `UNALIASED_CUSTOM_MMA_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1185 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1185 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1186 EN**: Invokes macro `UNALIASED_CUSTOM_MMA_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1186 CN**: 调用宏 `UNALIASED_CUSTOM_MMA_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1187 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1187 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1188 EN**: Invokes macro `UNALIASED_CUSTOM_MMA_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1188 CN**: 调用宏 `UNALIASED_CUSTOM_MMA_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1189 EN**: Continues the surrounding expression or declaration: `"mma,isa-future-instructions")`.
  **L1189 CN**: 继续构造周围的表达式或声明：`"mma,isa-future-instructions")`。
- **L1190 EN**: Blank line separating nearby declarations or logic blocks.
  **L1190 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1191 EN**: Comment explains nearby logic, constraints, or intent: `Elliptic Curve Cryptography Builtins`.
  **L1191 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Elliptic Curve Cryptography Builtins`。
- **L1192 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1192 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1193 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1193 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1194 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1194 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1195 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1195 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1196 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1196 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1197 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1197 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1198 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1198 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1199 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1199 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1200 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1200 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 1201-1224

````cpp
UNALIASED_CUSTOM_BUILTIN(xsaddsubsuqm, "VVV", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xsmerge2t1uqm, "VVV", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xsmerge2t2uqm, "VVV", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xsmerge2t3uqm, "VVV", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xsmerge3t1uqm, "VVV", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xsrebase2t1uqm, "VVV", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xsrebase2t2uqm, "VVV", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xsrebase2t3uqm, "VVV", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xsrebase2t4uqm, "VVV", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xsrebase3t1uqm, "VVV", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xsrebase3t2uqm, "VVV", false, "future-vector")
UNALIASED_CUSTOM_BUILTIN(xsrebase3t3uqm, "VVV", false, "future-vector")

// Deeply Compressed Weights built-ins.
TARGET_BUILTIN(__builtin_altivec_vucmprhn, "V16UcV16UcV16Uc", "",
               "isa-future-instructions")
TARGET_BUILTIN(__builtin_altivec_vucmprln, "V16UcV16UcV16Uc", "",
               "isa-future-instructions")
TARGET_BUILTIN(__builtin_altivec_vucmprhb, "V16UcV16UcV16Uc", "",
               "isa-future-instructions")
TARGET_BUILTIN(__builtin_altivec_vucmprlb, "V16UcV16UcV16Uc", "",
               "isa-future-instructions")
TARGET_BUILTIN(__builtin_altivec_vucmprhh, "V16UcV16UcV16Uc", "",
               "isa-future-instructions")
````
- **L1201 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1201 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1202 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1202 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1203 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1203 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1204 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1204 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1205 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1205 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1206 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1206 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1207 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1207 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1208 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1208 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1209 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1209 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1210 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1210 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1211 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1211 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1212 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1212 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1213 EN**: Blank line separating nearby declarations or logic blocks.
  **L1213 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1214 EN**: Comment explains nearby logic, constraints, or intent: `Deeply Compressed Weights built-ins.`.
  **L1214 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Deeply Compressed Weights built-ins.`。
- **L1215 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1215 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1216 EN**: Continues the surrounding expression or declaration: `"isa-future-instructions")`.
  **L1216 CN**: 继续构造周围的表达式或声明：`"isa-future-instructions")`。
- **L1217 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1217 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1218 EN**: Continues the surrounding expression or declaration: `"isa-future-instructions")`.
  **L1218 CN**: 继续构造周围的表达式或声明：`"isa-future-instructions")`。
- **L1219 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1219 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1220 EN**: Continues the surrounding expression or declaration: `"isa-future-instructions")`.
  **L1220 CN**: 继续构造周围的表达式或声明：`"isa-future-instructions")`。
- **L1221 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1221 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1222 EN**: Continues the surrounding expression or declaration: `"isa-future-instructions")`.
  **L1222 CN**: 继续构造周围的表达式或声明：`"isa-future-instructions")`。
- **L1223 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1223 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1224 EN**: Continues the surrounding expression or declaration: `"isa-future-instructions")`.
  **L1224 CN**: 继续构造周围的表达式或声明：`"isa-future-instructions")`。

### Lines 1225-1248

````cpp
TARGET_BUILTIN(__builtin_altivec_vucmprlh, "V16UcV16UcV16Uc", "",
               "isa-future-instructions")
TARGET_BUILTIN(__builtin_altivec_vupkhsntob, "V16UcV16Uc", "",
               "isa-future-instructions")
TARGET_BUILTIN(__builtin_altivec_vupklsntob, "V16UcV16Uc", "",
               "isa-future-instructions")
TARGET_BUILTIN(__builtin_altivec_vupkint4tobf16, "V16UcV16UcIi", "",
               "isa-future-instructions")
TARGET_BUILTIN(__builtin_altivec_vupkint8tobf16, "V16UcV16UcIi", "",
               "isa-future-instructions")
TARGET_BUILTIN(__builtin_altivec_vupkint4tofp32, "V16UcV16UcIi", "",
               "isa-future-instructions")
TARGET_BUILTIN(__builtin_altivec_vupkint8tofp32, "V16UcV16UcIi", "",
               "isa-future-instructions")

// AES Encrypt Paired builtins
UNALIASED_CUSTOM_BUILTIN(aes_encrypt_paired, "W256W256W256i2", false,
                         "future-vector,paired-vector-memops")
CUSTOM_BUILTIN(aes128_encrypt_paired, aes_encrypt_paired, "W256W256W256", false,
               "future-vector,paired-vector-memops")
CUSTOM_BUILTIN(aes192_encrypt_paired, aes_encrypt_paired, "W256W256W256", false,
               "future-vector,paired-vector-memops")
CUSTOM_BUILTIN(aes256_encrypt_paired, aes_encrypt_paired, "W256W256W256", false,
               "future-vector,paired-vector-memops")
````
- **L1225 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1225 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1226 EN**: Continues the surrounding expression or declaration: `"isa-future-instructions")`.
  **L1226 CN**: 继续构造周围的表达式或声明：`"isa-future-instructions")`。
- **L1227 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1227 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1228 EN**: Continues the surrounding expression or declaration: `"isa-future-instructions")`.
  **L1228 CN**: 继续构造周围的表达式或声明：`"isa-future-instructions")`。
- **L1229 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1229 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1230 EN**: Continues the surrounding expression or declaration: `"isa-future-instructions")`.
  **L1230 CN**: 继续构造周围的表达式或声明：`"isa-future-instructions")`。
- **L1231 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1231 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1232 EN**: Continues the surrounding expression or declaration: `"isa-future-instructions")`.
  **L1232 CN**: 继续构造周围的表达式或声明：`"isa-future-instructions")`。
- **L1233 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1233 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1234 EN**: Continues the surrounding expression or declaration: `"isa-future-instructions")`.
  **L1234 CN**: 继续构造周围的表达式或声明：`"isa-future-instructions")`。
- **L1235 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1235 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1236 EN**: Continues the surrounding expression or declaration: `"isa-future-instructions")`.
  **L1236 CN**: 继续构造周围的表达式或声明：`"isa-future-instructions")`。
- **L1237 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1237 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1238 EN**: Continues the surrounding expression or declaration: `"isa-future-instructions")`.
  **L1238 CN**: 继续构造周围的表达式或声明：`"isa-future-instructions")`。
- **L1239 EN**: Blank line separating nearby declarations or logic blocks.
  **L1239 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1240 EN**: Comment explains nearby logic, constraints, or intent: `AES Encrypt Paired builtins`.
  **L1240 CN**: 注释解释附近代码的逻辑、约束或设计意图：`AES Encrypt Paired builtins`。
- **L1241 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1241 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1242 EN**: Continues the surrounding expression or declaration: `"future-vector,paired-vector-memops")`.
  **L1242 CN**: 继续构造周围的表达式或声明：`"future-vector,paired-vector-memops")`。
- **L1243 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1243 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1244 EN**: Continues the surrounding expression or declaration: `"future-vector,paired-vector-memops")`.
  **L1244 CN**: 继续构造周围的表达式或声明：`"future-vector,paired-vector-memops")`。
- **L1245 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1245 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1246 EN**: Continues the surrounding expression or declaration: `"future-vector,paired-vector-memops")`.
  **L1246 CN**: 继续构造周围的表达式或声明：`"future-vector,paired-vector-memops")`。
- **L1247 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1247 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1248 EN**: Continues the surrounding expression or declaration: `"future-vector,paired-vector-memops")`.
  **L1248 CN**: 继续构造周围的表达式或声明：`"future-vector,paired-vector-memops")`。

### Lines 1249-1272

````cpp

// AES Decrypt Paired builtins
UNALIASED_CUSTOM_BUILTIN(aes_decrypt_paired, "W256W256W256i2", false,
                         "future-vector,paired-vector-memops")
CUSTOM_BUILTIN(aes128_decrypt_paired, aes_decrypt_paired, "W256W256W256", false,
               "future-vector,paired-vector-memops")
CUSTOM_BUILTIN(aes192_decrypt_paired, aes_decrypt_paired, "W256W256W256", false,
               "future-vector,paired-vector-memops")
CUSTOM_BUILTIN(aes256_decrypt_paired, aes_decrypt_paired, "W256W256W256", false,
               "future-vector,paired-vector-memops")

// AES Generate Last Key Paired builtins
UNALIASED_CUSTOM_BUILTIN(aes_genlastkey_paired, "W256W256i2", false,
                         "future-vector,paired-vector-memops")
CUSTOM_BUILTIN(aes128_genlastkey_paired, aes_genlastkey_paired, "W256W256", false,
               "future-vector,paired-vector-memops")
CUSTOM_BUILTIN(aes192_genlastkey_paired, aes_genlastkey_paired, "W256W256", false,
               "future-vector,paired-vector-memops")
CUSTOM_BUILTIN(aes256_genlastkey_paired, aes_genlastkey_paired, "W256W256", false,
               "future-vector,paired-vector-memops")

// Galois Field Multiplication builtins
UNALIASED_CUSTOM_BUILTIN(galois_field_mult, "VVVi1", false,
                         "future-vector")
````
- **L1249 EN**: Blank line separating nearby declarations or logic blocks.
  **L1249 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1250 EN**: Comment explains nearby logic, constraints, or intent: `AES Decrypt Paired builtins`.
  **L1250 CN**: 注释解释附近代码的逻辑、约束或设计意图：`AES Decrypt Paired builtins`。
- **L1251 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1251 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1252 EN**: Continues the surrounding expression or declaration: `"future-vector,paired-vector-memops")`.
  **L1252 CN**: 继续构造周围的表达式或声明：`"future-vector,paired-vector-memops")`。
- **L1253 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1253 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1254 EN**: Continues the surrounding expression or declaration: `"future-vector,paired-vector-memops")`.
  **L1254 CN**: 继续构造周围的表达式或声明：`"future-vector,paired-vector-memops")`。
- **L1255 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1255 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1256 EN**: Continues the surrounding expression or declaration: `"future-vector,paired-vector-memops")`.
  **L1256 CN**: 继续构造周围的表达式或声明：`"future-vector,paired-vector-memops")`。
- **L1257 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1257 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1258 EN**: Continues the surrounding expression or declaration: `"future-vector,paired-vector-memops")`.
  **L1258 CN**: 继续构造周围的表达式或声明：`"future-vector,paired-vector-memops")`。
- **L1259 EN**: Blank line separating nearby declarations or logic blocks.
  **L1259 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1260 EN**: Comment explains nearby logic, constraints, or intent: `AES Generate Last Key Paired builtins`.
  **L1260 CN**: 注释解释附近代码的逻辑、约束或设计意图：`AES Generate Last Key Paired builtins`。
- **L1261 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1261 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1262 EN**: Continues the surrounding expression or declaration: `"future-vector,paired-vector-memops")`.
  **L1262 CN**: 继续构造周围的表达式或声明：`"future-vector,paired-vector-memops")`。
- **L1263 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1263 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1264 EN**: Continues the surrounding expression or declaration: `"future-vector,paired-vector-memops")`.
  **L1264 CN**: 继续构造周围的表达式或声明：`"future-vector,paired-vector-memops")`。
- **L1265 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1265 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1266 EN**: Continues the surrounding expression or declaration: `"future-vector,paired-vector-memops")`.
  **L1266 CN**: 继续构造周围的表达式或声明：`"future-vector,paired-vector-memops")`。
- **L1267 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1267 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1268 EN**: Continues the surrounding expression or declaration: `"future-vector,paired-vector-memops")`.
  **L1268 CN**: 继续构造周围的表达式或声明：`"future-vector,paired-vector-memops")`。
- **L1269 EN**: Blank line separating nearby declarations or logic blocks.
  **L1269 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1270 EN**: Comment explains nearby logic, constraints, or intent: `Galois Field Multiplication builtins`.
  **L1270 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Galois Field Multiplication builtins`。
- **L1271 EN**: Invokes macro `UNALIASED_CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1271 CN**: 调用宏 `UNALIASED_CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1272 EN**: Continues the surrounding expression or declaration: `"future-vector")`.
  **L1272 CN**: 继续构造周围的表达式或声明：`"future-vector")`。

### Lines 1273-1284

````cpp
CUSTOM_BUILTIN(galois_field_mult_gcm, galois_field_mult, "VVV", false,
               "future-vector")
CUSTOM_BUILTIN(galois_field_mult_xts, galois_field_mult, "VVV", false,
               "future-vector")

// FIXME: Obviously incomplete.

#undef BUILTIN
#undef TARGET_BUILTIN
#undef CUSTOM_BUILTIN
#undef UNALIASED_CUSTOM_BUILTIN
#undef UNALIASED_CUSTOM_MMA_BUILTIN
````
- **L1273 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1273 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1274 EN**: Continues the surrounding expression or declaration: `"future-vector")`.
  **L1274 CN**: 继续构造周围的表达式或声明：`"future-vector")`。
- **L1275 EN**: Invokes macro `CUSTOM_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1275 CN**: 调用宏 `CUSTOM_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1276 EN**: Continues the surrounding expression or declaration: `"future-vector")`.
  **L1276 CN**: 继续构造周围的表达式或声明：`"future-vector")`。
- **L1277 EN**: Blank line separating nearby declarations or logic blocks.
  **L1277 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1278 EN**: Comment records a pending task or caution: `FIXME: Obviously incomplete.`.
  **L1278 CN**: 注释记录待办事项或注意点：`FIXME: Obviously incomplete.`。
- **L1279 EN**: Blank line separating nearby declarations or logic blocks.
  **L1279 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1280 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef BUILTIN`.
  **L1280 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef BUILTIN`。
- **L1281 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef TARGET_BUILTIN`.
  **L1281 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef TARGET_BUILTIN`。
- **L1282 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef CUSTOM_BUILTIN`.
  **L1282 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef CUSTOM_BUILTIN`。
- **L1283 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef UNALIASED_CUSTOM_BUILTIN`.
  **L1283 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef UNALIASED_CUSTOM_BUILTIN`。
- **L1284 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef UNALIASED_CUSTOM_MMA_BUILTIN`.
  **L1284 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef UNALIASED_CUSTOM_MMA_BUILTIN`。

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
- **Macro-driven generation / 宏驱动生成**
  - **EN**: Relies on macros or generated include fragments to keep large definition sets synchronized.
  - **CN**: 依赖宏或生成式 include 片段来同步大规模定义集合。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: `BUILTIN(ID,`, `TARGET_BUILTIN(ID,`, `CUSTOM_BUILTIN(ID,`, `UNALIASED_CUSTOM_BUILTIN(ID,`, `UNALIASED_CUSTOM_MMA_BUILTIN(ID,`
- **Types / 类型**: No obvious type declarations detected by the generator. / 生成器未检测到明显类型声明。
- **Functions or callables / 函数或可调用对象**: `ins`, `TARGET_BUILTIN`, `CUSTOM_BUILTIN`, `__builtin_mma_xvf16ger2`, `__builtin_mma_xvf16ger2nn`, `__builtin_mma_xvf16ger2np`, `__builtin_mma_xvf16ger2pn`, `UNALIASED_CUSTOM_BUILTIN`, `BUILTIN`, `decimal`, `double`, `int128`
- **TableGen records / TableGen 记录**: No prominent TableGen record names detected. / 未检测到明显的 TableGen 记录名称。
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
