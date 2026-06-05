# msa.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/msa.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: MIPS MSA intrinsics.
- **Purpose (CN)**: 提供 MIPS MSA intrinsic 接口。
- **Line Count / 行数**: 573

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````c
/*===---- msa.h - MIPS MSA intrinsics --------------------------------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

#ifndef _MSA_H
#define _MSA_H 1

#if defined(__mips_msa)
typedef signed char v16i8 __attribute__((vector_size(16), aligned(16)));
typedef signed char v16i8_b __attribute__((vector_size(16), aligned(1)));
typedef unsigned char v16u8 __attribute__((vector_size(16), aligned(16)));
typedef unsigned char v16u8_b __attribute__((vector_size(16), aligned(1)));
typedef short v8i16 __attribute__((vector_size(16), aligned(16)));
typedef short v8i16_h __attribute__((vector_size(16), aligned(2)));
typedef unsigned short v8u16 __attribute__((vector_size(16), aligned(16)));
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
- **L9 EN**: Blank line separating nearby declarations or logic blocks.
  **L9 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L10 EN**: Starts a preprocessor conditional block: `#ifndef _MSA_H`.
  **L10 CN**: 开始一个预处理条件块：`#ifndef _MSA_H`。
- **L11 EN**: Defines macro `_MSA_H` for conditional compilation, shorthand, or API generation.
  **L11 CN**: 定义宏 `_MSA_H`，用于条件编译、简写或 API 生成。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Starts a preprocessor conditional block: `#if defined(__mips_msa)`.
  **L13 CN**: 开始一个预处理条件块：`#if defined(__mips_msa)`。
- **L14 EN**: Introduces an alias or helper declaration: `typedef signed char v16i8 __attribute__((vector_size(16), aligned(16)));`.
  **L14 CN**: 引入一条别名或辅助声明：`typedef signed char v16i8 __attribute__((vector_size(16), aligned(16)));`。
- **L15 EN**: Introduces an alias or helper declaration: `typedef signed char v16i8_b __attribute__((vector_size(16), aligned(1)));`.
  **L15 CN**: 引入一条别名或辅助声明：`typedef signed char v16i8_b __attribute__((vector_size(16), aligned(1)));`。
- **L16 EN**: Introduces an alias or helper declaration: `typedef unsigned char v16u8 __attribute__((vector_size(16), aligned(16)));`.
  **L16 CN**: 引入一条别名或辅助声明：`typedef unsigned char v16u8 __attribute__((vector_size(16), aligned(16)));`。
- **L17 EN**: Introduces an alias or helper declaration: `typedef unsigned char v16u8_b __attribute__((vector_size(16), aligned(1)));`.
  **L17 CN**: 引入一条别名或辅助声明：`typedef unsigned char v16u8_b __attribute__((vector_size(16), aligned(1)));`。
- **L18 EN**: Introduces an alias or helper declaration: `typedef short v8i16 __attribute__((vector_size(16), aligned(16)));`.
  **L18 CN**: 引入一条别名或辅助声明：`typedef short v8i16 __attribute__((vector_size(16), aligned(16)));`。
- **L19 EN**: Introduces an alias or helper declaration: `typedef short v8i16_h __attribute__((vector_size(16), aligned(2)));`.
  **L19 CN**: 引入一条别名或辅助声明：`typedef short v8i16_h __attribute__((vector_size(16), aligned(2)));`。
- **L20 EN**: Introduces an alias or helper declaration: `typedef unsigned short v8u16 __attribute__((vector_size(16), aligned(16)));`.
  **L20 CN**: 引入一条别名或辅助声明：`typedef unsigned short v8u16 __attribute__((vector_size(16), aligned(16)));`。

### Lines 21-40

````c
typedef unsigned short v8u16_h __attribute__((vector_size(16), aligned(2)));
typedef int v4i32 __attribute__((vector_size(16), aligned(16)));
typedef int v4i32_w __attribute__((vector_size(16), aligned(4)));
typedef unsigned int v4u32 __attribute__((vector_size(16), aligned(16)));
typedef unsigned int v4u32_w __attribute__((vector_size(16), aligned(4)));
typedef long long v2i64 __attribute__((vector_size(16), aligned(16)));
typedef long long v2i64_d __attribute__((vector_size(16), aligned(8)));
typedef unsigned long long v2u64 __attribute__((vector_size(16), aligned(16)));
typedef unsigned long long v2u64_d __attribute__((vector_size(16), aligned(8)));
typedef float v4f32 __attribute__((vector_size(16), aligned(16)));
typedef float v4f32_w __attribute__((vector_size(16), aligned(4)));
typedef double v2f64 __attribute__ ((vector_size(16), aligned(16)));
typedef double v2f64_d __attribute__ ((vector_size(16), aligned(8)));

#define __msa_sll_b __builtin_msa_sll_b
#define __msa_sll_h __builtin_msa_sll_h
#define __msa_sll_w __builtin_msa_sll_w
#define __msa_sll_d __builtin_msa_sll_d
#define __msa_slli_b __builtin_msa_slli_b
#define __msa_slli_h __builtin_msa_slli_h
````
- **L21 EN**: Introduces an alias or helper declaration: `typedef unsigned short v8u16_h __attribute__((vector_size(16), aligned(2)));`.
  **L21 CN**: 引入一条别名或辅助声明：`typedef unsigned short v8u16_h __attribute__((vector_size(16), aligned(2)));`。
- **L22 EN**: Introduces an alias or helper declaration: `typedef int v4i32 __attribute__((vector_size(16), aligned(16)));`.
  **L22 CN**: 引入一条别名或辅助声明：`typedef int v4i32 __attribute__((vector_size(16), aligned(16)));`。
- **L23 EN**: Introduces an alias or helper declaration: `typedef int v4i32_w __attribute__((vector_size(16), aligned(4)));`.
  **L23 CN**: 引入一条别名或辅助声明：`typedef int v4i32_w __attribute__((vector_size(16), aligned(4)));`。
- **L24 EN**: Introduces an alias or helper declaration: `typedef unsigned int v4u32 __attribute__((vector_size(16), aligned(16)));`.
  **L24 CN**: 引入一条别名或辅助声明：`typedef unsigned int v4u32 __attribute__((vector_size(16), aligned(16)));`。
- **L25 EN**: Introduces an alias or helper declaration: `typedef unsigned int v4u32_w __attribute__((vector_size(16), aligned(4)));`.
  **L25 CN**: 引入一条别名或辅助声明：`typedef unsigned int v4u32_w __attribute__((vector_size(16), aligned(4)));`。
- **L26 EN**: Introduces an alias or helper declaration: `typedef long long v2i64 __attribute__((vector_size(16), aligned(16)));`.
  **L26 CN**: 引入一条别名或辅助声明：`typedef long long v2i64 __attribute__((vector_size(16), aligned(16)));`。
- **L27 EN**: Introduces an alias or helper declaration: `typedef long long v2i64_d __attribute__((vector_size(16), aligned(8)));`.
  **L27 CN**: 引入一条别名或辅助声明：`typedef long long v2i64_d __attribute__((vector_size(16), aligned(8)));`。
- **L28 EN**: Introduces an alias or helper declaration: `typedef unsigned long long v2u64 __attribute__((vector_size(16), aligned(16)));`.
  **L28 CN**: 引入一条别名或辅助声明：`typedef unsigned long long v2u64 __attribute__((vector_size(16), aligned(16)));`。
- **L29 EN**: Introduces an alias or helper declaration: `typedef unsigned long long v2u64_d __attribute__((vector_size(16), aligned(8)));`.
  **L29 CN**: 引入一条别名或辅助声明：`typedef unsigned long long v2u64_d __attribute__((vector_size(16), aligned(8)));`。
- **L30 EN**: Introduces an alias or helper declaration: `typedef float v4f32 __attribute__((vector_size(16), aligned(16)));`.
  **L30 CN**: 引入一条别名或辅助声明：`typedef float v4f32 __attribute__((vector_size(16), aligned(16)));`。
- **L31 EN**: Introduces an alias or helper declaration: `typedef float v4f32_w __attribute__((vector_size(16), aligned(4)));`.
  **L31 CN**: 引入一条别名或辅助声明：`typedef float v4f32_w __attribute__((vector_size(16), aligned(4)));`。
- **L32 EN**: Introduces an alias or helper declaration: `typedef double v2f64 __attribute__ ((vector_size(16), aligned(16)));`.
  **L32 CN**: 引入一条别名或辅助声明：`typedef double v2f64 __attribute__ ((vector_size(16), aligned(16)));`。
- **L33 EN**: Introduces an alias or helper declaration: `typedef double v2f64_d __attribute__ ((vector_size(16), aligned(8)));`.
  **L33 CN**: 引入一条别名或辅助声明：`typedef double v2f64_d __attribute__ ((vector_size(16), aligned(8)));`。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L35 EN**: Defines macro `__msa_sll_b` for conditional compilation, shorthand, or API generation.
  **L35 CN**: 定义宏 `__msa_sll_b`，用于条件编译、简写或 API 生成。
- **L36 EN**: Defines macro `__msa_sll_h` for conditional compilation, shorthand, or API generation.
  **L36 CN**: 定义宏 `__msa_sll_h`，用于条件编译、简写或 API 生成。
- **L37 EN**: Defines macro `__msa_sll_w` for conditional compilation, shorthand, or API generation.
  **L37 CN**: 定义宏 `__msa_sll_w`，用于条件编译、简写或 API 生成。
- **L38 EN**: Defines macro `__msa_sll_d` for conditional compilation, shorthand, or API generation.
  **L38 CN**: 定义宏 `__msa_sll_d`，用于条件编译、简写或 API 生成。
- **L39 EN**: Defines macro `__msa_slli_b` for conditional compilation, shorthand, or API generation.
  **L39 CN**: 定义宏 `__msa_slli_b`，用于条件编译、简写或 API 生成。
- **L40 EN**: Defines macro `__msa_slli_h` for conditional compilation, shorthand, or API generation.
  **L40 CN**: 定义宏 `__msa_slli_h`，用于条件编译、简写或 API 生成。

### Lines 41-60

````c
#define __msa_slli_w __builtin_msa_slli_w
#define __msa_slli_d __builtin_msa_slli_d
#define __msa_sra_b __builtin_msa_sra_b
#define __msa_sra_h __builtin_msa_sra_h
#define __msa_sra_w __builtin_msa_sra_w
#define __msa_sra_d __builtin_msa_sra_d
#define __msa_srai_b __builtin_msa_srai_b
#define __msa_srai_h __builtin_msa_srai_h
#define __msa_srai_w __builtin_msa_srai_w
#define __msa_srai_d __builtin_msa_srai_d
#define __msa_srar_b __builtin_msa_srar_b
#define __msa_srar_h __builtin_msa_srar_h
#define __msa_srar_w __builtin_msa_srar_w
#define __msa_srar_d __builtin_msa_srar_d
#define __msa_srari_b __builtin_msa_srari_b
#define __msa_srari_h __builtin_msa_srari_h
#define __msa_srari_w __builtin_msa_srari_w
#define __msa_srari_d __builtin_msa_srari_d
#define __msa_srl_b __builtin_msa_srl_b
#define __msa_srl_h __builtin_msa_srl_h
````
- **L41 EN**: Defines macro `__msa_slli_w` for conditional compilation, shorthand, or API generation.
  **L41 CN**: 定义宏 `__msa_slli_w`，用于条件编译、简写或 API 生成。
- **L42 EN**: Defines macro `__msa_slli_d` for conditional compilation, shorthand, or API generation.
  **L42 CN**: 定义宏 `__msa_slli_d`，用于条件编译、简写或 API 生成。
- **L43 EN**: Defines macro `__msa_sra_b` for conditional compilation, shorthand, or API generation.
  **L43 CN**: 定义宏 `__msa_sra_b`，用于条件编译、简写或 API 生成。
- **L44 EN**: Defines macro `__msa_sra_h` for conditional compilation, shorthand, or API generation.
  **L44 CN**: 定义宏 `__msa_sra_h`，用于条件编译、简写或 API 生成。
- **L45 EN**: Defines macro `__msa_sra_w` for conditional compilation, shorthand, or API generation.
  **L45 CN**: 定义宏 `__msa_sra_w`，用于条件编译、简写或 API 生成。
- **L46 EN**: Defines macro `__msa_sra_d` for conditional compilation, shorthand, or API generation.
  **L46 CN**: 定义宏 `__msa_sra_d`，用于条件编译、简写或 API 生成。
- **L47 EN**: Defines macro `__msa_srai_b` for conditional compilation, shorthand, or API generation.
  **L47 CN**: 定义宏 `__msa_srai_b`，用于条件编译、简写或 API 生成。
- **L48 EN**: Defines macro `__msa_srai_h` for conditional compilation, shorthand, or API generation.
  **L48 CN**: 定义宏 `__msa_srai_h`，用于条件编译、简写或 API 生成。
- **L49 EN**: Defines macro `__msa_srai_w` for conditional compilation, shorthand, or API generation.
  **L49 CN**: 定义宏 `__msa_srai_w`，用于条件编译、简写或 API 生成。
- **L50 EN**: Defines macro `__msa_srai_d` for conditional compilation, shorthand, or API generation.
  **L50 CN**: 定义宏 `__msa_srai_d`，用于条件编译、简写或 API 生成。
- **L51 EN**: Defines macro `__msa_srar_b` for conditional compilation, shorthand, or API generation.
  **L51 CN**: 定义宏 `__msa_srar_b`，用于条件编译、简写或 API 生成。
- **L52 EN**: Defines macro `__msa_srar_h` for conditional compilation, shorthand, or API generation.
  **L52 CN**: 定义宏 `__msa_srar_h`，用于条件编译、简写或 API 生成。
- **L53 EN**: Defines macro `__msa_srar_w` for conditional compilation, shorthand, or API generation.
  **L53 CN**: 定义宏 `__msa_srar_w`，用于条件编译、简写或 API 生成。
- **L54 EN**: Defines macro `__msa_srar_d` for conditional compilation, shorthand, or API generation.
  **L54 CN**: 定义宏 `__msa_srar_d`，用于条件编译、简写或 API 生成。
- **L55 EN**: Defines macro `__msa_srari_b` for conditional compilation, shorthand, or API generation.
  **L55 CN**: 定义宏 `__msa_srari_b`，用于条件编译、简写或 API 生成。
- **L56 EN**: Defines macro `__msa_srari_h` for conditional compilation, shorthand, or API generation.
  **L56 CN**: 定义宏 `__msa_srari_h`，用于条件编译、简写或 API 生成。
- **L57 EN**: Defines macro `__msa_srari_w` for conditional compilation, shorthand, or API generation.
  **L57 CN**: 定义宏 `__msa_srari_w`，用于条件编译、简写或 API 生成。
- **L58 EN**: Defines macro `__msa_srari_d` for conditional compilation, shorthand, or API generation.
  **L58 CN**: 定义宏 `__msa_srari_d`，用于条件编译、简写或 API 生成。
- **L59 EN**: Defines macro `__msa_srl_b` for conditional compilation, shorthand, or API generation.
  **L59 CN**: 定义宏 `__msa_srl_b`，用于条件编译、简写或 API 生成。
- **L60 EN**: Defines macro `__msa_srl_h` for conditional compilation, shorthand, or API generation.
  **L60 CN**: 定义宏 `__msa_srl_h`，用于条件编译、简写或 API 生成。

### Lines 61-80

````c
#define __msa_srl_w __builtin_msa_srl_w
#define __msa_srl_d __builtin_msa_srl_d
#define __msa_srli_b __builtin_msa_srli_b
#define __msa_srli_h __builtin_msa_srli_h
#define __msa_srli_w __builtin_msa_srli_w
#define __msa_srli_d __builtin_msa_srli_d
#define __msa_srlr_b __builtin_msa_srlr_b
#define __msa_srlr_h __builtin_msa_srlr_h
#define __msa_srlr_w __builtin_msa_srlr_w
#define __msa_srlr_d __builtin_msa_srlr_d
#define __msa_srlri_b __builtin_msa_srlri_b
#define __msa_srlri_h __builtin_msa_srlri_h
#define __msa_srlri_w __builtin_msa_srlri_w
#define __msa_srlri_d __builtin_msa_srlri_d
#define __msa_bclr_b __builtin_msa_bclr_b
#define __msa_bclr_h __builtin_msa_bclr_h
#define __msa_bclr_w __builtin_msa_bclr_w
#define __msa_bclr_d __builtin_msa_bclr_d
#define __msa_bclri_b __builtin_msa_bclri_b
#define __msa_bclri_h __builtin_msa_bclri_h
````
- **L61 EN**: Defines macro `__msa_srl_w` for conditional compilation, shorthand, or API generation.
  **L61 CN**: 定义宏 `__msa_srl_w`，用于条件编译、简写或 API 生成。
- **L62 EN**: Defines macro `__msa_srl_d` for conditional compilation, shorthand, or API generation.
  **L62 CN**: 定义宏 `__msa_srl_d`，用于条件编译、简写或 API 生成。
- **L63 EN**: Defines macro `__msa_srli_b` for conditional compilation, shorthand, or API generation.
  **L63 CN**: 定义宏 `__msa_srli_b`，用于条件编译、简写或 API 生成。
- **L64 EN**: Defines macro `__msa_srli_h` for conditional compilation, shorthand, or API generation.
  **L64 CN**: 定义宏 `__msa_srli_h`，用于条件编译、简写或 API 生成。
- **L65 EN**: Defines macro `__msa_srli_w` for conditional compilation, shorthand, or API generation.
  **L65 CN**: 定义宏 `__msa_srli_w`，用于条件编译、简写或 API 生成。
- **L66 EN**: Defines macro `__msa_srli_d` for conditional compilation, shorthand, or API generation.
  **L66 CN**: 定义宏 `__msa_srli_d`，用于条件编译、简写或 API 生成。
- **L67 EN**: Defines macro `__msa_srlr_b` for conditional compilation, shorthand, or API generation.
  **L67 CN**: 定义宏 `__msa_srlr_b`，用于条件编译、简写或 API 生成。
- **L68 EN**: Defines macro `__msa_srlr_h` for conditional compilation, shorthand, or API generation.
  **L68 CN**: 定义宏 `__msa_srlr_h`，用于条件编译、简写或 API 生成。
- **L69 EN**: Defines macro `__msa_srlr_w` for conditional compilation, shorthand, or API generation.
  **L69 CN**: 定义宏 `__msa_srlr_w`，用于条件编译、简写或 API 生成。
- **L70 EN**: Defines macro `__msa_srlr_d` for conditional compilation, shorthand, or API generation.
  **L70 CN**: 定义宏 `__msa_srlr_d`，用于条件编译、简写或 API 生成。
- **L71 EN**: Defines macro `__msa_srlri_b` for conditional compilation, shorthand, or API generation.
  **L71 CN**: 定义宏 `__msa_srlri_b`，用于条件编译、简写或 API 生成。
- **L72 EN**: Defines macro `__msa_srlri_h` for conditional compilation, shorthand, or API generation.
  **L72 CN**: 定义宏 `__msa_srlri_h`，用于条件编译、简写或 API 生成。
- **L73 EN**: Defines macro `__msa_srlri_w` for conditional compilation, shorthand, or API generation.
  **L73 CN**: 定义宏 `__msa_srlri_w`，用于条件编译、简写或 API 生成。
- **L74 EN**: Defines macro `__msa_srlri_d` for conditional compilation, shorthand, or API generation.
  **L74 CN**: 定义宏 `__msa_srlri_d`，用于条件编译、简写或 API 生成。
- **L75 EN**: Defines macro `__msa_bclr_b` for conditional compilation, shorthand, or API generation.
  **L75 CN**: 定义宏 `__msa_bclr_b`，用于条件编译、简写或 API 生成。
- **L76 EN**: Defines macro `__msa_bclr_h` for conditional compilation, shorthand, or API generation.
  **L76 CN**: 定义宏 `__msa_bclr_h`，用于条件编译、简写或 API 生成。
- **L77 EN**: Defines macro `__msa_bclr_w` for conditional compilation, shorthand, or API generation.
  **L77 CN**: 定义宏 `__msa_bclr_w`，用于条件编译、简写或 API 生成。
- **L78 EN**: Defines macro `__msa_bclr_d` for conditional compilation, shorthand, or API generation.
  **L78 CN**: 定义宏 `__msa_bclr_d`，用于条件编译、简写或 API 生成。
- **L79 EN**: Defines macro `__msa_bclri_b` for conditional compilation, shorthand, or API generation.
  **L79 CN**: 定义宏 `__msa_bclri_b`，用于条件编译、简写或 API 生成。
- **L80 EN**: Defines macro `__msa_bclri_h` for conditional compilation, shorthand, or API generation.
  **L80 CN**: 定义宏 `__msa_bclri_h`，用于条件编译、简写或 API 生成。

### Lines 81-100

````c
#define __msa_bclri_w __builtin_msa_bclri_w
#define __msa_bclri_d __builtin_msa_bclri_d
#define __msa_bset_b __builtin_msa_bset_b
#define __msa_bset_h __builtin_msa_bset_h
#define __msa_bset_w __builtin_msa_bset_w
#define __msa_bset_d __builtin_msa_bset_d
#define __msa_bseti_b __builtin_msa_bseti_b
#define __msa_bseti_h __builtin_msa_bseti_h
#define __msa_bseti_w __builtin_msa_bseti_w
#define __msa_bseti_d __builtin_msa_bseti_d
#define __msa_bneg_b __builtin_msa_bneg_b
#define __msa_bneg_h __builtin_msa_bneg_h
#define __msa_bneg_w __builtin_msa_bneg_w
#define __msa_bneg_d __builtin_msa_bneg_d
#define __msa_bnegi_b __builtin_msa_bnegi_b
#define __msa_bnegi_h __builtin_msa_bnegi_h
#define __msa_bnegi_w __builtin_msa_bnegi_w
#define __msa_bnegi_d __builtin_msa_bnegi_d
#define __msa_binsl_b __builtin_msa_binsl_b
#define __msa_binsl_h __builtin_msa_binsl_h
````
- **L81 EN**: Defines macro `__msa_bclri_w` for conditional compilation, shorthand, or API generation.
  **L81 CN**: 定义宏 `__msa_bclri_w`，用于条件编译、简写或 API 生成。
- **L82 EN**: Defines macro `__msa_bclri_d` for conditional compilation, shorthand, or API generation.
  **L82 CN**: 定义宏 `__msa_bclri_d`，用于条件编译、简写或 API 生成。
- **L83 EN**: Defines macro `__msa_bset_b` for conditional compilation, shorthand, or API generation.
  **L83 CN**: 定义宏 `__msa_bset_b`，用于条件编译、简写或 API 生成。
- **L84 EN**: Defines macro `__msa_bset_h` for conditional compilation, shorthand, or API generation.
  **L84 CN**: 定义宏 `__msa_bset_h`，用于条件编译、简写或 API 生成。
- **L85 EN**: Defines macro `__msa_bset_w` for conditional compilation, shorthand, or API generation.
  **L85 CN**: 定义宏 `__msa_bset_w`，用于条件编译、简写或 API 生成。
- **L86 EN**: Defines macro `__msa_bset_d` for conditional compilation, shorthand, or API generation.
  **L86 CN**: 定义宏 `__msa_bset_d`，用于条件编译、简写或 API 生成。
- **L87 EN**: Defines macro `__msa_bseti_b` for conditional compilation, shorthand, or API generation.
  **L87 CN**: 定义宏 `__msa_bseti_b`，用于条件编译、简写或 API 生成。
- **L88 EN**: Defines macro `__msa_bseti_h` for conditional compilation, shorthand, or API generation.
  **L88 CN**: 定义宏 `__msa_bseti_h`，用于条件编译、简写或 API 生成。
- **L89 EN**: Defines macro `__msa_bseti_w` for conditional compilation, shorthand, or API generation.
  **L89 CN**: 定义宏 `__msa_bseti_w`，用于条件编译、简写或 API 生成。
- **L90 EN**: Defines macro `__msa_bseti_d` for conditional compilation, shorthand, or API generation.
  **L90 CN**: 定义宏 `__msa_bseti_d`，用于条件编译、简写或 API 生成。
- **L91 EN**: Defines macro `__msa_bneg_b` for conditional compilation, shorthand, or API generation.
  **L91 CN**: 定义宏 `__msa_bneg_b`，用于条件编译、简写或 API 生成。
- **L92 EN**: Defines macro `__msa_bneg_h` for conditional compilation, shorthand, or API generation.
  **L92 CN**: 定义宏 `__msa_bneg_h`，用于条件编译、简写或 API 生成。
- **L93 EN**: Defines macro `__msa_bneg_w` for conditional compilation, shorthand, or API generation.
  **L93 CN**: 定义宏 `__msa_bneg_w`，用于条件编译、简写或 API 生成。
- **L94 EN**: Defines macro `__msa_bneg_d` for conditional compilation, shorthand, or API generation.
  **L94 CN**: 定义宏 `__msa_bneg_d`，用于条件编译、简写或 API 生成。
- **L95 EN**: Defines macro `__msa_bnegi_b` for conditional compilation, shorthand, or API generation.
  **L95 CN**: 定义宏 `__msa_bnegi_b`，用于条件编译、简写或 API 生成。
- **L96 EN**: Defines macro `__msa_bnegi_h` for conditional compilation, shorthand, or API generation.
  **L96 CN**: 定义宏 `__msa_bnegi_h`，用于条件编译、简写或 API 生成。
- **L97 EN**: Defines macro `__msa_bnegi_w` for conditional compilation, shorthand, or API generation.
  **L97 CN**: 定义宏 `__msa_bnegi_w`，用于条件编译、简写或 API 生成。
- **L98 EN**: Defines macro `__msa_bnegi_d` for conditional compilation, shorthand, or API generation.
  **L98 CN**: 定义宏 `__msa_bnegi_d`，用于条件编译、简写或 API 生成。
- **L99 EN**: Defines macro `__msa_binsl_b` for conditional compilation, shorthand, or API generation.
  **L99 CN**: 定义宏 `__msa_binsl_b`，用于条件编译、简写或 API 生成。
- **L100 EN**: Defines macro `__msa_binsl_h` for conditional compilation, shorthand, or API generation.
  **L100 CN**: 定义宏 `__msa_binsl_h`，用于条件编译、简写或 API 生成。

### Lines 101-120

````c
#define __msa_binsl_w __builtin_msa_binsl_w
#define __msa_binsl_d __builtin_msa_binsl_d
#define __msa_binsli_b __builtin_msa_binsli_b
#define __msa_binsli_h __builtin_msa_binsli_h
#define __msa_binsli_w __builtin_msa_binsli_w
#define __msa_binsli_d __builtin_msa_binsli_d
#define __msa_binsr_b __builtin_msa_binsr_b
#define __msa_binsr_h __builtin_msa_binsr_h
#define __msa_binsr_w __builtin_msa_binsr_w
#define __msa_binsr_d __builtin_msa_binsr_d
#define __msa_binsri_b __builtin_msa_binsri_b
#define __msa_binsri_h __builtin_msa_binsri_h
#define __msa_binsri_w __builtin_msa_binsri_w
#define __msa_binsri_d __builtin_msa_binsri_d
#define __msa_addv_b __builtin_msa_addv_b
#define __msa_addv_h __builtin_msa_addv_h
#define __msa_addv_w __builtin_msa_addv_w
#define __msa_addv_d __builtin_msa_addv_d
#define __msa_addvi_b __builtin_msa_addvi_b
#define __msa_addvi_h __builtin_msa_addvi_h
````
- **L101 EN**: Defines macro `__msa_binsl_w` for conditional compilation, shorthand, or API generation.
  **L101 CN**: 定义宏 `__msa_binsl_w`，用于条件编译、简写或 API 生成。
- **L102 EN**: Defines macro `__msa_binsl_d` for conditional compilation, shorthand, or API generation.
  **L102 CN**: 定义宏 `__msa_binsl_d`，用于条件编译、简写或 API 生成。
- **L103 EN**: Defines macro `__msa_binsli_b` for conditional compilation, shorthand, or API generation.
  **L103 CN**: 定义宏 `__msa_binsli_b`，用于条件编译、简写或 API 生成。
- **L104 EN**: Defines macro `__msa_binsli_h` for conditional compilation, shorthand, or API generation.
  **L104 CN**: 定义宏 `__msa_binsli_h`，用于条件编译、简写或 API 生成。
- **L105 EN**: Defines macro `__msa_binsli_w` for conditional compilation, shorthand, or API generation.
  **L105 CN**: 定义宏 `__msa_binsli_w`，用于条件编译、简写或 API 生成。
- **L106 EN**: Defines macro `__msa_binsli_d` for conditional compilation, shorthand, or API generation.
  **L106 CN**: 定义宏 `__msa_binsli_d`，用于条件编译、简写或 API 生成。
- **L107 EN**: Defines macro `__msa_binsr_b` for conditional compilation, shorthand, or API generation.
  **L107 CN**: 定义宏 `__msa_binsr_b`，用于条件编译、简写或 API 生成。
- **L108 EN**: Defines macro `__msa_binsr_h` for conditional compilation, shorthand, or API generation.
  **L108 CN**: 定义宏 `__msa_binsr_h`，用于条件编译、简写或 API 生成。
- **L109 EN**: Defines macro `__msa_binsr_w` for conditional compilation, shorthand, or API generation.
  **L109 CN**: 定义宏 `__msa_binsr_w`，用于条件编译、简写或 API 生成。
- **L110 EN**: Defines macro `__msa_binsr_d` for conditional compilation, shorthand, or API generation.
  **L110 CN**: 定义宏 `__msa_binsr_d`，用于条件编译、简写或 API 生成。
- **L111 EN**: Defines macro `__msa_binsri_b` for conditional compilation, shorthand, or API generation.
  **L111 CN**: 定义宏 `__msa_binsri_b`，用于条件编译、简写或 API 生成。
- **L112 EN**: Defines macro `__msa_binsri_h` for conditional compilation, shorthand, or API generation.
  **L112 CN**: 定义宏 `__msa_binsri_h`，用于条件编译、简写或 API 生成。
- **L113 EN**: Defines macro `__msa_binsri_w` for conditional compilation, shorthand, or API generation.
  **L113 CN**: 定义宏 `__msa_binsri_w`，用于条件编译、简写或 API 生成。
- **L114 EN**: Defines macro `__msa_binsri_d` for conditional compilation, shorthand, or API generation.
  **L114 CN**: 定义宏 `__msa_binsri_d`，用于条件编译、简写或 API 生成。
- **L115 EN**: Defines macro `__msa_addv_b` for conditional compilation, shorthand, or API generation.
  **L115 CN**: 定义宏 `__msa_addv_b`，用于条件编译、简写或 API 生成。
- **L116 EN**: Defines macro `__msa_addv_h` for conditional compilation, shorthand, or API generation.
  **L116 CN**: 定义宏 `__msa_addv_h`，用于条件编译、简写或 API 生成。
- **L117 EN**: Defines macro `__msa_addv_w` for conditional compilation, shorthand, or API generation.
  **L117 CN**: 定义宏 `__msa_addv_w`，用于条件编译、简写或 API 生成。
- **L118 EN**: Defines macro `__msa_addv_d` for conditional compilation, shorthand, or API generation.
  **L118 CN**: 定义宏 `__msa_addv_d`，用于条件编译、简写或 API 生成。
- **L119 EN**: Defines macro `__msa_addvi_b` for conditional compilation, shorthand, or API generation.
  **L119 CN**: 定义宏 `__msa_addvi_b`，用于条件编译、简写或 API 生成。
- **L120 EN**: Defines macro `__msa_addvi_h` for conditional compilation, shorthand, or API generation.
  **L120 CN**: 定义宏 `__msa_addvi_h`，用于条件编译、简写或 API 生成。

### Lines 121-140

````c
#define __msa_addvi_w __builtin_msa_addvi_w
#define __msa_addvi_d __builtin_msa_addvi_d
#define __msa_subv_b __builtin_msa_subv_b
#define __msa_subv_h __builtin_msa_subv_h
#define __msa_subv_w __builtin_msa_subv_w
#define __msa_subv_d __builtin_msa_subv_d
#define __msa_subvi_b __builtin_msa_subvi_b
#define __msa_subvi_h __builtin_msa_subvi_h
#define __msa_subvi_w __builtin_msa_subvi_w
#define __msa_subvi_d __builtin_msa_subvi_d
#define __msa_max_s_b __builtin_msa_max_s_b
#define __msa_max_s_h __builtin_msa_max_s_h
#define __msa_max_s_w __builtin_msa_max_s_w
#define __msa_max_s_d __builtin_msa_max_s_d
#define __msa_maxi_s_b __builtin_msa_maxi_s_b
#define __msa_maxi_s_h __builtin_msa_maxi_s_h
#define __msa_maxi_s_w __builtin_msa_maxi_s_w
#define __msa_maxi_s_d __builtin_msa_maxi_s_d
#define __msa_max_u_b __builtin_msa_max_u_b
#define __msa_max_u_h __builtin_msa_max_u_h
````
- **L121 EN**: Defines macro `__msa_addvi_w` for conditional compilation, shorthand, or API generation.
  **L121 CN**: 定义宏 `__msa_addvi_w`，用于条件编译、简写或 API 生成。
- **L122 EN**: Defines macro `__msa_addvi_d` for conditional compilation, shorthand, or API generation.
  **L122 CN**: 定义宏 `__msa_addvi_d`，用于条件编译、简写或 API 生成。
- **L123 EN**: Defines macro `__msa_subv_b` for conditional compilation, shorthand, or API generation.
  **L123 CN**: 定义宏 `__msa_subv_b`，用于条件编译、简写或 API 生成。
- **L124 EN**: Defines macro `__msa_subv_h` for conditional compilation, shorthand, or API generation.
  **L124 CN**: 定义宏 `__msa_subv_h`，用于条件编译、简写或 API 生成。
- **L125 EN**: Defines macro `__msa_subv_w` for conditional compilation, shorthand, or API generation.
  **L125 CN**: 定义宏 `__msa_subv_w`，用于条件编译、简写或 API 生成。
- **L126 EN**: Defines macro `__msa_subv_d` for conditional compilation, shorthand, or API generation.
  **L126 CN**: 定义宏 `__msa_subv_d`，用于条件编译、简写或 API 生成。
- **L127 EN**: Defines macro `__msa_subvi_b` for conditional compilation, shorthand, or API generation.
  **L127 CN**: 定义宏 `__msa_subvi_b`，用于条件编译、简写或 API 生成。
- **L128 EN**: Defines macro `__msa_subvi_h` for conditional compilation, shorthand, or API generation.
  **L128 CN**: 定义宏 `__msa_subvi_h`，用于条件编译、简写或 API 生成。
- **L129 EN**: Defines macro `__msa_subvi_w` for conditional compilation, shorthand, or API generation.
  **L129 CN**: 定义宏 `__msa_subvi_w`，用于条件编译、简写或 API 生成。
- **L130 EN**: Defines macro `__msa_subvi_d` for conditional compilation, shorthand, or API generation.
  **L130 CN**: 定义宏 `__msa_subvi_d`，用于条件编译、简写或 API 生成。
- **L131 EN**: Defines macro `__msa_max_s_b` for conditional compilation, shorthand, or API generation.
  **L131 CN**: 定义宏 `__msa_max_s_b`，用于条件编译、简写或 API 生成。
- **L132 EN**: Defines macro `__msa_max_s_h` for conditional compilation, shorthand, or API generation.
  **L132 CN**: 定义宏 `__msa_max_s_h`，用于条件编译、简写或 API 生成。
- **L133 EN**: Defines macro `__msa_max_s_w` for conditional compilation, shorthand, or API generation.
  **L133 CN**: 定义宏 `__msa_max_s_w`，用于条件编译、简写或 API 生成。
- **L134 EN**: Defines macro `__msa_max_s_d` for conditional compilation, shorthand, or API generation.
  **L134 CN**: 定义宏 `__msa_max_s_d`，用于条件编译、简写或 API 生成。
- **L135 EN**: Defines macro `__msa_maxi_s_b` for conditional compilation, shorthand, or API generation.
  **L135 CN**: 定义宏 `__msa_maxi_s_b`，用于条件编译、简写或 API 生成。
- **L136 EN**: Defines macro `__msa_maxi_s_h` for conditional compilation, shorthand, or API generation.
  **L136 CN**: 定义宏 `__msa_maxi_s_h`，用于条件编译、简写或 API 生成。
- **L137 EN**: Defines macro `__msa_maxi_s_w` for conditional compilation, shorthand, or API generation.
  **L137 CN**: 定义宏 `__msa_maxi_s_w`，用于条件编译、简写或 API 生成。
- **L138 EN**: Defines macro `__msa_maxi_s_d` for conditional compilation, shorthand, or API generation.
  **L138 CN**: 定义宏 `__msa_maxi_s_d`，用于条件编译、简写或 API 生成。
- **L139 EN**: Defines macro `__msa_max_u_b` for conditional compilation, shorthand, or API generation.
  **L139 CN**: 定义宏 `__msa_max_u_b`，用于条件编译、简写或 API 生成。
- **L140 EN**: Defines macro `__msa_max_u_h` for conditional compilation, shorthand, or API generation.
  **L140 CN**: 定义宏 `__msa_max_u_h`，用于条件编译、简写或 API 生成。

### Lines 141-160

````c
#define __msa_max_u_w __builtin_msa_max_u_w
#define __msa_max_u_d __builtin_msa_max_u_d
#define __msa_maxi_u_b __builtin_msa_maxi_u_b
#define __msa_maxi_u_h __builtin_msa_maxi_u_h
#define __msa_maxi_u_w __builtin_msa_maxi_u_w
#define __msa_maxi_u_d __builtin_msa_maxi_u_d
#define __msa_min_s_b __builtin_msa_min_s_b
#define __msa_min_s_h __builtin_msa_min_s_h
#define __msa_min_s_w __builtin_msa_min_s_w
#define __msa_min_s_d __builtin_msa_min_s_d
#define __msa_mini_s_b __builtin_msa_mini_s_b
#define __msa_mini_s_h __builtin_msa_mini_s_h
#define __msa_mini_s_w __builtin_msa_mini_s_w
#define __msa_mini_s_d __builtin_msa_mini_s_d
#define __msa_min_u_b __builtin_msa_min_u_b
#define __msa_min_u_h __builtin_msa_min_u_h
#define __msa_min_u_w __builtin_msa_min_u_w
#define __msa_min_u_d __builtin_msa_min_u_d
#define __msa_mini_u_b __builtin_msa_mini_u_b
#define __msa_mini_u_h __builtin_msa_mini_u_h
````
- **L141 EN**: Defines macro `__msa_max_u_w` for conditional compilation, shorthand, or API generation.
  **L141 CN**: 定义宏 `__msa_max_u_w`，用于条件编译、简写或 API 生成。
- **L142 EN**: Defines macro `__msa_max_u_d` for conditional compilation, shorthand, or API generation.
  **L142 CN**: 定义宏 `__msa_max_u_d`，用于条件编译、简写或 API 生成。
- **L143 EN**: Defines macro `__msa_maxi_u_b` for conditional compilation, shorthand, or API generation.
  **L143 CN**: 定义宏 `__msa_maxi_u_b`，用于条件编译、简写或 API 生成。
- **L144 EN**: Defines macro `__msa_maxi_u_h` for conditional compilation, shorthand, or API generation.
  **L144 CN**: 定义宏 `__msa_maxi_u_h`，用于条件编译、简写或 API 生成。
- **L145 EN**: Defines macro `__msa_maxi_u_w` for conditional compilation, shorthand, or API generation.
  **L145 CN**: 定义宏 `__msa_maxi_u_w`，用于条件编译、简写或 API 生成。
- **L146 EN**: Defines macro `__msa_maxi_u_d` for conditional compilation, shorthand, or API generation.
  **L146 CN**: 定义宏 `__msa_maxi_u_d`，用于条件编译、简写或 API 生成。
- **L147 EN**: Defines macro `__msa_min_s_b` for conditional compilation, shorthand, or API generation.
  **L147 CN**: 定义宏 `__msa_min_s_b`，用于条件编译、简写或 API 生成。
- **L148 EN**: Defines macro `__msa_min_s_h` for conditional compilation, shorthand, or API generation.
  **L148 CN**: 定义宏 `__msa_min_s_h`，用于条件编译、简写或 API 生成。
- **L149 EN**: Defines macro `__msa_min_s_w` for conditional compilation, shorthand, or API generation.
  **L149 CN**: 定义宏 `__msa_min_s_w`，用于条件编译、简写或 API 生成。
- **L150 EN**: Defines macro `__msa_min_s_d` for conditional compilation, shorthand, or API generation.
  **L150 CN**: 定义宏 `__msa_min_s_d`，用于条件编译、简写或 API 生成。
- **L151 EN**: Defines macro `__msa_mini_s_b` for conditional compilation, shorthand, or API generation.
  **L151 CN**: 定义宏 `__msa_mini_s_b`，用于条件编译、简写或 API 生成。
- **L152 EN**: Defines macro `__msa_mini_s_h` for conditional compilation, shorthand, or API generation.
  **L152 CN**: 定义宏 `__msa_mini_s_h`，用于条件编译、简写或 API 生成。
- **L153 EN**: Defines macro `__msa_mini_s_w` for conditional compilation, shorthand, or API generation.
  **L153 CN**: 定义宏 `__msa_mini_s_w`，用于条件编译、简写或 API 生成。
- **L154 EN**: Defines macro `__msa_mini_s_d` for conditional compilation, shorthand, or API generation.
  **L154 CN**: 定义宏 `__msa_mini_s_d`，用于条件编译、简写或 API 生成。
- **L155 EN**: Defines macro `__msa_min_u_b` for conditional compilation, shorthand, or API generation.
  **L155 CN**: 定义宏 `__msa_min_u_b`，用于条件编译、简写或 API 生成。
- **L156 EN**: Defines macro `__msa_min_u_h` for conditional compilation, shorthand, or API generation.
  **L156 CN**: 定义宏 `__msa_min_u_h`，用于条件编译、简写或 API 生成。
- **L157 EN**: Defines macro `__msa_min_u_w` for conditional compilation, shorthand, or API generation.
  **L157 CN**: 定义宏 `__msa_min_u_w`，用于条件编译、简写或 API 生成。
- **L158 EN**: Defines macro `__msa_min_u_d` for conditional compilation, shorthand, or API generation.
  **L158 CN**: 定义宏 `__msa_min_u_d`，用于条件编译、简写或 API 生成。
- **L159 EN**: Defines macro `__msa_mini_u_b` for conditional compilation, shorthand, or API generation.
  **L159 CN**: 定义宏 `__msa_mini_u_b`，用于条件编译、简写或 API 生成。
- **L160 EN**: Defines macro `__msa_mini_u_h` for conditional compilation, shorthand, or API generation.
  **L160 CN**: 定义宏 `__msa_mini_u_h`，用于条件编译、简写或 API 生成。

### Lines 161-180

````c
#define __msa_mini_u_w __builtin_msa_mini_u_w
#define __msa_mini_u_d __builtin_msa_mini_u_d
#define __msa_max_a_b __builtin_msa_max_a_b
#define __msa_max_a_h __builtin_msa_max_a_h
#define __msa_max_a_w __builtin_msa_max_a_w
#define __msa_max_a_d __builtin_msa_max_a_d
#define __msa_min_a_b __builtin_msa_min_a_b
#define __msa_min_a_h __builtin_msa_min_a_h
#define __msa_min_a_w __builtin_msa_min_a_w
#define __msa_min_a_d __builtin_msa_min_a_d
#define __msa_ceq_b __builtin_msa_ceq_b
#define __msa_ceq_h __builtin_msa_ceq_h
#define __msa_ceq_w __builtin_msa_ceq_w
#define __msa_ceq_d __builtin_msa_ceq_d
#define __msa_ceqi_b __builtin_msa_ceqi_b
#define __msa_ceqi_h __builtin_msa_ceqi_h
#define __msa_ceqi_w __builtin_msa_ceqi_w
#define __msa_ceqi_d __builtin_msa_ceqi_d
#define __msa_clt_s_b __builtin_msa_clt_s_b
#define __msa_clt_s_h __builtin_msa_clt_s_h
````
- **L161 EN**: Defines macro `__msa_mini_u_w` for conditional compilation, shorthand, or API generation.
  **L161 CN**: 定义宏 `__msa_mini_u_w`，用于条件编译、简写或 API 生成。
- **L162 EN**: Defines macro `__msa_mini_u_d` for conditional compilation, shorthand, or API generation.
  **L162 CN**: 定义宏 `__msa_mini_u_d`，用于条件编译、简写或 API 生成。
- **L163 EN**: Defines macro `__msa_max_a_b` for conditional compilation, shorthand, or API generation.
  **L163 CN**: 定义宏 `__msa_max_a_b`，用于条件编译、简写或 API 生成。
- **L164 EN**: Defines macro `__msa_max_a_h` for conditional compilation, shorthand, or API generation.
  **L164 CN**: 定义宏 `__msa_max_a_h`，用于条件编译、简写或 API 生成。
- **L165 EN**: Defines macro `__msa_max_a_w` for conditional compilation, shorthand, or API generation.
  **L165 CN**: 定义宏 `__msa_max_a_w`，用于条件编译、简写或 API 生成。
- **L166 EN**: Defines macro `__msa_max_a_d` for conditional compilation, shorthand, or API generation.
  **L166 CN**: 定义宏 `__msa_max_a_d`，用于条件编译、简写或 API 生成。
- **L167 EN**: Defines macro `__msa_min_a_b` for conditional compilation, shorthand, or API generation.
  **L167 CN**: 定义宏 `__msa_min_a_b`，用于条件编译、简写或 API 生成。
- **L168 EN**: Defines macro `__msa_min_a_h` for conditional compilation, shorthand, or API generation.
  **L168 CN**: 定义宏 `__msa_min_a_h`，用于条件编译、简写或 API 生成。
- **L169 EN**: Defines macro `__msa_min_a_w` for conditional compilation, shorthand, or API generation.
  **L169 CN**: 定义宏 `__msa_min_a_w`，用于条件编译、简写或 API 生成。
- **L170 EN**: Defines macro `__msa_min_a_d` for conditional compilation, shorthand, or API generation.
  **L170 CN**: 定义宏 `__msa_min_a_d`，用于条件编译、简写或 API 生成。
- **L171 EN**: Defines macro `__msa_ceq_b` for conditional compilation, shorthand, or API generation.
  **L171 CN**: 定义宏 `__msa_ceq_b`，用于条件编译、简写或 API 生成。
- **L172 EN**: Defines macro `__msa_ceq_h` for conditional compilation, shorthand, or API generation.
  **L172 CN**: 定义宏 `__msa_ceq_h`，用于条件编译、简写或 API 生成。
- **L173 EN**: Defines macro `__msa_ceq_w` for conditional compilation, shorthand, or API generation.
  **L173 CN**: 定义宏 `__msa_ceq_w`，用于条件编译、简写或 API 生成。
- **L174 EN**: Defines macro `__msa_ceq_d` for conditional compilation, shorthand, or API generation.
  **L174 CN**: 定义宏 `__msa_ceq_d`，用于条件编译、简写或 API 生成。
- **L175 EN**: Defines macro `__msa_ceqi_b` for conditional compilation, shorthand, or API generation.
  **L175 CN**: 定义宏 `__msa_ceqi_b`，用于条件编译、简写或 API 生成。
- **L176 EN**: Defines macro `__msa_ceqi_h` for conditional compilation, shorthand, or API generation.
  **L176 CN**: 定义宏 `__msa_ceqi_h`，用于条件编译、简写或 API 生成。
- **L177 EN**: Defines macro `__msa_ceqi_w` for conditional compilation, shorthand, or API generation.
  **L177 CN**: 定义宏 `__msa_ceqi_w`，用于条件编译、简写或 API 生成。
- **L178 EN**: Defines macro `__msa_ceqi_d` for conditional compilation, shorthand, or API generation.
  **L178 CN**: 定义宏 `__msa_ceqi_d`，用于条件编译、简写或 API 生成。
- **L179 EN**: Defines macro `__msa_clt_s_b` for conditional compilation, shorthand, or API generation.
  **L179 CN**: 定义宏 `__msa_clt_s_b`，用于条件编译、简写或 API 生成。
- **L180 EN**: Defines macro `__msa_clt_s_h` for conditional compilation, shorthand, or API generation.
  **L180 CN**: 定义宏 `__msa_clt_s_h`，用于条件编译、简写或 API 生成。

### Lines 181-200

````c
#define __msa_clt_s_w __builtin_msa_clt_s_w
#define __msa_clt_s_d __builtin_msa_clt_s_d
#define __msa_clti_s_b __builtin_msa_clti_s_b
#define __msa_clti_s_h __builtin_msa_clti_s_h
#define __msa_clti_s_w __builtin_msa_clti_s_w
#define __msa_clti_s_d __builtin_msa_clti_s_d
#define __msa_clt_u_b __builtin_msa_clt_u_b
#define __msa_clt_u_h __builtin_msa_clt_u_h
#define __msa_clt_u_w __builtin_msa_clt_u_w
#define __msa_clt_u_d __builtin_msa_clt_u_d
#define __msa_clti_u_b __builtin_msa_clti_u_b
#define __msa_clti_u_h __builtin_msa_clti_u_h
#define __msa_clti_u_w __builtin_msa_clti_u_w
#define __msa_clti_u_d __builtin_msa_clti_u_d
#define __msa_cle_s_b __builtin_msa_cle_s_b
#define __msa_cle_s_h __builtin_msa_cle_s_h
#define __msa_cle_s_w __builtin_msa_cle_s_w
#define __msa_cle_s_d __builtin_msa_cle_s_d
#define __msa_clei_s_b __builtin_msa_clei_s_b
#define __msa_clei_s_h __builtin_msa_clei_s_h
````
- **L181 EN**: Defines macro `__msa_clt_s_w` for conditional compilation, shorthand, or API generation.
  **L181 CN**: 定义宏 `__msa_clt_s_w`，用于条件编译、简写或 API 生成。
- **L182 EN**: Defines macro `__msa_clt_s_d` for conditional compilation, shorthand, or API generation.
  **L182 CN**: 定义宏 `__msa_clt_s_d`，用于条件编译、简写或 API 生成。
- **L183 EN**: Defines macro `__msa_clti_s_b` for conditional compilation, shorthand, or API generation.
  **L183 CN**: 定义宏 `__msa_clti_s_b`，用于条件编译、简写或 API 生成。
- **L184 EN**: Defines macro `__msa_clti_s_h` for conditional compilation, shorthand, or API generation.
  **L184 CN**: 定义宏 `__msa_clti_s_h`，用于条件编译、简写或 API 生成。
- **L185 EN**: Defines macro `__msa_clti_s_w` for conditional compilation, shorthand, or API generation.
  **L185 CN**: 定义宏 `__msa_clti_s_w`，用于条件编译、简写或 API 生成。
- **L186 EN**: Defines macro `__msa_clti_s_d` for conditional compilation, shorthand, or API generation.
  **L186 CN**: 定义宏 `__msa_clti_s_d`，用于条件编译、简写或 API 生成。
- **L187 EN**: Defines macro `__msa_clt_u_b` for conditional compilation, shorthand, or API generation.
  **L187 CN**: 定义宏 `__msa_clt_u_b`，用于条件编译、简写或 API 生成。
- **L188 EN**: Defines macro `__msa_clt_u_h` for conditional compilation, shorthand, or API generation.
  **L188 CN**: 定义宏 `__msa_clt_u_h`，用于条件编译、简写或 API 生成。
- **L189 EN**: Defines macro `__msa_clt_u_w` for conditional compilation, shorthand, or API generation.
  **L189 CN**: 定义宏 `__msa_clt_u_w`，用于条件编译、简写或 API 生成。
- **L190 EN**: Defines macro `__msa_clt_u_d` for conditional compilation, shorthand, or API generation.
  **L190 CN**: 定义宏 `__msa_clt_u_d`，用于条件编译、简写或 API 生成。
- **L191 EN**: Defines macro `__msa_clti_u_b` for conditional compilation, shorthand, or API generation.
  **L191 CN**: 定义宏 `__msa_clti_u_b`，用于条件编译、简写或 API 生成。
- **L192 EN**: Defines macro `__msa_clti_u_h` for conditional compilation, shorthand, or API generation.
  **L192 CN**: 定义宏 `__msa_clti_u_h`，用于条件编译、简写或 API 生成。
- **L193 EN**: Defines macro `__msa_clti_u_w` for conditional compilation, shorthand, or API generation.
  **L193 CN**: 定义宏 `__msa_clti_u_w`，用于条件编译、简写或 API 生成。
- **L194 EN**: Defines macro `__msa_clti_u_d` for conditional compilation, shorthand, or API generation.
  **L194 CN**: 定义宏 `__msa_clti_u_d`，用于条件编译、简写或 API 生成。
- **L195 EN**: Defines macro `__msa_cle_s_b` for conditional compilation, shorthand, or API generation.
  **L195 CN**: 定义宏 `__msa_cle_s_b`，用于条件编译、简写或 API 生成。
- **L196 EN**: Defines macro `__msa_cle_s_h` for conditional compilation, shorthand, or API generation.
  **L196 CN**: 定义宏 `__msa_cle_s_h`，用于条件编译、简写或 API 生成。
- **L197 EN**: Defines macro `__msa_cle_s_w` for conditional compilation, shorthand, or API generation.
  **L197 CN**: 定义宏 `__msa_cle_s_w`，用于条件编译、简写或 API 生成。
- **L198 EN**: Defines macro `__msa_cle_s_d` for conditional compilation, shorthand, or API generation.
  **L198 CN**: 定义宏 `__msa_cle_s_d`，用于条件编译、简写或 API 生成。
- **L199 EN**: Defines macro `__msa_clei_s_b` for conditional compilation, shorthand, or API generation.
  **L199 CN**: 定义宏 `__msa_clei_s_b`，用于条件编译、简写或 API 生成。
- **L200 EN**: Defines macro `__msa_clei_s_h` for conditional compilation, shorthand, or API generation.
  **L200 CN**: 定义宏 `__msa_clei_s_h`，用于条件编译、简写或 API 生成。

### Lines 201-220

````c
#define __msa_clei_s_w __builtin_msa_clei_s_w
#define __msa_clei_s_d __builtin_msa_clei_s_d
#define __msa_cle_u_b __builtin_msa_cle_u_b
#define __msa_cle_u_h __builtin_msa_cle_u_h
#define __msa_cle_u_w __builtin_msa_cle_u_w
#define __msa_cle_u_d __builtin_msa_cle_u_d
#define __msa_clei_u_b __builtin_msa_clei_u_b
#define __msa_clei_u_h __builtin_msa_clei_u_h
#define __msa_clei_u_w __builtin_msa_clei_u_w
#define __msa_clei_u_d __builtin_msa_clei_u_d
#define __msa_ld_b __builtin_msa_ld_b
#define __msa_ld_h __builtin_msa_ld_h
#define __msa_ld_w __builtin_msa_ld_w
#define __msa_ld_d __builtin_msa_ld_d
#define __msa_ldr_d __builtin_msa_ldr_d
#define __msa_ldr_w __builtin_msa_ldrq_w
#define __msa_st_b __builtin_msa_st_b
#define __msa_st_h __builtin_msa_st_h
#define __msa_st_w __builtin_msa_st_w
#define __msa_st_d __builtin_msa_st_d
````
- **L201 EN**: Defines macro `__msa_clei_s_w` for conditional compilation, shorthand, or API generation.
  **L201 CN**: 定义宏 `__msa_clei_s_w`，用于条件编译、简写或 API 生成。
- **L202 EN**: Defines macro `__msa_clei_s_d` for conditional compilation, shorthand, or API generation.
  **L202 CN**: 定义宏 `__msa_clei_s_d`，用于条件编译、简写或 API 生成。
- **L203 EN**: Defines macro `__msa_cle_u_b` for conditional compilation, shorthand, or API generation.
  **L203 CN**: 定义宏 `__msa_cle_u_b`，用于条件编译、简写或 API 生成。
- **L204 EN**: Defines macro `__msa_cle_u_h` for conditional compilation, shorthand, or API generation.
  **L204 CN**: 定义宏 `__msa_cle_u_h`，用于条件编译、简写或 API 生成。
- **L205 EN**: Defines macro `__msa_cle_u_w` for conditional compilation, shorthand, or API generation.
  **L205 CN**: 定义宏 `__msa_cle_u_w`，用于条件编译、简写或 API 生成。
- **L206 EN**: Defines macro `__msa_cle_u_d` for conditional compilation, shorthand, or API generation.
  **L206 CN**: 定义宏 `__msa_cle_u_d`，用于条件编译、简写或 API 生成。
- **L207 EN**: Defines macro `__msa_clei_u_b` for conditional compilation, shorthand, or API generation.
  **L207 CN**: 定义宏 `__msa_clei_u_b`，用于条件编译、简写或 API 生成。
- **L208 EN**: Defines macro `__msa_clei_u_h` for conditional compilation, shorthand, or API generation.
  **L208 CN**: 定义宏 `__msa_clei_u_h`，用于条件编译、简写或 API 生成。
- **L209 EN**: Defines macro `__msa_clei_u_w` for conditional compilation, shorthand, or API generation.
  **L209 CN**: 定义宏 `__msa_clei_u_w`，用于条件编译、简写或 API 生成。
- **L210 EN**: Defines macro `__msa_clei_u_d` for conditional compilation, shorthand, or API generation.
  **L210 CN**: 定义宏 `__msa_clei_u_d`，用于条件编译、简写或 API 生成。
- **L211 EN**: Defines macro `__msa_ld_b` for conditional compilation, shorthand, or API generation.
  **L211 CN**: 定义宏 `__msa_ld_b`，用于条件编译、简写或 API 生成。
- **L212 EN**: Defines macro `__msa_ld_h` for conditional compilation, shorthand, or API generation.
  **L212 CN**: 定义宏 `__msa_ld_h`，用于条件编译、简写或 API 生成。
- **L213 EN**: Defines macro `__msa_ld_w` for conditional compilation, shorthand, or API generation.
  **L213 CN**: 定义宏 `__msa_ld_w`，用于条件编译、简写或 API 生成。
- **L214 EN**: Defines macro `__msa_ld_d` for conditional compilation, shorthand, or API generation.
  **L214 CN**: 定义宏 `__msa_ld_d`，用于条件编译、简写或 API 生成。
- **L215 EN**: Defines macro `__msa_ldr_d` for conditional compilation, shorthand, or API generation.
  **L215 CN**: 定义宏 `__msa_ldr_d`，用于条件编译、简写或 API 生成。
- **L216 EN**: Defines macro `__msa_ldr_w` for conditional compilation, shorthand, or API generation.
  **L216 CN**: 定义宏 `__msa_ldr_w`，用于条件编译、简写或 API 生成。
- **L217 EN**: Defines macro `__msa_st_b` for conditional compilation, shorthand, or API generation.
  **L217 CN**: 定义宏 `__msa_st_b`，用于条件编译、简写或 API 生成。
- **L218 EN**: Defines macro `__msa_st_h` for conditional compilation, shorthand, or API generation.
  **L218 CN**: 定义宏 `__msa_st_h`，用于条件编译、简写或 API 生成。
- **L219 EN**: Defines macro `__msa_st_w` for conditional compilation, shorthand, or API generation.
  **L219 CN**: 定义宏 `__msa_st_w`，用于条件编译、简写或 API 生成。
- **L220 EN**: Defines macro `__msa_st_d` for conditional compilation, shorthand, or API generation.
  **L220 CN**: 定义宏 `__msa_st_d`，用于条件编译、简写或 API 生成。

### Lines 221-240

````c
#define __msa_str_d __builtin_msa_str_d
#define __msa_str_w __builtin_msa_strq_w
#define __msa_sat_s_b __builtin_msa_sat_s_b
#define __msa_sat_s_h __builtin_msa_sat_s_h
#define __msa_sat_s_w __builtin_msa_sat_s_w
#define __msa_sat_s_d __builtin_msa_sat_s_d
#define __msa_sat_u_b __builtin_msa_sat_u_b
#define __msa_sat_u_h __builtin_msa_sat_u_h
#define __msa_sat_u_w __builtin_msa_sat_u_w
#define __msa_sat_u_d __builtin_msa_sat_u_d
#define __msa_add_a_b __builtin_msa_add_a_b
#define __msa_add_a_h __builtin_msa_add_a_h
#define __msa_add_a_w __builtin_msa_add_a_w
#define __msa_add_a_d __builtin_msa_add_a_d
#define __msa_adds_a_b __builtin_msa_adds_a_b
#define __msa_adds_a_h __builtin_msa_adds_a_h
#define __msa_adds_a_w __builtin_msa_adds_a_w
#define __msa_adds_a_d __builtin_msa_adds_a_d
#define __msa_adds_s_b __builtin_msa_adds_s_b
#define __msa_adds_s_h __builtin_msa_adds_s_h
````
- **L221 EN**: Defines macro `__msa_str_d` for conditional compilation, shorthand, or API generation.
  **L221 CN**: 定义宏 `__msa_str_d`，用于条件编译、简写或 API 生成。
- **L222 EN**: Defines macro `__msa_str_w` for conditional compilation, shorthand, or API generation.
  **L222 CN**: 定义宏 `__msa_str_w`，用于条件编译、简写或 API 生成。
- **L223 EN**: Defines macro `__msa_sat_s_b` for conditional compilation, shorthand, or API generation.
  **L223 CN**: 定义宏 `__msa_sat_s_b`，用于条件编译、简写或 API 生成。
- **L224 EN**: Defines macro `__msa_sat_s_h` for conditional compilation, shorthand, or API generation.
  **L224 CN**: 定义宏 `__msa_sat_s_h`，用于条件编译、简写或 API 生成。
- **L225 EN**: Defines macro `__msa_sat_s_w` for conditional compilation, shorthand, or API generation.
  **L225 CN**: 定义宏 `__msa_sat_s_w`，用于条件编译、简写或 API 生成。
- **L226 EN**: Defines macro `__msa_sat_s_d` for conditional compilation, shorthand, or API generation.
  **L226 CN**: 定义宏 `__msa_sat_s_d`，用于条件编译、简写或 API 生成。
- **L227 EN**: Defines macro `__msa_sat_u_b` for conditional compilation, shorthand, or API generation.
  **L227 CN**: 定义宏 `__msa_sat_u_b`，用于条件编译、简写或 API 生成。
- **L228 EN**: Defines macro `__msa_sat_u_h` for conditional compilation, shorthand, or API generation.
  **L228 CN**: 定义宏 `__msa_sat_u_h`，用于条件编译、简写或 API 生成。
- **L229 EN**: Defines macro `__msa_sat_u_w` for conditional compilation, shorthand, or API generation.
  **L229 CN**: 定义宏 `__msa_sat_u_w`，用于条件编译、简写或 API 生成。
- **L230 EN**: Defines macro `__msa_sat_u_d` for conditional compilation, shorthand, or API generation.
  **L230 CN**: 定义宏 `__msa_sat_u_d`，用于条件编译、简写或 API 生成。
- **L231 EN**: Defines macro `__msa_add_a_b` for conditional compilation, shorthand, or API generation.
  **L231 CN**: 定义宏 `__msa_add_a_b`，用于条件编译、简写或 API 生成。
- **L232 EN**: Defines macro `__msa_add_a_h` for conditional compilation, shorthand, or API generation.
  **L232 CN**: 定义宏 `__msa_add_a_h`，用于条件编译、简写或 API 生成。
- **L233 EN**: Defines macro `__msa_add_a_w` for conditional compilation, shorthand, or API generation.
  **L233 CN**: 定义宏 `__msa_add_a_w`，用于条件编译、简写或 API 生成。
- **L234 EN**: Defines macro `__msa_add_a_d` for conditional compilation, shorthand, or API generation.
  **L234 CN**: 定义宏 `__msa_add_a_d`，用于条件编译、简写或 API 生成。
- **L235 EN**: Defines macro `__msa_adds_a_b` for conditional compilation, shorthand, or API generation.
  **L235 CN**: 定义宏 `__msa_adds_a_b`，用于条件编译、简写或 API 生成。
- **L236 EN**: Defines macro `__msa_adds_a_h` for conditional compilation, shorthand, or API generation.
  **L236 CN**: 定义宏 `__msa_adds_a_h`，用于条件编译、简写或 API 生成。
- **L237 EN**: Defines macro `__msa_adds_a_w` for conditional compilation, shorthand, or API generation.
  **L237 CN**: 定义宏 `__msa_adds_a_w`，用于条件编译、简写或 API 生成。
- **L238 EN**: Defines macro `__msa_adds_a_d` for conditional compilation, shorthand, or API generation.
  **L238 CN**: 定义宏 `__msa_adds_a_d`，用于条件编译、简写或 API 生成。
- **L239 EN**: Defines macro `__msa_adds_s_b` for conditional compilation, shorthand, or API generation.
  **L239 CN**: 定义宏 `__msa_adds_s_b`，用于条件编译、简写或 API 生成。
- **L240 EN**: Defines macro `__msa_adds_s_h` for conditional compilation, shorthand, or API generation.
  **L240 CN**: 定义宏 `__msa_adds_s_h`，用于条件编译、简写或 API 生成。

### Lines 241-260

````c
#define __msa_adds_s_w __builtin_msa_adds_s_w
#define __msa_adds_s_d __builtin_msa_adds_s_d
#define __msa_adds_u_b __builtin_msa_adds_u_b
#define __msa_adds_u_h __builtin_msa_adds_u_h
#define __msa_adds_u_w __builtin_msa_adds_u_w
#define __msa_adds_u_d __builtin_msa_adds_u_d
#define __msa_ave_s_b __builtin_msa_ave_s_b
#define __msa_ave_s_h __builtin_msa_ave_s_h
#define __msa_ave_s_w __builtin_msa_ave_s_w
#define __msa_ave_s_d __builtin_msa_ave_s_d
#define __msa_ave_u_b __builtin_msa_ave_u_b
#define __msa_ave_u_h __builtin_msa_ave_u_h
#define __msa_ave_u_w __builtin_msa_ave_u_w
#define __msa_ave_u_d __builtin_msa_ave_u_d
#define __msa_aver_s_b __builtin_msa_aver_s_b
#define __msa_aver_s_h __builtin_msa_aver_s_h
#define __msa_aver_s_w __builtin_msa_aver_s_w
#define __msa_aver_s_d __builtin_msa_aver_s_d
#define __msa_aver_u_b __builtin_msa_aver_u_b
#define __msa_aver_u_h __builtin_msa_aver_u_h
````
- **L241 EN**: Defines macro `__msa_adds_s_w` for conditional compilation, shorthand, or API generation.
  **L241 CN**: 定义宏 `__msa_adds_s_w`，用于条件编译、简写或 API 生成。
- **L242 EN**: Defines macro `__msa_adds_s_d` for conditional compilation, shorthand, or API generation.
  **L242 CN**: 定义宏 `__msa_adds_s_d`，用于条件编译、简写或 API 生成。
- **L243 EN**: Defines macro `__msa_adds_u_b` for conditional compilation, shorthand, or API generation.
  **L243 CN**: 定义宏 `__msa_adds_u_b`，用于条件编译、简写或 API 生成。
- **L244 EN**: Defines macro `__msa_adds_u_h` for conditional compilation, shorthand, or API generation.
  **L244 CN**: 定义宏 `__msa_adds_u_h`，用于条件编译、简写或 API 生成。
- **L245 EN**: Defines macro `__msa_adds_u_w` for conditional compilation, shorthand, or API generation.
  **L245 CN**: 定义宏 `__msa_adds_u_w`，用于条件编译、简写或 API 生成。
- **L246 EN**: Defines macro `__msa_adds_u_d` for conditional compilation, shorthand, or API generation.
  **L246 CN**: 定义宏 `__msa_adds_u_d`，用于条件编译、简写或 API 生成。
- **L247 EN**: Defines macro `__msa_ave_s_b` for conditional compilation, shorthand, or API generation.
  **L247 CN**: 定义宏 `__msa_ave_s_b`，用于条件编译、简写或 API 生成。
- **L248 EN**: Defines macro `__msa_ave_s_h` for conditional compilation, shorthand, or API generation.
  **L248 CN**: 定义宏 `__msa_ave_s_h`，用于条件编译、简写或 API 生成。
- **L249 EN**: Defines macro `__msa_ave_s_w` for conditional compilation, shorthand, or API generation.
  **L249 CN**: 定义宏 `__msa_ave_s_w`，用于条件编译、简写或 API 生成。
- **L250 EN**: Defines macro `__msa_ave_s_d` for conditional compilation, shorthand, or API generation.
  **L250 CN**: 定义宏 `__msa_ave_s_d`，用于条件编译、简写或 API 生成。
- **L251 EN**: Defines macro `__msa_ave_u_b` for conditional compilation, shorthand, or API generation.
  **L251 CN**: 定义宏 `__msa_ave_u_b`，用于条件编译、简写或 API 生成。
- **L252 EN**: Defines macro `__msa_ave_u_h` for conditional compilation, shorthand, or API generation.
  **L252 CN**: 定义宏 `__msa_ave_u_h`，用于条件编译、简写或 API 生成。
- **L253 EN**: Defines macro `__msa_ave_u_w` for conditional compilation, shorthand, or API generation.
  **L253 CN**: 定义宏 `__msa_ave_u_w`，用于条件编译、简写或 API 生成。
- **L254 EN**: Defines macro `__msa_ave_u_d` for conditional compilation, shorthand, or API generation.
  **L254 CN**: 定义宏 `__msa_ave_u_d`，用于条件编译、简写或 API 生成。
- **L255 EN**: Defines macro `__msa_aver_s_b` for conditional compilation, shorthand, or API generation.
  **L255 CN**: 定义宏 `__msa_aver_s_b`，用于条件编译、简写或 API 生成。
- **L256 EN**: Defines macro `__msa_aver_s_h` for conditional compilation, shorthand, or API generation.
  **L256 CN**: 定义宏 `__msa_aver_s_h`，用于条件编译、简写或 API 生成。
- **L257 EN**: Defines macro `__msa_aver_s_w` for conditional compilation, shorthand, or API generation.
  **L257 CN**: 定义宏 `__msa_aver_s_w`，用于条件编译、简写或 API 生成。
- **L258 EN**: Defines macro `__msa_aver_s_d` for conditional compilation, shorthand, or API generation.
  **L258 CN**: 定义宏 `__msa_aver_s_d`，用于条件编译、简写或 API 生成。
- **L259 EN**: Defines macro `__msa_aver_u_b` for conditional compilation, shorthand, or API generation.
  **L259 CN**: 定义宏 `__msa_aver_u_b`，用于条件编译、简写或 API 生成。
- **L260 EN**: Defines macro `__msa_aver_u_h` for conditional compilation, shorthand, or API generation.
  **L260 CN**: 定义宏 `__msa_aver_u_h`，用于条件编译、简写或 API 生成。

### Lines 261-280

````c
#define __msa_aver_u_w __builtin_msa_aver_u_w
#define __msa_aver_u_d __builtin_msa_aver_u_d
#define __msa_subs_s_b __builtin_msa_subs_s_b
#define __msa_subs_s_h __builtin_msa_subs_s_h
#define __msa_subs_s_w __builtin_msa_subs_s_w
#define __msa_subs_s_d __builtin_msa_subs_s_d
#define __msa_subs_u_b __builtin_msa_subs_u_b
#define __msa_subs_u_h __builtin_msa_subs_u_h
#define __msa_subs_u_w __builtin_msa_subs_u_w
#define __msa_subs_u_d __builtin_msa_subs_u_d
#define __msa_subsuu_s_b __builtin_msa_subsuu_s_b
#define __msa_subsuu_s_h __builtin_msa_subsuu_s_h
#define __msa_subsuu_s_w __builtin_msa_subsuu_s_w
#define __msa_subsuu_s_d __builtin_msa_subsuu_s_d
#define __msa_subsus_u_b __builtin_msa_subsus_u_b
#define __msa_subsus_u_h __builtin_msa_subsus_u_h
#define __msa_subsus_u_w __builtin_msa_subsus_u_w
#define __msa_subsus_u_d __builtin_msa_subsus_u_d
#define __msa_asub_s_b __builtin_msa_asub_s_b
#define __msa_asub_s_h __builtin_msa_asub_s_h
````
- **L261 EN**: Defines macro `__msa_aver_u_w` for conditional compilation, shorthand, or API generation.
  **L261 CN**: 定义宏 `__msa_aver_u_w`，用于条件编译、简写或 API 生成。
- **L262 EN**: Defines macro `__msa_aver_u_d` for conditional compilation, shorthand, or API generation.
  **L262 CN**: 定义宏 `__msa_aver_u_d`，用于条件编译、简写或 API 生成。
- **L263 EN**: Defines macro `__msa_subs_s_b` for conditional compilation, shorthand, or API generation.
  **L263 CN**: 定义宏 `__msa_subs_s_b`，用于条件编译、简写或 API 生成。
- **L264 EN**: Defines macro `__msa_subs_s_h` for conditional compilation, shorthand, or API generation.
  **L264 CN**: 定义宏 `__msa_subs_s_h`，用于条件编译、简写或 API 生成。
- **L265 EN**: Defines macro `__msa_subs_s_w` for conditional compilation, shorthand, or API generation.
  **L265 CN**: 定义宏 `__msa_subs_s_w`，用于条件编译、简写或 API 生成。
- **L266 EN**: Defines macro `__msa_subs_s_d` for conditional compilation, shorthand, or API generation.
  **L266 CN**: 定义宏 `__msa_subs_s_d`，用于条件编译、简写或 API 生成。
- **L267 EN**: Defines macro `__msa_subs_u_b` for conditional compilation, shorthand, or API generation.
  **L267 CN**: 定义宏 `__msa_subs_u_b`，用于条件编译、简写或 API 生成。
- **L268 EN**: Defines macro `__msa_subs_u_h` for conditional compilation, shorthand, or API generation.
  **L268 CN**: 定义宏 `__msa_subs_u_h`，用于条件编译、简写或 API 生成。
- **L269 EN**: Defines macro `__msa_subs_u_w` for conditional compilation, shorthand, or API generation.
  **L269 CN**: 定义宏 `__msa_subs_u_w`，用于条件编译、简写或 API 生成。
- **L270 EN**: Defines macro `__msa_subs_u_d` for conditional compilation, shorthand, or API generation.
  **L270 CN**: 定义宏 `__msa_subs_u_d`，用于条件编译、简写或 API 生成。
- **L271 EN**: Defines macro `__msa_subsuu_s_b` for conditional compilation, shorthand, or API generation.
  **L271 CN**: 定义宏 `__msa_subsuu_s_b`，用于条件编译、简写或 API 生成。
- **L272 EN**: Defines macro `__msa_subsuu_s_h` for conditional compilation, shorthand, or API generation.
  **L272 CN**: 定义宏 `__msa_subsuu_s_h`，用于条件编译、简写或 API 生成。
- **L273 EN**: Defines macro `__msa_subsuu_s_w` for conditional compilation, shorthand, or API generation.
  **L273 CN**: 定义宏 `__msa_subsuu_s_w`，用于条件编译、简写或 API 生成。
- **L274 EN**: Defines macro `__msa_subsuu_s_d` for conditional compilation, shorthand, or API generation.
  **L274 CN**: 定义宏 `__msa_subsuu_s_d`，用于条件编译、简写或 API 生成。
- **L275 EN**: Defines macro `__msa_subsus_u_b` for conditional compilation, shorthand, or API generation.
  **L275 CN**: 定义宏 `__msa_subsus_u_b`，用于条件编译、简写或 API 生成。
- **L276 EN**: Defines macro `__msa_subsus_u_h` for conditional compilation, shorthand, or API generation.
  **L276 CN**: 定义宏 `__msa_subsus_u_h`，用于条件编译、简写或 API 生成。
- **L277 EN**: Defines macro `__msa_subsus_u_w` for conditional compilation, shorthand, or API generation.
  **L277 CN**: 定义宏 `__msa_subsus_u_w`，用于条件编译、简写或 API 生成。
- **L278 EN**: Defines macro `__msa_subsus_u_d` for conditional compilation, shorthand, or API generation.
  **L278 CN**: 定义宏 `__msa_subsus_u_d`，用于条件编译、简写或 API 生成。
- **L279 EN**: Defines macro `__msa_asub_s_b` for conditional compilation, shorthand, or API generation.
  **L279 CN**: 定义宏 `__msa_asub_s_b`，用于条件编译、简写或 API 生成。
- **L280 EN**: Defines macro `__msa_asub_s_h` for conditional compilation, shorthand, or API generation.
  **L280 CN**: 定义宏 `__msa_asub_s_h`，用于条件编译、简写或 API 生成。

### Lines 281-300

````c
#define __msa_asub_s_w __builtin_msa_asub_s_w
#define __msa_asub_s_d __builtin_msa_asub_s_d
#define __msa_asub_u_b __builtin_msa_asub_u_b
#define __msa_asub_u_h __builtin_msa_asub_u_h
#define __msa_asub_u_w __builtin_msa_asub_u_w
#define __msa_asub_u_d __builtin_msa_asub_u_d
#define __msa_mulv_b __builtin_msa_mulv_b
#define __msa_mulv_h __builtin_msa_mulv_h
#define __msa_mulv_w __builtin_msa_mulv_w
#define __msa_mulv_d __builtin_msa_mulv_d
#define __msa_maddv_b __builtin_msa_maddv_b
#define __msa_maddv_h __builtin_msa_maddv_h
#define __msa_maddv_w __builtin_msa_maddv_w
#define __msa_maddv_d __builtin_msa_maddv_d
#define __msa_msubv_b __builtin_msa_msubv_b
#define __msa_msubv_h __builtin_msa_msubv_h
#define __msa_msubv_w __builtin_msa_msubv_w
#define __msa_msubv_d __builtin_msa_msubv_d
#define __msa_div_s_b __builtin_msa_div_s_b
#define __msa_div_s_h __builtin_msa_div_s_h
````
- **L281 EN**: Defines macro `__msa_asub_s_w` for conditional compilation, shorthand, or API generation.
  **L281 CN**: 定义宏 `__msa_asub_s_w`，用于条件编译、简写或 API 生成。
- **L282 EN**: Defines macro `__msa_asub_s_d` for conditional compilation, shorthand, or API generation.
  **L282 CN**: 定义宏 `__msa_asub_s_d`，用于条件编译、简写或 API 生成。
- **L283 EN**: Defines macro `__msa_asub_u_b` for conditional compilation, shorthand, or API generation.
  **L283 CN**: 定义宏 `__msa_asub_u_b`，用于条件编译、简写或 API 生成。
- **L284 EN**: Defines macro `__msa_asub_u_h` for conditional compilation, shorthand, or API generation.
  **L284 CN**: 定义宏 `__msa_asub_u_h`，用于条件编译、简写或 API 生成。
- **L285 EN**: Defines macro `__msa_asub_u_w` for conditional compilation, shorthand, or API generation.
  **L285 CN**: 定义宏 `__msa_asub_u_w`，用于条件编译、简写或 API 生成。
- **L286 EN**: Defines macro `__msa_asub_u_d` for conditional compilation, shorthand, or API generation.
  **L286 CN**: 定义宏 `__msa_asub_u_d`，用于条件编译、简写或 API 生成。
- **L287 EN**: Defines macro `__msa_mulv_b` for conditional compilation, shorthand, or API generation.
  **L287 CN**: 定义宏 `__msa_mulv_b`，用于条件编译、简写或 API 生成。
- **L288 EN**: Defines macro `__msa_mulv_h` for conditional compilation, shorthand, or API generation.
  **L288 CN**: 定义宏 `__msa_mulv_h`，用于条件编译、简写或 API 生成。
- **L289 EN**: Defines macro `__msa_mulv_w` for conditional compilation, shorthand, or API generation.
  **L289 CN**: 定义宏 `__msa_mulv_w`，用于条件编译、简写或 API 生成。
- **L290 EN**: Defines macro `__msa_mulv_d` for conditional compilation, shorthand, or API generation.
  **L290 CN**: 定义宏 `__msa_mulv_d`，用于条件编译、简写或 API 生成。
- **L291 EN**: Defines macro `__msa_maddv_b` for conditional compilation, shorthand, or API generation.
  **L291 CN**: 定义宏 `__msa_maddv_b`，用于条件编译、简写或 API 生成。
- **L292 EN**: Defines macro `__msa_maddv_h` for conditional compilation, shorthand, or API generation.
  **L292 CN**: 定义宏 `__msa_maddv_h`，用于条件编译、简写或 API 生成。
- **L293 EN**: Defines macro `__msa_maddv_w` for conditional compilation, shorthand, or API generation.
  **L293 CN**: 定义宏 `__msa_maddv_w`，用于条件编译、简写或 API 生成。
- **L294 EN**: Defines macro `__msa_maddv_d` for conditional compilation, shorthand, or API generation.
  **L294 CN**: 定义宏 `__msa_maddv_d`，用于条件编译、简写或 API 生成。
- **L295 EN**: Defines macro `__msa_msubv_b` for conditional compilation, shorthand, or API generation.
  **L295 CN**: 定义宏 `__msa_msubv_b`，用于条件编译、简写或 API 生成。
- **L296 EN**: Defines macro `__msa_msubv_h` for conditional compilation, shorthand, or API generation.
  **L296 CN**: 定义宏 `__msa_msubv_h`，用于条件编译、简写或 API 生成。
- **L297 EN**: Defines macro `__msa_msubv_w` for conditional compilation, shorthand, or API generation.
  **L297 CN**: 定义宏 `__msa_msubv_w`，用于条件编译、简写或 API 生成。
- **L298 EN**: Defines macro `__msa_msubv_d` for conditional compilation, shorthand, or API generation.
  **L298 CN**: 定义宏 `__msa_msubv_d`，用于条件编译、简写或 API 生成。
- **L299 EN**: Defines macro `__msa_div_s_b` for conditional compilation, shorthand, or API generation.
  **L299 CN**: 定义宏 `__msa_div_s_b`，用于条件编译、简写或 API 生成。
- **L300 EN**: Defines macro `__msa_div_s_h` for conditional compilation, shorthand, or API generation.
  **L300 CN**: 定义宏 `__msa_div_s_h`，用于条件编译、简写或 API 生成。

### Lines 301-320

````c
#define __msa_div_s_w __builtin_msa_div_s_w
#define __msa_div_s_d __builtin_msa_div_s_d
#define __msa_div_u_b __builtin_msa_div_u_b
#define __msa_div_u_h __builtin_msa_div_u_h
#define __msa_div_u_w __builtin_msa_div_u_w
#define __msa_div_u_d __builtin_msa_div_u_d
#define __msa_hadd_s_h __builtin_msa_hadd_s_h
#define __msa_hadd_s_w __builtin_msa_hadd_s_w
#define __msa_hadd_s_d __builtin_msa_hadd_s_d
#define __msa_hadd_u_h __builtin_msa_hadd_u_h
#define __msa_hadd_u_w __builtin_msa_hadd_u_w
#define __msa_hadd_u_d __builtin_msa_hadd_u_d
#define __msa_hsub_s_h __builtin_msa_hsub_s_h
#define __msa_hsub_s_w __builtin_msa_hsub_s_w
#define __msa_hsub_s_d __builtin_msa_hsub_s_d
#define __msa_hsub_u_h __builtin_msa_hsub_u_h
#define __msa_hsub_u_w __builtin_msa_hsub_u_w
#define __msa_hsub_u_d __builtin_msa_hsub_u_d
#define __msa_mod_s_b __builtin_msa_mod_s_b
#define __msa_mod_s_h __builtin_msa_mod_s_h
````
- **L301 EN**: Defines macro `__msa_div_s_w` for conditional compilation, shorthand, or API generation.
  **L301 CN**: 定义宏 `__msa_div_s_w`，用于条件编译、简写或 API 生成。
- **L302 EN**: Defines macro `__msa_div_s_d` for conditional compilation, shorthand, or API generation.
  **L302 CN**: 定义宏 `__msa_div_s_d`，用于条件编译、简写或 API 生成。
- **L303 EN**: Defines macro `__msa_div_u_b` for conditional compilation, shorthand, or API generation.
  **L303 CN**: 定义宏 `__msa_div_u_b`，用于条件编译、简写或 API 生成。
- **L304 EN**: Defines macro `__msa_div_u_h` for conditional compilation, shorthand, or API generation.
  **L304 CN**: 定义宏 `__msa_div_u_h`，用于条件编译、简写或 API 生成。
- **L305 EN**: Defines macro `__msa_div_u_w` for conditional compilation, shorthand, or API generation.
  **L305 CN**: 定义宏 `__msa_div_u_w`，用于条件编译、简写或 API 生成。
- **L306 EN**: Defines macro `__msa_div_u_d` for conditional compilation, shorthand, or API generation.
  **L306 CN**: 定义宏 `__msa_div_u_d`，用于条件编译、简写或 API 生成。
- **L307 EN**: Defines macro `__msa_hadd_s_h` for conditional compilation, shorthand, or API generation.
  **L307 CN**: 定义宏 `__msa_hadd_s_h`，用于条件编译、简写或 API 生成。
- **L308 EN**: Defines macro `__msa_hadd_s_w` for conditional compilation, shorthand, or API generation.
  **L308 CN**: 定义宏 `__msa_hadd_s_w`，用于条件编译、简写或 API 生成。
- **L309 EN**: Defines macro `__msa_hadd_s_d` for conditional compilation, shorthand, or API generation.
  **L309 CN**: 定义宏 `__msa_hadd_s_d`，用于条件编译、简写或 API 生成。
- **L310 EN**: Defines macro `__msa_hadd_u_h` for conditional compilation, shorthand, or API generation.
  **L310 CN**: 定义宏 `__msa_hadd_u_h`，用于条件编译、简写或 API 生成。
- **L311 EN**: Defines macro `__msa_hadd_u_w` for conditional compilation, shorthand, or API generation.
  **L311 CN**: 定义宏 `__msa_hadd_u_w`，用于条件编译、简写或 API 生成。
- **L312 EN**: Defines macro `__msa_hadd_u_d` for conditional compilation, shorthand, or API generation.
  **L312 CN**: 定义宏 `__msa_hadd_u_d`，用于条件编译、简写或 API 生成。
- **L313 EN**: Defines macro `__msa_hsub_s_h` for conditional compilation, shorthand, or API generation.
  **L313 CN**: 定义宏 `__msa_hsub_s_h`，用于条件编译、简写或 API 生成。
- **L314 EN**: Defines macro `__msa_hsub_s_w` for conditional compilation, shorthand, or API generation.
  **L314 CN**: 定义宏 `__msa_hsub_s_w`，用于条件编译、简写或 API 生成。
- **L315 EN**: Defines macro `__msa_hsub_s_d` for conditional compilation, shorthand, or API generation.
  **L315 CN**: 定义宏 `__msa_hsub_s_d`，用于条件编译、简写或 API 生成。
- **L316 EN**: Defines macro `__msa_hsub_u_h` for conditional compilation, shorthand, or API generation.
  **L316 CN**: 定义宏 `__msa_hsub_u_h`，用于条件编译、简写或 API 生成。
- **L317 EN**: Defines macro `__msa_hsub_u_w` for conditional compilation, shorthand, or API generation.
  **L317 CN**: 定义宏 `__msa_hsub_u_w`，用于条件编译、简写或 API 生成。
- **L318 EN**: Defines macro `__msa_hsub_u_d` for conditional compilation, shorthand, or API generation.
  **L318 CN**: 定义宏 `__msa_hsub_u_d`，用于条件编译、简写或 API 生成。
- **L319 EN**: Defines macro `__msa_mod_s_b` for conditional compilation, shorthand, or API generation.
  **L319 CN**: 定义宏 `__msa_mod_s_b`，用于条件编译、简写或 API 生成。
- **L320 EN**: Defines macro `__msa_mod_s_h` for conditional compilation, shorthand, or API generation.
  **L320 CN**: 定义宏 `__msa_mod_s_h`，用于条件编译、简写或 API 生成。

### Lines 321-340

````c
#define __msa_mod_s_w __builtin_msa_mod_s_w
#define __msa_mod_s_d __builtin_msa_mod_s_d
#define __msa_mod_u_b __builtin_msa_mod_u_b
#define __msa_mod_u_h __builtin_msa_mod_u_h
#define __msa_mod_u_w __builtin_msa_mod_u_w
#define __msa_mod_u_d __builtin_msa_mod_u_d
#define __msa_dotp_s_h __builtin_msa_dotp_s_h
#define __msa_dotp_s_w __builtin_msa_dotp_s_w
#define __msa_dotp_s_d __builtin_msa_dotp_s_d
#define __msa_dotp_u_h __builtin_msa_dotp_u_h
#define __msa_dotp_u_w __builtin_msa_dotp_u_w
#define __msa_dotp_u_d __builtin_msa_dotp_u_d
#define __msa_dpadd_s_h __builtin_msa_dpadd_s_h
#define __msa_dpadd_s_w __builtin_msa_dpadd_s_w
#define __msa_dpadd_s_d __builtin_msa_dpadd_s_d
#define __msa_dpadd_u_h __builtin_msa_dpadd_u_h
#define __msa_dpadd_u_w __builtin_msa_dpadd_u_w
#define __msa_dpadd_u_d __builtin_msa_dpadd_u_d
#define __msa_dpsub_s_h __builtin_msa_dpsub_s_h
#define __msa_dpsub_s_w __builtin_msa_dpsub_s_w
````
- **L321 EN**: Defines macro `__msa_mod_s_w` for conditional compilation, shorthand, or API generation.
  **L321 CN**: 定义宏 `__msa_mod_s_w`，用于条件编译、简写或 API 生成。
- **L322 EN**: Defines macro `__msa_mod_s_d` for conditional compilation, shorthand, or API generation.
  **L322 CN**: 定义宏 `__msa_mod_s_d`，用于条件编译、简写或 API 生成。
- **L323 EN**: Defines macro `__msa_mod_u_b` for conditional compilation, shorthand, or API generation.
  **L323 CN**: 定义宏 `__msa_mod_u_b`，用于条件编译、简写或 API 生成。
- **L324 EN**: Defines macro `__msa_mod_u_h` for conditional compilation, shorthand, or API generation.
  **L324 CN**: 定义宏 `__msa_mod_u_h`，用于条件编译、简写或 API 生成。
- **L325 EN**: Defines macro `__msa_mod_u_w` for conditional compilation, shorthand, or API generation.
  **L325 CN**: 定义宏 `__msa_mod_u_w`，用于条件编译、简写或 API 生成。
- **L326 EN**: Defines macro `__msa_mod_u_d` for conditional compilation, shorthand, or API generation.
  **L326 CN**: 定义宏 `__msa_mod_u_d`，用于条件编译、简写或 API 生成。
- **L327 EN**: Defines macro `__msa_dotp_s_h` for conditional compilation, shorthand, or API generation.
  **L327 CN**: 定义宏 `__msa_dotp_s_h`，用于条件编译、简写或 API 生成。
- **L328 EN**: Defines macro `__msa_dotp_s_w` for conditional compilation, shorthand, or API generation.
  **L328 CN**: 定义宏 `__msa_dotp_s_w`，用于条件编译、简写或 API 生成。
- **L329 EN**: Defines macro `__msa_dotp_s_d` for conditional compilation, shorthand, or API generation.
  **L329 CN**: 定义宏 `__msa_dotp_s_d`，用于条件编译、简写或 API 生成。
- **L330 EN**: Defines macro `__msa_dotp_u_h` for conditional compilation, shorthand, or API generation.
  **L330 CN**: 定义宏 `__msa_dotp_u_h`，用于条件编译、简写或 API 生成。
- **L331 EN**: Defines macro `__msa_dotp_u_w` for conditional compilation, shorthand, or API generation.
  **L331 CN**: 定义宏 `__msa_dotp_u_w`，用于条件编译、简写或 API 生成。
- **L332 EN**: Defines macro `__msa_dotp_u_d` for conditional compilation, shorthand, or API generation.
  **L332 CN**: 定义宏 `__msa_dotp_u_d`，用于条件编译、简写或 API 生成。
- **L333 EN**: Defines macro `__msa_dpadd_s_h` for conditional compilation, shorthand, or API generation.
  **L333 CN**: 定义宏 `__msa_dpadd_s_h`，用于条件编译、简写或 API 生成。
- **L334 EN**: Defines macro `__msa_dpadd_s_w` for conditional compilation, shorthand, or API generation.
  **L334 CN**: 定义宏 `__msa_dpadd_s_w`，用于条件编译、简写或 API 生成。
- **L335 EN**: Defines macro `__msa_dpadd_s_d` for conditional compilation, shorthand, or API generation.
  **L335 CN**: 定义宏 `__msa_dpadd_s_d`，用于条件编译、简写或 API 生成。
- **L336 EN**: Defines macro `__msa_dpadd_u_h` for conditional compilation, shorthand, or API generation.
  **L336 CN**: 定义宏 `__msa_dpadd_u_h`，用于条件编译、简写或 API 生成。
- **L337 EN**: Defines macro `__msa_dpadd_u_w` for conditional compilation, shorthand, or API generation.
  **L337 CN**: 定义宏 `__msa_dpadd_u_w`，用于条件编译、简写或 API 生成。
- **L338 EN**: Defines macro `__msa_dpadd_u_d` for conditional compilation, shorthand, or API generation.
  **L338 CN**: 定义宏 `__msa_dpadd_u_d`，用于条件编译、简写或 API 生成。
- **L339 EN**: Defines macro `__msa_dpsub_s_h` for conditional compilation, shorthand, or API generation.
  **L339 CN**: 定义宏 `__msa_dpsub_s_h`，用于条件编译、简写或 API 生成。
- **L340 EN**: Defines macro `__msa_dpsub_s_w` for conditional compilation, shorthand, or API generation.
  **L340 CN**: 定义宏 `__msa_dpsub_s_w`，用于条件编译、简写或 API 生成。

### Lines 341-360

````c
#define __msa_dpsub_s_d __builtin_msa_dpsub_s_d
#define __msa_dpsub_u_h __builtin_msa_dpsub_u_h
#define __msa_dpsub_u_w __builtin_msa_dpsub_u_w
#define __msa_dpsub_u_d __builtin_msa_dpsub_u_d
#define __msa_sld_b __builtin_msa_sld_b
#define __msa_sld_h __builtin_msa_sld_h
#define __msa_sld_w __builtin_msa_sld_w
#define __msa_sld_d __builtin_msa_sld_d
#define __msa_sldi_b __builtin_msa_sldi_b
#define __msa_sldi_h __builtin_msa_sldi_h
#define __msa_sldi_w __builtin_msa_sldi_w
#define __msa_sldi_d __builtin_msa_sldi_d
#define __msa_splat_b __builtin_msa_splat_b
#define __msa_splat_h __builtin_msa_splat_h
#define __msa_splat_w __builtin_msa_splat_w
#define __msa_splat_d __builtin_msa_splat_d
#define __msa_splati_b __builtin_msa_splati_b
#define __msa_splati_h __builtin_msa_splati_h
#define __msa_splati_w __builtin_msa_splati_w
#define __msa_splati_d __builtin_msa_splati_d
````
- **L341 EN**: Defines macro `__msa_dpsub_s_d` for conditional compilation, shorthand, or API generation.
  **L341 CN**: 定义宏 `__msa_dpsub_s_d`，用于条件编译、简写或 API 生成。
- **L342 EN**: Defines macro `__msa_dpsub_u_h` for conditional compilation, shorthand, or API generation.
  **L342 CN**: 定义宏 `__msa_dpsub_u_h`，用于条件编译、简写或 API 生成。
- **L343 EN**: Defines macro `__msa_dpsub_u_w` for conditional compilation, shorthand, or API generation.
  **L343 CN**: 定义宏 `__msa_dpsub_u_w`，用于条件编译、简写或 API 生成。
- **L344 EN**: Defines macro `__msa_dpsub_u_d` for conditional compilation, shorthand, or API generation.
  **L344 CN**: 定义宏 `__msa_dpsub_u_d`，用于条件编译、简写或 API 生成。
- **L345 EN**: Defines macro `__msa_sld_b` for conditional compilation, shorthand, or API generation.
  **L345 CN**: 定义宏 `__msa_sld_b`，用于条件编译、简写或 API 生成。
- **L346 EN**: Defines macro `__msa_sld_h` for conditional compilation, shorthand, or API generation.
  **L346 CN**: 定义宏 `__msa_sld_h`，用于条件编译、简写或 API 生成。
- **L347 EN**: Defines macro `__msa_sld_w` for conditional compilation, shorthand, or API generation.
  **L347 CN**: 定义宏 `__msa_sld_w`，用于条件编译、简写或 API 生成。
- **L348 EN**: Defines macro `__msa_sld_d` for conditional compilation, shorthand, or API generation.
  **L348 CN**: 定义宏 `__msa_sld_d`，用于条件编译、简写或 API 生成。
- **L349 EN**: Defines macro `__msa_sldi_b` for conditional compilation, shorthand, or API generation.
  **L349 CN**: 定义宏 `__msa_sldi_b`，用于条件编译、简写或 API 生成。
- **L350 EN**: Defines macro `__msa_sldi_h` for conditional compilation, shorthand, or API generation.
  **L350 CN**: 定义宏 `__msa_sldi_h`，用于条件编译、简写或 API 生成。
- **L351 EN**: Defines macro `__msa_sldi_w` for conditional compilation, shorthand, or API generation.
  **L351 CN**: 定义宏 `__msa_sldi_w`，用于条件编译、简写或 API 生成。
- **L352 EN**: Defines macro `__msa_sldi_d` for conditional compilation, shorthand, or API generation.
  **L352 CN**: 定义宏 `__msa_sldi_d`，用于条件编译、简写或 API 生成。
- **L353 EN**: Defines macro `__msa_splat_b` for conditional compilation, shorthand, or API generation.
  **L353 CN**: 定义宏 `__msa_splat_b`，用于条件编译、简写或 API 生成。
- **L354 EN**: Defines macro `__msa_splat_h` for conditional compilation, shorthand, or API generation.
  **L354 CN**: 定义宏 `__msa_splat_h`，用于条件编译、简写或 API 生成。
- **L355 EN**: Defines macro `__msa_splat_w` for conditional compilation, shorthand, or API generation.
  **L355 CN**: 定义宏 `__msa_splat_w`，用于条件编译、简写或 API 生成。
- **L356 EN**: Defines macro `__msa_splat_d` for conditional compilation, shorthand, or API generation.
  **L356 CN**: 定义宏 `__msa_splat_d`，用于条件编译、简写或 API 生成。
- **L357 EN**: Defines macro `__msa_splati_b` for conditional compilation, shorthand, or API generation.
  **L357 CN**: 定义宏 `__msa_splati_b`，用于条件编译、简写或 API 生成。
- **L358 EN**: Defines macro `__msa_splati_h` for conditional compilation, shorthand, or API generation.
  **L358 CN**: 定义宏 `__msa_splati_h`，用于条件编译、简写或 API 生成。
- **L359 EN**: Defines macro `__msa_splati_w` for conditional compilation, shorthand, or API generation.
  **L359 CN**: 定义宏 `__msa_splati_w`，用于条件编译、简写或 API 生成。
- **L360 EN**: Defines macro `__msa_splati_d` for conditional compilation, shorthand, or API generation.
  **L360 CN**: 定义宏 `__msa_splati_d`，用于条件编译、简写或 API 生成。

### Lines 361-380

````c
#define __msa_pckev_b __builtin_msa_pckev_b
#define __msa_pckev_h __builtin_msa_pckev_h
#define __msa_pckev_w __builtin_msa_pckev_w
#define __msa_pckev_d __builtin_msa_pckev_d
#define __msa_pckod_b __builtin_msa_pckod_b
#define __msa_pckod_h __builtin_msa_pckod_h
#define __msa_pckod_w __builtin_msa_pckod_w
#define __msa_pckod_d __builtin_msa_pckod_d
#define __msa_ilvl_b __builtin_msa_ilvl_b
#define __msa_ilvl_h __builtin_msa_ilvl_h
#define __msa_ilvl_w __builtin_msa_ilvl_w
#define __msa_ilvl_d __builtin_msa_ilvl_d
#define __msa_ilvr_b __builtin_msa_ilvr_b
#define __msa_ilvr_h __builtin_msa_ilvr_h
#define __msa_ilvr_w __builtin_msa_ilvr_w
#define __msa_ilvr_d __builtin_msa_ilvr_d
#define __msa_ilvev_b __builtin_msa_ilvev_b
#define __msa_ilvev_h __builtin_msa_ilvev_h
#define __msa_ilvev_w __builtin_msa_ilvev_w
#define __msa_ilvev_d __builtin_msa_ilvev_d
````
- **L361 EN**: Defines macro `__msa_pckev_b` for conditional compilation, shorthand, or API generation.
  **L361 CN**: 定义宏 `__msa_pckev_b`，用于条件编译、简写或 API 生成。
- **L362 EN**: Defines macro `__msa_pckev_h` for conditional compilation, shorthand, or API generation.
  **L362 CN**: 定义宏 `__msa_pckev_h`，用于条件编译、简写或 API 生成。
- **L363 EN**: Defines macro `__msa_pckev_w` for conditional compilation, shorthand, or API generation.
  **L363 CN**: 定义宏 `__msa_pckev_w`，用于条件编译、简写或 API 生成。
- **L364 EN**: Defines macro `__msa_pckev_d` for conditional compilation, shorthand, or API generation.
  **L364 CN**: 定义宏 `__msa_pckev_d`，用于条件编译、简写或 API 生成。
- **L365 EN**: Defines macro `__msa_pckod_b` for conditional compilation, shorthand, or API generation.
  **L365 CN**: 定义宏 `__msa_pckod_b`，用于条件编译、简写或 API 生成。
- **L366 EN**: Defines macro `__msa_pckod_h` for conditional compilation, shorthand, or API generation.
  **L366 CN**: 定义宏 `__msa_pckod_h`，用于条件编译、简写或 API 生成。
- **L367 EN**: Defines macro `__msa_pckod_w` for conditional compilation, shorthand, or API generation.
  **L367 CN**: 定义宏 `__msa_pckod_w`，用于条件编译、简写或 API 生成。
- **L368 EN**: Defines macro `__msa_pckod_d` for conditional compilation, shorthand, or API generation.
  **L368 CN**: 定义宏 `__msa_pckod_d`，用于条件编译、简写或 API 生成。
- **L369 EN**: Defines macro `__msa_ilvl_b` for conditional compilation, shorthand, or API generation.
  **L369 CN**: 定义宏 `__msa_ilvl_b`，用于条件编译、简写或 API 生成。
- **L370 EN**: Defines macro `__msa_ilvl_h` for conditional compilation, shorthand, or API generation.
  **L370 CN**: 定义宏 `__msa_ilvl_h`，用于条件编译、简写或 API 生成。
- **L371 EN**: Defines macro `__msa_ilvl_w` for conditional compilation, shorthand, or API generation.
  **L371 CN**: 定义宏 `__msa_ilvl_w`，用于条件编译、简写或 API 生成。
- **L372 EN**: Defines macro `__msa_ilvl_d` for conditional compilation, shorthand, or API generation.
  **L372 CN**: 定义宏 `__msa_ilvl_d`，用于条件编译、简写或 API 生成。
- **L373 EN**: Defines macro `__msa_ilvr_b` for conditional compilation, shorthand, or API generation.
  **L373 CN**: 定义宏 `__msa_ilvr_b`，用于条件编译、简写或 API 生成。
- **L374 EN**: Defines macro `__msa_ilvr_h` for conditional compilation, shorthand, or API generation.
  **L374 CN**: 定义宏 `__msa_ilvr_h`，用于条件编译、简写或 API 生成。
- **L375 EN**: Defines macro `__msa_ilvr_w` for conditional compilation, shorthand, or API generation.
  **L375 CN**: 定义宏 `__msa_ilvr_w`，用于条件编译、简写或 API 生成。
- **L376 EN**: Defines macro `__msa_ilvr_d` for conditional compilation, shorthand, or API generation.
  **L376 CN**: 定义宏 `__msa_ilvr_d`，用于条件编译、简写或 API 生成。
- **L377 EN**: Defines macro `__msa_ilvev_b` for conditional compilation, shorthand, or API generation.
  **L377 CN**: 定义宏 `__msa_ilvev_b`，用于条件编译、简写或 API 生成。
- **L378 EN**: Defines macro `__msa_ilvev_h` for conditional compilation, shorthand, or API generation.
  **L378 CN**: 定义宏 `__msa_ilvev_h`，用于条件编译、简写或 API 生成。
- **L379 EN**: Defines macro `__msa_ilvev_w` for conditional compilation, shorthand, or API generation.
  **L379 CN**: 定义宏 `__msa_ilvev_w`，用于条件编译、简写或 API 生成。
- **L380 EN**: Defines macro `__msa_ilvev_d` for conditional compilation, shorthand, or API generation.
  **L380 CN**: 定义宏 `__msa_ilvev_d`，用于条件编译、简写或 API 生成。

### Lines 381-400

````c
#define __msa_ilvod_b __builtin_msa_ilvod_b
#define __msa_ilvod_h __builtin_msa_ilvod_h
#define __msa_ilvod_w __builtin_msa_ilvod_w
#define __msa_ilvod_d __builtin_msa_ilvod_d
#define __msa_vshf_b __builtin_msa_vshf_b
#define __msa_vshf_h __builtin_msa_vshf_h
#define __msa_vshf_w __builtin_msa_vshf_w
#define __msa_vshf_d __builtin_msa_vshf_d
#define __msa_and_v __builtin_msa_and_v
#define __msa_andi_b __builtin_msa_andi_b
#define __msa_or_v __builtin_msa_or_v
#define __msa_ori_b __builtin_msa_ori_b
#define __msa_nor_v __builtin_msa_nor_v
#define __msa_nori_b __builtin_msa_nori_b
#define __msa_xor_v __builtin_msa_xor_v
#define __msa_xori_b __builtin_msa_xori_b
#define __msa_bmnz_v __builtin_msa_bmnz_v
#define __msa_bmnzi_b __builtin_msa_bmnzi_b
#define __msa_bmz_v __builtin_msa_bmz_v
#define __msa_bmzi_b __builtin_msa_bmzi_b
````
- **L381 EN**: Defines macro `__msa_ilvod_b` for conditional compilation, shorthand, or API generation.
  **L381 CN**: 定义宏 `__msa_ilvod_b`，用于条件编译、简写或 API 生成。
- **L382 EN**: Defines macro `__msa_ilvod_h` for conditional compilation, shorthand, or API generation.
  **L382 CN**: 定义宏 `__msa_ilvod_h`，用于条件编译、简写或 API 生成。
- **L383 EN**: Defines macro `__msa_ilvod_w` for conditional compilation, shorthand, or API generation.
  **L383 CN**: 定义宏 `__msa_ilvod_w`，用于条件编译、简写或 API 生成。
- **L384 EN**: Defines macro `__msa_ilvod_d` for conditional compilation, shorthand, or API generation.
  **L384 CN**: 定义宏 `__msa_ilvod_d`，用于条件编译、简写或 API 生成。
- **L385 EN**: Defines macro `__msa_vshf_b` for conditional compilation, shorthand, or API generation.
  **L385 CN**: 定义宏 `__msa_vshf_b`，用于条件编译、简写或 API 生成。
- **L386 EN**: Defines macro `__msa_vshf_h` for conditional compilation, shorthand, or API generation.
  **L386 CN**: 定义宏 `__msa_vshf_h`，用于条件编译、简写或 API 生成。
- **L387 EN**: Defines macro `__msa_vshf_w` for conditional compilation, shorthand, or API generation.
  **L387 CN**: 定义宏 `__msa_vshf_w`，用于条件编译、简写或 API 生成。
- **L388 EN**: Defines macro `__msa_vshf_d` for conditional compilation, shorthand, or API generation.
  **L388 CN**: 定义宏 `__msa_vshf_d`，用于条件编译、简写或 API 生成。
- **L389 EN**: Defines macro `__msa_and_v` for conditional compilation, shorthand, or API generation.
  **L389 CN**: 定义宏 `__msa_and_v`，用于条件编译、简写或 API 生成。
- **L390 EN**: Defines macro `__msa_andi_b` for conditional compilation, shorthand, or API generation.
  **L390 CN**: 定义宏 `__msa_andi_b`，用于条件编译、简写或 API 生成。
- **L391 EN**: Defines macro `__msa_or_v` for conditional compilation, shorthand, or API generation.
  **L391 CN**: 定义宏 `__msa_or_v`，用于条件编译、简写或 API 生成。
- **L392 EN**: Defines macro `__msa_ori_b` for conditional compilation, shorthand, or API generation.
  **L392 CN**: 定义宏 `__msa_ori_b`，用于条件编译、简写或 API 生成。
- **L393 EN**: Defines macro `__msa_nor_v` for conditional compilation, shorthand, or API generation.
  **L393 CN**: 定义宏 `__msa_nor_v`，用于条件编译、简写或 API 生成。
- **L394 EN**: Defines macro `__msa_nori_b` for conditional compilation, shorthand, or API generation.
  **L394 CN**: 定义宏 `__msa_nori_b`，用于条件编译、简写或 API 生成。
- **L395 EN**: Defines macro `__msa_xor_v` for conditional compilation, shorthand, or API generation.
  **L395 CN**: 定义宏 `__msa_xor_v`，用于条件编译、简写或 API 生成。
- **L396 EN**: Defines macro `__msa_xori_b` for conditional compilation, shorthand, or API generation.
  **L396 CN**: 定义宏 `__msa_xori_b`，用于条件编译、简写或 API 生成。
- **L397 EN**: Defines macro `__msa_bmnz_v` for conditional compilation, shorthand, or API generation.
  **L397 CN**: 定义宏 `__msa_bmnz_v`，用于条件编译、简写或 API 生成。
- **L398 EN**: Defines macro `__msa_bmnzi_b` for conditional compilation, shorthand, or API generation.
  **L398 CN**: 定义宏 `__msa_bmnzi_b`，用于条件编译、简写或 API 生成。
- **L399 EN**: Defines macro `__msa_bmz_v` for conditional compilation, shorthand, or API generation.
  **L399 CN**: 定义宏 `__msa_bmz_v`，用于条件编译、简写或 API 生成。
- **L400 EN**: Defines macro `__msa_bmzi_b` for conditional compilation, shorthand, or API generation.
  **L400 CN**: 定义宏 `__msa_bmzi_b`，用于条件编译、简写或 API 生成。

### Lines 401-420

````c
#define __msa_bsel_v __builtin_msa_bsel_v
#define __msa_bseli_b __builtin_msa_bseli_b
#define __msa_shf_b __builtin_msa_shf_b
#define __msa_shf_h __builtin_msa_shf_h
#define __msa_shf_w __builtin_msa_shf_w
#define __msa_test_bnz_v __builtin_msa_bnz_v
#define __msa_test_bz_v __builtin_msa_bz_v
#define __msa_fill_b __builtin_msa_fill_b
#define __msa_fill_h __builtin_msa_fill_h
#define __msa_fill_w __builtin_msa_fill_w
#define __msa_fill_d __builtin_msa_fill_d
#define __msa_pcnt_b __builtin_msa_pcnt_b
#define __msa_pcnt_h __builtin_msa_pcnt_h
#define __msa_pcnt_w __builtin_msa_pcnt_w
#define __msa_pcnt_d __builtin_msa_pcnt_d
#define __msa_nloc_b __builtin_msa_nloc_b
#define __msa_nloc_h __builtin_msa_nloc_h
#define __msa_nloc_w __builtin_msa_nloc_w
#define __msa_nloc_d __builtin_msa_nloc_d
#define __msa_nlzc_b __builtin_msa_nlzc_b
````
- **L401 EN**: Defines macro `__msa_bsel_v` for conditional compilation, shorthand, or API generation.
  **L401 CN**: 定义宏 `__msa_bsel_v`，用于条件编译、简写或 API 生成。
- **L402 EN**: Defines macro `__msa_bseli_b` for conditional compilation, shorthand, or API generation.
  **L402 CN**: 定义宏 `__msa_bseli_b`，用于条件编译、简写或 API 生成。
- **L403 EN**: Defines macro `__msa_shf_b` for conditional compilation, shorthand, or API generation.
  **L403 CN**: 定义宏 `__msa_shf_b`，用于条件编译、简写或 API 生成。
- **L404 EN**: Defines macro `__msa_shf_h` for conditional compilation, shorthand, or API generation.
  **L404 CN**: 定义宏 `__msa_shf_h`，用于条件编译、简写或 API 生成。
- **L405 EN**: Defines macro `__msa_shf_w` for conditional compilation, shorthand, or API generation.
  **L405 CN**: 定义宏 `__msa_shf_w`，用于条件编译、简写或 API 生成。
- **L406 EN**: Defines macro `__msa_test_bnz_v` for conditional compilation, shorthand, or API generation.
  **L406 CN**: 定义宏 `__msa_test_bnz_v`，用于条件编译、简写或 API 生成。
- **L407 EN**: Defines macro `__msa_test_bz_v` for conditional compilation, shorthand, or API generation.
  **L407 CN**: 定义宏 `__msa_test_bz_v`，用于条件编译、简写或 API 生成。
- **L408 EN**: Defines macro `__msa_fill_b` for conditional compilation, shorthand, or API generation.
  **L408 CN**: 定义宏 `__msa_fill_b`，用于条件编译、简写或 API 生成。
- **L409 EN**: Defines macro `__msa_fill_h` for conditional compilation, shorthand, or API generation.
  **L409 CN**: 定义宏 `__msa_fill_h`，用于条件编译、简写或 API 生成。
- **L410 EN**: Defines macro `__msa_fill_w` for conditional compilation, shorthand, or API generation.
  **L410 CN**: 定义宏 `__msa_fill_w`，用于条件编译、简写或 API 生成。
- **L411 EN**: Defines macro `__msa_fill_d` for conditional compilation, shorthand, or API generation.
  **L411 CN**: 定义宏 `__msa_fill_d`，用于条件编译、简写或 API 生成。
- **L412 EN**: Defines macro `__msa_pcnt_b` for conditional compilation, shorthand, or API generation.
  **L412 CN**: 定义宏 `__msa_pcnt_b`，用于条件编译、简写或 API 生成。
- **L413 EN**: Defines macro `__msa_pcnt_h` for conditional compilation, shorthand, or API generation.
  **L413 CN**: 定义宏 `__msa_pcnt_h`，用于条件编译、简写或 API 生成。
- **L414 EN**: Defines macro `__msa_pcnt_w` for conditional compilation, shorthand, or API generation.
  **L414 CN**: 定义宏 `__msa_pcnt_w`，用于条件编译、简写或 API 生成。
- **L415 EN**: Defines macro `__msa_pcnt_d` for conditional compilation, shorthand, or API generation.
  **L415 CN**: 定义宏 `__msa_pcnt_d`，用于条件编译、简写或 API 生成。
- **L416 EN**: Defines macro `__msa_nloc_b` for conditional compilation, shorthand, or API generation.
  **L416 CN**: 定义宏 `__msa_nloc_b`，用于条件编译、简写或 API 生成。
- **L417 EN**: Defines macro `__msa_nloc_h` for conditional compilation, shorthand, or API generation.
  **L417 CN**: 定义宏 `__msa_nloc_h`，用于条件编译、简写或 API 生成。
- **L418 EN**: Defines macro `__msa_nloc_w` for conditional compilation, shorthand, or API generation.
  **L418 CN**: 定义宏 `__msa_nloc_w`，用于条件编译、简写或 API 生成。
- **L419 EN**: Defines macro `__msa_nloc_d` for conditional compilation, shorthand, or API generation.
  **L419 CN**: 定义宏 `__msa_nloc_d`，用于条件编译、简写或 API 生成。
- **L420 EN**: Defines macro `__msa_nlzc_b` for conditional compilation, shorthand, or API generation.
  **L420 CN**: 定义宏 `__msa_nlzc_b`，用于条件编译、简写或 API 生成。

### Lines 421-440

````c
#define __msa_nlzc_h __builtin_msa_nlzc_h
#define __msa_nlzc_w __builtin_msa_nlzc_w
#define __msa_nlzc_d __builtin_msa_nlzc_d
#define __msa_copy_s_b __builtin_msa_copy_s_b
#define __msa_copy_s_h __builtin_msa_copy_s_h
#define __msa_copy_s_w __builtin_msa_copy_s_w
#define __msa_copy_s_d __builtin_msa_copy_s_d
#define __msa_copy_u_b __builtin_msa_copy_u_b
#define __msa_copy_u_h __builtin_msa_copy_u_h
#define __msa_copy_u_w __builtin_msa_copy_u_w
#define __msa_copy_u_d __builtin_msa_copy_u_d
#define __msa_insert_b __builtin_msa_insert_b
#define __msa_insert_h __builtin_msa_insert_h
#define __msa_insert_w __builtin_msa_insert_w
#define __msa_insert_d __builtin_msa_insert_d
#define __msa_insve_b __builtin_msa_insve_b
#define __msa_insve_h __builtin_msa_insve_h
#define __msa_insve_w __builtin_msa_insve_w
#define __msa_insve_d __builtin_msa_insve_d
#define __msa_test_bnz_b __builtin_msa_bnz_b
````
- **L421 EN**: Defines macro `__msa_nlzc_h` for conditional compilation, shorthand, or API generation.
  **L421 CN**: 定义宏 `__msa_nlzc_h`，用于条件编译、简写或 API 生成。
- **L422 EN**: Defines macro `__msa_nlzc_w` for conditional compilation, shorthand, or API generation.
  **L422 CN**: 定义宏 `__msa_nlzc_w`，用于条件编译、简写或 API 生成。
- **L423 EN**: Defines macro `__msa_nlzc_d` for conditional compilation, shorthand, or API generation.
  **L423 CN**: 定义宏 `__msa_nlzc_d`，用于条件编译、简写或 API 生成。
- **L424 EN**: Defines macro `__msa_copy_s_b` for conditional compilation, shorthand, or API generation.
  **L424 CN**: 定义宏 `__msa_copy_s_b`，用于条件编译、简写或 API 生成。
- **L425 EN**: Defines macro `__msa_copy_s_h` for conditional compilation, shorthand, or API generation.
  **L425 CN**: 定义宏 `__msa_copy_s_h`，用于条件编译、简写或 API 生成。
- **L426 EN**: Defines macro `__msa_copy_s_w` for conditional compilation, shorthand, or API generation.
  **L426 CN**: 定义宏 `__msa_copy_s_w`，用于条件编译、简写或 API 生成。
- **L427 EN**: Defines macro `__msa_copy_s_d` for conditional compilation, shorthand, or API generation.
  **L427 CN**: 定义宏 `__msa_copy_s_d`，用于条件编译、简写或 API 生成。
- **L428 EN**: Defines macro `__msa_copy_u_b` for conditional compilation, shorthand, or API generation.
  **L428 CN**: 定义宏 `__msa_copy_u_b`，用于条件编译、简写或 API 生成。
- **L429 EN**: Defines macro `__msa_copy_u_h` for conditional compilation, shorthand, or API generation.
  **L429 CN**: 定义宏 `__msa_copy_u_h`，用于条件编译、简写或 API 生成。
- **L430 EN**: Defines macro `__msa_copy_u_w` for conditional compilation, shorthand, or API generation.
  **L430 CN**: 定义宏 `__msa_copy_u_w`，用于条件编译、简写或 API 生成。
- **L431 EN**: Defines macro `__msa_copy_u_d` for conditional compilation, shorthand, or API generation.
  **L431 CN**: 定义宏 `__msa_copy_u_d`，用于条件编译、简写或 API 生成。
- **L432 EN**: Defines macro `__msa_insert_b` for conditional compilation, shorthand, or API generation.
  **L432 CN**: 定义宏 `__msa_insert_b`，用于条件编译、简写或 API 生成。
- **L433 EN**: Defines macro `__msa_insert_h` for conditional compilation, shorthand, or API generation.
  **L433 CN**: 定义宏 `__msa_insert_h`，用于条件编译、简写或 API 生成。
- **L434 EN**: Defines macro `__msa_insert_w` for conditional compilation, shorthand, or API generation.
  **L434 CN**: 定义宏 `__msa_insert_w`，用于条件编译、简写或 API 生成。
- **L435 EN**: Defines macro `__msa_insert_d` for conditional compilation, shorthand, or API generation.
  **L435 CN**: 定义宏 `__msa_insert_d`，用于条件编译、简写或 API 生成。
- **L436 EN**: Defines macro `__msa_insve_b` for conditional compilation, shorthand, or API generation.
  **L436 CN**: 定义宏 `__msa_insve_b`，用于条件编译、简写或 API 生成。
- **L437 EN**: Defines macro `__msa_insve_h` for conditional compilation, shorthand, or API generation.
  **L437 CN**: 定义宏 `__msa_insve_h`，用于条件编译、简写或 API 生成。
- **L438 EN**: Defines macro `__msa_insve_w` for conditional compilation, shorthand, or API generation.
  **L438 CN**: 定义宏 `__msa_insve_w`，用于条件编译、简写或 API 生成。
- **L439 EN**: Defines macro `__msa_insve_d` for conditional compilation, shorthand, or API generation.
  **L439 CN**: 定义宏 `__msa_insve_d`，用于条件编译、简写或 API 生成。
- **L440 EN**: Defines macro `__msa_test_bnz_b` for conditional compilation, shorthand, or API generation.
  **L440 CN**: 定义宏 `__msa_test_bnz_b`，用于条件编译、简写或 API 生成。

### Lines 441-460

````c
#define __msa_test_bnz_h __builtin_msa_bnz_h
#define __msa_test_bnz_w __builtin_msa_bnz_w
#define __msa_test_bnz_d __builtin_msa_bnz_d
#define __msa_test_bz_b __builtin_msa_bz_b
#define __msa_test_bz_h __builtin_msa_bz_h
#define __msa_test_bz_w __builtin_msa_bz_w
#define __msa_test_bz_d __builtin_msa_bz_d
#define __msa_ldi_b __builtin_msa_ldi_b
#define __msa_ldi_h __builtin_msa_ldi_h
#define __msa_ldi_w __builtin_msa_ldi_w
#define __msa_ldi_d __builtin_msa_ldi_d
#define __msa_fcaf_w __builtin_msa_fcaf_w
#define __msa_fcaf_d __builtin_msa_fcaf_d
#define __msa_fcor_w __builtin_msa_fcor_w
#define __msa_fcor_d __builtin_msa_fcor_d
#define __msa_fcun_w __builtin_msa_fcun_w
#define __msa_fcun_d __builtin_msa_fcun_d
#define __msa_fcune_w __builtin_msa_fcune_w
#define __msa_fcune_d __builtin_msa_fcune_d
#define __msa_fcueq_w __builtin_msa_fcueq_w
````
- **L441 EN**: Defines macro `__msa_test_bnz_h` for conditional compilation, shorthand, or API generation.
  **L441 CN**: 定义宏 `__msa_test_bnz_h`，用于条件编译、简写或 API 生成。
- **L442 EN**: Defines macro `__msa_test_bnz_w` for conditional compilation, shorthand, or API generation.
  **L442 CN**: 定义宏 `__msa_test_bnz_w`，用于条件编译、简写或 API 生成。
- **L443 EN**: Defines macro `__msa_test_bnz_d` for conditional compilation, shorthand, or API generation.
  **L443 CN**: 定义宏 `__msa_test_bnz_d`，用于条件编译、简写或 API 生成。
- **L444 EN**: Defines macro `__msa_test_bz_b` for conditional compilation, shorthand, or API generation.
  **L444 CN**: 定义宏 `__msa_test_bz_b`，用于条件编译、简写或 API 生成。
- **L445 EN**: Defines macro `__msa_test_bz_h` for conditional compilation, shorthand, or API generation.
  **L445 CN**: 定义宏 `__msa_test_bz_h`，用于条件编译、简写或 API 生成。
- **L446 EN**: Defines macro `__msa_test_bz_w` for conditional compilation, shorthand, or API generation.
  **L446 CN**: 定义宏 `__msa_test_bz_w`，用于条件编译、简写或 API 生成。
- **L447 EN**: Defines macro `__msa_test_bz_d` for conditional compilation, shorthand, or API generation.
  **L447 CN**: 定义宏 `__msa_test_bz_d`，用于条件编译、简写或 API 生成。
- **L448 EN**: Defines macro `__msa_ldi_b` for conditional compilation, shorthand, or API generation.
  **L448 CN**: 定义宏 `__msa_ldi_b`，用于条件编译、简写或 API 生成。
- **L449 EN**: Defines macro `__msa_ldi_h` for conditional compilation, shorthand, or API generation.
  **L449 CN**: 定义宏 `__msa_ldi_h`，用于条件编译、简写或 API 生成。
- **L450 EN**: Defines macro `__msa_ldi_w` for conditional compilation, shorthand, or API generation.
  **L450 CN**: 定义宏 `__msa_ldi_w`，用于条件编译、简写或 API 生成。
- **L451 EN**: Defines macro `__msa_ldi_d` for conditional compilation, shorthand, or API generation.
  **L451 CN**: 定义宏 `__msa_ldi_d`，用于条件编译、简写或 API 生成。
- **L452 EN**: Defines macro `__msa_fcaf_w` for conditional compilation, shorthand, or API generation.
  **L452 CN**: 定义宏 `__msa_fcaf_w`，用于条件编译、简写或 API 生成。
- **L453 EN**: Defines macro `__msa_fcaf_d` for conditional compilation, shorthand, or API generation.
  **L453 CN**: 定义宏 `__msa_fcaf_d`，用于条件编译、简写或 API 生成。
- **L454 EN**: Defines macro `__msa_fcor_w` for conditional compilation, shorthand, or API generation.
  **L454 CN**: 定义宏 `__msa_fcor_w`，用于条件编译、简写或 API 生成。
- **L455 EN**: Defines macro `__msa_fcor_d` for conditional compilation, shorthand, or API generation.
  **L455 CN**: 定义宏 `__msa_fcor_d`，用于条件编译、简写或 API 生成。
- **L456 EN**: Defines macro `__msa_fcun_w` for conditional compilation, shorthand, or API generation.
  **L456 CN**: 定义宏 `__msa_fcun_w`，用于条件编译、简写或 API 生成。
- **L457 EN**: Defines macro `__msa_fcun_d` for conditional compilation, shorthand, or API generation.
  **L457 CN**: 定义宏 `__msa_fcun_d`，用于条件编译、简写或 API 生成。
- **L458 EN**: Defines macro `__msa_fcune_w` for conditional compilation, shorthand, or API generation.
  **L458 CN**: 定义宏 `__msa_fcune_w`，用于条件编译、简写或 API 生成。
- **L459 EN**: Defines macro `__msa_fcune_d` for conditional compilation, shorthand, or API generation.
  **L459 CN**: 定义宏 `__msa_fcune_d`，用于条件编译、简写或 API 生成。
- **L460 EN**: Defines macro `__msa_fcueq_w` for conditional compilation, shorthand, or API generation.
  **L460 CN**: 定义宏 `__msa_fcueq_w`，用于条件编译、简写或 API 生成。

### Lines 461-480

````c
#define __msa_fcueq_d __builtin_msa_fcueq_d
#define __msa_fceq_w __builtin_msa_fceq_w
#define __msa_fceq_d __builtin_msa_fceq_d
#define __msa_fcne_w __builtin_msa_fcne_w
#define __msa_fcne_d __builtin_msa_fcne_d
#define __msa_fclt_w __builtin_msa_fclt_w
#define __msa_fclt_d __builtin_msa_fclt_d
#define __msa_fcult_w __builtin_msa_fcult_w
#define __msa_fcult_d __builtin_msa_fcult_d
#define __msa_fcle_w __builtin_msa_fcle_w
#define __msa_fcle_d __builtin_msa_fcle_d
#define __msa_fcule_w __builtin_msa_fcule_w
#define __msa_fcule_d __builtin_msa_fcule_d
#define __msa_fsaf_w __builtin_msa_fsaf_w
#define __msa_fsaf_d __builtin_msa_fsaf_d
#define __msa_fsor_w __builtin_msa_fsor_w
#define __msa_fsor_d __builtin_msa_fsor_d
#define __msa_fsun_w __builtin_msa_fsun_w
#define __msa_fsun_d __builtin_msa_fsun_d
#define __msa_fsune_w __builtin_msa_fsune_w
````
- **L461 EN**: Defines macro `__msa_fcueq_d` for conditional compilation, shorthand, or API generation.
  **L461 CN**: 定义宏 `__msa_fcueq_d`，用于条件编译、简写或 API 生成。
- **L462 EN**: Defines macro `__msa_fceq_w` for conditional compilation, shorthand, or API generation.
  **L462 CN**: 定义宏 `__msa_fceq_w`，用于条件编译、简写或 API 生成。
- **L463 EN**: Defines macro `__msa_fceq_d` for conditional compilation, shorthand, or API generation.
  **L463 CN**: 定义宏 `__msa_fceq_d`，用于条件编译、简写或 API 生成。
- **L464 EN**: Defines macro `__msa_fcne_w` for conditional compilation, shorthand, or API generation.
  **L464 CN**: 定义宏 `__msa_fcne_w`，用于条件编译、简写或 API 生成。
- **L465 EN**: Defines macro `__msa_fcne_d` for conditional compilation, shorthand, or API generation.
  **L465 CN**: 定义宏 `__msa_fcne_d`，用于条件编译、简写或 API 生成。
- **L466 EN**: Defines macro `__msa_fclt_w` for conditional compilation, shorthand, or API generation.
  **L466 CN**: 定义宏 `__msa_fclt_w`，用于条件编译、简写或 API 生成。
- **L467 EN**: Defines macro `__msa_fclt_d` for conditional compilation, shorthand, or API generation.
  **L467 CN**: 定义宏 `__msa_fclt_d`，用于条件编译、简写或 API 生成。
- **L468 EN**: Defines macro `__msa_fcult_w` for conditional compilation, shorthand, or API generation.
  **L468 CN**: 定义宏 `__msa_fcult_w`，用于条件编译、简写或 API 生成。
- **L469 EN**: Defines macro `__msa_fcult_d` for conditional compilation, shorthand, or API generation.
  **L469 CN**: 定义宏 `__msa_fcult_d`，用于条件编译、简写或 API 生成。
- **L470 EN**: Defines macro `__msa_fcle_w` for conditional compilation, shorthand, or API generation.
  **L470 CN**: 定义宏 `__msa_fcle_w`，用于条件编译、简写或 API 生成。
- **L471 EN**: Defines macro `__msa_fcle_d` for conditional compilation, shorthand, or API generation.
  **L471 CN**: 定义宏 `__msa_fcle_d`，用于条件编译、简写或 API 生成。
- **L472 EN**: Defines macro `__msa_fcule_w` for conditional compilation, shorthand, or API generation.
  **L472 CN**: 定义宏 `__msa_fcule_w`，用于条件编译、简写或 API 生成。
- **L473 EN**: Defines macro `__msa_fcule_d` for conditional compilation, shorthand, or API generation.
  **L473 CN**: 定义宏 `__msa_fcule_d`，用于条件编译、简写或 API 生成。
- **L474 EN**: Defines macro `__msa_fsaf_w` for conditional compilation, shorthand, or API generation.
  **L474 CN**: 定义宏 `__msa_fsaf_w`，用于条件编译、简写或 API 生成。
- **L475 EN**: Defines macro `__msa_fsaf_d` for conditional compilation, shorthand, or API generation.
  **L475 CN**: 定义宏 `__msa_fsaf_d`，用于条件编译、简写或 API 生成。
- **L476 EN**: Defines macro `__msa_fsor_w` for conditional compilation, shorthand, or API generation.
  **L476 CN**: 定义宏 `__msa_fsor_w`，用于条件编译、简写或 API 生成。
- **L477 EN**: Defines macro `__msa_fsor_d` for conditional compilation, shorthand, or API generation.
  **L477 CN**: 定义宏 `__msa_fsor_d`，用于条件编译、简写或 API 生成。
- **L478 EN**: Defines macro `__msa_fsun_w` for conditional compilation, shorthand, or API generation.
  **L478 CN**: 定义宏 `__msa_fsun_w`，用于条件编译、简写或 API 生成。
- **L479 EN**: Defines macro `__msa_fsun_d` for conditional compilation, shorthand, or API generation.
  **L479 CN**: 定义宏 `__msa_fsun_d`，用于条件编译、简写或 API 生成。
- **L480 EN**: Defines macro `__msa_fsune_w` for conditional compilation, shorthand, or API generation.
  **L480 CN**: 定义宏 `__msa_fsune_w`，用于条件编译、简写或 API 生成。

### Lines 481-500

````c
#define __msa_fsune_d __builtin_msa_fsune_d
#define __msa_fsueq_w __builtin_msa_fsueq_w
#define __msa_fsueq_d __builtin_msa_fsueq_d
#define __msa_fseq_w __builtin_msa_fseq_w
#define __msa_fseq_d __builtin_msa_fseq_d
#define __msa_fsne_w __builtin_msa_fsne_w
#define __msa_fsne_d __builtin_msa_fsne_d
#define __msa_fslt_w __builtin_msa_fslt_w
#define __msa_fslt_d __builtin_msa_fslt_d
#define __msa_fsult_w __builtin_msa_fsult_w
#define __msa_fsult_d __builtin_msa_fsult_d
#define __msa_fsle_w __builtin_msa_fsle_w
#define __msa_fsle_d __builtin_msa_fsle_d
#define __msa_fsule_w __builtin_msa_fsule_w
#define __msa_fsule_d __builtin_msa_fsule_d
#define __msa_fadd_w __builtin_msa_fadd_w
#define __msa_fadd_d __builtin_msa_fadd_d
#define __msa_fsub_w __builtin_msa_fsub_w
#define __msa_fsub_d __builtin_msa_fsub_d
#define __msa_fmul_w __builtin_msa_fmul_w
````
- **L481 EN**: Defines macro `__msa_fsune_d` for conditional compilation, shorthand, or API generation.
  **L481 CN**: 定义宏 `__msa_fsune_d`，用于条件编译、简写或 API 生成。
- **L482 EN**: Defines macro `__msa_fsueq_w` for conditional compilation, shorthand, or API generation.
  **L482 CN**: 定义宏 `__msa_fsueq_w`，用于条件编译、简写或 API 生成。
- **L483 EN**: Defines macro `__msa_fsueq_d` for conditional compilation, shorthand, or API generation.
  **L483 CN**: 定义宏 `__msa_fsueq_d`，用于条件编译、简写或 API 生成。
- **L484 EN**: Defines macro `__msa_fseq_w` for conditional compilation, shorthand, or API generation.
  **L484 CN**: 定义宏 `__msa_fseq_w`，用于条件编译、简写或 API 生成。
- **L485 EN**: Defines macro `__msa_fseq_d` for conditional compilation, shorthand, or API generation.
  **L485 CN**: 定义宏 `__msa_fseq_d`，用于条件编译、简写或 API 生成。
- **L486 EN**: Defines macro `__msa_fsne_w` for conditional compilation, shorthand, or API generation.
  **L486 CN**: 定义宏 `__msa_fsne_w`，用于条件编译、简写或 API 生成。
- **L487 EN**: Defines macro `__msa_fsne_d` for conditional compilation, shorthand, or API generation.
  **L487 CN**: 定义宏 `__msa_fsne_d`，用于条件编译、简写或 API 生成。
- **L488 EN**: Defines macro `__msa_fslt_w` for conditional compilation, shorthand, or API generation.
  **L488 CN**: 定义宏 `__msa_fslt_w`，用于条件编译、简写或 API 生成。
- **L489 EN**: Defines macro `__msa_fslt_d` for conditional compilation, shorthand, or API generation.
  **L489 CN**: 定义宏 `__msa_fslt_d`，用于条件编译、简写或 API 生成。
- **L490 EN**: Defines macro `__msa_fsult_w` for conditional compilation, shorthand, or API generation.
  **L490 CN**: 定义宏 `__msa_fsult_w`，用于条件编译、简写或 API 生成。
- **L491 EN**: Defines macro `__msa_fsult_d` for conditional compilation, shorthand, or API generation.
  **L491 CN**: 定义宏 `__msa_fsult_d`，用于条件编译、简写或 API 生成。
- **L492 EN**: Defines macro `__msa_fsle_w` for conditional compilation, shorthand, or API generation.
  **L492 CN**: 定义宏 `__msa_fsle_w`，用于条件编译、简写或 API 生成。
- **L493 EN**: Defines macro `__msa_fsle_d` for conditional compilation, shorthand, or API generation.
  **L493 CN**: 定义宏 `__msa_fsle_d`，用于条件编译、简写或 API 生成。
- **L494 EN**: Defines macro `__msa_fsule_w` for conditional compilation, shorthand, or API generation.
  **L494 CN**: 定义宏 `__msa_fsule_w`，用于条件编译、简写或 API 生成。
- **L495 EN**: Defines macro `__msa_fsule_d` for conditional compilation, shorthand, or API generation.
  **L495 CN**: 定义宏 `__msa_fsule_d`，用于条件编译、简写或 API 生成。
- **L496 EN**: Defines macro `__msa_fadd_w` for conditional compilation, shorthand, or API generation.
  **L496 CN**: 定义宏 `__msa_fadd_w`，用于条件编译、简写或 API 生成。
- **L497 EN**: Defines macro `__msa_fadd_d` for conditional compilation, shorthand, or API generation.
  **L497 CN**: 定义宏 `__msa_fadd_d`，用于条件编译、简写或 API 生成。
- **L498 EN**: Defines macro `__msa_fsub_w` for conditional compilation, shorthand, or API generation.
  **L498 CN**: 定义宏 `__msa_fsub_w`，用于条件编译、简写或 API 生成。
- **L499 EN**: Defines macro `__msa_fsub_d` for conditional compilation, shorthand, or API generation.
  **L499 CN**: 定义宏 `__msa_fsub_d`，用于条件编译、简写或 API 生成。
- **L500 EN**: Defines macro `__msa_fmul_w` for conditional compilation, shorthand, or API generation.
  **L500 CN**: 定义宏 `__msa_fmul_w`，用于条件编译、简写或 API 生成。

### Lines 501-520

````c
#define __msa_fmul_d __builtin_msa_fmul_d
#define __msa_fdiv_w __builtin_msa_fdiv_w
#define __msa_fdiv_d __builtin_msa_fdiv_d
#define __msa_fmadd_w __builtin_msa_fmadd_w
#define __msa_fmadd_d __builtin_msa_fmadd_d
#define __msa_fmsub_w __builtin_msa_fmsub_w
#define __msa_fmsub_d __builtin_msa_fmsub_d
#define __msa_fexp2_w __builtin_msa_fexp2_w
#define __msa_fexp2_d __builtin_msa_fexp2_d
#define __msa_fexdo_h __builtin_msa_fexdo_h
#define __msa_fexdo_w __builtin_msa_fexdo_w
#define __msa_ftq_h __builtin_msa_ftq_h
#define __msa_ftq_w __builtin_msa_ftq_w
#define __msa_fmin_w __builtin_msa_fmin_w
#define __msa_fmin_d __builtin_msa_fmin_d
#define __msa_fmin_a_w __builtin_msa_fmin_a_w
#define __msa_fmin_a_d __builtin_msa_fmin_a_d
#define __msa_fmax_w __builtin_msa_fmax_w
#define __msa_fmax_d __builtin_msa_fmax_d
#define __msa_fmax_a_w __builtin_msa_fmax_a_w
````
- **L501 EN**: Defines macro `__msa_fmul_d` for conditional compilation, shorthand, or API generation.
  **L501 CN**: 定义宏 `__msa_fmul_d`，用于条件编译、简写或 API 生成。
- **L502 EN**: Defines macro `__msa_fdiv_w` for conditional compilation, shorthand, or API generation.
  **L502 CN**: 定义宏 `__msa_fdiv_w`，用于条件编译、简写或 API 生成。
- **L503 EN**: Defines macro `__msa_fdiv_d` for conditional compilation, shorthand, or API generation.
  **L503 CN**: 定义宏 `__msa_fdiv_d`，用于条件编译、简写或 API 生成。
- **L504 EN**: Defines macro `__msa_fmadd_w` for conditional compilation, shorthand, or API generation.
  **L504 CN**: 定义宏 `__msa_fmadd_w`，用于条件编译、简写或 API 生成。
- **L505 EN**: Defines macro `__msa_fmadd_d` for conditional compilation, shorthand, or API generation.
  **L505 CN**: 定义宏 `__msa_fmadd_d`，用于条件编译、简写或 API 生成。
- **L506 EN**: Defines macro `__msa_fmsub_w` for conditional compilation, shorthand, or API generation.
  **L506 CN**: 定义宏 `__msa_fmsub_w`，用于条件编译、简写或 API 生成。
- **L507 EN**: Defines macro `__msa_fmsub_d` for conditional compilation, shorthand, or API generation.
  **L507 CN**: 定义宏 `__msa_fmsub_d`，用于条件编译、简写或 API 生成。
- **L508 EN**: Defines macro `__msa_fexp2_w` for conditional compilation, shorthand, or API generation.
  **L508 CN**: 定义宏 `__msa_fexp2_w`，用于条件编译、简写或 API 生成。
- **L509 EN**: Defines macro `__msa_fexp2_d` for conditional compilation, shorthand, or API generation.
  **L509 CN**: 定义宏 `__msa_fexp2_d`，用于条件编译、简写或 API 生成。
- **L510 EN**: Defines macro `__msa_fexdo_h` for conditional compilation, shorthand, or API generation.
  **L510 CN**: 定义宏 `__msa_fexdo_h`，用于条件编译、简写或 API 生成。
- **L511 EN**: Defines macro `__msa_fexdo_w` for conditional compilation, shorthand, or API generation.
  **L511 CN**: 定义宏 `__msa_fexdo_w`，用于条件编译、简写或 API 生成。
- **L512 EN**: Defines macro `__msa_ftq_h` for conditional compilation, shorthand, or API generation.
  **L512 CN**: 定义宏 `__msa_ftq_h`，用于条件编译、简写或 API 生成。
- **L513 EN**: Defines macro `__msa_ftq_w` for conditional compilation, shorthand, or API generation.
  **L513 CN**: 定义宏 `__msa_ftq_w`，用于条件编译、简写或 API 生成。
- **L514 EN**: Defines macro `__msa_fmin_w` for conditional compilation, shorthand, or API generation.
  **L514 CN**: 定义宏 `__msa_fmin_w`，用于条件编译、简写或 API 生成。
- **L515 EN**: Defines macro `__msa_fmin_d` for conditional compilation, shorthand, or API generation.
  **L515 CN**: 定义宏 `__msa_fmin_d`，用于条件编译、简写或 API 生成。
- **L516 EN**: Defines macro `__msa_fmin_a_w` for conditional compilation, shorthand, or API generation.
  **L516 CN**: 定义宏 `__msa_fmin_a_w`，用于条件编译、简写或 API 生成。
- **L517 EN**: Defines macro `__msa_fmin_a_d` for conditional compilation, shorthand, or API generation.
  **L517 CN**: 定义宏 `__msa_fmin_a_d`，用于条件编译、简写或 API 生成。
- **L518 EN**: Defines macro `__msa_fmax_w` for conditional compilation, shorthand, or API generation.
  **L518 CN**: 定义宏 `__msa_fmax_w`，用于条件编译、简写或 API 生成。
- **L519 EN**: Defines macro `__msa_fmax_d` for conditional compilation, shorthand, or API generation.
  **L519 CN**: 定义宏 `__msa_fmax_d`，用于条件编译、简写或 API 生成。
- **L520 EN**: Defines macro `__msa_fmax_a_w` for conditional compilation, shorthand, or API generation.
  **L520 CN**: 定义宏 `__msa_fmax_a_w`，用于条件编译、简写或 API 生成。

### Lines 521-540

````c
#define __msa_fmax_a_d __builtin_msa_fmax_a_d
#define __msa_mul_q_h __builtin_msa_mul_q_h
#define __msa_mul_q_w __builtin_msa_mul_q_w
#define __msa_mulr_q_h __builtin_msa_mulr_q_h
#define __msa_mulr_q_w __builtin_msa_mulr_q_w
#define __msa_madd_q_h __builtin_msa_madd_q_h
#define __msa_madd_q_w __builtin_msa_madd_q_w
#define __msa_maddr_q_h __builtin_msa_maddr_q_h
#define __msa_maddr_q_w __builtin_msa_maddr_q_w
#define __msa_msub_q_h __builtin_msa_msub_q_h
#define __msa_msub_q_w __builtin_msa_msub_q_w
#define __msa_msubr_q_h __builtin_msa_msubr_q_h
#define __msa_msubr_q_w __builtin_msa_msubr_q_w
#define __msa_fclass_w __builtin_msa_fclass_w
#define __msa_fclass_d __builtin_msa_fclass_d
#define __msa_fsqrt_w __builtin_msa_fsqrt_w
#define __msa_fsqrt_d __builtin_msa_fsqrt_d
#define __msa_frcp_w __builtin_msa_frcp_w
#define __msa_frcp_d __builtin_msa_frcp_d
#define __msa_frint_w __builtin_msa_frint_w
````
- **L521 EN**: Defines macro `__msa_fmax_a_d` for conditional compilation, shorthand, or API generation.
  **L521 CN**: 定义宏 `__msa_fmax_a_d`，用于条件编译、简写或 API 生成。
- **L522 EN**: Defines macro `__msa_mul_q_h` for conditional compilation, shorthand, or API generation.
  **L522 CN**: 定义宏 `__msa_mul_q_h`，用于条件编译、简写或 API 生成。
- **L523 EN**: Defines macro `__msa_mul_q_w` for conditional compilation, shorthand, or API generation.
  **L523 CN**: 定义宏 `__msa_mul_q_w`，用于条件编译、简写或 API 生成。
- **L524 EN**: Defines macro `__msa_mulr_q_h` for conditional compilation, shorthand, or API generation.
  **L524 CN**: 定义宏 `__msa_mulr_q_h`，用于条件编译、简写或 API 生成。
- **L525 EN**: Defines macro `__msa_mulr_q_w` for conditional compilation, shorthand, or API generation.
  **L525 CN**: 定义宏 `__msa_mulr_q_w`，用于条件编译、简写或 API 生成。
- **L526 EN**: Defines macro `__msa_madd_q_h` for conditional compilation, shorthand, or API generation.
  **L526 CN**: 定义宏 `__msa_madd_q_h`，用于条件编译、简写或 API 生成。
- **L527 EN**: Defines macro `__msa_madd_q_w` for conditional compilation, shorthand, or API generation.
  **L527 CN**: 定义宏 `__msa_madd_q_w`，用于条件编译、简写或 API 生成。
- **L528 EN**: Defines macro `__msa_maddr_q_h` for conditional compilation, shorthand, or API generation.
  **L528 CN**: 定义宏 `__msa_maddr_q_h`，用于条件编译、简写或 API 生成。
- **L529 EN**: Defines macro `__msa_maddr_q_w` for conditional compilation, shorthand, or API generation.
  **L529 CN**: 定义宏 `__msa_maddr_q_w`，用于条件编译、简写或 API 生成。
- **L530 EN**: Defines macro `__msa_msub_q_h` for conditional compilation, shorthand, or API generation.
  **L530 CN**: 定义宏 `__msa_msub_q_h`，用于条件编译、简写或 API 生成。
- **L531 EN**: Defines macro `__msa_msub_q_w` for conditional compilation, shorthand, or API generation.
  **L531 CN**: 定义宏 `__msa_msub_q_w`，用于条件编译、简写或 API 生成。
- **L532 EN**: Defines macro `__msa_msubr_q_h` for conditional compilation, shorthand, or API generation.
  **L532 CN**: 定义宏 `__msa_msubr_q_h`，用于条件编译、简写或 API 生成。
- **L533 EN**: Defines macro `__msa_msubr_q_w` for conditional compilation, shorthand, or API generation.
  **L533 CN**: 定义宏 `__msa_msubr_q_w`，用于条件编译、简写或 API 生成。
- **L534 EN**: Defines macro `__msa_fclass_w` for conditional compilation, shorthand, or API generation.
  **L534 CN**: 定义宏 `__msa_fclass_w`，用于条件编译、简写或 API 生成。
- **L535 EN**: Defines macro `__msa_fclass_d` for conditional compilation, shorthand, or API generation.
  **L535 CN**: 定义宏 `__msa_fclass_d`，用于条件编译、简写或 API 生成。
- **L536 EN**: Defines macro `__msa_fsqrt_w` for conditional compilation, shorthand, or API generation.
  **L536 CN**: 定义宏 `__msa_fsqrt_w`，用于条件编译、简写或 API 生成。
- **L537 EN**: Defines macro `__msa_fsqrt_d` for conditional compilation, shorthand, or API generation.
  **L537 CN**: 定义宏 `__msa_fsqrt_d`，用于条件编译、简写或 API 生成。
- **L538 EN**: Defines macro `__msa_frcp_w` for conditional compilation, shorthand, or API generation.
  **L538 CN**: 定义宏 `__msa_frcp_w`，用于条件编译、简写或 API 生成。
- **L539 EN**: Defines macro `__msa_frcp_d` for conditional compilation, shorthand, or API generation.
  **L539 CN**: 定义宏 `__msa_frcp_d`，用于条件编译、简写或 API 生成。
- **L540 EN**: Defines macro `__msa_frint_w` for conditional compilation, shorthand, or API generation.
  **L540 CN**: 定义宏 `__msa_frint_w`，用于条件编译、简写或 API 生成。

### Lines 541-560

````c
#define __msa_frint_d __builtin_msa_frint_d
#define __msa_frsqrt_w __builtin_msa_frsqrt_w
#define __msa_frsqrt_d __builtin_msa_frsqrt_d
#define __msa_flog2_w __builtin_msa_flog2_w
#define __msa_flog2_d __builtin_msa_flog2_d
#define __msa_fexupl_w __builtin_msa_fexupl_w
#define __msa_fexupl_d __builtin_msa_fexupl_d
#define __msa_fexupr_w __builtin_msa_fexupr_w
#define __msa_fexupr_d __builtin_msa_fexupr_d
#define __msa_ffql_w __builtin_msa_ffql_w
#define __msa_ffql_d __builtin_msa_ffql_d
#define __msa_ffqr_w __builtin_msa_ffqr_w
#define __msa_ffqr_d __builtin_msa_ffqr_d
#define __msa_ftint_s_w __builtin_msa_ftint_s_w
#define __msa_ftint_s_d __builtin_msa_ftint_s_d
#define __msa_ftint_u_w __builtin_msa_ftint_u_w
#define __msa_ftint_u_d __builtin_msa_ftint_u_d
#define __msa_ftrunc_s_w __builtin_msa_ftrunc_s_w
#define __msa_ftrunc_s_d __builtin_msa_ftrunc_s_d
#define __msa_ftrunc_u_w __builtin_msa_ftrunc_u_w
````
- **L541 EN**: Defines macro `__msa_frint_d` for conditional compilation, shorthand, or API generation.
  **L541 CN**: 定义宏 `__msa_frint_d`，用于条件编译、简写或 API 生成。
- **L542 EN**: Defines macro `__msa_frsqrt_w` for conditional compilation, shorthand, or API generation.
  **L542 CN**: 定义宏 `__msa_frsqrt_w`，用于条件编译、简写或 API 生成。
- **L543 EN**: Defines macro `__msa_frsqrt_d` for conditional compilation, shorthand, or API generation.
  **L543 CN**: 定义宏 `__msa_frsqrt_d`，用于条件编译、简写或 API 生成。
- **L544 EN**: Defines macro `__msa_flog2_w` for conditional compilation, shorthand, or API generation.
  **L544 CN**: 定义宏 `__msa_flog2_w`，用于条件编译、简写或 API 生成。
- **L545 EN**: Defines macro `__msa_flog2_d` for conditional compilation, shorthand, or API generation.
  **L545 CN**: 定义宏 `__msa_flog2_d`，用于条件编译、简写或 API 生成。
- **L546 EN**: Defines macro `__msa_fexupl_w` for conditional compilation, shorthand, or API generation.
  **L546 CN**: 定义宏 `__msa_fexupl_w`，用于条件编译、简写或 API 生成。
- **L547 EN**: Defines macro `__msa_fexupl_d` for conditional compilation, shorthand, or API generation.
  **L547 CN**: 定义宏 `__msa_fexupl_d`，用于条件编译、简写或 API 生成。
- **L548 EN**: Defines macro `__msa_fexupr_w` for conditional compilation, shorthand, or API generation.
  **L548 CN**: 定义宏 `__msa_fexupr_w`，用于条件编译、简写或 API 生成。
- **L549 EN**: Defines macro `__msa_fexupr_d` for conditional compilation, shorthand, or API generation.
  **L549 CN**: 定义宏 `__msa_fexupr_d`，用于条件编译、简写或 API 生成。
- **L550 EN**: Defines macro `__msa_ffql_w` for conditional compilation, shorthand, or API generation.
  **L550 CN**: 定义宏 `__msa_ffql_w`，用于条件编译、简写或 API 生成。
- **L551 EN**: Defines macro `__msa_ffql_d` for conditional compilation, shorthand, or API generation.
  **L551 CN**: 定义宏 `__msa_ffql_d`，用于条件编译、简写或 API 生成。
- **L552 EN**: Defines macro `__msa_ffqr_w` for conditional compilation, shorthand, or API generation.
  **L552 CN**: 定义宏 `__msa_ffqr_w`，用于条件编译、简写或 API 生成。
- **L553 EN**: Defines macro `__msa_ffqr_d` for conditional compilation, shorthand, or API generation.
  **L553 CN**: 定义宏 `__msa_ffqr_d`，用于条件编译、简写或 API 生成。
- **L554 EN**: Defines macro `__msa_ftint_s_w` for conditional compilation, shorthand, or API generation.
  **L554 CN**: 定义宏 `__msa_ftint_s_w`，用于条件编译、简写或 API 生成。
- **L555 EN**: Defines macro `__msa_ftint_s_d` for conditional compilation, shorthand, or API generation.
  **L555 CN**: 定义宏 `__msa_ftint_s_d`，用于条件编译、简写或 API 生成。
- **L556 EN**: Defines macro `__msa_ftint_u_w` for conditional compilation, shorthand, or API generation.
  **L556 CN**: 定义宏 `__msa_ftint_u_w`，用于条件编译、简写或 API 生成。
- **L557 EN**: Defines macro `__msa_ftint_u_d` for conditional compilation, shorthand, or API generation.
  **L557 CN**: 定义宏 `__msa_ftint_u_d`，用于条件编译、简写或 API 生成。
- **L558 EN**: Defines macro `__msa_ftrunc_s_w` for conditional compilation, shorthand, or API generation.
  **L558 CN**: 定义宏 `__msa_ftrunc_s_w`，用于条件编译、简写或 API 生成。
- **L559 EN**: Defines macro `__msa_ftrunc_s_d` for conditional compilation, shorthand, or API generation.
  **L559 CN**: 定义宏 `__msa_ftrunc_s_d`，用于条件编译、简写或 API 生成。
- **L560 EN**: Defines macro `__msa_ftrunc_u_w` for conditional compilation, shorthand, or API generation.
  **L560 CN**: 定义宏 `__msa_ftrunc_u_w`，用于条件编译、简写或 API 生成。

### Lines 561-573

````c
#define __msa_ftrunc_u_d __builtin_msa_ftrunc_u_d
#define __msa_ffint_s_w __builtin_msa_ffint_s_w
#define __msa_ffint_s_d __builtin_msa_ffint_s_d
#define __msa_ffint_u_w __builtin_msa_ffint_u_w
#define __msa_ffint_u_d __builtin_msa_ffint_u_d
#define __msa_cfcmsa __builtin_msa_cfcmsa
#define __msa_move_v __builtin_msa_move_v
#define __msa_cast_to_vector_float __builtin_msa_cast_to_vector_float
#define __msa_cast_to_vector_double __builtin_msa_cast_to_vector_double
#define __msa_cast_to_scalar_float __builtin_msa_cast_to_scalar_float
#define __msa_cast_to_scalar_double __builtin_msa_cast_to_scalar_double
#endif /* defined(__mips_msa) */
#endif /* _MSA_H */
````
- **L561 EN**: Defines macro `__msa_ftrunc_u_d` for conditional compilation, shorthand, or API generation.
  **L561 CN**: 定义宏 `__msa_ftrunc_u_d`，用于条件编译、简写或 API 生成。
- **L562 EN**: Defines macro `__msa_ffint_s_w` for conditional compilation, shorthand, or API generation.
  **L562 CN**: 定义宏 `__msa_ffint_s_w`，用于条件编译、简写或 API 生成。
- **L563 EN**: Defines macro `__msa_ffint_s_d` for conditional compilation, shorthand, or API generation.
  **L563 CN**: 定义宏 `__msa_ffint_s_d`，用于条件编译、简写或 API 生成。
- **L564 EN**: Defines macro `__msa_ffint_u_w` for conditional compilation, shorthand, or API generation.
  **L564 CN**: 定义宏 `__msa_ffint_u_w`，用于条件编译、简写或 API 生成。
- **L565 EN**: Defines macro `__msa_ffint_u_d` for conditional compilation, shorthand, or API generation.
  **L565 CN**: 定义宏 `__msa_ffint_u_d`，用于条件编译、简写或 API 生成。
- **L566 EN**: Defines macro `__msa_cfcmsa` for conditional compilation, shorthand, or API generation.
  **L566 CN**: 定义宏 `__msa_cfcmsa`，用于条件编译、简写或 API 生成。
- **L567 EN**: Defines macro `__msa_move_v` for conditional compilation, shorthand, or API generation.
  **L567 CN**: 定义宏 `__msa_move_v`，用于条件编译、简写或 API 生成。
- **L568 EN**: Defines macro `__msa_cast_to_vector_float` for conditional compilation, shorthand, or API generation.
  **L568 CN**: 定义宏 `__msa_cast_to_vector_float`，用于条件编译、简写或 API 生成。
- **L569 EN**: Defines macro `__msa_cast_to_vector_double` for conditional compilation, shorthand, or API generation.
  **L569 CN**: 定义宏 `__msa_cast_to_vector_double`，用于条件编译、简写或 API 生成。
- **L570 EN**: Defines macro `__msa_cast_to_scalar_float` for conditional compilation, shorthand, or API generation.
  **L570 CN**: 定义宏 `__msa_cast_to_scalar_float`，用于条件编译、简写或 API 生成。
- **L571 EN**: Defines macro `__msa_cast_to_scalar_double` for conditional compilation, shorthand, or API generation.
  **L571 CN**: 定义宏 `__msa_cast_to_scalar_double`，用于条件编译、简写或 API 生成。
- **L572 EN**: Closes the current preprocessor conditional block.
  **L572 CN**: 结束当前预处理条件块。
- **L573 EN**: Closes the current preprocessor conditional block.
  **L573 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **MIPS MSA intrinsics / MIPS MSA intrinsic 接口**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**
- **Compiler-specific attributes / 编译器特定属性**
- **Type aliasing and ABI shaping / 类型别名与 ABI 约束**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None. / 无。
- **Conditional macros / 条件宏**: `_MSA_H`, `__mips_msa`
- **External builtins / 外部 builtin**: `__builtin_msa_sll_b`, `__builtin_msa_sll_h`, `__builtin_msa_sll_w`, `__builtin_msa_sll_d`, `__builtin_msa_slli_b`, `__builtin_msa_slli_h`, `__builtin_msa_slli_w`, `__builtin_msa_slli_d`, `__builtin_msa_sra_b`, `__builtin_msa_sra_h`, `__builtin_msa_sra_w`, `__builtin_msa_sra_d`
