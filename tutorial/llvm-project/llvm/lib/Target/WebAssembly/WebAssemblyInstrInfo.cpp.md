# WebAssemblyInstrInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyInstrInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file contains the WebAssembly implementation of the TargetInstrInfo class.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyInstrInfo.cpp`，主要负责 WebAssembly 后端的指令信息与 lowering 辅助逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyInstrInfo.cpp - WebAssembly Instruction Information ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-12

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file contains the WebAssembly implementation of the
/// TargetInstrInfo class.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 13-19

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssemblyInstrInfo.h"
#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "WebAssembly.h"
#include "WebAssemblyMachineFunctionInfo.h"
#include "WebAssemblySubtarget.h"
```
- **EN**: Pulls in direct dependencies required by this instruction information and lowering helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该指令信息与 lowering 辅助逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 20-26

```cpp
#include "WebAssemblyUtilities.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineMemOperand.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
using namespace llvm;
```
- **EN**: Pulls in direct dependencies required by this instruction information and lowering helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该指令信息与 lowering 辅助逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 27-33

```cpp
#define DEBUG_TYPE "wasm-instr-info"

#define GET_INSTRINFO_CTOR_DTOR
#include "WebAssemblyGenInstrInfo.inc"

// defines WebAssembly::getNamedOperandIdx
#define GET_INSTRINFO_NAMED_OPS
```
- **EN**: Pulls in direct dependencies required by this instruction information and lowering helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该指令信息与 lowering 辅助逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 34-71

```cpp
#include "WebAssemblyGenInstrInfo.inc"

WebAssemblyInstrInfo::WebAssemblyInstrInfo(const WebAssemblySubtarget &STI)
    : WebAssemblyGenInstrInfo(STI, RI, WebAssembly::ADJCALLSTACKDOWN,
                              WebAssembly::ADJCALLSTACKUP,
                              WebAssembly::CATCHRET),
      RI(STI.getTargetTriple()) {}

bool WebAssemblyInstrInfo::isReMaterializableImpl(
    const MachineInstr &MI) const {
  switch (MI.getOpcode()) {
  case WebAssembly::CONST_I32:
  case WebAssembly::CONST_I64:
  case WebAssembly::CONST_F32:
  case WebAssembly::CONST_F64:
    // TargetInstrInfo::isReMaterializableImpl misses these
    // because of the ARGUMENTS implicit def, so we manualy override it here.
    return true;
  default:
    return TargetInstrInfo::isReMaterializableImpl(MI);
  }
}

void WebAssemblyInstrInfo::copyPhysReg(MachineBasicBlock &MBB,
                                       MachineBasicBlock::iterator I,
                                       const DebugLoc &DL, Register DestReg,
                                       Register SrcReg, bool KillSrc,
                                       bool RenamableDest,
                                       bool RenamableSrc) const {
  // This method is called by post-RA expansion, which expects only pregs to
  // exist. However we need to handle both here.
  auto &MRI = MBB.getParent()->getRegInfo();
  const TargetRegisterClass *RC =
      DestReg.isVirtual()
          ? MRI.getRegClass(DestReg)
          : MRI.getTargetRegisterInfo()->getMinimalPhysRegClass(DestReg);

  unsigned CopyOpcode = WebAssembly::getCopyOpcodeForRegClass(RC);
```
- **EN**: Pulls in direct dependencies required by this instruction information and lowering helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该指令信息与 lowering 辅助逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 72-109

```cpp

  BuildMI(MBB, I, DL, get(CopyOpcode), DestReg)
      .addReg(SrcReg, getKillRegState(KillSrc));
}

MachineInstr *WebAssemblyInstrInfo::commuteInstructionImpl(
    MachineInstr &MI, bool NewMI, unsigned OpIdx1, unsigned OpIdx2) const {
  // If the operands are stackified, we can't reorder them.
  WebAssemblyFunctionInfo &MFI =
      *MI.getParent()->getParent()->getInfo<WebAssemblyFunctionInfo>();
  if (MFI.isVRegStackified(MI.getOperand(OpIdx1).getReg()) ||
      MFI.isVRegStackified(MI.getOperand(OpIdx2).getReg()))
    return nullptr;

  // Otherwise use the default implementation.
  return TargetInstrInfo::commuteInstructionImpl(MI, NewMI, OpIdx1, OpIdx2);
}

// Branch analysis.
bool WebAssemblyInstrInfo::analyzeBranch(MachineBasicBlock &MBB,
                                         MachineBasicBlock *&TBB,
                                         MachineBasicBlock *&FBB,
                                         SmallVectorImpl<MachineOperand> &Cond,
                                         bool /*AllowModify*/) const {
  const auto &MFI = *MBB.getParent()->getInfo<WebAssemblyFunctionInfo>();
  // WebAssembly has control flow that doesn't have explicit branches or direct
  // fallthrough (e.g. try/catch), which can't be modeled by analyzeBranch. It
  // is created after CFGStackify.
  if (MFI.isCFGStackified())
    return true;

  bool HaveCond = false;
  for (MachineInstr &MI : MBB.terminators()) {
    switch (MI.getOpcode()) {
    default:
      // Unhandled instruction; bail out.
      return true;
    case WebAssembly::BR_IF:
```
- **EN**: Implements helper routine(s) `BuildMI`, `get`, `addReg` for this portion of the WebAssembly backend instruction information and lowering helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程 `BuildMI`, `get`, `addReg`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 110-147

```cpp
      if (HaveCond)
        return true;
      Cond.push_back(MachineOperand::CreateImm(true));
      Cond.push_back(MI.getOperand(1));
      TBB = MI.getOperand(0).getMBB();
      HaveCond = true;
      break;
    case WebAssembly::BR_UNLESS:
      if (HaveCond)
        return true;
      Cond.push_back(MachineOperand::CreateImm(false));
      Cond.push_back(MI.getOperand(1));
      TBB = MI.getOperand(0).getMBB();
      HaveCond = true;
      break;
    case WebAssembly::BR:
      if (!HaveCond)
        TBB = MI.getOperand(0).getMBB();
      else
        FBB = MI.getOperand(0).getMBB();
      break;
    }
    if (MI.isBarrier())
      break;
  }

  return false;
}

unsigned WebAssemblyInstrInfo::removeBranch(MachineBasicBlock &MBB,
                                            int *BytesRemoved) const {
  assert(!BytesRemoved && "code size not handled");

  MachineBasicBlock::instr_iterator I = MBB.instr_end();
  unsigned Count = 0;

  while (I != MBB.instr_begin()) {
    --I;
```
- **EN**: Implements helper routine(s) `push_back`, `CreateImm`, `getOperand` for this portion of the WebAssembly backend instruction information and lowering helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程 `push_back`, `CreateImm`, `getOperand`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 148-185

```cpp
    if (I->isDebugInstr())
      continue;
    if (!I->isTerminator())
      break;
    // Remove the branch.
    I->eraseFromParent();
    I = MBB.instr_end();
    ++Count;
  }

  return Count;
}

unsigned WebAssemblyInstrInfo::insertBranch(
    MachineBasicBlock &MBB, MachineBasicBlock *TBB, MachineBasicBlock *FBB,
    ArrayRef<MachineOperand> Cond, const DebugLoc &DL, int *BytesAdded) const {
  assert(!BytesAdded && "code size not handled");

  if (Cond.empty()) {
    if (!TBB)
      return 0;

    BuildMI(&MBB, DL, get(WebAssembly::BR)).addMBB(TBB);
    return 1;
  }

  assert(Cond.size() == 2 && "Expected a flag and a successor block");

  if (Cond[0].getImm())
    BuildMI(&MBB, DL, get(WebAssembly::BR_IF)).addMBB(TBB).add(Cond[1]);
  else
    BuildMI(&MBB, DL, get(WebAssembly::BR_UNLESS)).addMBB(TBB).add(Cond[1]);
  if (!FBB)
    return 1;

  BuildMI(&MBB, DL, get(WebAssembly::BR)).addMBB(FBB);
  return 2;
}
```
- **EN**: Implements helper routine(s) `isDebugInstr`, `isTerminator`, `eraseFromParent` for this portion of the WebAssembly backend instruction information and lowering helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程 `isDebugInstr`, `isTerminator`, `eraseFromParent`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 186-223

```cpp

bool WebAssemblyInstrInfo::reverseBranchCondition(
    SmallVectorImpl<MachineOperand> &Cond) const {
  assert(Cond.size() == 2 && "Expected a flag and a condition expression");
  Cond.front() = MachineOperand::CreateImm(!Cond.front().getImm());
  return false;
}

ArrayRef<std::pair<int, const char *>>
WebAssemblyInstrInfo::getSerializableTargetIndices() const {
  static const std::pair<int, const char *> TargetIndices[] = {
      {WebAssembly::TI_LOCAL, "wasm-local"},
      {WebAssembly::TI_GLOBAL_FIXED, "wasm-global-fixed"},
      {WebAssembly::TI_OPERAND_STACK, "wasm-operand-stack"},
      {WebAssembly::TI_GLOBAL_RELOC, "wasm-global-reloc"},
      {WebAssembly::TI_LOCAL_INDIRECT, "wasm-local-indirect"}};
  return ArrayRef(TargetIndices);
}

const MachineOperand &
WebAssemblyInstrInfo::getCalleeOperand(const MachineInstr &MI) const {
  return WebAssembly::getCalleeOp(MI);
}

// This returns true when the instruction defines a value of a TargetIndex
// operand that can be tracked by offsets. For Wasm, this returns true for only
// local.set/local.tees. This is currently used by LiveDebugValues analysis.
//
// These are not included:
// - In theory we need to add global.set here too, but we don't have global
//   indices at this point because they are relocatable and we address them by
//   names until linking, so we don't have 'offsets' (which are used to store
//   local/global indices) to deal with in LiveDebugValues. And we don't
//   associate debug info in values in globals anyway.
// - All other value-producing instructions, i.e. instructions with defs, can
//   define values in the Wasm stack, which is represented by TI_OPERAND_STACK
//   TargetIndex. But they don't have offset info within the instruction itself,
//   and debug info analysis for them is handled separately in
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This returns true when the instruction defines a value of a TargetIndex". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This returns true when the instruction defines a value of a TargetIndex”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 224-235

```cpp
//   WebAssemblyDebugFixup pass, so we don't worry about them here.
bool WebAssemblyInstrInfo::isExplicitTargetIndexDef(const MachineInstr &MI,
                                                    int &Index,
                                                    int64_t &Offset) const {
  unsigned Opc = MI.getOpcode();
  if (WebAssembly::isLocalSet(Opc) || WebAssembly::isLocalTee(Opc)) {
    Index = WebAssembly::TI_LOCAL;
    Offset = MI.explicit_uses().begin()->getImm();
    return true;
  }
  return false;
}
```
- **EN**: Implements helper routine(s) `isExplicitTargetIndexDef`, `getOpcode`, `isLocalSet` for this portion of the WebAssembly backend instruction information and lowering helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程 `isExplicitTargetIndexDef`, `getOpcode`, `isLocalSet`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Instruction information and lowering helpers / 指令信息与 lowering 辅助逻辑
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Object format integration / 目标文件格式集成
- Vector or SIMD behavior / 向量或 SIMD 行为

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyInstrInfo.h`
- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `WebAssembly.h`
- `WebAssemblyMachineFunctionInfo.h`
- `WebAssemblySubtarget.h`
- `WebAssemblyUtilities.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineMemOperand.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `WebAssemblyGenInstrInfo.inc`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
