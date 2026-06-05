# MipsOptimizePICCall.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsOptimizePICCall.cpp`
- Repository: `llvm-project`
- Purpose (EN): Implements `MipsOptimizePICCall` for the Mips backend, focusing on target-specific code-generation behavior.
- 用途 (CN): 实现 Mips 后端中的 `MipsOptimizePICCall`，重点处理目标相关代码生成行为。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===- MipsOptimizePICCall.cpp - Optimize PIC Calls -----------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass eliminates unnecessary instructions that set up $gp and replace
// instructions that load target function addresses with copy instructions.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 14-28
```cpp
#include "MCTargetDesc/MipsBaseInfo.h"
#include "Mips.h"
#include "MipsRegisterInfo.h"
#include "MipsSubtarget.h"
#include "llvm/ADT/PointerUnion.h"
#include "llvm/ADT/ScopedHashTable.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/CodeGen/MachineBasicBlock.h"
#include "llvm/CodeGen/MachineDominators.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineFunctionPass.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 29-39
```cpp
#include "llvm/CodeGen/TargetInstrInfo.h"
#include "llvm/CodeGen/TargetOpcodes.h"
#include "llvm/CodeGen/TargetRegisterInfo.h"
#include "llvm/CodeGen/TargetSubtargetInfo.h"
#include "llvm/CodeGenTypes/MachineValueType.h"
#include "llvm/Support/Allocator.h"
#include "llvm/Support/CommandLine.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/RecyclingAllocator.h"
#include <cassert>
#include <utility>
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 41-41
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 43-43
```cpp
#define DEBUG_TYPE "optimize-mips-pic-call"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 45-48
```cpp
static cl::opt<bool> LoadTargetFromGOT("mips-load-target-from-got",
                                       cl::init(true),
                                       cl::desc("Load target address from GOT"),
                                       cl::Hidden);
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 50-52
```cpp
static cl::opt<bool> EraseGPOpnd("mips-erase-gp-opnd",
                                 cl::init(true), cl::desc("Erase GP Operand"),
                                 cl::Hidden);
```
- EN: Declares `cl::init`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `cl::init`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 54-54
```cpp
namespace {
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 56-61
```cpp
using ValueType = PointerUnion<const Value *, const PseudoSourceValue *>;
using CntRegP = std::pair<unsigned, unsigned>;
using AllocatorTy = RecyclingAllocator<BumpPtrAllocator,
                                       ScopedHashTableVal<ValueType, CntRegP>>;
using ScopedHTType = ScopedHashTable<ValueType, CntRegP,
                                     DenseMapInfo<ValueType>, AllocatorTy>;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 63-65
```cpp
class MBBInfo {
public:
  MBBInfo(MachineDomTreeNode *N);
```
- EN: Declares `MBBInfo`, packaging target-specific state and APIs around `MipsOptimizePICCall`.
- CN: 这里声明 `MBBInfo`，把与 `MipsOptimizePICCall` 相关的目标特定状态和 API 组织在一起。

### Lines 67-70
```cpp
  const MachineDomTreeNode *getNode() const;
  bool isVisited() const;
  void preVisit(ScopedHTType &ScopedHT);
  void postVisit();
```
- EN: Declares `getNode`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getNode`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 72-75
```cpp
private:
  MachineDomTreeNode *Node;
  ScopedHTType::ScopeTy *HTScope;
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 77-79
```cpp
class OptimizePICCall : public MachineFunctionPass {
public:
  OptimizePICCall() : MachineFunctionPass(ID) {}
```
- EN: Declares `OptimizePICCall`, packaging target-specific state and APIs around `MipsOptimizePICCall`.
- CN: 这里声明 `OptimizePICCall`，把与 `MipsOptimizePICCall` 相关的目标特定状态和 API 组织在一起。

### Lines 81-81
```cpp
  StringRef getPassName() const override { return "Mips OptimizePICCall"; }
```
- EN: Implements `getPassName`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getPassName`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 83-83
```cpp
  bool runOnMachineFunction(MachineFunction &F) override;
```
- EN: Declares `runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 85-88
```cpp
  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.addRequired<MachineDominatorTreeWrapperPass>();
    MachineFunctionPass::getAnalysisUsage(AU);
  }
```
- EN: Implements `getAnalysisUsage`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getAnalysisUsage`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 90-92
```cpp
private:
  /// Visit MBB.
  bool visitNode(MBBInfo &MBBI);
```
- EN: Declares `visitNode`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `visitNode`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 94-100
```cpp
  /// Test if MI jumps to a function via a register.
  ///
  /// Also, return the virtual register containing the target function's address
  /// and the underlying object in Reg and Val respectively, if the function's
  /// address can be resolved lazily.
  bool isCallViaRegister(MachineInstr &MI, unsigned &Reg,
                         ValueType &Val) const;
```
- EN: Declares `isCallViaRegister`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `isCallViaRegister`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 102-104
```cpp
  /// Return the number of instructions that dominate the current
  /// instruction and load the function address from object Entry.
  unsigned getCount(ValueType Entry);
```
- EN: Declares `getCount`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getCount`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 106-108
```cpp
  /// Return the destination virtual register of the last instruction
  /// that loads from object Entry.
  unsigned getReg(ValueType Entry);
```
- EN: Declares `getReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `getReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 110-111
```cpp
  /// Update ScopedHT.
  void incCntAndSetReg(ValueType Entry, unsigned Reg);
```
- EN: Declares `incCntAndSetReg`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `incCntAndSetReg`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 113-113
```cpp
  ScopedHTType ScopedHT;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 115-116
```cpp
  static char ID;
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 118-118
```cpp
} // end of anonymous namespace
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 120-120
```cpp
char OptimizePICCall::ID = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 122-125
```cpp
/// Return the first MachineOperand of MI if it is a used virtual register.
static MachineOperand *getCallTargetRegOpnd(MachineInstr &MI) {
  if (MI.getNumOperands() == 0)
    return nullptr;
```
- EN: Implements `getCallTargetRegOpnd`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getCallTargetRegOpnd`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 127-127
```cpp
  MachineOperand &MO = MI.getOperand(0);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 129-130
```cpp
  if (!MO.isReg() || !MO.isUse() || !MO.getReg().isVirtual())
    return nullptr;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 132-133
```cpp
  return &MO;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 135-141
```cpp
/// Return type of register Reg.
static MVT::SimpleValueType getRegTy(unsigned Reg, MachineFunction &MF) {
  const TargetRegisterInfo &TRI = *MF.getSubtarget().getRegisterInfo();
  const TargetRegisterClass *RC = MF.getRegInfo().getRegClass(Reg);
  assert(TRI.legalclasstypes_end(*RC) - TRI.legalclasstypes_begin(*RC) == 1);
  return *TRI.legalclasstypes_begin(*RC);
}
```
- EN: Implements `getRegTy`, a query/helper routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getRegTy`，它是一个围绕机器函数状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 143-157
```cpp
/// Do the following transformation:
///
/// jalr $vreg
/// =>
/// copy $t9, $vreg
/// jalr $t9
static void setCallTargetReg(MachineBasicBlock *MBB,
                             MachineBasicBlock::iterator I) {
  MachineFunction &MF = *MBB->getParent();
  const TargetInstrInfo &TII = *MF.getSubtarget().getInstrInfo();
  Register SrcReg = I->getOperand(0).getReg();
  unsigned DstReg = getRegTy(SrcReg, MF) == MVT::i32 ? Mips::T9 : Mips::T9_64;
  BuildMI(*MBB, I, I->getDebugLoc(), TII.get(TargetOpcode::COPY), DstReg)
      .addReg(SrcReg);
  I->getOperand(0).setReg(DstReg);
```
- EN: Implements `setCallTargetReg`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setCallTargetReg`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 158-158
```cpp
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 160-163
```cpp
/// Search MI's operands for register GP and erase it.
static void eraseGPOpnd(MachineInstr &MI) {
  if (!EraseGPOpnd)
    return;
```
- EN: Implements `eraseGPOpnd`, a target-specific routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `eraseGPOpnd`，它是一个围绕机器指令展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 165-167
```cpp
  MachineFunction &MF = *MI.getParent()->getParent();
  MVT::SimpleValueType Ty = getRegTy(MI.getOperand(0).getReg(), MF);
  unsigned Reg = Ty == MVT::i32 ? Mips::GP : Mips::GP_64;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 169-175
```cpp
  for (unsigned I = 0; I < MI.getNumOperands(); ++I) {
    MachineOperand &MO = MI.getOperand(I);
    if (MO.isReg() && MO.getReg() == Reg) {
      MI.removeOperand(I);
      return;
    }
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 177-178
```cpp
  llvm_unreachable(nullptr);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 180-180
```cpp
MBBInfo::MBBInfo(MachineDomTreeNode *N) : Node(N), HTScope(nullptr) {}
```
- EN: Implements `MBBInfo::MBBInfo`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MBBInfo::MBBInfo`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 182-182
```cpp
const MachineDomTreeNode *MBBInfo::getNode() const { return Node; }
```
- EN: Implements `MBBInfo::getNode`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MBBInfo::getNode`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 184-184
```cpp
bool MBBInfo::isVisited() const { return HTScope; }
```
- EN: Implements `MBBInfo::isVisited`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MBBInfo::isVisited`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 186-188
```cpp
void MBBInfo::preVisit(ScopedHTType &ScopedHT) {
  HTScope = new ScopedHTType::ScopeTy(ScopedHT);
}
```
- EN: Implements `MBBInfo::preVisit`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MBBInfo::preVisit`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 190-192
```cpp
void MBBInfo::postVisit() {
  delete HTScope;
}
```
- EN: Implements `MBBInfo::postVisit`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `MBBInfo::postVisit`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 194-197
```cpp
// OptimizePICCall methods.
bool OptimizePICCall::runOnMachineFunction(MachineFunction &F) {
  if (F.getSubtarget<MipsSubtarget>().inMips16Mode())
    return false;
```
- EN: Implements `OptimizePICCall::runOnMachineFunction`, a pass-entry routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `OptimizePICCall::runOnMachineFunction`，它是一个围绕机器函数状态展开的Pass 入口例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 199-202
```cpp
  // Do a pre-order traversal of the dominator tree.
  MachineDominatorTree *MDT =
      &getAnalysis<MachineDominatorTreeWrapperPass>().getDomTree();
  bool Changed = false;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 204-204
```cpp
  SmallVector<MBBInfo, 8> WorkList(1, MBBInfo(MDT->getRootNode()));
```
- EN: Declares `WorkList`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `WorkList`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 206-207
```cpp
  while (!WorkList.empty()) {
    MBBInfo &MBBI = WorkList.back();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 209-215
```cpp
    // If this MBB has already been visited, destroy the scope for the MBB and
    // pop it from the work list.
    if (MBBI.isVisited()) {
      MBBI.postVisit();
      WorkList.pop_back();
      continue;
    }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 217-222
```cpp
    // Visit the MBB and add its children to the work list.
    MBBI.preVisit(ScopedHT);
    Changed |= visitNode(MBBI);
    const MachineDomTreeNode *Node = MBBI.getNode();
    WorkList.append(Node->begin(), Node->end());
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 224-225
```cpp
  return Changed;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 227-229
```cpp
bool OptimizePICCall::visitNode(MBBInfo &MBBI) {
  bool Changed = false;
  MachineBasicBlock *MBB = MBBI.getNode()->getBlock();
```
- EN: Implements `OptimizePICCall::visitNode`, a target-specific routine centered on machine basic blocks. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `OptimizePICCall::visitNode`，它是一个围绕机器基本块展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 231-234
```cpp
  for (MachineBasicBlock::iterator I = MBB->begin(), E = MBB->end(); I != E;
       ++I) {
    unsigned Reg;
    ValueType Entry;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 236-238
```cpp
    // Skip instructions that are not call instructions via registers.
    if (!isCallViaRegister(*I, Reg, Entry))
      continue;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 240-241
```cpp
    Changed = true;
    unsigned N = getCount(Entry);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 243-248
```cpp
    if (N != 0) {
      // If a function has been called more than twice, we do not have to emit a
      // load instruction to get the function address from the GOT, but can
      // instead reuse the address that has been loaded before.
      if (N >= 2 && !LoadTargetFromGOT)
        getCallTargetRegOpnd(*I)->setReg(getReg(Entry));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 250-254
```cpp
      // Erase the $gp operand if this isn't the first time a function has
      // been called. $gp needs to be set up only if the function call can go
      // through a lazy binding stub.
      eraseGPOpnd(*I);
    }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 256-257
```cpp
    if (Entry)
      incCntAndSetReg(Entry, Reg);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 259-260
```cpp
    setCallTargetReg(MBB, I);
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 262-263
```cpp
  return Changed;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 265-268
```cpp
bool OptimizePICCall::isCallViaRegister(MachineInstr &MI, unsigned &Reg,
                                        ValueType &Val) const {
  if (!MI.isCall())
    return false;
```
- EN: Implements `OptimizePICCall::isCallViaRegister`, a query/helper routine centered on machine instructions. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `OptimizePICCall::isCallViaRegister`，它是一个围绕机器指令展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 270-270
```cpp
  MachineOperand *MO = getCallTargetRegOpnd(MI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 272-274
```cpp
  // Return if MI is not a function call via a register.
  if (!MO)
    return false;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 276-280
```cpp
  // Get the instruction that loads the function address from the GOT.
  Reg = MO->getReg();
  Val = nullptr;
  MachineRegisterInfo &MRI = MI.getParent()->getParent()->getRegInfo();
  MachineInstr *DefMI = MRI.getVRegDef(Reg);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 282-282
```cpp
  assert(DefMI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 284-287
```cpp
  // See if DefMI is an instruction that loads from a GOT entry that holds the
  // address of a lazy binding stub.
  if (!DefMI->mayLoad() || DefMI->getNumOperands() < 3)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 289-289
```cpp
  unsigned Flags = DefMI->getOperand(2).getTargetFlags();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 291-292
```cpp
  if (Flags != MipsII::MO_GOT_CALL && Flags != MipsII::MO_CALL_LO16)
    return true;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 294-300
```cpp
  // Return the underlying object for the GOT entry in Val.
  assert(DefMI->hasOneMemOperand());
  Val = (*DefMI->memoperands_begin())->getValue();
  if (!Val)
    Val = (*DefMI->memoperands_begin())->getPseudoValue();
  return true;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 302-304
```cpp
unsigned OptimizePICCall::getCount(ValueType Entry) {
  return ScopedHT.lookup(Entry).first;
}
```
- EN: Implements `OptimizePICCall::getCount`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `OptimizePICCall::getCount`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 306-310
```cpp
unsigned OptimizePICCall::getReg(ValueType Entry) {
  unsigned Reg = ScopedHT.lookup(Entry).second;
  assert(Reg);
  return Reg;
}
```
- EN: Implements `OptimizePICCall::getReg`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `OptimizePICCall::getReg`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 312-315
```cpp
void OptimizePICCall::incCntAndSetReg(ValueType Entry, unsigned Reg) {
  CntRegP P = ScopedHT.lookup(Entry);
  ScopedHT.insert(Entry, std::make_pair(P.first + 1, Reg));
}
```
- EN: Implements `OptimizePICCall::incCntAndSetReg`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `OptimizePICCall::incCntAndSetReg`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 317-320
```cpp
/// Return an OptimizeCall object.
FunctionPass *llvm::createMipsOptimizePICCallPass() {
  return new OptimizePICCall();
}
```
- EN: Implements `llvm::createMipsOptimizePICCallPass`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `llvm::createMipsOptimizePICCallPass`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

## Key Concepts / 关键概念

- EN: Primary role: target-specific code-generation behavior.
  - CN: 核心职责：目标相关代码生成行为。
- EN: MachineInstr-level manipulation is central to this file.
  - CN: 该文件以 MachineInstr 层面的操作为核心。
- EN: MachineBasicBlock flow and branch structure matter to the implementation.
  - CN: 实现高度依赖 MachineBasicBlock 流程与分支结构。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。
- EN: The component plugs into LLVM machine-function passes or codegen pipelines.
  - CN: 该组件会接入 LLVM 的机器函数 Pass 或代码生成流水线。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MCTargetDesc/MipsBaseInfo.h`, `Mips.h`, `MipsRegisterInfo.h`, `MipsSubtarget.h`.
  - CN: 后端本地头文件：`MCTargetDesc/MipsBaseInfo.h`, `Mips.h`, `MipsRegisterInfo.h`, `MipsSubtarget.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/PointerUnion.h`, `llvm/ADT/ScopedHashTable.h`, `llvm/ADT/SmallVector.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineDominators.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h` ... (+12 more).
  - CN: LLVM 基础设施头文件：`llvm/ADT/PointerUnion.h`, `llvm/ADT/ScopedHashTable.h`, `llvm/ADT/SmallVector.h`, `llvm/CodeGen/MachineBasicBlock.h`, `llvm/CodeGen/MachineDominators.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineFunctionPass.h`, `llvm/CodeGen/MachineInstr.h` ... (+12 more)。
- EN: Standard/system headers: `cassert`, `utility`.
  - CN: 标准库/系统头文件：`cassert`, `utility`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
