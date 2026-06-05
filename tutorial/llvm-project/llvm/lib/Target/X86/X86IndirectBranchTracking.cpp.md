# X86IndirectBranchTracking.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86IndirectBranchTracking.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements indirect-branch tracking for the core X86 backend. / 实现X86 后端核心中的间接分支跟踪。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===---- X86IndirectBranchTracking.cpp - Enables CET IBT mechanism -------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines a pass that enables Indirect Branch Tracking (IBT) as part
// of Control-Flow Enforcement Technology (CET).
// The pass adds ENDBR (End Branch) machine instructions at the beginning of
// each basic block or function that is referenced by an indrect jump/call
// instruction.
// The ENDBR instructions have a NOP encoding and as such are ignored in
// targets that do not support CET IBT mechanism.
//===----------------------------------------------------------------------===//

#include "X86.h"
#include "X86InstrInfo.h"
#include "X86Subtarget.h"
#include "X86TargetMachine.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineModuleInfo.h"
#include "llvm/IR/Module.h"

using namespace llvm;

#define DEBUG_TYPE "x86-indirect-branch-tracking"
```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Namespace scope management / 命名空间作用域管理
```cpp

cl::opt<bool> IndirectBranchTracking(
    "x86-indirect-branch-tracking", cl::init(false), cl::Hidden,
    cl::desc("Enable X86 indirect branch tracking pass."));

STATISTIC(NumEndBranchAdded, "Number of ENDBR instructions added");

namespace {
class X86IndirectBranchTrackingLegacy : public MachineFunctionPass {
public:
  static char ID;

  X86IndirectBranchTrackingLegacy() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override {
    return "X86 Indirect Branch Tracking";
  }

  bool runOnMachineFunction(MachineFunction &MF) override;
};

/// Adds a new ENDBR instruction to the beginning of the MBB.
/// The function will not add it if already exists.
/// It will add ENDBR32 or ENDBR64 opcode, depending on the target.
/// \returns true if the ENDBR was added and false otherwise.
static bool addENDBR(MachineBasicBlock &MBB, MachineBasicBlock::iterator I) {
  MachineFunction &MF = *MBB.getParent();
  const X86Subtarget &SubTarget = MF.getSubtarget<X86Subtarget>();
  const X86InstrInfo *TII = SubTarget.getInstrInfo();
  unsigned EndbrOpcode = SubTarget.is64Bit() ? X86::ENDBR64 : X86::ENDBR32;
```
**EN:** This section implements namespace scope management for the core X86 backend. Key symbols include X86IndirectBranchTrackingLegacy. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的命名空间作用域管理。关键符号包括 X86IndirectBranchTrackingLegacy。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

  assert(TII && "Target instruction info was not initialized");
  assert((X86::ENDBR64 == EndbrOpcode || X86::ENDBR32 == EndbrOpcode) &&
         "Unexpected Endbr opcode");

  // If the MBB/I is empty or the current instruction is not ENDBR,
  // insert ENDBR instruction to the location of I.
  if (I == MBB.end() || I->getOpcode() != EndbrOpcode) {
    BuildMI(MBB, I, MBB.findDebugLoc(I), TII->get(EndbrOpcode));
    ++NumEndBranchAdded;
    return true;
  }
  return false;
}

} // end anonymous namespace

char X86IndirectBranchTrackingLegacy::ID = 0;

INITIALIZE_PASS(X86IndirectBranchTrackingLegacy, DEBUG_TYPE,
                "X86 Indirect Branch Tracking", false, false)

FunctionPass *llvm::createX86IndirectBranchTrackingLegacyPass() {
  return new X86IndirectBranchTrackingLegacy();
}

static bool IsCallReturnTwice(llvm::MachineOperand &MOp) {
  if (!MOp.isGlobal())
    return false;
  auto *CalleeFn = dyn_cast<Function>(MOp.getGlobal());
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  if (!CalleeFn)
    return false;
  AttributeList Attrs = CalleeFn->getAttributes();
  return Attrs.hasFnAttr(Attribute::ReturnsTwice);
}

// Checks if function should have an ENDBR in its prologue
static bool needsPrologueENDBR(MachineFunction &MF, const Module *M) {
  Function &F = MF.getFunction();

  if (F.doesNoCfCheck())
    return false;

  switch (MF.getTarget().getCodeModel()) {
  // Large code model functions always reachable through indirect calls.
  case CodeModel::Large:
    return true;
  // Address taken or externally linked functions may be reachable.
  default:
    return (F.hasAddressTaken() || !F.hasLocalLinkage());
  }
}

static bool runIndirectBranchTracking(MachineFunction &MF) {
  const Module *M = MF.getFunction().getParent();
  // Check that the cf-protection-branch is enabled.
  Metadata *isCFProtectionSupported = M->getModuleFlag("cf-protection-branch");

  //  NB: We need to enable IBT in jitted code if JIT compiler is CET
  //  enabled.
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-150: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
  const X86TargetMachine *TM =
      static_cast<const X86TargetMachine *>(&MF.getTarget());
#ifdef __CET__
  bool isJITwithCET = TM->isJIT();
#else
  bool isJITwithCET = false;
#endif
  if (!isCFProtectionSupported && !IndirectBranchTracking && !isJITwithCET)
    return false;

  // True if the current MF was changed and false otherwise.
  bool Changed = false;

  // If function is reachable indirectly, mark the first BB with ENDBR.
  if (needsPrologueENDBR(MF, M)) {
    auto MBB = MF.begin();
    Changed |= addENDBR(*MBB, MBB->begin());
  }

  for (auto &MBB : MF) {
    // Find all basic blocks that their address was taken (for example
    // in the case of indirect jump) and add ENDBR instruction.
    if (MBB.isMachineBlockAddressTaken() || MBB.isIRBlockAddressTaken())
      Changed |= addENDBR(MBB, MBB.begin());

    for (MachineBasicBlock::iterator I = MBB.begin(); I != MBB.end(); ++I) {
      if (I->isCall() && I->getNumOperands() > 0 &&
          IsCallReturnTwice(I->getOperand(0))) {
        Changed |= addENDBR(MBB, std::next(I));
      }
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-180: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
    }

    // Exception handle may indirectly jump to catch pad, So we should add
    // ENDBR before catch pad instructions. For SjLj exception model, it will
    // create a new BB(new landingpad) indirectly jump to the old landingpad.
    if (TM->Options.ExceptionModel == ExceptionHandling::SjLj) {
      for (MachineBasicBlock::iterator I = MBB.begin(); I != MBB.end(); ++I) {
        // New Landingpad BB without EHLabel.
        if (MBB.isEHPad()) {
          if (I->isDebugInstr())
            continue;
          Changed |= addENDBR(MBB, I);
          break;
        } else if (I->isEHLabel()) {
          // Old Landingpad BB (is not Landingpad now) with
          // the old "callee" EHLabel.
          MCSymbol *Sym = I->getOperand(0).getMCSymbol();
          if (!MF.hasCallSiteLandingPad(Sym))
            continue;
          Changed |= addENDBR(MBB, std::next(I));
          break;
        }
      }
    } else if (MBB.isEHPad()){
      for (MachineBasicBlock::iterator I = MBB.begin(); I != MBB.end(); ++I) {
        if (!I->isEHLabel())
          continue;
        Changed |= addENDBR(MBB, std::next(I));
        break;
      }
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 181-198: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
    }
  }
  return Changed;
}

bool X86IndirectBranchTrackingLegacy::runOnMachineFunction(
    MachineFunction &MF) {
  return runIndirectBranchTracking(MF);
}

PreservedAnalyses
X86IndirectBranchTrackingPass::run(MachineFunction &MF,
                                   MachineFunctionAnalysisManager &MFAM) {
  return runIndirectBranchTracking(MF)
             ? getMachineFunctionPassPreservedAnalyses()
                   .preserveSet<CFGAnalyses>()
             : PreservedAnalyses::all();
}
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: indirect-branch tracking. / 核心主题：间接分支跟踪。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86IndirectBranchTrackingLegacy. / 重要符号：X86IndirectBranchTrackingLegacy。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86InstrInfo.h, X86Subtarget.h, X86TargetMachine.h, llvm/ADT/Statistic.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineModuleInfo.h, llvm/IR/Module.h. / 直接包含：X86.h, X86InstrInfo.h, X86Subtarget.h, X86TargetMachine.h, llvm/ADT/Statistic.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineModuleInfo.h, llvm/IR/Module.h。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
