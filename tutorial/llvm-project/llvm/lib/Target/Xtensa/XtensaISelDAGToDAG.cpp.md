# XtensaISelDAGToDAG.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaISelDAGToDAG.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements DAG-to-DAG instruction selection that turns legalized SelectionDAG nodes into target machine instructions.
  - **CN**: 实现 DAG-to-DAG 指令选择，把合法化后的 SelectionDAG 节点转换为目标机器指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===- XtensaISelDAGToDAG.cpp - A dag to dag inst selector for Xtensa -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-21
```cpp
//
// This file defines an instruction selector for the Xtensa target.
//
//===----------------------------------------------------------------------===//

#include "MCTargetDesc/XtensaMCTargetDesc.h"
#include "Xtensa.h"
#include "XtensaSelectionDAGInfo.h"
#include "XtensaTargetMachine.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/SelectionDAGISel.h"
#include "llvm/IR/DiagnosticInfo.h"
#include "llvm/Support/Debug.h"
```
- **EN**: Pulls in the headers needed for this implementation, including `MCTargetDesc/XtensaMCTargetDesc.h`, `Xtensa.h`, `XtensaSelectionDAGInfo.h`, `XtensaTargetMachine.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MCTargetDesc/XtensaMCTargetDesc.h`, `Xtensa.h`, `XtensaSelectionDAGInfo.h`, `XtensaTargetMachine.h`。

### Lines 22-29
```cpp
#include "llvm/Support/raw_ostream.h"

using namespace llvm;

#define DEBUG_TYPE "xtensa-isel"

namespace {

```
- **EN**: Pulls in the headers needed for this implementation, including `llvm/Support/raw_ostream.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `llvm/Support/raw_ostream.h`。

### Lines 30-36
```cpp
class XtensaDAGToDAGISel : public SelectionDAGISel {
  const XtensaSubtarget *Subtarget = nullptr;

public:
  explicit XtensaDAGToDAGISel(XtensaTargetMachine &TM, CodeGenOptLevel OptLevel)
      : SelectionDAGISel(TM, OptLevel) {}

```
- **EN**: Introduces declarations for `XtensaDAGToDAGISel`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `XtensaDAGToDAGISel` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 37-43
```cpp
  bool runOnMachineFunction(MachineFunction &MF) override {
    Subtarget = &MF.getSubtarget<XtensaSubtarget>();
    return SelectionDAGISel::runOnMachineFunction(MF);
  }

  void Select(SDNode *Node) override;

```
- **EN**: Implements logic around `runOnMachineFunction`, `getSubtarget<XtensaSubtarget>`, `Select`; this block returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `runOnMachineFunction`, `getSubtarget<XtensaSubtarget>`, `Select` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 44-53
```cpp
  bool SelectInlineAsmMemoryOperand(const SDValue &Op,
                                    InlineAsm::ConstraintCode ConstraintID,
                                    std::vector<SDValue> &OutOps) override;

  // For load/store instructions generate (base+offset) pair from
  // memory address. The offset must be a multiple of scale argument.
  bool selectMemRegAddr(SDValue Addr, SDValue &Base, SDValue &Offset,
                        int Scale) {
    EVT ValTy = Addr.getValueType();

```
- **EN**: Implements logic around `SelectInlineAsmMemoryOperand`, `selectMemRegAddr`, `getValueType`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `SelectInlineAsmMemoryOperand`, `selectMemRegAddr`, `getValueType` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 54-61
```cpp
    // if Address is FI, get the TargetFrameIndex.
    if (FrameIndexSDNode *FIN = dyn_cast<FrameIndexSDNode>(Addr)) {
      Base = CurDAG->getTargetFrameIndex(FIN->getIndex(), ValTy);
      Offset = CurDAG->getTargetConstant(0, SDLoc(Addr), ValTy);

      return true;
    }

```
- **EN**: Implements logic around `getTargetFrameIndex`, `getTargetConstant`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getTargetFrameIndex`, `getTargetConstant` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 62-71
```cpp
    if (TM.isPositionIndependent()) {
      CurDAG->getContext()->diagnose(DiagnosticInfoUnsupported(
          CurDAG->getMachineFunction().getFunction(),
          "PIC relocations are not supported", Addr.getDebugLoc()));
    }

    if ((Addr.getOpcode() == ISD::TargetExternalSymbol ||
         Addr.getOpcode() == ISD::TargetGlobalAddress))
      return false;

```
- **EN**: Implements logic around `getContext`, `getMachineFunction`, `getDebugLoc`, `getOpcode`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getContext`, `getMachineFunction`, `getDebugLoc`, `getOpcode` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 72-79
```cpp
    // Addresses of the form FI+const
    bool Valid = false;
    if (CurDAG->isBaseWithConstantOffset(Addr)) {
      ConstantSDNode *CN = dyn_cast<ConstantSDNode>(Addr.getOperand(1));
      int64_t OffsetVal = CN->getSExtValue();

      Valid = Xtensa::isValidAddrOffset(Scale, OffsetVal);

```
- **EN**: Implements logic around `dyn_cast<ConstantSDNode>`, `getSExtValue`, `isValidAddrOffset`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `dyn_cast<ConstantSDNode>`, `getSExtValue`, `isValidAddrOffset` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 80-87
```cpp
      if (Valid) {
        // If the first operand is a FI, get the TargetFI Node.
        if (FrameIndexSDNode *FIN =
                dyn_cast<FrameIndexSDNode>(Addr.getOperand(0)))
          Base = CurDAG->getTargetFrameIndex(FIN->getIndex(), ValTy);
        else
          Base = Addr.getOperand(0);

```
- **EN**: Implements logic around `dyn_cast<FrameIndexSDNode>`, `getTargetFrameIndex`, `getOperand`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `dyn_cast<FrameIndexSDNode>`, `getTargetFrameIndex`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 88-99
```cpp
        Offset =
            CurDAG->getTargetConstant(CN->getZExtValue(), SDLoc(Addr), ValTy);
        return true;
      }
    }

    // Last case
    Base = Addr;
    Offset = CurDAG->getTargetConstant(0, SDLoc(Addr), Addr.getValueType());
    return true;
  }

```
- **EN**: Implements logic around `getTargetConstant`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getTargetConstant` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 100-107
```cpp
  bool selectMemRegAddrISH1(SDValue Addr, SDValue &Base, SDValue &Offset) {
    return selectMemRegAddr(Addr, Base, Offset, 1);
  }

  bool selectMemRegAddrISH2(SDValue Addr, SDValue &Base, SDValue &Offset) {
    return selectMemRegAddr(Addr, Base, Offset, 2);
  }

```
- **EN**: Implements logic around `selectMemRegAddrISH1`, `selectMemRegAddr`, `selectMemRegAddrISH2`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `selectMemRegAddrISH1`, `selectMemRegAddr`, `selectMemRegAddrISH2` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 108-115
```cpp
  bool selectMemRegAddrISH4(SDValue Addr, SDValue &Base, SDValue &Offset) {
    return selectMemRegAddr(Addr, Base, Offset, 4);
  }

// Include the pieces autogenerated from the target description.
#include "XtensaGenDAGISel.inc"
}; // namespace

```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaGenDAGISel.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaGenDAGISel.inc`。

### Lines 116-123
```cpp
class XtensaDAGToDAGISelLegacy : public SelectionDAGISelLegacy {
public:
  static char ID;

  XtensaDAGToDAGISelLegacy(XtensaTargetMachine &TM, CodeGenOptLevel OptLevel)
      : SelectionDAGISelLegacy(
            ID, std::make_unique<XtensaDAGToDAGISel>(TM, OptLevel)) {}

```
- **EN**: Introduces declarations for `XtensaDAGToDAGISelLegacy`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `XtensaDAGToDAGISelLegacy` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 124-131
```cpp
  StringRef getPassName() const override {
    return "Xtensa DAG->DAG Pattern Instruction Selection";
  }
};
} // end anonymous namespace

char XtensaDAGToDAGISelLegacy::ID = 0;

```
- **EN**: Implements logic around `getPassName`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getPassName` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 132-140
```cpp
FunctionPass *llvm::createXtensaISelDag(XtensaTargetMachine &TM,
                                        CodeGenOptLevel OptLevel) {
  return new XtensaDAGToDAGISelLegacy(TM, OptLevel);
}

void XtensaDAGToDAGISel::Select(SDNode *Node) {
  SDLoc DL(Node);
  EVT VT = Node->getValueType(0);

```
- **EN**: Implements logic around `createXtensaISelDag`, `XtensaDAGToDAGISelLegacy`, `Select`, `DL`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `createXtensaISelDag`, `XtensaDAGToDAGISelLegacy`, `Select`, `DL`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 141-154
```cpp
  // If we have a custom node, we already have selected!
  if (Node->isMachineOpcode()) {
    Node->setNodeId(-1);
    return;
  }

  switch (Node->getOpcode()) {
  case ISD::SHL: {
    SDValue N0 = Node->getOperand(0);
    SDValue N1 = Node->getOperand(1);
    auto *C = dyn_cast<ConstantSDNode>(N1);
    // If C is constant in range [1..31] then we can generate SLLI
    // instruction using pattern matching, otherwise generate SLL.
    if (!C || C->isZero()) {
```
- **EN**: Implements logic around `setNodeId`, `getOperand`, `dyn_cast<ConstantSDNode>`; this block uses `switch`-based dispatch; applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `setNodeId`, `getOperand`, `dyn_cast<ConstantSDNode>` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 155-167
```cpp
      SDNode *SSL = CurDAG->getMachineNode(Xtensa::SSL, DL, MVT::Glue, N1);
      SDNode *SLL =
          CurDAG->getMachineNode(Xtensa::SLL, DL, VT, N0, SDValue(SSL, 0));
      ReplaceNode(Node, SLL);
      return;
    }
    break;
  }
  case ISD::SRL: {
    SDValue N0 = Node->getOperand(0);
    SDValue N1 = Node->getOperand(1);
    auto *C = dyn_cast<ConstantSDNode>(N1);

```
- **EN**: Implements logic around `getMachineNode`, `ReplaceNode`, `getOperand`, `dyn_cast<ConstantSDNode>`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getMachineNode`, `ReplaceNode`, `getOperand`, `dyn_cast<ConstantSDNode>` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 168-180
```cpp
    // If C is constant then we can generate SRLI
    // instruction using pattern matching or EXTUI, otherwise generate SRL.
    if (C) {
      if (isUInt<4>(C->getZExtValue()))
        break;
      unsigned ShAmt = C->getZExtValue();
      SDNode *EXTUI = CurDAG->getMachineNode(
          Xtensa::EXTUI, DL, VT, N0, CurDAG->getTargetConstant(ShAmt, DL, VT),
          CurDAG->getTargetConstant(32 - ShAmt, DL, VT));
      ReplaceNode(Node, EXTUI);
      return;
    }

```
- **EN**: Implements logic around `getZExtValue`, `getMachineNode`, `getTargetConstant`, `ReplaceNode`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getZExtValue`, `getMachineNode`, `getTargetConstant`, `ReplaceNode` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 181-194
```cpp
    SDNode *SSR = CurDAG->getMachineNode(Xtensa::SSR, DL, MVT::Glue, N1);
    SDNode *SRL =
        CurDAG->getMachineNode(Xtensa::SRL, DL, VT, N0, SDValue(SSR, 0));
    ReplaceNode(Node, SRL);
    return;
  }
  case ISD::SRA: {
    SDValue N0 = Node->getOperand(0);
    SDValue N1 = Node->getOperand(1);
    auto *C = dyn_cast<ConstantSDNode>(N1);
    // If C is constant then we can generate SRAI
    // instruction using pattern matching, otherwise generate SRA.
    if (!C) {
      SDNode *SSR = CurDAG->getMachineNode(Xtensa::SSR, DL, MVT::Glue, N1);
```
- **EN**: Implements logic around `getMachineNode`, `ReplaceNode`, `getOperand`, `dyn_cast<ConstantSDNode>`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getMachineNode`, `ReplaceNode`, `getOperand`, `dyn_cast<ConstantSDNode>` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 195-208
```cpp
      SDNode *SRA =
          CurDAG->getMachineNode(Xtensa::SRA, DL, VT, N0, SDValue(SSR, 0));
      ReplaceNode(Node, SRA);
      return;
    }
    break;
  }
  case XtensaISD::SRCL: {
    SDValue N0 = Node->getOperand(0);
    SDValue N1 = Node->getOperand(1);
    SDValue N2 = Node->getOperand(2);
    SDNode *SSL = CurDAG->getMachineNode(Xtensa::SSL, DL, MVT::Glue, N2);
    SDNode *SRC =
        CurDAG->getMachineNode(Xtensa::SRC, DL, VT, N0, N1, SDValue(SSL, 0));
```
- **EN**: Implements logic around `getMachineNode`, `ReplaceNode`, `getOperand`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getMachineNode`, `ReplaceNode`, `getOperand` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 209-222
```cpp
    ReplaceNode(Node, SRC);
    return;
  }
  case XtensaISD::SRCR: {
    SDValue N0 = Node->getOperand(0);
    SDValue N1 = Node->getOperand(1);
    SDValue N2 = Node->getOperand(2);
    SDNode *SSR = CurDAG->getMachineNode(Xtensa::SSR, DL, MVT::Glue, N2);
    SDNode *SRC =
        CurDAG->getMachineNode(Xtensa::SRC, DL, VT, N0, N1, SDValue(SSR, 0));
    ReplaceNode(Node, SRC);
    return;
  }
  }
```
- **EN**: Implements logic around `ReplaceNode`, `getOperand`, `getMachineNode`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `ReplaceNode`, `getOperand`, `getMachineNode` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 223-235
```cpp

  SelectCode(Node);
}

bool XtensaDAGToDAGISel::SelectInlineAsmMemoryOperand(
    const SDValue &Op, InlineAsm::ConstraintCode ConstraintID,
    std::vector<SDValue> &OutOps) {
  switch (ConstraintID) {
  default:
    llvm_unreachable("Unexpected asm memory constraint");
  case InlineAsm::ConstraintCode::m: {
    SDValue Base, Offset;

```
- **EN**: Implements logic around `SelectCode`, `SelectInlineAsmMemoryOperand`, `llvm_unreachable`; this block uses `switch`-based dispatch; handles SelectionDAG-specific logic.
- **CN**: 围绕 `SelectCode`, `SelectInlineAsmMemoryOperand`, `llvm_unreachable` 实现具体逻辑；这一段使用 `switch` 分派，处理 SelectionDAG 专用逻辑。

### Lines 236-244
```cpp
    selectMemRegAddr(Op, Base, Offset, 4);
    OutOps.push_back(Base);
    OutOps.push_back(Offset);

    return false;
  }
  }
  return false;
}
```
- **EN**: Implements logic around `selectMemRegAddr`, `push_back`; this block returns target-specific results.
- **CN**: 围绕 `selectMemRegAddr`, `push_back` 实现具体逻辑；这一段返回目标相关结果。

## Key Concepts / 关键概念

- **Instruction selection / 指令选择**:
  - **EN**: Matches legalized DAG nodes to concrete machine instructions
  - **CN**: 把合法化后的 DAG 节点匹配为具体机器指令
- **Fixups and relocations / Fixup 与重定位**:
  - **EN**: Bridges symbolic references to concrete relocation records
  - **CN**: 把符号引用连接到具体重定位记录
- **DAG legalization / DAG 合法化**:
  - **EN**: Explains how generic IR-style operations are rewritten for this target
  - **CN**: 说明如何把通用操作改写为该目标可接受的形式
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MCTargetDesc/XtensaMCTargetDesc.h`, `Xtensa.h`, `XtensaSelectionDAGInfo.h`, `XtensaTargetMachine.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/SelectionDAGISel.h`, `llvm/IR/DiagnosticInfo.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `XtensaGenDAGISel.inc`
- **LLVM subsystems / LLVM 子系统**: CodeGen, IR, Support
