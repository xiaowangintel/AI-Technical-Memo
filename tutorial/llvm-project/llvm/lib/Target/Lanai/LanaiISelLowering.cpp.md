# LanaiISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/Lanai/LanaiISelLowering.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file implements the LanaiTargetLowering class.
- 目的（中文）: 实现 SelectionDAG 降级钩子，将通用 LLVM IR 操作映射到目标专用 DAG 节点与调用约定。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- LanaiISelLowering.cpp - Lanai DAG Lowering Implementation ---------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements the LanaiTargetLowering class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "LanaiISelLowering.h"
  14: #include "LanaiCondCode.h"
  15: #include "LanaiMachineFunctionInfo.h"
  16: #include "LanaiSubtarget.h"
  17: #include "LanaiTargetObjectFile.h"
  18: #include "MCTargetDesc/LanaiBaseInfo.h"
  19: #include "llvm/ADT/APInt.h"
  20: #include "llvm/ADT/ArrayRef.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 21-40

```cpp
  21: #include "llvm/ADT/SmallVector.h"
  22: #include "llvm/ADT/StringRef.h"
  23: #include "llvm/ADT/StringSwitch.h"
  24: #include "llvm/CodeGen/CallingConvLower.h"
  25: #include "llvm/CodeGen/MachineFrameInfo.h"
  26: #include "llvm/CodeGen/MachineFunction.h"
  27: #include "llvm/CodeGen/MachineMemOperand.h"
  28: #include "llvm/CodeGen/MachineRegisterInfo.h"
  29: #include "llvm/CodeGen/SelectionDAG.h"
  30: #include "llvm/CodeGen/SelectionDAGNodes.h"
  31: #include "llvm/CodeGen/TargetCallingConv.h"
  32: #include "llvm/CodeGen/ValueTypes.h"
  33: #include "llvm/CodeGenTypes/MachineValueType.h"
  34: #include "llvm/IR/CallingConv.h"
  35: #include "llvm/IR/DerivedTypes.h"
  36: #include "llvm/IR/Function.h"
  37: #include "llvm/IR/GlobalValue.h"
  38: #include "llvm/Support/Casting.h"
  39: #include "llvm/Support/CodeGen.h"
  40: #include "llvm/Support/CommandLine.h"
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 41-60

```cpp
  41: #include "llvm/Support/Debug.h"
  42: #include "llvm/Support/ErrorHandling.h"
  43: #include "llvm/Support/KnownBits.h"
  44: #include "llvm/Support/MathExtras.h"
  45: #include "llvm/Support/raw_ostream.h"
  46: #include "llvm/Target/TargetMachine.h"
  47: #include <cassert>
  48: #include <cmath>
  49: #include <cstdint>
  50: #include <cstdlib>
  51: #include <utility>
  52: 
  53: #define DEBUG_TYPE "lanai-lower"
  54: 
  55: using namespace llvm;
  56: 
  57: // Limit on number of instructions the lowered multiplication may have before a
  58: // call to the library function should be generated instead. The threshold is
  59: // currently set to 14 as this was the smallest threshold that resulted in all
  60: // constant multiplications being lowered. A threshold of 5 covered all cases
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 61-80

```cpp
  61: // except for one multiplication which required 14. mulsi3 requires 16
  62: // instructions (including the prologue and epilogue but excluding instructions
  63: // at call site). Until we can inline mulsi3, generating at most 14 instructions
  64: // will be faster than invoking mulsi3.
  65: static cl::opt<int> LanaiLowerConstantMulThreshold(
  66:     "lanai-constant-mul-threshold", cl::Hidden,
  67:     cl::desc("Maximum number of instruction to generate when lowering constant "
  68:              "multiplication instead of calling library function [default=14]"),
  69:     cl::init(14));
  70: 
  71: LanaiTargetLowering::LanaiTargetLowering(const TargetMachine &TM,
  72:                                          const LanaiSubtarget &STI)
  73:     : TargetLowering(TM, STI) {
  74:   // Set up the register classes.
  75:   addRegisterClass(MVT::i32, &Lanai::GPRRegClass);
  76: 
  77:   // Compute derived properties from the register classes
  78:   TRI = STI.getRegisterInfo();
  79:   computeRegisterProperties(TRI);
  80: 
```

- EN: Function bodies or method definitions such as TargetLowering contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: TargetLowering 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 81-100

```cpp
  81:   setStackPointerRegisterToSaveRestore(Lanai::SP);
  82: 
  83:   setOperationAction(ISD::BR_CC, MVT::i32, Custom);
  84:   setOperationAction(ISD::BR_JT, MVT::Other, Expand);
  85:   setOperationAction(ISD::BRCOND, MVT::Other, Expand);
  86:   setOperationAction(ISD::SETCC, MVT::i32, Custom);
  87:   setOperationAction(ISD::SELECT, MVT::i32, Expand);
  88:   setOperationAction(ISD::SELECT_CC, MVT::i32, Custom);
  89: 
  90:   setOperationAction(ISD::GlobalAddress, MVT::i32, Custom);
  91:   setOperationAction(ISD::BlockAddress, MVT::i32, Custom);
  92:   setOperationAction(ISD::JumpTable, MVT::i32, Custom);
  93:   setOperationAction(ISD::ConstantPool, MVT::i32, Custom);
  94: 
  95:   setOperationAction(ISD::DYNAMIC_STACKALLOC, MVT::i32, Custom);
  96:   setOperationAction(ISD::STACKSAVE, MVT::Other, Expand);
  97:   setOperationAction(ISD::STACKRESTORE, MVT::Other, Expand);
  98: 
  99:   setOperationAction(ISD::VASTART, MVT::Other, Custom);
 100:   setOperationAction(ISD::VAARG, MVT::Other, Expand);
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 101-120

```cpp
 101:   setOperationAction(ISD::VACOPY, MVT::Other, Expand);
 102:   setOperationAction(ISD::VAEND, MVT::Other, Expand);
 103: 
 104:   setOperationAction(ISD::SDIV, MVT::i32, Expand);
 105:   setOperationAction(ISD::UDIV, MVT::i32, Expand);
 106:   setOperationAction(ISD::SDIVREM, MVT::i32, Expand);
 107:   setOperationAction(ISD::UDIVREM, MVT::i32, Expand);
 108:   setOperationAction(ISD::SREM, MVT::i32, Expand);
 109:   setOperationAction(ISD::UREM, MVT::i32, Expand);
 110: 
 111:   setOperationAction(ISD::MUL, MVT::i32, Custom);
 112:   setOperationAction(ISD::MULHU, MVT::i32, Expand);
 113:   setOperationAction(ISD::MULHS, MVT::i32, Expand);
 114:   setOperationAction(ISD::UMUL_LOHI, MVT::i32, Expand);
 115:   setOperationAction(ISD::SMUL_LOHI, MVT::i32, Expand);
 116: 
 117:   setOperationAction(ISD::ROTR, MVT::i32, Expand);
 118:   setOperationAction(ISD::ROTL, MVT::i32, Expand);
 119:   setOperationAction(ISD::SHL_PARTS, MVT::i32, Custom);
 120:   setOperationAction(ISD::SRL_PARTS, MVT::i32, Custom);
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 121-140

```cpp
 121:   setOperationAction(ISD::SRA_PARTS, MVT::i32, Expand);
 122: 
 123:   setOperationAction(ISD::BSWAP, MVT::i32, Expand);
 124:   setOperationAction(ISD::CTPOP, MVT::i32, Legal);
 125:   setOperationAction(ISD::CTLZ, MVT::i32, Legal);
 126:   setOperationAction(ISD::CTTZ, MVT::i32, Legal);
 127: 
 128:   setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i1, Expand);
 129:   setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i8, Expand);
 130:   setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i16, Expand);
 131: 
 132:   // Extended load operations for i1 types must be promoted
 133:   for (MVT VT : MVT::integer_valuetypes()) {
 134:     setLoadExtAction(ISD::EXTLOAD, VT, MVT::i1, Promote);
 135:     setLoadExtAction(ISD::ZEXTLOAD, VT, MVT::i1, Promote);
 136:     setLoadExtAction(ISD::SEXTLOAD, VT, MVT::i1, Promote);
 137:   }
 138: 
 139:   setTargetDAGCombine({ISD::ADD, ISD::SUB, ISD::AND, ISD::OR, ISD::XOR});
 140: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 141-160

```cpp
 141:   // Function alignments
 142:   setMinFunctionAlignment(Align(4));
 143:   setPrefFunctionAlignment(Align(4));
 144: 
 145:   setJumpIsExpensive(true);
 146: 
 147:   // TODO: Setting the minimum jump table entries needed before a
 148:   // switch is transformed to a jump table to 100 to avoid creating jump tables
 149:   // as this was causing bad performance compared to a large group of if
 150:   // statements. Re-evaluate this on new benchmarks.
 151:   setMinimumJumpTableEntries(100);
 152: 
 153:   MaxStoresPerMemset = 16; // For @llvm.memset -> sequence of stores
 154:   MaxStoresPerMemsetOptSize = 8;
 155:   MaxStoresPerMemcpy = 16; // For @llvm.memcpy -> sequence of stores
 156:   MaxStoresPerMemcpyOptSize = 8;
 157:   MaxStoresPerMemmove = 16; // For @llvm.memmove -> sequence of stores
 158:   MaxStoresPerMemmoveOptSize = 8;
 159: 
 160:   // Booleans always contain 0 or 1.
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 161-180

```cpp
 161:   setBooleanContents(ZeroOrOneBooleanContent);
 162: 
 163:   setMaxAtomicSizeInBitsSupported(0);
 164: }
 165: 
 166: SDValue LanaiTargetLowering::LowerOperation(SDValue Op,
 167:                                             SelectionDAG &DAG) const {
 168:   switch (Op.getOpcode()) {
 169:   case ISD::MUL:
 170:     return LowerMUL(Op, DAG);
 171:   case ISD::BR_CC:
 172:     return LowerBR_CC(Op, DAG);
 173:   case ISD::ConstantPool:
 174:     return LowerConstantPool(Op, DAG);
 175:   case ISD::GlobalAddress:
 176:     return LowerGlobalAddress(Op, DAG);
 177:   case ISD::BlockAddress:
 178:     return LowerBlockAddress(Op, DAG);
 179:   case ISD::JumpTable:
 180:     return LowerJumpTable(Op, DAG);
```

- EN: Function bodies or method definitions such as LowerOperation contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerOperation 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 181-200

```cpp
 181:   case ISD::SELECT_CC:
 182:     return LowerSELECT_CC(Op, DAG);
 183:   case ISD::SETCC:
 184:     return LowerSETCC(Op, DAG);
 185:   case ISD::SHL_PARTS:
 186:     return LowerSHL_PARTS(Op, DAG);
 187:   case ISD::SRL_PARTS:
 188:     return LowerSRL_PARTS(Op, DAG);
 189:   case ISD::VASTART:
 190:     return LowerVASTART(Op, DAG);
 191:   case ISD::DYNAMIC_STACKALLOC:
 192:     return LowerDYNAMIC_STACKALLOC(Op, DAG);
 193:   case ISD::RETURNADDR:
 194:     return LowerRETURNADDR(Op, DAG);
 195:   case ISD::FRAMEADDR:
 196:     return LowerFRAMEADDR(Op, DAG);
 197:   default:
 198:     llvm_unreachable("unimplemented operand");
 199:   }
 200: }
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 201-220

```cpp
 201: 
 202: //===----------------------------------------------------------------------===//
 203: //                       Lanai Inline Assembly Support
 204: //===----------------------------------------------------------------------===//
 205: 
 206: Register LanaiTargetLowering::getRegisterByName(
 207:   const char *RegName, LLT /*VT*/,
 208:   const MachineFunction & /*MF*/) const {
 209:   // Only unallocatable registers should be matched here.
 210:   Register Reg = StringSwitch<Register>(RegName)
 211:                      .Case("pc", Lanai::PC)
 212:                      .Case("sp", Lanai::SP)
 213:                      .Case("fp", Lanai::FP)
 214:                      .Case("rr1", Lanai::RR1)
 215:                      .Case("r10", Lanai::R10)
 216:                      .Case("rr2", Lanai::RR2)
 217:                      .Case("r11", Lanai::R11)
 218:                      .Case("rca", Lanai::RCA)
 219:                      .Default(Register());
 220:   return Reg;
```

- EN: Function bodies or method definitions such as getRegisterByName contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getRegisterByName 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 221-240

```cpp
 221: }
 222: 
 223: std::pair<unsigned, const TargetRegisterClass *>
 224: LanaiTargetLowering::getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
 225:                                                   StringRef Constraint,
 226:                                                   MVT VT) const {
 227:   if (Constraint.size() == 1)
 228:     // GCC Constraint Letters
 229:     switch (Constraint[0]) {
 230:     case 'r': // GENERAL_REGS
 231:       return std::make_pair(0U, &Lanai::GPRRegClass);
 232:     default:
 233:       break;
 234:     }
 235: 
 236:   return TargetLowering::getRegForInlineAsmConstraint(TRI, Constraint, VT);
 237: }
 238: 
 239: // Examine constraint type and operand type and determine a weight value.
 240: // This object must already have been set up with the operand type
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 241-260

```cpp
 241: // and the current alternative constraint selected.
 242: TargetLowering::ConstraintWeight
 243: LanaiTargetLowering::getSingleConstraintMatchWeight(
 244:     AsmOperandInfo &Info, const char *Constraint) const {
 245:   ConstraintWeight Weight = CW_Invalid;
 246:   Value *CallOperandVal = Info.CallOperandVal;
 247:   // If we don't have a value, we can't do a match,
 248:   // but allow it at the lowest weight.
 249:   if (CallOperandVal == nullptr)
 250:     return CW_Default;
 251:   // Look at the constraint type.
 252:   switch (*Constraint) {
 253:   case 'I': // signed 16 bit immediate
 254:   case 'J': // integer zero
 255:   case 'K': // unsigned 16 bit immediate
 256:   case 'L': // immediate in the range 0 to 31
 257:   case 'M': // signed 32 bit immediate where lower 16 bits are 0
 258:   case 'N': // signed 26 bit immediate
 259:   case 'O': // integer zero
 260:     if (isa<ConstantInt>(CallOperandVal))
```

- EN: Function bodies or method definitions such as getSingleConstraintMatchWeight contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getSingleConstraintMatchWeight 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 261-280

```cpp
 261:       Weight = CW_Constant;
 262:     break;
 263:   default:
 264:     Weight = TargetLowering::getSingleConstraintMatchWeight(Info, Constraint);
 265:     break;
 266:   }
 267:   return Weight;
 268: }
 269: 
 270: // LowerAsmOperandForConstraint - Lower the specified operand into the Ops
 271: // vector.  If it is invalid, don't add anything to Ops.
 272: void LanaiTargetLowering::LowerAsmOperandForConstraint(
 273:     SDValue Op, StringRef Constraint, std::vector<SDValue> &Ops,
 274:     SelectionDAG &DAG) const {
 275:   SDValue Result;
 276: 
 277:   // Only support length 1 constraints for now.
 278:   if (Constraint.size() > 1)
 279:     return;
 280: 
```

- EN: Function bodies or method definitions such as LowerAsmOperandForConstraint contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerAsmOperandForConstraint 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 281-300

```cpp
 281:   char ConstraintLetter = Constraint[0];
 282:   switch (ConstraintLetter) {
 283:   case 'I': // Signed 16 bit constant
 284:     // If this fails, the parent routine will give an error
 285:     if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Op)) {
 286:       if (isInt<16>(C->getSExtValue())) {
 287:         Result = DAG.getTargetConstant(C->getSExtValue(), SDLoc(C),
 288:                                        Op.getValueType());
 289:         break;
 290:       }
 291:     }
 292:     return;
 293:   case 'J': // integer zero
 294:   case 'O':
 295:     if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Op)) {
 296:       if (C->getZExtValue() == 0) {
 297:         Result = DAG.getTargetConstant(0, SDLoc(C), Op.getValueType());
 298:         break;
 299:       }
 300:     }
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 301-320

```cpp
 301:     return;
 302:   case 'K': // unsigned 16 bit immediate
 303:     if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Op)) {
 304:       if (isUInt<16>(C->getZExtValue())) {
 305:         Result = DAG.getTargetConstant(C->getSExtValue(), SDLoc(C),
 306:                                        Op.getValueType());
 307:         break;
 308:       }
 309:     }
 310:     return;
 311:   case 'L': // immediate in the range 0 to 31
 312:     if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Op)) {
 313:       if (C->getZExtValue() <= 31) {
 314:         Result = DAG.getTargetConstant(C->getZExtValue(), SDLoc(C),
 315:                                        Op.getValueType());
 316:         break;
 317:       }
 318:     }
 319:     return;
 320:   case 'M': // signed 32 bit immediate where lower 16 bits are 0
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 321-340

```cpp
 321:     if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Op)) {
 322:       int64_t Val = C->getSExtValue();
 323:       if ((isInt<32>(Val)) && ((Val & 0xffff) == 0)) {
 324:         Result = DAG.getTargetConstant(Val, SDLoc(C), Op.getValueType());
 325:         break;
 326:       }
 327:     }
 328:     return;
 329:   case 'N': // signed 26 bit immediate
 330:     if (ConstantSDNode *C = dyn_cast<ConstantSDNode>(Op)) {
 331:       int64_t Val = C->getSExtValue();
 332:       if ((Val >= -33554432) && (Val <= 33554431)) {
 333:         Result = DAG.getTargetConstant(Val, SDLoc(C), Op.getValueType());
 334:         break;
 335:       }
 336:     }
 337:     return;
 338:   default:
 339:     break; // This will fall through to the generic implementation
 340:   }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 341-360

```cpp
 341: 
 342:   if (Result.getNode()) {
 343:     Ops.push_back(Result);
 344:     return;
 345:   }
 346: 
 347:   TargetLowering::LowerAsmOperandForConstraint(Op, Constraint, Ops, DAG);
 348: }
 349: 
 350: //===----------------------------------------------------------------------===//
 351: //                      Calling Convention Implementation
 352: //===----------------------------------------------------------------------===//
 353: 
 354: #include "LanaiGenCallingConv.inc"
 355: 
 356: static bool CC_Lanai32_VarArg(unsigned ValNo, MVT ValVT, MVT LocVT,
 357:                               CCValAssign::LocInfo LocInfo,
 358:                               ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
 359:                               CCState &State) {
 360:   // Handle fixed arguments with default CC.
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as CC_Lanai32_VarArg contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 CC_Lanai32_VarArg 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 361-380

```cpp
 361:   // Note: Both the default and fast CC handle VarArg the same and hence the
 362:   // calling convention of the function is not considered here.
 363:   if (!ArgFlags.isVarArg())
 364:     return CC_Lanai32(ValNo, ValVT, LocVT, LocInfo, ArgFlags, OrigTy, State);
 365: 
 366:   // Promote i8/i16 args to i32
 367:   if (LocVT == MVT::i8 || LocVT == MVT::i16) {
 368:     LocVT = MVT::i32;
 369:     if (ArgFlags.isSExt())
 370:       LocInfo = CCValAssign::SExt;
 371:     else if (ArgFlags.isZExt())
 372:       LocInfo = CCValAssign::ZExt;
 373:     else
 374:       LocInfo = CCValAssign::AExt;
 375:   }
 376: 
 377:   // VarArgs get passed on stack
 378:   unsigned Offset = State.AllocateStack(4, Align(4));
 379:   State.addLoc(CCValAssign::getMem(ValNo, ValVT, Offset, LocVT, LocInfo));
 380:   return false;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 381-400

```cpp
 381: }
 382: 
 383: SDValue LanaiTargetLowering::LowerFormalArguments(
 384:     SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
 385:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &DL,
 386:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
 387:   switch (CallConv) {
 388:   case CallingConv::C:
 389:   case CallingConv::Fast:
 390:     return LowerCCCArguments(Chain, CallConv, IsVarArg, Ins, DL, DAG, InVals);
 391:   default:
 392:     report_fatal_error("Unsupported calling convention");
 393:   }
 394: }
 395: 
 396: SDValue LanaiTargetLowering::LowerCall(TargetLowering::CallLoweringInfo &CLI,
 397:                                        SmallVectorImpl<SDValue> &InVals) const {
 398:   SelectionDAG &DAG = CLI.DAG;
 399:   SDLoc &DL = CLI.DL;
 400:   SmallVectorImpl<ISD::OutputArg> &Outs = CLI.Outs;
```

- EN: Function bodies or method definitions such as LowerFormalArguments, LowerCall contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerFormalArguments, LowerCall 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 401-420

```cpp
 401:   SmallVectorImpl<SDValue> &OutVals = CLI.OutVals;
 402:   SmallVectorImpl<ISD::InputArg> &Ins = CLI.Ins;
 403:   SDValue Chain = CLI.Chain;
 404:   SDValue Callee = CLI.Callee;
 405:   bool &IsTailCall = CLI.IsTailCall;
 406:   CallingConv::ID CallConv = CLI.CallConv;
 407:   bool IsVarArg = CLI.IsVarArg;
 408: 
 409:   // Lanai target does not yet support tail call optimization.
 410:   IsTailCall = false;
 411: 
 412:   switch (CallConv) {
 413:   case CallingConv::Fast:
 414:   case CallingConv::C:
 415:     return LowerCCCCallTo(Chain, Callee, CallConv, IsVarArg, IsTailCall, Outs,
 416:                           OutVals, Ins, DL, DAG, InVals);
 417:   default:
 418:     report_fatal_error("Unsupported calling convention");
 419:   }
 420: }
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 421-440

```cpp
 421: 
 422: // LowerCCCArguments - transform physical registers into virtual registers and
 423: // generate load operations for arguments places on the stack.
 424: SDValue LanaiTargetLowering::LowerCCCArguments(
 425:     SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
 426:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &DL,
 427:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
 428:   MachineFunction &MF = DAG.getMachineFunction();
 429:   MachineFrameInfo &MFI = MF.getFrameInfo();
 430:   MachineRegisterInfo &RegInfo = MF.getRegInfo();
 431:   LanaiMachineFunctionInfo *LanaiMFI = MF.getInfo<LanaiMachineFunctionInfo>();
 432: 
 433:   // Assign locations to all of the incoming arguments.
 434:   SmallVector<CCValAssign, 16> ArgLocs;
 435:   CCState CCInfo(CallConv, IsVarArg, DAG.getMachineFunction(), ArgLocs,
 436:                  *DAG.getContext());
 437:   if (CallConv == CallingConv::Fast) {
 438:     CCInfo.AnalyzeFormalArguments(Ins, CC_Lanai32_Fast);
 439:   } else {
 440:     CCInfo.AnalyzeFormalArguments(Ins, CC_Lanai32);
```

- EN: Function bodies or method definitions such as LowerCCCArguments contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerCCCArguments 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 441-460

```cpp
 441:   }
 442: 
 443:   for (const CCValAssign &VA : ArgLocs) {
 444:     if (VA.isRegLoc()) {
 445:       // Arguments passed in registers
 446:       EVT RegVT = VA.getLocVT();
 447:       switch (RegVT.getSimpleVT().SimpleTy) {
 448:       case MVT::i32: {
 449:         Register VReg = RegInfo.createVirtualRegister(&Lanai::GPRRegClass);
 450:         RegInfo.addLiveIn(VA.getLocReg(), VReg);
 451:         SDValue ArgValue = DAG.getCopyFromReg(Chain, DL, VReg, RegVT);
 452: 
 453:         // If this is an 8/16-bit value, it is really passed promoted to 32
 454:         // bits. Insert an assert[sz]ext to capture this, then truncate to the
 455:         // right size.
 456:         if (VA.getLocInfo() == CCValAssign::SExt)
 457:           ArgValue = DAG.getNode(ISD::AssertSext, DL, RegVT, ArgValue,
 458:                                  DAG.getValueType(VA.getValVT()));
 459:         else if (VA.getLocInfo() == CCValAssign::ZExt)
 460:           ArgValue = DAG.getNode(ISD::AssertZext, DL, RegVT, ArgValue,
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 461-480

```cpp
 461:                                  DAG.getValueType(VA.getValVT()));
 462: 
 463:         if (VA.getLocInfo() != CCValAssign::Full)
 464:           ArgValue = DAG.getNode(ISD::TRUNCATE, DL, VA.getValVT(), ArgValue);
 465: 
 466:         InVals.push_back(ArgValue);
 467:         break;
 468:       }
 469:       default:
 470:         LLVM_DEBUG(dbgs() << "LowerFormalArguments Unhandled argument type: "
 471:                           << RegVT << "\n");
 472:         llvm_unreachable("unhandled argument type");
 473:       }
 474:     } else {
 475:       // Only arguments passed on the stack should make it here.
 476:       assert(VA.isMemLoc());
 477:       // Load the argument to a virtual register
 478:       unsigned ObjSize = VA.getLocVT().getSizeInBits() / 8;
 479:       // Check that the argument fits in stack slot
 480:       if (ObjSize > 4) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 481-500

```cpp
 481:         errs() << "LowerFormalArguments Unhandled argument type: "
 482:                << VA.getLocVT() << "\n";
 483:       }
 484:       // Create the frame index object for this incoming parameter...
 485:       int FI = MFI.CreateFixedObject(ObjSize, VA.getLocMemOffset(), true);
 486: 
 487:       // Create the SelectionDAG nodes corresponding to a load
 488:       // from this parameter
 489:       SDValue FIN = DAG.getFrameIndex(FI, MVT::i32);
 490:       InVals.push_back(DAG.getLoad(
 491:           VA.getLocVT(), DL, Chain, FIN,
 492:           MachinePointerInfo::getFixedStack(DAG.getMachineFunction(), FI)));
 493:     }
 494:   }
 495: 
 496:   // The Lanai ABI for returning structs by value requires that we copy
 497:   // the sret argument into rv for the return. Save the argument into
 498:   // a virtual register so that we can access it from the return points.
 499:   if (MF.getFunction().hasStructRetAttr()) {
 500:     Register Reg = LanaiMFI->getSRetReturnReg();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 501-520

```cpp
 501:     if (!Reg) {
 502:       Reg = MF.getRegInfo().createVirtualRegister(getRegClassFor(MVT::i32));
 503:       LanaiMFI->setSRetReturnReg(Reg);
 504:     }
 505:     SDValue Copy = DAG.getCopyToReg(DAG.getEntryNode(), DL, Reg, InVals[0]);
 506:     Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, Copy, Chain);
 507:   }
 508: 
 509:   if (IsVarArg) {
 510:     // Record the frame index of the first variable argument
 511:     // which is a value necessary to VASTART.
 512:     int FI = MFI.CreateFixedObject(4, CCInfo.getStackSize(), true);
 513:     LanaiMFI->setVarArgsFrameIndex(FI);
 514:   }
 515: 
 516:   return Chain;
 517: }
 518: 
 519: bool LanaiTargetLowering::CanLowerReturn(
 520:     CallingConv::ID CallConv, MachineFunction &MF, bool IsVarArg,
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 521-540

```cpp
 521:     const SmallVectorImpl<ISD::OutputArg> &Outs, LLVMContext &Context,
 522:     const Type *RetTy) const {
 523:   SmallVector<CCValAssign, 16> RVLocs;
 524:   CCState CCInfo(CallConv, IsVarArg, MF, RVLocs, Context);
 525: 
 526:   return CCInfo.CheckReturn(Outs, RetCC_Lanai32);
 527: }
 528: 
 529: SDValue
 530: LanaiTargetLowering::LowerReturn(SDValue Chain, CallingConv::ID CallConv,
 531:                                  bool IsVarArg,
 532:                                  const SmallVectorImpl<ISD::OutputArg> &Outs,
 533:                                  const SmallVectorImpl<SDValue> &OutVals,
 534:                                  const SDLoc &DL, SelectionDAG &DAG) const {
 535:   // CCValAssign - represent the assignment of the return value to a location
 536:   SmallVector<CCValAssign, 16> RVLocs;
 537: 
 538:   // CCState - Info about the registers and stack slot.
 539:   CCState CCInfo(CallConv, IsVarArg, DAG.getMachineFunction(), RVLocs,
 540:                  *DAG.getContext());
```

- EN: Function bodies or method definitions such as LowerReturn contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerReturn 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 541-560

```cpp
 541: 
 542:   // Analize return values.
 543:   CCInfo.AnalyzeReturn(Outs, RetCC_Lanai32);
 544: 
 545:   SDValue Glue;
 546:   SmallVector<SDValue, 4> RetOps(1, Chain);
 547: 
 548:   // Copy the result values into the output registers.
 549:   for (unsigned i = 0; i != RVLocs.size(); ++i) {
 550:     CCValAssign &VA = RVLocs[i];
 551:     assert(VA.isRegLoc() && "Can only return in registers!");
 552: 
 553:     Chain = DAG.getCopyToReg(Chain, DL, VA.getLocReg(), OutVals[i], Glue);
 554: 
 555:     // Guarantee that all emitted copies are stuck together with flags.
 556:     Glue = Chain.getValue(1);
 557:     RetOps.push_back(DAG.getRegister(VA.getLocReg(), VA.getLocVT()));
 558:   }
 559: 
 560:   // The Lanai ABI for returning structs by value requires that we copy
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 561-580

```cpp
 561:   // the sret argument into rv for the return. We saved the argument into
 562:   // a virtual register in the entry block, so now we copy the value out
 563:   // and into rv.
 564:   if (DAG.getMachineFunction().getFunction().hasStructRetAttr()) {
 565:     MachineFunction &MF = DAG.getMachineFunction();
 566:     LanaiMachineFunctionInfo *LanaiMFI = MF.getInfo<LanaiMachineFunctionInfo>();
 567:     Register Reg = LanaiMFI->getSRetReturnReg();
 568:     assert(Reg &&
 569:            "SRetReturnReg should have been set in LowerFormalArguments().");
 570:     SDValue Val =
 571:         DAG.getCopyFromReg(Chain, DL, Reg, getPointerTy(DAG.getDataLayout()));
 572: 
 573:     Chain = DAG.getCopyToReg(Chain, DL, Lanai::RV, Val, Glue);
 574:     Glue = Chain.getValue(1);
 575:     RetOps.push_back(
 576:         DAG.getRegister(Lanai::RV, getPointerTy(DAG.getDataLayout())));
 577:   }
 578: 
 579:   RetOps[0] = Chain; // Update chain
 580: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 581-600

```cpp
 581:   unsigned Opc = LanaiISD::RET_GLUE;
 582:   if (Glue.getNode())
 583:     RetOps.push_back(Glue);
 584: 
 585:   // Return Void
 586:   return DAG.getNode(Opc, DL, MVT::Other,
 587:                      ArrayRef<SDValue>(&RetOps[0], RetOps.size()));
 588: }
 589: 
 590: // LowerCCCCallTo - functions arguments are copied from virtual regs to
 591: // (physical regs)/(stack frame), CALLSEQ_START and CALLSEQ_END are emitted.
 592: SDValue LanaiTargetLowering::LowerCCCCallTo(
 593:     SDValue Chain, SDValue Callee, CallingConv::ID CallConv, bool IsVarArg,
 594:     bool /*IsTailCall*/, const SmallVectorImpl<ISD::OutputArg> &Outs,
 595:     const SmallVectorImpl<SDValue> &OutVals,
 596:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &DL,
 597:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
 598:   // Analyze operands of the call, assigning locations to each operand.
 599:   SmallVector<CCValAssign, 16> ArgLocs;
 600:   CCState CCInfo(CallConv, IsVarArg, DAG.getMachineFunction(), ArgLocs,
```

- EN: Function bodies or method definitions such as LowerCCCCallTo contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerCCCCallTo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 601-620

```cpp
 601:                  *DAG.getContext());
 602:   GlobalAddressSDNode *G = dyn_cast<GlobalAddressSDNode>(Callee);
 603:   MachineFrameInfo &MFI = DAG.getMachineFunction().getFrameInfo();
 604: 
 605:   if (IsVarArg) {
 606:     CCInfo.AnalyzeCallOperands(Outs, CC_Lanai32_VarArg);
 607:   } else {
 608:     if (CallConv == CallingConv::Fast)
 609:       CCInfo.AnalyzeCallOperands(Outs, CC_Lanai32_Fast);
 610:     else
 611:       CCInfo.AnalyzeCallOperands(Outs, CC_Lanai32);
 612:   }
 613: 
 614:   // Get a count of how many bytes are to be pushed on the stack.
 615:   unsigned NumBytes = CCInfo.getStackSize();
 616: 
 617:   // Create local copies for byval args.
 618:   SmallVector<SDValue, 8> ByValArgs;
 619:   for (unsigned I = 0, E = Outs.size(); I != E; ++I) {
 620:     ISD::ArgFlagsTy Flags = Outs[I].Flags;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 621-640

```cpp
 621:     if (!Flags.isByVal())
 622:       continue;
 623: 
 624:     SDValue Arg = OutVals[I];
 625:     unsigned Size = Flags.getByValSize();
 626:     Align Alignment = Flags.getNonZeroByValAlign();
 627: 
 628:     int FI = MFI.CreateStackObject(Size, Alignment, false);
 629:     SDValue FIPtr = DAG.getFrameIndex(FI, getPointerTy(DAG.getDataLayout()));
 630:     SDValue SizeNode = DAG.getConstant(Size, DL, MVT::i32);
 631: 
 632:     Chain = DAG.getMemcpy(Chain, DL, FIPtr, Arg, SizeNode, Alignment,
 633:                           /*IsVolatile=*/false,
 634:                           /*AlwaysInline=*/false,
 635:                           /*CI=*/nullptr, std::nullopt, MachinePointerInfo(),
 636:                           MachinePointerInfo());
 637:     ByValArgs.push_back(FIPtr);
 638:   }
 639: 
 640:   Chain = DAG.getCALLSEQ_START(Chain, NumBytes, 0, DL);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 641-660

```cpp
 641: 
 642:   SmallVector<std::pair<unsigned, SDValue>, 4> RegsToPass;
 643:   SmallVector<SDValue, 12> MemOpChains;
 644:   SDValue StackPtr;
 645: 
 646:   // Walk the register/memloc assignments, inserting copies/loads.
 647:   for (unsigned I = 0, J = 0, E = ArgLocs.size(); I != E; ++I) {
 648:     CCValAssign &VA = ArgLocs[I];
 649:     SDValue Arg = OutVals[I];
 650:     ISD::ArgFlagsTy Flags = Outs[I].Flags;
 651: 
 652:     // Promote the value if needed.
 653:     switch (VA.getLocInfo()) {
 654:     case CCValAssign::Full:
 655:       break;
 656:     case CCValAssign::SExt:
 657:       Arg = DAG.getNode(ISD::SIGN_EXTEND, DL, VA.getLocVT(), Arg);
 658:       break;
 659:     case CCValAssign::ZExt:
 660:       Arg = DAG.getNode(ISD::ZERO_EXTEND, DL, VA.getLocVT(), Arg);
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 661-680

```cpp
 661:       break;
 662:     case CCValAssign::AExt:
 663:       Arg = DAG.getNode(ISD::ANY_EXTEND, DL, VA.getLocVT(), Arg);
 664:       break;
 665:     default:
 666:       llvm_unreachable("Unknown loc info!");
 667:     }
 668: 
 669:     // Use local copy if it is a byval arg.
 670:     if (Flags.isByVal())
 671:       Arg = ByValArgs[J++];
 672: 
 673:     // Arguments that can be passed on register must be kept at RegsToPass
 674:     // vector
 675:     if (VA.isRegLoc()) {
 676:       RegsToPass.push_back(std::make_pair(VA.getLocReg(), Arg));
 677:     } else {
 678:       assert(VA.isMemLoc());
 679: 
 680:       if (StackPtr.getNode() == nullptr)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 681-700

```cpp
 681:         StackPtr = DAG.getCopyFromReg(Chain, DL, Lanai::SP,
 682:                                       getPointerTy(DAG.getDataLayout()));
 683: 
 684:       SDValue PtrOff =
 685:           DAG.getNode(ISD::ADD, DL, getPointerTy(DAG.getDataLayout()), StackPtr,
 686:                       DAG.getIntPtrConstant(VA.getLocMemOffset(), DL));
 687: 
 688:       MemOpChains.push_back(
 689:           DAG.getStore(Chain, DL, Arg, PtrOff, MachinePointerInfo()));
 690:     }
 691:   }
 692: 
 693:   // Transform all store nodes into one single node because all store nodes are
 694:   // independent of each other.
 695:   if (!MemOpChains.empty())
 696:     Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other,
 697:                         ArrayRef<SDValue>(&MemOpChains[0], MemOpChains.size()));
 698: 
 699:   SDValue InGlue;
 700: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 701-720

```cpp
 701:   // Build a sequence of copy-to-reg nodes chained together with token chain and
 702:   // flag operands which copy the outgoing args into registers.  The InGlue in
 703:   // necessary since all emitted instructions must be stuck together.
 704:   for (const auto &[Reg, N] : RegsToPass) {
 705:     Chain = DAG.getCopyToReg(Chain, DL, Reg, N, InGlue);
 706:     InGlue = Chain.getValue(1);
 707:   }
 708: 
 709:   // If the callee is a GlobalAddress node (quite common, every direct call is)
 710:   // turn it into a TargetGlobalAddress node so that legalize doesn't hack it.
 711:   // Likewise ExternalSymbol -> TargetExternalSymbol.
 712:   uint8_t OpFlag = LanaiII::MO_NO_FLAG;
 713:   if (G) {
 714:     Callee = DAG.getTargetGlobalAddress(
 715:         G->getGlobal(), DL, getPointerTy(DAG.getDataLayout()), 0, OpFlag);
 716:   } else if (ExternalSymbolSDNode *E = dyn_cast<ExternalSymbolSDNode>(Callee)) {
 717:     Callee = DAG.getTargetExternalSymbol(
 718:         E->getSymbol(), getPointerTy(DAG.getDataLayout()), OpFlag);
 719:   }
 720: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 721-740

```cpp
 721:   // Returns a chain & a flag for retval copy to use.
 722:   SDVTList NodeTys = DAG.getVTList(MVT::Other, MVT::Glue);
 723:   SmallVector<SDValue, 8> Ops;
 724:   Ops.push_back(Chain);
 725:   Ops.push_back(Callee);
 726: 
 727:   // Add a register mask operand representing the call-preserved registers.
 728:   // TODO: Should return-twice functions be handled?
 729:   const uint32_t *Mask =
 730:       TRI->getCallPreservedMask(DAG.getMachineFunction(), CallConv);
 731:   assert(Mask && "Missing call preserved mask for calling convention");
 732:   Ops.push_back(DAG.getRegisterMask(Mask));
 733: 
 734:   // Add argument registers to the end of the list so that they are
 735:   // known live into the call.
 736:   for (const auto &[Reg, N] : RegsToPass)
 737:     Ops.push_back(DAG.getRegister(Reg, N.getValueType()));
 738: 
 739:   if (InGlue.getNode())
 740:     Ops.push_back(InGlue);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 741-760

```cpp
 741: 
 742:   Chain = DAG.getNode(LanaiISD::CALL, DL, NodeTys,
 743:                       ArrayRef<SDValue>(&Ops[0], Ops.size()));
 744:   InGlue = Chain.getValue(1);
 745: 
 746:   // Create the CALLSEQ_END node.
 747:   Chain = DAG.getCALLSEQ_END(Chain, NumBytes, 0, InGlue, DL);
 748:   InGlue = Chain.getValue(1);
 749: 
 750:   // Handle result values, copying them out of physregs into vregs that we
 751:   // return.
 752:   return LowerCallResult(Chain, InGlue, CallConv, IsVarArg, Ins, DL, DAG,
 753:                          InVals);
 754: }
 755: 
 756: // LowerCallResult - Lower the result values of a call into the
 757: // appropriate copies out of appropriate physical registers.
 758: SDValue LanaiTargetLowering::LowerCallResult(
 759:     SDValue Chain, SDValue InGlue, CallingConv::ID CallConv, bool IsVarArg,
 760:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &DL,
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 761-780

```cpp
 761:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
 762:   // Assign locations to each value returned by this call.
 763:   SmallVector<CCValAssign, 16> RVLocs;
 764:   CCState CCInfo(CallConv, IsVarArg, DAG.getMachineFunction(), RVLocs,
 765:                  *DAG.getContext());
 766: 
 767:   CCInfo.AnalyzeCallResult(Ins, RetCC_Lanai32);
 768: 
 769:   // Copy all of the result registers out of their specified physreg.
 770:   for (unsigned I = 0; I != RVLocs.size(); ++I) {
 771:     Chain = DAG.getCopyFromReg(Chain, DL, RVLocs[I].getLocReg(),
 772:                                RVLocs[I].getValVT(), InGlue)
 773:                 .getValue(1);
 774:     InGlue = Chain.getValue(2);
 775:     InVals.push_back(Chain.getValue(0));
 776:   }
 777: 
 778:   return Chain;
 779: }
 780: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 781-800

```cpp
 781: //===----------------------------------------------------------------------===//
 782: //                      Custom Lowerings
 783: //===----------------------------------------------------------------------===//
 784: 
 785: static LPCC::CondCode IntCondCCodeToICC(SDValue CC, const SDLoc &DL,
 786:                                         SDValue &RHS, SelectionDAG &DAG) {
 787:   ISD::CondCode SetCCOpcode = cast<CondCodeSDNode>(CC)->get();
 788: 
 789:   // For integer, only the SETEQ, SETNE, SETLT, SETLE, SETGT, SETGE, SETULT,
 790:   // SETULE, SETUGT, and SETUGE opcodes are used (see CodeGen/ISDOpcodes.h)
 791:   // and Lanai only supports integer comparisons, so only provide definitions
 792:   // for them.
 793:   switch (SetCCOpcode) {
 794:   case ISD::SETEQ:
 795:     return LPCC::ICC_EQ;
 796:   case ISD::SETGT:
 797:     if (ConstantSDNode *RHSC = dyn_cast<ConstantSDNode>(RHS))
 798:       if (RHSC->getZExtValue() == 0xFFFFFFFF) {
 799:         // X > -1 -> X >= 0 -> is_plus(X)
 800:         RHS = DAG.getConstant(0, DL, RHS.getValueType());
```

- EN: Function bodies or method definitions such as IntCondCCodeToICC, if contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: IntCondCCodeToICC, if 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 801-820

```cpp
 801:         return LPCC::ICC_PL;
 802:       }
 803:     return LPCC::ICC_GT;
 804:   case ISD::SETUGT:
 805:     return LPCC::ICC_UGT;
 806:   case ISD::SETLT:
 807:     if (ConstantSDNode *RHSC = dyn_cast<ConstantSDNode>(RHS))
 808:       if (RHSC->getZExtValue() == 0)
 809:         // X < 0 -> is_minus(X)
 810:         return LPCC::ICC_MI;
 811:     return LPCC::ICC_LT;
 812:   case ISD::SETULT:
 813:     return LPCC::ICC_ULT;
 814:   case ISD::SETLE:
 815:     if (ConstantSDNode *RHSC = dyn_cast<ConstantSDNode>(RHS))
 816:       if (RHSC->getZExtValue() == 0xFFFFFFFF) {
 817:         // X <= -1 -> X < 0 -> is_minus(X)
 818:         RHS = DAG.getConstant(0, DL, RHS.getValueType());
 819:         return LPCC::ICC_MI;
 820:       }
```

- EN: Function bodies or method definitions such as if contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: if 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 821-840

```cpp
 821:     return LPCC::ICC_LE;
 822:   case ISD::SETULE:
 823:     return LPCC::ICC_ULE;
 824:   case ISD::SETGE:
 825:     if (ConstantSDNode *RHSC = dyn_cast<ConstantSDNode>(RHS))
 826:       if (RHSC->getZExtValue() == 0)
 827:         // X >= 0 -> is_plus(X)
 828:         return LPCC::ICC_PL;
 829:     return LPCC::ICC_GE;
 830:   case ISD::SETUGE:
 831:     return LPCC::ICC_UGE;
 832:   case ISD::SETNE:
 833:     return LPCC::ICC_NE;
 834:   case ISD::SETONE:
 835:   case ISD::SETUNE:
 836:   case ISD::SETOGE:
 837:   case ISD::SETOLE:
 838:   case ISD::SETOLT:
 839:   case ISD::SETOGT:
 840:   case ISD::SETOEQ:
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 841-860

```cpp
 841:   case ISD::SETUEQ:
 842:   case ISD::SETO:
 843:   case ISD::SETUO:
 844:     llvm_unreachable("Unsupported comparison.");
 845:   default:
 846:     llvm_unreachable("Unknown integer condition code!");
 847:   }
 848: }
 849: 
 850: SDValue LanaiTargetLowering::LowerBR_CC(SDValue Op, SelectionDAG &DAG) const {
 851:   SDValue Chain = Op.getOperand(0);
 852:   SDValue Cond = Op.getOperand(1);
 853:   SDValue LHS = Op.getOperand(2);
 854:   SDValue RHS = Op.getOperand(3);
 855:   SDValue Dest = Op.getOperand(4);
 856:   SDLoc DL(Op);
 857: 
 858:   LPCC::CondCode CC = IntCondCCodeToICC(Cond, DL, RHS, DAG);
 859:   SDValue TargetCC = DAG.getConstant(CC, DL, MVT::i32);
 860:   SDValue Glue = DAG.getNode(LanaiISD::SET_FLAG, DL, MVT::Glue, LHS, RHS);
```

- EN: Function bodies or method definitions such as LowerBR_CC contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerBR_CC 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 861-880

```cpp
 861: 
 862:   return DAG.getNode(LanaiISD::BR_CC, DL, Op.getValueType(), Chain, Dest,
 863:                      TargetCC, Glue);
 864: }
 865: 
 866: SDValue LanaiTargetLowering::LowerMUL(SDValue Op, SelectionDAG &DAG) const {
 867:   EVT VT = Op->getValueType(0);
 868:   if (VT != MVT::i32)
 869:     return SDValue();
 870: 
 871:   ConstantSDNode *C = dyn_cast<ConstantSDNode>(Op->getOperand(1));
 872:   if (!C)
 873:     return SDValue();
 874: 
 875:   int64_t MulAmt = C->getSExtValue();
 876:   int32_t HighestOne = -1;
 877:   uint32_t NonzeroEntries = 0;
 878:   int SignedDigit[32] = {0};
 879: 
 880:   // Convert to non-adjacent form (NAF) signed-digit representation.
```

- EN: Function bodies or method definitions such as LowerMUL contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerMUL 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 881-900

```cpp
 881:   // NAF is a signed-digit form where no adjacent digits are non-zero. It is the
 882:   // minimal Hamming weight representation of a number (on average 1/3 of the
 883:   // digits will be non-zero vs 1/2 for regular binary representation). And as
 884:   // the non-zero digits will be the only digits contributing to the instruction
 885:   // count, this is desirable. The next loop converts it to NAF (following the
 886:   // approach in 'Guide to Elliptic Curve Cryptography' [ISBN: 038795273X]) by
 887:   // choosing the non-zero coefficients such that the resulting quotient is
 888:   // divisible by 2 which will cause the next coefficient to be zero.
 889:   int64_t E = std::abs(MulAmt);
 890:   int S = (MulAmt < 0 ? -1 : 1);
 891:   int I = 0;
 892:   while (E > 0) {
 893:     int ZI = 0;
 894:     if (E % 2 == 1) {
 895:       ZI = 2 - (E % 4);
 896:       if (ZI != 0)
 897:         ++NonzeroEntries;
 898:     }
 899:     SignedDigit[I] = S * ZI;
 900:     if (SignedDigit[I] == 1)
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 901-920

```cpp
 901:       HighestOne = I;
 902:     E = (E - ZI) / 2;
 903:     ++I;
 904:   }
 905: 
 906:   // Compute number of instructions required. Due to differences in lowering
 907:   // between the different processors this count is not exact.
 908:   // Start by assuming a shift and a add/sub for every non-zero entry (hence
 909:   // every non-zero entry requires 1 shift and 1 add/sub except for the first
 910:   // entry).
 911:   int32_t InstrRequired = 2 * NonzeroEntries - 1;
 912:   // Correct possible over-adding due to shift by 0 (which is not emitted).
 913:   if (std::abs(MulAmt) % 2 == 1)
 914:     --InstrRequired;
 915:   // Return if the form generated would exceed the instruction threshold.
 916:   if (InstrRequired > LanaiLowerConstantMulThreshold)
 917:     return SDValue();
 918: 
 919:   SDValue Res;
 920:   SDLoc DL(Op);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 921-940

```cpp
 921:   SDValue V = Op->getOperand(0);
 922: 
 923:   // Initialize the running sum. Set the running sum to the maximal shifted
 924:   // positive value (i.e., largest i such that zi == 1 and MulAmt has V<<i as a
 925:   // term NAF).
 926:   if (HighestOne == -1)
 927:     Res = DAG.getConstant(0, DL, MVT::i32);
 928:   else {
 929:     Res = DAG.getNode(ISD::SHL, DL, VT, V,
 930:                       DAG.getConstant(HighestOne, DL, MVT::i32));
 931:     SignedDigit[HighestOne] = 0;
 932:   }
 933: 
 934:   // Assemble multiplication from shift, add, sub using NAF form and running
 935:   // sum.
 936:   for (unsigned int I = 0; I < std::size(SignedDigit); ++I) {
 937:     if (SignedDigit[I] == 0)
 938:       continue;
 939: 
 940:     // Shifted multiplicand (v<<i).
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 941-960

```cpp
 941:     SDValue Op =
 942:         DAG.getNode(ISD::SHL, DL, VT, V, DAG.getConstant(I, DL, MVT::i32));
 943:     if (SignedDigit[I] == 1)
 944:       Res = DAG.getNode(ISD::ADD, DL, VT, Res, Op);
 945:     else if (SignedDigit[I] == -1)
 946:       Res = DAG.getNode(ISD::SUB, DL, VT, Res, Op);
 947:   }
 948:   return Res;
 949: }
 950: 
 951: SDValue LanaiTargetLowering::LowerSETCC(SDValue Op, SelectionDAG &DAG) const {
 952:   SDValue LHS = Op.getOperand(0);
 953:   SDValue RHS = Op.getOperand(1);
 954:   SDValue Cond = Op.getOperand(2);
 955:   SDLoc DL(Op);
 956: 
 957:   LPCC::CondCode CC = IntCondCCodeToICC(Cond, DL, RHS, DAG);
 958:   SDValue TargetCC = DAG.getConstant(CC, DL, MVT::i32);
 959:   SDValue Glue = DAG.getNode(LanaiISD::SET_FLAG, DL, MVT::Glue, LHS, RHS);
 960: 
```

- EN: Function bodies or method definitions such as LowerSETCC contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerSETCC 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 961-980

```cpp
 961:   return DAG.getNode(LanaiISD::SETCC, DL, Op.getValueType(), TargetCC, Glue);
 962: }
 963: 
 964: SDValue LanaiTargetLowering::LowerSELECT_CC(SDValue Op,
 965:                                             SelectionDAG &DAG) const {
 966:   SDValue LHS = Op.getOperand(0);
 967:   SDValue RHS = Op.getOperand(1);
 968:   SDValue TrueV = Op.getOperand(2);
 969:   SDValue FalseV = Op.getOperand(3);
 970:   SDValue Cond = Op.getOperand(4);
 971:   SDLoc DL(Op);
 972: 
 973:   LPCC::CondCode CC = IntCondCCodeToICC(Cond, DL, RHS, DAG);
 974:   SDValue TargetCC = DAG.getConstant(CC, DL, MVT::i32);
 975:   SDValue Glue = DAG.getNode(LanaiISD::SET_FLAG, DL, MVT::Glue, LHS, RHS);
 976: 
 977:   return DAG.getNode(LanaiISD::SELECT_CC, DL, Op.getValueType(), TrueV, FalseV,
 978:                      TargetCC, Glue);
 979: }
 980: 
```

- EN: Function bodies or method definitions such as LowerSELECT_CC contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerSELECT_CC 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 981-1000

```cpp
 981: SDValue LanaiTargetLowering::LowerVASTART(SDValue Op, SelectionDAG &DAG) const {
 982:   MachineFunction &MF = DAG.getMachineFunction();
 983:   LanaiMachineFunctionInfo *FuncInfo = MF.getInfo<LanaiMachineFunctionInfo>();
 984: 
 985:   SDLoc DL(Op);
 986:   SDValue FI = DAG.getFrameIndex(FuncInfo->getVarArgsFrameIndex(),
 987:                                  getPointerTy(DAG.getDataLayout()));
 988: 
 989:   // vastart just stores the address of the VarArgsFrameIndex slot into the
 990:   // memory location argument.
 991:   const Value *SV = cast<SrcValueSDNode>(Op.getOperand(2))->getValue();
 992:   return DAG.getStore(Op.getOperand(0), DL, FI, Op.getOperand(1),
 993:                       MachinePointerInfo(SV));
 994: }
 995: 
 996: SDValue LanaiTargetLowering::LowerDYNAMIC_STACKALLOC(SDValue Op,
 997:                                                      SelectionDAG &DAG) const {
 998:   SDValue Chain = Op.getOperand(0);
 999:   SDValue Size = Op.getOperand(1);
1000:   SDLoc DL(Op);
```

- EN: Function bodies or method definitions such as LowerVASTART, LowerDYNAMIC_STACKALLOC contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerVASTART, LowerDYNAMIC_STACKALLOC 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1001-1020

```cpp
1001: 
1002:   Register SPReg = getStackPointerRegisterToSaveRestore();
1003: 
1004:   // Get a reference to the stack pointer.
1005:   SDValue StackPointer = DAG.getCopyFromReg(Chain, DL, SPReg, MVT::i32);
1006: 
1007:   // Subtract the dynamic size from the actual stack size to
1008:   // obtain the new stack size.
1009:   SDValue Sub = DAG.getNode(ISD::SUB, DL, MVT::i32, StackPointer, Size);
1010: 
1011:   // For Lanai, the outgoing memory arguments area should be on top of the
1012:   // alloca area on the stack i.e., the outgoing memory arguments should be
1013:   // at a lower address than the alloca area. Move the alloca area down the
1014:   // stack by adding back the space reserved for outgoing arguments to SP
1015:   // here.
1016:   //
1017:   // We do not know what the size of the outgoing args is at this point.
1018:   // So, we add a pseudo instruction ADJDYNALLOC that will adjust the
1019:   // stack pointer. We replace this instruction with on that has the correct,
1020:   // known offset in emitPrologue().
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1021-1040

```cpp
1021:   SDValue ArgAdjust = DAG.getNode(LanaiISD::ADJDYNALLOC, DL, MVT::i32, Sub);
1022: 
1023:   // The Sub result contains the new stack start address, so it
1024:   // must be placed in the stack pointer register.
1025:   SDValue CopyChain = DAG.getCopyToReg(Chain, DL, SPReg, Sub);
1026: 
1027:   SDValue Ops[2] = {ArgAdjust, CopyChain};
1028:   return DAG.getMergeValues(Ops, DL);
1029: }
1030: 
1031: SDValue LanaiTargetLowering::LowerRETURNADDR(SDValue Op,
1032:                                              SelectionDAG &DAG) const {
1033:   MachineFunction &MF = DAG.getMachineFunction();
1034:   MachineFrameInfo &MFI = MF.getFrameInfo();
1035:   MFI.setReturnAddressIsTaken(true);
1036: 
1037:   EVT VT = Op.getValueType();
1038:   SDLoc DL(Op);
1039:   unsigned Depth = Op.getConstantOperandVal(0);
1040:   if (Depth) {
```

- EN: Function bodies or method definitions such as LowerRETURNADDR contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerRETURNADDR 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1041-1060

```cpp
1041:     SDValue FrameAddr = LowerFRAMEADDR(Op, DAG);
1042:     const unsigned Offset = -4;
1043:     SDValue Ptr = DAG.getNode(ISD::ADD, DL, VT, FrameAddr,
1044:                               DAG.getIntPtrConstant(Offset, DL));
1045:     return DAG.getLoad(VT, DL, DAG.getEntryNode(), Ptr, MachinePointerInfo());
1046:   }
1047: 
1048:   // Return the link register, which contains the return address.
1049:   // Mark it an implicit live-in.
1050:   Register Reg = MF.addLiveIn(TRI->getRARegister(), getRegClassFor(MVT::i32));
1051:   return DAG.getCopyFromReg(DAG.getEntryNode(), DL, Reg, VT);
1052: }
1053: 
1054: SDValue LanaiTargetLowering::LowerFRAMEADDR(SDValue Op,
1055:                                             SelectionDAG &DAG) const {
1056:   MachineFrameInfo &MFI = DAG.getMachineFunction().getFrameInfo();
1057:   MFI.setFrameAddressIsTaken(true);
1058: 
1059:   EVT VT = Op.getValueType();
1060:   SDLoc DL(Op);
```

- EN: Function bodies or method definitions such as LowerFRAMEADDR contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerFRAMEADDR 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1061-1080

```cpp
1061:   SDValue FrameAddr = DAG.getCopyFromReg(DAG.getEntryNode(), DL, Lanai::FP, VT);
1062:   unsigned Depth = Op.getConstantOperandVal(0);
1063:   while (Depth--) {
1064:     const unsigned Offset = -8;
1065:     SDValue Ptr = DAG.getNode(ISD::ADD, DL, VT, FrameAddr,
1066:                               DAG.getIntPtrConstant(Offset, DL));
1067:     FrameAddr =
1068:         DAG.getLoad(VT, DL, DAG.getEntryNode(), Ptr, MachinePointerInfo());
1069:   }
1070:   return FrameAddr;
1071: }
1072: 
1073: SDValue LanaiTargetLowering::LowerConstantPool(SDValue Op,
1074:                                                SelectionDAG &DAG) const {
1075:   SDLoc DL(Op);
1076:   ConstantPoolSDNode *N = cast<ConstantPoolSDNode>(Op);
1077:   const Constant *C = N->getConstVal();
1078:   const LanaiTargetObjectFile *TLOF =
1079:       static_cast<const LanaiTargetObjectFile *>(
1080:           getTargetMachine().getObjFileLowering());
```

- EN: Function bodies or method definitions such as LowerConstantPool contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerConstantPool 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1081-1100

```cpp
1081: 
1082:   // If the code model is small or constant will be placed in the small section,
1083:   // then assume address will fit in 21-bits.
1084:   if (getTargetMachine().getCodeModel() == CodeModel::Small ||
1085:       TLOF->isConstantInSmallSection(DAG.getDataLayout(), C)) {
1086:     SDValue Small = DAG.getTargetConstantPool(
1087:         C, MVT::i32, N->getAlign(), N->getOffset(), LanaiII::MO_NO_FLAG);
1088:     return DAG.getNode(ISD::OR, DL, MVT::i32,
1089:                        DAG.getRegister(Lanai::R0, MVT::i32),
1090:                        DAG.getNode(LanaiISD::SMALL, DL, MVT::i32, Small));
1091:   } else {
1092:     uint8_t OpFlagHi = LanaiII::MO_ABS_HI;
1093:     uint8_t OpFlagLo = LanaiII::MO_ABS_LO;
1094: 
1095:     SDValue Hi = DAG.getTargetConstantPool(C, MVT::i32, N->getAlign(),
1096:                                            N->getOffset(), OpFlagHi);
1097:     SDValue Lo = DAG.getTargetConstantPool(C, MVT::i32, N->getAlign(),
1098:                                            N->getOffset(), OpFlagLo);
1099:     Hi = DAG.getNode(LanaiISD::HI, DL, MVT::i32, Hi);
1100:     Lo = DAG.getNode(LanaiISD::LO, DL, MVT::i32, Lo);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1101-1120

```cpp
1101:     SDValue Result = DAG.getNode(ISD::OR, DL, MVT::i32, Hi, Lo);
1102:     return Result;
1103:   }
1104: }
1105: 
1106: SDValue LanaiTargetLowering::LowerGlobalAddress(SDValue Op,
1107:                                                 SelectionDAG &DAG) const {
1108:   SDLoc DL(Op);
1109:   const GlobalValue *GV = cast<GlobalAddressSDNode>(Op)->getGlobal();
1110:   int64_t Offset = cast<GlobalAddressSDNode>(Op)->getOffset();
1111: 
1112:   const LanaiTargetObjectFile *TLOF =
1113:       static_cast<const LanaiTargetObjectFile *>(
1114:           getTargetMachine().getObjFileLowering());
1115: 
1116:   // If the code model is small or global variable will be placed in the small
1117:   // section, then assume address will fit in 21-bits.
1118:   const GlobalObject *GO = GV->getAliaseeObject();
1119:   if (TLOF->isGlobalInSmallSection(GO, getTargetMachine())) {
1120:     SDValue Small = DAG.getTargetGlobalAddress(
```

- EN: Function bodies or method definitions such as LowerGlobalAddress contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerGlobalAddress 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1121-1140

```cpp
1121:         GV, DL, getPointerTy(DAG.getDataLayout()), Offset, LanaiII::MO_NO_FLAG);
1122:     return DAG.getNode(ISD::OR, DL, MVT::i32,
1123:                        DAG.getRegister(Lanai::R0, MVT::i32),
1124:                        DAG.getNode(LanaiISD::SMALL, DL, MVT::i32, Small));
1125:   } else {
1126:     uint8_t OpFlagHi = LanaiII::MO_ABS_HI;
1127:     uint8_t OpFlagLo = LanaiII::MO_ABS_LO;
1128: 
1129:     // Create the TargetGlobalAddress node, folding in the constant offset.
1130:     SDValue Hi = DAG.getTargetGlobalAddress(
1131:         GV, DL, getPointerTy(DAG.getDataLayout()), Offset, OpFlagHi);
1132:     SDValue Lo = DAG.getTargetGlobalAddress(
1133:         GV, DL, getPointerTy(DAG.getDataLayout()), Offset, OpFlagLo);
1134:     Hi = DAG.getNode(LanaiISD::HI, DL, MVT::i32, Hi);
1135:     Lo = DAG.getNode(LanaiISD::LO, DL, MVT::i32, Lo);
1136:     return DAG.getNode(ISD::OR, DL, MVT::i32, Hi, Lo);
1137:   }
1138: }
1139: 
1140: SDValue LanaiTargetLowering::LowerBlockAddress(SDValue Op,
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1141-1160

```cpp
1141:                                                SelectionDAG &DAG) const {
1142:   SDLoc DL(Op);
1143:   const BlockAddress *BA = cast<BlockAddressSDNode>(Op)->getBlockAddress();
1144: 
1145:   uint8_t OpFlagHi = LanaiII::MO_ABS_HI;
1146:   uint8_t OpFlagLo = LanaiII::MO_ABS_LO;
1147: 
1148:   SDValue Hi = DAG.getBlockAddress(BA, MVT::i32, true, OpFlagHi);
1149:   SDValue Lo = DAG.getBlockAddress(BA, MVT::i32, true, OpFlagLo);
1150:   Hi = DAG.getNode(LanaiISD::HI, DL, MVT::i32, Hi);
1151:   Lo = DAG.getNode(LanaiISD::LO, DL, MVT::i32, Lo);
1152:   SDValue Result = DAG.getNode(ISD::OR, DL, MVT::i32, Hi, Lo);
1153:   return Result;
1154: }
1155: 
1156: SDValue LanaiTargetLowering::LowerJumpTable(SDValue Op,
1157:                                             SelectionDAG &DAG) const {
1158:   SDLoc DL(Op);
1159:   JumpTableSDNode *JT = cast<JumpTableSDNode>(Op);
1160: 
```

- EN: Function bodies or method definitions such as LowerJumpTable contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerJumpTable 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1161-1180

```cpp
1161:   // If the code model is small assume address will fit in 21-bits.
1162:   if (getTargetMachine().getCodeModel() == CodeModel::Small) {
1163:     SDValue Small = DAG.getTargetJumpTable(
1164:         JT->getIndex(), getPointerTy(DAG.getDataLayout()), LanaiII::MO_NO_FLAG);
1165:     return DAG.getNode(ISD::OR, DL, MVT::i32,
1166:                        DAG.getRegister(Lanai::R0, MVT::i32),
1167:                        DAG.getNode(LanaiISD::SMALL, DL, MVT::i32, Small));
1168:   } else {
1169:     uint8_t OpFlagHi = LanaiII::MO_ABS_HI;
1170:     uint8_t OpFlagLo = LanaiII::MO_ABS_LO;
1171: 
1172:     SDValue Hi = DAG.getTargetJumpTable(
1173:         JT->getIndex(), getPointerTy(DAG.getDataLayout()), OpFlagHi);
1174:     SDValue Lo = DAG.getTargetJumpTable(
1175:         JT->getIndex(), getPointerTy(DAG.getDataLayout()), OpFlagLo);
1176:     Hi = DAG.getNode(LanaiISD::HI, DL, MVT::i32, Hi);
1177:     Lo = DAG.getNode(LanaiISD::LO, DL, MVT::i32, Lo);
1178:     SDValue Result = DAG.getNode(ISD::OR, DL, MVT::i32, Hi, Lo);
1179:     return Result;
1180:   }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1181-1200

```cpp
1181: }
1182: 
1183: SDValue LanaiTargetLowering::LowerSHL_PARTS(SDValue Op,
1184:                                             SelectionDAG &DAG) const {
1185:   EVT VT = Op.getValueType();
1186:   unsigned VTBits = VT.getSizeInBits();
1187:   SDLoc dl(Op);
1188:   assert(Op.getNumOperands() == 3 && "Unexpected SHL!");
1189:   SDValue ShOpLo = Op.getOperand(0);
1190:   SDValue ShOpHi = Op.getOperand(1);
1191:   SDValue ShAmt = Op.getOperand(2);
1192: 
1193:   // Performs the following for (ShOpLo + (ShOpHi << 32)) << ShAmt:
1194:   //   LoBitsForHi = (ShAmt == 0) ? 0 : (ShOpLo >> (32-ShAmt))
1195:   //   HiBitsForHi = ShOpHi << ShAmt
1196:   //   Hi = (ShAmt >= 32) ? (ShOpLo << (ShAmt-32)) : (LoBitsForHi | HiBitsForHi)
1197:   //   Lo = (ShAmt >= 32) ? 0 : (ShOpLo << ShAmt)
1198:   //   return (Hi << 32) | Lo;
1199: 
1200:   SDValue RevShAmt = DAG.getNode(ISD::SUB, dl, MVT::i32,
```

- EN: Function bodies or method definitions such as LowerSHL_PARTS contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerSHL_PARTS 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1201-1220

```cpp
1201:                                  DAG.getConstant(VTBits, dl, MVT::i32), ShAmt);
1202:   SDValue LoBitsForHi = DAG.getNode(ISD::SRL, dl, VT, ShOpLo, RevShAmt);
1203: 
1204:   // If ShAmt == 0, we just calculated "(SRL ShOpLo, 32)" which is "undef". We
1205:   // wanted 0, so CSEL it directly.
1206:   SDValue Zero = DAG.getConstant(0, dl, MVT::i32);
1207:   SDValue SetCC = DAG.getSetCC(dl, MVT::i32, ShAmt, Zero, ISD::SETEQ);
1208:   LoBitsForHi = DAG.getSelect(dl, MVT::i32, SetCC, Zero, LoBitsForHi);
1209: 
1210:   SDValue ExtraShAmt = DAG.getNode(ISD::SUB, dl, MVT::i32, ShAmt,
1211:                                    DAG.getConstant(VTBits, dl, MVT::i32));
1212:   SDValue HiBitsForHi = DAG.getNode(ISD::SHL, dl, VT, ShOpHi, ShAmt);
1213:   SDValue HiForNormalShift =
1214:       DAG.getNode(ISD::OR, dl, VT, LoBitsForHi, HiBitsForHi);
1215: 
1216:   SDValue HiForBigShift = DAG.getNode(ISD::SHL, dl, VT, ShOpLo, ExtraShAmt);
1217: 
1218:   SetCC = DAG.getSetCC(dl, MVT::i32, ExtraShAmt, Zero, ISD::SETGE);
1219:   SDValue Hi =
1220:       DAG.getSelect(dl, MVT::i32, SetCC, HiForBigShift, HiForNormalShift);
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1221-1240

```cpp
1221: 
1222:   // Lanai shifts of larger than register sizes are wrapped rather than
1223:   // clamped, so we can't just emit "lo << b" if b is too big.
1224:   SDValue LoForNormalShift = DAG.getNode(ISD::SHL, dl, VT, ShOpLo, ShAmt);
1225:   SDValue Lo = DAG.getSelect(
1226:       dl, MVT::i32, SetCC, DAG.getConstant(0, dl, MVT::i32), LoForNormalShift);
1227: 
1228:   SDValue Ops[2] = {Lo, Hi};
1229:   return DAG.getMergeValues(Ops, dl);
1230: }
1231: 
1232: SDValue LanaiTargetLowering::LowerSRL_PARTS(SDValue Op,
1233:                                             SelectionDAG &DAG) const {
1234:   MVT VT = Op.getSimpleValueType();
1235:   unsigned VTBits = VT.getSizeInBits();
1236:   SDLoc dl(Op);
1237:   SDValue ShOpLo = Op.getOperand(0);
1238:   SDValue ShOpHi = Op.getOperand(1);
1239:   SDValue ShAmt = Op.getOperand(2);
1240: 
```

- EN: Function bodies or method definitions such as LowerSRL_PARTS contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerSRL_PARTS 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1241-1260

```cpp
1241:   // Performs the following for a >> b:
1242:   //   unsigned r_high = a_high >> b;
1243:   //   r_high = (32 - b <= 0) ? 0 : r_high;
1244:   //
1245:   //   unsigned r_low = a_low >> b;
1246:   //   r_low = (32 - b <= 0) ? r_high : r_low;
1247:   //   r_low = (b == 0) ? r_low : r_low | (a_high << (32 - b));
1248:   //   return (unsigned long long)r_high << 32 | r_low;
1249:   // Note: This takes advantage of Lanai's shift behavior to avoid needing to
1250:   // mask the shift amount.
1251: 
1252:   SDValue Zero = DAG.getConstant(0, dl, MVT::i32);
1253:   SDValue NegatedPlus32 = DAG.getNode(
1254:       ISD::SUB, dl, MVT::i32, DAG.getConstant(VTBits, dl, MVT::i32), ShAmt);
1255:   SDValue SetCC = DAG.getSetCC(dl, MVT::i32, NegatedPlus32, Zero, ISD::SETLE);
1256: 
1257:   SDValue Hi = DAG.getNode(ISD::SRL, dl, MVT::i32, ShOpHi, ShAmt);
1258:   Hi = DAG.getSelect(dl, MVT::i32, SetCC, Zero, Hi);
1259: 
1260:   SDValue Lo = DAG.getNode(ISD::SRL, dl, MVT::i32, ShOpLo, ShAmt);
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1261-1280

```cpp
1261:   Lo = DAG.getSelect(dl, MVT::i32, SetCC, Hi, Lo);
1262:   SDValue CarryBits =
1263:       DAG.getNode(ISD::SHL, dl, MVT::i32, ShOpHi, NegatedPlus32);
1264:   SDValue ShiftIsZero = DAG.getSetCC(dl, MVT::i32, ShAmt, Zero, ISD::SETEQ);
1265:   Lo = DAG.getSelect(dl, MVT::i32, ShiftIsZero, Lo,
1266:                      DAG.getNode(ISD::OR, dl, MVT::i32, Lo, CarryBits));
1267: 
1268:   SDValue Ops[2] = {Lo, Hi};
1269:   return DAG.getMergeValues(Ops, dl);
1270: }
1271: 
1272: // Helper function that checks if N is a null or all ones constant.
1273: static inline bool isZeroOrAllOnes(SDValue N, bool AllOnes) {
1274:   return AllOnes ? isAllOnesConstant(N) : isNullConstant(N);
1275: }
1276: 
1277: // Return true if N is conditionally 0 or all ones.
1278: // Detects these expressions where cc is an i1 value:
1279: //
1280: //   (select cc 0, y)   [AllOnes=0]
```

- EN: Function bodies or method definitions such as isZeroOrAllOnes contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: isZeroOrAllOnes 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1281-1300

```cpp
1281: //   (select cc y, 0)   [AllOnes=0]
1282: //   (zext cc)          [AllOnes=0]
1283: //   (sext cc)          [AllOnes=0/1]
1284: //   (select cc -1, y)  [AllOnes=1]
1285: //   (select cc y, -1)  [AllOnes=1]
1286: //
1287: // * AllOnes determines whether to check for an all zero (AllOnes false) or an
1288: //   all ones operand (AllOnes true).
1289: // * Invert is set when N is the all zero/ones constant when CC is false.
1290: // * OtherOp is set to the alternative value of N.
1291: //
1292: // For example, for (select cc X, Y) and AllOnes = 0 if:
1293: // * X = 0, Invert = False and OtherOp = Y
1294: // * Y = 0, Invert = True and OtherOp = X
1295: static bool isConditionalZeroOrAllOnes(SDNode *N, bool AllOnes, SDValue &CC,
1296:                                        bool &Invert, SDValue &OtherOp,
1297:                                        SelectionDAG &DAG) {
1298:   switch (N->getOpcode()) {
1299:   default:
1300:     return false;
```

- EN: Function bodies or method definitions such as isConditionalZeroOrAllOnes contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: isConditionalZeroOrAllOnes 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1301-1320

```cpp
1301:   case ISD::SELECT: {
1302:     CC = N->getOperand(0);
1303:     SDValue N1 = N->getOperand(1);
1304:     SDValue N2 = N->getOperand(2);
1305:     if (isZeroOrAllOnes(N1, AllOnes)) {
1306:       Invert = false;
1307:       OtherOp = N2;
1308:       return true;
1309:     }
1310:     if (isZeroOrAllOnes(N2, AllOnes)) {
1311:       Invert = true;
1312:       OtherOp = N1;
1313:       return true;
1314:     }
1315:     return false;
1316:   }
1317:   case ISD::ZERO_EXTEND: {
1318:     // (zext cc) can never be the all ones value.
1319:     if (AllOnes)
1320:       return false;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1321-1340

```cpp
1321:     CC = N->getOperand(0);
1322:     if (CC.getValueType() != MVT::i1)
1323:       return false;
1324:     SDLoc dl(N);
1325:     EVT VT = N->getValueType(0);
1326:     OtherOp = DAG.getConstant(1, dl, VT);
1327:     Invert = true;
1328:     return true;
1329:   }
1330:   case ISD::SIGN_EXTEND: {
1331:     CC = N->getOperand(0);
1332:     if (CC.getValueType() != MVT::i1)
1333:       return false;
1334:     SDLoc dl(N);
1335:     EVT VT = N->getValueType(0);
1336:     Invert = !AllOnes;
1337:     if (AllOnes)
1338:       // When looking for an AllOnes constant, N is an sext, and the 'other'
1339:       // value is 0.
1340:       OtherOp = DAG.getConstant(0, dl, VT);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1341-1360

```cpp
1341:     else
1342:       OtherOp = DAG.getAllOnesConstant(dl, VT);
1343:     return true;
1344:   }
1345:   }
1346: }
1347: 
1348: // Combine a constant select operand into its use:
1349: //
1350: //   (add (select cc, 0, c), x)  -> (select cc, x, (add, x, c))
1351: //   (sub x, (select cc, 0, c))  -> (select cc, x, (sub, x, c))
1352: //   (and (select cc, -1, c), x) -> (select cc, x, (and, x, c))  [AllOnes=1]
1353: //   (or  (select cc, 0, c), x)  -> (select cc, x, (or, x, c))
1354: //   (xor (select cc, 0, c), x)  -> (select cc, x, (xor, x, c))
1355: //
1356: // The transform is rejected if the select doesn't have a constant operand that
1357: // is null, or all ones when AllOnes is set.
1358: //
1359: // Also recognize sext/zext from i1:
1360: //
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1361-1380

```cpp
1361: //   (add (zext cc), x) -> (select cc (add x, 1), x)
1362: //   (add (sext cc), x) -> (select cc (add x, -1), x)
1363: //
1364: // These transformations eventually create predicated instructions.
1365: static SDValue combineSelectAndUse(SDNode *N, SDValue Slct, SDValue OtherOp,
1366:                                    TargetLowering::DAGCombinerInfo &DCI,
1367:                                    bool AllOnes) {
1368:   SelectionDAG &DAG = DCI.DAG;
1369:   EVT VT = N->getValueType(0);
1370:   SDValue NonConstantVal;
1371:   SDValue CCOp;
1372:   bool SwapSelectOps;
1373:   if (!isConditionalZeroOrAllOnes(Slct.getNode(), AllOnes, CCOp, SwapSelectOps,
1374:                                   NonConstantVal, DAG))
1375:     return SDValue();
1376: 
1377:   // Slct is now know to be the desired identity constant when CC is true.
1378:   SDValue TrueVal = OtherOp;
1379:   SDValue FalseVal =
1380:       DAG.getNode(N->getOpcode(), SDLoc(N), VT, OtherOp, NonConstantVal);
```

- EN: Function bodies or method definitions such as combineSelectAndUse contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: combineSelectAndUse 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1381-1400

```cpp
1381:   // Unless SwapSelectOps says CC should be false.
1382:   if (SwapSelectOps)
1383:     std::swap(TrueVal, FalseVal);
1384: 
1385:   return DAG.getNode(ISD::SELECT, SDLoc(N), VT, CCOp, TrueVal, FalseVal);
1386: }
1387: 
1388: // Attempt combineSelectAndUse on each operand of a commutative operator N.
1389: static SDValue
1390: combineSelectAndUseCommutative(SDNode *N, TargetLowering::DAGCombinerInfo &DCI,
1391:                                bool AllOnes) {
1392:   SDValue N0 = N->getOperand(0);
1393:   SDValue N1 = N->getOperand(1);
1394:   if (N0.getNode()->hasOneUse())
1395:     if (SDValue Result = combineSelectAndUse(N, N0, N1, DCI, AllOnes))
1396:       return Result;
1397:   if (N1.getNode()->hasOneUse())
1398:     if (SDValue Result = combineSelectAndUse(N, N1, N0, DCI, AllOnes))
1399:       return Result;
1400:   return SDValue();
```

- EN: Function bodies or method definitions such as combineSelectAndUseCommutative contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: combineSelectAndUseCommutative 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1401-1420

```cpp
1401: }
1402: 
1403: // PerformSUBCombine - Target-specific dag combine xforms for ISD::SUB.
1404: static SDValue PerformSUBCombine(SDNode *N,
1405:                                  TargetLowering::DAGCombinerInfo &DCI) {
1406:   SDValue N0 = N->getOperand(0);
1407:   SDValue N1 = N->getOperand(1);
1408: 
1409:   // fold (sub x, (select cc, 0, c)) -> (select cc, x, (sub, x, c))
1410:   if (N1.getNode()->hasOneUse())
1411:     if (SDValue Result = combineSelectAndUse(N, N1, N0, DCI, /*AllOnes=*/false))
1412:       return Result;
1413: 
1414:   return SDValue();
1415: }
1416: 
1417: SDValue LanaiTargetLowering::PerformDAGCombine(SDNode *N,
1418:                                                DAGCombinerInfo &DCI) const {
1419:   switch (N->getOpcode()) {
1420:   default:
```

- EN: Function bodies or method definitions such as PerformSUBCombine, PerformDAGCombine contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: PerformSUBCombine, PerformDAGCombine 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1421-1440

```cpp
1421:     break;
1422:   case ISD::ADD:
1423:   case ISD::OR:
1424:   case ISD::XOR:
1425:     return combineSelectAndUseCommutative(N, DCI, /*AllOnes=*/false);
1426:   case ISD::AND:
1427:     return combineSelectAndUseCommutative(N, DCI, /*AllOnes=*/true);
1428:   case ISD::SUB:
1429:     return PerformSUBCombine(N, DCI);
1430:   }
1431: 
1432:   return SDValue();
1433: }
1434: 
1435: void LanaiTargetLowering::computeKnownBitsForTargetNode(
1436:     const SDValue Op, KnownBits &Known, const APInt &DemandedElts,
1437:     const SelectionDAG &DAG, unsigned Depth) const {
1438:   unsigned BitWidth = Known.getBitWidth();
1439:   switch (Op.getOpcode()) {
1440:   default:
```

- EN: Function bodies or method definitions such as computeKnownBitsForTargetNode contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: computeKnownBitsForTargetNode 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1441-1453

```cpp
1441:     break;
1442:   case LanaiISD::SETCC:
1443:     Known = KnownBits(BitWidth);
1444:     Known.Zero.setBits(1, BitWidth);
1445:     break;
1446:   case LanaiISD::SELECT_CC:
1447:     KnownBits Known2;
1448:     Known = DAG.computeKnownBits(Op->getOperand(0), Depth + 1);
1449:     Known2 = DAG.computeKnownBits(Op->getOperand(1), Depth + 1);
1450:     Known = Known.intersectWith(Known2);
1451:     break;
1452:   }
1453: }
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

## Key Concepts / 关键概念

- Custom DAG legalization / 自定义 DAG 合法化
- Calling-lowering hooks / 调用降级钩子
- SelectionDAG lowering / SelectionDAG 降级
- MachineFunction state / MachineFunction 状态
- Register classes / 寄存器类
- IR to target lowering / IR 到目标降级
- Target machine configuration / 目标机器配置
- CPU feature modelling / CPU 特性建模

## Dependencies / 依赖关系

- Direct includes / 直接包含: `LanaiISelLowering.h`, `LanaiCondCode.h`, `LanaiMachineFunctionInfo.h`, `LanaiSubtarget.h`, `LanaiTargetObjectFile.h`, `MCTargetDesc/LanaiBaseInfo.h`, `llvm/ADT/APInt.h`, `llvm/ADT/ArrayRef.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, SelectionDAG
- Generated or companion files / 生成或配套文件: `LanaiGenCallingConv.inc`
- Local companions / 本地配套文件: `LanaiISelLowering.h`
