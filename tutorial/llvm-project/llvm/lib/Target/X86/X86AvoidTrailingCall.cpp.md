# X86AvoidTrailingCall.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86AvoidTrailingCall.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements trailing-call avoidance for the core X86 backend. / 实现X86 后端核心中的尾部调用规避。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===----- X86AvoidTrailingCall.cpp - Insert int3 after trailing calls ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// The Windows x64 unwinder decodes the instruction stream during unwinding.
// The unwinder decodes forward from the current PC to detect epilogue code
// patterns.
//
// First, this means that there must be an instruction after every
// call instruction for the unwinder to decode. LLVM must maintain the invariant
// that the last instruction of a function or funclet is not a call, or the
// unwinder may decode into the next function. Similarly, a call may not
// immediately precede an epilogue code pattern. As of this writing, the
// SEH_Epilogue pseudo instruction takes care of that.
//
// Second, all non-tail call jump targets must be within the *half-open*
// interval of the bounds of the function. The unwinder distinguishes between
// internal jump instructions and tail calls in an epilogue sequence by checking
// the jump target against the function bounds from the .pdata section. This
// means that the last regular MBB of an LLVM function must not be empty if
// there are regular jumps targeting it.
//
// This pass upholds these invariants by ensuring that blocks at the end of a
// function or funclet are a) not empty and b) do not end in a CALL instruction.
//
// Unwinder implementation for reference:
```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
// https://github.com/dotnet/coreclr/blob/a9f3fc16483eecfc47fb79c362811d870be02249/src/unwinder/amd64/unwinder_amd64.cpp#L1015
//
//===----------------------------------------------------------------------===//

#include "X86.h"
#include "X86InstrInfo.h"
#include "X86Subtarget.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/IR/Analysis.h"
#include "llvm/IR/PassManager.h"

#define AVOIDCALL_DESC "X86 avoid trailing call pass"
#define AVOIDCALL_NAME "x86-avoid-trailing-call"

#define DEBUG_TYPE AVOIDCALL_NAME

using namespace llvm;

namespace {
class X86AvoidTrailingCallLegacyPass : public MachineFunctionPass {
public:
  X86AvoidTrailingCallLegacyPass() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  static char ID;

private:
  StringRef getPassName() const override { return AVOIDCALL_DESC; }
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. Key symbols include X86AvoidTrailingCallLegacyPass. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。关键符号包括 X86AvoidTrailingCallLegacyPass。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
};
} // end anonymous namespace

char X86AvoidTrailingCallLegacyPass::ID = 0;

FunctionPass *llvm::createX86AvoidTrailingCallLegacyPass() {
  return new X86AvoidTrailingCallLegacyPass();
}

INITIALIZE_PASS(X86AvoidTrailingCallLegacyPass, AVOIDCALL_NAME, AVOIDCALL_DESC,
                false, false)

// A real instruction is a non-meta, non-pseudo instruction.  Some pseudos
// expand to nothing, and some expand to code. This logic conservatively assumes
// they might expand to nothing.
static bool isCallOrRealInstruction(MachineInstr &MI) {
  return MI.isCall() || (!MI.isPseudo() && !MI.isMetaInstruction());
}

// Return true if this is a call instruction, but not a tail call.
static bool isCallInstruction(const MachineInstr &MI) {
  return MI.isCall() && !MI.isReturn();
}

bool UpdatedOnX86AvoidTrailingCallPass(MachineFunction &MF) {
  const X86Subtarget &STI = MF.getSubtarget<X86Subtarget>();
  const X86InstrInfo &TII = *STI.getInstrInfo();
  assert(STI.isTargetWin64() && "pass only runs on Win64");

  // We don't need to worry about any of the invariants described above if there
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
  // is no unwind info (CFI).
  if (!MF.hasWinCFI())
    return false;

  // FIXME: Perhaps this pass should also replace SEH_Epilogue by inserting nops
  // before epilogues.

  bool Changed = false;
  for (MachineBasicBlock &MBB : MF) {
    // Look for basic blocks that precede funclet entries or are at the end of
    // the function.
    MachineBasicBlock *NextMBB = MBB.getNextNode();
    if (NextMBB && !NextMBB->isEHFuncletEntry())
      continue;

    // Find the last real instruction in this block.
    auto LastRealInstr = llvm::find_if(reverse(MBB), isCallOrRealInstruction);

    // If the block is empty or the last real instruction is a call instruction,
    // insert an int3. If there is a call instruction, insert the int3 between
    // the call and any labels or other meta instructions. If the block is
    // empty, insert at block end.
    bool IsEmpty = LastRealInstr == MBB.rend();
    bool IsCall = !IsEmpty && isCallInstruction(*LastRealInstr);
    if (IsEmpty || IsCall) {
      LLVM_DEBUG({
        if (IsCall) {
          dbgs() << "inserting int3 after trailing call instruction:\n";
          LastRealInstr->dump();
          dbgs() << '\n';
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-150: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
        } else {
          dbgs() << "inserting int3 in trailing empty MBB:\n";
          MBB.dump();
        }
      });

      MachineBasicBlock::iterator MBBI = MBB.end();
      DebugLoc DL;
      if (IsCall) {
        MBBI = std::next(LastRealInstr.getReverse());
        DL = LastRealInstr->getDebugLoc();
      }
      BuildMI(MBB, MBBI, DL, TII.get(X86::INT3));
      Changed = true;
    }
  }

  return Changed;
}

bool X86AvoidTrailingCallLegacyPass::runOnMachineFunction(MachineFunction &MF) {
  return UpdatedOnX86AvoidTrailingCallPass(MF);
}

PreservedAnalyses
X86AvoidTrailingCallPass::run(MachineFunction &MF,
                              MachineFunctionAnalysisManager &MFAM) {
  bool Changed = UpdatedOnX86AvoidTrailingCallPass(MF);
  if (!Changed)
    return PreservedAnalyses::all();
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-153: Supporting logic and local data handling / 辅助逻辑与局部数据处理
```cpp

  return getMachineFunctionPassPreservedAnalyses().preserveSet<CFGAnalyses>();
}
```
**EN:** This section implements supporting logic and local data handling for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的辅助逻辑与局部数据处理。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: trailing-call avoidance. / 核心主题：尾部调用规避。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86AvoidTrailingCallLegacyPass. / 重要符号：X86AvoidTrailingCallLegacyPass。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86InstrInfo.h, X86Subtarget.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/IR/Analysis.h, llvm/IR/PassManager.h. / 直接包含：X86.h, X86InstrInfo.h, X86Subtarget.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/IR/Analysis.h, llvm/IR/PassManager.h。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
