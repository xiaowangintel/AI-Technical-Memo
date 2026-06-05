# AVRISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/AVR/AVRISelLowering.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file defines the interfaces that AVR uses to lower LLVM code into a selection DAG.
- 目的（中文）: 实现 SelectionDAG 降级钩子，将通用 LLVM IR 操作映射到目标专用 DAG 节点与调用约定。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- AVRISelLowering.cpp - AVR DAG Lowering Implementation -------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the interfaces that AVR uses to lower LLVM code into a
  10: // selection DAG.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "AVRISelLowering.h"
  15: 
  16: #include "llvm/ADT/ArrayRef.h"
  17: #include "llvm/ADT/StringSwitch.h"
  18: #include "llvm/CodeGen/CallingConvLower.h"
  19: #include "llvm/CodeGen/MachineFrameInfo.h"
  20: #include "llvm/CodeGen/MachineInstrBuilder.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/MachineRegisterInfo.h"
  22: #include "llvm/CodeGen/SelectionDAG.h"
  23: #include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
  24: #include "llvm/IR/Function.h"
  25: #include "llvm/Support/ErrorHandling.h"
  26: 
  27: #include "AVR.h"
  28: #include "AVRMachineFunctionInfo.h"
  29: #include "AVRSubtarget.h"
  30: #include "AVRTargetMachine.h"
  31: #include "MCTargetDesc/AVRMCTargetDesc.h"
  32: 
  33: namespace llvm {
  34: 
  35: AVRTargetLowering::AVRTargetLowering(const AVRTargetMachine &TM,
  36:                                      const AVRSubtarget &STI)
  37:     : TargetLowering(TM, STI), Subtarget(STI) {
  38:   // Set up the register classes.
  39:   addRegisterClass(MVT::i8, &AVR::GPR8RegClass);
  40:   addRegisterClass(MVT::i16, &AVR::DREGSRegClass);
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as TargetLowering contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 TargetLowering 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 41-60

```cpp
  41: 
  42:   // Compute derived properties from the register classes.
  43:   computeRegisterProperties(Subtarget.getRegisterInfo());
  44: 
  45:   setBooleanContents(ZeroOrOneBooleanContent);
  46:   setBooleanVectorContents(ZeroOrOneBooleanContent);
  47:   setSchedulingPreference(Sched::RegPressure);
  48:   setStackPointerRegisterToSaveRestore(AVR::SP);
  49:   setSupportsUnalignedAtomics(true);
  50: 
  51:   setOperationAction(ISD::GlobalAddress, MVT::i16, Custom);
  52:   setOperationAction(ISD::BlockAddress, MVT::i16, Custom);
  53: 
  54:   setOperationAction(ISD::STACKSAVE, MVT::Other, Expand);
  55:   setOperationAction(ISD::STACKRESTORE, MVT::Other, Expand);
  56:   setOperationAction(ISD::DYNAMIC_STACKALLOC, MVT::i8, Expand);
  57:   setOperationAction(ISD::DYNAMIC_STACKALLOC, MVT::i16, Expand);
  58: 
  59:   setOperationAction(ISD::INLINEASM, MVT::Other, Custom);
  60: 
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 61-80

```cpp
  61:   for (MVT VT : MVT::integer_valuetypes()) {
  62:     for (auto N : {ISD::EXTLOAD, ISD::SEXTLOAD, ISD::ZEXTLOAD}) {
  63:       setLoadExtAction(N, VT, MVT::i1, Promote);
  64:       setLoadExtAction(N, VT, MVT::i8, Expand);
  65:     }
  66:   }
  67: 
  68:   setTruncStoreAction(MVT::i16, MVT::i8, Expand);
  69: 
  70:   for (MVT VT : MVT::integer_valuetypes()) {
  71:     setOperationAction(ISD::ADDC, VT, Legal);
  72:     setOperationAction(ISD::SUBC, VT, Legal);
  73:     setOperationAction(ISD::ADDE, VT, Legal);
  74:     setOperationAction(ISD::SUBE, VT, Legal);
  75:   }
  76: 
  77:   // sub (x, imm) gets canonicalized to add (x, -imm), so for illegal types
  78:   // revert into a sub since we don't have an add with immediate instruction.
  79:   setOperationAction(ISD::ADD, MVT::i32, Custom);
  80:   setOperationAction(ISD::ADD, MVT::i64, Custom);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 81-100

```cpp
  81: 
  82:   // our shift instructions are only able to shift 1 bit at a time, so handle
  83:   // this in a custom way.
  84:   setOperationAction(ISD::SRA, MVT::i8, Custom);
  85:   setOperationAction(ISD::SHL, MVT::i8, Custom);
  86:   setOperationAction(ISD::SRL, MVT::i8, Custom);
  87:   setOperationAction(ISD::SRA, MVT::i16, Custom);
  88:   setOperationAction(ISD::SHL, MVT::i16, Custom);
  89:   setOperationAction(ISD::SRL, MVT::i16, Custom);
  90:   setOperationAction(ISD::SRA, MVT::i32, Custom);
  91:   setOperationAction(ISD::SHL, MVT::i32, Custom);
  92:   setOperationAction(ISD::SRL, MVT::i32, Custom);
  93:   setOperationAction(ISD::SHL_PARTS, MVT::i16, Expand);
  94:   setOperationAction(ISD::SRA_PARTS, MVT::i16, Expand);
  95:   setOperationAction(ISD::SRL_PARTS, MVT::i16, Expand);
  96: 
  97:   setOperationAction(ISD::ROTL, MVT::i8, Custom);
  98:   setOperationAction(ISD::ROTL, MVT::i16, Expand);
  99:   setOperationAction(ISD::ROTR, MVT::i8, Custom);
 100:   setOperationAction(ISD::ROTR, MVT::i16, Expand);
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 101-120

```cpp
 101: 
 102:   setOperationAction(ISD::BR_CC, MVT::i8, Custom);
 103:   setOperationAction(ISD::BR_CC, MVT::i16, Custom);
 104:   setOperationAction(ISD::BR_CC, MVT::i32, Custom);
 105:   setOperationAction(ISD::BR_CC, MVT::i64, Custom);
 106:   setOperationAction(ISD::BRCOND, MVT::Other, Expand);
 107: 
 108:   setOperationAction(ISD::SELECT_CC, MVT::i8, Custom);
 109:   setOperationAction(ISD::SELECT_CC, MVT::i16, Custom);
 110:   setOperationAction(ISD::SELECT_CC, MVT::i32, Expand);
 111:   setOperationAction(ISD::SELECT_CC, MVT::i64, Expand);
 112:   setOperationAction(ISD::SETCC, MVT::i8, Custom);
 113:   setOperationAction(ISD::SETCC, MVT::i16, Custom);
 114:   setOperationAction(ISD::SETCC, MVT::i32, Custom);
 115:   setOperationAction(ISD::SETCC, MVT::i64, Custom);
 116:   setOperationAction(ISD::SELECT, MVT::i8, Expand);
 117:   setOperationAction(ISD::SELECT, MVT::i16, Expand);
 118: 
 119:   setOperationAction(ISD::BSWAP, MVT::i16, Expand);
 120: 
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 121-140

```cpp
 121:   // Add support for postincrement and predecrement load/stores.
 122:   setIndexedLoadAction(ISD::POST_INC, MVT::i8, Legal);
 123:   setIndexedLoadAction(ISD::POST_INC, MVT::i16, Legal);
 124:   setIndexedLoadAction(ISD::PRE_DEC, MVT::i8, Legal);
 125:   setIndexedLoadAction(ISD::PRE_DEC, MVT::i16, Legal);
 126:   setIndexedStoreAction(ISD::POST_INC, MVT::i8, Legal);
 127:   setIndexedStoreAction(ISD::POST_INC, MVT::i16, Legal);
 128:   setIndexedStoreAction(ISD::PRE_DEC, MVT::i8, Legal);
 129:   setIndexedStoreAction(ISD::PRE_DEC, MVT::i16, Legal);
 130: 
 131:   setOperationAction(ISD::BR_JT, MVT::Other, Expand);
 132: 
 133:   setOperationAction(ISD::VASTART, MVT::Other, Custom);
 134:   setOperationAction(ISD::VAEND, MVT::Other, Expand);
 135:   setOperationAction(ISD::VAARG, MVT::Other, Expand);
 136:   setOperationAction(ISD::VACOPY, MVT::Other, Expand);
 137: 
 138:   // Atomic operations which must be lowered to rtlib calls
 139:   for (MVT VT : MVT::integer_valuetypes()) {
 140:     setOperationAction(ISD::ATOMIC_SWAP, VT, Expand);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 141-160

```cpp
 141:     setOperationAction(ISD::ATOMIC_CMP_SWAP, VT, Expand);
 142:     setOperationAction(ISD::ATOMIC_LOAD_NAND, VT, Expand);
 143:     setOperationAction(ISD::ATOMIC_LOAD_MAX, VT, Expand);
 144:     setOperationAction(ISD::ATOMIC_LOAD_MIN, VT, Expand);
 145:     setOperationAction(ISD::ATOMIC_LOAD_UMAX, VT, Expand);
 146:     setOperationAction(ISD::ATOMIC_LOAD_UMIN, VT, Expand);
 147:   }
 148: 
 149:   // Division/remainder
 150:   setOperationAction(ISD::UDIV, MVT::i8, Expand);
 151:   setOperationAction(ISD::UDIV, MVT::i16, Expand);
 152:   setOperationAction(ISD::UREM, MVT::i8, Expand);
 153:   setOperationAction(ISD::UREM, MVT::i16, Expand);
 154:   setOperationAction(ISD::SDIV, MVT::i8, Expand);
 155:   setOperationAction(ISD::SDIV, MVT::i16, Expand);
 156:   setOperationAction(ISD::SREM, MVT::i8, Expand);
 157:   setOperationAction(ISD::SREM, MVT::i16, Expand);
 158: 
 159:   // Make division and modulus custom
 160:   setOperationAction(ISD::UDIVREM, MVT::i8, Custom);
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 161-180

```cpp
 161:   setOperationAction(ISD::UDIVREM, MVT::i16, Custom);
 162:   setOperationAction(ISD::UDIVREM, MVT::i32, Custom);
 163:   setOperationAction(ISD::SDIVREM, MVT::i8, Custom);
 164:   setOperationAction(ISD::SDIVREM, MVT::i16, Custom);
 165:   setOperationAction(ISD::SDIVREM, MVT::i32, Custom);
 166: 
 167:   // Do not use MUL. The AVR instructions are closer to SMUL_LOHI &co.
 168:   setOperationAction(ISD::MUL, MVT::i8, Expand);
 169:   setOperationAction(ISD::MUL, MVT::i16, Expand);
 170: 
 171:   // Expand 16 bit multiplications.
 172:   setOperationAction(ISD::SMUL_LOHI, MVT::i16, Expand);
 173:   setOperationAction(ISD::UMUL_LOHI, MVT::i16, Expand);
 174: 
 175:   // Expand multiplications to libcalls when there is
 176:   // no hardware MUL.
 177:   if (!Subtarget.supportsMultiplication()) {
 178:     setOperationAction(ISD::SMUL_LOHI, MVT::i8, Expand);
 179:     setOperationAction(ISD::UMUL_LOHI, MVT::i8, Expand);
 180:   }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 181-200

```cpp
 181: 
 182:   for (MVT VT : MVT::integer_valuetypes()) {
 183:     setOperationAction(ISD::MULHS, VT, Expand);
 184:     setOperationAction(ISD::MULHU, VT, Expand);
 185:   }
 186: 
 187:   for (MVT VT : MVT::integer_valuetypes()) {
 188:     setOperationAction(ISD::CTPOP, VT, Expand);
 189:     setOperationAction(ISD::CTLZ, VT, Expand);
 190:     setOperationAction(ISD::CTTZ, VT, Expand);
 191:   }
 192: 
 193:   for (MVT VT : MVT::integer_valuetypes()) {
 194:     setOperationAction(ISD::SIGN_EXTEND_INREG, VT, Expand);
 195:     // TODO: The generated code is pretty poor. Investigate using the
 196:     // same "shift and subtract with carry" trick that we do for
 197:     // extending 8-bit to 16-bit. This may require infrastructure
 198:     // improvements in how we treat 16-bit "registers" to be feasible.
 199:   }
 200: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 201-220

```cpp
 201:   setMinFunctionAlignment(Align(2));
 202:   setMinimumJumpTableEntries(UINT_MAX);
 203: }
 204: 
 205: EVT AVRTargetLowering::getSetCCResultType(const DataLayout &DL, LLVMContext &,
 206:                                           EVT VT) const {
 207:   assert(!VT.isVector() && "No AVR SetCC type for vectors!");
 208:   return MVT::i8;
 209: }
 210: 
 211: SDValue AVRTargetLowering::LowerShifts(SDValue Op, SelectionDAG &DAG) const {
 212:   unsigned Opc8;
 213:   const SDNode *N = Op.getNode();
 214:   EVT VT = Op.getValueType();
 215:   SDLoc dl(N);
 216:   assert(llvm::has_single_bit<uint32_t>(VT.getSizeInBits()) &&
 217:          "Expected power-of-2 shift amount");
 218: 
 219:   if (VT.getSizeInBits() == 32) {
 220:     if (!isa<ConstantSDNode>(N->getOperand(1))) {
```

- EN: Function bodies or method definitions such as getSetCCResultType, LowerShifts contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getSetCCResultType, LowerShifts 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 221-240

```cpp
 221:       // 32-bit shifts are converted to a loop in IR.
 222:       // This should be unreachable.
 223:       report_fatal_error("Expected a constant shift amount!");
 224:     }
 225:     SDVTList ResTys = DAG.getVTList(MVT::i16, MVT::i16);
 226:     SDValue SrcLo =
 227:         DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i16, Op.getOperand(0),
 228:                     DAG.getConstant(0, dl, MVT::i16));
 229:     SDValue SrcHi =
 230:         DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i16, Op.getOperand(0),
 231:                     DAG.getConstant(1, dl, MVT::i16));
 232:     uint64_t ShiftAmount = N->getConstantOperandVal(1);
 233:     if (ShiftAmount == 16) {
 234:       // Special case these two operations because they appear to be used by the
 235:       // generic codegen parts to lower 32-bit numbers.
 236:       // TODO: perhaps we can lower shift amounts bigger than 16 to a 16-bit
 237:       // shift of a part of the 32-bit value?
 238:       switch (Op.getOpcode()) {
 239:       case ISD::SHL: {
 240:         SDValue Zero = DAG.getConstant(0, dl, MVT::i16);
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 241-260

```cpp
 241:         return DAG.getNode(ISD::BUILD_PAIR, dl, MVT::i32, Zero, SrcLo);
 242:       }
 243:       case ISD::SRL: {
 244:         SDValue Zero = DAG.getConstant(0, dl, MVT::i16);
 245:         return DAG.getNode(ISD::BUILD_PAIR, dl, MVT::i32, SrcHi, Zero);
 246:       }
 247:       }
 248:     }
 249:     SDValue Cnt = DAG.getTargetConstant(ShiftAmount, dl, MVT::i8);
 250:     unsigned Opc;
 251:     switch (Op.getOpcode()) {
 252:     default:
 253:       llvm_unreachable("Invalid 32-bit shift opcode!");
 254:     case ISD::SHL:
 255:       Opc = AVRISD::LSLW;
 256:       break;
 257:     case ISD::SRL:
 258:       Opc = AVRISD::LSRW;
 259:       break;
 260:     case ISD::SRA:
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 261-280

```cpp
 261:       Opc = AVRISD::ASRW;
 262:       break;
 263:     }
 264:     SDValue Result = DAG.getNode(Opc, dl, ResTys, SrcLo, SrcHi, Cnt);
 265:     return DAG.getNode(ISD::BUILD_PAIR, dl, MVT::i32, Result.getValue(0),
 266:                        Result.getValue(1));
 267:   }
 268: 
 269:   // Expand non-constant shifts to loops.
 270:   if (!isa<ConstantSDNode>(N->getOperand(1))) {
 271:     switch (Op.getOpcode()) {
 272:     default:
 273:       llvm_unreachable("Invalid shift opcode!");
 274:     case ISD::SHL:
 275:       return DAG.getNode(AVRISD::LSLLOOP, dl, VT, N->getOperand(0),
 276:                          N->getOperand(1));
 277:     case ISD::SRL:
 278:       return DAG.getNode(AVRISD::LSRLOOP, dl, VT, N->getOperand(0),
 279:                          N->getOperand(1));
 280:     case ISD::ROTL: {
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 281-300

```cpp
 281:       SDValue Amt = N->getOperand(1);
 282:       EVT AmtVT = Amt.getValueType();
 283:       Amt = DAG.getNode(ISD::AND, dl, AmtVT, Amt,
 284:                         DAG.getConstant(VT.getSizeInBits() - 1, dl, AmtVT));
 285:       return DAG.getNode(AVRISD::ROLLOOP, dl, VT, N->getOperand(0), Amt);
 286:     }
 287:     case ISD::ROTR: {
 288:       SDValue Amt = N->getOperand(1);
 289:       EVT AmtVT = Amt.getValueType();
 290:       Amt = DAG.getNode(ISD::AND, dl, AmtVT, Amt,
 291:                         DAG.getConstant(VT.getSizeInBits() - 1, dl, AmtVT));
 292:       return DAG.getNode(AVRISD::RORLOOP, dl, VT, N->getOperand(0), Amt);
 293:     }
 294:     case ISD::SRA:
 295:       return DAG.getNode(AVRISD::ASRLOOP, dl, VT, N->getOperand(0),
 296:                          N->getOperand(1));
 297:     }
 298:   }
 299: 
 300:   uint64_t ShiftAmount = N->getConstantOperandVal(1);
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 301-320

```cpp
 301:   SDValue Victim = N->getOperand(0);
 302: 
 303:   switch (Op.getOpcode()) {
 304:   case ISD::SRA:
 305:     Opc8 = AVRISD::ASR;
 306:     break;
 307:   case ISD::ROTL:
 308:     Opc8 = AVRISD::ROL;
 309:     ShiftAmount = ShiftAmount % VT.getSizeInBits();
 310:     break;
 311:   case ISD::ROTR:
 312:     Opc8 = AVRISD::ROR;
 313:     ShiftAmount = ShiftAmount % VT.getSizeInBits();
 314:     break;
 315:   case ISD::SRL:
 316:     Opc8 = AVRISD::LSR;
 317:     break;
 318:   case ISD::SHL:
 319:     Opc8 = AVRISD::LSL;
 320:     break;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 321-340

```cpp
 321:   default:
 322:     llvm_unreachable("Invalid shift opcode");
 323:   }
 324: 
 325:   // Optimize int8/int16 shifts.
 326:   if (VT.getSizeInBits() == 8) {
 327:     if (Op.getOpcode() == ISD::SHL && 4 <= ShiftAmount && ShiftAmount < 7) {
 328:       // Optimize LSL when 4 <= ShiftAmount <= 6.
 329:       Victim = DAG.getNode(AVRISD::SWAP, dl, VT, Victim);
 330:       Victim =
 331:           DAG.getNode(ISD::AND, dl, VT, Victim, DAG.getConstant(0xf0, dl, VT));
 332:       ShiftAmount -= 4;
 333:     } else if (Op.getOpcode() == ISD::SRL && 4 <= ShiftAmount &&
 334:                ShiftAmount < 7) {
 335:       // Optimize LSR when 4 <= ShiftAmount <= 6.
 336:       Victim = DAG.getNode(AVRISD::SWAP, dl, VT, Victim);
 337:       Victim =
 338:           DAG.getNode(ISD::AND, dl, VT, Victim, DAG.getConstant(0x0f, dl, VT));
 339:       ShiftAmount -= 4;
 340:     } else if (Op.getOpcode() == ISD::SHL && ShiftAmount == 7) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 341-360

```cpp
 341:       // Optimize LSL when ShiftAmount == 7.
 342:       Victim = DAG.getNode(AVRISD::LSLBN, dl, VT, Victim,
 343:                            DAG.getConstant(7, dl, VT));
 344:       ShiftAmount = 0;
 345:     } else if (Op.getOpcode() == ISD::SRL && ShiftAmount == 7) {
 346:       // Optimize LSR when ShiftAmount == 7.
 347:       Victim = DAG.getNode(AVRISD::LSRBN, dl, VT, Victim,
 348:                            DAG.getConstant(7, dl, VT));
 349:       ShiftAmount = 0;
 350:     } else if (Op.getOpcode() == ISD::SRA && ShiftAmount == 6) {
 351:       // Optimize ASR when ShiftAmount == 6.
 352:       Victim = DAG.getNode(AVRISD::ASRBN, dl, VT, Victim,
 353:                            DAG.getConstant(6, dl, VT));
 354:       ShiftAmount = 0;
 355:     } else if (Op.getOpcode() == ISD::SRA && ShiftAmount == 7) {
 356:       // Optimize ASR when ShiftAmount == 7.
 357:       Victim = DAG.getNode(AVRISD::ASRBN, dl, VT, Victim,
 358:                            DAG.getConstant(7, dl, VT));
 359:       ShiftAmount = 0;
 360:     } else if (Op.getOpcode() == ISD::ROTL && ShiftAmount == 3) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 361-380

```cpp
 361:       // Optimize left rotation 3 bits to swap then right rotation 1 bit.
 362:       Victim = DAG.getNode(AVRISD::SWAP, dl, VT, Victim);
 363:       Victim = DAG.getNode(AVRISD::ROR, dl, VT, Victim);
 364:       ShiftAmount = 0;
 365:     } else if (Op.getOpcode() == ISD::ROTR && ShiftAmount == 3) {
 366:       // Optimize right rotation 3 bits to swap then left rotation 1 bit.
 367:       Victim = DAG.getNode(AVRISD::SWAP, dl, VT, Victim);
 368:       Victim = DAG.getNode(AVRISD::ROL, dl, VT, Victim);
 369:       ShiftAmount = 0;
 370:     } else if (Op.getOpcode() == ISD::ROTL && ShiftAmount == 7) {
 371:       // Optimize left rotation 7 bits to right rotation 1 bit.
 372:       Victim = DAG.getNode(AVRISD::ROR, dl, VT, Victim);
 373:       ShiftAmount = 0;
 374:     } else if (Op.getOpcode() == ISD::ROTR && ShiftAmount == 7) {
 375:       // Optimize right rotation 7 bits to left rotation 1 bit.
 376:       Victim = DAG.getNode(AVRISD::ROL, dl, VT, Victim);
 377:       ShiftAmount = 0;
 378:     } else if ((Op.getOpcode() == ISD::ROTR || Op.getOpcode() == ISD::ROTL) &&
 379:                ShiftAmount >= 4) {
 380:       // Optimize left/right rotation with the SWAP instruction.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 381-400

```cpp
 381:       Victim = DAG.getNode(AVRISD::SWAP, dl, VT, Victim);
 382:       ShiftAmount -= 4;
 383:     }
 384:   } else if (VT.getSizeInBits() == 16) {
 385:     if (Op.getOpcode() == ISD::SRA)
 386:       // Special optimization for int16 arithmetic right shift.
 387:       switch (ShiftAmount) {
 388:       case 15:
 389:         Victim = DAG.getNode(AVRISD::ASRWN, dl, VT, Victim,
 390:                              DAG.getConstant(15, dl, VT));
 391:         ShiftAmount = 0;
 392:         break;
 393:       case 14:
 394:         Victim = DAG.getNode(AVRISD::ASRWN, dl, VT, Victim,
 395:                              DAG.getConstant(14, dl, VT));
 396:         ShiftAmount = 0;
 397:         break;
 398:       case 7:
 399:         Victim = DAG.getNode(AVRISD::ASRWN, dl, VT, Victim,
 400:                              DAG.getConstant(7, dl, VT));
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 401-420

```cpp
 401:         ShiftAmount = 0;
 402:         break;
 403:       default:
 404:         break;
 405:       }
 406:     if (4 <= ShiftAmount && ShiftAmount < 8)
 407:       switch (Op.getOpcode()) {
 408:       case ISD::SHL:
 409:         Victim = DAG.getNode(AVRISD::LSLWN, dl, VT, Victim,
 410:                              DAG.getConstant(4, dl, VT));
 411:         ShiftAmount -= 4;
 412:         break;
 413:       case ISD::SRL:
 414:         Victim = DAG.getNode(AVRISD::LSRWN, dl, VT, Victim,
 415:                              DAG.getConstant(4, dl, VT));
 416:         ShiftAmount -= 4;
 417:         break;
 418:       default:
 419:         break;
 420:       }
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 421-440

```cpp
 421:     else if (8 <= ShiftAmount && ShiftAmount < 12)
 422:       switch (Op.getOpcode()) {
 423:       case ISD::SHL:
 424:         Victim = DAG.getNode(AVRISD::LSLWN, dl, VT, Victim,
 425:                              DAG.getConstant(8, dl, VT));
 426:         ShiftAmount -= 8;
 427:         // Only operate on the higher byte for remaining shift bits.
 428:         Opc8 = AVRISD::LSLHI;
 429:         break;
 430:       case ISD::SRL:
 431:         Victim = DAG.getNode(AVRISD::LSRWN, dl, VT, Victim,
 432:                              DAG.getConstant(8, dl, VT));
 433:         ShiftAmount -= 8;
 434:         // Only operate on the lower byte for remaining shift bits.
 435:         Opc8 = AVRISD::LSRLO;
 436:         break;
 437:       case ISD::SRA:
 438:         Victim = DAG.getNode(AVRISD::ASRWN, dl, VT, Victim,
 439:                              DAG.getConstant(8, dl, VT));
 440:         ShiftAmount -= 8;
```

- EN: Function bodies or method definitions such as if contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: if 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 441-460

```cpp
 441:         // Only operate on the lower byte for remaining shift bits.
 442:         Opc8 = AVRISD::ASRLO;
 443:         break;
 444:       default:
 445:         break;
 446:       }
 447:     else if (12 <= ShiftAmount)
 448:       switch (Op.getOpcode()) {
 449:       case ISD::SHL:
 450:         Victim = DAG.getNode(AVRISD::LSLWN, dl, VT, Victim,
 451:                              DAG.getConstant(12, dl, VT));
 452:         ShiftAmount -= 12;
 453:         // Only operate on the higher byte for remaining shift bits.
 454:         Opc8 = AVRISD::LSLHI;
 455:         break;
 456:       case ISD::SRL:
 457:         Victim = DAG.getNode(AVRISD::LSRWN, dl, VT, Victim,
 458:                              DAG.getConstant(12, dl, VT));
 459:         ShiftAmount -= 12;
 460:         // Only operate on the lower byte for remaining shift bits.
```

- EN: Function bodies or method definitions such as if contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: if 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 461-480

```cpp
 461:         Opc8 = AVRISD::LSRLO;
 462:         break;
 463:       case ISD::SRA:
 464:         Victim = DAG.getNode(AVRISD::ASRWN, dl, VT, Victim,
 465:                              DAG.getConstant(8, dl, VT));
 466:         ShiftAmount -= 8;
 467:         // Only operate on the lower byte for remaining shift bits.
 468:         Opc8 = AVRISD::ASRLO;
 469:         break;
 470:       default:
 471:         break;
 472:       }
 473:   }
 474: 
 475:   while (ShiftAmount--) {
 476:     Victim = DAG.getNode(Opc8, dl, VT, Victim);
 477:   }
 478: 
 479:   return Victim;
 480: }
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 481-500

```cpp
 481: 
 482: SDValue AVRTargetLowering::LowerDivRem(SDValue Op, SelectionDAG &DAG) const {
 483:   unsigned Opcode = Op->getOpcode();
 484:   assert((Opcode == ISD::SDIVREM || Opcode == ISD::UDIVREM) &&
 485:          "Invalid opcode for Div/Rem lowering");
 486:   bool IsSigned = (Opcode == ISD::SDIVREM);
 487:   EVT VT = Op->getValueType(0);
 488:   Type *Ty = VT.getTypeForEVT(*DAG.getContext());
 489: 
 490:   RTLIB::Libcall LC;
 491:   switch (VT.getSimpleVT().SimpleTy) {
 492:   default:
 493:     llvm_unreachable("Unexpected request for libcall!");
 494:   case MVT::i8:
 495:     LC = IsSigned ? RTLIB::SDIVREM_I8 : RTLIB::UDIVREM_I8;
 496:     break;
 497:   case MVT::i16:
 498:     LC = IsSigned ? RTLIB::SDIVREM_I16 : RTLIB::UDIVREM_I16;
 499:     break;
 500:   case MVT::i32:
```

- EN: Function bodies or method definitions such as LowerDivRem contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerDivRem 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 501-520

```cpp
 501:     LC = IsSigned ? RTLIB::SDIVREM_I32 : RTLIB::UDIVREM_I32;
 502:     break;
 503:   }
 504: 
 505:   SDValue InChain = DAG.getEntryNode();
 506: 
 507:   TargetLowering::ArgListTy Args;
 508:   for (SDValue const &Value : Op->op_values()) {
 509:     TargetLowering::ArgListEntry Entry(
 510:         Value, Value.getValueType().getTypeForEVT(*DAG.getContext()));
 511:     Entry.IsSExt = IsSigned;
 512:     Entry.IsZExt = !IsSigned;
 513:     Args.push_back(Entry);
 514:   }
 515: 
 516:   RTLIB::LibcallImpl LCImpl = DAG.getLibcalls().getLibcallImpl(LC);
 517:   if (LCImpl == RTLIB::Unsupported)
 518:     return SDValue();
 519: 
 520:   SDValue Callee =
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 521-540

```cpp
 521:       DAG.getExternalSymbol(LCImpl, getPointerTy(DAG.getDataLayout()));
 522: 
 523:   Type *RetTy = (Type *)StructType::get(Ty, Ty);
 524: 
 525:   SDLoc dl(Op);
 526:   TargetLowering::CallLoweringInfo CLI(DAG);
 527:   CLI.setDebugLoc(dl)
 528:       .setChain(InChain)
 529:       .setLibCallee(DAG.getLibcalls().getLibcallImplCallingConv(LCImpl), RetTy,
 530:                     Callee, std::move(Args))
 531:       .setInRegister()
 532:       .setSExtResult(IsSigned)
 533:       .setZExtResult(!IsSigned);
 534: 
 535:   std::pair<SDValue, SDValue> CallInfo = LowerCallTo(CLI);
 536:   return CallInfo.first;
 537: }
 538: 
 539: SDValue AVRTargetLowering::LowerGlobalAddress(SDValue Op,
 540:                                               SelectionDAG &DAG) const {
```

- EN: Function bodies or method definitions such as LowerGlobalAddress contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerGlobalAddress 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 541-560

```cpp
 541:   auto DL = DAG.getDataLayout();
 542: 
 543:   const GlobalValue *GV = cast<GlobalAddressSDNode>(Op)->getGlobal();
 544:   int64_t Offset = cast<GlobalAddressSDNode>(Op)->getOffset();
 545: 
 546:   // Create the TargetGlobalAddress node, folding in the constant offset.
 547:   SDValue Result =
 548:       DAG.getTargetGlobalAddress(GV, SDLoc(Op), getPointerTy(DL), Offset);
 549:   return DAG.getNode(AVRISD::WRAPPER, SDLoc(Op), getPointerTy(DL), Result);
 550: }
 551: 
 552: SDValue AVRTargetLowering::LowerBlockAddress(SDValue Op,
 553:                                              SelectionDAG &DAG) const {
 554:   auto DL = DAG.getDataLayout();
 555:   const BlockAddress *BA = cast<BlockAddressSDNode>(Op)->getBlockAddress();
 556: 
 557:   SDValue Result = DAG.getTargetBlockAddress(BA, getPointerTy(DL));
 558: 
 559:   return DAG.getNode(AVRISD::WRAPPER, SDLoc(Op), getPointerTy(DL), Result);
 560: }
```

- EN: Function bodies or method definitions such as LowerBlockAddress contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerBlockAddress 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 561-580

```cpp
 561: 
 562: /// IntCCToAVRCC - Convert a DAG integer condition code to an AVR CC.
 563: static AVRCC::CondCodes intCCToAVRCC(ISD::CondCode CC) {
 564:   switch (CC) {
 565:   default:
 566:     llvm_unreachable("Unknown condition code!");
 567:   case ISD::SETEQ:
 568:     return AVRCC::COND_EQ;
 569:   case ISD::SETNE:
 570:     return AVRCC::COND_NE;
 571:   case ISD::SETGE:
 572:     return AVRCC::COND_GE;
 573:   case ISD::SETLT:
 574:     return AVRCC::COND_LT;
 575:   case ISD::SETUGE:
 576:     return AVRCC::COND_SH;
 577:   case ISD::SETULT:
 578:     return AVRCC::COND_LO;
 579:   }
 580: }
```

- EN: Function bodies or method definitions such as intCCToAVRCC contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: intCCToAVRCC 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 581-600

```cpp
 581: 
 582: /// Returns appropriate CP/CPI/CPC nodes code for the given 8/16-bit operands.
 583: SDValue AVRTargetLowering::getAVRCmp(SDValue LHS, SDValue RHS,
 584:                                      SelectionDAG &DAG, SDLoc DL) const {
 585:   assert((LHS.getSimpleValueType() == RHS.getSimpleValueType()) &&
 586:          "LHS and RHS have different types");
 587:   assert(((LHS.getSimpleValueType() == MVT::i16) ||
 588:           (LHS.getSimpleValueType() == MVT::i8)) &&
 589:          "invalid comparison type");
 590: 
 591:   SDValue Cmp;
 592: 
 593:   if (LHS.getSimpleValueType() == MVT::i16 && isa<ConstantSDNode>(RHS)) {
 594:     uint64_t Imm = RHS->getAsZExtVal();
 595:     // Generate a CPI/CPC pair if RHS is a 16-bit constant. Use the zero
 596:     // register for the constant RHS if its lower or higher byte is zero.
 597:     SDValue LHSlo = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i8, LHS,
 598:                                 DAG.getIntPtrConstant(0, DL));
 599:     SDValue LHShi = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i8, LHS,
 600:                                 DAG.getIntPtrConstant(1, DL));
```

- EN: Function bodies or method definitions such as getAVRCmp contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getAVRCmp 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 601-620

```cpp
 601:     SDValue RHSlo = (Imm & 0xff) == 0
 602:                         ? DAG.getRegister(Subtarget.getZeroRegister(), MVT::i8)
 603:                         : DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i8, RHS,
 604:                                       DAG.getIntPtrConstant(0, DL));
 605:     SDValue RHShi = (Imm & 0xff00) == 0
 606:                         ? DAG.getRegister(Subtarget.getZeroRegister(), MVT::i8)
 607:                         : DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i8, RHS,
 608:                                       DAG.getIntPtrConstant(1, DL));
 609:     Cmp = DAG.getNode(AVRISD::CMP, DL, MVT::Glue, LHSlo, RHSlo);
 610:     Cmp = DAG.getNode(AVRISD::CMPC, DL, MVT::Glue, LHShi, RHShi, Cmp);
 611:   } else if (RHS.getSimpleValueType() == MVT::i16 && isa<ConstantSDNode>(LHS)) {
 612:     // Generate a CPI/CPC pair if LHS is a 16-bit constant. Use the zero
 613:     // register for the constant LHS if its lower or higher byte is zero.
 614:     uint64_t Imm = LHS->getAsZExtVal();
 615:     SDValue LHSlo = (Imm & 0xff) == 0
 616:                         ? DAG.getRegister(Subtarget.getZeroRegister(), MVT::i8)
 617:                         : DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i8, LHS,
 618:                                       DAG.getIntPtrConstant(0, DL));
 619:     SDValue LHShi = (Imm & 0xff00) == 0
 620:                         ? DAG.getRegister(Subtarget.getZeroRegister(), MVT::i8)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 621-640

```cpp
 621:                         : DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i8, LHS,
 622:                                       DAG.getIntPtrConstant(1, DL));
 623:     SDValue RHSlo = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i8, RHS,
 624:                                 DAG.getIntPtrConstant(0, DL));
 625:     SDValue RHShi = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i8, RHS,
 626:                                 DAG.getIntPtrConstant(1, DL));
 627:     Cmp = DAG.getNode(AVRISD::CMP, DL, MVT::Glue, LHSlo, RHSlo);
 628:     Cmp = DAG.getNode(AVRISD::CMPC, DL, MVT::Glue, LHShi, RHShi, Cmp);
 629:   } else {
 630:     // Generate ordinary 16-bit comparison.
 631:     Cmp = DAG.getNode(AVRISD::CMP, DL, MVT::Glue, LHS, RHS);
 632:   }
 633: 
 634:   return Cmp;
 635: }
 636: 
 637: /// Returns appropriate AVR CMP/CMPC nodes and corresponding condition code for
 638: /// the given operands.
 639: SDValue AVRTargetLowering::getAVRCmp(SDValue LHS, SDValue RHS, ISD::CondCode CC,
 640:                                      SDValue &AVRcc, SelectionDAG &DAG,
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 641-660

```cpp
 641:                                      SDLoc DL) const {
 642:   SDValue Cmp;
 643:   EVT VT = LHS.getValueType();
 644:   bool UseTest = false;
 645: 
 646:   switch (CC) {
 647:   default:
 648:     break;
 649:   case ISD::SETLE: {
 650:     // Swap operands and reverse the branching condition.
 651:     std::swap(LHS, RHS);
 652:     CC = ISD::SETGE;
 653:     break;
 654:   }
 655:   case ISD::SETGT: {
 656:     if (const ConstantSDNode *C = dyn_cast<ConstantSDNode>(RHS)) {
 657:       switch (C->getSExtValue()) {
 658:       case -1: {
 659:         // When doing lhs > -1 use a tst instruction on the top part of lhs
 660:         // and use brpl instead of using a chain of cp/cpc.
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 661-680

```cpp
 661:         UseTest = true;
 662:         AVRcc = DAG.getConstant(AVRCC::COND_PL, DL, MVT::i8);
 663:         break;
 664:       }
 665:       case 0: {
 666:         // Turn lhs > 0 into 0 < lhs since 0 can be materialized with
 667:         // __zero_reg__ in lhs.
 668:         RHS = LHS;
 669:         LHS = DAG.getConstant(0, DL, VT);
 670:         CC = ISD::SETLT;
 671:         break;
 672:       }
 673:       default: {
 674:         // Turn lhs < rhs with lhs constant into rhs >= lhs+1, this allows
 675:         // us to  fold the constant into the cmp instruction.
 676:         RHS = DAG.getSignedConstant(C->getSExtValue() + 1, DL, VT);
 677:         CC = ISD::SETGE;
 678:         break;
 679:       }
 680:       }
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 681-700

```cpp
 681:       break;
 682:     }
 683:     // Swap operands and reverse the branching condition.
 684:     std::swap(LHS, RHS);
 685:     CC = ISD::SETLT;
 686:     break;
 687:   }
 688:   case ISD::SETLT: {
 689:     if (const ConstantSDNode *C = dyn_cast<ConstantSDNode>(RHS)) {
 690:       switch (C->getSExtValue()) {
 691:       case 1: {
 692:         // Turn lhs < 1 into 0 >= lhs since 0 can be materialized with
 693:         // __zero_reg__ in lhs.
 694:         RHS = LHS;
 695:         LHS = DAG.getConstant(0, DL, VT);
 696:         CC = ISD::SETGE;
 697:         break;
 698:       }
 699:       case 0: {
 700:         // When doing lhs < 0 use a tst instruction on the top part of lhs
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 701-720

```cpp
 701:         // and use brmi instead of using a chain of cp/cpc.
 702:         UseTest = true;
 703:         AVRcc = DAG.getConstant(AVRCC::COND_MI, DL, MVT::i8);
 704:         break;
 705:       }
 706:       }
 707:     }
 708:     break;
 709:   }
 710:   case ISD::SETULE: {
 711:     // Swap operands and reverse the branching condition.
 712:     std::swap(LHS, RHS);
 713:     CC = ISD::SETUGE;
 714:     break;
 715:   }
 716:   case ISD::SETUGT: {
 717:     // Turn `lhs > rhs` with constant rhs into `lhs >= rhs + 1`, because this
 718:     // allows us to fold the constant into the cmp instruction.
 719:     if (const ConstantSDNode *C = dyn_cast<ConstantSDNode>(RHS)) {
 720:       if (C->getConstantIntValue()->isMaxValue(false)) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 721-740

```cpp
 721:         // Applying this optimization requires calculating rhs+1, which we can't
 722:         // do if that overflows; it can happen during i128->i64 lowering.
 723:       } else {
 724:         RHS = DAG.getConstant(C->getZExtValue() + 1, DL, VT);
 725:         CC = ISD::SETUGE;
 726:         break;
 727:       }
 728:     }
 729:     // Swap operands and reverse the branching condition.
 730:     std::swap(LHS, RHS);
 731:     CC = ISD::SETULT;
 732:     break;
 733:   }
 734:   }
 735: 
 736:   // Expand 32 and 64 bit comparisons with custom CMP and CMPC nodes instead of
 737:   // using the default and/or/xor expansion code which is much longer.
 738:   if (VT == MVT::i32) {
 739:     SDValue LHSlo = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i16, LHS,
 740:                                 DAG.getIntPtrConstant(0, DL));
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 741-760

```cpp
 741:     SDValue LHShi = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i16, LHS,
 742:                                 DAG.getIntPtrConstant(1, DL));
 743:     SDValue RHSlo = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i16, RHS,
 744:                                 DAG.getIntPtrConstant(0, DL));
 745:     SDValue RHShi = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i16, RHS,
 746:                                 DAG.getIntPtrConstant(1, DL));
 747: 
 748:     if (UseTest) {
 749:       // When using tst we only care about the highest part.
 750:       SDValue Top = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i8, LHShi,
 751:                                 DAG.getIntPtrConstant(1, DL));
 752:       Cmp = DAG.getNode(AVRISD::TST, DL, MVT::Glue, Top);
 753:     } else {
 754:       Cmp = getAVRCmp(LHSlo, RHSlo, DAG, DL);
 755:       Cmp = DAG.getNode(AVRISD::CMPC, DL, MVT::Glue, LHShi, RHShi, Cmp);
 756:     }
 757:   } else if (VT == MVT::i64) {
 758:     SDValue LHS_0 = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i32, LHS,
 759:                                 DAG.getIntPtrConstant(0, DL));
 760:     SDValue LHS_1 = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i32, LHS,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 761-780

```cpp
 761:                                 DAG.getIntPtrConstant(1, DL));
 762: 
 763:     SDValue LHS0 = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i16, LHS_0,
 764:                                DAG.getIntPtrConstant(0, DL));
 765:     SDValue LHS1 = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i16, LHS_0,
 766:                                DAG.getIntPtrConstant(1, DL));
 767:     SDValue LHS2 = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i16, LHS_1,
 768:                                DAG.getIntPtrConstant(0, DL));
 769:     SDValue LHS3 = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i16, LHS_1,
 770:                                DAG.getIntPtrConstant(1, DL));
 771: 
 772:     SDValue RHS_0 = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i32, RHS,
 773:                                 DAG.getIntPtrConstant(0, DL));
 774:     SDValue RHS_1 = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i32, RHS,
 775:                                 DAG.getIntPtrConstant(1, DL));
 776: 
 777:     SDValue RHS0 = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i16, RHS_0,
 778:                                DAG.getIntPtrConstant(0, DL));
 779:     SDValue RHS1 = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i16, RHS_0,
 780:                                DAG.getIntPtrConstant(1, DL));
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 781-800

```cpp
 781:     SDValue RHS2 = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i16, RHS_1,
 782:                                DAG.getIntPtrConstant(0, DL));
 783:     SDValue RHS3 = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i16, RHS_1,
 784:                                DAG.getIntPtrConstant(1, DL));
 785: 
 786:     if (UseTest) {
 787:       // When using tst we only care about the highest part.
 788:       SDValue Top = DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i8, LHS3,
 789:                                 DAG.getIntPtrConstant(1, DL));
 790:       Cmp = DAG.getNode(AVRISD::TST, DL, MVT::Glue, Top);
 791:     } else {
 792:       Cmp = getAVRCmp(LHS0, RHS0, DAG, DL);
 793:       Cmp = DAG.getNode(AVRISD::CMPC, DL, MVT::Glue, LHS1, RHS1, Cmp);
 794:       Cmp = DAG.getNode(AVRISD::CMPC, DL, MVT::Glue, LHS2, RHS2, Cmp);
 795:       Cmp = DAG.getNode(AVRISD::CMPC, DL, MVT::Glue, LHS3, RHS3, Cmp);
 796:     }
 797:   } else if (VT == MVT::i8 || VT == MVT::i16) {
 798:     if (UseTest) {
 799:       // When using tst we only care about the highest part.
 800:       Cmp = DAG.getNode(AVRISD::TST, DL, MVT::Glue,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 801-820

```cpp
 801:                         (VT == MVT::i8)
 802:                             ? LHS
 803:                             : DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i8,
 804:                                           LHS, DAG.getIntPtrConstant(1, DL)));
 805:     } else {
 806:       Cmp = getAVRCmp(LHS, RHS, DAG, DL);
 807:     }
 808:   } else {
 809:     llvm_unreachable("Invalid comparison size");
 810:   }
 811: 
 812:   // When using a test instruction AVRcc is already set.
 813:   if (!UseTest) {
 814:     AVRcc = DAG.getConstant(intCCToAVRCC(CC), DL, MVT::i8);
 815:   }
 816: 
 817:   return Cmp;
 818: }
 819: 
 820: SDValue AVRTargetLowering::LowerBR_CC(SDValue Op, SelectionDAG &DAG) const {
```

- EN: Function bodies or method definitions such as LowerBR_CC contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerBR_CC 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 821-840

```cpp
 821:   SDValue Chain = Op.getOperand(0);
 822:   ISD::CondCode CC = cast<CondCodeSDNode>(Op.getOperand(1))->get();
 823:   SDValue LHS = Op.getOperand(2);
 824:   SDValue RHS = Op.getOperand(3);
 825:   SDValue Dest = Op.getOperand(4);
 826:   SDLoc dl(Op);
 827: 
 828:   SDValue TargetCC;
 829:   SDValue Cmp = getAVRCmp(LHS, RHS, CC, TargetCC, DAG, dl);
 830: 
 831:   return DAG.getNode(AVRISD::BRCOND, dl, MVT::Other, Chain, Dest, TargetCC,
 832:                      Cmp);
 833: }
 834: 
 835: SDValue AVRTargetLowering::LowerSELECT_CC(SDValue Op, SelectionDAG &DAG) const {
 836:   SDValue LHS = Op.getOperand(0);
 837:   SDValue RHS = Op.getOperand(1);
 838:   SDValue TrueV = Op.getOperand(2);
 839:   SDValue FalseV = Op.getOperand(3);
 840:   ISD::CondCode CC = cast<CondCodeSDNode>(Op.getOperand(4))->get();
```

- EN: Function bodies or method definitions such as LowerSELECT_CC contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerSELECT_CC 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 841-860

```cpp
 841:   SDLoc dl(Op);
 842: 
 843:   SDValue TargetCC;
 844:   SDValue Cmp = getAVRCmp(LHS, RHS, CC, TargetCC, DAG, dl);
 845: 
 846:   SDValue Ops[] = {TrueV, FalseV, TargetCC, Cmp};
 847: 
 848:   return DAG.getNode(AVRISD::SELECT_CC, dl, Op.getValueType(), Ops);
 849: }
 850: 
 851: SDValue AVRTargetLowering::LowerSETCC(SDValue Op, SelectionDAG &DAG) const {
 852:   SDValue LHS = Op.getOperand(0);
 853:   SDValue RHS = Op.getOperand(1);
 854:   ISD::CondCode CC = cast<CondCodeSDNode>(Op.getOperand(2))->get();
 855:   SDLoc DL(Op);
 856: 
 857:   SDValue TargetCC;
 858:   SDValue Cmp = getAVRCmp(LHS, RHS, CC, TargetCC, DAG, DL);
 859: 
 860:   SDValue TrueV = DAG.getConstant(1, DL, Op.getValueType());
```

- EN: Function bodies or method definitions such as LowerSETCC contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerSETCC 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 861-880

```cpp
 861:   SDValue FalseV = DAG.getConstant(0, DL, Op.getValueType());
 862:   SDValue Ops[] = {TrueV, FalseV, TargetCC, Cmp};
 863: 
 864:   return DAG.getNode(AVRISD::SELECT_CC, DL, Op.getValueType(), Ops);
 865: }
 866: 
 867: SDValue AVRTargetLowering::LowerVASTART(SDValue Op, SelectionDAG &DAG) const {
 868:   const MachineFunction &MF = DAG.getMachineFunction();
 869:   const AVRMachineFunctionInfo *AFI = MF.getInfo<AVRMachineFunctionInfo>();
 870:   const Value *SV = cast<SrcValueSDNode>(Op.getOperand(2))->getValue();
 871:   auto DL = DAG.getDataLayout();
 872:   SDLoc dl(Op);
 873: 
 874:   // Vastart just stores the address of the VarArgsFrameIndex slot into the
 875:   // memory location argument.
 876:   SDValue FI = DAG.getFrameIndex(AFI->getVarArgsFrameIndex(), getPointerTy(DL));
 877: 
 878:   return DAG.getStore(Op.getOperand(0), dl, FI, Op.getOperand(1),
 879:                       MachinePointerInfo(SV));
 880: }
```

- EN: Function bodies or method definitions such as LowerVASTART contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerVASTART 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 881-900

```cpp
 881: 
 882: // Modify the existing ISD::INLINEASM node to add the implicit zero register.
 883: SDValue AVRTargetLowering::LowerINLINEASM(SDValue Op, SelectionDAG &DAG) const {
 884:   SDValue ZeroReg = DAG.getRegister(Subtarget.getZeroRegister(), MVT::i8);
 885:   if (Op.getOperand(Op.getNumOperands() - 1) == ZeroReg ||
 886:       Op.getOperand(Op.getNumOperands() - 2) == ZeroReg) {
 887:     // Zero register has already been added. Don't add it again.
 888:     // If this isn't handled, we get called over and over again.
 889:     return Op;
 890:   }
 891: 
 892:   // Get a list of operands to the new INLINEASM node. This is mostly a copy,
 893:   // with some edits.
 894:   // Add the following operands at the end (but before the glue node, if it's
 895:   // there):
 896:   //  - The flags of the implicit zero register operand.
 897:   //  - The implicit zero register operand itself.
 898:   SDLoc dl(Op);
 899:   SmallVector<SDValue, 8> Ops;
 900:   SDNode *N = Op.getNode();
```

- EN: Function bodies or method definitions such as LowerINLINEASM contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerINLINEASM 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 901-920

```cpp
 901:   SDValue Glue;
 902:   for (unsigned I = 0; I < N->getNumOperands(); I++) {
 903:     SDValue Operand = N->getOperand(I);
 904:     if (Operand.getValueType() == MVT::Glue) {
 905:       // The glue operand always needs to be at the end, so we need to treat it
 906:       // specially.
 907:       Glue = Operand;
 908:     } else {
 909:       Ops.push_back(Operand);
 910:     }
 911:   }
 912:   InlineAsm::Flag Flags(InlineAsm::Kind::RegUse, 1);
 913:   Ops.push_back(DAG.getTargetConstant(Flags, dl, MVT::i32));
 914:   Ops.push_back(ZeroReg);
 915:   if (Glue) {
 916:     Ops.push_back(Glue);
 917:   }
 918: 
 919:   // Replace the current INLINEASM node with a new one that has the zero
 920:   // register as implicit parameter.
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 921-940

```cpp
 921:   SDValue New = DAG.getNode(N->getOpcode(), dl, N->getVTList(), Ops);
 922:   DAG.ReplaceAllUsesOfValueWith(Op, New);
 923:   DAG.ReplaceAllUsesOfValueWith(Op.getValue(1), New.getValue(1));
 924: 
 925:   return New;
 926: }
 927: 
 928: SDValue AVRTargetLowering::LowerOperation(SDValue Op, SelectionDAG &DAG) const {
 929:   switch (Op.getOpcode()) {
 930:   default:
 931:     llvm_unreachable("Don't know how to custom lower this!");
 932:   case ISD::SHL:
 933:   case ISD::SRA:
 934:   case ISD::SRL:
 935:   case ISD::ROTL:
 936:   case ISD::ROTR:
 937:     return LowerShifts(Op, DAG);
 938:   case ISD::GlobalAddress:
 939:     return LowerGlobalAddress(Op, DAG);
 940:   case ISD::BlockAddress:
```

- EN: Function bodies or method definitions such as LowerOperation contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerOperation 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 941-960

```cpp
 941:     return LowerBlockAddress(Op, DAG);
 942:   case ISD::BR_CC:
 943:     return LowerBR_CC(Op, DAG);
 944:   case ISD::SELECT_CC:
 945:     return LowerSELECT_CC(Op, DAG);
 946:   case ISD::SETCC:
 947:     return LowerSETCC(Op, DAG);
 948:   case ISD::VASTART:
 949:     return LowerVASTART(Op, DAG);
 950:   case ISD::SDIVREM:
 951:   case ISD::UDIVREM:
 952:     return LowerDivRem(Op, DAG);
 953:   case ISD::INLINEASM:
 954:     return LowerINLINEASM(Op, DAG);
 955:   }
 956: 
 957:   return SDValue();
 958: }
 959: 
 960: /// Replace a node with an illegal result type
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 961-980

```cpp
 961: /// with a new node built out of custom code.
 962: void AVRTargetLowering::ReplaceNodeResults(SDNode *N,
 963:                                            SmallVectorImpl<SDValue> &Results,
 964:                                            SelectionDAG &DAG) const {
 965:   SDLoc DL(N);
 966: 
 967:   switch (N->getOpcode()) {
 968:   case ISD::ADD: {
 969:     // Convert add (x, imm) into sub (x, -imm).
 970:     if (const ConstantSDNode *C = dyn_cast<ConstantSDNode>(N->getOperand(1))) {
 971:       SDValue Sub = DAG.getNode(
 972:           ISD::SUB, DL, N->getValueType(0), N->getOperand(0),
 973:           DAG.getConstant(-C->getAPIntValue(), DL, C->getValueType(0)));
 974:       Results.push_back(Sub);
 975:     }
 976:     break;
 977:   }
 978:   default: {
 979:     SDValue Res = LowerOperation(SDValue(N, 0), DAG);
 980: 
```

- EN: Function bodies or method definitions such as ReplaceNodeResults contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: ReplaceNodeResults 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 981-1000

```cpp
 981:     for (unsigned I = 0, E = Res->getNumValues(); I != E; ++I)
 982:       Results.push_back(Res.getValue(I));
 983: 
 984:     break;
 985:   }
 986:   }
 987: }
 988: 
 989: /// Return true if the addressing mode represented
 990: /// by AM is legal for this target, for a load/store of the specified type.
 991: bool AVRTargetLowering::isLegalAddressingMode(const DataLayout &DL,
 992:                                               const AddrMode &AM, Type *Ty,
 993:                                               unsigned AS,
 994:                                               Instruction *I) const {
 995:   int64_t Offs = AM.BaseOffs;
 996: 
 997:   // Allow absolute addresses.
 998:   if (AM.BaseGV && !AM.HasBaseReg && AM.Scale == 0 && Offs == 0) {
 999:     return true;
1000:   }
```

- EN: Function bodies or method definitions such as isLegalAddressingMode contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: isLegalAddressingMode 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1001-1020

```cpp
1001: 
1002:   // Flash memory instructions only allow zero offsets.
1003:   if (isa<PointerType>(Ty) && AS == AVR::ProgramMemory) {
1004:     return false;
1005:   }
1006: 
1007:   // Allow reg+<6bit> offset.
1008:   if (Offs < 0)
1009:     Offs = -Offs;
1010:   if (AM.BaseGV == nullptr && AM.HasBaseReg && AM.Scale == 0 &&
1011:       isUInt<6>(Offs)) {
1012:     return true;
1013:   }
1014: 
1015:   return false;
1016: }
1017: 
1018: /// Returns true by value, base pointer and
1019: /// offset pointer and addressing mode by reference if the node's address
1020: /// can be legally represented as pre-indexed load / store address.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1021-1040

```cpp
1021: bool AVRTargetLowering::getPreIndexedAddressParts(SDNode *N, SDValue &Base,
1022:                                                   SDValue &Offset,
1023:                                                   ISD::MemIndexedMode &AM,
1024:                                                   SelectionDAG &DAG) const {
1025:   EVT VT;
1026:   const SDNode *Op;
1027:   SDLoc DL(N);
1028: 
1029:   if (const LoadSDNode *LD = dyn_cast<LoadSDNode>(N)) {
1030:     VT = LD->getMemoryVT();
1031:     Op = LD->getBasePtr().getNode();
1032:     if (LD->getExtensionType() != ISD::NON_EXTLOAD)
1033:       return false;
1034:     if (AVR::isProgramMemoryAccess(LD)) {
1035:       return false;
1036:     }
1037:   } else if (const StoreSDNode *ST = dyn_cast<StoreSDNode>(N)) {
1038:     VT = ST->getMemoryVT();
1039:     Op = ST->getBasePtr().getNode();
1040:     if (AVR::isProgramMemoryAccess(ST)) {
```

- EN: Function bodies or method definitions such as getPreIndexedAddressParts contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getPreIndexedAddressParts 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1041-1060

```cpp
1041:       return false;
1042:     }
1043:   } else {
1044:     return false;
1045:   }
1046: 
1047:   if (VT != MVT::i8 && VT != MVT::i16) {
1048:     return false;
1049:   }
1050: 
1051:   if (Op->getOpcode() != ISD::ADD && Op->getOpcode() != ISD::SUB) {
1052:     return false;
1053:   }
1054: 
1055:   if (const ConstantSDNode *RHS = dyn_cast<ConstantSDNode>(Op->getOperand(1))) {
1056:     int RHSC = RHS->getSExtValue();
1057:     if (Op->getOpcode() == ISD::SUB)
1058:       RHSC = -RHSC;
1059: 
1060:     if ((VT == MVT::i16 && RHSC != -2) || (VT == MVT::i8 && RHSC != -1)) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1061-1080

```cpp
1061:       return false;
1062:     }
1063: 
1064:     Base = Op->getOperand(0);
1065:     Offset = DAG.getSignedConstant(RHSC, DL, MVT::i8);
1066:     AM = ISD::PRE_DEC;
1067: 
1068:     return true;
1069:   }
1070: 
1071:   return false;
1072: }
1073: 
1074: /// Returns true by value, base pointer and
1075: /// offset pointer and addressing mode by reference if this node can be
1076: /// combined with a load / store to form a post-indexed load / store.
1077: bool AVRTargetLowering::getPostIndexedAddressParts(SDNode *N, SDNode *Op,
1078:                                                    SDValue &Base,
1079:                                                    SDValue &Offset,
1080:                                                    ISD::MemIndexedMode &AM,
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1081-1100

```cpp
1081:                                                    SelectionDAG &DAG) const {
1082:   EVT VT;
1083:   SDValue Ptr;
1084:   SDLoc DL(N);
1085: 
1086:   if (const LoadSDNode *LD = dyn_cast<LoadSDNode>(N)) {
1087:     VT = LD->getMemoryVT();
1088:     Ptr = LD->getBasePtr();
1089:     if (LD->getExtensionType() != ISD::NON_EXTLOAD)
1090:       return false;
1091:   } else if (const StoreSDNode *ST = dyn_cast<StoreSDNode>(N)) {
1092:     VT = ST->getMemoryVT();
1093:     Ptr = ST->getBasePtr();
1094:     // We can not store to program memory.
1095:     if (AVR::isProgramMemoryAccess(ST))
1096:       return false;
1097:     // Since the high byte need to be stored first, we can not emit
1098:     // i16 post increment store like:
1099:     // st X+, r24
1100:     // st X+, r25
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1101-1120

```cpp
1101:     if (VT == MVT::i16 && !Subtarget.hasLowByteFirst())
1102:       return false;
1103:   } else {
1104:     return false;
1105:   }
1106: 
1107:   if (VT != MVT::i8 && VT != MVT::i16) {
1108:     return false;
1109:   }
1110: 
1111:   if (Op->getOpcode() != ISD::ADD && Op->getOpcode() != ISD::SUB) {
1112:     return false;
1113:   }
1114: 
1115:   if (const ConstantSDNode *RHS = dyn_cast<ConstantSDNode>(Op->getOperand(1))) {
1116:     int RHSC = RHS->getSExtValue();
1117:     if (Op->getOpcode() == ISD::SUB)
1118:       RHSC = -RHSC;
1119:     if ((VT == MVT::i16 && RHSC != 2) || (VT == MVT::i8 && RHSC != 1)) {
1120:       return false;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1121-1140

```cpp
1121:     }
1122: 
1123:     // FIXME: We temporarily disable post increment load from program memory,
1124:     //        due to bug https://github.com/llvm/llvm-project/issues/59914.
1125:     if (const LoadSDNode *LD = dyn_cast<LoadSDNode>(N))
1126:       if (AVR::isProgramMemoryAccess(LD))
1127:         return false;
1128: 
1129:     Base = Op->getOperand(0);
1130: 
1131:     // Post-indexing updates the base, so it's not a valid transform
1132:     // if that's not the same as the load's pointer.
1133:     if (Ptr != Base)
1134:       return false;
1135: 
1136:     Offset = DAG.getConstant(RHSC, DL, MVT::i8);
1137:     AM = ISD::POST_INC;
1138: 
1139:     return true;
1140:   }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1141-1160

```cpp
1141: 
1142:   return false;
1143: }
1144: 
1145: bool AVRTargetLowering::isOffsetFoldingLegal(
1146:     const GlobalAddressSDNode *GA) const {
1147:   return true;
1148: }
1149: 
1150: //===----------------------------------------------------------------------===//
1151: //             Formal Arguments Calling Convention Implementation
1152: //===----------------------------------------------------------------------===//
1153: 
1154: #include "AVRGenCallingConv.inc"
1155: 
1156: /// Registers for calling conventions, ordered in reverse as required by ABI.
1157: /// Both arrays must be of the same length.
1158: static const MCPhysReg RegList8AVR[] = {
1159:     AVR::R25, AVR::R24, AVR::R23, AVR::R22, AVR::R21, AVR::R20,
1160:     AVR::R19, AVR::R18, AVR::R17, AVR::R16, AVR::R15, AVR::R14,
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as isOffsetFoldingLegal contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 isOffsetFoldingLegal 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1161-1180

```cpp
1161:     AVR::R13, AVR::R12, AVR::R11, AVR::R10, AVR::R9,  AVR::R8};
1162: static const MCPhysReg RegList8Tiny[] = {AVR::R25, AVR::R24, AVR::R23,
1163:                                          AVR::R22, AVR::R21, AVR::R20};
1164: static const MCPhysReg RegList16AVR[] = {
1165:     AVR::R26R25, AVR::R25R24, AVR::R24R23, AVR::R23R22, AVR::R22R21,
1166:     AVR::R21R20, AVR::R20R19, AVR::R19R18, AVR::R18R17, AVR::R17R16,
1167:     AVR::R16R15, AVR::R15R14, AVR::R14R13, AVR::R13R12, AVR::R12R11,
1168:     AVR::R11R10, AVR::R10R9,  AVR::R9R8};
1169: static const MCPhysReg RegList16Tiny[] = {AVR::R26R25, AVR::R25R24,
1170:                                           AVR::R24R23, AVR::R23R22,
1171:                                           AVR::R22R21, AVR::R21R20};
1172: 
1173: static_assert(std::size(RegList8AVR) == std::size(RegList16AVR),
1174:               "8-bit and 16-bit register arrays must be of equal length");
1175: static_assert(std::size(RegList8Tiny) == std::size(RegList16Tiny),
1176:               "8-bit and 16-bit register arrays must be of equal length");
1177: 
1178: /// Analyze incoming and outgoing function arguments. We need custom C++ code
1179: /// to handle special constraints in the ABI.
1180: /// In addition, all pieces of a certain argument have to be passed either
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1181-1200

```cpp
1181: /// using registers or the stack but never mixing both.
1182: template <typename ArgT>
1183: static void analyzeArguments(TargetLowering::CallLoweringInfo *CLI,
1184:                              const Function *F, const DataLayout *TD,
1185:                              const SmallVectorImpl<ArgT> &Args,
1186:                              SmallVectorImpl<CCValAssign> &ArgLocs,
1187:                              CCState &CCInfo, bool Tiny) {
1188:   // Choose the proper register list for argument passing according to the ABI.
1189:   ArrayRef<MCPhysReg> RegList8;
1190:   ArrayRef<MCPhysReg> RegList16;
1191:   if (Tiny) {
1192:     RegList8 = ArrayRef(RegList8Tiny);
1193:     RegList16 = ArrayRef(RegList16Tiny);
1194:   } else {
1195:     RegList8 = ArrayRef(RegList8AVR);
1196:     RegList16 = ArrayRef(RegList16AVR);
1197:   }
1198: 
1199:   unsigned NumArgs = Args.size();
1200:   // This is the index of the last used register, in RegList*.
```

- EN: Function bodies or method definitions such as analyzeArguments contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: analyzeArguments 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1201-1220

```cpp
1201:   // -1 means R26 (R26 is never actually used in CC).
1202:   int RegLastIdx = -1;
1203:   // Once a value is passed to the stack it will always be used
1204:   bool UseStack = false;
1205:   for (unsigned i = 0; i != NumArgs;) {
1206:     MVT VT = Args[i].VT;
1207:     // We have to count the number of bytes for each function argument, that is
1208:     // those Args with the same OrigArgIndex. This is important in case the
1209:     // function takes an aggregate type.
1210:     // Current argument will be between [i..j).
1211:     unsigned ArgIndex = Args[i].OrigArgIndex;
1212:     unsigned TotalBytes = VT.getStoreSize();
1213:     unsigned j = i + 1;
1214:     for (; j != NumArgs; ++j) {
1215:       if (Args[j].OrigArgIndex != ArgIndex)
1216:         break;
1217:       TotalBytes += Args[j].VT.getStoreSize();
1218:     }
1219:     // Round up to even number of bytes.
1220:     TotalBytes = alignTo(TotalBytes, 2);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1221-1240

```cpp
1221:     // Skip zero sized arguments
1222:     if (TotalBytes == 0)
1223:       continue;
1224:     // The index of the first register to be used
1225:     unsigned RegIdx = RegLastIdx + TotalBytes;
1226:     RegLastIdx = RegIdx;
1227:     // If there are not enough registers, use the stack
1228:     if (RegIdx >= RegList8.size()) {
1229:       UseStack = true;
1230:     }
1231:     for (; i != j; ++i) {
1232:       MVT VT = Args[i].VT;
1233: 
1234:       if (UseStack) {
1235:         auto evt = EVT(VT).getTypeForEVT(CCInfo.getContext());
1236:         unsigned Offset = CCInfo.AllocateStack(TD->getTypeAllocSize(evt),
1237:                                                TD->getABITypeAlign(evt));
1238:         CCInfo.addLoc(
1239:             CCValAssign::getMem(i, VT, Offset, VT, CCValAssign::Full));
1240:       } else {
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1241-1260

```cpp
1241:         unsigned Reg;
1242:         if (VT == MVT::i8) {
1243:           Reg = CCInfo.AllocateReg(RegList8[RegIdx]);
1244:         } else if (VT == MVT::i16) {
1245:           Reg = CCInfo.AllocateReg(RegList16[RegIdx]);
1246:         } else {
1247:           llvm_unreachable(
1248:               "calling convention can only manage i8 and i16 types");
1249:         }
1250:         assert(Reg && "register not available in calling convention");
1251:         CCInfo.addLoc(CCValAssign::getReg(i, VT, Reg, VT, CCValAssign::Full));
1252:         // Registers inside a particular argument are sorted in increasing order
1253:         // (remember the array is reversed).
1254:         RegIdx -= VT.getStoreSize();
1255:       }
1256:     }
1257:   }
1258: }
1259: 
1260: /// Count the total number of bytes needed to pass or return these arguments.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1261-1280

```cpp
1261: template <typename ArgT>
1262: static unsigned
1263: getTotalArgumentsSizeInBytes(const SmallVectorImpl<ArgT> &Args) {
1264:   unsigned TotalBytes = 0;
1265: 
1266:   for (const ArgT &Arg : Args) {
1267:     TotalBytes += Arg.VT.getStoreSize();
1268:   }
1269:   return TotalBytes;
1270: }
1271: 
1272: /// Analyze incoming and outgoing value of returning from a function.
1273: /// The algorithm is similar to analyzeArguments, but there can only be
1274: /// one value, possibly an aggregate, and it is limited to 8 bytes.
1275: template <typename ArgT>
1276: static void analyzeReturnValues(const SmallVectorImpl<ArgT> &Args,
1277:                                 CCState &CCInfo, bool Tiny) {
1278:   unsigned NumArgs = Args.size();
1279:   unsigned TotalBytes = getTotalArgumentsSizeInBytes(Args);
1280:   // CanLowerReturn() guarantees this assertion.
```

- EN: Function bodies or method definitions such as getTotalArgumentsSizeInBytes, analyzeReturnValues contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getTotalArgumentsSizeInBytes, analyzeReturnValues 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1281-1300

```cpp
1281:   if (Tiny)
1282:     assert(TotalBytes <= 4 &&
1283:            "return values greater than 4 bytes cannot be lowered on AVRTiny");
1284:   else
1285:     assert(TotalBytes <= 8 &&
1286:            "return values greater than 8 bytes cannot be lowered on AVR");
1287: 
1288:   // Choose the proper register list for argument passing according to the ABI.
1289:   ArrayRef<MCPhysReg> RegList8;
1290:   ArrayRef<MCPhysReg> RegList16;
1291:   if (Tiny) {
1292:     RegList8 = ArrayRef(RegList8Tiny);
1293:     RegList16 = ArrayRef(RegList16Tiny);
1294:   } else {
1295:     RegList8 = ArrayRef(RegList8AVR);
1296:     RegList16 = ArrayRef(RegList16AVR);
1297:   }
1298: 
1299:   // GCC-ABI says that the size is rounded up to the next even number,
1300:   // but actually once it is more than 4 it will always round up to 8.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1301-1320

```cpp
1301:   if (TotalBytes > 4) {
1302:     TotalBytes = 8;
1303:   } else {
1304:     TotalBytes = alignTo(TotalBytes, 2);
1305:   }
1306: 
1307:   // The index of the first register to use.
1308:   int RegIdx = TotalBytes - 1;
1309:   for (unsigned i = 0; i != NumArgs; ++i) {
1310:     MVT VT = Args[i].VT;
1311:     unsigned Reg;
1312:     if (VT == MVT::i8) {
1313:       Reg = CCInfo.AllocateReg(RegList8[RegIdx]);
1314:     } else if (VT == MVT::i16) {
1315:       Reg = CCInfo.AllocateReg(RegList16[RegIdx]);
1316:     } else {
1317:       llvm_unreachable("calling convention can only manage i8 and i16 types");
1318:     }
1319:     assert(Reg && "register not available in calling convention");
1320:     CCInfo.addLoc(CCValAssign::getReg(i, VT, Reg, VT, CCValAssign::Full));
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1321-1340

```cpp
1321:     // Registers sort in increasing order
1322:     RegIdx -= VT.getStoreSize();
1323:   }
1324: }
1325: 
1326: SDValue AVRTargetLowering::LowerFormalArguments(
1327:     SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
1328:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &dl,
1329:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
1330:   MachineFunction &MF = DAG.getMachineFunction();
1331:   MachineFrameInfo &MFI = MF.getFrameInfo();
1332:   auto DL = DAG.getDataLayout();
1333: 
1334:   // Assign locations to all of the incoming arguments.
1335:   SmallVector<CCValAssign, 16> ArgLocs;
1336:   CCState CCInfo(CallConv, isVarArg, DAG.getMachineFunction(), ArgLocs,
1337:                  *DAG.getContext());
1338: 
1339:   // Variadic functions do not need all the analysis below.
1340:   if (isVarArg) {
```

- EN: Function bodies or method definitions such as LowerFormalArguments contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerFormalArguments 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1341-1360

```cpp
1341:     CCInfo.AnalyzeFormalArguments(Ins, ArgCC_AVR_Vararg);
1342:   } else {
1343:     analyzeArguments(nullptr, &MF.getFunction(), &DL, Ins, ArgLocs, CCInfo,
1344:                      Subtarget.hasTinyEncoding());
1345:   }
1346: 
1347:   SDValue ArgValue;
1348:   for (CCValAssign &VA : ArgLocs) {
1349: 
1350:     // Arguments stored on registers.
1351:     if (VA.isRegLoc()) {
1352:       EVT RegVT = VA.getLocVT();
1353:       const TargetRegisterClass *RC;
1354:       if (RegVT == MVT::i8) {
1355:         RC = &AVR::GPR8RegClass;
1356:       } else if (RegVT == MVT::i16) {
1357:         RC = &AVR::DREGSRegClass;
1358:       } else {
1359:         llvm_unreachable("Unknown argument type!");
1360:       }
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1361-1380

```cpp
1361: 
1362:       Register Reg = MF.addLiveIn(VA.getLocReg(), RC);
1363:       ArgValue = DAG.getCopyFromReg(Chain, dl, Reg, RegVT);
1364: 
1365:       // :NOTE: Clang should not promote any i8 into i16 but for safety the
1366:       // following code will handle zexts or sexts generated by other
1367:       // front ends. Otherwise:
1368:       // If this is an 8 bit value, it is really passed promoted
1369:       // to 16 bits. Insert an assert[sz]ext to capture this, then
1370:       // truncate to the right size.
1371:       switch (VA.getLocInfo()) {
1372:       default:
1373:         llvm_unreachable("Unknown loc info!");
1374:       case CCValAssign::Full:
1375:         break;
1376:       case CCValAssign::BCvt:
1377:         ArgValue = DAG.getNode(ISD::BITCAST, dl, VA.getValVT(), ArgValue);
1378:         break;
1379:       case CCValAssign::SExt:
1380:         ArgValue = DAG.getNode(ISD::AssertSext, dl, RegVT, ArgValue,
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1381-1400

```cpp
1381:                                DAG.getValueType(VA.getValVT()));
1382:         ArgValue = DAG.getNode(ISD::TRUNCATE, dl, VA.getValVT(), ArgValue);
1383:         break;
1384:       case CCValAssign::ZExt:
1385:         ArgValue = DAG.getNode(ISD::AssertZext, dl, RegVT, ArgValue,
1386:                                DAG.getValueType(VA.getValVT()));
1387:         ArgValue = DAG.getNode(ISD::TRUNCATE, dl, VA.getValVT(), ArgValue);
1388:         break;
1389:       }
1390: 
1391:       InVals.push_back(ArgValue);
1392:     } else {
1393:       // Only arguments passed on the stack should make it here.
1394:       assert(VA.isMemLoc());
1395: 
1396:       EVT LocVT = VA.getLocVT();
1397: 
1398:       // Create the frame index object for this incoming parameter.
1399:       int FI = MFI.CreateFixedObject(LocVT.getSizeInBits() / 8,
1400:                                      VA.getLocMemOffset(), true);
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1401-1420

```cpp
1401: 
1402:       // Create the SelectionDAG nodes corresponding to a load
1403:       // from this parameter.
1404:       SDValue FIN = DAG.getFrameIndex(FI, getPointerTy(DL));
1405:       InVals.push_back(DAG.getLoad(LocVT, dl, Chain, FIN,
1406:                                    MachinePointerInfo::getFixedStack(MF, FI)));
1407:     }
1408:   }
1409: 
1410:   // If the function takes variable number of arguments, make a frame index for
1411:   // the start of the first vararg value... for expansion of llvm.va_start.
1412:   if (isVarArg) {
1413:     unsigned StackSize = CCInfo.getStackSize();
1414:     AVRMachineFunctionInfo *AFI = MF.getInfo<AVRMachineFunctionInfo>();
1415: 
1416:     AFI->setVarArgsFrameIndex(MFI.CreateFixedObject(2, StackSize, true));
1417:   }
1418: 
1419:   return Chain;
1420: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1421-1440

```cpp
1421: 
1422: //===----------------------------------------------------------------------===//
1423: //                  Call Calling Convention Implementation
1424: //===----------------------------------------------------------------------===//
1425: 
1426: SDValue AVRTargetLowering::LowerCall(TargetLowering::CallLoweringInfo &CLI,
1427:                                      SmallVectorImpl<SDValue> &InVals) const {
1428:   SelectionDAG &DAG = CLI.DAG;
1429:   SDLoc &DL = CLI.DL;
1430:   SmallVectorImpl<ISD::OutputArg> &Outs = CLI.Outs;
1431:   SmallVectorImpl<SDValue> &OutVals = CLI.OutVals;
1432:   SmallVectorImpl<ISD::InputArg> &Ins = CLI.Ins;
1433:   SDValue Chain = CLI.Chain;
1434:   SDValue Callee = CLI.Callee;
1435:   bool &isTailCall = CLI.IsTailCall;
1436:   CallingConv::ID CallConv = CLI.CallConv;
1437:   bool isVarArg = CLI.IsVarArg;
1438: 
1439:   MachineFunction &MF = DAG.getMachineFunction();
1440: 
```

- EN: Function bodies or method definitions such as LowerCall contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerCall 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1441-1460

```cpp
1441:   // AVR does not yet support tail call optimization.
1442:   isTailCall = false;
1443: 
1444:   // Analyze operands of the call, assigning locations to each operand.
1445:   SmallVector<CCValAssign, 16> ArgLocs;
1446:   CCState CCInfo(CallConv, isVarArg, DAG.getMachineFunction(), ArgLocs,
1447:                  *DAG.getContext());
1448: 
1449:   // If the callee is a GlobalAddress/ExternalSymbol node (quite common, every
1450:   // direct call is) turn it into a TargetGlobalAddress/TargetExternalSymbol
1451:   // node so that legalize doesn't hack it.
1452:   const Function *F = nullptr;
1453:   if (const GlobalAddressSDNode *G = dyn_cast<GlobalAddressSDNode>(Callee)) {
1454:     const GlobalValue *GV = G->getGlobal();
1455:     if (isa<Function>(GV))
1456:       F = cast<Function>(GV);
1457:     Callee =
1458:         DAG.getTargetGlobalAddress(GV, DL, getPointerTy(DAG.getDataLayout()));
1459:   } else if (const ExternalSymbolSDNode *ES =
1460:                  dyn_cast<ExternalSymbolSDNode>(Callee)) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1461-1480

```cpp
1461:     Callee = DAG.getTargetExternalSymbol(ES->getSymbol(),
1462:                                          getPointerTy(DAG.getDataLayout()));
1463:   }
1464: 
1465:   // Variadic functions do not need all the analysis below.
1466:   if (isVarArg) {
1467:     CCInfo.AnalyzeCallOperands(Outs, ArgCC_AVR_Vararg);
1468:   } else {
1469:     analyzeArguments(&CLI, F, &DAG.getDataLayout(), Outs, ArgLocs, CCInfo,
1470:                      Subtarget.hasTinyEncoding());
1471:   }
1472: 
1473:   // Get a count of how many bytes are to be pushed on the stack.
1474:   unsigned NumBytes = CCInfo.getStackSize();
1475: 
1476:   Chain = DAG.getCALLSEQ_START(Chain, NumBytes, 0, DL);
1477: 
1478:   SmallVector<std::pair<unsigned, SDValue>, 8> RegsToPass;
1479: 
1480:   // First, walk the register assignments, inserting copies.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1481-1500

```cpp
1481:   unsigned AI, AE;
1482:   bool HasStackArgs = false;
1483:   for (AI = 0, AE = ArgLocs.size(); AI != AE; ++AI) {
1484:     CCValAssign &VA = ArgLocs[AI];
1485:     EVT RegVT = VA.getLocVT();
1486:     SDValue Arg = OutVals[AI];
1487: 
1488:     // Promote the value if needed. With Clang this should not happen.
1489:     switch (VA.getLocInfo()) {
1490:     default:
1491:       llvm_unreachable("Unknown loc info!");
1492:     case CCValAssign::Full:
1493:       break;
1494:     case CCValAssign::SExt:
1495:       Arg = DAG.getNode(ISD::SIGN_EXTEND, DL, RegVT, Arg);
1496:       break;
1497:     case CCValAssign::ZExt:
1498:       Arg = DAG.getNode(ISD::ZERO_EXTEND, DL, RegVT, Arg);
1499:       break;
1500:     case CCValAssign::AExt:
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1501-1520

```cpp
1501:       Arg = DAG.getNode(ISD::ANY_EXTEND, DL, RegVT, Arg);
1502:       break;
1503:     case CCValAssign::BCvt:
1504:       Arg = DAG.getNode(ISD::BITCAST, DL, RegVT, Arg);
1505:       break;
1506:     }
1507: 
1508:     // Stop when we encounter a stack argument, we need to process them
1509:     // in reverse order in the loop below.
1510:     if (VA.isMemLoc()) {
1511:       HasStackArgs = true;
1512:       break;
1513:     }
1514: 
1515:     // Arguments that can be passed on registers must be kept in the RegsToPass
1516:     // vector.
1517:     RegsToPass.push_back(std::make_pair(VA.getLocReg(), Arg));
1518:   }
1519: 
1520:   // Second, stack arguments have to walked.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1521-1540

```cpp
1521:   // Previously this code created chained stores but those chained stores appear
1522:   // to be unchained in the legalization phase. Therefore, do not attempt to
1523:   // chain them here. In fact, chaining them here somehow causes the first and
1524:   // second store to be reversed which is the exact opposite of the intended
1525:   // effect.
1526:   if (HasStackArgs) {
1527:     SmallVector<SDValue, 8> MemOpChains;
1528:     for (; AI != AE; AI++) {
1529:       CCValAssign &VA = ArgLocs[AI];
1530:       SDValue Arg = OutVals[AI];
1531: 
1532:       assert(VA.isMemLoc());
1533: 
1534:       // SP points to one stack slot further so add one to adjust it.
1535:       SDValue PtrOff = DAG.getNode(
1536:           ISD::ADD, DL, getPointerTy(DAG.getDataLayout()),
1537:           DAG.getRegister(AVR::SP, getPointerTy(DAG.getDataLayout())),
1538:           DAG.getIntPtrConstant(VA.getLocMemOffset() + 1, DL));
1539: 
1540:       MemOpChains.push_back(
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1541-1560

```cpp
1541:           DAG.getStore(Chain, DL, Arg, PtrOff,
1542:                        MachinePointerInfo::getStack(MF, VA.getLocMemOffset())));
1543:     }
1544: 
1545:     if (!MemOpChains.empty())
1546:       Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, MemOpChains);
1547:   }
1548: 
1549:   // Build a sequence of copy-to-reg nodes chained together with token chain and
1550:   // flag operands which copy the outgoing args into registers.  The InGlue in
1551:   // necessary since all emited instructions must be stuck together.
1552:   SDValue InGlue;
1553:   for (auto Reg : RegsToPass) {
1554:     Chain = DAG.getCopyToReg(Chain, DL, Reg.first, Reg.second, InGlue);
1555:     InGlue = Chain.getValue(1);
1556:   }
1557: 
1558:   // Returns a chain & a flag for retval copy to use.
1559:   SDVTList NodeTys = DAG.getVTList(MVT::Other, MVT::Glue);
1560:   SmallVector<SDValue, 8> Ops;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1561-1580

```cpp
1561:   Ops.push_back(Chain);
1562:   Ops.push_back(Callee);
1563: 
1564:   // Add argument registers to the end of the list so that they are known live
1565:   // into the call.
1566:   for (auto Reg : RegsToPass) {
1567:     Ops.push_back(DAG.getRegister(Reg.first, Reg.second.getValueType()));
1568:   }
1569: 
1570:   // The zero register (usually R1) must be passed as an implicit register so
1571:   // that this register is correctly zeroed in interrupts.
1572:   Ops.push_back(DAG.getRegister(Subtarget.getZeroRegister(), MVT::i8));
1573: 
1574:   // Add a register mask operand representing the call-preserved registers.
1575:   const TargetRegisterInfo *TRI = Subtarget.getRegisterInfo();
1576:   const uint32_t *Mask =
1577:       TRI->getCallPreservedMask(DAG.getMachineFunction(), CallConv);
1578:   assert(Mask && "Missing call preserved mask for calling convention");
1579:   Ops.push_back(DAG.getRegisterMask(Mask));
1580: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1581-1600

```cpp
1581:   if (InGlue.getNode()) {
1582:     Ops.push_back(InGlue);
1583:   }
1584: 
1585:   Chain = DAG.getNode(AVRISD::CALL, DL, NodeTys, Ops);
1586:   InGlue = Chain.getValue(1);
1587: 
1588:   // Create the CALLSEQ_END node.
1589:   Chain = DAG.getCALLSEQ_END(Chain, NumBytes, 0, InGlue, DL);
1590: 
1591:   if (!Ins.empty()) {
1592:     InGlue = Chain.getValue(1);
1593:   }
1594: 
1595:   // Handle result values, copying them out of physregs into vregs that we
1596:   // return.
1597:   return LowerCallResult(Chain, InGlue, CallConv, isVarArg, Ins, DL, DAG,
1598:                          InVals);
1599: }
1600: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1601-1620

```cpp
1601: /// Lower the result values of a call into the
1602: /// appropriate copies out of appropriate physical registers.
1603: ///
1604: SDValue AVRTargetLowering::LowerCallResult(
1605:     SDValue Chain, SDValue InGlue, CallingConv::ID CallConv, bool isVarArg,
1606:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &dl,
1607:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
1608: 
1609:   // Assign locations to each value returned by this call.
1610:   SmallVector<CCValAssign, 16> RVLocs;
1611:   CCState CCInfo(CallConv, isVarArg, DAG.getMachineFunction(), RVLocs,
1612:                  *DAG.getContext());
1613: 
1614:   // Handle runtime calling convs.
1615:   if (CallConv == CallingConv::AVR_BUILTIN) {
1616:     CCInfo.AnalyzeCallResult(Ins, RetCC_AVR_BUILTIN);
1617:   } else {
1618:     analyzeReturnValues(Ins, CCInfo, Subtarget.hasTinyEncoding());
1619:   }
1620: 
```

- EN: Function bodies or method definitions such as LowerCallResult contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerCallResult 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1621-1640

```cpp
1621:   // Copy all of the result registers out of their specified physreg.
1622:   for (CCValAssign const &RVLoc : RVLocs) {
1623:     Chain = DAG.getCopyFromReg(Chain, dl, RVLoc.getLocReg(), RVLoc.getValVT(),
1624:                                InGlue)
1625:                 .getValue(1);
1626:     InGlue = Chain.getValue(2);
1627:     InVals.push_back(Chain.getValue(0));
1628:   }
1629: 
1630:   return Chain;
1631: }
1632: 
1633: //===----------------------------------------------------------------------===//
1634: //               Return Value Calling Convention Implementation
1635: //===----------------------------------------------------------------------===//
1636: 
1637: bool AVRTargetLowering::CanLowerReturn(
1638:     CallingConv::ID CallConv, MachineFunction &MF, bool isVarArg,
1639:     const SmallVectorImpl<ISD::OutputArg> &Outs, LLVMContext &Context,
1640:     const Type *RetTy) const {
```

- EN: Function bodies or method definitions such as CanLowerReturn contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: CanLowerReturn 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1641-1660

```cpp
1641:   if (CallConv == CallingConv::AVR_BUILTIN) {
1642:     SmallVector<CCValAssign, 16> RVLocs;
1643:     CCState CCInfo(CallConv, isVarArg, MF, RVLocs, Context);
1644:     return CCInfo.CheckReturn(Outs, RetCC_AVR_BUILTIN);
1645:   }
1646: 
1647:   unsigned TotalBytes = getTotalArgumentsSizeInBytes(Outs);
1648:   return TotalBytes <= (unsigned)(Subtarget.hasTinyEncoding() ? 4 : 8);
1649: }
1650: 
1651: SDValue
1652: AVRTargetLowering::LowerReturn(SDValue Chain, CallingConv::ID CallConv,
1653:                                bool isVarArg,
1654:                                const SmallVectorImpl<ISD::OutputArg> &Outs,
1655:                                const SmallVectorImpl<SDValue> &OutVals,
1656:                                const SDLoc &dl, SelectionDAG &DAG) const {
1657:   // CCValAssign - represent the assignment of the return value to locations.
1658:   SmallVector<CCValAssign, 16> RVLocs;
1659: 
1660:   // CCState - Info about the registers and stack slot.
```

- EN: Function bodies or method definitions such as LowerReturn contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerReturn 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1661-1680

```cpp
1661:   CCState CCInfo(CallConv, isVarArg, DAG.getMachineFunction(), RVLocs,
1662:                  *DAG.getContext());
1663: 
1664:   MachineFunction &MF = DAG.getMachineFunction();
1665: 
1666:   // Analyze return values.
1667:   if (CallConv == CallingConv::AVR_BUILTIN) {
1668:     CCInfo.AnalyzeReturn(Outs, RetCC_AVR_BUILTIN);
1669:   } else {
1670:     analyzeReturnValues(Outs, CCInfo, Subtarget.hasTinyEncoding());
1671:   }
1672: 
1673:   SDValue Glue;
1674:   SmallVector<SDValue, 4> RetOps(1, Chain);
1675:   // Copy the result values into the output registers.
1676:   for (unsigned i = 0, e = RVLocs.size(); i != e; ++i) {
1677:     CCValAssign &VA = RVLocs[i];
1678:     assert(VA.isRegLoc() && "Can only return in registers!");
1679: 
1680:     Chain = DAG.getCopyToReg(Chain, dl, VA.getLocReg(), OutVals[i], Glue);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1681-1700

```cpp
1681: 
1682:     // Guarantee that all emitted copies are stuck together with flags.
1683:     Glue = Chain.getValue(1);
1684:     RetOps.push_back(DAG.getRegister(VA.getLocReg(), VA.getLocVT()));
1685:   }
1686: 
1687:   // Don't emit the ret/reti instruction when the naked attribute is present in
1688:   // the function being compiled.
1689:   if (MF.getFunction().getAttributes().hasFnAttr(Attribute::Naked)) {
1690:     return Chain;
1691:   }
1692: 
1693:   const AVRMachineFunctionInfo *AFI = MF.getInfo<AVRMachineFunctionInfo>();
1694: 
1695:   if (!AFI->isInterruptOrSignalHandler()) {
1696:     // The return instruction has an implicit zero register operand: it must
1697:     // contain zero on return.
1698:     // This is not needed in interrupts however, where the zero register is
1699:     // handled specially (only pushed/popped when needed).
1700:     RetOps.push_back(DAG.getRegister(Subtarget.getZeroRegister(), MVT::i8));
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1701-1720

```cpp
1701:   }
1702: 
1703:   unsigned RetOpc =
1704:       AFI->isInterruptOrSignalHandler() ? AVRISD::RETI_GLUE : AVRISD::RET_GLUE;
1705: 
1706:   RetOps[0] = Chain; // Update chain.
1707: 
1708:   if (Glue.getNode()) {
1709:     RetOps.push_back(Glue);
1710:   }
1711: 
1712:   return DAG.getNode(RetOpc, dl, MVT::Other, RetOps);
1713: }
1714: 
1715: //===----------------------------------------------------------------------===//
1716: //  Custom Inserters
1717: //===----------------------------------------------------------------------===//
1718: 
1719: MachineBasicBlock *AVRTargetLowering::insertShift(MachineInstr &MI,
1720:                                                   MachineBasicBlock *BB,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1721-1740

```cpp
1721:                                                   bool Tiny) const {
1722:   unsigned Opc;
1723:   const TargetRegisterClass *RC;
1724:   bool HasRepeatedOperand = false;
1725:   MachineFunction *F = BB->getParent();
1726:   MachineRegisterInfo &RI = F->getRegInfo();
1727:   const TargetInstrInfo &TII = *Subtarget.getInstrInfo();
1728:   DebugLoc dl = MI.getDebugLoc();
1729: 
1730:   switch (MI.getOpcode()) {
1731:   default:
1732:     llvm_unreachable("Invalid shift opcode!");
1733:   case AVR::Lsl8:
1734:     Opc = AVR::ADDRdRr; // LSL is an alias of ADD Rd, Rd
1735:     RC = &AVR::GPR8RegClass;
1736:     HasRepeatedOperand = true;
1737:     break;
1738:   case AVR::Lsl16:
1739:     Opc = AVR::LSLWRd;
1740:     RC = &AVR::DREGSRegClass;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1741-1760

```cpp
1741:     break;
1742:   case AVR::Asr8:
1743:     Opc = AVR::ASRRd;
1744:     RC = &AVR::GPR8RegClass;
1745:     break;
1746:   case AVR::Asr16:
1747:     Opc = AVR::ASRWRd;
1748:     RC = &AVR::DREGSRegClass;
1749:     break;
1750:   case AVR::Lsr8:
1751:     Opc = AVR::LSRRd;
1752:     RC = &AVR::GPR8RegClass;
1753:     break;
1754:   case AVR::Lsr16:
1755:     Opc = AVR::LSRWRd;
1756:     RC = &AVR::DREGSRegClass;
1757:     break;
1758:   case AVR::Rol8:
1759:     Opc = Tiny ? AVR::ROLBRdR17 : AVR::ROLBRdR1;
1760:     RC = &AVR::GPR8RegClass;
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1761-1780

```cpp
1761:     break;
1762:   case AVR::Rol16:
1763:     Opc = AVR::ROLWRd;
1764:     RC = &AVR::DREGSRegClass;
1765:     break;
1766:   case AVR::Ror8:
1767:     Opc = AVR::RORBRd;
1768:     RC = &AVR::GPR8RegClass;
1769:     break;
1770:   case AVR::Ror16:
1771:     Opc = AVR::RORWRd;
1772:     RC = &AVR::DREGSRegClass;
1773:     break;
1774:   }
1775: 
1776:   const BasicBlock *LLVM_BB = BB->getBasicBlock();
1777: 
1778:   MachineFunction::iterator I;
1779:   for (I = BB->getIterator(); I != F->end() && &(*I) != BB; ++I)
1780:     ;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1781-1800

```cpp
1781:   if (I != F->end())
1782:     ++I;
1783: 
1784:   // Create loop block.
1785:   MachineBasicBlock *LoopBB = F->CreateMachineBasicBlock(LLVM_BB);
1786:   MachineBasicBlock *CheckBB = F->CreateMachineBasicBlock(LLVM_BB);
1787:   MachineBasicBlock *RemBB = F->CreateMachineBasicBlock(LLVM_BB);
1788: 
1789:   F->insert(I, LoopBB);
1790:   F->insert(I, CheckBB);
1791:   F->insert(I, RemBB);
1792: 
1793:   // Update machine-CFG edges by transferring all successors of the current
1794:   // block to the block containing instructions after shift.
1795:   RemBB->splice(RemBB->begin(), BB, std::next(MachineBasicBlock::iterator(MI)),
1796:                 BB->end());
1797:   RemBB->transferSuccessorsAndUpdatePHIs(BB);
1798: 
1799:   // Add edges BB => LoopBB => CheckBB => RemBB, CheckBB => LoopBB.
1800:   BB->addSuccessor(CheckBB);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1801-1820

```cpp
1801:   LoopBB->addSuccessor(CheckBB);
1802:   CheckBB->addSuccessor(LoopBB);
1803:   CheckBB->addSuccessor(RemBB);
1804: 
1805:   Register ShiftAmtReg = RI.createVirtualRegister(&AVR::GPR8RegClass);
1806:   Register ShiftAmtReg2 = RI.createVirtualRegister(&AVR::GPR8RegClass);
1807:   Register ShiftReg = RI.createVirtualRegister(RC);
1808:   Register ShiftReg2 = RI.createVirtualRegister(RC);
1809:   Register ShiftAmtSrcReg = MI.getOperand(2).getReg();
1810:   Register SrcReg = MI.getOperand(1).getReg();
1811:   Register DstReg = MI.getOperand(0).getReg();
1812: 
1813:   // BB:
1814:   // rjmp CheckBB
1815:   BuildMI(BB, dl, TII.get(AVR::RJMPk)).addMBB(CheckBB);
1816: 
1817:   // LoopBB:
1818:   // ShiftReg2 = shift ShiftReg
1819:   auto ShiftMI = BuildMI(LoopBB, dl, TII.get(Opc), ShiftReg2).addReg(ShiftReg);
1820:   if (HasRepeatedOperand)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1821-1840

```cpp
1821:     ShiftMI.addReg(ShiftReg);
1822: 
1823:   // CheckBB:
1824:   // ShiftReg = phi [%SrcReg, BB], [%ShiftReg2, LoopBB]
1825:   // ShiftAmt = phi [%N,      BB], [%ShiftAmt2, LoopBB]
1826:   // DestReg  = phi [%SrcReg, BB], [%ShiftReg,  LoopBB]
1827:   // ShiftAmt2 = ShiftAmt - 1;
1828:   // if (ShiftAmt2 >= 0) goto LoopBB;
1829:   BuildMI(CheckBB, dl, TII.get(AVR::PHI), ShiftReg)
1830:       .addReg(SrcReg)
1831:       .addMBB(BB)
1832:       .addReg(ShiftReg2)
1833:       .addMBB(LoopBB);
1834:   BuildMI(CheckBB, dl, TII.get(AVR::PHI), ShiftAmtReg)
1835:       .addReg(ShiftAmtSrcReg)
1836:       .addMBB(BB)
1837:       .addReg(ShiftAmtReg2)
1838:       .addMBB(LoopBB);
1839:   BuildMI(CheckBB, dl, TII.get(AVR::PHI), DstReg)
1840:       .addReg(SrcReg)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1841-1860

```cpp
1841:       .addMBB(BB)
1842:       .addReg(ShiftReg2)
1843:       .addMBB(LoopBB);
1844: 
1845:   BuildMI(CheckBB, dl, TII.get(AVR::DECRd), ShiftAmtReg2).addReg(ShiftAmtReg);
1846:   BuildMI(CheckBB, dl, TII.get(AVR::BRPLk)).addMBB(LoopBB);
1847: 
1848:   MI.eraseFromParent(); // The pseudo instruction is gone now.
1849:   return RemBB;
1850: }
1851: 
1852: // Do a multibyte AVR shift. Insert shift instructions and put the output
1853: // registers in the Regs array.
1854: // Because AVR does not have a normal shift instruction (only a single bit shift
1855: // instruction), we have to emulate this behavior with other instructions.
1856: // It first tries large steps (moving registers around) and then smaller steps
1857: // like single bit shifts.
1858: // Large shifts actually reduce the number of shifted registers, so the below
1859: // algorithms have to work independently of the number of registers that are
1860: // shifted.
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1861-1880

```cpp
1861: // For more information and background, see this blogpost:
1862: // https://aykevl.nl/2021/02/avr-bitshift
1863: static void insertMultibyteShift(MachineInstr &MI, MachineBasicBlock *BB,
1864:                                  MutableArrayRef<std::pair<Register, int>> Regs,
1865:                                  ISD::NodeType Opc, int64_t ShiftAmt) {
1866:   const TargetInstrInfo &TII = *BB->getParent()->getSubtarget().getInstrInfo();
1867:   const AVRSubtarget &STI = BB->getParent()->getSubtarget<AVRSubtarget>();
1868:   MachineRegisterInfo &MRI = BB->getParent()->getRegInfo();
1869:   const DebugLoc &dl = MI.getDebugLoc();
1870: 
1871:   const bool ShiftLeft = Opc == ISD::SHL;
1872:   const bool ArithmeticShift = Opc == ISD::SRA;
1873: 
1874:   // Zero a register, for use in later operations.
1875:   Register ZeroReg = MRI.createVirtualRegister(&AVR::GPR8RegClass);
1876:   BuildMI(*BB, MI, dl, TII.get(AVR::COPY), ZeroReg)
1877:       .addReg(STI.getZeroRegister());
1878: 
1879:   // Do a shift modulo 6 or 7. This is a bit more complicated than most shifts
1880:   // and is hard to compose with the rest, so these are special cased.
```

- EN: Function bodies or method definitions such as insertMultibyteShift contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: insertMultibyteShift 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1881-1900

```cpp
1881:   // The basic idea is to shift one or two bits in the opposite direction and
1882:   // then move registers around to get the correct end result.
1883:   if (ShiftLeft && (ShiftAmt % 8) >= 6) {
1884:     // Left shift modulo 6 or 7.
1885: 
1886:     // Create a slice of the registers we're going to modify, to ease working
1887:     // with them.
1888:     size_t ShiftRegsOffset = ShiftAmt / 8;
1889:     size_t ShiftRegsSize = Regs.size() - ShiftRegsOffset;
1890:     MutableArrayRef<std::pair<Register, int>> ShiftRegs =
1891:         Regs.slice(ShiftRegsOffset, ShiftRegsSize);
1892: 
1893:     // Shift one to the right, keeping the least significant bit as the carry
1894:     // bit.
1895:     insertMultibyteShift(MI, BB, ShiftRegs, ISD::SRL, 1);
1896: 
1897:     // Rotate the least significant bit from the carry bit into a new register
1898:     // (that starts out zero).
1899:     Register LowByte = MRI.createVirtualRegister(&AVR::GPR8RegClass);
1900:     BuildMI(*BB, MI, dl, TII.get(AVR::RORRd), LowByte).addReg(ZeroReg);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1901-1920

```cpp
1901: 
1902:     // Shift one more to the right if this is a modulo-6 shift.
1903:     if (ShiftAmt % 8 == 6) {
1904:       insertMultibyteShift(MI, BB, ShiftRegs, ISD::SRL, 1);
1905:       Register NewLowByte = MRI.createVirtualRegister(&AVR::GPR8RegClass);
1906:       BuildMI(*BB, MI, dl, TII.get(AVR::RORRd), NewLowByte).addReg(LowByte);
1907:       LowByte = NewLowByte;
1908:     }
1909: 
1910:     // Move all registers to the left, zeroing the bottom registers as needed.
1911:     for (size_t I = 0; I < Regs.size(); I++) {
1912:       int ShiftRegsIdx = I + 1;
1913:       if (ShiftRegsIdx < (int)ShiftRegs.size()) {
1914:         Regs[I] = ShiftRegs[ShiftRegsIdx];
1915:       } else if (ShiftRegsIdx == (int)ShiftRegs.size()) {
1916:         Regs[I] = std::pair(LowByte, 0);
1917:       } else {
1918:         Regs[I] = std::pair(ZeroReg, 0);
1919:       }
1920:     }
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1921-1940

```cpp
1921: 
1922:     return;
1923:   }
1924: 
1925:   // Right shift modulo 6 or 7.
1926:   if (!ShiftLeft && (ShiftAmt % 8) >= 6) {
1927:     // Create a view on the registers we're going to modify, to ease working
1928:     // with them.
1929:     size_t ShiftRegsSize = Regs.size() - (ShiftAmt / 8);
1930:     MutableArrayRef<std::pair<Register, int>> ShiftRegs =
1931:         Regs.slice(0, ShiftRegsSize);
1932: 
1933:     // Shift one to the left.
1934:     insertMultibyteShift(MI, BB, ShiftRegs, ISD::SHL, 1);
1935: 
1936:     // Sign or zero extend the most significant register into a new register.
1937:     // The HighByte is the byte that still has one (or two) bits from the
1938:     // original value. The ExtByte is purely a zero/sign extend byte (all bits
1939:     // are either 0 or 1).
1940:     Register HighByte = MRI.createVirtualRegister(&AVR::GPR8RegClass);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1941-1960

```cpp
1941:     Register ExtByte = 0;
1942:     if (ArithmeticShift) {
1943:       // Sign-extend bit that was shifted out last.
1944:       BuildMI(*BB, MI, dl, TII.get(AVR::SBCRdRr), HighByte)
1945:           .addReg(HighByte, RegState::Undef)
1946:           .addReg(HighByte, RegState::Undef);
1947:       ExtByte = HighByte;
1948:       // The highest bit of the original value is the same as the zero-extend
1949:       // byte, so HighByte and ExtByte are the same.
1950:     } else {
1951:       // Use the zero register for zero extending.
1952:       ExtByte = ZeroReg;
1953:       // Rotate most significant bit into a new register (that starts out zero).
1954:       BuildMI(*BB, MI, dl, TII.get(AVR::ADCRdRr), HighByte)
1955:           .addReg(ExtByte)
1956:           .addReg(ExtByte);
1957:     }
1958: 
1959:     // Shift one more to the left for modulo 6 shifts.
1960:     if (ShiftAmt % 8 == 6) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1961-1980

```cpp
1961:       insertMultibyteShift(MI, BB, ShiftRegs, ISD::SHL, 1);
1962:       // Shift the topmost bit into the HighByte.
1963:       Register NewExt = MRI.createVirtualRegister(&AVR::GPR8RegClass);
1964:       BuildMI(*BB, MI, dl, TII.get(AVR::ADCRdRr), NewExt)
1965:           .addReg(HighByte)
1966:           .addReg(HighByte);
1967:       HighByte = NewExt;
1968:     }
1969: 
1970:     // Move all to the right, while sign or zero extending.
1971:     for (int I = Regs.size() - 1; I >= 0; I--) {
1972:       int ShiftRegsIdx = I - (Regs.size() - ShiftRegs.size()) - 1;
1973:       if (ShiftRegsIdx >= 0) {
1974:         Regs[I] = ShiftRegs[ShiftRegsIdx];
1975:       } else if (ShiftRegsIdx == -1) {
1976:         Regs[I] = std::pair(HighByte, 0);
1977:       } else {
1978:         Regs[I] = std::pair(ExtByte, 0);
1979:       }
1980:     }
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1981-2000

```cpp
1981: 
1982:     return;
1983:   }
1984: 
1985:   // For shift amounts of at least one register, simply rename the registers and
1986:   // zero the bottom registers.
1987:   while (ShiftLeft && ShiftAmt >= 8) {
1988:     // Move all registers one to the left.
1989:     for (size_t I = 0; I < Regs.size() - 1; I++) {
1990:       Regs[I] = Regs[I + 1];
1991:     }
1992: 
1993:     // Zero the least significant register.
1994:     Regs[Regs.size() - 1] = std::pair(ZeroReg, 0);
1995: 
1996:     // Continue shifts with the leftover registers.
1997:     Regs = Regs.drop_back(1);
1998: 
1999:     ShiftAmt -= 8;
2000:   }
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2001-2020

```cpp
2001: 
2002:   // And again, the same for right shifts.
2003:   Register ShrExtendReg = 0;
2004:   if (!ShiftLeft && ShiftAmt >= 8) {
2005:     if (ArithmeticShift) {
2006:       // Sign extend the most significant register into ShrExtendReg.
2007:       ShrExtendReg = MRI.createVirtualRegister(&AVR::GPR8RegClass);
2008:       Register Tmp = MRI.createVirtualRegister(&AVR::GPR8RegClass);
2009:       BuildMI(*BB, MI, dl, TII.get(AVR::ADDRdRr), Tmp)
2010:           .addReg(Regs[0].first, {}, Regs[0].second)
2011:           .addReg(Regs[0].first, {}, Regs[0].second);
2012:       BuildMI(*BB, MI, dl, TII.get(AVR::SBCRdRr), ShrExtendReg)
2013:           .addReg(Tmp)
2014:           .addReg(Tmp);
2015:     } else {
2016:       ShrExtendReg = ZeroReg;
2017:     }
2018:     for (; ShiftAmt >= 8; ShiftAmt -= 8) {
2019:       // Move all registers one to the right.
2020:       for (size_t I = Regs.size() - 1; I != 0; I--) {
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2021-2040

```cpp
2021:         Regs[I] = Regs[I - 1];
2022:       }
2023: 
2024:       // Zero or sign extend the most significant register.
2025:       Regs[0] = std::pair(ShrExtendReg, 0);
2026: 
2027:       // Continue shifts with the leftover registers.
2028:       Regs = Regs.drop_front(1);
2029:     }
2030:   }
2031: 
2032:   // The bigger shifts are already handled above.
2033:   assert((ShiftAmt < 8) && "Unexpect shift amount");
2034: 
2035:   // Shift by four bits, using a complicated swap/eor/andi/eor sequence.
2036:   // It only works for logical shifts because the bits shifted in are all
2037:   // zeroes.
2038:   // To shift a single byte right, it produces code like this:
2039:   //   swap r0
2040:   //   andi r0, 0x0f
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2041-2060

```cpp
2041:   // For a two-byte (16-bit) shift, it adds the following instructions to shift
2042:   // the upper byte into the lower byte:
2043:   //   swap r1
2044:   //   eor r0, r1
2045:   //   andi r1, 0x0f
2046:   //   eor r0, r1
2047:   // For bigger shifts, it repeats the above sequence. For example, for a 3-byte
2048:   // (24-bit) shift it adds:
2049:   //   swap r2
2050:   //   eor r1, r2
2051:   //   andi r2, 0x0f
2052:   //   eor r1, r2
2053:   if (!ArithmeticShift && ShiftAmt >= 4) {
2054:     Register Prev = 0;
2055:     for (size_t I = 0; I < Regs.size(); I++) {
2056:       size_t Idx = ShiftLeft ? I : Regs.size() - I - 1;
2057:       Register SwapReg = MRI.createVirtualRegister(&AVR::LD8RegClass);
2058:       BuildMI(*BB, MI, dl, TII.get(AVR::SWAPRd), SwapReg)
2059:           .addReg(Regs[Idx].first, {}, Regs[Idx].second);
2060:       if (I != 0) {
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2061-2080

```cpp
2061:         Register R = MRI.createVirtualRegister(&AVR::GPR8RegClass);
2062:         BuildMI(*BB, MI, dl, TII.get(AVR::EORRdRr), R)
2063:             .addReg(Prev)
2064:             .addReg(SwapReg);
2065:         Prev = R;
2066:       }
2067:       Register AndReg = MRI.createVirtualRegister(&AVR::LD8RegClass);
2068:       BuildMI(*BB, MI, dl, TII.get(AVR::ANDIRdK), AndReg)
2069:           .addReg(SwapReg)
2070:           .addImm(ShiftLeft ? 0xf0 : 0x0f);
2071:       if (I != 0) {
2072:         Register R = MRI.createVirtualRegister(&AVR::GPR8RegClass);
2073:         BuildMI(*BB, MI, dl, TII.get(AVR::EORRdRr), R)
2074:             .addReg(Prev)
2075:             .addReg(AndReg);
2076:         size_t PrevIdx = ShiftLeft ? Idx - 1 : Idx + 1;
2077:         Regs[PrevIdx] = std::pair(R, 0);
2078:       }
2079:       Prev = AndReg;
2080:       Regs[Idx] = std::pair(AndReg, 0);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2081-2100

```cpp
2081:     }
2082:     ShiftAmt -= 4;
2083:   }
2084: 
2085:   // Shift by one. This is the fallback that always works, and the shift
2086:   // operation that is used for 1, 2, and 3 bit shifts.
2087:   while (ShiftLeft && ShiftAmt) {
2088:     // Shift one to the left.
2089:     for (ssize_t I = Regs.size() - 1; I >= 0; I--) {
2090:       Register Out = MRI.createVirtualRegister(&AVR::GPR8RegClass);
2091:       Register In = Regs[I].first;
2092:       Register InSubreg = Regs[I].second;
2093:       if (I == (ssize_t)Regs.size() - 1) { // first iteration
2094:         BuildMI(*BB, MI, dl, TII.get(AVR::ADDRdRr), Out)
2095:             .addReg(In, {}, InSubreg)
2096:             .addReg(In, {}, InSubreg);
2097:       } else {
2098:         BuildMI(*BB, MI, dl, TII.get(AVR::ADCRdRr), Out)
2099:             .addReg(In, {}, InSubreg)
2100:             .addReg(In, {}, InSubreg);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2101-2120

```cpp
2101:       }
2102:       Regs[I] = std::pair(Out, 0);
2103:     }
2104:     ShiftAmt--;
2105:   }
2106:   while (!ShiftLeft && ShiftAmt) {
2107:     // Shift one to the right.
2108:     for (size_t I = 0; I < Regs.size(); I++) {
2109:       Register Out = MRI.createVirtualRegister(&AVR::GPR8RegClass);
2110:       Register In = Regs[I].first;
2111:       Register InSubreg = Regs[I].second;
2112:       if (I == 0) {
2113:         unsigned Opc = ArithmeticShift ? AVR::ASRRd : AVR::LSRRd;
2114:         BuildMI(*BB, MI, dl, TII.get(Opc), Out).addReg(In, {}, InSubreg);
2115:       } else {
2116:         BuildMI(*BB, MI, dl, TII.get(AVR::RORRd), Out).addReg(In, {}, InSubreg);
2117:       }
2118:       Regs[I] = std::pair(Out, 0);
2119:     }
2120:     ShiftAmt--;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2121-2140

```cpp
2121:   }
2122: 
2123:   if (ShiftAmt != 0) {
2124:     llvm_unreachable("don't know how to shift!"); // sanity check
2125:   }
2126: }
2127: 
2128: // Do a wide (32-bit) shift.
2129: MachineBasicBlock *
2130: AVRTargetLowering::insertWideShift(MachineInstr &MI,
2131:                                    MachineBasicBlock *BB) const {
2132:   const TargetInstrInfo &TII = *Subtarget.getInstrInfo();
2133:   const DebugLoc &dl = MI.getDebugLoc();
2134: 
2135:   // How much to shift to the right (meaning: a negative number indicates a left
2136:   // shift).
2137:   int64_t ShiftAmt = MI.getOperand(4).getImm();
2138:   ISD::NodeType Opc;
2139:   switch (MI.getOpcode()) {
2140:   case AVR::Lsl32:
```

- EN: Function bodies or method definitions such as insertWideShift contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: insertWideShift 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2141-2160

```cpp
2141:     Opc = ISD::SHL;
2142:     break;
2143:   case AVR::Lsr32:
2144:     Opc = ISD::SRL;
2145:     break;
2146:   case AVR::Asr32:
2147:     Opc = ISD::SRA;
2148:     break;
2149:   }
2150: 
2151:   // Read the input registers, with the most significant register at index 0.
2152:   std::array<std::pair<Register, int>, 4> Registers = {
2153:       std::pair(MI.getOperand(3).getReg(), AVR::sub_hi),
2154:       std::pair(MI.getOperand(3).getReg(), AVR::sub_lo),
2155:       std::pair(MI.getOperand(2).getReg(), AVR::sub_hi),
2156:       std::pair(MI.getOperand(2).getReg(), AVR::sub_lo),
2157:   };
2158: 
2159:   // Do the shift. The registers are modified in-place.
2160:   insertMultibyteShift(MI, BB, Registers, Opc, ShiftAmt);
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2161-2180

```cpp
2161: 
2162:   // Combine the 8-bit registers into 16-bit register pairs.
2163:   // This done either from LSB to MSB or from MSB to LSB, depending on the
2164:   // shift. It's an optimization so that the register allocator will use the
2165:   // fewest movs possible (which order we use isn't a correctness issue, just an
2166:   // optimization issue).
2167:   //   - lsl prefers starting from the most significant byte (2nd case).
2168:   //   - lshr prefers starting from the least significant byte (1st case).
2169:   //   - for ashr it depends on the number of shifted bytes.
2170:   // Some shift operations still don't get the most optimal mov sequences even
2171:   // with this distinction. TODO: figure out why and try to fix it (but we're
2172:   // already equal to or faster than avr-gcc in all cases except ashr 8).
2173:   if (Opc != ISD::SHL &&
2174:       (Opc != ISD::SRA || (ShiftAmt < 16 || ShiftAmt >= 22))) {
2175:     // Use the resulting registers starting with the least significant byte.
2176:     BuildMI(*BB, MI, dl, TII.get(AVR::REG_SEQUENCE), MI.getOperand(0).getReg())
2177:         .addReg(Registers[3].first, {}, Registers[3].second)
2178:         .addImm(AVR::sub_lo)
2179:         .addReg(Registers[2].first, {}, Registers[2].second)
2180:         .addImm(AVR::sub_hi);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2181-2200

```cpp
2181:     BuildMI(*BB, MI, dl, TII.get(AVR::REG_SEQUENCE), MI.getOperand(1).getReg())
2182:         .addReg(Registers[1].first, {}, Registers[1].second)
2183:         .addImm(AVR::sub_lo)
2184:         .addReg(Registers[0].first, {}, Registers[0].second)
2185:         .addImm(AVR::sub_hi);
2186:   } else {
2187:     // Use the resulting registers starting with the most significant byte.
2188:     BuildMI(*BB, MI, dl, TII.get(AVR::REG_SEQUENCE), MI.getOperand(1).getReg())
2189:         .addReg(Registers[0].first, {}, Registers[0].second)
2190:         .addImm(AVR::sub_hi)
2191:         .addReg(Registers[1].first, {}, Registers[1].second)
2192:         .addImm(AVR::sub_lo);
2193:     BuildMI(*BB, MI, dl, TII.get(AVR::REG_SEQUENCE), MI.getOperand(0).getReg())
2194:         .addReg(Registers[2].first, {}, Registers[2].second)
2195:         .addImm(AVR::sub_hi)
2196:         .addReg(Registers[3].first, {}, Registers[3].second)
2197:         .addImm(AVR::sub_lo);
2198:   }
2199: 
2200:   // Remove the pseudo instruction.
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2201-2220

```cpp
2201:   MI.eraseFromParent();
2202:   return BB;
2203: }
2204: 
2205: static bool isCopyMulResult(MachineBasicBlock::iterator const &I) {
2206:   if (I->getOpcode() == AVR::COPY) {
2207:     Register SrcReg = I->getOperand(1).getReg();
2208:     return (SrcReg == AVR::R0 || SrcReg == AVR::R1);
2209:   }
2210: 
2211:   return false;
2212: }
2213: 
2214: // The mul instructions wreak havock on our zero_reg R1. We need to clear it
2215: // after the result has been evacuated. This is probably not the best way to do
2216: // it, but it works for now.
2217: MachineBasicBlock *AVRTargetLowering::insertMul(MachineInstr &MI,
2218:                                                 MachineBasicBlock *BB) const {
2219:   const TargetInstrInfo &TII = *Subtarget.getInstrInfo();
2220:   MachineBasicBlock::iterator I(MI);
```

- EN: Function bodies or method definitions such as isCopyMulResult contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: isCopyMulResult 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2221-2240

```cpp
2221:   ++I; // in any case insert *after* the mul instruction
2222:   if (isCopyMulResult(I))
2223:     ++I;
2224:   if (isCopyMulResult(I))
2225:     ++I;
2226:   BuildMI(*BB, I, MI.getDebugLoc(), TII.get(AVR::EORRdRr), AVR::R1)
2227:       .addReg(AVR::R1)
2228:       .addReg(AVR::R1);
2229:   return BB;
2230: }
2231: 
2232: // Insert a read from the zero register.
2233: MachineBasicBlock *
2234: AVRTargetLowering::insertCopyZero(MachineInstr &MI,
2235:                                   MachineBasicBlock *BB) const {
2236:   const TargetInstrInfo &TII = *Subtarget.getInstrInfo();
2237:   MachineBasicBlock::iterator I(MI);
2238:   BuildMI(*BB, I, MI.getDebugLoc(), TII.get(AVR::COPY))
2239:       .add(MI.getOperand(0))
2240:       .addReg(Subtarget.getZeroRegister());
```

- EN: Function bodies or method definitions such as insertCopyZero contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: insertCopyZero 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2241-2260

```cpp
2241:   MI.eraseFromParent();
2242:   return BB;
2243: }
2244: 
2245: // Lower atomicrmw operation to disable interrupts, do operation, and restore
2246: // interrupts. This works because all AVR microcontrollers are single core.
2247: MachineBasicBlock *AVRTargetLowering::insertAtomicArithmeticOp(
2248:     MachineInstr &MI, MachineBasicBlock *BB, unsigned Opcode, int Width) const {
2249:   MachineRegisterInfo &MRI = BB->getParent()->getRegInfo();
2250:   const TargetInstrInfo &TII = *Subtarget.getInstrInfo();
2251:   MachineBasicBlock::iterator I(MI);
2252:   DebugLoc dl = MI.getDebugLoc();
2253: 
2254:   // Example instruction sequence, for an atomic 8-bit add:
2255:   //   ldi r25, 5
2256:   //   in r0, SREG
2257:   //   cli
2258:   //   ld r24, X
2259:   //   add r25, r24
2260:   //   st X, r25
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2261-2280

```cpp
2261:   //   out SREG, r0
2262: 
2263:   const TargetRegisterClass *RC =
2264:       (Width == 8) ? &AVR::GPR8RegClass : &AVR::DREGSNOZRegClass;
2265:   unsigned LoadOpcode = (Width == 8) ? AVR::LDRdPtr : AVR::LDWRdPtr;
2266:   unsigned StoreOpcode = (Width == 8) ? AVR::STPtrRr : AVR::STWPtrRr;
2267: 
2268:   // Disable interrupts.
2269:   BuildMI(*BB, I, dl, TII.get(AVR::INRdA), Subtarget.getTmpRegister())
2270:       .addImm(Subtarget.getIORegSREG());
2271:   BuildMI(*BB, I, dl, TII.get(AVR::BCLRs)).addImm(7);
2272: 
2273:   // Load the original value.
2274:   BuildMI(*BB, I, dl, TII.get(LoadOpcode), MI.getOperand(0).getReg())
2275:       .add(MI.getOperand(1));
2276: 
2277:   // Do the arithmetic operation.
2278:   Register Result = MRI.createVirtualRegister(RC);
2279:   BuildMI(*BB, I, dl, TII.get(Opcode), Result)
2280:       .addReg(MI.getOperand(0).getReg())
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2281-2300

```cpp
2281:       .add(MI.getOperand(2));
2282: 
2283:   // Store the result.
2284:   BuildMI(*BB, I, dl, TII.get(StoreOpcode))
2285:       .add(MI.getOperand(1))
2286:       .addReg(Result);
2287: 
2288:   // Restore interrupts.
2289:   BuildMI(*BB, I, dl, TII.get(AVR::OUTARr))
2290:       .addImm(Subtarget.getIORegSREG())
2291:       .addReg(Subtarget.getTmpRegister());
2292: 
2293:   // Remove the pseudo instruction.
2294:   MI.eraseFromParent();
2295:   return BB;
2296: }
2297: 
2298: MachineBasicBlock *
2299: AVRTargetLowering::EmitInstrWithCustomInserter(MachineInstr &MI,
2300:                                                MachineBasicBlock *MBB) const {
```

- EN: Function bodies or method definitions such as EmitInstrWithCustomInserter contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: EmitInstrWithCustomInserter 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2301-2320

```cpp
2301:   int Opc = MI.getOpcode();
2302:   const AVRSubtarget &STI = MBB->getParent()->getSubtarget<AVRSubtarget>();
2303: 
2304:   // Pseudo shift instructions with a non constant shift amount are expanded
2305:   // into a loop.
2306:   switch (Opc) {
2307:   case AVR::Lsl8:
2308:   case AVR::Lsl16:
2309:   case AVR::Lsr8:
2310:   case AVR::Lsr16:
2311:   case AVR::Rol8:
2312:   case AVR::Rol16:
2313:   case AVR::Ror8:
2314:   case AVR::Ror16:
2315:   case AVR::Asr8:
2316:   case AVR::Asr16:
2317:     return insertShift(MI, MBB, STI.hasTinyEncoding());
2318:   case AVR::Lsl32:
2319:   case AVR::Lsr32:
2320:   case AVR::Asr32:
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2321-2340

```cpp
2321:     return insertWideShift(MI, MBB);
2322:   case AVR::MULRdRr:
2323:   case AVR::MULSRdRr:
2324:     return insertMul(MI, MBB);
2325:   case AVR::CopyZero:
2326:     return insertCopyZero(MI, MBB);
2327:   case AVR::AtomicLoadAdd8:
2328:     return insertAtomicArithmeticOp(MI, MBB, AVR::ADDRdRr, 8);
2329:   case AVR::AtomicLoadAdd16:
2330:     return insertAtomicArithmeticOp(MI, MBB, AVR::ADDWRdRr, 16);
2331:   case AVR::AtomicLoadSub8:
2332:     return insertAtomicArithmeticOp(MI, MBB, AVR::SUBRdRr, 8);
2333:   case AVR::AtomicLoadSub16:
2334:     return insertAtomicArithmeticOp(MI, MBB, AVR::SUBWRdRr, 16);
2335:   case AVR::AtomicLoadAnd8:
2336:     return insertAtomicArithmeticOp(MI, MBB, AVR::ANDRdRr, 8);
2337:   case AVR::AtomicLoadAnd16:
2338:     return insertAtomicArithmeticOp(MI, MBB, AVR::ANDWRdRr, 16);
2339:   case AVR::AtomicLoadOr8:
2340:     return insertAtomicArithmeticOp(MI, MBB, AVR::ORRdRr, 8);
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2341-2360

```cpp
2341:   case AVR::AtomicLoadOr16:
2342:     return insertAtomicArithmeticOp(MI, MBB, AVR::ORWRdRr, 16);
2343:   case AVR::AtomicLoadXor8:
2344:     return insertAtomicArithmeticOp(MI, MBB, AVR::EORRdRr, 8);
2345:   case AVR::AtomicLoadXor16:
2346:     return insertAtomicArithmeticOp(MI, MBB, AVR::EORWRdRr, 16);
2347:   }
2348: 
2349:   assert((Opc == AVR::Select16 || Opc == AVR::Select8) &&
2350:          "Unexpected instr type to insert");
2351: 
2352:   const AVRInstrInfo &TII = (const AVRInstrInfo &)*MI.getParent()
2353:                                 ->getParent()
2354:                                 ->getSubtarget()
2355:                                 .getInstrInfo();
2356:   DebugLoc dl = MI.getDebugLoc();
2357: 
2358:   // To "insert" a SELECT instruction, we insert the diamond
2359:   // control-flow pattern. The incoming instruction knows the
2360:   // destination vreg to set, the condition code register to branch
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2361-2380

```cpp
2361:   // on, the true/false values to select between, and a branch opcode
2362:   // to use.
2363: 
2364:   MachineFunction *MF = MBB->getParent();
2365:   const BasicBlock *LLVM_BB = MBB->getBasicBlock();
2366:   MachineBasicBlock *FallThrough = MBB->getFallThrough();
2367: 
2368:   // If the current basic block falls through to another basic block,
2369:   // we must insert an unconditional branch to the fallthrough destination
2370:   // if we are to insert basic blocks at the prior fallthrough point.
2371:   if (FallThrough != nullptr) {
2372:     BuildMI(MBB, dl, TII.get(AVR::RJMPk)).addMBB(FallThrough);
2373:   }
2374: 
2375:   MachineBasicBlock *trueMBB = MF->CreateMachineBasicBlock(LLVM_BB);
2376:   MachineBasicBlock *falseMBB = MF->CreateMachineBasicBlock(LLVM_BB);
2377: 
2378:   MachineFunction::iterator I;
2379:   for (I = MF->begin(); I != MF->end() && &(*I) != MBB; ++I)
2380:     ;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2381-2400

```cpp
2381:   if (I != MF->end())
2382:     ++I;
2383:   MF->insert(I, trueMBB);
2384:   MF->insert(I, falseMBB);
2385: 
2386:   // Set the call frame size on entry to the new basic blocks.
2387:   unsigned CallFrameSize = TII.getCallFrameSizeAt(MI);
2388:   trueMBB->setCallFrameSize(CallFrameSize);
2389:   falseMBB->setCallFrameSize(CallFrameSize);
2390: 
2391:   // Transfer remaining instructions and all successors of the current
2392:   // block to the block which will contain the Phi node for the
2393:   // select.
2394:   trueMBB->splice(trueMBB->begin(), MBB,
2395:                   std::next(MachineBasicBlock::iterator(MI)), MBB->end());
2396:   trueMBB->transferSuccessorsAndUpdatePHIs(MBB);
2397: 
2398:   AVRCC::CondCodes CC = (AVRCC::CondCodes)MI.getOperand(3).getImm();
2399:   BuildMI(MBB, dl, TII.getBrCond(CC)).addMBB(trueMBB);
2400:   BuildMI(MBB, dl, TII.get(AVR::RJMPk)).addMBB(falseMBB);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2401-2420

```cpp
2401:   MBB->addSuccessor(falseMBB);
2402:   MBB->addSuccessor(trueMBB);
2403: 
2404:   // Unconditionally flow back to the true block
2405:   BuildMI(falseMBB, dl, TII.get(AVR::RJMPk)).addMBB(trueMBB);
2406:   falseMBB->addSuccessor(trueMBB);
2407: 
2408:   // Set up the Phi node to determine where we came from
2409:   BuildMI(*trueMBB, trueMBB->begin(), dl, TII.get(AVR::PHI),
2410:           MI.getOperand(0).getReg())
2411:       .addReg(MI.getOperand(1).getReg())
2412:       .addMBB(MBB)
2413:       .addReg(MI.getOperand(2).getReg())
2414:       .addMBB(falseMBB);
2415: 
2416:   MI.eraseFromParent(); // The pseudo instruction is gone now.
2417:   return trueMBB;
2418: }
2419: 
2420: //===----------------------------------------------------------------------===//
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2421-2440

```cpp
2421: //  Inline Asm Support
2422: //===----------------------------------------------------------------------===//
2423: 
2424: AVRTargetLowering::ConstraintType
2425: AVRTargetLowering::getConstraintType(StringRef Constraint) const {
2426:   if (Constraint.size() == 1) {
2427:     // See http://www.nongnu.org/avr-libc/user-manual/inline_asm.html
2428:     switch (Constraint[0]) {
2429:     default:
2430:       break;
2431:     case 'a': // Simple upper registers
2432:     case 'b': // Base pointer registers pairs
2433:     case 'd': // Upper register
2434:     case 'l': // Lower registers
2435:     case 'e': // Pointer register pairs
2436:     case 'q': // Stack pointer register
2437:     case 'r': // Any register
2438:     case 'w': // Special upper register pairs
2439:       return C_RegisterClass;
2440:     case 't': // Temporary register
```

- EN: Function bodies or method definitions such as getConstraintType contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getConstraintType 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2441-2460

```cpp
2441:     case 'x':
2442:     case 'X': // Pointer register pair X
2443:     case 'y':
2444:     case 'Y': // Pointer register pair Y
2445:     case 'z':
2446:     case 'Z': // Pointer register pair Z
2447:       return C_Register;
2448:     case 'Q': // A memory address based on Y or Z pointer with displacement.
2449:       return C_Memory;
2450:     case 'G': // Floating point constant
2451:     case 'I': // 6-bit positive integer constant
2452:     case 'J': // 6-bit negative integer constant
2453:     case 'K': // Integer constant (Range: 2)
2454:     case 'L': // Integer constant (Range: 0)
2455:     case 'M': // 8-bit integer constant
2456:     case 'N': // Integer constant (Range: -1)
2457:     case 'O': // Integer constant (Range: 8, 16, 24)
2458:     case 'P': // Integer constant (Range: 1)
2459:     case 'R': // Integer constant (Range: -6 to 5)x
2460:       return C_Immediate;
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2461-2480

```cpp
2461:     }
2462:   }
2463: 
2464:   return TargetLowering::getConstraintType(Constraint);
2465: }
2466: 
2467: InlineAsm::ConstraintCode
2468: AVRTargetLowering::getInlineAsmMemConstraint(StringRef ConstraintCode) const {
2469:   // Not sure if this is actually the right thing to do, but we got to do
2470:   // *something* [agnat]
2471:   switch (ConstraintCode[0]) {
2472:   case 'Q':
2473:     return InlineAsm::ConstraintCode::Q;
2474:   }
2475:   return TargetLowering::getInlineAsmMemConstraint(ConstraintCode);
2476: }
2477: 
2478: AVRTargetLowering::ConstraintWeight
2479: AVRTargetLowering::getSingleConstraintMatchWeight(
2480:     AsmOperandInfo &info, const char *constraint) const {
```

- EN: Function bodies or method definitions such as getInlineAsmMemConstraint, getSingleConstraintMatchWeight contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getInlineAsmMemConstraint, getSingleConstraintMatchWeight 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2481-2500

```cpp
2481:   ConstraintWeight weight = CW_Invalid;
2482:   Value *CallOperandVal = info.CallOperandVal;
2483: 
2484:   // If we don't have a value, we can't do a match,
2485:   // but allow it at the lowest weight.
2486:   // (this behaviour has been copied from the ARM backend)
2487:   if (!CallOperandVal) {
2488:     return CW_Default;
2489:   }
2490: 
2491:   // Look at the constraint type.
2492:   switch (*constraint) {
2493:   default:
2494:     weight = TargetLowering::getSingleConstraintMatchWeight(info, constraint);
2495:     break;
2496:   case 'd':
2497:   case 'r':
2498:   case 'l':
2499:     weight = CW_Register;
2500:     break;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2501-2520

```cpp
2501:   case 'a':
2502:   case 'b':
2503:   case 'e':
2504:   case 'q':
2505:   case 't':
2506:   case 'w':
2507:   case 'x':
2508:   case 'X':
2509:   case 'y':
2510:   case 'Y':
2511:   case 'z':
2512:   case 'Z':
2513:     weight = CW_SpecificReg;
2514:     break;
2515:   case 'G':
2516:     if (const ConstantFP *C = dyn_cast<ConstantFP>(CallOperandVal)) {
2517:       if (C->isZero()) {
2518:         weight = CW_Constant;
2519:       }
2520:     }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2521-2540

```cpp
2521:     break;
2522:   case 'I':
2523:     if (const ConstantInt *C = dyn_cast<ConstantInt>(CallOperandVal)) {
2524:       if (isUInt<6>(C->getZExtValue())) {
2525:         weight = CW_Constant;
2526:       }
2527:     }
2528:     break;
2529:   case 'J':
2530:     if (const ConstantInt *C = dyn_cast<ConstantInt>(CallOperandVal)) {
2531:       if ((C->getSExtValue() >= -63) && (C->getSExtValue() <= 0)) {
2532:         weight = CW_Constant;
2533:       }
2534:     }
2535:     break;
2536:   case 'K':
2537:     if (const ConstantInt *C = dyn_cast<ConstantInt>(CallOperandVal)) {
2538:       if (C->getZExtValue() == 2) {
2539:         weight = CW_Constant;
2540:       }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2541-2560

```cpp
2541:     }
2542:     break;
2543:   case 'L':
2544:     if (const ConstantInt *C = dyn_cast<ConstantInt>(CallOperandVal)) {
2545:       if (C->getZExtValue() == 0) {
2546:         weight = CW_Constant;
2547:       }
2548:     }
2549:     break;
2550:   case 'M':
2551:     if (const ConstantInt *C = dyn_cast<ConstantInt>(CallOperandVal)) {
2552:       if (isUInt<8>(C->getZExtValue())) {
2553:         weight = CW_Constant;
2554:       }
2555:     }
2556:     break;
2557:   case 'N':
2558:     if (const ConstantInt *C = dyn_cast<ConstantInt>(CallOperandVal)) {
2559:       if (C->getSExtValue() == -1) {
2560:         weight = CW_Constant;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2561-2580

```cpp
2561:       }
2562:     }
2563:     break;
2564:   case 'O':
2565:     if (const ConstantInt *C = dyn_cast<ConstantInt>(CallOperandVal)) {
2566:       if ((C->getZExtValue() == 8) || (C->getZExtValue() == 16) ||
2567:           (C->getZExtValue() == 24)) {
2568:         weight = CW_Constant;
2569:       }
2570:     }
2571:     break;
2572:   case 'P':
2573:     if (const ConstantInt *C = dyn_cast<ConstantInt>(CallOperandVal)) {
2574:       if (C->getZExtValue() == 1) {
2575:         weight = CW_Constant;
2576:       }
2577:     }
2578:     break;
2579:   case 'R':
2580:     if (const ConstantInt *C = dyn_cast<ConstantInt>(CallOperandVal)) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2581-2600

```cpp
2581:       if ((C->getSExtValue() >= -6) && (C->getSExtValue() <= 5)) {
2582:         weight = CW_Constant;
2583:       }
2584:     }
2585:     break;
2586:   case 'Q':
2587:     weight = CW_Memory;
2588:     break;
2589:   }
2590: 
2591:   return weight;
2592: }
2593: 
2594: std::pair<unsigned, const TargetRegisterClass *>
2595: AVRTargetLowering::getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
2596:                                                 StringRef Constraint,
2597:                                                 MVT VT) const {
2598:   if (Constraint.size() == 1) {
2599:     switch (Constraint[0]) {
2600:     case 'a': // Simple upper registers r16..r23.
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2601-2620

```cpp
2601:       if (VT == MVT::i8)
2602:         return std::make_pair(0U, &AVR::LD8loRegClass);
2603:       else if (VT == MVT::i16)
2604:         return std::make_pair(0U, &AVR::DREGSLD8loRegClass);
2605:       break;
2606:     case 'b': // Base pointer registers: y, z.
2607:       if (VT == MVT::i8 || VT == MVT::i16)
2608:         return std::make_pair(0U, &AVR::PTRDISPREGSRegClass);
2609:       break;
2610:     case 'd': // Upper registers r16..r31.
2611:       if (VT == MVT::i8)
2612:         return std::make_pair(0U, &AVR::LD8RegClass);
2613:       else if (VT == MVT::i16)
2614:         return std::make_pair(0U, &AVR::DLDREGSRegClass);
2615:       break;
2616:     case 'l': // Lower registers r0..r15.
2617:       if (VT == MVT::i8)
2618:         return std::make_pair(0U, &AVR::GPR8loRegClass);
2619:       else if (VT == MVT::i16)
2620:         return std::make_pair(0U, &AVR::DREGSloRegClass);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2621-2640

```cpp
2621:       break;
2622:     case 'e': // Pointer register pairs: x, y, z.
2623:       if (VT == MVT::i8 || VT == MVT::i16)
2624:         return std::make_pair(0U, &AVR::PTRREGSRegClass);
2625:       break;
2626:     case 'q': // Stack pointer register: SPH:SPL.
2627:       return std::make_pair(0U, &AVR::GPRSPRegClass);
2628:     case 'r': // Any register: r0..r31.
2629:       if (VT == MVT::i8)
2630:         return std::make_pair(0U, &AVR::GPR8RegClass);
2631:       else if (VT == MVT::i16)
2632:         return std::make_pair(0U, &AVR::DREGSRegClass);
2633:       break;
2634:     case 't': // Temporary register: r0.
2635:       if (VT == MVT::i8)
2636:         return std::make_pair(unsigned(Subtarget.getTmpRegister()),
2637:                               &AVR::GPR8RegClass);
2638:       break;
2639:     case 'w': // Special upper register pairs: r24, r26, r28, r30.
2640:       if (VT == MVT::i8 || VT == MVT::i16)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2641-2660

```cpp
2641:         return std::make_pair(0U, &AVR::IWREGSRegClass);
2642:       break;
2643:     case 'x': // Pointer register pair X: r27:r26.
2644:     case 'X':
2645:       if (VT == MVT::i8 || VT == MVT::i16)
2646:         return std::make_pair(unsigned(AVR::R27R26), &AVR::PTRREGSRegClass);
2647:       break;
2648:     case 'y': // Pointer register pair Y: r29:r28.
2649:     case 'Y':
2650:       if (VT == MVT::i8 || VT == MVT::i16)
2651:         return std::make_pair(unsigned(AVR::R29R28), &AVR::PTRREGSRegClass);
2652:       break;
2653:     case 'z': // Pointer register pair Z: r31:r30.
2654:     case 'Z':
2655:       if (VT == MVT::i8 || VT == MVT::i16)
2656:         return std::make_pair(unsigned(AVR::R31R30), &AVR::PTRREGSRegClass);
2657:       break;
2658:     default:
2659:       break;
2660:     }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2661-2680

```cpp
2661:   }
2662: 
2663:   return TargetLowering::getRegForInlineAsmConstraint(
2664:       Subtarget.getRegisterInfo(), Constraint, VT);
2665: }
2666: 
2667: void AVRTargetLowering::LowerAsmOperandForConstraint(SDValue Op,
2668:                                                      StringRef Constraint,
2669:                                                      std::vector<SDValue> &Ops,
2670:                                                      SelectionDAG &DAG) const {
2671:   SDValue Result;
2672:   SDLoc DL(Op);
2673:   EVT Ty = Op.getValueType();
2674: 
2675:   // Currently only support length 1 constraints.
2676:   if (Constraint.size() != 1) {
2677:     return;
2678:   }
2679: 
2680:   char ConstraintLetter = Constraint[0];
```

- EN: Function bodies or method definitions such as LowerAsmOperandForConstraint contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerAsmOperandForConstraint 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2681-2700

```cpp
2681:   switch (ConstraintLetter) {
2682:   default:
2683:     break;
2684:   // Deal with integers first:
2685:   case 'I':
2686:   case 'J':
2687:   case 'K':
2688:   case 'L':
2689:   case 'M':
2690:   case 'N':
2691:   case 'O':
2692:   case 'P':
2693:   case 'R': {
2694:     const ConstantSDNode *C = dyn_cast<ConstantSDNode>(Op);
2695:     if (!C) {
2696:       return;
2697:     }
2698: 
2699:     int64_t CVal64 = C->getSExtValue();
2700:     uint64_t CUVal64 = C->getZExtValue();
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2701-2720

```cpp
2701:     switch (ConstraintLetter) {
2702:     case 'I': // 0..63
2703:       if (!isUInt<6>(CUVal64))
2704:         return;
2705:       Result = DAG.getTargetConstant(CUVal64, DL, Ty);
2706:       break;
2707:     case 'J': // -63..0
2708:       if (CVal64 < -63 || CVal64 > 0)
2709:         return;
2710:       Result = DAG.getTargetConstant(CVal64, DL, Ty);
2711:       break;
2712:     case 'K': // 2
2713:       if (CUVal64 != 2)
2714:         return;
2715:       Result = DAG.getTargetConstant(CUVal64, DL, Ty);
2716:       break;
2717:     case 'L': // 0
2718:       if (CUVal64 != 0)
2719:         return;
2720:       Result = DAG.getTargetConstant(CUVal64, DL, Ty);
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2721-2740

```cpp
2721:       break;
2722:     case 'M': // 0..255
2723:       if (!isUInt<8>(CUVal64))
2724:         return;
2725:       // i8 type may be printed as a negative number,
2726:       // e.g. 254 would be printed as -2,
2727:       // so we force it to i16 at least.
2728:       if (Ty.getSimpleVT() == MVT::i8) {
2729:         Ty = MVT::i16;
2730:       }
2731:       Result = DAG.getTargetConstant(CUVal64, DL, Ty);
2732:       break;
2733:     case 'N': // -1
2734:       if (CVal64 != -1)
2735:         return;
2736:       Result = DAG.getTargetConstant(CVal64, DL, Ty);
2737:       break;
2738:     case 'O': // 8, 16, 24
2739:       if (CUVal64 != 8 && CUVal64 != 16 && CUVal64 != 24)
2740:         return;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2741-2760

```cpp
2741:       Result = DAG.getTargetConstant(CUVal64, DL, Ty);
2742:       break;
2743:     case 'P': // 1
2744:       if (CUVal64 != 1)
2745:         return;
2746:       Result = DAG.getTargetConstant(CUVal64, DL, Ty);
2747:       break;
2748:     case 'R': // -6..5
2749:       if (CVal64 < -6 || CVal64 > 5)
2750:         return;
2751:       Result = DAG.getTargetConstant(CVal64, DL, Ty);
2752:       break;
2753:     }
2754: 
2755:     break;
2756:   }
2757:   case 'G':
2758:     const ConstantFPSDNode *FC = dyn_cast<ConstantFPSDNode>(Op);
2759:     if (!FC || !FC->isZero())
2760:       return;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2761-2780

```cpp
2761:     // Soften float to i8 0
2762:     Result = DAG.getTargetConstant(0, DL, MVT::i8);
2763:     break;
2764:   }
2765: 
2766:   if (Result.getNode()) {
2767:     Ops.push_back(Result);
2768:     return;
2769:   }
2770: 
2771:   return TargetLowering::LowerAsmOperandForConstraint(Op, Constraint, Ops, DAG);
2772: }
2773: 
2774: Register AVRTargetLowering::getRegisterByName(const char *RegName, LLT VT,
2775:                                               const MachineFunction &MF) const {
2776:   Register Reg;
2777: 
2778:   if (VT == LLT::scalar(8)) {
2779:     Reg = StringSwitch<unsigned>(RegName)
2780:               .Case("r0", AVR::R0)
```

- EN: Function bodies or method definitions such as getRegisterByName contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getRegisterByName 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 2781-2797

```cpp
2781:               .Case("r1", AVR::R1)
2782:               .Default(0);
2783:   } else {
2784:     Reg = StringSwitch<unsigned>(RegName)
2785:               .Case("r0", AVR::R1R0)
2786:               .Case("sp", AVR::SP)
2787:               .Default(0);
2788:   }
2789: 
2790:   if (Reg)
2791:     return Reg;
2792: 
2793:   report_fatal_error(
2794:       Twine("Invalid register name \"" + StringRef(RegName) + "\"."));
2795: }
2796: 
2797: } // end of namespace llvm
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

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

- Direct includes / 直接包含: `AVRISelLowering.h`, `llvm/ADT/ArrayRef.h`, `llvm/ADT/StringSwitch.h`, `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineRegisterInfo.h`, `llvm/CodeGen/SelectionDAG.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, SelectionDAG
- Generated or companion files / 生成或配套文件: `AVRGenCallingConv.inc`
- Local companions / 本地配套文件: `AVRISelLowering.h`
