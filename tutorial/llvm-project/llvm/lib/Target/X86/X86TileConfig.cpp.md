# X86TileConfig.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86TileConfig.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements tile configuration handling for the core X86 backend. / 实现X86 后端核心中的tile 配置处理。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===-- X86TileConfig.cpp - Tile Register Configure----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file Pass to config the shape of AMX physical registers
/// AMX register need to be configured before use. In X86PreTileConfig pass
/// the pldtilecfg instruction is inserted, however at that time we don't
/// know the shape of each physical tile registers, because the register
/// allocation is not done yet. This pass runs after egister allocation
/// pass. It collects the shape information of each physical tile register
/// and store the shape in the stack slot that is allocated for load config
/// to tile config register.
//
//===----------------------------------------------------------------------===//

#include "X86.h"
#include "X86InstrBuilder.h"
#include "X86MachineFunctionInfo.h"
#include "X86Subtarget.h"
#include "llvm/CodeGen/LiveIntervals.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/Passes.h"
#include "llvm/CodeGen/TargetInstrInfo.h"
```
**EN:** This section implements file header, licensing, and opening context for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的文件头、许可证与开场上下文。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 31-60: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/CodeGen/TileShapeInfo.h"
#include "llvm/CodeGen/VirtRegMap.h"
#include "llvm/InitializePasses.h"

using namespace llvm;

#define DEBUG_TYPE "x86-tile-config"

namespace {

struct X86TileConfigLegacy : public MachineFunctionPass {

  X86TileConfigLegacy() : MachineFunctionPass(ID) {}

  /// Return the pass name.
  StringRef getPassName() const override { return "Tile Register Configure"; }

  /// X86TileConfig analysis usage.
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesAll();
    AU.addRequired<VirtRegMapWrapperLegacy>();
    AU.addRequired<LiveIntervalsWrapperPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }

  /// Perform register allocation.
  bool runOnMachineFunction(MachineFunction &MF) override;

  MachineFunctionProperties getRequiredProperties() const override {
```
**EN:** This section implements preprocessor directives and structural setup for the core X86 backend. Key symbols include X86TileConfigLegacy. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的预处理指令与结构初始化。关键符号包括 X86TileConfigLegacy。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 61-90: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
    return MachineFunctionProperties().setNoPHIs();
  }

  static char ID;
};

} // end anonymous namespace

char X86TileConfigLegacy::ID = 0;

INITIALIZE_PASS_BEGIN(X86TileConfigLegacy, DEBUG_TYPE,
                      "Tile Register Configure", false, false)
INITIALIZE_PASS_DEPENDENCY(VirtRegMapWrapperLegacy)
INITIALIZE_PASS_END(X86TileConfigLegacy, DEBUG_TYPE, "Tile Register Configure",
                    false, false)

static bool tileConfig(MachineFunction &MF,
                       llvm::function_ref<LiveIntervals *()> GetLIs,
                       llvm::function_ref<VirtRegMap *()> GetVRM) {
  X86MachineFunctionInfo *X86FI = MF.getInfo<X86MachineFunctionInfo>();
  // Early exit in the common case of non-AMX code.
  if (X86FI->getAMXProgModel() != AMXProgModelEnum::ManagedRA)
    return false;

  const X86Subtarget &ST = MF.getSubtarget<X86Subtarget>();
  const X86RegisterInfo *TRI = ST.getRegisterInfo();
  const TargetInstrInfo *TII = ST.getInstrInfo();
  MachineRegisterInfo &MRI = MF.getRegInfo();
  LiveIntervals &LIS = *GetLIs();
  VirtRegMap &VRM = *GetVRM();
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 91-120: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp

  if (VRM.isShapeMapEmpty())
    return false;

  int SS = INT_MAX;
  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : MBB) {
      if (MI.getOpcode() == X86::PLDTILECFGV) {
        SS = MI.getOperand(0).getIndex();
        break;
      }
    }
    if (SS != INT_MAX)
      break;
  }
  // Didn't find PLDTILECFGV, just return false;
  if (SS == INT_MAX)
    return false;

  // Try to find a point to insert MIs for constant shapes.
  // Here we are leveraging the palette id inserted in PreRA pass.
  unsigned ConstPos = 0;
  MachineInstr *ConstMI = nullptr;
  for (MachineInstr &MI : MF.front()) {
    if (MI.getOpcode() == X86::MOV8mi && SS == MI.getOperand(0).getIndex()) {
      ConstMI = &MI;
      break;
    }
    ++ConstPos;
  }
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 121-150: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
  assert(ConstMI && "Cannot find an insertion point");

  unsigned AMXRegNum = TRI->getRegClass(X86::TILERegClassID)->getNumRegs();
  SmallVector<Register, 8> Phys2Virt(AMXRegNum, 0);
  for (unsigned I = 0, E = MRI.getNumVirtRegs(); I != E; ++I) {
    Register VirtReg = Register::index2VirtReg(I);
    if (MRI.reg_nodbg_empty(VirtReg))
      continue;
    if (!TRI->isTileRegisterClass(MRI.getRegClass(VirtReg)))
      continue;
    MCRegister PhysReg = VRM.getPhys(VirtReg);
    if (!PhysReg)
      continue;
    unsigned Index = PhysReg - X86::TMM0;
    if (!Phys2Virt[Index])
      Phys2Virt[Index] = VirtReg;
  }

  // Fill in the shape of each tile physical register.
  for (unsigned I = 0; I < AMXRegNum; ++I) {
    if (!Phys2Virt[I])
      continue;
    DebugLoc DL;
    bool IsRow = true;
    MachineInstr *NewMI = nullptr;
    ShapeT Shape = VRM.getShape(Phys2Virt[I]);
    for (auto &R : {Shape.getRow()->getReg(), Shape.getCol()->getReg()}) {
      // Here is the data format for the tile config.
      // 0      palette
      // 1      start_row
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 151-180: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
      // 2-15   reserved, must be zero
      // 16-17  tile0.colsb Tile 0 bytes per row.
      // 18-19  tile1.colsb Tile 1 bytes per row.
      // 20-21  tile2.colsb Tile 2 bytes per row.
      // ... (sequence continues)
      // 30-31  tile7.colsb Tile 7 bytes per row.
      // 32-47  reserved, must be zero
      // 48     tile0.rows Tile 0 rows.
      // 49     tile1.rows Tile 1 rows.
      // 50     tile2.rows Tile 2 rows.
      // ... (sequence continues)
      // 55     tile7.rows Tile 7 rows.
      // 56-63  reserved, must be zero
      int64_t Imm = INT64_MAX;
      int Offset = IsRow ? 48 + I : 16 + I * 2;
      for (auto &DefMI : MRI.def_instructions(R)) {
        MachineBasicBlock &MBB = *DefMI.getParent();
        if (DefMI.isMoveImmediate()) {
          if (Imm != INT64_MAX) {
            // FIXME: We should handle this case in future.
            assert(Imm == DefMI.getOperand(1).getImm() &&
                   "Cannot initialize with different shapes");
            continue;
          }
          if (DefMI.getOperand(1).isImm()) {
            Imm = DefMI.getOperand(1).getImm();
          } else {
            assert(DefMI.getOpcode() == X86::MOV32r0 &&
                   "The opcode is assumed to be MOV32r0 if the operand is not "
                   "immediate.");
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 181-210: Control flow, matching, and data updates / 控制流、匹配与数据更新
```cpp
            Imm = 0;
          }

          NewMI = addFrameReference(
                      BuildMI(MF.front(), ++ConstMI->getIterator(), DL,
                              TII->get(IsRow ? X86::MOV8mi : X86::MOV16mi)),
                      SS, Offset)
                      .addImm(Imm);
          ConstMI = NewMI;
          LIS.InsertMachineInstrInMaps(*NewMI);
        } else {
          unsigned SubIdx = IsRow ? X86::sub_8bit : X86::sub_16bit;
          unsigned RegSize = TRI->getRegSizeInBits(*MRI.getRegClass(R));
          if ((IsRow && RegSize == 8) || (!IsRow && RegSize == 16))
            SubIdx = 0;
          auto Iter = DefMI.getIterator();
          if (&MBB == &MF.front() &&
              (unsigned)std::distance(MBB.instr_begin(), Iter) < ConstPos)
            Iter = ConstMI->getIterator();
          NewMI = addFrameReference(
                      BuildMI(MBB, ++Iter, DL,
                              TII->get(IsRow ? X86::MOV8mr : X86::MOV16mr)),
                      SS, Offset)
                      .addReg(R, {}, SubIdx);
          SlotIndex SIdx = LIS.InsertMachineInstrInMaps(*NewMI);
          LIS.extendToIndices(LIS.getInterval(R), {SIdx.getRegSlot()});
        }
      }
      IsRow = false;
    }
```
**EN:** This section implements control flow, matching, and data updates for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的控制流、匹配与数据更新。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

### Lines 211-234: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  }
  return true;
}

FunctionPass *llvm::createX86TileConfigLegacyPass() {
  return new X86TileConfigLegacy();
}

bool X86TileConfigLegacy::runOnMachineFunction(MachineFunction &MF) {
  return tileConfig(
      MF,
      [this]() { return &getAnalysis<LiveIntervalsWrapperPass>().getLIS(); },
      [this]() { return &getAnalysis<VirtRegMapWrapperLegacy>().getVRM(); });
}

PreservedAnalyses X86TileConfigPass::run(MachineFunction &MF,
                                         MachineFunctionAnalysisManager &MFAM) {
  bool Changed = tileConfig(
      MF, [&MFAM, &MF]() { return &MFAM.getResult<LiveIntervalsAnalysis>(MF); },
      [&MFAM, &MF]() { return &MFAM.getResult<VirtRegMapAnalysis>(MF); });
  return Changed ? getMachineFunctionPassPreservedAnalyses()
                       .preserveSet<CFGAnalyses>()
                 : PreservedAnalyses::all();
}
```
**EN:** This section implements function implementation and target-specific behavior for the core X86 backend. It shows how LLVM specializes generic code-generation infrastructure for X86.

**CN:** 本段实现了X86 后端核心中的函数实现与目标特定行为。它展示了 LLVM 如何为 X86 特化通用代码生成基础设施。

## Key Concepts / 关键概念
- Primary topic: tile configuration handling. / 核心主题：tile 配置处理。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86TileConfigLegacy. / 重要符号：X86TileConfigLegacy。
- The file emphasizes executable backend logic and target-specific decision making. / 该文件以可执行后端逻辑和目标特定决策为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: X86.h, X86InstrBuilder.h, X86MachineFunctionInfo.h, X86Subtarget.h, llvm/CodeGen/LiveIntervals.h, llvm/CodeGen/MachineFrameInfo.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/CodeGen/Passes.h. / 直接包含：X86.h, X86InstrBuilder.h, X86MachineFunctionInfo.h, X86Subtarget.h, llvm/CodeGen/LiveIntervals.h, llvm/CodeGen/MachineFrameInfo.h, llvm/CodeGen/MachineFunctionPass.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineRegisterInfo.h, llvm/CodeGen/Passes.h。
- Additional include dependencies: 5 more headers. / 额外包含依赖：还有 5 个头文件。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
