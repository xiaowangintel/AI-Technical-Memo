# WebAssemblyLegalizerInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/GISel/WebAssemblyLegalizerInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements the targeting of the Machinelegalizer class for WebAssembly.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/GISel/WebAssemblyLegalizerInfo.cpp`，主要负责 WebAssembly 后端的GlobalISel 合法化规则。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===- WebAssemblyLegalizerInfo.cpp ------------------------------*- C++ -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-10

```cpp
//===----------------------------------------------------------------------===//
/// \file
/// This file implements the targeting of the Machinelegalizer class for
/// WebAssembly.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 11-18

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssemblyLegalizerInfo.h"
#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "WebAssemblySubtarget.h"
#include "llvm/CodeGen/GlobalISel/LegalizerHelper.h"
#include "llvm/CodeGen/GlobalISel/LegalizerInfo.h"
```
- **EN**: Pulls in direct dependencies required by this GlobalISel legalization rules, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range participates in the GlobalISel pipeline.
- **CN**: 这一段引入该GlobalISel 合法化规则所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段参与 GlobalISel 流水线。

### Lines 19-56

```cpp
#define DEBUG_TYPE "wasm-legalinfo"

using namespace llvm;
using namespace LegalizeActions;

WebAssemblyLegalizerInfo::WebAssemblyLegalizerInfo(
    const WebAssemblySubtarget &ST) {
  using namespace TargetOpcode;

  const LLT i32 = LLT::integer(32);
  const LLT i64 = LLT::integer(64);

  const LLT f32 = LLT::floatIEEE(32);
  const LLT f64 = LLT::floatIEEE(64);

  const LLT s32 = LLT::scalar(32);
  const LLT s64 = LLT::scalar(64);

  getActionDefinitionsBuilder(G_IMPLICIT_DEF)
      .legalFor({i32, i64, f32, f64})
      .widenScalarToNextPow2(0)
      .clampScalar(0, s32, s64);

  getActionDefinitionsBuilder({G_CONSTANT, G_ADD, G_SUB, G_MUL, G_UDIV, G_SDIV,
                               G_UREM, G_SREM, G_AND, G_OR, G_XOR})
      .legalFor({i32, i64})
      .widenScalarToNextPow2(0)
      .clampScalar(0, s32, s64);

  getActionDefinitionsBuilder({G_ASHR, G_LSHR, G_SHL})
      .legalFor({{i32, i32}, {i64, i64}})
      .widenScalarToNextPow2(0)
      .clampScalar(0, s32, s64)
      .scalarSameSizeAs(1, 0);

  getActionDefinitionsBuilder({G_CTLZ, G_CTTZ, G_CTPOP})
      .legalFor({{i32, i32}, {i64, i64}})
      .widenScalarToNextPow2(1)
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Subtarget feature gating influences the behavior here.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 子目标特性裁剪会影响这里的行为。

### Lines 57-94

```cpp
      .clampScalar(1, s32, s64)
      .scalarSameSizeAs(0, 1);

  getActionDefinitionsBuilder({G_CTLZ_ZERO_POISON, G_CTTZ_ZERO_POISON}).lower();

  getActionDefinitionsBuilder({G_ROTL, G_ROTR})
      .legalFor({{i32, i32}, {i64, i64}})
      .scalarSameSizeAs(1, 0)
      .lower();

  getActionDefinitionsBuilder({G_FSHL, G_FSHR}).lower();

  getActionDefinitionsBuilder({G_ANYEXT, G_SEXT, G_ZEXT})
      .legalFor({{i64, i32}})
      .clampScalar(0, s64, s64)
      .clampScalar(1, s32, s32);

  getActionDefinitionsBuilder(G_TRUNC)
      .legalFor({{i32, i64}})
      .clampScalar(0, s32, s32)
      .clampScalar(1, s64, s64);

  getActionDefinitionsBuilder(G_SEXT_INREG)
      .customFor(ST.hasSignExt(), {i32, i64})
      .clampScalar(0, s32, s64)
      .lower();

  getActionDefinitionsBuilder({G_FCONSTANT, G_FABS, G_FNEG, G_FCEIL, G_FFLOOR,
                               G_INTRINSIC_TRUNC, G_FNEARBYINT, G_FRINT,
                               G_INTRINSIC_ROUNDEVEN, G_FSQRT, G_FADD, G_FSUB,
                               G_FMUL, G_FDIV})
      .legalFor({f32, f64})
      .minScalar(0, s32);

  getActionDefinitionsBuilder(G_FCOPYSIGN)
      .legalFor({f32, f64})
      .minScalar(0, s32)
      .scalarSameSizeAs(1, 0);
```
- **EN**: Implements helper routine(s) `clampScalar`, `scalarSameSizeAs`, `getActionDefinitionsBuilder` for this portion of the WebAssembly backend GlobalISel legalization rules.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 合法化规则所需的辅助例程 `clampScalar`, `scalarSameSizeAs`, `getActionDefinitionsBuilder`。

### Lines 95-131

```cpp

  getActionDefinitionsBuilder(G_FPEXT)
      .legalFor({{f64, f32}})
      .clampScalar(0, s64, s64)
      .clampScalar(1, s32, s32);

  getActionDefinitionsBuilder(G_FPTRUNC)
      .legalFor({{f32, f64}})
      .clampScalar(0, s32, s32)
      .clampScalar(1, s64, s64);

  getLegacyLegalizerInfo().computeTables();
}

bool WebAssemblyLegalizerInfo::legalizeCustom(
    LegalizerHelper &Helper, MachineInstr &MI,
    LostDebugLocObserver &LocObserver) const {
  switch (MI.getOpcode()) {
  case TargetOpcode::G_SEXT_INREG: {
    assert(MI.getOperand(2).isImm() && "Expected immediate");

    // Mark only 8/16/32-bit SEXT_INREG as legal
    auto [DstType, SrcType] = MI.getFirst2LLTs();
    auto ExtFromWidth = MI.getOperand(2).getImm();

    if (ExtFromWidth == 8 || ExtFromWidth == 16 ||
        (DstType.getScalarSizeInBits() == 64 && ExtFromWidth == 32)) {
      return true;
    }

    return Helper.lower(MI, 0, DstType) != LegalizerHelper::UnableToLegalize;
  }
  default:
    break;
  }
  return false;
}
```
- **EN**: Implements helper routine(s) `getActionDefinitionsBuilder`, `legalFor`, `clampScalar` for this portion of the WebAssembly backend GlobalISel legalization rules. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分GlobalISel 合法化规则所需的辅助例程 `getActionDefinitionsBuilder`, `legalFor`, `clampScalar`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- GlobalISel legalization rules / GlobalISel 合法化规则
- GlobalISel pipeline / GlobalISel 流水线
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Type or operation legalization / 类型或操作合法化
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyLegalizerInfo.h`
- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `WebAssemblySubtarget.h`
- `llvm/CodeGen/GlobalISel/LegalizerHelper.h`
- `llvm/CodeGen/GlobalISel/LegalizerInfo.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
