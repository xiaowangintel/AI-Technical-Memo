# X86FastTileConfig.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86FastTileConfig.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements tile configuration handling for the core X86 backend. / 实现X86 后端核心中的tile 配置处理。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===-- X86FastTileConfig.cpp - Fast Tile Register Configure---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file Pass to config the shape of AMX physical registers
/// AMX register need to be configured before use. Before FastRegAllocation pass
/// the ldtilecfg instruction is inserted, however at that time we don't
/// know the shape of each physical tile registers, because the register
/// allocation is not done yet. This pass runs after register allocation
/// pass. It collects the shape information of each physical tile register
/// and store the shape in the stack slot that is allocated for load config
/// to tile config register.
//
//===----------------------------------------------------------------------===//

#include "X86.h"
#include "X86InstrBuilder.h"
#include "X86MachineFunctionInfo.h"
#include "X86Subtarget.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp

using namespace llvm;

#define DEBUG_TYPE "x86-fast-tile-config"

namespace {

class X86FastTileConfigImpl {
public:
  bool runOnMachineFunction(MachineFunction &MF);

private:
  // context
  MachineFunction *MF = nullptr;
  const TargetInstrInfo *TII = nullptr;
  MachineRegisterInfo *MRI = nullptr;
  const TargetRegisterInfo *TRI = nullptr;
  X86MachineFunctionInfo *X86FI = nullptr;

  bool configBasicBlock(MachineBasicBlock &MBB);
};

class X86FastTileConfigLegacy : public MachineFunctionPass {
public:
  X86FastTileConfigLegacy() : MachineFunctionPass(ID) {}

  /// Return the pass name.
  StringRef getPassName() const override {
    return "Fast Tile Register Configure";
  }
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. Key symbols include X86FastTileConfigImpl, X86FastTileConfigLegacy. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。关键符号包括 X86FastTileConfigImpl, X86FastTileConfigLegacy。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesAll();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  /// Perform register allocation.
  bool runOnMachineFunction(MachineFunction &MFunc) override;

  MachineFunctionProperties getRequiredProperties() const override {
    return MachineFunctionProperties().setNoPHIs();
  }

  static char ID;
};

} // end anonymous namespace

char X86FastTileConfigLegacy::ID = 0;

INITIALIZE_PASS_BEGIN(X86FastTileConfigLegacy, DEBUG_TYPE,
                      "Fast Tile Register Configure", false, false)
INITIALIZE_PASS_END(X86FastTileConfigLegacy, DEBUG_TYPE,
                    "Fast Tile Register Configure", false, false)

static bool isTileDef(MachineRegisterInfo *MRI, MachineInstr &MI) {
  // There is no phi instruction after register allocation.
  assert(MI.isPHI() == false);
  // The instruction must have 3 operands: tile def, row, col.
  // It should be AMX pseudo instruction that have shape operand.
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  if (MI.isDebugInstr() || MI.isCopy() || MI.getNumOperands() < 3 ||
      !MI.isPseudo())
    return false;
  MachineOperand &MO = MI.getOperand(0);

  if (MO.isReg()) {
    Register Reg = MO.getReg();
    // FIXME: It may be used after Greedy RA and the physical
    // register is not rewritten yet.
    if (Reg.isVirtual()) {
      if (MRI->getRegClass(Reg)->getID() == X86::TILERegClassID)
        return true;
    }
    if (Reg >= X86::TMM0 && Reg <= X86::TMM7)
      return true;
  }

  return false;
}

static unsigned getTMMIndex(Register Reg) {
  if (Reg >= X86::TMM0 && Reg <= X86::TMM7)
    return Reg - X86::TMM0;
  llvm_unreachable("Invalid Tmm Reg!");
}

// PreTileConfig should configure the tile registers based on basic
// block.
bool X86FastTileConfigImpl::configBasicBlock(MachineBasicBlock &MBB) {
  bool Change = false;
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-150: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
  SmallVector<std::pair<unsigned, ShapeT>, 6> ShapeInfos;
  for (MachineInstr &MI : reverse(MBB)) {
    if (!isTileDef(MRI, MI) && MI.getOpcode() != X86::PLDTILECFGV)
      continue;
    // AMX instructions that define tile register.
    if (MI.getOpcode() != X86::PLDTILECFGV) {
      MachineOperand &Row = MI.getOperand(1);
      unsigned TMMIdx = getTMMIndex(MI.getOperand(0).getReg());
      MachineOperand &Col = MI.getOperand(2);
      ShapeInfos.push_back({TMMIdx, ShapeT(&Row, &Col)});
    } else { // PLDTILECFGV
      // Rewrite the shape information to memory. Stack slot should have
      // been initialized to zero in pre config.
      int SS = MI.getOperand(0).getIndex(); // tile config stack slot.
      for (auto &ShapeInfo : ShapeInfos) {
        DebugLoc DL;
        unsigned TMMIdx = ShapeInfo.first;
        Register RowReg = ShapeInfo.second.getRow()->getReg();
        Register ColReg = ShapeInfo.second.getCol()->getReg();
        // Here is the data format for the tile config.
        // 0      palette
        // 1      start_row
        // 2-15   reserved, must be zero
        // 16-17  tile0.colsb Tile 0 bytes per row.
        // 18-19  tile1.colsb Tile 1 bytes per row.
        // 20-21  tile2.colsb Tile 2 bytes per row.
        // ... (sequence continues)
        // 30-31  tile7.colsb Tile 7 bytes per row.
        // 32-47  reserved, must be zero
        // 48     tile0.rows Tile 0 rows.
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-180: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
        // 49     tile1.rows Tile 1 rows.
        // 50     tile2.rows Tile 2 rows.
        // ... (sequence continues)
        // 55     tile7.rows Tile 7 rows.
        // 56-63  reserved, must be zero
        int RowOffset = 48 + TMMIdx;
        int ColOffset = 16 + TMMIdx * 2;

        Register SubRowReg = TRI->getSubReg(RowReg, X86::sub_8bit);
        BuildMI(MBB, MI, DL, TII->get(X86::IMPLICIT_DEF), SubRowReg);
        MachineInstrBuilder StoreRow =
            BuildMI(MBB, MI, DL, TII->get(X86::MOV8mr));
        addFrameReference(StoreRow, SS, RowOffset).addReg(SubRowReg);

        MachineInstrBuilder StoreCol =
            BuildMI(MBB, MI, DL, TII->get(X86::MOV16mr));
        addFrameReference(StoreCol, SS, ColOffset).addReg(ColReg);
      }
      ShapeInfos.clear();
      Change = true;
    }
  }

  return Change;
}

bool X86FastTileConfigImpl::runOnMachineFunction(MachineFunction &MFunc) {
  X86FI = MFunc.getInfo<X86MachineFunctionInfo>();
  // Early exit in the common case of non-AMX code.
  if (X86FI->getAMXProgModel() != AMXProgModelEnum::ManagedRA)
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 181-210: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
    return false;

  MF = &MFunc;
  MRI = &MFunc.getRegInfo();
  const TargetSubtargetInfo *ST = &MFunc.getSubtarget<X86Subtarget>();
  TRI = ST->getRegisterInfo();
  TII = MFunc.getSubtarget().getInstrInfo();
  bool Change = false;

  // Loop over all of the basic blocks, eliminating virtual register references
  for (MachineBasicBlock &MBB : MFunc)
    Change |= configBasicBlock(MBB);

  return Change;
}

FunctionPass *llvm::createX86FastTileConfigLegacyPass() {
  return new X86FastTileConfigLegacy();
}

bool X86FastTileConfigLegacy::runOnMachineFunction(MachineFunction &MF) {
  X86FastTileConfigImpl Impl;
  return Impl.runOnMachineFunction(MF);
}

PreservedAnalyses
X86FastTileConfigPass::run(MachineFunction &MF,
                           MachineFunctionAnalysisManager &MFAM) {
  X86FastTileConfigImpl Impl;
  Impl.runOnMachineFunction(MF);
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 211-212: Supporting logic and local data handling / 辅助逻辑与局部数据处理
```cpp
  return getMachineFunctionPassPreservedAnalyses().preserveSet<CFGAnalyses>();
}
```
**EN:** This section implements supporting logic and local data handling for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的辅助逻辑与局部数据处理。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: tile configuration handling. / 核心主题：tile 配置处理。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86FastTileConfigImpl, X86FastTileConfigLegacy. / 重要符号：X86FastTileConfigImpl, X86FastTileConfigLegacy。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86InstrBuilder.h, X86MachineFunctionInfo.h, X86Subtarget.h, llvm/CodeGen/MachineFrameInfo.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/CodeGen/Passes.h, llvm/CodeGen/TargetInstrInfo.h. / 直接包含：X86.h, X86InstrBuilder.h, X86MachineFunctionInfo.h, X86Subtarget.h, llvm/CodeGen/MachineFrameInfo.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/CodeGen/Passes.h, llvm/CodeGen/TargetInstrInfo.h。
- Additional include dependencies: 1 more headers. / 额外包含依赖：还有 1 个头文件。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
