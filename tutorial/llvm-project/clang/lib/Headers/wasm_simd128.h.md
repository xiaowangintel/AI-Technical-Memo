# wasm_simd128.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/wasm_simd128.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: WebAssembly portable SIMD intrinsics.
- **Purpose (CN)**: 提供 WebAssembly portable SIMD intrinsic 接口。
- **Line Count / 行数**: 2152

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===---- wasm_simd128.h - WebAssembly portable SIMD intrinsics ------------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

#ifndef __WASM_SIMD128_H
#define __WASM_SIMD128_H

#include <stdbool.h>
#include <stdint.h>

// User-facing type
typedef int32_t v128_t __attribute__((__vector_size__(16), __aligned__(16)));

// Internal types determined by clang builtin definitions
typedef int32_t __v128_u __attribute__((__vector_size__(16), __aligned__(1)));
typedef signed char __i8x16
    __attribute__((__vector_size__(16), __aligned__(16)));
typedef unsigned char __u8x16
    __attribute__((__vector_size__(16), __aligned__(16)));
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
- **L10 EN**: Starts a preprocessor conditional block: `#ifndef __WASM_SIMD128_H`.
  **L10 CN**: 开始一个预处理条件块：`#ifndef __WASM_SIMD128_H`。
- **L11 EN**: Defines macro `__WASM_SIMD128_H` for conditional compilation, shorthand, or API generation.
  **L11 CN**: 定义宏 `__WASM_SIMD128_H`，用于条件编译、简写或 API 生成。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Includes <stdbool.h> to access related header declarations.
  **L13 CN**: 引入 <stdbool.h> 以使用相关头文件声明。
- **L14 EN**: Includes <stdint.h> to access fixed-width integer declarations.
  **L14 CN**: 引入 <stdint.h> 以使用定宽整数声明。
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L16 EN**: Comment explains nearby logic, constraints, or intent: `User-facing type`.
  **L16 CN**: 注释解释附近代码的逻辑、约束或设计意图：`User-facing type`。
- **L17 EN**: Introduces an alias or helper declaration: `typedef int32_t v128_t __attribute__((__vector_size__(16), __aligned__(16)));`.
  **L17 CN**: 引入一条别名或辅助声明：`typedef int32_t v128_t __attribute__((__vector_size__(16), __aligned__(16)));`。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L19 EN**: Comment explains nearby logic, constraints, or intent: `Internal types determined by clang builtin definitions`.
  **L19 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Internal types determined by clang builtin definitions`。
- **L20 EN**: Introduces an alias or helper declaration: `typedef int32_t __v128_u __attribute__((__vector_size__(16), __aligned__(1)));`.
  **L20 CN**: 引入一条别名或辅助声明：`typedef int32_t __v128_u __attribute__((__vector_size__(16), __aligned__(1)));`。
- **L21 EN**: Introduces an alias or helper declaration: `typedef signed char __i8x16`.
  **L21 CN**: 引入一条别名或辅助声明：`typedef signed char __i8x16`。
- **L22 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__vector_size__(16), __aligned__(16)));`.
  **L22 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__vector_size__(16), __aligned__(16)));`。
- **L23 EN**: Introduces an alias or helper declaration: `typedef unsigned char __u8x16`.
  **L23 CN**: 引入一条别名或辅助声明：`typedef unsigned char __u8x16`。
- **L24 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__vector_size__(16), __aligned__(16)));`.
  **L24 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__vector_size__(16), __aligned__(16)));`。

### Lines 25-48

````c
typedef short __i16x8 __attribute__((__vector_size__(16), __aligned__(16)));
typedef unsigned short __u16x8
    __attribute__((__vector_size__(16), __aligned__(16)));
typedef int __i32x4 __attribute__((__vector_size__(16), __aligned__(16)));
typedef unsigned int __u32x4
    __attribute__((__vector_size__(16), __aligned__(16)));
typedef long long __i64x2 __attribute__((__vector_size__(16), __aligned__(16)));
typedef unsigned long long __u64x2
    __attribute__((__vector_size__(16), __aligned__(16)));
typedef float __f32x4 __attribute__((__vector_size__(16), __aligned__(16)));
typedef double __f64x2 __attribute__((__vector_size__(16), __aligned__(16)));
typedef __fp16 __f16x8 __attribute__((__vector_size__(16), __aligned__(16)));

typedef signed char __i8x8 __attribute__((__vector_size__(8), __aligned__(8)));
typedef unsigned char __u8x8
    __attribute__((__vector_size__(8), __aligned__(8)));
typedef short __i16x4 __attribute__((__vector_size__(8), __aligned__(8)));
typedef unsigned short __u16x4
    __attribute__((__vector_size__(8), __aligned__(8)));
typedef int __i32x2 __attribute__((__vector_size__(8), __aligned__(8)));
typedef unsigned int __u32x2
    __attribute__((__vector_size__(8), __aligned__(8)));
typedef float __f32x2 __attribute__((__vector_size__(8), __aligned__(8)));
typedef __fp16 __f16x4 __attribute__((__vector_size__(8), __aligned__(8)));
````
- **L25 EN**: Introduces an alias or helper declaration: `typedef short __i16x8 __attribute__((__vector_size__(16), __aligned__(16)));`.
  **L25 CN**: 引入一条别名或辅助声明：`typedef short __i16x8 __attribute__((__vector_size__(16), __aligned__(16)));`。
- **L26 EN**: Introduces an alias or helper declaration: `typedef unsigned short __u16x8`.
  **L26 CN**: 引入一条别名或辅助声明：`typedef unsigned short __u16x8`。
- **L27 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__vector_size__(16), __aligned__(16)));`.
  **L27 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__vector_size__(16), __aligned__(16)));`。
- **L28 EN**: Introduces an alias or helper declaration: `typedef int __i32x4 __attribute__((__vector_size__(16), __aligned__(16)));`.
  **L28 CN**: 引入一条别名或辅助声明：`typedef int __i32x4 __attribute__((__vector_size__(16), __aligned__(16)));`。
- **L29 EN**: Introduces an alias or helper declaration: `typedef unsigned int __u32x4`.
  **L29 CN**: 引入一条别名或辅助声明：`typedef unsigned int __u32x4`。
- **L30 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__vector_size__(16), __aligned__(16)));`.
  **L30 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__vector_size__(16), __aligned__(16)));`。
- **L31 EN**: Introduces an alias or helper declaration: `typedef long long __i64x2 __attribute__((__vector_size__(16), __aligned__(16)));`.
  **L31 CN**: 引入一条别名或辅助声明：`typedef long long __i64x2 __attribute__((__vector_size__(16), __aligned__(16)));`。
- **L32 EN**: Introduces an alias or helper declaration: `typedef unsigned long long __u64x2`.
  **L32 CN**: 引入一条别名或辅助声明：`typedef unsigned long long __u64x2`。
- **L33 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__vector_size__(16), __aligned__(16)));`.
  **L33 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__vector_size__(16), __aligned__(16)));`。
- **L34 EN**: Introduces an alias or helper declaration: `typedef float __f32x4 __attribute__((__vector_size__(16), __aligned__(16)));`.
  **L34 CN**: 引入一条别名或辅助声明：`typedef float __f32x4 __attribute__((__vector_size__(16), __aligned__(16)));`。
- **L35 EN**: Introduces an alias or helper declaration: `typedef double __f64x2 __attribute__((__vector_size__(16), __aligned__(16)));`.
  **L35 CN**: 引入一条别名或辅助声明：`typedef double __f64x2 __attribute__((__vector_size__(16), __aligned__(16)));`。
- **L36 EN**: Introduces an alias or helper declaration: `typedef __fp16 __f16x8 __attribute__((__vector_size__(16), __aligned__(16)));`.
  **L36 CN**: 引入一条别名或辅助声明：`typedef __fp16 __f16x8 __attribute__((__vector_size__(16), __aligned__(16)));`。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L38 EN**: Introduces an alias or helper declaration: `typedef signed char __i8x8 __attribute__((__vector_size__(8), __aligned__(8)));`.
  **L38 CN**: 引入一条别名或辅助声明：`typedef signed char __i8x8 __attribute__((__vector_size__(8), __aligned__(8)));`。
- **L39 EN**: Introduces an alias or helper declaration: `typedef unsigned char __u8x8`.
  **L39 CN**: 引入一条别名或辅助声明：`typedef unsigned char __u8x8`。
- **L40 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__vector_size__(8), __aligned__(8)));`.
  **L40 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__vector_size__(8), __aligned__(8)));`。
- **L41 EN**: Introduces an alias or helper declaration: `typedef short __i16x4 __attribute__((__vector_size__(8), __aligned__(8)));`.
  **L41 CN**: 引入一条别名或辅助声明：`typedef short __i16x4 __attribute__((__vector_size__(8), __aligned__(8)));`。
- **L42 EN**: Introduces an alias or helper declaration: `typedef unsigned short __u16x4`.
  **L42 CN**: 引入一条别名或辅助声明：`typedef unsigned short __u16x4`。
- **L43 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__vector_size__(8), __aligned__(8)));`.
  **L43 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__vector_size__(8), __aligned__(8)));`。
- **L44 EN**: Introduces an alias or helper declaration: `typedef int __i32x2 __attribute__((__vector_size__(8), __aligned__(8)));`.
  **L44 CN**: 引入一条别名或辅助声明：`typedef int __i32x2 __attribute__((__vector_size__(8), __aligned__(8)));`。
- **L45 EN**: Introduces an alias or helper declaration: `typedef unsigned int __u32x2`.
  **L45 CN**: 引入一条别名或辅助声明：`typedef unsigned int __u32x2`。
- **L46 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__vector_size__(8), __aligned__(8)));`.
  **L46 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__vector_size__(8), __aligned__(8)));`。
- **L47 EN**: Introduces an alias or helper declaration: `typedef float __f32x2 __attribute__((__vector_size__(8), __aligned__(8)));`.
  **L47 CN**: 引入一条别名或辅助声明：`typedef float __f32x2 __attribute__((__vector_size__(8), __aligned__(8)));`。
- **L48 EN**: Introduces an alias or helper declaration: `typedef __fp16 __f16x4 __attribute__((__vector_size__(8), __aligned__(8)));`.
  **L48 CN**: 引入一条别名或辅助声明：`typedef __fp16 __f16x4 __attribute__((__vector_size__(8), __aligned__(8)));`。

### Lines 49-72

````c

#define __DEFAULT_FN_ATTRS                                                     \
  __attribute__((__always_inline__, __nodebug__, __target__("simd128"),        \
                 __min_vector_width__(128)))

#define __REQUIRE_CONSTANT(c)                                                  \
  __attribute__((__diagnose_if__(!__builtin_constant_p(c),                     \
                                 #c " must be constant", "error")))

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_load(const void *__mem) {
  // UB-free unaligned access copied from xmmintrin.h
  struct __wasm_v128_load_struct {
    __v128_u __v;
  } __attribute__((__packed__, __may_alias__));
  return ((const struct __wasm_v128_load_struct *)__mem)->__v;
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_v128_load8_splat(const void *__mem) {
  struct __wasm_v128_load8_splat_struct {
    uint8_t __v;
  } __attribute__((__packed__, __may_alias__));
  uint8_t __v = ((const struct __wasm_v128_load8_splat_struct *)__mem)->__v;
  return (v128_t)(__u8x16){__v, __v, __v, __v, __v, __v, __v, __v,
````
- **L49 EN**: Blank line separating nearby declarations or logic blocks.
  **L49 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L50 EN**: Defines macro `__DEFAULT_FN_ATTRS` for conditional compilation, shorthand, or API generation.
  **L50 CN**: 定义宏 `__DEFAULT_FN_ATTRS`，用于条件编译、简写或 API 生成。
- **L51 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("simd128"),        \`.
  **L51 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("simd128"),        \`。
- **L52 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L52 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L53 EN**: Blank line separating nearby declarations or logic blocks.
  **L53 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L54 EN**: Defines macro `__REQUIRE_CONSTANT(c)` for conditional compilation, shorthand, or API generation.
  **L54 CN**: 定义宏 `__REQUIRE_CONSTANT(c)`，用于条件编译、简写或 API 生成。
- **L55 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__diagnose_if__(!__builtin_constant_p(c),                     \`.
  **L55 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__diagnose_if__(!__builtin_constant_p(c),                     \`。
- **L56 EN**: Continues the surrounding expression or declaration: `#c " must be constant", "error")))`.
  **L56 CN**: 继续构造周围的表达式或声明：`#c " must be constant", "error")))`。
- **L57 EN**: Blank line separating nearby declarations or logic blocks.
  **L57 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L58 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_load(const void *__mem) {`.
  **L58 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_load(const void *__mem) {`。
- **L59 EN**: Comment explains nearby logic, constraints, or intent: `UB-free unaligned access copied from xmmintrin.h`.
  **L59 CN**: 注释解释附近代码的逻辑、约束或设计意图：`UB-free unaligned access copied from xmmintrin.h`。
- **L60 EN**: Declares struct `__wasm_v128_load_struct`.
  **L60 CN**: 声明 struct `__wasm_v128_load_struct`。
- **L61 EN**: Adds a standalone statement or declaration: `__v128_u __v;`.
  **L61 CN**: 添加一条独立语句或声明：`__v128_u __v;`。
- **L62 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L62 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L63 EN**: Returns from the current function with `((const struct __wasm_v128_load_struct *)__mem)->__v`.
  **L63 CN**: 以 `((const struct __wasm_v128_load_struct *)__mem)->__v` 从当前函数返回。
- **L64 EN**: Closes the current lexical scope or compound statement.
  **L64 CN**: 结束当前词法作用域或复合语句块。
- **L65 EN**: Blank line separating nearby declarations or logic blocks.
  **L65 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L66 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L66 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L67 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_v128_load8_splat(const void *__mem) {`.
  **L67 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_v128_load8_splat(const void *__mem) {`。
- **L68 EN**: Declares struct `__wasm_v128_load8_splat_struct`.
  **L68 CN**: 声明 struct `__wasm_v128_load8_splat_struct`。
- **L69 EN**: Adds a standalone statement or declaration: `uint8_t __v;`.
  **L69 CN**: 添加一条独立语句或声明：`uint8_t __v;`。
- **L70 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L70 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L71 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L71 CN**: 使用右侧表达式初始化变量 `__v`。
- **L72 EN**: Returns from the current function with `(v128_t)(__u8x16){__v, __v, __v, __v, __v, __v, __v, __v,`.
  **L72 CN**: 以 `(v128_t)(__u8x16){__v, __v, __v, __v, __v, __v, __v, __v,` 从当前函数返回。

### Lines 73-96

````c
                           __v, __v, __v, __v, __v, __v, __v, __v};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_v128_load16_splat(const void *__mem) {
  struct __wasm_v128_load16_splat_struct {
    uint16_t __v;
  } __attribute__((__packed__, __may_alias__));
  uint16_t __v = ((const struct __wasm_v128_load16_splat_struct *)__mem)->__v;
  return (v128_t)(__u16x8){__v, __v, __v, __v, __v, __v, __v, __v};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_v128_load32_splat(const void *__mem) {
  struct __wasm_v128_load32_splat_struct {
    uint32_t __v;
  } __attribute__((__packed__, __may_alias__));
  uint32_t __v = ((const struct __wasm_v128_load32_splat_struct *)__mem)->__v;
  return (v128_t)(__u32x4){__v, __v, __v, __v};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_v128_load64_splat(const void *__mem) {
  struct __wasm_v128_load64_splat_struct {
````
- **L73 EN**: Adds a standalone statement or declaration: `__v, __v, __v, __v, __v, __v, __v, __v};`.
  **L73 CN**: 添加一条独立语句或声明：`__v, __v, __v, __v, __v, __v, __v, __v};`。
- **L74 EN**: Closes the current lexical scope or compound statement.
  **L74 CN**: 结束当前词法作用域或复合语句块。
- **L75 EN**: Blank line separating nearby declarations or logic blocks.
  **L75 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L76 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L76 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L77 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_v128_load16_splat(const void *__mem) {`.
  **L77 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_v128_load16_splat(const void *__mem) {`。
- **L78 EN**: Declares struct `__wasm_v128_load16_splat_struct`.
  **L78 CN**: 声明 struct `__wasm_v128_load16_splat_struct`。
- **L79 EN**: Adds a standalone statement or declaration: `uint16_t __v;`.
  **L79 CN**: 添加一条独立语句或声明：`uint16_t __v;`。
- **L80 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L80 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L81 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L81 CN**: 使用右侧表达式初始化变量 `__v`。
- **L82 EN**: Returns from the current function with `(v128_t)(__u16x8){__v, __v, __v, __v, __v, __v, __v, __v}`.
  **L82 CN**: 以 `(v128_t)(__u16x8){__v, __v, __v, __v, __v, __v, __v, __v}` 从当前函数返回。
- **L83 EN**: Closes the current lexical scope or compound statement.
  **L83 CN**: 结束当前词法作用域或复合语句块。
- **L84 EN**: Blank line separating nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L85 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L85 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L86 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_v128_load32_splat(const void *__mem) {`.
  **L86 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_v128_load32_splat(const void *__mem) {`。
- **L87 EN**: Declares struct `__wasm_v128_load32_splat_struct`.
  **L87 CN**: 声明 struct `__wasm_v128_load32_splat_struct`。
- **L88 EN**: Adds a standalone statement or declaration: `uint32_t __v;`.
  **L88 CN**: 添加一条独立语句或声明：`uint32_t __v;`。
- **L89 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L89 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L90 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L90 CN**: 使用右侧表达式初始化变量 `__v`。
- **L91 EN**: Returns from the current function with `(v128_t)(__u32x4){__v, __v, __v, __v}`.
  **L91 CN**: 以 `(v128_t)(__u32x4){__v, __v, __v, __v}` 从当前函数返回。
- **L92 EN**: Closes the current lexical scope or compound statement.
  **L92 CN**: 结束当前词法作用域或复合语句块。
- **L93 EN**: Blank line separating nearby declarations or logic blocks.
  **L93 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L94 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L94 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L95 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_v128_load64_splat(const void *__mem) {`.
  **L95 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_v128_load64_splat(const void *__mem) {`。
- **L96 EN**: Declares struct `__wasm_v128_load64_splat_struct`.
  **L96 CN**: 声明 struct `__wasm_v128_load64_splat_struct`。

### Lines 97-120

````c
    uint64_t __v;
  } __attribute__((__packed__, __may_alias__));
  uint64_t __v = ((const struct __wasm_v128_load64_splat_struct *)__mem)->__v;
  return (v128_t)(__u64x2){__v, __v};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i16x8_load8x8(const void *__mem) {
  struct __wasm_i16x8_load8x8_struct {
    __i8x8 __v;
  } __attribute__((__packed__, __may_alias__));
  __i8x8 __v = ((const struct __wasm_i16x8_load8x8_struct *)__mem)->__v;
  return (v128_t) __builtin_convertvector(__v, __i16x8);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u16x8_load8x8(const void *__mem) {
  struct __wasm_u16x8_load8x8_struct {
    __u8x8 __v;
  } __attribute__((__packed__, __may_alias__));
  __u8x8 __v = ((const struct __wasm_u16x8_load8x8_struct *)__mem)->__v;
  return (v128_t) __builtin_convertvector(__v, __u16x8);
}

````
- **L97 EN**: Adds a standalone statement or declaration: `uint64_t __v;`.
  **L97 CN**: 添加一条独立语句或声明：`uint64_t __v;`。
- **L98 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L98 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L99 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L99 CN**: 使用右侧表达式初始化变量 `__v`。
- **L100 EN**: Returns from the current function with `(v128_t)(__u64x2){__v, __v}`.
  **L100 CN**: 以 `(v128_t)(__u64x2){__v, __v}` 从当前函数返回。
- **L101 EN**: Closes the current lexical scope or compound statement.
  **L101 CN**: 结束当前词法作用域或复合语句块。
- **L102 EN**: Blank line separating nearby declarations or logic blocks.
  **L102 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L103 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L103 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L104 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_load8x8(const void *__mem) {`.
  **L104 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_load8x8(const void *__mem) {`。
- **L105 EN**: Declares struct `__wasm_i16x8_load8x8_struct`.
  **L105 CN**: 声明 struct `__wasm_i16x8_load8x8_struct`。
- **L106 EN**: Adds a standalone statement or declaration: `__i8x8 __v;`.
  **L106 CN**: 添加一条独立语句或声明：`__i8x8 __v;`。
- **L107 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L107 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L108 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L108 CN**: 使用右侧表达式初始化变量 `__v`。
- **L109 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(__v, __i16x8)`.
  **L109 CN**: 以 `(v128_t) __builtin_convertvector(__v, __i16x8)` 从当前函数返回。
- **L110 EN**: Closes the current lexical scope or compound statement.
  **L110 CN**: 结束当前词法作用域或复合语句块。
- **L111 EN**: Blank line separating nearby declarations or logic blocks.
  **L111 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L112 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L112 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L113 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u16x8_load8x8(const void *__mem) {`.
  **L113 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u16x8_load8x8(const void *__mem) {`。
- **L114 EN**: Declares struct `__wasm_u16x8_load8x8_struct`.
  **L114 CN**: 声明 struct `__wasm_u16x8_load8x8_struct`。
- **L115 EN**: Adds a standalone statement or declaration: `__u8x8 __v;`.
  **L115 CN**: 添加一条独立语句或声明：`__u8x8 __v;`。
- **L116 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L116 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L117 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L117 CN**: 使用右侧表达式初始化变量 `__v`。
- **L118 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(__v, __u16x8)`.
  **L118 CN**: 以 `(v128_t) __builtin_convertvector(__v, __u16x8)` 从当前函数返回。
- **L119 EN**: Closes the current lexical scope or compound statement.
  **L119 CN**: 结束当前词法作用域或复合语句块。
- **L120 EN**: Blank line separating nearby declarations or logic blocks.
  **L120 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 121-144

````c
static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i32x4_load16x4(const void *__mem) {
  struct __wasm_i32x4_load16x4_struct {
    __i16x4 __v;
  } __attribute__((__packed__, __may_alias__));
  __i16x4 __v = ((const struct __wasm_i32x4_load16x4_struct *)__mem)->__v;
  return (v128_t) __builtin_convertvector(__v, __i32x4);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u32x4_load16x4(const void *__mem) {
  struct __wasm_u32x4_load16x4_struct {
    __u16x4 __v;
  } __attribute__((__packed__, __may_alias__));
  __u16x4 __v = ((const struct __wasm_u32x4_load16x4_struct *)__mem)->__v;
  return (v128_t) __builtin_convertvector(__v, __u32x4);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i64x2_load32x2(const void *__mem) {
  struct __wasm_i64x2_load32x2_struct {
    __i32x2 __v;
  } __attribute__((__packed__, __may_alias__));
  __i32x2 __v = ((const struct __wasm_i64x2_load32x2_struct *)__mem)->__v;
````
- **L121 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L121 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L122 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_load16x4(const void *__mem) {`.
  **L122 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_load16x4(const void *__mem) {`。
- **L123 EN**: Declares struct `__wasm_i32x4_load16x4_struct`.
  **L123 CN**: 声明 struct `__wasm_i32x4_load16x4_struct`。
- **L124 EN**: Adds a standalone statement or declaration: `__i16x4 __v;`.
  **L124 CN**: 添加一条独立语句或声明：`__i16x4 __v;`。
- **L125 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L125 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L126 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L126 CN**: 使用右侧表达式初始化变量 `__v`。
- **L127 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(__v, __i32x4)`.
  **L127 CN**: 以 `(v128_t) __builtin_convertvector(__v, __i32x4)` 从当前函数返回。
- **L128 EN**: Closes the current lexical scope or compound statement.
  **L128 CN**: 结束当前词法作用域或复合语句块。
- **L129 EN**: Blank line separating nearby declarations or logic blocks.
  **L129 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L130 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L130 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L131 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u32x4_load16x4(const void *__mem) {`.
  **L131 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u32x4_load16x4(const void *__mem) {`。
- **L132 EN**: Declares struct `__wasm_u32x4_load16x4_struct`.
  **L132 CN**: 声明 struct `__wasm_u32x4_load16x4_struct`。
- **L133 EN**: Adds a standalone statement or declaration: `__u16x4 __v;`.
  **L133 CN**: 添加一条独立语句或声明：`__u16x4 __v;`。
- **L134 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L134 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L135 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L135 CN**: 使用右侧表达式初始化变量 `__v`。
- **L136 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(__v, __u32x4)`.
  **L136 CN**: 以 `(v128_t) __builtin_convertvector(__v, __u32x4)` 从当前函数返回。
- **L137 EN**: Closes the current lexical scope or compound statement.
  **L137 CN**: 结束当前词法作用域或复合语句块。
- **L138 EN**: Blank line separating nearby declarations or logic blocks.
  **L138 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L139 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L139 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L140 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i64x2_load32x2(const void *__mem) {`.
  **L140 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i64x2_load32x2(const void *__mem) {`。
- **L141 EN**: Declares struct `__wasm_i64x2_load32x2_struct`.
  **L141 CN**: 声明 struct `__wasm_i64x2_load32x2_struct`。
- **L142 EN**: Adds a standalone statement or declaration: `__i32x2 __v;`.
  **L142 CN**: 添加一条独立语句或声明：`__i32x2 __v;`。
- **L143 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L143 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L144 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L144 CN**: 使用右侧表达式初始化变量 `__v`。

### Lines 145-168

````c
  return (v128_t) __builtin_convertvector(__v, __i64x2);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u64x2_load32x2(const void *__mem) {
  struct __wasm_u64x2_load32x2_struct {
    __u32x2 __v;
  } __attribute__((__packed__, __may_alias__));
  __u32x2 __v = ((const struct __wasm_u64x2_load32x2_struct *)__mem)->__v;
  return (v128_t) __builtin_convertvector(__v, __u64x2);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_v128_load32_zero(const void *__mem) {
  struct __wasm_v128_load32_zero_struct {
    int32_t __v;
  } __attribute__((__packed__, __may_alias__));
  int32_t __v = ((const struct __wasm_v128_load32_zero_struct *)__mem)->__v;
  return (v128_t)(__i32x4){__v, 0, 0, 0};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_v128_load64_zero(const void *__mem) {
  struct __wasm_v128_load64_zero_struct {
````
- **L145 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(__v, __i64x2)`.
  **L145 CN**: 以 `(v128_t) __builtin_convertvector(__v, __i64x2)` 从当前函数返回。
- **L146 EN**: Closes the current lexical scope or compound statement.
  **L146 CN**: 结束当前词法作用域或复合语句块。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L148 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L148 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L149 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u64x2_load32x2(const void *__mem) {`.
  **L149 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u64x2_load32x2(const void *__mem) {`。
- **L150 EN**: Declares struct `__wasm_u64x2_load32x2_struct`.
  **L150 CN**: 声明 struct `__wasm_u64x2_load32x2_struct`。
- **L151 EN**: Adds a standalone statement or declaration: `__u32x2 __v;`.
  **L151 CN**: 添加一条独立语句或声明：`__u32x2 __v;`。
- **L152 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L152 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L153 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L153 CN**: 使用右侧表达式初始化变量 `__v`。
- **L154 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(__v, __u64x2)`.
  **L154 CN**: 以 `(v128_t) __builtin_convertvector(__v, __u64x2)` 从当前函数返回。
- **L155 EN**: Closes the current lexical scope or compound statement.
  **L155 CN**: 结束当前词法作用域或复合语句块。
- **L156 EN**: Blank line separating nearby declarations or logic blocks.
  **L156 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L157 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L157 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L158 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_v128_load32_zero(const void *__mem) {`.
  **L158 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_v128_load32_zero(const void *__mem) {`。
- **L159 EN**: Declares struct `__wasm_v128_load32_zero_struct`.
  **L159 CN**: 声明 struct `__wasm_v128_load32_zero_struct`。
- **L160 EN**: Adds a standalone statement or declaration: `int32_t __v;`.
  **L160 CN**: 添加一条独立语句或声明：`int32_t __v;`。
- **L161 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L161 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L162 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L162 CN**: 使用右侧表达式初始化变量 `__v`。
- **L163 EN**: Returns from the current function with `(v128_t)(__i32x4){__v, 0, 0, 0}`.
  **L163 CN**: 以 `(v128_t)(__i32x4){__v, 0, 0, 0}` 从当前函数返回。
- **L164 EN**: Closes the current lexical scope or compound statement.
  **L164 CN**: 结束当前词法作用域或复合语句块。
- **L165 EN**: Blank line separating nearby declarations or logic blocks.
  **L165 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L166 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L166 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L167 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_v128_load64_zero(const void *__mem) {`.
  **L167 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_v128_load64_zero(const void *__mem) {`。
- **L168 EN**: Declares struct `__wasm_v128_load64_zero_struct`.
  **L168 CN**: 声明 struct `__wasm_v128_load64_zero_struct`。

### Lines 169-192

````c
    int64_t __v;
  } __attribute__((__packed__, __may_alias__));
  int64_t __v = ((const struct __wasm_v128_load64_zero_struct *)__mem)->__v;
  return (v128_t)(__i64x2){__v, 0};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_load8_lane(
    const void *__mem, v128_t __vec, int __i) __REQUIRE_CONSTANT(__i) {
  struct __wasm_v128_load8_lane_struct {
    int8_t __v;
  } __attribute__((__packed__, __may_alias__));
  int8_t __v = ((const struct __wasm_v128_load8_lane_struct *)__mem)->__v;
  __i8x16 __ret = (__i8x16)__vec;
  __ret[__i] = __v;
  return (v128_t)__ret;
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_load16_lane(
    const void *__mem, v128_t __vec, int __i) __REQUIRE_CONSTANT(__i) {
  struct __wasm_v128_load16_lane_struct {
    int16_t __v;
  } __attribute__((__packed__, __may_alias__));
  int16_t __v = ((const struct __wasm_v128_load16_lane_struct *)__mem)->__v;
  __i16x8 __ret = (__i16x8)__vec;
````
- **L169 EN**: Adds a standalone statement or declaration: `int64_t __v;`.
  **L169 CN**: 添加一条独立语句或声明：`int64_t __v;`。
- **L170 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L170 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L171 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L171 CN**: 使用右侧表达式初始化变量 `__v`。
- **L172 EN**: Returns from the current function with `(v128_t)(__i64x2){__v, 0}`.
  **L172 CN**: 以 `(v128_t)(__i64x2){__v, 0}` 从当前函数返回。
- **L173 EN**: Closes the current lexical scope or compound statement.
  **L173 CN**: 结束当前词法作用域或复合语句块。
- **L174 EN**: Blank line separating nearby declarations or logic blocks.
  **L174 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L175 EN**: Continues logic associated with callable symbol `wasm_v128_load8_lane`.
  **L175 CN**: 继续与可调用符号 `wasm_v128_load8_lane` 相关的逻辑。
- **L176 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `const void *__mem, v128_t __vec, int __i) __REQUIRE_CONSTANT(__i) {`.
  **L176 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`const void *__mem, v128_t __vec, int __i) __REQUIRE_CONSTANT(__i) {`。
- **L177 EN**: Declares struct `__wasm_v128_load8_lane_struct`.
  **L177 CN**: 声明 struct `__wasm_v128_load8_lane_struct`。
- **L178 EN**: Adds a standalone statement or declaration: `int8_t __v;`.
  **L178 CN**: 添加一条独立语句或声明：`int8_t __v;`。
- **L179 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L179 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L180 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L180 CN**: 使用右侧表达式初始化变量 `__v`。
- **L181 EN**: Initializes variable `__ret` from the expression on the right-hand side.
  **L181 CN**: 使用右侧表达式初始化变量 `__ret`。
- **L182 EN**: Adds a standalone statement or declaration: `__ret[__i] = __v;`.
  **L182 CN**: 添加一条独立语句或声明：`__ret[__i] = __v;`。
- **L183 EN**: Returns from the current function with `(v128_t)__ret`.
  **L183 CN**: 以 `(v128_t)__ret` 从当前函数返回。
- **L184 EN**: Closes the current lexical scope or compound statement.
  **L184 CN**: 结束当前词法作用域或复合语句块。
- **L185 EN**: Blank line separating nearby declarations or logic blocks.
  **L185 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L186 EN**: Continues logic associated with callable symbol `wasm_v128_load16_lane`.
  **L186 CN**: 继续与可调用符号 `wasm_v128_load16_lane` 相关的逻辑。
- **L187 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `const void *__mem, v128_t __vec, int __i) __REQUIRE_CONSTANT(__i) {`.
  **L187 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`const void *__mem, v128_t __vec, int __i) __REQUIRE_CONSTANT(__i) {`。
- **L188 EN**: Declares struct `__wasm_v128_load16_lane_struct`.
  **L188 CN**: 声明 struct `__wasm_v128_load16_lane_struct`。
- **L189 EN**: Adds a standalone statement or declaration: `int16_t __v;`.
  **L189 CN**: 添加一条独立语句或声明：`int16_t __v;`。
- **L190 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L190 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L191 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L191 CN**: 使用右侧表达式初始化变量 `__v`。
- **L192 EN**: Initializes variable `__ret` from the expression on the right-hand side.
  **L192 CN**: 使用右侧表达式初始化变量 `__ret`。

### Lines 193-216

````c
  __ret[__i] = __v;
  return (v128_t)__ret;
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_load32_lane(
    const void *__mem, v128_t __vec, int __i) __REQUIRE_CONSTANT(__i) {
  struct __wasm_v128_load32_lane_struct {
    int32_t __v;
  } __attribute__((__packed__, __may_alias__));
  int32_t __v = ((const struct __wasm_v128_load32_lane_struct *)__mem)->__v;
  __i32x4 __ret = (__i32x4)__vec;
  __ret[__i] = __v;
  return (v128_t)__ret;
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_load64_lane(
    const void *__mem, v128_t __vec, int __i) __REQUIRE_CONSTANT(__i) {
  struct __wasm_v128_load64_lane_struct {
    int64_t __v;
  } __attribute__((__packed__, __may_alias__));
  int64_t __v = ((const struct __wasm_v128_load64_lane_struct *)__mem)->__v;
  __i64x2 __ret = (__i64x2)__vec;
  __ret[__i] = __v;
  return (v128_t)__ret;
````
- **L193 EN**: Adds a standalone statement or declaration: `__ret[__i] = __v;`.
  **L193 CN**: 添加一条独立语句或声明：`__ret[__i] = __v;`。
- **L194 EN**: Returns from the current function with `(v128_t)__ret`.
  **L194 CN**: 以 `(v128_t)__ret` 从当前函数返回。
- **L195 EN**: Closes the current lexical scope or compound statement.
  **L195 CN**: 结束当前词法作用域或复合语句块。
- **L196 EN**: Blank line separating nearby declarations or logic blocks.
  **L196 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L197 EN**: Continues logic associated with callable symbol `wasm_v128_load32_lane`.
  **L197 CN**: 继续与可调用符号 `wasm_v128_load32_lane` 相关的逻辑。
- **L198 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `const void *__mem, v128_t __vec, int __i) __REQUIRE_CONSTANT(__i) {`.
  **L198 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`const void *__mem, v128_t __vec, int __i) __REQUIRE_CONSTANT(__i) {`。
- **L199 EN**: Declares struct `__wasm_v128_load32_lane_struct`.
  **L199 CN**: 声明 struct `__wasm_v128_load32_lane_struct`。
- **L200 EN**: Adds a standalone statement or declaration: `int32_t __v;`.
  **L200 CN**: 添加一条独立语句或声明：`int32_t __v;`。
- **L201 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L201 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L202 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L202 CN**: 使用右侧表达式初始化变量 `__v`。
- **L203 EN**: Initializes variable `__ret` from the expression on the right-hand side.
  **L203 CN**: 使用右侧表达式初始化变量 `__ret`。
- **L204 EN**: Adds a standalone statement or declaration: `__ret[__i] = __v;`.
  **L204 CN**: 添加一条独立语句或声明：`__ret[__i] = __v;`。
- **L205 EN**: Returns from the current function with `(v128_t)__ret`.
  **L205 CN**: 以 `(v128_t)__ret` 从当前函数返回。
- **L206 EN**: Closes the current lexical scope or compound statement.
  **L206 CN**: 结束当前词法作用域或复合语句块。
- **L207 EN**: Blank line separating nearby declarations or logic blocks.
  **L207 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L208 EN**: Continues logic associated with callable symbol `wasm_v128_load64_lane`.
  **L208 CN**: 继续与可调用符号 `wasm_v128_load64_lane` 相关的逻辑。
- **L209 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `const void *__mem, v128_t __vec, int __i) __REQUIRE_CONSTANT(__i) {`.
  **L209 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`const void *__mem, v128_t __vec, int __i) __REQUIRE_CONSTANT(__i) {`。
- **L210 EN**: Declares struct `__wasm_v128_load64_lane_struct`.
  **L210 CN**: 声明 struct `__wasm_v128_load64_lane_struct`。
- **L211 EN**: Adds a standalone statement or declaration: `int64_t __v;`.
  **L211 CN**: 添加一条独立语句或声明：`int64_t __v;`。
- **L212 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L212 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L213 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L213 CN**: 使用右侧表达式初始化变量 `__v`。
- **L214 EN**: Initializes variable `__ret` from the expression on the right-hand side.
  **L214 CN**: 使用右侧表达式初始化变量 `__ret`。
- **L215 EN**: Adds a standalone statement or declaration: `__ret[__i] = __v;`.
  **L215 CN**: 添加一条独立语句或声明：`__ret[__i] = __v;`。
- **L216 EN**: Returns from the current function with `(v128_t)__ret`.
  **L216 CN**: 以 `(v128_t)__ret` 从当前函数返回。

### Lines 217-240

````c
}

static __inline__ void __DEFAULT_FN_ATTRS wasm_v128_store(void *__mem,
                                                          v128_t __a) {
  // UB-free unaligned access copied from xmmintrin.h
  struct __wasm_v128_store_struct {
    __v128_u __v;
  } __attribute__((__packed__, __may_alias__));
  ((struct __wasm_v128_store_struct *)__mem)->__v = __a;
}

static __inline__ void __DEFAULT_FN_ATTRS wasm_v128_store8_lane(void *__mem,
                                                                v128_t __vec,
                                                                int __i)
    __REQUIRE_CONSTANT(__i) {
  struct __wasm_v128_store8_lane_struct {
    int8_t __v;
  } __attribute__((__packed__, __may_alias__));
  ((struct __wasm_v128_store8_lane_struct *)__mem)->__v = ((__i8x16)__vec)[__i];
}

static __inline__ void __DEFAULT_FN_ATTRS wasm_v128_store16_lane(void *__mem,
                                                                 v128_t __vec,
                                                                 int __i)
````
- **L217 EN**: Closes the current lexical scope or compound statement.
  **L217 CN**: 结束当前词法作用域或复合语句块。
- **L218 EN**: Blank line separating nearby declarations or logic blocks.
  **L218 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS wasm_v128_store(void *__mem,`.
  **L219 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS wasm_v128_store(void *__mem,`。
- **L220 EN**: Continues the surrounding expression or declaration: `v128_t __a) {`.
  **L220 CN**: 继续构造周围的表达式或声明：`v128_t __a) {`。
- **L221 EN**: Comment explains nearby logic, constraints, or intent: `UB-free unaligned access copied from xmmintrin.h`.
  **L221 CN**: 注释解释附近代码的逻辑、约束或设计意图：`UB-free unaligned access copied from xmmintrin.h`。
- **L222 EN**: Declares struct `__wasm_v128_store_struct`.
  **L222 CN**: 声明 struct `__wasm_v128_store_struct`。
- **L223 EN**: Adds a standalone statement or declaration: `__v128_u __v;`.
  **L223 CN**: 添加一条独立语句或声明：`__v128_u __v;`。
- **L224 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L224 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L225 EN**: Executes a call or declaration centered on `statement`.
  **L225 CN**: 执行以 `statement` 为核心的调用或声明。
- **L226 EN**: Closes the current lexical scope or compound statement.
  **L226 CN**: 结束当前词法作用域或复合语句块。
- **L227 EN**: Blank line separating nearby declarations or logic blocks.
  **L227 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS wasm_v128_store8_lane(void *__mem,`.
  **L228 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS wasm_v128_store8_lane(void *__mem,`。
- **L229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `v128_t __vec,`.
  **L229 CN**: 继续一个多行参数列表、初始化器或聚合项：`v128_t __vec,`。
- **L230 EN**: Continues the surrounding expression or declaration: `int __i)`.
  **L230 CN**: 继续构造周围的表达式或声明：`int __i)`。
- **L231 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L231 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L232 EN**: Declares struct `__wasm_v128_store8_lane_struct`.
  **L232 CN**: 声明 struct `__wasm_v128_store8_lane_struct`。
- **L233 EN**: Adds a standalone statement or declaration: `int8_t __v;`.
  **L233 CN**: 添加一条独立语句或声明：`int8_t __v;`。
- **L234 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L234 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L235 EN**: Executes a call or declaration centered on `statement`.
  **L235 CN**: 执行以 `statement` 为核心的调用或声明。
- **L236 EN**: Closes the current lexical scope or compound statement.
  **L236 CN**: 结束当前词法作用域或复合语句块。
- **L237 EN**: Blank line separating nearby declarations or logic blocks.
  **L237 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS wasm_v128_store16_lane(void *__mem,`.
  **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS wasm_v128_store16_lane(void *__mem,`。
- **L239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `v128_t __vec,`.
  **L239 CN**: 继续一个多行参数列表、初始化器或聚合项：`v128_t __vec,`。
- **L240 EN**: Continues the surrounding expression or declaration: `int __i)`.
  **L240 CN**: 继续构造周围的表达式或声明：`int __i)`。

### Lines 241-264

````c
    __REQUIRE_CONSTANT(__i) {
  struct __wasm_v128_store16_lane_struct {
    int16_t __v;
  } __attribute__((__packed__, __may_alias__));
  ((struct __wasm_v128_store16_lane_struct *)__mem)->__v =
      ((__i16x8)__vec)[__i];
}

static __inline__ void __DEFAULT_FN_ATTRS wasm_v128_store32_lane(void *__mem,
                                                                 v128_t __vec,
                                                                 int __i)
    __REQUIRE_CONSTANT(__i) {
  struct __wasm_v128_store32_lane_struct {
    int32_t __v;
  } __attribute__((__packed__, __may_alias__));
  ((struct __wasm_v128_store32_lane_struct *)__mem)->__v =
      ((__i32x4)__vec)[__i];
}

static __inline__ void __DEFAULT_FN_ATTRS wasm_v128_store64_lane(void *__mem,
                                                                 v128_t __vec,
                                                                 int __i)
    __REQUIRE_CONSTANT(__i) {
  struct __wasm_v128_store64_lane_struct {
````
- **L241 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L241 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L242 EN**: Declares struct `__wasm_v128_store16_lane_struct`.
  **L242 CN**: 声明 struct `__wasm_v128_store16_lane_struct`。
- **L243 EN**: Adds a standalone statement or declaration: `int16_t __v;`.
  **L243 CN**: 添加一条独立语句或声明：`int16_t __v;`。
- **L244 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L244 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L245 EN**: Continues the surrounding expression or declaration: `((struct __wasm_v128_store16_lane_struct *)__mem)->__v =`.
  **L245 CN**: 继续构造周围的表达式或声明：`((struct __wasm_v128_store16_lane_struct *)__mem)->__v =`。
- **L246 EN**: Executes a call or declaration centered on `statement`.
  **L246 CN**: 执行以 `statement` 为核心的调用或声明。
- **L247 EN**: Closes the current lexical scope or compound statement.
  **L247 CN**: 结束当前词法作用域或复合语句块。
- **L248 EN**: Blank line separating nearby declarations or logic blocks.
  **L248 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS wasm_v128_store32_lane(void *__mem,`.
  **L249 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS wasm_v128_store32_lane(void *__mem,`。
- **L250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `v128_t __vec,`.
  **L250 CN**: 继续一个多行参数列表、初始化器或聚合项：`v128_t __vec,`。
- **L251 EN**: Continues the surrounding expression or declaration: `int __i)`.
  **L251 CN**: 继续构造周围的表达式或声明：`int __i)`。
- **L252 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L252 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L253 EN**: Declares struct `__wasm_v128_store32_lane_struct`.
  **L253 CN**: 声明 struct `__wasm_v128_store32_lane_struct`。
- **L254 EN**: Adds a standalone statement or declaration: `int32_t __v;`.
  **L254 CN**: 添加一条独立语句或声明：`int32_t __v;`。
- **L255 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L255 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L256 EN**: Continues the surrounding expression or declaration: `((struct __wasm_v128_store32_lane_struct *)__mem)->__v =`.
  **L256 CN**: 继续构造周围的表达式或声明：`((struct __wasm_v128_store32_lane_struct *)__mem)->__v =`。
- **L257 EN**: Executes a call or declaration centered on `statement`.
  **L257 CN**: 执行以 `statement` 为核心的调用或声明。
- **L258 EN**: Closes the current lexical scope or compound statement.
  **L258 CN**: 结束当前词法作用域或复合语句块。
- **L259 EN**: Blank line separating nearby declarations or logic blocks.
  **L259 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ void __DEFAULT_FN_ATTRS wasm_v128_store64_lane(void *__mem,`.
  **L260 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ void __DEFAULT_FN_ATTRS wasm_v128_store64_lane(void *__mem,`。
- **L261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `v128_t __vec,`.
  **L261 CN**: 继续一个多行参数列表、初始化器或聚合项：`v128_t __vec,`。
- **L262 EN**: Continues the surrounding expression or declaration: `int __i)`.
  **L262 CN**: 继续构造周围的表达式或声明：`int __i)`。
- **L263 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L263 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L264 EN**: Declares struct `__wasm_v128_store64_lane_struct`.
  **L264 CN**: 声明 struct `__wasm_v128_store64_lane_struct`。

### Lines 265-288

````c
    int64_t __v;
  } __attribute__((__packed__, __may_alias__));
  ((struct __wasm_v128_store64_lane_struct *)__mem)->__v =
      ((__i64x2)__vec)[__i];
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i8x16_make(int8_t __c0, int8_t __c1, int8_t __c2, int8_t __c3, int8_t __c4,
                int8_t __c5, int8_t __c6, int8_t __c7, int8_t __c8, int8_t __c9,
                int8_t __c10, int8_t __c11, int8_t __c12, int8_t __c13,
                int8_t __c14, int8_t __c15) {
  return (v128_t)(__i8x16){__c0,  __c1,  __c2,  __c3, __c4,  __c5,
                           __c6,  __c7,  __c8,  __c9, __c10, __c11,
                           __c12, __c13, __c14, __c15};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u8x16_make(uint8_t __c0, uint8_t __c1, uint8_t __c2, uint8_t __c3,
                uint8_t __c4, uint8_t __c5, uint8_t __c6, uint8_t __c7,
                uint8_t __c8, uint8_t __c9, uint8_t __c10, uint8_t __c11,
                uint8_t __c12, uint8_t __c13, uint8_t __c14, uint8_t __c15) {
  return (v128_t)(__u8x16){__c0,  __c1,  __c2,  __c3, __c4,  __c5,
                           __c6,  __c7,  __c8,  __c9, __c10, __c11,
                           __c12, __c13, __c14, __c15};
````
- **L265 EN**: Adds a standalone statement or declaration: `int64_t __v;`.
  **L265 CN**: 添加一条独立语句或声明：`int64_t __v;`。
- **L266 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `} __attribute__((__packed__, __may_alias__));`.
  **L266 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`} __attribute__((__packed__, __may_alias__));`。
- **L267 EN**: Continues the surrounding expression or declaration: `((struct __wasm_v128_store64_lane_struct *)__mem)->__v =`.
  **L267 CN**: 继续构造周围的表达式或声明：`((struct __wasm_v128_store64_lane_struct *)__mem)->__v =`。
- **L268 EN**: Executes a call or declaration centered on `statement`.
  **L268 CN**: 执行以 `statement` 为核心的调用或声明。
- **L269 EN**: Closes the current lexical scope or compound statement.
  **L269 CN**: 结束当前词法作用域或复合语句块。
- **L270 EN**: Blank line separating nearby declarations or logic blocks.
  **L270 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L271 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L271 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `wasm_i8x16_make(int8_t __c0, int8_t __c1, int8_t __c2, int8_t __c3, int8_t __c4,`.
  **L272 CN**: 继续一个多行参数列表、初始化器或聚合项：`wasm_i8x16_make(int8_t __c0, int8_t __c1, int8_t __c2, int8_t __c3, int8_t __c4,`。
- **L273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int8_t __c5, int8_t __c6, int8_t __c7, int8_t __c8, int8_t __c9,`.
  **L273 CN**: 继续一个多行参数列表、初始化器或聚合项：`int8_t __c5, int8_t __c6, int8_t __c7, int8_t __c8, int8_t __c9,`。
- **L274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int8_t __c10, int8_t __c11, int8_t __c12, int8_t __c13,`.
  **L274 CN**: 继续一个多行参数列表、初始化器或聚合项：`int8_t __c10, int8_t __c11, int8_t __c12, int8_t __c13,`。
- **L275 EN**: Continues the surrounding expression or declaration: `int8_t __c14, int8_t __c15) {`.
  **L275 CN**: 继续构造周围的表达式或声明：`int8_t __c14, int8_t __c15) {`。
- **L276 EN**: Returns from the current function with `(v128_t)(__i8x16){__c0,  __c1,  __c2,  __c3, __c4,  __c5,`.
  **L276 CN**: 以 `(v128_t)(__i8x16){__c0,  __c1,  __c2,  __c3, __c4,  __c5,` 从当前函数返回。
- **L277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__c6,  __c7,  __c8,  __c9, __c10, __c11,`.
  **L277 CN**: 继续一个多行参数列表、初始化器或聚合项：`__c6,  __c7,  __c8,  __c9, __c10, __c11,`。
- **L278 EN**: Adds a standalone statement or declaration: `__c12, __c13, __c14, __c15};`.
  **L278 CN**: 添加一条独立语句或声明：`__c12, __c13, __c14, __c15};`。
- **L279 EN**: Closes the current lexical scope or compound statement.
  **L279 CN**: 结束当前词法作用域或复合语句块。
- **L280 EN**: Blank line separating nearby declarations or logic blocks.
  **L280 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L281 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L281 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `wasm_u8x16_make(uint8_t __c0, uint8_t __c1, uint8_t __c2, uint8_t __c3,`.
  **L282 CN**: 继续一个多行参数列表、初始化器或聚合项：`wasm_u8x16_make(uint8_t __c0, uint8_t __c1, uint8_t __c2, uint8_t __c3,`。
- **L283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `uint8_t __c4, uint8_t __c5, uint8_t __c6, uint8_t __c7,`.
  **L283 CN**: 继续一个多行参数列表、初始化器或聚合项：`uint8_t __c4, uint8_t __c5, uint8_t __c6, uint8_t __c7,`。
- **L284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `uint8_t __c8, uint8_t __c9, uint8_t __c10, uint8_t __c11,`.
  **L284 CN**: 继续一个多行参数列表、初始化器或聚合项：`uint8_t __c8, uint8_t __c9, uint8_t __c10, uint8_t __c11,`。
- **L285 EN**: Continues the surrounding expression or declaration: `uint8_t __c12, uint8_t __c13, uint8_t __c14, uint8_t __c15) {`.
  **L285 CN**: 继续构造周围的表达式或声明：`uint8_t __c12, uint8_t __c13, uint8_t __c14, uint8_t __c15) {`。
- **L286 EN**: Returns from the current function with `(v128_t)(__u8x16){__c0,  __c1,  __c2,  __c3, __c4,  __c5,`.
  **L286 CN**: 以 `(v128_t)(__u8x16){__c0,  __c1,  __c2,  __c3, __c4,  __c5,` 从当前函数返回。
- **L287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__c6,  __c7,  __c8,  __c9, __c10, __c11,`.
  **L287 CN**: 继续一个多行参数列表、初始化器或聚合项：`__c6,  __c7,  __c8,  __c9, __c10, __c11,`。
- **L288 EN**: Adds a standalone statement or declaration: `__c12, __c13, __c14, __c15};`.
  **L288 CN**: 添加一条独立语句或声明：`__c12, __c13, __c14, __c15};`。

### Lines 289-312

````c
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i16x8_make(int16_t __c0, int16_t __c1, int16_t __c2, int16_t __c3,
                int16_t __c4, int16_t __c5, int16_t __c6, int16_t __c7) {
  return (v128_t)(__i16x8){__c0, __c1, __c2, __c3, __c4, __c5, __c6, __c7};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u16x8_make(uint16_t __c0, uint16_t __c1, uint16_t __c2, uint16_t __c3,
                uint16_t __c4, uint16_t __c5, uint16_t __c6, uint16_t __c7) {
  return (v128_t)(__u16x8){__c0, __c1, __c2, __c3, __c4, __c5, __c6, __c7};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_make(int32_t __c0,
                                                            int32_t __c1,
                                                            int32_t __c2,
                                                            int32_t __c3) {
  return (v128_t)(__i32x4){__c0, __c1, __c2, __c3};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_make(uint32_t __c0,
                                                            uint32_t __c1,
                                                            uint32_t __c2,
````
- **L289 EN**: Closes the current lexical scope or compound statement.
  **L289 CN**: 结束当前词法作用域或复合语句块。
- **L290 EN**: Blank line separating nearby declarations or logic blocks.
  **L290 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L291 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L291 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `wasm_i16x8_make(int16_t __c0, int16_t __c1, int16_t __c2, int16_t __c3,`.
  **L292 CN**: 继续一个多行参数列表、初始化器或聚合项：`wasm_i16x8_make(int16_t __c0, int16_t __c1, int16_t __c2, int16_t __c3,`。
- **L293 EN**: Continues the surrounding expression or declaration: `int16_t __c4, int16_t __c5, int16_t __c6, int16_t __c7) {`.
  **L293 CN**: 继续构造周围的表达式或声明：`int16_t __c4, int16_t __c5, int16_t __c6, int16_t __c7) {`。
- **L294 EN**: Returns from the current function with `(v128_t)(__i16x8){__c0, __c1, __c2, __c3, __c4, __c5, __c6, __c7}`.
  **L294 CN**: 以 `(v128_t)(__i16x8){__c0, __c1, __c2, __c3, __c4, __c5, __c6, __c7}` 从当前函数返回。
- **L295 EN**: Closes the current lexical scope or compound statement.
  **L295 CN**: 结束当前词法作用域或复合语句块。
- **L296 EN**: Blank line separating nearby declarations or logic blocks.
  **L296 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L297 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L297 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `wasm_u16x8_make(uint16_t __c0, uint16_t __c1, uint16_t __c2, uint16_t __c3,`.
  **L298 CN**: 继续一个多行参数列表、初始化器或聚合项：`wasm_u16x8_make(uint16_t __c0, uint16_t __c1, uint16_t __c2, uint16_t __c3,`。
- **L299 EN**: Continues the surrounding expression or declaration: `uint16_t __c4, uint16_t __c5, uint16_t __c6, uint16_t __c7) {`.
  **L299 CN**: 继续构造周围的表达式或声明：`uint16_t __c4, uint16_t __c5, uint16_t __c6, uint16_t __c7) {`。
- **L300 EN**: Returns from the current function with `(v128_t)(__u16x8){__c0, __c1, __c2, __c3, __c4, __c5, __c6, __c7}`.
  **L300 CN**: 以 `(v128_t)(__u16x8){__c0, __c1, __c2, __c3, __c4, __c5, __c6, __c7}` 从当前函数返回。
- **L301 EN**: Closes the current lexical scope or compound statement.
  **L301 CN**: 结束当前词法作用域或复合语句块。
- **L302 EN**: Blank line separating nearby declarations or logic blocks.
  **L302 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_make(int32_t __c0,`.
  **L303 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_make(int32_t __c0,`。
- **L304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int32_t __c1,`.
  **L304 CN**: 继续一个多行参数列表、初始化器或聚合项：`int32_t __c1,`。
- **L305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int32_t __c2,`.
  **L305 CN**: 继续一个多行参数列表、初始化器或聚合项：`int32_t __c2,`。
- **L306 EN**: Continues the surrounding expression or declaration: `int32_t __c3) {`.
  **L306 CN**: 继续构造周围的表达式或声明：`int32_t __c3) {`。
- **L307 EN**: Returns from the current function with `(v128_t)(__i32x4){__c0, __c1, __c2, __c3}`.
  **L307 CN**: 以 `(v128_t)(__i32x4){__c0, __c1, __c2, __c3}` 从当前函数返回。
- **L308 EN**: Closes the current lexical scope or compound statement.
  **L308 CN**: 结束当前词法作用域或复合语句块。
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_make(uint32_t __c0,`.
  **L310 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_make(uint32_t __c0,`。
- **L311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `uint32_t __c1,`.
  **L311 CN**: 继续一个多行参数列表、初始化器或聚合项：`uint32_t __c1,`。
- **L312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `uint32_t __c2,`.
  **L312 CN**: 继续一个多行参数列表、初始化器或聚合项：`uint32_t __c2,`。

### Lines 313-336

````c
                                                            uint32_t __c3) {
  return (v128_t)(__u32x4){__c0, __c1, __c2, __c3};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_make(int64_t __c0,
                                                            int64_t __c1) {
  return (v128_t)(__i64x2){__c0, __c1};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u64x2_make(uint64_t __c0,
                                                            uint64_t __c1) {
  return (v128_t)(__u64x2){__c0, __c1};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_make(float __c0,
                                                            float __c1,
                                                            float __c2,
                                                            float __c3) {
  return (v128_t)(__f32x4){__c0, __c1, __c2, __c3};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_make(double __c0,
                                                            double __c1) {
  return (v128_t)(__f64x2){__c0, __c1};
````
- **L313 EN**: Continues the surrounding expression or declaration: `uint32_t __c3) {`.
  **L313 CN**: 继续构造周围的表达式或声明：`uint32_t __c3) {`。
- **L314 EN**: Returns from the current function with `(v128_t)(__u32x4){__c0, __c1, __c2, __c3}`.
  **L314 CN**: 以 `(v128_t)(__u32x4){__c0, __c1, __c2, __c3}` 从当前函数返回。
- **L315 EN**: Closes the current lexical scope or compound statement.
  **L315 CN**: 结束当前词法作用域或复合语句块。
- **L316 EN**: Blank line separating nearby declarations or logic blocks.
  **L316 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_make(int64_t __c0,`.
  **L317 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_make(int64_t __c0,`。
- **L318 EN**: Continues the surrounding expression or declaration: `int64_t __c1) {`.
  **L318 CN**: 继续构造周围的表达式或声明：`int64_t __c1) {`。
- **L319 EN**: Returns from the current function with `(v128_t)(__i64x2){__c0, __c1}`.
  **L319 CN**: 以 `(v128_t)(__i64x2){__c0, __c1}` 从当前函数返回。
- **L320 EN**: Closes the current lexical scope or compound statement.
  **L320 CN**: 结束当前词法作用域或复合语句块。
- **L321 EN**: Blank line separating nearby declarations or logic blocks.
  **L321 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L322 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u64x2_make(uint64_t __c0,`.
  **L322 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u64x2_make(uint64_t __c0,`。
- **L323 EN**: Continues the surrounding expression or declaration: `uint64_t __c1) {`.
  **L323 CN**: 继续构造周围的表达式或声明：`uint64_t __c1) {`。
- **L324 EN**: Returns from the current function with `(v128_t)(__u64x2){__c0, __c1}`.
  **L324 CN**: 以 `(v128_t)(__u64x2){__c0, __c1}` 从当前函数返回。
- **L325 EN**: Closes the current lexical scope or compound statement.
  **L325 CN**: 结束当前词法作用域或复合语句块。
- **L326 EN**: Blank line separating nearby declarations or logic blocks.
  **L326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_make(float __c0,`.
  **L327 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_make(float __c0,`。
- **L328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float __c1,`.
  **L328 CN**: 继续一个多行参数列表、初始化器或聚合项：`float __c1,`。
- **L329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `float __c2,`.
  **L329 CN**: 继续一个多行参数列表、初始化器或聚合项：`float __c2,`。
- **L330 EN**: Continues the surrounding expression or declaration: `float __c3) {`.
  **L330 CN**: 继续构造周围的表达式或声明：`float __c3) {`。
- **L331 EN**: Returns from the current function with `(v128_t)(__f32x4){__c0, __c1, __c2, __c3}`.
  **L331 CN**: 以 `(v128_t)(__f32x4){__c0, __c1, __c2, __c3}` 从当前函数返回。
- **L332 EN**: Closes the current lexical scope or compound statement.
  **L332 CN**: 结束当前词法作用域或复合语句块。
- **L333 EN**: Blank line separating nearby declarations or logic blocks.
  **L333 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_make(double __c0,`.
  **L334 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_make(double __c0,`。
- **L335 EN**: Continues the surrounding expression or declaration: `double __c1) {`.
  **L335 CN**: 继续构造周围的表达式或声明：`double __c1) {`。
- **L336 EN**: Returns from the current function with `(v128_t)(__f64x2){__c0, __c1}`.
  **L336 CN**: 以 `(v128_t)(__f64x2){__c0, __c1}` 从当前函数返回。

### Lines 337-360

````c
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i8x16_const(int8_t __c0, int8_t __c1, int8_t __c2, int8_t __c3,
                 int8_t __c4, int8_t __c5, int8_t __c6, int8_t __c7,
                 int8_t __c8, int8_t __c9, int8_t __c10, int8_t __c11,
                 int8_t __c12, int8_t __c13, int8_t __c14, int8_t __c15)
    __REQUIRE_CONSTANT(__c0) __REQUIRE_CONSTANT(__c1) __REQUIRE_CONSTANT(__c2)
        __REQUIRE_CONSTANT(__c3) __REQUIRE_CONSTANT(__c4)
            __REQUIRE_CONSTANT(__c5) __REQUIRE_CONSTANT(__c6)
                __REQUIRE_CONSTANT(__c7) __REQUIRE_CONSTANT(__c8)
                    __REQUIRE_CONSTANT(__c9) __REQUIRE_CONSTANT(__c10)
                        __REQUIRE_CONSTANT(__c11) __REQUIRE_CONSTANT(__c12)
                            __REQUIRE_CONSTANT(__c13) __REQUIRE_CONSTANT(__c14)
                                __REQUIRE_CONSTANT(__c15) {
  return (v128_t)(__i8x16){__c0,  __c1,  __c2,  __c3, __c4,  __c5,
                           __c6,  __c7,  __c8,  __c9, __c10, __c11,
                           __c12, __c13, __c14, __c15};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u8x16_const(uint8_t __c0, uint8_t __c1, uint8_t __c2, uint8_t __c3,
                 uint8_t __c4, uint8_t __c5, uint8_t __c6, uint8_t __c7,
                 uint8_t __c8, uint8_t __c9, uint8_t __c10, uint8_t __c11,
````
- **L337 EN**: Closes the current lexical scope or compound statement.
  **L337 CN**: 结束当前词法作用域或复合语句块。
- **L338 EN**: Blank line separating nearby declarations or logic blocks.
  **L338 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L339 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L339 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `wasm_i8x16_const(int8_t __c0, int8_t __c1, int8_t __c2, int8_t __c3,`.
  **L340 CN**: 继续一个多行参数列表、初始化器或聚合项：`wasm_i8x16_const(int8_t __c0, int8_t __c1, int8_t __c2, int8_t __c3,`。
- **L341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int8_t __c4, int8_t __c5, int8_t __c6, int8_t __c7,`.
  **L341 CN**: 继续一个多行参数列表、初始化器或聚合项：`int8_t __c4, int8_t __c5, int8_t __c6, int8_t __c7,`。
- **L342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int8_t __c8, int8_t __c9, int8_t __c10, int8_t __c11,`.
  **L342 CN**: 继续一个多行参数列表、初始化器或聚合项：`int8_t __c8, int8_t __c9, int8_t __c10, int8_t __c11,`。
- **L343 EN**: Continues the surrounding expression or declaration: `int8_t __c12, int8_t __c13, int8_t __c14, int8_t __c15)`.
  **L343 CN**: 继续构造周围的表达式或声明：`int8_t __c12, int8_t __c13, int8_t __c14, int8_t __c15)`。
- **L344 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L344 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L345 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L345 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L346 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L346 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L347 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L347 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L348 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L348 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L349 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L349 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L350 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L350 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L351 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c15) {`.
  **L351 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c15) {`。
- **L352 EN**: Returns from the current function with `(v128_t)(__i8x16){__c0,  __c1,  __c2,  __c3, __c4,  __c5,`.
  **L352 CN**: 以 `(v128_t)(__i8x16){__c0,  __c1,  __c2,  __c3, __c4,  __c5,` 从当前函数返回。
- **L353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__c6,  __c7,  __c8,  __c9, __c10, __c11,`.
  **L353 CN**: 继续一个多行参数列表、初始化器或聚合项：`__c6,  __c7,  __c8,  __c9, __c10, __c11,`。
- **L354 EN**: Adds a standalone statement or declaration: `__c12, __c13, __c14, __c15};`.
  **L354 CN**: 添加一条独立语句或声明：`__c12, __c13, __c14, __c15};`。
- **L355 EN**: Closes the current lexical scope or compound statement.
  **L355 CN**: 结束当前词法作用域或复合语句块。
- **L356 EN**: Blank line separating nearby declarations or logic blocks.
  **L356 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L357 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L357 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `wasm_u8x16_const(uint8_t __c0, uint8_t __c1, uint8_t __c2, uint8_t __c3,`.
  **L358 CN**: 继续一个多行参数列表、初始化器或聚合项：`wasm_u8x16_const(uint8_t __c0, uint8_t __c1, uint8_t __c2, uint8_t __c3,`。
- **L359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `uint8_t __c4, uint8_t __c5, uint8_t __c6, uint8_t __c7,`.
  **L359 CN**: 继续一个多行参数列表、初始化器或聚合项：`uint8_t __c4, uint8_t __c5, uint8_t __c6, uint8_t __c7,`。
- **L360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `uint8_t __c8, uint8_t __c9, uint8_t __c10, uint8_t __c11,`.
  **L360 CN**: 继续一个多行参数列表、初始化器或聚合项：`uint8_t __c8, uint8_t __c9, uint8_t __c10, uint8_t __c11,`。

### Lines 361-384

````c
                 uint8_t __c12, uint8_t __c13, uint8_t __c14, uint8_t __c15)
    __REQUIRE_CONSTANT(__c0) __REQUIRE_CONSTANT(__c1) __REQUIRE_CONSTANT(__c2)
        __REQUIRE_CONSTANT(__c3) __REQUIRE_CONSTANT(__c4)
            __REQUIRE_CONSTANT(__c5) __REQUIRE_CONSTANT(__c6)
                __REQUIRE_CONSTANT(__c7) __REQUIRE_CONSTANT(__c8)
                    __REQUIRE_CONSTANT(__c9) __REQUIRE_CONSTANT(__c10)
                        __REQUIRE_CONSTANT(__c11) __REQUIRE_CONSTANT(__c12)
                            __REQUIRE_CONSTANT(__c13) __REQUIRE_CONSTANT(__c14)
                                __REQUIRE_CONSTANT(__c15) {
  return (v128_t)(__u8x16){__c0,  __c1,  __c2,  __c3, __c4,  __c5,
                           __c6,  __c7,  __c8,  __c9, __c10, __c11,
                           __c12, __c13, __c14, __c15};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i16x8_const(int16_t __c0, int16_t __c1, int16_t __c2, int16_t __c3,
                 int16_t __c4, int16_t __c5, int16_t __c6, int16_t __c7)
    __REQUIRE_CONSTANT(__c0) __REQUIRE_CONSTANT(__c1) __REQUIRE_CONSTANT(__c2)
        __REQUIRE_CONSTANT(__c3) __REQUIRE_CONSTANT(__c4)
            __REQUIRE_CONSTANT(__c5) __REQUIRE_CONSTANT(__c6)
                __REQUIRE_CONSTANT(__c7) {
  return (v128_t)(__i16x8){__c0, __c1, __c2, __c3, __c4, __c5, __c6, __c7};
}

````
- **L361 EN**: Continues the surrounding expression or declaration: `uint8_t __c12, uint8_t __c13, uint8_t __c14, uint8_t __c15)`.
  **L361 CN**: 继续构造周围的表达式或声明：`uint8_t __c12, uint8_t __c13, uint8_t __c14, uint8_t __c15)`。
- **L362 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L362 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L363 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L363 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L364 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L364 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L365 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L365 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L366 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L366 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L367 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L367 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L368 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L368 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L369 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c15) {`.
  **L369 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c15) {`。
- **L370 EN**: Returns from the current function with `(v128_t)(__u8x16){__c0,  __c1,  __c2,  __c3, __c4,  __c5,`.
  **L370 CN**: 以 `(v128_t)(__u8x16){__c0,  __c1,  __c2,  __c3, __c4,  __c5,` 从当前函数返回。
- **L371 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__c6,  __c7,  __c8,  __c9, __c10, __c11,`.
  **L371 CN**: 继续一个多行参数列表、初始化器或聚合项：`__c6,  __c7,  __c8,  __c9, __c10, __c11,`。
- **L372 EN**: Adds a standalone statement or declaration: `__c12, __c13, __c14, __c15};`.
  **L372 CN**: 添加一条独立语句或声明：`__c12, __c13, __c14, __c15};`。
- **L373 EN**: Closes the current lexical scope or compound statement.
  **L373 CN**: 结束当前词法作用域或复合语句块。
- **L374 EN**: Blank line separating nearby declarations or logic blocks.
  **L374 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L375 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L375 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `wasm_i16x8_const(int16_t __c0, int16_t __c1, int16_t __c2, int16_t __c3,`.
  **L376 CN**: 继续一个多行参数列表、初始化器或聚合项：`wasm_i16x8_const(int16_t __c0, int16_t __c1, int16_t __c2, int16_t __c3,`。
- **L377 EN**: Continues the surrounding expression or declaration: `int16_t __c4, int16_t __c5, int16_t __c6, int16_t __c7)`.
  **L377 CN**: 继续构造周围的表达式或声明：`int16_t __c4, int16_t __c5, int16_t __c6, int16_t __c7)`。
- **L378 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L378 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L379 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L379 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L380 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L380 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L381 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c7) {`.
  **L381 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c7) {`。
- **L382 EN**: Returns from the current function with `(v128_t)(__i16x8){__c0, __c1, __c2, __c3, __c4, __c5, __c6, __c7}`.
  **L382 CN**: 以 `(v128_t)(__i16x8){__c0, __c1, __c2, __c3, __c4, __c5, __c6, __c7}` 从当前函数返回。
- **L383 EN**: Closes the current lexical scope or compound statement.
  **L383 CN**: 结束当前词法作用域或复合语句块。
- **L384 EN**: Blank line separating nearby declarations or logic blocks.
  **L384 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 385-408

````c
static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u16x8_const(uint16_t __c0, uint16_t __c1, uint16_t __c2, uint16_t __c3,
                 uint16_t __c4, uint16_t __c5, uint16_t __c6, uint16_t __c7)
    __REQUIRE_CONSTANT(__c0) __REQUIRE_CONSTANT(__c1) __REQUIRE_CONSTANT(__c2)
        __REQUIRE_CONSTANT(__c3) __REQUIRE_CONSTANT(__c4)
            __REQUIRE_CONSTANT(__c5) __REQUIRE_CONSTANT(__c6)
                __REQUIRE_CONSTANT(__c7) {
  return (v128_t)(__u16x8){__c0, __c1, __c2, __c3, __c4, __c5, __c6, __c7};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i32x4_const(int32_t __c0, int32_t __c1, int32_t __c2, int32_t __c3)
    __REQUIRE_CONSTANT(__c0) __REQUIRE_CONSTANT(__c1) __REQUIRE_CONSTANT(__c2)
        __REQUIRE_CONSTANT(__c3) {
  return (v128_t)(__i32x4){__c0, __c1, __c2, __c3};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u32x4_const(uint32_t __c0, uint32_t __c1, uint32_t __c2, uint32_t __c3)
    __REQUIRE_CONSTANT(__c0) __REQUIRE_CONSTANT(__c1) __REQUIRE_CONSTANT(__c2)
        __REQUIRE_CONSTANT(__c3) {
  return (v128_t)(__u32x4){__c0, __c1, __c2, __c3};
}

````
- **L385 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L385 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `wasm_u16x8_const(uint16_t __c0, uint16_t __c1, uint16_t __c2, uint16_t __c3,`.
  **L386 CN**: 继续一个多行参数列表、初始化器或聚合项：`wasm_u16x8_const(uint16_t __c0, uint16_t __c1, uint16_t __c2, uint16_t __c3,`。
- **L387 EN**: Continues the surrounding expression or declaration: `uint16_t __c4, uint16_t __c5, uint16_t __c6, uint16_t __c7)`.
  **L387 CN**: 继续构造周围的表达式或声明：`uint16_t __c4, uint16_t __c5, uint16_t __c6, uint16_t __c7)`。
- **L388 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L388 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L389 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L389 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L390 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L390 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L391 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c7) {`.
  **L391 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c7) {`。
- **L392 EN**: Returns from the current function with `(v128_t)(__u16x8){__c0, __c1, __c2, __c3, __c4, __c5, __c6, __c7}`.
  **L392 CN**: 以 `(v128_t)(__u16x8){__c0, __c1, __c2, __c3, __c4, __c5, __c6, __c7}` 从当前函数返回。
- **L393 EN**: Closes the current lexical scope or compound statement.
  **L393 CN**: 结束当前词法作用域或复合语句块。
- **L394 EN**: Blank line separating nearby declarations or logic blocks.
  **L394 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L395 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L395 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L396 EN**: Continues logic associated with callable symbol `wasm_i32x4_const`.
  **L396 CN**: 继续与可调用符号 `wasm_i32x4_const` 相关的逻辑。
- **L397 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L397 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L398 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c3) {`.
  **L398 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c3) {`。
- **L399 EN**: Returns from the current function with `(v128_t)(__i32x4){__c0, __c1, __c2, __c3}`.
  **L399 CN**: 以 `(v128_t)(__i32x4){__c0, __c1, __c2, __c3}` 从当前函数返回。
- **L400 EN**: Closes the current lexical scope or compound statement.
  **L400 CN**: 结束当前词法作用域或复合语句块。
- **L401 EN**: Blank line separating nearby declarations or logic blocks.
  **L401 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L402 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L402 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L403 EN**: Continues logic associated with callable symbol `wasm_u32x4_const`.
  **L403 CN**: 继续与可调用符号 `wasm_u32x4_const` 相关的逻辑。
- **L404 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L404 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L405 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c3) {`.
  **L405 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c3) {`。
- **L406 EN**: Returns from the current function with `(v128_t)(__u32x4){__c0, __c1, __c2, __c3}`.
  **L406 CN**: 以 `(v128_t)(__u32x4){__c0, __c1, __c2, __c3}` 从当前函数返回。
- **L407 EN**: Closes the current lexical scope or compound statement.
  **L407 CN**: 结束当前词法作用域或复合语句块。
- **L408 EN**: Blank line separating nearby declarations or logic blocks.
  **L408 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 409-432

````c
static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_const(int64_t __c0,
                                                             int64_t __c1)
    __REQUIRE_CONSTANT(__c0) __REQUIRE_CONSTANT(__c1) {
  return (v128_t)(__i64x2){__c0, __c1};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u64x2_const(uint64_t __c0,
                                                             uint64_t __c1)
    __REQUIRE_CONSTANT(__c0) __REQUIRE_CONSTANT(__c1) {
  return (v128_t)(__u64x2){__c0, __c1};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_f32x4_const(float __c0, float __c1, float __c2, float __c3)
    __REQUIRE_CONSTANT(__c0) __REQUIRE_CONSTANT(__c1) __REQUIRE_CONSTANT(__c2)
        __REQUIRE_CONSTANT(__c3) {
  return (v128_t)(__f32x4){__c0, __c1, __c2, __c3};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_const(double __c0,
                                                             double __c1)
    __REQUIRE_CONSTANT(__c0) __REQUIRE_CONSTANT(__c1) {
  return (v128_t)(__f64x2){__c0, __c1};
}
````
- **L409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_const(int64_t __c0,`.
  **L409 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_const(int64_t __c0,`。
- **L410 EN**: Continues the surrounding expression or declaration: `int64_t __c1)`.
  **L410 CN**: 继续构造周围的表达式或声明：`int64_t __c1)`。
- **L411 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c0) __REQUIRE_CONSTANT(__c1) {`.
  **L411 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c0) __REQUIRE_CONSTANT(__c1) {`。
- **L412 EN**: Returns from the current function with `(v128_t)(__i64x2){__c0, __c1}`.
  **L412 CN**: 以 `(v128_t)(__i64x2){__c0, __c1}` 从当前函数返回。
- **L413 EN**: Closes the current lexical scope or compound statement.
  **L413 CN**: 结束当前词法作用域或复合语句块。
- **L414 EN**: Blank line separating nearby declarations or logic blocks.
  **L414 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u64x2_const(uint64_t __c0,`.
  **L415 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u64x2_const(uint64_t __c0,`。
- **L416 EN**: Continues the surrounding expression or declaration: `uint64_t __c1)`.
  **L416 CN**: 继续构造周围的表达式或声明：`uint64_t __c1)`。
- **L417 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c0) __REQUIRE_CONSTANT(__c1) {`.
  **L417 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c0) __REQUIRE_CONSTANT(__c1) {`。
- **L418 EN**: Returns from the current function with `(v128_t)(__u64x2){__c0, __c1}`.
  **L418 CN**: 以 `(v128_t)(__u64x2){__c0, __c1}` 从当前函数返回。
- **L419 EN**: Closes the current lexical scope or compound statement.
  **L419 CN**: 结束当前词法作用域或复合语句块。
- **L420 EN**: Blank line separating nearby declarations or logic blocks.
  **L420 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L421 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L421 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L422 EN**: Continues logic associated with callable symbol `wasm_f32x4_const`.
  **L422 CN**: 继续与可调用符号 `wasm_f32x4_const` 相关的逻辑。
- **L423 EN**: Continues logic associated with callable symbol `__REQUIRE_CONSTANT`.
  **L423 CN**: 继续与可调用符号 `__REQUIRE_CONSTANT` 相关的逻辑。
- **L424 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c3) {`.
  **L424 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c3) {`。
- **L425 EN**: Returns from the current function with `(v128_t)(__f32x4){__c0, __c1, __c2, __c3}`.
  **L425 CN**: 以 `(v128_t)(__f32x4){__c0, __c1, __c2, __c3}` 从当前函数返回。
- **L426 EN**: Closes the current lexical scope or compound statement.
  **L426 CN**: 结束当前词法作用域或复合语句块。
- **L427 EN**: Blank line separating nearby declarations or logic blocks.
  **L427 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L428 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_const(double __c0,`.
  **L428 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_const(double __c0,`。
- **L429 EN**: Continues the surrounding expression or declaration: `double __c1)`.
  **L429 CN**: 继续构造周围的表达式或声明：`double __c1)`。
- **L430 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c0) __REQUIRE_CONSTANT(__c1) {`.
  **L430 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c0) __REQUIRE_CONSTANT(__c1) {`。
- **L431 EN**: Returns from the current function with `(v128_t)(__f64x2){__c0, __c1}`.
  **L431 CN**: 以 `(v128_t)(__f64x2){__c0, __c1}` 从当前函数返回。
- **L432 EN**: Closes the current lexical scope or compound statement.
  **L432 CN**: 结束当前词法作用域或复合语句块。

### Lines 433-456

````c

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_const_splat(int8_t __c)
    __REQUIRE_CONSTANT(__c) {
  return (v128_t)(__i8x16){__c, __c, __c, __c, __c, __c, __c, __c,
                           __c, __c, __c, __c, __c, __c, __c, __c};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_const_splat(uint8_t __c)
    __REQUIRE_CONSTANT(__c) {
  return (v128_t)(__u8x16){__c, __c, __c, __c, __c, __c, __c, __c,
                           __c, __c, __c, __c, __c, __c, __c, __c};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_const_splat(int16_t __c)
    __REQUIRE_CONSTANT(__c) {
  return (v128_t)(__i16x8){__c, __c, __c, __c, __c, __c, __c, __c};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_const_splat(uint16_t __c)
    __REQUIRE_CONSTANT(__c) {
  return (v128_t)(__u16x8){__c, __c, __c, __c, __c, __c, __c, __c};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_const_splat(int32_t __c)
````
- **L433 EN**: Blank line separating nearby declarations or logic blocks.
  **L433 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L434 EN**: Continues logic associated with callable symbol `wasm_i8x16_const_splat`.
  **L434 CN**: 继续与可调用符号 `wasm_i8x16_const_splat` 相关的逻辑。
- **L435 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c) {`.
  **L435 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c) {`。
- **L436 EN**: Returns from the current function with `(v128_t)(__i8x16){__c, __c, __c, __c, __c, __c, __c, __c,`.
  **L436 CN**: 以 `(v128_t)(__i8x16){__c, __c, __c, __c, __c, __c, __c, __c,` 从当前函数返回。
- **L437 EN**: Adds a standalone statement or declaration: `__c, __c, __c, __c, __c, __c, __c, __c};`.
  **L437 CN**: 添加一条独立语句或声明：`__c, __c, __c, __c, __c, __c, __c, __c};`。
- **L438 EN**: Closes the current lexical scope or compound statement.
  **L438 CN**: 结束当前词法作用域或复合语句块。
- **L439 EN**: Blank line separating nearby declarations or logic blocks.
  **L439 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L440 EN**: Continues logic associated with callable symbol `wasm_u8x16_const_splat`.
  **L440 CN**: 继续与可调用符号 `wasm_u8x16_const_splat` 相关的逻辑。
- **L441 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c) {`.
  **L441 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c) {`。
- **L442 EN**: Returns from the current function with `(v128_t)(__u8x16){__c, __c, __c, __c, __c, __c, __c, __c,`.
  **L442 CN**: 以 `(v128_t)(__u8x16){__c, __c, __c, __c, __c, __c, __c, __c,` 从当前函数返回。
- **L443 EN**: Adds a standalone statement or declaration: `__c, __c, __c, __c, __c, __c, __c, __c};`.
  **L443 CN**: 添加一条独立语句或声明：`__c, __c, __c, __c, __c, __c, __c, __c};`。
- **L444 EN**: Closes the current lexical scope or compound statement.
  **L444 CN**: 结束当前词法作用域或复合语句块。
- **L445 EN**: Blank line separating nearby declarations or logic blocks.
  **L445 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L446 EN**: Continues logic associated with callable symbol `wasm_i16x8_const_splat`.
  **L446 CN**: 继续与可调用符号 `wasm_i16x8_const_splat` 相关的逻辑。
- **L447 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c) {`.
  **L447 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c) {`。
- **L448 EN**: Returns from the current function with `(v128_t)(__i16x8){__c, __c, __c, __c, __c, __c, __c, __c}`.
  **L448 CN**: 以 `(v128_t)(__i16x8){__c, __c, __c, __c, __c, __c, __c, __c}` 从当前函数返回。
- **L449 EN**: Closes the current lexical scope or compound statement.
  **L449 CN**: 结束当前词法作用域或复合语句块。
- **L450 EN**: Blank line separating nearby declarations or logic blocks.
  **L450 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L451 EN**: Continues logic associated with callable symbol `wasm_u16x8_const_splat`.
  **L451 CN**: 继续与可调用符号 `wasm_u16x8_const_splat` 相关的逻辑。
- **L452 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c) {`.
  **L452 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c) {`。
- **L453 EN**: Returns from the current function with `(v128_t)(__u16x8){__c, __c, __c, __c, __c, __c, __c, __c}`.
  **L453 CN**: 以 `(v128_t)(__u16x8){__c, __c, __c, __c, __c, __c, __c, __c}` 从当前函数返回。
- **L454 EN**: Closes the current lexical scope or compound statement.
  **L454 CN**: 结束当前词法作用域或复合语句块。
- **L455 EN**: Blank line separating nearby declarations or logic blocks.
  **L455 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L456 EN**: Continues logic associated with callable symbol `wasm_i32x4_const_splat`.
  **L456 CN**: 继续与可调用符号 `wasm_i32x4_const_splat` 相关的逻辑。

### Lines 457-480

````c
    __REQUIRE_CONSTANT(__c) {
  return (v128_t)(__i32x4){__c, __c, __c, __c};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_const_splat(uint32_t __c)
    __REQUIRE_CONSTANT(__c) {
  return (v128_t)(__u32x4){__c, __c, __c, __c};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_const_splat(int64_t __c)
    __REQUIRE_CONSTANT(__c) {
  return (v128_t)(__i64x2){__c, __c};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u64x2_const_splat(uint64_t __c)
    __REQUIRE_CONSTANT(__c) {
  return (v128_t)(__u64x2){__c, __c};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_const_splat(float __c)
    __REQUIRE_CONSTANT(__c) {
  return (v128_t)(__f32x4){__c, __c, __c, __c};
}

````
- **L457 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c) {`.
  **L457 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c) {`。
- **L458 EN**: Returns from the current function with `(v128_t)(__i32x4){__c, __c, __c, __c}`.
  **L458 CN**: 以 `(v128_t)(__i32x4){__c, __c, __c, __c}` 从当前函数返回。
- **L459 EN**: Closes the current lexical scope or compound statement.
  **L459 CN**: 结束当前词法作用域或复合语句块。
- **L460 EN**: Blank line separating nearby declarations or logic blocks.
  **L460 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L461 EN**: Continues logic associated with callable symbol `wasm_u32x4_const_splat`.
  **L461 CN**: 继续与可调用符号 `wasm_u32x4_const_splat` 相关的逻辑。
- **L462 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c) {`.
  **L462 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c) {`。
- **L463 EN**: Returns from the current function with `(v128_t)(__u32x4){__c, __c, __c, __c}`.
  **L463 CN**: 以 `(v128_t)(__u32x4){__c, __c, __c, __c}` 从当前函数返回。
- **L464 EN**: Closes the current lexical scope or compound statement.
  **L464 CN**: 结束当前词法作用域或复合语句块。
- **L465 EN**: Blank line separating nearby declarations or logic blocks.
  **L465 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L466 EN**: Continues logic associated with callable symbol `wasm_i64x2_const_splat`.
  **L466 CN**: 继续与可调用符号 `wasm_i64x2_const_splat` 相关的逻辑。
- **L467 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c) {`.
  **L467 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c) {`。
- **L468 EN**: Returns from the current function with `(v128_t)(__i64x2){__c, __c}`.
  **L468 CN**: 以 `(v128_t)(__i64x2){__c, __c}` 从当前函数返回。
- **L469 EN**: Closes the current lexical scope or compound statement.
  **L469 CN**: 结束当前词法作用域或复合语句块。
- **L470 EN**: Blank line separating nearby declarations or logic blocks.
  **L470 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L471 EN**: Continues logic associated with callable symbol `wasm_u64x2_const_splat`.
  **L471 CN**: 继续与可调用符号 `wasm_u64x2_const_splat` 相关的逻辑。
- **L472 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c) {`.
  **L472 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c) {`。
- **L473 EN**: Returns from the current function with `(v128_t)(__u64x2){__c, __c}`.
  **L473 CN**: 以 `(v128_t)(__u64x2){__c, __c}` 从当前函数返回。
- **L474 EN**: Closes the current lexical scope or compound statement.
  **L474 CN**: 结束当前词法作用域或复合语句块。
- **L475 EN**: Blank line separating nearby declarations or logic blocks.
  **L475 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L476 EN**: Continues logic associated with callable symbol `wasm_f32x4_const_splat`.
  **L476 CN**: 继续与可调用符号 `wasm_f32x4_const_splat` 相关的逻辑。
- **L477 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c) {`.
  **L477 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c) {`。
- **L478 EN**: Returns from the current function with `(v128_t)(__f32x4){__c, __c, __c, __c}`.
  **L478 CN**: 以 `(v128_t)(__f32x4){__c, __c, __c, __c}` 从当前函数返回。
- **L479 EN**: Closes the current lexical scope or compound statement.
  **L479 CN**: 结束当前词法作用域或复合语句块。
- **L480 EN**: Blank line separating nearby declarations or logic blocks.
  **L480 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 481-504

````c
static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_const_splat(double __c)
    __REQUIRE_CONSTANT(__c) {
  return (v128_t)(__f64x2){__c, __c};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_splat(int8_t __a) {
  return (v128_t)(__i8x16){__a, __a, __a, __a, __a, __a, __a, __a,
                           __a, __a, __a, __a, __a, __a, __a, __a};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_splat(uint8_t __a) {
  return (v128_t)(__u8x16){__a, __a, __a, __a, __a, __a, __a, __a,
                           __a, __a, __a, __a, __a, __a, __a, __a};
}

static __inline__ int8_t __DEFAULT_FN_ATTRS wasm_i8x16_extract_lane(v128_t __a,
                                                                    int __i)
    __REQUIRE_CONSTANT(__i) {
  return ((__i8x16)__a)[__i];
}

static __inline__ uint8_t __DEFAULT_FN_ATTRS wasm_u8x16_extract_lane(v128_t __a,
                                                                     int __i)
    __REQUIRE_CONSTANT(__i) {
````
- **L481 EN**: Continues logic associated with callable symbol `wasm_f64x2_const_splat`.
  **L481 CN**: 继续与可调用符号 `wasm_f64x2_const_splat` 相关的逻辑。
- **L482 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__c) {`.
  **L482 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__c) {`。
- **L483 EN**: Returns from the current function with `(v128_t)(__f64x2){__c, __c}`.
  **L483 CN**: 以 `(v128_t)(__f64x2){__c, __c}` 从当前函数返回。
- **L484 EN**: Closes the current lexical scope or compound statement.
  **L484 CN**: 结束当前词法作用域或复合语句块。
- **L485 EN**: Blank line separating nearby declarations or logic blocks.
  **L485 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L486 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_splat(int8_t __a) {`.
  **L486 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_splat(int8_t __a) {`。
- **L487 EN**: Returns from the current function with `(v128_t)(__i8x16){__a, __a, __a, __a, __a, __a, __a, __a,`.
  **L487 CN**: 以 `(v128_t)(__i8x16){__a, __a, __a, __a, __a, __a, __a, __a,` 从当前函数返回。
- **L488 EN**: Adds a standalone statement or declaration: `__a, __a, __a, __a, __a, __a, __a, __a};`.
  **L488 CN**: 添加一条独立语句或声明：`__a, __a, __a, __a, __a, __a, __a, __a};`。
- **L489 EN**: Closes the current lexical scope or compound statement.
  **L489 CN**: 结束当前词法作用域或复合语句块。
- **L490 EN**: Blank line separating nearby declarations or logic blocks.
  **L490 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L491 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_splat(uint8_t __a) {`.
  **L491 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_splat(uint8_t __a) {`。
- **L492 EN**: Returns from the current function with `(v128_t)(__u8x16){__a, __a, __a, __a, __a, __a, __a, __a,`.
  **L492 CN**: 以 `(v128_t)(__u8x16){__a, __a, __a, __a, __a, __a, __a, __a,` 从当前函数返回。
- **L493 EN**: Adds a standalone statement or declaration: `__a, __a, __a, __a, __a, __a, __a, __a};`.
  **L493 CN**: 添加一条独立语句或声明：`__a, __a, __a, __a, __a, __a, __a, __a};`。
- **L494 EN**: Closes the current lexical scope or compound statement.
  **L494 CN**: 结束当前词法作用域或复合语句块。
- **L495 EN**: Blank line separating nearby declarations or logic blocks.
  **L495 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int8_t __DEFAULT_FN_ATTRS wasm_i8x16_extract_lane(v128_t __a,`.
  **L496 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int8_t __DEFAULT_FN_ATTRS wasm_i8x16_extract_lane(v128_t __a,`。
- **L497 EN**: Continues the surrounding expression or declaration: `int __i)`.
  **L497 CN**: 继续构造周围的表达式或声明：`int __i)`。
- **L498 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L498 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L499 EN**: Returns from the current function with `((__i8x16)__a)[__i]`.
  **L499 CN**: 以 `((__i8x16)__a)[__i]` 从当前函数返回。
- **L500 EN**: Closes the current lexical scope or compound statement.
  **L500 CN**: 结束当前词法作用域或复合语句块。
- **L501 EN**: Blank line separating nearby declarations or logic blocks.
  **L501 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ uint8_t __DEFAULT_FN_ATTRS wasm_u8x16_extract_lane(v128_t __a,`.
  **L502 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ uint8_t __DEFAULT_FN_ATTRS wasm_u8x16_extract_lane(v128_t __a,`。
- **L503 EN**: Continues the surrounding expression or declaration: `int __i)`.
  **L503 CN**: 继续构造周围的表达式或声明：`int __i)`。
- **L504 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L504 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。

### Lines 505-528

````c
  return ((__u8x16)__a)[__i];
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_replace_lane(v128_t __a,
                                                                    int __i,
                                                                    int8_t __b)
    __REQUIRE_CONSTANT(__i) {
  __i8x16 __v = (__i8x16)__a;
  __v[__i] = __b;
  return (v128_t)__v;
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_replace_lane(v128_t __a,
                                                                    int __i,
                                                                    uint8_t __b)
    __REQUIRE_CONSTANT(__i) {
  __u8x16 __v = (__u8x16)__a;
  __v[__i] = __b;
  return (v128_t)__v;
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_splat(int16_t __a) {
  return (v128_t)(__i16x8){__a, __a, __a, __a, __a, __a, __a, __a};
}
````
- **L505 EN**: Returns from the current function with `((__u8x16)__a)[__i]`.
  **L505 CN**: 以 `((__u8x16)__a)[__i]` 从当前函数返回。
- **L506 EN**: Closes the current lexical scope or compound statement.
  **L506 CN**: 结束当前词法作用域或复合语句块。
- **L507 EN**: Blank line separating nearby declarations or logic blocks.
  **L507 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L508 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_replace_lane(v128_t __a,`.
  **L508 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_replace_lane(v128_t __a,`。
- **L509 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int __i,`.
  **L509 CN**: 继续一个多行参数列表、初始化器或聚合项：`int __i,`。
- **L510 EN**: Continues the surrounding expression or declaration: `int8_t __b)`.
  **L510 CN**: 继续构造周围的表达式或声明：`int8_t __b)`。
- **L511 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L511 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L512 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L512 CN**: 使用右侧表达式初始化变量 `__v`。
- **L513 EN**: Adds a standalone statement or declaration: `__v[__i] = __b;`.
  **L513 CN**: 添加一条独立语句或声明：`__v[__i] = __b;`。
- **L514 EN**: Returns from the current function with `(v128_t)__v`.
  **L514 CN**: 以 `(v128_t)__v` 从当前函数返回。
- **L515 EN**: Closes the current lexical scope or compound statement.
  **L515 CN**: 结束当前词法作用域或复合语句块。
- **L516 EN**: Blank line separating nearby declarations or logic blocks.
  **L516 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_replace_lane(v128_t __a,`.
  **L517 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_replace_lane(v128_t __a,`。
- **L518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int __i,`.
  **L518 CN**: 继续一个多行参数列表、初始化器或聚合项：`int __i,`。
- **L519 EN**: Continues the surrounding expression or declaration: `uint8_t __b)`.
  **L519 CN**: 继续构造周围的表达式或声明：`uint8_t __b)`。
- **L520 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L520 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L521 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L521 CN**: 使用右侧表达式初始化变量 `__v`。
- **L522 EN**: Adds a standalone statement or declaration: `__v[__i] = __b;`.
  **L522 CN**: 添加一条独立语句或声明：`__v[__i] = __b;`。
- **L523 EN**: Returns from the current function with `(v128_t)__v`.
  **L523 CN**: 以 `(v128_t)__v` 从当前函数返回。
- **L524 EN**: Closes the current lexical scope or compound statement.
  **L524 CN**: 结束当前词法作用域或复合语句块。
- **L525 EN**: Blank line separating nearby declarations or logic blocks.
  **L525 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L526 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_splat(int16_t __a) {`.
  **L526 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_splat(int16_t __a) {`。
- **L527 EN**: Returns from the current function with `(v128_t)(__i16x8){__a, __a, __a, __a, __a, __a, __a, __a}`.
  **L527 CN**: 以 `(v128_t)(__i16x8){__a, __a, __a, __a, __a, __a, __a, __a}` 从当前函数返回。
- **L528 EN**: Closes the current lexical scope or compound statement.
  **L528 CN**: 结束当前词法作用域或复合语句块。

### Lines 529-552

````c

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_splat(uint16_t __a) {
  return (v128_t)(__u16x8){__a, __a, __a, __a, __a, __a, __a, __a};
}

static __inline__ int16_t __DEFAULT_FN_ATTRS wasm_i16x8_extract_lane(v128_t __a,
                                                                     int __i)
    __REQUIRE_CONSTANT(__i) {
  return ((__i16x8)__a)[__i];
}

static __inline__ uint16_t __DEFAULT_FN_ATTRS
wasm_u16x8_extract_lane(v128_t __a, int __i) __REQUIRE_CONSTANT(__i) {
  return ((__u16x8)__a)[__i];
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_replace_lane(v128_t __a,
                                                                    int __i,
                                                                    int16_t __b)
    __REQUIRE_CONSTANT(__i) {
  __i16x8 __v = (__i16x8)__a;
  __v[__i] = __b;
  return (v128_t)__v;
}
````
- **L529 EN**: Blank line separating nearby declarations or logic blocks.
  **L529 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L530 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_splat(uint16_t __a) {`.
  **L530 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_splat(uint16_t __a) {`。
- **L531 EN**: Returns from the current function with `(v128_t)(__u16x8){__a, __a, __a, __a, __a, __a, __a, __a}`.
  **L531 CN**: 以 `(v128_t)(__u16x8){__a, __a, __a, __a, __a, __a, __a, __a}` 从当前函数返回。
- **L532 EN**: Closes the current lexical scope or compound statement.
  **L532 CN**: 结束当前词法作用域或复合语句块。
- **L533 EN**: Blank line separating nearby declarations or logic blocks.
  **L533 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int16_t __DEFAULT_FN_ATTRS wasm_i16x8_extract_lane(v128_t __a,`.
  **L534 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int16_t __DEFAULT_FN_ATTRS wasm_i16x8_extract_lane(v128_t __a,`。
- **L535 EN**: Continues the surrounding expression or declaration: `int __i)`.
  **L535 CN**: 继续构造周围的表达式或声明：`int __i)`。
- **L536 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L536 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L537 EN**: Returns from the current function with `((__i16x8)__a)[__i]`.
  **L537 CN**: 以 `((__i16x8)__a)[__i]` 从当前函数返回。
- **L538 EN**: Closes the current lexical scope or compound statement.
  **L538 CN**: 结束当前词法作用域或复合语句块。
- **L539 EN**: Blank line separating nearby declarations or logic blocks.
  **L539 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L540 EN**: Continues the surrounding expression or declaration: `static __inline__ uint16_t __DEFAULT_FN_ATTRS`.
  **L540 CN**: 继续构造周围的表达式或声明：`static __inline__ uint16_t __DEFAULT_FN_ATTRS`。
- **L541 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u16x8_extract_lane(v128_t __a, int __i) __REQUIRE_CONSTANT(__i) {`.
  **L541 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u16x8_extract_lane(v128_t __a, int __i) __REQUIRE_CONSTANT(__i) {`。
- **L542 EN**: Returns from the current function with `((__u16x8)__a)[__i]`.
  **L542 CN**: 以 `((__u16x8)__a)[__i]` 从当前函数返回。
- **L543 EN**: Closes the current lexical scope or compound statement.
  **L543 CN**: 结束当前词法作用域或复合语句块。
- **L544 EN**: Blank line separating nearby declarations or logic blocks.
  **L544 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L545 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_replace_lane(v128_t __a,`.
  **L545 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_replace_lane(v128_t __a,`。
- **L546 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int __i,`.
  **L546 CN**: 继续一个多行参数列表、初始化器或聚合项：`int __i,`。
- **L547 EN**: Continues the surrounding expression or declaration: `int16_t __b)`.
  **L547 CN**: 继续构造周围的表达式或声明：`int16_t __b)`。
- **L548 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L548 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L549 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L549 CN**: 使用右侧表达式初始化变量 `__v`。
- **L550 EN**: Adds a standalone statement or declaration: `__v[__i] = __b;`.
  **L550 CN**: 添加一条独立语句或声明：`__v[__i] = __b;`。
- **L551 EN**: Returns from the current function with `(v128_t)__v`.
  **L551 CN**: 以 `(v128_t)__v` 从当前函数返回。
- **L552 EN**: Closes the current lexical scope or compound statement.
  **L552 CN**: 结束当前词法作用域或复合语句块。

### Lines 553-576

````c

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_replace_lane(
    v128_t __a, int __i, uint16_t __b) __REQUIRE_CONSTANT(__i) {
  __u16x8 __v = (__u16x8)__a;
  __v[__i] = __b;
  return (v128_t)__v;
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_splat(int32_t __a) {
  return (v128_t)(__i32x4){__a, __a, __a, __a};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_splat(uint32_t __a) {
  return (v128_t)(__u32x4){__a, __a, __a, __a};
}

static __inline__ int32_t __DEFAULT_FN_ATTRS wasm_i32x4_extract_lane(v128_t __a,
                                                                     int __i)
    __REQUIRE_CONSTANT(__i) {
  return ((__i32x4)__a)[__i];
}

static __inline__ uint32_t __DEFAULT_FN_ATTRS
wasm_u32x4_extract_lane(v128_t __a, int __i) __REQUIRE_CONSTANT(__i) {
````
- **L553 EN**: Blank line separating nearby declarations or logic blocks.
  **L553 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L554 EN**: Continues logic associated with callable symbol `wasm_u16x8_replace_lane`.
  **L554 CN**: 继续与可调用符号 `wasm_u16x8_replace_lane` 相关的逻辑。
- **L555 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `v128_t __a, int __i, uint16_t __b) __REQUIRE_CONSTANT(__i) {`.
  **L555 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`v128_t __a, int __i, uint16_t __b) __REQUIRE_CONSTANT(__i) {`。
- **L556 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L556 CN**: 使用右侧表达式初始化变量 `__v`。
- **L557 EN**: Adds a standalone statement or declaration: `__v[__i] = __b;`.
  **L557 CN**: 添加一条独立语句或声明：`__v[__i] = __b;`。
- **L558 EN**: Returns from the current function with `(v128_t)__v`.
  **L558 CN**: 以 `(v128_t)__v` 从当前函数返回。
- **L559 EN**: Closes the current lexical scope or compound statement.
  **L559 CN**: 结束当前词法作用域或复合语句块。
- **L560 EN**: Blank line separating nearby declarations or logic blocks.
  **L560 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L561 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_splat(int32_t __a) {`.
  **L561 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_splat(int32_t __a) {`。
- **L562 EN**: Returns from the current function with `(v128_t)(__i32x4){__a, __a, __a, __a}`.
  **L562 CN**: 以 `(v128_t)(__i32x4){__a, __a, __a, __a}` 从当前函数返回。
- **L563 EN**: Closes the current lexical scope or compound statement.
  **L563 CN**: 结束当前词法作用域或复合语句块。
- **L564 EN**: Blank line separating nearby declarations or logic blocks.
  **L564 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L565 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_splat(uint32_t __a) {`.
  **L565 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_splat(uint32_t __a) {`。
- **L566 EN**: Returns from the current function with `(v128_t)(__u32x4){__a, __a, __a, __a}`.
  **L566 CN**: 以 `(v128_t)(__u32x4){__a, __a, __a, __a}` 从当前函数返回。
- **L567 EN**: Closes the current lexical scope or compound statement.
  **L567 CN**: 结束当前词法作用域或复合语句块。
- **L568 EN**: Blank line separating nearby declarations or logic blocks.
  **L568 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L569 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int32_t __DEFAULT_FN_ATTRS wasm_i32x4_extract_lane(v128_t __a,`.
  **L569 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int32_t __DEFAULT_FN_ATTRS wasm_i32x4_extract_lane(v128_t __a,`。
- **L570 EN**: Continues the surrounding expression or declaration: `int __i)`.
  **L570 CN**: 继续构造周围的表达式或声明：`int __i)`。
- **L571 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L571 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L572 EN**: Returns from the current function with `((__i32x4)__a)[__i]`.
  **L572 CN**: 以 `((__i32x4)__a)[__i]` 从当前函数返回。
- **L573 EN**: Closes the current lexical scope or compound statement.
  **L573 CN**: 结束当前词法作用域或复合语句块。
- **L574 EN**: Blank line separating nearby declarations or logic blocks.
  **L574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L575 EN**: Continues the surrounding expression or declaration: `static __inline__ uint32_t __DEFAULT_FN_ATTRS`.
  **L575 CN**: 继续构造周围的表达式或声明：`static __inline__ uint32_t __DEFAULT_FN_ATTRS`。
- **L576 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u32x4_extract_lane(v128_t __a, int __i) __REQUIRE_CONSTANT(__i) {`.
  **L576 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u32x4_extract_lane(v128_t __a, int __i) __REQUIRE_CONSTANT(__i) {`。

### Lines 577-600

````c
  return ((__u32x4)__a)[__i];
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_replace_lane(v128_t __a,
                                                                    int __i,
                                                                    int32_t __b)
    __REQUIRE_CONSTANT(__i) {
  __i32x4 __v = (__i32x4)__a;
  __v[__i] = __b;
  return (v128_t)__v;
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_replace_lane(
    v128_t __a, int __i, uint32_t __b) __REQUIRE_CONSTANT(__i) {
  __u32x4 __v = (__u32x4)__a;
  __v[__i] = __b;
  return (v128_t)__v;
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_splat(int64_t __a) {
  return (v128_t)(__i64x2){__a, __a};
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u64x2_splat(uint64_t __a) {
````
- **L577 EN**: Returns from the current function with `((__u32x4)__a)[__i]`.
  **L577 CN**: 以 `((__u32x4)__a)[__i]` 从当前函数返回。
- **L578 EN**: Closes the current lexical scope or compound statement.
  **L578 CN**: 结束当前词法作用域或复合语句块。
- **L579 EN**: Blank line separating nearby declarations or logic blocks.
  **L579 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_replace_lane(v128_t __a,`.
  **L580 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_replace_lane(v128_t __a,`。
- **L581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int __i,`.
  **L581 CN**: 继续一个多行参数列表、初始化器或聚合项：`int __i,`。
- **L582 EN**: Continues the surrounding expression or declaration: `int32_t __b)`.
  **L582 CN**: 继续构造周围的表达式或声明：`int32_t __b)`。
- **L583 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L583 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L584 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L584 CN**: 使用右侧表达式初始化变量 `__v`。
- **L585 EN**: Adds a standalone statement or declaration: `__v[__i] = __b;`.
  **L585 CN**: 添加一条独立语句或声明：`__v[__i] = __b;`。
- **L586 EN**: Returns from the current function with `(v128_t)__v`.
  **L586 CN**: 以 `(v128_t)__v` 从当前函数返回。
- **L587 EN**: Closes the current lexical scope or compound statement.
  **L587 CN**: 结束当前词法作用域或复合语句块。
- **L588 EN**: Blank line separating nearby declarations or logic blocks.
  **L588 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L589 EN**: Continues logic associated with callable symbol `wasm_u32x4_replace_lane`.
  **L589 CN**: 继续与可调用符号 `wasm_u32x4_replace_lane` 相关的逻辑。
- **L590 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `v128_t __a, int __i, uint32_t __b) __REQUIRE_CONSTANT(__i) {`.
  **L590 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`v128_t __a, int __i, uint32_t __b) __REQUIRE_CONSTANT(__i) {`。
- **L591 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L591 CN**: 使用右侧表达式初始化变量 `__v`。
- **L592 EN**: Adds a standalone statement or declaration: `__v[__i] = __b;`.
  **L592 CN**: 添加一条独立语句或声明：`__v[__i] = __b;`。
- **L593 EN**: Returns from the current function with `(v128_t)__v`.
  **L593 CN**: 以 `(v128_t)__v` 从当前函数返回。
- **L594 EN**: Closes the current lexical scope or compound statement.
  **L594 CN**: 结束当前词法作用域或复合语句块。
- **L595 EN**: Blank line separating nearby declarations or logic blocks.
  **L595 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L596 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_splat(int64_t __a) {`.
  **L596 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_splat(int64_t __a) {`。
- **L597 EN**: Returns from the current function with `(v128_t)(__i64x2){__a, __a}`.
  **L597 CN**: 以 `(v128_t)(__i64x2){__a, __a}` 从当前函数返回。
- **L598 EN**: Closes the current lexical scope or compound statement.
  **L598 CN**: 结束当前词法作用域或复合语句块。
- **L599 EN**: Blank line separating nearby declarations or logic blocks.
  **L599 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L600 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u64x2_splat(uint64_t __a) {`.
  **L600 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u64x2_splat(uint64_t __a) {`。

### Lines 601-624

````c
  return (v128_t)(__u64x2){__a, __a};
}

static __inline__ int64_t __DEFAULT_FN_ATTRS wasm_i64x2_extract_lane(v128_t __a,
                                                                     int __i)
    __REQUIRE_CONSTANT(__i) {
  return ((__i64x2)__a)[__i];
}

static __inline__ uint64_t __DEFAULT_FN_ATTRS
wasm_u64x2_extract_lane(v128_t __a, int __i) __REQUIRE_CONSTANT(__i) {
  return ((__u64x2)__a)[__i];
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_replace_lane(v128_t __a,
                                                                    int __i,
                                                                    int64_t __b)
    __REQUIRE_CONSTANT(__i) {
  __i64x2 __v = (__i64x2)__a;
  __v[__i] = __b;
  return (v128_t)__v;
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u64x2_replace_lane(
````
- **L601 EN**: Returns from the current function with `(v128_t)(__u64x2){__a, __a}`.
  **L601 CN**: 以 `(v128_t)(__u64x2){__a, __a}` 从当前函数返回。
- **L602 EN**: Closes the current lexical scope or compound statement.
  **L602 CN**: 结束当前词法作用域或复合语句块。
- **L603 EN**: Blank line separating nearby declarations or logic blocks.
  **L603 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ int64_t __DEFAULT_FN_ATTRS wasm_i64x2_extract_lane(v128_t __a,`.
  **L604 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ int64_t __DEFAULT_FN_ATTRS wasm_i64x2_extract_lane(v128_t __a,`。
- **L605 EN**: Continues the surrounding expression or declaration: `int __i)`.
  **L605 CN**: 继续构造周围的表达式或声明：`int __i)`。
- **L606 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L606 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L607 EN**: Returns from the current function with `((__i64x2)__a)[__i]`.
  **L607 CN**: 以 `((__i64x2)__a)[__i]` 从当前函数返回。
- **L608 EN**: Closes the current lexical scope or compound statement.
  **L608 CN**: 结束当前词法作用域或复合语句块。
- **L609 EN**: Blank line separating nearby declarations or logic blocks.
  **L609 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L610 EN**: Continues the surrounding expression or declaration: `static __inline__ uint64_t __DEFAULT_FN_ATTRS`.
  **L610 CN**: 继续构造周围的表达式或声明：`static __inline__ uint64_t __DEFAULT_FN_ATTRS`。
- **L611 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u64x2_extract_lane(v128_t __a, int __i) __REQUIRE_CONSTANT(__i) {`.
  **L611 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u64x2_extract_lane(v128_t __a, int __i) __REQUIRE_CONSTANT(__i) {`。
- **L612 EN**: Returns from the current function with `((__u64x2)__a)[__i]`.
  **L612 CN**: 以 `((__u64x2)__a)[__i]` 从当前函数返回。
- **L613 EN**: Closes the current lexical scope or compound statement.
  **L613 CN**: 结束当前词法作用域或复合语句块。
- **L614 EN**: Blank line separating nearby declarations or logic blocks.
  **L614 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_replace_lane(v128_t __a,`.
  **L615 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_replace_lane(v128_t __a,`。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int __i,`.
  **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`int __i,`。
- **L617 EN**: Continues the surrounding expression or declaration: `int64_t __b)`.
  **L617 CN**: 继续构造周围的表达式或声明：`int64_t __b)`。
- **L618 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L618 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L619 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L619 CN**: 使用右侧表达式初始化变量 `__v`。
- **L620 EN**: Adds a standalone statement or declaration: `__v[__i] = __b;`.
  **L620 CN**: 添加一条独立语句或声明：`__v[__i] = __b;`。
- **L621 EN**: Returns from the current function with `(v128_t)__v`.
  **L621 CN**: 以 `(v128_t)__v` 从当前函数返回。
- **L622 EN**: Closes the current lexical scope or compound statement.
  **L622 CN**: 结束当前词法作用域或复合语句块。
- **L623 EN**: Blank line separating nearby declarations or logic blocks.
  **L623 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L624 EN**: Continues logic associated with callable symbol `wasm_u64x2_replace_lane`.
  **L624 CN**: 继续与可调用符号 `wasm_u64x2_replace_lane` 相关的逻辑。

### Lines 625-648

````c
    v128_t __a, int __i, uint64_t __b) __REQUIRE_CONSTANT(__i) {
  __u64x2 __v = (__u64x2)__a;
  __v[__i] = __b;
  return (v128_t)__v;
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_splat(float __a) {
  return (v128_t)(__f32x4){__a, __a, __a, __a};
}

static __inline__ float __DEFAULT_FN_ATTRS wasm_f32x4_extract_lane(v128_t __a,
                                                                   int __i)
    __REQUIRE_CONSTANT(__i) {
  return ((__f32x4)__a)[__i];
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_replace_lane(v128_t __a,
                                                                    int __i,
                                                                    float __b)
    __REQUIRE_CONSTANT(__i) {
  __f32x4 __v = (__f32x4)__a;
  __v[__i] = __b;
  return (v128_t)__v;
}
````
- **L625 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `v128_t __a, int __i, uint64_t __b) __REQUIRE_CONSTANT(__i) {`.
  **L625 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`v128_t __a, int __i, uint64_t __b) __REQUIRE_CONSTANT(__i) {`。
- **L626 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L626 CN**: 使用右侧表达式初始化变量 `__v`。
- **L627 EN**: Adds a standalone statement or declaration: `__v[__i] = __b;`.
  **L627 CN**: 添加一条独立语句或声明：`__v[__i] = __b;`。
- **L628 EN**: Returns from the current function with `(v128_t)__v`.
  **L628 CN**: 以 `(v128_t)__v` 从当前函数返回。
- **L629 EN**: Closes the current lexical scope or compound statement.
  **L629 CN**: 结束当前词法作用域或复合语句块。
- **L630 EN**: Blank line separating nearby declarations or logic blocks.
  **L630 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L631 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_splat(float __a) {`.
  **L631 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_splat(float __a) {`。
- **L632 EN**: Returns from the current function with `(v128_t)(__f32x4){__a, __a, __a, __a}`.
  **L632 CN**: 以 `(v128_t)(__f32x4){__a, __a, __a, __a}` 从当前函数返回。
- **L633 EN**: Closes the current lexical scope or compound statement.
  **L633 CN**: 结束当前词法作用域或复合语句块。
- **L634 EN**: Blank line separating nearby declarations or logic blocks.
  **L634 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ float __DEFAULT_FN_ATTRS wasm_f32x4_extract_lane(v128_t __a,`.
  **L635 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ float __DEFAULT_FN_ATTRS wasm_f32x4_extract_lane(v128_t __a,`。
- **L636 EN**: Continues the surrounding expression or declaration: `int __i)`.
  **L636 CN**: 继续构造周围的表达式或声明：`int __i)`。
- **L637 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L637 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L638 EN**: Returns from the current function with `((__f32x4)__a)[__i]`.
  **L638 CN**: 以 `((__f32x4)__a)[__i]` 从当前函数返回。
- **L639 EN**: Closes the current lexical scope or compound statement.
  **L639 CN**: 结束当前词法作用域或复合语句块。
- **L640 EN**: Blank line separating nearby declarations or logic blocks.
  **L640 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_replace_lane(v128_t __a,`.
  **L641 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_replace_lane(v128_t __a,`。
- **L642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int __i,`.
  **L642 CN**: 继续一个多行参数列表、初始化器或聚合项：`int __i,`。
- **L643 EN**: Continues the surrounding expression or declaration: `float __b)`.
  **L643 CN**: 继续构造周围的表达式或声明：`float __b)`。
- **L644 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L644 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L645 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L645 CN**: 使用右侧表达式初始化变量 `__v`。
- **L646 EN**: Adds a standalone statement or declaration: `__v[__i] = __b;`.
  **L646 CN**: 添加一条独立语句或声明：`__v[__i] = __b;`。
- **L647 EN**: Returns from the current function with `(v128_t)__v`.
  **L647 CN**: 以 `(v128_t)__v` 从当前函数返回。
- **L648 EN**: Closes the current lexical scope or compound statement.
  **L648 CN**: 结束当前词法作用域或复合语句块。

### Lines 649-672

````c

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_splat(double __a) {
  return (v128_t)(__f64x2){__a, __a};
}

static __inline__ double __DEFAULT_FN_ATTRS wasm_f64x2_extract_lane(v128_t __a,
                                                                    int __i)
    __REQUIRE_CONSTANT(__i) {
  return ((__f64x2)__a)[__i];
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_replace_lane(v128_t __a,
                                                                    int __i,
                                                                    double __b)
    __REQUIRE_CONSTANT(__i) {
  __f64x2 __v = (__f64x2)__a;
  __v[__i] = __b;
  return (v128_t)__v;
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_eq(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i8x16)__a == (__i8x16)__b);
}
````
- **L649 EN**: Blank line separating nearby declarations or logic blocks.
  **L649 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L650 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_splat(double __a) {`.
  **L650 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_splat(double __a) {`。
- **L651 EN**: Returns from the current function with `(v128_t)(__f64x2){__a, __a}`.
  **L651 CN**: 以 `(v128_t)(__f64x2){__a, __a}` 从当前函数返回。
- **L652 EN**: Closes the current lexical scope or compound statement.
  **L652 CN**: 结束当前词法作用域或复合语句块。
- **L653 EN**: Blank line separating nearby declarations or logic blocks.
  **L653 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L654 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ double __DEFAULT_FN_ATTRS wasm_f64x2_extract_lane(v128_t __a,`.
  **L654 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ double __DEFAULT_FN_ATTRS wasm_f64x2_extract_lane(v128_t __a,`。
- **L655 EN**: Continues the surrounding expression or declaration: `int __i)`.
  **L655 CN**: 继续构造周围的表达式或声明：`int __i)`。
- **L656 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L656 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L657 EN**: Returns from the current function with `((__f64x2)__a)[__i]`.
  **L657 CN**: 以 `((__f64x2)__a)[__i]` 从当前函数返回。
- **L658 EN**: Closes the current lexical scope or compound statement.
  **L658 CN**: 结束当前词法作用域或复合语句块。
- **L659 EN**: Blank line separating nearby declarations or logic blocks.
  **L659 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L660 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_replace_lane(v128_t __a,`.
  **L660 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_replace_lane(v128_t __a,`。
- **L661 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int __i,`.
  **L661 CN**: 继续一个多行参数列表、初始化器或聚合项：`int __i,`。
- **L662 EN**: Continues the surrounding expression or declaration: `double __b)`.
  **L662 CN**: 继续构造周围的表达式或声明：`double __b)`。
- **L663 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__REQUIRE_CONSTANT(__i) {`.
  **L663 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__REQUIRE_CONSTANT(__i) {`。
- **L664 EN**: Initializes variable `__v` from the expression on the right-hand side.
  **L664 CN**: 使用右侧表达式初始化变量 `__v`。
- **L665 EN**: Adds a standalone statement or declaration: `__v[__i] = __b;`.
  **L665 CN**: 添加一条独立语句或声明：`__v[__i] = __b;`。
- **L666 EN**: Returns from the current function with `(v128_t)__v`.
  **L666 CN**: 以 `(v128_t)__v` 从当前函数返回。
- **L667 EN**: Closes the current lexical scope or compound statement.
  **L667 CN**: 结束当前词法作用域或复合语句块。
- **L668 EN**: Blank line separating nearby declarations or logic blocks.
  **L668 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L669 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_eq(v128_t __a,`.
  **L669 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_eq(v128_t __a,`。
- **L670 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L670 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L671 EN**: Returns from the current function with `(v128_t)((__i8x16)__a == (__i8x16)__b)`.
  **L671 CN**: 以 `(v128_t)((__i8x16)__a == (__i8x16)__b)` 从当前函数返回。
- **L672 EN**: Closes the current lexical scope or compound statement.
  **L672 CN**: 结束当前词法作用域或复合语句块。

### Lines 673-696

````c

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_ne(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i8x16)__a != (__i8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_lt(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i8x16)__a < (__i8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_lt(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__u8x16)__a < (__u8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_gt(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i8x16)__a > (__i8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_gt(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__u8x16)__a > (__u8x16)__b);
````
- **L673 EN**: Blank line separating nearby declarations or logic blocks.
  **L673 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L674 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_ne(v128_t __a,`.
  **L674 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_ne(v128_t __a,`。
- **L675 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L675 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L676 EN**: Returns from the current function with `(v128_t)((__i8x16)__a != (__i8x16)__b)`.
  **L676 CN**: 以 `(v128_t)((__i8x16)__a != (__i8x16)__b)` 从当前函数返回。
- **L677 EN**: Closes the current lexical scope or compound statement.
  **L677 CN**: 结束当前词法作用域或复合语句块。
- **L678 EN**: Blank line separating nearby declarations or logic blocks.
  **L678 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L679 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_lt(v128_t __a,`.
  **L679 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_lt(v128_t __a,`。
- **L680 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L680 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L681 EN**: Returns from the current function with `(v128_t)((__i8x16)__a < (__i8x16)__b)`.
  **L681 CN**: 以 `(v128_t)((__i8x16)__a < (__i8x16)__b)` 从当前函数返回。
- **L682 EN**: Closes the current lexical scope or compound statement.
  **L682 CN**: 结束当前词法作用域或复合语句块。
- **L683 EN**: Blank line separating nearby declarations or logic blocks.
  **L683 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L684 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_lt(v128_t __a,`.
  **L684 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_lt(v128_t __a,`。
- **L685 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L685 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L686 EN**: Returns from the current function with `(v128_t)((__u8x16)__a < (__u8x16)__b)`.
  **L686 CN**: 以 `(v128_t)((__u8x16)__a < (__u8x16)__b)` 从当前函数返回。
- **L687 EN**: Closes the current lexical scope or compound statement.
  **L687 CN**: 结束当前词法作用域或复合语句块。
- **L688 EN**: Blank line separating nearby declarations or logic blocks.
  **L688 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L689 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_gt(v128_t __a,`.
  **L689 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_gt(v128_t __a,`。
- **L690 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L690 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L691 EN**: Returns from the current function with `(v128_t)((__i8x16)__a > (__i8x16)__b)`.
  **L691 CN**: 以 `(v128_t)((__i8x16)__a > (__i8x16)__b)` 从当前函数返回。
- **L692 EN**: Closes the current lexical scope or compound statement.
  **L692 CN**: 结束当前词法作用域或复合语句块。
- **L693 EN**: Blank line separating nearby declarations or logic blocks.
  **L693 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_gt(v128_t __a,`.
  **L694 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_gt(v128_t __a,`。
- **L695 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L695 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L696 EN**: Returns from the current function with `(v128_t)((__u8x16)__a > (__u8x16)__b)`.
  **L696 CN**: 以 `(v128_t)((__u8x16)__a > (__u8x16)__b)` 从当前函数返回。

### Lines 697-720

````c
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_le(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i8x16)__a <= (__i8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_le(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__u8x16)__a <= (__u8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_ge(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i8x16)__a >= (__i8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_ge(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__u8x16)__a >= (__u8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_eq(v128_t __a,
                                                          v128_t __b) {
````
- **L697 EN**: Closes the current lexical scope or compound statement.
  **L697 CN**: 结束当前词法作用域或复合语句块。
- **L698 EN**: Blank line separating nearby declarations or logic blocks.
  **L698 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L699 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_le(v128_t __a,`.
  **L699 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_le(v128_t __a,`。
- **L700 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L700 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L701 EN**: Returns from the current function with `(v128_t)((__i8x16)__a <= (__i8x16)__b)`.
  **L701 CN**: 以 `(v128_t)((__i8x16)__a <= (__i8x16)__b)` 从当前函数返回。
- **L702 EN**: Closes the current lexical scope or compound statement.
  **L702 CN**: 结束当前词法作用域或复合语句块。
- **L703 EN**: Blank line separating nearby declarations or logic blocks.
  **L703 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L704 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_le(v128_t __a,`.
  **L704 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_le(v128_t __a,`。
- **L705 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L705 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L706 EN**: Returns from the current function with `(v128_t)((__u8x16)__a <= (__u8x16)__b)`.
  **L706 CN**: 以 `(v128_t)((__u8x16)__a <= (__u8x16)__b)` 从当前函数返回。
- **L707 EN**: Closes the current lexical scope or compound statement.
  **L707 CN**: 结束当前词法作用域或复合语句块。
- **L708 EN**: Blank line separating nearby declarations or logic blocks.
  **L708 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L709 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_ge(v128_t __a,`.
  **L709 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_ge(v128_t __a,`。
- **L710 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L710 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L711 EN**: Returns from the current function with `(v128_t)((__i8x16)__a >= (__i8x16)__b)`.
  **L711 CN**: 以 `(v128_t)((__i8x16)__a >= (__i8x16)__b)` 从当前函数返回。
- **L712 EN**: Closes the current lexical scope or compound statement.
  **L712 CN**: 结束当前词法作用域或复合语句块。
- **L713 EN**: Blank line separating nearby declarations or logic blocks.
  **L713 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L714 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_ge(v128_t __a,`.
  **L714 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_ge(v128_t __a,`。
- **L715 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L715 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L716 EN**: Returns from the current function with `(v128_t)((__u8x16)__a >= (__u8x16)__b)`.
  **L716 CN**: 以 `(v128_t)((__u8x16)__a >= (__u8x16)__b)` 从当前函数返回。
- **L717 EN**: Closes the current lexical scope or compound statement.
  **L717 CN**: 结束当前词法作用域或复合语句块。
- **L718 EN**: Blank line separating nearby declarations or logic blocks.
  **L718 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L719 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_eq(v128_t __a,`.
  **L719 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_eq(v128_t __a,`。
- **L720 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L720 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。

### Lines 721-744

````c
  return (v128_t)((__i16x8)__a == (__i16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_ne(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__u16x8)__a != (__u16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_lt(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i16x8)__a < (__i16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_lt(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__u16x8)__a < (__u16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_gt(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i16x8)__a > (__i16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_gt(v128_t __a,
````
- **L721 EN**: Returns from the current function with `(v128_t)((__i16x8)__a == (__i16x8)__b)`.
  **L721 CN**: 以 `(v128_t)((__i16x8)__a == (__i16x8)__b)` 从当前函数返回。
- **L722 EN**: Closes the current lexical scope or compound statement.
  **L722 CN**: 结束当前词法作用域或复合语句块。
- **L723 EN**: Blank line separating nearby declarations or logic blocks.
  **L723 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_ne(v128_t __a,`.
  **L724 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_ne(v128_t __a,`。
- **L725 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L725 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L726 EN**: Returns from the current function with `(v128_t)((__u16x8)__a != (__u16x8)__b)`.
  **L726 CN**: 以 `(v128_t)((__u16x8)__a != (__u16x8)__b)` 从当前函数返回。
- **L727 EN**: Closes the current lexical scope or compound statement.
  **L727 CN**: 结束当前词法作用域或复合语句块。
- **L728 EN**: Blank line separating nearby declarations or logic blocks.
  **L728 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L729 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_lt(v128_t __a,`.
  **L729 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_lt(v128_t __a,`。
- **L730 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L730 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L731 EN**: Returns from the current function with `(v128_t)((__i16x8)__a < (__i16x8)__b)`.
  **L731 CN**: 以 `(v128_t)((__i16x8)__a < (__i16x8)__b)` 从当前函数返回。
- **L732 EN**: Closes the current lexical scope or compound statement.
  **L732 CN**: 结束当前词法作用域或复合语句块。
- **L733 EN**: Blank line separating nearby declarations or logic blocks.
  **L733 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L734 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_lt(v128_t __a,`.
  **L734 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_lt(v128_t __a,`。
- **L735 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L735 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L736 EN**: Returns from the current function with `(v128_t)((__u16x8)__a < (__u16x8)__b)`.
  **L736 CN**: 以 `(v128_t)((__u16x8)__a < (__u16x8)__b)` 从当前函数返回。
- **L737 EN**: Closes the current lexical scope or compound statement.
  **L737 CN**: 结束当前词法作用域或复合语句块。
- **L738 EN**: Blank line separating nearby declarations or logic blocks.
  **L738 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L739 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_gt(v128_t __a,`.
  **L739 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_gt(v128_t __a,`。
- **L740 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L740 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L741 EN**: Returns from the current function with `(v128_t)((__i16x8)__a > (__i16x8)__b)`.
  **L741 CN**: 以 `(v128_t)((__i16x8)__a > (__i16x8)__b)` 从当前函数返回。
- **L742 EN**: Closes the current lexical scope or compound statement.
  **L742 CN**: 结束当前词法作用域或复合语句块。
- **L743 EN**: Blank line separating nearby declarations or logic blocks.
  **L743 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_gt(v128_t __a,`.
  **L744 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_gt(v128_t __a,`。

### Lines 745-768

````c
                                                          v128_t __b) {
  return (v128_t)((__u16x8)__a > (__u16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_le(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i16x8)__a <= (__i16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_le(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__u16x8)__a <= (__u16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_ge(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i16x8)__a >= (__i16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_ge(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__u16x8)__a >= (__u16x8)__b);
}

````
- **L745 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L745 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L746 EN**: Returns from the current function with `(v128_t)((__u16x8)__a > (__u16x8)__b)`.
  **L746 CN**: 以 `(v128_t)((__u16x8)__a > (__u16x8)__b)` 从当前函数返回。
- **L747 EN**: Closes the current lexical scope or compound statement.
  **L747 CN**: 结束当前词法作用域或复合语句块。
- **L748 EN**: Blank line separating nearby declarations or logic blocks.
  **L748 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_le(v128_t __a,`.
  **L749 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_le(v128_t __a,`。
- **L750 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L750 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L751 EN**: Returns from the current function with `(v128_t)((__i16x8)__a <= (__i16x8)__b)`.
  **L751 CN**: 以 `(v128_t)((__i16x8)__a <= (__i16x8)__b)` 从当前函数返回。
- **L752 EN**: Closes the current lexical scope or compound statement.
  **L752 CN**: 结束当前词法作用域或复合语句块。
- **L753 EN**: Blank line separating nearby declarations or logic blocks.
  **L753 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L754 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_le(v128_t __a,`.
  **L754 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_le(v128_t __a,`。
- **L755 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L755 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L756 EN**: Returns from the current function with `(v128_t)((__u16x8)__a <= (__u16x8)__b)`.
  **L756 CN**: 以 `(v128_t)((__u16x8)__a <= (__u16x8)__b)` 从当前函数返回。
- **L757 EN**: Closes the current lexical scope or compound statement.
  **L757 CN**: 结束当前词法作用域或复合语句块。
- **L758 EN**: Blank line separating nearby declarations or logic blocks.
  **L758 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_ge(v128_t __a,`.
  **L759 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_ge(v128_t __a,`。
- **L760 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L760 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L761 EN**: Returns from the current function with `(v128_t)((__i16x8)__a >= (__i16x8)__b)`.
  **L761 CN**: 以 `(v128_t)((__i16x8)__a >= (__i16x8)__b)` 从当前函数返回。
- **L762 EN**: Closes the current lexical scope or compound statement.
  **L762 CN**: 结束当前词法作用域或复合语句块。
- **L763 EN**: Blank line separating nearby declarations or logic blocks.
  **L763 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_ge(v128_t __a,`.
  **L764 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_ge(v128_t __a,`。
- **L765 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L765 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L766 EN**: Returns from the current function with `(v128_t)((__u16x8)__a >= (__u16x8)__b)`.
  **L766 CN**: 以 `(v128_t)((__u16x8)__a >= (__u16x8)__b)` 从当前函数返回。
- **L767 EN**: Closes the current lexical scope or compound statement.
  **L767 CN**: 结束当前词法作用域或复合语句块。
- **L768 EN**: Blank line separating nearby declarations or logic blocks.
  **L768 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 769-792

````c
static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_eq(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i32x4)__a == (__i32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_ne(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i32x4)__a != (__i32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_lt(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i32x4)__a < (__i32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_lt(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__u32x4)__a < (__u32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_gt(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i32x4)__a > (__i32x4)__b);
}
````
- **L769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_eq(v128_t __a,`.
  **L769 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_eq(v128_t __a,`。
- **L770 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L770 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L771 EN**: Returns from the current function with `(v128_t)((__i32x4)__a == (__i32x4)__b)`.
  **L771 CN**: 以 `(v128_t)((__i32x4)__a == (__i32x4)__b)` 从当前函数返回。
- **L772 EN**: Closes the current lexical scope or compound statement.
  **L772 CN**: 结束当前词法作用域或复合语句块。
- **L773 EN**: Blank line separating nearby declarations or logic blocks.
  **L773 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_ne(v128_t __a,`.
  **L774 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_ne(v128_t __a,`。
- **L775 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L775 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L776 EN**: Returns from the current function with `(v128_t)((__i32x4)__a != (__i32x4)__b)`.
  **L776 CN**: 以 `(v128_t)((__i32x4)__a != (__i32x4)__b)` 从当前函数返回。
- **L777 EN**: Closes the current lexical scope or compound statement.
  **L777 CN**: 结束当前词法作用域或复合语句块。
- **L778 EN**: Blank line separating nearby declarations or logic blocks.
  **L778 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_lt(v128_t __a,`.
  **L779 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_lt(v128_t __a,`。
- **L780 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L780 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L781 EN**: Returns from the current function with `(v128_t)((__i32x4)__a < (__i32x4)__b)`.
  **L781 CN**: 以 `(v128_t)((__i32x4)__a < (__i32x4)__b)` 从当前函数返回。
- **L782 EN**: Closes the current lexical scope or compound statement.
  **L782 CN**: 结束当前词法作用域或复合语句块。
- **L783 EN**: Blank line separating nearby declarations or logic blocks.
  **L783 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L784 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_lt(v128_t __a,`.
  **L784 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_lt(v128_t __a,`。
- **L785 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L785 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L786 EN**: Returns from the current function with `(v128_t)((__u32x4)__a < (__u32x4)__b)`.
  **L786 CN**: 以 `(v128_t)((__u32x4)__a < (__u32x4)__b)` 从当前函数返回。
- **L787 EN**: Closes the current lexical scope or compound statement.
  **L787 CN**: 结束当前词法作用域或复合语句块。
- **L788 EN**: Blank line separating nearby declarations or logic blocks.
  **L788 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L789 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_gt(v128_t __a,`.
  **L789 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_gt(v128_t __a,`。
- **L790 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L790 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L791 EN**: Returns from the current function with `(v128_t)((__i32x4)__a > (__i32x4)__b)`.
  **L791 CN**: 以 `(v128_t)((__i32x4)__a > (__i32x4)__b)` 从当前函数返回。
- **L792 EN**: Closes the current lexical scope or compound statement.
  **L792 CN**: 结束当前词法作用域或复合语句块。

### Lines 793-816

````c

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_gt(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__u32x4)__a > (__u32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_le(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i32x4)__a <= (__i32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_le(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__u32x4)__a <= (__u32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_ge(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i32x4)__a >= (__i32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_ge(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__u32x4)__a >= (__u32x4)__b);
````
- **L793 EN**: Blank line separating nearby declarations or logic blocks.
  **L793 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L794 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_gt(v128_t __a,`.
  **L794 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_gt(v128_t __a,`。
- **L795 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L795 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L796 EN**: Returns from the current function with `(v128_t)((__u32x4)__a > (__u32x4)__b)`.
  **L796 CN**: 以 `(v128_t)((__u32x4)__a > (__u32x4)__b)` 从当前函数返回。
- **L797 EN**: Closes the current lexical scope or compound statement.
  **L797 CN**: 结束当前词法作用域或复合语句块。
- **L798 EN**: Blank line separating nearby declarations or logic blocks.
  **L798 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L799 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_le(v128_t __a,`.
  **L799 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_le(v128_t __a,`。
- **L800 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L800 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L801 EN**: Returns from the current function with `(v128_t)((__i32x4)__a <= (__i32x4)__b)`.
  **L801 CN**: 以 `(v128_t)((__i32x4)__a <= (__i32x4)__b)` 从当前函数返回。
- **L802 EN**: Closes the current lexical scope or compound statement.
  **L802 CN**: 结束当前词法作用域或复合语句块。
- **L803 EN**: Blank line separating nearby declarations or logic blocks.
  **L803 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_le(v128_t __a,`.
  **L804 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_le(v128_t __a,`。
- **L805 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L805 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L806 EN**: Returns from the current function with `(v128_t)((__u32x4)__a <= (__u32x4)__b)`.
  **L806 CN**: 以 `(v128_t)((__u32x4)__a <= (__u32x4)__b)` 从当前函数返回。
- **L807 EN**: Closes the current lexical scope or compound statement.
  **L807 CN**: 结束当前词法作用域或复合语句块。
- **L808 EN**: Blank line separating nearby declarations or logic blocks.
  **L808 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L809 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_ge(v128_t __a,`.
  **L809 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_ge(v128_t __a,`。
- **L810 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L810 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L811 EN**: Returns from the current function with `(v128_t)((__i32x4)__a >= (__i32x4)__b)`.
  **L811 CN**: 以 `(v128_t)((__i32x4)__a >= (__i32x4)__b)` 从当前函数返回。
- **L812 EN**: Closes the current lexical scope or compound statement.
  **L812 CN**: 结束当前词法作用域或复合语句块。
- **L813 EN**: Blank line separating nearby declarations or logic blocks.
  **L813 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L814 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_ge(v128_t __a,`.
  **L814 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_ge(v128_t __a,`。
- **L815 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L815 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L816 EN**: Returns from the current function with `(v128_t)((__u32x4)__a >= (__u32x4)__b)`.
  **L816 CN**: 以 `(v128_t)((__u32x4)__a >= (__u32x4)__b)` 从当前函数返回。

### Lines 817-840

````c
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_eq(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i64x2)__a == (__i64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_ne(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i64x2)__a != (__i64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_lt(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i64x2)__a < (__i64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_gt(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i64x2)__a > (__i64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_le(v128_t __a,
                                                          v128_t __b) {
````
- **L817 EN**: Closes the current lexical scope or compound statement.
  **L817 CN**: 结束当前词法作用域或复合语句块。
- **L818 EN**: Blank line separating nearby declarations or logic blocks.
  **L818 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L819 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_eq(v128_t __a,`.
  **L819 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_eq(v128_t __a,`。
- **L820 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L820 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L821 EN**: Returns from the current function with `(v128_t)((__i64x2)__a == (__i64x2)__b)`.
  **L821 CN**: 以 `(v128_t)((__i64x2)__a == (__i64x2)__b)` 从当前函数返回。
- **L822 EN**: Closes the current lexical scope or compound statement.
  **L822 CN**: 结束当前词法作用域或复合语句块。
- **L823 EN**: Blank line separating nearby declarations or logic blocks.
  **L823 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L824 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_ne(v128_t __a,`.
  **L824 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_ne(v128_t __a,`。
- **L825 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L825 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L826 EN**: Returns from the current function with `(v128_t)((__i64x2)__a != (__i64x2)__b)`.
  **L826 CN**: 以 `(v128_t)((__i64x2)__a != (__i64x2)__b)` 从当前函数返回。
- **L827 EN**: Closes the current lexical scope or compound statement.
  **L827 CN**: 结束当前词法作用域或复合语句块。
- **L828 EN**: Blank line separating nearby declarations or logic blocks.
  **L828 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L829 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_lt(v128_t __a,`.
  **L829 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_lt(v128_t __a,`。
- **L830 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L830 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L831 EN**: Returns from the current function with `(v128_t)((__i64x2)__a < (__i64x2)__b)`.
  **L831 CN**: 以 `(v128_t)((__i64x2)__a < (__i64x2)__b)` 从当前函数返回。
- **L832 EN**: Closes the current lexical scope or compound statement.
  **L832 CN**: 结束当前词法作用域或复合语句块。
- **L833 EN**: Blank line separating nearby declarations or logic blocks.
  **L833 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L834 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_gt(v128_t __a,`.
  **L834 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_gt(v128_t __a,`。
- **L835 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L835 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L836 EN**: Returns from the current function with `(v128_t)((__i64x2)__a > (__i64x2)__b)`.
  **L836 CN**: 以 `(v128_t)((__i64x2)__a > (__i64x2)__b)` 从当前函数返回。
- **L837 EN**: Closes the current lexical scope or compound statement.
  **L837 CN**: 结束当前词法作用域或复合语句块。
- **L838 EN**: Blank line separating nearby declarations or logic blocks.
  **L838 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L839 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_le(v128_t __a,`.
  **L839 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_le(v128_t __a,`。
- **L840 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L840 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。

### Lines 841-864

````c
  return (v128_t)((__i64x2)__a <= (__i64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_ge(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__i64x2)__a >= (__i64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_eq(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__f32x4)__a == (__f32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_ne(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__f32x4)__a != (__f32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_lt(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__f32x4)__a < (__f32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_gt(v128_t __a,
````
- **L841 EN**: Returns from the current function with `(v128_t)((__i64x2)__a <= (__i64x2)__b)`.
  **L841 CN**: 以 `(v128_t)((__i64x2)__a <= (__i64x2)__b)` 从当前函数返回。
- **L842 EN**: Closes the current lexical scope or compound statement.
  **L842 CN**: 结束当前词法作用域或复合语句块。
- **L843 EN**: Blank line separating nearby declarations or logic blocks.
  **L843 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L844 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_ge(v128_t __a,`.
  **L844 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_ge(v128_t __a,`。
- **L845 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L845 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L846 EN**: Returns from the current function with `(v128_t)((__i64x2)__a >= (__i64x2)__b)`.
  **L846 CN**: 以 `(v128_t)((__i64x2)__a >= (__i64x2)__b)` 从当前函数返回。
- **L847 EN**: Closes the current lexical scope or compound statement.
  **L847 CN**: 结束当前词法作用域或复合语句块。
- **L848 EN**: Blank line separating nearby declarations or logic blocks.
  **L848 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L849 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_eq(v128_t __a,`.
  **L849 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_eq(v128_t __a,`。
- **L850 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L850 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L851 EN**: Returns from the current function with `(v128_t)((__f32x4)__a == (__f32x4)__b)`.
  **L851 CN**: 以 `(v128_t)((__f32x4)__a == (__f32x4)__b)` 从当前函数返回。
- **L852 EN**: Closes the current lexical scope or compound statement.
  **L852 CN**: 结束当前词法作用域或复合语句块。
- **L853 EN**: Blank line separating nearby declarations or logic blocks.
  **L853 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L854 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_ne(v128_t __a,`.
  **L854 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_ne(v128_t __a,`。
- **L855 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L855 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L856 EN**: Returns from the current function with `(v128_t)((__f32x4)__a != (__f32x4)__b)`.
  **L856 CN**: 以 `(v128_t)((__f32x4)__a != (__f32x4)__b)` 从当前函数返回。
- **L857 EN**: Closes the current lexical scope or compound statement.
  **L857 CN**: 结束当前词法作用域或复合语句块。
- **L858 EN**: Blank line separating nearby declarations or logic blocks.
  **L858 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L859 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_lt(v128_t __a,`.
  **L859 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_lt(v128_t __a,`。
- **L860 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L860 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L861 EN**: Returns from the current function with `(v128_t)((__f32x4)__a < (__f32x4)__b)`.
  **L861 CN**: 以 `(v128_t)((__f32x4)__a < (__f32x4)__b)` 从当前函数返回。
- **L862 EN**: Closes the current lexical scope or compound statement.
  **L862 CN**: 结束当前词法作用域或复合语句块。
- **L863 EN**: Blank line separating nearby declarations or logic blocks.
  **L863 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L864 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_gt(v128_t __a,`.
  **L864 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_gt(v128_t __a,`。

### Lines 865-888

````c
                                                          v128_t __b) {
  return (v128_t)((__f32x4)__a > (__f32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_le(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__f32x4)__a <= (__f32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_ge(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__f32x4)__a >= (__f32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_eq(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__f64x2)__a == (__f64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_ne(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__f64x2)__a != (__f64x2)__b);
}

````
- **L865 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L865 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L866 EN**: Returns from the current function with `(v128_t)((__f32x4)__a > (__f32x4)__b)`.
  **L866 CN**: 以 `(v128_t)((__f32x4)__a > (__f32x4)__b)` 从当前函数返回。
- **L867 EN**: Closes the current lexical scope or compound statement.
  **L867 CN**: 结束当前词法作用域或复合语句块。
- **L868 EN**: Blank line separating nearby declarations or logic blocks.
  **L868 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L869 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_le(v128_t __a,`.
  **L869 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_le(v128_t __a,`。
- **L870 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L870 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L871 EN**: Returns from the current function with `(v128_t)((__f32x4)__a <= (__f32x4)__b)`.
  **L871 CN**: 以 `(v128_t)((__f32x4)__a <= (__f32x4)__b)` 从当前函数返回。
- **L872 EN**: Closes the current lexical scope or compound statement.
  **L872 CN**: 结束当前词法作用域或复合语句块。
- **L873 EN**: Blank line separating nearby declarations or logic blocks.
  **L873 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L874 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_ge(v128_t __a,`.
  **L874 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_ge(v128_t __a,`。
- **L875 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L875 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L876 EN**: Returns from the current function with `(v128_t)((__f32x4)__a >= (__f32x4)__b)`.
  **L876 CN**: 以 `(v128_t)((__f32x4)__a >= (__f32x4)__b)` 从当前函数返回。
- **L877 EN**: Closes the current lexical scope or compound statement.
  **L877 CN**: 结束当前词法作用域或复合语句块。
- **L878 EN**: Blank line separating nearby declarations or logic blocks.
  **L878 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L879 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_eq(v128_t __a,`.
  **L879 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_eq(v128_t __a,`。
- **L880 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L880 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L881 EN**: Returns from the current function with `(v128_t)((__f64x2)__a == (__f64x2)__b)`.
  **L881 CN**: 以 `(v128_t)((__f64x2)__a == (__f64x2)__b)` 从当前函数返回。
- **L882 EN**: Closes the current lexical scope or compound statement.
  **L882 CN**: 结束当前词法作用域或复合语句块。
- **L883 EN**: Blank line separating nearby declarations or logic blocks.
  **L883 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L884 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_ne(v128_t __a,`.
  **L884 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_ne(v128_t __a,`。
- **L885 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L885 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L886 EN**: Returns from the current function with `(v128_t)((__f64x2)__a != (__f64x2)__b)`.
  **L886 CN**: 以 `(v128_t)((__f64x2)__a != (__f64x2)__b)` 从当前函数返回。
- **L887 EN**: Closes the current lexical scope or compound statement.
  **L887 CN**: 结束当前词法作用域或复合语句块。
- **L888 EN**: Blank line separating nearby declarations or logic blocks.
  **L888 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 889-912

````c
static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_lt(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__f64x2)__a < (__f64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_gt(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__f64x2)__a > (__f64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_le(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__f64x2)__a <= (__f64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_ge(v128_t __a,
                                                          v128_t __b) {
  return (v128_t)((__f64x2)__a >= (__f64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_not(v128_t __a) {
  return ~__a;
}

````
- **L889 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_lt(v128_t __a,`.
  **L889 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_lt(v128_t __a,`。
- **L890 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L890 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L891 EN**: Returns from the current function with `(v128_t)((__f64x2)__a < (__f64x2)__b)`.
  **L891 CN**: 以 `(v128_t)((__f64x2)__a < (__f64x2)__b)` 从当前函数返回。
- **L892 EN**: Closes the current lexical scope or compound statement.
  **L892 CN**: 结束当前词法作用域或复合语句块。
- **L893 EN**: Blank line separating nearby declarations or logic blocks.
  **L893 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L894 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_gt(v128_t __a,`.
  **L894 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_gt(v128_t __a,`。
- **L895 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L895 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L896 EN**: Returns from the current function with `(v128_t)((__f64x2)__a > (__f64x2)__b)`.
  **L896 CN**: 以 `(v128_t)((__f64x2)__a > (__f64x2)__b)` 从当前函数返回。
- **L897 EN**: Closes the current lexical scope or compound statement.
  **L897 CN**: 结束当前词法作用域或复合语句块。
- **L898 EN**: Blank line separating nearby declarations or logic blocks.
  **L898 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L899 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_le(v128_t __a,`.
  **L899 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_le(v128_t __a,`。
- **L900 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L900 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L901 EN**: Returns from the current function with `(v128_t)((__f64x2)__a <= (__f64x2)__b)`.
  **L901 CN**: 以 `(v128_t)((__f64x2)__a <= (__f64x2)__b)` 从当前函数返回。
- **L902 EN**: Closes the current lexical scope or compound statement.
  **L902 CN**: 结束当前词法作用域或复合语句块。
- **L903 EN**: Blank line separating nearby declarations or logic blocks.
  **L903 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L904 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_ge(v128_t __a,`.
  **L904 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_ge(v128_t __a,`。
- **L905 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L905 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L906 EN**: Returns from the current function with `(v128_t)((__f64x2)__a >= (__f64x2)__b)`.
  **L906 CN**: 以 `(v128_t)((__f64x2)__a >= (__f64x2)__b)` 从当前函数返回。
- **L907 EN**: Closes the current lexical scope or compound statement.
  **L907 CN**: 结束当前词法作用域或复合语句块。
- **L908 EN**: Blank line separating nearby declarations or logic blocks.
  **L908 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L909 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_not(v128_t __a) {`.
  **L909 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_not(v128_t __a) {`。
- **L910 EN**: Returns from the current function with `~__a`.
  **L910 CN**: 以 `~__a` 从当前函数返回。
- **L911 EN**: Closes the current lexical scope or compound statement.
  **L911 CN**: 结束当前词法作用域或复合语句块。
- **L912 EN**: Blank line separating nearby declarations or logic blocks.
  **L912 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 913-936

````c
static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_and(v128_t __a,
                                                          v128_t __b) {
  return __a & __b;
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_or(v128_t __a,
                                                         v128_t __b) {
  return __a | __b;
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_xor(v128_t __a,
                                                          v128_t __b) {
  return __a ^ __b;
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_andnot(v128_t __a,
                                                             v128_t __b) {
  return __a & ~__b;
}

static __inline__ bool __DEFAULT_FN_ATTRS wasm_v128_any_true(v128_t __a) {
  return __builtin_wasm_any_true_v128((__i8x16)__a);
}

````
- **L913 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_and(v128_t __a,`.
  **L913 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_and(v128_t __a,`。
- **L914 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L914 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L915 EN**: Returns from the current function with `__a & __b`.
  **L915 CN**: 以 `__a & __b` 从当前函数返回。
- **L916 EN**: Closes the current lexical scope or compound statement.
  **L916 CN**: 结束当前词法作用域或复合语句块。
- **L917 EN**: Blank line separating nearby declarations or logic blocks.
  **L917 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L918 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_or(v128_t __a,`.
  **L918 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_or(v128_t __a,`。
- **L919 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L919 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L920 EN**: Returns from the current function with `__a | __b`.
  **L920 CN**: 以 `__a | __b` 从当前函数返回。
- **L921 EN**: Closes the current lexical scope or compound statement.
  **L921 CN**: 结束当前词法作用域或复合语句块。
- **L922 EN**: Blank line separating nearby declarations or logic blocks.
  **L922 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L923 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_xor(v128_t __a,`.
  **L923 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_xor(v128_t __a,`。
- **L924 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L924 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L925 EN**: Returns from the current function with `__a ^ __b`.
  **L925 CN**: 以 `__a ^ __b` 从当前函数返回。
- **L926 EN**: Closes the current lexical scope or compound statement.
  **L926 CN**: 结束当前词法作用域或复合语句块。
- **L927 EN**: Blank line separating nearby declarations or logic blocks.
  **L927 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L928 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_andnot(v128_t __a,`.
  **L928 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_andnot(v128_t __a,`。
- **L929 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L929 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L930 EN**: Returns from the current function with `__a & ~__b`.
  **L930 CN**: 以 `__a & ~__b` 从当前函数返回。
- **L931 EN**: Closes the current lexical scope or compound statement.
  **L931 CN**: 结束当前词法作用域或复合语句块。
- **L932 EN**: Blank line separating nearby declarations or logic blocks.
  **L932 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L933 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ bool __DEFAULT_FN_ATTRS wasm_v128_any_true(v128_t __a) {`.
  **L933 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ bool __DEFAULT_FN_ATTRS wasm_v128_any_true(v128_t __a) {`。
- **L934 EN**: Returns from the current function with `__builtin_wasm_any_true_v128((__i8x16)__a)`.
  **L934 CN**: 以 `__builtin_wasm_any_true_v128((__i8x16)__a)` 从当前函数返回。
- **L935 EN**: Closes the current lexical scope or compound statement.
  **L935 CN**: 结束当前词法作用域或复合语句块。
- **L936 EN**: Blank line separating nearby declarations or logic blocks.
  **L936 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 937-960

````c
static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_bitselect(v128_t __a,
                                                                v128_t __b,
                                                                v128_t __mask) {
  return (v128_t)__builtin_wasm_bitselect((__i32x4)__a, (__i32x4)__b,
                                          (__i32x4)__mask);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_abs(v128_t __a) {
  return (v128_t)__builtin_wasm_abs_i8x16((__i8x16)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_neg(v128_t __a) {
  return (v128_t)(-(__u8x16)__a);
}

static __inline__ bool __DEFAULT_FN_ATTRS wasm_i8x16_all_true(v128_t __a) {
  return __builtin_wasm_all_true_i8x16((__i8x16)__a);
}

static __inline__ uint32_t __DEFAULT_FN_ATTRS wasm_i8x16_bitmask(v128_t __a) {
  return __builtin_wasm_bitmask_i8x16((__i8x16)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_popcnt(v128_t __a) {
````
- **L937 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_bitselect(v128_t __a,`.
  **L937 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_v128_bitselect(v128_t __a,`。
- **L938 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `v128_t __b,`.
  **L938 CN**: 继续一个多行参数列表、初始化器或聚合项：`v128_t __b,`。
- **L939 EN**: Continues the surrounding expression or declaration: `v128_t __mask) {`.
  **L939 CN**: 继续构造周围的表达式或声明：`v128_t __mask) {`。
- **L940 EN**: Returns from the current function with `(v128_t)__builtin_wasm_bitselect((__i32x4)__a, (__i32x4)__b,`.
  **L940 CN**: 以 `(v128_t)__builtin_wasm_bitselect((__i32x4)__a, (__i32x4)__b,` 从当前函数返回。
- **L941 EN**: Executes a call or declaration centered on `statement`.
  **L941 CN**: 执行以 `statement` 为核心的调用或声明。
- **L942 EN**: Closes the current lexical scope or compound statement.
  **L942 CN**: 结束当前词法作用域或复合语句块。
- **L943 EN**: Blank line separating nearby declarations or logic blocks.
  **L943 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L944 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_abs(v128_t __a) {`.
  **L944 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_abs(v128_t __a) {`。
- **L945 EN**: Returns from the current function with `(v128_t)__builtin_wasm_abs_i8x16((__i8x16)__a)`.
  **L945 CN**: 以 `(v128_t)__builtin_wasm_abs_i8x16((__i8x16)__a)` 从当前函数返回。
- **L946 EN**: Closes the current lexical scope or compound statement.
  **L946 CN**: 结束当前词法作用域或复合语句块。
- **L947 EN**: Blank line separating nearby declarations or logic blocks.
  **L947 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L948 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_neg(v128_t __a) {`.
  **L948 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_neg(v128_t __a) {`。
- **L949 EN**: Returns from the current function with `(v128_t)(-(__u8x16)__a)`.
  **L949 CN**: 以 `(v128_t)(-(__u8x16)__a)` 从当前函数返回。
- **L950 EN**: Closes the current lexical scope or compound statement.
  **L950 CN**: 结束当前词法作用域或复合语句块。
- **L951 EN**: Blank line separating nearby declarations or logic blocks.
  **L951 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L952 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ bool __DEFAULT_FN_ATTRS wasm_i8x16_all_true(v128_t __a) {`.
  **L952 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ bool __DEFAULT_FN_ATTRS wasm_i8x16_all_true(v128_t __a) {`。
- **L953 EN**: Returns from the current function with `__builtin_wasm_all_true_i8x16((__i8x16)__a)`.
  **L953 CN**: 以 `__builtin_wasm_all_true_i8x16((__i8x16)__a)` 从当前函数返回。
- **L954 EN**: Closes the current lexical scope or compound statement.
  **L954 CN**: 结束当前词法作用域或复合语句块。
- **L955 EN**: Blank line separating nearby declarations or logic blocks.
  **L955 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L956 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ uint32_t __DEFAULT_FN_ATTRS wasm_i8x16_bitmask(v128_t __a) {`.
  **L956 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ uint32_t __DEFAULT_FN_ATTRS wasm_i8x16_bitmask(v128_t __a) {`。
- **L957 EN**: Returns from the current function with `__builtin_wasm_bitmask_i8x16((__i8x16)__a)`.
  **L957 CN**: 以 `__builtin_wasm_bitmask_i8x16((__i8x16)__a)` 从当前函数返回。
- **L958 EN**: Closes the current lexical scope or compound statement.
  **L958 CN**: 结束当前词法作用域或复合语句块。
- **L959 EN**: Blank line separating nearby declarations or logic blocks.
  **L959 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L960 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_popcnt(v128_t __a) {`.
  **L960 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_popcnt(v128_t __a) {`。

### Lines 961-984

````c
  return (v128_t)__builtin_elementwise_popcount((__i8x16)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_shl(v128_t __a,
                                                           uint32_t __b) {
  return (v128_t)((__i8x16)__a << (__b & 0x7));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_shr(v128_t __a,
                                                           uint32_t __b) {
  return (v128_t)((__i8x16)__a >> (__b & 0x7));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_shr(v128_t __a,
                                                           uint32_t __b) {
  return (v128_t)((__u8x16)__a >> (__b & 0x7));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_add(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__u8x16)__a + (__u8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_add_sat(v128_t __a,
````
- **L961 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_popcount((__i8x16)__a)`.
  **L961 CN**: 以 `(v128_t)__builtin_elementwise_popcount((__i8x16)__a)` 从当前函数返回。
- **L962 EN**: Closes the current lexical scope or compound statement.
  **L962 CN**: 结束当前词法作用域或复合语句块。
- **L963 EN**: Blank line separating nearby declarations or logic blocks.
  **L963 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L964 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_shl(v128_t __a,`.
  **L964 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_shl(v128_t __a,`。
- **L965 EN**: Continues the surrounding expression or declaration: `uint32_t __b) {`.
  **L965 CN**: 继续构造周围的表达式或声明：`uint32_t __b) {`。
- **L966 EN**: Returns from the current function with `(v128_t)((__i8x16)__a << (__b & 0x7))`.
  **L966 CN**: 以 `(v128_t)((__i8x16)__a << (__b & 0x7))` 从当前函数返回。
- **L967 EN**: Closes the current lexical scope or compound statement.
  **L967 CN**: 结束当前词法作用域或复合语句块。
- **L968 EN**: Blank line separating nearby declarations or logic blocks.
  **L968 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L969 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_shr(v128_t __a,`.
  **L969 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_shr(v128_t __a,`。
- **L970 EN**: Continues the surrounding expression or declaration: `uint32_t __b) {`.
  **L970 CN**: 继续构造周围的表达式或声明：`uint32_t __b) {`。
- **L971 EN**: Returns from the current function with `(v128_t)((__i8x16)__a >> (__b & 0x7))`.
  **L971 CN**: 以 `(v128_t)((__i8x16)__a >> (__b & 0x7))` 从当前函数返回。
- **L972 EN**: Closes the current lexical scope or compound statement.
  **L972 CN**: 结束当前词法作用域或复合语句块。
- **L973 EN**: Blank line separating nearby declarations or logic blocks.
  **L973 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L974 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_shr(v128_t __a,`.
  **L974 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_shr(v128_t __a,`。
- **L975 EN**: Continues the surrounding expression or declaration: `uint32_t __b) {`.
  **L975 CN**: 继续构造周围的表达式或声明：`uint32_t __b) {`。
- **L976 EN**: Returns from the current function with `(v128_t)((__u8x16)__a >> (__b & 0x7))`.
  **L976 CN**: 以 `(v128_t)((__u8x16)__a >> (__b & 0x7))` 从当前函数返回。
- **L977 EN**: Closes the current lexical scope or compound statement.
  **L977 CN**: 结束当前词法作用域或复合语句块。
- **L978 EN**: Blank line separating nearby declarations or logic blocks.
  **L978 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L979 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_add(v128_t __a,`.
  **L979 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_add(v128_t __a,`。
- **L980 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L980 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L981 EN**: Returns from the current function with `(v128_t)((__u8x16)__a + (__u8x16)__b)`.
  **L981 CN**: 以 `(v128_t)((__u8x16)__a + (__u8x16)__b)` 从当前函数返回。
- **L982 EN**: Closes the current lexical scope or compound statement.
  **L982 CN**: 结束当前词法作用域或复合语句块。
- **L983 EN**: Blank line separating nearby declarations or logic blocks.
  **L983 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L984 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_add_sat(v128_t __a,`.
  **L984 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_add_sat(v128_t __a,`。

### Lines 985-1008

````c
                                                               v128_t __b) {
  return (v128_t)__builtin_elementwise_add_sat((__i8x16)__a, (__i8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_add_sat(v128_t __a,
                                                               v128_t __b) {
  return (v128_t)__builtin_elementwise_add_sat((__u8x16)__a, (__u8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_sub(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__u8x16)__a - (__u8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_sub_sat(v128_t __a,
                                                               v128_t __b) {
  return (v128_t)__builtin_elementwise_sub_sat((__i8x16)__a, (__i8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_sub_sat(v128_t __a,
                                                               v128_t __b) {
  return (v128_t)__builtin_elementwise_sub_sat((__u8x16)__a, (__u8x16)__b);
}

````
- **L985 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L985 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L986 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_add_sat((__i8x16)__a, (__i8x16)__b)`.
  **L986 CN**: 以 `(v128_t)__builtin_elementwise_add_sat((__i8x16)__a, (__i8x16)__b)` 从当前函数返回。
- **L987 EN**: Closes the current lexical scope or compound statement.
  **L987 CN**: 结束当前词法作用域或复合语句块。
- **L988 EN**: Blank line separating nearby declarations or logic blocks.
  **L988 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L989 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_add_sat(v128_t __a,`.
  **L989 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_add_sat(v128_t __a,`。
- **L990 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L990 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L991 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_add_sat((__u8x16)__a, (__u8x16)__b)`.
  **L991 CN**: 以 `(v128_t)__builtin_elementwise_add_sat((__u8x16)__a, (__u8x16)__b)` 从当前函数返回。
- **L992 EN**: Closes the current lexical scope or compound statement.
  **L992 CN**: 结束当前词法作用域或复合语句块。
- **L993 EN**: Blank line separating nearby declarations or logic blocks.
  **L993 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L994 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_sub(v128_t __a,`.
  **L994 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_sub(v128_t __a,`。
- **L995 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L995 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L996 EN**: Returns from the current function with `(v128_t)((__u8x16)__a - (__u8x16)__b)`.
  **L996 CN**: 以 `(v128_t)((__u8x16)__a - (__u8x16)__b)` 从当前函数返回。
- **L997 EN**: Closes the current lexical scope or compound statement.
  **L997 CN**: 结束当前词法作用域或复合语句块。
- **L998 EN**: Blank line separating nearby declarations or logic blocks.
  **L998 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L999 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_sub_sat(v128_t __a,`.
  **L999 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_sub_sat(v128_t __a,`。
- **L1000 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1000 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1001 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_sub_sat((__i8x16)__a, (__i8x16)__b)`.
  **L1001 CN**: 以 `(v128_t)__builtin_elementwise_sub_sat((__i8x16)__a, (__i8x16)__b)` 从当前函数返回。
- **L1002 EN**: Closes the current lexical scope or compound statement.
  **L1002 CN**: 结束当前词法作用域或复合语句块。
- **L1003 EN**: Blank line separating nearby declarations or logic blocks.
  **L1003 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1004 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_sub_sat(v128_t __a,`.
  **L1004 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_sub_sat(v128_t __a,`。
- **L1005 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1005 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1006 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_sub_sat((__u8x16)__a, (__u8x16)__b)`.
  **L1006 CN**: 以 `(v128_t)__builtin_elementwise_sub_sat((__u8x16)__a, (__u8x16)__b)` 从当前函数返回。
- **L1007 EN**: Closes the current lexical scope or compound statement.
  **L1007 CN**: 结束当前词法作用域或复合语句块。
- **L1008 EN**: Blank line separating nearby declarations or logic blocks.
  **L1008 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1009-1032

````c
static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_min(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)__builtin_elementwise_min((__i8x16)__a, (__i8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_min(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)__builtin_elementwise_min((__u8x16)__a, (__u8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_max(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)__builtin_elementwise_max((__i8x16)__a, (__i8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_max(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)__builtin_elementwise_max((__u8x16)__a, (__u8x16)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_avgr(v128_t __a,
                                                            v128_t __b) {
  return (v128_t)__builtin_wasm_avgr_u_i8x16((__u8x16)__a, (__u8x16)__b);
}
````
- **L1009 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_min(v128_t __a,`.
  **L1009 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_min(v128_t __a,`。
- **L1010 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1010 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1011 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_min((__i8x16)__a, (__i8x16)__b)`.
  **L1011 CN**: 以 `(v128_t)__builtin_elementwise_min((__i8x16)__a, (__i8x16)__b)` 从当前函数返回。
- **L1012 EN**: Closes the current lexical scope or compound statement.
  **L1012 CN**: 结束当前词法作用域或复合语句块。
- **L1013 EN**: Blank line separating nearby declarations or logic blocks.
  **L1013 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1014 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_min(v128_t __a,`.
  **L1014 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_min(v128_t __a,`。
- **L1015 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1015 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1016 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_min((__u8x16)__a, (__u8x16)__b)`.
  **L1016 CN**: 以 `(v128_t)__builtin_elementwise_min((__u8x16)__a, (__u8x16)__b)` 从当前函数返回。
- **L1017 EN**: Closes the current lexical scope or compound statement.
  **L1017 CN**: 结束当前词法作用域或复合语句块。
- **L1018 EN**: Blank line separating nearby declarations or logic blocks.
  **L1018 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1019 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_max(v128_t __a,`.
  **L1019 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_max(v128_t __a,`。
- **L1020 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1020 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1021 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_max((__i8x16)__a, (__i8x16)__b)`.
  **L1021 CN**: 以 `(v128_t)__builtin_elementwise_max((__i8x16)__a, (__i8x16)__b)` 从当前函数返回。
- **L1022 EN**: Closes the current lexical scope or compound statement.
  **L1022 CN**: 结束当前词法作用域或复合语句块。
- **L1023 EN**: Blank line separating nearby declarations or logic blocks.
  **L1023 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1024 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_max(v128_t __a,`.
  **L1024 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_max(v128_t __a,`。
- **L1025 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1025 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1026 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_max((__u8x16)__a, (__u8x16)__b)`.
  **L1026 CN**: 以 `(v128_t)__builtin_elementwise_max((__u8x16)__a, (__u8x16)__b)` 从当前函数返回。
- **L1027 EN**: Closes the current lexical scope or compound statement.
  **L1027 CN**: 结束当前词法作用域或复合语句块。
- **L1028 EN**: Blank line separating nearby declarations or logic blocks.
  **L1028 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1029 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_avgr(v128_t __a,`.
  **L1029 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u8x16_avgr(v128_t __a,`。
- **L1030 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1030 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1031 EN**: Returns from the current function with `(v128_t)__builtin_wasm_avgr_u_i8x16((__u8x16)__a, (__u8x16)__b)`.
  **L1031 CN**: 以 `(v128_t)__builtin_wasm_avgr_u_i8x16((__u8x16)__a, (__u8x16)__b)` 从当前函数返回。
- **L1032 EN**: Closes the current lexical scope or compound statement.
  **L1032 CN**: 结束当前词法作用域或复合语句块。

### Lines 1033-1056

````c

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_abs(v128_t __a) {
  return (v128_t)__builtin_wasm_abs_i16x8((__i16x8)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_neg(v128_t __a) {
  return (v128_t)(-(__u16x8)__a);
}

static __inline__ bool __DEFAULT_FN_ATTRS wasm_i16x8_all_true(v128_t __a) {
  return __builtin_wasm_all_true_i16x8((__i16x8)__a);
}

static __inline__ uint32_t __DEFAULT_FN_ATTRS wasm_i16x8_bitmask(v128_t __a) {
  return __builtin_wasm_bitmask_i16x8((__i16x8)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_shl(v128_t __a,
                                                           uint32_t __b) {
  return (v128_t)((__i16x8)__a << (__b & 0xF));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_shr(v128_t __a,
                                                           uint32_t __b) {
````
- **L1033 EN**: Blank line separating nearby declarations or logic blocks.
  **L1033 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1034 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_abs(v128_t __a) {`.
  **L1034 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_abs(v128_t __a) {`。
- **L1035 EN**: Returns from the current function with `(v128_t)__builtin_wasm_abs_i16x8((__i16x8)__a)`.
  **L1035 CN**: 以 `(v128_t)__builtin_wasm_abs_i16x8((__i16x8)__a)` 从当前函数返回。
- **L1036 EN**: Closes the current lexical scope or compound statement.
  **L1036 CN**: 结束当前词法作用域或复合语句块。
- **L1037 EN**: Blank line separating nearby declarations or logic blocks.
  **L1037 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1038 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_neg(v128_t __a) {`.
  **L1038 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_neg(v128_t __a) {`。
- **L1039 EN**: Returns from the current function with `(v128_t)(-(__u16x8)__a)`.
  **L1039 CN**: 以 `(v128_t)(-(__u16x8)__a)` 从当前函数返回。
- **L1040 EN**: Closes the current lexical scope or compound statement.
  **L1040 CN**: 结束当前词法作用域或复合语句块。
- **L1041 EN**: Blank line separating nearby declarations or logic blocks.
  **L1041 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1042 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ bool __DEFAULT_FN_ATTRS wasm_i16x8_all_true(v128_t __a) {`.
  **L1042 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ bool __DEFAULT_FN_ATTRS wasm_i16x8_all_true(v128_t __a) {`。
- **L1043 EN**: Returns from the current function with `__builtin_wasm_all_true_i16x8((__i16x8)__a)`.
  **L1043 CN**: 以 `__builtin_wasm_all_true_i16x8((__i16x8)__a)` 从当前函数返回。
- **L1044 EN**: Closes the current lexical scope or compound statement.
  **L1044 CN**: 结束当前词法作用域或复合语句块。
- **L1045 EN**: Blank line separating nearby declarations or logic blocks.
  **L1045 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1046 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ uint32_t __DEFAULT_FN_ATTRS wasm_i16x8_bitmask(v128_t __a) {`.
  **L1046 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ uint32_t __DEFAULT_FN_ATTRS wasm_i16x8_bitmask(v128_t __a) {`。
- **L1047 EN**: Returns from the current function with `__builtin_wasm_bitmask_i16x8((__i16x8)__a)`.
  **L1047 CN**: 以 `__builtin_wasm_bitmask_i16x8((__i16x8)__a)` 从当前函数返回。
- **L1048 EN**: Closes the current lexical scope or compound statement.
  **L1048 CN**: 结束当前词法作用域或复合语句块。
- **L1049 EN**: Blank line separating nearby declarations or logic blocks.
  **L1049 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1050 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_shl(v128_t __a,`.
  **L1050 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_shl(v128_t __a,`。
- **L1051 EN**: Continues the surrounding expression or declaration: `uint32_t __b) {`.
  **L1051 CN**: 继续构造周围的表达式或声明：`uint32_t __b) {`。
- **L1052 EN**: Returns from the current function with `(v128_t)((__i16x8)__a << (__b & 0xF))`.
  **L1052 CN**: 以 `(v128_t)((__i16x8)__a << (__b & 0xF))` 从当前函数返回。
- **L1053 EN**: Closes the current lexical scope or compound statement.
  **L1053 CN**: 结束当前词法作用域或复合语句块。
- **L1054 EN**: Blank line separating nearby declarations or logic blocks.
  **L1054 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1055 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_shr(v128_t __a,`.
  **L1055 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_shr(v128_t __a,`。
- **L1056 EN**: Continues the surrounding expression or declaration: `uint32_t __b) {`.
  **L1056 CN**: 继续构造周围的表达式或声明：`uint32_t __b) {`。

### Lines 1057-1080

````c
  return (v128_t)((__i16x8)__a >> (__b & 0xF));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_shr(v128_t __a,
                                                           uint32_t __b) {
  return (v128_t)((__u16x8)__a >> (__b & 0xF));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_add(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__u16x8)__a + (__u16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_add_sat(v128_t __a,
                                                               v128_t __b) {
  return (v128_t)__builtin_elementwise_add_sat((__i16x8)__a, (__i16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_add_sat(v128_t __a,
                                                               v128_t __b) {
  return (v128_t)__builtin_elementwise_add_sat((__u16x8)__a, (__u16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_sub(v128_t __a,
````
- **L1057 EN**: Returns from the current function with `(v128_t)((__i16x8)__a >> (__b & 0xF))`.
  **L1057 CN**: 以 `(v128_t)((__i16x8)__a >> (__b & 0xF))` 从当前函数返回。
- **L1058 EN**: Closes the current lexical scope or compound statement.
  **L1058 CN**: 结束当前词法作用域或复合语句块。
- **L1059 EN**: Blank line separating nearby declarations or logic blocks.
  **L1059 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1060 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_shr(v128_t __a,`.
  **L1060 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_shr(v128_t __a,`。
- **L1061 EN**: Continues the surrounding expression or declaration: `uint32_t __b) {`.
  **L1061 CN**: 继续构造周围的表达式或声明：`uint32_t __b) {`。
- **L1062 EN**: Returns from the current function with `(v128_t)((__u16x8)__a >> (__b & 0xF))`.
  **L1062 CN**: 以 `(v128_t)((__u16x8)__a >> (__b & 0xF))` 从当前函数返回。
- **L1063 EN**: Closes the current lexical scope or compound statement.
  **L1063 CN**: 结束当前词法作用域或复合语句块。
- **L1064 EN**: Blank line separating nearby declarations or logic blocks.
  **L1064 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1065 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_add(v128_t __a,`.
  **L1065 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_add(v128_t __a,`。
- **L1066 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1066 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1067 EN**: Returns from the current function with `(v128_t)((__u16x8)__a + (__u16x8)__b)`.
  **L1067 CN**: 以 `(v128_t)((__u16x8)__a + (__u16x8)__b)` 从当前函数返回。
- **L1068 EN**: Closes the current lexical scope or compound statement.
  **L1068 CN**: 结束当前词法作用域或复合语句块。
- **L1069 EN**: Blank line separating nearby declarations or logic blocks.
  **L1069 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1070 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_add_sat(v128_t __a,`.
  **L1070 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_add_sat(v128_t __a,`。
- **L1071 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1071 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1072 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_add_sat((__i16x8)__a, (__i16x8)__b)`.
  **L1072 CN**: 以 `(v128_t)__builtin_elementwise_add_sat((__i16x8)__a, (__i16x8)__b)` 从当前函数返回。
- **L1073 EN**: Closes the current lexical scope or compound statement.
  **L1073 CN**: 结束当前词法作用域或复合语句块。
- **L1074 EN**: Blank line separating nearby declarations or logic blocks.
  **L1074 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1075 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_add_sat(v128_t __a,`.
  **L1075 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_add_sat(v128_t __a,`。
- **L1076 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1076 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1077 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_add_sat((__u16x8)__a, (__u16x8)__b)`.
  **L1077 CN**: 以 `(v128_t)__builtin_elementwise_add_sat((__u16x8)__a, (__u16x8)__b)` 从当前函数返回。
- **L1078 EN**: Closes the current lexical scope or compound statement.
  **L1078 CN**: 结束当前词法作用域或复合语句块。
- **L1079 EN**: Blank line separating nearby declarations or logic blocks.
  **L1079 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1080 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_sub(v128_t __a,`.
  **L1080 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_sub(v128_t __a,`。

### Lines 1081-1104

````c
                                                           v128_t __b) {
  return (v128_t)((__i16x8)__a - (__i16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_sub_sat(v128_t __a,
                                                               v128_t __b) {
  return (v128_t)__builtin_elementwise_sub_sat((__i16x8)__a, (__i16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_sub_sat(v128_t __a,
                                                               v128_t __b) {
  return (v128_t)__builtin_elementwise_sub_sat((__u16x8)__a, (__u16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_mul(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__u16x8)__a * (__u16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_min(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)__builtin_elementwise_min((__i16x8)__a, (__i16x8)__b);
}

````
- **L1081 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1081 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1082 EN**: Returns from the current function with `(v128_t)((__i16x8)__a - (__i16x8)__b)`.
  **L1082 CN**: 以 `(v128_t)((__i16x8)__a - (__i16x8)__b)` 从当前函数返回。
- **L1083 EN**: Closes the current lexical scope or compound statement.
  **L1083 CN**: 结束当前词法作用域或复合语句块。
- **L1084 EN**: Blank line separating nearby declarations or logic blocks.
  **L1084 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1085 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_sub_sat(v128_t __a,`.
  **L1085 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_sub_sat(v128_t __a,`。
- **L1086 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1086 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1087 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_sub_sat((__i16x8)__a, (__i16x8)__b)`.
  **L1087 CN**: 以 `(v128_t)__builtin_elementwise_sub_sat((__i16x8)__a, (__i16x8)__b)` 从当前函数返回。
- **L1088 EN**: Closes the current lexical scope or compound statement.
  **L1088 CN**: 结束当前词法作用域或复合语句块。
- **L1089 EN**: Blank line separating nearby declarations or logic blocks.
  **L1089 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1090 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_sub_sat(v128_t __a,`.
  **L1090 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_sub_sat(v128_t __a,`。
- **L1091 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1091 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1092 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_sub_sat((__u16x8)__a, (__u16x8)__b)`.
  **L1092 CN**: 以 `(v128_t)__builtin_elementwise_sub_sat((__u16x8)__a, (__u16x8)__b)` 从当前函数返回。
- **L1093 EN**: Closes the current lexical scope or compound statement.
  **L1093 CN**: 结束当前词法作用域或复合语句块。
- **L1094 EN**: Blank line separating nearby declarations or logic blocks.
  **L1094 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1095 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_mul(v128_t __a,`.
  **L1095 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_mul(v128_t __a,`。
- **L1096 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1096 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1097 EN**: Returns from the current function with `(v128_t)((__u16x8)__a * (__u16x8)__b)`.
  **L1097 CN**: 以 `(v128_t)((__u16x8)__a * (__u16x8)__b)` 从当前函数返回。
- **L1098 EN**: Closes the current lexical scope or compound statement.
  **L1098 CN**: 结束当前词法作用域或复合语句块。
- **L1099 EN**: Blank line separating nearby declarations or logic blocks.
  **L1099 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_min(v128_t __a,`.
  **L1100 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_min(v128_t __a,`。
- **L1101 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1101 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1102 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_min((__i16x8)__a, (__i16x8)__b)`.
  **L1102 CN**: 以 `(v128_t)__builtin_elementwise_min((__i16x8)__a, (__i16x8)__b)` 从当前函数返回。
- **L1103 EN**: Closes the current lexical scope or compound statement.
  **L1103 CN**: 结束当前词法作用域或复合语句块。
- **L1104 EN**: Blank line separating nearby declarations or logic blocks.
  **L1104 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1105-1128

````c
static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_min(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)__builtin_elementwise_min((__u16x8)__a, (__u16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_max(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)__builtin_elementwise_max((__i16x8)__a, (__i16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_max(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)__builtin_elementwise_max((__u16x8)__a, (__u16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_avgr(v128_t __a,
                                                            v128_t __b) {
  return (v128_t)__builtin_wasm_avgr_u_i16x8((__u16x8)__a, (__u16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_abs(v128_t __a) {
  return (v128_t)__builtin_wasm_abs_i32x4((__i32x4)__a);
}

````
- **L1105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_min(v128_t __a,`.
  **L1105 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_min(v128_t __a,`。
- **L1106 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1106 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1107 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_min((__u16x8)__a, (__u16x8)__b)`.
  **L1107 CN**: 以 `(v128_t)__builtin_elementwise_min((__u16x8)__a, (__u16x8)__b)` 从当前函数返回。
- **L1108 EN**: Closes the current lexical scope or compound statement.
  **L1108 CN**: 结束当前词法作用域或复合语句块。
- **L1109 EN**: Blank line separating nearby declarations or logic blocks.
  **L1109 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_max(v128_t __a,`.
  **L1110 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_max(v128_t __a,`。
- **L1111 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1111 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1112 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_max((__i16x8)__a, (__i16x8)__b)`.
  **L1112 CN**: 以 `(v128_t)__builtin_elementwise_max((__i16x8)__a, (__i16x8)__b)` 从当前函数返回。
- **L1113 EN**: Closes the current lexical scope or compound statement.
  **L1113 CN**: 结束当前词法作用域或复合语句块。
- **L1114 EN**: Blank line separating nearby declarations or logic blocks.
  **L1114 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_max(v128_t __a,`.
  **L1115 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_max(v128_t __a,`。
- **L1116 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1116 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1117 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_max((__u16x8)__a, (__u16x8)__b)`.
  **L1117 CN**: 以 `(v128_t)__builtin_elementwise_max((__u16x8)__a, (__u16x8)__b)` 从当前函数返回。
- **L1118 EN**: Closes the current lexical scope or compound statement.
  **L1118 CN**: 结束当前词法作用域或复合语句块。
- **L1119 EN**: Blank line separating nearby declarations or logic blocks.
  **L1119 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_avgr(v128_t __a,`.
  **L1120 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u16x8_avgr(v128_t __a,`。
- **L1121 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1121 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1122 EN**: Returns from the current function with `(v128_t)__builtin_wasm_avgr_u_i16x8((__u16x8)__a, (__u16x8)__b)`.
  **L1122 CN**: 以 `(v128_t)__builtin_wasm_avgr_u_i16x8((__u16x8)__a, (__u16x8)__b)` 从当前函数返回。
- **L1123 EN**: Closes the current lexical scope or compound statement.
  **L1123 CN**: 结束当前词法作用域或复合语句块。
- **L1124 EN**: Blank line separating nearby declarations or logic blocks.
  **L1124 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1125 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_abs(v128_t __a) {`.
  **L1125 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_abs(v128_t __a) {`。
- **L1126 EN**: Returns from the current function with `(v128_t)__builtin_wasm_abs_i32x4((__i32x4)__a)`.
  **L1126 CN**: 以 `(v128_t)__builtin_wasm_abs_i32x4((__i32x4)__a)` 从当前函数返回。
- **L1127 EN**: Closes the current lexical scope or compound statement.
  **L1127 CN**: 结束当前词法作用域或复合语句块。
- **L1128 EN**: Blank line separating nearby declarations or logic blocks.
  **L1128 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1129-1152

````c
static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_neg(v128_t __a) {
  return (v128_t)(-(__u32x4)__a);
}

static __inline__ bool __DEFAULT_FN_ATTRS wasm_i32x4_all_true(v128_t __a) {
  return __builtin_wasm_all_true_i32x4((__i32x4)__a);
}

static __inline__ uint32_t __DEFAULT_FN_ATTRS wasm_i32x4_bitmask(v128_t __a) {
  return __builtin_wasm_bitmask_i32x4((__i32x4)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_shl(v128_t __a,
                                                           uint32_t __b) {
  return (v128_t)((__i32x4)__a << (__b & 0x1F));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_shr(v128_t __a,
                                                           uint32_t __b) {
  return (v128_t)((__i32x4)__a >> (__b & 0x1F));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_shr(v128_t __a,
                                                           uint32_t __b) {
````
- **L1129 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_neg(v128_t __a) {`.
  **L1129 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_neg(v128_t __a) {`。
- **L1130 EN**: Returns from the current function with `(v128_t)(-(__u32x4)__a)`.
  **L1130 CN**: 以 `(v128_t)(-(__u32x4)__a)` 从当前函数返回。
- **L1131 EN**: Closes the current lexical scope or compound statement.
  **L1131 CN**: 结束当前词法作用域或复合语句块。
- **L1132 EN**: Blank line separating nearby declarations or logic blocks.
  **L1132 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1133 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ bool __DEFAULT_FN_ATTRS wasm_i32x4_all_true(v128_t __a) {`.
  **L1133 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ bool __DEFAULT_FN_ATTRS wasm_i32x4_all_true(v128_t __a) {`。
- **L1134 EN**: Returns from the current function with `__builtin_wasm_all_true_i32x4((__i32x4)__a)`.
  **L1134 CN**: 以 `__builtin_wasm_all_true_i32x4((__i32x4)__a)` 从当前函数返回。
- **L1135 EN**: Closes the current lexical scope or compound statement.
  **L1135 CN**: 结束当前词法作用域或复合语句块。
- **L1136 EN**: Blank line separating nearby declarations or logic blocks.
  **L1136 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1137 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ uint32_t __DEFAULT_FN_ATTRS wasm_i32x4_bitmask(v128_t __a) {`.
  **L1137 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ uint32_t __DEFAULT_FN_ATTRS wasm_i32x4_bitmask(v128_t __a) {`。
- **L1138 EN**: Returns from the current function with `__builtin_wasm_bitmask_i32x4((__i32x4)__a)`.
  **L1138 CN**: 以 `__builtin_wasm_bitmask_i32x4((__i32x4)__a)` 从当前函数返回。
- **L1139 EN**: Closes the current lexical scope or compound statement.
  **L1139 CN**: 结束当前词法作用域或复合语句块。
- **L1140 EN**: Blank line separating nearby declarations or logic blocks.
  **L1140 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_shl(v128_t __a,`.
  **L1141 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_shl(v128_t __a,`。
- **L1142 EN**: Continues the surrounding expression or declaration: `uint32_t __b) {`.
  **L1142 CN**: 继续构造周围的表达式或声明：`uint32_t __b) {`。
- **L1143 EN**: Returns from the current function with `(v128_t)((__i32x4)__a << (__b & 0x1F))`.
  **L1143 CN**: 以 `(v128_t)((__i32x4)__a << (__b & 0x1F))` 从当前函数返回。
- **L1144 EN**: Closes the current lexical scope or compound statement.
  **L1144 CN**: 结束当前词法作用域或复合语句块。
- **L1145 EN**: Blank line separating nearby declarations or logic blocks.
  **L1145 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_shr(v128_t __a,`.
  **L1146 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_shr(v128_t __a,`。
- **L1147 EN**: Continues the surrounding expression or declaration: `uint32_t __b) {`.
  **L1147 CN**: 继续构造周围的表达式或声明：`uint32_t __b) {`。
- **L1148 EN**: Returns from the current function with `(v128_t)((__i32x4)__a >> (__b & 0x1F))`.
  **L1148 CN**: 以 `(v128_t)((__i32x4)__a >> (__b & 0x1F))` 从当前函数返回。
- **L1149 EN**: Closes the current lexical scope or compound statement.
  **L1149 CN**: 结束当前词法作用域或复合语句块。
- **L1150 EN**: Blank line separating nearby declarations or logic blocks.
  **L1150 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_shr(v128_t __a,`.
  **L1151 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_shr(v128_t __a,`。
- **L1152 EN**: Continues the surrounding expression or declaration: `uint32_t __b) {`.
  **L1152 CN**: 继续构造周围的表达式或声明：`uint32_t __b) {`。

### Lines 1153-1176

````c
  return (v128_t)((__u32x4)__a >> (__b & 0x1F));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_add(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__u32x4)__a + (__u32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_sub(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__u32x4)__a - (__u32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_mul(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__u32x4)__a * (__u32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_min(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)__builtin_elementwise_min((__i32x4)__a, (__i32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_min(v128_t __a,
````
- **L1153 EN**: Returns from the current function with `(v128_t)((__u32x4)__a >> (__b & 0x1F))`.
  **L1153 CN**: 以 `(v128_t)((__u32x4)__a >> (__b & 0x1F))` 从当前函数返回。
- **L1154 EN**: Closes the current lexical scope or compound statement.
  **L1154 CN**: 结束当前词法作用域或复合语句块。
- **L1155 EN**: Blank line separating nearby declarations or logic blocks.
  **L1155 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_add(v128_t __a,`.
  **L1156 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_add(v128_t __a,`。
- **L1157 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1157 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1158 EN**: Returns from the current function with `(v128_t)((__u32x4)__a + (__u32x4)__b)`.
  **L1158 CN**: 以 `(v128_t)((__u32x4)__a + (__u32x4)__b)` 从当前函数返回。
- **L1159 EN**: Closes the current lexical scope or compound statement.
  **L1159 CN**: 结束当前词法作用域或复合语句块。
- **L1160 EN**: Blank line separating nearby declarations or logic blocks.
  **L1160 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_sub(v128_t __a,`.
  **L1161 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_sub(v128_t __a,`。
- **L1162 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1162 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1163 EN**: Returns from the current function with `(v128_t)((__u32x4)__a - (__u32x4)__b)`.
  **L1163 CN**: 以 `(v128_t)((__u32x4)__a - (__u32x4)__b)` 从当前函数返回。
- **L1164 EN**: Closes the current lexical scope or compound statement.
  **L1164 CN**: 结束当前词法作用域或复合语句块。
- **L1165 EN**: Blank line separating nearby declarations or logic blocks.
  **L1165 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_mul(v128_t __a,`.
  **L1166 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_mul(v128_t __a,`。
- **L1167 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1167 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1168 EN**: Returns from the current function with `(v128_t)((__u32x4)__a * (__u32x4)__b)`.
  **L1168 CN**: 以 `(v128_t)((__u32x4)__a * (__u32x4)__b)` 从当前函数返回。
- **L1169 EN**: Closes the current lexical scope or compound statement.
  **L1169 CN**: 结束当前词法作用域或复合语句块。
- **L1170 EN**: Blank line separating nearby declarations or logic blocks.
  **L1170 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_min(v128_t __a,`.
  **L1171 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_min(v128_t __a,`。
- **L1172 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1172 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1173 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_min((__i32x4)__a, (__i32x4)__b)`.
  **L1173 CN**: 以 `(v128_t)__builtin_elementwise_min((__i32x4)__a, (__i32x4)__b)` 从当前函数返回。
- **L1174 EN**: Closes the current lexical scope or compound statement.
  **L1174 CN**: 结束当前词法作用域或复合语句块。
- **L1175 EN**: Blank line separating nearby declarations or logic blocks.
  **L1175 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_min(v128_t __a,`.
  **L1176 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_min(v128_t __a,`。

### Lines 1177-1200

````c
                                                           v128_t __b) {
  return (v128_t)__builtin_elementwise_min((__u32x4)__a, (__u32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_max(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)__builtin_elementwise_max((__i32x4)__a, (__i32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_max(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)__builtin_elementwise_max((__u32x4)__a, (__u32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_dot_i16x8(v128_t __a,
                                                                 v128_t __b) {
  return (v128_t)__builtin_wasm_dot_s_i32x4_i16x8((__i16x8)__a, (__i16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_abs(v128_t __a) {
  return (v128_t)__builtin_wasm_abs_i64x2((__i64x2)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_neg(v128_t __a) {
````
- **L1177 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1177 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1178 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_min((__u32x4)__a, (__u32x4)__b)`.
  **L1178 CN**: 以 `(v128_t)__builtin_elementwise_min((__u32x4)__a, (__u32x4)__b)` 从当前函数返回。
- **L1179 EN**: Closes the current lexical scope or compound statement.
  **L1179 CN**: 结束当前词法作用域或复合语句块。
- **L1180 EN**: Blank line separating nearby declarations or logic blocks.
  **L1180 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_max(v128_t __a,`.
  **L1181 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_max(v128_t __a,`。
- **L1182 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1182 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1183 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_max((__i32x4)__a, (__i32x4)__b)`.
  **L1183 CN**: 以 `(v128_t)__builtin_elementwise_max((__i32x4)__a, (__i32x4)__b)` 从当前函数返回。
- **L1184 EN**: Closes the current lexical scope or compound statement.
  **L1184 CN**: 结束当前词法作用域或复合语句块。
- **L1185 EN**: Blank line separating nearby declarations or logic blocks.
  **L1185 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_max(v128_t __a,`.
  **L1186 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u32x4_max(v128_t __a,`。
- **L1187 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1187 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1188 EN**: Returns from the current function with `(v128_t)__builtin_elementwise_max((__u32x4)__a, (__u32x4)__b)`.
  **L1188 CN**: 以 `(v128_t)__builtin_elementwise_max((__u32x4)__a, (__u32x4)__b)` 从当前函数返回。
- **L1189 EN**: Closes the current lexical scope or compound statement.
  **L1189 CN**: 结束当前词法作用域或复合语句块。
- **L1190 EN**: Blank line separating nearby declarations or logic blocks.
  **L1190 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_dot_i16x8(v128_t __a,`.
  **L1191 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i32x4_dot_i16x8(v128_t __a,`。
- **L1192 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1192 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1193 EN**: Returns from the current function with `(v128_t)__builtin_wasm_dot_s_i32x4_i16x8((__i16x8)__a, (__i16x8)__b)`.
  **L1193 CN**: 以 `(v128_t)__builtin_wasm_dot_s_i32x4_i16x8((__i16x8)__a, (__i16x8)__b)` 从当前函数返回。
- **L1194 EN**: Closes the current lexical scope or compound statement.
  **L1194 CN**: 结束当前词法作用域或复合语句块。
- **L1195 EN**: Blank line separating nearby declarations or logic blocks.
  **L1195 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1196 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_abs(v128_t __a) {`.
  **L1196 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_abs(v128_t __a) {`。
- **L1197 EN**: Returns from the current function with `(v128_t)__builtin_wasm_abs_i64x2((__i64x2)__a)`.
  **L1197 CN**: 以 `(v128_t)__builtin_wasm_abs_i64x2((__i64x2)__a)` 从当前函数返回。
- **L1198 EN**: Closes the current lexical scope or compound statement.
  **L1198 CN**: 结束当前词法作用域或复合语句块。
- **L1199 EN**: Blank line separating nearby declarations or logic blocks.
  **L1199 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1200 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_neg(v128_t __a) {`.
  **L1200 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_neg(v128_t __a) {`。

### Lines 1201-1224

````c
  return (v128_t)(-(__u64x2)__a);
}

static __inline__ bool __DEFAULT_FN_ATTRS wasm_i64x2_all_true(v128_t __a) {
  return __builtin_wasm_all_true_i64x2((__i64x2)__a);
}

static __inline__ uint32_t __DEFAULT_FN_ATTRS wasm_i64x2_bitmask(v128_t __a) {
  return __builtin_wasm_bitmask_i64x2((__i64x2)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_shl(v128_t __a,
                                                           uint32_t __b) {
  return (v128_t)((__i64x2)__a << ((int64_t)__b & 0x3F));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_shr(v128_t __a,
                                                           uint32_t __b) {
  return (v128_t)((__i64x2)__a >> ((int64_t)__b & 0x3F));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u64x2_shr(v128_t __a,
                                                           uint32_t __b) {
  return (v128_t)((__u64x2)__a >> ((int64_t)__b & 0x3F));
````
- **L1201 EN**: Returns from the current function with `(v128_t)(-(__u64x2)__a)`.
  **L1201 CN**: 以 `(v128_t)(-(__u64x2)__a)` 从当前函数返回。
- **L1202 EN**: Closes the current lexical scope or compound statement.
  **L1202 CN**: 结束当前词法作用域或复合语句块。
- **L1203 EN**: Blank line separating nearby declarations or logic blocks.
  **L1203 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1204 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ bool __DEFAULT_FN_ATTRS wasm_i64x2_all_true(v128_t __a) {`.
  **L1204 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ bool __DEFAULT_FN_ATTRS wasm_i64x2_all_true(v128_t __a) {`。
- **L1205 EN**: Returns from the current function with `__builtin_wasm_all_true_i64x2((__i64x2)__a)`.
  **L1205 CN**: 以 `__builtin_wasm_all_true_i64x2((__i64x2)__a)` 从当前函数返回。
- **L1206 EN**: Closes the current lexical scope or compound statement.
  **L1206 CN**: 结束当前词法作用域或复合语句块。
- **L1207 EN**: Blank line separating nearby declarations or logic blocks.
  **L1207 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1208 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ uint32_t __DEFAULT_FN_ATTRS wasm_i64x2_bitmask(v128_t __a) {`.
  **L1208 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ uint32_t __DEFAULT_FN_ATTRS wasm_i64x2_bitmask(v128_t __a) {`。
- **L1209 EN**: Returns from the current function with `__builtin_wasm_bitmask_i64x2((__i64x2)__a)`.
  **L1209 CN**: 以 `__builtin_wasm_bitmask_i64x2((__i64x2)__a)` 从当前函数返回。
- **L1210 EN**: Closes the current lexical scope or compound statement.
  **L1210 CN**: 结束当前词法作用域或复合语句块。
- **L1211 EN**: Blank line separating nearby declarations or logic blocks.
  **L1211 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_shl(v128_t __a,`.
  **L1212 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_shl(v128_t __a,`。
- **L1213 EN**: Continues the surrounding expression or declaration: `uint32_t __b) {`.
  **L1213 CN**: 继续构造周围的表达式或声明：`uint32_t __b) {`。
- **L1214 EN**: Returns from the current function with `(v128_t)((__i64x2)__a << ((int64_t)__b & 0x3F))`.
  **L1214 CN**: 以 `(v128_t)((__i64x2)__a << ((int64_t)__b & 0x3F))` 从当前函数返回。
- **L1215 EN**: Closes the current lexical scope or compound statement.
  **L1215 CN**: 结束当前词法作用域或复合语句块。
- **L1216 EN**: Blank line separating nearby declarations or logic blocks.
  **L1216 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_shr(v128_t __a,`.
  **L1217 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_shr(v128_t __a,`。
- **L1218 EN**: Continues the surrounding expression or declaration: `uint32_t __b) {`.
  **L1218 CN**: 继续构造周围的表达式或声明：`uint32_t __b) {`。
- **L1219 EN**: Returns from the current function with `(v128_t)((__i64x2)__a >> ((int64_t)__b & 0x3F))`.
  **L1219 CN**: 以 `(v128_t)((__i64x2)__a >> ((int64_t)__b & 0x3F))` 从当前函数返回。
- **L1220 EN**: Closes the current lexical scope or compound statement.
  **L1220 CN**: 结束当前词法作用域或复合语句块。
- **L1221 EN**: Blank line separating nearby declarations or logic blocks.
  **L1221 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u64x2_shr(v128_t __a,`.
  **L1222 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_u64x2_shr(v128_t __a,`。
- **L1223 EN**: Continues the surrounding expression or declaration: `uint32_t __b) {`.
  **L1223 CN**: 继续构造周围的表达式或声明：`uint32_t __b) {`。
- **L1224 EN**: Returns from the current function with `(v128_t)((__u64x2)__a >> ((int64_t)__b & 0x3F))`.
  **L1224 CN**: 以 `(v128_t)((__u64x2)__a >> ((int64_t)__b & 0x3F))` 从当前函数返回。

### Lines 1225-1248

````c
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_add(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__u64x2)__a + (__u64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_sub(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__u64x2)__a - (__u64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_mul(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__u64x2)__a * (__u64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_abs(v128_t __a) {
  return (v128_t)__builtin_wasm_abs_f32x4((__f32x4)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_neg(v128_t __a) {
  return (v128_t)(-(__f32x4)__a);
}
````
- **L1225 EN**: Closes the current lexical scope or compound statement.
  **L1225 CN**: 结束当前词法作用域或复合语句块。
- **L1226 EN**: Blank line separating nearby declarations or logic blocks.
  **L1226 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_add(v128_t __a,`.
  **L1227 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_add(v128_t __a,`。
- **L1228 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1228 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1229 EN**: Returns from the current function with `(v128_t)((__u64x2)__a + (__u64x2)__b)`.
  **L1229 CN**: 以 `(v128_t)((__u64x2)__a + (__u64x2)__b)` 从当前函数返回。
- **L1230 EN**: Closes the current lexical scope or compound statement.
  **L1230 CN**: 结束当前词法作用域或复合语句块。
- **L1231 EN**: Blank line separating nearby declarations or logic blocks.
  **L1231 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_sub(v128_t __a,`.
  **L1232 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_sub(v128_t __a,`。
- **L1233 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1233 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1234 EN**: Returns from the current function with `(v128_t)((__u64x2)__a - (__u64x2)__b)`.
  **L1234 CN**: 以 `(v128_t)((__u64x2)__a - (__u64x2)__b)` 从当前函数返回。
- **L1235 EN**: Closes the current lexical scope or compound statement.
  **L1235 CN**: 结束当前词法作用域或复合语句块。
- **L1236 EN**: Blank line separating nearby declarations or logic blocks.
  **L1236 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_mul(v128_t __a,`.
  **L1237 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i64x2_mul(v128_t __a,`。
- **L1238 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1238 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1239 EN**: Returns from the current function with `(v128_t)((__u64x2)__a * (__u64x2)__b)`.
  **L1239 CN**: 以 `(v128_t)((__u64x2)__a * (__u64x2)__b)` 从当前函数返回。
- **L1240 EN**: Closes the current lexical scope or compound statement.
  **L1240 CN**: 结束当前词法作用域或复合语句块。
- **L1241 EN**: Blank line separating nearby declarations or logic blocks.
  **L1241 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1242 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_abs(v128_t __a) {`.
  **L1242 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_abs(v128_t __a) {`。
- **L1243 EN**: Returns from the current function with `(v128_t)__builtin_wasm_abs_f32x4((__f32x4)__a)`.
  **L1243 CN**: 以 `(v128_t)__builtin_wasm_abs_f32x4((__f32x4)__a)` 从当前函数返回。
- **L1244 EN**: Closes the current lexical scope or compound statement.
  **L1244 CN**: 结束当前词法作用域或复合语句块。
- **L1245 EN**: Blank line separating nearby declarations or logic blocks.
  **L1245 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1246 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_neg(v128_t __a) {`.
  **L1246 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_neg(v128_t __a) {`。
- **L1247 EN**: Returns from the current function with `(v128_t)(-(__f32x4)__a)`.
  **L1247 CN**: 以 `(v128_t)(-(__f32x4)__a)` 从当前函数返回。
- **L1248 EN**: Closes the current lexical scope or compound statement.
  **L1248 CN**: 结束当前词法作用域或复合语句块。

### Lines 1249-1272

````c

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_sqrt(v128_t __a) {
  return (v128_t)__builtin_wasm_sqrt_f32x4((__f32x4)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_ceil(v128_t __a) {
  return (v128_t)__builtin_wasm_ceil_f32x4((__f32x4)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_floor(v128_t __a) {
  return (v128_t)__builtin_wasm_floor_f32x4((__f32x4)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_trunc(v128_t __a) {
  return (v128_t)__builtin_wasm_trunc_f32x4((__f32x4)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_nearest(v128_t __a) {
  return (v128_t)__builtin_wasm_nearest_f32x4((__f32x4)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_add(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__f32x4)__a + (__f32x4)__b);
````
- **L1249 EN**: Blank line separating nearby declarations or logic blocks.
  **L1249 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1250 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_sqrt(v128_t __a) {`.
  **L1250 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_sqrt(v128_t __a) {`。
- **L1251 EN**: Returns from the current function with `(v128_t)__builtin_wasm_sqrt_f32x4((__f32x4)__a)`.
  **L1251 CN**: 以 `(v128_t)__builtin_wasm_sqrt_f32x4((__f32x4)__a)` 从当前函数返回。
- **L1252 EN**: Closes the current lexical scope or compound statement.
  **L1252 CN**: 结束当前词法作用域或复合语句块。
- **L1253 EN**: Blank line separating nearby declarations or logic blocks.
  **L1253 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1254 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_ceil(v128_t __a) {`.
  **L1254 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_ceil(v128_t __a) {`。
- **L1255 EN**: Returns from the current function with `(v128_t)__builtin_wasm_ceil_f32x4((__f32x4)__a)`.
  **L1255 CN**: 以 `(v128_t)__builtin_wasm_ceil_f32x4((__f32x4)__a)` 从当前函数返回。
- **L1256 EN**: Closes the current lexical scope or compound statement.
  **L1256 CN**: 结束当前词法作用域或复合语句块。
- **L1257 EN**: Blank line separating nearby declarations or logic blocks.
  **L1257 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1258 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_floor(v128_t __a) {`.
  **L1258 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_floor(v128_t __a) {`。
- **L1259 EN**: Returns from the current function with `(v128_t)__builtin_wasm_floor_f32x4((__f32x4)__a)`.
  **L1259 CN**: 以 `(v128_t)__builtin_wasm_floor_f32x4((__f32x4)__a)` 从当前函数返回。
- **L1260 EN**: Closes the current lexical scope or compound statement.
  **L1260 CN**: 结束当前词法作用域或复合语句块。
- **L1261 EN**: Blank line separating nearby declarations or logic blocks.
  **L1261 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1262 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_trunc(v128_t __a) {`.
  **L1262 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_trunc(v128_t __a) {`。
- **L1263 EN**: Returns from the current function with `(v128_t)__builtin_wasm_trunc_f32x4((__f32x4)__a)`.
  **L1263 CN**: 以 `(v128_t)__builtin_wasm_trunc_f32x4((__f32x4)__a)` 从当前函数返回。
- **L1264 EN**: Closes the current lexical scope or compound statement.
  **L1264 CN**: 结束当前词法作用域或复合语句块。
- **L1265 EN**: Blank line separating nearby declarations or logic blocks.
  **L1265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1266 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_nearest(v128_t __a) {`.
  **L1266 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_nearest(v128_t __a) {`。
- **L1267 EN**: Returns from the current function with `(v128_t)__builtin_wasm_nearest_f32x4((__f32x4)__a)`.
  **L1267 CN**: 以 `(v128_t)__builtin_wasm_nearest_f32x4((__f32x4)__a)` 从当前函数返回。
- **L1268 EN**: Closes the current lexical scope or compound statement.
  **L1268 CN**: 结束当前词法作用域或复合语句块。
- **L1269 EN**: Blank line separating nearby declarations or logic blocks.
  **L1269 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_add(v128_t __a,`.
  **L1270 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_add(v128_t __a,`。
- **L1271 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1271 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1272 EN**: Returns from the current function with `(v128_t)((__f32x4)__a + (__f32x4)__b)`.
  **L1272 CN**: 以 `(v128_t)((__f32x4)__a + (__f32x4)__b)` 从当前函数返回。

### Lines 1273-1296

````c
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_sub(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__f32x4)__a - (__f32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_mul(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__f32x4)__a * (__f32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_div(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__f32x4)__a / (__f32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_min(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)__builtin_wasm_min_f32x4((__f32x4)__a, (__f32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_max(v128_t __a,
                                                           v128_t __b) {
````
- **L1273 EN**: Closes the current lexical scope or compound statement.
  **L1273 CN**: 结束当前词法作用域或复合语句块。
- **L1274 EN**: Blank line separating nearby declarations or logic blocks.
  **L1274 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_sub(v128_t __a,`.
  **L1275 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_sub(v128_t __a,`。
- **L1276 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1276 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1277 EN**: Returns from the current function with `(v128_t)((__f32x4)__a - (__f32x4)__b)`.
  **L1277 CN**: 以 `(v128_t)((__f32x4)__a - (__f32x4)__b)` 从当前函数返回。
- **L1278 EN**: Closes the current lexical scope or compound statement.
  **L1278 CN**: 结束当前词法作用域或复合语句块。
- **L1279 EN**: Blank line separating nearby declarations or logic blocks.
  **L1279 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_mul(v128_t __a,`.
  **L1280 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_mul(v128_t __a,`。
- **L1281 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1281 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1282 EN**: Returns from the current function with `(v128_t)((__f32x4)__a * (__f32x4)__b)`.
  **L1282 CN**: 以 `(v128_t)((__f32x4)__a * (__f32x4)__b)` 从当前函数返回。
- **L1283 EN**: Closes the current lexical scope or compound statement.
  **L1283 CN**: 结束当前词法作用域或复合语句块。
- **L1284 EN**: Blank line separating nearby declarations or logic blocks.
  **L1284 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_div(v128_t __a,`.
  **L1285 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_div(v128_t __a,`。
- **L1286 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1286 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1287 EN**: Returns from the current function with `(v128_t)((__f32x4)__a / (__f32x4)__b)`.
  **L1287 CN**: 以 `(v128_t)((__f32x4)__a / (__f32x4)__b)` 从当前函数返回。
- **L1288 EN**: Closes the current lexical scope or compound statement.
  **L1288 CN**: 结束当前词法作用域或复合语句块。
- **L1289 EN**: Blank line separating nearby declarations or logic blocks.
  **L1289 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_min(v128_t __a,`.
  **L1290 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_min(v128_t __a,`。
- **L1291 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1291 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1292 EN**: Returns from the current function with `(v128_t)__builtin_wasm_min_f32x4((__f32x4)__a, (__f32x4)__b)`.
  **L1292 CN**: 以 `(v128_t)__builtin_wasm_min_f32x4((__f32x4)__a, (__f32x4)__b)` 从当前函数返回。
- **L1293 EN**: Closes the current lexical scope or compound statement.
  **L1293 CN**: 结束当前词法作用域或复合语句块。
- **L1294 EN**: Blank line separating nearby declarations or logic blocks.
  **L1294 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_max(v128_t __a,`.
  **L1295 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_max(v128_t __a,`。
- **L1296 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1296 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。

### Lines 1297-1320

````c
  return (v128_t)__builtin_wasm_max_f32x4((__f32x4)__a, (__f32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_pmin(v128_t __a,
                                                            v128_t __b) {
  return (v128_t)__builtin_wasm_pmin_f32x4((__f32x4)__a, (__f32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_pmax(v128_t __a,
                                                            v128_t __b) {
  return (v128_t)__builtin_wasm_pmax_f32x4((__f32x4)__a, (__f32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_abs(v128_t __a) {
  return (v128_t)__builtin_wasm_abs_f64x2((__f64x2)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_neg(v128_t __a) {
  return (v128_t)(-(__f64x2)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_sqrt(v128_t __a) {
  return (v128_t)__builtin_wasm_sqrt_f64x2((__f64x2)__a);
}
````
- **L1297 EN**: Returns from the current function with `(v128_t)__builtin_wasm_max_f32x4((__f32x4)__a, (__f32x4)__b)`.
  **L1297 CN**: 以 `(v128_t)__builtin_wasm_max_f32x4((__f32x4)__a, (__f32x4)__b)` 从当前函数返回。
- **L1298 EN**: Closes the current lexical scope or compound statement.
  **L1298 CN**: 结束当前词法作用域或复合语句块。
- **L1299 EN**: Blank line separating nearby declarations or logic blocks.
  **L1299 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_pmin(v128_t __a,`.
  **L1300 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_pmin(v128_t __a,`。
- **L1301 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1301 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1302 EN**: Returns from the current function with `(v128_t)__builtin_wasm_pmin_f32x4((__f32x4)__a, (__f32x4)__b)`.
  **L1302 CN**: 以 `(v128_t)__builtin_wasm_pmin_f32x4((__f32x4)__a, (__f32x4)__b)` 从当前函数返回。
- **L1303 EN**: Closes the current lexical scope or compound statement.
  **L1303 CN**: 结束当前词法作用域或复合语句块。
- **L1304 EN**: Blank line separating nearby declarations or logic blocks.
  **L1304 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_pmax(v128_t __a,`.
  **L1305 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f32x4_pmax(v128_t __a,`。
- **L1306 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1306 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1307 EN**: Returns from the current function with `(v128_t)__builtin_wasm_pmax_f32x4((__f32x4)__a, (__f32x4)__b)`.
  **L1307 CN**: 以 `(v128_t)__builtin_wasm_pmax_f32x4((__f32x4)__a, (__f32x4)__b)` 从当前函数返回。
- **L1308 EN**: Closes the current lexical scope or compound statement.
  **L1308 CN**: 结束当前词法作用域或复合语句块。
- **L1309 EN**: Blank line separating nearby declarations or logic blocks.
  **L1309 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1310 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_abs(v128_t __a) {`.
  **L1310 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_abs(v128_t __a) {`。
- **L1311 EN**: Returns from the current function with `(v128_t)__builtin_wasm_abs_f64x2((__f64x2)__a)`.
  **L1311 CN**: 以 `(v128_t)__builtin_wasm_abs_f64x2((__f64x2)__a)` 从当前函数返回。
- **L1312 EN**: Closes the current lexical scope or compound statement.
  **L1312 CN**: 结束当前词法作用域或复合语句块。
- **L1313 EN**: Blank line separating nearby declarations or logic blocks.
  **L1313 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1314 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_neg(v128_t __a) {`.
  **L1314 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_neg(v128_t __a) {`。
- **L1315 EN**: Returns from the current function with `(v128_t)(-(__f64x2)__a)`.
  **L1315 CN**: 以 `(v128_t)(-(__f64x2)__a)` 从当前函数返回。
- **L1316 EN**: Closes the current lexical scope or compound statement.
  **L1316 CN**: 结束当前词法作用域或复合语句块。
- **L1317 EN**: Blank line separating nearby declarations or logic blocks.
  **L1317 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1318 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_sqrt(v128_t __a) {`.
  **L1318 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_sqrt(v128_t __a) {`。
- **L1319 EN**: Returns from the current function with `(v128_t)__builtin_wasm_sqrt_f64x2((__f64x2)__a)`.
  **L1319 CN**: 以 `(v128_t)__builtin_wasm_sqrt_f64x2((__f64x2)__a)` 从当前函数返回。
- **L1320 EN**: Closes the current lexical scope or compound statement.
  **L1320 CN**: 结束当前词法作用域或复合语句块。

### Lines 1321-1344

````c

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_ceil(v128_t __a) {
  return (v128_t)__builtin_wasm_ceil_f64x2((__f64x2)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_floor(v128_t __a) {
  return (v128_t)__builtin_wasm_floor_f64x2((__f64x2)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_trunc(v128_t __a) {
  return (v128_t)__builtin_wasm_trunc_f64x2((__f64x2)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_nearest(v128_t __a) {
  return (v128_t)__builtin_wasm_nearest_f64x2((__f64x2)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_add(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__f64x2)__a + (__f64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_sub(v128_t __a,
                                                           v128_t __b) {
````
- **L1321 EN**: Blank line separating nearby declarations or logic blocks.
  **L1321 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1322 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_ceil(v128_t __a) {`.
  **L1322 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_ceil(v128_t __a) {`。
- **L1323 EN**: Returns from the current function with `(v128_t)__builtin_wasm_ceil_f64x2((__f64x2)__a)`.
  **L1323 CN**: 以 `(v128_t)__builtin_wasm_ceil_f64x2((__f64x2)__a)` 从当前函数返回。
- **L1324 EN**: Closes the current lexical scope or compound statement.
  **L1324 CN**: 结束当前词法作用域或复合语句块。
- **L1325 EN**: Blank line separating nearby declarations or logic blocks.
  **L1325 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1326 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_floor(v128_t __a) {`.
  **L1326 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_floor(v128_t __a) {`。
- **L1327 EN**: Returns from the current function with `(v128_t)__builtin_wasm_floor_f64x2((__f64x2)__a)`.
  **L1327 CN**: 以 `(v128_t)__builtin_wasm_floor_f64x2((__f64x2)__a)` 从当前函数返回。
- **L1328 EN**: Closes the current lexical scope or compound statement.
  **L1328 CN**: 结束当前词法作用域或复合语句块。
- **L1329 EN**: Blank line separating nearby declarations or logic blocks.
  **L1329 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1330 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_trunc(v128_t __a) {`.
  **L1330 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_trunc(v128_t __a) {`。
- **L1331 EN**: Returns from the current function with `(v128_t)__builtin_wasm_trunc_f64x2((__f64x2)__a)`.
  **L1331 CN**: 以 `(v128_t)__builtin_wasm_trunc_f64x2((__f64x2)__a)` 从当前函数返回。
- **L1332 EN**: Closes the current lexical scope or compound statement.
  **L1332 CN**: 结束当前词法作用域或复合语句块。
- **L1333 EN**: Blank line separating nearby declarations or logic blocks.
  **L1333 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1334 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_nearest(v128_t __a) {`.
  **L1334 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_nearest(v128_t __a) {`。
- **L1335 EN**: Returns from the current function with `(v128_t)__builtin_wasm_nearest_f64x2((__f64x2)__a)`.
  **L1335 CN**: 以 `(v128_t)__builtin_wasm_nearest_f64x2((__f64x2)__a)` 从当前函数返回。
- **L1336 EN**: Closes the current lexical scope or compound statement.
  **L1336 CN**: 结束当前词法作用域或复合语句块。
- **L1337 EN**: Blank line separating nearby declarations or logic blocks.
  **L1337 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_add(v128_t __a,`.
  **L1338 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_add(v128_t __a,`。
- **L1339 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1339 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1340 EN**: Returns from the current function with `(v128_t)((__f64x2)__a + (__f64x2)__b)`.
  **L1340 CN**: 以 `(v128_t)((__f64x2)__a + (__f64x2)__b)` 从当前函数返回。
- **L1341 EN**: Closes the current lexical scope or compound statement.
  **L1341 CN**: 结束当前词法作用域或复合语句块。
- **L1342 EN**: Blank line separating nearby declarations or logic blocks.
  **L1342 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_sub(v128_t __a,`.
  **L1343 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_sub(v128_t __a,`。
- **L1344 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1344 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。

### Lines 1345-1368

````c
  return (v128_t)((__f64x2)__a - (__f64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_mul(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__f64x2)__a * (__f64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_div(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)((__f64x2)__a / (__f64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_min(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)__builtin_wasm_min_f64x2((__f64x2)__a, (__f64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_max(v128_t __a,
                                                           v128_t __b) {
  return (v128_t)__builtin_wasm_max_f64x2((__f64x2)__a, (__f64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_pmin(v128_t __a,
````
- **L1345 EN**: Returns from the current function with `(v128_t)((__f64x2)__a - (__f64x2)__b)`.
  **L1345 CN**: 以 `(v128_t)((__f64x2)__a - (__f64x2)__b)` 从当前函数返回。
- **L1346 EN**: Closes the current lexical scope or compound statement.
  **L1346 CN**: 结束当前词法作用域或复合语句块。
- **L1347 EN**: Blank line separating nearby declarations or logic blocks.
  **L1347 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_mul(v128_t __a,`.
  **L1348 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_mul(v128_t __a,`。
- **L1349 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1349 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1350 EN**: Returns from the current function with `(v128_t)((__f64x2)__a * (__f64x2)__b)`.
  **L1350 CN**: 以 `(v128_t)((__f64x2)__a * (__f64x2)__b)` 从当前函数返回。
- **L1351 EN**: Closes the current lexical scope or compound statement.
  **L1351 CN**: 结束当前词法作用域或复合语句块。
- **L1352 EN**: Blank line separating nearby declarations or logic blocks.
  **L1352 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_div(v128_t __a,`.
  **L1353 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_div(v128_t __a,`。
- **L1354 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1354 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1355 EN**: Returns from the current function with `(v128_t)((__f64x2)__a / (__f64x2)__b)`.
  **L1355 CN**: 以 `(v128_t)((__f64x2)__a / (__f64x2)__b)` 从当前函数返回。
- **L1356 EN**: Closes the current lexical scope or compound statement.
  **L1356 CN**: 结束当前词法作用域或复合语句块。
- **L1357 EN**: Blank line separating nearby declarations or logic blocks.
  **L1357 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_min(v128_t __a,`.
  **L1358 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_min(v128_t __a,`。
- **L1359 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1359 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1360 EN**: Returns from the current function with `(v128_t)__builtin_wasm_min_f64x2((__f64x2)__a, (__f64x2)__b)`.
  **L1360 CN**: 以 `(v128_t)__builtin_wasm_min_f64x2((__f64x2)__a, (__f64x2)__b)` 从当前函数返回。
- **L1361 EN**: Closes the current lexical scope or compound statement.
  **L1361 CN**: 结束当前词法作用域或复合语句块。
- **L1362 EN**: Blank line separating nearby declarations or logic blocks.
  **L1362 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_max(v128_t __a,`.
  **L1363 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_max(v128_t __a,`。
- **L1364 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1364 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1365 EN**: Returns from the current function with `(v128_t)__builtin_wasm_max_f64x2((__f64x2)__a, (__f64x2)__b)`.
  **L1365 CN**: 以 `(v128_t)__builtin_wasm_max_f64x2((__f64x2)__a, (__f64x2)__b)` 从当前函数返回。
- **L1366 EN**: Closes the current lexical scope or compound statement.
  **L1366 CN**: 结束当前词法作用域或复合语句块。
- **L1367 EN**: Blank line separating nearby declarations or logic blocks.
  **L1367 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_pmin(v128_t __a,`.
  **L1368 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_pmin(v128_t __a,`。

### Lines 1369-1392

````c
                                                            v128_t __b) {
  return (v128_t)__builtin_wasm_pmin_f64x2((__f64x2)__a, (__f64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_pmax(v128_t __a,
                                                            v128_t __b) {
  return (v128_t)__builtin_wasm_pmax_f64x2((__f64x2)__a, (__f64x2)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i32x4_trunc_sat_f32x4(v128_t __a) {
  return (v128_t)__builtin_wasm_trunc_saturate_s_i32x4_f32x4((__f32x4)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u32x4_trunc_sat_f32x4(v128_t __a) {
  return (v128_t)__builtin_wasm_trunc_saturate_u_i32x4_f32x4((__f32x4)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_f32x4_convert_i32x4(v128_t __a) {
  return (v128_t) __builtin_convertvector((__i32x4)__a, __f32x4);
}

````
- **L1369 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1369 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1370 EN**: Returns from the current function with `(v128_t)__builtin_wasm_pmin_f64x2((__f64x2)__a, (__f64x2)__b)`.
  **L1370 CN**: 以 `(v128_t)__builtin_wasm_pmin_f64x2((__f64x2)__a, (__f64x2)__b)` 从当前函数返回。
- **L1371 EN**: Closes the current lexical scope or compound statement.
  **L1371 CN**: 结束当前词法作用域或复合语句块。
- **L1372 EN**: Blank line separating nearby declarations or logic blocks.
  **L1372 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_pmax(v128_t __a,`.
  **L1373 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_f64x2_pmax(v128_t __a,`。
- **L1374 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1374 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1375 EN**: Returns from the current function with `(v128_t)__builtin_wasm_pmax_f64x2((__f64x2)__a, (__f64x2)__b)`.
  **L1375 CN**: 以 `(v128_t)__builtin_wasm_pmax_f64x2((__f64x2)__a, (__f64x2)__b)` 从当前函数返回。
- **L1376 EN**: Closes the current lexical scope or compound statement.
  **L1376 CN**: 结束当前词法作用域或复合语句块。
- **L1377 EN**: Blank line separating nearby declarations or logic blocks.
  **L1377 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1378 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1378 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1379 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_trunc_sat_f32x4(v128_t __a) {`.
  **L1379 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_trunc_sat_f32x4(v128_t __a) {`。
- **L1380 EN**: Returns from the current function with `(v128_t)__builtin_wasm_trunc_saturate_s_i32x4_f32x4((__f32x4)__a)`.
  **L1380 CN**: 以 `(v128_t)__builtin_wasm_trunc_saturate_s_i32x4_f32x4((__f32x4)__a)` 从当前函数返回。
- **L1381 EN**: Closes the current lexical scope or compound statement.
  **L1381 CN**: 结束当前词法作用域或复合语句块。
- **L1382 EN**: Blank line separating nearby declarations or logic blocks.
  **L1382 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1383 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1383 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1384 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u32x4_trunc_sat_f32x4(v128_t __a) {`.
  **L1384 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u32x4_trunc_sat_f32x4(v128_t __a) {`。
- **L1385 EN**: Returns from the current function with `(v128_t)__builtin_wasm_trunc_saturate_u_i32x4_f32x4((__f32x4)__a)`.
  **L1385 CN**: 以 `(v128_t)__builtin_wasm_trunc_saturate_u_i32x4_f32x4((__f32x4)__a)` 从当前函数返回。
- **L1386 EN**: Closes the current lexical scope or compound statement.
  **L1386 CN**: 结束当前词法作用域或复合语句块。
- **L1387 EN**: Blank line separating nearby declarations or logic blocks.
  **L1387 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1388 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1388 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1389 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_f32x4_convert_i32x4(v128_t __a) {`.
  **L1389 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_f32x4_convert_i32x4(v128_t __a) {`。
- **L1390 EN**: Returns from the current function with `(v128_t) __builtin_convertvector((__i32x4)__a, __f32x4)`.
  **L1390 CN**: 以 `(v128_t) __builtin_convertvector((__i32x4)__a, __f32x4)` 从当前函数返回。
- **L1391 EN**: Closes the current lexical scope or compound statement.
  **L1391 CN**: 结束当前词法作用域或复合语句块。
- **L1392 EN**: Blank line separating nearby declarations or logic blocks.
  **L1392 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1393-1416

````c
static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_f32x4_convert_u32x4(v128_t __a) {
  return (v128_t) __builtin_convertvector((__u32x4)__a, __f32x4);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_f64x2_convert_low_i32x4(v128_t __a) {
  return (v128_t) __builtin_convertvector((__i32x2){__a[0], __a[1]}, __f64x2);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_f64x2_convert_low_u32x4(v128_t __a) {
  return (v128_t) __builtin_convertvector((__u32x2){__a[0], __a[1]}, __f64x2);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i32x4_trunc_sat_f64x2_zero(v128_t __a) {
  return (v128_t)__builtin_wasm_trunc_sat_s_zero_f64x2_i32x4((__f64x2)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u32x4_trunc_sat_f64x2_zero(v128_t __a) {
  return (v128_t)__builtin_wasm_trunc_sat_u_zero_f64x2_i32x4((__f64x2)__a);
}
````
- **L1393 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1393 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1394 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_f32x4_convert_u32x4(v128_t __a) {`.
  **L1394 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_f32x4_convert_u32x4(v128_t __a) {`。
- **L1395 EN**: Returns from the current function with `(v128_t) __builtin_convertvector((__u32x4)__a, __f32x4)`.
  **L1395 CN**: 以 `(v128_t) __builtin_convertvector((__u32x4)__a, __f32x4)` 从当前函数返回。
- **L1396 EN**: Closes the current lexical scope or compound statement.
  **L1396 CN**: 结束当前词法作用域或复合语句块。
- **L1397 EN**: Blank line separating nearby declarations or logic blocks.
  **L1397 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1398 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1398 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1399 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_f64x2_convert_low_i32x4(v128_t __a) {`.
  **L1399 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_f64x2_convert_low_i32x4(v128_t __a) {`。
- **L1400 EN**: Returns from the current function with `(v128_t) __builtin_convertvector((__i32x2){__a[0], __a[1]}, __f64x2)`.
  **L1400 CN**: 以 `(v128_t) __builtin_convertvector((__i32x2){__a[0], __a[1]}, __f64x2)` 从当前函数返回。
- **L1401 EN**: Closes the current lexical scope or compound statement.
  **L1401 CN**: 结束当前词法作用域或复合语句块。
- **L1402 EN**: Blank line separating nearby declarations or logic blocks.
  **L1402 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1403 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1403 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1404 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_f64x2_convert_low_u32x4(v128_t __a) {`.
  **L1404 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_f64x2_convert_low_u32x4(v128_t __a) {`。
- **L1405 EN**: Returns from the current function with `(v128_t) __builtin_convertvector((__u32x2){__a[0], __a[1]}, __f64x2)`.
  **L1405 CN**: 以 `(v128_t) __builtin_convertvector((__u32x2){__a[0], __a[1]}, __f64x2)` 从当前函数返回。
- **L1406 EN**: Closes the current lexical scope or compound statement.
  **L1406 CN**: 结束当前词法作用域或复合语句块。
- **L1407 EN**: Blank line separating nearby declarations or logic blocks.
  **L1407 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1408 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1408 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1409 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_trunc_sat_f64x2_zero(v128_t __a) {`.
  **L1409 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_trunc_sat_f64x2_zero(v128_t __a) {`。
- **L1410 EN**: Returns from the current function with `(v128_t)__builtin_wasm_trunc_sat_s_zero_f64x2_i32x4((__f64x2)__a)`.
  **L1410 CN**: 以 `(v128_t)__builtin_wasm_trunc_sat_s_zero_f64x2_i32x4((__f64x2)__a)` 从当前函数返回。
- **L1411 EN**: Closes the current lexical scope or compound statement.
  **L1411 CN**: 结束当前词法作用域或复合语句块。
- **L1412 EN**: Blank line separating nearby declarations or logic blocks.
  **L1412 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1413 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1413 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1414 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u32x4_trunc_sat_f64x2_zero(v128_t __a) {`.
  **L1414 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u32x4_trunc_sat_f64x2_zero(v128_t __a) {`。
- **L1415 EN**: Returns from the current function with `(v128_t)__builtin_wasm_trunc_sat_u_zero_f64x2_i32x4((__f64x2)__a)`.
  **L1415 CN**: 以 `(v128_t)__builtin_wasm_trunc_sat_u_zero_f64x2_i32x4((__f64x2)__a)` 从当前函数返回。
- **L1416 EN**: Closes the current lexical scope or compound statement.
  **L1416 CN**: 结束当前词法作用域或复合语句块。

### Lines 1417-1440

````c

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_f32x4_demote_f64x2_zero(v128_t __a) {
  return (v128_t) __builtin_convertvector(
      __builtin_shufflevector((__f64x2)__a, (__f64x2){0, 0}, 0, 1, 2, 3),
      __f32x4);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_f64x2_promote_low_f32x4(v128_t __a) {
  return (v128_t) __builtin_convertvector(
      (__f32x2){((__f32x4)__a)[0], ((__f32x4)__a)[1]}, __f64x2);
}

#define wasm_i8x16_shuffle(__a, __b, __c0, __c1, __c2, __c3, __c4, __c5, __c6, \
                           __c7, __c8, __c9, __c10, __c11, __c12, __c13,       \
                           __c14, __c15)                                       \
  ((v128_t)__builtin_wasm_shuffle_i8x16(                                       \
      (__i8x16)(__a), (__i8x16)(__b), __c0, __c1, __c2, __c3, __c4, __c5,      \
      __c6, __c7, __c8, __c9, __c10, __c11, __c12, __c13, __c14, __c15))

#define wasm_i16x8_shuffle(__a, __b, __c0, __c1, __c2, __c3, __c4, __c5, __c6, \
                           __c7)                                               \
  ((v128_t)__builtin_wasm_shuffle_i8x16(                                       \
````
- **L1417 EN**: Blank line separating nearby declarations or logic blocks.
  **L1417 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1418 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1418 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1419 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_f32x4_demote_f64x2_zero(v128_t __a) {`.
  **L1419 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_f32x4_demote_f64x2_zero(v128_t __a) {`。
- **L1420 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(`.
  **L1420 CN**: 以 `(v128_t) __builtin_convertvector(` 从当前函数返回。
- **L1421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__builtin_shufflevector((__f64x2)__a, (__f64x2){0, 0}, 0, 1, 2, 3),`.
  **L1421 CN**: 继续一个多行参数列表、初始化器或聚合项：`__builtin_shufflevector((__f64x2)__a, (__f64x2){0, 0}, 0, 1, 2, 3),`。
- **L1422 EN**: Adds a standalone statement or declaration: `__f32x4);`.
  **L1422 CN**: 添加一条独立语句或声明：`__f32x4);`。
- **L1423 EN**: Closes the current lexical scope or compound statement.
  **L1423 CN**: 结束当前词法作用域或复合语句块。
- **L1424 EN**: Blank line separating nearby declarations or logic blocks.
  **L1424 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1425 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1425 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1426 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_f64x2_promote_low_f32x4(v128_t __a) {`.
  **L1426 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_f64x2_promote_low_f32x4(v128_t __a) {`。
- **L1427 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(`.
  **L1427 CN**: 以 `(v128_t) __builtin_convertvector(` 从当前函数返回。
- **L1428 EN**: Executes a call or declaration centered on `statement`.
  **L1428 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1429 EN**: Closes the current lexical scope or compound statement.
  **L1429 CN**: 结束当前词法作用域或复合语句块。
- **L1430 EN**: Blank line separating nearby declarations or logic blocks.
  **L1430 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1431 EN**: Defines macro `wasm_i8x16_shuffle` for conditional compilation, shorthand, or API generation.
  **L1431 CN**: 定义宏 `wasm_i8x16_shuffle`，用于条件编译、简写或 API 生成。
- **L1432 EN**: Continues the surrounding expression or declaration: `__c7, __c8, __c9, __c10, __c11, __c12, __c13,       \`.
  **L1432 CN**: 继续构造周围的表达式或声明：`__c7, __c8, __c9, __c10, __c11, __c12, __c13,       \`。
- **L1433 EN**: Continues the surrounding expression or declaration: `__c14, __c15)                                       \`.
  **L1433 CN**: 继续构造周围的表达式或声明：`__c14, __c15)                                       \`。
- **L1434 EN**: Continues logic associated with callable symbol `__builtin_wasm_shuffle_i8x16`.
  **L1434 CN**: 继续与可调用符号 `__builtin_wasm_shuffle_i8x16` 相关的逻辑。
- **L1435 EN**: Continues the surrounding expression or declaration: `(__i8x16)(__a), (__i8x16)(__b), __c0, __c1, __c2, __c3, __c4, __c5,      \`.
  **L1435 CN**: 继续构造周围的表达式或声明：`(__i8x16)(__a), (__i8x16)(__b), __c0, __c1, __c2, __c3, __c4, __c5,      \`。
- **L1436 EN**: Continues the surrounding expression or declaration: `__c6, __c7, __c8, __c9, __c10, __c11, __c12, __c13, __c14, __c15))`.
  **L1436 CN**: 继续构造周围的表达式或声明：`__c6, __c7, __c8, __c9, __c10, __c11, __c12, __c13, __c14, __c15))`。
- **L1437 EN**: Blank line separating nearby declarations or logic blocks.
  **L1437 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1438 EN**: Defines macro `wasm_i16x8_shuffle` for conditional compilation, shorthand, or API generation.
  **L1438 CN**: 定义宏 `wasm_i16x8_shuffle`，用于条件编译、简写或 API 生成。
- **L1439 EN**: Continues the surrounding expression or declaration: `__c7)                                               \`.
  **L1439 CN**: 继续构造周围的表达式或声明：`__c7)                                               \`。
- **L1440 EN**: Continues logic associated with callable symbol `__builtin_wasm_shuffle_i8x16`.
  **L1440 CN**: 继续与可调用符号 `__builtin_wasm_shuffle_i8x16` 相关的逻辑。

### Lines 1441-1464

````c
      (__i8x16)(__a), (__i8x16)(__b), (__c0)*2, (__c0)*2 + 1, (__c1)*2,        \
      (__c1)*2 + 1, (__c2)*2, (__c2)*2 + 1, (__c3)*2, (__c3)*2 + 1, (__c4)*2,  \
      (__c4)*2 + 1, (__c5)*2, (__c5)*2 + 1, (__c6)*2, (__c6)*2 + 1, (__c7)*2,  \
      (__c7)*2 + 1))

#define wasm_i32x4_shuffle(__a, __b, __c0, __c1, __c2, __c3)                   \
  ((v128_t)__builtin_wasm_shuffle_i8x16(                                       \
      (__i8x16)(__a), (__i8x16)(__b), (__c0)*4, (__c0)*4 + 1, (__c0)*4 + 2,    \
      (__c0)*4 + 3, (__c1)*4, (__c1)*4 + 1, (__c1)*4 + 2, (__c1)*4 + 3,        \
      (__c2)*4, (__c2)*4 + 1, (__c2)*4 + 2, (__c2)*4 + 3, (__c3)*4,            \
      (__c3)*4 + 1, (__c3)*4 + 2, (__c3)*4 + 3))

#define wasm_i64x2_shuffle(__a, __b, __c0, __c1)                               \
  ((v128_t)__builtin_wasm_shuffle_i8x16(                                       \
      (__i8x16)(__a), (__i8x16)(__b), (__c0)*8, (__c0)*8 + 1, (__c0)*8 + 2,    \
      (__c0)*8 + 3, (__c0)*8 + 4, (__c0)*8 + 5, (__c0)*8 + 6, (__c0)*8 + 7,    \
      (__c1)*8, (__c1)*8 + 1, (__c1)*8 + 2, (__c1)*8 + 3, (__c1)*8 + 4,        \
      (__c1)*8 + 5, (__c1)*8 + 6, (__c1)*8 + 7))

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_swizzle(v128_t __a,
                                                               v128_t __b) {
  return (v128_t)__builtin_wasm_swizzle_i8x16((__i8x16)__a, (__i8x16)__b);
}

````
- **L1441 EN**: Continues the surrounding expression or declaration: `(__i8x16)(__a), (__i8x16)(__b), (__c0)*2, (__c0)*2 + 1, (__c1)*2,        \`.
  **L1441 CN**: 继续构造周围的表达式或声明：`(__i8x16)(__a), (__i8x16)(__b), (__c0)*2, (__c0)*2 + 1, (__c1)*2,        \`。
- **L1442 EN**: Continues the surrounding expression or declaration: `(__c1)*2 + 1, (__c2)*2, (__c2)*2 + 1, (__c3)*2, (__c3)*2 + 1, (__c4)*2,  \`.
  **L1442 CN**: 继续构造周围的表达式或声明：`(__c1)*2 + 1, (__c2)*2, (__c2)*2 + 1, (__c3)*2, (__c3)*2 + 1, (__c4)*2,  \`。
- **L1443 EN**: Continues the surrounding expression or declaration: `(__c4)*2 + 1, (__c5)*2, (__c5)*2 + 1, (__c6)*2, (__c6)*2 + 1, (__c7)*2,  \`.
  **L1443 CN**: 继续构造周围的表达式或声明：`(__c4)*2 + 1, (__c5)*2, (__c5)*2 + 1, (__c6)*2, (__c6)*2 + 1, (__c7)*2,  \`。
- **L1444 EN**: Continues the surrounding expression or declaration: `(__c7)*2 + 1))`.
  **L1444 CN**: 继续构造周围的表达式或声明：`(__c7)*2 + 1))`。
- **L1445 EN**: Blank line separating nearby declarations or logic blocks.
  **L1445 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1446 EN**: Defines macro `wasm_i32x4_shuffle(__a, __b, __c0, __c1, __c2, __c3)` for conditional compilation, shorthand, or API generation.
  **L1446 CN**: 定义宏 `wasm_i32x4_shuffle(__a, __b, __c0, __c1, __c2, __c3)`，用于条件编译、简写或 API 生成。
- **L1447 EN**: Continues logic associated with callable symbol `__builtin_wasm_shuffle_i8x16`.
  **L1447 CN**: 继续与可调用符号 `__builtin_wasm_shuffle_i8x16` 相关的逻辑。
- **L1448 EN**: Continues the surrounding expression or declaration: `(__i8x16)(__a), (__i8x16)(__b), (__c0)*4, (__c0)*4 + 1, (__c0)*4 + 2,    \`.
  **L1448 CN**: 继续构造周围的表达式或声明：`(__i8x16)(__a), (__i8x16)(__b), (__c0)*4, (__c0)*4 + 1, (__c0)*4 + 2,    \`。
- **L1449 EN**: Continues the surrounding expression or declaration: `(__c0)*4 + 3, (__c1)*4, (__c1)*4 + 1, (__c1)*4 + 2, (__c1)*4 + 3,        \`.
  **L1449 CN**: 继续构造周围的表达式或声明：`(__c0)*4 + 3, (__c1)*4, (__c1)*4 + 1, (__c1)*4 + 2, (__c1)*4 + 3,        \`。
- **L1450 EN**: Continues the surrounding expression or declaration: `(__c2)*4, (__c2)*4 + 1, (__c2)*4 + 2, (__c2)*4 + 3, (__c3)*4,            \`.
  **L1450 CN**: 继续构造周围的表达式或声明：`(__c2)*4, (__c2)*4 + 1, (__c2)*4 + 2, (__c2)*4 + 3, (__c3)*4,            \`。
- **L1451 EN**: Continues the surrounding expression or declaration: `(__c3)*4 + 1, (__c3)*4 + 2, (__c3)*4 + 3))`.
  **L1451 CN**: 继续构造周围的表达式或声明：`(__c3)*4 + 1, (__c3)*4 + 2, (__c3)*4 + 3))`。
- **L1452 EN**: Blank line separating nearby declarations or logic blocks.
  **L1452 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1453 EN**: Defines macro `wasm_i64x2_shuffle(__a, __b, __c0, __c1)` for conditional compilation, shorthand, or API generation.
  **L1453 CN**: 定义宏 `wasm_i64x2_shuffle(__a, __b, __c0, __c1)`，用于条件编译、简写或 API 生成。
- **L1454 EN**: Continues logic associated with callable symbol `__builtin_wasm_shuffle_i8x16`.
  **L1454 CN**: 继续与可调用符号 `__builtin_wasm_shuffle_i8x16` 相关的逻辑。
- **L1455 EN**: Continues the surrounding expression or declaration: `(__i8x16)(__a), (__i8x16)(__b), (__c0)*8, (__c0)*8 + 1, (__c0)*8 + 2,    \`.
  **L1455 CN**: 继续构造周围的表达式或声明：`(__i8x16)(__a), (__i8x16)(__b), (__c0)*8, (__c0)*8 + 1, (__c0)*8 + 2,    \`。
- **L1456 EN**: Continues the surrounding expression or declaration: `(__c0)*8 + 3, (__c0)*8 + 4, (__c0)*8 + 5, (__c0)*8 + 6, (__c0)*8 + 7,    \`.
  **L1456 CN**: 继续构造周围的表达式或声明：`(__c0)*8 + 3, (__c0)*8 + 4, (__c0)*8 + 5, (__c0)*8 + 6, (__c0)*8 + 7,    \`。
- **L1457 EN**: Continues the surrounding expression or declaration: `(__c1)*8, (__c1)*8 + 1, (__c1)*8 + 2, (__c1)*8 + 3, (__c1)*8 + 4,        \`.
  **L1457 CN**: 继续构造周围的表达式或声明：`(__c1)*8, (__c1)*8 + 1, (__c1)*8 + 2, (__c1)*8 + 3, (__c1)*8 + 4,        \`。
- **L1458 EN**: Continues the surrounding expression or declaration: `(__c1)*8 + 5, (__c1)*8 + 6, (__c1)*8 + 7))`.
  **L1458 CN**: 继续构造周围的表达式或声明：`(__c1)*8 + 5, (__c1)*8 + 6, (__c1)*8 + 7))`。
- **L1459 EN**: Blank line separating nearby declarations or logic blocks.
  **L1459 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1460 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_swizzle(v128_t __a,`.
  **L1460 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i8x16_swizzle(v128_t __a,`。
- **L1461 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1461 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1462 EN**: Returns from the current function with `(v128_t)__builtin_wasm_swizzle_i8x16((__i8x16)__a, (__i8x16)__b)`.
  **L1462 CN**: 以 `(v128_t)__builtin_wasm_swizzle_i8x16((__i8x16)__a, (__i8x16)__b)` 从当前函数返回。
- **L1463 EN**: Closes the current lexical scope or compound statement.
  **L1463 CN**: 结束当前词法作用域或复合语句块。
- **L1464 EN**: Blank line separating nearby declarations or logic blocks.
  **L1464 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1465-1488

````c
static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i8x16_narrow_i16x8(v128_t __a, v128_t __b) {
  return (v128_t)__builtin_wasm_narrow_s_i8x16_i16x8((__i16x8)__a,
                                                     (__i16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u8x16_narrow_i16x8(v128_t __a, v128_t __b) {
  return (v128_t)__builtin_wasm_narrow_u_i8x16_i16x8((__i16x8)__a,
                                                     (__i16x8)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i16x8_narrow_i32x4(v128_t __a, v128_t __b) {
  return (v128_t)__builtin_wasm_narrow_s_i16x8_i32x4((__i32x4)__a,
                                                     (__i32x4)__b);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u16x8_narrow_i32x4(v128_t __a, v128_t __b) {
  return (v128_t)__builtin_wasm_narrow_u_i16x8_i32x4((__i32x4)__a,
                                                     (__i32x4)__b);
}

````
- **L1465 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1465 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1466 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i8x16_narrow_i16x8(v128_t __a, v128_t __b) {`.
  **L1466 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i8x16_narrow_i16x8(v128_t __a, v128_t __b) {`。
- **L1467 EN**: Returns from the current function with `(v128_t)__builtin_wasm_narrow_s_i8x16_i16x8((__i16x8)__a,`.
  **L1467 CN**: 以 `(v128_t)__builtin_wasm_narrow_s_i8x16_i16x8((__i16x8)__a,` 从当前函数返回。
- **L1468 EN**: Executes a call or declaration centered on `statement`.
  **L1468 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1469 EN**: Closes the current lexical scope or compound statement.
  **L1469 CN**: 结束当前词法作用域或复合语句块。
- **L1470 EN**: Blank line separating nearby declarations or logic blocks.
  **L1470 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1471 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1471 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1472 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u8x16_narrow_i16x8(v128_t __a, v128_t __b) {`.
  **L1472 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u8x16_narrow_i16x8(v128_t __a, v128_t __b) {`。
- **L1473 EN**: Returns from the current function with `(v128_t)__builtin_wasm_narrow_u_i8x16_i16x8((__i16x8)__a,`.
  **L1473 CN**: 以 `(v128_t)__builtin_wasm_narrow_u_i8x16_i16x8((__i16x8)__a,` 从当前函数返回。
- **L1474 EN**: Executes a call or declaration centered on `statement`.
  **L1474 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1475 EN**: Closes the current lexical scope or compound statement.
  **L1475 CN**: 结束当前词法作用域或复合语句块。
- **L1476 EN**: Blank line separating nearby declarations or logic blocks.
  **L1476 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1477 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1477 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1478 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_narrow_i32x4(v128_t __a, v128_t __b) {`.
  **L1478 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_narrow_i32x4(v128_t __a, v128_t __b) {`。
- **L1479 EN**: Returns from the current function with `(v128_t)__builtin_wasm_narrow_s_i16x8_i32x4((__i32x4)__a,`.
  **L1479 CN**: 以 `(v128_t)__builtin_wasm_narrow_s_i16x8_i32x4((__i32x4)__a,` 从当前函数返回。
- **L1480 EN**: Executes a call or declaration centered on `statement`.
  **L1480 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1481 EN**: Closes the current lexical scope or compound statement.
  **L1481 CN**: 结束当前词法作用域或复合语句块。
- **L1482 EN**: Blank line separating nearby declarations or logic blocks.
  **L1482 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1483 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1483 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1484 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u16x8_narrow_i32x4(v128_t __a, v128_t __b) {`.
  **L1484 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u16x8_narrow_i32x4(v128_t __a, v128_t __b) {`。
- **L1485 EN**: Returns from the current function with `(v128_t)__builtin_wasm_narrow_u_i16x8_i32x4((__i32x4)__a,`.
  **L1485 CN**: 以 `(v128_t)__builtin_wasm_narrow_u_i16x8_i32x4((__i32x4)__a,` 从当前函数返回。
- **L1486 EN**: Executes a call or declaration centered on `statement`.
  **L1486 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1487 EN**: Closes the current lexical scope or compound statement.
  **L1487 CN**: 结束当前词法作用域或复合语句块。
- **L1488 EN**: Blank line separating nearby declarations or logic blocks.
  **L1488 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1489-1512

````c
static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i16x8_extend_low_i8x16(v128_t __a) {
  return (v128_t) __builtin_convertvector(
      (__i8x8){((__i8x16)__a)[0], ((__i8x16)__a)[1], ((__i8x16)__a)[2],
               ((__i8x16)__a)[3], ((__i8x16)__a)[4], ((__i8x16)__a)[5],
               ((__i8x16)__a)[6], ((__i8x16)__a)[7]},
      __i16x8);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i16x8_extend_high_i8x16(v128_t __a) {
  return (v128_t) __builtin_convertvector(
      (__i8x8){((__i8x16)__a)[8], ((__i8x16)__a)[9], ((__i8x16)__a)[10],
               ((__i8x16)__a)[11], ((__i8x16)__a)[12], ((__i8x16)__a)[13],
               ((__i8x16)__a)[14], ((__i8x16)__a)[15]},
      __i16x8);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u16x8_extend_low_u8x16(v128_t __a) {
  return (v128_t) __builtin_convertvector(
      (__u8x8){((__u8x16)__a)[0], ((__u8x16)__a)[1], ((__u8x16)__a)[2],
               ((__u8x16)__a)[3], ((__u8x16)__a)[4], ((__u8x16)__a)[5],
               ((__u8x16)__a)[6], ((__u8x16)__a)[7]},
````
- **L1489 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1489 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1490 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_extend_low_i8x16(v128_t __a) {`.
  **L1490 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_extend_low_i8x16(v128_t __a) {`。
- **L1491 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(`.
  **L1491 CN**: 以 `(v128_t) __builtin_convertvector(` 从当前函数返回。
- **L1492 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__i8x8){((__i8x16)__a)[0], ((__i8x16)__a)[1], ((__i8x16)__a)[2],`.
  **L1492 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__i8x8){((__i8x16)__a)[0], ((__i8x16)__a)[1], ((__i8x16)__a)[2],`。
- **L1493 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `((__i8x16)__a)[3], ((__i8x16)__a)[4], ((__i8x16)__a)[5],`.
  **L1493 CN**: 继续一个多行参数列表、初始化器或聚合项：`((__i8x16)__a)[3], ((__i8x16)__a)[4], ((__i8x16)__a)[5],`。
- **L1494 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `((__i8x16)__a)[6], ((__i8x16)__a)[7]},`.
  **L1494 CN**: 继续一个多行参数列表、初始化器或聚合项：`((__i8x16)__a)[6], ((__i8x16)__a)[7]},`。
- **L1495 EN**: Adds a standalone statement or declaration: `__i16x8);`.
  **L1495 CN**: 添加一条独立语句或声明：`__i16x8);`。
- **L1496 EN**: Closes the current lexical scope or compound statement.
  **L1496 CN**: 结束当前词法作用域或复合语句块。
- **L1497 EN**: Blank line separating nearby declarations or logic blocks.
  **L1497 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1498 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1498 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1499 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_extend_high_i8x16(v128_t __a) {`.
  **L1499 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_extend_high_i8x16(v128_t __a) {`。
- **L1500 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(`.
  **L1500 CN**: 以 `(v128_t) __builtin_convertvector(` 从当前函数返回。
- **L1501 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__i8x8){((__i8x16)__a)[8], ((__i8x16)__a)[9], ((__i8x16)__a)[10],`.
  **L1501 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__i8x8){((__i8x16)__a)[8], ((__i8x16)__a)[9], ((__i8x16)__a)[10],`。
- **L1502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `((__i8x16)__a)[11], ((__i8x16)__a)[12], ((__i8x16)__a)[13],`.
  **L1502 CN**: 继续一个多行参数列表、初始化器或聚合项：`((__i8x16)__a)[11], ((__i8x16)__a)[12], ((__i8x16)__a)[13],`。
- **L1503 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `((__i8x16)__a)[14], ((__i8x16)__a)[15]},`.
  **L1503 CN**: 继续一个多行参数列表、初始化器或聚合项：`((__i8x16)__a)[14], ((__i8x16)__a)[15]},`。
- **L1504 EN**: Adds a standalone statement or declaration: `__i16x8);`.
  **L1504 CN**: 添加一条独立语句或声明：`__i16x8);`。
- **L1505 EN**: Closes the current lexical scope or compound statement.
  **L1505 CN**: 结束当前词法作用域或复合语句块。
- **L1506 EN**: Blank line separating nearby declarations or logic blocks.
  **L1506 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1507 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1507 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1508 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u16x8_extend_low_u8x16(v128_t __a) {`.
  **L1508 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u16x8_extend_low_u8x16(v128_t __a) {`。
- **L1509 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(`.
  **L1509 CN**: 以 `(v128_t) __builtin_convertvector(` 从当前函数返回。
- **L1510 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__u8x8){((__u8x16)__a)[0], ((__u8x16)__a)[1], ((__u8x16)__a)[2],`.
  **L1510 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__u8x8){((__u8x16)__a)[0], ((__u8x16)__a)[1], ((__u8x16)__a)[2],`。
- **L1511 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `((__u8x16)__a)[3], ((__u8x16)__a)[4], ((__u8x16)__a)[5],`.
  **L1511 CN**: 继续一个多行参数列表、初始化器或聚合项：`((__u8x16)__a)[3], ((__u8x16)__a)[4], ((__u8x16)__a)[5],`。
- **L1512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `((__u8x16)__a)[6], ((__u8x16)__a)[7]},`.
  **L1512 CN**: 继续一个多行参数列表、初始化器或聚合项：`((__u8x16)__a)[6], ((__u8x16)__a)[7]},`。

### Lines 1513-1536

````c
      __u16x8);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u16x8_extend_high_u8x16(v128_t __a) {
  return (v128_t) __builtin_convertvector(
      (__u8x8){((__u8x16)__a)[8], ((__u8x16)__a)[9], ((__u8x16)__a)[10],
               ((__u8x16)__a)[11], ((__u8x16)__a)[12], ((__u8x16)__a)[13],
               ((__u8x16)__a)[14], ((__u8x16)__a)[15]},
      __u16x8);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i32x4_extend_low_i16x8(v128_t __a) {
  return (v128_t) __builtin_convertvector(
      (__i16x4){((__i16x8)__a)[0], ((__i16x8)__a)[1], ((__i16x8)__a)[2],
                ((__i16x8)__a)[3]},
      __i32x4);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i32x4_extend_high_i16x8(v128_t __a) {
  return (v128_t) __builtin_convertvector(
      (__i16x4){((__i16x8)__a)[4], ((__i16x8)__a)[5], ((__i16x8)__a)[6],
````
- **L1513 EN**: Adds a standalone statement or declaration: `__u16x8);`.
  **L1513 CN**: 添加一条独立语句或声明：`__u16x8);`。
- **L1514 EN**: Closes the current lexical scope or compound statement.
  **L1514 CN**: 结束当前词法作用域或复合语句块。
- **L1515 EN**: Blank line separating nearby declarations or logic blocks.
  **L1515 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1516 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1516 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1517 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u16x8_extend_high_u8x16(v128_t __a) {`.
  **L1517 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u16x8_extend_high_u8x16(v128_t __a) {`。
- **L1518 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(`.
  **L1518 CN**: 以 `(v128_t) __builtin_convertvector(` 从当前函数返回。
- **L1519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__u8x8){((__u8x16)__a)[8], ((__u8x16)__a)[9], ((__u8x16)__a)[10],`.
  **L1519 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__u8x8){((__u8x16)__a)[8], ((__u8x16)__a)[9], ((__u8x16)__a)[10],`。
- **L1520 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `((__u8x16)__a)[11], ((__u8x16)__a)[12], ((__u8x16)__a)[13],`.
  **L1520 CN**: 继续一个多行参数列表、初始化器或聚合项：`((__u8x16)__a)[11], ((__u8x16)__a)[12], ((__u8x16)__a)[13],`。
- **L1521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `((__u8x16)__a)[14], ((__u8x16)__a)[15]},`.
  **L1521 CN**: 继续一个多行参数列表、初始化器或聚合项：`((__u8x16)__a)[14], ((__u8x16)__a)[15]},`。
- **L1522 EN**: Adds a standalone statement or declaration: `__u16x8);`.
  **L1522 CN**: 添加一条独立语句或声明：`__u16x8);`。
- **L1523 EN**: Closes the current lexical scope or compound statement.
  **L1523 CN**: 结束当前词法作用域或复合语句块。
- **L1524 EN**: Blank line separating nearby declarations or logic blocks.
  **L1524 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1525 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1525 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1526 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_extend_low_i16x8(v128_t __a) {`.
  **L1526 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_extend_low_i16x8(v128_t __a) {`。
- **L1527 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(`.
  **L1527 CN**: 以 `(v128_t) __builtin_convertvector(` 从当前函数返回。
- **L1528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__i16x4){((__i16x8)__a)[0], ((__i16x8)__a)[1], ((__i16x8)__a)[2],`.
  **L1528 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__i16x4){((__i16x8)__a)[0], ((__i16x8)__a)[1], ((__i16x8)__a)[2],`。
- **L1529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `((__i16x8)__a)[3]},`.
  **L1529 CN**: 继续一个多行参数列表、初始化器或聚合项：`((__i16x8)__a)[3]},`。
- **L1530 EN**: Adds a standalone statement or declaration: `__i32x4);`.
  **L1530 CN**: 添加一条独立语句或声明：`__i32x4);`。
- **L1531 EN**: Closes the current lexical scope or compound statement.
  **L1531 CN**: 结束当前词法作用域或复合语句块。
- **L1532 EN**: Blank line separating nearby declarations or logic blocks.
  **L1532 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1533 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1533 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1534 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_extend_high_i16x8(v128_t __a) {`.
  **L1534 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_extend_high_i16x8(v128_t __a) {`。
- **L1535 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(`.
  **L1535 CN**: 以 `(v128_t) __builtin_convertvector(` 从当前函数返回。
- **L1536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__i16x4){((__i16x8)__a)[4], ((__i16x8)__a)[5], ((__i16x8)__a)[6],`.
  **L1536 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__i16x4){((__i16x8)__a)[4], ((__i16x8)__a)[5], ((__i16x8)__a)[6],`。

### Lines 1537-1560

````c
                ((__i16x8)__a)[7]},
      __i32x4);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u32x4_extend_low_u16x8(v128_t __a) {
  return (v128_t) __builtin_convertvector(
      (__u16x4){((__u16x8)__a)[0], ((__u16x8)__a)[1], ((__u16x8)__a)[2],
                ((__u16x8)__a)[3]},
      __u32x4);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u32x4_extend_high_u16x8(v128_t __a) {
  return (v128_t) __builtin_convertvector(
      (__u16x4){((__u16x8)__a)[4], ((__u16x8)__a)[5], ((__u16x8)__a)[6],
                ((__u16x8)__a)[7]},
      __u32x4);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i64x2_extend_low_i32x4(v128_t __a) {
  return (v128_t) __builtin_convertvector(
      (__i32x2){((__i32x4)__a)[0], ((__i32x4)__a)[1]}, __i64x2);
````
- **L1537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `((__i16x8)__a)[7]},`.
  **L1537 CN**: 继续一个多行参数列表、初始化器或聚合项：`((__i16x8)__a)[7]},`。
- **L1538 EN**: Adds a standalone statement or declaration: `__i32x4);`.
  **L1538 CN**: 添加一条独立语句或声明：`__i32x4);`。
- **L1539 EN**: Closes the current lexical scope or compound statement.
  **L1539 CN**: 结束当前词法作用域或复合语句块。
- **L1540 EN**: Blank line separating nearby declarations or logic blocks.
  **L1540 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1541 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1541 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1542 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u32x4_extend_low_u16x8(v128_t __a) {`.
  **L1542 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u32x4_extend_low_u16x8(v128_t __a) {`。
- **L1543 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(`.
  **L1543 CN**: 以 `(v128_t) __builtin_convertvector(` 从当前函数返回。
- **L1544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__u16x4){((__u16x8)__a)[0], ((__u16x8)__a)[1], ((__u16x8)__a)[2],`.
  **L1544 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__u16x4){((__u16x8)__a)[0], ((__u16x8)__a)[1], ((__u16x8)__a)[2],`。
- **L1545 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `((__u16x8)__a)[3]},`.
  **L1545 CN**: 继续一个多行参数列表、初始化器或聚合项：`((__u16x8)__a)[3]},`。
- **L1546 EN**: Adds a standalone statement or declaration: `__u32x4);`.
  **L1546 CN**: 添加一条独立语句或声明：`__u32x4);`。
- **L1547 EN**: Closes the current lexical scope or compound statement.
  **L1547 CN**: 结束当前词法作用域或复合语句块。
- **L1548 EN**: Blank line separating nearby declarations or logic blocks.
  **L1548 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1549 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1549 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1550 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u32x4_extend_high_u16x8(v128_t __a) {`.
  **L1550 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u32x4_extend_high_u16x8(v128_t __a) {`。
- **L1551 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(`.
  **L1551 CN**: 以 `(v128_t) __builtin_convertvector(` 从当前函数返回。
- **L1552 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__u16x4){((__u16x8)__a)[4], ((__u16x8)__a)[5], ((__u16x8)__a)[6],`.
  **L1552 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__u16x4){((__u16x8)__a)[4], ((__u16x8)__a)[5], ((__u16x8)__a)[6],`。
- **L1553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `((__u16x8)__a)[7]},`.
  **L1553 CN**: 继续一个多行参数列表、初始化器或聚合项：`((__u16x8)__a)[7]},`。
- **L1554 EN**: Adds a standalone statement or declaration: `__u32x4);`.
  **L1554 CN**: 添加一条独立语句或声明：`__u32x4);`。
- **L1555 EN**: Closes the current lexical scope or compound statement.
  **L1555 CN**: 结束当前词法作用域或复合语句块。
- **L1556 EN**: Blank line separating nearby declarations or logic blocks.
  **L1556 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1557 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1557 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1558 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i64x2_extend_low_i32x4(v128_t __a) {`.
  **L1558 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i64x2_extend_low_i32x4(v128_t __a) {`。
- **L1559 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(`.
  **L1559 CN**: 以 `(v128_t) __builtin_convertvector(` 从当前函数返回。
- **L1560 EN**: Executes a call or declaration centered on `statement`.
  **L1560 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1561-1584

````c
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i64x2_extend_high_i32x4(v128_t __a) {
  return (v128_t) __builtin_convertvector(
      (__i32x2){((__i32x4)__a)[2], ((__i32x4)__a)[3]}, __i64x2);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u64x2_extend_low_u32x4(v128_t __a) {
  return (v128_t) __builtin_convertvector(
      (__u32x2){((__u32x4)__a)[0], ((__u32x4)__a)[1]}, __u64x2);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u64x2_extend_high_u32x4(v128_t __a) {
  return (v128_t) __builtin_convertvector(
      (__u32x2){((__u32x4)__a)[2], ((__u32x4)__a)[3]}, __u64x2);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i16x8_extadd_pairwise_i8x16(v128_t __a) {
  return (v128_t)__builtin_wasm_extadd_pairwise_i8x16_s_i16x8((__i8x16)__a);
}
````
- **L1561 EN**: Closes the current lexical scope or compound statement.
  **L1561 CN**: 结束当前词法作用域或复合语句块。
- **L1562 EN**: Blank line separating nearby declarations or logic blocks.
  **L1562 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1563 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1563 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1564 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i64x2_extend_high_i32x4(v128_t __a) {`.
  **L1564 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i64x2_extend_high_i32x4(v128_t __a) {`。
- **L1565 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(`.
  **L1565 CN**: 以 `(v128_t) __builtin_convertvector(` 从当前函数返回。
- **L1566 EN**: Executes a call or declaration centered on `statement`.
  **L1566 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1567 EN**: Closes the current lexical scope or compound statement.
  **L1567 CN**: 结束当前词法作用域或复合语句块。
- **L1568 EN**: Blank line separating nearby declarations or logic blocks.
  **L1568 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1569 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1569 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1570 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u64x2_extend_low_u32x4(v128_t __a) {`.
  **L1570 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u64x2_extend_low_u32x4(v128_t __a) {`。
- **L1571 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(`.
  **L1571 CN**: 以 `(v128_t) __builtin_convertvector(` 从当前函数返回。
- **L1572 EN**: Executes a call or declaration centered on `statement`.
  **L1572 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1573 EN**: Closes the current lexical scope or compound statement.
  **L1573 CN**: 结束当前词法作用域或复合语句块。
- **L1574 EN**: Blank line separating nearby declarations or logic blocks.
  **L1574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1575 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1575 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1576 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u64x2_extend_high_u32x4(v128_t __a) {`.
  **L1576 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u64x2_extend_high_u32x4(v128_t __a) {`。
- **L1577 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(`.
  **L1577 CN**: 以 `(v128_t) __builtin_convertvector(` 从当前函数返回。
- **L1578 EN**: Executes a call or declaration centered on `statement`.
  **L1578 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1579 EN**: Closes the current lexical scope or compound statement.
  **L1579 CN**: 结束当前词法作用域或复合语句块。
- **L1580 EN**: Blank line separating nearby declarations or logic blocks.
  **L1580 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1581 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1581 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1582 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_extadd_pairwise_i8x16(v128_t __a) {`.
  **L1582 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_extadd_pairwise_i8x16(v128_t __a) {`。
- **L1583 EN**: Returns from the current function with `(v128_t)__builtin_wasm_extadd_pairwise_i8x16_s_i16x8((__i8x16)__a)`.
  **L1583 CN**: 以 `(v128_t)__builtin_wasm_extadd_pairwise_i8x16_s_i16x8((__i8x16)__a)` 从当前函数返回。
- **L1584 EN**: Closes the current lexical scope or compound statement.
  **L1584 CN**: 结束当前词法作用域或复合语句块。

### Lines 1585-1608

````c

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u16x8_extadd_pairwise_u8x16(v128_t __a) {
  return (v128_t)__builtin_wasm_extadd_pairwise_i8x16_u_i16x8((__u8x16)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i32x4_extadd_pairwise_i16x8(v128_t __a) {
  return (v128_t)__builtin_wasm_extadd_pairwise_i16x8_s_i32x4((__i16x8)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u32x4_extadd_pairwise_u16x8(v128_t __a) {
  return (v128_t)__builtin_wasm_extadd_pairwise_i16x8_u_i32x4((__u16x8)__a);
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i16x8_extmul_low_i8x16(v128_t __a, v128_t __b) {
  return (v128_t)((__i16x8)wasm_i16x8_extend_low_i8x16(__a) *
                  (__i16x8)wasm_i16x8_extend_low_i8x16(__b));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i16x8_extmul_high_i8x16(v128_t __a, v128_t __b) {
````
- **L1585 EN**: Blank line separating nearby declarations or logic blocks.
  **L1585 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1586 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1586 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1587 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u16x8_extadd_pairwise_u8x16(v128_t __a) {`.
  **L1587 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u16x8_extadd_pairwise_u8x16(v128_t __a) {`。
- **L1588 EN**: Returns from the current function with `(v128_t)__builtin_wasm_extadd_pairwise_i8x16_u_i16x8((__u8x16)__a)`.
  **L1588 CN**: 以 `(v128_t)__builtin_wasm_extadd_pairwise_i8x16_u_i16x8((__u8x16)__a)` 从当前函数返回。
- **L1589 EN**: Closes the current lexical scope or compound statement.
  **L1589 CN**: 结束当前词法作用域或复合语句块。
- **L1590 EN**: Blank line separating nearby declarations or logic blocks.
  **L1590 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1591 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1591 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1592 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_extadd_pairwise_i16x8(v128_t __a) {`.
  **L1592 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_extadd_pairwise_i16x8(v128_t __a) {`。
- **L1593 EN**: Returns from the current function with `(v128_t)__builtin_wasm_extadd_pairwise_i16x8_s_i32x4((__i16x8)__a)`.
  **L1593 CN**: 以 `(v128_t)__builtin_wasm_extadd_pairwise_i16x8_s_i32x4((__i16x8)__a)` 从当前函数返回。
- **L1594 EN**: Closes the current lexical scope or compound statement.
  **L1594 CN**: 结束当前词法作用域或复合语句块。
- **L1595 EN**: Blank line separating nearby declarations or logic blocks.
  **L1595 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1596 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1596 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1597 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u32x4_extadd_pairwise_u16x8(v128_t __a) {`.
  **L1597 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u32x4_extadd_pairwise_u16x8(v128_t __a) {`。
- **L1598 EN**: Returns from the current function with `(v128_t)__builtin_wasm_extadd_pairwise_i16x8_u_i32x4((__u16x8)__a)`.
  **L1598 CN**: 以 `(v128_t)__builtin_wasm_extadd_pairwise_i16x8_u_i32x4((__u16x8)__a)` 从当前函数返回。
- **L1599 EN**: Closes the current lexical scope or compound statement.
  **L1599 CN**: 结束当前词法作用域或复合语句块。
- **L1600 EN**: Blank line separating nearby declarations or logic blocks.
  **L1600 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1601 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1601 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1602 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_extmul_low_i8x16(v128_t __a, v128_t __b) {`.
  **L1602 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_extmul_low_i8x16(v128_t __a, v128_t __b) {`。
- **L1603 EN**: Returns from the current function with `(v128_t)((__i16x8)wasm_i16x8_extend_low_i8x16(__a) *`.
  **L1603 CN**: 以 `(v128_t)((__i16x8)wasm_i16x8_extend_low_i8x16(__a) *` 从当前函数返回。
- **L1604 EN**: Executes a call or declaration centered on `statement`.
  **L1604 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1605 EN**: Closes the current lexical scope or compound statement.
  **L1605 CN**: 结束当前词法作用域或复合语句块。
- **L1606 EN**: Blank line separating nearby declarations or logic blocks.
  **L1606 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1607 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1607 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1608 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_extmul_high_i8x16(v128_t __a, v128_t __b) {`.
  **L1608 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_extmul_high_i8x16(v128_t __a, v128_t __b) {`。

### Lines 1609-1632

````c
  return (v128_t)((__i16x8)wasm_i16x8_extend_high_i8x16(__a) *
                  (__i16x8)wasm_i16x8_extend_high_i8x16(__b));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u16x8_extmul_low_u8x16(v128_t __a, v128_t __b) {
  return (v128_t)((__u16x8)wasm_u16x8_extend_low_u8x16(__a) *
                  (__u16x8)wasm_u16x8_extend_low_u8x16(__b));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u16x8_extmul_high_u8x16(v128_t __a, v128_t __b) {
  return (v128_t)((__u16x8)wasm_u16x8_extend_high_u8x16(__a) *
                  (__u16x8)wasm_u16x8_extend_high_u8x16(__b));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i32x4_extmul_low_i16x8(v128_t __a, v128_t __b) {
  return (v128_t)((__i32x4)wasm_i32x4_extend_low_i16x8(__a) *
                  (__i32x4)wasm_i32x4_extend_low_i16x8(__b));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i32x4_extmul_high_i16x8(v128_t __a, v128_t __b) {
````
- **L1609 EN**: Returns from the current function with `(v128_t)((__i16x8)wasm_i16x8_extend_high_i8x16(__a) *`.
  **L1609 CN**: 以 `(v128_t)((__i16x8)wasm_i16x8_extend_high_i8x16(__a) *` 从当前函数返回。
- **L1610 EN**: Executes a call or declaration centered on `statement`.
  **L1610 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1611 EN**: Closes the current lexical scope or compound statement.
  **L1611 CN**: 结束当前词法作用域或复合语句块。
- **L1612 EN**: Blank line separating nearby declarations or logic blocks.
  **L1612 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1613 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1613 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1614 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u16x8_extmul_low_u8x16(v128_t __a, v128_t __b) {`.
  **L1614 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u16x8_extmul_low_u8x16(v128_t __a, v128_t __b) {`。
- **L1615 EN**: Returns from the current function with `(v128_t)((__u16x8)wasm_u16x8_extend_low_u8x16(__a) *`.
  **L1615 CN**: 以 `(v128_t)((__u16x8)wasm_u16x8_extend_low_u8x16(__a) *` 从当前函数返回。
- **L1616 EN**: Executes a call or declaration centered on `statement`.
  **L1616 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1617 EN**: Closes the current lexical scope or compound statement.
  **L1617 CN**: 结束当前词法作用域或复合语句块。
- **L1618 EN**: Blank line separating nearby declarations or logic blocks.
  **L1618 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1619 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1619 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1620 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u16x8_extmul_high_u8x16(v128_t __a, v128_t __b) {`.
  **L1620 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u16x8_extmul_high_u8x16(v128_t __a, v128_t __b) {`。
- **L1621 EN**: Returns from the current function with `(v128_t)((__u16x8)wasm_u16x8_extend_high_u8x16(__a) *`.
  **L1621 CN**: 以 `(v128_t)((__u16x8)wasm_u16x8_extend_high_u8x16(__a) *` 从当前函数返回。
- **L1622 EN**: Executes a call or declaration centered on `statement`.
  **L1622 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1623 EN**: Closes the current lexical scope or compound statement.
  **L1623 CN**: 结束当前词法作用域或复合语句块。
- **L1624 EN**: Blank line separating nearby declarations or logic blocks.
  **L1624 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1625 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1625 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1626 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_extmul_low_i16x8(v128_t __a, v128_t __b) {`.
  **L1626 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_extmul_low_i16x8(v128_t __a, v128_t __b) {`。
- **L1627 EN**: Returns from the current function with `(v128_t)((__i32x4)wasm_i32x4_extend_low_i16x8(__a) *`.
  **L1627 CN**: 以 `(v128_t)((__i32x4)wasm_i32x4_extend_low_i16x8(__a) *` 从当前函数返回。
- **L1628 EN**: Executes a call or declaration centered on `statement`.
  **L1628 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1629 EN**: Closes the current lexical scope or compound statement.
  **L1629 CN**: 结束当前词法作用域或复合语句块。
- **L1630 EN**: Blank line separating nearby declarations or logic blocks.
  **L1630 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1631 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1631 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1632 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_extmul_high_i16x8(v128_t __a, v128_t __b) {`.
  **L1632 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_extmul_high_i16x8(v128_t __a, v128_t __b) {`。

### Lines 1633-1656

````c
  return (v128_t)((__i32x4)wasm_i32x4_extend_high_i16x8(__a) *
                  (__i32x4)wasm_i32x4_extend_high_i16x8(__b));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u32x4_extmul_low_u16x8(v128_t __a, v128_t __b) {
  return (v128_t)((__u32x4)wasm_u32x4_extend_low_u16x8(__a) *
                  (__u32x4)wasm_u32x4_extend_low_u16x8(__b));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u32x4_extmul_high_u16x8(v128_t __a, v128_t __b) {
  return (v128_t)((__u32x4)wasm_u32x4_extend_high_u16x8(__a) *
                  (__u32x4)wasm_u32x4_extend_high_u16x8(__b));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i64x2_extmul_low_i32x4(v128_t __a, v128_t __b) {
  return (v128_t)((__i64x2)wasm_i64x2_extend_low_i32x4(__a) *
                  (__i64x2)wasm_i64x2_extend_low_i32x4(__b));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_i64x2_extmul_high_i32x4(v128_t __a, v128_t __b) {
````
- **L1633 EN**: Returns from the current function with `(v128_t)((__i32x4)wasm_i32x4_extend_high_i16x8(__a) *`.
  **L1633 CN**: 以 `(v128_t)((__i32x4)wasm_i32x4_extend_high_i16x8(__a) *` 从当前函数返回。
- **L1634 EN**: Executes a call or declaration centered on `statement`.
  **L1634 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1635 EN**: Closes the current lexical scope or compound statement.
  **L1635 CN**: 结束当前词法作用域或复合语句块。
- **L1636 EN**: Blank line separating nearby declarations or logic blocks.
  **L1636 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1637 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1637 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1638 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u32x4_extmul_low_u16x8(v128_t __a, v128_t __b) {`.
  **L1638 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u32x4_extmul_low_u16x8(v128_t __a, v128_t __b) {`。
- **L1639 EN**: Returns from the current function with `(v128_t)((__u32x4)wasm_u32x4_extend_low_u16x8(__a) *`.
  **L1639 CN**: 以 `(v128_t)((__u32x4)wasm_u32x4_extend_low_u16x8(__a) *` 从当前函数返回。
- **L1640 EN**: Executes a call or declaration centered on `statement`.
  **L1640 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1641 EN**: Closes the current lexical scope or compound statement.
  **L1641 CN**: 结束当前词法作用域或复合语句块。
- **L1642 EN**: Blank line separating nearby declarations or logic blocks.
  **L1642 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1643 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1643 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1644 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u32x4_extmul_high_u16x8(v128_t __a, v128_t __b) {`.
  **L1644 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u32x4_extmul_high_u16x8(v128_t __a, v128_t __b) {`。
- **L1645 EN**: Returns from the current function with `(v128_t)((__u32x4)wasm_u32x4_extend_high_u16x8(__a) *`.
  **L1645 CN**: 以 `(v128_t)((__u32x4)wasm_u32x4_extend_high_u16x8(__a) *` 从当前函数返回。
- **L1646 EN**: Executes a call or declaration centered on `statement`.
  **L1646 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1647 EN**: Closes the current lexical scope or compound statement.
  **L1647 CN**: 结束当前词法作用域或复合语句块。
- **L1648 EN**: Blank line separating nearby declarations or logic blocks.
  **L1648 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1649 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1649 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1650 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i64x2_extmul_low_i32x4(v128_t __a, v128_t __b) {`.
  **L1650 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i64x2_extmul_low_i32x4(v128_t __a, v128_t __b) {`。
- **L1651 EN**: Returns from the current function with `(v128_t)((__i64x2)wasm_i64x2_extend_low_i32x4(__a) *`.
  **L1651 CN**: 以 `(v128_t)((__i64x2)wasm_i64x2_extend_low_i32x4(__a) *` 从当前函数返回。
- **L1652 EN**: Executes a call or declaration centered on `statement`.
  **L1652 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1653 EN**: Closes the current lexical scope or compound statement.
  **L1653 CN**: 结束当前词法作用域或复合语句块。
- **L1654 EN**: Blank line separating nearby declarations or logic blocks.
  **L1654 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1655 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1655 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1656 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i64x2_extmul_high_i32x4(v128_t __a, v128_t __b) {`.
  **L1656 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i64x2_extmul_high_i32x4(v128_t __a, v128_t __b) {`。

### Lines 1657-1680

````c
  return (v128_t)((__i64x2)wasm_i64x2_extend_high_i32x4(__a) *
                  (__i64x2)wasm_i64x2_extend_high_i32x4(__b));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u64x2_extmul_low_u32x4(v128_t __a, v128_t __b) {
  return (v128_t)((__u64x2)wasm_u64x2_extend_low_u32x4(__a) *
                  (__u64x2)wasm_u64x2_extend_low_u32x4(__b));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS
wasm_u64x2_extmul_high_u32x4(v128_t __a, v128_t __b) {
  return (v128_t)((__u64x2)wasm_u64x2_extend_high_u32x4(__a) *
                  (__u64x2)wasm_u64x2_extend_high_u32x4(__b));
}

static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_q15mulr_sat(v128_t __a,
                                                                   v128_t __b) {
  return (v128_t)__builtin_wasm_q15mulr_sat_s_i16x8((__i16x8)__a, (__i16x8)__b);
}

// Old intrinsic names supported to ease transitioning to the standard names. Do
// not use these; they will be removed in the near future.

````
- **L1657 EN**: Returns from the current function with `(v128_t)((__i64x2)wasm_i64x2_extend_high_i32x4(__a) *`.
  **L1657 CN**: 以 `(v128_t)((__i64x2)wasm_i64x2_extend_high_i32x4(__a) *` 从当前函数返回。
- **L1658 EN**: Executes a call or declaration centered on `statement`.
  **L1658 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1659 EN**: Closes the current lexical scope or compound statement.
  **L1659 CN**: 结束当前词法作用域或复合语句块。
- **L1660 EN**: Blank line separating nearby declarations or logic blocks.
  **L1660 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1661 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1661 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1662 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u64x2_extmul_low_u32x4(v128_t __a, v128_t __b) {`.
  **L1662 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u64x2_extmul_low_u32x4(v128_t __a, v128_t __b) {`。
- **L1663 EN**: Returns from the current function with `(v128_t)((__u64x2)wasm_u64x2_extend_low_u32x4(__a) *`.
  **L1663 CN**: 以 `(v128_t)((__u64x2)wasm_u64x2_extend_low_u32x4(__a) *` 从当前函数返回。
- **L1664 EN**: Executes a call or declaration centered on `statement`.
  **L1664 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1665 EN**: Closes the current lexical scope or compound statement.
  **L1665 CN**: 结束当前词法作用域或复合语句块。
- **L1666 EN**: Blank line separating nearby declarations or logic blocks.
  **L1666 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1667 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __DEFAULT_FN_ATTRS`.
  **L1667 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __DEFAULT_FN_ATTRS`。
- **L1668 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u64x2_extmul_high_u32x4(v128_t __a, v128_t __b) {`.
  **L1668 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u64x2_extmul_high_u32x4(v128_t __a, v128_t __b) {`。
- **L1669 EN**: Returns from the current function with `(v128_t)((__u64x2)wasm_u64x2_extend_high_u32x4(__a) *`.
  **L1669 CN**: 以 `(v128_t)((__u64x2)wasm_u64x2_extend_high_u32x4(__a) *` 从当前函数返回。
- **L1670 EN**: Executes a call or declaration centered on `statement`.
  **L1670 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1671 EN**: Closes the current lexical scope or compound statement.
  **L1671 CN**: 结束当前词法作用域或复合语句块。
- **L1672 EN**: Blank line separating nearby declarations or logic blocks.
  **L1672 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1673 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_q15mulr_sat(v128_t __a,`.
  **L1673 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __DEFAULT_FN_ATTRS wasm_i16x8_q15mulr_sat(v128_t __a,`。
- **L1674 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1674 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1675 EN**: Returns from the current function with `(v128_t)__builtin_wasm_q15mulr_sat_s_i16x8((__i16x8)__a, (__i16x8)__b)`.
  **L1675 CN**: 以 `(v128_t)__builtin_wasm_q15mulr_sat_s_i16x8((__i16x8)__a, (__i16x8)__b)` 从当前函数返回。
- **L1676 EN**: Closes the current lexical scope or compound statement.
  **L1676 CN**: 结束当前词法作用域或复合语句块。
- **L1677 EN**: Blank line separating nearby declarations or logic blocks.
  **L1677 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1678 EN**: Comment explains nearby logic, constraints, or intent: `Old intrinsic names supported to ease transitioning to the standard names. Do`.
  **L1678 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Old intrinsic names supported to ease transitioning to the standard names. Do`。
- **L1679 EN**: Comment explains nearby logic, constraints, or intent: `not use these; they will be removed in the near future.`.
  **L1679 CN**: 注释解释附近代码的逻辑、约束或设计意图：`not use these; they will be removed in the near future.`。
- **L1680 EN**: Blank line separating nearby declarations or logic blocks.
  **L1680 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1681-1704

````c
#define __DEPRECATED_FN_ATTRS(__replacement)                                   \
  __DEFAULT_FN_ATTRS __attribute__(                                            \
      (deprecated("use " __replacement " instead", __replacement)))

#define __WASM_STR(X) #X

#ifdef __DEPRECATED
#define __DEPRECATED_WASM_MACRO(__name, __replacement)                         \
  _Pragma(__WASM_STR(GCC warning(                                              \
      "'" __name "' is deprecated: use '" __replacement "' instead")))
#else
#define __DEPRECATED_WASM_MACRO(__name, __replacement)
#endif

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_v128_load8_splat")
wasm_v8x16_load_splat(const void *__mem) {
  return wasm_v128_load8_splat(__mem);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_v128_load16_splat")
wasm_v16x8_load_splat(const void *__mem) {
  return wasm_v128_load16_splat(__mem);
}

````
- **L1681 EN**: Defines macro `__DEPRECATED_FN_ATTRS(__replacement)` for conditional compilation, shorthand, or API generation.
  **L1681 CN**: 定义宏 `__DEPRECATED_FN_ATTRS(__replacement)`，用于条件编译、简写或 API 生成。
- **L1682 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__DEFAULT_FN_ATTRS __attribute__(                                            \`.
  **L1682 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__DEFAULT_FN_ATTRS __attribute__(                                            \`。
- **L1683 EN**: Continues logic associated with callable symbol `deprecated`.
  **L1683 CN**: 继续与可调用符号 `deprecated` 相关的逻辑。
- **L1684 EN**: Blank line separating nearby declarations or logic blocks.
  **L1684 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1685 EN**: Defines macro `__WASM_STR(X)` for conditional compilation, shorthand, or API generation.
  **L1685 CN**: 定义宏 `__WASM_STR(X)`，用于条件编译、简写或 API 生成。
- **L1686 EN**: Blank line separating nearby declarations or logic blocks.
  **L1686 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1687 EN**: Starts a preprocessor conditional block: `#ifdef __DEPRECATED`.
  **L1687 CN**: 开始一个预处理条件块：`#ifdef __DEPRECATED`。
- **L1688 EN**: Defines macro `__DEPRECATED_WASM_MACRO(__name, __replacement)` for conditional compilation, shorthand, or API generation.
  **L1688 CN**: 定义宏 `__DEPRECATED_WASM_MACRO(__name, __replacement)`，用于条件编译、简写或 API 生成。
- **L1689 EN**: Continues logic associated with callable symbol `_Pragma`.
  **L1689 CN**: 继续与可调用符号 `_Pragma` 相关的逻辑。
- **L1690 EN**: Continues the surrounding expression or declaration: `"'" __name "' is deprecated: use '" __replacement "' instead")))`.
  **L1690 CN**: 继续构造周围的表达式或声明：`"'" __name "' is deprecated: use '" __replacement "' instead")))`。
- **L1691 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1691 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1692 EN**: Defines macro `__DEPRECATED_WASM_MACRO(__name, __replacement)` for conditional compilation, shorthand, or API generation.
  **L1692 CN**: 定义宏 `__DEPRECATED_WASM_MACRO(__name, __replacement)`，用于条件编译、简写或 API 生成。
- **L1693 EN**: Closes the current preprocessor conditional block.
  **L1693 CN**: 结束当前预处理条件块。
- **L1694 EN**: Blank line separating nearby declarations or logic blocks.
  **L1694 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1695 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L1695 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L1696 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_v8x16_load_splat(const void *__mem) {`.
  **L1696 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_v8x16_load_splat(const void *__mem) {`。
- **L1697 EN**: Returns from the current function with `wasm_v128_load8_splat(__mem)`.
  **L1697 CN**: 以 `wasm_v128_load8_splat(__mem)` 从当前函数返回。
- **L1698 EN**: Closes the current lexical scope or compound statement.
  **L1698 CN**: 结束当前词法作用域或复合语句块。
- **L1699 EN**: Blank line separating nearby declarations or logic blocks.
  **L1699 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1700 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L1700 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L1701 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_v16x8_load_splat(const void *__mem) {`.
  **L1701 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_v16x8_load_splat(const void *__mem) {`。
- **L1702 EN**: Returns from the current function with `wasm_v128_load16_splat(__mem)`.
  **L1702 CN**: 以 `wasm_v128_load16_splat(__mem)` 从当前函数返回。
- **L1703 EN**: Closes the current lexical scope or compound statement.
  **L1703 CN**: 结束当前词法作用域或复合语句块。
- **L1704 EN**: Blank line separating nearby declarations or logic blocks.
  **L1704 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1705-1728

````c
static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_v128_load32_splat")
wasm_v32x4_load_splat(const void *__mem) {
  return wasm_v128_load32_splat(__mem);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_v128_load64_splat")
wasm_v64x2_load_splat(const void *__mem) {
  return wasm_v128_load64_splat(__mem);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_i16x8_load8x8")
wasm_i16x8_load_8x8(const void *__mem) {
  return wasm_i16x8_load8x8(__mem);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_u16x8_load8x8")
wasm_u16x8_load_8x8(const void *__mem) {
  return wasm_u16x8_load8x8(__mem);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_i32x4_load16x4")
wasm_i32x4_load_16x4(const void *__mem) {
  return wasm_i32x4_load16x4(__mem);
}
````
- **L1705 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L1705 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L1706 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_v32x4_load_splat(const void *__mem) {`.
  **L1706 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_v32x4_load_splat(const void *__mem) {`。
- **L1707 EN**: Returns from the current function with `wasm_v128_load32_splat(__mem)`.
  **L1707 CN**: 以 `wasm_v128_load32_splat(__mem)` 从当前函数返回。
- **L1708 EN**: Closes the current lexical scope or compound statement.
  **L1708 CN**: 结束当前词法作用域或复合语句块。
- **L1709 EN**: Blank line separating nearby declarations or logic blocks.
  **L1709 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1710 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L1710 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L1711 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_v64x2_load_splat(const void *__mem) {`.
  **L1711 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_v64x2_load_splat(const void *__mem) {`。
- **L1712 EN**: Returns from the current function with `wasm_v128_load64_splat(__mem)`.
  **L1712 CN**: 以 `wasm_v128_load64_splat(__mem)` 从当前函数返回。
- **L1713 EN**: Closes the current lexical scope or compound statement.
  **L1713 CN**: 结束当前词法作用域或复合语句块。
- **L1714 EN**: Blank line separating nearby declarations or logic blocks.
  **L1714 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1715 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L1715 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L1716 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_load_8x8(const void *__mem) {`.
  **L1716 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_load_8x8(const void *__mem) {`。
- **L1717 EN**: Returns from the current function with `wasm_i16x8_load8x8(__mem)`.
  **L1717 CN**: 以 `wasm_i16x8_load8x8(__mem)` 从当前函数返回。
- **L1718 EN**: Closes the current lexical scope or compound statement.
  **L1718 CN**: 结束当前词法作用域或复合语句块。
- **L1719 EN**: Blank line separating nearby declarations or logic blocks.
  **L1719 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1720 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L1720 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L1721 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u16x8_load_8x8(const void *__mem) {`.
  **L1721 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u16x8_load_8x8(const void *__mem) {`。
- **L1722 EN**: Returns from the current function with `wasm_u16x8_load8x8(__mem)`.
  **L1722 CN**: 以 `wasm_u16x8_load8x8(__mem)` 从当前函数返回。
- **L1723 EN**: Closes the current lexical scope or compound statement.
  **L1723 CN**: 结束当前词法作用域或复合语句块。
- **L1724 EN**: Blank line separating nearby declarations or logic blocks.
  **L1724 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1725 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L1725 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L1726 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_load_16x4(const void *__mem) {`.
  **L1726 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_load_16x4(const void *__mem) {`。
- **L1727 EN**: Returns from the current function with `wasm_i32x4_load16x4(__mem)`.
  **L1727 CN**: 以 `wasm_i32x4_load16x4(__mem)` 从当前函数返回。
- **L1728 EN**: Closes the current lexical scope or compound statement.
  **L1728 CN**: 结束当前词法作用域或复合语句块。

### Lines 1729-1752

````c

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_u32x4_load16x4")
wasm_u32x4_load_16x4(const void *__mem) {
  return wasm_u32x4_load16x4(__mem);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_i64x2_load32x2")
wasm_i64x2_load_32x2(const void *__mem) {
  return wasm_i64x2_load32x2(__mem);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_u64x2_load32x2")
wasm_u64x2_load_32x2(const void *__mem) {
  return wasm_u64x2_load32x2(__mem);
}

#define wasm_v8x16_shuffle(__a, __b, __c0, __c1, __c2, __c3, __c4, __c5, __c6, \
                           __c7, __c8, __c9, __c10, __c11, __c12, __c13,       \
                           __c14, __c15)                                       \
  __DEPRECATED_WASM_MACRO("wasm_v8x16_shuffle", "wasm_i8x16_shuffle")          \
  wasm_i8x16_shuffle(__a, __b, __c0, __c1, __c2, __c3, __c4, __c5, __c6, __c7, \
                     __c8, __c9, __c10, __c11, __c12, __c13, __c14, __c15)

#define wasm_v16x8_shuffle(__a, __b, __c0, __c1, __c2, __c3, __c4, __c5, __c6, \
````
- **L1729 EN**: Blank line separating nearby declarations or logic blocks.
  **L1729 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1730 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L1730 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L1731 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u32x4_load_16x4(const void *__mem) {`.
  **L1731 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u32x4_load_16x4(const void *__mem) {`。
- **L1732 EN**: Returns from the current function with `wasm_u32x4_load16x4(__mem)`.
  **L1732 CN**: 以 `wasm_u32x4_load16x4(__mem)` 从当前函数返回。
- **L1733 EN**: Closes the current lexical scope or compound statement.
  **L1733 CN**: 结束当前词法作用域或复合语句块。
- **L1734 EN**: Blank line separating nearby declarations or logic blocks.
  **L1734 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1735 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L1735 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L1736 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i64x2_load_32x2(const void *__mem) {`.
  **L1736 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i64x2_load_32x2(const void *__mem) {`。
- **L1737 EN**: Returns from the current function with `wasm_i64x2_load32x2(__mem)`.
  **L1737 CN**: 以 `wasm_i64x2_load32x2(__mem)` 从当前函数返回。
- **L1738 EN**: Closes the current lexical scope or compound statement.
  **L1738 CN**: 结束当前词法作用域或复合语句块。
- **L1739 EN**: Blank line separating nearby declarations or logic blocks.
  **L1739 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1740 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L1740 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L1741 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u64x2_load_32x2(const void *__mem) {`.
  **L1741 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u64x2_load_32x2(const void *__mem) {`。
- **L1742 EN**: Returns from the current function with `wasm_u64x2_load32x2(__mem)`.
  **L1742 CN**: 以 `wasm_u64x2_load32x2(__mem)` 从当前函数返回。
- **L1743 EN**: Closes the current lexical scope or compound statement.
  **L1743 CN**: 结束当前词法作用域或复合语句块。
- **L1744 EN**: Blank line separating nearby declarations or logic blocks.
  **L1744 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1745 EN**: Defines macro `wasm_v8x16_shuffle` for conditional compilation, shorthand, or API generation.
  **L1745 CN**: 定义宏 `wasm_v8x16_shuffle`，用于条件编译、简写或 API 生成。
- **L1746 EN**: Continues the surrounding expression or declaration: `__c7, __c8, __c9, __c10, __c11, __c12, __c13,       \`.
  **L1746 CN**: 继续构造周围的表达式或声明：`__c7, __c8, __c9, __c10, __c11, __c12, __c13,       \`。
- **L1747 EN**: Continues the surrounding expression or declaration: `__c14, __c15)                                       \`.
  **L1747 CN**: 继续构造周围的表达式或声明：`__c14, __c15)                                       \`。
- **L1748 EN**: Continues logic associated with callable symbol `__DEPRECATED_WASM_MACRO`.
  **L1748 CN**: 继续与可调用符号 `__DEPRECATED_WASM_MACRO` 相关的逻辑。
- **L1749 EN**: Continues logic associated with callable symbol `wasm_i8x16_shuffle`.
  **L1749 CN**: 继续与可调用符号 `wasm_i8x16_shuffle` 相关的逻辑。
- **L1750 EN**: Continues the surrounding expression or declaration: `__c8, __c9, __c10, __c11, __c12, __c13, __c14, __c15)`.
  **L1750 CN**: 继续构造周围的表达式或声明：`__c8, __c9, __c10, __c11, __c12, __c13, __c14, __c15)`。
- **L1751 EN**: Blank line separating nearby declarations or logic blocks.
  **L1751 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1752 EN**: Defines macro `wasm_v16x8_shuffle` for conditional compilation, shorthand, or API generation.
  **L1752 CN**: 定义宏 `wasm_v16x8_shuffle`，用于条件编译、简写或 API 生成。

### Lines 1753-1776

````c
                           __c7)                                               \
  __DEPRECATED_WASM_MACRO("wasm_v16x8_shuffle", "wasm_i16x8_shuffle")          \
  wasm_i16x8_shuffle(__a, __b, __c0, __c1, __c2, __c3, __c4, __c5, __c6, __c7)

#define wasm_v32x4_shuffle(__a, __b, __c0, __c1, __c2, __c3)                   \
  __DEPRECATED_WASM_MACRO("wasm_v32x4_shuffle", "wasm_i32x4_shuffle")          \
  wasm_i32x4_shuffle(__a, __b, __c0, __c1, __c2, __c3)

#define wasm_v64x2_shuffle(__a, __b, __c0, __c1)                               \
  __DEPRECATED_WASM_MACRO("wasm_v64x2_shuffle", "wasm_i64x2_shuffle")          \
  wasm_i64x2_shuffle(__a, __b, __c0, __c1)

// Relaxed SIMD intrinsics

#define __RELAXED_FN_ATTRS                                                     \
  __attribute__((__always_inline__, __nodebug__, __target__("relaxed-simd"),   \
                 __min_vector_width__(128)))

static __inline__ v128_t __RELAXED_FN_ATTRS
wasm_f32x4_relaxed_madd(v128_t __a, v128_t __b, v128_t __c) {
  return (v128_t)__builtin_wasm_relaxed_madd_f32x4((__f32x4)__a, (__f32x4)__b,
                                                   (__f32x4)__c);
}

````
- **L1753 EN**: Continues the surrounding expression or declaration: `__c7)                                               \`.
  **L1753 CN**: 继续构造周围的表达式或声明：`__c7)                                               \`。
- **L1754 EN**: Continues logic associated with callable symbol `__DEPRECATED_WASM_MACRO`.
  **L1754 CN**: 继续与可调用符号 `__DEPRECATED_WASM_MACRO` 相关的逻辑。
- **L1755 EN**: Continues logic associated with callable symbol `wasm_i16x8_shuffle`.
  **L1755 CN**: 继续与可调用符号 `wasm_i16x8_shuffle` 相关的逻辑。
- **L1756 EN**: Blank line separating nearby declarations or logic blocks.
  **L1756 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1757 EN**: Defines macro `wasm_v32x4_shuffle(__a, __b, __c0, __c1, __c2, __c3)` for conditional compilation, shorthand, or API generation.
  **L1757 CN**: 定义宏 `wasm_v32x4_shuffle(__a, __b, __c0, __c1, __c2, __c3)`，用于条件编译、简写或 API 生成。
- **L1758 EN**: Continues logic associated with callable symbol `__DEPRECATED_WASM_MACRO`.
  **L1758 CN**: 继续与可调用符号 `__DEPRECATED_WASM_MACRO` 相关的逻辑。
- **L1759 EN**: Continues logic associated with callable symbol `wasm_i32x4_shuffle`.
  **L1759 CN**: 继续与可调用符号 `wasm_i32x4_shuffle` 相关的逻辑。
- **L1760 EN**: Blank line separating nearby declarations or logic blocks.
  **L1760 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1761 EN**: Defines macro `wasm_v64x2_shuffle(__a, __b, __c0, __c1)` for conditional compilation, shorthand, or API generation.
  **L1761 CN**: 定义宏 `wasm_v64x2_shuffle(__a, __b, __c0, __c1)`，用于条件编译、简写或 API 生成。
- **L1762 EN**: Continues logic associated with callable symbol `__DEPRECATED_WASM_MACRO`.
  **L1762 CN**: 继续与可调用符号 `__DEPRECATED_WASM_MACRO` 相关的逻辑。
- **L1763 EN**: Continues logic associated with callable symbol `wasm_i64x2_shuffle`.
  **L1763 CN**: 继续与可调用符号 `wasm_i64x2_shuffle` 相关的逻辑。
- **L1764 EN**: Blank line separating nearby declarations or logic blocks.
  **L1764 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1765 EN**: Comment explains nearby logic, constraints, or intent: `Relaxed SIMD intrinsics`.
  **L1765 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Relaxed SIMD intrinsics`。
- **L1766 EN**: Blank line separating nearby declarations or logic blocks.
  **L1766 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1767 EN**: Defines macro `__RELAXED_FN_ATTRS` for conditional compilation, shorthand, or API generation.
  **L1767 CN**: 定义宏 `__RELAXED_FN_ATTRS`，用于条件编译、简写或 API 生成。
- **L1768 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("relaxed-simd"),   \`.
  **L1768 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("relaxed-simd"),   \`。
- **L1769 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L1769 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L1770 EN**: Blank line separating nearby declarations or logic blocks.
  **L1770 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1771 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __RELAXED_FN_ATTRS`.
  **L1771 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __RELAXED_FN_ATTRS`。
- **L1772 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_f32x4_relaxed_madd(v128_t __a, v128_t __b, v128_t __c) {`.
  **L1772 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_f32x4_relaxed_madd(v128_t __a, v128_t __b, v128_t __c) {`。
- **L1773 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_madd_f32x4((__f32x4)__a, (__f32x4)__b,`.
  **L1773 CN**: 以 `(v128_t)__builtin_wasm_relaxed_madd_f32x4((__f32x4)__a, (__f32x4)__b,` 从当前函数返回。
- **L1774 EN**: Executes a call or declaration centered on `statement`.
  **L1774 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1775 EN**: Closes the current lexical scope or compound statement.
  **L1775 CN**: 结束当前词法作用域或复合语句块。
- **L1776 EN**: Blank line separating nearby declarations or logic blocks.
  **L1776 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1777-1800

````c
static __inline__ v128_t __RELAXED_FN_ATTRS
wasm_f32x4_relaxed_nmadd(v128_t __a, v128_t __b, v128_t __c) {
  return (v128_t)__builtin_wasm_relaxed_nmadd_f32x4((__f32x4)__a, (__f32x4)__b,
                                                    (__f32x4)__c);
}

static __inline__ v128_t __RELAXED_FN_ATTRS
wasm_f64x2_relaxed_madd(v128_t __a, v128_t __b, v128_t __c) {
  return (v128_t)__builtin_wasm_relaxed_madd_f64x2((__f64x2)__a, (__f64x2)__b,
                                                   (__f64x2)__c);
}

static __inline__ v128_t __RELAXED_FN_ATTRS
wasm_f64x2_relaxed_nmadd(v128_t __a, v128_t __b, v128_t __c) {
  return (v128_t)__builtin_wasm_relaxed_nmadd_f64x2((__f64x2)__a, (__f64x2)__b,
                                                    (__f64x2)__c);
}

static __inline__ v128_t __RELAXED_FN_ATTRS
wasm_i8x16_relaxed_laneselect(v128_t __a, v128_t __b, v128_t __m) {
  return (v128_t)__builtin_wasm_relaxed_laneselect_i8x16(
      (__i8x16)__a, (__i8x16)__b, (__i8x16)__m);
}

````
- **L1777 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __RELAXED_FN_ATTRS`.
  **L1777 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __RELAXED_FN_ATTRS`。
- **L1778 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_f32x4_relaxed_nmadd(v128_t __a, v128_t __b, v128_t __c) {`.
  **L1778 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_f32x4_relaxed_nmadd(v128_t __a, v128_t __b, v128_t __c) {`。
- **L1779 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_nmadd_f32x4((__f32x4)__a, (__f32x4)__b,`.
  **L1779 CN**: 以 `(v128_t)__builtin_wasm_relaxed_nmadd_f32x4((__f32x4)__a, (__f32x4)__b,` 从当前函数返回。
- **L1780 EN**: Executes a call or declaration centered on `statement`.
  **L1780 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1781 EN**: Closes the current lexical scope or compound statement.
  **L1781 CN**: 结束当前词法作用域或复合语句块。
- **L1782 EN**: Blank line separating nearby declarations or logic blocks.
  **L1782 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1783 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __RELAXED_FN_ATTRS`.
  **L1783 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __RELAXED_FN_ATTRS`。
- **L1784 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_f64x2_relaxed_madd(v128_t __a, v128_t __b, v128_t __c) {`.
  **L1784 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_f64x2_relaxed_madd(v128_t __a, v128_t __b, v128_t __c) {`。
- **L1785 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_madd_f64x2((__f64x2)__a, (__f64x2)__b,`.
  **L1785 CN**: 以 `(v128_t)__builtin_wasm_relaxed_madd_f64x2((__f64x2)__a, (__f64x2)__b,` 从当前函数返回。
- **L1786 EN**: Executes a call or declaration centered on `statement`.
  **L1786 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1787 EN**: Closes the current lexical scope or compound statement.
  **L1787 CN**: 结束当前词法作用域或复合语句块。
- **L1788 EN**: Blank line separating nearby declarations or logic blocks.
  **L1788 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1789 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __RELAXED_FN_ATTRS`.
  **L1789 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __RELAXED_FN_ATTRS`。
- **L1790 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_f64x2_relaxed_nmadd(v128_t __a, v128_t __b, v128_t __c) {`.
  **L1790 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_f64x2_relaxed_nmadd(v128_t __a, v128_t __b, v128_t __c) {`。
- **L1791 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_nmadd_f64x2((__f64x2)__a, (__f64x2)__b,`.
  **L1791 CN**: 以 `(v128_t)__builtin_wasm_relaxed_nmadd_f64x2((__f64x2)__a, (__f64x2)__b,` 从当前函数返回。
- **L1792 EN**: Executes a call or declaration centered on `statement`.
  **L1792 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1793 EN**: Closes the current lexical scope or compound statement.
  **L1793 CN**: 结束当前词法作用域或复合语句块。
- **L1794 EN**: Blank line separating nearby declarations or logic blocks.
  **L1794 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1795 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __RELAXED_FN_ATTRS`.
  **L1795 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __RELAXED_FN_ATTRS`。
- **L1796 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i8x16_relaxed_laneselect(v128_t __a, v128_t __b, v128_t __m) {`.
  **L1796 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i8x16_relaxed_laneselect(v128_t __a, v128_t __b, v128_t __m) {`。
- **L1797 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_laneselect_i8x16(`.
  **L1797 CN**: 以 `(v128_t)__builtin_wasm_relaxed_laneselect_i8x16(` 从当前函数返回。
- **L1798 EN**: Executes a call or declaration centered on `statement`.
  **L1798 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1799 EN**: Closes the current lexical scope or compound statement.
  **L1799 CN**: 结束当前词法作用域或复合语句块。
- **L1800 EN**: Blank line separating nearby declarations or logic blocks.
  **L1800 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1801-1824

````c
static __inline__ v128_t __RELAXED_FN_ATTRS
wasm_i16x8_relaxed_laneselect(v128_t __a, v128_t __b, v128_t __m) {
  return (v128_t)__builtin_wasm_relaxed_laneselect_i16x8(
      (__i16x8)__a, (__i16x8)__b, (__i16x8)__m);
}

static __inline__ v128_t __RELAXED_FN_ATTRS
wasm_i32x4_relaxed_laneselect(v128_t __a, v128_t __b, v128_t __m) {
  return (v128_t)__builtin_wasm_relaxed_laneselect_i32x4(
      (__i32x4)__a, (__i32x4)__b, (__i32x4)__m);
}

static __inline__ v128_t __RELAXED_FN_ATTRS
wasm_i64x2_relaxed_laneselect(v128_t __a, v128_t __b, v128_t __m) {
  return (v128_t)__builtin_wasm_relaxed_laneselect_i64x2(
      (__i64x2)__a, (__i64x2)__b, (__i64x2)__m);
}

static __inline__ v128_t __RELAXED_FN_ATTRS
wasm_i8x16_relaxed_swizzle(v128_t __a, v128_t __s) {
  return (v128_t)__builtin_wasm_relaxed_swizzle_i8x16((__i8x16)__a,
                                                      (__i8x16)__s);
}

````
- **L1801 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __RELAXED_FN_ATTRS`.
  **L1801 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __RELAXED_FN_ATTRS`。
- **L1802 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_relaxed_laneselect(v128_t __a, v128_t __b, v128_t __m) {`.
  **L1802 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_relaxed_laneselect(v128_t __a, v128_t __b, v128_t __m) {`。
- **L1803 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_laneselect_i16x8(`.
  **L1803 CN**: 以 `(v128_t)__builtin_wasm_relaxed_laneselect_i16x8(` 从当前函数返回。
- **L1804 EN**: Executes a call or declaration centered on `statement`.
  **L1804 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1805 EN**: Closes the current lexical scope or compound statement.
  **L1805 CN**: 结束当前词法作用域或复合语句块。
- **L1806 EN**: Blank line separating nearby declarations or logic blocks.
  **L1806 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1807 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __RELAXED_FN_ATTRS`.
  **L1807 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __RELAXED_FN_ATTRS`。
- **L1808 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_relaxed_laneselect(v128_t __a, v128_t __b, v128_t __m) {`.
  **L1808 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_relaxed_laneselect(v128_t __a, v128_t __b, v128_t __m) {`。
- **L1809 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_laneselect_i32x4(`.
  **L1809 CN**: 以 `(v128_t)__builtin_wasm_relaxed_laneselect_i32x4(` 从当前函数返回。
- **L1810 EN**: Executes a call or declaration centered on `statement`.
  **L1810 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1811 EN**: Closes the current lexical scope or compound statement.
  **L1811 CN**: 结束当前词法作用域或复合语句块。
- **L1812 EN**: Blank line separating nearby declarations or logic blocks.
  **L1812 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1813 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __RELAXED_FN_ATTRS`.
  **L1813 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __RELAXED_FN_ATTRS`。
- **L1814 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i64x2_relaxed_laneselect(v128_t __a, v128_t __b, v128_t __m) {`.
  **L1814 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i64x2_relaxed_laneselect(v128_t __a, v128_t __b, v128_t __m) {`。
- **L1815 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_laneselect_i64x2(`.
  **L1815 CN**: 以 `(v128_t)__builtin_wasm_relaxed_laneselect_i64x2(` 从当前函数返回。
- **L1816 EN**: Executes a call or declaration centered on `statement`.
  **L1816 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1817 EN**: Closes the current lexical scope or compound statement.
  **L1817 CN**: 结束当前词法作用域或复合语句块。
- **L1818 EN**: Blank line separating nearby declarations or logic blocks.
  **L1818 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1819 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __RELAXED_FN_ATTRS`.
  **L1819 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __RELAXED_FN_ATTRS`。
- **L1820 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i8x16_relaxed_swizzle(v128_t __a, v128_t __s) {`.
  **L1820 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i8x16_relaxed_swizzle(v128_t __a, v128_t __s) {`。
- **L1821 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_swizzle_i8x16((__i8x16)__a,`.
  **L1821 CN**: 以 `(v128_t)__builtin_wasm_relaxed_swizzle_i8x16((__i8x16)__a,` 从当前函数返回。
- **L1822 EN**: Executes a call or declaration centered on `statement`.
  **L1822 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1823 EN**: Closes the current lexical scope or compound statement.
  **L1823 CN**: 结束当前词法作用域或复合语句块。
- **L1824 EN**: Blank line separating nearby declarations or logic blocks.
  **L1824 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1825-1848

````c
static __inline__ v128_t __RELAXED_FN_ATTRS wasm_f32x4_relaxed_min(v128_t __a,
                                                                   v128_t __b) {
  return (v128_t)__builtin_wasm_relaxed_min_f32x4((__f32x4)__a, (__f32x4)__b);
}

static __inline__ v128_t __RELAXED_FN_ATTRS wasm_f32x4_relaxed_max(v128_t __a,
                                                                   v128_t __b) {
  return (v128_t)__builtin_wasm_relaxed_max_f32x4((__f32x4)__a, (__f32x4)__b);
}

static __inline__ v128_t __RELAXED_FN_ATTRS wasm_f64x2_relaxed_min(v128_t __a,
                                                                   v128_t __b) {
  return (v128_t)__builtin_wasm_relaxed_min_f64x2((__f64x2)__a, (__f64x2)__b);
}

static __inline__ v128_t __RELAXED_FN_ATTRS wasm_f64x2_relaxed_max(v128_t __a,
                                                                   v128_t __b) {
  return (v128_t)__builtin_wasm_relaxed_max_f64x2((__f64x2)__a, (__f64x2)__b);
}

static __inline__ v128_t __RELAXED_FN_ATTRS
wasm_i32x4_relaxed_trunc_f32x4(v128_t __a) {
  return (v128_t)__builtin_wasm_relaxed_trunc_s_i32x4_f32x4((__f32x4)__a);
}
````
- **L1825 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __RELAXED_FN_ATTRS wasm_f32x4_relaxed_min(v128_t __a,`.
  **L1825 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __RELAXED_FN_ATTRS wasm_f32x4_relaxed_min(v128_t __a,`。
- **L1826 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1826 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1827 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_min_f32x4((__f32x4)__a, (__f32x4)__b)`.
  **L1827 CN**: 以 `(v128_t)__builtin_wasm_relaxed_min_f32x4((__f32x4)__a, (__f32x4)__b)` 从当前函数返回。
- **L1828 EN**: Closes the current lexical scope or compound statement.
  **L1828 CN**: 结束当前词法作用域或复合语句块。
- **L1829 EN**: Blank line separating nearby declarations or logic blocks.
  **L1829 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1830 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __RELAXED_FN_ATTRS wasm_f32x4_relaxed_max(v128_t __a,`.
  **L1830 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __RELAXED_FN_ATTRS wasm_f32x4_relaxed_max(v128_t __a,`。
- **L1831 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1831 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1832 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_max_f32x4((__f32x4)__a, (__f32x4)__b)`.
  **L1832 CN**: 以 `(v128_t)__builtin_wasm_relaxed_max_f32x4((__f32x4)__a, (__f32x4)__b)` 从当前函数返回。
- **L1833 EN**: Closes the current lexical scope or compound statement.
  **L1833 CN**: 结束当前词法作用域或复合语句块。
- **L1834 EN**: Blank line separating nearby declarations or logic blocks.
  **L1834 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1835 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __RELAXED_FN_ATTRS wasm_f64x2_relaxed_min(v128_t __a,`.
  **L1835 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __RELAXED_FN_ATTRS wasm_f64x2_relaxed_min(v128_t __a,`。
- **L1836 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1836 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1837 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_min_f64x2((__f64x2)__a, (__f64x2)__b)`.
  **L1837 CN**: 以 `(v128_t)__builtin_wasm_relaxed_min_f64x2((__f64x2)__a, (__f64x2)__b)` 从当前函数返回。
- **L1838 EN**: Closes the current lexical scope or compound statement.
  **L1838 CN**: 结束当前词法作用域或复合语句块。
- **L1839 EN**: Blank line separating nearby declarations or logic blocks.
  **L1839 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1840 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __RELAXED_FN_ATTRS wasm_f64x2_relaxed_max(v128_t __a,`.
  **L1840 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __RELAXED_FN_ATTRS wasm_f64x2_relaxed_max(v128_t __a,`。
- **L1841 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1841 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1842 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_max_f64x2((__f64x2)__a, (__f64x2)__b)`.
  **L1842 CN**: 以 `(v128_t)__builtin_wasm_relaxed_max_f64x2((__f64x2)__a, (__f64x2)__b)` 从当前函数返回。
- **L1843 EN**: Closes the current lexical scope or compound statement.
  **L1843 CN**: 结束当前词法作用域或复合语句块。
- **L1844 EN**: Blank line separating nearby declarations or logic blocks.
  **L1844 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1845 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __RELAXED_FN_ATTRS`.
  **L1845 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __RELAXED_FN_ATTRS`。
- **L1846 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_relaxed_trunc_f32x4(v128_t __a) {`.
  **L1846 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_relaxed_trunc_f32x4(v128_t __a) {`。
- **L1847 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_trunc_s_i32x4_f32x4((__f32x4)__a)`.
  **L1847 CN**: 以 `(v128_t)__builtin_wasm_relaxed_trunc_s_i32x4_f32x4((__f32x4)__a)` 从当前函数返回。
- **L1848 EN**: Closes the current lexical scope or compound statement.
  **L1848 CN**: 结束当前词法作用域或复合语句块。

### Lines 1849-1872

````c

static __inline__ v128_t __RELAXED_FN_ATTRS
wasm_u32x4_relaxed_trunc_f32x4(v128_t __a) {
  return (v128_t)__builtin_wasm_relaxed_trunc_u_i32x4_f32x4((__f32x4)__a);
}

static __inline__ v128_t __RELAXED_FN_ATTRS
wasm_i32x4_relaxed_trunc_f64x2_zero(v128_t __a) {
  return (v128_t)__builtin_wasm_relaxed_trunc_s_zero_i32x4_f64x2((__f64x2)__a);
}

static __inline__ v128_t __RELAXED_FN_ATTRS
wasm_u32x4_relaxed_trunc_f64x2_zero(v128_t __a) {
  return (v128_t)__builtin_wasm_relaxed_trunc_u_zero_i32x4_f64x2((__f64x2)__a);
}

static __inline__ v128_t __RELAXED_FN_ATTRS
wasm_i16x8_relaxed_q15mulr(v128_t __a, v128_t __b) {
  return (v128_t)__builtin_wasm_relaxed_q15mulr_s_i16x8((__i16x8)__a,
                                                        (__i16x8)__b);
}

static __inline__ v128_t __RELAXED_FN_ATTRS
wasm_i16x8_relaxed_dot_i8x16_i7x16(v128_t __a, v128_t __b) {
````
- **L1849 EN**: Blank line separating nearby declarations or logic blocks.
  **L1849 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1850 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __RELAXED_FN_ATTRS`.
  **L1850 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __RELAXED_FN_ATTRS`。
- **L1851 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u32x4_relaxed_trunc_f32x4(v128_t __a) {`.
  **L1851 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u32x4_relaxed_trunc_f32x4(v128_t __a) {`。
- **L1852 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_trunc_u_i32x4_f32x4((__f32x4)__a)`.
  **L1852 CN**: 以 `(v128_t)__builtin_wasm_relaxed_trunc_u_i32x4_f32x4((__f32x4)__a)` 从当前函数返回。
- **L1853 EN**: Closes the current lexical scope or compound statement.
  **L1853 CN**: 结束当前词法作用域或复合语句块。
- **L1854 EN**: Blank line separating nearby declarations or logic blocks.
  **L1854 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1855 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __RELAXED_FN_ATTRS`.
  **L1855 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __RELAXED_FN_ATTRS`。
- **L1856 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_relaxed_trunc_f64x2_zero(v128_t __a) {`.
  **L1856 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_relaxed_trunc_f64x2_zero(v128_t __a) {`。
- **L1857 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_trunc_s_zero_i32x4_f64x2((__f64x2)__a)`.
  **L1857 CN**: 以 `(v128_t)__builtin_wasm_relaxed_trunc_s_zero_i32x4_f64x2((__f64x2)__a)` 从当前函数返回。
- **L1858 EN**: Closes the current lexical scope or compound statement.
  **L1858 CN**: 结束当前词法作用域或复合语句块。
- **L1859 EN**: Blank line separating nearby declarations or logic blocks.
  **L1859 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1860 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __RELAXED_FN_ATTRS`.
  **L1860 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __RELAXED_FN_ATTRS`。
- **L1861 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u32x4_relaxed_trunc_f64x2_zero(v128_t __a) {`.
  **L1861 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u32x4_relaxed_trunc_f64x2_zero(v128_t __a) {`。
- **L1862 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_trunc_u_zero_i32x4_f64x2((__f64x2)__a)`.
  **L1862 CN**: 以 `(v128_t)__builtin_wasm_relaxed_trunc_u_zero_i32x4_f64x2((__f64x2)__a)` 从当前函数返回。
- **L1863 EN**: Closes the current lexical scope or compound statement.
  **L1863 CN**: 结束当前词法作用域或复合语句块。
- **L1864 EN**: Blank line separating nearby declarations or logic blocks.
  **L1864 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1865 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __RELAXED_FN_ATTRS`.
  **L1865 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __RELAXED_FN_ATTRS`。
- **L1866 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_relaxed_q15mulr(v128_t __a, v128_t __b) {`.
  **L1866 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_relaxed_q15mulr(v128_t __a, v128_t __b) {`。
- **L1867 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_q15mulr_s_i16x8((__i16x8)__a,`.
  **L1867 CN**: 以 `(v128_t)__builtin_wasm_relaxed_q15mulr_s_i16x8((__i16x8)__a,` 从当前函数返回。
- **L1868 EN**: Executes a call or declaration centered on `statement`.
  **L1868 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1869 EN**: Closes the current lexical scope or compound statement.
  **L1869 CN**: 结束当前词法作用域或复合语句块。
- **L1870 EN**: Blank line separating nearby declarations or logic blocks.
  **L1870 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1871 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __RELAXED_FN_ATTRS`.
  **L1871 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __RELAXED_FN_ATTRS`。
- **L1872 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_relaxed_dot_i8x16_i7x16(v128_t __a, v128_t __b) {`.
  **L1872 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_relaxed_dot_i8x16_i7x16(v128_t __a, v128_t __b) {`。

### Lines 1873-1896

````c
  return (v128_t)__builtin_wasm_relaxed_dot_i8x16_i7x16_s_i16x8((__i8x16)__a,
                                                                (__i8x16)__b);
}

static __inline__ v128_t __RELAXED_FN_ATTRS
wasm_i32x4_relaxed_dot_i8x16_i7x16_add(v128_t __a, v128_t __b, v128_t __c) {
  return (v128_t)__builtin_wasm_relaxed_dot_i8x16_i7x16_add_s_i32x4(
      (__i8x16)__a, (__i8x16)__b, (__i32x4)__c);
}

// FP16 intrinsics
#define __FP16_FN_ATTRS                                                        \
  __attribute__((__always_inline__, __nodebug__, __target__("fp16"),           \
                 __min_vector_width__(128)))

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_splat(float __a) {
  return (v128_t)__builtin_wasm_splat_f16x8(__a);
}

#ifdef __wasm_fp16__
// TODO Replace the following macros with regular C functions and use normal
// target-independent vector code like the other replace/extract instructions.

#define wasm_f16x8_extract_lane(__a, __i)                                      \
````
- **L1873 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_dot_i8x16_i7x16_s_i16x8((__i8x16)__a,`.
  **L1873 CN**: 以 `(v128_t)__builtin_wasm_relaxed_dot_i8x16_i7x16_s_i16x8((__i8x16)__a,` 从当前函数返回。
- **L1874 EN**: Executes a call or declaration centered on `statement`.
  **L1874 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1875 EN**: Closes the current lexical scope or compound statement.
  **L1875 CN**: 结束当前词法作用域或复合语句块。
- **L1876 EN**: Blank line separating nearby declarations or logic blocks.
  **L1876 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1877 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __RELAXED_FN_ATTRS`.
  **L1877 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __RELAXED_FN_ATTRS`。
- **L1878 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_relaxed_dot_i8x16_i7x16_add(v128_t __a, v128_t __b, v128_t __c) {`.
  **L1878 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_relaxed_dot_i8x16_i7x16_add(v128_t __a, v128_t __b, v128_t __c) {`。
- **L1879 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_dot_i8x16_i7x16_add_s_i32x4(`.
  **L1879 CN**: 以 `(v128_t)__builtin_wasm_relaxed_dot_i8x16_i7x16_add_s_i32x4(` 从当前函数返回。
- **L1880 EN**: Executes a call or declaration centered on `statement`.
  **L1880 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1881 EN**: Closes the current lexical scope or compound statement.
  **L1881 CN**: 结束当前词法作用域或复合语句块。
- **L1882 EN**: Blank line separating nearby declarations or logic blocks.
  **L1882 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1883 EN**: Comment explains nearby logic, constraints, or intent: `FP16 intrinsics`.
  **L1883 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FP16 intrinsics`。
- **L1884 EN**: Defines macro `__FP16_FN_ATTRS` for conditional compilation, shorthand, or API generation.
  **L1884 CN**: 定义宏 `__FP16_FN_ATTRS`，用于条件编译、简写或 API 生成。
- **L1885 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__attribute__((__always_inline__, __nodebug__, __target__("fp16"),           \`.
  **L1885 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__attribute__((__always_inline__, __nodebug__, __target__("fp16"),           \`。
- **L1886 EN**: Continues logic associated with callable symbol `__min_vector_width__`.
  **L1886 CN**: 继续与可调用符号 `__min_vector_width__` 相关的逻辑。
- **L1887 EN**: Blank line separating nearby declarations or logic blocks.
  **L1887 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1888 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_splat(float __a) {`.
  **L1888 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_splat(float __a) {`。
- **L1889 EN**: Returns from the current function with `(v128_t)__builtin_wasm_splat_f16x8(__a)`.
  **L1889 CN**: 以 `(v128_t)__builtin_wasm_splat_f16x8(__a)` 从当前函数返回。
- **L1890 EN**: Closes the current lexical scope or compound statement.
  **L1890 CN**: 结束当前词法作用域或复合语句块。
- **L1891 EN**: Blank line separating nearby declarations or logic blocks.
  **L1891 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1892 EN**: Starts a preprocessor conditional block: `#ifdef __wasm_fp16__`.
  **L1892 CN**: 开始一个预处理条件块：`#ifdef __wasm_fp16__`。
- **L1893 EN**: Comment records a pending task or caution: `TODO Replace the following macros with regular C functions and use normal`.
  **L1893 CN**: 注释记录待办事项或注意点：`TODO Replace the following macros with regular C functions and use normal`。
- **L1894 EN**: Comment explains nearby logic, constraints, or intent: `target-independent vector code like the other replace/extract instructions.`.
  **L1894 CN**: 注释解释附近代码的逻辑、约束或设计意图：`target-independent vector code like the other replace/extract instructions.`。
- **L1895 EN**: Blank line separating nearby declarations or logic blocks.
  **L1895 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1896 EN**: Defines macro `wasm_f16x8_extract_lane(__a, __i)` for conditional compilation, shorthand, or API generation.
  **L1896 CN**: 定义宏 `wasm_f16x8_extract_lane(__a, __i)`，用于条件编译、简写或 API 生成。

### Lines 1897-1920

````c
  (__builtin_wasm_extract_lane_f16x8((__f16x8)(__a), __i))

#define wasm_f16x8_replace_lane(__a, __i, __b)                                 \
  ((v128_t)__builtin_wasm_replace_lane_f16x8((__f16x8)(__a), __i, __b))

#endif

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_abs(v128_t __a) {
  return (v128_t)__builtin_wasm_abs_f16x8((__f16x8)__a);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_neg(v128_t __a) {
  return (v128_t)(-(__f16x8)__a);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_sqrt(v128_t __a) {
  return (v128_t)__builtin_wasm_sqrt_f16x8((__f16x8)__a);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_ceil(v128_t __a) {
  return (v128_t)__builtin_wasm_ceil_f16x8((__f16x8)__a);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_floor(v128_t __a) {
````
- **L1897 EN**: Continues logic associated with callable symbol `__builtin_wasm_extract_lane_f16x8`.
  **L1897 CN**: 继续与可调用符号 `__builtin_wasm_extract_lane_f16x8` 相关的逻辑。
- **L1898 EN**: Blank line separating nearby declarations or logic blocks.
  **L1898 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1899 EN**: Defines macro `wasm_f16x8_replace_lane(__a, __i, __b)` for conditional compilation, shorthand, or API generation.
  **L1899 CN**: 定义宏 `wasm_f16x8_replace_lane(__a, __i, __b)`，用于条件编译、简写或 API 生成。
- **L1900 EN**: Continues logic associated with callable symbol `__builtin_wasm_replace_lane_f16x8`.
  **L1900 CN**: 继续与可调用符号 `__builtin_wasm_replace_lane_f16x8` 相关的逻辑。
- **L1901 EN**: Blank line separating nearby declarations or logic blocks.
  **L1901 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1902 EN**: Closes the current preprocessor conditional block.
  **L1902 CN**: 结束当前预处理条件块。
- **L1903 EN**: Blank line separating nearby declarations or logic blocks.
  **L1903 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1904 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_abs(v128_t __a) {`.
  **L1904 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_abs(v128_t __a) {`。
- **L1905 EN**: Returns from the current function with `(v128_t)__builtin_wasm_abs_f16x8((__f16x8)__a)`.
  **L1905 CN**: 以 `(v128_t)__builtin_wasm_abs_f16x8((__f16x8)__a)` 从当前函数返回。
- **L1906 EN**: Closes the current lexical scope or compound statement.
  **L1906 CN**: 结束当前词法作用域或复合语句块。
- **L1907 EN**: Blank line separating nearby declarations or logic blocks.
  **L1907 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1908 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_neg(v128_t __a) {`.
  **L1908 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_neg(v128_t __a) {`。
- **L1909 EN**: Returns from the current function with `(v128_t)(-(__f16x8)__a)`.
  **L1909 CN**: 以 `(v128_t)(-(__f16x8)__a)` 从当前函数返回。
- **L1910 EN**: Closes the current lexical scope or compound statement.
  **L1910 CN**: 结束当前词法作用域或复合语句块。
- **L1911 EN**: Blank line separating nearby declarations or logic blocks.
  **L1911 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1912 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_sqrt(v128_t __a) {`.
  **L1912 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_sqrt(v128_t __a) {`。
- **L1913 EN**: Returns from the current function with `(v128_t)__builtin_wasm_sqrt_f16x8((__f16x8)__a)`.
  **L1913 CN**: 以 `(v128_t)__builtin_wasm_sqrt_f16x8((__f16x8)__a)` 从当前函数返回。
- **L1914 EN**: Closes the current lexical scope or compound statement.
  **L1914 CN**: 结束当前词法作用域或复合语句块。
- **L1915 EN**: Blank line separating nearby declarations or logic blocks.
  **L1915 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1916 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_ceil(v128_t __a) {`.
  **L1916 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_ceil(v128_t __a) {`。
- **L1917 EN**: Returns from the current function with `(v128_t)__builtin_wasm_ceil_f16x8((__f16x8)__a)`.
  **L1917 CN**: 以 `(v128_t)__builtin_wasm_ceil_f16x8((__f16x8)__a)` 从当前函数返回。
- **L1918 EN**: Closes the current lexical scope or compound statement.
  **L1918 CN**: 结束当前词法作用域或复合语句块。
- **L1919 EN**: Blank line separating nearby declarations or logic blocks.
  **L1919 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1920 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_floor(v128_t __a) {`.
  **L1920 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_floor(v128_t __a) {`。

### Lines 1921-1944

````c
  return (v128_t)__builtin_wasm_floor_f16x8((__f16x8)__a);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_trunc(v128_t __a) {
  return (v128_t)__builtin_wasm_trunc_f16x8((__f16x8)__a);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_nearest(v128_t __a) {
  return (v128_t)__builtin_wasm_nearest_f16x8((__f16x8)__a);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_eq(v128_t __a, v128_t __b) {
  return (v128_t)((__f16x8)__a == (__f16x8)__b);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_ne(v128_t __a, v128_t __b) {
  return (v128_t)((__f16x8)__a != (__f16x8)__b);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_lt(v128_t __a, v128_t __b) {
  return (v128_t)((__f16x8)__a < (__f16x8)__b);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_gt(v128_t __a, v128_t __b) {
````
- **L1921 EN**: Returns from the current function with `(v128_t)__builtin_wasm_floor_f16x8((__f16x8)__a)`.
  **L1921 CN**: 以 `(v128_t)__builtin_wasm_floor_f16x8((__f16x8)__a)` 从当前函数返回。
- **L1922 EN**: Closes the current lexical scope or compound statement.
  **L1922 CN**: 结束当前词法作用域或复合语句块。
- **L1923 EN**: Blank line separating nearby declarations or logic blocks.
  **L1923 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1924 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_trunc(v128_t __a) {`.
  **L1924 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_trunc(v128_t __a) {`。
- **L1925 EN**: Returns from the current function with `(v128_t)__builtin_wasm_trunc_f16x8((__f16x8)__a)`.
  **L1925 CN**: 以 `(v128_t)__builtin_wasm_trunc_f16x8((__f16x8)__a)` 从当前函数返回。
- **L1926 EN**: Closes the current lexical scope or compound statement.
  **L1926 CN**: 结束当前词法作用域或复合语句块。
- **L1927 EN**: Blank line separating nearby declarations or logic blocks.
  **L1927 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1928 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_nearest(v128_t __a) {`.
  **L1928 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_nearest(v128_t __a) {`。
- **L1929 EN**: Returns from the current function with `(v128_t)__builtin_wasm_nearest_f16x8((__f16x8)__a)`.
  **L1929 CN**: 以 `(v128_t)__builtin_wasm_nearest_f16x8((__f16x8)__a)` 从当前函数返回。
- **L1930 EN**: Closes the current lexical scope or compound statement.
  **L1930 CN**: 结束当前词法作用域或复合语句块。
- **L1931 EN**: Blank line separating nearby declarations or logic blocks.
  **L1931 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1932 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_eq(v128_t __a, v128_t __b) {`.
  **L1932 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_eq(v128_t __a, v128_t __b) {`。
- **L1933 EN**: Returns from the current function with `(v128_t)((__f16x8)__a == (__f16x8)__b)`.
  **L1933 CN**: 以 `(v128_t)((__f16x8)__a == (__f16x8)__b)` 从当前函数返回。
- **L1934 EN**: Closes the current lexical scope or compound statement.
  **L1934 CN**: 结束当前词法作用域或复合语句块。
- **L1935 EN**: Blank line separating nearby declarations or logic blocks.
  **L1935 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1936 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_ne(v128_t __a, v128_t __b) {`.
  **L1936 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_ne(v128_t __a, v128_t __b) {`。
- **L1937 EN**: Returns from the current function with `(v128_t)((__f16x8)__a != (__f16x8)__b)`.
  **L1937 CN**: 以 `(v128_t)((__f16x8)__a != (__f16x8)__b)` 从当前函数返回。
- **L1938 EN**: Closes the current lexical scope or compound statement.
  **L1938 CN**: 结束当前词法作用域或复合语句块。
- **L1939 EN**: Blank line separating nearby declarations or logic blocks.
  **L1939 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1940 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_lt(v128_t __a, v128_t __b) {`.
  **L1940 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_lt(v128_t __a, v128_t __b) {`。
- **L1941 EN**: Returns from the current function with `(v128_t)((__f16x8)__a < (__f16x8)__b)`.
  **L1941 CN**: 以 `(v128_t)((__f16x8)__a < (__f16x8)__b)` 从当前函数返回。
- **L1942 EN**: Closes the current lexical scope or compound statement.
  **L1942 CN**: 结束当前词法作用域或复合语句块。
- **L1943 EN**: Blank line separating nearby declarations or logic blocks.
  **L1943 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1944 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_gt(v128_t __a, v128_t __b) {`.
  **L1944 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_gt(v128_t __a, v128_t __b) {`。

### Lines 1945-1968

````c
  return (v128_t)((__f16x8)__a > (__f16x8)__b);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_le(v128_t __a, v128_t __b) {
  return (v128_t)((__f16x8)__a <= (__f16x8)__b);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_ge(v128_t __a, v128_t __b) {
  return (v128_t)((__f16x8)__a >= (__f16x8)__b);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_add(v128_t __a,
                                                        v128_t __b) {
  return (v128_t)((__f16x8)__a + (__f16x8)__b);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_sub(v128_t __a,
                                                        v128_t __b) {
  return (v128_t)((__f16x8)__a - (__f16x8)__b);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_mul(v128_t __a,
                                                        v128_t __b) {
  return (v128_t)((__f16x8)__a * (__f16x8)__b);
````
- **L1945 EN**: Returns from the current function with `(v128_t)((__f16x8)__a > (__f16x8)__b)`.
  **L1945 CN**: 以 `(v128_t)((__f16x8)__a > (__f16x8)__b)` 从当前函数返回。
- **L1946 EN**: Closes the current lexical scope or compound statement.
  **L1946 CN**: 结束当前词法作用域或复合语句块。
- **L1947 EN**: Blank line separating nearby declarations or logic blocks.
  **L1947 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1948 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_le(v128_t __a, v128_t __b) {`.
  **L1948 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_le(v128_t __a, v128_t __b) {`。
- **L1949 EN**: Returns from the current function with `(v128_t)((__f16x8)__a <= (__f16x8)__b)`.
  **L1949 CN**: 以 `(v128_t)((__f16x8)__a <= (__f16x8)__b)` 从当前函数返回。
- **L1950 EN**: Closes the current lexical scope or compound statement.
  **L1950 CN**: 结束当前词法作用域或复合语句块。
- **L1951 EN**: Blank line separating nearby declarations or logic blocks.
  **L1951 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1952 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_ge(v128_t __a, v128_t __b) {`.
  **L1952 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_ge(v128_t __a, v128_t __b) {`。
- **L1953 EN**: Returns from the current function with `(v128_t)((__f16x8)__a >= (__f16x8)__b)`.
  **L1953 CN**: 以 `(v128_t)((__f16x8)__a >= (__f16x8)__b)` 从当前函数返回。
- **L1954 EN**: Closes the current lexical scope or compound statement.
  **L1954 CN**: 结束当前词法作用域或复合语句块。
- **L1955 EN**: Blank line separating nearby declarations or logic blocks.
  **L1955 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1956 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_add(v128_t __a,`.
  **L1956 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_add(v128_t __a,`。
- **L1957 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1957 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1958 EN**: Returns from the current function with `(v128_t)((__f16x8)__a + (__f16x8)__b)`.
  **L1958 CN**: 以 `(v128_t)((__f16x8)__a + (__f16x8)__b)` 从当前函数返回。
- **L1959 EN**: Closes the current lexical scope or compound statement.
  **L1959 CN**: 结束当前词法作用域或复合语句块。
- **L1960 EN**: Blank line separating nearby declarations or logic blocks.
  **L1960 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1961 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_sub(v128_t __a,`.
  **L1961 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_sub(v128_t __a,`。
- **L1962 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1962 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1963 EN**: Returns from the current function with `(v128_t)((__f16x8)__a - (__f16x8)__b)`.
  **L1963 CN**: 以 `(v128_t)((__f16x8)__a - (__f16x8)__b)` 从当前函数返回。
- **L1964 EN**: Closes the current lexical scope or compound statement.
  **L1964 CN**: 结束当前词法作用域或复合语句块。
- **L1965 EN**: Blank line separating nearby declarations or logic blocks.
  **L1965 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1966 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_mul(v128_t __a,`.
  **L1966 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_mul(v128_t __a,`。
- **L1967 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1967 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1968 EN**: Returns from the current function with `(v128_t)((__f16x8)__a * (__f16x8)__b)`.
  **L1968 CN**: 以 `(v128_t)((__f16x8)__a * (__f16x8)__b)` 从当前函数返回。

### Lines 1969-1992

````c
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_div(v128_t __a,
                                                        v128_t __b) {
  return (v128_t)((__f16x8)__a / (__f16x8)__b);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_min(v128_t __a,
                                                        v128_t __b) {
  return (v128_t)__builtin_wasm_min_f16x8((__f16x8)__a, (__f16x8)__b);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_max(v128_t __a,
                                                        v128_t __b) {
  return (v128_t)__builtin_wasm_max_f16x8((__f16x8)__a, (__f16x8)__b);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_pmin(v128_t __a,
                                                         v128_t __b) {
  return (v128_t)__builtin_wasm_pmin_f16x8((__f16x8)__a, (__f16x8)__b);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_pmax(v128_t __a,
                                                         v128_t __b) {
````
- **L1969 EN**: Closes the current lexical scope or compound statement.
  **L1969 CN**: 结束当前词法作用域或复合语句块。
- **L1970 EN**: Blank line separating nearby declarations or logic blocks.
  **L1970 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1971 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_div(v128_t __a,`.
  **L1971 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_div(v128_t __a,`。
- **L1972 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1972 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1973 EN**: Returns from the current function with `(v128_t)((__f16x8)__a / (__f16x8)__b)`.
  **L1973 CN**: 以 `(v128_t)((__f16x8)__a / (__f16x8)__b)` 从当前函数返回。
- **L1974 EN**: Closes the current lexical scope or compound statement.
  **L1974 CN**: 结束当前词法作用域或复合语句块。
- **L1975 EN**: Blank line separating nearby declarations or logic blocks.
  **L1975 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1976 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_min(v128_t __a,`.
  **L1976 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_min(v128_t __a,`。
- **L1977 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1977 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1978 EN**: Returns from the current function with `(v128_t)__builtin_wasm_min_f16x8((__f16x8)__a, (__f16x8)__b)`.
  **L1978 CN**: 以 `(v128_t)__builtin_wasm_min_f16x8((__f16x8)__a, (__f16x8)__b)` 从当前函数返回。
- **L1979 EN**: Closes the current lexical scope or compound statement.
  **L1979 CN**: 结束当前词法作用域或复合语句块。
- **L1980 EN**: Blank line separating nearby declarations or logic blocks.
  **L1980 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1981 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_max(v128_t __a,`.
  **L1981 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_max(v128_t __a,`。
- **L1982 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1982 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1983 EN**: Returns from the current function with `(v128_t)__builtin_wasm_max_f16x8((__f16x8)__a, (__f16x8)__b)`.
  **L1983 CN**: 以 `(v128_t)__builtin_wasm_max_f16x8((__f16x8)__a, (__f16x8)__b)` 从当前函数返回。
- **L1984 EN**: Closes the current lexical scope or compound statement.
  **L1984 CN**: 结束当前词法作用域或复合语句块。
- **L1985 EN**: Blank line separating nearby declarations or logic blocks.
  **L1985 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1986 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_pmin(v128_t __a,`.
  **L1986 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_pmin(v128_t __a,`。
- **L1987 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1987 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。
- **L1988 EN**: Returns from the current function with `(v128_t)__builtin_wasm_pmin_f16x8((__f16x8)__a, (__f16x8)__b)`.
  **L1988 CN**: 以 `(v128_t)__builtin_wasm_pmin_f16x8((__f16x8)__a, (__f16x8)__b)` 从当前函数返回。
- **L1989 EN**: Closes the current lexical scope or compound statement.
  **L1989 CN**: 结束当前词法作用域或复合语句块。
- **L1990 EN**: Blank line separating nearby declarations or logic blocks.
  **L1990 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1991 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_pmax(v128_t __a,`.
  **L1991 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_pmax(v128_t __a,`。
- **L1992 EN**: Continues the surrounding expression or declaration: `v128_t __b) {`.
  **L1992 CN**: 继续构造周围的表达式或声明：`v128_t __b) {`。

### Lines 1993-2016

````c
  return (v128_t)__builtin_wasm_pmax_f16x8((__f16x8)__a, (__f16x8)__b);
}

static __inline__ v128_t __FP16_FN_ATTRS
wasm_i16x8_trunc_sat_f16x8(v128_t __a) {
  return (v128_t)__builtin_wasm_trunc_saturate_s_i16x8_f16x8((__f16x8)__a);
}

static __inline__ v128_t __FP16_FN_ATTRS
wasm_u16x8_trunc_sat_f16x8(v128_t __a) {
  return (v128_t)__builtin_wasm_trunc_saturate_u_i16x8_f16x8((__f16x8)__a);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_convert_i16x8(v128_t __a) {
  return (v128_t) __builtin_convertvector((__i16x8)__a, __f16x8);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_convert_u16x8(v128_t __a) {
  return (v128_t) __builtin_convertvector((__u16x8)__a, __f16x8);
}

static __inline__ v128_t __FP16_FN_ATTRS
wasm_f32x4_promote_low_f16x8(v128_t __a) {
  return (v128_t) __builtin_convertvector(
````
- **L1993 EN**: Returns from the current function with `(v128_t)__builtin_wasm_pmax_f16x8((__f16x8)__a, (__f16x8)__b)`.
  **L1993 CN**: 以 `(v128_t)__builtin_wasm_pmax_f16x8((__f16x8)__a, (__f16x8)__b)` 从当前函数返回。
- **L1994 EN**: Closes the current lexical scope or compound statement.
  **L1994 CN**: 结束当前词法作用域或复合语句块。
- **L1995 EN**: Blank line separating nearby declarations or logic blocks.
  **L1995 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1996 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __FP16_FN_ATTRS`.
  **L1996 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __FP16_FN_ATTRS`。
- **L1997 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_trunc_sat_f16x8(v128_t __a) {`.
  **L1997 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_trunc_sat_f16x8(v128_t __a) {`。
- **L1998 EN**: Returns from the current function with `(v128_t)__builtin_wasm_trunc_saturate_s_i16x8_f16x8((__f16x8)__a)`.
  **L1998 CN**: 以 `(v128_t)__builtin_wasm_trunc_saturate_s_i16x8_f16x8((__f16x8)__a)` 从当前函数返回。
- **L1999 EN**: Closes the current lexical scope or compound statement.
  **L1999 CN**: 结束当前词法作用域或复合语句块。
- **L2000 EN**: Blank line separating nearby declarations or logic blocks.
  **L2000 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2001 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __FP16_FN_ATTRS`.
  **L2001 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __FP16_FN_ATTRS`。
- **L2002 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u16x8_trunc_sat_f16x8(v128_t __a) {`.
  **L2002 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u16x8_trunc_sat_f16x8(v128_t __a) {`。
- **L2003 EN**: Returns from the current function with `(v128_t)__builtin_wasm_trunc_saturate_u_i16x8_f16x8((__f16x8)__a)`.
  **L2003 CN**: 以 `(v128_t)__builtin_wasm_trunc_saturate_u_i16x8_f16x8((__f16x8)__a)` 从当前函数返回。
- **L2004 EN**: Closes the current lexical scope or compound statement.
  **L2004 CN**: 结束当前词法作用域或复合语句块。
- **L2005 EN**: Blank line separating nearby declarations or logic blocks.
  **L2005 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2006 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_convert_i16x8(v128_t __a) {`.
  **L2006 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_convert_i16x8(v128_t __a) {`。
- **L2007 EN**: Returns from the current function with `(v128_t) __builtin_convertvector((__i16x8)__a, __f16x8)`.
  **L2007 CN**: 以 `(v128_t) __builtin_convertvector((__i16x8)__a, __f16x8)` 从当前函数返回。
- **L2008 EN**: Closes the current lexical scope or compound statement.
  **L2008 CN**: 结束当前词法作用域或复合语句块。
- **L2009 EN**: Blank line separating nearby declarations or logic blocks.
  **L2009 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2010 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_convert_u16x8(v128_t __a) {`.
  **L2010 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_convert_u16x8(v128_t __a) {`。
- **L2011 EN**: Returns from the current function with `(v128_t) __builtin_convertvector((__u16x8)__a, __f16x8)`.
  **L2011 CN**: 以 `(v128_t) __builtin_convertvector((__u16x8)__a, __f16x8)` 从当前函数返回。
- **L2012 EN**: Closes the current lexical scope or compound statement.
  **L2012 CN**: 结束当前词法作用域或复合语句块。
- **L2013 EN**: Blank line separating nearby declarations or logic blocks.
  **L2013 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2014 EN**: Continues the surrounding expression or declaration: `static __inline__ v128_t __FP16_FN_ATTRS`.
  **L2014 CN**: 继续构造周围的表达式或声明：`static __inline__ v128_t __FP16_FN_ATTRS`。
- **L2015 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_f32x4_promote_low_f16x8(v128_t __a) {`.
  **L2015 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_f32x4_promote_low_f16x8(v128_t __a) {`。
- **L2016 EN**: Returns from the current function with `(v128_t) __builtin_convertvector(`.
  **L2016 CN**: 以 `(v128_t) __builtin_convertvector(` 从当前函数返回。

### Lines 2017-2040

````c
      (__f16x4){((__f16x8)__a)[0], ((__f16x8)__a)[1], ((__f16x8)__a)[2],
                ((__f16x8)__a)[3]},
      __f32x4);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_relaxed_madd(v128_t __a,
                                                                 v128_t __b,
                                                                 v128_t __c) {
  return (v128_t)__builtin_wasm_relaxed_madd_f16x8((__f16x8)__a, (__f16x8)__b,
                                                   (__f16x8)__c);
}

static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_relaxed_nmadd(v128_t __a,
                                                                  v128_t __b,
                                                                  v128_t __c) {
  return (v128_t)__builtin_wasm_relaxed_nmadd_f16x8((__f16x8)__a, (__f16x8)__b,
                                                    (__f16x8)__c);
}

// Deprecated intrinsics

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_i8x16_swizzle")
wasm_v8x16_swizzle(v128_t __a, v128_t __b) {
  return wasm_i8x16_swizzle(__a, __b);
````
- **L2017 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(__f16x4){((__f16x8)__a)[0], ((__f16x8)__a)[1], ((__f16x8)__a)[2],`.
  **L2017 CN**: 继续一个多行参数列表、初始化器或聚合项：`(__f16x4){((__f16x8)__a)[0], ((__f16x8)__a)[1], ((__f16x8)__a)[2],`。
- **L2018 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `((__f16x8)__a)[3]},`.
  **L2018 CN**: 继续一个多行参数列表、初始化器或聚合项：`((__f16x8)__a)[3]},`。
- **L2019 EN**: Adds a standalone statement or declaration: `__f32x4);`.
  **L2019 CN**: 添加一条独立语句或声明：`__f32x4);`。
- **L2020 EN**: Closes the current lexical scope or compound statement.
  **L2020 CN**: 结束当前词法作用域或复合语句块。
- **L2021 EN**: Blank line separating nearby declarations or logic blocks.
  **L2021 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2022 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_relaxed_madd(v128_t __a,`.
  **L2022 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_relaxed_madd(v128_t __a,`。
- **L2023 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `v128_t __b,`.
  **L2023 CN**: 继续一个多行参数列表、初始化器或聚合项：`v128_t __b,`。
- **L2024 EN**: Continues the surrounding expression or declaration: `v128_t __c) {`.
  **L2024 CN**: 继续构造周围的表达式或声明：`v128_t __c) {`。
- **L2025 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_madd_f16x8((__f16x8)__a, (__f16x8)__b,`.
  **L2025 CN**: 以 `(v128_t)__builtin_wasm_relaxed_madd_f16x8((__f16x8)__a, (__f16x8)__b,` 从当前函数返回。
- **L2026 EN**: Executes a call or declaration centered on `statement`.
  **L2026 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2027 EN**: Closes the current lexical scope or compound statement.
  **L2027 CN**: 结束当前词法作用域或复合语句块。
- **L2028 EN**: Blank line separating nearby declarations or logic blocks.
  **L2028 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2029 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_relaxed_nmadd(v128_t __a,`.
  **L2029 CN**: 继续一个多行参数列表、初始化器或聚合项：`static __inline__ v128_t __FP16_FN_ATTRS wasm_f16x8_relaxed_nmadd(v128_t __a,`。
- **L2030 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `v128_t __b,`.
  **L2030 CN**: 继续一个多行参数列表、初始化器或聚合项：`v128_t __b,`。
- **L2031 EN**: Continues the surrounding expression or declaration: `v128_t __c) {`.
  **L2031 CN**: 继续构造周围的表达式或声明：`v128_t __c) {`。
- **L2032 EN**: Returns from the current function with `(v128_t)__builtin_wasm_relaxed_nmadd_f16x8((__f16x8)__a, (__f16x8)__b,`.
  **L2032 CN**: 以 `(v128_t)__builtin_wasm_relaxed_nmadd_f16x8((__f16x8)__a, (__f16x8)__b,` 从当前函数返回。
- **L2033 EN**: Executes a call or declaration centered on `statement`.
  **L2033 CN**: 执行以 `statement` 为核心的调用或声明。
- **L2034 EN**: Closes the current lexical scope or compound statement.
  **L2034 CN**: 结束当前词法作用域或复合语句块。
- **L2035 EN**: Blank line separating nearby declarations or logic blocks.
  **L2035 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2036 EN**: Comment explains nearby logic, constraints, or intent: `Deprecated intrinsics`.
  **L2036 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Deprecated intrinsics`。
- **L2037 EN**: Blank line separating nearby declarations or logic blocks.
  **L2037 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2038 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2038 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2039 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_v8x16_swizzle(v128_t __a, v128_t __b) {`.
  **L2039 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_v8x16_swizzle(v128_t __a, v128_t __b) {`。
- **L2040 EN**: Returns from the current function with `wasm_i8x16_swizzle(__a, __b)`.
  **L2040 CN**: 以 `wasm_i8x16_swizzle(__a, __b)` 从当前函数返回。

### Lines 2041-2064

````c
}

static __inline__ bool __DEPRECATED_FN_ATTRS("wasm_v128_any_true")
wasm_i8x16_any_true(v128_t __a) {
  return wasm_v128_any_true(__a);
}

static __inline__ bool __DEPRECATED_FN_ATTRS("wasm_v128_any_true")
wasm_i16x8_any_true(v128_t __a) {
  return wasm_v128_any_true(__a);
}

static __inline__ bool __DEPRECATED_FN_ATTRS("wasm_v128_any_true")
wasm_i32x4_any_true(v128_t __a) {
  return wasm_v128_any_true(__a);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_i8x16_add_sat")
wasm_i8x16_add_saturate(v128_t __a, v128_t __b) {
  return wasm_i8x16_add_sat(__a, __b);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_u8x16_add_sat")
wasm_u8x16_add_saturate(v128_t __a, v128_t __b) {
````
- **L2041 EN**: Closes the current lexical scope or compound statement.
  **L2041 CN**: 结束当前词法作用域或复合语句块。
- **L2042 EN**: Blank line separating nearby declarations or logic blocks.
  **L2042 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2043 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2043 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2044 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i8x16_any_true(v128_t __a) {`.
  **L2044 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i8x16_any_true(v128_t __a) {`。
- **L2045 EN**: Returns from the current function with `wasm_v128_any_true(__a)`.
  **L2045 CN**: 以 `wasm_v128_any_true(__a)` 从当前函数返回。
- **L2046 EN**: Closes the current lexical scope or compound statement.
  **L2046 CN**: 结束当前词法作用域或复合语句块。
- **L2047 EN**: Blank line separating nearby declarations or logic blocks.
  **L2047 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2048 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2048 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2049 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_any_true(v128_t __a) {`.
  **L2049 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_any_true(v128_t __a) {`。
- **L2050 EN**: Returns from the current function with `wasm_v128_any_true(__a)`.
  **L2050 CN**: 以 `wasm_v128_any_true(__a)` 从当前函数返回。
- **L2051 EN**: Closes the current lexical scope or compound statement.
  **L2051 CN**: 结束当前词法作用域或复合语句块。
- **L2052 EN**: Blank line separating nearby declarations or logic blocks.
  **L2052 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2053 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2053 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2054 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_any_true(v128_t __a) {`.
  **L2054 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_any_true(v128_t __a) {`。
- **L2055 EN**: Returns from the current function with `wasm_v128_any_true(__a)`.
  **L2055 CN**: 以 `wasm_v128_any_true(__a)` 从当前函数返回。
- **L2056 EN**: Closes the current lexical scope or compound statement.
  **L2056 CN**: 结束当前词法作用域或复合语句块。
- **L2057 EN**: Blank line separating nearby declarations or logic blocks.
  **L2057 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2058 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2058 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2059 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i8x16_add_saturate(v128_t __a, v128_t __b) {`.
  **L2059 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i8x16_add_saturate(v128_t __a, v128_t __b) {`。
- **L2060 EN**: Returns from the current function with `wasm_i8x16_add_sat(__a, __b)`.
  **L2060 CN**: 以 `wasm_i8x16_add_sat(__a, __b)` 从当前函数返回。
- **L2061 EN**: Closes the current lexical scope or compound statement.
  **L2061 CN**: 结束当前词法作用域或复合语句块。
- **L2062 EN**: Blank line separating nearby declarations or logic blocks.
  **L2062 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2063 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2063 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2064 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u8x16_add_saturate(v128_t __a, v128_t __b) {`.
  **L2064 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u8x16_add_saturate(v128_t __a, v128_t __b) {`。

### Lines 2065-2088

````c
  return wasm_u8x16_add_sat(__a, __b);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_i8x16_sub_sat")
wasm_i8x16_sub_saturate(v128_t __a, v128_t __b) {
  return wasm_i8x16_sub_sat(__a, __b);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_u8x16_sub_sat")
wasm_u8x16_sub_saturate(v128_t __a, v128_t __b) {
  return wasm_u8x16_sub_sat(__a, __b);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_i16x8_add_sat")
wasm_i16x8_add_saturate(v128_t __a, v128_t __b) {
  return wasm_i16x8_add_sat(__a, __b);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_u16x8_add_sat")
wasm_u16x8_add_saturate(v128_t __a, v128_t __b) {
  return wasm_u16x8_add_sat(__a, __b);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_i16x8_sub_sat")
````
- **L2065 EN**: Returns from the current function with `wasm_u8x16_add_sat(__a, __b)`.
  **L2065 CN**: 以 `wasm_u8x16_add_sat(__a, __b)` 从当前函数返回。
- **L2066 EN**: Closes the current lexical scope or compound statement.
  **L2066 CN**: 结束当前词法作用域或复合语句块。
- **L2067 EN**: Blank line separating nearby declarations or logic blocks.
  **L2067 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2068 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2068 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2069 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i8x16_sub_saturate(v128_t __a, v128_t __b) {`.
  **L2069 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i8x16_sub_saturate(v128_t __a, v128_t __b) {`。
- **L2070 EN**: Returns from the current function with `wasm_i8x16_sub_sat(__a, __b)`.
  **L2070 CN**: 以 `wasm_i8x16_sub_sat(__a, __b)` 从当前函数返回。
- **L2071 EN**: Closes the current lexical scope or compound statement.
  **L2071 CN**: 结束当前词法作用域或复合语句块。
- **L2072 EN**: Blank line separating nearby declarations or logic blocks.
  **L2072 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2073 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2073 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2074 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u8x16_sub_saturate(v128_t __a, v128_t __b) {`.
  **L2074 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u8x16_sub_saturate(v128_t __a, v128_t __b) {`。
- **L2075 EN**: Returns from the current function with `wasm_u8x16_sub_sat(__a, __b)`.
  **L2075 CN**: 以 `wasm_u8x16_sub_sat(__a, __b)` 从当前函数返回。
- **L2076 EN**: Closes the current lexical scope or compound statement.
  **L2076 CN**: 结束当前词法作用域或复合语句块。
- **L2077 EN**: Blank line separating nearby declarations or logic blocks.
  **L2077 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2078 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2078 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2079 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_add_saturate(v128_t __a, v128_t __b) {`.
  **L2079 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_add_saturate(v128_t __a, v128_t __b) {`。
- **L2080 EN**: Returns from the current function with `wasm_i16x8_add_sat(__a, __b)`.
  **L2080 CN**: 以 `wasm_i16x8_add_sat(__a, __b)` 从当前函数返回。
- **L2081 EN**: Closes the current lexical scope or compound statement.
  **L2081 CN**: 结束当前词法作用域或复合语句块。
- **L2082 EN**: Blank line separating nearby declarations or logic blocks.
  **L2082 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2083 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2083 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2084 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u16x8_add_saturate(v128_t __a, v128_t __b) {`.
  **L2084 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u16x8_add_saturate(v128_t __a, v128_t __b) {`。
- **L2085 EN**: Returns from the current function with `wasm_u16x8_add_sat(__a, __b)`.
  **L2085 CN**: 以 `wasm_u16x8_add_sat(__a, __b)` 从当前函数返回。
- **L2086 EN**: Closes the current lexical scope or compound statement.
  **L2086 CN**: 结束当前词法作用域或复合语句块。
- **L2087 EN**: Blank line separating nearby declarations or logic blocks.
  **L2087 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2088 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2088 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。

### Lines 2089-2112

````c
wasm_i16x8_sub_saturate(v128_t __a, v128_t __b) {
  return wasm_i16x8_sub_sat(__a, __b);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_u16x8_sub_sat")
wasm_u16x8_sub_saturate(v128_t __a, v128_t __b) {
  return wasm_u16x8_sub_sat(__a, __b);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_i16x8_extend_low_i8x16")
wasm_i16x8_widen_low_i8x16(v128_t __a) {
  return wasm_i16x8_extend_low_i8x16(__a);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_i16x8_extend_high_i8x16")
wasm_i16x8_widen_high_i8x16(v128_t __a) {
  return wasm_i16x8_extend_high_i8x16(__a);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_u16x8_extend_low_u8x16")
wasm_i16x8_widen_low_u8x16(v128_t __a) {
  return wasm_u16x8_extend_low_u8x16(__a);
}

````
- **L2089 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_sub_saturate(v128_t __a, v128_t __b) {`.
  **L2089 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_sub_saturate(v128_t __a, v128_t __b) {`。
- **L2090 EN**: Returns from the current function with `wasm_i16x8_sub_sat(__a, __b)`.
  **L2090 CN**: 以 `wasm_i16x8_sub_sat(__a, __b)` 从当前函数返回。
- **L2091 EN**: Closes the current lexical scope or compound statement.
  **L2091 CN**: 结束当前词法作用域或复合语句块。
- **L2092 EN**: Blank line separating nearby declarations or logic blocks.
  **L2092 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2093 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2093 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2094 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u16x8_sub_saturate(v128_t __a, v128_t __b) {`.
  **L2094 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u16x8_sub_saturate(v128_t __a, v128_t __b) {`。
- **L2095 EN**: Returns from the current function with `wasm_u16x8_sub_sat(__a, __b)`.
  **L2095 CN**: 以 `wasm_u16x8_sub_sat(__a, __b)` 从当前函数返回。
- **L2096 EN**: Closes the current lexical scope or compound statement.
  **L2096 CN**: 结束当前词法作用域或复合语句块。
- **L2097 EN**: Blank line separating nearby declarations or logic blocks.
  **L2097 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2098 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2098 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2099 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_widen_low_i8x16(v128_t __a) {`.
  **L2099 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_widen_low_i8x16(v128_t __a) {`。
- **L2100 EN**: Returns from the current function with `wasm_i16x8_extend_low_i8x16(__a)`.
  **L2100 CN**: 以 `wasm_i16x8_extend_low_i8x16(__a)` 从当前函数返回。
- **L2101 EN**: Closes the current lexical scope or compound statement.
  **L2101 CN**: 结束当前词法作用域或复合语句块。
- **L2102 EN**: Blank line separating nearby declarations or logic blocks.
  **L2102 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2103 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2103 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2104 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_widen_high_i8x16(v128_t __a) {`.
  **L2104 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_widen_high_i8x16(v128_t __a) {`。
- **L2105 EN**: Returns from the current function with `wasm_i16x8_extend_high_i8x16(__a)`.
  **L2105 CN**: 以 `wasm_i16x8_extend_high_i8x16(__a)` 从当前函数返回。
- **L2106 EN**: Closes the current lexical scope or compound statement.
  **L2106 CN**: 结束当前词法作用域或复合语句块。
- **L2107 EN**: Blank line separating nearby declarations or logic blocks.
  **L2107 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2108 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2108 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2109 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_widen_low_u8x16(v128_t __a) {`.
  **L2109 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_widen_low_u8x16(v128_t __a) {`。
- **L2110 EN**: Returns from the current function with `wasm_u16x8_extend_low_u8x16(__a)`.
  **L2110 CN**: 以 `wasm_u16x8_extend_low_u8x16(__a)` 从当前函数返回。
- **L2111 EN**: Closes the current lexical scope or compound statement.
  **L2111 CN**: 结束当前词法作用域或复合语句块。
- **L2112 EN**: Blank line separating nearby declarations or logic blocks.
  **L2112 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 2113-2136

````c
static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_u16x8_extend_high_u8x16")
wasm_i16x8_widen_high_u8x16(v128_t __a) {
  return wasm_u16x8_extend_high_u8x16(__a);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_i32x4_extend_low_i16x8")
wasm_i32x4_widen_low_i16x8(v128_t __a) {
  return wasm_i32x4_extend_low_i16x8(__a);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_i32x4_extend_high_i16x8")
wasm_i32x4_widen_high_i16x8(v128_t __a) {
  return wasm_i32x4_extend_high_i16x8(__a);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_u32x4_extend_low_u16x8")
wasm_i32x4_widen_low_u16x8(v128_t __a) {
  return wasm_u32x4_extend_low_u16x8(__a);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_u32x4_extend_high_u16x8")
wasm_i32x4_widen_high_u16x8(v128_t __a) {
  return wasm_u32x4_extend_high_u16x8(__a);
}
````
- **L2113 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2113 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2114 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i16x8_widen_high_u8x16(v128_t __a) {`.
  **L2114 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i16x8_widen_high_u8x16(v128_t __a) {`。
- **L2115 EN**: Returns from the current function with `wasm_u16x8_extend_high_u8x16(__a)`.
  **L2115 CN**: 以 `wasm_u16x8_extend_high_u8x16(__a)` 从当前函数返回。
- **L2116 EN**: Closes the current lexical scope or compound statement.
  **L2116 CN**: 结束当前词法作用域或复合语句块。
- **L2117 EN**: Blank line separating nearby declarations or logic blocks.
  **L2117 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2118 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2118 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2119 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_widen_low_i16x8(v128_t __a) {`.
  **L2119 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_widen_low_i16x8(v128_t __a) {`。
- **L2120 EN**: Returns from the current function with `wasm_i32x4_extend_low_i16x8(__a)`.
  **L2120 CN**: 以 `wasm_i32x4_extend_low_i16x8(__a)` 从当前函数返回。
- **L2121 EN**: Closes the current lexical scope or compound statement.
  **L2121 CN**: 结束当前词法作用域或复合语句块。
- **L2122 EN**: Blank line separating nearby declarations or logic blocks.
  **L2122 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2123 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2123 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2124 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_widen_high_i16x8(v128_t __a) {`.
  **L2124 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_widen_high_i16x8(v128_t __a) {`。
- **L2125 EN**: Returns from the current function with `wasm_i32x4_extend_high_i16x8(__a)`.
  **L2125 CN**: 以 `wasm_i32x4_extend_high_i16x8(__a)` 从当前函数返回。
- **L2126 EN**: Closes the current lexical scope or compound statement.
  **L2126 CN**: 结束当前词法作用域或复合语句块。
- **L2127 EN**: Blank line separating nearby declarations or logic blocks.
  **L2127 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2128 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2128 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2129 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_widen_low_u16x8(v128_t __a) {`.
  **L2129 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_widen_low_u16x8(v128_t __a) {`。
- **L2130 EN**: Returns from the current function with `wasm_u32x4_extend_low_u16x8(__a)`.
  **L2130 CN**: 以 `wasm_u32x4_extend_low_u16x8(__a)` 从当前函数返回。
- **L2131 EN**: Closes the current lexical scope or compound statement.
  **L2131 CN**: 结束当前词法作用域或复合语句块。
- **L2132 EN**: Blank line separating nearby declarations or logic blocks.
  **L2132 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2133 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2133 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2134 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_widen_high_u16x8(v128_t __a) {`.
  **L2134 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_widen_high_u16x8(v128_t __a) {`。
- **L2135 EN**: Returns from the current function with `wasm_u32x4_extend_high_u16x8(__a)`.
  **L2135 CN**: 以 `wasm_u32x4_extend_high_u16x8(__a)` 从当前函数返回。
- **L2136 EN**: Closes the current lexical scope or compound statement.
  **L2136 CN**: 结束当前词法作用域或复合语句块。

### Lines 2137-2152

````c

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_i32x4_trunc_sat_f32x4")
wasm_i32x4_trunc_saturate_f32x4(v128_t __a) {
  return wasm_i32x4_trunc_sat_f32x4(__a);
}

static __inline__ v128_t __DEPRECATED_FN_ATTRS("wasm_u32x4_trunc_sat_f32x4")
wasm_u32x4_trunc_saturate_f32x4(v128_t __a) {
  return wasm_u32x4_trunc_sat_f32x4(__a);
}

// Undefine helper macros
#undef __DEFAULT_FN_ATTRS
#undef __DEPRECATED_FN_ATTRS

#endif // __WASM_SIMD128_H
````
- **L2137 EN**: Blank line separating nearby declarations or logic blocks.
  **L2137 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2138 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2138 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2139 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_i32x4_trunc_saturate_f32x4(v128_t __a) {`.
  **L2139 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_i32x4_trunc_saturate_f32x4(v128_t __a) {`。
- **L2140 EN**: Returns from the current function with `wasm_i32x4_trunc_sat_f32x4(__a)`.
  **L2140 CN**: 以 `wasm_i32x4_trunc_sat_f32x4(__a)` 从当前函数返回。
- **L2141 EN**: Closes the current lexical scope or compound statement.
  **L2141 CN**: 结束当前词法作用域或复合语句块。
- **L2142 EN**: Blank line separating nearby declarations or logic blocks.
  **L2142 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2143 EN**: Continues logic associated with callable symbol `__DEPRECATED_FN_ATTRS`.
  **L2143 CN**: 继续与可调用符号 `__DEPRECATED_FN_ATTRS` 相关的逻辑。
- **L2144 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `wasm_u32x4_trunc_saturate_f32x4(v128_t __a) {`.
  **L2144 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`wasm_u32x4_trunc_saturate_f32x4(v128_t __a) {`。
- **L2145 EN**: Returns from the current function with `wasm_u32x4_trunc_sat_f32x4(__a)`.
  **L2145 CN**: 以 `wasm_u32x4_trunc_sat_f32x4(__a)` 从当前函数返回。
- **L2146 EN**: Closes the current lexical scope or compound statement.
  **L2146 CN**: 结束当前词法作用域或复合语句块。
- **L2147 EN**: Blank line separating nearby declarations or logic blocks.
  **L2147 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2148 EN**: Comment explains nearby logic, constraints, or intent: `Undefine helper macros`.
  **L2148 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Undefine helper macros`。
- **L2149 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEFAULT_FN_ATTRS`.
  **L2149 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEFAULT_FN_ATTRS`。
- **L2150 EN**: Undefines a macro to keep temporary preprocessor state local: `#undef __DEPRECATED_FN_ATTRS`.
  **L2150 CN**: 取消一个宏定义，使临时预处理状态保持局部：`#undef __DEPRECATED_FN_ATTRS`。
- **L2151 EN**: Blank line separating nearby declarations or logic blocks.
  **L2151 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2152 EN**: Closes the current preprocessor conditional block.
  **L2152 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **WebAssembly builtins / WebAssembly 内建接口**
- **Header composition through includes / 通过 include 组合头文件**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**
- **Inline wrapper functions / 内联包装函数**
- **Compiler-specific attributes / 编译器特定属性**
- **Type aliasing and ABI shaping / 类型别名与 ABI 约束**
- **Structured data declarations / 结构化数据声明**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**:
  - `stdbool.h`: Provides related header declarations. / 提供相关头文件声明。
  - `stdint.h`: Provides fixed-width integer declarations. / 提供定宽整数声明。
- **Conditional macros / 条件宏**: `__WASM_SIMD128_H`, `__DEPRECATED`, `__wasm_fp16__`
- **External builtins / 外部 builtin**: `__builtin_constant_p`, `__builtin_convertvector`, `__builtin_wasm_any_true_v128`, `__builtin_wasm_bitselect`, `__builtin_wasm_abs_i8x16`, `__builtin_wasm_all_true_i8x16`, `__builtin_wasm_bitmask_i8x16`, `__builtin_elementwise_popcount`, `__builtin_elementwise_add_sat`, `__builtin_elementwise_sub_sat`, `__builtin_elementwise_min`, `__builtin_elementwise_max`
