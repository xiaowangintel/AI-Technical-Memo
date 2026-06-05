# GCNPreRALongBranchReg.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/GCNPreRALongBranchReg.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements GCNPreRALongBranchReg for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 GCNPreRALongBranchReg 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: File banner, includes, and setup
```cpp
//===-- GCNPreRALongBranchReg.cpp ----------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
// \file
// \brief Pass to estimate pre RA branch size and reserve a pair of SGPRs if
// there is a long branch. Branch size at this point is difficult to track since
// we have no idea what spills will be inserted later on. We just assume 8 bytes
// per instruction to compute approximations without computing the actual
// instruction size to see if we're in the neighborhood of the maximum branch
// distrance threshold tuning of what is considered "long" is handled through
// amdgpu-long-branch-factor cl argument which sets LongBranchFactor.
//===----------------------------------------------------------------------===//
#include "GCNPreRALongBranchReg.h"
#include "AMDGPU.h"
#include "GCNSubtarget.h"
#include "SIMachineFunctionInfo.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/InitializePasses.h"

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 24-41: Preprocessor guards and macros
```cpp
using namespace llvm;

#define DEBUG_TYPE "amdgpu-pre-ra-long-branch-reg"

namespace {

static cl::opt<double> LongBranchFactor(
    "amdgpu-long-branch-factor", cl::init(1.0), cl::Hidden,
    cl::desc("Factor to apply to what qualifies as a long branch "
             "to reserve a pair of scalar registers. If this value "
             "is 0 the long branch registers are never reserved. As this "
             "value grows the greater chance the branch distance will fall "
             "within the threshold and the registers will be marked to be "
             "reserved. We lean towards always reserving a register for  "
             "long jumps"));

class GCNPreRALongBranchReg {

```
**EN:** These lines define compile-time structure such as include guards, feature switches, or macros that shape how the file is compiled. Main symbols: `GCNPreRALongBranchReg`, `cl::init`, `cl::desc`.
**CN:** 这些语句定义了编译期结构，例如 include guard、特性开关或宏，用于决定文件的编译方式。 主要符号：`GCNPreRALongBranchReg`, `cl::init`, `cl::desc`。

### Lines 42-65: Declares struct BasicBlockInfo
```cpp
  struct BasicBlockInfo {
    // Offset - Distance from the beginning of the function to the beginning
    // of this basic block.
    uint64_t Offset = 0;
    // Size - Size of the basic block in bytes
    uint64_t Size = 0;
  };
  void generateBlockInfo(MachineFunction &MF,
                         SmallVectorImpl<BasicBlockInfo> &BlockInfo);

public:
  GCNPreRALongBranchReg() = default;
  bool run(MachineFunction &MF);
};

class GCNPreRALongBranchRegLegacy : public MachineFunctionPass {
public:
  static char ID;
  GCNPreRALongBranchRegLegacy() : MachineFunctionPass(ID) {}

  bool runOnMachineFunction(MachineFunction &MF) override {
    return GCNPreRALongBranchReg().run(MF);
  }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `BasicBlockInfo`, `GCNPreRALongBranchRegLegacy`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`BasicBlockInfo`, `GCNPreRALongBranchRegLegacy`。

### Lines 66-87: Registers LLVM passes
```cpp
  StringRef getPassName() const override {
    return "AMDGPU Pre-RA Long Branch Reg";
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesAll();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
};
} // End anonymous namespace.

char GCNPreRALongBranchRegLegacy::ID = 0;

INITIALIZE_PASS(GCNPreRALongBranchRegLegacy, DEBUG_TYPE,
                "AMDGPU Pre-RA Long Branch Reg", false, false)

char &llvm::GCNPreRALongBranchRegID = GCNPreRALongBranchRegLegacy::ID;
void GCNPreRALongBranchReg::generateBlockInfo(
    MachineFunction &MF, SmallVectorImpl<BasicBlockInfo> &BlockInfo) {

  BlockInfo.resize(MF.getNumBlockIDs());

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `MachineFunctionPass::getAnalysisUsage`, `GCNPreRALongBranchReg::generateBlockInfo`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`MachineFunctionPass::getAnalysisUsage`, `GCNPreRALongBranchReg::generateBlockInfo`。

### Lines 88-111: Conditional logic and checks
```cpp
  // Approximate the size of all basic blocks by just
  // assuming 8 bytes per instruction
  for (const MachineBasicBlock &MBB : MF) {
    uint64_t NumInstr = 0;
    // Loop through the basic block and add up all non-debug
    // non-meta instructions
    for (const MachineInstr &MI : MBB) {
      // isMetaInstruction is a superset of isDebugIstr
      if (MI.isMetaInstruction())
        continue;
      NumInstr += 1;
    }
    // Approximate size as just 8 bytes per instruction
    BlockInfo[MBB.getNumber()].Size = 8 * NumInstr;
  }
  uint64_t PrevNum = (&MF)->begin()->getNumber();
  for (auto &MBB :
       make_range(std::next(MachineFunction::iterator((&MF)->begin())),
                  (&MF)->end())) {
    uint64_t Num = MBB.getNumber();
    // Compute the offset immediately following this block.
    BlockInfo[Num].Offset = BlockInfo[PrevNum].Offset + BlockInfo[PrevNum].Size;
    PrevNum = Num;
  }
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `std::next`, `MachineFunction::iterator`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`std::next`, `MachineFunction::iterator`。

### Lines 112-134: Implements GCNPreRALongBranchReg::run
```cpp
}

bool GCNPreRALongBranchReg::run(MachineFunction &MF) {
  const GCNSubtarget &STM = MF.getSubtarget<GCNSubtarget>();
  const SIInstrInfo *TII = STM.getInstrInfo();
  const SIRegisterInfo *TRI = STM.getRegisterInfo();
  SIMachineFunctionInfo *MFI = MF.getInfo<SIMachineFunctionInfo>();
  MachineRegisterInfo &MRI = MF.getRegInfo();

  // For now, reserve highest available SGPR pair. After RA,
  // shift down to a lower unused pair of SGPRs
  // If all registers are used, then findUnusedRegister will return
  // AMDGPU::NoRegister.
  constexpr bool ReserveHighestRegister = true;
  Register LongBranchReservedReg = TRI->findUnusedRegister(
      MRI, &AMDGPU::SGPR_64RegClass, MF, ReserveHighestRegister);
  if (!LongBranchReservedReg)
    return false;

  // Approximate code size and offsets of each basic block
  SmallVector<BasicBlockInfo, 16> BlockInfo;
  generateBlockInfo(MF, BlockInfo);

```
**EN:** This section contains concrete logic for GCNPreRALongBranchReg::run. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `GCNPreRALongBranchReg::run`.
**CN:** 本节包含与 GCNPreRALongBranchReg::run 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`GCNPreRALongBranchReg::run`。

### Lines 135-157: Conditional logic and checks
```cpp
  for (const MachineBasicBlock &MBB : MF) {
    MachineBasicBlock::const_iterator Last = MBB.getLastNonDebugInstr();
    if (Last == MBB.end() || !Last->isUnconditionalBranch())
      continue;
    MachineBasicBlock *DestBB = TII->getBranchDestBlock(*Last);
    uint64_t BlockDistance = static_cast<uint64_t>(
        LongBranchFactor * BlockInfo[DestBB->getNumber()].Offset);
    // If the distance falls outside the threshold assume it is a long branch
    // and we need to reserve the registers
    if (!TII->isBranchOffsetInRange(Last->getOpcode(), BlockDistance)) {
      MFI->setLongBranchReservedReg(LongBranchReservedReg);
      return true;
    }
  }
  return false;
}

PreservedAnalyses
GCNPreRALongBranchRegPass::run(MachineFunction &MF,
                               MachineFunctionAnalysisManager &MFAM) {
  GCNPreRALongBranchReg().run(MF);
  return PreservedAnalyses::all();
}
```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties. Main symbols: `GCNPreRALongBranchRegPass::run`, `PreservedAnalyses::all`.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。 主要符号：`GCNPreRALongBranchRegPass::run`, `PreservedAnalyses::all`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `GCNPreRALongBranchReg`, `BasicBlockInfo`, `GCNPreRALongBranchRegLegacy`, `cl::init`, `cl::desc`, `MachineFunctionPass::getAnalysisUsage`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; lowering / 降低
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"GCNPreRALongBranchReg.h"`
- `"AMDGPU.h"`
- `"GCNSubtarget.h"`
- `"SIMachineFunctionInfo.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
- `"llvm/InitializePasses.h"`
