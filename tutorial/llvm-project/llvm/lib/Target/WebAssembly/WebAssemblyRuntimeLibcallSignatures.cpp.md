# WebAssemblyRuntimeLibcallSignatures.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyRuntimeLibcallSignatures.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file contains signature information for runtime libcalls.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyRuntimeLibcallSignatures.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
// CodeGen/RuntimeLibcallSignatures.cpp - R.T. Lib. Call Signatures -*- C++ -*--
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-18

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file contains signature information for runtime libcalls.
///
/// CodeGen uses external symbols, which it refers to by name. The WebAssembly
/// target needs type information for all functions. This file contains a big
/// table providing type signatures for all runtime library functions that LLVM
/// uses.
///
/// This is currently a fairly heavy-handed solution.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 19-26

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssemblyRuntimeLibcallSignatures.h"
#include "WebAssemblySubtarget.h"
#include "WebAssemblyUtilities.h"

using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 27-64

```cpp
namespace {

enum RuntimeLibcallSignature {
  func,
  f32_func_f32,
  f32_func_f64,
  f32_func_i32,
  f32_func_i64,
  f32_func_i16,
  f64_func_f32,
  f64_func_f64,
  f64_func_i32,
  f64_func_i64,
  i32_func_f32,
  i32_func_f64,
  i32_func_i32,
  i64_func_f32,
  i64_func_f64,
  i64_func_i64,
  f32_func_f32_f32,
  f32_func_f32_i32,
  f32_func_f32_iPTR,
  f32_func_i64_i64,
  f64_func_f64_f64,
  f64_func_f64_i32,
  f64_func_f64_iPTR,
  f64_func_i64_i64,
  i16_func_f32,
  i16_func_f64,
  i16_func_i64_i64,
  i8_func_i8_i8,
  func_f32_iPTR_iPTR,
  func_f64_iPTR_iPTR,
  i16_func_i16_i16,
  i32_func_f32_f32,
  i32_func_f64_f64,
  i32_func_i32_i32,
  i32_func_i32_i32_iPTR,
```
- **EN**: Introduces enumerated values such as `RuntimeLibcallSignature` to make target-specific cases explicit and type-safe.
- **CN**: 这里引入枚举值，例如 `RuntimeLibcallSignature`，用更显式且更安全的方式表达目标相关分支。

### Lines 65-93

```cpp
  i64_func_i64_i64,
  i64_func_i64_i64_iPTR,
  i64_i64_func_i32,
  i64_i64_func_i64,
  i64_i64_func_f32,
  i64_i64_func_f64,
  i16_i16_func_i16_i16,
  i32_i32_func_i32_i32,
  i64_i64_func_i64_i64,
  i64_i64_func_i64_i64_iPTR,
  i64_i64_func_i64_i64_i64_i64,
  i64_i64_func_i64_i64_i64_i64_iPTR,
  i64_i64_i64_i64_func_i64_i64_i64_i64,
  i64_i64_func_i64_i64_i32,
  i64_i64_func_i64_i64_i64_i64_i64_i64,
  iPTR_func_i32,
  iPTR_func_iPTR_i32_iPTR,
  iPTR_func_iPTR_iPTR_iPTR,
  f32_func_f32_f32_f32,
  f64_func_f64_f64_f64,
  func_i64_i64_iPTR_iPTR,
  i32_func_i64_i64,
  i32_func_i64_i64_i64_i64,
  iPTR_func_f32,
  iPTR_func_f64,
  iPTR_func_i64_i64,
  unsupported
};
```
- **EN**: Continues the WebAssembly backend backend implementation logic with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的后端实现逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 94-131

```cpp
struct RuntimeLibcallSignatureTable {
  std::vector<RuntimeLibcallSignature> Table;

  // Any newly-added libcalls will be unsupported by default.
  RuntimeLibcallSignatureTable() : Table(RTLIB::UNKNOWN_LIBCALL, unsupported) {
    // Integer
    Table[RTLIB::SHL_I16] = i16_func_i16_i16;
    Table[RTLIB::SHL_I32] = i32_func_i32_i32;
    Table[RTLIB::SHL_I64] = i64_func_i64_i64;
    Table[RTLIB::SHL_I128] = i64_i64_func_i64_i64_i32;
    Table[RTLIB::SRL_I16] = i16_func_i16_i16;
    Table[RTLIB::SRL_I32] = i32_func_i32_i32;
    Table[RTLIB::SRL_I64] = i64_func_i64_i64;
    Table[RTLIB::SRL_I128] = i64_i64_func_i64_i64_i32;
    Table[RTLIB::SRA_I16] = i16_func_i16_i16;
    Table[RTLIB::SRA_I32] = i32_func_i32_i32;
    Table[RTLIB::SRA_I64] = i64_func_i64_i64;
    Table[RTLIB::SRA_I128] = i64_i64_func_i64_i64_i32;
    Table[RTLIB::MUL_I8] = i8_func_i8_i8;
    Table[RTLIB::MUL_I16] = i16_func_i16_i16;
    Table[RTLIB::MUL_I32] = i32_func_i32_i32;
    Table[RTLIB::MUL_I64] = i64_func_i64_i64;
    Table[RTLIB::MUL_I128] = i64_i64_func_i64_i64_i64_i64;
    Table[RTLIB::MULO_I32] = i32_func_i32_i32_iPTR;
    Table[RTLIB::MULO_I64] = i64_func_i64_i64_iPTR;
    Table[RTLIB::MULO_I128] = i64_i64_func_i64_i64_i64_i64_iPTR;
    Table[RTLIB::SDIV_I8] = i8_func_i8_i8;
    Table[RTLIB::SDIV_I16] = i16_func_i16_i16;
    Table[RTLIB::SDIV_I32] = i32_func_i32_i32;
    Table[RTLIB::SDIV_I64] = i64_func_i64_i64;
    Table[RTLIB::SDIV_I128] = i64_i64_func_i64_i64_i64_i64;
    Table[RTLIB::UDIV_I8] = i8_func_i8_i8;
    Table[RTLIB::UDIV_I16] = i16_func_i16_i16;
    Table[RTLIB::UDIV_I32] = i32_func_i32_i32;
    Table[RTLIB::UDIV_I64] = i64_func_i64_i64;
    Table[RTLIB::UDIV_I128] = i64_i64_func_i64_i64_i64_i64;
    Table[RTLIB::SREM_I8] = i8_func_i8_i8;
    Table[RTLIB::SREM_I16] = i16_func_i16_i16;
```
- **EN**: Declares a backend-facing type `RuntimeLibcallSignatureTable`, `Table` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `RuntimeLibcallSignatureTable`, `Table`，并勾勒出周边代码会依赖的接口或状态。

### Lines 132-169

```cpp
    Table[RTLIB::SREM_I32] = i32_func_i32_i32;
    Table[RTLIB::SREM_I64] = i64_func_i64_i64;
    Table[RTLIB::SREM_I128] = i64_i64_func_i64_i64_i64_i64;
    Table[RTLIB::UREM_I8] = i8_func_i8_i8;
    Table[RTLIB::UREM_I16] = i16_func_i16_i16;
    Table[RTLIB::UREM_I32] = i32_func_i32_i32;
    Table[RTLIB::UREM_I64] = i64_func_i64_i64;
    Table[RTLIB::UREM_I128] = i64_i64_func_i64_i64_i64_i64;
    Table[RTLIB::SDIVREM_I8] = i8_func_i8_i8;
    Table[RTLIB::SDIVREM_I16] = i16_i16_func_i16_i16;
    Table[RTLIB::SDIVREM_I32] = i32_i32_func_i32_i32;
    Table[RTLIB::SDIVREM_I64] = i64_func_i64_i64;
    Table[RTLIB::SDIVREM_I128] = i64_i64_i64_i64_func_i64_i64_i64_i64;
    Table[RTLIB::UDIVREM_I8] = i8_func_i8_i8;
    Table[RTLIB::UDIVREM_I16] = i16_i16_func_i16_i16;
    Table[RTLIB::UDIVREM_I32] = i32_i32_func_i32_i32;
    Table[RTLIB::UDIVREM_I64] = i64_i64_func_i64_i64;
    Table[RTLIB::UDIVREM_I128] = i64_i64_i64_i64_func_i64_i64_i64_i64;
    Table[RTLIB::NEG_I32] = i32_func_i32;
    Table[RTLIB::NEG_I64] = i64_func_i64;

    // Floating-point.
    // All F80 and PPCF128 routines are unsupported.
    Table[RTLIB::ADD_F32] = f32_func_f32_f32;
    Table[RTLIB::ADD_F64] = f64_func_f64_f64;
    Table[RTLIB::ADD_F128] = i64_i64_func_i64_i64_i64_i64;
    Table[RTLIB::SUB_F32] = f32_func_f32_f32;
    Table[RTLIB::SUB_F64] = f64_func_f64_f64;
    Table[RTLIB::SUB_F128] = i64_i64_func_i64_i64_i64_i64;
    Table[RTLIB::MUL_F32] = f32_func_f32_f32;
    Table[RTLIB::MUL_F64] = f64_func_f64_f64;
    Table[RTLIB::MUL_F128] = i64_i64_func_i64_i64_i64_i64;
    Table[RTLIB::DIV_F32] = f32_func_f32_f32;
    Table[RTLIB::DIV_F64] = f64_func_f64_f64;
    Table[RTLIB::DIV_F128] = i64_i64_func_i64_i64_i64_i64;
    Table[RTLIB::REM_F32] = f32_func_f32_f32;
    Table[RTLIB::REM_F64] = f64_func_f64_f64;
    Table[RTLIB::REM_F128] = i64_i64_func_i64_i64_i64_i64;
```
- **EN**: Continues the WebAssembly backend backend implementation logic with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的后端实现逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 170-207

```cpp
    Table[RTLIB::FMA_F32] = f32_func_f32_f32_f32;
    Table[RTLIB::FMA_F64] = f64_func_f64_f64_f64;
    Table[RTLIB::FMA_F128] = i64_i64_func_i64_i64_i64_i64_i64_i64;
    Table[RTLIB::POWI_F32] = f32_func_f32_i32;
    Table[RTLIB::POWI_F64] = f64_func_f64_i32;
    Table[RTLIB::POWI_F128] = i64_i64_func_i64_i64_i32;
    Table[RTLIB::SQRT_F32] = f32_func_f32;
    Table[RTLIB::SQRT_F64] = f64_func_f64;
    Table[RTLIB::SQRT_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::CBRT_F32] = f32_func_f32;
    Table[RTLIB::CBRT_F64] = f64_func_f64;
    Table[RTLIB::CBRT_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::LOG_F32] = f32_func_f32;
    Table[RTLIB::LOG_F64] = f64_func_f64;
    Table[RTLIB::LOG_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::LOG2_F32] = f32_func_f32;
    Table[RTLIB::LOG2_F64] = f64_func_f64;
    Table[RTLIB::LOG2_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::LOG10_F32] = f32_func_f32;
    Table[RTLIB::LOG10_F64] = f64_func_f64;
    Table[RTLIB::LOG10_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::EXP_F32] = f32_func_f32;
    Table[RTLIB::EXP_F64] = f64_func_f64;
    Table[RTLIB::EXP_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::EXP2_F32] = f32_func_f32;
    Table[RTLIB::EXP2_F64] = f64_func_f64;
    Table[RTLIB::EXP2_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::EXP10_F32] = f32_func_f32;
    Table[RTLIB::EXP10_F64] = f64_func_f64;
    Table[RTLIB::EXP10_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::SIN_F32] = f32_func_f32;
    Table[RTLIB::SIN_F64] = f64_func_f64;
    Table[RTLIB::SIN_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::COS_F32] = f32_func_f32;
    Table[RTLIB::COS_F64] = f64_func_f64;
    Table[RTLIB::COS_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::TAN_F32] = f32_func_f32;
    Table[RTLIB::TAN_F64] = f64_func_f64;
```
- **EN**: Continues the WebAssembly backend backend implementation logic with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的后端实现逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 208-245

```cpp
    Table[RTLIB::TAN_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::ASIN_F32] = f32_func_f32;
    Table[RTLIB::ASIN_F64] = f64_func_f64;
    Table[RTLIB::ASIN_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::ACOS_F32] = f32_func_f32;
    Table[RTLIB::ACOS_F64] = f64_func_f64;
    Table[RTLIB::ACOS_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::ATAN_F32] = f32_func_f32;
    Table[RTLIB::ATAN_F64] = f64_func_f64;
    Table[RTLIB::ATAN_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::ATAN2_F32] = f32_func_f32_f32;
    Table[RTLIB::ATAN2_F64] = f64_func_f64_f64;
    Table[RTLIB::ATAN2_F128] = i64_i64_func_i64_i64_i64_i64;
    Table[RTLIB::SINH_F32] = f32_func_f32;
    Table[RTLIB::SINH_F64] = f64_func_f64;
    Table[RTLIB::SINH_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::COSH_F32] = f32_func_f32;
    Table[RTLIB::COSH_F64] = f64_func_f64;
    Table[RTLIB::COSH_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::TANH_F32] = f32_func_f32;
    Table[RTLIB::TANH_F64] = f64_func_f64;
    Table[RTLIB::TANH_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::SINCOS_F32] = func_f32_iPTR_iPTR;
    Table[RTLIB::SINCOS_F64] = func_f64_iPTR_iPTR;
    Table[RTLIB::SINCOS_F128] = func_i64_i64_iPTR_iPTR;
    Table[RTLIB::POW_F32] = f32_func_f32_f32;
    Table[RTLIB::POW_F64] = f64_func_f64_f64;
    Table[RTLIB::POW_F128] = i64_i64_func_i64_i64_i64_i64;
    Table[RTLIB::CEIL_F32] = f32_func_f32;
    Table[RTLIB::CEIL_F64] = f64_func_f64;
    Table[RTLIB::CEIL_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::TRUNC_F32] = f32_func_f32;
    Table[RTLIB::TRUNC_F64] = f64_func_f64;
    Table[RTLIB::TRUNC_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::RINT_F32] = f32_func_f32;
    Table[RTLIB::RINT_F64] = f64_func_f64;
    Table[RTLIB::RINT_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::NEARBYINT_F32] = f32_func_f32;
```
- **EN**: Continues the WebAssembly backend backend implementation logic with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的后端实现逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 246-283

```cpp
    Table[RTLIB::NEARBYINT_F64] = f64_func_f64;
    Table[RTLIB::NEARBYINT_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::ROUND_F32] = f32_func_f32;
    Table[RTLIB::ROUND_F64] = f64_func_f64;
    Table[RTLIB::ROUND_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::ROUNDEVEN_F32] = f32_func_f32;
    Table[RTLIB::ROUNDEVEN_F64] = f64_func_f64;
    Table[RTLIB::ROUNDEVEN_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::LROUND_F32] = iPTR_func_f32;
    Table[RTLIB::LROUND_F64] = iPTR_func_f64;
    Table[RTLIB::LROUND_F128] = iPTR_func_i64_i64;
    Table[RTLIB::LLROUND_F32] = i64_func_f32;
    Table[RTLIB::LLROUND_F64] = i64_func_f64;
    Table[RTLIB::LLROUND_F128] = i64_func_i64_i64;
    Table[RTLIB::LRINT_F32] = iPTR_func_f32;
    Table[RTLIB::LRINT_F64] = iPTR_func_f64;
    Table[RTLIB::LRINT_F128] = iPTR_func_i64_i64;
    Table[RTLIB::LLRINT_F32] = i64_func_f32;
    Table[RTLIB::LLRINT_F64] = i64_func_f64;
    Table[RTLIB::LLRINT_F128] = i64_func_i64_i64;
    Table[RTLIB::FLOOR_F32] = f32_func_f32;
    Table[RTLIB::FLOOR_F64] = f64_func_f64;
    Table[RTLIB::FLOOR_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::COPYSIGN_F32] = f32_func_f32_f32;
    Table[RTLIB::COPYSIGN_F64] = f64_func_f64_f64;
    Table[RTLIB::COPYSIGN_F128] = i64_i64_func_i64_i64_i64_i64;
    Table[RTLIB::FMIN_F32] = f32_func_f32_f32;
    Table[RTLIB::FMIN_F64] = f64_func_f64_f64;
    Table[RTLIB::FMIN_F128] = i64_i64_func_i64_i64_i64_i64;
    Table[RTLIB::FMAX_F32] = f32_func_f32_f32;
    Table[RTLIB::FMAX_F64] = f64_func_f64_f64;
    Table[RTLIB::FMAX_F128] = i64_i64_func_i64_i64_i64_i64;
    Table[RTLIB::LDEXP_F32] = f32_func_f32_i32;
    Table[RTLIB::LDEXP_F64] = f64_func_f64_i32;
    Table[RTLIB::LDEXP_F128] = i64_i64_func_i64_i64_i32;
    Table[RTLIB::FREXP_F32] = f32_func_f32_iPTR;
    Table[RTLIB::FREXP_F64] = f64_func_f64_iPTR;
    Table[RTLIB::FREXP_F128] = i64_i64_func_i64_i64_iPTR;
```
- **EN**: Continues the WebAssembly backend backend implementation logic with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的后端实现逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 284-321

```cpp
    Table[RTLIB::MODF_F32] = f32_func_f32_iPTR;
    Table[RTLIB::MODF_F64] = f64_func_f64_iPTR;
    Table[RTLIB::MODF_F128] = i64_i64_func_i64_i64_iPTR;

    // Conversion
    // All F80 and PPCF128 routines are unsupported.
    Table[RTLIB::FPEXT_F64_F128] = i64_i64_func_f64;
    Table[RTLIB::FPEXT_F32_F128] = i64_i64_func_f32;
    Table[RTLIB::FPEXT_F32_F64] = f64_func_f32;
    Table[RTLIB::FPEXT_F16_F32] = f32_func_i16;
    Table[RTLIB::FPROUND_F32_F16] = i16_func_f32;
    Table[RTLIB::FPROUND_F64_F16] = i16_func_f64;
    Table[RTLIB::FPROUND_F64_F32] = f32_func_f64;
    Table[RTLIB::FPROUND_F128_F16] = i16_func_i64_i64;
    Table[RTLIB::FPROUND_F128_F32] = f32_func_i64_i64;
    Table[RTLIB::FPROUND_F128_F64] = f64_func_i64_i64;
    Table[RTLIB::FPTOSINT_F32_I32] = i32_func_f32;
    Table[RTLIB::FPTOSINT_F32_I64] = i64_func_f32;
    Table[RTLIB::FPTOSINT_F32_I128] = i64_i64_func_f32;
    Table[RTLIB::FPTOSINT_F64_I32] = i32_func_f64;
    Table[RTLIB::FPTOSINT_F64_I64] = i64_func_f64;
    Table[RTLIB::FPTOSINT_F64_I128] = i64_i64_func_f64;
    Table[RTLIB::FPTOSINT_F128_I32] = i32_func_i64_i64;
    Table[RTLIB::FPTOSINT_F128_I64] = i64_func_i64_i64;
    Table[RTLIB::FPTOSINT_F128_I128] = i64_i64_func_i64_i64;
    Table[RTLIB::FPTOUINT_F32_I32] = i32_func_f32;
    Table[RTLIB::FPTOUINT_F32_I64] = i64_func_f32;
    Table[RTLIB::FPTOUINT_F32_I128] = i64_i64_func_f32;
    Table[RTLIB::FPTOUINT_F64_I32] = i32_func_f64;
    Table[RTLIB::FPTOUINT_F64_I64] = i64_func_f64;
    Table[RTLIB::FPTOUINT_F64_I128] = i64_i64_func_f64;
    Table[RTLIB::FPTOUINT_F128_I32] = i32_func_i64_i64;
    Table[RTLIB::FPTOUINT_F128_I64] = i64_func_i64_i64;
    Table[RTLIB::FPTOUINT_F128_I128] = i64_i64_func_i64_i64;
    Table[RTLIB::SINTTOFP_I32_F32] = f32_func_i32;
    Table[RTLIB::SINTTOFP_I32_F64] = f64_func_i32;
    Table[RTLIB::SINTTOFP_I32_F128] = i64_i64_func_i32;
    Table[RTLIB::SINTTOFP_I64_F32] = f32_func_i64;
```
- **EN**: Continues the WebAssembly backend backend implementation logic with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的后端实现逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 322-359

```cpp
    Table[RTLIB::SINTTOFP_I64_F64] = f64_func_i64;
    Table[RTLIB::SINTTOFP_I64_F128] = i64_i64_func_i64;
    Table[RTLIB::SINTTOFP_I128_F32] = f32_func_i64_i64;
    Table[RTLIB::SINTTOFP_I128_F64] = f64_func_i64_i64;
    Table[RTLIB::SINTTOFP_I128_F128] = i64_i64_func_i64_i64;
    Table[RTLIB::UINTTOFP_I32_F32] = f32_func_i32;
    Table[RTLIB::UINTTOFP_I32_F64] = f64_func_i64;
    Table[RTLIB::UINTTOFP_I32_F128] = i64_i64_func_i32;
    Table[RTLIB::UINTTOFP_I64_F32] = f32_func_i64;
    Table[RTLIB::UINTTOFP_I64_F64] = f64_func_i64;
    Table[RTLIB::UINTTOFP_I64_F128] = i64_i64_func_i64;
    Table[RTLIB::UINTTOFP_I128_F32] = f32_func_i64_i64;
    Table[RTLIB::UINTTOFP_I128_F64] = f64_func_i64_i64;
    Table[RTLIB::UINTTOFP_I128_F128] = i64_i64_func_i64_i64;

    // Comparison
    // ALl F80 and PPCF128 routines are unsupported.
    Table[RTLIB::OEQ_F32] = i32_func_f32_f32;
    Table[RTLIB::OEQ_F64] = i32_func_f64_f64;
    Table[RTLIB::OEQ_F128] = i32_func_i64_i64_i64_i64;
    Table[RTLIB::UNE_F32] = i32_func_f32_f32;
    Table[RTLIB::UNE_F64] = i32_func_f64_f64;
    Table[RTLIB::UNE_F128] = i32_func_i64_i64_i64_i64;
    Table[RTLIB::OGE_F32] = i32_func_f32_f32;
    Table[RTLIB::OGE_F64] = i32_func_f64_f64;
    Table[RTLIB::OGE_F128] = i32_func_i64_i64_i64_i64;
    Table[RTLIB::OLT_F32] = i32_func_f32_f32;
    Table[RTLIB::OLT_F64] = i32_func_f64_f64;
    Table[RTLIB::OLT_F128] = i32_func_i64_i64_i64_i64;
    Table[RTLIB::OLE_F32] = i32_func_f32_f32;
    Table[RTLIB::OLE_F64] = i32_func_f64_f64;
    Table[RTLIB::OLE_F128] = i32_func_i64_i64_i64_i64;
    Table[RTLIB::OGT_F32] = i32_func_f32_f32;
    Table[RTLIB::OGT_F64] = i32_func_f64_f64;
    Table[RTLIB::OGT_F128] = i32_func_i64_i64_i64_i64;
    Table[RTLIB::UO_F32] = i32_func_f32_f32;
    Table[RTLIB::UO_F64] = i32_func_f64_f64;
    Table[RTLIB::UO_F128] = i32_func_i64_i64_i64_i64;
```
- **EN**: Continues the WebAssembly backend backend implementation logic with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的后端实现逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 360-397

```cpp

    // Memory
    Table[RTLIB::MEMCPY] = iPTR_func_iPTR_iPTR_iPTR;
    Table[RTLIB::MEMSET] = iPTR_func_iPTR_i32_iPTR;
    Table[RTLIB::MEMMOVE] = iPTR_func_iPTR_iPTR_iPTR;

    // __stack_chk_fail
    Table[RTLIB::STACKPROTECTOR_CHECK_FAIL] = func;

    // Return address handling
    Table[RTLIB::RETURN_ADDRESS] = iPTR_func_i32;

    // Element-wise Atomic memory
    // TODO: Fix these when we implement atomic support
    Table[RTLIB::MEMCPY_ELEMENT_UNORDERED_ATOMIC_1] = unsupported;
    Table[RTLIB::MEMCPY_ELEMENT_UNORDERED_ATOMIC_2] = unsupported;
    Table[RTLIB::MEMCPY_ELEMENT_UNORDERED_ATOMIC_4] = unsupported;
    Table[RTLIB::MEMCPY_ELEMENT_UNORDERED_ATOMIC_8] = unsupported;
    Table[RTLIB::MEMCPY_ELEMENT_UNORDERED_ATOMIC_16] = unsupported;
    Table[RTLIB::MEMMOVE_ELEMENT_UNORDERED_ATOMIC_1] = unsupported;
    Table[RTLIB::MEMMOVE_ELEMENT_UNORDERED_ATOMIC_2] = unsupported;
    Table[RTLIB::MEMMOVE_ELEMENT_UNORDERED_ATOMIC_4] = unsupported;
    Table[RTLIB::MEMMOVE_ELEMENT_UNORDERED_ATOMIC_8] = unsupported;
    Table[RTLIB::MEMMOVE_ELEMENT_UNORDERED_ATOMIC_16] = unsupported;

    Table[RTLIB::MEMSET_ELEMENT_UNORDERED_ATOMIC_1] = unsupported;
    Table[RTLIB::MEMSET_ELEMENT_UNORDERED_ATOMIC_2] = unsupported;
    Table[RTLIB::MEMSET_ELEMENT_UNORDERED_ATOMIC_4] = unsupported;
    Table[RTLIB::MEMSET_ELEMENT_UNORDERED_ATOMIC_8] = unsupported;
    Table[RTLIB::MEMSET_ELEMENT_UNORDERED_ATOMIC_16] = unsupported;

    // Atomic '__sync_*' libcalls.
    // TODO: Fix these when we implement atomic support
    Table[RTLIB::SYNC_VAL_COMPARE_AND_SWAP_1] = unsupported;
    Table[RTLIB::SYNC_VAL_COMPARE_AND_SWAP_2] = unsupported;
    Table[RTLIB::SYNC_VAL_COMPARE_AND_SWAP_4] = unsupported;
    Table[RTLIB::SYNC_VAL_COMPARE_AND_SWAP_8] = unsupported;
    Table[RTLIB::SYNC_VAL_COMPARE_AND_SWAP_16] = unsupported;
```
- **EN**: Continues the WebAssembly backend backend implementation logic with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的后端实现逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 398-435

```cpp
    Table[RTLIB::SYNC_LOCK_TEST_AND_SET_1] = unsupported;
    Table[RTLIB::SYNC_LOCK_TEST_AND_SET_2] = unsupported;
    Table[RTLIB::SYNC_LOCK_TEST_AND_SET_4] = unsupported;
    Table[RTLIB::SYNC_LOCK_TEST_AND_SET_8] = unsupported;
    Table[RTLIB::SYNC_LOCK_TEST_AND_SET_16] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_ADD_1] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_ADD_2] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_ADD_4] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_ADD_8] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_ADD_16] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_SUB_1] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_SUB_2] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_SUB_4] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_SUB_8] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_SUB_16] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_AND_1] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_AND_2] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_AND_4] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_AND_8] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_AND_16] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_OR_1] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_OR_2] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_OR_4] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_OR_8] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_OR_16] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_XOR_1] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_XOR_2] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_XOR_4] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_XOR_8] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_XOR_16] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_NAND_1] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_NAND_2] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_NAND_4] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_NAND_8] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_NAND_16] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_MAX_1] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_MAX_2] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_MAX_4] = unsupported;
```
- **EN**: Continues the WebAssembly backend backend implementation logic with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的后端实现逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 436-473

```cpp
    Table[RTLIB::SYNC_FETCH_AND_MAX_8] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_MAX_16] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_UMAX_1] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_UMAX_2] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_UMAX_4] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_UMAX_8] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_UMAX_16] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_MIN_1] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_MIN_2] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_MIN_4] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_MIN_8] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_MIN_16] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_UMIN_1] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_UMIN_2] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_UMIN_4] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_UMIN_8] = unsupported;
    Table[RTLIB::SYNC_FETCH_AND_UMIN_16] = unsupported;

    // Atomic '__atomic_*' libcalls.
    // TODO: Fix these when we implement atomic support
    Table[RTLIB::ATOMIC_LOAD] = unsupported;
    Table[RTLIB::ATOMIC_LOAD_1] = unsupported;
    Table[RTLIB::ATOMIC_LOAD_2] = unsupported;
    Table[RTLIB::ATOMIC_LOAD_4] = unsupported;
    Table[RTLIB::ATOMIC_LOAD_8] = unsupported;
    Table[RTLIB::ATOMIC_LOAD_16] = unsupported;

    Table[RTLIB::ATOMIC_STORE] = unsupported;
    Table[RTLIB::ATOMIC_STORE_1] = unsupported;
    Table[RTLIB::ATOMIC_STORE_2] = unsupported;
    Table[RTLIB::ATOMIC_STORE_4] = unsupported;
    Table[RTLIB::ATOMIC_STORE_8] = unsupported;
    Table[RTLIB::ATOMIC_STORE_16] = unsupported;

    Table[RTLIB::ATOMIC_EXCHANGE] = unsupported;
    Table[RTLIB::ATOMIC_EXCHANGE_1] = unsupported;
    Table[RTLIB::ATOMIC_EXCHANGE_2] = unsupported;
    Table[RTLIB::ATOMIC_EXCHANGE_4] = unsupported;
```
- **EN**: Continues the WebAssembly backend backend implementation logic with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的后端实现逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 474-511

```cpp
    Table[RTLIB::ATOMIC_EXCHANGE_8] = unsupported;
    Table[RTLIB::ATOMIC_EXCHANGE_16] = unsupported;

    Table[RTLIB::ATOMIC_COMPARE_EXCHANGE] = unsupported;
    Table[RTLIB::ATOMIC_COMPARE_EXCHANGE_1] = unsupported;
    Table[RTLIB::ATOMIC_COMPARE_EXCHANGE_2] = unsupported;
    Table[RTLIB::ATOMIC_COMPARE_EXCHANGE_4] = unsupported;
    Table[RTLIB::ATOMIC_COMPARE_EXCHANGE_8] = unsupported;
    Table[RTLIB::ATOMIC_COMPARE_EXCHANGE_16] = unsupported;

    Table[RTLIB::ATOMIC_FETCH_ADD_1] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_ADD_2] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_ADD_4] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_ADD_8] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_ADD_16] = unsupported;

    Table[RTLIB::ATOMIC_FETCH_SUB_1] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_SUB_2] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_SUB_4] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_SUB_8] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_SUB_16] = unsupported;

    Table[RTLIB::ATOMIC_FETCH_AND_1] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_AND_2] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_AND_4] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_AND_8] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_AND_16] = unsupported;

    Table[RTLIB::ATOMIC_FETCH_OR_1] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_OR_2] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_OR_4] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_OR_8] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_OR_16] = unsupported;

    Table[RTLIB::ATOMIC_FETCH_XOR_1] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_XOR_2] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_XOR_4] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_XOR_8] = unsupported;
```
- **EN**: Continues the WebAssembly backend backend implementation logic with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的后端实现逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 512-528

```cpp
    Table[RTLIB::ATOMIC_FETCH_XOR_16] = unsupported;

    Table[RTLIB::ATOMIC_FETCH_NAND_1] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_NAND_2] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_NAND_4] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_NAND_8] = unsupported;
    Table[RTLIB::ATOMIC_FETCH_NAND_16] = unsupported;
  }
};

RuntimeLibcallSignatureTable &getRuntimeLibcallSignatures() {
  static RuntimeLibcallSignatureTable RuntimeLibcallSignatures;
  return RuntimeLibcallSignatures;
}

// Maps libcall names to their RTLIB::Libcall number. Builds the map in a
// constructor for use with a static variable
```
- **EN**: Implements helper routine(s) `getRuntimeLibcallSignatures` for this portion of the WebAssembly backend backend implementation logic.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `getRuntimeLibcallSignatures`。

### Lines 529-566

```cpp
struct StaticLibcallNameMap {
  StringMap<RTLIB::Libcall> Map;
  StaticLibcallNameMap(const Triple &TT) {
    // FIXME: This is broken if there are ever different triples compiled with
    // different libcalls.
    RTLIB::RuntimeLibcallsInfo RTCI(TT);

    ArrayRef<RuntimeLibcallSignature> Table =
        getRuntimeLibcallSignatures().Table;
    for (RTLIB::LibcallImpl Impl : RTLIB::libcall_impls()) {
      if (!RTCI.isAvailable(Impl))
        continue;
      RTLIB::Libcall LC = RTLIB::RuntimeLibcallsInfo::getLibcallFromImpl(Impl);
      if (Table[LC] != unsupported) {
        StringRef NameLibcall =
            RTLIB::RuntimeLibcallsInfo::getLibcallImplName(Impl);
        // FIXME: Map should be to LibcallImpl
        if (!Map.insert({NameLibcall, LC}).second)
          llvm_unreachable("duplicate libcall names in name map");
      }
    }
  }
};

} // end anonymous namespace

void WebAssembly::getLibcallSignature(const WebAssemblySubtarget &Subtarget,
                                      RTLIB::Libcall LC,
                                      SmallVectorImpl<wasm::ValType> &Rets,
                                      SmallVectorImpl<wasm::ValType> &Params) {
  assert(Rets.empty());
  assert(Params.empty());

  wasm::ValType PtrTy =
      Subtarget.hasAddr64() ? wasm::ValType::I64 : wasm::ValType::I32;

  auto &Table = getRuntimeLibcallSignatures().Table;
  switch (Table[LC]) {
```
- **EN**: Declares a backend-facing type `StaticLibcallNameMap`, `RTCI`, `getRuntimeLibcallSignatures` and outlines the API or state that nearby code will rely on. Subtarget feature gating influences the behavior here.
- **CN**: 这里声明面向后端的类型 `StaticLibcallNameMap`, `RTCI`, `getRuntimeLibcallSignatures`，并勾勒出周边代码会依赖的接口或状态。 子目标特性裁剪会影响这里的行为。

### Lines 567-604

```cpp
  case func:
    break;
  case f32_func_f32:
    Rets.push_back(wasm::ValType::F32);
    Params.push_back(wasm::ValType::F32);
    break;
  case f32_func_f64:
    Rets.push_back(wasm::ValType::F32);
    Params.push_back(wasm::ValType::F64);
    break;
  case f32_func_i32:
    Rets.push_back(wasm::ValType::F32);
    Params.push_back(wasm::ValType::I32);
    break;
  case f32_func_i64:
    Rets.push_back(wasm::ValType::F32);
    Params.push_back(wasm::ValType::I64);
    break;
  case f32_func_i16:
    Rets.push_back(wasm::ValType::F32);
    Params.push_back(wasm::ValType::I32);
    break;
  case f64_func_f32:
    Rets.push_back(wasm::ValType::F64);
    Params.push_back(wasm::ValType::F32);
    break;
  case f64_func_f64:
    Rets.push_back(wasm::ValType::F64);
    Params.push_back(wasm::ValType::F64);
    break;
  case f64_func_i32:
    Rets.push_back(wasm::ValType::F64);
    Params.push_back(wasm::ValType::I32);
    break;
  case f64_func_i64:
    Rets.push_back(wasm::ValType::F64);
    Params.push_back(wasm::ValType::I64);
    break;
```
- **EN**: Declares function entry points including `push_back` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `push_back`。

### Lines 605-642

```cpp
  case i32_func_f32:
    Rets.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::F32);
    break;
  case i32_func_f64:
    Rets.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::F64);
    break;
  case i32_func_i32:
    Rets.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::I32);
    break;
  case i64_func_f32:
    Rets.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::F32);
    break;
  case i64_func_f64:
    Rets.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::F64);
    break;
  case i64_func_i64:
    Rets.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    break;
  case f32_func_f32_f32:
    Rets.push_back(wasm::ValType::F32);
    Params.push_back(wasm::ValType::F32);
    Params.push_back(wasm::ValType::F32);
    break;
  case f32_func_f32_i32:
    Rets.push_back(wasm::ValType::F32);
    Params.push_back(wasm::ValType::F32);
    Params.push_back(wasm::ValType::I32);
    break;
  case f32_func_f32_iPTR:
    Rets.push_back(wasm::ValType::F32);
    Params.push_back(wasm::ValType::F32);
    Params.push_back(PtrTy);
```
- **EN**: Declares function entry points including `push_back` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `push_back`。

### Lines 643-680

```cpp
    break;
  case f32_func_i64_i64:
    Rets.push_back(wasm::ValType::F32);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    break;
  case f64_func_f64_f64:
    Rets.push_back(wasm::ValType::F64);
    Params.push_back(wasm::ValType::F64);
    Params.push_back(wasm::ValType::F64);
    break;
  case f64_func_f64_i32:
    Rets.push_back(wasm::ValType::F64);
    Params.push_back(wasm::ValType::F64);
    Params.push_back(wasm::ValType::I32);
    break;
  case f64_func_i64_i64:
    Rets.push_back(wasm::ValType::F64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    break;
  case f64_func_f64_iPTR:
    Rets.push_back(wasm::ValType::F64);
    Params.push_back(wasm::ValType::F64);
    Params.push_back(PtrTy);
    break;
  case i16_func_f32:
    Rets.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::F32);
    break;
  case i16_func_f64:
    Rets.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::F64);
    break;
  case i16_func_i64_i64:
    Rets.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
```
- **EN**: Declares function entry points including `push_back` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `push_back`。

### Lines 681-718

```cpp
    break;
  case i8_func_i8_i8:
    Rets.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::I32);
    break;
  case func_f32_iPTR_iPTR:
    Params.push_back(wasm::ValType::F32);
    Params.push_back(PtrTy);
    Params.push_back(PtrTy);
    break;
  case func_f64_iPTR_iPTR:
    Params.push_back(wasm::ValType::F64);
    Params.push_back(PtrTy);
    Params.push_back(PtrTy);
    break;
  case i16_func_i16_i16:
    Rets.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::I32);
    break;
  case i32_func_f32_f32:
    Rets.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::F32);
    Params.push_back(wasm::ValType::F32);
    break;
  case i32_func_f64_f64:
    Rets.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::F64);
    Params.push_back(wasm::ValType::F64);
    break;
  case i32_func_i32_i32:
    Rets.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::I32);
    break;
  case i32_func_i32_i32_iPTR:
    Rets.push_back(wasm::ValType::I32);
```
- **EN**: Declares function entry points including `push_back` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `push_back`。

### Lines 719-756

```cpp
    Params.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::I32);
    Params.push_back(PtrTy);
    break;
  case i64_func_i64_i64:
    Rets.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    break;
  case i64_func_i64_i64_iPTR:
    Rets.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(PtrTy);
    break;
  case i64_i64_func_f32:
    if (WebAssembly::canLowerMultivalueReturn(&Subtarget)) {
      Rets.push_back(wasm::ValType::I64);
      Rets.push_back(wasm::ValType::I64);
    } else {
      Params.push_back(PtrTy);
    }
    Params.push_back(wasm::ValType::F32);
    break;
  case i64_i64_func_f64:
    if (WebAssembly::canLowerMultivalueReturn(&Subtarget)) {
      Rets.push_back(wasm::ValType::I64);
      Rets.push_back(wasm::ValType::I64);
    } else {
      Params.push_back(PtrTy);
    }
    Params.push_back(wasm::ValType::F64);
    break;
  case i16_i16_func_i16_i16:
    if (WebAssembly::canLowerMultivalueReturn(&Subtarget)) {
      Rets.push_back(wasm::ValType::I32);
      Rets.push_back(wasm::ValType::I32);
    } else {
```
- **EN**: Implements helper routine(s) `push_back`, `canLowerMultivalueReturn` for this portion of the WebAssembly backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `push_back`, `canLowerMultivalueReturn`。 子目标特性裁剪会影响这里的行为。

### Lines 757-794

```cpp
      Params.push_back(PtrTy);
    }
    Params.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::I32);
    break;
  case i32_i32_func_i32_i32:
    if (WebAssembly::canLowerMultivalueReturn(&Subtarget)) {
      Rets.push_back(wasm::ValType::I32);
      Rets.push_back(wasm::ValType::I32);
    } else {
      Params.push_back(PtrTy);
    }
    Params.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::I32);
    break;
  case i64_i64_func_i64_i64:
    if (WebAssembly::canLowerMultivalueReturn(&Subtarget)) {
      Rets.push_back(wasm::ValType::I64);
      Rets.push_back(wasm::ValType::I64);
    } else {
      Params.push_back(PtrTy);
    }
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    break;
  case i64_i64_func_i64_i64_iPTR:
    if (WebAssembly::canLowerMultivalueReturn(&Subtarget)) {
      Rets.push_back(wasm::ValType::I64);
      Rets.push_back(wasm::ValType::I64);
    } else {
      Params.push_back(PtrTy);
    }
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(PtrTy);
    break;
  case i64_i64_func_i64_i64_i64_i64:
    if (WebAssembly::canLowerMultivalueReturn(&Subtarget)) {
```
- **EN**: Implements helper routine(s) `push_back`, `canLowerMultivalueReturn` for this portion of the WebAssembly backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `push_back`, `canLowerMultivalueReturn`。 子目标特性裁剪会影响这里的行为。

### Lines 795-832

```cpp
      Rets.push_back(wasm::ValType::I64);
      Rets.push_back(wasm::ValType::I64);
    } else {
      Params.push_back(PtrTy);
    }
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    break;
  case i64_i64_func_i64_i64_i64_i64_iPTR:
    if (WebAssembly::canLowerMultivalueReturn(&Subtarget)) {
      Rets.push_back(wasm::ValType::I64);
      Rets.push_back(wasm::ValType::I64);
    } else {
      Params.push_back(PtrTy);
    }
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(PtrTy);
    break;
  case i64_i64_i64_i64_func_i64_i64_i64_i64:
    if (WebAssembly::canLowerMultivalueReturn(&Subtarget)) {
      Rets.push_back(wasm::ValType::I64);
      Rets.push_back(wasm::ValType::I64);
      Rets.push_back(wasm::ValType::I64);
      Rets.push_back(wasm::ValType::I64);
    } else {
      Params.push_back(PtrTy);
    }
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    break;
  case i64_i64_func_i64_i64_i32:
```
- **EN**: Implements helper routine(s) `push_back`, `canLowerMultivalueReturn` for this portion of the WebAssembly backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `push_back`, `canLowerMultivalueReturn`。 子目标特性裁剪会影响这里的行为。

### Lines 833-870

```cpp
    if (WebAssembly::canLowerMultivalueReturn(&Subtarget)) {
      Rets.push_back(wasm::ValType::I64);
      Rets.push_back(wasm::ValType::I64);
    } else {
      Params.push_back(PtrTy);
    }
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I32);
    break;
  case iPTR_func_i32:
    Rets.push_back(PtrTy);
    Params.push_back(wasm::ValType::I32);
    break;
  case iPTR_func_iPTR_i32_iPTR:
    Rets.push_back(PtrTy);
    Params.push_back(PtrTy);
    Params.push_back(wasm::ValType::I32);
    Params.push_back(PtrTy);
    break;
  case iPTR_func_iPTR_iPTR_iPTR:
    Rets.push_back(PtrTy);
    Params.push_back(PtrTy);
    Params.push_back(PtrTy);
    Params.push_back(PtrTy);
    break;
  case f32_func_f32_f32_f32:
    Rets.push_back(wasm::ValType::F32);
    Params.push_back(wasm::ValType::F32);
    Params.push_back(wasm::ValType::F32);
    Params.push_back(wasm::ValType::F32);
    break;
  case f64_func_f64_f64_f64:
    Rets.push_back(wasm::ValType::F64);
    Params.push_back(wasm::ValType::F64);
    Params.push_back(wasm::ValType::F64);
    Params.push_back(wasm::ValType::F64);
    break;
```
- **EN**: Implements helper routine(s) `canLowerMultivalueReturn`, `push_back` for this portion of the WebAssembly backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `canLowerMultivalueReturn`, `push_back`。 子目标特性裁剪会影响这里的行为。

### Lines 871-908

```cpp
  case func_i64_i64_iPTR_iPTR:
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(PtrTy);
    Params.push_back(PtrTy);
    break;
  case i32_func_i64_i64:
    Rets.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    break;
  case i32_func_i64_i64_i64_i64:
    Rets.push_back(wasm::ValType::I32);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    break;
  case iPTR_func_f32:
    Rets.push_back(PtrTy);
    Params.push_back(wasm::ValType::F32);
    break;
  case iPTR_func_f64:
    Rets.push_back(PtrTy);
    Params.push_back(wasm::ValType::F64);
    break;
  case iPTR_func_i64_i64:
    Rets.push_back(PtrTy);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    break;
  case i64_i64_func_i64_i64_i64_i64_i64_i64:
    if (WebAssembly::canLowerMultivalueReturn(&Subtarget)) {
      Rets.push_back(wasm::ValType::I64);
      Rets.push_back(wasm::ValType::I64);
    } else {
      Params.push_back(PtrTy);
    }
```
- **EN**: Implements helper routine(s) `push_back`, `canLowerMultivalueReturn` for this portion of the WebAssembly backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `push_back`, `canLowerMultivalueReturn`。 子目标特性裁剪会影响这里的行为。

### Lines 909-946

```cpp
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    Params.push_back(wasm::ValType::I64);
    break;
  case i64_i64_func_i32:
    if (WebAssembly::canLowerMultivalueReturn(&Subtarget)) {
      Rets.push_back(wasm::ValType::I64);
      Rets.push_back(wasm::ValType::I64);
    } else {
      Params.push_back(PtrTy);
    }
    Params.push_back(wasm::ValType::I32);
    break;
  case i64_i64_func_i64:
    if (WebAssembly::canLowerMultivalueReturn(&Subtarget)) {
      Rets.push_back(wasm::ValType::I64);
      Rets.push_back(wasm::ValType::I64);
    } else {
      Params.push_back(PtrTy);
    }
    Params.push_back(wasm::ValType::I64);
    break;
  case unsupported:
    llvm_unreachable("unsupported runtime library signature");
  }
}

// TODO: If the RTLIB::Libcall-taking flavor of GetSignature remains unused
// other than here, just roll its logic into this version.
void WebAssembly::getLibcallSignature(const WebAssemblySubtarget &Subtarget,
                                      StringRef Name,
                                      SmallVectorImpl<wasm::ValType> &Rets,
                                      SmallVectorImpl<wasm::ValType> &Params) {
  static StaticLibcallNameMap LibcallNameMap(Subtarget.getTargetTriple());
  auto &Map = LibcallNameMap.Map;
```
- **EN**: Implements helper routine(s) `push_back`, `canLowerMultivalueReturn`, `llvm_unreachable` for this portion of the WebAssembly backend backend implementation logic. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `push_back`, `canLowerMultivalueReturn`, `llvm_unreachable`。 子目标特性裁剪会影响这里的行为。

### Lines 947-953

```cpp
  auto Val = Map.find(Name);
#ifndef NDEBUG
  if (Val == Map.end()) {
    auto Message =
        std::string("unexpected runtime library name: ") + std::string(Name);
    llvm_unreachable(Message.c_str());
  }
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `find`, `end`, `string`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `find`, `end`, `string`。

### Lines 954-956

```cpp
#endif
  return getLibcallSignature(Subtarget, Val->second, Rets, Params);
}
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Subtarget feature gating influences the behavior here.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 子目标特性裁剪会影响这里的行为。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Subtarget features / 子目标特性
- Declarative TableGen records / 声明式 TableGen 记录
- Atomic operation support / 原子操作支持
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyRuntimeLibcallSignatures.h`
- `WebAssemblySubtarget.h`
- `WebAssemblyUtilities.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
