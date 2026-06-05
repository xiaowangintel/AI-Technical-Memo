# X86InsertVZeroUpper.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86InsertVZeroUpper.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements vzeroupper insertion for the core X86 backend. / 实现X86 后端核心中的vzeroupper 插入。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===----------------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// \file
/// This file defines the pass which inserts x86 AVX vzeroupper instructions
/// before calls to SSE encoded functions. This avoids transition latency
/// penalty when transferring control between AVX encoded instructions and old
/// SSE encoding mode.
///
//===----------------------------------------------------------------------===//

#include "X86.h"
#include "X86InstrInfo.h"
#include "X86Subtarget.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/MachinePassManager.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/IR/Analysis.h"
#include "llvm/IR/CallingConv.h"
#include "llvm/IR/DebugLoc.h"
#include "llvm/IR/Function.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/raw_ostream.h"
#include <cassert>

using namespace llvm;

#define DEBUG_TYPE "x86-insert-vzeroupper"

static cl::opt<bool>
    UseVZeroUpper("x86-use-vzeroupper", cl::Hidden,
                  cl::desc("Minimize AVX to SSE transition penalty"),
                  cl::init(true));

STATISTIC(NumVZU, "Number of vzeroupper instructions inserted");

namespace {
class X86InsertVZeroUpperLegacy : public MachineFunctionPass {
public:
  static char ID;

  X86InsertVZeroUpperLegacy() : MachineFunctionPass(ID) {}

  StringRef getPassName() const override { return "X86 vzeroupper inserter"; }

```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. Key symbols include X86InsertVZeroUpperLegacy. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。关键符号包括 X86InsertVZeroUpperLegacy。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Type declarations and shared abstractions / 类型声明与共享抽象
```cpp
  bool runOnMachineFunction(MachineFunction &MF) override;

  MachineFunctionProperties getRequiredProperties() const override {
    return MachineFunctionProperties().setNoVRegs();
  }
};

enum BlockExitState { PASS_THROUGH, EXITS_CLEAN, EXITS_DIRTY };

// Core algorithm state:
// BlockState - Each block is either:
//   - PASS_THROUGH: There are neither YMM/ZMM dirtying instructions nor
//                   vzeroupper instructions in this block.
//   - EXITS_CLEAN: There is (or will be) a vzeroupper instruction in this
//                  block that will ensure that YMM/ZMM is clean on exit.
//   - EXITS_DIRTY: An instruction in the block dirties YMM/ZMM and no
//                  subsequent vzeroupper in the block clears it.
//
// AddedToDirtySuccessors - This flag is raised when a block is added to the
//                          DirtySuccessors list to ensure that it's not
//                          added multiple times.
//
// FirstUnguardedCall - Records the location of the first unguarded call in
//                      each basic block that may need to be guarded by a
//                      vzeroupper. We won't know whether it actually needs
//                      to be guarded until we discover a predecessor that
//                      is DIRTY_OUT.
struct BlockState {
  BlockExitState ExitState = PASS_THROUGH;
  bool AddedToDirtySuccessors = false;
```
**EN:** This section implements type declarations and shared abstractions for the core X86 backend. Key symbols include BlockExitState, BlockState. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的类型声明与共享抽象。关键符号包括 BlockExitState, BlockState。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
  MachineBasicBlock::iterator FirstUnguardedCall;

  BlockState() = default;
};

using BlockStateMap = SmallVector<BlockState, 8>;
using DirtySuccessorsWorkList = SmallVector<MachineBasicBlock *, 8>;
} // end anonymous namespace

char X86InsertVZeroUpperLegacy::ID = 0;

FunctionPass *llvm::createX86InsertVZeroUpperLegacyPass() {
  return new X86InsertVZeroUpperLegacy();
}

#ifndef NDEBUG
static const char *getBlockExitStateName(BlockExitState ST) {
  switch (ST) {
  case PASS_THROUGH:
    return "Pass-through";
  case EXITS_DIRTY:
    return "Exits-dirty";
  case EXITS_CLEAN:
    return "Exits-clean";
  }
  llvm_unreachable("Invalid block exit state.");
}
#endif

/// VZEROUPPER cleans state that is related to Y/ZMM0-15 only.
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-150: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
/// Thus, there is no need to check for Y/ZMM16 and above.
static bool isYmmOrZmmReg(MCRegister Reg) {
  return (Reg >= X86::YMM0 && Reg <= X86::YMM15) ||
         (Reg >= X86::ZMM0 && Reg <= X86::ZMM15);
}

static bool checkFnHasLiveInYmmOrZmm(MachineRegisterInfo &MRI) {
  for (std::pair<MCRegister, Register> LI : MRI.liveins())
    if (isYmmOrZmmReg(LI.first))
      return true;

  return false;
}

static bool clobbersAllYmmAndZmmRegs(const MachineOperand &MO) {
  for (unsigned reg = X86::YMM0; reg <= X86::YMM15; ++reg) {
    if (!MO.clobbersPhysReg(reg))
      return false;
  }
  for (unsigned reg = X86::ZMM0; reg <= X86::ZMM15; ++reg) {
    if (!MO.clobbersPhysReg(reg))
      return false;
  }
  return true;
}

static bool hasYmmOrZmmReg(MachineInstr &MI) {
  for (const MachineOperand &MO : MI.operands()) {
    if (MI.isCall() && MO.isRegMask() && !clobbersAllYmmAndZmmRegs(MO))
      return true;
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-180: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
    if (!MO.isReg())
      continue;
    if (MO.isDebug())
      continue;
    if (isYmmOrZmmReg(MO.getReg().asMCReg()))
      return true;
  }
  return false;
}

/// Check if given call instruction has a RegMask operand.
static bool callHasRegMask(MachineInstr &MI) {
  assert(MI.isCall() && "Can only be called on call instructions.");
  for (const MachineOperand &MO : MI.operands()) {
    if (MO.isRegMask())
      return true;
  }
  return false;
}

/// Insert a vzeroupper instruction before I.
static bool insertVZeroUpper(MachineBasicBlock::iterator I,
                             MachineBasicBlock &MBB,
                             const TargetInstrInfo *TII) {
  BuildMI(MBB, I, I->getDebugLoc(), TII->get(X86::VZEROUPPER));
  ++NumVZU;
  return true;
}

/// Add MBB to the DirtySuccessors list if it hasn't already been added.
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 181-210: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
static void addDirtySuccessor(MachineBasicBlock &MBB,
                              BlockStateMap &BlockStates,
                              DirtySuccessorsWorkList &DirtySuccessors) {
  if (!BlockStates[MBB.getNumber()].AddedToDirtySuccessors) {
    DirtySuccessors.push_back(&MBB);
    BlockStates[MBB.getNumber()].AddedToDirtySuccessors = true;
  }
}

/// Loop over all of the instructions in the basic block, inserting vzeroupper
/// instructions before function calls.
static bool processBasicBlock(MachineBasicBlock &MBB,
                              BlockStateMap &BlockStates,
                              DirtySuccessorsWorkList &DirtySuccessors,
                              bool IsX86INTR, const TargetInstrInfo *TII) {
  // Start by assuming that the block is PASS_THROUGH which implies no unguarded
  // calls.
  BlockExitState CurState = PASS_THROUGH;
  BlockStates[MBB.getNumber()].FirstUnguardedCall = MBB.end();
  bool MadeChange = false;

  for (MachineInstr &MI : MBB) {
    bool IsCall = MI.isCall();
    bool IsReturn = MI.isReturn();
    bool IsControlFlow = IsCall || IsReturn;

    // No need for vzeroupper before iret in interrupt handler function,
    // epilogue will restore YMM/ZMM registers if needed.
    if (IsX86INTR && IsReturn)
      continue;
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. Key symbols include addDirtySuccessor. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。关键符号包括 addDirtySuccessor。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 211-240: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp

    // An existing VZERO* instruction resets the state.
    if (MI.getOpcode() == X86::VZEROALL || MI.getOpcode() == X86::VZEROUPPER) {
      CurState = EXITS_CLEAN;
      continue;
    }

    // Shortcut: don't need to check regular instructions in dirty state.
    if (!IsControlFlow && CurState == EXITS_DIRTY)
      continue;

    if (hasYmmOrZmmReg(MI)) {
      // We found a ymm/zmm-using instruction; this could be an AVX/AVX512
      // instruction, or it could be control flow.
      CurState = EXITS_DIRTY;
      continue;
    }

    // Check for control-flow out of the current function (which might
    // indirectly execute SSE instructions).
    if (!IsControlFlow)
      continue;

    // If the call has no RegMask, skip it as well. It usually happens on
    // helper function calls (such as '_chkstk', '_ftol2') where standard
    // calling convention is not used (RegMask is not used to mark register
    // clobbered and register usage (def/implicit-def/use) is well-defined and
    // explicitly specified.
    if (IsCall && !callHasRegMask(MI))
      continue;
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 241-270: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp

    // The VZEROUPPER instruction resets the upper 128 bits of YMM0-YMM15
    // registers. In addition, the processor changes back to Clean state, after
    // which execution of SSE instructions or AVX instructions has no transition
    // penalty. Add the VZEROUPPER instruction before any function call/return
    // that might execute SSE code.
    // FIXME: In some cases, we may want to move the VZEROUPPER into a
    // predecessor block.
    if (CurState == EXITS_DIRTY) {
      // After the inserted VZEROUPPER the state becomes clean again, but
      // other YMM/ZMM may appear before other subsequent calls or even before
      // the end of the BB.
      MadeChange |= insertVZeroUpper(MI, MBB, TII);
      CurState = EXITS_CLEAN;
    } else if (CurState == PASS_THROUGH) {
      // If this block is currently in pass-through state and we encounter a
      // call then whether we need a vzeroupper or not depends on whether this
      // block has successors that exit dirty. Record the location of the call,
      // and set the state to EXITS_CLEAN, but do not insert the vzeroupper yet.
      // It will be inserted later if necessary.
      BlockStates[MBB.getNumber()].FirstUnguardedCall = MI;
      CurState = EXITS_CLEAN;
    }
  }

  LLVM_DEBUG(dbgs() << "MBB #" << MBB.getNumber() << " exit state: "
                    << getBlockExitStateName(CurState) << '\n');

  if (CurState == EXITS_DIRTY)
    for (MachineBasicBlock *Succ : MBB.successors())
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 271-300: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
      addDirtySuccessor(*Succ, BlockStates, DirtySuccessors);

  BlockStates[MBB.getNumber()].ExitState = CurState;
  return MadeChange;
}

/// Loop over all of the basic blocks, inserting vzeroupper instructions before
/// function calls.
static bool insertVZeroUpper(MachineFunction &MF) {
  if (!UseVZeroUpper)
    return false;

  const X86Subtarget &ST = MF.getSubtarget<X86Subtarget>();
  if (!ST.hasAVX() || !ST.insertVZEROUPPER())
    return false;

  MachineRegisterInfo &MRI = MF.getRegInfo();

  bool FnHasLiveInYmmOrZmm = checkFnHasLiveInYmmOrZmm(MRI);

  // Fast check: if the function doesn't use any ymm/zmm registers, we don't
  // need to insert any VZEROUPPER instructions.  This is constant-time, so it
  // is cheap in the common case of no ymm/zmm use.
  bool YmmOrZmmUsed = FnHasLiveInYmmOrZmm;
  for (const auto *RC : {&X86::VR256RegClass, &X86::VR512_0_15RegClass}) {
    if (!YmmOrZmmUsed) {
      for (MCPhysReg R : *RC) {
        if (!MRI.reg_nodbg_empty(R)) {
          YmmOrZmmUsed = true;
          break;
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 301-330: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
        }
      }
    }
  }
  if (!YmmOrZmmUsed)
    return false;

  const TargetInstrInfo *TII = ST.getInstrInfo();
  bool IsX86INTR = MF.getFunction().getCallingConv() == CallingConv::X86_INTR;
  bool EverMadeChange = false;
  BlockStateMap BlockStates(MF.getNumBlockIDs());
  DirtySuccessorsWorkList DirtySuccessors;

  assert(BlockStates.size() == MF.getNumBlockIDs() && DirtySuccessors.empty() &&
         "X86VZeroUpper state should be clear");

  // Process all blocks. This will compute block exit states, record the first
  // unguarded call in each block, and add successors of dirty blocks to the
  // DirtySuccessors list.
  for (MachineBasicBlock &MBB : MF)
    EverMadeChange |=
        processBasicBlock(MBB, BlockStates, DirtySuccessors, IsX86INTR, TII);

  // If any YMM/ZMM regs are live-in to this function, add the entry block to
  // the DirtySuccessors list
  if (FnHasLiveInYmmOrZmm)
    addDirtySuccessor(MF.front(), BlockStates, DirtySuccessors);

  // Re-visit all blocks that are successors of EXITS_DIRTY blocks. Add
  // vzeroupper instructions to unguarded calls, and propagate EXITS_DIRTY
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 331-360: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  // through PASS_THROUGH blocks.
  while (!DirtySuccessors.empty()) {
    MachineBasicBlock &MBB = *DirtySuccessors.back();
    DirtySuccessors.pop_back();
    BlockState &BBState = BlockStates[MBB.getNumber()];

    // MBB is a successor of a dirty block, so its first call needs to be
    // guarded.
    if (BBState.FirstUnguardedCall != MBB.end())
      EverMadeChange |= insertVZeroUpper(BBState.FirstUnguardedCall, MBB, TII);

    // If this successor was a pass-through block, then it is now dirty. Its
    // successors need to be added to the worklist (if they haven't been
    // already).
    if (BBState.ExitState == PASS_THROUGH) {
      LLVM_DEBUG(dbgs() << "MBB #" << MBB.getNumber()
                        << " was Pass-through, is now Dirty-out.\n");
      for (MachineBasicBlock *Succ : MBB.successors())
        addDirtySuccessor(*Succ, BlockStates, DirtySuccessors);
    }
  }

  return EverMadeChange;
}

bool X86InsertVZeroUpperLegacy::runOnMachineFunction(MachineFunction &MF) {
  return insertVZeroUpper(MF);
}

PreservedAnalyses
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 361-366: Supporting logic and local data handling / 辅助逻辑与局部数据处理
```cpp
X86InsertVZeroUpperPass::run(MachineFunction &MF,
                             MachineFunctionAnalysisManager &MFAM) {
  return insertVZeroUpper(MF) ? getMachineFunctionPassPreservedAnalyses()
                                    .preserveSet<CFGAnalyses>()
                              : PreservedAnalyses::all();
}
```
**EN:** This section implements supporting logic and local data handling for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的辅助逻辑与局部数据处理。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: vzeroupper insertion. / 核心主题：vzeroupper 插入。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86InsertVZeroUpperLegacy, BlockExitState, BlockState. / 重要符号：X86InsertVZeroUpperLegacy, BlockExitState, BlockState。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86InstrInfo.h, X86Subtarget.h, llvm/ADT/SmallVector.h, llvm/ADT/Statistic.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineInstrBuilder.h. / 直接包含：X86.h, X86InstrInfo.h, X86Subtarget.h, llvm/ADT/SmallVector.h, llvm/ADT/Statistic.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineInstrBuilder.h。
- Additional include dependencies: 13 more headers. / 额外包含依赖：还有 13 个头文件。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
