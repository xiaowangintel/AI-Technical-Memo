# VECustomDAG.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/VE/VECustomDAG.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines custom DAG node helpers and wrappers used by the backend's SelectionDAG lowering.
  - **CN**: 定义后端 SelectionDAG lowering 使用的自定义 DAG 节点辅助逻辑与封装。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===------------ VECustomDAG.h - VE Custom DAG Nodes -----------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 8-16
```cpp
//
// This file defines the helper functions that VE uses to lower LLVM code into a
// selection DAG.  For example, hiding SDLoc, and easy to use SDNodeFlags.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_VE_VECUSTOMDAG_H
#define LLVM_LIB_TARGET_VE_VECUSTOMDAG_H

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 17-23
```cpp
#include "VE.h"
#include "VEISelLowering.h"
#include "llvm/CodeGen/SelectionDAG.h"
#include "llvm/CodeGen/TargetLowering.h"

namespace llvm {

```
- **EN**: Pulls in the headers needed for this implementation, including `VE.h`, `VEISelLowering.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/TargetLowering.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `VE.h`, `VEISelLowering.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/TargetLowering.h`。

### Lines 24-31
```cpp
std::optional<unsigned> getVVPOpcode(unsigned Opcode);

bool isVVPUnaryOp(unsigned Opcode);
bool isVVPBinaryOp(unsigned Opcode);
bool isVVPReductionOp(unsigned Opcode);

MVT splitVectorType(MVT VT);

```
- **EN**: Implements logic around `getVVPOpcode`, `isVVPUnaryOp`, `isVVPBinaryOp`, `isVVPReductionOp`, ....
- **CN**: 围绕 `getVVPOpcode`, `isVVPUnaryOp`, `isVVPBinaryOp`, `isVVPReductionOp`, ... 实现具体逻辑。

### Lines 32-39
```cpp
bool isPackedVectorType(EVT SomeVT);

bool isMaskType(EVT SomeVT);

bool isMaskArithmetic(SDValue Op);

bool isVVPOrVEC(unsigned);

```
- **EN**: Implements logic around `isPackedVectorType`, `isMaskType`, `isMaskArithmetic`, `isVVPOrVEC`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `isPackedVectorType`, `isMaskType`, `isMaskArithmetic`, `isVVPOrVEC` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 40-46
```cpp
bool supportsPackedMode(unsigned Opcode, EVT IdiomVT);

bool isPackingSupportOpcode(unsigned Opc);

bool maySafelyIgnoreMask(SDValue Op);

/// The VE backend uses a two-staged process to lower and legalize vector
```
- **EN**: Implements logic around `supportsPackedMode`, `isPackingSupportOpcode`, `maySafelyIgnoreMask`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `supportsPackedMode`, `isPackingSupportOpcode`, `maySafelyIgnoreMask` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 47-60
```cpp
/// instructions:
//
/// 1. VP and standard vector SDNodes are lowered to SDNodes of the VVP_* layer.
//
//     All VVP nodes have a mask and an Active Vector Length (AVL) parameter.
//     The AVL parameters refers to the element position in the vector the VVP
//     node operates on.
//
//
//  2. The VVP SDNodes are legalized. The AVL in a legal VVP node refers to
//     chunks of 64bit. We track this by wrapping the AVL in a LEGALAVL node.
//
//     The AVL mechanism in the VE architecture always refers to chunks of
//     64bit, regardless of the actual element type vector instructions are
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 61-72
```cpp
//     operating on. For vector types v256.32 or v256.64 nothing needs to be
//     legalized since each element occupies a 64bit chunk - there is no
//     difference between counting 64bit chunks or element positions. However,
//     all vector types with > 256 elements store more than one logical element
//     per 64bit chunk and need to be transformed.
//     However legalization is performed, the resulting legal VVP SDNodes will
//     have a LEGALAVL node as their AVL operand. The LEGALAVL nodes wraps
//     around an AVL that refers to 64 bit chunks just as the architecture
//     demands - that is, the wrapped AVL is the correct setting for the VL
//     register for this VVP operation to get the desired behavior.
//
/// AVL Functions {
```
- **EN**: Documents the next group of declarations or explains a target-specific rule.
- **CN**: 为接下来的声明分组提供说明，或解释目标相关规则。

### Lines 73-81
```cpp
// The AVL operand position of this node.
std::optional<int> getAVLPos(unsigned);

// Whether this is a LEGALAVL node.
bool isLegalAVL(SDValue AVL);

// The AVL operand of this node.
SDValue getNodeAVL(SDValue);

```
- **EN**: Implements logic around `getAVLPos`, `isLegalAVL`, `getNodeAVL`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getAVLPos`, `isLegalAVL`, `getNodeAVL` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 82-90
```cpp
// Mask position of this node.
std::optional<int> getMaskPos(unsigned);

SDValue getNodeMask(SDValue);

// Return the AVL operand of this node. If it is a LEGALAVL node, unwrap it.
// Return with the boolean whether unwrapping happened.
std::pair<SDValue, bool> getAnnotatedNodeAVL(SDValue);

```
- **EN**: Implements logic around `getMaskPos`, `getNodeMask`, `getAnnotatedNodeAVL`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getMaskPos`, `getNodeMask`, `getAnnotatedNodeAVL` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 91-98
```cpp
/// } AVL Functions

/// Node Properties {

std::optional<EVT> getIdiomaticVectorType(SDNode *Op);

SDValue getLoadStoreStride(SDValue Op, VECustomDAG &CDAG);

```
- **EN**: Implements logic around `getIdiomaticVectorType`, `getLoadStoreStride`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getIdiomaticVectorType`, `getLoadStoreStride` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 99-106
```cpp
SDValue getMemoryPtr(SDValue Op);

SDValue getNodeChain(SDValue Op);

SDValue getStoredValue(SDValue Op);

SDValue getNodePassthru(SDValue Op);

```
- **EN**: Implements logic around `getMemoryPtr`, `getNodeChain`, `getStoredValue`, `getNodePassthru`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getMemoryPtr`, `getNodeChain`, `getStoredValue`, `getNodePassthru` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 107-116
```cpp
SDValue getGatherScatterIndex(SDValue Op);

SDValue getGatherScatterScale(SDValue Op);

unsigned getScalarReductionOpcode(unsigned VVPOC, bool IsMask);

// Whether this VP_REDUCE_*/ VECREDUCE_*/VVP_REDUCE_* SDNode has a start
// parameter.
bool hasReductionStartParam(unsigned VVPOC);

```
- **EN**: Implements logic around `getGatherScatterIndex`, `getGatherScatterScale`, `getScalarReductionOpcode`, `hasReductionStartParam`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getGatherScatterIndex`, `getGatherScatterScale`, `getScalarReductionOpcode`, `hasReductionStartParam` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 117-123
```cpp
/// } Node Properties

enum class Packing {
  Normal = 0, // 256 element standard mode.
  Dense = 1   // 512 element packed mode.
};

```
- **EN**: Introduces declarations for `Packing`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `Packing` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 124-134
```cpp
// Get the vector or mask register type for this packing and element type.
MVT getLegalVectorType(Packing P, MVT ElemVT);

// Whether this type belongs to a packed mask or vector register.
Packing getTypePacking(EVT);

enum class PackElem : int8_t {
  Lo = 0, // Integer (63, 32]
  Hi = 1  // Float   (32,  0]
};

```
- **EN**: Introduces declarations for `PackElem`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `PackElem` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 135-141
```cpp
struct VETargetMasks {
  SDValue Mask;
  SDValue AVL;
  VETargetMasks(SDValue Mask = SDValue(), SDValue AVL = SDValue())
      : Mask(Mask), AVL(AVL) {}
};

```
- **EN**: Introduces declarations for `VETargetMasks`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `VETargetMasks` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 142-148
```cpp
class VECustomDAG {
  SelectionDAG &DAG;
  SDLoc DL;

public:
  SelectionDAG *getDAG() const { return &DAG; }

```
- **EN**: Introduces declarations for `VECustomDAG`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `VECustomDAG` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 149-155
```cpp
  VECustomDAG(SelectionDAG &DAG, SDLoc DL) : DAG(DAG), DL(DL) {}

  VECustomDAG(SelectionDAG &DAG, SDValue WhereOp) : DAG(DAG), DL(WhereOp) {}

  VECustomDAG(SelectionDAG &DAG, const SDNode *WhereN) : DAG(DAG), DL(WhereN) {}

  /// getNode {
```
- **EN**: Implements logic around `VECustomDAG`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `VECustomDAG` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 156-163
```cpp
  SDValue getNode(unsigned OC, SDVTList VTL, ArrayRef<SDValue> OpV,
                  std::optional<SDNodeFlags> Flags = std::nullopt) const {
    auto N = DAG.getNode(OC, DL, VTL, OpV);
    if (Flags)
      N->setFlags(*Flags);
    return N;
  }

```
- **EN**: Implements logic around `getNode`, `setFlags`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNode`, `setFlags` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 164-171
```cpp
  SDValue getNode(unsigned OC, ArrayRef<EVT> ResVT, ArrayRef<SDValue> OpV,
                  std::optional<SDNodeFlags> Flags = std::nullopt) const {
    auto N = DAG.getNode(OC, DL, ResVT, OpV);
    if (Flags)
      N->setFlags(*Flags);
    return N;
  }

```
- **EN**: Implements logic around `getNode`, `setFlags`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNode`, `setFlags` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 172-179
```cpp
  SDValue getNode(unsigned OC, EVT ResVT, ArrayRef<SDValue> OpV,
                  std::optional<SDNodeFlags> Flags = std::nullopt) const {
    auto N = DAG.getNode(OC, DL, ResVT, OpV);
    if (Flags)
      N->setFlags(*Flags);
    return N;
  }

```
- **EN**: Implements logic around `getNode`, `setFlags`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNode`, `setFlags` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 180-187
```cpp
  SDValue getUNDEF(EVT VT) const { return DAG.getUNDEF(VT); }
  /// } getNode

  /// Legalizing getNode {
  SDValue getLegalReductionOpVVP(unsigned VVPOpcode, EVT ResVT, SDValue StartV,
                                 SDValue VectorV, SDValue Mask, SDValue AVL,
                                 SDNodeFlags Flags) const;
  /// } Legalizing getNode
```
- **EN**: Implements logic around `getUNDEF`, `getLegalReductionOpVVP`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getUNDEF`, `getLegalReductionOpVVP` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 188-197
```cpp

  /// Packing {
  SDValue getUnpack(EVT DestVT, SDValue Vec, PackElem Part, SDValue AVL) const;
  SDValue getPack(EVT DestVT, SDValue LoVec, SDValue HiVec, SDValue AVL) const;
  /// } Packing

  SDValue getMergeValues(ArrayRef<SDValue> Values) const {
    return DAG.getMergeValues(Values, DL);
  }

```
- **EN**: Implements logic around `getUnpack`, `getPack`, `getMergeValues`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getUnpack`, `getPack`, `getMergeValues` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 198-204
```cpp
  SDValue getConstant(uint64_t Val, EVT VT, bool IsTarget = false,
                      bool IsOpaque = false) const;

  SDValue getConstantMask(Packing Packing, bool AllTrue) const;
  SDValue getMaskBroadcast(EVT ResultVT, SDValue Scalar, SDValue AVL) const;
  SDValue getBroadcast(EVT ResultVT, SDValue Scalar, SDValue AVL) const;

```
- **EN**: Implements logic around `getConstant`, `getConstantMask`, `getMaskBroadcast`, `getBroadcast`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getConstant`, `getConstantMask`, `getMaskBroadcast`, `getBroadcast` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 205-218
```cpp
  // Wrap AVL in a LEGALAVL node (unless it is one already).
  SDValue annotateLegalAVL(SDValue AVL) const;
  VETargetMasks getTargetSplitMask(SDValue RawMask, SDValue RawAVL,
                                   PackElem Part) const;

  // Splitting support
  SDValue getSplitPtrOffset(SDValue Ptr, SDValue ByteStride,
                            PackElem Part) const;
  SDValue getSplitPtrStride(SDValue PackStride) const;
  SDValue getGatherScatterAddress(SDValue BasePtr, SDValue Scale, SDValue Index,
                                  SDValue Mask, SDValue AVL) const;
  EVT getVectorVT(EVT ElemVT, unsigned NumElems) const {
    return EVT::getVectorVT(*DAG.getContext(), ElemVT, NumElems);
  }
```
- **EN**: Implements logic around `annotateLegalAVL`, `getTargetSplitMask`, `getSplitPtrOffset`, `getSplitPtrStride`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `annotateLegalAVL`, `getTargetSplitMask`, `getSplitPtrOffset`, `getSplitPtrStride`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 219-223
```cpp
};

} // namespace llvm

#endif // LLVM_LIB_TARGET_VE_VECUSTOMDAG_H
```
- **EN**: Introduces declarations for `llvm`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `llvm` 等声明，定义本文件后续使用的数据结构或接口。

## Key Concepts / 关键概念

- **DAG legalization / DAG 合法化**:
  - **EN**: Explains how generic IR-style operations are rewritten for this target
  - **CN**: 说明如何把通用操作改写为该目标可接受的形式

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `VE.h`, `VEISelLowering.h`, `llvm/CodeGen/SelectionDAG.h`, `llvm/CodeGen/TargetLowering.h`
- **LLVM subsystems / LLVM 子系统**: CodeGen
- **Generated macros / 生成宏**: `GET_VE_VECUSTOMDAG_H`
