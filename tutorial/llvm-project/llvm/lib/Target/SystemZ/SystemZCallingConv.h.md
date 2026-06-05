# SystemZCallingConv.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZCallingConv.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file defines calling convention rules and register/stack assignments for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责定义调用约定规则以及寄存器/栈分配。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- SystemZCallingConv.h - Calling conventions for SystemZ --*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: 
   9: #ifndef LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZCALLINGCONV_H
  10: #define LLVM_LIB_TARGET_SYSTEMZ_SYSTEMZCALLINGCONV_H
  11: 
  12: #include "SystemZSubtarget.h"
  13: #include "llvm/ADT/SmallVector.h"
  14: #include "llvm/CodeGen/CallingConvLower.h"
  15: #include "llvm/MC/MCRegisterInfo.h"
  16: 
  17: namespace llvm {
  18: namespace SystemZ {
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZSubtarget.h`, `SmallVector.h`, `CallingConvLower.h`, `MCRegisterInfo.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZSubtarget.h`, `SmallVector.h`, `CallingConvLower.h`, `MCRegisterInfo.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19:   const unsigned ELFNumArgGPRs = 5;
  20:   extern const MCPhysReg ELFArgGPRs[ELFNumArgGPRs];
  21: 
  22:   const unsigned ELFNumArgFPRs = 4;
  23:   extern const MCPhysReg ELFArgFPRs[ELFNumArgFPRs];
  24: 
  25:   const unsigned XPLINK64NumArgGPRs = 3;
  26:   extern const MCPhysReg XPLINK64ArgGPRs[XPLINK64NumArgGPRs];
  27: 
  28:   const unsigned XPLINK64NumArgFPRs = 4;
  29:   extern const MCPhysReg XPLINK64ArgFPRs[XPLINK64NumArgFPRs];
  30: } // end namespace SystemZ
  31: 
  32: // Handle i128 argument types.  These need to be passed by implicit
  33: // reference.  This could be as simple as the following .td line:
  34: //    CCIfType<[i128], CCPassIndirect<i64>>,
  35: // except that i128 is not a legal type, and therefore gets split by
  36: // common code into a pair of i64 arguments.
```
- **EN**: This span continues the file's main responsibility: this file defines calling convention rules and register/stack assignments for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 37-54 / 第 37-54 行
```cpp
  37: inline bool CC_SystemZ_I128Indirect(unsigned &ValNo, MVT &ValVT,
  38:                                     MVT &LocVT,
  39:                                     CCValAssign::LocInfo &LocInfo,
  40:                                     ISD::ArgFlagsTy &ArgFlags,
  41:                                     CCState &State) {
  42:   SmallVectorImpl<CCValAssign> &PendingMembers = State.getPendingLocs();
  43: 
  44:   // ArgFlags.isSplit() is true on the first part of a i128 argument;
  45:   // PendingMembers.empty() is false on all subsequent parts.
  46:   if (!ArgFlags.isSplit() && PendingMembers.empty())
  47:     return false;
  48: 
  49:   // Push a pending Indirect value location for each part.
  50:   LocVT = MVT::i64;
  51:   LocInfo = CCValAssign::Indirect;
  52:   PendingMembers.push_back(CCValAssign::getPending(ValNo, ValVT,
  53:                                                    LocVT, LocInfo));
  54:   if (!ArgFlags.isSplitEnd())
```
- **EN**: The range implements or declares functions including `CC_SystemZ_I128Indirect`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `CC_SystemZ_I128Indirect` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 55-72 / 第 55-72 行
```cpp
  55:     return true;
  56: 
  57:   // OK, we've collected all parts in the pending list.  Allocate
  58:   // the location (register or stack slot) for the indirect pointer.
  59:   // (This duplicates the usual i64 calling convention rules.)
  60:   unsigned Reg;
  61:   const SystemZSubtarget &Subtarget =
  62:       State.getMachineFunction().getSubtarget<SystemZSubtarget>();
  63:   if (Subtarget.isTargetELF())
  64:     Reg = State.AllocateReg(SystemZ::ELFArgGPRs);
  65:   else if (Subtarget.isTargetXPLINK64())
  66:     Reg = State.AllocateReg(SystemZ::XPLINK64ArgGPRs);
  67:   else
  68:     llvm_unreachable("Unknown Calling Convention!");
  69: 
  70:   unsigned Offset = Reg && !Subtarget.isTargetXPLINK64()
  71:                         ? 0
  72:                         : State.AllocateStack(8, Align(8));
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 73-90 / 第 73-90 行
```cpp
  73: 
  74:   // Use that same location for all the pending parts.
  75:   for (auto &It : PendingMembers) {
  76:     if (Reg)
  77:       It.convertToReg(Reg);
  78:     else
  79:       It.convertToMem(Offset);
  80:     State.addLoc(It);
  81:   }
  82: 
  83:   PendingMembers.clear();
  84: 
  85:   return true;
  86: }
  87: 
  88: // A pointer in 64bit mode is always passed as 64bit.
  89: inline bool CC_XPLINK64_Pointer(unsigned &ValNo, MVT &ValVT, MVT &LocVT,
  90:                                 CCValAssign::LocInfo &LocInfo,
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 91-108 / 第 91-108 行
```cpp
  91:                                 ISD::ArgFlagsTy &ArgFlags, CCState &State) {
  92:   if (LocVT != MVT::i64) {
  93:     LocVT = MVT::i64;
  94:     LocInfo = CCValAssign::ZExt;
  95:   }
  96:   return false;
  97: }
  98: 
  99: inline bool CC_XPLINK64_Shadow_Reg(unsigned &ValNo, MVT &ValVT, MVT &LocVT,
 100:                                    CCValAssign::LocInfo &LocInfo,
 101:                                    ISD::ArgFlagsTy &ArgFlags, CCState &State) {
 102:   if (LocVT == MVT::f32 || LocVT == MVT::f64) {
 103:     State.AllocateReg(SystemZ::XPLINK64ArgGPRs);
 104:   }
 105:   if (LocVT == MVT::f128 || LocVT.is128BitVector()) {
 106:     // Shadow next two GPRs, if available.
 107:     State.AllocateReg(SystemZ::XPLINK64ArgGPRs);
 108:     State.AllocateReg(SystemZ::XPLINK64ArgGPRs);
```
- **EN**: The range implements or declares functions including `CC_XPLINK64_Shadow_Reg`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `CC_XPLINK64_Shadow_Reg` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 109-126 / 第 109-126 行
```cpp
 109: 
 110:     // Quad precision floating point needs to
 111:     // go inside pre-defined FPR pair.
 112:     if (LocVT == MVT::f128) {
 113:       for (unsigned I = 0; I < SystemZ::XPLINK64NumArgFPRs; I += 2)
 114:         if (State.isAllocated(SystemZ::XPLINK64ArgFPRs[I]))
 115:           State.AllocateReg(SystemZ::XPLINK64ArgFPRs[I + 1]);
 116:     }
 117:   }
 118:   return false;
 119: }
 120: 
 121: inline bool CC_XPLINK64_Allocate128BitVararg(unsigned &ValNo, MVT &ValVT,
 122:                                              MVT &LocVT,
 123:                                              CCValAssign::LocInfo &LocInfo,
 124:                                              ISD::ArgFlagsTy &ArgFlags,
 125:                                              CCState &State) {
 126:   // For any C or C++ program, this should always be
```
- **EN**: The range implements or declares functions including `CC_XPLINK64_Allocate128BitVararg`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `CC_XPLINK64_Allocate128BitVararg` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 127-144 / 第 127-144 行
```cpp
 127:   // false, since it is illegal to have a function
 128:   // where the first argument is variadic. Therefore
 129:   // the first fixed argument should already have
 130:   // allocated GPR1 either through shadowing it or
 131:   // using it for parameter passing.
 132:   State.AllocateReg(SystemZ::R1D);
 133: 
 134:   bool AllocGPR2 = State.AllocateReg(SystemZ::R2D);
 135:   bool AllocGPR3 = State.AllocateReg(SystemZ::R3D);
 136: 
 137:   // If GPR2 and GPR3 are available, then we may pass vararg in R2Q.
 138:   // If only GPR3 is available, we need to set custom handling to copy
 139:   // hi bits into GPR3.
 140:   // Either way, we allocate on the stack.
 141:   if (AllocGPR3) {
 142:     // For f128 and vector var arg case, set the bitcast flag to bitcast to
 143:     // i128.
 144:     LocVT = MVT::i128;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 145-162 / 第 145-162 行
```cpp
 145:     LocInfo = CCValAssign::BCvt;
 146:     auto Offset = State.AllocateStack(16, Align(8));
 147:     if (AllocGPR2)
 148:       State.addLoc(
 149:           CCValAssign::getReg(ValNo, ValVT, SystemZ::R2Q, LocVT, LocInfo));
 150:     else
 151:       State.addLoc(
 152:           CCValAssign::getCustomMem(ValNo, ValVT, Offset, LocVT, LocInfo));
 153:     return true;
 154:   }
 155: 
 156:   return false;
 157: }
 158: 
 159: inline bool RetCC_SystemZ_Error(unsigned &, MVT &, MVT &,
 160:                                 CCValAssign::LocInfo &, ISD::ArgFlagsTy &,
 161:                                 CCState &) {
 162:   llvm_unreachable("Return value calling convention currently unsupported.");
```
- **EN**: The range implements or declares functions including `RetCC_SystemZ_Error`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `RetCC_SystemZ_Error` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 163-179 / 第 163-179 行
```cpp
 163: }
 164: 
 165: inline bool CC_SystemZ_Error(unsigned &, MVT &, MVT &, CCValAssign::LocInfo &,
 166:                              ISD::ArgFlagsTy &, CCState &) {
 167:   llvm_unreachable("Argument calling convention currently unsupported.");
 168: }
 169: 
 170: inline bool CC_SystemZ_GHC_Error(unsigned &, MVT &, MVT &,
 171:                                  CCValAssign::LocInfo &, ISD::ArgFlagsTy &,
 172:                                  CCState &) {
 173:   report_fatal_error("No registers left in GHC calling convention");
 174:   return false;
 175: }
 176: 
 177: } // end namespace llvm
 178: 
 179: #endif
```
- **EN**: The range implements or declares functions including `CC_SystemZ_Error`, `CC_SystemZ_GHC_Error`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `CC_SystemZ_Error`, `CC_SystemZ_GHC_Error` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

## Key Concepts / 关键概念
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `SystemZSubtarget.h`
- `llvm/ADT/SmallVector.h`
- `llvm/CodeGen/CallingConvLower.h`
- `llvm/MC/MCRegisterInfo.h`
