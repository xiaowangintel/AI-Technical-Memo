# hexagon_types.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/hexagon_types.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This header supplies hexagon dsp intrinsics for `hexagon_types` inside Clang's public header set.
- **Purpose (CN)**: 该头文件在 Clang 公共头文件集合中为 `hexagon_types` 提供Hexagon DSP intrinsic 接口。
- **Line Count / 行数**: 2628

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
//===----------------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef HEXAGON_TYPES_H
#define HEXAGON_TYPES_H

#include <hexagon_protos.h>

// Save and undefine B0 to avoid conflicts with POSIX termios.h which
// defines B0 as a macro for baud rate 0.
#pragma push_macro("B0")
#undef B0

/* Hexagon names */
#define HEXAGON_Vect HEXAGON_Vect64
#define HEXAGON_V_GET_D HEXAGON_V64_GET_D
#define HEXAGON_V_GET_UD HEXAGON_V64_GET_UD
#define HEXAGON_V_GET_W0 HEXAGON_V64_GET_W0
#define HEXAGON_V_GET_W1 HEXAGON_V64_GET_W1
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
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L9 EN**: Starts a preprocessor conditional block: `#ifndef HEXAGON_TYPES_H`.
  **L9 CN**: 开始一个预处理条件块：`#ifndef HEXAGON_TYPES_H`。
- **L10 EN**: Defines macro `HEXAGON_TYPES_H` for conditional compilation, shorthand, or API generation.
  **L10 CN**: 定义宏 `HEXAGON_TYPES_H`，用于条件编译、简写或 API 生成。
- **L11 EN**: Blank line separating nearby declarations or logic blocks.
  **L11 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L12 EN**: Includes <hexagon_protos.h> to access related header declarations.
  **L12 CN**: 引入 <hexagon_protos.h> 以使用相关头文件声明。
- **L13 EN**: Blank line separating nearby declarations or logic blocks.
  **L13 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L14 EN**: Comment explains nearby logic, constraints, or intent: `Save and undefine B0 to avoid conflicts with POSIX termios.h which`.
  **L14 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Save and undefine B0 to avoid conflicts with POSIX termios.h which`。
- **L15 EN**: Comment explains nearby logic, constraints, or intent: `defines B0 as a macro for baud rate 0.`.
  **L15 CN**: 注释解释附近代码的逻辑、约束或设计意图：`defines B0 as a macro for baud rate 0.`。
- **L16 EN**: Applies a compiler pragma that adjusts parsing, attributes, or macro state: `#pragma push_macro("B0")`.
  **L16 CN**: 应用编译器 pragma，调整解析、属性或宏状态：`#pragma push_macro("B0")`。
- **L17 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef B0`.
  **L17 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef B0`。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L19 EN**: Comment explains nearby logic, constraints, or intent: `Hexagon names`.
  **L19 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Hexagon names`。
- **L20 EN**: Defines macro `HEXAGON_Vect` for conditional compilation, shorthand, or API generation.
  **L20 CN**: 定义宏 `HEXAGON_Vect`，用于条件编译、简写或 API 生成。
- **L21 EN**: Defines macro `HEXAGON_V_GET_D` for conditional compilation, shorthand, or API generation.
  **L21 CN**: 定义宏 `HEXAGON_V_GET_D`，用于条件编译、简写或 API 生成。
- **L22 EN**: Defines macro `HEXAGON_V_GET_UD` for conditional compilation, shorthand, or API generation.
  **L22 CN**: 定义宏 `HEXAGON_V_GET_UD`，用于条件编译、简写或 API 生成。
- **L23 EN**: Defines macro `HEXAGON_V_GET_W0` for conditional compilation, shorthand, or API generation.
  **L23 CN**: 定义宏 `HEXAGON_V_GET_W0`，用于条件编译、简写或 API 生成。
- **L24 EN**: Defines macro `HEXAGON_V_GET_W1` for conditional compilation, shorthand, or API generation.
  **L24 CN**: 定义宏 `HEXAGON_V_GET_W1`，用于条件编译、简写或 API 生成。

### Lines 25-48

````c
#define HEXAGON_V_GET_UW0 HEXAGON_V64_GET_UW0
#define HEXAGON_V_GET_UW1 HEXAGON_V64_GET_UW1
#define HEXAGON_V_GET_H0 HEXAGON_V64_GET_H0
#define HEXAGON_V_GET_H1 HEXAGON_V64_GET_H1
#define HEXAGON_V_GET_H2 HEXAGON_V64_GET_H2
#define HEXAGON_V_GET_H3 HEXAGON_V64_GET_H3
#define HEXAGON_V_GET_UH0 HEXAGON_V64_GET_UH0
#define HEXAGON_V_GET_UH1 HEXAGON_V64_GET_UH1
#define HEXAGON_V_GET_UH2 HEXAGON_V64_GET_UH2
#define HEXAGON_V_GET_UH3 HEXAGON_V64_GET_UH3
#define HEXAGON_V_GET_B0 HEXAGON_V64_GET_B0
#define HEXAGON_V_GET_B1 HEXAGON_V64_GET_B1
#define HEXAGON_V_GET_B2 HEXAGON_V64_GET_B2
#define HEXAGON_V_GET_B3 HEXAGON_V64_GET_B3
#define HEXAGON_V_GET_B4 HEXAGON_V64_GET_B4
#define HEXAGON_V_GET_B5 HEXAGON_V64_GET_B5
#define HEXAGON_V_GET_B6 HEXAGON_V64_GET_B6
#define HEXAGON_V_GET_B7 HEXAGON_V64_GET_B7
#define HEXAGON_V_GET_UB0 HEXAGON_V64_GET_UB0
#define HEXAGON_V_GET_UB1 HEXAGON_V64_GET_UB1
#define HEXAGON_V_GET_UB2 HEXAGON_V64_GET_UB2
#define HEXAGON_V_GET_UB3 HEXAGON_V64_GET_UB3
#define HEXAGON_V_GET_UB4 HEXAGON_V64_GET_UB4
#define HEXAGON_V_GET_UB5 HEXAGON_V64_GET_UB5
````
- **L25 EN**: Defines macro `HEXAGON_V_GET_UW0` for conditional compilation, shorthand, or API generation.
  **L25 CN**: 定义宏 `HEXAGON_V_GET_UW0`，用于条件编译、简写或 API 生成。
- **L26 EN**: Defines macro `HEXAGON_V_GET_UW1` for conditional compilation, shorthand, or API generation.
  **L26 CN**: 定义宏 `HEXAGON_V_GET_UW1`，用于条件编译、简写或 API 生成。
- **L27 EN**: Defines macro `HEXAGON_V_GET_H0` for conditional compilation, shorthand, or API generation.
  **L27 CN**: 定义宏 `HEXAGON_V_GET_H0`，用于条件编译、简写或 API 生成。
- **L28 EN**: Defines macro `HEXAGON_V_GET_H1` for conditional compilation, shorthand, or API generation.
  **L28 CN**: 定义宏 `HEXAGON_V_GET_H1`，用于条件编译、简写或 API 生成。
- **L29 EN**: Defines macro `HEXAGON_V_GET_H2` for conditional compilation, shorthand, or API generation.
  **L29 CN**: 定义宏 `HEXAGON_V_GET_H2`，用于条件编译、简写或 API 生成。
- **L30 EN**: Defines macro `HEXAGON_V_GET_H3` for conditional compilation, shorthand, or API generation.
  **L30 CN**: 定义宏 `HEXAGON_V_GET_H3`，用于条件编译、简写或 API 生成。
- **L31 EN**: Defines macro `HEXAGON_V_GET_UH0` for conditional compilation, shorthand, or API generation.
  **L31 CN**: 定义宏 `HEXAGON_V_GET_UH0`，用于条件编译、简写或 API 生成。
- **L32 EN**: Defines macro `HEXAGON_V_GET_UH1` for conditional compilation, shorthand, or API generation.
  **L32 CN**: 定义宏 `HEXAGON_V_GET_UH1`，用于条件编译、简写或 API 生成。
- **L33 EN**: Defines macro `HEXAGON_V_GET_UH2` for conditional compilation, shorthand, or API generation.
  **L33 CN**: 定义宏 `HEXAGON_V_GET_UH2`，用于条件编译、简写或 API 生成。
- **L34 EN**: Defines macro `HEXAGON_V_GET_UH3` for conditional compilation, shorthand, or API generation.
  **L34 CN**: 定义宏 `HEXAGON_V_GET_UH3`，用于条件编译、简写或 API 生成。
- **L35 EN**: Defines macro `HEXAGON_V_GET_B0` for conditional compilation, shorthand, or API generation.
  **L35 CN**: 定义宏 `HEXAGON_V_GET_B0`，用于条件编译、简写或 API 生成。
- **L36 EN**: Defines macro `HEXAGON_V_GET_B1` for conditional compilation, shorthand, or API generation.
  **L36 CN**: 定义宏 `HEXAGON_V_GET_B1`，用于条件编译、简写或 API 生成。
- **L37 EN**: Defines macro `HEXAGON_V_GET_B2` for conditional compilation, shorthand, or API generation.
  **L37 CN**: 定义宏 `HEXAGON_V_GET_B2`，用于条件编译、简写或 API 生成。
- **L38 EN**: Defines macro `HEXAGON_V_GET_B3` for conditional compilation, shorthand, or API generation.
  **L38 CN**: 定义宏 `HEXAGON_V_GET_B3`，用于条件编译、简写或 API 生成。
- **L39 EN**: Defines macro `HEXAGON_V_GET_B4` for conditional compilation, shorthand, or API generation.
  **L39 CN**: 定义宏 `HEXAGON_V_GET_B4`，用于条件编译、简写或 API 生成。
- **L40 EN**: Defines macro `HEXAGON_V_GET_B5` for conditional compilation, shorthand, or API generation.
  **L40 CN**: 定义宏 `HEXAGON_V_GET_B5`，用于条件编译、简写或 API 生成。
- **L41 EN**: Defines macro `HEXAGON_V_GET_B6` for conditional compilation, shorthand, or API generation.
  **L41 CN**: 定义宏 `HEXAGON_V_GET_B6`，用于条件编译、简写或 API 生成。
- **L42 EN**: Defines macro `HEXAGON_V_GET_B7` for conditional compilation, shorthand, or API generation.
  **L42 CN**: 定义宏 `HEXAGON_V_GET_B7`，用于条件编译、简写或 API 生成。
- **L43 EN**: Defines macro `HEXAGON_V_GET_UB0` for conditional compilation, shorthand, or API generation.
  **L43 CN**: 定义宏 `HEXAGON_V_GET_UB0`，用于条件编译、简写或 API 生成。
- **L44 EN**: Defines macro `HEXAGON_V_GET_UB1` for conditional compilation, shorthand, or API generation.
  **L44 CN**: 定义宏 `HEXAGON_V_GET_UB1`，用于条件编译、简写或 API 生成。
- **L45 EN**: Defines macro `HEXAGON_V_GET_UB2` for conditional compilation, shorthand, or API generation.
  **L45 CN**: 定义宏 `HEXAGON_V_GET_UB2`，用于条件编译、简写或 API 生成。
- **L46 EN**: Defines macro `HEXAGON_V_GET_UB3` for conditional compilation, shorthand, or API generation.
  **L46 CN**: 定义宏 `HEXAGON_V_GET_UB3`，用于条件编译、简写或 API 生成。
- **L47 EN**: Defines macro `HEXAGON_V_GET_UB4` for conditional compilation, shorthand, or API generation.
  **L47 CN**: 定义宏 `HEXAGON_V_GET_UB4`，用于条件编译、简写或 API 生成。
- **L48 EN**: Defines macro `HEXAGON_V_GET_UB5` for conditional compilation, shorthand, or API generation.
  **L48 CN**: 定义宏 `HEXAGON_V_GET_UB5`，用于条件编译、简写或 API 生成。

### Lines 49-72

````c
#define HEXAGON_V_GET_UB6 HEXAGON_V64_GET_UB6
#define HEXAGON_V_GET_UB7 HEXAGON_V64_GET_UB7
#define HEXAGON_V_PUT_D HEXAGON_V64_PUT_D
#define HEXAGON_V_PUT_W0 HEXAGON_V64_PUT_W0
#define HEXAGON_V_PUT_W1 HEXAGON_V64_PUT_W1
#define HEXAGON_V_PUT_H0 HEXAGON_V64_PUT_H0
#define HEXAGON_V_PUT_H1 HEXAGON_V64_PUT_H1
#define HEXAGON_V_PUT_H2 HEXAGON_V64_PUT_H2
#define HEXAGON_V_PUT_H3 HEXAGON_V64_PUT_H3
#define HEXAGON_V_PUT_B0 HEXAGON_V64_PUT_B0
#define HEXAGON_V_PUT_B1 HEXAGON_V64_PUT_B1
#define HEXAGON_V_PUT_B2 HEXAGON_V64_PUT_B2
#define HEXAGON_V_PUT_B3 HEXAGON_V64_PUT_B3
#define HEXAGON_V_PUT_B4 HEXAGON_V64_PUT_B4
#define HEXAGON_V_PUT_B5 HEXAGON_V64_PUT_B5
#define HEXAGON_V_PUT_B6 HEXAGON_V64_PUT_B6
#define HEXAGON_V_PUT_B7 HEXAGON_V64_PUT_B7
#define HEXAGON_V_CREATE_D HEXAGON_V64_CREATE_D
#define HEXAGON_V_CREATE_W HEXAGON_V64_CREATE_W
#define HEXAGON_V_CREATE_H HEXAGON_V64_CREATE_H
#define HEXAGON_V_CREATE_B HEXAGON_V64_CREATE_B

#ifdef __cplusplus
#define HEXAGON_VectC HEXAGON_Vect64C
````
- **L49 EN**: Defines macro `HEXAGON_V_GET_UB6` for conditional compilation, shorthand, or API generation.
  **L49 CN**: 定义宏 `HEXAGON_V_GET_UB6`，用于条件编译、简写或 API 生成。
- **L50 EN**: Defines macro `HEXAGON_V_GET_UB7` for conditional compilation, shorthand, or API generation.
  **L50 CN**: 定义宏 `HEXAGON_V_GET_UB7`，用于条件编译、简写或 API 生成。
- **L51 EN**: Defines macro `HEXAGON_V_PUT_D` for conditional compilation, shorthand, or API generation.
  **L51 CN**: 定义宏 `HEXAGON_V_PUT_D`，用于条件编译、简写或 API 生成。
- **L52 EN**: Defines macro `HEXAGON_V_PUT_W0` for conditional compilation, shorthand, or API generation.
  **L52 CN**: 定义宏 `HEXAGON_V_PUT_W0`，用于条件编译、简写或 API 生成。
- **L53 EN**: Defines macro `HEXAGON_V_PUT_W1` for conditional compilation, shorthand, or API generation.
  **L53 CN**: 定义宏 `HEXAGON_V_PUT_W1`，用于条件编译、简写或 API 生成。
- **L54 EN**: Defines macro `HEXAGON_V_PUT_H0` for conditional compilation, shorthand, or API generation.
  **L54 CN**: 定义宏 `HEXAGON_V_PUT_H0`，用于条件编译、简写或 API 生成。
- **L55 EN**: Defines macro `HEXAGON_V_PUT_H1` for conditional compilation, shorthand, or API generation.
  **L55 CN**: 定义宏 `HEXAGON_V_PUT_H1`，用于条件编译、简写或 API 生成。
- **L56 EN**: Defines macro `HEXAGON_V_PUT_H2` for conditional compilation, shorthand, or API generation.
  **L56 CN**: 定义宏 `HEXAGON_V_PUT_H2`，用于条件编译、简写或 API 生成。
- **L57 EN**: Defines macro `HEXAGON_V_PUT_H3` for conditional compilation, shorthand, or API generation.
  **L57 CN**: 定义宏 `HEXAGON_V_PUT_H3`，用于条件编译、简写或 API 生成。
- **L58 EN**: Defines macro `HEXAGON_V_PUT_B0` for conditional compilation, shorthand, or API generation.
  **L58 CN**: 定义宏 `HEXAGON_V_PUT_B0`，用于条件编译、简写或 API 生成。
- **L59 EN**: Defines macro `HEXAGON_V_PUT_B1` for conditional compilation, shorthand, or API generation.
  **L59 CN**: 定义宏 `HEXAGON_V_PUT_B1`，用于条件编译、简写或 API 生成。
- **L60 EN**: Defines macro `HEXAGON_V_PUT_B2` for conditional compilation, shorthand, or API generation.
  **L60 CN**: 定义宏 `HEXAGON_V_PUT_B2`，用于条件编译、简写或 API 生成。
- **L61 EN**: Defines macro `HEXAGON_V_PUT_B3` for conditional compilation, shorthand, or API generation.
  **L61 CN**: 定义宏 `HEXAGON_V_PUT_B3`，用于条件编译、简写或 API 生成。
- **L62 EN**: Defines macro `HEXAGON_V_PUT_B4` for conditional compilation, shorthand, or API generation.
  **L62 CN**: 定义宏 `HEXAGON_V_PUT_B4`，用于条件编译、简写或 API 生成。
- **L63 EN**: Defines macro `HEXAGON_V_PUT_B5` for conditional compilation, shorthand, or API generation.
  **L63 CN**: 定义宏 `HEXAGON_V_PUT_B5`，用于条件编译、简写或 API 生成。
- **L64 EN**: Defines macro `HEXAGON_V_PUT_B6` for conditional compilation, shorthand, or API generation.
  **L64 CN**: 定义宏 `HEXAGON_V_PUT_B6`，用于条件编译、简写或 API 生成。
- **L65 EN**: Defines macro `HEXAGON_V_PUT_B7` for conditional compilation, shorthand, or API generation.
  **L65 CN**: 定义宏 `HEXAGON_V_PUT_B7`，用于条件编译、简写或 API 生成。
- **L66 EN**: Defines macro `HEXAGON_V_CREATE_D` for conditional compilation, shorthand, or API generation.
  **L66 CN**: 定义宏 `HEXAGON_V_CREATE_D`，用于条件编译、简写或 API 生成。
- **L67 EN**: Defines macro `HEXAGON_V_CREATE_W` for conditional compilation, shorthand, or API generation.
  **L67 CN**: 定义宏 `HEXAGON_V_CREATE_W`，用于条件编译、简写或 API 生成。
- **L68 EN**: Defines macro `HEXAGON_V_CREATE_H` for conditional compilation, shorthand, or API generation.
  **L68 CN**: 定义宏 `HEXAGON_V_CREATE_H`，用于条件编译、简写或 API 生成。
- **L69 EN**: Defines macro `HEXAGON_V_CREATE_B` for conditional compilation, shorthand, or API generation.
  **L69 CN**: 定义宏 `HEXAGON_V_CREATE_B`，用于条件编译、简写或 API 生成。
- **L70 EN**: Blank line separating nearby declarations or logic blocks.
  **L70 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L71 EN**: Starts a preprocessor conditional block: `#ifdef __cplusplus`.
  **L71 CN**: 开始一个预处理条件块：`#ifdef __cplusplus`。
- **L72 EN**: Defines macro `HEXAGON_VectC` for conditional compilation, shorthand, or API generation.
  **L72 CN**: 定义宏 `HEXAGON_VectC`，用于条件编译、简写或 API 生成。

### Lines 73-96

````c
#endif /* __cplusplus */

/* 64 Bit Vectors */

typedef long long __attribute__((__may_alias__)) HEXAGON_Vect64;

/* Extract doubleword macros */

#define HEXAGON_V64_GET_D(v) (v)
#define HEXAGON_V64_GET_UD(v) ((unsigned long long)(v))

/* Extract word macros */

#define HEXAGON_V64_GET_W0(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      int w[2];                                                                \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.w[0];                                                \
  })
#define HEXAGON_V64_GET_W1(v)                                                        \
  __extension__({                                                              \
````
- **L73 EN**: Closes the current preprocessor conditional block.
  **L73 CN**: 结束当前预处理条件块。
- **L74 EN**: Blank line separating nearby declarations or logic blocks.
  **L74 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L75 EN**: Comment explains nearby logic, constraints, or intent: `64 Bit Vectors`.
  **L75 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 Bit Vectors`。
- **L76 EN**: Blank line separating nearby declarations or logic blocks.
  **L76 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L77 EN**: Introduces an alias or helper declaration: `typedef long long __attribute__((__may_alias__)) HEXAGON_Vect64;`.
  **L77 CN**: 引入一条别名或辅助声明：`typedef long long __attribute__((__may_alias__)) HEXAGON_Vect64;`。
- **L78 EN**: Blank line separating nearby declarations or logic blocks.
  **L78 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L79 EN**: Comment explains nearby logic, constraints, or intent: `Extract doubleword macros`.
  **L79 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract doubleword macros`。
- **L80 EN**: Blank line separating nearby declarations or logic blocks.
  **L80 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L81 EN**: Defines macro `HEXAGON_V64_GET_D(v)` for conditional compilation, shorthand, or API generation.
  **L81 CN**: 定义宏 `HEXAGON_V64_GET_D(v)`，用于条件编译、简写或 API 生成。
- **L82 EN**: Defines macro `HEXAGON_V64_GET_UD(v)` for conditional compilation, shorthand, or API generation.
  **L82 CN**: 定义宏 `HEXAGON_V64_GET_UD(v)`，用于条件编译、简写或 API 生成。
- **L83 EN**: Blank line separating nearby declarations or logic blocks.
  **L83 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L84 EN**: Comment explains nearby logic, constraints, or intent: `Extract word macros`.
  **L84 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract word macros`。
- **L85 EN**: Blank line separating nearby declarations or logic blocks.
  **L85 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L86 EN**: Defines macro `HEXAGON_V64_GET_W0(v)` for conditional compilation, shorthand, or API generation.
  **L86 CN**: 定义宏 `HEXAGON_V64_GET_W0(v)`，用于条件编译、简写或 API 生成。
- **L87 EN**: Continues logic associated with callable symbol `__extension__`.
  **L87 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L88 EN**: Declares union `union`.
  **L88 CN**: 声明 union `union`。
- **L89 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L89 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L90 EN**: Continues the surrounding expression or declaration: `int w[2];                                                                \`.
  **L90 CN**: 继续构造周围的表达式或声明：`int w[2];                                                                \`。
- **L91 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L91 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L92 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L92 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L93 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.w[0];                                                \`.
  **L93 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.w[0];                                                \`。
- **L94 EN**: Continues the surrounding expression or declaration: `})`.
  **L94 CN**: 继续构造周围的表达式或声明：`})`。
- **L95 EN**: Defines macro `HEXAGON_V64_GET_W1(v)` for conditional compilation, shorthand, or API generation.
  **L95 CN**: 定义宏 `HEXAGON_V64_GET_W1(v)`，用于条件编译、简写或 API 生成。
- **L96 EN**: Continues logic associated with callable symbol `__extension__`.
  **L96 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。

### Lines 97-120

````c
    union {                                                                    \
      long long d;                                                             \
      int w[2];                                                                \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.w[1];                                                \
  })
#define HEXAGON_V64_GET_UW0(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned int uw[2];                                                      \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.uw[0];                                               \
  })
#define HEXAGON_V64_GET_UW1(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned int uw[2];                                                      \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.uw[1];                                               \
````
- **L97 EN**: Declares union `union`.
  **L97 CN**: 声明 union `union`。
- **L98 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L98 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L99 EN**: Continues the surrounding expression or declaration: `int w[2];                                                                \`.
  **L99 CN**: 继续构造周围的表达式或声明：`int w[2];                                                                \`。
- **L100 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L100 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L101 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L101 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L102 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.w[1];                                                \`.
  **L102 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.w[1];                                                \`。
- **L103 EN**: Continues the surrounding expression or declaration: `})`.
  **L103 CN**: 继续构造周围的表达式或声明：`})`。
- **L104 EN**: Defines macro `HEXAGON_V64_GET_UW0(v)` for conditional compilation, shorthand, or API generation.
  **L104 CN**: 定义宏 `HEXAGON_V64_GET_UW0(v)`，用于条件编译、简写或 API 生成。
- **L105 EN**: Continues logic associated with callable symbol `__extension__`.
  **L105 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L106 EN**: Declares union `union`.
  **L106 CN**: 声明 union `union`。
- **L107 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L107 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L108 EN**: Continues the surrounding expression or declaration: `unsigned int uw[2];                                                      \`.
  **L108 CN**: 继续构造周围的表达式或声明：`unsigned int uw[2];                                                      \`。
- **L109 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L109 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L110 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L110 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L111 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.uw[0];                                               \`.
  **L111 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.uw[0];                                               \`。
- **L112 EN**: Continues the surrounding expression or declaration: `})`.
  **L112 CN**: 继续构造周围的表达式或声明：`})`。
- **L113 EN**: Defines macro `HEXAGON_V64_GET_UW1(v)` for conditional compilation, shorthand, or API generation.
  **L113 CN**: 定义宏 `HEXAGON_V64_GET_UW1(v)`，用于条件编译、简写或 API 生成。
- **L114 EN**: Continues logic associated with callable symbol `__extension__`.
  **L114 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L115 EN**: Declares union `union`.
  **L115 CN**: 声明 union `union`。
- **L116 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L116 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L117 EN**: Continues the surrounding expression or declaration: `unsigned int uw[2];                                                      \`.
  **L117 CN**: 继续构造周围的表达式或声明：`unsigned int uw[2];                                                      \`。
- **L118 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L118 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L119 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L119 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L120 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.uw[1];                                               \`.
  **L120 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.uw[1];                                               \`。

### Lines 121-144

````c
  })

/* Extract half word macros */

#define HEXAGON_V64_GET_H0(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.h[0];                                                \
  })
#define HEXAGON_V64_GET_H1(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.h[1];                                                \
  })
#define HEXAGON_V64_GET_H2(v)                                                        \
  __extension__({                                                              \
````
- **L121 EN**: Continues the surrounding expression or declaration: `})`.
  **L121 CN**: 继续构造周围的表达式或声明：`})`。
- **L122 EN**: Blank line separating nearby declarations or logic blocks.
  **L122 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L123 EN**: Comment explains nearby logic, constraints, or intent: `Extract half word macros`.
  **L123 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract half word macros`。
- **L124 EN**: Blank line separating nearby declarations or logic blocks.
  **L124 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L125 EN**: Defines macro `HEXAGON_V64_GET_H0(v)` for conditional compilation, shorthand, or API generation.
  **L125 CN**: 定义宏 `HEXAGON_V64_GET_H0(v)`，用于条件编译、简写或 API 生成。
- **L126 EN**: Continues logic associated with callable symbol `__extension__`.
  **L126 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L127 EN**: Declares union `union`.
  **L127 CN**: 声明 union `union`。
- **L128 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L128 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L129 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L129 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L130 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L130 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L131 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L131 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L132 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.h[0];                                                \`.
  **L132 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.h[0];                                                \`。
- **L133 EN**: Continues the surrounding expression or declaration: `})`.
  **L133 CN**: 继续构造周围的表达式或声明：`})`。
- **L134 EN**: Defines macro `HEXAGON_V64_GET_H1(v)` for conditional compilation, shorthand, or API generation.
  **L134 CN**: 定义宏 `HEXAGON_V64_GET_H1(v)`，用于条件编译、简写或 API 生成。
- **L135 EN**: Continues logic associated with callable symbol `__extension__`.
  **L135 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L136 EN**: Declares union `union`.
  **L136 CN**: 声明 union `union`。
- **L137 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L137 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L138 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L138 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L139 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L139 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L140 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L140 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L141 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.h[1];                                                \`.
  **L141 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.h[1];                                                \`。
- **L142 EN**: Continues the surrounding expression or declaration: `})`.
  **L142 CN**: 继续构造周围的表达式或声明：`})`。
- **L143 EN**: Defines macro `HEXAGON_V64_GET_H2(v)` for conditional compilation, shorthand, or API generation.
  **L143 CN**: 定义宏 `HEXAGON_V64_GET_H2(v)`，用于条件编译、简写或 API 生成。
- **L144 EN**: Continues logic associated with callable symbol `__extension__`.
  **L144 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。

### Lines 145-168

````c
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.h[2];                                                \
  })
#define HEXAGON_V64_GET_H3(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.h[3];                                                \
  })
#define HEXAGON_V64_GET_UH0(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned short uh[4];                                                    \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.uh[0];                                               \
````
- **L145 EN**: Declares union `union`.
  **L145 CN**: 声明 union `union`。
- **L146 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L146 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L147 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L147 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L148 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L148 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L149 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L149 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L150 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.h[2];                                                \`.
  **L150 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.h[2];                                                \`。
- **L151 EN**: Continues the surrounding expression or declaration: `})`.
  **L151 CN**: 继续构造周围的表达式或声明：`})`。
- **L152 EN**: Defines macro `HEXAGON_V64_GET_H3(v)` for conditional compilation, shorthand, or API generation.
  **L152 CN**: 定义宏 `HEXAGON_V64_GET_H3(v)`，用于条件编译、简写或 API 生成。
- **L153 EN**: Continues logic associated with callable symbol `__extension__`.
  **L153 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L154 EN**: Declares union `union`.
  **L154 CN**: 声明 union `union`。
- **L155 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L155 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L156 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L156 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L157 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L157 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L158 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L158 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L159 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.h[3];                                                \`.
  **L159 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.h[3];                                                \`。
- **L160 EN**: Continues the surrounding expression or declaration: `})`.
  **L160 CN**: 继续构造周围的表达式或声明：`})`。
- **L161 EN**: Defines macro `HEXAGON_V64_GET_UH0(v)` for conditional compilation, shorthand, or API generation.
  **L161 CN**: 定义宏 `HEXAGON_V64_GET_UH0(v)`，用于条件编译、简写或 API 生成。
- **L162 EN**: Continues logic associated with callable symbol `__extension__`.
  **L162 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L163 EN**: Declares union `union`.
  **L163 CN**: 声明 union `union`。
- **L164 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L164 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L165 EN**: Continues the surrounding expression or declaration: `unsigned short uh[4];                                                    \`.
  **L165 CN**: 继续构造周围的表达式或声明：`unsigned short uh[4];                                                    \`。
- **L166 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L166 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L167 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L167 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L168 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.uh[0];                                               \`.
  **L168 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.uh[0];                                               \`。

### Lines 169-192

````c
  })
#define HEXAGON_V64_GET_UH1(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned short uh[4];                                                    \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.uh[1];                                               \
  })
#define HEXAGON_V64_GET_UH2(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned short uh[4];                                                    \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.uh[2];                                               \
  })
#define HEXAGON_V64_GET_UH3(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned short uh[4];                                                    \
````
- **L169 EN**: Continues the surrounding expression or declaration: `})`.
  **L169 CN**: 继续构造周围的表达式或声明：`})`。
- **L170 EN**: Defines macro `HEXAGON_V64_GET_UH1(v)` for conditional compilation, shorthand, or API generation.
  **L170 CN**: 定义宏 `HEXAGON_V64_GET_UH1(v)`，用于条件编译、简写或 API 生成。
- **L171 EN**: Continues logic associated with callable symbol `__extension__`.
  **L171 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L172 EN**: Declares union `union`.
  **L172 CN**: 声明 union `union`。
- **L173 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L173 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L174 EN**: Continues the surrounding expression or declaration: `unsigned short uh[4];                                                    \`.
  **L174 CN**: 继续构造周围的表达式或声明：`unsigned short uh[4];                                                    \`。
- **L175 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L175 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L176 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L176 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L177 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.uh[1];                                               \`.
  **L177 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.uh[1];                                               \`。
- **L178 EN**: Continues the surrounding expression or declaration: `})`.
  **L178 CN**: 继续构造周围的表达式或声明：`})`。
- **L179 EN**: Defines macro `HEXAGON_V64_GET_UH2(v)` for conditional compilation, shorthand, or API generation.
  **L179 CN**: 定义宏 `HEXAGON_V64_GET_UH2(v)`，用于条件编译、简写或 API 生成。
- **L180 EN**: Continues logic associated with callable symbol `__extension__`.
  **L180 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L181 EN**: Declares union `union`.
  **L181 CN**: 声明 union `union`。
- **L182 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L182 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L183 EN**: Continues the surrounding expression or declaration: `unsigned short uh[4];                                                    \`.
  **L183 CN**: 继续构造周围的表达式或声明：`unsigned short uh[4];                                                    \`。
- **L184 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L184 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L185 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L185 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L186 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.uh[2];                                               \`.
  **L186 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.uh[2];                                               \`。
- **L187 EN**: Continues the surrounding expression or declaration: `})`.
  **L187 CN**: 继续构造周围的表达式或声明：`})`。
- **L188 EN**: Defines macro `HEXAGON_V64_GET_UH3(v)` for conditional compilation, shorthand, or API generation.
  **L188 CN**: 定义宏 `HEXAGON_V64_GET_UH3(v)`，用于条件编译、简写或 API 生成。
- **L189 EN**: Continues logic associated with callable symbol `__extension__`.
  **L189 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L190 EN**: Declares union `union`.
  **L190 CN**: 声明 union `union`。
- **L191 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L191 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L192 EN**: Continues the surrounding expression or declaration: `unsigned short uh[4];                                                    \`.
  **L192 CN**: 继续构造周围的表达式或声明：`unsigned short uh[4];                                                    \`。

### Lines 193-216

````c
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.uh[3];                                               \
  })

/* Extract byte macros */

#define HEXAGON_V64_GET_B0(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      signed char b[8];                                                        \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.b[0];                                                \
  })
#define HEXAGON_V64_GET_B1(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      signed char b[8];                                                        \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.b[1];                                                \
````
- **L193 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L193 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L194 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L194 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L195 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.uh[3];                                               \`.
  **L195 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.uh[3];                                               \`。
- **L196 EN**: Continues the surrounding expression or declaration: `})`.
  **L196 CN**: 继续构造周围的表达式或声明：`})`。
- **L197 EN**: Blank line separating nearby declarations or logic blocks.
  **L197 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L198 EN**: Comment explains nearby logic, constraints, or intent: `Extract byte macros`.
  **L198 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract byte macros`。
- **L199 EN**: Blank line separating nearby declarations or logic blocks.
  **L199 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L200 EN**: Defines macro `HEXAGON_V64_GET_B0(v)` for conditional compilation, shorthand, or API generation.
  **L200 CN**: 定义宏 `HEXAGON_V64_GET_B0(v)`，用于条件编译、简写或 API 生成。
- **L201 EN**: Continues logic associated with callable symbol `__extension__`.
  **L201 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L202 EN**: Declares union `union`.
  **L202 CN**: 声明 union `union`。
- **L203 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L203 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L204 EN**: Continues the surrounding expression or declaration: `signed char b[8];                                                        \`.
  **L204 CN**: 继续构造周围的表达式或声明：`signed char b[8];                                                        \`。
- **L205 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L205 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L206 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L206 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L207 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[0];                                                \`.
  **L207 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[0];                                                \`。
- **L208 EN**: Continues the surrounding expression or declaration: `})`.
  **L208 CN**: 继续构造周围的表达式或声明：`})`。
- **L209 EN**: Defines macro `HEXAGON_V64_GET_B1(v)` for conditional compilation, shorthand, or API generation.
  **L209 CN**: 定义宏 `HEXAGON_V64_GET_B1(v)`，用于条件编译、简写或 API 生成。
- **L210 EN**: Continues logic associated with callable symbol `__extension__`.
  **L210 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L211 EN**: Declares union `union`.
  **L211 CN**: 声明 union `union`。
- **L212 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L212 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L213 EN**: Continues the surrounding expression or declaration: `signed char b[8];                                                        \`.
  **L213 CN**: 继续构造周围的表达式或声明：`signed char b[8];                                                        \`。
- **L214 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L214 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L215 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L215 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L216 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[1];                                                \`.
  **L216 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[1];                                                \`。

### Lines 217-240

````c
  })
#define HEXAGON_V64_GET_B2(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      signed char b[8];                                                        \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.b[2];                                                \
  })
#define HEXAGON_V64_GET_B3(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      signed char b[8];                                                        \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.b[3];                                                \
  })
#define HEXAGON_V64_GET_B4(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      signed char b[8];                                                        \
````
- **L217 EN**: Continues the surrounding expression or declaration: `})`.
  **L217 CN**: 继续构造周围的表达式或声明：`})`。
- **L218 EN**: Defines macro `HEXAGON_V64_GET_B2(v)` for conditional compilation, shorthand, or API generation.
  **L218 CN**: 定义宏 `HEXAGON_V64_GET_B2(v)`，用于条件编译、简写或 API 生成。
- **L219 EN**: Continues logic associated with callable symbol `__extension__`.
  **L219 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L220 EN**: Declares union `union`.
  **L220 CN**: 声明 union `union`。
- **L221 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L221 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L222 EN**: Continues the surrounding expression or declaration: `signed char b[8];                                                        \`.
  **L222 CN**: 继续构造周围的表达式或声明：`signed char b[8];                                                        \`。
- **L223 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L223 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L224 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L224 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L225 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[2];                                                \`.
  **L225 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[2];                                                \`。
- **L226 EN**: Continues the surrounding expression or declaration: `})`.
  **L226 CN**: 继续构造周围的表达式或声明：`})`。
- **L227 EN**: Defines macro `HEXAGON_V64_GET_B3(v)` for conditional compilation, shorthand, or API generation.
  **L227 CN**: 定义宏 `HEXAGON_V64_GET_B3(v)`，用于条件编译、简写或 API 生成。
- **L228 EN**: Continues logic associated with callable symbol `__extension__`.
  **L228 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L229 EN**: Declares union `union`.
  **L229 CN**: 声明 union `union`。
- **L230 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L230 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L231 EN**: Continues the surrounding expression or declaration: `signed char b[8];                                                        \`.
  **L231 CN**: 继续构造周围的表达式或声明：`signed char b[8];                                                        \`。
- **L232 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L232 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L233 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L233 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L234 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[3];                                                \`.
  **L234 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[3];                                                \`。
- **L235 EN**: Continues the surrounding expression or declaration: `})`.
  **L235 CN**: 继续构造周围的表达式或声明：`})`。
- **L236 EN**: Defines macro `HEXAGON_V64_GET_B4(v)` for conditional compilation, shorthand, or API generation.
  **L236 CN**: 定义宏 `HEXAGON_V64_GET_B4(v)`，用于条件编译、简写或 API 生成。
- **L237 EN**: Continues logic associated with callable symbol `__extension__`.
  **L237 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L238 EN**: Declares union `union`.
  **L238 CN**: 声明 union `union`。
- **L239 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L239 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L240 EN**: Continues the surrounding expression or declaration: `signed char b[8];                                                        \`.
  **L240 CN**: 继续构造周围的表达式或声明：`signed char b[8];                                                        \`。

### Lines 241-264

````c
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.b[4];                                                \
  })
#define HEXAGON_V64_GET_B5(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      signed char b[8];                                                        \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.b[5];                                                \
  })
#define HEXAGON_V64_GET_B6(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      signed char b[8];                                                        \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.b[6];                                                \
  })
#define HEXAGON_V64_GET_B7(v)                                                        \
  __extension__({                                                              \
````
- **L241 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L241 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L242 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L242 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L243 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[4];                                                \`.
  **L243 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[4];                                                \`。
- **L244 EN**: Continues the surrounding expression or declaration: `})`.
  **L244 CN**: 继续构造周围的表达式或声明：`})`。
- **L245 EN**: Defines macro `HEXAGON_V64_GET_B5(v)` for conditional compilation, shorthand, or API generation.
  **L245 CN**: 定义宏 `HEXAGON_V64_GET_B5(v)`，用于条件编译、简写或 API 生成。
- **L246 EN**: Continues logic associated with callable symbol `__extension__`.
  **L246 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L247 EN**: Declares union `union`.
  **L247 CN**: 声明 union `union`。
- **L248 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L248 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L249 EN**: Continues the surrounding expression or declaration: `signed char b[8];                                                        \`.
  **L249 CN**: 继续构造周围的表达式或声明：`signed char b[8];                                                        \`。
- **L250 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L250 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L251 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L251 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L252 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[5];                                                \`.
  **L252 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[5];                                                \`。
- **L253 EN**: Continues the surrounding expression or declaration: `})`.
  **L253 CN**: 继续构造周围的表达式或声明：`})`。
- **L254 EN**: Defines macro `HEXAGON_V64_GET_B6(v)` for conditional compilation, shorthand, or API generation.
  **L254 CN**: 定义宏 `HEXAGON_V64_GET_B6(v)`，用于条件编译、简写或 API 生成。
- **L255 EN**: Continues logic associated with callable symbol `__extension__`.
  **L255 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L256 EN**: Declares union `union`.
  **L256 CN**: 声明 union `union`。
- **L257 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L257 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L258 EN**: Continues the surrounding expression or declaration: `signed char b[8];                                                        \`.
  **L258 CN**: 继续构造周围的表达式或声明：`signed char b[8];                                                        \`。
- **L259 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L259 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L260 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L260 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L261 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[6];                                                \`.
  **L261 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[6];                                                \`。
- **L262 EN**: Continues the surrounding expression or declaration: `})`.
  **L262 CN**: 继续构造周围的表达式或声明：`})`。
- **L263 EN**: Defines macro `HEXAGON_V64_GET_B7(v)` for conditional compilation, shorthand, or API generation.
  **L263 CN**: 定义宏 `HEXAGON_V64_GET_B7(v)`，用于条件编译、简写或 API 生成。
- **L264 EN**: Continues logic associated with callable symbol `__extension__`.
  **L264 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。

### Lines 265-288

````c
    union {                                                                    \
      long long d;                                                             \
      signed char b[8];                                                        \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.b[7];                                                \
  })
#define HEXAGON_V64_GET_UB0(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned char ub[8];                                                     \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.ub[0];                                               \
  })
#define HEXAGON_V64_GET_UB1(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned char ub[8];                                                     \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.ub[1];                                               \
````
- **L265 EN**: Declares union `union`.
  **L265 CN**: 声明 union `union`。
- **L266 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L266 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L267 EN**: Continues the surrounding expression or declaration: `signed char b[8];                                                        \`.
  **L267 CN**: 继续构造周围的表达式或声明：`signed char b[8];                                                        \`。
- **L268 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L268 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L269 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L269 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L270 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[7];                                                \`.
  **L270 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[7];                                                \`。
- **L271 EN**: Continues the surrounding expression or declaration: `})`.
  **L271 CN**: 继续构造周围的表达式或声明：`})`。
- **L272 EN**: Defines macro `HEXAGON_V64_GET_UB0(v)` for conditional compilation, shorthand, or API generation.
  **L272 CN**: 定义宏 `HEXAGON_V64_GET_UB0(v)`，用于条件编译、简写或 API 生成。
- **L273 EN**: Continues logic associated with callable symbol `__extension__`.
  **L273 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L274 EN**: Declares union `union`.
  **L274 CN**: 声明 union `union`。
- **L275 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L275 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L276 EN**: Continues the surrounding expression or declaration: `unsigned char ub[8];                                                     \`.
  **L276 CN**: 继续构造周围的表达式或声明：`unsigned char ub[8];                                                     \`。
- **L277 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L277 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L278 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L278 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L279 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.ub[0];                                               \`.
  **L279 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.ub[0];                                               \`。
- **L280 EN**: Continues the surrounding expression or declaration: `})`.
  **L280 CN**: 继续构造周围的表达式或声明：`})`。
- **L281 EN**: Defines macro `HEXAGON_V64_GET_UB1(v)` for conditional compilation, shorthand, or API generation.
  **L281 CN**: 定义宏 `HEXAGON_V64_GET_UB1(v)`，用于条件编译、简写或 API 生成。
- **L282 EN**: Continues logic associated with callable symbol `__extension__`.
  **L282 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L283 EN**: Declares union `union`.
  **L283 CN**: 声明 union `union`。
- **L284 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L284 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L285 EN**: Continues the surrounding expression or declaration: `unsigned char ub[8];                                                     \`.
  **L285 CN**: 继续构造周围的表达式或声明：`unsigned char ub[8];                                                     \`。
- **L286 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L286 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L287 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L287 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L288 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.ub[1];                                               \`.
  **L288 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.ub[1];                                               \`。

### Lines 289-312

````c
  })
#define HEXAGON_V64_GET_UB2(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned char ub[8];                                                     \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.ub[2];                                               \
  })
#define HEXAGON_V64_GET_UB3(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned char ub[8];                                                     \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.ub[3];                                               \
  })
#define HEXAGON_V64_GET_UB4(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned char ub[8];                                                     \
````
- **L289 EN**: Continues the surrounding expression or declaration: `})`.
  **L289 CN**: 继续构造周围的表达式或声明：`})`。
- **L290 EN**: Defines macro `HEXAGON_V64_GET_UB2(v)` for conditional compilation, shorthand, or API generation.
  **L290 CN**: 定义宏 `HEXAGON_V64_GET_UB2(v)`，用于条件编译、简写或 API 生成。
- **L291 EN**: Continues logic associated with callable symbol `__extension__`.
  **L291 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L292 EN**: Declares union `union`.
  **L292 CN**: 声明 union `union`。
- **L293 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L293 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L294 EN**: Continues the surrounding expression or declaration: `unsigned char ub[8];                                                     \`.
  **L294 CN**: 继续构造周围的表达式或声明：`unsigned char ub[8];                                                     \`。
- **L295 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L295 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L296 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L296 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L297 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.ub[2];                                               \`.
  **L297 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.ub[2];                                               \`。
- **L298 EN**: Continues the surrounding expression or declaration: `})`.
  **L298 CN**: 继续构造周围的表达式或声明：`})`。
- **L299 EN**: Defines macro `HEXAGON_V64_GET_UB3(v)` for conditional compilation, shorthand, or API generation.
  **L299 CN**: 定义宏 `HEXAGON_V64_GET_UB3(v)`，用于条件编译、简写或 API 生成。
- **L300 EN**: Continues logic associated with callable symbol `__extension__`.
  **L300 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L301 EN**: Declares union `union`.
  **L301 CN**: 声明 union `union`。
- **L302 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L302 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L303 EN**: Continues the surrounding expression or declaration: `unsigned char ub[8];                                                     \`.
  **L303 CN**: 继续构造周围的表达式或声明：`unsigned char ub[8];                                                     \`。
- **L304 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L304 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L305 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L305 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L306 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.ub[3];                                               \`.
  **L306 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.ub[3];                                               \`。
- **L307 EN**: Continues the surrounding expression or declaration: `})`.
  **L307 CN**: 继续构造周围的表达式或声明：`})`。
- **L308 EN**: Defines macro `HEXAGON_V64_GET_UB4(v)` for conditional compilation, shorthand, or API generation.
  **L308 CN**: 定义宏 `HEXAGON_V64_GET_UB4(v)`，用于条件编译、简写或 API 生成。
- **L309 EN**: Continues logic associated with callable symbol `__extension__`.
  **L309 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L310 EN**: Declares union `union`.
  **L310 CN**: 声明 union `union`。
- **L311 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L311 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L312 EN**: Continues the surrounding expression or declaration: `unsigned char ub[8];                                                     \`.
  **L312 CN**: 继续构造周围的表达式或声明：`unsigned char ub[8];                                                     \`。

### Lines 313-336

````c
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.ub[4];                                               \
  })
#define HEXAGON_V64_GET_UB5(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned char ub[8];                                                     \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.ub[5];                                               \
  })
#define HEXAGON_V64_GET_UB6(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned char ub[8];                                                     \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.ub[6];                                               \
  })
#define HEXAGON_V64_GET_UB7(v)                                                       \
  __extension__({                                                              \
````
- **L313 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L313 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L314 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L314 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L315 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.ub[4];                                               \`.
  **L315 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.ub[4];                                               \`。
- **L316 EN**: Continues the surrounding expression or declaration: `})`.
  **L316 CN**: 继续构造周围的表达式或声明：`})`。
- **L317 EN**: Defines macro `HEXAGON_V64_GET_UB5(v)` for conditional compilation, shorthand, or API generation.
  **L317 CN**: 定义宏 `HEXAGON_V64_GET_UB5(v)`，用于条件编译、简写或 API 生成。
- **L318 EN**: Continues logic associated with callable symbol `__extension__`.
  **L318 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L319 EN**: Declares union `union`.
  **L319 CN**: 声明 union `union`。
- **L320 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L320 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L321 EN**: Continues the surrounding expression or declaration: `unsigned char ub[8];                                                     \`.
  **L321 CN**: 继续构造周围的表达式或声明：`unsigned char ub[8];                                                     \`。
- **L322 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L322 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L323 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L323 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L324 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.ub[5];                                               \`.
  **L324 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.ub[5];                                               \`。
- **L325 EN**: Continues the surrounding expression or declaration: `})`.
  **L325 CN**: 继续构造周围的表达式或声明：`})`。
- **L326 EN**: Defines macro `HEXAGON_V64_GET_UB6(v)` for conditional compilation, shorthand, or API generation.
  **L326 CN**: 定义宏 `HEXAGON_V64_GET_UB6(v)`，用于条件编译、简写或 API 生成。
- **L327 EN**: Continues logic associated with callable symbol `__extension__`.
  **L327 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L328 EN**: Declares union `union`.
  **L328 CN**: 声明 union `union`。
- **L329 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L329 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L330 EN**: Continues the surrounding expression or declaration: `unsigned char ub[8];                                                     \`.
  **L330 CN**: 继续构造周围的表达式或声明：`unsigned char ub[8];                                                     \`。
- **L331 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L331 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L332 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L332 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L333 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.ub[6];                                               \`.
  **L333 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.ub[6];                                               \`。
- **L334 EN**: Continues the surrounding expression or declaration: `})`.
  **L334 CN**: 继续构造周围的表达式或声明：`})`。
- **L335 EN**: Defines macro `HEXAGON_V64_GET_UB7(v)` for conditional compilation, shorthand, or API generation.
  **L335 CN**: 定义宏 `HEXAGON_V64_GET_UB7(v)`，用于条件编译、简写或 API 生成。
- **L336 EN**: Continues logic associated with callable symbol `__extension__`.
  **L336 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。

### Lines 337-360

````c
    union {                                                                    \
      long long d;                                                             \
      unsigned char ub[8];                                                     \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.ub[7];                                               \
  })

/* NOTE: All set macros return a HEXAGON_Vect64 type */

/* Set doubleword macro */

#define HEXAGON_V64_PUT_D(v, new) (new)

/* Set word macros */

#ifdef __hexagon__

#define HEXAGON_V64_PUT_W0(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      int w[2];                                                                \
    } _HEXAGON_V64_internal_union;                                                   \
````
- **L337 EN**: Declares union `union`.
  **L337 CN**: 声明 union `union`。
- **L338 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L338 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L339 EN**: Continues the surrounding expression or declaration: `unsigned char ub[8];                                                     \`.
  **L339 CN**: 继续构造周围的表达式或声明：`unsigned char ub[8];                                                     \`。
- **L340 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L340 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L341 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L341 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L342 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.ub[7];                                               \`.
  **L342 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.ub[7];                                               \`。
- **L343 EN**: Continues the surrounding expression or declaration: `})`.
  **L343 CN**: 继续构造周围的表达式或声明：`})`。
- **L344 EN**: Blank line separating nearby declarations or logic blocks.
  **L344 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L345 EN**: Comment highlights an implementation note: `NOTE: All set macros return a HEXAGON_Vect64 type`.
  **L345 CN**: 注释强调一条实现说明：`NOTE: All set macros return a HEXAGON_Vect64 type`。
- **L346 EN**: Blank line separating nearby declarations or logic blocks.
  **L346 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L347 EN**: Comment explains nearby logic, constraints, or intent: `Set doubleword macro`.
  **L347 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set doubleword macro`。
- **L348 EN**: Blank line separating nearby declarations or logic blocks.
  **L348 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L349 EN**: Defines macro `HEXAGON_V64_PUT_D(v, new)` for conditional compilation, shorthand, or API generation.
  **L349 CN**: 定义宏 `HEXAGON_V64_PUT_D(v, new)`，用于条件编译、简写或 API 生成。
- **L350 EN**: Blank line separating nearby declarations or logic blocks.
  **L350 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L351 EN**: Comment explains nearby logic, constraints, or intent: `Set word macros`.
  **L351 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set word macros`。
- **L352 EN**: Blank line separating nearby declarations or logic blocks.
  **L352 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L353 EN**: Starts a preprocessor conditional block: `#ifdef __hexagon__`.
  **L353 CN**: 开始一个预处理条件块：`#ifdef __hexagon__`。
- **L354 EN**: Blank line separating nearby declarations or logic blocks.
  **L354 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L355 EN**: Defines macro `HEXAGON_V64_PUT_W0(v, new)` for conditional compilation, shorthand, or API generation.
  **L355 CN**: 定义宏 `HEXAGON_V64_PUT_W0(v, new)`，用于条件编译、简写或 API 生成。
- **L356 EN**: Continues logic associated with callable symbol `__extension__`.
  **L356 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L357 EN**: Declares union `union`.
  **L357 CN**: 声明 union `union`。
- **L358 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L358 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L359 EN**: Continues the surrounding expression or declaration: `int w[2];                                                                \`.
  **L359 CN**: 继续构造周围的表达式或声明：`int w[2];                                                                \`。
- **L360 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L360 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。

### Lines 361-384

````c
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.w[0] = (new);                                        \
    _HEXAGON_V64_internal_union.d;                                                   \
  })
#define HEXAGON_V64_PUT_W1(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      int w[2];                                                                \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.w[1] = (new);                                        \
    _HEXAGON_V64_internal_union.d;                                                   \
  })

#else /* !__hexagon__ */

#define HEXAGON_V64_PUT_W0(v, new)                                                   \
  (((v) & 0xffffffff00000000LL) | ((HEXAGON_Vect64)((unsigned int)(new))))
#define HEXAGON_V64_PUT_W1(v, new)                                                   \
  (((v) & 0x00000000ffffffffLL) | (((HEXAGON_Vect64)(new)) << 32LL))

#endif /* !__hexagon__ */

````
- **L361 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L361 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L362 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.w[0] = (new);                                        \`.
  **L362 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.w[0] = (new);                                        \`。
- **L363 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L363 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L364 EN**: Continues the surrounding expression or declaration: `})`.
  **L364 CN**: 继续构造周围的表达式或声明：`})`。
- **L365 EN**: Defines macro `HEXAGON_V64_PUT_W1(v, new)` for conditional compilation, shorthand, or API generation.
  **L365 CN**: 定义宏 `HEXAGON_V64_PUT_W1(v, new)`，用于条件编译、简写或 API 生成。
- **L366 EN**: Continues logic associated with callable symbol `__extension__`.
  **L366 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L367 EN**: Declares union `union`.
  **L367 CN**: 声明 union `union`。
- **L368 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L368 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L369 EN**: Continues the surrounding expression or declaration: `int w[2];                                                                \`.
  **L369 CN**: 继续构造周围的表达式或声明：`int w[2];                                                                \`。
- **L370 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L370 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L371 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L371 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L372 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.w[1] = (new);                                        \`.
  **L372 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.w[1] = (new);                                        \`。
- **L373 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L373 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L374 EN**: Continues the surrounding expression or declaration: `})`.
  **L374 CN**: 继续构造周围的表达式或声明：`})`。
- **L375 EN**: Blank line separating nearby declarations or logic blocks.
  **L375 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L376 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L376 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L377 EN**: Blank line separating nearby declarations or logic blocks.
  **L377 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L378 EN**: Defines macro `HEXAGON_V64_PUT_W0(v, new)` for conditional compilation, shorthand, or API generation.
  **L378 CN**: 定义宏 `HEXAGON_V64_PUT_W0(v, new)`，用于条件编译、简写或 API 生成。
- **L379 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffffff00000000LL) | ((HEXAGON_Vect64)((unsigned int)(new))))`.
  **L379 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffffff00000000LL) | ((HEXAGON_Vect64)((unsigned int)(new))))`。
- **L380 EN**: Defines macro `HEXAGON_V64_PUT_W1(v, new)` for conditional compilation, shorthand, or API generation.
  **L380 CN**: 定义宏 `HEXAGON_V64_PUT_W1(v, new)`，用于条件编译、简写或 API 生成。
- **L381 EN**: Continues the surrounding expression or declaration: `(((v) & 0x00000000ffffffffLL) | (((HEXAGON_Vect64)(new)) << 32LL))`.
  **L381 CN**: 继续构造周围的表达式或声明：`(((v) & 0x00000000ffffffffLL) | (((HEXAGON_Vect64)(new)) << 32LL))`。
- **L382 EN**: Blank line separating nearby declarations or logic blocks.
  **L382 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L383 EN**: Closes the current preprocessor conditional block.
  **L383 CN**: 结束当前预处理条件块。
- **L384 EN**: Blank line separating nearby declarations or logic blocks.
  **L384 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 385-408

````c
/* Set half word macros */

#ifdef __hexagon__

#define HEXAGON_V64_PUT_H0(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.h[0] = (new);                                        \
    _HEXAGON_V64_internal_union.d;                                                   \
  })
#define HEXAGON_V64_PUT_H1(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.h[1] = (new);                                        \
    _HEXAGON_V64_internal_union.d;                                                   \
  })
````
- **L385 EN**: Comment explains nearby logic, constraints, or intent: `Set half word macros`.
  **L385 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set half word macros`。
- **L386 EN**: Blank line separating nearby declarations or logic blocks.
  **L386 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L387 EN**: Starts a preprocessor conditional block: `#ifdef __hexagon__`.
  **L387 CN**: 开始一个预处理条件块：`#ifdef __hexagon__`。
- **L388 EN**: Blank line separating nearby declarations or logic blocks.
  **L388 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L389 EN**: Defines macro `HEXAGON_V64_PUT_H0(v, new)` for conditional compilation, shorthand, or API generation.
  **L389 CN**: 定义宏 `HEXAGON_V64_PUT_H0(v, new)`，用于条件编译、简写或 API 生成。
- **L390 EN**: Continues logic associated with callable symbol `__extension__`.
  **L390 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L391 EN**: Declares union `union`.
  **L391 CN**: 声明 union `union`。
- **L392 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L392 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L393 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L393 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L394 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L394 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L395 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L395 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L396 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.h[0] = (new);                                        \`.
  **L396 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.h[0] = (new);                                        \`。
- **L397 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L397 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L398 EN**: Continues the surrounding expression or declaration: `})`.
  **L398 CN**: 继续构造周围的表达式或声明：`})`。
- **L399 EN**: Defines macro `HEXAGON_V64_PUT_H1(v, new)` for conditional compilation, shorthand, or API generation.
  **L399 CN**: 定义宏 `HEXAGON_V64_PUT_H1(v, new)`，用于条件编译、简写或 API 生成。
- **L400 EN**: Continues logic associated with callable symbol `__extension__`.
  **L400 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L401 EN**: Declares union `union`.
  **L401 CN**: 声明 union `union`。
- **L402 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L402 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L403 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L403 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L404 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L404 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L405 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L405 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L406 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.h[1] = (new);                                        \`.
  **L406 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.h[1] = (new);                                        \`。
- **L407 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L407 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L408 EN**: Continues the surrounding expression or declaration: `})`.
  **L408 CN**: 继续构造周围的表达式或声明：`})`。

### Lines 409-432

````c
#define HEXAGON_V64_PUT_H2(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.h[2] = (new);                                        \
    _HEXAGON_V64_internal_union.d;                                                   \
  })
#define HEXAGON_V64_PUT_H3(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.h[3] = (new);                                        \
    _HEXAGON_V64_internal_union.d;                                                   \
  })

#else /* !__hexagon__ */

#define HEXAGON_V64_PUT_H0(v, new)                                                   \
````
- **L409 EN**: Defines macro `HEXAGON_V64_PUT_H2(v, new)` for conditional compilation, shorthand, or API generation.
  **L409 CN**: 定义宏 `HEXAGON_V64_PUT_H2(v, new)`，用于条件编译、简写或 API 生成。
- **L410 EN**: Continues logic associated with callable symbol `__extension__`.
  **L410 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L411 EN**: Declares union `union`.
  **L411 CN**: 声明 union `union`。
- **L412 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L412 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L413 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L413 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L414 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L414 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L415 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L415 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L416 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.h[2] = (new);                                        \`.
  **L416 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.h[2] = (new);                                        \`。
- **L417 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L417 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L418 EN**: Continues the surrounding expression or declaration: `})`.
  **L418 CN**: 继续构造周围的表达式或声明：`})`。
- **L419 EN**: Defines macro `HEXAGON_V64_PUT_H3(v, new)` for conditional compilation, shorthand, or API generation.
  **L419 CN**: 定义宏 `HEXAGON_V64_PUT_H3(v, new)`，用于条件编译、简写或 API 生成。
- **L420 EN**: Continues logic associated with callable symbol `__extension__`.
  **L420 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L421 EN**: Declares union `union`.
  **L421 CN**: 声明 union `union`。
- **L422 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L422 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L423 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L423 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L424 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L424 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L425 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L425 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L426 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.h[3] = (new);                                        \`.
  **L426 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.h[3] = (new);                                        \`。
- **L427 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L427 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L428 EN**: Continues the surrounding expression or declaration: `})`.
  **L428 CN**: 继续构造周围的表达式或声明：`})`。
- **L429 EN**: Blank line separating nearby declarations or logic blocks.
  **L429 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L430 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L430 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L431 EN**: Blank line separating nearby declarations or logic blocks.
  **L431 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L432 EN**: Defines macro `HEXAGON_V64_PUT_H0(v, new)` for conditional compilation, shorthand, or API generation.
  **L432 CN**: 定义宏 `HEXAGON_V64_PUT_H0(v, new)`，用于条件编译、简写或 API 生成。

### Lines 433-456

````c
  (((v) & 0xffffffffffff0000LL) | ((HEXAGON_Vect64)((unsigned short)(new))))
#define HEXAGON_V64_PUT_H1(v, new)                                                   \
  (((v) & 0xffffffff0000ffffLL) | (((HEXAGON_Vect64)((unsigned short)(new))) << 16LL))
#define HEXAGON_V64_PUT_H2(v, new)                                                   \
  (((v) & 0xffff0000ffffffffLL) | (((HEXAGON_Vect64)((unsigned short)(new))) << 32LL))
#define HEXAGON_V64_PUT_H3(v, new)                                                   \
  (((v) & 0x0000ffffffffffffLL) | (((HEXAGON_Vect64)(new)) << 48LL))

#endif /* !__hexagon__ */

/* Set byte macros */

#ifdef __hexagon__

#define HEXAGON_V64_PUT_B0(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.b[0] = (new);                                        \
    _HEXAGON_V64_internal_union.d;                                                   \
  })
````
- **L433 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffffffffff0000LL) | ((HEXAGON_Vect64)((unsigned short)(new))))`.
  **L433 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffffffffff0000LL) | ((HEXAGON_Vect64)((unsigned short)(new))))`。
- **L434 EN**: Defines macro `HEXAGON_V64_PUT_H1(v, new)` for conditional compilation, shorthand, or API generation.
  **L434 CN**: 定义宏 `HEXAGON_V64_PUT_H1(v, new)`，用于条件编译、简写或 API 生成。
- **L435 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffffff0000ffffLL) | (((HEXAGON_Vect64)((unsigned short)(new))) << 16LL))`.
  **L435 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffffff0000ffffLL) | (((HEXAGON_Vect64)((unsigned short)(new))) << 16LL))`。
- **L436 EN**: Defines macro `HEXAGON_V64_PUT_H2(v, new)` for conditional compilation, shorthand, or API generation.
  **L436 CN**: 定义宏 `HEXAGON_V64_PUT_H2(v, new)`，用于条件编译、简写或 API 生成。
- **L437 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffff0000ffffffffLL) | (((HEXAGON_Vect64)((unsigned short)(new))) << 32LL))`.
  **L437 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffff0000ffffffffLL) | (((HEXAGON_Vect64)((unsigned short)(new))) << 32LL))`。
- **L438 EN**: Defines macro `HEXAGON_V64_PUT_H3(v, new)` for conditional compilation, shorthand, or API generation.
  **L438 CN**: 定义宏 `HEXAGON_V64_PUT_H3(v, new)`，用于条件编译、简写或 API 生成。
- **L439 EN**: Continues the surrounding expression or declaration: `(((v) & 0x0000ffffffffffffLL) | (((HEXAGON_Vect64)(new)) << 48LL))`.
  **L439 CN**: 继续构造周围的表达式或声明：`(((v) & 0x0000ffffffffffffLL) | (((HEXAGON_Vect64)(new)) << 48LL))`。
- **L440 EN**: Blank line separating nearby declarations or logic blocks.
  **L440 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L441 EN**: Closes the current preprocessor conditional block.
  **L441 CN**: 结束当前预处理条件块。
- **L442 EN**: Blank line separating nearby declarations or logic blocks.
  **L442 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L443 EN**: Comment explains nearby logic, constraints, or intent: `Set byte macros`.
  **L443 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set byte macros`。
- **L444 EN**: Blank line separating nearby declarations or logic blocks.
  **L444 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L445 EN**: Starts a preprocessor conditional block: `#ifdef __hexagon__`.
  **L445 CN**: 开始一个预处理条件块：`#ifdef __hexagon__`。
- **L446 EN**: Blank line separating nearby declarations or logic blocks.
  **L446 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L447 EN**: Defines macro `HEXAGON_V64_PUT_B0(v, new)` for conditional compilation, shorthand, or API generation.
  **L447 CN**: 定义宏 `HEXAGON_V64_PUT_B0(v, new)`，用于条件编译、简写或 API 生成。
- **L448 EN**: Continues logic associated with callable symbol `__extension__`.
  **L448 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L449 EN**: Declares union `union`.
  **L449 CN**: 声明 union `union`。
- **L450 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L450 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L451 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L451 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L452 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L452 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L453 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L453 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L454 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[0] = (new);                                        \`.
  **L454 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[0] = (new);                                        \`。
- **L455 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L455 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L456 EN**: Continues the surrounding expression or declaration: `})`.
  **L456 CN**: 继续构造周围的表达式或声明：`})`。

### Lines 457-480

````c
#define HEXAGON_V64_PUT_B1(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.b[1] = (new);                                        \
    _HEXAGON_V64_internal_union.d;                                                   \
  })
#define HEXAGON_V64_PUT_B2(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.b[2] = (new);                                        \
    _HEXAGON_V64_internal_union.d;                                                   \
  })
#define HEXAGON_V64_PUT_B3(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
````
- **L457 EN**: Defines macro `HEXAGON_V64_PUT_B1(v, new)` for conditional compilation, shorthand, or API generation.
  **L457 CN**: 定义宏 `HEXAGON_V64_PUT_B1(v, new)`，用于条件编译、简写或 API 生成。
- **L458 EN**: Continues logic associated with callable symbol `__extension__`.
  **L458 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L459 EN**: Declares union `union`.
  **L459 CN**: 声明 union `union`。
- **L460 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L460 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L461 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L461 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L462 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L462 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L463 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L463 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L464 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[1] = (new);                                        \`.
  **L464 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[1] = (new);                                        \`。
- **L465 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L465 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L466 EN**: Continues the surrounding expression or declaration: `})`.
  **L466 CN**: 继续构造周围的表达式或声明：`})`。
- **L467 EN**: Defines macro `HEXAGON_V64_PUT_B2(v, new)` for conditional compilation, shorthand, or API generation.
  **L467 CN**: 定义宏 `HEXAGON_V64_PUT_B2(v, new)`，用于条件编译、简写或 API 生成。
- **L468 EN**: Continues logic associated with callable symbol `__extension__`.
  **L468 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L469 EN**: Declares union `union`.
  **L469 CN**: 声明 union `union`。
- **L470 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L470 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L471 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L471 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L472 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L472 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L473 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L473 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L474 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[2] = (new);                                        \`.
  **L474 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[2] = (new);                                        \`。
- **L475 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L475 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L476 EN**: Continues the surrounding expression or declaration: `})`.
  **L476 CN**: 继续构造周围的表达式或声明：`})`。
- **L477 EN**: Defines macro `HEXAGON_V64_PUT_B3(v, new)` for conditional compilation, shorthand, or API generation.
  **L477 CN**: 定义宏 `HEXAGON_V64_PUT_B3(v, new)`，用于条件编译、简写或 API 生成。
- **L478 EN**: Continues logic associated with callable symbol `__extension__`.
  **L478 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L479 EN**: Declares union `union`.
  **L479 CN**: 声明 union `union`。
- **L480 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L480 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。

### Lines 481-504

````c
      char b[8];                                                               \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.b[3] = (new);                                        \
    _HEXAGON_V64_internal_union.d;                                                   \
  })
#define HEXAGON_V64_PUT_B4(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.b[4] = (new);                                        \
    _HEXAGON_V64_internal_union.d;                                                   \
  })
#define HEXAGON_V64_PUT_B5(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.b[5] = (new);                                        \
````
- **L481 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L481 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L482 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L482 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L483 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L483 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L484 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[3] = (new);                                        \`.
  **L484 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[3] = (new);                                        \`。
- **L485 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L485 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L486 EN**: Continues the surrounding expression or declaration: `})`.
  **L486 CN**: 继续构造周围的表达式或声明：`})`。
- **L487 EN**: Defines macro `HEXAGON_V64_PUT_B4(v, new)` for conditional compilation, shorthand, or API generation.
  **L487 CN**: 定义宏 `HEXAGON_V64_PUT_B4(v, new)`，用于条件编译、简写或 API 生成。
- **L488 EN**: Continues logic associated with callable symbol `__extension__`.
  **L488 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L489 EN**: Declares union `union`.
  **L489 CN**: 声明 union `union`。
- **L490 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L490 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L491 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L491 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L492 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L492 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L493 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L493 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L494 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[4] = (new);                                        \`.
  **L494 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[4] = (new);                                        \`。
- **L495 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L495 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L496 EN**: Continues the surrounding expression or declaration: `})`.
  **L496 CN**: 继续构造周围的表达式或声明：`})`。
- **L497 EN**: Defines macro `HEXAGON_V64_PUT_B5(v, new)` for conditional compilation, shorthand, or API generation.
  **L497 CN**: 定义宏 `HEXAGON_V64_PUT_B5(v, new)`，用于条件编译、简写或 API 生成。
- **L498 EN**: Continues logic associated with callable symbol `__extension__`.
  **L498 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L499 EN**: Declares union `union`.
  **L499 CN**: 声明 union `union`。
- **L500 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L500 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L501 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L501 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L502 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L502 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L503 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L503 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L504 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[5] = (new);                                        \`.
  **L504 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[5] = (new);                                        \`。

### Lines 505-528

````c
    _HEXAGON_V64_internal_union.d;                                                   \
  })
#define HEXAGON_V64_PUT_B6(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.b[6] = (new);                                        \
    _HEXAGON_V64_internal_union.d;                                                   \
  })
#define HEXAGON_V64_PUT_B7(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.d = (v);                                             \
    _HEXAGON_V64_internal_union.b[7] = (new);                                        \
    _HEXAGON_V64_internal_union.d;                                                   \
  })

#else /* !__hexagon__ */
````
- **L505 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L505 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L506 EN**: Continues the surrounding expression or declaration: `})`.
  **L506 CN**: 继续构造周围的表达式或声明：`})`。
- **L507 EN**: Defines macro `HEXAGON_V64_PUT_B6(v, new)` for conditional compilation, shorthand, or API generation.
  **L507 CN**: 定义宏 `HEXAGON_V64_PUT_B6(v, new)`，用于条件编译、简写或 API 生成。
- **L508 EN**: Continues logic associated with callable symbol `__extension__`.
  **L508 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L509 EN**: Declares union `union`.
  **L509 CN**: 声明 union `union`。
- **L510 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L510 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L511 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L511 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L512 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L512 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L513 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L513 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L514 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[6] = (new);                                        \`.
  **L514 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[6] = (new);                                        \`。
- **L515 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L515 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L516 EN**: Continues the surrounding expression or declaration: `})`.
  **L516 CN**: 继续构造周围的表达式或声明：`})`。
- **L517 EN**: Defines macro `HEXAGON_V64_PUT_B7(v, new)` for conditional compilation, shorthand, or API generation.
  **L517 CN**: 定义宏 `HEXAGON_V64_PUT_B7(v, new)`，用于条件编译、简写或 API 生成。
- **L518 EN**: Continues logic associated with callable symbol `__extension__`.
  **L518 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L519 EN**: Declares union `union`.
  **L519 CN**: 声明 union `union`。
- **L520 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L520 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L521 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L521 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L522 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L522 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L523 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d = (v);                                             \`.
  **L523 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d = (v);                                             \`。
- **L524 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[7] = (new);                                        \`.
  **L524 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[7] = (new);                                        \`。
- **L525 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L525 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L526 EN**: Continues the surrounding expression or declaration: `})`.
  **L526 CN**: 继续构造周围的表达式或声明：`})`。
- **L527 EN**: Blank line separating nearby declarations or logic blocks.
  **L527 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L528 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L528 CN**: 继续为当前目标或特性集合选择预处理分支。

### Lines 529-552

````c

#define HEXAGON_V64_PUT_B0(v, new)                                                   \
  (((v) & 0xffffffffffffff00LL) | ((HEXAGON_Vect64)((unsigned char)(new))))
#define HEXAGON_V64_PUT_B1(v, new)                                                   \
  (((v) & 0xffffffffffff00ffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 8LL))
#define HEXAGON_V64_PUT_B2(v, new)                                                   \
  (((v) & 0xffffffffff00ffffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 16LL))
#define HEXAGON_V64_PUT_B3(v, new)                                                   \
  (((v) & 0xffffffff00ffffffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 24LL))
#define HEXAGON_V64_PUT_B4(v, new)                                                   \
  (((v) & 0xffffff00ffffffffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 32LL))
#define HEXAGON_V64_PUT_B5(v, new)                                                   \
  (((v) & 0xffff00ffffffffffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 40LL))
#define HEXAGON_V64_PUT_B6(v, new)                                                   \
  (((v) & 0xff00ffffffffffffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 48LL))
#define HEXAGON_V64_PUT_B7(v, new)                                                   \
  (((v) & 0x00ffffffffffffffLL) | (((HEXAGON_Vect64)(new)) << 56LL))

#endif /* !__hexagon__ */

/* NOTE: All create macros return a HEXAGON_Vect64 type */

/* Create from a doubleword */

````
- **L529 EN**: Blank line separating nearby declarations or logic blocks.
  **L529 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L530 EN**: Defines macro `HEXAGON_V64_PUT_B0(v, new)` for conditional compilation, shorthand, or API generation.
  **L530 CN**: 定义宏 `HEXAGON_V64_PUT_B0(v, new)`，用于条件编译、简写或 API 生成。
- **L531 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffffffffffff00LL) | ((HEXAGON_Vect64)((unsigned char)(new))))`.
  **L531 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffffffffffff00LL) | ((HEXAGON_Vect64)((unsigned char)(new))))`。
- **L532 EN**: Defines macro `HEXAGON_V64_PUT_B1(v, new)` for conditional compilation, shorthand, or API generation.
  **L532 CN**: 定义宏 `HEXAGON_V64_PUT_B1(v, new)`，用于条件编译、简写或 API 生成。
- **L533 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffffffffff00ffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 8LL))`.
  **L533 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffffffffff00ffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 8LL))`。
- **L534 EN**: Defines macro `HEXAGON_V64_PUT_B2(v, new)` for conditional compilation, shorthand, or API generation.
  **L534 CN**: 定义宏 `HEXAGON_V64_PUT_B2(v, new)`，用于条件编译、简写或 API 生成。
- **L535 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffffffff00ffffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 16LL))`.
  **L535 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffffffff00ffffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 16LL))`。
- **L536 EN**: Defines macro `HEXAGON_V64_PUT_B3(v, new)` for conditional compilation, shorthand, or API generation.
  **L536 CN**: 定义宏 `HEXAGON_V64_PUT_B3(v, new)`，用于条件编译、简写或 API 生成。
- **L537 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffffff00ffffffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 24LL))`.
  **L537 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffffff00ffffffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 24LL))`。
- **L538 EN**: Defines macro `HEXAGON_V64_PUT_B4(v, new)` for conditional compilation, shorthand, or API generation.
  **L538 CN**: 定义宏 `HEXAGON_V64_PUT_B4(v, new)`，用于条件编译、简写或 API 生成。
- **L539 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffff00ffffffffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 32LL))`.
  **L539 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffff00ffffffffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 32LL))`。
- **L540 EN**: Defines macro `HEXAGON_V64_PUT_B5(v, new)` for conditional compilation, shorthand, or API generation.
  **L540 CN**: 定义宏 `HEXAGON_V64_PUT_B5(v, new)`，用于条件编译、简写或 API 生成。
- **L541 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffff00ffffffffffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 40LL))`.
  **L541 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffff00ffffffffffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 40LL))`。
- **L542 EN**: Defines macro `HEXAGON_V64_PUT_B6(v, new)` for conditional compilation, shorthand, or API generation.
  **L542 CN**: 定义宏 `HEXAGON_V64_PUT_B6(v, new)`，用于条件编译、简写或 API 生成。
- **L543 EN**: Continues the surrounding expression or declaration: `(((v) & 0xff00ffffffffffffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 48LL))`.
  **L543 CN**: 继续构造周围的表达式或声明：`(((v) & 0xff00ffffffffffffLL) | (((HEXAGON_Vect64)((unsigned char)(new))) << 48LL))`。
- **L544 EN**: Defines macro `HEXAGON_V64_PUT_B7(v, new)` for conditional compilation, shorthand, or API generation.
  **L544 CN**: 定义宏 `HEXAGON_V64_PUT_B7(v, new)`，用于条件编译、简写或 API 生成。
- **L545 EN**: Continues the surrounding expression or declaration: `(((v) & 0x00ffffffffffffffLL) | (((HEXAGON_Vect64)(new)) << 56LL))`.
  **L545 CN**: 继续构造周围的表达式或声明：`(((v) & 0x00ffffffffffffffLL) | (((HEXAGON_Vect64)(new)) << 56LL))`。
- **L546 EN**: Blank line separating nearby declarations or logic blocks.
  **L546 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L547 EN**: Closes the current preprocessor conditional block.
  **L547 CN**: 结束当前预处理条件块。
- **L548 EN**: Blank line separating nearby declarations or logic blocks.
  **L548 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L549 EN**: Comment highlights an implementation note: `NOTE: All create macros return a HEXAGON_Vect64 type`.
  **L549 CN**: 注释强调一条实现说明：`NOTE: All create macros return a HEXAGON_Vect64 type`。
- **L550 EN**: Blank line separating nearby declarations or logic blocks.
  **L550 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L551 EN**: Comment explains nearby logic, constraints, or intent: `Create from a doubleword`.
  **L551 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create from a doubleword`。
- **L552 EN**: Blank line separating nearby declarations or logic blocks.
  **L552 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 553-576

````c
#define HEXAGON_V64_CREATE_D(d) (d)

/* Create from words */

#ifdef __hexagon__

#define HEXAGON_V64_CREATE_W(w1, w0)                                                 \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      int w[2];                                                                \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.w[0] = (w0);                                         \
    _HEXAGON_V64_internal_union.w[1] = (w1);                                         \
    _HEXAGON_V64_internal_union.d;                                                   \
  })

#else /* !__hexagon__ */

#define HEXAGON_V64_CREATE_W(w1, w0)                                                 \
  ((((HEXAGON_Vect64)(w1)) << 32LL) | ((HEXAGON_Vect64)((w0) & 0xffffffff)))

#endif /* !__hexagon__ */

````
- **L553 EN**: Defines macro `HEXAGON_V64_CREATE_D(d)` for conditional compilation, shorthand, or API generation.
  **L553 CN**: 定义宏 `HEXAGON_V64_CREATE_D(d)`，用于条件编译、简写或 API 生成。
- **L554 EN**: Blank line separating nearby declarations or logic blocks.
  **L554 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L555 EN**: Comment explains nearby logic, constraints, or intent: `Create from words`.
  **L555 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create from words`。
- **L556 EN**: Blank line separating nearby declarations or logic blocks.
  **L556 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L557 EN**: Starts a preprocessor conditional block: `#ifdef __hexagon__`.
  **L557 CN**: 开始一个预处理条件块：`#ifdef __hexagon__`。
- **L558 EN**: Blank line separating nearby declarations or logic blocks.
  **L558 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L559 EN**: Defines macro `HEXAGON_V64_CREATE_W(w1, w0)` for conditional compilation, shorthand, or API generation.
  **L559 CN**: 定义宏 `HEXAGON_V64_CREATE_W(w1, w0)`，用于条件编译、简写或 API 生成。
- **L560 EN**: Continues logic associated with callable symbol `__extension__`.
  **L560 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L561 EN**: Declares union `union`.
  **L561 CN**: 声明 union `union`。
- **L562 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L562 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L563 EN**: Continues the surrounding expression or declaration: `int w[2];                                                                \`.
  **L563 CN**: 继续构造周围的表达式或声明：`int w[2];                                                                \`。
- **L564 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L564 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L565 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.w[0] = (w0);                                         \`.
  **L565 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.w[0] = (w0);                                         \`。
- **L566 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.w[1] = (w1);                                         \`.
  **L566 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.w[1] = (w1);                                         \`。
- **L567 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L567 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L568 EN**: Continues the surrounding expression or declaration: `})`.
  **L568 CN**: 继续构造周围的表达式或声明：`})`。
- **L569 EN**: Blank line separating nearby declarations or logic blocks.
  **L569 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L570 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L570 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L571 EN**: Blank line separating nearby declarations or logic blocks.
  **L571 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L572 EN**: Defines macro `HEXAGON_V64_CREATE_W(w1, w0)` for conditional compilation, shorthand, or API generation.
  **L572 CN**: 定义宏 `HEXAGON_V64_CREATE_W(w1, w0)`，用于条件编译、简写或 API 生成。
- **L573 EN**: Continues the surrounding expression or declaration: `((((HEXAGON_Vect64)(w1)) << 32LL) | ((HEXAGON_Vect64)((w0) & 0xffffffff)))`.
  **L573 CN**: 继续构造周围的表达式或声明：`((((HEXAGON_Vect64)(w1)) << 32LL) | ((HEXAGON_Vect64)((w0) & 0xffffffff)))`。
- **L574 EN**: Blank line separating nearby declarations or logic blocks.
  **L574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L575 EN**: Closes the current preprocessor conditional block.
  **L575 CN**: 结束当前预处理条件块。
- **L576 EN**: Blank line separating nearby declarations or logic blocks.
  **L576 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 577-600

````c
/* Create from half words */

#ifdef __hexagon__

#define HEXAGON_V64_CREATE_H(h3, h2, h1, h0)                                         \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.h[0] = (h0);                                         \
    _HEXAGON_V64_internal_union.h[1] = (h1);                                         \
    _HEXAGON_V64_internal_union.h[2] = (h2);                                         \
    _HEXAGON_V64_internal_union.h[3] = (h3);                                         \
    _HEXAGON_V64_internal_union.d;                                                   \
  })

#else /* !__hexagon__ */

#define HEXAGON_V64_CREATE_H(h3, h2, h1, h0)                                         \
  ((((HEXAGON_Vect64)(h3)) << 48LL) | (((HEXAGON_Vect64)((h2) & 0xffff)) << 32LL) |        \
   (((HEXAGON_Vect64)((h1) & 0xffff)) << 16LL) | ((HEXAGON_Vect64)((h0) & 0xffff)))

#endif /* !__hexagon__ */
````
- **L577 EN**: Comment explains nearby logic, constraints, or intent: `Create from half words`.
  **L577 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create from half words`。
- **L578 EN**: Blank line separating nearby declarations or logic blocks.
  **L578 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L579 EN**: Starts a preprocessor conditional block: `#ifdef __hexagon__`.
  **L579 CN**: 开始一个预处理条件块：`#ifdef __hexagon__`。
- **L580 EN**: Blank line separating nearby declarations or logic blocks.
  **L580 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L581 EN**: Defines macro `HEXAGON_V64_CREATE_H(h3, h2, h1, h0)` for conditional compilation, shorthand, or API generation.
  **L581 CN**: 定义宏 `HEXAGON_V64_CREATE_H(h3, h2, h1, h0)`，用于条件编译、简写或 API 生成。
- **L582 EN**: Continues logic associated with callable symbol `__extension__`.
  **L582 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L583 EN**: Declares union `union`.
  **L583 CN**: 声明 union `union`。
- **L584 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L584 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L585 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L585 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L586 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L586 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L587 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.h[0] = (h0);                                         \`.
  **L587 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.h[0] = (h0);                                         \`。
- **L588 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.h[1] = (h1);                                         \`.
  **L588 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.h[1] = (h1);                                         \`。
- **L589 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.h[2] = (h2);                                         \`.
  **L589 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.h[2] = (h2);                                         \`。
- **L590 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.h[3] = (h3);                                         \`.
  **L590 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.h[3] = (h3);                                         \`。
- **L591 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L591 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L592 EN**: Continues the surrounding expression or declaration: `})`.
  **L592 CN**: 继续构造周围的表达式或声明：`})`。
- **L593 EN**: Blank line separating nearby declarations or logic blocks.
  **L593 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L594 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L594 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L595 EN**: Blank line separating nearby declarations or logic blocks.
  **L595 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L596 EN**: Defines macro `HEXAGON_V64_CREATE_H(h3, h2, h1, h0)` for conditional compilation, shorthand, or API generation.
  **L596 CN**: 定义宏 `HEXAGON_V64_CREATE_H(h3, h2, h1, h0)`，用于条件编译、简写或 API 生成。
- **L597 EN**: Continues the surrounding expression or declaration: `((((HEXAGON_Vect64)(h3)) << 48LL) | (((HEXAGON_Vect64)((h2) & 0xffff)) << 32LL) |        \`.
  **L597 CN**: 继续构造周围的表达式或声明：`((((HEXAGON_Vect64)(h3)) << 48LL) | (((HEXAGON_Vect64)((h2) & 0xffff)) << 32LL) |        \`。
- **L598 EN**: Continues the surrounding expression or declaration: `(((HEXAGON_Vect64)((h1) & 0xffff)) << 16LL) | ((HEXAGON_Vect64)((h0) & 0xffff)))`.
  **L598 CN**: 继续构造周围的表达式或声明：`(((HEXAGON_Vect64)((h1) & 0xffff)) << 16LL) | ((HEXAGON_Vect64)((h0) & 0xffff)))`。
- **L599 EN**: Blank line separating nearby declarations or logic blocks.
  **L599 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L600 EN**: Closes the current preprocessor conditional block.
  **L600 CN**: 结束当前预处理条件块。

### Lines 601-624

````c

/* Create from bytes */

#ifdef __hexagon__

#define HEXAGON_V64_CREATE_B(b7, b6, b5, b4, b3, b2, b1, b0)                         \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
    } _HEXAGON_V64_internal_union;                                                   \
    _HEXAGON_V64_internal_union.b[0] = (b0);                                         \
    _HEXAGON_V64_internal_union.b[1] = (b1);                                         \
    _HEXAGON_V64_internal_union.b[2] = (b2);                                         \
    _HEXAGON_V64_internal_union.b[3] = (b3);                                         \
    _HEXAGON_V64_internal_union.b[4] = (b4);                                         \
    _HEXAGON_V64_internal_union.b[5] = (b5);                                         \
    _HEXAGON_V64_internal_union.b[6] = (b6);                                         \
    _HEXAGON_V64_internal_union.b[7] = (b7);                                         \
    _HEXAGON_V64_internal_union.d;                                                   \
  })

#else /* !__hexagon__ */

````
- **L601 EN**: Blank line separating nearby declarations or logic blocks.
  **L601 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L602 EN**: Comment explains nearby logic, constraints, or intent: `Create from bytes`.
  **L602 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create from bytes`。
- **L603 EN**: Blank line separating nearby declarations or logic blocks.
  **L603 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L604 EN**: Starts a preprocessor conditional block: `#ifdef __hexagon__`.
  **L604 CN**: 开始一个预处理条件块：`#ifdef __hexagon__`。
- **L605 EN**: Blank line separating nearby declarations or logic blocks.
  **L605 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L606 EN**: Defines macro `HEXAGON_V64_CREATE_B(b7, b6, b5, b4, b3, b2, b1, b0)` for conditional compilation, shorthand, or API generation.
  **L606 CN**: 定义宏 `HEXAGON_V64_CREATE_B(b7, b6, b5, b4, b3, b2, b1, b0)`，用于条件编译、简写或 API 生成。
- **L607 EN**: Continues logic associated with callable symbol `__extension__`.
  **L607 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L608 EN**: Declares union `union`.
  **L608 CN**: 声明 union `union`。
- **L609 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L609 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L610 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L610 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L611 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V64_internal_union;                                                   \`.
  **L611 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V64_internal_union;                                                   \`。
- **L612 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[0] = (b0);                                         \`.
  **L612 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[0] = (b0);                                         \`。
- **L613 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[1] = (b1);                                         \`.
  **L613 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[1] = (b1);                                         \`。
- **L614 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[2] = (b2);                                         \`.
  **L614 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[2] = (b2);                                         \`。
- **L615 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[3] = (b3);                                         \`.
  **L615 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[3] = (b3);                                         \`。
- **L616 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[4] = (b4);                                         \`.
  **L616 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[4] = (b4);                                         \`。
- **L617 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[5] = (b5);                                         \`.
  **L617 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[5] = (b5);                                         \`。
- **L618 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[6] = (b6);                                         \`.
  **L618 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[6] = (b6);                                         \`。
- **L619 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.b[7] = (b7);                                         \`.
  **L619 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.b[7] = (b7);                                         \`。
- **L620 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V64_internal_union.d;                                                   \`.
  **L620 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V64_internal_union.d;                                                   \`。
- **L621 EN**: Continues the surrounding expression or declaration: `})`.
  **L621 CN**: 继续构造周围的表达式或声明：`})`。
- **L622 EN**: Blank line separating nearby declarations or logic blocks.
  **L622 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L623 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L623 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L624 EN**: Blank line separating nearby declarations or logic blocks.
  **L624 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 625-648

````c
#define HEXAGON_V64_CREATE_B(b7, b6, b5, b4, b3, b2, b1, b0)                         \
  ((((HEXAGON_Vect64)(b7)) << 56LL) | (((HEXAGON_Vect64)((b6) & 0xff)) << 48LL) |          \
   (((HEXAGON_Vect64)((b5) & 0xff)) << 40LL) | (((HEXAGON_Vect64)((b4) & 0xff)) << 32LL) | \
   (((HEXAGON_Vect64)((b3) & 0xff)) << 24LL) | (((HEXAGON_Vect64)((b2) & 0xff)) << 16LL) | \
   (((HEXAGON_Vect64)((b1) & 0xff)) << 8LL) | ((HEXAGON_Vect64)((b0) & 0xff)))

#endif /* !__hexagon__ */

#ifdef __cplusplus

class HEXAGON_Vect64C {
public:
  // Constructors
  HEXAGON_Vect64C(long long d = 0) : data(d) {};
  HEXAGON_Vect64C(int w1, int w0) : data(HEXAGON_V64_CREATE_W(w1, w0)) {};
  HEXAGON_Vect64C(short h3, short h2, short h1, short h0)
      : data(HEXAGON_V64_CREATE_H(h3, h2, h1, h0)) {};
  HEXAGON_Vect64C(signed char b7, signed char b6, signed char b5, signed char b4,
            signed char b3, signed char b2, signed char b1, signed char b0)
      : data(HEXAGON_V64_CREATE_B(b7, b6, b5, b4, b3, b2, b1, b0)) {};
  HEXAGON_Vect64C(const HEXAGON_Vect64C &v) : data(v.data) {};

  HEXAGON_Vect64C &operator=(const HEXAGON_Vect64C &v) {
    data = v.data;
````
- **L625 EN**: Defines macro `HEXAGON_V64_CREATE_B(b7, b6, b5, b4, b3, b2, b1, b0)` for conditional compilation, shorthand, or API generation.
  **L625 CN**: 定义宏 `HEXAGON_V64_CREATE_B(b7, b6, b5, b4, b3, b2, b1, b0)`，用于条件编译、简写或 API 生成。
- **L626 EN**: Continues the surrounding expression or declaration: `((((HEXAGON_Vect64)(b7)) << 56LL) | (((HEXAGON_Vect64)((b6) & 0xff)) << 48LL) |          \`.
  **L626 CN**: 继续构造周围的表达式或声明：`((((HEXAGON_Vect64)(b7)) << 56LL) | (((HEXAGON_Vect64)((b6) & 0xff)) << 48LL) |          \`。
- **L627 EN**: Continues the surrounding expression or declaration: `(((HEXAGON_Vect64)((b5) & 0xff)) << 40LL) | (((HEXAGON_Vect64)((b4) & 0xff)) << 32LL) | \`.
  **L627 CN**: 继续构造周围的表达式或声明：`(((HEXAGON_Vect64)((b5) & 0xff)) << 40LL) | (((HEXAGON_Vect64)((b4) & 0xff)) << 32LL) | \`。
- **L628 EN**: Continues the surrounding expression or declaration: `(((HEXAGON_Vect64)((b3) & 0xff)) << 24LL) | (((HEXAGON_Vect64)((b2) & 0xff)) << 16LL) | \`.
  **L628 CN**: 继续构造周围的表达式或声明：`(((HEXAGON_Vect64)((b3) & 0xff)) << 24LL) | (((HEXAGON_Vect64)((b2) & 0xff)) << 16LL) | \`。
- **L629 EN**: Continues the surrounding expression or declaration: `(((HEXAGON_Vect64)((b1) & 0xff)) << 8LL) | ((HEXAGON_Vect64)((b0) & 0xff)))`.
  **L629 CN**: 继续构造周围的表达式或声明：`(((HEXAGON_Vect64)((b1) & 0xff)) << 8LL) | ((HEXAGON_Vect64)((b0) & 0xff)))`。
- **L630 EN**: Blank line separating nearby declarations or logic blocks.
  **L630 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L631 EN**: Closes the current preprocessor conditional block.
  **L631 CN**: 结束当前预处理条件块。
- **L632 EN**: Blank line separating nearby declarations or logic blocks.
  **L632 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L633 EN**: Starts a preprocessor conditional block: `#ifdef __cplusplus`.
  **L633 CN**: 开始一个预处理条件块：`#ifdef __cplusplus`。
- **L634 EN**: Blank line separating nearby declarations or logic blocks.
  **L634 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L635 EN**: Declares class `HEXAGON_Vect64C`.
  **L635 CN**: 声明 class `HEXAGON_Vect64C`。
- **L636 EN**: Sets the access level for following class members to `public`.
  **L636 CN**: 将后续类成员的访问级别设为 `public`。
- **L637 EN**: Comment explains nearby logic, constraints, or intent: `Constructors`.
  **L637 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructors`。
- **L638 EN**: Executes a call or declaration centered on `HEXAGON_Vect64C`.
  **L638 CN**: 执行以 `HEXAGON_Vect64C` 为核心的调用或声明。
- **L639 EN**: Executes a call or declaration centered on `HEXAGON_Vect64C`.
  **L639 CN**: 执行以 `HEXAGON_Vect64C` 为核心的调用或声明。
- **L640 EN**: Continues logic associated with callable symbol `HEXAGON_Vect64C`.
  **L640 CN**: 继续与可调用符号 `HEXAGON_Vect64C` 相关的逻辑。
- **L641 EN**: Executes a call or declaration centered on `data`.
  **L641 CN**: 执行以 `data` 为核心的调用或声明。
- **L642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HEXAGON_Vect64C(signed char b7, signed char b6, signed char b5, signed char b4,`.
  **L642 CN**: 继续一个多行参数列表、初始化器或聚合项：`HEXAGON_Vect64C(signed char b7, signed char b6, signed char b5, signed char b4,`。
- **L643 EN**: Continues the surrounding expression or declaration: `signed char b3, signed char b2, signed char b1, signed char b0)`.
  **L643 CN**: 继续构造周围的表达式或声明：`signed char b3, signed char b2, signed char b1, signed char b0)`。
- **L644 EN**: Executes a call or declaration centered on `data`.
  **L644 CN**: 执行以 `data` 为核心的调用或声明。
- **L645 EN**: Executes a call or declaration centered on `HEXAGON_Vect64C`.
  **L645 CN**: 执行以 `HEXAGON_Vect64C` 为核心的调用或声明。
- **L646 EN**: Blank line separating nearby declarations or logic blocks.
  **L646 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L647 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect64C &operator=(const HEXAGON_Vect64C &v) {`.
  **L647 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect64C &operator=(const HEXAGON_Vect64C &v) {`。
- **L648 EN**: Adds a standalone statement or declaration: `data = v.data;`.
  **L648 CN**: 添加一条独立语句或声明：`data = v.data;`。

### Lines 649-672

````c
    return *this;
  };

  operator long long() {
    return data;
  };

  // Extract doubleword methods
  long long D(void) {
    return HEXAGON_V64_GET_D(data);
  };
  unsigned long long UD(void) {
    return HEXAGON_V64_GET_UD(data);
  };

  // Extract word methods
  int W0(void) {
    return HEXAGON_V64_GET_W0(data);
  };
  int W1(void) {
    return HEXAGON_V64_GET_W1(data);
  };
  unsigned int UW0(void) {
    return HEXAGON_V64_GET_UW0(data);
````
- **L649 EN**: Returns from the current function with `*this`.
  **L649 CN**: 以 `*this` 从当前函数返回。
- **L650 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L650 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L651 EN**: Blank line separating nearby declarations or logic blocks.
  **L651 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L652 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `operator long long() {`.
  **L652 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`operator long long() {`。
- **L653 EN**: Returns from the current function with `data`.
  **L653 CN**: 以 `data` 从当前函数返回。
- **L654 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L654 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L655 EN**: Blank line separating nearby declarations or logic blocks.
  **L655 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L656 EN**: Comment explains nearby logic, constraints, or intent: `Extract doubleword methods`.
  **L656 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract doubleword methods`。
- **L657 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `long long D(void) {`.
  **L657 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`long long D(void) {`。
- **L658 EN**: Returns from the current function with `HEXAGON_V64_GET_D(data)`.
  **L658 CN**: 以 `HEXAGON_V64_GET_D(data)` 从当前函数返回。
- **L659 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L659 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L660 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned long long UD(void) {`.
  **L660 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned long long UD(void) {`。
- **L661 EN**: Returns from the current function with `HEXAGON_V64_GET_UD(data)`.
  **L661 CN**: 以 `HEXAGON_V64_GET_UD(data)` 从当前函数返回。
- **L662 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L662 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L663 EN**: Blank line separating nearby declarations or logic blocks.
  **L663 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L664 EN**: Comment explains nearby logic, constraints, or intent: `Extract word methods`.
  **L664 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract word methods`。
- **L665 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `int W0(void) {`.
  **L665 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`int W0(void) {`。
- **L666 EN**: Returns from the current function with `HEXAGON_V64_GET_W0(data)`.
  **L666 CN**: 以 `HEXAGON_V64_GET_W0(data)` 从当前函数返回。
- **L667 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L667 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L668 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `int W1(void) {`.
  **L668 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`int W1(void) {`。
- **L669 EN**: Returns from the current function with `HEXAGON_V64_GET_W1(data)`.
  **L669 CN**: 以 `HEXAGON_V64_GET_W1(data)` 从当前函数返回。
- **L670 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L670 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L671 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned int UW0(void) {`.
  **L671 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned int UW0(void) {`。
- **L672 EN**: Returns from the current function with `HEXAGON_V64_GET_UW0(data)`.
  **L672 CN**: 以 `HEXAGON_V64_GET_UW0(data)` 从当前函数返回。

### Lines 673-696

````c
  };
  unsigned int UW1(void) {
    return HEXAGON_V64_GET_UW1(data);
  };

  // Extract half word methods
  short H0(void) {
    return HEXAGON_V64_GET_H0(data);
  };
  short H1(void) {
    return HEXAGON_V64_GET_H1(data);
  };
  short H2(void) {
    return HEXAGON_V64_GET_H2(data);
  };
  short H3(void) {
    return HEXAGON_V64_GET_H3(data);
  };
  unsigned short UH0(void) {
    return HEXAGON_V64_GET_UH0(data);
  };
  unsigned short UH1(void) {
    return HEXAGON_V64_GET_UH1(data);
  };
````
- **L673 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L673 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L674 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned int UW1(void) {`.
  **L674 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned int UW1(void) {`。
- **L675 EN**: Returns from the current function with `HEXAGON_V64_GET_UW1(data)`.
  **L675 CN**: 以 `HEXAGON_V64_GET_UW1(data)` 从当前函数返回。
- **L676 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L676 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L677 EN**: Blank line separating nearby declarations or logic blocks.
  **L677 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L678 EN**: Comment explains nearby logic, constraints, or intent: `Extract half word methods`.
  **L678 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract half word methods`。
- **L679 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `short H0(void) {`.
  **L679 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`short H0(void) {`。
- **L680 EN**: Returns from the current function with `HEXAGON_V64_GET_H0(data)`.
  **L680 CN**: 以 `HEXAGON_V64_GET_H0(data)` 从当前函数返回。
- **L681 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L681 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L682 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `short H1(void) {`.
  **L682 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`short H1(void) {`。
- **L683 EN**: Returns from the current function with `HEXAGON_V64_GET_H1(data)`.
  **L683 CN**: 以 `HEXAGON_V64_GET_H1(data)` 从当前函数返回。
- **L684 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L684 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L685 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `short H2(void) {`.
  **L685 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`short H2(void) {`。
- **L686 EN**: Returns from the current function with `HEXAGON_V64_GET_H2(data)`.
  **L686 CN**: 以 `HEXAGON_V64_GET_H2(data)` 从当前函数返回。
- **L687 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L687 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L688 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `short H3(void) {`.
  **L688 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`short H3(void) {`。
- **L689 EN**: Returns from the current function with `HEXAGON_V64_GET_H3(data)`.
  **L689 CN**: 以 `HEXAGON_V64_GET_H3(data)` 从当前函数返回。
- **L690 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L690 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L691 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned short UH0(void) {`.
  **L691 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned short UH0(void) {`。
- **L692 EN**: Returns from the current function with `HEXAGON_V64_GET_UH0(data)`.
  **L692 CN**: 以 `HEXAGON_V64_GET_UH0(data)` 从当前函数返回。
- **L693 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L693 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L694 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned short UH1(void) {`.
  **L694 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned short UH1(void) {`。
- **L695 EN**: Returns from the current function with `HEXAGON_V64_GET_UH1(data)`.
  **L695 CN**: 以 `HEXAGON_V64_GET_UH1(data)` 从当前函数返回。
- **L696 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L696 CN**: 结束当前声明作用域，例如结构体、枚举或类。

### Lines 697-720

````c
  unsigned short UH2(void) {
    return HEXAGON_V64_GET_UH2(data);
  };
  unsigned short UH3(void) {
    return HEXAGON_V64_GET_UH3(data);
  };

  // Extract byte methods
  signed char b0(void) { return HEXAGON_V64_GET_B0(data); };
  signed char B0(void) { return b0(); };
  signed char B1(void) {
    return HEXAGON_V64_GET_B1(data);
  };
  signed char B2(void) {
    return HEXAGON_V64_GET_B2(data);
  };
  signed char B3(void) {
    return HEXAGON_V64_GET_B3(data);
  };
  signed char B4(void) {
    return HEXAGON_V64_GET_B4(data);
  };
  signed char B5(void) {
    return HEXAGON_V64_GET_B5(data);
````
- **L697 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned short UH2(void) {`.
  **L697 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned short UH2(void) {`。
- **L698 EN**: Returns from the current function with `HEXAGON_V64_GET_UH2(data)`.
  **L698 CN**: 以 `HEXAGON_V64_GET_UH2(data)` 从当前函数返回。
- **L699 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L699 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L700 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned short UH3(void) {`.
  **L700 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned short UH3(void) {`。
- **L701 EN**: Returns from the current function with `HEXAGON_V64_GET_UH3(data)`.
  **L701 CN**: 以 `HEXAGON_V64_GET_UH3(data)` 从当前函数返回。
- **L702 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L702 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L703 EN**: Blank line separating nearby declarations or logic blocks.
  **L703 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L704 EN**: Comment explains nearby logic, constraints, or intent: `Extract byte methods`.
  **L704 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract byte methods`。
- **L705 EN**: Executes a call or declaration centered on `b0`.
  **L705 CN**: 执行以 `b0` 为核心的调用或声明。
- **L706 EN**: Executes a call or declaration centered on `B0`.
  **L706 CN**: 执行以 `B0` 为核心的调用或声明。
- **L707 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B1(void) {`.
  **L707 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B1(void) {`。
- **L708 EN**: Returns from the current function with `HEXAGON_V64_GET_B1(data)`.
  **L708 CN**: 以 `HEXAGON_V64_GET_B1(data)` 从当前函数返回。
- **L709 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L709 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L710 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B2(void) {`.
  **L710 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B2(void) {`。
- **L711 EN**: Returns from the current function with `HEXAGON_V64_GET_B2(data)`.
  **L711 CN**: 以 `HEXAGON_V64_GET_B2(data)` 从当前函数返回。
- **L712 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L712 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L713 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B3(void) {`.
  **L713 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B3(void) {`。
- **L714 EN**: Returns from the current function with `HEXAGON_V64_GET_B3(data)`.
  **L714 CN**: 以 `HEXAGON_V64_GET_B3(data)` 从当前函数返回。
- **L715 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L715 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L716 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B4(void) {`.
  **L716 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B4(void) {`。
- **L717 EN**: Returns from the current function with `HEXAGON_V64_GET_B4(data)`.
  **L717 CN**: 以 `HEXAGON_V64_GET_B4(data)` 从当前函数返回。
- **L718 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L718 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L719 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B5(void) {`.
  **L719 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B5(void) {`。
- **L720 EN**: Returns from the current function with `HEXAGON_V64_GET_B5(data)`.
  **L720 CN**: 以 `HEXAGON_V64_GET_B5(data)` 从当前函数返回。

### Lines 721-744

````c
  };
  signed char B6(void) {
    return HEXAGON_V64_GET_B6(data);
  };
  signed char B7(void) {
    return HEXAGON_V64_GET_B7(data);
  };
  unsigned char UB0(void) {
    return HEXAGON_V64_GET_UB0(data);
  };
  unsigned char UB1(void) {
    return HEXAGON_V64_GET_UB1(data);
  };
  unsigned char UB2(void) {
    return HEXAGON_V64_GET_UB2(data);
  };
  unsigned char UB3(void) {
    return HEXAGON_V64_GET_UB3(data);
  };
  unsigned char UB4(void) {
    return HEXAGON_V64_GET_UB4(data);
  };
  unsigned char UB5(void) {
    return HEXAGON_V64_GET_UB5(data);
````
- **L721 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L721 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L722 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B6(void) {`.
  **L722 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B6(void) {`。
- **L723 EN**: Returns from the current function with `HEXAGON_V64_GET_B6(data)`.
  **L723 CN**: 以 `HEXAGON_V64_GET_B6(data)` 从当前函数返回。
- **L724 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L724 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L725 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B7(void) {`.
  **L725 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B7(void) {`。
- **L726 EN**: Returns from the current function with `HEXAGON_V64_GET_B7(data)`.
  **L726 CN**: 以 `HEXAGON_V64_GET_B7(data)` 从当前函数返回。
- **L727 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L727 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L728 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB0(void) {`.
  **L728 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB0(void) {`。
- **L729 EN**: Returns from the current function with `HEXAGON_V64_GET_UB0(data)`.
  **L729 CN**: 以 `HEXAGON_V64_GET_UB0(data)` 从当前函数返回。
- **L730 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L730 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L731 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB1(void) {`.
  **L731 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB1(void) {`。
- **L732 EN**: Returns from the current function with `HEXAGON_V64_GET_UB1(data)`.
  **L732 CN**: 以 `HEXAGON_V64_GET_UB1(data)` 从当前函数返回。
- **L733 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L733 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L734 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB2(void) {`.
  **L734 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB2(void) {`。
- **L735 EN**: Returns from the current function with `HEXAGON_V64_GET_UB2(data)`.
  **L735 CN**: 以 `HEXAGON_V64_GET_UB2(data)` 从当前函数返回。
- **L736 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L736 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L737 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB3(void) {`.
  **L737 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB3(void) {`。
- **L738 EN**: Returns from the current function with `HEXAGON_V64_GET_UB3(data)`.
  **L738 CN**: 以 `HEXAGON_V64_GET_UB3(data)` 从当前函数返回。
- **L739 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L739 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L740 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB4(void) {`.
  **L740 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB4(void) {`。
- **L741 EN**: Returns from the current function with `HEXAGON_V64_GET_UB4(data)`.
  **L741 CN**: 以 `HEXAGON_V64_GET_UB4(data)` 从当前函数返回。
- **L742 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L742 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L743 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB5(void) {`.
  **L743 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB5(void) {`。
- **L744 EN**: Returns from the current function with `HEXAGON_V64_GET_UB5(data)`.
  **L744 CN**: 以 `HEXAGON_V64_GET_UB5(data)` 从当前函数返回。

### Lines 745-768

````c
  };
  unsigned char UB6(void) {
    return HEXAGON_V64_GET_UB6(data);
  };
  unsigned char UB7(void) {
    return HEXAGON_V64_GET_UB7(data);
  };

  // NOTE: All set methods return a HEXAGON_Vect64C type

  // Set doubleword method
  HEXAGON_Vect64C D(long long d) {
    return HEXAGON_Vect64C(HEXAGON_V64_PUT_D(data, d));
  };

  // Set word methods
  HEXAGON_Vect64C W0(int w) {
    return HEXAGON_Vect64C(HEXAGON_V64_PUT_W0(data, w));
  };
  HEXAGON_Vect64C W1(int w) {
    return HEXAGON_Vect64C(HEXAGON_V64_PUT_W1(data, w));
  };

  // Set half word methods
````
- **L745 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L745 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L746 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB6(void) {`.
  **L746 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB6(void) {`。
- **L747 EN**: Returns from the current function with `HEXAGON_V64_GET_UB6(data)`.
  **L747 CN**: 以 `HEXAGON_V64_GET_UB6(data)` 从当前函数返回。
- **L748 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L748 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L749 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB7(void) {`.
  **L749 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB7(void) {`。
- **L750 EN**: Returns from the current function with `HEXAGON_V64_GET_UB7(data)`.
  **L750 CN**: 以 `HEXAGON_V64_GET_UB7(data)` 从当前函数返回。
- **L751 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L751 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L752 EN**: Blank line separating nearby declarations or logic blocks.
  **L752 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L753 EN**: Comment highlights an implementation note: `NOTE: All set methods return a HEXAGON_Vect64C type`.
  **L753 CN**: 注释强调一条实现说明：`NOTE: All set methods return a HEXAGON_Vect64C type`。
- **L754 EN**: Blank line separating nearby declarations or logic blocks.
  **L754 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L755 EN**: Comment explains nearby logic, constraints, or intent: `Set doubleword method`.
  **L755 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set doubleword method`。
- **L756 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect64C D(long long d) {`.
  **L756 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect64C D(long long d) {`。
- **L757 EN**: Returns from the current function with `HEXAGON_Vect64C(HEXAGON_V64_PUT_D(data, d))`.
  **L757 CN**: 以 `HEXAGON_Vect64C(HEXAGON_V64_PUT_D(data, d))` 从当前函数返回。
- **L758 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L758 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L759 EN**: Blank line separating nearby declarations or logic blocks.
  **L759 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L760 EN**: Comment explains nearby logic, constraints, or intent: `Set word methods`.
  **L760 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set word methods`。
- **L761 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect64C W0(int w) {`.
  **L761 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect64C W0(int w) {`。
- **L762 EN**: Returns from the current function with `HEXAGON_Vect64C(HEXAGON_V64_PUT_W0(data, w))`.
  **L762 CN**: 以 `HEXAGON_Vect64C(HEXAGON_V64_PUT_W0(data, w))` 从当前函数返回。
- **L763 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L763 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L764 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect64C W1(int w) {`.
  **L764 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect64C W1(int w) {`。
- **L765 EN**: Returns from the current function with `HEXAGON_Vect64C(HEXAGON_V64_PUT_W1(data, w))`.
  **L765 CN**: 以 `HEXAGON_Vect64C(HEXAGON_V64_PUT_W1(data, w))` 从当前函数返回。
- **L766 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L766 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L767 EN**: Blank line separating nearby declarations or logic blocks.
  **L767 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L768 EN**: Comment explains nearby logic, constraints, or intent: `Set half word methods`.
  **L768 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set half word methods`。

### Lines 769-792

````c
  HEXAGON_Vect64C H0(short h) {
    return HEXAGON_Vect64C(HEXAGON_V64_PUT_H0(data, h));
  };
  HEXAGON_Vect64C H1(short h) {
    return HEXAGON_Vect64C(HEXAGON_V64_PUT_H1(data, h));
  };
  HEXAGON_Vect64C H2(short h) {
    return HEXAGON_Vect64C(HEXAGON_V64_PUT_H2(data, h));
  };
  HEXAGON_Vect64C H3(short h) {
    return HEXAGON_Vect64C(HEXAGON_V64_PUT_H3(data, h));
  };

  // Set byte methods
  HEXAGON_Vect64C b0(signed char b) {
    return HEXAGON_Vect64C(HEXAGON_V64_PUT_B0(data, b));
  };
  HEXAGON_Vect64C B0(signed char b) { return b0(b); };
  HEXAGON_Vect64C B1(signed char b) {
    return HEXAGON_Vect64C(HEXAGON_V64_PUT_B1(data, b));
  };
  HEXAGON_Vect64C B2(signed char b) {
    return HEXAGON_Vect64C(HEXAGON_V64_PUT_B2(data, b));
  };
````
- **L769 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect64C H0(short h) {`.
  **L769 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect64C H0(short h) {`。
- **L770 EN**: Returns from the current function with `HEXAGON_Vect64C(HEXAGON_V64_PUT_H0(data, h))`.
  **L770 CN**: 以 `HEXAGON_Vect64C(HEXAGON_V64_PUT_H0(data, h))` 从当前函数返回。
- **L771 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L771 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L772 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect64C H1(short h) {`.
  **L772 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect64C H1(short h) {`。
- **L773 EN**: Returns from the current function with `HEXAGON_Vect64C(HEXAGON_V64_PUT_H1(data, h))`.
  **L773 CN**: 以 `HEXAGON_Vect64C(HEXAGON_V64_PUT_H1(data, h))` 从当前函数返回。
- **L774 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L774 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L775 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect64C H2(short h) {`.
  **L775 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect64C H2(short h) {`。
- **L776 EN**: Returns from the current function with `HEXAGON_Vect64C(HEXAGON_V64_PUT_H2(data, h))`.
  **L776 CN**: 以 `HEXAGON_Vect64C(HEXAGON_V64_PUT_H2(data, h))` 从当前函数返回。
- **L777 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L777 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L778 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect64C H3(short h) {`.
  **L778 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect64C H3(short h) {`。
- **L779 EN**: Returns from the current function with `HEXAGON_Vect64C(HEXAGON_V64_PUT_H3(data, h))`.
  **L779 CN**: 以 `HEXAGON_Vect64C(HEXAGON_V64_PUT_H3(data, h))` 从当前函数返回。
- **L780 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L780 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L781 EN**: Blank line separating nearby declarations or logic blocks.
  **L781 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L782 EN**: Comment explains nearby logic, constraints, or intent: `Set byte methods`.
  **L782 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set byte methods`。
- **L783 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect64C b0(signed char b) {`.
  **L783 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect64C b0(signed char b) {`。
- **L784 EN**: Returns from the current function with `HEXAGON_Vect64C(HEXAGON_V64_PUT_B0(data, b))`.
  **L784 CN**: 以 `HEXAGON_Vect64C(HEXAGON_V64_PUT_B0(data, b))` 从当前函数返回。
- **L785 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L785 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L786 EN**: Executes a call or declaration centered on `B0`.
  **L786 CN**: 执行以 `B0` 为核心的调用或声明。
- **L787 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect64C B1(signed char b) {`.
  **L787 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect64C B1(signed char b) {`。
- **L788 EN**: Returns from the current function with `HEXAGON_Vect64C(HEXAGON_V64_PUT_B1(data, b))`.
  **L788 CN**: 以 `HEXAGON_Vect64C(HEXAGON_V64_PUT_B1(data, b))` 从当前函数返回。
- **L789 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L789 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L790 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect64C B2(signed char b) {`.
  **L790 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect64C B2(signed char b) {`。
- **L791 EN**: Returns from the current function with `HEXAGON_Vect64C(HEXAGON_V64_PUT_B2(data, b))`.
  **L791 CN**: 以 `HEXAGON_Vect64C(HEXAGON_V64_PUT_B2(data, b))` 从当前函数返回。
- **L792 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L792 CN**: 结束当前声明作用域，例如结构体、枚举或类。

### Lines 793-816

````c
  HEXAGON_Vect64C B3(signed char b) {
    return HEXAGON_Vect64C(HEXAGON_V64_PUT_B3(data, b));
  };
  HEXAGON_Vect64C B4(signed char b) {
    return HEXAGON_Vect64C(HEXAGON_V64_PUT_B4(data, b));
  };
  HEXAGON_Vect64C B5(signed char b) {
    return HEXAGON_Vect64C(HEXAGON_V64_PUT_B5(data, b));
  };
  HEXAGON_Vect64C B6(signed char b) {
    return HEXAGON_Vect64C(HEXAGON_V64_PUT_B6(data, b));
  };
  HEXAGON_Vect64C B7(signed char b) {
    return HEXAGON_Vect64C(HEXAGON_V64_PUT_B7(data, b));
  };

private:
  long long data;
};

#endif /* __cplusplus */

/* 32 Bit Vectors */

````
- **L793 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect64C B3(signed char b) {`.
  **L793 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect64C B3(signed char b) {`。
- **L794 EN**: Returns from the current function with `HEXAGON_Vect64C(HEXAGON_V64_PUT_B3(data, b))`.
  **L794 CN**: 以 `HEXAGON_Vect64C(HEXAGON_V64_PUT_B3(data, b))` 从当前函数返回。
- **L795 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L795 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L796 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect64C B4(signed char b) {`.
  **L796 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect64C B4(signed char b) {`。
- **L797 EN**: Returns from the current function with `HEXAGON_Vect64C(HEXAGON_V64_PUT_B4(data, b))`.
  **L797 CN**: 以 `HEXAGON_Vect64C(HEXAGON_V64_PUT_B4(data, b))` 从当前函数返回。
- **L798 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L798 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L799 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect64C B5(signed char b) {`.
  **L799 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect64C B5(signed char b) {`。
- **L800 EN**: Returns from the current function with `HEXAGON_Vect64C(HEXAGON_V64_PUT_B5(data, b))`.
  **L800 CN**: 以 `HEXAGON_Vect64C(HEXAGON_V64_PUT_B5(data, b))` 从当前函数返回。
- **L801 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L801 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L802 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect64C B6(signed char b) {`.
  **L802 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect64C B6(signed char b) {`。
- **L803 EN**: Returns from the current function with `HEXAGON_Vect64C(HEXAGON_V64_PUT_B6(data, b))`.
  **L803 CN**: 以 `HEXAGON_Vect64C(HEXAGON_V64_PUT_B6(data, b))` 从当前函数返回。
- **L804 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L804 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L805 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect64C B7(signed char b) {`.
  **L805 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect64C B7(signed char b) {`。
- **L806 EN**: Returns from the current function with `HEXAGON_Vect64C(HEXAGON_V64_PUT_B7(data, b))`.
  **L806 CN**: 以 `HEXAGON_Vect64C(HEXAGON_V64_PUT_B7(data, b))` 从当前函数返回。
- **L807 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L807 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L808 EN**: Blank line separating nearby declarations or logic blocks.
  **L808 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L809 EN**: Sets the access level for following class members to `private`.
  **L809 CN**: 将后续类成员的访问级别设为 `private`。
- **L810 EN**: Adds a standalone statement or declaration: `long long data;`.
  **L810 CN**: 添加一条独立语句或声明：`long long data;`。
- **L811 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L811 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L812 EN**: Blank line separating nearby declarations or logic blocks.
  **L812 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L813 EN**: Closes the current preprocessor conditional block.
  **L813 CN**: 结束当前预处理条件块。
- **L814 EN**: Blank line separating nearby declarations or logic blocks.
  **L814 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L815 EN**: Comment explains nearby logic, constraints, or intent: `32 Bit Vectors`.
  **L815 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 Bit Vectors`。
- **L816 EN**: Blank line separating nearby declarations or logic blocks.
  **L816 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 817-840

````c
typedef int HEXAGON_Vect32;

/* Extract word macros */

#define HEXAGON_V32_GET_W(v) (v)
#define HEXAGON_V32_GET_UW(v) ((unsigned int)(v))

/* Extract half word macros */

#define HEXAGON_V32_GET_H0(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      short h[2];                                                              \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.h[0];                                                \
  })
#define HEXAGON_V32_GET_H1(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      short h[2];                                                              \
    } _HEXAGON_V32_internal_union;                                                   \
````
- **L817 EN**: Introduces an alias or helper declaration: `typedef int HEXAGON_Vect32;`.
  **L817 CN**: 引入一条别名或辅助声明：`typedef int HEXAGON_Vect32;`。
- **L818 EN**: Blank line separating nearby declarations or logic blocks.
  **L818 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L819 EN**: Comment explains nearby logic, constraints, or intent: `Extract word macros`.
  **L819 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract word macros`。
- **L820 EN**: Blank line separating nearby declarations or logic blocks.
  **L820 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L821 EN**: Defines macro `HEXAGON_V32_GET_W(v)` for conditional compilation, shorthand, or API generation.
  **L821 CN**: 定义宏 `HEXAGON_V32_GET_W(v)`，用于条件编译、简写或 API 生成。
- **L822 EN**: Defines macro `HEXAGON_V32_GET_UW(v)` for conditional compilation, shorthand, or API generation.
  **L822 CN**: 定义宏 `HEXAGON_V32_GET_UW(v)`，用于条件编译、简写或 API 生成。
- **L823 EN**: Blank line separating nearby declarations or logic blocks.
  **L823 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L824 EN**: Comment explains nearby logic, constraints, or intent: `Extract half word macros`.
  **L824 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract half word macros`。
- **L825 EN**: Blank line separating nearby declarations or logic blocks.
  **L825 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L826 EN**: Defines macro `HEXAGON_V32_GET_H0(v)` for conditional compilation, shorthand, or API generation.
  **L826 CN**: 定义宏 `HEXAGON_V32_GET_H0(v)`，用于条件编译、简写或 API 生成。
- **L827 EN**: Continues logic associated with callable symbol `__extension__`.
  **L827 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L828 EN**: Declares union `union`.
  **L828 CN**: 声明 union `union`。
- **L829 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L829 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L830 EN**: Continues the surrounding expression or declaration: `short h[2];                                                              \`.
  **L830 CN**: 继续构造周围的表达式或声明：`short h[2];                                                              \`。
- **L831 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L831 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L832 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L832 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L833 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.h[0];                                                \`.
  **L833 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.h[0];                                                \`。
- **L834 EN**: Continues the surrounding expression or declaration: `})`.
  **L834 CN**: 继续构造周围的表达式或声明：`})`。
- **L835 EN**: Defines macro `HEXAGON_V32_GET_H1(v)` for conditional compilation, shorthand, or API generation.
  **L835 CN**: 定义宏 `HEXAGON_V32_GET_H1(v)`，用于条件编译、简写或 API 生成。
- **L836 EN**: Continues logic associated with callable symbol `__extension__`.
  **L836 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L837 EN**: Declares union `union`.
  **L837 CN**: 声明 union `union`。
- **L838 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L838 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L839 EN**: Continues the surrounding expression or declaration: `short h[2];                                                              \`.
  **L839 CN**: 继续构造周围的表达式或声明：`short h[2];                                                              \`。
- **L840 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L840 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。

### Lines 841-864

````c
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.h[1];                                                \
  })
#define HEXAGON_V32_GET_UH0(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      unsigned short uh[2];                                                    \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.uh[0];                                               \
  })
#define HEXAGON_V32_GET_UH1(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      unsigned short uh[2];                                                    \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.uh[1];                                               \
  })

/* Extract byte macros */

````
- **L841 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L841 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L842 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.h[1];                                                \`.
  **L842 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.h[1];                                                \`。
- **L843 EN**: Continues the surrounding expression or declaration: `})`.
  **L843 CN**: 继续构造周围的表达式或声明：`})`。
- **L844 EN**: Defines macro `HEXAGON_V32_GET_UH0(v)` for conditional compilation, shorthand, or API generation.
  **L844 CN**: 定义宏 `HEXAGON_V32_GET_UH0(v)`，用于条件编译、简写或 API 生成。
- **L845 EN**: Continues logic associated with callable symbol `__extension__`.
  **L845 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L846 EN**: Declares union `union`.
  **L846 CN**: 声明 union `union`。
- **L847 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L847 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L848 EN**: Continues the surrounding expression or declaration: `unsigned short uh[2];                                                    \`.
  **L848 CN**: 继续构造周围的表达式或声明：`unsigned short uh[2];                                                    \`。
- **L849 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L849 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L850 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L850 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L851 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.uh[0];                                               \`.
  **L851 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.uh[0];                                               \`。
- **L852 EN**: Continues the surrounding expression or declaration: `})`.
  **L852 CN**: 继续构造周围的表达式或声明：`})`。
- **L853 EN**: Defines macro `HEXAGON_V32_GET_UH1(v)` for conditional compilation, shorthand, or API generation.
  **L853 CN**: 定义宏 `HEXAGON_V32_GET_UH1(v)`，用于条件编译、简写或 API 生成。
- **L854 EN**: Continues logic associated with callable symbol `__extension__`.
  **L854 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L855 EN**: Declares union `union`.
  **L855 CN**: 声明 union `union`。
- **L856 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L856 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L857 EN**: Continues the surrounding expression or declaration: `unsigned short uh[2];                                                    \`.
  **L857 CN**: 继续构造周围的表达式或声明：`unsigned short uh[2];                                                    \`。
- **L858 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L858 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L859 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L859 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L860 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.uh[1];                                               \`.
  **L860 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.uh[1];                                               \`。
- **L861 EN**: Continues the surrounding expression or declaration: `})`.
  **L861 CN**: 继续构造周围的表达式或声明：`})`。
- **L862 EN**: Blank line separating nearby declarations or logic blocks.
  **L862 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L863 EN**: Comment explains nearby logic, constraints, or intent: `Extract byte macros`.
  **L863 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract byte macros`。
- **L864 EN**: Blank line separating nearby declarations or logic blocks.
  **L864 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 865-888

````c
#define HEXAGON_V32_GET_B0(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      signed char b[4];                                                        \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.b[0];                                                \
  })
#define HEXAGON_V32_GET_B1(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      signed char b[4];                                                        \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.b[1];                                                \
  })
#define HEXAGON_V32_GET_B2(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      signed char b[4];                                                        \
    } _HEXAGON_V32_internal_union;                                                   \
````
- **L865 EN**: Defines macro `HEXAGON_V32_GET_B0(v)` for conditional compilation, shorthand, or API generation.
  **L865 CN**: 定义宏 `HEXAGON_V32_GET_B0(v)`，用于条件编译、简写或 API 生成。
- **L866 EN**: Continues logic associated with callable symbol `__extension__`.
  **L866 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L867 EN**: Declares union `union`.
  **L867 CN**: 声明 union `union`。
- **L868 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L868 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L869 EN**: Continues the surrounding expression or declaration: `signed char b[4];                                                        \`.
  **L869 CN**: 继续构造周围的表达式或声明：`signed char b[4];                                                        \`。
- **L870 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L870 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L871 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L871 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L872 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.b[0];                                                \`.
  **L872 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.b[0];                                                \`。
- **L873 EN**: Continues the surrounding expression or declaration: `})`.
  **L873 CN**: 继续构造周围的表达式或声明：`})`。
- **L874 EN**: Defines macro `HEXAGON_V32_GET_B1(v)` for conditional compilation, shorthand, or API generation.
  **L874 CN**: 定义宏 `HEXAGON_V32_GET_B1(v)`，用于条件编译、简写或 API 生成。
- **L875 EN**: Continues logic associated with callable symbol `__extension__`.
  **L875 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L876 EN**: Declares union `union`.
  **L876 CN**: 声明 union `union`。
- **L877 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L877 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L878 EN**: Continues the surrounding expression or declaration: `signed char b[4];                                                        \`.
  **L878 CN**: 继续构造周围的表达式或声明：`signed char b[4];                                                        \`。
- **L879 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L879 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L880 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L880 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L881 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.b[1];                                                \`.
  **L881 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.b[1];                                                \`。
- **L882 EN**: Continues the surrounding expression or declaration: `})`.
  **L882 CN**: 继续构造周围的表达式或声明：`})`。
- **L883 EN**: Defines macro `HEXAGON_V32_GET_B2(v)` for conditional compilation, shorthand, or API generation.
  **L883 CN**: 定义宏 `HEXAGON_V32_GET_B2(v)`，用于条件编译、简写或 API 生成。
- **L884 EN**: Continues logic associated with callable symbol `__extension__`.
  **L884 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L885 EN**: Declares union `union`.
  **L885 CN**: 声明 union `union`。
- **L886 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L886 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L887 EN**: Continues the surrounding expression or declaration: `signed char b[4];                                                        \`.
  **L887 CN**: 继续构造周围的表达式或声明：`signed char b[4];                                                        \`。
- **L888 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L888 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。

### Lines 889-912

````c
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.b[2];                                                \
  })
#define HEXAGON_V32_GET_B3(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      signed char b[4];                                                        \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.b[3];                                                \
  })
#define HEXAGON_V32_GET_UB0(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      unsigned char ub[4];                                                     \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.ub[0];                                               \
  })
#define HEXAGON_V32_GET_UB1(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
````
- **L889 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L889 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L890 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.b[2];                                                \`.
  **L890 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.b[2];                                                \`。
- **L891 EN**: Continues the surrounding expression or declaration: `})`.
  **L891 CN**: 继续构造周围的表达式或声明：`})`。
- **L892 EN**: Defines macro `HEXAGON_V32_GET_B3(v)` for conditional compilation, shorthand, or API generation.
  **L892 CN**: 定义宏 `HEXAGON_V32_GET_B3(v)`，用于条件编译、简写或 API 生成。
- **L893 EN**: Continues logic associated with callable symbol `__extension__`.
  **L893 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L894 EN**: Declares union `union`.
  **L894 CN**: 声明 union `union`。
- **L895 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L895 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L896 EN**: Continues the surrounding expression or declaration: `signed char b[4];                                                        \`.
  **L896 CN**: 继续构造周围的表达式或声明：`signed char b[4];                                                        \`。
- **L897 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L897 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L898 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L898 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L899 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.b[3];                                                \`.
  **L899 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.b[3];                                                \`。
- **L900 EN**: Continues the surrounding expression or declaration: `})`.
  **L900 CN**: 继续构造周围的表达式或声明：`})`。
- **L901 EN**: Defines macro `HEXAGON_V32_GET_UB0(v)` for conditional compilation, shorthand, or API generation.
  **L901 CN**: 定义宏 `HEXAGON_V32_GET_UB0(v)`，用于条件编译、简写或 API 生成。
- **L902 EN**: Continues logic associated with callable symbol `__extension__`.
  **L902 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L903 EN**: Declares union `union`.
  **L903 CN**: 声明 union `union`。
- **L904 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L904 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L905 EN**: Continues the surrounding expression or declaration: `unsigned char ub[4];                                                     \`.
  **L905 CN**: 继续构造周围的表达式或声明：`unsigned char ub[4];                                                     \`。
- **L906 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L906 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L907 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L907 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L908 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.ub[0];                                               \`.
  **L908 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.ub[0];                                               \`。
- **L909 EN**: Continues the surrounding expression or declaration: `})`.
  **L909 CN**: 继续构造周围的表达式或声明：`})`。
- **L910 EN**: Defines macro `HEXAGON_V32_GET_UB1(v)` for conditional compilation, shorthand, or API generation.
  **L910 CN**: 定义宏 `HEXAGON_V32_GET_UB1(v)`，用于条件编译、简写或 API 生成。
- **L911 EN**: Continues logic associated with callable symbol `__extension__`.
  **L911 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L912 EN**: Declares union `union`.
  **L912 CN**: 声明 union `union`。

### Lines 913-936

````c
      int w;                                                                   \
      unsigned char ub[4];                                                     \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.ub[1];                                               \
  })
#define HEXAGON_V32_GET_UB2(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      unsigned char ub[4];                                                     \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.ub[2];                                               \
  })
#define HEXAGON_V32_GET_UB3(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      unsigned char ub[4];                                                     \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.ub[3];                                               \
  })
````
- **L913 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L913 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L914 EN**: Continues the surrounding expression or declaration: `unsigned char ub[4];                                                     \`.
  **L914 CN**: 继续构造周围的表达式或声明：`unsigned char ub[4];                                                     \`。
- **L915 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L915 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L916 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L916 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L917 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.ub[1];                                               \`.
  **L917 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.ub[1];                                               \`。
- **L918 EN**: Continues the surrounding expression or declaration: `})`.
  **L918 CN**: 继续构造周围的表达式或声明：`})`。
- **L919 EN**: Defines macro `HEXAGON_V32_GET_UB2(v)` for conditional compilation, shorthand, or API generation.
  **L919 CN**: 定义宏 `HEXAGON_V32_GET_UB2(v)`，用于条件编译、简写或 API 生成。
- **L920 EN**: Continues logic associated with callable symbol `__extension__`.
  **L920 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L921 EN**: Declares union `union`.
  **L921 CN**: 声明 union `union`。
- **L922 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L922 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L923 EN**: Continues the surrounding expression or declaration: `unsigned char ub[4];                                                     \`.
  **L923 CN**: 继续构造周围的表达式或声明：`unsigned char ub[4];                                                     \`。
- **L924 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L924 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L925 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L925 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L926 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.ub[2];                                               \`.
  **L926 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.ub[2];                                               \`。
- **L927 EN**: Continues the surrounding expression or declaration: `})`.
  **L927 CN**: 继续构造周围的表达式或声明：`})`。
- **L928 EN**: Defines macro `HEXAGON_V32_GET_UB3(v)` for conditional compilation, shorthand, or API generation.
  **L928 CN**: 定义宏 `HEXAGON_V32_GET_UB3(v)`，用于条件编译、简写或 API 生成。
- **L929 EN**: Continues logic associated with callable symbol `__extension__`.
  **L929 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L930 EN**: Declares union `union`.
  **L930 CN**: 声明 union `union`。
- **L931 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L931 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L932 EN**: Continues the surrounding expression or declaration: `unsigned char ub[4];                                                     \`.
  **L932 CN**: 继续构造周围的表达式或声明：`unsigned char ub[4];                                                     \`。
- **L933 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L933 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L934 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L934 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L935 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.ub[3];                                               \`.
  **L935 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.ub[3];                                               \`。
- **L936 EN**: Continues the surrounding expression or declaration: `})`.
  **L936 CN**: 继续构造周围的表达式或声明：`})`。

### Lines 937-960

````c

/* NOTE: All set macros return a HEXAGON_Vect32 type */

/* Set word macro */

#define HEXAGON_V32_PUT_W(v, new) (new)

/* Set half word macros */

#ifdef __hexagon__

#define HEXAGON_V32_PUT_H0(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      short h[2];                                                              \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.h[0] = (new);                                        \
    _HEXAGON_V32_internal_union.w;                                                   \
  })
#define HEXAGON_V32_PUT_H1(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
````
- **L937 EN**: Blank line separating nearby declarations or logic blocks.
  **L937 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L938 EN**: Comment highlights an implementation note: `NOTE: All set macros return a HEXAGON_Vect32 type`.
  **L938 CN**: 注释强调一条实现说明：`NOTE: All set macros return a HEXAGON_Vect32 type`。
- **L939 EN**: Blank line separating nearby declarations or logic blocks.
  **L939 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L940 EN**: Comment explains nearby logic, constraints, or intent: `Set word macro`.
  **L940 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set word macro`。
- **L941 EN**: Blank line separating nearby declarations or logic blocks.
  **L941 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L942 EN**: Defines macro `HEXAGON_V32_PUT_W(v, new)` for conditional compilation, shorthand, or API generation.
  **L942 CN**: 定义宏 `HEXAGON_V32_PUT_W(v, new)`，用于条件编译、简写或 API 生成。
- **L943 EN**: Blank line separating nearby declarations or logic blocks.
  **L943 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L944 EN**: Comment explains nearby logic, constraints, or intent: `Set half word macros`.
  **L944 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set half word macros`。
- **L945 EN**: Blank line separating nearby declarations or logic blocks.
  **L945 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L946 EN**: Starts a preprocessor conditional block: `#ifdef __hexagon__`.
  **L946 CN**: 开始一个预处理条件块：`#ifdef __hexagon__`。
- **L947 EN**: Blank line separating nearby declarations or logic blocks.
  **L947 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L948 EN**: Defines macro `HEXAGON_V32_PUT_H0(v, new)` for conditional compilation, shorthand, or API generation.
  **L948 CN**: 定义宏 `HEXAGON_V32_PUT_H0(v, new)`，用于条件编译、简写或 API 生成。
- **L949 EN**: Continues logic associated with callable symbol `__extension__`.
  **L949 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L950 EN**: Declares union `union`.
  **L950 CN**: 声明 union `union`。
- **L951 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L951 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L952 EN**: Continues the surrounding expression or declaration: `short h[2];                                                              \`.
  **L952 CN**: 继续构造周围的表达式或声明：`short h[2];                                                              \`。
- **L953 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L953 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L954 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L954 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L955 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.h[0] = (new);                                        \`.
  **L955 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.h[0] = (new);                                        \`。
- **L956 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w;                                                   \`.
  **L956 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w;                                                   \`。
- **L957 EN**: Continues the surrounding expression or declaration: `})`.
  **L957 CN**: 继续构造周围的表达式或声明：`})`。
- **L958 EN**: Defines macro `HEXAGON_V32_PUT_H1(v, new)` for conditional compilation, shorthand, or API generation.
  **L958 CN**: 定义宏 `HEXAGON_V32_PUT_H1(v, new)`，用于条件编译、简写或 API 生成。
- **L959 EN**: Continues logic associated with callable symbol `__extension__`.
  **L959 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L960 EN**: Declares union `union`.
  **L960 CN**: 声明 union `union`。

### Lines 961-984

````c
      int w;                                                                   \
      short h[2];                                                              \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.h[1] = (new);                                        \
    _HEXAGON_V32_internal_union.w;                                                   \
  })

#else /* !__hexagon__ */

#define HEXAGON_V32_PUT_H0(v, new)                                                   \
  (((v) & 0xffff0000) | ((HEXAGON_Vect32)((unsigned short)(new))))
#define HEXAGON_V32_PUT_H1(v, new) (((v) & 0x0000ffff) | (((HEXAGON_Vect32)(new)) << 16))

#endif /* !__hexagon__ */

/* Set byte macros */

#ifdef __hexagon__

#define HEXAGON_V32_PUT_B0(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
````
- **L961 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L961 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L962 EN**: Continues the surrounding expression or declaration: `short h[2];                                                              \`.
  **L962 CN**: 继续构造周围的表达式或声明：`short h[2];                                                              \`。
- **L963 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L963 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L964 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L964 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L965 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.h[1] = (new);                                        \`.
  **L965 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.h[1] = (new);                                        \`。
- **L966 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w;                                                   \`.
  **L966 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w;                                                   \`。
- **L967 EN**: Continues the surrounding expression or declaration: `})`.
  **L967 CN**: 继续构造周围的表达式或声明：`})`。
- **L968 EN**: Blank line separating nearby declarations or logic blocks.
  **L968 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L969 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L969 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L970 EN**: Blank line separating nearby declarations or logic blocks.
  **L970 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L971 EN**: Defines macro `HEXAGON_V32_PUT_H0(v, new)` for conditional compilation, shorthand, or API generation.
  **L971 CN**: 定义宏 `HEXAGON_V32_PUT_H0(v, new)`，用于条件编译、简写或 API 生成。
- **L972 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffff0000) | ((HEXAGON_Vect32)((unsigned short)(new))))`.
  **L972 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffff0000) | ((HEXAGON_Vect32)((unsigned short)(new))))`。
- **L973 EN**: Defines macro `HEXAGON_V32_PUT_H1(v, new)` for conditional compilation, shorthand, or API generation.
  **L973 CN**: 定义宏 `HEXAGON_V32_PUT_H1(v, new)`，用于条件编译、简写或 API 生成。
- **L974 EN**: Blank line separating nearby declarations or logic blocks.
  **L974 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L975 EN**: Closes the current preprocessor conditional block.
  **L975 CN**: 结束当前预处理条件块。
- **L976 EN**: Blank line separating nearby declarations or logic blocks.
  **L976 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L977 EN**: Comment explains nearby logic, constraints, or intent: `Set byte macros`.
  **L977 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set byte macros`。
- **L978 EN**: Blank line separating nearby declarations or logic blocks.
  **L978 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L979 EN**: Starts a preprocessor conditional block: `#ifdef __hexagon__`.
  **L979 CN**: 开始一个预处理条件块：`#ifdef __hexagon__`。
- **L980 EN**: Blank line separating nearby declarations or logic blocks.
  **L980 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L981 EN**: Defines macro `HEXAGON_V32_PUT_B0(v, new)` for conditional compilation, shorthand, or API generation.
  **L981 CN**: 定义宏 `HEXAGON_V32_PUT_B0(v, new)`，用于条件编译、简写或 API 生成。
- **L982 EN**: Continues logic associated with callable symbol `__extension__`.
  **L982 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L983 EN**: Declares union `union`.
  **L983 CN**: 声明 union `union`。
- **L984 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L984 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。

### Lines 985-1008

````c
      char b[4];                                                               \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.b[0] = (new);                                        \
    _HEXAGON_V32_internal_union.w;                                                   \
  })
#define HEXAGON_V32_PUT_B1(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      char b[4];                                                               \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.b[1] = (new);                                        \
    _HEXAGON_V32_internal_union.w;                                                   \
  })
#define HEXAGON_V32_PUT_B2(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      char b[4];                                                               \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.b[2] = (new);                                        \
````
- **L985 EN**: Continues the surrounding expression or declaration: `char b[4];                                                               \`.
  **L985 CN**: 继续构造周围的表达式或声明：`char b[4];                                                               \`。
- **L986 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L986 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L987 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L987 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L988 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.b[0] = (new);                                        \`.
  **L988 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.b[0] = (new);                                        \`。
- **L989 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w;                                                   \`.
  **L989 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w;                                                   \`。
- **L990 EN**: Continues the surrounding expression or declaration: `})`.
  **L990 CN**: 继续构造周围的表达式或声明：`})`。
- **L991 EN**: Defines macro `HEXAGON_V32_PUT_B1(v, new)` for conditional compilation, shorthand, or API generation.
  **L991 CN**: 定义宏 `HEXAGON_V32_PUT_B1(v, new)`，用于条件编译、简写或 API 生成。
- **L992 EN**: Continues logic associated with callable symbol `__extension__`.
  **L992 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L993 EN**: Declares union `union`.
  **L993 CN**: 声明 union `union`。
- **L994 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L994 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L995 EN**: Continues the surrounding expression or declaration: `char b[4];                                                               \`.
  **L995 CN**: 继续构造周围的表达式或声明：`char b[4];                                                               \`。
- **L996 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L996 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L997 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L997 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L998 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.b[1] = (new);                                        \`.
  **L998 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.b[1] = (new);                                        \`。
- **L999 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w;                                                   \`.
  **L999 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w;                                                   \`。
- **L1000 EN**: Continues the surrounding expression or declaration: `})`.
  **L1000 CN**: 继续构造周围的表达式或声明：`})`。
- **L1001 EN**: Defines macro `HEXAGON_V32_PUT_B2(v, new)` for conditional compilation, shorthand, or API generation.
  **L1001 CN**: 定义宏 `HEXAGON_V32_PUT_B2(v, new)`，用于条件编译、简写或 API 生成。
- **L1002 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1002 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1003 EN**: Declares union `union`.
  **L1003 CN**: 声明 union `union`。
- **L1004 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L1004 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L1005 EN**: Continues the surrounding expression or declaration: `char b[4];                                                               \`.
  **L1005 CN**: 继续构造周围的表达式或声明：`char b[4];                                                               \`。
- **L1006 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L1006 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L1007 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L1007 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L1008 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.b[2] = (new);                                        \`.
  **L1008 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.b[2] = (new);                                        \`。

### Lines 1009-1032

````c
    _HEXAGON_V32_internal_union.w;                                                   \
  })
#define HEXAGON_V32_PUT_B3(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      char b[4];                                                               \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.w = (v);                                             \
    _HEXAGON_V32_internal_union.b[3] = (new);                                        \
    _HEXAGON_V32_internal_union.w;                                                   \
  })

#else /* !__hexagon__ */

#define HEXAGON_V32_PUT_B0(v, new)                                                   \
  (((v) & 0xffffff00) | ((HEXAGON_Vect32)((unsigned char)(new))))
#define HEXAGON_V32_PUT_B1(v, new)                                                   \
  (((v) & 0xffff00ff) | (((HEXAGON_Vect32)((unsigned char)(new))) << 8))
#define HEXAGON_V32_PUT_B2(v, new)                                                   \
  (((v) & 0xff00ffff) | (((HEXAGON_Vect32)((unsigned char)(new))) << 16))
#define HEXAGON_V32_PUT_B3(v, new) (((v) & 0x00ffffff) | (((HEXAGON_Vect32)(new)) << 24))

#endif /* !__hexagon__ */
````
- **L1009 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w;                                                   \`.
  **L1009 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w;                                                   \`。
- **L1010 EN**: Continues the surrounding expression or declaration: `})`.
  **L1010 CN**: 继续构造周围的表达式或声明：`})`。
- **L1011 EN**: Defines macro `HEXAGON_V32_PUT_B3(v, new)` for conditional compilation, shorthand, or API generation.
  **L1011 CN**: 定义宏 `HEXAGON_V32_PUT_B3(v, new)`，用于条件编译、简写或 API 生成。
- **L1012 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1012 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1013 EN**: Declares union `union`.
  **L1013 CN**: 声明 union `union`。
- **L1014 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L1014 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L1015 EN**: Continues the surrounding expression or declaration: `char b[4];                                                               \`.
  **L1015 CN**: 继续构造周围的表达式或声明：`char b[4];                                                               \`。
- **L1016 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L1016 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L1017 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w = (v);                                             \`.
  **L1017 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w = (v);                                             \`。
- **L1018 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.b[3] = (new);                                        \`.
  **L1018 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.b[3] = (new);                                        \`。
- **L1019 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.w;                                                   \`.
  **L1019 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.w;                                                   \`。
- **L1020 EN**: Continues the surrounding expression or declaration: `})`.
  **L1020 CN**: 继续构造周围的表达式或声明：`})`。
- **L1021 EN**: Blank line separating nearby declarations or logic blocks.
  **L1021 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1022 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1022 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1023 EN**: Blank line separating nearby declarations or logic blocks.
  **L1023 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1024 EN**: Defines macro `HEXAGON_V32_PUT_B0(v, new)` for conditional compilation, shorthand, or API generation.
  **L1024 CN**: 定义宏 `HEXAGON_V32_PUT_B0(v, new)`，用于条件编译、简写或 API 生成。
- **L1025 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffff00) | ((HEXAGON_Vect32)((unsigned char)(new))))`.
  **L1025 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffff00) | ((HEXAGON_Vect32)((unsigned char)(new))))`。
- **L1026 EN**: Defines macro `HEXAGON_V32_PUT_B1(v, new)` for conditional compilation, shorthand, or API generation.
  **L1026 CN**: 定义宏 `HEXAGON_V32_PUT_B1(v, new)`，用于条件编译、简写或 API 生成。
- **L1027 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffff00ff) | (((HEXAGON_Vect32)((unsigned char)(new))) << 8))`.
  **L1027 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffff00ff) | (((HEXAGON_Vect32)((unsigned char)(new))) << 8))`。
- **L1028 EN**: Defines macro `HEXAGON_V32_PUT_B2(v, new)` for conditional compilation, shorthand, or API generation.
  **L1028 CN**: 定义宏 `HEXAGON_V32_PUT_B2(v, new)`，用于条件编译、简写或 API 生成。
- **L1029 EN**: Continues the surrounding expression or declaration: `(((v) & 0xff00ffff) | (((HEXAGON_Vect32)((unsigned char)(new))) << 16))`.
  **L1029 CN**: 继续构造周围的表达式或声明：`(((v) & 0xff00ffff) | (((HEXAGON_Vect32)((unsigned char)(new))) << 16))`。
- **L1030 EN**: Defines macro `HEXAGON_V32_PUT_B3(v, new)` for conditional compilation, shorthand, or API generation.
  **L1030 CN**: 定义宏 `HEXAGON_V32_PUT_B3(v, new)`，用于条件编译、简写或 API 生成。
- **L1031 EN**: Blank line separating nearby declarations or logic blocks.
  **L1031 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1032 EN**: Closes the current preprocessor conditional block.
  **L1032 CN**: 结束当前预处理条件块。

### Lines 1033-1056

````c

/* NOTE: All create macros return a HEXAGON_Vect32 type */

/* Create from a word */

#define HEXAGON_V32_CREATE_W(w) (w)

/* Create from half words */

#ifdef __hexagon__

#define HEXAGON_V32_CREATE_H(h1, h0)                                                 \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[2];                                                              \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.h[0] = (h0);                                         \
    _HEXAGON_V32_internal_union.h[1] = (h1);                                         \
    _HEXAGON_V32_internal_union.d;                                                   \
  })

#else /* !__hexagon__ */

````
- **L1033 EN**: Blank line separating nearby declarations or logic blocks.
  **L1033 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1034 EN**: Comment highlights an implementation note: `NOTE: All create macros return a HEXAGON_Vect32 type`.
  **L1034 CN**: 注释强调一条实现说明：`NOTE: All create macros return a HEXAGON_Vect32 type`。
- **L1035 EN**: Blank line separating nearby declarations or logic blocks.
  **L1035 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1036 EN**: Comment explains nearby logic, constraints, or intent: `Create from a word`.
  **L1036 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create from a word`。
- **L1037 EN**: Blank line separating nearby declarations or logic blocks.
  **L1037 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1038 EN**: Defines macro `HEXAGON_V32_CREATE_W(w)` for conditional compilation, shorthand, or API generation.
  **L1038 CN**: 定义宏 `HEXAGON_V32_CREATE_W(w)`，用于条件编译、简写或 API 生成。
- **L1039 EN**: Blank line separating nearby declarations or logic blocks.
  **L1039 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1040 EN**: Comment explains nearby logic, constraints, or intent: `Create from half words`.
  **L1040 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create from half words`。
- **L1041 EN**: Blank line separating nearby declarations or logic blocks.
  **L1041 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1042 EN**: Starts a preprocessor conditional block: `#ifdef __hexagon__`.
  **L1042 CN**: 开始一个预处理条件块：`#ifdef __hexagon__`。
- **L1043 EN**: Blank line separating nearby declarations or logic blocks.
  **L1043 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1044 EN**: Defines macro `HEXAGON_V32_CREATE_H(h1, h0)` for conditional compilation, shorthand, or API generation.
  **L1044 CN**: 定义宏 `HEXAGON_V32_CREATE_H(h1, h0)`，用于条件编译、简写或 API 生成。
- **L1045 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1045 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1046 EN**: Declares union `union`.
  **L1046 CN**: 声明 union `union`。
- **L1047 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1047 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1048 EN**: Continues the surrounding expression or declaration: `short h[2];                                                              \`.
  **L1048 CN**: 继续构造周围的表达式或声明：`short h[2];                                                              \`。
- **L1049 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L1049 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L1050 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.h[0] = (h0);                                         \`.
  **L1050 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.h[0] = (h0);                                         \`。
- **L1051 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.h[1] = (h1);                                         \`.
  **L1051 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.h[1] = (h1);                                         \`。
- **L1052 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.d;                                                   \`.
  **L1052 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.d;                                                   \`。
- **L1053 EN**: Continues the surrounding expression or declaration: `})`.
  **L1053 CN**: 继续构造周围的表达式或声明：`})`。
- **L1054 EN**: Blank line separating nearby declarations or logic blocks.
  **L1054 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1055 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1055 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1056 EN**: Blank line separating nearby declarations or logic blocks.
  **L1056 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1057-1080

````c
#define HEXAGON_V32_CREATE_H(h1, h0)                                                 \
  ((((HEXAGON_Vect32)(h1)) << 16) | ((HEXAGON_Vect32)((h0) & 0xffff)))

#endif /* !__hexagon__ */

/* Create from bytes */
#ifdef __hexagon__

#define HEXAGON_V32_CREATE_B(b3, b2, b1, b0)                                         \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[4];                                                               \
    } _HEXAGON_V32_internal_union;                                                   \
    _HEXAGON_V32_internal_union.b[0] = (b0);                                         \
    _HEXAGON_V32_internal_union.b[1] = (b1);                                         \
    _HEXAGON_V32_internal_union.b[2] = (b2);                                         \
    _HEXAGON_V32_internal_union.b[3] = (b3);                                         \
    _HEXAGON_V32_internal_union.d;                                                   \
  })

#else /* !__hexagon__ */

#define HEXAGON_V32_CREATE_B(b3, b2, b1, b0)                                         \
````
- **L1057 EN**: Defines macro `HEXAGON_V32_CREATE_H(h1, h0)` for conditional compilation, shorthand, or API generation.
  **L1057 CN**: 定义宏 `HEXAGON_V32_CREATE_H(h1, h0)`，用于条件编译、简写或 API 生成。
- **L1058 EN**: Continues the surrounding expression or declaration: `((((HEXAGON_Vect32)(h1)) << 16) | ((HEXAGON_Vect32)((h0) & 0xffff)))`.
  **L1058 CN**: 继续构造周围的表达式或声明：`((((HEXAGON_Vect32)(h1)) << 16) | ((HEXAGON_Vect32)((h0) & 0xffff)))`。
- **L1059 EN**: Blank line separating nearby declarations or logic blocks.
  **L1059 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1060 EN**: Closes the current preprocessor conditional block.
  **L1060 CN**: 结束当前预处理条件块。
- **L1061 EN**: Blank line separating nearby declarations or logic blocks.
  **L1061 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1062 EN**: Comment explains nearby logic, constraints, or intent: `Create from bytes`.
  **L1062 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create from bytes`。
- **L1063 EN**: Starts a preprocessor conditional block: `#ifdef __hexagon__`.
  **L1063 CN**: 开始一个预处理条件块：`#ifdef __hexagon__`。
- **L1064 EN**: Blank line separating nearby declarations or logic blocks.
  **L1064 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1065 EN**: Defines macro `HEXAGON_V32_CREATE_B(b3, b2, b1, b0)` for conditional compilation, shorthand, or API generation.
  **L1065 CN**: 定义宏 `HEXAGON_V32_CREATE_B(b3, b2, b1, b0)`，用于条件编译、简写或 API 生成。
- **L1066 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1066 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1067 EN**: Declares union `union`.
  **L1067 CN**: 声明 union `union`。
- **L1068 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1068 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1069 EN**: Continues the surrounding expression or declaration: `char b[4];                                                               \`.
  **L1069 CN**: 继续构造周围的表达式或声明：`char b[4];                                                               \`。
- **L1070 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_V32_internal_union;                                                   \`.
  **L1070 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_V32_internal_union;                                                   \`。
- **L1071 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.b[0] = (b0);                                         \`.
  **L1071 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.b[0] = (b0);                                         \`。
- **L1072 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.b[1] = (b1);                                         \`.
  **L1072 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.b[1] = (b1);                                         \`。
- **L1073 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.b[2] = (b2);                                         \`.
  **L1073 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.b[2] = (b2);                                         \`。
- **L1074 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.b[3] = (b3);                                         \`.
  **L1074 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.b[3] = (b3);                                         \`。
- **L1075 EN**: Continues the surrounding expression or declaration: `_HEXAGON_V32_internal_union.d;                                                   \`.
  **L1075 CN**: 继续构造周围的表达式或声明：`_HEXAGON_V32_internal_union.d;                                                   \`。
- **L1076 EN**: Continues the surrounding expression or declaration: `})`.
  **L1076 CN**: 继续构造周围的表达式或声明：`})`。
- **L1077 EN**: Blank line separating nearby declarations or logic blocks.
  **L1077 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1078 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1078 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1079 EN**: Blank line separating nearby declarations or logic blocks.
  **L1079 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1080 EN**: Defines macro `HEXAGON_V32_CREATE_B(b3, b2, b1, b0)` for conditional compilation, shorthand, or API generation.
  **L1080 CN**: 定义宏 `HEXAGON_V32_CREATE_B(b3, b2, b1, b0)`，用于条件编译、简写或 API 生成。

### Lines 1081-1104

````c
  ((((HEXAGON_Vect32)(b3)) << 24) | (((HEXAGON_Vect32)((b2) & 0xff)) << 16) |              \
   (((HEXAGON_Vect32)((b1) & 0xff)) << 8) | ((HEXAGON_Vect32)((b0) & 0xff)))

#endif /* !__hexagon__ */

#ifdef __cplusplus

class HEXAGON_Vect32C {
public:
  // Constructors
  HEXAGON_Vect32C(int w = 0) : data(w) {};
  HEXAGON_Vect32C(short h1, short h0) : data(HEXAGON_V32_CREATE_H(h1, h0)) {};
  HEXAGON_Vect32C(signed char b3, signed char b2, signed char b1, signed char b0)
      : data(HEXAGON_V32_CREATE_B(b3, b2, b1, b0)) {};
  HEXAGON_Vect32C(const HEXAGON_Vect32C &v) : data(v.data) {};

  HEXAGON_Vect32C &operator=(const HEXAGON_Vect32C &v) {
    data = v.data;
    return *this;
  };

  operator int() {
    return data;
  };
````
- **L1081 EN**: Continues the surrounding expression or declaration: `((((HEXAGON_Vect32)(b3)) << 24) | (((HEXAGON_Vect32)((b2) & 0xff)) << 16) |              \`.
  **L1081 CN**: 继续构造周围的表达式或声明：`((((HEXAGON_Vect32)(b3)) << 24) | (((HEXAGON_Vect32)((b2) & 0xff)) << 16) |              \`。
- **L1082 EN**: Continues the surrounding expression or declaration: `(((HEXAGON_Vect32)((b1) & 0xff)) << 8) | ((HEXAGON_Vect32)((b0) & 0xff)))`.
  **L1082 CN**: 继续构造周围的表达式或声明：`(((HEXAGON_Vect32)((b1) & 0xff)) << 8) | ((HEXAGON_Vect32)((b0) & 0xff)))`。
- **L1083 EN**: Blank line separating nearby declarations or logic blocks.
  **L1083 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1084 EN**: Closes the current preprocessor conditional block.
  **L1084 CN**: 结束当前预处理条件块。
- **L1085 EN**: Blank line separating nearby declarations or logic blocks.
  **L1085 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1086 EN**: Starts a preprocessor conditional block: `#ifdef __cplusplus`.
  **L1086 CN**: 开始一个预处理条件块：`#ifdef __cplusplus`。
- **L1087 EN**: Blank line separating nearby declarations or logic blocks.
  **L1087 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1088 EN**: Declares class `HEXAGON_Vect32C`.
  **L1088 CN**: 声明 class `HEXAGON_Vect32C`。
- **L1089 EN**: Sets the access level for following class members to `public`.
  **L1089 CN**: 将后续类成员的访问级别设为 `public`。
- **L1090 EN**: Comment explains nearby logic, constraints, or intent: `Constructors`.
  **L1090 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructors`。
- **L1091 EN**: Executes a call or declaration centered on `HEXAGON_Vect32C`.
  **L1091 CN**: 执行以 `HEXAGON_Vect32C` 为核心的调用或声明。
- **L1092 EN**: Executes a call or declaration centered on `HEXAGON_Vect32C`.
  **L1092 CN**: 执行以 `HEXAGON_Vect32C` 为核心的调用或声明。
- **L1093 EN**: Continues logic associated with callable symbol `HEXAGON_Vect32C`.
  **L1093 CN**: 继续与可调用符号 `HEXAGON_Vect32C` 相关的逻辑。
- **L1094 EN**: Executes a call or declaration centered on `data`.
  **L1094 CN**: 执行以 `data` 为核心的调用或声明。
- **L1095 EN**: Executes a call or declaration centered on `HEXAGON_Vect32C`.
  **L1095 CN**: 执行以 `HEXAGON_Vect32C` 为核心的调用或声明。
- **L1096 EN**: Blank line separating nearby declarations or logic blocks.
  **L1096 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1097 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect32C &operator=(const HEXAGON_Vect32C &v) {`.
  **L1097 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect32C &operator=(const HEXAGON_Vect32C &v) {`。
- **L1098 EN**: Adds a standalone statement or declaration: `data = v.data;`.
  **L1098 CN**: 添加一条独立语句或声明：`data = v.data;`。
- **L1099 EN**: Returns from the current function with `*this`.
  **L1099 CN**: 以 `*this` 从当前函数返回。
- **L1100 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1100 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1101 EN**: Blank line separating nearby declarations or logic blocks.
  **L1101 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1102 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `operator int() {`.
  **L1102 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`operator int() {`。
- **L1103 EN**: Returns from the current function with `data`.
  **L1103 CN**: 以 `data` 从当前函数返回。
- **L1104 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1104 CN**: 结束当前声明作用域，例如结构体、枚举或类。

### Lines 1105-1128

````c

  // Extract word methods
  int W(void) {
    return HEXAGON_V32_GET_W(data);
  };
  unsigned int UW(void) {
    return HEXAGON_V32_GET_UW(data);
  };

  // Extract half word methods
  short H0(void) {
    return HEXAGON_V32_GET_H0(data);
  };
  short H1(void) {
    return HEXAGON_V32_GET_H1(data);
  };
  unsigned short UH0(void) {
    return HEXAGON_V32_GET_UH0(data);
  };
  unsigned short UH1(void) {
    return HEXAGON_V32_GET_UH1(data);
  };

  // Extract byte methods
````
- **L1105 EN**: Blank line separating nearby declarations or logic blocks.
  **L1105 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1106 EN**: Comment explains nearby logic, constraints, or intent: `Extract word methods`.
  **L1106 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract word methods`。
- **L1107 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `int W(void) {`.
  **L1107 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`int W(void) {`。
- **L1108 EN**: Returns from the current function with `HEXAGON_V32_GET_W(data)`.
  **L1108 CN**: 以 `HEXAGON_V32_GET_W(data)` 从当前函数返回。
- **L1109 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1109 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1110 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned int UW(void) {`.
  **L1110 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned int UW(void) {`。
- **L1111 EN**: Returns from the current function with `HEXAGON_V32_GET_UW(data)`.
  **L1111 CN**: 以 `HEXAGON_V32_GET_UW(data)` 从当前函数返回。
- **L1112 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1112 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1113 EN**: Blank line separating nearby declarations or logic blocks.
  **L1113 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1114 EN**: Comment explains nearby logic, constraints, or intent: `Extract half word methods`.
  **L1114 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract half word methods`。
- **L1115 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `short H0(void) {`.
  **L1115 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`short H0(void) {`。
- **L1116 EN**: Returns from the current function with `HEXAGON_V32_GET_H0(data)`.
  **L1116 CN**: 以 `HEXAGON_V32_GET_H0(data)` 从当前函数返回。
- **L1117 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1117 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1118 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `short H1(void) {`.
  **L1118 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`short H1(void) {`。
- **L1119 EN**: Returns from the current function with `HEXAGON_V32_GET_H1(data)`.
  **L1119 CN**: 以 `HEXAGON_V32_GET_H1(data)` 从当前函数返回。
- **L1120 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1120 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1121 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned short UH0(void) {`.
  **L1121 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned short UH0(void) {`。
- **L1122 EN**: Returns from the current function with `HEXAGON_V32_GET_UH0(data)`.
  **L1122 CN**: 以 `HEXAGON_V32_GET_UH0(data)` 从当前函数返回。
- **L1123 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1123 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1124 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned short UH1(void) {`.
  **L1124 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned short UH1(void) {`。
- **L1125 EN**: Returns from the current function with `HEXAGON_V32_GET_UH1(data)`.
  **L1125 CN**: 以 `HEXAGON_V32_GET_UH1(data)` 从当前函数返回。
- **L1126 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1126 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1127 EN**: Blank line separating nearby declarations or logic blocks.
  **L1127 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1128 EN**: Comment explains nearby logic, constraints, or intent: `Extract byte methods`.
  **L1128 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract byte methods`。

### Lines 1129-1152

````c
  signed char b0(void) { return HEXAGON_V32_GET_B0(data); };
  signed char B0(void) { return b0(); };
  signed char B1(void) {
    return HEXAGON_V32_GET_B1(data);
  };
  signed char B2(void) {
    return HEXAGON_V32_GET_B2(data);
  };
  signed char B3(void) {
    return HEXAGON_V32_GET_B3(data);
  };
  unsigned char UB0(void) {
    return HEXAGON_V32_GET_UB0(data);
  };
  unsigned char UB1(void) {
    return HEXAGON_V32_GET_UB1(data);
  };
  unsigned char UB2(void) {
    return HEXAGON_V32_GET_UB2(data);
  };
  unsigned char UB3(void) {
    return HEXAGON_V32_GET_UB3(data);
  };

````
- **L1129 EN**: Executes a call or declaration centered on `b0`.
  **L1129 CN**: 执行以 `b0` 为核心的调用或声明。
- **L1130 EN**: Executes a call or declaration centered on `B0`.
  **L1130 CN**: 执行以 `B0` 为核心的调用或声明。
- **L1131 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B1(void) {`.
  **L1131 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B1(void) {`。
- **L1132 EN**: Returns from the current function with `HEXAGON_V32_GET_B1(data)`.
  **L1132 CN**: 以 `HEXAGON_V32_GET_B1(data)` 从当前函数返回。
- **L1133 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1133 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1134 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B2(void) {`.
  **L1134 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B2(void) {`。
- **L1135 EN**: Returns from the current function with `HEXAGON_V32_GET_B2(data)`.
  **L1135 CN**: 以 `HEXAGON_V32_GET_B2(data)` 从当前函数返回。
- **L1136 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1136 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1137 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B3(void) {`.
  **L1137 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B3(void) {`。
- **L1138 EN**: Returns from the current function with `HEXAGON_V32_GET_B3(data)`.
  **L1138 CN**: 以 `HEXAGON_V32_GET_B3(data)` 从当前函数返回。
- **L1139 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1139 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1140 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB0(void) {`.
  **L1140 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB0(void) {`。
- **L1141 EN**: Returns from the current function with `HEXAGON_V32_GET_UB0(data)`.
  **L1141 CN**: 以 `HEXAGON_V32_GET_UB0(data)` 从当前函数返回。
- **L1142 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1142 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1143 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB1(void) {`.
  **L1143 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB1(void) {`。
- **L1144 EN**: Returns from the current function with `HEXAGON_V32_GET_UB1(data)`.
  **L1144 CN**: 以 `HEXAGON_V32_GET_UB1(data)` 从当前函数返回。
- **L1145 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1145 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1146 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB2(void) {`.
  **L1146 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB2(void) {`。
- **L1147 EN**: Returns from the current function with `HEXAGON_V32_GET_UB2(data)`.
  **L1147 CN**: 以 `HEXAGON_V32_GET_UB2(data)` 从当前函数返回。
- **L1148 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1148 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1149 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB3(void) {`.
  **L1149 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB3(void) {`。
- **L1150 EN**: Returns from the current function with `HEXAGON_V32_GET_UB3(data)`.
  **L1150 CN**: 以 `HEXAGON_V32_GET_UB3(data)` 从当前函数返回。
- **L1151 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1151 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1152 EN**: Blank line separating nearby declarations or logic blocks.
  **L1152 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1153-1176

````c
  // NOTE: All set methods return a HEXAGON_Vect32C type

  // Set word method
  HEXAGON_Vect32C W(int w) {
    return HEXAGON_Vect32C(HEXAGON_V32_PUT_W(data, w));
  };

  // Set half word methods
  HEXAGON_Vect32C H0(short h) {
    return HEXAGON_Vect32C(HEXAGON_V32_PUT_H0(data, h));
  };
  HEXAGON_Vect32C H1(short h) {
    return HEXAGON_Vect32C(HEXAGON_V32_PUT_H1(data, h));
  };

  // Set byte methods
  HEXAGON_Vect32C b0(signed char b) {
    return HEXAGON_Vect32C(HEXAGON_V32_PUT_B0(data, b));
  };
  HEXAGON_Vect32C B0(signed char b) { return b0(b); };
  HEXAGON_Vect32C B1(signed char b) {
    return HEXAGON_Vect32C(HEXAGON_V32_PUT_B1(data, b));
  };
  HEXAGON_Vect32C B2(signed char b) {
````
- **L1153 EN**: Comment highlights an implementation note: `NOTE: All set methods return a HEXAGON_Vect32C type`.
  **L1153 CN**: 注释强调一条实现说明：`NOTE: All set methods return a HEXAGON_Vect32C type`。
- **L1154 EN**: Blank line separating nearby declarations or logic blocks.
  **L1154 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1155 EN**: Comment explains nearby logic, constraints, or intent: `Set word method`.
  **L1155 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set word method`。
- **L1156 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect32C W(int w) {`.
  **L1156 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect32C W(int w) {`。
- **L1157 EN**: Returns from the current function with `HEXAGON_Vect32C(HEXAGON_V32_PUT_W(data, w))`.
  **L1157 CN**: 以 `HEXAGON_Vect32C(HEXAGON_V32_PUT_W(data, w))` 从当前函数返回。
- **L1158 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1158 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1159 EN**: Blank line separating nearby declarations or logic blocks.
  **L1159 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1160 EN**: Comment explains nearby logic, constraints, or intent: `Set half word methods`.
  **L1160 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set half word methods`。
- **L1161 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect32C H0(short h) {`.
  **L1161 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect32C H0(short h) {`。
- **L1162 EN**: Returns from the current function with `HEXAGON_Vect32C(HEXAGON_V32_PUT_H0(data, h))`.
  **L1162 CN**: 以 `HEXAGON_Vect32C(HEXAGON_V32_PUT_H0(data, h))` 从当前函数返回。
- **L1163 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1163 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1164 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect32C H1(short h) {`.
  **L1164 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect32C H1(short h) {`。
- **L1165 EN**: Returns from the current function with `HEXAGON_Vect32C(HEXAGON_V32_PUT_H1(data, h))`.
  **L1165 CN**: 以 `HEXAGON_Vect32C(HEXAGON_V32_PUT_H1(data, h))` 从当前函数返回。
- **L1166 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1166 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1167 EN**: Blank line separating nearby declarations or logic blocks.
  **L1167 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1168 EN**: Comment explains nearby logic, constraints, or intent: `Set byte methods`.
  **L1168 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set byte methods`。
- **L1169 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect32C b0(signed char b) {`.
  **L1169 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect32C b0(signed char b) {`。
- **L1170 EN**: Returns from the current function with `HEXAGON_Vect32C(HEXAGON_V32_PUT_B0(data, b))`.
  **L1170 CN**: 以 `HEXAGON_Vect32C(HEXAGON_V32_PUT_B0(data, b))` 从当前函数返回。
- **L1171 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1171 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1172 EN**: Executes a call or declaration centered on `B0`.
  **L1172 CN**: 执行以 `B0` 为核心的调用或声明。
- **L1173 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect32C B1(signed char b) {`.
  **L1173 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect32C B1(signed char b) {`。
- **L1174 EN**: Returns from the current function with `HEXAGON_Vect32C(HEXAGON_V32_PUT_B1(data, b))`.
  **L1174 CN**: 以 `HEXAGON_Vect32C(HEXAGON_V32_PUT_B1(data, b))` 从当前函数返回。
- **L1175 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1175 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1176 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect32C B2(signed char b) {`.
  **L1176 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect32C B2(signed char b) {`。

### Lines 1177-1200

````c
    return HEXAGON_Vect32C(HEXAGON_V32_PUT_B2(data, b));
  };
  HEXAGON_Vect32C B3(signed char b) {
    return HEXAGON_Vect32C(HEXAGON_V32_PUT_B3(data, b));
  };

private:
  int data;
};

#endif /* __cplusplus */

// V65 Vector types
#if __HVX_ARCH__ >= 65
#if defined __HVX__ && (__HVX_LENGTH__ == 128)
  typedef long HEXAGON_VecPred128 __attribute__((__vector_size__(128)))
    __attribute__((aligned(128)));

  typedef long HEXAGON_Vect1024 __attribute__((__vector_size__(128)))
    __attribute__((aligned(128)));

  typedef long HEXAGON_Vect2048 __attribute__((__vector_size__(256)))
    __attribute__((aligned(256)));

````
- **L1177 EN**: Returns from the current function with `HEXAGON_Vect32C(HEXAGON_V32_PUT_B2(data, b))`.
  **L1177 CN**: 以 `HEXAGON_Vect32C(HEXAGON_V32_PUT_B2(data, b))` 从当前函数返回。
- **L1178 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1178 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1179 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `HEXAGON_Vect32C B3(signed char b) {`.
  **L1179 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`HEXAGON_Vect32C B3(signed char b) {`。
- **L1180 EN**: Returns from the current function with `HEXAGON_Vect32C(HEXAGON_V32_PUT_B3(data, b))`.
  **L1180 CN**: 以 `HEXAGON_Vect32C(HEXAGON_V32_PUT_B3(data, b))` 从当前函数返回。
- **L1181 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1181 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1182 EN**: Blank line separating nearby declarations or logic blocks.
  **L1182 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1183 EN**: Sets the access level for following class members to `private`.
  **L1183 CN**: 将后续类成员的访问级别设为 `private`。
- **L1184 EN**: Adds a standalone statement or declaration: `int data;`.
  **L1184 CN**: 添加一条独立语句或声明：`int data;`。
- **L1185 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1185 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1186 EN**: Blank line separating nearby declarations or logic blocks.
  **L1186 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1187 EN**: Closes the current preprocessor conditional block.
  **L1187 CN**: 结束当前预处理条件块。
- **L1188 EN**: Blank line separating nearby declarations or logic blocks.
  **L1188 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1189 EN**: Comment explains nearby logic, constraints, or intent: `V65 Vector types`.
  **L1189 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V65 Vector types`。
- **L1190 EN**: Starts a preprocessor conditional block: `#if __HVX_ARCH__ >= 65`.
  **L1190 CN**: 开始一个预处理条件块：`#if __HVX_ARCH__ >= 65`。
- **L1191 EN**: Starts a preprocessor conditional block: `#if defined __HVX__ && (__HVX_LENGTH__ == 128)`.
  **L1191 CN**: 开始一个预处理条件块：`#if defined __HVX__ && (__HVX_LENGTH__ == 128)`。
- **L1192 EN**: Introduces an alias or helper declaration: `typedef long HEXAGON_VecPred128 __attribute__((__vector_size__(128)))`.
  **L1192 CN**: 引入一条别名或辅助声明：`typedef long HEXAGON_VecPred128 __attribute__((__vector_size__(128)))`。
- **L1193 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((aligned(128)));`.
  **L1193 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((aligned(128)));`。
- **L1194 EN**: Blank line separating nearby declarations or logic blocks.
  **L1194 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1195 EN**: Introduces an alias or helper declaration: `typedef long HEXAGON_Vect1024 __attribute__((__vector_size__(128)))`.
  **L1195 CN**: 引入一条别名或辅助声明：`typedef long HEXAGON_Vect1024 __attribute__((__vector_size__(128)))`。
- **L1196 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((aligned(128)));`.
  **L1196 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((aligned(128)));`。
- **L1197 EN**: Blank line separating nearby declarations or logic blocks.
  **L1197 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1198 EN**: Introduces an alias or helper declaration: `typedef long HEXAGON_Vect2048 __attribute__((__vector_size__(256)))`.
  **L1198 CN**: 引入一条别名或辅助声明：`typedef long HEXAGON_Vect2048 __attribute__((__vector_size__(256)))`。
- **L1199 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((aligned(256)));`.
  **L1199 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((aligned(256)));`。
- **L1200 EN**: Blank line separating nearby declarations or logic blocks.
  **L1200 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1201-1224

````c
  typedef long HEXAGON_UVect1024 __attribute__((__vector_size__(128)))
    __attribute__((aligned(4)));

  typedef long HEXAGON_UVect2048 __attribute__((__vector_size__(256)))
    __attribute__((aligned(4)));

  #define HVX_VectorPred     HEXAGON_VecPred128
  #define HVX_Vector         HEXAGON_Vect1024
  #define HVX_VectorPair     HEXAGON_Vect2048
  #define HVX_UVector        HEXAGON_UVect1024
  #define HVX_UVectorPair    HEXAGON_UVect2048
#else /* defined __HVX__ && (__HVX_LENGTH__ == 128) */
#if defined __HVX__ &&  (__HVX_LENGTH__ == 64)
  typedef long HEXAGON_VecPred64 __attribute__((__vector_size__(64)))
    __attribute__((aligned(64)));

  typedef long HEXAGON_Vect512 __attribute__((__vector_size__(64)))
    __attribute__((aligned(64)));

  typedef long HEXAGON_Vect1024 __attribute__((__vector_size__(128)))
    __attribute__((aligned(128)));

  typedef long HEXAGON_UVect512 __attribute__((__vector_size__(64)))
    __attribute__((aligned(4)));
````
- **L1201 EN**: Introduces an alias or helper declaration: `typedef long HEXAGON_UVect1024 __attribute__((__vector_size__(128)))`.
  **L1201 CN**: 引入一条别名或辅助声明：`typedef long HEXAGON_UVect1024 __attribute__((__vector_size__(128)))`。
- **L1202 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((aligned(4)));`.
  **L1202 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((aligned(4)));`。
- **L1203 EN**: Blank line separating nearby declarations or logic blocks.
  **L1203 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1204 EN**: Introduces an alias or helper declaration: `typedef long HEXAGON_UVect2048 __attribute__((__vector_size__(256)))`.
  **L1204 CN**: 引入一条别名或辅助声明：`typedef long HEXAGON_UVect2048 __attribute__((__vector_size__(256)))`。
- **L1205 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((aligned(4)));`.
  **L1205 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((aligned(4)));`。
- **L1206 EN**: Blank line separating nearby declarations or logic blocks.
  **L1206 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1207 EN**: Defines macro `HVX_VectorPred` for conditional compilation, shorthand, or API generation.
  **L1207 CN**: 定义宏 `HVX_VectorPred`，用于条件编译、简写或 API 生成。
- **L1208 EN**: Defines macro `HVX_Vector` for conditional compilation, shorthand, or API generation.
  **L1208 CN**: 定义宏 `HVX_Vector`，用于条件编译、简写或 API 生成。
- **L1209 EN**: Defines macro `HVX_VectorPair` for conditional compilation, shorthand, or API generation.
  **L1209 CN**: 定义宏 `HVX_VectorPair`，用于条件编译、简写或 API 生成。
- **L1210 EN**: Defines macro `HVX_UVector` for conditional compilation, shorthand, or API generation.
  **L1210 CN**: 定义宏 `HVX_UVector`，用于条件编译、简写或 API 生成。
- **L1211 EN**: Defines macro `HVX_UVectorPair` for conditional compilation, shorthand, or API generation.
  **L1211 CN**: 定义宏 `HVX_UVectorPair`，用于条件编译、简写或 API 生成。
- **L1212 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1212 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1213 EN**: Starts a preprocessor conditional block: `#if defined __HVX__ &&  (__HVX_LENGTH__ == 64)`.
  **L1213 CN**: 开始一个预处理条件块：`#if defined __HVX__ &&  (__HVX_LENGTH__ == 64)`。
- **L1214 EN**: Introduces an alias or helper declaration: `typedef long HEXAGON_VecPred64 __attribute__((__vector_size__(64)))`.
  **L1214 CN**: 引入一条别名或辅助声明：`typedef long HEXAGON_VecPred64 __attribute__((__vector_size__(64)))`。
- **L1215 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((aligned(64)));`.
  **L1215 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((aligned(64)));`。
- **L1216 EN**: Blank line separating nearby declarations or logic blocks.
  **L1216 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1217 EN**: Introduces an alias or helper declaration: `typedef long HEXAGON_Vect512 __attribute__((__vector_size__(64)))`.
  **L1217 CN**: 引入一条别名或辅助声明：`typedef long HEXAGON_Vect512 __attribute__((__vector_size__(64)))`。
- **L1218 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((aligned(64)));`.
  **L1218 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((aligned(64)));`。
- **L1219 EN**: Blank line separating nearby declarations or logic blocks.
  **L1219 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1220 EN**: Introduces an alias or helper declaration: `typedef long HEXAGON_Vect1024 __attribute__((__vector_size__(128)))`.
  **L1220 CN**: 引入一条别名或辅助声明：`typedef long HEXAGON_Vect1024 __attribute__((__vector_size__(128)))`。
- **L1221 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((aligned(128)));`.
  **L1221 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((aligned(128)));`。
- **L1222 EN**: Blank line separating nearby declarations or logic blocks.
  **L1222 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1223 EN**: Introduces an alias or helper declaration: `typedef long HEXAGON_UVect512 __attribute__((__vector_size__(64)))`.
  **L1223 CN**: 引入一条别名或辅助声明：`typedef long HEXAGON_UVect512 __attribute__((__vector_size__(64)))`。
- **L1224 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((aligned(4)));`.
  **L1224 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((aligned(4)));`。

### Lines 1225-1248

````c

  typedef long HEXAGON_UVect1024 __attribute__((__vector_size__(128)))
    __attribute__((aligned(4)));

  #define HVX_VectorPred     HEXAGON_VecPred64
  #define HVX_Vector         HEXAGON_Vect512
  #define HVX_VectorPair     HEXAGON_Vect1024
  #define HVX_UVector        HEXAGON_UVect512
  #define HVX_UVectorPair    HEXAGON_UVect1024
#endif /* defined __HVX__ &&  (__HVX_LENGTH__ == 64) */
#endif /* defined __HVX__ && (__HVX_LENGTH__ == 128) */
#endif /* __HVX_ARCH__ >= 65 */

/* Predicates */

typedef int HEXAGON_Pred;

/***
 *** backward compatibility aliases
 ***/

/* Old names */
#define Q6Vect Q6Vect64
#define Q6V_GET_D Q6V64_GET_D
````
- **L1225 EN**: Blank line separating nearby declarations or logic blocks.
  **L1225 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1226 EN**: Introduces an alias or helper declaration: `typedef long HEXAGON_UVect1024 __attribute__((__vector_size__(128)))`.
  **L1226 CN**: 引入一条别名或辅助声明：`typedef long HEXAGON_UVect1024 __attribute__((__vector_size__(128)))`。
- **L1227 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((aligned(4)));`.
  **L1227 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((aligned(4)));`。
- **L1228 EN**: Blank line separating nearby declarations or logic blocks.
  **L1228 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1229 EN**: Defines macro `HVX_VectorPred` for conditional compilation, shorthand, or API generation.
  **L1229 CN**: 定义宏 `HVX_VectorPred`，用于条件编译、简写或 API 生成。
- **L1230 EN**: Defines macro `HVX_Vector` for conditional compilation, shorthand, or API generation.
  **L1230 CN**: 定义宏 `HVX_Vector`，用于条件编译、简写或 API 生成。
- **L1231 EN**: Defines macro `HVX_VectorPair` for conditional compilation, shorthand, or API generation.
  **L1231 CN**: 定义宏 `HVX_VectorPair`，用于条件编译、简写或 API 生成。
- **L1232 EN**: Defines macro `HVX_UVector` for conditional compilation, shorthand, or API generation.
  **L1232 CN**: 定义宏 `HVX_UVector`，用于条件编译、简写或 API 生成。
- **L1233 EN**: Defines macro `HVX_UVectorPair` for conditional compilation, shorthand, or API generation.
  **L1233 CN**: 定义宏 `HVX_UVectorPair`，用于条件编译、简写或 API 生成。
- **L1234 EN**: Closes the current preprocessor conditional block.
  **L1234 CN**: 结束当前预处理条件块。
- **L1235 EN**: Closes the current preprocessor conditional block.
  **L1235 CN**: 结束当前预处理条件块。
- **L1236 EN**: Closes the current preprocessor conditional block.
  **L1236 CN**: 结束当前预处理条件块。
- **L1237 EN**: Blank line separating nearby declarations or logic blocks.
  **L1237 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1238 EN**: Comment explains nearby logic, constraints, or intent: `Predicates`.
  **L1238 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Predicates`。
- **L1239 EN**: Blank line separating nearby declarations or logic blocks.
  **L1239 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1240 EN**: Introduces an alias or helper declaration: `typedef int HEXAGON_Pred;`.
  **L1240 CN**: 引入一条别名或辅助声明：`typedef int HEXAGON_Pred;`。
- **L1241 EN**: Blank line separating nearby declarations or logic blocks.
  **L1241 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1242 EN**: Separator comment used for visual grouping.
  **L1242 CN**: 用于视觉分组的分隔注释。
- **L1243 EN**: Comment explains nearby logic, constraints, or intent: `backward compatibility aliases`.
  **L1243 CN**: 注释解释附近代码的逻辑、约束或设计意图：`backward compatibility aliases`。
- **L1244 EN**: Separator comment used for visual grouping.
  **L1244 CN**: 用于视觉分组的分隔注释。
- **L1245 EN**: Blank line separating nearby declarations or logic blocks.
  **L1245 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1246 EN**: Comment explains nearby logic, constraints, or intent: `Old names`.
  **L1246 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Old names`。
- **L1247 EN**: Defines macro `Q6Vect` for conditional compilation, shorthand, or API generation.
  **L1247 CN**: 定义宏 `Q6Vect`，用于条件编译、简写或 API 生成。
- **L1248 EN**: Defines macro `Q6V_GET_D` for conditional compilation, shorthand, or API generation.
  **L1248 CN**: 定义宏 `Q6V_GET_D`，用于条件编译、简写或 API 生成。

### Lines 1249-1272

````c
#define Q6V_GET_UD Q6V64_GET_UD
#define Q6V_GET_W0 Q6V64_GET_W0
#define Q6V_GET_W1 Q6V64_GET_W1
#define Q6V_GET_UW0 Q6V64_GET_UW0
#define Q6V_GET_UW1 Q6V64_GET_UW1
#define Q6V_GET_H0 Q6V64_GET_H0
#define Q6V_GET_H1 Q6V64_GET_H1
#define Q6V_GET_H2 Q6V64_GET_H2
#define Q6V_GET_H3 Q6V64_GET_H3
#define Q6V_GET_UH0 Q6V64_GET_UH0
#define Q6V_GET_UH1 Q6V64_GET_UH1
#define Q6V_GET_UH2 Q6V64_GET_UH2
#define Q6V_GET_UH3 Q6V64_GET_UH3
#define Q6V_GET_B0 Q6V64_GET_B0
#define Q6V_GET_B1 Q6V64_GET_B1
#define Q6V_GET_B2 Q6V64_GET_B2
#define Q6V_GET_B3 Q6V64_GET_B3
#define Q6V_GET_B4 Q6V64_GET_B4
#define Q6V_GET_B5 Q6V64_GET_B5
#define Q6V_GET_B6 Q6V64_GET_B6
#define Q6V_GET_B7 Q6V64_GET_B7
#define Q6V_GET_UB0 Q6V64_GET_UB0
#define Q6V_GET_UB1 Q6V64_GET_UB1
#define Q6V_GET_UB2 Q6V64_GET_UB2
````
- **L1249 EN**: Defines macro `Q6V_GET_UD` for conditional compilation, shorthand, or API generation.
  **L1249 CN**: 定义宏 `Q6V_GET_UD`，用于条件编译、简写或 API 生成。
- **L1250 EN**: Defines macro `Q6V_GET_W0` for conditional compilation, shorthand, or API generation.
  **L1250 CN**: 定义宏 `Q6V_GET_W0`，用于条件编译、简写或 API 生成。
- **L1251 EN**: Defines macro `Q6V_GET_W1` for conditional compilation, shorthand, or API generation.
  **L1251 CN**: 定义宏 `Q6V_GET_W1`，用于条件编译、简写或 API 生成。
- **L1252 EN**: Defines macro `Q6V_GET_UW0` for conditional compilation, shorthand, or API generation.
  **L1252 CN**: 定义宏 `Q6V_GET_UW0`，用于条件编译、简写或 API 生成。
- **L1253 EN**: Defines macro `Q6V_GET_UW1` for conditional compilation, shorthand, or API generation.
  **L1253 CN**: 定义宏 `Q6V_GET_UW1`，用于条件编译、简写或 API 生成。
- **L1254 EN**: Defines macro `Q6V_GET_H0` for conditional compilation, shorthand, or API generation.
  **L1254 CN**: 定义宏 `Q6V_GET_H0`，用于条件编译、简写或 API 生成。
- **L1255 EN**: Defines macro `Q6V_GET_H1` for conditional compilation, shorthand, or API generation.
  **L1255 CN**: 定义宏 `Q6V_GET_H1`，用于条件编译、简写或 API 生成。
- **L1256 EN**: Defines macro `Q6V_GET_H2` for conditional compilation, shorthand, or API generation.
  **L1256 CN**: 定义宏 `Q6V_GET_H2`，用于条件编译、简写或 API 生成。
- **L1257 EN**: Defines macro `Q6V_GET_H3` for conditional compilation, shorthand, or API generation.
  **L1257 CN**: 定义宏 `Q6V_GET_H3`，用于条件编译、简写或 API 生成。
- **L1258 EN**: Defines macro `Q6V_GET_UH0` for conditional compilation, shorthand, or API generation.
  **L1258 CN**: 定义宏 `Q6V_GET_UH0`，用于条件编译、简写或 API 生成。
- **L1259 EN**: Defines macro `Q6V_GET_UH1` for conditional compilation, shorthand, or API generation.
  **L1259 CN**: 定义宏 `Q6V_GET_UH1`，用于条件编译、简写或 API 生成。
- **L1260 EN**: Defines macro `Q6V_GET_UH2` for conditional compilation, shorthand, or API generation.
  **L1260 CN**: 定义宏 `Q6V_GET_UH2`，用于条件编译、简写或 API 生成。
- **L1261 EN**: Defines macro `Q6V_GET_UH3` for conditional compilation, shorthand, or API generation.
  **L1261 CN**: 定义宏 `Q6V_GET_UH3`，用于条件编译、简写或 API 生成。
- **L1262 EN**: Defines macro `Q6V_GET_B0` for conditional compilation, shorthand, or API generation.
  **L1262 CN**: 定义宏 `Q6V_GET_B0`，用于条件编译、简写或 API 生成。
- **L1263 EN**: Defines macro `Q6V_GET_B1` for conditional compilation, shorthand, or API generation.
  **L1263 CN**: 定义宏 `Q6V_GET_B1`，用于条件编译、简写或 API 生成。
- **L1264 EN**: Defines macro `Q6V_GET_B2` for conditional compilation, shorthand, or API generation.
  **L1264 CN**: 定义宏 `Q6V_GET_B2`，用于条件编译、简写或 API 生成。
- **L1265 EN**: Defines macro `Q6V_GET_B3` for conditional compilation, shorthand, or API generation.
  **L1265 CN**: 定义宏 `Q6V_GET_B3`，用于条件编译、简写或 API 生成。
- **L1266 EN**: Defines macro `Q6V_GET_B4` for conditional compilation, shorthand, or API generation.
  **L1266 CN**: 定义宏 `Q6V_GET_B4`，用于条件编译、简写或 API 生成。
- **L1267 EN**: Defines macro `Q6V_GET_B5` for conditional compilation, shorthand, or API generation.
  **L1267 CN**: 定义宏 `Q6V_GET_B5`，用于条件编译、简写或 API 生成。
- **L1268 EN**: Defines macro `Q6V_GET_B6` for conditional compilation, shorthand, or API generation.
  **L1268 CN**: 定义宏 `Q6V_GET_B6`，用于条件编译、简写或 API 生成。
- **L1269 EN**: Defines macro `Q6V_GET_B7` for conditional compilation, shorthand, or API generation.
  **L1269 CN**: 定义宏 `Q6V_GET_B7`，用于条件编译、简写或 API 生成。
- **L1270 EN**: Defines macro `Q6V_GET_UB0` for conditional compilation, shorthand, or API generation.
  **L1270 CN**: 定义宏 `Q6V_GET_UB0`，用于条件编译、简写或 API 生成。
- **L1271 EN**: Defines macro `Q6V_GET_UB1` for conditional compilation, shorthand, or API generation.
  **L1271 CN**: 定义宏 `Q6V_GET_UB1`，用于条件编译、简写或 API 生成。
- **L1272 EN**: Defines macro `Q6V_GET_UB2` for conditional compilation, shorthand, or API generation.
  **L1272 CN**: 定义宏 `Q6V_GET_UB2`，用于条件编译、简写或 API 生成。

### Lines 1273-1296

````c
#define Q6V_GET_UB3 Q6V64_GET_UB3
#define Q6V_GET_UB4 Q6V64_GET_UB4
#define Q6V_GET_UB5 Q6V64_GET_UB5
#define Q6V_GET_UB6 Q6V64_GET_UB6
#define Q6V_GET_UB7 Q6V64_GET_UB7
#define Q6V_PUT_D Q6V64_PUT_D
#define Q6V_PUT_W0 Q6V64_PUT_W0
#define Q6V_PUT_W1 Q6V64_PUT_W1
#define Q6V_PUT_H0 Q6V64_PUT_H0
#define Q6V_PUT_H1 Q6V64_PUT_H1
#define Q6V_PUT_H2 Q6V64_PUT_H2
#define Q6V_PUT_H3 Q6V64_PUT_H3
#define Q6V_PUT_B0 Q6V64_PUT_B0
#define Q6V_PUT_B1 Q6V64_PUT_B1
#define Q6V_PUT_B2 Q6V64_PUT_B2
#define Q6V_PUT_B3 Q6V64_PUT_B3
#define Q6V_PUT_B4 Q6V64_PUT_B4
#define Q6V_PUT_B5 Q6V64_PUT_B5
#define Q6V_PUT_B6 Q6V64_PUT_B6
#define Q6V_PUT_B7 Q6V64_PUT_B7
#define Q6V_CREATE_D Q6V64_CREATE_D
#define Q6V_CREATE_W Q6V64_CREATE_W
#define Q6V_CREATE_H Q6V64_CREATE_H
#define Q6V_CREATE_B Q6V64_CREATE_B
````
- **L1273 EN**: Defines macro `Q6V_GET_UB3` for conditional compilation, shorthand, or API generation.
  **L1273 CN**: 定义宏 `Q6V_GET_UB3`，用于条件编译、简写或 API 生成。
- **L1274 EN**: Defines macro `Q6V_GET_UB4` for conditional compilation, shorthand, or API generation.
  **L1274 CN**: 定义宏 `Q6V_GET_UB4`，用于条件编译、简写或 API 生成。
- **L1275 EN**: Defines macro `Q6V_GET_UB5` for conditional compilation, shorthand, or API generation.
  **L1275 CN**: 定义宏 `Q6V_GET_UB5`，用于条件编译、简写或 API 生成。
- **L1276 EN**: Defines macro `Q6V_GET_UB6` for conditional compilation, shorthand, or API generation.
  **L1276 CN**: 定义宏 `Q6V_GET_UB6`，用于条件编译、简写或 API 生成。
- **L1277 EN**: Defines macro `Q6V_GET_UB7` for conditional compilation, shorthand, or API generation.
  **L1277 CN**: 定义宏 `Q6V_GET_UB7`，用于条件编译、简写或 API 生成。
- **L1278 EN**: Defines macro `Q6V_PUT_D` for conditional compilation, shorthand, or API generation.
  **L1278 CN**: 定义宏 `Q6V_PUT_D`，用于条件编译、简写或 API 生成。
- **L1279 EN**: Defines macro `Q6V_PUT_W0` for conditional compilation, shorthand, or API generation.
  **L1279 CN**: 定义宏 `Q6V_PUT_W0`，用于条件编译、简写或 API 生成。
- **L1280 EN**: Defines macro `Q6V_PUT_W1` for conditional compilation, shorthand, or API generation.
  **L1280 CN**: 定义宏 `Q6V_PUT_W1`，用于条件编译、简写或 API 生成。
- **L1281 EN**: Defines macro `Q6V_PUT_H0` for conditional compilation, shorthand, or API generation.
  **L1281 CN**: 定义宏 `Q6V_PUT_H0`，用于条件编译、简写或 API 生成。
- **L1282 EN**: Defines macro `Q6V_PUT_H1` for conditional compilation, shorthand, or API generation.
  **L1282 CN**: 定义宏 `Q6V_PUT_H1`，用于条件编译、简写或 API 生成。
- **L1283 EN**: Defines macro `Q6V_PUT_H2` for conditional compilation, shorthand, or API generation.
  **L1283 CN**: 定义宏 `Q6V_PUT_H2`，用于条件编译、简写或 API 生成。
- **L1284 EN**: Defines macro `Q6V_PUT_H3` for conditional compilation, shorthand, or API generation.
  **L1284 CN**: 定义宏 `Q6V_PUT_H3`，用于条件编译、简写或 API 生成。
- **L1285 EN**: Defines macro `Q6V_PUT_B0` for conditional compilation, shorthand, or API generation.
  **L1285 CN**: 定义宏 `Q6V_PUT_B0`，用于条件编译、简写或 API 生成。
- **L1286 EN**: Defines macro `Q6V_PUT_B1` for conditional compilation, shorthand, or API generation.
  **L1286 CN**: 定义宏 `Q6V_PUT_B1`，用于条件编译、简写或 API 生成。
- **L1287 EN**: Defines macro `Q6V_PUT_B2` for conditional compilation, shorthand, or API generation.
  **L1287 CN**: 定义宏 `Q6V_PUT_B2`，用于条件编译、简写或 API 生成。
- **L1288 EN**: Defines macro `Q6V_PUT_B3` for conditional compilation, shorthand, or API generation.
  **L1288 CN**: 定义宏 `Q6V_PUT_B3`，用于条件编译、简写或 API 生成。
- **L1289 EN**: Defines macro `Q6V_PUT_B4` for conditional compilation, shorthand, or API generation.
  **L1289 CN**: 定义宏 `Q6V_PUT_B4`，用于条件编译、简写或 API 生成。
- **L1290 EN**: Defines macro `Q6V_PUT_B5` for conditional compilation, shorthand, or API generation.
  **L1290 CN**: 定义宏 `Q6V_PUT_B5`，用于条件编译、简写或 API 生成。
- **L1291 EN**: Defines macro `Q6V_PUT_B6` for conditional compilation, shorthand, or API generation.
  **L1291 CN**: 定义宏 `Q6V_PUT_B6`，用于条件编译、简写或 API 生成。
- **L1292 EN**: Defines macro `Q6V_PUT_B7` for conditional compilation, shorthand, or API generation.
  **L1292 CN**: 定义宏 `Q6V_PUT_B7`，用于条件编译、简写或 API 生成。
- **L1293 EN**: Defines macro `Q6V_CREATE_D` for conditional compilation, shorthand, or API generation.
  **L1293 CN**: 定义宏 `Q6V_CREATE_D`，用于条件编译、简写或 API 生成。
- **L1294 EN**: Defines macro `Q6V_CREATE_W` for conditional compilation, shorthand, or API generation.
  **L1294 CN**: 定义宏 `Q6V_CREATE_W`，用于条件编译、简写或 API 生成。
- **L1295 EN**: Defines macro `Q6V_CREATE_H` for conditional compilation, shorthand, or API generation.
  **L1295 CN**: 定义宏 `Q6V_CREATE_H`，用于条件编译、简写或 API 生成。
- **L1296 EN**: Defines macro `Q6V_CREATE_B` for conditional compilation, shorthand, or API generation.
  **L1296 CN**: 定义宏 `Q6V_CREATE_B`，用于条件编译、简写或 API 生成。

### Lines 1297-1320

````c

#ifdef __cplusplus
#define Q6VectC Q6Vect64C
#endif /* __cplusplus */

/* 64 Bit Vectors */

typedef long long __attribute__((__may_alias__)) Q6Vect64;

/* Extract doubleword macros */

#define Q6V64_GET_D(v) (v)
#define Q6V64_GET_UD(v) ((unsigned long long)(v))

/* Extract word macros */

#define Q6V64_GET_W0(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      int w[2];                                                                \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.w[0];                                                \
````
- **L1297 EN**: Blank line separating nearby declarations or logic blocks.
  **L1297 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1298 EN**: Starts a preprocessor conditional block: `#ifdef __cplusplus`.
  **L1298 CN**: 开始一个预处理条件块：`#ifdef __cplusplus`。
- **L1299 EN**: Defines macro `Q6VectC` for conditional compilation, shorthand, or API generation.
  **L1299 CN**: 定义宏 `Q6VectC`，用于条件编译、简写或 API 生成。
- **L1300 EN**: Closes the current preprocessor conditional block.
  **L1300 CN**: 结束当前预处理条件块。
- **L1301 EN**: Blank line separating nearby declarations or logic blocks.
  **L1301 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1302 EN**: Comment explains nearby logic, constraints, or intent: `64 Bit Vectors`.
  **L1302 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64 Bit Vectors`。
- **L1303 EN**: Blank line separating nearby declarations or logic blocks.
  **L1303 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1304 EN**: Introduces an alias or helper declaration: `typedef long long __attribute__((__may_alias__)) Q6Vect64;`.
  **L1304 CN**: 引入一条别名或辅助声明：`typedef long long __attribute__((__may_alias__)) Q6Vect64;`。
- **L1305 EN**: Blank line separating nearby declarations or logic blocks.
  **L1305 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1306 EN**: Comment explains nearby logic, constraints, or intent: `Extract doubleword macros`.
  **L1306 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract doubleword macros`。
- **L1307 EN**: Blank line separating nearby declarations or logic blocks.
  **L1307 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1308 EN**: Defines macro `Q6V64_GET_D(v)` for conditional compilation, shorthand, or API generation.
  **L1308 CN**: 定义宏 `Q6V64_GET_D(v)`，用于条件编译、简写或 API 生成。
- **L1309 EN**: Defines macro `Q6V64_GET_UD(v)` for conditional compilation, shorthand, or API generation.
  **L1309 CN**: 定义宏 `Q6V64_GET_UD(v)`，用于条件编译、简写或 API 生成。
- **L1310 EN**: Blank line separating nearby declarations or logic blocks.
  **L1310 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1311 EN**: Comment explains nearby logic, constraints, or intent: `Extract word macros`.
  **L1311 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract word macros`。
- **L1312 EN**: Blank line separating nearby declarations or logic blocks.
  **L1312 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1313 EN**: Defines macro `Q6V64_GET_W0(v)` for conditional compilation, shorthand, or API generation.
  **L1313 CN**: 定义宏 `Q6V64_GET_W0(v)`，用于条件编译、简写或 API 生成。
- **L1314 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1314 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1315 EN**: Declares union `union`.
  **L1315 CN**: 声明 union `union`。
- **L1316 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1316 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1317 EN**: Continues the surrounding expression or declaration: `int w[2];                                                                \`.
  **L1317 CN**: 继续构造周围的表达式或声明：`int w[2];                                                                \`。
- **L1318 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1318 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1319 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1319 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1320 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.w[0];                                                \`.
  **L1320 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.w[0];                                                \`。

### Lines 1321-1344

````c
  })
#define Q6V64_GET_W1(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      int w[2];                                                                \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.w[1];                                                \
  })
#define Q6V64_GET_UW0(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned int uw[2];                                                      \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.uw[0];                                               \
  })
#define Q6V64_GET_UW1(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned int uw[2];                                                      \
````
- **L1321 EN**: Continues the surrounding expression or declaration: `})`.
  **L1321 CN**: 继续构造周围的表达式或声明：`})`。
- **L1322 EN**: Defines macro `Q6V64_GET_W1(v)` for conditional compilation, shorthand, or API generation.
  **L1322 CN**: 定义宏 `Q6V64_GET_W1(v)`，用于条件编译、简写或 API 生成。
- **L1323 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1323 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1324 EN**: Declares union `union`.
  **L1324 CN**: 声明 union `union`。
- **L1325 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1325 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1326 EN**: Continues the surrounding expression or declaration: `int w[2];                                                                \`.
  **L1326 CN**: 继续构造周围的表达式或声明：`int w[2];                                                                \`。
- **L1327 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1327 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1328 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1328 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1329 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.w[1];                                                \`.
  **L1329 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.w[1];                                                \`。
- **L1330 EN**: Continues the surrounding expression or declaration: `})`.
  **L1330 CN**: 继续构造周围的表达式或声明：`})`。
- **L1331 EN**: Defines macro `Q6V64_GET_UW0(v)` for conditional compilation, shorthand, or API generation.
  **L1331 CN**: 定义宏 `Q6V64_GET_UW0(v)`，用于条件编译、简写或 API 生成。
- **L1332 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1332 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1333 EN**: Declares union `union`.
  **L1333 CN**: 声明 union `union`。
- **L1334 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1334 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1335 EN**: Continues the surrounding expression or declaration: `unsigned int uw[2];                                                      \`.
  **L1335 CN**: 继续构造周围的表达式或声明：`unsigned int uw[2];                                                      \`。
- **L1336 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1336 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1337 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1337 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1338 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.uw[0];                                               \`.
  **L1338 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.uw[0];                                               \`。
- **L1339 EN**: Continues the surrounding expression or declaration: `})`.
  **L1339 CN**: 继续构造周围的表达式或声明：`})`。
- **L1340 EN**: Defines macro `Q6V64_GET_UW1(v)` for conditional compilation, shorthand, or API generation.
  **L1340 CN**: 定义宏 `Q6V64_GET_UW1(v)`，用于条件编译、简写或 API 生成。
- **L1341 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1341 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1342 EN**: Declares union `union`.
  **L1342 CN**: 声明 union `union`。
- **L1343 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1343 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1344 EN**: Continues the surrounding expression or declaration: `unsigned int uw[2];                                                      \`.
  **L1344 CN**: 继续构造周围的表达式或声明：`unsigned int uw[2];                                                      \`。

### Lines 1345-1368

````c
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.uw[1];                                               \
  })

/* Extract half word macros */

#define Q6V64_GET_H0(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.h[0];                                                \
  })
#define Q6V64_GET_H1(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.h[1];                                                \
````
- **L1345 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1345 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1346 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1346 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1347 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.uw[1];                                               \`.
  **L1347 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.uw[1];                                               \`。
- **L1348 EN**: Continues the surrounding expression or declaration: `})`.
  **L1348 CN**: 继续构造周围的表达式或声明：`})`。
- **L1349 EN**: Blank line separating nearby declarations or logic blocks.
  **L1349 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1350 EN**: Comment explains nearby logic, constraints, or intent: `Extract half word macros`.
  **L1350 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract half word macros`。
- **L1351 EN**: Blank line separating nearby declarations or logic blocks.
  **L1351 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1352 EN**: Defines macro `Q6V64_GET_H0(v)` for conditional compilation, shorthand, or API generation.
  **L1352 CN**: 定义宏 `Q6V64_GET_H0(v)`，用于条件编译、简写或 API 生成。
- **L1353 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1353 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1354 EN**: Declares union `union`.
  **L1354 CN**: 声明 union `union`。
- **L1355 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1355 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1356 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L1356 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L1357 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1357 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1358 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1358 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1359 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.h[0];                                                \`.
  **L1359 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.h[0];                                                \`。
- **L1360 EN**: Continues the surrounding expression or declaration: `})`.
  **L1360 CN**: 继续构造周围的表达式或声明：`})`。
- **L1361 EN**: Defines macro `Q6V64_GET_H1(v)` for conditional compilation, shorthand, or API generation.
  **L1361 CN**: 定义宏 `Q6V64_GET_H1(v)`，用于条件编译、简写或 API 生成。
- **L1362 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1362 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1363 EN**: Declares union `union`.
  **L1363 CN**: 声明 union `union`。
- **L1364 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1364 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1365 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L1365 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L1366 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1366 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1367 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1367 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1368 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.h[1];                                                \`.
  **L1368 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.h[1];                                                \`。

### Lines 1369-1392

````c
  })
#define Q6V64_GET_H2(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.h[2];                                                \
  })
#define Q6V64_GET_H3(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.h[3];                                                \
  })
#define Q6V64_GET_UH0(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned short uh[4];                                                    \
````
- **L1369 EN**: Continues the surrounding expression or declaration: `})`.
  **L1369 CN**: 继续构造周围的表达式或声明：`})`。
- **L1370 EN**: Defines macro `Q6V64_GET_H2(v)` for conditional compilation, shorthand, or API generation.
  **L1370 CN**: 定义宏 `Q6V64_GET_H2(v)`，用于条件编译、简写或 API 生成。
- **L1371 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1371 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1372 EN**: Declares union `union`.
  **L1372 CN**: 声明 union `union`。
- **L1373 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1373 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1374 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L1374 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L1375 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1375 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1376 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1376 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1377 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.h[2];                                                \`.
  **L1377 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.h[2];                                                \`。
- **L1378 EN**: Continues the surrounding expression or declaration: `})`.
  **L1378 CN**: 继续构造周围的表达式或声明：`})`。
- **L1379 EN**: Defines macro `Q6V64_GET_H3(v)` for conditional compilation, shorthand, or API generation.
  **L1379 CN**: 定义宏 `Q6V64_GET_H3(v)`，用于条件编译、简写或 API 生成。
- **L1380 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1380 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1381 EN**: Declares union `union`.
  **L1381 CN**: 声明 union `union`。
- **L1382 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1382 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1383 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L1383 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L1384 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1384 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1385 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1385 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1386 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.h[3];                                                \`.
  **L1386 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.h[3];                                                \`。
- **L1387 EN**: Continues the surrounding expression or declaration: `})`.
  **L1387 CN**: 继续构造周围的表达式或声明：`})`。
- **L1388 EN**: Defines macro `Q6V64_GET_UH0(v)` for conditional compilation, shorthand, or API generation.
  **L1388 CN**: 定义宏 `Q6V64_GET_UH0(v)`，用于条件编译、简写或 API 生成。
- **L1389 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1389 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1390 EN**: Declares union `union`.
  **L1390 CN**: 声明 union `union`。
- **L1391 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1391 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1392 EN**: Continues the surrounding expression or declaration: `unsigned short uh[4];                                                    \`.
  **L1392 CN**: 继续构造周围的表达式或声明：`unsigned short uh[4];                                                    \`。

### Lines 1393-1416

````c
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.uh[0];                                               \
  })
#define Q6V64_GET_UH1(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned short uh[4];                                                    \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.uh[1];                                               \
  })
#define Q6V64_GET_UH2(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned short uh[4];                                                    \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.uh[2];                                               \
  })
#define Q6V64_GET_UH3(v)                                                       \
  __extension__({                                                              \
````
- **L1393 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1393 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1394 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1394 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1395 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.uh[0];                                               \`.
  **L1395 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.uh[0];                                               \`。
- **L1396 EN**: Continues the surrounding expression or declaration: `})`.
  **L1396 CN**: 继续构造周围的表达式或声明：`})`。
- **L1397 EN**: Defines macro `Q6V64_GET_UH1(v)` for conditional compilation, shorthand, or API generation.
  **L1397 CN**: 定义宏 `Q6V64_GET_UH1(v)`，用于条件编译、简写或 API 生成。
- **L1398 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1398 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1399 EN**: Declares union `union`.
  **L1399 CN**: 声明 union `union`。
- **L1400 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1400 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1401 EN**: Continues the surrounding expression or declaration: `unsigned short uh[4];                                                    \`.
  **L1401 CN**: 继续构造周围的表达式或声明：`unsigned short uh[4];                                                    \`。
- **L1402 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1402 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1403 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1403 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1404 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.uh[1];                                               \`.
  **L1404 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.uh[1];                                               \`。
- **L1405 EN**: Continues the surrounding expression or declaration: `})`.
  **L1405 CN**: 继续构造周围的表达式或声明：`})`。
- **L1406 EN**: Defines macro `Q6V64_GET_UH2(v)` for conditional compilation, shorthand, or API generation.
  **L1406 CN**: 定义宏 `Q6V64_GET_UH2(v)`，用于条件编译、简写或 API 生成。
- **L1407 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1407 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1408 EN**: Declares union `union`.
  **L1408 CN**: 声明 union `union`。
- **L1409 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1409 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1410 EN**: Continues the surrounding expression or declaration: `unsigned short uh[4];                                                    \`.
  **L1410 CN**: 继续构造周围的表达式或声明：`unsigned short uh[4];                                                    \`。
- **L1411 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1411 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1412 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1412 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1413 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.uh[2];                                               \`.
  **L1413 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.uh[2];                                               \`。
- **L1414 EN**: Continues the surrounding expression or declaration: `})`.
  **L1414 CN**: 继续构造周围的表达式或声明：`})`。
- **L1415 EN**: Defines macro `Q6V64_GET_UH3(v)` for conditional compilation, shorthand, or API generation.
  **L1415 CN**: 定义宏 `Q6V64_GET_UH3(v)`，用于条件编译、简写或 API 生成。
- **L1416 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1416 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。

### Lines 1417-1440

````c
    union {                                                                    \
      long long d;                                                             \
      unsigned short uh[4];                                                    \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.uh[3];                                               \
  })

/* Extract byte macros */

#define Q6V64_GET_B0(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      signed char b[8];                                                        \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.b[0];                                                \
  })
#define Q6V64_GET_B1(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      signed char b[8];                                                        \
````
- **L1417 EN**: Declares union `union`.
  **L1417 CN**: 声明 union `union`。
- **L1418 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1418 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1419 EN**: Continues the surrounding expression or declaration: `unsigned short uh[4];                                                    \`.
  **L1419 CN**: 继续构造周围的表达式或声明：`unsigned short uh[4];                                                    \`。
- **L1420 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1420 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1421 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1421 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1422 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.uh[3];                                               \`.
  **L1422 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.uh[3];                                               \`。
- **L1423 EN**: Continues the surrounding expression or declaration: `})`.
  **L1423 CN**: 继续构造周围的表达式或声明：`})`。
- **L1424 EN**: Blank line separating nearby declarations or logic blocks.
  **L1424 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1425 EN**: Comment explains nearby logic, constraints, or intent: `Extract byte macros`.
  **L1425 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract byte macros`。
- **L1426 EN**: Blank line separating nearby declarations or logic blocks.
  **L1426 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1427 EN**: Defines macro `Q6V64_GET_B0(v)` for conditional compilation, shorthand, or API generation.
  **L1427 CN**: 定义宏 `Q6V64_GET_B0(v)`，用于条件编译、简写或 API 生成。
- **L1428 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1428 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1429 EN**: Declares union `union`.
  **L1429 CN**: 声明 union `union`。
- **L1430 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1430 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1431 EN**: Continues the surrounding expression or declaration: `signed char b[8];                                                        \`.
  **L1431 CN**: 继续构造周围的表达式或声明：`signed char b[8];                                                        \`。
- **L1432 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1432 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1433 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1433 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1434 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[0];                                                \`.
  **L1434 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[0];                                                \`。
- **L1435 EN**: Continues the surrounding expression or declaration: `})`.
  **L1435 CN**: 继续构造周围的表达式或声明：`})`。
- **L1436 EN**: Defines macro `Q6V64_GET_B1(v)` for conditional compilation, shorthand, or API generation.
  **L1436 CN**: 定义宏 `Q6V64_GET_B1(v)`，用于条件编译、简写或 API 生成。
- **L1437 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1437 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1438 EN**: Declares union `union`.
  **L1438 CN**: 声明 union `union`。
- **L1439 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1439 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1440 EN**: Continues the surrounding expression or declaration: `signed char b[8];                                                        \`.
  **L1440 CN**: 继续构造周围的表达式或声明：`signed char b[8];                                                        \`。

### Lines 1441-1464

````c
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.b[1];                                                \
  })
#define Q6V64_GET_B2(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      signed char b[8];                                                        \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.b[2];                                                \
  })
#define Q6V64_GET_B3(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      signed char b[8];                                                        \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.b[3];                                                \
  })
#define Q6V64_GET_B4(v)                                                        \
  __extension__({                                                              \
````
- **L1441 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1441 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1442 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1442 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1443 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[1];                                                \`.
  **L1443 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[1];                                                \`。
- **L1444 EN**: Continues the surrounding expression or declaration: `})`.
  **L1444 CN**: 继续构造周围的表达式或声明：`})`。
- **L1445 EN**: Defines macro `Q6V64_GET_B2(v)` for conditional compilation, shorthand, or API generation.
  **L1445 CN**: 定义宏 `Q6V64_GET_B2(v)`，用于条件编译、简写或 API 生成。
- **L1446 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1446 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1447 EN**: Declares union `union`.
  **L1447 CN**: 声明 union `union`。
- **L1448 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1448 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1449 EN**: Continues the surrounding expression or declaration: `signed char b[8];                                                        \`.
  **L1449 CN**: 继续构造周围的表达式或声明：`signed char b[8];                                                        \`。
- **L1450 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1450 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1451 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1451 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1452 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[2];                                                \`.
  **L1452 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[2];                                                \`。
- **L1453 EN**: Continues the surrounding expression or declaration: `})`.
  **L1453 CN**: 继续构造周围的表达式或声明：`})`。
- **L1454 EN**: Defines macro `Q6V64_GET_B3(v)` for conditional compilation, shorthand, or API generation.
  **L1454 CN**: 定义宏 `Q6V64_GET_B3(v)`，用于条件编译、简写或 API 生成。
- **L1455 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1455 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1456 EN**: Declares union `union`.
  **L1456 CN**: 声明 union `union`。
- **L1457 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1457 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1458 EN**: Continues the surrounding expression or declaration: `signed char b[8];                                                        \`.
  **L1458 CN**: 继续构造周围的表达式或声明：`signed char b[8];                                                        \`。
- **L1459 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1459 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1460 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1460 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1461 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[3];                                                \`.
  **L1461 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[3];                                                \`。
- **L1462 EN**: Continues the surrounding expression or declaration: `})`.
  **L1462 CN**: 继续构造周围的表达式或声明：`})`。
- **L1463 EN**: Defines macro `Q6V64_GET_B4(v)` for conditional compilation, shorthand, or API generation.
  **L1463 CN**: 定义宏 `Q6V64_GET_B4(v)`，用于条件编译、简写或 API 生成。
- **L1464 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1464 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。

### Lines 1465-1488

````c
    union {                                                                    \
      long long d;                                                             \
      signed char b[8];                                                        \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.b[4];                                                \
  })
#define Q6V64_GET_B5(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      signed char b[8];                                                        \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.b[5];                                                \
  })
#define Q6V64_GET_B6(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      signed char b[8];                                                        \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.b[6];                                                \
````
- **L1465 EN**: Declares union `union`.
  **L1465 CN**: 声明 union `union`。
- **L1466 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1466 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1467 EN**: Continues the surrounding expression or declaration: `signed char b[8];                                                        \`.
  **L1467 CN**: 继续构造周围的表达式或声明：`signed char b[8];                                                        \`。
- **L1468 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1468 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1469 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1469 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1470 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[4];                                                \`.
  **L1470 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[4];                                                \`。
- **L1471 EN**: Continues the surrounding expression or declaration: `})`.
  **L1471 CN**: 继续构造周围的表达式或声明：`})`。
- **L1472 EN**: Defines macro `Q6V64_GET_B5(v)` for conditional compilation, shorthand, or API generation.
  **L1472 CN**: 定义宏 `Q6V64_GET_B5(v)`，用于条件编译、简写或 API 生成。
- **L1473 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1473 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1474 EN**: Declares union `union`.
  **L1474 CN**: 声明 union `union`。
- **L1475 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1475 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1476 EN**: Continues the surrounding expression or declaration: `signed char b[8];                                                        \`.
  **L1476 CN**: 继续构造周围的表达式或声明：`signed char b[8];                                                        \`。
- **L1477 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1477 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1478 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1478 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1479 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[5];                                                \`.
  **L1479 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[5];                                                \`。
- **L1480 EN**: Continues the surrounding expression or declaration: `})`.
  **L1480 CN**: 继续构造周围的表达式或声明：`})`。
- **L1481 EN**: Defines macro `Q6V64_GET_B6(v)` for conditional compilation, shorthand, or API generation.
  **L1481 CN**: 定义宏 `Q6V64_GET_B6(v)`，用于条件编译、简写或 API 生成。
- **L1482 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1482 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1483 EN**: Declares union `union`.
  **L1483 CN**: 声明 union `union`。
- **L1484 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1484 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1485 EN**: Continues the surrounding expression or declaration: `signed char b[8];                                                        \`.
  **L1485 CN**: 继续构造周围的表达式或声明：`signed char b[8];                                                        \`。
- **L1486 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1486 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1487 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1487 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1488 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[6];                                                \`.
  **L1488 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[6];                                                \`。

### Lines 1489-1512

````c
  })
#define Q6V64_GET_B7(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      signed char b[8];                                                        \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.b[7];                                                \
  })
#define Q6V64_GET_UB0(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned char ub[8];                                                     \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.ub[0];                                               \
  })
#define Q6V64_GET_UB1(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned char ub[8];                                                     \
````
- **L1489 EN**: Continues the surrounding expression or declaration: `})`.
  **L1489 CN**: 继续构造周围的表达式或声明：`})`。
- **L1490 EN**: Defines macro `Q6V64_GET_B7(v)` for conditional compilation, shorthand, or API generation.
  **L1490 CN**: 定义宏 `Q6V64_GET_B7(v)`，用于条件编译、简写或 API 生成。
- **L1491 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1491 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1492 EN**: Declares union `union`.
  **L1492 CN**: 声明 union `union`。
- **L1493 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1493 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1494 EN**: Continues the surrounding expression or declaration: `signed char b[8];                                                        \`.
  **L1494 CN**: 继续构造周围的表达式或声明：`signed char b[8];                                                        \`。
- **L1495 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1495 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1496 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1496 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1497 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[7];                                                \`.
  **L1497 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[7];                                                \`。
- **L1498 EN**: Continues the surrounding expression or declaration: `})`.
  **L1498 CN**: 继续构造周围的表达式或声明：`})`。
- **L1499 EN**: Defines macro `Q6V64_GET_UB0(v)` for conditional compilation, shorthand, or API generation.
  **L1499 CN**: 定义宏 `Q6V64_GET_UB0(v)`，用于条件编译、简写或 API 生成。
- **L1500 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1500 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1501 EN**: Declares union `union`.
  **L1501 CN**: 声明 union `union`。
- **L1502 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1502 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1503 EN**: Continues the surrounding expression or declaration: `unsigned char ub[8];                                                     \`.
  **L1503 CN**: 继续构造周围的表达式或声明：`unsigned char ub[8];                                                     \`。
- **L1504 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1504 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1505 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1505 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1506 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.ub[0];                                               \`.
  **L1506 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.ub[0];                                               \`。
- **L1507 EN**: Continues the surrounding expression or declaration: `})`.
  **L1507 CN**: 继续构造周围的表达式或声明：`})`。
- **L1508 EN**: Defines macro `Q6V64_GET_UB1(v)` for conditional compilation, shorthand, or API generation.
  **L1508 CN**: 定义宏 `Q6V64_GET_UB1(v)`，用于条件编译、简写或 API 生成。
- **L1509 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1509 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1510 EN**: Declares union `union`.
  **L1510 CN**: 声明 union `union`。
- **L1511 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1511 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1512 EN**: Continues the surrounding expression or declaration: `unsigned char ub[8];                                                     \`.
  **L1512 CN**: 继续构造周围的表达式或声明：`unsigned char ub[8];                                                     \`。

### Lines 1513-1536

````c
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.ub[1];                                               \
  })
#define Q6V64_GET_UB2(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned char ub[8];                                                     \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.ub[2];                                               \
  })
#define Q6V64_GET_UB3(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned char ub[8];                                                     \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.ub[3];                                               \
  })
#define Q6V64_GET_UB4(v)                                                       \
  __extension__({                                                              \
````
- **L1513 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1513 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1514 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1514 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1515 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.ub[1];                                               \`.
  **L1515 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.ub[1];                                               \`。
- **L1516 EN**: Continues the surrounding expression or declaration: `})`.
  **L1516 CN**: 继续构造周围的表达式或声明：`})`。
- **L1517 EN**: Defines macro `Q6V64_GET_UB2(v)` for conditional compilation, shorthand, or API generation.
  **L1517 CN**: 定义宏 `Q6V64_GET_UB2(v)`，用于条件编译、简写或 API 生成。
- **L1518 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1518 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1519 EN**: Declares union `union`.
  **L1519 CN**: 声明 union `union`。
- **L1520 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1520 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1521 EN**: Continues the surrounding expression or declaration: `unsigned char ub[8];                                                     \`.
  **L1521 CN**: 继续构造周围的表达式或声明：`unsigned char ub[8];                                                     \`。
- **L1522 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1522 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1523 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1523 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1524 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.ub[2];                                               \`.
  **L1524 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.ub[2];                                               \`。
- **L1525 EN**: Continues the surrounding expression or declaration: `})`.
  **L1525 CN**: 继续构造周围的表达式或声明：`})`。
- **L1526 EN**: Defines macro `Q6V64_GET_UB3(v)` for conditional compilation, shorthand, or API generation.
  **L1526 CN**: 定义宏 `Q6V64_GET_UB3(v)`，用于条件编译、简写或 API 生成。
- **L1527 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1527 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1528 EN**: Declares union `union`.
  **L1528 CN**: 声明 union `union`。
- **L1529 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1529 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1530 EN**: Continues the surrounding expression or declaration: `unsigned char ub[8];                                                     \`.
  **L1530 CN**: 继续构造周围的表达式或声明：`unsigned char ub[8];                                                     \`。
- **L1531 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1531 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1532 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1532 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1533 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.ub[3];                                               \`.
  **L1533 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.ub[3];                                               \`。
- **L1534 EN**: Continues the surrounding expression or declaration: `})`.
  **L1534 CN**: 继续构造周围的表达式或声明：`})`。
- **L1535 EN**: Defines macro `Q6V64_GET_UB4(v)` for conditional compilation, shorthand, or API generation.
  **L1535 CN**: 定义宏 `Q6V64_GET_UB4(v)`，用于条件编译、简写或 API 生成。
- **L1536 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1536 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。

### Lines 1537-1560

````c
    union {                                                                    \
      long long d;                                                             \
      unsigned char ub[8];                                                     \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.ub[4];                                               \
  })
#define Q6V64_GET_UB5(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned char ub[8];                                                     \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.ub[5];                                               \
  })
#define Q6V64_GET_UB6(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned char ub[8];                                                     \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.ub[6];                                               \
````
- **L1537 EN**: Declares union `union`.
  **L1537 CN**: 声明 union `union`。
- **L1538 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1538 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1539 EN**: Continues the surrounding expression or declaration: `unsigned char ub[8];                                                     \`.
  **L1539 CN**: 继续构造周围的表达式或声明：`unsigned char ub[8];                                                     \`。
- **L1540 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1540 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1541 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1541 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1542 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.ub[4];                                               \`.
  **L1542 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.ub[4];                                               \`。
- **L1543 EN**: Continues the surrounding expression or declaration: `})`.
  **L1543 CN**: 继续构造周围的表达式或声明：`})`。
- **L1544 EN**: Defines macro `Q6V64_GET_UB5(v)` for conditional compilation, shorthand, or API generation.
  **L1544 CN**: 定义宏 `Q6V64_GET_UB5(v)`，用于条件编译、简写或 API 生成。
- **L1545 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1545 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1546 EN**: Declares union `union`.
  **L1546 CN**: 声明 union `union`。
- **L1547 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1547 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1548 EN**: Continues the surrounding expression or declaration: `unsigned char ub[8];                                                     \`.
  **L1548 CN**: 继续构造周围的表达式或声明：`unsigned char ub[8];                                                     \`。
- **L1549 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1549 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1550 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1550 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1551 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.ub[5];                                               \`.
  **L1551 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.ub[5];                                               \`。
- **L1552 EN**: Continues the surrounding expression or declaration: `})`.
  **L1552 CN**: 继续构造周围的表达式或声明：`})`。
- **L1553 EN**: Defines macro `Q6V64_GET_UB6(v)` for conditional compilation, shorthand, or API generation.
  **L1553 CN**: 定义宏 `Q6V64_GET_UB6(v)`，用于条件编译、简写或 API 生成。
- **L1554 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1554 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1555 EN**: Declares union `union`.
  **L1555 CN**: 声明 union `union`。
- **L1556 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1556 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1557 EN**: Continues the surrounding expression or declaration: `unsigned char ub[8];                                                     \`.
  **L1557 CN**: 继续构造周围的表达式或声明：`unsigned char ub[8];                                                     \`。
- **L1558 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1558 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1559 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1559 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1560 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.ub[6];                                               \`.
  **L1560 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.ub[6];                                               \`。

### Lines 1561-1584

````c
  })
#define Q6V64_GET_UB7(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      unsigned char ub[8];                                                     \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.ub[7];                                               \
  })

/* NOTE: All set macros return a Q6Vect64 type */

/* Set doubleword macro */

#define Q6V64_PUT_D(v, new) (new)

/* Set word macros */

#ifdef __qdsp6__

#define Q6V64_PUT_W0(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
````
- **L1561 EN**: Continues the surrounding expression or declaration: `})`.
  **L1561 CN**: 继续构造周围的表达式或声明：`})`。
- **L1562 EN**: Defines macro `Q6V64_GET_UB7(v)` for conditional compilation, shorthand, or API generation.
  **L1562 CN**: 定义宏 `Q6V64_GET_UB7(v)`，用于条件编译、简写或 API 生成。
- **L1563 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1563 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1564 EN**: Declares union `union`.
  **L1564 CN**: 声明 union `union`。
- **L1565 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1565 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1566 EN**: Continues the surrounding expression or declaration: `unsigned char ub[8];                                                     \`.
  **L1566 CN**: 继续构造周围的表达式或声明：`unsigned char ub[8];                                                     \`。
- **L1567 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1567 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1568 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1568 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1569 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.ub[7];                                               \`.
  **L1569 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.ub[7];                                               \`。
- **L1570 EN**: Continues the surrounding expression or declaration: `})`.
  **L1570 CN**: 继续构造周围的表达式或声明：`})`。
- **L1571 EN**: Blank line separating nearby declarations or logic blocks.
  **L1571 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1572 EN**: Comment highlights an implementation note: `NOTE: All set macros return a Q6Vect64 type`.
  **L1572 CN**: 注释强调一条实现说明：`NOTE: All set macros return a Q6Vect64 type`。
- **L1573 EN**: Blank line separating nearby declarations or logic blocks.
  **L1573 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1574 EN**: Comment explains nearby logic, constraints, or intent: `Set doubleword macro`.
  **L1574 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set doubleword macro`。
- **L1575 EN**: Blank line separating nearby declarations or logic blocks.
  **L1575 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1576 EN**: Defines macro `Q6V64_PUT_D(v, new)` for conditional compilation, shorthand, or API generation.
  **L1576 CN**: 定义宏 `Q6V64_PUT_D(v, new)`，用于条件编译、简写或 API 生成。
- **L1577 EN**: Blank line separating nearby declarations or logic blocks.
  **L1577 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1578 EN**: Comment explains nearby logic, constraints, or intent: `Set word macros`.
  **L1578 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set word macros`。
- **L1579 EN**: Blank line separating nearby declarations or logic blocks.
  **L1579 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1580 EN**: Starts a preprocessor conditional block: `#ifdef __qdsp6__`.
  **L1580 CN**: 开始一个预处理条件块：`#ifdef __qdsp6__`。
- **L1581 EN**: Blank line separating nearby declarations or logic blocks.
  **L1581 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1582 EN**: Defines macro `Q6V64_PUT_W0(v, new)` for conditional compilation, shorthand, or API generation.
  **L1582 CN**: 定义宏 `Q6V64_PUT_W0(v, new)`，用于条件编译、简写或 API 生成。
- **L1583 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1583 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1584 EN**: Declares union `union`.
  **L1584 CN**: 声明 union `union`。

### Lines 1585-1608

````c
      long long d;                                                             \
      int w[2];                                                                \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.w[0] = (new);                                        \
    _Q6V64_internal_union.d;                                                   \
  })
#define Q6V64_PUT_W1(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      int w[2];                                                                \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.w[1] = (new);                                        \
    _Q6V64_internal_union.d;                                                   \
  })

#else /* !__qdsp6__ */

#define Q6V64_PUT_W0(v, new)                                                   \
  (((v) & 0xffffffff00000000LL) | ((Q6Vect64)((unsigned int)(new))))
#define Q6V64_PUT_W1(v, new)                                                   \
  (((v) & 0x00000000ffffffffLL) | (((Q6Vect64)(new)) << 32LL))
````
- **L1585 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1585 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1586 EN**: Continues the surrounding expression or declaration: `int w[2];                                                                \`.
  **L1586 CN**: 继续构造周围的表达式或声明：`int w[2];                                                                \`。
- **L1587 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1587 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1588 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1588 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1589 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.w[0] = (new);                                        \`.
  **L1589 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.w[0] = (new);                                        \`。
- **L1590 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1590 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。
- **L1591 EN**: Continues the surrounding expression or declaration: `})`.
  **L1591 CN**: 继续构造周围的表达式或声明：`})`。
- **L1592 EN**: Defines macro `Q6V64_PUT_W1(v, new)` for conditional compilation, shorthand, or API generation.
  **L1592 CN**: 定义宏 `Q6V64_PUT_W1(v, new)`，用于条件编译、简写或 API 生成。
- **L1593 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1593 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1594 EN**: Declares union `union`.
  **L1594 CN**: 声明 union `union`。
- **L1595 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1595 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1596 EN**: Continues the surrounding expression or declaration: `int w[2];                                                                \`.
  **L1596 CN**: 继续构造周围的表达式或声明：`int w[2];                                                                \`。
- **L1597 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1597 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1598 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1598 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1599 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.w[1] = (new);                                        \`.
  **L1599 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.w[1] = (new);                                        \`。
- **L1600 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1600 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。
- **L1601 EN**: Continues the surrounding expression or declaration: `})`.
  **L1601 CN**: 继续构造周围的表达式或声明：`})`。
- **L1602 EN**: Blank line separating nearby declarations or logic blocks.
  **L1602 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1603 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1603 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1604 EN**: Blank line separating nearby declarations or logic blocks.
  **L1604 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1605 EN**: Defines macro `Q6V64_PUT_W0(v, new)` for conditional compilation, shorthand, or API generation.
  **L1605 CN**: 定义宏 `Q6V64_PUT_W0(v, new)`，用于条件编译、简写或 API 生成。
- **L1606 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffffff00000000LL) | ((Q6Vect64)((unsigned int)(new))))`.
  **L1606 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffffff00000000LL) | ((Q6Vect64)((unsigned int)(new))))`。
- **L1607 EN**: Defines macro `Q6V64_PUT_W1(v, new)` for conditional compilation, shorthand, or API generation.
  **L1607 CN**: 定义宏 `Q6V64_PUT_W1(v, new)`，用于条件编译、简写或 API 生成。
- **L1608 EN**: Continues the surrounding expression or declaration: `(((v) & 0x00000000ffffffffLL) | (((Q6Vect64)(new)) << 32LL))`.
  **L1608 CN**: 继续构造周围的表达式或声明：`(((v) & 0x00000000ffffffffLL) | (((Q6Vect64)(new)) << 32LL))`。

### Lines 1609-1632

````c

#endif /* !__qdsp6__ */

/* Set half word macros */

#ifdef __qdsp6__

#define Q6V64_PUT_H0(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.h[0] = (new);                                        \
    _Q6V64_internal_union.d;                                                   \
  })
#define Q6V64_PUT_H1(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
````
- **L1609 EN**: Blank line separating nearby declarations or logic blocks.
  **L1609 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1610 EN**: Closes the current preprocessor conditional block.
  **L1610 CN**: 结束当前预处理条件块。
- **L1611 EN**: Blank line separating nearby declarations or logic blocks.
  **L1611 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1612 EN**: Comment explains nearby logic, constraints, or intent: `Set half word macros`.
  **L1612 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set half word macros`。
- **L1613 EN**: Blank line separating nearby declarations or logic blocks.
  **L1613 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1614 EN**: Starts a preprocessor conditional block: `#ifdef __qdsp6__`.
  **L1614 CN**: 开始一个预处理条件块：`#ifdef __qdsp6__`。
- **L1615 EN**: Blank line separating nearby declarations or logic blocks.
  **L1615 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1616 EN**: Defines macro `Q6V64_PUT_H0(v, new)` for conditional compilation, shorthand, or API generation.
  **L1616 CN**: 定义宏 `Q6V64_PUT_H0(v, new)`，用于条件编译、简写或 API 生成。
- **L1617 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1617 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1618 EN**: Declares union `union`.
  **L1618 CN**: 声明 union `union`。
- **L1619 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1619 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1620 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L1620 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L1621 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1621 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1622 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1622 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1623 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.h[0] = (new);                                        \`.
  **L1623 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.h[0] = (new);                                        \`。
- **L1624 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1624 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。
- **L1625 EN**: Continues the surrounding expression or declaration: `})`.
  **L1625 CN**: 继续构造周围的表达式或声明：`})`。
- **L1626 EN**: Defines macro `Q6V64_PUT_H1(v, new)` for conditional compilation, shorthand, or API generation.
  **L1626 CN**: 定义宏 `Q6V64_PUT_H1(v, new)`，用于条件编译、简写或 API 生成。
- **L1627 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1627 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1628 EN**: Declares union `union`.
  **L1628 CN**: 声明 union `union`。
- **L1629 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1629 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1630 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L1630 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L1631 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1631 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1632 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1632 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。

### Lines 1633-1656

````c
    _Q6V64_internal_union.h[1] = (new);                                        \
    _Q6V64_internal_union.d;                                                   \
  })
#define Q6V64_PUT_H2(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.h[2] = (new);                                        \
    _Q6V64_internal_union.d;                                                   \
  })
#define Q6V64_PUT_H3(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.h[3] = (new);                                        \
    _Q6V64_internal_union.d;                                                   \
  })

````
- **L1633 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.h[1] = (new);                                        \`.
  **L1633 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.h[1] = (new);                                        \`。
- **L1634 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1634 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。
- **L1635 EN**: Continues the surrounding expression or declaration: `})`.
  **L1635 CN**: 继续构造周围的表达式或声明：`})`。
- **L1636 EN**: Defines macro `Q6V64_PUT_H2(v, new)` for conditional compilation, shorthand, or API generation.
  **L1636 CN**: 定义宏 `Q6V64_PUT_H2(v, new)`，用于条件编译、简写或 API 生成。
- **L1637 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1637 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1638 EN**: Declares union `union`.
  **L1638 CN**: 声明 union `union`。
- **L1639 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1639 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1640 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L1640 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L1641 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1641 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1642 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1642 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1643 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.h[2] = (new);                                        \`.
  **L1643 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.h[2] = (new);                                        \`。
- **L1644 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1644 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。
- **L1645 EN**: Continues the surrounding expression or declaration: `})`.
  **L1645 CN**: 继续构造周围的表达式或声明：`})`。
- **L1646 EN**: Defines macro `Q6V64_PUT_H3(v, new)` for conditional compilation, shorthand, or API generation.
  **L1646 CN**: 定义宏 `Q6V64_PUT_H3(v, new)`，用于条件编译、简写或 API 生成。
- **L1647 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1647 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1648 EN**: Declares union `union`.
  **L1648 CN**: 声明 union `union`。
- **L1649 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1649 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1650 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L1650 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L1651 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1651 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1652 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1652 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1653 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.h[3] = (new);                                        \`.
  **L1653 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.h[3] = (new);                                        \`。
- **L1654 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1654 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。
- **L1655 EN**: Continues the surrounding expression or declaration: `})`.
  **L1655 CN**: 继续构造周围的表达式或声明：`})`。
- **L1656 EN**: Blank line separating nearby declarations or logic blocks.
  **L1656 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1657-1680

````c
#else /* !__qdsp6__ */

#define Q6V64_PUT_H0(v, new)                                                   \
  (((v) & 0xffffffffffff0000LL) | ((Q6Vect64)((unsigned short)(new))))
#define Q6V64_PUT_H1(v, new)                                                   \
  (((v) & 0xffffffff0000ffffLL) | (((Q6Vect64)((unsigned short)(new))) << 16LL))
#define Q6V64_PUT_H2(v, new)                                                   \
  (((v) & 0xffff0000ffffffffLL) | (((Q6Vect64)((unsigned short)(new))) << 32LL))
#define Q6V64_PUT_H3(v, new)                                                   \
  (((v) & 0x0000ffffffffffffLL) | (((Q6Vect64)(new)) << 48LL))

#endif /* !__qdsp6__ */

/* Set byte macros */

#ifdef __qdsp6__

#define Q6V64_PUT_B0(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
````
- **L1657 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1657 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1658 EN**: Blank line separating nearby declarations or logic blocks.
  **L1658 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1659 EN**: Defines macro `Q6V64_PUT_H0(v, new)` for conditional compilation, shorthand, or API generation.
  **L1659 CN**: 定义宏 `Q6V64_PUT_H0(v, new)`，用于条件编译、简写或 API 生成。
- **L1660 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffffffffff0000LL) | ((Q6Vect64)((unsigned short)(new))))`.
  **L1660 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffffffffff0000LL) | ((Q6Vect64)((unsigned short)(new))))`。
- **L1661 EN**: Defines macro `Q6V64_PUT_H1(v, new)` for conditional compilation, shorthand, or API generation.
  **L1661 CN**: 定义宏 `Q6V64_PUT_H1(v, new)`，用于条件编译、简写或 API 生成。
- **L1662 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffffff0000ffffLL) | (((Q6Vect64)((unsigned short)(new))) << 16LL))`.
  **L1662 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffffff0000ffffLL) | (((Q6Vect64)((unsigned short)(new))) << 16LL))`。
- **L1663 EN**: Defines macro `Q6V64_PUT_H2(v, new)` for conditional compilation, shorthand, or API generation.
  **L1663 CN**: 定义宏 `Q6V64_PUT_H2(v, new)`，用于条件编译、简写或 API 生成。
- **L1664 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffff0000ffffffffLL) | (((Q6Vect64)((unsigned short)(new))) << 32LL))`.
  **L1664 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffff0000ffffffffLL) | (((Q6Vect64)((unsigned short)(new))) << 32LL))`。
- **L1665 EN**: Defines macro `Q6V64_PUT_H3(v, new)` for conditional compilation, shorthand, or API generation.
  **L1665 CN**: 定义宏 `Q6V64_PUT_H3(v, new)`，用于条件编译、简写或 API 生成。
- **L1666 EN**: Continues the surrounding expression or declaration: `(((v) & 0x0000ffffffffffffLL) | (((Q6Vect64)(new)) << 48LL))`.
  **L1666 CN**: 继续构造周围的表达式或声明：`(((v) & 0x0000ffffffffffffLL) | (((Q6Vect64)(new)) << 48LL))`。
- **L1667 EN**: Blank line separating nearby declarations or logic blocks.
  **L1667 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1668 EN**: Closes the current preprocessor conditional block.
  **L1668 CN**: 结束当前预处理条件块。
- **L1669 EN**: Blank line separating nearby declarations or logic blocks.
  **L1669 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1670 EN**: Comment explains nearby logic, constraints, or intent: `Set byte macros`.
  **L1670 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set byte macros`。
- **L1671 EN**: Blank line separating nearby declarations or logic blocks.
  **L1671 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1672 EN**: Starts a preprocessor conditional block: `#ifdef __qdsp6__`.
  **L1672 CN**: 开始一个预处理条件块：`#ifdef __qdsp6__`。
- **L1673 EN**: Blank line separating nearby declarations or logic blocks.
  **L1673 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1674 EN**: Defines macro `Q6V64_PUT_B0(v, new)` for conditional compilation, shorthand, or API generation.
  **L1674 CN**: 定义宏 `Q6V64_PUT_B0(v, new)`，用于条件编译、简写或 API 生成。
- **L1675 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1675 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1676 EN**: Declares union `union`.
  **L1676 CN**: 声明 union `union`。
- **L1677 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1677 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1678 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L1678 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L1679 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1679 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1680 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1680 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。

### Lines 1681-1704

````c
    _Q6V64_internal_union.b[0] = (new);                                        \
    _Q6V64_internal_union.d;                                                   \
  })
#define Q6V64_PUT_B1(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.b[1] = (new);                                        \
    _Q6V64_internal_union.d;                                                   \
  })
#define Q6V64_PUT_B2(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.b[2] = (new);                                        \
    _Q6V64_internal_union.d;                                                   \
  })
#define Q6V64_PUT_B3(v, new)                                                   \
````
- **L1681 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[0] = (new);                                        \`.
  **L1681 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[0] = (new);                                        \`。
- **L1682 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1682 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。
- **L1683 EN**: Continues the surrounding expression or declaration: `})`.
  **L1683 CN**: 继续构造周围的表达式或声明：`})`。
- **L1684 EN**: Defines macro `Q6V64_PUT_B1(v, new)` for conditional compilation, shorthand, or API generation.
  **L1684 CN**: 定义宏 `Q6V64_PUT_B1(v, new)`，用于条件编译、简写或 API 生成。
- **L1685 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1685 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1686 EN**: Declares union `union`.
  **L1686 CN**: 声明 union `union`。
- **L1687 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1687 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1688 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L1688 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L1689 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1689 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1690 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1690 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1691 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[1] = (new);                                        \`.
  **L1691 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[1] = (new);                                        \`。
- **L1692 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1692 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。
- **L1693 EN**: Continues the surrounding expression or declaration: `})`.
  **L1693 CN**: 继续构造周围的表达式或声明：`})`。
- **L1694 EN**: Defines macro `Q6V64_PUT_B2(v, new)` for conditional compilation, shorthand, or API generation.
  **L1694 CN**: 定义宏 `Q6V64_PUT_B2(v, new)`，用于条件编译、简写或 API 生成。
- **L1695 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1695 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1696 EN**: Declares union `union`.
  **L1696 CN**: 声明 union `union`。
- **L1697 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1697 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1698 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L1698 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L1699 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1699 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1700 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1700 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1701 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[2] = (new);                                        \`.
  **L1701 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[2] = (new);                                        \`。
- **L1702 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1702 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。
- **L1703 EN**: Continues the surrounding expression or declaration: `})`.
  **L1703 CN**: 继续构造周围的表达式或声明：`})`。
- **L1704 EN**: Defines macro `Q6V64_PUT_B3(v, new)` for conditional compilation, shorthand, or API generation.
  **L1704 CN**: 定义宏 `Q6V64_PUT_B3(v, new)`，用于条件编译、简写或 API 生成。

### Lines 1705-1728

````c
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.b[3] = (new);                                        \
    _Q6V64_internal_union.d;                                                   \
  })
#define Q6V64_PUT_B4(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.b[4] = (new);                                        \
    _Q6V64_internal_union.d;                                                   \
  })
#define Q6V64_PUT_B5(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
````
- **L1705 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1705 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1706 EN**: Declares union `union`.
  **L1706 CN**: 声明 union `union`。
- **L1707 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1707 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1708 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L1708 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L1709 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1709 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1710 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1710 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1711 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[3] = (new);                                        \`.
  **L1711 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[3] = (new);                                        \`。
- **L1712 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1712 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。
- **L1713 EN**: Continues the surrounding expression or declaration: `})`.
  **L1713 CN**: 继续构造周围的表达式或声明：`})`。
- **L1714 EN**: Defines macro `Q6V64_PUT_B4(v, new)` for conditional compilation, shorthand, or API generation.
  **L1714 CN**: 定义宏 `Q6V64_PUT_B4(v, new)`，用于条件编译、简写或 API 生成。
- **L1715 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1715 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1716 EN**: Declares union `union`.
  **L1716 CN**: 声明 union `union`。
- **L1717 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1717 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1718 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L1718 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L1719 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1719 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1720 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1720 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1721 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[4] = (new);                                        \`.
  **L1721 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[4] = (new);                                        \`。
- **L1722 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1722 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。
- **L1723 EN**: Continues the surrounding expression or declaration: `})`.
  **L1723 CN**: 继续构造周围的表达式或声明：`})`。
- **L1724 EN**: Defines macro `Q6V64_PUT_B5(v, new)` for conditional compilation, shorthand, or API generation.
  **L1724 CN**: 定义宏 `Q6V64_PUT_B5(v, new)`，用于条件编译、简写或 API 生成。
- **L1725 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1725 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1726 EN**: Declares union `union`.
  **L1726 CN**: 声明 union `union`。
- **L1727 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1727 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1728 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L1728 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。

### Lines 1729-1752

````c
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.b[5] = (new);                                        \
    _Q6V64_internal_union.d;                                                   \
  })
#define Q6V64_PUT_B6(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.b[6] = (new);                                        \
    _Q6V64_internal_union.d;                                                   \
  })
#define Q6V64_PUT_B7(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.d = (v);                                             \
    _Q6V64_internal_union.b[7] = (new);                                        \
    _Q6V64_internal_union.d;                                                   \
````
- **L1729 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1729 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1730 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1730 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1731 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[5] = (new);                                        \`.
  **L1731 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[5] = (new);                                        \`。
- **L1732 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1732 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。
- **L1733 EN**: Continues the surrounding expression or declaration: `})`.
  **L1733 CN**: 继续构造周围的表达式或声明：`})`。
- **L1734 EN**: Defines macro `Q6V64_PUT_B6(v, new)` for conditional compilation, shorthand, or API generation.
  **L1734 CN**: 定义宏 `Q6V64_PUT_B6(v, new)`，用于条件编译、简写或 API 生成。
- **L1735 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1735 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1736 EN**: Declares union `union`.
  **L1736 CN**: 声明 union `union`。
- **L1737 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1737 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1738 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L1738 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L1739 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1739 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1740 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1740 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1741 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[6] = (new);                                        \`.
  **L1741 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[6] = (new);                                        \`。
- **L1742 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1742 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。
- **L1743 EN**: Continues the surrounding expression or declaration: `})`.
  **L1743 CN**: 继续构造周围的表达式或声明：`})`。
- **L1744 EN**: Defines macro `Q6V64_PUT_B7(v, new)` for conditional compilation, shorthand, or API generation.
  **L1744 CN**: 定义宏 `Q6V64_PUT_B7(v, new)`，用于条件编译、简写或 API 生成。
- **L1745 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1745 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1746 EN**: Declares union `union`.
  **L1746 CN**: 声明 union `union`。
- **L1747 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1747 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1748 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L1748 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L1749 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1749 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1750 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d = (v);                                             \`.
  **L1750 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d = (v);                                             \`。
- **L1751 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[7] = (new);                                        \`.
  **L1751 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[7] = (new);                                        \`。
- **L1752 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1752 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。

### Lines 1753-1776

````c
  })

#else /* !__qdsp6__ */

#define Q6V64_PUT_B0(v, new)                                                   \
  (((v) & 0xffffffffffffff00LL) | ((Q6Vect64)((unsigned char)(new))))
#define Q6V64_PUT_B1(v, new)                                                   \
  (((v) & 0xffffffffffff00ffLL) | (((Q6Vect64)((unsigned char)(new))) << 8LL))
#define Q6V64_PUT_B2(v, new)                                                   \
  (((v) & 0xffffffffff00ffffLL) | (((Q6Vect64)((unsigned char)(new))) << 16LL))
#define Q6V64_PUT_B3(v, new)                                                   \
  (((v) & 0xffffffff00ffffffLL) | (((Q6Vect64)((unsigned char)(new))) << 24LL))
#define Q6V64_PUT_B4(v, new)                                                   \
  (((v) & 0xffffff00ffffffffLL) | (((Q6Vect64)((unsigned char)(new))) << 32LL))
#define Q6V64_PUT_B5(v, new)                                                   \
  (((v) & 0xffff00ffffffffffLL) | (((Q6Vect64)((unsigned char)(new))) << 40LL))
#define Q6V64_PUT_B6(v, new)                                                   \
  (((v) & 0xff00ffffffffffffLL) | (((Q6Vect64)((unsigned char)(new))) << 48LL))
#define Q6V64_PUT_B7(v, new)                                                   \
  (((v) & 0x00ffffffffffffffLL) | (((Q6Vect64)(new)) << 56LL))

#endif /* !__qdsp6__ */

/* NOTE: All create macros return a Q6Vect64 type */
````
- **L1753 EN**: Continues the surrounding expression or declaration: `})`.
  **L1753 CN**: 继续构造周围的表达式或声明：`})`。
- **L1754 EN**: Blank line separating nearby declarations or logic blocks.
  **L1754 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1755 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1755 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1756 EN**: Blank line separating nearby declarations or logic blocks.
  **L1756 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1757 EN**: Defines macro `Q6V64_PUT_B0(v, new)` for conditional compilation, shorthand, or API generation.
  **L1757 CN**: 定义宏 `Q6V64_PUT_B0(v, new)`，用于条件编译、简写或 API 生成。
- **L1758 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffffffffffff00LL) | ((Q6Vect64)((unsigned char)(new))))`.
  **L1758 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffffffffffff00LL) | ((Q6Vect64)((unsigned char)(new))))`。
- **L1759 EN**: Defines macro `Q6V64_PUT_B1(v, new)` for conditional compilation, shorthand, or API generation.
  **L1759 CN**: 定义宏 `Q6V64_PUT_B1(v, new)`，用于条件编译、简写或 API 生成。
- **L1760 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffffffffff00ffLL) | (((Q6Vect64)((unsigned char)(new))) << 8LL))`.
  **L1760 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffffffffff00ffLL) | (((Q6Vect64)((unsigned char)(new))) << 8LL))`。
- **L1761 EN**: Defines macro `Q6V64_PUT_B2(v, new)` for conditional compilation, shorthand, or API generation.
  **L1761 CN**: 定义宏 `Q6V64_PUT_B2(v, new)`，用于条件编译、简写或 API 生成。
- **L1762 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffffffff00ffffLL) | (((Q6Vect64)((unsigned char)(new))) << 16LL))`.
  **L1762 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffffffff00ffffLL) | (((Q6Vect64)((unsigned char)(new))) << 16LL))`。
- **L1763 EN**: Defines macro `Q6V64_PUT_B3(v, new)` for conditional compilation, shorthand, or API generation.
  **L1763 CN**: 定义宏 `Q6V64_PUT_B3(v, new)`，用于条件编译、简写或 API 生成。
- **L1764 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffffff00ffffffLL) | (((Q6Vect64)((unsigned char)(new))) << 24LL))`.
  **L1764 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffffff00ffffffLL) | (((Q6Vect64)((unsigned char)(new))) << 24LL))`。
- **L1765 EN**: Defines macro `Q6V64_PUT_B4(v, new)` for conditional compilation, shorthand, or API generation.
  **L1765 CN**: 定义宏 `Q6V64_PUT_B4(v, new)`，用于条件编译、简写或 API 生成。
- **L1766 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffff00ffffffffLL) | (((Q6Vect64)((unsigned char)(new))) << 32LL))`.
  **L1766 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffff00ffffffffLL) | (((Q6Vect64)((unsigned char)(new))) << 32LL))`。
- **L1767 EN**: Defines macro `Q6V64_PUT_B5(v, new)` for conditional compilation, shorthand, or API generation.
  **L1767 CN**: 定义宏 `Q6V64_PUT_B5(v, new)`，用于条件编译、简写或 API 生成。
- **L1768 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffff00ffffffffffLL) | (((Q6Vect64)((unsigned char)(new))) << 40LL))`.
  **L1768 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffff00ffffffffffLL) | (((Q6Vect64)((unsigned char)(new))) << 40LL))`。
- **L1769 EN**: Defines macro `Q6V64_PUT_B6(v, new)` for conditional compilation, shorthand, or API generation.
  **L1769 CN**: 定义宏 `Q6V64_PUT_B6(v, new)`，用于条件编译、简写或 API 生成。
- **L1770 EN**: Continues the surrounding expression or declaration: `(((v) & 0xff00ffffffffffffLL) | (((Q6Vect64)((unsigned char)(new))) << 48LL))`.
  **L1770 CN**: 继续构造周围的表达式或声明：`(((v) & 0xff00ffffffffffffLL) | (((Q6Vect64)((unsigned char)(new))) << 48LL))`。
- **L1771 EN**: Defines macro `Q6V64_PUT_B7(v, new)` for conditional compilation, shorthand, or API generation.
  **L1771 CN**: 定义宏 `Q6V64_PUT_B7(v, new)`，用于条件编译、简写或 API 生成。
- **L1772 EN**: Continues the surrounding expression or declaration: `(((v) & 0x00ffffffffffffffLL) | (((Q6Vect64)(new)) << 56LL))`.
  **L1772 CN**: 继续构造周围的表达式或声明：`(((v) & 0x00ffffffffffffffLL) | (((Q6Vect64)(new)) << 56LL))`。
- **L1773 EN**: Blank line separating nearby declarations or logic blocks.
  **L1773 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1774 EN**: Closes the current preprocessor conditional block.
  **L1774 CN**: 结束当前预处理条件块。
- **L1775 EN**: Blank line separating nearby declarations or logic blocks.
  **L1775 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1776 EN**: Comment highlights an implementation note: `NOTE: All create macros return a Q6Vect64 type`.
  **L1776 CN**: 注释强调一条实现说明：`NOTE: All create macros return a Q6Vect64 type`。

### Lines 1777-1800

````c

/* Create from a doubleword */

#define Q6V64_CREATE_D(d) (d)

/* Create from words */

#ifdef __qdsp6__

#define Q6V64_CREATE_W(w1, w0)                                                 \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      int w[2];                                                                \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.w[0] = (w0);                                         \
    _Q6V64_internal_union.w[1] = (w1);                                         \
    _Q6V64_internal_union.d;                                                   \
  })

#else /* !__qdsp6__ */

#define Q6V64_CREATE_W(w1, w0)                                                 \
  ((((Q6Vect64)(w1)) << 32LL) | ((Q6Vect64)((w0) & 0xffffffff)))
````
- **L1777 EN**: Blank line separating nearby declarations or logic blocks.
  **L1777 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1778 EN**: Comment explains nearby logic, constraints, or intent: `Create from a doubleword`.
  **L1778 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create from a doubleword`。
- **L1779 EN**: Blank line separating nearby declarations or logic blocks.
  **L1779 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1780 EN**: Defines macro `Q6V64_CREATE_D(d)` for conditional compilation, shorthand, or API generation.
  **L1780 CN**: 定义宏 `Q6V64_CREATE_D(d)`，用于条件编译、简写或 API 生成。
- **L1781 EN**: Blank line separating nearby declarations or logic blocks.
  **L1781 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1782 EN**: Comment explains nearby logic, constraints, or intent: `Create from words`.
  **L1782 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create from words`。
- **L1783 EN**: Blank line separating nearby declarations or logic blocks.
  **L1783 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1784 EN**: Starts a preprocessor conditional block: `#ifdef __qdsp6__`.
  **L1784 CN**: 开始一个预处理条件块：`#ifdef __qdsp6__`。
- **L1785 EN**: Blank line separating nearby declarations or logic blocks.
  **L1785 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1786 EN**: Defines macro `Q6V64_CREATE_W(w1, w0)` for conditional compilation, shorthand, or API generation.
  **L1786 CN**: 定义宏 `Q6V64_CREATE_W(w1, w0)`，用于条件编译、简写或 API 生成。
- **L1787 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1787 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1788 EN**: Declares union `union`.
  **L1788 CN**: 声明 union `union`。
- **L1789 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1789 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1790 EN**: Continues the surrounding expression or declaration: `int w[2];                                                                \`.
  **L1790 CN**: 继续构造周围的表达式或声明：`int w[2];                                                                \`。
- **L1791 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1791 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1792 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.w[0] = (w0);                                         \`.
  **L1792 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.w[0] = (w0);                                         \`。
- **L1793 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.w[1] = (w1);                                         \`.
  **L1793 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.w[1] = (w1);                                         \`。
- **L1794 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1794 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。
- **L1795 EN**: Continues the surrounding expression or declaration: `})`.
  **L1795 CN**: 继续构造周围的表达式或声明：`})`。
- **L1796 EN**: Blank line separating nearby declarations or logic blocks.
  **L1796 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1797 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1797 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1798 EN**: Blank line separating nearby declarations or logic blocks.
  **L1798 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1799 EN**: Defines macro `Q6V64_CREATE_W(w1, w0)` for conditional compilation, shorthand, or API generation.
  **L1799 CN**: 定义宏 `Q6V64_CREATE_W(w1, w0)`，用于条件编译、简写或 API 生成。
- **L1800 EN**: Continues the surrounding expression or declaration: `((((Q6Vect64)(w1)) << 32LL) | ((Q6Vect64)((w0) & 0xffffffff)))`.
  **L1800 CN**: 继续构造周围的表达式或声明：`((((Q6Vect64)(w1)) << 32LL) | ((Q6Vect64)((w0) & 0xffffffff)))`。

### Lines 1801-1824

````c

#endif /* !__qdsp6__ */

/* Create from half words */

#ifdef __qdsp6__

#define Q6V64_CREATE_H(h3, h2, h1, h0)                                         \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[4];                                                              \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.h[0] = (h0);                                         \
    _Q6V64_internal_union.h[1] = (h1);                                         \
    _Q6V64_internal_union.h[2] = (h2);                                         \
    _Q6V64_internal_union.h[3] = (h3);                                         \
    _Q6V64_internal_union.d;                                                   \
  })

#else /* !__qdsp6__ */

#define Q6V64_CREATE_H(h3, h2, h1, h0)                                         \
  ((((Q6Vect64)(h3)) << 48LL) | (((Q6Vect64)((h2) & 0xffff)) << 32LL) |        \
````
- **L1801 EN**: Blank line separating nearby declarations or logic blocks.
  **L1801 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1802 EN**: Closes the current preprocessor conditional block.
  **L1802 CN**: 结束当前预处理条件块。
- **L1803 EN**: Blank line separating nearby declarations or logic blocks.
  **L1803 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1804 EN**: Comment explains nearby logic, constraints, or intent: `Create from half words`.
  **L1804 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create from half words`。
- **L1805 EN**: Blank line separating nearby declarations or logic blocks.
  **L1805 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1806 EN**: Starts a preprocessor conditional block: `#ifdef __qdsp6__`.
  **L1806 CN**: 开始一个预处理条件块：`#ifdef __qdsp6__`。
- **L1807 EN**: Blank line separating nearby declarations or logic blocks.
  **L1807 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1808 EN**: Defines macro `Q6V64_CREATE_H(h3, h2, h1, h0)` for conditional compilation, shorthand, or API generation.
  **L1808 CN**: 定义宏 `Q6V64_CREATE_H(h3, h2, h1, h0)`，用于条件编译、简写或 API 生成。
- **L1809 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1809 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1810 EN**: Declares union `union`.
  **L1810 CN**: 声明 union `union`。
- **L1811 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1811 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1812 EN**: Continues the surrounding expression or declaration: `short h[4];                                                              \`.
  **L1812 CN**: 继续构造周围的表达式或声明：`short h[4];                                                              \`。
- **L1813 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1813 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1814 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.h[0] = (h0);                                         \`.
  **L1814 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.h[0] = (h0);                                         \`。
- **L1815 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.h[1] = (h1);                                         \`.
  **L1815 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.h[1] = (h1);                                         \`。
- **L1816 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.h[2] = (h2);                                         \`.
  **L1816 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.h[2] = (h2);                                         \`。
- **L1817 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.h[3] = (h3);                                         \`.
  **L1817 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.h[3] = (h3);                                         \`。
- **L1818 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1818 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。
- **L1819 EN**: Continues the surrounding expression or declaration: `})`.
  **L1819 CN**: 继续构造周围的表达式或声明：`})`。
- **L1820 EN**: Blank line separating nearby declarations or logic blocks.
  **L1820 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1821 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1821 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1822 EN**: Blank line separating nearby declarations or logic blocks.
  **L1822 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1823 EN**: Defines macro `Q6V64_CREATE_H(h3, h2, h1, h0)` for conditional compilation, shorthand, or API generation.
  **L1823 CN**: 定义宏 `Q6V64_CREATE_H(h3, h2, h1, h0)`，用于条件编译、简写或 API 生成。
- **L1824 EN**: Continues the surrounding expression or declaration: `((((Q6Vect64)(h3)) << 48LL) | (((Q6Vect64)((h2) & 0xffff)) << 32LL) |        \`.
  **L1824 CN**: 继续构造周围的表达式或声明：`((((Q6Vect64)(h3)) << 48LL) | (((Q6Vect64)((h2) & 0xffff)) << 32LL) |        \`。

### Lines 1825-1848

````c
   (((Q6Vect64)((h1) & 0xffff)) << 16LL) | ((Q6Vect64)((h0) & 0xffff)))

#endif /* !__qdsp6__ */

/* Create from bytes */

#ifdef __qdsp6__

#define Q6V64_CREATE_B(b7, b6, b5, b4, b3, b2, b1, b0)                         \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[8];                                                               \
    } _Q6V64_internal_union;                                                   \
    _Q6V64_internal_union.b[0] = (b0);                                         \
    _Q6V64_internal_union.b[1] = (b1);                                         \
    _Q6V64_internal_union.b[2] = (b2);                                         \
    _Q6V64_internal_union.b[3] = (b3);                                         \
    _Q6V64_internal_union.b[4] = (b4);                                         \
    _Q6V64_internal_union.b[5] = (b5);                                         \
    _Q6V64_internal_union.b[6] = (b6);                                         \
    _Q6V64_internal_union.b[7] = (b7);                                         \
    _Q6V64_internal_union.d;                                                   \
  })
````
- **L1825 EN**: Continues the surrounding expression or declaration: `(((Q6Vect64)((h1) & 0xffff)) << 16LL) | ((Q6Vect64)((h0) & 0xffff)))`.
  **L1825 CN**: 继续构造周围的表达式或声明：`(((Q6Vect64)((h1) & 0xffff)) << 16LL) | ((Q6Vect64)((h0) & 0xffff)))`。
- **L1826 EN**: Blank line separating nearby declarations or logic blocks.
  **L1826 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1827 EN**: Closes the current preprocessor conditional block.
  **L1827 CN**: 结束当前预处理条件块。
- **L1828 EN**: Blank line separating nearby declarations or logic blocks.
  **L1828 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1829 EN**: Comment explains nearby logic, constraints, or intent: `Create from bytes`.
  **L1829 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create from bytes`。
- **L1830 EN**: Blank line separating nearby declarations or logic blocks.
  **L1830 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1831 EN**: Starts a preprocessor conditional block: `#ifdef __qdsp6__`.
  **L1831 CN**: 开始一个预处理条件块：`#ifdef __qdsp6__`。
- **L1832 EN**: Blank line separating nearby declarations or logic blocks.
  **L1832 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1833 EN**: Defines macro `Q6V64_CREATE_B(b7, b6, b5, b4, b3, b2, b1, b0)` for conditional compilation, shorthand, or API generation.
  **L1833 CN**: 定义宏 `Q6V64_CREATE_B(b7, b6, b5, b4, b3, b2, b1, b0)`，用于条件编译、简写或 API 生成。
- **L1834 EN**: Continues logic associated with callable symbol `__extension__`.
  **L1834 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L1835 EN**: Declares union `union`.
  **L1835 CN**: 声明 union `union`。
- **L1836 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L1836 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L1837 EN**: Continues the surrounding expression or declaration: `char b[8];                                                               \`.
  **L1837 CN**: 继续构造周围的表达式或声明：`char b[8];                                                               \`。
- **L1838 EN**: Continues the surrounding expression or declaration: `} _Q6V64_internal_union;                                                   \`.
  **L1838 CN**: 继续构造周围的表达式或声明：`} _Q6V64_internal_union;                                                   \`。
- **L1839 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[0] = (b0);                                         \`.
  **L1839 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[0] = (b0);                                         \`。
- **L1840 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[1] = (b1);                                         \`.
  **L1840 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[1] = (b1);                                         \`。
- **L1841 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[2] = (b2);                                         \`.
  **L1841 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[2] = (b2);                                         \`。
- **L1842 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[3] = (b3);                                         \`.
  **L1842 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[3] = (b3);                                         \`。
- **L1843 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[4] = (b4);                                         \`.
  **L1843 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[4] = (b4);                                         \`。
- **L1844 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[5] = (b5);                                         \`.
  **L1844 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[5] = (b5);                                         \`。
- **L1845 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[6] = (b6);                                         \`.
  **L1845 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[6] = (b6);                                         \`。
- **L1846 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.b[7] = (b7);                                         \`.
  **L1846 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.b[7] = (b7);                                         \`。
- **L1847 EN**: Continues the surrounding expression or declaration: `_Q6V64_internal_union.d;                                                   \`.
  **L1847 CN**: 继续构造周围的表达式或声明：`_Q6V64_internal_union.d;                                                   \`。
- **L1848 EN**: Continues the surrounding expression or declaration: `})`.
  **L1848 CN**: 继续构造周围的表达式或声明：`})`。

### Lines 1849-1872

````c

#else /* !__qdsp6__ */

#define Q6V64_CREATE_B(b7, b6, b5, b4, b3, b2, b1, b0)                         \
  ((((Q6Vect64)(b7)) << 56LL) | (((Q6Vect64)((b6) & 0xff)) << 48LL) |          \
   (((Q6Vect64)((b5) & 0xff)) << 40LL) | (((Q6Vect64)((b4) & 0xff)) << 32LL) | \
   (((Q6Vect64)((b3) & 0xff)) << 24LL) | (((Q6Vect64)((b2) & 0xff)) << 16LL) | \
   (((Q6Vect64)((b1) & 0xff)) << 8LL) | ((Q6Vect64)((b0) & 0xff)))

#endif /* !__qdsp6__ */

#ifdef __cplusplus

class Q6Vect64C {
public:
  // Constructors
  Q6Vect64C(long long d = 0) : data(d) {};
  Q6Vect64C(int w1, int w0) : data(Q6V64_CREATE_W(w1, w0)) {};
  Q6Vect64C(short h3, short h2, short h1, short h0)
      : data(Q6V64_CREATE_H(h3, h2, h1, h0)) {};
  Q6Vect64C(signed char b7, signed char b6, signed char b5, signed char b4,
            signed char b3, signed char b2, signed char b1, signed char b0)
      : data(Q6V64_CREATE_B(b7, b6, b5, b4, b3, b2, b1, b0)) {};
  Q6Vect64C(const Q6Vect64C &v) : data(v.data) {};
````
- **L1849 EN**: Blank line separating nearby declarations or logic blocks.
  **L1849 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1850 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1850 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1851 EN**: Blank line separating nearby declarations or logic blocks.
  **L1851 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1852 EN**: Defines macro `Q6V64_CREATE_B(b7, b6, b5, b4, b3, b2, b1, b0)` for conditional compilation, shorthand, or API generation.
  **L1852 CN**: 定义宏 `Q6V64_CREATE_B(b7, b6, b5, b4, b3, b2, b1, b0)`，用于条件编译、简写或 API 生成。
- **L1853 EN**: Continues the surrounding expression or declaration: `((((Q6Vect64)(b7)) << 56LL) | (((Q6Vect64)((b6) & 0xff)) << 48LL) |          \`.
  **L1853 CN**: 继续构造周围的表达式或声明：`((((Q6Vect64)(b7)) << 56LL) | (((Q6Vect64)((b6) & 0xff)) << 48LL) |          \`。
- **L1854 EN**: Continues the surrounding expression or declaration: `(((Q6Vect64)((b5) & 0xff)) << 40LL) | (((Q6Vect64)((b4) & 0xff)) << 32LL) | \`.
  **L1854 CN**: 继续构造周围的表达式或声明：`(((Q6Vect64)((b5) & 0xff)) << 40LL) | (((Q6Vect64)((b4) & 0xff)) << 32LL) | \`。
- **L1855 EN**: Continues the surrounding expression or declaration: `(((Q6Vect64)((b3) & 0xff)) << 24LL) | (((Q6Vect64)((b2) & 0xff)) << 16LL) | \`.
  **L1855 CN**: 继续构造周围的表达式或声明：`(((Q6Vect64)((b3) & 0xff)) << 24LL) | (((Q6Vect64)((b2) & 0xff)) << 16LL) | \`。
- **L1856 EN**: Continues the surrounding expression or declaration: `(((Q6Vect64)((b1) & 0xff)) << 8LL) | ((Q6Vect64)((b0) & 0xff)))`.
  **L1856 CN**: 继续构造周围的表达式或声明：`(((Q6Vect64)((b1) & 0xff)) << 8LL) | ((Q6Vect64)((b0) & 0xff)))`。
- **L1857 EN**: Blank line separating nearby declarations or logic blocks.
  **L1857 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1858 EN**: Closes the current preprocessor conditional block.
  **L1858 CN**: 结束当前预处理条件块。
- **L1859 EN**: Blank line separating nearby declarations or logic blocks.
  **L1859 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1860 EN**: Starts a preprocessor conditional block: `#ifdef __cplusplus`.
  **L1860 CN**: 开始一个预处理条件块：`#ifdef __cplusplus`。
- **L1861 EN**: Blank line separating nearby declarations or logic blocks.
  **L1861 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1862 EN**: Declares class `Q6Vect64C`.
  **L1862 CN**: 声明 class `Q6Vect64C`。
- **L1863 EN**: Sets the access level for following class members to `public`.
  **L1863 CN**: 将后续类成员的访问级别设为 `public`。
- **L1864 EN**: Comment explains nearby logic, constraints, or intent: `Constructors`.
  **L1864 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructors`。
- **L1865 EN**: Executes a call or declaration centered on `Q6Vect64C`.
  **L1865 CN**: 执行以 `Q6Vect64C` 为核心的调用或声明。
- **L1866 EN**: Executes a call or declaration centered on `Q6Vect64C`.
  **L1866 CN**: 执行以 `Q6Vect64C` 为核心的调用或声明。
- **L1867 EN**: Continues logic associated with callable symbol `Q6Vect64C`.
  **L1867 CN**: 继续与可调用符号 `Q6Vect64C` 相关的逻辑。
- **L1868 EN**: Executes a call or declaration centered on `data`.
  **L1868 CN**: 执行以 `data` 为核心的调用或声明。
- **L1869 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Q6Vect64C(signed char b7, signed char b6, signed char b5, signed char b4,`.
  **L1869 CN**: 继续一个多行参数列表、初始化器或聚合项：`Q6Vect64C(signed char b7, signed char b6, signed char b5, signed char b4,`。
- **L1870 EN**: Continues the surrounding expression or declaration: `signed char b3, signed char b2, signed char b1, signed char b0)`.
  **L1870 CN**: 继续构造周围的表达式或声明：`signed char b3, signed char b2, signed char b1, signed char b0)`。
- **L1871 EN**: Executes a call or declaration centered on `data`.
  **L1871 CN**: 执行以 `data` 为核心的调用或声明。
- **L1872 EN**: Executes a call or declaration centered on `Q6Vect64C`.
  **L1872 CN**: 执行以 `Q6Vect64C` 为核心的调用或声明。

### Lines 1873-1896

````c

  Q6Vect64C &operator=(const Q6Vect64C &v) {
    data = v.data;
    return *this;
  };

  operator long long() {
    return data;
  };

  // Extract doubleword methods
  long long D(void) {
    return Q6V64_GET_D(data);
  };
  unsigned long long UD(void) {
    return Q6V64_GET_UD(data);
  };

  // Extract word methods
  int W0(void) {
    return Q6V64_GET_W0(data);
  };
  int W1(void) {
    return Q6V64_GET_W1(data);
````
- **L1873 EN**: Blank line separating nearby declarations or logic blocks.
  **L1873 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1874 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect64C &operator=(const Q6Vect64C &v) {`.
  **L1874 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect64C &operator=(const Q6Vect64C &v) {`。
- **L1875 EN**: Adds a standalone statement or declaration: `data = v.data;`.
  **L1875 CN**: 添加一条独立语句或声明：`data = v.data;`。
- **L1876 EN**: Returns from the current function with `*this`.
  **L1876 CN**: 以 `*this` 从当前函数返回。
- **L1877 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1877 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1878 EN**: Blank line separating nearby declarations or logic blocks.
  **L1878 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1879 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `operator long long() {`.
  **L1879 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`operator long long() {`。
- **L1880 EN**: Returns from the current function with `data`.
  **L1880 CN**: 以 `data` 从当前函数返回。
- **L1881 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1881 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1882 EN**: Blank line separating nearby declarations or logic blocks.
  **L1882 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1883 EN**: Comment explains nearby logic, constraints, or intent: `Extract doubleword methods`.
  **L1883 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract doubleword methods`。
- **L1884 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `long long D(void) {`.
  **L1884 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`long long D(void) {`。
- **L1885 EN**: Returns from the current function with `Q6V64_GET_D(data)`.
  **L1885 CN**: 以 `Q6V64_GET_D(data)` 从当前函数返回。
- **L1886 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1886 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1887 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned long long UD(void) {`.
  **L1887 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned long long UD(void) {`。
- **L1888 EN**: Returns from the current function with `Q6V64_GET_UD(data)`.
  **L1888 CN**: 以 `Q6V64_GET_UD(data)` 从当前函数返回。
- **L1889 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1889 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1890 EN**: Blank line separating nearby declarations or logic blocks.
  **L1890 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1891 EN**: Comment explains nearby logic, constraints, or intent: `Extract word methods`.
  **L1891 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract word methods`。
- **L1892 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `int W0(void) {`.
  **L1892 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`int W0(void) {`。
- **L1893 EN**: Returns from the current function with `Q6V64_GET_W0(data)`.
  **L1893 CN**: 以 `Q6V64_GET_W0(data)` 从当前函数返回。
- **L1894 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1894 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1895 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `int W1(void) {`.
  **L1895 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`int W1(void) {`。
- **L1896 EN**: Returns from the current function with `Q6V64_GET_W1(data)`.
  **L1896 CN**: 以 `Q6V64_GET_W1(data)` 从当前函数返回。

### Lines 1897-1920

````c
  };
  unsigned int UW0(void) {
    return Q6V64_GET_UW0(data);
  };
  unsigned int UW1(void) {
    return Q6V64_GET_UW1(data);
  };

  // Extract half word methods
  short H0(void) {
    return Q6V64_GET_H0(data);
  };
  short H1(void) {
    return Q6V64_GET_H1(data);
  };
  short H2(void) {
    return Q6V64_GET_H2(data);
  };
  short H3(void) {
    return Q6V64_GET_H3(data);
  };
  unsigned short UH0(void) {
    return Q6V64_GET_UH0(data);
  };
````
- **L1897 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1897 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1898 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned int UW0(void) {`.
  **L1898 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned int UW0(void) {`。
- **L1899 EN**: Returns from the current function with `Q6V64_GET_UW0(data)`.
  **L1899 CN**: 以 `Q6V64_GET_UW0(data)` 从当前函数返回。
- **L1900 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1900 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1901 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned int UW1(void) {`.
  **L1901 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned int UW1(void) {`。
- **L1902 EN**: Returns from the current function with `Q6V64_GET_UW1(data)`.
  **L1902 CN**: 以 `Q6V64_GET_UW1(data)` 从当前函数返回。
- **L1903 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1903 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1904 EN**: Blank line separating nearby declarations or logic blocks.
  **L1904 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1905 EN**: Comment explains nearby logic, constraints, or intent: `Extract half word methods`.
  **L1905 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract half word methods`。
- **L1906 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `short H0(void) {`.
  **L1906 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`short H0(void) {`。
- **L1907 EN**: Returns from the current function with `Q6V64_GET_H0(data)`.
  **L1907 CN**: 以 `Q6V64_GET_H0(data)` 从当前函数返回。
- **L1908 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1908 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1909 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `short H1(void) {`.
  **L1909 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`short H1(void) {`。
- **L1910 EN**: Returns from the current function with `Q6V64_GET_H1(data)`.
  **L1910 CN**: 以 `Q6V64_GET_H1(data)` 从当前函数返回。
- **L1911 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1911 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1912 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `short H2(void) {`.
  **L1912 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`short H2(void) {`。
- **L1913 EN**: Returns from the current function with `Q6V64_GET_H2(data)`.
  **L1913 CN**: 以 `Q6V64_GET_H2(data)` 从当前函数返回。
- **L1914 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1914 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1915 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `short H3(void) {`.
  **L1915 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`short H3(void) {`。
- **L1916 EN**: Returns from the current function with `Q6V64_GET_H3(data)`.
  **L1916 CN**: 以 `Q6V64_GET_H3(data)` 从当前函数返回。
- **L1917 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1917 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1918 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned short UH0(void) {`.
  **L1918 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned short UH0(void) {`。
- **L1919 EN**: Returns from the current function with `Q6V64_GET_UH0(data)`.
  **L1919 CN**: 以 `Q6V64_GET_UH0(data)` 从当前函数返回。
- **L1920 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1920 CN**: 结束当前声明作用域，例如结构体、枚举或类。

### Lines 1921-1944

````c
  unsigned short UH1(void) {
    return Q6V64_GET_UH1(data);
  };
  unsigned short UH2(void) {
    return Q6V64_GET_UH2(data);
  };
  unsigned short UH3(void) {
    return Q6V64_GET_UH3(data);
  };

  // Extract byte methods
  signed char b0(void) { return Q6V64_GET_B0(data); };
  signed char B0(void) { return b0(); };
  signed char B1(void) {
    return Q6V64_GET_B1(data);
  };
  signed char B2(void) {
    return Q6V64_GET_B2(data);
  };
  signed char B3(void) {
    return Q6V64_GET_B3(data);
  };
  signed char B4(void) {
    return Q6V64_GET_B4(data);
````
- **L1921 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned short UH1(void) {`.
  **L1921 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned short UH1(void) {`。
- **L1922 EN**: Returns from the current function with `Q6V64_GET_UH1(data)`.
  **L1922 CN**: 以 `Q6V64_GET_UH1(data)` 从当前函数返回。
- **L1923 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1923 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1924 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned short UH2(void) {`.
  **L1924 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned short UH2(void) {`。
- **L1925 EN**: Returns from the current function with `Q6V64_GET_UH2(data)`.
  **L1925 CN**: 以 `Q6V64_GET_UH2(data)` 从当前函数返回。
- **L1926 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1926 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1927 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned short UH3(void) {`.
  **L1927 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned short UH3(void) {`。
- **L1928 EN**: Returns from the current function with `Q6V64_GET_UH3(data)`.
  **L1928 CN**: 以 `Q6V64_GET_UH3(data)` 从当前函数返回。
- **L1929 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1929 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1930 EN**: Blank line separating nearby declarations or logic blocks.
  **L1930 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1931 EN**: Comment explains nearby logic, constraints, or intent: `Extract byte methods`.
  **L1931 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract byte methods`。
- **L1932 EN**: Executes a call or declaration centered on `b0`.
  **L1932 CN**: 执行以 `b0` 为核心的调用或声明。
- **L1933 EN**: Executes a call or declaration centered on `B0`.
  **L1933 CN**: 执行以 `B0` 为核心的调用或声明。
- **L1934 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B1(void) {`.
  **L1934 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B1(void) {`。
- **L1935 EN**: Returns from the current function with `Q6V64_GET_B1(data)`.
  **L1935 CN**: 以 `Q6V64_GET_B1(data)` 从当前函数返回。
- **L1936 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1936 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1937 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B2(void) {`.
  **L1937 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B2(void) {`。
- **L1938 EN**: Returns from the current function with `Q6V64_GET_B2(data)`.
  **L1938 CN**: 以 `Q6V64_GET_B2(data)` 从当前函数返回。
- **L1939 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1939 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1940 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B3(void) {`.
  **L1940 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B3(void) {`。
- **L1941 EN**: Returns from the current function with `Q6V64_GET_B3(data)`.
  **L1941 CN**: 以 `Q6V64_GET_B3(data)` 从当前函数返回。
- **L1942 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1942 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1943 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B4(void) {`.
  **L1943 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B4(void) {`。
- **L1944 EN**: Returns from the current function with `Q6V64_GET_B4(data)`.
  **L1944 CN**: 以 `Q6V64_GET_B4(data)` 从当前函数返回。

### Lines 1945-1968

````c
  };
  signed char B5(void) {
    return Q6V64_GET_B5(data);
  };
  signed char B6(void) {
    return Q6V64_GET_B6(data);
  };
  signed char B7(void) {
    return Q6V64_GET_B7(data);
  };
  unsigned char UB0(void) {
    return Q6V64_GET_UB0(data);
  };
  unsigned char UB1(void) {
    return Q6V64_GET_UB1(data);
  };
  unsigned char UB2(void) {
    return Q6V64_GET_UB2(data);
  };
  unsigned char UB3(void) {
    return Q6V64_GET_UB3(data);
  };
  unsigned char UB4(void) {
    return Q6V64_GET_UB4(data);
````
- **L1945 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1945 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1946 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B5(void) {`.
  **L1946 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B5(void) {`。
- **L1947 EN**: Returns from the current function with `Q6V64_GET_B5(data)`.
  **L1947 CN**: 以 `Q6V64_GET_B5(data)` 从当前函数返回。
- **L1948 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1948 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1949 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B6(void) {`.
  **L1949 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B6(void) {`。
- **L1950 EN**: Returns from the current function with `Q6V64_GET_B6(data)`.
  **L1950 CN**: 以 `Q6V64_GET_B6(data)` 从当前函数返回。
- **L1951 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1951 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1952 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B7(void) {`.
  **L1952 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B7(void) {`。
- **L1953 EN**: Returns from the current function with `Q6V64_GET_B7(data)`.
  **L1953 CN**: 以 `Q6V64_GET_B7(data)` 从当前函数返回。
- **L1954 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1954 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1955 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB0(void) {`.
  **L1955 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB0(void) {`。
- **L1956 EN**: Returns from the current function with `Q6V64_GET_UB0(data)`.
  **L1956 CN**: 以 `Q6V64_GET_UB0(data)` 从当前函数返回。
- **L1957 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1957 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1958 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB1(void) {`.
  **L1958 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB1(void) {`。
- **L1959 EN**: Returns from the current function with `Q6V64_GET_UB1(data)`.
  **L1959 CN**: 以 `Q6V64_GET_UB1(data)` 从当前函数返回。
- **L1960 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1960 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1961 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB2(void) {`.
  **L1961 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB2(void) {`。
- **L1962 EN**: Returns from the current function with `Q6V64_GET_UB2(data)`.
  **L1962 CN**: 以 `Q6V64_GET_UB2(data)` 从当前函数返回。
- **L1963 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1963 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1964 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB3(void) {`.
  **L1964 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB3(void) {`。
- **L1965 EN**: Returns from the current function with `Q6V64_GET_UB3(data)`.
  **L1965 CN**: 以 `Q6V64_GET_UB3(data)` 从当前函数返回。
- **L1966 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1966 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1967 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB4(void) {`.
  **L1967 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB4(void) {`。
- **L1968 EN**: Returns from the current function with `Q6V64_GET_UB4(data)`.
  **L1968 CN**: 以 `Q6V64_GET_UB4(data)` 从当前函数返回。

### Lines 1969-1992

````c
  };
  unsigned char UB5(void) {
    return Q6V64_GET_UB5(data);
  };
  unsigned char UB6(void) {
    return Q6V64_GET_UB6(data);
  };
  unsigned char UB7(void) {
    return Q6V64_GET_UB7(data);
  };

  // NOTE: All set methods return a Q6Vect64C type

  // Set doubleword method
  Q6Vect64C D(long long d) {
    return Q6Vect64C(Q6V64_PUT_D(data, d));
  };

  // Set word methods
  Q6Vect64C W0(int w) {
    return Q6Vect64C(Q6V64_PUT_W0(data, w));
  };
  Q6Vect64C W1(int w) {
    return Q6Vect64C(Q6V64_PUT_W1(data, w));
````
- **L1969 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1969 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1970 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB5(void) {`.
  **L1970 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB5(void) {`。
- **L1971 EN**: Returns from the current function with `Q6V64_GET_UB5(data)`.
  **L1971 CN**: 以 `Q6V64_GET_UB5(data)` 从当前函数返回。
- **L1972 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1972 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1973 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB6(void) {`.
  **L1973 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB6(void) {`。
- **L1974 EN**: Returns from the current function with `Q6V64_GET_UB6(data)`.
  **L1974 CN**: 以 `Q6V64_GET_UB6(data)` 从当前函数返回。
- **L1975 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1975 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1976 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB7(void) {`.
  **L1976 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB7(void) {`。
- **L1977 EN**: Returns from the current function with `Q6V64_GET_UB7(data)`.
  **L1977 CN**: 以 `Q6V64_GET_UB7(data)` 从当前函数返回。
- **L1978 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1978 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1979 EN**: Blank line separating nearby declarations or logic blocks.
  **L1979 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1980 EN**: Comment highlights an implementation note: `NOTE: All set methods return a Q6Vect64C type`.
  **L1980 CN**: 注释强调一条实现说明：`NOTE: All set methods return a Q6Vect64C type`。
- **L1981 EN**: Blank line separating nearby declarations or logic blocks.
  **L1981 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1982 EN**: Comment explains nearby logic, constraints, or intent: `Set doubleword method`.
  **L1982 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set doubleword method`。
- **L1983 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect64C D(long long d) {`.
  **L1983 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect64C D(long long d) {`。
- **L1984 EN**: Returns from the current function with `Q6Vect64C(Q6V64_PUT_D(data, d))`.
  **L1984 CN**: 以 `Q6Vect64C(Q6V64_PUT_D(data, d))` 从当前函数返回。
- **L1985 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1985 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1986 EN**: Blank line separating nearby declarations or logic blocks.
  **L1986 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1987 EN**: Comment explains nearby logic, constraints, or intent: `Set word methods`.
  **L1987 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set word methods`。
- **L1988 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect64C W0(int w) {`.
  **L1988 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect64C W0(int w) {`。
- **L1989 EN**: Returns from the current function with `Q6Vect64C(Q6V64_PUT_W0(data, w))`.
  **L1989 CN**: 以 `Q6Vect64C(Q6V64_PUT_W0(data, w))` 从当前函数返回。
- **L1990 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1990 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1991 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect64C W1(int w) {`.
  **L1991 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect64C W1(int w) {`。
- **L1992 EN**: Returns from the current function with `Q6Vect64C(Q6V64_PUT_W1(data, w))`.
  **L1992 CN**: 以 `Q6Vect64C(Q6V64_PUT_W1(data, w))` 从当前函数返回。

### Lines 1993-2016

````c
  };

  // Set half word methods
  Q6Vect64C H0(short h) {
    return Q6Vect64C(Q6V64_PUT_H0(data, h));
  };
  Q6Vect64C H1(short h) {
    return Q6Vect64C(Q6V64_PUT_H1(data, h));
  };
  Q6Vect64C H2(short h) {
    return Q6Vect64C(Q6V64_PUT_H2(data, h));
  };
  Q6Vect64C H3(short h) {
    return Q6Vect64C(Q6V64_PUT_H3(data, h));
  };

  // Set byte methods
  Q6Vect64C b0(signed char b) { return Q6Vect64C(Q6V64_PUT_B0(data, b)); };
  Q6Vect64C B0(signed char b) { return b0(b); };
  Q6Vect64C B1(signed char b) {
    return Q6Vect64C(Q6V64_PUT_B1(data, b));
  };
  Q6Vect64C B2(signed char b) {
    return Q6Vect64C(Q6V64_PUT_B2(data, b));
````
- **L1993 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1993 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1994 EN**: Blank line separating nearby declarations or logic blocks.
  **L1994 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1995 EN**: Comment explains nearby logic, constraints, or intent: `Set half word methods`.
  **L1995 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set half word methods`。
- **L1996 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect64C H0(short h) {`.
  **L1996 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect64C H0(short h) {`。
- **L1997 EN**: Returns from the current function with `Q6Vect64C(Q6V64_PUT_H0(data, h))`.
  **L1997 CN**: 以 `Q6Vect64C(Q6V64_PUT_H0(data, h))` 从当前函数返回。
- **L1998 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1998 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1999 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect64C H1(short h) {`.
  **L1999 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect64C H1(short h) {`。
- **L2000 EN**: Returns from the current function with `Q6Vect64C(Q6V64_PUT_H1(data, h))`.
  **L2000 CN**: 以 `Q6Vect64C(Q6V64_PUT_H1(data, h))` 从当前函数返回。
- **L2001 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2001 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2002 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect64C H2(short h) {`.
  **L2002 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect64C H2(short h) {`。
- **L2003 EN**: Returns from the current function with `Q6Vect64C(Q6V64_PUT_H2(data, h))`.
  **L2003 CN**: 以 `Q6Vect64C(Q6V64_PUT_H2(data, h))` 从当前函数返回。
- **L2004 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2004 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2005 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect64C H3(short h) {`.
  **L2005 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect64C H3(short h) {`。
- **L2006 EN**: Returns from the current function with `Q6Vect64C(Q6V64_PUT_H3(data, h))`.
  **L2006 CN**: 以 `Q6Vect64C(Q6V64_PUT_H3(data, h))` 从当前函数返回。
- **L2007 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2007 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2008 EN**: Blank line separating nearby declarations or logic blocks.
  **L2008 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2009 EN**: Comment explains nearby logic, constraints, or intent: `Set byte methods`.
  **L2009 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set byte methods`。
- **L2010 EN**: Executes a call or declaration centered on `b0`.
  **L2010 CN**: 执行以 `b0` 为核心的调用或声明。
- **L2011 EN**: Executes a call or declaration centered on `B0`.
  **L2011 CN**: 执行以 `B0` 为核心的调用或声明。
- **L2012 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect64C B1(signed char b) {`.
  **L2012 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect64C B1(signed char b) {`。
- **L2013 EN**: Returns from the current function with `Q6Vect64C(Q6V64_PUT_B1(data, b))`.
  **L2013 CN**: 以 `Q6Vect64C(Q6V64_PUT_B1(data, b))` 从当前函数返回。
- **L2014 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2014 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2015 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect64C B2(signed char b) {`.
  **L2015 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect64C B2(signed char b) {`。
- **L2016 EN**: Returns from the current function with `Q6Vect64C(Q6V64_PUT_B2(data, b))`.
  **L2016 CN**: 以 `Q6Vect64C(Q6V64_PUT_B2(data, b))` 从当前函数返回。

### Lines 2017-2040

````c
  };
  Q6Vect64C B3(signed char b) {
    return Q6Vect64C(Q6V64_PUT_B3(data, b));
  };
  Q6Vect64C B4(signed char b) {
    return Q6Vect64C(Q6V64_PUT_B4(data, b));
  };
  Q6Vect64C B5(signed char b) {
    return Q6Vect64C(Q6V64_PUT_B5(data, b));
  };
  Q6Vect64C B6(signed char b) {
    return Q6Vect64C(Q6V64_PUT_B6(data, b));
  };
  Q6Vect64C B7(signed char b) {
    return Q6Vect64C(Q6V64_PUT_B7(data, b));
  };

private:
  long long data;
};

#endif /* __cplusplus */

/* 32 Bit Vectors */
````
- **L2017 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2017 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2018 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect64C B3(signed char b) {`.
  **L2018 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect64C B3(signed char b) {`。
- **L2019 EN**: Returns from the current function with `Q6Vect64C(Q6V64_PUT_B3(data, b))`.
  **L2019 CN**: 以 `Q6Vect64C(Q6V64_PUT_B3(data, b))` 从当前函数返回。
- **L2020 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2020 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2021 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect64C B4(signed char b) {`.
  **L2021 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect64C B4(signed char b) {`。
- **L2022 EN**: Returns from the current function with `Q6Vect64C(Q6V64_PUT_B4(data, b))`.
  **L2022 CN**: 以 `Q6Vect64C(Q6V64_PUT_B4(data, b))` 从当前函数返回。
- **L2023 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2023 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2024 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect64C B5(signed char b) {`.
  **L2024 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect64C B5(signed char b) {`。
- **L2025 EN**: Returns from the current function with `Q6Vect64C(Q6V64_PUT_B5(data, b))`.
  **L2025 CN**: 以 `Q6Vect64C(Q6V64_PUT_B5(data, b))` 从当前函数返回。
- **L2026 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2026 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2027 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect64C B6(signed char b) {`.
  **L2027 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect64C B6(signed char b) {`。
- **L2028 EN**: Returns from the current function with `Q6Vect64C(Q6V64_PUT_B6(data, b))`.
  **L2028 CN**: 以 `Q6Vect64C(Q6V64_PUT_B6(data, b))` 从当前函数返回。
- **L2029 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2029 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2030 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect64C B7(signed char b) {`.
  **L2030 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect64C B7(signed char b) {`。
- **L2031 EN**: Returns from the current function with `Q6Vect64C(Q6V64_PUT_B7(data, b))`.
  **L2031 CN**: 以 `Q6Vect64C(Q6V64_PUT_B7(data, b))` 从当前函数返回。
- **L2032 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2032 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2033 EN**: Blank line separating nearby declarations or logic blocks.
  **L2033 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2034 EN**: Sets the access level for following class members to `private`.
  **L2034 CN**: 将后续类成员的访问级别设为 `private`。
- **L2035 EN**: Adds a standalone statement or declaration: `long long data;`.
  **L2035 CN**: 添加一条独立语句或声明：`long long data;`。
- **L2036 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2036 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2037 EN**: Blank line separating nearby declarations or logic blocks.
  **L2037 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2038 EN**: Closes the current preprocessor conditional block.
  **L2038 CN**: 结束当前预处理条件块。
- **L2039 EN**: Blank line separating nearby declarations or logic blocks.
  **L2039 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2040 EN**: Comment explains nearby logic, constraints, or intent: `32 Bit Vectors`.
  **L2040 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32 Bit Vectors`。

### Lines 2041-2064

````c

typedef int Q6Vect32;

/* Extract word macros */

#define Q6V32_GET_W(v) (v)
#define Q6V32_GET_UW(v) ((unsigned int)(v))

/* Extract half word macros */

#define Q6V32_GET_H0(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      short h[2];                                                              \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.h[0];                                                \
  })
#define Q6V32_GET_H1(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      short h[2];                                                              \
````
- **L2041 EN**: Blank line separating nearby declarations or logic blocks.
  **L2041 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2042 EN**: Introduces an alias or helper declaration: `typedef int Q6Vect32;`.
  **L2042 CN**: 引入一条别名或辅助声明：`typedef int Q6Vect32;`。
- **L2043 EN**: Blank line separating nearby declarations or logic blocks.
  **L2043 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2044 EN**: Comment explains nearby logic, constraints, or intent: `Extract word macros`.
  **L2044 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract word macros`。
- **L2045 EN**: Blank line separating nearby declarations or logic blocks.
  **L2045 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2046 EN**: Defines macro `Q6V32_GET_W(v)` for conditional compilation, shorthand, or API generation.
  **L2046 CN**: 定义宏 `Q6V32_GET_W(v)`，用于条件编译、简写或 API 生成。
- **L2047 EN**: Defines macro `Q6V32_GET_UW(v)` for conditional compilation, shorthand, or API generation.
  **L2047 CN**: 定义宏 `Q6V32_GET_UW(v)`，用于条件编译、简写或 API 生成。
- **L2048 EN**: Blank line separating nearby declarations or logic blocks.
  **L2048 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2049 EN**: Comment explains nearby logic, constraints, or intent: `Extract half word macros`.
  **L2049 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract half word macros`。
- **L2050 EN**: Blank line separating nearby declarations or logic blocks.
  **L2050 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2051 EN**: Defines macro `Q6V32_GET_H0(v)` for conditional compilation, shorthand, or API generation.
  **L2051 CN**: 定义宏 `Q6V32_GET_H0(v)`，用于条件编译、简写或 API 生成。
- **L2052 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2052 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2053 EN**: Declares union `union`.
  **L2053 CN**: 声明 union `union`。
- **L2054 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2054 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2055 EN**: Continues the surrounding expression or declaration: `short h[2];                                                              \`.
  **L2055 CN**: 继续构造周围的表达式或声明：`short h[2];                                                              \`。
- **L2056 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2056 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2057 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2057 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2058 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.h[0];                                                \`.
  **L2058 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.h[0];                                                \`。
- **L2059 EN**: Continues the surrounding expression or declaration: `})`.
  **L2059 CN**: 继续构造周围的表达式或声明：`})`。
- **L2060 EN**: Defines macro `Q6V32_GET_H1(v)` for conditional compilation, shorthand, or API generation.
  **L2060 CN**: 定义宏 `Q6V32_GET_H1(v)`，用于条件编译、简写或 API 生成。
- **L2061 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2061 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2062 EN**: Declares union `union`.
  **L2062 CN**: 声明 union `union`。
- **L2063 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2063 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2064 EN**: Continues the surrounding expression or declaration: `short h[2];                                                              \`.
  **L2064 CN**: 继续构造周围的表达式或声明：`short h[2];                                                              \`。

### Lines 2065-2088

````c
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.h[1];                                                \
  })
#define Q6V32_GET_UH0(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      unsigned short uh[2];                                                    \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.uh[0];                                               \
  })
#define Q6V32_GET_UH1(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      unsigned short uh[2];                                                    \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.uh[1];                                               \
  })

/* Extract byte macros */
````
- **L2065 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2065 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2066 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2066 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2067 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.h[1];                                                \`.
  **L2067 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.h[1];                                                \`。
- **L2068 EN**: Continues the surrounding expression or declaration: `})`.
  **L2068 CN**: 继续构造周围的表达式或声明：`})`。
- **L2069 EN**: Defines macro `Q6V32_GET_UH0(v)` for conditional compilation, shorthand, or API generation.
  **L2069 CN**: 定义宏 `Q6V32_GET_UH0(v)`，用于条件编译、简写或 API 生成。
- **L2070 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2070 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2071 EN**: Declares union `union`.
  **L2071 CN**: 声明 union `union`。
- **L2072 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2072 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2073 EN**: Continues the surrounding expression or declaration: `unsigned short uh[2];                                                    \`.
  **L2073 CN**: 继续构造周围的表达式或声明：`unsigned short uh[2];                                                    \`。
- **L2074 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2074 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2075 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2075 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2076 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.uh[0];                                               \`.
  **L2076 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.uh[0];                                               \`。
- **L2077 EN**: Continues the surrounding expression or declaration: `})`.
  **L2077 CN**: 继续构造周围的表达式或声明：`})`。
- **L2078 EN**: Defines macro `Q6V32_GET_UH1(v)` for conditional compilation, shorthand, or API generation.
  **L2078 CN**: 定义宏 `Q6V32_GET_UH1(v)`，用于条件编译、简写或 API 生成。
- **L2079 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2079 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2080 EN**: Declares union `union`.
  **L2080 CN**: 声明 union `union`。
- **L2081 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2081 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2082 EN**: Continues the surrounding expression or declaration: `unsigned short uh[2];                                                    \`.
  **L2082 CN**: 继续构造周围的表达式或声明：`unsigned short uh[2];                                                    \`。
- **L2083 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2083 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2084 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2084 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2085 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.uh[1];                                               \`.
  **L2085 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.uh[1];                                               \`。
- **L2086 EN**: Continues the surrounding expression or declaration: `})`.
  **L2086 CN**: 继续构造周围的表达式或声明：`})`。
- **L2087 EN**: Blank line separating nearby declarations or logic blocks.
  **L2087 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2088 EN**: Comment explains nearby logic, constraints, or intent: `Extract byte macros`.
  **L2088 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract byte macros`。

### Lines 2089-2112

````c

#define Q6V32_GET_B0(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      signed char b[4];                                                        \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.b[0];                                                \
  })
#define Q6V32_GET_B1(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      signed char b[4];                                                        \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.b[1];                                                \
  })
#define Q6V32_GET_B2(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      signed char b[4];                                                        \
````
- **L2089 EN**: Blank line separating nearby declarations or logic blocks.
  **L2089 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2090 EN**: Defines macro `Q6V32_GET_B0(v)` for conditional compilation, shorthand, or API generation.
  **L2090 CN**: 定义宏 `Q6V32_GET_B0(v)`，用于条件编译、简写或 API 生成。
- **L2091 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2091 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2092 EN**: Declares union `union`.
  **L2092 CN**: 声明 union `union`。
- **L2093 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2093 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2094 EN**: Continues the surrounding expression or declaration: `signed char b[4];                                                        \`.
  **L2094 CN**: 继续构造周围的表达式或声明：`signed char b[4];                                                        \`。
- **L2095 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2095 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2096 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2096 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2097 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.b[0];                                                \`.
  **L2097 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.b[0];                                                \`。
- **L2098 EN**: Continues the surrounding expression or declaration: `})`.
  **L2098 CN**: 继续构造周围的表达式或声明：`})`。
- **L2099 EN**: Defines macro `Q6V32_GET_B1(v)` for conditional compilation, shorthand, or API generation.
  **L2099 CN**: 定义宏 `Q6V32_GET_B1(v)`，用于条件编译、简写或 API 生成。
- **L2100 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2100 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2101 EN**: Declares union `union`.
  **L2101 CN**: 声明 union `union`。
- **L2102 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2102 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2103 EN**: Continues the surrounding expression or declaration: `signed char b[4];                                                        \`.
  **L2103 CN**: 继续构造周围的表达式或声明：`signed char b[4];                                                        \`。
- **L2104 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2104 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2105 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2105 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2106 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.b[1];                                                \`.
  **L2106 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.b[1];                                                \`。
- **L2107 EN**: Continues the surrounding expression or declaration: `})`.
  **L2107 CN**: 继续构造周围的表达式或声明：`})`。
- **L2108 EN**: Defines macro `Q6V32_GET_B2(v)` for conditional compilation, shorthand, or API generation.
  **L2108 CN**: 定义宏 `Q6V32_GET_B2(v)`，用于条件编译、简写或 API 生成。
- **L2109 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2109 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2110 EN**: Declares union `union`.
  **L2110 CN**: 声明 union `union`。
- **L2111 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2111 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2112 EN**: Continues the surrounding expression or declaration: `signed char b[4];                                                        \`.
  **L2112 CN**: 继续构造周围的表达式或声明：`signed char b[4];                                                        \`。

### Lines 2113-2136

````c
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.b[2];                                                \
  })
#define Q6V32_GET_B3(v)                                                        \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      signed char b[4];                                                        \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.b[3];                                                \
  })
#define Q6V32_GET_UB0(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      unsigned char ub[4];                                                     \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.ub[0];                                               \
  })
#define Q6V32_GET_UB1(v)                                                       \
  __extension__({                                                              \
````
- **L2113 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2113 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2114 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2114 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2115 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.b[2];                                                \`.
  **L2115 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.b[2];                                                \`。
- **L2116 EN**: Continues the surrounding expression or declaration: `})`.
  **L2116 CN**: 继续构造周围的表达式或声明：`})`。
- **L2117 EN**: Defines macro `Q6V32_GET_B3(v)` for conditional compilation, shorthand, or API generation.
  **L2117 CN**: 定义宏 `Q6V32_GET_B3(v)`，用于条件编译、简写或 API 生成。
- **L2118 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2118 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2119 EN**: Declares union `union`.
  **L2119 CN**: 声明 union `union`。
- **L2120 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2120 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2121 EN**: Continues the surrounding expression or declaration: `signed char b[4];                                                        \`.
  **L2121 CN**: 继续构造周围的表达式或声明：`signed char b[4];                                                        \`。
- **L2122 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2122 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2123 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2123 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2124 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.b[3];                                                \`.
  **L2124 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.b[3];                                                \`。
- **L2125 EN**: Continues the surrounding expression or declaration: `})`.
  **L2125 CN**: 继续构造周围的表达式或声明：`})`。
- **L2126 EN**: Defines macro `Q6V32_GET_UB0(v)` for conditional compilation, shorthand, or API generation.
  **L2126 CN**: 定义宏 `Q6V32_GET_UB0(v)`，用于条件编译、简写或 API 生成。
- **L2127 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2127 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2128 EN**: Declares union `union`.
  **L2128 CN**: 声明 union `union`。
- **L2129 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2129 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2130 EN**: Continues the surrounding expression or declaration: `unsigned char ub[4];                                                     \`.
  **L2130 CN**: 继续构造周围的表达式或声明：`unsigned char ub[4];                                                     \`。
- **L2131 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2131 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2132 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2132 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2133 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.ub[0];                                               \`.
  **L2133 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.ub[0];                                               \`。
- **L2134 EN**: Continues the surrounding expression or declaration: `})`.
  **L2134 CN**: 继续构造周围的表达式或声明：`})`。
- **L2135 EN**: Defines macro `Q6V32_GET_UB1(v)` for conditional compilation, shorthand, or API generation.
  **L2135 CN**: 定义宏 `Q6V32_GET_UB1(v)`，用于条件编译、简写或 API 生成。
- **L2136 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2136 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。

### Lines 2137-2160

````c
    union {                                                                    \
      int w;                                                                   \
      unsigned char ub[4];                                                     \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.ub[1];                                               \
  })
#define Q6V32_GET_UB2(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      unsigned char ub[4];                                                     \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.ub[2];                                               \
  })
#define Q6V32_GET_UB3(v)                                                       \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      unsigned char ub[4];                                                     \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.ub[3];                                               \
````
- **L2137 EN**: Declares union `union`.
  **L2137 CN**: 声明 union `union`。
- **L2138 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2138 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2139 EN**: Continues the surrounding expression or declaration: `unsigned char ub[4];                                                     \`.
  **L2139 CN**: 继续构造周围的表达式或声明：`unsigned char ub[4];                                                     \`。
- **L2140 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2140 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2141 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2141 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2142 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.ub[1];                                               \`.
  **L2142 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.ub[1];                                               \`。
- **L2143 EN**: Continues the surrounding expression or declaration: `})`.
  **L2143 CN**: 继续构造周围的表达式或声明：`})`。
- **L2144 EN**: Defines macro `Q6V32_GET_UB2(v)` for conditional compilation, shorthand, or API generation.
  **L2144 CN**: 定义宏 `Q6V32_GET_UB2(v)`，用于条件编译、简写或 API 生成。
- **L2145 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2145 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2146 EN**: Declares union `union`.
  **L2146 CN**: 声明 union `union`。
- **L2147 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2147 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2148 EN**: Continues the surrounding expression or declaration: `unsigned char ub[4];                                                     \`.
  **L2148 CN**: 继续构造周围的表达式或声明：`unsigned char ub[4];                                                     \`。
- **L2149 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2149 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2150 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2150 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2151 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.ub[2];                                               \`.
  **L2151 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.ub[2];                                               \`。
- **L2152 EN**: Continues the surrounding expression or declaration: `})`.
  **L2152 CN**: 继续构造周围的表达式或声明：`})`。
- **L2153 EN**: Defines macro `Q6V32_GET_UB3(v)` for conditional compilation, shorthand, or API generation.
  **L2153 CN**: 定义宏 `Q6V32_GET_UB3(v)`，用于条件编译、简写或 API 生成。
- **L2154 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2154 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2155 EN**: Declares union `union`.
  **L2155 CN**: 声明 union `union`。
- **L2156 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2156 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2157 EN**: Continues the surrounding expression or declaration: `unsigned char ub[4];                                                     \`.
  **L2157 CN**: 继续构造周围的表达式或声明：`unsigned char ub[4];                                                     \`。
- **L2158 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2158 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2159 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2159 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2160 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.ub[3];                                               \`.
  **L2160 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.ub[3];                                               \`。

### Lines 2161-2184

````c
  })

/* NOTE: All set macros return a Q6Vect32 type */

/* Set word macro */

#define Q6V32_PUT_W(v, new) (new)

/* Set half word macros */

#ifdef __qdsp6__

#define Q6V32_PUT_H0(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      short h[2];                                                              \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.h[0] = (new);                                        \
    _Q6V32_internal_union.w;                                                   \
  })
#define Q6V32_PUT_H1(v, new)                                                   \
  __extension__({                                                              \
````
- **L2161 EN**: Continues the surrounding expression or declaration: `})`.
  **L2161 CN**: 继续构造周围的表达式或声明：`})`。
- **L2162 EN**: Blank line separating nearby declarations or logic blocks.
  **L2162 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2163 EN**: Comment highlights an implementation note: `NOTE: All set macros return a Q6Vect32 type`.
  **L2163 CN**: 注释强调一条实现说明：`NOTE: All set macros return a Q6Vect32 type`。
- **L2164 EN**: Blank line separating nearby declarations or logic blocks.
  **L2164 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2165 EN**: Comment explains nearby logic, constraints, or intent: `Set word macro`.
  **L2165 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set word macro`。
- **L2166 EN**: Blank line separating nearby declarations or logic blocks.
  **L2166 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2167 EN**: Defines macro `Q6V32_PUT_W(v, new)` for conditional compilation, shorthand, or API generation.
  **L2167 CN**: 定义宏 `Q6V32_PUT_W(v, new)`，用于条件编译、简写或 API 生成。
- **L2168 EN**: Blank line separating nearby declarations or logic blocks.
  **L2168 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2169 EN**: Comment explains nearby logic, constraints, or intent: `Set half word macros`.
  **L2169 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set half word macros`。
- **L2170 EN**: Blank line separating nearby declarations or logic blocks.
  **L2170 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2171 EN**: Starts a preprocessor conditional block: `#ifdef __qdsp6__`.
  **L2171 CN**: 开始一个预处理条件块：`#ifdef __qdsp6__`。
- **L2172 EN**: Blank line separating nearby declarations or logic blocks.
  **L2172 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2173 EN**: Defines macro `Q6V32_PUT_H0(v, new)` for conditional compilation, shorthand, or API generation.
  **L2173 CN**: 定义宏 `Q6V32_PUT_H0(v, new)`，用于条件编译、简写或 API 生成。
- **L2174 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2174 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2175 EN**: Declares union `union`.
  **L2175 CN**: 声明 union `union`。
- **L2176 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2176 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2177 EN**: Continues the surrounding expression or declaration: `short h[2];                                                              \`.
  **L2177 CN**: 继续构造周围的表达式或声明：`short h[2];                                                              \`。
- **L2178 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2178 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2179 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2179 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2180 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.h[0] = (new);                                        \`.
  **L2180 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.h[0] = (new);                                        \`。
- **L2181 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w;                                                   \`.
  **L2181 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w;                                                   \`。
- **L2182 EN**: Continues the surrounding expression or declaration: `})`.
  **L2182 CN**: 继续构造周围的表达式或声明：`})`。
- **L2183 EN**: Defines macro `Q6V32_PUT_H1(v, new)` for conditional compilation, shorthand, or API generation.
  **L2183 CN**: 定义宏 `Q6V32_PUT_H1(v, new)`，用于条件编译、简写或 API 生成。
- **L2184 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2184 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。

### Lines 2185-2208

````c
    union {                                                                    \
      int w;                                                                   \
      short h[2];                                                              \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.h[1] = (new);                                        \
    _Q6V32_internal_union.w;                                                   \
  })

#else /* !__qdsp6__ */

#define Q6V32_PUT_H0(v, new)                                                   \
  (((v) & 0xffff0000) | ((Q6Vect32)((unsigned short)(new))))
#define Q6V32_PUT_H1(v, new) (((v) & 0x0000ffff) | (((Q6Vect32)(new)) << 16))

#endif /* !__qdsp6__ */

/* Set byte macros */

#ifdef __qdsp6__

#define Q6V32_PUT_B0(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
````
- **L2185 EN**: Declares union `union`.
  **L2185 CN**: 声明 union `union`。
- **L2186 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2186 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2187 EN**: Continues the surrounding expression or declaration: `short h[2];                                                              \`.
  **L2187 CN**: 继续构造周围的表达式或声明：`short h[2];                                                              \`。
- **L2188 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2188 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2189 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2189 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2190 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.h[1] = (new);                                        \`.
  **L2190 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.h[1] = (new);                                        \`。
- **L2191 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w;                                                   \`.
  **L2191 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w;                                                   \`。
- **L2192 EN**: Continues the surrounding expression or declaration: `})`.
  **L2192 CN**: 继续构造周围的表达式或声明：`})`。
- **L2193 EN**: Blank line separating nearby declarations or logic blocks.
  **L2193 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2194 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L2194 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L2195 EN**: Blank line separating nearby declarations or logic blocks.
  **L2195 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2196 EN**: Defines macro `Q6V32_PUT_H0(v, new)` for conditional compilation, shorthand, or API generation.
  **L2196 CN**: 定义宏 `Q6V32_PUT_H0(v, new)`，用于条件编译、简写或 API 生成。
- **L2197 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffff0000) | ((Q6Vect32)((unsigned short)(new))))`.
  **L2197 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffff0000) | ((Q6Vect32)((unsigned short)(new))))`。
- **L2198 EN**: Defines macro `Q6V32_PUT_H1(v, new)` for conditional compilation, shorthand, or API generation.
  **L2198 CN**: 定义宏 `Q6V32_PUT_H1(v, new)`，用于条件编译、简写或 API 生成。
- **L2199 EN**: Blank line separating nearby declarations or logic blocks.
  **L2199 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2200 EN**: Closes the current preprocessor conditional block.
  **L2200 CN**: 结束当前预处理条件块。
- **L2201 EN**: Blank line separating nearby declarations or logic blocks.
  **L2201 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2202 EN**: Comment explains nearby logic, constraints, or intent: `Set byte macros`.
  **L2202 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set byte macros`。
- **L2203 EN**: Blank line separating nearby declarations or logic blocks.
  **L2203 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2204 EN**: Starts a preprocessor conditional block: `#ifdef __qdsp6__`.
  **L2204 CN**: 开始一个预处理条件块：`#ifdef __qdsp6__`。
- **L2205 EN**: Blank line separating nearby declarations or logic blocks.
  **L2205 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2206 EN**: Defines macro `Q6V32_PUT_B0(v, new)` for conditional compilation, shorthand, or API generation.
  **L2206 CN**: 定义宏 `Q6V32_PUT_B0(v, new)`，用于条件编译、简写或 API 生成。
- **L2207 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2207 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2208 EN**: Declares union `union`.
  **L2208 CN**: 声明 union `union`。

### Lines 2209-2232

````c
      int w;                                                                   \
      char b[4];                                                               \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.b[0] = (new);                                        \
    _Q6V32_internal_union.w;                                                   \
  })
#define Q6V32_PUT_B1(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      char b[4];                                                               \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.b[1] = (new);                                        \
    _Q6V32_internal_union.w;                                                   \
  })
#define Q6V32_PUT_B2(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      char b[4];                                                               \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
````
- **L2209 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2209 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2210 EN**: Continues the surrounding expression or declaration: `char b[4];                                                               \`.
  **L2210 CN**: 继续构造周围的表达式或声明：`char b[4];                                                               \`。
- **L2211 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2211 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2212 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2212 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2213 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.b[0] = (new);                                        \`.
  **L2213 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.b[0] = (new);                                        \`。
- **L2214 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w;                                                   \`.
  **L2214 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w;                                                   \`。
- **L2215 EN**: Continues the surrounding expression or declaration: `})`.
  **L2215 CN**: 继续构造周围的表达式或声明：`})`。
- **L2216 EN**: Defines macro `Q6V32_PUT_B1(v, new)` for conditional compilation, shorthand, or API generation.
  **L2216 CN**: 定义宏 `Q6V32_PUT_B1(v, new)`，用于条件编译、简写或 API 生成。
- **L2217 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2217 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2218 EN**: Declares union `union`.
  **L2218 CN**: 声明 union `union`。
- **L2219 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2219 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2220 EN**: Continues the surrounding expression or declaration: `char b[4];                                                               \`.
  **L2220 CN**: 继续构造周围的表达式或声明：`char b[4];                                                               \`。
- **L2221 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2221 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2222 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2222 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2223 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.b[1] = (new);                                        \`.
  **L2223 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.b[1] = (new);                                        \`。
- **L2224 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w;                                                   \`.
  **L2224 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w;                                                   \`。
- **L2225 EN**: Continues the surrounding expression or declaration: `})`.
  **L2225 CN**: 继续构造周围的表达式或声明：`})`。
- **L2226 EN**: Defines macro `Q6V32_PUT_B2(v, new)` for conditional compilation, shorthand, or API generation.
  **L2226 CN**: 定义宏 `Q6V32_PUT_B2(v, new)`，用于条件编译、简写或 API 生成。
- **L2227 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2227 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2228 EN**: Declares union `union`.
  **L2228 CN**: 声明 union `union`。
- **L2229 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2229 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2230 EN**: Continues the surrounding expression or declaration: `char b[4];                                                               \`.
  **L2230 CN**: 继续构造周围的表达式或声明：`char b[4];                                                               \`。
- **L2231 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2231 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2232 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2232 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。

### Lines 2233-2256

````c
    _Q6V32_internal_union.b[2] = (new);                                        \
    _Q6V32_internal_union.w;                                                   \
  })
#define Q6V32_PUT_B3(v, new)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      int w;                                                                   \
      char b[4];                                                               \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.w = (v);                                             \
    _Q6V32_internal_union.b[3] = (new);                                        \
    _Q6V32_internal_union.w;                                                   \
  })

#else /* !__qdsp6__ */

#define Q6V32_PUT_B0(v, new)                                                   \
  (((v) & 0xffffff00) | ((Q6Vect32)((unsigned char)(new))))
#define Q6V32_PUT_B1(v, new)                                                   \
  (((v) & 0xffff00ff) | (((Q6Vect32)((unsigned char)(new))) << 8))
#define Q6V32_PUT_B2(v, new)                                                   \
  (((v) & 0xff00ffff) | (((Q6Vect32)((unsigned char)(new))) << 16))
#define Q6V32_PUT_B3(v, new) (((v) & 0x00ffffff) | (((Q6Vect32)(new)) << 24))

````
- **L2233 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.b[2] = (new);                                        \`.
  **L2233 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.b[2] = (new);                                        \`。
- **L2234 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w;                                                   \`.
  **L2234 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w;                                                   \`。
- **L2235 EN**: Continues the surrounding expression or declaration: `})`.
  **L2235 CN**: 继续构造周围的表达式或声明：`})`。
- **L2236 EN**: Defines macro `Q6V32_PUT_B3(v, new)` for conditional compilation, shorthand, or API generation.
  **L2236 CN**: 定义宏 `Q6V32_PUT_B3(v, new)`，用于条件编译、简写或 API 生成。
- **L2237 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2237 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2238 EN**: Declares union `union`.
  **L2238 CN**: 声明 union `union`。
- **L2239 EN**: Continues the surrounding expression or declaration: `int w;                                                                   \`.
  **L2239 CN**: 继续构造周围的表达式或声明：`int w;                                                                   \`。
- **L2240 EN**: Continues the surrounding expression or declaration: `char b[4];                                                               \`.
  **L2240 CN**: 继续构造周围的表达式或声明：`char b[4];                                                               \`。
- **L2241 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2241 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2242 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w = (v);                                             \`.
  **L2242 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w = (v);                                             \`。
- **L2243 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.b[3] = (new);                                        \`.
  **L2243 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.b[3] = (new);                                        \`。
- **L2244 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.w;                                                   \`.
  **L2244 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.w;                                                   \`。
- **L2245 EN**: Continues the surrounding expression or declaration: `})`.
  **L2245 CN**: 继续构造周围的表达式或声明：`})`。
- **L2246 EN**: Blank line separating nearby declarations or logic blocks.
  **L2246 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2247 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L2247 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L2248 EN**: Blank line separating nearby declarations or logic blocks.
  **L2248 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2249 EN**: Defines macro `Q6V32_PUT_B0(v, new)` for conditional compilation, shorthand, or API generation.
  **L2249 CN**: 定义宏 `Q6V32_PUT_B0(v, new)`，用于条件编译、简写或 API 生成。
- **L2250 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffffff00) | ((Q6Vect32)((unsigned char)(new))))`.
  **L2250 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffffff00) | ((Q6Vect32)((unsigned char)(new))))`。
- **L2251 EN**: Defines macro `Q6V32_PUT_B1(v, new)` for conditional compilation, shorthand, or API generation.
  **L2251 CN**: 定义宏 `Q6V32_PUT_B1(v, new)`，用于条件编译、简写或 API 生成。
- **L2252 EN**: Continues the surrounding expression or declaration: `(((v) & 0xffff00ff) | (((Q6Vect32)((unsigned char)(new))) << 8))`.
  **L2252 CN**: 继续构造周围的表达式或声明：`(((v) & 0xffff00ff) | (((Q6Vect32)((unsigned char)(new))) << 8))`。
- **L2253 EN**: Defines macro `Q6V32_PUT_B2(v, new)` for conditional compilation, shorthand, or API generation.
  **L2253 CN**: 定义宏 `Q6V32_PUT_B2(v, new)`，用于条件编译、简写或 API 生成。
- **L2254 EN**: Continues the surrounding expression or declaration: `(((v) & 0xff00ffff) | (((Q6Vect32)((unsigned char)(new))) << 16))`.
  **L2254 CN**: 继续构造周围的表达式或声明：`(((v) & 0xff00ffff) | (((Q6Vect32)((unsigned char)(new))) << 16))`。
- **L2255 EN**: Defines macro `Q6V32_PUT_B3(v, new)` for conditional compilation, shorthand, or API generation.
  **L2255 CN**: 定义宏 `Q6V32_PUT_B3(v, new)`，用于条件编译、简写或 API 生成。
- **L2256 EN**: Blank line separating nearby declarations or logic blocks.
  **L2256 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2257-2280

````c
#endif /* !__qdsp6__ */

/* NOTE: All create macros return a Q6Vect32 type */

/* Create from a word */

#define Q6V32_CREATE_W(w) (w)

/* Create from half words */

#ifdef __qdsp6__

#define Q6V32_CREATE_H(h1, h0)                                                 \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      short h[2];                                                              \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.h[0] = (h0);                                         \
    _Q6V32_internal_union.h[1] = (h1);                                         \
    _Q6V32_internal_union.d;                                                   \
  })

#else /* !__qdsp6__ */
````
- **L2257 EN**: Closes the current preprocessor conditional block.
  **L2257 CN**: 结束当前预处理条件块。
- **L2258 EN**: Blank line separating nearby declarations or logic blocks.
  **L2258 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2259 EN**: Comment highlights an implementation note: `NOTE: All create macros return a Q6Vect32 type`.
  **L2259 CN**: 注释强调一条实现说明：`NOTE: All create macros return a Q6Vect32 type`。
- **L2260 EN**: Blank line separating nearby declarations or logic blocks.
  **L2260 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2261 EN**: Comment explains nearby logic, constraints, or intent: `Create from a word`.
  **L2261 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create from a word`。
- **L2262 EN**: Blank line separating nearby declarations or logic blocks.
  **L2262 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2263 EN**: Defines macro `Q6V32_CREATE_W(w)` for conditional compilation, shorthand, or API generation.
  **L2263 CN**: 定义宏 `Q6V32_CREATE_W(w)`，用于条件编译、简写或 API 生成。
- **L2264 EN**: Blank line separating nearby declarations or logic blocks.
  **L2264 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2265 EN**: Comment explains nearby logic, constraints, or intent: `Create from half words`.
  **L2265 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create from half words`。
- **L2266 EN**: Blank line separating nearby declarations or logic blocks.
  **L2266 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2267 EN**: Starts a preprocessor conditional block: `#ifdef __qdsp6__`.
  **L2267 CN**: 开始一个预处理条件块：`#ifdef __qdsp6__`。
- **L2268 EN**: Blank line separating nearby declarations or logic blocks.
  **L2268 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2269 EN**: Defines macro `Q6V32_CREATE_H(h1, h0)` for conditional compilation, shorthand, or API generation.
  **L2269 CN**: 定义宏 `Q6V32_CREATE_H(h1, h0)`，用于条件编译、简写或 API 生成。
- **L2270 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2270 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2271 EN**: Declares union `union`.
  **L2271 CN**: 声明 union `union`。
- **L2272 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L2272 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L2273 EN**: Continues the surrounding expression or declaration: `short h[2];                                                              \`.
  **L2273 CN**: 继续构造周围的表达式或声明：`short h[2];                                                              \`。
- **L2274 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2274 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2275 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.h[0] = (h0);                                         \`.
  **L2275 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.h[0] = (h0);                                         \`。
- **L2276 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.h[1] = (h1);                                         \`.
  **L2276 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.h[1] = (h1);                                         \`。
- **L2277 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.d;                                                   \`.
  **L2277 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.d;                                                   \`。
- **L2278 EN**: Continues the surrounding expression or declaration: `})`.
  **L2278 CN**: 继续构造周围的表达式或声明：`})`。
- **L2279 EN**: Blank line separating nearby declarations or logic blocks.
  **L2279 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2280 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L2280 CN**: 继续为当前目标或特性集合选择预处理分支。

### Lines 2281-2304

````c

#define Q6V32_CREATE_H(h1, h0)                                                 \
  ((((Q6Vect32)(h1)) << 16) | ((Q6Vect32)((h0) & 0xffff)))

#endif /* !__qdsp6__ */

/* Create from bytes */
#ifdef __qdsp6__

#define Q6V32_CREATE_B(b3, b2, b1, b0)                                         \
  __extension__({                                                              \
    union {                                                                    \
      long long d;                                                             \
      char b[4];                                                               \
    } _Q6V32_internal_union;                                                   \
    _Q6V32_internal_union.b[0] = (b0);                                         \
    _Q6V32_internal_union.b[1] = (b1);                                         \
    _Q6V32_internal_union.b[2] = (b2);                                         \
    _Q6V32_internal_union.b[3] = (b3);                                         \
    _Q6V32_internal_union.d;                                                   \
  })

#else /* !__qdsp6__ */

````
- **L2281 EN**: Blank line separating nearby declarations or logic blocks.
  **L2281 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2282 EN**: Defines macro `Q6V32_CREATE_H(h1, h0)` for conditional compilation, shorthand, or API generation.
  **L2282 CN**: 定义宏 `Q6V32_CREATE_H(h1, h0)`，用于条件编译、简写或 API 生成。
- **L2283 EN**: Continues the surrounding expression or declaration: `((((Q6Vect32)(h1)) << 16) | ((Q6Vect32)((h0) & 0xffff)))`.
  **L2283 CN**: 继续构造周围的表达式或声明：`((((Q6Vect32)(h1)) << 16) | ((Q6Vect32)((h0) & 0xffff)))`。
- **L2284 EN**: Blank line separating nearby declarations or logic blocks.
  **L2284 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2285 EN**: Closes the current preprocessor conditional block.
  **L2285 CN**: 结束当前预处理条件块。
- **L2286 EN**: Blank line separating nearby declarations or logic blocks.
  **L2286 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2287 EN**: Comment explains nearby logic, constraints, or intent: `Create from bytes`.
  **L2287 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Create from bytes`。
- **L2288 EN**: Starts a preprocessor conditional block: `#ifdef __qdsp6__`.
  **L2288 CN**: 开始一个预处理条件块：`#ifdef __qdsp6__`。
- **L2289 EN**: Blank line separating nearby declarations or logic blocks.
  **L2289 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2290 EN**: Defines macro `Q6V32_CREATE_B(b3, b2, b1, b0)` for conditional compilation, shorthand, or API generation.
  **L2290 CN**: 定义宏 `Q6V32_CREATE_B(b3, b2, b1, b0)`，用于条件编译、简写或 API 生成。
- **L2291 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2291 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2292 EN**: Declares union `union`.
  **L2292 CN**: 声明 union `union`。
- **L2293 EN**: Continues the surrounding expression or declaration: `long long d;                                                             \`.
  **L2293 CN**: 继续构造周围的表达式或声明：`long long d;                                                             \`。
- **L2294 EN**: Continues the surrounding expression or declaration: `char b[4];                                                               \`.
  **L2294 CN**: 继续构造周围的表达式或声明：`char b[4];                                                               \`。
- **L2295 EN**: Continues the surrounding expression or declaration: `} _Q6V32_internal_union;                                                   \`.
  **L2295 CN**: 继续构造周围的表达式或声明：`} _Q6V32_internal_union;                                                   \`。
- **L2296 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.b[0] = (b0);                                         \`.
  **L2296 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.b[0] = (b0);                                         \`。
- **L2297 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.b[1] = (b1);                                         \`.
  **L2297 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.b[1] = (b1);                                         \`。
- **L2298 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.b[2] = (b2);                                         \`.
  **L2298 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.b[2] = (b2);                                         \`。
- **L2299 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.b[3] = (b3);                                         \`.
  **L2299 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.b[3] = (b3);                                         \`。
- **L2300 EN**: Continues the surrounding expression or declaration: `_Q6V32_internal_union.d;                                                   \`.
  **L2300 CN**: 继续构造周围的表达式或声明：`_Q6V32_internal_union.d;                                                   \`。
- **L2301 EN**: Continues the surrounding expression or declaration: `})`.
  **L2301 CN**: 继续构造周围的表达式或声明：`})`。
- **L2302 EN**: Blank line separating nearby declarations or logic blocks.
  **L2302 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2303 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L2303 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L2304 EN**: Blank line separating nearby declarations or logic blocks.
  **L2304 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2305-2328

````c
#define Q6V32_CREATE_B(b3, b2, b1, b0)                                         \
  ((((Q6Vect32)(b3)) << 24) | (((Q6Vect32)((b2) & 0xff)) << 16) |              \
   (((Q6Vect32)((b1) & 0xff)) << 8) | ((Q6Vect32)((b0) & 0xff)))

#endif /* !__qdsp6__ */

#ifdef __cplusplus

class Q6Vect32C {
public:
  // Constructors
  Q6Vect32C(int w = 0) : data(w) {};
  Q6Vect32C(short h1, short h0) : data(Q6V32_CREATE_H(h1, h0)) {};
  Q6Vect32C(signed char b3, signed char b2, signed char b1, signed char b0)
      : data(Q6V32_CREATE_B(b3, b2, b1, b0)) {};
  Q6Vect32C(const Q6Vect32C &v) : data(v.data) {};

  Q6Vect32C &operator=(const Q6Vect32C &v) {
    data = v.data;
    return *this;
  };

  operator int() {
    return data;
````
- **L2305 EN**: Defines macro `Q6V32_CREATE_B(b3, b2, b1, b0)` for conditional compilation, shorthand, or API generation.
  **L2305 CN**: 定义宏 `Q6V32_CREATE_B(b3, b2, b1, b0)`，用于条件编译、简写或 API 生成。
- **L2306 EN**: Continues the surrounding expression or declaration: `((((Q6Vect32)(b3)) << 24) | (((Q6Vect32)((b2) & 0xff)) << 16) |              \`.
  **L2306 CN**: 继续构造周围的表达式或声明：`((((Q6Vect32)(b3)) << 24) | (((Q6Vect32)((b2) & 0xff)) << 16) |              \`。
- **L2307 EN**: Continues the surrounding expression or declaration: `(((Q6Vect32)((b1) & 0xff)) << 8) | ((Q6Vect32)((b0) & 0xff)))`.
  **L2307 CN**: 继续构造周围的表达式或声明：`(((Q6Vect32)((b1) & 0xff)) << 8) | ((Q6Vect32)((b0) & 0xff)))`。
- **L2308 EN**: Blank line separating nearby declarations or logic blocks.
  **L2308 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2309 EN**: Closes the current preprocessor conditional block.
  **L2309 CN**: 结束当前预处理条件块。
- **L2310 EN**: Blank line separating nearby declarations or logic blocks.
  **L2310 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2311 EN**: Starts a preprocessor conditional block: `#ifdef __cplusplus`.
  **L2311 CN**: 开始一个预处理条件块：`#ifdef __cplusplus`。
- **L2312 EN**: Blank line separating nearby declarations or logic blocks.
  **L2312 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2313 EN**: Declares class `Q6Vect32C`.
  **L2313 CN**: 声明 class `Q6Vect32C`。
- **L2314 EN**: Sets the access level for following class members to `public`.
  **L2314 CN**: 将后续类成员的访问级别设为 `public`。
- **L2315 EN**: Comment explains nearby logic, constraints, or intent: `Constructors`.
  **L2315 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructors`。
- **L2316 EN**: Executes a call or declaration centered on `Q6Vect32C`.
  **L2316 CN**: 执行以 `Q6Vect32C` 为核心的调用或声明。
- **L2317 EN**: Executes a call or declaration centered on `Q6Vect32C`.
  **L2317 CN**: 执行以 `Q6Vect32C` 为核心的调用或声明。
- **L2318 EN**: Continues logic associated with callable symbol `Q6Vect32C`.
  **L2318 CN**: 继续与可调用符号 `Q6Vect32C` 相关的逻辑。
- **L2319 EN**: Executes a call or declaration centered on `data`.
  **L2319 CN**: 执行以 `data` 为核心的调用或声明。
- **L2320 EN**: Executes a call or declaration centered on `Q6Vect32C`.
  **L2320 CN**: 执行以 `Q6Vect32C` 为核心的调用或声明。
- **L2321 EN**: Blank line separating nearby declarations or logic blocks.
  **L2321 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2322 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect32C &operator=(const Q6Vect32C &v) {`.
  **L2322 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect32C &operator=(const Q6Vect32C &v) {`。
- **L2323 EN**: Adds a standalone statement or declaration: `data = v.data;`.
  **L2323 CN**: 添加一条独立语句或声明：`data = v.data;`。
- **L2324 EN**: Returns from the current function with `*this`.
  **L2324 CN**: 以 `*this` 从当前函数返回。
- **L2325 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2325 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2326 EN**: Blank line separating nearby declarations or logic blocks.
  **L2326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2327 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `operator int() {`.
  **L2327 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`operator int() {`。
- **L2328 EN**: Returns from the current function with `data`.
  **L2328 CN**: 以 `data` 从当前函数返回。

### Lines 2329-2352

````c
  };

  // Extract word methods
  int W(void) {
    return Q6V32_GET_W(data);
  };
  unsigned int UW(void) {
    return Q6V32_GET_UW(data);
  };

  // Extract half word methods
  short H0(void) {
    return Q6V32_GET_H0(data);
  };
  short H1(void) {
    return Q6V32_GET_H1(data);
  };
  unsigned short UH0(void) {
    return Q6V32_GET_UH0(data);
  };
  unsigned short UH1(void) {
    return Q6V32_GET_UH1(data);
  };

````
- **L2329 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2329 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2330 EN**: Blank line separating nearby declarations or logic blocks.
  **L2330 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2331 EN**: Comment explains nearby logic, constraints, or intent: `Extract word methods`.
  **L2331 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract word methods`。
- **L2332 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `int W(void) {`.
  **L2332 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`int W(void) {`。
- **L2333 EN**: Returns from the current function with `Q6V32_GET_W(data)`.
  **L2333 CN**: 以 `Q6V32_GET_W(data)` 从当前函数返回。
- **L2334 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2334 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2335 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned int UW(void) {`.
  **L2335 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned int UW(void) {`。
- **L2336 EN**: Returns from the current function with `Q6V32_GET_UW(data)`.
  **L2336 CN**: 以 `Q6V32_GET_UW(data)` 从当前函数返回。
- **L2337 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2337 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2338 EN**: Blank line separating nearby declarations or logic blocks.
  **L2338 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2339 EN**: Comment explains nearby logic, constraints, or intent: `Extract half word methods`.
  **L2339 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract half word methods`。
- **L2340 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `short H0(void) {`.
  **L2340 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`short H0(void) {`。
- **L2341 EN**: Returns from the current function with `Q6V32_GET_H0(data)`.
  **L2341 CN**: 以 `Q6V32_GET_H0(data)` 从当前函数返回。
- **L2342 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2342 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2343 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `short H1(void) {`.
  **L2343 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`short H1(void) {`。
- **L2344 EN**: Returns from the current function with `Q6V32_GET_H1(data)`.
  **L2344 CN**: 以 `Q6V32_GET_H1(data)` 从当前函数返回。
- **L2345 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2345 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2346 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned short UH0(void) {`.
  **L2346 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned short UH0(void) {`。
- **L2347 EN**: Returns from the current function with `Q6V32_GET_UH0(data)`.
  **L2347 CN**: 以 `Q6V32_GET_UH0(data)` 从当前函数返回。
- **L2348 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2348 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2349 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned short UH1(void) {`.
  **L2349 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned short UH1(void) {`。
- **L2350 EN**: Returns from the current function with `Q6V32_GET_UH1(data)`.
  **L2350 CN**: 以 `Q6V32_GET_UH1(data)` 从当前函数返回。
- **L2351 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2351 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2352 EN**: Blank line separating nearby declarations or logic blocks.
  **L2352 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2353-2376

````c
  // Extract byte methods
  signed char b0(void) { return Q6V32_GET_B0(data); };
  signed char B0(void) { return b0(); };
  signed char B1(void) {
    return Q6V32_GET_B1(data);
  };
  signed char B2(void) {
    return Q6V32_GET_B2(data);
  };
  signed char B3(void) {
    return Q6V32_GET_B3(data);
  };
  unsigned char UB0(void) {
    return Q6V32_GET_UB0(data);
  };
  unsigned char UB1(void) {
    return Q6V32_GET_UB1(data);
  };
  unsigned char UB2(void) {
    return Q6V32_GET_UB2(data);
  };
  unsigned char UB3(void) {
    return Q6V32_GET_UB3(data);
  };
````
- **L2353 EN**: Comment explains nearby logic, constraints, or intent: `Extract byte methods`.
  **L2353 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract byte methods`。
- **L2354 EN**: Executes a call or declaration centered on `b0`.
  **L2354 CN**: 执行以 `b0` 为核心的调用或声明。
- **L2355 EN**: Executes a call or declaration centered on `B0`.
  **L2355 CN**: 执行以 `B0` 为核心的调用或声明。
- **L2356 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B1(void) {`.
  **L2356 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B1(void) {`。
- **L2357 EN**: Returns from the current function with `Q6V32_GET_B1(data)`.
  **L2357 CN**: 以 `Q6V32_GET_B1(data)` 从当前函数返回。
- **L2358 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2358 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2359 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B2(void) {`.
  **L2359 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B2(void) {`。
- **L2360 EN**: Returns from the current function with `Q6V32_GET_B2(data)`.
  **L2360 CN**: 以 `Q6V32_GET_B2(data)` 从当前函数返回。
- **L2361 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2361 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2362 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `signed char B3(void) {`.
  **L2362 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`signed char B3(void) {`。
- **L2363 EN**: Returns from the current function with `Q6V32_GET_B3(data)`.
  **L2363 CN**: 以 `Q6V32_GET_B3(data)` 从当前函数返回。
- **L2364 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2364 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2365 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB0(void) {`.
  **L2365 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB0(void) {`。
- **L2366 EN**: Returns from the current function with `Q6V32_GET_UB0(data)`.
  **L2366 CN**: 以 `Q6V32_GET_UB0(data)` 从当前函数返回。
- **L2367 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2367 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2368 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB1(void) {`.
  **L2368 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB1(void) {`。
- **L2369 EN**: Returns from the current function with `Q6V32_GET_UB1(data)`.
  **L2369 CN**: 以 `Q6V32_GET_UB1(data)` 从当前函数返回。
- **L2370 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2370 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2371 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB2(void) {`.
  **L2371 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB2(void) {`。
- **L2372 EN**: Returns from the current function with `Q6V32_GET_UB2(data)`.
  **L2372 CN**: 以 `Q6V32_GET_UB2(data)` 从当前函数返回。
- **L2373 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2373 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2374 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `unsigned char UB3(void) {`.
  **L2374 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`unsigned char UB3(void) {`。
- **L2375 EN**: Returns from the current function with `Q6V32_GET_UB3(data)`.
  **L2375 CN**: 以 `Q6V32_GET_UB3(data)` 从当前函数返回。
- **L2376 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2376 CN**: 结束当前声明作用域，例如结构体、枚举或类。

### Lines 2377-2400

````c

  // NOTE: All set methods return a Q6Vect32C type

  // Set word method
  Q6Vect32C W(int w) {
    return Q6Vect32C(Q6V32_PUT_W(data, w));
  };

  // Set half word methods
  Q6Vect32C H0(short h) {
    return Q6Vect32C(Q6V32_PUT_H0(data, h));
  };
  Q6Vect32C H1(short h) {
    return Q6Vect32C(Q6V32_PUT_H1(data, h));
  };

  // Set byte methods
  Q6Vect32C b0(signed char b) { return Q6Vect32C(Q6V32_PUT_B0(data, b)); };
  Q6Vect32C B0(signed char b) { return b0(b); };
  Q6Vect32C B1(signed char b) {
    return Q6Vect32C(Q6V32_PUT_B1(data, b));
  };
  Q6Vect32C B2(signed char b) {
    return Q6Vect32C(Q6V32_PUT_B2(data, b));
````
- **L2377 EN**: Blank line separating nearby declarations or logic blocks.
  **L2377 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2378 EN**: Comment highlights an implementation note: `NOTE: All set methods return a Q6Vect32C type`.
  **L2378 CN**: 注释强调一条实现说明：`NOTE: All set methods return a Q6Vect32C type`。
- **L2379 EN**: Blank line separating nearby declarations or logic blocks.
  **L2379 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2380 EN**: Comment explains nearby logic, constraints, or intent: `Set word method`.
  **L2380 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set word method`。
- **L2381 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect32C W(int w) {`.
  **L2381 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect32C W(int w) {`。
- **L2382 EN**: Returns from the current function with `Q6Vect32C(Q6V32_PUT_W(data, w))`.
  **L2382 CN**: 以 `Q6Vect32C(Q6V32_PUT_W(data, w))` 从当前函数返回。
- **L2383 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2383 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2384 EN**: Blank line separating nearby declarations or logic blocks.
  **L2384 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2385 EN**: Comment explains nearby logic, constraints, or intent: `Set half word methods`.
  **L2385 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set half word methods`。
- **L2386 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect32C H0(short h) {`.
  **L2386 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect32C H0(short h) {`。
- **L2387 EN**: Returns from the current function with `Q6Vect32C(Q6V32_PUT_H0(data, h))`.
  **L2387 CN**: 以 `Q6Vect32C(Q6V32_PUT_H0(data, h))` 从当前函数返回。
- **L2388 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2388 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2389 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect32C H1(short h) {`.
  **L2389 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect32C H1(short h) {`。
- **L2390 EN**: Returns from the current function with `Q6Vect32C(Q6V32_PUT_H1(data, h))`.
  **L2390 CN**: 以 `Q6Vect32C(Q6V32_PUT_H1(data, h))` 从当前函数返回。
- **L2391 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2391 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2392 EN**: Blank line separating nearby declarations or logic blocks.
  **L2392 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2393 EN**: Comment explains nearby logic, constraints, or intent: `Set byte methods`.
  **L2393 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set byte methods`。
- **L2394 EN**: Executes a call or declaration centered on `b0`.
  **L2394 CN**: 执行以 `b0` 为核心的调用或声明。
- **L2395 EN**: Executes a call or declaration centered on `B0`.
  **L2395 CN**: 执行以 `B0` 为核心的调用或声明。
- **L2396 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect32C B1(signed char b) {`.
  **L2396 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect32C B1(signed char b) {`。
- **L2397 EN**: Returns from the current function with `Q6Vect32C(Q6V32_PUT_B1(data, b))`.
  **L2397 CN**: 以 `Q6Vect32C(Q6V32_PUT_B1(data, b))` 从当前函数返回。
- **L2398 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2398 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2399 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect32C B2(signed char b) {`.
  **L2399 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect32C B2(signed char b) {`。
- **L2400 EN**: Returns from the current function with `Q6Vect32C(Q6V32_PUT_B2(data, b))`.
  **L2400 CN**: 以 `Q6Vect32C(Q6V32_PUT_B2(data, b))` 从当前函数返回。

### Lines 2401-2424

````c
  };
  Q6Vect32C B3(signed char b) {
    return Q6Vect32C(Q6V32_PUT_B3(data, b));
  };

private:
  int data;
};

#endif /* __cplusplus */

// V65 Vector types
#if __HVX_ARCH__ >= 65
#if defined __HVX__ && (__HVX_LENGTH__ == 128)
typedef long Q6VecPred128 __attribute__((__vector_size__(128)))
    __attribute__((aligned(128)));

typedef long Q6Vect1024 __attribute__((__vector_size__(128)))
    __attribute__((aligned(128)));

typedef long Q6Vect2048 __attribute__((__vector_size__(256)))
    __attribute__((aligned(256)));

#else /* defined __HVX__ && (__HVX_LENGTH__ == 128) */
````
- **L2401 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2401 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2402 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Q6Vect32C B3(signed char b) {`.
  **L2402 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Q6Vect32C B3(signed char b) {`。
- **L2403 EN**: Returns from the current function with `Q6Vect32C(Q6V32_PUT_B3(data, b))`.
  **L2403 CN**: 以 `Q6Vect32C(Q6V32_PUT_B3(data, b))` 从当前函数返回。
- **L2404 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2404 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2405 EN**: Blank line separating nearby declarations or logic blocks.
  **L2405 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2406 EN**: Sets the access level for following class members to `private`.
  **L2406 CN**: 将后续类成员的访问级别设为 `private`。
- **L2407 EN**: Adds a standalone statement or declaration: `int data;`.
  **L2407 CN**: 添加一条独立语句或声明：`int data;`。
- **L2408 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2408 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2409 EN**: Blank line separating nearby declarations or logic blocks.
  **L2409 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2410 EN**: Closes the current preprocessor conditional block.
  **L2410 CN**: 结束当前预处理条件块。
- **L2411 EN**: Blank line separating nearby declarations or logic blocks.
  **L2411 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2412 EN**: Comment explains nearby logic, constraints, or intent: `V65 Vector types`.
  **L2412 CN**: 注释解释附近代码的逻辑、约束或设计意图：`V65 Vector types`。
- **L2413 EN**: Starts a preprocessor conditional block: `#if __HVX_ARCH__ >= 65`.
  **L2413 CN**: 开始一个预处理条件块：`#if __HVX_ARCH__ >= 65`。
- **L2414 EN**: Starts a preprocessor conditional block: `#if defined __HVX__ && (__HVX_LENGTH__ == 128)`.
  **L2414 CN**: 开始一个预处理条件块：`#if defined __HVX__ && (__HVX_LENGTH__ == 128)`。
- **L2415 EN**: Introduces an alias or helper declaration: `typedef long Q6VecPred128 __attribute__((__vector_size__(128)))`.
  **L2415 CN**: 引入一条别名或辅助声明：`typedef long Q6VecPred128 __attribute__((__vector_size__(128)))`。
- **L2416 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((aligned(128)));`.
  **L2416 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((aligned(128)));`。
- **L2417 EN**: Blank line separating nearby declarations or logic blocks.
  **L2417 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2418 EN**: Introduces an alias or helper declaration: `typedef long Q6Vect1024 __attribute__((__vector_size__(128)))`.
  **L2418 CN**: 引入一条别名或辅助声明：`typedef long Q6Vect1024 __attribute__((__vector_size__(128)))`。
- **L2419 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((aligned(128)));`.
  **L2419 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((aligned(128)));`。
- **L2420 EN**: Blank line separating nearby declarations or logic blocks.
  **L2420 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2421 EN**: Introduces an alias or helper declaration: `typedef long Q6Vect2048 __attribute__((__vector_size__(256)))`.
  **L2421 CN**: 引入一条别名或辅助声明：`typedef long Q6Vect2048 __attribute__((__vector_size__(256)))`。
- **L2422 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((aligned(256)));`.
  **L2422 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((aligned(256)));`。
- **L2423 EN**: Blank line separating nearby declarations or logic blocks.
  **L2423 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2424 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L2424 CN**: 继续为当前目标或特性集合选择预处理分支。

### Lines 2425-2448

````c
#if defined __HVX__ &&  (__HVX_LENGTH__ == 64)
typedef long Q6VecPred64 __attribute__((__vector_size__(64)))
    __attribute__((aligned(64)));

typedef long Q6Vect512 __attribute__((__vector_size__(64)))
    __attribute__((aligned(64)));

typedef long Q6Vect1024 __attribute__((__vector_size__(128)))
    __attribute__((aligned(128)));

#endif /* defined __HVX__ &&  (__HVX_LENGTH__ == 64) */
#endif /* defined __HVX__ && (__HVX_LENGTH__ == 128) */
#endif /* __HVX_ARCH__ >= 65 */

/* Predicates */

typedef int Q6Pred;


#ifdef __HVX__

// Extract HVX VectorPair macro.
#define HEXAGON_HVX_GET_W(v) (v)

````
- **L2425 EN**: Starts a preprocessor conditional block: `#if defined __HVX__ &&  (__HVX_LENGTH__ == 64)`.
  **L2425 CN**: 开始一个预处理条件块：`#if defined __HVX__ &&  (__HVX_LENGTH__ == 64)`。
- **L2426 EN**: Introduces an alias or helper declaration: `typedef long Q6VecPred64 __attribute__((__vector_size__(64)))`.
  **L2426 CN**: 引入一条别名或辅助声明：`typedef long Q6VecPred64 __attribute__((__vector_size__(64)))`。
- **L2427 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((aligned(64)));`.
  **L2427 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((aligned(64)));`。
- **L2428 EN**: Blank line separating nearby declarations or logic blocks.
  **L2428 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2429 EN**: Introduces an alias or helper declaration: `typedef long Q6Vect512 __attribute__((__vector_size__(64)))`.
  **L2429 CN**: 引入一条别名或辅助声明：`typedef long Q6Vect512 __attribute__((__vector_size__(64)))`。
- **L2430 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((aligned(64)));`.
  **L2430 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((aligned(64)));`。
- **L2431 EN**: Blank line separating nearby declarations or logic blocks.
  **L2431 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2432 EN**: Introduces an alias or helper declaration: `typedef long Q6Vect1024 __attribute__((__vector_size__(128)))`.
  **L2432 CN**: 引入一条别名或辅助声明：`typedef long Q6Vect1024 __attribute__((__vector_size__(128)))`。
- **L2433 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((aligned(128)));`.
  **L2433 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((aligned(128)));`。
- **L2434 EN**: Blank line separating nearby declarations or logic blocks.
  **L2434 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2435 EN**: Closes the current preprocessor conditional block.
  **L2435 CN**: 结束当前预处理条件块。
- **L2436 EN**: Closes the current preprocessor conditional block.
  **L2436 CN**: 结束当前预处理条件块。
- **L2437 EN**: Closes the current preprocessor conditional block.
  **L2437 CN**: 结束当前预处理条件块。
- **L2438 EN**: Blank line separating nearby declarations or logic blocks.
  **L2438 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2439 EN**: Comment explains nearby logic, constraints, or intent: `Predicates`.
  **L2439 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Predicates`。
- **L2440 EN**: Blank line separating nearby declarations or logic blocks.
  **L2440 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2441 EN**: Introduces an alias or helper declaration: `typedef int Q6Pred;`.
  **L2441 CN**: 引入一条别名或辅助声明：`typedef int Q6Pred;`。
- **L2442 EN**: Blank line separating nearby declarations or logic blocks.
  **L2442 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2443 EN**: Blank line separating nearby declarations or logic blocks.
  **L2443 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2444 EN**: Starts a preprocessor conditional block: `#ifdef __HVX__`.
  **L2444 CN**: 开始一个预处理条件块：`#ifdef __HVX__`。
- **L2445 EN**: Blank line separating nearby declarations or logic blocks.
  **L2445 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2446 EN**: Comment explains nearby logic, constraints, or intent: `Extract HVX VectorPair macro.`.
  **L2446 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract HVX VectorPair macro.`。
- **L2447 EN**: Defines macro `HEXAGON_HVX_GET_W(v)` for conditional compilation, shorthand, or API generation.
  **L2447 CN**: 定义宏 `HEXAGON_HVX_GET_W(v)`，用于条件编译、简写或 API 生成。
- **L2448 EN**: Blank line separating nearby declarations or logic blocks.
  **L2448 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2449-2472

````c
// Extract HVX Vector macros.
#define HEXAGON_HVX_GET_V0(v)                                                  \
  __extension__({                                                              \
    union {                                                                    \
      HVX_VectorPair W;                                                        \
      HVX_Vector V[2];                                                         \
    } _HEXAGON_HVX_internal_union;                                             \
    _HEXAGON_HVX_internal_union.W = (v);                                       \
    _HEXAGON_HVX_internal_union.V[0];                                          \
  })
#define HEXAGON_HVX_GET_V1(v)                                                  \
  __extension__({                                                              \
    union {                                                                    \
      HVX_VectorPair W;                                                        \
      HVX_Vector V[2];                                                         \
    } _HEXAGON_HVX_internal_union;                                             \
    _HEXAGON_HVX_internal_union.W = (v);                                       \
    _HEXAGON_HVX_internal_union.V[1];                                          \
  })
#define HEXAGON_HVX_GET_P(v)                                                   \
  __extension__({                                                              \
    union {                                                                    \
      HVX_VectorPair W;                                                        \
      HVX_VectorPred P[2];                                                     \
````
- **L2449 EN**: Comment explains nearby logic, constraints, or intent: `Extract HVX Vector macros.`.
  **L2449 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract HVX Vector macros.`。
- **L2450 EN**: Defines macro `HEXAGON_HVX_GET_V0(v)` for conditional compilation, shorthand, or API generation.
  **L2450 CN**: 定义宏 `HEXAGON_HVX_GET_V0(v)`，用于条件编译、简写或 API 生成。
- **L2451 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2451 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2452 EN**: Declares union `union`.
  **L2452 CN**: 声明 union `union`。
- **L2453 EN**: Continues the surrounding expression or declaration: `HVX_VectorPair W;                                                        \`.
  **L2453 CN**: 继续构造周围的表达式或声明：`HVX_VectorPair W;                                                        \`。
- **L2454 EN**: Continues the surrounding expression or declaration: `HVX_Vector V[2];                                                         \`.
  **L2454 CN**: 继续构造周围的表达式或声明：`HVX_Vector V[2];                                                         \`。
- **L2455 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_HVX_internal_union;                                             \`.
  **L2455 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_HVX_internal_union;                                             \`。
- **L2456 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.W = (v);                                       \`.
  **L2456 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.W = (v);                                       \`。
- **L2457 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.V[0];                                          \`.
  **L2457 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.V[0];                                          \`。
- **L2458 EN**: Continues the surrounding expression or declaration: `})`.
  **L2458 CN**: 继续构造周围的表达式或声明：`})`。
- **L2459 EN**: Defines macro `HEXAGON_HVX_GET_V1(v)` for conditional compilation, shorthand, or API generation.
  **L2459 CN**: 定义宏 `HEXAGON_HVX_GET_V1(v)`，用于条件编译、简写或 API 生成。
- **L2460 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2460 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2461 EN**: Declares union `union`.
  **L2461 CN**: 声明 union `union`。
- **L2462 EN**: Continues the surrounding expression or declaration: `HVX_VectorPair W;                                                        \`.
  **L2462 CN**: 继续构造周围的表达式或声明：`HVX_VectorPair W;                                                        \`。
- **L2463 EN**: Continues the surrounding expression or declaration: `HVX_Vector V[2];                                                         \`.
  **L2463 CN**: 继续构造周围的表达式或声明：`HVX_Vector V[2];                                                         \`。
- **L2464 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_HVX_internal_union;                                             \`.
  **L2464 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_HVX_internal_union;                                             \`。
- **L2465 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.W = (v);                                       \`.
  **L2465 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.W = (v);                                       \`。
- **L2466 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.V[1];                                          \`.
  **L2466 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.V[1];                                          \`。
- **L2467 EN**: Continues the surrounding expression or declaration: `})`.
  **L2467 CN**: 继续构造周围的表达式或声明：`})`。
- **L2468 EN**: Defines macro `HEXAGON_HVX_GET_P(v)` for conditional compilation, shorthand, or API generation.
  **L2468 CN**: 定义宏 `HEXAGON_HVX_GET_P(v)`，用于条件编译、简写或 API 生成。
- **L2469 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2469 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2470 EN**: Declares union `union`.
  **L2470 CN**: 声明 union `union`。
- **L2471 EN**: Continues the surrounding expression or declaration: `HVX_VectorPair W;                                                        \`.
  **L2471 CN**: 继续构造周围的表达式或声明：`HVX_VectorPair W;                                                        \`。
- **L2472 EN**: Continues the surrounding expression or declaration: `HVX_VectorPred P[2];                                                     \`.
  **L2472 CN**: 继续构造周围的表达式或声明：`HVX_VectorPred P[2];                                                     \`。

### Lines 2473-2496

````c
    } _HEXAGON_HVX_internal_union;                                             \
    _HEXAGON_HVX_internal_union.W = (v);                                       \
    _HEXAGON_HVX_internal_union.P[0];                                          \
  })

// Set HVX VectorPair macro.
#define HEXAGON_HVX_PUT_W(v, new) (new)

// Set HVX Vector macros.
#define HEXAGON_HVX_PUT_V0(v, new)                                             \
  __extension__({                                                              \
    union {                                                                    \
      HVX_VectorPair W;                                                        \
      HVX_Vector V[2];                                                         \
    } _HEXAGON_HVX_internal_union;                                             \
    _HEXAGON_HVX_internal_union.W = (v);                                       \
    _HEXAGON_HVX_internal_union.V[0] = (new);                                  \
    _HEXAGON_HVX_internal_union.W;                                             \
  })

#define HEXAGON_HVX_PUT_V1(v, new)                                             \
  __extension__({                                                              \
    union {                                                                    \
      HVX_VectorPair W;                                                        \
````
- **L2473 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_HVX_internal_union;                                             \`.
  **L2473 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_HVX_internal_union;                                             \`。
- **L2474 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.W = (v);                                       \`.
  **L2474 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.W = (v);                                       \`。
- **L2475 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.P[0];                                          \`.
  **L2475 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.P[0];                                          \`。
- **L2476 EN**: Continues the surrounding expression or declaration: `})`.
  **L2476 CN**: 继续构造周围的表达式或声明：`})`。
- **L2477 EN**: Blank line separating nearby declarations or logic blocks.
  **L2477 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2478 EN**: Comment explains nearby logic, constraints, or intent: `Set HVX VectorPair macro.`.
  **L2478 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set HVX VectorPair macro.`。
- **L2479 EN**: Defines macro `HEXAGON_HVX_PUT_W(v, new)` for conditional compilation, shorthand, or API generation.
  **L2479 CN**: 定义宏 `HEXAGON_HVX_PUT_W(v, new)`，用于条件编译、简写或 API 生成。
- **L2480 EN**: Blank line separating nearby declarations or logic blocks.
  **L2480 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2481 EN**: Comment explains nearby logic, constraints, or intent: `Set HVX Vector macros.`.
  **L2481 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set HVX Vector macros.`。
- **L2482 EN**: Defines macro `HEXAGON_HVX_PUT_V0(v, new)` for conditional compilation, shorthand, or API generation.
  **L2482 CN**: 定义宏 `HEXAGON_HVX_PUT_V0(v, new)`，用于条件编译、简写或 API 生成。
- **L2483 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2483 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2484 EN**: Declares union `union`.
  **L2484 CN**: 声明 union `union`。
- **L2485 EN**: Continues the surrounding expression or declaration: `HVX_VectorPair W;                                                        \`.
  **L2485 CN**: 继续构造周围的表达式或声明：`HVX_VectorPair W;                                                        \`。
- **L2486 EN**: Continues the surrounding expression or declaration: `HVX_Vector V[2];                                                         \`.
  **L2486 CN**: 继续构造周围的表达式或声明：`HVX_Vector V[2];                                                         \`。
- **L2487 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_HVX_internal_union;                                             \`.
  **L2487 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_HVX_internal_union;                                             \`。
- **L2488 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.W = (v);                                       \`.
  **L2488 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.W = (v);                                       \`。
- **L2489 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.V[0] = (new);                                  \`.
  **L2489 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.V[0] = (new);                                  \`。
- **L2490 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.W;                                             \`.
  **L2490 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.W;                                             \`。
- **L2491 EN**: Continues the surrounding expression or declaration: `})`.
  **L2491 CN**: 继续构造周围的表达式或声明：`})`。
- **L2492 EN**: Blank line separating nearby declarations or logic blocks.
  **L2492 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2493 EN**: Defines macro `HEXAGON_HVX_PUT_V1(v, new)` for conditional compilation, shorthand, or API generation.
  **L2493 CN**: 定义宏 `HEXAGON_HVX_PUT_V1(v, new)`，用于条件编译、简写或 API 生成。
- **L2494 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2494 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2495 EN**: Declares union `union`.
  **L2495 CN**: 声明 union `union`。
- **L2496 EN**: Continues the surrounding expression or declaration: `HVX_VectorPair W;                                                        \`.
  **L2496 CN**: 继续构造周围的表达式或声明：`HVX_VectorPair W;                                                        \`。

### Lines 2497-2520

````c
      HVX_Vector V[2];                                                         \
    } _HEXAGON_HVX_internal_union;                                             \
    _HEXAGON_HVX_internal_union.W = (v);                                       \
    _HEXAGON_HVX_internal_union.V[1] = (new);                                  \
    _HEXAGON_HVX_internal_union.W;                                             \
  })

#define HEXAGON_HVX_PUT_P(v, new)                                              \
  __extension__({                                                              \
    union {                                                                    \
      HVX_VectorPair W;                                                        \
      HVX_VectorPred P[2];                                                     \
    } _HEXAGON_HVX_internal_union;                                             \
    _HEXAGON_HVX_internal_union.W = (v);                                       \
    _HEXAGON_HVX_internal_union.P[0] = (new);                                  \
    _HEXAGON_HVX_internal_union.W;                                             \
  })


#define HEXAGON_HVX_CREATE_W(v1, v0)                                           \
  __extension__({                                                              \
    union {                                                                    \
      HVX_VectorPair W;                                                        \
      HVX_Vector V[2];                                                         \
````
- **L2497 EN**: Continues the surrounding expression or declaration: `HVX_Vector V[2];                                                         \`.
  **L2497 CN**: 继续构造周围的表达式或声明：`HVX_Vector V[2];                                                         \`。
- **L2498 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_HVX_internal_union;                                             \`.
  **L2498 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_HVX_internal_union;                                             \`。
- **L2499 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.W = (v);                                       \`.
  **L2499 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.W = (v);                                       \`。
- **L2500 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.V[1] = (new);                                  \`.
  **L2500 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.V[1] = (new);                                  \`。
- **L2501 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.W;                                             \`.
  **L2501 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.W;                                             \`。
- **L2502 EN**: Continues the surrounding expression or declaration: `})`.
  **L2502 CN**: 继续构造周围的表达式或声明：`})`。
- **L2503 EN**: Blank line separating nearby declarations or logic blocks.
  **L2503 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2504 EN**: Defines macro `HEXAGON_HVX_PUT_P(v, new)` for conditional compilation, shorthand, or API generation.
  **L2504 CN**: 定义宏 `HEXAGON_HVX_PUT_P(v, new)`，用于条件编译、简写或 API 生成。
- **L2505 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2505 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2506 EN**: Declares union `union`.
  **L2506 CN**: 声明 union `union`。
- **L2507 EN**: Continues the surrounding expression or declaration: `HVX_VectorPair W;                                                        \`.
  **L2507 CN**: 继续构造周围的表达式或声明：`HVX_VectorPair W;                                                        \`。
- **L2508 EN**: Continues the surrounding expression or declaration: `HVX_VectorPred P[2];                                                     \`.
  **L2508 CN**: 继续构造周围的表达式或声明：`HVX_VectorPred P[2];                                                     \`。
- **L2509 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_HVX_internal_union;                                             \`.
  **L2509 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_HVX_internal_union;                                             \`。
- **L2510 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.W = (v);                                       \`.
  **L2510 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.W = (v);                                       \`。
- **L2511 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.P[0] = (new);                                  \`.
  **L2511 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.P[0] = (new);                                  \`。
- **L2512 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.W;                                             \`.
  **L2512 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.W;                                             \`。
- **L2513 EN**: Continues the surrounding expression or declaration: `})`.
  **L2513 CN**: 继续构造周围的表达式或声明：`})`。
- **L2514 EN**: Blank line separating nearby declarations or logic blocks.
  **L2514 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2515 EN**: Blank line separating nearby declarations or logic blocks.
  **L2515 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2516 EN**: Defines macro `HEXAGON_HVX_CREATE_W(v1, v0)` for conditional compilation, shorthand, or API generation.
  **L2516 CN**: 定义宏 `HEXAGON_HVX_CREATE_W(v1, v0)`，用于条件编译、简写或 API 生成。
- **L2517 EN**: Continues logic associated with callable symbol `__extension__`.
  **L2517 CN**: 继续与可调用符号 `__extension__` 相关的逻辑。
- **L2518 EN**: Declares union `union`.
  **L2518 CN**: 声明 union `union`。
- **L2519 EN**: Continues the surrounding expression or declaration: `HVX_VectorPair W;                                                        \`.
  **L2519 CN**: 继续构造周围的表达式或声明：`HVX_VectorPair W;                                                        \`。
- **L2520 EN**: Continues the surrounding expression or declaration: `HVX_Vector V[2];                                                         \`.
  **L2520 CN**: 继续构造周围的表达式或声明：`HVX_Vector V[2];                                                         \`。

### Lines 2521-2544

````c
    } _HEXAGON_HVX_internal_union;                                             \
    _HEXAGON_HVX_internal_union.V[0] = (v0);                                   \
    _HEXAGON_HVX_internal_union.V[1] = (v1);                                   \
    _HEXAGON_HVX_internal_union.W;                                             \
  })

#ifdef __cplusplus

class HVX_Vect {
public:
  // Constructors.
  // Default.
  HVX_Vect() : data(Q6_W_vcombine_VV(Q6_V_vzero(), Q6_V_vzero())){};

  // Custom constructors.
  HVX_Vect(HVX_VectorPair W) : data(W){};
  HVX_Vect(HVX_Vector v1, HVX_Vector v0) : data(HEXAGON_HVX_CREATE_W(v1, v0)){};

  // Copy constructor.
  HVX_Vect(const HVX_Vect &W) = default;

  // Move constructor.
  HVX_Vect(HVX_Vect &&W) = default;

````
- **L2521 EN**: Continues the surrounding expression or declaration: `} _HEXAGON_HVX_internal_union;                                             \`.
  **L2521 CN**: 继续构造周围的表达式或声明：`} _HEXAGON_HVX_internal_union;                                             \`。
- **L2522 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.V[0] = (v0);                                   \`.
  **L2522 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.V[0] = (v0);                                   \`。
- **L2523 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.V[1] = (v1);                                   \`.
  **L2523 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.V[1] = (v1);                                   \`。
- **L2524 EN**: Continues the surrounding expression or declaration: `_HEXAGON_HVX_internal_union.W;                                             \`.
  **L2524 CN**: 继续构造周围的表达式或声明：`_HEXAGON_HVX_internal_union.W;                                             \`。
- **L2525 EN**: Continues the surrounding expression or declaration: `})`.
  **L2525 CN**: 继续构造周围的表达式或声明：`})`。
- **L2526 EN**: Blank line separating nearby declarations or logic blocks.
  **L2526 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2527 EN**: Starts a preprocessor conditional block: `#ifdef __cplusplus`.
  **L2527 CN**: 开始一个预处理条件块：`#ifdef __cplusplus`。
- **L2528 EN**: Blank line separating nearby declarations or logic blocks.
  **L2528 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2529 EN**: Declares class `HVX_Vect`.
  **L2529 CN**: 声明 class `HVX_Vect`。
- **L2530 EN**: Sets the access level for following class members to `public`.
  **L2530 CN**: 将后续类成员的访问级别设为 `public`。
- **L2531 EN**: Comment explains nearby logic, constraints, or intent: `Constructors.`.
  **L2531 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Constructors.`。
- **L2532 EN**: Comment explains nearby logic, constraints, or intent: `Default.`.
  **L2532 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Default.`。
- **L2533 EN**: Executes a call or declaration centered on `HVX_Vect`.
  **L2533 CN**: 执行以 `HVX_Vect` 为核心的调用或声明。
- **L2534 EN**: Blank line separating nearby declarations or logic blocks.
  **L2534 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2535 EN**: Comment explains nearby logic, constraints, or intent: `Custom constructors.`.
  **L2535 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Custom constructors.`。
- **L2536 EN**: Executes a call or declaration centered on `HVX_Vect`.
  **L2536 CN**: 执行以 `HVX_Vect` 为核心的调用或声明。
- **L2537 EN**: Executes a call or declaration centered on `HVX_Vect`.
  **L2537 CN**: 执行以 `HVX_Vect` 为核心的调用或声明。
- **L2538 EN**: Blank line separating nearby declarations or logic blocks.
  **L2538 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2539 EN**: Comment explains nearby logic, constraints, or intent: `Copy constructor.`.
  **L2539 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Copy constructor.`。
- **L2540 EN**: Executes a call or declaration centered on `HVX_Vect`.
  **L2540 CN**: 执行以 `HVX_Vect` 为核心的调用或声明。
- **L2541 EN**: Blank line separating nearby declarations or logic blocks.
  **L2541 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2542 EN**: Comment explains nearby logic, constraints, or intent: `Move constructor.`.
  **L2542 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Move constructor.`。
- **L2543 EN**: Executes a call or declaration centered on `HVX_Vect`.
  **L2543 CN**: 执行以 `HVX_Vect` 为核心的调用或声明。
- **L2544 EN**: Blank line separating nearby declarations or logic blocks.
  **L2544 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2545-2568

````c
  // Assignment operator.
  HVX_Vect &operator=(const HVX_Vect &W) = default;

  operator HVX_VectorPair() { return data; };

  // Extract VectorPair method.
  HVX_VectorPair W(void) { return HEXAGON_HVX_GET_W(data); };

  // Extract Vector methods.
  HVX_Vector V0(void) { return HEXAGON_HVX_GET_V0(data); };
  HVX_Vector V1(void) { return HEXAGON_HVX_GET_V1(data); };
  HVX_VectorPred P(void) { return HEXAGON_HVX_GET_P(data); };

  // NOTE: All set methods return a HVX_Vect type.
  // Set HVX VectorPair method.
  HVX_Vect W(HVX_VectorPair w) { return HVX_Vect(HEXAGON_HVX_PUT_W(data, w)); };

  // Set HVX Vector methods.
  HVX_Vect V0(HVX_Vector v) { return HVX_Vect(HEXAGON_HVX_PUT_V0(data, v)); };
  HVX_Vect V1(HVX_Vector v) { return HVX_Vect(HEXAGON_HVX_PUT_V1(data, v)); };
  HVX_Vect P(HVX_VectorPred p) { return HVX_Vect(HEXAGON_HVX_PUT_P(data, p)); };

private:
  HVX_VectorPair data;
````
- **L2545 EN**: Comment explains nearby logic, constraints, or intent: `Assignment operator.`.
  **L2545 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Assignment operator.`。
- **L2546 EN**: Executes a call or declaration centered on `&operator=`.
  **L2546 CN**: 执行以 `&operator=` 为核心的调用或声明。
- **L2547 EN**: Blank line separating nearby declarations or logic blocks.
  **L2547 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2548 EN**: Executes a call or declaration centered on `HVX_VectorPair`.
  **L2548 CN**: 执行以 `HVX_VectorPair` 为核心的调用或声明。
- **L2549 EN**: Blank line separating nearby declarations or logic blocks.
  **L2549 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2550 EN**: Comment explains nearby logic, constraints, or intent: `Extract VectorPair method.`.
  **L2550 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract VectorPair method.`。
- **L2551 EN**: Executes a call or declaration centered on `W`.
  **L2551 CN**: 执行以 `W` 为核心的调用或声明。
- **L2552 EN**: Blank line separating nearby declarations or logic blocks.
  **L2552 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2553 EN**: Comment explains nearby logic, constraints, or intent: `Extract Vector methods.`.
  **L2553 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Extract Vector methods.`。
- **L2554 EN**: Executes a call or declaration centered on `V0`.
  **L2554 CN**: 执行以 `V0` 为核心的调用或声明。
- **L2555 EN**: Executes a call or declaration centered on `V1`.
  **L2555 CN**: 执行以 `V1` 为核心的调用或声明。
- **L2556 EN**: Executes a call or declaration centered on `P`.
  **L2556 CN**: 执行以 `P` 为核心的调用或声明。
- **L2557 EN**: Blank line separating nearby declarations or logic blocks.
  **L2557 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2558 EN**: Comment highlights an implementation note: `NOTE: All set methods return a HVX_Vect type.`.
  **L2558 CN**: 注释强调一条实现说明：`NOTE: All set methods return a HVX_Vect type.`。
- **L2559 EN**: Comment explains nearby logic, constraints, or intent: `Set HVX VectorPair method.`.
  **L2559 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set HVX VectorPair method.`。
- **L2560 EN**: Executes a call or declaration centered on `W`.
  **L2560 CN**: 执行以 `W` 为核心的调用或声明。
- **L2561 EN**: Blank line separating nearby declarations or logic blocks.
  **L2561 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2562 EN**: Comment explains nearby logic, constraints, or intent: `Set HVX Vector methods.`.
  **L2562 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Set HVX Vector methods.`。
- **L2563 EN**: Executes a call or declaration centered on `V0`.
  **L2563 CN**: 执行以 `V0` 为核心的调用或声明。
- **L2564 EN**: Executes a call or declaration centered on `V1`.
  **L2564 CN**: 执行以 `V1` 为核心的调用或声明。
- **L2565 EN**: Executes a call or declaration centered on `P`.
  **L2565 CN**: 执行以 `P` 为核心的调用或声明。
- **L2566 EN**: Blank line separating nearby declarations or logic blocks.
  **L2566 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2567 EN**: Sets the access level for following class members to `private`.
  **L2567 CN**: 将后续类成员的访问级别设为 `private`。
- **L2568 EN**: Adds a standalone statement or declaration: `HVX_VectorPair data;`.
  **L2568 CN**: 添加一条独立语句或声明：`HVX_VectorPair data;`。

### Lines 2569-2592

````c
};

#endif /* __cplusplus */
#endif /* __HVX__ */

#define HEXAGON_UDMA_DM0_STATUS_IDLE             0x00000000
#define HEXAGON_UDMA_DM0_STATUS_RUN              0x00000001
#define HEXAGON_UDMA_DM0_STATUS_ERROR            0x00000002
#define HEXAGON_UDMA_DESC_DSTATE_INCOMPLETE      0
#define HEXAGON_UDMA_DESC_DSTATE_COMPLETE        1
#define HEXAGON_UDMA_DESC_ORDER_NOORDER          0
#define HEXAGON_UDMA_DESC_ORDER_ORDER            1
#define HEXAGON_UDMA_DESC_BYPASS_OFF             0
#define HEXAGON_UDMA_DESC_BYPASS_ON              1
#define HEXAGON_UDMA_DESC_COMP_NONE              0
#define HEXAGON_UDMA_DESC_COMP_DLBC              1
#define HEXAGON_UDMA_DESC_DESCTYPE_TYPE0         0
#define HEXAGON_UDMA_DESC_DESCTYPE_TYPE1         1

typedef struct hexagon_udma_descriptor_type0_s
{
    void *next;
    unsigned int length:24;
    unsigned int desctype:2;
````
- **L2569 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L2569 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L2570 EN**: Blank line separating nearby declarations or logic blocks.
  **L2570 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2571 EN**: Closes the current preprocessor conditional block.
  **L2571 CN**: 结束当前预处理条件块。
- **L2572 EN**: Closes the current preprocessor conditional block.
  **L2572 CN**: 结束当前预处理条件块。
- **L2573 EN**: Blank line separating nearby declarations or logic blocks.
  **L2573 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2574 EN**: Defines macro `HEXAGON_UDMA_DM0_STATUS_IDLE` for conditional compilation, shorthand, or API generation.
  **L2574 CN**: 定义宏 `HEXAGON_UDMA_DM0_STATUS_IDLE`，用于条件编译、简写或 API 生成。
- **L2575 EN**: Defines macro `HEXAGON_UDMA_DM0_STATUS_RUN` for conditional compilation, shorthand, or API generation.
  **L2575 CN**: 定义宏 `HEXAGON_UDMA_DM0_STATUS_RUN`，用于条件编译、简写或 API 生成。
- **L2576 EN**: Defines macro `HEXAGON_UDMA_DM0_STATUS_ERROR` for conditional compilation, shorthand, or API generation.
  **L2576 CN**: 定义宏 `HEXAGON_UDMA_DM0_STATUS_ERROR`，用于条件编译、简写或 API 生成。
- **L2577 EN**: Defines macro `HEXAGON_UDMA_DESC_DSTATE_INCOMPLETE` for conditional compilation, shorthand, or API generation.
  **L2577 CN**: 定义宏 `HEXAGON_UDMA_DESC_DSTATE_INCOMPLETE`，用于条件编译、简写或 API 生成。
- **L2578 EN**: Defines macro `HEXAGON_UDMA_DESC_DSTATE_COMPLETE` for conditional compilation, shorthand, or API generation.
  **L2578 CN**: 定义宏 `HEXAGON_UDMA_DESC_DSTATE_COMPLETE`，用于条件编译、简写或 API 生成。
- **L2579 EN**: Defines macro `HEXAGON_UDMA_DESC_ORDER_NOORDER` for conditional compilation, shorthand, or API generation.
  **L2579 CN**: 定义宏 `HEXAGON_UDMA_DESC_ORDER_NOORDER`，用于条件编译、简写或 API 生成。
- **L2580 EN**: Defines macro `HEXAGON_UDMA_DESC_ORDER_ORDER` for conditional compilation, shorthand, or API generation.
  **L2580 CN**: 定义宏 `HEXAGON_UDMA_DESC_ORDER_ORDER`，用于条件编译、简写或 API 生成。
- **L2581 EN**: Defines macro `HEXAGON_UDMA_DESC_BYPASS_OFF` for conditional compilation, shorthand, or API generation.
  **L2581 CN**: 定义宏 `HEXAGON_UDMA_DESC_BYPASS_OFF`，用于条件编译、简写或 API 生成。
- **L2582 EN**: Defines macro `HEXAGON_UDMA_DESC_BYPASS_ON` for conditional compilation, shorthand, or API generation.
  **L2582 CN**: 定义宏 `HEXAGON_UDMA_DESC_BYPASS_ON`，用于条件编译、简写或 API 生成。
- **L2583 EN**: Defines macro `HEXAGON_UDMA_DESC_COMP_NONE` for conditional compilation, shorthand, or API generation.
  **L2583 CN**: 定义宏 `HEXAGON_UDMA_DESC_COMP_NONE`，用于条件编译、简写或 API 生成。
- **L2584 EN**: Defines macro `HEXAGON_UDMA_DESC_COMP_DLBC` for conditional compilation, shorthand, or API generation.
  **L2584 CN**: 定义宏 `HEXAGON_UDMA_DESC_COMP_DLBC`，用于条件编译、简写或 API 生成。
- **L2585 EN**: Defines macro `HEXAGON_UDMA_DESC_DESCTYPE_TYPE0` for conditional compilation, shorthand, or API generation.
  **L2585 CN**: 定义宏 `HEXAGON_UDMA_DESC_DESCTYPE_TYPE0`，用于条件编译、简写或 API 生成。
- **L2586 EN**: Defines macro `HEXAGON_UDMA_DESC_DESCTYPE_TYPE1` for conditional compilation, shorthand, or API generation.
  **L2586 CN**: 定义宏 `HEXAGON_UDMA_DESC_DESCTYPE_TYPE1`，用于条件编译、简写或 API 生成。
- **L2587 EN**: Blank line separating nearby declarations or logic blocks.
  **L2587 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2588 EN**: Introduces an alias or helper declaration: `typedef struct hexagon_udma_descriptor_type0_s`.
  **L2588 CN**: 引入一条别名或辅助声明：`typedef struct hexagon_udma_descriptor_type0_s`。
- **L2589 EN**: Opens a new lexical scope or compound statement.
  **L2589 CN**: 打开一个新的词法作用域或复合语句块。
- **L2590 EN**: Adds a standalone statement or declaration: `void *next;`.
  **L2590 CN**: 添加一条独立语句或声明：`void *next;`。
- **L2591 EN**: Adds a standalone statement or declaration: `unsigned int length:24;`.
  **L2591 CN**: 添加一条独立语句或声明：`unsigned int length:24;`。
- **L2592 EN**: Adds a standalone statement or declaration: `unsigned int desctype:2;`.
  **L2592 CN**: 添加一条独立语句或声明：`unsigned int desctype:2;`。

### Lines 2593-2616

````c
    unsigned int dstcomp:1;
    unsigned int srccomp:1;
    unsigned int dstbypass:1;
    unsigned int srcbypass:1;
    unsigned int order:1;
    unsigned int dstate:1;
    void *src;
    void *dst;
} hexagon_udma_descriptor_type0_t;

typedef struct hexagon_udma_descriptor_type1_s
{
    void *next;
    unsigned int length:24;
    unsigned int desctype:2;
    unsigned int dstcomp:1;
    unsigned int srccomp:1;
    unsigned int dstbypass:1;
    unsigned int srcbypass:1;
    unsigned int order:1;
    unsigned int dstate:1;
    void *src;
    void *dst;
    unsigned int allocation:28;
````
- **L2593 EN**: Adds a standalone statement or declaration: `unsigned int dstcomp:1;`.
  **L2593 CN**: 添加一条独立语句或声明：`unsigned int dstcomp:1;`。
- **L2594 EN**: Adds a standalone statement or declaration: `unsigned int srccomp:1;`.
  **L2594 CN**: 添加一条独立语句或声明：`unsigned int srccomp:1;`。
- **L2595 EN**: Adds a standalone statement or declaration: `unsigned int dstbypass:1;`.
  **L2595 CN**: 添加一条独立语句或声明：`unsigned int dstbypass:1;`。
- **L2596 EN**: Adds a standalone statement or declaration: `unsigned int srcbypass:1;`.
  **L2596 CN**: 添加一条独立语句或声明：`unsigned int srcbypass:1;`。
- **L2597 EN**: Adds a standalone statement or declaration: `unsigned int order:1;`.
  **L2597 CN**: 添加一条独立语句或声明：`unsigned int order:1;`。
- **L2598 EN**: Adds a standalone statement or declaration: `unsigned int dstate:1;`.
  **L2598 CN**: 添加一条独立语句或声明：`unsigned int dstate:1;`。
- **L2599 EN**: Adds a standalone statement or declaration: `void *src;`.
  **L2599 CN**: 添加一条独立语句或声明：`void *src;`。
- **L2600 EN**: Adds a standalone statement or declaration: `void *dst;`.
  **L2600 CN**: 添加一条独立语句或声明：`void *dst;`。
- **L2601 EN**: Adds a standalone statement or declaration: `} hexagon_udma_descriptor_type0_t;`.
  **L2601 CN**: 添加一条独立语句或声明：`} hexagon_udma_descriptor_type0_t;`。
- **L2602 EN**: Blank line separating nearby declarations or logic blocks.
  **L2602 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2603 EN**: Introduces an alias or helper declaration: `typedef struct hexagon_udma_descriptor_type1_s`.
  **L2603 CN**: 引入一条别名或辅助声明：`typedef struct hexagon_udma_descriptor_type1_s`。
- **L2604 EN**: Opens a new lexical scope or compound statement.
  **L2604 CN**: 打开一个新的词法作用域或复合语句块。
- **L2605 EN**: Adds a standalone statement or declaration: `void *next;`.
  **L2605 CN**: 添加一条独立语句或声明：`void *next;`。
- **L2606 EN**: Adds a standalone statement or declaration: `unsigned int length:24;`.
  **L2606 CN**: 添加一条独立语句或声明：`unsigned int length:24;`。
- **L2607 EN**: Adds a standalone statement or declaration: `unsigned int desctype:2;`.
  **L2607 CN**: 添加一条独立语句或声明：`unsigned int desctype:2;`。
- **L2608 EN**: Adds a standalone statement or declaration: `unsigned int dstcomp:1;`.
  **L2608 CN**: 添加一条独立语句或声明：`unsigned int dstcomp:1;`。
- **L2609 EN**: Adds a standalone statement or declaration: `unsigned int srccomp:1;`.
  **L2609 CN**: 添加一条独立语句或声明：`unsigned int srccomp:1;`。
- **L2610 EN**: Adds a standalone statement or declaration: `unsigned int dstbypass:1;`.
  **L2610 CN**: 添加一条独立语句或声明：`unsigned int dstbypass:1;`。
- **L2611 EN**: Adds a standalone statement or declaration: `unsigned int srcbypass:1;`.
  **L2611 CN**: 添加一条独立语句或声明：`unsigned int srcbypass:1;`。
- **L2612 EN**: Adds a standalone statement or declaration: `unsigned int order:1;`.
  **L2612 CN**: 添加一条独立语句或声明：`unsigned int order:1;`。
- **L2613 EN**: Adds a standalone statement or declaration: `unsigned int dstate:1;`.
  **L2613 CN**: 添加一条独立语句或声明：`unsigned int dstate:1;`。
- **L2614 EN**: Adds a standalone statement or declaration: `void *src;`.
  **L2614 CN**: 添加一条独立语句或声明：`void *src;`。
- **L2615 EN**: Adds a standalone statement or declaration: `void *dst;`.
  **L2615 CN**: 添加一条独立语句或声明：`void *dst;`。
- **L2616 EN**: Adds a standalone statement or declaration: `unsigned int allocation:28;`.
  **L2616 CN**: 添加一条独立语句或声明：`unsigned int allocation:28;`。

### Lines 2617-2628

````c
    unsigned int padding:4;
    unsigned int roiwidth:16;
    unsigned int roiheight:16;
    unsigned int srcstride:16;
    unsigned int dststride:16;
    unsigned int srcwidthoffset:16;
    unsigned int dstwidthoffset:16;
} hexagon_udma_descriptor_type1_t;

#pragma pop_macro("B0")

#endif /* !HEXAGON_TYPES_H */
````
- **L2617 EN**: Adds a standalone statement or declaration: `unsigned int padding:4;`.
  **L2617 CN**: 添加一条独立语句或声明：`unsigned int padding:4;`。
- **L2618 EN**: Adds a standalone statement or declaration: `unsigned int roiwidth:16;`.
  **L2618 CN**: 添加一条独立语句或声明：`unsigned int roiwidth:16;`。
- **L2619 EN**: Adds a standalone statement or declaration: `unsigned int roiheight:16;`.
  **L2619 CN**: 添加一条独立语句或声明：`unsigned int roiheight:16;`。
- **L2620 EN**: Adds a standalone statement or declaration: `unsigned int srcstride:16;`.
  **L2620 CN**: 添加一条独立语句或声明：`unsigned int srcstride:16;`。
- **L2621 EN**: Adds a standalone statement or declaration: `unsigned int dststride:16;`.
  **L2621 CN**: 添加一条独立语句或声明：`unsigned int dststride:16;`。
- **L2622 EN**: Adds a standalone statement or declaration: `unsigned int srcwidthoffset:16;`.
  **L2622 CN**: 添加一条独立语句或声明：`unsigned int srcwidthoffset:16;`。
- **L2623 EN**: Adds a standalone statement or declaration: `unsigned int dstwidthoffset:16;`.
  **L2623 CN**: 添加一条独立语句或声明：`unsigned int dstwidthoffset:16;`。
- **L2624 EN**: Adds a standalone statement or declaration: `} hexagon_udma_descriptor_type1_t;`.
  **L2624 CN**: 添加一条独立语句或声明：`} hexagon_udma_descriptor_type1_t;`。
- **L2625 EN**: Blank line separating nearby declarations or logic blocks.
  **L2625 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2626 EN**: Applies a compiler pragma that adjusts parsing, attributes, or macro state: `#pragma pop_macro("B0")`.
  **L2626 CN**: 应用编译器 pragma，调整解析、属性或宏状态：`#pragma pop_macro("B0")`。
- **L2627 EN**: Blank line separating nearby declarations or logic blocks.
  **L2627 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2628 EN**: Closes the current preprocessor conditional block.
  **L2628 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **Hexagon DSP intrinsics / Hexagon DSP intrinsic 接口**
- **Header composition through includes / 通过 include 组合头文件**
- **Macro-based API construction / 基于宏的 API 构造**
- **Compiler-specific attributes / 编译器特定属性**
- **Pragma-controlled behavior / 由 pragma 控制的行为**
- **Type aliasing and ABI shaping / 类型别名与 ABI 约束**
- **Structured data declarations / 结构化数据声明**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**:
  - `hexagon_protos.h`: Provides related header declarations. / 提供相关头文件声明。
- **Conditional macros / 条件宏**: `HEXAGON_TYPES_H`, `__cplusplus`, `__hexagon__`, `__qdsp6__`, `__HVX__`
- **External builtins / 外部 builtin**: No direct builtin forwarding detected. / 未检测到直接的 builtin 转发。
