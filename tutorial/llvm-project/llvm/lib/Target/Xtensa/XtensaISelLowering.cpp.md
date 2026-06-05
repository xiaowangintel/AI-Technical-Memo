# XtensaISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/Xtensa/XtensaISelLowering.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Customizes SelectionDAG lowering, legalization, calling convention handling, and target-specific DAG nodes.
  - **CN**: 定制 SelectionDAG 降低、合法化、调用约定处理以及目标专用 DAG 节点。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32
```cpp
//===- XtensaISelLowering.cpp - Xtensa DAG Lowering Implementation --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the interfaces that Xtensa uses to lower LLVM code into a
// selection DAG.
//
//===----------------------------------------------------------------------===//

#include "XtensaISelLowering.h"
#include "XtensaConstantPoolValue.h"
#include "XtensaInstrInfo.h"
#include "XtensaMachineFunctionInfo.h"
#include "XtensaSelectionDAGInfo.h"
#include "XtensaSubtarget.h"
#include "XtensaTargetMachine.h"
#include "llvm/CodeGen/CallingConvLower.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineJumpTableInfo.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/MathExtras.h"
#include "llvm/Support/raw_ostream.h"
#include <deque>
```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaISelLowering.h`, `XtensaConstantPoolValue.h`, `XtensaInstrInfo.h`, `XtensaMachineFunctionInfo.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaISelLowering.h`, `XtensaConstantPoolValue.h`, `XtensaInstrInfo.h`, `XtensaMachineFunctionInfo.h`。

### Lines 33-54
```cpp

using namespace llvm;

#define DEBUG_TYPE "xtensa-lower"

// Return true if we must use long (in fact, indirect) function call.
// It's simplified version, production implimentation must
// resolve a functions in ROM (usually glibc functions)
static bool isLongCall(const char *str) {
  // Currently always use long calls
  return true;
}

// The calling conventions in XtensaCallingConv.td are described in terms of the
// callee's register window. This function translates registers to the
// corresponding caller window %o register.
static unsigned toCallerWindow(unsigned Reg) {
  if (Reg >= Xtensa::A2 && Reg <= Xtensa::A7)
    return Reg - Xtensa::A2 + Xtensa::A10;
  return Reg;
}

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 55-72
```cpp
XtensaTargetLowering::XtensaTargetLowering(const TargetMachine &TM,
                                           const XtensaSubtarget &STI)
    : TargetLowering(TM, STI), Subtarget(STI) {
  MVT PtrVT = MVT::i32;
  // Set up the register classes.
  addRegisterClass(MVT::i32, &Xtensa::ARRegClass);

  if (Subtarget.hasSingleFloat()) {
    addRegisterClass(MVT::f32, &Xtensa::FPRRegClass);
  }

  if (Subtarget.hasBoolean()) {
    addRegisterClass(MVT::v1i1, &Xtensa::BRRegClass);
  }

  // Set up special registers.
  setStackPointerRegisterToSaveRestore(Xtensa::SP);

```
- **EN**: Implements logic around `XtensaTargetLowering`, `TargetLowering`, `addRegisterClass`, `setStackPointerRegisterToSaveRestore`; this block applies conditional target rules; configures legalization and lowering behavior.
- **CN**: 围绕 `XtensaTargetLowering`, `TargetLowering`, `addRegisterClass`, `setStackPointerRegisterToSaveRestore` 实现具体逻辑；这一段应用条件化的目标规则，配置合法化与 lowering 行为。

### Lines 73-94
```cpp
  setSchedulingPreference(Sched::RegPressure);

  setMinFunctionAlignment(Align(4));

  setOperationAction(ISD::Constant, MVT::i32, Custom);
  setOperationAction(ISD::Constant, MVT::i64, Expand);
  setOperationAction(ISD::ConstantFP, MVT::f32, Expand);
  setOperationAction(ISD::ConstantFP, MVT::f64, Expand);

  setBooleanContents(ZeroOrOneBooleanContent);

  setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i1, Expand);
  setOperationAction(ISD::SIGN_EXTEND_INREG, {MVT::i8, MVT::i16},
                     Subtarget.hasSEXT() ? Legal : Expand);

  setOperationAction(ISD::BITCAST, MVT::i32, Expand);
  setOperationAction(ISD::BITCAST, MVT::f32, Expand);
  setOperationAction(ISD::UINT_TO_FP, MVT::i32, Expand);
  setOperationAction(ISD::SINT_TO_FP, MVT::i32, Expand);
  setOperationAction(ISD::FP_TO_UINT, MVT::i32, Expand);
  setOperationAction(ISD::FP_TO_SINT, MVT::i32, Expand);

```
- **EN**: Implements logic around `setSchedulingPreference`, `setMinFunctionAlignment`, `setOperationAction`, `setBooleanContents`, ...; this block configures legalization and lowering behavior.
- **CN**: 围绕 `setSchedulingPreference`, `setMinFunctionAlignment`, `setOperationAction`, `setBooleanContents`, ... 实现具体逻辑；这一段配置合法化与 lowering 行为。

### Lines 95-112
```cpp
  // No sign extend instructions for i1 and sign extend load i8
  for (MVT VT : MVT::integer_valuetypes()) {
    setLoadExtAction(ISD::SEXTLOAD, VT, MVT::i1, Promote);
    setLoadExtAction(ISD::ZEXTLOAD, VT, MVT::i1, Promote);
    setLoadExtAction(ISD::EXTLOAD, VT, MVT::i1, Promote);
    setLoadExtAction(ISD::SEXTLOAD, VT, MVT::i8, Expand);
  }

  setOperationAction(ISD::ConstantPool, PtrVT, Custom);
  setOperationAction(ISD::GlobalAddress, PtrVT, Custom);
  setOperationAction(ISD::GlobalTLSAddress, PtrVT, Custom);
  setOperationAction(ISD::BlockAddress, PtrVT, Custom);
  setOperationAction(ISD::JumpTable, PtrVT, Custom);

  // Expand jump table branches as address arithmetic followed by an
  // indirect jump.
  setOperationAction(ISD::BR_JT, MVT::Other, Custom);

```
- **EN**: Implements logic around `setLoadExtAction`, `setOperationAction`; this block configures legalization and lowering behavior.
- **CN**: 围绕 `setLoadExtAction`, `setOperationAction` 实现具体逻辑；这一段配置合法化与 lowering 行为。

### Lines 113-130
```cpp
  setOperationAction(ISD::BR_CC, MVT::i32, Legal);
  setOperationAction(ISD::BR_CC, MVT::i64, Expand);

  setOperationAction(ISD::SELECT, MVT::i32, Expand);
  setOperationAction(ISD::SELECT, MVT::f32, Expand);
  setOperationAction(ISD::SELECT_CC, MVT::i32, Custom);

  if (Subtarget.hasSingleFloat()) {
    setOperationAction(ISD::BR_CC, MVT::f32, Legal);
    setOperationAction(ISD::SELECT_CC, MVT::f32, Custom);
  } else {
    setOperationAction(ISD::BR_CC, MVT::f32, Expand);
    setOperationAction(ISD::SELECT_CC, MVT::f32, Expand);
  }

  setOperationAction(ISD::SETCC, MVT::i32, Expand);
  setOperationAction(ISD::SETCC, MVT::f32, Expand);

```
- **EN**: Implements logic around `setOperationAction`; this block applies conditional target rules; configures legalization and lowering behavior.
- **CN**: 围绕 `setOperationAction` 实现具体逻辑；这一段应用条件化的目标规则，配置合法化与 lowering 行为。

### Lines 131-148
```cpp
  setCondCodeAction(ISD::SETGT, MVT::i32, Expand);
  setCondCodeAction(ISD::SETLE, MVT::i32, Expand);
  setCondCodeAction(ISD::SETUGT, MVT::i32, Expand);
  setCondCodeAction(ISD::SETULE, MVT::i32, Expand);

  if (Subtarget.hasMul32())
    setOperationAction(ISD::MUL, MVT::i32, Legal);
  else
    setOperationAction(ISD::MUL, MVT::i32, Expand);

  if (Subtarget.hasMul32High()) {
    setOperationAction(ISD::MULHU, MVT::i32, Legal);
    setOperationAction(ISD::MULHS, MVT::i32, Legal);
  } else {
    setOperationAction(ISD::MULHU, MVT::i32, Expand);
    setOperationAction(ISD::MULHS, MVT::i32, Expand);
  }

```
- **EN**: Implements logic around `setCondCodeAction`, `setOperationAction`; this block applies conditional target rules; configures legalization and lowering behavior.
- **CN**: 围绕 `setCondCodeAction`, `setOperationAction` 实现具体逻辑；这一段应用条件化的目标规则，配置合法化与 lowering 行为。

### Lines 149-166
```cpp
  setOperationAction(ISD::SMUL_LOHI, MVT::i32, Expand);
  setOperationAction(ISD::UMUL_LOHI, MVT::i32, Expand);

  if (Subtarget.hasDiv32()) {
    setOperationAction(ISD::SDIV, MVT::i32, Legal);
    setOperationAction(ISD::UDIV, MVT::i32, Legal);
    setOperationAction(ISD::SREM, MVT::i32, Legal);
    setOperationAction(ISD::UREM, MVT::i32, Legal);
  } else {
    setOperationAction(ISD::SDIV, MVT::i32, Expand);
    setOperationAction(ISD::UDIV, MVT::i32, Expand);
    setOperationAction(ISD::SREM, MVT::i32, Expand);
    setOperationAction(ISD::UREM, MVT::i32, Expand);
  }

  setOperationAction(ISD::SDIVREM, MVT::i32, Expand);
  setOperationAction(ISD::UDIVREM, MVT::i32, Expand);

```
- **EN**: Implements logic around `setOperationAction`; this block applies conditional target rules; configures legalization and lowering behavior.
- **CN**: 围绕 `setOperationAction` 实现具体逻辑；这一段应用条件化的目标规则，配置合法化与 lowering 行为。

### Lines 167-182
```cpp
  setOperationAction(ISD::SHL_PARTS, MVT::i32, Custom);
  setOperationAction(ISD::SRA_PARTS, MVT::i32, Custom);
  setOperationAction(ISD::SRL_PARTS, MVT::i32, Custom);

  setOperationAction(ISD::BSWAP, MVT::i32, Expand);
  setOperationAction(ISD::ROTL, MVT::i32, Expand);
  setOperationAction(ISD::ROTR, MVT::i32, Expand);
  setOperationAction(ISD::CTPOP, MVT::i32, Custom);
  setOperationAction(ISD::CTTZ, MVT::i32, Expand);
  setOperationAction(ISD::CTLZ, MVT::i32, Expand);
  setOperationAction(ISD::CTTZ_ZERO_POISON, MVT::i32, Expand);
  setOperationAction(ISD::CTLZ_ZERO_POISON, MVT::i32, Expand);

  setOperationAction({ISD::SMIN, ISD::SMAX, ISD::UMIN, ISD::UMAX}, MVT::i32,
                     Subtarget.hasMINMAX() ? Legal : Expand);

```
- **EN**: Implements logic around `setOperationAction`, `hasMINMAX`; this block configures legalization and lowering behavior.
- **CN**: 围绕 `setOperationAction`, `hasMINMAX` 实现具体逻辑；这一段配置合法化与 lowering 行为。

### Lines 183-214
```cpp
  // Implement custom stack allocations
  setOperationAction(ISD::DYNAMIC_STACKALLOC, PtrVT, Custom);
  // Implement custom stack save and restore
  setOperationAction(ISD::STACKSAVE, MVT::Other, Custom);
  setOperationAction(ISD::STACKRESTORE, MVT::Other, Custom);

  // VASTART, VAARG and VACOPY need to deal with the Xtensa-specific varargs
  // structure, but VAEND is a no-op.
  setOperationAction(ISD::VASTART, MVT::Other, Custom);
  setOperationAction(ISD::VAARG, MVT::Other, Custom);
  setOperationAction(ISD::VACOPY, MVT::Other, Custom);
  setOperationAction(ISD::VAEND, MVT::Other, Expand);

  // Handle floating-point types.
  for (unsigned I = MVT::FIRST_FP_VALUETYPE; I <= MVT::LAST_FP_VALUETYPE; ++I) {
    MVT VT = MVT::SimpleValueType(I);
    if (isTypeLegal(VT)) {
      if (VT.getSizeInBits() == 32 && Subtarget.hasSingleFloat()) {
        setOperationAction(ISD::FABS, VT, Legal);
        setOperationAction(ISD::FADD, VT, Legal);
        setOperationAction(ISD::FSUB, VT, Legal);
        setOperationAction(ISD::FMA, VT, Legal);
        setOperationAction(ISD::FMUL, VT, Legal);
        setOperationAction(ISD::FNEG, VT, Legal);
      } else {
        setOperationAction(ISD::FABS, VT, Expand);
        setOperationAction(ISD::FADD, VT, Expand);
        setOperationAction(ISD::FSUB, VT, Expand);
        setOperationAction(ISD::FMA, VT, Expand);
        setOperationAction(ISD::FMUL, VT, Expand);
        setOperationAction(ISD::FNEG, VT, Expand);
      }
```
- **EN**: Implements logic around `setOperationAction`, `SimpleValueType`; this block applies conditional target rules; configures legalization and lowering behavior.
- **CN**: 围绕 `setOperationAction`, `SimpleValueType` 实现具体逻辑；这一段应用条件化的目标规则，配置合法化与 lowering 行为。

### Lines 215-244
```cpp

      // TODO: once implemented in InstrInfo uncomment
      setOperationAction(ISD::FSQRT, VT, Expand);
      setOperationAction(ISD::FSIN, VT, Expand);
      setOperationAction(ISD::FCOS, VT, Expand);
      setOperationAction(ISD::FREM, VT, LibCall);
      setOperationAction(ISD::FDIV, VT, Expand);
      setOperationAction(ISD::FPOW, VT, Expand);
      setOperationAction(ISD::FSQRT, VT, Expand);
      setOperationAction(ISD::FCOPYSIGN, VT, Expand);
    }
  }

  // Handle floating-point types.
  if (Subtarget.hasSingleFloat()) {
    setOperationAction(ISD::BITCAST, MVT::i32, Legal);
    setOperationAction(ISD::BITCAST, MVT::f32, Legal);
    setOperationAction(ISD::UINT_TO_FP, MVT::i32, Legal);
    setOperationAction(ISD::SINT_TO_FP, MVT::i32, Legal);
    setOperationAction(ISD::FP_TO_UINT, MVT::i32, Legal);
    setOperationAction(ISD::FP_TO_SINT, MVT::i32, Legal);
  } else {
    setOperationAction(ISD::BITCAST, MVT::i32, Expand);
    setOperationAction(ISD::BITCAST, MVT::f32, Expand);
    setOperationAction(ISD::UINT_TO_FP, MVT::i32, Expand);
    setOperationAction(ISD::SINT_TO_FP, MVT::i32, Expand);
    setOperationAction(ISD::FP_TO_UINT, MVT::i32, Expand);
    setOperationAction(ISD::FP_TO_SINT, MVT::i32, Expand);
  }

```
- **EN**: Implements logic around `setOperationAction`; this block applies conditional target rules; configures legalization and lowering behavior.
- **CN**: 围绕 `setOperationAction` 实现具体逻辑；这一段应用条件化的目标规则，配置合法化与 lowering 行为。

### Lines 245-260
```cpp
  // Floating-point truncation and stores need to be done separately.
  setTruncStoreAction(MVT::f64, MVT::f32, Expand);

  if (Subtarget.hasS32C1I()) {
    setMaxAtomicSizeInBitsSupported(32);
    setMinCmpXchgSizeInBits(32);
  } else if (Subtarget.hasForcedAtomics()) {
    setMaxAtomicSizeInBitsSupported(32);
  } else {
    setMaxAtomicSizeInBitsSupported(0);
  }

  // Compute derived properties from the register classes
  computeRegisterProperties(STI.getRegisterInfo());
}

```
- **EN**: Implements logic around `setTruncStoreAction`, `setMaxAtomicSizeInBitsSupported`, `setMinCmpXchgSizeInBits`, `computeRegisterProperties`; this block applies conditional target rules; configures legalization and lowering behavior.
- **CN**: 围绕 `setTruncStoreAction`, `setMaxAtomicSizeInBitsSupported`, `setMinCmpXchgSizeInBits`, `computeRegisterProperties` 实现具体逻辑；这一段应用条件化的目标规则，配置合法化与 lowering 行为。

### Lines 261-287
```cpp
bool XtensaTargetLowering::isOffsetFoldingLegal(
    const GlobalAddressSDNode *GA) const {
  // The Xtensa target isn't yet aware of offsets.
  return false;
}

bool XtensaTargetLowering::isFPImmLegal(const APFloat &Imm, EVT VT,
                                        bool ForCodeSize) const {
  return false;
}

//===----------------------------------------------------------------------===//
// Inline asm support
//===----------------------------------------------------------------------===//
TargetLowering::ConstraintType
XtensaTargetLowering::getConstraintType(StringRef Constraint) const {
  if (Constraint.size() == 1) {
    switch (Constraint[0]) {
    case 'r':
      return C_RegisterClass;
    default:
      break;
    }
  }
  return TargetLowering::getConstraintType(Constraint);
}

```
- **EN**: Implements logic around `isOffsetFoldingLegal`, `isFPImmLegal`, `getConstraintType`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `isOffsetFoldingLegal`, `isFPImmLegal`, `getConstraintType` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 288-312
```cpp
TargetLowering::ConstraintWeight
XtensaTargetLowering::getSingleConstraintMatchWeight(
    AsmOperandInfo &Info, const char *Constraint) const {
  ConstraintWeight Weight = CW_Invalid;
  Value *CallOperandVal = Info.CallOperandVal;
  // If we don't have a value, we can't do a match,
  // but allow it at the lowest weight.
  if (!CallOperandVal)
    return CW_Default;

  Type *Ty = CallOperandVal->getType();

  // Look at the constraint type.
  switch (*Constraint) {
  default:
    Weight = TargetLowering::getSingleConstraintMatchWeight(Info, Constraint);
    break;
  case 'r':
    if (Ty->isIntegerTy())
      Weight = CW_Register;
    break;
  }
  return Weight;
}

```
- **EN**: Implements logic around `getSingleConstraintMatchWeight`, `getType`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getSingleConstraintMatchWeight`, `getType` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 313-332
```cpp
std::pair<unsigned, const TargetRegisterClass *>
XtensaTargetLowering::getRegForInlineAsmConstraint(
    const TargetRegisterInfo *TRI, StringRef Constraint, MVT VT) const {
  if (Constraint.size() == 1) {
    // GCC Constraint Letters
    switch (Constraint[0]) {
    default:
      break;
    case 'r': // General-purpose register
      return std::make_pair(0U, &Xtensa::ARRegClass);
    }
  }
  return TargetLowering::getRegForInlineAsmConstraint(TRI, Constraint, VT);
}

void XtensaTargetLowering::LowerAsmOperandForConstraint(
    SDValue Op, StringRef Constraint, std::vector<SDValue> &Ops,
    SelectionDAG &DAG) const {
  SDLoc DL(Op);

```
- **EN**: Implements logic around `getRegForInlineAsmConstraint`, `make_pair`, `LowerAsmOperandForConstraint`, `DL`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getRegForInlineAsmConstraint`, `make_pair`, `LowerAsmOperandForConstraint`, `DL` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 333-348
```cpp
  // Only support length 1 constraints for now.
  if (Constraint.size() > 1)
    return;

  TargetLowering::LowerAsmOperandForConstraint(Op, Constraint, Ops, DAG);
}

//===----------------------------------------------------------------------===//
// Calling conventions
//===----------------------------------------------------------------------===//

#include "XtensaGenCallingConv.inc"

static const MCPhysReg IntRegs[] = {Xtensa::A2, Xtensa::A3, Xtensa::A4,
                                    Xtensa::A5, Xtensa::A6, Xtensa::A7};

```
- **EN**: Pulls in the headers needed for this implementation, including `XtensaGenCallingConv.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `XtensaGenCallingConv.inc`。

### Lines 349-370
```cpp
static bool CC_Xtensa_Custom(unsigned ValNo, MVT ValVT, MVT LocVT,
                             CCValAssign::LocInfo LocInfo,
                             ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
                             CCState &State) {
  if (ArgFlags.isByVal()) {
    Align ByValAlign = ArgFlags.getNonZeroByValAlign();
    unsigned ByValSize = ArgFlags.getByValSize();
    if (ByValSize < 4) {
      ByValSize = 4;
    }
    if (ByValAlign < Align(4)) {
      ByValAlign = Align(4);
    }
    unsigned Offset = State.AllocateStack(ByValSize, ByValAlign);
    State.addLoc(CCValAssign::getMem(ValNo, ValVT, Offset, LocVT, LocInfo));
    // Mark all unused registers as allocated to avoid misuse
    // of such registers.
    while (State.AllocateReg(IntRegs))
      ;
    return false;
  }

```
- **EN**: Implements logic around `CC_Xtensa_Custom`, `getNonZeroByValAlign`, `getByValSize`, `Align`, ...; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `CC_Xtensa_Custom`, `getNonZeroByValAlign`, `getByValSize`, `Align`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 371-387
```cpp
  // Promote i8 and i16
  if (LocVT == MVT::i8 || LocVT == MVT::i16) {
    LocVT = MVT::i32;
    if (ArgFlags.isSExt())
      LocInfo = CCValAssign::SExt;
    else if (ArgFlags.isZExt())
      LocInfo = CCValAssign::ZExt;
    else
      LocInfo = CCValAssign::AExt;
  }

  unsigned Register;

  Align OrigAlign = ArgFlags.getNonZeroOrigAlign();
  bool needs64BitAlign = (ValVT == MVT::i32 && OrigAlign == Align(8));
  bool needs128BitAlign = (ValVT == MVT::i32 && OrigAlign == Align(16));

```
- **EN**: Implements logic around `getNonZeroOrigAlign`, `Align`; this block applies conditional target rules.
- **CN**: 围绕 `getNonZeroOrigAlign`, `Align` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 388-412
```cpp
  if (ValVT == MVT::i32) {
    Register = State.AllocateReg(IntRegs);
    // If this is the first part of an i64 arg,
    // the allocated register must be either A2, A4 or A6.
    if (needs64BitAlign && (Register == Xtensa::A3 || Register == Xtensa::A5 ||
                            Register == Xtensa::A7))
      Register = State.AllocateReg(IntRegs);
    // arguments with 16byte alignment must be passed in the first register or
    // passed via stack
    if (needs128BitAlign && (Register != Xtensa::A2))
      while ((Register = State.AllocateReg(IntRegs)))
        ;
    LocVT = MVT::i32;
  } else if (ValVT == MVT::f64) {
    // Allocate int register and shadow next int register.
    Register = State.AllocateReg(IntRegs);
    if (Register == Xtensa::A3 || Register == Xtensa::A5 ||
        Register == Xtensa::A7)
      Register = State.AllocateReg(IntRegs);
    State.AllocateReg(IntRegs);
    LocVT = MVT::i32;
  } else {
    report_fatal_error("Cannot handle this ValVT.");
  }

```
- **EN**: Implements logic around `AllocateReg`, `report_fatal_error`; this block applies conditional target rules.
- **CN**: 围绕 `AllocateReg`, `report_fatal_error` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 413-429
```cpp
  if (!Register) {
    unsigned Offset = State.AllocateStack(ValVT.getStoreSize(), OrigAlign);
    State.addLoc(CCValAssign::getMem(ValNo, ValVT, Offset, LocVT, LocInfo));
  } else {
    State.addLoc(CCValAssign::getReg(ValNo, ValVT, Register, LocVT, LocInfo));
  }

  return false;
}

/// Return the register type for a given MVT
MVT XtensaTargetLowering::getRegisterTypeForCallingConv(LLVMContext &Context,
                                                        CallingConv::ID CC,
                                                        EVT VT) const {
  if (VT.isFloatingPoint())
    return MVT::i32;

```
- **EN**: Implements logic around `AllocateStack`, `addLoc`, `getRegisterTypeForCallingConv`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `AllocateStack`, `addLoc`, `getRegisterTypeForCallingConv` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 430-445
```cpp
  return TargetLowering::getRegisterTypeForCallingConv(Context, CC, VT);
}

CCAssignFn *XtensaTargetLowering::CCAssignFnForCall(CallingConv::ID CC,
                                                    bool IsVarArg) const {
  return CC_Xtensa_Custom;
}

SDValue XtensaTargetLowering::LowerFormalArguments(
    SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
    const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &DL,
    SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
  MachineFunction &MF = DAG.getMachineFunction();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  XtensaMachineFunctionInfo *XtensaFI = MF.getInfo<XtensaMachineFunctionInfo>();

```
- **EN**: Implements logic around `getRegisterTypeForCallingConv`, `CCAssignFnForCall`, `LowerFormalArguments`, `getMachineFunction`, ...; this block returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getRegisterTypeForCallingConv`, `CCAssignFnForCall`, `LowerFormalArguments`, `getMachineFunction`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 446-461
```cpp
  // Used with vargs to acumulate store chains.
  std::vector<SDValue> OutChains;

  // Assign locations to all of the incoming arguments.
  SmallVector<CCValAssign, 16> ArgLocs;
  CCState CCInfo(CallConv, IsVarArg, DAG.getMachineFunction(), ArgLocs,
                 *DAG.getContext());

  CCInfo.AnalyzeFormalArguments(Ins, CCAssignFnForCall(CallConv, IsVarArg));

  for (unsigned i = 0, e = ArgLocs.size(); i != e; ++i) {
    CCValAssign &VA = ArgLocs[i];
    // Arguments stored on registers
    if (VA.isRegLoc()) {
      EVT RegVT = VA.getLocVT();

```
- **EN**: Implements logic around `CCInfo`, `getContext`, `AnalyzeFormalArguments`, `getLocVT`; this block applies conditional target rules; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `CCInfo`, `getContext`, `AnalyzeFormalArguments`, `getLocVT` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 462-478
```cpp
      if (RegVT != MVT::i32)
        report_fatal_error("RegVT not supported by FormalArguments Lowering");

      // Transform the arguments stored on
      // physical registers into virtual ones
      Register Reg = 0;
      MCRegister FrameReg = Subtarget.getRegisterInfo()->getFrameRegister(MF);

      // Argument passed in FrameReg in Windowed ABI we save in A8 (in
      // emitPrologue), so load argument from A8
      if (Subtarget.isWindowedABI() && (VA.getLocReg() == FrameReg)) {
        Reg = MF.addLiveIn(Xtensa::A8, &Xtensa::ARRegClass);
        XtensaFI->setSaveFrameRegister();
      } else {
        Reg = MF.addLiveIn(VA.getLocReg(), &Xtensa::ARRegClass);
      }

```
- **EN**: Implements logic around `report_fatal_error`, `getRegisterInfo`, `addLiveIn`, `setSaveFrameRegister`; this block applies conditional target rules.
- **CN**: 围绕 `report_fatal_error`, `getRegisterInfo`, `addLiveIn`, `setSaveFrameRegister` 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 479-497
```cpp
      SDValue ArgValue = DAG.getCopyFromReg(Chain, DL, Reg, RegVT);

      // If this is an 8 or 16-bit value, it has been passed promoted
      // to 32 bits.  Insert an assert[sz]ext to capture this, then
      // truncate to the right size.
      if (VA.getLocInfo() != CCValAssign::Full) {
        unsigned Opcode = 0;
        if (VA.getLocInfo() == CCValAssign::SExt)
          Opcode = ISD::AssertSext;
        else if (VA.getLocInfo() == CCValAssign::ZExt)
          Opcode = ISD::AssertZext;
        if (Opcode)
          ArgValue = DAG.getNode(Opcode, DL, RegVT, ArgValue,
                                 DAG.getValueType(VA.getValVT()));
        ArgValue = DAG.getNode((VA.getValVT() == MVT::f32) ? ISD::BITCAST
                                                           : ISD::TRUNCATE,
                               DL, VA.getValVT(), ArgValue);
      }

```
- **EN**: Implements logic around `getCopyFromReg`, `getNode`, `getValueType`, `getValVT`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getCopyFromReg`, `getNode`, `getValueType`, `getValVT` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 498-523
```cpp
      InVals.push_back(ArgValue);

    } else {
      assert(VA.isMemLoc());

      EVT ValVT = VA.getValVT();

      // The stack pointer offset is relative to the caller stack frame.
      int FI = MFI.CreateFixedObject(ValVT.getStoreSize(), VA.getLocMemOffset(),
                                     true);

      if (Ins[VA.getValNo()].Flags.isByVal()) {
        // Assume that in this case load operation is created
        SDValue FIN = DAG.getFrameIndex(FI, MVT::i32);
        InVals.push_back(FIN);
      } else {
        // Create load nodes to retrieve arguments from the stack
        SDValue FIN =
            DAG.getFrameIndex(FI, getFrameIndexTy(DAG.getDataLayout()));
        InVals.push_back(DAG.getLoad(
            ValVT, DL, Chain, FIN,
            MachinePointerInfo::getFixedStack(DAG.getMachineFunction(), FI)));
      }
    }
  }

```
- **EN**: Implements logic around `push_back`, `assert`, `getValVT`, `CreateFixedObject`, ...; this block applies conditional target rules; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `push_back`, `assert`, `getValVT`, `CreateFixedObject`, ... 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 524-543
```cpp
  if (IsVarArg) {
    unsigned Idx = CCInfo.getFirstUnallocated(IntRegs);
    unsigned ArgRegsNum = std::size(IntRegs);
    const TargetRegisterClass *RC = &Xtensa::ARRegClass;
    MachineFrameInfo &MFI = MF.getFrameInfo();
    MachineRegisterInfo &RegInfo = MF.getRegInfo();
    unsigned RegSize = 4;
    MVT RegTy = MVT::i32;
    MVT FITy = getFrameIndexTy(DAG.getDataLayout());

    XtensaFI->setVarArgsFirstGPR(Idx + 2); // 2 - number of a2 register

    XtensaFI->setVarArgsOnStackFrameIndex(
        MFI.CreateFixedObject(4, CCInfo.getStackSize(), true));

    // Offset of the first variable argument from stack pointer, and size of
    // the vararg save area. For now, the varargs save area is either zero or
    // large enough to hold a0-a7.
    int VaArgOffset, VarArgsSaveSize;

```
- **EN**: Implements logic around `getFirstUnallocated`, `size`, `getFrameInfo`, `getRegInfo`, ...; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getFirstUnallocated`, `size`, `getFrameInfo`, `getRegInfo`, ... 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 544-563
```cpp
    // If all registers are allocated, then all varargs must be passed on the
    // stack and we don't need to save any argregs.
    if (ArgRegsNum == Idx) {
      VaArgOffset = CCInfo.getStackSize();
      VarArgsSaveSize = 0;
    } else {
      VarArgsSaveSize = RegSize * (ArgRegsNum - Idx);
      VaArgOffset = -VarArgsSaveSize;

      // Record the frame index of the first variable argument
      // which is a value necessary to VASTART.
      int FI = MFI.CreateFixedObject(RegSize, VaArgOffset, true);
      XtensaFI->setVarArgsInRegsFrameIndex(FI);

      // Copy the integer registers that may have been used for passing varargs
      // to the vararg save area.
      for (unsigned I = Idx; I < ArgRegsNum; ++I, VaArgOffset += RegSize) {
        const Register Reg = RegInfo.createVirtualRegister(RC);
        RegInfo.addLiveIn(IntRegs[I], Reg);

```
- **EN**: Implements logic around `getStackSize`, `CreateFixedObject`, `setVarArgsInRegsFrameIndex`, `createVirtualRegister`, ...; this block applies conditional target rules.
- **CN**: 围绕 `getStackSize`, `CreateFixedObject`, `setVarArgsInRegsFrameIndex`, `createVirtualRegister`, ... 实现具体逻辑；这一段应用条件化的目标规则。

### Lines 564-580
```cpp
        SDValue ArgValue = DAG.getCopyFromReg(Chain, DL, Reg, RegTy);
        FI = MFI.CreateFixedObject(RegSize, VaArgOffset, true);
        SDValue PtrOff = DAG.getFrameIndex(FI, FITy);
        SDValue Store = DAG.getStore(Chain, DL, ArgValue, PtrOff,
                                     MachinePointerInfo::getFixedStack(MF, FI));
        OutChains.push_back(Store);
      }
    }
  }

  // All stores are grouped in one node to allow the matching between
  // the size of Ins and InVals. This only happens when on varg functions
  if (!OutChains.empty()) {
    OutChains.push_back(Chain);
    Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, OutChains);
  }

```
- **EN**: Implements logic around `getCopyFromReg`, `CreateFixedObject`, `getFrameIndex`, `getStore`, ...; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getCopyFromReg`, `CreateFixedObject`, `getFrameIndex`, `getStore`, ... 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 581-597
```cpp
  return Chain;
}

SDValue
XtensaTargetLowering::LowerCall(CallLoweringInfo &CLI,
                                SmallVectorImpl<SDValue> &InVals) const {
  SelectionDAG &DAG = CLI.DAG;
  SDLoc &DL = CLI.DL;
  SmallVector<ISD::OutputArg, 32> &Outs = CLI.Outs;
  SmallVector<SDValue, 32> &OutVals = CLI.OutVals;
  SmallVector<ISD::InputArg, 32> &Ins = CLI.Ins;
  SDValue Chain = CLI.Chain;
  SDValue Callee = CLI.Callee;
  bool &IsTailCall = CLI.IsTailCall;
  CallingConv::ID CallConv = CLI.CallConv;
  bool IsVarArg = CLI.IsVarArg;

```
- **EN**: Implements logic around `LowerCall`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerCall` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 598-615
```cpp
  MachineFunction &MF = DAG.getMachineFunction();
  EVT PtrVT = getPointerTy(DAG.getDataLayout());
  const TargetFrameLowering *TFL = Subtarget.getFrameLowering();

  // TODO: Support tail call optimization.
  IsTailCall = false;

  // Analyze the operands of the call, assigning locations to each operand.
  SmallVector<CCValAssign, 16> ArgLocs;
  CCState CCInfo(CallConv, IsVarArg, MF, ArgLocs, *DAG.getContext());

  CCAssignFn *CC = CCAssignFnForCall(CallConv, IsVarArg);

  CCInfo.AnalyzeCallOperands(Outs, CC);

  // Get a count of how many bytes are to be pushed on the stack.
  unsigned NumBytes = CCInfo.getStackSize();

```
- **EN**: Implements logic around `getMachineFunction`, `getPointerTy`, `getFrameLowering`, `CCInfo`, ...; this block handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getMachineFunction`, `getPointerTy`, `getFrameLowering`, `CCInfo`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 616-639
```cpp
  Align StackAlignment = TFL->getStackAlign();
  unsigned NextStackOffset = alignTo(NumBytes, StackAlignment);

  Chain = DAG.getCALLSEQ_START(Chain, NextStackOffset, 0, DL);

  // Copy argument values to their designated locations.
  std::deque<std::pair<unsigned, SDValue>> RegsToPass;
  SmallVector<SDValue, 8> MemOpChains;
  SDValue StackPtr;
  for (unsigned I = 0, E = ArgLocs.size(); I != E; ++I) {
    CCValAssign &VA = ArgLocs[I];
    SDValue ArgValue = OutVals[I];
    ISD::ArgFlagsTy Flags = Outs[I].Flags;

    if (VA.isRegLoc())
      // Queue up the argument copies and emit them at the end.
      RegsToPass.push_back(std::make_pair(VA.getLocReg(), ArgValue));
    else if (Flags.isByVal()) {
      assert(VA.isMemLoc());
      assert(Flags.getByValSize() &&
             "ByVal args of size 0 should have been ignored by front-end.");
      assert(!IsTailCall &&
             "Do not tail-call optimize if there is a byval argument.");

```
- **EN**: Implements logic around `getStackAlign`, `alignTo`, `getCALLSEQ_START`, `push_back`, ...; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getStackAlign`, `alignTo`, `getCALLSEQ_START`, `push_back`, ... 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 640-662
```cpp
      if (!StackPtr.getNode())
        StackPtr = DAG.getCopyFromReg(Chain, DL, Xtensa::SP, PtrVT);
      unsigned Offset = VA.getLocMemOffset();
      SDValue Address = DAG.getNode(ISD::ADD, DL, PtrVT, StackPtr,
                                    DAG.getIntPtrConstant(Offset, DL));
      SDValue SizeNode = DAG.getConstant(Flags.getByValSize(), DL, MVT::i32);
      SDValue Memcpy = DAG.getMemcpy(
          Chain, DL, Address, ArgValue, SizeNode, Flags.getNonZeroByValAlign(),
          /*isVolatile=*/false, /*AlwaysInline=*/false,
          /*CI=*/nullptr, std::nullopt, MachinePointerInfo(),
          MachinePointerInfo());
      MemOpChains.push_back(Memcpy);
    } else {
      assert(VA.isMemLoc() && "Argument not register or memory");

      // Work out the address of the stack slot.  Unpromoted ints and
      // floats are passed as right-justified 8-byte values.
      if (!StackPtr.getNode())
        StackPtr = DAG.getCopyFromReg(Chain, DL, Xtensa::SP, PtrVT);
      unsigned Offset = VA.getLocMemOffset();
      SDValue Address = DAG.getNode(ISD::ADD, DL, PtrVT, StackPtr,
                                    DAG.getIntPtrConstant(Offset, DL));

```
- **EN**: Implements logic around `getCopyFromReg`, `getLocMemOffset`, `getNode`, `getIntPtrConstant`, ...; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getCopyFromReg`, `getLocMemOffset`, `getNode`, `getIntPtrConstant`, ... 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 663-684
```cpp
      // Emit the store.
      MemOpChains.push_back(
          DAG.getStore(Chain, DL, ArgValue, Address, MachinePointerInfo()));
    }
  }

  // Join the stores, which are independent of one another.
  if (!MemOpChains.empty())
    Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, MemOpChains);

  // Build a sequence of copy-to-reg nodes, chained and glued together.
  SDValue Glue;
  for (unsigned I = 0, E = RegsToPass.size(); I != E; ++I) {
    unsigned Reg = RegsToPass[I].first;
    if (Subtarget.isWindowedABI())
      Reg = toCallerWindow(Reg);
    Chain = DAG.getCopyToReg(Chain, DL, Reg, RegsToPass[I].second, Glue);
    Glue = Chain.getValue(1);
  }
  std::string name;
  unsigned char TF = 0;

```
- **EN**: Implements logic around `push_back`, `getStore`, `getNode`, `toCallerWindow`, ...; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `push_back`, `getStore`, `getNode`, `toCallerWindow`, ... 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 685-705
```cpp
  // Accept direct calls by converting symbolic call addresses to the
  // associated Target* opcodes.
  if (ExternalSymbolSDNode *E = dyn_cast<ExternalSymbolSDNode>(Callee)) {
    name = E->getSymbol();
    TF = E->getTargetFlags();
    if (isPositionIndependent()) {
      report_fatal_error("PIC relocations is not supported");
    } else
      Callee = DAG.getTargetExternalSymbol(E->getSymbol(), PtrVT, TF);
  } else if (GlobalAddressSDNode *G = dyn_cast<GlobalAddressSDNode>(Callee)) {
    const GlobalValue *GV = G->getGlobal();
    name = GV->getName().str();
  }

  if ((!name.empty()) && isLongCall(name.c_str())) {
    // Create a constant pool entry for the callee address
    XtensaCP::XtensaCPModifier Modifier = XtensaCP::no_modifier;
    XtensaMachineFunctionInfo *XtensaFI =
        MF.getInfo<XtensaMachineFunctionInfo>();
    unsigned LabelId = XtensaFI->createCPLabelId();

```
- **EN**: Implements logic around `getSymbol`, `getTargetFlags`, `report_fatal_error`, `getTargetExternalSymbol`, ...; this block applies conditional target rules; maps fixups or relocations; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getSymbol`, `getTargetFlags`, `report_fatal_error`, `getTargetExternalSymbol`, ... 实现具体逻辑；这一段应用条件化的目标规则，映射 fixup 或重定位，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 706-721
```cpp
    XtensaConstantPoolValue *CPV = XtensaConstantPoolSymbol::Create(
        *DAG.getContext(), name.c_str(), LabelId, false, Modifier);

    // Get the address of the callee into a register
    SDValue CPAddr = DAG.getTargetConstantPool(CPV, PtrVT, Align(4), 0, TF);
    SDValue CPWrap = getAddrPCRel(CPAddr, DAG);
    Callee = DAG.getLoad(
        PtrVT, DL, DAG.getEntryNode(), CPWrap,
        MachinePointerInfo::getConstantPool(DAG.getMachineFunction()));
  }

  // The first call operand is the chain and the second is the target address.
  SmallVector<SDValue, 8> Ops;
  Ops.push_back(Chain);
  Ops.push_back(Callee);

```
- **EN**: Implements logic around `Create`, `getContext`, `getTargetConstantPool`, `getAddrPCRel`, ...; this block handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `Create`, `getContext`, `getTargetConstantPool`, `getAddrPCRel`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 722-740
```cpp
  // Add a register mask operand representing the call-preserved registers.
  const TargetRegisterInfo *TRI = Subtarget.getRegisterInfo();
  const uint32_t *Mask = TRI->getCallPreservedMask(MF, CallConv);
  assert(Mask && "Missing call preserved mask for calling convention");
  Ops.push_back(DAG.getRegisterMask(Mask));

  // Add argument registers to the end of the list so that they are
  // known live into the call.
  for (unsigned I = 0, E = RegsToPass.size(); I != E; ++I) {
    unsigned Reg = RegsToPass[I].first;
    if (Subtarget.isWindowedABI())
      Reg = toCallerWindow(Reg);
    Ops.push_back(DAG.getRegister(Reg, RegsToPass[I].second.getValueType()));
  }

  // Glue the call to the argument copies, if any.
  if (Glue.getNode())
    Ops.push_back(Glue);

```
- **EN**: Implements logic around `getRegisterInfo`, `getCallPreservedMask`, `assert`, `push_back`, ...; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getRegisterInfo`, `getCallPreservedMask`, `assert`, `push_back`, ... 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 741-757
```cpp
  SDVTList NodeTys = DAG.getVTList(MVT::Other, MVT::Glue);
  Chain = DAG.getNode(Subtarget.isWindowedABI() ? XtensaISD::CALLW8
                                                : XtensaISD::CALL,
                      DL, NodeTys, Ops);
  Glue = Chain.getValue(1);

  // Mark the end of the call, which is glued to the call itself.
  Chain = DAG.getCALLSEQ_END(Chain, DAG.getConstant(NumBytes, DL, PtrVT, true),
                             DAG.getConstant(0, DL, PtrVT, true), Glue, DL);
  Glue = Chain.getValue(1);

  // Assign locations to each value returned by this call.
  SmallVector<CCValAssign, 16> RetLocs;
  CCState RetCCInfo(CallConv, IsVarArg, MF, RetLocs, *DAG.getContext());
  RetCCInfo.AnalyzeCallResult(Ins, Subtarget.isWindowedABI() ? RetCCW8_Xtensa
                                                             : RetCC_Xtensa);

```
- **EN**: Implements logic around `getVTList`, `getNode`, `getValue`, `getCALLSEQ_END`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getVTList`, `getNode`, `getValue`, `getCALLSEQ_END`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 758-781
```cpp
  // Copy all of the result registers out of their specified physreg.
  for (unsigned I = 0, E = RetLocs.size(); I != E; ++I) {
    CCValAssign &VA = RetLocs[I];

    // Copy the value out, gluing the copy to the end of the call sequence.
    unsigned Reg = VA.getLocReg();
    SDValue RetValue = DAG.getCopyFromReg(Chain, DL, Reg, VA.getLocVT(), Glue);
    Chain = RetValue.getValue(1);
    Glue = RetValue.getValue(2);

    InVals.push_back(RetValue);
  }
  return Chain;
}

bool XtensaTargetLowering::CanLowerReturn(
    CallingConv::ID CallConv, MachineFunction &MF, bool IsVarArg,
    const SmallVectorImpl<ISD::OutputArg> &Outs, LLVMContext &Context,
    const Type *RetTy) const {
  SmallVector<CCValAssign, 16> RVLocs;
  CCState CCInfo(CallConv, IsVarArg, MF, RVLocs, Context);
  return CCInfo.CheckReturn(Outs, RetCC_Xtensa);
}

```
- **EN**: Implements logic around `getLocReg`, `getCopyFromReg`, `getValue`, `push_back`, ...; this block returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getLocReg`, `getCopyFromReg`, `getValue`, `push_back`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 782-801
```cpp
SDValue
XtensaTargetLowering::LowerReturn(SDValue Chain, CallingConv::ID CallConv,
                                  bool IsVarArg,
                                  const SmallVectorImpl<ISD::OutputArg> &Outs,
                                  const SmallVectorImpl<SDValue> &OutVals,
                                  const SDLoc &DL, SelectionDAG &DAG) const {
  MachineFunction &MF = DAG.getMachineFunction();

  // Assign locations to each returned value.
  SmallVector<CCValAssign, 16> RetLocs;
  CCState RetCCInfo(CallConv, IsVarArg, MF, RetLocs, *DAG.getContext());
  RetCCInfo.AnalyzeReturn(Outs, RetCC_Xtensa);

  SDValue Glue;
  // Quick exit for void returns
  if (RetLocs.empty())
    return DAG.getNode(Subtarget.isWindowedABI() ? XtensaISD::RETW
                                                 : XtensaISD::RET,
                       DL, MVT::Other, Chain);

```
- **EN**: Implements logic around `LowerReturn`, `getMachineFunction`, `RetCCInfo`, `AnalyzeReturn`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LowerReturn`, `getMachineFunction`, `RetCCInfo`, `AnalyzeReturn`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 802-818
```cpp
  // Copy the result values into the output registers.
  SmallVector<SDValue, 4> RetOps;
  RetOps.push_back(Chain);
  for (unsigned I = 0, E = RetLocs.size(); I != E; ++I) {
    CCValAssign &VA = RetLocs[I];
    SDValue RetValue = OutVals[I];

    // Make the return register live on exit.
    assert(VA.isRegLoc() && "Can only return in registers!");

    // Chain and glue the copies together.
    unsigned Register = VA.getLocReg();
    Chain = DAG.getCopyToReg(Chain, DL, Register, RetValue, Glue);
    Glue = Chain.getValue(1);
    RetOps.push_back(DAG.getRegister(Register, VA.getLocVT()));
  }

```
- **EN**: Implements logic around `push_back`, `assert`, `getLocReg`, `getCopyToReg`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `push_back`, `assert`, `getLocReg`, `getCopyToReg`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 819-850
```cpp
  // Update chain and glue.
  RetOps[0] = Chain;
  if (Glue.getNode())
    RetOps.push_back(Glue);

  return DAG.getNode(Subtarget.isWindowedABI() ? XtensaISD::RETW
                                               : XtensaISD::RET,
                     DL, MVT::Other, RetOps);
}

static unsigned getBranchOpcode(ISD::CondCode Cond) {
  switch (Cond) {
  case ISD::SETEQ:
    return Xtensa::BEQ;
  case ISD::SETNE:
    return Xtensa::BNE;
  case ISD::SETLT:
    return Xtensa::BLT;
  case ISD::SETLE:
    return Xtensa::BGE;
  case ISD::SETGT:
    return Xtensa::BLT;
  case ISD::SETGE:
    return Xtensa::BGE;
  case ISD::SETULT:
    return Xtensa::BLTU;
  case ISD::SETULE:
    return Xtensa::BGEU;
  case ISD::SETUGT:
    return Xtensa::BLTU;
  case ISD::SETUGE:
    return Xtensa::BGEU;
```
- **EN**: Implements logic around `push_back`, `getNode`, `getBranchOpcode`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `push_back`, `getNode`, `getBranchOpcode` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 851-882
```cpp
  default:
    llvm_unreachable("Unknown branch kind");
  }
}

static std::pair<unsigned, unsigned> getFPBranchKind(ISD::CondCode Cond) {
  switch (Cond) {
  case ISD::SETUNE:
    return std::make_pair(Xtensa::BF, Xtensa::OEQ_S);
  case ISD::SETUO:
    return std::make_pair(Xtensa::BT, Xtensa::UN_S);
  case ISD::SETO:
    return std::make_pair(Xtensa::BF, Xtensa::UN_S);
  case ISD::SETUEQ:
    return std::make_pair(Xtensa::BT, Xtensa::UEQ_S);
  case ISD::SETULE:
    return std::make_pair(Xtensa::BT, Xtensa::ULE_S);
  case ISD::SETULT:
    return std::make_pair(Xtensa::BT, Xtensa::ULT_S);
  case ISD::SETEQ:
  case ISD::SETOEQ:
    return std::make_pair(Xtensa::BT, Xtensa::OEQ_S);
  case ISD::SETNE:
    return std::make_pair(Xtensa::BF, Xtensa::OEQ_S);
  case ISD::SETLE:
  case ISD::SETOLE:
    return std::make_pair(Xtensa::BT, Xtensa::OLE_S);
  case ISD::SETLT:
  case ISD::SETOLT:
    return std::make_pair(Xtensa::BT, Xtensa::OLT_S);
  case ISD::SETGE:
    return std::make_pair(Xtensa::BF, Xtensa::OLT_S);
```
- **EN**: Implements logic around `llvm_unreachable`, `getFPBranchKind`, `make_pair`; this block uses `switch`-based dispatch; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `llvm_unreachable`, `getFPBranchKind`, `make_pair` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 883-899
```cpp
  case ISD::SETGT:
    return std::make_pair(Xtensa::BF, Xtensa::OLE_S);
  case ISD::SETOGT:
    return std::make_pair(Xtensa::BF, Xtensa::ULE_S);
  case ISD::SETOGE:
    return std::make_pair(Xtensa::BF, Xtensa::ULT_S);
  case ISD::SETONE:
    return std::make_pair(Xtensa::BF, Xtensa::UEQ_S);
  case ISD::SETUGT:
    return std::make_pair(Xtensa::BF, Xtensa::OLE_S);
  case ISD::SETUGE:
    return std::make_pair(Xtensa::BF, Xtensa::OLT_S);
  default:
    llvm_unreachable("Invalid condition!");
  }
}

```
- **EN**: Implements logic around `make_pair`, `llvm_unreachable`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `make_pair`, `llvm_unreachable` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 900-929
```cpp
SDValue XtensaTargetLowering::LowerSELECT_CC(SDValue Op,
                                             SelectionDAG &DAG) const {
  SDLoc DL(Op);
  EVT Ty = Op.getValueType();
  SDValue LHS = Op.getOperand(0);
  SDValue RHS = Op.getOperand(1);
  SDValue TrueValue = Op.getOperand(2);
  SDValue FalseValue = Op.getOperand(3);
  ISD::CondCode CC = cast<CondCodeSDNode>(Op->getOperand(4))->get();

  if (LHS.getValueType() == MVT::i32) {
    unsigned BrOpcode = getBranchOpcode(CC);
    SDValue TargetCC = DAG.getConstant(BrOpcode, DL, MVT::i32);

    SDValue Res = DAG.getNode(XtensaISD::SELECT_CC, DL, Ty, LHS, RHS, TrueValue,
                              FalseValue, TargetCC, Op->getFlags());
    return Res;
  }
  assert(LHS.getValueType() == MVT::f32 &&
         "We expect MVT::f32 type of the LHS Operand in SELECT_CC");
  unsigned BrOpcode;
  unsigned CmpOpCode;
  std::tie(BrOpcode, CmpOpCode) = getFPBranchKind(CC);
  SDValue TargetCC = DAG.getConstant(CmpOpCode, DL, MVT::i32);
  SDValue TargetBC = DAG.getConstant(BrOpcode, DL, MVT::i32);
  return DAG.getNode(XtensaISD::SELECT_CC_FP, DL, Ty,
                     {LHS, RHS, TrueValue, FalseValue, TargetCC, TargetBC},
                     Op->getFlags());
}

```
- **EN**: Implements logic around `LowerSELECT_CC`, `DL`, `getValueType`, `getOperand`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerSELECT_CC`, `DL`, `getValueType`, `getOperand`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 930-950
```cpp
SDValue XtensaTargetLowering::LowerRETURNADDR(SDValue Op,
                                              SelectionDAG &DAG) const {
  // This nodes represent llvm.returnaddress on the DAG.
  // It takes one operand, the index of the return address to return.
  // An index of zero corresponds to the current function's return address.
  // An index of one to the parent's return address, and so on.
  // Depths > 0 not supported yet!
  if (Op.getConstantOperandVal(0) != 0)
    return SDValue();

  MachineFunction &MF = DAG.getMachineFunction();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  EVT VT = Op.getValueType();
  MFI.setReturnAddressIsTaken(true);

  // Return RA, which contains the return address. Mark it an implicit
  // live-in.
  Register RA = MF.addLiveIn(Xtensa::A0, getRegClassFor(MVT::i32));
  return DAG.getCopyFromReg(DAG.getEntryNode(), SDLoc(Op), RA, VT);
}

```
- **EN**: Implements logic around `LowerRETURNADDR`, `SDValue`, `getMachineFunction`, `getFrameInfo`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LowerRETURNADDR`, `SDValue`, `getMachineFunction`, `getFrameInfo`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 951-975
```cpp
SDValue XtensaTargetLowering::LowerImmediate(SDValue Op,
                                             SelectionDAG &DAG) const {
  const ConstantSDNode *CN = cast<ConstantSDNode>(Op);
  SDLoc DL(CN);
  APInt APVal = CN->getAPIntValue();
  int64_t Value = APVal.getSExtValue();
  if (Op.getValueType() == MVT::i32) {
    // Check if use node maybe lowered to the MOVI instruction
    if (Value > -2048 && Value <= 2047)
      return Op;
    // Check if use node maybe lowered to the ADDMI instruction
    SDNode &OpNode = *Op.getNode();
    if ((OpNode.hasOneUse() && OpNode.user_begin()->getOpcode() == ISD::ADD) &&
        isShiftedInt<8, 8>(Value))
      return Op;
    Type *Ty = Type::getInt32Ty(*DAG.getContext());
    Constant *CV = ConstantInt::getSigned(Ty, Value);
    SDValue CP = DAG.getConstantPool(CV, MVT::i32);
    SDValue Res =
        DAG.getLoad(MVT::i32, DL, DAG.getEntryNode(), CP, MachinePointerInfo());
    return Res;
  }
  return Op;
}

```
- **EN**: Implements logic around `LowerImmediate`, `cast<ConstantSDNode>`, `DL`, `getAPIntValue`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerImmediate`, `cast<ConstantSDNode>`, `DL`, `getAPIntValue`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 976-997
```cpp
SDValue XtensaTargetLowering::LowerGlobalAddress(SDValue Op,
                                                 SelectionDAG &DAG) const {
  const GlobalAddressSDNode *G = cast<GlobalAddressSDNode>(Op);
  SDLoc DL(Op);
  auto PtrVT = Op.getValueType();
  const GlobalValue *GV = G->getGlobal();

  SDValue CPAddr = DAG.getTargetConstantPool(GV, PtrVT, Align(4));
  SDValue CPWrap = getAddrPCRel(CPAddr, DAG);
  SDValue Res = DAG.getLoad(
      PtrVT, DL, DAG.getEntryNode(), CPWrap,
      MachinePointerInfo::getConstantPool(DAG.getMachineFunction()));
  return Res;
}

SDValue XtensaTargetLowering::LowerGlobalTLSAddress(SDValue Op,
                                                    SelectionDAG &DAG) const {
  const GlobalAddressSDNode *G = cast<GlobalAddressSDNode>(Op);
  SDLoc DL(Op);
  EVT PtrVT = Op.getValueType();
  const GlobalValue *GV = G->getGlobal();

```
- **EN**: Implements logic around `LowerGlobalAddress`, `cast<GlobalAddressSDNode>`, `DL`, `getValueType`, ...; this block returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LowerGlobalAddress`, `cast<GlobalAddressSDNode>`, `DL`, `getValueType`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 998-1016
```cpp
  if (DAG.getTarget().useEmulatedTLS())
    return LowerToTLSEmulatedModel(G, DAG);

  TLSModel::Model model = getTargetMachine().getTLSModel(GV);

  if (!Subtarget.hasTHREADPTR()) {
    DAG.getContext()->diagnose(DiagnosticInfoUnsupported(
        DAG.getMachineFunction().getFunction(), "only emulated TLS supported",
        DL.getDebugLoc()));
    return DAG.getPOISON(Op->getValueType(0));
  }

  if (model == TLSModel::LocalExec || model == TLSModel::InitialExec) {
    bool Priv = GV->isPrivateLinkage(GV->getLinkage());
    MachineFunction &MF = DAG.getMachineFunction();
    XtensaMachineFunctionInfo *XtensaFI =
        MF.getInfo<XtensaMachineFunctionInfo>();
    unsigned LabelId = XtensaFI->createCPLabelId();

```
- **EN**: Implements logic around `LowerToTLSEmulatedModel`, `getTargetMachine`, `getContext`, `getMachineFunction`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LowerToTLSEmulatedModel`, `getTargetMachine`, `getContext`, `getMachineFunction`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 1017-1032
```cpp
    // Create a constant pool entry for the callee address
    XtensaConstantPoolValue *CPV = XtensaConstantPoolSymbol::Create(
        *DAG.getContext(), GV->getName().str().c_str(), LabelId, Priv,
        XtensaCP::TPOFF);

    // Get the address of the callee into a register
    SDValue CPAddr = DAG.getTargetConstantPool(CPV, PtrVT, Align(4));
    SDValue CPWrap = getAddrPCRel(CPAddr, DAG);
    SDValue Addr = DAG.getLoad(
        PtrVT, DL, DAG.getEntryNode(), CPWrap,
        MachinePointerInfo::getConstantPool(DAG.getMachineFunction()));

    SDValue TPRegister = DAG.getRegister(Xtensa::THREADPTR, MVT::i32);
    SDValue ThreadPointer =
        DAG.getNode(XtensaISD::RUR, DL, MVT::i32, TPRegister);

```
- **EN**: Implements logic around `Create`, `getContext`, `getTargetConstantPool`, `getAddrPCRel`, ...; this block handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `Create`, `getContext`, `getTargetConstantPool`, `getAddrPCRel`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 1033-1052
```cpp
    return DAG.getNode(ISD::ADD, DL, PtrVT, ThreadPointer, Addr);
  }

  DAG.getContext()->diagnose(DiagnosticInfoUnsupported(
      DAG.getMachineFunction().getFunction(),
      "only local-exec and initial-exec TLS mode supported", DL.getDebugLoc()));

  return DAG.getPOISON(Op->getValueType(0));
}

SDValue XtensaTargetLowering::LowerBlockAddress(SDValue Op,
                                                SelectionDAG &DAG) const {
  BlockAddressSDNode *Node = cast<BlockAddressSDNode>(Op);
  SDLoc DL(Op);
  const BlockAddress *BA = Node->getBlockAddress();
  EVT PtrVT = Op.getValueType();
  MachineFunction &MF = DAG.getMachineFunction();
  XtensaMachineFunctionInfo *XtensaFI = MF.getInfo<XtensaMachineFunctionInfo>();
  unsigned LabelId = XtensaFI->createCPLabelId();

```
- **EN**: Implements logic around `getNode`, `getContext`, `getMachineFunction`, `getDebugLoc`, ...; this block returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getNode`, `getContext`, `getMachineFunction`, `getDebugLoc`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 1053-1075
```cpp
  XtensaConstantPoolValue *CPV =
      XtensaConstantPoolConstant::Create(BA, LabelId, XtensaCP::CPBlockAddress);
  SDValue CPAddr = DAG.getTargetConstantPool(CPV, PtrVT, Align(4));
  SDValue CPWrap = getAddrPCRel(CPAddr, DAG);
  SDValue Res = DAG.getLoad(
      PtrVT, DL, DAG.getEntryNode(), CPWrap,
      MachinePointerInfo::getConstantPool(DAG.getMachineFunction()));
  return Res;
}

SDValue XtensaTargetLowering::LowerBR_JT(SDValue Op, SelectionDAG &DAG) const {
  SDValue Chain = Op.getOperand(0);
  SDValue Table = Op.getOperand(1);
  SDValue Index = Op.getOperand(2);
  SDLoc DL(Op);
  JumpTableSDNode *JT = cast<JumpTableSDNode>(Table);
  MachineFunction &MF = DAG.getMachineFunction();
  const MachineJumpTableInfo *MJTI = MF.getJumpTableInfo();
  SDValue TargetJT = DAG.getTargetJumpTable(JT->getIndex(), MVT::i32);
  const DataLayout &TD = DAG.getDataLayout();
  EVT PtrVT = Table.getValueType();
  unsigned EntrySize = MJTI->getEntrySize(TD);

```
- **EN**: Implements logic around `Create`, `getTargetConstantPool`, `getAddrPCRel`, `getLoad`, ...; this block returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `Create`, `getTargetConstantPool`, `getAddrPCRel`, `getLoad`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 1076-1096
```cpp
  assert((MJTI->getEntrySize(TD) == 4) && "Unsupported jump-table entry size");

  Index = DAG.getNode(
      ISD::SHL, DL, Index.getValueType(), Index,
      DAG.getConstant(Log2_32(EntrySize), DL, Index.getValueType()));

  SDValue Addr = DAG.getNode(ISD::ADD, DL, Index.getValueType(), Index, Table);
  SDValue LD =
      DAG.getLoad(PtrVT, DL, Chain, Addr,
                  MachinePointerInfo::getJumpTable(DAG.getMachineFunction()));

  return DAG.getNode(XtensaISD::BR_JT, DL, MVT::Other, LD.getValue(1), LD,
                     TargetJT);
}

SDValue XtensaTargetLowering::LowerJumpTable(SDValue Op,
                                             SelectionDAG &DAG) const {
  JumpTableSDNode *JT = cast<JumpTableSDNode>(Op);
  EVT PtrVT = Op.getValueType();
  SDLoc DL(Op);

```
- **EN**: Implements logic around `assert`, `getNode`, `getValueType`, `getConstant`, ...; this block returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `assert`, `getNode`, `getValueType`, `getConstant`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 1097-1116
```cpp
  // Create a constant pool entry for the jumptable address
  XtensaConstantPoolValue *CPV =
      XtensaConstantPoolJumpTable::Create(*DAG.getContext(), JT->getIndex());

  // Get the address of the jumptable into a register
  SDValue CPAddr = DAG.getTargetConstantPool(CPV, PtrVT, Align(4));

  SDValue Res = DAG.getLoad(
      PtrVT, DL, DAG.getEntryNode(), getAddrPCRel(CPAddr, DAG),
      MachinePointerInfo::getConstantPool(DAG.getMachineFunction()));
  return Res;
}

SDValue XtensaTargetLowering::getAddrPCRel(SDValue Op,
                                           SelectionDAG &DAG) const {
  SDLoc DL(Op);
  EVT Ty = Op.getValueType();
  return DAG.getNode(XtensaISD::PCREL_WRAPPER, DL, Ty, Op);
}

```
- **EN**: Implements logic around `Create`, `getTargetConstantPool`, `getLoad`, `getEntryNode`, ...; this block returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `Create`, `getTargetConstantPool`, `getLoad`, `getEntryNode`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 1117-1132
```cpp
SDValue XtensaTargetLowering::LowerConstantPool(SDValue Op,
                                                SelectionDAG &DAG) const {
  EVT PtrVT = Op.getValueType();
  ConstantPoolSDNode *CP = cast<ConstantPoolSDNode>(Op);
  SDValue Result;

  if (!CP->isMachineConstantPoolEntry()) {
    Result = DAG.getTargetConstantPool(CP->getConstVal(), PtrVT, CP->getAlign(),
                                       CP->getOffset());
  } else {
    report_fatal_error("This constantpool type is not supported yet");
  }

  return getAddrPCRel(Result, DAG);
}

```
- **EN**: Implements logic around `LowerConstantPool`, `getValueType`, `cast<ConstantPoolSDNode>`, `getTargetConstantPool`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerConstantPool`, `getValueType`, `cast<ConstantPoolSDNode>`, `getTargetConstantPool`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1133-1150
```cpp
SDValue XtensaTargetLowering::LowerSTACKSAVE(SDValue Op,
                                             SelectionDAG &DAG) const {
  return DAG.getCopyFromReg(Op.getOperand(0), SDLoc(Op), Xtensa::SP,
                            Op.getValueType());
}

SDValue XtensaTargetLowering::LowerSTACKRESTORE(SDValue Op,
                                                SelectionDAG &DAG) const {
  SDValue Chain = Op.getOperand(0);
  SDValue NewSP = Op.getOperand(1);

  if (Subtarget.isWindowedABI()) {
    return DAG.getNode(XtensaISD::MOVSP, SDLoc(Op), MVT::Other, Chain, NewSP);
  }

  return DAG.getCopyToReg(Chain, SDLoc(Op), Xtensa::SP, NewSP);
}

```
- **EN**: Implements logic around `LowerSTACKSAVE`, `getCopyFromReg`, `getValueType`, `LowerSTACKRESTORE`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerSTACKSAVE`, `getCopyFromReg`, `getValueType`, `LowerSTACKRESTORE`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1151-1166
```cpp
SDValue XtensaTargetLowering::LowerFRAMEADDR(SDValue Op,
                                             SelectionDAG &DAG) const {
  // This nodes represent llvm.frameaddress on the DAG.
  // It takes one operand, the index of the frame address to return.
  // An index of zero corresponds to the current function's frame address.
  // An index of one to the parent's frame address, and so on.
  // Depths > 0 not supported yet!
  if (Op.getConstantOperandVal(0) != 0)
    return SDValue();

  MachineFunction &MF = DAG.getMachineFunction();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  MFI.setFrameAddressIsTaken(true);
  EVT VT = Op.getValueType();
  SDLoc DL(Op);

```
- **EN**: Implements logic around `LowerFRAMEADDR`, `SDValue`, `getMachineFunction`, `getFrameInfo`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LowerFRAMEADDR`, `SDValue`, `getMachineFunction`, `getFrameInfo`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 1167-1185
```cpp
  MCRegister FrameRegister = Subtarget.getRegisterInfo()->getFrameRegister(MF);
  SDValue FrameAddr =
      DAG.getCopyFromReg(DAG.getEntryNode(), DL, FrameRegister, VT);
  return FrameAddr;
}

SDValue XtensaTargetLowering::LowerDYNAMIC_STACKALLOC(SDValue Op,
                                                      SelectionDAG &DAG) const {
  SDValue Chain = Op.getOperand(0); // Legalize the chain.
  SDValue Size = Op.getOperand(1);  // Legalize the size.
  EVT VT = Size->getValueType(0);
  SDLoc DL(Op);

  // Round up Size to 32
  SDValue SizeTmp =
      DAG.getNode(ISD::ADD, DL, VT, Size, DAG.getConstant(31, DL, MVT::i32));
  SDValue SizeRoundUp = DAG.getNode(ISD::AND, DL, VT, SizeTmp,
                                    DAG.getSignedConstant(~31, DL, MVT::i32));

```
- **EN**: Implements logic around `getRegisterInfo`, `getCopyFromReg`, `LowerDYNAMIC_STACKALLOC`, `getOperand`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getRegisterInfo`, `getCopyFromReg`, `LowerDYNAMIC_STACKALLOC`, `getOperand`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1186-1202
```cpp
  MCRegister SPReg = Xtensa::SP;
  SDValue SP = DAG.getCopyFromReg(Chain, DL, SPReg, VT);
  SDValue NewSP = DAG.getNode(ISD::SUB, DL, VT, SP, SizeRoundUp); // Value
  if (Subtarget.isWindowedABI()) {
    Chain = DAG.getNode(XtensaISD::MOVSP, SDLoc(Op), MVT::Other, SP.getValue(1),
                        NewSP);
  } else {
    Chain = DAG.getCopyToReg(SP.getValue(1), DL, SPReg, NewSP); // Output chain
  }

  SDValue NewVal = DAG.getCopyFromReg(Chain, DL, SPReg, MVT::i32);
  Chain = NewVal.getValue(1);

  SDValue Ops[2] = {NewVal, Chain};
  return DAG.getMergeValues(Ops, DL);
}

```
- **EN**: Implements logic around `getCopyFromReg`, `getNode`, `getCopyToReg`, `getValue`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getCopyFromReg`, `getNode`, `getCopyToReg`, `getValue`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1203-1222
```cpp
SDValue XtensaTargetLowering::LowerVASTART(SDValue Op,
                                           SelectionDAG &DAG) const {
  MachineFunction &MF = DAG.getMachineFunction();
  XtensaMachineFunctionInfo *XtensaFI = MF.getInfo<XtensaMachineFunctionInfo>();
  SDValue Chain = Op.getOperand(0);
  SDValue Addr = Op.getOperand(1);
  EVT PtrVT = Addr.getValueType();
  SDLoc DL(Op);

  // Struct va_list_tag
  // int32 *va_stk - points to the arguments passed in memory
  // int32 *va_reg - points to the registers with arguments saved in memory
  // int32 va_ndx  - offset from va_stk or va_reg pointers which points to  the
  // next variable argument

  SDValue VAIndex;
  SDValue StackOffsetFI =
      DAG.getFrameIndex(XtensaFI->getVarArgsOnStackFrameIndex(), PtrVT);
  unsigned ArgWords = XtensaFI->getVarArgsFirstGPR() - 2;

```
- **EN**: Implements logic around `LowerVASTART`, `getMachineFunction`, `getInfo<XtensaMachineFunctionInfo>`, `getOperand`, ...; this block handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LowerVASTART`, `getMachineFunction`, `getInfo<XtensaMachineFunctionInfo>`, `getOperand`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 1223-1241
```cpp
  // If first variable argument passed in registers (maximum words in registers
  // is 6) then set va_ndx to the position of this argument in registers area
  // stored in memory (va_reg pointer). Otherwise va_ndx should point to the
  // position of the first variable argument on stack (va_stk pointer).
  if (ArgWords < 6) {
    VAIndex = DAG.getConstant(ArgWords * 4, DL, MVT::i32);
  } else {
    VAIndex = DAG.getConstant(32, DL, MVT::i32);
  }

  SDValue FrameIndex =
      DAG.getFrameIndex(XtensaFI->getVarArgsInRegsFrameIndex(), PtrVT);
  uint64_t FrameOffset = PtrVT.getStoreSize();
  const Value *SV = cast<SrcValueSDNode>(Op.getOperand(2))->getValue();

  // Store pointer to arguments given on stack (va_stk)
  SDValue StackPtr = DAG.getNode(ISD::SUB, DL, PtrVT, StackOffsetFI,
                                 DAG.getConstant(32, DL, PtrVT));

```
- **EN**: Implements logic around `getConstant`, `getFrameIndex`, `getStoreSize`, `cast<SrcValueSDNode>`, ...; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getConstant`, `getFrameIndex`, `getStoreSize`, `cast<SrcValueSDNode>`, ... 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 1242-1259
```cpp
  SDValue StoreStackPtr =
      DAG.getStore(Chain, DL, StackPtr, Addr, MachinePointerInfo(SV));

  uint64_t NextOffset = FrameOffset;
  SDValue NextPtr =
      DAG.getObjectPtrOffset(DL, Addr, TypeSize::getFixed(NextOffset));

  // Store pointer to arguments given on registers (va_reg)
  SDValue StoreRegPtr = DAG.getStore(StoreStackPtr, DL, FrameIndex, NextPtr,
                                     MachinePointerInfo(SV, NextOffset));
  NextOffset += FrameOffset;
  NextPtr = DAG.getObjectPtrOffset(DL, Addr, TypeSize::getFixed(NextOffset));

  // Store third word : position in bytes of the first VA argument (va_ndx)
  return DAG.getStore(StoreRegPtr, DL, VAIndex, NextPtr,
                      MachinePointerInfo(SV, NextOffset));
}

```
- **EN**: Implements logic around `getStore`, `getObjectPtrOffset`, `MachinePointerInfo`; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getStore`, `getObjectPtrOffset`, `MachinePointerInfo` 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1260-1276
```cpp
SDValue XtensaTargetLowering::LowerVACOPY(SDValue Op, SelectionDAG &DAG) const {
  // Size of the va_list_tag structure
  constexpr unsigned VAListSize = 3 * 4;
  SDValue Chain = Op.getOperand(0);
  SDValue DstPtr = Op.getOperand(1);
  SDValue SrcPtr = Op.getOperand(2);
  const Value *DstSV = cast<SrcValueSDNode>(Op.getOperand(3))->getValue();
  const Value *SrcSV = cast<SrcValueSDNode>(Op.getOperand(4))->getValue();
  SDLoc DL(Op);

  return DAG.getMemcpy(Chain, DL, DstPtr, SrcPtr,
                       DAG.getConstant(VAListSize, SDLoc(Op), MVT::i32),
                       Align(4), /*isVolatile*/ false, /*AlwaysInline*/ true,
                       /*CI=*/nullptr, std::nullopt, MachinePointerInfo(DstSV),
                       MachinePointerInfo(SrcSV));
}

```
- **EN**: Implements logic around `LowerVACOPY`, `getOperand`, `cast<SrcValueSDNode>`, `DL`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerVACOPY`, `getOperand`, `cast<SrcValueSDNode>`, `DL`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1277-1296
```cpp
SDValue XtensaTargetLowering::LowerVAARG(SDValue Op, SelectionDAG &DAG) const {
  SDNode *Node = Op.getNode();
  EVT VT = Node->getValueType(0);
  Type *Ty = VT.getTypeForEVT(*DAG.getContext());
  EVT PtrVT = Op.getValueType();
  SDValue InChain = Node->getOperand(0);
  SDValue VAListPtr = Node->getOperand(1);
  const Value *SV = cast<SrcValueSDNode>(Node->getOperand(2))->getValue();
  SDLoc DL(Node);
  auto &TD = DAG.getDataLayout();
  Align ArgAlignment = TD.getABITypeAlign(Ty);
  unsigned ArgAlignInBytes = ArgAlignment.value();
  unsigned ArgSizeInBytes = TD.getTypeAllocSize(Ty);
  unsigned VASizeInBytes = llvm::alignTo(ArgSizeInBytes, 4);

  // va_stk
  SDValue VAStack =
      DAG.getLoad(MVT::i32, DL, InChain, VAListPtr, MachinePointerInfo());
  InChain = VAStack.getValue(1);

```
- **EN**: Implements logic around `LowerVAARG`, `getNode`, `getValueType`, `getTypeForEVT`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerVAARG`, `getNode`, `getValueType`, `getTypeForEVT`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 1297-1312
```cpp
  // va_reg
  SDValue VARegPtr =
      DAG.getObjectPtrOffset(DL, VAListPtr, TypeSize::getFixed(4));
  SDValue VAReg =
      DAG.getLoad(MVT::i32, DL, InChain, VARegPtr, MachinePointerInfo());
  InChain = VAReg.getValue(1);

  // va_ndx
  SDValue VarArgIndexPtr =
      DAG.getObjectPtrOffset(DL, VARegPtr, TypeSize::getFixed(4));
  SDValue VAIndex =
      DAG.getLoad(MVT::i32, DL, InChain, VarArgIndexPtr, MachinePointerInfo());
  InChain = VAIndex.getValue(1);

  SDValue OrigIndex = VAIndex;

```
- **EN**: Implements logic around `getObjectPtrOffset`, `getLoad`, `getValue`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getObjectPtrOffset`, `getLoad`, `getValue` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 1313-1330
```cpp
  if (ArgAlignInBytes > 4) {
    OrigIndex = DAG.getNode(ISD::ADD, DL, PtrVT, OrigIndex,
                            DAG.getConstant(ArgAlignInBytes - 1, DL, MVT::i32));
    OrigIndex =
        DAG.getNode(ISD::AND, DL, PtrVT, OrigIndex,
                    DAG.getSignedConstant(-ArgAlignInBytes, DL, MVT::i32));
  }

  VAIndex = DAG.getNode(ISD::ADD, DL, PtrVT, OrigIndex,
                        DAG.getConstant(VASizeInBytes, DL, MVT::i32));

  SDValue CC = DAG.getSetCC(DL, MVT::i32, OrigIndex,
                            DAG.getConstant(6 * 4, DL, MVT::i32), ISD::SETLE);

  SDValue StkIndex =
      DAG.getNode(ISD::ADD, DL, PtrVT, VAIndex,
                  DAG.getConstant(32 + VASizeInBytes, DL, MVT::i32));

```
- **EN**: Implements logic around `getNode`, `getConstant`, `getSignedConstant`, `getSetCC`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNode`, `getConstant`, `getSignedConstant`, `getSetCC` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 1331-1347
```cpp
  CC = DAG.getSetCC(DL, MVT::i32, VAIndex, DAG.getConstant(6 * 4, DL, MVT::i32),
                    ISD::SETLE);

  SDValue Array = DAG.getNode(ISD::SELECT, DL, MVT::i32, CC, VAReg, VAStack);

  VAIndex = DAG.getNode(ISD::SELECT, DL, MVT::i32, CC, VAIndex, StkIndex);

  CC = DAG.getSetCC(DL, MVT::i32, VAIndex, DAG.getConstant(6 * 4, DL, MVT::i32),
                    ISD::SETLE);

  SDValue VAIndexStore = DAG.getStore(InChain, DL, VAIndex, VarArgIndexPtr,
                                      MachinePointerInfo(SV));
  InChain = VAIndexStore;

  SDValue Addr = DAG.getNode(ISD::SUB, DL, PtrVT, VAIndex,
                             DAG.getConstant(VASizeInBytes, DL, MVT::i32));

```
- **EN**: Implements logic around `getSetCC`, `getNode`, `getStore`, `MachinePointerInfo`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getSetCC`, `getNode`, `getStore`, `MachinePointerInfo`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 1348-1366
```cpp
  Addr = DAG.getNode(ISD::ADD, DL, PtrVT, Array, Addr);

  return DAG.getLoad(VT, DL, InChain, Addr, MachinePointerInfo());
}

SDValue XtensaTargetLowering::LowerShiftLeftParts(SDValue Op,
                                                  SelectionDAG &DAG) const {
  SDLoc DL(Op);
  MVT VT = MVT::i32;
  SDValue Lo = Op.getOperand(0), Hi = Op.getOperand(1);
  SDValue Shamt = Op.getOperand(2);

  // if Shamt - register size < 0: // Shamt < register size
  //   Lo = Lo << Shamt
  //   Hi = (Hi << Shamt) | (Lo >>u (register size - Shamt))
  // else:
  //   Lo = 0
  //   Hi = Lo << (Shamt - register size)

```
- **EN**: Implements logic around `getNode`, `getLoad`, `LowerShiftLeftParts`, `DL`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNode`, `getLoad`, `LowerShiftLeftParts`, `DL`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1367-1382
```cpp
  SDValue MinusRegisterSize = DAG.getSignedConstant(-32, DL, VT);
  SDValue ShamtMinusRegisterSize =
      DAG.getNode(ISD::ADD, DL, VT, Shamt, MinusRegisterSize);

  SDValue LoTrue = DAG.getNode(ISD::SHL, DL, VT, Lo, Shamt);
  SDValue HiTrue = DAG.getNode(XtensaISD::SRCL, DL, VT, Hi, Lo, Shamt);
  SDValue Zero = DAG.getConstant(0, DL, VT);
  SDValue HiFalse = DAG.getNode(ISD::SHL, DL, VT, Lo, ShamtMinusRegisterSize);

  SDValue Cond = DAG.getSetCC(DL, VT, ShamtMinusRegisterSize, Zero, ISD::SETLT);
  Lo = DAG.getNode(ISD::SELECT, DL, VT, Cond, LoTrue, Zero);
  Hi = DAG.getNode(ISD::SELECT, DL, VT, Cond, HiTrue, HiFalse);

  return DAG.getMergeValues({Lo, Hi}, DL);
}

```
- **EN**: Implements logic around `getSignedConstant`, `getNode`, `getConstant`, `getSetCC`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getSignedConstant`, `getNode`, `getConstant`, `getSetCC`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1383-1406
```cpp
SDValue XtensaTargetLowering::LowerShiftRightParts(SDValue Op,
                                                   SelectionDAG &DAG,
                                                   bool IsSRA) const {
  SDLoc DL(Op);
  SDValue Lo = Op.getOperand(0), Hi = Op.getOperand(1);
  SDValue Shamt = Op.getOperand(2);
  MVT VT = MVT::i32;

  // SRA expansion:
  //   if Shamt - register size < 0: // Shamt < register size
  //     Lo = (Lo >>u Shamt) | (Hi << u (register size - Shamt))
  //     Hi = Hi >>s Shamt
  //   else:
  //     Lo = Hi >>s (Shamt - register size);
  //     Hi = Hi >>s (register size - 1)
  //
  // SRL expansion:
  //   if Shamt - register size < 0: // Shamt < register size
  //     Lo = (Lo >>u Shamt) | (Hi << u (register size - Shamt))
  //     Hi = Hi >>u Shamt
  //   else:
  //     Lo = Hi >>u (Shamt - register size);
  //     Hi = 0;

```
- **EN**: Implements logic around `LowerShiftRightParts`, `DL`, `getOperand`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerShiftRightParts`, `DL`, `getOperand` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 1407-1425
```cpp
  unsigned ShiftRightOp = IsSRA ? ISD::SRA : ISD::SRL;
  SDValue MinusRegisterSize = DAG.getSignedConstant(-32, DL, VT);
  SDValue RegisterSizeMinus1 = DAG.getConstant(32 - 1, DL, VT);
  SDValue ShamtMinusRegisterSize =
      DAG.getNode(ISD::ADD, DL, VT, Shamt, MinusRegisterSize);

  SDValue LoTrue = DAG.getNode(XtensaISD::SRCR, DL, VT, Hi, Lo, Shamt);
  SDValue HiTrue = DAG.getNode(ShiftRightOp, DL, VT, Hi, Shamt);
  SDValue Zero = DAG.getConstant(0, DL, VT);
  SDValue LoFalse =
      DAG.getNode(ShiftRightOp, DL, VT, Hi, ShamtMinusRegisterSize);
  SDValue HiFalse;

  if (IsSRA) {
    HiFalse = DAG.getNode(ShiftRightOp, DL, VT, Hi, RegisterSizeMinus1);
  } else {
    HiFalse = Zero;
  }

```
- **EN**: Implements logic around `getSignedConstant`, `getConstant`, `getNode`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getSignedConstant`, `getConstant`, `getNode` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 1426-1442
```cpp
  SDValue Cond = DAG.getSetCC(DL, VT, ShamtMinusRegisterSize, Zero, ISD::SETLT);
  Lo = DAG.getNode(ISD::SELECT, DL, VT, Cond, LoTrue, LoFalse);
  Hi = DAG.getNode(ISD::SELECT, DL, VT, Cond, HiTrue, HiFalse);

  return DAG.getMergeValues({Lo, Hi}, DL);
}

SDValue XtensaTargetLowering::LowerCTPOP(SDValue Op, SelectionDAG &DAG) const {
  auto &TLI = DAG.getTargetLoweringInfo();
  return TLI.expandCTPOP(Op.getNode(), DAG);
}

bool XtensaTargetLowering::decomposeMulByConstant(LLVMContext &Context, EVT VT,
                                                  SDValue C) const {
  APInt Imm;
  unsigned EltSizeInBits;

```
- **EN**: Implements logic around `getSetCC`, `getNode`, `getMergeValues`, `LowerCTPOP`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getSetCC`, `getNode`, `getMergeValues`, `LowerCTPOP`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1443-1458
```cpp
  if (ISD::isConstantSplatVector(C.getNode(), Imm)) {
    EltSizeInBits = VT.getScalarSizeInBits();
  } else if (VT.isScalarInteger()) {
    EltSizeInBits = VT.getSizeInBits();
    if (auto *ConstNode = dyn_cast<ConstantSDNode>(C.getNode()))
      Imm = ConstNode->getAPIntValue();
    else
      return false;
  } else {
    return false;
  }

  // Omit if data size exceeds.
  if (EltSizeInBits > 32)
    return false;

```
- **EN**: Implements logic around `getScalarSizeInBits`, `getSizeInBits`, `getAPIntValue`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getScalarSizeInBits`, `getSizeInBits`, `getAPIntValue` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 1459-1490
```cpp
  // Convert MULT to LSL.
  if (Imm.isPowerOf2() && Imm.isIntN(5))
    return true;

  return false;
}

SDValue XtensaTargetLowering::LowerOperation(SDValue Op,
                                             SelectionDAG &DAG) const {
  switch (Op.getOpcode()) {
  case ISD::BR_JT:
    return LowerBR_JT(Op, DAG);
  case ISD::Constant:
    return LowerImmediate(Op, DAG);
  case ISD::RETURNADDR:
    return LowerRETURNADDR(Op, DAG);
  case ISD::GlobalAddress:
    return LowerGlobalAddress(Op, DAG);
  case ISD::GlobalTLSAddress:
    return LowerGlobalTLSAddress(Op, DAG);
  case ISD::BlockAddress:
    return LowerBlockAddress(Op, DAG);
  case ISD::JumpTable:
    return LowerJumpTable(Op, DAG);
  case ISD::CTPOP:
    return LowerCTPOP(Op, DAG);
  case ISD::ConstantPool:
    return LowerConstantPool(Op, DAG);
  case ISD::SELECT_CC:
    return LowerSELECT_CC(Op, DAG);
  case ISD::STACKSAVE:
    return LowerSTACKSAVE(Op, DAG);
```
- **EN**: Implements logic around `LowerOperation`, `LowerBR_JT`, `LowerImmediate`, `LowerRETURNADDR`, ...; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerOperation`, `LowerBR_JT`, `LowerImmediate`, `LowerRETURNADDR`, ... 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1491-1513
```cpp
  case ISD::STACKRESTORE:
    return LowerSTACKRESTORE(Op, DAG);
  case ISD::FRAMEADDR:
    return LowerFRAMEADDR(Op, DAG);
  case ISD::DYNAMIC_STACKALLOC:
    return LowerDYNAMIC_STACKALLOC(Op, DAG);
  case ISD::VASTART:
    return LowerVASTART(Op, DAG);
  case ISD::VAARG:
    return LowerVAARG(Op, DAG);
  case ISD::VACOPY:
    return LowerVACOPY(Op, DAG);
  case ISD::SHL_PARTS:
    return LowerShiftLeftParts(Op, DAG);
  case ISD::SRA_PARTS:
    return LowerShiftRightParts(Op, DAG, true);
  case ISD::SRL_PARTS:
    return LowerShiftRightParts(Op, DAG, false);
  default:
    report_fatal_error("Unexpected node to lower");
  }
}

```
- **EN**: Implements logic around `LowerSTACKRESTORE`, `LowerFRAMEADDR`, `LowerDYNAMIC_STACKALLOC`, `LowerVASTART`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerSTACKRESTORE`, `LowerFRAMEADDR`, `LowerDYNAMIC_STACKALLOC`, `LowerVASTART`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1514-1533
```cpp
TargetLowering::AtomicExpansionKind
XtensaTargetLowering::shouldExpandAtomicRMWInIR(const AtomicRMWInst *AI) const {
  return AtomicExpansionKind::CmpXChg;
}

//===----------------------------------------------------------------------===//
// Custom insertion
//===----------------------------------------------------------------------===//

MachineBasicBlock *
XtensaTargetLowering::emitSelectCC(MachineInstr &MI,
                                   MachineBasicBlock *MBB) const {
  const TargetInstrInfo &TII = *Subtarget.getInstrInfo();
  DebugLoc DL = MI.getDebugLoc();

  MachineOperand &LHS = MI.getOperand(1);
  MachineOperand &RHS = MI.getOperand(2);
  MachineOperand &TrueValue = MI.getOperand(3);
  MachineOperand &FalseValue = MI.getOperand(4);

```
- **EN**: Implements logic around `shouldExpandAtomicRMWInIR`, `emitSelectCC`, `getInstrInfo`, `getDebugLoc`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `shouldExpandAtomicRMWInIR`, `emitSelectCC`, `getInstrInfo`, `getDebugLoc`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 1534-1552
```cpp
  // To "insert" a SELECT_CC instruction, we actually have to insert
  // CopyMBB and SinkMBB  blocks and add branch to MBB. We build phi
  // operation in SinkMBB like phi (TrueVakue,FalseValue), where TrueValue
  // is passed from MMB and FalseValue is passed from CopyMBB.
  //   MBB
  //   |   \
  //   |   CopyMBB
  //   |   /
  //   SinkMBB
  // The incoming instruction knows the
  // destination vreg to set, the condition code register to branch on, the
  // true/false values to select between, and a branch opcode to use.
  const BasicBlock *LLVM_BB = MBB->getBasicBlock();
  MachineFunction::iterator It = ++MBB->getIterator();

  MachineFunction *F = MBB->getParent();
  MachineBasicBlock *CopyMBB = F->CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *SinkMBB = F->CreateMachineBasicBlock(LLVM_BB);

```
- **EN**: Implements logic around `getBasicBlock`, `getIterator`, `getParent`, `CreateMachineBasicBlock`; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getBasicBlock`, `getIterator`, `getParent`, `CreateMachineBasicBlock` 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 1553-1569
```cpp
  F->insert(It, CopyMBB);
  F->insert(It, SinkMBB);

  // Transfer the remainder of MBB and its successor edges to SinkMBB.
  SinkMBB->splice(SinkMBB->begin(), MBB,
                  std::next(MachineBasicBlock::iterator(MI)), MBB->end());
  SinkMBB->transferSuccessorsAndUpdatePHIs(MBB);

  MBB->addSuccessor(CopyMBB);
  MBB->addSuccessor(SinkMBB);

  if (MI.getOpcode() == Xtensa::SELECT_CC_FP_FP ||
      MI.getOpcode() == Xtensa::SELECT_CC_FP_INT) {
    unsigned CmpKind = MI.getOperand(5).getImm();
    unsigned BrKind = MI.getOperand(6).getImm();
    MCPhysReg BReg = Xtensa::B0;

```
- **EN**: Implements logic around `insert`, `splice`, `next`, `transferSuccessorsAndUpdatePHIs`, ...; this block applies conditional target rules; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `insert`, `splice`, `next`, `transferSuccessorsAndUpdatePHIs`, ... 实现具体逻辑；这一段应用条件化的目标规则，工作在 MachineInstr/MachineFunction 层。

### Lines 1570-1585
```cpp
    BuildMI(MBB, DL, TII.get(CmpKind), BReg)
        .addReg(LHS.getReg())
        .addReg(RHS.getReg());
    BuildMI(MBB, DL, TII.get(BrKind))
        .addReg(BReg, RegState::Kill)
        .addMBB(SinkMBB);
  } else {
    unsigned BrKind = MI.getOperand(5).getImm();
    BuildMI(MBB, DL, TII.get(BrKind))
        .addReg(LHS.getReg())
        .addReg(RHS.getReg())
        .addMBB(SinkMBB);
  }

  CopyMBB->addSuccessor(SinkMBB);

```
- **EN**: Implements logic around `BuildMI`, `addReg`, `addMBB`, `getOperand`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `addMBB`, `getOperand`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 1586-1605
```cpp
  //  SinkMBB:
  //   %Result = phi [ %FalseValue, CopyMBB ], [ %TrueValue, MBB ]
  //  ...

  BuildMI(*SinkMBB, SinkMBB->begin(), DL, TII.get(Xtensa::PHI),
          MI.getOperand(0).getReg())
      .addReg(FalseValue.getReg())
      .addMBB(CopyMBB)
      .addReg(TrueValue.getReg())
      .addMBB(MBB);

  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return SinkMBB;
}

MachineBasicBlock *XtensaTargetLowering::EmitInstrWithCustomInserter(
    MachineInstr &MI, MachineBasicBlock *MBB) const {
  DebugLoc DL = MI.getDebugLoc();
  const XtensaInstrInfo &TII = *Subtarget.getInstrInfo();

```
- **EN**: Implements logic around `BuildMI`, `getOperand`, `addReg`, `addMBB`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `getOperand`, `addReg`, `addMBB`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 1606-1624
```cpp
  switch (MI.getOpcode()) {
  case Xtensa::BRCC_FP: {
    MachineOperand &Cond = MI.getOperand(0);
    MachineOperand &LHS = MI.getOperand(1);
    MachineOperand &RHS = MI.getOperand(2);
    MachineBasicBlock *TargetBB = MI.getOperand(3).getMBB();
    unsigned BrKind = 0;
    unsigned CmpKind = 0;
    ISD::CondCode CondCode = (ISD::CondCode)Cond.getImm();
    MCPhysReg BReg = Xtensa::B0;

    std::tie(BrKind, CmpKind) = getFPBranchKind(CondCode);
    BuildMI(*MBB, MI, DL, TII.get(CmpKind), BReg)
        .addReg(LHS.getReg())
        .addReg(RHS.getReg());
    BuildMI(*MBB, MI, DL, TII.get(BrKind))
        .addReg(BReg, RegState::Kill)
        .addMBB(TargetBB);

```
- **EN**: Implements logic around `getOperand`, `getImm`, `tie`, `BuildMI`, ...; this block uses `switch`-based dispatch; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand`, `getImm`, `tie`, `BuildMI`, ... 实现具体逻辑；这一段使用 `switch` 分派，工作在 MachineInstr/MachineFunction 层。

### Lines 1625-1656
```cpp
    MI.eraseFromParent();
    return MBB;
  }
  case Xtensa::SELECT_CC_FP_FP:
  case Xtensa::SELECT_CC_FP_INT:
  case Xtensa::SELECT_CC_INT_FP:
  case Xtensa::SELECT:
    return emitSelectCC(MI, MBB);
  case Xtensa::S8I:
  case Xtensa::S16I:
  case Xtensa::S32I:
  case Xtensa::S32I_N:
  case Xtensa::SSI:
  case Xtensa::SSIP:
  case Xtensa::SSX:
  case Xtensa::SSXP:
  case Xtensa::L8UI:
  case Xtensa::L16SI:
  case Xtensa::L16UI:
  case Xtensa::L32I:
  case Xtensa::L32I_N:
  case Xtensa::LSI:
  case Xtensa::LSIP:
  case Xtensa::LSX:
  case Xtensa::LSXP: {
    // Insert memory wait instruction "memw" before volatile load/store as it is
    // implemented in gcc. If memoperands is empty then assume that it aslo
    // maybe volatile load/store and insert "memw".
    if (MI.memoperands_empty() || (*MI.memoperands_begin())->isVolatile()) {
      BuildMI(*MBB, MI, DL, TII.get(Xtensa::MEMW));
    }
    return MBB;
```
- **EN**: Implements logic around `eraseFromParent`, `emitSelectCC`, `BuildMI`; this block applies conditional target rules; returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `eraseFromParent`, `emitSelectCC`, `BuildMI` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 1657-1672
```cpp
  }
  case Xtensa::MOVSP_P: {
    MachineOperand &NewSP = MI.getOperand(0);

    BuildMI(*MBB, MI, DL, TII.get(Xtensa::MOVSP), Xtensa::SP)
        .addReg(NewSP.getReg());
    MI.eraseFromParent();

    return MBB;
  }
  case Xtensa::ATOMIC_CMP_SWAP_32_P: {
    MachineOperand &R = MI.getOperand(0);
    MachineOperand &Addr = MI.getOperand(1);
    MachineOperand &Cmp = MI.getOperand(2);
    MachineOperand &Swap = MI.getOperand(3);

```
- **EN**: Implements logic around `getOperand`, `BuildMI`, `addReg`, `eraseFromParent`; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getOperand`, `BuildMI`, `addReg`, `eraseFromParent` 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

### Lines 1673-1687
```cpp
    BuildMI(*MBB, MI, DL, TII.get(Xtensa::WSR), Xtensa::SCOMPARE1)
        .addReg(Cmp.getReg());

    BuildMI(*MBB, MI, DL, TII.get(Xtensa::S32C1I), R.getReg())
        .addReg(Swap.getReg())
        .addReg(Addr.getReg())
        .addImm(0);

    MI.eraseFromParent();
    return MBB;
  }
  default:
    llvm_unreachable("Unexpected instr type to insert");
  }
}
```
- **EN**: Implements logic around `BuildMI`, `addReg`, `addImm`, `eraseFromParent`, ...; this block returns target-specific results; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `addImm`, `eraseFromParent`, ... 实现具体逻辑；这一段返回目标相关结果，工作在 MachineInstr/MachineFunction 层。

## Key Concepts / 关键概念

- **SelectionDAG lowering / SelectionDAG 降低**:
  - **EN**: Custom lowering/legalization rules for target operations
  - **CN**: 为目标操作定制 lowering 与合法化规则
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

- **Direct includes / 直接包含**: `XtensaISelLowering.h`, `XtensaConstantPoolValue.h`, `XtensaInstrInfo.h`, `XtensaMachineFunctionInfo.h`, `XtensaSelectionDAGInfo.h`, `XtensaSubtarget.h`, `XtensaTargetMachine.h`, `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineJumpTableInfo.h`, `llvm/CodeGen/MachineRegisterInfo.h` ... (+8 more)
- **LLVM subsystems / LLVM 子系统**: CodeGen, IR, Support
