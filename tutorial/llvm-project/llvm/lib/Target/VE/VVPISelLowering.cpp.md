# VVPISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VVPISelLowering.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Customizes SelectionDAG lowering, legalization, calling convention handling, and target-specific DAG nodes.
  - **CN**: 定制 SelectionDAG 降低、合法化、调用约定处理以及目标专用 DAG 节点。

## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```cpp
//===-- VVPISelLowering.cpp - VE DAG Lowering Implementation --------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the lowering and legalization of vector instructions to
// VVP_*layer SDNodes.
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 13-21
```cpp

#include "VECustomDAG.h"
#include "VEISelLowering.h"
#include "VESelectionDAGInfo.h"

using namespace llvm;

#define DEBUG_TYPE "ve-lower"

```
- **EN**: Pulls in the headers needed for this implementation, including `VECustomDAG.h`, `VEISelLowering.h`, `VESelectionDAGInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `VECustomDAG.h`, `VEISelLowering.h`, `VESelectionDAGInfo.h`。

### Lines 22-38
```cpp
SDValue VETargetLowering::splitMaskArithmetic(SDValue Op,
                                              SelectionDAG &DAG) const {
  VECustomDAG CDAG(DAG, Op);
  SDValue AVL =
      CDAG.getConstant(Op.getValueType().getVectorNumElements(), MVT::i32);
  SDValue A = Op->getOperand(0);
  SDValue B = Op->getOperand(1);
  SDValue LoA = CDAG.getUnpack(MVT::v256i1, A, PackElem::Lo, AVL);
  SDValue HiA = CDAG.getUnpack(MVT::v256i1, A, PackElem::Hi, AVL);
  SDValue LoB = CDAG.getUnpack(MVT::v256i1, B, PackElem::Lo, AVL);
  SDValue HiB = CDAG.getUnpack(MVT::v256i1, B, PackElem::Hi, AVL);
  unsigned Opc = Op.getOpcode();
  auto LoRes = CDAG.getNode(Opc, MVT::v256i1, {LoA, LoB});
  auto HiRes = CDAG.getNode(Opc, MVT::v256i1, {HiA, HiB});
  return CDAG.getPack(MVT::v512i1, LoRes, HiRes, AVL);
}

```
- **EN**: Implements logic around `splitMaskArithmetic`, `CDAG`, `getConstant`, `getOperand`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `splitMaskArithmetic`, `CDAG`, `getConstant`, `getOperand`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 39-47
```cpp
SDValue VETargetLowering::lowerToVVP(SDValue Op, SelectionDAG &DAG) const {
  // Can we represent this as a VVP node.
  const unsigned Opcode = Op->getOpcode();
  auto VVPOpcodeOpt = getVVPOpcode(Opcode);
  if (!VVPOpcodeOpt)
    return SDValue();
  unsigned VVPOpcode = *VVPOpcodeOpt;
  const bool FromVP = ISD::isVPOpcode(Opcode);

```
- **EN**: Implements logic around `lowerToVVP`, `getOpcode`, `getVVPOpcode`, `SDValue`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `lowerToVVP`, `getOpcode`, `getVVPOpcode`, `SDValue`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 48-59
```cpp
  // The representative and legalized vector type of this operation.
  VECustomDAG CDAG(DAG, Op);
  // Dispatch to complex lowering functions.
  switch (VVPOpcode) {
  case VEISD::VVP_LOAD:
  case VEISD::VVP_STORE:
    return lowerVVP_LOAD_STORE(Op, CDAG);
  case VEISD::VVP_GATHER:
  case VEISD::VVP_SCATTER:
    return lowerVVP_GATHER_SCATTER(Op, CDAG);
  }

```
- **EN**: Implements logic around `CDAG`, `lowerVVP_LOAD_STORE`, `lowerVVP_GATHER_SCATTER`; this block uses `switch`-based dispatch; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `CDAG`, `lowerVVP_LOAD_STORE`, `lowerVVP_GATHER_SCATTER` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 60-76
```cpp
  EVT OpVecVT = *getIdiomaticVectorType(Op.getNode());
  EVT LegalVecVT = getTypeToTransformTo(*DAG.getContext(), OpVecVT);
  auto Packing = getTypePacking(LegalVecVT.getSimpleVT());

  SDValue AVL;
  SDValue Mask;

  if (FromVP) {
    // All upstream VP SDNodes always have a mask and avl.
    auto MaskIdx = ISD::getVPMaskIdx(Opcode);
    auto AVLIdx = ISD::getVPExplicitVectorLengthIdx(Opcode);
    if (MaskIdx)
      Mask = Op->getOperand(*MaskIdx);
    if (AVLIdx)
      AVL = Op->getOperand(*AVLIdx);
  }

```
- **EN**: Implements logic around `getIdiomaticVectorType`, `getTypeToTransformTo`, `getTypePacking`, `getVPMaskIdx`, ...; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getIdiomaticVectorType`, `getTypeToTransformTo`, `getTypePacking`, `getVPMaskIdx`, ... 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 77-94
```cpp
  // Materialize default mask and avl.
  if (!AVL)
    AVL = CDAG.getConstant(OpVecVT.getVectorNumElements(), MVT::i32);
  if (!Mask)
    Mask = CDAG.getConstantMask(Packing, true);

  assert(LegalVecVT.isSimple());
  if (isVVPUnaryOp(VVPOpcode))
    return CDAG.getNode(VVPOpcode, LegalVecVT, {Op->getOperand(0), Mask, AVL});
  if (isVVPBinaryOp(VVPOpcode))
    return CDAG.getNode(VVPOpcode, LegalVecVT,
                        {Op->getOperand(0), Op->getOperand(1), Mask, AVL});
  if (isVVPReductionOp(VVPOpcode)) {
    auto SrcHasStart = hasReductionStartParam(Op->getOpcode());
    SDValue StartV = SrcHasStart ? Op->getOperand(0) : SDValue();
    SDValue VectorV = Op->getOperand(SrcHasStart ? 1 : 0);
    return CDAG.getLegalReductionOpVVP(VVPOpcode, Op.getValueType(), StartV,
                                       VectorV, Mask, AVL, Op->getFlags());
```
- **EN**: Implements logic around `getConstant`, `getConstantMask`, `assert`, `getNode`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getConstant`, `getConstantMask`, `assert`, `getNode`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 95-112
```cpp
  }

  switch (VVPOpcode) {
  default:
    llvm_unreachable("lowerToVVP called for unexpected SDNode.");
  case VEISD::VVP_FFMA: {
    // VE has a swizzled operand order in FMA (compared to LLVM IR and
    // SDNodes).
    auto X = Op->getOperand(2);
    auto Y = Op->getOperand(0);
    auto Z = Op->getOperand(1);
    return CDAG.getNode(VVPOpcode, LegalVecVT, {X, Y, Z, Mask, AVL});
  }
  case VEISD::VVP_SELECT: {
    auto Mask = Op->getOperand(0);
    auto OnTrue = Op->getOperand(1);
    auto OnFalse = Op->getOperand(2);
    return CDAG.getNode(VVPOpcode, LegalVecVT, {OnTrue, OnFalse, Mask, AVL});
```
- **EN**: Implements logic around `llvm_unreachable`, `getOperand`, `getNode`; this block uses `switch`-based dispatch; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `llvm_unreachable`, `getOperand`, `getNode` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 113-123
```cpp
  }
  case VEISD::VVP_SETCC: {
    EVT LegalResVT = getTypeToTransformTo(*DAG.getContext(), Op.getValueType());
    auto LHS = Op->getOperand(0);
    auto RHS = Op->getOperand(1);
    auto Pred = Op->getOperand(2);
    return CDAG.getNode(VVPOpcode, LegalResVT, {LHS, RHS, Pred, Mask, AVL});
  }
  }
}

```
- **EN**: Implements logic around `getTypeToTransformTo`, `getOperand`, `getNode`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getTypeToTransformTo`, `getOperand`, `getNode` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 124-138
```cpp
SDValue VETargetLowering::lowerVVP_LOAD_STORE(SDValue Op,
                                              VECustomDAG &CDAG) const {
  auto VVPOpc = *getVVPOpcode(Op->getOpcode());
  const bool IsLoad = (VVPOpc == VEISD::VVP_LOAD);

  // Shares.
  SDValue BasePtr = getMemoryPtr(Op);
  SDValue Mask = getNodeMask(Op);
  SDValue Chain = getNodeChain(Op);
  SDValue AVL = getNodeAVL(Op);
  // Store specific.
  SDValue Data = getStoredValue(Op);
  // Load specific.
  SDValue PassThru = getNodePassthru(Op);

```
- **EN**: Implements logic around `lowerVVP_LOAD_STORE`, `getVVPOpcode`, `getMemoryPtr`, `getNodeMask`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `lowerVVP_LOAD_STORE`, `getVVPOpcode`, `getMemoryPtr`, `getNodeMask`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 139-147
```cpp
  SDValue StrideV = getLoadStoreStride(Op, CDAG);

  auto DataVT = *getIdiomaticVectorType(Op.getNode());
  auto Packing = getTypePacking(DataVT);

  // TODO: Infer lower AVL from mask.
  if (!AVL)
    AVL = CDAG.getConstant(DataVT.getVectorNumElements(), MVT::i32);

```
- **EN**: Implements logic around `getLoadStoreStride`, `getIdiomaticVectorType`, `getTypePacking`, `getConstant`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getLoadStoreStride`, `getIdiomaticVectorType`, `getTypePacking`, `getConstant` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 148-158
```cpp
  // Default to the all-true mask.
  if (!Mask)
    Mask = CDAG.getConstantMask(Packing, true);

  if (IsLoad) {
    MVT LegalDataVT = getLegalVectorType(
        Packing, DataVT.getVectorElementType().getSimpleVT());

    auto NewLoadV = CDAG.getNode(VEISD::VVP_LOAD, {LegalDataVT, MVT::Other},
                                 {Chain, BasePtr, StrideV, Mask, AVL});

```
- **EN**: Implements logic around `getConstantMask`, `getLegalVectorType`, `getVectorElementType`, `getNode`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getConstantMask`, `getLegalVectorType`, `getVectorElementType`, `getNode` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 159-170
```cpp
    if (!PassThru || PassThru->isUndef())
      return NewLoadV;

    // Convert passthru to an explicit select node.
    SDValue DataV = CDAG.getNode(VEISD::VVP_SELECT, DataVT,
                                 {NewLoadV, PassThru, Mask, AVL});
    SDValue NewLoadChainV = SDValue(NewLoadV.getNode(), 1);

    // Merge them back into one node.
    return CDAG.getMergeValues({DataV, NewLoadChainV});
  }

```
- **EN**: Implements logic around `getNode`, `SDValue`, `getMergeValues`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNode`, `SDValue`, `getMergeValues` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 171-182
```cpp
  // VVP_STORE
  assert(VVPOpc == VEISD::VVP_STORE);
  if (getTypeAction(*CDAG.getDAG()->getContext(), Data.getValueType()) !=
      TargetLowering::TypeLegal)
    // Doesn't lower store instruction if an operand is not lowered yet.
    // If it isn't, return SDValue().  In this way, LLVM will try to lower
    // store instruction again after lowering all operands.
    return SDValue();
  return CDAG.getNode(VEISD::VVP_STORE, Op.getNode()->getVTList(),
                      {Chain, Data, BasePtr, StrideV, Mask, AVL});
}

```
- **EN**: Implements logic around `assert`, `SDValue`, `getNode`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `assert`, `SDValue`, `getNode` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 183-192
```cpp
SDValue VETargetLowering::splitPackedLoadStore(SDValue Op,
                                               VECustomDAG &CDAG) const {
  auto VVPOC = *getVVPOpcode(Op.getOpcode());
  assert((VVPOC == VEISD::VVP_LOAD) || (VVPOC == VEISD::VVP_STORE));

  MVT DataVT = getIdiomaticVectorType(Op.getNode())->getSimpleVT();
  assert(getTypePacking(DataVT) == Packing::Dense &&
         "Can only split packed load/store");
  MVT SplitDataVT = splitVectorType(DataVT);

```
- **EN**: Implements logic around `splitPackedLoadStore`, `getVVPOpcode`, `assert`, `getIdiomaticVectorType`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `splitPackedLoadStore`, `getVVPOpcode`, `assert`, `getIdiomaticVectorType`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 193-202
```cpp
  assert(!getNodePassthru(Op) &&
         "Should have been folded in lowering to VVP layer");

  // Analyze the operation
  SDValue PackedMask = getNodeMask(Op);
  SDValue PackedAVL = getAnnotatedNodeAVL(Op).first;
  SDValue PackPtr = getMemoryPtr(Op);
  SDValue PackData = getStoredValue(Op);
  SDValue PackStride = getLoadStoreStride(Op, CDAG);

```
- **EN**: Implements logic around `assert`, `getNodeMask`, `getAnnotatedNodeAVL`, `getMemoryPtr`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `assert`, `getNodeMask`, `getAnnotatedNodeAVL`, `getMemoryPtr`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 203-212
```cpp
  unsigned ChainResIdx = PackData ? 0 : 1;

  SDValue PartOps[2];

  SDValue UpperPartAVL; // we will use this for packing things back together
  for (PackElem Part : {PackElem::Hi, PackElem::Lo}) {
    // VP ops already have an explicit mask and AVL. When expanding from non-VP
    // attach those additional inputs here.
    auto SplitTM = CDAG.getTargetSplitMask(PackedMask, PackedAVL, Part);

```
- **EN**: Implements logic around `getTargetSplitMask`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getTargetSplitMask` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 213-222
```cpp
    // Keep track of the (higher) lvl.
    if (Part == PackElem::Hi)
      UpperPartAVL = SplitTM.AVL;

    // Attach non-predicating value operands
    SmallVector<SDValue, 4> OpVec;

    // Chain
    OpVec.push_back(getNodeChain(Op));

```
- **EN**: Implements logic around `push_back`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `push_back` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 223-236
```cpp
    // Data
    if (PackData) {
      SDValue PartData =
          CDAG.getUnpack(SplitDataVT, PackData, Part, SplitTM.AVL);
      OpVec.push_back(PartData);
    }

    // Ptr & Stride
    // Push (ptr + ElemBytes * <Part>, 2 * ElemBytes)
    // Stride info
    // EVT DataVT = LegalizeVectorType(getMemoryDataVT(Op), Op, DAG, Mode);
    OpVec.push_back(CDAG.getSplitPtrOffset(PackPtr, PackStride, Part));
    OpVec.push_back(CDAG.getSplitPtrStride(PackStride));

```
- **EN**: Implements logic around `getUnpack`, `push_back`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getUnpack`, `push_back` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 237-250
```cpp
    // Add predicating args and generate part node
    OpVec.push_back(SplitTM.Mask);
    OpVec.push_back(SplitTM.AVL);

    if (PackData) {
      // Store
      PartOps[(int)Part] = CDAG.getNode(VVPOC, MVT::Other, OpVec);
    } else {
      // Load
      PartOps[(int)Part] =
          CDAG.getNode(VVPOC, {SplitDataVT, MVT::Other}, OpVec);
    }
  }

```
- **EN**: Implements logic around `push_back`, `getNode`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `push_back`, `getNode` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 251-260
```cpp
  // Merge the chains
  SDValue LowChain = SDValue(PartOps[(int)PackElem::Lo].getNode(), ChainResIdx);
  SDValue HiChain = SDValue(PartOps[(int)PackElem::Hi].getNode(), ChainResIdx);
  SDValue FusedChains =
      CDAG.getNode(ISD::TokenFactor, MVT::Other, {LowChain, HiChain});

  // Chain only [store]
  if (PackData)
    return FusedChains;

```
- **EN**: Implements logic around `SDValue`, `getNode`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `SDValue`, `getNode` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 261-269
```cpp
  // Re-pack into full packed vector result
  MVT PackedVT =
      getLegalVectorType(Packing::Dense, DataVT.getVectorElementType());
  SDValue PackedVals = CDAG.getPack(PackedVT, PartOps[(int)PackElem::Lo],
                                    PartOps[(int)PackElem::Hi], UpperPartAVL);

  return CDAG.getMergeValues({PackedVals, FusedChains});
}

```
- **EN**: Implements logic around `getLegalVectorType`, `getPack`, `getMergeValues`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getLegalVectorType`, `getPack`, `getMergeValues` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 270-287
```cpp
SDValue VETargetLowering::lowerVVP_GATHER_SCATTER(SDValue Op,
                                                  VECustomDAG &CDAG) const {
  EVT DataVT = *getIdiomaticVectorType(Op.getNode());
  auto Packing = getTypePacking(DataVT);
  MVT LegalDataVT =
      getLegalVectorType(Packing, DataVT.getVectorElementType().getSimpleVT());

  SDValue AVL = getAnnotatedNodeAVL(Op).first;
  SDValue Index = getGatherScatterIndex(Op);
  SDValue BasePtr = getMemoryPtr(Op);
  SDValue Mask = getNodeMask(Op);
  SDValue Chain = getNodeChain(Op);
  SDValue Scale = getGatherScatterScale(Op);
  SDValue PassThru = getNodePassthru(Op);
  SDValue StoredValue = getStoredValue(Op);
  if (PassThru && PassThru->isUndef())
    PassThru = SDValue();

```
- **EN**: Implements logic around `lowerVVP_GATHER_SCATTER`, `getIdiomaticVectorType`, `getTypePacking`, `getLegalVectorType`, ...; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `lowerVVP_GATHER_SCATTER`, `getIdiomaticVectorType`, `getTypePacking`, `getLegalVectorType`, ... 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 288-297
```cpp
  bool IsScatter = (bool)StoredValue;

  // TODO: Infer lower AVL from mask.
  if (!AVL)
    AVL = CDAG.getConstant(DataVT.getVectorNumElements(), MVT::i32);

  // Default to the all-true mask.
  if (!Mask)
    Mask = CDAG.getConstantMask(Packing, true);

```
- **EN**: Implements logic around `getConstant`, `getConstantMask`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getConstant`, `getConstantMask` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 298-307
```cpp
  SDValue AddressVec =
      CDAG.getGatherScatterAddress(BasePtr, Scale, Index, Mask, AVL);
  if (IsScatter)
    return CDAG.getNode(VEISD::VVP_SCATTER, MVT::Other,
                        {Chain, StoredValue, AddressVec, Mask, AVL});

  // Gather.
  SDValue NewLoadV = CDAG.getNode(VEISD::VVP_GATHER, {LegalDataVT, MVT::Other},
                                  {Chain, AddressVec, Mask, AVL});

```
- **EN**: Implements logic around `getGatherScatterAddress`, `getNode`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getGatherScatterAddress`, `getNode` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 308-317
```cpp
  if (!PassThru)
    return NewLoadV;

  // TODO: Use vvp_select
  SDValue DataV = CDAG.getNode(VEISD::VVP_SELECT, LegalDataVT,
                               {NewLoadV, PassThru, Mask, AVL});
  SDValue NewLoadChainV = SDValue(NewLoadV.getNode(), 1);
  return CDAG.getMergeValues({DataV, NewLoadChainV});
}

```
- **EN**: Implements logic around `getNode`, `SDValue`, `getMergeValues`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNode`, `SDValue`, `getMergeValues` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 318-326
```cpp
SDValue VETargetLowering::legalizeInternalLoadStoreOp(SDValue Op,
                                                      VECustomDAG &CDAG) const {
  LLVM_DEBUG(dbgs() << "::legalizeInternalLoadStoreOp\n";);
  MVT DataVT = getIdiomaticVectorType(Op.getNode())->getSimpleVT();

  // TODO: Recognize packable load,store.
  if (isPackedVectorType(DataVT))
    return splitPackedLoadStore(Op, CDAG);

```
- **EN**: Implements logic around `legalizeInternalLoadStoreOp`, `getIdiomaticVectorType`, `splitPackedLoadStore`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `legalizeInternalLoadStoreOp`, `getIdiomaticVectorType`, `splitPackedLoadStore` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 327-341
```cpp
  return legalizePackedAVL(Op, CDAG);
}

SDValue VETargetLowering::legalizeInternalVectorOp(SDValue Op,
                                                   SelectionDAG &DAG) const {
  LLVM_DEBUG(dbgs() << "::legalizeInternalVectorOp\n";);
  VECustomDAG CDAG(DAG, Op);

  // Dispatch to specialized legalization functions.
  switch (Op->getOpcode()) {
  case VEISD::VVP_LOAD:
  case VEISD::VVP_STORE:
    return legalizeInternalLoadStoreOp(Op, CDAG);
  }

```
- **EN**: Implements logic around `legalizePackedAVL`, `legalizeInternalVectorOp`, `CDAG`, `legalizeInternalLoadStoreOp`; this block uses `switch`-based dispatch; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `legalizePackedAVL`, `legalizeInternalVectorOp`, `CDAG`, `legalizeInternalLoadStoreOp` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 342-350
```cpp
  EVT IdiomVT = Op.getValueType();
  if (isPackedVectorType(IdiomVT) &&
      !supportsPackedMode(Op.getOpcode(), IdiomVT))
    return splitVectorOp(Op, CDAG);

  // TODO: Implement odd/even splitting.
  return legalizePackedAVL(Op, CDAG);
}

```
- **EN**: Implements logic around `getValueType`, `supportsPackedMode`, `splitVectorOp`, `legalizePackedAVL`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getValueType`, `supportsPackedMode`, `splitVectorOp`, `legalizePackedAVL` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 351-361
```cpp
SDValue VETargetLowering::splitVectorOp(SDValue Op, VECustomDAG &CDAG) const {
  MVT ResVT = splitVectorType(Op.getValue(0).getSimpleValueType());

  auto AVLPos = getAVLPos(Op->getOpcode());
  auto MaskPos = getMaskPos(Op->getOpcode());

  SDValue PackedMask = getNodeMask(Op);
  auto AVLPair = getAnnotatedNodeAVL(Op);
  SDValue PackedAVL = AVLPair.first;
  assert(!AVLPair.second && "Expecting non pack-legalized oepration");

```
- **EN**: Implements logic around `splitVectorOp`, `splitVectorType`, `getAVLPos`, `getMaskPos`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `splitVectorOp`, `splitVectorType`, `getAVLPos`, `getMaskPos`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 362-370
```cpp
  // request the parts
  SDValue PartOps[2];

  SDValue UpperPartAVL; // we will use this for packing things back together
  for (PackElem Part : {PackElem::Hi, PackElem::Lo}) {
    // VP ops already have an explicit mask and AVL. When expanding from non-VP
    // attach those additional inputs here.
    auto SplitTM = CDAG.getTargetSplitMask(PackedMask, PackedAVL, Part);

```
- **EN**: Implements logic around `getTargetSplitMask`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getTargetSplitMask` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 371-381
```cpp
    if (Part == PackElem::Hi)
      UpperPartAVL = SplitTM.AVL;

    // Attach non-predicating value operands
    SmallVector<SDValue, 4> OpVec;
    for (unsigned i = 0; i < Op.getNumOperands(); ++i) {
      if (AVLPos && ((int)i) == *AVLPos)
        continue;
      if (MaskPos && ((int)i) == *MaskPos)
        continue;

```
- **EN**: Contains supporting implementation details for the surrounding backend logic.
- **CN**: 包含周边后端逻辑所需的辅助实现细节。

### Lines 382-397
```cpp
      // Value operand
      auto PackedOperand = Op.getOperand(i);
      auto UnpackedOpVT = splitVectorType(PackedOperand.getSimpleValueType());
      SDValue PartV =
          CDAG.getUnpack(UnpackedOpVT, PackedOperand, Part, SplitTM.AVL);
      OpVec.push_back(PartV);
    }

    // Add predicating args and generate part node.
    OpVec.push_back(SplitTM.Mask);
    OpVec.push_back(SplitTM.AVL);
    // Emit legal VVP nodes.
    PartOps[(int)Part] =
        CDAG.getNode(Op.getOpcode(), ResVT, OpVec, Op->getFlags());
  }

```
- **EN**: Implements logic around `getOperand`, `splitVectorType`, `getUnpack`, `push_back`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getOperand`, `splitVectorType`, `getUnpack`, `push_back`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 398-409
```cpp
  // Re-package vectors.
  return CDAG.getPack(Op.getValueType(), PartOps[(int)PackElem::Lo],
                      PartOps[(int)PackElem::Hi], UpperPartAVL);
}

SDValue VETargetLowering::legalizePackedAVL(SDValue Op,
                                            VECustomDAG &CDAG) const {
  LLVM_DEBUG(dbgs() << "::legalizePackedAVL\n";);
  // Only required for VEC and VVP ops.
  if (!isVVPOrVEC(Op->getOpcode()))
    return Op;

```
- **EN**: Implements logic around `getPack`, `legalizePackedAVL`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getPack`, `legalizePackedAVL` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 410-421
```cpp
  // Operation already has a legal AVL.
  auto AVL = getNodeAVL(Op);
  if (isLegalAVL(AVL))
    return Op;

  // Half and round up EVL for 32bit element types.
  SDValue LegalAVL = AVL;
  MVT IdiomVT = getIdiomaticVectorType(Op.getNode())->getSimpleVT();
  if (isPackedVectorType(IdiomVT)) {
    assert(maySafelyIgnoreMask(Op) &&
           "TODO Shift predication from EVL into Mask");

```
- **EN**: Implements logic around `getNodeAVL`, `getIdiomaticVectorType`, `assert`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNodeAVL`, `getIdiomaticVectorType`, `assert` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 422-430
```cpp
    if (auto *ConstAVL = dyn_cast<ConstantSDNode>(AVL)) {
      LegalAVL = CDAG.getConstant((ConstAVL->getZExtValue() + 1) / 2, MVT::i32);
    } else {
      auto ConstOne = CDAG.getConstant(1, MVT::i32);
      auto PlusOne = CDAG.getNode(ISD::ADD, MVT::i32, {AVL, ConstOne});
      LegalAVL = CDAG.getNode(ISD::SRL, MVT::i32, {PlusOne, ConstOne});
    }
  }

```
- **EN**: Implements logic around `getConstant`, `getNode`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getConstant`, `getNode` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 431-444
```cpp
  SDValue AnnotatedLegalAVL = CDAG.annotateLegalAVL(LegalAVL);

  // Copy the operand list.
  int NumOp = Op->getNumOperands();
  auto AVLPos = getAVLPos(Op->getOpcode());
  std::vector<SDValue> FixedOperands;
  for (int i = 0; i < NumOp; ++i) {
    if (AVLPos && (i == *AVLPos)) {
      FixedOperands.push_back(AnnotatedLegalAVL);
      continue;
    }
    FixedOperands.push_back(Op->getOperand(i));
  }

```
- **EN**: Implements logic around `annotateLegalAVL`, `getNumOperands`, `getAVLPos`, `push_back`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `annotateLegalAVL`, `getNumOperands`, `getAVLPos`, `push_back` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 445-450
```cpp
  // Clone the operation with fixed operands.
  auto Flags = Op->getFlags();
  SDValue NewN =
      CDAG.getNode(Op->getOpcode(), Op->getVTList(), FixedOperands, Flags);
  return NewN;
}
```
- **EN**: Implements logic around `getFlags`, `getNode`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getFlags`, `getNode` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

## Key Concepts / 关键概念

- **SelectionDAG lowering / SelectionDAG 降低**:
  - **EN**: Custom lowering/legalization rules for target operations
  - **CN**: 为目标操作定制 lowering 与合法化规则
- **DAG legalization / DAG 合法化**:
  - **EN**: Explains how generic IR-style operations are rewritten for this target
  - **CN**: 说明如何把通用操作改写为该目标可接受的形式

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `VECustomDAG.h`, `VEISelLowering.h`, `VESelectionDAGInfo.h`
