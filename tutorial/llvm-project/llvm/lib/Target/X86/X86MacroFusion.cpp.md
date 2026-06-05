# X86MacroFusion.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86MacroFusion.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements macro-fusion modeling for the core X86 backend. / 实现X86 后端核心中的宏融合建模。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===- X86MacroFusion.cpp - X86 Macro Fusion ------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file This file contains the X86 implementation of the DAG scheduling
/// mutation to pair instructions back to back.
//
//===----------------------------------------------------------------------===//

#include "X86MacroFusion.h"
#include "MCTargetDesc/X86BaseInfo.h"
#include "X86Subtarget.h"
#include "llvm/CodeGen/MacroFusion.h"
#include "llvm/CodeGen/ScheduleDAGMutation.h"
#include "llvm/CodeGen/TargetInstrInfo.h"

```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 21-40: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
using namespace llvm;

static X86::FirstMacroFusionInstKind classifyFirst(const MachineInstr &MI) {
  return X86::classifyFirstOpcodeInMacroFusion(MI.getOpcode());
}

static X86::SecondMacroFusionInstKind classifySecond(const MachineInstr &MI) {
  X86::CondCode CC = X86::getCondFromBranch(MI);
  return X86::classifySecondCondCodeInMacroFusion(CC);
}

/// Check if the instr pair, FirstMI and SecondMI, should be fused
/// together. Given SecondMI, when FirstMI is unspecified, then check if
/// SecondMI may be part of a fused pair at all.
static bool shouldScheduleAdjacent(const TargetInstrInfo &TII,
                                   const TargetSubtargetInfo &TSI,
                                   const MachineInstr *FirstMI,
                                   const MachineInstr &SecondMI) {
  const X86Subtarget &ST = static_cast<const X86Subtarget &>(TSI);

```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 41-60: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
  // Check if this processor supports any kind of fusion.
  if (!(ST.hasBranchFusion() || ST.hasMacroFusion()))
    return false;

  const X86::SecondMacroFusionInstKind BranchKind = classifySecond(SecondMI);

  if (BranchKind == X86::SecondMacroFusionInstKind::Invalid)
    return false; // Second cannot be fused with anything.

  if (FirstMI == nullptr)
    return true; // We're only checking whether Second can be fused at all.

  const X86::FirstMacroFusionInstKind TestKind = classifyFirst(*FirstMI);

  if (ST.hasBranchFusion()) {
    // Branch fusion can merge CMP and TEST with all conditional jumps.
    return (TestKind == X86::FirstMacroFusionInstKind::Cmp ||
            TestKind == X86::FirstMacroFusionInstKind::Test);
  }

```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-75: Namespace scope management / 命名空间作用域管理
```cpp
  if (ST.hasMacroFusion()) {
    return X86::isMacroFused(TestKind, BranchKind);
  }

  llvm_unreachable("unknown fusion type");
}

namespace llvm {

std::unique_ptr<ScheduleDAGMutation> createX86MacroFusionDAGMutation() {
  return createMacroFusionDAGMutation(shouldScheduleAdjacent,
                                      /*BranchOnly=*/true);
}

} // end namespace llvm
```
**EN:** This section implements namespace scope management for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的命名空间作用域管理。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: macro-fusion modeling. / 核心主题：宏融合建模。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86MacroFusion.h, MCTargetDesc/X86BaseInfo.h, X86Subtarget.h, llvm/CodeGen/MacroFusion.h, llvm/CodeGen/ScheduleDAGMutation.h, llvm/CodeGen/TargetInstrInfo.h. / 直接包含：X86MacroFusion.h, MCTargetDesc/X86BaseInfo.h, X86Subtarget.h, llvm/CodeGen/MacroFusion.h, llvm/CodeGen/ScheduleDAGMutation.h, llvm/CodeGen/TargetInstrInfo.h。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
