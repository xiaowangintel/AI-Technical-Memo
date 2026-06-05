# M68kISelLowering.h — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kISelLowering.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements SelectionDAG lowering for the target backend for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责实现目标后端的 SelectionDAG 降低。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18 / 第 1-18 行
```cpp
   1: //===-- M68kISelLowering.h - M68k DAG Lowering Interface --------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: ///
   9: /// \file
  10: /// This file defines the interfaces that M68k uses to lower LLVM code into a
  11: /// selection DAG.
  12: ///
  13: //===----------------------------------------------------------------------===//
  14: 
  15: #ifndef LLVM_LIB_TARGET_M68K_M68KISELLOWERING_H
  16: #define LLVM_LIB_TARGET_M68K_M68KISELLOWERING_H
  17: 
  18: #include "M68k.h"
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68k.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68k.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 19-36 / 第 19-36 行
```cpp
  19: 
  20: #include "llvm/CodeGen/CallingConvLower.h"
  21: #include "llvm/CodeGen/SelectionDAG.h"
  22: #include "llvm/CodeGen/TargetLowering.h"
  23: #include "llvm/IR/Function.h"
  24: 
  25: #include <deque>
  26: 
  27: namespace llvm {
  28: 
  29: /// Define some predicates that are used for node matching.
  30: namespace M68k {
  31: 
  32: /// Determines whether the callee is required to pop its
  33: /// own arguments. Callee pop is necessary to support tail calls.
  34: bool isCalleePop(CallingConv::ID CallingConv, bool IsVarArg, bool GuaranteeTCO);
  35: 
  36: } // end namespace M68k
```
- **EN**: It imports dependencies such as `CallingConvLower.h`, `SelectionDAG.h`, `TargetLowering.h`, `Function.h`, `deque` that expose the LLVM and target interfaces used in later logic.
- **CN**: 它引入了 `CallingConvLower.h`, `SelectionDAG.h`, `TargetLowering.h`, `Function.h`, `deque` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。

### Lines 37-54 / 第 37-54 行
```cpp
  37: 
  38: //===--------------------------------------------------------------------===//
  39: // TargetLowering Implementation
  40: //===--------------------------------------------------------------------===//
  41: 
  42: class M68kMachineFunctionInfo;
  43: class M68kSubtarget;
  44: 
  45: class M68kTargetLowering : public TargetLowering {
  46:   const M68kSubtarget &Subtarget;
  47:   const M68kTargetMachine &TM;
  48: 
  49: public:
  50:   explicit M68kTargetLowering(const M68kTargetMachine &TM,
  51:                               const M68kSubtarget &STI);
  52: 
  53:   static const M68kTargetLowering *create(const M68kTargetMachine &TM,
  54:                                           const M68kSubtarget &STI);
```
- **EN**: This block declares or refines TableGen records such as `M68kMachineFunctionInfo`, `M68kSubtarget`, `M68kTargetLowering`.
- **CN**: 该代码块声明或细化了 `M68kMachineFunctionInfo`, `M68kSubtarget`, `M68kTargetLowering` 等 TableGen 记录。

### Lines 55-72 / 第 55-72 行
```cpp
  55: 
  56:   /// Return the value type to use for ISD::SETCC.
  57:   EVT getSetCCResultType(const DataLayout &DL, LLVMContext &Context,
  58:                          EVT VT) const override;
  59: 
  60:   /// EVT is not used in-tree, but is used by out-of-tree target.
  61:   virtual MVT getScalarShiftAmountTy(const DataLayout &, EVT) const override;
  62: 
  63:   /// Provide custom lowering hooks for some operations.
  64:   SDValue LowerOperation(SDValue Op, SelectionDAG &DAG) const override;
  65: 
  66:   /// Return the entry encoding for a jump table in the current function.
  67:   /// The returned value is a member of the  MachineJumpTableInfo::JTEntryKind
  68:   /// enum.
  69:   unsigned getJumpTableEncoding() const override;
  70: 
  71:   const MCExpr *LowerCustomJumpTableEntry(const MachineJumpTableInfo *MJTI,
  72:                                           const MachineBasicBlock *MBB,
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 73-90 / 第 73-90 行
```cpp
  73:                                           unsigned uid,
  74:                                           MCContext &Ctx) const override;
  75: 
  76:   /// Returns relocation base for the given PIC jumptable.
  77:   SDValue getPICJumpTableRelocBase(SDValue Table,
  78:                                    SelectionDAG &DAG) const override;
  79: 
  80:   /// This returns the relocation base for the given PIC jumptable,
  81:   /// the same as getPICJumpTableRelocBase, but as an MCExpr.
  82:   const MCExpr *getPICJumpTableRelocBaseExpr(const MachineFunction *MF,
  83:                                              unsigned JTI,
  84:                                              MCContext &Ctx) const override;
  85: 
  86:   ConstraintType getConstraintType(StringRef ConstraintStr) const override;
  87: 
  88:   std::pair<unsigned, const TargetRegisterClass *>
  89:   getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
  90:                                StringRef Constraint, MVT VT) const override;
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 91-108 / 第 91-108 行
```cpp
  91: 
  92:   // Lower operand with C_Immediate and C_Other constraint type
  93:   void LowerAsmOperandForConstraint(SDValue Op, StringRef Constraint,
  94:                                     std::vector<SDValue> &Ops,
  95:                                     SelectionDAG &DAG) const override;
  96: 
  97:   MachineBasicBlock *
  98:   EmitInstrWithCustomInserter(MachineInstr &MI,
  99:                               MachineBasicBlock *MBB) const override;
 100: 
 101:   CCAssignFn *getCCAssignFn(CallingConv::ID CC, bool Return,
 102:                             bool IsVarArg) const;
 103: 
 104:   AtomicExpansionKind
 105:   shouldExpandAtomicRMWInIR(const AtomicRMWInst *RMW) const override;
 106: 
 107:   /// If a physical register, this returns the register that receives the
 108:   /// exception address on entry to an EH pad.
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 109-126 / 第 109-126 行
```cpp
 109:   Register
 110:   getExceptionPointerRegister(const Constant *PersonalityFn) const override;
 111: 
 112:   /// If a physical register, this returns the register that receives the
 113:   /// exception typeid on entry to a landing pad.
 114:   Register
 115:   getExceptionSelectorRegister(const Constant *PersonalityFn) const override;
 116: 
 117:   InlineAsm::ConstraintCode
 118:   getInlineAsmMemConstraint(StringRef ConstraintCode) const override;
 119: 
 120:   // We need this for DAGCombiner to eliminate as many ISD::SELECT as possible.
 121:   // Otherwise we might end up with M68kISD::CMOV.
 122:   bool convertSelectOfConstantsToMath(EVT VT) const override { return true; }
 123: 
 124: private:
 125:   unsigned GetAlignedArgumentStackSize(unsigned StackSize,
 126:                                        SelectionDAG &DAG) const;
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 127-144 / 第 127-144 行
```cpp
 127: 
 128:   bool isOffsetFoldingLegal(const GlobalAddressSDNode *GA) const override {
 129:     // In many cases, `GA` doesn't give the correct offset to fold. It's
 130:     // hard to know if the real offset actually fits into the displacement
 131:     // of the perspective addressing mode.
 132:     // Thus, we disable offset folding altogether and leave that to ISel
 133:     // patterns.
 134:     return false;
 135:   }
 136: 
 137:   SDValue getReturnAddressFrameIndex(SelectionDAG &DAG) const;
 138: 
 139:   /// Emit a load of return address if tail call
 140:   /// optimization is performed and it is required.
 141:   SDValue EmitTailCallLoadRetAddr(SelectionDAG &DAG, SDValue &OutRetAddr,
 142:                                   SDValue Chain, bool IsTailCall, int FPDiff,
 143:                                   const SDLoc &DL) const;
 144: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 145-162 / 第 145-162 行
```cpp
 145:   /// Emit a store of the return address if tail call
 146:   /// optimization is performed and it is required (FPDiff!=0).
 147:   SDValue EmitTailCallStoreRetAddr(SelectionDAG &DAG, MachineFunction &MF,
 148:                                    SDValue Chain, SDValue RetAddrFrIdx,
 149:                                    EVT PtrVT, unsigned SlotSize, int FPDiff,
 150:                                    const SDLoc &DL) const;
 151: 
 152:   SDValue LowerMemArgument(SDValue Chain, CallingConv::ID CallConv,
 153:                            const SmallVectorImpl<ISD::InputArg> &ArgInfo,
 154:                            const SDLoc &DL, SelectionDAG &DAG,
 155:                            const CCValAssign &VA, MachineFrameInfo &MFI,
 156:                            unsigned ArgIdx) const;
 157: 
 158:   SDValue LowerMemOpCallTo(SDValue Chain, SDValue StackPtr, SDValue Arg,
 159:                            const SDLoc &DL, SelectionDAG &DAG,
 160:                            const CCValAssign &VA, ISD::ArgFlagsTy Flags) const;
 161: 
 162:   SDValue LowerXALUO(SDValue Op, SelectionDAG &DAG) const;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 163-180 / 第 163-180 行
```cpp
 163:   SDValue LowerToBTST(SDValue And, ISD::CondCode CC, const SDLoc &DL,
 164:                       SelectionDAG &DAG) const;
 165:   SDValue LowerSETCC(SDValue Op, SelectionDAG &DAG) const;
 166:   SDValue LowerSETCCCARRY(SDValue Op, SelectionDAG &DAG) const;
 167:   SDValue LowerSELECT(SDValue Op, SelectionDAG &DAG) const;
 168:   SDValue LowerBRCOND(SDValue Op, SelectionDAG &DAG) const;
 169:   SDValue LowerADDC_ADDE_SUBC_SUBE(SDValue Op, SelectionDAG &DAG) const;
 170:   SDValue LowerConstantPool(SDValue Op, SelectionDAG &DAG) const;
 171:   SDValue LowerJumpTable(SDValue Op, SelectionDAG &DAG) const;
 172:   SDValue LowerExternalSymbol(SDValue Op, SelectionDAG &DAG) const;
 173:   SDValue LowerBlockAddress(SDValue Op, SelectionDAG &DAG) const;
 174:   SDValue LowerGlobalAddress(const GlobalValue *GV, const SDLoc &DL,
 175:                              int64_t Offset, SelectionDAG &DAG) const;
 176:   SDValue LowerGlobalAddress(SDValue Op, SelectionDAG &DAG) const;
 177:   SDValue LowerVASTART(SDValue Op, SelectionDAG &DAG) const;
 178:   SDValue LowerDYNAMIC_STACKALLOC(SDValue Op, SelectionDAG &DAG) const;
 179:   SDValue LowerShiftLeftParts(SDValue Op, SelectionDAG &DAG) const;
 180:   SDValue LowerShiftRightParts(SDValue Op, SelectionDAG &DAG, bool IsSRA) const;
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 181-198 / 第 181-198 行
```cpp
 181: 
 182:   SDValue LowerATOMICFENCE(SDValue Op, SelectionDAG &DAG) const;
 183: 
 184:   SDValue LowerCallResult(SDValue Chain, SDValue InGlue,
 185:                           CallingConv::ID CallConv, bool IsVarArg,
 186:                           const SmallVectorImpl<ISD::InputArg> &Ins,
 187:                           const SDLoc &DL, SelectionDAG &DAG,
 188:                           SmallVectorImpl<SDValue> &InVals) const;
 189:   SDValue LowerGlobalTLSAddress(SDValue Op, SelectionDAG &DAG) const;
 190: 
 191:   /// LowerFormalArguments - transform physical registers into virtual
 192:   /// registers and generate load operations for arguments places on the stack.
 193:   SDValue LowerFormalArguments(SDValue Chain, CallingConv::ID CCID,
 194:                                bool IsVarArg,
 195:                                const SmallVectorImpl<ISD::InputArg> &Ins,
 196:                                const SDLoc &DL, SelectionDAG &DAG,
 197:                                SmallVectorImpl<SDValue> &InVals) const override;
 198: 
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 199-216 / 第 199-216 行
```cpp
 199:   SDValue LowerCall(CallLoweringInfo &CLI,
 200:                     SmallVectorImpl<SDValue> &InVals) const override;
 201: 
 202:   bool CanLowerReturn(CallingConv::ID CallConv, MachineFunction &MF,
 203:                       bool isVarArg,
 204:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
 205:                       LLVMContext &Context, const Type *RetTy) const override;
 206: 
 207:   /// Lower the result values of a call into the
 208:   /// appropriate copies out of appropriate physical registers.
 209:   SDValue LowerReturn(SDValue Chain, CallingConv::ID CCID, bool IsVarArg,
 210:                       const SmallVectorImpl<ISD::OutputArg> &Outs,
 211:                       const SmallVectorImpl<SDValue> &OutVals, const SDLoc &DL,
 212:                       SelectionDAG &DAG) const override;
 213: 
 214:   SDValue LowerExternalSymbolCall(SelectionDAG &DAG, SDLoc loc,
 215:                                   llvm::StringRef SymbolName,
 216:                                   ArgListTy &&ArgList) const;
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 217-234 / 第 217-234 行
```cpp
 217:   SDValue getTLSGetAddr(GlobalAddressSDNode *GA, SelectionDAG &DAG,
 218:                         unsigned TargetFlags) const;
 219:   SDValue getM68kReadTp(SDLoc Loc, SelectionDAG &DAG) const;
 220: 
 221:   SDValue LowerTLSGeneralDynamic(GlobalAddressSDNode *GA,
 222:                                  SelectionDAG &DAG) const;
 223:   SDValue LowerTLSLocalDynamic(GlobalAddressSDNode *GA,
 224:                                SelectionDAG &DAG) const;
 225:   SDValue LowerTLSInitialExec(GlobalAddressSDNode *GA, SelectionDAG &DAG) const;
 226:   SDValue LowerTLSLocalExec(GlobalAddressSDNode *GA, SelectionDAG &DAG) const;
 227: 
 228:   bool decomposeMulByConstant(LLVMContext &Context, EVT VT,
 229:                               SDValue C) const override;
 230: 
 231:   MachineBasicBlock *EmitLoweredSelect(MachineInstr &I,
 232:                                        MachineBasicBlock *MBB) const;
 233:   MachineBasicBlock *EmitLoweredSegAlloca(MachineInstr &MI,
 234:                                           MachineBasicBlock *BB) const;
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 235-252 / 第 235-252 行
```cpp
 235: 
 236:   /// Emit nodes that will be selected as "test Op0,Op0", or something
 237:   /// equivalent, for use with the given M68k condition code.
 238:   SDValue EmitTest(SDValue Op0, unsigned M68kCC, const SDLoc &dl,
 239:                    SelectionDAG &DAG) const;
 240: 
 241:   /// Emit nodes that will be selected as "cmp Op0,Op1", or something
 242:   /// equivalent, for use with the given M68k condition code.
 243:   SDValue EmitCmp(SDValue Op0, SDValue Op1, unsigned M68kCC, const SDLoc &dl,
 244:                   SelectionDAG &DAG) const;
 245: 
 246:   /// Check whether the call is eligible for tail call optimization. Targets
 247:   /// that want to do tail call optimization should implement this function.
 248:   bool IsEligibleForTailCallOptimization(
 249:       SDValue Callee, CallingConv::ID CalleeCC, bool IsVarArg,
 250:       bool IsCalleeStructRet, bool IsCallerStructRet, Type *RetTy,
 251:       const SmallVectorImpl<ISD::OutputArg> &Outs,
 252:       const SmallVectorImpl<SDValue> &OutVals,
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 253-259 / 第 253-259 行
```cpp
 253:       const SmallVectorImpl<ISD::InputArg> &Ins, SelectionDAG &DAG) const;
 254: 
 255:   SDValue PerformDAGCombine(SDNode *N, DAGCombinerInfo &DCI) const override;
 256: };
 257: } // namespace llvm
 258: 
 259: #endif // LLVM_LIB_TARGET_M68K_M68KISELLOWERING_H
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the M68k backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

## Key Concepts / 关键概念
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。
- **Subtarget features**: Tracks CPU capabilities that gate instructions and schedules. / 跟踪决定指令和调度的 CPU 能力。

## Dependencies / 依赖关系
- `M68k.h`
- `llvm/CodeGen/CallingConvLower.h`
- `llvm/CodeGen/SelectionDAG.h`
- `llvm/CodeGen/TargetLowering.h`
- `llvm/IR/Function.h`
- `deque`
