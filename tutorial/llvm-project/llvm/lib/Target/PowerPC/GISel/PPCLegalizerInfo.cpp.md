# PPCLegalizerInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/GISel/PPCLegalizerInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements the targeting of the Machinelegalizer class for PowerPC
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/GISel/PPCLegalizerInfo.cpp`，主要负责 PowerPC 后端的GlobalISel 合法化规则。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===- PPCLegalizerInfo.h ----------------------------------------*- C++ -*-==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-9

```cpp
//===----------------------------------------------------------------------===//
/// \file
/// This file implements the targeting of the Machinelegalizer class for PowerPC
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 10-47

```cpp
//===----------------------------------------------------------------------===//

#include "PPCLegalizerInfo.h"

#define DEBUG_TYPE "ppc-legalinfo"

using namespace llvm;
using namespace LegalizeActions;
using namespace LegalizeMutations;
using namespace LegalityPredicates;

static LegalityPredicate isRegisterType(unsigned TypeIdx) {
  return [=](const LegalityQuery &Query) {
    const LLT QueryTy = Query.Types[TypeIdx];
    unsigned TypeSize = QueryTy.getSizeInBits();

    if (TypeSize % 32 == 1 || TypeSize > 128)
      return false;

    // Check if this is a legal PowerPC vector type.
    if (QueryTy.isVector()) {
      const int EltSize = QueryTy.getElementType().getSizeInBits();
      return (EltSize == 8 || EltSize == 16 || EltSize == 32 || EltSize == 64);
    }

    return true;
  };
}

PPCLegalizerInfo::PPCLegalizerInfo(const PPCSubtarget &ST) {
  using namespace TargetOpcode;
  const LLT P0 = LLT::pointer(0, 64);
  const LLT S1 = LLT::scalar(1);
  const LLT S8 = LLT::scalar(8);
  const LLT S16 = LLT::scalar(16);
  const LLT S32 = LLT::scalar(32);
  const LLT S64 = LLT::scalar(64);
  const LLT V16S8 = LLT::fixed_vector(16, 8);
```
- **EN**: Pulls in direct dependencies required by this GlobalISel legalization rules, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该GlobalISel 合法化规则所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 48-85

```cpp
  const LLT V8S16 = LLT::fixed_vector(8, 16);
  const LLT V4S32 = LLT::fixed_vector(4, 32);
  const LLT V2S64 = LLT::fixed_vector(2, 64);
  getActionDefinitionsBuilder(G_IMPLICIT_DEF).legalFor({S64});
  getActionDefinitionsBuilder(G_CONSTANT)
      .legalFor({S32, S64})
      .clampScalar(0, S64, S64);
  getActionDefinitionsBuilder({G_ZEXT, G_SEXT, G_ANYEXT})
      .legalForCartesianProduct({S64}, {S1, S8, S16, S32})
      .clampScalar(0, S64, S64);
  getActionDefinitionsBuilder({G_AND, G_OR, G_XOR})
      .legalFor({S64, V4S32})
      .clampScalar(0, S64, S64)
      .bitcastIf(typeIsNot(0, V4S32), changeTo(0, V4S32));
  getActionDefinitionsBuilder({G_ADD, G_SUB})
      .legalFor({S64, V16S8, V8S16, V4S32, V2S64})
      .clampScalar(0, S64, S64);
  getActionDefinitionsBuilder(G_BITCAST)
      .legalIf(all(isRegisterType(0), isRegisterType(1)))
      .lower();

  getActionDefinitionsBuilder({G_FADD, G_FSUB, G_FMUL, G_FDIV})
      .legalFor({S32, S64, V4S32, V2S64});

  getActionDefinitionsBuilder(G_FCMP).legalForCartesianProduct({S1},
                                                               {S32, S64});

  getActionDefinitionsBuilder({G_FPTOSI, G_FPTOUI})
      .legalForCartesianProduct({S64}, {S32, S64});

  getActionDefinitionsBuilder({G_SITOFP, G_UITOFP})
      .legalForCartesianProduct({S32, S64}, {S64});

  getActionDefinitionsBuilder({G_LOAD, G_STORE})
      .legalForTypesWithMemDesc({{S64, P0, S64, 8}, {S32, P0, S32, 4}});

  getActionDefinitionsBuilder(G_FCONSTANT).lowerFor({S32, S64});
  getActionDefinitionsBuilder(G_CONSTANT_POOL).legalFor({P0});
```
- **EN**: Implements helper routine(s) `fixed_vector`, `getActionDefinitionsBuilder`, `legalFor` for this portion of the PowerPC backend GlobalISel legalization rules.
- **CN**: 这里实现了 PowerPC 后端该部分GlobalISel 合法化规则所需的辅助例程 `fixed_vector`, `getActionDefinitionsBuilder`, `legalFor`。

### Lines 86-93

```cpp

  getActionDefinitionsBuilder(G_BR).alwaysLegal();

  getActionDefinitionsBuilder({G_INTRINSIC, G_INTRINSIC_W_SIDE_EFFECTS})
      .alwaysLegal();

  getLegacyLegalizerInfo().computeTables();
}
```
- **EN**: Implements helper routine(s) `getActionDefinitionsBuilder`, `alwaysLegal`, `getLegacyLegalizerInfo` for this portion of the PowerPC backend GlobalISel legalization rules.
- **CN**: 这里实现了 PowerPC 后端该部分GlobalISel 合法化规则所需的辅助例程 `getActionDefinitionsBuilder`, `alwaysLegal`, `getLegacyLegalizerInfo`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- GlobalISel legalization rules / GlobalISel 合法化规则
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Type or operation legalization / 类型或操作合法化
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPCLegalizerInfo.h`

### Important Collaborators / 重要协作组件

- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
