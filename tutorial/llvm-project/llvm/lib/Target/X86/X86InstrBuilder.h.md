# X86InstrBuilder.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/X86/X86InstrBuilder.h`
- **Repository**: llvm/llvm-project
- **Purpose**: Declares interfaces and shared data structures for instruction definitions and metadata in the core X86 backend. / 为X86 后端核心中的指令定义与元数据声明接口与共享数据结构。

## Line-by-Line Analysis / 逐行分析
### Lines 1-30: File header, licensing, and opening context / 文件头、许可证与开场上下文
```cpp
//===-- X86InstrBuilder.h - Functions to aid building x86 insts -*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file exposes functions that may be used with BuildMI from the
// MachineInstrBuilder.h file to handle X86'isms in a clean way.
//
// The BuildMem function may be used with the BuildMI function to add entire
// memory references in a single, typed, function call.  X86 memory references
// can be very complex expressions (described in the README), so wrapping them
// up behind an easier to use interface makes sense.  Descriptions of the
// functions are included below.
//
// For reference, the order of operands for memory references is:
// (Operand), Base, Scale, Index, Displacement.
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_X86_X86INSTRBUILDER_H
#define LLVM_LIB_TARGET_X86_X86INSTRBUILDER_H

#include "llvm/ADT/SmallVector.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstr.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 31-60: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp
#include "llvm/CodeGen/MachineMemOperand.h"
#include "llvm/CodeGen/MachineOperand.h"
#include "llvm/MC/MCInstrDesc.h"
#include <cassert>

namespace llvm {

/// X86AddressMode - This struct holds a generalized full x86 address mode.
/// The base register can be a frame index, which will eventually be replaced
/// with BP or SP and Disp being offsetted accordingly.  The displacement may
/// also include the offset of a global value.
struct X86AddressMode {
  enum { RegBase, FrameIndexBase } BaseType = RegBase;

  union BaseUnion {
    Register Reg;
    int FrameIndex;

    BaseUnion() : Reg() {}
  } Base;

  unsigned Scale = 1;
  Register IndexReg;
  int Disp = 0;
  const GlobalValue *GV = nullptr;
  unsigned GVOpFlags = 0;
  bool CP = false;

  void getFullAddress(SmallVectorImpl<MachineOperand> &MO) {
    assert(Scale == 1 || Scale == 2 || Scale == 4 || Scale == 8);
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. Key symbols include X86AddressMode. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。关键符号包括 X86AddressMode。这些内容定义了实现文件所依赖的契约。

### Lines 61-90: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp

    if (BaseType == X86AddressMode::RegBase)
      MO.push_back(MachineOperand::CreateReg(Base.Reg, false, false, false,
                                             false, false, false, 0, false));
    else {
      assert(BaseType == X86AddressMode::FrameIndexBase);
      MO.push_back(MachineOperand::CreateFI(Base.FrameIndex));
    }

    MO.push_back(MachineOperand::CreateImm(Scale));
    MO.push_back(MachineOperand::CreateReg(IndexReg, false, false, false, false,
                                           false, false, 0, false));

    if (GV)
      MO.push_back(MachineOperand::CreateGA(GV, Disp, GVOpFlags));
    else
      MO.push_back(MachineOperand::CreateImm(Disp));

    MO.push_back(MachineOperand::CreateReg(0, false, false, false, false, false,
                                           false, 0, false));
  }
};

/// Compute the addressing mode from an machine instruction starting with the
/// given operand.
static inline X86AddressMode getAddressFromInstr(const MachineInstr *MI,
                                                 unsigned Operand) {
  X86AddressMode AM;
  const MachineOperand &Op0 = MI->getOperand(Operand);
  if (Op0.isReg()) {
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 91-120: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
    AM.BaseType = X86AddressMode::RegBase;
    AM.Base.Reg = Op0.getReg();
  } else {
    AM.BaseType = X86AddressMode::FrameIndexBase;
    AM.Base.FrameIndex = Op0.getIndex();
  }

  const MachineOperand &Op1 = MI->getOperand(Operand + 1);
  AM.Scale = Op1.getImm();

  const MachineOperand &Op2 = MI->getOperand(Operand + 2);
  AM.IndexReg = Op2.getReg();

  const MachineOperand &Op3 = MI->getOperand(Operand + 3);
  if (Op3.isGlobal())
    AM.GV = Op3.getGlobal();
  else
    AM.Disp = Op3.getImm();

  return AM;
}

/// addDirectMem - This function is used to add a direct memory reference to the
/// current instruction -- that is, a dereference of an address in a register,
/// with no scale, index or displacement. An example is: DWORD PTR [EAX].
///
static inline const MachineInstrBuilder &
addDirectMem(const MachineInstrBuilder &MIB, Register Reg) {
  // Because memory references are always represented with five
  // values, this adds: Reg, 1, NoReg, 0, NoReg to the instruction.
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 121-150: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  return MIB.addReg(Reg).addImm(1).addReg(0).addImm(0).addReg(0);
}

/// Replace the address used in the instruction with the direct memory
/// reference.
static inline void setDirectAddressInInstr(MachineInstr *MI, unsigned Operand,
                                           Register Reg) {
  // Direct memory address is in a form of: Reg/FI, 1 (Scale), NoReg, 0, NoReg.
  MI->getOperand(Operand).ChangeToRegister(Reg, /*isDef=*/false);
  MI->getOperand(Operand + 1).setImm(1);
  MI->getOperand(Operand + 2).setReg(0);
  MI->getOperand(Operand + 3).ChangeToImmediate(0);
  MI->getOperand(Operand + 4).setReg(0);
}

static inline const MachineInstrBuilder &
addOffset(const MachineInstrBuilder &MIB, int Offset) {
  return MIB.addImm(1).addReg(0).addImm(Offset).addReg(0);
}

static inline const MachineInstrBuilder &
addOffset(const MachineInstrBuilder &MIB, const MachineOperand& Offset) {
  return MIB.addImm(1).addReg(0).add(Offset).addReg(0);
}

/// addRegOffset - This function is used to add a memory reference of the form
/// [Reg + Offset], i.e., one with no scale or index, but with a
/// displacement. An example is: DWORD PTR [EAX + 4].
///
static inline const MachineInstrBuilder &
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 151-180: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
addRegOffset(const MachineInstrBuilder &MIB, Register Reg, bool isKill,
             int Offset) {
  return addOffset(MIB.addReg(Reg, getKillRegState(isKill)), Offset);
}

/// addRegReg - This function is used to add a memory reference of the form:
/// [Reg + Reg].
static inline const MachineInstrBuilder &
addRegReg(const MachineInstrBuilder &MIB, Register Reg1, bool isKill1,
          unsigned SubReg1, Register Reg2, bool isKill2, unsigned SubReg2) {
  return MIB.addReg(Reg1, getKillRegState(isKill1), SubReg1)
      .addImm(1)
      .addReg(Reg2, getKillRegState(isKill2), SubReg2)
      .addImm(0)
      .addReg(0);
}

static inline const MachineInstrBuilder &
addFullAddress(const MachineInstrBuilder &MIB,
               const X86AddressMode &AM) {
  assert(AM.Scale == 1 || AM.Scale == 2 || AM.Scale == 4 || AM.Scale == 8);

  if (AM.BaseType == X86AddressMode::RegBase)
    MIB.addReg(AM.Base.Reg);
  else {
    assert(AM.BaseType == X86AddressMode::FrameIndexBase);
    MIB.addFrameIndex(AM.Base.FrameIndex);
  }

  MIB.addImm(AM.Scale).addReg(AM.IndexReg);
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 181-210: Function implementation and target-specific behavior / 函数实现与目标特定行为
```cpp
  if (AM.GV)
    MIB.addGlobalAddress(AM.GV, AM.Disp, AM.GVOpFlags);
  else
    MIB.addImm(AM.Disp);

  return MIB.addReg(0);
}

/// addFrameReference - This function is used to add a reference to the base of
/// an abstract object on the stack frame of the current function.  This
/// reference has base register as the FrameIndex offset until it is resolved.
/// This allows a constant offset to be specified as well...
///
static inline const MachineInstrBuilder &
addFrameReference(const MachineInstrBuilder &MIB, int FI, int Offset = 0) {
  MachineInstr *MI = MIB;
  MachineFunction &MF = *MI->getParent()->getParent();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  const MCInstrDesc &MCID = MI->getDesc();
  auto Flags = MachineMemOperand::MONone;
  if (MCID.mayLoad())
    Flags |= MachineMemOperand::MOLoad;
  if (MCID.mayStore())
    Flags |= MachineMemOperand::MOStore;
  MachineMemOperand *MMO = MF.getMachineMemOperand(
      MachinePointerInfo::getFixedStack(MF, FI, Offset), Flags,
      MFI.getObjectSize(FI), MFI.getObjectAlign(FI));
  return addOffset(MIB.addFrameIndex(FI), Offset)
            .addMemOperand(MMO);
}
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

### Lines 211-229: Preprocessor directives and structural setup / 预处理指令与结构初始化
```cpp

/// addConstantPoolReference - This function is used to add a reference to the
/// base of a constant value spilled to the per-function constant pool.  The
/// reference uses the abstract ConstantPoolIndex which is retained until
/// either machine code emission or assembly output. In PIC mode on x86-32,
/// the GlobalBaseReg parameter can be used to make this a
/// GlobalBaseReg-relative reference.
///
static inline const MachineInstrBuilder &
addConstantPoolReference(const MachineInstrBuilder &MIB, unsigned CPI,
                         Register GlobalBaseReg, unsigned char OpFlags) {
  //FIXME: factor this
  return MIB.addReg(GlobalBaseReg).addImm(1).addReg(0)
    .addConstantPoolIndex(CPI, 0, OpFlags).addReg(0);
}

} // end namespace llvm

#endif // LLVM_LIB_TARGET_X86_X86INSTRBUILDER_H
```
**EN:** This section declares APIs, helper types, or inline logic related to the core X86 backend. It defines contracts that implementation files depend on.

**CN:** 本段声明与X86 后端核心相关的 API、辅助类型或内联逻辑。这些内容定义了实现文件所依赖的契约。

## Key Concepts / 关键概念
- Primary topic: instruction definitions and metadata. / 核心主题：指令定义与元数据。
- Subsystem: the core X86 backend. / 所属子系统：X86 后端核心。
- Notable symbols: X86AddressMode. / 重要符号：X86AddressMode。
- The file emphasizes declarations, interfaces, and reusable helpers. / 该文件以声明、接口和可复用辅助逻辑为主。
- The implementation follows LLVM namespace conventions. / 实现遵循 LLVM 的命名空间约定。

## Dependencies / 依赖关系
- Direct includes: llvm/ADT/SmallVector.h, llvm/CodeGen/MachineFrameInfo.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineMemOperand.h, llvm/CodeGen/MachineOperand.h, llvm/MC/MCInstrDesc.h, cassert. / 直接包含：llvm/ADT/SmallVector.h, llvm/CodeGen/MachineFrameInfo.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineInstrBuilder.h, llvm/CodeGen/MachineMemOperand.h, llvm/CodeGen/MachineOperand.h, llvm/MC/MCInstrDesc.h, cassert。
- Integrates with shared LLVM backend layers such as IR, CodeGen, MC, and X86 target support as needed. / 按需与 LLVM 的 IR、CodeGen、MC 和 X86 目标支持层集成。
- Implementation files in the same subsystem rely on these declarations. / 同一子系统中的实现文件依赖这些声明。
