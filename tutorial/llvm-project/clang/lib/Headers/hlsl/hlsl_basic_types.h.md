# hlsl_basic_types.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/hlsl/hlsl_basic_types.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: HLSL definitions for basic types.
- **Purpose (CN)**: 该头文件主要作用是：HLSL definitions for basic types。
- **Line Count / 行数**: 370

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````c
//===----- hlsl_basic_types.h - HLSL definitions for basic types ----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef _HLSL_HLSL_BASIC_TYPES_H_
#define _HLSL_HLSL_BASIC_TYPES_H_

namespace hlsl {
// built-in scalar data types:

/// \typedef template<typename Ty, int Size> using vector = Ty
/// __attribute__((ext_vector_type(Size)))
///
/// \tparam Ty The base type of the vector may be any builtin integral or
/// floating point type.
/// \tparam Size The size of the vector may be any value between 1 and 4.
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
- **L9 EN**: Starts a preprocessor conditional block: `#ifndef _HLSL_HLSL_BASIC_TYPES_H_`.
  **L9 CN**: 开始一个预处理条件块：`#ifndef _HLSL_HLSL_BASIC_TYPES_H_`。
- **L10 EN**: Defines macro `_HLSL_HLSL_BASIC_TYPES_H_` for conditional compilation, shorthand, or API generation.
  **L10 CN**: 定义宏 `_HLSL_HLSL_BASIC_TYPES_H_`，用于条件编译、简写或 API 生成。
- **L11 EN**: Blank line separating nearby declarations or logic blocks.
  **L11 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L12 EN**: Opens namespace scope `hlsl`.
  **L12 CN**: 打开命名空间作用域 `hlsl`。
- **L13 EN**: Comment explains nearby logic, constraints, or intent: `built-in scalar data types:`.
  **L13 CN**: 注释解释附近代码的逻辑、约束或设计意图：`built-in scalar data types:`。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L15 EN**: Comment explains nearby logic, constraints, or intent: `typedef template<typename Ty, int Size> using vector Ty`.
  **L15 CN**: 注释解释附近代码的逻辑、约束或设计意图：`typedef template<typename Ty, int Size> using vector Ty`。
- **L16 EN**: Comment explains nearby logic, constraints, or intent: `__attribute__((ext_vector_type(Size)))`.
  **L16 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__attribute__((ext_vector_type(Size)))`。
- **L17 EN**: Separator comment used for visual grouping.
  **L17 CN**: 用于视觉分组的分隔注释。
- **L18 EN**: Comment explains nearby logic, constraints, or intent: `tparam Ty The base type of the vector may be any builtin integral or`.
  **L18 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tparam Ty The base type of the vector may be any builtin integral or`。
- **L19 EN**: Comment explains nearby logic, constraints, or intent: `floating point type.`.
  **L19 CN**: 注释解释附近代码的逻辑、约束或设计意图：`floating point type.`。
- **L20 EN**: Comment explains nearby logic, constraints, or intent: `tparam Size The size of the vector may be any value between 1 and 4.`.
  **L20 CN**: 注释解释附近代码的逻辑、约束或设计意图：`tparam Size The size of the vector may be any value between 1 and 4.`。

### Lines 21-40

````c

#ifdef __HLSL_ENABLE_16_BIT
// 16-bit integer.
typedef unsigned short uint16_t;
typedef short int16_t;

// 16-bit floating point.
typedef half float16_t;
#endif

// 32-bit integer.
typedef int int32_t;

// unsigned 32-bit integer.
typedef unsigned int uint;
typedef unsigned int uint32_t;

// 32-bit floating point.
typedef float float32_t;

````
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L22 EN**: Starts a preprocessor conditional block: `#ifdef __HLSL_ENABLE_16_BIT`.
  **L22 CN**: 开始一个预处理条件块：`#ifdef __HLSL_ENABLE_16_BIT`。
- **L23 EN**: Comment explains nearby logic, constraints, or intent: `16-bit integer.`.
  **L23 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit integer.`。
- **L24 EN**: Introduces an alias or helper declaration: `typedef unsigned short uint16_t;`.
  **L24 CN**: 引入一条别名或辅助声明：`typedef unsigned short uint16_t;`。
- **L25 EN**: Introduces an alias or helper declaration: `typedef short int16_t;`.
  **L25 CN**: 引入一条别名或辅助声明：`typedef short int16_t;`。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L27 EN**: Comment explains nearby logic, constraints, or intent: `16-bit floating point.`.
  **L27 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16-bit floating point.`。
- **L28 EN**: Introduces an alias or helper declaration: `typedef half float16_t;`.
  **L28 CN**: 引入一条别名或辅助声明：`typedef half float16_t;`。
- **L29 EN**: Closes the current preprocessor conditional block.
  **L29 CN**: 结束当前预处理条件块。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L31 EN**: Comment explains nearby logic, constraints, or intent: `32-bit integer.`.
  **L31 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32-bit integer.`。
- **L32 EN**: Introduces an alias or helper declaration: `typedef int int32_t;`.
  **L32 CN**: 引入一条别名或辅助声明：`typedef int int32_t;`。
- **L33 EN**: Blank line separating nearby declarations or logic blocks.
  **L33 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L34 EN**: Comment explains nearby logic, constraints, or intent: `unsigned 32-bit integer.`.
  **L34 CN**: 注释解释附近代码的逻辑、约束或设计意图：`unsigned 32-bit integer.`。
- **L35 EN**: Introduces an alias or helper declaration: `typedef unsigned int uint;`.
  **L35 CN**: 引入一条别名或辅助声明：`typedef unsigned int uint;`。
- **L36 EN**: Introduces an alias or helper declaration: `typedef unsigned int uint32_t;`.
  **L36 CN**: 引入一条别名或辅助声明：`typedef unsigned int uint32_t;`。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L38 EN**: Comment explains nearby logic, constraints, or intent: `32-bit floating point.`.
  **L38 CN**: 注释解释附近代码的逻辑、约束或设计意图：`32-bit floating point.`。
- **L39 EN**: Introduces an alias or helper declaration: `typedef float float32_t;`.
  **L39 CN**: 引入一条别名或辅助声明：`typedef float float32_t;`。
- **L40 EN**: Blank line separating nearby declarations or logic blocks.
  **L40 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 41-60

````c
// 64-bit integer.
typedef unsigned long uint64_t;
typedef long int64_t;

// 64-bit floating point
typedef double float64_t;

// built-in vector data types:

#ifdef __HLSL_ENABLE_16_BIT
typedef vector<int16_t, 1> int16_t1;
typedef vector<int16_t, 2> int16_t2;
typedef vector<int16_t, 3> int16_t3;
typedef vector<int16_t, 4> int16_t4;
typedef vector<uint16_t, 1> uint16_t1;
typedef vector<uint16_t, 2> uint16_t2;
typedef vector<uint16_t, 3> uint16_t3;
typedef vector<uint16_t, 4> uint16_t4;
#endif
typedef vector<bool, 1> bool1;
````
- **L41 EN**: Comment explains nearby logic, constraints, or intent: `64-bit integer.`.
  **L41 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64-bit integer.`。
- **L42 EN**: Introduces an alias or helper declaration: `typedef unsigned long uint64_t;`.
  **L42 CN**: 引入一条别名或辅助声明：`typedef unsigned long uint64_t;`。
- **L43 EN**: Introduces an alias or helper declaration: `typedef long int64_t;`.
  **L43 CN**: 引入一条别名或辅助声明：`typedef long int64_t;`。
- **L44 EN**: Blank line separating nearby declarations or logic blocks.
  **L44 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L45 EN**: Comment explains nearby logic, constraints, or intent: `64-bit floating point`.
  **L45 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64-bit floating point`。
- **L46 EN**: Introduces an alias or helper declaration: `typedef double float64_t;`.
  **L46 CN**: 引入一条别名或辅助声明：`typedef double float64_t;`。
- **L47 EN**: Blank line separating nearby declarations or logic blocks.
  **L47 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L48 EN**: Comment explains nearby logic, constraints, or intent: `built-in vector data types:`.
  **L48 CN**: 注释解释附近代码的逻辑、约束或设计意图：`built-in vector data types:`。
- **L49 EN**: Blank line separating nearby declarations or logic blocks.
  **L49 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L50 EN**: Starts a preprocessor conditional block: `#ifdef __HLSL_ENABLE_16_BIT`.
  **L50 CN**: 开始一个预处理条件块：`#ifdef __HLSL_ENABLE_16_BIT`。
- **L51 EN**: Introduces an alias or helper declaration: `typedef vector<int16_t, 1> int16_t1;`.
  **L51 CN**: 引入一条别名或辅助声明：`typedef vector<int16_t, 1> int16_t1;`。
- **L52 EN**: Introduces an alias or helper declaration: `typedef vector<int16_t, 2> int16_t2;`.
  **L52 CN**: 引入一条别名或辅助声明：`typedef vector<int16_t, 2> int16_t2;`。
- **L53 EN**: Introduces an alias or helper declaration: `typedef vector<int16_t, 3> int16_t3;`.
  **L53 CN**: 引入一条别名或辅助声明：`typedef vector<int16_t, 3> int16_t3;`。
- **L54 EN**: Introduces an alias or helper declaration: `typedef vector<int16_t, 4> int16_t4;`.
  **L54 CN**: 引入一条别名或辅助声明：`typedef vector<int16_t, 4> int16_t4;`。
- **L55 EN**: Introduces an alias or helper declaration: `typedef vector<uint16_t, 1> uint16_t1;`.
  **L55 CN**: 引入一条别名或辅助声明：`typedef vector<uint16_t, 1> uint16_t1;`。
- **L56 EN**: Introduces an alias or helper declaration: `typedef vector<uint16_t, 2> uint16_t2;`.
  **L56 CN**: 引入一条别名或辅助声明：`typedef vector<uint16_t, 2> uint16_t2;`。
- **L57 EN**: Introduces an alias or helper declaration: `typedef vector<uint16_t, 3> uint16_t3;`.
  **L57 CN**: 引入一条别名或辅助声明：`typedef vector<uint16_t, 3> uint16_t3;`。
- **L58 EN**: Introduces an alias or helper declaration: `typedef vector<uint16_t, 4> uint16_t4;`.
  **L58 CN**: 引入一条别名或辅助声明：`typedef vector<uint16_t, 4> uint16_t4;`。
- **L59 EN**: Closes the current preprocessor conditional block.
  **L59 CN**: 结束当前预处理条件块。
- **L60 EN**: Introduces an alias or helper declaration: `typedef vector<bool, 1> bool1;`.
  **L60 CN**: 引入一条别名或辅助声明：`typedef vector<bool, 1> bool1;`。

### Lines 61-80

````c
typedef vector<bool, 2> bool2;
typedef vector<bool, 3> bool3;
typedef vector<bool, 4> bool4;
typedef vector<int, 1> int1;
typedef vector<int, 2> int2;
typedef vector<int, 3> int3;
typedef vector<int, 4> int4;
typedef vector<uint, 1> uint1;
typedef vector<uint, 2> uint2;
typedef vector<uint, 3> uint3;
typedef vector<uint, 4> uint4;
typedef vector<int32_t, 1> int32_t1;
typedef vector<int32_t, 2> int32_t2;
typedef vector<int32_t, 3> int32_t3;
typedef vector<int32_t, 4> int32_t4;
typedef vector<uint32_t, 1> uint32_t1;
typedef vector<uint32_t, 2> uint32_t2;
typedef vector<uint32_t, 3> uint32_t3;
typedef vector<uint32_t, 4> uint32_t4;
typedef vector<int64_t, 1> int64_t1;
````
- **L61 EN**: Introduces an alias or helper declaration: `typedef vector<bool, 2> bool2;`.
  **L61 CN**: 引入一条别名或辅助声明：`typedef vector<bool, 2> bool2;`。
- **L62 EN**: Introduces an alias or helper declaration: `typedef vector<bool, 3> bool3;`.
  **L62 CN**: 引入一条别名或辅助声明：`typedef vector<bool, 3> bool3;`。
- **L63 EN**: Introduces an alias or helper declaration: `typedef vector<bool, 4> bool4;`.
  **L63 CN**: 引入一条别名或辅助声明：`typedef vector<bool, 4> bool4;`。
- **L64 EN**: Introduces an alias or helper declaration: `typedef vector<int, 1> int1;`.
  **L64 CN**: 引入一条别名或辅助声明：`typedef vector<int, 1> int1;`。
- **L65 EN**: Introduces an alias or helper declaration: `typedef vector<int, 2> int2;`.
  **L65 CN**: 引入一条别名或辅助声明：`typedef vector<int, 2> int2;`。
- **L66 EN**: Introduces an alias or helper declaration: `typedef vector<int, 3> int3;`.
  **L66 CN**: 引入一条别名或辅助声明：`typedef vector<int, 3> int3;`。
- **L67 EN**: Introduces an alias or helper declaration: `typedef vector<int, 4> int4;`.
  **L67 CN**: 引入一条别名或辅助声明：`typedef vector<int, 4> int4;`。
- **L68 EN**: Introduces an alias or helper declaration: `typedef vector<uint, 1> uint1;`.
  **L68 CN**: 引入一条别名或辅助声明：`typedef vector<uint, 1> uint1;`。
- **L69 EN**: Introduces an alias or helper declaration: `typedef vector<uint, 2> uint2;`.
  **L69 CN**: 引入一条别名或辅助声明：`typedef vector<uint, 2> uint2;`。
- **L70 EN**: Introduces an alias or helper declaration: `typedef vector<uint, 3> uint3;`.
  **L70 CN**: 引入一条别名或辅助声明：`typedef vector<uint, 3> uint3;`。
- **L71 EN**: Introduces an alias or helper declaration: `typedef vector<uint, 4> uint4;`.
  **L71 CN**: 引入一条别名或辅助声明：`typedef vector<uint, 4> uint4;`。
- **L72 EN**: Introduces an alias or helper declaration: `typedef vector<int32_t, 1> int32_t1;`.
  **L72 CN**: 引入一条别名或辅助声明：`typedef vector<int32_t, 1> int32_t1;`。
- **L73 EN**: Introduces an alias or helper declaration: `typedef vector<int32_t, 2> int32_t2;`.
  **L73 CN**: 引入一条别名或辅助声明：`typedef vector<int32_t, 2> int32_t2;`。
- **L74 EN**: Introduces an alias or helper declaration: `typedef vector<int32_t, 3> int32_t3;`.
  **L74 CN**: 引入一条别名或辅助声明：`typedef vector<int32_t, 3> int32_t3;`。
- **L75 EN**: Introduces an alias or helper declaration: `typedef vector<int32_t, 4> int32_t4;`.
  **L75 CN**: 引入一条别名或辅助声明：`typedef vector<int32_t, 4> int32_t4;`。
- **L76 EN**: Introduces an alias or helper declaration: `typedef vector<uint32_t, 1> uint32_t1;`.
  **L76 CN**: 引入一条别名或辅助声明：`typedef vector<uint32_t, 1> uint32_t1;`。
- **L77 EN**: Introduces an alias or helper declaration: `typedef vector<uint32_t, 2> uint32_t2;`.
  **L77 CN**: 引入一条别名或辅助声明：`typedef vector<uint32_t, 2> uint32_t2;`。
- **L78 EN**: Introduces an alias or helper declaration: `typedef vector<uint32_t, 3> uint32_t3;`.
  **L78 CN**: 引入一条别名或辅助声明：`typedef vector<uint32_t, 3> uint32_t3;`。
- **L79 EN**: Introduces an alias or helper declaration: `typedef vector<uint32_t, 4> uint32_t4;`.
  **L79 CN**: 引入一条别名或辅助声明：`typedef vector<uint32_t, 4> uint32_t4;`。
- **L80 EN**: Introduces an alias or helper declaration: `typedef vector<int64_t, 1> int64_t1;`.
  **L80 CN**: 引入一条别名或辅助声明：`typedef vector<int64_t, 1> int64_t1;`。

### Lines 81-100

````c
typedef vector<int64_t, 2> int64_t2;
typedef vector<int64_t, 3> int64_t3;
typedef vector<int64_t, 4> int64_t4;
typedef vector<uint64_t, 1> uint64_t1;
typedef vector<uint64_t, 2> uint64_t2;
typedef vector<uint64_t, 3> uint64_t3;
typedef vector<uint64_t, 4> uint64_t4;

typedef vector<half, 1> half1;
typedef vector<half, 2> half2;
typedef vector<half, 3> half3;
typedef vector<half, 4> half4;
typedef vector<float, 1> float1;
typedef vector<float, 2> float2;
typedef vector<float, 3> float3;
typedef vector<float, 4> float4;
typedef vector<double, 1> double1;
typedef vector<double, 2> double2;
typedef vector<double, 3> double3;
typedef vector<double, 4> double4;
````
- **L81 EN**: Introduces an alias or helper declaration: `typedef vector<int64_t, 2> int64_t2;`.
  **L81 CN**: 引入一条别名或辅助声明：`typedef vector<int64_t, 2> int64_t2;`。
- **L82 EN**: Introduces an alias or helper declaration: `typedef vector<int64_t, 3> int64_t3;`.
  **L82 CN**: 引入一条别名或辅助声明：`typedef vector<int64_t, 3> int64_t3;`。
- **L83 EN**: Introduces an alias or helper declaration: `typedef vector<int64_t, 4> int64_t4;`.
  **L83 CN**: 引入一条别名或辅助声明：`typedef vector<int64_t, 4> int64_t4;`。
- **L84 EN**: Introduces an alias or helper declaration: `typedef vector<uint64_t, 1> uint64_t1;`.
  **L84 CN**: 引入一条别名或辅助声明：`typedef vector<uint64_t, 1> uint64_t1;`。
- **L85 EN**: Introduces an alias or helper declaration: `typedef vector<uint64_t, 2> uint64_t2;`.
  **L85 CN**: 引入一条别名或辅助声明：`typedef vector<uint64_t, 2> uint64_t2;`。
- **L86 EN**: Introduces an alias or helper declaration: `typedef vector<uint64_t, 3> uint64_t3;`.
  **L86 CN**: 引入一条别名或辅助声明：`typedef vector<uint64_t, 3> uint64_t3;`。
- **L87 EN**: Introduces an alias or helper declaration: `typedef vector<uint64_t, 4> uint64_t4;`.
  **L87 CN**: 引入一条别名或辅助声明：`typedef vector<uint64_t, 4> uint64_t4;`。
- **L88 EN**: Blank line separating nearby declarations or logic blocks.
  **L88 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L89 EN**: Introduces an alias or helper declaration: `typedef vector<half, 1> half1;`.
  **L89 CN**: 引入一条别名或辅助声明：`typedef vector<half, 1> half1;`。
- **L90 EN**: Introduces an alias or helper declaration: `typedef vector<half, 2> half2;`.
  **L90 CN**: 引入一条别名或辅助声明：`typedef vector<half, 2> half2;`。
- **L91 EN**: Introduces an alias or helper declaration: `typedef vector<half, 3> half3;`.
  **L91 CN**: 引入一条别名或辅助声明：`typedef vector<half, 3> half3;`。
- **L92 EN**: Introduces an alias or helper declaration: `typedef vector<half, 4> half4;`.
  **L92 CN**: 引入一条别名或辅助声明：`typedef vector<half, 4> half4;`。
- **L93 EN**: Introduces an alias or helper declaration: `typedef vector<float, 1> float1;`.
  **L93 CN**: 引入一条别名或辅助声明：`typedef vector<float, 1> float1;`。
- **L94 EN**: Introduces an alias or helper declaration: `typedef vector<float, 2> float2;`.
  **L94 CN**: 引入一条别名或辅助声明：`typedef vector<float, 2> float2;`。
- **L95 EN**: Introduces an alias or helper declaration: `typedef vector<float, 3> float3;`.
  **L95 CN**: 引入一条别名或辅助声明：`typedef vector<float, 3> float3;`。
- **L96 EN**: Introduces an alias or helper declaration: `typedef vector<float, 4> float4;`.
  **L96 CN**: 引入一条别名或辅助声明：`typedef vector<float, 4> float4;`。
- **L97 EN**: Introduces an alias or helper declaration: `typedef vector<double, 1> double1;`.
  **L97 CN**: 引入一条别名或辅助声明：`typedef vector<double, 1> double1;`。
- **L98 EN**: Introduces an alias or helper declaration: `typedef vector<double, 2> double2;`.
  **L98 CN**: 引入一条别名或辅助声明：`typedef vector<double, 2> double2;`。
- **L99 EN**: Introduces an alias or helper declaration: `typedef vector<double, 3> double3;`.
  **L99 CN**: 引入一条别名或辅助声明：`typedef vector<double, 3> double3;`。
- **L100 EN**: Introduces an alias or helper declaration: `typedef vector<double, 4> double4;`.
  **L100 CN**: 引入一条别名或辅助声明：`typedef vector<double, 4> double4;`。

### Lines 101-120

````c

#ifdef __HLSL_ENABLE_16_BIT
typedef vector<float16_t, 1> float16_t1;
typedef vector<float16_t, 2> float16_t2;
typedef vector<float16_t, 3> float16_t3;
typedef vector<float16_t, 4> float16_t4;
#endif

typedef vector<float32_t, 1> float32_t1;
typedef vector<float32_t, 2> float32_t2;
typedef vector<float32_t, 3> float32_t3;
typedef vector<float32_t, 4> float32_t4;
typedef vector<float64_t, 1> float64_t1;
typedef vector<float64_t, 2> float64_t2;
typedef vector<float64_t, 3> float64_t3;
typedef vector<float64_t, 4> float64_t4;

#ifdef __HLSL_ENABLE_16_BIT
typedef matrix<int16_t, 1, 1> int16_t1x1;
typedef matrix<int16_t, 1, 2> int16_t1x2;
````
- **L101 EN**: Blank line separating nearby declarations or logic blocks.
  **L101 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L102 EN**: Starts a preprocessor conditional block: `#ifdef __HLSL_ENABLE_16_BIT`.
  **L102 CN**: 开始一个预处理条件块：`#ifdef __HLSL_ENABLE_16_BIT`。
- **L103 EN**: Introduces an alias or helper declaration: `typedef vector<float16_t, 1> float16_t1;`.
  **L103 CN**: 引入一条别名或辅助声明：`typedef vector<float16_t, 1> float16_t1;`。
- **L104 EN**: Introduces an alias or helper declaration: `typedef vector<float16_t, 2> float16_t2;`.
  **L104 CN**: 引入一条别名或辅助声明：`typedef vector<float16_t, 2> float16_t2;`。
- **L105 EN**: Introduces an alias or helper declaration: `typedef vector<float16_t, 3> float16_t3;`.
  **L105 CN**: 引入一条别名或辅助声明：`typedef vector<float16_t, 3> float16_t3;`。
- **L106 EN**: Introduces an alias or helper declaration: `typedef vector<float16_t, 4> float16_t4;`.
  **L106 CN**: 引入一条别名或辅助声明：`typedef vector<float16_t, 4> float16_t4;`。
- **L107 EN**: Closes the current preprocessor conditional block.
  **L107 CN**: 结束当前预处理条件块。
- **L108 EN**: Blank line separating nearby declarations or logic blocks.
  **L108 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L109 EN**: Introduces an alias or helper declaration: `typedef vector<float32_t, 1> float32_t1;`.
  **L109 CN**: 引入一条别名或辅助声明：`typedef vector<float32_t, 1> float32_t1;`。
- **L110 EN**: Introduces an alias or helper declaration: `typedef vector<float32_t, 2> float32_t2;`.
  **L110 CN**: 引入一条别名或辅助声明：`typedef vector<float32_t, 2> float32_t2;`。
- **L111 EN**: Introduces an alias or helper declaration: `typedef vector<float32_t, 3> float32_t3;`.
  **L111 CN**: 引入一条别名或辅助声明：`typedef vector<float32_t, 3> float32_t3;`。
- **L112 EN**: Introduces an alias or helper declaration: `typedef vector<float32_t, 4> float32_t4;`.
  **L112 CN**: 引入一条别名或辅助声明：`typedef vector<float32_t, 4> float32_t4;`。
- **L113 EN**: Introduces an alias or helper declaration: `typedef vector<float64_t, 1> float64_t1;`.
  **L113 CN**: 引入一条别名或辅助声明：`typedef vector<float64_t, 1> float64_t1;`。
- **L114 EN**: Introduces an alias or helper declaration: `typedef vector<float64_t, 2> float64_t2;`.
  **L114 CN**: 引入一条别名或辅助声明：`typedef vector<float64_t, 2> float64_t2;`。
- **L115 EN**: Introduces an alias or helper declaration: `typedef vector<float64_t, 3> float64_t3;`.
  **L115 CN**: 引入一条别名或辅助声明：`typedef vector<float64_t, 3> float64_t3;`。
- **L116 EN**: Introduces an alias or helper declaration: `typedef vector<float64_t, 4> float64_t4;`.
  **L116 CN**: 引入一条别名或辅助声明：`typedef vector<float64_t, 4> float64_t4;`。
- **L117 EN**: Blank line separating nearby declarations or logic blocks.
  **L117 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L118 EN**: Starts a preprocessor conditional block: `#ifdef __HLSL_ENABLE_16_BIT`.
  **L118 CN**: 开始一个预处理条件块：`#ifdef __HLSL_ENABLE_16_BIT`。
- **L119 EN**: Introduces an alias or helper declaration: `typedef matrix<int16_t, 1, 1> int16_t1x1;`.
  **L119 CN**: 引入一条别名或辅助声明：`typedef matrix<int16_t, 1, 1> int16_t1x1;`。
- **L120 EN**: Introduces an alias or helper declaration: `typedef matrix<int16_t, 1, 2> int16_t1x2;`.
  **L120 CN**: 引入一条别名或辅助声明：`typedef matrix<int16_t, 1, 2> int16_t1x2;`。

### Lines 121-140

````c
typedef matrix<int16_t, 1, 3> int16_t1x3;
typedef matrix<int16_t, 1, 4> int16_t1x4;
typedef matrix<int16_t, 2, 1> int16_t2x1;
typedef matrix<int16_t, 2, 2> int16_t2x2;
typedef matrix<int16_t, 2, 3> int16_t2x3;
typedef matrix<int16_t, 2, 4> int16_t2x4;
typedef matrix<int16_t, 3, 1> int16_t3x1;
typedef matrix<int16_t, 3, 2> int16_t3x2;
typedef matrix<int16_t, 3, 3> int16_t3x3;
typedef matrix<int16_t, 3, 4> int16_t3x4;
typedef matrix<int16_t, 4, 1> int16_t4x1;
typedef matrix<int16_t, 4, 2> int16_t4x2;
typedef matrix<int16_t, 4, 3> int16_t4x3;
typedef matrix<int16_t, 4, 4> int16_t4x4;
typedef matrix<uint16_t, 1, 1> uint16_t1x1;
typedef matrix<uint16_t, 1, 2> uint16_t1x2;
typedef matrix<uint16_t, 1, 3> uint16_t1x3;
typedef matrix<uint16_t, 1, 4> uint16_t1x4;
typedef matrix<uint16_t, 2, 1> uint16_t2x1;
typedef matrix<uint16_t, 2, 2> uint16_t2x2;
````
- **L121 EN**: Introduces an alias or helper declaration: `typedef matrix<int16_t, 1, 3> int16_t1x3;`.
  **L121 CN**: 引入一条别名或辅助声明：`typedef matrix<int16_t, 1, 3> int16_t1x3;`。
- **L122 EN**: Introduces an alias or helper declaration: `typedef matrix<int16_t, 1, 4> int16_t1x4;`.
  **L122 CN**: 引入一条别名或辅助声明：`typedef matrix<int16_t, 1, 4> int16_t1x4;`。
- **L123 EN**: Introduces an alias or helper declaration: `typedef matrix<int16_t, 2, 1> int16_t2x1;`.
  **L123 CN**: 引入一条别名或辅助声明：`typedef matrix<int16_t, 2, 1> int16_t2x1;`。
- **L124 EN**: Introduces an alias or helper declaration: `typedef matrix<int16_t, 2, 2> int16_t2x2;`.
  **L124 CN**: 引入一条别名或辅助声明：`typedef matrix<int16_t, 2, 2> int16_t2x2;`。
- **L125 EN**: Introduces an alias or helper declaration: `typedef matrix<int16_t, 2, 3> int16_t2x3;`.
  **L125 CN**: 引入一条别名或辅助声明：`typedef matrix<int16_t, 2, 3> int16_t2x3;`。
- **L126 EN**: Introduces an alias or helper declaration: `typedef matrix<int16_t, 2, 4> int16_t2x4;`.
  **L126 CN**: 引入一条别名或辅助声明：`typedef matrix<int16_t, 2, 4> int16_t2x4;`。
- **L127 EN**: Introduces an alias or helper declaration: `typedef matrix<int16_t, 3, 1> int16_t3x1;`.
  **L127 CN**: 引入一条别名或辅助声明：`typedef matrix<int16_t, 3, 1> int16_t3x1;`。
- **L128 EN**: Introduces an alias or helper declaration: `typedef matrix<int16_t, 3, 2> int16_t3x2;`.
  **L128 CN**: 引入一条别名或辅助声明：`typedef matrix<int16_t, 3, 2> int16_t3x2;`。
- **L129 EN**: Introduces an alias or helper declaration: `typedef matrix<int16_t, 3, 3> int16_t3x3;`.
  **L129 CN**: 引入一条别名或辅助声明：`typedef matrix<int16_t, 3, 3> int16_t3x3;`。
- **L130 EN**: Introduces an alias or helper declaration: `typedef matrix<int16_t, 3, 4> int16_t3x4;`.
  **L130 CN**: 引入一条别名或辅助声明：`typedef matrix<int16_t, 3, 4> int16_t3x4;`。
- **L131 EN**: Introduces an alias or helper declaration: `typedef matrix<int16_t, 4, 1> int16_t4x1;`.
  **L131 CN**: 引入一条别名或辅助声明：`typedef matrix<int16_t, 4, 1> int16_t4x1;`。
- **L132 EN**: Introduces an alias or helper declaration: `typedef matrix<int16_t, 4, 2> int16_t4x2;`.
  **L132 CN**: 引入一条别名或辅助声明：`typedef matrix<int16_t, 4, 2> int16_t4x2;`。
- **L133 EN**: Introduces an alias or helper declaration: `typedef matrix<int16_t, 4, 3> int16_t4x3;`.
  **L133 CN**: 引入一条别名或辅助声明：`typedef matrix<int16_t, 4, 3> int16_t4x3;`。
- **L134 EN**: Introduces an alias or helper declaration: `typedef matrix<int16_t, 4, 4> int16_t4x4;`.
  **L134 CN**: 引入一条别名或辅助声明：`typedef matrix<int16_t, 4, 4> int16_t4x4;`。
- **L135 EN**: Introduces an alias or helper declaration: `typedef matrix<uint16_t, 1, 1> uint16_t1x1;`.
  **L135 CN**: 引入一条别名或辅助声明：`typedef matrix<uint16_t, 1, 1> uint16_t1x1;`。
- **L136 EN**: Introduces an alias or helper declaration: `typedef matrix<uint16_t, 1, 2> uint16_t1x2;`.
  **L136 CN**: 引入一条别名或辅助声明：`typedef matrix<uint16_t, 1, 2> uint16_t1x2;`。
- **L137 EN**: Introduces an alias or helper declaration: `typedef matrix<uint16_t, 1, 3> uint16_t1x3;`.
  **L137 CN**: 引入一条别名或辅助声明：`typedef matrix<uint16_t, 1, 3> uint16_t1x3;`。
- **L138 EN**: Introduces an alias or helper declaration: `typedef matrix<uint16_t, 1, 4> uint16_t1x4;`.
  **L138 CN**: 引入一条别名或辅助声明：`typedef matrix<uint16_t, 1, 4> uint16_t1x4;`。
- **L139 EN**: Introduces an alias or helper declaration: `typedef matrix<uint16_t, 2, 1> uint16_t2x1;`.
  **L139 CN**: 引入一条别名或辅助声明：`typedef matrix<uint16_t, 2, 1> uint16_t2x1;`。
- **L140 EN**: Introduces an alias or helper declaration: `typedef matrix<uint16_t, 2, 2> uint16_t2x2;`.
  **L140 CN**: 引入一条别名或辅助声明：`typedef matrix<uint16_t, 2, 2> uint16_t2x2;`。

### Lines 141-160

````c
typedef matrix<uint16_t, 2, 3> uint16_t2x3;
typedef matrix<uint16_t, 2, 4> uint16_t2x4;
typedef matrix<uint16_t, 3, 1> uint16_t3x1;
typedef matrix<uint16_t, 3, 2> uint16_t3x2;
typedef matrix<uint16_t, 3, 3> uint16_t3x3;
typedef matrix<uint16_t, 3, 4> uint16_t3x4;
typedef matrix<uint16_t, 4, 1> uint16_t4x1;
typedef matrix<uint16_t, 4, 2> uint16_t4x2;
typedef matrix<uint16_t, 4, 3> uint16_t4x3;
typedef matrix<uint16_t, 4, 4> uint16_t4x4;
#endif

typedef matrix<bool, 1, 1> bool1x1;
typedef matrix<bool, 1, 2> bool1x2;
typedef matrix<bool, 1, 3> bool1x3;
typedef matrix<bool, 1, 4> bool1x4;
typedef matrix<bool, 2, 1> bool2x1;
typedef matrix<bool, 2, 2> bool2x2;
typedef matrix<bool, 2, 3> bool2x3;
typedef matrix<bool, 2, 4> bool2x4;
````
- **L141 EN**: Introduces an alias or helper declaration: `typedef matrix<uint16_t, 2, 3> uint16_t2x3;`.
  **L141 CN**: 引入一条别名或辅助声明：`typedef matrix<uint16_t, 2, 3> uint16_t2x3;`。
- **L142 EN**: Introduces an alias or helper declaration: `typedef matrix<uint16_t, 2, 4> uint16_t2x4;`.
  **L142 CN**: 引入一条别名或辅助声明：`typedef matrix<uint16_t, 2, 4> uint16_t2x4;`。
- **L143 EN**: Introduces an alias or helper declaration: `typedef matrix<uint16_t, 3, 1> uint16_t3x1;`.
  **L143 CN**: 引入一条别名或辅助声明：`typedef matrix<uint16_t, 3, 1> uint16_t3x1;`。
- **L144 EN**: Introduces an alias or helper declaration: `typedef matrix<uint16_t, 3, 2> uint16_t3x2;`.
  **L144 CN**: 引入一条别名或辅助声明：`typedef matrix<uint16_t, 3, 2> uint16_t3x2;`。
- **L145 EN**: Introduces an alias or helper declaration: `typedef matrix<uint16_t, 3, 3> uint16_t3x3;`.
  **L145 CN**: 引入一条别名或辅助声明：`typedef matrix<uint16_t, 3, 3> uint16_t3x3;`。
- **L146 EN**: Introduces an alias or helper declaration: `typedef matrix<uint16_t, 3, 4> uint16_t3x4;`.
  **L146 CN**: 引入一条别名或辅助声明：`typedef matrix<uint16_t, 3, 4> uint16_t3x4;`。
- **L147 EN**: Introduces an alias or helper declaration: `typedef matrix<uint16_t, 4, 1> uint16_t4x1;`.
  **L147 CN**: 引入一条别名或辅助声明：`typedef matrix<uint16_t, 4, 1> uint16_t4x1;`。
- **L148 EN**: Introduces an alias or helper declaration: `typedef matrix<uint16_t, 4, 2> uint16_t4x2;`.
  **L148 CN**: 引入一条别名或辅助声明：`typedef matrix<uint16_t, 4, 2> uint16_t4x2;`。
- **L149 EN**: Introduces an alias or helper declaration: `typedef matrix<uint16_t, 4, 3> uint16_t4x3;`.
  **L149 CN**: 引入一条别名或辅助声明：`typedef matrix<uint16_t, 4, 3> uint16_t4x3;`。
- **L150 EN**: Introduces an alias or helper declaration: `typedef matrix<uint16_t, 4, 4> uint16_t4x4;`.
  **L150 CN**: 引入一条别名或辅助声明：`typedef matrix<uint16_t, 4, 4> uint16_t4x4;`。
- **L151 EN**: Closes the current preprocessor conditional block.
  **L151 CN**: 结束当前预处理条件块。
- **L152 EN**: Blank line separating nearby declarations or logic blocks.
  **L152 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L153 EN**: Introduces an alias or helper declaration: `typedef matrix<bool, 1, 1> bool1x1;`.
  **L153 CN**: 引入一条别名或辅助声明：`typedef matrix<bool, 1, 1> bool1x1;`。
- **L154 EN**: Introduces an alias or helper declaration: `typedef matrix<bool, 1, 2> bool1x2;`.
  **L154 CN**: 引入一条别名或辅助声明：`typedef matrix<bool, 1, 2> bool1x2;`。
- **L155 EN**: Introduces an alias or helper declaration: `typedef matrix<bool, 1, 3> bool1x3;`.
  **L155 CN**: 引入一条别名或辅助声明：`typedef matrix<bool, 1, 3> bool1x3;`。
- **L156 EN**: Introduces an alias or helper declaration: `typedef matrix<bool, 1, 4> bool1x4;`.
  **L156 CN**: 引入一条别名或辅助声明：`typedef matrix<bool, 1, 4> bool1x4;`。
- **L157 EN**: Introduces an alias or helper declaration: `typedef matrix<bool, 2, 1> bool2x1;`.
  **L157 CN**: 引入一条别名或辅助声明：`typedef matrix<bool, 2, 1> bool2x1;`。
- **L158 EN**: Introduces an alias or helper declaration: `typedef matrix<bool, 2, 2> bool2x2;`.
  **L158 CN**: 引入一条别名或辅助声明：`typedef matrix<bool, 2, 2> bool2x2;`。
- **L159 EN**: Introduces an alias or helper declaration: `typedef matrix<bool, 2, 3> bool2x3;`.
  **L159 CN**: 引入一条别名或辅助声明：`typedef matrix<bool, 2, 3> bool2x3;`。
- **L160 EN**: Introduces an alias or helper declaration: `typedef matrix<bool, 2, 4> bool2x4;`.
  **L160 CN**: 引入一条别名或辅助声明：`typedef matrix<bool, 2, 4> bool2x4;`。

### Lines 161-180

````c
typedef matrix<bool, 3, 1> bool3x1;
typedef matrix<bool, 3, 2> bool3x2;
typedef matrix<bool, 3, 3> bool3x3;
typedef matrix<bool, 3, 4> bool3x4;
typedef matrix<bool, 4, 1> bool4x1;
typedef matrix<bool, 4, 2> bool4x2;
typedef matrix<bool, 4, 3> bool4x3;
typedef matrix<bool, 4, 4> bool4x4;

typedef matrix<int, 1, 1> int1x1;
typedef matrix<int, 1, 2> int1x2;
typedef matrix<int, 1, 3> int1x3;
typedef matrix<int, 1, 4> int1x4;
typedef matrix<int, 2, 1> int2x1;
typedef matrix<int, 2, 2> int2x2;
typedef matrix<int, 2, 3> int2x3;
typedef matrix<int, 2, 4> int2x4;
typedef matrix<int, 3, 1> int3x1;
typedef matrix<int, 3, 2> int3x2;
typedef matrix<int, 3, 3> int3x3;
````
- **L161 EN**: Introduces an alias or helper declaration: `typedef matrix<bool, 3, 1> bool3x1;`.
  **L161 CN**: 引入一条别名或辅助声明：`typedef matrix<bool, 3, 1> bool3x1;`。
- **L162 EN**: Introduces an alias or helper declaration: `typedef matrix<bool, 3, 2> bool3x2;`.
  **L162 CN**: 引入一条别名或辅助声明：`typedef matrix<bool, 3, 2> bool3x2;`。
- **L163 EN**: Introduces an alias or helper declaration: `typedef matrix<bool, 3, 3> bool3x3;`.
  **L163 CN**: 引入一条别名或辅助声明：`typedef matrix<bool, 3, 3> bool3x3;`。
- **L164 EN**: Introduces an alias or helper declaration: `typedef matrix<bool, 3, 4> bool3x4;`.
  **L164 CN**: 引入一条别名或辅助声明：`typedef matrix<bool, 3, 4> bool3x4;`。
- **L165 EN**: Introduces an alias or helper declaration: `typedef matrix<bool, 4, 1> bool4x1;`.
  **L165 CN**: 引入一条别名或辅助声明：`typedef matrix<bool, 4, 1> bool4x1;`。
- **L166 EN**: Introduces an alias or helper declaration: `typedef matrix<bool, 4, 2> bool4x2;`.
  **L166 CN**: 引入一条别名或辅助声明：`typedef matrix<bool, 4, 2> bool4x2;`。
- **L167 EN**: Introduces an alias or helper declaration: `typedef matrix<bool, 4, 3> bool4x3;`.
  **L167 CN**: 引入一条别名或辅助声明：`typedef matrix<bool, 4, 3> bool4x3;`。
- **L168 EN**: Introduces an alias or helper declaration: `typedef matrix<bool, 4, 4> bool4x4;`.
  **L168 CN**: 引入一条别名或辅助声明：`typedef matrix<bool, 4, 4> bool4x4;`。
- **L169 EN**: Blank line separating nearby declarations or logic blocks.
  **L169 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L170 EN**: Introduces an alias or helper declaration: `typedef matrix<int, 1, 1> int1x1;`.
  **L170 CN**: 引入一条别名或辅助声明：`typedef matrix<int, 1, 1> int1x1;`。
- **L171 EN**: Introduces an alias or helper declaration: `typedef matrix<int, 1, 2> int1x2;`.
  **L171 CN**: 引入一条别名或辅助声明：`typedef matrix<int, 1, 2> int1x2;`。
- **L172 EN**: Introduces an alias or helper declaration: `typedef matrix<int, 1, 3> int1x3;`.
  **L172 CN**: 引入一条别名或辅助声明：`typedef matrix<int, 1, 3> int1x3;`。
- **L173 EN**: Introduces an alias or helper declaration: `typedef matrix<int, 1, 4> int1x4;`.
  **L173 CN**: 引入一条别名或辅助声明：`typedef matrix<int, 1, 4> int1x4;`。
- **L174 EN**: Introduces an alias or helper declaration: `typedef matrix<int, 2, 1> int2x1;`.
  **L174 CN**: 引入一条别名或辅助声明：`typedef matrix<int, 2, 1> int2x1;`。
- **L175 EN**: Introduces an alias or helper declaration: `typedef matrix<int, 2, 2> int2x2;`.
  **L175 CN**: 引入一条别名或辅助声明：`typedef matrix<int, 2, 2> int2x2;`。
- **L176 EN**: Introduces an alias or helper declaration: `typedef matrix<int, 2, 3> int2x3;`.
  **L176 CN**: 引入一条别名或辅助声明：`typedef matrix<int, 2, 3> int2x3;`。
- **L177 EN**: Introduces an alias or helper declaration: `typedef matrix<int, 2, 4> int2x4;`.
  **L177 CN**: 引入一条别名或辅助声明：`typedef matrix<int, 2, 4> int2x4;`。
- **L178 EN**: Introduces an alias or helper declaration: `typedef matrix<int, 3, 1> int3x1;`.
  **L178 CN**: 引入一条别名或辅助声明：`typedef matrix<int, 3, 1> int3x1;`。
- **L179 EN**: Introduces an alias or helper declaration: `typedef matrix<int, 3, 2> int3x2;`.
  **L179 CN**: 引入一条别名或辅助声明：`typedef matrix<int, 3, 2> int3x2;`。
- **L180 EN**: Introduces an alias or helper declaration: `typedef matrix<int, 3, 3> int3x3;`.
  **L180 CN**: 引入一条别名或辅助声明：`typedef matrix<int, 3, 3> int3x3;`。

### Lines 181-200

````c
typedef matrix<int, 3, 4> int3x4;
typedef matrix<int, 4, 1> int4x1;
typedef matrix<int, 4, 2> int4x2;
typedef matrix<int, 4, 3> int4x3;
typedef matrix<int, 4, 4> int4x4;
typedef matrix<uint, 1, 1> uint1x1;
typedef matrix<uint, 1, 2> uint1x2;
typedef matrix<uint, 1, 3> uint1x3;
typedef matrix<uint, 1, 4> uint1x4;
typedef matrix<uint, 2, 1> uint2x1;
typedef matrix<uint, 2, 2> uint2x2;
typedef matrix<uint, 2, 3> uint2x3;
typedef matrix<uint, 2, 4> uint2x4;
typedef matrix<uint, 3, 1> uint3x1;
typedef matrix<uint, 3, 2> uint3x2;
typedef matrix<uint, 3, 3> uint3x3;
typedef matrix<uint, 3, 4> uint3x4;
typedef matrix<uint, 4, 1> uint4x1;
typedef matrix<uint, 4, 2> uint4x2;
typedef matrix<uint, 4, 3> uint4x3;
````
- **L181 EN**: Introduces an alias or helper declaration: `typedef matrix<int, 3, 4> int3x4;`.
  **L181 CN**: 引入一条别名或辅助声明：`typedef matrix<int, 3, 4> int3x4;`。
- **L182 EN**: Introduces an alias or helper declaration: `typedef matrix<int, 4, 1> int4x1;`.
  **L182 CN**: 引入一条别名或辅助声明：`typedef matrix<int, 4, 1> int4x1;`。
- **L183 EN**: Introduces an alias or helper declaration: `typedef matrix<int, 4, 2> int4x2;`.
  **L183 CN**: 引入一条别名或辅助声明：`typedef matrix<int, 4, 2> int4x2;`。
- **L184 EN**: Introduces an alias or helper declaration: `typedef matrix<int, 4, 3> int4x3;`.
  **L184 CN**: 引入一条别名或辅助声明：`typedef matrix<int, 4, 3> int4x3;`。
- **L185 EN**: Introduces an alias or helper declaration: `typedef matrix<int, 4, 4> int4x4;`.
  **L185 CN**: 引入一条别名或辅助声明：`typedef matrix<int, 4, 4> int4x4;`。
- **L186 EN**: Introduces an alias or helper declaration: `typedef matrix<uint, 1, 1> uint1x1;`.
  **L186 CN**: 引入一条别名或辅助声明：`typedef matrix<uint, 1, 1> uint1x1;`。
- **L187 EN**: Introduces an alias or helper declaration: `typedef matrix<uint, 1, 2> uint1x2;`.
  **L187 CN**: 引入一条别名或辅助声明：`typedef matrix<uint, 1, 2> uint1x2;`。
- **L188 EN**: Introduces an alias or helper declaration: `typedef matrix<uint, 1, 3> uint1x3;`.
  **L188 CN**: 引入一条别名或辅助声明：`typedef matrix<uint, 1, 3> uint1x3;`。
- **L189 EN**: Introduces an alias or helper declaration: `typedef matrix<uint, 1, 4> uint1x4;`.
  **L189 CN**: 引入一条别名或辅助声明：`typedef matrix<uint, 1, 4> uint1x4;`。
- **L190 EN**: Introduces an alias or helper declaration: `typedef matrix<uint, 2, 1> uint2x1;`.
  **L190 CN**: 引入一条别名或辅助声明：`typedef matrix<uint, 2, 1> uint2x1;`。
- **L191 EN**: Introduces an alias or helper declaration: `typedef matrix<uint, 2, 2> uint2x2;`.
  **L191 CN**: 引入一条别名或辅助声明：`typedef matrix<uint, 2, 2> uint2x2;`。
- **L192 EN**: Introduces an alias or helper declaration: `typedef matrix<uint, 2, 3> uint2x3;`.
  **L192 CN**: 引入一条别名或辅助声明：`typedef matrix<uint, 2, 3> uint2x3;`。
- **L193 EN**: Introduces an alias or helper declaration: `typedef matrix<uint, 2, 4> uint2x4;`.
  **L193 CN**: 引入一条别名或辅助声明：`typedef matrix<uint, 2, 4> uint2x4;`。
- **L194 EN**: Introduces an alias or helper declaration: `typedef matrix<uint, 3, 1> uint3x1;`.
  **L194 CN**: 引入一条别名或辅助声明：`typedef matrix<uint, 3, 1> uint3x1;`。
- **L195 EN**: Introduces an alias or helper declaration: `typedef matrix<uint, 3, 2> uint3x2;`.
  **L195 CN**: 引入一条别名或辅助声明：`typedef matrix<uint, 3, 2> uint3x2;`。
- **L196 EN**: Introduces an alias or helper declaration: `typedef matrix<uint, 3, 3> uint3x3;`.
  **L196 CN**: 引入一条别名或辅助声明：`typedef matrix<uint, 3, 3> uint3x3;`。
- **L197 EN**: Introduces an alias or helper declaration: `typedef matrix<uint, 3, 4> uint3x4;`.
  **L197 CN**: 引入一条别名或辅助声明：`typedef matrix<uint, 3, 4> uint3x4;`。
- **L198 EN**: Introduces an alias or helper declaration: `typedef matrix<uint, 4, 1> uint4x1;`.
  **L198 CN**: 引入一条别名或辅助声明：`typedef matrix<uint, 4, 1> uint4x1;`。
- **L199 EN**: Introduces an alias or helper declaration: `typedef matrix<uint, 4, 2> uint4x2;`.
  **L199 CN**: 引入一条别名或辅助声明：`typedef matrix<uint, 4, 2> uint4x2;`。
- **L200 EN**: Introduces an alias or helper declaration: `typedef matrix<uint, 4, 3> uint4x3;`.
  **L200 CN**: 引入一条别名或辅助声明：`typedef matrix<uint, 4, 3> uint4x3;`。

### Lines 201-220

````c
typedef matrix<uint, 4, 4> uint4x4;
typedef matrix<int32_t, 1, 1> int32_t1x1;
typedef matrix<int32_t, 1, 2> int32_t1x2;
typedef matrix<int32_t, 1, 3> int32_t1x3;
typedef matrix<int32_t, 1, 4> int32_t1x4;
typedef matrix<int32_t, 2, 1> int32_t2x1;
typedef matrix<int32_t, 2, 2> int32_t2x2;
typedef matrix<int32_t, 2, 3> int32_t2x3;
typedef matrix<int32_t, 2, 4> int32_t2x4;
typedef matrix<int32_t, 3, 1> int32_t3x1;
typedef matrix<int32_t, 3, 2> int32_t3x2;
typedef matrix<int32_t, 3, 3> int32_t3x3;
typedef matrix<int32_t, 3, 4> int32_t3x4;
typedef matrix<int32_t, 4, 1> int32_t4x1;
typedef matrix<int32_t, 4, 2> int32_t4x2;
typedef matrix<int32_t, 4, 3> int32_t4x3;
typedef matrix<int32_t, 4, 4> int32_t4x4;
typedef matrix<uint32_t, 1, 1> uint32_t1x1;
typedef matrix<uint32_t, 1, 2> uint32_t1x2;
typedef matrix<uint32_t, 1, 3> uint32_t1x3;
````
- **L201 EN**: Introduces an alias or helper declaration: `typedef matrix<uint, 4, 4> uint4x4;`.
  **L201 CN**: 引入一条别名或辅助声明：`typedef matrix<uint, 4, 4> uint4x4;`。
- **L202 EN**: Introduces an alias or helper declaration: `typedef matrix<int32_t, 1, 1> int32_t1x1;`.
  **L202 CN**: 引入一条别名或辅助声明：`typedef matrix<int32_t, 1, 1> int32_t1x1;`。
- **L203 EN**: Introduces an alias or helper declaration: `typedef matrix<int32_t, 1, 2> int32_t1x2;`.
  **L203 CN**: 引入一条别名或辅助声明：`typedef matrix<int32_t, 1, 2> int32_t1x2;`。
- **L204 EN**: Introduces an alias or helper declaration: `typedef matrix<int32_t, 1, 3> int32_t1x3;`.
  **L204 CN**: 引入一条别名或辅助声明：`typedef matrix<int32_t, 1, 3> int32_t1x3;`。
- **L205 EN**: Introduces an alias or helper declaration: `typedef matrix<int32_t, 1, 4> int32_t1x4;`.
  **L205 CN**: 引入一条别名或辅助声明：`typedef matrix<int32_t, 1, 4> int32_t1x4;`。
- **L206 EN**: Introduces an alias or helper declaration: `typedef matrix<int32_t, 2, 1> int32_t2x1;`.
  **L206 CN**: 引入一条别名或辅助声明：`typedef matrix<int32_t, 2, 1> int32_t2x1;`。
- **L207 EN**: Introduces an alias or helper declaration: `typedef matrix<int32_t, 2, 2> int32_t2x2;`.
  **L207 CN**: 引入一条别名或辅助声明：`typedef matrix<int32_t, 2, 2> int32_t2x2;`。
- **L208 EN**: Introduces an alias or helper declaration: `typedef matrix<int32_t, 2, 3> int32_t2x3;`.
  **L208 CN**: 引入一条别名或辅助声明：`typedef matrix<int32_t, 2, 3> int32_t2x3;`。
- **L209 EN**: Introduces an alias or helper declaration: `typedef matrix<int32_t, 2, 4> int32_t2x4;`.
  **L209 CN**: 引入一条别名或辅助声明：`typedef matrix<int32_t, 2, 4> int32_t2x4;`。
- **L210 EN**: Introduces an alias or helper declaration: `typedef matrix<int32_t, 3, 1> int32_t3x1;`.
  **L210 CN**: 引入一条别名或辅助声明：`typedef matrix<int32_t, 3, 1> int32_t3x1;`。
- **L211 EN**: Introduces an alias or helper declaration: `typedef matrix<int32_t, 3, 2> int32_t3x2;`.
  **L211 CN**: 引入一条别名或辅助声明：`typedef matrix<int32_t, 3, 2> int32_t3x2;`。
- **L212 EN**: Introduces an alias or helper declaration: `typedef matrix<int32_t, 3, 3> int32_t3x3;`.
  **L212 CN**: 引入一条别名或辅助声明：`typedef matrix<int32_t, 3, 3> int32_t3x3;`。
- **L213 EN**: Introduces an alias or helper declaration: `typedef matrix<int32_t, 3, 4> int32_t3x4;`.
  **L213 CN**: 引入一条别名或辅助声明：`typedef matrix<int32_t, 3, 4> int32_t3x4;`。
- **L214 EN**: Introduces an alias or helper declaration: `typedef matrix<int32_t, 4, 1> int32_t4x1;`.
  **L214 CN**: 引入一条别名或辅助声明：`typedef matrix<int32_t, 4, 1> int32_t4x1;`。
- **L215 EN**: Introduces an alias or helper declaration: `typedef matrix<int32_t, 4, 2> int32_t4x2;`.
  **L215 CN**: 引入一条别名或辅助声明：`typedef matrix<int32_t, 4, 2> int32_t4x2;`。
- **L216 EN**: Introduces an alias or helper declaration: `typedef matrix<int32_t, 4, 3> int32_t4x3;`.
  **L216 CN**: 引入一条别名或辅助声明：`typedef matrix<int32_t, 4, 3> int32_t4x3;`。
- **L217 EN**: Introduces an alias or helper declaration: `typedef matrix<int32_t, 4, 4> int32_t4x4;`.
  **L217 CN**: 引入一条别名或辅助声明：`typedef matrix<int32_t, 4, 4> int32_t4x4;`。
- **L218 EN**: Introduces an alias or helper declaration: `typedef matrix<uint32_t, 1, 1> uint32_t1x1;`.
  **L218 CN**: 引入一条别名或辅助声明：`typedef matrix<uint32_t, 1, 1> uint32_t1x1;`。
- **L219 EN**: Introduces an alias or helper declaration: `typedef matrix<uint32_t, 1, 2> uint32_t1x2;`.
  **L219 CN**: 引入一条别名或辅助声明：`typedef matrix<uint32_t, 1, 2> uint32_t1x2;`。
- **L220 EN**: Introduces an alias or helper declaration: `typedef matrix<uint32_t, 1, 3> uint32_t1x3;`.
  **L220 CN**: 引入一条别名或辅助声明：`typedef matrix<uint32_t, 1, 3> uint32_t1x3;`。

### Lines 221-240

````c
typedef matrix<uint32_t, 1, 4> uint32_t1x4;
typedef matrix<uint32_t, 2, 1> uint32_t2x1;
typedef matrix<uint32_t, 2, 2> uint32_t2x2;
typedef matrix<uint32_t, 2, 3> uint32_t2x3;
typedef matrix<uint32_t, 2, 4> uint32_t2x4;
typedef matrix<uint32_t, 3, 1> uint32_t3x1;
typedef matrix<uint32_t, 3, 2> uint32_t3x2;
typedef matrix<uint32_t, 3, 3> uint32_t3x3;
typedef matrix<uint32_t, 3, 4> uint32_t3x4;
typedef matrix<uint32_t, 4, 1> uint32_t4x1;
typedef matrix<uint32_t, 4, 2> uint32_t4x2;
typedef matrix<uint32_t, 4, 3> uint32_t4x3;
typedef matrix<uint32_t, 4, 4> uint32_t4x4;
typedef matrix<int64_t, 1, 1> int64_t1x1;
typedef matrix<int64_t, 1, 2> int64_t1x2;
typedef matrix<int64_t, 1, 3> int64_t1x3;
typedef matrix<int64_t, 1, 4> int64_t1x4;
typedef matrix<int64_t, 2, 1> int64_t2x1;
typedef matrix<int64_t, 2, 2> int64_t2x2;
typedef matrix<int64_t, 2, 3> int64_t2x3;
````
- **L221 EN**: Introduces an alias or helper declaration: `typedef matrix<uint32_t, 1, 4> uint32_t1x4;`.
  **L221 CN**: 引入一条别名或辅助声明：`typedef matrix<uint32_t, 1, 4> uint32_t1x4;`。
- **L222 EN**: Introduces an alias or helper declaration: `typedef matrix<uint32_t, 2, 1> uint32_t2x1;`.
  **L222 CN**: 引入一条别名或辅助声明：`typedef matrix<uint32_t, 2, 1> uint32_t2x1;`。
- **L223 EN**: Introduces an alias or helper declaration: `typedef matrix<uint32_t, 2, 2> uint32_t2x2;`.
  **L223 CN**: 引入一条别名或辅助声明：`typedef matrix<uint32_t, 2, 2> uint32_t2x2;`。
- **L224 EN**: Introduces an alias or helper declaration: `typedef matrix<uint32_t, 2, 3> uint32_t2x3;`.
  **L224 CN**: 引入一条别名或辅助声明：`typedef matrix<uint32_t, 2, 3> uint32_t2x3;`。
- **L225 EN**: Introduces an alias or helper declaration: `typedef matrix<uint32_t, 2, 4> uint32_t2x4;`.
  **L225 CN**: 引入一条别名或辅助声明：`typedef matrix<uint32_t, 2, 4> uint32_t2x4;`。
- **L226 EN**: Introduces an alias or helper declaration: `typedef matrix<uint32_t, 3, 1> uint32_t3x1;`.
  **L226 CN**: 引入一条别名或辅助声明：`typedef matrix<uint32_t, 3, 1> uint32_t3x1;`。
- **L227 EN**: Introduces an alias or helper declaration: `typedef matrix<uint32_t, 3, 2> uint32_t3x2;`.
  **L227 CN**: 引入一条别名或辅助声明：`typedef matrix<uint32_t, 3, 2> uint32_t3x2;`。
- **L228 EN**: Introduces an alias or helper declaration: `typedef matrix<uint32_t, 3, 3> uint32_t3x3;`.
  **L228 CN**: 引入一条别名或辅助声明：`typedef matrix<uint32_t, 3, 3> uint32_t3x3;`。
- **L229 EN**: Introduces an alias or helper declaration: `typedef matrix<uint32_t, 3, 4> uint32_t3x4;`.
  **L229 CN**: 引入一条别名或辅助声明：`typedef matrix<uint32_t, 3, 4> uint32_t3x4;`。
- **L230 EN**: Introduces an alias or helper declaration: `typedef matrix<uint32_t, 4, 1> uint32_t4x1;`.
  **L230 CN**: 引入一条别名或辅助声明：`typedef matrix<uint32_t, 4, 1> uint32_t4x1;`。
- **L231 EN**: Introduces an alias or helper declaration: `typedef matrix<uint32_t, 4, 2> uint32_t4x2;`.
  **L231 CN**: 引入一条别名或辅助声明：`typedef matrix<uint32_t, 4, 2> uint32_t4x2;`。
- **L232 EN**: Introduces an alias or helper declaration: `typedef matrix<uint32_t, 4, 3> uint32_t4x3;`.
  **L232 CN**: 引入一条别名或辅助声明：`typedef matrix<uint32_t, 4, 3> uint32_t4x3;`。
- **L233 EN**: Introduces an alias or helper declaration: `typedef matrix<uint32_t, 4, 4> uint32_t4x4;`.
  **L233 CN**: 引入一条别名或辅助声明：`typedef matrix<uint32_t, 4, 4> uint32_t4x4;`。
- **L234 EN**: Introduces an alias or helper declaration: `typedef matrix<int64_t, 1, 1> int64_t1x1;`.
  **L234 CN**: 引入一条别名或辅助声明：`typedef matrix<int64_t, 1, 1> int64_t1x1;`。
- **L235 EN**: Introduces an alias or helper declaration: `typedef matrix<int64_t, 1, 2> int64_t1x2;`.
  **L235 CN**: 引入一条别名或辅助声明：`typedef matrix<int64_t, 1, 2> int64_t1x2;`。
- **L236 EN**: Introduces an alias or helper declaration: `typedef matrix<int64_t, 1, 3> int64_t1x3;`.
  **L236 CN**: 引入一条别名或辅助声明：`typedef matrix<int64_t, 1, 3> int64_t1x3;`。
- **L237 EN**: Introduces an alias or helper declaration: `typedef matrix<int64_t, 1, 4> int64_t1x4;`.
  **L237 CN**: 引入一条别名或辅助声明：`typedef matrix<int64_t, 1, 4> int64_t1x4;`。
- **L238 EN**: Introduces an alias or helper declaration: `typedef matrix<int64_t, 2, 1> int64_t2x1;`.
  **L238 CN**: 引入一条别名或辅助声明：`typedef matrix<int64_t, 2, 1> int64_t2x1;`。
- **L239 EN**: Introduces an alias or helper declaration: `typedef matrix<int64_t, 2, 2> int64_t2x2;`.
  **L239 CN**: 引入一条别名或辅助声明：`typedef matrix<int64_t, 2, 2> int64_t2x2;`。
- **L240 EN**: Introduces an alias or helper declaration: `typedef matrix<int64_t, 2, 3> int64_t2x3;`.
  **L240 CN**: 引入一条别名或辅助声明：`typedef matrix<int64_t, 2, 3> int64_t2x3;`。

### Lines 241-260

````c
typedef matrix<int64_t, 2, 4> int64_t2x4;
typedef matrix<int64_t, 3, 1> int64_t3x1;
typedef matrix<int64_t, 3, 2> int64_t3x2;
typedef matrix<int64_t, 3, 3> int64_t3x3;
typedef matrix<int64_t, 3, 4> int64_t3x4;
typedef matrix<int64_t, 4, 1> int64_t4x1;
typedef matrix<int64_t, 4, 2> int64_t4x2;
typedef matrix<int64_t, 4, 3> int64_t4x3;
typedef matrix<int64_t, 4, 4> int64_t4x4;
typedef matrix<uint64_t, 1, 1> uint64_t1x1;
typedef matrix<uint64_t, 1, 2> uint64_t1x2;
typedef matrix<uint64_t, 1, 3> uint64_t1x3;
typedef matrix<uint64_t, 1, 4> uint64_t1x4;
typedef matrix<uint64_t, 2, 1> uint64_t2x1;
typedef matrix<uint64_t, 2, 2> uint64_t2x2;
typedef matrix<uint64_t, 2, 3> uint64_t2x3;
typedef matrix<uint64_t, 2, 4> uint64_t2x4;
typedef matrix<uint64_t, 3, 1> uint64_t3x1;
typedef matrix<uint64_t, 3, 2> uint64_t3x2;
typedef matrix<uint64_t, 3, 3> uint64_t3x3;
````
- **L241 EN**: Introduces an alias or helper declaration: `typedef matrix<int64_t, 2, 4> int64_t2x4;`.
  **L241 CN**: 引入一条别名或辅助声明：`typedef matrix<int64_t, 2, 4> int64_t2x4;`。
- **L242 EN**: Introduces an alias or helper declaration: `typedef matrix<int64_t, 3, 1> int64_t3x1;`.
  **L242 CN**: 引入一条别名或辅助声明：`typedef matrix<int64_t, 3, 1> int64_t3x1;`。
- **L243 EN**: Introduces an alias or helper declaration: `typedef matrix<int64_t, 3, 2> int64_t3x2;`.
  **L243 CN**: 引入一条别名或辅助声明：`typedef matrix<int64_t, 3, 2> int64_t3x2;`。
- **L244 EN**: Introduces an alias or helper declaration: `typedef matrix<int64_t, 3, 3> int64_t3x3;`.
  **L244 CN**: 引入一条别名或辅助声明：`typedef matrix<int64_t, 3, 3> int64_t3x3;`。
- **L245 EN**: Introduces an alias or helper declaration: `typedef matrix<int64_t, 3, 4> int64_t3x4;`.
  **L245 CN**: 引入一条别名或辅助声明：`typedef matrix<int64_t, 3, 4> int64_t3x4;`。
- **L246 EN**: Introduces an alias or helper declaration: `typedef matrix<int64_t, 4, 1> int64_t4x1;`.
  **L246 CN**: 引入一条别名或辅助声明：`typedef matrix<int64_t, 4, 1> int64_t4x1;`。
- **L247 EN**: Introduces an alias or helper declaration: `typedef matrix<int64_t, 4, 2> int64_t4x2;`.
  **L247 CN**: 引入一条别名或辅助声明：`typedef matrix<int64_t, 4, 2> int64_t4x2;`。
- **L248 EN**: Introduces an alias or helper declaration: `typedef matrix<int64_t, 4, 3> int64_t4x3;`.
  **L248 CN**: 引入一条别名或辅助声明：`typedef matrix<int64_t, 4, 3> int64_t4x3;`。
- **L249 EN**: Introduces an alias or helper declaration: `typedef matrix<int64_t, 4, 4> int64_t4x4;`.
  **L249 CN**: 引入一条别名或辅助声明：`typedef matrix<int64_t, 4, 4> int64_t4x4;`。
- **L250 EN**: Introduces an alias or helper declaration: `typedef matrix<uint64_t, 1, 1> uint64_t1x1;`.
  **L250 CN**: 引入一条别名或辅助声明：`typedef matrix<uint64_t, 1, 1> uint64_t1x1;`。
- **L251 EN**: Introduces an alias or helper declaration: `typedef matrix<uint64_t, 1, 2> uint64_t1x2;`.
  **L251 CN**: 引入一条别名或辅助声明：`typedef matrix<uint64_t, 1, 2> uint64_t1x2;`。
- **L252 EN**: Introduces an alias or helper declaration: `typedef matrix<uint64_t, 1, 3> uint64_t1x3;`.
  **L252 CN**: 引入一条别名或辅助声明：`typedef matrix<uint64_t, 1, 3> uint64_t1x3;`。
- **L253 EN**: Introduces an alias or helper declaration: `typedef matrix<uint64_t, 1, 4> uint64_t1x4;`.
  **L253 CN**: 引入一条别名或辅助声明：`typedef matrix<uint64_t, 1, 4> uint64_t1x4;`。
- **L254 EN**: Introduces an alias or helper declaration: `typedef matrix<uint64_t, 2, 1> uint64_t2x1;`.
  **L254 CN**: 引入一条别名或辅助声明：`typedef matrix<uint64_t, 2, 1> uint64_t2x1;`。
- **L255 EN**: Introduces an alias or helper declaration: `typedef matrix<uint64_t, 2, 2> uint64_t2x2;`.
  **L255 CN**: 引入一条别名或辅助声明：`typedef matrix<uint64_t, 2, 2> uint64_t2x2;`。
- **L256 EN**: Introduces an alias or helper declaration: `typedef matrix<uint64_t, 2, 3> uint64_t2x3;`.
  **L256 CN**: 引入一条别名或辅助声明：`typedef matrix<uint64_t, 2, 3> uint64_t2x3;`。
- **L257 EN**: Introduces an alias or helper declaration: `typedef matrix<uint64_t, 2, 4> uint64_t2x4;`.
  **L257 CN**: 引入一条别名或辅助声明：`typedef matrix<uint64_t, 2, 4> uint64_t2x4;`。
- **L258 EN**: Introduces an alias or helper declaration: `typedef matrix<uint64_t, 3, 1> uint64_t3x1;`.
  **L258 CN**: 引入一条别名或辅助声明：`typedef matrix<uint64_t, 3, 1> uint64_t3x1;`。
- **L259 EN**: Introduces an alias or helper declaration: `typedef matrix<uint64_t, 3, 2> uint64_t3x2;`.
  **L259 CN**: 引入一条别名或辅助声明：`typedef matrix<uint64_t, 3, 2> uint64_t3x2;`。
- **L260 EN**: Introduces an alias or helper declaration: `typedef matrix<uint64_t, 3, 3> uint64_t3x3;`.
  **L260 CN**: 引入一条别名或辅助声明：`typedef matrix<uint64_t, 3, 3> uint64_t3x3;`。

### Lines 261-280

````c
typedef matrix<uint64_t, 3, 4> uint64_t3x4;
typedef matrix<uint64_t, 4, 1> uint64_t4x1;
typedef matrix<uint64_t, 4, 2> uint64_t4x2;
typedef matrix<uint64_t, 4, 3> uint64_t4x3;
typedef matrix<uint64_t, 4, 4> uint64_t4x4;

typedef matrix<half, 1, 1> half1x1;
typedef matrix<half, 1, 2> half1x2;
typedef matrix<half, 1, 3> half1x3;
typedef matrix<half, 1, 4> half1x4;
typedef matrix<half, 2, 1> half2x1;
typedef matrix<half, 2, 2> half2x2;
typedef matrix<half, 2, 3> half2x3;
typedef matrix<half, 2, 4> half2x4;
typedef matrix<half, 3, 1> half3x1;
typedef matrix<half, 3, 2> half3x2;
typedef matrix<half, 3, 3> half3x3;
typedef matrix<half, 3, 4> half3x4;
typedef matrix<half, 4, 1> half4x1;
typedef matrix<half, 4, 2> half4x2;
````
- **L261 EN**: Introduces an alias or helper declaration: `typedef matrix<uint64_t, 3, 4> uint64_t3x4;`.
  **L261 CN**: 引入一条别名或辅助声明：`typedef matrix<uint64_t, 3, 4> uint64_t3x4;`。
- **L262 EN**: Introduces an alias or helper declaration: `typedef matrix<uint64_t, 4, 1> uint64_t4x1;`.
  **L262 CN**: 引入一条别名或辅助声明：`typedef matrix<uint64_t, 4, 1> uint64_t4x1;`。
- **L263 EN**: Introduces an alias or helper declaration: `typedef matrix<uint64_t, 4, 2> uint64_t4x2;`.
  **L263 CN**: 引入一条别名或辅助声明：`typedef matrix<uint64_t, 4, 2> uint64_t4x2;`。
- **L264 EN**: Introduces an alias or helper declaration: `typedef matrix<uint64_t, 4, 3> uint64_t4x3;`.
  **L264 CN**: 引入一条别名或辅助声明：`typedef matrix<uint64_t, 4, 3> uint64_t4x3;`。
- **L265 EN**: Introduces an alias or helper declaration: `typedef matrix<uint64_t, 4, 4> uint64_t4x4;`.
  **L265 CN**: 引入一条别名或辅助声明：`typedef matrix<uint64_t, 4, 4> uint64_t4x4;`。
- **L266 EN**: Blank line separating nearby declarations or logic blocks.
  **L266 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L267 EN**: Introduces an alias or helper declaration: `typedef matrix<half, 1, 1> half1x1;`.
  **L267 CN**: 引入一条别名或辅助声明：`typedef matrix<half, 1, 1> half1x1;`。
- **L268 EN**: Introduces an alias or helper declaration: `typedef matrix<half, 1, 2> half1x2;`.
  **L268 CN**: 引入一条别名或辅助声明：`typedef matrix<half, 1, 2> half1x2;`。
- **L269 EN**: Introduces an alias or helper declaration: `typedef matrix<half, 1, 3> half1x3;`.
  **L269 CN**: 引入一条别名或辅助声明：`typedef matrix<half, 1, 3> half1x3;`。
- **L270 EN**: Introduces an alias or helper declaration: `typedef matrix<half, 1, 4> half1x4;`.
  **L270 CN**: 引入一条别名或辅助声明：`typedef matrix<half, 1, 4> half1x4;`。
- **L271 EN**: Introduces an alias or helper declaration: `typedef matrix<half, 2, 1> half2x1;`.
  **L271 CN**: 引入一条别名或辅助声明：`typedef matrix<half, 2, 1> half2x1;`。
- **L272 EN**: Introduces an alias or helper declaration: `typedef matrix<half, 2, 2> half2x2;`.
  **L272 CN**: 引入一条别名或辅助声明：`typedef matrix<half, 2, 2> half2x2;`。
- **L273 EN**: Introduces an alias or helper declaration: `typedef matrix<half, 2, 3> half2x3;`.
  **L273 CN**: 引入一条别名或辅助声明：`typedef matrix<half, 2, 3> half2x3;`。
- **L274 EN**: Introduces an alias or helper declaration: `typedef matrix<half, 2, 4> half2x4;`.
  **L274 CN**: 引入一条别名或辅助声明：`typedef matrix<half, 2, 4> half2x4;`。
- **L275 EN**: Introduces an alias or helper declaration: `typedef matrix<half, 3, 1> half3x1;`.
  **L275 CN**: 引入一条别名或辅助声明：`typedef matrix<half, 3, 1> half3x1;`。
- **L276 EN**: Introduces an alias or helper declaration: `typedef matrix<half, 3, 2> half3x2;`.
  **L276 CN**: 引入一条别名或辅助声明：`typedef matrix<half, 3, 2> half3x2;`。
- **L277 EN**: Introduces an alias or helper declaration: `typedef matrix<half, 3, 3> half3x3;`.
  **L277 CN**: 引入一条别名或辅助声明：`typedef matrix<half, 3, 3> half3x3;`。
- **L278 EN**: Introduces an alias or helper declaration: `typedef matrix<half, 3, 4> half3x4;`.
  **L278 CN**: 引入一条别名或辅助声明：`typedef matrix<half, 3, 4> half3x4;`。
- **L279 EN**: Introduces an alias or helper declaration: `typedef matrix<half, 4, 1> half4x1;`.
  **L279 CN**: 引入一条别名或辅助声明：`typedef matrix<half, 4, 1> half4x1;`。
- **L280 EN**: Introduces an alias or helper declaration: `typedef matrix<half, 4, 2> half4x2;`.
  **L280 CN**: 引入一条别名或辅助声明：`typedef matrix<half, 4, 2> half4x2;`。

### Lines 281-300

````c
typedef matrix<half, 4, 3> half4x3;
typedef matrix<half, 4, 4> half4x4;
typedef matrix<float, 1, 1> float1x1;
typedef matrix<float, 1, 2> float1x2;
typedef matrix<float, 1, 3> float1x3;
typedef matrix<float, 1, 4> float1x4;
typedef matrix<float, 2, 1> float2x1;
typedef matrix<float, 2, 2> float2x2;
typedef matrix<float, 2, 3> float2x3;
typedef matrix<float, 2, 4> float2x4;
typedef matrix<float, 3, 1> float3x1;
typedef matrix<float, 3, 2> float3x2;
typedef matrix<float, 3, 3> float3x3;
typedef matrix<float, 3, 4> float3x4;
typedef matrix<float, 4, 1> float4x1;
typedef matrix<float, 4, 2> float4x2;
typedef matrix<float, 4, 3> float4x3;
typedef matrix<float, 4, 4> float4x4;
typedef matrix<double, 1, 1> double1x1;
typedef matrix<double, 1, 2> double1x2;
````
- **L281 EN**: Introduces an alias or helper declaration: `typedef matrix<half, 4, 3> half4x3;`.
  **L281 CN**: 引入一条别名或辅助声明：`typedef matrix<half, 4, 3> half4x3;`。
- **L282 EN**: Introduces an alias or helper declaration: `typedef matrix<half, 4, 4> half4x4;`.
  **L282 CN**: 引入一条别名或辅助声明：`typedef matrix<half, 4, 4> half4x4;`。
- **L283 EN**: Introduces an alias or helper declaration: `typedef matrix<float, 1, 1> float1x1;`.
  **L283 CN**: 引入一条别名或辅助声明：`typedef matrix<float, 1, 1> float1x1;`。
- **L284 EN**: Introduces an alias or helper declaration: `typedef matrix<float, 1, 2> float1x2;`.
  **L284 CN**: 引入一条别名或辅助声明：`typedef matrix<float, 1, 2> float1x2;`。
- **L285 EN**: Introduces an alias or helper declaration: `typedef matrix<float, 1, 3> float1x3;`.
  **L285 CN**: 引入一条别名或辅助声明：`typedef matrix<float, 1, 3> float1x3;`。
- **L286 EN**: Introduces an alias or helper declaration: `typedef matrix<float, 1, 4> float1x4;`.
  **L286 CN**: 引入一条别名或辅助声明：`typedef matrix<float, 1, 4> float1x4;`。
- **L287 EN**: Introduces an alias or helper declaration: `typedef matrix<float, 2, 1> float2x1;`.
  **L287 CN**: 引入一条别名或辅助声明：`typedef matrix<float, 2, 1> float2x1;`。
- **L288 EN**: Introduces an alias or helper declaration: `typedef matrix<float, 2, 2> float2x2;`.
  **L288 CN**: 引入一条别名或辅助声明：`typedef matrix<float, 2, 2> float2x2;`。
- **L289 EN**: Introduces an alias or helper declaration: `typedef matrix<float, 2, 3> float2x3;`.
  **L289 CN**: 引入一条别名或辅助声明：`typedef matrix<float, 2, 3> float2x3;`。
- **L290 EN**: Introduces an alias or helper declaration: `typedef matrix<float, 2, 4> float2x4;`.
  **L290 CN**: 引入一条别名或辅助声明：`typedef matrix<float, 2, 4> float2x4;`。
- **L291 EN**: Introduces an alias or helper declaration: `typedef matrix<float, 3, 1> float3x1;`.
  **L291 CN**: 引入一条别名或辅助声明：`typedef matrix<float, 3, 1> float3x1;`。
- **L292 EN**: Introduces an alias or helper declaration: `typedef matrix<float, 3, 2> float3x2;`.
  **L292 CN**: 引入一条别名或辅助声明：`typedef matrix<float, 3, 2> float3x2;`。
- **L293 EN**: Introduces an alias or helper declaration: `typedef matrix<float, 3, 3> float3x3;`.
  **L293 CN**: 引入一条别名或辅助声明：`typedef matrix<float, 3, 3> float3x3;`。
- **L294 EN**: Introduces an alias or helper declaration: `typedef matrix<float, 3, 4> float3x4;`.
  **L294 CN**: 引入一条别名或辅助声明：`typedef matrix<float, 3, 4> float3x4;`。
- **L295 EN**: Introduces an alias or helper declaration: `typedef matrix<float, 4, 1> float4x1;`.
  **L295 CN**: 引入一条别名或辅助声明：`typedef matrix<float, 4, 1> float4x1;`。
- **L296 EN**: Introduces an alias or helper declaration: `typedef matrix<float, 4, 2> float4x2;`.
  **L296 CN**: 引入一条别名或辅助声明：`typedef matrix<float, 4, 2> float4x2;`。
- **L297 EN**: Introduces an alias or helper declaration: `typedef matrix<float, 4, 3> float4x3;`.
  **L297 CN**: 引入一条别名或辅助声明：`typedef matrix<float, 4, 3> float4x3;`。
- **L298 EN**: Introduces an alias or helper declaration: `typedef matrix<float, 4, 4> float4x4;`.
  **L298 CN**: 引入一条别名或辅助声明：`typedef matrix<float, 4, 4> float4x4;`。
- **L299 EN**: Introduces an alias or helper declaration: `typedef matrix<double, 1, 1> double1x1;`.
  **L299 CN**: 引入一条别名或辅助声明：`typedef matrix<double, 1, 1> double1x1;`。
- **L300 EN**: Introduces an alias or helper declaration: `typedef matrix<double, 1, 2> double1x2;`.
  **L300 CN**: 引入一条别名或辅助声明：`typedef matrix<double, 1, 2> double1x2;`。

### Lines 301-320

````c
typedef matrix<double, 1, 3> double1x3;
typedef matrix<double, 1, 4> double1x4;
typedef matrix<double, 2, 1> double2x1;
typedef matrix<double, 2, 2> double2x2;
typedef matrix<double, 2, 3> double2x3;
typedef matrix<double, 2, 4> double2x4;
typedef matrix<double, 3, 1> double3x1;
typedef matrix<double, 3, 2> double3x2;
typedef matrix<double, 3, 3> double3x3;
typedef matrix<double, 3, 4> double3x4;
typedef matrix<double, 4, 1> double4x1;
typedef matrix<double, 4, 2> double4x2;
typedef matrix<double, 4, 3> double4x3;
typedef matrix<double, 4, 4> double4x4;

#ifdef __HLSL_ENABLE_16_BIT
typedef matrix<float16_t, 1, 1> float16_t1x1;
typedef matrix<float16_t, 1, 2> float16_t1x2;
typedef matrix<float16_t, 1, 3> float16_t1x3;
typedef matrix<float16_t, 1, 4> float16_t1x4;
````
- **L301 EN**: Introduces an alias or helper declaration: `typedef matrix<double, 1, 3> double1x3;`.
  **L301 CN**: 引入一条别名或辅助声明：`typedef matrix<double, 1, 3> double1x3;`。
- **L302 EN**: Introduces an alias or helper declaration: `typedef matrix<double, 1, 4> double1x4;`.
  **L302 CN**: 引入一条别名或辅助声明：`typedef matrix<double, 1, 4> double1x4;`。
- **L303 EN**: Introduces an alias or helper declaration: `typedef matrix<double, 2, 1> double2x1;`.
  **L303 CN**: 引入一条别名或辅助声明：`typedef matrix<double, 2, 1> double2x1;`。
- **L304 EN**: Introduces an alias or helper declaration: `typedef matrix<double, 2, 2> double2x2;`.
  **L304 CN**: 引入一条别名或辅助声明：`typedef matrix<double, 2, 2> double2x2;`。
- **L305 EN**: Introduces an alias or helper declaration: `typedef matrix<double, 2, 3> double2x3;`.
  **L305 CN**: 引入一条别名或辅助声明：`typedef matrix<double, 2, 3> double2x3;`。
- **L306 EN**: Introduces an alias or helper declaration: `typedef matrix<double, 2, 4> double2x4;`.
  **L306 CN**: 引入一条别名或辅助声明：`typedef matrix<double, 2, 4> double2x4;`。
- **L307 EN**: Introduces an alias or helper declaration: `typedef matrix<double, 3, 1> double3x1;`.
  **L307 CN**: 引入一条别名或辅助声明：`typedef matrix<double, 3, 1> double3x1;`。
- **L308 EN**: Introduces an alias or helper declaration: `typedef matrix<double, 3, 2> double3x2;`.
  **L308 CN**: 引入一条别名或辅助声明：`typedef matrix<double, 3, 2> double3x2;`。
- **L309 EN**: Introduces an alias or helper declaration: `typedef matrix<double, 3, 3> double3x3;`.
  **L309 CN**: 引入一条别名或辅助声明：`typedef matrix<double, 3, 3> double3x3;`。
- **L310 EN**: Introduces an alias or helper declaration: `typedef matrix<double, 3, 4> double3x4;`.
  **L310 CN**: 引入一条别名或辅助声明：`typedef matrix<double, 3, 4> double3x4;`。
- **L311 EN**: Introduces an alias or helper declaration: `typedef matrix<double, 4, 1> double4x1;`.
  **L311 CN**: 引入一条别名或辅助声明：`typedef matrix<double, 4, 1> double4x1;`。
- **L312 EN**: Introduces an alias or helper declaration: `typedef matrix<double, 4, 2> double4x2;`.
  **L312 CN**: 引入一条别名或辅助声明：`typedef matrix<double, 4, 2> double4x2;`。
- **L313 EN**: Introduces an alias or helper declaration: `typedef matrix<double, 4, 3> double4x3;`.
  **L313 CN**: 引入一条别名或辅助声明：`typedef matrix<double, 4, 3> double4x3;`。
- **L314 EN**: Introduces an alias or helper declaration: `typedef matrix<double, 4, 4> double4x4;`.
  **L314 CN**: 引入一条别名或辅助声明：`typedef matrix<double, 4, 4> double4x4;`。
- **L315 EN**: Blank line separating nearby declarations or logic blocks.
  **L315 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L316 EN**: Starts a preprocessor conditional block: `#ifdef __HLSL_ENABLE_16_BIT`.
  **L316 CN**: 开始一个预处理条件块：`#ifdef __HLSL_ENABLE_16_BIT`。
- **L317 EN**: Introduces an alias or helper declaration: `typedef matrix<float16_t, 1, 1> float16_t1x1;`.
  **L317 CN**: 引入一条别名或辅助声明：`typedef matrix<float16_t, 1, 1> float16_t1x1;`。
- **L318 EN**: Introduces an alias or helper declaration: `typedef matrix<float16_t, 1, 2> float16_t1x2;`.
  **L318 CN**: 引入一条别名或辅助声明：`typedef matrix<float16_t, 1, 2> float16_t1x2;`。
- **L319 EN**: Introduces an alias or helper declaration: `typedef matrix<float16_t, 1, 3> float16_t1x3;`.
  **L319 CN**: 引入一条别名或辅助声明：`typedef matrix<float16_t, 1, 3> float16_t1x3;`。
- **L320 EN**: Introduces an alias or helper declaration: `typedef matrix<float16_t, 1, 4> float16_t1x4;`.
  **L320 CN**: 引入一条别名或辅助声明：`typedef matrix<float16_t, 1, 4> float16_t1x4;`。

### Lines 321-340

````c
typedef matrix<float16_t, 2, 1> float16_t2x1;
typedef matrix<float16_t, 2, 2> float16_t2x2;
typedef matrix<float16_t, 2, 3> float16_t2x3;
typedef matrix<float16_t, 2, 4> float16_t2x4;
typedef matrix<float16_t, 3, 1> float16_t3x1;
typedef matrix<float16_t, 3, 2> float16_t3x2;
typedef matrix<float16_t, 3, 3> float16_t3x3;
typedef matrix<float16_t, 3, 4> float16_t3x4;
typedef matrix<float16_t, 4, 1> float16_t4x1;
typedef matrix<float16_t, 4, 2> float16_t4x2;
typedef matrix<float16_t, 4, 3> float16_t4x3;
typedef matrix<float16_t, 4, 4> float16_t4x4;
#endif

typedef matrix<float32_t, 1, 1> float32_t1x1;
typedef matrix<float32_t, 1, 2> float32_t1x2;
typedef matrix<float32_t, 1, 3> float32_t1x3;
typedef matrix<float32_t, 1, 4> float32_t1x4;
typedef matrix<float32_t, 2, 1> float32_t2x1;
typedef matrix<float32_t, 2, 2> float32_t2x2;
````
- **L321 EN**: Introduces an alias or helper declaration: `typedef matrix<float16_t, 2, 1> float16_t2x1;`.
  **L321 CN**: 引入一条别名或辅助声明：`typedef matrix<float16_t, 2, 1> float16_t2x1;`。
- **L322 EN**: Introduces an alias or helper declaration: `typedef matrix<float16_t, 2, 2> float16_t2x2;`.
  **L322 CN**: 引入一条别名或辅助声明：`typedef matrix<float16_t, 2, 2> float16_t2x2;`。
- **L323 EN**: Introduces an alias or helper declaration: `typedef matrix<float16_t, 2, 3> float16_t2x3;`.
  **L323 CN**: 引入一条别名或辅助声明：`typedef matrix<float16_t, 2, 3> float16_t2x3;`。
- **L324 EN**: Introduces an alias or helper declaration: `typedef matrix<float16_t, 2, 4> float16_t2x4;`.
  **L324 CN**: 引入一条别名或辅助声明：`typedef matrix<float16_t, 2, 4> float16_t2x4;`。
- **L325 EN**: Introduces an alias or helper declaration: `typedef matrix<float16_t, 3, 1> float16_t3x1;`.
  **L325 CN**: 引入一条别名或辅助声明：`typedef matrix<float16_t, 3, 1> float16_t3x1;`。
- **L326 EN**: Introduces an alias or helper declaration: `typedef matrix<float16_t, 3, 2> float16_t3x2;`.
  **L326 CN**: 引入一条别名或辅助声明：`typedef matrix<float16_t, 3, 2> float16_t3x2;`。
- **L327 EN**: Introduces an alias or helper declaration: `typedef matrix<float16_t, 3, 3> float16_t3x3;`.
  **L327 CN**: 引入一条别名或辅助声明：`typedef matrix<float16_t, 3, 3> float16_t3x3;`。
- **L328 EN**: Introduces an alias or helper declaration: `typedef matrix<float16_t, 3, 4> float16_t3x4;`.
  **L328 CN**: 引入一条别名或辅助声明：`typedef matrix<float16_t, 3, 4> float16_t3x4;`。
- **L329 EN**: Introduces an alias or helper declaration: `typedef matrix<float16_t, 4, 1> float16_t4x1;`.
  **L329 CN**: 引入一条别名或辅助声明：`typedef matrix<float16_t, 4, 1> float16_t4x1;`。
- **L330 EN**: Introduces an alias or helper declaration: `typedef matrix<float16_t, 4, 2> float16_t4x2;`.
  **L330 CN**: 引入一条别名或辅助声明：`typedef matrix<float16_t, 4, 2> float16_t4x2;`。
- **L331 EN**: Introduces an alias or helper declaration: `typedef matrix<float16_t, 4, 3> float16_t4x3;`.
  **L331 CN**: 引入一条别名或辅助声明：`typedef matrix<float16_t, 4, 3> float16_t4x3;`。
- **L332 EN**: Introduces an alias or helper declaration: `typedef matrix<float16_t, 4, 4> float16_t4x4;`.
  **L332 CN**: 引入一条别名或辅助声明：`typedef matrix<float16_t, 4, 4> float16_t4x4;`。
- **L333 EN**: Closes the current preprocessor conditional block.
  **L333 CN**: 结束当前预处理条件块。
- **L334 EN**: Blank line separating nearby declarations or logic blocks.
  **L334 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L335 EN**: Introduces an alias or helper declaration: `typedef matrix<float32_t, 1, 1> float32_t1x1;`.
  **L335 CN**: 引入一条别名或辅助声明：`typedef matrix<float32_t, 1, 1> float32_t1x1;`。
- **L336 EN**: Introduces an alias or helper declaration: `typedef matrix<float32_t, 1, 2> float32_t1x2;`.
  **L336 CN**: 引入一条别名或辅助声明：`typedef matrix<float32_t, 1, 2> float32_t1x2;`。
- **L337 EN**: Introduces an alias or helper declaration: `typedef matrix<float32_t, 1, 3> float32_t1x3;`.
  **L337 CN**: 引入一条别名或辅助声明：`typedef matrix<float32_t, 1, 3> float32_t1x3;`。
- **L338 EN**: Introduces an alias or helper declaration: `typedef matrix<float32_t, 1, 4> float32_t1x4;`.
  **L338 CN**: 引入一条别名或辅助声明：`typedef matrix<float32_t, 1, 4> float32_t1x4;`。
- **L339 EN**: Introduces an alias or helper declaration: `typedef matrix<float32_t, 2, 1> float32_t2x1;`.
  **L339 CN**: 引入一条别名或辅助声明：`typedef matrix<float32_t, 2, 1> float32_t2x1;`。
- **L340 EN**: Introduces an alias or helper declaration: `typedef matrix<float32_t, 2, 2> float32_t2x2;`.
  **L340 CN**: 引入一条别名或辅助声明：`typedef matrix<float32_t, 2, 2> float32_t2x2;`。

### Lines 341-360

````c
typedef matrix<float32_t, 2, 3> float32_t2x3;
typedef matrix<float32_t, 2, 4> float32_t2x4;
typedef matrix<float32_t, 3, 1> float32_t3x1;
typedef matrix<float32_t, 3, 2> float32_t3x2;
typedef matrix<float32_t, 3, 3> float32_t3x3;
typedef matrix<float32_t, 3, 4> float32_t3x4;
typedef matrix<float32_t, 4, 1> float32_t4x1;
typedef matrix<float32_t, 4, 2> float32_t4x2;
typedef matrix<float32_t, 4, 3> float32_t4x3;
typedef matrix<float32_t, 4, 4> float32_t4x4;
typedef matrix<float64_t, 1, 1> float64_t1x1;
typedef matrix<float64_t, 1, 2> float64_t1x2;
typedef matrix<float64_t, 1, 3> float64_t1x3;
typedef matrix<float64_t, 1, 4> float64_t1x4;
typedef matrix<float64_t, 2, 1> float64_t2x1;
typedef matrix<float64_t, 2, 2> float64_t2x2;
typedef matrix<float64_t, 2, 3> float64_t2x3;
typedef matrix<float64_t, 2, 4> float64_t2x4;
typedef matrix<float64_t, 3, 1> float64_t3x1;
typedef matrix<float64_t, 3, 2> float64_t3x2;
````
- **L341 EN**: Introduces an alias or helper declaration: `typedef matrix<float32_t, 2, 3> float32_t2x3;`.
  **L341 CN**: 引入一条别名或辅助声明：`typedef matrix<float32_t, 2, 3> float32_t2x3;`。
- **L342 EN**: Introduces an alias or helper declaration: `typedef matrix<float32_t, 2, 4> float32_t2x4;`.
  **L342 CN**: 引入一条别名或辅助声明：`typedef matrix<float32_t, 2, 4> float32_t2x4;`。
- **L343 EN**: Introduces an alias or helper declaration: `typedef matrix<float32_t, 3, 1> float32_t3x1;`.
  **L343 CN**: 引入一条别名或辅助声明：`typedef matrix<float32_t, 3, 1> float32_t3x1;`。
- **L344 EN**: Introduces an alias or helper declaration: `typedef matrix<float32_t, 3, 2> float32_t3x2;`.
  **L344 CN**: 引入一条别名或辅助声明：`typedef matrix<float32_t, 3, 2> float32_t3x2;`。
- **L345 EN**: Introduces an alias or helper declaration: `typedef matrix<float32_t, 3, 3> float32_t3x3;`.
  **L345 CN**: 引入一条别名或辅助声明：`typedef matrix<float32_t, 3, 3> float32_t3x3;`。
- **L346 EN**: Introduces an alias or helper declaration: `typedef matrix<float32_t, 3, 4> float32_t3x4;`.
  **L346 CN**: 引入一条别名或辅助声明：`typedef matrix<float32_t, 3, 4> float32_t3x4;`。
- **L347 EN**: Introduces an alias or helper declaration: `typedef matrix<float32_t, 4, 1> float32_t4x1;`.
  **L347 CN**: 引入一条别名或辅助声明：`typedef matrix<float32_t, 4, 1> float32_t4x1;`。
- **L348 EN**: Introduces an alias or helper declaration: `typedef matrix<float32_t, 4, 2> float32_t4x2;`.
  **L348 CN**: 引入一条别名或辅助声明：`typedef matrix<float32_t, 4, 2> float32_t4x2;`。
- **L349 EN**: Introduces an alias or helper declaration: `typedef matrix<float32_t, 4, 3> float32_t4x3;`.
  **L349 CN**: 引入一条别名或辅助声明：`typedef matrix<float32_t, 4, 3> float32_t4x3;`。
- **L350 EN**: Introduces an alias or helper declaration: `typedef matrix<float32_t, 4, 4> float32_t4x4;`.
  **L350 CN**: 引入一条别名或辅助声明：`typedef matrix<float32_t, 4, 4> float32_t4x4;`。
- **L351 EN**: Introduces an alias or helper declaration: `typedef matrix<float64_t, 1, 1> float64_t1x1;`.
  **L351 CN**: 引入一条别名或辅助声明：`typedef matrix<float64_t, 1, 1> float64_t1x1;`。
- **L352 EN**: Introduces an alias or helper declaration: `typedef matrix<float64_t, 1, 2> float64_t1x2;`.
  **L352 CN**: 引入一条别名或辅助声明：`typedef matrix<float64_t, 1, 2> float64_t1x2;`。
- **L353 EN**: Introduces an alias or helper declaration: `typedef matrix<float64_t, 1, 3> float64_t1x3;`.
  **L353 CN**: 引入一条别名或辅助声明：`typedef matrix<float64_t, 1, 3> float64_t1x3;`。
- **L354 EN**: Introduces an alias or helper declaration: `typedef matrix<float64_t, 1, 4> float64_t1x4;`.
  **L354 CN**: 引入一条别名或辅助声明：`typedef matrix<float64_t, 1, 4> float64_t1x4;`。
- **L355 EN**: Introduces an alias or helper declaration: `typedef matrix<float64_t, 2, 1> float64_t2x1;`.
  **L355 CN**: 引入一条别名或辅助声明：`typedef matrix<float64_t, 2, 1> float64_t2x1;`。
- **L356 EN**: Introduces an alias or helper declaration: `typedef matrix<float64_t, 2, 2> float64_t2x2;`.
  **L356 CN**: 引入一条别名或辅助声明：`typedef matrix<float64_t, 2, 2> float64_t2x2;`。
- **L357 EN**: Introduces an alias or helper declaration: `typedef matrix<float64_t, 2, 3> float64_t2x3;`.
  **L357 CN**: 引入一条别名或辅助声明：`typedef matrix<float64_t, 2, 3> float64_t2x3;`。
- **L358 EN**: Introduces an alias or helper declaration: `typedef matrix<float64_t, 2, 4> float64_t2x4;`.
  **L358 CN**: 引入一条别名或辅助声明：`typedef matrix<float64_t, 2, 4> float64_t2x4;`。
- **L359 EN**: Introduces an alias or helper declaration: `typedef matrix<float64_t, 3, 1> float64_t3x1;`.
  **L359 CN**: 引入一条别名或辅助声明：`typedef matrix<float64_t, 3, 1> float64_t3x1;`。
- **L360 EN**: Introduces an alias or helper declaration: `typedef matrix<float64_t, 3, 2> float64_t3x2;`.
  **L360 CN**: 引入一条别名或辅助声明：`typedef matrix<float64_t, 3, 2> float64_t3x2;`。

### Lines 361-370

````c
typedef matrix<float64_t, 3, 3> float64_t3x3;
typedef matrix<float64_t, 3, 4> float64_t3x4;
typedef matrix<float64_t, 4, 1> float64_t4x1;
typedef matrix<float64_t, 4, 2> float64_t4x2;
typedef matrix<float64_t, 4, 3> float64_t4x3;
typedef matrix<float64_t, 4, 4> float64_t4x4;

} // namespace hlsl

#endif //_HLSL_HLSL_BASIC_TYPES_H_
````
- **L361 EN**: Introduces an alias or helper declaration: `typedef matrix<float64_t, 3, 3> float64_t3x3;`.
  **L361 CN**: 引入一条别名或辅助声明：`typedef matrix<float64_t, 3, 3> float64_t3x3;`。
- **L362 EN**: Introduces an alias or helper declaration: `typedef matrix<float64_t, 3, 4> float64_t3x4;`.
  **L362 CN**: 引入一条别名或辅助声明：`typedef matrix<float64_t, 3, 4> float64_t3x4;`。
- **L363 EN**: Introduces an alias or helper declaration: `typedef matrix<float64_t, 4, 1> float64_t4x1;`.
  **L363 CN**: 引入一条别名或辅助声明：`typedef matrix<float64_t, 4, 1> float64_t4x1;`。
- **L364 EN**: Introduces an alias or helper declaration: `typedef matrix<float64_t, 4, 2> float64_t4x2;`.
  **L364 CN**: 引入一条别名或辅助声明：`typedef matrix<float64_t, 4, 2> float64_t4x2;`。
- **L365 EN**: Introduces an alias or helper declaration: `typedef matrix<float64_t, 4, 3> float64_t4x3;`.
  **L365 CN**: 引入一条别名或辅助声明：`typedef matrix<float64_t, 4, 3> float64_t4x3;`。
- **L366 EN**: Introduces an alias or helper declaration: `typedef matrix<float64_t, 4, 4> float64_t4x4;`.
  **L366 CN**: 引入一条别名或辅助声明：`typedef matrix<float64_t, 4, 4> float64_t4x4;`。
- **L367 EN**: Blank line separating nearby declarations or logic blocks.
  **L367 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L368 EN**: Closes a namespace scope and documents it with a trailing comment: `} // namespace hlsl`.
  **L368 CN**: 结束一个命名空间作用域，并用尾注释标明其名称：`} // namespace hlsl`。
- **L369 EN**: Blank line separating nearby declarations or logic blocks.
  **L369 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L370 EN**: Closes the current preprocessor conditional block.
  **L370 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **HLSL compatibility surface / HLSL 兼容接口**
- **Macro-based API construction / 基于宏的 API 构造**
- **Compiler-specific attributes / 编译器特定属性**
- **Type aliasing and ABI shaping / 类型别名与 ABI 约束**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None. / 无。
- **Conditional macros / 条件宏**: `_HLSL_HLSL_BASIC_TYPES_H_`, `__HLSL_ENABLE_16_BIT`
- **External builtins / 外部 builtin**: No direct builtin forwarding detected. / 未检测到直接的 builtin 转发。
