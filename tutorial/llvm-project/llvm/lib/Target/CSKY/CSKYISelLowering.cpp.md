# CSKYISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/CSKY/CSKYISelLowering.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file defines the interfaces that CSKY uses to lower LLVM code into a selection DAG.
- 目的（中文）: 实现 SelectionDAG 降级钩子，将通用 LLVM IR 操作映射到目标专用 DAG 节点与调用约定。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- CSKYISelLowering.cpp - CSKY DAG Lowering Implementation  ----------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the interfaces that CSKY uses to lower LLVM code into a
  10: // selection DAG.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "CSKYISelLowering.h"
  15: #include "CSKYCallingConv.h"
  16: #include "CSKYConstantPoolValue.h"
  17: #include "CSKYMachineFunctionInfo.h"
  18: #include "CSKYRegisterInfo.h"
  19: #include "CSKYSubtarget.h"
  20: #include "llvm/ADT/Statistic.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/CallingConvLower.h"
  22: #include "llvm/CodeGen/MachineFrameInfo.h"
  23: #include "llvm/CodeGen/MachineJumpTableInfo.h"
  24: #include "llvm/Support/Debug.h"
  25: 
  26: using namespace llvm;
  27: 
  28: #define DEBUG_TYPE "csky-isel-lowering"
  29: 
  30: STATISTIC(NumTailCalls, "Number of tail calls");
  31: 
  32: #include "CSKYGenCallingConv.inc"
  33: 
  34: static const MCPhysReg GPRArgRegs[] = {CSKY::R0, CSKY::R1, CSKY::R2, CSKY::R3};
  35: 
  36: CSKYTargetLowering::CSKYTargetLowering(const TargetMachine &TM,
  37:                                        const CSKYSubtarget &STI)
  38:     : TargetLowering(TM, STI), Subtarget(STI) {
  39:   // Register Class
  40:   addRegisterClass(MVT::i32, &CSKY::GPRRegClass);
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as TargetLowering contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 TargetLowering 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 41-60

```cpp
  41: 
  42:   if (STI.useHardFloat()) {
  43:     if (STI.hasFPUv2SingleFloat())
  44:       addRegisterClass(MVT::f32, &CSKY::sFPR32RegClass);
  45:     else if (STI.hasFPUv3SingleFloat())
  46:       addRegisterClass(MVT::f32, &CSKY::FPR32RegClass);
  47: 
  48:     if (STI.hasFPUv2DoubleFloat())
  49:       addRegisterClass(MVT::f64, &CSKY::sFPR64RegClass);
  50:     else if (STI.hasFPUv3DoubleFloat())
  51:       addRegisterClass(MVT::f64, &CSKY::FPR64RegClass);
  52:   }
  53: 
  54:   setOperationAction(ISD::UADDO_CARRY, MVT::i32, Legal);
  55:   setOperationAction(ISD::USUBO_CARRY, MVT::i32, Legal);
  56:   setOperationAction(ISD::BITREVERSE, MVT::i32, Legal);
  57: 
  58:   setOperationAction(ISD::SREM, MVT::i32, Expand);
  59:   setOperationAction(ISD::UREM, MVT::i32, Expand);
  60:   setOperationAction(ISD::UDIVREM, MVT::i32, Expand);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 61-80

```cpp
  61:   setOperationAction(ISD::SDIVREM, MVT::i32, Expand);
  62:   setOperationAction(ISD::CTPOP, MVT::i32, Expand);
  63:   setOperationAction(ISD::ROTR, MVT::i32, Expand);
  64:   setOperationAction(ISD::SHL_PARTS, MVT::i32, Expand);
  65:   setOperationAction(ISD::SRL_PARTS, MVT::i32, Expand);
  66:   setOperationAction(ISD::SRA_PARTS, MVT::i32, Expand);
  67:   setOperationAction(ISD::UMUL_LOHI, MVT::i32, Expand);
  68:   setOperationAction(ISD::SMUL_LOHI, MVT::i32, Expand);
  69:   setOperationAction(ISD::SELECT_CC, MVT::i32, Expand);
  70:   setOperationAction(ISD::BR_CC, MVT::i32, Expand);
  71:   setOperationAction(ISD::BR_JT, MVT::Other, Expand);
  72:   setOperationAction(ISD::DYNAMIC_STACKALLOC, MVT::i32, Expand);
  73:   setOperationAction(ISD::STACKSAVE, MVT::Other, Expand);
  74:   setOperationAction(ISD::STACKRESTORE, MVT::Other, Expand);
  75:   setOperationAction(ISD::MULHS, MVT::i32, Expand);
  76:   setOperationAction(ISD::MULHU, MVT::i32, Expand);
  77:   setOperationAction(ISD::VAARG, MVT::Other, Expand);
  78:   setOperationAction(ISD::VACOPY, MVT::Other, Expand);
  79:   setOperationAction(ISD::VAEND, MVT::Other, Expand);
  80: 
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 81-100

```cpp
  81:   setLoadExtAction(ISD::EXTLOAD, MVT::i32, MVT::i1, Promote);
  82:   setLoadExtAction(ISD::SEXTLOAD, MVT::i32, MVT::i1, Promote);
  83:   setLoadExtAction(ISD::ZEXTLOAD, MVT::i32, MVT::i1, Promote);
  84: 
  85:   setOperationAction(ISD::GlobalAddress, MVT::i32, Custom);
  86:   setOperationAction(ISD::ExternalSymbol, MVT::i32, Custom);
  87:   setOperationAction(ISD::GlobalTLSAddress, MVT::i32, Custom);
  88:   setOperationAction(ISD::BlockAddress, MVT::i32, Custom);
  89:   if (!Subtarget.hasE2()) {
  90:     setOperationAction(ISD::ConstantPool, MVT::i32, Custom);
  91:   }
  92:   setOperationAction(ISD::JumpTable, MVT::i32, Custom);
  93:   setOperationAction(ISD::VASTART, MVT::Other, Custom);
  94: 
  95:   if (!Subtarget.hasE2()) {
  96:     setLoadExtAction(ISD::SEXTLOAD, MVT::i32, MVT::i8, Expand);
  97:     setLoadExtAction(ISD::SEXTLOAD, MVT::i32, MVT::i16, Expand);
  98:     setOperationAction(ISD::CTLZ, MVT::i32, Expand);
  99:     setOperationAction(ISD::BSWAP, MVT::i32, Expand);
 100:   }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 101-120

```cpp
 101: 
 102:   if (!Subtarget.has2E3()) {
 103:     setOperationAction(ISD::ABS, MVT::i32, Expand);
 104:     setOperationAction(ISD::BITREVERSE, MVT::i32, Expand);
 105:     setOperationAction(ISD::CTTZ, MVT::i32, Expand);
 106:     setOperationAction(ISD::SDIV, MVT::i32, Expand);
 107:     setOperationAction(ISD::UDIV, MVT::i32, Expand);
 108:   }
 109: 
 110:   setOperationAction(ISD::ATOMIC_FENCE, MVT::Other, Expand);
 111: 
 112:   // Float
 113: 
 114:   ISD::CondCode FPCCToExtend[] = {
 115:       ISD::SETONE, ISD::SETUEQ, ISD::SETUGT,
 116:       ISD::SETUGE, ISD::SETULT, ISD::SETULE,
 117:   };
 118: 
 119:   ISD::NodeType FPOpToExpand[] = {
 120:       ISD::FSIN,      ISD::FCOS,       ISD::FSINCOS,   ISD::FPOW,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 121-140

```cpp
 121:       ISD::FCOPYSIGN, ISD::FP16_TO_FP, ISD::FP_TO_FP16};
 122: 
 123:   if (STI.useHardFloat()) {
 124: 
 125:     MVT AllVTy[] = {MVT::f32, MVT::f64};
 126: 
 127:     for (auto VT : AllVTy) {
 128:       setOperationAction(ISD::FREM, VT, LibCall);
 129:       setOperationAction(ISD::SELECT_CC, VT, Expand);
 130:       setOperationAction(ISD::BR_CC, VT, Expand);
 131: 
 132:       for (auto CC : FPCCToExtend)
 133:         setCondCodeAction(CC, VT, Expand);
 134:       for (auto Op : FPOpToExpand)
 135:         setOperationAction(Op, VT, Expand);
 136:     }
 137: 
 138:     if (STI.hasFPUv2SingleFloat() || STI.hasFPUv3SingleFloat()) {
 139:       setOperationAction(ISD::ConstantFP, MVT::f32, Legal);
 140:       setLoadExtAction(ISD::EXTLOAD, MVT::f32, MVT::f16, Expand);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 141-160

```cpp
 141:       setTruncStoreAction(MVT::f32, MVT::f16, Expand);
 142:     }
 143:     if (STI.hasFPUv2DoubleFloat() || STI.hasFPUv3DoubleFloat()) {
 144:       setLoadExtAction(ISD::EXTLOAD, MVT::f64, MVT::f32, Expand);
 145:       setTruncStoreAction(MVT::f64, MVT::f32, Expand);
 146:       setLoadExtAction(ISD::EXTLOAD, MVT::f64, MVT::f16, Expand);
 147:       setTruncStoreAction(MVT::f64, MVT::f16, Expand);
 148:     }
 149:   }
 150: 
 151:   // Compute derived properties from the register classes.
 152:   computeRegisterProperties(STI.getRegisterInfo());
 153: 
 154:   setBooleanContents(UndefinedBooleanContent);
 155:   setBooleanVectorContents(ZeroOrNegativeOneBooleanContent);
 156: 
 157:   // TODO: Add atomic support fully.
 158:   setMaxAtomicSizeInBitsSupported(0);
 159: 
 160:   setStackPointerRegisterToSaveRestore(CSKY::R14);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 161-180

```cpp
 161:   setMinFunctionAlignment(Align(2));
 162:   setSchedulingPreference(Sched::Source);
 163: }
 164: 
 165: SDValue CSKYTargetLowering::LowerOperation(SDValue Op,
 166:                                            SelectionDAG &DAG) const {
 167:   switch (Op.getOpcode()) {
 168:   default:
 169:     llvm_unreachable("unimplemented op");
 170:   case ISD::GlobalAddress:
 171:     return LowerGlobalAddress(Op, DAG);
 172:   case ISD::ExternalSymbol:
 173:     return LowerExternalSymbol(Op, DAG);
 174:   case ISD::GlobalTLSAddress:
 175:     return LowerGlobalTLSAddress(Op, DAG);
 176:   case ISD::JumpTable:
 177:     return LowerJumpTable(Op, DAG);
 178:   case ISD::BlockAddress:
 179:     return LowerBlockAddress(Op, DAG);
 180:   case ISD::ConstantPool:
```

- EN: Function bodies or method definitions such as LowerOperation contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerOperation 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 181-200

```cpp
 181:     return LowerConstantPool(Op, DAG);
 182:   case ISD::VASTART:
 183:     return LowerVASTART(Op, DAG);
 184:   case ISD::FRAMEADDR:
 185:     return LowerFRAMEADDR(Op, DAG);
 186:   case ISD::RETURNADDR:
 187:     return LowerRETURNADDR(Op, DAG);
 188:   }
 189: }
 190: 
 191: EVT CSKYTargetLowering::getSetCCResultType(const DataLayout &DL,
 192:                                            LLVMContext &Context, EVT VT) const {
 193:   if (!VT.isVector())
 194:     return MVT::i32;
 195: 
 196:   return VT.changeVectorElementTypeToInteger();
 197: }
 198: 
 199: static SDValue convertValVTToLocVT(SelectionDAG &DAG, SDValue Val,
 200:                                    const CCValAssign &VA, const SDLoc &DL) {
```

- EN: Function bodies or method definitions such as getSetCCResultType, convertValVTToLocVT contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getSetCCResultType, convertValVTToLocVT 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 201-220

```cpp
 201:   EVT LocVT = VA.getLocVT();
 202: 
 203:   switch (VA.getLocInfo()) {
 204:   default:
 205:     llvm_unreachable("Unexpected CCValAssign::LocInfo");
 206:   case CCValAssign::Full:
 207:     break;
 208:   case CCValAssign::BCvt:
 209:     Val = DAG.getNode(ISD::BITCAST, DL, LocVT, Val);
 210:     break;
 211:   }
 212:   return Val;
 213: }
 214: 
 215: static SDValue convertLocVTToValVT(SelectionDAG &DAG, SDValue Val,
 216:                                    const CCValAssign &VA, const SDLoc &DL) {
 217:   switch (VA.getLocInfo()) {
 218:   default:
 219:     llvm_unreachable("Unexpected CCValAssign::LocInfo");
 220:   case CCValAssign::Full:
```

- EN: Function bodies or method definitions such as convertLocVTToValVT contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: convertLocVTToValVT 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 221-240

```cpp
 221:     break;
 222:   case CCValAssign::BCvt:
 223:     Val = DAG.getNode(ISD::BITCAST, DL, VA.getValVT(), Val);
 224:     break;
 225:   }
 226:   return Val;
 227: }
 228: 
 229: static SDValue unpackFromRegLoc(const CSKYSubtarget &Subtarget,
 230:                                 SelectionDAG &DAG, SDValue Chain,
 231:                                 const CCValAssign &VA, const SDLoc &DL) {
 232:   MachineFunction &MF = DAG.getMachineFunction();
 233:   MachineRegisterInfo &RegInfo = MF.getRegInfo();
 234:   EVT LocVT = VA.getLocVT();
 235:   SDValue Val;
 236:   const TargetRegisterClass *RC;
 237: 
 238:   switch (LocVT.getSimpleVT().SimpleTy) {
 239:   default:
 240:     llvm_unreachable("Unexpected register type");
```

- EN: Function bodies or method definitions such as unpackFromRegLoc contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: unpackFromRegLoc 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 241-260

```cpp
 241:   case MVT::i32:
 242:     RC = &CSKY::GPRRegClass;
 243:     break;
 244:   case MVT::f32:
 245:     RC = Subtarget.hasFPUv2SingleFloat() ? &CSKY::sFPR32RegClass
 246:                                          : &CSKY::FPR32RegClass;
 247:     break;
 248:   case MVT::f64:
 249:     RC = Subtarget.hasFPUv2DoubleFloat() ? &CSKY::sFPR64RegClass
 250:                                          : &CSKY::FPR64RegClass;
 251:     break;
 252:   }
 253: 
 254:   Register VReg = RegInfo.createVirtualRegister(RC);
 255:   RegInfo.addLiveIn(VA.getLocReg(), VReg);
 256:   Val = DAG.getCopyFromReg(Chain, DL, VReg, LocVT);
 257: 
 258:   return convertLocVTToValVT(DAG, Val, VA, DL);
 259: }
 260: 
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 261-280

```cpp
 261: static SDValue unpackFromMemLoc(SelectionDAG &DAG, SDValue Chain,
 262:                                 const CCValAssign &VA, const SDLoc &DL) {
 263:   MachineFunction &MF = DAG.getMachineFunction();
 264:   MachineFrameInfo &MFI = MF.getFrameInfo();
 265:   EVT LocVT = VA.getLocVT();
 266:   EVT ValVT = VA.getValVT();
 267:   EVT PtrVT = MVT::getIntegerVT(DAG.getDataLayout().getPointerSizeInBits(0));
 268:   int FI = MFI.CreateFixedObject(ValVT.getSizeInBits() / 8,
 269:                                  VA.getLocMemOffset(), /*Immutable=*/true);
 270:   SDValue FIN = DAG.getFrameIndex(FI, PtrVT);
 271:   SDValue Val;
 272: 
 273:   ISD::LoadExtType ExtType;
 274:   switch (VA.getLocInfo()) {
 275:   default:
 276:     llvm_unreachable("Unexpected CCValAssign::LocInfo");
 277:   case CCValAssign::Full:
 278:   case CCValAssign::BCvt:
 279:     ExtType = ISD::NON_EXTLOAD;
 280:     break;
```

- EN: Function bodies or method definitions such as unpackFromMemLoc contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: unpackFromMemLoc 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 281-300

```cpp
 281:   }
 282:   Val = DAG.getExtLoad(
 283:       ExtType, DL, LocVT, Chain, FIN,
 284:       MachinePointerInfo::getFixedStack(DAG.getMachineFunction(), FI), ValVT);
 285:   return Val;
 286: }
 287: 
 288: static SDValue unpack64(SelectionDAG &DAG, SDValue Chain, const CCValAssign &VA,
 289:                         const SDLoc &DL) {
 290:   assert(VA.getLocVT() == MVT::i32 &&
 291:          (VA.getValVT() == MVT::f64 || VA.getValVT() == MVT::i64) &&
 292:          "Unexpected VA");
 293:   MachineFunction &MF = DAG.getMachineFunction();
 294:   MachineFrameInfo &MFI = MF.getFrameInfo();
 295:   MachineRegisterInfo &RegInfo = MF.getRegInfo();
 296: 
 297:   if (VA.isMemLoc()) {
 298:     // f64/i64 is passed on the stack.
 299:     int FI = MFI.CreateFixedObject(8, VA.getLocMemOffset(), /*Immutable=*/true);
 300:     SDValue FIN = DAG.getFrameIndex(FI, MVT::i32);
```

- EN: Function bodies or method definitions such as unpack64 contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: unpack64 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 301-320

```cpp
 301:     return DAG.getLoad(VA.getValVT(), DL, Chain, FIN,
 302:                        MachinePointerInfo::getFixedStack(MF, FI));
 303:   }
 304: 
 305:   assert(VA.isRegLoc() && "Expected register VA assignment");
 306: 
 307:   Register LoVReg = RegInfo.createVirtualRegister(&CSKY::GPRRegClass);
 308:   RegInfo.addLiveIn(VA.getLocReg(), LoVReg);
 309:   SDValue Lo = DAG.getCopyFromReg(Chain, DL, LoVReg, MVT::i32);
 310:   SDValue Hi;
 311:   if (VA.getLocReg() == CSKY::R3) {
 312:     // Second half of f64/i64 is passed on the stack.
 313:     int FI = MFI.CreateFixedObject(4, 0, /*Immutable=*/true);
 314:     SDValue FIN = DAG.getFrameIndex(FI, MVT::i32);
 315:     Hi = DAG.getLoad(MVT::i32, DL, Chain, FIN,
 316:                      MachinePointerInfo::getFixedStack(MF, FI));
 317:   } else {
 318:     // Second half of f64/i64 is passed in another GPR.
 319:     Register HiVReg = RegInfo.createVirtualRegister(&CSKY::GPRRegClass);
 320:     RegInfo.addLiveIn(VA.getLocReg() + 1, HiVReg);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 321-340

```cpp
 321:     Hi = DAG.getCopyFromReg(Chain, DL, HiVReg, MVT::i32);
 322:   }
 323:   return DAG.getNode(CSKYISD::BITCAST_FROM_LOHI, DL, VA.getValVT(), Lo, Hi);
 324: }
 325: 
 326: // Transform physical registers into virtual registers.
 327: SDValue CSKYTargetLowering::LowerFormalArguments(
 328:     SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
 329:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &DL,
 330:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
 331: 
 332:   switch (CallConv) {
 333:   default:
 334:     report_fatal_error("Unsupported calling convention");
 335:   case CallingConv::C:
 336:   case CallingConv::Fast:
 337:     break;
 338:   }
 339: 
 340:   MachineFunction &MF = DAG.getMachineFunction();
```

- EN: Function bodies or method definitions such as LowerFormalArguments contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerFormalArguments 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 341-360

```cpp
 341: 
 342:   // Used with vargs to acumulate store chains.
 343:   std::vector<SDValue> OutChains;
 344: 
 345:   // Assign locations to all of the incoming arguments.
 346:   SmallVector<CCValAssign, 16> ArgLocs;
 347:   CCState CCInfo(CallConv, IsVarArg, MF, ArgLocs, *DAG.getContext());
 348: 
 349:   CCInfo.AnalyzeFormalArguments(Ins, CCAssignFnForCall(CallConv, IsVarArg));
 350: 
 351:   for (unsigned i = 0, e = ArgLocs.size(); i != e; ++i) {
 352:     CCValAssign &VA = ArgLocs[i];
 353:     SDValue ArgValue;
 354: 
 355:     bool IsF64OnCSKY = VA.getLocVT() == MVT::i32 && VA.getValVT() == MVT::f64;
 356: 
 357:     if (IsF64OnCSKY)
 358:       ArgValue = unpack64(DAG, Chain, VA, DL);
 359:     else if (VA.isRegLoc())
 360:       ArgValue = unpackFromRegLoc(Subtarget, DAG, Chain, VA, DL);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 361-380

```cpp
 361:     else
 362:       ArgValue = unpackFromMemLoc(DAG, Chain, VA, DL);
 363: 
 364:     InVals.push_back(ArgValue);
 365:   }
 366: 
 367:   if (IsVarArg) {
 368:     const unsigned XLenInBytes = 4;
 369:     const MVT XLenVT = MVT::i32;
 370: 
 371:     ArrayRef<MCPhysReg> ArgRegs = ArrayRef(GPRArgRegs);
 372:     unsigned Idx = CCInfo.getFirstUnallocated(ArgRegs);
 373:     const TargetRegisterClass *RC = &CSKY::GPRRegClass;
 374:     MachineFrameInfo &MFI = MF.getFrameInfo();
 375:     MachineRegisterInfo &RegInfo = MF.getRegInfo();
 376:     CSKYMachineFunctionInfo *CSKYFI = MF.getInfo<CSKYMachineFunctionInfo>();
 377: 
 378:     // Offset of the first variable argument from stack pointer, and size of
 379:     // the vararg save area. For now, the varargs save area is either zero or
 380:     // large enough to hold a0-a4.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 381-400

```cpp
 381:     int VaArgOffset, VarArgsSaveSize;
 382: 
 383:     // If all registers are allocated, then all varargs must be passed on the
 384:     // stack and we don't need to save any argregs.
 385:     if (ArgRegs.size() == Idx) {
 386:       VaArgOffset = CCInfo.getStackSize();
 387:       VarArgsSaveSize = 0;
 388:     } else {
 389:       VarArgsSaveSize = XLenInBytes * (ArgRegs.size() - Idx);
 390:       VaArgOffset = -VarArgsSaveSize;
 391:     }
 392: 
 393:     // Record the frame index of the first variable argument
 394:     // which is a value necessary to VASTART.
 395:     int FI = MFI.CreateFixedObject(XLenInBytes, VaArgOffset, true);
 396:     CSKYFI->setVarArgsFrameIndex(FI);
 397: 
 398:     // Copy the integer registers that may have been used for passing varargs
 399:     // to the vararg save area.
 400:     for (unsigned I = Idx; I < ArgRegs.size();
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 401-420

```cpp
 401:          ++I, VaArgOffset += XLenInBytes) {
 402:       const Register Reg = RegInfo.createVirtualRegister(RC);
 403:       RegInfo.addLiveIn(ArgRegs[I], Reg);
 404:       SDValue ArgValue = DAG.getCopyFromReg(Chain, DL, Reg, XLenVT);
 405:       FI = MFI.CreateFixedObject(XLenInBytes, VaArgOffset, true);
 406:       SDValue PtrOff = DAG.getFrameIndex(FI, getPointerTy(DAG.getDataLayout()));
 407:       SDValue Store = DAG.getStore(Chain, DL, ArgValue, PtrOff,
 408:                                    MachinePointerInfo::getFixedStack(MF, FI));
 409:       cast<StoreSDNode>(Store.getNode())
 410:           ->getMemOperand()
 411:           ->setValue((Value *)nullptr);
 412:       OutChains.push_back(Store);
 413:     }
 414:     CSKYFI->setVarArgsSaveSize(VarArgsSaveSize);
 415:   }
 416: 
 417:   // All stores are grouped in one node to allow the matching between
 418:   // the size of Ins and InVals. This only happens for vararg functions.
 419:   if (!OutChains.empty()) {
 420:     OutChains.push_back(Chain);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 421-440

```cpp
 421:     Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, OutChains);
 422:   }
 423: 
 424:   return Chain;
 425: }
 426: 
 427: bool CSKYTargetLowering::CanLowerReturn(
 428:     CallingConv::ID CallConv, MachineFunction &MF, bool IsVarArg,
 429:     const SmallVectorImpl<ISD::OutputArg> &Outs, LLVMContext &Context,
 430:     const Type *RetTy) const {
 431:   SmallVector<CCValAssign, 16> CSKYLocs;
 432:   CCState CCInfo(CallConv, IsVarArg, MF, CSKYLocs, Context);
 433:   return CCInfo.CheckReturn(Outs, CCAssignFnForReturn(CallConv, IsVarArg));
 434: }
 435: 
 436: SDValue
 437: CSKYTargetLowering::LowerReturn(SDValue Chain, CallingConv::ID CallConv,
 438:                                 bool IsVarArg,
 439:                                 const SmallVectorImpl<ISD::OutputArg> &Outs,
 440:                                 const SmallVectorImpl<SDValue> &OutVals,
```

- EN: Function bodies or method definitions such as CanLowerReturn contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: CanLowerReturn 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 441-460

```cpp
 441:                                 const SDLoc &DL, SelectionDAG &DAG) const {
 442:   // Stores the assignment of the return value to a location.
 443:   SmallVector<CCValAssign, 16> CSKYLocs;
 444: 
 445:   // Info about the registers and stack slot.
 446:   CCState CCInfo(CallConv, IsVarArg, DAG.getMachineFunction(), CSKYLocs,
 447:                  *DAG.getContext());
 448:   CCInfo.AnalyzeReturn(Outs, CCAssignFnForReturn(CallConv, IsVarArg));
 449: 
 450:   SDValue Glue;
 451:   SmallVector<SDValue, 4> RetOps(1, Chain);
 452: 
 453:   // Copy the result values into the output registers.
 454:   for (unsigned i = 0, e = CSKYLocs.size(); i < e; ++i) {
 455:     SDValue Val = OutVals[i];
 456:     CCValAssign &VA = CSKYLocs[i];
 457:     assert(VA.isRegLoc() && "Can only return in registers!");
 458: 
 459:     bool IsF64OnCSKY = VA.getLocVT() == MVT::i32 && VA.getValVT() == MVT::f64;
 460: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 461-480

```cpp
 461:     if (IsF64OnCSKY) {
 462: 
 463:       assert(VA.isRegLoc() && "Expected return via registers");
 464:       SDValue Split64 = DAG.getNode(CSKYISD::BITCAST_TO_LOHI, DL,
 465:                                     DAG.getVTList(MVT::i32, MVT::i32), Val);
 466:       SDValue Lo = Split64.getValue(0);
 467:       SDValue Hi = Split64.getValue(1);
 468: 
 469:       Register RegLo = VA.getLocReg();
 470:       assert(RegLo < CSKY::R31 && "Invalid register pair");
 471:       Register RegHi = RegLo + 1;
 472: 
 473:       Chain = DAG.getCopyToReg(Chain, DL, RegLo, Lo, Glue);
 474:       Glue = Chain.getValue(1);
 475:       RetOps.push_back(DAG.getRegister(RegLo, MVT::i32));
 476:       Chain = DAG.getCopyToReg(Chain, DL, RegHi, Hi, Glue);
 477:       Glue = Chain.getValue(1);
 478:       RetOps.push_back(DAG.getRegister(RegHi, MVT::i32));
 479:     } else {
 480:       // Handle a 'normal' return.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 481-500

```cpp
 481:       Val = convertValVTToLocVT(DAG, Val, VA, DL);
 482:       Chain = DAG.getCopyToReg(Chain, DL, VA.getLocReg(), Val, Glue);
 483: 
 484:       // Guarantee that all emitted copies are stuck together.
 485:       Glue = Chain.getValue(1);
 486:       RetOps.push_back(DAG.getRegister(VA.getLocReg(), VA.getLocVT()));
 487:     }
 488:   }
 489: 
 490:   RetOps[0] = Chain; // Update chain.
 491: 
 492:   // Add the glue node if we have it.
 493:   if (Glue.getNode()) {
 494:     RetOps.push_back(Glue);
 495:   }
 496: 
 497:   // Interrupt service routines use different return instructions.
 498:   if (DAG.getMachineFunction().getFunction().hasFnAttribute("interrupt"))
 499:     return DAG.getNode(CSKYISD::NIR, DL, MVT::Other, RetOps);
 500: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 501-520

```cpp
 501:   return DAG.getNode(CSKYISD::RET, DL, MVT::Other, RetOps);
 502: }
 503: 
 504: // Lower a call to a callseq_start + CALL + callseq_end chain, and add input
 505: // and output parameter nodes.
 506: SDValue CSKYTargetLowering::LowerCall(CallLoweringInfo &CLI,
 507:                                       SmallVectorImpl<SDValue> &InVals) const {
 508:   SelectionDAG &DAG = CLI.DAG;
 509:   SDLoc &DL = CLI.DL;
 510:   SmallVectorImpl<ISD::OutputArg> &Outs = CLI.Outs;
 511:   SmallVectorImpl<SDValue> &OutVals = CLI.OutVals;
 512:   SmallVectorImpl<ISD::InputArg> &Ins = CLI.Ins;
 513:   SDValue Chain = CLI.Chain;
 514:   SDValue Callee = CLI.Callee;
 515:   bool &IsTailCall = CLI.IsTailCall;
 516:   CallingConv::ID CallConv = CLI.CallConv;
 517:   bool IsVarArg = CLI.IsVarArg;
 518:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
 519:   MVT XLenVT = MVT::i32;
 520: 
```

- EN: Function bodies or method definitions such as LowerCall contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerCall 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 521-540

```cpp
 521:   MachineFunction &MF = DAG.getMachineFunction();
 522: 
 523:   // Analyze the operands of the call, assigning locations to each operand.
 524:   SmallVector<CCValAssign, 16> ArgLocs;
 525:   CCState ArgCCInfo(CallConv, IsVarArg, MF, ArgLocs, *DAG.getContext());
 526: 
 527:   ArgCCInfo.AnalyzeCallOperands(Outs, CCAssignFnForCall(CallConv, IsVarArg));
 528: 
 529:   // Check if it's really possible to do a tail call.
 530:   if (IsTailCall)
 531:     IsTailCall = false; // TODO: TailCallOptimization;
 532: 
 533:   if (IsTailCall)
 534:     ++NumTailCalls;
 535:   else if (CLI.CB && CLI.CB->isMustTailCall())
 536:     report_fatal_error("failed to perform tail call elimination on a call "
 537:                        "site marked musttail");
 538: 
 539:   // Get a count of how many bytes are to be pushed on the stack.
 540:   unsigned NumBytes = ArgCCInfo.getStackSize();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 541-560

```cpp
 541: 
 542:   // Create local copies for byval args
 543:   SmallVector<SDValue, 8> ByValArgs;
 544:   for (unsigned i = 0, e = Outs.size(); i != e; ++i) {
 545:     ISD::ArgFlagsTy Flags = Outs[i].Flags;
 546:     if (!Flags.isByVal())
 547:       continue;
 548: 
 549:     SDValue Arg = OutVals[i];
 550:     unsigned Size = Flags.getByValSize();
 551:     Align Alignment = Flags.getNonZeroByValAlign();
 552: 
 553:     int FI =
 554:         MF.getFrameInfo().CreateStackObject(Size, Alignment, /*isSS=*/false);
 555:     SDValue FIPtr = DAG.getFrameIndex(FI, getPointerTy(DAG.getDataLayout()));
 556:     SDValue SizeNode = DAG.getConstant(Size, DL, XLenVT);
 557: 
 558:     Chain = DAG.getMemcpy(Chain, DL, FIPtr, Arg, SizeNode, Alignment,
 559:                           /*IsVolatile=*/false,
 560:                           /*AlwaysInline=*/false, /*CI=*/nullptr, IsTailCall,
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 561-580

```cpp
 561:                           MachinePointerInfo(), MachinePointerInfo());
 562:     ByValArgs.push_back(FIPtr);
 563:   }
 564: 
 565:   if (!IsTailCall)
 566:     Chain = DAG.getCALLSEQ_START(Chain, NumBytes, 0, CLI.DL);
 567: 
 568:   // Copy argument values to their designated locations.
 569:   SmallVector<std::pair<Register, SDValue>, 8> RegsToPass;
 570:   SmallVector<SDValue, 8> MemOpChains;
 571:   SDValue StackPtr;
 572:   for (unsigned i = 0, j = 0, e = ArgLocs.size(); i != e; ++i) {
 573:     CCValAssign &VA = ArgLocs[i];
 574:     SDValue ArgValue = OutVals[i];
 575:     ISD::ArgFlagsTy Flags = Outs[i].Flags;
 576: 
 577:     bool IsF64OnCSKY = VA.getLocVT() == MVT::i32 && VA.getValVT() == MVT::f64;
 578: 
 579:     if (IsF64OnCSKY && VA.isRegLoc()) {
 580:       SDValue Split64 =
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 581-600

```cpp
 581:           DAG.getNode(CSKYISD::BITCAST_TO_LOHI, DL,
 582:                       DAG.getVTList(MVT::i32, MVT::i32), ArgValue);
 583:       SDValue Lo = Split64.getValue(0);
 584:       SDValue Hi = Split64.getValue(1);
 585: 
 586:       Register RegLo = VA.getLocReg();
 587:       RegsToPass.push_back(std::make_pair(RegLo, Lo));
 588: 
 589:       if (RegLo == CSKY::R3) {
 590:         // Second half of f64/i64 is passed on the stack.
 591:         // Work out the address of the stack slot.
 592:         if (!StackPtr.getNode())
 593:           StackPtr = DAG.getCopyFromReg(Chain, DL, CSKY::R14, PtrVT);
 594:         // Emit the store.
 595:         MemOpChains.push_back(
 596:             DAG.getStore(Chain, DL, Hi, StackPtr, MachinePointerInfo()));
 597:       } else {
 598:         // Second half of f64/i64 is passed in another GPR.
 599:         assert(RegLo < CSKY::R31 && "Invalid register pair");
 600:         Register RegHigh = RegLo + 1;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 601-620

```cpp
 601:         RegsToPass.push_back(std::make_pair(RegHigh, Hi));
 602:       }
 603:       continue;
 604:     }
 605: 
 606:     ArgValue = convertValVTToLocVT(DAG, ArgValue, VA, DL);
 607: 
 608:     // Use local copy if it is a byval arg.
 609:     if (Flags.isByVal())
 610:       ArgValue = ByValArgs[j++];
 611: 
 612:     if (VA.isRegLoc()) {
 613:       // Queue up the argument copies and emit them at the end.
 614:       RegsToPass.push_back(std::make_pair(VA.getLocReg(), ArgValue));
 615:     } else {
 616:       assert(VA.isMemLoc() && "Argument not register or memory");
 617:       assert(!IsTailCall && "Tail call not allowed if stack is used "
 618:                             "for passing parameters");
 619: 
 620:       // Work out the address of the stack slot.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 621-640

```cpp
 621:       if (!StackPtr.getNode())
 622:         StackPtr = DAG.getCopyFromReg(Chain, DL, CSKY::R14, PtrVT);
 623:       SDValue Address =
 624:           DAG.getNode(ISD::ADD, DL, PtrVT, StackPtr,
 625:                       DAG.getIntPtrConstant(VA.getLocMemOffset(), DL));
 626: 
 627:       // Emit the store.
 628:       MemOpChains.push_back(
 629:           DAG.getStore(Chain, DL, ArgValue, Address, MachinePointerInfo()));
 630:     }
 631:   }
 632: 
 633:   // Join the stores, which are independent of one another.
 634:   if (!MemOpChains.empty())
 635:     Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, MemOpChains);
 636: 
 637:   SDValue Glue;
 638: 
 639:   // Build a sequence of copy-to-reg nodes, chained and glued together.
 640:   for (auto &Reg : RegsToPass) {
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 641-660

```cpp
 641:     Chain = DAG.getCopyToReg(Chain, DL, Reg.first, Reg.second, Glue);
 642:     Glue = Chain.getValue(1);
 643:   }
 644: 
 645:   SmallVector<SDValue, 8> Ops;
 646:   EVT Ty = getPointerTy(DAG.getDataLayout());
 647:   bool IsRegCall = false;
 648: 
 649:   Ops.push_back(Chain);
 650: 
 651:   if (GlobalAddressSDNode *S = dyn_cast<GlobalAddressSDNode>(Callee)) {
 652:     const GlobalValue *GV = S->getGlobal();
 653:     bool IsLocal = getTargetMachine().shouldAssumeDSOLocal(GV);
 654: 
 655:     if (isPositionIndependent() || !Subtarget.has2E3()) {
 656:       IsRegCall = true;
 657:       Ops.push_back(getAddr<GlobalAddressSDNode, true>(S, DAG, IsLocal));
 658:     } else {
 659:       Ops.push_back(getTargetNode(cast<GlobalAddressSDNode>(Callee), DL, Ty,
 660:                                   DAG, CSKYII::MO_None));
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 661-680

```cpp
 661:       Ops.push_back(getTargetConstantPoolValue(
 662:           cast<GlobalAddressSDNode>(Callee), Ty, DAG, CSKYII::MO_None));
 663:     }
 664:   } else if (ExternalSymbolSDNode *S = dyn_cast<ExternalSymbolSDNode>(Callee)) {
 665:     bool IsLocal = getTargetMachine().shouldAssumeDSOLocal(nullptr);
 666: 
 667:     if (isPositionIndependent() || !Subtarget.has2E3()) {
 668:       IsRegCall = true;
 669:       Ops.push_back(getAddr<ExternalSymbolSDNode, true>(S, DAG, IsLocal));
 670:     } else {
 671:       Ops.push_back(getTargetNode(cast<ExternalSymbolSDNode>(Callee), DL, Ty,
 672:                                   DAG, CSKYII::MO_None));
 673:       Ops.push_back(getTargetConstantPoolValue(
 674:           cast<ExternalSymbolSDNode>(Callee), Ty, DAG, CSKYII::MO_None));
 675:     }
 676:   } else {
 677:     IsRegCall = true;
 678:     Ops.push_back(Callee);
 679:   }
 680: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 681-700

```cpp
 681:   // Add argument registers to the end of the list so that they are
 682:   // known live into the call.
 683:   for (auto &Reg : RegsToPass)
 684:     Ops.push_back(DAG.getRegister(Reg.first, Reg.second.getValueType()));
 685: 
 686:   if (!IsTailCall) {
 687:     // Add a register mask operand representing the call-preserved registers.
 688:     const TargetRegisterInfo *TRI = Subtarget.getRegisterInfo();
 689:     const uint32_t *Mask = TRI->getCallPreservedMask(MF, CallConv);
 690:     assert(Mask && "Missing call preserved mask for calling convention");
 691:     Ops.push_back(DAG.getRegisterMask(Mask));
 692:   }
 693: 
 694:   // Glue the call to the argument copies, if any.
 695:   if (Glue.getNode())
 696:     Ops.push_back(Glue);
 697: 
 698:   // Emit the call.
 699:   SDVTList NodeTys = DAG.getVTList(MVT::Other, MVT::Glue);
 700: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 701-720

```cpp
 701:   if (IsTailCall) {
 702:     MF.getFrameInfo().setHasTailCall();
 703:     return DAG.getNode(IsRegCall ? CSKYISD::TAILReg : CSKYISD::TAIL, DL,
 704:                        NodeTys, Ops);
 705:   }
 706: 
 707:   Chain = DAG.getNode(IsRegCall ? CSKYISD::CALLReg : CSKYISD::CALL, DL, NodeTys,
 708:                       Ops);
 709:   DAG.addNoMergeSiteInfo(Chain.getNode(), CLI.NoMerge);
 710:   Glue = Chain.getValue(1);
 711: 
 712:   // Mark the end of the call, which is glued to the call itself.
 713:   Chain = DAG.getCALLSEQ_END(Chain, NumBytes, 0, Glue, DL);
 714:   Glue = Chain.getValue(1);
 715: 
 716:   // Assign locations to each value returned by this call.
 717:   SmallVector<CCValAssign, 16> CSKYLocs;
 718:   CCState RetCCInfo(CallConv, IsVarArg, MF, CSKYLocs, *DAG.getContext());
 719:   RetCCInfo.AnalyzeCallResult(Ins, CCAssignFnForReturn(CallConv, IsVarArg));
 720: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 721-740

```cpp
 721:   // Copy all of the result registers out of their specified physreg.
 722:   for (auto &VA : CSKYLocs) {
 723:     // Copy the value out
 724:     SDValue RetValue =
 725:         DAG.getCopyFromReg(Chain, DL, VA.getLocReg(), VA.getLocVT(), Glue);
 726:     // Glue the RetValue to the end of the call sequence
 727:     Chain = RetValue.getValue(1);
 728:     Glue = RetValue.getValue(2);
 729: 
 730:     bool IsF64OnCSKY = VA.getLocVT() == MVT::i32 && VA.getValVT() == MVT::f64;
 731: 
 732:     if (IsF64OnCSKY) {
 733:       assert(VA.getLocReg() == GPRArgRegs[0] && "Unexpected reg assignment");
 734:       SDValue RetValue2 =
 735:           DAG.getCopyFromReg(Chain, DL, GPRArgRegs[1], MVT::i32, Glue);
 736:       Chain = RetValue2.getValue(1);
 737:       Glue = RetValue2.getValue(2);
 738:       RetValue = DAG.getNode(CSKYISD::BITCAST_FROM_LOHI, DL, VA.getValVT(),
 739:                              RetValue, RetValue2);
 740:     }
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 741-760

```cpp
 741: 
 742:     RetValue = convertLocVTToValVT(DAG, RetValue, VA, DL);
 743: 
 744:     InVals.push_back(RetValue);
 745:   }
 746: 
 747:   return Chain;
 748: }
 749: 
 750: CCAssignFn *CSKYTargetLowering::CCAssignFnForReturn(CallingConv::ID CC,
 751:                                                     bool IsVarArg) const {
 752:   if (IsVarArg || !Subtarget.useHardFloatABI())
 753:     return RetCC_CSKY_ABIV2_SOFT;
 754:   else
 755:     return RetCC_CSKY_ABIV2_FP;
 756: }
 757: 
 758: CCAssignFn *CSKYTargetLowering::CCAssignFnForCall(CallingConv::ID CC,
 759:                                                   bool IsVarArg) const {
 760:   if (IsVarArg || !Subtarget.useHardFloatABI())
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 761-780

```cpp
 761:     return CC_CSKY_ABIV2_SOFT;
 762:   else
 763:     return CC_CSKY_ABIV2_FP;
 764: }
 765: 
 766: static CSKYCP::CSKYCPModifier getModifier(unsigned Flags) {
 767: 
 768:   if (Flags == CSKYII::MO_ADDR32)
 769:     return CSKYCP::ADDR;
 770:   else if (Flags == CSKYII::MO_GOT32)
 771:     return CSKYCP::GOT;
 772:   else if (Flags == CSKYII::MO_GOTOFF)
 773:     return CSKYCP::GOTOFF;
 774:   else if (Flags == CSKYII::MO_PLT32)
 775:     return CSKYCP::PLT;
 776:   else if (Flags == CSKYII::MO_None)
 777:     return CSKYCP::NO_MOD;
 778:   else
 779:     assert(0 && "unknown CSKYII Modifier");
 780:   return CSKYCP::NO_MOD;
```

- EN: Function bodies or method definitions such as getModifier contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getModifier 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 781-800

```cpp
 781: }
 782: 
 783: SDValue CSKYTargetLowering::getTargetConstantPoolValue(GlobalAddressSDNode *N,
 784:                                                        EVT Ty,
 785:                                                        SelectionDAG &DAG,
 786:                                                        unsigned Flags) const {
 787:   CSKYConstantPoolValue *CPV = CSKYConstantPoolConstant::Create(
 788:       N->getGlobal(), CSKYCP::CPValue, 0, getModifier(Flags), false);
 789: 
 790:   return DAG.getTargetConstantPool(CPV, Ty);
 791: }
 792: 
 793: CSKYTargetLowering::ConstraintType
 794: CSKYTargetLowering::getConstraintType(StringRef Constraint) const {
 795:   if (Constraint.size() == 1) {
 796:     switch (Constraint[0]) {
 797:     default:
 798:       break;
 799:     case 'a':
 800:     case 'b':
```

- EN: Function bodies or method definitions such as getTargetConstantPoolValue, getConstraintType contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getTargetConstantPoolValue, getConstraintType 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 801-820

```cpp
 801:     case 'v':
 802:     case 'w':
 803:     case 'y':
 804:       return C_RegisterClass;
 805:     case 'c':
 806:     case 'l':
 807:     case 'h':
 808:     case 'z':
 809:       return C_Register;
 810:     }
 811:   }
 812:   return TargetLowering::getConstraintType(Constraint);
 813: }
 814: 
 815: std::pair<unsigned, const TargetRegisterClass *>
 816: CSKYTargetLowering::getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
 817:                                                  StringRef Constraint,
 818:                                                  MVT VT) const {
 819:   if (Constraint.size() == 1) {
 820:     switch (Constraint[0]) {
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 821-840

```cpp
 821:     case 'r':
 822:       return std::make_pair(0U, &CSKY::GPRRegClass);
 823:     case 'a':
 824:       return std::make_pair(0U, &CSKY::mGPRRegClass);
 825:     case 'b':
 826:       return std::make_pair(0U, &CSKY::sGPRRegClass);
 827:     case 'z':
 828:       return std::make_pair(CSKY::R14, &CSKY::GPRRegClass);
 829:     case 'c':
 830:       return std::make_pair(CSKY::C, &CSKY::CARRYRegClass);
 831:     case 'w':
 832:       if ((Subtarget.hasFPUv2SingleFloat() ||
 833:            Subtarget.hasFPUv3SingleFloat()) &&
 834:           VT == MVT::f32)
 835:         return std::make_pair(0U, &CSKY::sFPR32RegClass);
 836:       if ((Subtarget.hasFPUv2DoubleFloat() ||
 837:            Subtarget.hasFPUv3DoubleFloat()) &&
 838:           VT == MVT::f64)
 839:         return std::make_pair(0U, &CSKY::sFPR64RegClass);
 840:       break;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 841-860

```cpp
 841:     case 'v':
 842:       if (Subtarget.hasFPUv2SingleFloat() && VT == MVT::f32)
 843:         return std::make_pair(0U, &CSKY::sFPR32RegClass);
 844:       if (Subtarget.hasFPUv3SingleFloat() && VT == MVT::f32)
 845:         return std::make_pair(0U, &CSKY::FPR32RegClass);
 846:       if (Subtarget.hasFPUv2DoubleFloat() && VT == MVT::f64)
 847:         return std::make_pair(0U, &CSKY::sFPR64RegClass);
 848:       if (Subtarget.hasFPUv3DoubleFloat() && VT == MVT::f64)
 849:         return std::make_pair(0U, &CSKY::FPR64RegClass);
 850:       break;
 851:     default:
 852:       break;
 853:     }
 854:   }
 855: 
 856:   if (Constraint == "{c}")
 857:     return std::make_pair(CSKY::C, &CSKY::CARRYRegClass);
 858: 
 859:   // Clang will correctly decode the usage of register name aliases into their
 860:   // official names. However, other frontends like `rustc` do not. This allows
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 861-880

```cpp
 861:   // users of these frontends to use the ABI names for registers in LLVM-style
 862:   // register constraints.
 863:   unsigned XRegFromAlias = StringSwitch<unsigned>(Constraint.lower())
 864:                                .Case("{a0}", CSKY::R0)
 865:                                .Case("{a1}", CSKY::R1)
 866:                                .Case("{a2}", CSKY::R2)
 867:                                .Case("{a3}", CSKY::R3)
 868:                                .Case("{l0}", CSKY::R4)
 869:                                .Case("{l1}", CSKY::R5)
 870:                                .Case("{l2}", CSKY::R6)
 871:                                .Case("{l3}", CSKY::R7)
 872:                                .Case("{l4}", CSKY::R8)
 873:                                .Case("{l5}", CSKY::R9)
 874:                                .Case("{l6}", CSKY::R10)
 875:                                .Case("{l7}", CSKY::R11)
 876:                                .Case("{t0}", CSKY::R12)
 877:                                .Case("{t1}", CSKY::R13)
 878:                                .Case("{sp}", CSKY::R14)
 879:                                .Case("{lr}", CSKY::R15)
 880:                                .Case("{l8}", CSKY::R16)
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 881-900

```cpp
 881:                                .Case("{l9}", CSKY::R17)
 882:                                .Case("{t2}", CSKY::R18)
 883:                                .Case("{t3}", CSKY::R19)
 884:                                .Case("{t4}", CSKY::R20)
 885:                                .Case("{t5}", CSKY::R21)
 886:                                .Case("{t6}", CSKY::R22)
 887:                                .Cases({"{t7}", "{fp}"}, CSKY::R23)
 888:                                .Cases({"{t8}", "{top}"}, CSKY::R24)
 889:                                .Cases({"{t9}", "{bsp}"}, CSKY::R25)
 890:                                .Case("{r26}", CSKY::R26)
 891:                                .Case("{r27}", CSKY::R27)
 892:                                .Cases({"{gb}", "{rgb}", "{rdb}"}, CSKY::R28)
 893:                                .Cases({"{tb}", "{rtb}"}, CSKY::R29)
 894:                                .Case("{svbr}", CSKY::R30)
 895:                                .Case("{tls}", CSKY::R31)
 896:                                .Default(CSKY::NoRegister);
 897: 
 898:   if (XRegFromAlias != CSKY::NoRegister)
 899:     return std::make_pair(XRegFromAlias, &CSKY::GPRRegClass);
 900: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 901-920

```cpp
 901:   // Since TargetLowering::getRegForInlineAsmConstraint uses the name of the
 902:   // TableGen record rather than the AsmName to choose registers for InlineAsm
 903:   // constraints, plus we want to match those names to the widest floating point
 904:   // register type available, manually select floating point registers here.
 905:   //
 906:   // The second case is the ABI name of the register, so that frontends can also
 907:   // use the ABI names in register constraint lists.
 908:   if (Subtarget.useHardFloat()) {
 909:     unsigned FReg = StringSwitch<unsigned>(Constraint.lower())
 910:                         .Cases({"{fr0}", "{vr0}"}, CSKY::F0_32)
 911:                         .Cases({"{fr1}", "{vr1}"}, CSKY::F1_32)
 912:                         .Cases({"{fr2}", "{vr2}"}, CSKY::F2_32)
 913:                         .Cases({"{fr3}", "{vr3}"}, CSKY::F3_32)
 914:                         .Cases({"{fr4}", "{vr4}"}, CSKY::F4_32)
 915:                         .Cases({"{fr5}", "{vr5}"}, CSKY::F5_32)
 916:                         .Cases({"{fr6}", "{vr6}"}, CSKY::F6_32)
 917:                         .Cases({"{fr7}", "{vr7}"}, CSKY::F7_32)
 918:                         .Cases({"{fr8}", "{vr8}"}, CSKY::F8_32)
 919:                         .Cases({"{fr9}", "{vr9}"}, CSKY::F9_32)
 920:                         .Cases({"{fr10}", "{vr10}"}, CSKY::F10_32)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 921-940

```cpp
 921:                         .Cases({"{fr11}", "{vr11}"}, CSKY::F11_32)
 922:                         .Cases({"{fr12}", "{vr12}"}, CSKY::F12_32)
 923:                         .Cases({"{fr13}", "{vr13}"}, CSKY::F13_32)
 924:                         .Cases({"{fr14}", "{vr14}"}, CSKY::F14_32)
 925:                         .Cases({"{fr15}", "{vr15}"}, CSKY::F15_32)
 926:                         .Cases({"{fr16}", "{vr16}"}, CSKY::F16_32)
 927:                         .Cases({"{fr17}", "{vr17}"}, CSKY::F17_32)
 928:                         .Cases({"{fr18}", "{vr18}"}, CSKY::F18_32)
 929:                         .Cases({"{fr19}", "{vr19}"}, CSKY::F19_32)
 930:                         .Cases({"{fr20}", "{vr20}"}, CSKY::F20_32)
 931:                         .Cases({"{fr21}", "{vr21}"}, CSKY::F21_32)
 932:                         .Cases({"{fr22}", "{vr22}"}, CSKY::F22_32)
 933:                         .Cases({"{fr23}", "{vr23}"}, CSKY::F23_32)
 934:                         .Cases({"{fr24}", "{vr24}"}, CSKY::F24_32)
 935:                         .Cases({"{fr25}", "{vr25}"}, CSKY::F25_32)
 936:                         .Cases({"{fr26}", "{vr26}"}, CSKY::F26_32)
 937:                         .Cases({"{fr27}", "{vr27}"}, CSKY::F27_32)
 938:                         .Cases({"{fr28}", "{vr28}"}, CSKY::F28_32)
 939:                         .Cases({"{fr29}", "{vr29}"}, CSKY::F29_32)
 940:                         .Cases({"{fr30}", "{vr30}"}, CSKY::F30_32)
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 941-960

```cpp
 941:                         .Cases({"{fr31}", "{vr31}"}, CSKY::F31_32)
 942:                         .Default(CSKY::NoRegister);
 943:     if (FReg != CSKY::NoRegister) {
 944:       assert(CSKY::F0_32 <= FReg && FReg <= CSKY::F31_32 && "Unknown fp-reg");
 945:       unsigned RegNo = FReg - CSKY::F0_32;
 946:       unsigned DReg = CSKY::F0_64 + RegNo;
 947: 
 948:       if (Subtarget.hasFPUv2DoubleFloat())
 949:         return std::make_pair(DReg, &CSKY::sFPR64RegClass);
 950:       else if (Subtarget.hasFPUv3DoubleFloat())
 951:         return std::make_pair(DReg, &CSKY::FPR64RegClass);
 952:       else if (Subtarget.hasFPUv2SingleFloat())
 953:         return std::make_pair(FReg, &CSKY::sFPR32RegClass);
 954:       else if (Subtarget.hasFPUv3SingleFloat())
 955:         return std::make_pair(FReg, &CSKY::FPR32RegClass);
 956:     }
 957:   }
 958: 
 959:   return TargetLowering::getRegForInlineAsmConstraint(TRI, Constraint, VT);
 960: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 961-980

```cpp
 961: 
 962: static MachineBasicBlock *
 963: emitSelectPseudo(MachineInstr &MI, MachineBasicBlock *BB, unsigned Opcode) {
 964: 
 965:   const TargetInstrInfo &TII = *BB->getParent()->getSubtarget().getInstrInfo();
 966:   DebugLoc DL = MI.getDebugLoc();
 967: 
 968:   // To "insert" a SELECT instruction, we actually have to insert the
 969:   // diamond control-flow pattern.  The incoming instruction knows the
 970:   // destination vreg to set, the condition code register to branch on, the
 971:   // true/false values to select between, and a branch opcode to use.
 972:   const BasicBlock *LLVM_BB = BB->getBasicBlock();
 973:   MachineFunction::iterator It = ++BB->getIterator();
 974: 
 975:   //  thisMBB:
 976:   //  ...
 977:   //   TrueVal = ...
 978:   //   bt32 c, sinkMBB
 979:   //   fallthrough --> copyMBB
 980:   MachineBasicBlock *thisMBB = BB;
```

- EN: Function bodies or method definitions such as emitSelectPseudo contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: emitSelectPseudo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 981-1000

```cpp
 981:   MachineFunction *F = BB->getParent();
 982:   MachineBasicBlock *copyMBB = F->CreateMachineBasicBlock(LLVM_BB);
 983:   MachineBasicBlock *sinkMBB = F->CreateMachineBasicBlock(LLVM_BB);
 984:   F->insert(It, copyMBB);
 985:   F->insert(It, sinkMBB);
 986: 
 987:   // Transfer the remainder of BB and its successor edges to sinkMBB.
 988:   sinkMBB->splice(sinkMBB->begin(), BB,
 989:                   std::next(MachineBasicBlock::iterator(MI)), BB->end());
 990:   sinkMBB->transferSuccessorsAndUpdatePHIs(BB);
 991: 
 992:   // Next, add the true and fallthrough blocks as its successors.
 993:   BB->addSuccessor(copyMBB);
 994:   BB->addSuccessor(sinkMBB);
 995: 
 996:   // bt32 condition, sinkMBB
 997:   BuildMI(BB, DL, TII.get(Opcode))
 998:       .addReg(MI.getOperand(1).getReg())
 999:       .addMBB(sinkMBB);
1000: 
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1001-1020

```cpp
1001:   //  copyMBB:
1002:   //   %FalseValue = ...
1003:   //   # fallthrough to sinkMBB
1004:   BB = copyMBB;
1005: 
1006:   // Update machine-CFG edges
1007:   BB->addSuccessor(sinkMBB);
1008: 
1009:   //  sinkMBB:
1010:   //   %Result = phi [ %TrueValue, thisMBB ], [ %FalseValue, copyMBB ]
1011:   //  ...
1012:   BB = sinkMBB;
1013: 
1014:   BuildMI(*BB, BB->begin(), DL, TII.get(CSKY::PHI), MI.getOperand(0).getReg())
1015:       .addReg(MI.getOperand(2).getReg())
1016:       .addMBB(thisMBB)
1017:       .addReg(MI.getOperand(3).getReg())
1018:       .addMBB(copyMBB);
1019: 
1020:   MI.eraseFromParent(); // The pseudo instruction is gone now.
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1021-1040

```cpp
1021: 
1022:   return BB;
1023: }
1024: 
1025: MachineBasicBlock *
1026: CSKYTargetLowering::EmitInstrWithCustomInserter(MachineInstr &MI,
1027:                                                 MachineBasicBlock *BB) const {
1028:   switch (MI.getOpcode()) {
1029:   default:
1030:     llvm_unreachable("Unexpected instr type to insert");
1031:   case CSKY::FSELS:
1032:   case CSKY::FSELD:
1033:     if (Subtarget.hasE2())
1034:       return emitSelectPseudo(MI, BB, CSKY::BT32);
1035:     else
1036:       return emitSelectPseudo(MI, BB, CSKY::BT16);
1037:   case CSKY::ISEL32:
1038:     return emitSelectPseudo(MI, BB, CSKY::BT32);
1039:   case CSKY::ISEL16:
1040:     return emitSelectPseudo(MI, BB, CSKY::BT16);
```

- EN: Function bodies or method definitions such as EmitInstrWithCustomInserter contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: EmitInstrWithCustomInserter 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1041-1060

```cpp
1041:   }
1042: }
1043: 
1044: SDValue CSKYTargetLowering::getTargetConstantPoolValue(ExternalSymbolSDNode *N,
1045:                                                        EVT Ty,
1046:                                                        SelectionDAG &DAG,
1047:                                                        unsigned Flags) const {
1048:   CSKYConstantPoolValue *CPV =
1049:       CSKYConstantPoolSymbol::Create(Type::getInt32Ty(*DAG.getContext()),
1050:                                      N->getSymbol(), 0, getModifier(Flags));
1051: 
1052:   return DAG.getTargetConstantPool(CPV, Ty);
1053: }
1054: 
1055: SDValue CSKYTargetLowering::getTargetConstantPoolValue(JumpTableSDNode *N,
1056:                                                        EVT Ty,
1057:                                                        SelectionDAG &DAG,
1058:                                                        unsigned Flags) const {
1059:   CSKYConstantPoolValue *CPV =
1060:       CSKYConstantPoolJT::Create(Type::getInt32Ty(*DAG.getContext()),
```

- EN: Function bodies or method definitions such as getTargetConstantPoolValue contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getTargetConstantPoolValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1061-1080

```cpp
1061:                                  N->getIndex(), 0, getModifier(Flags));
1062:   return DAG.getTargetConstantPool(CPV, Ty);
1063: }
1064: 
1065: SDValue CSKYTargetLowering::getTargetConstantPoolValue(BlockAddressSDNode *N,
1066:                                                        EVT Ty,
1067:                                                        SelectionDAG &DAG,
1068:                                                        unsigned Flags) const {
1069:   assert(N->getOffset() == 0);
1070:   CSKYConstantPoolValue *CPV = CSKYConstantPoolConstant::Create(
1071:       N->getBlockAddress(), CSKYCP::CPBlockAddress, 0, getModifier(Flags),
1072:       false);
1073:   return DAG.getTargetConstantPool(CPV, Ty);
1074: }
1075: 
1076: SDValue CSKYTargetLowering::getTargetConstantPoolValue(ConstantPoolSDNode *N,
1077:                                                        EVT Ty,
1078:                                                        SelectionDAG &DAG,
1079:                                                        unsigned Flags) const {
1080:   assert(N->getOffset() == 0);
```

- EN: Function bodies or method definitions such as getTargetConstantPoolValue contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getTargetConstantPoolValue 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1081-1100

```cpp
1081:   CSKYConstantPoolValue *CPV = CSKYConstantPoolConstant::Create(
1082:       N->getConstVal(), Type::getInt32Ty(*DAG.getContext()),
1083:       CSKYCP::CPConstPool, 0, getModifier(Flags), false);
1084:   return DAG.getTargetConstantPool(CPV, Ty);
1085: }
1086: 
1087: SDValue CSKYTargetLowering::getTargetNode(GlobalAddressSDNode *N, SDLoc DL,
1088:                                           EVT Ty, SelectionDAG &DAG,
1089:                                           unsigned Flags) const {
1090:   return DAG.getTargetGlobalAddress(N->getGlobal(), DL, Ty, 0, Flags);
1091: }
1092: 
1093: SDValue CSKYTargetLowering::getTargetNode(ExternalSymbolSDNode *N, SDLoc DL,
1094:                                           EVT Ty, SelectionDAG &DAG,
1095:                                           unsigned Flags) const {
1096:   return DAG.getTargetExternalSymbol(N->getSymbol(), Ty, Flags);
1097: }
1098: 
1099: SDValue CSKYTargetLowering::getTargetNode(JumpTableSDNode *N, SDLoc DL, EVT Ty,
1100:                                           SelectionDAG &DAG,
```

- EN: Function bodies or method definitions such as getTargetNode contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getTargetNode 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1101-1120

```cpp
1101:                                           unsigned Flags) const {
1102:   return DAG.getTargetJumpTable(N->getIndex(), Ty, Flags);
1103: }
1104: 
1105: SDValue CSKYTargetLowering::getTargetNode(BlockAddressSDNode *N, SDLoc DL,
1106:                                           EVT Ty, SelectionDAG &DAG,
1107:                                           unsigned Flags) const {
1108:   return DAG.getTargetBlockAddress(N->getBlockAddress(), Ty, N->getOffset(),
1109:                                    Flags);
1110: }
1111: 
1112: SDValue CSKYTargetLowering::getTargetNode(ConstantPoolSDNode *N, SDLoc DL,
1113:                                           EVT Ty, SelectionDAG &DAG,
1114:                                           unsigned Flags) const {
1115: 
1116:   return DAG.getTargetConstantPool(N->getConstVal(), Ty, N->getAlign(),
1117:                                    N->getOffset(), Flags);
1118: }
1119: 
1120: SDValue CSKYTargetLowering::LowerGlobalAddress(SDValue Op,
```

- EN: Function bodies or method definitions such as getTargetNode contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getTargetNode 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1121-1140

```cpp
1121:                                                SelectionDAG &DAG) const {
1122:   SDLoc DL(Op);
1123:   EVT Ty = Op.getValueType();
1124:   GlobalAddressSDNode *N = cast<GlobalAddressSDNode>(Op);
1125:   int64_t Offset = N->getOffset();
1126: 
1127:   const GlobalValue *GV = N->getGlobal();
1128:   bool IsLocal = getTargetMachine().shouldAssumeDSOLocal(GV);
1129:   SDValue Addr = getAddr<GlobalAddressSDNode, false>(N, DAG, IsLocal);
1130: 
1131:   // In order to maximise the opportunity for common subexpression elimination,
1132:   // emit a separate ADD node for the global address offset instead of folding
1133:   // it in the global address node. Later peephole optimisations may choose to
1134:   // fold it back in when profitable.
1135:   if (Offset != 0)
1136:     return DAG.getNode(ISD::ADD, DL, Ty, Addr,
1137:                        DAG.getConstant(Offset, DL, MVT::i32));
1138:   return Addr;
1139: }
1140: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1141-1160

```cpp
1141: SDValue CSKYTargetLowering::LowerExternalSymbol(SDValue Op,
1142:                                                 SelectionDAG &DAG) const {
1143:   ExternalSymbolSDNode *N = cast<ExternalSymbolSDNode>(Op);
1144: 
1145:   return getAddr(N, DAG, false);
1146: }
1147: 
1148: SDValue CSKYTargetLowering::LowerJumpTable(SDValue Op,
1149:                                            SelectionDAG &DAG) const {
1150:   JumpTableSDNode *N = cast<JumpTableSDNode>(Op);
1151: 
1152:   return getAddr<JumpTableSDNode, false>(N, DAG);
1153: }
1154: 
1155: SDValue CSKYTargetLowering::LowerBlockAddress(SDValue Op,
1156:                                               SelectionDAG &DAG) const {
1157:   BlockAddressSDNode *N = cast<BlockAddressSDNode>(Op);
1158: 
1159:   return getAddr(N, DAG);
1160: }
```

- EN: Function bodies or method definitions such as LowerExternalSymbol, LowerJumpTable, LowerBlockAddress contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerExternalSymbol, LowerJumpTable, LowerBlockAddress 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1161-1180

```cpp
1161: 
1162: SDValue CSKYTargetLowering::LowerConstantPool(SDValue Op,
1163:                                               SelectionDAG &DAG) const {
1164:   assert(!Subtarget.hasE2());
1165:   ConstantPoolSDNode *N = cast<ConstantPoolSDNode>(Op);
1166: 
1167:   return getAddr(N, DAG);
1168: }
1169: 
1170: SDValue CSKYTargetLowering::LowerVASTART(SDValue Op, SelectionDAG &DAG) const {
1171:   MachineFunction &MF = DAG.getMachineFunction();
1172:   CSKYMachineFunctionInfo *FuncInfo = MF.getInfo<CSKYMachineFunctionInfo>();
1173: 
1174:   SDLoc DL(Op);
1175:   SDValue FI = DAG.getFrameIndex(FuncInfo->getVarArgsFrameIndex(),
1176:                                  getPointerTy(MF.getDataLayout()));
1177: 
1178:   // vastart just stores the address of the VarArgsFrameIndex slot into the
1179:   // memory location argument.
1180:   const Value *SV = cast<SrcValueSDNode>(Op.getOperand(2))->getValue();
```

- EN: Function bodies or method definitions such as LowerConstantPool, LowerVASTART contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerConstantPool, LowerVASTART 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1181-1200

```cpp
1181:   return DAG.getStore(Op.getOperand(0), DL, FI, Op.getOperand(1),
1182:                       MachinePointerInfo(SV));
1183: }
1184: 
1185: SDValue CSKYTargetLowering::LowerFRAMEADDR(SDValue Op,
1186:                                            SelectionDAG &DAG) const {
1187:   const CSKYRegisterInfo &RI = *Subtarget.getRegisterInfo();
1188:   MachineFunction &MF = DAG.getMachineFunction();
1189:   MachineFrameInfo &MFI = MF.getFrameInfo();
1190:   MFI.setFrameAddressIsTaken(true);
1191: 
1192:   EVT VT = Op.getValueType();
1193:   SDLoc dl(Op);
1194:   unsigned Depth = Op.getConstantOperandVal(0);
1195:   Register FrameReg = RI.getFrameRegister(MF);
1196:   SDValue FrameAddr = DAG.getCopyFromReg(DAG.getEntryNode(), dl, FrameReg, VT);
1197:   while (Depth--)
1198:     FrameAddr = DAG.getLoad(VT, dl, DAG.getEntryNode(), FrameAddr,
1199:                             MachinePointerInfo());
1200:   return FrameAddr;
```

- EN: Function bodies or method definitions such as LowerFRAMEADDR contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerFRAMEADDR 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1201-1220

```cpp
1201: }
1202: 
1203: SDValue CSKYTargetLowering::LowerRETURNADDR(SDValue Op,
1204:                                             SelectionDAG &DAG) const {
1205:   const CSKYRegisterInfo &RI = *Subtarget.getRegisterInfo();
1206:   MachineFunction &MF = DAG.getMachineFunction();
1207:   MachineFrameInfo &MFI = MF.getFrameInfo();
1208:   MFI.setReturnAddressIsTaken(true);
1209: 
1210:   EVT VT = Op.getValueType();
1211:   SDLoc dl(Op);
1212:   unsigned Depth = Op.getConstantOperandVal(0);
1213:   if (Depth) {
1214:     SDValue FrameAddr = LowerFRAMEADDR(Op, DAG);
1215:     SDValue Offset = DAG.getConstant(4, dl, MVT::i32);
1216:     return DAG.getLoad(VT, dl, DAG.getEntryNode(),
1217:                        DAG.getNode(ISD::ADD, dl, VT, FrameAddr, Offset),
1218:                        MachinePointerInfo());
1219:   }
1220:   // Return the value of the return address register, marking it an implicit
```

- EN: Function bodies or method definitions such as LowerRETURNADDR contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerRETURNADDR 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1221-1240

```cpp
1221:   // live-in.
1222:   unsigned Reg = MF.addLiveIn(RI.getRARegister(), getRegClassFor(MVT::i32));
1223:   return DAG.getCopyFromReg(DAG.getEntryNode(), dl, Reg, VT);
1224: }
1225: 
1226: Register CSKYTargetLowering::getExceptionPointerRegister(
1227:     const Constant *PersonalityFn) const {
1228:   return CSKY::R0;
1229: }
1230: 
1231: Register CSKYTargetLowering::getExceptionSelectorRegister(
1232:     const Constant *PersonalityFn) const {
1233:   return CSKY::R1;
1234: }
1235: 
1236: SDValue CSKYTargetLowering::LowerGlobalTLSAddress(SDValue Op,
1237:                                                   SelectionDAG &DAG) const {
1238:   SDLoc DL(Op);
1239:   EVT Ty = Op.getValueType();
1240:   GlobalAddressSDNode *N = cast<GlobalAddressSDNode>(Op);
```

- EN: Function bodies or method definitions such as getExceptionPointerRegister, getExceptionSelectorRegister, LowerGlobalTLSAddress contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getExceptionPointerRegister, getExceptionSelectorRegister, LowerGlobalTLSAddress 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1241-1260

```cpp
1241:   int64_t Offset = N->getOffset();
1242:   MVT XLenVT = MVT::i32;
1243: 
1244:   TLSModel::Model Model = getTargetMachine().getTLSModel(N->getGlobal());
1245:   SDValue Addr;
1246:   switch (Model) {
1247:   case TLSModel::LocalExec:
1248:     Addr = getStaticTLSAddr(N, DAG, /*UseGOT=*/false);
1249:     break;
1250:   case TLSModel::InitialExec:
1251:     Addr = getStaticTLSAddr(N, DAG, /*UseGOT=*/true);
1252:     break;
1253:   case TLSModel::LocalDynamic:
1254:   case TLSModel::GeneralDynamic:
1255:     Addr = getDynamicTLSAddr(N, DAG);
1256:     break;
1257:   }
1258: 
1259:   // In order to maximise the opportunity for common subexpression elimination,
1260:   // emit a separate ADD node for the global address offset instead of folding
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1261-1280

```cpp
1261:   // it in the global address node. Later peephole optimisations may choose to
1262:   // fold it back in when profitable.
1263:   if (Offset != 0)
1264:     return DAG.getNode(ISD::ADD, DL, Ty, Addr,
1265:                        DAG.getConstant(Offset, DL, XLenVT));
1266:   return Addr;
1267: }
1268: 
1269: SDValue CSKYTargetLowering::getStaticTLSAddr(GlobalAddressSDNode *N,
1270:                                              SelectionDAG &DAG,
1271:                                              bool UseGOT) const {
1272:   MachineFunction &MF = DAG.getMachineFunction();
1273:   CSKYMachineFunctionInfo *CFI = MF.getInfo<CSKYMachineFunctionInfo>();
1274: 
1275:   unsigned CSKYPCLabelIndex = CFI->createPICLabelUId();
1276: 
1277:   SDLoc DL(N);
1278:   EVT Ty = getPointerTy(DAG.getDataLayout());
1279: 
1280:   CSKYCP::CSKYCPModifier Flag = UseGOT ? CSKYCP::TLSIE : CSKYCP::TLSLE;
```

- EN: Function bodies or method definitions such as getStaticTLSAddr contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getStaticTLSAddr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1281-1300

```cpp
1281:   bool AddCurrentAddr = UseGOT ? true : false;
1282:   unsigned char PCAjust = UseGOT ? 4 : 0;
1283: 
1284:   CSKYConstantPoolValue *CPV =
1285:       CSKYConstantPoolConstant::Create(N->getGlobal(), CSKYCP::CPValue, PCAjust,
1286:                                        Flag, AddCurrentAddr, CSKYPCLabelIndex);
1287:   SDValue CAddr = DAG.getTargetConstantPool(CPV, Ty);
1288: 
1289:   SDValue Load;
1290:   if (UseGOT) {
1291:     SDValue PICLabel = DAG.getTargetConstant(CSKYPCLabelIndex, DL, MVT::i32);
1292:     auto *LRWGRS = DAG.getMachineNode(CSKY::PseudoTLSLA32, DL, {Ty, Ty},
1293:                                       {CAddr, PICLabel});
1294:     auto LRWADDGRS =
1295:         DAG.getNode(ISD::ADD, DL, Ty, SDValue(LRWGRS, 0), SDValue(LRWGRS, 1));
1296:     Load = DAG.getLoad(Ty, DL, DAG.getEntryNode(), LRWADDGRS,
1297:                        MachinePointerInfo(N->getGlobal()));
1298:   } else {
1299:     Load = SDValue(DAG.getMachineNode(CSKY::LRW32, DL, Ty, CAddr), 0);
1300:   }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1301-1320

```cpp
1301: 
1302:   // Add the thread pointer.
1303:   SDValue TPReg = DAG.getRegister(CSKY::R31, MVT::i32);
1304:   return DAG.getNode(ISD::ADD, DL, Ty, Load, TPReg);
1305: }
1306: 
1307: SDValue CSKYTargetLowering::getDynamicTLSAddr(GlobalAddressSDNode *N,
1308:                                               SelectionDAG &DAG) const {
1309:   MachineFunction &MF = DAG.getMachineFunction();
1310:   CSKYMachineFunctionInfo *CFI = MF.getInfo<CSKYMachineFunctionInfo>();
1311: 
1312:   unsigned CSKYPCLabelIndex = CFI->createPICLabelUId();
1313: 
1314:   SDLoc DL(N);
1315:   EVT Ty = getPointerTy(DAG.getDataLayout());
1316:   IntegerType *CallTy = Type::getIntNTy(*DAG.getContext(), Ty.getSizeInBits());
1317: 
1318:   CSKYConstantPoolValue *CPV =
1319:       CSKYConstantPoolConstant::Create(N->getGlobal(), CSKYCP::CPValue, 4,
1320:                                        CSKYCP::TLSGD, true, CSKYPCLabelIndex);
```

- EN: Function bodies or method definitions such as getDynamicTLSAddr contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getDynamicTLSAddr 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1321-1340

```cpp
1321:   SDValue Addr = DAG.getTargetConstantPool(CPV, Ty);
1322:   SDValue PICLabel = DAG.getTargetConstant(CSKYPCLabelIndex, DL, MVT::i32);
1323: 
1324:   auto *LRWGRS =
1325:       DAG.getMachineNode(CSKY::PseudoTLSLA32, DL, {Ty, Ty}, {Addr, PICLabel});
1326: 
1327:   auto Load =
1328:       DAG.getNode(ISD::ADD, DL, Ty, SDValue(LRWGRS, 0), SDValue(LRWGRS, 1));
1329: 
1330:   // Prepare argument list to generate call.
1331:   ArgListTy Args;
1332:   Args.emplace_back(Load, CallTy);
1333: 
1334:   // Setup call to __tls_get_addr.
1335:   TargetLowering::CallLoweringInfo CLI(DAG);
1336:   CLI.setDebugLoc(DL)
1337:       .setChain(DAG.getEntryNode())
1338:       .setLibCallee(CallingConv::C, CallTy,
1339:                     DAG.getExternalSymbol("__tls_get_addr", Ty),
1340:                     std::move(Args));
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1341-1360

```cpp
1341:   SDValue V = LowerCallTo(CLI).first;
1342: 
1343:   return V;
1344: }
1345: 
1346: bool CSKYTargetLowering::decomposeMulByConstant(LLVMContext &Context, EVT VT,
1347:                                                 SDValue C) const {
1348:   if (!VT.isScalarInteger())
1349:     return false;
1350: 
1351:   // Omit if data size exceeds.
1352:   if (VT.getSizeInBits() > Subtarget.XLen)
1353:     return false;
1354: 
1355:   if (auto *ConstNode = dyn_cast<ConstantSDNode>(C.getNode())) {
1356:     const APInt &Imm = ConstNode->getAPIntValue();
1357:     // Break MULT to LSLI + ADDU/SUBU.
1358:     if ((Imm + 1).isPowerOf2() || (Imm - 1).isPowerOf2() ||
1359:         (1 - Imm).isPowerOf2())
1360:       return true;
```

- EN: Function bodies or method definitions such as decomposeMulByConstant contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: decomposeMulByConstant 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1361-1380

```cpp
1361:     // Only break MULT for sub targets without MULT32, since an extra
1362:     // instruction will be generated against the above 3 cases. We leave it
1363:     // unchanged on sub targets with MULT32, since not sure it is better.
1364:     if (!Subtarget.hasE2() && (-1 - Imm).isPowerOf2())
1365:       return true;
1366:     // Break (MULT x, imm) to ([IXH32|IXW32|IXD32] (LSLI32 x, i0), x) when
1367:     // imm=(1<<i0)+[2|4|8] and imm has to be composed via a MOVIH32/ORI32 pair.
1368:     if (Imm.ugt(0xffff) && ((Imm - 2).isPowerOf2() || (Imm - 4).isPowerOf2()) &&
1369:         Subtarget.hasE2())
1370:       return true;
1371:     if (Imm.ugt(0xffff) && (Imm - 8).isPowerOf2() && Subtarget.has2E3())
1372:       return true;
1373:   }
1374: 
1375:   return false;
1376: }
1377: 
1378: bool CSKYTargetLowering::isCheapToSpeculateCttz(Type *Ty) const {
1379:   return Subtarget.has2E3();
1380: }
```

- EN: Function bodies or method definitions such as isCheapToSpeculateCttz contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: isCheapToSpeculateCttz 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1381-1384

```cpp
1381: 
1382: bool CSKYTargetLowering::isCheapToSpeculateCtlz(Type *Ty) const {
1383:   return Subtarget.hasE2();
1384: }
```

- EN: Function bodies or method definitions such as isCheapToSpeculateCtlz contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: isCheapToSpeculateCtlz 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

## Key Concepts / 关键概念

- Custom DAG legalization / 自定义 DAG 合法化
- Calling-lowering hooks / 调用降级钩子
- SelectionDAG lowering / SelectionDAG 降级
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Basic block level transformation / 基本块级转换
- Register classes / 寄存器类
- Instruction semantics helpers / 指令语义辅助逻辑

## Dependencies / 依赖关系

- Direct includes / 直接包含: `CSKYISelLowering.h`, `CSKYCallingConv.h`, `CSKYConstantPoolValue.h`, `CSKYMachineFunctionInfo.h`, `CSKYRegisterInfo.h`, `CSKYSubtarget.h`, `llvm/ADT/Statistic.h`, `llvm/CodeGen/CallingConvLower.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, SelectionDAG
- Generated or companion files / 生成或配套文件: `CSKYGenCallingConv.inc`
- Local companions / 本地配套文件: `CSKYISelLowering.h`
