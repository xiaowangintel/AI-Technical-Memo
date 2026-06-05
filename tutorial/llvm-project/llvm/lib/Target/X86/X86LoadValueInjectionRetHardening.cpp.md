# X86LoadValueInjectionRetHardening.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86LoadValueInjectionRetHardening.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements load value injection hardening for the core X86 backend. / 实现X86 后端核心中的负载值注入加固。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===-- X86LoadValueInjectionRetHardening.cpp - LVI RET hardening for x86 --==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// Description: Replaces every `ret` instruction with the sequence:
/// ```
/// pop <scratch-reg>
/// lfence
/// jmp *<scratch-reg>
/// ```
/// where `<scratch-reg>` is some available scratch register, according to the
/// calling convention of the function being mitigated.
///
//===----------------------------------------------------------------------===//

#include "X86.h"
#include "X86InstrBuilder.h"
#include "X86Subtarget.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/IR/Function.h"
#include "llvm/Support/Debug.h"

```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
using namespace llvm;

#define PASS_KEY "x86-lvi-ret"
#define DEBUG_TYPE PASS_KEY

STATISTIC(NumFences, "Number of LFENCEs inserted for LVI mitigation");
STATISTIC(NumFunctionsConsidered, "Number of functions analyzed");
STATISTIC(NumFunctionsMitigated, "Number of functions for which mitigations "
                                 "were deployed");

namespace {

constexpr StringRef X86LVIRetPassName =
    "X86 Load Value Injection (LVI) Ret-Hardening";

class X86LoadValueInjectionRetHardeningLegacy : public MachineFunctionPass {
public:
  X86LoadValueInjectionRetHardeningLegacy() : MachineFunctionPass(ID) {}
  StringRef getPassName() const override { return X86LVIRetPassName; }
  bool runOnMachineFunction(MachineFunction &MF) override;

  static char ID;
};

} // end anonymous namespace

char X86LoadValueInjectionRetHardeningLegacy::ID = 0;

static bool runX86LoadValueInjectionRetHardening(MachineFunction &MF) {
  const X86Subtarget *Subtarget = &MF.getSubtarget<X86Subtarget>();
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. Key symbols include X86LoadValueInjectionRetHardeningLegacy. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。关键符号包括 X86LoadValueInjectionRetHardeningLegacy。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
  if (!Subtarget->useLVIControlFlowIntegrity() || !Subtarget->is64Bit())
    return false; // FIXME: support 32-bit

  LLVM_DEBUG(dbgs() << "***** " << X86LVIRetPassName << " : " << MF.getName()
                    << " *****\n");
  ++NumFunctionsConsidered;
  const X86RegisterInfo *TRI = Subtarget->getRegisterInfo();
  const X86InstrInfo *TII = Subtarget->getInstrInfo();

  bool Modified = false;
  for (auto &MBB : MF) {
    for (auto MBBI = MBB.begin(); MBBI != MBB.end(); ++MBBI) {
      if (MBBI->getOpcode() != X86::RET64)
        continue;

      unsigned ClobberReg = TRI->findDeadCallerSavedReg(MBB, MBBI);
      if (ClobberReg != X86::NoRegister) {
        BuildMI(MBB, MBBI, DebugLoc(), TII->get(X86::POP64r))
            .addReg(ClobberReg, RegState::Define)
            .setMIFlag(MachineInstr::FrameDestroy);
        BuildMI(MBB, MBBI, DebugLoc(), TII->get(X86::LFENCE));
        BuildMI(MBB, MBBI, DebugLoc(), TII->get(X86::JMP64r))
            .addReg(ClobberReg);
        MBB.erase(MBBI);
      } else {
        // In case there is no available scratch register, we can still read
        // from RSP to assert that RSP points to a valid page. The write to RSP
        // is also helpful because it verifies that the stack's write
        // permissions are intact.
        MachineInstr *Fence =
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
            BuildMI(MBB, MBBI, DebugLoc(), TII->get(X86::LFENCE));
        addRegOffset(BuildMI(MBB, Fence, DebugLoc(), TII->get(X86::SHL64mi)),
                     X86::RSP, false, 0)
            .addImm(0)
            ->addRegisterDead(X86::EFLAGS, TRI);
      }

      ++NumFences;
      Modified = true;
      break;
    }
  }

  if (Modified)
    ++NumFunctionsMitigated;
  return Modified;
}

bool X86LoadValueInjectionRetHardeningLegacy::runOnMachineFunction(
    MachineFunction &MF) {
  // Don't skip functions with the "optnone" attr but participate in opt-bisect.
  // Note: NewPM implements this behavior by default.
  const Function &F = MF.getFunction();
  if (!F.hasOptNone() && skipFunction(F))
    return false;

  return runX86LoadValueInjectionRetHardening(MF);
}

PreservedAnalyses X86LoadValueInjectionRetHardeningPass::run(
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-133: Supporting logic and local data handling / 辅助逻辑与局部数据处理
```cpp
    MachineFunction &MF, MachineFunctionAnalysisManager &MFAM) {
  return runX86LoadValueInjectionRetHardening(MF)
             ? getMachineFunctionPassPreservedAnalyses()
                   .preserveSet<CFGAnalyses>()
             : PreservedAnalyses::all();
}

INITIALIZE_PASS(X86LoadValueInjectionRetHardeningLegacy, PASS_KEY,
                "X86 LVI ret hardener", false, false)

FunctionPass *llvm::createX86LoadValueInjectionRetHardeningLegacyPass() {
  return new X86LoadValueInjectionRetHardeningLegacy();
}
```
**EN:** This section implements supporting logic and local data handling for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的辅助逻辑与局部数据处理。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: load value injection hardening. / 核心主题：负载值注入加固。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86LoadValueInjectionRetHardeningLegacy. / 重要符号：X86LoadValueInjectionRetHardeningLegacy。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86InstrBuilder.h, X86Subtarget.h, llvm/ADT/Statistic.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/IR/Function.h, llvm/Support/Debug.h. / 直接包含：X86.h, X86InstrBuilder.h, X86Subtarget.h, llvm/ADT/Statistic.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/IR/Function.h, llvm/Support/Debug.h。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
