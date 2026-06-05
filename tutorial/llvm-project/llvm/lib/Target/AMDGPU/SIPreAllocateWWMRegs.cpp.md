# SIPreAllocateWWMRegs.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/SIPreAllocateWWMRegs.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements SIPreAllocateWWMRegs for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 SIPreAllocateWWMRegs 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: File banner, includes, and setup
```cpp
//===- SIPreAllocateWWMRegs.cpp - WWM Register Pre-allocation -------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// Pass to pre-allocated WWM registers
//
//===----------------------------------------------------------------------===//

#include "SIPreAllocateWWMRegs.h"
#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "MCTargetDesc/AMDGPUMCTargetDesc.h"
#include "SIMachineFunctionInfo.h"
#include "llvm/ADT/PostOrderIterator.h"
#include "llvm/CodeGen/LiveIntervals.h"
#include "llvm/CodeGen/LiveRegMatrix.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/RegisterClassInfo.h"
#include "llvm/CodeGen/VirtRegMap.h"
#include "llvm/InitializePasses.h"

using namespace llvm;

#define DEBUG_TYPE "si-pre-allocate-wwm-regs"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 32-65: Preprocessor guards and macros
```cpp
static cl::opt<bool>
    EnablePreallocateSGPRSpillVGPRs("amdgpu-prealloc-sgpr-spill-vgprs",
                                    cl::init(false), cl::Hidden);

namespace {

class SIPreAllocateWWMRegs {
private:
  const SIInstrInfo *TII;
  const SIRegisterInfo *TRI;
  MachineRegisterInfo *MRI;
  LiveIntervals *LIS;
  LiveRegMatrix *Matrix;
  VirtRegMap *VRM;
  RegisterClassInfo RegClassInfo;

  std::vector<unsigned> RegsToRewrite;
#ifndef NDEBUG
  void printWWMInfo(const MachineInstr &MI);
#endif
  bool processDef(MachineOperand &MO);
  void rewriteRegs(MachineFunction &MF);

public:
  SIPreAllocateWWMRegs(LiveIntervals *LIS, LiveRegMatrix *Matrix,
                       VirtRegMap *VRM)
      : LIS(LIS), Matrix(Matrix), VRM(VRM) {}
  bool run(MachineFunction &MF);
};

class SIPreAllocateWWMRegsLegacy : public MachineFunctionPass {
public:
  static char ID;

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `SIPreAllocateWWMRegs`, `SIPreAllocateWWMRegsLegacy`, `cl::init`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`SIPreAllocateWWMRegs`, `SIPreAllocateWWMRegsLegacy`, `cl::init`。

### Lines 66-96: Registers LLVM passes
```cpp
  SIPreAllocateWWMRegsLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override;

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<LiveIntervalsWrapperPass>();
    AU.addRequired<VirtRegMapWrapperLegacy>();
    AU.addRequired<LiveRegMatrixWrapperLegacy>();
    AU.setPreservesAll();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};

} // End anonymous namespace.

INITIALIZE_PASS_BEGIN(SIPreAllocateWWMRegsLegacy, DEBUG_TYPE,
                      "SI Pre-allocate WWM Registers", false, false)
INITIALIZE_PASS_DEPENDENCY(LiveIntervalsWrapperPass)
INITIALIZE_PASS_DEPENDENCY(VirtRegMapWrapperLegacy)
INITIALIZE_PASS_DEPENDENCY(LiveRegMatrixWrapperLegacy)
INITIALIZE_PASS_END(SIPreAllocateWWMRegsLegacy, DEBUG_TYPE,
                    "SI Pre-allocate WWM Registers", false, false)

char SIPreAllocateWWMRegsLegacy::ID = 0;

char &llvm::SIPreAllocateWWMRegsLegacyID = SIPreAllocateWWMRegsLegacy::ID;

FunctionPass *llvm::createSIPreAllocateWWMRegsLegacyPass() {
  return new SIPreAllocateWWMRegsLegacy();
}

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `MachineFunctionPass::getAnalysisUsage`, `llvm::createSIPreAllocateWWMRegsLegacyPass`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`MachineFunctionPass::getAnalysisUsage`, `llvm::createSIPreAllocateWWMRegsLegacyPass`。

### Lines 97-129: Implements SIPreAllocateWWMRegs::processDef
```cpp
bool SIPreAllocateWWMRegs::processDef(MachineOperand &MO) {
  Register Reg = MO.getReg();
  if (Reg.isPhysical())
    return false;

  if (!TRI->isVGPR(*MRI, Reg))
    return false;

  if (VRM->hasPhys(Reg))
    return false;

  LiveInterval &LI = LIS->getInterval(Reg);

  for (MCRegister PhysReg : RegClassInfo.getOrder(MRI->getRegClass(Reg))) {
    if (!MRI->isPhysRegUsed(PhysReg, /*SkipRegMaskTest=*/true) &&
        Matrix->checkInterference(LI, PhysReg) == LiveRegMatrix::IK_Free) {
      Matrix->assign(LI, PhysReg);
      assert(PhysReg != 0);
      RegsToRewrite.push_back(Reg);
      return true;
    }
  }

  llvm_unreachable("physreg not found for WWM expression");
}

void SIPreAllocateWWMRegs::rewriteRegs(MachineFunction &MF) {
  for (MachineBasicBlock &MBB : MF) {
    for (MachineInstr &MI : MBB) {
      for (MachineOperand &MO : MI.operands()) {
        if (!MO.isReg())
          continue;

```
**EN:** This section contains concrete logic for SIPreAllocateWWMRegs::processDef. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIPreAllocateWWMRegs::processDef`, `SIPreAllocateWWMRegs::rewriteRegs`.
**CN:** 本节包含与 SIPreAllocateWWMRegs::processDef 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIPreAllocateWWMRegs::processDef`, `SIPreAllocateWWMRegs::rewriteRegs`。

### Lines 130-162: Conditional logic and checks
```cpp
        const Register VirtReg = MO.getReg();
        if (VirtReg.isPhysical())
          continue;

        if (!VirtReg.isValid())
          continue;

        if (!VRM->hasPhys(VirtReg))
          continue;

        Register PhysReg = VRM->getPhys(VirtReg);
        const unsigned SubReg = MO.getSubReg();
        if (SubReg != 0) {
          PhysReg = TRI->getSubReg(PhysReg, SubReg);
          MO.setSubReg(0);
        }

        MO.setReg(PhysReg);
        MO.setIsRenamable(false);
      }
    }
  }

  SIMachineFunctionInfo *MFI = MF.getInfo<SIMachineFunctionInfo>();

  for (unsigned Reg : RegsToRewrite) {
    const Register PhysReg = VRM->getPhys(Reg);
    assert(PhysReg != 0);

    LiveInterval &LI = LIS->getInterval(Reg);
    Matrix->unassign(LI, /*ClearAllReferencingSegments=*/true);
    LIS->removeInterval(Reg);

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 163-196: Preprocessor guards and macros
```cpp
    MFI->reserveWWMRegister(PhysReg);
  }

  RegsToRewrite.clear();

  // Update the set of reserved registers to include WWM ones.
  MRI->freezeReservedRegs();
}

#ifndef NDEBUG
LLVM_DUMP_METHOD void
SIPreAllocateWWMRegs::printWWMInfo(const MachineInstr &MI) {

  unsigned Opc = MI.getOpcode();

  if (Opc == AMDGPU::ENTER_STRICT_WWM || Opc == AMDGPU::ENTER_STRICT_WQM) {
    dbgs() << "Entering ";
  } else {
    assert(Opc == AMDGPU::EXIT_STRICT_WWM || Opc == AMDGPU::EXIT_STRICT_WQM);
    dbgs() << "Exiting ";
  }

  if (Opc == AMDGPU::ENTER_STRICT_WWM || Opc == AMDGPU::EXIT_STRICT_WWM) {
    dbgs() << "Strict WWM ";
  } else {
    assert(Opc == AMDGPU::ENTER_STRICT_WQM || Opc == AMDGPU::EXIT_STRICT_WQM);
    dbgs() << "Strict WQM ";
  }

  dbgs() << "region: " << MI;
}

#endif

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `SIPreAllocateWWMRegs::printWWMInfo`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`SIPreAllocateWWMRegs::printWWMInfo`。

### Lines 197-227: Implements SIPreAllocateWWMRegsLegacy::runOnMachineFunction
```cpp
bool SIPreAllocateWWMRegsLegacy::runOnMachineFunction(MachineFunction &MF) {
  auto *LIS = &getAnalysis<LiveIntervalsWrapperPass>().getLIS();
  auto *Matrix = &getAnalysis<LiveRegMatrixWrapperLegacy>().getLRM();
  auto *VRM = &getAnalysis<VirtRegMapWrapperLegacy>().getVRM();
  return SIPreAllocateWWMRegs(LIS, Matrix, VRM).run(MF);
}

bool SIPreAllocateWWMRegs::run(MachineFunction &MF) {
  LLVM_DEBUG(dbgs() << "SIPreAllocateWWMRegs: function " << MF.getName() << "\n");

  const GCNSubtarget &ST = MF.getSubtarget<GCNSubtarget>();

  TII = ST.getInstrInfo();
  TRI = &TII->getRegisterInfo();
  MRI = &MF.getRegInfo();

  RegClassInfo.runOnMachineFunction(MF);

  bool PreallocateSGPRSpillVGPRs =
      EnablePreallocateSGPRSpillVGPRs ||
      MF.getFunction().hasFnAttribute("amdgpu-prealloc-sgpr-spill-vgprs");

  bool RegsAssigned = false;

  // We use a reverse post-order traversal of the control-flow graph to
  // guarantee that we visit definitions in dominance order. Since WWM
  // expressions are guaranteed to never involve phi nodes, and we can only
  // escape WWM through the special WWM instruction, this means that this is a
  // perfect elimination order, so we can never do any better.
  ReversePostOrderTraversal<MachineFunction*> RPOT(&MF);

```
**EN:** This section contains concrete logic for SIPreAllocateWWMRegsLegacy::runOnMachineFunction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `SIPreAllocateWWMRegsLegacy::runOnMachineFunction`, `SIPreAllocateWWMRegs::run`.
**CN:** 本节包含与 SIPreAllocateWWMRegsLegacy::runOnMachineFunction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`SIPreAllocateWWMRegsLegacy::runOnMachineFunction`, `SIPreAllocateWWMRegs::run`。

### Lines 228-260: Conditional logic and checks
```cpp
  for (MachineBasicBlock *MBB : RPOT) {
    bool InWWM = false;
    for (MachineInstr &MI : *MBB) {
      if (MI.getOpcode() == AMDGPU::SI_SPILL_S32_TO_VGPR) {
        if (PreallocateSGPRSpillVGPRs)
          RegsAssigned |= processDef(MI.getOperand(0));
        continue;
      }

      if (MI.getOpcode() == AMDGPU::ENTER_STRICT_WWM ||
          MI.getOpcode() == AMDGPU::ENTER_STRICT_WQM) {
        LLVM_DEBUG(printWWMInfo(MI));
        InWWM = true;
        continue;
      }

      if (MI.getOpcode() == AMDGPU::EXIT_STRICT_WWM ||
          MI.getOpcode() == AMDGPU::EXIT_STRICT_WQM) {
        LLVM_DEBUG(printWWMInfo(MI));
        InWWM = false;
      }

      if (!InWWM)
        continue;

      LLVM_DEBUG(dbgs() << "Processing " << MI);

      for (MachineOperand &DefOpnd : MI.defs()) {
        RegsAssigned |= processDef(DefOpnd);
      }
    }
  }

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 261-276: Declares rewriteRegs
```cpp
  if (!RegsAssigned)
    return false;

  rewriteRegs(MF);
  return true;
}

PreservedAnalyses
SIPreAllocateWWMRegsPass::run(MachineFunction &MF,
                              MachineFunctionAnalysisManager &MFAM) {
  auto *LIS = &MFAM.getResult<LiveIntervalsAnalysis>(MF);
  auto *Matrix = &MFAM.getResult<LiveRegMatrixAnalysis>(MF);
  auto *VRM = &MFAM.getResult<VirtRegMapAnalysis>(MF);
  SIPreAllocateWWMRegs(LIS, Matrix, VRM).run(MF);
  return PreservedAnalyses::all();
}
```
**EN:** This section declares callable interfaces or named entities that are consumed by the rest of the AMDGPU backend. Main symbols: `SIPreAllocateWWMRegsPass::run`, `PreservedAnalyses::all`.
**CN:** 本节声明可调用接口或具名实体，供 AMDGPU 后端其余部分使用。 主要符号：`SIPreAllocateWWMRegsPass::run`, `PreservedAnalyses::all`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `SIPreAllocateWWMRegs`, `SIPreAllocateWWMRegsLegacy`, `cl::init`, `MachineFunctionPass::getAnalysisUsage`, `llvm::createSIPreAllocateWWMRegsLegacyPass`, `SIPreAllocateWWMRegs::processDef`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; LLVM pass integration / LLVM Pass 集成
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"SIPreAllocateWWMRegs.h"`
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"MCTargetDesc/AMDGPUMCTargetDesc.h"`
- `"SIMachineFunctionInfo.h"`
- `"llvm/ADT/PostOrderIterator.h"`
- `"llvm/CodeGen/LiveIntervals.h"`
- `"llvm/CodeGen/LiveRegMatrix.h"`
- `"llvm/CodeGen/MachineFrameInfo.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
- `"llvm/CodeGen/RegisterClassInfo.h"`
- `"llvm/CodeGen/VirtRegMap.h"`
- `"llvm/InitializePasses.h"`
