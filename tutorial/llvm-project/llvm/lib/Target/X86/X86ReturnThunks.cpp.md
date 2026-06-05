# X86ReturnThunks.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86ReturnThunks.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements X86 target definitions for the core X86 backend. / 实现X86 后端核心中的X86 目标定义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//==- X86ReturnThunks.cpp - Replace rets with thunks or inline thunks --=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
/// \file
///
/// Pass that replaces ret instructions with a jmp to __x86_return_thunk.
///
/// This corresponds to -mfunction-return=thunk-extern or
/// __attribute__((function_return("thunk-extern").
///
/// This pass is a minimal implementation necessary to help mitigate
/// RetBleed for the Linux kernel.
///
/// Should support for thunk or thunk-inline be necessary in the future, then
/// this pass should be combined with x86-retpoline-thunks which already has
/// machinery to emit thunks. Until then, YAGNI.
```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 21-40: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
///
/// This pass is very similar to x86-lvi-ret.
///
//===----------------------------------------------------------------------===//

#include "X86.h"
#include "X86InstrInfo.h"
#include "X86Subtarget.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineModuleInfo.h"
#include "llvm/IR/Module.h"
#include "llvm/MC/MCInstrDesc.h"
#include "llvm/Support/Debug.h"
#include "llvm/TargetParser/Triple.h"
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 41-60: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp

using namespace llvm;

#define PASS_KEY "x86-return-thunks"
#define DEBUG_TYPE PASS_KEY

constexpr StringRef X86ReturnThunksPassName = "X86 Return Thunks";

namespace {
struct X86ReturnThunksLegacy final : public MachineFunctionPass {
  static char ID;
  X86ReturnThunksLegacy() : MachineFunctionPass(ID) {}
  StringRef getPassName() const override { return X86ReturnThunksPassName; }
  bool runOnMachineFunction(MachineFunction &MF) override;
};
} // namespace

char X86ReturnThunksLegacy::ID = 0;

static bool runX86ReturnThunks(MachineFunction &MF) {
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. Key symbols include X86ReturnThunksLegacy. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。关键符号包括 X86ReturnThunksLegacy。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-80: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
  LLVM_DEBUG(dbgs() << X86ReturnThunksPassName << "\n");

  bool Modified = false;

  if (!MF.getFunction().hasFnAttribute(llvm::Attribute::FnRetThunkExtern))
    return Modified;

  StringRef ThunkName = "__x86_return_thunk";
  if (MF.getFunction().getName() == ThunkName)
    return Modified;

  const auto &ST = MF.getSubtarget<X86Subtarget>();
  const bool Is64Bit = ST.getTargetTriple().isX86_64();
  const unsigned RetOpc = Is64Bit ? X86::RET64 : X86::RET32;
  SmallVector<MachineInstr *, 16> Rets;

  for (MachineBasicBlock &MBB : MF)
    for (MachineInstr &Term : MBB.terminators())
      if (Term.getOpcode() == RetOpc)
        Rets.push_back(&Term);
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 81-100: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

  bool IndCS =
      MF.getFunction().getParent()->getModuleFlag("indirect_branch_cs_prefix");
  const MCInstrDesc &CS = ST.getInstrInfo()->get(X86::CS_PREFIX);
  const MCInstrDesc &JMP = ST.getInstrInfo()->get(X86::TAILJMPd);

  for (MachineInstr *Ret : Rets) {
    if (IndCS)
      BuildMI(Ret->getParent(), Ret->getDebugLoc(), CS);
    BuildMI(Ret->getParent(), Ret->getDebugLoc(), JMP)
        .addExternalSymbol(ThunkName.data());
    Ret->eraseFromParent();
    Modified = true;
  }

  return Modified;
}

bool X86ReturnThunksLegacy::runOnMachineFunction(MachineFunction &MF) {
  return runX86ReturnThunks(MF);
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 101-116: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
}

PreservedAnalyses
X86ReturnThunksPass::run(MachineFunction &MF,
                         MachineFunctionAnalysisManager &MFAM) {
  return runX86ReturnThunks(MF) ? getMachineFunctionPassPreservedAnalyses()
                                      .preserveSet<CFGAnalyses>()
                                : PreservedAnalyses::all();
}

INITIALIZE_PASS(X86ReturnThunksLegacy, PASS_KEY, "X86 Return Thunks", false,
                false)

FunctionPass *llvm::createX86ReturnThunksLegacyPass() {
  return new X86ReturnThunksLegacy();
}
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: X86 target definitions. / 核心主题：X86 目标定义。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86ReturnThunksLegacy. / 重要符号：X86ReturnThunksLegacy。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86InstrInfo.h, X86Subtarget.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineInstrBuilder.h. / 直接包含：X86.h, X86InstrInfo.h, X86Subtarget.h, llvm/ADT/SmallVector.h, llvm/ADT/StringRef.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineInstrBuilder.h。
- Additional include dependencies: 5 more headers. / 额外包含依赖：还有 5 个头文件。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
