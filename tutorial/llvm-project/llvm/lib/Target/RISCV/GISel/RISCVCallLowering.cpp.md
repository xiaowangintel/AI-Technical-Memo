# RISCVCallLowering.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/GISel/RISCVCallLowering.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: Implements GlobalISel call lowering for RISC-V calling conventions. / 实现面向 RISC-V 调用约定的 GlobalISel 调用下降。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVCallLowering.cpp - Call lowering -------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This file implements the lowering of LLVM calls to machine code calls for
/// GlobalISel.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 15-28: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
#include "RISCVCallLowering.h"
#include "RISCVCallingConv.h"
#include "RISCVISelLowering.h"
#include "RISCVMachineFunctionInfo.h"
#include "RISCVSubtarget.h"
#include "llvm/CodeGen/Analysis.h"
#include "llvm/CodeGen/FunctionLoweringInfo.h"
#include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
#include "llvm/CodeGen/MachineFrameInfo.h"

using namespace llvm;

namespace {
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 29-43: Type declaration for RISCVOutgoingValueHandler / RISCVOutgoingValueHandler 的类型声明
```cpp
struct RISCVOutgoingValueHandler : public CallLowering::OutgoingValueHandler {
  RISCVOutgoingValueHandler(MachineIRBuilder &B, MachineRegisterInfo &MRI,
                            MachineInstrBuilder MIB)
      : OutgoingValueHandler(B, MRI), MIB(MIB),
        Subtarget(MIRBuilder.getMF().getSubtarget<RISCVSubtarget>()) {}
  Register getStackAddress(uint64_t MemSize, int64_t Offset,
                           MachinePointerInfo &MPO,
                           ISD::ArgFlagsTy Flags) override {
    MachineFunction &MF = MIRBuilder.getMF();
    LLT p0 = LLT::pointer(0, Subtarget.getXLen());
    LLT sXLen = LLT::scalar(Subtarget.getXLen());

    if (!SPReg)
      SPReg = MIRBuilder.buildCopy(p0, Register(RISCV::X2)).getReg(0);
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 44-57: Function implementation: MachinePointerInfo::getStack / 函数实现：MachinePointerInfo::getStack
```cpp
    auto OffsetReg = MIRBuilder.buildConstant(sXLen, Offset);

    auto AddrReg = MIRBuilder.buildPtrAdd(p0, SPReg, OffsetReg);

    MPO = MachinePointerInfo::getStack(MF, Offset);
    return AddrReg.getReg(0);
  }

  void assignValueToAddress(Register ValVReg, Register Addr, LLT MemTy,
                            const MachinePointerInfo &MPO,
                            const CCValAssign &VA) override {
    MachineFunction &MF = MIRBuilder.getMF();
    uint64_t LocMemOffset = VA.getLocMemOffset();
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 58-73: Function implementation / 函数实现
```cpp
    // TODO: Move StackAlignment to subtarget and share with FrameLowering.
    auto MMO =
        MF.getMachineMemOperand(MPO, MachineMemOperand::MOStore, MemTy,
                                commonAlignment(Align(16), LocMemOffset));

    Register ExtReg = extendRegister(ValVReg, VA);
    MIRBuilder.buildStore(ExtReg, Addr, *MMO);
  }

  void assignValueToReg(Register ValVReg, Register PhysReg,
                        const CCValAssign &VA,
                        ISD::ArgFlagsTy Flags = {}) override {
    Register ExtReg = extendRegister(ValVReg, VA);
    MIRBuilder.buildCopy(PhysReg, ExtReg);
    MIB.addUse(PhysReg, RegState::Implicit);
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 74-87: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  unsigned assignCustomValue(CallLowering::ArgInfo &Arg,
                             ArrayRef<CCValAssign> VAs,
                             std::function<void()> *Thunk) override {
    const CCValAssign &VA = VAs[0];
    if ((VA.getLocVT() == MVT::i64 && VA.getValVT() == MVT::f32) ||
        (VA.getLocVT().isInteger() && VA.getValVT() == MVT::f16)) {
      Register PhysReg = VA.getLocReg();

      auto assignFunc = [=]() {
        auto Trunc = MIRBuilder.buildAnyExt(LLT(VA.getLocVT()), Arg.Regs[0]);
        MIRBuilder.buildCopy(PhysReg, Trunc);
        MIB.addUse(PhysReg, RegState::Implicit);
      };
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 88-104: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

      if (Thunk) {
        *Thunk = std::move(assignFunc);
        return 1;
      }

      assignFunc();
      return 1;
    }

    assert(VAs.size() >= 2 && "Expected at least 2 VAs.");
    const CCValAssign &VAHi = VAs[1];

    assert(VAHi.needsCustom() && "Value doesn't need custom handling");
    assert(VA.getValNo() == VAHi.getValNo() &&
           "Values belong to different arguments");
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 105-119: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    assert(VA.getLocVT() == MVT::i32 && VAHi.getLocVT() == MVT::i32 &&
           VA.getValVT() == MVT::f64 && VAHi.getValVT() == MVT::f64 &&
           "unexpected custom value");

    Register NewRegs[] = {MRI.createGenericVirtualRegister(LLT::scalar(32)),
                          MRI.createGenericVirtualRegister(LLT::scalar(32))};
    MIRBuilder.buildUnmerge(NewRegs, Arg.Regs[0]);

    if (VAHi.isMemLoc()) {
      LLT MemTy(VAHi.getLocVT());

      MachinePointerInfo MPO;
      Register StackAddr = getStackAddress(
          MemTy.getSizeInBytes(), VAHi.getLocMemOffset(), MPO, Arg.Flags[0]);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 120-133: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
      assignValueToAddress(NewRegs[1], StackAddr, MemTy, MPO,
                           const_cast<CCValAssign &>(VAHi));
    }

    auto assignFunc = [=]() {
      assignValueToReg(NewRegs[0], VA.getLocReg(), VA);
      if (VAHi.isRegLoc())
        assignValueToReg(NewRegs[1], VAHi.getLocReg(), VAHi);
    };

    if (Thunk) {
      *Thunk = std::move(assignFunc);
      return 2;
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 134-147: Definitions and supporting logic / 定义与支撑逻辑
```cpp

    assignFunc();
    return 2;
  }

private:
  MachineInstrBuilder MIB;

  // Cache the SP register vreg if we need it more than once in this call site.
  Register SPReg;

  const RISCVSubtarget &Subtarget;
};
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 148-162: Type declaration for RISCVIncomingValueHandler / RISCVIncomingValueHandler 的类型声明
```cpp
struct RISCVIncomingValueHandler : public CallLowering::IncomingValueHandler {
  RISCVIncomingValueHandler(MachineIRBuilder &B, MachineRegisterInfo &MRI)
      : IncomingValueHandler(B, MRI),
        Subtarget(MIRBuilder.getMF().getSubtarget<RISCVSubtarget>()) {}

  Register getStackAddress(uint64_t MemSize, int64_t Offset,
                           MachinePointerInfo &MPO,
                           ISD::ArgFlagsTy Flags) override {
    MachineFrameInfo &MFI = MIRBuilder.getMF().getFrameInfo();

    int FI = MFI.CreateFixedObject(MemSize, Offset, /*Immutable=*/true);
    MPO = MachinePointerInfo::getFixedStack(MIRBuilder.getMF(), FI);
    return MIRBuilder.buildFrameIndex(LLT::pointer(0, Subtarget.getXLen()), FI)
        .getReg(0);
  }
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 163-178: Function implementation: IncomingValueHandler::assignValueToReg / 函数实现：IncomingValueHandler::assignValueToReg
```cpp

  void assignValueToAddress(Register ValVReg, Register Addr, LLT MemTy,
                            const MachinePointerInfo &MPO,
                            const CCValAssign &VA) override {
    MachineFunction &MF = MIRBuilder.getMF();
    auto MMO = MF.getMachineMemOperand(MPO, MachineMemOperand::MOLoad, MemTy,
                                       inferAlignFromPtrInfo(MF, MPO));
    MIRBuilder.buildLoad(ValVReg, Addr, *MMO);
  }

  void assignValueToReg(Register ValVReg, Register PhysReg,
                        const CCValAssign &VA,
                        ISD::ArgFlagsTy Flags = {}) override {
    markPhysRegUsed(PhysReg);
    IncomingValueHandler::assignValueToReg(ValVReg, PhysReg, VA);
  }
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 179-192: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  unsigned assignCustomValue(CallLowering::ArgInfo &Arg,
                             ArrayRef<CCValAssign> VAs,
                             std::function<void()> *Thunk) override {
    const CCValAssign &VA = VAs[0];
    if ((VA.getLocVT() == MVT::i64 && VA.getValVT() == MVT::f32) ||
        (VA.getLocVT().isInteger() && VA.getValVT() == MVT::f16)) {
      Register PhysReg = VA.getLocReg();

      markPhysRegUsed(PhysReg);

      LLT LocTy(VA.getLocVT());
      auto Copy = MIRBuilder.buildCopy(LocTy, PhysReg);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 193-207: Definitions and supporting logic / 定义与支撑逻辑
```cpp
      MIRBuilder.buildTrunc(Arg.Regs[0], Copy.getReg(0));
      return 1;
    }

    assert(VAs.size() >= 2 && "Expected at least 2 VAs.");
    const CCValAssign &VAHi = VAs[1];

    assert(VAHi.needsCustom() && "Value doesn't need custom handling");
    assert(VA.getValNo() == VAHi.getValNo() &&
           "Values belong to different arguments");

    assert(VA.getLocVT() == MVT::i32 && VAHi.getLocVT() == MVT::i32 &&
           VA.getValVT() == MVT::f64 && VAHi.getValVT() == MVT::f64 &&
           "unexpected custom value");
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 208-221: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    Register NewRegs[] = {MRI.createGenericVirtualRegister(LLT::scalar(32)),
                          MRI.createGenericVirtualRegister(LLT::scalar(32))};

    if (VAHi.isMemLoc()) {
      LLT MemTy(VAHi.getLocVT());

      MachinePointerInfo MPO;
      Register StackAddr = getStackAddress(
          MemTy.getSizeInBytes(), VAHi.getLocMemOffset(), MPO, Arg.Flags[0]);

      assignValueToAddress(NewRegs[1], StackAddr, MemTy, MPO,
                           const_cast<CCValAssign &>(VAHi));
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 222-235: Definitions and supporting logic / 定义与支撑逻辑
```cpp
    assignValueToReg(NewRegs[0], VA.getLocReg(), VA);
    if (VAHi.isRegLoc())
      assignValueToReg(NewRegs[1], VAHi.getLocReg(), VAHi);

    MIRBuilder.buildMergeLikeInstr(Arg.Regs[0], NewRegs);

    return 2;
  }

  /// How the physical register gets marked varies between formal
  /// parameters (it's a basic-block live-in), and a call instruction
  /// (it's an implicit-def of the BL).
  virtual void markPhysRegUsed(MCRegister PhysReg) = 0;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 236-249: Type declaration for RISCVFormalArgHandler / RISCVFormalArgHandler 的类型声明
```cpp
private:
  const RISCVSubtarget &Subtarget;
};

struct RISCVFormalArgHandler : public RISCVIncomingValueHandler {
  RISCVFormalArgHandler(MachineIRBuilder &B, MachineRegisterInfo &MRI)
      : RISCVIncomingValueHandler(B, MRI) {}

  void markPhysRegUsed(MCRegister PhysReg) override {
    MIRBuilder.getMRI()->addLiveIn(PhysReg);
    MIRBuilder.getMBB().addLiveIn(PhysReg);
  }
};
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 250-263: Namespace and file-scope setup / 命名空间与文件作用域设置
```cpp
struct RISCVCallReturnHandler : public RISCVIncomingValueHandler {
  RISCVCallReturnHandler(MachineIRBuilder &B, MachineRegisterInfo &MRI,
                         MachineInstrBuilder &MIB)
      : RISCVIncomingValueHandler(B, MRI), MIB(MIB) {}

  void markPhysRegUsed(MCRegister PhysReg) override {
    MIB.addDef(PhysReg, RegState::Implicit);
  }

  MachineInstrBuilder MIB;
};

} // namespace
```
**EN:** This block establishes namespaces, aliases, or small file-scope helpers that keep later code concise.

**CN:** 该区段建立命名空间、别名或文件级辅助项，以便后续代码保持简洁。

### Lines 264-286: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
RISCVCallLowering::RISCVCallLowering(const RISCVTargetLowering &TLI)
    : CallLowering(&TLI) {}

/// Return true if scalable vector with ScalarTy is legal for lowering.
static bool isLegalElementTypeForRVV(Type *EltTy,
                                     const RISCVSubtarget &Subtarget) {
  if (EltTy->isPointerTy())
    return Subtarget.is64Bit() ? Subtarget.hasVInstructionsI64() : true;
  if (EltTy->isIntegerTy(1) || EltTy->isIntegerTy(8) ||
      EltTy->isIntegerTy(16) || EltTy->isIntegerTy(32))
    return true;
  if (EltTy->isIntegerTy(64))
    return Subtarget.hasVInstructionsI64();
  if (EltTy->isHalfTy())
    return Subtarget.hasVInstructionsF16Minimal();
  if (EltTy->isBFloatTy())
    return Subtarget.hasVInstructionsBF16Minimal();
  if (EltTy->isFloatTy())
    return Subtarget.hasVInstructionsF32();
  if (EltTy->isDoubleTy())
    return Subtarget.hasVInstructionsF64();
  return false;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 287-307: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

// TODO: Support all argument types.
// TODO: Remove IsLowerArgs argument by adding support for vectors in lowerCall.
static bool isSupportedArgumentType(Type *T, const RISCVSubtarget &Subtarget,
                                    bool IsLowerArgs = false) {
  if (T->isIntegerTy())
    return true;
  if (T->isHalfTy() || T->isFloatTy() || T->isDoubleTy() || T->isFP128Ty())
    return true;
  if (T->isPointerTy())
    return true;
  if (T->isArrayTy())
    return isSupportedArgumentType(T->getArrayElementType(), Subtarget,
                                   IsLowerArgs);
  // TODO: Support fixed vector types.
  if (IsLowerArgs && T->isVectorTy() && Subtarget.hasVInstructions() &&
      T->isScalableTy() &&
      isLegalElementTypeForRVV(T->getScalarType(), Subtarget))
    return true;
  return false;
}
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 308-326: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

// TODO: Only integer, pointer and aggregate types are supported now.
// TODO: Remove IsLowerRetVal argument by adding support for vectors in
// lowerCall.
static bool isSupportedReturnType(Type *T, const RISCVSubtarget &Subtarget,
                                  bool IsLowerRetVal = false) {
  if (T->isIntegerTy() || T->isFloatingPointTy() || T->isPointerTy())
    return true;

  if (T->isArrayTy())
    return isSupportedReturnType(T->getArrayElementType(), Subtarget);

  if (T->isStructTy()) {
    auto StructT = cast<StructType>(T);
    for (unsigned i = 0, e = StructT->getNumElements(); i != e; ++i)
      if (!isSupportedReturnType(StructT->getElementType(i), Subtarget))
        return false;
    return true;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 327-341: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  if (IsLowerRetVal && T->isVectorTy() && Subtarget.hasVInstructions() &&
      T->isScalableTy() &&
      isLegalElementTypeForRVV(T->getScalarType(), Subtarget))
    return true;

  return false;
}

bool RISCVCallLowering::lowerReturn(MachineIRBuilder &MIRBuilder,
                                    const Value *Val, ArrayRef<Register> VRegs,
                                    FunctionLoweringInfo &FLI) const {
  assert(!Val == VRegs.empty() && "Return value without a vreg");
  MachineInstrBuilder Ret = MIRBuilder.buildInstrNoInsert(RISCV::PseudoRET);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 342-355: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (!FLI.CanLowerReturn) {
    insertSRetStores(MIRBuilder, Val->getType(), VRegs, FLI.DemoteRegister);
  } else if (!VRegs.empty()) {
    const RISCVSubtarget &Subtarget =
        MIRBuilder.getMF().getSubtarget<RISCVSubtarget>();
    if (!isSupportedReturnType(Val->getType(), Subtarget,
                               /*IsLowerRetVal=*/true))
      return false;

    MachineFunction &MF = MIRBuilder.getMF();
    const DataLayout &DL = MF.getDataLayout();
    const Function &F = MF.getFunction();
    CallingConv::ID CC = F.getCallingConv();
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 356-370: Definitions and supporting logic / 定义与支撑逻辑
```cpp
    ArgInfo OrigRetInfo(VRegs, Val->getType(), 0);
    setArgFlags(OrigRetInfo, AttributeList::ReturnIndex, DL, F);

    SmallVector<ArgInfo, 4> SplitRetInfos;
    splitToValueTypes(OrigRetInfo, SplitRetInfos, DL, CC);

    OutgoingValueAssigner Assigner(RetCC_RISCV);
    RISCVOutgoingValueHandler Handler(MIRBuilder, MF.getRegInfo(), Ret);

    SmallVector<CCValAssign, 16> RetLocs;
    CCState CCInfo(CC, F.isVarArg(), MF, RetLocs, F.getContext());
    if (!determineAssignments(Assigner, SplitRetInfos, CCInfo) ||
        !handleAssignments(Handler, SplitRetInfos, CCInfo, RetLocs, MIRBuilder))
      return false;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 371-388: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    if (any_of(RetLocs, [](CCValAssign &VA) {
          return VA.getLocVT().isScalableVector();
        }))
      MF.getInfo<RISCVMachineFunctionInfo>()->setIsVectorCall();
  }

  MIRBuilder.insertInstr(Ret);
  return true;
}

bool RISCVCallLowering::canLowerReturn(MachineFunction &MF,
                                       CallingConv::ID CallConv,
                                       SmallVectorImpl<BaseArgInfo> &Outs,
                                       bool IsVarArg) const {
  SmallVector<CCValAssign, 16> RetLocs;
  CCState CCInfo(CallConv, IsVarArg, MF, RetLocs,
                 MF.getFunction().getContext());
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 389-405: Function implementation: RISCVCallLowering::saveVarArgRegisters / 函数实现：RISCVCallLowering::saveVarArgRegisters
```cpp
  return checkReturn(CCInfo, Outs, RetCC_RISCV);
}

/// If there are varargs that were passed in a0-a7, the data in those registers
/// must be copied to the varargs save area on the stack.
void RISCVCallLowering::saveVarArgRegisters(
    MachineIRBuilder &MIRBuilder, CallLowering::IncomingValueHandler &Handler,
    IncomingValueAssigner &Assigner, CCState &CCInfo) const {
  MachineFunction &MF = MIRBuilder.getMF();
  const RISCVSubtarget &Subtarget = MF.getSubtarget<RISCVSubtarget>();
  unsigned XLenInBytes = Subtarget.getXLen() / 8;
  ArrayRef<MCPhysReg> ArgRegs = RISCV::getArgGPRs(Subtarget.getTargetABI());
  MachineRegisterInfo &MRI = MF.getRegInfo();
  unsigned Idx = CCInfo.getFirstUnallocated(ArgRegs);
  MachineFrameInfo &MFI = MF.getFrameInfo();
  RISCVMachineFunctionInfo *RVFI = MF.getInfo<RISCVMachineFunctionInfo>();
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 406-419: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  // Size of the vararg save area. For now, the varargs save area is either
  // zero or large enough to hold a0-a7.
  int VarArgsSaveSize = XLenInBytes * (ArgRegs.size() - Idx);
  int FI;

  // If all registers are allocated, then all varargs must be passed on the
  // stack and we don't need to save any argregs.
  if (VarArgsSaveSize == 0) {
    int VaArgOffset = Assigner.StackSize;
    FI = MFI.CreateFixedObject(XLenInBytes, VaArgOffset, true);
  } else {
    int VaArgOffset = -VarArgsSaveSize;
    FI = MFI.CreateFixedObject(VarArgsSaveSize, VaArgOffset, true);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 420-436: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    // If saving an odd number of registers then create an extra stack slot to
    // ensure that the frame pointer is 2*XLEN-aligned, which in turn ensures
    // offsets to even-numbered registered remain 2*XLEN-aligned.
    if (Idx % 2) {
      MFI.CreateFixedObject(XLenInBytes,
                            VaArgOffset - static_cast<int>(XLenInBytes), true);
      VarArgsSaveSize += XLenInBytes;
    }

    const LLT p0 = LLT::pointer(MF.getDataLayout().getAllocaAddrSpace(),
                                Subtarget.getXLen());
    const LLT sXLen = LLT::scalar(Subtarget.getXLen());

    auto FIN = MIRBuilder.buildFrameIndex(p0, FI);
    auto Offset = MIRBuilder.buildConstant(
        MRI.createGenericVirtualRegister(sXLen), XLenInBytes);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 437-451: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
    // Copy the integer registers that may have been used for passing varargs
    // to the vararg save area.
    const MVT XLenVT = Subtarget.getXLenVT();
    for (unsigned I = Idx; I < ArgRegs.size(); ++I) {
      const Register VReg = MRI.createGenericVirtualRegister(sXLen);
      Handler.assignValueToReg(
          VReg, ArgRegs[I],
          CCValAssign::getReg(I + MF.getFunction().getNumOperands(), XLenVT,
                              ArgRegs[I], XLenVT, CCValAssign::Full));
      auto MPO =
          MachinePointerInfo::getFixedStack(MF, FI, (I - Idx) * XLenInBytes);
      MIRBuilder.buildStore(VReg, FIN, MPO, inferAlignFromPtrInfo(MF, MPO));
      FIN = MIRBuilder.buildPtrAdd(MRI.createGenericVirtualRegister(p0),
                                   FIN.getReg(0), Offset);
    }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 452-465: Function implementation: RISCVCallLowering::lowerFormalArguments / 函数实现：RISCVCallLowering::lowerFormalArguments
```cpp
  }

  // Record the frame index of the first variable argument which is a value
  // necessary to G_VASTART.
  RVFI->setVarArgsFrameIndex(FI);
  RVFI->setVarArgsSaveSize(VarArgsSaveSize);
}

bool RISCVCallLowering::lowerFormalArguments(MachineIRBuilder &MIRBuilder,
                                             const Function &F,
                                             ArrayRef<ArrayRef<Register>> VRegs,
                                             FunctionLoweringInfo &FLI) const {
  MachineFunction &MF = MIRBuilder.getMF();
```
**EN:** This block implements a focused unit of backend behavior and cooperates with surrounding helpers to realize RISC-V semantics.

**CN:** 该区段实现一个集中的后端行为单元，并与周边辅助逻辑协作以落实 RISC-V 语义。

### Lines 466-483: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  const RISCVSubtarget &Subtarget = MF.getSubtarget<RISCVSubtarget>();
  for (auto &Arg : F.args()) {
    if (!isSupportedArgumentType(Arg.getType(), Subtarget,
                                 /*IsLowerArgs=*/true))
      return false;
  }

  MachineRegisterInfo &MRI = MF.getRegInfo();
  const DataLayout &DL = MF.getDataLayout();
  CallingConv::ID CC = F.getCallingConv();

  SmallVector<ArgInfo, 32> SplitArgInfos;

  // Insert the hidden sret parameter if the return value won't fit in the
  // return registers.
  if (!FLI.CanLowerReturn)
    insertSRetIncomingArgument(F, SplitArgInfos, FLI.DemoteRegister, MRI, DL);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 484-497: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  unsigned Index = 0;
  for (auto &Arg : F.args()) {
    // Construct the ArgInfo object from destination register and argument type.
    ArgInfo AInfo(VRegs[Index], Arg.getType(), Index);
    setArgFlags(AInfo, Index + AttributeList::FirstArgIndex, DL, F);

    // Handle any required merging from split value types from physical
    // registers into the desired VReg. ArgInfo objects are constructed
    // correspondingly and appended to SplitArgInfos.
    splitToValueTypes(AInfo, SplitArgInfos, DL, CC);

    ++Index;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 498-513: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  IncomingValueAssigner Assigner(CC_RISCV);
  RISCVFormalArgHandler Handler(MIRBuilder, MF.getRegInfo());

  SmallVector<CCValAssign, 16> ArgLocs;
  CCState CCInfo(CC, F.isVarArg(), MIRBuilder.getMF(), ArgLocs, F.getContext());
  if (!determineAssignments(Assigner, SplitArgInfos, CCInfo) ||
      !handleAssignments(Handler, SplitArgInfos, CCInfo, ArgLocs, MIRBuilder))
    return false;

  if (any_of(ArgLocs,
             [](CCValAssign &VA) { return VA.getLocVT().isScalableVector(); }))
    MF.getInfo<RISCVMachineFunctionInfo>()->setIsVectorCall();

  if (F.isVarArg())
    saveVarArgRegisters(MIRBuilder, Handler, Assigner, CCInfo);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 514-530: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  return true;
}

bool RISCVCallLowering::lowerCall(MachineIRBuilder &MIRBuilder,
                                  CallLoweringInfo &Info) const {
  MachineFunction &MF = MIRBuilder.getMF();
  const DataLayout &DL = MF.getDataLayout();
  CallingConv::ID CC = Info.CallConv;

  const RISCVSubtarget &Subtarget =
      MIRBuilder.getMF().getSubtarget<RISCVSubtarget>();
  for (auto &AInfo : Info.OrigArgs) {
    if (!isSupportedArgumentType(AInfo.Ty, Subtarget))
      return false;
    if (AInfo.Flags[0].isByVal())
      return false;
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 531-545: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp

  if (!Info.OrigRet.Ty->isVoidTy() &&
      !isSupportedReturnType(Info.OrigRet.Ty, Subtarget))
    return false;

  MachineInstrBuilder CallSeqStart =
      MIRBuilder.buildInstr(RISCV::ADJCALLSTACKDOWN);

  SmallVector<ArgInfo, 32> SplitArgInfos;
  for (auto &AInfo : Info.OrigArgs) {
    // Handle any required unmerging of split value types from a given VReg into
    // physical registers. ArgInfo objects are constructed correspondingly and
    // appended to SplitArgInfos.
    splitToValueTypes(AInfo, SplitArgInfos, DL, CC);
  }
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 546-561: Definitions and supporting logic / 定义与支撑逻辑
```cpp

  // TODO: Support tail calls.
  Info.IsTailCall = false;

  // Select the recommended relocation type R_RISCV_CALL_PLT.
  if (!Info.Callee.isReg())
    Info.Callee.setTargetFlags(RISCVII::MO_CALL);

  MachineInstrBuilder Call =
      MIRBuilder
          .buildInstrNoInsert(Info.Callee.isReg() ? RISCV::PseudoCALLIndirect
                                                  : RISCV::PseudoCALL)
          .add(Info.Callee);
  const TargetRegisterInfo *TRI = Subtarget.getRegisterInfo();
  Call.addRegMask(TRI->getCallPreservedMask(MF, Info.CallConv));
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 562-583: Type declaration for matching / matching 的类型声明
```cpp
  OutgoingValueAssigner ArgAssigner(CC_RISCV);
  RISCVOutgoingValueHandler ArgHandler(MIRBuilder, MF.getRegInfo(), Call);
  if (!determineAndHandleAssignments(ArgHandler, ArgAssigner, SplitArgInfos,
                                     MIRBuilder, CC, Info.IsVarArg))
    return false;

  MIRBuilder.insertInstr(Call);

  CallSeqStart.addImm(ArgAssigner.StackSize).addImm(0);
  MIRBuilder.buildInstr(RISCV::ADJCALLSTACKUP)
      .addImm(ArgAssigner.StackSize)
      .addImm(0);

  // If Callee is a reg, since it is used by a target specific
  // instruction, it must have a register class matching the
  // constraint of that instruction.
  if (Call->getOperand(0).isReg())
    constrainOperandRegClass(MF, *TRI, MF.getRegInfo(),
                             *Subtarget.getInstrInfo(),
                             *Subtarget.getRegBankInfo(), *Call,
                             Call->getDesc(), Call->getOperand(0), 0);
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 584-598: Control flow and target-specific decision logic / 控制流与目标相关判定逻辑
```cpp
  if (Info.CanLowerReturn && !Info.OrigRet.Ty->isVoidTy()) {
    SmallVector<ArgInfo, 4> SplitRetInfos;
    splitToValueTypes(Info.OrigRet, SplitRetInfos, DL, CC);

    IncomingValueAssigner RetAssigner(RetCC_RISCV);
    RISCVCallReturnHandler RetHandler(MIRBuilder, MF.getRegInfo(), Call);
    if (!determineAndHandleAssignments(RetHandler, RetAssigner, SplitRetInfos,
                                       MIRBuilder, CC, Info.IsVarArg))
      return false;
  }

  if (!Info.CanLowerReturn)
    insertSRetLoads(MIRBuilder, Info.OrigRet.Ty, Info.OrigRet.Regs,
                    Info.DemoteRegister, Info.DemoteStackIndex);
```
**EN:** This block performs validation, dispatch, or transformation decisions based on opcodes, operands, features, or subtarget state.

**CN:** 该区段依据操作码、操作数、特性或子目标状态执行校验、分派或转换决策。

### Lines 599-600: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  return true;
}
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

## Key Concepts / 关键概念
- **GlobalISel pipeline** / **GlobalISel 流水线**
- **Calling-convention lowering** / **调用约定下降**

## Dependencies / 依赖关系
- `RISCVCallLowering.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVCallingConv.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVISelLowering.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVMachineFunctionInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `RISCVSubtarget.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/Analysis.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/FunctionLoweringInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/GlobalISel/MachineIRBuilder.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/CodeGen/MachineFrameInfo.h` — Directly referenced by this file. / 该文件直接引用的依赖。
