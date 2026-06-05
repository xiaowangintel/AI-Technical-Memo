# R600ClauseMergePass.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/R600ClauseMergePass.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This source file implements R600ClauseMergePass for the LLVM AMDGPU backend. It contains target-specific logic used during analysis, lowering, code generation, or pass execution. / 该源文件实现 LLVM AMDGPU 后端中的 R600ClauseMergePass 相关功能。它包含分析、降低、代码生成或 Pass 执行过程中使用的目标专用逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: File banner, includes, and setup
```cpp
//===-- R600ClauseMergePass - Merge consecutive CF_ALU -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// R600EmitClauseMarker pass emits CFAlu instruction in a conservative manner.
/// This pass is merging consecutive CFAlus where applicable.
/// It needs to be called after IfCvt for best results.
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/R600MCTargetDesc.h"
#include "R600.h"
#include "R600Subtarget.h"
#include "llvm/CodeGen/MachineFunctionPass.h"

using namespace llvm;

#define DEBUG_TYPE "r600mergeclause"

namespace {

```
**EN:** This opening section identifies the file, documents its intent, and imports the declarations needed by the remaining implementation.
**CN:** 开头部分给出文件身份与总体意图，并导入后续实现所需的声明。

### Lines 26-59: Declares class R600ClauseMergePass
```cpp
static bool isCFAlu(const MachineInstr &MI) {
  switch (MI.getOpcode()) {
  case R600::CF_ALU:
  case R600::CF_ALU_PUSH_BEFORE:
    return true;
  default:
    return false;
  }
}

class R600ClauseMergePass : public MachineFunctionPass {

private:
  const R600InstrInfo *TII;

  unsigned getCFAluSize(const MachineInstr &MI) const;
  bool isCFAluEnabled(const MachineInstr &MI) const;

  /// IfCvt pass can generate "disabled" ALU clause marker that need to be
  /// removed and their content affected to the previous alu clause.
  /// This function parse instructions after CFAlu until it find a disabled
  /// CFAlu and merge the content, or an enabled CFAlu.
  void cleanPotentialDisabledCFAlu(MachineInstr &CFAlu) const;

  /// Check whether LatrCFAlu can be merged into RootCFAlu and do it if
  /// it is the case.
  bool mergeIfPossible(MachineInstr &RootCFAlu,
                       const MachineInstr &LatrCFAlu) const;

public:
  static char ID;

  R600ClauseMergePass() : MachineFunctionPass(ID) { }

```
**EN:** This section introduces a core type and defines the interface or stored state that other backend code will use. Main symbols: `R600ClauseMergePass`.
**CN:** 本节引入核心类型，并定义其他后端代码会依赖的接口或内部状态。 主要符号：`R600ClauseMergePass`。

### Lines 60-89: Registers LLVM passes
```cpp
  bool runOnMachineFunction(MachineFunction &MF) override;

  StringRef getPassName() const override;
};

} // end anonymous namespace

INITIALIZE_PASS_BEGIN(R600ClauseMergePass, DEBUG_TYPE,
                      "R600 Clause Merge", false, false)
INITIALIZE_PASS_END(R600ClauseMergePass, DEBUG_TYPE,
                    "R600 Clause Merge", false, false)

char R600ClauseMergePass::ID = 0;

char &llvm::R600ClauseMergePassID = R600ClauseMergePass::ID;

unsigned R600ClauseMergePass::getCFAluSize(const MachineInstr &MI) const {
  assert(isCFAlu(MI));
  return MI
      .getOperand(TII->getOperandIdx(MI.getOpcode(), R600::OpName::COUNT))
      .getImm();
}

bool R600ClauseMergePass::isCFAluEnabled(const MachineInstr &MI) const {
  assert(isCFAlu(MI));
  return MI
      .getOperand(TII->getOperandIdx(MI.getOpcode(), R600::OpName::Enabled))
      .getImm();
}

```
**EN:** This section connects the implementation to LLVM pass registration so the AMDGPU-specific analysis or transformation can be constructed by the pass framework. Main symbols: `R600ClauseMergePass::getCFAluSize`, `R600ClauseMergePass::isCFAluEnabled`.
**CN:** 本节把实现接入 LLVM 的 Pass 注册机制，使 AMDGPU 专用分析或变换能够被 Pass 框架创建和调用。 主要符号：`R600ClauseMergePass::getCFAluSize`, `R600ClauseMergePass::isCFAluEnabled`。

### Lines 90-107: Implements R600ClauseMergePass::cleanPotentialDisabledCFAlu
```cpp
void R600ClauseMergePass::cleanPotentialDisabledCFAlu(
    MachineInstr &CFAlu) const {
  int CntIdx = TII->getOperandIdx(R600::CF_ALU, R600::OpName::COUNT);
  MachineBasicBlock::iterator I = CFAlu, E = CFAlu.getParent()->end();
  I++;
  do {
    while (I != E && !isCFAlu(*I))
      I++;
    if (I == E)
      return;
    MachineInstr &MI = *I++;
    if (isCFAluEnabled(MI))
      break;
    CFAlu.getOperand(CntIdx).setImm(getCFAluSize(CFAlu) + getCFAluSize(MI));
    MI.eraseFromParent();
  } while (I != E);
}

```
**EN:** This section contains concrete logic for R600ClauseMergePass::cleanPotentialDisabledCFAlu. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600ClauseMergePass::cleanPotentialDisabledCFAlu`.
**CN:** 本节包含与 R600ClauseMergePass::cleanPotentialDisabledCFAlu 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600ClauseMergePass::cleanPotentialDisabledCFAlu`。

### Lines 108-141: Implements R600ClauseMergePass::mergeIfPossible
```cpp
bool R600ClauseMergePass::mergeIfPossible(MachineInstr &RootCFAlu,
                                          const MachineInstr &LatrCFAlu) const {
  assert(isCFAlu(RootCFAlu) && isCFAlu(LatrCFAlu));
  int CntIdx = TII->getOperandIdx(R600::CF_ALU, R600::OpName::COUNT);
  unsigned RootInstCount = getCFAluSize(RootCFAlu),
      LaterInstCount = getCFAluSize(LatrCFAlu);
  unsigned CumuledInsts = RootInstCount + LaterInstCount;
  if (CumuledInsts >= TII->getMaxAlusPerClause()) {
    LLVM_DEBUG(dbgs() << "Excess inst counts\n");
    return false;
  }
  if (RootCFAlu.getOpcode() == R600::CF_ALU_PUSH_BEFORE)
    return false;
  // Is KCache Bank 0 compatible ?
  int Mode0Idx =
      TII->getOperandIdx(R600::CF_ALU, R600::OpName::KCACHE_MODE0);
  int KBank0Idx =
      TII->getOperandIdx(R600::CF_ALU, R600::OpName::KCACHE_BANK0);
  int KBank0LineIdx =
      TII->getOperandIdx(R600::CF_ALU, R600::OpName::KCACHE_ADDR0);
  if (LatrCFAlu.getOperand(Mode0Idx).getImm() &&
      RootCFAlu.getOperand(Mode0Idx).getImm() &&
      (LatrCFAlu.getOperand(KBank0Idx).getImm() !=
           RootCFAlu.getOperand(KBank0Idx).getImm() ||
       LatrCFAlu.getOperand(KBank0LineIdx).getImm() !=
           RootCFAlu.getOperand(KBank0LineIdx).getImm())) {
    LLVM_DEBUG(dbgs() << "Wrong KC0\n");
    return false;
  }
  // Is KCache Bank 1 compatible ?
  int Mode1Idx =
      TII->getOperandIdx(R600::CF_ALU, R600::OpName::KCACHE_MODE1);
  int KBank1Idx =
      TII->getOperandIdx(R600::CF_ALU, R600::OpName::KCACHE_BANK1);
```
**EN:** This section contains concrete logic for R600ClauseMergePass::mergeIfPossible. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600ClauseMergePass::mergeIfPossible`.
**CN:** 本节包含与 R600ClauseMergePass::mergeIfPossible 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600ClauseMergePass::mergeIfPossible`。

### Lines 142-173: Conditional logic and checks
```cpp
  int KBank1LineIdx =
      TII->getOperandIdx(R600::CF_ALU, R600::OpName::KCACHE_ADDR1);
  if (LatrCFAlu.getOperand(Mode1Idx).getImm() &&
      RootCFAlu.getOperand(Mode1Idx).getImm() &&
      (LatrCFAlu.getOperand(KBank1Idx).getImm() !=
           RootCFAlu.getOperand(KBank1Idx).getImm() ||
       LatrCFAlu.getOperand(KBank1LineIdx).getImm() !=
           RootCFAlu.getOperand(KBank1LineIdx).getImm())) {
    LLVM_DEBUG(dbgs() << "Wrong KC0\n");
    return false;
  }
  if (LatrCFAlu.getOperand(Mode0Idx).getImm()) {
    RootCFAlu.getOperand(Mode0Idx).setImm(
        LatrCFAlu.getOperand(Mode0Idx).getImm());
    RootCFAlu.getOperand(KBank0Idx).setImm(
        LatrCFAlu.getOperand(KBank0Idx).getImm());
    RootCFAlu.getOperand(KBank0LineIdx)
        .setImm(LatrCFAlu.getOperand(KBank0LineIdx).getImm());
  }
  if (LatrCFAlu.getOperand(Mode1Idx).getImm()) {
    RootCFAlu.getOperand(Mode1Idx).setImm(
        LatrCFAlu.getOperand(Mode1Idx).getImm());
    RootCFAlu.getOperand(KBank1Idx).setImm(
        LatrCFAlu.getOperand(KBank1Idx).getImm());
    RootCFAlu.getOperand(KBank1LineIdx)
        .setImm(LatrCFAlu.getOperand(KBank1LineIdx).getImm());
  }
  RootCFAlu.getOperand(CntIdx).setImm(CumuledInsts);
  RootCFAlu.setDesc(TII->get(LatrCFAlu.getOpcode()));
  return true;
}

```
**EN:** This section evaluates backend conditions and selects behavior based on target state, IR structure, or machine-level properties.
**CN:** 本节根据目标状态、IR 结构或机器级属性进行条件判断，并选择相应的后端行为。

### Lines 174-207: Implements R600ClauseMergePass::runOnMachineFunction
```cpp
bool R600ClauseMergePass::runOnMachineFunction(MachineFunction &MF) {
  if (skipFunction(MF.getFunction()))
    return false;

  const R600Subtarget &ST = MF.getSubtarget<R600Subtarget>();
  TII = ST.getInstrInfo();

  for (MachineBasicBlock &MBB : MF) {
    MachineBasicBlock::iterator I = MBB.begin(),  E = MBB.end();
    MachineBasicBlock::iterator LatestCFAlu = E;
    while (I != E) {
      MachineInstr &MI = *I++;
      if ((!TII->canBeConsideredALU(MI) && !isCFAlu(MI)) ||
          TII->mustBeLastInClause(MI.getOpcode()))
        LatestCFAlu = E;
      if (!isCFAlu(MI))
        continue;
      cleanPotentialDisabledCFAlu(MI);

      if (LatestCFAlu != E && mergeIfPossible(*LatestCFAlu, MI)) {
        MI.eraseFromParent();
      } else {
        assert(MI.getOperand(8).getImm() && "CF ALU instruction disabled");
        LatestCFAlu = MI;
      }
    }
  }
  return false;
}

StringRef R600ClauseMergePass::getPassName() const {
  return "R600 Merge Clause Markers Pass";
}

```
**EN:** This section contains concrete logic for R600ClauseMergePass::runOnMachineFunction. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `R600ClauseMergePass::runOnMachineFunction`, `R600ClauseMergePass::getPassName`.
**CN:** 本节包含与 R600ClauseMergePass::runOnMachineFunction 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`R600ClauseMergePass::runOnMachineFunction`, `R600ClauseMergePass::getPassName`。

### Lines 208-210: Implements llvm::createR600ClauseMergePass
```cpp
llvm::FunctionPass *llvm::createR600ClauseMergePass() {
  return new R600ClauseMergePass();
}
```
**EN:** This section contains concrete logic for llvm::createR600ClauseMergePass. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `llvm::createR600ClauseMergePass`.
**CN:** 本节包含与 llvm::createR600ClauseMergePass 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`llvm::createR600ClauseMergePass`。

## Key Concepts / 关键概念
- **Language / 语言**: C++ source
- **Primary symbols / 主要符号**: `R600ClauseMergePass`, `R600ClauseMergePass::getCFAluSize`, `R600ClauseMergePass::isCFAluEnabled`, `R600ClauseMergePass::cleanPotentialDisabledCFAlu`, `R600ClauseMergePass::mergeIfPossible`, `R600ClauseMergePass::runOnMachineFunction`
- **Main themes / 核心主题**: instruction semantics / 指令语义; subtarget modeling / 子目标建模; LLVM pass integration / LLVM Pass 集成; machine-function state / MachineFunction 状态
- **Compilation role / 编译角色**: Part of the LLVM AMDGPU backend implementation / 属于 LLVM AMDGPU 后端实现的一部分

## Dependencies / 依赖关系
- `"MCTargetDesc/R600MCTargetDesc.h"`
- `"R600.h"`
- `"R600Subtarget.h"`
- `"llvm/CodeGen/MachineFunctionPass.h"`
