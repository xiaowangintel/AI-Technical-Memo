# SystemZISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/SystemZ/SystemZISelLowering.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **用途 (CN)**: 该文件用于 SystemZ 后端，负责实现目标后端的 SelectionDAG 降低。

## Line-by-Line Analysis / 逐行分析
### Lines 1-75 / 第 1-75 行
```cpp
   1: //===-- SystemZISelLowering.cpp - SystemZ DAG lowering implementation -----===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file implements the SystemZTargetLowering class.
  10: //
  11: //===----------------------------------------------------------------------===//
  12: 
  13: #include "SystemZISelLowering.h"
  14: #include "SystemZCallingConv.h"
  15: #include "SystemZConstantPoolValue.h"
  16: #include "SystemZMachineFunctionInfo.h"
  17: #include "SystemZTargetMachine.h"
  18: #include "llvm/ADT/SmallSet.h"
  19: #include "llvm/CodeGen/CallingConvLower.h"
  20: #include "llvm/CodeGen/ISDOpcodes.h"
  21: #include "llvm/CodeGen/MachineInstrBuilder.h"
  22: #include "llvm/CodeGen/MachineRegisterInfo.h"
  23: #include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
  24: #include "llvm/IR/GlobalAlias.h"
  25: #include "llvm/IR/IntrinsicInst.h"
  26: #include "llvm/IR/Intrinsics.h"
  27: #include "llvm/IR/IntrinsicsS390.h"
  28: #include "llvm/IR/PatternMatch.h"
  29: #include "llvm/Support/CommandLine.h"
  30: #include "llvm/Support/ErrorHandling.h"
  31: #include "llvm/Support/KnownBits.h"
  32: #include <cctype>
  33: #include <optional>
  34: 
  35: using namespace llvm;
  36: 
  37: #define DEBUG_TYPE "systemz-lower"
  38: 
  39: // Temporarily let this be disabled by default until all known problems
  40: // related to argument extensions are fixed.
  41: static cl::opt<bool> EnableIntArgExtCheck(
  42:     "argext-abi-check", cl::init(false),
  43:     cl::desc("Verify that narrow int args are properly extended per the "
  44:              "SystemZ ABI."));
  45: 
  46: namespace {
  47: // Represents information about a comparison.
  48: struct Comparison {
  49:   Comparison(SDValue Op0In, SDValue Op1In, SDValue ChainIn)
  50:     : Op0(Op0In), Op1(Op1In), Chain(ChainIn),
  51:       Opcode(0), ICmpType(0), CCValid(0), CCMask(0) {}
  52: 
  53:   // The operands to the comparison.
  54:   SDValue Op0, Op1;
  55: 
  56:   // Chain if this is a strict floating-point comparison.
  57:   SDValue Chain;
  58: 
  59:   // The opcode that should be used to compare Op0 and Op1.
  60:   unsigned Opcode;
  61: 
  62:   // A SystemZICMP value.  Only used for integer comparisons.
  63:   unsigned ICmpType;
  64: 
  65:   // The mask of CC values that Opcode can produce.
  66:   unsigned CCValid;
  67: 
  68:   // The mask of CC values for which the original condition is true.
  69:   unsigned CCMask;
  70: };
  71: } // end anonymous namespace
  72: 
  73: // Classify VT as either 32 or 64 bit.
  74: static bool is32Bit(EVT VT) {
  75:   switch (VT.getSimpleVT().SimpleTy) {
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `SystemZISelLowering.h`, `SystemZCallingConv.h`, `SystemZConstantPoolValue.h`, `SystemZMachineFunctionInfo.h`, `SystemZTargetMachine.h`, `SmallSet.h` that expose the LLVM and target interfaces used in later logic. It introduces interface types such as `Comparison`, shaping how other backend components interact with this file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `SystemZISelLowering.h`, `SystemZCallingConv.h`, `SystemZConstantPoolValue.h`, `SystemZMachineFunctionInfo.h`, `SystemZTargetMachine.h`, `SmallSet.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 它引入了 `Comparison` 等接口类型，定义了其他后端组件与本文件交互的方式。

### Lines 76-150 / 第 76-150 行
```cpp
  76:   case MVT::i32:
  77:     return true;
  78:   case MVT::i64:
  79:     return false;
  80:   default:
  81:     llvm_unreachable("Unsupported type");
  82:   }
  83: }
  84: 
  85: // Return a version of MachineOperand that can be safely used before the
  86: // final use.
  87: static MachineOperand earlyUseOperand(MachineOperand Op) {
  88:   if (Op.isReg())
  89:     Op.setIsKill(false);
  90:   return Op;
  91: }
  92: 
  93: SystemZTargetLowering::SystemZTargetLowering(const TargetMachine &TM,
  94:                                              const SystemZSubtarget &STI)
  95:     : TargetLowering(TM, STI), Subtarget(STI) {
  96:   MVT PtrVT = MVT::getIntegerVT(TM.getPointerSizeInBits(0));
  97: 
  98:   auto *Regs = STI.getSpecialRegisters();
  99: 
 100:   // Set up the register classes.
 101:   if (Subtarget.hasHighWord())
 102:     addRegisterClass(MVT::i32, &SystemZ::GRX32BitRegClass);
 103:   else
 104:     addRegisterClass(MVT::i32, &SystemZ::GR32BitRegClass);
 105:   addRegisterClass(MVT::i64, &SystemZ::GR64BitRegClass);
 106:   if (!useSoftFloat()) {
 107:     if (Subtarget.hasVector()) {
 108:       addRegisterClass(MVT::f16, &SystemZ::VR16BitRegClass);
 109:       addRegisterClass(MVT::f32, &SystemZ::VR32BitRegClass);
 110:       addRegisterClass(MVT::f64, &SystemZ::VR64BitRegClass);
 111:     } else {
 112:       addRegisterClass(MVT::f16, &SystemZ::FP16BitRegClass);
 113:       addRegisterClass(MVT::f32, &SystemZ::FP32BitRegClass);
 114:       addRegisterClass(MVT::f64, &SystemZ::FP64BitRegClass);
 115:     }
 116:     if (Subtarget.hasVectorEnhancements1())
 117:       addRegisterClass(MVT::f128, &SystemZ::VR128BitRegClass);
 118:     else
 119:       addRegisterClass(MVT::f128, &SystemZ::FP128BitRegClass);
 120: 
 121:     if (Subtarget.hasVector()) {
 122:       addRegisterClass(MVT::v16i8, &SystemZ::VR128BitRegClass);
 123:       addRegisterClass(MVT::v8i16, &SystemZ::VR128BitRegClass);
 124:       addRegisterClass(MVT::v4i32, &SystemZ::VR128BitRegClass);
 125:       addRegisterClass(MVT::v2i64, &SystemZ::VR128BitRegClass);
 126:       addRegisterClass(MVT::v8f16, &SystemZ::VR128BitRegClass);
 127:       addRegisterClass(MVT::v4f32, &SystemZ::VR128BitRegClass);
 128:       addRegisterClass(MVT::v2f64, &SystemZ::VR128BitRegClass);
 129:     }
 130: 
 131:     if (Subtarget.hasVector())
 132:       addRegisterClass(MVT::i128, &SystemZ::VR128BitRegClass);
 133:   }
 134: 
 135:   // Compute derived properties from the register classes
 136:   computeRegisterProperties(Subtarget.getRegisterInfo());
 137: 
 138:   // Set up special registers.
 139:   setStackPointerRegisterToSaveRestore(Regs->getStackPointerRegister());
 140: 
 141:   // TODO: It may be better to default to latency-oriented scheduling, however
 142:   // LLVM's current latency-oriented scheduler can't handle physreg definitions
 143:   // such as SystemZ has with CC, so set this to the register-pressure
 144:   // scheduler, because it can.
 145:   setSchedulingPreference(Sched::RegPressure);
 146: 
 147:   setBooleanContents(ZeroOrOneBooleanContent);
 148:   setBooleanVectorContents(ZeroOrNegativeOneBooleanContent);
 149: 
 150:   setMaxAtomicSizeInBitsSupported(128);
```
- **EN**: The range implements or declares functions including `earlyUseOperand`, `TargetLowering`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `earlyUseOperand`, `TargetLowering` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 151-225 / 第 151-225 行
```cpp
 151: 
 152:   // Instructions are strings of 2-byte aligned 2-byte values.
 153:   setMinFunctionAlignment(Align(2));
 154:   // For performance reasons we prefer 16-byte alignment.
 155:   setPrefFunctionAlignment(Align(16));
 156: 
 157:   // Handle operations that are handled in a similar way for all types.
 158:   for (unsigned I = MVT::FIRST_INTEGER_VALUETYPE;
 159:        I <= MVT::LAST_FP_VALUETYPE;
 160:        ++I) {
 161:     MVT VT = MVT::SimpleValueType(I);
 162:     if (isTypeLegal(VT)) {
 163:       // Lower SET_CC into an IPM-based sequence.
 164:       setOperationAction(ISD::SETCC, VT, Custom);
 165:       setOperationAction(ISD::STRICT_FSETCC, VT, Custom);
 166:       setOperationAction(ISD::STRICT_FSETCCS, VT, Custom);
 167: 
 168:       // Expand SELECT(C, A, B) into SELECT_CC(X, 0, A, B, NE).
 169:       setOperationAction(ISD::SELECT, VT, Expand);
 170: 
 171:       // Lower SELECT_CC and BR_CC into separate comparisons and branches.
 172:       setOperationAction(ISD::SELECT_CC, VT, Custom);
 173:       setOperationAction(ISD::BR_CC,     VT, Custom);
 174:     }
 175:   }
 176: 
 177:   // Expand jump table branches as address arithmetic followed by an
 178:   // indirect jump.
 179:   setOperationAction(ISD::BR_JT, MVT::Other, Expand);
 180: 
 181:   // Expand BRCOND into a BR_CC (see above).
 182:   setOperationAction(ISD::BRCOND, MVT::Other, Expand);
 183: 
 184:   // Handle integer types except i128.
 185:   for (unsigned I = MVT::FIRST_INTEGER_VALUETYPE;
 186:        I <= MVT::LAST_INTEGER_VALUETYPE;
 187:        ++I) {
 188:     MVT VT = MVT::SimpleValueType(I);
 189:     if (isTypeLegal(VT) && VT != MVT::i128) {
 190:       setOperationAction(ISD::ABS, VT, Legal);
 191: 
 192:       // Expand individual DIV and REMs into DIVREMs.
 193:       setOperationAction(ISD::SDIV, VT, Expand);
 194:       setOperationAction(ISD::UDIV, VT, Expand);
 195:       setOperationAction(ISD::SREM, VT, Expand);
 196:       setOperationAction(ISD::UREM, VT, Expand);
 197:       setOperationAction(ISD::SDIVREM, VT, Custom);
 198:       setOperationAction(ISD::UDIVREM, VT, Custom);
 199: 
 200:       // Support addition/subtraction with overflow.
 201:       setOperationAction(ISD::SADDO, VT, Custom);
 202:       setOperationAction(ISD::SSUBO, VT, Custom);
 203: 
 204:       // Support addition/subtraction with carry.
 205:       setOperationAction(ISD::UADDO, VT, Custom);
 206:       setOperationAction(ISD::USUBO, VT, Custom);
 207: 
 208:       // Support carry in as value rather than glue.
 209:       setOperationAction(ISD::UADDO_CARRY, VT, Custom);
 210:       setOperationAction(ISD::USUBO_CARRY, VT, Custom);
 211: 
 212:       // Lower ATOMIC_LOAD_SUB into ATOMIC_LOAD_ADD if LAA and LAAG are
 213:       // available, or if the operand is constant.
 214:       setOperationAction(ISD::ATOMIC_LOAD_SUB, VT, Custom);
 215: 
 216:       // Use POPCNT on z196 and above.
 217:       if (Subtarget.hasPopulationCount())
 218:         setOperationAction(ISD::CTPOP, VT, Custom);
 219:       else
 220:         setOperationAction(ISD::CTPOP, VT, Expand);
 221: 
 222:       // No special instructions for these.
 223:       setOperationAction(ISD::CTTZ,            VT, Expand);
 224:       setOperationAction(ISD::ROTR,            VT, Expand);
 225: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 226-300 / 第 226-300 行
```cpp
 226:       // Use *MUL_LOHI where possible instead of MULH*.
 227:       setOperationAction(ISD::MULHS, VT, Expand);
 228:       setOperationAction(ISD::MULHU, VT, Expand);
 229:       setOperationAction(ISD::SMUL_LOHI, VT, Custom);
 230:       setOperationAction(ISD::UMUL_LOHI, VT, Custom);
 231: 
 232:       // The fp<=>i32/i64 conversions are all Legal except for f16 and for
 233:       // unsigned on z10 (only z196 and above have native support for
 234:       // unsigned conversions).
 235:       for (auto Op : {ISD::FP_TO_SINT, ISD::STRICT_FP_TO_SINT,
 236:                       ISD::SINT_TO_FP, ISD::STRICT_SINT_TO_FP})
 237:         setOperationAction(Op, VT, Custom);
 238:       for (auto Op : {ISD::FP_TO_UINT, ISD::STRICT_FP_TO_UINT})
 239:         setOperationAction(Op, VT, Custom);
 240:       for (auto Op : {ISD::UINT_TO_FP, ISD::STRICT_UINT_TO_FP}) {
 241:         // Handle unsigned 32-bit input types as signed 64-bit types on z10.
 242:         auto OpAction =
 243:             (!Subtarget.hasFPExtension() && VT == MVT::i32) ? Promote : Custom;
 244:         setOperationAction(Op, VT, OpAction);
 245:       }
 246:     }
 247:   }
 248: 
 249:   // Handle i128 if legal.
 250:   if (isTypeLegal(MVT::i128)) {
 251:     // No special instructions for these.
 252:     setOperationAction(ISD::SDIVREM,   MVT::i128, Expand);
 253:     setOperationAction(ISD::UDIVREM,   MVT::i128, Expand);
 254:     setOperationAction(ISD::SMUL_LOHI, MVT::i128, Expand);
 255:     setOperationAction(ISD::UMUL_LOHI, MVT::i128, Expand);
 256:     setOperationAction(ISD::ROTR,      MVT::i128, Expand);
 257:     setOperationAction(ISD::ROTL,      MVT::i128, Expand);
 258: 
 259:     // We may be able to use VSLDB/VSLD/VSRD for these.
 260:     setOperationAction(ISD::FSHL,      MVT::i128, Custom);
 261:     setOperationAction(ISD::FSHR,      MVT::i128, Custom);
 262: 
 263:     // No special instructions for these before z17.
 264:     if (!Subtarget.hasVectorEnhancements3()) {
 265:       setOperationAction(ISD::MUL,   MVT::i128, Expand);
 266:       setOperationAction(ISD::MULHS, MVT::i128, Expand);
 267:       setOperationAction(ISD::MULHU, MVT::i128, Expand);
 268:       setOperationAction(ISD::SDIV,  MVT::i128, Expand);
 269:       setOperationAction(ISD::UDIV,  MVT::i128, Expand);
 270:       setOperationAction(ISD::SREM,  MVT::i128, Expand);
 271:       setOperationAction(ISD::UREM,  MVT::i128, Expand);
 272:       setOperationAction(ISD::CTLZ,  MVT::i128, Expand);
 273:       setOperationAction(ISD::CTTZ,  MVT::i128, Expand);
 274:     } else {
 275:       // Even if we do have a legal 128-bit multiply, we do not
 276:       // want 64-bit multiply-high operations to use it.
 277:       setOperationAction(ISD::MULHS, MVT::i64, Custom);
 278:       setOperationAction(ISD::MULHU, MVT::i64, Custom);
 279:     }
 280: 
 281:     // Support addition/subtraction with carry.
 282:     setOperationAction(ISD::UADDO, MVT::i128, Custom);
 283:     setOperationAction(ISD::USUBO, MVT::i128, Custom);
 284:     setOperationAction(ISD::UADDO_CARRY, MVT::i128, Custom);
 285:     setOperationAction(ISD::USUBO_CARRY, MVT::i128, Custom);
 286: 
 287:     // Use VPOPCT and add up partial results.
 288:     setOperationAction(ISD::CTPOP, MVT::i128, Custom);
 289: 
 290:     // Additional instructions available with z17.
 291:     if (Subtarget.hasVectorEnhancements3()) {
 292:       setOperationAction(ISD::ABS, MVT::i128, Legal);
 293: 
 294:       setOperationAction({ISD::SMIN, ISD::UMIN, ISD::SMAX, ISD::UMAX},
 295:                          MVT::i128, Legal);
 296:     }
 297:   }
 298: 
 299:   // These need custom handling in order to handle the f16 conversions.
 300:   setOperationAction(ISD::FP_TO_UINT, MVT::i128, Custom);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 301-375 / 第 301-375 行
```cpp
 301:   setOperationAction(ISD::FP_TO_SINT, MVT::i128, Custom);
 302:   setOperationAction(ISD::UINT_TO_FP, MVT::i128, Custom);
 303:   setOperationAction(ISD::SINT_TO_FP, MVT::i128, Custom);
 304:   setOperationAction(ISD::STRICT_FP_TO_UINT, MVT::i128, Custom);
 305:   setOperationAction(ISD::STRICT_FP_TO_SINT, MVT::i128, Custom);
 306:   setOperationAction(ISD::STRICT_UINT_TO_FP, MVT::i128, Custom);
 307:   setOperationAction(ISD::STRICT_SINT_TO_FP, MVT::i128, Custom);
 308: 
 309:   // Type legalization will convert 8- and 16-bit atomic operations into
 310:   // forms that operate on i32s (but still keeping the original memory VT).
 311:   // Lower them into full i32 operations.
 312:   setOperationAction(ISD::ATOMIC_SWAP,      MVT::i32, Custom);
 313:   setOperationAction(ISD::ATOMIC_LOAD_ADD,  MVT::i32, Custom);
 314:   setOperationAction(ISD::ATOMIC_LOAD_SUB,  MVT::i32, Custom);
 315:   setOperationAction(ISD::ATOMIC_LOAD_AND,  MVT::i32, Custom);
 316:   setOperationAction(ISD::ATOMIC_LOAD_OR,   MVT::i32, Custom);
 317:   setOperationAction(ISD::ATOMIC_LOAD_XOR,  MVT::i32, Custom);
 318:   setOperationAction(ISD::ATOMIC_LOAD_NAND, MVT::i32, Custom);
 319:   setOperationAction(ISD::ATOMIC_LOAD_MIN,  MVT::i32, Custom);
 320:   setOperationAction(ISD::ATOMIC_LOAD_MAX,  MVT::i32, Custom);
 321:   setOperationAction(ISD::ATOMIC_LOAD_UMIN, MVT::i32, Custom);
 322:   setOperationAction(ISD::ATOMIC_LOAD_UMAX, MVT::i32, Custom);
 323: 
 324:   // Whether or not i128 is not a legal type, we need to custom lower
 325:   // the atomic operations in order to exploit SystemZ instructions.
 326:   setOperationAction(ISD::ATOMIC_LOAD,     MVT::i128, Custom);
 327:   setOperationAction(ISD::ATOMIC_STORE,    MVT::i128, Custom);
 328:   setOperationAction(ISD::ATOMIC_LOAD, MVT::f128, Custom);
 329:   setOperationAction(ISD::ATOMIC_STORE, MVT::f128, Custom);
 330: 
 331:   // Mark sign/zero extending atomic loads as legal, which will make
 332:   // DAGCombiner fold extensions into atomic loads if possible.
 333:   setAtomicLoadExtAction({ISD::SEXTLOAD, ISD::ZEXTLOAD}, MVT::i64,
 334:                          {MVT::i8, MVT::i16, MVT::i32}, Legal);
 335:   setAtomicLoadExtAction({ISD::SEXTLOAD, ISD::ZEXTLOAD}, MVT::i32,
 336:                          {MVT::i8, MVT::i16}, Legal);
 337:   setAtomicLoadExtAction({ISD::SEXTLOAD, ISD::ZEXTLOAD}, MVT::i16,
 338:                          MVT::i8, Legal);
 339: 
 340:   // We can use the CC result of compare-and-swap to implement
 341:   // the "success" result of ATOMIC_CMP_SWAP_WITH_SUCCESS.
 342:   setOperationAction(ISD::ATOMIC_CMP_SWAP_WITH_SUCCESS, MVT::i32, Custom);
 343:   setOperationAction(ISD::ATOMIC_CMP_SWAP_WITH_SUCCESS, MVT::i64, Custom);
 344:   setOperationAction(ISD::ATOMIC_CMP_SWAP_WITH_SUCCESS, MVT::i128, Custom);
 345: 
 346:   setOperationAction(ISD::ATOMIC_FENCE, MVT::Other, Custom);
 347: 
 348:   // Traps are legal, as we will convert them to "j .+2".
 349:   setOperationAction(ISD::TRAP, MVT::Other, Legal);
 350: 
 351:   // We have native support for a 64-bit CTLZ, via FLOGR.
 352:   setOperationAction(ISD::CTLZ, MVT::i32, Promote);
 353:   setOperationAction(ISD::CTLZ_ZERO_POISON, MVT::i32, Promote);
 354:   setOperationAction(ISD::CTLZ, MVT::i64, Legal);
 355: 
 356:   // On z17 we have native support for a 64-bit CTTZ.
 357:   if (Subtarget.hasMiscellaneousExtensions4()) {
 358:     setOperationAction(ISD::CTTZ, MVT::i32, Promote);
 359:     setOperationAction(ISD::CTTZ_ZERO_POISON, MVT::i32, Promote);
 360:     setOperationAction(ISD::CTTZ, MVT::i64, Legal);
 361:   }
 362: 
 363:   // On z15 we have native support for a 64-bit CTPOP.
 364:   if (Subtarget.hasMiscellaneousExtensions3()) {
 365:     setOperationAction(ISD::CTPOP, MVT::i32, Promote);
 366:     setOperationAction(ISD::CTPOP, MVT::i64, Legal);
 367:   }
 368: 
 369:   // Give LowerOperation the chance to replace 64-bit ORs with subregs.
 370:   setOperationAction(ISD::OR, MVT::i64, Custom);
 371: 
 372:   // Expand 128 bit shifts without using a libcall.
 373:   setOperationAction(ISD::SRL_PARTS, MVT::i64, Expand);
 374:   setOperationAction(ISD::SHL_PARTS, MVT::i64, Expand);
 375:   setOperationAction(ISD::SRA_PARTS, MVT::i64, Expand);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 376-450 / 第 376-450 行
```cpp
 376: 
 377:   // Also expand 256 bit shifts if i128 is a legal type.
 378:   if (isTypeLegal(MVT::i128)) {
 379:     setOperationAction(ISD::SRL_PARTS, MVT::i128, Expand);
 380:     setOperationAction(ISD::SHL_PARTS, MVT::i128, Expand);
 381:     setOperationAction(ISD::SRA_PARTS, MVT::i128, Expand);
 382:   }
 383: 
 384:   // Handle bitcast from fp128 to i128.
 385:   if (!isTypeLegal(MVT::i128))
 386:     setOperationAction(ISD::BITCAST, MVT::i128, Custom);
 387: 
 388:   // We have native instructions for i8, i16 and i32 extensions, but not i1.
 389:   setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i1, Expand);
 390:   for (MVT VT : MVT::integer_valuetypes()) {
 391:     setLoadExtAction(ISD::SEXTLOAD, VT, MVT::i1, Promote);
 392:     setLoadExtAction(ISD::ZEXTLOAD, VT, MVT::i1, Promote);
 393:     setLoadExtAction(ISD::EXTLOAD,  VT, MVT::i1, Promote);
 394:   }
 395: 
 396:   // Handle the various types of symbolic address.
 397:   setOperationAction(ISD::ConstantPool,     PtrVT, Custom);
 398:   setOperationAction(ISD::GlobalAddress,    PtrVT, Custom);
 399:   setOperationAction(ISD::GlobalTLSAddress, PtrVT, Custom);
 400:   setOperationAction(ISD::BlockAddress,     PtrVT, Custom);
 401:   setOperationAction(ISD::JumpTable,        PtrVT, Custom);
 402: 
 403:   // We need to handle dynamic allocations specially because of the
 404:   // 160-byte area at the bottom of the stack.
 405:   setOperationAction(ISD::DYNAMIC_STACKALLOC, PtrVT, Custom);
 406:   setOperationAction(ISD::GET_DYNAMIC_AREA_OFFSET, PtrVT, Custom);
 407: 
 408:   setOperationAction(ISD::STACKSAVE,    MVT::Other, Custom);
 409:   setOperationAction(ISD::STACKRESTORE, MVT::Other, Custom);
 410: 
 411:   // Handle prefetches with PFD or PFDRL.
 412:   setOperationAction(ISD::PREFETCH, MVT::Other, Custom);
 413: 
 414:   // Handle readcyclecounter with STCKF.
 415:   setOperationAction(ISD::READCYCLECOUNTER, MVT::i64, Custom);
 416: 
 417:   for (MVT VT : MVT::fixedlen_vector_valuetypes()) {
 418:     // Assume by default that all vector operations need to be expanded.
 419:     for (unsigned Opcode = 0; Opcode < ISD::BUILTIN_OP_END; ++Opcode)
 420:       if (getOperationAction(Opcode, VT) == Legal)
 421:         setOperationAction(Opcode, VT, Expand);
 422: 
 423:     // Likewise all truncating stores and extending loads.
 424:     for (MVT InnerVT : MVT::fixedlen_vector_valuetypes()) {
 425:       setTruncStoreAction(VT, InnerVT, Expand);
 426:       setLoadExtAction(ISD::SEXTLOAD, VT, InnerVT, Expand);
 427:       setLoadExtAction(ISD::ZEXTLOAD, VT, InnerVT, Expand);
 428:       setLoadExtAction(ISD::EXTLOAD, VT, InnerVT, Expand);
 429:     }
 430: 
 431:     if (isTypeLegal(VT)) {
 432:       // These operations are legal for anything that can be stored in a
 433:       // vector register, even if there is no native support for the format
 434:       // as such.  In particular, we can do these for v4f32 even though there
 435:       // are no specific instructions for that format.
 436:       setOperationAction(ISD::LOAD, VT, Legal);
 437:       setOperationAction(ISD::STORE, VT, Legal);
 438:       setOperationAction(ISD::VSELECT, VT, Legal);
 439:       setOperationAction(ISD::BITCAST, VT, Legal);
 440:       setOperationAction(ISD::UNDEF, VT, Legal);
 441: 
 442:       // Likewise, except that we need to replace the nodes with something
 443:       // more specific.
 444:       setOperationAction(ISD::BUILD_VECTOR, VT, Custom);
 445:       setOperationAction(ISD::VECTOR_SHUFFLE, VT, Custom);
 446:     }
 447:   }
 448: 
 449:   // Handle integer vector types.
 450:   for (MVT VT : MVT::integer_fixedlen_vector_valuetypes()) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 451-525 / 第 451-525 行
```cpp
 451:     if (isTypeLegal(VT)) {
 452:       // These operations have direct equivalents.
 453:       setOperationAction(ISD::EXTRACT_VECTOR_ELT, VT, Legal);
 454:       setOperationAction(ISD::INSERT_VECTOR_ELT, VT, Legal);
 455:       setOperationAction(ISD::ADD, VT, Legal);
 456:       setOperationAction(ISD::SUB, VT, Legal);
 457:       if (VT != MVT::v2i64 || Subtarget.hasVectorEnhancements3()) {
 458:         setOperationAction(ISD::MUL, VT, Legal);
 459:         setOperationAction(ISD::MULHS, VT, Legal);
 460:         setOperationAction(ISD::MULHU, VT, Legal);
 461:       }
 462:       if (Subtarget.hasVectorEnhancements3() &&
 463:           VT != MVT::v16i8 && VT != MVT::v8i16) {
 464:         setOperationAction(ISD::SDIV, VT, Legal);
 465:         setOperationAction(ISD::UDIV, VT, Legal);
 466:         setOperationAction(ISD::SREM, VT, Legal);
 467:         setOperationAction(ISD::UREM, VT, Legal);
 468:       }
 469:       setOperationAction(ISD::ABS, VT, Legal);
 470:       setOperationAction(ISD::AND, VT, Legal);
 471:       setOperationAction(ISD::OR, VT, Legal);
 472:       setOperationAction(ISD::XOR, VT, Legal);
 473:       if (Subtarget.hasVectorEnhancements1())
 474:         setOperationAction(ISD::CTPOP, VT, Legal);
 475:       else
 476:         setOperationAction(ISD::CTPOP, VT, Custom);
 477:       setOperationAction(ISD::CTTZ, VT, Legal);
 478:       setOperationAction(ISD::CTLZ, VT, Legal);
 479: 
 480:       // Convert a GPR scalar to a vector by inserting it into element 0.
 481:       setOperationAction(ISD::SCALAR_TO_VECTOR, VT, Custom);
 482: 
 483:       // Use a series of unpacks for extensions.
 484:       setOperationAction(ISD::SIGN_EXTEND_VECTOR_INREG, VT, Custom);
 485:       setOperationAction(ISD::ZERO_EXTEND_VECTOR_INREG, VT, Custom);
 486: 
 487:       // Detect shifts/rotates by a scalar amount and convert them into
 488:       // V*_BY_SCALAR.
 489:       setOperationAction(ISD::SHL, VT, Custom);
 490:       setOperationAction(ISD::SRA, VT, Custom);
 491:       setOperationAction(ISD::SRL, VT, Custom);
 492:       setOperationAction(ISD::ROTL, VT, Custom);
 493: 
 494:       // Add ISD::VECREDUCE_ADD as custom in order to implement
 495:       // it with VZERO+VSUM
 496:       setOperationAction(ISD::VECREDUCE_ADD, VT, Custom);
 497: 
 498:       // Map SETCCs onto one of VCE, VCH or VCHL, swapping the operands
 499:       // and inverting the result as necessary.
 500:       setOperationAction(ISD::SETCC, VT, Custom);
 501: 
 502:       setOperationAction({ISD::SMIN, ISD::UMIN, ISD::SMAX, ISD::UMAX}, VT,
 503:                          Legal);
 504:     }
 505:   }
 506: 
 507:   if (Subtarget.hasVector()) {
 508:     // There should be no need to check for float types other than v2f64
 509:     // since <2 x f32> isn't a legal type.
 510:     setOperationAction(ISD::FP_TO_SINT, MVT::v2i64, Legal);
 511:     setOperationAction(ISD::FP_TO_SINT, MVT::v2f64, Legal);
 512:     setOperationAction(ISD::FP_TO_UINT, MVT::v2i64, Legal);
 513:     setOperationAction(ISD::FP_TO_UINT, MVT::v2f64, Legal);
 514:     setOperationAction(ISD::SINT_TO_FP, MVT::v2i64, Legal);
 515:     setOperationAction(ISD::SINT_TO_FP, MVT::v2f64, Legal);
 516:     setOperationAction(ISD::UINT_TO_FP, MVT::v2i64, Legal);
 517:     setOperationAction(ISD::UINT_TO_FP, MVT::v2f64, Legal);
 518: 
 519:     setOperationAction(ISD::STRICT_FP_TO_SINT, MVT::v2i64, Legal);
 520:     setOperationAction(ISD::STRICT_FP_TO_SINT, MVT::v2f64, Legal);
 521:     setOperationAction(ISD::STRICT_FP_TO_UINT, MVT::v2i64, Legal);
 522:     setOperationAction(ISD::STRICT_FP_TO_UINT, MVT::v2f64, Legal);
 523:     setOperationAction(ISD::STRICT_SINT_TO_FP, MVT::v2i64, Legal);
 524:     setOperationAction(ISD::STRICT_SINT_TO_FP, MVT::v2f64, Legal);
 525:     setOperationAction(ISD::STRICT_UINT_TO_FP, MVT::v2i64, Legal);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 526-600 / 第 526-600 行
```cpp
 526:     setOperationAction(ISD::STRICT_UINT_TO_FP, MVT::v2f64, Legal);
 527:   }
 528: 
 529:   if (Subtarget.hasVectorEnhancements2()) {
 530:     setOperationAction(ISD::FP_TO_SINT, MVT::v4i32, Legal);
 531:     setOperationAction(ISD::FP_TO_SINT, MVT::v4f32, Legal);
 532:     setOperationAction(ISD::FP_TO_UINT, MVT::v4i32, Legal);
 533:     setOperationAction(ISD::FP_TO_UINT, MVT::v4f32, Legal);
 534:     setOperationAction(ISD::SINT_TO_FP, MVT::v4i32, Legal);
 535:     setOperationAction(ISD::SINT_TO_FP, MVT::v4f32, Legal);
 536:     setOperationAction(ISD::UINT_TO_FP, MVT::v4i32, Legal);
 537:     setOperationAction(ISD::UINT_TO_FP, MVT::v4f32, Legal);
 538: 
 539:     setOperationAction(ISD::STRICT_FP_TO_SINT, MVT::v4i32, Legal);
 540:     setOperationAction(ISD::STRICT_FP_TO_SINT, MVT::v4f32, Legal);
 541:     setOperationAction(ISD::STRICT_FP_TO_UINT, MVT::v4i32, Legal);
 542:     setOperationAction(ISD::STRICT_FP_TO_UINT, MVT::v4f32, Legal);
 543:     setOperationAction(ISD::STRICT_SINT_TO_FP, MVT::v4i32, Legal);
 544:     setOperationAction(ISD::STRICT_SINT_TO_FP, MVT::v4f32, Legal);
 545:     setOperationAction(ISD::STRICT_UINT_TO_FP, MVT::v4i32, Legal);
 546:     setOperationAction(ISD::STRICT_UINT_TO_FP, MVT::v4f32, Legal);
 547:   }
 548: 
 549:   // Handle floating-point types.
 550:   if (!useSoftFloat()) {
 551:     // Promote all f16 operations to float, with some exceptions below.
 552:     for (unsigned Opc = 0; Opc < ISD::BUILTIN_OP_END; ++Opc)
 553:       setOperationAction(Opc, MVT::f16, Promote);
 554:     setOperationAction(ISD::ConstantFP, MVT::f16, Expand);
 555:     for (MVT VT : {MVT::f32, MVT::f64, MVT::f128}) {
 556:       setLoadExtAction(ISD::EXTLOAD, VT, MVT::f16, Expand);
 557:       setTruncStoreAction(VT, MVT::f16, Expand);
 558:     }
 559:     for (auto Op : {ISD::LOAD, ISD::ATOMIC_LOAD, ISD::STORE, ISD::ATOMIC_STORE})
 560:       setOperationAction(Op, MVT::f16, Subtarget.hasVector() ? Legal : Custom);
 561:     setOperationAction(ISD::FP_ROUND, MVT::f16, LibCall);
 562:     setOperationAction(ISD::STRICT_FP_ROUND, MVT::f16, LibCall);
 563:     setOperationAction(ISD::BITCAST, MVT::i16, Custom);
 564: 
 565:     for (auto Op : {ISD::FNEG, ISD::FABS, ISD::FCOPYSIGN})
 566:       setOperationAction(Op, MVT::f16, Legal);
 567:   }
 568: 
 569:   for (unsigned I = MVT::FIRST_FP_VALUETYPE;
 570:        I <= MVT::LAST_FP_VALUETYPE;
 571:        ++I) {
 572:     MVT VT = MVT::SimpleValueType(I);
 573:     if (isTypeLegal(VT) && VT != MVT::f16) {
 574:       // We can use FI for FRINT.
 575:       setOperationAction(ISD::FRINT, VT, Legal);
 576: 
 577:       // We can use the extended form of FI for other rounding operations.
 578:       if (Subtarget.hasFPExtension()) {
 579:         setOperationAction(ISD::FNEARBYINT, VT, Legal);
 580:         setOperationAction(ISD::FFLOOR, VT, Legal);
 581:         setOperationAction(ISD::FCEIL, VT, Legal);
 582:         setOperationAction(ISD::FTRUNC, VT, Legal);
 583:         setOperationAction(ISD::FROUND, VT, Legal);
 584:         setOperationAction(ISD::FROUNDEVEN, VT, Legal);
 585:       }
 586: 
 587:       // No special instructions for these.
 588:       setOperationAction(ISD::FSIN, VT, Expand);
 589:       setOperationAction(ISD::FCOS, VT, Expand);
 590:       setOperationAction(ISD::FSINCOS, VT, Expand);
 591:       setOperationAction(ISD::FREM, VT, LibCall);
 592:       setOperationAction(ISD::FPOW, VT, Expand);
 593: 
 594:       // Special treatment.
 595:       setOperationAction(ISD::IS_FPCLASS, VT, Custom);
 596: 
 597:       // Handle constrained floating-point operations.
 598:       setOperationAction(ISD::STRICT_FADD, VT, Legal);
 599:       setOperationAction(ISD::STRICT_FSUB, VT, Legal);
 600:       setOperationAction(ISD::STRICT_FMUL, VT, Legal);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 601-675 / 第 601-675 行
```cpp
 601:       setOperationAction(ISD::STRICT_FDIV, VT, Legal);
 602:       setOperationAction(ISD::STRICT_FMA, VT, Legal);
 603:       setOperationAction(ISD::STRICT_FSQRT, VT, Legal);
 604:       setOperationAction(ISD::STRICT_FRINT, VT, Legal);
 605:       setOperationAction(ISD::STRICT_FP_ROUND, VT, Legal);
 606:       if (Subtarget.hasFPExtension()) {
 607:         setOperationAction(ISD::STRICT_FNEARBYINT, VT, Legal);
 608:         setOperationAction(ISD::STRICT_FFLOOR, VT, Legal);
 609:         setOperationAction(ISD::STRICT_FCEIL, VT, Legal);
 610:         setOperationAction(ISD::STRICT_FTRUNC, VT, Legal);
 611:         setOperationAction(ISD::STRICT_FROUND, VT, Legal);
 612:         setOperationAction(ISD::STRICT_FROUNDEVEN, VT, Legal);
 613:       }
 614: 
 615:       // Extension from f16 needs libcall.
 616:       setOperationAction(ISD::FP_EXTEND, VT, Custom);
 617:       setOperationAction(ISD::STRICT_FP_EXTEND, VT, Custom);
 618:     }
 619:   }
 620: 
 621:   // Handle floating-point vector types.
 622:   if (Subtarget.hasVector()) {
 623:     // Scalar-to-vector conversion is just a subreg.
 624:     setOperationAction(ISD::SCALAR_TO_VECTOR, MVT::v8f16, Legal);
 625:     setOperationAction(ISD::SCALAR_TO_VECTOR, MVT::v4f32, Legal);
 626:     setOperationAction(ISD::SCALAR_TO_VECTOR, MVT::v2f64, Legal);
 627: 
 628:     // Some insertions and extractions can be done directly but others
 629:     // need to go via integers.
 630:     setOperationAction(ISD::INSERT_VECTOR_ELT, MVT::v8f16, Custom);
 631:     setOperationAction(ISD::INSERT_VECTOR_ELT, MVT::v4f32, Custom);
 632:     setOperationAction(ISD::INSERT_VECTOR_ELT, MVT::v2f64, Custom);
 633:     setOperationAction(ISD::EXTRACT_VECTOR_ELT, MVT::v8f16, Custom);
 634:     setOperationAction(ISD::EXTRACT_VECTOR_ELT, MVT::v4f32, Custom);
 635:     setOperationAction(ISD::EXTRACT_VECTOR_ELT, MVT::v2f64, Custom);
 636: 
 637:     // These operations have direct equivalents.
 638:     setOperationAction(ISD::FADD, MVT::v2f64, Legal);
 639:     setOperationAction(ISD::FNEG, MVT::v2f64, Legal);
 640:     setOperationAction(ISD::FSUB, MVT::v2f64, Legal);
 641:     setOperationAction(ISD::FMUL, MVT::v2f64, Legal);
 642:     setOperationAction(ISD::FMA, MVT::v2f64, Legal);
 643:     setOperationAction(ISD::FDIV, MVT::v2f64, Legal);
 644:     setOperationAction(ISD::FABS, MVT::v2f64, Legal);
 645:     setOperationAction(ISD::FSQRT, MVT::v2f64, Legal);
 646:     setOperationAction(ISD::FRINT, MVT::v2f64, Legal);
 647:     setOperationAction(ISD::FNEARBYINT, MVT::v2f64, Legal);
 648:     setOperationAction(ISD::FFLOOR, MVT::v2f64, Legal);
 649:     setOperationAction(ISD::FCEIL, MVT::v2f64, Legal);
 650:     setOperationAction(ISD::FTRUNC, MVT::v2f64, Legal);
 651:     setOperationAction(ISD::FROUND, MVT::v2f64, Legal);
 652:     setOperationAction(ISD::FROUNDEVEN, MVT::v2f64, Legal);
 653: 
 654:     // Handle constrained floating-point operations.
 655:     setOperationAction(ISD::STRICT_FADD, MVT::v2f64, Legal);
 656:     setOperationAction(ISD::STRICT_FSUB, MVT::v2f64, Legal);
 657:     setOperationAction(ISD::STRICT_FMUL, MVT::v2f64, Legal);
 658:     setOperationAction(ISD::STRICT_FMA, MVT::v2f64, Legal);
 659:     setOperationAction(ISD::STRICT_FDIV, MVT::v2f64, Legal);
 660:     setOperationAction(ISD::STRICT_FSQRT, MVT::v2f64, Legal);
 661:     setOperationAction(ISD::STRICT_FRINT, MVT::v2f64, Legal);
 662:     setOperationAction(ISD::STRICT_FNEARBYINT, MVT::v2f64, Legal);
 663:     setOperationAction(ISD::STRICT_FFLOOR, MVT::v2f64, Legal);
 664:     setOperationAction(ISD::STRICT_FCEIL, MVT::v2f64, Legal);
 665:     setOperationAction(ISD::STRICT_FTRUNC, MVT::v2f64, Legal);
 666:     setOperationAction(ISD::STRICT_FROUND, MVT::v2f64, Legal);
 667:     setOperationAction(ISD::STRICT_FROUNDEVEN, MVT::v2f64, Legal);
 668: 
 669:     setOperationAction(ISD::SETCC, MVT::v2f64, Custom);
 670:     setOperationAction(ISD::SETCC, MVT::v4f32, Custom);
 671:     setOperationAction(ISD::STRICT_FSETCC, MVT::v2f64, Custom);
 672:     setOperationAction(ISD::STRICT_FSETCC, MVT::v4f32, Custom);
 673:     if (Subtarget.hasVectorEnhancements1()) {
 674:       setOperationAction(ISD::STRICT_FSETCCS, MVT::v2f64, Custom);
 675:       setOperationAction(ISD::STRICT_FSETCCS, MVT::v4f32, Custom);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 676-750 / 第 676-750 行
```cpp
 676:     }
 677:   }
 678: 
 679:   // The vector enhancements facility 1 has instructions for these.
 680:   if (Subtarget.hasVectorEnhancements1()) {
 681:     setOperationAction(ISD::FADD, MVT::v4f32, Legal);
 682:     setOperationAction(ISD::FNEG, MVT::v4f32, Legal);
 683:     setOperationAction(ISD::FSUB, MVT::v4f32, Legal);
 684:     setOperationAction(ISD::FMUL, MVT::v4f32, Legal);
 685:     setOperationAction(ISD::FMA, MVT::v4f32, Legal);
 686:     setOperationAction(ISD::FDIV, MVT::v4f32, Legal);
 687:     setOperationAction(ISD::FABS, MVT::v4f32, Legal);
 688:     setOperationAction(ISD::FSQRT, MVT::v4f32, Legal);
 689:     setOperationAction(ISD::FRINT, MVT::v4f32, Legal);
 690:     setOperationAction(ISD::FNEARBYINT, MVT::v4f32, Legal);
 691:     setOperationAction(ISD::FFLOOR, MVT::v4f32, Legal);
 692:     setOperationAction(ISD::FCEIL, MVT::v4f32, Legal);
 693:     setOperationAction(ISD::FTRUNC, MVT::v4f32, Legal);
 694:     setOperationAction(ISD::FROUND, MVT::v4f32, Legal);
 695:     setOperationAction(ISD::FROUNDEVEN, MVT::v4f32, Legal);
 696: 
 697:     for (MVT Type : {MVT::f64, MVT::v2f64, MVT::f32, MVT::v4f32, MVT::f128}) {
 698:       setOperationAction(ISD::FMAXNUM, Type, Legal);
 699:       setOperationAction(ISD::FMAXIMUM, Type, Legal);
 700:       setOperationAction(ISD::FMAXIMUMNUM, Type, Legal);
 701:       setOperationAction(ISD::FMINNUM, Type, Legal);
 702:       setOperationAction(ISD::FMINIMUM, Type, Legal);
 703:       setOperationAction(ISD::FMINIMUMNUM, Type, Legal);
 704:     }
 705: 
 706:     // Handle constrained floating-point operations.
 707:     setOperationAction(ISD::STRICT_FADD, MVT::v4f32, Legal);
 708:     setOperationAction(ISD::STRICT_FSUB, MVT::v4f32, Legal);
 709:     setOperationAction(ISD::STRICT_FMUL, MVT::v4f32, Legal);
 710:     setOperationAction(ISD::STRICT_FMA, MVT::v4f32, Legal);
 711:     setOperationAction(ISD::STRICT_FDIV, MVT::v4f32, Legal);
 712:     setOperationAction(ISD::STRICT_FSQRT, MVT::v4f32, Legal);
 713:     setOperationAction(ISD::STRICT_FRINT, MVT::v4f32, Legal);
 714:     setOperationAction(ISD::STRICT_FNEARBYINT, MVT::v4f32, Legal);
 715:     setOperationAction(ISD::STRICT_FFLOOR, MVT::v4f32, Legal);
 716:     setOperationAction(ISD::STRICT_FCEIL, MVT::v4f32, Legal);
 717:     setOperationAction(ISD::STRICT_FTRUNC, MVT::v4f32, Legal);
 718:     setOperationAction(ISD::STRICT_FROUND, MVT::v4f32, Legal);
 719:     setOperationAction(ISD::STRICT_FROUNDEVEN, MVT::v4f32, Legal);
 720:     for (auto VT : { MVT::f32, MVT::f64, MVT::f128,
 721:                      MVT::v4f32, MVT::v2f64 }) {
 722:       setOperationAction(ISD::STRICT_FMAXNUM, VT, Legal);
 723:       setOperationAction(ISD::STRICT_FMINNUM, VT, Legal);
 724:       setOperationAction(ISD::STRICT_FMAXIMUM, VT, Legal);
 725:       setOperationAction(ISD::STRICT_FMINIMUM, VT, Legal);
 726:     }
 727:   }
 728: 
 729:   // We only have fused f128 multiply-addition on vector registers.
 730:   if (!Subtarget.hasVectorEnhancements1()) {
 731:     setOperationAction(ISD::FMA, MVT::f128, Expand);
 732:     setOperationAction(ISD::STRICT_FMA, MVT::f128, Expand);
 733:   }
 734: 
 735:   // We don't have a copysign instruction on vector registers.
 736:   if (Subtarget.hasVectorEnhancements1())
 737:     setOperationAction(ISD::FCOPYSIGN, MVT::f128, Expand);
 738: 
 739:   // Needed so that we don't try to implement f128 constant loads using
 740:   // a load-and-extend of a f80 constant (in cases where the constant
 741:   // would fit in an f80).
 742:   for (MVT VT : MVT::fp_valuetypes())
 743:     setLoadExtAction(ISD::EXTLOAD, VT, MVT::f80, Expand);
 744: 
 745:   // We don't have extending load instruction on vector registers.
 746:   if (Subtarget.hasVectorEnhancements1()) {
 747:     setLoadExtAction(ISD::EXTLOAD, MVT::f128, MVT::f32, Expand);
 748:     setLoadExtAction(ISD::EXTLOAD, MVT::f128, MVT::f64, Expand);
 749:   }
 750: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 751-825 / 第 751-825 行
```cpp
 751:   // Floating-point truncation and stores need to be done separately.
 752:   setTruncStoreAction(MVT::f64,  MVT::f32, Expand);
 753:   setTruncStoreAction(MVT::f128, MVT::f32, Expand);
 754:   setTruncStoreAction(MVT::f128, MVT::f64, Expand);
 755: 
 756:   // We have 64-bit FPR<->GPR moves, but need special handling for
 757:   // 32-bit forms.
 758:   if (!Subtarget.hasVector()) {
 759:     setOperationAction(ISD::BITCAST, MVT::i32, Custom);
 760:     setOperationAction(ISD::BITCAST, MVT::f32, Custom);
 761:   }
 762: 
 763:   // VASTART and VACOPY need to deal with the SystemZ-specific varargs
 764:   // structure, but VAEND is a no-op.
 765:   setOperationAction(ISD::VASTART, MVT::Other, Custom);
 766:   setOperationAction(ISD::VACOPY,  MVT::Other, Custom);
 767:   setOperationAction(ISD::VAEND,   MVT::Other, Expand);
 768: 
 769:   if (Subtarget.isTargetzOS()) {
 770:     // Handle address space casts between mixed sized pointers.
 771:     setOperationAction(ISD::ADDRSPACECAST, MVT::i32, Custom);
 772:     setOperationAction(ISD::ADDRSPACECAST, MVT::i64, Custom);
 773:   }
 774: 
 775:   setOperationAction(ISD::GET_ROUNDING, MVT::i32, Custom);
 776: 
 777:   // Codes for which we want to perform some z-specific combinations.
 778:   setTargetDAGCombine({ISD::ZERO_EXTEND,
 779:                        ISD::SIGN_EXTEND,
 780:                        ISD::SIGN_EXTEND_INREG,
 781:                        ISD::LOAD,
 782:                        ISD::STORE,
 783:                        ISD::VECTOR_SHUFFLE,
 784:                        ISD::EXTRACT_VECTOR_ELT,
 785:                        ISD::FP_ROUND,
 786:                        ISD::STRICT_FP_ROUND,
 787:                        ISD::FP_EXTEND,
 788:                        ISD::SINT_TO_FP,
 789:                        ISD::UINT_TO_FP,
 790:                        ISD::STRICT_FP_EXTEND,
 791:                        ISD::FCOPYSIGN,
 792:                        ISD::BSWAP,
 793:                        ISD::SETCC,
 794:                        ISD::SRL,
 795:                        ISD::SRA,
 796:                        ISD::MUL,
 797:                        ISD::SDIV,
 798:                        ISD::UDIV,
 799:                        ISD::SREM,
 800:                        ISD::UREM,
 801:                        ISD::INTRINSIC_VOID,
 802:                        ISD::INTRINSIC_W_CHAIN});
 803: 
 804:   // Handle intrinsics.
 805:   setOperationAction(ISD::INTRINSIC_W_CHAIN, MVT::Other, Custom);
 806:   setOperationAction(ISD::INTRINSIC_WO_CHAIN, MVT::Other, Custom);
 807: 
 808:   // We're not using SJLJ for exception handling, but they're implemented
 809:   // solely to support use of __builtin_setjmp / __builtin_longjmp.
 810:   setOperationAction(ISD::EH_SJLJ_SETJMP, MVT::i32, Custom);
 811:   setOperationAction(ISD::EH_SJLJ_LONGJMP, MVT::Other, Custom);
 812: 
 813:   // We want to use MVC in preference to even a single load/store pair.
 814:   MaxStoresPerMemcpy = Subtarget.hasVector() ? 2 : 0;
 815:   MaxStoresPerMemcpyOptSize = 0;
 816: 
 817:   // Same with memmove.
 818:   MaxStoresPerMemmove = Subtarget.hasVector() ? 2 : 0;
 819:   MaxStoresPerMemmoveOptSize = 0;
 820: 
 821:   // The main memset sequence is a byte store followed by an MVC.
 822:   // Two STC or MV..I stores win over that, but the kind of fused stores
 823:   // generated by target-independent code don't when the byte value is
 824:   // variable.  E.g.  "STC <reg>;MHI <reg>,257;STH <reg>" is not better
 825:   // than "STC;MVC".  Handle the choice in target-specific code instead.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 826-900 / 第 826-900 行
```cpp
 826:   MaxStoresPerMemset = Subtarget.hasVector() ? 2 : 0;
 827:   MaxStoresPerMemsetOptSize = 0;
 828: 
 829:   // Default to having -disable-strictnode-mutation on
 830:   IsStrictFPEnabled = true;
 831: }
 832: 
 833: bool SystemZTargetLowering::useSoftFloat() const {
 834:   return Subtarget.hasSoftFloat();
 835: }
 836: 
 837: unsigned SystemZTargetLowering::getVectorTypeBreakdownForCallingConv(
 838:     LLVMContext &Context, CallingConv::ID CC, EVT VT, EVT &IntermediateVT,
 839:     unsigned &NumIntermediates, MVT &RegisterVT) const {
 840:   // Pass fp16 vectors in VR(s).
 841:   if (Subtarget.hasVector() && VT.isVector() && VT.getScalarType() == MVT::f16) {
 842:     IntermediateVT = RegisterVT = MVT::v8f16;
 843:     return NumIntermediates =
 844:                divideCeil(VT.getVectorNumElements(), SystemZ::VectorBytes / 2);
 845:   }
 846:   return TargetLowering::getVectorTypeBreakdownForCallingConv(
 847:       Context, CC, VT, IntermediateVT, NumIntermediates, RegisterVT);
 848: }
 849: 
 850: MVT SystemZTargetLowering::getRegisterTypeForCallingConv(LLVMContext &Context,
 851:                                                          CallingConv::ID CC,
 852:                                                          EVT VT) const {
 853:   // 128-bit single-element vector types are passed like other vectors,
 854:   // not like their element type.
 855:   if (Subtarget.hasVector() && VT.isVector() && VT.getSizeInBits() == 128 &&
 856:       VT.getVectorNumElements() == 1)
 857:     return MVT::v16i8;
 858:   // Pass fp16 vectors in VR(s).
 859:   if (Subtarget.hasVector() && VT.isVector() && VT.getScalarType() == MVT::f16)
 860:     return MVT::v8f16;
 861:   return TargetLowering::getRegisterTypeForCallingConv(Context, CC, VT);
 862: }
 863: 
 864: unsigned SystemZTargetLowering::getNumRegistersForCallingConv(
 865:     LLVMContext &Context, CallingConv::ID CC, EVT VT) const {
 866:   // Pass fp16 vectors in VR(s).
 867:   if (Subtarget.hasVector() && VT.isVector() && VT.getScalarType() == MVT::f16)
 868:     return divideCeil(VT.getVectorNumElements(), SystemZ::VectorBytes / 2);
 869:   return TargetLowering::getNumRegistersForCallingConv(Context, CC, VT);
 870: }
 871: 
 872: EVT SystemZTargetLowering::getSetCCResultType(const DataLayout &DL,
 873:                                               LLVMContext &, EVT VT) const {
 874:   if (!VT.isVector())
 875:     return MVT::i32;
 876:   return VT.changeVectorElementTypeToInteger();
 877: }
 878: 
 879: bool SystemZTargetLowering::isFMAFasterThanFMulAndFAdd(
 880:     const MachineFunction &MF, EVT VT) const {
 881:   if (useSoftFloat())
 882:     return false;
 883: 
 884:   VT = VT.getScalarType();
 885: 
 886:   if (!VT.isSimple())
 887:     return false;
 888: 
 889:   switch (VT.getSimpleVT().SimpleTy) {
 890:   case MVT::f32:
 891:   case MVT::f64:
 892:     return true;
 893:   case MVT::f128:
 894:     return Subtarget.hasVectorEnhancements1();
 895:   default:
 896:     break;
 897:   }
 898: 
 899:   return false;
 900: }
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::useSoftFloat`, `SystemZTargetLowering::getVectorTypeBreakdownForCallingConv`, `SystemZTargetLowering::getRegisterTypeForCallingConv`, `SystemZTargetLowering::getNumRegistersForCallingConv`, `SystemZTargetLowering::getSetCCResultType`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::useSoftFloat`, `SystemZTargetLowering::getVectorTypeBreakdownForCallingConv`, `SystemZTargetLowering::getRegisterTypeForCallingConv`, `SystemZTargetLowering::getNumRegistersForCallingConv`, `SystemZTargetLowering::getSetCCResultType` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 901-975 / 第 901-975 行
```cpp
 901: 
 902: // Return true if the constant can be generated with a vector instruction,
 903: // such as VGM, VGMB or VREPI.
 904: bool SystemZVectorConstantInfo::isVectorConstantLegal(
 905:     const SystemZSubtarget &Subtarget) {
 906:   const SystemZInstrInfo *TII = Subtarget.getInstrInfo();
 907:   if (!Subtarget.hasVector() ||
 908:       (isFP128 && !Subtarget.hasVectorEnhancements1()))
 909:     return false;
 910: 
 911:   // Try using VECTOR GENERATE BYTE MASK.  This is the architecturally-
 912:   // preferred way of creating all-zero and all-one vectors so give it
 913:   // priority over other methods below.
 914:   unsigned Mask = 0;
 915:   unsigned I = 0;
 916:   for (; I < SystemZ::VectorBytes; ++I) {
 917:     uint64_t Byte = IntBits.lshr(I * 8).trunc(8).getZExtValue();
 918:     if (Byte == 0xff)
 919:       Mask |= 1ULL << I;
 920:     else if (Byte != 0)
 921:       break;
 922:   }
 923:   if (I == SystemZ::VectorBytes) {
 924:     Opcode = SystemZISD::BYTE_MASK;
 925:     OpVals.push_back(Mask);
 926:     VecVT = MVT::getVectorVT(MVT::getIntegerVT(8), 16);
 927:     return true;
 928:   }
 929: 
 930:   if (SplatBitSize > 64)
 931:     return false;
 932: 
 933:   auto TryValue = [&](uint64_t Value) -> bool {
 934:     // Try VECTOR REPLICATE IMMEDIATE
 935:     int64_t SignedValue = SignExtend64(Value, SplatBitSize);
 936:     if (isInt<16>(SignedValue)) {
 937:       OpVals.push_back(((unsigned) SignedValue));
 938:       Opcode = SystemZISD::REPLICATE;
 939:       VecVT = MVT::getVectorVT(MVT::getIntegerVT(SplatBitSize),
 940:                                SystemZ::VectorBits / SplatBitSize);
 941:       return true;
 942:     }
 943:     // Try VECTOR GENERATE MASK
 944:     unsigned Start, End;
 945:     if (TII->isRxSBGMask(Value, SplatBitSize, Start, End)) {
 946:       // isRxSBGMask returns the bit numbers for a full 64-bit value, with 0
 947:       // denoting 1 << 63 and 63 denoting 1.  Convert them to bit numbers for
 948:       // an SplatBitSize value, so that 0 denotes 1 << (SplatBitSize-1).
 949:       OpVals.push_back(Start - (64 - SplatBitSize));
 950:       OpVals.push_back(End - (64 - SplatBitSize));
 951:       Opcode = SystemZISD::ROTATE_MASK;
 952:       VecVT = MVT::getVectorVT(MVT::getIntegerVT(SplatBitSize),
 953:                                SystemZ::VectorBits / SplatBitSize);
 954:       return true;
 955:     }
 956:     return false;
 957:   };
 958: 
 959:   // First try assuming that any undefined bits above the highest set bit
 960:   // and below the lowest set bit are 1s.  This increases the likelihood of
 961:   // being able to use a sign-extended element value in VECTOR REPLICATE
 962:   // IMMEDIATE or a wraparound mask in VECTOR GENERATE MASK.
 963:   uint64_t SplatBitsZ = SplatBits.getZExtValue();
 964:   uint64_t SplatUndefZ = SplatUndef.getZExtValue();
 965:   unsigned LowerBits = llvm::countr_zero(SplatBitsZ);
 966:   unsigned UpperBits = llvm::countl_zero(SplatBitsZ);
 967:   uint64_t Lower = SplatUndefZ & maskTrailingOnes<uint64_t>(LowerBits);
 968:   uint64_t Upper = SplatUndefZ & maskLeadingOnes<uint64_t>(UpperBits);
 969:   if (TryValue(SplatBitsZ | Upper | Lower))
 970:     return true;
 971: 
 972:   // Now try assuming that any undefined bits between the first and
 973:   // last defined set bits are set.  This increases the chances of
 974:   // using a non-wraparound mask.
 975:   uint64_t Middle = SplatUndefZ & ~Upper & ~Lower;
```
- **EN**: The range implements or declares functions including `SystemZVectorConstantInfo::isVectorConstantLegal`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZVectorConstantInfo::isVectorConstantLegal` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 976-1050 / 第 976-1050 行
```cpp
 976:   return TryValue(SplatBitsZ | Middle);
 977: }
 978: 
 979: SystemZVectorConstantInfo::SystemZVectorConstantInfo(APInt IntImm) {
 980:   if (IntImm.isSingleWord()) {
 981:     IntBits = APInt(128, IntImm.getZExtValue());
 982:     IntBits <<= (SystemZ::VectorBits - IntImm.getBitWidth());
 983:   } else
 984:     IntBits = IntImm;
 985:   assert(IntBits.getBitWidth() == 128 && "Unsupported APInt.");
 986: 
 987:   // Find the smallest splat.
 988:   SplatBits = IntImm;
 989:   unsigned Width = SplatBits.getBitWidth();
 990:   while (Width > 8) {
 991:     unsigned HalfSize = Width / 2;
 992:     APInt HighValue = SplatBits.lshr(HalfSize).trunc(HalfSize);
 993:     APInt LowValue = SplatBits.trunc(HalfSize);
 994: 
 995:     // If the two halves do not match, stop here.
 996:     if (HighValue != LowValue || 8 > HalfSize)
 997:       break;
 998: 
 999:     SplatBits = HighValue;
1000:     Width = HalfSize;
1001:   }
1002:   SplatUndef = 0;
1003:   SplatBitSize = Width;
1004: }
1005: 
1006: SystemZVectorConstantInfo::SystemZVectorConstantInfo(BuildVectorSDNode *BVN) {
1007:   assert(BVN->isConstant() && "Expected a constant BUILD_VECTOR");
1008:   bool HasAnyUndefs;
1009: 
1010:   // Get IntBits by finding the 128 bit splat.
1011:   BVN->isConstantSplat(IntBits, SplatUndef, SplatBitSize, HasAnyUndefs, 128,
1012:                        true);
1013: 
1014:   // Get SplatBits by finding the 8 bit or greater splat.
1015:   BVN->isConstantSplat(SplatBits, SplatUndef, SplatBitSize, HasAnyUndefs, 8,
1016:                        true);
1017: }
1018: 
1019: bool SystemZTargetLowering::isFPImmLegal(const APFloat &Imm, EVT VT,
1020:                                          bool ForCodeSize) const {
1021:   // We can load zero using LZ?R and negative zero using LZ?R;LC?BR.
1022:   if (Imm.isZero() || Imm.isNegZero())
1023:     return true;
1024: 
1025:   return SystemZVectorConstantInfo(Imm).isVectorConstantLegal(Subtarget);
1026: }
1027: 
1028: MachineBasicBlock *
1029: SystemZTargetLowering::emitEHSjLjSetJmp(MachineInstr &MI,
1030:                                         MachineBasicBlock *MBB) const {
1031:   DebugLoc DL = MI.getDebugLoc();
1032:   const TargetInstrInfo *TII = Subtarget.getInstrInfo();
1033:   const SystemZRegisterInfo *TRI = Subtarget.getRegisterInfo();
1034: 
1035:   MachineFunction *MF = MBB->getParent();
1036:   MachineRegisterInfo &MRI = MF->getRegInfo();
1037: 
1038:   const BasicBlock *BB = MBB->getBasicBlock();
1039:   MachineFunction::iterator I = ++MBB->getIterator();
1040: 
1041:   Register DstReg = MI.getOperand(0).getReg();
1042:   const TargetRegisterClass *RC = MRI.getRegClass(DstReg);
1043:   assert(TRI->isTypeLegalForClass(*RC, MVT::i32) && "Invalid destination!");
1044:   (void)TRI;
1045:   Register MainDstReg = MRI.createVirtualRegister(RC);
1046:   Register RestoreDstReg = MRI.createVirtualRegister(RC);
1047: 
1048:   MVT PVT = getPointerTy(MF->getDataLayout());
1049:   assert((PVT == MVT::i64 || PVT == MVT::i32) && "Invalid Pointer Size!");
1050:   // For v = setjmp(buf), we generate.
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::isFPImmLegal`, `SystemZTargetLowering::emitEHSjLjSetJmp`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::isFPImmLegal`, `SystemZTargetLowering::emitEHSjLjSetJmp` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1051-1125 / 第 1051-1125 行
```cpp
1051:   // Algorithm:
1052:   //
1053:   //                 ---------
1054:   //                | thisMBB |
1055:   //                 ---------
1056:   //                     |
1057:   //         ------------------------
1058:   //        |                        |
1059:   //     ----------           ---------------
1060:   //    |  mainMBB |         | restoreMBB    |
1061:   //    |   v = 0  |         |  v = 1        |
1062:   //     ----------           ---------------
1063:   //        |                        |
1064:   //         -------------------------
1065:   //                     |
1066:   //           -----------------------------
1067:   //          |       sinkMBB               |
1068:   //          | phi(v_mainMBB,v_restoreMBB) |
1069:   //           -----------------------------
1070:   // thisMBB:
1071:   //  buf[FPOffset] = Frame Pointer if hasFP.
1072:   //  buf[LabelOffset] = restoreMBB <-- takes address of restoreMBB.
1073:   //  buf[BCOffset] = Backchain value if building with -mbackchain.
1074:   //  buf[SPOffset] = Stack Pointer.
1075:   //  buf[LPOffset] = We never write this slot with  R13, gcc stores R13 always.
1076:   //  SjLjSetup restoreMBB
1077:   // mainMBB:
1078:   //  v_main = 0
1079:   // sinkMBB:
1080:   //  v = phi(v_main, v_restore)
1081:   // restoreMBB:
1082:   //  v_restore = 1
1083: 
1084:   MachineBasicBlock *ThisMBB = MBB;
1085:   MachineBasicBlock *MainMBB = MF->CreateMachineBasicBlock(BB);
1086:   MachineBasicBlock *SinkMBB = MF->CreateMachineBasicBlock(BB);
1087:   MachineBasicBlock *RestoreMBB = MF->CreateMachineBasicBlock(BB);
1088: 
1089:   MF->insert(I, MainMBB);
1090:   MF->insert(I, SinkMBB);
1091:   MF->push_back(RestoreMBB);
1092:   RestoreMBB->setMachineBlockAddressTaken();
1093: 
1094:   MachineInstrBuilder MIB;
1095: 
1096:   // Transfer the remainder of BB and its successor edges to sinkMBB.
1097:   SinkMBB->splice(SinkMBB->begin(), MBB,
1098:                   std::next(MachineBasicBlock::iterator(MI)), MBB->end());
1099:   SinkMBB->transferSuccessorsAndUpdatePHIs(MBB);
1100: 
1101:   // thisMBB:
1102:   const int64_t FPOffset = 0;                         // Slot 1.
1103:   const int64_t LabelOffset = 1 * PVT.getStoreSize(); // Slot 2.
1104:   const int64_t BCOffset = 2 * PVT.getStoreSize();    // Slot 3.
1105:   const int64_t SPOffset = 3 * PVT.getStoreSize();    // Slot 4.
1106: 
1107:   // Buf address.
1108:   Register BufReg = MI.getOperand(1).getReg();
1109: 
1110:   const TargetRegisterClass *PtrRC = getRegClassFor(PVT);
1111:   Register LabelReg = MRI.createVirtualRegister(PtrRC);
1112: 
1113:   // Prepare IP for longjmp.
1114:   BuildMI(*ThisMBB, MI, DL, TII->get(SystemZ::LARL), LabelReg)
1115:       .addMBB(RestoreMBB);
1116:   // Store IP for return from jmp, slot 2, offset = 1.
1117:   BuildMI(*ThisMBB, MI, DL, TII->get(SystemZ::STG))
1118:       .addReg(LabelReg)
1119:       .addReg(BufReg)
1120:       .addImm(LabelOffset)
1121:       .addReg(0);
1122: 
1123:   auto *SpecialRegs = Subtarget.getSpecialRegisters();
1124:   bool HasFP = Subtarget.getFrameLowering()->hasFP(*MF);
1125:   if (HasFP) {
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1126-1200 / 第 1126-1200 行
```cpp
1126:     BuildMI(*ThisMBB, MI, DL, TII->get(SystemZ::STG))
1127:         .addReg(SpecialRegs->getFramePointerRegister())
1128:         .addReg(BufReg)
1129:         .addImm(FPOffset)
1130:         .addReg(0);
1131:   }
1132: 
1133:   // Store SP.
1134:   BuildMI(*ThisMBB, MI, DL, TII->get(SystemZ::STG))
1135:       .addReg(SpecialRegs->getStackPointerRegister())
1136:       .addReg(BufReg)
1137:       .addImm(SPOffset)
1138:       .addReg(0);
1139: 
1140:   // Slot 3(Offset = 2) Backchain value (if building with -mbackchain).
1141:   bool BackChain = MF->getSubtarget<SystemZSubtarget>().hasBackChain();
1142:   if (BackChain) {
1143:     Register BCReg = MRI.createVirtualRegister(PtrRC);
1144:     auto *TFL = Subtarget.getFrameLowering<SystemZFrameLowering>();
1145:     MIB = BuildMI(*ThisMBB, MI, DL, TII->get(SystemZ::LG), BCReg)
1146:               .addReg(SpecialRegs->getStackPointerRegister())
1147:               .addImm(TFL->getBackchainOffset(*MF))
1148:               .addReg(0);
1149: 
1150:     BuildMI(*ThisMBB, MI, DL, TII->get(SystemZ::STG))
1151:         .addReg(BCReg)
1152:         .addReg(BufReg)
1153:         .addImm(BCOffset)
1154:         .addReg(0);
1155:   }
1156: 
1157:   // Setup.
1158:   MIB = BuildMI(*ThisMBB, MI, DL, TII->get(SystemZ::EH_SjLj_Setup))
1159:             .addMBB(RestoreMBB);
1160: 
1161:   const SystemZRegisterInfo *RegInfo = Subtarget.getRegisterInfo();
1162:   MIB.addRegMask(RegInfo->getNoPreservedMask());
1163: 
1164:   ThisMBB->addSuccessor(MainMBB);
1165:   ThisMBB->addSuccessor(RestoreMBB);
1166: 
1167:   // mainMBB:
1168:   BuildMI(MainMBB, DL, TII->get(SystemZ::LHI), MainDstReg).addImm(0);
1169:   MainMBB->addSuccessor(SinkMBB);
1170: 
1171:   // sinkMBB:
1172:   BuildMI(*SinkMBB, SinkMBB->begin(), DL, TII->get(SystemZ::PHI), DstReg)
1173:       .addReg(MainDstReg)
1174:       .addMBB(MainMBB)
1175:       .addReg(RestoreDstReg)
1176:       .addMBB(RestoreMBB);
1177: 
1178:   // restoreMBB.
1179:   BuildMI(RestoreMBB, DL, TII->get(SystemZ::LHI), RestoreDstReg).addImm(1);
1180:   BuildMI(RestoreMBB, DL, TII->get(SystemZ::J)).addMBB(SinkMBB);
1181:   RestoreMBB->addSuccessor(SinkMBB);
1182: 
1183:   MI.eraseFromParent();
1184: 
1185:   return SinkMBB;
1186: }
1187: 
1188: MachineBasicBlock *
1189: SystemZTargetLowering::emitEHSjLjLongJmp(MachineInstr &MI,
1190:                                          MachineBasicBlock *MBB) const {
1191: 
1192:   DebugLoc DL = MI.getDebugLoc();
1193:   const TargetInstrInfo *TII = Subtarget.getInstrInfo();
1194: 
1195:   MachineFunction *MF = MBB->getParent();
1196:   MachineRegisterInfo &MRI = MF->getRegInfo();
1197: 
1198:   MVT PVT = getPointerTy(MF->getDataLayout());
1199:   assert((PVT == MVT::i64 || PVT == MVT::i32) && "Invalid Pointer Size!");
1200:   Register BufReg = MI.getOperand(0).getReg();
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `SystemZTargetLowering::emitEHSjLjLongJmp`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `SystemZTargetLowering::emitEHSjLjLongJmp` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1201-1275 / 第 1201-1275 行
```cpp
1201:   const TargetRegisterClass *RC = MRI.getRegClass(BufReg);
1202:   auto *SpecialRegs = Subtarget.getSpecialRegisters();
1203: 
1204:   Register Tmp = MRI.createVirtualRegister(RC);
1205:   Register BCReg = MRI.createVirtualRegister(RC);
1206: 
1207:   MachineInstrBuilder MIB;
1208: 
1209:   const int64_t FPOffset = 0;
1210:   const int64_t LabelOffset = 1 * PVT.getStoreSize();
1211:   const int64_t BCOffset = 2 * PVT.getStoreSize();
1212:   const int64_t SPOffset = 3 * PVT.getStoreSize();
1213:   const int64_t LPOffset = 4 * PVT.getStoreSize();
1214: 
1215:   MIB = BuildMI(*MBB, MI, DL, TII->get(SystemZ::LG), Tmp)
1216:             .addReg(BufReg)
1217:             .addImm(LabelOffset)
1218:             .addReg(0);
1219: 
1220:   MIB = BuildMI(*MBB, MI, DL, TII->get(SystemZ::LG),
1221:                 SpecialRegs->getFramePointerRegister())
1222:             .addReg(BufReg)
1223:             .addImm(FPOffset)
1224:             .addReg(0);
1225: 
1226:   // We are restoring R13 even though we never stored in setjmp from llvm,
1227:   // as gcc always stores R13 in builtin_setjmp. We could have mixed code
1228:   // gcc setjmp and llvm longjmp.
1229:   MIB = BuildMI(*MBB, MI, DL, TII->get(SystemZ::LG), SystemZ::R13D)
1230:             .addReg(BufReg)
1231:             .addImm(LPOffset)
1232:             .addReg(0);
1233: 
1234:   bool BackChain = MF->getSubtarget<SystemZSubtarget>().hasBackChain();
1235:   if (BackChain) {
1236:     MIB = BuildMI(*MBB, MI, DL, TII->get(SystemZ::LG), BCReg)
1237:               .addReg(BufReg)
1238:               .addImm(BCOffset)
1239:               .addReg(0);
1240:   }
1241: 
1242:   MIB = BuildMI(*MBB, MI, DL, TII->get(SystemZ::LG),
1243:                 SpecialRegs->getStackPointerRegister())
1244:             .addReg(BufReg)
1245:             .addImm(SPOffset)
1246:             .addReg(0);
1247: 
1248:   if (BackChain) {
1249:     auto *TFL = Subtarget.getFrameLowering<SystemZFrameLowering>();
1250:     BuildMI(*MBB, MI, DL, TII->get(SystemZ::STG))
1251:         .addReg(BCReg)
1252:         .addReg(SpecialRegs->getStackPointerRegister())
1253:         .addImm(TFL->getBackchainOffset(*MF))
1254:         .addReg(0);
1255:   }
1256: 
1257:   MIB = BuildMI(*MBB, MI, DL, TII->get(SystemZ::BR)).addReg(Tmp);
1258: 
1259:   MI.eraseFromParent();
1260:   return MBB;
1261: }
1262: 
1263: /// Returns true if stack probing through inline assembly is requested.
1264: bool SystemZTargetLowering::hasInlineStackProbe(const MachineFunction &MF) const {
1265:   // If the function specifically requests inline stack probes, emit them.
1266:   if (MF.getFunction().hasFnAttribute("probe-stack"))
1267:     return MF.getFunction().getFnAttribute("probe-stack").getValueAsString() ==
1268:            "inline-asm";
1269:   return false;
1270: }
1271: 
1272: TargetLowering::AtomicExpansionKind
1273: SystemZTargetLowering::shouldCastAtomicLoadInIR(LoadInst *LI) const {
1274:   return AtomicExpansionKind::None;
1275: }
```
- **EN**: The range implements or declares functions including `BuildMI`, `SystemZTargetLowering::hasInlineStackProbe`, `SystemZTargetLowering::shouldCastAtomicLoadInIR`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `SystemZTargetLowering::hasInlineStackProbe`, `SystemZTargetLowering::shouldCastAtomicLoadInIR` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1276-1350 / 第 1276-1350 行
```cpp
1276: 
1277: TargetLowering::AtomicExpansionKind
1278: SystemZTargetLowering::shouldCastAtomicStoreInIR(StoreInst *SI) const {
1279:   return AtomicExpansionKind::None;
1280: }
1281: 
1282: TargetLowering::AtomicExpansionKind
1283: SystemZTargetLowering::shouldExpandAtomicRMWInIR(
1284:     const AtomicRMWInst *RMW) const {
1285:   // Don't expand subword operations as they require special treatment.
1286:   if (RMW->getType()->isIntegerTy(8) || RMW->getType()->isIntegerTy(16))
1287:     return AtomicExpansionKind::None;
1288: 
1289:   // Don't expand if there is a target instruction available.
1290:   if (Subtarget.hasInterlockedAccess1() &&
1291:       (RMW->getType()->isIntegerTy(32) || RMW->getType()->isIntegerTy(64)) &&
1292:       (RMW->getOperation() == AtomicRMWInst::BinOp::Add ||
1293:        RMW->getOperation() == AtomicRMWInst::BinOp::Sub ||
1294:        RMW->getOperation() == AtomicRMWInst::BinOp::And ||
1295:        RMW->getOperation() == AtomicRMWInst::BinOp::Or ||
1296:        RMW->getOperation() == AtomicRMWInst::BinOp::Xor))
1297:     return AtomicExpansionKind::None;
1298: 
1299:   return AtomicExpansionKind::CmpXChg;
1300: }
1301: 
1302: bool SystemZTargetLowering::isLegalICmpImmediate(int64_t Imm) const {
1303:   // We can use CGFI or CLGFI.
1304:   return isInt<32>(Imm) || isUInt<32>(Imm);
1305: }
1306: 
1307: bool SystemZTargetLowering::isLegalAddImmediate(int64_t Imm) const {
1308:   // We can use ALGFI or SLGFI.
1309:   return isUInt<32>(Imm) || isUInt<32>(-Imm);
1310: }
1311: 
1312: bool SystemZTargetLowering::allowsMisalignedMemoryAccesses(
1313:     EVT VT, unsigned, Align, MachineMemOperand::Flags, unsigned *Fast) const {
1314:   // Unaligned accesses should never be slower than the expanded version.
1315:   // We check specifically for aligned accesses in the few cases where
1316:   // they are required.
1317:   if (Fast)
1318:     *Fast = 1;
1319:   return true;
1320: }
1321: 
1322: bool SystemZTargetLowering::hasAndNot(SDValue Y) const {
1323:   EVT VT = Y.getValueType();
1324: 
1325:   // We can use NC(G)RK for types in GPRs ...
1326:   if (VT == MVT::i32 || VT == MVT::i64)
1327:     return Subtarget.hasMiscellaneousExtensions3();
1328: 
1329:   // ... or VNC for types in VRs.
1330:   if (VT.isVector() || VT == MVT::i128)
1331:     return Subtarget.hasVector();
1332: 
1333:   return false;
1334: }
1335: 
1336: // Information about the addressing mode for a memory access.
1337: struct AddressingMode {
1338:   // True if a long displacement is supported.
1339:   bool LongDisplacement;
1340: 
1341:   // True if use of index register is supported.
1342:   bool IndexReg;
1343: 
1344:   AddressingMode(bool LongDispl, bool IdxReg) :
1345:     LongDisplacement(LongDispl), IndexReg(IdxReg) {}
1346: };
1347: 
1348: // Return the desired addressing mode for a Load which has only one use (in
1349: // the same block) which is a Store.
1350: static AddressingMode getLoadStoreAddrMode(bool HasVector,
```
- **EN**: It introduces interface types such as `AddressingMode`, shaping how other backend components interact with this file. The range implements or declares functions including `SystemZTargetLowering::shouldCastAtomicStoreInIR`, `SystemZTargetLowering::shouldExpandAtomicRMWInIR`, `SystemZTargetLowering::isLegalICmpImmediate`, `SystemZTargetLowering::isLegalAddImmediate`, `SystemZTargetLowering::allowsMisalignedMemoryAccesses`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `AddressingMode` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `SystemZTargetLowering::shouldCastAtomicStoreInIR`, `SystemZTargetLowering::shouldExpandAtomicRMWInIR`, `SystemZTargetLowering::isLegalICmpImmediate`, `SystemZTargetLowering::isLegalAddImmediate`, `SystemZTargetLowering::allowsMisalignedMemoryAccesses` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1351-1425 / 第 1351-1425 行
```cpp
1351:                                           Type *Ty) {
1352:   // With vector support a Load->Store combination may be combined to either
1353:   // an MVC or vector operations and it seems to work best to allow the
1354:   // vector addressing mode.
1355:   if (HasVector)
1356:     return AddressingMode(false/*LongDispl*/, true/*IdxReg*/);
1357: 
1358:   // Otherwise only the MVC case is special.
1359:   bool MVC = Ty->isIntegerTy(8);
1360:   return AddressingMode(!MVC/*LongDispl*/, !MVC/*IdxReg*/);
1361: }
1362: 
1363: // Return the addressing mode which seems most desirable given an LLVM
1364: // Instruction pointer.
1365: static AddressingMode
1366: supportedAddressingMode(Instruction *I, bool HasVector) {
1367:   if (IntrinsicInst *II = dyn_cast<IntrinsicInst>(I)) {
1368:     switch (II->getIntrinsicID()) {
1369:     default: break;
1370:     case Intrinsic::memset:
1371:     case Intrinsic::memmove:
1372:     case Intrinsic::memcpy:
1373:       return AddressingMode(false/*LongDispl*/, false/*IdxReg*/);
1374:     }
1375:   }
1376: 
1377:   if (isa<LoadInst>(I) && I->hasOneUse()) {
1378:     auto *SingleUser = cast<Instruction>(*I->user_begin());
1379:     if (SingleUser->getParent() == I->getParent()) {
1380:       if (isa<ICmpInst>(SingleUser)) {
1381:         if (auto *C = dyn_cast<ConstantInt>(SingleUser->getOperand(1)))
1382:           if (C->getBitWidth() <= 64 &&
1383:               (isInt<16>(C->getSExtValue()) || isUInt<16>(C->getZExtValue())))
1384:             // Comparison of memory with 16 bit signed / unsigned immediate
1385:             return AddressingMode(false/*LongDispl*/, false/*IdxReg*/);
1386:       } else if (isa<StoreInst>(SingleUser))
1387:         // Load->Store
1388:         return getLoadStoreAddrMode(HasVector, I->getType());
1389:     }
1390:   } else if (auto *StoreI = dyn_cast<StoreInst>(I)) {
1391:     if (auto *LoadI = dyn_cast<LoadInst>(StoreI->getValueOperand()))
1392:       if (LoadI->hasOneUse() && LoadI->getParent() == I->getParent())
1393:         // Load->Store
1394:         return getLoadStoreAddrMode(HasVector, LoadI->getType());
1395:   }
1396: 
1397:   if (HasVector && (isa<LoadInst>(I) || isa<StoreInst>(I))) {
1398: 
1399:     // * Use LDE instead of LE/LEY for z13 to avoid partial register
1400:     //   dependencies (LDE only supports small offsets).
1401:     // * Utilize the vector registers to hold floating point
1402:     //   values (vector load / store instructions only support small
1403:     //   offsets).
1404: 
1405:     Type *MemAccessTy = (isa<LoadInst>(I) ? I->getType() :
1406:                          I->getOperand(0)->getType());
1407:     bool IsFPAccess = MemAccessTy->isFloatingPointTy();
1408:     bool IsVectorAccess = MemAccessTy->isVectorTy();
1409: 
1410:     // A store of an extracted vector element will be combined into a VSTE type
1411:     // instruction.
1412:     if (!IsVectorAccess && isa<StoreInst>(I)) {
1413:       Value *DataOp = I->getOperand(0);
1414:       if (isa<ExtractElementInst>(DataOp))
1415:         IsVectorAccess = true;
1416:     }
1417: 
1418:     // A load which gets inserted into a vector element will be combined into a
1419:     // VLE type instruction.
1420:     if (!IsVectorAccess && isa<LoadInst>(I) && I->hasOneUse()) {
1421:       User *LoadUser = *I->user_begin();
1422:       if (isa<InsertElementInst>(LoadUser))
1423:         IsVectorAccess = true;
1424:     }
1425: 
```
- **EN**: The range implements or declares functions including `supportedAddressingMode`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `supportedAddressingMode` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1426-1500 / 第 1426-1500 行
```cpp
1426:     if (IsFPAccess || IsVectorAccess)
1427:       return AddressingMode(false/*LongDispl*/, true/*IdxReg*/);
1428:   }
1429: 
1430:   return AddressingMode(true/*LongDispl*/, true/*IdxReg*/);
1431: }
1432: 
1433: bool SystemZTargetLowering::isLegalAddressingMode(const DataLayout &DL,
1434:        const AddrMode &AM, Type *Ty, unsigned AS, Instruction *I) const {
1435:   // Punt on globals for now, although they can be used in limited
1436:   // RELATIVE LONG cases.
1437:   if (AM.BaseGV)
1438:     return false;
1439: 
1440:   // Require a 20-bit signed offset.
1441:   if (!isInt<20>(AM.BaseOffs))
1442:     return false;
1443: 
1444:   bool RequireD12 =
1445:       Subtarget.hasVector() && (Ty->isVectorTy() || Ty->isIntegerTy(128));
1446:   AddressingMode SupportedAM(!RequireD12, true);
1447:   if (I != nullptr)
1448:     SupportedAM = supportedAddressingMode(I, Subtarget.hasVector());
1449: 
1450:   if (!SupportedAM.LongDisplacement && !isUInt<12>(AM.BaseOffs))
1451:     return false;
1452: 
1453:   if (!SupportedAM.IndexReg)
1454:     // No indexing allowed.
1455:     return AM.Scale == 0;
1456:   else
1457:     // Indexing is OK but no scale factor can be applied.
1458:     return AM.Scale == 0 || AM.Scale == 1;
1459: }
1460: 
1461: bool SystemZTargetLowering::findOptimalMemOpLowering(
1462:     LLVMContext &Context, std::vector<EVT> &MemOps, unsigned Limit,
1463:     const MemOp &Op, unsigned DstAS, unsigned SrcAS,
1464:     const AttributeList &FuncAttributes, EVT *LargestVT) const {
1465: 
1466:   assert(Limit != ~0U &&
1467:          "Expected EmitTargetCodeForMemXXX() to handle AlwaysInline cases.");
1468: 
1469:   if (Op.isZeroMemset())
1470:     return false; // Memset zero: Use XC.
1471: 
1472:   const int MVCFastLen = 16;
1473:   // Use MVC up to 16 bytes.  Small memset uses STC/MVI for first byte.
1474:   if ((Op.isMemset() ? Op.size() - 1 : Op.size()) <= MVCFastLen)
1475:     return false;
1476: 
1477:   // Avoid unaligned VL/VST:s.
1478:   if (!Op.isAligned(Align(8)) || (Op.size() >= 25 && Op.size() <= 31))
1479:     return false;
1480: 
1481:   return TargetLowering::findOptimalMemOpLowering(
1482:       Context, MemOps, Limit, Op, DstAS, SrcAS, FuncAttributes, LargestVT);
1483: }
1484: 
1485: EVT SystemZTargetLowering::getOptimalMemOpType(
1486:     LLVMContext &Context, const MemOp &Op,
1487:     const AttributeList &FuncAttributes) const {
1488:   return Subtarget.hasVector() ? MVT::v2i64 : MVT::Other;
1489: }
1490: 
1491: bool SystemZTargetLowering::isTruncateFree(Type *FromType, Type *ToType) const {
1492:   if (!FromType->isIntegerTy() || !ToType->isIntegerTy())
1493:     return false;
1494:   unsigned FromBits = FromType->getPrimitiveSizeInBits().getFixedValue();
1495:   unsigned ToBits = ToType->getPrimitiveSizeInBits().getFixedValue();
1496:   return FromBits > ToBits;
1497: }
1498: 
1499: bool SystemZTargetLowering::isTruncateFree(EVT FromVT, EVT ToVT) const {
1500:   if (!FromVT.isInteger() || !ToVT.isInteger())
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::isLegalAddressingMode`, `SystemZTargetLowering::findOptimalMemOpLowering`, `SystemZTargetLowering::getOptimalMemOpType`, `SystemZTargetLowering::isTruncateFree`, `SystemZTargetLowering::isTruncateFree`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::isLegalAddressingMode`, `SystemZTargetLowering::findOptimalMemOpLowering`, `SystemZTargetLowering::getOptimalMemOpType`, `SystemZTargetLowering::isTruncateFree`, `SystemZTargetLowering::isTruncateFree` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1501-1575 / 第 1501-1575 行
```cpp
1501:     return false;
1502:   unsigned FromBits = FromVT.getFixedSizeInBits();
1503:   unsigned ToBits = ToVT.getFixedSizeInBits();
1504:   return FromBits > ToBits;
1505: }
1506: 
1507: //===----------------------------------------------------------------------===//
1508: // Inline asm support
1509: //===----------------------------------------------------------------------===//
1510: 
1511: TargetLowering::ConstraintType
1512: SystemZTargetLowering::getConstraintType(StringRef Constraint) const {
1513:   if (Constraint.size() == 1) {
1514:     switch (Constraint[0]) {
1515:     case 'a': // Address register
1516:     case 'd': // Data register (equivalent to 'r')
1517:     case 'f': // Floating-point register
1518:     case 'h': // High-part register
1519:     case 'r': // General-purpose register
1520:     case 'v': // Vector register
1521:       return C_RegisterClass;
1522: 
1523:     case 'Q': // Memory with base and unsigned 12-bit displacement
1524:     case 'R': // Likewise, plus an index
1525:     case 'S': // Memory with base and signed 20-bit displacement
1526:     case 'T': // Likewise, plus an index
1527:     case 'm': // Equivalent to 'T'.
1528:       return C_Memory;
1529: 
1530:     case 'I': // Unsigned 8-bit constant
1531:     case 'J': // Unsigned 12-bit constant
1532:     case 'K': // Signed 16-bit constant
1533:     case 'L': // Signed 20-bit displacement (on all targets we support)
1534:     case 'M': // 0x7fffffff
1535:       return C_Immediate;
1536: 
1537:     default:
1538:       break;
1539:     }
1540:   } else if (Constraint.size() == 2 && Constraint[0] == 'Z') {
1541:     switch (Constraint[1]) {
1542:     case 'Q': // Address with base and unsigned 12-bit displacement
1543:     case 'R': // Likewise, plus an index
1544:     case 'S': // Address with base and signed 20-bit displacement
1545:     case 'T': // Likewise, plus an index
1546:       return C_Address;
1547: 
1548:     default:
1549:       break;
1550:     }
1551:   } else if (Constraint.size() == 5 && Constraint.starts_with("{")) {
1552:     if (StringRef("{@cc}").compare(Constraint) == 0)
1553:       return C_Other;
1554:   }
1555:   return TargetLowering::getConstraintType(Constraint);
1556: }
1557: 
1558: TargetLowering::ConstraintWeight
1559: SystemZTargetLowering::getSingleConstraintMatchWeight(
1560:     AsmOperandInfo &Info, const char *Constraint) const {
1561:   ConstraintWeight Weight = CW_Invalid;
1562:   Value *CallOperandVal = Info.CallOperandVal;
1563:   // If we don't have a value, we can't do a match,
1564:   // but allow it at the lowest weight.
1565:   if (!CallOperandVal)
1566:     return CW_Default;
1567:   Type *type = CallOperandVal->getType();
1568:   // Look at the constraint type.
1569:   switch (*Constraint) {
1570:   default:
1571:     Weight = TargetLowering::getSingleConstraintMatchWeight(Info, Constraint);
1572:     break;
1573: 
1574:   case 'a': // Address register
1575:   case 'd': // Data register (equivalent to 'r')
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::getConstraintType`, `SystemZTargetLowering::getSingleConstraintMatchWeight`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::getConstraintType`, `SystemZTargetLowering::getSingleConstraintMatchWeight` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1576-1650 / 第 1576-1650 行
```cpp
1576:   case 'h': // High-part register
1577:   case 'r': // General-purpose register
1578:     Weight =
1579:         CallOperandVal->getType()->isIntegerTy() ? CW_Register : CW_Default;
1580:     break;
1581: 
1582:   case 'f': // Floating-point register
1583:     if (!useSoftFloat())
1584:       Weight = type->isFloatingPointTy() ? CW_Register : CW_Default;
1585:     break;
1586: 
1587:   case 'v': // Vector register
1588:     if (Subtarget.hasVector())
1589:       Weight = (type->isVectorTy() || type->isFloatingPointTy()) ? CW_Register
1590:                                                                  : CW_Default;
1591:     break;
1592: 
1593:   case 'I': // Unsigned 8-bit constant
1594:     if (auto *C = dyn_cast<ConstantInt>(CallOperandVal))
1595:       if (isUInt<8>(C->getZExtValue()))
1596:         Weight = CW_Constant;
1597:     break;
1598: 
1599:   case 'J': // Unsigned 12-bit constant
1600:     if (auto *C = dyn_cast<ConstantInt>(CallOperandVal))
1601:       if (isUInt<12>(C->getZExtValue()))
1602:         Weight = CW_Constant;
1603:     break;
1604: 
1605:   case 'K': // Signed 16-bit constant
1606:     if (auto *C = dyn_cast<ConstantInt>(CallOperandVal))
1607:       if (isInt<16>(C->getSExtValue()))
1608:         Weight = CW_Constant;
1609:     break;
1610: 
1611:   case 'L': // Signed 20-bit displacement (on all targets we support)
1612:     if (auto *C = dyn_cast<ConstantInt>(CallOperandVal))
1613:       if (isInt<20>(C->getSExtValue()))
1614:         Weight = CW_Constant;
1615:     break;
1616: 
1617:   case 'M': // 0x7fffffff
1618:     if (auto *C = dyn_cast<ConstantInt>(CallOperandVal))
1619:       if (C->getZExtValue() == 0x7fffffff)
1620:         Weight = CW_Constant;
1621:     break;
1622:   }
1623:   return Weight;
1624: }
1625: 
1626: // Parse a "{tNNN}" register constraint for which the register type "t"
1627: // has already been verified.  MC is the class associated with "t" and
1628: // Map maps 0-based register numbers to LLVM register numbers.
1629: static std::pair<unsigned, const TargetRegisterClass *>
1630: parseRegisterNumber(StringRef Constraint, const TargetRegisterClass *RC,
1631:                     const unsigned *Map, unsigned Size) {
1632:   assert(*(Constraint.end()-1) == '}' && "Missing '}'");
1633:   if (isdigit(Constraint[2])) {
1634:     unsigned Index;
1635:     bool Failed =
1636:         Constraint.slice(2, Constraint.size() - 1).getAsInteger(10, Index);
1637:     if (!Failed && Index < Size && Map[Index])
1638:       return std::make_pair(Map[Index], RC);
1639:   }
1640:   return std::make_pair(0U, nullptr);
1641: }
1642: 
1643: std::pair<unsigned, const TargetRegisterClass *>
1644: SystemZTargetLowering::getRegForInlineAsmConstraint(
1645:     const TargetRegisterInfo *TRI, StringRef Constraint, MVT VT) const {
1646:   if (Constraint.size() == 1) {
1647:     // GCC Constraint Letters
1648:     switch (Constraint[0]) {
1649:     default: break;
1650:     case 'd': // Data register (equivalent to 'r')
```
- **EN**: The range implements or declares functions including `parseRegisterNumber`, `SystemZTargetLowering::getRegForInlineAsmConstraint`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `parseRegisterNumber`, `SystemZTargetLowering::getRegForInlineAsmConstraint` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1651-1725 / 第 1651-1725 行
```cpp
1651:     case 'r': // General-purpose register
1652:       if (VT.getSizeInBits() == 64)
1653:         return std::make_pair(0U, &SystemZ::GR64BitRegClass);
1654:       else if (VT.getSizeInBits() == 128)
1655:         return std::make_pair(0U, &SystemZ::GR128BitRegClass);
1656:       return std::make_pair(0U, &SystemZ::GR32BitRegClass);
1657: 
1658:     case 'a': // Address register
1659:       if (VT == MVT::i64)
1660:         return std::make_pair(0U, &SystemZ::ADDR64BitRegClass);
1661:       else if (VT == MVT::i128)
1662:         return std::make_pair(0U, &SystemZ::ADDR128BitRegClass);
1663:       return std::make_pair(0U, &SystemZ::ADDR32BitRegClass);
1664: 
1665:     case 'h': // High-part register (an LLVM extension)
1666:       return std::make_pair(0U, &SystemZ::GRH32BitRegClass);
1667: 
1668:     case 'f': // Floating-point register
1669:       if (!useSoftFloat()) {
1670:         if (VT.getSizeInBits() == 16)
1671:           return std::make_pair(0U, &SystemZ::FP16BitRegClass);
1672:         else if (VT.getSizeInBits() == 64)
1673:           return std::make_pair(0U, &SystemZ::FP64BitRegClass);
1674:         else if (VT.getSizeInBits() == 128)
1675:           return std::make_pair(0U, &SystemZ::FP128BitRegClass);
1676:         return std::make_pair(0U, &SystemZ::FP32BitRegClass);
1677:       }
1678:       break;
1679: 
1680:     case 'v': // Vector register
1681:       if (Subtarget.hasVector()) {
1682:         if (VT.getSizeInBits() == 16)
1683:           return std::make_pair(0U, &SystemZ::VR16BitRegClass);
1684:         if (VT.getSizeInBits() == 32)
1685:           return std::make_pair(0U, &SystemZ::VR32BitRegClass);
1686:         if (VT.getSizeInBits() == 64)
1687:           return std::make_pair(0U, &SystemZ::VR64BitRegClass);
1688:         return std::make_pair(0U, &SystemZ::VR128BitRegClass);
1689:       }
1690:       break;
1691:     }
1692:   }
1693:   if (Constraint.starts_with("{")) {
1694: 
1695:     // A clobber constraint (e.g. ~{f0}) will have MVT::Other which is illegal
1696:     // to check the size on.
1697:     auto getVTSizeInBits = [&VT]() {
1698:       return VT == MVT::Other ? 0 : VT.getSizeInBits();
1699:     };
1700: 
1701:     // We need to override the default register parsing for GPRs and FPRs
1702:     // because the interpretation depends on VT.  The internal names of
1703:     // the registers are also different from the external names
1704:     // (F0D and F0S instead of F0, etc.).
1705:     if (Constraint[1] == 'r') {
1706:       if (getVTSizeInBits() == 32)
1707:         return parseRegisterNumber(Constraint, &SystemZ::GR32BitRegClass,
1708:                                    SystemZMC::GR32Regs, 16);
1709:       if (getVTSizeInBits() == 128)
1710:         return parseRegisterNumber(Constraint, &SystemZ::GR128BitRegClass,
1711:                                    SystemZMC::GR128Regs, 16);
1712:       return parseRegisterNumber(Constraint, &SystemZ::GR64BitRegClass,
1713:                                  SystemZMC::GR64Regs, 16);
1714:     }
1715:     if (Constraint[1] == 'f') {
1716:       if (useSoftFloat())
1717:         return std::make_pair(
1718:             0u, static_cast<const TargetRegisterClass *>(nullptr));
1719:       if (getVTSizeInBits() == 16)
1720:         return parseRegisterNumber(Constraint, &SystemZ::FP16BitRegClass,
1721:                                    SystemZMC::FP16Regs, 16);
1722:       if (getVTSizeInBits() == 32)
1723:         return parseRegisterNumber(Constraint, &SystemZ::FP32BitRegClass,
1724:                                    SystemZMC::FP32Regs, 16);
1725:       if (getVTSizeInBits() == 128)
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1726-1800 / 第 1726-1800 行
```cpp
1726:         return parseRegisterNumber(Constraint, &SystemZ::FP128BitRegClass,
1727:                                    SystemZMC::FP128Regs, 16);
1728:       return parseRegisterNumber(Constraint, &SystemZ::FP64BitRegClass,
1729:                                  SystemZMC::FP64Regs, 16);
1730:     }
1731:     if (Constraint[1] == 'v') {
1732:       if (!Subtarget.hasVector())
1733:         return std::make_pair(
1734:             0u, static_cast<const TargetRegisterClass *>(nullptr));
1735:       if (getVTSizeInBits() == 16)
1736:         return parseRegisterNumber(Constraint, &SystemZ::VR16BitRegClass,
1737:                                    SystemZMC::VR16Regs, 32);
1738:       if (getVTSizeInBits() == 32)
1739:         return parseRegisterNumber(Constraint, &SystemZ::VR32BitRegClass,
1740:                                    SystemZMC::VR32Regs, 32);
1741:       if (getVTSizeInBits() == 64)
1742:         return parseRegisterNumber(Constraint, &SystemZ::VR64BitRegClass,
1743:                                    SystemZMC::VR64Regs, 32);
1744:       return parseRegisterNumber(Constraint, &SystemZ::VR128BitRegClass,
1745:                                  SystemZMC::VR128Regs, 32);
1746:     }
1747:     if (Constraint[1] == '@') {
1748:       if (StringRef("{@cc}").compare(Constraint) == 0)
1749:         return std::make_pair(SystemZ::CC, &SystemZ::CCRRegClass);
1750:     }
1751:   }
1752:   return TargetLowering::getRegForInlineAsmConstraint(TRI, Constraint, VT);
1753: }
1754: 
1755: // FIXME? Maybe this could be a TableGen attribute on some registers and
1756: // this table could be generated automatically from RegInfo.
1757: Register
1758: SystemZTargetLowering::getRegisterByName(const char *RegName, LLT VT,
1759:                                          const MachineFunction &MF) const {
1760:   Register Reg =
1761:       StringSwitch<Register>(RegName)
1762:           .Case("r4", Subtarget.isTargetXPLINK64() ? SystemZ::R4D
1763:                                                    : SystemZ::NoRegister)
1764:           .Case("r15",
1765:                 Subtarget.isTargetELF() ? SystemZ::R15D : SystemZ::NoRegister)
1766:           .Default(Register());
1767: 
1768:   return Reg;
1769: }
1770: 
1771: Register SystemZTargetLowering::getExceptionPointerRegister(
1772:     const Constant *PersonalityFn) const {
1773:   return Subtarget.isTargetXPLINK64() ? SystemZ::R1D : SystemZ::R6D;
1774: }
1775: 
1776: Register SystemZTargetLowering::getExceptionSelectorRegister(
1777:     const Constant *PersonalityFn) const {
1778:   return Subtarget.isTargetXPLINK64() ? SystemZ::R2D : SystemZ::R7D;
1779: }
1780: 
1781: // Convert condition code in CCReg to an i32 value.
1782: static SDValue getCCResult(SelectionDAG &DAG, SDValue CCReg) {
1783:   SDLoc DL(CCReg);
1784:   SDValue IPM = DAG.getNode(SystemZISD::IPM, DL, MVT::i32, CCReg);
1785:   return DAG.getNode(ISD::SRL, DL, MVT::i32, IPM,
1786:                      DAG.getConstant(SystemZ::IPM_CC, DL, MVT::i32));
1787: }
1788: 
1789: // Lower @cc targets via setcc.
1790: SDValue SystemZTargetLowering::LowerAsmOutputForConstraint(
1791:     SDValue &Chain, SDValue &Glue, const SDLoc &DL,
1792:     const AsmOperandInfo &OpInfo, SelectionDAG &DAG) const {
1793:   if (StringRef("{@cc}").compare(OpInfo.ConstraintCode) != 0)
1794:     return SDValue();
1795: 
1796:   // Check that return type is valid.
1797:   if (OpInfo.ConstraintVT.isVector() || !OpInfo.ConstraintVT.isInteger() ||
1798:       OpInfo.ConstraintVT.getSizeInBits() < 8)
1799:     report_fatal_error("Glue output operand is of invalid type");
1800: 
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::getRegisterByName`, `SystemZTargetLowering::getExceptionPointerRegister`, `SystemZTargetLowering::getExceptionSelectorRegister`, `getCCResult`, `SystemZTargetLowering::LowerAsmOutputForConstraint`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::getRegisterByName`, `SystemZTargetLowering::getExceptionPointerRegister`, `SystemZTargetLowering::getExceptionSelectorRegister`, `getCCResult`, `SystemZTargetLowering::LowerAsmOutputForConstraint` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1801-1875 / 第 1801-1875 行
```cpp
1801:   if (Glue.getNode()) {
1802:     Glue = DAG.getCopyFromReg(Chain, DL, SystemZ::CC, MVT::i32, Glue);
1803:     Chain = Glue.getValue(1);
1804:   } else
1805:     Glue = DAG.getCopyFromReg(Chain, DL, SystemZ::CC, MVT::i32);
1806:   return getCCResult(DAG, Glue);
1807: }
1808: 
1809: void SystemZTargetLowering::LowerAsmOperandForConstraint(
1810:     SDValue Op, StringRef Constraint, std::vector<SDValue> &Ops,
1811:     SelectionDAG &DAG) const {
1812:   // Only support length 1 constraints for now.
1813:   if (Constraint.size() == 1) {
1814:     switch (Constraint[0]) {
1815:     case 'I': // Unsigned 8-bit constant
1816:       if (auto *C = dyn_cast<ConstantSDNode>(Op))
1817:         if (isUInt<8>(C->getZExtValue()))
1818:           Ops.push_back(DAG.getTargetConstant(C->getZExtValue(), SDLoc(Op),
1819:                                               Op.getValueType()));
1820:       return;
1821: 
1822:     case 'J': // Unsigned 12-bit constant
1823:       if (auto *C = dyn_cast<ConstantSDNode>(Op))
1824:         if (isUInt<12>(C->getZExtValue()))
1825:           Ops.push_back(DAG.getTargetConstant(C->getZExtValue(), SDLoc(Op),
1826:                                               Op.getValueType()));
1827:       return;
1828: 
1829:     case 'K': // Signed 16-bit constant
1830:       if (auto *C = dyn_cast<ConstantSDNode>(Op))
1831:         if (isInt<16>(C->getSExtValue()))
1832:           Ops.push_back(DAG.getSignedTargetConstant(
1833:               C->getSExtValue(), SDLoc(Op), Op.getValueType()));
1834:       return;
1835: 
1836:     case 'L': // Signed 20-bit displacement (on all targets we support)
1837:       if (auto *C = dyn_cast<ConstantSDNode>(Op))
1838:         if (isInt<20>(C->getSExtValue()))
1839:           Ops.push_back(DAG.getSignedTargetConstant(
1840:               C->getSExtValue(), SDLoc(Op), Op.getValueType()));
1841:       return;
1842: 
1843:     case 'M': // 0x7fffffff
1844:       if (auto *C = dyn_cast<ConstantSDNode>(Op))
1845:         if (C->getZExtValue() == 0x7fffffff)
1846:           Ops.push_back(DAG.getTargetConstant(C->getZExtValue(), SDLoc(Op),
1847:                                               Op.getValueType()));
1848:       return;
1849:     }
1850:   }
1851:   TargetLowering::LowerAsmOperandForConstraint(Op, Constraint, Ops, DAG);
1852: }
1853: 
1854: //===----------------------------------------------------------------------===//
1855: // Calling conventions
1856: //===----------------------------------------------------------------------===//
1857: 
1858: #include "SystemZGenCallingConv.inc"
1859: 
1860: const MCPhysReg *SystemZTargetLowering::getScratchRegisters(
1861:   CallingConv::ID) const {
1862:   static const MCPhysReg ScratchRegs[] = { SystemZ::R0D, SystemZ::R1D,
1863:                                            SystemZ::R14D, 0 };
1864:   return ScratchRegs;
1865: }
1866: 
1867: bool SystemZTargetLowering::allowTruncateForTailCall(Type *FromType,
1868:                                                      Type *ToType) const {
1869:   return isTruncateFree(FromType, ToType);
1870: }
1871: 
1872: bool SystemZTargetLowering::mayBeEmittedAsTailCall(const CallInst *CI) const {
1873:   return CI->isTailCall();
1874: }
1875: 
```
- **EN**: It imports dependencies such as `SystemZGenCallingConv.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `SystemZTargetLowering::LowerAsmOperandForConstraint`, `SystemZTargetLowering::allowTruncateForTailCall`, `SystemZTargetLowering::mayBeEmittedAsTailCall`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 它引入了 `SystemZGenCallingConv.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `SystemZTargetLowering::LowerAsmOperandForConstraint`, `SystemZTargetLowering::allowTruncateForTailCall`, `SystemZTargetLowering::mayBeEmittedAsTailCall` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 1876-1950 / 第 1876-1950 行
```cpp
1876: // Value is a value that has been passed to us in the location described by VA
1877: // (and so has type VA.getLocVT()).  Convert Value to VA.getValVT(), chaining
1878: // any loads onto Chain.
1879: static SDValue convertLocVTToValVT(SelectionDAG &DAG, const SDLoc &DL,
1880:                                    CCValAssign &VA, SDValue Chain,
1881:                                    SDValue Value) {
1882:   // If the argument has been promoted from a smaller type, insert an
1883:   // assertion to capture this.
1884:   if (VA.getLocInfo() == CCValAssign::SExt)
1885:     Value = DAG.getNode(ISD::AssertSext, DL, VA.getLocVT(), Value,
1886:                         DAG.getValueType(VA.getValVT()));
1887:   else if (VA.getLocInfo() == CCValAssign::ZExt)
1888:     Value = DAG.getNode(ISD::AssertZext, DL, VA.getLocVT(), Value,
1889:                         DAG.getValueType(VA.getValVT()));
1890: 
1891:   if (VA.isExtInLoc())
1892:     Value = DAG.getNode(ISD::TRUNCATE, DL, VA.getValVT(), Value);
1893:   else if (VA.getLocInfo() == CCValAssign::BCvt) {
1894:     // If this is a short vector argument loaded from the stack,
1895:     // extend from i64 to full vector size and then bitcast.
1896:     assert(VA.getLocVT() == MVT::i64);
1897:     assert(VA.getValVT().isVector());
1898:     Value = DAG.getBuildVector(MVT::v2i64, DL, {Value, DAG.getUNDEF(MVT::i64)});
1899:     Value = DAG.getNode(ISD::BITCAST, DL, VA.getValVT(), Value);
1900:   } else
1901:     assert(VA.getLocInfo() == CCValAssign::Full && "Unsupported getLocInfo");
1902:   return Value;
1903: }
1904: 
1905: // Value is a value of type VA.getValVT() that we need to copy into
1906: // the location described by VA.  Return a copy of Value converted to
1907: // VA.getValVT().  The caller is responsible for handling indirect values.
1908: static SDValue convertValVTToLocVT(SelectionDAG &DAG, const SDLoc &DL,
1909:                                    CCValAssign &VA, SDValue Value) {
1910:   switch (VA.getLocInfo()) {
1911:   case CCValAssign::SExt:
1912:     return DAG.getNode(ISD::SIGN_EXTEND, DL, VA.getLocVT(), Value);
1913:   case CCValAssign::ZExt:
1914:     return DAG.getNode(ISD::ZERO_EXTEND, DL, VA.getLocVT(), Value);
1915:   case CCValAssign::AExt:
1916:     return DAG.getNode(ISD::ANY_EXTEND, DL, VA.getLocVT(), Value);
1917:   case CCValAssign::BCvt: {
1918:     assert(VA.getLocVT() == MVT::i64 || VA.getLocVT() == MVT::i128);
1919:     assert(VA.getValVT().isVector() || VA.getValVT() == MVT::f32 ||
1920:            VA.getValVT() == MVT::f64 || VA.getValVT() == MVT::f128);
1921:     // For an f32 vararg we need to first promote it to an f64 and then
1922:     // bitcast it to an i64.
1923:     if (VA.getValVT() == MVT::f32 && VA.getLocVT() == MVT::i64)
1924:       Value = DAG.getNode(ISD::FP_EXTEND, DL, MVT::f64, Value);
1925:     MVT BitCastToType = VA.getValVT().isVector() && VA.getLocVT() == MVT::i64
1926:                             ? MVT::v2i64
1927:                             : VA.getLocVT();
1928:     Value = DAG.getNode(ISD::BITCAST, DL, BitCastToType, Value);
1929:     // For ELF, this is a short vector argument to be stored to the stack,
1930:     // bitcast to v2i64 and then extract first element.
1931:     if (BitCastToType == MVT::v2i64)
1932:       return DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, VA.getLocVT(), Value,
1933:                          DAG.getConstant(0, DL, MVT::i32));
1934:     return Value;
1935:   }
1936:   case CCValAssign::Full:
1937:     return Value;
1938:   default:
1939:     llvm_unreachable("Unhandled getLocInfo()");
1940:   }
1941: }
1942: 
1943: static SDValue lowerI128ToGR128(SelectionDAG &DAG, SDValue In) {
1944:   SDLoc DL(In);
1945:   SDValue Lo, Hi;
1946:   if (DAG.getTargetLoweringInfo().isTypeLegal(MVT::i128)) {
1947:     Lo = DAG.getNode(ISD::TRUNCATE, DL, MVT::i64, In);
1948:     Hi = DAG.getNode(ISD::TRUNCATE, DL, MVT::i64,
1949:                      DAG.getNode(ISD::SRL, DL, MVT::i128, In,
1950:                                  DAG.getConstant(64, DL, MVT::i32)));
```
- **EN**: The range implements or declares functions including `convertLocVTToValVT`, `convertValVTToLocVT`, `lowerI128ToGR128`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `convertLocVTToValVT`, `convertValVTToLocVT`, `lowerI128ToGR128` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1951-2025 / 第 1951-2025 行
```cpp
1951:   } else {
1952:     std::tie(Lo, Hi) = DAG.SplitScalar(In, DL, MVT::i64, MVT::i64);
1953:   }
1954: 
1955:   // FIXME: If v2i64 were a legal type, we could use it instead of
1956:   // Untyped here.  This might enable improved folding.
1957:   SDNode *Pair = DAG.getMachineNode(SystemZ::PAIR128, DL,
1958:                                     MVT::Untyped, Hi, Lo);
1959:   return SDValue(Pair, 0);
1960: }
1961: 
1962: static SDValue lowerGR128ToI128(SelectionDAG &DAG, SDValue In) {
1963:   SDLoc DL(In);
1964:   SDValue Hi = DAG.getTargetExtractSubreg(SystemZ::subreg_h64,
1965:                                           DL, MVT::i64, In);
1966:   SDValue Lo = DAG.getTargetExtractSubreg(SystemZ::subreg_l64,
1967:                                           DL, MVT::i64, In);
1968: 
1969:   if (DAG.getTargetLoweringInfo().isTypeLegal(MVT::i128)) {
1970:     Lo = DAG.getNode(ISD::ZERO_EXTEND, DL, MVT::i128, Lo);
1971:     Hi = DAG.getNode(ISD::ZERO_EXTEND, DL, MVT::i128, Hi);
1972:     Hi = DAG.getNode(ISD::SHL, DL, MVT::i128, Hi,
1973:                      DAG.getConstant(64, DL, MVT::i32));
1974:     return DAG.getNode(ISD::OR, DL, MVT::i128, Lo, Hi);
1975:   } else {
1976:     return DAG.getNode(ISD::BUILD_PAIR, DL, MVT::i128, Lo, Hi);
1977:   }
1978: }
1979: 
1980: bool SystemZTargetLowering::splitValueIntoRegisterParts(
1981:     SelectionDAG &DAG, const SDLoc &DL, SDValue Val, SDValue *Parts,
1982:     unsigned NumParts, MVT PartVT, std::optional<CallingConv::ID> CC) const {
1983:   EVT ValueVT = Val.getValueType();
1984:   if (ValueVT.getSizeInBits() == 128 && NumParts == 1 && PartVT == MVT::Untyped) {
1985:     // Inline assembly operand.
1986:     Parts[0] = lowerI128ToGR128(DAG, DAG.getBitcast(MVT::i128, Val));
1987:     return true;
1988:   }
1989: 
1990:   return false;
1991: }
1992: 
1993: SDValue SystemZTargetLowering::joinRegisterPartsIntoValue(
1994:     SelectionDAG &DAG, const SDLoc &DL, const SDValue *Parts, unsigned NumParts,
1995:     MVT PartVT, EVT ValueVT, std::optional<CallingConv::ID> CC) const {
1996:   if (ValueVT.getSizeInBits() == 128 && NumParts == 1 && PartVT == MVT::Untyped) {
1997:     // Inline assembly operand.
1998:     SDValue Res = lowerGR128ToI128(DAG, Parts[0]);
1999:     return DAG.getBitcast(ValueVT, Res);
2000:   }
2001: 
2002:   return SDValue();
2003: }
2004: 
2005: // The first part of a split stack argument is at index I in Args (and
2006: // ArgLocs). Return the type of a part and the number of them by reference.
2007: template <class ArgTy>
2008: static bool analyzeArgSplit(const SmallVectorImpl<ArgTy> &Args,
2009:                             SmallVector<CCValAssign, 16> &ArgLocs, unsigned I,
2010:                             MVT &PartVT, unsigned &NumParts) {
2011:   if (!Args[I].Flags.isSplit())
2012:     return false;
2013:   assert(I < ArgLocs.size() && ArgLocs.size() == Args.size() &&
2014:          "ArgLocs havoc.");
2015:   PartVT = ArgLocs[I].getValVT();
2016:   NumParts = 1;
2017:   for (unsigned PartIdx = I + 1;; ++PartIdx) {
2018:     assert(PartIdx != ArgLocs.size() && "SplitEnd not found.");
2019:     assert(ArgLocs[PartIdx].getValVT() == PartVT && "Unsupported split.");
2020:     ++NumParts;
2021:     if (Args[PartIdx].Flags.isSplitEnd())
2022:       break;
2023:   }
2024:   return true;
2025: }
```
- **EN**: The range implements or declares functions including `lowerGR128ToI128`, `SystemZTargetLowering::splitValueIntoRegisterParts`, `SystemZTargetLowering::joinRegisterPartsIntoValue`, `analyzeArgSplit`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerGR128ToI128`, `SystemZTargetLowering::splitValueIntoRegisterParts`, `SystemZTargetLowering::joinRegisterPartsIntoValue`, `analyzeArgSplit` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2026-2100 / 第 2026-2100 行
```cpp
2026: 
2027: SDValue SystemZTargetLowering::LowerFormalArguments(
2028:     SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
2029:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &DL,
2030:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
2031:   MachineFunction &MF = DAG.getMachineFunction();
2032:   MachineFrameInfo &MFI = MF.getFrameInfo();
2033:   MachineRegisterInfo &MRI = MF.getRegInfo();
2034:   SystemZMachineFunctionInfo *FuncInfo =
2035:       MF.getInfo<SystemZMachineFunctionInfo>();
2036:   auto *TFL = Subtarget.getFrameLowering<SystemZELFFrameLowering>();
2037:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
2038: 
2039:   // Assign locations to all of the incoming arguments.
2040:   SmallVector<CCValAssign, 16> ArgLocs;
2041:   CCState CCInfo(CallConv, IsVarArg, MF, ArgLocs, *DAG.getContext());
2042:   CCInfo.AnalyzeFormalArguments(Ins, CC_SystemZ);
2043:   FuncInfo->setSizeOfFnParams(CCInfo.getStackSize());
2044: 
2045:   unsigned NumFixedGPRs = 0;
2046:   unsigned NumFixedFPRs = 0;
2047:   for (unsigned I = 0, E = ArgLocs.size(); I != E; ++I) {
2048:     SDValue ArgValue;
2049:     CCValAssign &VA = ArgLocs[I];
2050:     EVT LocVT = VA.getLocVT();
2051:     if (VA.isRegLoc()) {
2052:       // Arguments passed in registers
2053:       const TargetRegisterClass *RC;
2054:       switch (LocVT.getSimpleVT().SimpleTy) {
2055:       default:
2056:         // Integers smaller than i64 should be promoted to i64.
2057:         llvm_unreachable("Unexpected argument type");
2058:       case MVT::i32:
2059:         NumFixedGPRs += 1;
2060:         RC = &SystemZ::GR32BitRegClass;
2061:         break;
2062:       case MVT::i64:
2063:         NumFixedGPRs += 1;
2064:         RC = &SystemZ::GR64BitRegClass;
2065:         break;
2066:       case MVT::f16:
2067:         NumFixedFPRs += 1;
2068:         RC = &SystemZ::FP16BitRegClass;
2069:         break;
2070:       case MVT::f32:
2071:         NumFixedFPRs += 1;
2072:         RC = &SystemZ::FP32BitRegClass;
2073:         break;
2074:       case MVT::f64:
2075:         NumFixedFPRs += 1;
2076:         RC = &SystemZ::FP64BitRegClass;
2077:         break;
2078:       case MVT::f128:
2079:         NumFixedFPRs += 2;
2080:         RC = &SystemZ::FP128BitRegClass;
2081:         break;
2082:       case MVT::v16i8:
2083:       case MVT::v8i16:
2084:       case MVT::v4i32:
2085:       case MVT::v2i64:
2086:       case MVT::v8f16:
2087:       case MVT::v4f32:
2088:       case MVT::v2f64:
2089:         RC = &SystemZ::VR128BitRegClass;
2090:         break;
2091:       }
2092: 
2093:       Register VReg = MRI.createVirtualRegister(RC);
2094:       MRI.addLiveIn(VA.getLocReg(), VReg);
2095:       ArgValue = DAG.getCopyFromReg(Chain, DL, VReg, LocVT);
2096:     } else {
2097:       assert(VA.isMemLoc() && "Argument not register or memory");
2098: 
2099:       // Create the frame index object for this incoming parameter.
2100:       // FIXME: Pre-include call frame size in the offset, should not
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::LowerFormalArguments`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::LowerFormalArguments` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2101-2175 / 第 2101-2175 行
```cpp
2101:       // need to manually add it here.
2102:       int64_t ArgSPOffset = VA.getLocMemOffset();
2103:       if (Subtarget.isTargetXPLINK64()) {
2104:         auto &XPRegs =
2105:             Subtarget.getSpecialRegisters<SystemZXPLINK64Registers>();
2106:         ArgSPOffset += XPRegs.getCallFrameSize();
2107:       }
2108:       int FI =
2109:           MFI.CreateFixedObject(LocVT.getSizeInBits() / 8, ArgSPOffset, true);
2110: 
2111:       // Create the SelectionDAG nodes corresponding to a load
2112:       // from this parameter.  Unpromoted ints and floats are
2113:       // passed as right-justified 8-byte values.
2114:       SDValue FIN = DAG.getFrameIndex(FI, PtrVT);
2115:       if (VA.getLocVT() == MVT::i32 || VA.getLocVT() == MVT::f32 ||
2116:           VA.getLocVT() == MVT::f16) {
2117:         unsigned SlotOffs = VA.getLocVT() == MVT::f16 ? 6 : 4;
2118:         FIN = DAG.getNode(ISD::ADD, DL, PtrVT, FIN,
2119:                           DAG.getIntPtrConstant(SlotOffs, DL));
2120:       }
2121:       ArgValue = DAG.getLoad(LocVT, DL, Chain, FIN,
2122:                              MachinePointerInfo::getFixedStack(MF, FI));
2123:     }
2124: 
2125:     // Convert the value of the argument register into the value that's
2126:     // being passed.
2127:     if (VA.getLocInfo() == CCValAssign::Indirect) {
2128:       InVals.push_back(DAG.getLoad(VA.getValVT(), DL, Chain, ArgValue,
2129:                                    MachinePointerInfo()));
2130:       // If the original argument was split (e.g. i128), we need
2131:       // to load all parts of it here (using the same address).
2132:       MVT PartVT;
2133:       unsigned NumParts;
2134:       if (analyzeArgSplit(Ins, ArgLocs, I, PartVT, NumParts)) {
2135:         for (unsigned PartIdx = 1; PartIdx < NumParts; ++PartIdx) {
2136:           ++I;
2137:           CCValAssign &PartVA = ArgLocs[I];
2138:           unsigned PartOffset = Ins[I].PartOffset;
2139:           SDValue Address = DAG.getNode(ISD::ADD, DL, PtrVT, ArgValue,
2140:                                         DAG.getIntPtrConstant(PartOffset, DL));
2141:           InVals.push_back(DAG.getLoad(PartVA.getValVT(), DL, Chain, Address,
2142:                                        MachinePointerInfo()));
2143:           assert(PartOffset && "Offset should be non-zero.");
2144:         }
2145:       }
2146:     } else
2147:       InVals.push_back(convertLocVTToValVT(DAG, DL, VA, Chain, ArgValue));
2148:   }
2149: 
2150:   if (IsVarArg && Subtarget.isTargetXPLINK64()) {
2151:     // Save the number of non-varargs registers for later use by va_start, etc.
2152:     FuncInfo->setVarArgsFirstGPR(NumFixedGPRs);
2153:     FuncInfo->setVarArgsFirstFPR(NumFixedFPRs);
2154: 
2155:     auto *Regs = static_cast<SystemZXPLINK64Registers *>(
2156:         Subtarget.getSpecialRegisters());
2157: 
2158:     // Likewise the address (in the form of a frame index) of where the
2159:     // first stack vararg would be.  The 1-byte size here is arbitrary.
2160:     // FIXME: Pre-include call frame size in the offset, should not
2161:     // need to manually add it here.
2162:     int64_t VarArgOffset = CCInfo.getStackSize() + Regs->getCallFrameSize();
2163:     int FI = MFI.CreateFixedObject(1, VarArgOffset, true);
2164:     FuncInfo->setVarArgsFrameIndex(FI);
2165:   }
2166: 
2167:   if (IsVarArg && Subtarget.isTargetELF()) {
2168:     // Save the number of non-varargs registers for later use by va_start, etc.
2169:     FuncInfo->setVarArgsFirstGPR(NumFixedGPRs);
2170:     FuncInfo->setVarArgsFirstFPR(NumFixedFPRs);
2171: 
2172:     // Likewise the address (in the form of a frame index) of where the
2173:     // first stack vararg would be.  The 1-byte size here is arbitrary.
2174:     int64_t VarArgsOffset = CCInfo.getStackSize();
2175:     FuncInfo->setVarArgsFrameIndex(
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 2176-2250 / 第 2176-2250 行
```cpp
2176:         MFI.CreateFixedObject(1, VarArgsOffset, true));
2177: 
2178:     // ...and a similar frame index for the caller-allocated save area
2179:     // that will be used to store the incoming registers.
2180:     int64_t RegSaveOffset =
2181:       -SystemZMC::ELFCallFrameSize + TFL->getRegSpillOffset(MF, SystemZ::R2D) - 16;
2182:     unsigned RegSaveIndex = MFI.CreateFixedObject(1, RegSaveOffset, true);
2183:     FuncInfo->setRegSaveFrameIndex(RegSaveIndex);
2184: 
2185:     // Store the FPR varargs in the reserved frame slots.  (We store the
2186:     // GPRs as part of the prologue.)
2187:     if (NumFixedFPRs < SystemZ::ELFNumArgFPRs && !useSoftFloat()) {
2188:       SDValue MemOps[SystemZ::ELFNumArgFPRs];
2189:       for (unsigned I = NumFixedFPRs; I < SystemZ::ELFNumArgFPRs; ++I) {
2190:         unsigned Offset = TFL->getRegSpillOffset(MF, SystemZ::ELFArgFPRs[I]);
2191:         int FI =
2192:           MFI.CreateFixedObject(8, -SystemZMC::ELFCallFrameSize + Offset, true);
2193:         SDValue FIN = DAG.getFrameIndex(FI, getPointerTy(DAG.getDataLayout()));
2194:         Register VReg = MF.addLiveIn(SystemZ::ELFArgFPRs[I],
2195:                                      &SystemZ::FP64BitRegClass);
2196:         SDValue ArgValue = DAG.getCopyFromReg(Chain, DL, VReg, MVT::f64);
2197:         MemOps[I] = DAG.getStore(ArgValue.getValue(1), DL, ArgValue, FIN,
2198:                                  MachinePointerInfo::getFixedStack(MF, FI));
2199:       }
2200:       // Join the stores, which are independent of one another.
2201:       Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other,
2202:                           ArrayRef(&MemOps[NumFixedFPRs],
2203:                                    SystemZ::ELFNumArgFPRs - NumFixedFPRs));
2204:     }
2205:   }
2206: 
2207:   if (Subtarget.isTargetXPLINK64()) {
2208:     // Create virual register  for handling incoming "ADA" special register (R5)
2209:     const TargetRegisterClass *RC = &SystemZ::ADDR64BitRegClass;
2210:     Register ADAvReg = MRI.createVirtualRegister(RC);
2211:     auto *Regs = static_cast<SystemZXPLINK64Registers *>(
2212:         Subtarget.getSpecialRegisters());
2213:     MRI.addLiveIn(Regs->getADARegister(), ADAvReg);
2214:     FuncInfo->setADAVirtualRegister(ADAvReg);
2215:   }
2216:   return Chain;
2217: }
2218: 
2219: static bool canUseSiblingCall(const CCState &ArgCCInfo,
2220:                               SmallVectorImpl<CCValAssign> &ArgLocs,
2221:                               SmallVectorImpl<ISD::OutputArg> &Outs) {
2222:   // Punt if there are any indirect or stack arguments, or if the call
2223:   // needs the callee-saved argument register R6, or if the call uses
2224:   // the callee-saved register arguments SwiftSelf and SwiftError.
2225:   for (unsigned I = 0, E = ArgLocs.size(); I != E; ++I) {
2226:     CCValAssign &VA = ArgLocs[I];
2227:     if (VA.getLocInfo() == CCValAssign::Indirect)
2228:       return false;
2229:     if (!VA.isRegLoc())
2230:       return false;
2231:     Register Reg = VA.getLocReg();
2232:     if (Reg == SystemZ::R6H || Reg == SystemZ::R6L || Reg == SystemZ::R6D)
2233:       return false;
2234:     if (Outs[I].Flags.isSwiftSelf() || Outs[I].Flags.isSwiftError())
2235:       return false;
2236:   }
2237:   return true;
2238: }
2239: 
2240: static SDValue getADAEntry(SelectionDAG &DAG, SDValue Val, SDLoc DL,
2241:                            unsigned Offset, bool LoadAdr = false) {
2242:   MachineFunction &MF = DAG.getMachineFunction();
2243:   SystemZMachineFunctionInfo *MFI = MF.getInfo<SystemZMachineFunctionInfo>();
2244:   Register ADAvReg = MFI->getADAVirtualRegister();
2245:   EVT PtrVT = DAG.getTargetLoweringInfo().getPointerTy(DAG.getDataLayout());
2246: 
2247:   SDValue Reg = DAG.getRegister(ADAvReg, PtrVT);
2248:   SDValue Ofs = DAG.getTargetConstant(Offset, DL, PtrVT);
2249: 
2250:   SDValue Result = DAG.getNode(SystemZISD::ADA_ENTRY, DL, PtrVT, Val, Reg, Ofs);
```
- **EN**: The range implements or declares functions including `canUseSiblingCall`, `getADAEntry`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `canUseSiblingCall`, `getADAEntry` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2251-2325 / 第 2251-2325 行
```cpp
2251:   if (!LoadAdr)
2252:     Result = DAG.getLoad(
2253:         PtrVT, DL, DAG.getEntryNode(), Result, MachinePointerInfo(), Align(8),
2254:         MachineMemOperand::MODereferenceable | MachineMemOperand::MOInvariant);
2255: 
2256:   return Result;
2257: }
2258: 
2259: // ADA access using Global value
2260: // Note: for functions, address of descriptor is returned
2261: static SDValue getADAEntry(SelectionDAG &DAG, const GlobalValue *GV, SDLoc DL,
2262:                            EVT PtrVT) {
2263:   unsigned ADAtype;
2264:   bool LoadAddr = false;
2265:   const GlobalAlias *GA = dyn_cast<GlobalAlias>(GV);
2266:   bool IsFunction =
2267:       (isa<Function>(GV)) || (GA && isa<Function>(GA->getAliaseeObject()));
2268:   bool IsInternal = (GV->hasInternalLinkage() || GV->hasPrivateLinkage());
2269: 
2270:   if (IsFunction) {
2271:     if (IsInternal) {
2272:       ADAtype = SystemZII::MO_ADA_DIRECT_FUNC_DESC;
2273:       LoadAddr = true;
2274:     } else
2275:       ADAtype = SystemZII::MO_ADA_INDIRECT_FUNC_DESC;
2276:   } else {
2277:     ADAtype = SystemZII::MO_ADA_DATA_SYMBOL_ADDR;
2278:   }
2279:   SDValue Val = DAG.getTargetGlobalAddress(GV, DL, PtrVT, 0, ADAtype);
2280: 
2281:   return getADAEntry(DAG, Val, DL, 0, LoadAddr);
2282: }
2283: 
2284: static bool getzOSCalleeAndADA(SelectionDAG &DAG, SDValue &Callee, SDValue &ADA,
2285:                                SDLoc &DL, SDValue &Chain) {
2286:   unsigned ADADelta = 0; // ADA offset in desc.
2287:   unsigned EPADelta = 8; // EPA offset in desc.
2288:   MachineFunction &MF = DAG.getMachineFunction();
2289:   EVT PtrVT = DAG.getTargetLoweringInfo().getPointerTy(DAG.getDataLayout());
2290: 
2291:   // XPLink calling convention.
2292:   if (auto *G = dyn_cast<GlobalAddressSDNode>(Callee)) {
2293:     bool IsInternal = (G->getGlobal()->hasInternalLinkage() ||
2294:                        G->getGlobal()->hasPrivateLinkage());
2295:     if (IsInternal) {
2296:       SystemZMachineFunctionInfo *MFI =
2297:           MF.getInfo<SystemZMachineFunctionInfo>();
2298:       Register ADAvReg = MFI->getADAVirtualRegister();
2299:       ADA = DAG.getCopyFromReg(Chain, DL, ADAvReg, PtrVT);
2300:       Callee = DAG.getTargetGlobalAddress(G->getGlobal(), DL, PtrVT);
2301:       Callee = DAG.getNode(SystemZISD::PCREL_WRAPPER, DL, PtrVT, Callee);
2302:       return true;
2303:     } else {
2304:       SDValue GA = DAG.getTargetGlobalAddress(
2305:           G->getGlobal(), DL, PtrVT, 0, SystemZII::MO_ADA_DIRECT_FUNC_DESC);
2306:       ADA = getADAEntry(DAG, GA, DL, ADADelta);
2307:       Callee = getADAEntry(DAG, GA, DL, EPADelta);
2308:     }
2309:   } else if (auto *E = dyn_cast<ExternalSymbolSDNode>(Callee)) {
2310:     SDValue ES = DAG.getTargetExternalSymbol(
2311:         E->getSymbol(), PtrVT, SystemZII::MO_ADA_DIRECT_FUNC_DESC);
2312:     ADA = getADAEntry(DAG, ES, DL, ADADelta);
2313:     Callee = getADAEntry(DAG, ES, DL, EPADelta);
2314:   } else {
2315:     // Function pointer case
2316:     ADA = DAG.getNode(ISD::ADD, DL, PtrVT, Callee,
2317:                       DAG.getConstant(ADADelta, DL, PtrVT));
2318:     ADA = DAG.getLoad(PtrVT, DL, DAG.getEntryNode(), ADA,
2319:                       MachinePointerInfo::getGOT(DAG.getMachineFunction()));
2320:     Callee = DAG.getNode(ISD::ADD, DL, PtrVT, Callee,
2321:                          DAG.getConstant(EPADelta, DL, PtrVT));
2322:     Callee = DAG.getLoad(PtrVT, DL, DAG.getEntryNode(), Callee,
2323:                          MachinePointerInfo::getGOT(DAG.getMachineFunction()));
2324:   }
2325:   return false;
```
- **EN**: The range implements or declares functions including `getADAEntry`, `getzOSCalleeAndADA`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getADAEntry`, `getzOSCalleeAndADA` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2326-2400 / 第 2326-2400 行
```cpp
2326: }
2327: 
2328: SDValue
2329: SystemZTargetLowering::LowerCall(CallLoweringInfo &CLI,
2330:                                  SmallVectorImpl<SDValue> &InVals) const {
2331:   SelectionDAG &DAG = CLI.DAG;
2332:   SDLoc &DL = CLI.DL;
2333:   SmallVectorImpl<ISD::OutputArg> &Outs = CLI.Outs;
2334:   SmallVectorImpl<SDValue> &OutVals = CLI.OutVals;
2335:   SmallVectorImpl<ISD::InputArg> &Ins = CLI.Ins;
2336:   SDValue Chain = CLI.Chain;
2337:   SDValue Callee = CLI.Callee;
2338:   bool &IsTailCall = CLI.IsTailCall;
2339:   CallingConv::ID CallConv = CLI.CallConv;
2340:   bool IsVarArg = CLI.IsVarArg;
2341:   MachineFunction &MF = DAG.getMachineFunction();
2342:   EVT PtrVT = getPointerTy(MF.getDataLayout());
2343:   LLVMContext &Ctx = *DAG.getContext();
2344:   SystemZCallingConventionRegisters *Regs = Subtarget.getSpecialRegisters();
2345: 
2346:   // FIXME: z/OS support to be added in later.
2347:   if (Subtarget.isTargetXPLINK64())
2348:     IsTailCall = false;
2349: 
2350:   // Integer args <=32 bits should have an extension attribute.
2351:   verifyNarrowIntegerArgs_Call(Outs, &MF.getFunction(), Callee);
2352: 
2353:   // Analyze the operands of the call, assigning locations to each operand.
2354:   SmallVector<CCValAssign, 16> ArgLocs;
2355:   CCState ArgCCInfo(CallConv, IsVarArg, MF, ArgLocs, Ctx);
2356:   ArgCCInfo.AnalyzeCallOperands(Outs, CC_SystemZ);
2357: 
2358:   // We don't support GuaranteedTailCallOpt, only automatically-detected
2359:   // sibling calls.
2360:   if (IsTailCall && !canUseSiblingCall(ArgCCInfo, ArgLocs, Outs))
2361:     IsTailCall = false;
2362: 
2363:   // Get a count of how many bytes are to be pushed on the stack.
2364:   unsigned NumBytes = ArgCCInfo.getStackSize();
2365: 
2366:   // Mark the start of the call.
2367:   if (!IsTailCall)
2368:     Chain = DAG.getCALLSEQ_START(Chain, NumBytes, 0, DL);
2369: 
2370:   // Copy argument values to their designated locations.
2371:   SmallVector<std::pair<unsigned, SDValue>, 9> RegsToPass;
2372:   SmallVector<SDValue, 8> MemOpChains;
2373:   SDValue StackPtr;
2374:   for (unsigned I = 0, E = ArgLocs.size(); I != E; ++I) {
2375:     CCValAssign &VA = ArgLocs[I];
2376:     SDValue ArgValue = OutVals[I];
2377: 
2378:     if (VA.getLocInfo() == CCValAssign::Indirect) {
2379:       // Store the argument in a stack slot and pass its address.
2380:       EVT SlotVT;
2381:       MVT PartVT;
2382:       unsigned NumParts = 1;
2383:       if (analyzeArgSplit(Outs, ArgLocs, I, PartVT, NumParts))
2384:         SlotVT = EVT::getIntegerVT(Ctx, PartVT.getSizeInBits() * NumParts);
2385:       else
2386:         SlotVT = Outs[I].VT;
2387:       SDValue SpillSlot = DAG.CreateStackTemporary(SlotVT);
2388:       int FI = cast<FrameIndexSDNode>(SpillSlot)->getIndex();
2389: 
2390:       MachinePointerInfo StackPtrInfo =
2391:           MachinePointerInfo::getFixedStack(MF, FI);
2392:       MemOpChains.push_back(
2393:           DAG.getStore(Chain, DL, ArgValue, SpillSlot, StackPtrInfo));
2394:       // If the original argument was split (e.g. i128), we need
2395:       // to store all parts of it here (and pass just one address).
2396:       assert(Outs[I].PartOffset == 0);
2397:       for (unsigned PartIdx = 1; PartIdx < NumParts; ++PartIdx) {
2398:         ++I;
2399:         SDValue PartValue = OutVals[I];
2400:         unsigned PartOffset = Outs[I].PartOffset;
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::LowerCall`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::LowerCall` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2401-2475 / 第 2401-2475 行
```cpp
2401:         SDValue Address = DAG.getNode(ISD::ADD, DL, PtrVT, SpillSlot,
2402:                                       DAG.getIntPtrConstant(PartOffset, DL));
2403:         MemOpChains.push_back(
2404:             DAG.getStore(Chain, DL, PartValue, Address,
2405:                          StackPtrInfo.getWithOffset(PartOffset)));
2406:         assert(PartOffset && "Offset should be non-zero.");
2407:         assert((PartOffset + PartValue.getValueType().getStoreSize() <=
2408:                 SlotVT.getStoreSize()) && "Not enough space for argument part!");
2409:       }
2410:       ArgValue = SpillSlot;
2411:     } else
2412:       ArgValue = convertValVTToLocVT(DAG, DL, VA, ArgValue);
2413: 
2414:     if (VA.isRegLoc()) {
2415:       // In XPLINK64, for the 128-bit vararg case, ArgValue is bitcasted to a
2416:       // MVT::i128 type. We decompose the 128-bit type to a pair of its high
2417:       // and low values.
2418:       if (VA.getLocVT() == MVT::i128)
2419:         ArgValue = lowerI128ToGR128(DAG, ArgValue);
2420:       // Queue up the argument copies and emit them at the end.
2421:       RegsToPass.push_back(std::make_pair(VA.getLocReg(), ArgValue));
2422:     } else {
2423:       assert(VA.isMemLoc() && "Argument not register or memory");
2424: 
2425:       // Work out the address of the stack slot.  Unpromoted ints and
2426:       // floats are passed as right-justified 8-byte values.
2427:       if (!StackPtr.getNode())
2428:         StackPtr = DAG.getCopyFromReg(Chain, DL,
2429:                                       Regs->getStackPointerRegister(), PtrVT);
2430:       unsigned Offset = Regs->getStackPointerBias() + Regs->getCallFrameSize() +
2431:                         VA.getLocMemOffset();
2432:       if (VA.getLocVT() == MVT::i32 || VA.getLocVT() == MVT::f32)
2433:         Offset += 4;
2434:       else if (VA.getLocVT() == MVT::f16)
2435:         Offset += 6;
2436:       SDValue Address = DAG.getNode(ISD::ADD, DL, PtrVT, StackPtr,
2437:                                     DAG.getIntPtrConstant(Offset, DL));
2438: 
2439:       // Emit the store.
2440:       MemOpChains.push_back(
2441:           DAG.getStore(Chain, DL, ArgValue, Address, MachinePointerInfo()));
2442: 
2443:       // Although long doubles or vectors are passed through the stack when
2444:       // they are vararg (non-fixed arguments), if a long double or vector
2445:       // occupies the third and fourth slot of the argument list GPR3 should
2446:       // still shadow the third slot of the argument list.
2447:       if (Subtarget.isTargetXPLINK64() && VA.needsCustom()) {
2448:         SDValue ShadowArgValue =
2449:             DAG.getNode(ISD::EXTRACT_ELEMENT, DL, MVT::i64, ArgValue,
2450:                         DAG.getIntPtrConstant(1, DL));
2451:         RegsToPass.push_back(std::make_pair(SystemZ::R3D, ShadowArgValue));
2452:       }
2453:     }
2454:   }
2455: 
2456:   // Join the stores, which are independent of one another.
2457:   if (!MemOpChains.empty())
2458:     Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, MemOpChains);
2459: 
2460:   // Accept direct calls by converting symbolic call addresses to the
2461:   // associated Target* opcodes.  Force %r1 to be used for indirect
2462:   // tail calls.
2463:   SDValue Glue;
2464: 
2465:   if (Subtarget.isTargetXPLINK64()) {
2466:     SDValue ADA;
2467:     bool IsBRASL = getzOSCalleeAndADA(DAG, Callee, ADA, DL, Chain);
2468:     if (!IsBRASL) {
2469:       unsigned CalleeReg = static_cast<SystemZXPLINK64Registers *>(Regs)
2470:                                ->getAddressOfCalleeRegister();
2471:       Chain = DAG.getCopyToReg(Chain, DL, CalleeReg, Callee, Glue);
2472:       Glue = Chain.getValue(1);
2473:       Callee = DAG.getRegister(CalleeReg, Callee.getValueType());
2474:     }
2475:     RegsToPass.push_back(std::make_pair(
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 2476-2550 / 第 2476-2550 行
```cpp
2476:         static_cast<SystemZXPLINK64Registers *>(Regs)->getADARegister(), ADA));
2477:   } else {
2478:     if (auto *G = dyn_cast<GlobalAddressSDNode>(Callee)) {
2479:       Callee = DAG.getTargetGlobalAddress(G->getGlobal(), DL, PtrVT);
2480:       Callee = DAG.getNode(SystemZISD::PCREL_WRAPPER, DL, PtrVT, Callee);
2481:     } else if (auto *E = dyn_cast<ExternalSymbolSDNode>(Callee)) {
2482:       Callee = DAG.getTargetExternalSymbol(E->getSymbol(), PtrVT);
2483:       Callee = DAG.getNode(SystemZISD::PCREL_WRAPPER, DL, PtrVT, Callee);
2484:     } else if (IsTailCall) {
2485:       Chain = DAG.getCopyToReg(Chain, DL, SystemZ::R1D, Callee, Glue);
2486:       Glue = Chain.getValue(1);
2487:       Callee = DAG.getRegister(SystemZ::R1D, Callee.getValueType());
2488:     }
2489:   }
2490: 
2491:   // Build a sequence of copy-to-reg nodes, chained and glued together.
2492:   for (const auto &[Reg, N] : RegsToPass) {
2493:     Chain = DAG.getCopyToReg(Chain, DL, Reg, N, Glue);
2494:     Glue = Chain.getValue(1);
2495:   }
2496: 
2497:   // The first call operand is the chain and the second is the target address.
2498:   SmallVector<SDValue, 8> Ops;
2499:   Ops.push_back(Chain);
2500:   Ops.push_back(Callee);
2501: 
2502:   // Add argument registers to the end of the list so that they are
2503:   // known live into the call.
2504:   for (const auto &[Reg, N] : RegsToPass)
2505:     Ops.push_back(DAG.getRegister(Reg, N.getValueType()));
2506: 
2507:   // Add a register mask operand representing the call-preserved registers.
2508:   const TargetRegisterInfo *TRI = Subtarget.getRegisterInfo();
2509:   const uint32_t *Mask = TRI->getCallPreservedMask(MF, CallConv);
2510:   assert(Mask && "Missing call preserved mask for calling convention");
2511:   Ops.push_back(DAG.getRegisterMask(Mask));
2512: 
2513:   // Glue the call to the argument copies, if any.
2514:   if (Glue.getNode())
2515:     Ops.push_back(Glue);
2516: 
2517:   // Emit the call.
2518:   SDVTList NodeTys = DAG.getVTList(MVT::Other, MVT::Glue);
2519:   if (IsTailCall) {
2520:     SDValue Ret = DAG.getNode(SystemZISD::SIBCALL, DL, NodeTys, Ops);
2521:     DAG.addNoMergeSiteInfo(Ret.getNode(), CLI.NoMerge);
2522:     return Ret;
2523:   }
2524:   Chain = DAG.getNode(SystemZISD::CALL, DL, NodeTys, Ops);
2525:   DAG.addNoMergeSiteInfo(Chain.getNode(), CLI.NoMerge);
2526:   Glue = Chain.getValue(1);
2527: 
2528:   // Mark the end of the call, which is glued to the call itself.
2529:   Chain = DAG.getCALLSEQ_END(Chain, NumBytes, 0, Glue, DL);
2530:   Glue = Chain.getValue(1);
2531: 
2532:   // Assign locations to each value returned by this call.
2533:   SmallVector<CCValAssign, 16> RetLocs;
2534:   CCState RetCCInfo(CallConv, IsVarArg, MF, RetLocs, Ctx);
2535:   RetCCInfo.AnalyzeCallResult(Ins, RetCC_SystemZ);
2536: 
2537:   // Copy all of the result registers out of their specified physreg.
2538:   for (CCValAssign &VA : RetLocs) {
2539:     // Copy the value out, gluing the copy to the end of the call sequence.
2540:     SDValue RetValue = DAG.getCopyFromReg(Chain, DL, VA.getLocReg(),
2541:                                           VA.getLocVT(), Glue);
2542:     Chain = RetValue.getValue(1);
2543:     Glue = RetValue.getValue(2);
2544: 
2545:     // Convert the value of the return register into the value that's
2546:     // being returned.
2547:     InVals.push_back(convertLocVTToValVT(DAG, DL, VA, Chain, RetValue));
2548:   }
2549: 
2550:   return Chain;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 2551-2625 / 第 2551-2625 行
```cpp
2551: }
2552: 
2553: // Generate a call taking the given operands as arguments and returning a
2554: // result of type RetVT.
2555: std::pair<SDValue, SDValue> SystemZTargetLowering::makeExternalCall(
2556:     SDValue Chain, SelectionDAG &DAG, const char *CalleeName, EVT RetVT,
2557:     ArrayRef<SDValue> Ops, CallingConv::ID CallConv, bool IsSigned, SDLoc DL,
2558:     bool DoesNotReturn, bool IsReturnValueUsed) const {
2559:   TargetLowering::ArgListTy Args;
2560:   Args.reserve(Ops.size());
2561: 
2562:   for (SDValue Op : Ops) {
2563:     TargetLowering::ArgListEntry Entry(
2564:         Op, Op.getValueType().getTypeForEVT(*DAG.getContext()));
2565:     Entry.IsSExt = shouldSignExtendTypeInLibCall(Entry.Ty, IsSigned);
2566:     Entry.IsZExt = !Entry.IsSExt;
2567:     Args.push_back(Entry);
2568:   }
2569: 
2570:   SDValue Callee =
2571:       DAG.getExternalSymbol(CalleeName, getPointerTy(DAG.getDataLayout()));
2572: 
2573:   Type *RetTy = RetVT.getTypeForEVT(*DAG.getContext());
2574:   TargetLowering::CallLoweringInfo CLI(DAG);
2575:   bool SignExtend = shouldSignExtendTypeInLibCall(RetTy, IsSigned);
2576:   CLI.setDebugLoc(DL)
2577:       .setChain(Chain)
2578:       .setCallee(CallConv, RetTy, Callee, std::move(Args))
2579:       .setNoReturn(DoesNotReturn)
2580:       .setDiscardResult(!IsReturnValueUsed)
2581:       .setSExtResult(SignExtend)
2582:       .setZExtResult(!SignExtend);
2583:   return LowerCallTo(CLI);
2584: }
2585: 
2586: bool SystemZTargetLowering::CanLowerReturn(
2587:     CallingConv::ID CallConv, MachineFunction &MF, bool IsVarArg,
2588:     const SmallVectorImpl<ISD::OutputArg> &Outs, LLVMContext &Context,
2589:     const Type *RetTy) const {
2590:   // Special case that we cannot easily detect in RetCC_SystemZ since
2591:   // i128 may not be a legal type.
2592:   for (auto &Out : Outs)
2593:     if (Out.ArgVT.isScalarInteger() && Out.ArgVT.getSizeInBits() > 64)
2594:       return false;
2595: 
2596:   SmallVector<CCValAssign, 16> RetLocs;
2597:   CCState RetCCInfo(CallConv, IsVarArg, MF, RetLocs, Context);
2598:   return RetCCInfo.CheckReturn(Outs, RetCC_SystemZ);
2599: }
2600: 
2601: SDValue
2602: SystemZTargetLowering::LowerReturn(SDValue Chain, CallingConv::ID CallConv,
2603:                                    bool IsVarArg,
2604:                                    const SmallVectorImpl<ISD::OutputArg> &Outs,
2605:                                    const SmallVectorImpl<SDValue> &OutVals,
2606:                                    const SDLoc &DL, SelectionDAG &DAG) const {
2607:   MachineFunction &MF = DAG.getMachineFunction();
2608: 
2609:   // Integer args <=32 bits should have an extension attribute.
2610:   verifyNarrowIntegerArgs_Ret(Outs, &MF.getFunction());
2611: 
2612:   // Assign locations to each returned value.
2613:   SmallVector<CCValAssign, 16> RetLocs;
2614:   CCState RetCCInfo(CallConv, IsVarArg, MF, RetLocs, *DAG.getContext());
2615:   RetCCInfo.AnalyzeReturn(Outs, RetCC_SystemZ);
2616: 
2617:   // Quick exit for void returns
2618:   if (RetLocs.empty())
2619:     return DAG.getNode(SystemZISD::RET_GLUE, DL, MVT::Other, Chain);
2620: 
2621:   if (CallConv == CallingConv::GHC)
2622:     report_fatal_error("GHC functions return void only");
2623: 
2624:   // Copy the result values into the output registers.
2625:   SDValue Glue;
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::makeExternalCall`, `SystemZTargetLowering::CanLowerReturn`, `SystemZTargetLowering::LowerReturn`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::makeExternalCall`, `SystemZTargetLowering::CanLowerReturn`, `SystemZTargetLowering::LowerReturn` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2626-2700 / 第 2626-2700 行
```cpp
2626:   SmallVector<SDValue, 4> RetOps;
2627:   RetOps.push_back(Chain);
2628:   for (unsigned I = 0, E = RetLocs.size(); I != E; ++I) {
2629:     CCValAssign &VA = RetLocs[I];
2630:     SDValue RetValue = OutVals[I];
2631: 
2632:     // Make the return register live on exit.
2633:     assert(VA.isRegLoc() && "Can only return in registers!");
2634: 
2635:     // Promote the value as required.
2636:     RetValue = convertValVTToLocVT(DAG, DL, VA, RetValue);
2637: 
2638:     // Chain and glue the copies together.
2639:     Register Reg = VA.getLocReg();
2640:     Chain = DAG.getCopyToReg(Chain, DL, Reg, RetValue, Glue);
2641:     Glue = Chain.getValue(1);
2642:     RetOps.push_back(DAG.getRegister(Reg, VA.getLocVT()));
2643:   }
2644: 
2645:   // Update chain and glue.
2646:   RetOps[0] = Chain;
2647:   if (Glue.getNode())
2648:     RetOps.push_back(Glue);
2649: 
2650:   return DAG.getNode(SystemZISD::RET_GLUE, DL, MVT::Other, RetOps);
2651: }
2652: 
2653: // Return true if Op is an intrinsic node with chain that returns the CC value
2654: // as its only (other) argument.  Provide the associated SystemZISD opcode and
2655: // the mask of valid CC values if so.
2656: static bool isIntrinsicWithCCAndChain(SDValue Op, unsigned &Opcode,
2657:                                       unsigned &CCValid) {
2658:   unsigned Id = Op.getConstantOperandVal(1);
2659:   switch (Id) {
2660:   case Intrinsic::s390_tbegin:
2661:     Opcode = SystemZISD::TBEGIN;
2662:     CCValid = SystemZ::CCMASK_TBEGIN;
2663:     return true;
2664: 
2665:   case Intrinsic::s390_tbegin_nofloat:
2666:     Opcode = SystemZISD::TBEGIN_NOFLOAT;
2667:     CCValid = SystemZ::CCMASK_TBEGIN;
2668:     return true;
2669: 
2670:   case Intrinsic::s390_tend:
2671:     Opcode = SystemZISD::TEND;
2672:     CCValid = SystemZ::CCMASK_TEND;
2673:     return true;
2674: 
2675:   default:
2676:     return false;
2677:   }
2678: }
2679: 
2680: // Return true if Op is an intrinsic node without chain that returns the
2681: // CC value as its final argument.  Provide the associated SystemZISD
2682: // opcode and the mask of valid CC values if so.
2683: static bool isIntrinsicWithCC(SDValue Op, unsigned &Opcode, unsigned &CCValid) {
2684:   unsigned Id = Op.getConstantOperandVal(0);
2685:   switch (Id) {
2686:   case Intrinsic::s390_vpkshs:
2687:   case Intrinsic::s390_vpksfs:
2688:   case Intrinsic::s390_vpksgs:
2689:     Opcode = SystemZISD::PACKS_CC;
2690:     CCValid = SystemZ::CCMASK_VCMP;
2691:     return true;
2692: 
2693:   case Intrinsic::s390_vpklshs:
2694:   case Intrinsic::s390_vpklsfs:
2695:   case Intrinsic::s390_vpklsgs:
2696:     Opcode = SystemZISD::PACKLS_CC;
2697:     CCValid = SystemZ::CCMASK_VCMP;
2698:     return true;
2699: 
2700:   case Intrinsic::s390_vceqbs:
```
- **EN**: The range implements or declares functions including `isIntrinsicWithCCAndChain`, `isIntrinsicWithCC`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isIntrinsicWithCCAndChain`, `isIntrinsicWithCC` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2701-2775 / 第 2701-2775 行
```cpp
2701:   case Intrinsic::s390_vceqhs:
2702:   case Intrinsic::s390_vceqfs:
2703:   case Intrinsic::s390_vceqgs:
2704:   case Intrinsic::s390_vceqqs:
2705:     Opcode = SystemZISD::VICMPES;
2706:     CCValid = SystemZ::CCMASK_VCMP;
2707:     return true;
2708: 
2709:   case Intrinsic::s390_vchbs:
2710:   case Intrinsic::s390_vchhs:
2711:   case Intrinsic::s390_vchfs:
2712:   case Intrinsic::s390_vchgs:
2713:   case Intrinsic::s390_vchqs:
2714:     Opcode = SystemZISD::VICMPHS;
2715:     CCValid = SystemZ::CCMASK_VCMP;
2716:     return true;
2717: 
2718:   case Intrinsic::s390_vchlbs:
2719:   case Intrinsic::s390_vchlhs:
2720:   case Intrinsic::s390_vchlfs:
2721:   case Intrinsic::s390_vchlgs:
2722:   case Intrinsic::s390_vchlqs:
2723:     Opcode = SystemZISD::VICMPHLS;
2724:     CCValid = SystemZ::CCMASK_VCMP;
2725:     return true;
2726: 
2727:   case Intrinsic::s390_vtm:
2728:     Opcode = SystemZISD::VTM;
2729:     CCValid = SystemZ::CCMASK_VCMP;
2730:     return true;
2731: 
2732:   case Intrinsic::s390_vfaebs:
2733:   case Intrinsic::s390_vfaehs:
2734:   case Intrinsic::s390_vfaefs:
2735:     Opcode = SystemZISD::VFAE_CC;
2736:     CCValid = SystemZ::CCMASK_ANY;
2737:     return true;
2738: 
2739:   case Intrinsic::s390_vfaezbs:
2740:   case Intrinsic::s390_vfaezhs:
2741:   case Intrinsic::s390_vfaezfs:
2742:     Opcode = SystemZISD::VFAEZ_CC;
2743:     CCValid = SystemZ::CCMASK_ANY;
2744:     return true;
2745: 
2746:   case Intrinsic::s390_vfeebs:
2747:   case Intrinsic::s390_vfeehs:
2748:   case Intrinsic::s390_vfeefs:
2749:     Opcode = SystemZISD::VFEE_CC;
2750:     CCValid = SystemZ::CCMASK_ANY;
2751:     return true;
2752: 
2753:   case Intrinsic::s390_vfeezbs:
2754:   case Intrinsic::s390_vfeezhs:
2755:   case Intrinsic::s390_vfeezfs:
2756:     Opcode = SystemZISD::VFEEZ_CC;
2757:     CCValid = SystemZ::CCMASK_ANY;
2758:     return true;
2759: 
2760:   case Intrinsic::s390_vfenebs:
2761:   case Intrinsic::s390_vfenehs:
2762:   case Intrinsic::s390_vfenefs:
2763:     Opcode = SystemZISD::VFENE_CC;
2764:     CCValid = SystemZ::CCMASK_ANY;
2765:     return true;
2766: 
2767:   case Intrinsic::s390_vfenezbs:
2768:   case Intrinsic::s390_vfenezhs:
2769:   case Intrinsic::s390_vfenezfs:
2770:     Opcode = SystemZISD::VFENEZ_CC;
2771:     CCValid = SystemZ::CCMASK_ANY;
2772:     return true;
2773: 
2774:   case Intrinsic::s390_vistrbs:
2775:   case Intrinsic::s390_vistrhs:
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2776-2850 / 第 2776-2850 行
```cpp
2776:   case Intrinsic::s390_vistrfs:
2777:     Opcode = SystemZISD::VISTR_CC;
2778:     CCValid = SystemZ::CCMASK_0 | SystemZ::CCMASK_3;
2779:     return true;
2780: 
2781:   case Intrinsic::s390_vstrcbs:
2782:   case Intrinsic::s390_vstrchs:
2783:   case Intrinsic::s390_vstrcfs:
2784:     Opcode = SystemZISD::VSTRC_CC;
2785:     CCValid = SystemZ::CCMASK_ANY;
2786:     return true;
2787: 
2788:   case Intrinsic::s390_vstrczbs:
2789:   case Intrinsic::s390_vstrczhs:
2790:   case Intrinsic::s390_vstrczfs:
2791:     Opcode = SystemZISD::VSTRCZ_CC;
2792:     CCValid = SystemZ::CCMASK_ANY;
2793:     return true;
2794: 
2795:   case Intrinsic::s390_vstrsb:
2796:   case Intrinsic::s390_vstrsh:
2797:   case Intrinsic::s390_vstrsf:
2798:     Opcode = SystemZISD::VSTRS_CC;
2799:     CCValid = SystemZ::CCMASK_ANY;
2800:     return true;
2801: 
2802:   case Intrinsic::s390_vstrszb:
2803:   case Intrinsic::s390_vstrszh:
2804:   case Intrinsic::s390_vstrszf:
2805:     Opcode = SystemZISD::VSTRSZ_CC;
2806:     CCValid = SystemZ::CCMASK_ANY;
2807:     return true;
2808: 
2809:   case Intrinsic::s390_vfcedbs:
2810:   case Intrinsic::s390_vfcesbs:
2811:     Opcode = SystemZISD::VFCMPES;
2812:     CCValid = SystemZ::CCMASK_VCMP;
2813:     return true;
2814: 
2815:   case Intrinsic::s390_vfchdbs:
2816:   case Intrinsic::s390_vfchsbs:
2817:     Opcode = SystemZISD::VFCMPHS;
2818:     CCValid = SystemZ::CCMASK_VCMP;
2819:     return true;
2820: 
2821:   case Intrinsic::s390_vfchedbs:
2822:   case Intrinsic::s390_vfchesbs:
2823:     Opcode = SystemZISD::VFCMPHES;
2824:     CCValid = SystemZ::CCMASK_VCMP;
2825:     return true;
2826: 
2827:   case Intrinsic::s390_vftcidb:
2828:   case Intrinsic::s390_vftcisb:
2829:     Opcode = SystemZISD::VFTCI;
2830:     CCValid = SystemZ::CCMASK_VCMP;
2831:     return true;
2832: 
2833:   case Intrinsic::s390_tdc:
2834:     Opcode = SystemZISD::TDC;
2835:     CCValid = SystemZ::CCMASK_TDC;
2836:     return true;
2837: 
2838:   default:
2839:     return false;
2840:   }
2841: }
2842: 
2843: // Emit an intrinsic with chain and an explicit CC register result.
2844: static SDNode *emitIntrinsicWithCCAndChain(SelectionDAG &DAG, SDValue Op,
2845:                                            unsigned Opcode) {
2846:   // Copy all operands except the intrinsic ID.
2847:   unsigned NumOps = Op.getNumOperands();
2848:   SmallVector<SDValue, 6> Ops;
2849:   Ops.reserve(NumOps - 1);
2850:   Ops.push_back(Op.getOperand(0));
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 2851-2925 / 第 2851-2925 行
```cpp
2851:   for (unsigned I = 2; I < NumOps; ++I)
2852:     Ops.push_back(Op.getOperand(I));
2853: 
2854:   assert(Op->getNumValues() == 2 && "Expected only CC result and chain");
2855:   SDVTList RawVTs = DAG.getVTList(MVT::i32, MVT::Other);
2856:   SDValue Intr = DAG.getNode(Opcode, SDLoc(Op), RawVTs, Ops);
2857:   SDValue OldChain = SDValue(Op.getNode(), 1);
2858:   SDValue NewChain = SDValue(Intr.getNode(), 1);
2859:   DAG.ReplaceAllUsesOfValueWith(OldChain, NewChain);
2860:   return Intr.getNode();
2861: }
2862: 
2863: // Emit an intrinsic with an explicit CC register result.
2864: static SDNode *emitIntrinsicWithCC(SelectionDAG &DAG, SDValue Op,
2865:                                    unsigned Opcode) {
2866:   // Copy all operands except the intrinsic ID.
2867:   SDLoc DL(Op);
2868:   unsigned NumOps = Op.getNumOperands();
2869:   SmallVector<SDValue, 6> Ops;
2870:   Ops.reserve(NumOps - 1);
2871:   for (unsigned I = 1; I < NumOps; ++I) {
2872:     SDValue CurrOper = Op.getOperand(I);
2873:     if (CurrOper.getValueType() == MVT::f16) {
2874:       assert((Op.getConstantOperandVal(0) == Intrinsic::s390_tdc && I == 1) &&
2875:              "Unhandled intrinsic with f16 operand.");
2876:       CurrOper = DAG.getFPExtendOrRound(CurrOper, DL, MVT::f32);
2877:     }
2878:     Ops.push_back(CurrOper);
2879:   }
2880: 
2881:   SDValue Intr = DAG.getNode(Opcode, DL, Op->getVTList(), Ops);
2882:   return Intr.getNode();
2883: }
2884: 
2885: // CC is a comparison that will be implemented using an integer or
2886: // floating-point comparison.  Return the condition code mask for
2887: // a branch on true.  In the integer case, CCMASK_CMP_UO is set for
2888: // unsigned comparisons and clear for signed ones.  In the floating-point
2889: // case, CCMASK_CMP_UO has its normal mask meaning (unordered).
2890: static unsigned CCMaskForCondCode(ISD::CondCode CC) {
2891: #define CONV(X) \
2892:   case ISD::SET##X: return SystemZ::CCMASK_CMP_##X; \
2893:   case ISD::SETO##X: return SystemZ::CCMASK_CMP_##X; \
2894:   case ISD::SETU##X: return SystemZ::CCMASK_CMP_UO | SystemZ::CCMASK_CMP_##X
2895: 
2896:   switch (CC) {
2897:   default:
2898:     llvm_unreachable("Invalid integer condition!");
2899: 
2900:   CONV(EQ);
2901:   CONV(NE);
2902:   CONV(GT);
2903:   CONV(GE);
2904:   CONV(LT);
2905:   CONV(LE);
2906: 
2907:   case ISD::SETO:  return SystemZ::CCMASK_CMP_O;
2908:   case ISD::SETUO: return SystemZ::CCMASK_CMP_UO;
2909:   }
2910: #undef CONV
2911: }
2912: 
2913: // If C can be converted to a comparison against zero, adjust the operands
2914: // as necessary.
2915: static void adjustZeroCmp(SelectionDAG &DAG, const SDLoc &DL, Comparison &C) {
2916:   if (C.ICmpType == SystemZICMP::UnsignedOnly)
2917:     return;
2918: 
2919:   auto *ConstOp1 = dyn_cast<ConstantSDNode>(C.Op1.getNode());
2920:   if (!ConstOp1 || ConstOp1->getValueSizeInBits(0) > 64)
2921:     return;
2922: 
2923:   int64_t Value = ConstOp1->getSExtValue();
2924:   if ((Value == -1 && C.CCMask == SystemZ::CCMASK_CMP_GT) ||
2925:       (Value == -1 && C.CCMask == SystemZ::CCMASK_CMP_LE) ||
```
- **EN**: The range implements or declares functions including `CCMaskForCondCode`, `adjustZeroCmp`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `CCMaskForCondCode`, `adjustZeroCmp` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2926-3000 / 第 2926-3000 行
```cpp
2926:       (Value == 1 && C.CCMask == SystemZ::CCMASK_CMP_LT) ||
2927:       (Value == 1 && C.CCMask == SystemZ::CCMASK_CMP_GE)) {
2928:     C.CCMask ^= SystemZ::CCMASK_CMP_EQ;
2929:     C.Op1 = DAG.getConstant(0, DL, C.Op1.getValueType());
2930:   }
2931: }
2932: 
2933: // If a comparison described by C is suitable for CLI(Y), CHHSI or CLHHSI,
2934: // adjust the operands as necessary.
2935: static void adjustSubwordCmp(SelectionDAG &DAG, const SDLoc &DL,
2936:                              Comparison &C) {
2937:   // For us to make any changes, it must a comparison between a single-use
2938:   // load and a constant.
2939:   if (!C.Op0.hasOneUse() ||
2940:       C.Op0.getOpcode() != ISD::LOAD ||
2941:       C.Op1.getOpcode() != ISD::Constant)
2942:     return;
2943: 
2944:   // We must have an 8- or 16-bit load.
2945:   auto *Load = cast<LoadSDNode>(C.Op0);
2946:   unsigned NumBits = Load->getMemoryVT().getSizeInBits();
2947:   if ((NumBits != 8 && NumBits != 16) ||
2948:       NumBits != Load->getMemoryVT().getStoreSizeInBits())
2949:     return;
2950: 
2951:   // The load must be an extending one and the constant must be within the
2952:   // range of the unextended value.
2953:   auto *ConstOp1 = cast<ConstantSDNode>(C.Op1);
2954:   if (!ConstOp1 || ConstOp1->getValueSizeInBits(0) > 64)
2955:     return;
2956:   uint64_t Value = ConstOp1->getZExtValue();
2957:   uint64_t Mask = (1 << NumBits) - 1;
2958:   if (Load->getExtensionType() == ISD::SEXTLOAD) {
2959:     // Make sure that ConstOp1 is in range of C.Op0.
2960:     int64_t SignedValue = ConstOp1->getSExtValue();
2961:     if (uint64_t(SignedValue) + (uint64_t(1) << (NumBits - 1)) > Mask)
2962:       return;
2963:     if (C.ICmpType != SystemZICMP::SignedOnly) {
2964:       // Unsigned comparison between two sign-extended values is equivalent
2965:       // to unsigned comparison between two zero-extended values.
2966:       Value &= Mask;
2967:     } else if (NumBits == 8) {
2968:       // Try to treat the comparison as unsigned, so that we can use CLI.
2969:       // Adjust CCMask and Value as necessary.
2970:       if (Value == 0 && C.CCMask == SystemZ::CCMASK_CMP_LT)
2971:         // Test whether the high bit of the byte is set.
2972:         Value = 127, C.CCMask = SystemZ::CCMASK_CMP_GT;
2973:       else if (Value == 0 && C.CCMask == SystemZ::CCMASK_CMP_GE)
2974:         // Test whether the high bit of the byte is clear.
2975:         Value = 128, C.CCMask = SystemZ::CCMASK_CMP_LT;
2976:       else
2977:         // No instruction exists for this combination.
2978:         return;
2979:       C.ICmpType = SystemZICMP::UnsignedOnly;
2980:     }
2981:   } else if (Load->getExtensionType() == ISD::ZEXTLOAD) {
2982:     if (Value > Mask)
2983:       return;
2984:     // If the constant is in range, we can use any comparison.
2985:     C.ICmpType = SystemZICMP::Any;
2986:   } else
2987:     return;
2988: 
2989:   // Make sure that the first operand is an i32 of the right extension type.
2990:   ISD::LoadExtType ExtType = (C.ICmpType == SystemZICMP::SignedOnly ?
2991:                               ISD::SEXTLOAD :
2992:                               ISD::ZEXTLOAD);
2993:   if (C.Op0.getValueType() != MVT::i32 ||
2994:       Load->getExtensionType() != ExtType) {
2995:     C.Op0 = DAG.getExtLoad(ExtType, SDLoc(Load), MVT::i32, Load->getChain(),
2996:                            Load->getBasePtr(), Load->getPointerInfo(),
2997:                            Load->getMemoryVT(), Load->getAlign(),
2998:                            Load->getMemOperand()->getFlags());
2999:     // Update the chain uses.
3000:     DAG.ReplaceAllUsesOfValueWith(SDValue(Load, 1), C.Op0.getValue(1));
```
- **EN**: The range implements or declares functions including `adjustSubwordCmp`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `adjustSubwordCmp` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3001-3075 / 第 3001-3075 行
```cpp
3001:   }
3002: 
3003:   // Make sure that the second operand is an i32 with the right value.
3004:   if (C.Op1.getValueType() != MVT::i32 ||
3005:       Value != ConstOp1->getZExtValue())
3006:     C.Op1 = DAG.getConstant((uint32_t)Value, DL, MVT::i32);
3007: }
3008: 
3009: // Return true if Op is either an unextended load, or a load suitable
3010: // for integer register-memory comparisons of type ICmpType.
3011: static bool isNaturalMemoryOperand(SDValue Op, unsigned ICmpType) {
3012:   auto *Load = dyn_cast<LoadSDNode>(Op.getNode());
3013:   if (Load) {
3014:     // There are no instructions to compare a register with a memory byte.
3015:     if (Load->getMemoryVT() == MVT::i8)
3016:       return false;
3017:     // Otherwise decide on extension type.
3018:     switch (Load->getExtensionType()) {
3019:     case ISD::NON_EXTLOAD:
3020:       return true;
3021:     case ISD::SEXTLOAD:
3022:       return ICmpType != SystemZICMP::UnsignedOnly;
3023:     case ISD::ZEXTLOAD:
3024:       return ICmpType != SystemZICMP::SignedOnly;
3025:     default:
3026:       break;
3027:     }
3028:   }
3029:   return false;
3030: }
3031: 
3032: // Return true if it is better to swap the operands of C.
3033: static bool shouldSwapCmpOperands(const Comparison &C) {
3034:   // Leave i128 and f128 comparisons alone, since they have no memory forms.
3035:   if (C.Op0.getValueType() == MVT::i128)
3036:     return false;
3037:   if (C.Op0.getValueType() == MVT::f128)
3038:     return false;
3039: 
3040:   // Always keep a floating-point constant second, since comparisons with
3041:   // zero can use LOAD TEST and comparisons with other constants make a
3042:   // natural memory operand.
3043:   if (isa<ConstantFPSDNode>(C.Op1))
3044:     return false;
3045: 
3046:   // Never swap comparisons with zero since there are many ways to optimize
3047:   // those later.
3048:   auto *ConstOp1 = dyn_cast<ConstantSDNode>(C.Op1);
3049:   if (ConstOp1 && ConstOp1->getZExtValue() == 0)
3050:     return false;
3051: 
3052:   // Also keep natural memory operands second if the loaded value is
3053:   // only used here.  Several comparisons have memory forms.
3054:   if (isNaturalMemoryOperand(C.Op1, C.ICmpType) && C.Op1.hasOneUse())
3055:     return false;
3056: 
3057:   // Look for cases where Cmp0 is a single-use load and Cmp1 isn't.
3058:   // In that case we generally prefer the memory to be second.
3059:   if (isNaturalMemoryOperand(C.Op0, C.ICmpType) && C.Op0.hasOneUse()) {
3060:     // The only exceptions are when the second operand is a constant and
3061:     // we can use things like CHHSI.
3062:     if (!ConstOp1)
3063:       return true;
3064:     // The unsigned memory-immediate instructions can handle 16-bit
3065:     // unsigned integers.
3066:     if (C.ICmpType != SystemZICMP::SignedOnly &&
3067:         isUInt<16>(ConstOp1->getZExtValue()))
3068:       return false;
3069:     // The signed memory-immediate instructions can handle 16-bit
3070:     // signed integers.
3071:     if (C.ICmpType != SystemZICMP::UnsignedOnly &&
3072:         isInt<16>(ConstOp1->getSExtValue()))
3073:       return false;
3074:     return true;
3075:   }
```
- **EN**: The range implements or declares functions including `isNaturalMemoryOperand`, `shouldSwapCmpOperands`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isNaturalMemoryOperand`, `shouldSwapCmpOperands` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3076-3150 / 第 3076-3150 行
```cpp
3076: 
3077:   // Try to promote the use of CGFR and CLGFR.
3078:   unsigned Opcode0 = C.Op0.getOpcode();
3079:   if (C.ICmpType != SystemZICMP::UnsignedOnly && Opcode0 == ISD::SIGN_EXTEND)
3080:     return true;
3081:   if (C.ICmpType != SystemZICMP::SignedOnly && Opcode0 == ISD::ZERO_EXTEND)
3082:     return true;
3083:   if (C.ICmpType != SystemZICMP::SignedOnly && Opcode0 == ISD::AND &&
3084:       C.Op0.getOperand(1).getOpcode() == ISD::Constant &&
3085:       C.Op0.getConstantOperandVal(1) == 0xffffffff)
3086:     return true;
3087: 
3088:   return false;
3089: }
3090: 
3091: // Check whether C tests for equality between X and Y and whether X - Y
3092: // or Y - X is also computed.  In that case it's better to compare the
3093: // result of the subtraction against zero.
3094: static void adjustForSubtraction(SelectionDAG &DAG, const SDLoc &DL,
3095:                                  Comparison &C) {
3096:   if (C.CCMask == SystemZ::CCMASK_CMP_EQ ||
3097:       C.CCMask == SystemZ::CCMASK_CMP_NE) {
3098:     for (SDNode *N : C.Op0->users()) {
3099:       if (N->getOpcode() == ISD::SUB &&
3100:           ((N->getOperand(0) == C.Op0 && N->getOperand(1) == C.Op1) ||
3101:            (N->getOperand(0) == C.Op1 && N->getOperand(1) == C.Op0))) {
3102:         // Disable the nsw and nuw flags: the backend needs to handle
3103:         // overflow as well during comparison elimination.
3104:         N->dropFlags(SDNodeFlags::NoWrap);
3105:         C.Op0 = SDValue(N, 0);
3106:         C.Op1 = DAG.getConstant(0, DL, N->getValueType(0));
3107:         return;
3108:       }
3109:     }
3110:   }
3111: }
3112: 
3113: // Check whether C compares a floating-point value with zero and if that
3114: // floating-point value is also negated.  In this case we can use the
3115: // negation to set CC, so avoiding separate LOAD AND TEST and
3116: // LOAD (NEGATIVE/COMPLEMENT) instructions.
3117: static void adjustForFNeg(Comparison &C) {
3118:   // This optimization is invalid for strict comparisons, since FNEG
3119:   // does not raise any exceptions.
3120:   if (C.Chain)
3121:     return;
3122:   auto *C1 = dyn_cast<ConstantFPSDNode>(C.Op1);
3123:   if (C1 && C1->isZero()) {
3124:     for (SDNode *N : C.Op0->users()) {
3125:       if (N->getOpcode() == ISD::FNEG) {
3126:         C.Op0 = SDValue(N, 0);
3127:         C.CCMask = SystemZ::reverseCCMask(C.CCMask);
3128:         return;
3129:       }
3130:     }
3131:   }
3132: }
3133: 
3134: // Check whether C compares (shl X, 32) with 0 and whether X is
3135: // also sign-extended.  In that case it is better to test the result
3136: // of the sign extension using LTGFR.
3137: //
3138: // This case is important because InstCombine transforms a comparison
3139: // with (sext (trunc X)) into a comparison with (shl X, 32).
3140: static void adjustForLTGFR(Comparison &C) {
3141:   // Check for a comparison between (shl X, 32) and 0.
3142:   if (C.Op0.getOpcode() == ISD::SHL && C.Op0.getValueType() == MVT::i64 &&
3143:       C.Op1.getOpcode() == ISD::Constant && C.Op1->getAsZExtVal() == 0) {
3144:     auto *C1 = dyn_cast<ConstantSDNode>(C.Op0.getOperand(1));
3145:     if (C1 && C1->getZExtValue() == 32) {
3146:       SDValue ShlOp0 = C.Op0.getOperand(0);
3147:       // See whether X has any SIGN_EXTEND_INREG uses.
3148:       for (SDNode *N : ShlOp0->users()) {
3149:         if (N->getOpcode() == ISD::SIGN_EXTEND_INREG &&
3150:             cast<VTSDNode>(N->getOperand(1))->getVT() == MVT::i32) {
```
- **EN**: The range implements or declares functions including `adjustForSubtraction`, `adjustForFNeg`, `adjustForLTGFR`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `adjustForSubtraction`, `adjustForFNeg`, `adjustForLTGFR` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 3151-3225 / 第 3151-3225 行
```cpp
3151:           C.Op0 = SDValue(N, 0);
3152:           return;
3153:         }
3154:       }
3155:     }
3156:   }
3157: }
3158: 
3159: // If C compares the truncation of an extending load, try to compare
3160: // the untruncated value instead.  This exposes more opportunities to
3161: // reuse CC.
3162: static void adjustICmpTruncate(SelectionDAG &DAG, const SDLoc &DL,
3163:                                Comparison &C) {
3164:   if (C.Op0.getOpcode() == ISD::TRUNCATE &&
3165:       C.Op0.getOperand(0).getOpcode() == ISD::LOAD &&
3166:       C.Op1.getOpcode() == ISD::Constant &&
3167:       cast<ConstantSDNode>(C.Op1)->getValueSizeInBits(0) <= 64 &&
3168:       C.Op1->getAsZExtVal() == 0) {
3169:     auto *L = cast<LoadSDNode>(C.Op0.getOperand(0));
3170:     if (L->getMemoryVT().getStoreSizeInBits().getFixedValue() <=
3171:         C.Op0.getValueSizeInBits().getFixedValue()) {
3172:       unsigned Type = L->getExtensionType();
3173:       if ((Type == ISD::ZEXTLOAD && C.ICmpType != SystemZICMP::SignedOnly) ||
3174:           (Type == ISD::SEXTLOAD && C.ICmpType != SystemZICMP::UnsignedOnly)) {
3175:         C.Op0 = C.Op0.getOperand(0);
3176:         C.Op1 = DAG.getConstant(0, DL, C.Op0.getValueType());
3177:       }
3178:     }
3179:   }
3180: }
3181: 
3182: // Return true if shift operation N has an in-range constant shift value.
3183: // Store it in ShiftVal if so.
3184: static bool isSimpleShift(SDValue N, unsigned &ShiftVal) {
3185:   auto *Shift = dyn_cast<ConstantSDNode>(N.getOperand(1));
3186:   if (!Shift)
3187:     return false;
3188: 
3189:   uint64_t Amount = Shift->getZExtValue();
3190:   if (Amount >= N.getValueSizeInBits())
3191:     return false;
3192: 
3193:   ShiftVal = Amount;
3194:   return true;
3195: }
3196: 
3197: // Check whether an AND with Mask is suitable for a TEST UNDER MASK
3198: // instruction and whether the CC value is descriptive enough to handle
3199: // a comparison of type Opcode between the AND result and CmpVal.
3200: // CCMask says which comparison result is being tested and BitSize is
3201: // the number of bits in the operands.  If TEST UNDER MASK can be used,
3202: // return the corresponding CC mask, otherwise return 0.
3203: static unsigned getTestUnderMaskCond(unsigned BitSize, unsigned CCMask,
3204:                                      uint64_t Mask, uint64_t CmpVal,
3205:                                      unsigned ICmpType) {
3206:   assert(Mask != 0 && "ANDs with zero should have been removed by now");
3207: 
3208:   // Check whether the mask is suitable for TMHH, TMHL, TMLH or TMLL.
3209:   if (!SystemZ::isImmLL(Mask) && !SystemZ::isImmLH(Mask) &&
3210:       !SystemZ::isImmHL(Mask) && !SystemZ::isImmHH(Mask))
3211:     return 0;
3212: 
3213:   // Work out the masks for the lowest and highest bits.
3214:   uint64_t High = llvm::bit_floor(Mask);
3215:   uint64_t Low = uint64_t(1) << llvm::countr_zero(Mask);
3216: 
3217:   // Signed ordered comparisons are effectively unsigned if the sign
3218:   // bit is dropped.
3219:   bool EffectivelyUnsigned = (ICmpType != SystemZICMP::SignedOnly);
3220: 
3221:   // Check for equality comparisons with 0, or the equivalent.
3222:   if (CmpVal == 0) {
3223:     if (CCMask == SystemZ::CCMASK_CMP_EQ)
3224:       return SystemZ::CCMASK_TM_ALL_0;
3225:     if (CCMask == SystemZ::CCMASK_CMP_NE)
```
- **EN**: The range implements or declares functions including `adjustICmpTruncate`, `isSimpleShift`, `getTestUnderMaskCond`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `adjustICmpTruncate`, `isSimpleShift`, `getTestUnderMaskCond` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 3226-3300 / 第 3226-3300 行
```cpp
3226:       return SystemZ::CCMASK_TM_SOME_1;
3227:   }
3228:   if (EffectivelyUnsigned && CmpVal > 0 && CmpVal <= Low) {
3229:     if (CCMask == SystemZ::CCMASK_CMP_LT)
3230:       return SystemZ::CCMASK_TM_ALL_0;
3231:     if (CCMask == SystemZ::CCMASK_CMP_GE)
3232:       return SystemZ::CCMASK_TM_SOME_1;
3233:   }
3234:   if (EffectivelyUnsigned && CmpVal < Low) {
3235:     if (CCMask == SystemZ::CCMASK_CMP_LE)
3236:       return SystemZ::CCMASK_TM_ALL_0;
3237:     if (CCMask == SystemZ::CCMASK_CMP_GT)
3238:       return SystemZ::CCMASK_TM_SOME_1;
3239:   }
3240: 
3241:   // Check for equality comparisons with the mask, or the equivalent.
3242:   if (CmpVal == Mask) {
3243:     if (CCMask == SystemZ::CCMASK_CMP_EQ)
3244:       return SystemZ::CCMASK_TM_ALL_1;
3245:     if (CCMask == SystemZ::CCMASK_CMP_NE)
3246:       return SystemZ::CCMASK_TM_SOME_0;
3247:   }
3248:   if (EffectivelyUnsigned && CmpVal >= Mask - Low && CmpVal < Mask) {
3249:     if (CCMask == SystemZ::CCMASK_CMP_GT)
3250:       return SystemZ::CCMASK_TM_ALL_1;
3251:     if (CCMask == SystemZ::CCMASK_CMP_LE)
3252:       return SystemZ::CCMASK_TM_SOME_0;
3253:   }
3254:   if (EffectivelyUnsigned && CmpVal > Mask - Low && CmpVal <= Mask) {
3255:     if (CCMask == SystemZ::CCMASK_CMP_GE)
3256:       return SystemZ::CCMASK_TM_ALL_1;
3257:     if (CCMask == SystemZ::CCMASK_CMP_LT)
3258:       return SystemZ::CCMASK_TM_SOME_0;
3259:   }
3260: 
3261:   // Check for ordered comparisons with the top bit.
3262:   if (EffectivelyUnsigned && CmpVal >= Mask - High && CmpVal < High) {
3263:     if (CCMask == SystemZ::CCMASK_CMP_LE)
3264:       return SystemZ::CCMASK_TM_MSB_0;
3265:     if (CCMask == SystemZ::CCMASK_CMP_GT)
3266:       return SystemZ::CCMASK_TM_MSB_1;
3267:   }
3268:   if (EffectivelyUnsigned && CmpVal > Mask - High && CmpVal <= High) {
3269:     if (CCMask == SystemZ::CCMASK_CMP_LT)
3270:       return SystemZ::CCMASK_TM_MSB_0;
3271:     if (CCMask == SystemZ::CCMASK_CMP_GE)
3272:       return SystemZ::CCMASK_TM_MSB_1;
3273:   }
3274: 
3275:   // If there are just two bits, we can do equality checks for Low and High
3276:   // as well.
3277:   if (Mask == Low + High) {
3278:     if (CCMask == SystemZ::CCMASK_CMP_EQ && CmpVal == Low)
3279:       return SystemZ::CCMASK_TM_MIXED_MSB_0;
3280:     if (CCMask == SystemZ::CCMASK_CMP_NE && CmpVal == Low)
3281:       return SystemZ::CCMASK_TM_MIXED_MSB_0 ^ SystemZ::CCMASK_ANY;
3282:     if (CCMask == SystemZ::CCMASK_CMP_EQ && CmpVal == High)
3283:       return SystemZ::CCMASK_TM_MIXED_MSB_1;
3284:     if (CCMask == SystemZ::CCMASK_CMP_NE && CmpVal == High)
3285:       return SystemZ::CCMASK_TM_MIXED_MSB_1 ^ SystemZ::CCMASK_ANY;
3286:   }
3287: 
3288:   // Looks like we've exhausted our options.
3289:   return 0;
3290: }
3291: 
3292: // See whether C can be implemented as a TEST UNDER MASK instruction.
3293: // Update the arguments with the TM version if so.
3294: static void adjustForTestUnderMask(SelectionDAG &DAG, const SDLoc &DL,
3295:                                    Comparison &C) {
3296:   // Use VECTOR TEST UNDER MASK for i128 operations.
3297:   if (C.Op0.getValueType() == MVT::i128) {
3298:     // We can use VTM for EQ/NE comparisons of x & y against 0.
3299:     if (C.Op0.getOpcode() == ISD::AND &&
3300:         (C.CCMask == SystemZ::CCMASK_CMP_EQ ||
```
- **EN**: The range implements or declares functions including `adjustForTestUnderMask`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `adjustForTestUnderMask` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3301-3375 / 第 3301-3375 行
```cpp
3301:          C.CCMask == SystemZ::CCMASK_CMP_NE)) {
3302:       auto *Mask = dyn_cast<ConstantSDNode>(C.Op1);
3303:       if (Mask && Mask->getAPIntValue() == 0) {
3304:         C.Opcode = SystemZISD::VTM;
3305:         C.Op1 = DAG.getNode(ISD::BITCAST, DL, MVT::v16i8, C.Op0.getOperand(1));
3306:         C.Op0 = DAG.getNode(ISD::BITCAST, DL, MVT::v16i8, C.Op0.getOperand(0));
3307:         C.CCValid = SystemZ::CCMASK_VCMP;
3308:         if (C.CCMask == SystemZ::CCMASK_CMP_EQ)
3309:           C.CCMask = SystemZ::CCMASK_VCMP_ALL;
3310:         else
3311:           C.CCMask = SystemZ::CCMASK_VCMP_ALL ^ C.CCValid;
3312:       }
3313:     }
3314:     return;
3315:   }
3316: 
3317:   // Check that we have a comparison with a constant.
3318:   auto *ConstOp1 = dyn_cast<ConstantSDNode>(C.Op1);
3319:   if (!ConstOp1)
3320:     return;
3321:   uint64_t CmpVal = ConstOp1->getZExtValue();
3322: 
3323:   // Check whether the nonconstant input is an AND with a constant mask.
3324:   Comparison NewC(C);
3325:   uint64_t MaskVal;
3326:   ConstantSDNode *Mask = nullptr;
3327:   if (C.Op0.getOpcode() == ISD::AND) {
3328:     NewC.Op0 = C.Op0.getOperand(0);
3329:     NewC.Op1 = C.Op0.getOperand(1);
3330:     Mask = dyn_cast<ConstantSDNode>(NewC.Op1);
3331:     if (!Mask)
3332:       return;
3333:     MaskVal = Mask->getZExtValue();
3334:   } else {
3335:     // There is no instruction to compare with a 64-bit immediate
3336:     // so use TMHH instead if possible.  We need an unsigned ordered
3337:     // comparison with an i64 immediate.
3338:     if (NewC.Op0.getValueType() != MVT::i64 ||
3339:         NewC.CCMask == SystemZ::CCMASK_CMP_EQ ||
3340:         NewC.CCMask == SystemZ::CCMASK_CMP_NE ||
3341:         NewC.ICmpType == SystemZICMP::SignedOnly)
3342:       return;
3343:     // Convert LE and GT comparisons into LT and GE.
3344:     if (NewC.CCMask == SystemZ::CCMASK_CMP_LE ||
3345:         NewC.CCMask == SystemZ::CCMASK_CMP_GT) {
3346:       if (CmpVal == uint64_t(-1))
3347:         return;
3348:       CmpVal += 1;
3349:       NewC.CCMask ^= SystemZ::CCMASK_CMP_EQ;
3350:     }
3351:     // If the low N bits of Op1 are zero than the low N bits of Op0 can
3352:     // be masked off without changing the result.
3353:     MaskVal = -(CmpVal & -CmpVal);
3354:     NewC.ICmpType = SystemZICMP::UnsignedOnly;
3355:   }
3356:   if (!MaskVal)
3357:     return;
3358: 
3359:   // Check whether the combination of mask, comparison value and comparison
3360:   // type are suitable.
3361:   unsigned BitSize = NewC.Op0.getValueSizeInBits();
3362:   unsigned NewCCMask, ShiftVal;
3363:   if (NewC.ICmpType != SystemZICMP::SignedOnly &&
3364:       NewC.Op0.getOpcode() == ISD::SHL &&
3365:       isSimpleShift(NewC.Op0, ShiftVal) &&
3366:       (MaskVal >> ShiftVal != 0) &&
3367:       ((CmpVal >> ShiftVal) << ShiftVal) == CmpVal &&
3368:       (NewCCMask = getTestUnderMaskCond(BitSize, NewC.CCMask,
3369:                                         MaskVal >> ShiftVal,
3370:                                         CmpVal >> ShiftVal,
3371:                                         SystemZICMP::Any))) {
3372:     NewC.Op0 = NewC.Op0.getOperand(0);
3373:     MaskVal >>= ShiftVal;
3374:   } else if (NewC.ICmpType != SystemZICMP::SignedOnly &&
3375:              NewC.Op0.getOpcode() == ISD::SRL &&
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3376-3450 / 第 3376-3450 行
```cpp
3376:              isSimpleShift(NewC.Op0, ShiftVal) &&
3377:              (MaskVal << ShiftVal != 0) &&
3378:              ((CmpVal << ShiftVal) >> ShiftVal) == CmpVal &&
3379:              (NewCCMask = getTestUnderMaskCond(BitSize, NewC.CCMask,
3380:                                                MaskVal << ShiftVal,
3381:                                                CmpVal << ShiftVal,
3382:                                                SystemZICMP::UnsignedOnly))) {
3383:     NewC.Op0 = NewC.Op0.getOperand(0);
3384:     MaskVal <<= ShiftVal;
3385:   } else {
3386:     NewCCMask = getTestUnderMaskCond(BitSize, NewC.CCMask, MaskVal, CmpVal,
3387:                                      NewC.ICmpType);
3388:     if (!NewCCMask)
3389:       return;
3390:   }
3391: 
3392:   // Go ahead and make the change.
3393:   C.Opcode = SystemZISD::TM;
3394:   C.Op0 = NewC.Op0;
3395:   if (Mask && Mask->getZExtValue() == MaskVal)
3396:     C.Op1 = SDValue(Mask, 0);
3397:   else
3398:     C.Op1 = DAG.getConstant(MaskVal, DL, C.Op0.getValueType());
3399:   C.CCValid = SystemZ::CCMASK_TM;
3400:   C.CCMask = NewCCMask;
3401: }
3402: 
3403: // Implement i128 comparison in vector registers.
3404: static void adjustICmp128(SelectionDAG &DAG, const SDLoc &DL,
3405:                           Comparison &C) {
3406:   if (C.Opcode != SystemZISD::ICMP)
3407:     return;
3408:   if (C.Op0.getValueType() != MVT::i128)
3409:     return;
3410: 
3411:   // Recognize vector comparison reductions.
3412:   if ((C.CCMask == SystemZ::CCMASK_CMP_EQ ||
3413:        C.CCMask == SystemZ::CCMASK_CMP_NE) &&
3414:       (isNullConstant(C.Op1) || isAllOnesConstant(C.Op1))) {
3415:     bool CmpEq = C.CCMask == SystemZ::CCMASK_CMP_EQ;
3416:     bool CmpNull = isNullConstant(C.Op1);
3417:     SDValue Src = peekThroughBitcasts(C.Op0);
3418:     if (Src.hasOneUse() && isBitwiseNot(Src)) {
3419:       Src = Src.getOperand(0);
3420:       CmpNull = !CmpNull;
3421:     }
3422:     unsigned Opcode = 0;
3423:     if (Src.hasOneUse()) {
3424:       switch (Src.getOpcode()) {
3425:       case SystemZISD::VICMPE: Opcode = SystemZISD::VICMPES; break;
3426:       case SystemZISD::VICMPH: Opcode = SystemZISD::VICMPHS; break;
3427:       case SystemZISD::VICMPHL: Opcode = SystemZISD::VICMPHLS; break;
3428:       case SystemZISD::VFCMPE: Opcode = SystemZISD::VFCMPES; break;
3429:       case SystemZISD::VFCMPH: Opcode = SystemZISD::VFCMPHS; break;
3430:       case SystemZISD::VFCMPHE: Opcode = SystemZISD::VFCMPHES; break;
3431:       default: break;
3432:       }
3433:     }
3434:     if (Opcode) {
3435:       C.Opcode = Opcode;
3436:       C.Op0 = Src->getOperand(0);
3437:       C.Op1 = Src->getOperand(1);
3438:       C.CCValid = SystemZ::CCMASK_VCMP;
3439:       C.CCMask = CmpNull ? SystemZ::CCMASK_VCMP_NONE : SystemZ::CCMASK_VCMP_ALL;
3440:       if (!CmpEq)
3441:         C.CCMask ^= C.CCValid;
3442:       return;
3443:     }
3444:   }
3445: 
3446:   // Everything below here is not useful if we have native i128 compares.
3447:   if (DAG.getSubtarget<SystemZSubtarget>().hasVectorEnhancements3())
3448:     return;
3449: 
3450:   // (In-)Equality comparisons can be implemented via VCEQGS.
```
- **EN**: The range implements or declares functions including `isSimpleShift`, `adjustICmp128`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isSimpleShift`, `adjustICmp128` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3451-3525 / 第 3451-3525 行
```cpp
3451:   if (C.CCMask == SystemZ::CCMASK_CMP_EQ ||
3452:       C.CCMask == SystemZ::CCMASK_CMP_NE) {
3453:     C.Opcode = SystemZISD::VICMPES;
3454:     C.Op0 = DAG.getNode(ISD::BITCAST, DL, MVT::v2i64, C.Op0);
3455:     C.Op1 = DAG.getNode(ISD::BITCAST, DL, MVT::v2i64, C.Op1);
3456:     C.CCValid = SystemZ::CCMASK_VCMP;
3457:     if (C.CCMask == SystemZ::CCMASK_CMP_EQ)
3458:       C.CCMask = SystemZ::CCMASK_VCMP_ALL;
3459:     else
3460:       C.CCMask = SystemZ::CCMASK_VCMP_ALL ^ C.CCValid;
3461:     return;
3462:   }
3463: 
3464:   // Normalize other comparisons to GT.
3465:   bool Swap = false, Invert = false;
3466:   switch (C.CCMask) {
3467:     case SystemZ::CCMASK_CMP_GT: break;
3468:     case SystemZ::CCMASK_CMP_LT: Swap = true; break;
3469:     case SystemZ::CCMASK_CMP_LE: Invert = true; break;
3470:     case SystemZ::CCMASK_CMP_GE: Swap = Invert = true; break;
3471:     default: llvm_unreachable("Invalid integer condition!");
3472:   }
3473:   if (Swap)
3474:     std::swap(C.Op0, C.Op1);
3475: 
3476:   if (C.ICmpType == SystemZICMP::UnsignedOnly)
3477:     C.Opcode = SystemZISD::UCMP128HI;
3478:   else
3479:     C.Opcode = SystemZISD::SCMP128HI;
3480:   C.CCValid = SystemZ::CCMASK_ANY;
3481:   C.CCMask = SystemZ::CCMASK_1;
3482: 
3483:   if (Invert)
3484:     C.CCMask ^= C.CCValid;
3485: }
3486: 
3487: // See whether the comparison argument contains a redundant AND
3488: // and remove it if so.  This sometimes happens due to the generic
3489: // BRCOND expansion.
3490: static void adjustForRedundantAnd(SelectionDAG &DAG, const SDLoc &DL,
3491:                                   Comparison &C) {
3492:   if (C.Op0.getOpcode() != ISD::AND)
3493:     return;
3494:   auto *Mask = dyn_cast<ConstantSDNode>(C.Op0.getOperand(1));
3495:   if (!Mask || Mask->getValueSizeInBits(0) > 64)
3496:     return;
3497:   KnownBits Known = DAG.computeKnownBits(C.Op0.getOperand(0));
3498:   if ((~Known.Zero).getZExtValue() & ~Mask->getZExtValue())
3499:     return;
3500: 
3501:   C.Op0 = C.Op0.getOperand(0);
3502: }
3503: 
3504: // Return a Comparison that tests the condition-code result of intrinsic
3505: // node Call against constant integer CC using comparison code Cond.
3506: // Opcode is the opcode of the SystemZISD operation for the intrinsic
3507: // and CCValid is the set of possible condition-code results.
3508: static Comparison getIntrinsicCmp(SelectionDAG &DAG, unsigned Opcode,
3509:                                   SDValue Call, unsigned CCValid, uint64_t CC,
3510:                                   ISD::CondCode Cond) {
3511:   Comparison C(Call, SDValue(), SDValue());
3512:   C.Opcode = Opcode;
3513:   C.CCValid = CCValid;
3514:   if (Cond == ISD::SETEQ)
3515:     // bit 3 for CC==0, bit 0 for CC==3, always false for CC>3.
3516:     C.CCMask = CC < 4 ? 1 << (3 - CC) : 0;
3517:   else if (Cond == ISD::SETNE)
3518:     // ...and the inverse of that.
3519:     C.CCMask = CC < 4 ? ~(1 << (3 - CC)) : -1;
3520:   else if (Cond == ISD::SETLT || Cond == ISD::SETULT)
3521:     // bits above bit 3 for CC==0 (always false), bits above bit 0 for CC==3,
3522:     // always true for CC>3.
3523:     C.CCMask = CC < 4 ? ~0U << (4 - CC) : -1;
3524:   else if (Cond == ISD::SETGE || Cond == ISD::SETUGE)
3525:     // ...and the inverse of that.
```
- **EN**: The range implements or declares functions including `adjustForRedundantAnd`, `getIntrinsicCmp`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `adjustForRedundantAnd`, `getIntrinsicCmp` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3526-3600 / 第 3526-3600 行
```cpp
3526:     C.CCMask = CC < 4 ? ~(~0U << (4 - CC)) : 0;
3527:   else if (Cond == ISD::SETLE || Cond == ISD::SETULE)
3528:     // bit 3 and above for CC==0, bit 0 and above for CC==3 (always true),
3529:     // always true for CC>3.
3530:     C.CCMask = CC < 4 ? ~0U << (3 - CC) : -1;
3531:   else if (Cond == ISD::SETGT || Cond == ISD::SETUGT)
3532:     // ...and the inverse of that.
3533:     C.CCMask = CC < 4 ? ~(~0U << (3 - CC)) : 0;
3534:   else
3535:     llvm_unreachable("Unexpected integer comparison type");
3536:   C.CCMask &= CCValid;
3537:   return C;
3538: }
3539: 
3540: // Decide how to implement a comparison of type Cond between CmpOp0 with CmpOp1.
3541: static Comparison getCmp(SelectionDAG &DAG, SDValue CmpOp0, SDValue CmpOp1,
3542:                          ISD::CondCode Cond, const SDLoc &DL,
3543:                          SDValue Chain = SDValue(),
3544:                          bool IsSignaling = false) {
3545:   if (CmpOp1.getOpcode() == ISD::Constant) {
3546:     assert(!Chain);
3547:     unsigned Opcode, CCValid;
3548:     if (CmpOp0.getOpcode() == ISD::INTRINSIC_W_CHAIN &&
3549:         CmpOp0.getResNo() == 0 && CmpOp0->hasNUsesOfValue(1, 0) &&
3550:         isIntrinsicWithCCAndChain(CmpOp0, Opcode, CCValid))
3551:       return getIntrinsicCmp(DAG, Opcode, CmpOp0, CCValid,
3552:                              CmpOp1->getAsZExtVal(), Cond);
3553:     if (CmpOp0.getOpcode() == ISD::INTRINSIC_WO_CHAIN &&
3554:         CmpOp0.getResNo() == CmpOp0->getNumValues() - 1 &&
3555:         isIntrinsicWithCC(CmpOp0, Opcode, CCValid))
3556:       return getIntrinsicCmp(DAG, Opcode, CmpOp0, CCValid,
3557:                              CmpOp1->getAsZExtVal(), Cond);
3558:   }
3559:   Comparison C(CmpOp0, CmpOp1, Chain);
3560:   C.CCMask = CCMaskForCondCode(Cond);
3561:   if (C.Op0.getValueType().isFloatingPoint()) {
3562:     C.CCValid = SystemZ::CCMASK_FCMP;
3563:     if (!C.Chain)
3564:       C.Opcode = SystemZISD::FCMP;
3565:     else if (!IsSignaling)
3566:       C.Opcode = SystemZISD::STRICT_FCMP;
3567:     else
3568:       C.Opcode = SystemZISD::STRICT_FCMPS;
3569:     adjustForFNeg(C);
3570:   } else {
3571:     assert(!C.Chain);
3572:     C.CCValid = SystemZ::CCMASK_ICMP;
3573:     C.Opcode = SystemZISD::ICMP;
3574:     // Choose the type of comparison.  Equality and inequality tests can
3575:     // use either signed or unsigned comparisons.  The choice also doesn't
3576:     // matter if both sign bits are known to be clear.  In those cases we
3577:     // want to give the main isel code the freedom to choose whichever
3578:     // form fits best.
3579:     if (C.CCMask == SystemZ::CCMASK_CMP_EQ ||
3580:         C.CCMask == SystemZ::CCMASK_CMP_NE ||
3581:         (DAG.SignBitIsZero(C.Op0) && DAG.SignBitIsZero(C.Op1)))
3582:       C.ICmpType = SystemZICMP::Any;
3583:     else if (C.CCMask & SystemZ::CCMASK_CMP_UO)
3584:       C.ICmpType = SystemZICMP::UnsignedOnly;
3585:     else
3586:       C.ICmpType = SystemZICMP::SignedOnly;
3587:     C.CCMask &= ~SystemZ::CCMASK_CMP_UO;
3588:     adjustForRedundantAnd(DAG, DL, C);
3589:     adjustZeroCmp(DAG, DL, C);
3590:     adjustSubwordCmp(DAG, DL, C);
3591:     adjustForSubtraction(DAG, DL, C);
3592:     adjustForLTGFR(C);
3593:     adjustICmpTruncate(DAG, DL, C);
3594:   }
3595: 
3596:   if (shouldSwapCmpOperands(C)) {
3597:     std::swap(C.Op0, C.Op1);
3598:     C.CCMask = SystemZ::reverseCCMask(C.CCMask);
3599:   }
3600: 
```
- **EN**: The range implements or declares functions including `getCmp`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `getCmp` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 3601-3675 / 第 3601-3675 行
```cpp
3601:   adjustForTestUnderMask(DAG, DL, C);
3602:   adjustICmp128(DAG, DL, C);
3603:   return C;
3604: }
3605: 
3606: // Emit the comparison instruction described by C.
3607: static SDValue emitCmp(SelectionDAG &DAG, const SDLoc &DL, Comparison &C) {
3608:   if (!C.Op1.getNode()) {
3609:     SDNode *Node;
3610:     switch (C.Op0.getOpcode()) {
3611:     case ISD::INTRINSIC_W_CHAIN:
3612:       Node = emitIntrinsicWithCCAndChain(DAG, C.Op0, C.Opcode);
3613:       return SDValue(Node, 0);
3614:     case ISD::INTRINSIC_WO_CHAIN:
3615:       Node = emitIntrinsicWithCC(DAG, C.Op0, C.Opcode);
3616:       return SDValue(Node, Node->getNumValues() - 1);
3617:     default:
3618:       llvm_unreachable("Invalid comparison operands");
3619:     }
3620:   }
3621:   if (C.Opcode == SystemZISD::ICMP)
3622:     return DAG.getNode(SystemZISD::ICMP, DL, MVT::i32, C.Op0, C.Op1,
3623:                        DAG.getTargetConstant(C.ICmpType, DL, MVT::i32));
3624:   if (C.Opcode == SystemZISD::TM) {
3625:     bool RegisterOnly = (bool(C.CCMask & SystemZ::CCMASK_TM_MIXED_MSB_0) !=
3626:                          bool(C.CCMask & SystemZ::CCMASK_TM_MIXED_MSB_1));
3627:     return DAG.getNode(SystemZISD::TM, DL, MVT::i32, C.Op0, C.Op1,
3628:                        DAG.getTargetConstant(RegisterOnly, DL, MVT::i32));
3629:   }
3630:   if (C.Opcode == SystemZISD::VICMPES ||
3631:       C.Opcode == SystemZISD::VICMPHS ||
3632:       C.Opcode == SystemZISD::VICMPHLS ||
3633:       C.Opcode == SystemZISD::VFCMPES ||
3634:       C.Opcode == SystemZISD::VFCMPHS ||
3635:       C.Opcode == SystemZISD::VFCMPHES) {
3636:     EVT IntVT = C.Op0.getValueType().changeVectorElementTypeToInteger();
3637:     SDVTList VTs = DAG.getVTList(IntVT, MVT::i32);
3638:     SDValue Val = DAG.getNode(C.Opcode, DL, VTs, C.Op0, C.Op1);
3639:     return SDValue(Val.getNode(), 1);
3640:   }
3641:   if (C.Chain) {
3642:     SDVTList VTs = DAG.getVTList(MVT::i32, MVT::Other);
3643:     return DAG.getNode(C.Opcode, DL, VTs, C.Chain, C.Op0, C.Op1);
3644:   }
3645:   return DAG.getNode(C.Opcode, DL, MVT::i32, C.Op0, C.Op1);
3646: }
3647: 
3648: // Implement a 32-bit *MUL_LOHI operation by extending both operands to
3649: // 64 bits.  Extend is the extension type to use.  Store the high part
3650: // in Hi and the low part in Lo.
3651: static void lowerMUL_LOHI32(SelectionDAG &DAG, const SDLoc &DL, unsigned Extend,
3652:                             SDValue Op0, SDValue Op1, SDValue &Hi,
3653:                             SDValue &Lo) {
3654:   Op0 = DAG.getNode(Extend, DL, MVT::i64, Op0);
3655:   Op1 = DAG.getNode(Extend, DL, MVT::i64, Op1);
3656:   SDValue Mul = DAG.getNode(ISD::MUL, DL, MVT::i64, Op0, Op1);
3657:   Hi = DAG.getNode(ISD::SRL, DL, MVT::i64, Mul,
3658:                    DAG.getConstant(32, DL, MVT::i64));
3659:   Hi = DAG.getNode(ISD::TRUNCATE, DL, MVT::i32, Hi);
3660:   Lo = DAG.getNode(ISD::TRUNCATE, DL, MVT::i32, Mul);
3661: }
3662: 
3663: // Lower a binary operation that produces two VT results, one in each
3664: // half of a GR128 pair.  Op0 and Op1 are the VT operands to the operation,
3665: // and Opcode performs the GR128 operation.  Store the even register result
3666: // in Even and the odd register result in Odd.
3667: static void lowerGR128Binary(SelectionDAG &DAG, const SDLoc &DL, EVT VT,
3668:                              unsigned Opcode, SDValue Op0, SDValue Op1,
3669:                              SDValue &Even, SDValue &Odd) {
3670:   SDValue Result = DAG.getNode(Opcode, DL, MVT::Untyped, Op0, Op1);
3671:   bool Is32Bit = is32Bit(VT);
3672:   Even = DAG.getTargetExtractSubreg(SystemZ::even128(Is32Bit), DL, VT, Result);
3673:   Odd = DAG.getTargetExtractSubreg(SystemZ::odd128(Is32Bit), DL, VT, Result);
3674: }
3675: 
```
- **EN**: The range implements or declares functions including `emitCmp`, `lowerMUL_LOHI32`, `lowerGR128Binary`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `emitCmp`, `lowerMUL_LOHI32`, `lowerGR128Binary` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3676-3750 / 第 3676-3750 行
```cpp
3676: // Return an i32 value that is 1 if the CC value produced by CCReg is
3677: // in the mask CCMask and 0 otherwise.  CC is known to have a value
3678: // in CCValid, so other values can be ignored.
3679: static SDValue emitSETCC(SelectionDAG &DAG, const SDLoc &DL, SDValue CCReg,
3680:                          unsigned CCValid, unsigned CCMask) {
3681:   SDValue Ops[] = {DAG.getConstant(1, DL, MVT::i32),
3682:                    DAG.getConstant(0, DL, MVT::i32),
3683:                    DAG.getTargetConstant(CCValid, DL, MVT::i32),
3684:                    DAG.getTargetConstant(CCMask, DL, MVT::i32), CCReg};
3685:   return DAG.getNode(SystemZISD::SELECT_CCMASK, DL, MVT::i32, Ops);
3686: }
3687: 
3688: // Return the SystemISD vector comparison operation for CC, or 0 if it cannot
3689: // be done directly.  Mode is CmpMode::Int for integer comparisons, CmpMode::FP
3690: // for regular floating-point comparisons, CmpMode::StrictFP for strict (quiet)
3691: // floating-point comparisons, and CmpMode::SignalingFP for strict signaling
3692: // floating-point comparisons.
3693: enum class CmpMode { Int, FP, StrictFP, SignalingFP };
3694: static unsigned getVectorComparison(ISD::CondCode CC, CmpMode Mode) {
3695:   switch (CC) {
3696:   case ISD::SETOEQ:
3697:   case ISD::SETEQ:
3698:     switch (Mode) {
3699:     case CmpMode::Int:         return SystemZISD::VICMPE;
3700:     case CmpMode::FP:          return SystemZISD::VFCMPE;
3701:     case CmpMode::StrictFP:    return SystemZISD::STRICT_VFCMPE;
3702:     case CmpMode::SignalingFP: return SystemZISD::STRICT_VFCMPES;
3703:     }
3704:     llvm_unreachable("Bad mode");
3705: 
3706:   case ISD::SETOGE:
3707:   case ISD::SETGE:
3708:     switch (Mode) {
3709:     case CmpMode::Int:         return 0;
3710:     case CmpMode::FP:          return SystemZISD::VFCMPHE;
3711:     case CmpMode::StrictFP:    return SystemZISD::STRICT_VFCMPHE;
3712:     case CmpMode::SignalingFP: return SystemZISD::STRICT_VFCMPHES;
3713:     }
3714:     llvm_unreachable("Bad mode");
3715: 
3716:   case ISD::SETOGT:
3717:   case ISD::SETGT:
3718:     switch (Mode) {
3719:     case CmpMode::Int:         return SystemZISD::VICMPH;
3720:     case CmpMode::FP:          return SystemZISD::VFCMPH;
3721:     case CmpMode::StrictFP:    return SystemZISD::STRICT_VFCMPH;
3722:     case CmpMode::SignalingFP: return SystemZISD::STRICT_VFCMPHS;
3723:     }
3724:     llvm_unreachable("Bad mode");
3725: 
3726:   case ISD::SETUGT:
3727:     switch (Mode) {
3728:     case CmpMode::Int:         return SystemZISD::VICMPHL;
3729:     case CmpMode::FP:          return 0;
3730:     case CmpMode::StrictFP:    return 0;
3731:     case CmpMode::SignalingFP: return 0;
3732:     }
3733:     llvm_unreachable("Bad mode");
3734: 
3735:   default:
3736:     return 0;
3737:   }
3738: }
3739: 
3740: // Return the SystemZISD vector comparison operation for CC or its inverse,
3741: // or 0 if neither can be done directly.  Indicate in Invert whether the
3742: // result is for the inverse of CC.  Mode is as above.
3743: static unsigned getVectorComparisonOrInvert(ISD::CondCode CC, CmpMode Mode,
3744:                                             bool &Invert) {
3745:   if (unsigned Opcode = getVectorComparison(CC, Mode)) {
3746:     Invert = false;
3747:     return Opcode;
3748:   }
3749: 
3750:   CC = ISD::getSetCCInverse(CC, Mode == CmpMode::Int ? MVT::i32 : MVT::f32);
```
- **EN**: The range implements or declares functions including `emitSETCC`, `getVectorComparison`, `getVectorComparisonOrInvert`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `emitSETCC`, `getVectorComparison`, `getVectorComparisonOrInvert` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3751-3825 / 第 3751-3825 行
```cpp
3751:   if (unsigned Opcode = getVectorComparison(CC, Mode)) {
3752:     Invert = true;
3753:     return Opcode;
3754:   }
3755: 
3756:   return 0;
3757: }
3758: 
3759: // Return a v2f64 that contains the extended form of elements Start and Start+1
3760: // of v4f32 value Op.  If Chain is nonnull, return the strict form.
3761: static SDValue expandV4F32ToV2F64(SelectionDAG &DAG, int Start, const SDLoc &DL,
3762:                                   SDValue Op, SDValue Chain) {
3763:   int Mask[] = { Start, -1, Start + 1, -1 };
3764:   Op = DAG.getVectorShuffle(MVT::v4f32, DL, Op, DAG.getUNDEF(MVT::v4f32), Mask);
3765:   if (Chain) {
3766:     SDVTList VTs = DAG.getVTList(MVT::v2f64, MVT::Other);
3767:     return DAG.getNode(SystemZISD::STRICT_VEXTEND, DL, VTs, Chain, Op);
3768:   }
3769:   return DAG.getNode(SystemZISD::VEXTEND, DL, MVT::v2f64, Op);
3770: }
3771: 
3772: // Build a comparison of vectors CmpOp0 and CmpOp1 using opcode Opcode,
3773: // producing a result of type VT.  If Chain is nonnull, return the strict form.
3774: SDValue SystemZTargetLowering::getVectorCmp(SelectionDAG &DAG, unsigned Opcode,
3775:                                             const SDLoc &DL, EVT VT,
3776:                                             SDValue CmpOp0,
3777:                                             SDValue CmpOp1,
3778:                                             SDValue Chain) const {
3779:   // There is no hardware support for v4f32 (unless we have the vector
3780:   // enhancements facility 1), so extend the vector into two v2f64s
3781:   // and compare those.
3782:   if (CmpOp0.getValueType() == MVT::v4f32 &&
3783:       !Subtarget.hasVectorEnhancements1()) {
3784:     SDValue H0 = expandV4F32ToV2F64(DAG, 0, DL, CmpOp0, Chain);
3785:     SDValue L0 = expandV4F32ToV2F64(DAG, 2, DL, CmpOp0, Chain);
3786:     SDValue H1 = expandV4F32ToV2F64(DAG, 0, DL, CmpOp1, Chain);
3787:     SDValue L1 = expandV4F32ToV2F64(DAG, 2, DL, CmpOp1, Chain);
3788:     if (Chain) {
3789:       SDVTList VTs = DAG.getVTList(MVT::v2i64, MVT::Other);
3790:       SDValue HRes = DAG.getNode(Opcode, DL, VTs, Chain, H0, H1);
3791:       SDValue LRes = DAG.getNode(Opcode, DL, VTs, Chain, L0, L1);
3792:       SDValue Res = DAG.getNode(SystemZISD::PACK, DL, VT, HRes, LRes);
3793:       SDValue Chains[6] = { H0.getValue(1), L0.getValue(1),
3794:                             H1.getValue(1), L1.getValue(1),
3795:                             HRes.getValue(1), LRes.getValue(1) };
3796:       SDValue NewChain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, Chains);
3797:       SDValue Ops[2] = { Res, NewChain };
3798:       return DAG.getMergeValues(Ops, DL);
3799:     }
3800:     SDValue HRes = DAG.getNode(Opcode, DL, MVT::v2i64, H0, H1);
3801:     SDValue LRes = DAG.getNode(Opcode, DL, MVT::v2i64, L0, L1);
3802:     return DAG.getNode(SystemZISD::PACK, DL, VT, HRes, LRes);
3803:   }
3804:   if (Chain) {
3805:     SDVTList VTs = DAG.getVTList(VT, MVT::Other);
3806:     return DAG.getNode(Opcode, DL, VTs, Chain, CmpOp0, CmpOp1);
3807:   }
3808:   return DAG.getNode(Opcode, DL, VT, CmpOp0, CmpOp1);
3809: }
3810: 
3811: // Lower a vector comparison of type CC between CmpOp0 and CmpOp1, producing
3812: // an integer mask of type VT.  If Chain is nonnull, we have a strict
3813: // floating-point comparison.  If in addition IsSignaling is true, we have
3814: // a strict signaling floating-point comparison.
3815: SDValue SystemZTargetLowering::lowerVectorSETCC(SelectionDAG &DAG,
3816:                                                 const SDLoc &DL, EVT VT,
3817:                                                 ISD::CondCode CC,
3818:                                                 SDValue CmpOp0,
3819:                                                 SDValue CmpOp1,
3820:                                                 SDValue Chain,
3821:                                                 bool IsSignaling) const {
3822:   bool IsFP = CmpOp0.getValueType().isFloatingPoint();
3823:   assert (!Chain || IsFP);
3824:   assert (!IsSignaling || Chain);
3825:   CmpMode Mode = IsSignaling ? CmpMode::SignalingFP :
```
- **EN**: The range implements or declares functions including `expandV4F32ToV2F64`, `SystemZTargetLowering::getVectorCmp`, `SystemZTargetLowering::lowerVectorSETCC`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `expandV4F32ToV2F64`, `SystemZTargetLowering::getVectorCmp`, `SystemZTargetLowering::lowerVectorSETCC` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3826-3900 / 第 3826-3900 行
```cpp
3826:                  Chain ? CmpMode::StrictFP : IsFP ? CmpMode::FP : CmpMode::Int;
3827:   bool Invert = false;
3828:   SDValue Cmp;
3829:   switch (CC) {
3830:     // Handle tests for order using (or (ogt y x) (oge x y)).
3831:   case ISD::SETUO:
3832:     Invert = true;
3833:     [[fallthrough]];
3834:   case ISD::SETO: {
3835:     assert(IsFP && "Unexpected integer comparison");
3836:     SDValue LT = getVectorCmp(DAG, getVectorComparison(ISD::SETOGT, Mode),
3837:                               DL, VT, CmpOp1, CmpOp0, Chain);
3838:     SDValue GE = getVectorCmp(DAG, getVectorComparison(ISD::SETOGE, Mode),
3839:                               DL, VT, CmpOp0, CmpOp1, Chain);
3840:     Cmp = DAG.getNode(ISD::OR, DL, VT, LT, GE);
3841:     if (Chain)
3842:       Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other,
3843:                           LT.getValue(1), GE.getValue(1));
3844:     break;
3845:   }
3846: 
3847:     // Handle <> tests using (or (ogt y x) (ogt x y)).
3848:   case ISD::SETUEQ:
3849:     Invert = true;
3850:     [[fallthrough]];
3851:   case ISD::SETONE: {
3852:     assert(IsFP && "Unexpected integer comparison");
3853:     SDValue LT = getVectorCmp(DAG, getVectorComparison(ISD::SETOGT, Mode),
3854:                               DL, VT, CmpOp1, CmpOp0, Chain);
3855:     SDValue GT = getVectorCmp(DAG, getVectorComparison(ISD::SETOGT, Mode),
3856:                               DL, VT, CmpOp0, CmpOp1, Chain);
3857:     Cmp = DAG.getNode(ISD::OR, DL, VT, LT, GT);
3858:     if (Chain)
3859:       Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other,
3860:                           LT.getValue(1), GT.getValue(1));
3861:     break;
3862:   }
3863: 
3864:     // Otherwise a single comparison is enough.  It doesn't really
3865:     // matter whether we try the inversion or the swap first, since
3866:     // there are no cases where both work.
3867:   default:
3868:     // Optimize sign-bit comparisons to signed compares.
3869:     if (Mode == CmpMode::Int && (CC == ISD::SETEQ || CC == ISD::SETNE) &&
3870:         ISD::isConstantSplatVectorAllZeros(CmpOp1.getNode())) {
3871:       unsigned EltSize = VT.getVectorElementType().getSizeInBits();
3872:       APInt Mask;
3873:       if (CmpOp0.getOpcode() == ISD::AND
3874:           && ISD::isConstantSplatVector(CmpOp0.getOperand(1).getNode(), Mask)
3875:           && Mask == APInt::getSignMask(EltSize)) {
3876:         CC = CC == ISD::SETEQ ? ISD::SETGE : ISD::SETLT;
3877:         CmpOp0 = CmpOp0.getOperand(0);
3878:       }
3879:     }
3880:     if (unsigned Opcode = getVectorComparisonOrInvert(CC, Mode, Invert))
3881:       Cmp = getVectorCmp(DAG, Opcode, DL, VT, CmpOp0, CmpOp1, Chain);
3882:     else {
3883:       CC = ISD::getSetCCSwappedOperands(CC);
3884:       if (unsigned Opcode = getVectorComparisonOrInvert(CC, Mode, Invert))
3885:         Cmp = getVectorCmp(DAG, Opcode, DL, VT, CmpOp1, CmpOp0, Chain);
3886:       else
3887:         llvm_unreachable("Unhandled comparison");
3888:     }
3889:     if (Chain)
3890:       Chain = Cmp.getValue(1);
3891:     break;
3892:   }
3893:   if (Invert) {
3894:     SDValue Mask =
3895:         DAG.getSplatBuildVector(VT, DL, DAG.getAllOnesConstant(DL, MVT::i64));
3896:     Cmp = DAG.getNode(ISD::XOR, DL, VT, Cmp, Mask);
3897:   }
3898:   if (Chain && Chain.getNode() != Cmp.getNode()) {
3899:     SDValue Ops[2] = { Cmp, Chain };
3900:     Cmp = DAG.getMergeValues(Ops, DL);
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 3901-3975 / 第 3901-3975 行
```cpp
3901:   }
3902:   return Cmp;
3903: }
3904: 
3905: SDValue SystemZTargetLowering::lowerSETCC(SDValue Op,
3906:                                           SelectionDAG &DAG) const {
3907:   SDValue CmpOp0   = Op.getOperand(0);
3908:   SDValue CmpOp1   = Op.getOperand(1);
3909:   ISD::CondCode CC = cast<CondCodeSDNode>(Op.getOperand(2))->get();
3910:   SDLoc DL(Op);
3911:   EVT VT = Op.getValueType();
3912:   if (VT.isVector())
3913:     return lowerVectorSETCC(DAG, DL, VT, CC, CmpOp0, CmpOp1);
3914: 
3915:   Comparison C(getCmp(DAG, CmpOp0, CmpOp1, CC, DL));
3916:   SDValue CCReg = emitCmp(DAG, DL, C);
3917:   return emitSETCC(DAG, DL, CCReg, C.CCValid, C.CCMask);
3918: }
3919: 
3920: SDValue SystemZTargetLowering::lowerSTRICT_FSETCC(SDValue Op,
3921:                                                   SelectionDAG &DAG,
3922:                                                   bool IsSignaling) const {
3923:   SDValue Chain    = Op.getOperand(0);
3924:   SDValue CmpOp0   = Op.getOperand(1);
3925:   SDValue CmpOp1   = Op.getOperand(2);
3926:   ISD::CondCode CC = cast<CondCodeSDNode>(Op.getOperand(3))->get();
3927:   SDLoc DL(Op);
3928:   EVT VT = Op.getNode()->getValueType(0);
3929:   if (VT.isVector()) {
3930:     SDValue Res = lowerVectorSETCC(DAG, DL, VT, CC, CmpOp0, CmpOp1,
3931:                                    Chain, IsSignaling);
3932:     return Res.getValue(Op.getResNo());
3933:   }
3934: 
3935:   Comparison C(getCmp(DAG, CmpOp0, CmpOp1, CC, DL, Chain, IsSignaling));
3936:   SDValue CCReg = emitCmp(DAG, DL, C);
3937:   CCReg->setFlags(Op->getFlags());
3938:   SDValue Result = emitSETCC(DAG, DL, CCReg, C.CCValid, C.CCMask);
3939:   SDValue Ops[2] = { Result, CCReg.getValue(1) };
3940:   return DAG.getMergeValues(Ops, DL);
3941: }
3942: 
3943: SDValue SystemZTargetLowering::lowerBR_CC(SDValue Op, SelectionDAG &DAG) const {
3944:   ISD::CondCode CC = cast<CondCodeSDNode>(Op.getOperand(1))->get();
3945:   SDValue CmpOp0   = Op.getOperand(2);
3946:   SDValue CmpOp1   = Op.getOperand(3);
3947:   SDValue Dest     = Op.getOperand(4);
3948:   SDLoc DL(Op);
3949: 
3950:   Comparison C(getCmp(DAG, CmpOp0, CmpOp1, CC, DL));
3951:   SDValue CCReg = emitCmp(DAG, DL, C);
3952:   return DAG.getNode(
3953:       SystemZISD::BR_CCMASK, DL, Op.getValueType(), Op.getOperand(0),
3954:       DAG.getTargetConstant(C.CCValid, DL, MVT::i32),
3955:       DAG.getTargetConstant(C.CCMask, DL, MVT::i32), Dest, CCReg);
3956: }
3957: 
3958: // Return true if Pos is CmpOp and Neg is the negative of CmpOp,
3959: // allowing Pos and Neg to be wider than CmpOp.
3960: static bool isAbsolute(SDValue CmpOp, SDValue Pos, SDValue Neg) {
3961:   return (Neg.getOpcode() == ISD::SUB &&
3962:           Neg.getOperand(0).getOpcode() == ISD::Constant &&
3963:           Neg.getConstantOperandVal(0) == 0 && Neg.getOperand(1) == Pos &&
3964:           (Pos == CmpOp || (Pos.getOpcode() == ISD::SIGN_EXTEND &&
3965:                             Pos.getOperand(0) == CmpOp)));
3966: }
3967: 
3968: // Return the absolute or negative absolute of Op; IsNegative decides which.
3969: static SDValue getAbsolute(SelectionDAG &DAG, const SDLoc &DL, SDValue Op,
3970:                            bool IsNegative) {
3971:   Op = DAG.getNode(ISD::ABS, DL, Op.getValueType(), Op);
3972:   if (IsNegative)
3973:     Op = DAG.getNode(ISD::SUB, DL, Op.getValueType(),
3974:                      DAG.getConstant(0, DL, Op.getValueType()), Op);
3975:   return Op;
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerSETCC`, `SystemZTargetLowering::lowerSTRICT_FSETCC`, `SystemZTargetLowering::lowerBR_CC`, `isAbsolute`, `getAbsolute`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerSETCC`, `SystemZTargetLowering::lowerSTRICT_FSETCC`, `SystemZTargetLowering::lowerBR_CC`, `isAbsolute`, `getAbsolute` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3976-4050 / 第 3976-4050 行
```cpp
3976: }
3977: 
3978: static SDValue getI128Select(SelectionDAG &DAG, const SDLoc &DL,
3979:                              Comparison C, SDValue TrueOp, SDValue FalseOp) {
3980:   EVT VT = MVT::i128;
3981:   unsigned Op;
3982: 
3983:   if (C.CCMask == SystemZ::CCMASK_CMP_NE ||
3984:       C.CCMask == SystemZ::CCMASK_CMP_GE ||
3985:       C.CCMask == SystemZ::CCMASK_CMP_LE) {
3986:     std::swap(TrueOp, FalseOp);
3987:     C.CCMask ^= C.CCValid;
3988:   }
3989:   if (C.CCMask == SystemZ::CCMASK_CMP_LT) {
3990:     std::swap(C.Op0, C.Op1);
3991:     C.CCMask = SystemZ::CCMASK_CMP_GT;
3992:   }
3993:   switch (C.CCMask) {
3994:   case SystemZ::CCMASK_CMP_EQ:
3995:     Op = SystemZISD::VICMPE;
3996:     break;
3997:   case SystemZ::CCMASK_CMP_GT:
3998:     if (C.ICmpType == SystemZICMP::UnsignedOnly)
3999:       Op = SystemZISD::VICMPHL;
4000:     else
4001:       Op = SystemZISD::VICMPH;
4002:     break;
4003:   default:
4004:     llvm_unreachable("Unhandled comparison");
4005:     break;
4006:   }
4007: 
4008:   SDValue Mask = DAG.getNode(Op, DL, VT, C.Op0, C.Op1);
4009:   TrueOp = DAG.getNode(ISD::AND, DL, VT, TrueOp, Mask);
4010:   FalseOp = DAG.getNode(ISD::AND, DL, VT, FalseOp, DAG.getNOT(DL, Mask, VT));
4011:   return DAG.getNode(ISD::OR, DL, VT, TrueOp, FalseOp);
4012: }
4013: 
4014: SDValue SystemZTargetLowering::lowerSELECT_CC(SDValue Op,
4015:                                               SelectionDAG &DAG) const {
4016:   SDValue CmpOp0   = Op.getOperand(0);
4017:   SDValue CmpOp1   = Op.getOperand(1);
4018:   SDValue TrueOp   = Op.getOperand(2);
4019:   SDValue FalseOp  = Op.getOperand(3);
4020:   ISD::CondCode CC = cast<CondCodeSDNode>(Op.getOperand(4))->get();
4021:   SDLoc DL(Op);
4022: 
4023:   // SELECT_CC involving f16 will not have the cmp-ops promoted by the
4024:   // legalizer, as it will be handled according to the type of the resulting
4025:   // value. Extend them here if needed.
4026:   if (CmpOp0.getSimpleValueType() == MVT::f16) {
4027:     CmpOp0 = DAG.getFPExtendOrRound(CmpOp0, SDLoc(CmpOp0), MVT::f32);
4028:     CmpOp1 = DAG.getFPExtendOrRound(CmpOp1, SDLoc(CmpOp1), MVT::f32);
4029:   }
4030: 
4031:   Comparison C(getCmp(DAG, CmpOp0, CmpOp1, CC, DL));
4032: 
4033:   // Check for absolute and negative-absolute selections, including those
4034:   // where the comparison value is sign-extended (for LPGFR and LNGFR).
4035:   // This check supplements the one in DAGCombiner.
4036:   if (C.Opcode == SystemZISD::ICMP && C.CCMask != SystemZ::CCMASK_CMP_EQ &&
4037:       C.CCMask != SystemZ::CCMASK_CMP_NE &&
4038:       C.Op1.getOpcode() == ISD::Constant &&
4039:       cast<ConstantSDNode>(C.Op1)->getValueSizeInBits(0) <= 64 &&
4040:       C.Op1->getAsZExtVal() == 0) {
4041:     if (isAbsolute(C.Op0, TrueOp, FalseOp))
4042:       return getAbsolute(DAG, DL, TrueOp, C.CCMask & SystemZ::CCMASK_CMP_LT);
4043:     if (isAbsolute(C.Op0, FalseOp, TrueOp))
4044:       return getAbsolute(DAG, DL, FalseOp, C.CCMask & SystemZ::CCMASK_CMP_GT);
4045:   }
4046: 
4047:   if (Subtarget.hasVectorEnhancements3() &&
4048:       C.Opcode == SystemZISD::ICMP &&
4049:       C.Op0.getValueType() == MVT::i128 &&
4050:       TrueOp.getValueType() == MVT::i128) {
```
- **EN**: The range implements or declares functions including `getI128Select`, `SystemZTargetLowering::lowerSELECT_CC`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getI128Select`, `SystemZTargetLowering::lowerSELECT_CC` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 4051-4125 / 第 4051-4125 行
```cpp
4051:     return getI128Select(DAG, DL, C, TrueOp, FalseOp);
4052:   }
4053: 
4054:   SDValue CCReg = emitCmp(DAG, DL, C);
4055:   SDValue Ops[] = {TrueOp, FalseOp,
4056:                    DAG.getTargetConstant(C.CCValid, DL, MVT::i32),
4057:                    DAG.getTargetConstant(C.CCMask, DL, MVT::i32), CCReg};
4058: 
4059:   return DAG.getNode(SystemZISD::SELECT_CCMASK, DL, Op.getValueType(), Ops);
4060: }
4061: 
4062: SDValue SystemZTargetLowering::lowerGlobalAddress(GlobalAddressSDNode *Node,
4063:                                                   SelectionDAG &DAG) const {
4064:   SDLoc DL(Node);
4065:   const GlobalValue *GV = Node->getGlobal();
4066:   int64_t Offset = Node->getOffset();
4067:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
4068:   CodeModel::Model CM = DAG.getTarget().getCodeModel();
4069: 
4070:   SDValue Result;
4071:   if (Subtarget.isPC32DBLSymbol(GV, CM)) {
4072:     if (isInt<32>(Offset)) {
4073:       // Assign anchors at 1<<12 byte boundaries.
4074:       uint64_t Anchor = Offset & ~uint64_t(0xfff);
4075:       Result = DAG.getTargetGlobalAddress(GV, DL, PtrVT, Anchor);
4076:       Result = DAG.getNode(SystemZISD::PCREL_WRAPPER, DL, PtrVT, Result);
4077: 
4078:       // The offset can be folded into the address if it is aligned to a
4079:       // halfword.
4080:       Offset -= Anchor;
4081:       if (Offset != 0 && (Offset & 1) == 0) {
4082:         SDValue Full =
4083:           DAG.getTargetGlobalAddress(GV, DL, PtrVT, Anchor + Offset);
4084:         Result = DAG.getNode(SystemZISD::PCREL_OFFSET, DL, PtrVT, Full, Result);
4085:         Offset = 0;
4086:       }
4087:     } else {
4088:       // Conservatively load a constant offset greater than 32 bits into a
4089:       // register below.
4090:       Result = DAG.getTargetGlobalAddress(GV, DL, PtrVT);
4091:       Result = DAG.getNode(SystemZISD::PCREL_WRAPPER, DL, PtrVT, Result);
4092:     }
4093:   } else if (Subtarget.isTargetELF()) {
4094:     Result = DAG.getTargetGlobalAddress(GV, DL, PtrVT, 0, SystemZII::MO_GOT);
4095:     Result = DAG.getNode(SystemZISD::PCREL_WRAPPER, DL, PtrVT, Result);
4096:     Result = DAG.getLoad(PtrVT, DL, DAG.getEntryNode(), Result,
4097:                          MachinePointerInfo::getGOT(DAG.getMachineFunction()));
4098:   } else if (Subtarget.isTargetzOS()) {
4099:     Result = getADAEntry(DAG, GV, DL, PtrVT);
4100:   } else
4101:     llvm_unreachable("Unexpected Subtarget");
4102: 
4103:   // If there was a non-zero offset that we didn't fold, create an explicit
4104:   // addition for it.
4105:   if (Offset != 0)
4106:     Result = DAG.getNode(ISD::ADD, DL, PtrVT, Result,
4107:                          DAG.getSignedConstant(Offset, DL, PtrVT));
4108: 
4109:   return Result;
4110: }
4111: 
4112: SDValue SystemZTargetLowering::lowerTLSGetOffset(GlobalAddressSDNode *Node,
4113:                                                  SelectionDAG &DAG,
4114:                                                  unsigned Opcode,
4115:                                                  SDValue GOTOffset) const {
4116:   SDLoc DL(Node);
4117:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
4118:   SDValue Chain = DAG.getEntryNode();
4119:   SDValue Glue;
4120: 
4121:   if (DAG.getMachineFunction().getFunction().getCallingConv() ==
4122:       CallingConv::GHC)
4123:     report_fatal_error("In GHC calling convention TLS is not supported");
4124: 
4125:   // __tls_get_offset takes the GOT offset in %r2 and the GOT in %r12.
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerGlobalAddress`, `SystemZTargetLowering::lowerTLSGetOffset`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerGlobalAddress`, `SystemZTargetLowering::lowerTLSGetOffset` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 4126-4200 / 第 4126-4200 行
```cpp
4126:   SDValue GOT = DAG.getGLOBAL_OFFSET_TABLE(PtrVT);
4127:   Chain = DAG.getCopyToReg(Chain, DL, SystemZ::R12D, GOT, Glue);
4128:   Glue = Chain.getValue(1);
4129:   Chain = DAG.getCopyToReg(Chain, DL, SystemZ::R2D, GOTOffset, Glue);
4130:   Glue = Chain.getValue(1);
4131: 
4132:   // The first call operand is the chain and the second is the TLS symbol.
4133:   SmallVector<SDValue, 8> Ops;
4134:   Ops.push_back(Chain);
4135:   Ops.push_back(DAG.getTargetGlobalAddress(Node->getGlobal(), DL,
4136:                                            Node->getValueType(0),
4137:                                            0, 0));
4138: 
4139:   // Add argument registers to the end of the list so that they are
4140:   // known live into the call.
4141:   Ops.push_back(DAG.getRegister(SystemZ::R2D, PtrVT));
4142:   Ops.push_back(DAG.getRegister(SystemZ::R12D, PtrVT));
4143: 
4144:   // Add a register mask operand representing the call-preserved registers.
4145:   const TargetRegisterInfo *TRI = Subtarget.getRegisterInfo();
4146:   const uint32_t *Mask =
4147:       TRI->getCallPreservedMask(DAG.getMachineFunction(), CallingConv::C);
4148:   assert(Mask && "Missing call preserved mask for calling convention");
4149:   Ops.push_back(DAG.getRegisterMask(Mask));
4150: 
4151:   // Glue the call to the argument copies.
4152:   Ops.push_back(Glue);
4153: 
4154:   // Emit the call.
4155:   SDVTList NodeTys = DAG.getVTList(MVT::Other, MVT::Glue);
4156:   Chain = DAG.getNode(Opcode, DL, NodeTys, Ops);
4157:   Glue = Chain.getValue(1);
4158: 
4159:   // Copy the return value from %r2.
4160:   return DAG.getCopyFromReg(Chain, DL, SystemZ::R2D, PtrVT, Glue);
4161: }
4162: 
4163: SDValue SystemZTargetLowering::lowerThreadPointer(const SDLoc &DL,
4164:                                                   SelectionDAG &DAG) const {
4165:   SDValue Chain = DAG.getEntryNode();
4166:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
4167: 
4168:   // The high part of the thread pointer is in access register 0.
4169:   SDValue TPHi = DAG.getCopyFromReg(Chain, DL, SystemZ::A0, MVT::i32);
4170:   TPHi = DAG.getNode(ISD::ANY_EXTEND, DL, PtrVT, TPHi);
4171: 
4172:   // The low part of the thread pointer is in access register 1.
4173:   SDValue TPLo = DAG.getCopyFromReg(Chain, DL, SystemZ::A1, MVT::i32);
4174:   TPLo = DAG.getNode(ISD::ZERO_EXTEND, DL, PtrVT, TPLo);
4175: 
4176:   // Merge them into a single 64-bit address.
4177:   SDValue TPHiShifted = DAG.getNode(ISD::SHL, DL, PtrVT, TPHi,
4178:                                     DAG.getConstant(32, DL, PtrVT));
4179:   return DAG.getNode(ISD::OR, DL, PtrVT, TPHiShifted, TPLo);
4180: }
4181: 
4182: SDValue SystemZTargetLowering::lowerGlobalTLSAddress(GlobalAddressSDNode *Node,
4183:                                                      SelectionDAG &DAG) const {
4184:   if (DAG.getTarget().useEmulatedTLS())
4185:     return LowerToTLSEmulatedModel(Node, DAG);
4186:   SDLoc DL(Node);
4187:   const GlobalValue *GV = Node->getGlobal();
4188:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
4189:   TLSModel::Model model = DAG.getTarget().getTLSModel(GV);
4190: 
4191:   if (DAG.getMachineFunction().getFunction().getCallingConv() ==
4192:       CallingConv::GHC)
4193:     report_fatal_error("In GHC calling convention TLS is not supported");
4194: 
4195:   SDValue TP = lowerThreadPointer(DL, DAG);
4196: 
4197:   // Get the offset of GA from the thread pointer, based on the TLS model.
4198:   SDValue Offset;
4199:   switch (model) {
4200:     case TLSModel::GeneralDynamic: {
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerThreadPointer`, `SystemZTargetLowering::lowerGlobalTLSAddress`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerThreadPointer`, `SystemZTargetLowering::lowerGlobalTLSAddress` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 4201-4275 / 第 4201-4275 行
```cpp
4201:       // Load the GOT offset of the tls_index (module ID / per-symbol offset).
4202:       SystemZConstantPoolValue *CPV =
4203:         SystemZConstantPoolValue::Create(GV, SystemZCP::TLSGD);
4204: 
4205:       Offset = DAG.getConstantPool(CPV, PtrVT, Align(8));
4206:       Offset = DAG.getLoad(
4207:           PtrVT, DL, DAG.getEntryNode(), Offset,
4208:           MachinePointerInfo::getConstantPool(DAG.getMachineFunction()));
4209: 
4210:       // Call __tls_get_offset to retrieve the offset.
4211:       Offset = lowerTLSGetOffset(Node, DAG, SystemZISD::TLS_GDCALL, Offset);
4212:       break;
4213:     }
4214: 
4215:     case TLSModel::LocalDynamic: {
4216:       // Load the GOT offset of the module ID.
4217:       SystemZConstantPoolValue *CPV =
4218:         SystemZConstantPoolValue::Create(GV, SystemZCP::TLSLDM);
4219: 
4220:       Offset = DAG.getConstantPool(CPV, PtrVT, Align(8));
4221:       Offset = DAG.getLoad(
4222:           PtrVT, DL, DAG.getEntryNode(), Offset,
4223:           MachinePointerInfo::getConstantPool(DAG.getMachineFunction()));
4224: 
4225:       // Call __tls_get_offset to retrieve the module base offset.
4226:       Offset = lowerTLSGetOffset(Node, DAG, SystemZISD::TLS_LDCALL, Offset);
4227: 
4228:       // Note: The SystemZLDCleanupPass will remove redundant computations
4229:       // of the module base offset.  Count total number of local-dynamic
4230:       // accesses to trigger execution of that pass.
4231:       SystemZMachineFunctionInfo* MFI =
4232:         DAG.getMachineFunction().getInfo<SystemZMachineFunctionInfo>();
4233:       MFI->incNumLocalDynamicTLSAccesses();
4234: 
4235:       // Add the per-symbol offset.
4236:       CPV = SystemZConstantPoolValue::Create(GV, SystemZCP::DTPOFF);
4237: 
4238:       SDValue DTPOffset = DAG.getConstantPool(CPV, PtrVT, Align(8));
4239:       DTPOffset = DAG.getLoad(
4240:           PtrVT, DL, DAG.getEntryNode(), DTPOffset,
4241:           MachinePointerInfo::getConstantPool(DAG.getMachineFunction()));
4242: 
4243:       Offset = DAG.getNode(ISD::ADD, DL, PtrVT, Offset, DTPOffset);
4244:       break;
4245:     }
4246: 
4247:     case TLSModel::InitialExec: {
4248:       // Load the offset from the GOT.
4249:       Offset = DAG.getTargetGlobalAddress(GV, DL, PtrVT, 0,
4250:                                           SystemZII::MO_INDNTPOFF);
4251:       Offset = DAG.getNode(SystemZISD::PCREL_WRAPPER, DL, PtrVT, Offset);
4252:       Offset =
4253:           DAG.getLoad(PtrVT, DL, DAG.getEntryNode(), Offset,
4254:                       MachinePointerInfo::getGOT(DAG.getMachineFunction()));
4255:       break;
4256:     }
4257: 
4258:     case TLSModel::LocalExec: {
4259:       // Force the offset into the constant pool and load it from there.
4260:       SystemZConstantPoolValue *CPV =
4261:         SystemZConstantPoolValue::Create(GV, SystemZCP::NTPOFF);
4262: 
4263:       Offset = DAG.getConstantPool(CPV, PtrVT, Align(8));
4264:       Offset = DAG.getLoad(
4265:           PtrVT, DL, DAG.getEntryNode(), Offset,
4266:           MachinePointerInfo::getConstantPool(DAG.getMachineFunction()));
4267:       break;
4268:     }
4269:   }
4270: 
4271:   // Add the base and offset together.
4272:   return DAG.getNode(ISD::ADD, DL, PtrVT, TP, Offset);
4273: }
4274: 
4275: SDValue SystemZTargetLowering::lowerBlockAddress(BlockAddressSDNode *Node,
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 4276-4350 / 第 4276-4350 行
```cpp
4276:                                                  SelectionDAG &DAG) const {
4277:   SDLoc DL(Node);
4278:   const BlockAddress *BA = Node->getBlockAddress();
4279:   int64_t Offset = Node->getOffset();
4280:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
4281: 
4282:   SDValue Result = DAG.getTargetBlockAddress(BA, PtrVT, Offset);
4283:   Result = DAG.getNode(SystemZISD::PCREL_WRAPPER, DL, PtrVT, Result);
4284:   return Result;
4285: }
4286: 
4287: SDValue SystemZTargetLowering::lowerJumpTable(JumpTableSDNode *JT,
4288:                                               SelectionDAG &DAG) const {
4289:   SDLoc DL(JT);
4290:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
4291:   SDValue Result = DAG.getTargetJumpTable(JT->getIndex(), PtrVT);
4292: 
4293:   // Use LARL to load the address of the table.
4294:   return DAG.getNode(SystemZISD::PCREL_WRAPPER, DL, PtrVT, Result);
4295: }
4296: 
4297: SDValue SystemZTargetLowering::lowerConstantPool(ConstantPoolSDNode *CP,
4298:                                                  SelectionDAG &DAG) const {
4299:   SDLoc DL(CP);
4300:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
4301: 
4302:   SDValue Result;
4303:   if (CP->isMachineConstantPoolEntry())
4304:     Result =
4305:         DAG.getTargetConstantPool(CP->getMachineCPVal(), PtrVT, CP->getAlign());
4306:   else
4307:     Result = DAG.getTargetConstantPool(CP->getConstVal(), PtrVT, CP->getAlign(),
4308:                                        CP->getOffset());
4309: 
4310:   // Use LARL to load the address of the constant pool entry.
4311:   return DAG.getNode(SystemZISD::PCREL_WRAPPER, DL, PtrVT, Result);
4312: }
4313: 
4314: SDValue SystemZTargetLowering::lowerFRAMEADDR(SDValue Op,
4315:                                               SelectionDAG &DAG) const {
4316:   auto *TFL = Subtarget.getFrameLowering<SystemZFrameLowering>();
4317:   MachineFunction &MF = DAG.getMachineFunction();
4318:   MachineFrameInfo &MFI = MF.getFrameInfo();
4319:   MFI.setFrameAddressIsTaken(true);
4320: 
4321:   SDLoc DL(Op);
4322:   unsigned Depth = Op.getConstantOperandVal(0);
4323:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
4324: 
4325:   // By definition, the frame address is the address of the back chain.  (In
4326:   // the case of packed stack without backchain, return the address where the
4327:   // backchain would have been stored. This will either be an unused space or
4328:   // contain a saved register).
4329:   int BackChainIdx = TFL->getOrCreateFramePointerSaveIndex(MF);
4330:   SDValue BackChain = DAG.getFrameIndex(BackChainIdx, PtrVT);
4331: 
4332:   if (Depth > 0) {
4333:     // FIXME The frontend should detect this case.
4334:     if (!MF.getSubtarget<SystemZSubtarget>().hasBackChain())
4335:       report_fatal_error("Unsupported stack frame traversal count");
4336: 
4337:     SDValue Offset = DAG.getConstant(TFL->getBackchainOffset(MF), DL, PtrVT);
4338:     while (Depth--) {
4339:       BackChain = DAG.getLoad(PtrVT, DL, DAG.getEntryNode(), BackChain,
4340:                               MachinePointerInfo());
4341:       BackChain = DAG.getNode(ISD::ADD, DL, PtrVT, BackChain, Offset);
4342:     }
4343:   }
4344: 
4345:   return BackChain;
4346: }
4347: 
4348: SDValue SystemZTargetLowering::lowerRETURNADDR(SDValue Op,
4349:                                                SelectionDAG &DAG) const {
4350:   MachineFunction &MF = DAG.getMachineFunction();
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerJumpTable`, `SystemZTargetLowering::lowerConstantPool`, `SystemZTargetLowering::lowerFRAMEADDR`, `SystemZTargetLowering::lowerRETURNADDR`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerJumpTable`, `SystemZTargetLowering::lowerConstantPool`, `SystemZTargetLowering::lowerFRAMEADDR`, `SystemZTargetLowering::lowerRETURNADDR` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 4351-4425 / 第 4351-4425 行
```cpp
4351:   MachineFrameInfo &MFI = MF.getFrameInfo();
4352:   MFI.setReturnAddressIsTaken(true);
4353: 
4354:   SDLoc DL(Op);
4355:   unsigned Depth = Op.getConstantOperandVal(0);
4356:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
4357: 
4358:   if (Depth > 0) {
4359:     // FIXME The frontend should detect this case.
4360:     if (!MF.getSubtarget<SystemZSubtarget>().hasBackChain())
4361:       report_fatal_error("Unsupported stack frame traversal count");
4362: 
4363:     SDValue FrameAddr = lowerFRAMEADDR(Op, DAG);
4364:     const auto *TFL = Subtarget.getFrameLowering<SystemZFrameLowering>();
4365:     int Offset = TFL->getReturnAddressOffset(MF);
4366:     SDValue Ptr = DAG.getNode(ISD::ADD, DL, PtrVT, FrameAddr,
4367:                               DAG.getSignedConstant(Offset, DL, PtrVT));
4368:     return DAG.getLoad(PtrVT, DL, DAG.getEntryNode(), Ptr,
4369:                        MachinePointerInfo());
4370:   }
4371: 
4372:   // Return R14D (Elf) / R7D (XPLINK), which has the return address. Mark it an
4373:   // implicit live-in.
4374:   SystemZCallingConventionRegisters *CCR = Subtarget.getSpecialRegisters();
4375:   Register LinkReg = MF.addLiveIn(CCR->getReturnFunctionAddressRegister(),
4376:                                   &SystemZ::GR64BitRegClass);
4377:   return DAG.getCopyFromReg(DAG.getEntryNode(), DL, LinkReg, PtrVT);
4378: }
4379: 
4380: SDValue SystemZTargetLowering::lowerBITCAST(SDValue Op,
4381:                                             SelectionDAG &DAG) const {
4382:   SDLoc DL(Op);
4383:   SDValue In = Op.getOperand(0);
4384:   EVT InVT = In.getValueType();
4385:   EVT ResVT = Op.getValueType();
4386: 
4387:   // Convert loads directly.  This is normally done by DAGCombiner,
4388:   // but we need this case for bitcasts that are created during lowering
4389:   // and which are then lowered themselves.
4390:   if (auto *LoadN = dyn_cast<LoadSDNode>(In))
4391:     if (ISD::isNormalLoad(LoadN)) {
4392:       SDValue NewLoad = DAG.getLoad(ResVT, DL, LoadN->getChain(),
4393:                                     LoadN->getBasePtr(), LoadN->getMemOperand());
4394:       // Update the chain uses.
4395:       DAG.ReplaceAllUsesOfValueWith(SDValue(LoadN, 1), NewLoad.getValue(1));
4396:       return NewLoad;
4397:     }
4398: 
4399:   if (InVT == MVT::i32 && ResVT == MVT::f32) {
4400:     SDValue In64;
4401:     if (Subtarget.hasHighWord()) {
4402:       SDNode *U64 = DAG.getMachineNode(TargetOpcode::IMPLICIT_DEF, DL,
4403:                                        MVT::i64);
4404:       In64 = DAG.getTargetInsertSubreg(SystemZ::subreg_h32, DL,
4405:                                        MVT::i64, SDValue(U64, 0), In);
4406:     } else {
4407:       In64 = DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, In);
4408:       In64 = DAG.getNode(ISD::SHL, DL, MVT::i64, In64,
4409:                          DAG.getConstant(32, DL, MVT::i64));
4410:     }
4411:     SDValue Out64 = DAG.getNode(ISD::BITCAST, DL, MVT::f64, In64);
4412:     return DAG.getTargetExtractSubreg(SystemZ::subreg_h32,
4413:                                       DL, MVT::f32, Out64);
4414:   }
4415:   if (InVT == MVT::f32 && ResVT == MVT::i32) {
4416:     SDNode *U64 = DAG.getMachineNode(TargetOpcode::IMPLICIT_DEF, DL, MVT::f64);
4417:     SDValue In64 = DAG.getTargetInsertSubreg(SystemZ::subreg_h32, DL,
4418:                                              MVT::f64, SDValue(U64, 0), In);
4419:     SDValue Out64 = DAG.getNode(ISD::BITCAST, DL, MVT::i64, In64);
4420:     if (Subtarget.hasHighWord())
4421:       return DAG.getTargetExtractSubreg(SystemZ::subreg_h32, DL,
4422:                                         MVT::i32, Out64);
4423:     SDValue Shift = DAG.getNode(ISD::SRL, DL, MVT::i64, Out64,
4424:                                 DAG.getConstant(32, DL, MVT::i64));
4425:     return DAG.getNode(ISD::TRUNCATE, DL, MVT::i32, Shift);
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerBITCAST`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerBITCAST` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 4426-4500 / 第 4426-4500 行
```cpp
4426:   }
4427:   llvm_unreachable("Unexpected bitcast combination");
4428: }
4429: 
4430: SDValue SystemZTargetLowering::lowerVASTART(SDValue Op,
4431:                                             SelectionDAG &DAG) const {
4432: 
4433:   if (Subtarget.isTargetXPLINK64())
4434:     return lowerVASTART_XPLINK(Op, DAG);
4435:   else
4436:     return lowerVASTART_ELF(Op, DAG);
4437: }
4438: 
4439: SDValue SystemZTargetLowering::lowerVASTART_XPLINK(SDValue Op,
4440:                                                    SelectionDAG &DAG) const {
4441:   MachineFunction &MF = DAG.getMachineFunction();
4442:   SystemZMachineFunctionInfo *FuncInfo =
4443:       MF.getInfo<SystemZMachineFunctionInfo>();
4444: 
4445:   SDLoc DL(Op);
4446: 
4447:   // vastart just stores the address of the VarArgsFrameIndex slot into the
4448:   // memory location argument.
4449:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
4450:   SDValue FR = DAG.getFrameIndex(FuncInfo->getVarArgsFrameIndex(), PtrVT);
4451:   const Value *SV = cast<SrcValueSDNode>(Op.getOperand(2))->getValue();
4452:   return DAG.getStore(Op.getOperand(0), DL, FR, Op.getOperand(1),
4453:                       MachinePointerInfo(SV));
4454: }
4455: 
4456: SDValue SystemZTargetLowering::lowerVASTART_ELF(SDValue Op,
4457:                                                 SelectionDAG &DAG) const {
4458:   MachineFunction &MF = DAG.getMachineFunction();
4459:   SystemZMachineFunctionInfo *FuncInfo =
4460:     MF.getInfo<SystemZMachineFunctionInfo>();
4461:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
4462: 
4463:   SDValue Chain   = Op.getOperand(0);
4464:   SDValue Addr    = Op.getOperand(1);
4465:   const Value *SV = cast<SrcValueSDNode>(Op.getOperand(2))->getValue();
4466:   SDLoc DL(Op);
4467: 
4468:   // The initial values of each field.
4469:   const unsigned NumFields = 4;
4470:   SDValue Fields[NumFields] = {
4471:     DAG.getConstant(FuncInfo->getVarArgsFirstGPR(), DL, PtrVT),
4472:     DAG.getConstant(FuncInfo->getVarArgsFirstFPR(), DL, PtrVT),
4473:     DAG.getFrameIndex(FuncInfo->getVarArgsFrameIndex(), PtrVT),
4474:     DAG.getFrameIndex(FuncInfo->getRegSaveFrameIndex(), PtrVT)
4475:   };
4476: 
4477:   // Store each field into its respective slot.
4478:   SDValue MemOps[NumFields];
4479:   unsigned Offset = 0;
4480:   for (unsigned I = 0; I < NumFields; ++I) {
4481:     SDValue FieldAddr = Addr;
4482:     if (Offset != 0)
4483:       FieldAddr = DAG.getNode(ISD::ADD, DL, PtrVT, FieldAddr,
4484:                               DAG.getIntPtrConstant(Offset, DL));
4485:     MemOps[I] = DAG.getStore(Chain, DL, Fields[I], FieldAddr,
4486:                              MachinePointerInfo(SV, Offset));
4487:     Offset += 8;
4488:   }
4489:   return DAG.getNode(ISD::TokenFactor, DL, MVT::Other, MemOps);
4490: }
4491: 
4492: SDValue SystemZTargetLowering::lowerVACOPY(SDValue Op,
4493:                                            SelectionDAG &DAG) const {
4494:   SDValue Chain      = Op.getOperand(0);
4495:   SDValue DstPtr     = Op.getOperand(1);
4496:   SDValue SrcPtr     = Op.getOperand(2);
4497:   const Value *DstSV = cast<SrcValueSDNode>(Op.getOperand(3))->getValue();
4498:   const Value *SrcSV = cast<SrcValueSDNode>(Op.getOperand(4))->getValue();
4499:   SDLoc DL(Op);
4500: 
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerVASTART`, `SystemZTargetLowering::lowerVASTART_XPLINK`, `SystemZTargetLowering::lowerVASTART_ELF`, `SystemZTargetLowering::lowerVACOPY`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerVASTART`, `SystemZTargetLowering::lowerVASTART_XPLINK`, `SystemZTargetLowering::lowerVASTART_ELF`, `SystemZTargetLowering::lowerVACOPY` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 4501-4575 / 第 4501-4575 行
```cpp
4501:   uint32_t Sz =
4502:       Subtarget.isTargetXPLINK64() ? getTargetMachine().getPointerSize(0) : 32;
4503:   return DAG.getMemcpy(Chain, DL, DstPtr, SrcPtr, DAG.getIntPtrConstant(Sz, DL),
4504:                        Align(8), /*isVolatile*/ false, /*AlwaysInline*/ false,
4505:                        /*CI=*/nullptr, std::nullopt, MachinePointerInfo(DstSV),
4506:                        MachinePointerInfo(SrcSV));
4507: }
4508: 
4509: SDValue
4510: SystemZTargetLowering::lowerDYNAMIC_STACKALLOC(SDValue Op,
4511:                                                SelectionDAG &DAG) const {
4512:   if (Subtarget.isTargetXPLINK64())
4513:     return lowerDYNAMIC_STACKALLOC_XPLINK(Op, DAG);
4514:   else
4515:     return lowerDYNAMIC_STACKALLOC_ELF(Op, DAG);
4516: }
4517: 
4518: SDValue
4519: SystemZTargetLowering::lowerDYNAMIC_STACKALLOC_XPLINK(SDValue Op,
4520:                                                       SelectionDAG &DAG) const {
4521:   const TargetFrameLowering *TFI = Subtarget.getFrameLowering();
4522:   MachineFunction &MF = DAG.getMachineFunction();
4523:   bool RealignOpt = !MF.getFunction().hasFnAttribute("no-realign-stack");
4524:   SDValue Chain = Op.getOperand(0);
4525:   SDValue Size = Op.getOperand(1);
4526:   SDValue Align = Op.getOperand(2);
4527:   SDLoc DL(Op);
4528: 
4529:   // If user has set the no alignment function attribute, ignore
4530:   // alloca alignments.
4531:   uint64_t AlignVal = (RealignOpt ? Align->getAsZExtVal() : 0);
4532: 
4533:   uint64_t StackAlign = TFI->getStackAlignment();
4534:   uint64_t RequiredAlign = std::max(AlignVal, StackAlign);
4535:   uint64_t ExtraAlignSpace = RequiredAlign - StackAlign;
4536: 
4537:   SDValue NeededSpace = Size;
4538: 
4539:   // Add extra space for alignment if needed.
4540:   EVT PtrVT = getPointerTy(MF.getDataLayout());
4541:   if (ExtraAlignSpace)
4542:     NeededSpace = DAG.getNode(ISD::ADD, DL, PtrVT, NeededSpace,
4543:                               DAG.getConstant(ExtraAlignSpace, DL, PtrVT));
4544: 
4545:   bool IsSigned = false;
4546:   bool DoesNotReturn = false;
4547:   bool IsReturnValueUsed = false;
4548:   EVT VT = Op.getValueType();
4549:   SDValue AllocaCall =
4550:       makeExternalCall(Chain, DAG, "@@ALCAXP", VT, ArrayRef(NeededSpace),
4551:                        CallingConv::C, IsSigned, DL, DoesNotReturn,
4552:                        IsReturnValueUsed)
4553:           .first;
4554: 
4555:   // Perform a CopyFromReg from %GPR4 (stack pointer register). Chain and Glue
4556:   // to end of call in order to ensure it isn't broken up from the call
4557:   // sequence.
4558:   auto &Regs = Subtarget.getSpecialRegisters<SystemZXPLINK64Registers>();
4559:   Register SPReg = Regs.getStackPointerRegister();
4560:   Chain = AllocaCall.getValue(1);
4561:   SDValue Glue = AllocaCall.getValue(2);
4562:   SDValue NewSPRegNode = DAG.getCopyFromReg(Chain, DL, SPReg, PtrVT, Glue);
4563:   Chain = NewSPRegNode.getValue(1);
4564: 
4565:   MVT PtrMVT = getPointerMemTy(MF.getDataLayout());
4566:   SDValue ArgAdjust = DAG.getNode(SystemZISD::ADJDYNALLOC, DL, PtrMVT);
4567:   SDValue Result = DAG.getNode(ISD::ADD, DL, PtrMVT, NewSPRegNode, ArgAdjust);
4568: 
4569:   // Dynamically realign if needed.
4570:   if (ExtraAlignSpace) {
4571:     Result = DAG.getNode(ISD::ADD, DL, PtrVT, Result,
4572:                          DAG.getConstant(ExtraAlignSpace, DL, PtrVT));
4573:     Result = DAG.getNode(ISD::AND, DL, PtrVT, Result,
4574:                          DAG.getConstant(~(RequiredAlign - 1), DL, PtrVT));
4575:   }
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerDYNAMIC_STACKALLOC`, `SystemZTargetLowering::lowerDYNAMIC_STACKALLOC_XPLINK`, `makeExternalCall`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerDYNAMIC_STACKALLOC`, `SystemZTargetLowering::lowerDYNAMIC_STACKALLOC_XPLINK`, `makeExternalCall` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 4576-4650 / 第 4576-4650 行
```cpp
4576: 
4577:   SDValue Ops[2] = {Result, Chain};
4578:   return DAG.getMergeValues(Ops, DL);
4579: }
4580: 
4581: SDValue
4582: SystemZTargetLowering::lowerDYNAMIC_STACKALLOC_ELF(SDValue Op,
4583:                                                    SelectionDAG &DAG) const {
4584:   const TargetFrameLowering *TFI = Subtarget.getFrameLowering();
4585:   MachineFunction &MF = DAG.getMachineFunction();
4586:   bool RealignOpt = !MF.getFunction().hasFnAttribute("no-realign-stack");
4587:   bool StoreBackchain = MF.getSubtarget<SystemZSubtarget>().hasBackChain();
4588: 
4589:   SDValue Chain = Op.getOperand(0);
4590:   SDValue Size  = Op.getOperand(1);
4591:   SDValue Align = Op.getOperand(2);
4592:   SDLoc DL(Op);
4593: 
4594:   // If user has set the no alignment function attribute, ignore
4595:   // alloca alignments.
4596:   uint64_t AlignVal = (RealignOpt ? Align->getAsZExtVal() : 0);
4597: 
4598:   uint64_t StackAlign = TFI->getStackAlignment();
4599:   uint64_t RequiredAlign = std::max(AlignVal, StackAlign);
4600:   uint64_t ExtraAlignSpace = RequiredAlign - StackAlign;
4601: 
4602:   Register SPReg = getStackPointerRegisterToSaveRestore();
4603:   SDValue NeededSpace = Size;
4604: 
4605:   // Get a reference to the stack pointer.
4606:   SDValue OldSP = DAG.getCopyFromReg(Chain, DL, SPReg, MVT::i64);
4607: 
4608:   // If we need a backchain, save it now.
4609:   SDValue Backchain;
4610:   if (StoreBackchain)
4611:     Backchain = DAG.getLoad(MVT::i64, DL, Chain, getBackchainAddress(OldSP, DAG),
4612:                             MachinePointerInfo());
4613: 
4614:   // Add extra space for alignment if needed.
4615:   if (ExtraAlignSpace)
4616:     NeededSpace = DAG.getNode(ISD::ADD, DL, MVT::i64, NeededSpace,
4617:                               DAG.getConstant(ExtraAlignSpace, DL, MVT::i64));
4618: 
4619:   // Get the new stack pointer value.
4620:   SDValue NewSP;
4621:   if (hasInlineStackProbe(MF)) {
4622:     NewSP = DAG.getNode(SystemZISD::PROBED_ALLOCA, DL,
4623:                 DAG.getVTList(MVT::i64, MVT::Other), Chain, OldSP, NeededSpace);
4624:     Chain = NewSP.getValue(1);
4625:   }
4626:   else {
4627:     NewSP = DAG.getNode(ISD::SUB, DL, MVT::i64, OldSP, NeededSpace);
4628:     // Copy the new stack pointer back.
4629:     Chain = DAG.getCopyToReg(Chain, DL, SPReg, NewSP);
4630:   }
4631: 
4632:   // The allocated data lives above the 160 bytes allocated for the standard
4633:   // frame, plus any outgoing stack arguments.  We don't know how much that
4634:   // amounts to yet, so emit a special ADJDYNALLOC placeholder.
4635:   SDValue ArgAdjust = DAG.getNode(SystemZISD::ADJDYNALLOC, DL, MVT::i64);
4636:   SDValue Result = DAG.getNode(ISD::ADD, DL, MVT::i64, NewSP, ArgAdjust);
4637: 
4638:   // Dynamically realign if needed.
4639:   if (RequiredAlign > StackAlign) {
4640:     Result =
4641:       DAG.getNode(ISD::ADD, DL, MVT::i64, Result,
4642:                   DAG.getConstant(ExtraAlignSpace, DL, MVT::i64));
4643:     Result =
4644:       DAG.getNode(ISD::AND, DL, MVT::i64, Result,
4645:                   DAG.getConstant(~(RequiredAlign - 1), DL, MVT::i64));
4646:   }
4647: 
4648:   if (StoreBackchain)
4649:     Chain = DAG.getStore(Chain, DL, Backchain, getBackchainAddress(NewSP, DAG),
4650:                          MachinePointerInfo());
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerDYNAMIC_STACKALLOC_ELF`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerDYNAMIC_STACKALLOC_ELF` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 4651-4725 / 第 4651-4725 行
```cpp
4651: 
4652:   SDValue Ops[2] = { Result, Chain };
4653:   return DAG.getMergeValues(Ops, DL);
4654: }
4655: 
4656: SDValue SystemZTargetLowering::lowerGET_DYNAMIC_AREA_OFFSET(
4657:     SDValue Op, SelectionDAG &DAG) const {
4658:   SDLoc DL(Op);
4659: 
4660:   return DAG.getNode(SystemZISD::ADJDYNALLOC, DL, MVT::i64);
4661: }
4662: 
4663: SDValue SystemZTargetLowering::lowerMULH(SDValue Op,
4664:                                          SelectionDAG &DAG,
4665:                                          unsigned Opcode) const {
4666:   EVT VT = Op.getValueType();
4667:   SDLoc DL(Op);
4668:   SDValue Even, Odd;
4669: 
4670:   // This custom expander is only used on z17 and later for 64-bit types.
4671:   assert(!is32Bit(VT));
4672:   assert(Subtarget.hasMiscellaneousExtensions2());
4673: 
4674:   // SystemZISD::xMUL_LOHI returns the low result in the odd register and
4675:   // the high result in the even register.  Return the latter.
4676:   lowerGR128Binary(DAG, DL, VT, Opcode,
4677:                    Op.getOperand(0), Op.getOperand(1), Even, Odd);
4678:   return Even;
4679: }
4680: 
4681: SDValue SystemZTargetLowering::lowerSMUL_LOHI(SDValue Op,
4682:                                               SelectionDAG &DAG) const {
4683:   EVT VT = Op.getValueType();
4684:   SDLoc DL(Op);
4685:   SDValue Ops[2];
4686:   if (is32Bit(VT))
4687:     // Just do a normal 64-bit multiplication and extract the results.
4688:     // We define this so that it can be used for constant division.
4689:     lowerMUL_LOHI32(DAG, DL, ISD::SIGN_EXTEND, Op.getOperand(0),
4690:                     Op.getOperand(1), Ops[1], Ops[0]);
4691:   else if (Subtarget.hasMiscellaneousExtensions2())
4692:     // SystemZISD::SMUL_LOHI returns the low result in the odd register and
4693:     // the high result in the even register.  ISD::SMUL_LOHI is defined to
4694:     // return the low half first, so the results are in reverse order.
4695:     lowerGR128Binary(DAG, DL, VT, SystemZISD::SMUL_LOHI,
4696:                      Op.getOperand(0), Op.getOperand(1), Ops[1], Ops[0]);
4697:   else {
4698:     // Do a full 128-bit multiplication based on SystemZISD::UMUL_LOHI:
4699:     //
4700:     //   (ll * rl) + ((lh * rl) << 64) + ((ll * rh) << 64)
4701:     //
4702:     // but using the fact that the upper halves are either all zeros
4703:     // or all ones:
4704:     //
4705:     //   (ll * rl) - ((lh & rl) << 64) - ((ll & rh) << 64)
4706:     //
4707:     // and grouping the right terms together since they are quicker than the
4708:     // multiplication:
4709:     //
4710:     //   (ll * rl) - (((lh & rl) + (ll & rh)) << 64)
4711:     SDValue C63 = DAG.getConstant(63, DL, MVT::i64);
4712:     SDValue LL = Op.getOperand(0);
4713:     SDValue RL = Op.getOperand(1);
4714:     SDValue LH = DAG.getNode(ISD::SRA, DL, VT, LL, C63);
4715:     SDValue RH = DAG.getNode(ISD::SRA, DL, VT, RL, C63);
4716:     // SystemZISD::UMUL_LOHI returns the low result in the odd register and
4717:     // the high result in the even register.  ISD::SMUL_LOHI is defined to
4718:     // return the low half first, so the results are in reverse order.
4719:     lowerGR128Binary(DAG, DL, VT, SystemZISD::UMUL_LOHI,
4720:                      LL, RL, Ops[1], Ops[0]);
4721:     SDValue NegLLTimesRH = DAG.getNode(ISD::AND, DL, VT, LL, RH);
4722:     SDValue NegLHTimesRL = DAG.getNode(ISD::AND, DL, VT, LH, RL);
4723:     SDValue NegSum = DAG.getNode(ISD::ADD, DL, VT, NegLLTimesRH, NegLHTimesRL);
4724:     Ops[1] = DAG.getNode(ISD::SUB, DL, VT, Ops[1], NegSum);
4725:   }
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerGET_DYNAMIC_AREA_OFFSET`, `SystemZTargetLowering::lowerMULH`, `SystemZTargetLowering::lowerSMUL_LOHI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerGET_DYNAMIC_AREA_OFFSET`, `SystemZTargetLowering::lowerMULH`, `SystemZTargetLowering::lowerSMUL_LOHI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 4726-4800 / 第 4726-4800 行
```cpp
4726:   return DAG.getMergeValues(Ops, DL);
4727: }
4728: 
4729: SDValue SystemZTargetLowering::lowerUMUL_LOHI(SDValue Op,
4730:                                               SelectionDAG &DAG) const {
4731:   EVT VT = Op.getValueType();
4732:   SDLoc DL(Op);
4733:   SDValue Ops[2];
4734:   if (is32Bit(VT))
4735:     // Just do a normal 64-bit multiplication and extract the results.
4736:     // We define this so that it can be used for constant division.
4737:     lowerMUL_LOHI32(DAG, DL, ISD::ZERO_EXTEND, Op.getOperand(0),
4738:                     Op.getOperand(1), Ops[1], Ops[0]);
4739:   else
4740:     // SystemZISD::UMUL_LOHI returns the low result in the odd register and
4741:     // the high result in the even register.  ISD::UMUL_LOHI is defined to
4742:     // return the low half first, so the results are in reverse order.
4743:     lowerGR128Binary(DAG, DL, VT, SystemZISD::UMUL_LOHI,
4744:                      Op.getOperand(0), Op.getOperand(1), Ops[1], Ops[0]);
4745:   return DAG.getMergeValues(Ops, DL);
4746: }
4747: 
4748: SDValue SystemZTargetLowering::lowerSDIVREM(SDValue Op,
4749:                                             SelectionDAG &DAG) const {
4750:   SDValue Op0 = Op.getOperand(0);
4751:   SDValue Op1 = Op.getOperand(1);
4752:   EVT VT = Op.getValueType();
4753:   SDLoc DL(Op);
4754: 
4755:   // We use DSGF for 32-bit division.  This means the first operand must
4756:   // always be 64-bit, and the second operand should be 32-bit whenever
4757:   // that is possible, to improve performance.
4758:   if (is32Bit(VT))
4759:     Op0 = DAG.getNode(ISD::SIGN_EXTEND, DL, MVT::i64, Op0);
4760:   else if (DAG.ComputeNumSignBits(Op1) > 32)
4761:     Op1 = DAG.getNode(ISD::TRUNCATE, DL, MVT::i32, Op1);
4762: 
4763:   // DSG(F) returns the remainder in the even register and the
4764:   // quotient in the odd register.
4765:   SDValue Ops[2];
4766:   lowerGR128Binary(DAG, DL, VT, SystemZISD::SDIVREM, Op0, Op1, Ops[1], Ops[0]);
4767:   return DAG.getMergeValues(Ops, DL);
4768: }
4769: 
4770: SDValue SystemZTargetLowering::lowerUDIVREM(SDValue Op,
4771:                                             SelectionDAG &DAG) const {
4772:   EVT VT = Op.getValueType();
4773:   SDLoc DL(Op);
4774: 
4775:   // DL(G) returns the remainder in the even register and the
4776:   // quotient in the odd register.
4777:   SDValue Ops[2];
4778:   lowerGR128Binary(DAG, DL, VT, SystemZISD::UDIVREM,
4779:                    Op.getOperand(0), Op.getOperand(1), Ops[1], Ops[0]);
4780:   return DAG.getMergeValues(Ops, DL);
4781: }
4782: 
4783: SDValue SystemZTargetLowering::lowerOR(SDValue Op, SelectionDAG &DAG) const {
4784:   assert(Op.getValueType() == MVT::i64 && "Should be 64-bit operation");
4785: 
4786:   // Get the known-zero masks for each operand.
4787:   SDValue Ops[] = {Op.getOperand(0), Op.getOperand(1)};
4788:   KnownBits Known[2] = {DAG.computeKnownBits(Ops[0]),
4789:                         DAG.computeKnownBits(Ops[1])};
4790: 
4791:   // See if the upper 32 bits of one operand and the lower 32 bits of the
4792:   // other are known zero.  They are the low and high operands respectively.
4793:   uint64_t Masks[] = { Known[0].Zero.getZExtValue(),
4794:                        Known[1].Zero.getZExtValue() };
4795:   unsigned High, Low;
4796:   if ((Masks[0] >> 32) == 0xffffffff && uint32_t(Masks[1]) == 0xffffffff)
4797:     High = 1, Low = 0;
4798:   else if ((Masks[1] >> 32) == 0xffffffff && uint32_t(Masks[0]) == 0xffffffff)
4799:     High = 0, Low = 1;
4800:   else
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerUMUL_LOHI`, `SystemZTargetLowering::lowerSDIVREM`, `SystemZTargetLowering::lowerUDIVREM`, `SystemZTargetLowering::lowerOR`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerUMUL_LOHI`, `SystemZTargetLowering::lowerSDIVREM`, `SystemZTargetLowering::lowerUDIVREM`, `SystemZTargetLowering::lowerOR` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 4801-4875 / 第 4801-4875 行
```cpp
4801:     return Op;
4802: 
4803:   SDValue LowOp = Ops[Low];
4804:   SDValue HighOp = Ops[High];
4805: 
4806:   // If the high part is a constant, we're better off using IILH.
4807:   if (HighOp.getOpcode() == ISD::Constant)
4808:     return Op;
4809: 
4810:   // If the low part is a constant that is outside the range of LHI,
4811:   // then we're better off using IILF.
4812:   if (LowOp.getOpcode() == ISD::Constant) {
4813:     int64_t Value = int32_t(LowOp->getAsZExtVal());
4814:     if (!isInt<16>(Value))
4815:       return Op;
4816:   }
4817: 
4818:   // Check whether the high part is an AND that doesn't change the
4819:   // high 32 bits and just masks out low bits.  We can skip it if so.
4820:   if (HighOp.getOpcode() == ISD::AND &&
4821:       HighOp.getOperand(1).getOpcode() == ISD::Constant) {
4822:     SDValue HighOp0 = HighOp.getOperand(0);
4823:     uint64_t Mask = HighOp.getConstantOperandVal(1);
4824:     if (DAG.MaskedValueIsZero(HighOp0, APInt(64, ~(Mask | 0xffffffff))))
4825:       HighOp = HighOp0;
4826:   }
4827: 
4828:   // Take advantage of the fact that all GR32 operations only change the
4829:   // low 32 bits by truncating Low to an i32 and inserting it directly
4830:   // using a subreg.  The interesting cases are those where the truncation
4831:   // can be folded.
4832:   SDLoc DL(Op);
4833:   SDValue Low32 = DAG.getNode(ISD::TRUNCATE, DL, MVT::i32, LowOp);
4834:   return DAG.getTargetInsertSubreg(SystemZ::subreg_l32, DL,
4835:                                    MVT::i64, HighOp, Low32);
4836: }
4837: 
4838: // Lower SADDO/SSUBO/UADDO/USUBO nodes.
4839: SDValue SystemZTargetLowering::lowerXALUO(SDValue Op,
4840:                                           SelectionDAG &DAG) const {
4841:   SDNode *N = Op.getNode();
4842:   SDValue LHS = N->getOperand(0);
4843:   SDValue RHS = N->getOperand(1);
4844:   SDLoc DL(N);
4845: 
4846:   if (N->getValueType(0) == MVT::i128) {
4847:     unsigned BaseOp = 0;
4848:     unsigned FlagOp = 0;
4849:     bool IsBorrow = false;
4850:     switch (Op.getOpcode()) {
4851:     default: llvm_unreachable("Unknown instruction!");
4852:     case ISD::UADDO:
4853:       BaseOp = ISD::ADD;
4854:       FlagOp = SystemZISD::VACC;
4855:       break;
4856:     case ISD::USUBO:
4857:       BaseOp = ISD::SUB;
4858:       FlagOp = SystemZISD::VSCBI;
4859:       IsBorrow = true;
4860:       break;
4861:     }
4862:     SDValue Result = DAG.getNode(BaseOp, DL, MVT::i128, LHS, RHS);
4863:     SDValue Flag = DAG.getNode(FlagOp, DL, MVT::i128, LHS, RHS);
4864:     Flag = DAG.getNode(ISD::AssertZext, DL, MVT::i128, Flag,
4865:                        DAG.getValueType(MVT::i1));
4866:     Flag = DAG.getZExtOrTrunc(Flag, DL, N->getValueType(1));
4867:     if (IsBorrow)
4868:       Flag = DAG.getNode(ISD::XOR, DL, Flag.getValueType(),
4869:                          Flag, DAG.getConstant(1, DL, Flag.getValueType()));
4870:     return DAG.getNode(ISD::MERGE_VALUES, DL, N->getVTList(), Result, Flag);
4871:   }
4872: 
4873:   unsigned BaseOp = 0;
4874:   unsigned CCValid = 0;
4875:   unsigned CCMask = 0;
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerXALUO`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerXALUO` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 4876-4950 / 第 4876-4950 行
```cpp
4876: 
4877:   switch (Op.getOpcode()) {
4878:   default: llvm_unreachable("Unknown instruction!");
4879:   case ISD::SADDO:
4880:     BaseOp = SystemZISD::SADDO;
4881:     CCValid = SystemZ::CCMASK_ARITH;
4882:     CCMask = SystemZ::CCMASK_ARITH_OVERFLOW;
4883:     break;
4884:   case ISD::SSUBO:
4885:     BaseOp = SystemZISD::SSUBO;
4886:     CCValid = SystemZ::CCMASK_ARITH;
4887:     CCMask = SystemZ::CCMASK_ARITH_OVERFLOW;
4888:     break;
4889:   case ISD::UADDO:
4890:     BaseOp = SystemZISD::UADDO;
4891:     CCValid = SystemZ::CCMASK_LOGICAL;
4892:     CCMask = SystemZ::CCMASK_LOGICAL_CARRY;
4893:     break;
4894:   case ISD::USUBO:
4895:     BaseOp = SystemZISD::USUBO;
4896:     CCValid = SystemZ::CCMASK_LOGICAL;
4897:     CCMask = SystemZ::CCMASK_LOGICAL_BORROW;
4898:     break;
4899:   }
4900: 
4901:   SDVTList VTs = DAG.getVTList(N->getValueType(0), MVT::i32);
4902:   SDValue Result = DAG.getNode(BaseOp, DL, VTs, LHS, RHS);
4903: 
4904:   SDValue SetCC = emitSETCC(DAG, DL, Result.getValue(1), CCValid, CCMask);
4905:   if (N->getValueType(1) == MVT::i1)
4906:     SetCC = DAG.getNode(ISD::TRUNCATE, DL, MVT::i1, SetCC);
4907: 
4908:   return DAG.getNode(ISD::MERGE_VALUES, DL, N->getVTList(), Result, SetCC);
4909: }
4910: 
4911: static bool isAddCarryChain(SDValue Carry) {
4912:   while (Carry.getOpcode() == ISD::UADDO_CARRY &&
4913:          Carry->getValueType(0) != MVT::i128)
4914:     Carry = Carry.getOperand(2);
4915:   return Carry.getOpcode() == ISD::UADDO &&
4916:          Carry->getValueType(0) != MVT::i128;
4917: }
4918: 
4919: static bool isSubBorrowChain(SDValue Carry) {
4920:   while (Carry.getOpcode() == ISD::USUBO_CARRY &&
4921:          Carry->getValueType(0) != MVT::i128)
4922:     Carry = Carry.getOperand(2);
4923:   return Carry.getOpcode() == ISD::USUBO &&
4924:          Carry->getValueType(0) != MVT::i128;
4925: }
4926: 
4927: // Lower UADDO_CARRY/USUBO_CARRY nodes.
4928: SDValue SystemZTargetLowering::lowerUADDSUBO_CARRY(SDValue Op,
4929:                                                    SelectionDAG &DAG) const {
4930: 
4931:   SDNode *N = Op.getNode();
4932:   MVT VT = N->getSimpleValueType(0);
4933: 
4934:   // Let legalize expand this if it isn't a legal type yet.
4935:   if (!DAG.getTargetLoweringInfo().isTypeLegal(VT))
4936:     return SDValue();
4937: 
4938:   SDValue LHS = N->getOperand(0);
4939:   SDValue RHS = N->getOperand(1);
4940:   SDValue Carry = Op.getOperand(2);
4941:   SDLoc DL(N);
4942: 
4943:   if (VT == MVT::i128) {
4944:     unsigned BaseOp = 0;
4945:     unsigned FlagOp = 0;
4946:     bool IsBorrow = false;
4947:     switch (Op.getOpcode()) {
4948:     default: llvm_unreachable("Unknown instruction!");
4949:     case ISD::UADDO_CARRY:
4950:       BaseOp = SystemZISD::VAC;
```
- **EN**: The range implements or declares functions including `isAddCarryChain`, `isSubBorrowChain`, `SystemZTargetLowering::lowerUADDSUBO_CARRY`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isAddCarryChain`, `isSubBorrowChain`, `SystemZTargetLowering::lowerUADDSUBO_CARRY` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 4951-5025 / 第 4951-5025 行
```cpp
4951:       FlagOp = SystemZISD::VACCC;
4952:       break;
4953:     case ISD::USUBO_CARRY:
4954:       BaseOp = SystemZISD::VSBI;
4955:       FlagOp = SystemZISD::VSBCBI;
4956:       IsBorrow = true;
4957:       break;
4958:     }
4959:     if (IsBorrow)
4960:       Carry = DAG.getNode(ISD::XOR, DL, Carry.getValueType(),
4961:                           Carry, DAG.getConstant(1, DL, Carry.getValueType()));
4962:     Carry = DAG.getZExtOrTrunc(Carry, DL, MVT::i128);
4963:     SDValue Result = DAG.getNode(BaseOp, DL, MVT::i128, LHS, RHS, Carry);
4964:     SDValue Flag = DAG.getNode(FlagOp, DL, MVT::i128, LHS, RHS, Carry);
4965:     Flag = DAG.getNode(ISD::AssertZext, DL, MVT::i128, Flag,
4966:                        DAG.getValueType(MVT::i1));
4967:     Flag = DAG.getZExtOrTrunc(Flag, DL, N->getValueType(1));
4968:     if (IsBorrow)
4969:       Flag = DAG.getNode(ISD::XOR, DL, Flag.getValueType(),
4970:                          Flag, DAG.getConstant(1, DL, Flag.getValueType()));
4971:     return DAG.getNode(ISD::MERGE_VALUES, DL, N->getVTList(), Result, Flag);
4972:   }
4973: 
4974:   unsigned BaseOp = 0;
4975:   unsigned CCValid = 0;
4976:   unsigned CCMask = 0;
4977: 
4978:   switch (Op.getOpcode()) {
4979:   default: llvm_unreachable("Unknown instruction!");
4980:   case ISD::UADDO_CARRY:
4981:     if (!isAddCarryChain(Carry))
4982:       return SDValue();
4983: 
4984:     BaseOp = SystemZISD::ADDCARRY;
4985:     CCValid = SystemZ::CCMASK_LOGICAL;
4986:     CCMask = SystemZ::CCMASK_LOGICAL_CARRY;
4987:     break;
4988:   case ISD::USUBO_CARRY:
4989:     if (!isSubBorrowChain(Carry))
4990:       return SDValue();
4991: 
4992:     BaseOp = SystemZISD::SUBCARRY;
4993:     CCValid = SystemZ::CCMASK_LOGICAL;
4994:     CCMask = SystemZ::CCMASK_LOGICAL_BORROW;
4995:     break;
4996:   }
4997: 
4998:   // Set the condition code from the carry flag.
4999:   Carry = DAG.getNode(SystemZISD::GET_CCMASK, DL, MVT::i32, Carry,
5000:                       DAG.getConstant(CCValid, DL, MVT::i32),
5001:                       DAG.getConstant(CCMask, DL, MVT::i32));
5002: 
5003:   SDVTList VTs = DAG.getVTList(VT, MVT::i32);
5004:   SDValue Result = DAG.getNode(BaseOp, DL, VTs, LHS, RHS, Carry);
5005: 
5006:   SDValue SetCC = emitSETCC(DAG, DL, Result.getValue(1), CCValid, CCMask);
5007:   if (N->getValueType(1) == MVT::i1)
5008:     SetCC = DAG.getNode(ISD::TRUNCATE, DL, MVT::i1, SetCC);
5009: 
5010:   return DAG.getNode(ISD::MERGE_VALUES, DL, N->getVTList(), Result, SetCC);
5011: }
5012: 
5013: SDValue SystemZTargetLowering::lowerCTPOP(SDValue Op,
5014:                                           SelectionDAG &DAG) const {
5015:   EVT VT = Op.getValueType();
5016:   SDLoc DL(Op);
5017:   Op = Op.getOperand(0);
5018: 
5019:   if (VT.getScalarSizeInBits() == 128) {
5020:     Op = DAG.getNode(ISD::BITCAST, DL, MVT::v2i64, Op);
5021:     Op = DAG.getNode(ISD::CTPOP, DL, MVT::v2i64, Op);
5022:     SDValue Tmp = DAG.getSplatBuildVector(MVT::v2i64, DL,
5023:                                           DAG.getConstant(0, DL, MVT::i64));
5024:     Op = DAG.getNode(SystemZISD::VSUM, DL, VT, Op, Tmp);
5025:     return Op;
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerCTPOP`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerCTPOP` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 5026-5100 / 第 5026-5100 行
```cpp
5026:   }
5027: 
5028:   // Handle vector types via VPOPCT.
5029:   if (VT.isVector()) {
5030:     Op = DAG.getNode(ISD::BITCAST, DL, MVT::v16i8, Op);
5031:     Op = DAG.getNode(SystemZISD::POPCNT, DL, MVT::v16i8, Op);
5032:     switch (VT.getScalarSizeInBits()) {
5033:     case 8:
5034:       break;
5035:     case 16: {
5036:       Op = DAG.getNode(ISD::BITCAST, DL, VT, Op);
5037:       SDValue Shift = DAG.getConstant(8, DL, MVT::i32);
5038:       SDValue Tmp = DAG.getNode(SystemZISD::VSHL_BY_SCALAR, DL, VT, Op, Shift);
5039:       Op = DAG.getNode(ISD::ADD, DL, VT, Op, Tmp);
5040:       Op = DAG.getNode(SystemZISD::VSRL_BY_SCALAR, DL, VT, Op, Shift);
5041:       break;
5042:     }
5043:     case 32: {
5044:       SDValue Tmp = DAG.getSplatBuildVector(MVT::v16i8, DL,
5045:                                             DAG.getConstant(0, DL, MVT::i32));
5046:       Op = DAG.getNode(SystemZISD::VSUM, DL, VT, Op, Tmp);
5047:       break;
5048:     }
5049:     case 64: {
5050:       SDValue Tmp = DAG.getSplatBuildVector(MVT::v16i8, DL,
5051:                                             DAG.getConstant(0, DL, MVT::i32));
5052:       Op = DAG.getNode(SystemZISD::VSUM, DL, MVT::v4i32, Op, Tmp);
5053:       Op = DAG.getNode(SystemZISD::VSUM, DL, VT, Op, Tmp);
5054:       break;
5055:     }
5056:     default:
5057:       llvm_unreachable("Unexpected type");
5058:     }
5059:     return Op;
5060:   }
5061: 
5062:   // Get the known-zero mask for the operand.
5063:   KnownBits Known = DAG.computeKnownBits(Op);
5064:   unsigned NumSignificantBits = Known.getMaxValue().getActiveBits();
5065:   if (NumSignificantBits == 0)
5066:     return DAG.getConstant(0, DL, VT);
5067: 
5068:   // Skip known-zero high parts of the operand.
5069:   int64_t OrigBitSize = VT.getSizeInBits();
5070:   int64_t BitSize = llvm::bit_ceil(NumSignificantBits);
5071:   BitSize = std::min(BitSize, OrigBitSize);
5072: 
5073:   // The POPCNT instruction counts the number of bits in each byte.
5074:   Op = DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, Op);
5075:   Op = DAG.getNode(SystemZISD::POPCNT, DL, MVT::i64, Op);
5076:   Op = DAG.getNode(ISD::TRUNCATE, DL, VT, Op);
5077: 
5078:   // Add up per-byte counts in a binary tree.  All bits of Op at
5079:   // position larger than BitSize remain zero throughout.
5080:   for (int64_t I = BitSize / 2; I >= 8; I = I / 2) {
5081:     SDValue Tmp = DAG.getNode(ISD::SHL, DL, VT, Op, DAG.getConstant(I, DL, VT));
5082:     if (BitSize != OrigBitSize)
5083:       Tmp = DAG.getNode(ISD::AND, DL, VT, Tmp,
5084:                         DAG.getConstant(((uint64_t)1 << BitSize) - 1, DL, VT));
5085:     Op = DAG.getNode(ISD::ADD, DL, VT, Op, Tmp);
5086:   }
5087: 
5088:   // Extract overall result from high byte.
5089:   if (BitSize > 8)
5090:     Op = DAG.getNode(ISD::SRL, DL, VT, Op,
5091:                      DAG.getConstant(BitSize - 8, DL, VT));
5092: 
5093:   return Op;
5094: }
5095: 
5096: SDValue SystemZTargetLowering::lowerATOMIC_FENCE(SDValue Op,
5097:                                                  SelectionDAG &DAG) const {
5098:   SDLoc DL(Op);
5099:   AtomicOrdering FenceOrdering =
5100:       static_cast<AtomicOrdering>(Op.getConstantOperandVal(1));
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerATOMIC_FENCE`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerATOMIC_FENCE` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 5101-5175 / 第 5101-5175 行
```cpp
5101:   SyncScope::ID FenceSSID =
5102:       static_cast<SyncScope::ID>(Op.getConstantOperandVal(2));
5103: 
5104:   // The only fence that needs an instruction is a sequentially-consistent
5105:   // cross-thread fence.
5106:   if (FenceOrdering == AtomicOrdering::SequentiallyConsistent &&
5107:       FenceSSID == SyncScope::System) {
5108:     return SDValue(DAG.getMachineNode(SystemZ::Serialize, DL, MVT::Other,
5109:                                       Op.getOperand(0)),
5110:                    0);
5111:   }
5112: 
5113:   // MEMBARRIER is a compiler barrier; it codegens to a no-op.
5114:   return DAG.getNode(ISD::MEMBARRIER, DL, MVT::Other, Op.getOperand(0));
5115: }
5116: 
5117: SDValue SystemZTargetLowering::lowerATOMIC_LOAD(SDValue Op,
5118:                                                 SelectionDAG &DAG) const {
5119:   EVT RegVT = Op.getValueType();
5120:   if (RegVT.getSizeInBits() == 128)
5121:     return lowerATOMIC_LDST_I128(Op, DAG);
5122:   return lowerLoadF16(Op, DAG);
5123: }
5124: 
5125: SDValue SystemZTargetLowering::lowerATOMIC_STORE(SDValue Op,
5126:                                                  SelectionDAG &DAG) const {
5127:   auto *Node = cast<AtomicSDNode>(Op.getNode());
5128:   if (Node->getMemoryVT().getSizeInBits() == 128)
5129:     return lowerATOMIC_LDST_I128(Op, DAG);
5130:   return lowerStoreF16(Op, DAG);
5131: }
5132: 
5133: SDValue SystemZTargetLowering::lowerATOMIC_LDST_I128(SDValue Op,
5134:                                                      SelectionDAG &DAG) const {
5135:   auto *Node = cast<AtomicSDNode>(Op.getNode());
5136:   assert(
5137:       (Node->getMemoryVT() == MVT::i128 || Node->getMemoryVT() == MVT::f128) &&
5138:       "Only custom lowering i128 or f128.");
5139:   // Use same code to handle both legal and non-legal i128 types.
5140:   SmallVector<SDValue, 2> Results;
5141:   LowerOperationWrapper(Node, Results, DAG);
5142:   return DAG.getMergeValues(Results, SDLoc(Op));
5143: }
5144: 
5145: // Prepare for a Compare And Swap for a subword operation. This needs to be
5146: // done in memory with 4 bytes at natural alignment.
5147: static void getCSAddressAndShifts(SDValue Addr, SelectionDAG &DAG, SDLoc DL,
5148:                                   SDValue &AlignedAddr, SDValue &BitShift,
5149:                                   SDValue &NegBitShift) {
5150:   EVT PtrVT = Addr.getValueType();
5151:   EVT WideVT = MVT::i32;
5152: 
5153:   // Get the address of the containing word.
5154:   AlignedAddr = DAG.getNode(ISD::AND, DL, PtrVT, Addr,
5155:                             DAG.getSignedConstant(-4, DL, PtrVT));
5156: 
5157:   // Get the number of bits that the word must be rotated left in order
5158:   // to bring the field to the top bits of a GR32.
5159:   BitShift = DAG.getNode(ISD::SHL, DL, PtrVT, Addr,
5160:                          DAG.getConstant(3, DL, PtrVT));
5161:   BitShift = DAG.getNode(ISD::TRUNCATE, DL, WideVT, BitShift);
5162: 
5163:   // Get the complementing shift amount, for rotating a field in the top
5164:   // bits back to its proper position.
5165:   NegBitShift = DAG.getNode(ISD::SUB, DL, WideVT,
5166:                             DAG.getConstant(0, DL, WideVT), BitShift);
5167: 
5168: }
5169: 
5170: // Op is an 8-, 16-bit or 32-bit ATOMIC_LOAD_* operation.  Lower the first
5171: // two into the fullword ATOMIC_LOADW_* operation given by Opcode.
5172: SDValue SystemZTargetLowering::lowerATOMIC_LOAD_OP(SDValue Op,
5173:                                                    SelectionDAG &DAG,
5174:                                                    unsigned Opcode) const {
5175:   auto *Node = cast<AtomicSDNode>(Op.getNode());
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerATOMIC_LOAD`, `SystemZTargetLowering::lowerATOMIC_STORE`, `SystemZTargetLowering::lowerATOMIC_LDST_I128`, `getCSAddressAndShifts`, `SystemZTargetLowering::lowerATOMIC_LOAD_OP`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerATOMIC_LOAD`, `SystemZTargetLowering::lowerATOMIC_STORE`, `SystemZTargetLowering::lowerATOMIC_LDST_I128`, `getCSAddressAndShifts`, `SystemZTargetLowering::lowerATOMIC_LOAD_OP` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 5176-5250 / 第 5176-5250 行
```cpp
5176: 
5177:   // 32-bit operations need no special handling.
5178:   EVT NarrowVT = Node->getMemoryVT();
5179:   EVT WideVT = MVT::i32;
5180:   if (NarrowVT == WideVT)
5181:     return Op;
5182: 
5183:   int64_t BitSize = NarrowVT.getSizeInBits();
5184:   SDValue ChainIn = Node->getChain();
5185:   SDValue Addr = Node->getBasePtr();
5186:   SDValue Src2 = Node->getVal();
5187:   MachineMemOperand *MMO = Node->getMemOperand();
5188:   SDLoc DL(Node);
5189: 
5190:   // Convert atomic subtracts of constants into additions.
5191:   if (Opcode == SystemZISD::ATOMIC_LOADW_SUB)
5192:     if (auto *Const = dyn_cast<ConstantSDNode>(Src2)) {
5193:       Opcode = SystemZISD::ATOMIC_LOADW_ADD;
5194:       Src2 = DAG.getSignedConstant(-Const->getSExtValue(), DL,
5195:                                    Src2.getValueType());
5196:     }
5197: 
5198:   SDValue AlignedAddr, BitShift, NegBitShift;
5199:   getCSAddressAndShifts(Addr, DAG, DL, AlignedAddr, BitShift, NegBitShift);
5200: 
5201:   // Extend the source operand to 32 bits and prepare it for the inner loop.
5202:   // ATOMIC_SWAPW uses RISBG to rotate the field left, but all other
5203:   // operations require the source to be shifted in advance.  (This shift
5204:   // can be folded if the source is constant.)  For AND and NAND, the lower
5205:   // bits must be set, while for other opcodes they should be left clear.
5206:   if (Opcode != SystemZISD::ATOMIC_SWAPW)
5207:     Src2 = DAG.getNode(ISD::SHL, DL, WideVT, Src2,
5208:                        DAG.getConstant(32 - BitSize, DL, WideVT));
5209:   if (Opcode == SystemZISD::ATOMIC_LOADW_AND ||
5210:       Opcode == SystemZISD::ATOMIC_LOADW_NAND)
5211:     Src2 = DAG.getNode(ISD::OR, DL, WideVT, Src2,
5212:                        DAG.getConstant(uint32_t(-1) >> BitSize, DL, WideVT));
5213: 
5214:   // Construct the ATOMIC_LOADW_* node.
5215:   SDVTList VTList = DAG.getVTList(WideVT, MVT::Other);
5216:   SDValue Ops[] = { ChainIn, AlignedAddr, Src2, BitShift, NegBitShift,
5217:                     DAG.getConstant(BitSize, DL, WideVT) };
5218:   SDValue AtomicOp = DAG.getMemIntrinsicNode(Opcode, DL, VTList, Ops,
5219:                                              NarrowVT, MMO);
5220: 
5221:   // Rotate the result of the final CS so that the field is in the lower
5222:   // bits of a GR32, then truncate it.
5223:   SDValue ResultShift = DAG.getNode(ISD::ADD, DL, WideVT, BitShift,
5224:                                     DAG.getConstant(BitSize, DL, WideVT));
5225:   SDValue Result = DAG.getNode(ISD::ROTL, DL, WideVT, AtomicOp, ResultShift);
5226: 
5227:   SDValue RetOps[2] = { Result, AtomicOp.getValue(1) };
5228:   return DAG.getMergeValues(RetOps, DL);
5229: }
5230: 
5231: // Op is an ATOMIC_LOAD_SUB operation.  Lower 8- and 16-bit operations into
5232: // ATOMIC_LOADW_SUBs and convert 32- and 64-bit operations into additions.
5233: SDValue SystemZTargetLowering::lowerATOMIC_LOAD_SUB(SDValue Op,
5234:                                                     SelectionDAG &DAG) const {
5235:   auto *Node = cast<AtomicSDNode>(Op.getNode());
5236:   EVT MemVT = Node->getMemoryVT();
5237:   if (MemVT == MVT::i32 || MemVT == MVT::i64) {
5238:     // A full-width operation: negate and use LAA(G).
5239:     assert(Op.getValueType() == MemVT && "Mismatched VTs");
5240:     assert(Subtarget.hasInterlockedAccess1() &&
5241:            "Should have been expanded by AtomicExpand pass.");
5242:     SDValue Src2 = Node->getVal();
5243:     SDLoc DL(Src2);
5244:     SDValue NegSrc2 =
5245:       DAG.getNode(ISD::SUB, DL, MemVT, DAG.getConstant(0, DL, MemVT), Src2);
5246:     return DAG.getAtomic(ISD::ATOMIC_LOAD_ADD, DL, MemVT,
5247:                          Node->getChain(), Node->getBasePtr(), NegSrc2,
5248:                          Node->getMemOperand());
5249:   }
5250: 
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerATOMIC_LOAD_SUB`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerATOMIC_LOAD_SUB` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 5251-5325 / 第 5251-5325 行
```cpp
5251:   return lowerATOMIC_LOAD_OP(Op, DAG, SystemZISD::ATOMIC_LOADW_SUB);
5252: }
5253: 
5254: // Lower 8/16/32/64-bit ATOMIC_CMP_SWAP_WITH_SUCCESS node.
5255: SDValue SystemZTargetLowering::lowerATOMIC_CMP_SWAP(SDValue Op,
5256:                                                     SelectionDAG &DAG) const {
5257:   auto *Node = cast<AtomicSDNode>(Op.getNode());
5258:   SDValue ChainIn = Node->getOperand(0);
5259:   SDValue Addr = Node->getOperand(1);
5260:   SDValue CmpVal = Node->getOperand(2);
5261:   SDValue SwapVal = Node->getOperand(3);
5262:   MachineMemOperand *MMO = Node->getMemOperand();
5263:   SDLoc DL(Node);
5264: 
5265:   if (Node->getMemoryVT() == MVT::i128) {
5266:     // Use same code to handle both legal and non-legal i128 types.
5267:     SmallVector<SDValue, 3> Results;
5268:     LowerOperationWrapper(Node, Results, DAG);
5269:     return DAG.getMergeValues(Results, DL);
5270:   }
5271: 
5272:   // We have native support for 32-bit and 64-bit compare and swap, but we
5273:   // still need to expand extracting the "success" result from the CC.
5274:   EVT NarrowVT = Node->getMemoryVT();
5275:   EVT WideVT = NarrowVT == MVT::i64 ? MVT::i64 : MVT::i32;
5276:   if (NarrowVT == WideVT) {
5277:     SDVTList Tys = DAG.getVTList(WideVT, MVT::i32, MVT::Other);
5278:     SDValue Ops[] = { ChainIn, Addr, CmpVal, SwapVal };
5279:     SDValue AtomicOp = DAG.getMemIntrinsicNode(SystemZISD::ATOMIC_CMP_SWAP,
5280:                                                DL, Tys, Ops, NarrowVT, MMO);
5281:     SDValue Success = emitSETCC(DAG, DL, AtomicOp.getValue(1),
5282:                                 SystemZ::CCMASK_CS, SystemZ::CCMASK_CS_EQ);
5283: 
5284:     DAG.ReplaceAllUsesOfValueWith(Op.getValue(0), AtomicOp.getValue(0));
5285:     DAG.ReplaceAllUsesOfValueWith(Op.getValue(1), Success);
5286:     DAG.ReplaceAllUsesOfValueWith(Op.getValue(2), AtomicOp.getValue(2));
5287:     return SDValue();
5288:   }
5289: 
5290:   // Convert 8-bit and 16-bit compare and swap to a loop, implemented
5291:   // via a fullword ATOMIC_CMP_SWAPW operation.
5292:   int64_t BitSize = NarrowVT.getSizeInBits();
5293: 
5294:   SDValue AlignedAddr, BitShift, NegBitShift;
5295:   getCSAddressAndShifts(Addr, DAG, DL, AlignedAddr, BitShift, NegBitShift);
5296: 
5297:   // Construct the ATOMIC_CMP_SWAPW node.
5298:   SDVTList VTList = DAG.getVTList(WideVT, MVT::i32, MVT::Other);
5299:   SDValue Ops[] = { ChainIn, AlignedAddr, CmpVal, SwapVal, BitShift,
5300:                     NegBitShift, DAG.getConstant(BitSize, DL, WideVT) };
5301:   SDValue AtomicOp = DAG.getMemIntrinsicNode(SystemZISD::ATOMIC_CMP_SWAPW, DL,
5302:                                              VTList, Ops, NarrowVT, MMO);
5303:   SDValue Success = emitSETCC(DAG, DL, AtomicOp.getValue(1),
5304:                               SystemZ::CCMASK_ICMP, SystemZ::CCMASK_CMP_EQ);
5305: 
5306:   // emitAtomicCmpSwapW() will zero extend the result (original value).
5307:   SDValue OrigVal = DAG.getNode(ISD::AssertZext, DL, WideVT, AtomicOp.getValue(0),
5308:                                 DAG.getValueType(NarrowVT));
5309:   DAG.ReplaceAllUsesOfValueWith(Op.getValue(0), OrigVal);
5310:   DAG.ReplaceAllUsesOfValueWith(Op.getValue(1), Success);
5311:   DAG.ReplaceAllUsesOfValueWith(Op.getValue(2), AtomicOp.getValue(2));
5312:   return SDValue();
5313: }
5314: 
5315: MachineMemOperand::Flags
5316: SystemZTargetLowering::getTargetMMOFlags(const Instruction &I) const {
5317:   // Because of how we convert atomic_load and atomic_store to normal loads and
5318:   // stores in the DAG, we need to ensure that the MMOs are marked volatile
5319:   // since DAGCombine hasn't been updated to account for atomic, but non
5320:   // volatile loads.  (See D57601)
5321:   if (auto *SI = dyn_cast<StoreInst>(&I))
5322:     if (SI->isAtomic())
5323:       return MachineMemOperand::MOVolatile;
5324:   if (auto *LI = dyn_cast<LoadInst>(&I))
5325:     if (LI->isAtomic())
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerATOMIC_CMP_SWAP`, `SystemZTargetLowering::getTargetMMOFlags`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerATOMIC_CMP_SWAP`, `SystemZTargetLowering::getTargetMMOFlags` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 5326-5400 / 第 5326-5400 行
```cpp
5326:       return MachineMemOperand::MOVolatile;
5327:   if (auto *AI = dyn_cast<AtomicRMWInst>(&I))
5328:     if (AI->isAtomic())
5329:       return MachineMemOperand::MOVolatile;
5330:   if (auto *AI = dyn_cast<AtomicCmpXchgInst>(&I))
5331:     if (AI->isAtomic())
5332:       return MachineMemOperand::MOVolatile;
5333:   return MachineMemOperand::MONone;
5334: }
5335: 
5336: SDValue SystemZTargetLowering::lowerSTACKSAVE(SDValue Op,
5337:                                               SelectionDAG &DAG) const {
5338:   MachineFunction &MF = DAG.getMachineFunction();
5339:   auto *Regs = Subtarget.getSpecialRegisters();
5340:   if (MF.getFunction().getCallingConv() == CallingConv::GHC)
5341:     report_fatal_error("Variable-sized stack allocations are not supported "
5342:                        "in GHC calling convention");
5343:   return DAG.getCopyFromReg(Op.getOperand(0), SDLoc(Op),
5344:                             Regs->getStackPointerRegister(), Op.getValueType());
5345: }
5346: 
5347: SDValue SystemZTargetLowering::lowerSTACKRESTORE(SDValue Op,
5348:                                                  SelectionDAG &DAG) const {
5349:   MachineFunction &MF = DAG.getMachineFunction();
5350:   auto *Regs = Subtarget.getSpecialRegisters();
5351:   bool StoreBackchain = MF.getSubtarget<SystemZSubtarget>().hasBackChain();
5352: 
5353:   if (MF.getFunction().getCallingConv() == CallingConv::GHC)
5354:     report_fatal_error("Variable-sized stack allocations are not supported "
5355:                        "in GHC calling convention");
5356: 
5357:   SDValue Chain = Op.getOperand(0);
5358:   SDValue NewSP = Op.getOperand(1);
5359:   SDValue Backchain;
5360:   SDLoc DL(Op);
5361: 
5362:   if (StoreBackchain) {
5363:     SDValue OldSP = DAG.getCopyFromReg(
5364:         Chain, DL, Regs->getStackPointerRegister(), MVT::i64);
5365:     Backchain = DAG.getLoad(MVT::i64, DL, Chain, getBackchainAddress(OldSP, DAG),
5366:                             MachinePointerInfo());
5367:   }
5368: 
5369:   Chain = DAG.getCopyToReg(Chain, DL, Regs->getStackPointerRegister(), NewSP);
5370: 
5371:   if (StoreBackchain)
5372:     Chain = DAG.getStore(Chain, DL, Backchain, getBackchainAddress(NewSP, DAG),
5373:                          MachinePointerInfo());
5374: 
5375:   return Chain;
5376: }
5377: 
5378: SDValue SystemZTargetLowering::lowerPREFETCH(SDValue Op,
5379:                                              SelectionDAG &DAG) const {
5380:   bool IsData = Op.getConstantOperandVal(4);
5381:   if (!IsData)
5382:     // Just preserve the chain.
5383:     return Op.getOperand(0);
5384: 
5385:   SDLoc DL(Op);
5386:   bool IsWrite = Op.getConstantOperandVal(2);
5387:   unsigned Code = IsWrite ? SystemZ::PFD_WRITE : SystemZ::PFD_READ;
5388:   auto *Node = cast<MemIntrinsicSDNode>(Op.getNode());
5389:   SDValue Ops[] = {Op.getOperand(0), DAG.getTargetConstant(Code, DL, MVT::i32),
5390:                    Op.getOperand(1)};
5391:   return DAG.getMemIntrinsicNode(SystemZISD::PREFETCH, DL,
5392:                                  Node->getVTList(), Ops,
5393:                                  Node->getMemoryVT(), Node->getMemOperand());
5394: }
5395: 
5396: SDValue
5397: SystemZTargetLowering::lowerINTRINSIC_W_CHAIN(SDValue Op,
5398:                                               SelectionDAG &DAG) const {
5399:   unsigned Opcode, CCValid;
5400:   if (isIntrinsicWithCCAndChain(Op, Opcode, CCValid)) {
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerSTACKSAVE`, `SystemZTargetLowering::lowerSTACKRESTORE`, `SystemZTargetLowering::lowerPREFETCH`, `SystemZTargetLowering::lowerINTRINSIC_W_CHAIN`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerSTACKSAVE`, `SystemZTargetLowering::lowerSTACKRESTORE`, `SystemZTargetLowering::lowerPREFETCH`, `SystemZTargetLowering::lowerINTRINSIC_W_CHAIN` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 5401-5475 / 第 5401-5475 行
```cpp
5401:     assert(Op->getNumValues() == 2 && "Expected only CC result and chain");
5402:     SDNode *Node = emitIntrinsicWithCCAndChain(DAG, Op, Opcode);
5403:     SDValue CC = getCCResult(DAG, SDValue(Node, 0));
5404:     DAG.ReplaceAllUsesOfValueWith(SDValue(Op.getNode(), 0), CC);
5405:     return SDValue();
5406:   }
5407: 
5408:   return SDValue();
5409: }
5410: 
5411: SDValue
5412: SystemZTargetLowering::lowerINTRINSIC_WO_CHAIN(SDValue Op,
5413:                                                SelectionDAG &DAG) const {
5414:   unsigned Opcode, CCValid;
5415:   if (isIntrinsicWithCC(Op, Opcode, CCValid)) {
5416:     SDNode *Node = emitIntrinsicWithCC(DAG, Op, Opcode);
5417:     if (Op->getNumValues() == 1)
5418:       return getCCResult(DAG, SDValue(Node, 0));
5419:     assert(Op->getNumValues() == 2 && "Expected a CC and non-CC result");
5420:     return DAG.getNode(ISD::MERGE_VALUES, SDLoc(Op), Op->getVTList(),
5421:                        SDValue(Node, 0), getCCResult(DAG, SDValue(Node, 1)));
5422:   }
5423: 
5424:   unsigned Id = Op.getConstantOperandVal(0);
5425:   switch (Id) {
5426:   case Intrinsic::thread_pointer:
5427:     return lowerThreadPointer(SDLoc(Op), DAG);
5428: 
5429:   case Intrinsic::s390_vpdi:
5430:     return DAG.getNode(SystemZISD::PERMUTE_DWORDS, SDLoc(Op), Op.getValueType(),
5431:                        Op.getOperand(1), Op.getOperand(2), Op.getOperand(3));
5432: 
5433:   case Intrinsic::s390_vperm:
5434:     return DAG.getNode(SystemZISD::PERMUTE, SDLoc(Op), Op.getValueType(),
5435:                        Op.getOperand(1), Op.getOperand(2), Op.getOperand(3));
5436: 
5437:   case Intrinsic::s390_vuphb:
5438:   case Intrinsic::s390_vuphh:
5439:   case Intrinsic::s390_vuphf:
5440:   case Intrinsic::s390_vuphg:
5441:     return DAG.getNode(SystemZISD::UNPACK_HIGH, SDLoc(Op), Op.getValueType(),
5442:                        Op.getOperand(1));
5443: 
5444:   case Intrinsic::s390_vuplhb:
5445:   case Intrinsic::s390_vuplhh:
5446:   case Intrinsic::s390_vuplhf:
5447:   case Intrinsic::s390_vuplhg:
5448:     return DAG.getNode(SystemZISD::UNPACKL_HIGH, SDLoc(Op), Op.getValueType(),
5449:                        Op.getOperand(1));
5450: 
5451:   case Intrinsic::s390_vuplb:
5452:   case Intrinsic::s390_vuplhw:
5453:   case Intrinsic::s390_vuplf:
5454:   case Intrinsic::s390_vuplg:
5455:     return DAG.getNode(SystemZISD::UNPACK_LOW, SDLoc(Op), Op.getValueType(),
5456:                        Op.getOperand(1));
5457: 
5458:   case Intrinsic::s390_vupllb:
5459:   case Intrinsic::s390_vupllh:
5460:   case Intrinsic::s390_vupllf:
5461:   case Intrinsic::s390_vupllg:
5462:     return DAG.getNode(SystemZISD::UNPACKL_LOW, SDLoc(Op), Op.getValueType(),
5463:                        Op.getOperand(1));
5464: 
5465:   case Intrinsic::s390_vsumb:
5466:   case Intrinsic::s390_vsumh:
5467:   case Intrinsic::s390_vsumgh:
5468:   case Intrinsic::s390_vsumgf:
5469:   case Intrinsic::s390_vsumqf:
5470:   case Intrinsic::s390_vsumqg:
5471:     return DAG.getNode(SystemZISD::VSUM, SDLoc(Op), Op.getValueType(),
5472:                        Op.getOperand(1), Op.getOperand(2));
5473: 
5474:   case Intrinsic::s390_vaq:
5475:     return DAG.getNode(ISD::ADD, SDLoc(Op), Op.getValueType(),
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerINTRINSIC_WO_CHAIN`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerINTRINSIC_WO_CHAIN` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 5476-5550 / 第 5476-5550 行
```cpp
5476:                        Op.getOperand(1), Op.getOperand(2));
5477:   case Intrinsic::s390_vaccb:
5478:   case Intrinsic::s390_vacch:
5479:   case Intrinsic::s390_vaccf:
5480:   case Intrinsic::s390_vaccg:
5481:   case Intrinsic::s390_vaccq:
5482:     return DAG.getNode(SystemZISD::VACC, SDLoc(Op), Op.getValueType(),
5483:                        Op.getOperand(1), Op.getOperand(2));
5484:   case Intrinsic::s390_vacq:
5485:     return DAG.getNode(SystemZISD::VAC, SDLoc(Op), Op.getValueType(),
5486:                        Op.getOperand(1), Op.getOperand(2), Op.getOperand(3));
5487:   case Intrinsic::s390_vacccq:
5488:     return DAG.getNode(SystemZISD::VACCC, SDLoc(Op), Op.getValueType(),
5489:                        Op.getOperand(1), Op.getOperand(2), Op.getOperand(3));
5490: 
5491:   case Intrinsic::s390_vsq:
5492:     return DAG.getNode(ISD::SUB, SDLoc(Op), Op.getValueType(),
5493:                        Op.getOperand(1), Op.getOperand(2));
5494:   case Intrinsic::s390_vscbib:
5495:   case Intrinsic::s390_vscbih:
5496:   case Intrinsic::s390_vscbif:
5497:   case Intrinsic::s390_vscbig:
5498:   case Intrinsic::s390_vscbiq:
5499:     return DAG.getNode(SystemZISD::VSCBI, SDLoc(Op), Op.getValueType(),
5500:                        Op.getOperand(1), Op.getOperand(2));
5501:   case Intrinsic::s390_vsbiq:
5502:     return DAG.getNode(SystemZISD::VSBI, SDLoc(Op), Op.getValueType(),
5503:                        Op.getOperand(1), Op.getOperand(2), Op.getOperand(3));
5504:   case Intrinsic::s390_vsbcbiq:
5505:     return DAG.getNode(SystemZISD::VSBCBI, SDLoc(Op), Op.getValueType(),
5506:                        Op.getOperand(1), Op.getOperand(2), Op.getOperand(3));
5507: 
5508:   case Intrinsic::s390_vmhb:
5509:   case Intrinsic::s390_vmhh:
5510:   case Intrinsic::s390_vmhf:
5511:   case Intrinsic::s390_vmhg:
5512:   case Intrinsic::s390_vmhq:
5513:     return DAG.getNode(ISD::MULHS, SDLoc(Op), Op.getValueType(),
5514:                        Op.getOperand(1), Op.getOperand(2));
5515:   case Intrinsic::s390_vmlhb:
5516:   case Intrinsic::s390_vmlhh:
5517:   case Intrinsic::s390_vmlhf:
5518:   case Intrinsic::s390_vmlhg:
5519:   case Intrinsic::s390_vmlhq:
5520:     return DAG.getNode(ISD::MULHU, SDLoc(Op), Op.getValueType(),
5521:                        Op.getOperand(1), Op.getOperand(2));
5522: 
5523:   case Intrinsic::s390_vmahb:
5524:   case Intrinsic::s390_vmahh:
5525:   case Intrinsic::s390_vmahf:
5526:   case Intrinsic::s390_vmahg:
5527:   case Intrinsic::s390_vmahq:
5528:     return DAG.getNode(SystemZISD::VMAH, SDLoc(Op), Op.getValueType(),
5529:                        Op.getOperand(1), Op.getOperand(2), Op.getOperand(3));
5530:   case Intrinsic::s390_vmalhb:
5531:   case Intrinsic::s390_vmalhh:
5532:   case Intrinsic::s390_vmalhf:
5533:   case Intrinsic::s390_vmalhg:
5534:   case Intrinsic::s390_vmalhq:
5535:     return DAG.getNode(SystemZISD::VMALH, SDLoc(Op), Op.getValueType(),
5536:                        Op.getOperand(1), Op.getOperand(2), Op.getOperand(3));
5537: 
5538:   case Intrinsic::s390_vmeb:
5539:   case Intrinsic::s390_vmeh:
5540:   case Intrinsic::s390_vmef:
5541:   case Intrinsic::s390_vmeg:
5542:     return DAG.getNode(SystemZISD::VME, SDLoc(Op), Op.getValueType(),
5543:                        Op.getOperand(1), Op.getOperand(2));
5544:   case Intrinsic::s390_vmleb:
5545:   case Intrinsic::s390_vmleh:
5546:   case Intrinsic::s390_vmlef:
5547:   case Intrinsic::s390_vmleg:
5548:     return DAG.getNode(SystemZISD::VMLE, SDLoc(Op), Op.getValueType(),
5549:                        Op.getOperand(1), Op.getOperand(2));
5550:   case Intrinsic::s390_vmob:
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 5551-5625 / 第 5551-5625 行
```cpp
5551:   case Intrinsic::s390_vmoh:
5552:   case Intrinsic::s390_vmof:
5553:   case Intrinsic::s390_vmog:
5554:     return DAG.getNode(SystemZISD::VMO, SDLoc(Op), Op.getValueType(),
5555:                        Op.getOperand(1), Op.getOperand(2));
5556:   case Intrinsic::s390_vmlob:
5557:   case Intrinsic::s390_vmloh:
5558:   case Intrinsic::s390_vmlof:
5559:   case Intrinsic::s390_vmlog:
5560:     return DAG.getNode(SystemZISD::VMLO, SDLoc(Op), Op.getValueType(),
5561:                        Op.getOperand(1), Op.getOperand(2));
5562: 
5563:   case Intrinsic::s390_vmaeb:
5564:   case Intrinsic::s390_vmaeh:
5565:   case Intrinsic::s390_vmaef:
5566:   case Intrinsic::s390_vmaeg:
5567:     return DAG.getNode(ISD::ADD, SDLoc(Op), Op.getValueType(),
5568:                        DAG.getNode(SystemZISD::VME, SDLoc(Op), Op.getValueType(),
5569:                                    Op.getOperand(1), Op.getOperand(2)),
5570:                        Op.getOperand(3));
5571:   case Intrinsic::s390_vmaleb:
5572:   case Intrinsic::s390_vmaleh:
5573:   case Intrinsic::s390_vmalef:
5574:   case Intrinsic::s390_vmaleg:
5575:     return DAG.getNode(ISD::ADD, SDLoc(Op), Op.getValueType(),
5576:                        DAG.getNode(SystemZISD::VMLE, SDLoc(Op), Op.getValueType(),
5577:                                    Op.getOperand(1), Op.getOperand(2)),
5578:                        Op.getOperand(3));
5579:   case Intrinsic::s390_vmaob:
5580:   case Intrinsic::s390_vmaoh:
5581:   case Intrinsic::s390_vmaof:
5582:   case Intrinsic::s390_vmaog:
5583:     return DAG.getNode(ISD::ADD, SDLoc(Op), Op.getValueType(),
5584:                        DAG.getNode(SystemZISD::VMO, SDLoc(Op), Op.getValueType(),
5585:                                    Op.getOperand(1), Op.getOperand(2)),
5586:                        Op.getOperand(3));
5587:   case Intrinsic::s390_vmalob:
5588:   case Intrinsic::s390_vmaloh:
5589:   case Intrinsic::s390_vmalof:
5590:   case Intrinsic::s390_vmalog:
5591:     return DAG.getNode(ISD::ADD, SDLoc(Op), Op.getValueType(),
5592:                        DAG.getNode(SystemZISD::VMLO, SDLoc(Op), Op.getValueType(),
5593:                                    Op.getOperand(1), Op.getOperand(2)),
5594:                        Op.getOperand(3));
5595:   }
5596: 
5597:   return SDValue();
5598: }
5599: 
5600: namespace {
5601: // Says that SystemZISD operation Opcode can be used to perform the equivalent
5602: // of a VPERM with permute vector Bytes.  If Opcode takes three operands,
5603: // Operand is the constant third operand, otherwise it is the number of
5604: // bytes in each element of the result.
5605: struct Permute {
5606:   unsigned Opcode;
5607:   unsigned Operand;
5608:   unsigned char Bytes[SystemZ::VectorBytes];
5609: };
5610: }
5611: 
5612: static const Permute PermuteForms[] = {
5613:   // VMRHG
5614:   { SystemZISD::MERGE_HIGH, 8,
5615:     { 0, 1, 2, 3, 4, 5, 6, 7, 16, 17, 18, 19, 20, 21, 22, 23 } },
5616:   // VMRHF
5617:   { SystemZISD::MERGE_HIGH, 4,
5618:     { 0, 1, 2, 3, 16, 17, 18, 19, 4, 5, 6, 7, 20, 21, 22, 23 } },
5619:   // VMRHH
5620:   { SystemZISD::MERGE_HIGH, 2,
5621:     { 0, 1, 16, 17, 2, 3, 18, 19, 4, 5, 20, 21, 6, 7, 22, 23 } },
5622:   // VMRHB
5623:   { SystemZISD::MERGE_HIGH, 1,
5624:     { 0, 16, 1, 17, 2, 18, 3, 19, 4, 20, 5, 21, 6, 22, 7, 23 } },
5625:   // VMRLG
```
- **EN**: It introduces interface types such as `Permute`, shaping how other backend components interact with this file.
- **CN**: 它引入了 `Permute` 等接口类型，定义了其他后端组件与本文件交互的方式。

### Lines 5626-5700 / 第 5626-5700 行
```cpp
5626:   { SystemZISD::MERGE_LOW, 8,
5627:     { 8, 9, 10, 11, 12, 13, 14, 15, 24, 25, 26, 27, 28, 29, 30, 31 } },
5628:   // VMRLF
5629:   { SystemZISD::MERGE_LOW, 4,
5630:     { 8, 9, 10, 11, 24, 25, 26, 27, 12, 13, 14, 15, 28, 29, 30, 31 } },
5631:   // VMRLH
5632:   { SystemZISD::MERGE_LOW, 2,
5633:     { 8, 9, 24, 25, 10, 11, 26, 27, 12, 13, 28, 29, 14, 15, 30, 31 } },
5634:   // VMRLB
5635:   { SystemZISD::MERGE_LOW, 1,
5636:     { 8, 24, 9, 25, 10, 26, 11, 27, 12, 28, 13, 29, 14, 30, 15, 31 } },
5637:   // VPKG
5638:   { SystemZISD::PACK, 4,
5639:     { 4, 5, 6, 7, 12, 13, 14, 15, 20, 21, 22, 23, 28, 29, 30, 31 } },
5640:   // VPKF
5641:   { SystemZISD::PACK, 2,
5642:     { 2, 3, 6, 7, 10, 11, 14, 15, 18, 19, 22, 23, 26, 27, 30, 31 } },
5643:   // VPKH
5644:   { SystemZISD::PACK, 1,
5645:     { 1, 3, 5, 7, 9, 11, 13, 15, 17, 19, 21, 23, 25, 27, 29, 31 } },
5646:   // VPDI V1, V2, 4  (low half of V1, high half of V2)
5647:   { SystemZISD::PERMUTE_DWORDS, 4,
5648:     { 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23 } },
5649:   // VPDI V1, V2, 1  (high half of V1, low half of V2)
5650:   { SystemZISD::PERMUTE_DWORDS, 1,
5651:     { 0, 1, 2, 3, 4, 5, 6, 7, 24, 25, 26, 27, 28, 29, 30, 31 } }
5652: };
5653: 
5654: // Called after matching a vector shuffle against a particular pattern.
5655: // Both the original shuffle and the pattern have two vector operands.
5656: // OpNos[0] is the operand of the original shuffle that should be used for
5657: // operand 0 of the pattern, or -1 if operand 0 of the pattern can be anything.
5658: // OpNos[1] is the same for operand 1 of the pattern.  Resolve these -1s and
5659: // set OpNo0 and OpNo1 to the shuffle operands that should actually be used
5660: // for operands 0 and 1 of the pattern.
5661: static bool chooseShuffleOpNos(int *OpNos, unsigned &OpNo0, unsigned &OpNo1) {
5662:   if (OpNos[0] < 0) {
5663:     if (OpNos[1] < 0)
5664:       return false;
5665:     OpNo0 = OpNo1 = OpNos[1];
5666:   } else if (OpNos[1] < 0) {
5667:     OpNo0 = OpNo1 = OpNos[0];
5668:   } else {
5669:     OpNo0 = OpNos[0];
5670:     OpNo1 = OpNos[1];
5671:   }
5672:   return true;
5673: }
5674: 
5675: // Bytes is a VPERM-like permute vector, except that -1 is used for
5676: // undefined bytes.  Return true if the VPERM can be implemented using P.
5677: // When returning true set OpNo0 to the VPERM operand that should be
5678: // used for operand 0 of P and likewise OpNo1 for operand 1 of P.
5679: //
5680: // For example, if swapping the VPERM operands allows P to match, OpNo0
5681: // will be 1 and OpNo1 will be 0.  If instead Bytes only refers to one
5682: // operand, but rewriting it to use two duplicated operands allows it to
5683: // match P, then OpNo0 and OpNo1 will be the same.
5684: static bool matchPermute(const SmallVectorImpl<int> &Bytes, const Permute &P,
5685:                          unsigned &OpNo0, unsigned &OpNo1) {
5686:   int OpNos[] = { -1, -1 };
5687:   for (unsigned I = 0; I < SystemZ::VectorBytes; ++I) {
5688:     int Elt = Bytes[I];
5689:     if (Elt >= 0) {
5690:       // Make sure that the two permute vectors use the same suboperand
5691:       // byte number.  Only the operand numbers (the high bits) are
5692:       // allowed to differ.
5693:       if ((Elt ^ P.Bytes[I]) & (SystemZ::VectorBytes - 1))
5694:         return false;
5695:       int ModelOpNo = P.Bytes[I] / SystemZ::VectorBytes;
5696:       int RealOpNo = unsigned(Elt) / SystemZ::VectorBytes;
5697:       // Make sure that the operand mappings are consistent with previous
5698:       // elements.
5699:       if (OpNos[ModelOpNo] == 1 - RealOpNo)
5700:         return false;
```
- **EN**: The range implements or declares functions including `chooseShuffleOpNos`, `matchPermute`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `chooseShuffleOpNos`, `matchPermute` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 5701-5775 / 第 5701-5775 行
```cpp
5701:       OpNos[ModelOpNo] = RealOpNo;
5702:     }
5703:   }
5704:   return chooseShuffleOpNos(OpNos, OpNo0, OpNo1);
5705: }
5706: 
5707: // As above, but search for a matching permute.
5708: static const Permute *matchPermute(const SmallVectorImpl<int> &Bytes,
5709:                                    unsigned &OpNo0, unsigned &OpNo1) {
5710:   for (auto &P : PermuteForms)
5711:     if (matchPermute(Bytes, P, OpNo0, OpNo1))
5712:       return &P;
5713:   return nullptr;
5714: }
5715: 
5716: // Bytes is a VPERM-like permute vector, except that -1 is used for
5717: // undefined bytes.  This permute is an operand of an outer permute.
5718: // See whether redistributing the -1 bytes gives a shuffle that can be
5719: // implemented using P.  If so, set Transform to a VPERM-like permute vector
5720: // that, when applied to the result of P, gives the original permute in Bytes.
5721: static bool matchDoublePermute(const SmallVectorImpl<int> &Bytes,
5722:                                const Permute &P,
5723:                                SmallVectorImpl<int> &Transform) {
5724:   unsigned To = 0;
5725:   for (unsigned From = 0; From < SystemZ::VectorBytes; ++From) {
5726:     int Elt = Bytes[From];
5727:     if (Elt < 0)
5728:       // Byte number From of the result is undefined.
5729:       Transform[From] = -1;
5730:     else {
5731:       while (P.Bytes[To] != Elt) {
5732:         To += 1;
5733:         if (To == SystemZ::VectorBytes)
5734:           return false;
5735:       }
5736:       Transform[From] = To;
5737:     }
5738:   }
5739:   return true;
5740: }
5741: 
5742: // As above, but search for a matching permute.
5743: static const Permute *matchDoublePermute(const SmallVectorImpl<int> &Bytes,
5744:                                          SmallVectorImpl<int> &Transform) {
5745:   for (auto &P : PermuteForms)
5746:     if (matchDoublePermute(Bytes, P, Transform))
5747:       return &P;
5748:   return nullptr;
5749: }
5750: 
5751: // Convert the mask of the given shuffle op into a byte-level mask,
5752: // as if it had type vNi8.
5753: static bool getVPermMask(SDValue ShuffleOp,
5754:                          SmallVectorImpl<int> &Bytes) {
5755:   EVT VT = ShuffleOp.getValueType();
5756:   unsigned NumElements = VT.getVectorNumElements();
5757:   unsigned BytesPerElement = VT.getVectorElementType().getStoreSize();
5758: 
5759:   if (auto *VSN = dyn_cast<ShuffleVectorSDNode>(ShuffleOp)) {
5760:     Bytes.resize(NumElements * BytesPerElement, -1);
5761:     for (unsigned I = 0; I < NumElements; ++I) {
5762:       int Index = VSN->getMaskElt(I);
5763:       if (Index >= 0)
5764:         for (unsigned J = 0; J < BytesPerElement; ++J)
5765:           Bytes[I * BytesPerElement + J] = Index * BytesPerElement + J;
5766:     }
5767:     return true;
5768:   }
5769:   if (SystemZISD::SPLAT == ShuffleOp.getOpcode() &&
5770:       isa<ConstantSDNode>(ShuffleOp.getOperand(1))) {
5771:     unsigned Index = ShuffleOp.getConstantOperandVal(1);
5772:     Bytes.resize(NumElements * BytesPerElement, -1);
5773:     for (unsigned I = 0; I < NumElements; ++I)
5774:       for (unsigned J = 0; J < BytesPerElement; ++J)
5775:         Bytes[I * BytesPerElement + J] = Index * BytesPerElement + J;
```
- **EN**: The range implements or declares functions including `matchDoublePermute`, `getVPermMask`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `matchDoublePermute`, `getVPermMask` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 5776-5850 / 第 5776-5850 行
```cpp
5776:     return true;
5777:   }
5778:   return false;
5779: }
5780: 
5781: // Bytes is a VPERM-like permute vector, except that -1 is used for
5782: // undefined bytes.  See whether bytes [Start, Start + BytesPerElement) of
5783: // the result come from a contiguous sequence of bytes from one input.
5784: // Set Base to the selector for the first byte if so.
5785: static bool getShuffleInput(const SmallVectorImpl<int> &Bytes, unsigned Start,
5786:                             unsigned BytesPerElement, int &Base) {
5787:   Base = -1;
5788:   for (unsigned I = 0; I < BytesPerElement; ++I) {
5789:     if (Bytes[Start + I] >= 0) {
5790:       unsigned Elem = Bytes[Start + I];
5791:       if (Base < 0) {
5792:         Base = Elem - I;
5793:         // Make sure the bytes would come from one input operand.
5794:         if (unsigned(Base) % Bytes.size() + BytesPerElement > Bytes.size())
5795:           return false;
5796:       } else if (unsigned(Base) != Elem - I)
5797:         return false;
5798:     }
5799:   }
5800:   return true;
5801: }
5802: 
5803: // Bytes is a VPERM-like permute vector, except that -1 is used for
5804: // undefined bytes.  Return true if it can be performed using VSLDB.
5805: // When returning true, set StartIndex to the shift amount and OpNo0
5806: // and OpNo1 to the VPERM operands that should be used as the first
5807: // and second shift operand respectively.
5808: static bool isShlDoublePermute(const SmallVectorImpl<int> &Bytes,
5809:                                unsigned &StartIndex, unsigned &OpNo0,
5810:                                unsigned &OpNo1) {
5811:   int OpNos[] = { -1, -1 };
5812:   int Shift = -1;
5813:   for (unsigned I = 0; I < 16; ++I) {
5814:     int Index = Bytes[I];
5815:     if (Index >= 0) {
5816:       int ExpectedShift = (Index - I) % SystemZ::VectorBytes;
5817:       int ModelOpNo = unsigned(ExpectedShift + I) / SystemZ::VectorBytes;
5818:       int RealOpNo = unsigned(Index) / SystemZ::VectorBytes;
5819:       if (Shift < 0)
5820:         Shift = ExpectedShift;
5821:       else if (Shift != ExpectedShift)
5822:         return false;
5823:       // Make sure that the operand mappings are consistent with previous
5824:       // elements.
5825:       if (OpNos[ModelOpNo] == 1 - RealOpNo)
5826:         return false;
5827:       OpNos[ModelOpNo] = RealOpNo;
5828:     }
5829:   }
5830:   StartIndex = Shift;
5831:   return chooseShuffleOpNos(OpNos, OpNo0, OpNo1);
5832: }
5833: 
5834: // Create a node that performs P on operands Op0 and Op1, casting the
5835: // operands to the appropriate type.  The type of the result is determined by P.
5836: static SDValue getPermuteNode(SelectionDAG &DAG, const SDLoc &DL,
5837:                               const Permute &P, SDValue Op0, SDValue Op1) {
5838:   // VPDI (PERMUTE_DWORDS) always operates on v2i64s.  The input
5839:   // elements of a PACK are twice as wide as the outputs.
5840:   unsigned InBytes = (P.Opcode == SystemZISD::PERMUTE_DWORDS ? 8 :
5841:                       P.Opcode == SystemZISD::PACK ? P.Operand * 2 :
5842:                       P.Operand);
5843:   // Cast both operands to the appropriate type.
5844:   MVT InVT = MVT::getVectorVT(MVT::getIntegerVT(InBytes * 8),
5845:                               SystemZ::VectorBytes / InBytes);
5846:   Op0 = DAG.getNode(ISD::BITCAST, DL, InVT, Op0);
5847:   Op1 = DAG.getNode(ISD::BITCAST, DL, InVT, Op1);
5848:   SDValue Op;
5849:   if (P.Opcode == SystemZISD::PERMUTE_DWORDS) {
5850:     SDValue Op2 = DAG.getTargetConstant(P.Operand, DL, MVT::i32);
```
- **EN**: The range implements or declares functions including `getShuffleInput`, `isShlDoublePermute`, `getPermuteNode`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `getShuffleInput`, `isShlDoublePermute`, `getPermuteNode` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 5851-5925 / 第 5851-5925 行
```cpp
5851:     Op = DAG.getNode(SystemZISD::PERMUTE_DWORDS, DL, InVT, Op0, Op1, Op2);
5852:   } else if (P.Opcode == SystemZISD::PACK) {
5853:     MVT OutVT = MVT::getVectorVT(MVT::getIntegerVT(P.Operand * 8),
5854:                                  SystemZ::VectorBytes / P.Operand);
5855:     Op = DAG.getNode(SystemZISD::PACK, DL, OutVT, Op0, Op1);
5856:   } else {
5857:     Op = DAG.getNode(P.Opcode, DL, InVT, Op0, Op1);
5858:   }
5859:   return Op;
5860: }
5861: 
5862: static bool isZeroVector(SDValue N) {
5863:   if (N->getOpcode() == ISD::BITCAST)
5864:     N = N->getOperand(0);
5865:   if (N->getOpcode() == ISD::SPLAT_VECTOR)
5866:     if (auto *Op = dyn_cast<ConstantSDNode>(N->getOperand(0)))
5867:       return Op->getZExtValue() == 0;
5868:   return ISD::isBuildVectorAllZeros(N.getNode());
5869: }
5870: 
5871: // Return the index of the zero/undef vector, or UINT32_MAX if not found.
5872: static uint32_t findZeroVectorIdx(SDValue *Ops, unsigned Num) {
5873:   for (unsigned I = 0; I < Num ; I++)
5874:     if (isZeroVector(Ops[I]))
5875:       return I;
5876:   return UINT32_MAX;
5877: }
5878: 
5879: // Bytes is a VPERM-like permute vector, except that -1 is used for
5880: // undefined bytes.  Implement it on operands Ops[0] and Ops[1] using
5881: // VSLDB or VPERM.
5882: static SDValue getGeneralPermuteNode(SelectionDAG &DAG, const SDLoc &DL,
5883:                                      SDValue *Ops,
5884:                                      const SmallVectorImpl<int> &Bytes) {
5885:   for (unsigned I = 0; I < 2; ++I)
5886:     Ops[I] = DAG.getNode(ISD::BITCAST, DL, MVT::v16i8, Ops[I]);
5887: 
5888:   // First see whether VSLDB can be used.
5889:   unsigned StartIndex, OpNo0, OpNo1;
5890:   if (isShlDoublePermute(Bytes, StartIndex, OpNo0, OpNo1))
5891:     return DAG.getNode(SystemZISD::SHL_DOUBLE, DL, MVT::v16i8, Ops[OpNo0],
5892:                        Ops[OpNo1],
5893:                        DAG.getTargetConstant(StartIndex, DL, MVT::i32));
5894: 
5895:   // Fall back on VPERM.  Construct an SDNode for the permute vector.  Try to
5896:   // eliminate a zero vector by reusing any zero index in the permute vector.
5897:   unsigned ZeroVecIdx = findZeroVectorIdx(&Ops[0], 2);
5898:   if (ZeroVecIdx != UINT32_MAX) {
5899:     bool MaskFirst = true;
5900:     int ZeroIdx = -1;
5901:     for (unsigned I = 0; I < SystemZ::VectorBytes; ++I) {
5902:       unsigned OpNo = unsigned(Bytes[I]) / SystemZ::VectorBytes;
5903:       unsigned Byte = unsigned(Bytes[I]) % SystemZ::VectorBytes;
5904:       if (OpNo == ZeroVecIdx && I == 0) {
5905:         // If the first byte is zero, use mask as first operand.
5906:         ZeroIdx = 0;
5907:         break;
5908:       }
5909:       if (OpNo != ZeroVecIdx && Byte == 0) {
5910:         // If mask contains a zero, use it by placing that vector first.
5911:         ZeroIdx = I + SystemZ::VectorBytes;
5912:         MaskFirst = false;
5913:         break;
5914:       }
5915:     }
5916:     if (ZeroIdx != -1) {
5917:       SDValue IndexNodes[SystemZ::VectorBytes];
5918:       for (unsigned I = 0; I < SystemZ::VectorBytes; ++I) {
5919:         if (Bytes[I] >= 0) {
5920:           unsigned OpNo = unsigned(Bytes[I]) / SystemZ::VectorBytes;
5921:           unsigned Byte = unsigned(Bytes[I]) % SystemZ::VectorBytes;
5922:           if (OpNo == ZeroVecIdx)
5923:             IndexNodes[I] = DAG.getConstant(ZeroIdx, DL, MVT::i32);
5924:           else {
5925:             unsigned BIdx = MaskFirst ? Byte + SystemZ::VectorBytes : Byte;
```
- **EN**: The range implements or declares functions including `isZeroVector`, `findZeroVectorIdx`, `getGeneralPermuteNode`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `isZeroVector`, `findZeroVectorIdx`, `getGeneralPermuteNode` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 5926-6000 / 第 5926-6000 行
```cpp
5926:             IndexNodes[I] = DAG.getConstant(BIdx, DL, MVT::i32);
5927:           }
5928:         } else
5929:           IndexNodes[I] = DAG.getUNDEF(MVT::i32);
5930:       }
5931:       SDValue Mask = DAG.getBuildVector(MVT::v16i8, DL, IndexNodes);
5932:       SDValue Src = ZeroVecIdx == 0 ? Ops[1] : Ops[0];
5933:       if (MaskFirst)
5934:         return DAG.getNode(SystemZISD::PERMUTE, DL, MVT::v16i8, Mask, Src,
5935:                            Mask);
5936:       else
5937:         return DAG.getNode(SystemZISD::PERMUTE, DL, MVT::v16i8, Src, Mask,
5938:                            Mask);
5939:     }
5940:   }
5941: 
5942:   SDValue IndexNodes[SystemZ::VectorBytes];
5943:   for (unsigned I = 0; I < SystemZ::VectorBytes; ++I)
5944:     if (Bytes[I] >= 0)
5945:       IndexNodes[I] = DAG.getConstant(Bytes[I], DL, MVT::i32);
5946:     else
5947:       IndexNodes[I] = DAG.getUNDEF(MVT::i32);
5948:   SDValue Op2 = DAG.getBuildVector(MVT::v16i8, DL, IndexNodes);
5949:   return DAG.getNode(SystemZISD::PERMUTE, DL, MVT::v16i8, Ops[0],
5950:                      (!Ops[1].isUndef() ? Ops[1] : Ops[0]), Op2);
5951: }
5952: 
5953: namespace {
5954: // Describes a general N-operand vector shuffle.
5955: struct GeneralShuffle {
5956:   GeneralShuffle(EVT vt)
5957:       : VT(vt), UnpackFromEltSize(UINT_MAX), UnpackLow(false) {}
5958:   void addUndef();
5959:   bool add(SDValue, unsigned);
5960:   SDValue getNode(SelectionDAG &, const SDLoc &);
5961:   void tryPrepareForUnpack();
5962:   bool unpackWasPrepared() { return UnpackFromEltSize <= 4; }
5963:   SDValue insertUnpackIfPrepared(SelectionDAG &DAG, const SDLoc &DL, SDValue Op);
5964: 
5965:   // The operands of the shuffle.
5966:   SmallVector<SDValue, SystemZ::VectorBytes> Ops;
5967: 
5968:   // Index I is -1 if byte I of the result is undefined.  Otherwise the
5969:   // result comes from byte Bytes[I] % SystemZ::VectorBytes of operand
5970:   // Bytes[I] / SystemZ::VectorBytes.
5971:   SmallVector<int, SystemZ::VectorBytes> Bytes;
5972: 
5973:   // The type of the shuffle result.
5974:   EVT VT;
5975: 
5976:   // Holds a value of 1, 2 or 4 if a final unpack has been prepared for.
5977:   unsigned UnpackFromEltSize;
5978:   // True if the final unpack uses the low half.
5979:   bool UnpackLow;
5980: };
5981: } // namespace
5982: 
5983: // Add an extra undefined element to the shuffle.
5984: void GeneralShuffle::addUndef() {
5985:   unsigned BytesPerElement = VT.getVectorElementType().getStoreSize();
5986:   for (unsigned I = 0; I < BytesPerElement; ++I)
5987:     Bytes.push_back(-1);
5988: }
5989: 
5990: // Add an extra element to the shuffle, taking it from element Elem of Op.
5991: // A null Op indicates a vector input whose value will be calculated later;
5992: // there is at most one such input per shuffle and it always has the same
5993: // type as the result. Aborts and returns false if the source vector elements
5994: // of an EXTRACT_VECTOR_ELT are smaller than the destination elements. Per
5995: // LLVM they become implicitly extended, but this is rare and not optimized.
5996: bool GeneralShuffle::add(SDValue Op, unsigned Elem) {
5997:   unsigned BytesPerElement = VT.getVectorElementType().getStoreSize();
5998: 
5999:   // The source vector can have wider elements than the result,
6000:   // either through an explicit TRUNCATE or because of type legalization.
```
- **EN**: It introduces interface types such as `GeneralShuffle`, shaping how other backend components interact with this file. The range implements or declares functions including `GeneralShuffle`, `unpackWasPrepared`, `GeneralShuffle::addUndef`, `GeneralShuffle::add`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `GeneralShuffle` 等接口类型，定义了其他后端组件与本文件交互的方式。 这一段实现或声明了 `GeneralShuffle`, `unpackWasPrepared`, `GeneralShuffle::addUndef`, `GeneralShuffle::add` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 6001-6075 / 第 6001-6075 行
```cpp
6001:   // We want the least significant part.
6002:   EVT FromVT = Op.getNode() ? Op.getValueType() : VT;
6003:   unsigned FromBytesPerElement = FromVT.getVectorElementType().getStoreSize();
6004: 
6005:   // Return false if the source elements are smaller than their destination
6006:   // elements.
6007:   if (FromBytesPerElement < BytesPerElement)
6008:     return false;
6009: 
6010:   unsigned Byte = ((Elem * FromBytesPerElement) % SystemZ::VectorBytes +
6011:                    (FromBytesPerElement - BytesPerElement));
6012: 
6013:   // Look through things like shuffles and bitcasts.
6014:   while (Op.getNode()) {
6015:     if (Op.getOpcode() == ISD::BITCAST)
6016:       Op = Op.getOperand(0);
6017:     else if (Op.getOpcode() == ISD::VECTOR_SHUFFLE && Op.hasOneUse()) {
6018:       // See whether the bytes we need come from a contiguous part of one
6019:       // operand.
6020:       SmallVector<int, SystemZ::VectorBytes> OpBytes;
6021:       if (!getVPermMask(Op, OpBytes))
6022:         break;
6023:       int NewByte;
6024:       if (!getShuffleInput(OpBytes, Byte, BytesPerElement, NewByte))
6025:         break;
6026:       if (NewByte < 0) {
6027:         addUndef();
6028:         return true;
6029:       }
6030:       Op = Op.getOperand(unsigned(NewByte) / SystemZ::VectorBytes);
6031:       Byte = unsigned(NewByte) % SystemZ::VectorBytes;
6032:     } else if (Op.isUndef()) {
6033:       addUndef();
6034:       return true;
6035:     } else
6036:       break;
6037:   }
6038: 
6039:   // Make sure that the source of the extraction is in Ops.
6040:   unsigned OpNo = 0;
6041:   for (; OpNo < Ops.size(); ++OpNo)
6042:     if (Ops[OpNo] == Op)
6043:       break;
6044:   if (OpNo == Ops.size())
6045:     Ops.push_back(Op);
6046: 
6047:   // Add the element to Bytes.
6048:   unsigned Base = OpNo * SystemZ::VectorBytes + Byte;
6049:   for (unsigned I = 0; I < BytesPerElement; ++I)
6050:     Bytes.push_back(Base + I);
6051: 
6052:   return true;
6053: }
6054: 
6055: // Return SDNodes for the completed shuffle.
6056: SDValue GeneralShuffle::getNode(SelectionDAG &DAG, const SDLoc &DL) {
6057:   assert(Bytes.size() == SystemZ::VectorBytes && "Incomplete vector");
6058: 
6059:   if (Ops.size() == 0)
6060:     return DAG.getUNDEF(VT);
6061: 
6062:   // Use a single unpack if possible as the last operation.
6063:   tryPrepareForUnpack();
6064: 
6065:   // Make sure that there are at least two shuffle operands.
6066:   if (Ops.size() == 1)
6067:     Ops.push_back(DAG.getUNDEF(MVT::v16i8));
6068: 
6069:   // Create a tree of shuffles, deferring root node until after the loop.
6070:   // Try to redistribute the undefined elements of non-root nodes so that
6071:   // the non-root shuffles match something like a pack or merge, then adjust
6072:   // the parent node's permute vector to compensate for the new order.
6073:   // Among other things, this copes with vectors like <2 x i16> that were
6074:   // padded with undefined elements during type legalization.
6075:   //
```
- **EN**: The range implements or declares functions including `GeneralShuffle::getNode`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `GeneralShuffle::getNode` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 6076-6150 / 第 6076-6150 行
```cpp
6076:   // In the best case this redistribution will lead to the whole tree
6077:   // using packs and merges.  It should rarely be a loss in other cases.
6078:   unsigned Stride = 1;
6079:   for (; Stride * 2 < Ops.size(); Stride *= 2) {
6080:     for (unsigned I = 0; I < Ops.size() - Stride; I += Stride * 2) {
6081:       SDValue SubOps[] = { Ops[I], Ops[I + Stride] };
6082: 
6083:       // Create a mask for just these two operands.
6084:       SmallVector<int, SystemZ::VectorBytes> NewBytes(SystemZ::VectorBytes);
6085:       for (unsigned J = 0; J < SystemZ::VectorBytes; ++J) {
6086:         unsigned OpNo = unsigned(Bytes[J]) / SystemZ::VectorBytes;
6087:         unsigned Byte = unsigned(Bytes[J]) % SystemZ::VectorBytes;
6088:         if (OpNo == I)
6089:           NewBytes[J] = Byte;
6090:         else if (OpNo == I + Stride)
6091:           NewBytes[J] = SystemZ::VectorBytes + Byte;
6092:         else
6093:           NewBytes[J] = -1;
6094:       }
6095:       // See if it would be better to reorganize NewMask to avoid using VPERM.
6096:       SmallVector<int, SystemZ::VectorBytes> NewBytesMap(SystemZ::VectorBytes);
6097:       if (const Permute *P = matchDoublePermute(NewBytes, NewBytesMap)) {
6098:         Ops[I] = getPermuteNode(DAG, DL, *P, SubOps[0], SubOps[1]);
6099:         // Applying NewBytesMap to Ops[I] gets back to NewBytes.
6100:         for (unsigned J = 0; J < SystemZ::VectorBytes; ++J) {
6101:           if (NewBytes[J] >= 0) {
6102:             assert(unsigned(NewBytesMap[J]) < SystemZ::VectorBytes &&
6103:                    "Invalid double permute");
6104:             Bytes[J] = I * SystemZ::VectorBytes + NewBytesMap[J];
6105:           } else
6106:             assert(NewBytesMap[J] < 0 && "Invalid double permute");
6107:         }
6108:       } else {
6109:         // Just use NewBytes on the operands.
6110:         Ops[I] = getGeneralPermuteNode(DAG, DL, SubOps, NewBytes);
6111:         for (unsigned J = 0; J < SystemZ::VectorBytes; ++J)
6112:           if (NewBytes[J] >= 0)
6113:             Bytes[J] = I * SystemZ::VectorBytes + J;
6114:       }
6115:     }
6116:   }
6117: 
6118:   // Now we just have 2 inputs.  Put the second operand in Ops[1].
6119:   if (Stride > 1) {
6120:     Ops[1] = Ops[Stride];
6121:     for (unsigned I = 0; I < SystemZ::VectorBytes; ++I)
6122:       if (Bytes[I] >= int(SystemZ::VectorBytes))
6123:         Bytes[I] -= (Stride - 1) * SystemZ::VectorBytes;
6124:   }
6125: 
6126:   // Look for an instruction that can do the permute without resorting
6127:   // to VPERM.
6128:   unsigned OpNo0, OpNo1;
6129:   SDValue Op;
6130:   if (unpackWasPrepared() && Ops[1].isUndef())
6131:     Op = Ops[0];
6132:   else if (const Permute *P = matchPermute(Bytes, OpNo0, OpNo1))
6133:     Op = getPermuteNode(DAG, DL, *P, Ops[OpNo0], Ops[OpNo1]);
6134:   else
6135:     Op = getGeneralPermuteNode(DAG, DL, &Ops[0], Bytes);
6136: 
6137:   Op = insertUnpackIfPrepared(DAG, DL, Op);
6138: 
6139:   return DAG.getNode(ISD::BITCAST, DL, VT, Op);
6140: }
6141: 
6142: #ifndef NDEBUG
6143: static void dumpBytes(const SmallVectorImpl<int> &Bytes, std::string Msg) {
6144:   dbgs() << Msg.c_str() << " { ";
6145:   for (unsigned I = 0; I < Bytes.size(); I++)
6146:     dbgs() << Bytes[I] << " ";
6147:   dbgs() << "}\n";
6148: }
6149: #endif
6150: 
```
- **EN**: The range implements or declares functions including `dumpBytes`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `dumpBytes` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 6151-6225 / 第 6151-6225 行
```cpp
6151: // If the Bytes vector matches an unpack operation, prepare to do the unpack
6152: // after all else by removing the zero vector and the effect of the unpack on
6153: // Bytes.
6154: void GeneralShuffle::tryPrepareForUnpack() {
6155:   uint32_t ZeroVecOpNo = findZeroVectorIdx(&Ops[0], Ops.size());
6156:   if (ZeroVecOpNo == UINT32_MAX || Ops.size() == 1)
6157:     return;
6158: 
6159:   // Only do this if removing the zero vector reduces the depth, otherwise
6160:   // the critical path will increase with the final unpack.
6161:   if (Ops.size() > 2 &&
6162:       Log2_32_Ceil(Ops.size()) == Log2_32_Ceil(Ops.size() - 1))
6163:     return;
6164: 
6165:   // Find an unpack that would allow removing the zero vector from Ops.
6166:   UnpackFromEltSize = 1;
6167:   for (; UnpackFromEltSize <= 4; UnpackFromEltSize *= 2) {
6168:     bool MatchUnpack = true;
6169:     SmallVector<int, SystemZ::VectorBytes> SrcBytes;
6170:     for (unsigned Elt = 0; Elt < SystemZ::VectorBytes; Elt++) {
6171:       unsigned ToEltSize = UnpackFromEltSize * 2;
6172:       bool IsZextByte = (Elt % ToEltSize) < UnpackFromEltSize;
6173:       if (!IsZextByte)
6174:         SrcBytes.push_back(Bytes[Elt]);
6175:       if (Bytes[Elt] != -1) {
6176:         unsigned OpNo = unsigned(Bytes[Elt]) / SystemZ::VectorBytes;
6177:         if (IsZextByte != (OpNo == ZeroVecOpNo)) {
6178:           MatchUnpack = false;
6179:           break;
6180:         }
6181:       }
6182:     }
6183:     if (MatchUnpack) {
6184:       if (Ops.size() == 2) {
6185:         // Don't use unpack if a single source operand needs rearrangement.
6186:         bool CanUseUnpackLow = true, CanUseUnpackHigh = true;
6187:         for (unsigned i = 0; i < SystemZ::VectorBytes / 2; i++) {
6188:           if (SrcBytes[i] == -1)
6189:             continue;
6190:           if (SrcBytes[i] % 16 != int(i))
6191:             CanUseUnpackHigh = false;
6192:           if (SrcBytes[i] % 16 != int(i + SystemZ::VectorBytes / 2))
6193:             CanUseUnpackLow = false;
6194:           if (!CanUseUnpackLow && !CanUseUnpackHigh) {
6195:             UnpackFromEltSize = UINT_MAX;
6196:             return;
6197:           }
6198:         }
6199:         if (!CanUseUnpackHigh)
6200:           UnpackLow = true;
6201:       }
6202:       break;
6203:     }
6204:   }
6205:   if (UnpackFromEltSize > 4)
6206:     return;
6207: 
6208:   LLVM_DEBUG(dbgs() << "Preparing for final unpack of element size "
6209:              << UnpackFromEltSize << ". Zero vector is Op#" << ZeroVecOpNo
6210:              << ".\n";
6211:              dumpBytes(Bytes, "Original Bytes vector:"););
6212: 
6213:   // Apply the unpack in reverse to the Bytes array.
6214:   unsigned B = 0;
6215:   if (UnpackLow) {
6216:     while (B < SystemZ::VectorBytes / 2)
6217:       Bytes[B++] = -1;
6218:   }
6219:   for (unsigned Elt = 0; Elt < SystemZ::VectorBytes;) {
6220:     Elt += UnpackFromEltSize;
6221:     for (unsigned i = 0; i < UnpackFromEltSize; i++, Elt++, B++)
6222:       Bytes[B] = Bytes[Elt];
6223:   }
6224:   if (!UnpackLow) {
6225:     while (B < SystemZ::VectorBytes)
```
- **EN**: The range implements or declares functions including `GeneralShuffle::tryPrepareForUnpack`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `GeneralShuffle::tryPrepareForUnpack` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 6226-6300 / 第 6226-6300 行
```cpp
6226:       Bytes[B++] = -1;
6227:   }
6228: 
6229:   // Remove the zero vector from Ops
6230:   Ops.erase(&Ops[ZeroVecOpNo]);
6231:   for (unsigned I = 0; I < SystemZ::VectorBytes; ++I)
6232:     if (Bytes[I] >= 0) {
6233:       unsigned OpNo = unsigned(Bytes[I]) / SystemZ::VectorBytes;
6234:       if (OpNo > ZeroVecOpNo)
6235:         Bytes[I] -= SystemZ::VectorBytes;
6236:     }
6237: 
6238:   LLVM_DEBUG(dumpBytes(Bytes, "Resulting Bytes vector, zero vector removed:");
6239:              dbgs() << "\n";);
6240: }
6241: 
6242: SDValue GeneralShuffle::insertUnpackIfPrepared(SelectionDAG &DAG,
6243:                                                const SDLoc &DL,
6244:                                                SDValue Op) {
6245:   if (!unpackWasPrepared())
6246:     return Op;
6247:   unsigned InBits = UnpackFromEltSize * 8;
6248:   EVT InVT = MVT::getVectorVT(MVT::getIntegerVT(InBits),
6249:                                 SystemZ::VectorBits / InBits);
6250:   SDValue PackedOp = DAG.getNode(ISD::BITCAST, DL, InVT, Op);
6251:   unsigned OutBits = InBits * 2;
6252:   EVT OutVT = MVT::getVectorVT(MVT::getIntegerVT(OutBits),
6253:                                SystemZ::VectorBits / OutBits);
6254:   return DAG.getNode(UnpackLow ? SystemZISD::UNPACKL_LOW
6255:                                : SystemZISD::UNPACKL_HIGH,
6256:                      DL, OutVT, PackedOp);
6257: }
6258: 
6259: // Return true if the given BUILD_VECTOR is a scalar-to-vector conversion.
6260: static bool isScalarToVector(SDValue Op) {
6261:   for (unsigned I = 1, E = Op.getNumOperands(); I != E; ++I)
6262:     if (!Op.getOperand(I).isUndef())
6263:       return false;
6264:   return true;
6265: }
6266: 
6267: // Return a vector of type VT that contains Value in the first element.
6268: // The other elements don't matter.
6269: static SDValue buildScalarToVector(SelectionDAG &DAG, const SDLoc &DL, EVT VT,
6270:                                    SDValue Value) {
6271:   // If we have a constant, replicate it to all elements and let the
6272:   // BUILD_VECTOR lowering take care of it.
6273:   if (Value.getOpcode() == ISD::Constant ||
6274:       Value.getOpcode() == ISD::ConstantFP) {
6275:     SmallVector<SDValue, 16> Ops(VT.getVectorNumElements(), Value);
6276:     return DAG.getBuildVector(VT, DL, Ops);
6277:   }
6278:   if (Value.isUndef())
6279:     return DAG.getUNDEF(VT);
6280:   return DAG.getNode(ISD::SCALAR_TO_VECTOR, DL, VT, Value);
6281: }
6282: 
6283: // Return a vector of type VT in which Op0 is in element 0 and Op1 is in
6284: // element 1.  Used for cases in which replication is cheap.
6285: static SDValue buildMergeScalars(SelectionDAG &DAG, const SDLoc &DL, EVT VT,
6286:                                  SDValue Op0, SDValue Op1) {
6287:   if (Op0.isUndef()) {
6288:     if (Op1.isUndef())
6289:       return DAG.getUNDEF(VT);
6290:     return DAG.getNode(SystemZISD::REPLICATE, DL, VT, Op1);
6291:   }
6292:   if (Op1.isUndef())
6293:     return DAG.getNode(SystemZISD::REPLICATE, DL, VT, Op0);
6294:   return DAG.getNode(SystemZISD::MERGE_HIGH, DL, VT,
6295:                      buildScalarToVector(DAG, DL, VT, Op0),
6296:                      buildScalarToVector(DAG, DL, VT, Op1));
6297: }
6298: 
6299: // Extend GPR scalars Op0 and Op1 to doublewords and return a v2i64
6300: // vector for them.
```
- **EN**: The range implements or declares functions including `GeneralShuffle::insertUnpackIfPrepared`, `isScalarToVector`, `buildScalarToVector`, `buildMergeScalars`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `GeneralShuffle::insertUnpackIfPrepared`, `isScalarToVector`, `buildScalarToVector`, `buildMergeScalars` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 6301-6375 / 第 6301-6375 行
```cpp
6301: static SDValue joinDwords(SelectionDAG &DAG, const SDLoc &DL, SDValue Op0,
6302:                           SDValue Op1) {
6303:   if (Op0.isUndef() && Op1.isUndef())
6304:     return DAG.getUNDEF(MVT::v2i64);
6305:   // If one of the two inputs is undefined then replicate the other one,
6306:   // in order to avoid using another register unnecessarily.
6307:   if (Op0.isUndef())
6308:     Op0 = Op1 = DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, Op1);
6309:   else if (Op1.isUndef())
6310:     Op0 = Op1 = DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, Op0);
6311:   else {
6312:     Op0 = DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, Op0);
6313:     Op1 = DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, Op1);
6314:   }
6315:   return DAG.getNode(SystemZISD::JOIN_DWORDS, DL, MVT::v2i64, Op0, Op1);
6316: }
6317: 
6318: // If a BUILD_VECTOR contains some EXTRACT_VECTOR_ELTs, it's usually
6319: // better to use VECTOR_SHUFFLEs on them, only using BUILD_VECTOR for
6320: // the non-EXTRACT_VECTOR_ELT elements.  See if the given BUILD_VECTOR
6321: // would benefit from this representation and return it if so.
6322: static SDValue tryBuildVectorShuffle(SelectionDAG &DAG,
6323:                                      BuildVectorSDNode *BVN) {
6324:   EVT VT = BVN->getValueType(0);
6325:   unsigned NumElements = VT.getVectorNumElements();
6326: 
6327:   // Represent the BUILD_VECTOR as an N-operand VECTOR_SHUFFLE-like operation
6328:   // on byte vectors.  If there are non-EXTRACT_VECTOR_ELT elements that still
6329:   // need a BUILD_VECTOR, add an additional placeholder operand for that
6330:   // BUILD_VECTOR and store its operands in ResidueOps.
6331:   GeneralShuffle GS(VT);
6332:   SmallVector<SDValue, SystemZ::VectorBytes> ResidueOps;
6333:   bool FoundOne = false;
6334:   for (unsigned I = 0; I < NumElements; ++I) {
6335:     SDValue Op = BVN->getOperand(I);
6336:     if (Op.getOpcode() == ISD::TRUNCATE)
6337:       Op = Op.getOperand(0);
6338:     if (Op.getOpcode() == ISD::EXTRACT_VECTOR_ELT &&
6339:         Op.getOperand(1).getOpcode() == ISD::Constant) {
6340:       unsigned Elem = Op.getConstantOperandVal(1);
6341:       if (!GS.add(Op.getOperand(0), Elem))
6342:         return SDValue();
6343:       FoundOne = true;
6344:     } else if (Op.isUndef()) {
6345:       GS.addUndef();
6346:     } else {
6347:       if (!GS.add(SDValue(), ResidueOps.size()))
6348:         return SDValue();
6349:       ResidueOps.push_back(BVN->getOperand(I));
6350:     }
6351:   }
6352: 
6353:   // Nothing to do if there are no EXTRACT_VECTOR_ELTs.
6354:   if (!FoundOne)
6355:     return SDValue();
6356: 
6357:   // Create the BUILD_VECTOR for the remaining elements, if any.
6358:   if (!ResidueOps.empty()) {
6359:     while (ResidueOps.size() < NumElements)
6360:       ResidueOps.push_back(DAG.getUNDEF(ResidueOps[0].getValueType()));
6361:     for (auto &Op : GS.Ops) {
6362:       if (!Op.getNode()) {
6363:         Op = DAG.getBuildVector(VT, SDLoc(BVN), ResidueOps);
6364:         break;
6365:       }
6366:     }
6367:   }
6368:   return GS.getNode(DAG, SDLoc(BVN));
6369: }
6370: 
6371: bool SystemZTargetLowering::isVectorElementLoad(SDValue Op) const {
6372:   if (Op.getOpcode() == ISD::LOAD && cast<LoadSDNode>(Op)->isUnindexed())
6373:     return true;
6374:   if (auto *AL = dyn_cast<AtomicSDNode>(Op))
6375:     if (AL->getOpcode() == ISD::ATOMIC_LOAD)
```
- **EN**: The range implements or declares functions including `joinDwords`, `tryBuildVectorShuffle`, `SystemZTargetLowering::isVectorElementLoad`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `joinDwords`, `tryBuildVectorShuffle`, `SystemZTargetLowering::isVectorElementLoad` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 6376-6450 / 第 6376-6450 行
```cpp
6376:       return true;
6377:   if (Subtarget.hasVectorEnhancements2() && Op.getOpcode() == SystemZISD::LRV)
6378:     return true;
6379:   return false;
6380: }
6381: 
6382: static SDValue mergeHighParts(SelectionDAG &DAG, const SDLoc &DL,
6383:                               unsigned MergedBits, EVT VT, SDValue Op0,
6384:                               SDValue Op1) {
6385:   MVT IntVecVT = MVT::getVectorVT(MVT::getIntegerVT(MergedBits),
6386:                                   SystemZ::VectorBits / MergedBits);
6387:   assert(VT.getSizeInBits() == 128 && IntVecVT.getSizeInBits() == 128 &&
6388:          "Handling full vectors only.");
6389:   Op0 = DAG.getNode(ISD::BITCAST, DL, IntVecVT, Op0);
6390:   Op1 = DAG.getNode(ISD::BITCAST, DL, IntVecVT, Op1);
6391:   SDValue Op = DAG.getNode(SystemZISD::MERGE_HIGH, DL, IntVecVT, Op0, Op1);
6392:   return DAG.getNode(ISD::BITCAST, DL, VT, Op);
6393: }
6394: 
6395: static SDValue buildFPVecFromScalars4(SelectionDAG &DAG, const SDLoc &DL,
6396:                                       EVT VT, SmallVectorImpl<SDValue> &Elems,
6397:                                       unsigned Pos) {
6398:   SDValue Op01 = buildMergeScalars(DAG, DL, VT, Elems[Pos + 0], Elems[Pos + 1]);
6399:   SDValue Op23 = buildMergeScalars(DAG, DL, VT, Elems[Pos + 2], Elems[Pos + 3]);
6400:   // Avoid unnecessary undefs by reusing the other operand.
6401:   if (Op01.isUndef()) {
6402:     if (Op23.isUndef())
6403:       return Op01;
6404:     Op01 = Op23;
6405:   } else if (Op23.isUndef())
6406:     Op23 = Op01;
6407:   // Merging identical replications is a no-op.
6408:   if (Op01.getOpcode() == SystemZISD::REPLICATE && Op01 == Op23)
6409:     return Op01;
6410:   unsigned MergedBits = VT.getSimpleVT().getScalarSizeInBits() * 2;
6411:   return mergeHighParts(DAG, DL, MergedBits, VT, Op01, Op23);
6412: }
6413: 
6414: // Combine GPR scalar values Elems into a vector of type VT.
6415: SDValue
6416: SystemZTargetLowering::buildVector(SelectionDAG &DAG, const SDLoc &DL, EVT VT,
6417:                                    SmallVectorImpl<SDValue> &Elems) const {
6418:   // See whether there is a single replicated value.
6419:   SDValue Single;
6420:   unsigned int NumElements = Elems.size();
6421:   unsigned int Count = 0;
6422:   for (auto Elem : Elems) {
6423:     if (!Elem.isUndef()) {
6424:       if (!Single.getNode())
6425:         Single = Elem;
6426:       else if (Elem != Single) {
6427:         Single = SDValue();
6428:         break;
6429:       }
6430:       Count += 1;
6431:     }
6432:   }
6433:   // There are three cases here:
6434:   //
6435:   // - if the only defined element is a loaded one, the best sequence
6436:   //   is a replicating load.
6437:   //
6438:   // - otherwise, if the only defined element is an i64 value, we will
6439:   //   end up with the same VLVGP sequence regardless of whether we short-cut
6440:   //   for replication or fall through to the later code.
6441:   //
6442:   // - otherwise, if the only defined element is an i32 or smaller value,
6443:   //   we would need 2 instructions to replicate it: VLVGP followed by VREPx.
6444:   //   This is only a win if the single defined element is used more than once.
6445:   //   In other cases we're better off using a single VLVGx.
6446:   if (Single.getNode() && (Count > 1 || isVectorElementLoad(Single)))
6447:     return DAG.getNode(SystemZISD::REPLICATE, DL, VT, Single);
6448: 
6449:   // If all elements are loads, use VLREP/VLEs (below).
6450:   bool AllLoads = true;
```
- **EN**: The range implements or declares functions including `mergeHighParts`, `buildFPVecFromScalars4`, `SystemZTargetLowering::buildVector`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `mergeHighParts`, `buildFPVecFromScalars4`, `SystemZTargetLowering::buildVector` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 6451-6525 / 第 6451-6525 行
```cpp
6451:   for (auto Elem : Elems)
6452:     if (!isVectorElementLoad(Elem)) {
6453:       AllLoads = false;
6454:       break;
6455:     }
6456: 
6457:   // The best way of building a v2i64 from two i64s is to use VLVGP.
6458:   if (VT == MVT::v2i64 && !AllLoads)
6459:     return joinDwords(DAG, DL, Elems[0], Elems[1]);
6460: 
6461:   // Use a 64-bit merge high to combine two doubles.
6462:   if (VT == MVT::v2f64 && !AllLoads)
6463:     return buildMergeScalars(DAG, DL, VT, Elems[0], Elems[1]);
6464: 
6465:   // Build v4f32 values directly from the FPRs:
6466:   //
6467:   //   <Axxx> <Bxxx> <Cxxxx> <Dxxx>
6468:   //         V              V         VMRHF
6469:   //      <ABxx>         <CDxx>
6470:   //                V                 VMRHG
6471:   //              <ABCD>
6472:   if (VT == MVT::v4f32 && !AllLoads)
6473:     return buildFPVecFromScalars4(DAG, DL, VT, Elems, 0);
6474: 
6475:   // Same for v8f16.
6476:   if (VT == MVT::v8f16 && !AllLoads) {
6477:     SDValue Op0123 = buildFPVecFromScalars4(DAG, DL, VT, Elems, 0);
6478:     SDValue Op4567 = buildFPVecFromScalars4(DAG, DL, VT, Elems, 4);
6479:     // Avoid unnecessary undefs by reusing the other operand.
6480:     if (Op0123.isUndef())
6481:       Op0123 = Op4567;
6482:     else if (Op4567.isUndef())
6483:       Op4567 = Op0123;
6484:     // Merging identical replications is a no-op.
6485:     if (Op0123.getOpcode() == SystemZISD::REPLICATE && Op0123 == Op4567)
6486:       return Op0123;
6487:     return mergeHighParts(DAG, DL, 64, VT, Op0123, Op4567);
6488:   }
6489: 
6490:   // Collect the constant terms.
6491:   SmallVector<SDValue, SystemZ::VectorBytes> Constants(NumElements, SDValue());
6492:   SmallVector<bool, SystemZ::VectorBytes> Done(NumElements, false);
6493: 
6494:   unsigned NumConstants = 0;
6495:   for (unsigned I = 0; I < NumElements; ++I) {
6496:     SDValue Elem = Elems[I];
6497:     if (Elem.getOpcode() == ISD::Constant ||
6498:         Elem.getOpcode() == ISD::ConstantFP) {
6499:       NumConstants += 1;
6500:       Constants[I] = Elem;
6501:       Done[I] = true;
6502:     }
6503:   }
6504:   // If there was at least one constant, fill in the other elements of
6505:   // Constants with undefs to get a full vector constant and use that
6506:   // as the starting point.
6507:   SDValue Result;
6508:   SDValue ReplicatedVal;
6509:   if (NumConstants > 0) {
6510:     for (unsigned I = 0; I < NumElements; ++I)
6511:       if (!Constants[I].getNode())
6512:         Constants[I] = DAG.getUNDEF(Elems[I].getValueType());
6513:     Result = DAG.getBuildVector(VT, DL, Constants);
6514:   } else {
6515:     // Otherwise try to use VLREP or VLVGP to start the sequence in order to
6516:     // avoid a false dependency on any previous contents of the vector
6517:     // register.
6518: 
6519:     // Use a VLREP if at least one element is a load. Make sure to replicate
6520:     // the load with the most elements having its value.
6521:     std::map<const SDNode*, unsigned> UseCounts;
6522:     SDNode *LoadMaxUses = nullptr;
6523:     for (unsigned I = 0; I < NumElements; ++I)
6524:       if (isVectorElementLoad(Elems[I])) {
6525:         SDNode *Ld = Elems[I].getNode();
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 6526-6600 / 第 6526-6600 行
```cpp
6526:         unsigned Count = ++UseCounts[Ld];
6527:         if (LoadMaxUses == nullptr || UseCounts[LoadMaxUses] < Count)
6528:           LoadMaxUses = Ld;
6529:       }
6530:     if (LoadMaxUses != nullptr) {
6531:       ReplicatedVal = SDValue(LoadMaxUses, 0);
6532:       Result = DAG.getNode(SystemZISD::REPLICATE, DL, VT, ReplicatedVal);
6533:     } else {
6534:       // Try to use VLVGP.
6535:       unsigned I1 = NumElements / 2 - 1;
6536:       unsigned I2 = NumElements - 1;
6537:       bool Def1 = !Elems[I1].isUndef();
6538:       bool Def2 = !Elems[I2].isUndef();
6539:       if (Def1 || Def2) {
6540:         SDValue Elem1 = Elems[Def1 ? I1 : I2];
6541:         SDValue Elem2 = Elems[Def2 ? I2 : I1];
6542:         Result = DAG.getNode(ISD::BITCAST, DL, VT,
6543:                              joinDwords(DAG, DL, Elem1, Elem2));
6544:         Done[I1] = true;
6545:         Done[I2] = true;
6546:       } else
6547:         Result = DAG.getUNDEF(VT);
6548:     }
6549:   }
6550: 
6551:   // Use VLVGx to insert the other elements.
6552:   for (unsigned I = 0; I < NumElements; ++I)
6553:     if (!Done[I] && !Elems[I].isUndef() && Elems[I] != ReplicatedVal)
6554:       Result = DAG.getNode(ISD::INSERT_VECTOR_ELT, DL, VT, Result, Elems[I],
6555:                            DAG.getConstant(I, DL, MVT::i32));
6556:   return Result;
6557: }
6558: 
6559: SDValue SystemZTargetLowering::lowerBUILD_VECTOR(SDValue Op,
6560:                                                  SelectionDAG &DAG) const {
6561:   auto *BVN = cast<BuildVectorSDNode>(Op.getNode());
6562:   SDLoc DL(Op);
6563:   EVT VT = Op.getValueType();
6564: 
6565:   if (BVN->isConstant()) {
6566:     if (SystemZVectorConstantInfo(BVN).isVectorConstantLegal(Subtarget))
6567:       return Op;
6568: 
6569:     // Fall back to loading it from memory.
6570:     return SDValue();
6571:   }
6572: 
6573:   // See if we should use shuffles to construct the vector from other vectors.
6574:   if (SDValue Res = tryBuildVectorShuffle(DAG, BVN))
6575:     return Res;
6576: 
6577:   // Detect SCALAR_TO_VECTOR conversions.
6578:   if (isOperationLegal(ISD::SCALAR_TO_VECTOR, VT) && isScalarToVector(Op))
6579:     return buildScalarToVector(DAG, DL, VT, Op.getOperand(0));
6580: 
6581:   // Otherwise use buildVector to build the vector up from GPRs.
6582:   unsigned NumElements = Op.getNumOperands();
6583:   SmallVector<SDValue, SystemZ::VectorBytes> Ops(NumElements);
6584:   for (unsigned I = 0; I < NumElements; ++I)
6585:     Ops[I] = Op.getOperand(I);
6586:   return buildVector(DAG, DL, VT, Ops);
6587: }
6588: 
6589: SDValue SystemZTargetLowering::lowerVECTOR_SHUFFLE(SDValue Op,
6590:                                                    SelectionDAG &DAG) const {
6591:   auto *VSN = cast<ShuffleVectorSDNode>(Op.getNode());
6592:   SDLoc DL(Op);
6593:   EVT VT = Op.getValueType();
6594:   unsigned NumElements = VT.getVectorNumElements();
6595: 
6596:   if (VSN->isSplat()) {
6597:     SDValue Op0 = Op.getOperand(0);
6598:     unsigned Index = VSN->getSplatIndex();
6599:     assert(Index < VT.getVectorNumElements() &&
6600:            "Splat index should be defined and in first operand");
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerBUILD_VECTOR`, `SystemZTargetLowering::lowerVECTOR_SHUFFLE`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerBUILD_VECTOR`, `SystemZTargetLowering::lowerVECTOR_SHUFFLE` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 6601-6675 / 第 6601-6675 行
```cpp
6601:     // See whether the value we're splatting is directly available as a scalar.
6602:     if ((Index == 0 && Op0.getOpcode() == ISD::SCALAR_TO_VECTOR) ||
6603:         Op0.getOpcode() == ISD::BUILD_VECTOR)
6604:       return DAG.getNode(SystemZISD::REPLICATE, DL, VT, Op0.getOperand(Index));
6605:     // Otherwise keep it as a vector-to-vector operation.
6606:     return DAG.getNode(SystemZISD::SPLAT, DL, VT, Op.getOperand(0),
6607:                        DAG.getTargetConstant(Index, DL, MVT::i32));
6608:   }
6609: 
6610:   GeneralShuffle GS(VT);
6611:   for (unsigned I = 0; I < NumElements; ++I) {
6612:     int Elt = VSN->getMaskElt(I);
6613:     if (Elt < 0)
6614:       GS.addUndef();
6615:     else if (!GS.add(Op.getOperand(unsigned(Elt) / NumElements),
6616:                      unsigned(Elt) % NumElements))
6617:       return SDValue();
6618:   }
6619:   return GS.getNode(DAG, SDLoc(VSN));
6620: }
6621: 
6622: SDValue SystemZTargetLowering::lowerSCALAR_TO_VECTOR(SDValue Op,
6623:                                                      SelectionDAG &DAG) const {
6624:   SDLoc DL(Op);
6625:   // Just insert the scalar into element 0 of an undefined vector.
6626:   return DAG.getNode(ISD::INSERT_VECTOR_ELT, DL,
6627:                      Op.getValueType(), DAG.getUNDEF(Op.getValueType()),
6628:                      Op.getOperand(0), DAG.getConstant(0, DL, MVT::i32));
6629: }
6630: 
6631: // Shift the lower 2 bytes of Op to the left in order to insert into the
6632: // upper 2 bytes of the FP register.
6633: static SDValue convertToF16(SDValue Op, SelectionDAG &DAG) {
6634:   assert(Op.getSimpleValueType() == MVT::i64 &&
6635:          "Expexted to convert i64 to f16.");
6636:   SDLoc DL(Op);
6637:   SDValue Shft = DAG.getNode(ISD::SHL, DL, MVT::i64, Op,
6638:                              DAG.getConstant(48, DL, MVT::i64));
6639:   SDValue BCast = DAG.getNode(ISD::BITCAST, DL, MVT::f64, Shft);
6640:   SDValue F16Val =
6641:       DAG.getTargetExtractSubreg(SystemZ::subreg_h16, DL, MVT::f16, BCast);
6642:   return F16Val;
6643: }
6644: 
6645: // Extract Op into GPR and shift the 2 f16 bytes to the right.
6646: static SDValue convertFromF16(SDValue Op, SDLoc DL, SelectionDAG &DAG) {
6647:   assert(Op.getSimpleValueType() == MVT::f16 &&
6648:          "Expected to convert f16 to i64.");
6649:   SDNode *U32 = DAG.getMachineNode(TargetOpcode::IMPLICIT_DEF, DL, MVT::f64);
6650:   SDValue In64 = DAG.getTargetInsertSubreg(SystemZ::subreg_h16, DL, MVT::f64,
6651:                                            SDValue(U32, 0), Op);
6652:   SDValue BCast = DAG.getNode(ISD::BITCAST, DL, MVT::i64, In64);
6653:   SDValue Shft = DAG.getNode(ISD::SRL, DL, MVT::i64, BCast,
6654:                              DAG.getConstant(48, DL, MVT::i32));
6655:   return Shft;
6656: }
6657: 
6658: SDValue SystemZTargetLowering::lowerINSERT_VECTOR_ELT(SDValue Op,
6659:                                                       SelectionDAG &DAG) const {
6660:   // Handle insertions of floating-point values.
6661:   SDLoc DL(Op);
6662:   SDValue Op0 = Op.getOperand(0);
6663:   SDValue Op1 = Op.getOperand(1);
6664:   SDValue Op2 = Op.getOperand(2);
6665:   EVT VT = Op.getValueType();
6666: 
6667:   // Insertions into constant indices of a v2f64 can be done using VPDI.
6668:   // However, if the inserted value is a bitcast or a constant then it's
6669:   // better to use GPRs, as below.
6670:   if (VT == MVT::v2f64 &&
6671:       Op1.getOpcode() != ISD::BITCAST &&
6672:       Op1.getOpcode() != ISD::ConstantFP &&
6673:       Op2.getOpcode() == ISD::Constant) {
6674:     uint64_t Index = Op2->getAsZExtVal();
6675:     unsigned Mask = VT.getVectorNumElements() - 1;
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerSCALAR_TO_VECTOR`, `convertToF16`, `convertFromF16`, `SystemZTargetLowering::lowerINSERT_VECTOR_ELT`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerSCALAR_TO_VECTOR`, `convertToF16`, `convertFromF16`, `SystemZTargetLowering::lowerINSERT_VECTOR_ELT` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 6676-6750 / 第 6676-6750 行
```cpp
6676:     if (Index <= Mask)
6677:       return Op;
6678:   }
6679: 
6680:   // Otherwise bitcast to the equivalent integer form and insert via a GPR.
6681:   MVT IntVT = MVT::getIntegerVT(VT.getScalarSizeInBits());
6682:   MVT IntVecVT = MVT::getVectorVT(IntVT, VT.getVectorNumElements());
6683:   SDValue IntOp1 =
6684:       VT == MVT::v8f16
6685:           ? DAG.getZExtOrTrunc(convertFromF16(Op1, DL, DAG), DL, MVT::i32)
6686:           : DAG.getNode(ISD::BITCAST, DL, IntVT, Op1);
6687:   SDValue Res =
6688:       DAG.getNode(ISD::INSERT_VECTOR_ELT, DL, IntVecVT,
6689:                   DAG.getNode(ISD::BITCAST, DL, IntVecVT, Op0), IntOp1, Op2);
6690:   return DAG.getNode(ISD::BITCAST, DL, VT, Res);
6691: }
6692: 
6693: SDValue
6694: SystemZTargetLowering::lowerEXTRACT_VECTOR_ELT(SDValue Op,
6695:                                                SelectionDAG &DAG) const {
6696:   // Handle extractions of floating-point values.
6697:   SDLoc DL(Op);
6698:   SDValue Op0 = Op.getOperand(0);
6699:   SDValue Op1 = Op.getOperand(1);
6700:   EVT VT = Op.getValueType();
6701:   EVT VecVT = Op0.getValueType();
6702: 
6703:   // Extractions of constant indices can be done directly.
6704:   if (auto *CIndexN = dyn_cast<ConstantSDNode>(Op1)) {
6705:     uint64_t Index = CIndexN->getZExtValue();
6706:     unsigned Mask = VecVT.getVectorNumElements() - 1;
6707:     if (Index <= Mask)
6708:       return Op;
6709:   }
6710: 
6711:   // Otherwise bitcast to the equivalent integer form and extract via a GPR.
6712:   MVT IntVT = MVT::getIntegerVT(VT.getSizeInBits());
6713:   MVT IntVecVT = MVT::getVectorVT(IntVT, VecVT.getVectorNumElements());
6714:   MVT ExtrVT = IntVT == MVT::i16 ? MVT::i32 : IntVT;
6715:   SDValue Extr = DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, ExtrVT,
6716:                              DAG.getNode(ISD::BITCAST, DL, IntVecVT, Op0), Op1);
6717:   if (VT == MVT::f16)
6718:     return convertToF16(DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, Extr), DAG);
6719:   return DAG.getNode(ISD::BITCAST, DL, VT, Extr);
6720: }
6721: 
6722: SDValue SystemZTargetLowering::
6723: lowerSIGN_EXTEND_VECTOR_INREG(SDValue Op, SelectionDAG &DAG) const {
6724:   SDValue PackedOp = Op.getOperand(0);
6725:   EVT OutVT = Op.getValueType();
6726:   EVT InVT = PackedOp.getValueType();
6727:   unsigned ToBits = OutVT.getScalarSizeInBits();
6728:   unsigned FromBits = InVT.getScalarSizeInBits();
6729:   unsigned StartOffset = 0;
6730: 
6731:   // If the input is a VECTOR_SHUFFLE, there are a number of important
6732:   // cases where we can directly implement the sign-extension of the
6733:   // original input lanes of the shuffle.
6734:   if (PackedOp.getOpcode() == ISD::VECTOR_SHUFFLE) {
6735:     ShuffleVectorSDNode *SVN = cast<ShuffleVectorSDNode>(PackedOp.getNode());
6736:     ArrayRef<int> ShuffleMask = SVN->getMask();
6737:     int OutNumElts = OutVT.getVectorNumElements();
6738: 
6739:     // Recognize the special case where the sign-extension can be done
6740:     // by the VSEG instruction.  Handled via the default expander.
6741:     if (ToBits == 64 && OutNumElts == 2) {
6742:       int NumElem = ToBits / FromBits;
6743:       if (ShuffleMask[0] == NumElem - 1 && ShuffleMask[1] == 2 * NumElem - 1)
6744:         return SDValue();
6745:     }
6746: 
6747:     // Recognize the special case where we can fold the shuffle by
6748:     // replacing some of the UNPACK_HIGH with UNPACK_LOW.
6749:     int StartOffsetCandidate = -1;
6750:     for (int Elt = 0; Elt < OutNumElts; Elt++) {
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerEXTRACT_VECTOR_ELT`, `lowerSIGN_EXTEND_VECTOR_INREG`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerEXTRACT_VECTOR_ELT`, `lowerSIGN_EXTEND_VECTOR_INREG` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 6751-6825 / 第 6751-6825 行
```cpp
6751:       if (ShuffleMask[Elt] == -1)
6752:         continue;
6753:       if (ShuffleMask[Elt] % OutNumElts == Elt) {
6754:         if (StartOffsetCandidate == -1)
6755:           StartOffsetCandidate = ShuffleMask[Elt] - Elt;
6756:         if (StartOffsetCandidate == ShuffleMask[Elt] - Elt)
6757:           continue;
6758:       }
6759:       StartOffsetCandidate = -1;
6760:       break;
6761:     }
6762:     if (StartOffsetCandidate != -1) {
6763:       StartOffset = StartOffsetCandidate;
6764:       PackedOp = PackedOp.getOperand(0);
6765:     }
6766:   }
6767: 
6768:   do {
6769:     FromBits *= 2;
6770:     unsigned OutNumElts = SystemZ::VectorBits / FromBits;
6771:     EVT OutVT = MVT::getVectorVT(MVT::getIntegerVT(FromBits), OutNumElts);
6772:     unsigned Opcode = SystemZISD::UNPACK_HIGH;
6773:     if (StartOffset >= OutNumElts) {
6774:       Opcode = SystemZISD::UNPACK_LOW;
6775:       StartOffset -= OutNumElts;
6776:     }
6777:     PackedOp = DAG.getNode(Opcode, SDLoc(PackedOp), OutVT, PackedOp);
6778:   } while (FromBits != ToBits);
6779:   return PackedOp;
6780: }
6781: 
6782: // Lower a ZERO_EXTEND_VECTOR_INREG to a vector shuffle with a zero vector.
6783: SDValue SystemZTargetLowering::
6784: lowerZERO_EXTEND_VECTOR_INREG(SDValue Op, SelectionDAG &DAG) const {
6785:   SDValue PackedOp = Op.getOperand(0);
6786:   SDLoc DL(Op);
6787:   EVT OutVT = Op.getValueType();
6788:   EVT InVT = PackedOp.getValueType();
6789:   unsigned InNumElts = InVT.getVectorNumElements();
6790:   unsigned OutNumElts = OutVT.getVectorNumElements();
6791:   unsigned NumInPerOut = InNumElts / OutNumElts;
6792: 
6793:   SDValue ZeroVec =
6794:     DAG.getSplatVector(InVT, DL, DAG.getConstant(0, DL, InVT.getScalarType()));
6795: 
6796:   SmallVector<int, 16> Mask(InNumElts);
6797:   unsigned ZeroVecElt = InNumElts;
6798:   for (unsigned PackedElt = 0; PackedElt < OutNumElts; PackedElt++) {
6799:     unsigned MaskElt = PackedElt * NumInPerOut;
6800:     unsigned End = MaskElt + NumInPerOut - 1;
6801:     for (; MaskElt < End; MaskElt++)
6802:       Mask[MaskElt] = ZeroVecElt++;
6803:     Mask[MaskElt] = PackedElt;
6804:   }
6805:   SDValue Shuf = DAG.getVectorShuffle(InVT, DL, PackedOp, ZeroVec, Mask);
6806:   return DAG.getNode(ISD::BITCAST, DL, OutVT, Shuf);
6807: }
6808: 
6809: SDValue SystemZTargetLowering::lowerShift(SDValue Op, SelectionDAG &DAG,
6810:                                           unsigned ByScalar) const {
6811:   // Look for cases where a vector shift can use the *_BY_SCALAR form.
6812:   SDValue Op0 = Op.getOperand(0);
6813:   SDValue Op1 = Op.getOperand(1);
6814:   SDLoc DL(Op);
6815:   EVT VT = Op.getValueType();
6816:   unsigned ElemBitSize = VT.getScalarSizeInBits();
6817: 
6818:   // See whether the shift vector is a splat represented as BUILD_VECTOR.
6819:   if (auto *BVN = dyn_cast<BuildVectorSDNode>(Op1)) {
6820:     APInt SplatBits, SplatUndef;
6821:     unsigned SplatBitSize;
6822:     bool HasAnyUndefs;
6823:     // Check for constant splats.  Use ElemBitSize as the minimum element
6824:     // width and reject splats that need wider elements.
6825:     if (BVN->isConstantSplat(SplatBits, SplatUndef, SplatBitSize, HasAnyUndefs,
```
- **EN**: The range implements or declares functions including `lowerZERO_EXTEND_VECTOR_INREG`, `SystemZTargetLowering::lowerShift`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerZERO_EXTEND_VECTOR_INREG`, `SystemZTargetLowering::lowerShift` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 6826-6900 / 第 6826-6900 行
```cpp
6826:                              ElemBitSize, true) &&
6827:         SplatBitSize == ElemBitSize) {
6828:       SDValue Shift = DAG.getConstant(SplatBits.getZExtValue() & 0xfff,
6829:                                       DL, MVT::i32);
6830:       return DAG.getNode(ByScalar, DL, VT, Op0, Shift);
6831:     }
6832:     // Check for variable splats.
6833:     BitVector UndefElements;
6834:     SDValue Splat = BVN->getSplatValue(&UndefElements);
6835:     if (Splat) {
6836:       // Since i32 is the smallest legal type, we either need a no-op
6837:       // or a truncation.
6838:       SDValue Shift = DAG.getNode(ISD::TRUNCATE, DL, MVT::i32, Splat);
6839:       return DAG.getNode(ByScalar, DL, VT, Op0, Shift);
6840:     }
6841:   }
6842: 
6843:   // See whether the shift vector is a splat represented as SHUFFLE_VECTOR,
6844:   // and the shift amount is directly available in a GPR.
6845:   if (auto *VSN = dyn_cast<ShuffleVectorSDNode>(Op1)) {
6846:     if (VSN->isSplat()) {
6847:       SDValue VSNOp0 = VSN->getOperand(0);
6848:       unsigned Index = VSN->getSplatIndex();
6849:       assert(Index < VT.getVectorNumElements() &&
6850:              "Splat index should be defined and in first operand");
6851:       if ((Index == 0 && VSNOp0.getOpcode() == ISD::SCALAR_TO_VECTOR) ||
6852:           VSNOp0.getOpcode() == ISD::BUILD_VECTOR) {
6853:         // Since i32 is the smallest legal type, we either need a no-op
6854:         // or a truncation.
6855:         SDValue Shift = DAG.getNode(ISD::TRUNCATE, DL, MVT::i32,
6856:                                     VSNOp0.getOperand(Index));
6857:         return DAG.getNode(ByScalar, DL, VT, Op0, Shift);
6858:       }
6859:     }
6860:   }
6861: 
6862:   // Otherwise just treat the current form as legal.
6863:   return Op;
6864: }
6865: 
6866: SDValue SystemZTargetLowering::lowerFSHL(SDValue Op, SelectionDAG &DAG) const {
6867:   SDLoc DL(Op);
6868: 
6869:   // i128 FSHL with a constant amount that is a multiple of 8 can be
6870:   // implemented via VECTOR_SHUFFLE.  If we have the vector-enhancements-2
6871:   // facility, FSHL with a constant amount less than 8 can be implemented
6872:   // via SHL_DOUBLE_BIT, and FSHL with other constant amounts by a
6873:   // combination of the two.
6874:   if (auto *ShiftAmtNode = dyn_cast<ConstantSDNode>(Op.getOperand(2))) {
6875:     uint64_t ShiftAmt = ShiftAmtNode->getZExtValue() & 127;
6876:     if ((ShiftAmt & 7) == 0 || Subtarget.hasVectorEnhancements2()) {
6877:       SDValue Op0 = DAG.getBitcast(MVT::v16i8, Op.getOperand(0));
6878:       SDValue Op1 = DAG.getBitcast(MVT::v16i8, Op.getOperand(1));
6879:       if (ShiftAmt > 120) {
6880:         // For N in 121..128, fshl N == fshr (128 - N), and for 1 <= N < 8
6881:         // SHR_DOUBLE_BIT emits fewer instructions.
6882:         SDValue Val =
6883:             DAG.getNode(SystemZISD::SHR_DOUBLE_BIT, DL, MVT::v16i8, Op0, Op1,
6884:                         DAG.getTargetConstant(128 - ShiftAmt, DL, MVT::i32));
6885:         return DAG.getBitcast(MVT::i128, Val);
6886:       }
6887:       SmallVector<int, 16> Mask(16);
6888:       for (unsigned Elt = 0; Elt < 16; Elt++)
6889:         Mask[Elt] = (ShiftAmt >> 3) + Elt;
6890:       SDValue Shuf1 = DAG.getVectorShuffle(MVT::v16i8, DL, Op0, Op1, Mask);
6891:       if ((ShiftAmt & 7) == 0)
6892:         return DAG.getBitcast(MVT::i128, Shuf1);
6893:       SDValue Shuf2 = DAG.getVectorShuffle(MVT::v16i8, DL, Op1, Op1, Mask);
6894:       SDValue Val =
6895:           DAG.getNode(SystemZISD::SHL_DOUBLE_BIT, DL, MVT::v16i8, Shuf1, Shuf2,
6896:                       DAG.getTargetConstant(ShiftAmt & 7, DL, MVT::i32));
6897:       return DAG.getBitcast(MVT::i128, Val);
6898:     }
6899:   }
6900: 
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerFSHL`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerFSHL` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 6901-6975 / 第 6901-6975 行
```cpp
6901:   return SDValue();
6902: }
6903: 
6904: SDValue SystemZTargetLowering::lowerFSHR(SDValue Op, SelectionDAG &DAG) const {
6905:   SDLoc DL(Op);
6906: 
6907:   // i128 FSHR with a constant amount that is a multiple of 8 can be
6908:   // implemented via VECTOR_SHUFFLE.  If we have the vector-enhancements-2
6909:   // facility, FSHR with a constant amount less than 8 can be implemented
6910:   // via SHR_DOUBLE_BIT, and FSHR with other constant amounts by a
6911:   // combination of the two.
6912:   if (auto *ShiftAmtNode = dyn_cast<ConstantSDNode>(Op.getOperand(2))) {
6913:     uint64_t ShiftAmt = ShiftAmtNode->getZExtValue() & 127;
6914:     if ((ShiftAmt & 7) == 0 || Subtarget.hasVectorEnhancements2()) {
6915:       SDValue Op0 = DAG.getBitcast(MVT::v16i8, Op.getOperand(0));
6916:       SDValue Op1 = DAG.getBitcast(MVT::v16i8, Op.getOperand(1));
6917:       if (ShiftAmt > 120) {
6918:         // For N in 121..128, fshr N == fshl (128 - N), and for 1 <= N < 8
6919:         // SHL_DOUBLE_BIT emits fewer instructions.
6920:         SDValue Val =
6921:             DAG.getNode(SystemZISD::SHL_DOUBLE_BIT, DL, MVT::v16i8, Op0, Op1,
6922:                         DAG.getTargetConstant(128 - ShiftAmt, DL, MVT::i32));
6923:         return DAG.getBitcast(MVT::i128, Val);
6924:       }
6925:       SmallVector<int, 16> Mask(16);
6926:       for (unsigned Elt = 0; Elt < 16; Elt++)
6927:         Mask[Elt] = 16 - (ShiftAmt >> 3) + Elt;
6928:       SDValue Shuf1 = DAG.getVectorShuffle(MVT::v16i8, DL, Op0, Op1, Mask);
6929:       if ((ShiftAmt & 7) == 0)
6930:         return DAG.getBitcast(MVT::i128, Shuf1);
6931:       SDValue Shuf2 = DAG.getVectorShuffle(MVT::v16i8, DL, Op0, Op0, Mask);
6932:       SDValue Val =
6933:           DAG.getNode(SystemZISD::SHR_DOUBLE_BIT, DL, MVT::v16i8, Shuf2, Shuf1,
6934:                       DAG.getTargetConstant(ShiftAmt & 7, DL, MVT::i32));
6935:       return DAG.getBitcast(MVT::i128, Val);
6936:     }
6937:   }
6938: 
6939:   return SDValue();
6940: }
6941: 
6942: static SDValue lowerAddrSpaceCast(SDValue Op, SelectionDAG &DAG) {
6943:   SDLoc DL(Op);
6944:   SDValue Src = Op.getOperand(0);
6945:   MVT DstVT = Op.getSimpleValueType();
6946: 
6947:   AddrSpaceCastSDNode *N = cast<AddrSpaceCastSDNode>(Op.getNode());
6948:   unsigned SrcAS = N->getSrcAddressSpace();
6949: 
6950:   assert(SrcAS != N->getDestAddressSpace() &&
6951:          "addrspacecast must be between different address spaces");
6952: 
6953:   // addrspacecast [0 <- 1] : Assinging a ptr32 value to a 64-bit pointer.
6954:   // addrspacecast [1 <- 0] : Assigining a 64-bit pointer to a ptr32 value.
6955:   if (SrcAS == SYSTEMZAS::PTR32 && DstVT == MVT::i64) {
6956:     Op = DAG.getNode(ISD::AND, DL, MVT::i32, Src,
6957:                      DAG.getConstant(0x7fffffff, DL, MVT::i32));
6958:     Op = DAG.getNode(ISD::ZERO_EXTEND, DL, DstVT, Op);
6959:   } else if (DstVT == MVT::i32) {
6960:     Op = DAG.getNode(ISD::TRUNCATE, DL, DstVT, Src);
6961:     Op = DAG.getNode(ISD::AND, DL, MVT::i32, Op,
6962:                      DAG.getConstant(0x7fffffff, DL, MVT::i32));
6963:     Op = DAG.getNode(ISD::ZERO_EXTEND, DL, DstVT, Op);
6964:   } else {
6965:     report_fatal_error("Bad address space in addrspacecast");
6966:   }
6967:   return Op;
6968: }
6969: 
6970: SDValue SystemZTargetLowering::lowerFP_EXTEND(SDValue Op,
6971:                                               SelectionDAG &DAG) const {
6972:   SDValue In = Op.getOperand(Op->isStrictFPOpcode() ? 1 : 0);
6973:   if (In.getSimpleValueType() != MVT::f16)
6974:     return Op;      // Legal
6975:   return SDValue(); // Let legalizer emit the libcall.
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerFSHR`, `lowerAddrSpaceCast`, `SystemZTargetLowering::lowerFP_EXTEND`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerFSHR`, `lowerAddrSpaceCast`, `SystemZTargetLowering::lowerFP_EXTEND` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 6976-7050 / 第 6976-7050 行
```cpp
6976: }
6977: 
6978: SDValue SystemZTargetLowering::useLibCall(SelectionDAG &DAG, RTLIB::Libcall LC,
6979:                                           MVT VT, SDValue Arg, SDLoc DL,
6980:                                           SDValue Chain, bool IsStrict) const {
6981:   assert(LC != RTLIB::UNKNOWN_LIBCALL && "Unexpected request for libcall!");
6982:   MakeLibCallOptions CallOptions;
6983:   SDValue Result;
6984:   std::tie(Result, Chain) =
6985:       makeLibCall(DAG, LC, VT, Arg, CallOptions, DL, Chain);
6986:   return IsStrict ? DAG.getMergeValues({Result, Chain}, DL) : Result;
6987: }
6988: 
6989: SDValue SystemZTargetLowering::lower_FP_TO_INT(SDValue Op,
6990:                                                SelectionDAG &DAG) const {
6991:   bool IsSigned = (Op->getOpcode() == ISD::FP_TO_SINT ||
6992:                    Op->getOpcode() == ISD::STRICT_FP_TO_SINT);
6993:   bool IsStrict = Op->isStrictFPOpcode();
6994:   SDLoc DL(Op);
6995:   MVT VT = Op.getSimpleValueType();
6996:   SDValue InOp = Op.getOperand(IsStrict ? 1 : 0);
6997:   SDValue Chain = IsStrict ? Op.getOperand(0) : DAG.getEntryNode();
6998:   EVT InVT = InOp.getValueType();
6999: 
7000:   // FP to unsigned is not directly supported on z10.  Promoting an i32
7001:   // result to (signed) i64 doesn't generate an inexact condition (fp
7002:   // exception) for values that are outside the i32 range but in the i64
7003:   // range, so use the default expansion.
7004:   if (!Subtarget.hasFPExtension() && !IsSigned)
7005:     // Expand i32/i64. F16 values will be recognized to fit and extended.
7006:     return SDValue();
7007: 
7008:   // Conversion from f16 is done via f32.
7009:   if (InOp.getSimpleValueType() == MVT::f16) {
7010:     SmallVector<SDValue, 2> Results;
7011:     LowerOperationWrapper(Op.getNode(), Results, DAG);
7012:     return DAG.getMergeValues(Results, DL);
7013:   }
7014: 
7015:   if (VT == MVT::i128) {
7016:     RTLIB::Libcall LC =
7017:         IsSigned ? RTLIB::getFPTOSINT(InVT, VT) : RTLIB::getFPTOUINT(InVT, VT);
7018:     return useLibCall(DAG, LC, VT, InOp, DL, Chain, IsStrict);
7019:   }
7020: 
7021:   return Op; // Legal
7022: }
7023: 
7024: SDValue SystemZTargetLowering::lower_INT_TO_FP(SDValue Op,
7025:                                                SelectionDAG &DAG) const {
7026:   bool IsSigned = (Op->getOpcode() == ISD::SINT_TO_FP ||
7027:                    Op->getOpcode() == ISD::STRICT_SINT_TO_FP);
7028:   bool IsStrict = Op->isStrictFPOpcode();
7029:   SDLoc DL(Op);
7030:   MVT VT = Op.getSimpleValueType();
7031:   SDValue InOp = Op.getOperand(IsStrict ? 1 : 0);
7032:   SDValue Chain = IsStrict ? Op.getOperand(0) : DAG.getEntryNode();
7033:   EVT InVT = InOp.getValueType();
7034: 
7035:   // Conversion to f16 is done via f32.
7036:   if (VT == MVT::f16) {
7037:     SmallVector<SDValue, 2> Results;
7038:     LowerOperationWrapper(Op.getNode(), Results, DAG);
7039:     return DAG.getMergeValues(Results, DL);
7040:   }
7041: 
7042:   // Unsigned to fp is not directly supported on z10.
7043:   if (!Subtarget.hasFPExtension() && !IsSigned)
7044:     return SDValue(); // Expand i64.
7045: 
7046:   if (InVT == MVT::i128) {
7047:     RTLIB::Libcall LC =
7048:         IsSigned ? RTLIB::getSINTTOFP(InVT, VT) : RTLIB::getUINTTOFP(InVT, VT);
7049:     return useLibCall(DAG, LC, VT, InOp, DL, Chain, IsStrict);
7050:   }
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::useLibCall`, `SystemZTargetLowering::lower_FP_TO_INT`, `SystemZTargetLowering::lower_INT_TO_FP`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::useLibCall`, `SystemZTargetLowering::lower_FP_TO_INT`, `SystemZTargetLowering::lower_INT_TO_FP` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 7051-7125 / 第 7051-7125 行
```cpp
7051: 
7052:   return Op; // Legal
7053: }
7054: 
7055: // Lower an f16 LOAD in case of no vector support.
7056: SDValue SystemZTargetLowering::lowerLoadF16(SDValue Op,
7057:                                             SelectionDAG &DAG) const {
7058:   EVT RegVT = Op.getValueType();
7059:   assert(RegVT == MVT::f16 && "Expected to lower an f16 load.");
7060:   (void)RegVT;
7061: 
7062:   // Load as integer.
7063:   SDLoc DL(Op);
7064:   SDValue NewLd;
7065:   if (auto *AtomicLd = dyn_cast<AtomicSDNode>(Op.getNode())) {
7066:     assert(EVT(RegVT) == AtomicLd->getMemoryVT() && "Unhandled f16 load");
7067:     NewLd = DAG.getAtomicLoad(ISD::EXTLOAD, DL, MVT::i16, MVT::i64,
7068:                               AtomicLd->getChain(), AtomicLd->getBasePtr(),
7069:                               AtomicLd->getMemOperand());
7070:   } else {
7071:     LoadSDNode *Ld = cast<LoadSDNode>(Op.getNode());
7072:     assert(EVT(RegVT) == Ld->getMemoryVT() && "Unhandled f16 load");
7073:     NewLd = DAG.getExtLoad(ISD::EXTLOAD, DL, MVT::i64, Ld->getChain(),
7074:                            Ld->getBasePtr(), Ld->getPointerInfo(), MVT::i16,
7075:                            Ld->getBaseAlign(), Ld->getMemOperand()->getFlags());
7076:   }
7077:   SDValue F16Val = convertToF16(NewLd, DAG);
7078:   return DAG.getMergeValues({F16Val, NewLd.getValue(1)}, DL);
7079: }
7080: 
7081: // Lower an f16 STORE in case of no vector support.
7082: SDValue SystemZTargetLowering::lowerStoreF16(SDValue Op,
7083:                                              SelectionDAG &DAG) const {
7084:   SDLoc DL(Op);
7085:   SDValue Shft = convertFromF16(Op->getOperand(1), DL, DAG);
7086: 
7087:   if (auto *AtomicSt = dyn_cast<AtomicSDNode>(Op.getNode()))
7088:     return DAG.getAtomic(ISD::ATOMIC_STORE, DL, MVT::i16, AtomicSt->getChain(),
7089:                          Shft, AtomicSt->getBasePtr(),
7090:                          AtomicSt->getMemOperand());
7091: 
7092:   StoreSDNode *St = cast<StoreSDNode>(Op.getNode());
7093:   return DAG.getTruncStore(St->getChain(), DL, Shft, St->getBasePtr(), MVT::i16,
7094:                            St->getMemOperand());
7095: }
7096: 
7097: SDValue SystemZTargetLowering::lowerIS_FPCLASS(SDValue Op,
7098:                                                SelectionDAG &DAG) const {
7099:   SDLoc DL(Op);
7100:   MVT ResultVT = Op.getSimpleValueType();
7101:   SDValue Arg = Op.getOperand(0);
7102:   unsigned Check = Op.getConstantOperandVal(1);
7103: 
7104:   unsigned TDCMask = 0;
7105:   if (Check & fcSNan)
7106:     TDCMask |= SystemZ::TDCMASK_SNAN_PLUS | SystemZ::TDCMASK_SNAN_MINUS;
7107:   if (Check & fcQNan)
7108:     TDCMask |= SystemZ::TDCMASK_QNAN_PLUS | SystemZ::TDCMASK_QNAN_MINUS;
7109:   if (Check & fcPosInf)
7110:     TDCMask |= SystemZ::TDCMASK_INFINITY_PLUS;
7111:   if (Check & fcNegInf)
7112:     TDCMask |= SystemZ::TDCMASK_INFINITY_MINUS;
7113:   if (Check & fcPosNormal)
7114:     TDCMask |= SystemZ::TDCMASK_NORMAL_PLUS;
7115:   if (Check & fcNegNormal)
7116:     TDCMask |= SystemZ::TDCMASK_NORMAL_MINUS;
7117:   if (Check & fcPosSubnormal)
7118:     TDCMask |= SystemZ::TDCMASK_SUBNORMAL_PLUS;
7119:   if (Check & fcNegSubnormal)
7120:     TDCMask |= SystemZ::TDCMASK_SUBNORMAL_MINUS;
7121:   if (Check & fcPosZero)
7122:     TDCMask |= SystemZ::TDCMASK_ZERO_PLUS;
7123:   if (Check & fcNegZero)
7124:     TDCMask |= SystemZ::TDCMASK_ZERO_MINUS;
7125:   SDValue TDCMaskV = DAG.getConstant(TDCMask, DL, MVT::i64);
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerLoadF16`, `SystemZTargetLowering::lowerStoreF16`, `SystemZTargetLowering::lowerIS_FPCLASS`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerLoadF16`, `SystemZTargetLowering::lowerStoreF16`, `SystemZTargetLowering::lowerIS_FPCLASS` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 7126-7200 / 第 7126-7200 行
```cpp
7126: 
7127:   SDValue Intr = DAG.getNode(SystemZISD::TDC, DL, ResultVT, Arg, TDCMaskV);
7128:   return getCCResult(DAG, Intr);
7129: }
7130: 
7131: SDValue SystemZTargetLowering::lowerREADCYCLECOUNTER(SDValue Op,
7132:                                                      SelectionDAG &DAG) const {
7133:   SDLoc DL(Op);
7134:   SDValue Chain = Op.getOperand(0);
7135: 
7136:   // STCKF only supports a memory operand, so we have to use a temporary.
7137:   SDValue StackPtr = DAG.CreateStackTemporary(MVT::i64);
7138:   int SPFI = cast<FrameIndexSDNode>(StackPtr.getNode())->getIndex();
7139:   MachinePointerInfo MPI =
7140:     MachinePointerInfo::getFixedStack(DAG.getMachineFunction(), SPFI);
7141: 
7142:   // Use STCFK to store the TOD clock into the temporary.
7143:   SDValue StoreOps[] = {Chain, StackPtr};
7144:   Chain = DAG.getMemIntrinsicNode(
7145:     SystemZISD::STCKF, DL, DAG.getVTList(MVT::Other), StoreOps, MVT::i64,
7146:     MPI, MaybeAlign(), MachineMemOperand::MOStore);
7147: 
7148:   // And read it back from there.
7149:   return DAG.getLoad(MVT::i64, DL, Chain, StackPtr, MPI);
7150: }
7151: 
7152: SDValue SystemZTargetLowering::LowerOperation(SDValue Op,
7153:                                               SelectionDAG &DAG) const {
7154:   switch (Op.getOpcode()) {
7155:   case ISD::FRAMEADDR:
7156:     return lowerFRAMEADDR(Op, DAG);
7157:   case ISD::RETURNADDR:
7158:     return lowerRETURNADDR(Op, DAG);
7159:   case ISD::BR_CC:
7160:     return lowerBR_CC(Op, DAG);
7161:   case ISD::SELECT_CC:
7162:     return lowerSELECT_CC(Op, DAG);
7163:   case ISD::SETCC:
7164:     return lowerSETCC(Op, DAG);
7165:   case ISD::STRICT_FSETCC:
7166:     return lowerSTRICT_FSETCC(Op, DAG, false);
7167:   case ISD::STRICT_FSETCCS:
7168:     return lowerSTRICT_FSETCC(Op, DAG, true);
7169:   case ISD::GlobalAddress:
7170:     return lowerGlobalAddress(cast<GlobalAddressSDNode>(Op), DAG);
7171:   case ISD::GlobalTLSAddress:
7172:     return lowerGlobalTLSAddress(cast<GlobalAddressSDNode>(Op), DAG);
7173:   case ISD::BlockAddress:
7174:     return lowerBlockAddress(cast<BlockAddressSDNode>(Op), DAG);
7175:   case ISD::JumpTable:
7176:     return lowerJumpTable(cast<JumpTableSDNode>(Op), DAG);
7177:   case ISD::ConstantPool:
7178:     return lowerConstantPool(cast<ConstantPoolSDNode>(Op), DAG);
7179:   case ISD::BITCAST:
7180:     return lowerBITCAST(Op, DAG);
7181:   case ISD::VASTART:
7182:     return lowerVASTART(Op, DAG);
7183:   case ISD::VACOPY:
7184:     return lowerVACOPY(Op, DAG);
7185:   case ISD::DYNAMIC_STACKALLOC:
7186:     return lowerDYNAMIC_STACKALLOC(Op, DAG);
7187:   case ISD::GET_DYNAMIC_AREA_OFFSET:
7188:     return lowerGET_DYNAMIC_AREA_OFFSET(Op, DAG);
7189:   case ISD::MULHS:
7190:     return lowerMULH(Op, DAG, SystemZISD::SMUL_LOHI);
7191:   case ISD::MULHU:
7192:     return lowerMULH(Op, DAG, SystemZISD::UMUL_LOHI);
7193:   case ISD::SMUL_LOHI:
7194:     return lowerSMUL_LOHI(Op, DAG);
7195:   case ISD::UMUL_LOHI:
7196:     return lowerUMUL_LOHI(Op, DAG);
7197:   case ISD::SDIVREM:
7198:     return lowerSDIVREM(Op, DAG);
7199:   case ISD::UDIVREM:
7200:     return lowerUDIVREM(Op, DAG);
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerREADCYCLECOUNTER`, `SystemZTargetLowering::LowerOperation`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerREADCYCLECOUNTER`, `SystemZTargetLowering::LowerOperation` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 7201-7275 / 第 7201-7275 行
```cpp
7201:   case ISD::SADDO:
7202:   case ISD::SSUBO:
7203:   case ISD::UADDO:
7204:   case ISD::USUBO:
7205:     return lowerXALUO(Op, DAG);
7206:   case ISD::UADDO_CARRY:
7207:   case ISD::USUBO_CARRY:
7208:     return lowerUADDSUBO_CARRY(Op, DAG);
7209:   case ISD::OR:
7210:     return lowerOR(Op, DAG);
7211:   case ISD::CTPOP:
7212:     return lowerCTPOP(Op, DAG);
7213:   case ISD::VECREDUCE_ADD:
7214:     return lowerVECREDUCE_ADD(Op, DAG);
7215:   case ISD::ATOMIC_FENCE:
7216:     return lowerATOMIC_FENCE(Op, DAG);
7217:   case ISD::ATOMIC_SWAP:
7218:     return lowerATOMIC_LOAD_OP(Op, DAG, SystemZISD::ATOMIC_SWAPW);
7219:   case ISD::ATOMIC_STORE:
7220:     return lowerATOMIC_STORE(Op, DAG);
7221:   case ISD::ATOMIC_LOAD:
7222:     return lowerATOMIC_LOAD(Op, DAG);
7223:   case ISD::ATOMIC_LOAD_ADD:
7224:     return lowerATOMIC_LOAD_OP(Op, DAG, SystemZISD::ATOMIC_LOADW_ADD);
7225:   case ISD::ATOMIC_LOAD_SUB:
7226:     return lowerATOMIC_LOAD_SUB(Op, DAG);
7227:   case ISD::ATOMIC_LOAD_AND:
7228:     return lowerATOMIC_LOAD_OP(Op, DAG, SystemZISD::ATOMIC_LOADW_AND);
7229:   case ISD::ATOMIC_LOAD_OR:
7230:     return lowerATOMIC_LOAD_OP(Op, DAG, SystemZISD::ATOMIC_LOADW_OR);
7231:   case ISD::ATOMIC_LOAD_XOR:
7232:     return lowerATOMIC_LOAD_OP(Op, DAG, SystemZISD::ATOMIC_LOADW_XOR);
7233:   case ISD::ATOMIC_LOAD_NAND:
7234:     return lowerATOMIC_LOAD_OP(Op, DAG, SystemZISD::ATOMIC_LOADW_NAND);
7235:   case ISD::ATOMIC_LOAD_MIN:
7236:     return lowerATOMIC_LOAD_OP(Op, DAG, SystemZISD::ATOMIC_LOADW_MIN);
7237:   case ISD::ATOMIC_LOAD_MAX:
7238:     return lowerATOMIC_LOAD_OP(Op, DAG, SystemZISD::ATOMIC_LOADW_MAX);
7239:   case ISD::ATOMIC_LOAD_UMIN:
7240:     return lowerATOMIC_LOAD_OP(Op, DAG, SystemZISD::ATOMIC_LOADW_UMIN);
7241:   case ISD::ATOMIC_LOAD_UMAX:
7242:     return lowerATOMIC_LOAD_OP(Op, DAG, SystemZISD::ATOMIC_LOADW_UMAX);
7243:   case ISD::ATOMIC_CMP_SWAP_WITH_SUCCESS:
7244:     return lowerATOMIC_CMP_SWAP(Op, DAG);
7245:   case ISD::STACKSAVE:
7246:     return lowerSTACKSAVE(Op, DAG);
7247:   case ISD::STACKRESTORE:
7248:     return lowerSTACKRESTORE(Op, DAG);
7249:   case ISD::PREFETCH:
7250:     return lowerPREFETCH(Op, DAG);
7251:   case ISD::INTRINSIC_W_CHAIN:
7252:     return lowerINTRINSIC_W_CHAIN(Op, DAG);
7253:   case ISD::INTRINSIC_WO_CHAIN:
7254:     return lowerINTRINSIC_WO_CHAIN(Op, DAG);
7255:   case ISD::BUILD_VECTOR:
7256:     return lowerBUILD_VECTOR(Op, DAG);
7257:   case ISD::VECTOR_SHUFFLE:
7258:     return lowerVECTOR_SHUFFLE(Op, DAG);
7259:   case ISD::SCALAR_TO_VECTOR:
7260:     return lowerSCALAR_TO_VECTOR(Op, DAG);
7261:   case ISD::INSERT_VECTOR_ELT:
7262:     return lowerINSERT_VECTOR_ELT(Op, DAG);
7263:   case ISD::EXTRACT_VECTOR_ELT:
7264:     return lowerEXTRACT_VECTOR_ELT(Op, DAG);
7265:   case ISD::SIGN_EXTEND_VECTOR_INREG:
7266:     return lowerSIGN_EXTEND_VECTOR_INREG(Op, DAG);
7267:   case ISD::ZERO_EXTEND_VECTOR_INREG:
7268:     return lowerZERO_EXTEND_VECTOR_INREG(Op, DAG);
7269:   case ISD::SHL:
7270:     return lowerShift(Op, DAG, SystemZISD::VSHL_BY_SCALAR);
7271:   case ISD::SRL:
7272:     return lowerShift(Op, DAG, SystemZISD::VSRL_BY_SCALAR);
7273:   case ISD::SRA:
7274:     return lowerShift(Op, DAG, SystemZISD::VSRA_BY_SCALAR);
7275:   case ISD::ADDRSPACECAST:
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 7276-7350 / 第 7276-7350 行
```cpp
7276:     return lowerAddrSpaceCast(Op, DAG);
7277:   case ISD::ROTL:
7278:     return lowerShift(Op, DAG, SystemZISD::VROTL_BY_SCALAR);
7279:   case ISD::FSHL:
7280:     return lowerFSHL(Op, DAG);
7281:   case ISD::FSHR:
7282:     return lowerFSHR(Op, DAG);
7283:   case ISD::FP_EXTEND:
7284:   case ISD::STRICT_FP_EXTEND:
7285:     return lowerFP_EXTEND(Op, DAG);
7286:   case ISD::FP_TO_UINT:
7287:   case ISD::FP_TO_SINT:
7288:   case ISD::STRICT_FP_TO_UINT:
7289:   case ISD::STRICT_FP_TO_SINT:
7290:     return lower_FP_TO_INT(Op, DAG);
7291:   case ISD::UINT_TO_FP:
7292:   case ISD::SINT_TO_FP:
7293:   case ISD::STRICT_UINT_TO_FP:
7294:   case ISD::STRICT_SINT_TO_FP:
7295:     return lower_INT_TO_FP(Op, DAG);
7296:   case ISD::LOAD:
7297:     return lowerLoadF16(Op, DAG);
7298:   case ISD::STORE:
7299:     return lowerStoreF16(Op, DAG);
7300:   case ISD::IS_FPCLASS:
7301:     return lowerIS_FPCLASS(Op, DAG);
7302:   case ISD::GET_ROUNDING:
7303:     return lowerGET_ROUNDING(Op, DAG);
7304:   case ISD::READCYCLECOUNTER:
7305:     return lowerREADCYCLECOUNTER(Op, DAG);
7306:   case ISD::EH_SJLJ_SETJMP:
7307:   case ISD::EH_SJLJ_LONGJMP:
7308:     // These operations are legal on our platform, but we cannot actually
7309:     // set the operation action to Legal as common code would treat this
7310:     // as equivalent to Expand. Instead, we keep the operation action to
7311:     // Custom and just leave them unchanged here.
7312:     return Op;
7313: 
7314:   default:
7315:     llvm_unreachable("Unexpected node to lower");
7316:   }
7317: }
7318: 
7319: static SDValue expandBitCastI128ToF128(SelectionDAG &DAG, SDValue Src,
7320:                                        const SDLoc &SL) {
7321:   // If i128 is legal, just use a normal bitcast.
7322:   if (DAG.getTargetLoweringInfo().isTypeLegal(MVT::i128))
7323:     return DAG.getBitcast(MVT::f128, Src);
7324: 
7325:   // Otherwise, f128 must live in FP128, so do a partwise move.
7326:   assert(DAG.getTargetLoweringInfo().getRepRegClassFor(MVT::f128) ==
7327:          &SystemZ::FP128BitRegClass);
7328: 
7329:   SDValue Hi, Lo;
7330:   std::tie(Lo, Hi) = DAG.SplitScalar(Src, SL, MVT::i64, MVT::i64);
7331: 
7332:   Hi = DAG.getBitcast(MVT::f64, Hi);
7333:   Lo = DAG.getBitcast(MVT::f64, Lo);
7334: 
7335:   SDNode *Pair = DAG.getMachineNode(
7336:       SystemZ::REG_SEQUENCE, SL, MVT::f128,
7337:       {DAG.getTargetConstant(SystemZ::FP128BitRegClassID, SL, MVT::i32), Lo,
7338:        DAG.getTargetConstant(SystemZ::subreg_l64, SL, MVT::i32), Hi,
7339:        DAG.getTargetConstant(SystemZ::subreg_h64, SL, MVT::i32)});
7340:   return SDValue(Pair, 0);
7341: }
7342: 
7343: static SDValue expandBitCastF128ToI128(SelectionDAG &DAG, SDValue Src,
7344:                                        const SDLoc &SL) {
7345:   // If i128 is legal, just use a normal bitcast.
7346:   if (DAG.getTargetLoweringInfo().isTypeLegal(MVT::i128))
7347:     return DAG.getBitcast(MVT::i128, Src);
7348: 
7349:   // Otherwise, f128 must live in FP128, so do a partwise move.
7350:   assert(DAG.getTargetLoweringInfo().getRepRegClassFor(MVT::f128) ==
```
- **EN**: The range implements or declares functions including `expandBitCastI128ToF128`, `expandBitCastF128ToI128`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `expandBitCastI128ToF128`, `expandBitCastF128ToI128` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 7351-7425 / 第 7351-7425 行
```cpp
7351:          &SystemZ::FP128BitRegClass);
7352: 
7353:   SDValue LoFP =
7354:       DAG.getTargetExtractSubreg(SystemZ::subreg_l64, SL, MVT::f64, Src);
7355:   SDValue HiFP =
7356:       DAG.getTargetExtractSubreg(SystemZ::subreg_h64, SL, MVT::f64, Src);
7357:   SDValue Lo = DAG.getNode(ISD::BITCAST, SL, MVT::i64, LoFP);
7358:   SDValue Hi = DAG.getNode(ISD::BITCAST, SL, MVT::i64, HiFP);
7359: 
7360:   return DAG.getNode(ISD::BUILD_PAIR, SL, MVT::i128, Lo, Hi);
7361: }
7362: 
7363: // Lower operations with invalid operand or result types.
7364: void
7365: SystemZTargetLowering::LowerOperationWrapper(SDNode *N,
7366:                                              SmallVectorImpl<SDValue> &Results,
7367:                                              SelectionDAG &DAG) const {
7368:   switch (N->getOpcode()) {
7369:   case ISD::ATOMIC_LOAD: {
7370:     SDLoc DL(N);
7371:     SDVTList Tys = DAG.getVTList(MVT::Untyped, MVT::Other);
7372:     SDValue Ops[] = { N->getOperand(0), N->getOperand(1) };
7373:     MachineMemOperand *MMO = cast<AtomicSDNode>(N)->getMemOperand();
7374:     SDValue Res = DAG.getMemIntrinsicNode(SystemZISD::ATOMIC_LOAD_128,
7375:                                           DL, Tys, Ops, MVT::i128, MMO);
7376: 
7377:     SDValue Lowered = lowerGR128ToI128(DAG, Res);
7378:     if (N->getValueType(0) == MVT::f128)
7379:       Lowered = expandBitCastI128ToF128(DAG, Lowered, DL);
7380:     Results.push_back(Lowered);
7381:     Results.push_back(Res.getValue(1));
7382:     break;
7383:   }
7384:   case ISD::ATOMIC_STORE: {
7385:     SDLoc DL(N);
7386:     SDVTList Tys = DAG.getVTList(MVT::Other);
7387:     SDValue Val = N->getOperand(1);
7388:     if (Val.getValueType() == MVT::f128)
7389:       Val = expandBitCastF128ToI128(DAG, Val, DL);
7390:     Val = lowerI128ToGR128(DAG, Val);
7391: 
7392:     SDValue Ops[] = {N->getOperand(0), Val, N->getOperand(2)};
7393:     MachineMemOperand *MMO = cast<AtomicSDNode>(N)->getMemOperand();
7394:     SDValue Res = DAG.getMemIntrinsicNode(SystemZISD::ATOMIC_STORE_128,
7395:                                           DL, Tys, Ops, MVT::i128, MMO);
7396:     // We have to enforce sequential consistency by performing a
7397:     // serialization operation after the store.
7398:     if (cast<AtomicSDNode>(N)->getSuccessOrdering() ==
7399:         AtomicOrdering::SequentiallyConsistent)
7400:       Res = SDValue(DAG.getMachineNode(SystemZ::Serialize, DL,
7401:                                        MVT::Other, Res), 0);
7402:     Results.push_back(Res);
7403:     break;
7404:   }
7405:   case ISD::ATOMIC_CMP_SWAP_WITH_SUCCESS: {
7406:     SDLoc DL(N);
7407:     SDVTList Tys = DAG.getVTList(MVT::Untyped, MVT::i32, MVT::Other);
7408:     SDValue Ops[] = { N->getOperand(0), N->getOperand(1),
7409:                       lowerI128ToGR128(DAG, N->getOperand(2)),
7410:                       lowerI128ToGR128(DAG, N->getOperand(3)) };
7411:     MachineMemOperand *MMO = cast<AtomicSDNode>(N)->getMemOperand();
7412:     SDValue Res = DAG.getMemIntrinsicNode(SystemZISD::ATOMIC_CMP_SWAP_128,
7413:                                           DL, Tys, Ops, MVT::i128, MMO);
7414:     SDValue Success = emitSETCC(DAG, DL, Res.getValue(1),
7415:                                 SystemZ::CCMASK_CS, SystemZ::CCMASK_CS_EQ);
7416:     Success = DAG.getZExtOrTrunc(Success, DL, N->getValueType(1));
7417:     Results.push_back(lowerGR128ToI128(DAG, Res));
7418:     Results.push_back(Success);
7419:     Results.push_back(Res.getValue(2));
7420:     break;
7421:   }
7422:   case ISD::BITCAST: {
7423:     if (useSoftFloat())
7424:       return;
7425:     SDLoc DL(N);
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::LowerOperationWrapper`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::LowerOperationWrapper` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 7426-7500 / 第 7426-7500 行
```cpp
7426:     SDValue Src = N->getOperand(0);
7427:     EVT SrcVT = Src.getValueType();
7428:     EVT ResVT = N->getValueType(0);
7429:     if (ResVT == MVT::i128 && SrcVT == MVT::f128)
7430:       Results.push_back(expandBitCastF128ToI128(DAG, Src, DL));
7431:     else if (SrcVT == MVT::i16 && ResVT == MVT::f16) {
7432:       if (Subtarget.hasVector()) {
7433:         SDValue In32 = DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i32, Src);
7434:         Results.push_back(SDValue(
7435:             DAG.getMachineNode(SystemZ::LEFR_16, DL, MVT::f16, In32), 0));
7436:       } else {
7437:         SDValue In64 = DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, Src);
7438:         Results.push_back(convertToF16(In64, DAG));
7439:       }
7440:     } else if (SrcVT == MVT::f16 && ResVT == MVT::i16) {
7441:       SDValue ExtractedVal =
7442:           Subtarget.hasVector()
7443:               ? SDValue(DAG.getMachineNode(SystemZ::LFER_16, DL, MVT::i32, Src),
7444:                         0)
7445:               : convertFromF16(Src, DL, DAG);
7446:       Results.push_back(DAG.getZExtOrTrunc(ExtractedVal, DL, ResVT));
7447:     }
7448:     break;
7449:   }
7450:   case ISD::UINT_TO_FP:
7451:   case ISD::SINT_TO_FP:
7452:   case ISD::STRICT_UINT_TO_FP:
7453:   case ISD::STRICT_SINT_TO_FP: {
7454:     if (useSoftFloat())
7455:       return;
7456:     bool IsStrict = N->isStrictFPOpcode();
7457:     SDLoc DL(N);
7458:     SDValue InOp = N->getOperand(IsStrict ? 1 : 0);
7459:     EVT ResVT = N->getValueType(0);
7460:     SDValue Chain = IsStrict ? N->getOperand(0) : DAG.getEntryNode();
7461:     if (ResVT == MVT::f16) {
7462:       if (!IsStrict) {
7463:         SDValue OpF32 = DAG.getNode(N->getOpcode(), DL, MVT::f32, InOp);
7464:         Results.push_back(DAG.getFPExtendOrRound(OpF32, DL, MVT::f16));
7465:       } else {
7466:         SDValue OpF32 =
7467:             DAG.getNode(N->getOpcode(), DL, DAG.getVTList(MVT::f32, MVT::Other),
7468:                         {Chain, InOp});
7469:         SDValue F16Res;
7470:         std::tie(F16Res, Chain) = DAG.getStrictFPExtendOrRound(
7471:             OpF32, OpF32.getValue(1), DL, MVT::f16);
7472:         Results.push_back(F16Res);
7473:         Results.push_back(Chain);
7474:       }
7475:     }
7476:     break;
7477:   }
7478:   case ISD::FP_TO_UINT:
7479:   case ISD::FP_TO_SINT:
7480:   case ISD::STRICT_FP_TO_UINT:
7481:   case ISD::STRICT_FP_TO_SINT: {
7482:     if (useSoftFloat())
7483:       return;
7484:     bool IsStrict = N->isStrictFPOpcode();
7485:     SDLoc DL(N);
7486:     EVT ResVT = N->getValueType(0);
7487:     SDValue InOp = N->getOperand(IsStrict ? 1 : 0);
7488:     EVT InVT = InOp->getValueType(0);
7489:     SDValue Chain = IsStrict ? N->getOperand(0) : DAG.getEntryNode();
7490:     if (InVT == MVT::f16) {
7491:       if (!IsStrict) {
7492:         SDValue InF32 = DAG.getFPExtendOrRound(InOp, DL, MVT::f32);
7493:         Results.push_back(DAG.getNode(N->getOpcode(), DL, ResVT, InF32));
7494:       } else {
7495:         SDValue InF32;
7496:         std::tie(InF32, Chain) =
7497:             DAG.getStrictFPExtendOrRound(InOp, Chain, DL, MVT::f32);
7498:         SDValue OpF32 =
7499:             DAG.getNode(N->getOpcode(), DL, DAG.getVTList(ResVT, MVT::Other),
7500:                         {Chain, InF32});
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 7501-7575 / 第 7501-7575 行
```cpp
7501:         Results.push_back(OpF32);
7502:         Results.push_back(OpF32.getValue(1));
7503:       }
7504:     }
7505:     break;
7506:   }
7507:   default:
7508:     llvm_unreachable("Unexpected node to lower");
7509:   }
7510: }
7511: 
7512: void
7513: SystemZTargetLowering::ReplaceNodeResults(SDNode *N,
7514:                                           SmallVectorImpl<SDValue> &Results,
7515:                                           SelectionDAG &DAG) const {
7516:   return LowerOperationWrapper(N, Results, DAG);
7517: }
7518: 
7519: // Return true if VT is a vector whose elements are a whole number of bytes
7520: // in width. Also check for presence of vector support.
7521: bool SystemZTargetLowering::canTreatAsByteVector(EVT VT) const {
7522:   if (!Subtarget.hasVector())
7523:     return false;
7524: 
7525:   return VT.isVector() && VT.getScalarSizeInBits() % 8 == 0 && VT.isSimple();
7526: }
7527: 
7528: // Try to simplify an EXTRACT_VECTOR_ELT from a vector of type VecVT
7529: // producing a result of type ResVT.  Op is a possibly bitcast version
7530: // of the input vector and Index is the index (based on type VecVT) that
7531: // should be extracted.  Return the new extraction if a simplification
7532: // was possible or if Force is true.
7533: SDValue SystemZTargetLowering::combineExtract(const SDLoc &DL, EVT ResVT,
7534:                                               EVT VecVT, SDValue Op,
7535:                                               unsigned Index,
7536:                                               DAGCombinerInfo &DCI,
7537:                                               bool Force) const {
7538:   SelectionDAG &DAG = DCI.DAG;
7539: 
7540:   // The number of bytes being extracted.
7541:   unsigned BytesPerElement = VecVT.getVectorElementType().getStoreSize();
7542: 
7543:   for (;;) {
7544:     unsigned Opcode = Op.getOpcode();
7545:     if (Opcode == ISD::BITCAST)
7546:       // Look through bitcasts.
7547:       Op = Op.getOperand(0);
7548:     else if ((Opcode == ISD::VECTOR_SHUFFLE || Opcode == SystemZISD::SPLAT) &&
7549:              canTreatAsByteVector(Op.getValueType())) {
7550:       // Get a VPERM-like permute mask and see whether the bytes covered
7551:       // by the extracted element are a contiguous sequence from one
7552:       // source operand.
7553:       SmallVector<int, SystemZ::VectorBytes> Bytes;
7554:       if (!getVPermMask(Op, Bytes))
7555:         break;
7556:       int First;
7557:       if (!getShuffleInput(Bytes, Index * BytesPerElement,
7558:                            BytesPerElement, First))
7559:         break;
7560:       if (First < 0)
7561:         return DAG.getUNDEF(ResVT);
7562:       // Make sure the contiguous sequence starts at a multiple of the
7563:       // original element size.
7564:       unsigned Byte = unsigned(First) % Bytes.size();
7565:       if (Byte % BytesPerElement != 0)
7566:         break;
7567:       // We can get the extracted value directly from an input.
7568:       Index = Byte / BytesPerElement;
7569:       Op = Op.getOperand(unsigned(First) / Bytes.size());
7570:       Force = true;
7571:     } else if (Opcode == ISD::BUILD_VECTOR &&
7572:                canTreatAsByteVector(Op.getValueType())) {
7573:       // We can only optimize this case if the BUILD_VECTOR elements are
7574:       // at least as wide as the extracted value.
7575:       EVT OpVT = Op.getValueType();
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::ReplaceNodeResults`, `SystemZTargetLowering::canTreatAsByteVector`, `SystemZTargetLowering::combineExtract`, `canTreatAsByteVector`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::ReplaceNodeResults`, `SystemZTargetLowering::canTreatAsByteVector`, `SystemZTargetLowering::combineExtract`, `canTreatAsByteVector` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 7576-7650 / 第 7576-7650 行
```cpp
7576:       unsigned OpBytesPerElement = OpVT.getVectorElementType().getStoreSize();
7577:       if (OpBytesPerElement < BytesPerElement)
7578:         break;
7579:       // Make sure that the least-significant bit of the extracted value
7580:       // is the least significant bit of an input.
7581:       unsigned End = (Index + 1) * BytesPerElement;
7582:       if (End % OpBytesPerElement != 0)
7583:         break;
7584:       // We're extracting the low part of one operand of the BUILD_VECTOR.
7585:       Op = Op.getOperand(End / OpBytesPerElement - 1);
7586:       if (!Op.getValueType().isInteger()) {
7587:         EVT VT = MVT::getIntegerVT(Op.getValueSizeInBits());
7588:         Op = DAG.getNode(ISD::BITCAST, DL, VT, Op);
7589:         DCI.AddToWorklist(Op.getNode());
7590:       }
7591:       EVT VT = MVT::getIntegerVT(ResVT.getSizeInBits());
7592:       Op = DAG.getNode(ISD::TRUNCATE, DL, VT, Op);
7593:       if (VT != ResVT) {
7594:         DCI.AddToWorklist(Op.getNode());
7595:         Op = DAG.getNode(ISD::BITCAST, DL, ResVT, Op);
7596:       }
7597:       return Op;
7598:     } else if ((Opcode == ISD::SIGN_EXTEND_VECTOR_INREG ||
7599:                 Opcode == ISD::ZERO_EXTEND_VECTOR_INREG ||
7600:                 Opcode == ISD::ANY_EXTEND_VECTOR_INREG) &&
7601:                canTreatAsByteVector(Op.getValueType()) &&
7602:                canTreatAsByteVector(Op.getOperand(0).getValueType())) {
7603:       // Make sure that only the unextended bits are significant.
7604:       EVT ExtVT = Op.getValueType();
7605:       EVT OpVT = Op.getOperand(0).getValueType();
7606:       unsigned ExtBytesPerElement = ExtVT.getVectorElementType().getStoreSize();
7607:       unsigned OpBytesPerElement = OpVT.getVectorElementType().getStoreSize();
7608:       unsigned Byte = Index * BytesPerElement;
7609:       unsigned SubByte = Byte % ExtBytesPerElement;
7610:       unsigned MinSubByte = ExtBytesPerElement - OpBytesPerElement;
7611:       if (SubByte < MinSubByte ||
7612:           SubByte + BytesPerElement > ExtBytesPerElement)
7613:         break;
7614:       // Get the byte offset of the unextended element
7615:       Byte = Byte / ExtBytesPerElement * OpBytesPerElement;
7616:       // ...then add the byte offset relative to that element.
7617:       Byte += SubByte - MinSubByte;
7618:       if (Byte % BytesPerElement != 0)
7619:         break;
7620:       Op = Op.getOperand(0);
7621:       Index = Byte / BytesPerElement;
7622:       Force = true;
7623:     } else
7624:       break;
7625:   }
7626:   if (Force) {
7627:     if (Op.getValueType() != VecVT) {
7628:       Op = DAG.getNode(ISD::BITCAST, DL, VecVT, Op);
7629:       DCI.AddToWorklist(Op.getNode());
7630:     }
7631:     return DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, ResVT, Op,
7632:                        DAG.getConstant(Index, DL, MVT::i32));
7633:   }
7634:   return SDValue();
7635: }
7636: 
7637: // Optimize vector operations in scalar value Op on the basis that Op
7638: // is truncated to TruncVT.
7639: SDValue SystemZTargetLowering::combineTruncateExtract(
7640:     const SDLoc &DL, EVT TruncVT, SDValue Op, DAGCombinerInfo &DCI) const {
7641:   // If we have (trunc (extract_vector_elt X, Y)), try to turn it into
7642:   // (extract_vector_elt (bitcast X), Y'), where (bitcast X) has elements
7643:   // of type TruncVT.
7644:   if (Op.getOpcode() == ISD::EXTRACT_VECTOR_ELT &&
7645:       TruncVT.getSizeInBits() % 8 == 0) {
7646:     SDValue Vec = Op.getOperand(0);
7647:     EVT VecVT = Vec.getValueType();
7648:     if (canTreatAsByteVector(VecVT)) {
7649:       if (auto *IndexN = dyn_cast<ConstantSDNode>(Op.getOperand(1))) {
7650:         unsigned BytesPerElement = VecVT.getVectorElementType().getStoreSize();
```
- **EN**: The range implements or declares functions including `canTreatAsByteVector`, `SystemZTargetLowering::combineTruncateExtract`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `canTreatAsByteVector`, `SystemZTargetLowering::combineTruncateExtract` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 7651-7725 / 第 7651-7725 行
```cpp
7651:         unsigned TruncBytes = TruncVT.getStoreSize();
7652:         if (BytesPerElement % TruncBytes == 0) {
7653:           // Calculate the value of Y' in the above description.  We are
7654:           // splitting the original elements into Scale equal-sized pieces
7655:           // and for truncation purposes want the last (least-significant)
7656:           // of these pieces for IndexN.  This is easiest to do by calculating
7657:           // the start index of the following element and then subtracting 1.
7658:           unsigned Scale = BytesPerElement / TruncBytes;
7659:           unsigned NewIndex = (IndexN->getZExtValue() + 1) * Scale - 1;
7660: 
7661:           // Defer the creation of the bitcast from X to combineExtract,
7662:           // which might be able to optimize the extraction.
7663:           VecVT = EVT::getVectorVT(*DCI.DAG.getContext(),
7664:                                    MVT::getIntegerVT(TruncBytes * 8),
7665:                                    VecVT.getStoreSize() / TruncBytes);
7666:           EVT ResVT = (TruncBytes < 4 ? MVT::i32 : TruncVT);
7667:           return combineExtract(DL, ResVT, VecVT, Vec, NewIndex, DCI, true);
7668:         }
7669:       }
7670:     }
7671:   }
7672:   return SDValue();
7673: }
7674: 
7675: SDValue SystemZTargetLowering::combineZERO_EXTEND(
7676:     SDNode *N, DAGCombinerInfo &DCI) const {
7677:   // Convert (zext (select_ccmask C1, C2)) into (select_ccmask C1', C2')
7678:   SelectionDAG &DAG = DCI.DAG;
7679:   SDValue N0 = N->getOperand(0);
7680:   EVT VT = N->getValueType(0);
7681:   if (N0.getOpcode() == SystemZISD::SELECT_CCMASK) {
7682:     auto *TrueOp = dyn_cast<ConstantSDNode>(N0.getOperand(0));
7683:     auto *FalseOp = dyn_cast<ConstantSDNode>(N0.getOperand(1));
7684:     if (TrueOp && FalseOp) {
7685:       SDLoc DL(N0);
7686:       SDValue Ops[] = { DAG.getConstant(TrueOp->getZExtValue(), DL, VT),
7687:                         DAG.getConstant(FalseOp->getZExtValue(), DL, VT),
7688:                         N0.getOperand(2), N0.getOperand(3), N0.getOperand(4) };
7689:       SDValue NewSelect = DAG.getNode(SystemZISD::SELECT_CCMASK, DL, VT, Ops);
7690:       // If N0 has multiple uses, change other uses as well.
7691:       if (!N0.hasOneUse()) {
7692:         SDValue TruncSelect =
7693:           DAG.getNode(ISD::TRUNCATE, DL, N0.getValueType(), NewSelect);
7694:         DCI.CombineTo(N0.getNode(), TruncSelect);
7695:       }
7696:       return NewSelect;
7697:     }
7698:   }
7699:   // Convert (zext (xor (trunc X), C)) into (xor (trunc X), C') if the size
7700:   // of the result is smaller than the size of X and all the truncated bits
7701:   // of X are already zero.
7702:   if (N0.getOpcode() == ISD::XOR &&
7703:       N0.hasOneUse() && N0.getOperand(0).hasOneUse() &&
7704:       N0.getOperand(0).getOpcode() == ISD::TRUNCATE &&
7705:       N0.getOperand(1).getOpcode() == ISD::Constant) {
7706:     SDValue X = N0.getOperand(0).getOperand(0);
7707:     if (VT.isScalarInteger() && VT.getSizeInBits() < X.getValueSizeInBits()) {
7708:       KnownBits Known = DAG.computeKnownBits(X);
7709:       APInt TruncatedBits = APInt::getBitsSet(X.getValueSizeInBits(),
7710:                                               N0.getValueSizeInBits(),
7711:                                               VT.getSizeInBits());
7712:       if (TruncatedBits.isSubsetOf(Known.Zero)) {
7713:         X = DAG.getNode(ISD::TRUNCATE, SDLoc(X), VT, X);
7714:         APInt Mask = N0.getConstantOperandAPInt(1).zext(VT.getSizeInBits());
7715:         return DAG.getNode(ISD::XOR, SDLoc(N0), VT,
7716:                            X, DAG.getConstant(Mask, SDLoc(N0), VT));
7717:       }
7718:     }
7719:   }
7720:   // Recognize patterns for VECTOR SUBTRACT COMPUTE BORROW INDICATION
7721:   // and VECTOR ADD COMPUTE CARRY for i128:
7722:   //   (zext (setcc_uge X Y)) --> (VSCBI X Y)
7723:   //   (zext (setcc_ule Y X)) --> (VSCBI X Y)
7724:   //   (zext (setcc_ult (add X Y) X/Y) -> (VACC X Y)
7725:   //   (zext (setcc_ugt X/Y (add X Y)) -> (VACC X Y)
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::combineZERO_EXTEND`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::combineZERO_EXTEND` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 7726-7800 / 第 7726-7800 行
```cpp
7726:   // For vector types, these patterns are recognized in the .td file.
7727:   if (N0.getOpcode() == ISD::SETCC && isTypeLegal(VT) && VT == MVT::i128 &&
7728:       N0.getOperand(0).getValueType() == VT) {
7729:     SDValue Op0 = N0.getOperand(0);
7730:     SDValue Op1 = N0.getOperand(1);
7731:     const ISD::CondCode CC = cast<CondCodeSDNode>(N0.getOperand(2))->get();
7732:     switch (CC) {
7733:     case ISD::SETULE:
7734:       std::swap(Op0, Op1);
7735:       [[fallthrough]];
7736:     case ISD::SETUGE:
7737:       return DAG.getNode(SystemZISD::VSCBI, SDLoc(N0), VT, Op0, Op1);
7738:     case ISD::SETUGT:
7739:       std::swap(Op0, Op1);
7740:       [[fallthrough]];
7741:     case ISD::SETULT:
7742:       if (Op0->hasOneUse() && Op0->getOpcode() == ISD::ADD &&
7743:           (Op0->getOperand(0) == Op1 || Op0->getOperand(1) == Op1))
7744:         return DAG.getNode(SystemZISD::VACC, SDLoc(N0), VT, Op0->getOperand(0),
7745:                            Op0->getOperand(1));
7746:       break;
7747:     default:
7748:       break;
7749:     }
7750:   }
7751: 
7752:   return SDValue();
7753: }
7754: 
7755: SDValue SystemZTargetLowering::combineSIGN_EXTEND_INREG(
7756:     SDNode *N, DAGCombinerInfo &DCI) const {
7757:   // Convert (sext_in_reg (setcc LHS, RHS, COND), i1)
7758:   // and (sext_in_reg (any_extend (setcc LHS, RHS, COND)), i1)
7759:   // into (select_cc LHS, RHS, -1, 0, COND)
7760:   SelectionDAG &DAG = DCI.DAG;
7761:   SDValue N0 = N->getOperand(0);
7762:   EVT VT = N->getValueType(0);
7763:   EVT EVT = cast<VTSDNode>(N->getOperand(1))->getVT();
7764:   if (N0.hasOneUse() && N0.getOpcode() == ISD::ANY_EXTEND)
7765:     N0 = N0.getOperand(0);
7766:   if (EVT == MVT::i1 && N0.hasOneUse() && N0.getOpcode() == ISD::SETCC) {
7767:     SDLoc DL(N0);
7768:     SDValue Ops[] = { N0.getOperand(0), N0.getOperand(1),
7769:                       DAG.getAllOnesConstant(DL, VT),
7770:                       DAG.getConstant(0, DL, VT), N0.getOperand(2) };
7771:     return DAG.getNode(ISD::SELECT_CC, DL, VT, Ops);
7772:   }
7773:   return SDValue();
7774: }
7775: 
7776: SDValue SystemZTargetLowering::combineSIGN_EXTEND(
7777:     SDNode *N, DAGCombinerInfo &DCI) const {
7778:   // Convert (sext (ashr (shl X, C1), C2)) to
7779:   // (ashr (shl (anyext X), C1'), C2')), since wider shifts are as
7780:   // cheap as narrower ones.
7781:   SelectionDAG &DAG = DCI.DAG;
7782:   SDValue N0 = N->getOperand(0);
7783:   EVT VT = N->getValueType(0);
7784:   if (N0.hasOneUse() && N0.getOpcode() == ISD::SRA) {
7785:     auto *SraAmt = dyn_cast<ConstantSDNode>(N0.getOperand(1));
7786:     SDValue Inner = N0.getOperand(0);
7787:     if (SraAmt && Inner.hasOneUse() && Inner.getOpcode() == ISD::SHL) {
7788:       if (auto *ShlAmt = dyn_cast<ConstantSDNode>(Inner.getOperand(1))) {
7789:         unsigned Extra = (VT.getSizeInBits() - N0.getValueSizeInBits());
7790:         unsigned NewShlAmt = ShlAmt->getZExtValue() + Extra;
7791:         unsigned NewSraAmt = SraAmt->getZExtValue() + Extra;
7792:         EVT ShiftVT = N0.getOperand(1).getValueType();
7793:         SDValue Ext = DAG.getNode(ISD::ANY_EXTEND, SDLoc(Inner), VT,
7794:                                   Inner.getOperand(0));
7795:         SDValue Shl = DAG.getNode(ISD::SHL, SDLoc(Inner), VT, Ext,
7796:                                   DAG.getConstant(NewShlAmt, SDLoc(Inner),
7797:                                                   ShiftVT));
7798:         return DAG.getNode(ISD::SRA, SDLoc(N0), VT, Shl,
7799:                            DAG.getConstant(NewSraAmt, SDLoc(N0), ShiftVT));
7800:       }
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::combineSIGN_EXTEND_INREG`, `SystemZTargetLowering::combineSIGN_EXTEND`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::combineSIGN_EXTEND_INREG`, `SystemZTargetLowering::combineSIGN_EXTEND` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 7801-7875 / 第 7801-7875 行
```cpp
7801:     }
7802:   }
7803: 
7804:   return SDValue();
7805: }
7806: 
7807: SDValue SystemZTargetLowering::combineMERGE(
7808:     SDNode *N, DAGCombinerInfo &DCI) const {
7809:   SelectionDAG &DAG = DCI.DAG;
7810:   unsigned Opcode = N->getOpcode();
7811:   SDValue Op0 = N->getOperand(0);
7812:   SDValue Op1 = N->getOperand(1);
7813:   if (Op0.getOpcode() == ISD::BITCAST)
7814:     Op0 = Op0.getOperand(0);
7815:   if (ISD::isBuildVectorAllZeros(Op0.getNode())) {
7816:     // (z_merge_* 0, 0) -> 0.  This is mostly useful for using VLLEZF
7817:     // for v4f32.
7818:     if (Op1 == N->getOperand(0))
7819:       return Op1;
7820:     // (z_merge_? 0, X) -> (z_unpackl_? 0, X).
7821:     EVT VT = Op1.getValueType();
7822:     unsigned ElemBytes = VT.getVectorElementType().getStoreSize();
7823:     if (ElemBytes <= 4) {
7824:       Opcode = (Opcode == SystemZISD::MERGE_HIGH ?
7825:                 SystemZISD::UNPACKL_HIGH : SystemZISD::UNPACKL_LOW);
7826:       EVT InVT = VT.changeVectorElementTypeToInteger();
7827:       EVT OutVT = MVT::getVectorVT(MVT::getIntegerVT(ElemBytes * 16),
7828:                                    SystemZ::VectorBytes / ElemBytes / 2);
7829:       if (VT != InVT) {
7830:         Op1 = DAG.getNode(ISD::BITCAST, SDLoc(N), InVT, Op1);
7831:         DCI.AddToWorklist(Op1.getNode());
7832:       }
7833:       SDValue Op = DAG.getNode(Opcode, SDLoc(N), OutVT, Op1);
7834:       DCI.AddToWorklist(Op.getNode());
7835:       return DAG.getNode(ISD::BITCAST, SDLoc(N), VT, Op);
7836:     }
7837:   }
7838:   return SDValue();
7839: }
7840: 
7841: static bool isI128MovedToParts(LoadSDNode *LD, SDNode *&LoPart,
7842:                                SDNode *&HiPart) {
7843:   LoPart = HiPart = nullptr;
7844: 
7845:   // Scan through all users.
7846:   for (SDUse &Use : LD->uses()) {
7847:     // Skip the uses of the chain.
7848:     if (Use.getResNo() != 0)
7849:       continue;
7850: 
7851:     // Verify every user is a TRUNCATE to i64 of the low or high half.
7852:     SDNode *User = Use.getUser();
7853:     bool IsLoPart = true;
7854:     if (User->getOpcode() == ISD::SRL &&
7855:         User->getOperand(1).getOpcode() == ISD::Constant &&
7856:         User->getConstantOperandVal(1) == 64 && User->hasOneUse()) {
7857:       User = *User->user_begin();
7858:       IsLoPart = false;
7859:     }
7860:     if (User->getOpcode() != ISD::TRUNCATE || User->getValueType(0) != MVT::i64)
7861:       return false;
7862: 
7863:     if (IsLoPart) {
7864:       if (LoPart)
7865:         return false;
7866:       LoPart = User;
7867:     } else {
7868:       if (HiPart)
7869:         return false;
7870:       HiPart = User;
7871:     }
7872:   }
7873:   return true;
7874: }
7875: 
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::combineMERGE`, `isI128MovedToParts`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::combineMERGE`, `isI128MovedToParts` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 7876-7950 / 第 7876-7950 行
```cpp
7876: static bool isF128MovedToParts(LoadSDNode *LD, SDNode *&LoPart,
7877:                                SDNode *&HiPart) {
7878:   LoPart = HiPart = nullptr;
7879: 
7880:   // Scan through all users.
7881:   for (SDUse &Use : LD->uses()) {
7882:     // Skip the uses of the chain.
7883:     if (Use.getResNo() != 0)
7884:       continue;
7885: 
7886:     // Verify every user is an EXTRACT_SUBREG of the low or high half.
7887:     SDNode *User = Use.getUser();
7888:     if (!User->hasOneUse() || !User->isMachineOpcode() ||
7889:         User->getMachineOpcode() != TargetOpcode::EXTRACT_SUBREG)
7890:       return false;
7891: 
7892:     switch (User->getConstantOperandVal(1)) {
7893:     case SystemZ::subreg_l64:
7894:       if (LoPart)
7895:         return false;
7896:       LoPart = User;
7897:       break;
7898:     case SystemZ::subreg_h64:
7899:       if (HiPart)
7900:         return false;
7901:       HiPart = User;
7902:       break;
7903:     default:
7904:       return false;
7905:     }
7906:   }
7907:   return true;
7908: }
7909: 
7910: SDValue SystemZTargetLowering::combineLOAD(
7911:     SDNode *N, DAGCombinerInfo &DCI) const {
7912:   SelectionDAG &DAG = DCI.DAG;
7913:   EVT LdVT = N->getValueType(0);
7914:   if (auto *LN = dyn_cast<LoadSDNode>(N)) {
7915:     if (LN->getAddressSpace() == SYSTEMZAS::PTR32) {
7916:       MVT PtrVT = getPointerTy(DAG.getDataLayout());
7917:       MVT LoadNodeVT = LN->getBasePtr().getSimpleValueType();
7918:       if (PtrVT != LoadNodeVT) {
7919:         SDLoc DL(LN);
7920:         SDValue AddrSpaceCast = DAG.getAddrSpaceCast(
7921:             DL, PtrVT, LN->getBasePtr(), SYSTEMZAS::PTR32, 0);
7922:         return DAG.getExtLoad(LN->getExtensionType(), DL, LN->getValueType(0),
7923:                               LN->getChain(), AddrSpaceCast, LN->getMemoryVT(),
7924:                               LN->getMemOperand());
7925:       }
7926:     }
7927:   }
7928:   SDLoc DL(N);
7929: 
7930:   // Replace a 128-bit load that is used solely to move its value into GPRs
7931:   // by separate loads of both halves.
7932:   LoadSDNode *LD = cast<LoadSDNode>(N);
7933:   if (LD->isSimple() && ISD::isNormalLoad(LD)) {
7934:     SDNode *LoPart, *HiPart;
7935:     if ((LdVT == MVT::i128 && isI128MovedToParts(LD, LoPart, HiPart)) ||
7936:         (LdVT == MVT::f128 && isF128MovedToParts(LD, LoPart, HiPart))) {
7937:       // Rewrite each extraction as an independent load.
7938:       SmallVector<SDValue, 2> ArgChains;
7939:       if (HiPart) {
7940:         SDValue EltLoad = DAG.getLoad(
7941:             HiPart->getValueType(0), DL, LD->getChain(), LD->getBasePtr(),
7942:             LD->getPointerInfo(), LD->getBaseAlign(),
7943:             LD->getMemOperand()->getFlags(), LD->getAAInfo());
7944: 
7945:         DCI.CombineTo(HiPart, EltLoad, true);
7946:         ArgChains.push_back(EltLoad.getValue(1));
7947:       }
7948:       if (LoPart) {
7949:         SDValue EltLoad = DAG.getLoad(
7950:             LoPart->getValueType(0), DL, LD->getChain(),
```
- **EN**: The range implements or declares functions including `isF128MovedToParts`, `SystemZTargetLowering::combineLOAD`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isF128MovedToParts`, `SystemZTargetLowering::combineLOAD` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 7951-8025 / 第 7951-8025 行
```cpp
7951:             DAG.getObjectPtrOffset(DL, LD->getBasePtr(), TypeSize::getFixed(8)),
7952:             LD->getPointerInfo().getWithOffset(8), LD->getBaseAlign(),
7953:             LD->getMemOperand()->getFlags(), LD->getAAInfo());
7954: 
7955:         DCI.CombineTo(LoPart, EltLoad, true);
7956:         ArgChains.push_back(EltLoad.getValue(1));
7957:       }
7958: 
7959:       // Collect all chains via TokenFactor.
7960:       SDValue Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, ArgChains);
7961:       DAG.ReplaceAllUsesOfValueWith(SDValue(N, 1), Chain);
7962:       DCI.AddToWorklist(Chain.getNode());
7963:       return SDValue(N, 0);
7964:     }
7965:   }
7966: 
7967:   if (LdVT.isVector() || LdVT.isInteger())
7968:     return SDValue();
7969:   // Transform a scalar load that is REPLICATEd as well as having other
7970:   // use(s) to the form where the other use(s) use the first element of the
7971:   // REPLICATE instead of the load. Otherwise instruction selection will not
7972:   // produce a VLREP. Avoid extracting to a GPR, so only do this for floating
7973:   // point loads.
7974: 
7975:   SDValue Replicate;
7976:   SmallVector<SDNode*, 8> OtherUses;
7977:   for (SDUse &Use : N->uses()) {
7978:     if (Use.getUser()->getOpcode() == SystemZISD::REPLICATE) {
7979:       if (Replicate)
7980:         return SDValue(); // Should never happen
7981:       Replicate = SDValue(Use.getUser(), 0);
7982:     } else if (Use.getResNo() == 0)
7983:       OtherUses.push_back(Use.getUser());
7984:   }
7985:   if (!Replicate || OtherUses.empty())
7986:     return SDValue();
7987: 
7988:   SDValue Extract0 = DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, LdVT,
7989:                               Replicate, DAG.getConstant(0, DL, MVT::i32));
7990:   // Update uses of the loaded Value while preserving old chains.
7991:   for (SDNode *U : OtherUses) {
7992:     SmallVector<SDValue, 8> Ops;
7993:     for (SDValue Op : U->ops())
7994:       Ops.push_back((Op.getNode() == N && Op.getResNo() == 0) ? Extract0 : Op);
7995:     DAG.UpdateNodeOperands(U, Ops);
7996:   }
7997:   return SDValue(N, 0);
7998: }
7999: 
8000: bool SystemZTargetLowering::canLoadStoreByteSwapped(EVT VT) const {
8001:   if (VT == MVT::i16 || VT == MVT::i32 || VT == MVT::i64)
8002:     return true;
8003:   if (Subtarget.hasVectorEnhancements2())
8004:     if (VT == MVT::v8i16 || VT == MVT::v4i32 || VT == MVT::v2i64 || VT == MVT::i128)
8005:       return true;
8006:   return false;
8007: }
8008: 
8009: static bool isVectorElementSwap(ArrayRef<int> M, EVT VT) {
8010:   if (!VT.isVector() || !VT.isSimple() ||
8011:       VT.getSizeInBits() != 128 ||
8012:       VT.getScalarSizeInBits() % 8 != 0)
8013:     return false;
8014: 
8015:   unsigned NumElts = VT.getVectorNumElements();
8016:   for (unsigned i = 0; i < NumElts; ++i) {
8017:     if (M[i] < 0) continue; // ignore UNDEF indices
8018:     if ((unsigned) M[i] != NumElts - 1 - i)
8019:       return false;
8020:   }
8021: 
8022:   return true;
8023: }
8024: 
8025: static bool isOnlyUsedByStores(SDValue StoredVal, SelectionDAG &DAG) {
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::canLoadStoreByteSwapped`, `isVectorElementSwap`, `isOnlyUsedByStores`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::canLoadStoreByteSwapped`, `isVectorElementSwap`, `isOnlyUsedByStores` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 8026-8100 / 第 8026-8100 行
```cpp
8026:   for (auto *U : StoredVal->users()) {
8027:     if (StoreSDNode *ST = dyn_cast<StoreSDNode>(U)) {
8028:       EVT CurrMemVT = ST->getMemoryVT().getScalarType();
8029:       if (CurrMemVT.isRound() && CurrMemVT.getStoreSize() <= 16)
8030:         continue;
8031:     } else if (isa<BuildVectorSDNode>(U)) {
8032:       SDValue BuildVector = SDValue(U, 0);
8033:       if (DAG.isSplatValue(BuildVector, true/*AllowUndefs*/) &&
8034:           isOnlyUsedByStores(BuildVector, DAG))
8035:         continue;
8036:     }
8037:     return false;
8038:   }
8039:   return true;
8040: }
8041: 
8042: static bool isI128MovedFromParts(SDValue Val, SDValue &LoPart,
8043:                                  SDValue &HiPart) {
8044:   if (Val.getOpcode() != ISD::OR || !Val.getNode()->hasOneUse())
8045:     return false;
8046: 
8047:   SDValue Op0 = Val.getOperand(0);
8048:   SDValue Op1 = Val.getOperand(1);
8049: 
8050:   if (Op0.getOpcode() == ISD::SHL)
8051:     std::swap(Op0, Op1);
8052:   if (Op1.getOpcode() != ISD::SHL || !Op1.getNode()->hasOneUse() ||
8053:       Op1.getOperand(1).getOpcode() != ISD::Constant ||
8054:       Op1.getConstantOperandVal(1) != 64)
8055:     return false;
8056:   Op1 = Op1.getOperand(0);
8057: 
8058:   if (Op0.getOpcode() != ISD::ZERO_EXTEND || !Op0.getNode()->hasOneUse() ||
8059:       Op0.getOperand(0).getValueType() != MVT::i64)
8060:     return false;
8061:   if (Op1.getOpcode() != ISD::ANY_EXTEND || !Op1.getNode()->hasOneUse() ||
8062:       Op1.getOperand(0).getValueType() != MVT::i64)
8063:     return false;
8064: 
8065:   LoPart = Op0.getOperand(0);
8066:   HiPart = Op1.getOperand(0);
8067:   return true;
8068: }
8069: 
8070: static bool isF128MovedFromParts(SDValue Val, SDValue &LoPart,
8071:                                  SDValue &HiPart) {
8072:   if (!Val.getNode()->hasOneUse() || !Val.isMachineOpcode() ||
8073:       Val.getMachineOpcode() != TargetOpcode::REG_SEQUENCE)
8074:     return false;
8075: 
8076:   if (Val->getNumOperands() != 5 ||
8077:       Val->getOperand(0)->getAsZExtVal() != SystemZ::FP128BitRegClassID ||
8078:       Val->getOperand(2)->getAsZExtVal() != SystemZ::subreg_l64 ||
8079:       Val->getOperand(4)->getAsZExtVal() != SystemZ::subreg_h64)
8080:     return false;
8081: 
8082:   LoPart = Val->getOperand(1);
8083:   HiPart = Val->getOperand(3);
8084:   return true;
8085: }
8086: 
8087: SDValue SystemZTargetLowering::combineSTORE(
8088:     SDNode *N, DAGCombinerInfo &DCI) const {
8089:   SelectionDAG &DAG = DCI.DAG;
8090:   auto *SN = cast<StoreSDNode>(N);
8091:   auto &Op1 = N->getOperand(1);
8092:   EVT MemVT = SN->getMemoryVT();
8093: 
8094:   if (SN->getAddressSpace() == SYSTEMZAS::PTR32) {
8095:     MVT PtrVT = getPointerTy(DAG.getDataLayout());
8096:     MVT StoreNodeVT = SN->getBasePtr().getSimpleValueType();
8097:     if (PtrVT != StoreNodeVT) {
8098:       SDLoc DL(SN);
8099:       SDValue AddrSpaceCast = DAG.getAddrSpaceCast(DL, PtrVT, SN->getBasePtr(),
8100:                                                    SYSTEMZAS::PTR32, 0);
```
- **EN**: The range implements or declares functions including `isI128MovedFromParts`, `isF128MovedFromParts`, `SystemZTargetLowering::combineSTORE`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `isI128MovedFromParts`, `isF128MovedFromParts`, `SystemZTargetLowering::combineSTORE` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 8101-8175 / 第 8101-8175 行
```cpp
8101:       return DAG.getStore(SN->getChain(), DL, SN->getValue(), AddrSpaceCast,
8102:                           SN->getPointerInfo(), SN->getBaseAlign(),
8103:                           SN->getMemOperand()->getFlags(), SN->getAAInfo());
8104:     }
8105:   }
8106: 
8107:   // If we have (truncstoreiN (extract_vector_elt X, Y), Z) then it is better
8108:   // for the extraction to be done on a vMiN value, so that we can use VSTE.
8109:   // If X has wider elements then convert it to:
8110:   // (truncstoreiN (extract_vector_elt (bitcast X), Y2), Z).
8111:   if (MemVT.isInteger() && SN->isTruncatingStore()) {
8112:     if (SDValue Value =
8113:             combineTruncateExtract(SDLoc(N), MemVT, SN->getValue(), DCI)) {
8114:       DCI.AddToWorklist(Value.getNode());
8115: 
8116:       // Rewrite the store with the new form of stored value.
8117:       return DAG.getTruncStore(SN->getChain(), SDLoc(SN), Value,
8118:                                SN->getBasePtr(), SN->getMemoryVT(),
8119:                                SN->getMemOperand());
8120:     }
8121:   }
8122:   // Combine STORE (BSWAP) into STRVH/STRV/STRVG/VSTBR
8123:   if (!SN->isTruncatingStore() &&
8124:       Op1.getOpcode() == ISD::BSWAP &&
8125:       Op1.getNode()->hasOneUse() &&
8126:       canLoadStoreByteSwapped(Op1.getValueType())) {
8127: 
8128:       SDValue BSwapOp = Op1.getOperand(0);
8129: 
8130:       if (BSwapOp.getValueType() == MVT::i16)
8131:         BSwapOp = DAG.getNode(ISD::ANY_EXTEND, SDLoc(N), MVT::i32, BSwapOp);
8132: 
8133:       SDValue Ops[] = {
8134:         N->getOperand(0), BSwapOp, N->getOperand(2)
8135:       };
8136: 
8137:       return
8138:         DAG.getMemIntrinsicNode(SystemZISD::STRV, SDLoc(N), DAG.getVTList(MVT::Other),
8139:                                 Ops, MemVT, SN->getMemOperand());
8140:     }
8141:   // Combine STORE (element-swap) into VSTER
8142:   if (!SN->isTruncatingStore() &&
8143:       Op1.getOpcode() == ISD::VECTOR_SHUFFLE &&
8144:       Op1.getNode()->hasOneUse() &&
8145:       Subtarget.hasVectorEnhancements2()) {
8146:     ShuffleVectorSDNode *SVN = cast<ShuffleVectorSDNode>(Op1.getNode());
8147:     ArrayRef<int> ShuffleMask = SVN->getMask();
8148:     if (isVectorElementSwap(ShuffleMask, Op1.getValueType())) {
8149:       SDValue Ops[] = {
8150:         N->getOperand(0), Op1.getOperand(0), N->getOperand(2)
8151:       };
8152: 
8153:       return DAG.getMemIntrinsicNode(SystemZISD::VSTER, SDLoc(N),
8154:                                      DAG.getVTList(MVT::Other),
8155:                                      Ops, MemVT, SN->getMemOperand());
8156:     }
8157:   }
8158: 
8159:   // Combine STORE (READCYCLECOUNTER) into STCKF.
8160:   if (!SN->isTruncatingStore() &&
8161:       Op1.getOpcode() == ISD::READCYCLECOUNTER &&
8162:       Op1.hasOneUse() &&
8163:       N->getOperand(0).reachesChainWithoutSideEffects(SDValue(Op1.getNode(), 1))) {
8164:       SDValue Ops[] = { Op1.getOperand(0), N->getOperand(2) };
8165:       return DAG.getMemIntrinsicNode(SystemZISD::STCKF, SDLoc(N),
8166:                                      DAG.getVTList(MVT::Other),
8167:                                      Ops, MemVT, SN->getMemOperand());
8168:   }
8169: 
8170:   // Transform a store of a 128-bit value moved from parts into two stores.
8171:   if (SN->isSimple() && ISD::isNormalStore(SN)) {
8172:     SDValue LoPart, HiPart;
8173:     if ((MemVT == MVT::i128 && isI128MovedFromParts(Op1, LoPart, HiPart)) ||
8174:         (MemVT == MVT::f128 && isF128MovedFromParts(Op1, LoPart, HiPart))) {
8175:       SDLoc DL(SN);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 8176-8250 / 第 8176-8250 行
```cpp
8176:       SDValue Chain0 = DAG.getStore(
8177:           SN->getChain(), DL, HiPart, SN->getBasePtr(), SN->getPointerInfo(),
8178:           SN->getBaseAlign(), SN->getMemOperand()->getFlags(), SN->getAAInfo());
8179:       SDValue Chain1 = DAG.getStore(
8180:           SN->getChain(), DL, LoPart,
8181:           DAG.getObjectPtrOffset(DL, SN->getBasePtr(), TypeSize::getFixed(8)),
8182:           SN->getPointerInfo().getWithOffset(8), SN->getBaseAlign(),
8183:           SN->getMemOperand()->getFlags(), SN->getAAInfo());
8184: 
8185:       return DAG.getNode(ISD::TokenFactor, DL, MVT::Other, Chain0, Chain1);
8186:     }
8187:   }
8188: 
8189:   // Replicate a reg or immediate with VREP instead of scalar multiply or
8190:   // immediate load. It seems best to do this during the first DAGCombine as
8191:   // it is straight-forward to handle the zero-extend node in the initial
8192:   // DAG, and also not worry about the keeping the new MemVT legal (e.g. when
8193:   // extracting an i16 element from a v16i8 vector).
8194:   if (Subtarget.hasVector() && DCI.Level == BeforeLegalizeTypes &&
8195:       isOnlyUsedByStores(Op1, DAG)) {
8196:     SDValue Word = SDValue();
8197:     EVT WordVT;
8198: 
8199:     // Find a replicated immediate and return it if found in Word and its
8200:     // type in WordVT.
8201:     auto FindReplicatedImm = [&](ConstantSDNode *C, unsigned TotBytes) {
8202:       // Some constants are better handled with a scalar store.
8203:       if (C->getAPIntValue().getBitWidth() > 64 || C->isAllOnes() ||
8204:           isInt<16>(C->getSExtValue()) || MemVT.getStoreSize() <= 2)
8205:         return;
8206: 
8207:       APInt Val = C->getAPIntValue();
8208:       // Truncate Val in case of a truncating store.
8209:       if (!llvm::isUIntN(TotBytes * 8, Val.getZExtValue())) {
8210:         assert(SN->isTruncatingStore() &&
8211:                "Non-truncating store and immediate value does not fit?");
8212:         Val = Val.trunc(TotBytes * 8);
8213:       }
8214: 
8215:       SystemZVectorConstantInfo VCI(APInt(TotBytes * 8, Val.getZExtValue()));
8216:       if (VCI.isVectorConstantLegal(Subtarget) &&
8217:           VCI.Opcode == SystemZISD::REPLICATE) {
8218:         Word = DAG.getConstant(VCI.OpVals[0], SDLoc(SN), MVT::i32);
8219:         WordVT = VCI.VecVT.getScalarType();
8220:       }
8221:     };
8222: 
8223:     // Find a replicated register and return it if found in Word and its type
8224:     // in WordVT.
8225:     auto FindReplicatedReg = [&](SDValue MulOp) {
8226:       EVT MulVT = MulOp.getValueType();
8227:       if (MulOp->getOpcode() == ISD::MUL &&
8228:           (MulVT == MVT::i16 || MulVT == MVT::i32 || MulVT == MVT::i64)) {
8229:         // Find a zero extended value and its type.
8230:         SDValue LHS = MulOp->getOperand(0);
8231:         if (LHS->getOpcode() == ISD::ZERO_EXTEND)
8232:           WordVT = LHS->getOperand(0).getValueType();
8233:         else if (LHS->getOpcode() == ISD::AssertZext)
8234:           WordVT = cast<VTSDNode>(LHS->getOperand(1))->getVT();
8235:         else
8236:           return;
8237:         // Find a replicating constant, e.g. 0x00010001.
8238:         if (auto *C = dyn_cast<ConstantSDNode>(MulOp->getOperand(1))) {
8239:           SystemZVectorConstantInfo VCI(
8240:               APInt(MulVT.getSizeInBits(), C->getZExtValue()));
8241:           if (VCI.isVectorConstantLegal(Subtarget) &&
8242:               VCI.Opcode == SystemZISD::REPLICATE && VCI.OpVals[0] == 1 &&
8243:               WordVT == VCI.VecVT.getScalarType())
8244:             Word = DAG.getZExtOrTrunc(LHS->getOperand(0), SDLoc(SN), WordVT);
8245:         }
8246:       }
8247:     };
8248: 
8249:     if (isa<BuildVectorSDNode>(Op1) &&
8250:         DAG.isSplatValue(Op1, true/*AllowUndefs*/)) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 8251-8325 / 第 8251-8325 行
```cpp
8251:       SDValue SplatVal = Op1->getOperand(0);
8252:       if (auto *C = dyn_cast<ConstantSDNode>(SplatVal))
8253:         FindReplicatedImm(C, SplatVal.getValueType().getStoreSize());
8254:       else
8255:         FindReplicatedReg(SplatVal);
8256:     } else {
8257:       if (auto *C = dyn_cast<ConstantSDNode>(Op1))
8258:         FindReplicatedImm(C, MemVT.getStoreSize());
8259:       else
8260:         FindReplicatedReg(Op1);
8261:     }
8262: 
8263:     if (Word != SDValue()) {
8264:       assert(MemVT.getSizeInBits() % WordVT.getSizeInBits() == 0 &&
8265:              "Bad type handling");
8266:       unsigned NumElts = MemVT.getSizeInBits() / WordVT.getSizeInBits();
8267:       EVT SplatVT = EVT::getVectorVT(*DAG.getContext(), WordVT, NumElts);
8268:       SDValue SplatVal = DAG.getSplatVector(SplatVT, SDLoc(SN), Word);
8269:       return DAG.getStore(SN->getChain(), SDLoc(SN), SplatVal,
8270:                           SN->getBasePtr(), SN->getMemOperand());
8271:     }
8272:   }
8273: 
8274:   return SDValue();
8275: }
8276: 
8277: SDValue SystemZTargetLowering::combineVECTOR_SHUFFLE(
8278:     SDNode *N, DAGCombinerInfo &DCI) const {
8279:   SelectionDAG &DAG = DCI.DAG;
8280:   // Combine element-swap (LOAD) into VLER
8281:   if (ISD::isNON_EXTLoad(N->getOperand(0).getNode()) &&
8282:       N->getOperand(0).hasOneUse() &&
8283:       Subtarget.hasVectorEnhancements2()) {
8284:     ShuffleVectorSDNode *SVN = cast<ShuffleVectorSDNode>(N);
8285:     ArrayRef<int> ShuffleMask = SVN->getMask();
8286:     if (isVectorElementSwap(ShuffleMask, N->getValueType(0))) {
8287:       SDValue Load = N->getOperand(0);
8288:       LoadSDNode *LD = cast<LoadSDNode>(Load);
8289: 
8290:       // Create the element-swapping load.
8291:       SDValue Ops[] = {
8292:         LD->getChain(),    // Chain
8293:         LD->getBasePtr()   // Ptr
8294:       };
8295:       SDValue ESLoad =
8296:         DAG.getMemIntrinsicNode(SystemZISD::VLER, SDLoc(N),
8297:                                 DAG.getVTList(LD->getValueType(0), MVT::Other),
8298:                                 Ops, LD->getMemoryVT(), LD->getMemOperand());
8299: 
8300:       // First, combine the VECTOR_SHUFFLE away.  This makes the value produced
8301:       // by the load dead.
8302:       DCI.CombineTo(N, ESLoad);
8303: 
8304:       // Next, combine the load away, we give it a bogus result value but a real
8305:       // chain result.  The result value is dead because the shuffle is dead.
8306:       DCI.CombineTo(Load.getNode(), ESLoad, ESLoad.getValue(1));
8307: 
8308:       // Return N so it doesn't get rechecked!
8309:       return SDValue(N, 0);
8310:     }
8311:   }
8312: 
8313:   return SDValue();
8314: }
8315: 
8316: SDValue SystemZTargetLowering::combineEXTRACT_VECTOR_ELT(
8317:     SDNode *N, DAGCombinerInfo &DCI) const {
8318:   SelectionDAG &DAG = DCI.DAG;
8319: 
8320:   if (!Subtarget.hasVector())
8321:     return SDValue();
8322: 
8323:   // Look through bitcasts that retain the number of vector elements.
8324:   SDValue Op = N->getOperand(0);
8325:   if (Op.getOpcode() == ISD::BITCAST &&
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::combineVECTOR_SHUFFLE`, `SystemZTargetLowering::combineEXTRACT_VECTOR_ELT`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::combineVECTOR_SHUFFLE`, `SystemZTargetLowering::combineEXTRACT_VECTOR_ELT` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 8326-8400 / 第 8326-8400 行
```cpp
8326:       Op.getValueType().isVector() &&
8327:       Op.getOperand(0).getValueType().isVector() &&
8328:       Op.getValueType().getVectorNumElements() ==
8329:       Op.getOperand(0).getValueType().getVectorNumElements())
8330:     Op = Op.getOperand(0);
8331: 
8332:   // Pull BSWAP out of a vector extraction.
8333:   if (Op.getOpcode() == ISD::BSWAP && Op.hasOneUse()) {
8334:     EVT VecVT = Op.getValueType();
8335:     EVT EltVT = VecVT.getVectorElementType();
8336:     Op = DAG.getNode(ISD::EXTRACT_VECTOR_ELT, SDLoc(N), EltVT,
8337:                      Op.getOperand(0), N->getOperand(1));
8338:     DCI.AddToWorklist(Op.getNode());
8339:     Op = DAG.getNode(ISD::BSWAP, SDLoc(N), EltVT, Op);
8340:     if (EltVT != N->getValueType(0)) {
8341:       DCI.AddToWorklist(Op.getNode());
8342:       Op = DAG.getNode(ISD::BITCAST, SDLoc(N), N->getValueType(0), Op);
8343:     }
8344:     return Op;
8345:   }
8346: 
8347:   // Try to simplify a vector extraction.
8348:   if (auto *IndexN = dyn_cast<ConstantSDNode>(N->getOperand(1))) {
8349:     SDValue Op0 = N->getOperand(0);
8350:     EVT VecVT = Op0.getValueType();
8351:     if (canTreatAsByteVector(VecVT))
8352:       return combineExtract(SDLoc(N), N->getValueType(0), VecVT, Op0,
8353:                             IndexN->getZExtValue(), DCI, false);
8354:   }
8355:   return SDValue();
8356: }
8357: 
8358: SDValue SystemZTargetLowering::combineJOIN_DWORDS(
8359:     SDNode *N, DAGCombinerInfo &DCI) const {
8360:   SelectionDAG &DAG = DCI.DAG;
8361:   // (join_dwords X, X) == (replicate X)
8362:   if (N->getOperand(0) == N->getOperand(1))
8363:     return DAG.getNode(SystemZISD::REPLICATE, SDLoc(N), N->getValueType(0),
8364:                        N->getOperand(0));
8365:   return SDValue();
8366: }
8367: 
8368: static SDValue MergeInputChains(SDNode *N1, SDNode *N2) {
8369:   SDValue Chain1 = N1->getOperand(0);
8370:   SDValue Chain2 = N2->getOperand(0);
8371: 
8372:   // Trivial case: both nodes take the same chain.
8373:   if (Chain1 == Chain2)
8374:     return Chain1;
8375: 
8376:   // FIXME - we could handle more complex cases via TokenFactor,
8377:   // assuming we can verify that this would not create a cycle.
8378:   return SDValue();
8379: }
8380: 
8381: SDValue SystemZTargetLowering::combineFP_ROUND(
8382:     SDNode *N, DAGCombinerInfo &DCI) const {
8383: 
8384:   if (!Subtarget.hasVector())
8385:     return SDValue();
8386: 
8387:   // (fpround (extract_vector_elt X 0))
8388:   // (fpround (extract_vector_elt X 1)) ->
8389:   // (extract_vector_elt (VROUND X) 0)
8390:   // (extract_vector_elt (VROUND X) 2)
8391:   //
8392:   // This is a special case since the target doesn't really support v2f32s.
8393:   unsigned OpNo = N->isStrictFPOpcode() ? 1 : 0;
8394:   SelectionDAG &DAG = DCI.DAG;
8395:   SDValue Op0 = N->getOperand(OpNo);
8396:   if (N->getValueType(0) == MVT::f32 && Op0.hasOneUse() &&
8397:       Op0.getOpcode() == ISD::EXTRACT_VECTOR_ELT &&
8398:       Op0.getOperand(0).getValueType() == MVT::v2f64 &&
8399:       Op0.getOperand(1).getOpcode() == ISD::Constant &&
8400:       Op0.getConstantOperandVal(1) == 0) {
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::combineJOIN_DWORDS`, `MergeInputChains`, `SystemZTargetLowering::combineFP_ROUND`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::combineJOIN_DWORDS`, `MergeInputChains`, `SystemZTargetLowering::combineFP_ROUND` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 8401-8475 / 第 8401-8475 行
```cpp
8401:     SDValue Vec = Op0.getOperand(0);
8402:     for (auto *U : Vec->users()) {
8403:       if (U != Op0.getNode() && U->hasOneUse() &&
8404:           U->getOpcode() == ISD::EXTRACT_VECTOR_ELT &&
8405:           U->getOperand(0) == Vec &&
8406:           U->getOperand(1).getOpcode() == ISD::Constant &&
8407:           U->getConstantOperandVal(1) == 1) {
8408:         SDValue OtherRound = SDValue(*U->user_begin(), 0);
8409:         if (OtherRound.getOpcode() == N->getOpcode() &&
8410:             OtherRound.getOperand(OpNo) == SDValue(U, 0) &&
8411:             OtherRound.getValueType() == MVT::f32) {
8412:           SDValue VRound, Chain;
8413:           if (N->isStrictFPOpcode()) {
8414:             Chain = MergeInputChains(N, OtherRound.getNode());
8415:             if (!Chain)
8416:               continue;
8417:             VRound = DAG.getNode(SystemZISD::STRICT_VROUND, SDLoc(N),
8418:                                  {MVT::v4f32, MVT::Other}, {Chain, Vec});
8419:             Chain = VRound.getValue(1);
8420:           } else
8421:             VRound = DAG.getNode(SystemZISD::VROUND, SDLoc(N),
8422:                                  MVT::v4f32, Vec);
8423:           DCI.AddToWorklist(VRound.getNode());
8424:           SDValue Extract1 =
8425:             DAG.getNode(ISD::EXTRACT_VECTOR_ELT, SDLoc(U), MVT::f32,
8426:                         VRound, DAG.getConstant(2, SDLoc(U), MVT::i32));
8427:           DCI.AddToWorklist(Extract1.getNode());
8428:           DAG.ReplaceAllUsesOfValueWith(OtherRound, Extract1);
8429:           if (Chain)
8430:             DAG.ReplaceAllUsesOfValueWith(OtherRound.getValue(1), Chain);
8431:           SDValue Extract0 =
8432:             DAG.getNode(ISD::EXTRACT_VECTOR_ELT, SDLoc(Op0), MVT::f32,
8433:                         VRound, DAG.getConstant(0, SDLoc(Op0), MVT::i32));
8434:           if (Chain)
8435:             return DAG.getNode(ISD::MERGE_VALUES, SDLoc(Op0),
8436:                                N->getVTList(), Extract0, Chain);
8437:           return Extract0;
8438:         }
8439:       }
8440:     }
8441:   }
8442:   return SDValue();
8443: }
8444: 
8445: SDValue SystemZTargetLowering::combineFP_EXTEND(
8446:     SDNode *N, DAGCombinerInfo &DCI) const {
8447: 
8448:   if (!Subtarget.hasVector())
8449:     return SDValue();
8450: 
8451:   // (fpextend (extract_vector_elt X 0))
8452:   // (fpextend (extract_vector_elt X 2)) ->
8453:   // (extract_vector_elt (VEXTEND X) 0)
8454:   // (extract_vector_elt (VEXTEND X) 1)
8455:   //
8456:   // This is a special case since the target doesn't really support v2f32s.
8457:   unsigned OpNo = N->isStrictFPOpcode() ? 1 : 0;
8458:   SelectionDAG &DAG = DCI.DAG;
8459:   SDValue Op0 = N->getOperand(OpNo);
8460:   if (N->getValueType(0) == MVT::f64 && Op0.hasOneUse() &&
8461:       Op0.getOpcode() == ISD::EXTRACT_VECTOR_ELT &&
8462:       Op0.getOperand(0).getValueType() == MVT::v4f32 &&
8463:       Op0.getOperand(1).getOpcode() == ISD::Constant &&
8464:       Op0.getConstantOperandVal(1) == 0) {
8465:     SDValue Vec = Op0.getOperand(0);
8466:     for (auto *U : Vec->users()) {
8467:       if (U != Op0.getNode() && U->hasOneUse() &&
8468:           U->getOpcode() == ISD::EXTRACT_VECTOR_ELT &&
8469:           U->getOperand(0) == Vec &&
8470:           U->getOperand(1).getOpcode() == ISD::Constant &&
8471:           U->getConstantOperandVal(1) == 2) {
8472:         SDValue OtherExtend = SDValue(*U->user_begin(), 0);
8473:         if (OtherExtend.getOpcode() == N->getOpcode() &&
8474:             OtherExtend.getOperand(OpNo) == SDValue(U, 0) &&
8475:             OtherExtend.getValueType() == MVT::f64) {
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::combineFP_EXTEND`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::combineFP_EXTEND` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 8476-8550 / 第 8476-8550 行
```cpp
8476:           SDValue VExtend, Chain;
8477:           if (N->isStrictFPOpcode()) {
8478:             Chain = MergeInputChains(N, OtherExtend.getNode());
8479:             if (!Chain)
8480:               continue;
8481:             VExtend = DAG.getNode(SystemZISD::STRICT_VEXTEND, SDLoc(N),
8482:                                   {MVT::v2f64, MVT::Other}, {Chain, Vec});
8483:             Chain = VExtend.getValue(1);
8484:           } else
8485:             VExtend = DAG.getNode(SystemZISD::VEXTEND, SDLoc(N),
8486:                                   MVT::v2f64, Vec);
8487:           DCI.AddToWorklist(VExtend.getNode());
8488:           SDValue Extract1 =
8489:             DAG.getNode(ISD::EXTRACT_VECTOR_ELT, SDLoc(U), MVT::f64,
8490:                         VExtend, DAG.getConstant(1, SDLoc(U), MVT::i32));
8491:           DCI.AddToWorklist(Extract1.getNode());
8492:           DAG.ReplaceAllUsesOfValueWith(OtherExtend, Extract1);
8493:           if (Chain)
8494:             DAG.ReplaceAllUsesOfValueWith(OtherExtend.getValue(1), Chain);
8495:           SDValue Extract0 =
8496:             DAG.getNode(ISD::EXTRACT_VECTOR_ELT, SDLoc(Op0), MVT::f64,
8497:                         VExtend, DAG.getConstant(0, SDLoc(Op0), MVT::i32));
8498:           if (Chain)
8499:             return DAG.getNode(ISD::MERGE_VALUES, SDLoc(Op0),
8500:                                N->getVTList(), Extract0, Chain);
8501:           return Extract0;
8502:         }
8503:       }
8504:     }
8505:   }
8506:   return SDValue();
8507: }
8508: 
8509: SDValue SystemZTargetLowering::combineINT_TO_FP(
8510:     SDNode *N, DAGCombinerInfo &DCI) const {
8511:   if (DCI.Level != BeforeLegalizeTypes)
8512:     return SDValue();
8513:   SelectionDAG &DAG = DCI.DAG;
8514:   LLVMContext &Ctx = *DAG.getContext();
8515:   unsigned Opcode = N->getOpcode();
8516:   EVT OutVT = N->getValueType(0);
8517:   Type *OutLLVMTy = OutVT.getTypeForEVT(Ctx);
8518:   SDValue Op = N->getOperand(0);
8519:   unsigned OutScalarBits = OutLLVMTy->getScalarSizeInBits();
8520:   unsigned InScalarBits = Op->getValueType(0).getScalarSizeInBits();
8521: 
8522:   // Insert an extension before type-legalization to avoid scalarization, e.g.:
8523:   // v2f64 = uint_to_fp v2i16
8524:   // =>
8525:   // v2f64 = uint_to_fp (v2i64 zero_extend v2i16)
8526:   if (OutLLVMTy->isVectorTy() && OutScalarBits > InScalarBits &&
8527:       OutScalarBits <= 64) {
8528:     unsigned NumElts = cast<FixedVectorType>(OutLLVMTy)->getNumElements();
8529:     EVT ExtVT = EVT::getVectorVT(
8530:         Ctx, EVT::getIntegerVT(Ctx, OutLLVMTy->getScalarSizeInBits()), NumElts);
8531:     unsigned ExtOpcode =
8532:         (Opcode == ISD::UINT_TO_FP ? ISD::ZERO_EXTEND : ISD::SIGN_EXTEND);
8533:     SDValue ExtOp = DAG.getNode(ExtOpcode, SDLoc(N), ExtVT, Op);
8534:     return DAG.getNode(Opcode, SDLoc(N), OutVT, ExtOp);
8535:   }
8536:   return SDValue();
8537: }
8538: 
8539: SDValue SystemZTargetLowering::combineFCOPYSIGN(
8540:     SDNode *N, DAGCombinerInfo &DCI) const {
8541:   SelectionDAG &DAG = DCI.DAG;
8542:   EVT VT = N->getValueType(0);
8543:   SDValue ValOp = N->getOperand(0);
8544:   SDValue SignOp = N->getOperand(1);
8545: 
8546:   // Remove the rounding which is not needed.
8547:   if (SignOp.getOpcode() == ISD::FP_ROUND) {
8548:     SDValue WideOp = SignOp.getOperand(0);
8549:     return DAG.getNode(ISD::FCOPYSIGN, SDLoc(N), VT, ValOp, WideOp);
8550:   }
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::combineINT_TO_FP`, `SystemZTargetLowering::combineFCOPYSIGN`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::combineINT_TO_FP`, `SystemZTargetLowering::combineFCOPYSIGN` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 8551-8625 / 第 8551-8625 行
```cpp
8551: 
8552:   return SDValue();
8553: }
8554: 
8555: SDValue SystemZTargetLowering::combineBSWAP(
8556:     SDNode *N, DAGCombinerInfo &DCI) const {
8557:   SelectionDAG &DAG = DCI.DAG;
8558:   // Combine BSWAP (LOAD) into LRVH/LRV/LRVG/VLBR
8559:   if (ISD::isNON_EXTLoad(N->getOperand(0).getNode()) &&
8560:       N->getOperand(0).hasOneUse() &&
8561:       canLoadStoreByteSwapped(N->getValueType(0))) {
8562:       SDValue Load = N->getOperand(0);
8563:       LoadSDNode *LD = cast<LoadSDNode>(Load);
8564: 
8565:       // Create the byte-swapping load.
8566:       SDValue Ops[] = {
8567:         LD->getChain(),    // Chain
8568:         LD->getBasePtr()   // Ptr
8569:       };
8570:       EVT LoadVT = N->getValueType(0);
8571:       if (LoadVT == MVT::i16)
8572:         LoadVT = MVT::i32;
8573:       SDValue BSLoad =
8574:         DAG.getMemIntrinsicNode(SystemZISD::LRV, SDLoc(N),
8575:                                 DAG.getVTList(LoadVT, MVT::Other),
8576:                                 Ops, LD->getMemoryVT(), LD->getMemOperand());
8577: 
8578:       // If this is an i16 load, insert the truncate.
8579:       SDValue ResVal = BSLoad;
8580:       if (N->getValueType(0) == MVT::i16)
8581:         ResVal = DAG.getNode(ISD::TRUNCATE, SDLoc(N), MVT::i16, BSLoad);
8582: 
8583:       // First, combine the bswap away.  This makes the value produced by the
8584:       // load dead.
8585:       DCI.CombineTo(N, ResVal);
8586: 
8587:       // Next, combine the load away, we give it a bogus result value but a real
8588:       // chain result.  The result value is dead because the bswap is dead.
8589:       DCI.CombineTo(Load.getNode(), ResVal, BSLoad.getValue(1));
8590: 
8591:       // Return N so it doesn't get rechecked!
8592:       return SDValue(N, 0);
8593:     }
8594: 
8595:   // Look through bitcasts that retain the number of vector elements.
8596:   SDValue Op = N->getOperand(0);
8597:   if (Op.getOpcode() == ISD::BITCAST &&
8598:       Op.getValueType().isVector() &&
8599:       Op.getOperand(0).getValueType().isVector() &&
8600:       Op.getValueType().getVectorNumElements() ==
8601:       Op.getOperand(0).getValueType().getVectorNumElements())
8602:     Op = Op.getOperand(0);
8603: 
8604:   // Push BSWAP into a vector insertion if at least one side then simplifies.
8605:   if (Op.getOpcode() == ISD::INSERT_VECTOR_ELT && Op.hasOneUse()) {
8606:     SDValue Vec = Op.getOperand(0);
8607:     SDValue Elt = Op.getOperand(1);
8608:     SDValue Idx = Op.getOperand(2);
8609: 
8610:     if (DAG.isConstantIntBuildVectorOrConstantInt(Vec) ||
8611:         Vec.getOpcode() == ISD::BSWAP || Vec.isUndef() ||
8612:         DAG.isConstantIntBuildVectorOrConstantInt(Elt) ||
8613:         Elt.getOpcode() == ISD::BSWAP || Elt.isUndef() ||
8614:         (canLoadStoreByteSwapped(N->getValueType(0)) &&
8615:          ISD::isNON_EXTLoad(Elt.getNode()) && Elt.hasOneUse())) {
8616:       EVT VecVT = N->getValueType(0);
8617:       EVT EltVT = N->getValueType(0).getVectorElementType();
8618:       if (VecVT != Vec.getValueType()) {
8619:         Vec = DAG.getNode(ISD::BITCAST, SDLoc(N), VecVT, Vec);
8620:         DCI.AddToWorklist(Vec.getNode());
8621:       }
8622:       if (EltVT != Elt.getValueType()) {
8623:         Elt = DAG.getNode(ISD::BITCAST, SDLoc(N), EltVT, Elt);
8624:         DCI.AddToWorklist(Elt.getNode());
8625:       }
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::combineBSWAP`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::combineBSWAP` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 8626-8700 / 第 8626-8700 行
```cpp
8626:       Vec = DAG.getNode(ISD::BSWAP, SDLoc(N), VecVT, Vec);
8627:       DCI.AddToWorklist(Vec.getNode());
8628:       Elt = DAG.getNode(ISD::BSWAP, SDLoc(N), EltVT, Elt);
8629:       DCI.AddToWorklist(Elt.getNode());
8630:       return DAG.getNode(ISD::INSERT_VECTOR_ELT, SDLoc(N), VecVT,
8631:                          Vec, Elt, Idx);
8632:     }
8633:   }
8634: 
8635:   // Push BSWAP into a vector shuffle if at least one side then simplifies.
8636:   ShuffleVectorSDNode *SV = dyn_cast<ShuffleVectorSDNode>(Op);
8637:   if (SV && Op.hasOneUse()) {
8638:     SDValue Op0 = Op.getOperand(0);
8639:     SDValue Op1 = Op.getOperand(1);
8640: 
8641:     if (DAG.isConstantIntBuildVectorOrConstantInt(Op0) ||
8642:         Op0.getOpcode() == ISD::BSWAP || Op0.isUndef() ||
8643:         DAG.isConstantIntBuildVectorOrConstantInt(Op1) ||
8644:         Op1.getOpcode() == ISD::BSWAP || Op1.isUndef()) {
8645:       EVT VecVT = N->getValueType(0);
8646:       if (VecVT != Op0.getValueType()) {
8647:         Op0 = DAG.getNode(ISD::BITCAST, SDLoc(N), VecVT, Op0);
8648:         DCI.AddToWorklist(Op0.getNode());
8649:       }
8650:       if (VecVT != Op1.getValueType()) {
8651:         Op1 = DAG.getNode(ISD::BITCAST, SDLoc(N), VecVT, Op1);
8652:         DCI.AddToWorklist(Op1.getNode());
8653:       }
8654:       Op0 = DAG.getNode(ISD::BSWAP, SDLoc(N), VecVT, Op0);
8655:       DCI.AddToWorklist(Op0.getNode());
8656:       Op1 = DAG.getNode(ISD::BSWAP, SDLoc(N), VecVT, Op1);
8657:       DCI.AddToWorklist(Op1.getNode());
8658:       return DAG.getVectorShuffle(VecVT, SDLoc(N), Op0, Op1, SV->getMask());
8659:     }
8660:   }
8661: 
8662:   return SDValue();
8663: }
8664: 
8665: SDValue SystemZTargetLowering::combineSETCC(
8666:     SDNode *N, DAGCombinerInfo &DCI) const {
8667:   SelectionDAG &DAG = DCI.DAG;
8668:   const ISD::CondCode CC = cast<CondCodeSDNode>(N->getOperand(2))->get();
8669:   const SDValue LHS = N->getOperand(0);
8670:   const SDValue RHS = N->getOperand(1);
8671:   bool CmpNull = isNullConstant(RHS);
8672:   bool CmpAllOnes = isAllOnesConstant(RHS);
8673:   EVT VT = N->getValueType(0);
8674:   SDLoc DL(N);
8675: 
8676:   // Match icmp_eq/ne(bitcast(icmp(X,Y)),0/-1) reduction patterns, and
8677:   // change the outer compare to a i128 compare.  This will normally
8678:   // allow the reduction to be recognized in adjustICmp128, and even if
8679:   // not, the i128 compare will still generate better code.
8680:   if ((CC == ISD::SETNE || CC == ISD::SETEQ) && (CmpNull || CmpAllOnes)) {
8681:     SDValue Src = peekThroughBitcasts(LHS);
8682:     if (Src.getOpcode() == ISD::SETCC &&
8683:         Src.getValueType().isFixedLengthVector() &&
8684:         Src.getValueType().getScalarType() == MVT::i1) {
8685:       EVT CmpVT = Src.getOperand(0).getValueType();
8686:       if (CmpVT.getSizeInBits() == 128) {
8687:         EVT IntVT = CmpVT.changeVectorElementTypeToInteger();
8688:         SDValue LHS =
8689:             DAG.getBitcast(MVT::i128, DAG.getSExtOrTrunc(Src, DL, IntVT));
8690:         SDValue RHS = CmpNull ? DAG.getConstant(0, DL, MVT::i128)
8691:                               : DAG.getAllOnesConstant(DL, MVT::i128);
8692:         return DAG.getNode(ISD::SETCC, DL, VT, LHS, RHS, N->getOperand(2),
8693:                            N->getFlags());
8694:       }
8695:     }
8696:   }
8697: 
8698:   return SDValue();
8699: }
8700: 
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::combineSETCC`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::combineSETCC` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 8701-8775 / 第 8701-8775 行
```cpp
8701: static std::pair<SDValue, int> findCCUse(const SDValue &Val,
8702:                                          unsigned Depth = 0) {
8703:   // Limit depth of potentially exponential walk.
8704:   if (Depth > 5)
8705:     return std::make_pair(SDValue(), SystemZ::CCMASK_NONE);
8706: 
8707:   switch (Val.getOpcode()) {
8708:   default:
8709:     return std::make_pair(SDValue(), SystemZ::CCMASK_NONE);
8710:   case SystemZISD::IPM:
8711:     if (Val.getOperand(0).getOpcode() == SystemZISD::CLC ||
8712:         Val.getOperand(0).getOpcode() == SystemZISD::STRCMP)
8713:       return std::make_pair(Val.getOperand(0), SystemZ::CCMASK_ICMP);
8714:     return std::make_pair(Val.getOperand(0), SystemZ::CCMASK_ANY);
8715:   case SystemZISD::SELECT_CCMASK: {
8716:     SDValue Op4CCReg = Val.getOperand(4);
8717:     if (Op4CCReg.getOpcode() == SystemZISD::ICMP ||
8718:         Op4CCReg.getOpcode() == SystemZISD::TM) {
8719:       auto [OpCC, OpCCValid] = findCCUse(Op4CCReg.getOperand(0), Depth + 1);
8720:       if (OpCC != SDValue())
8721:         return std::make_pair(OpCC, OpCCValid);
8722:     }
8723:     auto *CCValid = dyn_cast<ConstantSDNode>(Val.getOperand(2));
8724:     if (!CCValid)
8725:       return std::make_pair(SDValue(), SystemZ::CCMASK_NONE);
8726:     int CCValidVal = CCValid->getZExtValue();
8727:     return std::make_pair(Op4CCReg, CCValidVal);
8728:   }
8729:   case ISD::ADD:
8730:   case ISD::AND:
8731:   case ISD::OR:
8732:   case ISD::XOR:
8733:   case ISD::SHL:
8734:   case ISD::SRA:
8735:   case ISD::SRL:
8736:     auto [Op0CC, Op0CCValid] = findCCUse(Val.getOperand(0), Depth + 1);
8737:     if (Op0CC != SDValue())
8738:       return std::make_pair(Op0CC, Op0CCValid);
8739:     return findCCUse(Val.getOperand(1), Depth + 1);
8740:   }
8741: }
8742: 
8743: static bool combineCCMask(SDValue &CCReg, int &CCValid, int &CCMask,
8744:                           SelectionDAG &DAG);
8745: 
8746: SmallVector<SDValue, 4> static simplifyAssumingCCVal(SDValue &Val, SDValue &CC,
8747:                                                      SelectionDAG &DAG) {
8748:   SDLoc DL(Val);
8749:   auto Opcode = Val.getOpcode();
8750:   switch (Opcode) {
8751:   default:
8752:     return {};
8753:   case ISD::Constant:
8754:     return {Val, Val, Val, Val};
8755:   case SystemZISD::IPM: {
8756:     SDValue IPMOp0 = Val.getOperand(0);
8757:     if (IPMOp0 != CC)
8758:       return {};
8759:     SmallVector<SDValue, 4> ShiftedCCVals;
8760:     for (auto CC : {0, 1, 2, 3})
8761:       ShiftedCCVals.emplace_back(
8762:           DAG.getConstant((CC << SystemZ::IPM_CC), DL, MVT::i32));
8763:     return ShiftedCCVals;
8764:   }
8765:   case SystemZISD::SELECT_CCMASK: {
8766:     SDValue TrueVal = Val.getOperand(0), FalseVal = Val.getOperand(1);
8767:     auto *CCValid = dyn_cast<ConstantSDNode>(Val.getOperand(2));
8768:     auto *CCMask = dyn_cast<ConstantSDNode>(Val.getOperand(3));
8769:     if (!CCValid || !CCMask)
8770:       return {};
8771: 
8772:     int CCValidVal = CCValid->getZExtValue();
8773:     int CCMaskVal = CCMask->getZExtValue();
8774:     // Pruning search tree early - Moving CC test and combineCCMask ahead of
8775:     // recursive call to simplifyAssumingCCVal.
```
- **EN**: The range implements or declares functions including `findCCUse`, `simplifyAssumingCCVal`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `findCCUse`, `simplifyAssumingCCVal` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 8776-8850 / 第 8776-8850 行
```cpp
8776:     SDValue Op4CCReg = Val.getOperand(4);
8777:     if (Op4CCReg != CC)
8778:       combineCCMask(Op4CCReg, CCValidVal, CCMaskVal, DAG);
8779:     if (Op4CCReg != CC)
8780:       return {};
8781:     const auto &&TrueSDVals = simplifyAssumingCCVal(TrueVal, CC, DAG);
8782:     const auto &&FalseSDVals = simplifyAssumingCCVal(FalseVal, CC, DAG);
8783:     if (TrueSDVals.empty() || FalseSDVals.empty())
8784:       return {};
8785:     SmallVector<SDValue, 4> MergedSDVals;
8786:     for (auto &CCVal : {0, 1, 2, 3})
8787:       MergedSDVals.emplace_back(((CCMaskVal & (1 << (3 - CCVal))) != 0)
8788:                                     ? TrueSDVals[CCVal]
8789:                                     : FalseSDVals[CCVal]);
8790:     return MergedSDVals;
8791:   }
8792:   case ISD::ADD:
8793:   case ISD::AND:
8794:   case ISD::OR:
8795:   case ISD::XOR:
8796:   case ISD::SRA:
8797:     // Avoid introducing CC spills (because ADD/AND/OR/XOR/SRA
8798:     // would clobber CC).
8799:     if (!Val.hasOneUse())
8800:       return {};
8801:     [[fallthrough]];
8802:   case ISD::SHL:
8803:   case ISD::SRL:
8804:     SDValue Op0 = Val.getOperand(0), Op1 = Val.getOperand(1);
8805:     const auto &&Op0SDVals = simplifyAssumingCCVal(Op0, CC, DAG);
8806:     const auto &&Op1SDVals = simplifyAssumingCCVal(Op1, CC, DAG);
8807:     if (Op0SDVals.empty() || Op1SDVals.empty())
8808:       return {};
8809:     SmallVector<SDValue, 4> BinaryOpSDVals;
8810:     for (auto CCVal : {0, 1, 2, 3})
8811:       BinaryOpSDVals.emplace_back(DAG.getNode(
8812:           Opcode, DL, Val.getValueType(), Op0SDVals[CCVal], Op1SDVals[CCVal]));
8813:     return BinaryOpSDVals;
8814:   }
8815: }
8816: 
8817: static bool combineCCMask(SDValue &CCReg, int &CCValid, int &CCMask,
8818:                           SelectionDAG &DAG) {
8819:   // We have a SELECT_CCMASK or BR_CCMASK comparing the condition code
8820:   // set by the CCReg instruction using the CCValid / CCMask masks,
8821:   // If the CCReg instruction is itself a ICMP / TM  testing the condition
8822:   // code set by some other instruction, see whether we can directly
8823:   // use that condition code.
8824:   auto *CCNode = CCReg.getNode();
8825:   if (!CCNode)
8826:     return false;
8827: 
8828:   if (CCNode->getOpcode() == SystemZISD::TM) {
8829:     if (CCValid != SystemZ::CCMASK_TM)
8830:       return false;
8831:     auto emulateTMCCMask = [](const SDValue &Op0Val, const SDValue &Op1Val) {
8832:       auto *Op0Node = dyn_cast<ConstantSDNode>(Op0Val.getNode());
8833:       auto *Op1Node = dyn_cast<ConstantSDNode>(Op1Val.getNode());
8834:       if (!Op0Node || !Op1Node)
8835:         return -1;
8836:       auto Op0APVal = Op0Node->getAPIntValue();
8837:       auto Op1APVal = Op1Node->getAPIntValue();
8838:       auto Result = Op0APVal & Op1APVal;
8839:       bool AllOnes = Result == Op1APVal;
8840:       bool AllZeros = Result == 0;
8841:       bool IsLeftMostBitSet = Result[Op1APVal.getActiveBits()] != 0;
8842:       return AllZeros ? 0 : AllOnes ? 3 : IsLeftMostBitSet ? 2 : 1;
8843:     };
8844:     SDValue Op0 = CCNode->getOperand(0);
8845:     SDValue Op1 = CCNode->getOperand(1);
8846:     auto [Op0CC, Op0CCValid] = findCCUse(Op0);
8847:     if (Op0CC == SDValue())
8848:       return false;
8849:     const auto &&Op0SDVals = simplifyAssumingCCVal(Op0, Op0CC, DAG);
8850:     const auto &&Op1SDVals = simplifyAssumingCCVal(Op1, Op0CC, DAG);
```
- **EN**: The range implements or declares functions including `combineCCMask`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `combineCCMask` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 8851-8925 / 第 8851-8925 行
```cpp
8851:     if (Op0SDVals.empty() || Op1SDVals.empty())
8852:       return false;
8853:     int NewCCMask = 0;
8854:     for (auto CC : {0, 1, 2, 3}) {
8855:       auto CCVal = emulateTMCCMask(Op0SDVals[CC], Op1SDVals[CC]);
8856:       if (CCVal < 0)
8857:         return false;
8858:       NewCCMask <<= 1;
8859:       NewCCMask |= (CCMask & (1 << (3 - CCVal))) != 0;
8860:     }
8861:     NewCCMask &= Op0CCValid;
8862:     CCReg = Op0CC;
8863:     CCMask = NewCCMask;
8864:     CCValid = Op0CCValid;
8865:     return true;
8866:   }
8867:   if (CCNode->getOpcode() != SystemZISD::ICMP ||
8868:       CCValid != SystemZ::CCMASK_ICMP)
8869:     return false;
8870: 
8871:   SDValue CmpOp0 = CCNode->getOperand(0);
8872:   SDValue CmpOp1 = CCNode->getOperand(1);
8873:   SDValue CmpOp2 = CCNode->getOperand(2);
8874:   auto [Op0CC, Op0CCValid] = findCCUse(CmpOp0);
8875:   if (Op0CC != SDValue()) {
8876:     const auto &&Op0SDVals = simplifyAssumingCCVal(CmpOp0, Op0CC, DAG);
8877:     const auto &&Op1SDVals = simplifyAssumingCCVal(CmpOp1, Op0CC, DAG);
8878:     if (Op0SDVals.empty() || Op1SDVals.empty())
8879:       return false;
8880: 
8881:     auto *CmpType = dyn_cast<ConstantSDNode>(CmpOp2);
8882:     auto CmpTypeVal = CmpType->getZExtValue();
8883:     const auto compareCCSigned = [&CmpTypeVal](const SDValue &Op0Val,
8884:                                                const SDValue &Op1Val) {
8885:       auto *Op0Node = dyn_cast<ConstantSDNode>(Op0Val.getNode());
8886:       auto *Op1Node = dyn_cast<ConstantSDNode>(Op1Val.getNode());
8887:       if (!Op0Node || !Op1Node)
8888:         return -1;
8889:       auto Op0APVal = Op0Node->getAPIntValue();
8890:       auto Op1APVal = Op1Node->getAPIntValue();
8891:       if (CmpTypeVal == SystemZICMP::SignedOnly)
8892:         return Op0APVal == Op1APVal ? 0 : Op0APVal.slt(Op1APVal) ? 1 : 2;
8893:       return Op0APVal == Op1APVal ? 0 : Op0APVal.ult(Op1APVal) ? 1 : 2;
8894:     };
8895:     int NewCCMask = 0;
8896:     for (auto CC : {0, 1, 2, 3}) {
8897:       auto CCVal = compareCCSigned(Op0SDVals[CC], Op1SDVals[CC]);
8898:       if (CCVal < 0)
8899:         return false;
8900:       NewCCMask <<= 1;
8901:       NewCCMask |= (CCMask & (1 << (3 - CCVal))) != 0;
8902:     }
8903:     NewCCMask &= Op0CCValid;
8904:     CCMask = NewCCMask;
8905:     CCReg = Op0CC;
8906:     CCValid = Op0CCValid;
8907:     return true;
8908:   }
8909: 
8910:   return false;
8911: }
8912: 
8913: // Merging versus split in multiple branches cost.
8914: TargetLoweringBase::CondMergingParams
8915: SystemZTargetLowering::getJumpConditionMergingParams(Instruction::BinaryOps Opc,
8916:                                                      const Value *Lhs,
8917:                                                      const Value *Rhs) const {
8918:   const auto isFlagOutOpCC = [](const Value *V) {
8919:     using namespace llvm::PatternMatch;
8920:     const Value *RHSVal;
8921:     const APInt *RHSC;
8922:     if (const auto *I = dyn_cast<Instruction>(V)) {
8923:       // PatternMatch.h provides concise tree-based pattern match of llvm IR.
8924:       if (match(I->getOperand(0), m_And(m_Value(RHSVal), m_APInt(RHSC))) ||
8925:           match(I, m_Cmp(m_Value(RHSVal), m_APInt(RHSC)))) {
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::getJumpConditionMergingParams`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::getJumpConditionMergingParams` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 8926-9000 / 第 8926-9000 行
```cpp
8926:         if (const auto *CB = dyn_cast<CallBase>(RHSVal)) {
8927:           if (CB->isInlineAsm()) {
8928:             const InlineAsm *IA = cast<InlineAsm>(CB->getCalledOperand());
8929:             return IA && IA->getConstraintString().contains("{@cc}");
8930:           }
8931:         }
8932:       }
8933:     }
8934:     return false;
8935:   };
8936:   // Pattern (ICmp %asm) or (ICmp (And %asm)).
8937:   // Cost of longest dependency chain (ICmp, And) is 2. CostThreshold or
8938:   // BaseCost can be set >=2. If cost of instruction <= CostThreshold
8939:   // conditionals will be merged or else conditionals will be split.
8940:   if (isFlagOutOpCC(Lhs) && isFlagOutOpCC(Rhs))
8941:     return {3, 0, -1};
8942:   // Default.
8943:   return {-1, -1, -1};
8944: }
8945: 
8946: SDValue SystemZTargetLowering::combineBR_CCMASK(SDNode *N,
8947:                                                 DAGCombinerInfo &DCI) const {
8948:   SelectionDAG &DAG = DCI.DAG;
8949: 
8950:   // Combine BR_CCMASK (ICMP (SELECT_CCMASK)) into a single BR_CCMASK.
8951:   auto *CCValid = dyn_cast<ConstantSDNode>(N->getOperand(1));
8952:   auto *CCMask = dyn_cast<ConstantSDNode>(N->getOperand(2));
8953:   if (!CCValid || !CCMask)
8954:     return SDValue();
8955: 
8956:   int CCValidVal = CCValid->getZExtValue();
8957:   int CCMaskVal = CCMask->getZExtValue();
8958:   SDValue Chain = N->getOperand(0);
8959:   SDValue CCReg = N->getOperand(4);
8960:   // If combineCMask was able to merge or simplify ccvalid or ccmask, re-emit
8961:   // the modified BR_CCMASK with the new values.
8962:   // In order to avoid conditional branches with full or empty cc masks, do not
8963:   // do this if ccmask is 0 or equal to ccvalid.
8964:   if (combineCCMask(CCReg, CCValidVal, CCMaskVal, DAG) && CCMaskVal != 0 &&
8965:       CCMaskVal != CCValidVal)
8966:     return DAG.getNode(SystemZISD::BR_CCMASK, SDLoc(N), N->getValueType(0),
8967:                        Chain,
8968:                        DAG.getTargetConstant(CCValidVal, SDLoc(N), MVT::i32),
8969:                        DAG.getTargetConstant(CCMaskVal, SDLoc(N), MVT::i32),
8970:                        N->getOperand(3), CCReg);
8971:   return SDValue();
8972: }
8973: 
8974: SDValue SystemZTargetLowering::combineSELECT_CCMASK(
8975:     SDNode *N, DAGCombinerInfo &DCI) const {
8976:   SelectionDAG &DAG = DCI.DAG;
8977: 
8978:   // Combine SELECT_CCMASK (ICMP (SELECT_CCMASK)) into a single SELECT_CCMASK.
8979:   auto *CCValid = dyn_cast<ConstantSDNode>(N->getOperand(2));
8980:   auto *CCMask = dyn_cast<ConstantSDNode>(N->getOperand(3));
8981:   if (!CCValid || !CCMask)
8982:     return SDValue();
8983: 
8984:   int CCValidVal = CCValid->getZExtValue();
8985:   int CCMaskVal = CCMask->getZExtValue();
8986:   SDValue CCReg = N->getOperand(4);
8987: 
8988:   bool IsCombinedCCReg = combineCCMask(CCReg, CCValidVal, CCMaskVal, DAG);
8989: 
8990:   // Populate SDVals vector for each condition code ccval for given Val, which
8991:   // can again be another nested select_ccmask with the same CC.
8992:   const auto constructCCSDValsFromSELECT = [&CCReg](SDValue &Val) {
8993:     if (Val.getOpcode() == SystemZISD::SELECT_CCMASK) {
8994:       SmallVector<SDValue, 4> Res;
8995:       if (Val.getOperand(4) != CCReg)
8996:         return SmallVector<SDValue, 4>{};
8997:       SDValue TrueVal = Val.getOperand(0), FalseVal = Val.getOperand(1);
8998:       auto *CCMask = dyn_cast<ConstantSDNode>(Val.getOperand(3));
8999:       if (!CCMask)
9000:         return SmallVector<SDValue, 4>{};
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::combineBR_CCMASK`, `SystemZTargetLowering::combineSELECT_CCMASK`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::combineBR_CCMASK`, `SystemZTargetLowering::combineSELECT_CCMASK` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 9001-9075 / 第 9001-9075 行
```cpp
9001: 
9002:       int CCMaskVal = CCMask->getZExtValue();
9003:       for (auto &CC : {0, 1, 2, 3})
9004:         Res.emplace_back(((CCMaskVal & (1 << (3 - CC))) != 0) ? TrueVal
9005:                                                               : FalseVal);
9006:       return Res;
9007:     }
9008:     return SmallVector<SDValue, 4>{Val, Val, Val, Val};
9009:   };
9010:   // Attempting to optimize TrueVal/FalseVal in outermost select_ccmask either
9011:   // with CCReg found by combineCCMask or original CCReg.
9012:   SDValue TrueVal = N->getOperand(0);
9013:   SDValue FalseVal = N->getOperand(1);
9014:   auto &&TrueSDVals = simplifyAssumingCCVal(TrueVal, CCReg, DAG);
9015:   auto &&FalseSDVals = simplifyAssumingCCVal(FalseVal, CCReg, DAG);
9016:   // TrueSDVals/FalseSDVals might be empty in case of non-constant
9017:   // TrueVal/FalseVal for select_ccmask, which can not be optimized further.
9018:   if (TrueSDVals.empty())
9019:     TrueSDVals = constructCCSDValsFromSELECT(TrueVal);
9020:   if (FalseSDVals.empty())
9021:     FalseSDVals = constructCCSDValsFromSELECT(FalseVal);
9022:   if (!TrueSDVals.empty() && !FalseSDVals.empty()) {
9023:     SmallSet<SDValue, 4> MergedSDValsSet;
9024:     // Ignoring CC values outside CCValiid.
9025:     for (auto CC : {0, 1, 2, 3}) {
9026:       if ((CCValidVal & ((1 << (3 - CC)))) != 0)
9027:         MergedSDValsSet.insert(((CCMaskVal & (1 << (3 - CC))) != 0)
9028:                                    ? TrueSDVals[CC]
9029:                                    : FalseSDVals[CC]);
9030:     }
9031:     if (MergedSDValsSet.size() == 1)
9032:       return *MergedSDValsSet.begin();
9033:     if (MergedSDValsSet.size() == 2) {
9034:       auto BeginIt = MergedSDValsSet.begin();
9035:       SDValue NewTrueVal = *BeginIt, NewFalseVal = *next(BeginIt);
9036:       if (NewTrueVal == FalseVal || NewFalseVal == TrueVal)
9037:         std::swap(NewTrueVal, NewFalseVal);
9038:       int NewCCMask = 0;
9039:       for (auto CC : {0, 1, 2, 3}) {
9040:         NewCCMask <<= 1;
9041:         NewCCMask |= ((CCMaskVal & (1 << (3 - CC))) != 0)
9042:                          ? (TrueSDVals[CC] == NewTrueVal)
9043:                          : (FalseSDVals[CC] == NewTrueVal);
9044:       }
9045:       CCMaskVal = NewCCMask;
9046:       CCMaskVal &= CCValidVal;
9047:       TrueVal = NewTrueVal;
9048:       FalseVal = NewFalseVal;
9049:       IsCombinedCCReg = true;
9050:     }
9051:   }
9052:   // If the condition is trivially false or trivially true after
9053:   // combineCCMask, just collapse this SELECT_CCMASK to the indicated value
9054:   // (possibly modified by constructCCSDValsFromSELECT).
9055:   if (CCMaskVal == 0)
9056:     return FalseVal;
9057:   if (CCMaskVal == CCValidVal)
9058:     return TrueVal;
9059: 
9060:   if (IsCombinedCCReg)
9061:     return DAG.getNode(
9062:         SystemZISD::SELECT_CCMASK, SDLoc(N), N->getValueType(0), TrueVal,
9063:         FalseVal, DAG.getTargetConstant(CCValidVal, SDLoc(N), MVT::i32),
9064:         DAG.getTargetConstant(CCMaskVal, SDLoc(N), MVT::i32), CCReg);
9065: 
9066:   return SDValue();
9067: }
9068: 
9069: SDValue SystemZTargetLowering::combineGET_CCMASK(
9070:     SDNode *N, DAGCombinerInfo &DCI) const {
9071: 
9072:   // Optimize away GET_CCMASK (SELECT_CCMASK) if the CC masks are compatible
9073:   auto *CCValid = dyn_cast<ConstantSDNode>(N->getOperand(1));
9074:   auto *CCMask = dyn_cast<ConstantSDNode>(N->getOperand(2));
9075:   if (!CCValid || !CCMask)
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::combineGET_CCMASK`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::combineGET_CCMASK` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 9076-9150 / 第 9076-9150 行
```cpp
9076:     return SDValue();
9077:   int CCValidVal = CCValid->getZExtValue();
9078:   int CCMaskVal = CCMask->getZExtValue();
9079: 
9080:   SDValue Select = N->getOperand(0);
9081:   if (Select->getOpcode() == ISD::TRUNCATE)
9082:     Select = Select->getOperand(0);
9083:   if (Select->getOpcode() != SystemZISD::SELECT_CCMASK)
9084:     return SDValue();
9085: 
9086:   auto *SelectCCValid = dyn_cast<ConstantSDNode>(Select->getOperand(2));
9087:   auto *SelectCCMask = dyn_cast<ConstantSDNode>(Select->getOperand(3));
9088:   if (!SelectCCValid || !SelectCCMask)
9089:     return SDValue();
9090:   int SelectCCValidVal = SelectCCValid->getZExtValue();
9091:   int SelectCCMaskVal = SelectCCMask->getZExtValue();
9092: 
9093:   auto *TrueVal = dyn_cast<ConstantSDNode>(Select->getOperand(0));
9094:   auto *FalseVal = dyn_cast<ConstantSDNode>(Select->getOperand(1));
9095:   if (!TrueVal || !FalseVal)
9096:     return SDValue();
9097:   if (TrueVal->getZExtValue() == 1 && FalseVal->getZExtValue() == 0)
9098:     ;
9099:   else if (TrueVal->getZExtValue() == 0 && FalseVal->getZExtValue() == 1)
9100:     SelectCCMaskVal ^= SelectCCValidVal;
9101:   else
9102:     return SDValue();
9103: 
9104:   if (SelectCCValidVal & ~CCValidVal)
9105:     return SDValue();
9106:   if (SelectCCMaskVal != (CCMaskVal & SelectCCValidVal))
9107:     return SDValue();
9108: 
9109:   return Select->getOperand(4);
9110: }
9111: 
9112: SDValue SystemZTargetLowering::combineIntDIVREM(
9113:     SDNode *N, DAGCombinerInfo &DCI) const {
9114:   SelectionDAG &DAG = DCI.DAG;
9115:   EVT VT = N->getValueType(0);
9116:   // In the case where the divisor is a vector of constants a cheaper
9117:   // sequence of instructions can replace the divide. BuildSDIV is called to
9118:   // do this during DAG combining, but it only succeeds when it can build a
9119:   // multiplication node. The only option for SystemZ is ISD::SMUL_LOHI, and
9120:   // since it is not Legal but Custom it can only happen before
9121:   // legalization. Therefore we must scalarize this early before Combine
9122:   // 1. For widened vectors, this is already the result of type legalization.
9123:   if (DCI.Level == BeforeLegalizeTypes && VT.isVector() && isTypeLegal(VT) &&
9124:       DAG.isConstantIntBuildVectorOrConstantInt(N->getOperand(1)))
9125:     return DAG.UnrollVectorOp(N);
9126:   return SDValue();
9127: }
9128: 
9129: 
9130: // Transform a right shift of a multiply-and-add into a multiply-and-add-high.
9131: // This is closely modeled after the common-code combineShiftToMULH.
9132: SDValue SystemZTargetLowering::combineShiftToMulAddHigh(
9133:     SDNode *N, DAGCombinerInfo &DCI) const {
9134:   SelectionDAG &DAG = DCI.DAG;
9135:   SDLoc DL(N);
9136: 
9137:   assert((N->getOpcode() == ISD::SRL || N->getOpcode() == ISD::SRA) &&
9138:          "SRL or SRA node is required here!");
9139: 
9140:   if (!Subtarget.hasVector())
9141:     return SDValue();
9142: 
9143:   // Check the shift amount. Proceed with the transformation if the shift
9144:   // amount is constant.
9145:   ConstantSDNode *ShiftAmtSrc = isConstOrConstSplat(N->getOperand(1));
9146:   if (!ShiftAmtSrc)
9147:     return SDValue();
9148: 
9149:   // The operation feeding into the shift must be an add.
9150:   SDValue ShiftOperand = N->getOperand(0);
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::combineIntDIVREM`, `SystemZTargetLowering::combineShiftToMulAddHigh`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::combineIntDIVREM`, `SystemZTargetLowering::combineShiftToMulAddHigh` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 9151-9225 / 第 9151-9225 行
```cpp
9151:   if (ShiftOperand.getOpcode() != ISD::ADD)
9152:     return SDValue();
9153: 
9154:   // One operand of the add must be a multiply.
9155:   SDValue MulOp = ShiftOperand.getOperand(0);
9156:   SDValue AddOp = ShiftOperand.getOperand(1);
9157:   if (MulOp.getOpcode() != ISD::MUL) {
9158:     if (AddOp.getOpcode() != ISD::MUL)
9159:       return SDValue();
9160:     std::swap(MulOp, AddOp);
9161:   }
9162: 
9163:   // All operands must be equivalent extend nodes.
9164:   SDValue LeftOp = MulOp.getOperand(0);
9165:   SDValue RightOp = MulOp.getOperand(1);
9166: 
9167:   bool IsSignExt = LeftOp.getOpcode() == ISD::SIGN_EXTEND;
9168:   bool IsZeroExt = LeftOp.getOpcode() == ISD::ZERO_EXTEND;
9169: 
9170:   if (!IsSignExt && !IsZeroExt)
9171:     return SDValue();
9172: 
9173:   EVT NarrowVT = LeftOp.getOperand(0).getValueType();
9174:   unsigned NarrowVTSize = NarrowVT.getScalarSizeInBits();
9175: 
9176:   SDValue MulhRightOp;
9177:   if (ConstantSDNode *Constant = isConstOrConstSplat(RightOp)) {
9178:     unsigned ActiveBits = IsSignExt
9179:                               ? Constant->getAPIntValue().getSignificantBits()
9180:                               : Constant->getAPIntValue().getActiveBits();
9181:     if (ActiveBits > NarrowVTSize)
9182:       return SDValue();
9183:     MulhRightOp = DAG.getConstant(
9184:         Constant->getAPIntValue().trunc(NarrowVT.getScalarSizeInBits()), DL,
9185:         NarrowVT);
9186:   } else {
9187:     if (LeftOp.getOpcode() != RightOp.getOpcode())
9188:       return SDValue();
9189:     // Check that the two extend nodes are the same type.
9190:     if (NarrowVT != RightOp.getOperand(0).getValueType())
9191:       return SDValue();
9192:     MulhRightOp = RightOp.getOperand(0);
9193:   }
9194: 
9195:   SDValue MulhAddOp;
9196:   if (ConstantSDNode *Constant = isConstOrConstSplat(AddOp)) {
9197:     unsigned ActiveBits = IsSignExt
9198:                               ? Constant->getAPIntValue().getSignificantBits()
9199:                               : Constant->getAPIntValue().getActiveBits();
9200:     if (ActiveBits > NarrowVTSize)
9201:       return SDValue();
9202:     MulhAddOp = DAG.getConstant(
9203:         Constant->getAPIntValue().trunc(NarrowVT.getScalarSizeInBits()), DL,
9204:         NarrowVT);
9205:   } else {
9206:     if (LeftOp.getOpcode() != AddOp.getOpcode())
9207:       return SDValue();
9208:     // Check that the two extend nodes are the same type.
9209:     if (NarrowVT != AddOp.getOperand(0).getValueType())
9210:       return SDValue();
9211:     MulhAddOp = AddOp.getOperand(0);
9212:   }
9213: 
9214:   EVT WideVT = LeftOp.getValueType();
9215:   // Proceed with the transformation if the wide types match.
9216:   assert((WideVT == RightOp.getValueType()) &&
9217:          "Cannot have a multiply node with two different operand types.");
9218:   assert((WideVT == AddOp.getValueType()) &&
9219:          "Cannot have an add node with two different operand types.");
9220: 
9221:   // Proceed with the transformation if the wide type is twice as large
9222:   // as the narrow type.
9223:   if (WideVT.getScalarSizeInBits() != 2 * NarrowVTSize)
9224:     return SDValue();
9225: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 9226-9300 / 第 9226-9300 行
```cpp
9226:   // Check the shift amount with the narrow type size.
9227:   // Proceed with the transformation if the shift amount is the width
9228:   // of the narrow type.
9229:   unsigned ShiftAmt = ShiftAmtSrc->getZExtValue();
9230:   if (ShiftAmt != NarrowVTSize)
9231:     return SDValue();
9232: 
9233:   // Proceed if we support the multiply-and-add-high operation.
9234:   if (!(NarrowVT == MVT::v16i8 || NarrowVT == MVT::v8i16 ||
9235:         NarrowVT == MVT::v4i32 ||
9236:         (Subtarget.hasVectorEnhancements3() &&
9237:          (NarrowVT == MVT::v2i64 || NarrowVT == MVT::i128))))
9238:     return SDValue();
9239: 
9240:   // Emit the VMAH (signed) or VMALH (unsigned) operation.
9241:   SDValue Result = DAG.getNode(IsSignExt ? SystemZISD::VMAH : SystemZISD::VMALH,
9242:                                DL, NarrowVT, LeftOp.getOperand(0),
9243:                                MulhRightOp, MulhAddOp);
9244:   bool IsSigned = N->getOpcode() == ISD::SRA;
9245:   return DAG.getExtOrTrunc(IsSigned, Result, DL, WideVT);
9246: }
9247: 
9248: // Op is an operand of a multiplication.  Check whether this can be folded
9249: // into an even/odd widening operation; if so, return the opcode to be used
9250: // and update Op to the appropriate sub-operand.  Note that the caller must
9251: // verify that *both* operands of the multiplication support the operation.
9252: static unsigned detectEvenOddMultiplyOperand(const SelectionDAG &DAG,
9253:                                              const SystemZSubtarget &Subtarget,
9254:                                              SDValue &Op) {
9255:   EVT VT = Op.getValueType();
9256: 
9257:   // Check for (sign/zero_extend_vector_inreg (vector_shuffle)) corresponding
9258:   // to selecting the even or odd vector elements.
9259:   if (VT.isVector() && DAG.getTargetLoweringInfo().isTypeLegal(VT) &&
9260:       (Op.getOpcode() == ISD::SIGN_EXTEND_VECTOR_INREG ||
9261:        Op.getOpcode() == ISD::ZERO_EXTEND_VECTOR_INREG)) {
9262:     bool IsSigned = Op.getOpcode() == ISD::SIGN_EXTEND_VECTOR_INREG;
9263:     unsigned NumElts = VT.getVectorNumElements();
9264:     Op = Op.getOperand(0);
9265:     if (Op.getValueType().getVectorNumElements() == 2 * NumElts &&
9266:         Op.getOpcode() == ISD::VECTOR_SHUFFLE) {
9267:       ShuffleVectorSDNode *SVN = cast<ShuffleVectorSDNode>(Op.getNode());
9268:       ArrayRef<int> ShuffleMask = SVN->getMask();
9269:       bool CanUseEven = true, CanUseOdd = true;
9270:       for (unsigned Elt = 0; Elt < NumElts; Elt++) {
9271:         if (ShuffleMask[Elt] == -1)
9272:           continue;
9273:         if (unsigned(ShuffleMask[Elt]) != 2 * Elt)
9274:           CanUseEven = false;
9275:         if (unsigned(ShuffleMask[Elt]) != 2 * Elt + 1)
9276:           CanUseOdd = false;
9277:       }
9278:       Op = Op.getOperand(0);
9279:       if (CanUseEven)
9280:         return IsSigned ? SystemZISD::VME : SystemZISD::VMLE;
9281:       if (CanUseOdd)
9282:         return IsSigned ? SystemZISD::VMO : SystemZISD::VMLO;
9283:     }
9284:   }
9285: 
9286:   // For z17, we can also support the v2i64->i128 case, which looks like
9287:   // (sign/zero_extend (extract_vector_elt X 0/1))
9288:   if (VT == MVT::i128 && Subtarget.hasVectorEnhancements3() &&
9289:       (Op.getOpcode() == ISD::SIGN_EXTEND ||
9290:        Op.getOpcode() == ISD::ZERO_EXTEND)) {
9291:     bool IsSigned = Op.getOpcode() == ISD::SIGN_EXTEND;
9292:     Op = Op.getOperand(0);
9293:     if (Op.getOpcode() == ISD::EXTRACT_VECTOR_ELT &&
9294:         Op.getOperand(0).getValueType() == MVT::v2i64 &&
9295:         Op.getOperand(1).getOpcode() == ISD::Constant) {
9296:       unsigned Elem = Op.getConstantOperandVal(1);
9297:       Op = Op.getOperand(0);
9298:       if (Elem == 0)
9299:         return IsSigned ? SystemZISD::VME : SystemZISD::VMLE;
9300:       if (Elem == 1)
```
- **EN**: The range implements or declares functions including `detectEvenOddMultiplyOperand`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `detectEvenOddMultiplyOperand` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 9301-9375 / 第 9301-9375 行
```cpp
9301:         return IsSigned ? SystemZISD::VMO : SystemZISD::VMLO;
9302:     }
9303:   }
9304: 
9305:   return 0;
9306: }
9307: 
9308: SDValue SystemZTargetLowering::combineMUL(
9309:     SDNode *N, DAGCombinerInfo &DCI) const {
9310:   SelectionDAG &DAG = DCI.DAG;
9311: 
9312:   // Detect even/odd widening multiplication.
9313:   SDValue Op0 = N->getOperand(0);
9314:   SDValue Op1 = N->getOperand(1);
9315:   unsigned OpcodeCand0 = detectEvenOddMultiplyOperand(DAG, Subtarget, Op0);
9316:   unsigned OpcodeCand1 = detectEvenOddMultiplyOperand(DAG, Subtarget, Op1);
9317:   if (OpcodeCand0 && OpcodeCand0 == OpcodeCand1)
9318:     return DAG.getNode(OpcodeCand0, SDLoc(N), N->getValueType(0), Op0, Op1);
9319: 
9320:   return SDValue();
9321: }
9322: 
9323: SDValue SystemZTargetLowering::combineINTRINSIC(
9324:     SDNode *N, DAGCombinerInfo &DCI) const {
9325:   SelectionDAG &DAG = DCI.DAG;
9326: 
9327:   unsigned Id = N->getConstantOperandVal(1);
9328:   switch (Id) {
9329:   // VECTOR LOAD (RIGHTMOST) WITH LENGTH with a length operand of 15
9330:   // or larger is simply a vector load.
9331:   case Intrinsic::s390_vll:
9332:   case Intrinsic::s390_vlrl:
9333:     if (auto *C = dyn_cast<ConstantSDNode>(N->getOperand(2)))
9334:       if (C->getZExtValue() >= 15)
9335:         return DAG.getLoad(N->getValueType(0), SDLoc(N), N->getOperand(0),
9336:                            N->getOperand(3), MachinePointerInfo());
9337:     break;
9338:   // Likewise for VECTOR STORE (RIGHTMOST) WITH LENGTH.
9339:   case Intrinsic::s390_vstl:
9340:   case Intrinsic::s390_vstrl:
9341:     if (auto *C = dyn_cast<ConstantSDNode>(N->getOperand(3)))
9342:       if (C->getZExtValue() >= 15)
9343:         return DAG.getStore(N->getOperand(0), SDLoc(N), N->getOperand(2),
9344:                             N->getOperand(4), MachinePointerInfo());
9345:     break;
9346:   }
9347: 
9348:   return SDValue();
9349: }
9350: 
9351: SDValue SystemZTargetLowering::unwrapAddress(SDValue N) const {
9352:   if (N->getOpcode() == SystemZISD::PCREL_WRAPPER)
9353:     return N->getOperand(0);
9354:   return N;
9355: }
9356: 
9357: SDValue SystemZTargetLowering::PerformDAGCombine(SDNode *N,
9358:                                                  DAGCombinerInfo &DCI) const {
9359:   switch(N->getOpcode()) {
9360:   default: break;
9361:   case ISD::ZERO_EXTEND:        return combineZERO_EXTEND(N, DCI);
9362:   case ISD::SIGN_EXTEND:        return combineSIGN_EXTEND(N, DCI);
9363:   case ISD::SIGN_EXTEND_INREG:  return combineSIGN_EXTEND_INREG(N, DCI);
9364:   case SystemZISD::MERGE_HIGH:
9365:   case SystemZISD::MERGE_LOW:   return combineMERGE(N, DCI);
9366:   case ISD::LOAD:               return combineLOAD(N, DCI);
9367:   case ISD::STORE:              return combineSTORE(N, DCI);
9368:   case ISD::VECTOR_SHUFFLE:     return combineVECTOR_SHUFFLE(N, DCI);
9369:   case ISD::EXTRACT_VECTOR_ELT: return combineEXTRACT_VECTOR_ELT(N, DCI);
9370:   case SystemZISD::JOIN_DWORDS: return combineJOIN_DWORDS(N, DCI);
9371:   case ISD::STRICT_FP_ROUND:
9372:   case ISD::FP_ROUND:           return combineFP_ROUND(N, DCI);
9373:   case ISD::STRICT_FP_EXTEND:
9374:   case ISD::FP_EXTEND:          return combineFP_EXTEND(N, DCI);
9375:   case ISD::SINT_TO_FP:
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::combineMUL`, `SystemZTargetLowering::combineINTRINSIC`, `SystemZTargetLowering::unwrapAddress`, `SystemZTargetLowering::PerformDAGCombine`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::combineMUL`, `SystemZTargetLowering::combineINTRINSIC`, `SystemZTargetLowering::unwrapAddress`, `SystemZTargetLowering::PerformDAGCombine` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 9376-9450 / 第 9376-9450 行
```cpp
9376:   case ISD::UINT_TO_FP:         return combineINT_TO_FP(N, DCI);
9377:   case ISD::FCOPYSIGN:          return combineFCOPYSIGN(N, DCI);
9378:   case ISD::BSWAP:              return combineBSWAP(N, DCI);
9379:   case ISD::SETCC:              return combineSETCC(N, DCI);
9380:   case SystemZISD::BR_CCMASK:   return combineBR_CCMASK(N, DCI);
9381:   case SystemZISD::SELECT_CCMASK: return combineSELECT_CCMASK(N, DCI);
9382:   case SystemZISD::GET_CCMASK:  return combineGET_CCMASK(N, DCI);
9383:   case ISD::SRL:
9384:   case ISD::SRA:                return combineShiftToMulAddHigh(N, DCI);
9385:   case ISD::MUL:                return combineMUL(N, DCI);
9386:   case ISD::SDIV:
9387:   case ISD::UDIV:
9388:   case ISD::SREM:
9389:   case ISD::UREM:               return combineIntDIVREM(N, DCI);
9390:   case ISD::INTRINSIC_W_CHAIN:
9391:   case ISD::INTRINSIC_VOID:     return combineINTRINSIC(N, DCI);
9392:   }
9393: 
9394:   return SDValue();
9395: }
9396: 
9397: // Return the demanded elements for the OpNo source operand of Op. DemandedElts
9398: // are for Op.
9399: static APInt getDemandedSrcElements(SDValue Op, const APInt &DemandedElts,
9400:                                     unsigned OpNo) {
9401:   EVT VT = Op.getValueType();
9402:   unsigned NumElts = (VT.isVector() ? VT.getVectorNumElements() : 1);
9403:   APInt SrcDemE;
9404:   unsigned Opcode = Op.getOpcode();
9405:   if (Opcode == ISD::INTRINSIC_WO_CHAIN) {
9406:     unsigned Id = Op.getConstantOperandVal(0);
9407:     switch (Id) {
9408:     case Intrinsic::s390_vpksh:   // PACKS
9409:     case Intrinsic::s390_vpksf:
9410:     case Intrinsic::s390_vpksg:
9411:     case Intrinsic::s390_vpkshs:  // PACKS_CC
9412:     case Intrinsic::s390_vpksfs:
9413:     case Intrinsic::s390_vpksgs:
9414:     case Intrinsic::s390_vpklsh:  // PACKLS
9415:     case Intrinsic::s390_vpklsf:
9416:     case Intrinsic::s390_vpklsg:
9417:     case Intrinsic::s390_vpklshs: // PACKLS_CC
9418:     case Intrinsic::s390_vpklsfs:
9419:     case Intrinsic::s390_vpklsgs:
9420:       // VECTOR PACK truncates the elements of two source vectors into one.
9421:       SrcDemE = DemandedElts;
9422:       if (OpNo == 2)
9423:         SrcDemE.lshrInPlace(NumElts / 2);
9424:       SrcDemE = SrcDemE.trunc(NumElts / 2);
9425:       break;
9426:       // VECTOR UNPACK extends half the elements of the source vector.
9427:     case Intrinsic::s390_vuphb:  // VECTOR UNPACK HIGH
9428:     case Intrinsic::s390_vuphh:
9429:     case Intrinsic::s390_vuphf:
9430:     case Intrinsic::s390_vuplhb: // VECTOR UNPACK LOGICAL HIGH
9431:     case Intrinsic::s390_vuplhh:
9432:     case Intrinsic::s390_vuplhf:
9433:       SrcDemE = APInt(NumElts * 2, 0);
9434:       SrcDemE.insertBits(DemandedElts, 0);
9435:       break;
9436:     case Intrinsic::s390_vuplb:  // VECTOR UNPACK LOW
9437:     case Intrinsic::s390_vuplhw:
9438:     case Intrinsic::s390_vuplf:
9439:     case Intrinsic::s390_vupllb: // VECTOR UNPACK LOGICAL LOW
9440:     case Intrinsic::s390_vupllh:
9441:     case Intrinsic::s390_vupllf:
9442:       SrcDemE = APInt(NumElts * 2, 0);
9443:       SrcDemE.insertBits(DemandedElts, NumElts);
9444:       break;
9445:     case Intrinsic::s390_vpdi: {
9446:       // VECTOR PERMUTE DWORD IMMEDIATE selects one element from each source.
9447:       SrcDemE = APInt(NumElts, 0);
9448:       if (!DemandedElts[OpNo - 1])
9449:         break;
9450:       unsigned Mask = Op.getConstantOperandVal(3);
```
- **EN**: The range implements or declares functions including `getDemandedSrcElements`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getDemandedSrcElements` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 9451-9525 / 第 9451-9525 行
```cpp
9451:       unsigned MaskBit = ((OpNo - 1) ? 1 : 4);
9452:       // Demand input element 0 or 1, given by the mask bit value.
9453:       SrcDemE.setBit((Mask & MaskBit)? 1 : 0);
9454:       break;
9455:     }
9456:     case Intrinsic::s390_vsldb: {
9457:       // VECTOR SHIFT LEFT DOUBLE BY BYTE
9458:       assert(VT == MVT::v16i8 && "Unexpected type.");
9459:       unsigned FirstIdx = Op.getConstantOperandVal(3);
9460:       assert (FirstIdx > 0 && FirstIdx < 16 && "Unused operand.");
9461:       unsigned NumSrc0Els = 16 - FirstIdx;
9462:       SrcDemE = APInt(NumElts, 0);
9463:       if (OpNo == 1) {
9464:         APInt DemEls = DemandedElts.trunc(NumSrc0Els);
9465:         SrcDemE.insertBits(DemEls, FirstIdx);
9466:       } else {
9467:         APInt DemEls = DemandedElts.lshr(NumSrc0Els);
9468:         SrcDemE.insertBits(DemEls, 0);
9469:       }
9470:       break;
9471:     }
9472:     case Intrinsic::s390_vperm:
9473:       SrcDemE = APInt::getAllOnes(NumElts);
9474:       break;
9475:     default:
9476:       llvm_unreachable("Unhandled intrinsic.");
9477:       break;
9478:     }
9479:   } else {
9480:     switch (Opcode) {
9481:     case SystemZISD::JOIN_DWORDS:
9482:       // Scalar operand.
9483:       SrcDemE = APInt(1, 1);
9484:       break;
9485:     case SystemZISD::SELECT_CCMASK:
9486:       SrcDemE = DemandedElts;
9487:       break;
9488:     default:
9489:       llvm_unreachable("Unhandled opcode.");
9490:       break;
9491:     }
9492:   }
9493:   return SrcDemE;
9494: }
9495: 
9496: static void computeKnownBitsBinOp(const SDValue Op, KnownBits &Known,
9497:                                   const APInt &DemandedElts,
9498:                                   const SelectionDAG &DAG, unsigned Depth,
9499:                                   unsigned OpNo) {
9500:   APInt Src0DemE = getDemandedSrcElements(Op, DemandedElts, OpNo);
9501:   APInt Src1DemE = getDemandedSrcElements(Op, DemandedElts, OpNo + 1);
9502:   KnownBits LHSKnown =
9503:       DAG.computeKnownBits(Op.getOperand(OpNo), Src0DemE, Depth + 1);
9504:   KnownBits RHSKnown =
9505:       DAG.computeKnownBits(Op.getOperand(OpNo + 1), Src1DemE, Depth + 1);
9506:   Known = LHSKnown.intersectWith(RHSKnown);
9507: }
9508: 
9509: void
9510: SystemZTargetLowering::computeKnownBitsForTargetNode(const SDValue Op,
9511:                                                      KnownBits &Known,
9512:                                                      const APInt &DemandedElts,
9513:                                                      const SelectionDAG &DAG,
9514:                                                      unsigned Depth) const {
9515:   Known.resetAll();
9516: 
9517:   // Intrinsic CC result is returned in the two low bits.
9518:   unsigned Tmp0, Tmp1; // not used
9519:   if (Op.getResNo() == 1 && isIntrinsicWithCC(Op, Tmp0, Tmp1)) {
9520:     Known.Zero.setBitsFrom(2);
9521:     return;
9522:   }
9523:   EVT VT = Op.getValueType();
9524:   if (Op.getResNo() != 0 || VT == MVT::Untyped)
9525:     return;
```
- **EN**: The range implements or declares functions including `computeKnownBitsBinOp`, `SystemZTargetLowering::computeKnownBitsForTargetNode`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `computeKnownBitsBinOp`, `SystemZTargetLowering::computeKnownBitsForTargetNode` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 9526-9600 / 第 9526-9600 行
```cpp
9526:   assert (Known.getBitWidth() == VT.getScalarSizeInBits() &&
9527:           "KnownBits does not match VT in bitwidth");
9528:   assert ((!VT.isVector() ||
9529:            (DemandedElts.getBitWidth() == VT.getVectorNumElements())) &&
9530:           "DemandedElts does not match VT number of elements");
9531:   unsigned BitWidth = Known.getBitWidth();
9532:   unsigned Opcode = Op.getOpcode();
9533:   if (Opcode == ISD::INTRINSIC_WO_CHAIN) {
9534:     bool IsLogical = false;
9535:     unsigned Id = Op.getConstantOperandVal(0);
9536:     switch (Id) {
9537:     case Intrinsic::s390_vpksh:   // PACKS
9538:     case Intrinsic::s390_vpksf:
9539:     case Intrinsic::s390_vpksg:
9540:     case Intrinsic::s390_vpkshs:  // PACKS_CC
9541:     case Intrinsic::s390_vpksfs:
9542:     case Intrinsic::s390_vpksgs:
9543:     case Intrinsic::s390_vpklsh:  // PACKLS
9544:     case Intrinsic::s390_vpklsf:
9545:     case Intrinsic::s390_vpklsg:
9546:     case Intrinsic::s390_vpklshs: // PACKLS_CC
9547:     case Intrinsic::s390_vpklsfs:
9548:     case Intrinsic::s390_vpklsgs:
9549:     case Intrinsic::s390_vpdi:
9550:     case Intrinsic::s390_vsldb:
9551:     case Intrinsic::s390_vperm:
9552:       computeKnownBitsBinOp(Op, Known, DemandedElts, DAG, Depth, 1);
9553:       break;
9554:     case Intrinsic::s390_vuplhb: // VECTOR UNPACK LOGICAL HIGH
9555:     case Intrinsic::s390_vuplhh:
9556:     case Intrinsic::s390_vuplhf:
9557:     case Intrinsic::s390_vupllb: // VECTOR UNPACK LOGICAL LOW
9558:     case Intrinsic::s390_vupllh:
9559:     case Intrinsic::s390_vupllf:
9560:       IsLogical = true;
9561:       [[fallthrough]];
9562:     case Intrinsic::s390_vuphb:  // VECTOR UNPACK HIGH
9563:     case Intrinsic::s390_vuphh:
9564:     case Intrinsic::s390_vuphf:
9565:     case Intrinsic::s390_vuplb:  // VECTOR UNPACK LOW
9566:     case Intrinsic::s390_vuplhw:
9567:     case Intrinsic::s390_vuplf: {
9568:       SDValue SrcOp = Op.getOperand(1);
9569:       APInt SrcDemE = getDemandedSrcElements(Op, DemandedElts, 0);
9570:       Known = DAG.computeKnownBits(SrcOp, SrcDemE, Depth + 1);
9571:       if (IsLogical) {
9572:         Known = Known.zext(BitWidth);
9573:       } else
9574:         Known = Known.sext(BitWidth);
9575:       break;
9576:     }
9577:     default:
9578:       break;
9579:     }
9580:   } else {
9581:     switch (Opcode) {
9582:     case SystemZISD::JOIN_DWORDS:
9583:     case SystemZISD::SELECT_CCMASK:
9584:       computeKnownBitsBinOp(Op, Known, DemandedElts, DAG, Depth, 0);
9585:       break;
9586:     case SystemZISD::REPLICATE: {
9587:       SDValue SrcOp = Op.getOperand(0);
9588:       Known = DAG.computeKnownBits(SrcOp, Depth + 1);
9589:       if (Known.getBitWidth() < BitWidth && isa<ConstantSDNode>(SrcOp))
9590:         Known = Known.sext(BitWidth); // VREPI sign extends the immedate.
9591:       break;
9592:     }
9593:     default:
9594:       break;
9595:     }
9596:   }
9597: 
9598:   // Known has the width of the source operand(s). Adjust if needed to match
9599:   // the passed bitwidth.
9600:   if (Known.getBitWidth() != BitWidth)
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 9601-9675 / 第 9601-9675 行
```cpp
9601:     Known = Known.anyextOrTrunc(BitWidth);
9602: }
9603: 
9604: static unsigned computeNumSignBitsBinOp(SDValue Op, const APInt &DemandedElts,
9605:                                         const SelectionDAG &DAG, unsigned Depth,
9606:                                         unsigned OpNo) {
9607:   APInt Src0DemE = getDemandedSrcElements(Op, DemandedElts, OpNo);
9608:   unsigned LHS = DAG.ComputeNumSignBits(Op.getOperand(OpNo), Src0DemE, Depth + 1);
9609:   if (LHS == 1) return 1; // Early out.
9610:   APInt Src1DemE = getDemandedSrcElements(Op, DemandedElts, OpNo + 1);
9611:   unsigned RHS = DAG.ComputeNumSignBits(Op.getOperand(OpNo + 1), Src1DemE, Depth + 1);
9612:   if (RHS == 1) return 1; // Early out.
9613:   unsigned Common = std::min(LHS, RHS);
9614:   unsigned SrcBitWidth = Op.getOperand(OpNo).getScalarValueSizeInBits();
9615:   EVT VT = Op.getValueType();
9616:   unsigned VTBits = VT.getScalarSizeInBits();
9617:   if (SrcBitWidth > VTBits) { // PACK
9618:     unsigned SrcExtraBits = SrcBitWidth - VTBits;
9619:     if (Common > SrcExtraBits)
9620:       return (Common - SrcExtraBits);
9621:     return 1;
9622:   }
9623:   assert (SrcBitWidth == VTBits && "Expected operands of same bitwidth.");
9624:   return Common;
9625: }
9626: 
9627: unsigned
9628: SystemZTargetLowering::ComputeNumSignBitsForTargetNode(
9629:     SDValue Op, const APInt &DemandedElts, const SelectionDAG &DAG,
9630:     unsigned Depth) const {
9631:   if (Op.getResNo() != 0)
9632:     return 1;
9633:   unsigned Opcode = Op.getOpcode();
9634:   if (Opcode == ISD::INTRINSIC_WO_CHAIN) {
9635:     unsigned Id = Op.getConstantOperandVal(0);
9636:     switch (Id) {
9637:     case Intrinsic::s390_vpksh:   // PACKS
9638:     case Intrinsic::s390_vpksf:
9639:     case Intrinsic::s390_vpksg:
9640:     case Intrinsic::s390_vpkshs:  // PACKS_CC
9641:     case Intrinsic::s390_vpksfs:
9642:     case Intrinsic::s390_vpksgs:
9643:     case Intrinsic::s390_vpklsh:  // PACKLS
9644:     case Intrinsic::s390_vpklsf:
9645:     case Intrinsic::s390_vpklsg:
9646:     case Intrinsic::s390_vpklshs: // PACKLS_CC
9647:     case Intrinsic::s390_vpklsfs:
9648:     case Intrinsic::s390_vpklsgs:
9649:     case Intrinsic::s390_vpdi:
9650:     case Intrinsic::s390_vsldb:
9651:     case Intrinsic::s390_vperm:
9652:       return computeNumSignBitsBinOp(Op, DemandedElts, DAG, Depth, 1);
9653:     case Intrinsic::s390_vuphb:  // VECTOR UNPACK HIGH
9654:     case Intrinsic::s390_vuphh:
9655:     case Intrinsic::s390_vuphf:
9656:     case Intrinsic::s390_vuplb:  // VECTOR UNPACK LOW
9657:     case Intrinsic::s390_vuplhw:
9658:     case Intrinsic::s390_vuplf: {
9659:       SDValue PackedOp = Op.getOperand(1);
9660:       APInt SrcDemE = getDemandedSrcElements(Op, DemandedElts, 1);
9661:       unsigned Tmp = DAG.ComputeNumSignBits(PackedOp, SrcDemE, Depth + 1);
9662:       EVT VT = Op.getValueType();
9663:       unsigned VTBits = VT.getScalarSizeInBits();
9664:       Tmp += VTBits - PackedOp.getScalarValueSizeInBits();
9665:       return Tmp;
9666:     }
9667:     default:
9668:       break;
9669:     }
9670:   } else {
9671:     switch (Opcode) {
9672:     case SystemZISD::SELECT_CCMASK:
9673:       return computeNumSignBitsBinOp(Op, DemandedElts, DAG, Depth, 0);
9674:     default:
9675:       break;
```
- **EN**: The range implements or declares functions including `computeNumSignBitsBinOp`, `SystemZTargetLowering::ComputeNumSignBitsForTargetNode`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `computeNumSignBitsBinOp`, `SystemZTargetLowering::ComputeNumSignBitsForTargetNode` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 9676-9750 / 第 9676-9750 行
```cpp
9676:     }
9677:   }
9678: 
9679:   return 1;
9680: }
9681: 
9682: bool SystemZTargetLowering::isGuaranteedNotToBeUndefOrPoisonForTargetNode(
9683:     SDValue Op, const APInt &DemandedElts, const SelectionDAG &DAG,
9684:     UndefPoisonKind Kind, unsigned Depth) const {
9685:   switch (Op->getOpcode()) {
9686:   case SystemZISD::PCREL_WRAPPER:
9687:   case SystemZISD::PCREL_OFFSET:
9688:     return true;
9689:   }
9690:   return false;
9691: }
9692: 
9693: unsigned
9694: SystemZTargetLowering::getStackProbeSize(const MachineFunction &MF) const {
9695:   const TargetFrameLowering *TFI = Subtarget.getFrameLowering();
9696:   unsigned StackAlign = TFI->getStackAlignment();
9697:   assert(StackAlign >=1 && isPowerOf2_32(StackAlign) &&
9698:          "Unexpected stack alignment");
9699:   // The default stack probe size is 4096 if the function has no
9700:   // stack-probe-size attribute.
9701:   unsigned StackProbeSize =
9702:       MF.getFunction().getFnAttributeAsParsedInteger("stack-probe-size", 4096);
9703:   // Round down to the stack alignment.
9704:   StackProbeSize &= ~(StackAlign - 1);
9705:   return StackProbeSize ? StackProbeSize : StackAlign;
9706: }
9707: 
9708: //===----------------------------------------------------------------------===//
9709: // Custom insertion
9710: //===----------------------------------------------------------------------===//
9711: 
9712: // Force base value Base into a register before MI.  Return the register.
9713: static Register forceReg(MachineInstr &MI, MachineOperand &Base,
9714:                          const SystemZInstrInfo *TII) {
9715:   MachineBasicBlock *MBB = MI.getParent();
9716:   MachineFunction &MF = *MBB->getParent();
9717:   MachineRegisterInfo &MRI = MF.getRegInfo();
9718: 
9719:   if (Base.isReg()) {
9720:     // Copy Base into a new virtual register to help register coalescing in
9721:     // cases with multiple uses.
9722:     Register Reg = MRI.createVirtualRegister(&SystemZ::ADDR64BitRegClass);
9723:     BuildMI(*MBB, MI, MI.getDebugLoc(), TII->get(SystemZ::COPY), Reg)
9724:       .add(Base);
9725:     return Reg;
9726:   }
9727: 
9728:   Register Reg = MRI.createVirtualRegister(&SystemZ::ADDR64BitRegClass);
9729:   BuildMI(*MBB, MI, MI.getDebugLoc(), TII->get(SystemZ::LA), Reg)
9730:       .add(Base)
9731:       .addImm(0)
9732:       .addReg(0);
9733:   return Reg;
9734: }
9735: 
9736: // The CC operand of MI might be missing a kill marker because there
9737: // were multiple uses of CC, and ISel didn't know which to mark.
9738: // Figure out whether MI should have had a kill marker.
9739: static bool checkCCKill(MachineInstr &MI, MachineBasicBlock *MBB) {
9740:   // Scan forward through BB for a use/def of CC.
9741:   MachineBasicBlock::iterator miI(std::next(MachineBasicBlock::iterator(MI)));
9742:   for (MachineBasicBlock::iterator miE = MBB->end(); miI != miE; ++miI) {
9743:     const MachineInstr &MI = *miI;
9744:     if (MI.readsRegister(SystemZ::CC, /*TRI=*/nullptr))
9745:       return false;
9746:     if (MI.definesRegister(SystemZ::CC, /*TRI=*/nullptr))
9747:       break; // Should have kill-flag - update below.
9748:   }
9749: 
9750:   // If we hit the end of the block, check whether CC is live into a
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::isGuaranteedNotToBeUndefOrPoisonForTargetNode`, `SystemZTargetLowering::getStackProbeSize`, `forceReg`, `BuildMI`, `BuildMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::isGuaranteedNotToBeUndefOrPoisonForTargetNode`, `SystemZTargetLowering::getStackProbeSize`, `forceReg`, `BuildMI`, `BuildMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 9751-9825 / 第 9751-9825 行
```cpp
9751:   // successor.
9752:   if (miI == MBB->end()) {
9753:     for (const MachineBasicBlock *Succ : MBB->successors())
9754:       if (Succ->isLiveIn(SystemZ::CC))
9755:         return false;
9756:   }
9757: 
9758:   return true;
9759: }
9760: 
9761: // Return true if it is OK for this Select pseudo-opcode to be cascaded
9762: // together with other Select pseudo-opcodes into a single basic-block with
9763: // a conditional jump around it.
9764: static bool isSelectPseudo(MachineInstr &MI) {
9765:   switch (MI.getOpcode()) {
9766:   case SystemZ::Select32:
9767:   case SystemZ::Select64:
9768:   case SystemZ::Select128:
9769:   case SystemZ::SelectF32:
9770:   case SystemZ::SelectF64:
9771:   case SystemZ::SelectF128:
9772:   case SystemZ::SelectVR32:
9773:   case SystemZ::SelectVR64:
9774:   case SystemZ::SelectVR128:
9775:     return true;
9776: 
9777:   default:
9778:     return false;
9779:   }
9780: }
9781: 
9782: // Helper function, which inserts PHI functions into SinkMBB:
9783: //   %Result(i) = phi [ %FalseValue(i), FalseMBB ], [ %TrueValue(i), TrueMBB ],
9784: // where %FalseValue(i) and %TrueValue(i) are taken from Selects.
9785: static void createPHIsForSelects(SmallVector<MachineInstr*, 8> &Selects,
9786:                                  MachineBasicBlock *TrueMBB,
9787:                                  MachineBasicBlock *FalseMBB,
9788:                                  MachineBasicBlock *SinkMBB) {
9789:   MachineFunction *MF = TrueMBB->getParent();
9790:   const TargetInstrInfo *TII = MF->getSubtarget().getInstrInfo();
9791: 
9792:   MachineInstr *FirstMI = Selects.front();
9793:   unsigned CCValid = FirstMI->getOperand(3).getImm();
9794:   unsigned CCMask = FirstMI->getOperand(4).getImm();
9795: 
9796:   MachineBasicBlock::iterator SinkInsertionPoint = SinkMBB->begin();
9797: 
9798:   // As we are creating the PHIs, we have to be careful if there is more than
9799:   // one.  Later Selects may reference the results of earlier Selects, but later
9800:   // PHIs have to reference the individual true/false inputs from earlier PHIs.
9801:   // That also means that PHI construction must work forward from earlier to
9802:   // later, and that the code must maintain a mapping from earlier PHI's
9803:   // destination registers, and the registers that went into the PHI.
9804:   DenseMap<unsigned, std::pair<unsigned, unsigned>> RegRewriteTable;
9805: 
9806:   for (auto *MI : Selects) {
9807:     Register DestReg = MI->getOperand(0).getReg();
9808:     Register TrueReg = MI->getOperand(1).getReg();
9809:     Register FalseReg = MI->getOperand(2).getReg();
9810: 
9811:     // If this Select we are generating is the opposite condition from
9812:     // the jump we generated, then we have to swap the operands for the
9813:     // PHI that is going to be generated.
9814:     if (MI->getOperand(4).getImm() == (CCValid ^ CCMask))
9815:       std::swap(TrueReg, FalseReg);
9816: 
9817:     if (auto It = RegRewriteTable.find(TrueReg); It != RegRewriteTable.end())
9818:       TrueReg = It->second.first;
9819: 
9820:     if (auto It = RegRewriteTable.find(FalseReg); It != RegRewriteTable.end())
9821:       FalseReg = It->second.second;
9822: 
9823:     DebugLoc DL = MI->getDebugLoc();
9824:     BuildMI(*SinkMBB, SinkInsertionPoint, DL, TII->get(SystemZ::PHI), DestReg)
9825:       .addReg(TrueReg).addMBB(TrueMBB)
```
- **EN**: The range implements or declares functions including `isSelectPseudo`, `createPHIsForSelects`, `BuildMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isSelectPseudo`, `createPHIsForSelects`, `BuildMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 9826-9900 / 第 9826-9900 行
```cpp
9826:       .addReg(FalseReg).addMBB(FalseMBB);
9827: 
9828:     // Add this PHI to the rewrite table.
9829:     RegRewriteTable[DestReg] = std::make_pair(TrueReg, FalseReg);
9830:   }
9831: 
9832:   MF->getProperties().resetNoPHIs();
9833: }
9834: 
9835: MachineBasicBlock *
9836: SystemZTargetLowering::emitAdjCallStack(MachineInstr &MI,
9837:                                         MachineBasicBlock *BB) const {
9838:   MachineFunction &MF = *BB->getParent();
9839:   MachineFrameInfo &MFI = MF.getFrameInfo();
9840:   auto *TFL = Subtarget.getFrameLowering<SystemZFrameLowering>();
9841:   assert(TFL->hasReservedCallFrame(MF) &&
9842:          "ADJSTACKDOWN and ADJSTACKUP should be no-ops");
9843:   (void)TFL;
9844:   // Get the MaxCallFrameSize value and erase MI since it serves no further
9845:   // purpose as the call frame is statically reserved in the prolog. Set
9846:   // AdjustsStack as MI is *not* mapped as a frame instruction.
9847:   uint32_t NumBytes = MI.getOperand(0).getImm();
9848:   if (NumBytes > MFI.getMaxCallFrameSize())
9849:     MFI.setMaxCallFrameSize(NumBytes);
9850:   MFI.setAdjustsStack(true);
9851: 
9852:   MI.eraseFromParent();
9853:   return BB;
9854: }
9855: 
9856: // Implement EmitInstrWithCustomInserter for pseudo Select* instruction MI.
9857: MachineBasicBlock *
9858: SystemZTargetLowering::emitSelect(MachineInstr &MI,
9859:                                   MachineBasicBlock *MBB) const {
9860:   assert(isSelectPseudo(MI) && "Bad call to emitSelect()");
9861:   const SystemZInstrInfo *TII = Subtarget.getInstrInfo();
9862: 
9863:   unsigned CCValid = MI.getOperand(3).getImm();
9864:   unsigned CCMask = MI.getOperand(4).getImm();
9865: 
9866:   // If we have a sequence of Select* pseudo instructions using the
9867:   // same condition code value, we want to expand all of them into
9868:   // a single pair of basic blocks using the same condition.
9869:   SmallVector<MachineInstr*, 8> Selects;
9870:   SmallVector<MachineInstr*, 8> DbgValues;
9871:   Selects.push_back(&MI);
9872:   unsigned Count = 0;
9873:   for (MachineInstr &NextMI : llvm::make_range(
9874:            std::next(MachineBasicBlock::iterator(MI)), MBB->end())) {
9875:     if (isSelectPseudo(NextMI)) {
9876:       assert(NextMI.getOperand(3).getImm() == CCValid &&
9877:              "Bad CCValid operands since CC was not redefined.");
9878:       if (NextMI.getOperand(4).getImm() == CCMask ||
9879:           NextMI.getOperand(4).getImm() == (CCValid ^ CCMask)) {
9880:         Selects.push_back(&NextMI);
9881:         continue;
9882:       }
9883:       break;
9884:     }
9885:     if (NextMI.definesRegister(SystemZ::CC, /*TRI=*/nullptr) ||
9886:         NextMI.usesCustomInsertionHook())
9887:       break;
9888:     bool User = false;
9889:     for (auto *SelMI : Selects)
9890:       if (NextMI.readsVirtualRegister(SelMI->getOperand(0).getReg())) {
9891:         User = true;
9892:         break;
9893:       }
9894:     if (NextMI.isDebugInstr()) {
9895:       if (User) {
9896:         assert(NextMI.isDebugValue() && "Unhandled debug opcode.");
9897:         DbgValues.push_back(&NextMI);
9898:       }
9899:     } else if (User || ++Count > 20)
9900:       break;
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::emitAdjCallStack`, `SystemZTargetLowering::emitSelect`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::emitAdjCallStack`, `SystemZTargetLowering::emitSelect` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 9901-9975 / 第 9901-9975 行
```cpp
9901:   }
9902: 
9903:   MachineInstr *LastMI = Selects.back();
9904:   bool CCKilled = (LastMI->killsRegister(SystemZ::CC, /*TRI=*/nullptr) ||
9905:                    checkCCKill(*LastMI, MBB));
9906:   MachineBasicBlock *StartMBB = MBB;
9907:   MachineBasicBlock *JoinMBB  = SystemZ::splitBlockAfter(LastMI, MBB);
9908:   MachineBasicBlock *FalseMBB = SystemZ::emitBlockAfter(StartMBB);
9909: 
9910:   // Unless CC was killed in the last Select instruction, mark it as
9911:   // live-in to both FalseMBB and JoinMBB.
9912:   if (!CCKilled) {
9913:     FalseMBB->addLiveIn(SystemZ::CC);
9914:     JoinMBB->addLiveIn(SystemZ::CC);
9915:   }
9916: 
9917:   //  StartMBB:
9918:   //   BRC CCMask, JoinMBB
9919:   //   # fallthrough to FalseMBB
9920:   MBB = StartMBB;
9921:   BuildMI(MBB, MI.getDebugLoc(), TII->get(SystemZ::BRC))
9922:     .addImm(CCValid).addImm(CCMask).addMBB(JoinMBB);
9923:   MBB->addSuccessor(JoinMBB);
9924:   MBB->addSuccessor(FalseMBB);
9925: 
9926:   //  FalseMBB:
9927:   //   # fallthrough to JoinMBB
9928:   MBB = FalseMBB;
9929:   MBB->addSuccessor(JoinMBB);
9930: 
9931:   //  JoinMBB:
9932:   //   %Result = phi [ %FalseReg, FalseMBB ], [ %TrueReg, StartMBB ]
9933:   //  ...
9934:   MBB = JoinMBB;
9935:   createPHIsForSelects(Selects, StartMBB, FalseMBB, MBB);
9936:   for (auto *SelMI : Selects)
9937:     SelMI->eraseFromParent();
9938: 
9939:   MachineBasicBlock::iterator InsertPos = MBB->getFirstNonPHI();
9940:   for (auto *DbgMI : DbgValues)
9941:     MBB->splice(InsertPos, StartMBB, DbgMI);
9942: 
9943:   return JoinMBB;
9944: }
9945: 
9946: // Implement EmitInstrWithCustomInserter for pseudo CondStore* instruction MI.
9947: // StoreOpcode is the store to use and Invert says whether the store should
9948: // happen when the condition is false rather than true.  If a STORE ON
9949: // CONDITION is available, STOCOpcode is its opcode, otherwise it is 0.
9950: MachineBasicBlock *SystemZTargetLowering::emitCondStore(MachineInstr &MI,
9951:                                                         MachineBasicBlock *MBB,
9952:                                                         unsigned StoreOpcode,
9953:                                                         unsigned STOCOpcode,
9954:                                                         bool Invert) const {
9955:   const SystemZInstrInfo *TII = Subtarget.getInstrInfo();
9956: 
9957:   Register SrcReg = MI.getOperand(0).getReg();
9958:   MachineOperand Base = MI.getOperand(1);
9959:   int64_t Disp = MI.getOperand(2).getImm();
9960:   Register IndexReg = MI.getOperand(3).getReg();
9961:   unsigned CCValid = MI.getOperand(4).getImm();
9962:   unsigned CCMask = MI.getOperand(5).getImm();
9963:   DebugLoc DL = MI.getDebugLoc();
9964: 
9965:   StoreOpcode = TII->getOpcodeForOffset(StoreOpcode, Disp);
9966: 
9967:   // ISel pattern matching also adds a load memory operand of the same
9968:   // address, so take special care to find the storing memory operand.
9969:   MachineMemOperand *MMO = nullptr;
9970:   for (auto *I : MI.memoperands())
9971:     if (I->isStore()) {
9972:       MMO = I;
9973:       break;
9974:     }
9975: 
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 9976-10050 / 第 9976-10050 行
```cpp
9976:   // Use STOCOpcode if possible.  We could use different store patterns in
9977:   // order to avoid matching the index register, but the performance trade-offs
9978:   // might be more complicated in that case.
9979:   if (STOCOpcode && !IndexReg && Subtarget.hasLoadStoreOnCond()) {
9980:     if (Invert)
9981:       CCMask ^= CCValid;
9982: 
9983:     BuildMI(*MBB, MI, DL, TII->get(STOCOpcode))
9984:       .addReg(SrcReg)
9985:       .add(Base)
9986:       .addImm(Disp)
9987:       .addImm(CCValid)
9988:       .addImm(CCMask)
9989:       .addMemOperand(MMO);
9990: 
9991:     MI.eraseFromParent();
9992:     return MBB;
9993:   }
9994: 
9995:   // Get the condition needed to branch around the store.
9996:   if (!Invert)
9997:     CCMask ^= CCValid;
9998: 
9999:   MachineBasicBlock *StartMBB = MBB;
10000:   MachineBasicBlock *JoinMBB  = SystemZ::splitBlockBefore(MI, MBB);
10001:   MachineBasicBlock *FalseMBB = SystemZ::emitBlockAfter(StartMBB);
10002: 
10003:   // Unless CC was killed in the CondStore instruction, mark it as
10004:   // live-in to both FalseMBB and JoinMBB.
10005:   if (!MI.killsRegister(SystemZ::CC, /*TRI=*/nullptr) &&
10006:       !checkCCKill(MI, JoinMBB)) {
10007:     FalseMBB->addLiveIn(SystemZ::CC);
10008:     JoinMBB->addLiveIn(SystemZ::CC);
10009:   }
10010: 
10011:   //  StartMBB:
10012:   //   BRC CCMask, JoinMBB
10013:   //   # fallthrough to FalseMBB
10014:   MBB = StartMBB;
10015:   BuildMI(MBB, DL, TII->get(SystemZ::BRC))
10016:     .addImm(CCValid).addImm(CCMask).addMBB(JoinMBB);
10017:   MBB->addSuccessor(JoinMBB);
10018:   MBB->addSuccessor(FalseMBB);
10019: 
10020:   //  FalseMBB:
10021:   //   store %SrcReg, %Disp(%Index,%Base)
10022:   //   # fallthrough to JoinMBB
10023:   MBB = FalseMBB;
10024:   BuildMI(MBB, DL, TII->get(StoreOpcode))
10025:       .addReg(SrcReg)
10026:       .add(Base)
10027:       .addImm(Disp)
10028:       .addReg(IndexReg)
10029:       .addMemOperand(MMO);
10030:   MBB->addSuccessor(JoinMBB);
10031: 
10032:   MI.eraseFromParent();
10033:   return JoinMBB;
10034: }
10035: 
10036: // Implement EmitInstrWithCustomInserter for pseudo [SU]Cmp128Hi instruction MI.
10037: MachineBasicBlock *
10038: SystemZTargetLowering::emitICmp128Hi(MachineInstr &MI,
10039:                                      MachineBasicBlock *MBB,
10040:                                      bool Unsigned) const {
10041:   MachineFunction &MF = *MBB->getParent();
10042:   const SystemZInstrInfo *TII = Subtarget.getInstrInfo();
10043:   MachineRegisterInfo &MRI = MF.getRegInfo();
10044: 
10045:   // Synthetic instruction to compare 128-bit values.
10046:   // Sets CC 1 if Op0 > Op1, sets a different CC otherwise.
10047:   Register Op0 = MI.getOperand(0).getReg();
10048:   Register Op1 = MI.getOperand(1).getReg();
10049: 
10050:   MachineBasicBlock *StartMBB = MBB;
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `SystemZTargetLowering::emitICmp128Hi`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `SystemZTargetLowering::emitICmp128Hi` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10051-10125 / 第 10051-10125 行
```cpp
10051:   MachineBasicBlock *JoinMBB  = SystemZ::splitBlockAfter(MI, MBB);
10052:   MachineBasicBlock *HiEqMBB = SystemZ::emitBlockAfter(StartMBB);
10053: 
10054:   //  StartMBB:
10055:   //
10056:   //  Use VECTOR ELEMENT COMPARE [LOGICAL] to compare the high parts.
10057:   //  Swap the inputs to get:
10058:   //    CC 1 if high(Op0) > high(Op1)
10059:   //    CC 2 if high(Op0) < high(Op1)
10060:   //    CC 0 if high(Op0) == high(Op1)
10061:   //
10062:   //  If CC != 0, we'd done, so jump over the next instruction.
10063:   //
10064:   //   VEC[L]G Op1, Op0
10065:   //   JNE JoinMBB
10066:   //   # fallthrough to HiEqMBB
10067:   MBB = StartMBB;
10068:   int HiOpcode = Unsigned? SystemZ::VECLG : SystemZ::VECG;
10069:   BuildMI(MBB, MI.getDebugLoc(), TII->get(HiOpcode))
10070:     .addReg(Op1).addReg(Op0);
10071:   BuildMI(MBB, MI.getDebugLoc(), TII->get(SystemZ::BRC))
10072:     .addImm(SystemZ::CCMASK_ICMP).addImm(SystemZ::CCMASK_CMP_NE).addMBB(JoinMBB);
10073:   MBB->addSuccessor(JoinMBB);
10074:   MBB->addSuccessor(HiEqMBB);
10075: 
10076:   //  HiEqMBB:
10077:   //
10078:   //  Otherwise, use VECTOR COMPARE HIGH LOGICAL.
10079:   //  Since we already know the high parts are equal, the CC
10080:   //  result will only depend on the low parts:
10081:   //     CC 1 if low(Op0) > low(Op1)
10082:   //     CC 3 if low(Op0) <= low(Op1)
10083:   //
10084:   //   VCHLGS Tmp, Op0, Op1
10085:   //   # fallthrough to JoinMBB
10086:   MBB = HiEqMBB;
10087:   Register Temp = MRI.createVirtualRegister(&SystemZ::VR128BitRegClass);
10088:   BuildMI(MBB, MI.getDebugLoc(), TII->get(SystemZ::VCHLGS), Temp)
10089:     .addReg(Op0).addReg(Op1);
10090:   MBB->addSuccessor(JoinMBB);
10091: 
10092:   // Mark CC as live-in to JoinMBB.
10093:   JoinMBB->addLiveIn(SystemZ::CC);
10094: 
10095:   MI.eraseFromParent();
10096:   return JoinMBB;
10097: }
10098: 
10099: // Implement EmitInstrWithCustomInserter for subword pseudo ATOMIC_LOADW_* or
10100: // ATOMIC_SWAPW instruction MI.  BinOpcode is the instruction that performs
10101: // the binary operation elided by "*", or 0 for ATOMIC_SWAPW.  Invert says
10102: // whether the field should be inverted after performing BinOpcode (e.g. for
10103: // NAND).
10104: MachineBasicBlock *SystemZTargetLowering::emitAtomicLoadBinary(
10105:     MachineInstr &MI, MachineBasicBlock *MBB, unsigned BinOpcode,
10106:     bool Invert) const {
10107:   MachineFunction &MF = *MBB->getParent();
10108:   const SystemZInstrInfo *TII = Subtarget.getInstrInfo();
10109:   MachineRegisterInfo &MRI = MF.getRegInfo();
10110: 
10111:   // Extract the operands.  Base can be a register or a frame index.
10112:   // Src2 can be a register or immediate.
10113:   Register Dest = MI.getOperand(0).getReg();
10114:   MachineOperand Base = earlyUseOperand(MI.getOperand(1));
10115:   int64_t Disp = MI.getOperand(2).getImm();
10116:   MachineOperand Src2 = earlyUseOperand(MI.getOperand(3));
10117:   Register BitShift = MI.getOperand(4).getReg();
10118:   Register NegBitShift = MI.getOperand(5).getReg();
10119:   unsigned BitSize = MI.getOperand(6).getImm();
10120:   DebugLoc DL = MI.getDebugLoc();
10121: 
10122:   // Get the right opcodes for the displacement.
10123:   unsigned LOpcode  = TII->getOpcodeForOffset(SystemZ::L,  Disp);
10124:   unsigned CSOpcode = TII->getOpcodeForOffset(SystemZ::CS, Disp);
10125:   assert(LOpcode && CSOpcode && "Displacement out of range");
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 10126-10200 / 第 10126-10200 行
```cpp
10126: 
10127:   // Create virtual registers for temporary results.
10128:   Register OrigVal       = MRI.createVirtualRegister(&SystemZ::GR32BitRegClass);
10129:   Register OldVal        = MRI.createVirtualRegister(&SystemZ::GR32BitRegClass);
10130:   Register NewVal        = MRI.createVirtualRegister(&SystemZ::GR32BitRegClass);
10131:   Register RotatedOldVal = MRI.createVirtualRegister(&SystemZ::GR32BitRegClass);
10132:   Register RotatedNewVal = MRI.createVirtualRegister(&SystemZ::GR32BitRegClass);
10133: 
10134:   // Insert a basic block for the main loop.
10135:   MachineBasicBlock *StartMBB = MBB;
10136:   MachineBasicBlock *DoneMBB  = SystemZ::splitBlockBefore(MI, MBB);
10137:   MachineBasicBlock *LoopMBB  = SystemZ::emitBlockAfter(StartMBB);
10138: 
10139:   //  StartMBB:
10140:   //   ...
10141:   //   %OrigVal = L Disp(%Base)
10142:   //   # fall through to LoopMBB
10143:   MBB = StartMBB;
10144:   BuildMI(MBB, DL, TII->get(LOpcode), OrigVal).add(Base).addImm(Disp).addReg(0);
10145:   MBB->addSuccessor(LoopMBB);
10146: 
10147:   //  LoopMBB:
10148:   //   %OldVal        = phi [ %OrigVal, StartMBB ], [ %Dest, LoopMBB ]
10149:   //   %RotatedOldVal = RLL %OldVal, 0(%BitShift)
10150:   //   %RotatedNewVal = OP %RotatedOldVal, %Src2
10151:   //   %NewVal        = RLL %RotatedNewVal, 0(%NegBitShift)
10152:   //   %Dest          = CS %OldVal, %NewVal, Disp(%Base)
10153:   //   JNE LoopMBB
10154:   //   # fall through to DoneMBB
10155:   MBB = LoopMBB;
10156:   BuildMI(MBB, DL, TII->get(SystemZ::PHI), OldVal)
10157:     .addReg(OrigVal).addMBB(StartMBB)
10158:     .addReg(Dest).addMBB(LoopMBB);
10159:   BuildMI(MBB, DL, TII->get(SystemZ::RLL), RotatedOldVal)
10160:     .addReg(OldVal).addReg(BitShift).addImm(0);
10161:   if (Invert) {
10162:     // Perform the operation normally and then invert every bit of the field.
10163:     Register Tmp = MRI.createVirtualRegister(&SystemZ::GR32BitRegClass);
10164:     BuildMI(MBB, DL, TII->get(BinOpcode), Tmp).addReg(RotatedOldVal).add(Src2);
10165:     // XILF with the upper BitSize bits set.
10166:     BuildMI(MBB, DL, TII->get(SystemZ::XILF), RotatedNewVal)
10167:       .addReg(Tmp).addImm(-1U << (32 - BitSize));
10168:   } else if (BinOpcode)
10169:     // A simply binary operation.
10170:     BuildMI(MBB, DL, TII->get(BinOpcode), RotatedNewVal)
10171:         .addReg(RotatedOldVal)
10172:         .add(Src2);
10173:   else
10174:     // Use RISBG to rotate Src2 into position and use it to replace the
10175:     // field in RotatedOldVal.
10176:     BuildMI(MBB, DL, TII->get(SystemZ::RISBG32), RotatedNewVal)
10177:       .addReg(RotatedOldVal).addReg(Src2.getReg())
10178:       .addImm(32).addImm(31 + BitSize).addImm(32 - BitSize);
10179:   BuildMI(MBB, DL, TII->get(SystemZ::RLL), NewVal)
10180:     .addReg(RotatedNewVal).addReg(NegBitShift).addImm(0);
10181:   BuildMI(MBB, DL, TII->get(CSOpcode), Dest)
10182:       .addReg(OldVal)
10183:       .addReg(NewVal)
10184:       .add(Base)
10185:       .addImm(Disp);
10186:   BuildMI(MBB, DL, TII->get(SystemZ::BRC))
10187:     .addImm(SystemZ::CCMASK_CS).addImm(SystemZ::CCMASK_CS_NE).addMBB(LoopMBB);
10188:   MBB->addSuccessor(LoopMBB);
10189:   MBB->addSuccessor(DoneMBB);
10190: 
10191:   MI.eraseFromParent();
10192:   return DoneMBB;
10193: }
10194: 
10195: // Implement EmitInstrWithCustomInserter for subword pseudo
10196: // ATOMIC_LOADW_{,U}{MIN,MAX} instruction MI.  CompareOpcode is the
10197: // instruction that should be used to compare the current field with the
10198: // minimum or maximum value.  KeepOldMask is the BRC condition-code mask
10199: // for when the current field should be kept.
10200: MachineBasicBlock *SystemZTargetLowering::emitAtomicLoadMinMax(
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10201-10275 / 第 10201-10275 行
```cpp
10201:     MachineInstr &MI, MachineBasicBlock *MBB, unsigned CompareOpcode,
10202:     unsigned KeepOldMask) const {
10203:   MachineFunction &MF = *MBB->getParent();
10204:   const SystemZInstrInfo *TII = Subtarget.getInstrInfo();
10205:   MachineRegisterInfo &MRI = MF.getRegInfo();
10206: 
10207:   // Extract the operands.  Base can be a register or a frame index.
10208:   Register Dest = MI.getOperand(0).getReg();
10209:   MachineOperand Base = earlyUseOperand(MI.getOperand(1));
10210:   int64_t Disp = MI.getOperand(2).getImm();
10211:   Register Src2 = MI.getOperand(3).getReg();
10212:   Register BitShift = MI.getOperand(4).getReg();
10213:   Register NegBitShift = MI.getOperand(5).getReg();
10214:   unsigned BitSize = MI.getOperand(6).getImm();
10215:   DebugLoc DL = MI.getDebugLoc();
10216: 
10217:   // Get the right opcodes for the displacement.
10218:   unsigned LOpcode  = TII->getOpcodeForOffset(SystemZ::L,  Disp);
10219:   unsigned CSOpcode = TII->getOpcodeForOffset(SystemZ::CS, Disp);
10220:   assert(LOpcode && CSOpcode && "Displacement out of range");
10221: 
10222:   // Create virtual registers for temporary results.
10223:   Register OrigVal       = MRI.createVirtualRegister(&SystemZ::GR32BitRegClass);
10224:   Register OldVal        = MRI.createVirtualRegister(&SystemZ::GR32BitRegClass);
10225:   Register NewVal        = MRI.createVirtualRegister(&SystemZ::GR32BitRegClass);
10226:   Register RotatedOldVal = MRI.createVirtualRegister(&SystemZ::GR32BitRegClass);
10227:   Register RotatedAltVal = MRI.createVirtualRegister(&SystemZ::GR32BitRegClass);
10228:   Register RotatedNewVal = MRI.createVirtualRegister(&SystemZ::GR32BitRegClass);
10229: 
10230:   // Insert 3 basic blocks for the loop.
10231:   MachineBasicBlock *StartMBB  = MBB;
10232:   MachineBasicBlock *DoneMBB   = SystemZ::splitBlockBefore(MI, MBB);
10233:   MachineBasicBlock *LoopMBB   = SystemZ::emitBlockAfter(StartMBB);
10234:   MachineBasicBlock *UseAltMBB = SystemZ::emitBlockAfter(LoopMBB);
10235:   MachineBasicBlock *UpdateMBB = SystemZ::emitBlockAfter(UseAltMBB);
10236: 
10237:   //  StartMBB:
10238:   //   ...
10239:   //   %OrigVal     = L Disp(%Base)
10240:   //   # fall through to LoopMBB
10241:   MBB = StartMBB;
10242:   BuildMI(MBB, DL, TII->get(LOpcode), OrigVal).add(Base).addImm(Disp).addReg(0);
10243:   MBB->addSuccessor(LoopMBB);
10244: 
10245:   //  LoopMBB:
10246:   //   %OldVal        = phi [ %OrigVal, StartMBB ], [ %Dest, UpdateMBB ]
10247:   //   %RotatedOldVal = RLL %OldVal, 0(%BitShift)
10248:   //   CompareOpcode %RotatedOldVal, %Src2
10249:   //   BRC KeepOldMask, UpdateMBB
10250:   MBB = LoopMBB;
10251:   BuildMI(MBB, DL, TII->get(SystemZ::PHI), OldVal)
10252:     .addReg(OrigVal).addMBB(StartMBB)
10253:     .addReg(Dest).addMBB(UpdateMBB);
10254:   BuildMI(MBB, DL, TII->get(SystemZ::RLL), RotatedOldVal)
10255:     .addReg(OldVal).addReg(BitShift).addImm(0);
10256:   BuildMI(MBB, DL, TII->get(CompareOpcode))
10257:     .addReg(RotatedOldVal).addReg(Src2);
10258:   BuildMI(MBB, DL, TII->get(SystemZ::BRC))
10259:     .addImm(SystemZ::CCMASK_ICMP).addImm(KeepOldMask).addMBB(UpdateMBB);
10260:   MBB->addSuccessor(UpdateMBB);
10261:   MBB->addSuccessor(UseAltMBB);
10262: 
10263:   //  UseAltMBB:
10264:   //   %RotatedAltVal = RISBG %RotatedOldVal, %Src2, 32, 31 + BitSize, 0
10265:   //   # fall through to UpdateMBB
10266:   MBB = UseAltMBB;
10267:   BuildMI(MBB, DL, TII->get(SystemZ::RISBG32), RotatedAltVal)
10268:     .addReg(RotatedOldVal).addReg(Src2)
10269:     .addImm(32).addImm(31 + BitSize).addImm(0);
10270:   MBB->addSuccessor(UpdateMBB);
10271: 
10272:   //  UpdateMBB:
10273:   //   %RotatedNewVal = PHI [ %RotatedOldVal, LoopMBB ],
10274:   //                        [ %RotatedAltVal, UseAltMBB ]
10275:   //   %NewVal        = RLL %RotatedNewVal, 0(%NegBitShift)
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 10276-10350 / 第 10276-10350 行
```cpp
10276:   //   %Dest          = CS %OldVal, %NewVal, Disp(%Base)
10277:   //   JNE LoopMBB
10278:   //   # fall through to DoneMBB
10279:   MBB = UpdateMBB;
10280:   BuildMI(MBB, DL, TII->get(SystemZ::PHI), RotatedNewVal)
10281:     .addReg(RotatedOldVal).addMBB(LoopMBB)
10282:     .addReg(RotatedAltVal).addMBB(UseAltMBB);
10283:   BuildMI(MBB, DL, TII->get(SystemZ::RLL), NewVal)
10284:     .addReg(RotatedNewVal).addReg(NegBitShift).addImm(0);
10285:   BuildMI(MBB, DL, TII->get(CSOpcode), Dest)
10286:       .addReg(OldVal)
10287:       .addReg(NewVal)
10288:       .add(Base)
10289:       .addImm(Disp);
10290:   BuildMI(MBB, DL, TII->get(SystemZ::BRC))
10291:     .addImm(SystemZ::CCMASK_CS).addImm(SystemZ::CCMASK_CS_NE).addMBB(LoopMBB);
10292:   MBB->addSuccessor(LoopMBB);
10293:   MBB->addSuccessor(DoneMBB);
10294: 
10295:   MI.eraseFromParent();
10296:   return DoneMBB;
10297: }
10298: 
10299: // Implement EmitInstrWithCustomInserter for subword pseudo ATOMIC_CMP_SWAPW
10300: // instruction MI.
10301: MachineBasicBlock *
10302: SystemZTargetLowering::emitAtomicCmpSwapW(MachineInstr &MI,
10303:                                           MachineBasicBlock *MBB) const {
10304:   MachineFunction &MF = *MBB->getParent();
10305:   const SystemZInstrInfo *TII = Subtarget.getInstrInfo();
10306:   MachineRegisterInfo &MRI = MF.getRegInfo();
10307: 
10308:   // Extract the operands.  Base can be a register or a frame index.
10309:   Register Dest = MI.getOperand(0).getReg();
10310:   MachineOperand Base = earlyUseOperand(MI.getOperand(1));
10311:   int64_t Disp = MI.getOperand(2).getImm();
10312:   Register CmpVal = MI.getOperand(3).getReg();
10313:   Register OrigSwapVal = MI.getOperand(4).getReg();
10314:   Register BitShift = MI.getOperand(5).getReg();
10315:   Register NegBitShift = MI.getOperand(6).getReg();
10316:   int64_t BitSize = MI.getOperand(7).getImm();
10317:   DebugLoc DL = MI.getDebugLoc();
10318: 
10319:   const TargetRegisterClass *RC = &SystemZ::GR32BitRegClass;
10320: 
10321:   // Get the right opcodes for the displacement and zero-extension.
10322:   unsigned LOpcode  = TII->getOpcodeForOffset(SystemZ::L,  Disp);
10323:   unsigned CSOpcode = TII->getOpcodeForOffset(SystemZ::CS, Disp);
10324:   unsigned ZExtOpcode  = BitSize == 8 ? SystemZ::LLCR : SystemZ::LLHR;
10325:   assert(LOpcode && CSOpcode && "Displacement out of range");
10326: 
10327:   // Create virtual registers for temporary results.
10328:   Register OrigOldVal = MRI.createVirtualRegister(RC);
10329:   Register OldVal = MRI.createVirtualRegister(RC);
10330:   Register SwapVal = MRI.createVirtualRegister(RC);
10331:   Register StoreVal = MRI.createVirtualRegister(RC);
10332:   Register OldValRot = MRI.createVirtualRegister(RC);
10333:   Register RetryOldVal = MRI.createVirtualRegister(RC);
10334:   Register RetrySwapVal = MRI.createVirtualRegister(RC);
10335: 
10336:   // Insert 2 basic blocks for the loop.
10337:   MachineBasicBlock *StartMBB = MBB;
10338:   MachineBasicBlock *DoneMBB  = SystemZ::splitBlockBefore(MI, MBB);
10339:   MachineBasicBlock *LoopMBB  = SystemZ::emitBlockAfter(StartMBB);
10340:   MachineBasicBlock *SetMBB   = SystemZ::emitBlockAfter(LoopMBB);
10341: 
10342:   //  StartMBB:
10343:   //   ...
10344:   //   %OrigOldVal     = L Disp(%Base)
10345:   //   # fall through to LoopMBB
10346:   MBB = StartMBB;
10347:   BuildMI(MBB, DL, TII->get(LOpcode), OrigOldVal)
10348:       .add(Base)
10349:       .addImm(Disp)
10350:       .addReg(0);
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `SystemZTargetLowering::emitAtomicCmpSwapW`. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `SystemZTargetLowering::emitAtomicCmpSwapW` 等函数。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 10351-10425 / 第 10351-10425 行
```cpp
10351:   MBB->addSuccessor(LoopMBB);
10352: 
10353:   //  LoopMBB:
10354:   //   %OldVal        = phi [ %OrigOldVal, EntryBB ], [ %RetryOldVal, SetMBB ]
10355:   //   %SwapVal       = phi [ %OrigSwapVal, EntryBB ], [ %RetrySwapVal, SetMBB ]
10356:   //   %OldValRot     = RLL %OldVal, BitSize(%BitShift)
10357:   //                      ^^ The low BitSize bits contain the field
10358:   //                         of interest.
10359:   //   %RetrySwapVal = RISBG32 %SwapVal, %OldValRot, 32, 63-BitSize, 0
10360:   //                      ^^ Replace the upper 32-BitSize bits of the
10361:   //                         swap value with those that we loaded and rotated.
10362:   //   %Dest = LL[CH] %OldValRot
10363:   //   CR %Dest, %CmpVal
10364:   //   JNE DoneMBB
10365:   //   # Fall through to SetMBB
10366:   MBB = LoopMBB;
10367:   BuildMI(MBB, DL, TII->get(SystemZ::PHI), OldVal)
10368:     .addReg(OrigOldVal).addMBB(StartMBB)
10369:     .addReg(RetryOldVal).addMBB(SetMBB);
10370:   BuildMI(MBB, DL, TII->get(SystemZ::PHI), SwapVal)
10371:     .addReg(OrigSwapVal).addMBB(StartMBB)
10372:     .addReg(RetrySwapVal).addMBB(SetMBB);
10373:   BuildMI(MBB, DL, TII->get(SystemZ::RLL), OldValRot)
10374:     .addReg(OldVal).addReg(BitShift).addImm(BitSize);
10375:   BuildMI(MBB, DL, TII->get(SystemZ::RISBG32), RetrySwapVal)
10376:     .addReg(SwapVal).addReg(OldValRot).addImm(32).addImm(63 - BitSize).addImm(0);
10377:   BuildMI(MBB, DL, TII->get(ZExtOpcode), Dest)
10378:     .addReg(OldValRot);
10379:   BuildMI(MBB, DL, TII->get(SystemZ::CR))
10380:     .addReg(Dest).addReg(CmpVal);
10381:   BuildMI(MBB, DL, TII->get(SystemZ::BRC))
10382:     .addImm(SystemZ::CCMASK_ICMP)
10383:     .addImm(SystemZ::CCMASK_CMP_NE).addMBB(DoneMBB);
10384:   MBB->addSuccessor(DoneMBB);
10385:   MBB->addSuccessor(SetMBB);
10386: 
10387:   //  SetMBB:
10388:   //   %StoreVal     = RLL %RetrySwapVal, -BitSize(%NegBitShift)
10389:   //                      ^^ Rotate the new field to its proper position.
10390:   //   %RetryOldVal  = CS %OldVal, %StoreVal, Disp(%Base)
10391:   //   JNE LoopMBB
10392:   //   # fall through to ExitMBB
10393:   MBB = SetMBB;
10394:   BuildMI(MBB, DL, TII->get(SystemZ::RLL), StoreVal)
10395:     .addReg(RetrySwapVal).addReg(NegBitShift).addImm(-BitSize);
10396:   BuildMI(MBB, DL, TII->get(CSOpcode), RetryOldVal)
10397:       .addReg(OldVal)
10398:       .addReg(StoreVal)
10399:       .add(Base)
10400:       .addImm(Disp);
10401:   BuildMI(MBB, DL, TII->get(SystemZ::BRC))
10402:     .addImm(SystemZ::CCMASK_CS).addImm(SystemZ::CCMASK_CS_NE).addMBB(LoopMBB);
10403:   MBB->addSuccessor(LoopMBB);
10404:   MBB->addSuccessor(DoneMBB);
10405: 
10406:   // If the CC def wasn't dead in the ATOMIC_CMP_SWAPW, mark CC as live-in
10407:   // to the block after the loop.  At this point, CC may have been defined
10408:   // either by the CR in LoopMBB or by the CS in SetMBB.
10409:   if (!MI.registerDefIsDead(SystemZ::CC, /*TRI=*/nullptr))
10410:     DoneMBB->addLiveIn(SystemZ::CC);
10411: 
10412:   MI.eraseFromParent();
10413:   return DoneMBB;
10414: }
10415: 
10416: // Emit a move from two GR64s to a GR128.
10417: MachineBasicBlock *
10418: SystemZTargetLowering::emitPair128(MachineInstr &MI,
10419:                                    MachineBasicBlock *MBB) const {
10420:   const SystemZInstrInfo *TII = Subtarget.getInstrInfo();
10421:   const DebugLoc &DL = MI.getDebugLoc();
10422: 
10423:   Register Dest = MI.getOperand(0).getReg();
10424:   BuildMI(*MBB, MI, DL, TII->get(TargetOpcode::REG_SEQUENCE), Dest)
10425:       .add(MI.getOperand(1))
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10426-10500 / 第 10426-10500 行
```cpp
10426:       .addImm(SystemZ::subreg_h64)
10427:       .add(MI.getOperand(2))
10428:       .addImm(SystemZ::subreg_l64);
10429:   MI.eraseFromParent();
10430:   return MBB;
10431: }
10432: 
10433: // Emit an extension from a GR64 to a GR128.  ClearEven is true
10434: // if the high register of the GR128 value must be cleared or false if
10435: // it's "don't care".
10436: MachineBasicBlock *SystemZTargetLowering::emitExt128(MachineInstr &MI,
10437:                                                      MachineBasicBlock *MBB,
10438:                                                      bool ClearEven) const {
10439:   MachineFunction &MF = *MBB->getParent();
10440:   const SystemZInstrInfo *TII = Subtarget.getInstrInfo();
10441:   MachineRegisterInfo &MRI = MF.getRegInfo();
10442:   DebugLoc DL = MI.getDebugLoc();
10443: 
10444:   Register Dest = MI.getOperand(0).getReg();
10445:   Register Src = MI.getOperand(1).getReg();
10446:   Register In128 = MRI.createVirtualRegister(&SystemZ::GR128BitRegClass);
10447: 
10448:   BuildMI(*MBB, MI, DL, TII->get(TargetOpcode::IMPLICIT_DEF), In128);
10449:   if (ClearEven) {
10450:     Register NewIn128 = MRI.createVirtualRegister(&SystemZ::GR128BitRegClass);
10451:     Register Zero64 = MRI.createVirtualRegister(&SystemZ::GR64BitRegClass);
10452: 
10453:     BuildMI(*MBB, MI, DL, TII->get(SystemZ::LLILL), Zero64)
10454:       .addImm(0);
10455:     BuildMI(*MBB, MI, DL, TII->get(TargetOpcode::INSERT_SUBREG), NewIn128)
10456:       .addReg(In128).addReg(Zero64).addImm(SystemZ::subreg_h64);
10457:     In128 = NewIn128;
10458:   }
10459:   BuildMI(*MBB, MI, DL, TII->get(TargetOpcode::INSERT_SUBREG), Dest)
10460:     .addReg(In128).addReg(Src).addImm(SystemZ::subreg_l64);
10461: 
10462:   MI.eraseFromParent();
10463:   return MBB;
10464: }
10465: 
10466: MachineBasicBlock *
10467: SystemZTargetLowering::emitMemMemWrapper(MachineInstr &MI,
10468:                                          MachineBasicBlock *MBB,
10469:                                          unsigned Opcode, bool IsMemset) const {
10470:   MachineFunction &MF = *MBB->getParent();
10471:   const SystemZInstrInfo *TII = Subtarget.getInstrInfo();
10472:   MachineRegisterInfo &MRI = MF.getRegInfo();
10473:   DebugLoc DL = MI.getDebugLoc();
10474: 
10475:   MachineOperand DestBase = earlyUseOperand(MI.getOperand(0));
10476:   uint64_t DestDisp = MI.getOperand(1).getImm();
10477:   MachineOperand SrcBase = MachineOperand::CreateReg(0U, false);
10478:   uint64_t SrcDisp;
10479: 
10480:   // Fold the displacement Disp if it is out of range.
10481:   auto foldDisplIfNeeded = [&](MachineOperand &Base, uint64_t &Disp) -> void {
10482:     if (!isUInt<12>(Disp)) {
10483:       Register Reg = MRI.createVirtualRegister(&SystemZ::ADDR64BitRegClass);
10484:       unsigned Opcode = TII->getOpcodeForOffset(SystemZ::LA, Disp);
10485:       BuildMI(*MI.getParent(), MI, MI.getDebugLoc(), TII->get(Opcode), Reg)
10486:         .add(Base).addImm(Disp).addReg(0);
10487:       Base = MachineOperand::CreateReg(Reg, false);
10488:       Disp = 0;
10489:     }
10490:   };
10491: 
10492:   if (!IsMemset) {
10493:     SrcBase = earlyUseOperand(MI.getOperand(2));
10494:     SrcDisp = MI.getOperand(3).getImm();
10495:   } else {
10496:     SrcBase = DestBase;
10497:     SrcDisp = DestDisp++;
10498:     foldDisplIfNeeded(DestBase, DestDisp);
10499:   }
10500: 
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `SystemZTargetLowering::emitMemMemWrapper`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `SystemZTargetLowering::emitMemMemWrapper`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10501-10575 / 第 10501-10575 行
```cpp
10501:   MachineOperand &LengthMO = MI.getOperand(IsMemset ? 2 : 4);
10502:   bool IsImmForm = LengthMO.isImm();
10503:   bool IsRegForm = !IsImmForm;
10504: 
10505:   // Build and insert one Opcode of Length, with special treatment for memset.
10506:   auto insertMemMemOp = [&](MachineBasicBlock *InsMBB,
10507:                             MachineBasicBlock::iterator InsPos,
10508:                             MachineOperand DBase, uint64_t DDisp,
10509:                             MachineOperand SBase, uint64_t SDisp,
10510:                             unsigned Length) -> void {
10511:     assert(Length > 0 && Length <= 256 && "Building memory op with bad length.");
10512:     if (IsMemset) {
10513:       MachineOperand ByteMO = earlyUseOperand(MI.getOperand(3));
10514:       if (ByteMO.isImm())
10515:         BuildMI(*InsMBB, InsPos, DL, TII->get(SystemZ::MVI))
10516:           .add(SBase).addImm(SDisp).add(ByteMO);
10517:       else
10518:         BuildMI(*InsMBB, InsPos, DL, TII->get(SystemZ::STC))
10519:           .add(ByteMO).add(SBase).addImm(SDisp).addReg(0);
10520:       if (--Length == 0)
10521:         return;
10522:     }
10523:     BuildMI(*MBB, InsPos, DL, TII->get(Opcode))
10524:       .add(DBase).addImm(DDisp).addImm(Length)
10525:       .add(SBase).addImm(SDisp)
10526:       .setMemRefs(MI.memoperands());
10527:   };
10528: 
10529:   bool NeedsLoop = false;
10530:   uint64_t ImmLength = 0;
10531:   Register LenAdjReg = SystemZ::NoRegister;
10532:   if (IsImmForm) {
10533:     ImmLength = LengthMO.getImm();
10534:     ImmLength += IsMemset ? 2 : 1; // Add back the subtracted adjustment.
10535:     if (ImmLength == 0) {
10536:       MI.eraseFromParent();
10537:       return MBB;
10538:     }
10539:     if (Opcode == SystemZ::CLC) {
10540:       if (ImmLength > 3 * 256)
10541:         // A two-CLC sequence is a clear win over a loop, not least because
10542:         // it needs only one branch.  A three-CLC sequence needs the same
10543:         // number of branches as a loop (i.e. 2), but is shorter.  That
10544:         // brings us to lengths greater than 768 bytes.  It seems relatively
10545:         // likely that a difference will be found within the first 768 bytes,
10546:         // so we just optimize for the smallest number of branch
10547:         // instructions, in order to avoid polluting the prediction buffer
10548:         // too much.
10549:         NeedsLoop = true;
10550:     } else if (ImmLength > 6 * 256)
10551:       // The heuristic we use is to prefer loops for anything that would
10552:       // require 7 or more MVCs.  With these kinds of sizes there isn't much
10553:       // to choose between straight-line code and looping code, since the
10554:       // time will be dominated by the MVCs themselves.
10555:       NeedsLoop = true;
10556:   } else {
10557:     NeedsLoop = true;
10558:     LenAdjReg = LengthMO.getReg();
10559:   }
10560: 
10561:   // When generating more than one CLC, all but the last will need to
10562:   // branch to the end when a difference is found.
10563:   MachineBasicBlock *EndMBB =
10564:       (Opcode == SystemZ::CLC && (ImmLength > 256 || NeedsLoop)
10565:            ? SystemZ::splitBlockAfter(MI, MBB)
10566:            : nullptr);
10567: 
10568:   if (NeedsLoop) {
10569:     Register StartCountReg =
10570:       MRI.createVirtualRegister(&SystemZ::GR64BitRegClass);
10571:     if (IsImmForm) {
10572:       TII->loadImmediate(*MBB, MI, StartCountReg, ImmLength / 256);
10573:       ImmLength &= 255;
10574:     } else {
10575:       BuildMI(*MBB, MI, DL, TII->get(SystemZ::SRLG), StartCountReg)
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 10576-10650 / 第 10576-10650 行
```cpp
10576:         .addReg(LenAdjReg)
10577:         .addReg(0)
10578:         .addImm(8);
10579:     }
10580: 
10581:     bool HaveSingleBase = DestBase.isIdenticalTo(SrcBase);
10582:     auto loadZeroAddress = [&]() -> MachineOperand {
10583:       Register Reg = MRI.createVirtualRegister(&SystemZ::ADDR64BitRegClass);
10584:       BuildMI(*MBB, MI, DL, TII->get(SystemZ::LGHI), Reg).addImm(0);
10585:       return MachineOperand::CreateReg(Reg, false);
10586:     };
10587:     if (DestBase.isReg() && DestBase.getReg() == SystemZ::NoRegister)
10588:       DestBase = loadZeroAddress();
10589:     if (SrcBase.isReg() && SrcBase.getReg() == SystemZ::NoRegister)
10590:       SrcBase = HaveSingleBase ? DestBase : loadZeroAddress();
10591: 
10592:     MachineBasicBlock *StartMBB = nullptr;
10593:     MachineBasicBlock *LoopMBB = nullptr;
10594:     MachineBasicBlock *NextMBB = nullptr;
10595:     MachineBasicBlock *DoneMBB = nullptr;
10596:     MachineBasicBlock *AllDoneMBB = nullptr;
10597: 
10598:     Register StartSrcReg = forceReg(MI, SrcBase, TII);
10599:     Register StartDestReg =
10600:         (HaveSingleBase ? StartSrcReg : forceReg(MI, DestBase, TII));
10601: 
10602:     const TargetRegisterClass *RC = &SystemZ::ADDR64BitRegClass;
10603:     Register ThisSrcReg  = MRI.createVirtualRegister(RC);
10604:     Register ThisDestReg =
10605:         (HaveSingleBase ? ThisSrcReg : MRI.createVirtualRegister(RC));
10606:     Register NextSrcReg  = MRI.createVirtualRegister(RC);
10607:     Register NextDestReg =
10608:         (HaveSingleBase ? NextSrcReg : MRI.createVirtualRegister(RC));
10609:     RC = &SystemZ::GR64BitRegClass;
10610:     Register ThisCountReg = MRI.createVirtualRegister(RC);
10611:     Register NextCountReg = MRI.createVirtualRegister(RC);
10612: 
10613:     if (IsRegForm) {
10614:       AllDoneMBB = SystemZ::splitBlockBefore(MI, MBB);
10615:       StartMBB = SystemZ::emitBlockAfter(MBB);
10616:       LoopMBB = SystemZ::emitBlockAfter(StartMBB);
10617:       NextMBB = (EndMBB ? SystemZ::emitBlockAfter(LoopMBB) : LoopMBB);
10618:       DoneMBB = SystemZ::emitBlockAfter(NextMBB);
10619: 
10620:       //  MBB:
10621:       //   # Jump to AllDoneMBB if LenAdjReg means 0, or fall thru to StartMBB.
10622:       BuildMI(MBB, DL, TII->get(SystemZ::CGHI))
10623:         .addReg(LenAdjReg).addImm(IsMemset ? -2 : -1);
10624:       BuildMI(MBB, DL, TII->get(SystemZ::BRC))
10625:         .addImm(SystemZ::CCMASK_ICMP).addImm(SystemZ::CCMASK_CMP_EQ)
10626:         .addMBB(AllDoneMBB);
10627:       MBB->addSuccessor(AllDoneMBB);
10628:       if (!IsMemset)
10629:         MBB->addSuccessor(StartMBB);
10630:       else {
10631:         // MemsetOneCheckMBB:
10632:         // # Jump to MemsetOneMBB for a memset of length 1, or
10633:         // # fall thru to StartMBB.
10634:         MachineBasicBlock *MemsetOneCheckMBB = SystemZ::emitBlockAfter(MBB);
10635:         MachineBasicBlock *MemsetOneMBB = SystemZ::emitBlockAfter(&*MF.rbegin());
10636:         MBB->addSuccessor(MemsetOneCheckMBB);
10637:         MBB = MemsetOneCheckMBB;
10638:         BuildMI(MBB, DL, TII->get(SystemZ::CGHI))
10639:           .addReg(LenAdjReg).addImm(-1);
10640:         BuildMI(MBB, DL, TII->get(SystemZ::BRC))
10641:           .addImm(SystemZ::CCMASK_ICMP).addImm(SystemZ::CCMASK_CMP_EQ)
10642:           .addMBB(MemsetOneMBB);
10643:         MBB->addSuccessor(MemsetOneMBB, {10, 100});
10644:         MBB->addSuccessor(StartMBB, {90, 100});
10645: 
10646:         // MemsetOneMBB:
10647:         // # Jump back to AllDoneMBB after a single MVI or STC.
10648:         MBB = MemsetOneMBB;
10649:         insertMemMemOp(MBB, MBB->end(),
10650:                        MachineOperand::CreateReg(StartDestReg, false), DestDisp,
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10651-10725 / 第 10651-10725 行
```cpp
10651:                        MachineOperand::CreateReg(StartSrcReg, false), SrcDisp,
10652:                        1);
10653:         BuildMI(MBB, DL, TII->get(SystemZ::J)).addMBB(AllDoneMBB);
10654:         MBB->addSuccessor(AllDoneMBB);
10655:       }
10656: 
10657:       // StartMBB:
10658:       // # Jump to DoneMBB if %StartCountReg is zero, or fall through to LoopMBB.
10659:       MBB = StartMBB;
10660:       BuildMI(MBB, DL, TII->get(SystemZ::CGHI))
10661:         .addReg(StartCountReg).addImm(0);
10662:       BuildMI(MBB, DL, TII->get(SystemZ::BRC))
10663:         .addImm(SystemZ::CCMASK_ICMP).addImm(SystemZ::CCMASK_CMP_EQ)
10664:         .addMBB(DoneMBB);
10665:       MBB->addSuccessor(DoneMBB);
10666:       MBB->addSuccessor(LoopMBB);
10667:     }
10668:     else {
10669:       StartMBB = MBB;
10670:       DoneMBB = SystemZ::splitBlockBefore(MI, MBB);
10671:       LoopMBB = SystemZ::emitBlockAfter(StartMBB);
10672:       NextMBB = (EndMBB ? SystemZ::emitBlockAfter(LoopMBB) : LoopMBB);
10673: 
10674:       //  StartMBB:
10675:       //   # fall through to LoopMBB
10676:       MBB->addSuccessor(LoopMBB);
10677: 
10678:       DestBase = MachineOperand::CreateReg(NextDestReg, false);
10679:       SrcBase = MachineOperand::CreateReg(NextSrcReg, false);
10680:       if (EndMBB && !ImmLength)
10681:         // If the loop handled the whole CLC range, DoneMBB will be empty with
10682:         // CC live-through into EndMBB, so add it as live-in.
10683:         DoneMBB->addLiveIn(SystemZ::CC);
10684:     }
10685: 
10686:     //  LoopMBB:
10687:     //   %ThisDestReg = phi [ %StartDestReg, StartMBB ],
10688:     //                      [ %NextDestReg, NextMBB ]
10689:     //   %ThisSrcReg = phi [ %StartSrcReg, StartMBB ],
10690:     //                     [ %NextSrcReg, NextMBB ]
10691:     //   %ThisCountReg = phi [ %StartCountReg, StartMBB ],
10692:     //                       [ %NextCountReg, NextMBB ]
10693:     //   ( PFD 2, 768+DestDisp(%ThisDestReg) )
10694:     //   Opcode DestDisp(256,%ThisDestReg), SrcDisp(%ThisSrcReg)
10695:     //   ( JLH EndMBB )
10696:     //
10697:     // The prefetch is used only for MVC.  The JLH is used only for CLC.
10698:     MBB = LoopMBB;
10699:     BuildMI(MBB, DL, TII->get(SystemZ::PHI), ThisDestReg)
10700:       .addReg(StartDestReg).addMBB(StartMBB)
10701:       .addReg(NextDestReg).addMBB(NextMBB);
10702:     if (!HaveSingleBase)
10703:       BuildMI(MBB, DL, TII->get(SystemZ::PHI), ThisSrcReg)
10704:         .addReg(StartSrcReg).addMBB(StartMBB)
10705:         .addReg(NextSrcReg).addMBB(NextMBB);
10706:     BuildMI(MBB, DL, TII->get(SystemZ::PHI), ThisCountReg)
10707:       .addReg(StartCountReg).addMBB(StartMBB)
10708:       .addReg(NextCountReg).addMBB(NextMBB);
10709:     if (Opcode == SystemZ::MVC)
10710:       BuildMI(MBB, DL, TII->get(SystemZ::PFD))
10711:         .addImm(SystemZ::PFD_WRITE)
10712:         .addReg(ThisDestReg).addImm(DestDisp - IsMemset + 768).addReg(0);
10713:     insertMemMemOp(MBB, MBB->end(),
10714:                    MachineOperand::CreateReg(ThisDestReg, false), DestDisp,
10715:                    MachineOperand::CreateReg(ThisSrcReg, false), SrcDisp, 256);
10716:     if (EndMBB) {
10717:       BuildMI(MBB, DL, TII->get(SystemZ::BRC))
10718:         .addImm(SystemZ::CCMASK_ICMP).addImm(SystemZ::CCMASK_CMP_NE)
10719:         .addMBB(EndMBB);
10720:       MBB->addSuccessor(EndMBB);
10721:       MBB->addSuccessor(NextMBB);
10722:     }
10723: 
10724:     // NextMBB:
10725:     //   %NextDestReg = LA 256(%ThisDestReg)
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10726-10800 / 第 10726-10800 行
```cpp
10726:     //   %NextSrcReg = LA 256(%ThisSrcReg)
10727:     //   %NextCountReg = AGHI %ThisCountReg, -1
10728:     //   CGHI %NextCountReg, 0
10729:     //   JLH LoopMBB
10730:     //   # fall through to DoneMBB
10731:     //
10732:     // The AGHI, CGHI and JLH should be converted to BRCTG by later passes.
10733:     MBB = NextMBB;
10734:     BuildMI(MBB, DL, TII->get(SystemZ::LA), NextDestReg)
10735:       .addReg(ThisDestReg).addImm(256).addReg(0);
10736:     if (!HaveSingleBase)
10737:       BuildMI(MBB, DL, TII->get(SystemZ::LA), NextSrcReg)
10738:         .addReg(ThisSrcReg).addImm(256).addReg(0);
10739:     BuildMI(MBB, DL, TII->get(SystemZ::AGHI), NextCountReg)
10740:       .addReg(ThisCountReg).addImm(-1);
10741:     BuildMI(MBB, DL, TII->get(SystemZ::CGHI))
10742:       .addReg(NextCountReg).addImm(0);
10743:     BuildMI(MBB, DL, TII->get(SystemZ::BRC))
10744:       .addImm(SystemZ::CCMASK_ICMP).addImm(SystemZ::CCMASK_CMP_NE)
10745:       .addMBB(LoopMBB);
10746:     MBB->addSuccessor(LoopMBB);
10747:     MBB->addSuccessor(DoneMBB);
10748: 
10749:     MBB = DoneMBB;
10750:     if (IsRegForm) {
10751:       // DoneMBB:
10752:       // # Make PHIs for RemDestReg/RemSrcReg as the loop may or may not run.
10753:       // # Use EXecute Relative Long for the remainder of the bytes. The target
10754:       //   instruction of the EXRL will have a length field of 1 since 0 is an
10755:       //   illegal value. The number of bytes processed becomes (%LenAdjReg &
10756:       //   0xff) + 1.
10757:       // # Fall through to AllDoneMBB.
10758:       Register RemSrcReg  = MRI.createVirtualRegister(&SystemZ::ADDR64BitRegClass);
10759:       Register RemDestReg = HaveSingleBase ? RemSrcReg
10760:         : MRI.createVirtualRegister(&SystemZ::ADDR64BitRegClass);
10761:       BuildMI(MBB, DL, TII->get(SystemZ::PHI), RemDestReg)
10762:         .addReg(StartDestReg).addMBB(StartMBB)
10763:         .addReg(NextDestReg).addMBB(NextMBB);
10764:       if (!HaveSingleBase)
10765:         BuildMI(MBB, DL, TII->get(SystemZ::PHI), RemSrcReg)
10766:           .addReg(StartSrcReg).addMBB(StartMBB)
10767:           .addReg(NextSrcReg).addMBB(NextMBB);
10768:       if (IsMemset)
10769:         insertMemMemOp(MBB, MBB->end(),
10770:                        MachineOperand::CreateReg(RemDestReg, false), DestDisp,
10771:                        MachineOperand::CreateReg(RemSrcReg, false), SrcDisp, 1);
10772:       MachineInstrBuilder EXRL_MIB =
10773:         BuildMI(MBB, DL, TII->get(SystemZ::EXRL_Pseudo))
10774:           .addImm(Opcode)
10775:           .addReg(LenAdjReg)
10776:           .addReg(RemDestReg).addImm(DestDisp)
10777:           .addReg(RemSrcReg).addImm(SrcDisp);
10778:       MBB->addSuccessor(AllDoneMBB);
10779:       MBB = AllDoneMBB;
10780:       if (Opcode != SystemZ::MVC) {
10781:         EXRL_MIB.addReg(SystemZ::CC, RegState::ImplicitDefine);
10782:         if (EndMBB)
10783:           MBB->addLiveIn(SystemZ::CC);
10784:       }
10785:     }
10786:     MF.getProperties().resetNoPHIs();
10787:   }
10788: 
10789:   // Handle any remaining bytes with straight-line code.
10790:   while (ImmLength > 0) {
10791:     uint64_t ThisLength = std::min(ImmLength, uint64_t(256));
10792:     // The previous iteration might have created out-of-range displacements.
10793:     // Apply them using LA/LAY if so.
10794:     foldDisplIfNeeded(DestBase, DestDisp);
10795:     foldDisplIfNeeded(SrcBase, SrcDisp);
10796:     insertMemMemOp(MBB, MI, DestBase, DestDisp, SrcBase, SrcDisp, ThisLength);
10797:     DestDisp += ThisLength;
10798:     SrcDisp += ThisLength;
10799:     ImmLength -= ThisLength;
10800:     // If there's another CLC to go, branch to the end if a difference
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 10801-10875 / 第 10801-10875 行
```cpp
10801:     // was found.
10802:     if (EndMBB && ImmLength > 0) {
10803:       MachineBasicBlock *NextMBB = SystemZ::splitBlockBefore(MI, MBB);
10804:       BuildMI(MBB, DL, TII->get(SystemZ::BRC))
10805:         .addImm(SystemZ::CCMASK_ICMP).addImm(SystemZ::CCMASK_CMP_NE)
10806:         .addMBB(EndMBB);
10807:       MBB->addSuccessor(EndMBB);
10808:       MBB->addSuccessor(NextMBB);
10809:       MBB = NextMBB;
10810:     }
10811:   }
10812:   if (EndMBB) {
10813:     MBB->addSuccessor(EndMBB);
10814:     MBB = EndMBB;
10815:     MBB->addLiveIn(SystemZ::CC);
10816:   }
10817: 
10818:   MI.eraseFromParent();
10819:   return MBB;
10820: }
10821: 
10822: // Decompose string pseudo-instruction MI into a loop that continually performs
10823: // Opcode until CC != 3.
10824: MachineBasicBlock *SystemZTargetLowering::emitStringWrapper(
10825:     MachineInstr &MI, MachineBasicBlock *MBB, unsigned Opcode) const {
10826:   MachineFunction &MF = *MBB->getParent();
10827:   const SystemZInstrInfo *TII = Subtarget.getInstrInfo();
10828:   MachineRegisterInfo &MRI = MF.getRegInfo();
10829:   DebugLoc DL = MI.getDebugLoc();
10830: 
10831:   uint64_t End1Reg = MI.getOperand(0).getReg();
10832:   uint64_t Start1Reg = MI.getOperand(1).getReg();
10833:   uint64_t Start2Reg = MI.getOperand(2).getReg();
10834:   uint64_t CharReg = MI.getOperand(3).getReg();
10835: 
10836:   const TargetRegisterClass *RC = &SystemZ::GR64BitRegClass;
10837:   uint64_t This1Reg = MRI.createVirtualRegister(RC);
10838:   uint64_t This2Reg = MRI.createVirtualRegister(RC);
10839:   uint64_t End2Reg  = MRI.createVirtualRegister(RC);
10840: 
10841:   MachineBasicBlock *StartMBB = MBB;
10842:   MachineBasicBlock *DoneMBB = SystemZ::splitBlockBefore(MI, MBB);
10843:   MachineBasicBlock *LoopMBB = SystemZ::emitBlockAfter(StartMBB);
10844: 
10845:   //  StartMBB:
10846:   //   # fall through to LoopMBB
10847:   MBB->addSuccessor(LoopMBB);
10848: 
10849:   //  LoopMBB:
10850:   //   %This1Reg = phi [ %Start1Reg, StartMBB ], [ %End1Reg, LoopMBB ]
10851:   //   %This2Reg = phi [ %Start2Reg, StartMBB ], [ %End2Reg, LoopMBB ]
10852:   //   R0L = %CharReg
10853:   //   %End1Reg, %End2Reg = CLST %This1Reg, %This2Reg -- uses R0L
10854:   //   JO LoopMBB
10855:   //   # fall through to DoneMBB
10856:   //
10857:   // The load of R0L can be hoisted by post-RA LICM.
10858:   MBB = LoopMBB;
10859: 
10860:   BuildMI(MBB, DL, TII->get(SystemZ::PHI), This1Reg)
10861:     .addReg(Start1Reg).addMBB(StartMBB)
10862:     .addReg(End1Reg).addMBB(LoopMBB);
10863:   BuildMI(MBB, DL, TII->get(SystemZ::PHI), This2Reg)
10864:     .addReg(Start2Reg).addMBB(StartMBB)
10865:     .addReg(End2Reg).addMBB(LoopMBB);
10866:   BuildMI(MBB, DL, TII->get(TargetOpcode::COPY), SystemZ::R0L).addReg(CharReg);
10867:   BuildMI(MBB, DL, TII->get(Opcode))
10868:     .addReg(End1Reg, RegState::Define).addReg(End2Reg, RegState::Define)
10869:     .addReg(This1Reg).addReg(This2Reg);
10870:   BuildMI(MBB, DL, TII->get(SystemZ::BRC))
10871:     .addImm(SystemZ::CCMASK_ANY).addImm(SystemZ::CCMASK_3).addMBB(LoopMBB);
10872:   MBB->addSuccessor(LoopMBB);
10873:   MBB->addSuccessor(DoneMBB);
10874: 
10875:   DoneMBB->addLiveIn(SystemZ::CC);
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10876-10950 / 第 10876-10950 行
```cpp
10876: 
10877:   MI.eraseFromParent();
10878:   return DoneMBB;
10879: }
10880: 
10881: // Update TBEGIN instruction with final opcode and register clobbers.
10882: MachineBasicBlock *SystemZTargetLowering::emitTransactionBegin(
10883:     MachineInstr &MI, MachineBasicBlock *MBB, unsigned Opcode,
10884:     bool NoFloat) const {
10885:   MachineFunction &MF = *MBB->getParent();
10886:   const TargetFrameLowering *TFI = Subtarget.getFrameLowering();
10887:   const SystemZInstrInfo *TII = Subtarget.getInstrInfo();
10888: 
10889:   // Update opcode.
10890:   MI.setDesc(TII->get(Opcode));
10891: 
10892:   // We cannot handle a TBEGIN that clobbers the stack or frame pointer.
10893:   // Make sure to add the corresponding GRSM bits if they are missing.
10894:   uint64_t Control = MI.getOperand(2).getImm();
10895:   static const unsigned GPRControlBit[16] = {
10896:     0x8000, 0x8000, 0x4000, 0x4000, 0x2000, 0x2000, 0x1000, 0x1000,
10897:     0x0800, 0x0800, 0x0400, 0x0400, 0x0200, 0x0200, 0x0100, 0x0100
10898:   };
10899:   Control |= GPRControlBit[15];
10900:   if (TFI->hasFP(MF))
10901:     Control |= GPRControlBit[11];
10902:   MI.getOperand(2).setImm(Control);
10903: 
10904:   // Add GPR clobbers.
10905:   for (int I = 0; I < 16; I++) {
10906:     if ((Control & GPRControlBit[I]) == 0) {
10907:       unsigned Reg = SystemZMC::GR64Regs[I];
10908:       MI.addOperand(MachineOperand::CreateReg(Reg, true, true));
10909:     }
10910:   }
10911: 
10912:   // Add FPR/VR clobbers.
10913:   if (!NoFloat && (Control & 4) != 0) {
10914:     if (Subtarget.hasVector()) {
10915:       for (unsigned Reg : SystemZMC::VR128Regs) {
10916:         MI.addOperand(MachineOperand::CreateReg(Reg, true, true));
10917:       }
10918:     } else {
10919:       for (unsigned Reg : SystemZMC::FP64Regs) {
10920:         MI.addOperand(MachineOperand::CreateReg(Reg, true, true));
10921:       }
10922:     }
10923:   }
10924: 
10925:   return MBB;
10926: }
10927: 
10928: MachineBasicBlock *SystemZTargetLowering::emitLoadAndTestCmp0(
10929:     MachineInstr &MI, MachineBasicBlock *MBB, unsigned Opcode) const {
10930:   MachineFunction &MF = *MBB->getParent();
10931:   MachineRegisterInfo *MRI = &MF.getRegInfo();
10932:   const SystemZInstrInfo *TII = Subtarget.getInstrInfo();
10933:   DebugLoc DL = MI.getDebugLoc();
10934: 
10935:   Register SrcReg = MI.getOperand(0).getReg();
10936: 
10937:   // Create new virtual register of the same class as source.
10938:   const TargetRegisterClass *RC = MRI->getRegClass(SrcReg);
10939:   Register DstReg = MRI->createVirtualRegister(RC);
10940: 
10941:   // Replace pseudo with a normal load-and-test that models the def as
10942:   // well.
10943:   BuildMI(*MBB, MI, DL, TII->get(Opcode), DstReg)
10944:     .addReg(SrcReg)
10945:     .setMIFlags(MI.getFlags());
10946:   MI.eraseFromParent();
10947: 
10948:   return MBB;
10949: }
10950: 
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 10951-11025 / 第 10951-11025 行
```cpp
10951: MachineBasicBlock *SystemZTargetLowering::emitProbedAlloca(
10952:     MachineInstr &MI, MachineBasicBlock *MBB) const {
10953:   MachineFunction &MF = *MBB->getParent();
10954:   MachineRegisterInfo *MRI = &MF.getRegInfo();
10955:   const SystemZInstrInfo *TII = Subtarget.getInstrInfo();
10956:   DebugLoc DL = MI.getDebugLoc();
10957:   const unsigned ProbeSize = getStackProbeSize(MF);
10958:   Register DstReg = MI.getOperand(0).getReg();
10959:   Register SizeReg = MI.getOperand(2).getReg();
10960: 
10961:   MachineBasicBlock *StartMBB = MBB;
10962:   MachineBasicBlock *DoneMBB  = SystemZ::splitBlockAfter(MI, MBB);
10963:   MachineBasicBlock *LoopTestMBB  = SystemZ::emitBlockAfter(StartMBB);
10964:   MachineBasicBlock *LoopBodyMBB = SystemZ::emitBlockAfter(LoopTestMBB);
10965:   MachineBasicBlock *TailTestMBB = SystemZ::emitBlockAfter(LoopBodyMBB);
10966:   MachineBasicBlock *TailMBB = SystemZ::emitBlockAfter(TailTestMBB);
10967: 
10968:   MachineMemOperand *VolLdMMO = MF.getMachineMemOperand(MachinePointerInfo(),
10969:     MachineMemOperand::MOVolatile | MachineMemOperand::MOLoad, 8, Align(1));
10970: 
10971:   Register PHIReg = MRI->createVirtualRegister(&SystemZ::ADDR64BitRegClass);
10972:   Register IncReg = MRI->createVirtualRegister(&SystemZ::ADDR64BitRegClass);
10973: 
10974:   //  LoopTestMBB
10975:   //  BRC TailTestMBB
10976:   //  # fallthrough to LoopBodyMBB
10977:   StartMBB->addSuccessor(LoopTestMBB);
10978:   MBB = LoopTestMBB;
10979:   BuildMI(MBB, DL, TII->get(SystemZ::PHI), PHIReg)
10980:     .addReg(SizeReg)
10981:     .addMBB(StartMBB)
10982:     .addReg(IncReg)
10983:     .addMBB(LoopBodyMBB);
10984:   BuildMI(MBB, DL, TII->get(SystemZ::CLGFI))
10985:     .addReg(PHIReg)
10986:     .addImm(ProbeSize);
10987:   BuildMI(MBB, DL, TII->get(SystemZ::BRC))
10988:     .addImm(SystemZ::CCMASK_ICMP).addImm(SystemZ::CCMASK_CMP_LT)
10989:     .addMBB(TailTestMBB);
10990:   MBB->addSuccessor(LoopBodyMBB);
10991:   MBB->addSuccessor(TailTestMBB);
10992: 
10993:   //  LoopBodyMBB: Allocate and probe by means of a volatile compare.
10994:   //  J LoopTestMBB
10995:   MBB = LoopBodyMBB;
10996:   BuildMI(MBB, DL, TII->get(SystemZ::SLGFI), IncReg)
10997:     .addReg(PHIReg)
10998:     .addImm(ProbeSize);
10999:   BuildMI(MBB, DL, TII->get(SystemZ::SLGFI), SystemZ::R15D)
11000:     .addReg(SystemZ::R15D)
11001:     .addImm(ProbeSize);
11002:   BuildMI(MBB, DL, TII->get(SystemZ::CG)).addReg(SystemZ::R15D)
11003:     .addReg(SystemZ::R15D).addImm(ProbeSize - 8).addReg(0)
11004:     .setMemRefs(VolLdMMO);
11005:   BuildMI(MBB, DL, TII->get(SystemZ::J)).addMBB(LoopTestMBB);
11006:   MBB->addSuccessor(LoopTestMBB);
11007: 
11008:   //  TailTestMBB
11009:   //  BRC DoneMBB
11010:   //  # fallthrough to TailMBB
11011:   MBB = TailTestMBB;
11012:   BuildMI(MBB, DL, TII->get(SystemZ::CGHI))
11013:     .addReg(PHIReg)
11014:     .addImm(0);
11015:   BuildMI(MBB, DL, TII->get(SystemZ::BRC))
11016:     .addImm(SystemZ::CCMASK_ICMP).addImm(SystemZ::CCMASK_CMP_EQ)
11017:     .addMBB(DoneMBB);
11018:   MBB->addSuccessor(TailMBB);
11019:   MBB->addSuccessor(DoneMBB);
11020: 
11021:   //  TailMBB
11022:   //  # fallthrough to DoneMBB
11023:   MBB = TailMBB;
11024:   BuildMI(MBB, DL, TII->get(SystemZ::SLGR), SystemZ::R15D)
11025:     .addReg(SystemZ::R15D)
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。

### Lines 11026-11100 / 第 11026-11100 行
```cpp
11026:     .addReg(PHIReg);
11027:   BuildMI(MBB, DL, TII->get(SystemZ::CG)).addReg(SystemZ::R15D)
11028:     .addReg(SystemZ::R15D).addImm(-8).addReg(PHIReg)
11029:     .setMemRefs(VolLdMMO);
11030:   MBB->addSuccessor(DoneMBB);
11031: 
11032:   //  DoneMBB
11033:   MBB = DoneMBB;
11034:   BuildMI(*MBB, MBB->begin(), DL, TII->get(TargetOpcode::COPY), DstReg)
11035:     .addReg(SystemZ::R15D);
11036: 
11037:   MI.eraseFromParent();
11038:   return DoneMBB;
11039: }
11040: 
11041: SDValue SystemZTargetLowering::
11042: getBackchainAddress(SDValue SP, SelectionDAG &DAG) const {
11043:   MachineFunction &MF = DAG.getMachineFunction();
11044:   auto *TFL = Subtarget.getFrameLowering<SystemZELFFrameLowering>();
11045:   SDLoc DL(SP);
11046:   return DAG.getNode(ISD::ADD, DL, MVT::i64, SP,
11047:                      DAG.getIntPtrConstant(TFL->getBackchainOffset(MF), DL));
11048: }
11049: 
11050: MachineBasicBlock *SystemZTargetLowering::EmitInstrWithCustomInserter(
11051:     MachineInstr &MI, MachineBasicBlock *MBB) const {
11052:   switch (MI.getOpcode()) {
11053:   case SystemZ::ADJCALLSTACKDOWN:
11054:   case SystemZ::ADJCALLSTACKUP:
11055:     return emitAdjCallStack(MI, MBB);
11056: 
11057:   case SystemZ::Select32:
11058:   case SystemZ::Select64:
11059:   case SystemZ::Select128:
11060:   case SystemZ::SelectF32:
11061:   case SystemZ::SelectF64:
11062:   case SystemZ::SelectF128:
11063:   case SystemZ::SelectVR32:
11064:   case SystemZ::SelectVR64:
11065:   case SystemZ::SelectVR128:
11066:     return emitSelect(MI, MBB);
11067: 
11068:   case SystemZ::CondStore8Mux:
11069:     return emitCondStore(MI, MBB, SystemZ::STCMux, 0, false);
11070:   case SystemZ::CondStore8MuxInv:
11071:     return emitCondStore(MI, MBB, SystemZ::STCMux, 0, true);
11072:   case SystemZ::CondStore16Mux:
11073:     return emitCondStore(MI, MBB, SystemZ::STHMux, 0, false);
11074:   case SystemZ::CondStore16MuxInv:
11075:     return emitCondStore(MI, MBB, SystemZ::STHMux, 0, true);
11076:   case SystemZ::CondStore32Mux:
11077:     return emitCondStore(MI, MBB, SystemZ::STMux, SystemZ::STOCMux, false);
11078:   case SystemZ::CondStore32MuxInv:
11079:     return emitCondStore(MI, MBB, SystemZ::STMux, SystemZ::STOCMux, true);
11080:   case SystemZ::CondStore8:
11081:     return emitCondStore(MI, MBB, SystemZ::STC, 0, false);
11082:   case SystemZ::CondStore8Inv:
11083:     return emitCondStore(MI, MBB, SystemZ::STC, 0, true);
11084:   case SystemZ::CondStore16:
11085:     return emitCondStore(MI, MBB, SystemZ::STH, 0, false);
11086:   case SystemZ::CondStore16Inv:
11087:     return emitCondStore(MI, MBB, SystemZ::STH, 0, true);
11088:   case SystemZ::CondStore32:
11089:     return emitCondStore(MI, MBB, SystemZ::ST, SystemZ::STOC, false);
11090:   case SystemZ::CondStore32Inv:
11091:     return emitCondStore(MI, MBB, SystemZ::ST, SystemZ::STOC, true);
11092:   case SystemZ::CondStore64:
11093:     return emitCondStore(MI, MBB, SystemZ::STG, SystemZ::STOCG, false);
11094:   case SystemZ::CondStore64Inv:
11095:     return emitCondStore(MI, MBB, SystemZ::STG, SystemZ::STOCG, true);
11096:   case SystemZ::CondStoreF32:
11097:     return emitCondStore(MI, MBB, SystemZ::STE, 0, false);
11098:   case SystemZ::CondStoreF32Inv:
11099:     return emitCondStore(MI, MBB, SystemZ::STE, 0, true);
11100:   case SystemZ::CondStoreF64:
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `getBackchainAddress`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `getBackchainAddress` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 11101-11175 / 第 11101-11175 行
```cpp
11101:     return emitCondStore(MI, MBB, SystemZ::STD, 0, false);
11102:   case SystemZ::CondStoreF64Inv:
11103:     return emitCondStore(MI, MBB, SystemZ::STD, 0, true);
11104: 
11105:   case SystemZ::SCmp128Hi:
11106:     return emitICmp128Hi(MI, MBB, false);
11107:   case SystemZ::UCmp128Hi:
11108:     return emitICmp128Hi(MI, MBB, true);
11109: 
11110:   case SystemZ::PAIR128:
11111:     return emitPair128(MI, MBB);
11112:   case SystemZ::AEXT128:
11113:     return emitExt128(MI, MBB, false);
11114:   case SystemZ::ZEXT128:
11115:     return emitExt128(MI, MBB, true);
11116: 
11117:   case SystemZ::ATOMIC_SWAPW:
11118:     return emitAtomicLoadBinary(MI, MBB, 0);
11119: 
11120:   case SystemZ::ATOMIC_LOADW_AR:
11121:     return emitAtomicLoadBinary(MI, MBB, SystemZ::AR);
11122:   case SystemZ::ATOMIC_LOADW_AFI:
11123:     return emitAtomicLoadBinary(MI, MBB, SystemZ::AFI);
11124: 
11125:   case SystemZ::ATOMIC_LOADW_SR:
11126:     return emitAtomicLoadBinary(MI, MBB, SystemZ::SR);
11127: 
11128:   case SystemZ::ATOMIC_LOADW_NR:
11129:     return emitAtomicLoadBinary(MI, MBB, SystemZ::NR);
11130:   case SystemZ::ATOMIC_LOADW_NILH:
11131:     return emitAtomicLoadBinary(MI, MBB, SystemZ::NILH);
11132: 
11133:   case SystemZ::ATOMIC_LOADW_OR:
11134:     return emitAtomicLoadBinary(MI, MBB, SystemZ::OR);
11135:   case SystemZ::ATOMIC_LOADW_OILH:
11136:     return emitAtomicLoadBinary(MI, MBB, SystemZ::OILH);
11137: 
11138:   case SystemZ::ATOMIC_LOADW_XR:
11139:     return emitAtomicLoadBinary(MI, MBB, SystemZ::XR);
11140:   case SystemZ::ATOMIC_LOADW_XILF:
11141:     return emitAtomicLoadBinary(MI, MBB, SystemZ::XILF);
11142: 
11143:   case SystemZ::ATOMIC_LOADW_NRi:
11144:     return emitAtomicLoadBinary(MI, MBB, SystemZ::NR, true);
11145:   case SystemZ::ATOMIC_LOADW_NILHi:
11146:     return emitAtomicLoadBinary(MI, MBB, SystemZ::NILH, true);
11147: 
11148:   case SystemZ::ATOMIC_LOADW_MIN:
11149:     return emitAtomicLoadMinMax(MI, MBB, SystemZ::CR, SystemZ::CCMASK_CMP_LE);
11150:   case SystemZ::ATOMIC_LOADW_MAX:
11151:     return emitAtomicLoadMinMax(MI, MBB, SystemZ::CR, SystemZ::CCMASK_CMP_GE);
11152:   case SystemZ::ATOMIC_LOADW_UMIN:
11153:     return emitAtomicLoadMinMax(MI, MBB, SystemZ::CLR, SystemZ::CCMASK_CMP_LE);
11154:   case SystemZ::ATOMIC_LOADW_UMAX:
11155:     return emitAtomicLoadMinMax(MI, MBB, SystemZ::CLR, SystemZ::CCMASK_CMP_GE);
11156: 
11157:   case SystemZ::ATOMIC_CMP_SWAPW:
11158:     return emitAtomicCmpSwapW(MI, MBB);
11159:   case SystemZ::MVCImm:
11160:   case SystemZ::MVCReg:
11161:     return emitMemMemWrapper(MI, MBB, SystemZ::MVC);
11162:   case SystemZ::NCImm:
11163:     return emitMemMemWrapper(MI, MBB, SystemZ::NC);
11164:   case SystemZ::OCImm:
11165:     return emitMemMemWrapper(MI, MBB, SystemZ::OC);
11166:   case SystemZ::XCImm:
11167:   case SystemZ::XCReg:
11168:     return emitMemMemWrapper(MI, MBB, SystemZ::XC);
11169:   case SystemZ::CLCImm:
11170:   case SystemZ::CLCReg:
11171:     return emitMemMemWrapper(MI, MBB, SystemZ::CLC);
11172:   case SystemZ::MemsetImmImm:
11173:   case SystemZ::MemsetImmReg:
11174:   case SystemZ::MemsetRegImm:
11175:   case SystemZ::MemsetRegReg:
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the SystemZ backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 11176-11250 / 第 11176-11250 行
```cpp
11176:     return emitMemMemWrapper(MI, MBB, SystemZ::MVC, true/*IsMemset*/);
11177:   case SystemZ::CLSTLoop:
11178:     return emitStringWrapper(MI, MBB, SystemZ::CLST);
11179:   case SystemZ::MVSTLoop:
11180:     return emitStringWrapper(MI, MBB, SystemZ::MVST);
11181:   case SystemZ::SRSTLoop:
11182:     return emitStringWrapper(MI, MBB, SystemZ::SRST);
11183:   case SystemZ::TBEGIN:
11184:     return emitTransactionBegin(MI, MBB, SystemZ::TBEGIN, false);
11185:   case SystemZ::TBEGIN_nofloat:
11186:     return emitTransactionBegin(MI, MBB, SystemZ::TBEGIN, true);
11187:   case SystemZ::TBEGINC:
11188:     return emitTransactionBegin(MI, MBB, SystemZ::TBEGINC, true);
11189:   case SystemZ::LTEBRCompare_Pseudo:
11190:     return emitLoadAndTestCmp0(MI, MBB, SystemZ::LTEBR);
11191:   case SystemZ::LTDBRCompare_Pseudo:
11192:     return emitLoadAndTestCmp0(MI, MBB, SystemZ::LTDBR);
11193:   case SystemZ::LTXBRCompare_Pseudo:
11194:     return emitLoadAndTestCmp0(MI, MBB, SystemZ::LTXBR);
11195: 
11196:   case SystemZ::PROBED_ALLOCA:
11197:     return emitProbedAlloca(MI, MBB);
11198:   case SystemZ::EH_SjLj_SetJmp:
11199:     return emitEHSjLjSetJmp(MI, MBB);
11200:   case SystemZ::EH_SjLj_LongJmp:
11201:     return emitEHSjLjLongJmp(MI, MBB);
11202: 
11203:   case TargetOpcode::STACKMAP:
11204:   case TargetOpcode::PATCHPOINT:
11205:     return emitPatchPoint(MI, MBB);
11206: 
11207:   default:
11208:     llvm_unreachable("Unexpected instr type to insert");
11209:   }
11210: }
11211: 
11212: // This is only used by the isel schedulers, and is needed only to prevent
11213: // compiler from crashing when list-ilp is used.
11214: const TargetRegisterClass *
11215: SystemZTargetLowering::getRepRegClassFor(MVT VT) const {
11216:   if (VT == MVT::Untyped)
11217:     return &SystemZ::ADDR128BitRegClass;
11218:   return TargetLowering::getRepRegClassFor(VT);
11219: }
11220: 
11221: SDValue SystemZTargetLowering::lowerGET_ROUNDING(SDValue Op,
11222:                                                  SelectionDAG &DAG) const {
11223:   SDLoc dl(Op);
11224:   /*
11225:    The rounding method is in FPC Byte 3 bits 6-7, and has the following
11226:    settings:
11227:      00 Round to nearest
11228:      01 Round to 0
11229:      10 Round to +inf
11230:      11 Round to -inf
11231: 
11232:   FLT_ROUNDS, on the other hand, expects the following:
11233:     -1 Undefined
11234:      0 Round to 0
11235:      1 Round to nearest
11236:      2 Round to +inf
11237:      3 Round to -inf
11238:   */
11239: 
11240:   // Save FPC to register.
11241:   SDValue Chain = Op.getOperand(0);
11242:   SDValue EFPC(
11243:       DAG.getMachineNode(SystemZ::EFPC, dl, {MVT::i32, MVT::Other}, Chain), 0);
11244:   Chain = EFPC.getValue(1);
11245: 
11246:   // Transform as necessary
11247:   SDValue CWD1 = DAG.getNode(ISD::AND, dl, MVT::i32, EFPC,
11248:                              DAG.getConstant(3, dl, MVT::i32));
11249:   // RetVal = (CWD1 ^ (CWD1 >> 1)) ^ 1
11250:   SDValue CWD2 = DAG.getNode(ISD::XOR, dl, MVT::i32, CWD1,
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::getRepRegClassFor`, `SystemZTargetLowering::lowerGET_ROUNDING`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::getRepRegClassFor`, `SystemZTargetLowering::lowerGET_ROUNDING` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 11251-11325 / 第 11251-11325 行
```cpp
11251:                              DAG.getNode(ISD::SRL, dl, MVT::i32, CWD1,
11252:                                          DAG.getConstant(1, dl, MVT::i32)));
11253: 
11254:   SDValue RetVal = DAG.getNode(ISD::XOR, dl, MVT::i32, CWD2,
11255:                                DAG.getConstant(1, dl, MVT::i32));
11256:   RetVal = DAG.getZExtOrTrunc(RetVal, dl, Op.getValueType());
11257: 
11258:   return DAG.getMergeValues({RetVal, Chain}, dl);
11259: }
11260: 
11261: SDValue SystemZTargetLowering::lowerVECREDUCE_ADD(SDValue Op,
11262:                                                   SelectionDAG &DAG) const {
11263:   EVT VT = Op.getValueType();
11264:   Op = Op.getOperand(0);
11265:   EVT OpVT = Op.getValueType();
11266: 
11267:   assert(OpVT.isVector() && "Operand type for VECREDUCE_ADD is not a vector.");
11268: 
11269:   SDLoc DL(Op);
11270: 
11271:   // load a 0 vector for the third operand of VSUM.
11272:   SDValue Zero = DAG.getSplatBuildVector(OpVT, DL, DAG.getConstant(0, DL, VT));
11273: 
11274:   // execute VSUM.
11275:   switch (OpVT.getScalarSizeInBits()) {
11276:   case 8:
11277:   case 16:
11278:     Op = DAG.getNode(SystemZISD::VSUM, DL, MVT::v4i32, Op, Zero);
11279:     [[fallthrough]];
11280:   case 32:
11281:   case 64:
11282:     Op = DAG.getNode(SystemZISD::VSUM, DL, MVT::i128, Op,
11283:                      DAG.getBitcast(Op.getValueType(), Zero));
11284:     break;
11285:   case 128:
11286:     break; // VSUM over v1i128 should not happen and would be a noop
11287:   default:
11288:     llvm_unreachable("Unexpected scalar size.");
11289:   }
11290:   // Cast to original vector type, retrieve last element.
11291:   return DAG.getNode(
11292:       ISD::EXTRACT_VECTOR_ELT, DL, VT, DAG.getBitcast(OpVT, Op),
11293:       DAG.getConstant(OpVT.getVectorNumElements() - 1, DL, MVT::i32));
11294: }
11295: 
11296: static void printFunctionArgExts(const Function *F, raw_fd_ostream &OS) {
11297:   FunctionType *FT = F->getFunctionType();
11298:   const AttributeList &Attrs = F->getAttributes();
11299:   if (Attrs.hasRetAttrs())
11300:     OS << Attrs.getAsString(AttributeList::ReturnIndex) << " ";
11301:   OS << *F->getReturnType() << " @" << F->getName() << "(";
11302:   for (unsigned I = 0, E = FT->getNumParams(); I != E; ++I) {
11303:     if (I)
11304:       OS << ", ";
11305:     OS << *FT->getParamType(I);
11306:     AttributeSet ArgAttrs = Attrs.getParamAttrs(I);
11307:     for (auto A : {Attribute::SExt, Attribute::ZExt, Attribute::NoExt})
11308:       if (ArgAttrs.hasAttribute(A))
11309:         OS << " " << Attribute::getNameFromAttrKind(A);
11310:   }
11311:   OS << ")\n";
11312: }
11313: 
11314: bool SystemZTargetLowering::isInternal(const Function *Fn) const {
11315:   std::map<const Function *, bool>::iterator Itr = IsInternalCache.find(Fn);
11316:   if (Itr == IsInternalCache.end())
11317:     Itr = IsInternalCache
11318:               .insert(std::pair<const Function *, bool>(
11319:                   Fn, (Fn->hasLocalLinkage() && !Fn->hasAddressTaken())))
11320:               .first;
11321:   return Itr->second;
11322: }
11323: 
11324: void SystemZTargetLowering::
11325: verifyNarrowIntegerArgs_Call(const SmallVectorImpl<ISD::OutputArg> &Outs,
```
- **EN**: The range implements or declares functions including `SystemZTargetLowering::lowerVECREDUCE_ADD`, `printFunctionArgExts`, `SystemZTargetLowering::isInternal`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `SystemZTargetLowering::lowerVECREDUCE_ADD`, `printFunctionArgExts`, `SystemZTargetLowering::isInternal` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 11326-11392 / 第 11326-11392 行
```cpp
11326:                              const Function *F, SDValue Callee) const {
11327:   // Temporarily only do the check when explicitly requested, until it can be
11328:   // enabled by default.
11329:   if (!EnableIntArgExtCheck)
11330:     return;
11331: 
11332:   bool IsInternal = false;
11333:   const Function *CalleeFn = nullptr;
11334:   if (auto *G = dyn_cast<GlobalAddressSDNode>(Callee))
11335:     if ((CalleeFn = dyn_cast<Function>(G->getGlobal())))
11336:       IsInternal = isInternal(CalleeFn);
11337:   if (!IsInternal && !verifyNarrowIntegerArgs(Outs)) {
11338:     errs() << "ERROR: Missing extension attribute of passed "
11339:            << "value in call to function:\n" << "Callee:  ";
11340:     if (CalleeFn != nullptr)
11341:       printFunctionArgExts(CalleeFn, errs());
11342:     else
11343:       errs() << "-\n";
11344:     errs() << "Caller:  ";
11345:     printFunctionArgExts(F, errs());
11346:     llvm_unreachable("");
11347:   }
11348: }
11349: 
11350: void SystemZTargetLowering::
11351: verifyNarrowIntegerArgs_Ret(const SmallVectorImpl<ISD::OutputArg> &Outs,
11352:                             const Function *F) const {
11353:   // Temporarily only do the check when explicitly requested, until it can be
11354:   // enabled by default.
11355:   if (!EnableIntArgExtCheck)
11356:     return;
11357: 
11358:   if (!isInternal(F) && !verifyNarrowIntegerArgs(Outs)) {
11359:     errs() << "ERROR: Missing extension attribute of returned "
11360:            << "value from function:\n";
11361:     printFunctionArgExts(F, errs());
11362:     llvm_unreachable("");
11363:   }
11364: }
11365: 
11366: // Verify that narrow integer arguments are extended as required by the ABI.
11367: // Return false if an error is found.
11368: bool SystemZTargetLowering::verifyNarrowIntegerArgs(
11369:     const SmallVectorImpl<ISD::OutputArg> &Outs) const {
11370:   if (!Subtarget.isTargetELF())
11371:     return true;
11372: 
11373:   if (EnableIntArgExtCheck.getNumOccurrences()) {
11374:     if (!EnableIntArgExtCheck)
11375:       return true;
11376:   } else if (!getTargetMachine().Options.VerifyArgABICompliance)
11377:     return true;
11378: 
11379:   for (unsigned i = 0; i < Outs.size(); ++i) {
11380:     MVT VT = Outs[i].VT;
11381:     ISD::ArgFlagsTy Flags = Outs[i].Flags;
11382:     if (VT.isInteger()) {
11383:       assert((VT == MVT::i32 || VT.getSizeInBits() >= 64) &&
11384:              "Unexpected integer argument VT.");
11385:       if (VT == MVT::i32 &&
11386:           !Flags.isSExt() && !Flags.isZExt() && !Flags.isNoExt())
11387:         return false;
11388:     }
11389:   }
11390: 
11391:   return true;
11392: }
```
- **EN**: The range implements or declares functions including `verifyNarrowIntegerArgs_Ret`, `SystemZTargetLowering::verifyNarrowIntegerArgs`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `verifyNarrowIntegerArgs_Ret`, `SystemZTargetLowering::verifyNarrowIntegerArgs` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- `SystemZISelLowering.h`
- `SystemZCallingConv.h`
- `SystemZConstantPoolValue.h`
- `SystemZMachineFunctionInfo.h`
- `SystemZTargetMachine.h`
- `llvm/ADT/SmallSet.h`
- `llvm/CodeGen/CallingConvLower.h`
- `llvm/CodeGen/ISDOpcodes.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/TargetLoweringObjectFileImpl.h`
- `llvm/IR/GlobalAlias.h`
- `llvm/IR/IntrinsicInst.h`
- `llvm/IR/Intrinsics.h`
- `llvm/IR/IntrinsicsS390.h`
- `llvm/IR/PatternMatch.h`
- `...` (6 more include dependencies omitted for brevity / 其余 6 个 include 依赖已省略)
