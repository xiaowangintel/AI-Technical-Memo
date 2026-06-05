# __clang_cuda_device_functions.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/__clang_cuda_device_functions.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: CUDA runtime support.
- **Purpose (CN)**: 该头文件主要作用是：CUDA runtime support。
- **Line Count / 行数**: 1572

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
/*===---- __clang_cuda_device_functions.h - CUDA runtime support -----------===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

#ifndef __CLANG_CUDA_DEVICE_FUNCTIONS_H__
#define __CLANG_CUDA_DEVICE_FUNCTIONS_H__

#ifndef __OPENMP_NVPTX__
#if CUDA_VERSION < 9000
#error This file is intended to be used with CUDA-9+ only.
#endif
#endif

// __DEVICE__ is a helper macro with common set of attributes for the wrappers
// we implement in this file. We need static in order to avoid emitting unused
// functions and __forceinline__ helps inlining these wrappers at -O1.
#pragma push_macro("__DEVICE__")
#ifdef __OPENMP_NVPTX__
#define __DEVICE__ static __attribute__((always_inline, nothrow))
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
- **L10 EN**: Starts a preprocessor conditional block: `#ifndef __CLANG_CUDA_DEVICE_FUNCTIONS_H__`.
  **L10 CN**: 开始一个预处理条件块：`#ifndef __CLANG_CUDA_DEVICE_FUNCTIONS_H__`。
- **L11 EN**: Defines macro `__CLANG_CUDA_DEVICE_FUNCTIONS_H__` for conditional compilation, shorthand, or API generation.
  **L11 CN**: 定义宏 `__CLANG_CUDA_DEVICE_FUNCTIONS_H__`，用于条件编译、简写或 API 生成。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Starts a preprocessor conditional block: `#ifndef __OPENMP_NVPTX__`.
  **L13 CN**: 开始一个预处理条件块：`#ifndef __OPENMP_NVPTX__`。
- **L14 EN**: Starts a preprocessor conditional block: `#if CUDA_VERSION < 9000`.
  **L14 CN**: 开始一个预处理条件块：`#if CUDA_VERSION < 9000`。
- **L15 EN**: Emits a compilation error for an unsupported configuration: `#error This file is intended to be used with CUDA-9+ only.`.
  **L15 CN**: 为不受支持的配置触发编译错误：`#error This file is intended to be used with CUDA-9+ only.`。
- **L16 EN**: Closes the current preprocessor conditional block.
  **L16 CN**: 结束当前预处理条件块。
- **L17 EN**: Closes the current preprocessor conditional block.
  **L17 CN**: 结束当前预处理条件块。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L19 EN**: Comment explains nearby logic, constraints, or intent: `__DEVICE__ is a helper macro with common set of attributes for the wrappers`.
  **L19 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__DEVICE__ is a helper macro with common set of attributes for the wrappers`。
- **L20 EN**: Comment explains nearby logic, constraints, or intent: `we implement in this file. We need static in order to avoid emitting unused`.
  **L20 CN**: 注释解释附近代码的逻辑、约束或设计意图：`we implement in this file. We need static in order to avoid emitting unused`。
- **L21 EN**: Comment explains nearby logic, constraints, or intent: `functions and __forceinline__ helps inlining these wrappers at -O1.`.
  **L21 CN**: 注释解释附近代码的逻辑、约束或设计意图：`functions and __forceinline__ helps inlining these wrappers at -O1.`。
- **L22 EN**: Applies a compiler pragma that adjusts parsing, attributes, or macro state: `#pragma push_macro("__DEVICE__")`.
  **L22 CN**: 应用编译器 pragma，调整解析、属性或宏状态：`#pragma push_macro("__DEVICE__")`。
- **L23 EN**: Starts a preprocessor conditional block: `#ifdef __OPENMP_NVPTX__`.
  **L23 CN**: 开始一个预处理条件块：`#ifdef __OPENMP_NVPTX__`。
- **L24 EN**: Defines macro `__DEVICE__` for conditional compilation, shorthand, or API generation.
  **L24 CN**: 定义宏 `__DEVICE__`，用于条件编译、简写或 API 生成。

### Lines 25-48

````c
#else
#define __DEVICE__ static __device__ __forceinline__
#endif

__DEVICE__ int __all(int __a) { return __nvvm_vote_all(__a); }
__DEVICE__ int __any(int __a) { return __nvvm_vote_any(__a); }
__DEVICE__ unsigned int __ballot(int __a) { return __nvvm_vote_ballot(__a); }
__DEVICE__ unsigned int __brev(unsigned int __a) { return __nv_brev(__a); }
__DEVICE__ unsigned long long __brevll(unsigned long long __a) {
  return __nv_brevll(__a);
}
#if defined(__cplusplus)
__DEVICE__ void __brkpt() { __asm__ __volatile__("brkpt;"); }
__DEVICE__ void __brkpt(int __a) { __brkpt(); }
#else
__DEVICE__ void __attribute__((overloadable)) __brkpt(void) {
  __asm__ __volatile__("brkpt;");
}
__DEVICE__ void __attribute__((overloadable)) __brkpt(int __a) { __brkpt(); }
#endif
__DEVICE__ unsigned int __byte_perm(unsigned int __a, unsigned int __b,
                                    unsigned int __c) {
  return __nv_byte_perm(__a, __b, __c);
}
````
- **L25 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L25 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L26 EN**: Defines macro `__DEVICE__` for conditional compilation, shorthand, or API generation.
  **L26 CN**: 定义宏 `__DEVICE__`，用于条件编译、简写或 API 生成。
- **L27 EN**: Closes the current preprocessor conditional block.
  **L27 CN**: 结束当前预处理条件块。
- **L28 EN**: Blank line separating nearby declarations or logic blocks.
  **L28 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L29 EN**: Continues logic associated with callable symbol `__all`.
  **L29 CN**: 继续与可调用符号 `__all` 相关的逻辑。
- **L30 EN**: Continues logic associated with callable symbol `__any`.
  **L30 CN**: 继续与可调用符号 `__any` 相关的逻辑。
- **L31 EN**: Continues logic associated with callable symbol `__ballot`.
  **L31 CN**: 继续与可调用符号 `__ballot` 相关的逻辑。
- **L32 EN**: Continues logic associated with callable symbol `__brev`.
  **L32 CN**: 继续与可调用符号 `__brev` 相关的逻辑。
- **L33 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned long long __brevll(unsigned long long __a) {`.
  **L33 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned long long __brevll(unsigned long long __a) {`。
- **L34 EN**: Returns from the current function with `__nv_brevll(__a)`.
  **L34 CN**: 以 `__nv_brevll(__a)` 从当前函数返回。
- **L35 EN**: Closes the current lexical scope or compound statement.
  **L35 CN**: 结束当前词法作用域或复合语句块。
- **L36 EN**: Starts a preprocessor conditional block: `#if defined(__cplusplus)`.
  **L36 CN**: 开始一个预处理条件块：`#if defined(__cplusplus)`。
- **L37 EN**: Continues logic associated with callable symbol `__brkpt`.
  **L37 CN**: 继续与可调用符号 `__brkpt` 相关的逻辑。
- **L38 EN**: Continues logic associated with callable symbol `__brkpt`.
  **L38 CN**: 继续与可调用符号 `__brkpt` 相关的逻辑。
- **L39 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L39 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L40 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__DEVICE__ void __attribute__((overloadable)) __brkpt(void) {`.
  **L40 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__DEVICE__ void __attribute__((overloadable)) __brkpt(void) {`。
- **L41 EN**: Executes a call or declaration centered on `__volatile__`.
  **L41 CN**: 执行以 `__volatile__` 为核心的调用或声明。
- **L42 EN**: Closes the current lexical scope or compound statement.
  **L42 CN**: 结束当前词法作用域或复合语句块。
- **L43 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__DEVICE__ void __attribute__((overloadable)) __brkpt(int __a) { __brkpt(); }`.
  **L43 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__DEVICE__ void __attribute__((overloadable)) __brkpt(int __a) { __brkpt(); }`。
- **L44 EN**: Closes the current preprocessor conditional block.
  **L44 CN**: 结束当前预处理条件块。
- **L45 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __byte_perm(unsigned int __a, unsigned int __b,`.
  **L45 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __byte_perm(unsigned int __a, unsigned int __b,`。
- **L46 EN**: Continues the surrounding expression or declaration: `unsigned int __c) {`.
  **L46 CN**: 继续构造周围的表达式或声明：`unsigned int __c) {`。
- **L47 EN**: Returns from the current function with `__nv_byte_perm(__a, __b, __c)`.
  **L47 CN**: 以 `__nv_byte_perm(__a, __b, __c)` 从当前函数返回。
- **L48 EN**: Closes the current lexical scope or compound statement.
  **L48 CN**: 结束当前词法作用域或复合语句块。

### Lines 49-72

````c
__DEVICE__ int __clz(int __a) { return __nv_clz(__a); }
__DEVICE__ int __clzll(long long __a) { return __nv_clzll(__a); }
__DEVICE__ float __cosf(float __a) { return __nv_fast_cosf(__a); }
__DEVICE__ double __dAtomicAdd(double *__p, double __v) {
  return __nvvm_atom_add_gen_d(__p, __v);
}
__DEVICE__ double __dAtomicAdd_block(double *__p, double __v) {
  return __nvvm_atom_cta_add_gen_d(__p, __v);
}
__DEVICE__ double __dAtomicAdd_system(double *__p, double __v) {
  return __nvvm_atom_sys_add_gen_d(__p, __v);
}
__DEVICE__ double __dadd_rd(double __a, double __b) {
  return __nv_dadd_rd(__a, __b);
}
__DEVICE__ double __dadd_rn(double __a, double __b) {
  return __nv_dadd_rn(__a, __b);
}
__DEVICE__ double __dadd_ru(double __a, double __b) {
  return __nv_dadd_ru(__a, __b);
}
__DEVICE__ double __dadd_rz(double __a, double __b) {
  return __nv_dadd_rz(__a, __b);
}
````
- **L49 EN**: Continues logic associated with callable symbol `__clz`.
  **L49 CN**: 继续与可调用符号 `__clz` 相关的逻辑。
- **L50 EN**: Continues logic associated with callable symbol `__clzll`.
  **L50 CN**: 继续与可调用符号 `__clzll` 相关的逻辑。
- **L51 EN**: Continues logic associated with callable symbol `__cosf`.
  **L51 CN**: 继续与可调用符号 `__cosf` 相关的逻辑。
- **L52 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __dAtomicAdd(double *__p, double __v) {`.
  **L52 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __dAtomicAdd(double *__p, double __v) {`。
- **L53 EN**: Returns from the current function with `__nvvm_atom_add_gen_d(__p, __v)`.
  **L53 CN**: 以 `__nvvm_atom_add_gen_d(__p, __v)` 从当前函数返回。
- **L54 EN**: Closes the current lexical scope or compound statement.
  **L54 CN**: 结束当前词法作用域或复合语句块。
- **L55 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __dAtomicAdd_block(double *__p, double __v) {`.
  **L55 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __dAtomicAdd_block(double *__p, double __v) {`。
- **L56 EN**: Returns from the current function with `__nvvm_atom_cta_add_gen_d(__p, __v)`.
  **L56 CN**: 以 `__nvvm_atom_cta_add_gen_d(__p, __v)` 从当前函数返回。
- **L57 EN**: Closes the current lexical scope or compound statement.
  **L57 CN**: 结束当前词法作用域或复合语句块。
- **L58 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __dAtomicAdd_system(double *__p, double __v) {`.
  **L58 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __dAtomicAdd_system(double *__p, double __v) {`。
- **L59 EN**: Returns from the current function with `__nvvm_atom_sys_add_gen_d(__p, __v)`.
  **L59 CN**: 以 `__nvvm_atom_sys_add_gen_d(__p, __v)` 从当前函数返回。
- **L60 EN**: Closes the current lexical scope or compound statement.
  **L60 CN**: 结束当前词法作用域或复合语句块。
- **L61 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __dadd_rd(double __a, double __b) {`.
  **L61 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __dadd_rd(double __a, double __b) {`。
- **L62 EN**: Returns from the current function with `__nv_dadd_rd(__a, __b)`.
  **L62 CN**: 以 `__nv_dadd_rd(__a, __b)` 从当前函数返回。
- **L63 EN**: Closes the current lexical scope or compound statement.
  **L63 CN**: 结束当前词法作用域或复合语句块。
- **L64 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __dadd_rn(double __a, double __b) {`.
  **L64 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __dadd_rn(double __a, double __b) {`。
- **L65 EN**: Returns from the current function with `__nv_dadd_rn(__a, __b)`.
  **L65 CN**: 以 `__nv_dadd_rn(__a, __b)` 从当前函数返回。
- **L66 EN**: Closes the current lexical scope or compound statement.
  **L66 CN**: 结束当前词法作用域或复合语句块。
- **L67 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __dadd_ru(double __a, double __b) {`.
  **L67 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __dadd_ru(double __a, double __b) {`。
- **L68 EN**: Returns from the current function with `__nv_dadd_ru(__a, __b)`.
  **L68 CN**: 以 `__nv_dadd_ru(__a, __b)` 从当前函数返回。
- **L69 EN**: Closes the current lexical scope or compound statement.
  **L69 CN**: 结束当前词法作用域或复合语句块。
- **L70 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __dadd_rz(double __a, double __b) {`.
  **L70 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __dadd_rz(double __a, double __b) {`。
- **L71 EN**: Returns from the current function with `__nv_dadd_rz(__a, __b)`.
  **L71 CN**: 以 `__nv_dadd_rz(__a, __b)` 从当前函数返回。
- **L72 EN**: Closes the current lexical scope or compound statement.
  **L72 CN**: 结束当前词法作用域或复合语句块。

### Lines 73-96

````c
__DEVICE__ double __ddiv_rd(double __a, double __b) {
  return __nv_ddiv_rd(__a, __b);
}
__DEVICE__ double __ddiv_rn(double __a, double __b) {
  return __nv_ddiv_rn(__a, __b);
}
__DEVICE__ double __ddiv_ru(double __a, double __b) {
  return __nv_ddiv_ru(__a, __b);
}
__DEVICE__ double __ddiv_rz(double __a, double __b) {
  return __nv_ddiv_rz(__a, __b);
}
__DEVICE__ double __dmul_rd(double __a, double __b) {
  return __nv_dmul_rd(__a, __b);
}
__DEVICE__ double __dmul_rn(double __a, double __b) {
  return __nv_dmul_rn(__a, __b);
}
__DEVICE__ double __dmul_ru(double __a, double __b) {
  return __nv_dmul_ru(__a, __b);
}
__DEVICE__ double __dmul_rz(double __a, double __b) {
  return __nv_dmul_rz(__a, __b);
}
````
- **L73 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __ddiv_rd(double __a, double __b) {`.
  **L73 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __ddiv_rd(double __a, double __b) {`。
- **L74 EN**: Returns from the current function with `__nv_ddiv_rd(__a, __b)`.
  **L74 CN**: 以 `__nv_ddiv_rd(__a, __b)` 从当前函数返回。
- **L75 EN**: Closes the current lexical scope or compound statement.
  **L75 CN**: 结束当前词法作用域或复合语句块。
- **L76 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __ddiv_rn(double __a, double __b) {`.
  **L76 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __ddiv_rn(double __a, double __b) {`。
- **L77 EN**: Returns from the current function with `__nv_ddiv_rn(__a, __b)`.
  **L77 CN**: 以 `__nv_ddiv_rn(__a, __b)` 从当前函数返回。
- **L78 EN**: Closes the current lexical scope or compound statement.
  **L78 CN**: 结束当前词法作用域或复合语句块。
- **L79 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __ddiv_ru(double __a, double __b) {`.
  **L79 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __ddiv_ru(double __a, double __b) {`。
- **L80 EN**: Returns from the current function with `__nv_ddiv_ru(__a, __b)`.
  **L80 CN**: 以 `__nv_ddiv_ru(__a, __b)` 从当前函数返回。
- **L81 EN**: Closes the current lexical scope or compound statement.
  **L81 CN**: 结束当前词法作用域或复合语句块。
- **L82 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __ddiv_rz(double __a, double __b) {`.
  **L82 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __ddiv_rz(double __a, double __b) {`。
- **L83 EN**: Returns from the current function with `__nv_ddiv_rz(__a, __b)`.
  **L83 CN**: 以 `__nv_ddiv_rz(__a, __b)` 从当前函数返回。
- **L84 EN**: Closes the current lexical scope or compound statement.
  **L84 CN**: 结束当前词法作用域或复合语句块。
- **L85 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __dmul_rd(double __a, double __b) {`.
  **L85 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __dmul_rd(double __a, double __b) {`。
- **L86 EN**: Returns from the current function with `__nv_dmul_rd(__a, __b)`.
  **L86 CN**: 以 `__nv_dmul_rd(__a, __b)` 从当前函数返回。
- **L87 EN**: Closes the current lexical scope or compound statement.
  **L87 CN**: 结束当前词法作用域或复合语句块。
- **L88 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __dmul_rn(double __a, double __b) {`.
  **L88 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __dmul_rn(double __a, double __b) {`。
- **L89 EN**: Returns from the current function with `__nv_dmul_rn(__a, __b)`.
  **L89 CN**: 以 `__nv_dmul_rn(__a, __b)` 从当前函数返回。
- **L90 EN**: Closes the current lexical scope or compound statement.
  **L90 CN**: 结束当前词法作用域或复合语句块。
- **L91 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __dmul_ru(double __a, double __b) {`.
  **L91 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __dmul_ru(double __a, double __b) {`。
- **L92 EN**: Returns from the current function with `__nv_dmul_ru(__a, __b)`.
  **L92 CN**: 以 `__nv_dmul_ru(__a, __b)` 从当前函数返回。
- **L93 EN**: Closes the current lexical scope or compound statement.
  **L93 CN**: 结束当前词法作用域或复合语句块。
- **L94 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __dmul_rz(double __a, double __b) {`.
  **L94 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __dmul_rz(double __a, double __b) {`。
- **L95 EN**: Returns from the current function with `__nv_dmul_rz(__a, __b)`.
  **L95 CN**: 以 `__nv_dmul_rz(__a, __b)` 从当前函数返回。
- **L96 EN**: Closes the current lexical scope or compound statement.
  **L96 CN**: 结束当前词法作用域或复合语句块。

### Lines 97-120

````c
__DEVICE__ float __double2float_rd(double __a) {
  return __nv_double2float_rd(__a);
}
__DEVICE__ float __double2float_rn(double __a) {
  return __nv_double2float_rn(__a);
}
__DEVICE__ float __double2float_ru(double __a) {
  return __nv_double2float_ru(__a);
}
__DEVICE__ float __double2float_rz(double __a) {
  return __nv_double2float_rz(__a);
}
__DEVICE__ int __double2hiint(double __a) { return __nv_double2hiint(__a); }
__DEVICE__ int __double2int_rd(double __a) { return __nv_double2int_rd(__a); }
__DEVICE__ int __double2int_rn(double __a) { return __nv_double2int_rn(__a); }
__DEVICE__ int __double2int_ru(double __a) { return __nv_double2int_ru(__a); }
__DEVICE__ int __double2int_rz(double __a) { return __nv_double2int_rz(__a); }
__DEVICE__ long long __double2ll_rd(double __a) {
  return __nv_double2ll_rd(__a);
}
__DEVICE__ long long __double2ll_rn(double __a) {
  return __nv_double2ll_rn(__a);
}
__DEVICE__ long long __double2ll_ru(double __a) {
````
- **L97 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __double2float_rd(double __a) {`.
  **L97 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __double2float_rd(double __a) {`。
- **L98 EN**: Returns from the current function with `__nv_double2float_rd(__a)`.
  **L98 CN**: 以 `__nv_double2float_rd(__a)` 从当前函数返回。
- **L99 EN**: Closes the current lexical scope or compound statement.
  **L99 CN**: 结束当前词法作用域或复合语句块。
- **L100 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __double2float_rn(double __a) {`.
  **L100 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __double2float_rn(double __a) {`。
- **L101 EN**: Returns from the current function with `__nv_double2float_rn(__a)`.
  **L101 CN**: 以 `__nv_double2float_rn(__a)` 从当前函数返回。
- **L102 EN**: Closes the current lexical scope or compound statement.
  **L102 CN**: 结束当前词法作用域或复合语句块。
- **L103 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __double2float_ru(double __a) {`.
  **L103 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __double2float_ru(double __a) {`。
- **L104 EN**: Returns from the current function with `__nv_double2float_ru(__a)`.
  **L104 CN**: 以 `__nv_double2float_ru(__a)` 从当前函数返回。
- **L105 EN**: Closes the current lexical scope or compound statement.
  **L105 CN**: 结束当前词法作用域或复合语句块。
- **L106 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __double2float_rz(double __a) {`.
  **L106 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __double2float_rz(double __a) {`。
- **L107 EN**: Returns from the current function with `__nv_double2float_rz(__a)`.
  **L107 CN**: 以 `__nv_double2float_rz(__a)` 从当前函数返回。
- **L108 EN**: Closes the current lexical scope or compound statement.
  **L108 CN**: 结束当前词法作用域或复合语句块。
- **L109 EN**: Continues logic associated with callable symbol `__double2hiint`.
  **L109 CN**: 继续与可调用符号 `__double2hiint` 相关的逻辑。
- **L110 EN**: Continues logic associated with callable symbol `__double2int_rd`.
  **L110 CN**: 继续与可调用符号 `__double2int_rd` 相关的逻辑。
- **L111 EN**: Continues logic associated with callable symbol `__double2int_rn`.
  **L111 CN**: 继续与可调用符号 `__double2int_rn` 相关的逻辑。
- **L112 EN**: Continues logic associated with callable symbol `__double2int_ru`.
  **L112 CN**: 继续与可调用符号 `__double2int_ru` 相关的逻辑。
- **L113 EN**: Continues logic associated with callable symbol `__double2int_rz`.
  **L113 CN**: 继续与可调用符号 `__double2int_rz` 相关的逻辑。
- **L114 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __double2ll_rd(double __a) {`.
  **L114 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __double2ll_rd(double __a) {`。
- **L115 EN**: Returns from the current function with `__nv_double2ll_rd(__a)`.
  **L115 CN**: 以 `__nv_double2ll_rd(__a)` 从当前函数返回。
- **L116 EN**: Closes the current lexical scope or compound statement.
  **L116 CN**: 结束当前词法作用域或复合语句块。
- **L117 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __double2ll_rn(double __a) {`.
  **L117 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __double2ll_rn(double __a) {`。
- **L118 EN**: Returns from the current function with `__nv_double2ll_rn(__a)`.
  **L118 CN**: 以 `__nv_double2ll_rn(__a)` 从当前函数返回。
- **L119 EN**: Closes the current lexical scope or compound statement.
  **L119 CN**: 结束当前词法作用域或复合语句块。
- **L120 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __double2ll_ru(double __a) {`.
  **L120 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __double2ll_ru(double __a) {`。

### Lines 121-144

````c
  return __nv_double2ll_ru(__a);
}
__DEVICE__ long long __double2ll_rz(double __a) {
  return __nv_double2ll_rz(__a);
}
__DEVICE__ int __double2loint(double __a) { return __nv_double2loint(__a); }
__DEVICE__ unsigned int __double2uint_rd(double __a) {
  return __nv_double2uint_rd(__a);
}
__DEVICE__ unsigned int __double2uint_rn(double __a) {
  return __nv_double2uint_rn(__a);
}
__DEVICE__ unsigned int __double2uint_ru(double __a) {
  return __nv_double2uint_ru(__a);
}
__DEVICE__ unsigned int __double2uint_rz(double __a) {
  return __nv_double2uint_rz(__a);
}
__DEVICE__ unsigned long long __double2ull_rd(double __a) {
  return __nv_double2ull_rd(__a);
}
__DEVICE__ unsigned long long __double2ull_rn(double __a) {
  return __nv_double2ull_rn(__a);
}
````
- **L121 EN**: Returns from the current function with `__nv_double2ll_ru(__a)`.
  **L121 CN**: 以 `__nv_double2ll_ru(__a)` 从当前函数返回。
- **L122 EN**: Closes the current lexical scope or compound statement.
  **L122 CN**: 结束当前词法作用域或复合语句块。
- **L123 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __double2ll_rz(double __a) {`.
  **L123 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __double2ll_rz(double __a) {`。
- **L124 EN**: Returns from the current function with `__nv_double2ll_rz(__a)`.
  **L124 CN**: 以 `__nv_double2ll_rz(__a)` 从当前函数返回。
- **L125 EN**: Closes the current lexical scope or compound statement.
  **L125 CN**: 结束当前词法作用域或复合语句块。
- **L126 EN**: Continues logic associated with callable symbol `__double2loint`.
  **L126 CN**: 继续与可调用符号 `__double2loint` 相关的逻辑。
- **L127 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __double2uint_rd(double __a) {`.
  **L127 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __double2uint_rd(double __a) {`。
- **L128 EN**: Returns from the current function with `__nv_double2uint_rd(__a)`.
  **L128 CN**: 以 `__nv_double2uint_rd(__a)` 从当前函数返回。
- **L129 EN**: Closes the current lexical scope or compound statement.
  **L129 CN**: 结束当前词法作用域或复合语句块。
- **L130 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __double2uint_rn(double __a) {`.
  **L130 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __double2uint_rn(double __a) {`。
- **L131 EN**: Returns from the current function with `__nv_double2uint_rn(__a)`.
  **L131 CN**: 以 `__nv_double2uint_rn(__a)` 从当前函数返回。
- **L132 EN**: Closes the current lexical scope or compound statement.
  **L132 CN**: 结束当前词法作用域或复合语句块。
- **L133 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __double2uint_ru(double __a) {`.
  **L133 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __double2uint_ru(double __a) {`。
- **L134 EN**: Returns from the current function with `__nv_double2uint_ru(__a)`.
  **L134 CN**: 以 `__nv_double2uint_ru(__a)` 从当前函数返回。
- **L135 EN**: Closes the current lexical scope or compound statement.
  **L135 CN**: 结束当前词法作用域或复合语句块。
- **L136 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __double2uint_rz(double __a) {`.
  **L136 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __double2uint_rz(double __a) {`。
- **L137 EN**: Returns from the current function with `__nv_double2uint_rz(__a)`.
  **L137 CN**: 以 `__nv_double2uint_rz(__a)` 从当前函数返回。
- **L138 EN**: Closes the current lexical scope or compound statement.
  **L138 CN**: 结束当前词法作用域或复合语句块。
- **L139 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned long long __double2ull_rd(double __a) {`.
  **L139 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned long long __double2ull_rd(double __a) {`。
- **L140 EN**: Returns from the current function with `__nv_double2ull_rd(__a)`.
  **L140 CN**: 以 `__nv_double2ull_rd(__a)` 从当前函数返回。
- **L141 EN**: Closes the current lexical scope or compound statement.
  **L141 CN**: 结束当前词法作用域或复合语句块。
- **L142 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned long long __double2ull_rn(double __a) {`.
  **L142 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned long long __double2ull_rn(double __a) {`。
- **L143 EN**: Returns from the current function with `__nv_double2ull_rn(__a)`.
  **L143 CN**: 以 `__nv_double2ull_rn(__a)` 从当前函数返回。
- **L144 EN**: Closes the current lexical scope or compound statement.
  **L144 CN**: 结束当前词法作用域或复合语句块。

### Lines 145-168

````c
__DEVICE__ unsigned long long __double2ull_ru(double __a) {
  return __nv_double2ull_ru(__a);
}
__DEVICE__ unsigned long long __double2ull_rz(double __a) {
  return __nv_double2ull_rz(__a);
}
__DEVICE__ long long __double_as_longlong(double __a) {
  return __nv_double_as_longlong(__a);
}
__DEVICE__ double __drcp_rd(double __a) { return __nv_drcp_rd(__a); }
__DEVICE__ double __drcp_rn(double __a) { return __nv_drcp_rn(__a); }
__DEVICE__ double __drcp_ru(double __a) { return __nv_drcp_ru(__a); }
__DEVICE__ double __drcp_rz(double __a) { return __nv_drcp_rz(__a); }
__DEVICE__ double __dsqrt_rd(double __a) { return __nv_dsqrt_rd(__a); }
__DEVICE__ double __dsqrt_rn(double __a) { return __nv_dsqrt_rn(__a); }
__DEVICE__ double __dsqrt_ru(double __a) { return __nv_dsqrt_ru(__a); }
__DEVICE__ double __dsqrt_rz(double __a) { return __nv_dsqrt_rz(__a); }
__DEVICE__ double __dsub_rd(double __a, double __b) {
  return __nv_dsub_rd(__a, __b);
}
__DEVICE__ double __dsub_rn(double __a, double __b) {
  return __nv_dsub_rn(__a, __b);
}
__DEVICE__ double __dsub_ru(double __a, double __b) {
````
- **L145 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned long long __double2ull_ru(double __a) {`.
  **L145 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned long long __double2ull_ru(double __a) {`。
- **L146 EN**: Returns from the current function with `__nv_double2ull_ru(__a)`.
  **L146 CN**: 以 `__nv_double2ull_ru(__a)` 从当前函数返回。
- **L147 EN**: Closes the current lexical scope or compound statement.
  **L147 CN**: 结束当前词法作用域或复合语句块。
- **L148 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned long long __double2ull_rz(double __a) {`.
  **L148 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned long long __double2ull_rz(double __a) {`。
- **L149 EN**: Returns from the current function with `__nv_double2ull_rz(__a)`.
  **L149 CN**: 以 `__nv_double2ull_rz(__a)` 从当前函数返回。
- **L150 EN**: Closes the current lexical scope or compound statement.
  **L150 CN**: 结束当前词法作用域或复合语句块。
- **L151 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __double_as_longlong(double __a) {`.
  **L151 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __double_as_longlong(double __a) {`。
- **L152 EN**: Returns from the current function with `__nv_double_as_longlong(__a)`.
  **L152 CN**: 以 `__nv_double_as_longlong(__a)` 从当前函数返回。
- **L153 EN**: Closes the current lexical scope or compound statement.
  **L153 CN**: 结束当前词法作用域或复合语句块。
- **L154 EN**: Continues logic associated with callable symbol `__drcp_rd`.
  **L154 CN**: 继续与可调用符号 `__drcp_rd` 相关的逻辑。
- **L155 EN**: Continues logic associated with callable symbol `__drcp_rn`.
  **L155 CN**: 继续与可调用符号 `__drcp_rn` 相关的逻辑。
- **L156 EN**: Continues logic associated with callable symbol `__drcp_ru`.
  **L156 CN**: 继续与可调用符号 `__drcp_ru` 相关的逻辑。
- **L157 EN**: Continues logic associated with callable symbol `__drcp_rz`.
  **L157 CN**: 继续与可调用符号 `__drcp_rz` 相关的逻辑。
- **L158 EN**: Continues logic associated with callable symbol `__dsqrt_rd`.
  **L158 CN**: 继续与可调用符号 `__dsqrt_rd` 相关的逻辑。
- **L159 EN**: Continues logic associated with callable symbol `__dsqrt_rn`.
  **L159 CN**: 继续与可调用符号 `__dsqrt_rn` 相关的逻辑。
- **L160 EN**: Continues logic associated with callable symbol `__dsqrt_ru`.
  **L160 CN**: 继续与可调用符号 `__dsqrt_ru` 相关的逻辑。
- **L161 EN**: Continues logic associated with callable symbol `__dsqrt_rz`.
  **L161 CN**: 继续与可调用符号 `__dsqrt_rz` 相关的逻辑。
- **L162 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __dsub_rd(double __a, double __b) {`.
  **L162 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __dsub_rd(double __a, double __b) {`。
- **L163 EN**: Returns from the current function with `__nv_dsub_rd(__a, __b)`.
  **L163 CN**: 以 `__nv_dsub_rd(__a, __b)` 从当前函数返回。
- **L164 EN**: Closes the current lexical scope or compound statement.
  **L164 CN**: 结束当前词法作用域或复合语句块。
- **L165 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __dsub_rn(double __a, double __b) {`.
  **L165 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __dsub_rn(double __a, double __b) {`。
- **L166 EN**: Returns from the current function with `__nv_dsub_rn(__a, __b)`.
  **L166 CN**: 以 `__nv_dsub_rn(__a, __b)` 从当前函数返回。
- **L167 EN**: Closes the current lexical scope or compound statement.
  **L167 CN**: 结束当前词法作用域或复合语句块。
- **L168 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __dsub_ru(double __a, double __b) {`.
  **L168 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __dsub_ru(double __a, double __b) {`。

### Lines 169-192

````c
  return __nv_dsub_ru(__a, __b);
}
__DEVICE__ double __dsub_rz(double __a, double __b) {
  return __nv_dsub_rz(__a, __b);
}
__DEVICE__ float __exp10f(float __a) { return __nv_fast_exp10f(__a); }
__DEVICE__ float __expf(float __a) { return __nv_fast_expf(__a); }
__DEVICE__ float __fAtomicAdd(float *__p, float __v) {
  return __nvvm_atom_add_gen_f(__p, __v);
}
__DEVICE__ float __fAtomicAdd_block(float *__p, float __v) {
  return __nvvm_atom_cta_add_gen_f(__p, __v);
}
__DEVICE__ float __fAtomicAdd_system(float *__p, float __v) {
  return __nvvm_atom_sys_add_gen_f(__p, __v);
}
__DEVICE__ float __fAtomicExch(float *__p, float __v) {
  return __nv_int_as_float(
      __nvvm_atom_xchg_gen_i((int *)__p, __nv_float_as_int(__v)));
}
__DEVICE__ float __fAtomicExch_block(float *__p, float __v) {
  return __nv_int_as_float(
      __nvvm_atom_cta_xchg_gen_i((int *)__p, __nv_float_as_int(__v)));
}
````
- **L169 EN**: Returns from the current function with `__nv_dsub_ru(__a, __b)`.
  **L169 CN**: 以 `__nv_dsub_ru(__a, __b)` 从当前函数返回。
- **L170 EN**: Closes the current lexical scope or compound statement.
  **L170 CN**: 结束当前词法作用域或复合语句块。
- **L171 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __dsub_rz(double __a, double __b) {`.
  **L171 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __dsub_rz(double __a, double __b) {`。
- **L172 EN**: Returns from the current function with `__nv_dsub_rz(__a, __b)`.
  **L172 CN**: 以 `__nv_dsub_rz(__a, __b)` 从当前函数返回。
- **L173 EN**: Closes the current lexical scope or compound statement.
  **L173 CN**: 结束当前词法作用域或复合语句块。
- **L174 EN**: Continues logic associated with callable symbol `__exp10f`.
  **L174 CN**: 继续与可调用符号 `__exp10f` 相关的逻辑。
- **L175 EN**: Continues logic associated with callable symbol `__expf`.
  **L175 CN**: 继续与可调用符号 `__expf` 相关的逻辑。
- **L176 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fAtomicAdd(float *__p, float __v) {`.
  **L176 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fAtomicAdd(float *__p, float __v) {`。
- **L177 EN**: Returns from the current function with `__nvvm_atom_add_gen_f(__p, __v)`.
  **L177 CN**: 以 `__nvvm_atom_add_gen_f(__p, __v)` 从当前函数返回。
- **L178 EN**: Closes the current lexical scope or compound statement.
  **L178 CN**: 结束当前词法作用域或复合语句块。
- **L179 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fAtomicAdd_block(float *__p, float __v) {`.
  **L179 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fAtomicAdd_block(float *__p, float __v) {`。
- **L180 EN**: Returns from the current function with `__nvvm_atom_cta_add_gen_f(__p, __v)`.
  **L180 CN**: 以 `__nvvm_atom_cta_add_gen_f(__p, __v)` 从当前函数返回。
- **L181 EN**: Closes the current lexical scope or compound statement.
  **L181 CN**: 结束当前词法作用域或复合语句块。
- **L182 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fAtomicAdd_system(float *__p, float __v) {`.
  **L182 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fAtomicAdd_system(float *__p, float __v) {`。
- **L183 EN**: Returns from the current function with `__nvvm_atom_sys_add_gen_f(__p, __v)`.
  **L183 CN**: 以 `__nvvm_atom_sys_add_gen_f(__p, __v)` 从当前函数返回。
- **L184 EN**: Closes the current lexical scope or compound statement.
  **L184 CN**: 结束当前词法作用域或复合语句块。
- **L185 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fAtomicExch(float *__p, float __v) {`.
  **L185 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fAtomicExch(float *__p, float __v) {`。
- **L186 EN**: Returns from the current function with `__nv_int_as_float(`.
  **L186 CN**: 以 `__nv_int_as_float(` 从当前函数返回。
- **L187 EN**: Executes a call or declaration centered on `__nvvm_atom_xchg_gen_i`.
  **L187 CN**: 执行以 `__nvvm_atom_xchg_gen_i` 为核心的调用或声明。
- **L188 EN**: Closes the current lexical scope or compound statement.
  **L188 CN**: 结束当前词法作用域或复合语句块。
- **L189 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fAtomicExch_block(float *__p, float __v) {`.
  **L189 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fAtomicExch_block(float *__p, float __v) {`。
- **L190 EN**: Returns from the current function with `__nv_int_as_float(`.
  **L190 CN**: 以 `__nv_int_as_float(` 从当前函数返回。
- **L191 EN**: Executes a call or declaration centered on `__nvvm_atom_cta_xchg_gen_i`.
  **L191 CN**: 执行以 `__nvvm_atom_cta_xchg_gen_i` 为核心的调用或声明。
- **L192 EN**: Closes the current lexical scope or compound statement.
  **L192 CN**: 结束当前词法作用域或复合语句块。

### Lines 193-216

````c
__DEVICE__ float __fAtomicExch_system(float *__p, float __v) {
  return __nv_int_as_float(
      __nvvm_atom_sys_xchg_gen_i((int *)__p, __nv_float_as_int(__v)));
}
__DEVICE__ float __fadd_rd(float __a, float __b) {
  return __nv_fadd_rd(__a, __b);
}
__DEVICE__ float __fadd_rn(float __a, float __b) {
  return __nv_fadd_rn(__a, __b);
}
__DEVICE__ float __fadd_ru(float __a, float __b) {
  return __nv_fadd_ru(__a, __b);
}
__DEVICE__ float __fadd_rz(float __a, float __b) {
  return __nv_fadd_rz(__a, __b);
}
__DEVICE__ float __fdiv_rd(float __a, float __b) {
  return __nv_fdiv_rd(__a, __b);
}
__DEVICE__ float __fdiv_rn(float __a, float __b) {
  return __nv_fdiv_rn(__a, __b);
}
__DEVICE__ float __fdiv_ru(float __a, float __b) {
  return __nv_fdiv_ru(__a, __b);
````
- **L193 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fAtomicExch_system(float *__p, float __v) {`.
  **L193 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fAtomicExch_system(float *__p, float __v) {`。
- **L194 EN**: Returns from the current function with `__nv_int_as_float(`.
  **L194 CN**: 以 `__nv_int_as_float(` 从当前函数返回。
- **L195 EN**: Executes a call or declaration centered on `__nvvm_atom_sys_xchg_gen_i`.
  **L195 CN**: 执行以 `__nvvm_atom_sys_xchg_gen_i` 为核心的调用或声明。
- **L196 EN**: Closes the current lexical scope or compound statement.
  **L196 CN**: 结束当前词法作用域或复合语句块。
- **L197 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fadd_rd(float __a, float __b) {`.
  **L197 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fadd_rd(float __a, float __b) {`。
- **L198 EN**: Returns from the current function with `__nv_fadd_rd(__a, __b)`.
  **L198 CN**: 以 `__nv_fadd_rd(__a, __b)` 从当前函数返回。
- **L199 EN**: Closes the current lexical scope or compound statement.
  **L199 CN**: 结束当前词法作用域或复合语句块。
- **L200 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fadd_rn(float __a, float __b) {`.
  **L200 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fadd_rn(float __a, float __b) {`。
- **L201 EN**: Returns from the current function with `__nv_fadd_rn(__a, __b)`.
  **L201 CN**: 以 `__nv_fadd_rn(__a, __b)` 从当前函数返回。
- **L202 EN**: Closes the current lexical scope or compound statement.
  **L202 CN**: 结束当前词法作用域或复合语句块。
- **L203 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fadd_ru(float __a, float __b) {`.
  **L203 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fadd_ru(float __a, float __b) {`。
- **L204 EN**: Returns from the current function with `__nv_fadd_ru(__a, __b)`.
  **L204 CN**: 以 `__nv_fadd_ru(__a, __b)` 从当前函数返回。
- **L205 EN**: Closes the current lexical scope or compound statement.
  **L205 CN**: 结束当前词法作用域或复合语句块。
- **L206 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fadd_rz(float __a, float __b) {`.
  **L206 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fadd_rz(float __a, float __b) {`。
- **L207 EN**: Returns from the current function with `__nv_fadd_rz(__a, __b)`.
  **L207 CN**: 以 `__nv_fadd_rz(__a, __b)` 从当前函数返回。
- **L208 EN**: Closes the current lexical scope or compound statement.
  **L208 CN**: 结束当前词法作用域或复合语句块。
- **L209 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fdiv_rd(float __a, float __b) {`.
  **L209 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fdiv_rd(float __a, float __b) {`。
- **L210 EN**: Returns from the current function with `__nv_fdiv_rd(__a, __b)`.
  **L210 CN**: 以 `__nv_fdiv_rd(__a, __b)` 从当前函数返回。
- **L211 EN**: Closes the current lexical scope or compound statement.
  **L211 CN**: 结束当前词法作用域或复合语句块。
- **L212 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fdiv_rn(float __a, float __b) {`.
  **L212 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fdiv_rn(float __a, float __b) {`。
- **L213 EN**: Returns from the current function with `__nv_fdiv_rn(__a, __b)`.
  **L213 CN**: 以 `__nv_fdiv_rn(__a, __b)` 从当前函数返回。
- **L214 EN**: Closes the current lexical scope or compound statement.
  **L214 CN**: 结束当前词法作用域或复合语句块。
- **L215 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fdiv_ru(float __a, float __b) {`.
  **L215 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fdiv_ru(float __a, float __b) {`。
- **L216 EN**: Returns from the current function with `__nv_fdiv_ru(__a, __b)`.
  **L216 CN**: 以 `__nv_fdiv_ru(__a, __b)` 从当前函数返回。

### Lines 217-240

````c
}
__DEVICE__ float __fdiv_rz(float __a, float __b) {
  return __nv_fdiv_rz(__a, __b);
}
__DEVICE__ float __fdividef(float __a, float __b) {
  return __nv_fast_fdividef(__a, __b);
}
__DEVICE__ int __ffs(int __a) { return __nv_ffs(__a); }
__DEVICE__ int __ffsll(long long __a) { return __nv_ffsll(__a); }
__DEVICE__ int __finite(double __a) { return __nv_isfinited(__a); }
__DEVICE__ int __finitef(float __a) { return __nv_finitef(__a); }
#ifdef _MSC_VER
__DEVICE__ int __finitel(long double __a);
#endif
__DEVICE__ int __float2int_rd(float __a) { return __nv_float2int_rd(__a); }
__DEVICE__ int __float2int_rn(float __a) { return __nv_float2int_rn(__a); }
__DEVICE__ int __float2int_ru(float __a) { return __nv_float2int_ru(__a); }
__DEVICE__ int __float2int_rz(float __a) { return __nv_float2int_rz(__a); }
__DEVICE__ long long __float2ll_rd(float __a) { return __nv_float2ll_rd(__a); }
__DEVICE__ long long __float2ll_rn(float __a) { return __nv_float2ll_rn(__a); }
__DEVICE__ long long __float2ll_ru(float __a) { return __nv_float2ll_ru(__a); }
__DEVICE__ long long __float2ll_rz(float __a) { return __nv_float2ll_rz(__a); }
__DEVICE__ unsigned int __float2uint_rd(float __a) {
  return __nv_float2uint_rd(__a);
````
- **L217 EN**: Closes the current lexical scope or compound statement.
  **L217 CN**: 结束当前词法作用域或复合语句块。
- **L218 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fdiv_rz(float __a, float __b) {`.
  **L218 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fdiv_rz(float __a, float __b) {`。
- **L219 EN**: Returns from the current function with `__nv_fdiv_rz(__a, __b)`.
  **L219 CN**: 以 `__nv_fdiv_rz(__a, __b)` 从当前函数返回。
- **L220 EN**: Closes the current lexical scope or compound statement.
  **L220 CN**: 结束当前词法作用域或复合语句块。
- **L221 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fdividef(float __a, float __b) {`.
  **L221 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fdividef(float __a, float __b) {`。
- **L222 EN**: Returns from the current function with `__nv_fast_fdividef(__a, __b)`.
  **L222 CN**: 以 `__nv_fast_fdividef(__a, __b)` 从当前函数返回。
- **L223 EN**: Closes the current lexical scope or compound statement.
  **L223 CN**: 结束当前词法作用域或复合语句块。
- **L224 EN**: Continues logic associated with callable symbol `__ffs`.
  **L224 CN**: 继续与可调用符号 `__ffs` 相关的逻辑。
- **L225 EN**: Continues logic associated with callable symbol `__ffsll`.
  **L225 CN**: 继续与可调用符号 `__ffsll` 相关的逻辑。
- **L226 EN**: Continues logic associated with callable symbol `__finite`.
  **L226 CN**: 继续与可调用符号 `__finite` 相关的逻辑。
- **L227 EN**: Continues logic associated with callable symbol `__finitef`.
  **L227 CN**: 继续与可调用符号 `__finitef` 相关的逻辑。
- **L228 EN**: Starts a preprocessor conditional block: `#ifdef _MSC_VER`.
  **L228 CN**: 开始一个预处理条件块：`#ifdef _MSC_VER`。
- **L229 EN**: Executes a call or declaration centered on `__finitel`.
  **L229 CN**: 执行以 `__finitel` 为核心的调用或声明。
- **L230 EN**: Closes the current preprocessor conditional block.
  **L230 CN**: 结束当前预处理条件块。
- **L231 EN**: Continues logic associated with callable symbol `__float2int_rd`.
  **L231 CN**: 继续与可调用符号 `__float2int_rd` 相关的逻辑。
- **L232 EN**: Continues logic associated with callable symbol `__float2int_rn`.
  **L232 CN**: 继续与可调用符号 `__float2int_rn` 相关的逻辑。
- **L233 EN**: Continues logic associated with callable symbol `__float2int_ru`.
  **L233 CN**: 继续与可调用符号 `__float2int_ru` 相关的逻辑。
- **L234 EN**: Continues logic associated with callable symbol `__float2int_rz`.
  **L234 CN**: 继续与可调用符号 `__float2int_rz` 相关的逻辑。
- **L235 EN**: Continues logic associated with callable symbol `__float2ll_rd`.
  **L235 CN**: 继续与可调用符号 `__float2ll_rd` 相关的逻辑。
- **L236 EN**: Continues logic associated with callable symbol `__float2ll_rn`.
  **L236 CN**: 继续与可调用符号 `__float2ll_rn` 相关的逻辑。
- **L237 EN**: Continues logic associated with callable symbol `__float2ll_ru`.
  **L237 CN**: 继续与可调用符号 `__float2ll_ru` 相关的逻辑。
- **L238 EN**: Continues logic associated with callable symbol `__float2ll_rz`.
  **L238 CN**: 继续与可调用符号 `__float2ll_rz` 相关的逻辑。
- **L239 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __float2uint_rd(float __a) {`.
  **L239 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __float2uint_rd(float __a) {`。
- **L240 EN**: Returns from the current function with `__nv_float2uint_rd(__a)`.
  **L240 CN**: 以 `__nv_float2uint_rd(__a)` 从当前函数返回。

### Lines 241-264

````c
}
__DEVICE__ unsigned int __float2uint_rn(float __a) {
  return __nv_float2uint_rn(__a);
}
__DEVICE__ unsigned int __float2uint_ru(float __a) {
  return __nv_float2uint_ru(__a);
}
__DEVICE__ unsigned int __float2uint_rz(float __a) {
  return __nv_float2uint_rz(__a);
}
__DEVICE__ unsigned long long __float2ull_rd(float __a) {
  return __nv_float2ull_rd(__a);
}
__DEVICE__ unsigned long long __float2ull_rn(float __a) {
  return __nv_float2ull_rn(__a);
}
__DEVICE__ unsigned long long __float2ull_ru(float __a) {
  return __nv_float2ull_ru(__a);
}
__DEVICE__ unsigned long long __float2ull_rz(float __a) {
  return __nv_float2ull_rz(__a);
}
__DEVICE__ int __float_as_int(float __a) { return __nv_float_as_int(__a); }
__DEVICE__ unsigned int __float_as_uint(float __a) {
````
- **L241 EN**: Closes the current lexical scope or compound statement.
  **L241 CN**: 结束当前词法作用域或复合语句块。
- **L242 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __float2uint_rn(float __a) {`.
  **L242 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __float2uint_rn(float __a) {`。
- **L243 EN**: Returns from the current function with `__nv_float2uint_rn(__a)`.
  **L243 CN**: 以 `__nv_float2uint_rn(__a)` 从当前函数返回。
- **L244 EN**: Closes the current lexical scope or compound statement.
  **L244 CN**: 结束当前词法作用域或复合语句块。
- **L245 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __float2uint_ru(float __a) {`.
  **L245 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __float2uint_ru(float __a) {`。
- **L246 EN**: Returns from the current function with `__nv_float2uint_ru(__a)`.
  **L246 CN**: 以 `__nv_float2uint_ru(__a)` 从当前函数返回。
- **L247 EN**: Closes the current lexical scope or compound statement.
  **L247 CN**: 结束当前词法作用域或复合语句块。
- **L248 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __float2uint_rz(float __a) {`.
  **L248 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __float2uint_rz(float __a) {`。
- **L249 EN**: Returns from the current function with `__nv_float2uint_rz(__a)`.
  **L249 CN**: 以 `__nv_float2uint_rz(__a)` 从当前函数返回。
- **L250 EN**: Closes the current lexical scope or compound statement.
  **L250 CN**: 结束当前词法作用域或复合语句块。
- **L251 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned long long __float2ull_rd(float __a) {`.
  **L251 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned long long __float2ull_rd(float __a) {`。
- **L252 EN**: Returns from the current function with `__nv_float2ull_rd(__a)`.
  **L252 CN**: 以 `__nv_float2ull_rd(__a)` 从当前函数返回。
- **L253 EN**: Closes the current lexical scope or compound statement.
  **L253 CN**: 结束当前词法作用域或复合语句块。
- **L254 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned long long __float2ull_rn(float __a) {`.
  **L254 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned long long __float2ull_rn(float __a) {`。
- **L255 EN**: Returns from the current function with `__nv_float2ull_rn(__a)`.
  **L255 CN**: 以 `__nv_float2ull_rn(__a)` 从当前函数返回。
- **L256 EN**: Closes the current lexical scope or compound statement.
  **L256 CN**: 结束当前词法作用域或复合语句块。
- **L257 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned long long __float2ull_ru(float __a) {`.
  **L257 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned long long __float2ull_ru(float __a) {`。
- **L258 EN**: Returns from the current function with `__nv_float2ull_ru(__a)`.
  **L258 CN**: 以 `__nv_float2ull_ru(__a)` 从当前函数返回。
- **L259 EN**: Closes the current lexical scope or compound statement.
  **L259 CN**: 结束当前词法作用域或复合语句块。
- **L260 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned long long __float2ull_rz(float __a) {`.
  **L260 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned long long __float2ull_rz(float __a) {`。
- **L261 EN**: Returns from the current function with `__nv_float2ull_rz(__a)`.
  **L261 CN**: 以 `__nv_float2ull_rz(__a)` 从当前函数返回。
- **L262 EN**: Closes the current lexical scope or compound statement.
  **L262 CN**: 结束当前词法作用域或复合语句块。
- **L263 EN**: Continues logic associated with callable symbol `__float_as_int`.
  **L263 CN**: 继续与可调用符号 `__float_as_int` 相关的逻辑。
- **L264 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __float_as_uint(float __a) {`.
  **L264 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __float_as_uint(float __a) {`。

### Lines 265-288

````c
  return __nv_float_as_uint(__a);
}
__DEVICE__ double __fma_rd(double __a, double __b, double __c) {
  return __nv_fma_rd(__a, __b, __c);
}
__DEVICE__ double __fma_rn(double __a, double __b, double __c) {
  return __nv_fma_rn(__a, __b, __c);
}
__DEVICE__ double __fma_ru(double __a, double __b, double __c) {
  return __nv_fma_ru(__a, __b, __c);
}
__DEVICE__ double __fma_rz(double __a, double __b, double __c) {
  return __nv_fma_rz(__a, __b, __c);
}
__DEVICE__ float __fmaf_ieee_rd(float __a, float __b, float __c) {
  return __nv_fmaf_ieee_rd(__a, __b, __c);
}
__DEVICE__ float __fmaf_ieee_rn(float __a, float __b, float __c) {
  return __nv_fmaf_ieee_rn(__a, __b, __c);
}
__DEVICE__ float __fmaf_ieee_ru(float __a, float __b, float __c) {
  return __nv_fmaf_ieee_ru(__a, __b, __c);
}
__DEVICE__ float __fmaf_ieee_rz(float __a, float __b, float __c) {
````
- **L265 EN**: Returns from the current function with `__nv_float_as_uint(__a)`.
  **L265 CN**: 以 `__nv_float_as_uint(__a)` 从当前函数返回。
- **L266 EN**: Closes the current lexical scope or compound statement.
  **L266 CN**: 结束当前词法作用域或复合语句块。
- **L267 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __fma_rd(double __a, double __b, double __c) {`.
  **L267 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __fma_rd(double __a, double __b, double __c) {`。
- **L268 EN**: Returns from the current function with `__nv_fma_rd(__a, __b, __c)`.
  **L268 CN**: 以 `__nv_fma_rd(__a, __b, __c)` 从当前函数返回。
- **L269 EN**: Closes the current lexical scope or compound statement.
  **L269 CN**: 结束当前词法作用域或复合语句块。
- **L270 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __fma_rn(double __a, double __b, double __c) {`.
  **L270 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __fma_rn(double __a, double __b, double __c) {`。
- **L271 EN**: Returns from the current function with `__nv_fma_rn(__a, __b, __c)`.
  **L271 CN**: 以 `__nv_fma_rn(__a, __b, __c)` 从当前函数返回。
- **L272 EN**: Closes the current lexical scope or compound statement.
  **L272 CN**: 结束当前词法作用域或复合语句块。
- **L273 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __fma_ru(double __a, double __b, double __c) {`.
  **L273 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __fma_ru(double __a, double __b, double __c) {`。
- **L274 EN**: Returns from the current function with `__nv_fma_ru(__a, __b, __c)`.
  **L274 CN**: 以 `__nv_fma_ru(__a, __b, __c)` 从当前函数返回。
- **L275 EN**: Closes the current lexical scope or compound statement.
  **L275 CN**: 结束当前词法作用域或复合语句块。
- **L276 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __fma_rz(double __a, double __b, double __c) {`.
  **L276 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __fma_rz(double __a, double __b, double __c) {`。
- **L277 EN**: Returns from the current function with `__nv_fma_rz(__a, __b, __c)`.
  **L277 CN**: 以 `__nv_fma_rz(__a, __b, __c)` 从当前函数返回。
- **L278 EN**: Closes the current lexical scope or compound statement.
  **L278 CN**: 结束当前词法作用域或复合语句块。
- **L279 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fmaf_ieee_rd(float __a, float __b, float __c) {`.
  **L279 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fmaf_ieee_rd(float __a, float __b, float __c) {`。
- **L280 EN**: Returns from the current function with `__nv_fmaf_ieee_rd(__a, __b, __c)`.
  **L280 CN**: 以 `__nv_fmaf_ieee_rd(__a, __b, __c)` 从当前函数返回。
- **L281 EN**: Closes the current lexical scope or compound statement.
  **L281 CN**: 结束当前词法作用域或复合语句块。
- **L282 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fmaf_ieee_rn(float __a, float __b, float __c) {`.
  **L282 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fmaf_ieee_rn(float __a, float __b, float __c) {`。
- **L283 EN**: Returns from the current function with `__nv_fmaf_ieee_rn(__a, __b, __c)`.
  **L283 CN**: 以 `__nv_fmaf_ieee_rn(__a, __b, __c)` 从当前函数返回。
- **L284 EN**: Closes the current lexical scope or compound statement.
  **L284 CN**: 结束当前词法作用域或复合语句块。
- **L285 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fmaf_ieee_ru(float __a, float __b, float __c) {`.
  **L285 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fmaf_ieee_ru(float __a, float __b, float __c) {`。
- **L286 EN**: Returns from the current function with `__nv_fmaf_ieee_ru(__a, __b, __c)`.
  **L286 CN**: 以 `__nv_fmaf_ieee_ru(__a, __b, __c)` 从当前函数返回。
- **L287 EN**: Closes the current lexical scope or compound statement.
  **L287 CN**: 结束当前词法作用域或复合语句块。
- **L288 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fmaf_ieee_rz(float __a, float __b, float __c) {`.
  **L288 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fmaf_ieee_rz(float __a, float __b, float __c) {`。

### Lines 289-312

````c
  return __nv_fmaf_ieee_rz(__a, __b, __c);
}
__DEVICE__ float __fmaf_rd(float __a, float __b, float __c) {
  return __nv_fmaf_rd(__a, __b, __c);
}
__DEVICE__ float __fmaf_rn(float __a, float __b, float __c) {
  return __nv_fmaf_rn(__a, __b, __c);
}
__DEVICE__ float __fmaf_ru(float __a, float __b, float __c) {
  return __nv_fmaf_ru(__a, __b, __c);
}
__DEVICE__ float __fmaf_rz(float __a, float __b, float __c) {
  return __nv_fmaf_rz(__a, __b, __c);
}
__DEVICE__ float __fmul_rd(float __a, float __b) {
  return __nv_fmul_rd(__a, __b);
}
__DEVICE__ float __fmul_rn(float __a, float __b) {
  return __nv_fmul_rn(__a, __b);
}
__DEVICE__ float __fmul_ru(float __a, float __b) {
  return __nv_fmul_ru(__a, __b);
}
__DEVICE__ float __fmul_rz(float __a, float __b) {
````
- **L289 EN**: Returns from the current function with `__nv_fmaf_ieee_rz(__a, __b, __c)`.
  **L289 CN**: 以 `__nv_fmaf_ieee_rz(__a, __b, __c)` 从当前函数返回。
- **L290 EN**: Closes the current lexical scope or compound statement.
  **L290 CN**: 结束当前词法作用域或复合语句块。
- **L291 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fmaf_rd(float __a, float __b, float __c) {`.
  **L291 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fmaf_rd(float __a, float __b, float __c) {`。
- **L292 EN**: Returns from the current function with `__nv_fmaf_rd(__a, __b, __c)`.
  **L292 CN**: 以 `__nv_fmaf_rd(__a, __b, __c)` 从当前函数返回。
- **L293 EN**: Closes the current lexical scope or compound statement.
  **L293 CN**: 结束当前词法作用域或复合语句块。
- **L294 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fmaf_rn(float __a, float __b, float __c) {`.
  **L294 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fmaf_rn(float __a, float __b, float __c) {`。
- **L295 EN**: Returns from the current function with `__nv_fmaf_rn(__a, __b, __c)`.
  **L295 CN**: 以 `__nv_fmaf_rn(__a, __b, __c)` 从当前函数返回。
- **L296 EN**: Closes the current lexical scope or compound statement.
  **L296 CN**: 结束当前词法作用域或复合语句块。
- **L297 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fmaf_ru(float __a, float __b, float __c) {`.
  **L297 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fmaf_ru(float __a, float __b, float __c) {`。
- **L298 EN**: Returns from the current function with `__nv_fmaf_ru(__a, __b, __c)`.
  **L298 CN**: 以 `__nv_fmaf_ru(__a, __b, __c)` 从当前函数返回。
- **L299 EN**: Closes the current lexical scope or compound statement.
  **L299 CN**: 结束当前词法作用域或复合语句块。
- **L300 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fmaf_rz(float __a, float __b, float __c) {`.
  **L300 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fmaf_rz(float __a, float __b, float __c) {`。
- **L301 EN**: Returns from the current function with `__nv_fmaf_rz(__a, __b, __c)`.
  **L301 CN**: 以 `__nv_fmaf_rz(__a, __b, __c)` 从当前函数返回。
- **L302 EN**: Closes the current lexical scope or compound statement.
  **L302 CN**: 结束当前词法作用域或复合语句块。
- **L303 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fmul_rd(float __a, float __b) {`.
  **L303 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fmul_rd(float __a, float __b) {`。
- **L304 EN**: Returns from the current function with `__nv_fmul_rd(__a, __b)`.
  **L304 CN**: 以 `__nv_fmul_rd(__a, __b)` 从当前函数返回。
- **L305 EN**: Closes the current lexical scope or compound statement.
  **L305 CN**: 结束当前词法作用域或复合语句块。
- **L306 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fmul_rn(float __a, float __b) {`.
  **L306 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fmul_rn(float __a, float __b) {`。
- **L307 EN**: Returns from the current function with `__nv_fmul_rn(__a, __b)`.
  **L307 CN**: 以 `__nv_fmul_rn(__a, __b)` 从当前函数返回。
- **L308 EN**: Closes the current lexical scope or compound statement.
  **L308 CN**: 结束当前词法作用域或复合语句块。
- **L309 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fmul_ru(float __a, float __b) {`.
  **L309 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fmul_ru(float __a, float __b) {`。
- **L310 EN**: Returns from the current function with `__nv_fmul_ru(__a, __b)`.
  **L310 CN**: 以 `__nv_fmul_ru(__a, __b)` 从当前函数返回。
- **L311 EN**: Closes the current lexical scope or compound statement.
  **L311 CN**: 结束当前词法作用域或复合语句块。
- **L312 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fmul_rz(float __a, float __b) {`.
  **L312 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fmul_rz(float __a, float __b) {`。

### Lines 313-336

````c
  return __nv_fmul_rz(__a, __b);
}
__DEVICE__ float __frcp_rd(float __a) { return __nv_frcp_rd(__a); }
__DEVICE__ float __frcp_rn(float __a) { return __nv_frcp_rn(__a); }
__DEVICE__ float __frcp_ru(float __a) { return __nv_frcp_ru(__a); }
__DEVICE__ float __frcp_rz(float __a) { return __nv_frcp_rz(__a); }
__DEVICE__ float __frsqrt_rn(float __a) { return __nv_frsqrt_rn(__a); }
__DEVICE__ float __fsqrt_rd(float __a) { return __nv_fsqrt_rd(__a); }
__DEVICE__ float __fsqrt_rn(float __a) { return __nv_fsqrt_rn(__a); }
__DEVICE__ float __fsqrt_ru(float __a) { return __nv_fsqrt_ru(__a); }
__DEVICE__ float __fsqrt_rz(float __a) { return __nv_fsqrt_rz(__a); }
__DEVICE__ float __fsub_rd(float __a, float __b) {
  return __nv_fsub_rd(__a, __b);
}
__DEVICE__ float __fsub_rn(float __a, float __b) {
  return __nv_fsub_rn(__a, __b);
}
__DEVICE__ float __fsub_ru(float __a, float __b) {
  return __nv_fsub_ru(__a, __b);
}
__DEVICE__ float __fsub_rz(float __a, float __b) {
  return __nv_fsub_rz(__a, __b);
}
__DEVICE__ int __hadd(int __a, int __b) { return __nv_hadd(__a, __b); }
````
- **L313 EN**: Returns from the current function with `__nv_fmul_rz(__a, __b)`.
  **L313 CN**: 以 `__nv_fmul_rz(__a, __b)` 从当前函数返回。
- **L314 EN**: Closes the current lexical scope or compound statement.
  **L314 CN**: 结束当前词法作用域或复合语句块。
- **L315 EN**: Continues logic associated with callable symbol `__frcp_rd`.
  **L315 CN**: 继续与可调用符号 `__frcp_rd` 相关的逻辑。
- **L316 EN**: Continues logic associated with callable symbol `__frcp_rn`.
  **L316 CN**: 继续与可调用符号 `__frcp_rn` 相关的逻辑。
- **L317 EN**: Continues logic associated with callable symbol `__frcp_ru`.
  **L317 CN**: 继续与可调用符号 `__frcp_ru` 相关的逻辑。
- **L318 EN**: Continues logic associated with callable symbol `__frcp_rz`.
  **L318 CN**: 继续与可调用符号 `__frcp_rz` 相关的逻辑。
- **L319 EN**: Continues logic associated with callable symbol `__frsqrt_rn`.
  **L319 CN**: 继续与可调用符号 `__frsqrt_rn` 相关的逻辑。
- **L320 EN**: Continues logic associated with callable symbol `__fsqrt_rd`.
  **L320 CN**: 继续与可调用符号 `__fsqrt_rd` 相关的逻辑。
- **L321 EN**: Continues logic associated with callable symbol `__fsqrt_rn`.
  **L321 CN**: 继续与可调用符号 `__fsqrt_rn` 相关的逻辑。
- **L322 EN**: Continues logic associated with callable symbol `__fsqrt_ru`.
  **L322 CN**: 继续与可调用符号 `__fsqrt_ru` 相关的逻辑。
- **L323 EN**: Continues logic associated with callable symbol `__fsqrt_rz`.
  **L323 CN**: 继续与可调用符号 `__fsqrt_rz` 相关的逻辑。
- **L324 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fsub_rd(float __a, float __b) {`.
  **L324 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fsub_rd(float __a, float __b) {`。
- **L325 EN**: Returns from the current function with `__nv_fsub_rd(__a, __b)`.
  **L325 CN**: 以 `__nv_fsub_rd(__a, __b)` 从当前函数返回。
- **L326 EN**: Closes the current lexical scope or compound statement.
  **L326 CN**: 结束当前词法作用域或复合语句块。
- **L327 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fsub_rn(float __a, float __b) {`.
  **L327 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fsub_rn(float __a, float __b) {`。
- **L328 EN**: Returns from the current function with `__nv_fsub_rn(__a, __b)`.
  **L328 CN**: 以 `__nv_fsub_rn(__a, __b)` 从当前函数返回。
- **L329 EN**: Closes the current lexical scope or compound statement.
  **L329 CN**: 结束当前词法作用域或复合语句块。
- **L330 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fsub_ru(float __a, float __b) {`.
  **L330 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fsub_ru(float __a, float __b) {`。
- **L331 EN**: Returns from the current function with `__nv_fsub_ru(__a, __b)`.
  **L331 CN**: 以 `__nv_fsub_ru(__a, __b)` 从当前函数返回。
- **L332 EN**: Closes the current lexical scope or compound statement.
  **L332 CN**: 结束当前词法作用域或复合语句块。
- **L333 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __fsub_rz(float __a, float __b) {`.
  **L333 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __fsub_rz(float __a, float __b) {`。
- **L334 EN**: Returns from the current function with `__nv_fsub_rz(__a, __b)`.
  **L334 CN**: 以 `__nv_fsub_rz(__a, __b)` 从当前函数返回。
- **L335 EN**: Closes the current lexical scope or compound statement.
  **L335 CN**: 结束当前词法作用域或复合语句块。
- **L336 EN**: Continues logic associated with callable symbol `__hadd`.
  **L336 CN**: 继续与可调用符号 `__hadd` 相关的逻辑。

### Lines 337-360

````c
__DEVICE__ double __hiloint2double(int __a, int __b) {
  return __nv_hiloint2double(__a, __b);
}
__DEVICE__ int __iAtomicAdd(int *__p, int __v) {
  return __nvvm_atom_add_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicAdd_block(int *__p, int __v) {
  return __nvvm_atom_cta_add_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicAdd_system(int *__p, int __v) {
  return __nvvm_atom_sys_add_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicAnd(int *__p, int __v) {
  return __nvvm_atom_and_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicAnd_block(int *__p, int __v) {
  return __nvvm_atom_cta_and_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicAnd_system(int *__p, int __v) {
  return __nvvm_atom_sys_and_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicCAS(int *__p, int __cmp, int __v) {
  return __nvvm_atom_cas_gen_i(__p, __cmp, __v);
}
````
- **L337 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __hiloint2double(int __a, int __b) {`.
  **L337 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __hiloint2double(int __a, int __b) {`。
- **L338 EN**: Returns from the current function with `__nv_hiloint2double(__a, __b)`.
  **L338 CN**: 以 `__nv_hiloint2double(__a, __b)` 从当前函数返回。
- **L339 EN**: Closes the current lexical scope or compound statement.
  **L339 CN**: 结束当前词法作用域或复合语句块。
- **L340 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicAdd(int *__p, int __v) {`.
  **L340 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicAdd(int *__p, int __v) {`。
- **L341 EN**: Returns from the current function with `__nvvm_atom_add_gen_i(__p, __v)`.
  **L341 CN**: 以 `__nvvm_atom_add_gen_i(__p, __v)` 从当前函数返回。
- **L342 EN**: Closes the current lexical scope or compound statement.
  **L342 CN**: 结束当前词法作用域或复合语句块。
- **L343 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicAdd_block(int *__p, int __v) {`.
  **L343 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicAdd_block(int *__p, int __v) {`。
- **L344 EN**: Returns from the current function with `__nvvm_atom_cta_add_gen_i(__p, __v)`.
  **L344 CN**: 以 `__nvvm_atom_cta_add_gen_i(__p, __v)` 从当前函数返回。
- **L345 EN**: Closes the current lexical scope or compound statement.
  **L345 CN**: 结束当前词法作用域或复合语句块。
- **L346 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicAdd_system(int *__p, int __v) {`.
  **L346 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicAdd_system(int *__p, int __v) {`。
- **L347 EN**: Returns from the current function with `__nvvm_atom_sys_add_gen_i(__p, __v)`.
  **L347 CN**: 以 `__nvvm_atom_sys_add_gen_i(__p, __v)` 从当前函数返回。
- **L348 EN**: Closes the current lexical scope or compound statement.
  **L348 CN**: 结束当前词法作用域或复合语句块。
- **L349 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicAnd(int *__p, int __v) {`.
  **L349 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicAnd(int *__p, int __v) {`。
- **L350 EN**: Returns from the current function with `__nvvm_atom_and_gen_i(__p, __v)`.
  **L350 CN**: 以 `__nvvm_atom_and_gen_i(__p, __v)` 从当前函数返回。
- **L351 EN**: Closes the current lexical scope or compound statement.
  **L351 CN**: 结束当前词法作用域或复合语句块。
- **L352 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicAnd_block(int *__p, int __v) {`.
  **L352 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicAnd_block(int *__p, int __v) {`。
- **L353 EN**: Returns from the current function with `__nvvm_atom_cta_and_gen_i(__p, __v)`.
  **L353 CN**: 以 `__nvvm_atom_cta_and_gen_i(__p, __v)` 从当前函数返回。
- **L354 EN**: Closes the current lexical scope or compound statement.
  **L354 CN**: 结束当前词法作用域或复合语句块。
- **L355 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicAnd_system(int *__p, int __v) {`.
  **L355 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicAnd_system(int *__p, int __v) {`。
- **L356 EN**: Returns from the current function with `__nvvm_atom_sys_and_gen_i(__p, __v)`.
  **L356 CN**: 以 `__nvvm_atom_sys_and_gen_i(__p, __v)` 从当前函数返回。
- **L357 EN**: Closes the current lexical scope or compound statement.
  **L357 CN**: 结束当前词法作用域或复合语句块。
- **L358 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicCAS(int *__p, int __cmp, int __v) {`.
  **L358 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicCAS(int *__p, int __cmp, int __v) {`。
- **L359 EN**: Returns from the current function with `__nvvm_atom_cas_gen_i(__p, __cmp, __v)`.
  **L359 CN**: 以 `__nvvm_atom_cas_gen_i(__p, __cmp, __v)` 从当前函数返回。
- **L360 EN**: Closes the current lexical scope or compound statement.
  **L360 CN**: 结束当前词法作用域或复合语句块。

### Lines 361-384

````c
__DEVICE__ int __iAtomicCAS_block(int *__p, int __cmp, int __v) {
  return __nvvm_atom_cta_cas_gen_i(__p, __cmp, __v);
}
__DEVICE__ int __iAtomicCAS_system(int *__p, int __cmp, int __v) {
  return __nvvm_atom_sys_cas_gen_i(__p, __cmp, __v);
}
__DEVICE__ int __iAtomicExch(int *__p, int __v) {
  return __nvvm_atom_xchg_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicExch_block(int *__p, int __v) {
  return __nvvm_atom_cta_xchg_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicExch_system(int *__p, int __v) {
  return __nvvm_atom_sys_xchg_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicMax(int *__p, int __v) {
  return __nvvm_atom_max_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicMax_block(int *__p, int __v) {
  return __nvvm_atom_cta_max_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicMax_system(int *__p, int __v) {
  return __nvvm_atom_sys_max_gen_i(__p, __v);
}
````
- **L361 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicCAS_block(int *__p, int __cmp, int __v) {`.
  **L361 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicCAS_block(int *__p, int __cmp, int __v) {`。
- **L362 EN**: Returns from the current function with `__nvvm_atom_cta_cas_gen_i(__p, __cmp, __v)`.
  **L362 CN**: 以 `__nvvm_atom_cta_cas_gen_i(__p, __cmp, __v)` 从当前函数返回。
- **L363 EN**: Closes the current lexical scope or compound statement.
  **L363 CN**: 结束当前词法作用域或复合语句块。
- **L364 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicCAS_system(int *__p, int __cmp, int __v) {`.
  **L364 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicCAS_system(int *__p, int __cmp, int __v) {`。
- **L365 EN**: Returns from the current function with `__nvvm_atom_sys_cas_gen_i(__p, __cmp, __v)`.
  **L365 CN**: 以 `__nvvm_atom_sys_cas_gen_i(__p, __cmp, __v)` 从当前函数返回。
- **L366 EN**: Closes the current lexical scope or compound statement.
  **L366 CN**: 结束当前词法作用域或复合语句块。
- **L367 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicExch(int *__p, int __v) {`.
  **L367 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicExch(int *__p, int __v) {`。
- **L368 EN**: Returns from the current function with `__nvvm_atom_xchg_gen_i(__p, __v)`.
  **L368 CN**: 以 `__nvvm_atom_xchg_gen_i(__p, __v)` 从当前函数返回。
- **L369 EN**: Closes the current lexical scope or compound statement.
  **L369 CN**: 结束当前词法作用域或复合语句块。
- **L370 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicExch_block(int *__p, int __v) {`.
  **L370 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicExch_block(int *__p, int __v) {`。
- **L371 EN**: Returns from the current function with `__nvvm_atom_cta_xchg_gen_i(__p, __v)`.
  **L371 CN**: 以 `__nvvm_atom_cta_xchg_gen_i(__p, __v)` 从当前函数返回。
- **L372 EN**: Closes the current lexical scope or compound statement.
  **L372 CN**: 结束当前词法作用域或复合语句块。
- **L373 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicExch_system(int *__p, int __v) {`.
  **L373 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicExch_system(int *__p, int __v) {`。
- **L374 EN**: Returns from the current function with `__nvvm_atom_sys_xchg_gen_i(__p, __v)`.
  **L374 CN**: 以 `__nvvm_atom_sys_xchg_gen_i(__p, __v)` 从当前函数返回。
- **L375 EN**: Closes the current lexical scope or compound statement.
  **L375 CN**: 结束当前词法作用域或复合语句块。
- **L376 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicMax(int *__p, int __v) {`.
  **L376 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicMax(int *__p, int __v) {`。
- **L377 EN**: Returns from the current function with `__nvvm_atom_max_gen_i(__p, __v)`.
  **L377 CN**: 以 `__nvvm_atom_max_gen_i(__p, __v)` 从当前函数返回。
- **L378 EN**: Closes the current lexical scope or compound statement.
  **L378 CN**: 结束当前词法作用域或复合语句块。
- **L379 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicMax_block(int *__p, int __v) {`.
  **L379 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicMax_block(int *__p, int __v) {`。
- **L380 EN**: Returns from the current function with `__nvvm_atom_cta_max_gen_i(__p, __v)`.
  **L380 CN**: 以 `__nvvm_atom_cta_max_gen_i(__p, __v)` 从当前函数返回。
- **L381 EN**: Closes the current lexical scope or compound statement.
  **L381 CN**: 结束当前词法作用域或复合语句块。
- **L382 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicMax_system(int *__p, int __v) {`.
  **L382 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicMax_system(int *__p, int __v) {`。
- **L383 EN**: Returns from the current function with `__nvvm_atom_sys_max_gen_i(__p, __v)`.
  **L383 CN**: 以 `__nvvm_atom_sys_max_gen_i(__p, __v)` 从当前函数返回。
- **L384 EN**: Closes the current lexical scope or compound statement.
  **L384 CN**: 结束当前词法作用域或复合语句块。

### Lines 385-408

````c
__DEVICE__ int __iAtomicMin(int *__p, int __v) {
  return __nvvm_atom_min_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicMin_block(int *__p, int __v) {
  return __nvvm_atom_cta_min_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicMin_system(int *__p, int __v) {
  return __nvvm_atom_sys_min_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicOr(int *__p, int __v) {
  return __nvvm_atom_or_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicOr_block(int *__p, int __v) {
  return __nvvm_atom_cta_or_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicOr_system(int *__p, int __v) {
  return __nvvm_atom_sys_or_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicXor(int *__p, int __v) {
  return __nvvm_atom_xor_gen_i(__p, __v);
}
__DEVICE__ int __iAtomicXor_block(int *__p, int __v) {
  return __nvvm_atom_cta_xor_gen_i(__p, __v);
}
````
- **L385 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicMin(int *__p, int __v) {`.
  **L385 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicMin(int *__p, int __v) {`。
- **L386 EN**: Returns from the current function with `__nvvm_atom_min_gen_i(__p, __v)`.
  **L386 CN**: 以 `__nvvm_atom_min_gen_i(__p, __v)` 从当前函数返回。
- **L387 EN**: Closes the current lexical scope or compound statement.
  **L387 CN**: 结束当前词法作用域或复合语句块。
- **L388 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicMin_block(int *__p, int __v) {`.
  **L388 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicMin_block(int *__p, int __v) {`。
- **L389 EN**: Returns from the current function with `__nvvm_atom_cta_min_gen_i(__p, __v)`.
  **L389 CN**: 以 `__nvvm_atom_cta_min_gen_i(__p, __v)` 从当前函数返回。
- **L390 EN**: Closes the current lexical scope or compound statement.
  **L390 CN**: 结束当前词法作用域或复合语句块。
- **L391 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicMin_system(int *__p, int __v) {`.
  **L391 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicMin_system(int *__p, int __v) {`。
- **L392 EN**: Returns from the current function with `__nvvm_atom_sys_min_gen_i(__p, __v)`.
  **L392 CN**: 以 `__nvvm_atom_sys_min_gen_i(__p, __v)` 从当前函数返回。
- **L393 EN**: Closes the current lexical scope or compound statement.
  **L393 CN**: 结束当前词法作用域或复合语句块。
- **L394 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicOr(int *__p, int __v) {`.
  **L394 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicOr(int *__p, int __v) {`。
- **L395 EN**: Returns from the current function with `__nvvm_atom_or_gen_i(__p, __v)`.
  **L395 CN**: 以 `__nvvm_atom_or_gen_i(__p, __v)` 从当前函数返回。
- **L396 EN**: Closes the current lexical scope or compound statement.
  **L396 CN**: 结束当前词法作用域或复合语句块。
- **L397 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicOr_block(int *__p, int __v) {`.
  **L397 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicOr_block(int *__p, int __v) {`。
- **L398 EN**: Returns from the current function with `__nvvm_atom_cta_or_gen_i(__p, __v)`.
  **L398 CN**: 以 `__nvvm_atom_cta_or_gen_i(__p, __v)` 从当前函数返回。
- **L399 EN**: Closes the current lexical scope or compound statement.
  **L399 CN**: 结束当前词法作用域或复合语句块。
- **L400 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicOr_system(int *__p, int __v) {`.
  **L400 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicOr_system(int *__p, int __v) {`。
- **L401 EN**: Returns from the current function with `__nvvm_atom_sys_or_gen_i(__p, __v)`.
  **L401 CN**: 以 `__nvvm_atom_sys_or_gen_i(__p, __v)` 从当前函数返回。
- **L402 EN**: Closes the current lexical scope or compound statement.
  **L402 CN**: 结束当前词法作用域或复合语句块。
- **L403 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicXor(int *__p, int __v) {`.
  **L403 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicXor(int *__p, int __v) {`。
- **L404 EN**: Returns from the current function with `__nvvm_atom_xor_gen_i(__p, __v)`.
  **L404 CN**: 以 `__nvvm_atom_xor_gen_i(__p, __v)` 从当前函数返回。
- **L405 EN**: Closes the current lexical scope or compound statement.
  **L405 CN**: 结束当前词法作用域或复合语句块。
- **L406 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicXor_block(int *__p, int __v) {`.
  **L406 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicXor_block(int *__p, int __v) {`。
- **L407 EN**: Returns from the current function with `__nvvm_atom_cta_xor_gen_i(__p, __v)`.
  **L407 CN**: 以 `__nvvm_atom_cta_xor_gen_i(__p, __v)` 从当前函数返回。
- **L408 EN**: Closes the current lexical scope or compound statement.
  **L408 CN**: 结束当前词法作用域或复合语句块。

### Lines 409-432

````c
__DEVICE__ int __iAtomicXor_system(int *__p, int __v) {
  return __nvvm_atom_sys_xor_gen_i(__p, __v);
}
__DEVICE__ long long __illAtomicMax(long long *__p, long long __v) {
  return __nvvm_atom_max_gen_ll(__p, __v);
}
__DEVICE__ long long __illAtomicMax_block(long long *__p, long long __v) {
  return __nvvm_atom_cta_max_gen_ll(__p, __v);
}
__DEVICE__ long long __illAtomicMax_system(long long *__p, long long __v) {
  return __nvvm_atom_sys_max_gen_ll(__p, __v);
}
__DEVICE__ long long __illAtomicMin(long long *__p, long long __v) {
  return __nvvm_atom_min_gen_ll(__p, __v);
}
__DEVICE__ long long __illAtomicMin_block(long long *__p, long long __v) {
  return __nvvm_atom_cta_min_gen_ll(__p, __v);
}
__DEVICE__ long long __illAtomicMin_system(long long *__p, long long __v) {
  return __nvvm_atom_sys_min_gen_ll(__p, __v);
}
__DEVICE__ double __int2double_rn(int __a) { return __nv_int2double_rn(__a); }
__DEVICE__ float __int2float_rd(int __a) { return __nv_int2float_rd(__a); }
__DEVICE__ float __int2float_rn(int __a) { return __nv_int2float_rn(__a); }
````
- **L409 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ int __iAtomicXor_system(int *__p, int __v) {`.
  **L409 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ int __iAtomicXor_system(int *__p, int __v) {`。
- **L410 EN**: Returns from the current function with `__nvvm_atom_sys_xor_gen_i(__p, __v)`.
  **L410 CN**: 以 `__nvvm_atom_sys_xor_gen_i(__p, __v)` 从当前函数返回。
- **L411 EN**: Closes the current lexical scope or compound statement.
  **L411 CN**: 结束当前词法作用域或复合语句块。
- **L412 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __illAtomicMax(long long *__p, long long __v) {`.
  **L412 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __illAtomicMax(long long *__p, long long __v) {`。
- **L413 EN**: Returns from the current function with `__nvvm_atom_max_gen_ll(__p, __v)`.
  **L413 CN**: 以 `__nvvm_atom_max_gen_ll(__p, __v)` 从当前函数返回。
- **L414 EN**: Closes the current lexical scope or compound statement.
  **L414 CN**: 结束当前词法作用域或复合语句块。
- **L415 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __illAtomicMax_block(long long *__p, long long __v) {`.
  **L415 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __illAtomicMax_block(long long *__p, long long __v) {`。
- **L416 EN**: Returns from the current function with `__nvvm_atom_cta_max_gen_ll(__p, __v)`.
  **L416 CN**: 以 `__nvvm_atom_cta_max_gen_ll(__p, __v)` 从当前函数返回。
- **L417 EN**: Closes the current lexical scope or compound statement.
  **L417 CN**: 结束当前词法作用域或复合语句块。
- **L418 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __illAtomicMax_system(long long *__p, long long __v) {`.
  **L418 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __illAtomicMax_system(long long *__p, long long __v) {`。
- **L419 EN**: Returns from the current function with `__nvvm_atom_sys_max_gen_ll(__p, __v)`.
  **L419 CN**: 以 `__nvvm_atom_sys_max_gen_ll(__p, __v)` 从当前函数返回。
- **L420 EN**: Closes the current lexical scope or compound statement.
  **L420 CN**: 结束当前词法作用域或复合语句块。
- **L421 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __illAtomicMin(long long *__p, long long __v) {`.
  **L421 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __illAtomicMin(long long *__p, long long __v) {`。
- **L422 EN**: Returns from the current function with `__nvvm_atom_min_gen_ll(__p, __v)`.
  **L422 CN**: 以 `__nvvm_atom_min_gen_ll(__p, __v)` 从当前函数返回。
- **L423 EN**: Closes the current lexical scope or compound statement.
  **L423 CN**: 结束当前词法作用域或复合语句块。
- **L424 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __illAtomicMin_block(long long *__p, long long __v) {`.
  **L424 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __illAtomicMin_block(long long *__p, long long __v) {`。
- **L425 EN**: Returns from the current function with `__nvvm_atom_cta_min_gen_ll(__p, __v)`.
  **L425 CN**: 以 `__nvvm_atom_cta_min_gen_ll(__p, __v)` 从当前函数返回。
- **L426 EN**: Closes the current lexical scope or compound statement.
  **L426 CN**: 结束当前词法作用域或复合语句块。
- **L427 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __illAtomicMin_system(long long *__p, long long __v) {`.
  **L427 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __illAtomicMin_system(long long *__p, long long __v) {`。
- **L428 EN**: Returns from the current function with `__nvvm_atom_sys_min_gen_ll(__p, __v)`.
  **L428 CN**: 以 `__nvvm_atom_sys_min_gen_ll(__p, __v)` 从当前函数返回。
- **L429 EN**: Closes the current lexical scope or compound statement.
  **L429 CN**: 结束当前词法作用域或复合语句块。
- **L430 EN**: Continues logic associated with callable symbol `__int2double_rn`.
  **L430 CN**: 继续与可调用符号 `__int2double_rn` 相关的逻辑。
- **L431 EN**: Continues logic associated with callable symbol `__int2float_rd`.
  **L431 CN**: 继续与可调用符号 `__int2float_rd` 相关的逻辑。
- **L432 EN**: Continues logic associated with callable symbol `__int2float_rn`.
  **L432 CN**: 继续与可调用符号 `__int2float_rn` 相关的逻辑。

### Lines 433-456

````c
__DEVICE__ float __int2float_ru(int __a) { return __nv_int2float_ru(__a); }
__DEVICE__ float __int2float_rz(int __a) { return __nv_int2float_rz(__a); }
__DEVICE__ float __int_as_float(int __a) { return __nv_int_as_float(__a); }
__DEVICE__ int __isfinited(double __a) { return __nv_isfinited(__a); }
__DEVICE__ int __isinf(double __a) { return __nv_isinfd(__a); }
__DEVICE__ int __isinff(float __a) { return __nv_isinff(__a); }
#ifdef _MSC_VER
__DEVICE__ int __isinfl(long double __a);
#endif
__DEVICE__ int __isnan(double __a) { return __nv_isnand(__a); }
__DEVICE__ int __isnanf(float __a) { return __nv_isnanf(__a); }
#ifdef _MSC_VER
__DEVICE__ int __isnanl(long double __a);
#endif
__DEVICE__ double __ll2double_rd(long long __a) {
  return __nv_ll2double_rd(__a);
}
__DEVICE__ double __ll2double_rn(long long __a) {
  return __nv_ll2double_rn(__a);
}
__DEVICE__ double __ll2double_ru(long long __a) {
  return __nv_ll2double_ru(__a);
}
__DEVICE__ double __ll2double_rz(long long __a) {
````
- **L433 EN**: Continues logic associated with callable symbol `__int2float_ru`.
  **L433 CN**: 继续与可调用符号 `__int2float_ru` 相关的逻辑。
- **L434 EN**: Continues logic associated with callable symbol `__int2float_rz`.
  **L434 CN**: 继续与可调用符号 `__int2float_rz` 相关的逻辑。
- **L435 EN**: Continues logic associated with callable symbol `__int_as_float`.
  **L435 CN**: 继续与可调用符号 `__int_as_float` 相关的逻辑。
- **L436 EN**: Continues logic associated with callable symbol `__isfinited`.
  **L436 CN**: 继续与可调用符号 `__isfinited` 相关的逻辑。
- **L437 EN**: Continues logic associated with callable symbol `__isinf`.
  **L437 CN**: 继续与可调用符号 `__isinf` 相关的逻辑。
- **L438 EN**: Continues logic associated with callable symbol `__isinff`.
  **L438 CN**: 继续与可调用符号 `__isinff` 相关的逻辑。
- **L439 EN**: Starts a preprocessor conditional block: `#ifdef _MSC_VER`.
  **L439 CN**: 开始一个预处理条件块：`#ifdef _MSC_VER`。
- **L440 EN**: Executes a call or declaration centered on `__isinfl`.
  **L440 CN**: 执行以 `__isinfl` 为核心的调用或声明。
- **L441 EN**: Closes the current preprocessor conditional block.
  **L441 CN**: 结束当前预处理条件块。
- **L442 EN**: Continues logic associated with callable symbol `__isnan`.
  **L442 CN**: 继续与可调用符号 `__isnan` 相关的逻辑。
- **L443 EN**: Continues logic associated with callable symbol `__isnanf`.
  **L443 CN**: 继续与可调用符号 `__isnanf` 相关的逻辑。
- **L444 EN**: Starts a preprocessor conditional block: `#ifdef _MSC_VER`.
  **L444 CN**: 开始一个预处理条件块：`#ifdef _MSC_VER`。
- **L445 EN**: Executes a call or declaration centered on `__isnanl`.
  **L445 CN**: 执行以 `__isnanl` 为核心的调用或声明。
- **L446 EN**: Closes the current preprocessor conditional block.
  **L446 CN**: 结束当前预处理条件块。
- **L447 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __ll2double_rd(long long __a) {`.
  **L447 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __ll2double_rd(long long __a) {`。
- **L448 EN**: Returns from the current function with `__nv_ll2double_rd(__a)`.
  **L448 CN**: 以 `__nv_ll2double_rd(__a)` 从当前函数返回。
- **L449 EN**: Closes the current lexical scope or compound statement.
  **L449 CN**: 结束当前词法作用域或复合语句块。
- **L450 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __ll2double_rn(long long __a) {`.
  **L450 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __ll2double_rn(long long __a) {`。
- **L451 EN**: Returns from the current function with `__nv_ll2double_rn(__a)`.
  **L451 CN**: 以 `__nv_ll2double_rn(__a)` 从当前函数返回。
- **L452 EN**: Closes the current lexical scope or compound statement.
  **L452 CN**: 结束当前词法作用域或复合语句块。
- **L453 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __ll2double_ru(long long __a) {`.
  **L453 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __ll2double_ru(long long __a) {`。
- **L454 EN**: Returns from the current function with `__nv_ll2double_ru(__a)`.
  **L454 CN**: 以 `__nv_ll2double_ru(__a)` 从当前函数返回。
- **L455 EN**: Closes the current lexical scope or compound statement.
  **L455 CN**: 结束当前词法作用域或复合语句块。
- **L456 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __ll2double_rz(long long __a) {`.
  **L456 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __ll2double_rz(long long __a) {`。

### Lines 457-480

````c
  return __nv_ll2double_rz(__a);
}
__DEVICE__ float __ll2float_rd(long long __a) { return __nv_ll2float_rd(__a); }
__DEVICE__ float __ll2float_rn(long long __a) { return __nv_ll2float_rn(__a); }
__DEVICE__ float __ll2float_ru(long long __a) { return __nv_ll2float_ru(__a); }
__DEVICE__ float __ll2float_rz(long long __a) { return __nv_ll2float_rz(__a); }
__DEVICE__ long long __llAtomicAnd(long long *__p, long long __v) {
  return __nvvm_atom_and_gen_ll(__p, __v);
}
__DEVICE__ long long __llAtomicAnd_block(long long *__p, long long __v) {
  return __nvvm_atom_cta_and_gen_ll(__p, __v);
}
__DEVICE__ long long __llAtomicAnd_system(long long *__p, long long __v) {
  return __nvvm_atom_sys_and_gen_ll(__p, __v);
}
__DEVICE__ long long __llAtomicOr(long long *__p, long long __v) {
  return __nvvm_atom_or_gen_ll(__p, __v);
}
__DEVICE__ long long __llAtomicOr_block(long long *__p, long long __v) {
  return __nvvm_atom_cta_or_gen_ll(__p, __v);
}
__DEVICE__ long long __llAtomicOr_system(long long *__p, long long __v) {
  return __nvvm_atom_sys_or_gen_ll(__p, __v);
}
````
- **L457 EN**: Returns from the current function with `__nv_ll2double_rz(__a)`.
  **L457 CN**: 以 `__nv_ll2double_rz(__a)` 从当前函数返回。
- **L458 EN**: Closes the current lexical scope or compound statement.
  **L458 CN**: 结束当前词法作用域或复合语句块。
- **L459 EN**: Continues logic associated with callable symbol `__ll2float_rd`.
  **L459 CN**: 继续与可调用符号 `__ll2float_rd` 相关的逻辑。
- **L460 EN**: Continues logic associated with callable symbol `__ll2float_rn`.
  **L460 CN**: 继续与可调用符号 `__ll2float_rn` 相关的逻辑。
- **L461 EN**: Continues logic associated with callable symbol `__ll2float_ru`.
  **L461 CN**: 继续与可调用符号 `__ll2float_ru` 相关的逻辑。
- **L462 EN**: Continues logic associated with callable symbol `__ll2float_rz`.
  **L462 CN**: 继续与可调用符号 `__ll2float_rz` 相关的逻辑。
- **L463 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __llAtomicAnd(long long *__p, long long __v) {`.
  **L463 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __llAtomicAnd(long long *__p, long long __v) {`。
- **L464 EN**: Returns from the current function with `__nvvm_atom_and_gen_ll(__p, __v)`.
  **L464 CN**: 以 `__nvvm_atom_and_gen_ll(__p, __v)` 从当前函数返回。
- **L465 EN**: Closes the current lexical scope or compound statement.
  **L465 CN**: 结束当前词法作用域或复合语句块。
- **L466 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __llAtomicAnd_block(long long *__p, long long __v) {`.
  **L466 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __llAtomicAnd_block(long long *__p, long long __v) {`。
- **L467 EN**: Returns from the current function with `__nvvm_atom_cta_and_gen_ll(__p, __v)`.
  **L467 CN**: 以 `__nvvm_atom_cta_and_gen_ll(__p, __v)` 从当前函数返回。
- **L468 EN**: Closes the current lexical scope or compound statement.
  **L468 CN**: 结束当前词法作用域或复合语句块。
- **L469 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __llAtomicAnd_system(long long *__p, long long __v) {`.
  **L469 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __llAtomicAnd_system(long long *__p, long long __v) {`。
- **L470 EN**: Returns from the current function with `__nvvm_atom_sys_and_gen_ll(__p, __v)`.
  **L470 CN**: 以 `__nvvm_atom_sys_and_gen_ll(__p, __v)` 从当前函数返回。
- **L471 EN**: Closes the current lexical scope or compound statement.
  **L471 CN**: 结束当前词法作用域或复合语句块。
- **L472 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __llAtomicOr(long long *__p, long long __v) {`.
  **L472 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __llAtomicOr(long long *__p, long long __v) {`。
- **L473 EN**: Returns from the current function with `__nvvm_atom_or_gen_ll(__p, __v)`.
  **L473 CN**: 以 `__nvvm_atom_or_gen_ll(__p, __v)` 从当前函数返回。
- **L474 EN**: Closes the current lexical scope or compound statement.
  **L474 CN**: 结束当前词法作用域或复合语句块。
- **L475 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __llAtomicOr_block(long long *__p, long long __v) {`.
  **L475 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __llAtomicOr_block(long long *__p, long long __v) {`。
- **L476 EN**: Returns from the current function with `__nvvm_atom_cta_or_gen_ll(__p, __v)`.
  **L476 CN**: 以 `__nvvm_atom_cta_or_gen_ll(__p, __v)` 从当前函数返回。
- **L477 EN**: Closes the current lexical scope or compound statement.
  **L477 CN**: 结束当前词法作用域或复合语句块。
- **L478 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __llAtomicOr_system(long long *__p, long long __v) {`.
  **L478 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __llAtomicOr_system(long long *__p, long long __v) {`。
- **L479 EN**: Returns from the current function with `__nvvm_atom_sys_or_gen_ll(__p, __v)`.
  **L479 CN**: 以 `__nvvm_atom_sys_or_gen_ll(__p, __v)` 从当前函数返回。
- **L480 EN**: Closes the current lexical scope or compound statement.
  **L480 CN**: 结束当前词法作用域或复合语句块。

### Lines 481-504

````c
__DEVICE__ long long __llAtomicXor(long long *__p, long long __v) {
  return __nvvm_atom_xor_gen_ll(__p, __v);
}
__DEVICE__ long long __llAtomicXor_block(long long *__p, long long __v) {
  return __nvvm_atom_cta_xor_gen_ll(__p, __v);
}
__DEVICE__ long long __llAtomicXor_system(long long *__p, long long __v) {
  return __nvvm_atom_sys_xor_gen_ll(__p, __v);
}
__DEVICE__ float __log10f(float __a) { return __nv_fast_log10f(__a); }
__DEVICE__ float __log2f(float __a) { return __nv_fast_log2f(__a); }
__DEVICE__ float __logf(float __a) { return __nv_fast_logf(__a); }
__DEVICE__ double __longlong_as_double(long long __a) {
  return __nv_longlong_as_double(__a);
}
__DEVICE__ int __mul24(int __a, int __b) { return __nv_mul24(__a, __b); }
__DEVICE__ long long __mul64hi(long long __a, long long __b) {
  return __nv_mul64hi(__a, __b);
}
__DEVICE__ int __mulhi(int __a, int __b) { return __nv_mulhi(__a, __b); }
__DEVICE__ unsigned int __pm0(void) { return __nvvm_read_ptx_sreg_pm0(); }
__DEVICE__ unsigned int __pm1(void) { return __nvvm_read_ptx_sreg_pm1(); }
__DEVICE__ unsigned int __pm2(void) { return __nvvm_read_ptx_sreg_pm2(); }
__DEVICE__ unsigned int __pm3(void) { return __nvvm_read_ptx_sreg_pm3(); }
````
- **L481 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __llAtomicXor(long long *__p, long long __v) {`.
  **L481 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __llAtomicXor(long long *__p, long long __v) {`。
- **L482 EN**: Returns from the current function with `__nvvm_atom_xor_gen_ll(__p, __v)`.
  **L482 CN**: 以 `__nvvm_atom_xor_gen_ll(__p, __v)` 从当前函数返回。
- **L483 EN**: Closes the current lexical scope or compound statement.
  **L483 CN**: 结束当前词法作用域或复合语句块。
- **L484 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __llAtomicXor_block(long long *__p, long long __v) {`.
  **L484 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __llAtomicXor_block(long long *__p, long long __v) {`。
- **L485 EN**: Returns from the current function with `__nvvm_atom_cta_xor_gen_ll(__p, __v)`.
  **L485 CN**: 以 `__nvvm_atom_cta_xor_gen_ll(__p, __v)` 从当前函数返回。
- **L486 EN**: Closes the current lexical scope or compound statement.
  **L486 CN**: 结束当前词法作用域或复合语句块。
- **L487 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __llAtomicXor_system(long long *__p, long long __v) {`.
  **L487 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __llAtomicXor_system(long long *__p, long long __v) {`。
- **L488 EN**: Returns from the current function with `__nvvm_atom_sys_xor_gen_ll(__p, __v)`.
  **L488 CN**: 以 `__nvvm_atom_sys_xor_gen_ll(__p, __v)` 从当前函数返回。
- **L489 EN**: Closes the current lexical scope or compound statement.
  **L489 CN**: 结束当前词法作用域或复合语句块。
- **L490 EN**: Continues logic associated with callable symbol `__log10f`.
  **L490 CN**: 继续与可调用符号 `__log10f` 相关的逻辑。
- **L491 EN**: Continues logic associated with callable symbol `__log2f`.
  **L491 CN**: 继续与可调用符号 `__log2f` 相关的逻辑。
- **L492 EN**: Continues logic associated with callable symbol `__logf`.
  **L492 CN**: 继续与可调用符号 `__logf` 相关的逻辑。
- **L493 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __longlong_as_double(long long __a) {`.
  **L493 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __longlong_as_double(long long __a) {`。
- **L494 EN**: Returns from the current function with `__nv_longlong_as_double(__a)`.
  **L494 CN**: 以 `__nv_longlong_as_double(__a)` 从当前函数返回。
- **L495 EN**: Closes the current lexical scope or compound statement.
  **L495 CN**: 结束当前词法作用域或复合语句块。
- **L496 EN**: Continues logic associated with callable symbol `__mul24`.
  **L496 CN**: 继续与可调用符号 `__mul24` 相关的逻辑。
- **L497 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ long long __mul64hi(long long __a, long long __b) {`.
  **L497 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ long long __mul64hi(long long __a, long long __b) {`。
- **L498 EN**: Returns from the current function with `__nv_mul64hi(__a, __b)`.
  **L498 CN**: 以 `__nv_mul64hi(__a, __b)` 从当前函数返回。
- **L499 EN**: Closes the current lexical scope or compound statement.
  **L499 CN**: 结束当前词法作用域或复合语句块。
- **L500 EN**: Continues logic associated with callable symbol `__mulhi`.
  **L500 CN**: 继续与可调用符号 `__mulhi` 相关的逻辑。
- **L501 EN**: Continues logic associated with callable symbol `__pm0`.
  **L501 CN**: 继续与可调用符号 `__pm0` 相关的逻辑。
- **L502 EN**: Continues logic associated with callable symbol `__pm1`.
  **L502 CN**: 继续与可调用符号 `__pm1` 相关的逻辑。
- **L503 EN**: Continues logic associated with callable symbol `__pm2`.
  **L503 CN**: 继续与可调用符号 `__pm2` 相关的逻辑。
- **L504 EN**: Continues logic associated with callable symbol `__pm3`.
  **L504 CN**: 继续与可调用符号 `__pm3` 相关的逻辑。

### Lines 505-528

````c
__DEVICE__ int __popc(unsigned int __a) { return __nv_popc(__a); }
__DEVICE__ int __popcll(unsigned long long __a) { return __nv_popcll(__a); }
__DEVICE__ float __powf(float __a, float __b) {
  return __nv_fast_powf(__a, __b);
}

// Parameter must have a known integer value.
#define __prof_trigger(__a) __asm__ __volatile__("pmevent \t%0;" ::"i"(__a))
__DEVICE__ int __rhadd(int __a, int __b) { return __nv_rhadd(__a, __b); }
__DEVICE__ unsigned int __sad(int __a, int __b, unsigned int __c) {
  return __nv_sad(__a, __b, __c);
}
__DEVICE__ float __saturatef(float __a) { return __nv_saturatef(__a); }
__DEVICE__ int __signbitd(double __a) { return __nv_signbitd(__a); }
__DEVICE__ int __signbitf(float __a) { return __nv_signbitf(__a); }
__DEVICE__ void __sincosf(float __a, float *__s, float *__c) {
  return __nv_fast_sincosf(__a, __s, __c);
}
__DEVICE__ float __sinf(float __a) { return __nv_fast_sinf(__a); }
__DEVICE__ int __syncthreads_and(int __a) { return __nvvm_bar0_and(__a); }
__DEVICE__ int __syncthreads_count(int __a) { return __nvvm_bar0_popc(__a); }
__DEVICE__ int __syncthreads_or(int __a) { return __nvvm_bar0_or(__a); }
__DEVICE__ float __tanf(float __a) { return __nv_fast_tanf(__a); }
__DEVICE__ void __threadfence(void) { __nvvm_membar_gl(); }
````
- **L505 EN**: Continues logic associated with callable symbol `__popc`.
  **L505 CN**: 继续与可调用符号 `__popc` 相关的逻辑。
- **L506 EN**: Continues logic associated with callable symbol `__popcll`.
  **L506 CN**: 继续与可调用符号 `__popcll` 相关的逻辑。
- **L507 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __powf(float __a, float __b) {`.
  **L507 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __powf(float __a, float __b) {`。
- **L508 EN**: Returns from the current function with `__nv_fast_powf(__a, __b)`.
  **L508 CN**: 以 `__nv_fast_powf(__a, __b)` 从当前函数返回。
- **L509 EN**: Closes the current lexical scope or compound statement.
  **L509 CN**: 结束当前词法作用域或复合语句块。
- **L510 EN**: Blank line separating nearby declarations or logic blocks.
  **L510 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L511 EN**: Comment explains nearby logic, constraints, or intent: `Parameter must have a known integer value.`.
  **L511 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Parameter must have a known integer value.`。
- **L512 EN**: Defines macro `__prof_trigger(__a)` for conditional compilation, shorthand, or API generation.
  **L512 CN**: 定义宏 `__prof_trigger(__a)`，用于条件编译、简写或 API 生成。
- **L513 EN**: Continues logic associated with callable symbol `__rhadd`.
  **L513 CN**: 继续与可调用符号 `__rhadd` 相关的逻辑。
- **L514 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __sad(int __a, int __b, unsigned int __c) {`.
  **L514 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __sad(int __a, int __b, unsigned int __c) {`。
- **L515 EN**: Returns from the current function with `__nv_sad(__a, __b, __c)`.
  **L515 CN**: 以 `__nv_sad(__a, __b, __c)` 从当前函数返回。
- **L516 EN**: Closes the current lexical scope or compound statement.
  **L516 CN**: 结束当前词法作用域或复合语句块。
- **L517 EN**: Continues logic associated with callable symbol `__saturatef`.
  **L517 CN**: 继续与可调用符号 `__saturatef` 相关的逻辑。
- **L518 EN**: Continues logic associated with callable symbol `__signbitd`.
  **L518 CN**: 继续与可调用符号 `__signbitd` 相关的逻辑。
- **L519 EN**: Continues logic associated with callable symbol `__signbitf`.
  **L519 CN**: 继续与可调用符号 `__signbitf` 相关的逻辑。
- **L520 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ void __sincosf(float __a, float *__s, float *__c) {`.
  **L520 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ void __sincosf(float __a, float *__s, float *__c) {`。
- **L521 EN**: Returns from the current function with `__nv_fast_sincosf(__a, __s, __c)`.
  **L521 CN**: 以 `__nv_fast_sincosf(__a, __s, __c)` 从当前函数返回。
- **L522 EN**: Closes the current lexical scope or compound statement.
  **L522 CN**: 结束当前词法作用域或复合语句块。
- **L523 EN**: Continues logic associated with callable symbol `__sinf`.
  **L523 CN**: 继续与可调用符号 `__sinf` 相关的逻辑。
- **L524 EN**: Continues logic associated with callable symbol `__syncthreads_and`.
  **L524 CN**: 继续与可调用符号 `__syncthreads_and` 相关的逻辑。
- **L525 EN**: Continues logic associated with callable symbol `__syncthreads_count`.
  **L525 CN**: 继续与可调用符号 `__syncthreads_count` 相关的逻辑。
- **L526 EN**: Continues logic associated with callable symbol `__syncthreads_or`.
  **L526 CN**: 继续与可调用符号 `__syncthreads_or` 相关的逻辑。
- **L527 EN**: Continues logic associated with callable symbol `__tanf`.
  **L527 CN**: 继续与可调用符号 `__tanf` 相关的逻辑。
- **L528 EN**: Continues logic associated with callable symbol `__threadfence`.
  **L528 CN**: 继续与可调用符号 `__threadfence` 相关的逻辑。

### Lines 529-552

````c
__DEVICE__ void __threadfence_block(void) { __nvvm_membar_cta(); };
__DEVICE__ void __threadfence_system(void) { __nvvm_membar_sys(); };
__DEVICE__ __attribute__((noreturn)) void __trap(void) { __builtin_trap(); }
__DEVICE__ unsigned short
__usAtomicCAS(unsigned short *__p, unsigned short __cmp, unsigned short __v) {
  return __nvvm_atom_cas_gen_us(__p, __cmp, __v);
}
__DEVICE__ unsigned short __usAtomicCAS_block(unsigned short *__p,
                                              unsigned short __cmp,
                                              unsigned short __v) {
  return __nvvm_atom_cta_cas_gen_us(__p, __cmp, __v);
}
__DEVICE__ unsigned short __usAtomicCAS_system(unsigned short *__p,
                                               unsigned short __cmp,
                                               unsigned short __v) {
  return __nvvm_atom_sys_cas_gen_us(__p, __cmp, __v);
}
__DEVICE__ unsigned int __uAtomicAdd(unsigned int *__p, unsigned int __v) {
  return __nvvm_atom_add_gen_i((int *)__p, __v);
}
__DEVICE__ unsigned int __uAtomicAdd_block(unsigned int *__p,
                                           unsigned int __v) {
  return __nvvm_atom_cta_add_gen_i((int *)__p, __v);
}
````
- **L529 EN**: Executes a call or declaration centered on `__threadfence_block`.
  **L529 CN**: 执行以 `__threadfence_block` 为核心的调用或声明。
- **L530 EN**: Executes a call or declaration centered on `__threadfence_system`.
  **L530 CN**: 执行以 `__threadfence_system` 为核心的调用或声明。
- **L531 EN**: Applies compiler-specific attributes that refine ABI, inlining, or diagnostics: `__DEVICE__ __attribute__((noreturn)) void __trap(void) { __builtin_trap(); }`.
  **L531 CN**: 应用编译器特定属性，以细化 ABI、内联或诊断行为：`__DEVICE__ __attribute__((noreturn)) void __trap(void) { __builtin_trap(); }`。
- **L532 EN**: Continues the surrounding expression or declaration: `__DEVICE__ unsigned short`.
  **L532 CN**: 继续构造周围的表达式或声明：`__DEVICE__ unsigned short`。
- **L533 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__usAtomicCAS(unsigned short *__p, unsigned short __cmp, unsigned short __v) {`.
  **L533 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__usAtomicCAS(unsigned short *__p, unsigned short __cmp, unsigned short __v) {`。
- **L534 EN**: Returns from the current function with `__nvvm_atom_cas_gen_us(__p, __cmp, __v)`.
  **L534 CN**: 以 `__nvvm_atom_cas_gen_us(__p, __cmp, __v)` 从当前函数返回。
- **L535 EN**: Closes the current lexical scope or compound statement.
  **L535 CN**: 结束当前词法作用域或复合语句块。
- **L536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned short __usAtomicCAS_block(unsigned short *__p,`.
  **L536 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned short __usAtomicCAS_block(unsigned short *__p,`。
- **L537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned short __cmp,`.
  **L537 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned short __cmp,`。
- **L538 EN**: Continues the surrounding expression or declaration: `unsigned short __v) {`.
  **L538 CN**: 继续构造周围的表达式或声明：`unsigned short __v) {`。
- **L539 EN**: Returns from the current function with `__nvvm_atom_cta_cas_gen_us(__p, __cmp, __v)`.
  **L539 CN**: 以 `__nvvm_atom_cta_cas_gen_us(__p, __cmp, __v)` 从当前函数返回。
- **L540 EN**: Closes the current lexical scope or compound statement.
  **L540 CN**: 结束当前词法作用域或复合语句块。
- **L541 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned short __usAtomicCAS_system(unsigned short *__p,`.
  **L541 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned short __usAtomicCAS_system(unsigned short *__p,`。
- **L542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned short __cmp,`.
  **L542 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned short __cmp,`。
- **L543 EN**: Continues the surrounding expression or declaration: `unsigned short __v) {`.
  **L543 CN**: 继续构造周围的表达式或声明：`unsigned short __v) {`。
- **L544 EN**: Returns from the current function with `__nvvm_atom_sys_cas_gen_us(__p, __cmp, __v)`.
  **L544 CN**: 以 `__nvvm_atom_sys_cas_gen_us(__p, __cmp, __v)` 从当前函数返回。
- **L545 EN**: Closes the current lexical scope or compound statement.
  **L545 CN**: 结束当前词法作用域或复合语句块。
- **L546 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __uAtomicAdd(unsigned int *__p, unsigned int __v) {`.
  **L546 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __uAtomicAdd(unsigned int *__p, unsigned int __v) {`。
- **L547 EN**: Returns from the current function with `__nvvm_atom_add_gen_i((int *)__p, __v)`.
  **L547 CN**: 以 `__nvvm_atom_add_gen_i((int *)__p, __v)` 从当前函数返回。
- **L548 EN**: Closes the current lexical scope or compound statement.
  **L548 CN**: 结束当前词法作用域或复合语句块。
- **L549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicAdd_block(unsigned int *__p,`.
  **L549 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicAdd_block(unsigned int *__p,`。
- **L550 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L550 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L551 EN**: Returns from the current function with `__nvvm_atom_cta_add_gen_i((int *)__p, __v)`.
  **L551 CN**: 以 `__nvvm_atom_cta_add_gen_i((int *)__p, __v)` 从当前函数返回。
- **L552 EN**: Closes the current lexical scope or compound statement.
  **L552 CN**: 结束当前词法作用域或复合语句块。

### Lines 553-576

````c
__DEVICE__ unsigned int __uAtomicAdd_system(unsigned int *__p,
                                            unsigned int __v) {
  return __nvvm_atom_sys_add_gen_i((int *)__p, __v);
}
__DEVICE__ unsigned int __uAtomicAnd(unsigned int *__p, unsigned int __v) {
  return __nvvm_atom_and_gen_i((int *)__p, __v);
}
__DEVICE__ unsigned int __uAtomicAnd_block(unsigned int *__p,
                                           unsigned int __v) {
  return __nvvm_atom_cta_and_gen_i((int *)__p, __v);
}
__DEVICE__ unsigned int __uAtomicAnd_system(unsigned int *__p,
                                            unsigned int __v) {
  return __nvvm_atom_sys_and_gen_i((int *)__p, __v);
}
__DEVICE__ unsigned int __uAtomicCAS(unsigned int *__p, unsigned int __cmp,
                                     unsigned int __v) {
  return __nvvm_atom_cas_gen_i((int *)__p, __cmp, __v);
}
__DEVICE__ unsigned int
__uAtomicCAS_block(unsigned int *__p, unsigned int __cmp, unsigned int __v) {
  return __nvvm_atom_cta_cas_gen_i((int *)__p, __cmp, __v);
}
__DEVICE__ unsigned int
````
- **L553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicAdd_system(unsigned int *__p,`.
  **L553 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicAdd_system(unsigned int *__p,`。
- **L554 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L554 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L555 EN**: Returns from the current function with `__nvvm_atom_sys_add_gen_i((int *)__p, __v)`.
  **L555 CN**: 以 `__nvvm_atom_sys_add_gen_i((int *)__p, __v)` 从当前函数返回。
- **L556 EN**: Closes the current lexical scope or compound statement.
  **L556 CN**: 结束当前词法作用域或复合语句块。
- **L557 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __uAtomicAnd(unsigned int *__p, unsigned int __v) {`.
  **L557 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __uAtomicAnd(unsigned int *__p, unsigned int __v) {`。
- **L558 EN**: Returns from the current function with `__nvvm_atom_and_gen_i((int *)__p, __v)`.
  **L558 CN**: 以 `__nvvm_atom_and_gen_i((int *)__p, __v)` 从当前函数返回。
- **L559 EN**: Closes the current lexical scope or compound statement.
  **L559 CN**: 结束当前词法作用域或复合语句块。
- **L560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicAnd_block(unsigned int *__p,`.
  **L560 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicAnd_block(unsigned int *__p,`。
- **L561 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L561 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L562 EN**: Returns from the current function with `__nvvm_atom_cta_and_gen_i((int *)__p, __v)`.
  **L562 CN**: 以 `__nvvm_atom_cta_and_gen_i((int *)__p, __v)` 从当前函数返回。
- **L563 EN**: Closes the current lexical scope or compound statement.
  **L563 CN**: 结束当前词法作用域或复合语句块。
- **L564 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicAnd_system(unsigned int *__p,`.
  **L564 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicAnd_system(unsigned int *__p,`。
- **L565 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L565 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L566 EN**: Returns from the current function with `__nvvm_atom_sys_and_gen_i((int *)__p, __v)`.
  **L566 CN**: 以 `__nvvm_atom_sys_and_gen_i((int *)__p, __v)` 从当前函数返回。
- **L567 EN**: Closes the current lexical scope or compound statement.
  **L567 CN**: 结束当前词法作用域或复合语句块。
- **L568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicCAS(unsigned int *__p, unsigned int __cmp,`.
  **L568 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicCAS(unsigned int *__p, unsigned int __cmp,`。
- **L569 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L569 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L570 EN**: Returns from the current function with `__nvvm_atom_cas_gen_i((int *)__p, __cmp, __v)`.
  **L570 CN**: 以 `__nvvm_atom_cas_gen_i((int *)__p, __cmp, __v)` 从当前函数返回。
- **L571 EN**: Closes the current lexical scope or compound statement.
  **L571 CN**: 结束当前词法作用域或复合语句块。
- **L572 EN**: Continues the surrounding expression or declaration: `__DEVICE__ unsigned int`.
  **L572 CN**: 继续构造周围的表达式或声明：`__DEVICE__ unsigned int`。
- **L573 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__uAtomicCAS_block(unsigned int *__p, unsigned int __cmp, unsigned int __v) {`.
  **L573 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__uAtomicCAS_block(unsigned int *__p, unsigned int __cmp, unsigned int __v) {`。
- **L574 EN**: Returns from the current function with `__nvvm_atom_cta_cas_gen_i((int *)__p, __cmp, __v)`.
  **L574 CN**: 以 `__nvvm_atom_cta_cas_gen_i((int *)__p, __cmp, __v)` 从当前函数返回。
- **L575 EN**: Closes the current lexical scope or compound statement.
  **L575 CN**: 结束当前词法作用域或复合语句块。
- **L576 EN**: Continues the surrounding expression or declaration: `__DEVICE__ unsigned int`.
  **L576 CN**: 继续构造周围的表达式或声明：`__DEVICE__ unsigned int`。

### Lines 577-600

````c
__uAtomicCAS_system(unsigned int *__p, unsigned int __cmp, unsigned int __v) {
  return __nvvm_atom_sys_cas_gen_i((int *)__p, __cmp, __v);
}
__DEVICE__ unsigned int __uAtomicDec(unsigned int *__p, unsigned int __v) {
  return __nvvm_atom_dec_gen_ui(__p, __v);
}
__DEVICE__ unsigned int __uAtomicDec_block(unsigned int *__p,
                                           unsigned int __v) {
  return __nvvm_atom_cta_dec_gen_ui(__p, __v);
}
__DEVICE__ unsigned int __uAtomicDec_system(unsigned int *__p,
                                            unsigned int __v) {
  return __nvvm_atom_sys_dec_gen_ui(__p, __v);
}
__DEVICE__ unsigned int __uAtomicExch(unsigned int *__p, unsigned int __v) {
  return __nvvm_atom_xchg_gen_i((int *)__p, __v);
}
__DEVICE__ unsigned int __uAtomicExch_block(unsigned int *__p,
                                            unsigned int __v) {
  return __nvvm_atom_cta_xchg_gen_i((int *)__p, __v);
}
__DEVICE__ unsigned int __uAtomicExch_system(unsigned int *__p,
                                             unsigned int __v) {
  return __nvvm_atom_sys_xchg_gen_i((int *)__p, __v);
````
- **L577 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__uAtomicCAS_system(unsigned int *__p, unsigned int __cmp, unsigned int __v) {`.
  **L577 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__uAtomicCAS_system(unsigned int *__p, unsigned int __cmp, unsigned int __v) {`。
- **L578 EN**: Returns from the current function with `__nvvm_atom_sys_cas_gen_i((int *)__p, __cmp, __v)`.
  **L578 CN**: 以 `__nvvm_atom_sys_cas_gen_i((int *)__p, __cmp, __v)` 从当前函数返回。
- **L579 EN**: Closes the current lexical scope or compound statement.
  **L579 CN**: 结束当前词法作用域或复合语句块。
- **L580 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __uAtomicDec(unsigned int *__p, unsigned int __v) {`.
  **L580 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __uAtomicDec(unsigned int *__p, unsigned int __v) {`。
- **L581 EN**: Returns from the current function with `__nvvm_atom_dec_gen_ui(__p, __v)`.
  **L581 CN**: 以 `__nvvm_atom_dec_gen_ui(__p, __v)` 从当前函数返回。
- **L582 EN**: Closes the current lexical scope or compound statement.
  **L582 CN**: 结束当前词法作用域或复合语句块。
- **L583 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicDec_block(unsigned int *__p,`.
  **L583 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicDec_block(unsigned int *__p,`。
- **L584 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L584 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L585 EN**: Returns from the current function with `__nvvm_atom_cta_dec_gen_ui(__p, __v)`.
  **L585 CN**: 以 `__nvvm_atom_cta_dec_gen_ui(__p, __v)` 从当前函数返回。
- **L586 EN**: Closes the current lexical scope or compound statement.
  **L586 CN**: 结束当前词法作用域或复合语句块。
- **L587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicDec_system(unsigned int *__p,`.
  **L587 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicDec_system(unsigned int *__p,`。
- **L588 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L588 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L589 EN**: Returns from the current function with `__nvvm_atom_sys_dec_gen_ui(__p, __v)`.
  **L589 CN**: 以 `__nvvm_atom_sys_dec_gen_ui(__p, __v)` 从当前函数返回。
- **L590 EN**: Closes the current lexical scope or compound statement.
  **L590 CN**: 结束当前词法作用域或复合语句块。
- **L591 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __uAtomicExch(unsigned int *__p, unsigned int __v) {`.
  **L591 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __uAtomicExch(unsigned int *__p, unsigned int __v) {`。
- **L592 EN**: Returns from the current function with `__nvvm_atom_xchg_gen_i((int *)__p, __v)`.
  **L592 CN**: 以 `__nvvm_atom_xchg_gen_i((int *)__p, __v)` 从当前函数返回。
- **L593 EN**: Closes the current lexical scope or compound statement.
  **L593 CN**: 结束当前词法作用域或复合语句块。
- **L594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicExch_block(unsigned int *__p,`.
  **L594 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicExch_block(unsigned int *__p,`。
- **L595 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L595 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L596 EN**: Returns from the current function with `__nvvm_atom_cta_xchg_gen_i((int *)__p, __v)`.
  **L596 CN**: 以 `__nvvm_atom_cta_xchg_gen_i((int *)__p, __v)` 从当前函数返回。
- **L597 EN**: Closes the current lexical scope or compound statement.
  **L597 CN**: 结束当前词法作用域或复合语句块。
- **L598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicExch_system(unsigned int *__p,`.
  **L598 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicExch_system(unsigned int *__p,`。
- **L599 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L599 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L600 EN**: Returns from the current function with `__nvvm_atom_sys_xchg_gen_i((int *)__p, __v)`.
  **L600 CN**: 以 `__nvvm_atom_sys_xchg_gen_i((int *)__p, __v)` 从当前函数返回。

### Lines 601-624

````c
}
__DEVICE__ unsigned int __uAtomicInc(unsigned int *__p, unsigned int __v) {
  return __nvvm_atom_inc_gen_ui(__p, __v);
}
__DEVICE__ unsigned int __uAtomicInc_block(unsigned int *__p,
                                           unsigned int __v) {
  return __nvvm_atom_cta_inc_gen_ui(__p, __v);
}
__DEVICE__ unsigned int __uAtomicInc_system(unsigned int *__p,
                                            unsigned int __v) {
  return __nvvm_atom_sys_inc_gen_ui(__p, __v);
}
__DEVICE__ unsigned int __uAtomicMax(unsigned int *__p, unsigned int __v) {
  return __nvvm_atom_max_gen_ui(__p, __v);
}
__DEVICE__ unsigned int __uAtomicMax_block(unsigned int *__p,
                                           unsigned int __v) {
  return __nvvm_atom_cta_max_gen_ui(__p, __v);
}
__DEVICE__ unsigned int __uAtomicMax_system(unsigned int *__p,
                                            unsigned int __v) {
  return __nvvm_atom_sys_max_gen_ui(__p, __v);
}
__DEVICE__ unsigned int __uAtomicMin(unsigned int *__p, unsigned int __v) {
````
- **L601 EN**: Closes the current lexical scope or compound statement.
  **L601 CN**: 结束当前词法作用域或复合语句块。
- **L602 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __uAtomicInc(unsigned int *__p, unsigned int __v) {`.
  **L602 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __uAtomicInc(unsigned int *__p, unsigned int __v) {`。
- **L603 EN**: Returns from the current function with `__nvvm_atom_inc_gen_ui(__p, __v)`.
  **L603 CN**: 以 `__nvvm_atom_inc_gen_ui(__p, __v)` 从当前函数返回。
- **L604 EN**: Closes the current lexical scope or compound statement.
  **L604 CN**: 结束当前词法作用域或复合语句块。
- **L605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicInc_block(unsigned int *__p,`.
  **L605 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicInc_block(unsigned int *__p,`。
- **L606 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L606 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L607 EN**: Returns from the current function with `__nvvm_atom_cta_inc_gen_ui(__p, __v)`.
  **L607 CN**: 以 `__nvvm_atom_cta_inc_gen_ui(__p, __v)` 从当前函数返回。
- **L608 EN**: Closes the current lexical scope or compound statement.
  **L608 CN**: 结束当前词法作用域或复合语句块。
- **L609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicInc_system(unsigned int *__p,`.
  **L609 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicInc_system(unsigned int *__p,`。
- **L610 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L610 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L611 EN**: Returns from the current function with `__nvvm_atom_sys_inc_gen_ui(__p, __v)`.
  **L611 CN**: 以 `__nvvm_atom_sys_inc_gen_ui(__p, __v)` 从当前函数返回。
- **L612 EN**: Closes the current lexical scope or compound statement.
  **L612 CN**: 结束当前词法作用域或复合语句块。
- **L613 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __uAtomicMax(unsigned int *__p, unsigned int __v) {`.
  **L613 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __uAtomicMax(unsigned int *__p, unsigned int __v) {`。
- **L614 EN**: Returns from the current function with `__nvvm_atom_max_gen_ui(__p, __v)`.
  **L614 CN**: 以 `__nvvm_atom_max_gen_ui(__p, __v)` 从当前函数返回。
- **L615 EN**: Closes the current lexical scope or compound statement.
  **L615 CN**: 结束当前词法作用域或复合语句块。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicMax_block(unsigned int *__p,`.
  **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicMax_block(unsigned int *__p,`。
- **L617 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L617 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L618 EN**: Returns from the current function with `__nvvm_atom_cta_max_gen_ui(__p, __v)`.
  **L618 CN**: 以 `__nvvm_atom_cta_max_gen_ui(__p, __v)` 从当前函数返回。
- **L619 EN**: Closes the current lexical scope or compound statement.
  **L619 CN**: 结束当前词法作用域或复合语句块。
- **L620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicMax_system(unsigned int *__p,`.
  **L620 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicMax_system(unsigned int *__p,`。
- **L621 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L621 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L622 EN**: Returns from the current function with `__nvvm_atom_sys_max_gen_ui(__p, __v)`.
  **L622 CN**: 以 `__nvvm_atom_sys_max_gen_ui(__p, __v)` 从当前函数返回。
- **L623 EN**: Closes the current lexical scope or compound statement.
  **L623 CN**: 结束当前词法作用域或复合语句块。
- **L624 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __uAtomicMin(unsigned int *__p, unsigned int __v) {`.
  **L624 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __uAtomicMin(unsigned int *__p, unsigned int __v) {`。

### Lines 625-648

````c
  return __nvvm_atom_min_gen_ui(__p, __v);
}
__DEVICE__ unsigned int __uAtomicMin_block(unsigned int *__p,
                                           unsigned int __v) {
  return __nvvm_atom_cta_min_gen_ui(__p, __v);
}
__DEVICE__ unsigned int __uAtomicMin_system(unsigned int *__p,
                                            unsigned int __v) {
  return __nvvm_atom_sys_min_gen_ui(__p, __v);
}
__DEVICE__ unsigned int __uAtomicOr(unsigned int *__p, unsigned int __v) {
  return __nvvm_atom_or_gen_i((int *)__p, __v);
}
__DEVICE__ unsigned int __uAtomicOr_block(unsigned int *__p, unsigned int __v) {
  return __nvvm_atom_cta_or_gen_i((int *)__p, __v);
}
__DEVICE__ unsigned int __uAtomicOr_system(unsigned int *__p,
                                           unsigned int __v) {
  return __nvvm_atom_sys_or_gen_i((int *)__p, __v);
}
__DEVICE__ unsigned int __uAtomicXor(unsigned int *__p, unsigned int __v) {
  return __nvvm_atom_xor_gen_i((int *)__p, __v);
}
__DEVICE__ unsigned int __uAtomicXor_block(unsigned int *__p,
````
- **L625 EN**: Returns from the current function with `__nvvm_atom_min_gen_ui(__p, __v)`.
  **L625 CN**: 以 `__nvvm_atom_min_gen_ui(__p, __v)` 从当前函数返回。
- **L626 EN**: Closes the current lexical scope or compound statement.
  **L626 CN**: 结束当前词法作用域或复合语句块。
- **L627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicMin_block(unsigned int *__p,`.
  **L627 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicMin_block(unsigned int *__p,`。
- **L628 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L628 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L629 EN**: Returns from the current function with `__nvvm_atom_cta_min_gen_ui(__p, __v)`.
  **L629 CN**: 以 `__nvvm_atom_cta_min_gen_ui(__p, __v)` 从当前函数返回。
- **L630 EN**: Closes the current lexical scope or compound statement.
  **L630 CN**: 结束当前词法作用域或复合语句块。
- **L631 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicMin_system(unsigned int *__p,`.
  **L631 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicMin_system(unsigned int *__p,`。
- **L632 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L632 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L633 EN**: Returns from the current function with `__nvvm_atom_sys_min_gen_ui(__p, __v)`.
  **L633 CN**: 以 `__nvvm_atom_sys_min_gen_ui(__p, __v)` 从当前函数返回。
- **L634 EN**: Closes the current lexical scope or compound statement.
  **L634 CN**: 结束当前词法作用域或复合语句块。
- **L635 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __uAtomicOr(unsigned int *__p, unsigned int __v) {`.
  **L635 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __uAtomicOr(unsigned int *__p, unsigned int __v) {`。
- **L636 EN**: Returns from the current function with `__nvvm_atom_or_gen_i((int *)__p, __v)`.
  **L636 CN**: 以 `__nvvm_atom_or_gen_i((int *)__p, __v)` 从当前函数返回。
- **L637 EN**: Closes the current lexical scope or compound statement.
  **L637 CN**: 结束当前词法作用域或复合语句块。
- **L638 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __uAtomicOr_block(unsigned int *__p, unsigned int __v) {`.
  **L638 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __uAtomicOr_block(unsigned int *__p, unsigned int __v) {`。
- **L639 EN**: Returns from the current function with `__nvvm_atom_cta_or_gen_i((int *)__p, __v)`.
  **L639 CN**: 以 `__nvvm_atom_cta_or_gen_i((int *)__p, __v)` 从当前函数返回。
- **L640 EN**: Closes the current lexical scope or compound statement.
  **L640 CN**: 结束当前词法作用域或复合语句块。
- **L641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicOr_system(unsigned int *__p,`.
  **L641 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicOr_system(unsigned int *__p,`。
- **L642 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L642 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L643 EN**: Returns from the current function with `__nvvm_atom_sys_or_gen_i((int *)__p, __v)`.
  **L643 CN**: 以 `__nvvm_atom_sys_or_gen_i((int *)__p, __v)` 从当前函数返回。
- **L644 EN**: Closes the current lexical scope or compound statement.
  **L644 CN**: 结束当前词法作用域或复合语句块。
- **L645 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __uAtomicXor(unsigned int *__p, unsigned int __v) {`.
  **L645 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __uAtomicXor(unsigned int *__p, unsigned int __v) {`。
- **L646 EN**: Returns from the current function with `__nvvm_atom_xor_gen_i((int *)__p, __v)`.
  **L646 CN**: 以 `__nvvm_atom_xor_gen_i((int *)__p, __v)` 从当前函数返回。
- **L647 EN**: Closes the current lexical scope or compound statement.
  **L647 CN**: 结束当前词法作用域或复合语句块。
- **L648 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicXor_block(unsigned int *__p,`.
  **L648 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicXor_block(unsigned int *__p,`。

### Lines 649-672

````c
                                           unsigned int __v) {
  return __nvvm_atom_cta_xor_gen_i((int *)__p, __v);
}
__DEVICE__ unsigned int __uAtomicXor_system(unsigned int *__p,
                                            unsigned int __v) {
  return __nvvm_atom_sys_xor_gen_i((int *)__p, __v);
}
__DEVICE__ unsigned int __uhadd(unsigned int __a, unsigned int __b) {
  return __nv_uhadd(__a, __b);
}
__DEVICE__ double __uint2double_rn(unsigned int __a) {
  return __nv_uint2double_rn(__a);
}
__DEVICE__ float __uint2float_rd(unsigned int __a) {
  return __nv_uint2float_rd(__a);
}
__DEVICE__ float __uint2float_rn(unsigned int __a) {
  return __nv_uint2float_rn(__a);
}
__DEVICE__ float __uint2float_ru(unsigned int __a) {
  return __nv_uint2float_ru(__a);
}
__DEVICE__ float __uint2float_rz(unsigned int __a) {
  return __nv_uint2float_rz(__a);
````
- **L649 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L649 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L650 EN**: Returns from the current function with `__nvvm_atom_cta_xor_gen_i((int *)__p, __v)`.
  **L650 CN**: 以 `__nvvm_atom_cta_xor_gen_i((int *)__p, __v)` 从当前函数返回。
- **L651 EN**: Closes the current lexical scope or compound statement.
  **L651 CN**: 结束当前词法作用域或复合语句块。
- **L652 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __uAtomicXor_system(unsigned int *__p,`.
  **L652 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __uAtomicXor_system(unsigned int *__p,`。
- **L653 EN**: Continues the surrounding expression or declaration: `unsigned int __v) {`.
  **L653 CN**: 继续构造周围的表达式或声明：`unsigned int __v) {`。
- **L654 EN**: Returns from the current function with `__nvvm_atom_sys_xor_gen_i((int *)__p, __v)`.
  **L654 CN**: 以 `__nvvm_atom_sys_xor_gen_i((int *)__p, __v)` 从当前函数返回。
- **L655 EN**: Closes the current lexical scope or compound statement.
  **L655 CN**: 结束当前词法作用域或复合语句块。
- **L656 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __uhadd(unsigned int __a, unsigned int __b) {`.
  **L656 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __uhadd(unsigned int __a, unsigned int __b) {`。
- **L657 EN**: Returns from the current function with `__nv_uhadd(__a, __b)`.
  **L657 CN**: 以 `__nv_uhadd(__a, __b)` 从当前函数返回。
- **L658 EN**: Closes the current lexical scope or compound statement.
  **L658 CN**: 结束当前词法作用域或复合语句块。
- **L659 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __uint2double_rn(unsigned int __a) {`.
  **L659 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __uint2double_rn(unsigned int __a) {`。
- **L660 EN**: Returns from the current function with `__nv_uint2double_rn(__a)`.
  **L660 CN**: 以 `__nv_uint2double_rn(__a)` 从当前函数返回。
- **L661 EN**: Closes the current lexical scope or compound statement.
  **L661 CN**: 结束当前词法作用域或复合语句块。
- **L662 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __uint2float_rd(unsigned int __a) {`.
  **L662 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __uint2float_rd(unsigned int __a) {`。
- **L663 EN**: Returns from the current function with `__nv_uint2float_rd(__a)`.
  **L663 CN**: 以 `__nv_uint2float_rd(__a)` 从当前函数返回。
- **L664 EN**: Closes the current lexical scope or compound statement.
  **L664 CN**: 结束当前词法作用域或复合语句块。
- **L665 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __uint2float_rn(unsigned int __a) {`.
  **L665 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __uint2float_rn(unsigned int __a) {`。
- **L666 EN**: Returns from the current function with `__nv_uint2float_rn(__a)`.
  **L666 CN**: 以 `__nv_uint2float_rn(__a)` 从当前函数返回。
- **L667 EN**: Closes the current lexical scope or compound statement.
  **L667 CN**: 结束当前词法作用域或复合语句块。
- **L668 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __uint2float_ru(unsigned int __a) {`.
  **L668 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __uint2float_ru(unsigned int __a) {`。
- **L669 EN**: Returns from the current function with `__nv_uint2float_ru(__a)`.
  **L669 CN**: 以 `__nv_uint2float_ru(__a)` 从当前函数返回。
- **L670 EN**: Closes the current lexical scope or compound statement.
  **L670 CN**: 结束当前词法作用域或复合语句块。
- **L671 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __uint2float_rz(unsigned int __a) {`.
  **L671 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __uint2float_rz(unsigned int __a) {`。
- **L672 EN**: Returns from the current function with `__nv_uint2float_rz(__a)`.
  **L672 CN**: 以 `__nv_uint2float_rz(__a)` 从当前函数返回。

### Lines 673-696

````c
}
__DEVICE__ float __uint_as_float(unsigned int __a) {
  return __nv_uint_as_float(__a);
} //
__DEVICE__ double __ull2double_rd(unsigned long long __a) {
  return __nv_ull2double_rd(__a);
}
__DEVICE__ double __ull2double_rn(unsigned long long __a) {
  return __nv_ull2double_rn(__a);
}
__DEVICE__ double __ull2double_ru(unsigned long long __a) {
  return __nv_ull2double_ru(__a);
}
__DEVICE__ double __ull2double_rz(unsigned long long __a) {
  return __nv_ull2double_rz(__a);
}
__DEVICE__ float __ull2float_rd(unsigned long long __a) {
  return __nv_ull2float_rd(__a);
}
__DEVICE__ float __ull2float_rn(unsigned long long __a) {
  return __nv_ull2float_rn(__a);
}
__DEVICE__ float __ull2float_ru(unsigned long long __a) {
  return __nv_ull2float_ru(__a);
````
- **L673 EN**: Closes the current lexical scope or compound statement.
  **L673 CN**: 结束当前词法作用域或复合语句块。
- **L674 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __uint_as_float(unsigned int __a) {`.
  **L674 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __uint_as_float(unsigned int __a) {`。
- **L675 EN**: Returns from the current function with `__nv_uint_as_float(__a)`.
  **L675 CN**: 以 `__nv_uint_as_float(__a)` 从当前函数返回。
- **L676 EN**: Continues the surrounding expression or declaration: `} //`.
  **L676 CN**: 继续构造周围的表达式或声明：`} //`。
- **L677 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __ull2double_rd(unsigned long long __a) {`.
  **L677 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __ull2double_rd(unsigned long long __a) {`。
- **L678 EN**: Returns from the current function with `__nv_ull2double_rd(__a)`.
  **L678 CN**: 以 `__nv_ull2double_rd(__a)` 从当前函数返回。
- **L679 EN**: Closes the current lexical scope or compound statement.
  **L679 CN**: 结束当前词法作用域或复合语句块。
- **L680 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __ull2double_rn(unsigned long long __a) {`.
  **L680 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __ull2double_rn(unsigned long long __a) {`。
- **L681 EN**: Returns from the current function with `__nv_ull2double_rn(__a)`.
  **L681 CN**: 以 `__nv_ull2double_rn(__a)` 从当前函数返回。
- **L682 EN**: Closes the current lexical scope or compound statement.
  **L682 CN**: 结束当前词法作用域或复合语句块。
- **L683 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __ull2double_ru(unsigned long long __a) {`.
  **L683 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __ull2double_ru(unsigned long long __a) {`。
- **L684 EN**: Returns from the current function with `__nv_ull2double_ru(__a)`.
  **L684 CN**: 以 `__nv_ull2double_ru(__a)` 从当前函数返回。
- **L685 EN**: Closes the current lexical scope or compound statement.
  **L685 CN**: 结束当前词法作用域或复合语句块。
- **L686 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ double __ull2double_rz(unsigned long long __a) {`.
  **L686 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ double __ull2double_rz(unsigned long long __a) {`。
- **L687 EN**: Returns from the current function with `__nv_ull2double_rz(__a)`.
  **L687 CN**: 以 `__nv_ull2double_rz(__a)` 从当前函数返回。
- **L688 EN**: Closes the current lexical scope or compound statement.
  **L688 CN**: 结束当前词法作用域或复合语句块。
- **L689 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __ull2float_rd(unsigned long long __a) {`.
  **L689 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __ull2float_rd(unsigned long long __a) {`。
- **L690 EN**: Returns from the current function with `__nv_ull2float_rd(__a)`.
  **L690 CN**: 以 `__nv_ull2float_rd(__a)` 从当前函数返回。
- **L691 EN**: Closes the current lexical scope or compound statement.
  **L691 CN**: 结束当前词法作用域或复合语句块。
- **L692 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __ull2float_rn(unsigned long long __a) {`.
  **L692 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __ull2float_rn(unsigned long long __a) {`。
- **L693 EN**: Returns from the current function with `__nv_ull2float_rn(__a)`.
  **L693 CN**: 以 `__nv_ull2float_rn(__a)` 从当前函数返回。
- **L694 EN**: Closes the current lexical scope or compound statement.
  **L694 CN**: 结束当前词法作用域或复合语句块。
- **L695 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __ull2float_ru(unsigned long long __a) {`.
  **L695 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __ull2float_ru(unsigned long long __a) {`。
- **L696 EN**: Returns from the current function with `__nv_ull2float_ru(__a)`.
  **L696 CN**: 以 `__nv_ull2float_ru(__a)` 从当前函数返回。

### Lines 697-720

````c
}
__DEVICE__ float __ull2float_rz(unsigned long long __a) {
  return __nv_ull2float_rz(__a);
}
__DEVICE__ unsigned long long __ullAtomicAdd(unsigned long long *__p,
                                             unsigned long long __v) {
  return __nvvm_atom_add_gen_ll((long long *)__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicAdd_block(unsigned long long *__p,
                                                   unsigned long long __v) {
  return __nvvm_atom_cta_add_gen_ll((long long *)__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicAdd_system(unsigned long long *__p,
                                                    unsigned long long __v) {
  return __nvvm_atom_sys_add_gen_ll((long long *)__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicAnd(unsigned long long *__p,
                                             unsigned long long __v) {
  return __nvvm_atom_and_gen_ll((long long *)__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicAnd_block(unsigned long long *__p,
                                                   unsigned long long __v) {
  return __nvvm_atom_cta_and_gen_ll((long long *)__p, __v);
}
````
- **L697 EN**: Closes the current lexical scope or compound statement.
  **L697 CN**: 结束当前词法作用域或复合语句块。
- **L698 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ float __ull2float_rz(unsigned long long __a) {`.
  **L698 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ float __ull2float_rz(unsigned long long __a) {`。
- **L699 EN**: Returns from the current function with `__nv_ull2float_rz(__a)`.
  **L699 CN**: 以 `__nv_ull2float_rz(__a)` 从当前函数返回。
- **L700 EN**: Closes the current lexical scope or compound statement.
  **L700 CN**: 结束当前词法作用域或复合语句块。
- **L701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicAdd(unsigned long long *__p,`.
  **L701 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicAdd(unsigned long long *__p,`。
- **L702 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L702 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L703 EN**: Returns from the current function with `__nvvm_atom_add_gen_ll((long long *)__p, __v)`.
  **L703 CN**: 以 `__nvvm_atom_add_gen_ll((long long *)__p, __v)` 从当前函数返回。
- **L704 EN**: Closes the current lexical scope or compound statement.
  **L704 CN**: 结束当前词法作用域或复合语句块。
- **L705 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicAdd_block(unsigned long long *__p,`.
  **L705 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicAdd_block(unsigned long long *__p,`。
- **L706 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L706 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L707 EN**: Returns from the current function with `__nvvm_atom_cta_add_gen_ll((long long *)__p, __v)`.
  **L707 CN**: 以 `__nvvm_atom_cta_add_gen_ll((long long *)__p, __v)` 从当前函数返回。
- **L708 EN**: Closes the current lexical scope or compound statement.
  **L708 CN**: 结束当前词法作用域或复合语句块。
- **L709 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicAdd_system(unsigned long long *__p,`.
  **L709 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicAdd_system(unsigned long long *__p,`。
- **L710 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L710 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L711 EN**: Returns from the current function with `__nvvm_atom_sys_add_gen_ll((long long *)__p, __v)`.
  **L711 CN**: 以 `__nvvm_atom_sys_add_gen_ll((long long *)__p, __v)` 从当前函数返回。
- **L712 EN**: Closes the current lexical scope or compound statement.
  **L712 CN**: 结束当前词法作用域或复合语句块。
- **L713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicAnd(unsigned long long *__p,`.
  **L713 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicAnd(unsigned long long *__p,`。
- **L714 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L714 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L715 EN**: Returns from the current function with `__nvvm_atom_and_gen_ll((long long *)__p, __v)`.
  **L715 CN**: 以 `__nvvm_atom_and_gen_ll((long long *)__p, __v)` 从当前函数返回。
- **L716 EN**: Closes the current lexical scope or compound statement.
  **L716 CN**: 结束当前词法作用域或复合语句块。
- **L717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicAnd_block(unsigned long long *__p,`.
  **L717 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicAnd_block(unsigned long long *__p,`。
- **L718 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L718 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L719 EN**: Returns from the current function with `__nvvm_atom_cta_and_gen_ll((long long *)__p, __v)`.
  **L719 CN**: 以 `__nvvm_atom_cta_and_gen_ll((long long *)__p, __v)` 从当前函数返回。
- **L720 EN**: Closes the current lexical scope or compound statement.
  **L720 CN**: 结束当前词法作用域或复合语句块。

### Lines 721-744

````c
__DEVICE__ unsigned long long __ullAtomicAnd_system(unsigned long long *__p,
                                                    unsigned long long __v) {
  return __nvvm_atom_sys_and_gen_ll((long long *)__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicCAS(unsigned long long *__p,
                                             unsigned long long __cmp,
                                             unsigned long long __v) {
  return __nvvm_atom_cas_gen_ll((long long *)__p, __cmp, __v);
}
__DEVICE__ unsigned long long __ullAtomicCAS_block(unsigned long long *__p,
                                                   unsigned long long __cmp,
                                                   unsigned long long __v) {
  return __nvvm_atom_cta_cas_gen_ll((long long *)__p, __cmp, __v);
}
__DEVICE__ unsigned long long __ullAtomicCAS_system(unsigned long long *__p,
                                                    unsigned long long __cmp,
                                                    unsigned long long __v) {
  return __nvvm_atom_sys_cas_gen_ll((long long *)__p, __cmp, __v);
}
__DEVICE__ unsigned long long __ullAtomicExch(unsigned long long *__p,
                                              unsigned long long __v) {
  return __nvvm_atom_xchg_gen_ll((long long *)__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicExch_block(unsigned long long *__p,
````
- **L721 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicAnd_system(unsigned long long *__p,`.
  **L721 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicAnd_system(unsigned long long *__p,`。
- **L722 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L722 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L723 EN**: Returns from the current function with `__nvvm_atom_sys_and_gen_ll((long long *)__p, __v)`.
  **L723 CN**: 以 `__nvvm_atom_sys_and_gen_ll((long long *)__p, __v)` 从当前函数返回。
- **L724 EN**: Closes the current lexical scope or compound statement.
  **L724 CN**: 结束当前词法作用域或复合语句块。
- **L725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicCAS(unsigned long long *__p,`.
  **L725 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicCAS(unsigned long long *__p,`。
- **L726 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned long long __cmp,`.
  **L726 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned long long __cmp,`。
- **L727 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L727 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L728 EN**: Returns from the current function with `__nvvm_atom_cas_gen_ll((long long *)__p, __cmp, __v)`.
  **L728 CN**: 以 `__nvvm_atom_cas_gen_ll((long long *)__p, __cmp, __v)` 从当前函数返回。
- **L729 EN**: Closes the current lexical scope or compound statement.
  **L729 CN**: 结束当前词法作用域或复合语句块。
- **L730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicCAS_block(unsigned long long *__p,`.
  **L730 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicCAS_block(unsigned long long *__p,`。
- **L731 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned long long __cmp,`.
  **L731 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned long long __cmp,`。
- **L732 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L732 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L733 EN**: Returns from the current function with `__nvvm_atom_cta_cas_gen_ll((long long *)__p, __cmp, __v)`.
  **L733 CN**: 以 `__nvvm_atom_cta_cas_gen_ll((long long *)__p, __cmp, __v)` 从当前函数返回。
- **L734 EN**: Closes the current lexical scope or compound statement.
  **L734 CN**: 结束当前词法作用域或复合语句块。
- **L735 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicCAS_system(unsigned long long *__p,`.
  **L735 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicCAS_system(unsigned long long *__p,`。
- **L736 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned long long __cmp,`.
  **L736 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned long long __cmp,`。
- **L737 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L737 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L738 EN**: Returns from the current function with `__nvvm_atom_sys_cas_gen_ll((long long *)__p, __cmp, __v)`.
  **L738 CN**: 以 `__nvvm_atom_sys_cas_gen_ll((long long *)__p, __cmp, __v)` 从当前函数返回。
- **L739 EN**: Closes the current lexical scope or compound statement.
  **L739 CN**: 结束当前词法作用域或复合语句块。
- **L740 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicExch(unsigned long long *__p,`.
  **L740 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicExch(unsigned long long *__p,`。
- **L741 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L741 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L742 EN**: Returns from the current function with `__nvvm_atom_xchg_gen_ll((long long *)__p, __v)`.
  **L742 CN**: 以 `__nvvm_atom_xchg_gen_ll((long long *)__p, __v)` 从当前函数返回。
- **L743 EN**: Closes the current lexical scope or compound statement.
  **L743 CN**: 结束当前词法作用域或复合语句块。
- **L744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicExch_block(unsigned long long *__p,`.
  **L744 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicExch_block(unsigned long long *__p,`。

### Lines 745-768

````c
                                                    unsigned long long __v) {
  return __nvvm_atom_cta_xchg_gen_ll((long long *)__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicExch_system(unsigned long long *__p,
                                                     unsigned long long __v) {
  return __nvvm_atom_sys_xchg_gen_ll((long long *)__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicMax(unsigned long long *__p,
                                             unsigned long long __v) {
  return __nvvm_atom_max_gen_ull(__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicMax_block(unsigned long long *__p,
                                                   unsigned long long __v) {
  return __nvvm_atom_cta_max_gen_ull(__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicMax_system(unsigned long long *__p,
                                                    unsigned long long __v) {
  return __nvvm_atom_sys_max_gen_ull(__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicMin(unsigned long long *__p,
                                             unsigned long long __v) {
  return __nvvm_atom_min_gen_ull(__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicMin_block(unsigned long long *__p,
````
- **L745 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L745 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L746 EN**: Returns from the current function with `__nvvm_atom_cta_xchg_gen_ll((long long *)__p, __v)`.
  **L746 CN**: 以 `__nvvm_atom_cta_xchg_gen_ll((long long *)__p, __v)` 从当前函数返回。
- **L747 EN**: Closes the current lexical scope or compound statement.
  **L747 CN**: 结束当前词法作用域或复合语句块。
- **L748 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicExch_system(unsigned long long *__p,`.
  **L748 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicExch_system(unsigned long long *__p,`。
- **L749 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L749 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L750 EN**: Returns from the current function with `__nvvm_atom_sys_xchg_gen_ll((long long *)__p, __v)`.
  **L750 CN**: 以 `__nvvm_atom_sys_xchg_gen_ll((long long *)__p, __v)` 从当前函数返回。
- **L751 EN**: Closes the current lexical scope or compound statement.
  **L751 CN**: 结束当前词法作用域或复合语句块。
- **L752 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicMax(unsigned long long *__p,`.
  **L752 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicMax(unsigned long long *__p,`。
- **L753 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L753 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L754 EN**: Returns from the current function with `__nvvm_atom_max_gen_ull(__p, __v)`.
  **L754 CN**: 以 `__nvvm_atom_max_gen_ull(__p, __v)` 从当前函数返回。
- **L755 EN**: Closes the current lexical scope or compound statement.
  **L755 CN**: 结束当前词法作用域或复合语句块。
- **L756 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicMax_block(unsigned long long *__p,`.
  **L756 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicMax_block(unsigned long long *__p,`。
- **L757 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L757 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L758 EN**: Returns from the current function with `__nvvm_atom_cta_max_gen_ull(__p, __v)`.
  **L758 CN**: 以 `__nvvm_atom_cta_max_gen_ull(__p, __v)` 从当前函数返回。
- **L759 EN**: Closes the current lexical scope or compound statement.
  **L759 CN**: 结束当前词法作用域或复合语句块。
- **L760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicMax_system(unsigned long long *__p,`.
  **L760 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicMax_system(unsigned long long *__p,`。
- **L761 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L761 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L762 EN**: Returns from the current function with `__nvvm_atom_sys_max_gen_ull(__p, __v)`.
  **L762 CN**: 以 `__nvvm_atom_sys_max_gen_ull(__p, __v)` 从当前函数返回。
- **L763 EN**: Closes the current lexical scope or compound statement.
  **L763 CN**: 结束当前词法作用域或复合语句块。
- **L764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicMin(unsigned long long *__p,`.
  **L764 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicMin(unsigned long long *__p,`。
- **L765 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L765 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L766 EN**: Returns from the current function with `__nvvm_atom_min_gen_ull(__p, __v)`.
  **L766 CN**: 以 `__nvvm_atom_min_gen_ull(__p, __v)` 从当前函数返回。
- **L767 EN**: Closes the current lexical scope or compound statement.
  **L767 CN**: 结束当前词法作用域或复合语句块。
- **L768 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicMin_block(unsigned long long *__p,`.
  **L768 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicMin_block(unsigned long long *__p,`。

### Lines 769-792

````c
                                                   unsigned long long __v) {
  return __nvvm_atom_cta_min_gen_ull(__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicMin_system(unsigned long long *__p,
                                                    unsigned long long __v) {
  return __nvvm_atom_sys_min_gen_ull(__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicOr(unsigned long long *__p,
                                            unsigned long long __v) {
  return __nvvm_atom_or_gen_ll((long long *)__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicOr_block(unsigned long long *__p,
                                                  unsigned long long __v) {
  return __nvvm_atom_cta_or_gen_ll((long long *)__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicOr_system(unsigned long long *__p,
                                                   unsigned long long __v) {
  return __nvvm_atom_sys_or_gen_ll((long long *)__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicXor(unsigned long long *__p,
                                             unsigned long long __v) {
  return __nvvm_atom_xor_gen_ll((long long *)__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicXor_block(unsigned long long *__p,
````
- **L769 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L769 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L770 EN**: Returns from the current function with `__nvvm_atom_cta_min_gen_ull(__p, __v)`.
  **L770 CN**: 以 `__nvvm_atom_cta_min_gen_ull(__p, __v)` 从当前函数返回。
- **L771 EN**: Closes the current lexical scope or compound statement.
  **L771 CN**: 结束当前词法作用域或复合语句块。
- **L772 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicMin_system(unsigned long long *__p,`.
  **L772 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicMin_system(unsigned long long *__p,`。
- **L773 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L773 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L774 EN**: Returns from the current function with `__nvvm_atom_sys_min_gen_ull(__p, __v)`.
  **L774 CN**: 以 `__nvvm_atom_sys_min_gen_ull(__p, __v)` 从当前函数返回。
- **L775 EN**: Closes the current lexical scope or compound statement.
  **L775 CN**: 结束当前词法作用域或复合语句块。
- **L776 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicOr(unsigned long long *__p,`.
  **L776 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicOr(unsigned long long *__p,`。
- **L777 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L777 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L778 EN**: Returns from the current function with `__nvvm_atom_or_gen_ll((long long *)__p, __v)`.
  **L778 CN**: 以 `__nvvm_atom_or_gen_ll((long long *)__p, __v)` 从当前函数返回。
- **L779 EN**: Closes the current lexical scope or compound statement.
  **L779 CN**: 结束当前词法作用域或复合语句块。
- **L780 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicOr_block(unsigned long long *__p,`.
  **L780 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicOr_block(unsigned long long *__p,`。
- **L781 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L781 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L782 EN**: Returns from the current function with `__nvvm_atom_cta_or_gen_ll((long long *)__p, __v)`.
  **L782 CN**: 以 `__nvvm_atom_cta_or_gen_ll((long long *)__p, __v)` 从当前函数返回。
- **L783 EN**: Closes the current lexical scope or compound statement.
  **L783 CN**: 结束当前词法作用域或复合语句块。
- **L784 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicOr_system(unsigned long long *__p,`.
  **L784 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicOr_system(unsigned long long *__p,`。
- **L785 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L785 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L786 EN**: Returns from the current function with `__nvvm_atom_sys_or_gen_ll((long long *)__p, __v)`.
  **L786 CN**: 以 `__nvvm_atom_sys_or_gen_ll((long long *)__p, __v)` 从当前函数返回。
- **L787 EN**: Closes the current lexical scope or compound statement.
  **L787 CN**: 结束当前词法作用域或复合语句块。
- **L788 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicXor(unsigned long long *__p,`.
  **L788 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicXor(unsigned long long *__p,`。
- **L789 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L789 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L790 EN**: Returns from the current function with `__nvvm_atom_xor_gen_ll((long long *)__p, __v)`.
  **L790 CN**: 以 `__nvvm_atom_xor_gen_ll((long long *)__p, __v)` 从当前函数返回。
- **L791 EN**: Closes the current lexical scope or compound statement.
  **L791 CN**: 结束当前词法作用域或复合语句块。
- **L792 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicXor_block(unsigned long long *__p,`.
  **L792 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicXor_block(unsigned long long *__p,`。

### Lines 793-816

````c
                                                   unsigned long long __v) {
  return __nvvm_atom_cta_xor_gen_ll((long long *)__p, __v);
}
__DEVICE__ unsigned long long __ullAtomicXor_system(unsigned long long *__p,
                                                    unsigned long long __v) {
  return __nvvm_atom_sys_xor_gen_ll((long long *)__p, __v);
}
__DEVICE__ unsigned int __umul24(unsigned int __a, unsigned int __b) {
  return __nv_umul24(__a, __b);
}
__DEVICE__ unsigned long long __umul64hi(unsigned long long __a,
                                         unsigned long long __b) {
  return __nv_umul64hi(__a, __b);
}
__DEVICE__ unsigned int __umulhi(unsigned int __a, unsigned int __b) {
  return __nv_umulhi(__a, __b);
}
__DEVICE__ unsigned int __urhadd(unsigned int __a, unsigned int __b) {
  return __nv_urhadd(__a, __b);
}
__DEVICE__ unsigned int __usad(unsigned int __a, unsigned int __b,
                               unsigned int __c) {
  return __nv_usad(__a, __b, __c);
}
````
- **L793 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L793 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L794 EN**: Returns from the current function with `__nvvm_atom_cta_xor_gen_ll((long long *)__p, __v)`.
  **L794 CN**: 以 `__nvvm_atom_cta_xor_gen_ll((long long *)__p, __v)` 从当前函数返回。
- **L795 EN**: Closes the current lexical scope or compound statement.
  **L795 CN**: 结束当前词法作用域或复合语句块。
- **L796 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __ullAtomicXor_system(unsigned long long *__p,`.
  **L796 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __ullAtomicXor_system(unsigned long long *__p,`。
- **L797 EN**: Continues the surrounding expression or declaration: `unsigned long long __v) {`.
  **L797 CN**: 继续构造周围的表达式或声明：`unsigned long long __v) {`。
- **L798 EN**: Returns from the current function with `__nvvm_atom_sys_xor_gen_ll((long long *)__p, __v)`.
  **L798 CN**: 以 `__nvvm_atom_sys_xor_gen_ll((long long *)__p, __v)` 从当前函数返回。
- **L799 EN**: Closes the current lexical scope or compound statement.
  **L799 CN**: 结束当前词法作用域或复合语句块。
- **L800 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __umul24(unsigned int __a, unsigned int __b) {`.
  **L800 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __umul24(unsigned int __a, unsigned int __b) {`。
- **L801 EN**: Returns from the current function with `__nv_umul24(__a, __b)`.
  **L801 CN**: 以 `__nv_umul24(__a, __b)` 从当前函数返回。
- **L802 EN**: Closes the current lexical scope or compound statement.
  **L802 CN**: 结束当前词法作用域或复合语句块。
- **L803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __umul64hi(unsigned long long __a,`.
  **L803 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __umul64hi(unsigned long long __a,`。
- **L804 EN**: Continues the surrounding expression or declaration: `unsigned long long __b) {`.
  **L804 CN**: 继续构造周围的表达式或声明：`unsigned long long __b) {`。
- **L805 EN**: Returns from the current function with `__nv_umul64hi(__a, __b)`.
  **L805 CN**: 以 `__nv_umul64hi(__a, __b)` 从当前函数返回。
- **L806 EN**: Closes the current lexical scope or compound statement.
  **L806 CN**: 结束当前词法作用域或复合语句块。
- **L807 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __umulhi(unsigned int __a, unsigned int __b) {`.
  **L807 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __umulhi(unsigned int __a, unsigned int __b) {`。
- **L808 EN**: Returns from the current function with `__nv_umulhi(__a, __b)`.
  **L808 CN**: 以 `__nv_umulhi(__a, __b)` 从当前函数返回。
- **L809 EN**: Closes the current lexical scope or compound statement.
  **L809 CN**: 结束当前词法作用域或复合语句块。
- **L810 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __urhadd(unsigned int __a, unsigned int __b) {`.
  **L810 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __urhadd(unsigned int __a, unsigned int __b) {`。
- **L811 EN**: Returns from the current function with `__nv_urhadd(__a, __b)`.
  **L811 CN**: 以 `__nv_urhadd(__a, __b)` 从当前函数返回。
- **L812 EN**: Closes the current lexical scope or compound statement.
  **L812 CN**: 结束当前词法作用域或复合语句块。
- **L813 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __usad(unsigned int __a, unsigned int __b,`.
  **L813 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __usad(unsigned int __a, unsigned int __b,`。
- **L814 EN**: Continues the surrounding expression or declaration: `unsigned int __c) {`.
  **L814 CN**: 继续构造周围的表达式或声明：`unsigned int __c) {`。
- **L815 EN**: Returns from the current function with `__nv_usad(__a, __b, __c)`.
  **L815 CN**: 以 `__nv_usad(__a, __b, __c)` 从当前函数返回。
- **L816 EN**: Closes the current lexical scope or compound statement.
  **L816 CN**: 结束当前词法作用域或复合语句块。

### Lines 817-840

````c

#if CUDA_VERSION >= 9000 && CUDA_VERSION < 9020
__DEVICE__ unsigned int __vabs2(unsigned int __a) { return __nv_vabs2(__a); }
__DEVICE__ unsigned int __vabs4(unsigned int __a) { return __nv_vabs4(__a); }
__DEVICE__ unsigned int __vabsdiffs2(unsigned int __a, unsigned int __b) {
  return __nv_vabsdiffs2(__a, __b);
}
__DEVICE__ unsigned int __vabsdiffs4(unsigned int __a, unsigned int __b) {
  return __nv_vabsdiffs4(__a, __b);
}
__DEVICE__ unsigned int __vabsdiffu2(unsigned int __a, unsigned int __b) {
  return __nv_vabsdiffu2(__a, __b);
}
__DEVICE__ unsigned int __vabsdiffu4(unsigned int __a, unsigned int __b) {
  return __nv_vabsdiffu4(__a, __b);
}
__DEVICE__ unsigned int __vabsss2(unsigned int __a) {
  return __nv_vabsss2(__a);
}
__DEVICE__ unsigned int __vabsss4(unsigned int __a) {
  return __nv_vabsss4(__a);
}
__DEVICE__ unsigned int __vadd2(unsigned int __a, unsigned int __b) {
  return __nv_vadd2(__a, __b);
````
- **L817 EN**: Blank line separating nearby declarations or logic blocks.
  **L817 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L818 EN**: Starts a preprocessor conditional block: `#if CUDA_VERSION >= 9000 && CUDA_VERSION < 9020`.
  **L818 CN**: 开始一个预处理条件块：`#if CUDA_VERSION >= 9000 && CUDA_VERSION < 9020`。
- **L819 EN**: Continues logic associated with callable symbol `__vabs2`.
  **L819 CN**: 继续与可调用符号 `__vabs2` 相关的逻辑。
- **L820 EN**: Continues logic associated with callable symbol `__vabs4`.
  **L820 CN**: 继续与可调用符号 `__vabs4` 相关的逻辑。
- **L821 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vabsdiffs2(unsigned int __a, unsigned int __b) {`.
  **L821 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vabsdiffs2(unsigned int __a, unsigned int __b) {`。
- **L822 EN**: Returns from the current function with `__nv_vabsdiffs2(__a, __b)`.
  **L822 CN**: 以 `__nv_vabsdiffs2(__a, __b)` 从当前函数返回。
- **L823 EN**: Closes the current lexical scope or compound statement.
  **L823 CN**: 结束当前词法作用域或复合语句块。
- **L824 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vabsdiffs4(unsigned int __a, unsigned int __b) {`.
  **L824 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vabsdiffs4(unsigned int __a, unsigned int __b) {`。
- **L825 EN**: Returns from the current function with `__nv_vabsdiffs4(__a, __b)`.
  **L825 CN**: 以 `__nv_vabsdiffs4(__a, __b)` 从当前函数返回。
- **L826 EN**: Closes the current lexical scope or compound statement.
  **L826 CN**: 结束当前词法作用域或复合语句块。
- **L827 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vabsdiffu2(unsigned int __a, unsigned int __b) {`.
  **L827 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vabsdiffu2(unsigned int __a, unsigned int __b) {`。
- **L828 EN**: Returns from the current function with `__nv_vabsdiffu2(__a, __b)`.
  **L828 CN**: 以 `__nv_vabsdiffu2(__a, __b)` 从当前函数返回。
- **L829 EN**: Closes the current lexical scope or compound statement.
  **L829 CN**: 结束当前词法作用域或复合语句块。
- **L830 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vabsdiffu4(unsigned int __a, unsigned int __b) {`.
  **L830 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vabsdiffu4(unsigned int __a, unsigned int __b) {`。
- **L831 EN**: Returns from the current function with `__nv_vabsdiffu4(__a, __b)`.
  **L831 CN**: 以 `__nv_vabsdiffu4(__a, __b)` 从当前函数返回。
- **L832 EN**: Closes the current lexical scope or compound statement.
  **L832 CN**: 结束当前词法作用域或复合语句块。
- **L833 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vabsss2(unsigned int __a) {`.
  **L833 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vabsss2(unsigned int __a) {`。
- **L834 EN**: Returns from the current function with `__nv_vabsss2(__a)`.
  **L834 CN**: 以 `__nv_vabsss2(__a)` 从当前函数返回。
- **L835 EN**: Closes the current lexical scope or compound statement.
  **L835 CN**: 结束当前词法作用域或复合语句块。
- **L836 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vabsss4(unsigned int __a) {`.
  **L836 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vabsss4(unsigned int __a) {`。
- **L837 EN**: Returns from the current function with `__nv_vabsss4(__a)`.
  **L837 CN**: 以 `__nv_vabsss4(__a)` 从当前函数返回。
- **L838 EN**: Closes the current lexical scope or compound statement.
  **L838 CN**: 结束当前词法作用域或复合语句块。
- **L839 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vadd2(unsigned int __a, unsigned int __b) {`.
  **L839 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vadd2(unsigned int __a, unsigned int __b) {`。
- **L840 EN**: Returns from the current function with `__nv_vadd2(__a, __b)`.
  **L840 CN**: 以 `__nv_vadd2(__a, __b)` 从当前函数返回。

### Lines 841-864

````c
}
__DEVICE__ unsigned int __vadd4(unsigned int __a, unsigned int __b) {
  return __nv_vadd4(__a, __b);
}
__DEVICE__ unsigned int __vaddss2(unsigned int __a, unsigned int __b) {
  return __nv_vaddss2(__a, __b);
}
__DEVICE__ unsigned int __vaddss4(unsigned int __a, unsigned int __b) {
  return __nv_vaddss4(__a, __b);
}
__DEVICE__ unsigned int __vaddus2(unsigned int __a, unsigned int __b) {
  return __nv_vaddus2(__a, __b);
}
__DEVICE__ unsigned int __vaddus4(unsigned int __a, unsigned int __b) {
  return __nv_vaddus4(__a, __b);
}
__DEVICE__ unsigned int __vavgs2(unsigned int __a, unsigned int __b) {
  return __nv_vavgs2(__a, __b);
}
__DEVICE__ unsigned int __vavgs4(unsigned int __a, unsigned int __b) {
  return __nv_vavgs4(__a, __b);
}
__DEVICE__ unsigned int __vavgu2(unsigned int __a, unsigned int __b) {
  return __nv_vavgu2(__a, __b);
````
- **L841 EN**: Closes the current lexical scope or compound statement.
  **L841 CN**: 结束当前词法作用域或复合语句块。
- **L842 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vadd4(unsigned int __a, unsigned int __b) {`.
  **L842 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vadd4(unsigned int __a, unsigned int __b) {`。
- **L843 EN**: Returns from the current function with `__nv_vadd4(__a, __b)`.
  **L843 CN**: 以 `__nv_vadd4(__a, __b)` 从当前函数返回。
- **L844 EN**: Closes the current lexical scope or compound statement.
  **L844 CN**: 结束当前词法作用域或复合语句块。
- **L845 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vaddss2(unsigned int __a, unsigned int __b) {`.
  **L845 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vaddss2(unsigned int __a, unsigned int __b) {`。
- **L846 EN**: Returns from the current function with `__nv_vaddss2(__a, __b)`.
  **L846 CN**: 以 `__nv_vaddss2(__a, __b)` 从当前函数返回。
- **L847 EN**: Closes the current lexical scope or compound statement.
  **L847 CN**: 结束当前词法作用域或复合语句块。
- **L848 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vaddss4(unsigned int __a, unsigned int __b) {`.
  **L848 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vaddss4(unsigned int __a, unsigned int __b) {`。
- **L849 EN**: Returns from the current function with `__nv_vaddss4(__a, __b)`.
  **L849 CN**: 以 `__nv_vaddss4(__a, __b)` 从当前函数返回。
- **L850 EN**: Closes the current lexical scope or compound statement.
  **L850 CN**: 结束当前词法作用域或复合语句块。
- **L851 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vaddus2(unsigned int __a, unsigned int __b) {`.
  **L851 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vaddus2(unsigned int __a, unsigned int __b) {`。
- **L852 EN**: Returns from the current function with `__nv_vaddus2(__a, __b)`.
  **L852 CN**: 以 `__nv_vaddus2(__a, __b)` 从当前函数返回。
- **L853 EN**: Closes the current lexical scope or compound statement.
  **L853 CN**: 结束当前词法作用域或复合语句块。
- **L854 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vaddus4(unsigned int __a, unsigned int __b) {`.
  **L854 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vaddus4(unsigned int __a, unsigned int __b) {`。
- **L855 EN**: Returns from the current function with `__nv_vaddus4(__a, __b)`.
  **L855 CN**: 以 `__nv_vaddus4(__a, __b)` 从当前函数返回。
- **L856 EN**: Closes the current lexical scope or compound statement.
  **L856 CN**: 结束当前词法作用域或复合语句块。
- **L857 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vavgs2(unsigned int __a, unsigned int __b) {`.
  **L857 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vavgs2(unsigned int __a, unsigned int __b) {`。
- **L858 EN**: Returns from the current function with `__nv_vavgs2(__a, __b)`.
  **L858 CN**: 以 `__nv_vavgs2(__a, __b)` 从当前函数返回。
- **L859 EN**: Closes the current lexical scope or compound statement.
  **L859 CN**: 结束当前词法作用域或复合语句块。
- **L860 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vavgs4(unsigned int __a, unsigned int __b) {`.
  **L860 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vavgs4(unsigned int __a, unsigned int __b) {`。
- **L861 EN**: Returns from the current function with `__nv_vavgs4(__a, __b)`.
  **L861 CN**: 以 `__nv_vavgs4(__a, __b)` 从当前函数返回。
- **L862 EN**: Closes the current lexical scope or compound statement.
  **L862 CN**: 结束当前词法作用域或复合语句块。
- **L863 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vavgu2(unsigned int __a, unsigned int __b) {`.
  **L863 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vavgu2(unsigned int __a, unsigned int __b) {`。
- **L864 EN**: Returns from the current function with `__nv_vavgu2(__a, __b)`.
  **L864 CN**: 以 `__nv_vavgu2(__a, __b)` 从当前函数返回。

### Lines 865-888

````c
}
__DEVICE__ unsigned int __vavgu4(unsigned int __a, unsigned int __b) {
  return __nv_vavgu4(__a, __b);
}
__DEVICE__ unsigned int __vcmpeq2(unsigned int __a, unsigned int __b) {
  return __nv_vcmpeq2(__a, __b);
}
__DEVICE__ unsigned int __vcmpeq4(unsigned int __a, unsigned int __b) {
  return __nv_vcmpeq4(__a, __b);
}
__DEVICE__ unsigned int __vcmpges2(unsigned int __a, unsigned int __b) {
  return __nv_vcmpges2(__a, __b);
}
__DEVICE__ unsigned int __vcmpges4(unsigned int __a, unsigned int __b) {
  return __nv_vcmpges4(__a, __b);
}
__DEVICE__ unsigned int __vcmpgeu2(unsigned int __a, unsigned int __b) {
  return __nv_vcmpgeu2(__a, __b);
}
__DEVICE__ unsigned int __vcmpgeu4(unsigned int __a, unsigned int __b) {
  return __nv_vcmpgeu4(__a, __b);
}
__DEVICE__ unsigned int __vcmpgts2(unsigned int __a, unsigned int __b) {
  return __nv_vcmpgts2(__a, __b);
````
- **L865 EN**: Closes the current lexical scope or compound statement.
  **L865 CN**: 结束当前词法作用域或复合语句块。
- **L866 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vavgu4(unsigned int __a, unsigned int __b) {`.
  **L866 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vavgu4(unsigned int __a, unsigned int __b) {`。
- **L867 EN**: Returns from the current function with `__nv_vavgu4(__a, __b)`.
  **L867 CN**: 以 `__nv_vavgu4(__a, __b)` 从当前函数返回。
- **L868 EN**: Closes the current lexical scope or compound statement.
  **L868 CN**: 结束当前词法作用域或复合语句块。
- **L869 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpeq2(unsigned int __a, unsigned int __b) {`.
  **L869 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpeq2(unsigned int __a, unsigned int __b) {`。
- **L870 EN**: Returns from the current function with `__nv_vcmpeq2(__a, __b)`.
  **L870 CN**: 以 `__nv_vcmpeq2(__a, __b)` 从当前函数返回。
- **L871 EN**: Closes the current lexical scope or compound statement.
  **L871 CN**: 结束当前词法作用域或复合语句块。
- **L872 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpeq4(unsigned int __a, unsigned int __b) {`.
  **L872 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpeq4(unsigned int __a, unsigned int __b) {`。
- **L873 EN**: Returns from the current function with `__nv_vcmpeq4(__a, __b)`.
  **L873 CN**: 以 `__nv_vcmpeq4(__a, __b)` 从当前函数返回。
- **L874 EN**: Closes the current lexical scope or compound statement.
  **L874 CN**: 结束当前词法作用域或复合语句块。
- **L875 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpges2(unsigned int __a, unsigned int __b) {`.
  **L875 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpges2(unsigned int __a, unsigned int __b) {`。
- **L876 EN**: Returns from the current function with `__nv_vcmpges2(__a, __b)`.
  **L876 CN**: 以 `__nv_vcmpges2(__a, __b)` 从当前函数返回。
- **L877 EN**: Closes the current lexical scope or compound statement.
  **L877 CN**: 结束当前词法作用域或复合语句块。
- **L878 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpges4(unsigned int __a, unsigned int __b) {`.
  **L878 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpges4(unsigned int __a, unsigned int __b) {`。
- **L879 EN**: Returns from the current function with `__nv_vcmpges4(__a, __b)`.
  **L879 CN**: 以 `__nv_vcmpges4(__a, __b)` 从当前函数返回。
- **L880 EN**: Closes the current lexical scope or compound statement.
  **L880 CN**: 结束当前词法作用域或复合语句块。
- **L881 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpgeu2(unsigned int __a, unsigned int __b) {`.
  **L881 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpgeu2(unsigned int __a, unsigned int __b) {`。
- **L882 EN**: Returns from the current function with `__nv_vcmpgeu2(__a, __b)`.
  **L882 CN**: 以 `__nv_vcmpgeu2(__a, __b)` 从当前函数返回。
- **L883 EN**: Closes the current lexical scope or compound statement.
  **L883 CN**: 结束当前词法作用域或复合语句块。
- **L884 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpgeu4(unsigned int __a, unsigned int __b) {`.
  **L884 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpgeu4(unsigned int __a, unsigned int __b) {`。
- **L885 EN**: Returns from the current function with `__nv_vcmpgeu4(__a, __b)`.
  **L885 CN**: 以 `__nv_vcmpgeu4(__a, __b)` 从当前函数返回。
- **L886 EN**: Closes the current lexical scope or compound statement.
  **L886 CN**: 结束当前词法作用域或复合语句块。
- **L887 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpgts2(unsigned int __a, unsigned int __b) {`.
  **L887 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpgts2(unsigned int __a, unsigned int __b) {`。
- **L888 EN**: Returns from the current function with `__nv_vcmpgts2(__a, __b)`.
  **L888 CN**: 以 `__nv_vcmpgts2(__a, __b)` 从当前函数返回。

### Lines 889-912

````c
}
__DEVICE__ unsigned int __vcmpgts4(unsigned int __a, unsigned int __b) {
  return __nv_vcmpgts4(__a, __b);
}
__DEVICE__ unsigned int __vcmpgtu2(unsigned int __a, unsigned int __b) {
  return __nv_vcmpgtu2(__a, __b);
}
__DEVICE__ unsigned int __vcmpgtu4(unsigned int __a, unsigned int __b) {
  return __nv_vcmpgtu4(__a, __b);
}
__DEVICE__ unsigned int __vcmples2(unsigned int __a, unsigned int __b) {
  return __nv_vcmples2(__a, __b);
}
__DEVICE__ unsigned int __vcmples4(unsigned int __a, unsigned int __b) {
  return __nv_vcmples4(__a, __b);
}
__DEVICE__ unsigned int __vcmpleu2(unsigned int __a, unsigned int __b) {
  return __nv_vcmpleu2(__a, __b);
}
__DEVICE__ unsigned int __vcmpleu4(unsigned int __a, unsigned int __b) {
  return __nv_vcmpleu4(__a, __b);
}
__DEVICE__ unsigned int __vcmplts2(unsigned int __a, unsigned int __b) {
  return __nv_vcmplts2(__a, __b);
````
- **L889 EN**: Closes the current lexical scope or compound statement.
  **L889 CN**: 结束当前词法作用域或复合语句块。
- **L890 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpgts4(unsigned int __a, unsigned int __b) {`.
  **L890 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpgts4(unsigned int __a, unsigned int __b) {`。
- **L891 EN**: Returns from the current function with `__nv_vcmpgts4(__a, __b)`.
  **L891 CN**: 以 `__nv_vcmpgts4(__a, __b)` 从当前函数返回。
- **L892 EN**: Closes the current lexical scope or compound statement.
  **L892 CN**: 结束当前词法作用域或复合语句块。
- **L893 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpgtu2(unsigned int __a, unsigned int __b) {`.
  **L893 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpgtu2(unsigned int __a, unsigned int __b) {`。
- **L894 EN**: Returns from the current function with `__nv_vcmpgtu2(__a, __b)`.
  **L894 CN**: 以 `__nv_vcmpgtu2(__a, __b)` 从当前函数返回。
- **L895 EN**: Closes the current lexical scope or compound statement.
  **L895 CN**: 结束当前词法作用域或复合语句块。
- **L896 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpgtu4(unsigned int __a, unsigned int __b) {`.
  **L896 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpgtu4(unsigned int __a, unsigned int __b) {`。
- **L897 EN**: Returns from the current function with `__nv_vcmpgtu4(__a, __b)`.
  **L897 CN**: 以 `__nv_vcmpgtu4(__a, __b)` 从当前函数返回。
- **L898 EN**: Closes the current lexical scope or compound statement.
  **L898 CN**: 结束当前词法作用域或复合语句块。
- **L899 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmples2(unsigned int __a, unsigned int __b) {`.
  **L899 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmples2(unsigned int __a, unsigned int __b) {`。
- **L900 EN**: Returns from the current function with `__nv_vcmples2(__a, __b)`.
  **L900 CN**: 以 `__nv_vcmples2(__a, __b)` 从当前函数返回。
- **L901 EN**: Closes the current lexical scope or compound statement.
  **L901 CN**: 结束当前词法作用域或复合语句块。
- **L902 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmples4(unsigned int __a, unsigned int __b) {`.
  **L902 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmples4(unsigned int __a, unsigned int __b) {`。
- **L903 EN**: Returns from the current function with `__nv_vcmples4(__a, __b)`.
  **L903 CN**: 以 `__nv_vcmples4(__a, __b)` 从当前函数返回。
- **L904 EN**: Closes the current lexical scope or compound statement.
  **L904 CN**: 结束当前词法作用域或复合语句块。
- **L905 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpleu2(unsigned int __a, unsigned int __b) {`.
  **L905 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpleu2(unsigned int __a, unsigned int __b) {`。
- **L906 EN**: Returns from the current function with `__nv_vcmpleu2(__a, __b)`.
  **L906 CN**: 以 `__nv_vcmpleu2(__a, __b)` 从当前函数返回。
- **L907 EN**: Closes the current lexical scope or compound statement.
  **L907 CN**: 结束当前词法作用域或复合语句块。
- **L908 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpleu4(unsigned int __a, unsigned int __b) {`.
  **L908 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpleu4(unsigned int __a, unsigned int __b) {`。
- **L909 EN**: Returns from the current function with `__nv_vcmpleu4(__a, __b)`.
  **L909 CN**: 以 `__nv_vcmpleu4(__a, __b)` 从当前函数返回。
- **L910 EN**: Closes the current lexical scope or compound statement.
  **L910 CN**: 结束当前词法作用域或复合语句块。
- **L911 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmplts2(unsigned int __a, unsigned int __b) {`.
  **L911 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmplts2(unsigned int __a, unsigned int __b) {`。
- **L912 EN**: Returns from the current function with `__nv_vcmplts2(__a, __b)`.
  **L912 CN**: 以 `__nv_vcmplts2(__a, __b)` 从当前函数返回。

### Lines 913-936

````c
}
__DEVICE__ unsigned int __vcmplts4(unsigned int __a, unsigned int __b) {
  return __nv_vcmplts4(__a, __b);
}
__DEVICE__ unsigned int __vcmpltu2(unsigned int __a, unsigned int __b) {
  return __nv_vcmpltu2(__a, __b);
}
__DEVICE__ unsigned int __vcmpltu4(unsigned int __a, unsigned int __b) {
  return __nv_vcmpltu4(__a, __b);
}
__DEVICE__ unsigned int __vcmpne2(unsigned int __a, unsigned int __b) {
  return __nv_vcmpne2(__a, __b);
}
__DEVICE__ unsigned int __vcmpne4(unsigned int __a, unsigned int __b) {
  return __nv_vcmpne4(__a, __b);
}
__DEVICE__ unsigned int __vhaddu2(unsigned int __a, unsigned int __b) {
  return __nv_vhaddu2(__a, __b);
}
__DEVICE__ unsigned int __vhaddu4(unsigned int __a, unsigned int __b) {
  return __nv_vhaddu4(__a, __b);
}
__DEVICE__ unsigned int __vmaxs2(unsigned int __a, unsigned int __b) {
  return __nv_vmaxs2(__a, __b);
````
- **L913 EN**: Closes the current lexical scope or compound statement.
  **L913 CN**: 结束当前词法作用域或复合语句块。
- **L914 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmplts4(unsigned int __a, unsigned int __b) {`.
  **L914 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmplts4(unsigned int __a, unsigned int __b) {`。
- **L915 EN**: Returns from the current function with `__nv_vcmplts4(__a, __b)`.
  **L915 CN**: 以 `__nv_vcmplts4(__a, __b)` 从当前函数返回。
- **L916 EN**: Closes the current lexical scope or compound statement.
  **L916 CN**: 结束当前词法作用域或复合语句块。
- **L917 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpltu2(unsigned int __a, unsigned int __b) {`.
  **L917 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpltu2(unsigned int __a, unsigned int __b) {`。
- **L918 EN**: Returns from the current function with `__nv_vcmpltu2(__a, __b)`.
  **L918 CN**: 以 `__nv_vcmpltu2(__a, __b)` 从当前函数返回。
- **L919 EN**: Closes the current lexical scope or compound statement.
  **L919 CN**: 结束当前词法作用域或复合语句块。
- **L920 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpltu4(unsigned int __a, unsigned int __b) {`.
  **L920 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpltu4(unsigned int __a, unsigned int __b) {`。
- **L921 EN**: Returns from the current function with `__nv_vcmpltu4(__a, __b)`.
  **L921 CN**: 以 `__nv_vcmpltu4(__a, __b)` 从当前函数返回。
- **L922 EN**: Closes the current lexical scope or compound statement.
  **L922 CN**: 结束当前词法作用域或复合语句块。
- **L923 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpne2(unsigned int __a, unsigned int __b) {`.
  **L923 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpne2(unsigned int __a, unsigned int __b) {`。
- **L924 EN**: Returns from the current function with `__nv_vcmpne2(__a, __b)`.
  **L924 CN**: 以 `__nv_vcmpne2(__a, __b)` 从当前函数返回。
- **L925 EN**: Closes the current lexical scope or compound statement.
  **L925 CN**: 结束当前词法作用域或复合语句块。
- **L926 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpne4(unsigned int __a, unsigned int __b) {`.
  **L926 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpne4(unsigned int __a, unsigned int __b) {`。
- **L927 EN**: Returns from the current function with `__nv_vcmpne4(__a, __b)`.
  **L927 CN**: 以 `__nv_vcmpne4(__a, __b)` 从当前函数返回。
- **L928 EN**: Closes the current lexical scope or compound statement.
  **L928 CN**: 结束当前词法作用域或复合语句块。
- **L929 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vhaddu2(unsigned int __a, unsigned int __b) {`.
  **L929 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vhaddu2(unsigned int __a, unsigned int __b) {`。
- **L930 EN**: Returns from the current function with `__nv_vhaddu2(__a, __b)`.
  **L930 CN**: 以 `__nv_vhaddu2(__a, __b)` 从当前函数返回。
- **L931 EN**: Closes the current lexical scope or compound statement.
  **L931 CN**: 结束当前词法作用域或复合语句块。
- **L932 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vhaddu4(unsigned int __a, unsigned int __b) {`.
  **L932 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vhaddu4(unsigned int __a, unsigned int __b) {`。
- **L933 EN**: Returns from the current function with `__nv_vhaddu4(__a, __b)`.
  **L933 CN**: 以 `__nv_vhaddu4(__a, __b)` 从当前函数返回。
- **L934 EN**: Closes the current lexical scope or compound statement.
  **L934 CN**: 结束当前词法作用域或复合语句块。
- **L935 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vmaxs2(unsigned int __a, unsigned int __b) {`.
  **L935 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vmaxs2(unsigned int __a, unsigned int __b) {`。
- **L936 EN**: Returns from the current function with `__nv_vmaxs2(__a, __b)`.
  **L936 CN**: 以 `__nv_vmaxs2(__a, __b)` 从当前函数返回。

### Lines 937-960

````c
}
__DEVICE__ unsigned int __vmaxs4(unsigned int __a, unsigned int __b) {
  return __nv_vmaxs4(__a, __b);
}
__DEVICE__ unsigned int __vmaxu2(unsigned int __a, unsigned int __b) {
  return __nv_vmaxu2(__a, __b);
}
__DEVICE__ unsigned int __vmaxu4(unsigned int __a, unsigned int __b) {
  return __nv_vmaxu4(__a, __b);
}
__DEVICE__ unsigned int __vmins2(unsigned int __a, unsigned int __b) {
  return __nv_vmins2(__a, __b);
}
__DEVICE__ unsigned int __vmins4(unsigned int __a, unsigned int __b) {
  return __nv_vmins4(__a, __b);
}
__DEVICE__ unsigned int __vminu2(unsigned int __a, unsigned int __b) {
  return __nv_vminu2(__a, __b);
}
__DEVICE__ unsigned int __vminu4(unsigned int __a, unsigned int __b) {
  return __nv_vminu4(__a, __b);
}
__DEVICE__ unsigned int __vneg2(unsigned int __a) { return __nv_vneg2(__a); }
__DEVICE__ unsigned int __vneg4(unsigned int __a) { return __nv_vneg4(__a); }
````
- **L937 EN**: Closes the current lexical scope or compound statement.
  **L937 CN**: 结束当前词法作用域或复合语句块。
- **L938 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vmaxs4(unsigned int __a, unsigned int __b) {`.
  **L938 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vmaxs4(unsigned int __a, unsigned int __b) {`。
- **L939 EN**: Returns from the current function with `__nv_vmaxs4(__a, __b)`.
  **L939 CN**: 以 `__nv_vmaxs4(__a, __b)` 从当前函数返回。
- **L940 EN**: Closes the current lexical scope or compound statement.
  **L940 CN**: 结束当前词法作用域或复合语句块。
- **L941 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vmaxu2(unsigned int __a, unsigned int __b) {`.
  **L941 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vmaxu2(unsigned int __a, unsigned int __b) {`。
- **L942 EN**: Returns from the current function with `__nv_vmaxu2(__a, __b)`.
  **L942 CN**: 以 `__nv_vmaxu2(__a, __b)` 从当前函数返回。
- **L943 EN**: Closes the current lexical scope or compound statement.
  **L943 CN**: 结束当前词法作用域或复合语句块。
- **L944 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vmaxu4(unsigned int __a, unsigned int __b) {`.
  **L944 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vmaxu4(unsigned int __a, unsigned int __b) {`。
- **L945 EN**: Returns from the current function with `__nv_vmaxu4(__a, __b)`.
  **L945 CN**: 以 `__nv_vmaxu4(__a, __b)` 从当前函数返回。
- **L946 EN**: Closes the current lexical scope or compound statement.
  **L946 CN**: 结束当前词法作用域或复合语句块。
- **L947 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vmins2(unsigned int __a, unsigned int __b) {`.
  **L947 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vmins2(unsigned int __a, unsigned int __b) {`。
- **L948 EN**: Returns from the current function with `__nv_vmins2(__a, __b)`.
  **L948 CN**: 以 `__nv_vmins2(__a, __b)` 从当前函数返回。
- **L949 EN**: Closes the current lexical scope or compound statement.
  **L949 CN**: 结束当前词法作用域或复合语句块。
- **L950 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vmins4(unsigned int __a, unsigned int __b) {`.
  **L950 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vmins4(unsigned int __a, unsigned int __b) {`。
- **L951 EN**: Returns from the current function with `__nv_vmins4(__a, __b)`.
  **L951 CN**: 以 `__nv_vmins4(__a, __b)` 从当前函数返回。
- **L952 EN**: Closes the current lexical scope or compound statement.
  **L952 CN**: 结束当前词法作用域或复合语句块。
- **L953 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vminu2(unsigned int __a, unsigned int __b) {`.
  **L953 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vminu2(unsigned int __a, unsigned int __b) {`。
- **L954 EN**: Returns from the current function with `__nv_vminu2(__a, __b)`.
  **L954 CN**: 以 `__nv_vminu2(__a, __b)` 从当前函数返回。
- **L955 EN**: Closes the current lexical scope or compound statement.
  **L955 CN**: 结束当前词法作用域或复合语句块。
- **L956 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vminu4(unsigned int __a, unsigned int __b) {`.
  **L956 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vminu4(unsigned int __a, unsigned int __b) {`。
- **L957 EN**: Returns from the current function with `__nv_vminu4(__a, __b)`.
  **L957 CN**: 以 `__nv_vminu4(__a, __b)` 从当前函数返回。
- **L958 EN**: Closes the current lexical scope or compound statement.
  **L958 CN**: 结束当前词法作用域或复合语句块。
- **L959 EN**: Continues logic associated with callable symbol `__vneg2`.
  **L959 CN**: 继续与可调用符号 `__vneg2` 相关的逻辑。
- **L960 EN**: Continues logic associated with callable symbol `__vneg4`.
  **L960 CN**: 继续与可调用符号 `__vneg4` 相关的逻辑。

### Lines 961-984

````c
__DEVICE__ unsigned int __vnegss2(unsigned int __a) {
  return __nv_vnegss2(__a);
}
__DEVICE__ unsigned int __vnegss4(unsigned int __a) {
  return __nv_vnegss4(__a);
}
__DEVICE__ unsigned int __vsads2(unsigned int __a, unsigned int __b) {
  return __nv_vsads2(__a, __b);
}
__DEVICE__ unsigned int __vsads4(unsigned int __a, unsigned int __b) {
  return __nv_vsads4(__a, __b);
}
__DEVICE__ unsigned int __vsadu2(unsigned int __a, unsigned int __b) {
  return __nv_vsadu2(__a, __b);
}
__DEVICE__ unsigned int __vsadu4(unsigned int __a, unsigned int __b) {
  return __nv_vsadu4(__a, __b);
}
__DEVICE__ unsigned int __vseteq2(unsigned int __a, unsigned int __b) {
  return __nv_vseteq2(__a, __b);
}
__DEVICE__ unsigned int __vseteq4(unsigned int __a, unsigned int __b) {
  return __nv_vseteq4(__a, __b);
}
````
- **L961 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vnegss2(unsigned int __a) {`.
  **L961 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vnegss2(unsigned int __a) {`。
- **L962 EN**: Returns from the current function with `__nv_vnegss2(__a)`.
  **L962 CN**: 以 `__nv_vnegss2(__a)` 从当前函数返回。
- **L963 EN**: Closes the current lexical scope or compound statement.
  **L963 CN**: 结束当前词法作用域或复合语句块。
- **L964 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vnegss4(unsigned int __a) {`.
  **L964 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vnegss4(unsigned int __a) {`。
- **L965 EN**: Returns from the current function with `__nv_vnegss4(__a)`.
  **L965 CN**: 以 `__nv_vnegss4(__a)` 从当前函数返回。
- **L966 EN**: Closes the current lexical scope or compound statement.
  **L966 CN**: 结束当前词法作用域或复合语句块。
- **L967 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsads2(unsigned int __a, unsigned int __b) {`.
  **L967 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsads2(unsigned int __a, unsigned int __b) {`。
- **L968 EN**: Returns from the current function with `__nv_vsads2(__a, __b)`.
  **L968 CN**: 以 `__nv_vsads2(__a, __b)` 从当前函数返回。
- **L969 EN**: Closes the current lexical scope or compound statement.
  **L969 CN**: 结束当前词法作用域或复合语句块。
- **L970 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsads4(unsigned int __a, unsigned int __b) {`.
  **L970 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsads4(unsigned int __a, unsigned int __b) {`。
- **L971 EN**: Returns from the current function with `__nv_vsads4(__a, __b)`.
  **L971 CN**: 以 `__nv_vsads4(__a, __b)` 从当前函数返回。
- **L972 EN**: Closes the current lexical scope or compound statement.
  **L972 CN**: 结束当前词法作用域或复合语句块。
- **L973 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsadu2(unsigned int __a, unsigned int __b) {`.
  **L973 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsadu2(unsigned int __a, unsigned int __b) {`。
- **L974 EN**: Returns from the current function with `__nv_vsadu2(__a, __b)`.
  **L974 CN**: 以 `__nv_vsadu2(__a, __b)` 从当前函数返回。
- **L975 EN**: Closes the current lexical scope or compound statement.
  **L975 CN**: 结束当前词法作用域或复合语句块。
- **L976 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsadu4(unsigned int __a, unsigned int __b) {`.
  **L976 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsadu4(unsigned int __a, unsigned int __b) {`。
- **L977 EN**: Returns from the current function with `__nv_vsadu4(__a, __b)`.
  **L977 CN**: 以 `__nv_vsadu4(__a, __b)` 从当前函数返回。
- **L978 EN**: Closes the current lexical scope or compound statement.
  **L978 CN**: 结束当前词法作用域或复合语句块。
- **L979 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vseteq2(unsigned int __a, unsigned int __b) {`.
  **L979 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vseteq2(unsigned int __a, unsigned int __b) {`。
- **L980 EN**: Returns from the current function with `__nv_vseteq2(__a, __b)`.
  **L980 CN**: 以 `__nv_vseteq2(__a, __b)` 从当前函数返回。
- **L981 EN**: Closes the current lexical scope or compound statement.
  **L981 CN**: 结束当前词法作用域或复合语句块。
- **L982 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vseteq4(unsigned int __a, unsigned int __b) {`.
  **L982 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vseteq4(unsigned int __a, unsigned int __b) {`。
- **L983 EN**: Returns from the current function with `__nv_vseteq4(__a, __b)`.
  **L983 CN**: 以 `__nv_vseteq4(__a, __b)` 从当前函数返回。
- **L984 EN**: Closes the current lexical scope or compound statement.
  **L984 CN**: 结束当前词法作用域或复合语句块。

### Lines 985-1008

````c
__DEVICE__ unsigned int __vsetges2(unsigned int __a, unsigned int __b) {
  return __nv_vsetges2(__a, __b);
}
__DEVICE__ unsigned int __vsetges4(unsigned int __a, unsigned int __b) {
  return __nv_vsetges4(__a, __b);
}
__DEVICE__ unsigned int __vsetgeu2(unsigned int __a, unsigned int __b) {
  return __nv_vsetgeu2(__a, __b);
}
__DEVICE__ unsigned int __vsetgeu4(unsigned int __a, unsigned int __b) {
  return __nv_vsetgeu4(__a, __b);
}
__DEVICE__ unsigned int __vsetgts2(unsigned int __a, unsigned int __b) {
  return __nv_vsetgts2(__a, __b);
}
__DEVICE__ unsigned int __vsetgts4(unsigned int __a, unsigned int __b) {
  return __nv_vsetgts4(__a, __b);
}
__DEVICE__ unsigned int __vsetgtu2(unsigned int __a, unsigned int __b) {
  return __nv_vsetgtu2(__a, __b);
}
__DEVICE__ unsigned int __vsetgtu4(unsigned int __a, unsigned int __b) {
  return __nv_vsetgtu4(__a, __b);
}
````
- **L985 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetges2(unsigned int __a, unsigned int __b) {`.
  **L985 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetges2(unsigned int __a, unsigned int __b) {`。
- **L986 EN**: Returns from the current function with `__nv_vsetges2(__a, __b)`.
  **L986 CN**: 以 `__nv_vsetges2(__a, __b)` 从当前函数返回。
- **L987 EN**: Closes the current lexical scope or compound statement.
  **L987 CN**: 结束当前词法作用域或复合语句块。
- **L988 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetges4(unsigned int __a, unsigned int __b) {`.
  **L988 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetges4(unsigned int __a, unsigned int __b) {`。
- **L989 EN**: Returns from the current function with `__nv_vsetges4(__a, __b)`.
  **L989 CN**: 以 `__nv_vsetges4(__a, __b)` 从当前函数返回。
- **L990 EN**: Closes the current lexical scope or compound statement.
  **L990 CN**: 结束当前词法作用域或复合语句块。
- **L991 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetgeu2(unsigned int __a, unsigned int __b) {`.
  **L991 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetgeu2(unsigned int __a, unsigned int __b) {`。
- **L992 EN**: Returns from the current function with `__nv_vsetgeu2(__a, __b)`.
  **L992 CN**: 以 `__nv_vsetgeu2(__a, __b)` 从当前函数返回。
- **L993 EN**: Closes the current lexical scope or compound statement.
  **L993 CN**: 结束当前词法作用域或复合语句块。
- **L994 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetgeu4(unsigned int __a, unsigned int __b) {`.
  **L994 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetgeu4(unsigned int __a, unsigned int __b) {`。
- **L995 EN**: Returns from the current function with `__nv_vsetgeu4(__a, __b)`.
  **L995 CN**: 以 `__nv_vsetgeu4(__a, __b)` 从当前函数返回。
- **L996 EN**: Closes the current lexical scope or compound statement.
  **L996 CN**: 结束当前词法作用域或复合语句块。
- **L997 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetgts2(unsigned int __a, unsigned int __b) {`.
  **L997 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetgts2(unsigned int __a, unsigned int __b) {`。
- **L998 EN**: Returns from the current function with `__nv_vsetgts2(__a, __b)`.
  **L998 CN**: 以 `__nv_vsetgts2(__a, __b)` 从当前函数返回。
- **L999 EN**: Closes the current lexical scope or compound statement.
  **L999 CN**: 结束当前词法作用域或复合语句块。
- **L1000 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetgts4(unsigned int __a, unsigned int __b) {`.
  **L1000 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetgts4(unsigned int __a, unsigned int __b) {`。
- **L1001 EN**: Returns from the current function with `__nv_vsetgts4(__a, __b)`.
  **L1001 CN**: 以 `__nv_vsetgts4(__a, __b)` 从当前函数返回。
- **L1002 EN**: Closes the current lexical scope or compound statement.
  **L1002 CN**: 结束当前词法作用域或复合语句块。
- **L1003 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetgtu2(unsigned int __a, unsigned int __b) {`.
  **L1003 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetgtu2(unsigned int __a, unsigned int __b) {`。
- **L1004 EN**: Returns from the current function with `__nv_vsetgtu2(__a, __b)`.
  **L1004 CN**: 以 `__nv_vsetgtu2(__a, __b)` 从当前函数返回。
- **L1005 EN**: Closes the current lexical scope or compound statement.
  **L1005 CN**: 结束当前词法作用域或复合语句块。
- **L1006 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetgtu4(unsigned int __a, unsigned int __b) {`.
  **L1006 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetgtu4(unsigned int __a, unsigned int __b) {`。
- **L1007 EN**: Returns from the current function with `__nv_vsetgtu4(__a, __b)`.
  **L1007 CN**: 以 `__nv_vsetgtu4(__a, __b)` 从当前函数返回。
- **L1008 EN**: Closes the current lexical scope or compound statement.
  **L1008 CN**: 结束当前词法作用域或复合语句块。

### Lines 1009-1032

````c
__DEVICE__ unsigned int __vsetles2(unsigned int __a, unsigned int __b) {
  return __nv_vsetles2(__a, __b);
}
__DEVICE__ unsigned int __vsetles4(unsigned int __a, unsigned int __b) {
  return __nv_vsetles4(__a, __b);
}
__DEVICE__ unsigned int __vsetleu2(unsigned int __a, unsigned int __b) {
  return __nv_vsetleu2(__a, __b);
}
__DEVICE__ unsigned int __vsetleu4(unsigned int __a, unsigned int __b) {
  return __nv_vsetleu4(__a, __b);
}
__DEVICE__ unsigned int __vsetlts2(unsigned int __a, unsigned int __b) {
  return __nv_vsetlts2(__a, __b);
}
__DEVICE__ unsigned int __vsetlts4(unsigned int __a, unsigned int __b) {
  return __nv_vsetlts4(__a, __b);
}
__DEVICE__ unsigned int __vsetltu2(unsigned int __a, unsigned int __b) {
  return __nv_vsetltu2(__a, __b);
}
__DEVICE__ unsigned int __vsetltu4(unsigned int __a, unsigned int __b) {
  return __nv_vsetltu4(__a, __b);
}
````
- **L1009 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetles2(unsigned int __a, unsigned int __b) {`.
  **L1009 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetles2(unsigned int __a, unsigned int __b) {`。
- **L1010 EN**: Returns from the current function with `__nv_vsetles2(__a, __b)`.
  **L1010 CN**: 以 `__nv_vsetles2(__a, __b)` 从当前函数返回。
- **L1011 EN**: Closes the current lexical scope or compound statement.
  **L1011 CN**: 结束当前词法作用域或复合语句块。
- **L1012 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetles4(unsigned int __a, unsigned int __b) {`.
  **L1012 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetles4(unsigned int __a, unsigned int __b) {`。
- **L1013 EN**: Returns from the current function with `__nv_vsetles4(__a, __b)`.
  **L1013 CN**: 以 `__nv_vsetles4(__a, __b)` 从当前函数返回。
- **L1014 EN**: Closes the current lexical scope or compound statement.
  **L1014 CN**: 结束当前词法作用域或复合语句块。
- **L1015 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetleu2(unsigned int __a, unsigned int __b) {`.
  **L1015 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetleu2(unsigned int __a, unsigned int __b) {`。
- **L1016 EN**: Returns from the current function with `__nv_vsetleu2(__a, __b)`.
  **L1016 CN**: 以 `__nv_vsetleu2(__a, __b)` 从当前函数返回。
- **L1017 EN**: Closes the current lexical scope or compound statement.
  **L1017 CN**: 结束当前词法作用域或复合语句块。
- **L1018 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetleu4(unsigned int __a, unsigned int __b) {`.
  **L1018 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetleu4(unsigned int __a, unsigned int __b) {`。
- **L1019 EN**: Returns from the current function with `__nv_vsetleu4(__a, __b)`.
  **L1019 CN**: 以 `__nv_vsetleu4(__a, __b)` 从当前函数返回。
- **L1020 EN**: Closes the current lexical scope or compound statement.
  **L1020 CN**: 结束当前词法作用域或复合语句块。
- **L1021 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetlts2(unsigned int __a, unsigned int __b) {`.
  **L1021 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetlts2(unsigned int __a, unsigned int __b) {`。
- **L1022 EN**: Returns from the current function with `__nv_vsetlts2(__a, __b)`.
  **L1022 CN**: 以 `__nv_vsetlts2(__a, __b)` 从当前函数返回。
- **L1023 EN**: Closes the current lexical scope or compound statement.
  **L1023 CN**: 结束当前词法作用域或复合语句块。
- **L1024 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetlts4(unsigned int __a, unsigned int __b) {`.
  **L1024 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetlts4(unsigned int __a, unsigned int __b) {`。
- **L1025 EN**: Returns from the current function with `__nv_vsetlts4(__a, __b)`.
  **L1025 CN**: 以 `__nv_vsetlts4(__a, __b)` 从当前函数返回。
- **L1026 EN**: Closes the current lexical scope or compound statement.
  **L1026 CN**: 结束当前词法作用域或复合语句块。
- **L1027 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetltu2(unsigned int __a, unsigned int __b) {`.
  **L1027 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetltu2(unsigned int __a, unsigned int __b) {`。
- **L1028 EN**: Returns from the current function with `__nv_vsetltu2(__a, __b)`.
  **L1028 CN**: 以 `__nv_vsetltu2(__a, __b)` 从当前函数返回。
- **L1029 EN**: Closes the current lexical scope or compound statement.
  **L1029 CN**: 结束当前词法作用域或复合语句块。
- **L1030 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetltu4(unsigned int __a, unsigned int __b) {`.
  **L1030 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetltu4(unsigned int __a, unsigned int __b) {`。
- **L1031 EN**: Returns from the current function with `__nv_vsetltu4(__a, __b)`.
  **L1031 CN**: 以 `__nv_vsetltu4(__a, __b)` 从当前函数返回。
- **L1032 EN**: Closes the current lexical scope or compound statement.
  **L1032 CN**: 结束当前词法作用域或复合语句块。

### Lines 1033-1056

````c
__DEVICE__ unsigned int __vsetne2(unsigned int __a, unsigned int __b) {
  return __nv_vsetne2(__a, __b);
}
__DEVICE__ unsigned int __vsetne4(unsigned int __a, unsigned int __b) {
  return __nv_vsetne4(__a, __b);
}
__DEVICE__ unsigned int __vsub2(unsigned int __a, unsigned int __b) {
  return __nv_vsub2(__a, __b);
}
__DEVICE__ unsigned int __vsub4(unsigned int __a, unsigned int __b) {
  return __nv_vsub4(__a, __b);
}
__DEVICE__ unsigned int __vsubss2(unsigned int __a, unsigned int __b) {
  return __nv_vsubss2(__a, __b);
}
__DEVICE__ unsigned int __vsubss4(unsigned int __a, unsigned int __b) {
  return __nv_vsubss4(__a, __b);
}
__DEVICE__ unsigned int __vsubus2(unsigned int __a, unsigned int __b) {
  return __nv_vsubus2(__a, __b);
}
__DEVICE__ unsigned int __vsubus4(unsigned int __a, unsigned int __b) {
  return __nv_vsubus4(__a, __b);
}
````
- **L1033 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetne2(unsigned int __a, unsigned int __b) {`.
  **L1033 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetne2(unsigned int __a, unsigned int __b) {`。
- **L1034 EN**: Returns from the current function with `__nv_vsetne2(__a, __b)`.
  **L1034 CN**: 以 `__nv_vsetne2(__a, __b)` 从当前函数返回。
- **L1035 EN**: Closes the current lexical scope or compound statement.
  **L1035 CN**: 结束当前词法作用域或复合语句块。
- **L1036 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetne4(unsigned int __a, unsigned int __b) {`.
  **L1036 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetne4(unsigned int __a, unsigned int __b) {`。
- **L1037 EN**: Returns from the current function with `__nv_vsetne4(__a, __b)`.
  **L1037 CN**: 以 `__nv_vsetne4(__a, __b)` 从当前函数返回。
- **L1038 EN**: Closes the current lexical scope or compound statement.
  **L1038 CN**: 结束当前词法作用域或复合语句块。
- **L1039 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsub2(unsigned int __a, unsigned int __b) {`.
  **L1039 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsub2(unsigned int __a, unsigned int __b) {`。
- **L1040 EN**: Returns from the current function with `__nv_vsub2(__a, __b)`.
  **L1040 CN**: 以 `__nv_vsub2(__a, __b)` 从当前函数返回。
- **L1041 EN**: Closes the current lexical scope or compound statement.
  **L1041 CN**: 结束当前词法作用域或复合语句块。
- **L1042 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsub4(unsigned int __a, unsigned int __b) {`.
  **L1042 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsub4(unsigned int __a, unsigned int __b) {`。
- **L1043 EN**: Returns from the current function with `__nv_vsub4(__a, __b)`.
  **L1043 CN**: 以 `__nv_vsub4(__a, __b)` 从当前函数返回。
- **L1044 EN**: Closes the current lexical scope or compound statement.
  **L1044 CN**: 结束当前词法作用域或复合语句块。
- **L1045 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsubss2(unsigned int __a, unsigned int __b) {`.
  **L1045 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsubss2(unsigned int __a, unsigned int __b) {`。
- **L1046 EN**: Returns from the current function with `__nv_vsubss2(__a, __b)`.
  **L1046 CN**: 以 `__nv_vsubss2(__a, __b)` 从当前函数返回。
- **L1047 EN**: Closes the current lexical scope or compound statement.
  **L1047 CN**: 结束当前词法作用域或复合语句块。
- **L1048 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsubss4(unsigned int __a, unsigned int __b) {`.
  **L1048 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsubss4(unsigned int __a, unsigned int __b) {`。
- **L1049 EN**: Returns from the current function with `__nv_vsubss4(__a, __b)`.
  **L1049 CN**: 以 `__nv_vsubss4(__a, __b)` 从当前函数返回。
- **L1050 EN**: Closes the current lexical scope or compound statement.
  **L1050 CN**: 结束当前词法作用域或复合语句块。
- **L1051 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsubus2(unsigned int __a, unsigned int __b) {`.
  **L1051 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsubus2(unsigned int __a, unsigned int __b) {`。
- **L1052 EN**: Returns from the current function with `__nv_vsubus2(__a, __b)`.
  **L1052 CN**: 以 `__nv_vsubus2(__a, __b)` 从当前函数返回。
- **L1053 EN**: Closes the current lexical scope or compound statement.
  **L1053 CN**: 结束当前词法作用域或复合语句块。
- **L1054 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsubus4(unsigned int __a, unsigned int __b) {`.
  **L1054 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsubus4(unsigned int __a, unsigned int __b) {`。
- **L1055 EN**: Returns from the current function with `__nv_vsubus4(__a, __b)`.
  **L1055 CN**: 以 `__nv_vsubus4(__a, __b)` 从当前函数返回。
- **L1056 EN**: Closes the current lexical scope or compound statement.
  **L1056 CN**: 结束当前词法作用域或复合语句块。

### Lines 1057-1080

````c
#else // CUDA_VERSION >= 9020
// CUDA no longer provides inline assembly (or bitcode) implementation of these
// functions, so we have to reimplment them. The implementation is naive and is
// not optimized for performance.

// Helper function to convert N-bit boolean subfields into all-0 or all-1.
// E.g. __bool2mask(0x01000100,8) -> 0xff00ff00
//      __bool2mask(0x00010000,16) -> 0xffff0000
__DEVICE__ unsigned int __bool2mask(unsigned int __a, int shift) {
  return (__a << shift) - __a;
}
__DEVICE__ unsigned int __vabs2(unsigned int __a) {
  unsigned int r;
  __asm__("vabsdiff2.s32.s32.s32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(0), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vabs4(unsigned int __a) {
  unsigned int r;
  __asm__("vabsdiff4.s32.s32.s32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(0), "r"(0));
  return r;
````
- **L1057 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L1057 CN**: 继续为当前目标或特性集合选择预处理分支。
- **L1058 EN**: Comment explains nearby logic, constraints, or intent: `CUDA no longer provides inline assembly (or bitcode) implementation of these`.
  **L1058 CN**: 注释解释附近代码的逻辑、约束或设计意图：`CUDA no longer provides inline assembly (or bitcode) implementation of these`。
- **L1059 EN**: Comment explains nearby logic, constraints, or intent: `functions, so we have to reimplment them. The implementation is naive and is`.
  **L1059 CN**: 注释解释附近代码的逻辑、约束或设计意图：`functions, so we have to reimplment them. The implementation is naive and is`。
- **L1060 EN**: Comment explains nearby logic, constraints, or intent: `not optimized for performance.`.
  **L1060 CN**: 注释解释附近代码的逻辑、约束或设计意图：`not optimized for performance.`。
- **L1061 EN**: Blank line separating nearby declarations or logic blocks.
  **L1061 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1062 EN**: Comment explains nearby logic, constraints, or intent: `Helper function to convert N-bit boolean subfields into all-0 or all-1.`.
  **L1062 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Helper function to convert N-bit boolean subfields into all-0 or all-1.`。
- **L1063 EN**: Comment explains nearby logic, constraints, or intent: `E.g. __bool2mask(0x01000100,8) -> 0xff00ff00`.
  **L1063 CN**: 注释解释附近代码的逻辑、约束或设计意图：`E.g. __bool2mask(0x01000100,8) -> 0xff00ff00`。
- **L1064 EN**: Comment explains nearby logic, constraints, or intent: `__bool2mask(0x00010000,16) -> 0xffff0000`.
  **L1064 CN**: 注释解释附近代码的逻辑、约束或设计意图：`__bool2mask(0x00010000,16) -> 0xffff0000`。
- **L1065 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __bool2mask(unsigned int __a, int shift) {`.
  **L1065 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __bool2mask(unsigned int __a, int shift) {`。
- **L1066 EN**: Returns from the current function with `(__a << shift) - __a`.
  **L1066 CN**: 以 `(__a << shift) - __a` 从当前函数返回。
- **L1067 EN**: Closes the current lexical scope or compound statement.
  **L1067 CN**: 结束当前词法作用域或复合语句块。
- **L1068 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vabs2(unsigned int __a) {`.
  **L1068 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vabs2(unsigned int __a) {`。
- **L1069 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1069 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1070 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1070 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1071 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1071 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1072 EN**: Executes a call or declaration centered on `"r"`.
  **L1072 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1073 EN**: Returns from the current function with `r`.
  **L1073 CN**: 以 `r` 从当前函数返回。
- **L1074 EN**: Closes the current lexical scope or compound statement.
  **L1074 CN**: 结束当前词法作用域或复合语句块。
- **L1075 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vabs4(unsigned int __a) {`.
  **L1075 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vabs4(unsigned int __a) {`。
- **L1076 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1076 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1077 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1077 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1078 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1078 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1079 EN**: Executes a call or declaration centered on `"r"`.
  **L1079 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1080 EN**: Returns from the current function with `r`.
  **L1080 CN**: 以 `r` 从当前函数返回。

### Lines 1081-1104

````c
}
__DEVICE__ unsigned int __vabsdiffs2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vabsdiff2.s32.s32.s32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}

__DEVICE__ unsigned int __vabsdiffs4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vabsdiff4.s32.s32.s32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vabsdiffu2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vabsdiff2.u32.u32.u32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vabsdiffu4(unsigned int __a, unsigned int __b) {
````
- **L1081 EN**: Closes the current lexical scope or compound statement.
  **L1081 CN**: 结束当前词法作用域或复合语句块。
- **L1082 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vabsdiffs2(unsigned int __a, unsigned int __b) {`.
  **L1082 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vabsdiffs2(unsigned int __a, unsigned int __b) {`。
- **L1083 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1083 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1084 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1084 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1085 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1085 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1086 EN**: Executes a call or declaration centered on `"r"`.
  **L1086 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1087 EN**: Returns from the current function with `r`.
  **L1087 CN**: 以 `r` 从当前函数返回。
- **L1088 EN**: Closes the current lexical scope or compound statement.
  **L1088 CN**: 结束当前词法作用域或复合语句块。
- **L1089 EN**: Blank line separating nearby declarations or logic blocks.
  **L1089 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1090 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vabsdiffs4(unsigned int __a, unsigned int __b) {`.
  **L1090 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vabsdiffs4(unsigned int __a, unsigned int __b) {`。
- **L1091 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1091 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1092 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1092 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1093 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1093 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1094 EN**: Executes a call or declaration centered on `"r"`.
  **L1094 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1095 EN**: Returns from the current function with `r`.
  **L1095 CN**: 以 `r` 从当前函数返回。
- **L1096 EN**: Closes the current lexical scope or compound statement.
  **L1096 CN**: 结束当前词法作用域或复合语句块。
- **L1097 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vabsdiffu2(unsigned int __a, unsigned int __b) {`.
  **L1097 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vabsdiffu2(unsigned int __a, unsigned int __b) {`。
- **L1098 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1098 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1099 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1099 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1100 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1100 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1101 EN**: Executes a call or declaration centered on `"r"`.
  **L1101 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1102 EN**: Returns from the current function with `r`.
  **L1102 CN**: 以 `r` 从当前函数返回。
- **L1103 EN**: Closes the current lexical scope or compound statement.
  **L1103 CN**: 结束当前词法作用域或复合语句块。
- **L1104 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vabsdiffu4(unsigned int __a, unsigned int __b) {`.
  **L1104 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vabsdiffu4(unsigned int __a, unsigned int __b) {`。

### Lines 1105-1128

````c
  unsigned int r;
  __asm__("vabsdiff4.u32.u32.u32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vabsss2(unsigned int __a) {
  unsigned int r;
  __asm__("vabsdiff2.s32.s32.s32.sat %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(0), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vabsss4(unsigned int __a) {
  unsigned int r;
  __asm__("vabsdiff4.s32.s32.s32.sat %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(0), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vadd2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vadd2.u32.u32.u32 %0,%1,%2,%3;"
          : "=r"(r)
````
- **L1105 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1105 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1106 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1106 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1107 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1107 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1108 EN**: Executes a call or declaration centered on `"r"`.
  **L1108 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1109 EN**: Returns from the current function with `r`.
  **L1109 CN**: 以 `r` 从当前函数返回。
- **L1110 EN**: Closes the current lexical scope or compound statement.
  **L1110 CN**: 结束当前词法作用域或复合语句块。
- **L1111 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vabsss2(unsigned int __a) {`.
  **L1111 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vabsss2(unsigned int __a) {`。
- **L1112 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1112 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1113 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1113 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1114 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1114 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1115 EN**: Executes a call or declaration centered on `"r"`.
  **L1115 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1116 EN**: Returns from the current function with `r`.
  **L1116 CN**: 以 `r` 从当前函数返回。
- **L1117 EN**: Closes the current lexical scope or compound statement.
  **L1117 CN**: 结束当前词法作用域或复合语句块。
- **L1118 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vabsss4(unsigned int __a) {`.
  **L1118 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vabsss4(unsigned int __a) {`。
- **L1119 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1119 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1120 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1120 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1121 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1121 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1122 EN**: Executes a call or declaration centered on `"r"`.
  **L1122 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1123 EN**: Returns from the current function with `r`.
  **L1123 CN**: 以 `r` 从当前函数返回。
- **L1124 EN**: Closes the current lexical scope or compound statement.
  **L1124 CN**: 结束当前词法作用域或复合语句块。
- **L1125 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vadd2(unsigned int __a, unsigned int __b) {`.
  **L1125 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vadd2(unsigned int __a, unsigned int __b) {`。
- **L1126 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1126 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1127 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1127 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1128 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1128 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。

### Lines 1129-1152

````c
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vadd4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vadd4.u32.u32.u32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vaddss2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vadd2.s32.s32.s32.sat %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vaddss4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vadd4.s32.s32.s32.sat %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
````
- **L1129 EN**: Executes a call or declaration centered on `"r"`.
  **L1129 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1130 EN**: Returns from the current function with `r`.
  **L1130 CN**: 以 `r` 从当前函数返回。
- **L1131 EN**: Closes the current lexical scope or compound statement.
  **L1131 CN**: 结束当前词法作用域或复合语句块。
- **L1132 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vadd4(unsigned int __a, unsigned int __b) {`.
  **L1132 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vadd4(unsigned int __a, unsigned int __b) {`。
- **L1133 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1133 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1134 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1134 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1135 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1135 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1136 EN**: Executes a call or declaration centered on `"r"`.
  **L1136 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1137 EN**: Returns from the current function with `r`.
  **L1137 CN**: 以 `r` 从当前函数返回。
- **L1138 EN**: Closes the current lexical scope or compound statement.
  **L1138 CN**: 结束当前词法作用域或复合语句块。
- **L1139 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vaddss2(unsigned int __a, unsigned int __b) {`.
  **L1139 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vaddss2(unsigned int __a, unsigned int __b) {`。
- **L1140 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1140 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1141 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1141 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1142 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1142 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1143 EN**: Executes a call or declaration centered on `"r"`.
  **L1143 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1144 EN**: Returns from the current function with `r`.
  **L1144 CN**: 以 `r` 从当前函数返回。
- **L1145 EN**: Closes the current lexical scope or compound statement.
  **L1145 CN**: 结束当前词法作用域或复合语句块。
- **L1146 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vaddss4(unsigned int __a, unsigned int __b) {`.
  **L1146 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vaddss4(unsigned int __a, unsigned int __b) {`。
- **L1147 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1147 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1148 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1148 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1149 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1149 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1150 EN**: Executes a call or declaration centered on `"r"`.
  **L1150 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1151 EN**: Returns from the current function with `r`.
  **L1151 CN**: 以 `r` 从当前函数返回。
- **L1152 EN**: Closes the current lexical scope or compound statement.
  **L1152 CN**: 结束当前词法作用域或复合语句块。

### Lines 1153-1176

````c
__DEVICE__ unsigned int __vaddus2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vadd2.u32.u32.u32.sat %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vaddus4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vadd4.u32.u32.u32.sat %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vavgs2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vavrg2.s32.s32.s32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vavgs4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vavrg4.s32.s32.s32 %0,%1,%2,%3;"
````
- **L1153 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vaddus2(unsigned int __a, unsigned int __b) {`.
  **L1153 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vaddus2(unsigned int __a, unsigned int __b) {`。
- **L1154 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1154 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1155 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1155 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1156 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1156 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1157 EN**: Executes a call or declaration centered on `"r"`.
  **L1157 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1158 EN**: Returns from the current function with `r`.
  **L1158 CN**: 以 `r` 从当前函数返回。
- **L1159 EN**: Closes the current lexical scope or compound statement.
  **L1159 CN**: 结束当前词法作用域或复合语句块。
- **L1160 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vaddus4(unsigned int __a, unsigned int __b) {`.
  **L1160 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vaddus4(unsigned int __a, unsigned int __b) {`。
- **L1161 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1161 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1162 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1162 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1163 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1163 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1164 EN**: Executes a call or declaration centered on `"r"`.
  **L1164 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1165 EN**: Returns from the current function with `r`.
  **L1165 CN**: 以 `r` 从当前函数返回。
- **L1166 EN**: Closes the current lexical scope or compound statement.
  **L1166 CN**: 结束当前词法作用域或复合语句块。
- **L1167 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vavgs2(unsigned int __a, unsigned int __b) {`.
  **L1167 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vavgs2(unsigned int __a, unsigned int __b) {`。
- **L1168 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1168 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1169 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1169 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1170 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1170 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1171 EN**: Executes a call or declaration centered on `"r"`.
  **L1171 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1172 EN**: Returns from the current function with `r`.
  **L1172 CN**: 以 `r` 从当前函数返回。
- **L1173 EN**: Closes the current lexical scope or compound statement.
  **L1173 CN**: 结束当前词法作用域或复合语句块。
- **L1174 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vavgs4(unsigned int __a, unsigned int __b) {`.
  **L1174 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vavgs4(unsigned int __a, unsigned int __b) {`。
- **L1175 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1175 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1176 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1176 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。

### Lines 1177-1200

````c
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vavgu2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vavrg2.u32.u32.u32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vavgu4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vavrg4.u32.u32.u32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vseteq2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset2.u32.u32.eq %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
````
- **L1177 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1177 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1178 EN**: Executes a call or declaration centered on `"r"`.
  **L1178 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1179 EN**: Returns from the current function with `r`.
  **L1179 CN**: 以 `r` 从当前函数返回。
- **L1180 EN**: Closes the current lexical scope or compound statement.
  **L1180 CN**: 结束当前词法作用域或复合语句块。
- **L1181 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vavgu2(unsigned int __a, unsigned int __b) {`.
  **L1181 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vavgu2(unsigned int __a, unsigned int __b) {`。
- **L1182 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1182 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1183 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1183 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1184 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1184 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1185 EN**: Executes a call or declaration centered on `"r"`.
  **L1185 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1186 EN**: Returns from the current function with `r`.
  **L1186 CN**: 以 `r` 从当前函数返回。
- **L1187 EN**: Closes the current lexical scope or compound statement.
  **L1187 CN**: 结束当前词法作用域或复合语句块。
- **L1188 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vavgu4(unsigned int __a, unsigned int __b) {`.
  **L1188 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vavgu4(unsigned int __a, unsigned int __b) {`。
- **L1189 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1189 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1190 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1190 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1191 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1191 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1192 EN**: Executes a call or declaration centered on `"r"`.
  **L1192 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1193 EN**: Returns from the current function with `r`.
  **L1193 CN**: 以 `r` 从当前函数返回。
- **L1194 EN**: Closes the current lexical scope or compound statement.
  **L1194 CN**: 结束当前词法作用域或复合语句块。
- **L1195 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vseteq2(unsigned int __a, unsigned int __b) {`.
  **L1195 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vseteq2(unsigned int __a, unsigned int __b) {`。
- **L1196 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1196 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1197 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1197 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1198 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1198 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1199 EN**: Executes a call or declaration centered on `"r"`.
  **L1199 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1200 EN**: Returns from the current function with `r`.
  **L1200 CN**: 以 `r` 从当前函数返回。

### Lines 1201-1224

````c
}
__DEVICE__ unsigned int __vcmpeq2(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vseteq2(__a, __b), 16);
}
__DEVICE__ unsigned int __vseteq4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset4.u32.u32.eq %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmpeq4(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vseteq4(__a, __b), 8);
}
__DEVICE__ unsigned int __vsetges2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset2.s32.s32.ge %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmpges2(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vsetges2(__a, __b), 16);
}
````
- **L1201 EN**: Closes the current lexical scope or compound statement.
  **L1201 CN**: 结束当前词法作用域或复合语句块。
- **L1202 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpeq2(unsigned int __a, unsigned int __b) {`.
  **L1202 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpeq2(unsigned int __a, unsigned int __b) {`。
- **L1203 EN**: Returns from the current function with `__bool2mask(__vseteq2(__a, __b), 16)`.
  **L1203 CN**: 以 `__bool2mask(__vseteq2(__a, __b), 16)` 从当前函数返回。
- **L1204 EN**: Closes the current lexical scope or compound statement.
  **L1204 CN**: 结束当前词法作用域或复合语句块。
- **L1205 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vseteq4(unsigned int __a, unsigned int __b) {`.
  **L1205 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vseteq4(unsigned int __a, unsigned int __b) {`。
- **L1206 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1206 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1207 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1207 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1208 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1208 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1209 EN**: Executes a call or declaration centered on `"r"`.
  **L1209 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1210 EN**: Returns from the current function with `r`.
  **L1210 CN**: 以 `r` 从当前函数返回。
- **L1211 EN**: Closes the current lexical scope or compound statement.
  **L1211 CN**: 结束当前词法作用域或复合语句块。
- **L1212 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpeq4(unsigned int __a, unsigned int __b) {`.
  **L1212 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpeq4(unsigned int __a, unsigned int __b) {`。
- **L1213 EN**: Returns from the current function with `__bool2mask(__vseteq4(__a, __b), 8)`.
  **L1213 CN**: 以 `__bool2mask(__vseteq4(__a, __b), 8)` 从当前函数返回。
- **L1214 EN**: Closes the current lexical scope or compound statement.
  **L1214 CN**: 结束当前词法作用域或复合语句块。
- **L1215 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetges2(unsigned int __a, unsigned int __b) {`.
  **L1215 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetges2(unsigned int __a, unsigned int __b) {`。
- **L1216 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1216 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1217 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1217 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1218 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1218 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1219 EN**: Executes a call or declaration centered on `"r"`.
  **L1219 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1220 EN**: Returns from the current function with `r`.
  **L1220 CN**: 以 `r` 从当前函数返回。
- **L1221 EN**: Closes the current lexical scope or compound statement.
  **L1221 CN**: 结束当前词法作用域或复合语句块。
- **L1222 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpges2(unsigned int __a, unsigned int __b) {`.
  **L1222 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpges2(unsigned int __a, unsigned int __b) {`。
- **L1223 EN**: Returns from the current function with `__bool2mask(__vsetges2(__a, __b), 16)`.
  **L1223 CN**: 以 `__bool2mask(__vsetges2(__a, __b), 16)` 从当前函数返回。
- **L1224 EN**: Closes the current lexical scope or compound statement.
  **L1224 CN**: 结束当前词法作用域或复合语句块。

### Lines 1225-1248

````c
__DEVICE__ unsigned int __vsetges4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset4.s32.s32.ge %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmpges4(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vsetges4(__a, __b), 8);
}
__DEVICE__ unsigned int __vsetgeu2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset2.u32.u32.ge %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmpgeu2(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vsetgeu2(__a, __b), 16);
}
__DEVICE__ unsigned int __vsetgeu4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset4.u32.u32.ge %0,%1,%2,%3;"
          : "=r"(r)
````
- **L1225 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetges4(unsigned int __a, unsigned int __b) {`.
  **L1225 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetges4(unsigned int __a, unsigned int __b) {`。
- **L1226 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1226 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1227 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1227 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1228 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1228 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1229 EN**: Executes a call or declaration centered on `"r"`.
  **L1229 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1230 EN**: Returns from the current function with `r`.
  **L1230 CN**: 以 `r` 从当前函数返回。
- **L1231 EN**: Closes the current lexical scope or compound statement.
  **L1231 CN**: 结束当前词法作用域或复合语句块。
- **L1232 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpges4(unsigned int __a, unsigned int __b) {`.
  **L1232 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpges4(unsigned int __a, unsigned int __b) {`。
- **L1233 EN**: Returns from the current function with `__bool2mask(__vsetges4(__a, __b), 8)`.
  **L1233 CN**: 以 `__bool2mask(__vsetges4(__a, __b), 8)` 从当前函数返回。
- **L1234 EN**: Closes the current lexical scope or compound statement.
  **L1234 CN**: 结束当前词法作用域或复合语句块。
- **L1235 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetgeu2(unsigned int __a, unsigned int __b) {`.
  **L1235 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetgeu2(unsigned int __a, unsigned int __b) {`。
- **L1236 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1236 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1237 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1237 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1238 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1238 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1239 EN**: Executes a call or declaration centered on `"r"`.
  **L1239 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1240 EN**: Returns from the current function with `r`.
  **L1240 CN**: 以 `r` 从当前函数返回。
- **L1241 EN**: Closes the current lexical scope or compound statement.
  **L1241 CN**: 结束当前词法作用域或复合语句块。
- **L1242 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpgeu2(unsigned int __a, unsigned int __b) {`.
  **L1242 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpgeu2(unsigned int __a, unsigned int __b) {`。
- **L1243 EN**: Returns from the current function with `__bool2mask(__vsetgeu2(__a, __b), 16)`.
  **L1243 CN**: 以 `__bool2mask(__vsetgeu2(__a, __b), 16)` 从当前函数返回。
- **L1244 EN**: Closes the current lexical scope or compound statement.
  **L1244 CN**: 结束当前词法作用域或复合语句块。
- **L1245 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetgeu4(unsigned int __a, unsigned int __b) {`.
  **L1245 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetgeu4(unsigned int __a, unsigned int __b) {`。
- **L1246 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1246 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1247 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1247 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1248 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1248 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。

### Lines 1249-1272

````c
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmpgeu4(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vsetgeu4(__a, __b), 8);
}
__DEVICE__ unsigned int __vsetgts2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset2.s32.s32.gt %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmpgts2(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vsetgts2(__a, __b), 16);
}
__DEVICE__ unsigned int __vsetgts4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset4.s32.s32.gt %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmpgts4(unsigned int __a, unsigned int __b) {
````
- **L1249 EN**: Executes a call or declaration centered on `"r"`.
  **L1249 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1250 EN**: Returns from the current function with `r`.
  **L1250 CN**: 以 `r` 从当前函数返回。
- **L1251 EN**: Closes the current lexical scope or compound statement.
  **L1251 CN**: 结束当前词法作用域或复合语句块。
- **L1252 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpgeu4(unsigned int __a, unsigned int __b) {`.
  **L1252 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpgeu4(unsigned int __a, unsigned int __b) {`。
- **L1253 EN**: Returns from the current function with `__bool2mask(__vsetgeu4(__a, __b), 8)`.
  **L1253 CN**: 以 `__bool2mask(__vsetgeu4(__a, __b), 8)` 从当前函数返回。
- **L1254 EN**: Closes the current lexical scope or compound statement.
  **L1254 CN**: 结束当前词法作用域或复合语句块。
- **L1255 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetgts2(unsigned int __a, unsigned int __b) {`.
  **L1255 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetgts2(unsigned int __a, unsigned int __b) {`。
- **L1256 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1256 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1257 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1257 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1258 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1258 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1259 EN**: Executes a call or declaration centered on `"r"`.
  **L1259 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1260 EN**: Returns from the current function with `r`.
  **L1260 CN**: 以 `r` 从当前函数返回。
- **L1261 EN**: Closes the current lexical scope or compound statement.
  **L1261 CN**: 结束当前词法作用域或复合语句块。
- **L1262 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpgts2(unsigned int __a, unsigned int __b) {`.
  **L1262 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpgts2(unsigned int __a, unsigned int __b) {`。
- **L1263 EN**: Returns from the current function with `__bool2mask(__vsetgts2(__a, __b), 16)`.
  **L1263 CN**: 以 `__bool2mask(__vsetgts2(__a, __b), 16)` 从当前函数返回。
- **L1264 EN**: Closes the current lexical scope or compound statement.
  **L1264 CN**: 结束当前词法作用域或复合语句块。
- **L1265 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetgts4(unsigned int __a, unsigned int __b) {`.
  **L1265 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetgts4(unsigned int __a, unsigned int __b) {`。
- **L1266 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1266 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1267 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1267 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1268 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1268 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1269 EN**: Executes a call or declaration centered on `"r"`.
  **L1269 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1270 EN**: Returns from the current function with `r`.
  **L1270 CN**: 以 `r` 从当前函数返回。
- **L1271 EN**: Closes the current lexical scope or compound statement.
  **L1271 CN**: 结束当前词法作用域或复合语句块。
- **L1272 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpgts4(unsigned int __a, unsigned int __b) {`.
  **L1272 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpgts4(unsigned int __a, unsigned int __b) {`。

### Lines 1273-1296

````c
  return __bool2mask(__vsetgts4(__a, __b), 8);
}
__DEVICE__ unsigned int __vsetgtu2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset2.u32.u32.gt %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmpgtu2(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vsetgtu2(__a, __b), 16);
}
__DEVICE__ unsigned int __vsetgtu4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset4.u32.u32.gt %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmpgtu4(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vsetgtu4(__a, __b), 8);
}
__DEVICE__ unsigned int __vsetles2(unsigned int __a, unsigned int __b) {
  unsigned int r;
````
- **L1273 EN**: Returns from the current function with `__bool2mask(__vsetgts4(__a, __b), 8)`.
  **L1273 CN**: 以 `__bool2mask(__vsetgts4(__a, __b), 8)` 从当前函数返回。
- **L1274 EN**: Closes the current lexical scope or compound statement.
  **L1274 CN**: 结束当前词法作用域或复合语句块。
- **L1275 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetgtu2(unsigned int __a, unsigned int __b) {`.
  **L1275 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetgtu2(unsigned int __a, unsigned int __b) {`。
- **L1276 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1276 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1277 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1277 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1278 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1278 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1279 EN**: Executes a call or declaration centered on `"r"`.
  **L1279 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1280 EN**: Returns from the current function with `r`.
  **L1280 CN**: 以 `r` 从当前函数返回。
- **L1281 EN**: Closes the current lexical scope or compound statement.
  **L1281 CN**: 结束当前词法作用域或复合语句块。
- **L1282 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpgtu2(unsigned int __a, unsigned int __b) {`.
  **L1282 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpgtu2(unsigned int __a, unsigned int __b) {`。
- **L1283 EN**: Returns from the current function with `__bool2mask(__vsetgtu2(__a, __b), 16)`.
  **L1283 CN**: 以 `__bool2mask(__vsetgtu2(__a, __b), 16)` 从当前函数返回。
- **L1284 EN**: Closes the current lexical scope or compound statement.
  **L1284 CN**: 结束当前词法作用域或复合语句块。
- **L1285 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetgtu4(unsigned int __a, unsigned int __b) {`.
  **L1285 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetgtu4(unsigned int __a, unsigned int __b) {`。
- **L1286 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1286 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1287 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1287 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1288 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1288 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1289 EN**: Executes a call or declaration centered on `"r"`.
  **L1289 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1290 EN**: Returns from the current function with `r`.
  **L1290 CN**: 以 `r` 从当前函数返回。
- **L1291 EN**: Closes the current lexical scope or compound statement.
  **L1291 CN**: 结束当前词法作用域或复合语句块。
- **L1292 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpgtu4(unsigned int __a, unsigned int __b) {`.
  **L1292 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpgtu4(unsigned int __a, unsigned int __b) {`。
- **L1293 EN**: Returns from the current function with `__bool2mask(__vsetgtu4(__a, __b), 8)`.
  **L1293 CN**: 以 `__bool2mask(__vsetgtu4(__a, __b), 8)` 从当前函数返回。
- **L1294 EN**: Closes the current lexical scope or compound statement.
  **L1294 CN**: 结束当前词法作用域或复合语句块。
- **L1295 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetles2(unsigned int __a, unsigned int __b) {`.
  **L1295 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetles2(unsigned int __a, unsigned int __b) {`。
- **L1296 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1296 CN**: 添加一条独立语句或声明：`unsigned int r;`。

### Lines 1297-1320

````c
  __asm__("vset2.s32.s32.le %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmples2(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vsetles2(__a, __b), 16);
}
__DEVICE__ unsigned int __vsetles4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset4.s32.s32.le %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmples4(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vsetles4(__a, __b), 8);
}
__DEVICE__ unsigned int __vsetleu2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset2.u32.u32.le %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
````
- **L1297 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1297 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1298 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1298 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1299 EN**: Executes a call or declaration centered on `"r"`.
  **L1299 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1300 EN**: Returns from the current function with `r`.
  **L1300 CN**: 以 `r` 从当前函数返回。
- **L1301 EN**: Closes the current lexical scope or compound statement.
  **L1301 CN**: 结束当前词法作用域或复合语句块。
- **L1302 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmples2(unsigned int __a, unsigned int __b) {`.
  **L1302 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmples2(unsigned int __a, unsigned int __b) {`。
- **L1303 EN**: Returns from the current function with `__bool2mask(__vsetles2(__a, __b), 16)`.
  **L1303 CN**: 以 `__bool2mask(__vsetles2(__a, __b), 16)` 从当前函数返回。
- **L1304 EN**: Closes the current lexical scope or compound statement.
  **L1304 CN**: 结束当前词法作用域或复合语句块。
- **L1305 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetles4(unsigned int __a, unsigned int __b) {`.
  **L1305 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetles4(unsigned int __a, unsigned int __b) {`。
- **L1306 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1306 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1307 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1307 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1308 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1308 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1309 EN**: Executes a call or declaration centered on `"r"`.
  **L1309 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1310 EN**: Returns from the current function with `r`.
  **L1310 CN**: 以 `r` 从当前函数返回。
- **L1311 EN**: Closes the current lexical scope or compound statement.
  **L1311 CN**: 结束当前词法作用域或复合语句块。
- **L1312 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmples4(unsigned int __a, unsigned int __b) {`.
  **L1312 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmples4(unsigned int __a, unsigned int __b) {`。
- **L1313 EN**: Returns from the current function with `__bool2mask(__vsetles4(__a, __b), 8)`.
  **L1313 CN**: 以 `__bool2mask(__vsetles4(__a, __b), 8)` 从当前函数返回。
- **L1314 EN**: Closes the current lexical scope or compound statement.
  **L1314 CN**: 结束当前词法作用域或复合语句块。
- **L1315 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetleu2(unsigned int __a, unsigned int __b) {`.
  **L1315 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetleu2(unsigned int __a, unsigned int __b) {`。
- **L1316 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1316 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1317 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1317 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1318 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1318 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1319 EN**: Executes a call or declaration centered on `"r"`.
  **L1319 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1320 EN**: Returns from the current function with `r`.
  **L1320 CN**: 以 `r` 从当前函数返回。

### Lines 1321-1344

````c
}
__DEVICE__ unsigned int __vcmpleu2(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vsetleu2(__a, __b), 16);
}
__DEVICE__ unsigned int __vsetleu4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset4.u32.u32.le %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmpleu4(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vsetleu4(__a, __b), 8);
}
__DEVICE__ unsigned int __vsetlts2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset2.s32.s32.lt %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmplts2(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vsetlts2(__a, __b), 16);
}
````
- **L1321 EN**: Closes the current lexical scope or compound statement.
  **L1321 CN**: 结束当前词法作用域或复合语句块。
- **L1322 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpleu2(unsigned int __a, unsigned int __b) {`.
  **L1322 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpleu2(unsigned int __a, unsigned int __b) {`。
- **L1323 EN**: Returns from the current function with `__bool2mask(__vsetleu2(__a, __b), 16)`.
  **L1323 CN**: 以 `__bool2mask(__vsetleu2(__a, __b), 16)` 从当前函数返回。
- **L1324 EN**: Closes the current lexical scope or compound statement.
  **L1324 CN**: 结束当前词法作用域或复合语句块。
- **L1325 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetleu4(unsigned int __a, unsigned int __b) {`.
  **L1325 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetleu4(unsigned int __a, unsigned int __b) {`。
- **L1326 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1326 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1327 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1327 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1328 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1328 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1329 EN**: Executes a call or declaration centered on `"r"`.
  **L1329 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1330 EN**: Returns from the current function with `r`.
  **L1330 CN**: 以 `r` 从当前函数返回。
- **L1331 EN**: Closes the current lexical scope or compound statement.
  **L1331 CN**: 结束当前词法作用域或复合语句块。
- **L1332 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpleu4(unsigned int __a, unsigned int __b) {`.
  **L1332 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpleu4(unsigned int __a, unsigned int __b) {`。
- **L1333 EN**: Returns from the current function with `__bool2mask(__vsetleu4(__a, __b), 8)`.
  **L1333 CN**: 以 `__bool2mask(__vsetleu4(__a, __b), 8)` 从当前函数返回。
- **L1334 EN**: Closes the current lexical scope or compound statement.
  **L1334 CN**: 结束当前词法作用域或复合语句块。
- **L1335 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetlts2(unsigned int __a, unsigned int __b) {`.
  **L1335 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetlts2(unsigned int __a, unsigned int __b) {`。
- **L1336 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1336 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1337 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1337 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1338 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1338 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1339 EN**: Executes a call or declaration centered on `"r"`.
  **L1339 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1340 EN**: Returns from the current function with `r`.
  **L1340 CN**: 以 `r` 从当前函数返回。
- **L1341 EN**: Closes the current lexical scope or compound statement.
  **L1341 CN**: 结束当前词法作用域或复合语句块。
- **L1342 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmplts2(unsigned int __a, unsigned int __b) {`.
  **L1342 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmplts2(unsigned int __a, unsigned int __b) {`。
- **L1343 EN**: Returns from the current function with `__bool2mask(__vsetlts2(__a, __b), 16)`.
  **L1343 CN**: 以 `__bool2mask(__vsetlts2(__a, __b), 16)` 从当前函数返回。
- **L1344 EN**: Closes the current lexical scope or compound statement.
  **L1344 CN**: 结束当前词法作用域或复合语句块。

### Lines 1345-1368

````c
__DEVICE__ unsigned int __vsetlts4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset4.s32.s32.lt %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmplts4(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vsetlts4(__a, __b), 8);
}
__DEVICE__ unsigned int __vsetltu2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset2.u32.u32.lt %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmpltu2(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vsetltu2(__a, __b), 16);
}
__DEVICE__ unsigned int __vsetltu4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset4.u32.u32.lt %0,%1,%2,%3;"
          : "=r"(r)
````
- **L1345 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetlts4(unsigned int __a, unsigned int __b) {`.
  **L1345 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetlts4(unsigned int __a, unsigned int __b) {`。
- **L1346 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1346 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1347 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1347 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1348 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1348 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1349 EN**: Executes a call or declaration centered on `"r"`.
  **L1349 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1350 EN**: Returns from the current function with `r`.
  **L1350 CN**: 以 `r` 从当前函数返回。
- **L1351 EN**: Closes the current lexical scope or compound statement.
  **L1351 CN**: 结束当前词法作用域或复合语句块。
- **L1352 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmplts4(unsigned int __a, unsigned int __b) {`.
  **L1352 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmplts4(unsigned int __a, unsigned int __b) {`。
- **L1353 EN**: Returns from the current function with `__bool2mask(__vsetlts4(__a, __b), 8)`.
  **L1353 CN**: 以 `__bool2mask(__vsetlts4(__a, __b), 8)` 从当前函数返回。
- **L1354 EN**: Closes the current lexical scope or compound statement.
  **L1354 CN**: 结束当前词法作用域或复合语句块。
- **L1355 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetltu2(unsigned int __a, unsigned int __b) {`.
  **L1355 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetltu2(unsigned int __a, unsigned int __b) {`。
- **L1356 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1356 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1357 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1357 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1358 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1358 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1359 EN**: Executes a call or declaration centered on `"r"`.
  **L1359 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1360 EN**: Returns from the current function with `r`.
  **L1360 CN**: 以 `r` 从当前函数返回。
- **L1361 EN**: Closes the current lexical scope or compound statement.
  **L1361 CN**: 结束当前词法作用域或复合语句块。
- **L1362 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpltu2(unsigned int __a, unsigned int __b) {`.
  **L1362 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpltu2(unsigned int __a, unsigned int __b) {`。
- **L1363 EN**: Returns from the current function with `__bool2mask(__vsetltu2(__a, __b), 16)`.
  **L1363 CN**: 以 `__bool2mask(__vsetltu2(__a, __b), 16)` 从当前函数返回。
- **L1364 EN**: Closes the current lexical scope or compound statement.
  **L1364 CN**: 结束当前词法作用域或复合语句块。
- **L1365 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetltu4(unsigned int __a, unsigned int __b) {`.
  **L1365 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetltu4(unsigned int __a, unsigned int __b) {`。
- **L1366 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1366 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1367 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1367 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1368 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1368 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。

### Lines 1369-1392

````c
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmpltu4(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vsetltu4(__a, __b), 8);
}
__DEVICE__ unsigned int __vsetne2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset2.u32.u32.ne %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmpne2(unsigned int __a, unsigned int __b) {
  return __bool2mask(__vsetne2(__a, __b), 16);
}
__DEVICE__ unsigned int __vsetne4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vset4.u32.u32.ne %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vcmpne4(unsigned int __a, unsigned int __b) {
````
- **L1369 EN**: Executes a call or declaration centered on `"r"`.
  **L1369 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1370 EN**: Returns from the current function with `r`.
  **L1370 CN**: 以 `r` 从当前函数返回。
- **L1371 EN**: Closes the current lexical scope or compound statement.
  **L1371 CN**: 结束当前词法作用域或复合语句块。
- **L1372 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpltu4(unsigned int __a, unsigned int __b) {`.
  **L1372 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpltu4(unsigned int __a, unsigned int __b) {`。
- **L1373 EN**: Returns from the current function with `__bool2mask(__vsetltu4(__a, __b), 8)`.
  **L1373 CN**: 以 `__bool2mask(__vsetltu4(__a, __b), 8)` 从当前函数返回。
- **L1374 EN**: Closes the current lexical scope or compound statement.
  **L1374 CN**: 结束当前词法作用域或复合语句块。
- **L1375 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetne2(unsigned int __a, unsigned int __b) {`.
  **L1375 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetne2(unsigned int __a, unsigned int __b) {`。
- **L1376 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1376 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1377 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1377 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1378 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1378 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1379 EN**: Executes a call or declaration centered on `"r"`.
  **L1379 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1380 EN**: Returns from the current function with `r`.
  **L1380 CN**: 以 `r` 从当前函数返回。
- **L1381 EN**: Closes the current lexical scope or compound statement.
  **L1381 CN**: 结束当前词法作用域或复合语句块。
- **L1382 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpne2(unsigned int __a, unsigned int __b) {`.
  **L1382 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpne2(unsigned int __a, unsigned int __b) {`。
- **L1383 EN**: Returns from the current function with `__bool2mask(__vsetne2(__a, __b), 16)`.
  **L1383 CN**: 以 `__bool2mask(__vsetne2(__a, __b), 16)` 从当前函数返回。
- **L1384 EN**: Closes the current lexical scope or compound statement.
  **L1384 CN**: 结束当前词法作用域或复合语句块。
- **L1385 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsetne4(unsigned int __a, unsigned int __b) {`.
  **L1385 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsetne4(unsigned int __a, unsigned int __b) {`。
- **L1386 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1386 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1387 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1387 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1388 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1388 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1389 EN**: Executes a call or declaration centered on `"r"`.
  **L1389 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1390 EN**: Returns from the current function with `r`.
  **L1390 CN**: 以 `r` 从当前函数返回。
- **L1391 EN**: Closes the current lexical scope or compound statement.
  **L1391 CN**: 结束当前词法作用域或复合语句块。
- **L1392 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vcmpne4(unsigned int __a, unsigned int __b) {`.
  **L1392 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vcmpne4(unsigned int __a, unsigned int __b) {`。

### Lines 1393-1416

````c
  return __bool2mask(__vsetne4(__a, __b), 8);
}

// Based on ITEM 23 in AIM-239: http://dspace.mit.edu/handle/1721.1/6086
// (a & b) + (a | b) = a + b = (a ^ b) + 2 * (a & b) =>
// (a + b) / 2 = ((a ^ b) >> 1) + (a & b)
// To operate on multiple sub-elements we need to make sure to mask out bits
// that crossed over into adjacent elements during the shift.
__DEVICE__ unsigned int __vhaddu2(unsigned int __a, unsigned int __b) {
  return (((__a ^ __b) >> 1) & ~0x80008000u) + (__a & __b);
}
__DEVICE__ unsigned int __vhaddu4(unsigned int __a, unsigned int __b) {
  return (((__a ^ __b) >> 1) & ~0x80808080u) + (__a & __b);
}

__DEVICE__ unsigned int __vmaxs2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  if ((__a & 0x8000) && (__b & 0x8000)) {
    // Work around a bug in ptxas which produces invalid result if low element
    // is negative.
    unsigned mask = __vcmpgts2(__a, __b);
    r = (__a & mask) | (__b & ~mask);
  } else {
    __asm__("vmax2.s32.s32.s32 %0,%1,%2,%3;"
````
- **L1393 EN**: Returns from the current function with `__bool2mask(__vsetne4(__a, __b), 8)`.
  **L1393 CN**: 以 `__bool2mask(__vsetne4(__a, __b), 8)` 从当前函数返回。
- **L1394 EN**: Closes the current lexical scope or compound statement.
  **L1394 CN**: 结束当前词法作用域或复合语句块。
- **L1395 EN**: Blank line separating nearby declarations or logic blocks.
  **L1395 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1396 EN**: Comment explains nearby logic, constraints, or intent: `Based on ITEM 23 in AIM-239: http://dspace.mit.edu/handle/1721.1/6086`.
  **L1396 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Based on ITEM 23 in AIM-239: http://dspace.mit.edu/handle/1721.1/6086`。
- **L1397 EN**: Comment explains nearby logic, constraints, or intent: `(a & b) + (a | b) a + b (a ^ b) + 2 * (a & b) >`.
  **L1397 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(a & b) + (a | b) a + b (a ^ b) + 2 * (a & b) >`。
- **L1398 EN**: Comment explains nearby logic, constraints, or intent: `(a + b) / 2 ((a ^ b) >> 1) + (a & b)`.
  **L1398 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(a + b) / 2 ((a ^ b) >> 1) + (a & b)`。
- **L1399 EN**: Comment explains nearby logic, constraints, or intent: `To operate on multiple sub-elements we need to make sure to mask out bits`.
  **L1399 CN**: 注释解释附近代码的逻辑、约束或设计意图：`To operate on multiple sub-elements we need to make sure to mask out bits`。
- **L1400 EN**: Comment explains nearby logic, constraints, or intent: `that crossed over into adjacent elements during the shift.`.
  **L1400 CN**: 注释解释附近代码的逻辑、约束或设计意图：`that crossed over into adjacent elements during the shift.`。
- **L1401 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vhaddu2(unsigned int __a, unsigned int __b) {`.
  **L1401 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vhaddu2(unsigned int __a, unsigned int __b) {`。
- **L1402 EN**: Returns from the current function with `(((__a ^ __b) >> 1) & ~0x80008000u) + (__a & __b)`.
  **L1402 CN**: 以 `(((__a ^ __b) >> 1) & ~0x80008000u) + (__a & __b)` 从当前函数返回。
- **L1403 EN**: Closes the current lexical scope or compound statement.
  **L1403 CN**: 结束当前词法作用域或复合语句块。
- **L1404 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vhaddu4(unsigned int __a, unsigned int __b) {`.
  **L1404 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vhaddu4(unsigned int __a, unsigned int __b) {`。
- **L1405 EN**: Returns from the current function with `(((__a ^ __b) >> 1) & ~0x80808080u) + (__a & __b)`.
  **L1405 CN**: 以 `(((__a ^ __b) >> 1) & ~0x80808080u) + (__a & __b)` 从当前函数返回。
- **L1406 EN**: Closes the current lexical scope or compound statement.
  **L1406 CN**: 结束当前词法作用域或复合语句块。
- **L1407 EN**: Blank line separating nearby declarations or logic blocks.
  **L1407 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1408 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vmaxs2(unsigned int __a, unsigned int __b) {`.
  **L1408 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vmaxs2(unsigned int __a, unsigned int __b) {`。
- **L1409 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1409 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1410 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1410 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1411 EN**: Comment explains nearby logic, constraints, or intent: `Work around a bug in ptxas which produces invalid result if low element`.
  **L1411 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Work around a bug in ptxas which produces invalid result if low element`。
- **L1412 EN**: Comment explains nearby logic, constraints, or intent: `is negative.`.
  **L1412 CN**: 注释解释附近代码的逻辑、约束或设计意图：`is negative.`。
- **L1413 EN**: Initializes variable `mask` from the expression on the right-hand side.
  **L1413 CN**: 使用右侧表达式初始化变量 `mask`。
- **L1414 EN**: Executes a call or declaration centered on `=`.
  **L1414 CN**: 执行以 `=` 为核心的调用或声明。
- **L1415 EN**: Continues the surrounding expression or declaration: `} else {`.
  **L1415 CN**: 继续构造周围的表达式或声明：`} else {`。
- **L1416 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1416 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。

### Lines 1417-1440

````c
            : "=r"(r)
            : "r"(__a), "r"(__b), "r"(0));
  }
  return r;
}
__DEVICE__ unsigned int __vmaxs4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vmax4.s32.s32.s32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vmaxu2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vmax2.u32.u32.u32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vmaxu4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vmax4.u32.u32.u32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
````
- **L1417 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1417 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1418 EN**: Executes a call or declaration centered on `"r"`.
  **L1418 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1419 EN**: Closes the current lexical scope or compound statement.
  **L1419 CN**: 结束当前词法作用域或复合语句块。
- **L1420 EN**: Returns from the current function with `r`.
  **L1420 CN**: 以 `r` 从当前函数返回。
- **L1421 EN**: Closes the current lexical scope or compound statement.
  **L1421 CN**: 结束当前词法作用域或复合语句块。
- **L1422 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vmaxs4(unsigned int __a, unsigned int __b) {`.
  **L1422 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vmaxs4(unsigned int __a, unsigned int __b) {`。
- **L1423 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1423 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1424 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1424 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1425 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1425 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1426 EN**: Executes a call or declaration centered on `"r"`.
  **L1426 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1427 EN**: Returns from the current function with `r`.
  **L1427 CN**: 以 `r` 从当前函数返回。
- **L1428 EN**: Closes the current lexical scope or compound statement.
  **L1428 CN**: 结束当前词法作用域或复合语句块。
- **L1429 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vmaxu2(unsigned int __a, unsigned int __b) {`.
  **L1429 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vmaxu2(unsigned int __a, unsigned int __b) {`。
- **L1430 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1430 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1431 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1431 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1432 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1432 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1433 EN**: Executes a call or declaration centered on `"r"`.
  **L1433 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1434 EN**: Returns from the current function with `r`.
  **L1434 CN**: 以 `r` 从当前函数返回。
- **L1435 EN**: Closes the current lexical scope or compound statement.
  **L1435 CN**: 结束当前词法作用域或复合语句块。
- **L1436 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vmaxu4(unsigned int __a, unsigned int __b) {`.
  **L1436 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vmaxu4(unsigned int __a, unsigned int __b) {`。
- **L1437 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1437 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1438 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1438 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1439 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1439 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1440 EN**: Executes a call or declaration centered on `"r"`.
  **L1440 CN**: 执行以 `"r"` 为核心的调用或声明。

### Lines 1441-1464

````c
  return r;
}
__DEVICE__ unsigned int __vmins2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vmin2.s32.s32.s32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vmins4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vmin4.s32.s32.s32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vminu2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vmin2.u32.u32.u32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vminu4(unsigned int __a, unsigned int __b) {
````
- **L1441 EN**: Returns from the current function with `r`.
  **L1441 CN**: 以 `r` 从当前函数返回。
- **L1442 EN**: Closes the current lexical scope or compound statement.
  **L1442 CN**: 结束当前词法作用域或复合语句块。
- **L1443 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vmins2(unsigned int __a, unsigned int __b) {`.
  **L1443 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vmins2(unsigned int __a, unsigned int __b) {`。
- **L1444 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1444 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1445 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1445 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1446 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1446 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1447 EN**: Executes a call or declaration centered on `"r"`.
  **L1447 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1448 EN**: Returns from the current function with `r`.
  **L1448 CN**: 以 `r` 从当前函数返回。
- **L1449 EN**: Closes the current lexical scope or compound statement.
  **L1449 CN**: 结束当前词法作用域或复合语句块。
- **L1450 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vmins4(unsigned int __a, unsigned int __b) {`.
  **L1450 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vmins4(unsigned int __a, unsigned int __b) {`。
- **L1451 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1451 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1452 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1452 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1453 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1453 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1454 EN**: Executes a call or declaration centered on `"r"`.
  **L1454 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1455 EN**: Returns from the current function with `r`.
  **L1455 CN**: 以 `r` 从当前函数返回。
- **L1456 EN**: Closes the current lexical scope or compound statement.
  **L1456 CN**: 结束当前词法作用域或复合语句块。
- **L1457 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vminu2(unsigned int __a, unsigned int __b) {`.
  **L1457 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vminu2(unsigned int __a, unsigned int __b) {`。
- **L1458 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1458 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1459 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1459 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1460 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1460 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1461 EN**: Executes a call or declaration centered on `"r"`.
  **L1461 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1462 EN**: Returns from the current function with `r`.
  **L1462 CN**: 以 `r` 从当前函数返回。
- **L1463 EN**: Closes the current lexical scope or compound statement.
  **L1463 CN**: 结束当前词法作用域或复合语句块。
- **L1464 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vminu4(unsigned int __a, unsigned int __b) {`.
  **L1464 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vminu4(unsigned int __a, unsigned int __b) {`。

### Lines 1465-1488

````c
  unsigned int r;
  __asm__("vmin4.u32.u32.u32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vsads2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vabsdiff2.s32.s32.s32.add %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vsads4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vabsdiff4.s32.s32.s32.add %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vsadu2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vabsdiff2.u32.u32.u32.add %0,%1,%2,%3;"
          : "=r"(r)
````
- **L1465 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1465 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1466 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1466 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1467 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1467 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1468 EN**: Executes a call or declaration centered on `"r"`.
  **L1468 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1469 EN**: Returns from the current function with `r`.
  **L1469 CN**: 以 `r` 从当前函数返回。
- **L1470 EN**: Closes the current lexical scope or compound statement.
  **L1470 CN**: 结束当前词法作用域或复合语句块。
- **L1471 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsads2(unsigned int __a, unsigned int __b) {`.
  **L1471 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsads2(unsigned int __a, unsigned int __b) {`。
- **L1472 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1472 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1473 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1473 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1474 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1474 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1475 EN**: Executes a call or declaration centered on `"r"`.
  **L1475 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1476 EN**: Returns from the current function with `r`.
  **L1476 CN**: 以 `r` 从当前函数返回。
- **L1477 EN**: Closes the current lexical scope or compound statement.
  **L1477 CN**: 结束当前词法作用域或复合语句块。
- **L1478 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsads4(unsigned int __a, unsigned int __b) {`.
  **L1478 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsads4(unsigned int __a, unsigned int __b) {`。
- **L1479 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1479 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1480 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1480 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1481 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1481 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1482 EN**: Executes a call or declaration centered on `"r"`.
  **L1482 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1483 EN**: Returns from the current function with `r`.
  **L1483 CN**: 以 `r` 从当前函数返回。
- **L1484 EN**: Closes the current lexical scope or compound statement.
  **L1484 CN**: 结束当前词法作用域或复合语句块。
- **L1485 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsadu2(unsigned int __a, unsigned int __b) {`.
  **L1485 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsadu2(unsigned int __a, unsigned int __b) {`。
- **L1486 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1486 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1487 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1487 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1488 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1488 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。

### Lines 1489-1512

````c
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vsadu4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vabsdiff4.u32.u32.u32.add %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}

__DEVICE__ unsigned int __vsub2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vsub2.u32.u32.u32 %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vneg2(unsigned int __a) { return __vsub2(0, __a); }

__DEVICE__ unsigned int __vsub4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vsub4.u32.u32.u32 %0,%1,%2,%3;"
          : "=r"(r)
````
- **L1489 EN**: Executes a call or declaration centered on `"r"`.
  **L1489 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1490 EN**: Returns from the current function with `r`.
  **L1490 CN**: 以 `r` 从当前函数返回。
- **L1491 EN**: Closes the current lexical scope or compound statement.
  **L1491 CN**: 结束当前词法作用域或复合语句块。
- **L1492 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsadu4(unsigned int __a, unsigned int __b) {`.
  **L1492 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsadu4(unsigned int __a, unsigned int __b) {`。
- **L1493 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1493 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1494 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1494 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1495 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1495 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1496 EN**: Executes a call or declaration centered on `"r"`.
  **L1496 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1497 EN**: Returns from the current function with `r`.
  **L1497 CN**: 以 `r` 从当前函数返回。
- **L1498 EN**: Closes the current lexical scope or compound statement.
  **L1498 CN**: 结束当前词法作用域或复合语句块。
- **L1499 EN**: Blank line separating nearby declarations or logic blocks.
  **L1499 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1500 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsub2(unsigned int __a, unsigned int __b) {`.
  **L1500 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsub2(unsigned int __a, unsigned int __b) {`。
- **L1501 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1501 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1502 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1502 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1503 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1503 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1504 EN**: Executes a call or declaration centered on `"r"`.
  **L1504 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1505 EN**: Returns from the current function with `r`.
  **L1505 CN**: 以 `r` 从当前函数返回。
- **L1506 EN**: Closes the current lexical scope or compound statement.
  **L1506 CN**: 结束当前词法作用域或复合语句块。
- **L1507 EN**: Continues logic associated with callable symbol `__vneg2`.
  **L1507 CN**: 继续与可调用符号 `__vneg2` 相关的逻辑。
- **L1508 EN**: Blank line separating nearby declarations or logic blocks.
  **L1508 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1509 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsub4(unsigned int __a, unsigned int __b) {`.
  **L1509 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsub4(unsigned int __a, unsigned int __b) {`。
- **L1510 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1510 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1511 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1511 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1512 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1512 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。

### Lines 1513-1536

````c
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vneg4(unsigned int __a) { return __vsub4(0, __a); }
__DEVICE__ unsigned int __vsubss2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vsub2.s32.s32.s32.sat %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vnegss2(unsigned int __a) {
  return __vsubss2(0, __a);
}
__DEVICE__ unsigned int __vsubss4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vsub4.s32.s32.s32.sat %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vnegss4(unsigned int __a) {
  return __vsubss4(0, __a);
}
````
- **L1513 EN**: Executes a call or declaration centered on `"r"`.
  **L1513 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1514 EN**: Returns from the current function with `r`.
  **L1514 CN**: 以 `r` 从当前函数返回。
- **L1515 EN**: Closes the current lexical scope or compound statement.
  **L1515 CN**: 结束当前词法作用域或复合语句块。
- **L1516 EN**: Continues logic associated with callable symbol `__vneg4`.
  **L1516 CN**: 继续与可调用符号 `__vneg4` 相关的逻辑。
- **L1517 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsubss2(unsigned int __a, unsigned int __b) {`.
  **L1517 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsubss2(unsigned int __a, unsigned int __b) {`。
- **L1518 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1518 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1519 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1519 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1520 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1520 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1521 EN**: Executes a call or declaration centered on `"r"`.
  **L1521 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1522 EN**: Returns from the current function with `r`.
  **L1522 CN**: 以 `r` 从当前函数返回。
- **L1523 EN**: Closes the current lexical scope or compound statement.
  **L1523 CN**: 结束当前词法作用域或复合语句块。
- **L1524 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vnegss2(unsigned int __a) {`.
  **L1524 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vnegss2(unsigned int __a) {`。
- **L1525 EN**: Returns from the current function with `__vsubss2(0, __a)`.
  **L1525 CN**: 以 `__vsubss2(0, __a)` 从当前函数返回。
- **L1526 EN**: Closes the current lexical scope or compound statement.
  **L1526 CN**: 结束当前词法作用域或复合语句块。
- **L1527 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsubss4(unsigned int __a, unsigned int __b) {`.
  **L1527 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsubss4(unsigned int __a, unsigned int __b) {`。
- **L1528 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1528 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1529 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1529 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1530 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1530 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1531 EN**: Executes a call or declaration centered on `"r"`.
  **L1531 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1532 EN**: Returns from the current function with `r`.
  **L1532 CN**: 以 `r` 从当前函数返回。
- **L1533 EN**: Closes the current lexical scope or compound statement.
  **L1533 CN**: 结束当前词法作用域或复合语句块。
- **L1534 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vnegss4(unsigned int __a) {`.
  **L1534 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vnegss4(unsigned int __a) {`。
- **L1535 EN**: Returns from the current function with `__vsubss4(0, __a)`.
  **L1535 CN**: 以 `__vsubss4(0, __a)` 从当前函数返回。
- **L1536 EN**: Closes the current lexical scope or compound statement.
  **L1536 CN**: 结束当前词法作用域或复合语句块。

### Lines 1537-1560

````c
__DEVICE__ unsigned int __vsubus2(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vsub2.u32.u32.u32.sat %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
__DEVICE__ unsigned int __vsubus4(unsigned int __a, unsigned int __b) {
  unsigned int r;
  __asm__("vsub4.u32.u32.u32.sat %0,%1,%2,%3;"
          : "=r"(r)
          : "r"(__a), "r"(__b), "r"(0));
  return r;
}
#endif // CUDA_VERSION >= 9020

// For OpenMP we require the user to include <time.h> as we need to know what
// clock_t is on the system.
#ifndef __OPENMP_NVPTX__
__DEVICE__ /* clock_t= */ int clock() { return __nvvm_read_ptx_sreg_clock(); }
#endif
__DEVICE__ long long clock64() { return __nvvm_read_ptx_sreg_clock64(); }

// These functions shouldn't be declared when including this header
````
- **L1537 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsubus2(unsigned int __a, unsigned int __b) {`.
  **L1537 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsubus2(unsigned int __a, unsigned int __b) {`。
- **L1538 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1538 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1539 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1539 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1540 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1540 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1541 EN**: Executes a call or declaration centered on `"r"`.
  **L1541 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1542 EN**: Returns from the current function with `r`.
  **L1542 CN**: 以 `r` 从当前函数返回。
- **L1543 EN**: Closes the current lexical scope or compound statement.
  **L1543 CN**: 结束当前词法作用域或复合语句块。
- **L1544 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ unsigned int __vsubus4(unsigned int __a, unsigned int __b) {`.
  **L1544 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ unsigned int __vsubus4(unsigned int __a, unsigned int __b) {`。
- **L1545 EN**: Adds a standalone statement or declaration: `unsigned int r;`.
  **L1545 CN**: 添加一条独立语句或声明：`unsigned int r;`。
- **L1546 EN**: Continues logic associated with callable symbol `__asm__`.
  **L1546 CN**: 继续与可调用符号 `__asm__` 相关的逻辑。
- **L1547 EN**: Continues the surrounding expression or declaration: `: "=r"(r)`.
  **L1547 CN**: 继续构造周围的表达式或声明：`: "=r"(r)`。
- **L1548 EN**: Executes a call or declaration centered on `"r"`.
  **L1548 CN**: 执行以 `"r"` 为核心的调用或声明。
- **L1549 EN**: Returns from the current function with `r`.
  **L1549 CN**: 以 `r` 从当前函数返回。
- **L1550 EN**: Closes the current lexical scope or compound statement.
  **L1550 CN**: 结束当前词法作用域或复合语句块。
- **L1551 EN**: Closes the current preprocessor conditional block.
  **L1551 CN**: 结束当前预处理条件块。
- **L1552 EN**: Blank line separating nearby declarations or logic blocks.
  **L1552 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1553 EN**: Comment explains nearby logic, constraints, or intent: `For OpenMP we require the user to include <time.h> as we need to know what`.
  **L1553 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For OpenMP we require the user to include <time.h> as we need to know what`。
- **L1554 EN**: Comment explains nearby logic, constraints, or intent: `clock_t is on the system.`.
  **L1554 CN**: 注释解释附近代码的逻辑、约束或设计意图：`clock_t is on the system.`。
- **L1555 EN**: Starts a preprocessor conditional block: `#ifndef __OPENMP_NVPTX__`.
  **L1555 CN**: 开始一个预处理条件块：`#ifndef __OPENMP_NVPTX__`。
- **L1556 EN**: Continues logic associated with callable symbol `clock`.
  **L1556 CN**: 继续与可调用符号 `clock` 相关的逻辑。
- **L1557 EN**: Closes the current preprocessor conditional block.
  **L1557 CN**: 结束当前预处理条件块。
- **L1558 EN**: Continues logic associated with callable symbol `clock64`.
  **L1558 CN**: 继续与可调用符号 `clock64` 相关的逻辑。
- **L1559 EN**: Blank line separating nearby declarations or logic blocks.
  **L1559 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1560 EN**: Comment explains nearby logic, constraints, or intent: `These functions shouldn't be declared when including this header`.
  **L1560 CN**: 注释解释附近代码的逻辑、约束或设计意图：`These functions shouldn't be declared when including this header`。

### Lines 1561-1572

````c
// for math function resolution purposes.
#ifndef __OPENMP_NVPTX__
__DEVICE__ void *memcpy(void *__a, const void *__b, size_t __c) {
  return __builtin_memcpy(__a, __b, __c);
}
__DEVICE__ void *memset(void *__a, int __b, size_t __c) {
  return __builtin_memset(__a, __b, __c);
}
#endif

#pragma pop_macro("__DEVICE__")
#endif // __CLANG_CUDA_DEVICE_FUNCTIONS_H__
````
- **L1561 EN**: Comment explains nearby logic, constraints, or intent: `for math function resolution purposes.`.
  **L1561 CN**: 注释解释附近代码的逻辑、约束或设计意图：`for math function resolution purposes.`。
- **L1562 EN**: Starts a preprocessor conditional block: `#ifndef __OPENMP_NVPTX__`.
  **L1562 CN**: 开始一个预处理条件块：`#ifndef __OPENMP_NVPTX__`。
- **L1563 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ void *memcpy(void *__a, const void *__b, size_t __c) {`.
  **L1563 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ void *memcpy(void *__a, const void *__b, size_t __c) {`。
- **L1564 EN**: Returns from the current function with `__builtin_memcpy(__a, __b, __c)`.
  **L1564 CN**: 以 `__builtin_memcpy(__a, __b, __c)` 从当前函数返回。
- **L1565 EN**: Closes the current lexical scope or compound statement.
  **L1565 CN**: 结束当前词法作用域或复合语句块。
- **L1566 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `__DEVICE__ void *memset(void *__a, int __b, size_t __c) {`.
  **L1566 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`__DEVICE__ void *memset(void *__a, int __b, size_t __c) {`。
- **L1567 EN**: Returns from the current function with `__builtin_memset(__a, __b, __c)`.
  **L1567 CN**: 以 `__builtin_memset(__a, __b, __c)` 从当前函数返回。
- **L1568 EN**: Closes the current lexical scope or compound statement.
  **L1568 CN**: 结束当前词法作用域或复合语句块。
- **L1569 EN**: Closes the current preprocessor conditional block.
  **L1569 CN**: 结束当前预处理条件块。
- **L1570 EN**: Blank line separating nearby declarations or logic blocks.
  **L1570 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1571 EN**: Applies a compiler pragma that adjusts parsing, attributes, or macro state: `#pragma pop_macro("__DEVICE__")`.
  **L1571 CN**: 应用编译器 pragma，调整解析、属性或宏状态：`#pragma pop_macro("__DEVICE__")`。
- **L1572 EN**: Closes the current preprocessor conditional block.
  **L1572 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **CUDA device support / CUDA 设备支持**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**
- **Compiler-specific attributes / 编译器特定属性**
- **Pragma-controlled behavior / 由 pragma 控制的行为**
- **GPU device annotations / GPU 设备注解**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None. / 无。
- **Conditional macros / 条件宏**: `__CLANG_CUDA_DEVICE_FUNCTIONS_H__`, `__OPENMP_NVPTX__`, `__cplusplus`, `_MSC_VER`
- **External builtins / 外部 builtin**: `__nv_brev`, `__nv_brevll`, `__nv_byte_perm`, `__nv_clz`, `__nv_clzll`, `__nv_fast_cosf`, `__nv_dadd_rd`, `__nv_dadd_rn`, `__nv_dadd_ru`, `__nv_dadd_rz`, `__nv_ddiv_rd`, `__nv_ddiv_rn`
