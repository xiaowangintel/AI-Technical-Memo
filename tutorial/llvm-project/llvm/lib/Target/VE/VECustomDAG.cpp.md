# VECustomDAG.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VECustomDAG.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines custom DAG node helpers and wrappers used by the backend's SelectionDAG lowering.
  - **CN**: 定义后端 SelectionDAG lowering 使用的自定义 DAG 节点辅助逻辑与封装。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===-- VECustomDAG.h - VE Custom DAG Nodes ------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the interfaces that VE uses to lower LLVM code into a
// selection DAG.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 13-22
```cpp

#include "VECustomDAG.h"
#include "VESelectionDAGInfo.h"

#ifndef DEBUG_TYPE
#define DEBUG_TYPE "vecustomdag"
#endif

namespace llvm {

```
- **EN**: Pulls in the headers needed for this implementation, including `VECustomDAG.h`, `VESelectionDAGInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `VECustomDAG.h`, `VESelectionDAGInfo.h`。

### Lines 23-34
```cpp
bool isPackedVectorType(EVT SomeVT) {
  if (!SomeVT.isVector())
    return false;
  return SomeVT.getVectorNumElements() > StandardVectorWidth;
}

MVT splitVectorType(MVT VT) {
  if (!VT.isVector())
    return VT;
  return MVT::getVectorVT(VT.getVectorElementType(), StandardVectorWidth);
}

```
- **EN**: Implements logic around `isPackedVectorType`, `getVectorNumElements`, `splitVectorType`, `getVectorVT`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `isPackedVectorType`, `getVectorNumElements`, `splitVectorType`, `getVectorVT` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 35-44
```cpp
MVT getLegalVectorType(Packing P, MVT ElemVT) {
  return MVT::getVectorVT(ElemVT, P == Packing::Normal ? StandardVectorWidth
                                                       : PackedVectorWidth);
}

Packing getTypePacking(EVT VT) {
  assert(VT.isVector());
  return isPackedVectorType(VT) ? Packing::Dense : Packing::Normal;
}

```
- **EN**: Implements logic around `getLegalVectorType`, `getVectorVT`, `getTypePacking`, `assert`, ...; this block returns target-specific results.
- **CN**: 围绕 `getLegalVectorType`, `getVectorVT`, `getTypePacking`, `assert`, ... 实现具体逻辑；这一段返回目标相关结果。

### Lines 45-61
```cpp
bool isMaskType(EVT SomeVT) {
  if (!SomeVT.isVector())
    return false;
  return SomeVT.getVectorElementType() == MVT::i1;
}

bool isMaskArithmetic(SDValue Op) {
  switch (Op.getOpcode()) {
  default:
    return false;
  case ISD::AND:
  case ISD::XOR:
  case ISD::OR:
    return isMaskType(Op.getValueType());
  }
}

```
- **EN**: Implements logic around `isMaskType`, `getVectorElementType`, `isMaskArithmetic`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `isMaskType`, `getVectorElementType`, `isMaskArithmetic` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 62-79
```cpp
/// \returns the VVP_* SDNode opcode corresponsing to \p OC.
std::optional<unsigned> getVVPOpcode(unsigned Opcode) {
  switch (Opcode) {
  case ISD::MLOAD:
    return VEISD::VVP_LOAD;
  case ISD::MSTORE:
    return VEISD::VVP_STORE;
#define HANDLE_VP_TO_VVP(VPOPC, VVPNAME)                                       \
  case ISD::VPOPC:                                                             \
    return VEISD::VVPNAME;
#define ADD_VVP_OP(VVPNAME, SDNAME)                                            \
  case VEISD::VVPNAME:                                                         \
  case ISD::SDNAME:                                                            \
    return VEISD::VVPNAME;
#include "VVPNodes.def"
  // TODO: Map those in VVPNodes.def too
  case ISD::EXPERIMENTAL_VP_STRIDED_LOAD:
    return VEISD::VVP_LOAD;
```
- **EN**: Pulls in the headers needed for this implementation, including `VVPNodes.def`.
- **CN**: 引入该实现所需的头文件，其中包括 `VVPNodes.def`。

### Lines 80-89
```cpp
  case ISD::EXPERIMENTAL_VP_STRIDED_STORE:
    return VEISD::VVP_STORE;
  }
  return std::nullopt;
}

bool maySafelyIgnoreMask(SDValue Op) {
  auto VVPOpc = getVVPOpcode(Op->getOpcode());
  auto Opc = VVPOpc.value_or(Op->getOpcode());

```
- **EN**: Implements logic around `maySafelyIgnoreMask`, `getVVPOpcode`, `value_or`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `maySafelyIgnoreMask`, `getVVPOpcode`, `value_or` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 90-101
```cpp
  switch (Opc) {
  case VEISD::VVP_SDIV:
  case VEISD::VVP_UDIV:
  case VEISD::VVP_FDIV:
  case VEISD::VVP_SELECT:
    return false;

  default:
    return true;
  }
}

```
- **EN**: Implements target-specific case analysis using a `switch` over opcodes, fixups, or enum values.
- **CN**: 通过对 opcode、fixup 或枚举值执行 `switch` 分析来实现目标相关逻辑。

### Lines 102-116
```cpp
bool supportsPackedMode(unsigned Opcode, EVT IdiomVT) {
  bool IsPackedOp = isPackedVectorType(IdiomVT);
  bool IsMaskOp = isMaskType(IdiomVT);
  switch (Opcode) {
  default:
    return false;

  case VEISD::VEC_BROADCAST:
    return true;
#define REGISTER_PACKED(VVP_NAME) case VEISD::VVP_NAME:
#include "VVPNodes.def"
    return IsPackedOp && !IsMaskOp;
  }
}

```
- **EN**: Pulls in the headers needed for this implementation, including `VVPNodes.def`.
- **CN**: 引入该实现所需的头文件，其中包括 `VVPNodes.def`。

### Lines 117-126
```cpp
bool isPackingSupportOpcode(unsigned Opc) {
  switch (Opc) {
  case VEISD::VEC_PACK:
  case VEISD::VEC_UNPACK_LO:
  case VEISD::VEC_UNPACK_HI:
    return true;
  }
  return false;
}

```
- **EN**: Implements logic around `isPackingSupportOpcode`; this block uses `switch`-based dispatch; returns target-specific results.
- **CN**: 围绕 `isPackingSupportOpcode` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果。

### Lines 127-136
```cpp
bool isVVPOrVEC(unsigned Opcode) {
  switch (Opcode) {
  case VEISD::VEC_BROADCAST:
#define ADD_VVP_OP(VVPNAME, ...) case VEISD::VVPNAME:
#include "VVPNodes.def"
    return true;
  }
  return false;
}

```
- **EN**: Pulls in the headers needed for this implementation, including `VVPNodes.def`.
- **CN**: 引入该实现所需的头文件，其中包括 `VVPNodes.def`。

### Lines 137-146
```cpp
bool isVVPUnaryOp(unsigned VVPOpcode) {
  switch (VVPOpcode) {
#define ADD_UNARY_VVP_OP(VVPNAME, ...)                                         \
  case VEISD::VVPNAME:                                                         \
    return true;
#include "VVPNodes.def"
  }
  return false;
}

```
- **EN**: Pulls in the headers needed for this implementation, including `VVPNodes.def`.
- **CN**: 引入该实现所需的头文件，其中包括 `VVPNodes.def`。

### Lines 147-156
```cpp
bool isVVPBinaryOp(unsigned VVPOpcode) {
  switch (VVPOpcode) {
#define ADD_BINARY_VVP_OP(VVPNAME, ...)                                        \
  case VEISD::VVPNAME:                                                         \
    return true;
#include "VVPNodes.def"
  }
  return false;
}

```
- **EN**: Pulls in the headers needed for this implementation, including `VVPNodes.def`.
- **CN**: 引入该实现所需的头文件，其中包括 `VVPNodes.def`。

### Lines 157-165
```cpp
bool isVVPReductionOp(unsigned Opcode) {
  switch (Opcode) {
#define ADD_REDUCE_VVP_OP(VVP_NAME, SDNAME) case VEISD::VVP_NAME:
#include "VVPNodes.def"
    return true;
  }
  return false;
}

```
- **EN**: Pulls in the headers needed for this implementation, including `VVPNodes.def`.
- **CN**: 引入该实现所需的头文件，其中包括 `VVPNodes.def`。

### Lines 166-176
```cpp
// Return the AVL operand position for this VVP or VEC Op.
std::optional<int> getAVLPos(unsigned Opc) {
  // This is only available for VP SDNodes
  auto PosOpt = ISD::getVPExplicitVectorLengthIdx(Opc);
  if (PosOpt)
    return *PosOpt;

  // VVP Opcodes.
  if (isVVPBinaryOp(Opc))
    return 3;

```
- **EN**: Implements logic around `getAVLPos`, `getVPExplicitVectorLengthIdx`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getAVLPos`, `getVPExplicitVectorLengthIdx` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 177-188
```cpp
  // VM Opcodes.
  switch (Opc) {
  case VEISD::VEC_BROADCAST:
    return 1;
  case VEISD::VVP_SELECT:
    return 3;
  case VEISD::VVP_LOAD:
    return 4;
  case VEISD::VVP_STORE:
    return 5;
  }

```
- **EN**: Implements target-specific case analysis using a `switch` over opcodes, fixups, or enum values.
- **CN**: 通过对 opcode、fixup 或枚举值执行 `switch` 分析来实现目标相关逻辑。

### Lines 189-197
```cpp
  return std::nullopt;
}

std::optional<int> getMaskPos(unsigned Opc) {
  // This is only available for VP SDNodes
  auto PosOpt = ISD::getVPMaskIdx(Opc);
  if (PosOpt)
    return *PosOpt;

```
- **EN**: Implements logic around `getMaskPos`, `getVPMaskIdx`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getMaskPos`, `getVPMaskIdx` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 198-211
```cpp
  // VVP Opcodes.
  if (isVVPBinaryOp(Opc))
    return 2;

  // Other opcodes.
  switch (Opc) {
  case ISD::MSTORE:
    return 4;
  case ISD::MLOAD:
    return 3;
  case VEISD::VVP_SELECT:
    return 2;
  }

```
- **EN**: Implements target-specific case analysis using a `switch` over opcodes, fixups, or enum values.
- **CN**: 通过对 opcode、fixup 或枚举值执行 `switch` 分析来实现目标相关逻辑。

### Lines 212-222
```cpp
  return std::nullopt;
}

bool isLegalAVL(SDValue AVL) { return AVL->getOpcode() == VEISD::LEGALAVL; }

/// Node Properties {

SDValue getNodeChain(SDValue Op) {
  if (MemSDNode *MemN = dyn_cast<MemSDNode>(Op.getNode()))
    return MemN->getChain();

```
- **EN**: Implements logic around `isLegalAVL`, `getNodeChain`, `getChain`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `isLegalAVL`, `getNodeChain`, `getChain` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 223-234
```cpp
  switch (Op->getOpcode()) {
  case VEISD::VVP_LOAD:
  case VEISD::VVP_STORE:
    return Op->getOperand(0);
  }
  return SDValue();
}

SDValue getMemoryPtr(SDValue Op) {
  if (auto *MemN = dyn_cast<MemSDNode>(Op.getNode()))
    return MemN->getBasePtr();

```
- **EN**: Implements logic around `getOperand`, `SDValue`, `getMemoryPtr`, `getBasePtr`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getOperand`, `SDValue`, `getMemoryPtr`, `getBasePtr` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 235-243
```cpp
  switch (Op->getOpcode()) {
  case VEISD::VVP_LOAD:
    return Op->getOperand(1);
  case VEISD::VVP_STORE:
    return Op->getOperand(2);
  }
  return SDValue();
}

```
- **EN**: Implements logic around `getOperand`, `SDValue`; this block uses `switch`-based dispatch; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getOperand`, `SDValue` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 244-261
```cpp
std::optional<EVT> getIdiomaticVectorType(SDNode *Op) {
  unsigned OC = Op->getOpcode();

  // For memory ops -> the transfered data type
  if (auto MemN = dyn_cast<MemSDNode>(Op))
    return MemN->getMemoryVT();

  switch (OC) {
  // Standard ISD.
  case ISD::SELECT: // not aliased with VVP_SELECT
  case ISD::CONCAT_VECTORS:
  case ISD::EXTRACT_SUBVECTOR:
  case ISD::VECTOR_SHUFFLE:
  case ISD::BUILD_VECTOR:
  case ISD::SCALAR_TO_VECTOR:
    return Op->getValueType(0);
  }

```
- **EN**: Implements logic around `getIdiomaticVectorType`, `getOpcode`, `getMemoryVT`, `getValueType`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getIdiomaticVectorType`, `getOpcode`, `getMemoryVT`, `getValueType` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 262-270
```cpp
  // Translate to VVP where possible.
  unsigned OriginalOC = OC;
  if (auto VVPOpc = getVVPOpcode(OC))
    OC = *VVPOpc;

  if (isVVPReductionOp(OC))
    return Op->getOperand(hasReductionStartParam(OriginalOC) ? 1 : 0)
        .getValueType();

```
- **EN**: Implements logic around `getOperand`, `getValueType`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getOperand`, `getValueType` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 271-280
```cpp
  switch (OC) {
  default:
  case VEISD::VVP_SETCC:
    return Op->getOperand(0).getValueType();

  case VEISD::VVP_SELECT:
#define ADD_BINARY_VVP_OP(VVP_NAME, ...) case VEISD::VVP_NAME:
#include "VVPNodes.def"
    return Op->getValueType(0);

```
- **EN**: Pulls in the headers needed for this implementation, including `VVPNodes.def`.
- **CN**: 引入该实现所需的头文件，其中包括 `VVPNodes.def`。

### Lines 281-292
```cpp
  case VEISD::VVP_LOAD:
    return Op->getValueType(0);

  case VEISD::VVP_STORE:
    return Op->getOperand(1)->getValueType(0);

  // VEC
  case VEISD::VEC_BROADCAST:
    return Op->getValueType(0);
  }
}

```
- **EN**: Implements logic around `getValueType`, `getOperand`; this block returns target-specific results.
- **CN**: 围绕 `getValueType`, `getOperand` 实现具体逻辑；这一段返回目标相关结果。

### Lines 293-305
```cpp
SDValue getLoadStoreStride(SDValue Op, VECustomDAG &CDAG) {
  switch (Op->getOpcode()) {
  case VEISD::VVP_STORE:
    return Op->getOperand(3);
  case VEISD::VVP_LOAD:
    return Op->getOperand(2);
  }

  if (auto *StoreN = dyn_cast<VPStridedStoreSDNode>(Op.getNode()))
    return StoreN->getStride();
  if (auto *StoreN = dyn_cast<VPStridedLoadSDNode>(Op.getNode()))
    return StoreN->getStride();

```
- **EN**: Implements logic around `getLoadStoreStride`, `getOperand`, `getStride`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getLoadStoreStride`, `getOperand`, `getStride` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 306-316
```cpp
  if (isa<MemSDNode>(Op.getNode())) {
    // Regular MLOAD/MSTORE/LOAD/STORE
    // No stride argument -> use the contiguous element size as stride.
    uint64_t ElemStride = getIdiomaticVectorType(Op.getNode())
                              ->getVectorElementType()
                              .getStoreSize();
    return CDAG.getConstant(ElemStride, MVT::i64);
  }
  return SDValue();
}

```
- **EN**: Implements logic around `getIdiomaticVectorType`, `getVectorElementType`, `getStoreSize`, `getConstant`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getIdiomaticVectorType`, `getVectorElementType`, `getStoreSize`, `getConstant`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 317-332
```cpp
SDValue getGatherScatterIndex(SDValue Op) {
  if (auto *N = dyn_cast<MaskedGatherScatterSDNode>(Op.getNode()))
    return N->getIndex();
  if (auto *N = dyn_cast<VPGatherScatterSDNode>(Op.getNode()))
    return N->getIndex();
  return SDValue();
}

SDValue getGatherScatterScale(SDValue Op) {
  if (auto *N = dyn_cast<MaskedGatherScatterSDNode>(Op.getNode()))
    return N->getScale();
  if (auto *N = dyn_cast<VPGatherScatterSDNode>(Op.getNode()))
    return N->getScale();
  return SDValue();
}

```
- **EN**: Implements logic around `getGatherScatterIndex`, `getIndex`, `SDValue`, `getGatherScatterScale`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getGatherScatterIndex`, `getIndex`, `SDValue`, `getGatherScatterScale`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 333-350
```cpp
SDValue getStoredValue(SDValue Op) {
  switch (Op->getOpcode()) {
  case ISD::EXPERIMENTAL_VP_STRIDED_STORE:
  case VEISD::VVP_STORE:
    return Op->getOperand(1);
  }
  if (auto *StoreN = dyn_cast<StoreSDNode>(Op.getNode()))
    return StoreN->getValue();
  if (auto *StoreN = dyn_cast<MaskedStoreSDNode>(Op.getNode()))
    return StoreN->getValue();
  if (auto *StoreN = dyn_cast<VPStridedStoreSDNode>(Op.getNode()))
    return StoreN->getValue();
  if (auto *StoreN = dyn_cast<VPStoreSDNode>(Op.getNode()))
    return StoreN->getValue();
  if (auto *StoreN = dyn_cast<MaskedScatterSDNode>(Op.getNode()))
    return StoreN->getValue();
  if (auto *StoreN = dyn_cast<VPScatterSDNode>(Op.getNode()))
    return StoreN->getValue();
```
- **EN**: Implements logic around `getStoredValue`, `getOperand`, `getValue`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getStoredValue`, `getOperand`, `getValue` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 351-359
```cpp
  return SDValue();
}

SDValue getNodePassthru(SDValue Op) {
  if (auto *N = dyn_cast<MaskedLoadSDNode>(Op.getNode()))
    return N->getPassThru();
  if (auto *N = dyn_cast<MaskedGatherSDNode>(Op.getNode()))
    return N->getPassThru();

```
- **EN**: Implements logic around `SDValue`, `getNodePassthru`, `getPassThru`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `SDValue`, `getNodePassthru`, `getPassThru` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 360-369
```cpp
  return SDValue();
}

bool hasReductionStartParam(unsigned OPC) {
  // TODO: Ordered reduction opcodes.
  if (ISD::isVPReduction(OPC))
    return true;
  return false;
}

```
- **EN**: Implements logic around `SDValue`, `hasReductionStartParam`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `SDValue`, `hasReductionStartParam` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 370-383
```cpp
unsigned getScalarReductionOpcode(unsigned VVPOC, bool IsMask) {
  assert(!IsMask && "Mask reduction isel");

  switch (VVPOC) {
#define HANDLE_VVP_REDUCE_TO_SCALAR(VVP_RED_ISD, REDUCE_ISD)                   \
  case VEISD::VVP_RED_ISD:                                                     \
    return ISD::REDUCE_ISD;
#include "VVPNodes.def"
  default:
    break;
  }
  llvm_unreachable("Cannot not scalarize this reduction Opcode!");
}

```
- **EN**: Pulls in the headers needed for this implementation, including `VVPNodes.def`.
- **CN**: 引入该实现所需的头文件，其中包括 `VVPNodes.def`。

### Lines 384-395
```cpp
/// } Node Properties

SDValue getNodeAVL(SDValue Op) {
  auto PosOpt = getAVLPos(Op->getOpcode());
  return PosOpt ? Op->getOperand(*PosOpt) : SDValue();
}

SDValue getNodeMask(SDValue Op) {
  auto PosOpt = getMaskPos(Op->getOpcode());
  return PosOpt ? Op->getOperand(*PosOpt) : SDValue();
}

```
- **EN**: Implements logic around `getNodeAVL`, `getAVLPos`, `getOperand`, `getNodeMask`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNodeAVL`, `getAVLPos`, `getOperand`, `getNodeMask`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 396-404
```cpp
std::pair<SDValue, bool> getAnnotatedNodeAVL(SDValue Op) {
  SDValue AVL = getNodeAVL(Op);
  if (!AVL)
    return {SDValue(), true};
  if (isLegalAVL(AVL))
    return {AVL->getOperand(0), true};
  return {AVL, false};
}

```
- **EN**: Implements logic around `getAnnotatedNodeAVL`, `getNodeAVL`, `SDValue`, `getOperand`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getAnnotatedNodeAVL`, `getNodeAVL`, `SDValue`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 405-419
```cpp
SDValue VECustomDAG::getConstant(uint64_t Val, EVT VT, bool IsTarget,
                                 bool IsOpaque) const {
  return DAG.getConstant(Val, DL, VT, IsTarget, IsOpaque);
}

SDValue VECustomDAG::getConstantMask(Packing Packing, bool AllTrue) const {
  auto MaskVT = getLegalVectorType(Packing, MVT::i1);

  // VEISelDAGtoDAG will replace this pattern with the constant-true VM.
  auto TrueVal = DAG.getAllOnesConstant(DL, MVT::i32);
  auto AVL = getConstant(MaskVT.getVectorNumElements(), MVT::i32);
  auto Res = getNode(VEISD::VEC_BROADCAST, MaskVT, {TrueVal, AVL});
  if (AllTrue)
    return Res;

```
- **EN**: Implements logic around `getConstant`, `getConstantMask`, `getLegalVectorType`, `getAllOnesConstant`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getConstant`, `getConstantMask`, `getLegalVectorType`, `getAllOnesConstant`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 420-429
```cpp
  return DAG.getNOT(DL, Res, Res.getValueType());
}

SDValue VECustomDAG::getMaskBroadcast(EVT ResultVT, SDValue Scalar,
                                      SDValue AVL) const {
  // Constant mask splat.
  if (auto BcConst = dyn_cast<ConstantSDNode>(Scalar))
    return getConstantMask(getTypePacking(ResultVT),
                           BcConst->getSExtValue() != 0);

```
- **EN**: Implements logic around `getNOT`, `getMaskBroadcast`, `getConstantMask`, `getSExtValue`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNOT`, `getMaskBroadcast`, `getConstantMask`, `getSExtValue` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 430-438
```cpp
  // Expand the broadcast to a vector comparison.
  auto ScalarBoolVT = Scalar.getSimpleValueType();
  assert(ScalarBoolVT == MVT::i32);

  // Cast to i32 ty.
  SDValue CmpElem = DAG.getSExtOrTrunc(Scalar, DL, MVT::i32);
  unsigned ElemCount = ResultVT.getVectorNumElements();
  MVT CmpVecTy = MVT::getVectorVT(ScalarBoolVT, ElemCount);

```
- **EN**: Implements logic around `getSimpleValueType`, `assert`, `getSExtOrTrunc`, `getVectorNumElements`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getSimpleValueType`, `assert`, `getSExtOrTrunc`, `getVectorNumElements`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 439-451
```cpp
  // Broadcast to vector.
  SDValue BCVec =
      DAG.getNode(VEISD::VEC_BROADCAST, DL, CmpVecTy, {CmpElem, AVL});
  SDValue ZeroVec =
      getBroadcast(CmpVecTy, {DAG.getConstant(0, DL, ScalarBoolVT)}, AVL);

  MVT BoolVecTy = MVT::getVectorVT(MVT::i1, ElemCount);

  // Broadcast(Data) != Broadcast(0)
  // TODO: Use a VVP operation for this.
  return DAG.getSetCC(DL, BoolVecTy, BCVec, ZeroVec, ISD::CondCode::SETNE);
}

```
- **EN**: Implements logic around `getNode`, `getBroadcast`, `getVectorVT`, `getSetCC`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNode`, `getBroadcast`, `getVectorVT`, `getSetCC` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 452-469
```cpp
SDValue VECustomDAG::getBroadcast(EVT ResultVT, SDValue Scalar,
                                  SDValue AVL) const {
  assert(ResultVT.isVector());
  auto ScaVT = Scalar.getValueType();

  if (isMaskType(ResultVT))
    return getMaskBroadcast(ResultVT, Scalar, AVL);

  if (isPackedVectorType(ResultVT)) {
    // v512x packed mode broadcast
    // Replicate the scalar reg (f32 or i32) onto the opposing half of the full
    // scalar register. If it's an I64 type, assume that this has already
    // happened.
    if (ScaVT == MVT::f32) {
      Scalar = getNode(VEISD::REPL_F32, MVT::i64, Scalar);
    } else if (ScaVT == MVT::i32) {
      Scalar = getNode(VEISD::REPL_I32, MVT::i64, Scalar);
    }
```
- **EN**: Implements logic around `getBroadcast`, `assert`, `getValueType`, `getMaskBroadcast`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getBroadcast`, `assert`, `getValueType`, `getMaskBroadcast`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 470-480
```cpp
  }

  return getNode(VEISD::VEC_BROADCAST, ResultVT, {Scalar, AVL});
}

SDValue VECustomDAG::annotateLegalAVL(SDValue AVL) const {
  if (isLegalAVL(AVL))
    return AVL;
  return getNode(VEISD::LEGALAVL, AVL.getValueType(), AVL);
}

```
- **EN**: Implements logic around `getNode`, `annotateLegalAVL`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNode`, `annotateLegalAVL` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 481-490
```cpp
SDValue VECustomDAG::getUnpack(EVT DestVT, SDValue Vec, PackElem Part,
                               SDValue AVL) const {
  assert(getAnnotatedNodeAVL(AVL).second && "Expected a pack-legalized AVL");

  // TODO: Peek through VEC_PACK and VEC_BROADCAST(REPL_<sth> ..) operands.
  unsigned OC =
      (Part == PackElem::Lo) ? VEISD::VEC_UNPACK_LO : VEISD::VEC_UNPACK_HI;
  return DAG.getNode(OC, DL, DestVT, Vec, AVL);
}

```
- **EN**: Implements logic around `getUnpack`, `assert`, `getNode`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getUnpack`, `assert`, `getNode` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 491-508
```cpp
SDValue VECustomDAG::getPack(EVT DestVT, SDValue LoVec, SDValue HiVec,
                             SDValue AVL) const {
  assert(getAnnotatedNodeAVL(AVL).second && "Expected a pack-legalized AVL");

  // TODO: Peek through VEC_UNPACK_LO|HI operands.
  return DAG.getNode(VEISD::VEC_PACK, DL, DestVT, LoVec, HiVec, AVL);
}

VETargetMasks VECustomDAG::getTargetSplitMask(SDValue RawMask, SDValue RawAVL,
                                              PackElem Part) const {
  // Adjust AVL for this part
  SDValue NewAVL;
  SDValue OneV = getConstant(1, MVT::i32);
  if (Part == PackElem::Hi)
    NewAVL = getNode(ISD::ADD, MVT::i32, {RawAVL, OneV});
  else
    NewAVL = RawAVL;
  NewAVL = getNode(ISD::SRL, MVT::i32, {NewAVL, OneV});
```
- **EN**: Implements logic around `getPack`, `assert`, `getNode`, `getTargetSplitMask`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getPack`, `assert`, `getNode`, `getTargetSplitMask`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 509-518
```cpp

  NewAVL = annotateLegalAVL(NewAVL);

  // Legalize Mask (unpack or all-true)
  SDValue NewMask;
  if (!RawMask)
    NewMask = getConstantMask(Packing::Normal, true);
  else
    NewMask = getUnpack(MVT::v256i1, RawMask, Part, NewAVL);

```
- **EN**: Implements logic around `annotateLegalAVL`, `getConstantMask`, `getUnpack`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `annotateLegalAVL`, `getConstantMask`, `getUnpack` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 519-530
```cpp
  return VETargetMasks(NewMask, NewAVL);
}

SDValue VECustomDAG::getSplitPtrOffset(SDValue Ptr, SDValue ByteStride,
                                       PackElem Part) const {
  // High starts at base ptr but has more significant bits in the 64bit vector
  // element.
  if (Part == PackElem::Hi)
    return Ptr;
  return getNode(ISD::ADD, MVT::i64, {Ptr, ByteStride});
}

```
- **EN**: Implements logic around `VETargetMasks`, `getSplitPtrOffset`, `getNode`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `VETargetMasks`, `getSplitPtrOffset`, `getNode` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 531-541
```cpp
SDValue VECustomDAG::getSplitPtrStride(SDValue PackStride) const {
  if (auto ConstBytes = dyn_cast<ConstantSDNode>(PackStride))
    return getConstant(2 * ConstBytes->getSExtValue(), MVT::i64);
  return getNode(ISD::SHL, MVT::i64, {PackStride, getConstant(1, MVT::i32)});
}

SDValue VECustomDAG::getGatherScatterAddress(SDValue BasePtr, SDValue Scale,
                                             SDValue Index, SDValue Mask,
                                             SDValue AVL) const {
  EVT IndexVT = Index.getValueType();

```
- **EN**: Implements logic around `getSplitPtrStride`, `getConstant`, `getNode`, `getGatherScatterAddress`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getSplitPtrStride`, `getConstant`, `getNode`, `getGatherScatterAddress`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 542-551
```cpp
  // Apply scale.
  SDValue ScaledIndex;
  if (!Scale || isOneConstant(Scale))
    ScaledIndex = Index;
  else {
    SDValue ScaleBroadcast = getBroadcast(IndexVT, Scale, AVL);
    ScaledIndex =
        getNode(VEISD::VVP_MUL, IndexVT, {Index, ScaleBroadcast, Mask, AVL});
  }

```
- **EN**: Implements logic around `getBroadcast`, `getNode`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getBroadcast`, `getNode` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 552-562
```cpp
  // Add basePtr.
  if (isNullConstant(BasePtr))
    return ScaledIndex;

  // re-constitute pointer vector (basePtr + index * scale)
  SDValue BaseBroadcast = getBroadcast(IndexVT, BasePtr, AVL);
  auto ResPtr =
      getNode(VEISD::VVP_ADD, IndexVT, {BaseBroadcast, ScaledIndex, Mask, AVL});
  return ResPtr;
}

```
- **EN**: Implements logic around `getBroadcast`, `getNode`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getBroadcast`, `getNode` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 563-579
```cpp
SDValue VECustomDAG::getLegalReductionOpVVP(unsigned VVPOpcode, EVT ResVT,
                                            SDValue StartV, SDValue VectorV,
                                            SDValue Mask, SDValue AVL,
                                            SDNodeFlags Flags) const {

  // Optionally attach the start param with a scalar op (where it is
  // unsupported).
  bool scalarizeStartParam = StartV && !hasReductionStartParam(VVPOpcode);
  bool IsMaskReduction = isMaskType(VectorV.getValueType());
  assert(!IsMaskReduction && "TODO Implement");
  auto AttachStartValue = [&](SDValue ReductionResV) {
    if (!scalarizeStartParam)
      return ReductionResV;
    auto ScalarOC = getScalarReductionOpcode(VVPOpcode, IsMaskReduction);
    return getNode(ScalarOC, ResVT, {StartV, ReductionResV});
  };

```
- **EN**: Implements logic around `getLegalReductionOpVVP`, `hasReductionStartParam`, `isMaskType`, `assert`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getLegalReductionOpVVP`, `hasReductionStartParam`, `isMaskType`, `assert`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 580-589
```cpp
  // Fixup: Always Use sequential 'fmul' reduction.
  if (!scalarizeStartParam && StartV) {
    assert(hasReductionStartParam(VVPOpcode));
    return AttachStartValue(
        getNode(VVPOpcode, ResVT, {StartV, VectorV, Mask, AVL}, Flags));
  } else
    return AttachStartValue(
        getNode(VVPOpcode, ResVT, {VectorV, Mask, AVL}, Flags));
}

```
- **EN**: Implements logic around `assert`, `AttachStartValue`, `getNode`; this block applies conditional target rules; returns target-specific results; maps fixups or relocations.
- **CN**: 围绕 `assert`, `AttachStartValue`, `getNode` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，映射 fixup 或重定位。

### Lines 590-590
```cpp
} // namespace llvm
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **DAG legalization / DAG 合法化**:
  - **EN**: Explains how generic IR-style operations are rewritten for this target
  - **CN**: 说明如何把通用操作改写为该目标可接受的形式

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `VECustomDAG.h`, `VESelectionDAGInfo.h`, `VVPNodes.def`
