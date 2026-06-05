# M68kCallLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/GISel/M68kCallLowering.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file performs target-specific lowering for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责执行目标相关的降低。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- M68kCallLowering.cpp - Call lowering --------------------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: /// \file
  10: /// This file implements the lowering of LLVM calls to machine code calls for
  11: /// GlobalISel.
  12: //
  13: //===----------------------------------------------------------------------===//
  14: 
  15: #include "M68kCallLowering.h"
  16: #include "M68kISelLowering.h"
  17: #include "M68kInstrInfo.h"
  18: #include "M68kSubtarget.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68kCallLowering.h`, `M68kISelLowering.h`, `M68kInstrInfo.h`, `M68kSubtarget.h` that expose the LLVM and target interfaces used in later logic.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68kCallLowering.h`, `M68kISelLowering.h`, `M68kInstrInfo.h`, `M68kSubtarget.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 19-36 / 第 19-36 行
```cpp
  19: #include "M68kTargetMachine.h"
  20: #include "llvm/CodeGen/CallingConvLower.h"
  21: #include "llvm/CodeGen/GlobalISel/CallLowering.h"
  22: #include "llvm/CodeGen/GlobalISel/MachineIRBuilder.h"
  23: #include "llvm/CodeGen/MachineFrameInfo.h"
  24: #include "llvm/CodeGen/TargetCallingConv.h"
  25: 
  26: using namespace llvm;
  27: 
  28: namespace {
  29: 
  30: struct M68kFormalArgHandler : public M68kIncomingValueHandler {
  31:   M68kFormalArgHandler(MachineIRBuilder &MIRBuilder, MachineRegisterInfo &MRI)
  32:       : M68kIncomingValueHandler(MIRBuilder, MRI) {}
  33: };
  34: 
  35: struct CallReturnHandler : public M68kIncomingValueHandler {
  36:   CallReturnHandler(MachineIRBuilder &MIRBuilder, MachineRegisterInfo &MRI,
```
- **EN**: It imports dependencies such as `M68kTargetMachine.h`, `CallingConvLower.h`, `CallLowering.h`, `MachineIRBuilder.h`, `MachineFrameInfo.h`, `TargetCallingConv.h` that expose the LLVM and target interfaces used in later logic. It introduces interface types such as `M68kFormalArgHandler`, `CallReturnHandler`, shaping how other backend components interact with this file. The range implements or declares functions including `M68kFormalArgHandler`.
- **CN**: 它引入了 `M68kTargetMachine.h`, `CallingConvLower.h`, `CallLowering.h`, `MachineIRBuilder.h`, `MachineFrameInfo.h`, `TargetCallingConv.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 它引入了 `M68kFormalArgHandler`, `CallReturnHandler` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `M68kFormalArgHandler` 等函数。

### Lines 37-54 / 第 37-54 行
```cpp
  37:                     MachineInstrBuilder &MIB)
  38:       : M68kIncomingValueHandler(MIRBuilder, MRI), MIB(MIB) {}
  39: 
  40: private:
  41:   void assignValueToReg(Register ValVReg, Register PhysReg,
  42:                         const CCValAssign &VA,
  43:                         ISD::ArgFlagsTy Flags = {}) override;
  44: 
  45:   MachineInstrBuilder &MIB;
  46: };
  47: 
  48: } // end anonymous namespace
  49: 
  50: M68kCallLowering::M68kCallLowering(const M68kTargetLowering &TLI)
  51:     : CallLowering(&TLI) {}
  52: 
  53: struct M68kOutgoingArgHandler : public CallLowering::OutgoingValueHandler {
  54:   M68kOutgoingArgHandler(MachineIRBuilder &MIRBuilder, MachineRegisterInfo &MRI,
```
- **EN**: It introduces interface types such as `M68kOutgoingArgHandler`, shaping how other backend components interact with this file. The range implements or declares functions including `M68kIncomingValueHandler`, `CallLowering`.
- **CN**: 它引入了 `M68kOutgoingArgHandler` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `M68kIncomingValueHandler`, `CallLowering` 等函数。

### Lines 55-72 / 第 55-72 行
```cpp
  55:                          MachineInstrBuilder MIB)
  56:       : OutgoingValueHandler(MIRBuilder, MRI), MIB(MIB),
  57:         DL(MIRBuilder.getMF().getDataLayout()),
  58:         STI(MIRBuilder.getMF().getSubtarget<M68kSubtarget>()) {}
  59: 
  60:   void assignValueToReg(Register ValVReg, Register PhysReg,
  61:                         const CCValAssign &VA,
  62:                         ISD::ArgFlagsTy Flags = {}) override {
  63:     MIB.addUse(PhysReg, RegState::Implicit);
  64:     Register ExtReg = extendRegister(ValVReg, VA);
  65:     MIRBuilder.buildCopy(PhysReg, ExtReg);
  66:   }
  67: 
  68:   void assignValueToAddress(Register ValVReg, Register Addr, LLT MemTy,
  69:                             const MachinePointerInfo &MPO,
  70:                             const CCValAssign &VA) override {
  71:     MachineFunction &MF = MIRBuilder.getMF();
  72:     Register ExtReg = extendRegister(ValVReg, VA);
```
- **EN**: The range implements or declares functions including `OutgoingValueHandler`.
- **CN**: 这一段实现或声明了 `OutgoingValueHandler` 等函数。

### Lines 73-90 / 第 73-90 行
```cpp
  73: 
  74:     auto *MMO = MF.getMachineMemOperand(MPO, MachineMemOperand::MOStore, MemTy,
  75:                                         inferAlignFromPtrInfo(MF, MPO));
  76:     MIRBuilder.buildStore(ExtReg, Addr, *MMO);
  77:   }
  78: 
  79:   Register getStackAddress(uint64_t Size, int64_t Offset,
  80:                            MachinePointerInfo &MPO,
  81:                            ISD::ArgFlagsTy Flags) override {
  82:     LLT p0 = LLT::pointer(0, DL.getPointerSizeInBits(0));
  83:     LLT SType = LLT::scalar(DL.getPointerSizeInBits(0));
  84:     Register StackReg = STI.getRegisterInfo()->getStackRegister();
  85:     auto SPReg = MIRBuilder.buildCopy(p0, StackReg).getReg(0);
  86:     auto OffsetReg = MIRBuilder.buildConstant(SType, Offset);
  87:     auto AddrReg = MIRBuilder.buildPtrAdd(p0, SPReg, OffsetReg);
  88:     MPO = MachinePointerInfo::getStack(MIRBuilder.getMF(), Offset);
  89:     return AddrReg.getReg(0);
  90:   }
```
- **EN**: This span continues the file's main responsibility: this file performs target-specific lowering for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 91-108 / 第 91-108 行
```cpp
  91:   MachineInstrBuilder MIB;
  92:   const DataLayout &DL;
  93:   const M68kSubtarget &STI;
  94: };
  95: bool M68kCallLowering::lowerReturn(MachineIRBuilder &MIRBuilder,
  96:                                    const Value *Val, ArrayRef<Register> VRegs,
  97:                                    FunctionLoweringInfo &FLI,
  98:                                    Register SwiftErrorVReg) const {
  99: 
 100:   auto MIB = MIRBuilder.buildInstrNoInsert(M68k::RTS);
 101:   bool Success = true;
 102:   MachineFunction &MF = MIRBuilder.getMF();
 103:   const Function &F = MF.getFunction();
 104:   MachineRegisterInfo &MRI = MF.getRegInfo();
 105:   const M68kTargetLowering &TLI = *getTLI<M68kTargetLowering>();
 106:   CCAssignFn *AssignFn =
 107:       TLI.getCCAssignFn(F.getCallingConv(), true, F.isVarArg());
 108:   auto &DL = F.getDataLayout();
```
- **EN**: The range implements or declares functions including `M68kCallLowering::lowerReturn`.
- **CN**: 这一段实现或声明了 `M68kCallLowering::lowerReturn` 等函数。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   if (!VRegs.empty()) {
 110:     SmallVector<ArgInfo, 8> SplitArgs;
 111:     ArgInfo OrigArg{VRegs, Val->getType(), 0};
 112:     setArgFlags(OrigArg, AttributeList::ReturnIndex, DL, F);
 113:     splitToValueTypes(OrigArg, SplitArgs, DL, F.getCallingConv());
 114:     OutgoingValueAssigner ArgAssigner(AssignFn);
 115:     M68kOutgoingArgHandler ArgHandler(MIRBuilder, MRI, MIB);
 116:     Success = determineAndHandleAssignments(ArgHandler, ArgAssigner, SplitArgs,
 117:                                             MIRBuilder, F.getCallingConv(),
 118:                                             F.isVarArg());
 119:   }
 120:   MIRBuilder.insertInstr(MIB);
 121:   return Success;
 122: }
 123: 
 124: bool M68kCallLowering::lowerFormalArguments(MachineIRBuilder &MIRBuilder,
 125:                                             const Function &F,
 126:                                             ArrayRef<ArrayRef<Register>> VRegs,
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 127-144 / 第 127-144 行
```cpp
 127:                                             FunctionLoweringInfo &FLI) const {
 128:   MachineFunction &MF = MIRBuilder.getMF();
 129:   MachineRegisterInfo &MRI = MF.getRegInfo();
 130:   const auto &DL = F.getDataLayout();
 131:   auto &TLI = *getTLI<M68kTargetLowering>();
 132: 
 133:   SmallVector<ArgInfo, 8> SplitArgs;
 134:   unsigned I = 0;
 135:   for (const auto &Arg : F.args()) {
 136:     ArgInfo OrigArg{VRegs[I], Arg.getType(), I};
 137:     setArgFlags(OrigArg, I + AttributeList::FirstArgIndex, DL, F);
 138:     splitToValueTypes(OrigArg, SplitArgs, DL, F.getCallingConv());
 139:     ++I;
 140:   }
 141: 
 142:   CCAssignFn *AssignFn =
 143:       TLI.getCCAssignFn(F.getCallingConv(), false, F.isVarArg());
 144:   IncomingValueAssigner ArgAssigner(AssignFn);
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 145-162 / 第 145-162 行
```cpp
 145:   M68kFormalArgHandler ArgHandler(MIRBuilder, MRI);
 146:   return determineAndHandleAssignments(ArgHandler, ArgAssigner, SplitArgs,
 147:                                        MIRBuilder, F.getCallingConv(),
 148:                                        F.isVarArg());
 149: }
 150: 
 151: void M68kIncomingValueHandler::assignValueToReg(Register ValVReg,
 152:                                                 Register PhysReg,
 153:                                                 const CCValAssign &VA,
 154:                                                 ISD::ArgFlagsTy Flags) {
 155:   MIRBuilder.getMRI()->addLiveIn(PhysReg);
 156:   MIRBuilder.getMBB().addLiveIn(PhysReg);
 157:   IncomingValueHandler::assignValueToReg(ValVReg, PhysReg, VA);
 158: }
 159: 
 160: void M68kIncomingValueHandler::assignValueToAddress(
 161:     Register ValVReg, Register Addr, LLT MemTy, const MachinePointerInfo &MPO,
 162:     const CCValAssign &VA) {
```
- **EN**: The range implements or declares functions including `M68kIncomingValueHandler::assignValueToReg`, `M68kIncomingValueHandler::assignValueToAddress`.
- **CN**: 这一段实现或声明了 `M68kIncomingValueHandler::assignValueToReg`, `M68kIncomingValueHandler::assignValueToAddress` 等函数。

### Lines 163-180 / 第 163-180 行
```cpp
 163:   MachineFunction &MF = MIRBuilder.getMF();
 164:   auto *MMO = MF.getMachineMemOperand(MPO, MachineMemOperand::MOLoad, MemTy,
 165:                                       inferAlignFromPtrInfo(MF, MPO));
 166:   MIRBuilder.buildLoad(ValVReg, Addr, *MMO);
 167: }
 168: 
 169: Register M68kIncomingValueHandler::getStackAddress(uint64_t Size,
 170:                                                    int64_t Offset,
 171:                                                    MachinePointerInfo &MPO,
 172:                                                    ISD::ArgFlagsTy Flags) {
 173:   auto &MFI = MIRBuilder.getMF().getFrameInfo();
 174:   const bool IsImmutable = !Flags.isByVal();
 175:   int FI = MFI.CreateFixedObject(Size, Offset, IsImmutable);
 176:   MPO = MachinePointerInfo::getFixedStack(MIRBuilder.getMF(), FI);
 177: 
 178:   // Build Frame Index
 179:   llvm::LLT FramePtr = LLT::pointer(
 180:       0, MIRBuilder.getMF().getDataLayout().getPointerSizeInBits());
```
- **EN**: The range implements or declares functions including `M68kIncomingValueHandler::getStackAddress`.
- **CN**: 这一段实现或声明了 `M68kIncomingValueHandler::getStackAddress` 等函数。

### Lines 181-198 / 第 181-198 行
```cpp
 181:   MachineInstrBuilder AddrReg = MIRBuilder.buildFrameIndex(FramePtr, FI);
 182:   StackUsed = std::max(StackUsed, Size + Offset);
 183:   return AddrReg.getReg(0);
 184: }
 185: 
 186: void CallReturnHandler::assignValueToReg(Register ValVReg, Register PhysReg,
 187:                                          const CCValAssign &VA,
 188:                                          ISD::ArgFlagsTy Flags) {
 189:   MIB.addDef(PhysReg, RegState::Implicit);
 190:   MIRBuilder.buildCopy(ValVReg, PhysReg);
 191: }
 192: 
 193: bool M68kCallLowering::lowerCall(MachineIRBuilder &MIRBuilder,
 194:                                  CallLoweringInfo &Info) const {
 195:   MachineFunction &MF = MIRBuilder.getMF();
 196:   Function &F = MF.getFunction();
 197:   MachineRegisterInfo &MRI = MF.getRegInfo();
 198:   auto &DL = F.getDataLayout();
```
- **EN**: The range implements or declares functions including `CallReturnHandler::assignValueToReg`, `M68kCallLowering::lowerCall`.
- **CN**: 这一段实现或声明了 `CallReturnHandler::assignValueToReg`, `M68kCallLowering::lowerCall` 等函数。

### Lines 199-216 / 第 199-216 行
```cpp
 199:   const M68kTargetLowering &TLI = *getTLI<M68kTargetLowering>();
 200:   const M68kSubtarget &STI = MF.getSubtarget<M68kSubtarget>();
 201:   const TargetInstrInfo &TII = *STI.getInstrInfo();
 202:   const M68kRegisterInfo *TRI = STI.getRegisterInfo();
 203: 
 204:   SmallVector<ArgInfo, 8> OutArgs;
 205:   for (auto &OrigArg : Info.OrigArgs)
 206:     splitToValueTypes(OrigArg, OutArgs, DL, Info.CallConv);
 207: 
 208:   SmallVector<ArgInfo, 8> InArgs;
 209:   if (!Info.OrigRet.Ty->isVoidTy())
 210:     splitToValueTypes(Info.OrigRet, InArgs, DL, Info.CallConv);
 211: 
 212:   unsigned AdjStackDown = TII.getCallFrameSetupOpcode();
 213:   auto CallSeqStart = MIRBuilder.buildInstr(AdjStackDown);
 214: 
 215:   unsigned Opc = TLI.getTargetMachine().isPositionIndependent() ? M68k::CALLq
 216:                  : Info.Callee.isReg()                          ? M68k::CALLj
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 217-234 / 第 217-234 行
```cpp
 217:                                                                 : M68k::CALLb;
 218: 
 219:   auto MIB = MIRBuilder.buildInstrNoInsert(Opc)
 220:                  .add(Info.Callee)
 221:                  .addRegMask(TRI->getCallPreservedMask(MF, Info.CallConv));
 222: 
 223:   CCAssignFn *AssignFn = TLI.getCCAssignFn(Info.CallConv, false, Info.IsVarArg);
 224:   OutgoingValueAssigner Assigner(AssignFn);
 225:   M68kOutgoingArgHandler Handler(MIRBuilder, MRI, MIB);
 226:   if (!determineAndHandleAssignments(Handler, Assigner, OutArgs, MIRBuilder,
 227:                                      Info.CallConv, Info.IsVarArg))
 228:     return false;
 229: 
 230:   if (Info.Callee.isReg())
 231:     constrainOperandRegClass(MF, *TRI, MRI, *STI.getInstrInfo(),
 232:                              *STI.getRegBankInfo(), *MIB, MIB->getDesc(),
 233:                              Info.Callee, 0);
 234: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 235-252 / 第 235-252 行
```cpp
 235:   MIRBuilder.insertInstr(MIB);
 236: 
 237:   if (!Info.OrigRet.Ty->isVoidTy()) {
 238:     CCAssignFn *RetAssignFn =
 239:         TLI.getCCAssignFn(Info.CallConv, true, Info.IsVarArg);
 240: 
 241:     OutgoingValueAssigner Assigner(RetAssignFn, RetAssignFn);
 242:     CallReturnHandler Handler(MIRBuilder, MRI, MIB);
 243:     if (!determineAndHandleAssignments(Handler, Assigner, InArgs, MIRBuilder,
 244:                                        Info.CallConv, Info.IsVarArg))
 245:       return false;
 246:   }
 247: 
 248:   CallSeqStart.addImm(Assigner.StackSize).addImm(0);
 249: 
 250:   unsigned AdjStackUp = TII.getCallFrameDestroyOpcode();
 251:   MIRBuilder.buildInstr(AdjStackUp).addImm(Assigner.StackSize).addImm(0);
 252: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 253-256 / 第 253-256 行
```cpp
 253:   return true;
 254: }
 255: 
 256: bool M68kCallLowering::enableBigEndian() const { return true; }
```
- **EN**: The range implements or declares functions including `M68kCallLowering::enableBigEndian`.
- **CN**: 这一段实现或声明了 `M68kCallLowering::enableBigEndian` 等函数。

## Key Concepts / 关键概念
- **GlobalISel**: Uses legalization, register banks, and instruction selection after generic IR lowering. / 在通用 IR 降低后使用合法化、寄存器银行和指令选择。
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Register classes**: Groups registers by capability so instruction operands and allocators can constrain them. / 按能力对寄存器分组，以便指令操作数和分配器施加约束。

## Dependencies / 依赖关系
- `M68kCallLowering.h`
- `M68kISelLowering.h`
- `M68kInstrInfo.h`
- `M68kSubtarget.h`
- `M68kTargetMachine.h`
- `llvm/CodeGen/CallingConvLower.h`
- `llvm/CodeGen/GlobalISel/CallLowering.h`
- `llvm/CodeGen/GlobalISel/MachineIRBuilder.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/TargetCallingConv.h`
