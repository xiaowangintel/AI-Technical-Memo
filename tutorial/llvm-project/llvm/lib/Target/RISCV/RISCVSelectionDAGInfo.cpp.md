# RISCVSelectionDAGInfo.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVSelectionDAGInfo.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements target-specific SelectionDAG helper hooks for RISC-V. / 实现RISC-V 的目标专用 SelectionDAG 辅助钩子。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
//===----------------------------------------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "RISCVSelectionDAGInfo.h"
#include "RISCVSubtarget.h"
#include "llvm/CodeGen/SelectionDAG.h"
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 13-22: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
#define GET_SDNODE_DESC
#include "RISCVGenSDNodeInfo.inc"

using namespace llvm;

RISCVSelectionDAGInfo::RISCVSelectionDAGInfo()
    : SelectionDAGGenTargetInfo(RISCVGenSDNodeInfo) {}

RISCVSelectionDAGInfo::~RISCVSelectionDAGInfo() = default;
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 23-42: Header guard and interface framing / 头文件保护与接口框架
```cpp
void RISCVSelectionDAGInfo::verifyTargetNode(const SelectionDAG &DAG,
                                             const SDNode *N) const {
  SelectionDAGGenTargetInfo::verifyTargetNode(DAG, N);

#ifndef NDEBUG
  // Some additional checks not yet implemented by verifyTargetNode.
  switch (N->getOpcode()) {
  case RISCVISD::TUPLE_EXTRACT:
    assert(N->getOperand(1).getOpcode() == ISD::TargetConstant &&
           "Expected index to be a target constant!");
    break;
  case RISCVISD::TUPLE_INSERT:
    assert(N->getOperand(2).getOpcode() == ISD::TargetConstant &&
           "Expected index to be a target constant!");
    break;
  case RISCVISD::VDOT4A_VL:
  case RISCVISD::VDOT4AU_VL:
  case RISCVISD::VDOT4ASU_VL: {
    EVT VT = N->getValueType(0);
    assert(VT.isScalableVector() && VT.getVectorElementType() == MVT::i32 &&
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 43-54: Definitions and supporting logic / 定义与支撑逻辑
```cpp
           "Expected result to be an i32 scalable vector");
    assert(N->getOperand(0).getValueType() == VT &&
           N->getOperand(1).getValueType() == VT &&
           N->getOperand(2).getValueType() == VT &&
           "Expected result and first 3 operands to have the same type!");
    EVT MaskVT = N->getOperand(3).getValueType();
    assert(MaskVT.isScalableVector() &&
           MaskVT.getVectorElementCount() == VT.getVectorElementCount() &&
           "Expected mask VT to be an i1 scalable vector with same number of "
           "elements as the result");
    break;
  }
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 55-67: Header guard and interface framing / 头文件保护与接口框架
```cpp
  }
#endif
}

SDValue RISCVSelectionDAGInfo::EmitTargetCodeForMemset(
    SelectionDAG &DAG, const SDLoc &dl, SDValue Chain, SDValue Dst, SDValue Src,
    SDValue Size, Align Alignment, bool isVolatile, bool AlwaysInline,
    MachinePointerInfo DstPtrInfo) const {
  const auto &Subtarget = DAG.getSubtarget<RISCVSubtarget>();
  // We currently do this only for Xqcilsm
  if (!Subtarget.hasVendorXqcilsm())
    return SDValue();
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

### Lines 68-78: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Do this only if we know the size at compile time.
  ConstantSDNode *ConstantSize = dyn_cast<ConstantSDNode>(Size);
  if (!ConstantSize)
    return SDValue();

  uint64_t NumberOfBytesToWrite = ConstantSize->getZExtValue();

  // Do this only if it is word aligned and we write a multiple of 4 bytes.
  if (!(Alignment >= 4) || !((NumberOfBytesToWrite & 3) == 0))
    return SDValue();
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 79-97: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  SmallVector<SDValue, 8> OutChains;
  SDValue SrcValueReplicated = DAG.getNode(ISD::ZERO_EXTEND, dl, MVT::i32, Src);
  int NumberOfWords = NumberOfBytesToWrite / 4;
  MachineFunction &MF = DAG.getMachineFunction();
  auto Volatile =
      isVolatile ? MachineMemOperand::MOVolatile : MachineMemOperand::MONone;

  // Helper for constructing the QC_SETWMI instruction
  auto getSetwmiNode = [&](uint8_t SizeWords, uint8_t OffsetSetwmi) -> SDValue {
    SDValue Ops[] = {Chain, SrcValueReplicated, Dst,
                     DAG.getTargetConstant(SizeWords, dl, MVT::i32),
                     DAG.getTargetConstant(OffsetSetwmi, dl, MVT::i32)};
    MachineMemOperand *BaseMemOperand = MF.getMachineMemOperand(
        DstPtrInfo.getWithOffset(OffsetSetwmi),
        MachineMemOperand::MOStore | Volatile, SizeWords * 4, Align(4));
    return DAG.getMemIntrinsicNode(RISCVISD::QC_SETWMI, dl,
                                   DAG.getVTList(MVT::Other), Ops, MVT::i32,
                                   BaseMemOperand);
  };
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 98-117: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  // If i8 type and constant non-zero value.
  if ((Src.getValueType() == MVT::i8) && !isNullConstant(Src))
    // Replicate byte to word by multiplication with 0x01010101.
    SrcValueReplicated =
        DAG.getNode(ISD::MUL, dl, MVT::i32, SrcValueReplicated,
                    DAG.getConstant(0x01010101ul, dl, MVT::i32));

  // We limit a QC_SETWMI to 16 words or less to improve interruptibility.
  // So for 1-16 words we use a single QC_SETWMI:
  //
  // QC_SETWMI reg1, N, 0(reg2)
  //
  // For 17-32 words we use two QC_SETWMI's with the first as 16 words and the
  // second for the remainder:
  //
  // QC_SETWMI reg1, 16, 0(reg2)
  // QC_SETWMI reg1, N, 64(reg2)
  //
  // For 33-48 words, we would like to use (16, 16, n), but that means the last
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 118-130: Commentary and design intent / 注释与设计意图
```cpp
  // QC_SETWMI needs an offset of 128 which the instruction doesn't support.
  // So in this case we use a length of 15 for the second instruction and we do
  // the rest with the third instruction.
  // This means the maximum inlined number of words is 47 (for now):
  //
  // QC_SETWMI R2, R0, 16, 0
  // QC_SETWMI R2, R0, 15, 64
  // QC_SETWMI R2, R0, N, 124
  //
  // For 48 words or more, call the target independent memset
  if (NumberOfWords >= 48)
    return SDValue();
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 131-145: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (NumberOfWords <= 16) {
    // 1 - 16 words
    return getSetwmiNode(NumberOfWords, 0);
  }

  if (NumberOfWords <= 32) {
    // 17 - 32 words
    OutChains.push_back(getSetwmiNode(NumberOfWords - 16, 64));
    OutChains.push_back(getSetwmiNode(16, 0));
  } else {
    // 33 - 47 words
    OutChains.push_back(getSetwmiNode(NumberOfWords - 31, 124));
    OutChains.push_back(getSetwmiNode(15, 64));
    OutChains.push_back(getSetwmiNode(16, 0));
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 146-148: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  return DAG.getNode(ISD::TokenFactor, dl, MVT::Other, OutChains);
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCVSelectionDAGInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/SelectionDAG.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVGenSDNodeInfo.inc` — Directly referenced by this file. / 该文件直接引用的依赖。
