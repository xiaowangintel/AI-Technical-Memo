# WebAssemblyDebugValueManager.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyDebugValueManager.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements the manager for MachineInstr DebugValues.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyDebugValueManager.cpp`，主要负责 WebAssembly 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyDebugValueManager.cpp - WebAssembly DebugValue Manager -===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-11

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file implements the manager for MachineInstr DebugValues.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 12-18

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssemblyDebugValueManager.h"
#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "WebAssembly.h"
#include "WebAssemblyMachineFunctionInfo.h"
#include "llvm/CodeGen/MachineInstr.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 19-56

```cpp
#include "llvm/IR/DebugInfoMetadata.h"
#include "llvm/IR/Function.h"

using namespace llvm;

WebAssemblyDebugValueManager::WebAssemblyDebugValueManager(MachineInstr *Def)
    : Def(Def) {
  if (!Def->getMF()->getFunction().getSubprogram())
    return;

  // This code differs from MachineInstr::collectDebugValues in that it scans
  // all users in the BB, not just contiguous DBG_VALUEs, until another
  // definition to the same register is encountered.
  if (!Def->getOperand(0).isReg())
    return;
  CurrentReg = Def->getOperand(0).getReg();

  // Collect all the uses of this def.
  MachineRegisterInfo &MRI = Def->getMF()->getRegInfo();
  SmallVector<MachineInstr *, 2> Candidates;
  for (MachineInstr &MI : MRI.use_instructions(CurrentReg)) {
    if (MI.isDebugValue() && MI.getParent() == Def->getParent())
      Candidates.push_back(&MI);
  }
  if (Candidates.empty())
    return;

  // To preserve the order of DBG_VALUEs and correctly handle non-SSA cases,
  // we scan the BB as far as needed to find all candidates.
  for (MachineBasicBlock::iterator MI = std::next(Def->getIterator()),
                                   ME = Def->getParent()->end();
       MI != ME; ++MI) {
    // If another definition appears, stop
    if (MI->definesRegister(CurrentReg, /*TRI=*/nullptr))
      break;
    if (MI->isDebugValue() && MI->hasDebugOperandForReg(CurrentReg)) {
      DbgValues.push_back(&*MI);
      if (DbgValues.size() == Candidates.size())
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 57-94

```cpp
        break;
    }
  }
}

// Returns true if both A and B are the same CONST_I32/I64/F32/F64 instructions.
// Doesn't include CONST_V128.
static bool isSameScalarConst(const MachineInstr *A, const MachineInstr *B) {
  if (A->getOpcode() != B->getOpcode() ||
      !WebAssembly::isScalarConst(A->getOpcode()) ||
      !WebAssembly::isScalarConst(B->getOpcode()))
    return false;
  const MachineOperand &OpA = A->getOperand(1), &OpB = B->getOperand(1);
  if ((OpA.isImm() && OpB.isImm() && OpA.getImm() == OpB.getImm()) ||
      (OpA.isFPImm() && OpB.isFPImm() && OpA.getFPImm() == OpB.getFPImm()) ||
      (OpA.isGlobal() && OpB.isGlobal() && OpA.getGlobal() == OpB.getGlobal()))
    return true;
  return false;
}

SmallVector<MachineInstr *, 1>
WebAssemblyDebugValueManager::getSinkableDebugValues(
    MachineInstr *Insert) const {
  if (DbgValues.empty())
    return {};
  // DBG_VALUEs between Def and Insert
  SmallVector<MachineInstr *, 8> DbgValuesInBetween;

  if (Def->getParent() == Insert->getParent()) {
    // When Def and Insert are within the same BB, check if Insert comes after
    // Def, because we only support sinking.
    bool DefFirst = false;
    for (MachineBasicBlock::iterator MI = std::next(Def->getIterator()),
                                     ME = Def->getParent()->end();
         MI != ME; ++MI) {
      if (&*MI == Insert) {
        DefFirst = true;
        break;
```
- **EN**: Implements helper routine(s) `isSameScalarConst`, `getOpcode`, `isScalarConst` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `isSameScalarConst`, `getOpcode`, `isScalarConst`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 95-132

```cpp
      }
      if (MI->isDebugValue())
        DbgValuesInBetween.push_back(&*MI);
    }
    if (!DefFirst) // Not a sink
      return {};

  } else { // Def and Insert are in different BBs
    // If Def and Insert are in different BBs, we only handle a simple case in
    // which Insert's BB is a successor of Def's BB.
    if (!Def->getParent()->isSuccessor(Insert->getParent()))
      return {};

    // Gather DBG_VALUEs between 'Def~Def BB's end' and
    // 'Insert BB's begin~Insert'
    for (MachineBasicBlock::iterator MI = std::next(Def->getIterator()),
                                     ME = Def->getParent()->end();
         MI != ME; ++MI) {
      if (MI->isDebugValue())
        DbgValuesInBetween.push_back(&*MI);
    }
    for (MachineBasicBlock::iterator MI = Insert->getParent()->begin(),
                                     ME = Insert->getIterator();
         MI != ME; ++MI) {
      if (MI->isDebugValue())
        DbgValuesInBetween.push_back(&*MI);
    }
  }

  // Gather DebugVariables that are seen between Def and Insert, excluding our
  // own DBG_VALUEs in DbgValues.
  SmallDenseMap<DebugVariable, SmallVector<MachineInstr *, 2>>
      SeenDbgVarToDbgValues;
  for (auto *DV : DbgValuesInBetween) {
    if (!llvm::is_contained(DbgValues, DV)) {
      DebugVariable Var(DV->getDebugVariable(), DV->getDebugExpression(),
                        DV->getDebugLoc()->getInlinedAt());
      SeenDbgVarToDbgValues[Var].push_back(DV);
```
- **EN**: Implements helper routine(s) `isDebugValue`, `push_back`, `getParent` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `isDebugValue`, `push_back`, `getParent`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 133-170

```cpp
    }
  }

  // Gather sinkable DBG_VALUEs. We should not sink a DBG_VALUE if there is
  // another DBG_VALUE between Def and Insert referring to the same
  // DebugVariable. For example,
  //   %0 = someinst
  //   DBG_VALUE %0, !"a", !DIExpression() // Should not sink with %0
  //   %1 = anotherinst
  //   DBG_VALUE %1, !"a", !DIExpression()
  // Where if %0 were to sink, the DBG_VAUE should not sink with it, as that
  // would re-order assignments.
  SmallVector<MachineInstr *, 1> SinkableDbgValues;
  MachineRegisterInfo &MRI = Def->getParent()->getParent()->getRegInfo();
  for (auto *DV : DbgValues) {
    DebugVariable Var(DV->getDebugVariable(), DV->getDebugExpression(),
                      DV->getDebugLoc()->getInlinedAt());
    auto It = SeenDbgVarToDbgValues.find(Var);
    if (It == SeenDbgVarToDbgValues.end()) {
      SinkableDbgValues.push_back(DV);
      continue;
    }
    if (!WebAssembly::isScalarConst(Def->getOpcode()))
      continue;
    auto &OverlappingDbgValues = It->second;
    bool Sinkable = true;
    for (auto *OverlappingDV : OverlappingDbgValues) {
      MachineOperand &DbgOp = OverlappingDV->getDebugOperand(0);
      if (!DbgOp.isReg()) {
        Sinkable = false;
        break;
      }
      Register OtherReg = DbgOp.getReg();
      MachineInstr *OtherDef = MRI.getUniqueVRegDef(OtherReg);
      // We have an exception to allow encoutering other DBG_VALUEs with the
      // smae DebugVariables, only when they are referring to the same scalar
      // CONST instruction. For example,
      //   %0 = CONST_I32 1
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Gather sinkable DBG_VALUEs. We should not sink a DBG_VALUE if there is". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Gather sinkable DBG_VALUEs. We should not sink a DBG_VALUE if there is”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 171-208

```cpp
      //   DBG_VALUE %0, !"a", !DIExpression() // Can sink with %0
      //   %1 = CONST_I32 1
      //   DBG_VALUE %1, !"a", !DIExpression()
      // When %0 were to be sunk/cloneed, the DBG_VALUE can be sunk/cloned with
      // it because even though the second DBG_VALUE refers to the same
      // DebugVariable, its value in effect is the same CONST instruction.
      //
      // This is to allow a case that can happen with RegStackify's
      // "rematerializeCheapDef". For example, we have this program with two
      // BBs:
      // bb0:
      //   %0 = CONST_I32 1
      //   DBG_VALUE %0, !"a", ...
      //   ...
      //   INST0 ..., $0 ...
      //  bb1:
      //   INST1 ..., $0 ...
      //   INST2 ..., $0 ...
      //
      // We process bb0 first. Because %0 is used multiple times, %0 is cloned
      // before INST0:
      // bb0:
      //   %0 = CONST_I32 1
      //   DBG_VALUE %0, !"a", ...
      //   ...
      //   %1 = CONST_I32 1
      //   DBG_VALUE %1, !"a", ...
      //   INST0 ..., $1 ...
      //
      // And when we process bb1, we clone %0 and its DBG_VALUE again:
      // bb0:
      //   %0 = CONST_I32 1
      //   DBG_VALUE %0, !"a", ...
      //   ...
      //   %1 = CONST_I32 1
      //   DBG_VALUE %1, !"a", ...
      //   INST0 ..., $1 ...
      //  bb1:
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "DBG_VALUE %0, !"a", !DIExpression() // Can sink with %0". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“DBG_VALUE %0, !"a", !DIExpression() // Can sink with %0”。 该区间与栈帧布局或栈访问相关。

### Lines 209-246

```cpp
      //   %2 = CONST_I32 1
      //   DBG_VALUE %2, !"a", ... // !!!
      //   INST1 ..., $2 ...
      //   %3 = CONST_I32 1
      //   DBG_VALUE %3, !"a", ... // !!!
      //   INST2 ..., $3 ...
      //
      // But (without this exception) the cloned DBG_VALUEs marked with !!! are
      // not possible to be cloned, because there is a previously cloned
      // 'DBG_VALUE %1, !"a"' at the end of bb0 referring to the same
      // DebugVariable "a". But in this case they are OK to be cloned, because
      // the interfering DBG_VALUE is pointing to the same 'CONST_I32 1',
      // because it was cloned from the same instruction.
      if (!OtherDef || !isSameScalarConst(Def, OtherDef)) {
        Sinkable = false;
        break;
      }
    }
    if (Sinkable)
      SinkableDbgValues.push_back(DV);
  }
  return SinkableDbgValues;
}

// Returns true if the insertion point is the same as the current place.
// Following DBG_VALUEs for 'Def' are ignored.
bool WebAssemblyDebugValueManager::isInsertSamePlace(
    MachineInstr *Insert) const {
  if (Def->getParent() != Insert->getParent())
    return false;
  for (MachineBasicBlock::iterator MI = std::next(Def->getIterator()),
                                   ME = Insert;
       MI != ME; ++MI) {
    if (!llvm::is_contained(DbgValues, MI)) {
      return false;
    }
  }
  return true;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "%2 = CONST_I32 1". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“%2 = CONST_I32 1”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 247-284

```cpp
}

// Returns true if any instruction in MBB has the same debug location as DL.
// Also returns true if DL is an empty location.
static bool hasSameDebugLoc(const MachineBasicBlock *MBB, DebugLoc DL) {
  for (const auto &MI : *MBB)
    if (MI.getDebugLoc() == DL)
      return true;
  return false;
}

// Sink 'Def', and also sink its eligible DBG_VALUEs to the place before
// 'Insert'. Convert the original DBG_VALUEs into undefs.
//
// For DBG_VALUEs to sink properly, if 'Def' and 'Insert' are within the same
// BB, 'Insert' should be below 'Def'; if they are in different BBs, 'Insert'
// should be in one of 'Def's BBs successors. Def will be sunk regardless of the
// location.
//
// This DebugValueManager's new Def and DbgValues will be updated to the newly
// sinked Def + DBG_VALUEs.
void WebAssemblyDebugValueManager::sink(MachineInstr *Insert) {
  // In case Def is requested to be sunk to
  // the same place, we don't need to do anything. If we actually do the sink,
  // it will create unnecessary undef DBG_VALUEs. For example, if the original
  // code is:
  //   %0 = someinst           // Def
  //   DBG_VALUE %0, ...
  //   %1 = anotherinst        // Insert
  //
  // If we actually sink %0 and the following DBG_VALUE and setting the original
  // DBG_VALUE undef, the result will be:
  //   DBG_VALUE %noreg, ...   // Unnecessary!
  //   %0 = someinst           // Def
  //   DBG_VALUE %0, ...
  //   %1 = anotherinst        // Insert
  if (isInsertSamePlace(Insert))
    return;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Returns true if any instruction in MBB has the same debug location as DL.". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Returns true if any instruction in MBB has the same debug location as DL.”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 285-322

```cpp

  MachineBasicBlock *MBB = Insert->getParent();
  MachineFunction *MF = MBB->getParent();

  // Get the list of sinkable DBG_VALUEs. This should be done before sinking
  // Def, because we need to examine instructions between Def and Insert.
  SmallVector<MachineInstr *, 1> SinkableDbgValues =
      getSinkableDebugValues(Insert);

  // Sink Def first.
  //
  // When moving to a different BB, we preserve the debug loc only if the
  // destination BB contains the same location. See
  // https://llvm.org/docs/HowToUpdateDebugInfo.html#when-to-preserve-an-instruction-location.
  if (Def->getParent() != MBB && !hasSameDebugLoc(MBB, Def->getDebugLoc()))
      Def->setDebugLoc(DebugLoc());
  MBB->splice(Insert, Def->getParent(), Def);

  if (DbgValues.empty())
    return;

  // Clone sinkable DBG_VALUEs and insert them.
  SmallVector<MachineInstr *, 1> NewDbgValues;
  for (MachineInstr *DV : SinkableDbgValues) {
    MachineInstr *Clone = MF->CloneMachineInstr(DV);
    MBB->insert(Insert, Clone);
    NewDbgValues.push_back(Clone);
  }

  // When sinking a Def and its DBG_VALUEs, we shouldn't just remove the
  // original DBG_VALUE instructions; we should set them to undef not to create
  // an impossible combination of variable assignments in the original program.
  // For example, this is the original program in order:
  //   %0 = CONST_I32 0
  //   DBG_VALUE %0, !"a", !DIExpression()  // a = 0, b = ?
  //   %1 = CONST_I32 1
  //   DBG_VALUE %1, !"b", !DIExpression()  // a = 0, b = 1
  //   %2 = CONST_I32 2
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Get the list of sinkable DBG_VALUEs. This should be done before sinking". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Get the list of sinkable DBG_VALUEs. This should be done before sinking”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 323-360

```cpp
  //   DBG_VALUE %2, !"a", !DIExpression()  // a = 2, b = 1
  //   %3 = CONST_I32 3
  //   DBG_VALUE %3, !"b", !DIExpression()  // a = 2, b = 3
  //
  // If %2 were to sink below %3, if we just sink DBG_VALUE %1 with it, the
  // debug info will show the variable "b" is updated to 2, creating the
  // variable assignment combination of (a = 0, b = 3), which is not possible in
  // the original program:
  //   %0 = CONST_I32 0
  //   DBG_VALUE %0, !"a", !DIExpression()  // a = 0, b = ?
  //   %1 = CONST_I32 1
  //   DBG_VALUE %1, !"b", !DIExpression()  // a = 0, b = 1
  //   %3 = CONST_I32 3
  //   DBG_VALUE %3, !"b", !DIExpression()  // a = 0, b = 3 (Incorrect!)
  //   %2 = CONST_I32 2
  //   DBG_VALUE %2, !"a", !DIExpression()  // a = 2, b = 3
  //
  // To fix this,we leave an undef DBG_VALUE in its original place, so that the
  // result will be
  //   %0 = CONST_I32 0
  //   DBG_VALUE %0, !"a", !DIExpression()      // a = 0, b = ?
  //   %1 = CONST_I32 1
  //   DBG_VALUE %1, !"b", !DIExpression()      // a = 0, b = 1
  //   DBG_VALUE $noreg, !"a", !DIExpression()  // a = ?, b = 1
  //   %3 = CONST_I32 3
  //   DBG_VALUE %3, !"b", !DIExpression()      // a = ?, b = 3
  //   %2 = CONST_I32 2
  //   DBG_VALUE %2, !"a", !DIExpression()      // a = 2, b = 3
  // Now in the middle "a" will be shown as "optimized out", but it wouldn't
  // show the impossible combination of (a = 0, b = 3).
  for (MachineInstr *DV : DbgValues)
    DV->setDebugValueUndef();

  DbgValues.swap(NewDbgValues);
}

// Clone 'Def', and also clone its eligible DBG_VALUEs to the place before
// 'Insert'.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "DBG_VALUE %2, !"a", !DIExpression()  // a = 2, b = 1". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“DBG_VALUE %2, !"a", !DIExpression()  // a = 2, b = 1”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 361-398

```cpp
//
// For DBG_VALUEs to be cloned properly, if 'Def' and 'Insert' are within the
// same BB, 'Insert' should be below 'Def'; if they are in different BBs,
// 'Insert' should be in one of 'Def's BBs successors. Def will be cloned
// regardless of the location.
//
// If NewReg is not $noreg, the newly cloned DBG_VALUEs will have the new
// register as its operand.
void WebAssemblyDebugValueManager::cloneSink(MachineInstr *Insert,
                                             Register NewReg,
                                             bool CloneDef) const {
  MachineBasicBlock *MBB = Insert->getParent();
  MachineFunction *MF = MBB->getParent();

  SmallVector<MachineInstr *> SinkableDbgValues =
      getSinkableDebugValues(Insert);

  // Clone Def first.
  if (CloneDef) {
    MachineInstr *Clone = MF->CloneMachineInstr(Def);
    // When cloning to a different BB, we preserve the debug loc only if the
    // destination BB contains the same location. See
    // https://llvm.org/docs/HowToUpdateDebugInfo.html#when-to-preserve-an-instruction-location.
    if (Def->getParent() != MBB && !hasSameDebugLoc(MBB, Def->getDebugLoc()))
      Clone->setDebugLoc(DebugLoc());
    if (NewReg != CurrentReg && NewReg.isValid())
      Clone->getOperand(0).setReg(NewReg);
    MBB->insert(Insert, Clone);
  }

  if (DbgValues.empty())
    return;

  // Clone sinkable DBG_VALUEs and insert them.
  SmallVector<MachineInstr *, 1> NewDbgValues;
  for (MachineInstr *DV : SinkableDbgValues) {
    MachineInstr *Clone = MF->CloneMachineInstr(DV);
    MBB->insert(Insert, Clone);
```
- **EN**: Implements helper routine(s) `cloneSink`, `getParent`, `getSinkableDebugValues` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `cloneSink`, `getParent`, `getSinkableDebugValues`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 399-434

```cpp
    NewDbgValues.push_back(Clone);
  }

  if (NewReg != CurrentReg && NewReg.isValid())
    for (auto *DBI : NewDbgValues)
      for (auto &MO : DBI->getDebugOperandsForReg(CurrentReg))
        MO.setReg(NewReg);
}

// Update the register for Def and DBG_VALUEs.
void WebAssemblyDebugValueManager::updateReg(Register Reg) {
  if (Reg != CurrentReg && Reg.isValid()) {
    for (auto *DBI : DbgValues)
      for (auto &MO : DBI->getDebugOperandsForReg(CurrentReg))
        MO.setReg(Reg);
    CurrentReg = Reg;
    Def->getOperand(0).setReg(Reg);
  }
}

void WebAssemblyDebugValueManager::replaceWithLocal(unsigned LocalId) {
  for (auto *DBI : DbgValues) {
    auto IndexType = DBI->isIndirectDebugValue()
                         ? llvm::WebAssembly::TI_LOCAL_INDIRECT
                         : llvm::WebAssembly::TI_LOCAL;
    for (auto &MO : DBI->getDebugOperandsForReg(CurrentReg))
      MO.ChangeToTargetIndex(IndexType, LocalId);
  }
}

// Remove Def, and set its DBG_VALUEs to undef.
void WebAssemblyDebugValueManager::removeDef() {
  Def->removeFromParent();
  for (MachineInstr *DV : DbgValues)
    DV->setDebugValueUndef();
}
```
- **EN**: Implements helper routine(s) `push_back`, `isValid`, `getDebugOperandsForReg` for this portion of the WebAssembly backend backend implementation logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分后端实现逻辑所需的辅助例程 `push_back`, `isValid`, `getDebugOperandsForReg`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Backend implementation logic / 后端实现逻辑
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyDebugValueManager.h`
- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `WebAssembly.h`
- `WebAssemblyMachineFunctionInfo.h`
- `llvm/CodeGen/MachineInstr.h`
- `llvm/IR/DebugInfoMetadata.h`
- `llvm/IR/Function.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
