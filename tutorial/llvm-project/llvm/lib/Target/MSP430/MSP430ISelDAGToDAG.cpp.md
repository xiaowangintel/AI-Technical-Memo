# MSP430ISelDAGToDAG.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/MSP430/MSP430ISelDAGToDAG.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Implements DAG-to-DAG instruction selection that turns legalized SelectionDAG nodes into target machine instructions.
  - **CN**: 实现 DAG-to-DAG 指令选择，把合法化后的 SelectionDAG 节点转换为目标机器指令。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- MSP430ISelDAGToDAG.cpp - A dag to dag inst selector for MSP430 ----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines an instruction selector for the MSP430 target.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 12-27
```cpp

#include "MSP430.h"
#include "MSP430SelectionDAGInfo.h"
#include "MSP430TargetMachine.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/SelectionDAG.h"
#include "llvm/CodeGen/SelectionDAGISel.h"
#include "llvm/Config/llvm-config.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/Function.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/raw_ostream.h"
using namespace llvm;

```
- **EN**: Pulls in the headers needed for this implementation, including `MSP430.h`, `MSP430SelectionDAGInfo.h`, `MSP430TargetMachine.h`, `llvm/CodeGen/MachineFrameInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `MSP430.h`, `MSP430SelectionDAGInfo.h`, `MSP430TargetMachine.h`, `llvm/CodeGen/MachineFrameInfo.h`。

### Lines 28-37
```cpp
#define DEBUG_TYPE "msp430-isel"
#define PASS_NAME "MSP430 DAG->DAG Pattern Instruction Selection"

namespace {
  struct MSP430ISelAddressMode {
    enum {
      RegBase,
      FrameIndexBase
    } BaseType = RegBase;

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 38-50
```cpp
    struct {            // This is really a union, discriminated by BaseType!
      SDValue Reg;
      int FrameIndex = 0;
    } Base;

    int16_t Disp = 0;
    const GlobalValue *GV = nullptr;
    const Constant *CP = nullptr;
    const BlockAddress *BlockAddr = nullptr;
    const char *ES = nullptr;
    int JT = -1;
    Align Alignment; // CP alignment.

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 51-68
```cpp
    MSP430ISelAddressMode() = default;

    bool hasSymbolicDisplacement() const {
      return GV != nullptr || CP != nullptr || ES != nullptr || JT != -1;
    }

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
    LLVM_DUMP_METHOD void dump() {
      errs() << "MSP430ISelAddressMode " << this << '\n';
      if (BaseType == RegBase && Base.Reg.getNode() != nullptr) {
        errs() << "Base.Reg ";
        Base.Reg.getNode()->dump();
      } else if (BaseType == FrameIndexBase) {
        errs() << " Base.FrameIndex " << Base.FrameIndex << '\n';
      }
      errs() << " Disp " << Disp << '\n';
      if (GV) {
        errs() << "GV ";
```
- **EN**: Implements logic around `MSP430ISelAddressMode`, `hasSymbolicDisplacement`, `dump`, `errs`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `MSP430ISelAddressMode`, `hasSymbolicDisplacement`, `dump`, `errs`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 69-83
```cpp
        GV->dump();
      } else if (CP) {
        errs() << " CP ";
        CP->dump();
        errs() << " Align" << Alignment.value() << '\n';
      } else if (ES) {
        errs() << "ES ";
        errs() << ES << '\n';
      } else if (JT != -1)
        errs() << " JT" << JT << " Align" << Alignment.value() << '\n';
    }
#endif
  };
}

```
- **EN**: Implements logic around `dump`, `errs`; this block applies conditional target rules.
- **CN**: 围绕 `dump`, `errs` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 84-94
```cpp
/// MSP430DAGToDAGISel - MSP430 specific code to select MSP430 machine
/// instructions for SelectionDAG operations.
///
namespace {
  class MSP430DAGToDAGISel : public SelectionDAGISel {
  public:
    MSP430DAGToDAGISel() = delete;

    MSP430DAGToDAGISel(MSP430TargetMachine &TM, CodeGenOptLevel OptLevel)
        : SelectionDAGISel(TM, OptLevel) {}

```
- **EN**: Introduces declarations for `MSP430DAGToDAGISel`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `MSP430DAGToDAGISel` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 95-103
```cpp
  private:
    bool MatchAddress(SDValue N, MSP430ISelAddressMode &AM);
    bool MatchWrapper(SDValue N, MSP430ISelAddressMode &AM);
    bool MatchAddressBase(SDValue N, MSP430ISelAddressMode &AM);

    bool SelectInlineAsmMemoryOperand(const SDValue &Op,
                                      InlineAsm::ConstraintCode ConstraintID,
                                      std::vector<SDValue> &OutOps) override;

```
- **EN**: Implements logic around `MatchAddress`, `MatchWrapper`, `MatchAddressBase`, `SelectInlineAsmMemoryOperand`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `MatchAddress`, `MatchWrapper`, `MatchAddressBase`, `SelectInlineAsmMemoryOperand` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 104-113
```cpp
    // Include the pieces autogenerated from the target description.
  #include "MSP430GenDAGISel.inc"

    // Main method to transform nodes into machine nodes.
    void Select(SDNode *N) override;

    bool tryIndexedLoad(SDNode *Op);
    bool tryIndexedBinOp(SDNode *Op, SDValue N1, SDValue N2, unsigned Opc8,
                         unsigned Opc16);

```
- **EN**: Pulls in the headers needed for this implementation, including `MSP430GenDAGISel.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `MSP430GenDAGISel.inc`。

### Lines 114-125
```cpp
    bool SelectAddr(SDValue Addr, SDValue &Base, SDValue &Disp);
  };

  class MSP430DAGToDAGISelLegacy : public SelectionDAGISelLegacy {
  public:
    static char ID;
    MSP430DAGToDAGISelLegacy(MSP430TargetMachine &TM, CodeGenOptLevel OptLevel)
        : SelectionDAGISelLegacy(
              ID, std::make_unique<MSP430DAGToDAGISel>(TM, OptLevel)) {}
  };
}  // end anonymous namespace

```
- **EN**: Introduces declarations for `MSP430DAGToDAGISelLegacy`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `MSP430DAGToDAGISelLegacy` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 126-137
```cpp
char MSP430DAGToDAGISelLegacy::ID;

INITIALIZE_PASS(MSP430DAGToDAGISelLegacy, DEBUG_TYPE, PASS_NAME, false, false)

/// createMSP430ISelDag - This pass converts a legalized DAG into a
/// MSP430-specific DAG, ready for instruction scheduling.
///
FunctionPass *llvm::createMSP430ISelDag(MSP430TargetMachine &TM,
                                        CodeGenOptLevel OptLevel) {
  return new MSP430DAGToDAGISelLegacy(TM, OptLevel);
}

```
- **EN**: Implements logic around `INITIALIZE_PASS`, `createMSP430ISelDag`, `MSP430DAGToDAGISelLegacy`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `INITIALIZE_PASS`, `createMSP430ISelDag`, `MSP430DAGToDAGISelLegacy` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 138-146
```cpp
/// MatchWrapper - Try to match MSP430ISD::Wrapper node into an addressing mode.
/// These wrap things that will resolve down into a symbol reference.  If no
/// match is possible, this returns true, otherwise it returns false.
bool MSP430DAGToDAGISel::MatchWrapper(SDValue N, MSP430ISelAddressMode &AM) {
  // If the addressing mode already has a symbol as the displacement, we can
  // never match another symbol.
  if (AM.hasSymbolicDisplacement())
    return true;

```
- **EN**: Implements logic around `MatchWrapper`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `MatchWrapper` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 147-164
```cpp
  SDValue N0 = N.getOperand(0);

  if (GlobalAddressSDNode *G = dyn_cast<GlobalAddressSDNode>(N0)) {
    AM.GV = G->getGlobal();
    AM.Disp += G->getOffset();
    //AM.SymbolFlags = G->getTargetFlags();
  } else if (ConstantPoolSDNode *CP = dyn_cast<ConstantPoolSDNode>(N0)) {
    AM.CP = CP->getConstVal();
    AM.Alignment = CP->getAlign();
    AM.Disp += CP->getOffset();
    //AM.SymbolFlags = CP->getTargetFlags();
  } else if (ExternalSymbolSDNode *S = dyn_cast<ExternalSymbolSDNode>(N0)) {
    AM.ES = S->getSymbol();
    //AM.SymbolFlags = S->getTargetFlags();
  } else if (JumpTableSDNode *J = dyn_cast<JumpTableSDNode>(N0)) {
    AM.JT = J->getIndex();
    //AM.SymbolFlags = J->getTargetFlags();
  } else {
```
- **EN**: Implements logic around `getOperand`, `getGlobal`, `getOffset`, `getConstVal`, ...; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getOperand`, `getGlobal`, `getOffset`, `getConstVal`, ... 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 165-179
```cpp
    AM.BlockAddr = cast<BlockAddressSDNode>(N0)->getBlockAddress();
    //AM.SymbolFlags = cast<BlockAddressSDNode>(N0)->getTargetFlags();
  }
  return false;
}

/// MatchAddressBase - Helper for MatchAddress. Add the specified node to the
/// specified addressing mode without any further recursion.
bool MSP430DAGToDAGISel::MatchAddressBase(SDValue N, MSP430ISelAddressMode &AM) {
  // Is the base register already occupied?
  if (AM.BaseType != MSP430ISelAddressMode::RegBase || AM.Base.Reg.getNode()) {
    // If so, we cannot select it.
    return true;
  }

```
- **EN**: Implements logic around `cast<BlockAddressSDNode>`, `MatchAddressBase`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `cast<BlockAddressSDNode>`, `MatchAddressBase` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 180-188
```cpp
  // Default, generate it as a register.
  AM.BaseType = MSP430ISelAddressMode::RegBase;
  AM.Base.Reg = N;
  return false;
}

bool MSP430DAGToDAGISel::MatchAddress(SDValue N, MSP430ISelAddressMode &AM) {
  LLVM_DEBUG(errs() << "MatchAddress: "; AM.dump());

```
- **EN**: Implements logic around `MatchAddress`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `MatchAddress` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 189-201
```cpp
  switch (N.getOpcode()) {
  default: break;
  case ISD::Constant: {
    uint64_t Val = cast<ConstantSDNode>(N)->getSExtValue();
    AM.Disp += Val;
    return false;
  }

  case MSP430ISD::Wrapper:
    if (!MatchWrapper(N, AM))
      return false;
    break;

```
- **EN**: Implements logic around `cast<ConstantSDNode>`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `cast<ConstantSDNode>` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 202-210
```cpp
  case ISD::FrameIndex:
    if (AM.BaseType == MSP430ISelAddressMode::RegBase
        && AM.Base.Reg.getNode() == nullptr) {
      AM.BaseType = MSP430ISelAddressMode::FrameIndexBase;
      AM.Base.FrameIndex = cast<FrameIndexSDNode>(N)->getIndex();
      return false;
    }
    break;

```
- **EN**: Implements logic around `getNode`, `cast<FrameIndexSDNode>`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNode`, `cast<FrameIndexSDNode>` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 211-221
```cpp
  case ISD::ADD: {
    MSP430ISelAddressMode Backup = AM;
    if (!MatchAddress(N.getNode()->getOperand(0), AM) &&
        !MatchAddress(N.getNode()->getOperand(1), AM))
      return false;
    AM = Backup;
    if (!MatchAddress(N.getNode()->getOperand(1), AM) &&
        !MatchAddress(N.getNode()->getOperand(0), AM))
      return false;
    AM = Backup;

```
- **EN**: Implements logic around `MatchAddress`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `MatchAddress` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 222-239
```cpp
    break;
  }

  case ISD::OR:
    // Handle "X | C" as "X + C" iff X is known to have C bits clear.
    if (ConstantSDNode *CN = dyn_cast<ConstantSDNode>(N.getOperand(1))) {
      MSP430ISelAddressMode Backup = AM;
      uint64_t Offset = CN->getSExtValue();
      // Start with the LHS as an addr mode.
      if (!MatchAddress(N.getOperand(0), AM) &&
          // Address could not have picked a GV address for the displacement.
          AM.GV == nullptr &&
          // Check to see if the LHS & C is zero.
          CurDAG->MaskedValueIsZero(N.getOperand(0), CN->getAPIntValue())) {
        AM.Disp += Offset;
        return false;
      }
      AM = Backup;
```
- **EN**: Implements logic around `getSExtValue`, `MaskedValueIsZero`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getSExtValue`, `MaskedValueIsZero` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 240-248
```cpp
    }
    break;
  }

  return MatchAddressBase(N, AM);
}

/// SelectAddr - returns true if it is able pattern match an addressing mode.
/// It returns the operands which make up the maximal addressing mode it can
```
- **EN**: Implements logic around `MatchAddressBase`; this block returns target-specific results.
- **CN**: 围绕 `MatchAddressBase` 实现具体逻辑；这一段返回目标相关结果。

### Lines 249-260
```cpp
/// match by reference.
bool MSP430DAGToDAGISel::SelectAddr(SDValue N,
                                    SDValue &Base, SDValue &Disp) {
  MSP430ISelAddressMode AM;

  if (MatchAddress(N, AM))
    return false;

  if (AM.BaseType == MSP430ISelAddressMode::RegBase)
    if (!AM.Base.Reg.getNode())
      AM.Base.Reg = CurDAG->getRegister(MSP430::SR, MVT::i16);

```
- **EN**: Implements logic around `SelectAddr`, `getRegister`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `SelectAddr`, `getRegister` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 261-278
```cpp
  Base = (AM.BaseType == MSP430ISelAddressMode::FrameIndexBase)
             ? CurDAG->getTargetFrameIndex(
                   AM.Base.FrameIndex,
                   N.getValueType())
             : AM.Base.Reg;

  if (AM.GV)
    Disp = CurDAG->getTargetGlobalAddress(AM.GV, SDLoc(N),
                                          MVT::i16, AM.Disp,
                                          0/*AM.SymbolFlags*/);
  else if (AM.CP)
    Disp = CurDAG->getTargetConstantPool(AM.CP, MVT::i16, AM.Alignment, AM.Disp,
                                         0 /*AM.SymbolFlags*/);
  else if (AM.ES)
    Disp = CurDAG->getTargetExternalSymbol(AM.ES, MVT::i16, 0/*AM.SymbolFlags*/);
  else if (AM.JT != -1)
    Disp = CurDAG->getTargetJumpTable(AM.JT, MVT::i16, 0/*AM.SymbolFlags*/);
  else if (AM.BlockAddr)
```
- **EN**: Implements logic around `getTargetFrameIndex`, `getValueType`, `getTargetGlobalAddress`, `getTargetConstantPool`, ...; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getTargetFrameIndex`, `getValueType`, `getTargetGlobalAddress`, `getTargetConstantPool`, ... 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 279-296
```cpp
    Disp = CurDAG->getTargetBlockAddress(AM.BlockAddr, MVT::i32, 0,
                                         0/*AM.SymbolFlags*/);
  else
    Disp = CurDAG->getSignedTargetConstant(AM.Disp, SDLoc(N), MVT::i16);

  return true;
}

bool MSP430DAGToDAGISel::SelectInlineAsmMemoryOperand(
    const SDValue &Op, InlineAsm::ConstraintCode ConstraintID,
    std::vector<SDValue> &OutOps) {
  SDValue Op0, Op1;
  switch (ConstraintID) {
  default: return true;
  case InlineAsm::ConstraintCode::m: // memory
    if (!SelectAddr(Op, Op0, Op1))
      return true;
    break;
```
- **EN**: Implements logic around `getTargetBlockAddress`, `getSignedTargetConstant`, `SelectInlineAsmMemoryOperand`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getTargetBlockAddress`, `getSignedTargetConstant`, `SelectInlineAsmMemoryOperand` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 297-308
```cpp
  }

  OutOps.push_back(Op0);
  OutOps.push_back(Op1);
  return false;
}

static bool isValidIndexedLoad(const LoadSDNode *LD) {
  ISD::MemIndexedMode AM = LD->getAddressingMode();
  if (AM != ISD::POST_INC || LD->getExtensionType() != ISD::NON_EXTLOAD)
    return false;

```
- **EN**: Implements logic around `push_back`, `isValidIndexedLoad`, `getAddressingMode`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `push_back`, `isValidIndexedLoad`, `getAddressingMode` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 309-320
```cpp
  EVT VT = LD->getMemoryVT();

  switch (VT.getSimpleVT().SimpleTy) {
  case MVT::i8:
    if (LD->getOffset()->getAsZExtVal() != 1)
      return false;

    break;
  case MVT::i16:
    if (LD->getOffset()->getAsZExtVal() != 2)
      return false;

```
- **EN**: Implements logic around `getMemoryVT`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getMemoryVT` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 321-333
```cpp
    break;
  default:
    return false;
  }

  return true;
}

bool MSP430DAGToDAGISel::tryIndexedLoad(SDNode *N) {
  LoadSDNode *LD = cast<LoadSDNode>(N);
  if (!isValidIndexedLoad(LD))
    return false;

```
- **EN**: Implements logic around `tryIndexedLoad`, `cast<LoadSDNode>`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `tryIndexedLoad`, `cast<LoadSDNode>` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 334-347
```cpp
  MVT VT = LD->getMemoryVT().getSimpleVT();

  unsigned Opcode = 0;
  switch (VT.SimpleTy) {
  case MVT::i8:
    Opcode = MSP430::MOV8rp;
    break;
  case MVT::i16:
    Opcode = MSP430::MOV16rp;
    break;
  default:
    return false;
  }

```
- **EN**: Implements logic around `getMemoryVT`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `getMemoryVT` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 348-362
```cpp
  ReplaceNode(N,
              CurDAG->getMachineNode(Opcode, SDLoc(N), VT, MVT::i16, MVT::Other,
                                     LD->getBasePtr(), LD->getChain()));
  return true;
}

bool MSP430DAGToDAGISel::tryIndexedBinOp(SDNode *Op, SDValue N1, SDValue N2,
                                         unsigned Opc8, unsigned Opc16) {
  if (N1.getOpcode() == ISD::LOAD &&
      N1.hasOneUse() &&
      IsLegalToFold(N1, Op, Op, OptLevel)) {
    LoadSDNode *LD = cast<LoadSDNode>(N1);
    if (!isValidIndexedLoad(LD))
      return false;

```
- **EN**: Implements logic around `ReplaceNode`, `getMachineNode`, `getBasePtr`, `tryIndexedBinOp`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `ReplaceNode`, `getMachineNode`, `getBasePtr`, `tryIndexedBinOp`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 363-376
```cpp
    MVT VT = LD->getMemoryVT().getSimpleVT();
    unsigned Opc = (VT == MVT::i16 ? Opc16 : Opc8);
    MachineMemOperand *MemRef = cast<MemSDNode>(N1)->getMemOperand();
    SDValue Ops0[] = { N2, LD->getBasePtr(), LD->getChain() };
    SDNode *ResNode =
      CurDAG->SelectNodeTo(Op, Opc, VT, MVT::i16, MVT::Other, Ops0);
    CurDAG->setNodeMemRefs(cast<MachineSDNode>(ResNode), {MemRef});
    // Transfer chain.
    ReplaceUses(SDValue(N1.getNode(), 2), SDValue(ResNode, 2));
    // Transfer writeback.
    ReplaceUses(SDValue(N1.getNode(), 1), SDValue(ResNode, 1));
    return true;
  }

```
- **EN**: Implements logic around `getMemoryVT`, `cast<MemSDNode>`, `getBasePtr`, `SelectNodeTo`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getMemoryVT`, `cast<MemSDNode>`, `getBasePtr`, `SelectNodeTo`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 377-390
```cpp
  return false;
}


void MSP430DAGToDAGISel::Select(SDNode *Node) {
  SDLoc dl(Node);

  // If we have a custom node, we already have selected!
  if (Node->isMachineOpcode()) {
    LLVM_DEBUG(errs() << "== "; Node->dump(CurDAG); errs() << "\n");
    Node->setNodeId(-1);
    return;
  }

```
- **EN**: Implements logic around `Select`, `dl`, `setNodeId`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `Select`, `dl`, `setNodeId` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 391-408
```cpp
  // Few custom selection stuff.
  switch (Node->getOpcode()) {
  default: break;
  case ISD::FrameIndex: {
    assert(Node->getValueType(0) == MVT::i16);
    int FI = cast<FrameIndexSDNode>(Node)->getIndex();
    SDValue TFI = CurDAG->getTargetFrameIndex(FI, MVT::i16);
    if (Node->hasOneUse()) {
      CurDAG->SelectNodeTo(Node, MSP430::ADDframe, MVT::i16, TFI,
                           CurDAG->getTargetConstant(0, dl, MVT::i16));
      return;
    }
    ReplaceNode(Node, CurDAG->getMachineNode(
                          MSP430::ADDframe, dl, MVT::i16, TFI,
                          CurDAG->getTargetConstant(0, dl, MVT::i16)));
    return;
  }
  case ISD::LOAD:
```
- **EN**: Implements logic around `assert`, `cast<FrameIndexSDNode>`, `getTargetFrameIndex`, `SelectNodeTo`, ...; this block uses `switch`-based dispatch; applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `assert`, `cast<FrameIndexSDNode>`, `getTargetFrameIndex`, `SelectNodeTo`, ... 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 409-420
```cpp
    if (tryIndexedLoad(Node))
      return;
    // Other cases are autogenerated.
    break;
  case ISD::ADD:
    if (tryIndexedBinOp(Node, Node->getOperand(0), Node->getOperand(1),
                        MSP430::ADD8rp, MSP430::ADD16rp))
      return;
    else if (tryIndexedBinOp(Node, Node->getOperand(1), Node->getOperand(0),
                             MSP430::ADD8rp, MSP430::ADD16rp))
      return;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 421-437
```cpp
    // Other cases are autogenerated.
    break;
  case ISD::SUB:
    if (tryIndexedBinOp(Node, Node->getOperand(0), Node->getOperand(1),
                        MSP430::SUB8rp, MSP430::SUB16rp))
      return;

    // Other cases are autogenerated.
    break;
  case ISD::AND:
    if (tryIndexedBinOp(Node, Node->getOperand(0), Node->getOperand(1),
                        MSP430::AND8rp, MSP430::AND16rp))
      return;
    else if (tryIndexedBinOp(Node, Node->getOperand(1), Node->getOperand(0),
                             MSP430::AND8rp, MSP430::AND16rp))
      return;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 438-447
```cpp
    // Other cases are autogenerated.
    break;
  case ISD::OR:
    if (tryIndexedBinOp(Node, Node->getOperand(0), Node->getOperand(1),
                        MSP430::BIS8rp, MSP430::BIS16rp))
      return;
    else if (tryIndexedBinOp(Node, Node->getOperand(1), Node->getOperand(0),
                             MSP430::BIS8rp, MSP430::BIS16rp))
      return;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 448-457
```cpp
    // Other cases are autogenerated.
    break;
  case ISD::XOR:
    if (tryIndexedBinOp(Node, Node->getOperand(0), Node->getOperand(1),
                        MSP430::XOR8rp, MSP430::XOR16rp))
      return;
    else if (tryIndexedBinOp(Node, Node->getOperand(1), Node->getOperand(0),
                             MSP430::XOR8rp, MSP430::XOR16rp))
      return;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 458-464
```cpp
    // Other cases are autogenerated.
    break;
  }

  // Select the default instruction
  SelectCode(Node);
}
```
- **EN**: Implements logic around `SelectCode`.
- **CN**: 围绕 `SelectCode` 实现具体逻辑。

## Key Concepts / 关键概念

- **Instruction selection / 指令选择**:
  - **EN**: Matches legalized DAG nodes to concrete machine instructions
  - **CN**: 把合法化后的 DAG 节点匹配为具体机器指令
- **DAG legalization / DAG 合法化**:
  - **EN**: Explains how generic IR-style operations are rewritten for this target
  - **CN**: 说明如何把通用操作改写为该目标可接受的形式
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `MSP430.h`, `MSP430SelectionDAGInfo.h`, `MSP430TargetMachine.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/SelectionDAGISel.h`, `llvm/Config/llvm-config.h`, `llvm/IR/Constants.h`, `llvm/IR/Function.h`, `llvm/Support/Debug.h`, `llvm/Support/ErrorHandling.h` ... (+2 more)
- **LLVM subsystems / LLVM 子系统**: CodeGen, IR, Support
