# AArch64CodeGenUtils.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/AArch64CodeGenUtils.h`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: AArch64CodeGenUtils.h *- C++.
- **Purpose (CN)**: 声明与 `AArch64CodeGenUtils` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 667

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

````cpp
//===--- AArch64CodeGenUtils.h ----------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// Utilities used for generating code for AArch64 that are shared between the
/// classic and ClangIR code-gen.
///
//===----------------------------------------------------------------------===//

#ifndef LLVM_CLANG_BASIC_AARCH64CODEGENUTILS_H
#define LLVM_CLANG_BASIC_AARCH64CODEGENUTILS_H

#include "clang/Basic/TargetBuiltins.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/IR/IntrinsicsAArch64.h"
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
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `file`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`file`。
- **L10 EN**: Comment explains nearby logic, constraints, or intent: `Utilities used for generating code for AArch64 that are shared between the`.
  **L10 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Utilities used for generating code for AArch64 that are shared between the`。
- **L11 EN**: Comment explains nearby logic, constraints, or intent: `classic and ClangIR code-gen.`.
  **L11 CN**: 注释解释附近代码的逻辑、约束或设计意图：`classic and ClangIR code-gen.`。
- **L12 EN**: Separator comment used for visual grouping.
  **L12 CN**: 用于视觉分组的分隔注释。
- **L13 EN**: Banner comment marking a file or section boundary.
  **L13 CN**: 横幅注释，用于标记文件或章节边界。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L15 EN**: Starts a preprocessor conditional block: `#ifndef LLVM_CLANG_BASIC_AARCH64CODEGENUTILS_H`.
  **L15 CN**: 开始一个预处理条件块：`#ifndef LLVM_CLANG_BASIC_AARCH64CODEGENUTILS_H`。
- **L16 EN**: Defines macro `LLVM_CLANG_BASIC_AARCH64CODEGENUTILS_H` for conditional compilation, shorthand, or table-driven expansion.
  **L16 CN**: 定义宏 `LLVM_CLANG_BASIC_AARCH64CODEGENUTILS_H`，用于条件编译、简写或表驱动展开。
- **L17 EN**: Blank line separating nearby declarations or logic blocks.
  **L17 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L18 EN**: Includes "clang/Basic/TargetBuiltins.h" to access Clang basic-layer enums, diagnostics, target data, and utilities.
  **L18 CN**: 引入 "clang/Basic/TargetBuiltins.h" 以使用Clang Basic 层枚举、诊断、目标数据与工具。
- **L19 EN**: Includes "llvm/IR/Intrinsics.h" to access LLVM IR core abstractions.
  **L19 CN**: 引入 "llvm/IR/Intrinsics.h" 以使用LLVM IR 核心抽象。
- **L20 EN**: Includes "llvm/IR/IntrinsicsAArch64.h" to access LLVM IR core abstractions.
  **L20 CN**: 引入 "llvm/IR/IntrinsicsAArch64.h" 以使用LLVM IR 核心抽象。

### Lines 21-40

````cpp

namespace clang {
namespace aarch64 {

//===----------------------------------------------------------------------===//
//  Intrinsics maps
//
//  Maps that help automate code-generation.
//===----------------------------------------------------------------------===//
enum {
  AddRetType = (1 << 0),
  Add1ArgType = (1 << 1),
  Add2ArgTypes = (1 << 2),

  VectorizeRetType = (1 << 3),
  VectorizeArgTypes = (1 << 4),

  InventFloatType = (1 << 5),
  UnsignedAlts = (1 << 6),

````
- **L21 EN**: Blank line separating nearby declarations or logic blocks.
  **L21 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L22 EN**: Opens namespace scope `clang`.
  **L22 CN**: 打开命名空间作用域 `clang`。
- **L23 EN**: Opens namespace scope `aarch64`.
  **L23 CN**: 打开命名空间作用域 `aarch64`。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L25 EN**: Banner comment marking a file or section boundary.
  **L25 CN**: 横幅注释，用于标记文件或章节边界。
- **L26 EN**: Comment explains nearby logic, constraints, or intent: `Intrinsics maps`.
  **L26 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intrinsics maps`。
- **L27 EN**: Separator comment used for visual grouping.
  **L27 CN**: 用于视觉分组的分隔注释。
- **L28 EN**: Comment explains nearby logic, constraints, or intent: `Maps that help automate code-generation.`.
  **L28 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Maps that help automate code-generation.`。
- **L29 EN**: Banner comment marking a file or section boundary.
  **L29 CN**: 横幅注释，用于标记文件或章节边界。
- **L30 EN**: Declares enum `enum`.
  **L30 CN**: 声明 enum `enum`。
- **L31 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AddRetType = (1 << 0),`.
  **L31 CN**: 继续一个多行参数列表、初始化器或聚合项：`AddRetType = (1 << 0),`。
- **L32 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Add1ArgType = (1 << 1),`.
  **L32 CN**: 继续一个多行参数列表、初始化器或聚合项：`Add1ArgType = (1 << 1),`。
- **L33 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Add2ArgTypes = (1 << 2),`.
  **L33 CN**: 继续一个多行参数列表、初始化器或聚合项：`Add2ArgTypes = (1 << 2),`。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L35 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VectorizeRetType = (1 << 3),`.
  **L35 CN**: 继续一个多行参数列表、初始化器或聚合项：`VectorizeRetType = (1 << 3),`。
- **L36 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VectorizeArgTypes = (1 << 4),`.
  **L36 CN**: 继续一个多行参数列表、初始化器或聚合项：`VectorizeArgTypes = (1 << 4),`。
- **L37 EN**: Blank line separating nearby declarations or logic blocks.
  **L37 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L38 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `InventFloatType = (1 << 5),`.
  **L38 CN**: 继续一个多行参数列表、初始化器或聚合项：`InventFloatType = (1 << 5),`。
- **L39 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnsignedAlts = (1 << 6),`.
  **L39 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnsignedAlts = (1 << 6),`。
- **L40 EN**: Blank line separating nearby declarations or logic blocks.
  **L40 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 41-60

````cpp
  Use64BitVectors = (1 << 7),
  Use128BitVectors = (1 << 8),

  Vectorize1ArgType = Add1ArgType | VectorizeArgTypes,
  VectorRet = AddRetType | VectorizeRetType,
  VectorRetGetArgs01 =
      AddRetType | Add2ArgTypes | VectorizeRetType | VectorizeArgTypes,
  FpCmpzModifiers =
      AddRetType | VectorizeRetType | Add1ArgType | InventFloatType
};

struct ARMVectorIntrinsicInfo {
  const char *NameHint;
  unsigned BuiltinID;
  unsigned LLVMIntrinsic;
  unsigned AltLLVMIntrinsic;
  uint64_t TypeModifier;

  bool operator<(unsigned RHSBuiltinID) const {
    return BuiltinID < RHSBuiltinID;
````
- **L41 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Use64BitVectors = (1 << 7),`.
  **L41 CN**: 继续一个多行参数列表、初始化器或聚合项：`Use64BitVectors = (1 << 7),`。
- **L42 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Use128BitVectors = (1 << 8),`.
  **L42 CN**: 继续一个多行参数列表、初始化器或聚合项：`Use128BitVectors = (1 << 8),`。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L44 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vectorize1ArgType = Add1ArgType | VectorizeArgTypes,`.
  **L44 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vectorize1ArgType = Add1ArgType | VectorizeArgTypes,`。
- **L45 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VectorRet = AddRetType | VectorizeRetType,`.
  **L45 CN**: 继续一个多行参数列表、初始化器或聚合项：`VectorRet = AddRetType | VectorizeRetType,`。
- **L46 EN**: Continues the surrounding expression or declaration: `VectorRetGetArgs01 =`.
  **L46 CN**: 继续构造周围的表达式或声明：`VectorRetGetArgs01 =`。
- **L47 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `AddRetType | Add2ArgTypes | VectorizeRetType | VectorizeArgTypes,`.
  **L47 CN**: 继续一个多行参数列表、初始化器或聚合项：`AddRetType | Add2ArgTypes | VectorizeRetType | VectorizeArgTypes,`。
- **L48 EN**: Continues the surrounding expression or declaration: `FpCmpzModifiers =`.
  **L48 CN**: 继续构造周围的表达式或声明：`FpCmpzModifiers =`。
- **L49 EN**: Continues the surrounding expression or declaration: `AddRetType | VectorizeRetType | Add1ArgType | InventFloatType`.
  **L49 CN**: 继续构造周围的表达式或声明：`AddRetType | VectorizeRetType | Add1ArgType | InventFloatType`。
- **L50 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L50 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L52 EN**: Declares struct `ARMVectorIntrinsicInfo`.
  **L52 CN**: 声明 struct `ARMVectorIntrinsicInfo`。
- **L53 EN**: Adds a standalone statement or declaration: `const char *NameHint;`.
  **L53 CN**: 添加一条独立语句或声明：`const char *NameHint;`。
- **L54 EN**: Adds a standalone statement or declaration: `unsigned BuiltinID;`.
  **L54 CN**: 添加一条独立语句或声明：`unsigned BuiltinID;`。
- **L55 EN**: Adds a standalone statement or declaration: `unsigned LLVMIntrinsic;`.
  **L55 CN**: 添加一条独立语句或声明：`unsigned LLVMIntrinsic;`。
- **L56 EN**: Adds a standalone statement or declaration: `unsigned AltLLVMIntrinsic;`.
  **L56 CN**: 添加一条独立语句或声明：`unsigned AltLLVMIntrinsic;`。
- **L57 EN**: Adds a standalone statement or declaration: `uint64_t TypeModifier;`.
  **L57 CN**: 添加一条独立语句或声明：`uint64_t TypeModifier;`。
- **L58 EN**: Blank line separating nearby declarations or logic blocks.
  **L58 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L59 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `bool operator<(unsigned RHSBuiltinID) const {`.
  **L59 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`bool operator<(unsigned RHSBuiltinID) const {`。
- **L60 EN**: Returns from the current function with `BuiltinID < RHSBuiltinID`.
  **L60 CN**: 以 `BuiltinID < RHSBuiltinID` 从当前函数返回。

### Lines 61-80

````cpp
  }
  bool operator<(const ARMVectorIntrinsicInfo &TE) const {
    return BuiltinID < TE.BuiltinID;
  }
};

#define NEONMAP0(NameBase)                                                     \
  {#NameBase, NEON::BI__builtin_neon_##NameBase, 0, 0, 0}

#define NEONMAP1(NameBase, LLVMIntrinsic, TypeModifier)                        \
  {#NameBase, NEON::BI__builtin_neon_##NameBase,                               \
   llvm::Intrinsic::LLVMIntrinsic, 0, TypeModifier}

#define NEONMAP2(NameBase, LLVMIntrinsic, AltLLVMIntrinsic, TypeModifier)      \
  {#NameBase, NEON::BI__builtin_neon_##NameBase,                               \
   llvm::Intrinsic::LLVMIntrinsic, llvm::Intrinsic::AltLLVMIntrinsic,          \
   TypeModifier}

// clang-format off
const inline ARMVectorIntrinsicInfo AArch64SIMDIntrinsicMap [] = {
````
- **L61 EN**: Closes the current lexical scope or compound statement.
  **L61 CN**: 结束当前词法作用域或复合语句块。
- **L62 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `bool operator<(const ARMVectorIntrinsicInfo &TE) const {`.
  **L62 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`bool operator<(const ARMVectorIntrinsicInfo &TE) const {`。
- **L63 EN**: Returns from the current function with `BuiltinID < TE.BuiltinID`.
  **L63 CN**: 以 `BuiltinID < TE.BuiltinID` 从当前函数返回。
- **L64 EN**: Closes the current lexical scope or compound statement.
  **L64 CN**: 结束当前词法作用域或复合语句块。
- **L65 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L65 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L66 EN**: Blank line separating nearby declarations or logic blocks.
  **L66 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L67 EN**: Defines macro `NEONMAP0(NameBase)` for conditional compilation, shorthand, or table-driven expansion.
  **L67 CN**: 定义宏 `NEONMAP0(NameBase)`，用于条件编译、简写或表驱动展开。
- **L68 EN**: Continues the surrounding expression or declaration: `{#NameBase, NEON::BI__builtin_neon_##NameBase, 0, 0, 0}`.
  **L68 CN**: 继续构造周围的表达式或声明：`{#NameBase, NEON::BI__builtin_neon_##NameBase, 0, 0, 0}`。
- **L69 EN**: Blank line separating nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L70 EN**: Defines macro `NEONMAP1(NameBase,` for conditional compilation, shorthand, or table-driven expansion.
  **L70 CN**: 定义宏 `NEONMAP1(NameBase,`，用于条件编译、简写或表驱动展开。
- **L71 EN**: Continues the surrounding expression or declaration: `{#NameBase, NEON::BI__builtin_neon_##NameBase,                               \`.
  **L71 CN**: 继续构造周围的表达式或声明：`{#NameBase, NEON::BI__builtin_neon_##NameBase,                               \`。
- **L72 EN**: Continues the surrounding expression or declaration: `llvm::Intrinsic::LLVMIntrinsic, 0, TypeModifier}`.
  **L72 CN**: 继续构造周围的表达式或声明：`llvm::Intrinsic::LLVMIntrinsic, 0, TypeModifier}`。
- **L73 EN**: Blank line separating nearby declarations or logic blocks.
  **L73 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L74 EN**: Defines macro `NEONMAP2(NameBase,` for conditional compilation, shorthand, or table-driven expansion.
  **L74 CN**: 定义宏 `NEONMAP2(NameBase,`，用于条件编译、简写或表驱动展开。
- **L75 EN**: Continues the surrounding expression or declaration: `{#NameBase, NEON::BI__builtin_neon_##NameBase,                               \`.
  **L75 CN**: 继续构造周围的表达式或声明：`{#NameBase, NEON::BI__builtin_neon_##NameBase,                               \`。
- **L76 EN**: Continues the surrounding expression or declaration: `llvm::Intrinsic::LLVMIntrinsic, llvm::Intrinsic::AltLLVMIntrinsic,          \`.
  **L76 CN**: 继续构造周围的表达式或声明：`llvm::Intrinsic::LLVMIntrinsic, llvm::Intrinsic::AltLLVMIntrinsic,          \`。
- **L77 EN**: Continues the surrounding expression or declaration: `TypeModifier}`.
  **L77 CN**: 继续构造周围的表达式或声明：`TypeModifier}`。
- **L78 EN**: Blank line separating nearby declarations or logic blocks.
  **L78 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L79 EN**: Comment explains nearby logic, constraints, or intent: `clang-format off`.
  **L79 CN**: 注释解释附近代码的逻辑、约束或设计意图：`clang-format off`。
- **L80 EN**: Continues the surrounding expression or declaration: `const inline ARMVectorIntrinsicInfo AArch64SIMDIntrinsicMap [] = {`.
  **L80 CN**: 继续构造周围的表达式或声明：`const inline ARMVectorIntrinsicInfo AArch64SIMDIntrinsicMap [] = {`。

### Lines 81-100

````cpp
  NEONMAP0(splat_lane_v),
  NEONMAP0(splat_laneq_v),
  NEONMAP0(splatq_lane_v),
  NEONMAP0(splatq_laneq_v),
  NEONMAP1(vabs_v, aarch64_neon_abs, 0),
  NEONMAP1(vabsq_v, aarch64_neon_abs, 0),
  NEONMAP0(vadd_v),
  NEONMAP0(vaddhn_v),
  NEONMAP0(vaddq_v),
  NEONMAP1(vaesdq_u8, aarch64_crypto_aesd, 0),
  NEONMAP1(vaeseq_u8, aarch64_crypto_aese, 0),
  NEONMAP1(vaesimcq_u8, aarch64_crypto_aesimc, 0),
  NEONMAP1(vaesmcq_u8, aarch64_crypto_aesmc, 0),
  NEONMAP2(vbcaxq_s16, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),
  NEONMAP2(vbcaxq_s32, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),
  NEONMAP2(vbcaxq_s64, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),
  NEONMAP2(vbcaxq_s8, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),
  NEONMAP2(vbcaxq_u16, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),
  NEONMAP2(vbcaxq_u32, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),
  NEONMAP2(vbcaxq_u64, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),
````
- **L81 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(splat_lane_v),`.
  **L81 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(splat_lane_v),`。
- **L82 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(splat_laneq_v),`.
  **L82 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(splat_laneq_v),`。
- **L83 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(splatq_lane_v),`.
  **L83 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(splatq_lane_v),`。
- **L84 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(splatq_laneq_v),`.
  **L84 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(splatq_laneq_v),`。
- **L85 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vabs_v, aarch64_neon_abs, 0),`.
  **L85 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vabs_v, aarch64_neon_abs, 0),`。
- **L86 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vabsq_v, aarch64_neon_abs, 0),`.
  **L86 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vabsq_v, aarch64_neon_abs, 0),`。
- **L87 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vadd_v),`.
  **L87 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vadd_v),`。
- **L88 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vaddhn_v),`.
  **L88 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vaddhn_v),`。
- **L89 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vaddq_v),`.
  **L89 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vaddq_v),`。
- **L90 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaesdq_u8, aarch64_crypto_aesd, 0),`.
  **L90 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaesdq_u8, aarch64_crypto_aesd, 0),`。
- **L91 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaeseq_u8, aarch64_crypto_aese, 0),`.
  **L91 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaeseq_u8, aarch64_crypto_aese, 0),`。
- **L92 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaesimcq_u8, aarch64_crypto_aesimc, 0),`.
  **L92 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaesimcq_u8, aarch64_crypto_aesimc, 0),`。
- **L93 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaesmcq_u8, aarch64_crypto_aesmc, 0),`.
  **L93 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaesmcq_u8, aarch64_crypto_aesmc, 0),`。
- **L94 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vbcaxq_s16, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),`.
  **L94 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vbcaxq_s16, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),`。
- **L95 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vbcaxq_s32, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),`.
  **L95 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vbcaxq_s32, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),`。
- **L96 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vbcaxq_s64, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),`.
  **L96 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vbcaxq_s64, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),`。
- **L97 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vbcaxq_s8, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),`.
  **L97 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vbcaxq_s8, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),`。
- **L98 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vbcaxq_u16, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),`.
  **L98 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vbcaxq_u16, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),`。
- **L99 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vbcaxq_u32, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),`.
  **L99 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vbcaxq_u32, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),`。
- **L100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vbcaxq_u64, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),`.
  **L100 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vbcaxq_u64, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),`。

### Lines 101-120

````cpp
  NEONMAP2(vbcaxq_u8, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),
  NEONMAP1(vbfdot_f32, aarch64_neon_bfdot, 0),
  NEONMAP1(vbfdotq_f32, aarch64_neon_bfdot, 0),
  NEONMAP1(vbfmlalbq_f32, aarch64_neon_bfmlalb, 0),
  NEONMAP1(vbfmlaltq_f32, aarch64_neon_bfmlalt, 0),
  NEONMAP1(vbfmmlaq_f32, aarch64_neon_bfmmla, 0),
  NEONMAP1(vcadd_rot270_f16, aarch64_neon_vcadd_rot270, Add1ArgType),
  NEONMAP1(vcadd_rot270_f32, aarch64_neon_vcadd_rot270, Add1ArgType),
  NEONMAP1(vcadd_rot90_f16, aarch64_neon_vcadd_rot90, Add1ArgType),
  NEONMAP1(vcadd_rot90_f32, aarch64_neon_vcadd_rot90, Add1ArgType),
  NEONMAP1(vcaddq_rot270_f16, aarch64_neon_vcadd_rot270, Add1ArgType),
  NEONMAP1(vcaddq_rot270_f32, aarch64_neon_vcadd_rot270, Add1ArgType),
  NEONMAP1(vcaddq_rot270_f64, aarch64_neon_vcadd_rot270, Add1ArgType),
  NEONMAP1(vcaddq_rot90_f16, aarch64_neon_vcadd_rot90, Add1ArgType),
  NEONMAP1(vcaddq_rot90_f32, aarch64_neon_vcadd_rot90, Add1ArgType),
  NEONMAP1(vcaddq_rot90_f64, aarch64_neon_vcadd_rot90, Add1ArgType),
  NEONMAP1(vcage_v, aarch64_neon_facge, 0),
  NEONMAP1(vcageq_v, aarch64_neon_facge, 0),
  NEONMAP1(vcagt_v, aarch64_neon_facgt, 0),
  NEONMAP1(vcagtq_v, aarch64_neon_facgt, 0),
````
- **L101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vbcaxq_u8, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),`.
  **L101 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vbcaxq_u8, aarch64_crypto_bcaxu, aarch64_crypto_bcaxs, Add1ArgType | UnsignedAlts),`。
- **L102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vbfdot_f32, aarch64_neon_bfdot, 0),`.
  **L102 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vbfdot_f32, aarch64_neon_bfdot, 0),`。
- **L103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vbfdotq_f32, aarch64_neon_bfdot, 0),`.
  **L103 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vbfdotq_f32, aarch64_neon_bfdot, 0),`。
- **L104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vbfmlalbq_f32, aarch64_neon_bfmlalb, 0),`.
  **L104 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vbfmlalbq_f32, aarch64_neon_bfmlalb, 0),`。
- **L105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vbfmlaltq_f32, aarch64_neon_bfmlalt, 0),`.
  **L105 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vbfmlaltq_f32, aarch64_neon_bfmlalt, 0),`。
- **L106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vbfmmlaq_f32, aarch64_neon_bfmmla, 0),`.
  **L106 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vbfmmlaq_f32, aarch64_neon_bfmmla, 0),`。
- **L107 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcadd_rot270_f16, aarch64_neon_vcadd_rot270, Add1ArgType),`.
  **L107 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcadd_rot270_f16, aarch64_neon_vcadd_rot270, Add1ArgType),`。
- **L108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcadd_rot270_f32, aarch64_neon_vcadd_rot270, Add1ArgType),`.
  **L108 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcadd_rot270_f32, aarch64_neon_vcadd_rot270, Add1ArgType),`。
- **L109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcadd_rot90_f16, aarch64_neon_vcadd_rot90, Add1ArgType),`.
  **L109 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcadd_rot90_f16, aarch64_neon_vcadd_rot90, Add1ArgType),`。
- **L110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcadd_rot90_f32, aarch64_neon_vcadd_rot90, Add1ArgType),`.
  **L110 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcadd_rot90_f32, aarch64_neon_vcadd_rot90, Add1ArgType),`。
- **L111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcaddq_rot270_f16, aarch64_neon_vcadd_rot270, Add1ArgType),`.
  **L111 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcaddq_rot270_f16, aarch64_neon_vcadd_rot270, Add1ArgType),`。
- **L112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcaddq_rot270_f32, aarch64_neon_vcadd_rot270, Add1ArgType),`.
  **L112 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcaddq_rot270_f32, aarch64_neon_vcadd_rot270, Add1ArgType),`。
- **L113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcaddq_rot270_f64, aarch64_neon_vcadd_rot270, Add1ArgType),`.
  **L113 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcaddq_rot270_f64, aarch64_neon_vcadd_rot270, Add1ArgType),`。
- **L114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcaddq_rot90_f16, aarch64_neon_vcadd_rot90, Add1ArgType),`.
  **L114 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcaddq_rot90_f16, aarch64_neon_vcadd_rot90, Add1ArgType),`。
- **L115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcaddq_rot90_f32, aarch64_neon_vcadd_rot90, Add1ArgType),`.
  **L115 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcaddq_rot90_f32, aarch64_neon_vcadd_rot90, Add1ArgType),`。
- **L116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcaddq_rot90_f64, aarch64_neon_vcadd_rot90, Add1ArgType),`.
  **L116 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcaddq_rot90_f64, aarch64_neon_vcadd_rot90, Add1ArgType),`。
- **L117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcage_v, aarch64_neon_facge, 0),`.
  **L117 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcage_v, aarch64_neon_facge, 0),`。
- **L118 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcageq_v, aarch64_neon_facge, 0),`.
  **L118 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcageq_v, aarch64_neon_facge, 0),`。
- **L119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcagt_v, aarch64_neon_facgt, 0),`.
  **L119 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcagt_v, aarch64_neon_facgt, 0),`。
- **L120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcagtq_v, aarch64_neon_facgt, 0),`.
  **L120 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcagtq_v, aarch64_neon_facgt, 0),`。

### Lines 121-140

````cpp
  NEONMAP1(vcale_v, aarch64_neon_facge, 0),
  NEONMAP1(vcaleq_v, aarch64_neon_facge, 0),
  NEONMAP1(vcalt_v, aarch64_neon_facgt, 0),
  NEONMAP1(vcaltq_v, aarch64_neon_facgt, 0),
  NEONMAP0(vceqz_v),
  NEONMAP0(vceqzq_v),
  NEONMAP0(vcgez_v),
  NEONMAP0(vcgezq_v),
  NEONMAP0(vcgtz_v),
  NEONMAP0(vcgtzq_v),
  NEONMAP0(vclez_v),
  NEONMAP0(vclezq_v),
  NEONMAP1(vcls_v, aarch64_neon_cls, Add1ArgType),
  NEONMAP1(vclsq_v, aarch64_neon_cls, Add1ArgType),
  NEONMAP0(vcltz_v),
  NEONMAP0(vcltzq_v),
  NEONMAP1(vclz_v, ctlz, Add1ArgType),
  NEONMAP1(vclzq_v, ctlz, Add1ArgType),
  NEONMAP1(vcmla_f16, aarch64_neon_vcmla_rot0, Add1ArgType),
  NEONMAP1(vcmla_f32, aarch64_neon_vcmla_rot0, Add1ArgType),
````
- **L121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcale_v, aarch64_neon_facge, 0),`.
  **L121 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcale_v, aarch64_neon_facge, 0),`。
- **L122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcaleq_v, aarch64_neon_facge, 0),`.
  **L122 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcaleq_v, aarch64_neon_facge, 0),`。
- **L123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcalt_v, aarch64_neon_facgt, 0),`.
  **L123 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcalt_v, aarch64_neon_facgt, 0),`。
- **L124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcaltq_v, aarch64_neon_facgt, 0),`.
  **L124 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcaltq_v, aarch64_neon_facgt, 0),`。
- **L125 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vceqz_v),`.
  **L125 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vceqz_v),`。
- **L126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vceqzq_v),`.
  **L126 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vceqzq_v),`。
- **L127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vcgez_v),`.
  **L127 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vcgez_v),`。
- **L128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vcgezq_v),`.
  **L128 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vcgezq_v),`。
- **L129 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vcgtz_v),`.
  **L129 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vcgtz_v),`。
- **L130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vcgtzq_v),`.
  **L130 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vcgtzq_v),`。
- **L131 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vclez_v),`.
  **L131 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vclez_v),`。
- **L132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vclezq_v),`.
  **L132 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vclezq_v),`。
- **L133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcls_v, aarch64_neon_cls, Add1ArgType),`.
  **L133 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcls_v, aarch64_neon_cls, Add1ArgType),`。
- **L134 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vclsq_v, aarch64_neon_cls, Add1ArgType),`.
  **L134 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vclsq_v, aarch64_neon_cls, Add1ArgType),`。
- **L135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vcltz_v),`.
  **L135 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vcltz_v),`。
- **L136 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vcltzq_v),`.
  **L136 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vcltzq_v),`。
- **L137 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vclz_v, ctlz, Add1ArgType),`.
  **L137 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vclz_v, ctlz, Add1ArgType),`。
- **L138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vclzq_v, ctlz, Add1ArgType),`.
  **L138 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vclzq_v, ctlz, Add1ArgType),`。
- **L139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmla_f16, aarch64_neon_vcmla_rot0, Add1ArgType),`.
  **L139 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmla_f16, aarch64_neon_vcmla_rot0, Add1ArgType),`。
- **L140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmla_f32, aarch64_neon_vcmla_rot0, Add1ArgType),`.
  **L140 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmla_f32, aarch64_neon_vcmla_rot0, Add1ArgType),`。

### Lines 141-160

````cpp
  NEONMAP1(vcmla_rot180_f16, aarch64_neon_vcmla_rot180, Add1ArgType),
  NEONMAP1(vcmla_rot180_f32, aarch64_neon_vcmla_rot180, Add1ArgType),
  NEONMAP1(vcmla_rot270_f16, aarch64_neon_vcmla_rot270, Add1ArgType),
  NEONMAP1(vcmla_rot270_f32, aarch64_neon_vcmla_rot270, Add1ArgType),
  NEONMAP1(vcmla_rot90_f16, aarch64_neon_vcmla_rot90, Add1ArgType),
  NEONMAP1(vcmla_rot90_f32, aarch64_neon_vcmla_rot90, Add1ArgType),
  NEONMAP1(vcmlaq_f16, aarch64_neon_vcmla_rot0, Add1ArgType),
  NEONMAP1(vcmlaq_f32, aarch64_neon_vcmla_rot0, Add1ArgType),
  NEONMAP1(vcmlaq_f64, aarch64_neon_vcmla_rot0, Add1ArgType),
  NEONMAP1(vcmlaq_rot180_f16, aarch64_neon_vcmla_rot180, Add1ArgType),
  NEONMAP1(vcmlaq_rot180_f32, aarch64_neon_vcmla_rot180, Add1ArgType),
  NEONMAP1(vcmlaq_rot180_f64, aarch64_neon_vcmla_rot180, Add1ArgType),
  NEONMAP1(vcmlaq_rot270_f16, aarch64_neon_vcmla_rot270, Add1ArgType),
  NEONMAP1(vcmlaq_rot270_f32, aarch64_neon_vcmla_rot270, Add1ArgType),
  NEONMAP1(vcmlaq_rot270_f64, aarch64_neon_vcmla_rot270, Add1ArgType),
  NEONMAP1(vcmlaq_rot90_f16, aarch64_neon_vcmla_rot90, Add1ArgType),
  NEONMAP1(vcmlaq_rot90_f32, aarch64_neon_vcmla_rot90, Add1ArgType),
  NEONMAP1(vcmlaq_rot90_f64, aarch64_neon_vcmla_rot90, Add1ArgType),
  NEONMAP1(vcnt_v, ctpop, Add1ArgType),
  NEONMAP1(vcntq_v, ctpop, Add1ArgType),
````
- **L141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmla_rot180_f16, aarch64_neon_vcmla_rot180, Add1ArgType),`.
  **L141 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmla_rot180_f16, aarch64_neon_vcmla_rot180, Add1ArgType),`。
- **L142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmla_rot180_f32, aarch64_neon_vcmla_rot180, Add1ArgType),`.
  **L142 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmla_rot180_f32, aarch64_neon_vcmla_rot180, Add1ArgType),`。
- **L143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmla_rot270_f16, aarch64_neon_vcmla_rot270, Add1ArgType),`.
  **L143 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmla_rot270_f16, aarch64_neon_vcmla_rot270, Add1ArgType),`。
- **L144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmla_rot270_f32, aarch64_neon_vcmla_rot270, Add1ArgType),`.
  **L144 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmla_rot270_f32, aarch64_neon_vcmla_rot270, Add1ArgType),`。
- **L145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmla_rot90_f16, aarch64_neon_vcmla_rot90, Add1ArgType),`.
  **L145 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmla_rot90_f16, aarch64_neon_vcmla_rot90, Add1ArgType),`。
- **L146 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmla_rot90_f32, aarch64_neon_vcmla_rot90, Add1ArgType),`.
  **L146 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmla_rot90_f32, aarch64_neon_vcmla_rot90, Add1ArgType),`。
- **L147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmlaq_f16, aarch64_neon_vcmla_rot0, Add1ArgType),`.
  **L147 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmlaq_f16, aarch64_neon_vcmla_rot0, Add1ArgType),`。
- **L148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmlaq_f32, aarch64_neon_vcmla_rot0, Add1ArgType),`.
  **L148 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmlaq_f32, aarch64_neon_vcmla_rot0, Add1ArgType),`。
- **L149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmlaq_f64, aarch64_neon_vcmla_rot0, Add1ArgType),`.
  **L149 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmlaq_f64, aarch64_neon_vcmla_rot0, Add1ArgType),`。
- **L150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmlaq_rot180_f16, aarch64_neon_vcmla_rot180, Add1ArgType),`.
  **L150 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmlaq_rot180_f16, aarch64_neon_vcmla_rot180, Add1ArgType),`。
- **L151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmlaq_rot180_f32, aarch64_neon_vcmla_rot180, Add1ArgType),`.
  **L151 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmlaq_rot180_f32, aarch64_neon_vcmla_rot180, Add1ArgType),`。
- **L152 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmlaq_rot180_f64, aarch64_neon_vcmla_rot180, Add1ArgType),`.
  **L152 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmlaq_rot180_f64, aarch64_neon_vcmla_rot180, Add1ArgType),`。
- **L153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmlaq_rot270_f16, aarch64_neon_vcmla_rot270, Add1ArgType),`.
  **L153 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmlaq_rot270_f16, aarch64_neon_vcmla_rot270, Add1ArgType),`。
- **L154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmlaq_rot270_f32, aarch64_neon_vcmla_rot270, Add1ArgType),`.
  **L154 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmlaq_rot270_f32, aarch64_neon_vcmla_rot270, Add1ArgType),`。
- **L155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmlaq_rot270_f64, aarch64_neon_vcmla_rot270, Add1ArgType),`.
  **L155 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmlaq_rot270_f64, aarch64_neon_vcmla_rot270, Add1ArgType),`。
- **L156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmlaq_rot90_f16, aarch64_neon_vcmla_rot90, Add1ArgType),`.
  **L156 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmlaq_rot90_f16, aarch64_neon_vcmla_rot90, Add1ArgType),`。
- **L157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmlaq_rot90_f32, aarch64_neon_vcmla_rot90, Add1ArgType),`.
  **L157 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmlaq_rot90_f32, aarch64_neon_vcmla_rot90, Add1ArgType),`。
- **L158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcmlaq_rot90_f64, aarch64_neon_vcmla_rot90, Add1ArgType),`.
  **L158 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcmlaq_rot90_f64, aarch64_neon_vcmla_rot90, Add1ArgType),`。
- **L159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcnt_v, ctpop, Add1ArgType),`.
  **L159 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcnt_v, ctpop, Add1ArgType),`。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcntq_v, ctpop, Add1ArgType),`.
  **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcntq_v, ctpop, Add1ArgType),`。

### Lines 161-180

````cpp
  NEONMAP1(vcvt_f16_f32, aarch64_neon_vcvtfp2hf, 0),
  NEONMAP0(vcvt_f16_s16),
  NEONMAP0(vcvt_f16_u16),
  NEONMAP1(vcvt_f32_f16, aarch64_neon_vcvthf2fp, 0),
  NEONMAP0(vcvt_f32_v),
  NEONMAP1(vcvt_n_f16_s16, aarch64_neon_vcvtfxs2fp, 0),
  NEONMAP1(vcvt_n_f16_u16, aarch64_neon_vcvtfxu2fp, 0),
  NEONMAP2(vcvt_n_f32_v, aarch64_neon_vcvtfxu2fp, aarch64_neon_vcvtfxs2fp, 0),
  NEONMAP2(vcvt_n_f64_v, aarch64_neon_vcvtfxu2fp, aarch64_neon_vcvtfxs2fp, 0),
  NEONMAP1(vcvt_n_s16_f16, aarch64_neon_vcvtfp2fxs, 0),
  NEONMAP1(vcvt_n_s32_v, aarch64_neon_vcvtfp2fxs, 0),
  NEONMAP1(vcvt_n_s64_v, aarch64_neon_vcvtfp2fxs, 0),
  NEONMAP1(vcvt_n_u16_f16, aarch64_neon_vcvtfp2fxu, 0),
  NEONMAP1(vcvt_n_u32_v, aarch64_neon_vcvtfp2fxu, 0),
  NEONMAP1(vcvt_n_u64_v, aarch64_neon_vcvtfp2fxu, 0),
  NEONMAP1(vcvt_s16_f16, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),
  NEONMAP1(vcvt_s32_v, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),
  NEONMAP1(vcvt_s64_v, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),
  NEONMAP1(vcvt_u16_f16, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),
  NEONMAP1(vcvt_u32_v, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),
````
- **L161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvt_f16_f32, aarch64_neon_vcvtfp2hf, 0),`.
  **L161 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvt_f16_f32, aarch64_neon_vcvtfp2hf, 0),`。
- **L162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vcvt_f16_s16),`.
  **L162 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vcvt_f16_s16),`。
- **L163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vcvt_f16_u16),`.
  **L163 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vcvt_f16_u16),`。
- **L164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvt_f32_f16, aarch64_neon_vcvthf2fp, 0),`.
  **L164 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvt_f32_f16, aarch64_neon_vcvthf2fp, 0),`。
- **L165 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vcvt_f32_v),`.
  **L165 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vcvt_f32_v),`。
- **L166 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvt_n_f16_s16, aarch64_neon_vcvtfxs2fp, 0),`.
  **L166 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvt_n_f16_s16, aarch64_neon_vcvtfxs2fp, 0),`。
- **L167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvt_n_f16_u16, aarch64_neon_vcvtfxu2fp, 0),`.
  **L167 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvt_n_f16_u16, aarch64_neon_vcvtfxu2fp, 0),`。
- **L168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vcvt_n_f32_v, aarch64_neon_vcvtfxu2fp, aarch64_neon_vcvtfxs2fp, 0),`.
  **L168 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vcvt_n_f32_v, aarch64_neon_vcvtfxu2fp, aarch64_neon_vcvtfxs2fp, 0),`。
- **L169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vcvt_n_f64_v, aarch64_neon_vcvtfxu2fp, aarch64_neon_vcvtfxs2fp, 0),`.
  **L169 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vcvt_n_f64_v, aarch64_neon_vcvtfxu2fp, aarch64_neon_vcvtfxs2fp, 0),`。
- **L170 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvt_n_s16_f16, aarch64_neon_vcvtfp2fxs, 0),`.
  **L170 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvt_n_s16_f16, aarch64_neon_vcvtfp2fxs, 0),`。
- **L171 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvt_n_s32_v, aarch64_neon_vcvtfp2fxs, 0),`.
  **L171 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvt_n_s32_v, aarch64_neon_vcvtfp2fxs, 0),`。
- **L172 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvt_n_s64_v, aarch64_neon_vcvtfp2fxs, 0),`.
  **L172 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvt_n_s64_v, aarch64_neon_vcvtfp2fxs, 0),`。
- **L173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvt_n_u16_f16, aarch64_neon_vcvtfp2fxu, 0),`.
  **L173 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvt_n_u16_f16, aarch64_neon_vcvtfp2fxu, 0),`。
- **L174 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvt_n_u32_v, aarch64_neon_vcvtfp2fxu, 0),`.
  **L174 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvt_n_u32_v, aarch64_neon_vcvtfp2fxu, 0),`。
- **L175 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvt_n_u64_v, aarch64_neon_vcvtfp2fxu, 0),`.
  **L175 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvt_n_u64_v, aarch64_neon_vcvtfp2fxu, 0),`。
- **L176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvt_s16_f16, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`.
  **L176 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvt_s16_f16, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`。
- **L177 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvt_s32_v, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`.
  **L177 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvt_s32_v, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`。
- **L178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvt_s64_v, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`.
  **L178 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvt_s64_v, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`。
- **L179 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvt_u16_f16, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`.
  **L179 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvt_u16_f16, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`。
- **L180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvt_u32_v, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`.
  **L180 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvt_u32_v, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`。

### Lines 181-200

````cpp
  NEONMAP1(vcvt_u64_v, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),
  NEONMAP0(vcvtq_f16_s16),
  NEONMAP0(vcvtq_f16_u16),
  NEONMAP0(vcvtq_f32_v),
  NEONMAP1(vcvtq_n_f16_s16, aarch64_neon_vcvtfxs2fp, 0),
  NEONMAP1(vcvtq_n_f16_u16, aarch64_neon_vcvtfxu2fp, 0),
  NEONMAP2(vcvtq_n_f32_v, aarch64_neon_vcvtfxu2fp, aarch64_neon_vcvtfxs2fp, 0),
  NEONMAP2(vcvtq_n_f64_v, aarch64_neon_vcvtfxu2fp, aarch64_neon_vcvtfxs2fp, 0),
  NEONMAP1(vcvtq_n_s16_f16, aarch64_neon_vcvtfp2fxs, 0),
  NEONMAP1(vcvtq_n_s32_v, aarch64_neon_vcvtfp2fxs, 0),
  NEONMAP1(vcvtq_n_s64_v, aarch64_neon_vcvtfp2fxs, 0),
  NEONMAP1(vcvtq_n_u16_f16, aarch64_neon_vcvtfp2fxu, 0),
  NEONMAP1(vcvtq_n_u32_v, aarch64_neon_vcvtfp2fxu, 0),
  NEONMAP1(vcvtq_n_u64_v, aarch64_neon_vcvtfp2fxu, 0),
  NEONMAP1(vcvtq_s16_f16, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),
  NEONMAP1(vcvtq_s32_v, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),
  NEONMAP1(vcvtq_s64_v, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),
  NEONMAP1(vcvtq_u16_f16, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtq_u32_v, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtq_u64_v, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),
````
- **L181 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvt_u64_v, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`.
  **L181 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvt_u64_v, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`。
- **L182 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vcvtq_f16_s16),`.
  **L182 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vcvtq_f16_s16),`。
- **L183 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vcvtq_f16_u16),`.
  **L183 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vcvtq_f16_u16),`。
- **L184 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vcvtq_f32_v),`.
  **L184 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vcvtq_f32_v),`。
- **L185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtq_n_f16_s16, aarch64_neon_vcvtfxs2fp, 0),`.
  **L185 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtq_n_f16_s16, aarch64_neon_vcvtfxs2fp, 0),`。
- **L186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtq_n_f16_u16, aarch64_neon_vcvtfxu2fp, 0),`.
  **L186 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtq_n_f16_u16, aarch64_neon_vcvtfxu2fp, 0),`。
- **L187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vcvtq_n_f32_v, aarch64_neon_vcvtfxu2fp, aarch64_neon_vcvtfxs2fp, 0),`.
  **L187 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vcvtq_n_f32_v, aarch64_neon_vcvtfxu2fp, aarch64_neon_vcvtfxs2fp, 0),`。
- **L188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vcvtq_n_f64_v, aarch64_neon_vcvtfxu2fp, aarch64_neon_vcvtfxs2fp, 0),`.
  **L188 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vcvtq_n_f64_v, aarch64_neon_vcvtfxu2fp, aarch64_neon_vcvtfxs2fp, 0),`。
- **L189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtq_n_s16_f16, aarch64_neon_vcvtfp2fxs, 0),`.
  **L189 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtq_n_s16_f16, aarch64_neon_vcvtfp2fxs, 0),`。
- **L190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtq_n_s32_v, aarch64_neon_vcvtfp2fxs, 0),`.
  **L190 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtq_n_s32_v, aarch64_neon_vcvtfp2fxs, 0),`。
- **L191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtq_n_s64_v, aarch64_neon_vcvtfp2fxs, 0),`.
  **L191 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtq_n_s64_v, aarch64_neon_vcvtfp2fxs, 0),`。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtq_n_u16_f16, aarch64_neon_vcvtfp2fxu, 0),`.
  **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtq_n_u16_f16, aarch64_neon_vcvtfp2fxu, 0),`。
- **L193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtq_n_u32_v, aarch64_neon_vcvtfp2fxu, 0),`.
  **L193 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtq_n_u32_v, aarch64_neon_vcvtfp2fxu, 0),`。
- **L194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtq_n_u64_v, aarch64_neon_vcvtfp2fxu, 0),`.
  **L194 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtq_n_u64_v, aarch64_neon_vcvtfp2fxu, 0),`。
- **L195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtq_s16_f16, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`.
  **L195 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtq_s16_f16, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`。
- **L196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtq_s32_v, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`.
  **L196 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtq_s32_v, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`。
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtq_s64_v, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtq_s64_v, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`。
- **L198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtq_u16_f16, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`.
  **L198 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtq_u16_f16, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`。
- **L199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtq_u32_v, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`.
  **L199 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtq_u32_v, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`。
- **L200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtq_u64_v, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`.
  **L200 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtq_u64_v, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`。

### Lines 201-220

````cpp
  NEONMAP1(vcvtx_f32_v, aarch64_neon_fcvtxn, AddRetType | Add1ArgType),
  NEONMAP1(vdot_s32, aarch64_neon_sdot, 0),
  NEONMAP1(vdot_u32, aarch64_neon_udot, 0),
  NEONMAP1(vdotq_s32, aarch64_neon_sdot, 0),
  NEONMAP1(vdotq_u32, aarch64_neon_udot, 0),
  NEONMAP2(veor3q_s16, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),
  NEONMAP2(veor3q_s32, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),
  NEONMAP2(veor3q_s64, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),
  NEONMAP2(veor3q_s8, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),
  NEONMAP2(veor3q_u16, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),
  NEONMAP2(veor3q_u32, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),
  NEONMAP2(veor3q_u64, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),
  NEONMAP2(veor3q_u8, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),
  NEONMAP0(vext_v),
  NEONMAP0(vextq_v),
  NEONMAP0(vfma_v),
  NEONMAP0(vfmaq_v),
  NEONMAP1(vfmlal_high_f16, aarch64_neon_fmlal2, 0),
  NEONMAP1(vfmlal_low_f16, aarch64_neon_fmlal, 0),
  NEONMAP1(vfmlalq_high_f16, aarch64_neon_fmlal2, 0),
````
- **L201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtx_f32_v, aarch64_neon_fcvtxn, AddRetType | Add1ArgType),`.
  **L201 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtx_f32_v, aarch64_neon_fcvtxn, AddRetType | Add1ArgType),`。
- **L202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vdot_s32, aarch64_neon_sdot, 0),`.
  **L202 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vdot_s32, aarch64_neon_sdot, 0),`。
- **L203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vdot_u32, aarch64_neon_udot, 0),`.
  **L203 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vdot_u32, aarch64_neon_udot, 0),`。
- **L204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vdotq_s32, aarch64_neon_sdot, 0),`.
  **L204 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vdotq_s32, aarch64_neon_sdot, 0),`。
- **L205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vdotq_u32, aarch64_neon_udot, 0),`.
  **L205 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vdotq_u32, aarch64_neon_udot, 0),`。
- **L206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(veor3q_s16, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),`.
  **L206 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(veor3q_s16, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),`。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(veor3q_s32, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),`.
  **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(veor3q_s32, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),`。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(veor3q_s64, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),`.
  **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(veor3q_s64, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),`。
- **L209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(veor3q_s8, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),`.
  **L209 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(veor3q_s8, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),`。
- **L210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(veor3q_u16, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),`.
  **L210 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(veor3q_u16, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),`。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(veor3q_u32, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(veor3q_u32, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),`。
- **L212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(veor3q_u64, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),`.
  **L212 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(veor3q_u64, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),`。
- **L213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(veor3q_u8, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),`.
  **L213 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(veor3q_u8, aarch64_crypto_eor3u, aarch64_crypto_eor3s, Add1ArgType | UnsignedAlts),`。
- **L214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vext_v),`.
  **L214 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vext_v),`。
- **L215 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vextq_v),`.
  **L215 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vextq_v),`。
- **L216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vfma_v),`.
  **L216 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vfma_v),`。
- **L217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vfmaq_v),`.
  **L217 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vfmaq_v),`。
- **L218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vfmlal_high_f16, aarch64_neon_fmlal2, 0),`.
  **L218 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vfmlal_high_f16, aarch64_neon_fmlal2, 0),`。
- **L219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vfmlal_low_f16, aarch64_neon_fmlal, 0),`.
  **L219 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vfmlal_low_f16, aarch64_neon_fmlal, 0),`。
- **L220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vfmlalq_high_f16, aarch64_neon_fmlal2, 0),`.
  **L220 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vfmlalq_high_f16, aarch64_neon_fmlal2, 0),`。

### Lines 221-240

````cpp
  NEONMAP1(vfmlalq_low_f16, aarch64_neon_fmlal, 0),
  NEONMAP1(vfmlsl_high_f16, aarch64_neon_fmlsl2, 0),
  NEONMAP1(vfmlsl_low_f16, aarch64_neon_fmlsl, 0),
  NEONMAP1(vfmlslq_high_f16, aarch64_neon_fmlsl2, 0),
  NEONMAP1(vfmlslq_low_f16, aarch64_neon_fmlsl, 0),
  NEONMAP2(vhadd_v, aarch64_neon_uhadd, aarch64_neon_shadd, Add1ArgType | UnsignedAlts),
  NEONMAP2(vhaddq_v, aarch64_neon_uhadd, aarch64_neon_shadd, Add1ArgType | UnsignedAlts),
  NEONMAP2(vhsub_v, aarch64_neon_uhsub, aarch64_neon_shsub, Add1ArgType | UnsignedAlts),
  NEONMAP2(vhsubq_v, aarch64_neon_uhsub, aarch64_neon_shsub, Add1ArgType | UnsignedAlts),
  NEONMAP1(vld1_x2_v, aarch64_neon_ld1x2, 0),
  NEONMAP1(vld1_x3_v, aarch64_neon_ld1x3, 0),
  NEONMAP1(vld1_x4_v, aarch64_neon_ld1x4, 0),
  NEONMAP1(vld1q_x2_v, aarch64_neon_ld1x2, 0),
  NEONMAP1(vld1q_x3_v, aarch64_neon_ld1x3, 0),
  NEONMAP1(vld1q_x4_v, aarch64_neon_ld1x4, 0),
  NEONMAP1(vmmlaq_f16_f16, aarch64_neon_fmmla, 0),
  NEONMAP1(vmmlaq_f32_f16, aarch64_neon_fmmla, 0),
  NEONMAP1(vmmlaq_s32, aarch64_neon_smmla, 0),
  NEONMAP1(vmmlaq_u32, aarch64_neon_ummla, 0),
  NEONMAP0(vmovl_v),
````
- **L221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vfmlalq_low_f16, aarch64_neon_fmlal, 0),`.
  **L221 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vfmlalq_low_f16, aarch64_neon_fmlal, 0),`。
- **L222 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vfmlsl_high_f16, aarch64_neon_fmlsl2, 0),`.
  **L222 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vfmlsl_high_f16, aarch64_neon_fmlsl2, 0),`。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vfmlsl_low_f16, aarch64_neon_fmlsl, 0),`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vfmlsl_low_f16, aarch64_neon_fmlsl, 0),`。
- **L224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vfmlslq_high_f16, aarch64_neon_fmlsl2, 0),`.
  **L224 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vfmlslq_high_f16, aarch64_neon_fmlsl2, 0),`。
- **L225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vfmlslq_low_f16, aarch64_neon_fmlsl, 0),`.
  **L225 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vfmlslq_low_f16, aarch64_neon_fmlsl, 0),`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vhadd_v, aarch64_neon_uhadd, aarch64_neon_shadd, Add1ArgType | UnsignedAlts),`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vhadd_v, aarch64_neon_uhadd, aarch64_neon_shadd, Add1ArgType | UnsignedAlts),`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vhaddq_v, aarch64_neon_uhadd, aarch64_neon_shadd, Add1ArgType | UnsignedAlts),`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vhaddq_v, aarch64_neon_uhadd, aarch64_neon_shadd, Add1ArgType | UnsignedAlts),`。
- **L228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vhsub_v, aarch64_neon_uhsub, aarch64_neon_shsub, Add1ArgType | UnsignedAlts),`.
  **L228 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vhsub_v, aarch64_neon_uhsub, aarch64_neon_shsub, Add1ArgType | UnsignedAlts),`。
- **L229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vhsubq_v, aarch64_neon_uhsub, aarch64_neon_shsub, Add1ArgType | UnsignedAlts),`.
  **L229 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vhsubq_v, aarch64_neon_uhsub, aarch64_neon_shsub, Add1ArgType | UnsignedAlts),`。
- **L230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vld1_x2_v, aarch64_neon_ld1x2, 0),`.
  **L230 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vld1_x2_v, aarch64_neon_ld1x2, 0),`。
- **L231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vld1_x3_v, aarch64_neon_ld1x3, 0),`.
  **L231 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vld1_x3_v, aarch64_neon_ld1x3, 0),`。
- **L232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vld1_x4_v, aarch64_neon_ld1x4, 0),`.
  **L232 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vld1_x4_v, aarch64_neon_ld1x4, 0),`。
- **L233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vld1q_x2_v, aarch64_neon_ld1x2, 0),`.
  **L233 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vld1q_x2_v, aarch64_neon_ld1x2, 0),`。
- **L234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vld1q_x3_v, aarch64_neon_ld1x3, 0),`.
  **L234 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vld1q_x3_v, aarch64_neon_ld1x3, 0),`。
- **L235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vld1q_x4_v, aarch64_neon_ld1x4, 0),`.
  **L235 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vld1q_x4_v, aarch64_neon_ld1x4, 0),`。
- **L236 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmmlaq_f16_f16, aarch64_neon_fmmla, 0),`.
  **L236 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmmlaq_f16_f16, aarch64_neon_fmmla, 0),`。
- **L237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmmlaq_f32_f16, aarch64_neon_fmmla, 0),`.
  **L237 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmmlaq_f32_f16, aarch64_neon_fmmla, 0),`。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmmlaq_s32, aarch64_neon_smmla, 0),`.
  **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmmlaq_s32, aarch64_neon_smmla, 0),`。
- **L239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmmlaq_u32, aarch64_neon_ummla, 0),`.
  **L239 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmmlaq_u32, aarch64_neon_ummla, 0),`。
- **L240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vmovl_v),`.
  **L240 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vmovl_v),`。

### Lines 241-260

````cpp
  NEONMAP0(vmovn_v),
  NEONMAP1(vmul_v, aarch64_neon_pmul, Add1ArgType),
  NEONMAP1(vmulq_v, aarch64_neon_pmul, Add1ArgType),
  NEONMAP1(vpadd_v, aarch64_neon_addp, Add1ArgType),
  NEONMAP2(vpaddl_v, aarch64_neon_uaddlp, aarch64_neon_saddlp, UnsignedAlts),
  NEONMAP2(vpaddlq_v, aarch64_neon_uaddlp, aarch64_neon_saddlp, UnsignedAlts),
  NEONMAP1(vpaddq_v, aarch64_neon_addp, Add1ArgType),
  NEONMAP1(vqabs_v, aarch64_neon_sqabs, Add1ArgType),
  NEONMAP1(vqabsq_v, aarch64_neon_sqabs, Add1ArgType),
  NEONMAP2(vqadd_v, aarch64_neon_uqadd, aarch64_neon_sqadd, Add1ArgType | UnsignedAlts),
  NEONMAP2(vqaddq_v, aarch64_neon_uqadd, aarch64_neon_sqadd, Add1ArgType | UnsignedAlts),
  NEONMAP2(vqdmlal_v, aarch64_neon_sqdmull, aarch64_neon_sqadd, 0),
  NEONMAP2(vqdmlsl_v, aarch64_neon_sqdmull, aarch64_neon_sqsub, 0),
  NEONMAP1(vqdmulh_lane_v, aarch64_neon_sqdmulh_lane, 0),
  NEONMAP1(vqdmulh_laneq_v, aarch64_neon_sqdmulh_laneq, 0),
  NEONMAP1(vqdmulh_v, aarch64_neon_sqdmulh, Add1ArgType),
  NEONMAP1(vqdmulhq_lane_v, aarch64_neon_sqdmulh_lane, 0),
  NEONMAP1(vqdmulhq_laneq_v, aarch64_neon_sqdmulh_laneq, 0),
  NEONMAP1(vqdmulhq_v, aarch64_neon_sqdmulh, Add1ArgType),
  NEONMAP1(vqdmull_v, aarch64_neon_sqdmull, Add1ArgType),
````
- **L241 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vmovn_v),`.
  **L241 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vmovn_v),`。
- **L242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmul_v, aarch64_neon_pmul, Add1ArgType),`.
  **L242 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmul_v, aarch64_neon_pmul, Add1ArgType),`。
- **L243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmulq_v, aarch64_neon_pmul, Add1ArgType),`.
  **L243 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmulq_v, aarch64_neon_pmul, Add1ArgType),`。
- **L244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vpadd_v, aarch64_neon_addp, Add1ArgType),`.
  **L244 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vpadd_v, aarch64_neon_addp, Add1ArgType),`。
- **L245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vpaddl_v, aarch64_neon_uaddlp, aarch64_neon_saddlp, UnsignedAlts),`.
  **L245 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vpaddl_v, aarch64_neon_uaddlp, aarch64_neon_saddlp, UnsignedAlts),`。
- **L246 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vpaddlq_v, aarch64_neon_uaddlp, aarch64_neon_saddlp, UnsignedAlts),`.
  **L246 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vpaddlq_v, aarch64_neon_uaddlp, aarch64_neon_saddlp, UnsignedAlts),`。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vpaddq_v, aarch64_neon_addp, Add1ArgType),`.
  **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vpaddq_v, aarch64_neon_addp, Add1ArgType),`。
- **L248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqabs_v, aarch64_neon_sqabs, Add1ArgType),`.
  **L248 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqabs_v, aarch64_neon_sqabs, Add1ArgType),`。
- **L249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqabsq_v, aarch64_neon_sqabs, Add1ArgType),`.
  **L249 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqabsq_v, aarch64_neon_sqabs, Add1ArgType),`。
- **L250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vqadd_v, aarch64_neon_uqadd, aarch64_neon_sqadd, Add1ArgType | UnsignedAlts),`.
  **L250 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vqadd_v, aarch64_neon_uqadd, aarch64_neon_sqadd, Add1ArgType | UnsignedAlts),`。
- **L251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vqaddq_v, aarch64_neon_uqadd, aarch64_neon_sqadd, Add1ArgType | UnsignedAlts),`.
  **L251 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vqaddq_v, aarch64_neon_uqadd, aarch64_neon_sqadd, Add1ArgType | UnsignedAlts),`。
- **L252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vqdmlal_v, aarch64_neon_sqdmull, aarch64_neon_sqadd, 0),`.
  **L252 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vqdmlal_v, aarch64_neon_sqdmull, aarch64_neon_sqadd, 0),`。
- **L253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vqdmlsl_v, aarch64_neon_sqdmull, aarch64_neon_sqsub, 0),`.
  **L253 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vqdmlsl_v, aarch64_neon_sqdmull, aarch64_neon_sqsub, 0),`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqdmulh_lane_v, aarch64_neon_sqdmulh_lane, 0),`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqdmulh_lane_v, aarch64_neon_sqdmulh_lane, 0),`。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqdmulh_laneq_v, aarch64_neon_sqdmulh_laneq, 0),`.
  **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqdmulh_laneq_v, aarch64_neon_sqdmulh_laneq, 0),`。
- **L256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqdmulh_v, aarch64_neon_sqdmulh, Add1ArgType),`.
  **L256 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqdmulh_v, aarch64_neon_sqdmulh, Add1ArgType),`。
- **L257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqdmulhq_lane_v, aarch64_neon_sqdmulh_lane, 0),`.
  **L257 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqdmulhq_lane_v, aarch64_neon_sqdmulh_lane, 0),`。
- **L258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqdmulhq_laneq_v, aarch64_neon_sqdmulh_laneq, 0),`.
  **L258 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqdmulhq_laneq_v, aarch64_neon_sqdmulh_laneq, 0),`。
- **L259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqdmulhq_v, aarch64_neon_sqdmulh, Add1ArgType),`.
  **L259 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqdmulhq_v, aarch64_neon_sqdmulh, Add1ArgType),`。
- **L260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqdmull_v, aarch64_neon_sqdmull, Add1ArgType),`.
  **L260 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqdmull_v, aarch64_neon_sqdmull, Add1ArgType),`。

### Lines 261-280

````cpp
  NEONMAP2(vqmovn_v, aarch64_neon_uqxtn, aarch64_neon_sqxtn, Add1ArgType | UnsignedAlts),
  NEONMAP1(vqmovun_v, aarch64_neon_sqxtun, Add1ArgType),
  NEONMAP1(vqneg_v, aarch64_neon_sqneg, Add1ArgType),
  NEONMAP1(vqnegq_v, aarch64_neon_sqneg, Add1ArgType),
  NEONMAP1(vqrdmlah_s16, aarch64_neon_sqrdmlah, Add1ArgType),
  NEONMAP1(vqrdmlah_s32, aarch64_neon_sqrdmlah, Add1ArgType),
  NEONMAP1(vqrdmlahq_s16, aarch64_neon_sqrdmlah, Add1ArgType),
  NEONMAP1(vqrdmlahq_s32, aarch64_neon_sqrdmlah, Add1ArgType),
  NEONMAP1(vqrdmlsh_s16, aarch64_neon_sqrdmlsh, Add1ArgType),
  NEONMAP1(vqrdmlsh_s32, aarch64_neon_sqrdmlsh, Add1ArgType),
  NEONMAP1(vqrdmlshq_s16, aarch64_neon_sqrdmlsh, Add1ArgType),
  NEONMAP1(vqrdmlshq_s32, aarch64_neon_sqrdmlsh, Add1ArgType),
  NEONMAP1(vqrdmulh_lane_v, aarch64_neon_sqrdmulh_lane, 0),
  NEONMAP1(vqrdmulh_laneq_v, aarch64_neon_sqrdmulh_laneq, 0),
  NEONMAP1(vqrdmulh_v, aarch64_neon_sqrdmulh, Add1ArgType),
  NEONMAP1(vqrdmulhq_lane_v, aarch64_neon_sqrdmulh_lane, 0),
  NEONMAP1(vqrdmulhq_laneq_v, aarch64_neon_sqrdmulh_laneq, 0),
  NEONMAP1(vqrdmulhq_v, aarch64_neon_sqrdmulh, Add1ArgType),
  NEONMAP2(vqrshl_v, aarch64_neon_uqrshl, aarch64_neon_sqrshl, Add1ArgType | UnsignedAlts),
  NEONMAP2(vqrshlq_v, aarch64_neon_uqrshl, aarch64_neon_sqrshl, Add1ArgType | UnsignedAlts),
````
- **L261 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vqmovn_v, aarch64_neon_uqxtn, aarch64_neon_sqxtn, Add1ArgType | UnsignedAlts),`.
  **L261 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vqmovn_v, aarch64_neon_uqxtn, aarch64_neon_sqxtn, Add1ArgType | UnsignedAlts),`。
- **L262 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqmovun_v, aarch64_neon_sqxtun, Add1ArgType),`.
  **L262 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqmovun_v, aarch64_neon_sqxtun, Add1ArgType),`。
- **L263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqneg_v, aarch64_neon_sqneg, Add1ArgType),`.
  **L263 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqneg_v, aarch64_neon_sqneg, Add1ArgType),`。
- **L264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqnegq_v, aarch64_neon_sqneg, Add1ArgType),`.
  **L264 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqnegq_v, aarch64_neon_sqneg, Add1ArgType),`。
- **L265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmlah_s16, aarch64_neon_sqrdmlah, Add1ArgType),`.
  **L265 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmlah_s16, aarch64_neon_sqrdmlah, Add1ArgType),`。
- **L266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmlah_s32, aarch64_neon_sqrdmlah, Add1ArgType),`.
  **L266 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmlah_s32, aarch64_neon_sqrdmlah, Add1ArgType),`。
- **L267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmlahq_s16, aarch64_neon_sqrdmlah, Add1ArgType),`.
  **L267 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmlahq_s16, aarch64_neon_sqrdmlah, Add1ArgType),`。
- **L268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmlahq_s32, aarch64_neon_sqrdmlah, Add1ArgType),`.
  **L268 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmlahq_s32, aarch64_neon_sqrdmlah, Add1ArgType),`。
- **L269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmlsh_s16, aarch64_neon_sqrdmlsh, Add1ArgType),`.
  **L269 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmlsh_s16, aarch64_neon_sqrdmlsh, Add1ArgType),`。
- **L270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmlsh_s32, aarch64_neon_sqrdmlsh, Add1ArgType),`.
  **L270 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmlsh_s32, aarch64_neon_sqrdmlsh, Add1ArgType),`。
- **L271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmlshq_s16, aarch64_neon_sqrdmlsh, Add1ArgType),`.
  **L271 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmlshq_s16, aarch64_neon_sqrdmlsh, Add1ArgType),`。
- **L272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmlshq_s32, aarch64_neon_sqrdmlsh, Add1ArgType),`.
  **L272 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmlshq_s32, aarch64_neon_sqrdmlsh, Add1ArgType),`。
- **L273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmulh_lane_v, aarch64_neon_sqrdmulh_lane, 0),`.
  **L273 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmulh_lane_v, aarch64_neon_sqrdmulh_lane, 0),`。
- **L274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmulh_laneq_v, aarch64_neon_sqrdmulh_laneq, 0),`.
  **L274 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmulh_laneq_v, aarch64_neon_sqrdmulh_laneq, 0),`。
- **L275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmulh_v, aarch64_neon_sqrdmulh, Add1ArgType),`.
  **L275 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmulh_v, aarch64_neon_sqrdmulh, Add1ArgType),`。
- **L276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmulhq_lane_v, aarch64_neon_sqrdmulh_lane, 0),`.
  **L276 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmulhq_lane_v, aarch64_neon_sqrdmulh_lane, 0),`。
- **L277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmulhq_laneq_v, aarch64_neon_sqrdmulh_laneq, 0),`.
  **L277 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmulhq_laneq_v, aarch64_neon_sqrdmulh_laneq, 0),`。
- **L278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmulhq_v, aarch64_neon_sqrdmulh, Add1ArgType),`.
  **L278 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmulhq_v, aarch64_neon_sqrdmulh, Add1ArgType),`。
- **L279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vqrshl_v, aarch64_neon_uqrshl, aarch64_neon_sqrshl, Add1ArgType | UnsignedAlts),`.
  **L279 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vqrshl_v, aarch64_neon_uqrshl, aarch64_neon_sqrshl, Add1ArgType | UnsignedAlts),`。
- **L280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vqrshlq_v, aarch64_neon_uqrshl, aarch64_neon_sqrshl, Add1ArgType | UnsignedAlts),`.
  **L280 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vqrshlq_v, aarch64_neon_uqrshl, aarch64_neon_sqrshl, Add1ArgType | UnsignedAlts),`。

### Lines 281-300

````cpp
  NEONMAP2(vqshl_n_v, aarch64_neon_uqshl, aarch64_neon_sqshl, UnsignedAlts),
  NEONMAP2(vqshl_v, aarch64_neon_uqshl, aarch64_neon_sqshl, Add1ArgType | UnsignedAlts),
  NEONMAP2(vqshlq_n_v, aarch64_neon_uqshl, aarch64_neon_sqshl,UnsignedAlts),
  NEONMAP2(vqshlq_v, aarch64_neon_uqshl, aarch64_neon_sqshl, Add1ArgType | UnsignedAlts),
  NEONMAP1(vqshlu_n_v, aarch64_neon_sqshlu, 0),
  NEONMAP1(vqshluq_n_v, aarch64_neon_sqshlu, 0),
  NEONMAP2(vqsub_v, aarch64_neon_uqsub, aarch64_neon_sqsub, Add1ArgType | UnsignedAlts),
  NEONMAP2(vqsubq_v, aarch64_neon_uqsub, aarch64_neon_sqsub, Add1ArgType | UnsignedAlts),
  NEONMAP1(vraddhn_v, aarch64_neon_raddhn, Add1ArgType),
  NEONMAP1(vrax1q_u64, aarch64_crypto_rax1, 0),
  NEONMAP2(vrecpe_v, aarch64_neon_frecpe, aarch64_neon_urecpe, 0),
  NEONMAP2(vrecpeq_v, aarch64_neon_frecpe, aarch64_neon_urecpe, 0),
  NEONMAP1(vrecps_v, aarch64_neon_frecps, Add1ArgType),
  NEONMAP1(vrecpsq_v, aarch64_neon_frecps, Add1ArgType),
  NEONMAP2(vrhadd_v, aarch64_neon_urhadd, aarch64_neon_srhadd, Add1ArgType | UnsignedAlts),
  NEONMAP2(vrhaddq_v, aarch64_neon_urhadd, aarch64_neon_srhadd, Add1ArgType | UnsignedAlts),
  NEONMAP1(vrnd32x_f32, aarch64_neon_frint32x, Add1ArgType),
  NEONMAP1(vrnd32x_f64, aarch64_neon_frint32x, Add1ArgType),
  NEONMAP1(vrnd32xq_f32, aarch64_neon_frint32x, Add1ArgType),
  NEONMAP1(vrnd32xq_f64, aarch64_neon_frint32x, Add1ArgType),
````
- **L281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vqshl_n_v, aarch64_neon_uqshl, aarch64_neon_sqshl, UnsignedAlts),`.
  **L281 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vqshl_n_v, aarch64_neon_uqshl, aarch64_neon_sqshl, UnsignedAlts),`。
- **L282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vqshl_v, aarch64_neon_uqshl, aarch64_neon_sqshl, Add1ArgType | UnsignedAlts),`.
  **L282 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vqshl_v, aarch64_neon_uqshl, aarch64_neon_sqshl, Add1ArgType | UnsignedAlts),`。
- **L283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vqshlq_n_v, aarch64_neon_uqshl, aarch64_neon_sqshl,UnsignedAlts),`.
  **L283 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vqshlq_n_v, aarch64_neon_uqshl, aarch64_neon_sqshl,UnsignedAlts),`。
- **L284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vqshlq_v, aarch64_neon_uqshl, aarch64_neon_sqshl, Add1ArgType | UnsignedAlts),`.
  **L284 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vqshlq_v, aarch64_neon_uqshl, aarch64_neon_sqshl, Add1ArgType | UnsignedAlts),`。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshlu_n_v, aarch64_neon_sqshlu, 0),`.
  **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshlu_n_v, aarch64_neon_sqshlu, 0),`。
- **L286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshluq_n_v, aarch64_neon_sqshlu, 0),`.
  **L286 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshluq_n_v, aarch64_neon_sqshlu, 0),`。
- **L287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vqsub_v, aarch64_neon_uqsub, aarch64_neon_sqsub, Add1ArgType | UnsignedAlts),`.
  **L287 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vqsub_v, aarch64_neon_uqsub, aarch64_neon_sqsub, Add1ArgType | UnsignedAlts),`。
- **L288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vqsubq_v, aarch64_neon_uqsub, aarch64_neon_sqsub, Add1ArgType | UnsignedAlts),`.
  **L288 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vqsubq_v, aarch64_neon_uqsub, aarch64_neon_sqsub, Add1ArgType | UnsignedAlts),`。
- **L289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vraddhn_v, aarch64_neon_raddhn, Add1ArgType),`.
  **L289 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vraddhn_v, aarch64_neon_raddhn, Add1ArgType),`。
- **L290 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrax1q_u64, aarch64_crypto_rax1, 0),`.
  **L290 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrax1q_u64, aarch64_crypto_rax1, 0),`。
- **L291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vrecpe_v, aarch64_neon_frecpe, aarch64_neon_urecpe, 0),`.
  **L291 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vrecpe_v, aarch64_neon_frecpe, aarch64_neon_urecpe, 0),`。
- **L292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vrecpeq_v, aarch64_neon_frecpe, aarch64_neon_urecpe, 0),`.
  **L292 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vrecpeq_v, aarch64_neon_frecpe, aarch64_neon_urecpe, 0),`。
- **L293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrecps_v, aarch64_neon_frecps, Add1ArgType),`.
  **L293 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrecps_v, aarch64_neon_frecps, Add1ArgType),`。
- **L294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrecpsq_v, aarch64_neon_frecps, Add1ArgType),`.
  **L294 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrecpsq_v, aarch64_neon_frecps, Add1ArgType),`。
- **L295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vrhadd_v, aarch64_neon_urhadd, aarch64_neon_srhadd, Add1ArgType | UnsignedAlts),`.
  **L295 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vrhadd_v, aarch64_neon_urhadd, aarch64_neon_srhadd, Add1ArgType | UnsignedAlts),`。
- **L296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vrhaddq_v, aarch64_neon_urhadd, aarch64_neon_srhadd, Add1ArgType | UnsignedAlts),`.
  **L296 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vrhaddq_v, aarch64_neon_urhadd, aarch64_neon_srhadd, Add1ArgType | UnsignedAlts),`。
- **L297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrnd32x_f32, aarch64_neon_frint32x, Add1ArgType),`.
  **L297 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrnd32x_f32, aarch64_neon_frint32x, Add1ArgType),`。
- **L298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrnd32x_f64, aarch64_neon_frint32x, Add1ArgType),`.
  **L298 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrnd32x_f64, aarch64_neon_frint32x, Add1ArgType),`。
- **L299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrnd32xq_f32, aarch64_neon_frint32x, Add1ArgType),`.
  **L299 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrnd32xq_f32, aarch64_neon_frint32x, Add1ArgType),`。
- **L300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrnd32xq_f64, aarch64_neon_frint32x, Add1ArgType),`.
  **L300 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrnd32xq_f64, aarch64_neon_frint32x, Add1ArgType),`。

### Lines 301-320

````cpp
  NEONMAP1(vrnd32z_f32, aarch64_neon_frint32z, Add1ArgType),
  NEONMAP1(vrnd32z_f64, aarch64_neon_frint32z, Add1ArgType),
  NEONMAP1(vrnd32zq_f32, aarch64_neon_frint32z, Add1ArgType),
  NEONMAP1(vrnd32zq_f64, aarch64_neon_frint32z, Add1ArgType),
  NEONMAP1(vrnd64x_f32, aarch64_neon_frint64x, Add1ArgType),
  NEONMAP1(vrnd64x_f64, aarch64_neon_frint64x, Add1ArgType),
  NEONMAP1(vrnd64xq_f32, aarch64_neon_frint64x, Add1ArgType),
  NEONMAP1(vrnd64xq_f64, aarch64_neon_frint64x, Add1ArgType),
  NEONMAP1(vrnd64z_f32, aarch64_neon_frint64z, Add1ArgType),
  NEONMAP1(vrnd64z_f64, aarch64_neon_frint64z, Add1ArgType),
  NEONMAP1(vrnd64zq_f32, aarch64_neon_frint64z, Add1ArgType),
  NEONMAP1(vrnd64zq_f64, aarch64_neon_frint64z, Add1ArgType),
  NEONMAP0(vrndi_v),
  NEONMAP0(vrndiq_v),
  NEONMAP2(vrshl_v, aarch64_neon_urshl, aarch64_neon_srshl, Add1ArgType | UnsignedAlts),
  NEONMAP2(vrshlq_v, aarch64_neon_urshl, aarch64_neon_srshl, Add1ArgType | UnsignedAlts),
  NEONMAP2(vrshr_n_v, aarch64_neon_urshl, aarch64_neon_srshl, UnsignedAlts),
  NEONMAP2(vrshrq_n_v, aarch64_neon_urshl, aarch64_neon_srshl, UnsignedAlts),
  NEONMAP2(vrsqrte_v, aarch64_neon_frsqrte, aarch64_neon_ursqrte, 0),
  NEONMAP2(vrsqrteq_v, aarch64_neon_frsqrte, aarch64_neon_ursqrte, 0),
````
- **L301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrnd32z_f32, aarch64_neon_frint32z, Add1ArgType),`.
  **L301 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrnd32z_f32, aarch64_neon_frint32z, Add1ArgType),`。
- **L302 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrnd32z_f64, aarch64_neon_frint32z, Add1ArgType),`.
  **L302 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrnd32z_f64, aarch64_neon_frint32z, Add1ArgType),`。
- **L303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrnd32zq_f32, aarch64_neon_frint32z, Add1ArgType),`.
  **L303 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrnd32zq_f32, aarch64_neon_frint32z, Add1ArgType),`。
- **L304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrnd32zq_f64, aarch64_neon_frint32z, Add1ArgType),`.
  **L304 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrnd32zq_f64, aarch64_neon_frint32z, Add1ArgType),`。
- **L305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrnd64x_f32, aarch64_neon_frint64x, Add1ArgType),`.
  **L305 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrnd64x_f32, aarch64_neon_frint64x, Add1ArgType),`。
- **L306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrnd64x_f64, aarch64_neon_frint64x, Add1ArgType),`.
  **L306 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrnd64x_f64, aarch64_neon_frint64x, Add1ArgType),`。
- **L307 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrnd64xq_f32, aarch64_neon_frint64x, Add1ArgType),`.
  **L307 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrnd64xq_f32, aarch64_neon_frint64x, Add1ArgType),`。
- **L308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrnd64xq_f64, aarch64_neon_frint64x, Add1ArgType),`.
  **L308 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrnd64xq_f64, aarch64_neon_frint64x, Add1ArgType),`。
- **L309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrnd64z_f32, aarch64_neon_frint64z, Add1ArgType),`.
  **L309 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrnd64z_f32, aarch64_neon_frint64z, Add1ArgType),`。
- **L310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrnd64z_f64, aarch64_neon_frint64z, Add1ArgType),`.
  **L310 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrnd64z_f64, aarch64_neon_frint64z, Add1ArgType),`。
- **L311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrnd64zq_f32, aarch64_neon_frint64z, Add1ArgType),`.
  **L311 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrnd64zq_f32, aarch64_neon_frint64z, Add1ArgType),`。
- **L312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrnd64zq_f64, aarch64_neon_frint64z, Add1ArgType),`.
  **L312 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrnd64zq_f64, aarch64_neon_frint64z, Add1ArgType),`。
- **L313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vrndi_v),`.
  **L313 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vrndi_v),`。
- **L314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vrndiq_v),`.
  **L314 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vrndiq_v),`。
- **L315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vrshl_v, aarch64_neon_urshl, aarch64_neon_srshl, Add1ArgType | UnsignedAlts),`.
  **L315 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vrshl_v, aarch64_neon_urshl, aarch64_neon_srshl, Add1ArgType | UnsignedAlts),`。
- **L316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vrshlq_v, aarch64_neon_urshl, aarch64_neon_srshl, Add1ArgType | UnsignedAlts),`.
  **L316 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vrshlq_v, aarch64_neon_urshl, aarch64_neon_srshl, Add1ArgType | UnsignedAlts),`。
- **L317 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vrshr_n_v, aarch64_neon_urshl, aarch64_neon_srshl, UnsignedAlts),`.
  **L317 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vrshr_n_v, aarch64_neon_urshl, aarch64_neon_srshl, UnsignedAlts),`。
- **L318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vrshrq_n_v, aarch64_neon_urshl, aarch64_neon_srshl, UnsignedAlts),`.
  **L318 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vrshrq_n_v, aarch64_neon_urshl, aarch64_neon_srshl, UnsignedAlts),`。
- **L319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vrsqrte_v, aarch64_neon_frsqrte, aarch64_neon_ursqrte, 0),`.
  **L319 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vrsqrte_v, aarch64_neon_frsqrte, aarch64_neon_ursqrte, 0),`。
- **L320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vrsqrteq_v, aarch64_neon_frsqrte, aarch64_neon_ursqrte, 0),`.
  **L320 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vrsqrteq_v, aarch64_neon_frsqrte, aarch64_neon_ursqrte, 0),`。

### Lines 321-340

````cpp
  NEONMAP1(vrsqrts_v, aarch64_neon_frsqrts, Add1ArgType),
  NEONMAP1(vrsqrtsq_v, aarch64_neon_frsqrts, Add1ArgType),
  NEONMAP1(vrsubhn_v, aarch64_neon_rsubhn, Add1ArgType),
  NEONMAP1(vsha1su0q_u32, aarch64_crypto_sha1su0, 0),
  NEONMAP1(vsha1su1q_u32, aarch64_crypto_sha1su1, 0),
  NEONMAP1(vsha256h2q_u32, aarch64_crypto_sha256h2, 0),
  NEONMAP1(vsha256hq_u32, aarch64_crypto_sha256h, 0),
  NEONMAP1(vsha256su0q_u32, aarch64_crypto_sha256su0, 0),
  NEONMAP1(vsha256su1q_u32, aarch64_crypto_sha256su1, 0),
  NEONMAP1(vsha512h2q_u64, aarch64_crypto_sha512h2, 0),
  NEONMAP1(vsha512hq_u64, aarch64_crypto_sha512h, 0),
  NEONMAP1(vsha512su0q_u64, aarch64_crypto_sha512su0, 0),
  NEONMAP1(vsha512su1q_u64, aarch64_crypto_sha512su1, 0),
  NEONMAP0(vshl_n_v),
  NEONMAP2(vshl_v, aarch64_neon_ushl, aarch64_neon_sshl, Add1ArgType | UnsignedAlts),
  NEONMAP0(vshll_n_v),
  NEONMAP0(vshlq_n_v),
  NEONMAP2(vshlq_v, aarch64_neon_ushl, aarch64_neon_sshl, Add1ArgType | UnsignedAlts),
  NEONMAP0(vshr_n_v),
  NEONMAP0(vshrn_n_v),
````
- **L321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrsqrts_v, aarch64_neon_frsqrts, Add1ArgType),`.
  **L321 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrsqrts_v, aarch64_neon_frsqrts, Add1ArgType),`。
- **L322 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrsqrtsq_v, aarch64_neon_frsqrts, Add1ArgType),`.
  **L322 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrsqrtsq_v, aarch64_neon_frsqrts, Add1ArgType),`。
- **L323 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrsubhn_v, aarch64_neon_rsubhn, Add1ArgType),`.
  **L323 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrsubhn_v, aarch64_neon_rsubhn, Add1ArgType),`。
- **L324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsha1su0q_u32, aarch64_crypto_sha1su0, 0),`.
  **L324 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsha1su0q_u32, aarch64_crypto_sha1su0, 0),`。
- **L325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsha1su1q_u32, aarch64_crypto_sha1su1, 0),`.
  **L325 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsha1su1q_u32, aarch64_crypto_sha1su1, 0),`。
- **L326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsha256h2q_u32, aarch64_crypto_sha256h2, 0),`.
  **L326 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsha256h2q_u32, aarch64_crypto_sha256h2, 0),`。
- **L327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsha256hq_u32, aarch64_crypto_sha256h, 0),`.
  **L327 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsha256hq_u32, aarch64_crypto_sha256h, 0),`。
- **L328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsha256su0q_u32, aarch64_crypto_sha256su0, 0),`.
  **L328 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsha256su0q_u32, aarch64_crypto_sha256su0, 0),`。
- **L329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsha256su1q_u32, aarch64_crypto_sha256su1, 0),`.
  **L329 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsha256su1q_u32, aarch64_crypto_sha256su1, 0),`。
- **L330 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsha512h2q_u64, aarch64_crypto_sha512h2, 0),`.
  **L330 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsha512h2q_u64, aarch64_crypto_sha512h2, 0),`。
- **L331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsha512hq_u64, aarch64_crypto_sha512h, 0),`.
  **L331 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsha512hq_u64, aarch64_crypto_sha512h, 0),`。
- **L332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsha512su0q_u64, aarch64_crypto_sha512su0, 0),`.
  **L332 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsha512su0q_u64, aarch64_crypto_sha512su0, 0),`。
- **L333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsha512su1q_u64, aarch64_crypto_sha512su1, 0),`.
  **L333 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsha512su1q_u64, aarch64_crypto_sha512su1, 0),`。
- **L334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vshl_n_v),`.
  **L334 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vshl_n_v),`。
- **L335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vshl_v, aarch64_neon_ushl, aarch64_neon_sshl, Add1ArgType | UnsignedAlts),`.
  **L335 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vshl_v, aarch64_neon_ushl, aarch64_neon_sshl, Add1ArgType | UnsignedAlts),`。
- **L336 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vshll_n_v),`.
  **L336 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vshll_n_v),`。
- **L337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vshlq_n_v),`.
  **L337 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vshlq_n_v),`。
- **L338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP2(vshlq_v, aarch64_neon_ushl, aarch64_neon_sshl, Add1ArgType | UnsignedAlts),`.
  **L338 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP2(vshlq_v, aarch64_neon_ushl, aarch64_neon_sshl, Add1ArgType | UnsignedAlts),`。
- **L339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vshr_n_v),`.
  **L339 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vshr_n_v),`。
- **L340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vshrn_n_v),`.
  **L340 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vshrn_n_v),`。

### Lines 341-360

````cpp
  NEONMAP0(vshrq_n_v),
  NEONMAP1(vsm3partw1q_u32, aarch64_crypto_sm3partw1, 0),
  NEONMAP1(vsm3partw2q_u32, aarch64_crypto_sm3partw2, 0),
  NEONMAP1(vsm3ss1q_u32, aarch64_crypto_sm3ss1, 0),
  NEONMAP1(vsm3tt1aq_u32, aarch64_crypto_sm3tt1a, 0),
  NEONMAP1(vsm3tt1bq_u32, aarch64_crypto_sm3tt1b, 0),
  NEONMAP1(vsm3tt2aq_u32, aarch64_crypto_sm3tt2a, 0),
  NEONMAP1(vsm3tt2bq_u32, aarch64_crypto_sm3tt2b, 0),
  NEONMAP1(vsm4ekeyq_u32, aarch64_crypto_sm4ekey, 0),
  NEONMAP1(vsm4eq_u32, aarch64_crypto_sm4e, 0),
  NEONMAP1(vst1_x2_v, aarch64_neon_st1x2, 0),
  NEONMAP1(vst1_x3_v, aarch64_neon_st1x3, 0),
  NEONMAP1(vst1_x4_v, aarch64_neon_st1x4, 0),
  NEONMAP1(vst1q_x2_v, aarch64_neon_st1x2, 0),
  NEONMAP1(vst1q_x3_v, aarch64_neon_st1x3, 0),
  NEONMAP1(vst1q_x4_v, aarch64_neon_st1x4, 0),
  NEONMAP0(vsubhn_v),
  NEONMAP0(vtst_v),
  NEONMAP0(vtstq_v),
  NEONMAP1(vusdot_s32, aarch64_neon_usdot, 0),
````
- **L341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vshrq_n_v),`.
  **L341 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vshrq_n_v),`。
- **L342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsm3partw1q_u32, aarch64_crypto_sm3partw1, 0),`.
  **L342 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsm3partw1q_u32, aarch64_crypto_sm3partw1, 0),`。
- **L343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsm3partw2q_u32, aarch64_crypto_sm3partw2, 0),`.
  **L343 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsm3partw2q_u32, aarch64_crypto_sm3partw2, 0),`。
- **L344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsm3ss1q_u32, aarch64_crypto_sm3ss1, 0),`.
  **L344 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsm3ss1q_u32, aarch64_crypto_sm3ss1, 0),`。
- **L345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsm3tt1aq_u32, aarch64_crypto_sm3tt1a, 0),`.
  **L345 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsm3tt1aq_u32, aarch64_crypto_sm3tt1a, 0),`。
- **L346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsm3tt1bq_u32, aarch64_crypto_sm3tt1b, 0),`.
  **L346 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsm3tt1bq_u32, aarch64_crypto_sm3tt1b, 0),`。
- **L347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsm3tt2aq_u32, aarch64_crypto_sm3tt2a, 0),`.
  **L347 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsm3tt2aq_u32, aarch64_crypto_sm3tt2a, 0),`。
- **L348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsm3tt2bq_u32, aarch64_crypto_sm3tt2b, 0),`.
  **L348 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsm3tt2bq_u32, aarch64_crypto_sm3tt2b, 0),`。
- **L349 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsm4ekeyq_u32, aarch64_crypto_sm4ekey, 0),`.
  **L349 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsm4ekeyq_u32, aarch64_crypto_sm4ekey, 0),`。
- **L350 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsm4eq_u32, aarch64_crypto_sm4e, 0),`.
  **L350 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsm4eq_u32, aarch64_crypto_sm4e, 0),`。
- **L351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vst1_x2_v, aarch64_neon_st1x2, 0),`.
  **L351 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vst1_x2_v, aarch64_neon_st1x2, 0),`。
- **L352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vst1_x3_v, aarch64_neon_st1x3, 0),`.
  **L352 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vst1_x3_v, aarch64_neon_st1x3, 0),`。
- **L353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vst1_x4_v, aarch64_neon_st1x4, 0),`.
  **L353 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vst1_x4_v, aarch64_neon_st1x4, 0),`。
- **L354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vst1q_x2_v, aarch64_neon_st1x2, 0),`.
  **L354 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vst1q_x2_v, aarch64_neon_st1x2, 0),`。
- **L355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vst1q_x3_v, aarch64_neon_st1x3, 0),`.
  **L355 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vst1q_x3_v, aarch64_neon_st1x3, 0),`。
- **L356 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vst1q_x4_v, aarch64_neon_st1x4, 0),`.
  **L356 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vst1q_x4_v, aarch64_neon_st1x4, 0),`。
- **L357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vsubhn_v),`.
  **L357 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vsubhn_v),`。
- **L358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vtst_v),`.
  **L358 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vtst_v),`。
- **L359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vtstq_v),`.
  **L359 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vtstq_v),`。
- **L360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vusdot_s32, aarch64_neon_usdot, 0),`.
  **L360 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vusdot_s32, aarch64_neon_usdot, 0),`。

### Lines 361-380

````cpp
  NEONMAP1(vusdotq_s32, aarch64_neon_usdot, 0),
  NEONMAP1(vusmmlaq_s32, aarch64_neon_usmmla, 0),
  NEONMAP1(vxarq_u64, aarch64_crypto_xar, 0),
};


// Single-Instruction-Single-Data (SISD) intrinsics.
//
// The name is somewhat misleading: not all intrinsics in this table are
// strictly SISD. While many builtins operate on scalars,
//   * some take vector operands (e.g. reduction builtins such as
//     `vminvq_u16` and `vaddvq_s32`), and
//   * some take both scalar and vector operands (e.g. crypto builtins
//     such as `vsha1cq_u32`).
//
// TODO: Either rename this table to better reflect its contents, or
// restrict it to true SISD intrinsics only.
const inline ARMVectorIntrinsicInfo AArch64SISDIntrinsicMap[] = {
  NEONMAP1(vabdd_f64, aarch64_sisd_fabd, Add1ArgType),
  NEONMAP1(vabds_f32, aarch64_sisd_fabd, Add1ArgType),
````
- **L361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vusdotq_s32, aarch64_neon_usdot, 0),`.
  **L361 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vusdotq_s32, aarch64_neon_usdot, 0),`。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vusmmlaq_s32, aarch64_neon_usmmla, 0),`.
  **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vusmmlaq_s32, aarch64_neon_usmmla, 0),`。
- **L363 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vxarq_u64, aarch64_crypto_xar, 0),`.
  **L363 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vxarq_u64, aarch64_crypto_xar, 0),`。
- **L364 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L364 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L365 EN**: Blank line separating nearby declarations or logic blocks.
  **L365 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L366 EN**: Blank line separating nearby declarations or logic blocks.
  **L366 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L367 EN**: Comment explains nearby logic, constraints, or intent: `Single-Instruction-Single-Data (SISD) intrinsics.`.
  **L367 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Single-Instruction-Single-Data (SISD) intrinsics.`。
- **L368 EN**: Separator comment used for visual grouping.
  **L368 CN**: 用于视觉分组的分隔注释。
- **L369 EN**: Comment explains nearby logic, constraints, or intent: `The name is somewhat misleading: not all intrinsics in this table are`.
  **L369 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The name is somewhat misleading: not all intrinsics in this table are`。
- **L370 EN**: Comment explains nearby logic, constraints, or intent: `strictly SISD. While many builtins operate on scalars,`.
  **L370 CN**: 注释解释附近代码的逻辑、约束或设计意图：`strictly SISD. While many builtins operate on scalars,`。
- **L371 EN**: Comment explains nearby logic, constraints, or intent: `some take vector operands (e.g. reduction builtins such as`.
  **L371 CN**: 注释解释附近代码的逻辑、约束或设计意图：`some take vector operands (e.g. reduction builtins such as`。
- **L372 EN**: Comment explains nearby logic, constraints, or intent: ``vminvq_u16` and `vaddvq_s32`), and`.
  **L372 CN**: 注释解释附近代码的逻辑、约束或设计意图：``vminvq_u16` and `vaddvq_s32`), and`。
- **L373 EN**: Comment explains nearby logic, constraints, or intent: `some take both scalar and vector operands (e.g. crypto builtins`.
  **L373 CN**: 注释解释附近代码的逻辑、约束或设计意图：`some take both scalar and vector operands (e.g. crypto builtins`。
- **L374 EN**: Comment explains nearby logic, constraints, or intent: `such as `vsha1cq_u32`).`.
  **L374 CN**: 注释解释附近代码的逻辑、约束或设计意图：`such as `vsha1cq_u32`).`。
- **L375 EN**: Separator comment used for visual grouping.
  **L375 CN**: 用于视觉分组的分隔注释。
- **L376 EN**: Comment records a pending task or caution: `TODO: Either rename this table to better reflect its contents, or`.
  **L376 CN**: 注释记录待办事项或注意点：`TODO: Either rename this table to better reflect its contents, or`。
- **L377 EN**: Comment explains nearby logic, constraints, or intent: `restrict it to true SISD intrinsics only.`.
  **L377 CN**: 注释解释附近代码的逻辑、约束或设计意图：`restrict it to true SISD intrinsics only.`。
- **L378 EN**: Continues the surrounding expression or declaration: `const inline ARMVectorIntrinsicInfo AArch64SISDIntrinsicMap[] = {`.
  **L378 CN**: 继续构造周围的表达式或声明：`const inline ARMVectorIntrinsicInfo AArch64SISDIntrinsicMap[] = {`。
- **L379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vabdd_f64, aarch64_sisd_fabd, Add1ArgType),`.
  **L379 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vabdd_f64, aarch64_sisd_fabd, Add1ArgType),`。
- **L380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vabds_f32, aarch64_sisd_fabd, Add1ArgType),`.
  **L380 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vabds_f32, aarch64_sisd_fabd, Add1ArgType),`。

### Lines 381-400

````cpp
  NEONMAP1(vabsd_s64, aarch64_neon_abs, Add1ArgType),
  NEONMAP1(vaddlv_s32, aarch64_neon_saddlv, AddRetType | Add1ArgType),
  NEONMAP1(vaddlv_u32, aarch64_neon_uaddlv, AddRetType | Add1ArgType),
  NEONMAP1(vaddlvq_s32, aarch64_neon_saddlv, AddRetType | Add1ArgType),
  NEONMAP1(vaddlvq_u32, aarch64_neon_uaddlv, AddRetType | Add1ArgType),
  NEONMAP1(vaddv_f32, aarch64_neon_faddv, AddRetType | Add1ArgType),
  NEONMAP1(vaddv_s16, vector_reduce_add, Add1ArgType),
  NEONMAP1(vaddv_s32, vector_reduce_add, Add1ArgType),
  NEONMAP1(vaddv_s8, vector_reduce_add, Add1ArgType),
  NEONMAP1(vaddv_u16, vector_reduce_add, Add1ArgType),
  NEONMAP1(vaddv_u32, vector_reduce_add, Add1ArgType),
  NEONMAP1(vaddv_u8, vector_reduce_add, Add1ArgType),
  NEONMAP1(vaddvq_f32, aarch64_neon_faddv, AddRetType | Add1ArgType),
  NEONMAP1(vaddvq_f64, aarch64_neon_faddv, AddRetType | Add1ArgType),
  NEONMAP1(vaddvq_s16, vector_reduce_add, Add1ArgType),
  NEONMAP1(vaddvq_s32, vector_reduce_add, Add1ArgType),
  NEONMAP1(vaddvq_s64, vector_reduce_add, Add1ArgType),
  NEONMAP1(vaddvq_s8, vector_reduce_add, Add1ArgType),
  NEONMAP1(vaddvq_u16, vector_reduce_add, Add1ArgType),
  NEONMAP1(vaddvq_u32, vector_reduce_add, Add1ArgType),
````
- **L381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vabsd_s64, aarch64_neon_abs, Add1ArgType),`.
  **L381 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vabsd_s64, aarch64_neon_abs, Add1ArgType),`。
- **L382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddlv_s32, aarch64_neon_saddlv, AddRetType | Add1ArgType),`.
  **L382 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddlv_s32, aarch64_neon_saddlv, AddRetType | Add1ArgType),`。
- **L383 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddlv_u32, aarch64_neon_uaddlv, AddRetType | Add1ArgType),`.
  **L383 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddlv_u32, aarch64_neon_uaddlv, AddRetType | Add1ArgType),`。
- **L384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddlvq_s32, aarch64_neon_saddlv, AddRetType | Add1ArgType),`.
  **L384 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddlvq_s32, aarch64_neon_saddlv, AddRetType | Add1ArgType),`。
- **L385 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddlvq_u32, aarch64_neon_uaddlv, AddRetType | Add1ArgType),`.
  **L385 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddlvq_u32, aarch64_neon_uaddlv, AddRetType | Add1ArgType),`。
- **L386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddv_f32, aarch64_neon_faddv, AddRetType | Add1ArgType),`.
  **L386 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddv_f32, aarch64_neon_faddv, AddRetType | Add1ArgType),`。
- **L387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddv_s16, vector_reduce_add, Add1ArgType),`.
  **L387 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddv_s16, vector_reduce_add, Add1ArgType),`。
- **L388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddv_s32, vector_reduce_add, Add1ArgType),`.
  **L388 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddv_s32, vector_reduce_add, Add1ArgType),`。
- **L389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddv_s8, vector_reduce_add, Add1ArgType),`.
  **L389 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddv_s8, vector_reduce_add, Add1ArgType),`。
- **L390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddv_u16, vector_reduce_add, Add1ArgType),`.
  **L390 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddv_u16, vector_reduce_add, Add1ArgType),`。
- **L391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddv_u32, vector_reduce_add, Add1ArgType),`.
  **L391 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddv_u32, vector_reduce_add, Add1ArgType),`。
- **L392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddv_u8, vector_reduce_add, Add1ArgType),`.
  **L392 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddv_u8, vector_reduce_add, Add1ArgType),`。
- **L393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddvq_f32, aarch64_neon_faddv, AddRetType | Add1ArgType),`.
  **L393 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddvq_f32, aarch64_neon_faddv, AddRetType | Add1ArgType),`。
- **L394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddvq_f64, aarch64_neon_faddv, AddRetType | Add1ArgType),`.
  **L394 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddvq_f64, aarch64_neon_faddv, AddRetType | Add1ArgType),`。
- **L395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddvq_s16, vector_reduce_add, Add1ArgType),`.
  **L395 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddvq_s16, vector_reduce_add, Add1ArgType),`。
- **L396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddvq_s32, vector_reduce_add, Add1ArgType),`.
  **L396 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddvq_s32, vector_reduce_add, Add1ArgType),`。
- **L397 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddvq_s64, vector_reduce_add, Add1ArgType),`.
  **L397 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddvq_s64, vector_reduce_add, Add1ArgType),`。
- **L398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddvq_s8, vector_reduce_add, Add1ArgType),`.
  **L398 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddvq_s8, vector_reduce_add, Add1ArgType),`。
- **L399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddvq_u16, vector_reduce_add, Add1ArgType),`.
  **L399 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddvq_u16, vector_reduce_add, Add1ArgType),`。
- **L400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddvq_u32, vector_reduce_add, Add1ArgType),`.
  **L400 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddvq_u32, vector_reduce_add, Add1ArgType),`。

### Lines 401-420

````cpp
  NEONMAP1(vaddvq_u64, vector_reduce_add, Add1ArgType),
  NEONMAP1(vaddvq_u8, vector_reduce_add, Add1ArgType),
  NEONMAP1(vcaged_f64, aarch64_neon_facge, AddRetType | Add1ArgType),
  NEONMAP1(vcages_f32, aarch64_neon_facge, AddRetType | Add1ArgType),
  NEONMAP1(vcagtd_f64, aarch64_neon_facgt, AddRetType | Add1ArgType),
  NEONMAP1(vcagts_f32, aarch64_neon_facgt, AddRetType | Add1ArgType),
  NEONMAP1(vcaled_f64, aarch64_neon_facge, AddRetType | Add1ArgType),
  NEONMAP1(vcales_f32, aarch64_neon_facge, AddRetType | Add1ArgType),
  NEONMAP1(vcaltd_f64, aarch64_neon_facgt, AddRetType | Add1ArgType),
  NEONMAP1(vcalts_f32, aarch64_neon_facgt, AddRetType | Add1ArgType),
  NEONMAP1(vcvtad_s32_f64, aarch64_neon_fcvtas, AddRetType | Add1ArgType),
  NEONMAP1(vcvtad_s64_f64, aarch64_neon_fcvtas, AddRetType | Add1ArgType),
  NEONMAP1(vcvtad_u32_f64, aarch64_neon_fcvtau, AddRetType | Add1ArgType),
  NEONMAP1(vcvtad_u64_f64, aarch64_neon_fcvtau, AddRetType | Add1ArgType),
  NEONMAP1(vcvtas_s32_f32, aarch64_neon_fcvtas, AddRetType | Add1ArgType),
  NEONMAP1(vcvtas_s64_f32, aarch64_neon_fcvtas, AddRetType | Add1ArgType),
  NEONMAP1(vcvtas_u32_f32, aarch64_neon_fcvtau, AddRetType | Add1ArgType),
  NEONMAP1(vcvtas_u64_f32, aarch64_neon_fcvtau, AddRetType | Add1ArgType),
  NEONMAP1(vcvtd_n_f64_s64, aarch64_neon_vcvtfxs2fp, AddRetType | Add1ArgType),
  NEONMAP1(vcvtd_n_f64_u64, aarch64_neon_vcvtfxu2fp, AddRetType | Add1ArgType),
````
- **L401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddvq_u64, vector_reduce_add, Add1ArgType),`.
  **L401 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddvq_u64, vector_reduce_add, Add1ArgType),`。
- **L402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vaddvq_u8, vector_reduce_add, Add1ArgType),`.
  **L402 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vaddvq_u8, vector_reduce_add, Add1ArgType),`。
- **L403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcaged_f64, aarch64_neon_facge, AddRetType | Add1ArgType),`.
  **L403 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcaged_f64, aarch64_neon_facge, AddRetType | Add1ArgType),`。
- **L404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcages_f32, aarch64_neon_facge, AddRetType | Add1ArgType),`.
  **L404 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcages_f32, aarch64_neon_facge, AddRetType | Add1ArgType),`。
- **L405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcagtd_f64, aarch64_neon_facgt, AddRetType | Add1ArgType),`.
  **L405 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcagtd_f64, aarch64_neon_facgt, AddRetType | Add1ArgType),`。
- **L406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcagts_f32, aarch64_neon_facgt, AddRetType | Add1ArgType),`.
  **L406 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcagts_f32, aarch64_neon_facgt, AddRetType | Add1ArgType),`。
- **L407 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcaled_f64, aarch64_neon_facge, AddRetType | Add1ArgType),`.
  **L407 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcaled_f64, aarch64_neon_facge, AddRetType | Add1ArgType),`。
- **L408 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcales_f32, aarch64_neon_facge, AddRetType | Add1ArgType),`.
  **L408 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcales_f32, aarch64_neon_facge, AddRetType | Add1ArgType),`。
- **L409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcaltd_f64, aarch64_neon_facgt, AddRetType | Add1ArgType),`.
  **L409 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcaltd_f64, aarch64_neon_facgt, AddRetType | Add1ArgType),`。
- **L410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcalts_f32, aarch64_neon_facgt, AddRetType | Add1ArgType),`.
  **L410 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcalts_f32, aarch64_neon_facgt, AddRetType | Add1ArgType),`。
- **L411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtad_s32_f64, aarch64_neon_fcvtas, AddRetType | Add1ArgType),`.
  **L411 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtad_s32_f64, aarch64_neon_fcvtas, AddRetType | Add1ArgType),`。
- **L412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtad_s64_f64, aarch64_neon_fcvtas, AddRetType | Add1ArgType),`.
  **L412 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtad_s64_f64, aarch64_neon_fcvtas, AddRetType | Add1ArgType),`。
- **L413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtad_u32_f64, aarch64_neon_fcvtau, AddRetType | Add1ArgType),`.
  **L413 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtad_u32_f64, aarch64_neon_fcvtau, AddRetType | Add1ArgType),`。
- **L414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtad_u64_f64, aarch64_neon_fcvtau, AddRetType | Add1ArgType),`.
  **L414 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtad_u64_f64, aarch64_neon_fcvtau, AddRetType | Add1ArgType),`。
- **L415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtas_s32_f32, aarch64_neon_fcvtas, AddRetType | Add1ArgType),`.
  **L415 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtas_s32_f32, aarch64_neon_fcvtas, AddRetType | Add1ArgType),`。
- **L416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtas_s64_f32, aarch64_neon_fcvtas, AddRetType | Add1ArgType),`.
  **L416 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtas_s64_f32, aarch64_neon_fcvtas, AddRetType | Add1ArgType),`。
- **L417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtas_u32_f32, aarch64_neon_fcvtau, AddRetType | Add1ArgType),`.
  **L417 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtas_u32_f32, aarch64_neon_fcvtau, AddRetType | Add1ArgType),`。
- **L418 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtas_u64_f32, aarch64_neon_fcvtau, AddRetType | Add1ArgType),`.
  **L418 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtas_u64_f32, aarch64_neon_fcvtau, AddRetType | Add1ArgType),`。
- **L419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtd_n_f64_s64, aarch64_neon_vcvtfxs2fp, AddRetType | Add1ArgType),`.
  **L419 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtd_n_f64_s64, aarch64_neon_vcvtfxs2fp, AddRetType | Add1ArgType),`。
- **L420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtd_n_f64_u64, aarch64_neon_vcvtfxu2fp, AddRetType | Add1ArgType),`.
  **L420 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtd_n_f64_u64, aarch64_neon_vcvtfxu2fp, AddRetType | Add1ArgType),`。

### Lines 421-440

````cpp
  NEONMAP1(vcvtd_n_s64_f64, aarch64_neon_vcvtfp2fxs, AddRetType | Add1ArgType),
  NEONMAP1(vcvtd_n_u64_f64, aarch64_neon_vcvtfp2fxu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtd_s32_f64, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),
  NEONMAP1(vcvtd_s64_f64, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),
  NEONMAP1(vcvtd_u32_f64, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtd_u64_f64, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),
  NEONMAP0(vcvth_bf16_f32),
  NEONMAP1(vcvtmd_s32_f64, aarch64_neon_fcvtms, AddRetType | Add1ArgType),
  NEONMAP1(vcvtmd_s64_f64, aarch64_neon_fcvtms, AddRetType | Add1ArgType),
  NEONMAP1(vcvtmd_u32_f64, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtmd_u64_f64, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtms_s32_f32, aarch64_neon_fcvtms, AddRetType | Add1ArgType),
  NEONMAP1(vcvtms_s64_f32, aarch64_neon_fcvtms, AddRetType | Add1ArgType),
  NEONMAP1(vcvtms_u32_f32, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtms_u64_f32, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtnd_s32_f64, aarch64_neon_fcvtns, AddRetType | Add1ArgType),
  NEONMAP1(vcvtnd_s64_f64, aarch64_neon_fcvtns, AddRetType | Add1ArgType),
  NEONMAP1(vcvtnd_u32_f64, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtnd_u64_f64, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtns_s32_f32, aarch64_neon_fcvtns, AddRetType | Add1ArgType),
````
- **L421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtd_n_s64_f64, aarch64_neon_vcvtfp2fxs, AddRetType | Add1ArgType),`.
  **L421 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtd_n_s64_f64, aarch64_neon_vcvtfp2fxs, AddRetType | Add1ArgType),`。
- **L422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtd_n_u64_f64, aarch64_neon_vcvtfp2fxu, AddRetType | Add1ArgType),`.
  **L422 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtd_n_u64_f64, aarch64_neon_vcvtfp2fxu, AddRetType | Add1ArgType),`。
- **L423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtd_s32_f64, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`.
  **L423 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtd_s32_f64, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`。
- **L424 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtd_s64_f64, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`.
  **L424 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtd_s64_f64, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`。
- **L425 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtd_u32_f64, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`.
  **L425 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtd_u32_f64, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`。
- **L426 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtd_u64_f64, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`.
  **L426 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtd_u64_f64, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`。
- **L427 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP0(vcvth_bf16_f32),`.
  **L427 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP0(vcvth_bf16_f32),`。
- **L428 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtmd_s32_f64, aarch64_neon_fcvtms, AddRetType | Add1ArgType),`.
  **L428 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtmd_s32_f64, aarch64_neon_fcvtms, AddRetType | Add1ArgType),`。
- **L429 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtmd_s64_f64, aarch64_neon_fcvtms, AddRetType | Add1ArgType),`.
  **L429 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtmd_s64_f64, aarch64_neon_fcvtms, AddRetType | Add1ArgType),`。
- **L430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtmd_u32_f64, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),`.
  **L430 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtmd_u32_f64, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),`。
- **L431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtmd_u64_f64, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),`.
  **L431 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtmd_u64_f64, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),`。
- **L432 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtms_s32_f32, aarch64_neon_fcvtms, AddRetType | Add1ArgType),`.
  **L432 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtms_s32_f32, aarch64_neon_fcvtms, AddRetType | Add1ArgType),`。
- **L433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtms_s64_f32, aarch64_neon_fcvtms, AddRetType | Add1ArgType),`.
  **L433 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtms_s64_f32, aarch64_neon_fcvtms, AddRetType | Add1ArgType),`。
- **L434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtms_u32_f32, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),`.
  **L434 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtms_u32_f32, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),`。
- **L435 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtms_u64_f32, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),`.
  **L435 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtms_u64_f32, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),`。
- **L436 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtnd_s32_f64, aarch64_neon_fcvtns, AddRetType | Add1ArgType),`.
  **L436 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtnd_s32_f64, aarch64_neon_fcvtns, AddRetType | Add1ArgType),`。
- **L437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtnd_s64_f64, aarch64_neon_fcvtns, AddRetType | Add1ArgType),`.
  **L437 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtnd_s64_f64, aarch64_neon_fcvtns, AddRetType | Add1ArgType),`。
- **L438 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtnd_u32_f64, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),`.
  **L438 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtnd_u32_f64, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),`。
- **L439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtnd_u64_f64, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),`.
  **L439 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtnd_u64_f64, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),`。
- **L440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtns_s32_f32, aarch64_neon_fcvtns, AddRetType | Add1ArgType),`.
  **L440 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtns_s32_f32, aarch64_neon_fcvtns, AddRetType | Add1ArgType),`。

### Lines 441-460

````cpp
  NEONMAP1(vcvtns_s64_f32, aarch64_neon_fcvtns, AddRetType | Add1ArgType),
  NEONMAP1(vcvtns_u32_f32, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtns_u64_f32, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtpd_s32_f64, aarch64_neon_fcvtps, AddRetType | Add1ArgType),
  NEONMAP1(vcvtpd_s64_f64, aarch64_neon_fcvtps, AddRetType | Add1ArgType),
  NEONMAP1(vcvtpd_u32_f64, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtpd_u64_f64, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtps_s32_f32, aarch64_neon_fcvtps, AddRetType | Add1ArgType),
  NEONMAP1(vcvtps_s64_f32, aarch64_neon_fcvtps, AddRetType | Add1ArgType),
  NEONMAP1(vcvtps_u32_f32, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtps_u64_f32, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),
  NEONMAP1(vcvts_n_f32_s32, aarch64_neon_vcvtfxs2fp, AddRetType | Add1ArgType),
  NEONMAP1(vcvts_n_f32_u32, aarch64_neon_vcvtfxu2fp, AddRetType | Add1ArgType),
  NEONMAP1(vcvts_n_s32_f32, aarch64_neon_vcvtfp2fxs, AddRetType | Add1ArgType),
  NEONMAP1(vcvts_n_u32_f32, aarch64_neon_vcvtfp2fxu, AddRetType | Add1ArgType),
  NEONMAP1(vcvts_s32_f32, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),
  NEONMAP1(vcvts_s64_f32, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),
  NEONMAP1(vcvts_u32_f32, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),
  NEONMAP1(vcvts_u64_f32, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtxd_f32_f64, aarch64_sisd_fcvtxn, 0),
````
- **L441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtns_s64_f32, aarch64_neon_fcvtns, AddRetType | Add1ArgType),`.
  **L441 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtns_s64_f32, aarch64_neon_fcvtns, AddRetType | Add1ArgType),`。
- **L442 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtns_u32_f32, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),`.
  **L442 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtns_u32_f32, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),`。
- **L443 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtns_u64_f32, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),`.
  **L443 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtns_u64_f32, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),`。
- **L444 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtpd_s32_f64, aarch64_neon_fcvtps, AddRetType | Add1ArgType),`.
  **L444 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtpd_s32_f64, aarch64_neon_fcvtps, AddRetType | Add1ArgType),`。
- **L445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtpd_s64_f64, aarch64_neon_fcvtps, AddRetType | Add1ArgType),`.
  **L445 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtpd_s64_f64, aarch64_neon_fcvtps, AddRetType | Add1ArgType),`。
- **L446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtpd_u32_f64, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),`.
  **L446 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtpd_u32_f64, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),`。
- **L447 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtpd_u64_f64, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),`.
  **L447 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtpd_u64_f64, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),`。
- **L448 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtps_s32_f32, aarch64_neon_fcvtps, AddRetType | Add1ArgType),`.
  **L448 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtps_s32_f32, aarch64_neon_fcvtps, AddRetType | Add1ArgType),`。
- **L449 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtps_s64_f32, aarch64_neon_fcvtps, AddRetType | Add1ArgType),`.
  **L449 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtps_s64_f32, aarch64_neon_fcvtps, AddRetType | Add1ArgType),`。
- **L450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtps_u32_f32, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),`.
  **L450 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtps_u32_f32, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),`。
- **L451 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtps_u64_f32, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),`.
  **L451 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtps_u64_f32, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),`。
- **L452 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvts_n_f32_s32, aarch64_neon_vcvtfxs2fp, AddRetType | Add1ArgType),`.
  **L452 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvts_n_f32_s32, aarch64_neon_vcvtfxs2fp, AddRetType | Add1ArgType),`。
- **L453 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvts_n_f32_u32, aarch64_neon_vcvtfxu2fp, AddRetType | Add1ArgType),`.
  **L453 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvts_n_f32_u32, aarch64_neon_vcvtfxu2fp, AddRetType | Add1ArgType),`。
- **L454 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvts_n_s32_f32, aarch64_neon_vcvtfp2fxs, AddRetType | Add1ArgType),`.
  **L454 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvts_n_s32_f32, aarch64_neon_vcvtfp2fxs, AddRetType | Add1ArgType),`。
- **L455 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvts_n_u32_f32, aarch64_neon_vcvtfp2fxu, AddRetType | Add1ArgType),`.
  **L455 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvts_n_u32_f32, aarch64_neon_vcvtfp2fxu, AddRetType | Add1ArgType),`。
- **L456 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvts_s32_f32, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`.
  **L456 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvts_s32_f32, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`。
- **L457 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvts_s64_f32, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`.
  **L457 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvts_s64_f32, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`。
- **L458 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvts_u32_f32, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`.
  **L458 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvts_u32_f32, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`。
- **L459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvts_u64_f32, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`.
  **L459 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvts_u64_f32, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`。
- **L460 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtxd_f32_f64, aarch64_sisd_fcvtxn, 0),`.
  **L460 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtxd_f32_f64, aarch64_sisd_fcvtxn, 0),`。

### Lines 461-480

````cpp
  NEONMAP1(vmaxnmv_f32, aarch64_neon_fmaxnmv, AddRetType | Add1ArgType),
  NEONMAP1(vmaxnmvq_f32, aarch64_neon_fmaxnmv, AddRetType | Add1ArgType),
  NEONMAP1(vmaxnmvq_f64, aarch64_neon_fmaxnmv, AddRetType | Add1ArgType),
  NEONMAP1(vmaxv_f32, aarch64_neon_fmaxv, AddRetType | Add1ArgType),
  NEONMAP1(vmaxv_s16, vector_reduce_smax, Add1ArgType),
  NEONMAP1(vmaxv_s32, vector_reduce_smax, Add1ArgType),
  NEONMAP1(vmaxv_s8, vector_reduce_smax, Add1ArgType),
  NEONMAP1(vmaxv_u16, vector_reduce_umax, Add1ArgType),
  NEONMAP1(vmaxv_u32, vector_reduce_umax, Add1ArgType),
  NEONMAP1(vmaxv_u8, vector_reduce_umax, Add1ArgType),
  NEONMAP1(vmaxvq_f32, aarch64_neon_fmaxv, AddRetType | Add1ArgType),
  NEONMAP1(vmaxvq_f64, aarch64_neon_fmaxv, AddRetType | Add1ArgType),
  NEONMAP1(vmaxvq_s16, vector_reduce_smax, Add1ArgType),
  NEONMAP1(vmaxvq_s32, vector_reduce_smax, Add1ArgType),
  NEONMAP1(vmaxvq_s8, vector_reduce_smax, Add1ArgType),
  NEONMAP1(vmaxvq_u16, vector_reduce_umax, Add1ArgType),
  NEONMAP1(vmaxvq_u32, vector_reduce_umax, Add1ArgType),
  NEONMAP1(vmaxvq_u8, vector_reduce_umax, Add1ArgType),
  NEONMAP1(vminnmv_f32, aarch64_neon_fminnmv, AddRetType | Add1ArgType),
  NEONMAP1(vminnmvq_f32, aarch64_neon_fminnmv, AddRetType | Add1ArgType),
````
- **L461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxnmv_f32, aarch64_neon_fmaxnmv, AddRetType | Add1ArgType),`.
  **L461 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxnmv_f32, aarch64_neon_fmaxnmv, AddRetType | Add1ArgType),`。
- **L462 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxnmvq_f32, aarch64_neon_fmaxnmv, AddRetType | Add1ArgType),`.
  **L462 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxnmvq_f32, aarch64_neon_fmaxnmv, AddRetType | Add1ArgType),`。
- **L463 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxnmvq_f64, aarch64_neon_fmaxnmv, AddRetType | Add1ArgType),`.
  **L463 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxnmvq_f64, aarch64_neon_fmaxnmv, AddRetType | Add1ArgType),`。
- **L464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxv_f32, aarch64_neon_fmaxv, AddRetType | Add1ArgType),`.
  **L464 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxv_f32, aarch64_neon_fmaxv, AddRetType | Add1ArgType),`。
- **L465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxv_s16, vector_reduce_smax, Add1ArgType),`.
  **L465 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxv_s16, vector_reduce_smax, Add1ArgType),`。
- **L466 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxv_s32, vector_reduce_smax, Add1ArgType),`.
  **L466 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxv_s32, vector_reduce_smax, Add1ArgType),`。
- **L467 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxv_s8, vector_reduce_smax, Add1ArgType),`.
  **L467 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxv_s8, vector_reduce_smax, Add1ArgType),`。
- **L468 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxv_u16, vector_reduce_umax, Add1ArgType),`.
  **L468 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxv_u16, vector_reduce_umax, Add1ArgType),`。
- **L469 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxv_u32, vector_reduce_umax, Add1ArgType),`.
  **L469 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxv_u32, vector_reduce_umax, Add1ArgType),`。
- **L470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxv_u8, vector_reduce_umax, Add1ArgType),`.
  **L470 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxv_u8, vector_reduce_umax, Add1ArgType),`。
- **L471 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxvq_f32, aarch64_neon_fmaxv, AddRetType | Add1ArgType),`.
  **L471 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxvq_f32, aarch64_neon_fmaxv, AddRetType | Add1ArgType),`。
- **L472 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxvq_f64, aarch64_neon_fmaxv, AddRetType | Add1ArgType),`.
  **L472 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxvq_f64, aarch64_neon_fmaxv, AddRetType | Add1ArgType),`。
- **L473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxvq_s16, vector_reduce_smax, Add1ArgType),`.
  **L473 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxvq_s16, vector_reduce_smax, Add1ArgType),`。
- **L474 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxvq_s32, vector_reduce_smax, Add1ArgType),`.
  **L474 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxvq_s32, vector_reduce_smax, Add1ArgType),`。
- **L475 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxvq_s8, vector_reduce_smax, Add1ArgType),`.
  **L475 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxvq_s8, vector_reduce_smax, Add1ArgType),`。
- **L476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxvq_u16, vector_reduce_umax, Add1ArgType),`.
  **L476 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxvq_u16, vector_reduce_umax, Add1ArgType),`。
- **L477 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxvq_u32, vector_reduce_umax, Add1ArgType),`.
  **L477 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxvq_u32, vector_reduce_umax, Add1ArgType),`。
- **L478 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmaxvq_u8, vector_reduce_umax, Add1ArgType),`.
  **L478 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmaxvq_u8, vector_reduce_umax, Add1ArgType),`。
- **L479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminnmv_f32, aarch64_neon_fminnmv, AddRetType | Add1ArgType),`.
  **L479 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminnmv_f32, aarch64_neon_fminnmv, AddRetType | Add1ArgType),`。
- **L480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminnmvq_f32, aarch64_neon_fminnmv, AddRetType | Add1ArgType),`.
  **L480 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminnmvq_f32, aarch64_neon_fminnmv, AddRetType | Add1ArgType),`。

### Lines 481-500

````cpp
  NEONMAP1(vminnmvq_f64, aarch64_neon_fminnmv, AddRetType | Add1ArgType),
  NEONMAP1(vminv_f32, aarch64_neon_fminv, AddRetType | Add1ArgType),
  NEONMAP1(vminv_s16, vector_reduce_smin, Add1ArgType),
  NEONMAP1(vminv_s32, vector_reduce_smin, Add1ArgType),
  NEONMAP1(vminv_s8, vector_reduce_smin, Add1ArgType),
  NEONMAP1(vminv_u16, vector_reduce_umin, Add1ArgType),
  NEONMAP1(vminv_u32, vector_reduce_umin, Add1ArgType),
  NEONMAP1(vminv_u8, vector_reduce_umin, Add1ArgType),
  NEONMAP1(vminvq_f32, aarch64_neon_fminv, AddRetType | Add1ArgType),
  NEONMAP1(vminvq_f64, aarch64_neon_fminv, AddRetType | Add1ArgType),
  NEONMAP1(vminvq_s16, vector_reduce_smin, Add1ArgType),
  NEONMAP1(vminvq_s32, vector_reduce_smin, Add1ArgType),
  NEONMAP1(vminvq_s8, vector_reduce_smin, Add1ArgType),
  NEONMAP1(vminvq_u16, vector_reduce_umin, Add1ArgType),
  NEONMAP1(vminvq_u32, vector_reduce_umin, Add1ArgType),
  NEONMAP1(vminvq_u8, vector_reduce_umin, Add1ArgType),
  NEONMAP1(vmull_p64, aarch64_neon_pmull64, 0),
  NEONMAP1(vmulxd_f64, aarch64_neon_fmulx, Add1ArgType),
  NEONMAP1(vmulxs_f32, aarch64_neon_fmulx, Add1ArgType),
  NEONMAP1(vpaddd_s64, vector_reduce_add, Add1ArgType),
````
- **L481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminnmvq_f64, aarch64_neon_fminnmv, AddRetType | Add1ArgType),`.
  **L481 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminnmvq_f64, aarch64_neon_fminnmv, AddRetType | Add1ArgType),`。
- **L482 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminv_f32, aarch64_neon_fminv, AddRetType | Add1ArgType),`.
  **L482 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminv_f32, aarch64_neon_fminv, AddRetType | Add1ArgType),`。
- **L483 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminv_s16, vector_reduce_smin, Add1ArgType),`.
  **L483 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminv_s16, vector_reduce_smin, Add1ArgType),`。
- **L484 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminv_s32, vector_reduce_smin, Add1ArgType),`.
  **L484 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminv_s32, vector_reduce_smin, Add1ArgType),`。
- **L485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminv_s8, vector_reduce_smin, Add1ArgType),`.
  **L485 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminv_s8, vector_reduce_smin, Add1ArgType),`。
- **L486 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminv_u16, vector_reduce_umin, Add1ArgType),`.
  **L486 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminv_u16, vector_reduce_umin, Add1ArgType),`。
- **L487 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminv_u32, vector_reduce_umin, Add1ArgType),`.
  **L487 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminv_u32, vector_reduce_umin, Add1ArgType),`。
- **L488 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminv_u8, vector_reduce_umin, Add1ArgType),`.
  **L488 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminv_u8, vector_reduce_umin, Add1ArgType),`。
- **L489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminvq_f32, aarch64_neon_fminv, AddRetType | Add1ArgType),`.
  **L489 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminvq_f32, aarch64_neon_fminv, AddRetType | Add1ArgType),`。
- **L490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminvq_f64, aarch64_neon_fminv, AddRetType | Add1ArgType),`.
  **L490 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminvq_f64, aarch64_neon_fminv, AddRetType | Add1ArgType),`。
- **L491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminvq_s16, vector_reduce_smin, Add1ArgType),`.
  **L491 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminvq_s16, vector_reduce_smin, Add1ArgType),`。
- **L492 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminvq_s32, vector_reduce_smin, Add1ArgType),`.
  **L492 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminvq_s32, vector_reduce_smin, Add1ArgType),`。
- **L493 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminvq_s8, vector_reduce_smin, Add1ArgType),`.
  **L493 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminvq_s8, vector_reduce_smin, Add1ArgType),`。
- **L494 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminvq_u16, vector_reduce_umin, Add1ArgType),`.
  **L494 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminvq_u16, vector_reduce_umin, Add1ArgType),`。
- **L495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminvq_u32, vector_reduce_umin, Add1ArgType),`.
  **L495 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminvq_u32, vector_reduce_umin, Add1ArgType),`。
- **L496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vminvq_u8, vector_reduce_umin, Add1ArgType),`.
  **L496 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vminvq_u8, vector_reduce_umin, Add1ArgType),`。
- **L497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmull_p64, aarch64_neon_pmull64, 0),`.
  **L497 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmull_p64, aarch64_neon_pmull64, 0),`。
- **L498 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmulxd_f64, aarch64_neon_fmulx, Add1ArgType),`.
  **L498 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmulxd_f64, aarch64_neon_fmulx, Add1ArgType),`。
- **L499 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmulxs_f32, aarch64_neon_fmulx, Add1ArgType),`.
  **L499 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmulxs_f32, aarch64_neon_fmulx, Add1ArgType),`。
- **L500 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vpaddd_s64, vector_reduce_add, Add1ArgType),`.
  **L500 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vpaddd_s64, vector_reduce_add, Add1ArgType),`。

### Lines 501-520

````cpp
  NEONMAP1(vpaddd_u64, vector_reduce_add, Add1ArgType),
  NEONMAP1(vpmaxnmqd_f64, aarch64_neon_fmaxnmv, AddRetType | Add1ArgType),
  NEONMAP1(vpmaxnms_f32, aarch64_neon_fmaxnmv, AddRetType | Add1ArgType),
  NEONMAP1(vpmaxqd_f64, aarch64_neon_fmaxv, AddRetType | Add1ArgType),
  NEONMAP1(vpmaxs_f32, aarch64_neon_fmaxv, AddRetType | Add1ArgType),
  NEONMAP1(vpminnmqd_f64, aarch64_neon_fminnmv, AddRetType | Add1ArgType),
  NEONMAP1(vpminnms_f32, aarch64_neon_fminnmv, AddRetType | Add1ArgType),
  NEONMAP1(vpminqd_f64, aarch64_neon_fminv, AddRetType | Add1ArgType),
  NEONMAP1(vpmins_f32, aarch64_neon_fminv, AddRetType | Add1ArgType),
  NEONMAP1(vqabsb_s8, aarch64_neon_sqabs, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqabsd_s64, aarch64_neon_sqabs, Add1ArgType),
  NEONMAP1(vqabsh_s16, aarch64_neon_sqabs, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqabss_s32, aarch64_neon_sqabs, Add1ArgType),
  NEONMAP1(vqaddb_s8, aarch64_neon_sqadd, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqaddb_u8, aarch64_neon_uqadd, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqaddd_s64, aarch64_neon_sqadd, Add1ArgType),
  NEONMAP1(vqaddd_u64, aarch64_neon_uqadd, Add1ArgType),
  NEONMAP1(vqaddh_s16, aarch64_neon_sqadd, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqaddh_u16, aarch64_neon_uqadd, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqadds_s32, aarch64_neon_sqadd, Add1ArgType),
````
- **L501 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vpaddd_u64, vector_reduce_add, Add1ArgType),`.
  **L501 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vpaddd_u64, vector_reduce_add, Add1ArgType),`。
- **L502 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vpmaxnmqd_f64, aarch64_neon_fmaxnmv, AddRetType | Add1ArgType),`.
  **L502 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vpmaxnmqd_f64, aarch64_neon_fmaxnmv, AddRetType | Add1ArgType),`。
- **L503 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vpmaxnms_f32, aarch64_neon_fmaxnmv, AddRetType | Add1ArgType),`.
  **L503 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vpmaxnms_f32, aarch64_neon_fmaxnmv, AddRetType | Add1ArgType),`。
- **L504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vpmaxqd_f64, aarch64_neon_fmaxv, AddRetType | Add1ArgType),`.
  **L504 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vpmaxqd_f64, aarch64_neon_fmaxv, AddRetType | Add1ArgType),`。
- **L505 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vpmaxs_f32, aarch64_neon_fmaxv, AddRetType | Add1ArgType),`.
  **L505 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vpmaxs_f32, aarch64_neon_fmaxv, AddRetType | Add1ArgType),`。
- **L506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vpminnmqd_f64, aarch64_neon_fminnmv, AddRetType | Add1ArgType),`.
  **L506 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vpminnmqd_f64, aarch64_neon_fminnmv, AddRetType | Add1ArgType),`。
- **L507 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vpminnms_f32, aarch64_neon_fminnmv, AddRetType | Add1ArgType),`.
  **L507 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vpminnms_f32, aarch64_neon_fminnmv, AddRetType | Add1ArgType),`。
- **L508 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vpminqd_f64, aarch64_neon_fminv, AddRetType | Add1ArgType),`.
  **L508 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vpminqd_f64, aarch64_neon_fminv, AddRetType | Add1ArgType),`。
- **L509 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vpmins_f32, aarch64_neon_fminv, AddRetType | Add1ArgType),`.
  **L509 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vpmins_f32, aarch64_neon_fminv, AddRetType | Add1ArgType),`。
- **L510 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqabsb_s8, aarch64_neon_sqabs, Vectorize1ArgType | Use64BitVectors),`.
  **L510 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqabsb_s8, aarch64_neon_sqabs, Vectorize1ArgType | Use64BitVectors),`。
- **L511 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqabsd_s64, aarch64_neon_sqabs, Add1ArgType),`.
  **L511 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqabsd_s64, aarch64_neon_sqabs, Add1ArgType),`。
- **L512 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqabsh_s16, aarch64_neon_sqabs, Vectorize1ArgType | Use64BitVectors),`.
  **L512 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqabsh_s16, aarch64_neon_sqabs, Vectorize1ArgType | Use64BitVectors),`。
- **L513 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqabss_s32, aarch64_neon_sqabs, Add1ArgType),`.
  **L513 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqabss_s32, aarch64_neon_sqabs, Add1ArgType),`。
- **L514 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqaddb_s8, aarch64_neon_sqadd, Vectorize1ArgType | Use64BitVectors),`.
  **L514 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqaddb_s8, aarch64_neon_sqadd, Vectorize1ArgType | Use64BitVectors),`。
- **L515 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqaddb_u8, aarch64_neon_uqadd, Vectorize1ArgType | Use64BitVectors),`.
  **L515 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqaddb_u8, aarch64_neon_uqadd, Vectorize1ArgType | Use64BitVectors),`。
- **L516 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqaddd_s64, aarch64_neon_sqadd, Add1ArgType),`.
  **L516 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqaddd_s64, aarch64_neon_sqadd, Add1ArgType),`。
- **L517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqaddd_u64, aarch64_neon_uqadd, Add1ArgType),`.
  **L517 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqaddd_u64, aarch64_neon_uqadd, Add1ArgType),`。
- **L518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqaddh_s16, aarch64_neon_sqadd, Vectorize1ArgType | Use64BitVectors),`.
  **L518 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqaddh_s16, aarch64_neon_sqadd, Vectorize1ArgType | Use64BitVectors),`。
- **L519 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqaddh_u16, aarch64_neon_uqadd, Vectorize1ArgType | Use64BitVectors),`.
  **L519 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqaddh_u16, aarch64_neon_uqadd, Vectorize1ArgType | Use64BitVectors),`。
- **L520 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqadds_s32, aarch64_neon_sqadd, Add1ArgType),`.
  **L520 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqadds_s32, aarch64_neon_sqadd, Add1ArgType),`。

### Lines 521-540

````cpp
  NEONMAP1(vqadds_u32, aarch64_neon_uqadd, Add1ArgType),
  NEONMAP1(vqdmulhh_s16, aarch64_neon_sqdmulh, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqdmulhs_s32, aarch64_neon_sqdmulh, Add1ArgType),
  NEONMAP1(vqdmullh_s16, aarch64_neon_sqdmull, VectorRet | Use128BitVectors),
  NEONMAP1(vqdmulls_s32, aarch64_neon_sqdmulls_scalar, 0),
  NEONMAP1(vqmovnd_s64, aarch64_neon_scalar_sqxtn, AddRetType | Add1ArgType),
  NEONMAP1(vqmovnd_u64, aarch64_neon_scalar_uqxtn, AddRetType | Add1ArgType),
  NEONMAP1(vqmovnh_s16, aarch64_neon_sqxtn, VectorRet | Use64BitVectors),
  NEONMAP1(vqmovnh_u16, aarch64_neon_uqxtn, VectorRet | Use64BitVectors),
  NEONMAP1(vqmovns_s32, aarch64_neon_sqxtn, VectorRet | Use64BitVectors),
  NEONMAP1(vqmovns_u32, aarch64_neon_uqxtn, VectorRet | Use64BitVectors),
  NEONMAP1(vqmovund_s64, aarch64_neon_scalar_sqxtun, AddRetType | Add1ArgType),
  NEONMAP1(vqmovunh_s16, aarch64_neon_sqxtun, VectorRet | Use64BitVectors),
  NEONMAP1(vqmovuns_s32, aarch64_neon_sqxtun, VectorRet | Use64BitVectors),
  NEONMAP1(vqnegb_s8, aarch64_neon_sqneg, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqnegd_s64, aarch64_neon_sqneg, Add1ArgType),
  NEONMAP1(vqnegh_s16, aarch64_neon_sqneg, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqnegs_s32, aarch64_neon_sqneg, Add1ArgType),
  NEONMAP1(vqrdmlahh_s16, aarch64_neon_sqrdmlah, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqrdmlahs_s32, aarch64_neon_sqrdmlah, Add1ArgType),
````
- **L521 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqadds_u32, aarch64_neon_uqadd, Add1ArgType),`.
  **L521 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqadds_u32, aarch64_neon_uqadd, Add1ArgType),`。
- **L522 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqdmulhh_s16, aarch64_neon_sqdmulh, Vectorize1ArgType | Use64BitVectors),`.
  **L522 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqdmulhh_s16, aarch64_neon_sqdmulh, Vectorize1ArgType | Use64BitVectors),`。
- **L523 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqdmulhs_s32, aarch64_neon_sqdmulh, Add1ArgType),`.
  **L523 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqdmulhs_s32, aarch64_neon_sqdmulh, Add1ArgType),`。
- **L524 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqdmullh_s16, aarch64_neon_sqdmull, VectorRet | Use128BitVectors),`.
  **L524 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqdmullh_s16, aarch64_neon_sqdmull, VectorRet | Use128BitVectors),`。
- **L525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqdmulls_s32, aarch64_neon_sqdmulls_scalar, 0),`.
  **L525 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqdmulls_s32, aarch64_neon_sqdmulls_scalar, 0),`。
- **L526 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqmovnd_s64, aarch64_neon_scalar_sqxtn, AddRetType | Add1ArgType),`.
  **L526 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqmovnd_s64, aarch64_neon_scalar_sqxtn, AddRetType | Add1ArgType),`。
- **L527 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqmovnd_u64, aarch64_neon_scalar_uqxtn, AddRetType | Add1ArgType),`.
  **L527 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqmovnd_u64, aarch64_neon_scalar_uqxtn, AddRetType | Add1ArgType),`。
- **L528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqmovnh_s16, aarch64_neon_sqxtn, VectorRet | Use64BitVectors),`.
  **L528 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqmovnh_s16, aarch64_neon_sqxtn, VectorRet | Use64BitVectors),`。
- **L529 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqmovnh_u16, aarch64_neon_uqxtn, VectorRet | Use64BitVectors),`.
  **L529 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqmovnh_u16, aarch64_neon_uqxtn, VectorRet | Use64BitVectors),`。
- **L530 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqmovns_s32, aarch64_neon_sqxtn, VectorRet | Use64BitVectors),`.
  **L530 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqmovns_s32, aarch64_neon_sqxtn, VectorRet | Use64BitVectors),`。
- **L531 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqmovns_u32, aarch64_neon_uqxtn, VectorRet | Use64BitVectors),`.
  **L531 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqmovns_u32, aarch64_neon_uqxtn, VectorRet | Use64BitVectors),`。
- **L532 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqmovund_s64, aarch64_neon_scalar_sqxtun, AddRetType | Add1ArgType),`.
  **L532 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqmovund_s64, aarch64_neon_scalar_sqxtun, AddRetType | Add1ArgType),`。
- **L533 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqmovunh_s16, aarch64_neon_sqxtun, VectorRet | Use64BitVectors),`.
  **L533 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqmovunh_s16, aarch64_neon_sqxtun, VectorRet | Use64BitVectors),`。
- **L534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqmovuns_s32, aarch64_neon_sqxtun, VectorRet | Use64BitVectors),`.
  **L534 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqmovuns_s32, aarch64_neon_sqxtun, VectorRet | Use64BitVectors),`。
- **L535 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqnegb_s8, aarch64_neon_sqneg, Vectorize1ArgType | Use64BitVectors),`.
  **L535 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqnegb_s8, aarch64_neon_sqneg, Vectorize1ArgType | Use64BitVectors),`。
- **L536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqnegd_s64, aarch64_neon_sqneg, Add1ArgType),`.
  **L536 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqnegd_s64, aarch64_neon_sqneg, Add1ArgType),`。
- **L537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqnegh_s16, aarch64_neon_sqneg, Vectorize1ArgType | Use64BitVectors),`.
  **L537 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqnegh_s16, aarch64_neon_sqneg, Vectorize1ArgType | Use64BitVectors),`。
- **L538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqnegs_s32, aarch64_neon_sqneg, Add1ArgType),`.
  **L538 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqnegs_s32, aarch64_neon_sqneg, Add1ArgType),`。
- **L539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmlahh_s16, aarch64_neon_sqrdmlah, Vectorize1ArgType | Use64BitVectors),`.
  **L539 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmlahh_s16, aarch64_neon_sqrdmlah, Vectorize1ArgType | Use64BitVectors),`。
- **L540 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmlahs_s32, aarch64_neon_sqrdmlah, Add1ArgType),`.
  **L540 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmlahs_s32, aarch64_neon_sqrdmlah, Add1ArgType),`。

### Lines 541-560

````cpp
  NEONMAP1(vqrdmlshh_s16, aarch64_neon_sqrdmlsh, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqrdmlshs_s32, aarch64_neon_sqrdmlsh, Add1ArgType),
  NEONMAP1(vqrdmulhh_s16, aarch64_neon_sqrdmulh, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqrdmulhs_s32, aarch64_neon_sqrdmulh, Add1ArgType),
  NEONMAP1(vqrshlb_s8, aarch64_neon_sqrshl, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqrshlb_u8, aarch64_neon_uqrshl, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqrshld_s64, aarch64_neon_sqrshl, Add1ArgType),
  NEONMAP1(vqrshld_u64, aarch64_neon_uqrshl, Add1ArgType),
  NEONMAP1(vqrshlh_s16, aarch64_neon_sqrshl, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqrshlh_u16, aarch64_neon_uqrshl, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqrshls_s32, aarch64_neon_sqrshl, Add1ArgType),
  NEONMAP1(vqrshls_u32, aarch64_neon_uqrshl, Add1ArgType),
  NEONMAP1(vqrshrnd_n_s64, aarch64_neon_sqrshrn, AddRetType),
  NEONMAP1(vqrshrnd_n_u64, aarch64_neon_uqrshrn, AddRetType),
  NEONMAP1(vqrshrnh_n_s16, aarch64_neon_sqrshrn, VectorRet | Use64BitVectors),
  NEONMAP1(vqrshrnh_n_u16, aarch64_neon_uqrshrn, VectorRet | Use64BitVectors),
  NEONMAP1(vqrshrns_n_s32, aarch64_neon_sqrshrn, VectorRet | Use64BitVectors),
  NEONMAP1(vqrshrns_n_u32, aarch64_neon_uqrshrn, VectorRet | Use64BitVectors),
  NEONMAP1(vqrshrund_n_s64, aarch64_neon_sqrshrun, AddRetType),
  NEONMAP1(vqrshrunh_n_s16, aarch64_neon_sqrshrun, VectorRet | Use64BitVectors),
````
- **L541 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmlshh_s16, aarch64_neon_sqrdmlsh, Vectorize1ArgType | Use64BitVectors),`.
  **L541 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmlshh_s16, aarch64_neon_sqrdmlsh, Vectorize1ArgType | Use64BitVectors),`。
- **L542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmlshs_s32, aarch64_neon_sqrdmlsh, Add1ArgType),`.
  **L542 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmlshs_s32, aarch64_neon_sqrdmlsh, Add1ArgType),`。
- **L543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmulhh_s16, aarch64_neon_sqrdmulh, Vectorize1ArgType | Use64BitVectors),`.
  **L543 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmulhh_s16, aarch64_neon_sqrdmulh, Vectorize1ArgType | Use64BitVectors),`。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrdmulhs_s32, aarch64_neon_sqrdmulh, Add1ArgType),`.
  **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrdmulhs_s32, aarch64_neon_sqrdmulh, Add1ArgType),`。
- **L545 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshlb_s8, aarch64_neon_sqrshl, Vectorize1ArgType | Use64BitVectors),`.
  **L545 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshlb_s8, aarch64_neon_sqrshl, Vectorize1ArgType | Use64BitVectors),`。
- **L546 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshlb_u8, aarch64_neon_uqrshl, Vectorize1ArgType | Use64BitVectors),`.
  **L546 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshlb_u8, aarch64_neon_uqrshl, Vectorize1ArgType | Use64BitVectors),`。
- **L547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshld_s64, aarch64_neon_sqrshl, Add1ArgType),`.
  **L547 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshld_s64, aarch64_neon_sqrshl, Add1ArgType),`。
- **L548 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshld_u64, aarch64_neon_uqrshl, Add1ArgType),`.
  **L548 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshld_u64, aarch64_neon_uqrshl, Add1ArgType),`。
- **L549 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshlh_s16, aarch64_neon_sqrshl, Vectorize1ArgType | Use64BitVectors),`.
  **L549 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshlh_s16, aarch64_neon_sqrshl, Vectorize1ArgType | Use64BitVectors),`。
- **L550 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshlh_u16, aarch64_neon_uqrshl, Vectorize1ArgType | Use64BitVectors),`.
  **L550 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshlh_u16, aarch64_neon_uqrshl, Vectorize1ArgType | Use64BitVectors),`。
- **L551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshls_s32, aarch64_neon_sqrshl, Add1ArgType),`.
  **L551 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshls_s32, aarch64_neon_sqrshl, Add1ArgType),`。
- **L552 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshls_u32, aarch64_neon_uqrshl, Add1ArgType),`.
  **L552 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshls_u32, aarch64_neon_uqrshl, Add1ArgType),`。
- **L553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshrnd_n_s64, aarch64_neon_sqrshrn, AddRetType),`.
  **L553 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshrnd_n_s64, aarch64_neon_sqrshrn, AddRetType),`。
- **L554 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshrnd_n_u64, aarch64_neon_uqrshrn, AddRetType),`.
  **L554 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshrnd_n_u64, aarch64_neon_uqrshrn, AddRetType),`。
- **L555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshrnh_n_s16, aarch64_neon_sqrshrn, VectorRet | Use64BitVectors),`.
  **L555 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshrnh_n_s16, aarch64_neon_sqrshrn, VectorRet | Use64BitVectors),`。
- **L556 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshrnh_n_u16, aarch64_neon_uqrshrn, VectorRet | Use64BitVectors),`.
  **L556 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshrnh_n_u16, aarch64_neon_uqrshrn, VectorRet | Use64BitVectors),`。
- **L557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshrns_n_s32, aarch64_neon_sqrshrn, VectorRet | Use64BitVectors),`.
  **L557 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshrns_n_s32, aarch64_neon_sqrshrn, VectorRet | Use64BitVectors),`。
- **L558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshrns_n_u32, aarch64_neon_uqrshrn, VectorRet | Use64BitVectors),`.
  **L558 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshrns_n_u32, aarch64_neon_uqrshrn, VectorRet | Use64BitVectors),`。
- **L559 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshrund_n_s64, aarch64_neon_sqrshrun, AddRetType),`.
  **L559 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshrund_n_s64, aarch64_neon_sqrshrun, AddRetType),`。
- **L560 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshrunh_n_s16, aarch64_neon_sqrshrun, VectorRet | Use64BitVectors),`.
  **L560 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshrunh_n_s16, aarch64_neon_sqrshrun, VectorRet | Use64BitVectors),`。

### Lines 561-580

````cpp
  NEONMAP1(vqrshruns_n_s32, aarch64_neon_sqrshrun, VectorRet | Use64BitVectors),
  NEONMAP1(vqshlb_n_s8, aarch64_neon_sqshl, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqshlb_n_u8, aarch64_neon_uqshl, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqshlb_s8, aarch64_neon_sqshl, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqshlb_u8, aarch64_neon_uqshl, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqshld_s64, aarch64_neon_sqshl, Add1ArgType),
  NEONMAP1(vqshld_u64, aarch64_neon_uqshl, Add1ArgType),
  NEONMAP1(vqshlh_n_s16, aarch64_neon_sqshl, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqshlh_n_u16, aarch64_neon_uqshl, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqshlh_s16, aarch64_neon_sqshl, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqshlh_u16, aarch64_neon_uqshl, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqshls_n_s32, aarch64_neon_sqshl, Add1ArgType),
  NEONMAP1(vqshls_n_u32, aarch64_neon_uqshl, Add1ArgType),
  NEONMAP1(vqshls_s32, aarch64_neon_sqshl, Add1ArgType),
  NEONMAP1(vqshls_u32, aarch64_neon_uqshl, Add1ArgType),
  NEONMAP1(vqshlub_n_s8, aarch64_neon_sqshlu, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqshluh_n_s16, aarch64_neon_sqshlu, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqshlus_n_s32, aarch64_neon_sqshlu, Add1ArgType),
  NEONMAP1(vqshrnd_n_s64, aarch64_neon_sqshrn, AddRetType),
  NEONMAP1(vqshrnd_n_u64, aarch64_neon_uqshrn, AddRetType),
````
- **L561 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqrshruns_n_s32, aarch64_neon_sqrshrun, VectorRet | Use64BitVectors),`.
  **L561 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqrshruns_n_s32, aarch64_neon_sqrshrun, VectorRet | Use64BitVectors),`。
- **L562 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshlb_n_s8, aarch64_neon_sqshl, Vectorize1ArgType | Use64BitVectors),`.
  **L562 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshlb_n_s8, aarch64_neon_sqshl, Vectorize1ArgType | Use64BitVectors),`。
- **L563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshlb_n_u8, aarch64_neon_uqshl, Vectorize1ArgType | Use64BitVectors),`.
  **L563 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshlb_n_u8, aarch64_neon_uqshl, Vectorize1ArgType | Use64BitVectors),`。
- **L564 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshlb_s8, aarch64_neon_sqshl, Vectorize1ArgType | Use64BitVectors),`.
  **L564 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshlb_s8, aarch64_neon_sqshl, Vectorize1ArgType | Use64BitVectors),`。
- **L565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshlb_u8, aarch64_neon_uqshl, Vectorize1ArgType | Use64BitVectors),`.
  **L565 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshlb_u8, aarch64_neon_uqshl, Vectorize1ArgType | Use64BitVectors),`。
- **L566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshld_s64, aarch64_neon_sqshl, Add1ArgType),`.
  **L566 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshld_s64, aarch64_neon_sqshl, Add1ArgType),`。
- **L567 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshld_u64, aarch64_neon_uqshl, Add1ArgType),`.
  **L567 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshld_u64, aarch64_neon_uqshl, Add1ArgType),`。
- **L568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshlh_n_s16, aarch64_neon_sqshl, Vectorize1ArgType | Use64BitVectors),`.
  **L568 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshlh_n_s16, aarch64_neon_sqshl, Vectorize1ArgType | Use64BitVectors),`。
- **L569 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshlh_n_u16, aarch64_neon_uqshl, Vectorize1ArgType | Use64BitVectors),`.
  **L569 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshlh_n_u16, aarch64_neon_uqshl, Vectorize1ArgType | Use64BitVectors),`。
- **L570 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshlh_s16, aarch64_neon_sqshl, Vectorize1ArgType | Use64BitVectors),`.
  **L570 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshlh_s16, aarch64_neon_sqshl, Vectorize1ArgType | Use64BitVectors),`。
- **L571 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshlh_u16, aarch64_neon_uqshl, Vectorize1ArgType | Use64BitVectors),`.
  **L571 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshlh_u16, aarch64_neon_uqshl, Vectorize1ArgType | Use64BitVectors),`。
- **L572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshls_n_s32, aarch64_neon_sqshl, Add1ArgType),`.
  **L572 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshls_n_s32, aarch64_neon_sqshl, Add1ArgType),`。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshls_n_u32, aarch64_neon_uqshl, Add1ArgType),`.
  **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshls_n_u32, aarch64_neon_uqshl, Add1ArgType),`。
- **L574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshls_s32, aarch64_neon_sqshl, Add1ArgType),`.
  **L574 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshls_s32, aarch64_neon_sqshl, Add1ArgType),`。
- **L575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshls_u32, aarch64_neon_uqshl, Add1ArgType),`.
  **L575 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshls_u32, aarch64_neon_uqshl, Add1ArgType),`。
- **L576 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshlub_n_s8, aarch64_neon_sqshlu, Vectorize1ArgType | Use64BitVectors),`.
  **L576 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshlub_n_s8, aarch64_neon_sqshlu, Vectorize1ArgType | Use64BitVectors),`。
- **L577 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshluh_n_s16, aarch64_neon_sqshlu, Vectorize1ArgType | Use64BitVectors),`.
  **L577 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshluh_n_s16, aarch64_neon_sqshlu, Vectorize1ArgType | Use64BitVectors),`。
- **L578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshlus_n_s32, aarch64_neon_sqshlu, Add1ArgType),`.
  **L578 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshlus_n_s32, aarch64_neon_sqshlu, Add1ArgType),`。
- **L579 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshrnd_n_s64, aarch64_neon_sqshrn, AddRetType),`.
  **L579 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshrnd_n_s64, aarch64_neon_sqshrn, AddRetType),`。
- **L580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshrnd_n_u64, aarch64_neon_uqshrn, AddRetType),`.
  **L580 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshrnd_n_u64, aarch64_neon_uqshrn, AddRetType),`。

### Lines 581-600

````cpp
  NEONMAP1(vqshrnh_n_s16, aarch64_neon_sqshrn, VectorRet | Use64BitVectors),
  NEONMAP1(vqshrnh_n_u16, aarch64_neon_uqshrn, VectorRet | Use64BitVectors),
  NEONMAP1(vqshrns_n_s32, aarch64_neon_sqshrn, VectorRet | Use64BitVectors),
  NEONMAP1(vqshrns_n_u32, aarch64_neon_uqshrn, VectorRet | Use64BitVectors),
  NEONMAP1(vqshrund_n_s64, aarch64_neon_sqshrun, AddRetType),
  NEONMAP1(vqshrunh_n_s16, aarch64_neon_sqshrun, VectorRet | Use64BitVectors),
  NEONMAP1(vqshruns_n_s32, aarch64_neon_sqshrun, VectorRet | Use64BitVectors),
  NEONMAP1(vqsubb_s8, aarch64_neon_sqsub, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqsubb_u8, aarch64_neon_uqsub, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqsubd_s64, aarch64_neon_sqsub, Add1ArgType),
  NEONMAP1(vqsubd_u64, aarch64_neon_uqsub, Add1ArgType),
  NEONMAP1(vqsubh_s16, aarch64_neon_sqsub, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqsubh_u16, aarch64_neon_uqsub, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vqsubs_s32, aarch64_neon_sqsub, Add1ArgType),
  NEONMAP1(vqsubs_u32, aarch64_neon_uqsub, Add1ArgType),
  NEONMAP1(vrecped_f64, aarch64_neon_frecpe, Add1ArgType),
  NEONMAP1(vrecpes_f32, aarch64_neon_frecpe, Add1ArgType),
  NEONMAP1(vrecpxd_f64, aarch64_neon_frecpx, Add1ArgType),
  NEONMAP1(vrecpxs_f32, aarch64_neon_frecpx, Add1ArgType),
  NEONMAP1(vrshld_s64, aarch64_neon_srshl, Add1ArgType),
````
- **L581 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshrnh_n_s16, aarch64_neon_sqshrn, VectorRet | Use64BitVectors),`.
  **L581 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshrnh_n_s16, aarch64_neon_sqshrn, VectorRet | Use64BitVectors),`。
- **L582 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshrnh_n_u16, aarch64_neon_uqshrn, VectorRet | Use64BitVectors),`.
  **L582 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshrnh_n_u16, aarch64_neon_uqshrn, VectorRet | Use64BitVectors),`。
- **L583 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshrns_n_s32, aarch64_neon_sqshrn, VectorRet | Use64BitVectors),`.
  **L583 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshrns_n_s32, aarch64_neon_sqshrn, VectorRet | Use64BitVectors),`。
- **L584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshrns_n_u32, aarch64_neon_uqshrn, VectorRet | Use64BitVectors),`.
  **L584 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshrns_n_u32, aarch64_neon_uqshrn, VectorRet | Use64BitVectors),`。
- **L585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshrund_n_s64, aarch64_neon_sqshrun, AddRetType),`.
  **L585 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshrund_n_s64, aarch64_neon_sqshrun, AddRetType),`。
- **L586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshrunh_n_s16, aarch64_neon_sqshrun, VectorRet | Use64BitVectors),`.
  **L586 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshrunh_n_s16, aarch64_neon_sqshrun, VectorRet | Use64BitVectors),`。
- **L587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqshruns_n_s32, aarch64_neon_sqshrun, VectorRet | Use64BitVectors),`.
  **L587 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqshruns_n_s32, aarch64_neon_sqshrun, VectorRet | Use64BitVectors),`。
- **L588 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqsubb_s8, aarch64_neon_sqsub, Vectorize1ArgType | Use64BitVectors),`.
  **L588 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqsubb_s8, aarch64_neon_sqsub, Vectorize1ArgType | Use64BitVectors),`。
- **L589 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqsubb_u8, aarch64_neon_uqsub, Vectorize1ArgType | Use64BitVectors),`.
  **L589 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqsubb_u8, aarch64_neon_uqsub, Vectorize1ArgType | Use64BitVectors),`。
- **L590 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqsubd_s64, aarch64_neon_sqsub, Add1ArgType),`.
  **L590 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqsubd_s64, aarch64_neon_sqsub, Add1ArgType),`。
- **L591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqsubd_u64, aarch64_neon_uqsub, Add1ArgType),`.
  **L591 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqsubd_u64, aarch64_neon_uqsub, Add1ArgType),`。
- **L592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqsubh_s16, aarch64_neon_sqsub, Vectorize1ArgType | Use64BitVectors),`.
  **L592 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqsubh_s16, aarch64_neon_sqsub, Vectorize1ArgType | Use64BitVectors),`。
- **L593 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqsubh_u16, aarch64_neon_uqsub, Vectorize1ArgType | Use64BitVectors),`.
  **L593 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqsubh_u16, aarch64_neon_uqsub, Vectorize1ArgType | Use64BitVectors),`。
- **L594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqsubs_s32, aarch64_neon_sqsub, Add1ArgType),`.
  **L594 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqsubs_s32, aarch64_neon_sqsub, Add1ArgType),`。
- **L595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vqsubs_u32, aarch64_neon_uqsub, Add1ArgType),`.
  **L595 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vqsubs_u32, aarch64_neon_uqsub, Add1ArgType),`。
- **L596 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrecped_f64, aarch64_neon_frecpe, Add1ArgType),`.
  **L596 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrecped_f64, aarch64_neon_frecpe, Add1ArgType),`。
- **L597 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrecpes_f32, aarch64_neon_frecpe, Add1ArgType),`.
  **L597 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrecpes_f32, aarch64_neon_frecpe, Add1ArgType),`。
- **L598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrecpxd_f64, aarch64_neon_frecpx, Add1ArgType),`.
  **L598 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrecpxd_f64, aarch64_neon_frecpx, Add1ArgType),`。
- **L599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrecpxs_f32, aarch64_neon_frecpx, Add1ArgType),`.
  **L599 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrecpxs_f32, aarch64_neon_frecpx, Add1ArgType),`。
- **L600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrshld_s64, aarch64_neon_srshl, Add1ArgType),`.
  **L600 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrshld_s64, aarch64_neon_srshl, Add1ArgType),`。

### Lines 601-620

````cpp
  NEONMAP1(vrshld_u64, aarch64_neon_urshl, Add1ArgType),
  NEONMAP1(vrsqrted_f64, aarch64_neon_frsqrte, Add1ArgType),
  NEONMAP1(vrsqrtes_f32, aarch64_neon_frsqrte, Add1ArgType),
  NEONMAP1(vrsqrtsd_f64, aarch64_neon_frsqrts, Add1ArgType),
  NEONMAP1(vrsqrtss_f32, aarch64_neon_frsqrts, Add1ArgType),
  NEONMAP1(vsha1cq_u32, aarch64_crypto_sha1c, 0),
  NEONMAP1(vsha1h_u32, aarch64_crypto_sha1h, 0),
  NEONMAP1(vsha1mq_u32, aarch64_crypto_sha1m, 0),
  NEONMAP1(vsha1pq_u32, aarch64_crypto_sha1p, 0),
  NEONMAP1(vshld_s64, aarch64_neon_sshl, Add1ArgType),
  NEONMAP1(vshld_u64, aarch64_neon_ushl, Add1ArgType),
  NEONMAP1(vslid_n_s64, aarch64_neon_vsli, Vectorize1ArgType),
  NEONMAP1(vslid_n_u64, aarch64_neon_vsli, Vectorize1ArgType),
  NEONMAP1(vsqaddb_u8, aarch64_neon_usqadd, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vsqaddd_u64, aarch64_neon_usqadd, Add1ArgType),
  NEONMAP1(vsqaddh_u16, aarch64_neon_usqadd, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vsqadds_u32, aarch64_neon_usqadd, Add1ArgType),
  NEONMAP1(vsrid_n_s64, aarch64_neon_vsri, Vectorize1ArgType),
  NEONMAP1(vsrid_n_u64, aarch64_neon_vsri, Vectorize1ArgType),
  NEONMAP1(vuqaddb_s8, aarch64_neon_suqadd, Vectorize1ArgType | Use64BitVectors),
````
- **L601 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrshld_u64, aarch64_neon_urshl, Add1ArgType),`.
  **L601 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrshld_u64, aarch64_neon_urshl, Add1ArgType),`。
- **L602 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrsqrted_f64, aarch64_neon_frsqrte, Add1ArgType),`.
  **L602 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrsqrted_f64, aarch64_neon_frsqrte, Add1ArgType),`。
- **L603 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrsqrtes_f32, aarch64_neon_frsqrte, Add1ArgType),`.
  **L603 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrsqrtes_f32, aarch64_neon_frsqrte, Add1ArgType),`。
- **L604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrsqrtsd_f64, aarch64_neon_frsqrts, Add1ArgType),`.
  **L604 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrsqrtsd_f64, aarch64_neon_frsqrts, Add1ArgType),`。
- **L605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrsqrtss_f32, aarch64_neon_frsqrts, Add1ArgType),`.
  **L605 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrsqrtss_f32, aarch64_neon_frsqrts, Add1ArgType),`。
- **L606 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsha1cq_u32, aarch64_crypto_sha1c, 0),`.
  **L606 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsha1cq_u32, aarch64_crypto_sha1c, 0),`。
- **L607 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsha1h_u32, aarch64_crypto_sha1h, 0),`.
  **L607 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsha1h_u32, aarch64_crypto_sha1h, 0),`。
- **L608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsha1mq_u32, aarch64_crypto_sha1m, 0),`.
  **L608 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsha1mq_u32, aarch64_crypto_sha1m, 0),`。
- **L609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsha1pq_u32, aarch64_crypto_sha1p, 0),`.
  **L609 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsha1pq_u32, aarch64_crypto_sha1p, 0),`。
- **L610 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vshld_s64, aarch64_neon_sshl, Add1ArgType),`.
  **L610 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vshld_s64, aarch64_neon_sshl, Add1ArgType),`。
- **L611 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vshld_u64, aarch64_neon_ushl, Add1ArgType),`.
  **L611 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vshld_u64, aarch64_neon_ushl, Add1ArgType),`。
- **L612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vslid_n_s64, aarch64_neon_vsli, Vectorize1ArgType),`.
  **L612 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vslid_n_s64, aarch64_neon_vsli, Vectorize1ArgType),`。
- **L613 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vslid_n_u64, aarch64_neon_vsli, Vectorize1ArgType),`.
  **L613 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vslid_n_u64, aarch64_neon_vsli, Vectorize1ArgType),`。
- **L614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsqaddb_u8, aarch64_neon_usqadd, Vectorize1ArgType | Use64BitVectors),`.
  **L614 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsqaddb_u8, aarch64_neon_usqadd, Vectorize1ArgType | Use64BitVectors),`。
- **L615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsqaddd_u64, aarch64_neon_usqadd, Add1ArgType),`.
  **L615 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsqaddd_u64, aarch64_neon_usqadd, Add1ArgType),`。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsqaddh_u16, aarch64_neon_usqadd, Vectorize1ArgType | Use64BitVectors),`.
  **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsqaddh_u16, aarch64_neon_usqadd, Vectorize1ArgType | Use64BitVectors),`。
- **L617 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsqadds_u32, aarch64_neon_usqadd, Add1ArgType),`.
  **L617 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsqadds_u32, aarch64_neon_usqadd, Add1ArgType),`。
- **L618 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsrid_n_s64, aarch64_neon_vsri, Vectorize1ArgType),`.
  **L618 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsrid_n_s64, aarch64_neon_vsri, Vectorize1ArgType),`。
- **L619 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vsrid_n_u64, aarch64_neon_vsri, Vectorize1ArgType),`.
  **L619 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vsrid_n_u64, aarch64_neon_vsri, Vectorize1ArgType),`。
- **L620 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vuqaddb_s8, aarch64_neon_suqadd, Vectorize1ArgType | Use64BitVectors),`.
  **L620 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vuqaddb_s8, aarch64_neon_suqadd, Vectorize1ArgType | Use64BitVectors),`。

### Lines 621-640

````cpp
  NEONMAP1(vuqaddd_s64, aarch64_neon_suqadd, Add1ArgType),
  NEONMAP1(vuqaddh_s16, aarch64_neon_suqadd, Vectorize1ArgType | Use64BitVectors),
  NEONMAP1(vuqadds_s32, aarch64_neon_suqadd, Add1ArgType),
  // FP16 scalar intrinisics go here.
  NEONMAP1(vabdh_f16, aarch64_sisd_fabd, Add1ArgType),
  NEONMAP1(vcvtah_s32_f16, aarch64_neon_fcvtas, AddRetType | Add1ArgType),
  NEONMAP1(vcvtah_s64_f16, aarch64_neon_fcvtas, AddRetType | Add1ArgType),
  NEONMAP1(vcvtah_u32_f16, aarch64_neon_fcvtau, AddRetType | Add1ArgType),
  NEONMAP1(vcvtah_u64_f16, aarch64_neon_fcvtau, AddRetType | Add1ArgType),
  NEONMAP1(vcvth_n_f16_s32, aarch64_neon_vcvtfxs2fp, AddRetType | Add1ArgType),
  NEONMAP1(vcvth_n_f16_s64, aarch64_neon_vcvtfxs2fp, AddRetType | Add1ArgType),
  NEONMAP1(vcvth_n_f16_u32, aarch64_neon_vcvtfxu2fp, AddRetType | Add1ArgType),
  NEONMAP1(vcvth_n_f16_u64, aarch64_neon_vcvtfxu2fp, AddRetType | Add1ArgType),
  NEONMAP1(vcvth_n_s32_f16, aarch64_neon_vcvtfp2fxs, AddRetType | Add1ArgType),
  NEONMAP1(vcvth_n_s64_f16, aarch64_neon_vcvtfp2fxs, AddRetType | Add1ArgType),
  NEONMAP1(vcvth_n_u32_f16, aarch64_neon_vcvtfp2fxu, AddRetType | Add1ArgType),
  NEONMAP1(vcvth_n_u64_f16, aarch64_neon_vcvtfp2fxu, AddRetType | Add1ArgType),
  NEONMAP1(vcvth_s16_f16, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),
  NEONMAP1(vcvth_s32_f16, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),
  NEONMAP1(vcvth_s64_f16, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),
````
- **L621 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vuqaddd_s64, aarch64_neon_suqadd, Add1ArgType),`.
  **L621 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vuqaddd_s64, aarch64_neon_suqadd, Add1ArgType),`。
- **L622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vuqaddh_s16, aarch64_neon_suqadd, Vectorize1ArgType | Use64BitVectors),`.
  **L622 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vuqaddh_s16, aarch64_neon_suqadd, Vectorize1ArgType | Use64BitVectors),`。
- **L623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vuqadds_s32, aarch64_neon_suqadd, Add1ArgType),`.
  **L623 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vuqadds_s32, aarch64_neon_suqadd, Add1ArgType),`。
- **L624 EN**: Comment explains nearby logic, constraints, or intent: `FP16 scalar intrinisics go here.`.
  **L624 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FP16 scalar intrinisics go here.`。
- **L625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vabdh_f16, aarch64_sisd_fabd, Add1ArgType),`.
  **L625 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vabdh_f16, aarch64_sisd_fabd, Add1ArgType),`。
- **L626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtah_s32_f16, aarch64_neon_fcvtas, AddRetType | Add1ArgType),`.
  **L626 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtah_s32_f16, aarch64_neon_fcvtas, AddRetType | Add1ArgType),`。
- **L627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtah_s64_f16, aarch64_neon_fcvtas, AddRetType | Add1ArgType),`.
  **L627 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtah_s64_f16, aarch64_neon_fcvtas, AddRetType | Add1ArgType),`。
- **L628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtah_u32_f16, aarch64_neon_fcvtau, AddRetType | Add1ArgType),`.
  **L628 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtah_u32_f16, aarch64_neon_fcvtau, AddRetType | Add1ArgType),`。
- **L629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtah_u64_f16, aarch64_neon_fcvtau, AddRetType | Add1ArgType),`.
  **L629 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtah_u64_f16, aarch64_neon_fcvtau, AddRetType | Add1ArgType),`。
- **L630 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvth_n_f16_s32, aarch64_neon_vcvtfxs2fp, AddRetType | Add1ArgType),`.
  **L630 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvth_n_f16_s32, aarch64_neon_vcvtfxs2fp, AddRetType | Add1ArgType),`。
- **L631 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvth_n_f16_s64, aarch64_neon_vcvtfxs2fp, AddRetType | Add1ArgType),`.
  **L631 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvth_n_f16_s64, aarch64_neon_vcvtfxs2fp, AddRetType | Add1ArgType),`。
- **L632 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvth_n_f16_u32, aarch64_neon_vcvtfxu2fp, AddRetType | Add1ArgType),`.
  **L632 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvth_n_f16_u32, aarch64_neon_vcvtfxu2fp, AddRetType | Add1ArgType),`。
- **L633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvth_n_f16_u64, aarch64_neon_vcvtfxu2fp, AddRetType | Add1ArgType),`.
  **L633 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvth_n_f16_u64, aarch64_neon_vcvtfxu2fp, AddRetType | Add1ArgType),`。
- **L634 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvth_n_s32_f16, aarch64_neon_vcvtfp2fxs, AddRetType | Add1ArgType),`.
  **L634 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvth_n_s32_f16, aarch64_neon_vcvtfp2fxs, AddRetType | Add1ArgType),`。
- **L635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvth_n_s64_f16, aarch64_neon_vcvtfp2fxs, AddRetType | Add1ArgType),`.
  **L635 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvth_n_s64_f16, aarch64_neon_vcvtfp2fxs, AddRetType | Add1ArgType),`。
- **L636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvth_n_u32_f16, aarch64_neon_vcvtfp2fxu, AddRetType | Add1ArgType),`.
  **L636 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvth_n_u32_f16, aarch64_neon_vcvtfp2fxu, AddRetType | Add1ArgType),`。
- **L637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvth_n_u64_f16, aarch64_neon_vcvtfp2fxu, AddRetType | Add1ArgType),`.
  **L637 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvth_n_u64_f16, aarch64_neon_vcvtfp2fxu, AddRetType | Add1ArgType),`。
- **L638 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvth_s16_f16, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`.
  **L638 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvth_s16_f16, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`。
- **L639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvth_s32_f16, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`.
  **L639 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvth_s32_f16, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`。
- **L640 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvth_s64_f16, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`.
  **L640 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvth_s64_f16, aarch64_neon_fcvtzs, AddRetType | Add1ArgType),`。

### Lines 641-660

````cpp
  NEONMAP1(vcvth_u16_f16, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),
  NEONMAP1(vcvth_u32_f16, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),
  NEONMAP1(vcvth_u64_f16, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtmh_s32_f16, aarch64_neon_fcvtms, AddRetType | Add1ArgType),
  NEONMAP1(vcvtmh_s64_f16, aarch64_neon_fcvtms, AddRetType | Add1ArgType),
  NEONMAP1(vcvtmh_u32_f16, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtmh_u64_f16, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtnh_s32_f16, aarch64_neon_fcvtns, AddRetType | Add1ArgType),
  NEONMAP1(vcvtnh_s64_f16, aarch64_neon_fcvtns, AddRetType | Add1ArgType),
  NEONMAP1(vcvtnh_u32_f16, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtnh_u64_f16, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtph_s32_f16, aarch64_neon_fcvtps, AddRetType | Add1ArgType),
  NEONMAP1(vcvtph_s64_f16, aarch64_neon_fcvtps, AddRetType | Add1ArgType),
  NEONMAP1(vcvtph_u32_f16, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),
  NEONMAP1(vcvtph_u64_f16, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),
  NEONMAP1(vmulxh_f16, aarch64_neon_fmulx, Add1ArgType),
  NEONMAP1(vrecpeh_f16, aarch64_neon_frecpe, Add1ArgType),
  NEONMAP1(vrecpxh_f16, aarch64_neon_frecpx, Add1ArgType),
  NEONMAP1(vrsqrteh_f16, aarch64_neon_frsqrte, Add1ArgType),
  NEONMAP1(vrsqrtsh_f16, aarch64_neon_frsqrts, Add1ArgType),
````
- **L641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvth_u16_f16, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`.
  **L641 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvth_u16_f16, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`。
- **L642 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvth_u32_f16, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`.
  **L642 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvth_u32_f16, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`。
- **L643 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvth_u64_f16, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`.
  **L643 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvth_u64_f16, aarch64_neon_fcvtzu, AddRetType | Add1ArgType),`。
- **L644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtmh_s32_f16, aarch64_neon_fcvtms, AddRetType | Add1ArgType),`.
  **L644 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtmh_s32_f16, aarch64_neon_fcvtms, AddRetType | Add1ArgType),`。
- **L645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtmh_s64_f16, aarch64_neon_fcvtms, AddRetType | Add1ArgType),`.
  **L645 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtmh_s64_f16, aarch64_neon_fcvtms, AddRetType | Add1ArgType),`。
- **L646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtmh_u32_f16, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),`.
  **L646 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtmh_u32_f16, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),`。
- **L647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtmh_u64_f16, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),`.
  **L647 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtmh_u64_f16, aarch64_neon_fcvtmu, AddRetType | Add1ArgType),`。
- **L648 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtnh_s32_f16, aarch64_neon_fcvtns, AddRetType | Add1ArgType),`.
  **L648 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtnh_s32_f16, aarch64_neon_fcvtns, AddRetType | Add1ArgType),`。
- **L649 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtnh_s64_f16, aarch64_neon_fcvtns, AddRetType | Add1ArgType),`.
  **L649 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtnh_s64_f16, aarch64_neon_fcvtns, AddRetType | Add1ArgType),`。
- **L650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtnh_u32_f16, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),`.
  **L650 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtnh_u32_f16, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),`。
- **L651 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtnh_u64_f16, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),`.
  **L651 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtnh_u64_f16, aarch64_neon_fcvtnu, AddRetType | Add1ArgType),`。
- **L652 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtph_s32_f16, aarch64_neon_fcvtps, AddRetType | Add1ArgType),`.
  **L652 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtph_s32_f16, aarch64_neon_fcvtps, AddRetType | Add1ArgType),`。
- **L653 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtph_s64_f16, aarch64_neon_fcvtps, AddRetType | Add1ArgType),`.
  **L653 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtph_s64_f16, aarch64_neon_fcvtps, AddRetType | Add1ArgType),`。
- **L654 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtph_u32_f16, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),`.
  **L654 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtph_u32_f16, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),`。
- **L655 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vcvtph_u64_f16, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),`.
  **L655 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vcvtph_u64_f16, aarch64_neon_fcvtpu, AddRetType | Add1ArgType),`。
- **L656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vmulxh_f16, aarch64_neon_fmulx, Add1ArgType),`.
  **L656 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vmulxh_f16, aarch64_neon_fmulx, Add1ArgType),`。
- **L657 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrecpeh_f16, aarch64_neon_frecpe, Add1ArgType),`.
  **L657 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrecpeh_f16, aarch64_neon_frecpe, Add1ArgType),`。
- **L658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrecpxh_f16, aarch64_neon_frecpx, Add1ArgType),`.
  **L658 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrecpxh_f16, aarch64_neon_frecpx, Add1ArgType),`。
- **L659 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrsqrteh_f16, aarch64_neon_frsqrte, Add1ArgType),`.
  **L659 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrsqrteh_f16, aarch64_neon_frsqrte, Add1ArgType),`。
- **L660 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NEONMAP1(vrsqrtsh_f16, aarch64_neon_frsqrts, Add1ArgType),`.
  **L660 CN**: 继续一个多行参数列表、初始化器或聚合项：`NEONMAP1(vrsqrtsh_f16, aarch64_neon_frsqrts, Add1ArgType),`。

### Lines 661-667

````cpp
};
// clang-format on

} // namespace aarch64
} // namespace clang

#endif
````
- **L661 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L661 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L662 EN**: Comment explains nearby logic, constraints, or intent: `clang-format on`.
  **L662 CN**: 注释解释附近代码的逻辑、约束或设计意图：`clang-format on`。
- **L663 EN**: Blank line separating nearby declarations or logic blocks.
  **L663 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L664 EN**: Closes a namespace scope and documents it with a trailing comment: `} // namespace aarch64`.
  **L664 CN**: 结束一个命名空间作用域，并用尾注释标明其名称：`} // namespace aarch64`。
- **L665 EN**: Closes a namespace scope and documents it with a trailing comment: `} // namespace clang`.
  **L665 CN**: 结束一个命名空间作用域，并用尾注释标明其名称：`} // namespace clang`。
- **L666 EN**: Blank line separating nearby declarations or logic blocks.
  **L666 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L667 EN**: Closes the current preprocessor conditional block.
  **L667 CN**: 结束当前预处理条件块。

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
- **Builtin descriptions / Builtin 描述**
  - **EN**: Describes compiler builtins, their signatures, properties, or target-specific availability.
  - **CN**: 描述编译器 builtin 的签名、属性或目标相关可用性。
- **Type-system support / 类型系统支撑**
  - **EN**: Provides metadata or helpers used to model Clang types and related dispatch logic.
  - **CN**: 提供建模 Clang 类型及相关分发逻辑所需的元数据或辅助工具。
- **AArch64 target support / AArch64 目标支持**
  - **EN**: Encodes AArch64-specific compiler metadata or builtin descriptions.
  - **CN**: 编码 AArch64 专用的编译器元数据或 builtin 描述。
- **Arm target support / Arm 目标支持**
  - **EN**: Encodes Arm-specific language extensions, intrinsics, or target metadata.
  - **CN**: 编码 Arm 专用语言扩展、intrinsic 或目标元数据。
- **Header composition / 头文件组合**
  - **EN**: Uses direct includes to layer declarations on top of existing Clang/LLVM interfaces.
  - **CN**: 通过直接 include 在既有 Clang/LLVM 接口之上分层构建声明。
- **Macro-driven generation / 宏驱动生成**
  - **EN**: Relies on macros or generated include fragments to keep large definition sets synchronized.
  - **CN**: 依赖宏或生成式 include 片段来同步大规模定义集合。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**:
  - `clang/Basic/TargetBuiltins.h`: Provides Clang basic-layer enums, diagnostics, target data, and utilities. / 提供Clang Basic 层枚举、诊断、目标数据与工具。
  - `llvm/IR/Intrinsics.h`: Provides LLVM IR core abstractions. / 提供LLVM IR 核心抽象。
  - `llvm/IR/IntrinsicsAArch64.h`: Provides LLVM IR core abstractions. / 提供LLVM IR 核心抽象。
- **Macros / 宏**: `LLVM_CLANG_BASIC_AARCH64CODEGENUTILS_H`, `NEONMAP0(NameBase)`, `NEONMAP1(NameBase,`, `NEONMAP2(NameBase,`
- **Types / 类型**: `ARMVectorIntrinsicInfo`
- **Functions or callables / 函数或可调用对象**: `operator<`, `NEONMAP0`, `NEONMAP1`, `NEONMAP2`, `Data`
- **TableGen records / TableGen 记录**: No prominent TableGen record names detected. / 未检测到明显的 TableGen 记录名称。
- **Namespaces / 命名空间**: `clang`, `aarch64`
