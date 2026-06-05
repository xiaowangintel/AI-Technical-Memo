# __clang_cuda_libdevice_declares.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/__clang_cuda_libdevice_declares.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: decls for libdevice functions.
- **Purpose (CN)**: 该头文件主要作用是：decls for libdevice functions。
- **Line Count / 行数**: 468

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````c
/*===-- __clang_cuda_libdevice_declares.h - decls for libdevice functions --===
 *
 * Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
 * See https://llvm.org/LICENSE.txt for license information.
 * SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
 *
 *===-----------------------------------------------------------------------===
 */

#ifndef __CLANG_CUDA_LIBDEVICE_DECLARES_H__
#define __CLANG_CUDA_LIBDEVICE_DECLARES_H__

#if defined(__cplusplus)
extern "C" {
#endif

#if defined(__OPENMP_NVPTX__)
#define __DEVICE__
#pragma omp begin assumes ext_spmd_amenable no_openmp
#elif defined(__CUDA__)
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
- **L10 EN**: Starts a preprocessor conditional block: `#ifndef __CLANG_CUDA_LIBDEVICE_DECLARES_H__`.
  **L10 CN**: 开始一个预处理条件块：`#ifndef __CLANG_CUDA_LIBDEVICE_DECLARES_H__`。
- **L11 EN**: Defines macro `__CLANG_CUDA_LIBDEVICE_DECLARES_H__` for conditional compilation, shorthand, or API generation.
  **L11 CN**: 定义宏 `__CLANG_CUDA_LIBDEVICE_DECLARES_H__`，用于条件编译、简写或 API 生成。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Starts a preprocessor conditional block: `#if defined(__cplusplus)`.
  **L13 CN**: 开始一个预处理条件块：`#if defined(__cplusplus)`。
- **L14 EN**: Switches the following declarations to C linkage.
  **L14 CN**: 将后续声明切换为 C 链接方式。
- **L15 EN**: Closes the current preprocessor conditional block.
  **L15 CN**: 结束当前预处理条件块。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L17 EN**: Starts a preprocessor conditional block: `#if defined(__OPENMP_NVPTX__)`.
  **L17 CN**: 开始一个预处理条件块：`#if defined(__OPENMP_NVPTX__)`。
- **L18 EN**: Defines macro `__DEVICE__` for conditional compilation, shorthand, or API generation.
  **L18 CN**: 定义宏 `__DEVICE__`，用于条件编译、简写或 API 生成。
- **L19 EN**: Applies a compiler pragma that adjusts parsing, attributes, or macro state: `#pragma omp begin assumes ext_spmd_amenable no_openmp`.
  **L19 CN**: 应用编译器 pragma，调整解析、属性或宏状态：`#pragma omp begin assumes ext_spmd_amenable no_openmp`。
- **L20 EN**: Continues preprocessor branch selection for the active target or feature set.
  **L20 CN**: 继续为当前目标或特性集合选择预处理分支。

### Lines 21-40

````c
#define __DEVICE__ __device__
#endif

__DEVICE__ int __nv_abs(int __a);
__DEVICE__ double __nv_acos(double __a);
__DEVICE__ float __nv_acosf(float __a);
__DEVICE__ double __nv_acosh(double __a);
__DEVICE__ float __nv_acoshf(float __a);
__DEVICE__ double __nv_asin(double __a);
__DEVICE__ float __nv_asinf(float __a);
__DEVICE__ double __nv_asinh(double __a);
__DEVICE__ float __nv_asinhf(float __a);
__DEVICE__ double __nv_atan2(double __a, double __b);
__DEVICE__ float __nv_atan2f(float __a, float __b);
__DEVICE__ double __nv_atan(double __a);
__DEVICE__ float __nv_atanf(float __a);
__DEVICE__ double __nv_atanh(double __a);
__DEVICE__ float __nv_atanhf(float __a);
__DEVICE__ int __nv_brev(int __a);
__DEVICE__ long long __nv_brevll(long long __a);
````
- **L21 EN**: Defines macro `__DEVICE__` for conditional compilation, shorthand, or API generation.
  **L21 CN**: 定义宏 `__DEVICE__`，用于条件编译、简写或 API 生成。
- **L22 EN**: Closes the current preprocessor conditional block.
  **L22 CN**: 结束当前预处理条件块。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L24 EN**: Executes a call or declaration centered on `__nv_abs`.
  **L24 CN**: 执行以 `__nv_abs` 为核心的调用或声明。
- **L25 EN**: Executes a call or declaration centered on `__nv_acos`.
  **L25 CN**: 执行以 `__nv_acos` 为核心的调用或声明。
- **L26 EN**: Executes a call or declaration centered on `__nv_acosf`.
  **L26 CN**: 执行以 `__nv_acosf` 为核心的调用或声明。
- **L27 EN**: Executes a call or declaration centered on `__nv_acosh`.
  **L27 CN**: 执行以 `__nv_acosh` 为核心的调用或声明。
- **L28 EN**: Executes a call or declaration centered on `__nv_acoshf`.
  **L28 CN**: 执行以 `__nv_acoshf` 为核心的调用或声明。
- **L29 EN**: Executes a call or declaration centered on `__nv_asin`.
  **L29 CN**: 执行以 `__nv_asin` 为核心的调用或声明。
- **L30 EN**: Executes a call or declaration centered on `__nv_asinf`.
  **L30 CN**: 执行以 `__nv_asinf` 为核心的调用或声明。
- **L31 EN**: Executes a call or declaration centered on `__nv_asinh`.
  **L31 CN**: 执行以 `__nv_asinh` 为核心的调用或声明。
- **L32 EN**: Executes a call or declaration centered on `__nv_asinhf`.
  **L32 CN**: 执行以 `__nv_asinhf` 为核心的调用或声明。
- **L33 EN**: Executes a call or declaration centered on `__nv_atan2`.
  **L33 CN**: 执行以 `__nv_atan2` 为核心的调用或声明。
- **L34 EN**: Executes a call or declaration centered on `__nv_atan2f`.
  **L34 CN**: 执行以 `__nv_atan2f` 为核心的调用或声明。
- **L35 EN**: Executes a call or declaration centered on `__nv_atan`.
  **L35 CN**: 执行以 `__nv_atan` 为核心的调用或声明。
- **L36 EN**: Executes a call or declaration centered on `__nv_atanf`.
  **L36 CN**: 执行以 `__nv_atanf` 为核心的调用或声明。
- **L37 EN**: Executes a call or declaration centered on `__nv_atanh`.
  **L37 CN**: 执行以 `__nv_atanh` 为核心的调用或声明。
- **L38 EN**: Executes a call or declaration centered on `__nv_atanhf`.
  **L38 CN**: 执行以 `__nv_atanhf` 为核心的调用或声明。
- **L39 EN**: Executes a call or declaration centered on `__nv_brev`.
  **L39 CN**: 执行以 `__nv_brev` 为核心的调用或声明。
- **L40 EN**: Executes a call or declaration centered on `__nv_brevll`.
  **L40 CN**: 执行以 `__nv_brevll` 为核心的调用或声明。

### Lines 41-60

````c
__DEVICE__ int __nv_byte_perm(int __a, int __b, int __c);
__DEVICE__ double __nv_cbrt(double __a);
__DEVICE__ float __nv_cbrtf(float __a);
__DEVICE__ double __nv_ceil(double __a);
__DEVICE__ float __nv_ceilf(float __a);
__DEVICE__ int __nv_clz(int __a);
__DEVICE__ int __nv_clzll(long long __a);
__DEVICE__ double __nv_copysign(double __a, double __b);
__DEVICE__ float __nv_copysignf(float __a, float __b);
__DEVICE__ double __nv_cos(double __a);
__DEVICE__ float __nv_cosf(float __a);
__DEVICE__ double __nv_cosh(double __a);
__DEVICE__ float __nv_coshf(float __a);
__DEVICE__ double __nv_cospi(double __a);
__DEVICE__ float __nv_cospif(float __a);
__DEVICE__ double __nv_cyl_bessel_i0(double __a);
__DEVICE__ float __nv_cyl_bessel_i0f(float __a);
__DEVICE__ double __nv_cyl_bessel_i1(double __a);
__DEVICE__ float __nv_cyl_bessel_i1f(float __a);
__DEVICE__ double __nv_dadd_rd(double __a, double __b);
````
- **L41 EN**: Executes a call or declaration centered on `__nv_byte_perm`.
  **L41 CN**: 执行以 `__nv_byte_perm` 为核心的调用或声明。
- **L42 EN**: Executes a call or declaration centered on `__nv_cbrt`.
  **L42 CN**: 执行以 `__nv_cbrt` 为核心的调用或声明。
- **L43 EN**: Executes a call or declaration centered on `__nv_cbrtf`.
  **L43 CN**: 执行以 `__nv_cbrtf` 为核心的调用或声明。
- **L44 EN**: Executes a call or declaration centered on `__nv_ceil`.
  **L44 CN**: 执行以 `__nv_ceil` 为核心的调用或声明。
- **L45 EN**: Executes a call or declaration centered on `__nv_ceilf`.
  **L45 CN**: 执行以 `__nv_ceilf` 为核心的调用或声明。
- **L46 EN**: Executes a call or declaration centered on `__nv_clz`.
  **L46 CN**: 执行以 `__nv_clz` 为核心的调用或声明。
- **L47 EN**: Executes a call or declaration centered on `__nv_clzll`.
  **L47 CN**: 执行以 `__nv_clzll` 为核心的调用或声明。
- **L48 EN**: Executes a call or declaration centered on `__nv_copysign`.
  **L48 CN**: 执行以 `__nv_copysign` 为核心的调用或声明。
- **L49 EN**: Executes a call or declaration centered on `__nv_copysignf`.
  **L49 CN**: 执行以 `__nv_copysignf` 为核心的调用或声明。
- **L50 EN**: Executes a call or declaration centered on `__nv_cos`.
  **L50 CN**: 执行以 `__nv_cos` 为核心的调用或声明。
- **L51 EN**: Executes a call or declaration centered on `__nv_cosf`.
  **L51 CN**: 执行以 `__nv_cosf` 为核心的调用或声明。
- **L52 EN**: Executes a call or declaration centered on `__nv_cosh`.
  **L52 CN**: 执行以 `__nv_cosh` 为核心的调用或声明。
- **L53 EN**: Executes a call or declaration centered on `__nv_coshf`.
  **L53 CN**: 执行以 `__nv_coshf` 为核心的调用或声明。
- **L54 EN**: Executes a call or declaration centered on `__nv_cospi`.
  **L54 CN**: 执行以 `__nv_cospi` 为核心的调用或声明。
- **L55 EN**: Executes a call or declaration centered on `__nv_cospif`.
  **L55 CN**: 执行以 `__nv_cospif` 为核心的调用或声明。
- **L56 EN**: Executes a call or declaration centered on `__nv_cyl_bessel_i0`.
  **L56 CN**: 执行以 `__nv_cyl_bessel_i0` 为核心的调用或声明。
- **L57 EN**: Executes a call or declaration centered on `__nv_cyl_bessel_i0f`.
  **L57 CN**: 执行以 `__nv_cyl_bessel_i0f` 为核心的调用或声明。
- **L58 EN**: Executes a call or declaration centered on `__nv_cyl_bessel_i1`.
  **L58 CN**: 执行以 `__nv_cyl_bessel_i1` 为核心的调用或声明。
- **L59 EN**: Executes a call or declaration centered on `__nv_cyl_bessel_i1f`.
  **L59 CN**: 执行以 `__nv_cyl_bessel_i1f` 为核心的调用或声明。
- **L60 EN**: Executes a call or declaration centered on `__nv_dadd_rd`.
  **L60 CN**: 执行以 `__nv_dadd_rd` 为核心的调用或声明。

### Lines 61-80

````c
__DEVICE__ double __nv_dadd_rn(double __a, double __b);
__DEVICE__ double __nv_dadd_ru(double __a, double __b);
__DEVICE__ double __nv_dadd_rz(double __a, double __b);
__DEVICE__ double __nv_ddiv_rd(double __a, double __b);
__DEVICE__ double __nv_ddiv_rn(double __a, double __b);
__DEVICE__ double __nv_ddiv_ru(double __a, double __b);
__DEVICE__ double __nv_ddiv_rz(double __a, double __b);
__DEVICE__ double __nv_dmul_rd(double __a, double __b);
__DEVICE__ double __nv_dmul_rn(double __a, double __b);
__DEVICE__ double __nv_dmul_ru(double __a, double __b);
__DEVICE__ double __nv_dmul_rz(double __a, double __b);
__DEVICE__ float __nv_double2float_rd(double __a);
__DEVICE__ float __nv_double2float_rn(double __a);
__DEVICE__ float __nv_double2float_ru(double __a);
__DEVICE__ float __nv_double2float_rz(double __a);
__DEVICE__ int __nv_double2hiint(double __a);
__DEVICE__ int __nv_double2int_rd(double __a);
__DEVICE__ int __nv_double2int_rn(double __a);
__DEVICE__ int __nv_double2int_ru(double __a);
__DEVICE__ int __nv_double2int_rz(double __a);
````
- **L61 EN**: Executes a call or declaration centered on `__nv_dadd_rn`.
  **L61 CN**: 执行以 `__nv_dadd_rn` 为核心的调用或声明。
- **L62 EN**: Executes a call or declaration centered on `__nv_dadd_ru`.
  **L62 CN**: 执行以 `__nv_dadd_ru` 为核心的调用或声明。
- **L63 EN**: Executes a call or declaration centered on `__nv_dadd_rz`.
  **L63 CN**: 执行以 `__nv_dadd_rz` 为核心的调用或声明。
- **L64 EN**: Executes a call or declaration centered on `__nv_ddiv_rd`.
  **L64 CN**: 执行以 `__nv_ddiv_rd` 为核心的调用或声明。
- **L65 EN**: Executes a call or declaration centered on `__nv_ddiv_rn`.
  **L65 CN**: 执行以 `__nv_ddiv_rn` 为核心的调用或声明。
- **L66 EN**: Executes a call or declaration centered on `__nv_ddiv_ru`.
  **L66 CN**: 执行以 `__nv_ddiv_ru` 为核心的调用或声明。
- **L67 EN**: Executes a call or declaration centered on `__nv_ddiv_rz`.
  **L67 CN**: 执行以 `__nv_ddiv_rz` 为核心的调用或声明。
- **L68 EN**: Executes a call or declaration centered on `__nv_dmul_rd`.
  **L68 CN**: 执行以 `__nv_dmul_rd` 为核心的调用或声明。
- **L69 EN**: Executes a call or declaration centered on `__nv_dmul_rn`.
  **L69 CN**: 执行以 `__nv_dmul_rn` 为核心的调用或声明。
- **L70 EN**: Executes a call or declaration centered on `__nv_dmul_ru`.
  **L70 CN**: 执行以 `__nv_dmul_ru` 为核心的调用或声明。
- **L71 EN**: Executes a call or declaration centered on `__nv_dmul_rz`.
  **L71 CN**: 执行以 `__nv_dmul_rz` 为核心的调用或声明。
- **L72 EN**: Executes a call or declaration centered on `__nv_double2float_rd`.
  **L72 CN**: 执行以 `__nv_double2float_rd` 为核心的调用或声明。
- **L73 EN**: Executes a call or declaration centered on `__nv_double2float_rn`.
  **L73 CN**: 执行以 `__nv_double2float_rn` 为核心的调用或声明。
- **L74 EN**: Executes a call or declaration centered on `__nv_double2float_ru`.
  **L74 CN**: 执行以 `__nv_double2float_ru` 为核心的调用或声明。
- **L75 EN**: Executes a call or declaration centered on `__nv_double2float_rz`.
  **L75 CN**: 执行以 `__nv_double2float_rz` 为核心的调用或声明。
- **L76 EN**: Executes a call or declaration centered on `__nv_double2hiint`.
  **L76 CN**: 执行以 `__nv_double2hiint` 为核心的调用或声明。
- **L77 EN**: Executes a call or declaration centered on `__nv_double2int_rd`.
  **L77 CN**: 执行以 `__nv_double2int_rd` 为核心的调用或声明。
- **L78 EN**: Executes a call or declaration centered on `__nv_double2int_rn`.
  **L78 CN**: 执行以 `__nv_double2int_rn` 为核心的调用或声明。
- **L79 EN**: Executes a call or declaration centered on `__nv_double2int_ru`.
  **L79 CN**: 执行以 `__nv_double2int_ru` 为核心的调用或声明。
- **L80 EN**: Executes a call or declaration centered on `__nv_double2int_rz`.
  **L80 CN**: 执行以 `__nv_double2int_rz` 为核心的调用或声明。

### Lines 81-100

````c
__DEVICE__ long long __nv_double2ll_rd(double __a);
__DEVICE__ long long __nv_double2ll_rn(double __a);
__DEVICE__ long long __nv_double2ll_ru(double __a);
__DEVICE__ long long __nv_double2ll_rz(double __a);
__DEVICE__ int __nv_double2loint(double __a);
__DEVICE__ unsigned int __nv_double2uint_rd(double __a);
__DEVICE__ unsigned int __nv_double2uint_rn(double __a);
__DEVICE__ unsigned int __nv_double2uint_ru(double __a);
__DEVICE__ unsigned int __nv_double2uint_rz(double __a);
__DEVICE__ unsigned long long __nv_double2ull_rd(double __a);
__DEVICE__ unsigned long long __nv_double2ull_rn(double __a);
__DEVICE__ unsigned long long __nv_double2ull_ru(double __a);
__DEVICE__ unsigned long long __nv_double2ull_rz(double __a);
__DEVICE__ unsigned long long __nv_double_as_longlong(double __a);
__DEVICE__ double __nv_drcp_rd(double __a);
__DEVICE__ double __nv_drcp_rn(double __a);
__DEVICE__ double __nv_drcp_ru(double __a);
__DEVICE__ double __nv_drcp_rz(double __a);
__DEVICE__ double __nv_dsqrt_rd(double __a);
__DEVICE__ double __nv_dsqrt_rn(double __a);
````
- **L81 EN**: Executes a call or declaration centered on `__nv_double2ll_rd`.
  **L81 CN**: 执行以 `__nv_double2ll_rd` 为核心的调用或声明。
- **L82 EN**: Executes a call or declaration centered on `__nv_double2ll_rn`.
  **L82 CN**: 执行以 `__nv_double2ll_rn` 为核心的调用或声明。
- **L83 EN**: Executes a call or declaration centered on `__nv_double2ll_ru`.
  **L83 CN**: 执行以 `__nv_double2ll_ru` 为核心的调用或声明。
- **L84 EN**: Executes a call or declaration centered on `__nv_double2ll_rz`.
  **L84 CN**: 执行以 `__nv_double2ll_rz` 为核心的调用或声明。
- **L85 EN**: Executes a call or declaration centered on `__nv_double2loint`.
  **L85 CN**: 执行以 `__nv_double2loint` 为核心的调用或声明。
- **L86 EN**: Executes a call or declaration centered on `__nv_double2uint_rd`.
  **L86 CN**: 执行以 `__nv_double2uint_rd` 为核心的调用或声明。
- **L87 EN**: Executes a call or declaration centered on `__nv_double2uint_rn`.
  **L87 CN**: 执行以 `__nv_double2uint_rn` 为核心的调用或声明。
- **L88 EN**: Executes a call or declaration centered on `__nv_double2uint_ru`.
  **L88 CN**: 执行以 `__nv_double2uint_ru` 为核心的调用或声明。
- **L89 EN**: Executes a call or declaration centered on `__nv_double2uint_rz`.
  **L89 CN**: 执行以 `__nv_double2uint_rz` 为核心的调用或声明。
- **L90 EN**: Executes a call or declaration centered on `__nv_double2ull_rd`.
  **L90 CN**: 执行以 `__nv_double2ull_rd` 为核心的调用或声明。
- **L91 EN**: Executes a call or declaration centered on `__nv_double2ull_rn`.
  **L91 CN**: 执行以 `__nv_double2ull_rn` 为核心的调用或声明。
- **L92 EN**: Executes a call or declaration centered on `__nv_double2ull_ru`.
  **L92 CN**: 执行以 `__nv_double2ull_ru` 为核心的调用或声明。
- **L93 EN**: Executes a call or declaration centered on `__nv_double2ull_rz`.
  **L93 CN**: 执行以 `__nv_double2ull_rz` 为核心的调用或声明。
- **L94 EN**: Executes a call or declaration centered on `__nv_double_as_longlong`.
  **L94 CN**: 执行以 `__nv_double_as_longlong` 为核心的调用或声明。
- **L95 EN**: Executes a call or declaration centered on `__nv_drcp_rd`.
  **L95 CN**: 执行以 `__nv_drcp_rd` 为核心的调用或声明。
- **L96 EN**: Executes a call or declaration centered on `__nv_drcp_rn`.
  **L96 CN**: 执行以 `__nv_drcp_rn` 为核心的调用或声明。
- **L97 EN**: Executes a call or declaration centered on `__nv_drcp_ru`.
  **L97 CN**: 执行以 `__nv_drcp_ru` 为核心的调用或声明。
- **L98 EN**: Executes a call or declaration centered on `__nv_drcp_rz`.
  **L98 CN**: 执行以 `__nv_drcp_rz` 为核心的调用或声明。
- **L99 EN**: Executes a call or declaration centered on `__nv_dsqrt_rd`.
  **L99 CN**: 执行以 `__nv_dsqrt_rd` 为核心的调用或声明。
- **L100 EN**: Executes a call or declaration centered on `__nv_dsqrt_rn`.
  **L100 CN**: 执行以 `__nv_dsqrt_rn` 为核心的调用或声明。

### Lines 101-120

````c
__DEVICE__ double __nv_dsqrt_ru(double __a);
__DEVICE__ double __nv_dsqrt_rz(double __a);
__DEVICE__ double __nv_dsub_rd(double __a, double __b);
__DEVICE__ double __nv_dsub_rn(double __a, double __b);
__DEVICE__ double __nv_dsub_ru(double __a, double __b);
__DEVICE__ double __nv_dsub_rz(double __a, double __b);
__DEVICE__ double __nv_erfc(double __a);
__DEVICE__ float __nv_erfcf(float __a);
__DEVICE__ double __nv_erfcinv(double __a);
__DEVICE__ float __nv_erfcinvf(float __a);
__DEVICE__ double __nv_erfcx(double __a);
__DEVICE__ float __nv_erfcxf(float __a);
__DEVICE__ double __nv_erf(double __a);
__DEVICE__ float __nv_erff(float __a);
__DEVICE__ double __nv_erfinv(double __a);
__DEVICE__ float __nv_erfinvf(float __a);
__DEVICE__ double __nv_exp10(double __a);
__DEVICE__ float __nv_exp10f(float __a);
__DEVICE__ double __nv_exp2(double __a);
__DEVICE__ float __nv_exp2f(float __a);
````
- **L101 EN**: Executes a call or declaration centered on `__nv_dsqrt_ru`.
  **L101 CN**: 执行以 `__nv_dsqrt_ru` 为核心的调用或声明。
- **L102 EN**: Executes a call or declaration centered on `__nv_dsqrt_rz`.
  **L102 CN**: 执行以 `__nv_dsqrt_rz` 为核心的调用或声明。
- **L103 EN**: Executes a call or declaration centered on `__nv_dsub_rd`.
  **L103 CN**: 执行以 `__nv_dsub_rd` 为核心的调用或声明。
- **L104 EN**: Executes a call or declaration centered on `__nv_dsub_rn`.
  **L104 CN**: 执行以 `__nv_dsub_rn` 为核心的调用或声明。
- **L105 EN**: Executes a call or declaration centered on `__nv_dsub_ru`.
  **L105 CN**: 执行以 `__nv_dsub_ru` 为核心的调用或声明。
- **L106 EN**: Executes a call or declaration centered on `__nv_dsub_rz`.
  **L106 CN**: 执行以 `__nv_dsub_rz` 为核心的调用或声明。
- **L107 EN**: Executes a call or declaration centered on `__nv_erfc`.
  **L107 CN**: 执行以 `__nv_erfc` 为核心的调用或声明。
- **L108 EN**: Executes a call or declaration centered on `__nv_erfcf`.
  **L108 CN**: 执行以 `__nv_erfcf` 为核心的调用或声明。
- **L109 EN**: Executes a call or declaration centered on `__nv_erfcinv`.
  **L109 CN**: 执行以 `__nv_erfcinv` 为核心的调用或声明。
- **L110 EN**: Executes a call or declaration centered on `__nv_erfcinvf`.
  **L110 CN**: 执行以 `__nv_erfcinvf` 为核心的调用或声明。
- **L111 EN**: Executes a call or declaration centered on `__nv_erfcx`.
  **L111 CN**: 执行以 `__nv_erfcx` 为核心的调用或声明。
- **L112 EN**: Executes a call or declaration centered on `__nv_erfcxf`.
  **L112 CN**: 执行以 `__nv_erfcxf` 为核心的调用或声明。
- **L113 EN**: Executes a call or declaration centered on `__nv_erf`.
  **L113 CN**: 执行以 `__nv_erf` 为核心的调用或声明。
- **L114 EN**: Executes a call or declaration centered on `__nv_erff`.
  **L114 CN**: 执行以 `__nv_erff` 为核心的调用或声明。
- **L115 EN**: Executes a call or declaration centered on `__nv_erfinv`.
  **L115 CN**: 执行以 `__nv_erfinv` 为核心的调用或声明。
- **L116 EN**: Executes a call or declaration centered on `__nv_erfinvf`.
  **L116 CN**: 执行以 `__nv_erfinvf` 为核心的调用或声明。
- **L117 EN**: Executes a call or declaration centered on `__nv_exp10`.
  **L117 CN**: 执行以 `__nv_exp10` 为核心的调用或声明。
- **L118 EN**: Executes a call or declaration centered on `__nv_exp10f`.
  **L118 CN**: 执行以 `__nv_exp10f` 为核心的调用或声明。
- **L119 EN**: Executes a call or declaration centered on `__nv_exp2`.
  **L119 CN**: 执行以 `__nv_exp2` 为核心的调用或声明。
- **L120 EN**: Executes a call or declaration centered on `__nv_exp2f`.
  **L120 CN**: 执行以 `__nv_exp2f` 为核心的调用或声明。

### Lines 121-140

````c
__DEVICE__ double __nv_exp(double __a);
__DEVICE__ float __nv_expf(float __a);
__DEVICE__ double __nv_expm1(double __a);
__DEVICE__ float __nv_expm1f(float __a);
__DEVICE__ double __nv_fabs(double __a);
__DEVICE__ float __nv_fabsf(float __a);
__DEVICE__ float __nv_fadd_rd(float __a, float __b);
__DEVICE__ float __nv_fadd_rn(float __a, float __b);
__DEVICE__ float __nv_fadd_ru(float __a, float __b);
__DEVICE__ float __nv_fadd_rz(float __a, float __b);
__DEVICE__ float __nv_fast_cosf(float __a);
__DEVICE__ float __nv_fast_exp10f(float __a);
__DEVICE__ float __nv_fast_expf(float __a);
__DEVICE__ float __nv_fast_fdividef(float __a, float __b);
__DEVICE__ float __nv_fast_log10f(float __a);
__DEVICE__ float __nv_fast_log2f(float __a);
__DEVICE__ float __nv_fast_logf(float __a);
__DEVICE__ float __nv_fast_powf(float __a, float __b);
__DEVICE__ void __nv_fast_sincosf(float __a, float *__s, float *__c);
__DEVICE__ float __nv_fast_sinf(float __a);
````
- **L121 EN**: Executes a call or declaration centered on `__nv_exp`.
  **L121 CN**: 执行以 `__nv_exp` 为核心的调用或声明。
- **L122 EN**: Executes a call or declaration centered on `__nv_expf`.
  **L122 CN**: 执行以 `__nv_expf` 为核心的调用或声明。
- **L123 EN**: Executes a call or declaration centered on `__nv_expm1`.
  **L123 CN**: 执行以 `__nv_expm1` 为核心的调用或声明。
- **L124 EN**: Executes a call or declaration centered on `__nv_expm1f`.
  **L124 CN**: 执行以 `__nv_expm1f` 为核心的调用或声明。
- **L125 EN**: Executes a call or declaration centered on `__nv_fabs`.
  **L125 CN**: 执行以 `__nv_fabs` 为核心的调用或声明。
- **L126 EN**: Executes a call or declaration centered on `__nv_fabsf`.
  **L126 CN**: 执行以 `__nv_fabsf` 为核心的调用或声明。
- **L127 EN**: Executes a call or declaration centered on `__nv_fadd_rd`.
  **L127 CN**: 执行以 `__nv_fadd_rd` 为核心的调用或声明。
- **L128 EN**: Executes a call or declaration centered on `__nv_fadd_rn`.
  **L128 CN**: 执行以 `__nv_fadd_rn` 为核心的调用或声明。
- **L129 EN**: Executes a call or declaration centered on `__nv_fadd_ru`.
  **L129 CN**: 执行以 `__nv_fadd_ru` 为核心的调用或声明。
- **L130 EN**: Executes a call or declaration centered on `__nv_fadd_rz`.
  **L130 CN**: 执行以 `__nv_fadd_rz` 为核心的调用或声明。
- **L131 EN**: Executes a call or declaration centered on `__nv_fast_cosf`.
  **L131 CN**: 执行以 `__nv_fast_cosf` 为核心的调用或声明。
- **L132 EN**: Executes a call or declaration centered on `__nv_fast_exp10f`.
  **L132 CN**: 执行以 `__nv_fast_exp10f` 为核心的调用或声明。
- **L133 EN**: Executes a call or declaration centered on `__nv_fast_expf`.
  **L133 CN**: 执行以 `__nv_fast_expf` 为核心的调用或声明。
- **L134 EN**: Executes a call or declaration centered on `__nv_fast_fdividef`.
  **L134 CN**: 执行以 `__nv_fast_fdividef` 为核心的调用或声明。
- **L135 EN**: Executes a call or declaration centered on `__nv_fast_log10f`.
  **L135 CN**: 执行以 `__nv_fast_log10f` 为核心的调用或声明。
- **L136 EN**: Executes a call or declaration centered on `__nv_fast_log2f`.
  **L136 CN**: 执行以 `__nv_fast_log2f` 为核心的调用或声明。
- **L137 EN**: Executes a call or declaration centered on `__nv_fast_logf`.
  **L137 CN**: 执行以 `__nv_fast_logf` 为核心的调用或声明。
- **L138 EN**: Executes a call or declaration centered on `__nv_fast_powf`.
  **L138 CN**: 执行以 `__nv_fast_powf` 为核心的调用或声明。
- **L139 EN**: Executes a call or declaration centered on `__nv_fast_sincosf`.
  **L139 CN**: 执行以 `__nv_fast_sincosf` 为核心的调用或声明。
- **L140 EN**: Executes a call or declaration centered on `__nv_fast_sinf`.
  **L140 CN**: 执行以 `__nv_fast_sinf` 为核心的调用或声明。

### Lines 141-160

````c
__DEVICE__ float __nv_fast_tanf(float __a);
__DEVICE__ double __nv_fdim(double __a, double __b);
__DEVICE__ float __nv_fdimf(float __a, float __b);
__DEVICE__ float __nv_fdiv_rd(float __a, float __b);
__DEVICE__ float __nv_fdiv_rn(float __a, float __b);
__DEVICE__ float __nv_fdiv_ru(float __a, float __b);
__DEVICE__ float __nv_fdiv_rz(float __a, float __b);
__DEVICE__ int __nv_ffs(int __a);
__DEVICE__ int __nv_ffsll(long long __a);
__DEVICE__ int __nv_finitef(float __a);
__DEVICE__ unsigned short __nv_float2half_rn(float __a);
__DEVICE__ int __nv_float2int_rd(float __a);
__DEVICE__ int __nv_float2int_rn(float __a);
__DEVICE__ int __nv_float2int_ru(float __a);
__DEVICE__ int __nv_float2int_rz(float __a);
__DEVICE__ long long __nv_float2ll_rd(float __a);
__DEVICE__ long long __nv_float2ll_rn(float __a);
__DEVICE__ long long __nv_float2ll_ru(float __a);
__DEVICE__ long long __nv_float2ll_rz(float __a);
__DEVICE__ unsigned int __nv_float2uint_rd(float __a);
````
- **L141 EN**: Executes a call or declaration centered on `__nv_fast_tanf`.
  **L141 CN**: 执行以 `__nv_fast_tanf` 为核心的调用或声明。
- **L142 EN**: Executes a call or declaration centered on `__nv_fdim`.
  **L142 CN**: 执行以 `__nv_fdim` 为核心的调用或声明。
- **L143 EN**: Executes a call or declaration centered on `__nv_fdimf`.
  **L143 CN**: 执行以 `__nv_fdimf` 为核心的调用或声明。
- **L144 EN**: Executes a call or declaration centered on `__nv_fdiv_rd`.
  **L144 CN**: 执行以 `__nv_fdiv_rd` 为核心的调用或声明。
- **L145 EN**: Executes a call or declaration centered on `__nv_fdiv_rn`.
  **L145 CN**: 执行以 `__nv_fdiv_rn` 为核心的调用或声明。
- **L146 EN**: Executes a call or declaration centered on `__nv_fdiv_ru`.
  **L146 CN**: 执行以 `__nv_fdiv_ru` 为核心的调用或声明。
- **L147 EN**: Executes a call or declaration centered on `__nv_fdiv_rz`.
  **L147 CN**: 执行以 `__nv_fdiv_rz` 为核心的调用或声明。
- **L148 EN**: Executes a call or declaration centered on `__nv_ffs`.
  **L148 CN**: 执行以 `__nv_ffs` 为核心的调用或声明。
- **L149 EN**: Executes a call or declaration centered on `__nv_ffsll`.
  **L149 CN**: 执行以 `__nv_ffsll` 为核心的调用或声明。
- **L150 EN**: Executes a call or declaration centered on `__nv_finitef`.
  **L150 CN**: 执行以 `__nv_finitef` 为核心的调用或声明。
- **L151 EN**: Executes a call or declaration centered on `__nv_float2half_rn`.
  **L151 CN**: 执行以 `__nv_float2half_rn` 为核心的调用或声明。
- **L152 EN**: Executes a call or declaration centered on `__nv_float2int_rd`.
  **L152 CN**: 执行以 `__nv_float2int_rd` 为核心的调用或声明。
- **L153 EN**: Executes a call or declaration centered on `__nv_float2int_rn`.
  **L153 CN**: 执行以 `__nv_float2int_rn` 为核心的调用或声明。
- **L154 EN**: Executes a call or declaration centered on `__nv_float2int_ru`.
  **L154 CN**: 执行以 `__nv_float2int_ru` 为核心的调用或声明。
- **L155 EN**: Executes a call or declaration centered on `__nv_float2int_rz`.
  **L155 CN**: 执行以 `__nv_float2int_rz` 为核心的调用或声明。
- **L156 EN**: Executes a call or declaration centered on `__nv_float2ll_rd`.
  **L156 CN**: 执行以 `__nv_float2ll_rd` 为核心的调用或声明。
- **L157 EN**: Executes a call or declaration centered on `__nv_float2ll_rn`.
  **L157 CN**: 执行以 `__nv_float2ll_rn` 为核心的调用或声明。
- **L158 EN**: Executes a call or declaration centered on `__nv_float2ll_ru`.
  **L158 CN**: 执行以 `__nv_float2ll_ru` 为核心的调用或声明。
- **L159 EN**: Executes a call or declaration centered on `__nv_float2ll_rz`.
  **L159 CN**: 执行以 `__nv_float2ll_rz` 为核心的调用或声明。
- **L160 EN**: Executes a call or declaration centered on `__nv_float2uint_rd`.
  **L160 CN**: 执行以 `__nv_float2uint_rd` 为核心的调用或声明。

### Lines 161-180

````c
__DEVICE__ unsigned int __nv_float2uint_rn(float __a);
__DEVICE__ unsigned int __nv_float2uint_ru(float __a);
__DEVICE__ unsigned int __nv_float2uint_rz(float __a);
__DEVICE__ unsigned long long __nv_float2ull_rd(float __a);
__DEVICE__ unsigned long long __nv_float2ull_rn(float __a);
__DEVICE__ unsigned long long __nv_float2ull_ru(float __a);
__DEVICE__ unsigned long long __nv_float2ull_rz(float __a);
__DEVICE__ int __nv_float_as_int(float __a);
__DEVICE__ unsigned int __nv_float_as_uint(float __a);
__DEVICE__ double __nv_floor(double __a);
__DEVICE__ float __nv_floorf(float __a);
__DEVICE__ double __nv_fma(double __a, double __b, double __c);
__DEVICE__ float __nv_fmaf(float __a, float __b, float __c);
__DEVICE__ float __nv_fmaf_ieee_rd(float __a, float __b, float __c);
__DEVICE__ float __nv_fmaf_ieee_rn(float __a, float __b, float __c);
__DEVICE__ float __nv_fmaf_ieee_ru(float __a, float __b, float __c);
__DEVICE__ float __nv_fmaf_ieee_rz(float __a, float __b, float __c);
__DEVICE__ float __nv_fmaf_rd(float __a, float __b, float __c);
__DEVICE__ float __nv_fmaf_rn(float __a, float __b, float __c);
__DEVICE__ float __nv_fmaf_ru(float __a, float __b, float __c);
````
- **L161 EN**: Executes a call or declaration centered on `__nv_float2uint_rn`.
  **L161 CN**: 执行以 `__nv_float2uint_rn` 为核心的调用或声明。
- **L162 EN**: Executes a call or declaration centered on `__nv_float2uint_ru`.
  **L162 CN**: 执行以 `__nv_float2uint_ru` 为核心的调用或声明。
- **L163 EN**: Executes a call or declaration centered on `__nv_float2uint_rz`.
  **L163 CN**: 执行以 `__nv_float2uint_rz` 为核心的调用或声明。
- **L164 EN**: Executes a call or declaration centered on `__nv_float2ull_rd`.
  **L164 CN**: 执行以 `__nv_float2ull_rd` 为核心的调用或声明。
- **L165 EN**: Executes a call or declaration centered on `__nv_float2ull_rn`.
  **L165 CN**: 执行以 `__nv_float2ull_rn` 为核心的调用或声明。
- **L166 EN**: Executes a call or declaration centered on `__nv_float2ull_ru`.
  **L166 CN**: 执行以 `__nv_float2ull_ru` 为核心的调用或声明。
- **L167 EN**: Executes a call or declaration centered on `__nv_float2ull_rz`.
  **L167 CN**: 执行以 `__nv_float2ull_rz` 为核心的调用或声明。
- **L168 EN**: Executes a call or declaration centered on `__nv_float_as_int`.
  **L168 CN**: 执行以 `__nv_float_as_int` 为核心的调用或声明。
- **L169 EN**: Executes a call or declaration centered on `__nv_float_as_uint`.
  **L169 CN**: 执行以 `__nv_float_as_uint` 为核心的调用或声明。
- **L170 EN**: Executes a call or declaration centered on `__nv_floor`.
  **L170 CN**: 执行以 `__nv_floor` 为核心的调用或声明。
- **L171 EN**: Executes a call or declaration centered on `__nv_floorf`.
  **L171 CN**: 执行以 `__nv_floorf` 为核心的调用或声明。
- **L172 EN**: Executes a call or declaration centered on `__nv_fma`.
  **L172 CN**: 执行以 `__nv_fma` 为核心的调用或声明。
- **L173 EN**: Executes a call or declaration centered on `__nv_fmaf`.
  **L173 CN**: 执行以 `__nv_fmaf` 为核心的调用或声明。
- **L174 EN**: Executes a call or declaration centered on `__nv_fmaf_ieee_rd`.
  **L174 CN**: 执行以 `__nv_fmaf_ieee_rd` 为核心的调用或声明。
- **L175 EN**: Executes a call or declaration centered on `__nv_fmaf_ieee_rn`.
  **L175 CN**: 执行以 `__nv_fmaf_ieee_rn` 为核心的调用或声明。
- **L176 EN**: Executes a call or declaration centered on `__nv_fmaf_ieee_ru`.
  **L176 CN**: 执行以 `__nv_fmaf_ieee_ru` 为核心的调用或声明。
- **L177 EN**: Executes a call or declaration centered on `__nv_fmaf_ieee_rz`.
  **L177 CN**: 执行以 `__nv_fmaf_ieee_rz` 为核心的调用或声明。
- **L178 EN**: Executes a call or declaration centered on `__nv_fmaf_rd`.
  **L178 CN**: 执行以 `__nv_fmaf_rd` 为核心的调用或声明。
- **L179 EN**: Executes a call or declaration centered on `__nv_fmaf_rn`.
  **L179 CN**: 执行以 `__nv_fmaf_rn` 为核心的调用或声明。
- **L180 EN**: Executes a call or declaration centered on `__nv_fmaf_ru`.
  **L180 CN**: 执行以 `__nv_fmaf_ru` 为核心的调用或声明。

### Lines 181-200

````c
__DEVICE__ float __nv_fmaf_rz(float __a, float __b, float __c);
__DEVICE__ double __nv_fma_rd(double __a, double __b, double __c);
__DEVICE__ double __nv_fma_rn(double __a, double __b, double __c);
__DEVICE__ double __nv_fma_ru(double __a, double __b, double __c);
__DEVICE__ double __nv_fma_rz(double __a, double __b, double __c);
__DEVICE__ double __nv_fmax(double __a, double __b);
__DEVICE__ float __nv_fmaxf(float __a, float __b);
__DEVICE__ double __nv_fmin(double __a, double __b);
__DEVICE__ float __nv_fminf(float __a, float __b);
__DEVICE__ double __nv_fmod(double __a, double __b);
__DEVICE__ float __nv_fmodf(float __a, float __b);
__DEVICE__ float __nv_fmul_rd(float __a, float __b);
__DEVICE__ float __nv_fmul_rn(float __a, float __b);
__DEVICE__ float __nv_fmul_ru(float __a, float __b);
__DEVICE__ float __nv_fmul_rz(float __a, float __b);
__DEVICE__ float __nv_frcp_rd(float __a);
__DEVICE__ float __nv_frcp_rn(float __a);
__DEVICE__ float __nv_frcp_ru(float __a);
__DEVICE__ float __nv_frcp_rz(float __a);
__DEVICE__ double __nv_frexp(double __a, int *__b);
````
- **L181 EN**: Executes a call or declaration centered on `__nv_fmaf_rz`.
  **L181 CN**: 执行以 `__nv_fmaf_rz` 为核心的调用或声明。
- **L182 EN**: Executes a call or declaration centered on `__nv_fma_rd`.
  **L182 CN**: 执行以 `__nv_fma_rd` 为核心的调用或声明。
- **L183 EN**: Executes a call or declaration centered on `__nv_fma_rn`.
  **L183 CN**: 执行以 `__nv_fma_rn` 为核心的调用或声明。
- **L184 EN**: Executes a call or declaration centered on `__nv_fma_ru`.
  **L184 CN**: 执行以 `__nv_fma_ru` 为核心的调用或声明。
- **L185 EN**: Executes a call or declaration centered on `__nv_fma_rz`.
  **L185 CN**: 执行以 `__nv_fma_rz` 为核心的调用或声明。
- **L186 EN**: Executes a call or declaration centered on `__nv_fmax`.
  **L186 CN**: 执行以 `__nv_fmax` 为核心的调用或声明。
- **L187 EN**: Executes a call or declaration centered on `__nv_fmaxf`.
  **L187 CN**: 执行以 `__nv_fmaxf` 为核心的调用或声明。
- **L188 EN**: Executes a call or declaration centered on `__nv_fmin`.
  **L188 CN**: 执行以 `__nv_fmin` 为核心的调用或声明。
- **L189 EN**: Executes a call or declaration centered on `__nv_fminf`.
  **L189 CN**: 执行以 `__nv_fminf` 为核心的调用或声明。
- **L190 EN**: Executes a call or declaration centered on `__nv_fmod`.
  **L190 CN**: 执行以 `__nv_fmod` 为核心的调用或声明。
- **L191 EN**: Executes a call or declaration centered on `__nv_fmodf`.
  **L191 CN**: 执行以 `__nv_fmodf` 为核心的调用或声明。
- **L192 EN**: Executes a call or declaration centered on `__nv_fmul_rd`.
  **L192 CN**: 执行以 `__nv_fmul_rd` 为核心的调用或声明。
- **L193 EN**: Executes a call or declaration centered on `__nv_fmul_rn`.
  **L193 CN**: 执行以 `__nv_fmul_rn` 为核心的调用或声明。
- **L194 EN**: Executes a call or declaration centered on `__nv_fmul_ru`.
  **L194 CN**: 执行以 `__nv_fmul_ru` 为核心的调用或声明。
- **L195 EN**: Executes a call or declaration centered on `__nv_fmul_rz`.
  **L195 CN**: 执行以 `__nv_fmul_rz` 为核心的调用或声明。
- **L196 EN**: Executes a call or declaration centered on `__nv_frcp_rd`.
  **L196 CN**: 执行以 `__nv_frcp_rd` 为核心的调用或声明。
- **L197 EN**: Executes a call or declaration centered on `__nv_frcp_rn`.
  **L197 CN**: 执行以 `__nv_frcp_rn` 为核心的调用或声明。
- **L198 EN**: Executes a call or declaration centered on `__nv_frcp_ru`.
  **L198 CN**: 执行以 `__nv_frcp_ru` 为核心的调用或声明。
- **L199 EN**: Executes a call or declaration centered on `__nv_frcp_rz`.
  **L199 CN**: 执行以 `__nv_frcp_rz` 为核心的调用或声明。
- **L200 EN**: Executes a call or declaration centered on `__nv_frexp`.
  **L200 CN**: 执行以 `__nv_frexp` 为核心的调用或声明。

### Lines 201-220

````c
__DEVICE__ float __nv_frexpf(float __a, int *__b);
__DEVICE__ float __nv_frsqrt_rn(float __a);
__DEVICE__ float __nv_fsqrt_rd(float __a);
__DEVICE__ float __nv_fsqrt_rn(float __a);
__DEVICE__ float __nv_fsqrt_ru(float __a);
__DEVICE__ float __nv_fsqrt_rz(float __a);
__DEVICE__ float __nv_fsub_rd(float __a, float __b);
__DEVICE__ float __nv_fsub_rn(float __a, float __b);
__DEVICE__ float __nv_fsub_ru(float __a, float __b);
__DEVICE__ float __nv_fsub_rz(float __a, float __b);
__DEVICE__ int __nv_hadd(int __a, int __b);
__DEVICE__ float __nv_half2float(unsigned short __h);
__DEVICE__ double __nv_hiloint2double(int __a, int __b);
__DEVICE__ double __nv_hypot(double __a, double __b);
__DEVICE__ float __nv_hypotf(float __a, float __b);
__DEVICE__ int __nv_ilogb(double __a);
__DEVICE__ int __nv_ilogbf(float __a);
__DEVICE__ double __nv_int2double_rn(int __a);
__DEVICE__ float __nv_int2float_rd(int __a);
__DEVICE__ float __nv_int2float_rn(int __a);
````
- **L201 EN**: Executes a call or declaration centered on `__nv_frexpf`.
  **L201 CN**: 执行以 `__nv_frexpf` 为核心的调用或声明。
- **L202 EN**: Executes a call or declaration centered on `__nv_frsqrt_rn`.
  **L202 CN**: 执行以 `__nv_frsqrt_rn` 为核心的调用或声明。
- **L203 EN**: Executes a call or declaration centered on `__nv_fsqrt_rd`.
  **L203 CN**: 执行以 `__nv_fsqrt_rd` 为核心的调用或声明。
- **L204 EN**: Executes a call or declaration centered on `__nv_fsqrt_rn`.
  **L204 CN**: 执行以 `__nv_fsqrt_rn` 为核心的调用或声明。
- **L205 EN**: Executes a call or declaration centered on `__nv_fsqrt_ru`.
  **L205 CN**: 执行以 `__nv_fsqrt_ru` 为核心的调用或声明。
- **L206 EN**: Executes a call or declaration centered on `__nv_fsqrt_rz`.
  **L206 CN**: 执行以 `__nv_fsqrt_rz` 为核心的调用或声明。
- **L207 EN**: Executes a call or declaration centered on `__nv_fsub_rd`.
  **L207 CN**: 执行以 `__nv_fsub_rd` 为核心的调用或声明。
- **L208 EN**: Executes a call or declaration centered on `__nv_fsub_rn`.
  **L208 CN**: 执行以 `__nv_fsub_rn` 为核心的调用或声明。
- **L209 EN**: Executes a call or declaration centered on `__nv_fsub_ru`.
  **L209 CN**: 执行以 `__nv_fsub_ru` 为核心的调用或声明。
- **L210 EN**: Executes a call or declaration centered on `__nv_fsub_rz`.
  **L210 CN**: 执行以 `__nv_fsub_rz` 为核心的调用或声明。
- **L211 EN**: Executes a call or declaration centered on `__nv_hadd`.
  **L211 CN**: 执行以 `__nv_hadd` 为核心的调用或声明。
- **L212 EN**: Executes a call or declaration centered on `__nv_half2float`.
  **L212 CN**: 执行以 `__nv_half2float` 为核心的调用或声明。
- **L213 EN**: Executes a call or declaration centered on `__nv_hiloint2double`.
  **L213 CN**: 执行以 `__nv_hiloint2double` 为核心的调用或声明。
- **L214 EN**: Executes a call or declaration centered on `__nv_hypot`.
  **L214 CN**: 执行以 `__nv_hypot` 为核心的调用或声明。
- **L215 EN**: Executes a call or declaration centered on `__nv_hypotf`.
  **L215 CN**: 执行以 `__nv_hypotf` 为核心的调用或声明。
- **L216 EN**: Executes a call or declaration centered on `__nv_ilogb`.
  **L216 CN**: 执行以 `__nv_ilogb` 为核心的调用或声明。
- **L217 EN**: Executes a call or declaration centered on `__nv_ilogbf`.
  **L217 CN**: 执行以 `__nv_ilogbf` 为核心的调用或声明。
- **L218 EN**: Executes a call or declaration centered on `__nv_int2double_rn`.
  **L218 CN**: 执行以 `__nv_int2double_rn` 为核心的调用或声明。
- **L219 EN**: Executes a call or declaration centered on `__nv_int2float_rd`.
  **L219 CN**: 执行以 `__nv_int2float_rd` 为核心的调用或声明。
- **L220 EN**: Executes a call or declaration centered on `__nv_int2float_rn`.
  **L220 CN**: 执行以 `__nv_int2float_rn` 为核心的调用或声明。

### Lines 221-240

````c
__DEVICE__ float __nv_int2float_ru(int __a);
__DEVICE__ float __nv_int2float_rz(int __a);
__DEVICE__ float __nv_int_as_float(int __a);
__DEVICE__ int __nv_isfinited(double __a);
__DEVICE__ int __nv_isinfd(double __a);
__DEVICE__ int __nv_isinff(float __a);
__DEVICE__ int __nv_isnand(double __a);
__DEVICE__ int __nv_isnanf(float __a);
__DEVICE__ double __nv_j0(double __a);
__DEVICE__ float __nv_j0f(float __a);
__DEVICE__ double __nv_j1(double __a);
__DEVICE__ float __nv_j1f(float __a);
__DEVICE__ float __nv_jnf(int __a, float __b);
__DEVICE__ double __nv_jn(int __a, double __b);
__DEVICE__ double __nv_ldexp(double __a, int __b);
__DEVICE__ float __nv_ldexpf(float __a, int __b);
__DEVICE__ double __nv_lgamma(double __a);
__DEVICE__ float __nv_lgammaf(float __a);
__DEVICE__ double __nv_ll2double_rd(long long __a);
__DEVICE__ double __nv_ll2double_rn(long long __a);
````
- **L221 EN**: Executes a call or declaration centered on `__nv_int2float_ru`.
  **L221 CN**: 执行以 `__nv_int2float_ru` 为核心的调用或声明。
- **L222 EN**: Executes a call or declaration centered on `__nv_int2float_rz`.
  **L222 CN**: 执行以 `__nv_int2float_rz` 为核心的调用或声明。
- **L223 EN**: Executes a call or declaration centered on `__nv_int_as_float`.
  **L223 CN**: 执行以 `__nv_int_as_float` 为核心的调用或声明。
- **L224 EN**: Executes a call or declaration centered on `__nv_isfinited`.
  **L224 CN**: 执行以 `__nv_isfinited` 为核心的调用或声明。
- **L225 EN**: Executes a call or declaration centered on `__nv_isinfd`.
  **L225 CN**: 执行以 `__nv_isinfd` 为核心的调用或声明。
- **L226 EN**: Executes a call or declaration centered on `__nv_isinff`.
  **L226 CN**: 执行以 `__nv_isinff` 为核心的调用或声明。
- **L227 EN**: Executes a call or declaration centered on `__nv_isnand`.
  **L227 CN**: 执行以 `__nv_isnand` 为核心的调用或声明。
- **L228 EN**: Executes a call or declaration centered on `__nv_isnanf`.
  **L228 CN**: 执行以 `__nv_isnanf` 为核心的调用或声明。
- **L229 EN**: Executes a call or declaration centered on `__nv_j0`.
  **L229 CN**: 执行以 `__nv_j0` 为核心的调用或声明。
- **L230 EN**: Executes a call or declaration centered on `__nv_j0f`.
  **L230 CN**: 执行以 `__nv_j0f` 为核心的调用或声明。
- **L231 EN**: Executes a call or declaration centered on `__nv_j1`.
  **L231 CN**: 执行以 `__nv_j1` 为核心的调用或声明。
- **L232 EN**: Executes a call or declaration centered on `__nv_j1f`.
  **L232 CN**: 执行以 `__nv_j1f` 为核心的调用或声明。
- **L233 EN**: Executes a call or declaration centered on `__nv_jnf`.
  **L233 CN**: 执行以 `__nv_jnf` 为核心的调用或声明。
- **L234 EN**: Executes a call or declaration centered on `__nv_jn`.
  **L234 CN**: 执行以 `__nv_jn` 为核心的调用或声明。
- **L235 EN**: Executes a call or declaration centered on `__nv_ldexp`.
  **L235 CN**: 执行以 `__nv_ldexp` 为核心的调用或声明。
- **L236 EN**: Executes a call or declaration centered on `__nv_ldexpf`.
  **L236 CN**: 执行以 `__nv_ldexpf` 为核心的调用或声明。
- **L237 EN**: Executes a call or declaration centered on `__nv_lgamma`.
  **L237 CN**: 执行以 `__nv_lgamma` 为核心的调用或声明。
- **L238 EN**: Executes a call or declaration centered on `__nv_lgammaf`.
  **L238 CN**: 执行以 `__nv_lgammaf` 为核心的调用或声明。
- **L239 EN**: Executes a call or declaration centered on `__nv_ll2double_rd`.
  **L239 CN**: 执行以 `__nv_ll2double_rd` 为核心的调用或声明。
- **L240 EN**: Executes a call or declaration centered on `__nv_ll2double_rn`.
  **L240 CN**: 执行以 `__nv_ll2double_rn` 为核心的调用或声明。

### Lines 241-260

````c
__DEVICE__ double __nv_ll2double_ru(long long __a);
__DEVICE__ double __nv_ll2double_rz(long long __a);
__DEVICE__ float __nv_ll2float_rd(long long __a);
__DEVICE__ float __nv_ll2float_rn(long long __a);
__DEVICE__ float __nv_ll2float_ru(long long __a);
__DEVICE__ float __nv_ll2float_rz(long long __a);
__DEVICE__ long long __nv_llabs(long long __a);
__DEVICE__ long long __nv_llmax(long long __a, long long __b);
__DEVICE__ long long __nv_llmin(long long __a, long long __b);
__DEVICE__ long long __nv_llrint(double __a);
__DEVICE__ long long __nv_llrintf(float __a);
__DEVICE__ long long __nv_llround(double __a);
__DEVICE__ long long __nv_llroundf(float __a);
__DEVICE__ double __nv_log10(double __a);
__DEVICE__ float __nv_log10f(float __a);
__DEVICE__ double __nv_log1p(double __a);
__DEVICE__ float __nv_log1pf(float __a);
__DEVICE__ double __nv_log2(double __a);
__DEVICE__ float __nv_log2f(float __a);
__DEVICE__ double __nv_logb(double __a);
````
- **L241 EN**: Executes a call or declaration centered on `__nv_ll2double_ru`.
  **L241 CN**: 执行以 `__nv_ll2double_ru` 为核心的调用或声明。
- **L242 EN**: Executes a call or declaration centered on `__nv_ll2double_rz`.
  **L242 CN**: 执行以 `__nv_ll2double_rz` 为核心的调用或声明。
- **L243 EN**: Executes a call or declaration centered on `__nv_ll2float_rd`.
  **L243 CN**: 执行以 `__nv_ll2float_rd` 为核心的调用或声明。
- **L244 EN**: Executes a call or declaration centered on `__nv_ll2float_rn`.
  **L244 CN**: 执行以 `__nv_ll2float_rn` 为核心的调用或声明。
- **L245 EN**: Executes a call or declaration centered on `__nv_ll2float_ru`.
  **L245 CN**: 执行以 `__nv_ll2float_ru` 为核心的调用或声明。
- **L246 EN**: Executes a call or declaration centered on `__nv_ll2float_rz`.
  **L246 CN**: 执行以 `__nv_ll2float_rz` 为核心的调用或声明。
- **L247 EN**: Executes a call or declaration centered on `__nv_llabs`.
  **L247 CN**: 执行以 `__nv_llabs` 为核心的调用或声明。
- **L248 EN**: Executes a call or declaration centered on `__nv_llmax`.
  **L248 CN**: 执行以 `__nv_llmax` 为核心的调用或声明。
- **L249 EN**: Executes a call or declaration centered on `__nv_llmin`.
  **L249 CN**: 执行以 `__nv_llmin` 为核心的调用或声明。
- **L250 EN**: Executes a call or declaration centered on `__nv_llrint`.
  **L250 CN**: 执行以 `__nv_llrint` 为核心的调用或声明。
- **L251 EN**: Executes a call or declaration centered on `__nv_llrintf`.
  **L251 CN**: 执行以 `__nv_llrintf` 为核心的调用或声明。
- **L252 EN**: Executes a call or declaration centered on `__nv_llround`.
  **L252 CN**: 执行以 `__nv_llround` 为核心的调用或声明。
- **L253 EN**: Executes a call or declaration centered on `__nv_llroundf`.
  **L253 CN**: 执行以 `__nv_llroundf` 为核心的调用或声明。
- **L254 EN**: Executes a call or declaration centered on `__nv_log10`.
  **L254 CN**: 执行以 `__nv_log10` 为核心的调用或声明。
- **L255 EN**: Executes a call or declaration centered on `__nv_log10f`.
  **L255 CN**: 执行以 `__nv_log10f` 为核心的调用或声明。
- **L256 EN**: Executes a call or declaration centered on `__nv_log1p`.
  **L256 CN**: 执行以 `__nv_log1p` 为核心的调用或声明。
- **L257 EN**: Executes a call or declaration centered on `__nv_log1pf`.
  **L257 CN**: 执行以 `__nv_log1pf` 为核心的调用或声明。
- **L258 EN**: Executes a call or declaration centered on `__nv_log2`.
  **L258 CN**: 执行以 `__nv_log2` 为核心的调用或声明。
- **L259 EN**: Executes a call or declaration centered on `__nv_log2f`.
  **L259 CN**: 执行以 `__nv_log2f` 为核心的调用或声明。
- **L260 EN**: Executes a call or declaration centered on `__nv_logb`.
  **L260 CN**: 执行以 `__nv_logb` 为核心的调用或声明。

### Lines 261-280

````c
__DEVICE__ float __nv_logbf(float __a);
__DEVICE__ double __nv_log(double __a);
__DEVICE__ float __nv_logf(float __a);
__DEVICE__ double __nv_longlong_as_double(long long __a);
__DEVICE__ int __nv_max(int __a, int __b);
__DEVICE__ int __nv_min(int __a, int __b);
__DEVICE__ double __nv_modf(double __a, double *__b);
__DEVICE__ float __nv_modff(float __a, float *__b);
__DEVICE__ int __nv_mul24(int __a, int __b);
__DEVICE__ long long __nv_mul64hi(long long __a, long long __b);
__DEVICE__ int __nv_mulhi(int __a, int __b);
__DEVICE__ double __nv_nan(const signed char *__a);
__DEVICE__ float __nv_nanf(const signed char *__a);
__DEVICE__ double __nv_nearbyint(double __a);
__DEVICE__ float __nv_nearbyintf(float __a);
__DEVICE__ double __nv_nextafter(double __a, double __b);
__DEVICE__ float __nv_nextafterf(float __a, float __b);
__DEVICE__ double __nv_norm3d(double __a, double __b, double __c);
__DEVICE__ float __nv_norm3df(float __a, float __b, float __c);
__DEVICE__ double __nv_norm4d(double __a, double __b, double __c, double __d);
````
- **L261 EN**: Executes a call or declaration centered on `__nv_logbf`.
  **L261 CN**: 执行以 `__nv_logbf` 为核心的调用或声明。
- **L262 EN**: Executes a call or declaration centered on `__nv_log`.
  **L262 CN**: 执行以 `__nv_log` 为核心的调用或声明。
- **L263 EN**: Executes a call or declaration centered on `__nv_logf`.
  **L263 CN**: 执行以 `__nv_logf` 为核心的调用或声明。
- **L264 EN**: Executes a call or declaration centered on `__nv_longlong_as_double`.
  **L264 CN**: 执行以 `__nv_longlong_as_double` 为核心的调用或声明。
- **L265 EN**: Executes a call or declaration centered on `__nv_max`.
  **L265 CN**: 执行以 `__nv_max` 为核心的调用或声明。
- **L266 EN**: Executes a call or declaration centered on `__nv_min`.
  **L266 CN**: 执行以 `__nv_min` 为核心的调用或声明。
- **L267 EN**: Executes a call or declaration centered on `__nv_modf`.
  **L267 CN**: 执行以 `__nv_modf` 为核心的调用或声明。
- **L268 EN**: Executes a call or declaration centered on `__nv_modff`.
  **L268 CN**: 执行以 `__nv_modff` 为核心的调用或声明。
- **L269 EN**: Executes a call or declaration centered on `__nv_mul24`.
  **L269 CN**: 执行以 `__nv_mul24` 为核心的调用或声明。
- **L270 EN**: Executes a call or declaration centered on `__nv_mul64hi`.
  **L270 CN**: 执行以 `__nv_mul64hi` 为核心的调用或声明。
- **L271 EN**: Executes a call or declaration centered on `__nv_mulhi`.
  **L271 CN**: 执行以 `__nv_mulhi` 为核心的调用或声明。
- **L272 EN**: Executes a call or declaration centered on `__nv_nan`.
  **L272 CN**: 执行以 `__nv_nan` 为核心的调用或声明。
- **L273 EN**: Executes a call or declaration centered on `__nv_nanf`.
  **L273 CN**: 执行以 `__nv_nanf` 为核心的调用或声明。
- **L274 EN**: Executes a call or declaration centered on `__nv_nearbyint`.
  **L274 CN**: 执行以 `__nv_nearbyint` 为核心的调用或声明。
- **L275 EN**: Executes a call or declaration centered on `__nv_nearbyintf`.
  **L275 CN**: 执行以 `__nv_nearbyintf` 为核心的调用或声明。
- **L276 EN**: Executes a call or declaration centered on `__nv_nextafter`.
  **L276 CN**: 执行以 `__nv_nextafter` 为核心的调用或声明。
- **L277 EN**: Executes a call or declaration centered on `__nv_nextafterf`.
  **L277 CN**: 执行以 `__nv_nextafterf` 为核心的调用或声明。
- **L278 EN**: Executes a call or declaration centered on `__nv_norm3d`.
  **L278 CN**: 执行以 `__nv_norm3d` 为核心的调用或声明。
- **L279 EN**: Executes a call or declaration centered on `__nv_norm3df`.
  **L279 CN**: 执行以 `__nv_norm3df` 为核心的调用或声明。
- **L280 EN**: Executes a call or declaration centered on `__nv_norm4d`.
  **L280 CN**: 执行以 `__nv_norm4d` 为核心的调用或声明。

### Lines 281-300

````c
__DEVICE__ float __nv_norm4df(float __a, float __b, float __c, float __d);
__DEVICE__ double __nv_normcdf(double __a);
__DEVICE__ float __nv_normcdff(float __a);
__DEVICE__ double __nv_normcdfinv(double __a);
__DEVICE__ float __nv_normcdfinvf(float __a);
__DEVICE__ float __nv_normf(int __a, const float *__b);
__DEVICE__ double __nv_norm(int __a, const double *__b);
__DEVICE__ int __nv_popc(unsigned int __a);
__DEVICE__ int __nv_popcll(unsigned long long __a);
__DEVICE__ double __nv_pow(double __a, double __b);
__DEVICE__ float __nv_powf(float __a, float __b);
__DEVICE__ double __nv_powi(double __a, int __b);
__DEVICE__ float __nv_powif(float __a, int __b);
__DEVICE__ double __nv_rcbrt(double __a);
__DEVICE__ float __nv_rcbrtf(float __a);
__DEVICE__ double __nv_rcp64h(double __a);
__DEVICE__ double __nv_remainder(double __a, double __b);
__DEVICE__ float __nv_remainderf(float __a, float __b);
__DEVICE__ double __nv_remquo(double __a, double __b, int *__c);
__DEVICE__ float __nv_remquof(float __a, float __b, int *__c);
````
- **L281 EN**: Executes a call or declaration centered on `__nv_norm4df`.
  **L281 CN**: 执行以 `__nv_norm4df` 为核心的调用或声明。
- **L282 EN**: Executes a call or declaration centered on `__nv_normcdf`.
  **L282 CN**: 执行以 `__nv_normcdf` 为核心的调用或声明。
- **L283 EN**: Executes a call or declaration centered on `__nv_normcdff`.
  **L283 CN**: 执行以 `__nv_normcdff` 为核心的调用或声明。
- **L284 EN**: Executes a call or declaration centered on `__nv_normcdfinv`.
  **L284 CN**: 执行以 `__nv_normcdfinv` 为核心的调用或声明。
- **L285 EN**: Executes a call or declaration centered on `__nv_normcdfinvf`.
  **L285 CN**: 执行以 `__nv_normcdfinvf` 为核心的调用或声明。
- **L286 EN**: Executes a call or declaration centered on `__nv_normf`.
  **L286 CN**: 执行以 `__nv_normf` 为核心的调用或声明。
- **L287 EN**: Executes a call or declaration centered on `__nv_norm`.
  **L287 CN**: 执行以 `__nv_norm` 为核心的调用或声明。
- **L288 EN**: Executes a call or declaration centered on `__nv_popc`.
  **L288 CN**: 执行以 `__nv_popc` 为核心的调用或声明。
- **L289 EN**: Executes a call or declaration centered on `__nv_popcll`.
  **L289 CN**: 执行以 `__nv_popcll` 为核心的调用或声明。
- **L290 EN**: Executes a call or declaration centered on `__nv_pow`.
  **L290 CN**: 执行以 `__nv_pow` 为核心的调用或声明。
- **L291 EN**: Executes a call or declaration centered on `__nv_powf`.
  **L291 CN**: 执行以 `__nv_powf` 为核心的调用或声明。
- **L292 EN**: Executes a call or declaration centered on `__nv_powi`.
  **L292 CN**: 执行以 `__nv_powi` 为核心的调用或声明。
- **L293 EN**: Executes a call or declaration centered on `__nv_powif`.
  **L293 CN**: 执行以 `__nv_powif` 为核心的调用或声明。
- **L294 EN**: Executes a call or declaration centered on `__nv_rcbrt`.
  **L294 CN**: 执行以 `__nv_rcbrt` 为核心的调用或声明。
- **L295 EN**: Executes a call or declaration centered on `__nv_rcbrtf`.
  **L295 CN**: 执行以 `__nv_rcbrtf` 为核心的调用或声明。
- **L296 EN**: Executes a call or declaration centered on `__nv_rcp64h`.
  **L296 CN**: 执行以 `__nv_rcp64h` 为核心的调用或声明。
- **L297 EN**: Executes a call or declaration centered on `__nv_remainder`.
  **L297 CN**: 执行以 `__nv_remainder` 为核心的调用或声明。
- **L298 EN**: Executes a call or declaration centered on `__nv_remainderf`.
  **L298 CN**: 执行以 `__nv_remainderf` 为核心的调用或声明。
- **L299 EN**: Executes a call or declaration centered on `__nv_remquo`.
  **L299 CN**: 执行以 `__nv_remquo` 为核心的调用或声明。
- **L300 EN**: Executes a call or declaration centered on `__nv_remquof`.
  **L300 CN**: 执行以 `__nv_remquof` 为核心的调用或声明。

### Lines 301-320

````c
__DEVICE__ int __nv_rhadd(int __a, int __b);
__DEVICE__ double __nv_rhypot(double __a, double __b);
__DEVICE__ float __nv_rhypotf(float __a, float __b);
__DEVICE__ double __nv_rint(double __a);
__DEVICE__ float __nv_rintf(float __a);
__DEVICE__ double __nv_rnorm3d(double __a, double __b, double __c);
__DEVICE__ float __nv_rnorm3df(float __a, float __b, float __c);
__DEVICE__ double __nv_rnorm4d(double __a, double __b, double __c, double __d);
__DEVICE__ float __nv_rnorm4df(float __a, float __b, float __c, float __d);
__DEVICE__ float __nv_rnormf(int __a, const float *__b);
__DEVICE__ double __nv_rnorm(int __a, const double *__b);
__DEVICE__ double __nv_round(double __a);
__DEVICE__ float __nv_roundf(float __a);
__DEVICE__ double __nv_rsqrt(double __a);
__DEVICE__ float __nv_rsqrtf(float __a);
__DEVICE__ int __nv_sad(int __a, int __b, int __c);
__DEVICE__ float __nv_saturatef(float __a);
__DEVICE__ double __nv_scalbn(double __a, int __b);
__DEVICE__ float __nv_scalbnf(float __a, int __b);
__DEVICE__ int __nv_signbitd(double __a);
````
- **L301 EN**: Executes a call or declaration centered on `__nv_rhadd`.
  **L301 CN**: 执行以 `__nv_rhadd` 为核心的调用或声明。
- **L302 EN**: Executes a call or declaration centered on `__nv_rhypot`.
  **L302 CN**: 执行以 `__nv_rhypot` 为核心的调用或声明。
- **L303 EN**: Executes a call or declaration centered on `__nv_rhypotf`.
  **L303 CN**: 执行以 `__nv_rhypotf` 为核心的调用或声明。
- **L304 EN**: Executes a call or declaration centered on `__nv_rint`.
  **L304 CN**: 执行以 `__nv_rint` 为核心的调用或声明。
- **L305 EN**: Executes a call or declaration centered on `__nv_rintf`.
  **L305 CN**: 执行以 `__nv_rintf` 为核心的调用或声明。
- **L306 EN**: Executes a call or declaration centered on `__nv_rnorm3d`.
  **L306 CN**: 执行以 `__nv_rnorm3d` 为核心的调用或声明。
- **L307 EN**: Executes a call or declaration centered on `__nv_rnorm3df`.
  **L307 CN**: 执行以 `__nv_rnorm3df` 为核心的调用或声明。
- **L308 EN**: Executes a call or declaration centered on `__nv_rnorm4d`.
  **L308 CN**: 执行以 `__nv_rnorm4d` 为核心的调用或声明。
- **L309 EN**: Executes a call or declaration centered on `__nv_rnorm4df`.
  **L309 CN**: 执行以 `__nv_rnorm4df` 为核心的调用或声明。
- **L310 EN**: Executes a call or declaration centered on `__nv_rnormf`.
  **L310 CN**: 执行以 `__nv_rnormf` 为核心的调用或声明。
- **L311 EN**: Executes a call or declaration centered on `__nv_rnorm`.
  **L311 CN**: 执行以 `__nv_rnorm` 为核心的调用或声明。
- **L312 EN**: Executes a call or declaration centered on `__nv_round`.
  **L312 CN**: 执行以 `__nv_round` 为核心的调用或声明。
- **L313 EN**: Executes a call or declaration centered on `__nv_roundf`.
  **L313 CN**: 执行以 `__nv_roundf` 为核心的调用或声明。
- **L314 EN**: Executes a call or declaration centered on `__nv_rsqrt`.
  **L314 CN**: 执行以 `__nv_rsqrt` 为核心的调用或声明。
- **L315 EN**: Executes a call or declaration centered on `__nv_rsqrtf`.
  **L315 CN**: 执行以 `__nv_rsqrtf` 为核心的调用或声明。
- **L316 EN**: Executes a call or declaration centered on `__nv_sad`.
  **L316 CN**: 执行以 `__nv_sad` 为核心的调用或声明。
- **L317 EN**: Executes a call or declaration centered on `__nv_saturatef`.
  **L317 CN**: 执行以 `__nv_saturatef` 为核心的调用或声明。
- **L318 EN**: Executes a call or declaration centered on `__nv_scalbn`.
  **L318 CN**: 执行以 `__nv_scalbn` 为核心的调用或声明。
- **L319 EN**: Executes a call or declaration centered on `__nv_scalbnf`.
  **L319 CN**: 执行以 `__nv_scalbnf` 为核心的调用或声明。
- **L320 EN**: Executes a call or declaration centered on `__nv_signbitd`.
  **L320 CN**: 执行以 `__nv_signbitd` 为核心的调用或声明。

### Lines 321-340

````c
__DEVICE__ int __nv_signbitf(float __a);
__DEVICE__ void __nv_sincos(double __a, double *__b, double *__c);
__DEVICE__ void __nv_sincosf(float __a, float *__b, float *__c);
__DEVICE__ void __nv_sincospi(double __a, double *__b, double *__c);
__DEVICE__ void __nv_sincospif(float __a, float *__b, float *__c);
__DEVICE__ double __nv_sin(double __a);
__DEVICE__ float __nv_sinf(float __a);
__DEVICE__ double __nv_sinh(double __a);
__DEVICE__ float __nv_sinhf(float __a);
__DEVICE__ double __nv_sinpi(double __a);
__DEVICE__ float __nv_sinpif(float __a);
__DEVICE__ double __nv_sqrt(double __a);
__DEVICE__ float __nv_sqrtf(float __a);
__DEVICE__ double __nv_tan(double __a);
__DEVICE__ float __nv_tanf(float __a);
__DEVICE__ double __nv_tanh(double __a);
__DEVICE__ float __nv_tanhf(float __a);
__DEVICE__ double __nv_tgamma(double __a);
__DEVICE__ float __nv_tgammaf(float __a);
__DEVICE__ double __nv_trunc(double __a);
````
- **L321 EN**: Executes a call or declaration centered on `__nv_signbitf`.
  **L321 CN**: 执行以 `__nv_signbitf` 为核心的调用或声明。
- **L322 EN**: Executes a call or declaration centered on `__nv_sincos`.
  **L322 CN**: 执行以 `__nv_sincos` 为核心的调用或声明。
- **L323 EN**: Executes a call or declaration centered on `__nv_sincosf`.
  **L323 CN**: 执行以 `__nv_sincosf` 为核心的调用或声明。
- **L324 EN**: Executes a call or declaration centered on `__nv_sincospi`.
  **L324 CN**: 执行以 `__nv_sincospi` 为核心的调用或声明。
- **L325 EN**: Executes a call or declaration centered on `__nv_sincospif`.
  **L325 CN**: 执行以 `__nv_sincospif` 为核心的调用或声明。
- **L326 EN**: Executes a call or declaration centered on `__nv_sin`.
  **L326 CN**: 执行以 `__nv_sin` 为核心的调用或声明。
- **L327 EN**: Executes a call or declaration centered on `__nv_sinf`.
  **L327 CN**: 执行以 `__nv_sinf` 为核心的调用或声明。
- **L328 EN**: Executes a call or declaration centered on `__nv_sinh`.
  **L328 CN**: 执行以 `__nv_sinh` 为核心的调用或声明。
- **L329 EN**: Executes a call or declaration centered on `__nv_sinhf`.
  **L329 CN**: 执行以 `__nv_sinhf` 为核心的调用或声明。
- **L330 EN**: Executes a call or declaration centered on `__nv_sinpi`.
  **L330 CN**: 执行以 `__nv_sinpi` 为核心的调用或声明。
- **L331 EN**: Executes a call or declaration centered on `__nv_sinpif`.
  **L331 CN**: 执行以 `__nv_sinpif` 为核心的调用或声明。
- **L332 EN**: Executes a call or declaration centered on `__nv_sqrt`.
  **L332 CN**: 执行以 `__nv_sqrt` 为核心的调用或声明。
- **L333 EN**: Executes a call or declaration centered on `__nv_sqrtf`.
  **L333 CN**: 执行以 `__nv_sqrtf` 为核心的调用或声明。
- **L334 EN**: Executes a call or declaration centered on `__nv_tan`.
  **L334 CN**: 执行以 `__nv_tan` 为核心的调用或声明。
- **L335 EN**: Executes a call or declaration centered on `__nv_tanf`.
  **L335 CN**: 执行以 `__nv_tanf` 为核心的调用或声明。
- **L336 EN**: Executes a call or declaration centered on `__nv_tanh`.
  **L336 CN**: 执行以 `__nv_tanh` 为核心的调用或声明。
- **L337 EN**: Executes a call or declaration centered on `__nv_tanhf`.
  **L337 CN**: 执行以 `__nv_tanhf` 为核心的调用或声明。
- **L338 EN**: Executes a call or declaration centered on `__nv_tgamma`.
  **L338 CN**: 执行以 `__nv_tgamma` 为核心的调用或声明。
- **L339 EN**: Executes a call or declaration centered on `__nv_tgammaf`.
  **L339 CN**: 执行以 `__nv_tgammaf` 为核心的调用或声明。
- **L340 EN**: Executes a call or declaration centered on `__nv_trunc`.
  **L340 CN**: 执行以 `__nv_trunc` 为核心的调用或声明。

### Lines 341-360

````c
__DEVICE__ float __nv_truncf(float __a);
__DEVICE__ int __nv_uhadd(unsigned int __a, unsigned int __b);
__DEVICE__ double __nv_uint2double_rn(unsigned int __i);
__DEVICE__ float __nv_uint2float_rd(unsigned int __a);
__DEVICE__ float __nv_uint2float_rn(unsigned int __a);
__DEVICE__ float __nv_uint2float_ru(unsigned int __a);
__DEVICE__ float __nv_uint2float_rz(unsigned int __a);
__DEVICE__ float __nv_uint_as_float(unsigned int __a);
__DEVICE__ double __nv_ull2double_rd(unsigned long long __a);
__DEVICE__ double __nv_ull2double_rn(unsigned long long __a);
__DEVICE__ double __nv_ull2double_ru(unsigned long long __a);
__DEVICE__ double __nv_ull2double_rz(unsigned long long __a);
__DEVICE__ float __nv_ull2float_rd(unsigned long long __a);
__DEVICE__ float __nv_ull2float_rn(unsigned long long __a);
__DEVICE__ float __nv_ull2float_ru(unsigned long long __a);
__DEVICE__ float __nv_ull2float_rz(unsigned long long __a);
__DEVICE__ unsigned long long __nv_ullmax(unsigned long long __a,
                                          unsigned long long __b);
__DEVICE__ unsigned long long __nv_ullmin(unsigned long long __a,
                                          unsigned long long __b);
````
- **L341 EN**: Executes a call or declaration centered on `__nv_truncf`.
  **L341 CN**: 执行以 `__nv_truncf` 为核心的调用或声明。
- **L342 EN**: Executes a call or declaration centered on `__nv_uhadd`.
  **L342 CN**: 执行以 `__nv_uhadd` 为核心的调用或声明。
- **L343 EN**: Executes a call or declaration centered on `__nv_uint2double_rn`.
  **L343 CN**: 执行以 `__nv_uint2double_rn` 为核心的调用或声明。
- **L344 EN**: Executes a call or declaration centered on `__nv_uint2float_rd`.
  **L344 CN**: 执行以 `__nv_uint2float_rd` 为核心的调用或声明。
- **L345 EN**: Executes a call or declaration centered on `__nv_uint2float_rn`.
  **L345 CN**: 执行以 `__nv_uint2float_rn` 为核心的调用或声明。
- **L346 EN**: Executes a call or declaration centered on `__nv_uint2float_ru`.
  **L346 CN**: 执行以 `__nv_uint2float_ru` 为核心的调用或声明。
- **L347 EN**: Executes a call or declaration centered on `__nv_uint2float_rz`.
  **L347 CN**: 执行以 `__nv_uint2float_rz` 为核心的调用或声明。
- **L348 EN**: Executes a call or declaration centered on `__nv_uint_as_float`.
  **L348 CN**: 执行以 `__nv_uint_as_float` 为核心的调用或声明。
- **L349 EN**: Executes a call or declaration centered on `__nv_ull2double_rd`.
  **L349 CN**: 执行以 `__nv_ull2double_rd` 为核心的调用或声明。
- **L350 EN**: Executes a call or declaration centered on `__nv_ull2double_rn`.
  **L350 CN**: 执行以 `__nv_ull2double_rn` 为核心的调用或声明。
- **L351 EN**: Executes a call or declaration centered on `__nv_ull2double_ru`.
  **L351 CN**: 执行以 `__nv_ull2double_ru` 为核心的调用或声明。
- **L352 EN**: Executes a call or declaration centered on `__nv_ull2double_rz`.
  **L352 CN**: 执行以 `__nv_ull2double_rz` 为核心的调用或声明。
- **L353 EN**: Executes a call or declaration centered on `__nv_ull2float_rd`.
  **L353 CN**: 执行以 `__nv_ull2float_rd` 为核心的调用或声明。
- **L354 EN**: Executes a call or declaration centered on `__nv_ull2float_rn`.
  **L354 CN**: 执行以 `__nv_ull2float_rn` 为核心的调用或声明。
- **L355 EN**: Executes a call or declaration centered on `__nv_ull2float_ru`.
  **L355 CN**: 执行以 `__nv_ull2float_ru` 为核心的调用或声明。
- **L356 EN**: Executes a call or declaration centered on `__nv_ull2float_rz`.
  **L356 CN**: 执行以 `__nv_ull2float_rz` 为核心的调用或声明。
- **L357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __nv_ullmax(unsigned long long __a,`.
  **L357 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __nv_ullmax(unsigned long long __a,`。
- **L358 EN**: Adds a standalone statement or declaration: `unsigned long long __b);`.
  **L358 CN**: 添加一条独立语句或声明：`unsigned long long __b);`。
- **L359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __nv_ullmin(unsigned long long __a,`.
  **L359 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __nv_ullmin(unsigned long long __a,`。
- **L360 EN**: Adds a standalone statement or declaration: `unsigned long long __b);`.
  **L360 CN**: 添加一条独立语句或声明：`unsigned long long __b);`。

### Lines 361-380

````c
__DEVICE__ unsigned int __nv_umax(unsigned int __a, unsigned int __b);
__DEVICE__ unsigned int __nv_umin(unsigned int __a, unsigned int __b);
__DEVICE__ unsigned int __nv_umul24(unsigned int __a, unsigned int __b);
__DEVICE__ unsigned long long __nv_umul64hi(unsigned long long __a,
                                            unsigned long long __b);
__DEVICE__ unsigned int __nv_umulhi(unsigned int __a, unsigned int __b);
__DEVICE__ unsigned int __nv_urhadd(unsigned int __a, unsigned int __b);
__DEVICE__ unsigned int __nv_usad(unsigned int __a, unsigned int __b,
                                  unsigned int __c);
#if CUDA_VERSION >= 9000 && CUDA_VERSION < 9020
__DEVICE__ int __nv_vabs2(int __a);
__DEVICE__ int __nv_vabs4(int __a);
__DEVICE__ int __nv_vabsdiffs2(int __a, int __b);
__DEVICE__ int __nv_vabsdiffs4(int __a, int __b);
__DEVICE__ int __nv_vabsdiffu2(int __a, int __b);
__DEVICE__ int __nv_vabsdiffu4(int __a, int __b);
__DEVICE__ int __nv_vabsss2(int __a);
__DEVICE__ int __nv_vabsss4(int __a);
__DEVICE__ int __nv_vadd2(int __a, int __b);
__DEVICE__ int __nv_vadd4(int __a, int __b);
````
- **L361 EN**: Executes a call or declaration centered on `__nv_umax`.
  **L361 CN**: 执行以 `__nv_umax` 为核心的调用或声明。
- **L362 EN**: Executes a call or declaration centered on `__nv_umin`.
  **L362 CN**: 执行以 `__nv_umin` 为核心的调用或声明。
- **L363 EN**: Executes a call or declaration centered on `__nv_umul24`.
  **L363 CN**: 执行以 `__nv_umul24` 为核心的调用或声明。
- **L364 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned long long __nv_umul64hi(unsigned long long __a,`.
  **L364 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned long long __nv_umul64hi(unsigned long long __a,`。
- **L365 EN**: Adds a standalone statement or declaration: `unsigned long long __b);`.
  **L365 CN**: 添加一条独立语句或声明：`unsigned long long __b);`。
- **L366 EN**: Executes a call or declaration centered on `__nv_umulhi`.
  **L366 CN**: 执行以 `__nv_umulhi` 为核心的调用或声明。
- **L367 EN**: Executes a call or declaration centered on `__nv_urhadd`.
  **L367 CN**: 执行以 `__nv_urhadd` 为核心的调用或声明。
- **L368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__DEVICE__ unsigned int __nv_usad(unsigned int __a, unsigned int __b,`.
  **L368 CN**: 继续一个多行参数列表、初始化器或聚合项：`__DEVICE__ unsigned int __nv_usad(unsigned int __a, unsigned int __b,`。
- **L369 EN**: Adds a standalone statement or declaration: `unsigned int __c);`.
  **L369 CN**: 添加一条独立语句或声明：`unsigned int __c);`。
- **L370 EN**: Starts a preprocessor conditional block: `#if CUDA_VERSION >= 9000 && CUDA_VERSION < 9020`.
  **L370 CN**: 开始一个预处理条件块：`#if CUDA_VERSION >= 9000 && CUDA_VERSION < 9020`。
- **L371 EN**: Executes a call or declaration centered on `__nv_vabs2`.
  **L371 CN**: 执行以 `__nv_vabs2` 为核心的调用或声明。
- **L372 EN**: Executes a call or declaration centered on `__nv_vabs4`.
  **L372 CN**: 执行以 `__nv_vabs4` 为核心的调用或声明。
- **L373 EN**: Executes a call or declaration centered on `__nv_vabsdiffs2`.
  **L373 CN**: 执行以 `__nv_vabsdiffs2` 为核心的调用或声明。
- **L374 EN**: Executes a call or declaration centered on `__nv_vabsdiffs4`.
  **L374 CN**: 执行以 `__nv_vabsdiffs4` 为核心的调用或声明。
- **L375 EN**: Executes a call or declaration centered on `__nv_vabsdiffu2`.
  **L375 CN**: 执行以 `__nv_vabsdiffu2` 为核心的调用或声明。
- **L376 EN**: Executes a call or declaration centered on `__nv_vabsdiffu4`.
  **L376 CN**: 执行以 `__nv_vabsdiffu4` 为核心的调用或声明。
- **L377 EN**: Executes a call or declaration centered on `__nv_vabsss2`.
  **L377 CN**: 执行以 `__nv_vabsss2` 为核心的调用或声明。
- **L378 EN**: Executes a call or declaration centered on `__nv_vabsss4`.
  **L378 CN**: 执行以 `__nv_vabsss4` 为核心的调用或声明。
- **L379 EN**: Executes a call or declaration centered on `__nv_vadd2`.
  **L379 CN**: 执行以 `__nv_vadd2` 为核心的调用或声明。
- **L380 EN**: Executes a call or declaration centered on `__nv_vadd4`.
  **L380 CN**: 执行以 `__nv_vadd4` 为核心的调用或声明。

### Lines 381-400

````c
__DEVICE__ int __nv_vaddss2(int __a, int __b);
__DEVICE__ int __nv_vaddss4(int __a, int __b);
__DEVICE__ int __nv_vaddus2(int __a, int __b);
__DEVICE__ int __nv_vaddus4(int __a, int __b);
__DEVICE__ int __nv_vavgs2(int __a, int __b);
__DEVICE__ int __nv_vavgs4(int __a, int __b);
__DEVICE__ int __nv_vavgu2(int __a, int __b);
__DEVICE__ int __nv_vavgu4(int __a, int __b);
__DEVICE__ int __nv_vcmpeq2(int __a, int __b);
__DEVICE__ int __nv_vcmpeq4(int __a, int __b);
__DEVICE__ int __nv_vcmpges2(int __a, int __b);
__DEVICE__ int __nv_vcmpges4(int __a, int __b);
__DEVICE__ int __nv_vcmpgeu2(int __a, int __b);
__DEVICE__ int __nv_vcmpgeu4(int __a, int __b);
__DEVICE__ int __nv_vcmpgts2(int __a, int __b);
__DEVICE__ int __nv_vcmpgts4(int __a, int __b);
__DEVICE__ int __nv_vcmpgtu2(int __a, int __b);
__DEVICE__ int __nv_vcmpgtu4(int __a, int __b);
__DEVICE__ int __nv_vcmples2(int __a, int __b);
__DEVICE__ int __nv_vcmples4(int __a, int __b);
````
- **L381 EN**: Executes a call or declaration centered on `__nv_vaddss2`.
  **L381 CN**: 执行以 `__nv_vaddss2` 为核心的调用或声明。
- **L382 EN**: Executes a call or declaration centered on `__nv_vaddss4`.
  **L382 CN**: 执行以 `__nv_vaddss4` 为核心的调用或声明。
- **L383 EN**: Executes a call or declaration centered on `__nv_vaddus2`.
  **L383 CN**: 执行以 `__nv_vaddus2` 为核心的调用或声明。
- **L384 EN**: Executes a call or declaration centered on `__nv_vaddus4`.
  **L384 CN**: 执行以 `__nv_vaddus4` 为核心的调用或声明。
- **L385 EN**: Executes a call or declaration centered on `__nv_vavgs2`.
  **L385 CN**: 执行以 `__nv_vavgs2` 为核心的调用或声明。
- **L386 EN**: Executes a call or declaration centered on `__nv_vavgs4`.
  **L386 CN**: 执行以 `__nv_vavgs4` 为核心的调用或声明。
- **L387 EN**: Executes a call or declaration centered on `__nv_vavgu2`.
  **L387 CN**: 执行以 `__nv_vavgu2` 为核心的调用或声明。
- **L388 EN**: Executes a call or declaration centered on `__nv_vavgu4`.
  **L388 CN**: 执行以 `__nv_vavgu4` 为核心的调用或声明。
- **L389 EN**: Executes a call or declaration centered on `__nv_vcmpeq2`.
  **L389 CN**: 执行以 `__nv_vcmpeq2` 为核心的调用或声明。
- **L390 EN**: Executes a call or declaration centered on `__nv_vcmpeq4`.
  **L390 CN**: 执行以 `__nv_vcmpeq4` 为核心的调用或声明。
- **L391 EN**: Executes a call or declaration centered on `__nv_vcmpges2`.
  **L391 CN**: 执行以 `__nv_vcmpges2` 为核心的调用或声明。
- **L392 EN**: Executes a call or declaration centered on `__nv_vcmpges4`.
  **L392 CN**: 执行以 `__nv_vcmpges4` 为核心的调用或声明。
- **L393 EN**: Executes a call or declaration centered on `__nv_vcmpgeu2`.
  **L393 CN**: 执行以 `__nv_vcmpgeu2` 为核心的调用或声明。
- **L394 EN**: Executes a call or declaration centered on `__nv_vcmpgeu4`.
  **L394 CN**: 执行以 `__nv_vcmpgeu4` 为核心的调用或声明。
- **L395 EN**: Executes a call or declaration centered on `__nv_vcmpgts2`.
  **L395 CN**: 执行以 `__nv_vcmpgts2` 为核心的调用或声明。
- **L396 EN**: Executes a call or declaration centered on `__nv_vcmpgts4`.
  **L396 CN**: 执行以 `__nv_vcmpgts4` 为核心的调用或声明。
- **L397 EN**: Executes a call or declaration centered on `__nv_vcmpgtu2`.
  **L397 CN**: 执行以 `__nv_vcmpgtu2` 为核心的调用或声明。
- **L398 EN**: Executes a call or declaration centered on `__nv_vcmpgtu4`.
  **L398 CN**: 执行以 `__nv_vcmpgtu4` 为核心的调用或声明。
- **L399 EN**: Executes a call or declaration centered on `__nv_vcmples2`.
  **L399 CN**: 执行以 `__nv_vcmples2` 为核心的调用或声明。
- **L400 EN**: Executes a call or declaration centered on `__nv_vcmples4`.
  **L400 CN**: 执行以 `__nv_vcmples4` 为核心的调用或声明。

### Lines 401-420

````c
__DEVICE__ int __nv_vcmpleu2(int __a, int __b);
__DEVICE__ int __nv_vcmpleu4(int __a, int __b);
__DEVICE__ int __nv_vcmplts2(int __a, int __b);
__DEVICE__ int __nv_vcmplts4(int __a, int __b);
__DEVICE__ int __nv_vcmpltu2(int __a, int __b);
__DEVICE__ int __nv_vcmpltu4(int __a, int __b);
__DEVICE__ int __nv_vcmpne2(int __a, int __b);
__DEVICE__ int __nv_vcmpne4(int __a, int __b);
__DEVICE__ int __nv_vhaddu2(int __a, int __b);
__DEVICE__ int __nv_vhaddu4(int __a, int __b);
__DEVICE__ int __nv_vmaxs2(int __a, int __b);
__DEVICE__ int __nv_vmaxs4(int __a, int __b);
__DEVICE__ int __nv_vmaxu2(int __a, int __b);
__DEVICE__ int __nv_vmaxu4(int __a, int __b);
__DEVICE__ int __nv_vmins2(int __a, int __b);
__DEVICE__ int __nv_vmins4(int __a, int __b);
__DEVICE__ int __nv_vminu2(int __a, int __b);
__DEVICE__ int __nv_vminu4(int __a, int __b);
__DEVICE__ int __nv_vneg2(int __a);
__DEVICE__ int __nv_vneg4(int __a);
````
- **L401 EN**: Executes a call or declaration centered on `__nv_vcmpleu2`.
  **L401 CN**: 执行以 `__nv_vcmpleu2` 为核心的调用或声明。
- **L402 EN**: Executes a call or declaration centered on `__nv_vcmpleu4`.
  **L402 CN**: 执行以 `__nv_vcmpleu4` 为核心的调用或声明。
- **L403 EN**: Executes a call or declaration centered on `__nv_vcmplts2`.
  **L403 CN**: 执行以 `__nv_vcmplts2` 为核心的调用或声明。
- **L404 EN**: Executes a call or declaration centered on `__nv_vcmplts4`.
  **L404 CN**: 执行以 `__nv_vcmplts4` 为核心的调用或声明。
- **L405 EN**: Executes a call or declaration centered on `__nv_vcmpltu2`.
  **L405 CN**: 执行以 `__nv_vcmpltu2` 为核心的调用或声明。
- **L406 EN**: Executes a call or declaration centered on `__nv_vcmpltu4`.
  **L406 CN**: 执行以 `__nv_vcmpltu4` 为核心的调用或声明。
- **L407 EN**: Executes a call or declaration centered on `__nv_vcmpne2`.
  **L407 CN**: 执行以 `__nv_vcmpne2` 为核心的调用或声明。
- **L408 EN**: Executes a call or declaration centered on `__nv_vcmpne4`.
  **L408 CN**: 执行以 `__nv_vcmpne4` 为核心的调用或声明。
- **L409 EN**: Executes a call or declaration centered on `__nv_vhaddu2`.
  **L409 CN**: 执行以 `__nv_vhaddu2` 为核心的调用或声明。
- **L410 EN**: Executes a call or declaration centered on `__nv_vhaddu4`.
  **L410 CN**: 执行以 `__nv_vhaddu4` 为核心的调用或声明。
- **L411 EN**: Executes a call or declaration centered on `__nv_vmaxs2`.
  **L411 CN**: 执行以 `__nv_vmaxs2` 为核心的调用或声明。
- **L412 EN**: Executes a call or declaration centered on `__nv_vmaxs4`.
  **L412 CN**: 执行以 `__nv_vmaxs4` 为核心的调用或声明。
- **L413 EN**: Executes a call or declaration centered on `__nv_vmaxu2`.
  **L413 CN**: 执行以 `__nv_vmaxu2` 为核心的调用或声明。
- **L414 EN**: Executes a call or declaration centered on `__nv_vmaxu4`.
  **L414 CN**: 执行以 `__nv_vmaxu4` 为核心的调用或声明。
- **L415 EN**: Executes a call or declaration centered on `__nv_vmins2`.
  **L415 CN**: 执行以 `__nv_vmins2` 为核心的调用或声明。
- **L416 EN**: Executes a call or declaration centered on `__nv_vmins4`.
  **L416 CN**: 执行以 `__nv_vmins4` 为核心的调用或声明。
- **L417 EN**: Executes a call or declaration centered on `__nv_vminu2`.
  **L417 CN**: 执行以 `__nv_vminu2` 为核心的调用或声明。
- **L418 EN**: Executes a call or declaration centered on `__nv_vminu4`.
  **L418 CN**: 执行以 `__nv_vminu4` 为核心的调用或声明。
- **L419 EN**: Executes a call or declaration centered on `__nv_vneg2`.
  **L419 CN**: 执行以 `__nv_vneg2` 为核心的调用或声明。
- **L420 EN**: Executes a call or declaration centered on `__nv_vneg4`.
  **L420 CN**: 执行以 `__nv_vneg4` 为核心的调用或声明。

### Lines 421-440

````c
__DEVICE__ int __nv_vnegss2(int __a);
__DEVICE__ int __nv_vnegss4(int __a);
__DEVICE__ int __nv_vsads2(int __a, int __b);
__DEVICE__ int __nv_vsads4(int __a, int __b);
__DEVICE__ int __nv_vsadu2(int __a, int __b);
__DEVICE__ int __nv_vsadu4(int __a, int __b);
__DEVICE__ int __nv_vseteq2(int __a, int __b);
__DEVICE__ int __nv_vseteq4(int __a, int __b);
__DEVICE__ int __nv_vsetges2(int __a, int __b);
__DEVICE__ int __nv_vsetges4(int __a, int __b);
__DEVICE__ int __nv_vsetgeu2(int __a, int __b);
__DEVICE__ int __nv_vsetgeu4(int __a, int __b);
__DEVICE__ int __nv_vsetgts2(int __a, int __b);
__DEVICE__ int __nv_vsetgts4(int __a, int __b);
__DEVICE__ int __nv_vsetgtu2(int __a, int __b);
__DEVICE__ int __nv_vsetgtu4(int __a, int __b);
__DEVICE__ int __nv_vsetles2(int __a, int __b);
__DEVICE__ int __nv_vsetles4(int __a, int __b);
__DEVICE__ int __nv_vsetleu2(int __a, int __b);
__DEVICE__ int __nv_vsetleu4(int __a, int __b);
````
- **L421 EN**: Executes a call or declaration centered on `__nv_vnegss2`.
  **L421 CN**: 执行以 `__nv_vnegss2` 为核心的调用或声明。
- **L422 EN**: Executes a call or declaration centered on `__nv_vnegss4`.
  **L422 CN**: 执行以 `__nv_vnegss4` 为核心的调用或声明。
- **L423 EN**: Executes a call or declaration centered on `__nv_vsads2`.
  **L423 CN**: 执行以 `__nv_vsads2` 为核心的调用或声明。
- **L424 EN**: Executes a call or declaration centered on `__nv_vsads4`.
  **L424 CN**: 执行以 `__nv_vsads4` 为核心的调用或声明。
- **L425 EN**: Executes a call or declaration centered on `__nv_vsadu2`.
  **L425 CN**: 执行以 `__nv_vsadu2` 为核心的调用或声明。
- **L426 EN**: Executes a call or declaration centered on `__nv_vsadu4`.
  **L426 CN**: 执行以 `__nv_vsadu4` 为核心的调用或声明。
- **L427 EN**: Executes a call or declaration centered on `__nv_vseteq2`.
  **L427 CN**: 执行以 `__nv_vseteq2` 为核心的调用或声明。
- **L428 EN**: Executes a call or declaration centered on `__nv_vseteq4`.
  **L428 CN**: 执行以 `__nv_vseteq4` 为核心的调用或声明。
- **L429 EN**: Executes a call or declaration centered on `__nv_vsetges2`.
  **L429 CN**: 执行以 `__nv_vsetges2` 为核心的调用或声明。
- **L430 EN**: Executes a call or declaration centered on `__nv_vsetges4`.
  **L430 CN**: 执行以 `__nv_vsetges4` 为核心的调用或声明。
- **L431 EN**: Executes a call or declaration centered on `__nv_vsetgeu2`.
  **L431 CN**: 执行以 `__nv_vsetgeu2` 为核心的调用或声明。
- **L432 EN**: Executes a call or declaration centered on `__nv_vsetgeu4`.
  **L432 CN**: 执行以 `__nv_vsetgeu4` 为核心的调用或声明。
- **L433 EN**: Executes a call or declaration centered on `__nv_vsetgts2`.
  **L433 CN**: 执行以 `__nv_vsetgts2` 为核心的调用或声明。
- **L434 EN**: Executes a call or declaration centered on `__nv_vsetgts4`.
  **L434 CN**: 执行以 `__nv_vsetgts4` 为核心的调用或声明。
- **L435 EN**: Executes a call or declaration centered on `__nv_vsetgtu2`.
  **L435 CN**: 执行以 `__nv_vsetgtu2` 为核心的调用或声明。
- **L436 EN**: Executes a call or declaration centered on `__nv_vsetgtu4`.
  **L436 CN**: 执行以 `__nv_vsetgtu4` 为核心的调用或声明。
- **L437 EN**: Executes a call or declaration centered on `__nv_vsetles2`.
  **L437 CN**: 执行以 `__nv_vsetles2` 为核心的调用或声明。
- **L438 EN**: Executes a call or declaration centered on `__nv_vsetles4`.
  **L438 CN**: 执行以 `__nv_vsetles4` 为核心的调用或声明。
- **L439 EN**: Executes a call or declaration centered on `__nv_vsetleu2`.
  **L439 CN**: 执行以 `__nv_vsetleu2` 为核心的调用或声明。
- **L440 EN**: Executes a call or declaration centered on `__nv_vsetleu4`.
  **L440 CN**: 执行以 `__nv_vsetleu4` 为核心的调用或声明。

### Lines 441-460

````c
__DEVICE__ int __nv_vsetlts2(int __a, int __b);
__DEVICE__ int __nv_vsetlts4(int __a, int __b);
__DEVICE__ int __nv_vsetltu2(int __a, int __b);
__DEVICE__ int __nv_vsetltu4(int __a, int __b);
__DEVICE__ int __nv_vsetne2(int __a, int __b);
__DEVICE__ int __nv_vsetne4(int __a, int __b);
__DEVICE__ int __nv_vsub2(int __a, int __b);
__DEVICE__ int __nv_vsub4(int __a, int __b);
__DEVICE__ int __nv_vsubss2(int __a, int __b);
__DEVICE__ int __nv_vsubss4(int __a, int __b);
__DEVICE__ int __nv_vsubus2(int __a, int __b);
__DEVICE__ int __nv_vsubus4(int __a, int __b);
#endif  // CUDA_VERSION
__DEVICE__ double __nv_y0(double __a);
__DEVICE__ float __nv_y0f(float __a);
__DEVICE__ double __nv_y1(double __a);
__DEVICE__ float __nv_y1f(float __a);
__DEVICE__ float __nv_ynf(int __a, float __b);
__DEVICE__ double __nv_yn(int __a, double __b);

````
- **L441 EN**: Executes a call or declaration centered on `__nv_vsetlts2`.
  **L441 CN**: 执行以 `__nv_vsetlts2` 为核心的调用或声明。
- **L442 EN**: Executes a call or declaration centered on `__nv_vsetlts4`.
  **L442 CN**: 执行以 `__nv_vsetlts4` 为核心的调用或声明。
- **L443 EN**: Executes a call or declaration centered on `__nv_vsetltu2`.
  **L443 CN**: 执行以 `__nv_vsetltu2` 为核心的调用或声明。
- **L444 EN**: Executes a call or declaration centered on `__nv_vsetltu4`.
  **L444 CN**: 执行以 `__nv_vsetltu4` 为核心的调用或声明。
- **L445 EN**: Executes a call or declaration centered on `__nv_vsetne2`.
  **L445 CN**: 执行以 `__nv_vsetne2` 为核心的调用或声明。
- **L446 EN**: Executes a call or declaration centered on `__nv_vsetne4`.
  **L446 CN**: 执行以 `__nv_vsetne4` 为核心的调用或声明。
- **L447 EN**: Executes a call or declaration centered on `__nv_vsub2`.
  **L447 CN**: 执行以 `__nv_vsub2` 为核心的调用或声明。
- **L448 EN**: Executes a call or declaration centered on `__nv_vsub4`.
  **L448 CN**: 执行以 `__nv_vsub4` 为核心的调用或声明。
- **L449 EN**: Executes a call or declaration centered on `__nv_vsubss2`.
  **L449 CN**: 执行以 `__nv_vsubss2` 为核心的调用或声明。
- **L450 EN**: Executes a call or declaration centered on `__nv_vsubss4`.
  **L450 CN**: 执行以 `__nv_vsubss4` 为核心的调用或声明。
- **L451 EN**: Executes a call or declaration centered on `__nv_vsubus2`.
  **L451 CN**: 执行以 `__nv_vsubus2` 为核心的调用或声明。
- **L452 EN**: Executes a call or declaration centered on `__nv_vsubus4`.
  **L452 CN**: 执行以 `__nv_vsubus4` 为核心的调用或声明。
- **L453 EN**: Closes the current preprocessor conditional block.
  **L453 CN**: 结束当前预处理条件块。
- **L454 EN**: Executes a call or declaration centered on `__nv_y0`.
  **L454 CN**: 执行以 `__nv_y0` 为核心的调用或声明。
- **L455 EN**: Executes a call or declaration centered on `__nv_y0f`.
  **L455 CN**: 执行以 `__nv_y0f` 为核心的调用或声明。
- **L456 EN**: Executes a call or declaration centered on `__nv_y1`.
  **L456 CN**: 执行以 `__nv_y1` 为核心的调用或声明。
- **L457 EN**: Executes a call or declaration centered on `__nv_y1f`.
  **L457 CN**: 执行以 `__nv_y1f` 为核心的调用或声明。
- **L458 EN**: Executes a call or declaration centered on `__nv_ynf`.
  **L458 CN**: 执行以 `__nv_ynf` 为核心的调用或声明。
- **L459 EN**: Executes a call or declaration centered on `__nv_yn`.
  **L459 CN**: 执行以 `__nv_yn` 为核心的调用或声明。
- **L460 EN**: Blank line separating nearby declarations or logic blocks.
  **L460 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 461-468

````c
#if defined(__OPENMP_NVPTX__)
#pragma omp end assumes ext_spmd_amenable no_openmp
#endif

#if defined(__cplusplus)
} // extern "C"
#endif
#endif // __CLANG_CUDA_LIBDEVICE_DECLARES_H__
````
- **L461 EN**: Starts a preprocessor conditional block: `#if defined(__OPENMP_NVPTX__)`.
  **L461 CN**: 开始一个预处理条件块：`#if defined(__OPENMP_NVPTX__)`。
- **L462 EN**: Applies a compiler pragma that adjusts parsing, attributes, or macro state: `#pragma omp end assumes ext_spmd_amenable no_openmp`.
  **L462 CN**: 应用编译器 pragma，调整解析、属性或宏状态：`#pragma omp end assumes ext_spmd_amenable no_openmp`。
- **L463 EN**: Closes the current preprocessor conditional block.
  **L463 CN**: 结束当前预处理条件块。
- **L464 EN**: Blank line separating nearby declarations or logic blocks.
  **L464 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L465 EN**: Starts a preprocessor conditional block: `#if defined(__cplusplus)`.
  **L465 CN**: 开始一个预处理条件块：`#if defined(__cplusplus)`。
- **L466 EN**: Continues the surrounding expression or declaration: `} // extern "C"`.
  **L466 CN**: 继续构造周围的表达式或声明：`} // extern "C"`。
- **L467 EN**: Closes the current preprocessor conditional block.
  **L467 CN**: 结束当前预处理条件块。
- **L468 EN**: Closes the current preprocessor conditional block.
  **L468 CN**: 结束当前预处理条件块。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **CUDA device support / CUDA 设备支持**
- **Macro-based API construction / 基于宏的 API 构造**
- **Pragma-controlled behavior / 由 pragma 控制的行为**
- **GPU device annotations / GPU 设备注解**
- **Header inclusion control / 头文件包含控制**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None. / 无。
- **Conditional macros / 条件宏**: `__CLANG_CUDA_LIBDEVICE_DECLARES_H__`, `__cplusplus`, `__OPENMP_NVPTX__`, `__CUDA__`
- **External builtins / 外部 builtin**: `__nv_abs`, `__nv_acos`, `__nv_acosf`, `__nv_acosh`, `__nv_acoshf`, `__nv_asin`, `__nv_asinf`, `__nv_asinh`, `__nv_asinhf`, `__nv_atan2`, `__nv_atan2f`, `__nv_atan`
