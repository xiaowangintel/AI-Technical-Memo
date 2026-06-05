# RISCVISelDAGToDAG.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVISelDAGToDAG.h`
- **Repository**: llvm/llvm-project
- **Purpose**: Declares interfaces, data structures, and helper APIs for SelectionDAG-to-DAG instruction selection for RISC-V. / 声明RISC-V 的 SelectionDAG 到 DAG 指令选择所需的接口、数据结构与辅助 API。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```cpp
//===---- RISCVISelDAGToDAG.h - A dag to dag inst selector for RISC-V -----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines an instruction selector for the RISC-V target.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-26: File prologue and imported dependencies / 文件前言与导入依赖
```cpp

#ifndef LLVM_LIB_TARGET_RISCV_RISCVISELDAGTODAG_H
#define LLVM_LIB_TARGET_RISCV_RISCVISELDAGTODAG_H

#include "RISCV.h"
#include "RISCVTargetMachine.h"
#include "llvm/CodeGen/SelectionDAGISel.h"
#include "llvm/Support/KnownBits.h"

// RISC-V specific code to select RISC-V machine instructions for
// SelectionDAG operations.
namespace llvm {
class RISCVDAGToDAGISel : public SelectionDAGISel {
  const RISCVSubtarget *Subtarget = nullptr;
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 27-37: Function implementation: SelectionDAGISel::runOnMachineFunction / 函数实现：SelectionDAGISel::runOnMachineFunction
```cpp
public:
  RISCVDAGToDAGISel() = delete;

  explicit RISCVDAGToDAGISel(RISCVTargetMachine &TargetMachine,
                             CodeGenOptLevel OptLevel)
      : SelectionDAGISel(TargetMachine, OptLevel) {}

  bool runOnMachineFunction(MachineFunction &MF) override {
    Subtarget = &MF.getSubtarget<RISCVSubtarget>();
    return SelectionDAGISel::runOnMachineFunction(MF);
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 38-47: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  void PreprocessISelDAG() override;
  void PostprocessISelDAG() override;

  void Select(SDNode *Node) override;

  bool SelectInlineAsmMemoryOperand(const SDValue &Op,
                                    InlineAsm::ConstraintCode ConstraintID,
                                    std::vector<SDValue> &OutOps) override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 48-57: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  bool areOffsetsWithinAlignment(SDValue Addr, Align Alignment);

  bool SelectAddrFrameIndex(SDValue Addr, SDValue &Base, SDValue &Offset);
  bool SelectAddrRegImm(SDValue Addr, SDValue &Base, SDValue &Offset);
  bool SelectAddrRegImm9(SDValue Addr, SDValue &Base, SDValue &Offset);
  bool SelectAddrRegImmLsb00000(SDValue Addr, SDValue &Base, SDValue &Offset);

  bool SelectAddrRegRegScale(SDValue Addr, unsigned MaxShiftAmount,
                             SDValue &Base, SDValue &Index, SDValue &Scale);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 58-67: Function implementation: SelectAddrRegRegScale / 函数实现：SelectAddrRegRegScale
```cpp
  template <unsigned MaxShift>
  bool SelectAddrRegRegScale(SDValue Addr, SDValue &Base, SDValue &Index,
                             SDValue &Scale) {
    return SelectAddrRegRegScale(Addr, MaxShift, Base, Index, Scale);
  }

  bool SelectAddrRegZextRegScale(SDValue Addr, unsigned MaxShiftAmount,
                                 unsigned Bits, SDValue &Base, SDValue &Index,
                                 SDValue &Scale);
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 68-85: Function implementation: SelectAddrRegZextRegScale / 函数实现：SelectAddrRegZextRegScale
```cpp
  template <unsigned MaxShift, unsigned Bits>
  bool SelectAddrRegZextRegScale(SDValue Addr, SDValue &Base, SDValue &Index,
                                 SDValue &Scale) {
    return SelectAddrRegZextRegScale(Addr, MaxShift, Bits, Base, Index, Scale);
  }

  bool SelectAddrRegReg(SDValue Addr, SDValue &Base, SDValue &Offset);

  bool tryShrinkShlLogicImm(SDNode *Node);
  bool trySignedBitfieldExtract(SDNode *Node);
  bool trySignedBitfieldInsertInSign(SDNode *Node);
  bool tryUnsignedBitfieldExtract(SDNode *Node, const SDLoc &DL, MVT VT,
                                  SDValue X, unsigned Msb, unsigned Lsb);
  bool tryUnsignedBitfieldInsertInZero(SDNode *Node, const SDLoc &DL, MVT VT,
                                       SDValue X, unsigned Msb, unsigned Lsb);
  bool tryIndexedLoad(SDNode *Node);
  bool tryWideningMulAcc(SDNode *Node, const SDLoc &DL);
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 86-97: Function implementation: selectShiftMaskXLen / 函数实现：selectShiftMaskXLen
```cpp
  bool selectShiftMask(SDValue N, unsigned ShiftWidth, SDValue &ShAmt);
  bool selectShiftMaskXLen(SDValue N, SDValue &ShAmt) {
    return selectShiftMask(N, Subtarget->getXLen(), ShAmt);
  }
  template <unsigned Size> bool selectShiftMask(SDValue N, SDValue &ShAmt) {
    return selectShiftMask(N, Size, ShAmt);
  }

  bool selectSETCC(SDValue N, ISD::CondCode ExpectedCCVal, SDValue &Val);
  bool selectSETNE(SDValue N, SDValue &Val) {
    return selectSETCC(N, ISD::SETNE, Val);
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 98-109: Function implementation: selectSETEQ / 函数实现：selectSETEQ
```cpp
  bool selectSETEQ(SDValue N, SDValue &Val) {
    return selectSETCC(N, ISD::SETEQ, Val);
  }

  bool selectSExtBits(SDValue N, unsigned Bits, SDValue &Val);
  template <unsigned Bits> bool selectSExtBits(SDValue N, SDValue &Val) {
    return selectSExtBits(N, Bits, Val);
  }
  bool selectZExtBits(SDValue N, unsigned Bits, SDValue &Val);
  template <unsigned Bits> bool selectZExtBits(SDValue N, SDValue &Val) {
    return selectZExtBits(N, Bits, Val);
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 110-119: Function implementation: selectSHXADDOp / 函数实现：selectSHXADDOp
```cpp

  bool selectSHXADDOp(SDValue N, unsigned ShAmt, SDValue &Val);
  template <unsigned ShAmt> bool selectSHXADDOp(SDValue N, SDValue &Val) {
    return selectSHXADDOp(N, ShAmt, Val);
  }

  bool selectSHXADD_UWOp(SDValue N, unsigned ShAmt, SDValue &Val);
  template <unsigned ShAmt> bool selectSHXADD_UWOp(SDValue N, SDValue &Val) {
    return selectSHXADD_UWOp(N, ShAmt, Val);
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 120-131: Function implementation: hasAllBUsers / 函数实现：hasAllBUsers
```cpp

  bool selectZExtImm32(SDValue N, SDValue &Val);
  bool selectNegImm(SDValue N, SDValue &Val);
  bool selectInvLogicImm(SDValue N, SDValue &Val);

  bool orDisjoint(const SDNode *Node) const;
  bool hasAllNBitUsers(SDNode *Node, unsigned Bits,
                       const unsigned Depth = 0) const;
  bool hasAllBUsers(SDNode *Node) const { return hasAllNBitUsers(Node, 8); }
  bool hasAllHUsers(SDNode *Node) const { return hasAllNBitUsers(Node, 16); }
  bool hasAllWUsers(SDNode *Node) const { return hasAllNBitUsers(Node, 32); }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 132-141: Function implementation: selectVSplatUimmBits / 函数实现：selectVSplatUimmBits
```cpp
  bool selectSimm5Shl2(SDValue N, SDValue &Simm5, SDValue &Shl2);

  bool selectVLOp(SDValue N, SDValue &VL);

  bool selectVSplat(SDValue N, SDValue &SplatVal);
  bool selectVSplatSimm5(SDValue N, SDValue &SplatVal);
  bool selectVSplatUimm(SDValue N, unsigned Bits, SDValue &SplatVal);
  template <unsigned Bits> bool selectVSplatUimmBits(SDValue N, SDValue &Val) {
    return selectVSplatUimm(N, Bits, Val);
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 142-154: Function implementation: selectRVVSimm5 / 函数实现：selectRVVSimm5
```cpp
  bool selectVSplatSimm5Plus1(SDValue N, SDValue &SplatVal);
  bool selectVSplatSimm5Plus1NoDec(SDValue N, SDValue &SplatVal);
  bool selectVSplatSimm5Plus1NonZero(SDValue N, SDValue &SplatVal);
  bool selectVSplatImm64Neg(SDValue N, SDValue &SplatVal);
  // Matches the splat of a value which can be extended or truncated, such that
  // only the bottom 8 bits are preserved.
  bool selectLow8BitsVSplat(SDValue N, SDValue &SplatVal);
  bool selectScalarFPAsInt(SDValue N, SDValue &Imm);

  bool selectRVVSimm5(SDValue N, unsigned Width, SDValue &Imm);
  template <unsigned Width> bool selectRVVSimm5(SDValue N, SDValue &Imm) {
    return selectRVVSimm5(N, Width, Imm);
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 155-164: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  bool selectVMNOTOp(SDValue N, SDValue &Res);
  bool selectVMNOT_VLOp(SDNode *Parent, SDValue N, SDValue &Res);

  void addVectorLoadStoreOperands(SDNode *Node, unsigned SEWImm,
                                  const SDLoc &DL, unsigned CurOp,
                                  bool IsMasked, bool IsStridedOrIndexed,
                                  SmallVectorImpl<SDValue> &Operands,
                                  bool IsLoad = false, MVT *IndexVT = nullptr);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 165-175: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  void selectVLSEG(SDNode *Node, unsigned NF, bool IsMasked, bool IsStrided);
  void selectVLSEGFF(SDNode *Node, unsigned NF, bool IsMasked);
  void selectVLXSEG(SDNode *Node, unsigned NF, bool IsMasked, bool IsOrdered);
  void selectVSSEG(SDNode *Node, unsigned NF, bool IsMasked, bool IsStrided);
  void selectVSXSEG(SDNode *Node, unsigned NF, bool IsMasked, bool IsOrdered);

  void selectVSETVLI(SDNode *Node);
  void selectXSfmmVSET(SDNode *Node);

  void selectSF_VC_X_SE(SDNode *Node);
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 176-195: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Return the RISC-V condition code that matches the given DAG integer
  // condition code. The CondCode must be one of those supported by the RISC-V
  // ISA (see translateSetCCForBranch).
  static RISCVCC::CondCode getRISCVCCForIntCC(ISD::CondCode CC) {
    switch (CC) {
    default:
      llvm_unreachable("Unsupported CondCode");
    case ISD::SETEQ:
      return RISCVCC::COND_EQ;
    case ISD::SETNE:
      return RISCVCC::COND_NE;
    case ISD::SETLT:
      return RISCVCC::COND_LT;
    case ISD::SETGE:
      return RISCVCC::COND_GE;
    case ISD::SETULT:
      return RISCVCC::COND_LTU;
    case ISD::SETUGE:
      return RISCVCC::COND_GEU;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 196-209: File prologue and imported dependencies / 文件前言与导入依赖
```cpp
  }

// Include the pieces autogenerated from the target description.
#define GET_DAGISEL_DECL
#include "RISCVGenDAGISel.inc"

private:
  bool doPeepholeSExtW(SDNode *Node);
  bool doPeepholeMaskedRVV(MachineSDNode *Node);
  bool doPeepholeNoRegPassThru();
  bool performCombineVMergeAndVOps(SDNode *N);
  bool selectImm64IfCheaper(int64_t Imm, int64_t OrigImm, SDValue N,
                            SDValue &Val);
};
```
**EN:** This opening block combines banner comments with the first wave of includes, giving readers both context and the main compile-time dependencies.

**CN:** 该开头区段同时包含说明性注释与首批 include，让读者先看到文件定位，再看到主要编译期依赖。

### Lines 210-219: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp

class RISCVDAGToDAGISelLegacy : public SelectionDAGISelLegacy {
public:
  static char ID;
  explicit RISCVDAGToDAGISelLegacy(RISCVTargetMachine &TargetMachine,
                                   CodeGenOptLevel OptLevel);
};

} // namespace llvm
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 220-220: Header guard and interface framing / 头文件保护与接口框架
```cpp
#endif
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

## Key Concepts / 关键概念
- **RISC-V backend structure** / **RISC-V 后端结构**

## Dependencies / 依赖关系
- `RISCV.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVTargetMachine.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/SelectionDAGISel.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/Support/KnownBits.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVGenDAGISel.inc` — Directly referenced by this file. / 该文件直接引用的依赖。
