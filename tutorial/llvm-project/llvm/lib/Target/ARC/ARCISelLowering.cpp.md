# ARCISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/ARC/ARCISelLowering.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file implements the ARCTargetLowering class.
- 目的（中文）: 实现 SelectionDAG 降级钩子，将通用 LLVM IR 操作映射到目标专用 DAG 节点与调用约定。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===- ARCISelLowering.cpp - ARC DAG Lowering Impl --------------*- C++ -*-===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements the ARCTargetLowering class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "ARCISelLowering.h"
  14: #include "ARC.h"
  15: #include "ARCMachineFunctionInfo.h"
  16: #include "ARCSelectionDAGInfo.h"
  17: #include "ARCSubtarget.h"
  18: #include "ARCTargetMachine.h"
  19: #include "MCTargetDesc/ARCInfo.h"
  20: #include "llvm/CodeGen/CallingConvLower.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/MachineFrameInfo.h"
  22: #include "llvm/CodeGen/MachineFunction.h"
  23: #include "llvm/CodeGen/MachineInstrBuilder.h"
  24: #include "llvm/CodeGen/MachineJumpTableInfo.h"
  25: #include "llvm/CodeGen/MachineRegisterInfo.h"
  26: #include "llvm/CodeGen/ValueTypes.h"
  27: #include "llvm/IR/CallingConv.h"
  28: #include "llvm/IR/Intrinsics.h"
  29: #include "llvm/Support/Debug.h"
  30: #include <algorithm>
  31: 
  32: #define DEBUG_TYPE "arc-lower"
  33: 
  34: using namespace llvm;
  35: 
  36: static SDValue lowerCallResult(SDValue Chain, SDValue InGlue,
  37:                                const SmallVectorImpl<CCValAssign> &RVLocs,
  38:                                SDLoc dl, SelectionDAG &DAG,
  39:                                SmallVectorImpl<SDValue> &InVals);
  40: 
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 41-60

```cpp
  41: static ARCCC::CondCode ISDCCtoARCCC(ISD::CondCode isdCC) {
  42:   switch (isdCC) {
  43:   case ISD::SETUEQ:
  44:     return ARCCC::EQ;
  45:   case ISD::SETUGT:
  46:     return ARCCC::HI;
  47:   case ISD::SETUGE:
  48:     return ARCCC::HS;
  49:   case ISD::SETULT:
  50:     return ARCCC::LO;
  51:   case ISD::SETULE:
  52:     return ARCCC::LS;
  53:   case ISD::SETUNE:
  54:     return ARCCC::NE;
  55:   case ISD::SETEQ:
  56:     return ARCCC::EQ;
  57:   case ISD::SETGT:
  58:     return ARCCC::GT;
  59:   case ISD::SETGE:
  60:     return ARCCC::GE;
```

- EN: Function bodies or method definitions such as ISDCCtoARCCC contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: ISDCCtoARCCC 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 61-80

```cpp
  61:   case ISD::SETLT:
  62:     return ARCCC::LT;
  63:   case ISD::SETLE:
  64:     return ARCCC::LE;
  65:   case ISD::SETNE:
  66:     return ARCCC::NE;
  67:   default:
  68:     llvm_unreachable("Unhandled ISDCC code.");
  69:   }
  70: }
  71: 
  72: void ARCTargetLowering::ReplaceNodeResults(SDNode *N,
  73:                                            SmallVectorImpl<SDValue> &Results,
  74:                                            SelectionDAG &DAG) const {
  75:   LLVM_DEBUG(dbgs() << "[ARC-ISEL] ReplaceNodeResults ");
  76:   LLVM_DEBUG(N->dump(&DAG));
  77:   LLVM_DEBUG(dbgs() << "; use_count=" << N->use_size() << "\n");
  78: 
  79:   switch (N->getOpcode()) {
  80:   case ISD::READCYCLECOUNTER:
```

- EN: Function bodies or method definitions such as ReplaceNodeResults contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: ReplaceNodeResults 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 81-100

```cpp
  81:     if (N->getValueType(0) == MVT::i64) {
  82:       // We read the TIMER0 and zero-extend it to 64-bits as the intrinsic
  83:       // requires.
  84:       SDValue V =
  85:           DAG.getNode(ISD::READCYCLECOUNTER, SDLoc(N),
  86:                       DAG.getVTList(MVT::i32, MVT::Other), N->getOperand(0));
  87:       SDValue Op = DAG.getNode(ISD::ZERO_EXTEND, SDLoc(N), MVT::i64, V);
  88:       Results.push_back(Op);
  89:       Results.push_back(V.getValue(1));
  90:     }
  91:     break;
  92:   default:
  93:     break;
  94:   }
  95: }
  96: 
  97: ARCTargetLowering::ARCTargetLowering(const TargetMachine &TM,
  98:                                      const ARCSubtarget &Subtarget)
  99:     : TargetLowering(TM, Subtarget), Subtarget(Subtarget) {
 100:   // Set up the register classes.
```

- EN: Function bodies or method definitions such as TargetLowering contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: TargetLowering 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 101-120

```cpp
 101:   addRegisterClass(MVT::i32, &ARC::GPR32RegClass);
 102: 
 103:   // Compute derived properties from the register classes
 104:   computeRegisterProperties(Subtarget.getRegisterInfo());
 105: 
 106:   setStackPointerRegisterToSaveRestore(ARC::SP);
 107: 
 108:   setSchedulingPreference(Sched::Source);
 109: 
 110:   // Use i32 for setcc operations results (slt, sgt, ...).
 111:   setBooleanContents(ZeroOrOneBooleanContent);
 112:   setBooleanVectorContents(ZeroOrOneBooleanContent);
 113: 
 114:   for (unsigned Opc = 0; Opc < ISD::BUILTIN_OP_END; ++Opc)
 115:     setOperationAction(Opc, MVT::i32, Expand);
 116: 
 117:   // Operations to get us off of the ground.
 118:   // Basic.
 119:   setOperationAction(ISD::ADD, MVT::i32, Legal);
 120:   setOperationAction(ISD::SUB, MVT::i32, Legal);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 121-140

```cpp
 121:   setOperationAction(ISD::AND, MVT::i32, Legal);
 122:   setOperationAction(ISD::SMAX, MVT::i32, Legal);
 123:   setOperationAction(ISD::SMIN, MVT::i32, Legal);
 124: 
 125:   setOperationAction(ISD::ADDC, MVT::i32, Legal);
 126:   setOperationAction(ISD::ADDE, MVT::i32, Legal);
 127:   setOperationAction(ISD::SUBC, MVT::i32, Legal);
 128:   setOperationAction(ISD::SUBE, MVT::i32, Legal);
 129: 
 130:   // Need barrel shifter.
 131:   setOperationAction(ISD::SHL, MVT::i32, Legal);
 132:   setOperationAction(ISD::SRA, MVT::i32, Legal);
 133:   setOperationAction(ISD::SRL, MVT::i32, Legal);
 134:   setOperationAction(ISD::ROTR, MVT::i32, Legal);
 135: 
 136:   setOperationAction(ISD::Constant, MVT::i32, Legal);
 137:   setOperationAction(ISD::UNDEF, MVT::i32, Legal);
 138: 
 139:   // Need multiplier
 140:   setOperationAction(ISD::MUL, MVT::i32, Legal);
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 141-160

```cpp
 141:   setOperationAction(ISD::MULHS, MVT::i32, Legal);
 142:   setOperationAction(ISD::MULHU, MVT::i32, Legal);
 143:   setOperationAction(ISD::LOAD, MVT::i32, Legal);
 144:   setOperationAction(ISD::STORE, MVT::i32, Legal);
 145: 
 146:   setOperationAction(ISD::SELECT_CC, MVT::i32, Custom);
 147:   setOperationAction(ISD::BR_CC, MVT::i32, Custom);
 148:   setOperationAction(ISD::BRCOND, MVT::Other, Expand);
 149:   setOperationAction(ISD::BR_JT, MVT::Other, Expand);
 150:   setOperationAction(ISD::JumpTable, MVT::i32, Custom);
 151: 
 152:   // Have pseudo instruction for frame addresses.
 153:   setOperationAction(ISD::FRAMEADDR, MVT::i32, Legal);
 154:   // Custom lower global addresses.
 155:   setOperationAction(ISD::GlobalAddress, MVT::i32, Custom);
 156: 
 157:   // Expand var-args ops.
 158:   setOperationAction(ISD::VASTART, MVT::Other, Custom);
 159:   setOperationAction(ISD::VAEND, MVT::Other, Expand);
 160:   setOperationAction(ISD::VAARG, MVT::Other, Expand);
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 161-180

```cpp
 161:   setOperationAction(ISD::VACOPY, MVT::Other, Expand);
 162: 
 163:   // Other expansions
 164:   setOperationAction(ISD::STACKSAVE, MVT::Other, Expand);
 165:   setOperationAction(ISD::STACKRESTORE, MVT::Other, Expand);
 166: 
 167:   // Sign extend inreg
 168:   setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i1, Custom);
 169: 
 170:   // TODO: Predicate these with `options.hasBitScan() ? Legal : Expand`
 171:   //       when the HasBitScan predicate is available.
 172:   setOperationAction(ISD::CTLZ, MVT::i32, Legal);
 173:   setOperationAction(ISD::CTTZ, MVT::i32, Legal);
 174: 
 175:   setOperationAction(ISD::READCYCLECOUNTER, MVT::i32, Legal);
 176:   setOperationAction(ISD::READCYCLECOUNTER, MVT::i64,
 177:                      isTypeLegal(MVT::i64) ? Legal : Custom);
 178: 
 179:   setMaxAtomicSizeInBitsSupported(0);
 180: }
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 181-200

```cpp
 181: 
 182: //===----------------------------------------------------------------------===//
 183: //  Misc Lower Operation implementation
 184: //===----------------------------------------------------------------------===//
 185: 
 186: SDValue ARCTargetLowering::LowerSELECT_CC(SDValue Op, SelectionDAG &DAG) const {
 187:   SDValue LHS = Op.getOperand(0);
 188:   SDValue RHS = Op.getOperand(1);
 189:   ISD::CondCode CC = cast<CondCodeSDNode>(Op.getOperand(4))->get();
 190:   SDValue TVal = Op.getOperand(2);
 191:   SDValue FVal = Op.getOperand(3);
 192:   SDLoc dl(Op);
 193:   ARCCC::CondCode ArcCC = ISDCCtoARCCC(CC);
 194:   assert(LHS.getValueType() == MVT::i32 && "Only know how to SELECT_CC i32");
 195:   SDValue Cmp = DAG.getNode(ARCISD::CMP, dl, MVT::Glue, LHS, RHS);
 196:   return DAG.getNode(ARCISD::CMOV, dl, TVal.getValueType(), TVal, FVal,
 197:                      DAG.getConstant(ArcCC, dl, MVT::i32), Cmp);
 198: }
 199: 
 200: SDValue ARCTargetLowering::LowerSIGN_EXTEND_INREG(SDValue Op,
```

- EN: Function bodies or method definitions such as LowerSELECT_CC contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerSELECT_CC 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 201-220

```cpp
 201:                                                   SelectionDAG &DAG) const {
 202:   SDValue Op0 = Op.getOperand(0);
 203:   SDLoc dl(Op);
 204:   assert(Op.getValueType() == MVT::i32 &&
 205:          "Unhandled target sign_extend_inreg.");
 206:   // These are legal
 207:   unsigned Width = cast<VTSDNode>(Op.getOperand(1))->getVT().getSizeInBits();
 208:   if (Width == 16 || Width == 8)
 209:     return Op;
 210:   if (Width >= 32) {
 211:     return {};
 212:   }
 213:   SDValue LS = DAG.getNode(ISD::SHL, dl, MVT::i32, Op0,
 214:                            DAG.getConstant(32 - Width, dl, MVT::i32));
 215:   SDValue SR = DAG.getNode(ISD::SRA, dl, MVT::i32, LS,
 216:                            DAG.getConstant(32 - Width, dl, MVT::i32));
 217:   return SR;
 218: }
 219: 
 220: SDValue ARCTargetLowering::LowerBR_CC(SDValue Op, SelectionDAG &DAG) const {
```

- EN: Function bodies or method definitions such as LowerBR_CC contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerBR_CC 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 221-240

```cpp
 221:   SDValue Chain = Op.getOperand(0);
 222:   ISD::CondCode CC = cast<CondCodeSDNode>(Op.getOperand(1))->get();
 223:   SDValue LHS = Op.getOperand(2);
 224:   SDValue RHS = Op.getOperand(3);
 225:   SDValue Dest = Op.getOperand(4);
 226:   SDLoc dl(Op);
 227:   ARCCC::CondCode arcCC = ISDCCtoARCCC(CC);
 228:   assert(LHS.getValueType() == MVT::i32 && "Only know how to BR_CC i32");
 229:   return DAG.getNode(ARCISD::BRcc, dl, MVT::Other, Chain, Dest, LHS, RHS,
 230:                      DAG.getConstant(arcCC, dl, MVT::i32));
 231: }
 232: 
 233: SDValue ARCTargetLowering::LowerJumpTable(SDValue Op, SelectionDAG &DAG) const {
 234:   auto *N = cast<JumpTableSDNode>(Op);
 235:   SDValue GA = DAG.getTargetJumpTable(N->getIndex(), MVT::i32);
 236:   return DAG.getNode(ARCISD::GAWRAPPER, SDLoc(N), MVT::i32, GA);
 237: }
 238: 
 239: #include "ARCGenCallingConv.inc"
 240: 
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as LowerJumpTable contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 LowerJumpTable 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 241-260

```cpp
 241: //===----------------------------------------------------------------------===//
 242: //                  Call Calling Convention Implementation
 243: //===----------------------------------------------------------------------===//
 244: 
 245: /// ARC call implementation
 246: SDValue ARCTargetLowering::LowerCall(TargetLowering::CallLoweringInfo &CLI,
 247:                                      SmallVectorImpl<SDValue> &InVals) const {
 248:   SelectionDAG &DAG = CLI.DAG;
 249:   SDLoc &dl = CLI.DL;
 250:   SmallVectorImpl<ISD::OutputArg> &Outs = CLI.Outs;
 251:   SmallVectorImpl<SDValue> &OutVals = CLI.OutVals;
 252:   SmallVectorImpl<ISD::InputArg> &Ins = CLI.Ins;
 253:   SDValue Chain = CLI.Chain;
 254:   SDValue Callee = CLI.Callee;
 255:   CallingConv::ID CallConv = CLI.CallConv;
 256:   bool IsVarArg = CLI.IsVarArg;
 257:   bool &IsTailCall = CLI.IsTailCall;
 258: 
 259:   IsTailCall = false; // Do not support tail calls yet.
 260: 
```

- EN: Function bodies or method definitions such as LowerCall contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerCall 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 261-280

```cpp
 261:   SmallVector<CCValAssign, 16> ArgLocs;
 262:   CCState CCInfo(CallConv, IsVarArg, DAG.getMachineFunction(), ArgLocs,
 263:                  *DAG.getContext());
 264: 
 265:   CCInfo.AnalyzeCallOperands(Outs, CC_ARC);
 266: 
 267:   SmallVector<CCValAssign, 16> RVLocs;
 268:   // Analyze return values to determine the number of bytes of stack required.
 269:   CCState RetCCInfo(CallConv, IsVarArg, DAG.getMachineFunction(), RVLocs,
 270:                     *DAG.getContext());
 271:   RetCCInfo.AllocateStack(CCInfo.getStackSize(), Align(4));
 272:   RetCCInfo.AnalyzeCallResult(Ins, RetCC_ARC);
 273: 
 274:   // Get a count of how many bytes are to be pushed on the stack.
 275:   unsigned NumBytes = RetCCInfo.getStackSize();
 276: 
 277:   Chain = DAG.getCALLSEQ_START(Chain, NumBytes, 0, dl);
 278: 
 279:   SmallVector<std::pair<unsigned, SDValue>, 4> RegsToPass;
 280:   SmallVector<SDValue, 12> MemOpChains;
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 281-300

```cpp
 281: 
 282:   SDValue StackPtr;
 283:   // Walk the register/memloc assignments, inserting copies/loads.
 284:   for (unsigned i = 0, e = ArgLocs.size(); i != e; ++i) {
 285:     CCValAssign &VA = ArgLocs[i];
 286:     SDValue Arg = OutVals[i];
 287: 
 288:     // Promote the value if needed.
 289:     switch (VA.getLocInfo()) {
 290:     default:
 291:       llvm_unreachable("Unknown loc info!");
 292:     case CCValAssign::Full:
 293:       break;
 294:     case CCValAssign::SExt:
 295:       Arg = DAG.getNode(ISD::SIGN_EXTEND, dl, VA.getLocVT(), Arg);
 296:       break;
 297:     case CCValAssign::ZExt:
 298:       Arg = DAG.getNode(ISD::ZERO_EXTEND, dl, VA.getLocVT(), Arg);
 299:       break;
 300:     case CCValAssign::AExt:
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 301-320

```cpp
 301:       Arg = DAG.getNode(ISD::ANY_EXTEND, dl, VA.getLocVT(), Arg);
 302:       break;
 303:     }
 304: 
 305:     // Arguments that can be passed on register must be kept at
 306:     // RegsToPass vector
 307:     if (VA.isRegLoc()) {
 308:       RegsToPass.push_back(std::make_pair(VA.getLocReg(), Arg));
 309:     } else {
 310:       assert(VA.isMemLoc() && "Must be register or memory argument.");
 311:       if (!StackPtr.getNode())
 312:         StackPtr = DAG.getCopyFromReg(Chain, dl, ARC::SP,
 313:                                       getPointerTy(DAG.getDataLayout()));
 314:       // Calculate the stack position.
 315:       SDValue SOffset = DAG.getIntPtrConstant(VA.getLocMemOffset(), dl);
 316:       SDValue PtrOff = DAG.getNode(
 317:           ISD::ADD, dl, getPointerTy(DAG.getDataLayout()), StackPtr, SOffset);
 318: 
 319:       SDValue Store =
 320:           DAG.getStore(Chain, dl, Arg, PtrOff, MachinePointerInfo());
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 321-340

```cpp
 321:       MemOpChains.push_back(Store);
 322:       IsTailCall = false;
 323:     }
 324:   }
 325: 
 326:   // Transform all store nodes into one single node because
 327:   // all store nodes are independent of each other.
 328:   if (!MemOpChains.empty())
 329:     Chain = DAG.getNode(ISD::TokenFactor, dl, MVT::Other, MemOpChains);
 330: 
 331:   // Build a sequence of copy-to-reg nodes chained together with token
 332:   // chain and flag operands which copy the outgoing args into registers.
 333:   // The Glue in necessary since all emitted instructions must be
 334:   // stuck together.
 335:   SDValue Glue;
 336:   for (unsigned i = 0, e = RegsToPass.size(); i != e; ++i) {
 337:     Chain = DAG.getCopyToReg(Chain, dl, RegsToPass[i].first,
 338:                              RegsToPass[i].second, Glue);
 339:     Glue = Chain.getValue(1);
 340:   }
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 341-360

```cpp
 341: 
 342:   // If the callee is a GlobalAddress node (quite common, every direct call is)
 343:   // turn it into a TargetGlobalAddress node so that legalize doesn't hack it.
 344:   // Likewise ExternalSymbol -> TargetExternalSymbol.
 345:   bool IsDirect = true;
 346:   if (auto *G = dyn_cast<GlobalAddressSDNode>(Callee))
 347:     Callee = DAG.getTargetGlobalAddress(G->getGlobal(), dl, MVT::i32);
 348:   else if (auto *E = dyn_cast<ExternalSymbolSDNode>(Callee))
 349:     Callee = DAG.getTargetExternalSymbol(E->getSymbol(), MVT::i32);
 350:   else
 351:     IsDirect = false;
 352:   // Branch + Link = #chain, #target_address, #opt_in_flags...
 353:   //             = Chain, Callee, Reg#1, Reg#2, ...
 354:   //
 355:   // Returns a chain & a glue for retval copy to use.
 356:   SDVTList NodeTys = DAG.getVTList(MVT::Other, MVT::Glue);
 357:   SmallVector<SDValue, 8> Ops;
 358:   Ops.push_back(Chain);
 359:   Ops.push_back(Callee);
 360: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 361-380

```cpp
 361:   for (unsigned i = 0, e = RegsToPass.size(); i != e; ++i)
 362:     Ops.push_back(DAG.getRegister(RegsToPass[i].first,
 363:                                   RegsToPass[i].second.getValueType()));
 364: 
 365:   // Add a register mask operand representing the call-preserved registers.
 366:   const TargetRegisterInfo *TRI = Subtarget.getRegisterInfo();
 367:   const uint32_t *Mask =
 368:       TRI->getCallPreservedMask(DAG.getMachineFunction(), CallConv);
 369:   assert(Mask && "Missing call preserved mask for calling convention");
 370:   Ops.push_back(DAG.getRegisterMask(Mask));
 371: 
 372:   if (Glue.getNode())
 373:     Ops.push_back(Glue);
 374: 
 375:   Chain = DAG.getNode(IsDirect ? ARCISD::BL : ARCISD::JL, dl, NodeTys, Ops);
 376:   Glue = Chain.getValue(1);
 377: 
 378:   // Create the CALLSEQ_END node.
 379:   Chain = DAG.getCALLSEQ_END(Chain, NumBytes, 0, Glue, dl);
 380:   Glue = Chain.getValue(1);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 381-400

```cpp
 381: 
 382:   // Handle result values, copying them out of physregs into vregs that we
 383:   // return.
 384:   if (IsTailCall)
 385:     return Chain;
 386:   return lowerCallResult(Chain, Glue, RVLocs, dl, DAG, InVals);
 387: }
 388: 
 389: /// Lower the result values of a call into the appropriate copies out of
 390: /// physical registers / memory locations.
 391: static SDValue lowerCallResult(SDValue Chain, SDValue Glue,
 392:                                const SmallVectorImpl<CCValAssign> &RVLocs,
 393:                                SDLoc dl, SelectionDAG &DAG,
 394:                                SmallVectorImpl<SDValue> &InVals) {
 395:   SmallVector<std::pair<int, unsigned>, 4> ResultMemLocs;
 396:   // Copy results out of physical registers.
 397:   for (unsigned i = 0, e = RVLocs.size(); i != e; ++i) {
 398:     const CCValAssign &VA = RVLocs[i];
 399:     if (VA.isRegLoc()) {
 400:       SDValue RetValue;
```

- EN: Function bodies or method definitions such as lowerCallResult contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: lowerCallResult 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 401-420

```cpp
 401:       RetValue =
 402:           DAG.getCopyFromReg(Chain, dl, VA.getLocReg(), VA.getValVT(), Glue);
 403:       Chain = RetValue.getValue(1);
 404:       Glue = RetValue.getValue(2);
 405:       InVals.push_back(RetValue);
 406:     } else {
 407:       assert(VA.isMemLoc() && "Must be memory location.");
 408:       ResultMemLocs.push_back(
 409:           std::make_pair(VA.getLocMemOffset(), InVals.size()));
 410: 
 411:       // Reserve space for this result.
 412:       InVals.push_back(SDValue());
 413:     }
 414:   }
 415: 
 416:   // Copy results out of memory.
 417:   SmallVector<SDValue, 4> MemOpChains;
 418:   for (unsigned i = 0, e = ResultMemLocs.size(); i != e; ++i) {
 419:     int Offset = ResultMemLocs[i].first;
 420:     unsigned Index = ResultMemLocs[i].second;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 421-440

```cpp
 421:     SDValue StackPtr = DAG.getRegister(ARC::SP, MVT::i32);
 422:     SDValue SpLoc = DAG.getNode(ISD::ADD, dl, MVT::i32, StackPtr,
 423:                                 DAG.getConstant(Offset, dl, MVT::i32));
 424:     SDValue Load =
 425:         DAG.getLoad(MVT::i32, dl, Chain, SpLoc, MachinePointerInfo());
 426:     InVals[Index] = Load;
 427:     MemOpChains.push_back(Load.getValue(1));
 428:   }
 429: 
 430:   // Transform all loads nodes into one single node because
 431:   // all load nodes are independent of each other.
 432:   if (!MemOpChains.empty())
 433:     Chain = DAG.getNode(ISD::TokenFactor, dl, MVT::Other, MemOpChains);
 434: 
 435:   return Chain;
 436: }
 437: 
 438: //===----------------------------------------------------------------------===//
 439: //             Formal Arguments Calling Convention Implementation
 440: //===----------------------------------------------------------------------===//
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 441-460

```cpp
 441: 
 442: namespace {
 443: 
 444: struct ArgDataPair {
 445:   SDValue SDV;
 446:   ISD::ArgFlagsTy Flags;
 447: };
 448: 
 449: } // end anonymous namespace
 450: 
 451: /// ARC formal arguments implementation
 452: SDValue ARCTargetLowering::LowerFormalArguments(
 453:     SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
 454:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &dl,
 455:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
 456:   switch (CallConv) {
 457:   default:
 458:     llvm_unreachable("Unsupported calling convention");
 459:   case CallingConv::C:
 460:   case CallingConv::Fast:
```

- EN: This chunk introduces interfaces or data structures such as ArgDataPair, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as LowerFormalArguments contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这一段引入了 ArgDataPair 等接口或数据结构，用于组织该文件暴露的目标专用行为。 LowerFormalArguments 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 461-480

```cpp
 461:     return LowerCallArguments(Chain, CallConv, IsVarArg, Ins, dl, DAG, InVals);
 462:   }
 463: }
 464: 
 465: /// Transform physical registers into virtual registers, and generate load
 466: /// operations for argument places on the stack.
 467: SDValue ARCTargetLowering::LowerCallArguments(
 468:     SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
 469:     const SmallVectorImpl<ISD::InputArg> &Ins, SDLoc dl, SelectionDAG &DAG,
 470:     SmallVectorImpl<SDValue> &InVals) const {
 471:   MachineFunction &MF = DAG.getMachineFunction();
 472:   MachineFrameInfo &MFI = MF.getFrameInfo();
 473:   MachineRegisterInfo &RegInfo = MF.getRegInfo();
 474:   auto *AFI = MF.getInfo<ARCFunctionInfo>();
 475: 
 476:   // Assign locations to all of the incoming arguments.
 477:   SmallVector<CCValAssign, 16> ArgLocs;
 478:   CCState CCInfo(CallConv, IsVarArg, DAG.getMachineFunction(), ArgLocs,
 479:                  *DAG.getContext());
 480: 
```

- EN: Function bodies or method definitions such as LowerCallArguments contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerCallArguments 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 481-500

```cpp
 481:   CCInfo.AnalyzeFormalArguments(Ins, CC_ARC);
 482: 
 483:   unsigned StackSlotSize = 4;
 484: 
 485:   if (!IsVarArg)
 486:     AFI->setReturnStackOffset(CCInfo.getStackSize());
 487: 
 488:   // All getCopyFromReg ops must precede any getMemcpys to prevent the
 489:   // scheduler clobbering a register before it has been copied.
 490:   // The stages are:
 491:   // 1. CopyFromReg (and load) arg & vararg registers.
 492:   // 2. Chain CopyFromReg nodes into a TokenFactor.
 493:   // 3. Memcpy 'byVal' args & push final InVals.
 494:   // 4. Chain mem ops nodes into a TokenFactor.
 495:   SmallVector<SDValue, 4> CFRegNode;
 496:   SmallVector<ArgDataPair, 4> ArgData;
 497:   SmallVector<SDValue, 4> MemOps;
 498: 
 499:   // 1a. CopyFromReg (and load) arg registers.
 500:   for (unsigned i = 0, e = ArgLocs.size(); i != e; ++i) {
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 501-520

```cpp
 501:     CCValAssign &VA = ArgLocs[i];
 502:     SDValue ArgIn;
 503: 
 504:     if (VA.isRegLoc()) {
 505:       // Arguments passed in registers
 506:       EVT RegVT = VA.getLocVT();
 507:       switch (RegVT.getSimpleVT().SimpleTy) {
 508:       default: {
 509:         LLVM_DEBUG(errs() << "LowerFormalArguments Unhandled argument type: "
 510:                           << (unsigned)RegVT.getSimpleVT().SimpleTy << "\n");
 511:         llvm_unreachable("Unhandled LowerFormalArguments type.");
 512:       }
 513:       case MVT::i32:
 514:         unsigned VReg = RegInfo.createVirtualRegister(&ARC::GPR32RegClass);
 515:         RegInfo.addLiveIn(VA.getLocReg(), VReg);
 516:         ArgIn = DAG.getCopyFromReg(Chain, dl, VReg, RegVT);
 517:         CFRegNode.push_back(ArgIn.getValue(ArgIn->getNumValues() - 1));
 518:       }
 519:     } else {
 520:       // Only arguments passed on the stack should make it here.
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 521-540

```cpp
 521:       assert(VA.isMemLoc());
 522:       // Load the argument to a virtual register
 523:       unsigned ObjSize = VA.getLocVT().getStoreSize();
 524:       assert((ObjSize <= StackSlotSize) && "Unhandled argument");
 525: 
 526:       // Create the frame index object for this incoming parameter...
 527:       int FI = MFI.CreateFixedObject(ObjSize, VA.getLocMemOffset(), true);
 528: 
 529:       // Create the SelectionDAG nodes corresponding to a load
 530:       // from this parameter
 531:       SDValue FIN = DAG.getFrameIndex(FI, MVT::i32);
 532:       ArgIn = DAG.getLoad(VA.getLocVT(), dl, Chain, FIN,
 533:                           MachinePointerInfo::getFixedStack(MF, FI));
 534:     }
 535:     const ArgDataPair ADP = {ArgIn, Ins[i].Flags};
 536:     ArgData.push_back(ADP);
 537:   }
 538: 
 539:   // 1b. CopyFromReg vararg registers.
 540:   if (IsVarArg) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 541-560

```cpp
 541:     // Argument registers
 542:     static const MCPhysReg ArgRegs[] = {ARC::R0, ARC::R1, ARC::R2, ARC::R3,
 543:                                         ARC::R4, ARC::R5, ARC::R6, ARC::R7};
 544:     auto *AFI = MF.getInfo<ARCFunctionInfo>();
 545:     unsigned FirstVAReg = CCInfo.getFirstUnallocated(ArgRegs);
 546:     if (FirstVAReg < std::size(ArgRegs)) {
 547:       int Offset = 0;
 548:       // Save remaining registers, storing higher register numbers at a higher
 549:       // address
 550:       // There are (std::size(ArgRegs) - FirstVAReg) registers which
 551:       // need to be saved.
 552:       int VarFI = MFI.CreateFixedObject((std::size(ArgRegs) - FirstVAReg) * 4,
 553:                                         CCInfo.getStackSize(), true);
 554:       AFI->setVarArgsFrameIndex(VarFI);
 555:       SDValue FIN = DAG.getFrameIndex(VarFI, MVT::i32);
 556:       for (unsigned i = FirstVAReg; i < std::size(ArgRegs); i++) {
 557:         // Move argument from phys reg -> virt reg
 558:         unsigned VReg = RegInfo.createVirtualRegister(&ARC::GPR32RegClass);
 559:         RegInfo.addLiveIn(ArgRegs[i], VReg);
 560:         SDValue Val = DAG.getCopyFromReg(Chain, dl, VReg, MVT::i32);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 561-580

```cpp
 561:         CFRegNode.push_back(Val.getValue(Val->getNumValues() - 1));
 562:         SDValue VAObj = DAG.getNode(ISD::ADD, dl, MVT::i32, FIN,
 563:                                     DAG.getConstant(Offset, dl, MVT::i32));
 564:         // Move argument from virt reg -> stack
 565:         SDValue Store =
 566:             DAG.getStore(Val.getValue(1), dl, Val, VAObj, MachinePointerInfo());
 567:         MemOps.push_back(Store);
 568:         Offset += 4;
 569:       }
 570:     } else {
 571:       llvm_unreachable("Too many var args parameters.");
 572:     }
 573:   }
 574: 
 575:   // 2. Chain CopyFromReg nodes into a TokenFactor.
 576:   if (!CFRegNode.empty())
 577:     Chain = DAG.getNode(ISD::TokenFactor, dl, MVT::Other, CFRegNode);
 578: 
 579:   // 3. Memcpy 'byVal' args & push final InVals.
 580:   // Aggregates passed "byVal" need to be copied by the callee.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 581-600

```cpp
 581:   // The callee will use a pointer to this copy, rather than the original
 582:   // pointer.
 583:   for (const auto &ArgDI : ArgData) {
 584:     if (ArgDI.Flags.isByVal() && ArgDI.Flags.getByValSize()) {
 585:       unsigned Size = ArgDI.Flags.getByValSize();
 586:       Align Alignment =
 587:           std::max(Align(StackSlotSize), ArgDI.Flags.getNonZeroByValAlign());
 588:       // Create a new object on the stack and copy the pointee into it.
 589:       int FI = MFI.CreateStackObject(Size, Alignment, false);
 590:       SDValue FIN = DAG.getFrameIndex(FI, MVT::i32);
 591:       InVals.push_back(FIN);
 592:       MemOps.push_back(DAG.getMemcpy(
 593:           Chain, dl, FIN, ArgDI.SDV, DAG.getConstant(Size, dl, MVT::i32),
 594:           Alignment, false, false, /*CI=*/nullptr, false, MachinePointerInfo(),
 595:           MachinePointerInfo()));
 596:     } else {
 597:       InVals.push_back(ArgDI.SDV);
 598:     }
 599:   }
 600: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 601-620

```cpp
 601:   // 4. Chain mem ops nodes into a TokenFactor.
 602:   if (!MemOps.empty()) {
 603:     MemOps.push_back(Chain);
 604:     Chain = DAG.getNode(ISD::TokenFactor, dl, MVT::Other, MemOps);
 605:   }
 606: 
 607:   return Chain;
 608: }
 609: 
 610: //===----------------------------------------------------------------------===//
 611: //               Return Value Calling Convention Implementation
 612: //===----------------------------------------------------------------------===//
 613: 
 614: bool ARCTargetLowering::CanLowerReturn(
 615:     CallingConv::ID CallConv, MachineFunction &MF, bool IsVarArg,
 616:     const SmallVectorImpl<ISD::OutputArg> &Outs, LLVMContext &Context,
 617:     const Type *RetTy) const {
 618:   SmallVector<CCValAssign, 16> RVLocs;
 619:   CCState CCInfo(CallConv, IsVarArg, MF, RVLocs, Context);
 620:   if (!CCInfo.CheckReturn(Outs, RetCC_ARC))
```

- EN: Function bodies or method definitions such as CanLowerReturn contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: CanLowerReturn 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 621-640

```cpp
 621:     return false;
 622:   if (CCInfo.getStackSize() != 0 && IsVarArg)
 623:     return false;
 624:   return true;
 625: }
 626: 
 627: SDValue
 628: ARCTargetLowering::LowerReturn(SDValue Chain, CallingConv::ID CallConv,
 629:                                bool IsVarArg,
 630:                                const SmallVectorImpl<ISD::OutputArg> &Outs,
 631:                                const SmallVectorImpl<SDValue> &OutVals,
 632:                                const SDLoc &dl, SelectionDAG &DAG) const {
 633:   auto *AFI = DAG.getMachineFunction().getInfo<ARCFunctionInfo>();
 634:   MachineFrameInfo &MFI = DAG.getMachineFunction().getFrameInfo();
 635: 
 636:   // CCValAssign - represent the assignment of
 637:   // the return value to a location
 638:   SmallVector<CCValAssign, 16> RVLocs;
 639: 
 640:   // CCState - Info about the registers and stack slot.
```

- EN: Function bodies or method definitions such as LowerReturn contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerReturn 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 641-660

```cpp
 641:   CCState CCInfo(CallConv, IsVarArg, DAG.getMachineFunction(), RVLocs,
 642:                  *DAG.getContext());
 643: 
 644:   // Analyze return values.
 645:   if (!IsVarArg)
 646:     CCInfo.AllocateStack(AFI->getReturnStackOffset(), Align(4));
 647: 
 648:   CCInfo.AnalyzeReturn(Outs, RetCC_ARC);
 649: 
 650:   SDValue Glue;
 651:   SmallVector<SDValue, 4> RetOps(1, Chain);
 652:   SmallVector<SDValue, 4> MemOpChains;
 653:   // Handle return values that must be copied to memory.
 654:   for (unsigned i = 0, e = RVLocs.size(); i != e; ++i) {
 655:     CCValAssign &VA = RVLocs[i];
 656:     if (VA.isRegLoc())
 657:       continue;
 658:     assert(VA.isMemLoc());
 659:     if (IsVarArg) {
 660:       report_fatal_error("Can't return value from vararg function in memory");
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 661-680

```cpp
 661:     }
 662: 
 663:     int Offset = VA.getLocMemOffset();
 664:     unsigned ObjSize = VA.getLocVT().getStoreSize();
 665:     // Create the frame index object for the memory location.
 666:     int FI = MFI.CreateFixedObject(ObjSize, Offset, false);
 667: 
 668:     // Create a SelectionDAG node corresponding to a store
 669:     // to this memory location.
 670:     SDValue FIN = DAG.getFrameIndex(FI, MVT::i32);
 671:     MemOpChains.push_back(DAG.getStore(
 672:         Chain, dl, OutVals[i], FIN,
 673:         MachinePointerInfo::getFixedStack(DAG.getMachineFunction(), FI)));
 674:   }
 675: 
 676:   // Transform all store nodes into one single node because
 677:   // all stores are independent of each other.
 678:   if (!MemOpChains.empty())
 679:     Chain = DAG.getNode(ISD::TokenFactor, dl, MVT::Other, MemOpChains);
 680: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 681-700

```cpp
 681:   // Now handle return values copied to registers.
 682:   for (unsigned i = 0, e = RVLocs.size(); i != e; ++i) {
 683:     CCValAssign &VA = RVLocs[i];
 684:     if (!VA.isRegLoc())
 685:       continue;
 686:     // Copy the result values into the output registers.
 687:     Chain = DAG.getCopyToReg(Chain, dl, VA.getLocReg(), OutVals[i], Glue);
 688: 
 689:     // guarantee that all emitted copies are
 690:     // stuck together, avoiding something bad
 691:     Glue = Chain.getValue(1);
 692:     RetOps.push_back(DAG.getRegister(VA.getLocReg(), VA.getLocVT()));
 693:   }
 694: 
 695:   RetOps[0] = Chain; // Update chain.
 696: 
 697:   // Add the glue if we have it.
 698:   if (Glue.getNode())
 699:     RetOps.push_back(Glue);
 700: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 701-720

```cpp
 701:   // What to do with the RetOps?
 702:   return DAG.getNode(ARCISD::RET, dl, MVT::Other, RetOps);
 703: }
 704: 
 705: //===----------------------------------------------------------------------===//
 706: // Target Optimization Hooks
 707: //===----------------------------------------------------------------------===//
 708: 
 709: SDValue ARCTargetLowering::PerformDAGCombine(SDNode *N,
 710:                                              DAGCombinerInfo &DCI) const {
 711:   return {};
 712: }
 713: 
 714: //===----------------------------------------------------------------------===//
 715: //  Addressing mode description hooks
 716: //===----------------------------------------------------------------------===//
 717: 
 718: /// Return true if the addressing mode represented by AM is legal for this
 719: /// target, for a load/store of the specified type.
 720: bool ARCTargetLowering::isLegalAddressingMode(const DataLayout &DL,
```

- EN: Function bodies or method definitions such as PerformDAGCombine contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: PerformDAGCombine 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 721-740

```cpp
 721:                                               const AddrMode &AM, Type *Ty,
 722:                                               unsigned AS,
 723:                                               Instruction *I) const {
 724:   return AM.Scale == 0;
 725: }
 726: 
 727: // Don't emit tail calls for the time being.
 728: bool ARCTargetLowering::mayBeEmittedAsTailCall(const CallInst *CI) const {
 729:   return false;
 730: }
 731: 
 732: SDValue ARCTargetLowering::LowerFRAMEADDR(SDValue Op, SelectionDAG &DAG) const {
 733:   const ARCRegisterInfo &ARI = *Subtarget.getRegisterInfo();
 734:   MachineFunction &MF = DAG.getMachineFunction();
 735:   MachineFrameInfo &MFI = MF.getFrameInfo();
 736:   MFI.setFrameAddressIsTaken(true);
 737: 
 738:   EVT VT = Op.getValueType();
 739:   SDLoc dl(Op);
 740:   assert(Op.getConstantOperandVal(0) == 0 &&
```

- EN: Function bodies or method definitions such as mayBeEmittedAsTailCall, LowerFRAMEADDR contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: mayBeEmittedAsTailCall, LowerFRAMEADDR 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 741-760

```cpp
 741:          "Only support lowering frame addr of current frame.");
 742:   Register FrameReg = ARI.getFrameRegister(MF);
 743:   return DAG.getCopyFromReg(DAG.getEntryNode(), dl, FrameReg, VT);
 744: }
 745: 
 746: SDValue ARCTargetLowering::LowerGlobalAddress(SDValue Op,
 747:                                               SelectionDAG &DAG) const {
 748:   const GlobalAddressSDNode *GN = cast<GlobalAddressSDNode>(Op);
 749:   const GlobalValue *GV = GN->getGlobal();
 750:   SDLoc dl(GN);
 751:   int64_t Offset = GN->getOffset();
 752:   SDValue GA = DAG.getTargetGlobalAddress(GV, dl, MVT::i32, Offset);
 753:   return DAG.getNode(ARCISD::GAWRAPPER, dl, MVT::i32, GA);
 754: }
 755: 
 756: static SDValue LowerVASTART(SDValue Op, SelectionDAG &DAG) {
 757:   MachineFunction &MF = DAG.getMachineFunction();
 758:   auto *FuncInfo = MF.getInfo<ARCFunctionInfo>();
 759: 
 760:   // vastart just stores the address of the VarArgsFrameIndex slot into the
```

- EN: Function bodies or method definitions such as LowerGlobalAddress, LowerVASTART contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerGlobalAddress, LowerVASTART 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 761-780

```cpp
 761:   // memory location argument.
 762:   SDLoc dl(Op);
 763:   EVT PtrVT = DAG.getTargetLoweringInfo().getPointerTy(DAG.getDataLayout());
 764:   SDValue FR = DAG.getFrameIndex(FuncInfo->getVarArgsFrameIndex(), PtrVT);
 765:   const Value *SV = cast<SrcValueSDNode>(Op.getOperand(2))->getValue();
 766:   return DAG.getStore(Op.getOperand(0), dl, FR, Op.getOperand(1),
 767:                       MachinePointerInfo(SV));
 768: }
 769: 
 770: SDValue ARCTargetLowering::LowerOperation(SDValue Op, SelectionDAG &DAG) const {
 771:   switch (Op.getOpcode()) {
 772:   case ISD::GlobalAddress:
 773:     return LowerGlobalAddress(Op, DAG);
 774:   case ISD::FRAMEADDR:
 775:     return LowerFRAMEADDR(Op, DAG);
 776:   case ISD::SELECT_CC:
 777:     return LowerSELECT_CC(Op, DAG);
 778:   case ISD::BR_CC:
 779:     return LowerBR_CC(Op, DAG);
 780:   case ISD::SIGN_EXTEND_INREG:
```

- EN: Function bodies or method definitions such as LowerOperation contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerOperation 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 781-796

```cpp
 781:     return LowerSIGN_EXTEND_INREG(Op, DAG);
 782:   case ISD::JumpTable:
 783:     return LowerJumpTable(Op, DAG);
 784:   case ISD::VASTART:
 785:     return LowerVASTART(Op, DAG);
 786:   case ISD::READCYCLECOUNTER:
 787:     // As of LLVM 3.8, the lowering code insists that we customize it even
 788:     // though we've declared the i32 version as legal. This is because it only
 789:     // thinks i64 is the truly supported version. We've already converted the
 790:     // i64 version to a widened i32.
 791:     assert(Op.getSimpleValueType() == MVT::i32);
 792:     return Op;
 793:   default:
 794:     llvm_unreachable("unimplemented operand");
 795:   }
 796: }
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

## Key Concepts / 关键概念

- Custom DAG legalization / 自定义 DAG 合法化
- Calling-lowering hooks / 调用降级钩子
- SelectionDAG lowering / SelectionDAG 降级
- Machine instruction manipulation / 机器指令操作
- MachineFunction state / MachineFunction 状态
- Register classes / 寄存器类
- IR to target lowering / IR 到目标降级
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

- Direct includes / 直接包含: `ARCISelLowering.h`, `ARC.h`, `ARCMachineFunctionInfo.h`, `ARCSelectionDAGInfo.h`, `ARCSubtarget.h`, `ARCTargetMachine.h`, `MCTargetDesc/ARCInfo.h`, `llvm/CodeGen/CallingConvLower.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, SelectionDAG
- Generated or companion files / 生成或配套文件: `ARCGenCallingConv.inc`
- Local companions / 本地配套文件: `ARCISelLowering.h`
