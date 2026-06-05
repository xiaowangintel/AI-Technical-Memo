# LoongArchISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/LoongArch/LoongArchISelLowering.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **用途 (CN)**: 该文件用于 LoongArch 后端，负责实现目标后端的 SelectionDAG 降低。

## Line-by-Line Analysis / 逐行分析
### Lines 1-75 / 第 1-75 行
```cpp
   1: //=- LoongArchISelLowering.cpp - LoongArch DAG Lowering Implementation  ---===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the interfaces that LoongArch uses to lower LLVM code into
  10: // a selection DAG.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "LoongArchISelLowering.h"
  15: #include "LoongArch.h"
  16: #include "LoongArchMachineFunctionInfo.h"
  17: #include "LoongArchRegisterInfo.h"
  18: #include "LoongArchSelectionDAGInfo.h"
  19: #include "LoongArchSubtarget.h"
  20: #include "MCTargetDesc/LoongArchBaseInfo.h"
  21: #include "MCTargetDesc/LoongArchMCTargetDesc.h"
  22: #include "MCTargetDesc/LoongArchMatInt.h"
  23: #include "llvm/ADT/SmallSet.h"
  24: #include "llvm/ADT/Statistic.h"
  25: #include "llvm/ADT/StringExtras.h"
  26: #include "llvm/CodeGen/ISDOpcodes.h"
  27: #include "llvm/CodeGen/MachineInstrBuilder.h"
  28: #include "llvm/CodeGen/RuntimeLibcallUtil.h"
  29: #include "llvm/CodeGen/SelectionDAGNodes.h"
  30: #include "llvm/IR/IRBuilder.h"
  31: #include "llvm/IR/IntrinsicInst.h"
  32: #include "llvm/IR/IntrinsicsLoongArch.h"
  33: #include "llvm/Support/CodeGen.h"
  34: #include "llvm/Support/Debug.h"
  35: #include "llvm/Support/ErrorHandling.h"
  36: #include "llvm/Support/KnownBits.h"
  37: #include "llvm/Support/MathExtras.h"
  38: #include <llvm/Analysis/VectorUtils.h>
  39: 
  40: using namespace llvm;
  41: 
  42: #define DEBUG_TYPE "loongarch-isel-lowering"
  43: 
  44: STATISTIC(NumTailCalls, "Number of tail calls");
  45: 
  46: enum MaterializeFPImm {
  47:   NoMaterializeFPImm = 0,
  48:   MaterializeFPImm2Ins = 2,
  49:   MaterializeFPImm3Ins = 3,
  50:   MaterializeFPImm4Ins = 4,
  51:   MaterializeFPImm5Ins = 5,
  52:   MaterializeFPImm6Ins = 6
  53: };
  54: 
  55: static cl::opt<MaterializeFPImm> MaterializeFPImmInsNum(
  56:     "loongarch-materialize-float-imm", cl::Hidden,
  57:     cl::desc("Maximum number of instructions used (including code sequence "
  58:              "to generate the value and moving the value to FPR) when "
  59:              "materializing floating-point immediates (default = 3)"),
  60:     cl::init(MaterializeFPImm3Ins),
  61:     cl::values(clEnumValN(NoMaterializeFPImm, "0", "Use constant pool"),
  62:                clEnumValN(MaterializeFPImm2Ins, "2",
  63:                           "Materialize FP immediate within 2 instructions"),
  64:                clEnumValN(MaterializeFPImm3Ins, "3",
  65:                           "Materialize FP immediate within 3 instructions"),
  66:                clEnumValN(MaterializeFPImm4Ins, "4",
  67:                           "Materialize FP immediate within 4 instructions"),
  68:                clEnumValN(MaterializeFPImm5Ins, "5",
  69:                           "Materialize FP immediate within 5 instructions"),
  70:                clEnumValN(MaterializeFPImm6Ins, "6",
  71:                           "Materialize FP immediate within 6 instructions "
  72:                           "(behaves same as 5 on loongarch64)")));
  73: 
  74: static cl::opt<bool> ZeroDivCheck("loongarch-check-zero-division", cl::Hidden,
  75:                                   cl::desc("Trap on integer division by zero."),
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `LoongArchISelLowering.h`, `LoongArch.h`, `LoongArchMachineFunctionInfo.h`, `LoongArchRegisterInfo.h`, `LoongArchSelectionDAGInfo.h`, `LoongArchSubtarget.h` that expose the LLVM and target interfaces used in later logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `LoongArchISelLowering.h`, `LoongArch.h`, `LoongArchMachineFunctionInfo.h`, `LoongArchRegisterInfo.h`, `LoongArchSelectionDAGInfo.h`, `LoongArchSubtarget.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 76-150 / 第 76-150 行
```cpp
  76:                                   cl::init(false));
  77: 
  78: LoongArchTargetLowering::LoongArchTargetLowering(const TargetMachine &TM,
  79:                                                  const LoongArchSubtarget &STI)
  80:     : TargetLowering(TM, STI), Subtarget(STI) {
  81: 
  82:   MVT GRLenVT = Subtarget.getGRLenVT();
  83: 
  84:   // Set up the register classes.
  85: 
  86:   addRegisterClass(GRLenVT, &LoongArch::GPRRegClass);
  87:   if (Subtarget.hasBasicF())
  88:     addRegisterClass(MVT::f32, &LoongArch::FPR32RegClass);
  89:   if (Subtarget.hasBasicD())
  90:     addRegisterClass(MVT::f64, &LoongArch::FPR64RegClass);
  91: 
  92:   static const MVT::SimpleValueType LSXVTs[] = {
  93:       MVT::v16i8, MVT::v8i16, MVT::v4i32, MVT::v2i64, MVT::v4f32, MVT::v2f64};
  94:   static const MVT::SimpleValueType LASXVTs[] = {
  95:       MVT::v32i8, MVT::v16i16, MVT::v8i32, MVT::v4i64, MVT::v8f32, MVT::v4f64};
  96: 
  97:   if (Subtarget.hasExtLSX())
  98:     for (MVT VT : LSXVTs)
  99:       addRegisterClass(VT, &LoongArch::LSX128RegClass);
 100: 
 101:   if (Subtarget.hasExtLASX())
 102:     for (MVT VT : LASXVTs)
 103:       addRegisterClass(VT, &LoongArch::LASX256RegClass);
 104: 
 105:   // Set operations for LA32 and LA64.
 106: 
 107:   setLoadExtAction({ISD::EXTLOAD, ISD::SEXTLOAD, ISD::ZEXTLOAD}, GRLenVT,
 108:                    MVT::i1, Promote);
 109: 
 110:   setOperationAction(ISD::SHL_PARTS, GRLenVT, Custom);
 111:   setOperationAction(ISD::SRA_PARTS, GRLenVT, Custom);
 112:   setOperationAction(ISD::SRL_PARTS, GRLenVT, Custom);
 113:   setOperationAction(ISD::FP_TO_SINT, GRLenVT, Custom);
 114:   setOperationAction(ISD::ROTL, GRLenVT, Expand);
 115:   setOperationAction(ISD::CTPOP, GRLenVT, Expand);
 116: 
 117:   setOperationAction({ISD::GlobalAddress, ISD::BlockAddress, ISD::ConstantPool,
 118:                       ISD::JumpTable, ISD::GlobalTLSAddress},
 119:                      GRLenVT, Custom);
 120: 
 121:   setOperationAction(ISD::EH_DWARF_CFA, GRLenVT, Custom);
 122: 
 123:   setOperationAction(ISD::DYNAMIC_STACKALLOC, GRLenVT, Expand);
 124:   setOperationAction({ISD::STACKSAVE, ISD::STACKRESTORE}, MVT::Other, Expand);
 125:   setOperationAction(ISD::VASTART, MVT::Other, Custom);
 126:   setOperationAction({ISD::VAARG, ISD::VACOPY, ISD::VAEND}, MVT::Other, Expand);
 127: 
 128:   setOperationAction(ISD::DEBUGTRAP, MVT::Other, Legal);
 129:   setOperationAction(ISD::TRAP, MVT::Other, Legal);
 130: 
 131:   setOperationAction(ISD::INTRINSIC_VOID, MVT::Other, Custom);
 132:   setOperationAction(ISD::INTRINSIC_W_CHAIN, MVT::Other, Custom);
 133:   setOperationAction(ISD::INTRINSIC_WO_CHAIN, MVT::Other, Custom);
 134: 
 135:   setOperationAction(ISD::PREFETCH, MVT::Other, Custom);
 136: 
 137:   // BITREV/REVB requires the 32S feature.
 138:   if (STI.has32S()) {
 139:     // Expand bitreverse.i16 with native-width bitrev and shift for now, before
 140:     // we get to know which of sll and revb.2h is faster.
 141:     setOperationAction(ISD::BITREVERSE, MVT::i8, Custom);
 142:     setOperationAction(ISD::BITREVERSE, GRLenVT, Legal);
 143: 
 144:     // LA32 does not have REVB.2W and REVB.D due to the 64-bit operands, and
 145:     // the narrower REVB.W does not exist. But LA32 does have REVB.2H, so i16
 146:     // and i32 could still be byte-swapped relatively cheaply.
 147:     setOperationAction(ISD::BSWAP, MVT::i16, Custom);
 148:   } else {
 149:     setOperationAction(ISD::BSWAP, GRLenVT, Expand);
 150:     setOperationAction(ISD::CTTZ, GRLenVT, Expand);
```
- **EN**: The range implements or declares functions including `TargetLowering`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `TargetLowering` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 151-225 / 第 151-225 行
```cpp
 151:     setOperationAction(ISD::CTLZ, GRLenVT, Expand);
 152:     setOperationAction(ISD::ROTR, GRLenVT, Expand);
 153:     setOperationAction(ISD::SELECT, GRLenVT, Custom);
 154:     setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i8, Expand);
 155:     setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i16, Expand);
 156:   }
 157: 
 158:   setOperationAction(ISD::BR_JT, MVT::Other, Expand);
 159:   setOperationAction(ISD::BR_CC, GRLenVT, Expand);
 160:   setOperationAction(ISD::BRCOND, MVT::Other, Custom);
 161:   setOperationAction(ISD::SELECT_CC, GRLenVT, Expand);
 162:   setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i1, Expand);
 163:   setOperationAction({ISD::SMUL_LOHI, ISD::UMUL_LOHI}, GRLenVT, Expand);
 164: 
 165:   setOperationAction(ISD::FP_TO_UINT, GRLenVT, Custom);
 166:   setOperationAction(ISD::UINT_TO_FP, GRLenVT, Expand);
 167: 
 168:   // Set operations for LA64 only.
 169: 
 170:   if (Subtarget.is64Bit()) {
 171:     setOperationAction(ISD::ADD, MVT::i32, Custom);
 172:     setOperationAction(ISD::SUB, MVT::i32, Custom);
 173:     setOperationAction(ISD::SHL, MVT::i32, Custom);
 174:     setOperationAction(ISD::SRA, MVT::i32, Custom);
 175:     setOperationAction(ISD::SRL, MVT::i32, Custom);
 176:     setOperationAction(ISD::FP_TO_SINT, MVT::i32, Custom);
 177:     setOperationAction(ISD::BITCAST, MVT::i32, Custom);
 178:     setOperationAction(ISD::ROTR, MVT::i32, Custom);
 179:     setOperationAction(ISD::ROTL, MVT::i32, Custom);
 180:     setOperationAction(ISD::CTTZ, MVT::i32, Custom);
 181:     setOperationAction(ISD::CTLZ, MVT::i32, Custom);
 182:     setOperationAction(ISD::EH_DWARF_CFA, MVT::i32, Custom);
 183:     setOperationAction(ISD::READ_REGISTER, MVT::i32, Custom);
 184:     setOperationAction(ISD::WRITE_REGISTER, MVT::i32, Custom);
 185:     setOperationAction(ISD::INTRINSIC_VOID, MVT::i32, Custom);
 186:     setOperationAction(ISD::INTRINSIC_WO_CHAIN, MVT::i32, Custom);
 187:     setOperationAction(ISD::INTRINSIC_W_CHAIN, MVT::i32, Custom);
 188: 
 189:     setOperationAction(ISD::BITREVERSE, MVT::i32, Custom);
 190:     setOperationAction(ISD::BSWAP, MVT::i32, Custom);
 191:     setOperationAction({ISD::SDIV, ISD::UDIV, ISD::SREM, ISD::UREM}, MVT::i32,
 192:                        Custom);
 193:     setOperationAction(ISD::LROUND, MVT::i32, Custom);
 194:   }
 195: 
 196:   // Set operations for LA32 only.
 197: 
 198:   if (!Subtarget.is64Bit()) {
 199:     setOperationAction(ISD::READ_REGISTER, MVT::i64, Custom);
 200:     setOperationAction(ISD::WRITE_REGISTER, MVT::i64, Custom);
 201:     setOperationAction(ISD::INTRINSIC_VOID, MVT::i64, Custom);
 202:     setOperationAction(ISD::INTRINSIC_WO_CHAIN, MVT::i64, Custom);
 203:     setOperationAction(ISD::INTRINSIC_W_CHAIN, MVT::i64, Custom);
 204:     if (Subtarget.hasBasicD())
 205:       setOperationAction(ISD::BITCAST, MVT::i64, Custom);
 206:   }
 207: 
 208:   setOperationAction(ISD::ATOMIC_FENCE, MVT::Other, Custom);
 209: 
 210:   static const ISD::CondCode FPCCToExpand[] = {
 211:       ISD::SETOGT, ISD::SETOGE, ISD::SETUGT, ISD::SETUGE,
 212:       ISD::SETGE,  ISD::SETNE,  ISD::SETGT};
 213: 
 214:   // Set operations for 'F' feature.
 215: 
 216:   if (Subtarget.hasBasicF()) {
 217:     setLoadExtAction(ISD::EXTLOAD, MVT::f32, MVT::f16, Expand);
 218:     setTruncStoreAction(MVT::f32, MVT::f16, Expand);
 219:     setLoadExtAction(ISD::EXTLOAD, MVT::f32, MVT::bf16, Expand);
 220:     setTruncStoreAction(MVT::f32, MVT::bf16, Expand);
 221:     setCondCodeAction(FPCCToExpand, MVT::f32, Expand);
 222: 
 223:     setOperationAction(ISD::ConstantFP, MVT::f32, Custom);
 224:     setOperationAction(ISD::SELECT_CC, MVT::f32, Expand);
 225:     setOperationAction(ISD::BR_CC, MVT::f32, Expand);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 226-300 / 第 226-300 行
```cpp
 226:     setOperationAction(ISD::FMA, MVT::f32, Legal);
 227:     setOperationAction(ISD::FMINNUM_IEEE, MVT::f32, Legal);
 228:     setOperationAction(ISD::FMINNUM, MVT::f32, Legal);
 229:     setOperationAction(ISD::FMAXNUM_IEEE, MVT::f32, Legal);
 230:     setOperationAction(ISD::FMAXNUM, MVT::f32, Legal);
 231:     setOperationAction(ISD::FCANONICALIZE, MVT::f32, Legal);
 232:     setOperationAction(ISD::STRICT_FSETCCS, MVT::f32, Legal);
 233:     setOperationAction(ISD::STRICT_FSETCC, MVT::f32, Legal);
 234:     setOperationAction(ISD::IS_FPCLASS, MVT::f32, Legal);
 235:     setOperationAction(ISD::FSIN, MVT::f32, Expand);
 236:     setOperationAction(ISD::FCOS, MVT::f32, Expand);
 237:     setOperationAction(ISD::FSINCOS, MVT::f32, Expand);
 238:     setOperationAction(ISD::FPOW, MVT::f32, Expand);
 239:     setOperationAction(ISD::FREM, MVT::f32, LibCall);
 240:     setOperationAction(ISD::FP16_TO_FP, MVT::f32,
 241:                        Subtarget.isSoftFPABI() ? LibCall : Custom);
 242:     setOperationAction(ISD::FP_TO_FP16, MVT::f32,
 243:                        Subtarget.isSoftFPABI() ? LibCall : Custom);
 244:     setOperationAction(ISD::BF16_TO_FP, MVT::f32, Custom);
 245:     setOperationAction(ISD::FP_TO_BF16, MVT::f32,
 246:                        Subtarget.isSoftFPABI() ? LibCall : Custom);
 247: 
 248:     if (Subtarget.is64Bit())
 249:       setOperationAction(ISD::FRINT, MVT::f32, Legal);
 250: 
 251:     if (!Subtarget.hasBasicD()) {
 252:       setOperationAction(ISD::FP_TO_UINT, MVT::i32, Custom);
 253:       if (Subtarget.is64Bit()) {
 254:         setOperationAction(ISD::SINT_TO_FP, MVT::i64, Custom);
 255:         setOperationAction(ISD::UINT_TO_FP, MVT::i64, Custom);
 256:       }
 257:     }
 258:   }
 259: 
 260:   // Set operations for 'D' feature.
 261: 
 262:   if (Subtarget.hasBasicD()) {
 263:     setLoadExtAction(ISD::EXTLOAD, MVT::f64, MVT::f16, Expand);
 264:     setLoadExtAction(ISD::EXTLOAD, MVT::f64, MVT::f32, Expand);
 265:     setLoadExtAction(ISD::EXTLOAD, MVT::f64, MVT::bf16, Expand);
 266:     setTruncStoreAction(MVT::f64, MVT::bf16, Expand);
 267:     setTruncStoreAction(MVT::f64, MVT::f16, Expand);
 268:     setTruncStoreAction(MVT::f64, MVT::f32, Expand);
 269:     setCondCodeAction(FPCCToExpand, MVT::f64, Expand);
 270: 
 271:     setOperationAction(ISD::ConstantFP, MVT::f64, Custom);
 272:     setOperationAction(ISD::SELECT_CC, MVT::f64, Expand);
 273:     setOperationAction(ISD::BR_CC, MVT::f64, Expand);
 274:     setOperationAction(ISD::STRICT_FSETCCS, MVT::f64, Legal);
 275:     setOperationAction(ISD::STRICT_FSETCC, MVT::f64, Legal);
 276:     setOperationAction(ISD::FMA, MVT::f64, Legal);
 277:     setOperationAction(ISD::FMINNUM_IEEE, MVT::f64, Legal);
 278:     setOperationAction(ISD::FMINNUM, MVT::f64, Legal);
 279:     setOperationAction(ISD::FMAXNUM_IEEE, MVT::f64, Legal);
 280:     setOperationAction(ISD::FCANONICALIZE, MVT::f64, Legal);
 281:     setOperationAction(ISD::FMAXNUM, MVT::f64, Legal);
 282:     setOperationAction(ISD::IS_FPCLASS, MVT::f64, Legal);
 283:     setOperationAction(ISD::FSIN, MVT::f64, Expand);
 284:     setOperationAction(ISD::FCOS, MVT::f64, Expand);
 285:     setOperationAction(ISD::FSINCOS, MVT::f64, Expand);
 286:     setOperationAction(ISD::FPOW, MVT::f64, Expand);
 287:     setOperationAction(ISD::FREM, MVT::f64, LibCall);
 288:     setOperationAction(ISD::FP16_TO_FP, MVT::f64, Expand);
 289:     setOperationAction(ISD::FP_TO_FP16, MVT::f64,
 290:                        Subtarget.isSoftFPABI() ? LibCall : Custom);
 291:     setOperationAction(ISD::BF16_TO_FP, MVT::f64, Custom);
 292:     setOperationAction(ISD::FP_TO_BF16, MVT::f64,
 293:                        Subtarget.isSoftFPABI() ? LibCall : Custom);
 294: 
 295:     if (Subtarget.is64Bit())
 296:       setOperationAction(ISD::FRINT, MVT::f64, Legal);
 297:   }
 298: 
 299:   // Set operations for 'LSX' feature.
 300: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 301-375 / 第 301-375 行
```cpp
 301:   if (Subtarget.hasExtLSX()) {
 302:     for (MVT VT : MVT::fixedlen_vector_valuetypes()) {
 303:       // Expand all truncating stores and extending loads.
 304:       for (MVT InnerVT : MVT::fixedlen_vector_valuetypes()) {
 305:         setTruncStoreAction(VT, InnerVT, Expand);
 306:         setLoadExtAction(ISD::SEXTLOAD, VT, InnerVT, Expand);
 307:         setLoadExtAction(ISD::ZEXTLOAD, VT, InnerVT, Expand);
 308:         setLoadExtAction(ISD::EXTLOAD, VT, InnerVT, Expand);
 309:       }
 310:       // By default everything must be expanded. Then we will selectively turn
 311:       // on ones that can be effectively codegen'd.
 312:       for (unsigned Op = 0; Op < ISD::BUILTIN_OP_END; ++Op)
 313:         setOperationAction(Op, VT, Expand);
 314:     }
 315: 
 316:     for (MVT VT : LSXVTs) {
 317:       setOperationAction({ISD::LOAD, ISD::STORE}, VT, Legal);
 318:       setOperationAction(ISD::BITCAST, VT, Legal);
 319:       setOperationAction(ISD::UNDEF, VT, Legal);
 320: 
 321:       setOperationAction(ISD::INSERT_VECTOR_ELT, VT, Custom);
 322:       setOperationAction(ISD::EXTRACT_VECTOR_ELT, VT, Legal);
 323:       setOperationAction(ISD::BUILD_VECTOR, VT, Custom);
 324: 
 325:       setOperationAction(ISD::SETCC, VT, Legal);
 326:       setOperationAction(ISD::VSELECT, VT, Legal);
 327:       setOperationAction(ISD::VECTOR_SHUFFLE, VT, Custom);
 328:       setOperationAction(ISD::EXTRACT_SUBVECTOR, VT, Legal);
 329:     }
 330:     for (MVT VT : {MVT::v16i8, MVT::v8i16, MVT::v4i32, MVT::v2i64}) {
 331:       setOperationAction({ISD::ADD, ISD::SUB}, VT, Legal);
 332:       setOperationAction({ISD::UMAX, ISD::UMIN, ISD::SMAX, ISD::SMIN}, VT,
 333:                          Legal);
 334:       setOperationAction({ISD::MUL, ISD::SDIV, ISD::SREM, ISD::UDIV, ISD::UREM},
 335:                          VT, Legal);
 336:       setOperationAction({ISD::AND, ISD::OR, ISD::XOR}, VT, Legal);
 337:       setOperationAction({ISD::SHL, ISD::SRA, ISD::SRL}, VT, Legal);
 338:       setOperationAction({ISD::CTPOP, ISD::CTLZ}, VT, Legal);
 339:       setOperationAction({ISD::MULHS, ISD::MULHU}, VT, Legal);
 340:       setCondCodeAction(
 341:           {ISD::SETNE, ISD::SETGE, ISD::SETGT, ISD::SETUGE, ISD::SETUGT}, VT,
 342:           Expand);
 343:       setOperationAction(ISD::SCALAR_TO_VECTOR, VT, Custom);
 344:       setOperationAction(ISD::ABS, VT, Legal);
 345:       setOperationAction(ISD::ABDS, VT, Legal);
 346:       setOperationAction(ISD::ABDU, VT, Legal);
 347:       setOperationAction(ISD::SADDSAT, VT, Legal);
 348:       setOperationAction(ISD::SSUBSAT, VT, Legal);
 349:       setOperationAction(ISD::UADDSAT, VT, Legal);
 350:       setOperationAction(ISD::USUBSAT, VT, Legal);
 351:       setOperationAction(ISD::ROTL, VT, Custom);
 352:       setOperationAction(ISD::ROTR, VT, Custom);
 353:       setOperationAction(ISD::AVGFLOORS, VT, Legal);
 354:       setOperationAction(ISD::AVGFLOORU, VT, Legal);
 355:       setOperationAction(ISD::AVGCEILS, VT, Legal);
 356:       setOperationAction(ISD::AVGCEILU, VT, Legal);
 357:     }
 358:     for (MVT VT : {MVT::v16i8, MVT::v8i16, MVT::v4i32})
 359:       setOperationAction(ISD::BITREVERSE, VT, Custom);
 360:     for (MVT VT : {MVT::v8i16, MVT::v4i32, MVT::v2i64})
 361:       setOperationAction(ISD::BSWAP, VT, Legal);
 362:     for (MVT VT : {MVT::v4i32, MVT::v2i64}) {
 363:       setOperationAction({ISD::SINT_TO_FP, ISD::UINT_TO_FP}, VT, Legal);
 364:       setOperationAction({ISD::FP_TO_SINT, ISD::FP_TO_UINT}, VT, Legal);
 365:     }
 366:     for (MVT VT : {MVT::v4f32, MVT::v2f64}) {
 367:       setOperationAction({ISD::FADD, ISD::FSUB}, VT, Legal);
 368:       setOperationAction({ISD::FMUL, ISD::FDIV}, VT, Legal);
 369:       setOperationAction(ISD::FMA, VT, Legal);
 370:       setOperationAction(ISD::FSQRT, VT, Legal);
 371:       setOperationAction(ISD::FNEG, VT, Legal);
 372:       setCondCodeAction({ISD::SETGE, ISD::SETGT, ISD::SETOGE, ISD::SETOGT,
 373:                          ISD::SETUGE, ISD::SETUGT},
 374:                         VT, Expand);
 375:       setOperationAction(ISD::SCALAR_TO_VECTOR, VT, Legal);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 376-450 / 第 376-450 行
```cpp
 376:       setOperationAction(ISD::FCEIL, VT, Legal);
 377:       setOperationAction(ISD::FFLOOR, VT, Legal);
 378:       setOperationAction(ISD::FTRUNC, VT, Legal);
 379:       setOperationAction(ISD::FROUNDEVEN, VT, Legal);
 380:       setOperationAction(ISD::FMINNUM, VT, Legal);
 381:       setOperationAction(ISD::FMAXNUM, VT, Legal);
 382:     }
 383:     setOperationAction(ISD::CTPOP, GRLenVT, Legal);
 384:     setOperationAction(ISD::FCEIL, {MVT::f32, MVT::f64}, Legal);
 385:     setOperationAction(ISD::FFLOOR, {MVT::f32, MVT::f64}, Legal);
 386:     setOperationAction(ISD::FTRUNC, {MVT::f32, MVT::f64}, Legal);
 387:     setOperationAction(ISD::FROUNDEVEN, {MVT::f32, MVT::f64}, Legal);
 388: 
 389:     for (MVT VT :
 390:          {MVT::v16i8, MVT::v8i8, MVT::v4i8, MVT::v2i8, MVT::v8i16, MVT::v4i16,
 391:           MVT::v2i16, MVT::v4i32, MVT::v2i32, MVT::v2i64}) {
 392:       setOperationAction(ISD::TRUNCATE, VT, Custom);
 393:       setOperationAction(ISD::VECREDUCE_ADD, VT, Custom);
 394:       setOperationAction(ISD::VECREDUCE_AND, VT, Custom);
 395:       setOperationAction(ISD::VECREDUCE_OR, VT, Custom);
 396:       setOperationAction(ISD::VECREDUCE_XOR, VT, Custom);
 397:       setOperationAction(ISD::VECREDUCE_SMAX, VT, Custom);
 398:       setOperationAction(ISD::VECREDUCE_SMIN, VT, Custom);
 399:       setOperationAction(ISD::VECREDUCE_UMAX, VT, Custom);
 400:       setOperationAction(ISD::VECREDUCE_UMIN, VT, Custom);
 401:     }
 402:     setOperationAction(ISD::FP_ROUND, MVT::v2f32, Custom);
 403:     setOperationAction(ISD::FP_EXTEND, MVT::v2f32, Custom);
 404:     // We want to legalize this to an f64 load rather than an i64 load.
 405:     setOperationAction(ISD::LOAD, MVT::v2f32, Custom);
 406:     for (MVT VT : {MVT::v2i64, MVT::v4i32, MVT::v8i16})
 407:       setOperationAction(ISD::SIGN_EXTEND_VECTOR_INREG, VT, Custom);
 408:     for (MVT VT : {MVT::v16i16, MVT::v8i32, MVT::v4i64, MVT::v16i32, MVT::v8i64,
 409:                    MVT::v16i64})
 410:       setOperationAction(ISD::SIGN_EXTEND, VT, Custom);
 411:   }
 412: 
 413:   // Set operations for 'LASX' feature.
 414: 
 415:   if (Subtarget.hasExtLASX()) {
 416:     for (MVT VT : LASXVTs) {
 417:       setOperationAction({ISD::LOAD, ISD::STORE}, VT, Legal);
 418:       setOperationAction(ISD::BITCAST, VT, Legal);
 419:       setOperationAction(ISD::UNDEF, VT, Legal);
 420: 
 421:       setOperationAction(ISD::INSERT_VECTOR_ELT, VT, Custom);
 422:       setOperationAction(ISD::EXTRACT_VECTOR_ELT, VT, Custom);
 423:       setOperationAction(ISD::BUILD_VECTOR, VT, Custom);
 424:       setOperationAction(ISD::CONCAT_VECTORS, VT, Custom);
 425:       setOperationAction(ISD::INSERT_SUBVECTOR, VT, Legal);
 426: 
 427:       setOperationAction(ISD::SETCC, VT, Custom);
 428:       setOperationAction(ISD::VSELECT, VT, Legal);
 429:       setOperationAction(ISD::VECTOR_SHUFFLE, VT, Custom);
 430:     }
 431:     for (MVT VT : {MVT::v4i64, MVT::v8i32, MVT::v16i16, MVT::v32i8}) {
 432:       setOperationAction({ISD::ADD, ISD::SUB}, VT, Legal);
 433:       setOperationAction({ISD::UMAX, ISD::UMIN, ISD::SMAX, ISD::SMIN}, VT,
 434:                          Legal);
 435:       setOperationAction({ISD::MUL, ISD::SDIV, ISD::SREM, ISD::UDIV, ISD::UREM},
 436:                          VT, Legal);
 437:       setOperationAction({ISD::AND, ISD::OR, ISD::XOR}, VT, Legal);
 438:       setOperationAction({ISD::SHL, ISD::SRA, ISD::SRL}, VT, Legal);
 439:       setOperationAction({ISD::CTPOP, ISD::CTLZ}, VT, Legal);
 440:       setOperationAction({ISD::MULHS, ISD::MULHU}, VT, Legal);
 441:       setCondCodeAction(
 442:           {ISD::SETNE, ISD::SETGE, ISD::SETGT, ISD::SETUGE, ISD::SETUGT}, VT,
 443:           Expand);
 444:       setOperationAction(ISD::SCALAR_TO_VECTOR, VT, Custom);
 445:       setOperationAction(ISD::ABS, VT, Legal);
 446:       setOperationAction(ISD::ABDS, VT, Legal);
 447:       setOperationAction(ISD::ABDU, VT, Legal);
 448:       setOperationAction(ISD::SADDSAT, VT, Legal);
 449:       setOperationAction(ISD::SSUBSAT, VT, Legal);
 450:       setOperationAction(ISD::UADDSAT, VT, Legal);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 451-525 / 第 451-525 行
```cpp
 451:       setOperationAction(ISD::USUBSAT, VT, Legal);
 452:       setOperationAction(ISD::VECREDUCE_ADD, VT, Custom);
 453:       setOperationAction(ISD::ROTL, VT, Custom);
 454:       setOperationAction(ISD::ROTR, VT, Custom);
 455:       setOperationAction(ISD::AVGFLOORS, VT, Legal);
 456:       setOperationAction(ISD::AVGFLOORU, VT, Legal);
 457:       setOperationAction(ISD::AVGCEILS, VT, Legal);
 458:       setOperationAction(ISD::AVGCEILU, VT, Legal);
 459:     }
 460:     for (MVT VT : {MVT::v32i8, MVT::v16i16, MVT::v8i32})
 461:       setOperationAction(ISD::BITREVERSE, VT, Custom);
 462:     for (MVT VT : {MVT::v16i16, MVT::v8i32, MVT::v4i64})
 463:       setOperationAction(ISD::BSWAP, VT, Legal);
 464:     for (MVT VT : {MVT::v8i32, MVT::v4i32, MVT::v4i64}) {
 465:       setOperationAction({ISD::SINT_TO_FP, ISD::UINT_TO_FP}, VT, Legal);
 466:       setOperationAction({ISD::FP_TO_SINT, ISD::FP_TO_UINT}, VT, Legal);
 467:     }
 468:     for (MVT VT : {MVT::v8f32, MVT::v4f64}) {
 469:       setOperationAction({ISD::FADD, ISD::FSUB}, VT, Legal);
 470:       setOperationAction({ISD::FMUL, ISD::FDIV}, VT, Legal);
 471:       setOperationAction(ISD::FMA, VT, Legal);
 472:       setOperationAction(ISD::FSQRT, VT, Legal);
 473:       setOperationAction(ISD::FNEG, VT, Legal);
 474:       setCondCodeAction({ISD::SETGE, ISD::SETGT, ISD::SETOGE, ISD::SETOGT,
 475:                          ISD::SETUGE, ISD::SETUGT},
 476:                         VT, Expand);
 477:       setOperationAction(ISD::SCALAR_TO_VECTOR, VT, Legal);
 478:       setOperationAction(ISD::FCEIL, VT, Legal);
 479:       setOperationAction(ISD::FFLOOR, VT, Legal);
 480:       setOperationAction(ISD::FTRUNC, VT, Legal);
 481:       setOperationAction(ISD::FROUNDEVEN, VT, Legal);
 482:       setOperationAction(ISD::FMINNUM, VT, Legal);
 483:       setOperationAction(ISD::FMAXNUM, VT, Legal);
 484:     }
 485:     setOperationAction(ISD::FP_ROUND, MVT::v4f32, Custom);
 486:     setOperationAction(ISD::FP_EXTEND, MVT::v4f64, Custom);
 487:     for (MVT VT : {MVT::v4i64, MVT::v8i32, MVT::v16i16}) {
 488:       setOperationAction(ISD::SIGN_EXTEND, VT, Legal);
 489:       setOperationAction(ISD::ZERO_EXTEND, VT, Legal);
 490:     }
 491:     for (MVT VT :
 492:          {MVT::v2i64, MVT::v4i32, MVT::v4i64, MVT::v8i16, MVT::v8i32}) {
 493:       setOperationAction(ISD::SIGN_EXTEND_VECTOR_INREG, VT, Legal);
 494:       setOperationAction(ISD::ZERO_EXTEND_VECTOR_INREG, VT, Legal);
 495:     }
 496:   }
 497: 
 498:   // Set DAG combine for LA32 and LA64.
 499:   if (Subtarget.hasBasicF()) {
 500:     setTargetDAGCombine(ISD::SINT_TO_FP);
 501:   }
 502: 
 503:   setTargetDAGCombine(ISD::AND);
 504:   setTargetDAGCombine(ISD::OR);
 505:   setTargetDAGCombine(ISD::SRL);
 506:   setTargetDAGCombine(ISD::SETCC);
 507: 
 508:   // Set DAG combine for 'LSX' feature.
 509: 
 510:   if (Subtarget.hasExtLSX()) {
 511:     setTargetDAGCombine(ISD::ADD);
 512:     setTargetDAGCombine(ISD::INTRINSIC_WO_CHAIN);
 513:     setTargetDAGCombine(ISD::BITCAST);
 514:     setTargetDAGCombine(ISD::VSELECT);
 515:   }
 516: 
 517:   // Set DAG combine for 'LASX' feature.
 518:   if (Subtarget.hasExtLASX()) {
 519:     setTargetDAGCombine(ISD::ANY_EXTEND);
 520:     setTargetDAGCombine(ISD::ZERO_EXTEND);
 521:     setTargetDAGCombine(ISD::SIGN_EXTEND);
 522:     setTargetDAGCombine(ISD::CONCAT_VECTORS);
 523:   }
 524: 
 525:   // Compute derived properties from the register classes.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 526-600 / 第 526-600 行
```cpp
 526:   computeRegisterProperties(Subtarget.getRegisterInfo());
 527: 
 528:   setStackPointerRegisterToSaveRestore(LoongArch::R3);
 529: 
 530:   setBooleanContents(ZeroOrOneBooleanContent);
 531:   setBooleanVectorContents(ZeroOrNegativeOneBooleanContent);
 532: 
 533:   setMaxAtomicSizeInBitsSupported(Subtarget.getGRLen());
 534: 
 535:   setMinCmpXchgSizeInBits(32);
 536: 
 537:   // Function alignments.
 538:   setMinFunctionAlignment(Align(4));
 539:   // Set preferred alignments.
 540:   setPrefFunctionAlignment(Subtarget.getPrefFunctionAlignment());
 541:   setPrefLoopAlignment(Subtarget.getPrefLoopAlignment());
 542:   setMaxBytesForAlignment(Subtarget.getMaxBytesForAlignment());
 543: 
 544:   // cmpxchg sizes down to 8 bits become legal if LAMCAS is available.
 545:   if (Subtarget.hasLAMCAS())
 546:     setMinCmpXchgSizeInBits(8);
 547: 
 548:   if (Subtarget.hasSCQ()) {
 549:     setMaxAtomicSizeInBitsSupported(128);
 550:     setOperationAction(ISD::ATOMIC_CMP_SWAP, MVT::i128, Custom);
 551:   }
 552: 
 553:   // Disable strict node mutation.
 554:   IsStrictFPEnabled = true;
 555: }
 556: 
 557: bool LoongArchTargetLowering::isOffsetFoldingLegal(
 558:     const GlobalAddressSDNode *GA) const {
 559:   // In order to maximise the opportunity for common subexpression elimination,
 560:   // keep a separate ADD node for the global address offset instead of folding
 561:   // it in the global address node. Later peephole optimisations may choose to
 562:   // fold it back in when profitable.
 563:   return false;
 564: }
 565: 
 566: SDValue LoongArchTargetLowering::LowerOperation(SDValue Op,
 567:                                                 SelectionDAG &DAG) const {
 568:   switch (Op.getOpcode()) {
 569:   case ISD::ATOMIC_FENCE:
 570:     return lowerATOMIC_FENCE(Op, DAG);
 571:   case ISD::EH_DWARF_CFA:
 572:     return lowerEH_DWARF_CFA(Op, DAG);
 573:   case ISD::GlobalAddress:
 574:     return lowerGlobalAddress(Op, DAG);
 575:   case ISD::GlobalTLSAddress:
 576:     return lowerGlobalTLSAddress(Op, DAG);
 577:   case ISD::INTRINSIC_WO_CHAIN:
 578:     return lowerINTRINSIC_WO_CHAIN(Op, DAG);
 579:   case ISD::INTRINSIC_W_CHAIN:
 580:     return lowerINTRINSIC_W_CHAIN(Op, DAG);
 581:   case ISD::INTRINSIC_VOID:
 582:     return lowerINTRINSIC_VOID(Op, DAG);
 583:   case ISD::BlockAddress:
 584:     return lowerBlockAddress(Op, DAG);
 585:   case ISD::JumpTable:
 586:     return lowerJumpTable(Op, DAG);
 587:   case ISD::SHL_PARTS:
 588:     return lowerShiftLeftParts(Op, DAG);
 589:   case ISD::SRA_PARTS:
 590:     return lowerShiftRightParts(Op, DAG, true);
 591:   case ISD::SRL_PARTS:
 592:     return lowerShiftRightParts(Op, DAG, false);
 593:   case ISD::ConstantPool:
 594:     return lowerConstantPool(Op, DAG);
 595:   case ISD::FP_TO_SINT:
 596:     return lowerFP_TO_SINT(Op, DAG);
 597:   case ISD::BITCAST:
 598:     return lowerBITCAST(Op, DAG);
 599:   case ISD::UINT_TO_FP:
 600:     return lowerUINT_TO_FP(Op, DAG);
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::isOffsetFoldingLegal`, `LoongArchTargetLowering::LowerOperation`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::isOffsetFoldingLegal`, `LoongArchTargetLowering::LowerOperation` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 601-675 / 第 601-675 行
```cpp
 601:   case ISD::SINT_TO_FP:
 602:     return lowerSINT_TO_FP(Op, DAG);
 603:   case ISD::VASTART:
 604:     return lowerVASTART(Op, DAG);
 605:   case ISD::FRAMEADDR:
 606:     return lowerFRAMEADDR(Op, DAG);
 607:   case ISD::RETURNADDR:
 608:     return lowerRETURNADDR(Op, DAG);
 609:   case ISD::WRITE_REGISTER:
 610:     return lowerWRITE_REGISTER(Op, DAG);
 611:   case ISD::INSERT_VECTOR_ELT:
 612:     return lowerINSERT_VECTOR_ELT(Op, DAG);
 613:   case ISD::EXTRACT_VECTOR_ELT:
 614:     return lowerEXTRACT_VECTOR_ELT(Op, DAG);
 615:   case ISD::BUILD_VECTOR:
 616:     return lowerBUILD_VECTOR(Op, DAG);
 617:   case ISD::CONCAT_VECTORS:
 618:     return lowerCONCAT_VECTORS(Op, DAG);
 619:   case ISD::VECTOR_SHUFFLE:
 620:     return lowerVECTOR_SHUFFLE(Op, DAG);
 621:   case ISD::BITREVERSE:
 622:     return lowerBITREVERSE(Op, DAG);
 623:   case ISD::SCALAR_TO_VECTOR:
 624:     return lowerSCALAR_TO_VECTOR(Op, DAG);
 625:   case ISD::PREFETCH:
 626:     return lowerPREFETCH(Op, DAG);
 627:   case ISD::SELECT:
 628:     return lowerSELECT(Op, DAG);
 629:   case ISD::BRCOND:
 630:     return lowerBRCOND(Op, DAG);
 631:   case ISD::FP_TO_FP16:
 632:     return lowerFP_TO_FP16(Op, DAG);
 633:   case ISD::FP16_TO_FP:
 634:     return lowerFP16_TO_FP(Op, DAG);
 635:   case ISD::FP_TO_BF16:
 636:     return lowerFP_TO_BF16(Op, DAG);
 637:   case ISD::BF16_TO_FP:
 638:     return lowerBF16_TO_FP(Op, DAG);
 639:   case ISD::VECREDUCE_ADD:
 640:     return lowerVECREDUCE_ADD(Op, DAG);
 641:   case ISD::ROTL:
 642:   case ISD::ROTR:
 643:     return lowerRotate(Op, DAG);
 644:   case ISD::VECREDUCE_AND:
 645:   case ISD::VECREDUCE_OR:
 646:   case ISD::VECREDUCE_XOR:
 647:   case ISD::VECREDUCE_SMAX:
 648:   case ISD::VECREDUCE_SMIN:
 649:   case ISD::VECREDUCE_UMAX:
 650:   case ISD::VECREDUCE_UMIN:
 651:     return lowerVECREDUCE(Op, DAG);
 652:   case ISD::ConstantFP:
 653:     return lowerConstantFP(Op, DAG);
 654:   case ISD::SETCC:
 655:     return lowerSETCC(Op, DAG);
 656:   case ISD::FP_ROUND:
 657:     return lowerFP_ROUND(Op, DAG);
 658:   case ISD::FP_EXTEND:
 659:     return lowerFP_EXTEND(Op, DAG);
 660:   case ISD::SIGN_EXTEND_VECTOR_INREG:
 661:     return lowerSIGN_EXTEND_VECTOR_INREG(Op, DAG);
 662:   }
 663:   return SDValue();
 664: }
 665: 
 666: // Helper to attempt to return a cheaper, bit-inverted version of \p V.
 667: static SDValue isNOT(SDValue V, SelectionDAG &DAG) {
 668:   // TODO: don't always ignore oneuse constraints.
 669:   V = peekThroughBitcasts(V);
 670:   EVT VT = V.getValueType();
 671: 
 672:   // Match not(xor X, -1) -> X.
 673:   if (V.getOpcode() == ISD::XOR &&
 674:       (ISD::isBuildVectorAllOnes(V.getOperand(1).getNode()) ||
 675:        isAllOnesConstant(V.getOperand(1))))
```
- **EN**: The range implements or declares functions including `isNOT`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isNOT` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 676-750 / 第 676-750 行
```cpp
 676:     return V.getOperand(0);
 677: 
 678:   // Match not(extract_subvector(not(X)) -> extract_subvector(X).
 679:   if (V.getOpcode() == ISD::EXTRACT_SUBVECTOR &&
 680:       (isNullConstant(V.getOperand(1)) || V.getOperand(0).hasOneUse())) {
 681:     if (SDValue Not = isNOT(V.getOperand(0), DAG)) {
 682:       Not = DAG.getBitcast(V.getOperand(0).getValueType(), Not);
 683:       return DAG.getNode(ISD::EXTRACT_SUBVECTOR, SDLoc(Not), VT, Not,
 684:                          V.getOperand(1));
 685:     }
 686:   }
 687: 
 688:   // Match not(SplatVector(not(X)) -> SplatVector(X).
 689:   if (V.getOpcode() == ISD::BUILD_VECTOR) {
 690:     if (SDValue SplatValue =
 691:             cast<BuildVectorSDNode>(V.getNode())->getSplatValue()) {
 692:       if (!V->isOnlyUserOf(SplatValue.getNode()))
 693:         return SDValue();
 694: 
 695:       if (SDValue Not = isNOT(SplatValue, DAG)) {
 696:         Not = DAG.getBitcast(V.getOperand(0).getValueType(), Not);
 697:         return DAG.getSplat(VT, SDLoc(Not), Not);
 698:       }
 699:     }
 700:   }
 701: 
 702:   // Match not(or(not(X),not(Y))) -> and(X, Y).
 703:   if (V.getOpcode() == ISD::OR && DAG.getTargetLoweringInfo().isTypeLegal(VT) &&
 704:       V.getOperand(0).hasOneUse() && V.getOperand(1).hasOneUse()) {
 705:     // TODO: Handle cases with single NOT operand -> VANDN
 706:     if (SDValue Op1 = isNOT(V.getOperand(1), DAG))
 707:       if (SDValue Op0 = isNOT(V.getOperand(0), DAG))
 708:         return DAG.getNode(ISD::AND, SDLoc(V), VT, DAG.getBitcast(VT, Op0),
 709:                            DAG.getBitcast(VT, Op1));
 710:   }
 711: 
 712:   // TODO: Add more matching patterns. Such as,
 713:   // not(concat_vectors(not(X), not(Y))) -> concat_vectors(X, Y).
 714:   // not(slt(C, X)) -> slt(X - 1, C)
 715:   return SDValue();
 716: }
 717: 
 718: // Combine two ISD::FP_ROUND / LoongArchISD::VFCVT nodes with same type to
 719: // LoongArchISD::VFCVT. For example:
 720: //   x1 = fp_round x, 0
 721: //   y1 = fp_round y, 0
 722: //   z = concat_vectors x1, y1
 723: // Or
 724: //   x1 = LoongArch::VFCVT undef, x
 725: //   y1 = LoongArch::VFCVT undef, y
 726: //   z = LoongArchISD::VPACKEV y1, x1; or LoongArchISD::VPERMI y1, x1, 68
 727: // can be combined to:
 728: //   z = LoongArch::VFCVT y, x
 729: static SDValue combineFP_ROUND(SDValue N, const SDLoc &DL, SelectionDAG &DAG,
 730:                                const LoongArchSubtarget &Subtarget) {
 731:   assert(((N->getOpcode() == ISD::CONCAT_VECTORS && N->getNumOperands() == 2) ||
 732:           (N->getOpcode() == LoongArchISD::VPACKEV) ||
 733:           (N->getOpcode() == LoongArchISD::VPERMI)) &&
 734:          "Invalid Node");
 735: 
 736:   SDValue Op0 = peekThroughBitcasts(N->getOperand(0));
 737:   SDValue Op1 = peekThroughBitcasts(N->getOperand(1));
 738:   unsigned Opcode0 = Op0.getOpcode();
 739:   unsigned Opcode1 = Op1.getOpcode();
 740:   if (Opcode0 != Opcode1)
 741:     return SDValue();
 742: 
 743:   if (Opcode0 != ISD::FP_ROUND && Opcode0 != LoongArchISD::VFCVT)
 744:     return SDValue();
 745: 
 746:   // Check if two nodes have only one use.
 747:   if (!Op0.hasOneUse() || !Op1.hasOneUse())
 748:     return SDValue();
 749: 
 750:   EVT VT = N.getValueType();
```
- **EN**: The range implements or declares functions including `combineFP_ROUND`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `combineFP_ROUND` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 751-825 / 第 751-825 行
```cpp
 751:   EVT SVT0 = Op0.getValueType();
 752:   EVT SVT1 = Op1.getValueType();
 753:   // Check if two nodes have the same result type.
 754:   if (SVT0 != SVT1)
 755:     return SDValue();
 756: 
 757:   // Check if two nodes have the same operand type.
 758:   EVT SSVT0 = Op0.getOperand(0).getValueType();
 759:   EVT SSVT1 = Op1.getOperand(0).getValueType();
 760:   if (SSVT0 != SSVT1)
 761:     return SDValue();
 762: 
 763:   if (N->getOpcode() == ISD::CONCAT_VECTORS && Opcode0 == ISD::FP_ROUND) {
 764:     if (Subtarget.hasExtLASX() && VT.is256BitVector() && SVT0 == MVT::v4f32 &&
 765:         SSVT0 == MVT::v4f64) {
 766:       // A vector_shuffle is required in the final step, as xvfcvt instruction
 767:       // operates on each 128-bit segament as a lane.
 768:       SDValue Res = DAG.getNode(LoongArchISD::VFCVT, DL, MVT::v8f32,
 769:                                 Op1.getOperand(0), Op0.getOperand(0));
 770:       SDValue Undef = DAG.getUNDEF(Res.getValueType());
 771:       // After VFCVT, the high part of Res comes from the high parts of Op0 and
 772:       // Op1, and the low part comes from the low parts of Op0 and Op1. However,
 773:       // the desired order requires Op0 to fully occupy the lower half and Op1
 774:       // the upper half of Res. The Mask reorders the elements of Res to achieve
 775:       // this:
 776:       // - The first four elements (0, 1, 4, 5) come from Op0.
 777:       // - The next four elements (2, 3, 6, 7) come from Op1.
 778:       SmallVector<int, 8> Mask = {0, 1, 4, 5, 2, 3, 6, 7};
 779:       Res = DAG.getVectorShuffle(Res.getValueType(), DL, Res, Undef, Mask);
 780:       return DAG.getBitcast(VT, Res);
 781:     }
 782:   }
 783: 
 784:   if ((N->getOpcode() == LoongArchISD::VPACKEV ||
 785:        N->getOpcode() == LoongArchISD::VPERMI) &&
 786:       Opcode0 == LoongArchISD::VFCVT) {
 787:     // For VPACKEV or VPERMI, check if the first operation of VFCVT is undef.
 788:     if (!Op0.getOperand(0).isUndef() || !Op1.getOperand(0).isUndef())
 789:       return SDValue();
 790: 
 791:     if (!Subtarget.hasExtLSX() || SVT0 != MVT::v4f32 || SSVT0 != MVT::v2f64)
 792:       return SDValue();
 793: 
 794:     if (N->getOpcode() == LoongArchISD::VPACKEV &&
 795:         (VT == MVT::v2i64 || VT == MVT::v2f64)) {
 796:       SDValue Res = DAG.getNode(LoongArchISD::VFCVT, DL, MVT::v4f32,
 797:                                 Op0.getOperand(1), Op1.getOperand(1));
 798:       return DAG.getBitcast(VT, Res);
 799:     }
 800: 
 801:     if (N->getOpcode() == LoongArchISD::VPERMI && VT == MVT::v4f32) {
 802:       int64_t Imm = cast<ConstantSDNode>(N->getOperand(2))->getSExtValue();
 803:       if (Imm != 68)
 804:         return SDValue();
 805:       return DAG.getNode(LoongArchISD::VFCVT, DL, MVT::v4f32, Op0.getOperand(1),
 806:                          Op1.getOperand(1));
 807:     }
 808:   }
 809: 
 810:   return SDValue();
 811: }
 812: 
 813: SDValue LoongArchTargetLowering::lowerFP_ROUND(SDValue Op,
 814:                                                SelectionDAG &DAG) const {
 815:   SDLoc DL(Op);
 816:   SDValue In = Op.getOperand(0);
 817:   MVT VT = Op.getSimpleValueType();
 818:   MVT SVT = In.getSimpleValueType();
 819: 
 820:   if (VT == MVT::v4f32 && SVT == MVT::v4f64) {
 821:     SDValue Lo, Hi;
 822:     std::tie(Lo, Hi) = DAG.SplitVector(In, DL);
 823:     return DAG.getNode(LoongArchISD::VFCVT, DL, VT, Hi, Lo);
 824:   }
 825: 
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerFP_ROUND`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerFP_ROUND` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 826-900 / 第 826-900 行
```cpp
 826:   return SDValue();
 827: }
 828: 
 829: SDValue LoongArchTargetLowering::lowerFP_EXTEND(SDValue Op,
 830:                                                 SelectionDAG &DAG) const {
 831: 
 832:   SDLoc DL(Op);
 833:   EVT VT = Op.getValueType();
 834:   SDValue Src = Op->getOperand(0);
 835:   EVT SVT = Src.getValueType();
 836: 
 837:   bool V2F32ToV2F64 =
 838:       VT == MVT::v2f64 && SVT == MVT::v2f32 && Subtarget.hasExtLSX();
 839:   bool V4F32ToV4F64 =
 840:       VT == MVT::v4f64 && SVT == MVT::v4f32 && Subtarget.hasExtLASX();
 841:   if (!V2F32ToV2F64 && !V4F32ToV4F64)
 842:     return SDValue();
 843: 
 844:   // Check if Op is the high part of vector.
 845:   auto CheckVecHighPart = [](SDValue Op) {
 846:     Op = peekThroughBitcasts(Op);
 847:     if (Op.getOpcode() == ISD::EXTRACT_SUBVECTOR) {
 848:       SDValue SOp = Op.getOperand(0);
 849:       EVT SVT = SOp.getValueType();
 850:       if (!SVT.isVector() || (SVT.getVectorNumElements() % 2 != 0))
 851:         return SDValue();
 852: 
 853:       const uint64_t Imm = Op.getConstantOperandVal(1);
 854:       if (Imm == SVT.getVectorNumElements() / 2)
 855:         return SOp;
 856:       return SDValue();
 857:     }
 858:     return SDValue();
 859:   };
 860: 
 861:   unsigned Opcode;
 862:   SDValue VFCVTOp;
 863:   EVT WideOpVT = SVT.getSimpleVT().getDoubleNumVectorElementsVT();
 864:   SDValue ZeroIdx = DAG.getVectorIdxConstant(0, DL);
 865: 
 866:   // If the operand of ISD::FP_EXTEND comes from the high part of vector,
 867:   // generate LoongArchISD::VFCVTH, otherwise LoongArchISD::VFCVTL.
 868:   if (SDValue V = CheckVecHighPart(Src)) {
 869:     assert(V.getValueSizeInBits() == WideOpVT.getSizeInBits() &&
 870:            "Unexpected wide vector");
 871:     Opcode = LoongArchISD::VFCVTH;
 872:     VFCVTOp = DAG.getBitcast(WideOpVT, V);
 873:   } else {
 874:     Opcode = LoongArchISD::VFCVTL;
 875:     VFCVTOp = DAG.getNode(ISD::INSERT_SUBVECTOR, DL, WideOpVT,
 876:                           DAG.getUNDEF(WideOpVT), Src, ZeroIdx);
 877:   }
 878: 
 879:   // v2f64 = fp_extend v2f32
 880:   if (V2F32ToV2F64)
 881:     return DAG.getNode(Opcode, DL, VT, VFCVTOp);
 882: 
 883:   // v4f64 = fp_extend v4f32
 884:   if (V4F32ToV4F64) {
 885:     // XVFCVT instruction operates on each 128-bit segment as a lane, so a
 886:     // vector_shuffle is required firstly.
 887:     SmallVector<int, 8> Mask = {0, 1, 4, 5, 2, 3, 6, 7};
 888:     SDValue Res = DAG.getVectorShuffle(WideOpVT, DL, VFCVTOp,
 889:                                        DAG.getUNDEF(WideOpVT), Mask);
 890:     Res = DAG.getNode(Opcode, DL, VT, Res);
 891:     return Res;
 892:   }
 893: 
 894:   return SDValue();
 895: }
 896: 
 897: SDValue LoongArchTargetLowering::lowerConstantFP(SDValue Op,
 898:                                                  SelectionDAG &DAG) const {
 899:   EVT VT = Op.getValueType();
 900:   ConstantFPSDNode *CFP = cast<ConstantFPSDNode>(Op);
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerFP_EXTEND`, `LoongArchTargetLowering::lowerConstantFP`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerFP_EXTEND`, `LoongArchTargetLowering::lowerConstantFP` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 901-975 / 第 901-975 行
```cpp
 901:   const APFloat &FPVal = CFP->getValueAPF();
 902:   SDLoc DL(CFP);
 903: 
 904:   assert((VT == MVT::f32 && Subtarget.hasBasicF()) ||
 905:          (VT == MVT::f64 && Subtarget.hasBasicD()));
 906: 
 907:   // If value is 0.0 or -0.0, just ignore it.
 908:   if (FPVal.isZero())
 909:     return SDValue();
 910: 
 911:   // If lsx enabled, use cheaper 'vldi' instruction if possible.
 912:   if (isFPImmVLDILegal(FPVal, VT))
 913:     return SDValue();
 914: 
 915:   // Construct as integer, and move to float register.
 916:   APInt INTVal = FPVal.bitcastToAPInt();
 917: 
 918:   // If more than MaterializeFPImmInsNum instructions will be used to
 919:   // generate the INTVal and move it to float register, fallback to
 920:   // use floating point load from the constant pool.
 921:   auto Seq = LoongArchMatInt::generateInstSeq(INTVal.getSExtValue());
 922:   int InsNum = Seq.size() + ((VT == MVT::f64 && !Subtarget.is64Bit()) ? 2 : 1);
 923:   if (InsNum > MaterializeFPImmInsNum && !FPVal.isExactlyValue(+1.0))
 924:     return SDValue();
 925: 
 926:   switch (VT.getSimpleVT().SimpleTy) {
 927:   default:
 928:     llvm_unreachable("Unexpected floating point type!");
 929:     break;
 930:   case MVT::f32: {
 931:     SDValue NewVal = DAG.getConstant(INTVal, DL, MVT::i32);
 932:     if (Subtarget.is64Bit())
 933:       NewVal = DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, NewVal);
 934:     return DAG.getNode(Subtarget.is64Bit() ? LoongArchISD::MOVGR2FR_W_LA64
 935:                                            : LoongArchISD::MOVGR2FR_W,
 936:                        DL, VT, NewVal);
 937:   }
 938:   case MVT::f64: {
 939:     if (Subtarget.is64Bit()) {
 940:       SDValue NewVal = DAG.getConstant(INTVal, DL, MVT::i64);
 941:       return DAG.getNode(LoongArchISD::MOVGR2FR_D, DL, VT, NewVal);
 942:     }
 943:     SDValue Lo = DAG.getConstant(INTVal.trunc(32), DL, MVT::i32);
 944:     SDValue Hi = DAG.getConstant(INTVal.lshr(32).trunc(32), DL, MVT::i32);
 945:     return DAG.getNode(LoongArchISD::MOVGR2FR_D_LO_HI, DL, VT, Lo, Hi);
 946:   }
 947:   }
 948: 
 949:   return SDValue();
 950: }
 951: 
 952: // Ensure SETCC result and operand have the same bit width; isel does not
 953: // support mismatched widths.
 954: SDValue LoongArchTargetLowering::lowerSETCC(SDValue Op,
 955:                                             SelectionDAG &DAG) const {
 956:   SDLoc DL(Op);
 957:   EVT ResultVT = Op.getValueType();
 958:   EVT OperandVT = Op.getOperand(0).getValueType();
 959: 
 960:   EVT SetCCResultVT =
 961:       getSetCCResultType(DAG.getDataLayout(), *DAG.getContext(), OperandVT);
 962: 
 963:   if (ResultVT == SetCCResultVT)
 964:     return Op;
 965: 
 966:   assert(Op.getOperand(0).getValueType() == Op.getOperand(1).getValueType() &&
 967:          "SETCC operands must have the same type!");
 968: 
 969:   SDValue SetCCNode =
 970:       DAG.getNode(ISD::SETCC, DL, SetCCResultVT, Op.getOperand(0),
 971:                   Op.getOperand(1), Op.getOperand(2));
 972: 
 973:   if (ResultVT.bitsGT(SetCCResultVT))
 974:     SetCCNode = DAG.getNode(ISD::SIGN_EXTEND, DL, ResultVT, SetCCNode);
 975:   else if (ResultVT.bitsLT(SetCCResultVT))
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerSETCC`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerSETCC` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 976-1050 / 第 976-1050 行
```cpp
 976:     SetCCNode = DAG.getNode(ISD::TRUNCATE, DL, ResultVT, SetCCNode);
 977: 
 978:   return SetCCNode;
 979: }
 980: 
 981: // Lower sext_invec using vslti instructions.
 982: // For example:
 983: //  %b = sext <4 x i16> %a to <4 x i32>
 984: // can be lowered to:
 985: //  VSLTI_H vr2, vr1, 0
 986: //  VILVL.H vr1, vr2, vr1
 987: SDValue LoongArchTargetLowering::lowerSIGN_EXTEND_VECTOR_INREG(
 988:     SDValue Op, SelectionDAG &DAG) const {
 989:   SDLoc DL(Op);
 990:   SDValue Src = Op.getOperand(0);
 991:   MVT SrcVT = Src.getSimpleValueType();
 992:   MVT DstVT = Op.getSimpleValueType();
 993: 
 994:   if (!SrcVT.is128BitVector())
 995:     return SDValue();
 996: 
 997:   // lower to VSLTI + VILVL if extend could be done in single step.
 998:   if (DstVT.getScalarSizeInBits() / SrcVT.getScalarSizeInBits() == 2) {
 999:     SDValue Zero = DAG.getConstant(0, DL, SrcVT);
1000:     SDValue Mask = DAG.getNode(ISD::SETCC, DL, SrcVT, Src, Zero,
1001:                                DAG.getCondCode(ISD::SETLT));
1002:     SDValue LoInterleaved =
1003:         DAG.getNode(LoongArchISD::VILVL, DL, SrcVT, Mask, Src);
1004: 
1005:     return DAG.getBitcast(DstVT, LoInterleaved);
1006:   }
1007: 
1008:   return SDValue();
1009: }
1010: 
1011: // Lower vecreduce_add using vhaddw instructions.
1012: // For Example:
1013: //  call i32 @llvm.vector.reduce.add.v4i32(<4 x i32> %a)
1014: // can be lowered to:
1015: //  VHADDW_D_W    vr0, vr0, vr0
1016: //  VHADDW_Q_D    vr0, vr0, vr0
1017: //  VPICKVE2GR_D  a0,  vr0, 0
1018: //  ADDI_W        a0,  a0,  0
1019: SDValue LoongArchTargetLowering::lowerVECREDUCE_ADD(SDValue Op,
1020:                                                     SelectionDAG &DAG) const {
1021: 
1022:   SDLoc DL(Op);
1023:   MVT OpVT = Op.getSimpleValueType();
1024:   SDValue Val = Op.getOperand(0);
1025: 
1026:   unsigned NumEles = Val.getSimpleValueType().getVectorNumElements();
1027:   unsigned EleBits = Val.getSimpleValueType().getScalarSizeInBits();
1028:   unsigned ResBits = OpVT.getScalarSizeInBits();
1029: 
1030:   unsigned LegalVecSize = 128;
1031:   bool isLASX256Vector =
1032:       Subtarget.hasExtLASX() && Val.getValueSizeInBits() == 256;
1033: 
1034:   // Ensure operand type legal or enable it legal.
1035:   while (!isTypeLegal(Val.getSimpleValueType())) {
1036:     Val = DAG.WidenVector(Val, DL);
1037:   }
1038: 
1039:   // NumEles is designed for iterations count, v4i32 for LSX
1040:   // and v8i32 for LASX should have the same count.
1041:   if (isLASX256Vector) {
1042:     NumEles /= 2;
1043:     LegalVecSize = 256;
1044:   }
1045: 
1046:   for (unsigned i = 1; i < NumEles; i *= 2, EleBits *= 2) {
1047:     MVT IntTy = MVT::getIntegerVT(EleBits);
1048:     MVT VecTy = MVT::getVectorVT(IntTy, LegalVecSize / EleBits);
1049:     Val = DAG.getNode(LoongArchISD::VHADDW, DL, VecTy, Val, Val);
1050:   }
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerSIGN_EXTEND_VECTOR_INREG`, `LoongArchTargetLowering::lowerVECREDUCE_ADD`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerSIGN_EXTEND_VECTOR_INREG`, `LoongArchTargetLowering::lowerVECREDUCE_ADD` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1051-1125 / 第 1051-1125 行
```cpp
1051: 
1052:   if (isLASX256Vector) {
1053:     SDValue Tmp = DAG.getNode(LoongArchISD::XVPERMI, DL, MVT::v4i64, Val,
1054:                               DAG.getConstant(2, DL, Subtarget.getGRLenVT()));
1055:     Val = DAG.getNode(ISD::ADD, DL, MVT::v4i64, Tmp, Val);
1056:   }
1057: 
1058:   Val = DAG.getBitcast(MVT::getVectorVT(OpVT, LegalVecSize / ResBits), Val);
1059:   return DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, OpVT, Val,
1060:                      DAG.getConstant(0, DL, Subtarget.getGRLenVT()));
1061: }
1062: 
1063: // Lower vecreduce_and/or/xor/[s/u]max/[s/u]min.
1064: // For Example:
1065: //  call i32 @llvm.vector.reduce.smax.v4i32(<4 x i32> %a)
1066: // can be lowered to:
1067: //  VBSRL_V       vr1, vr0, 8
1068: //  VMAX_W        vr0, vr1, vr0
1069: //  VBSRL_V       vr1, vr0, 4
1070: //  VMAX_W        vr0, vr1, vr0
1071: //  VPICKVE2GR_W  a0,  vr0, 0
1072: // For 256 bit vector, it is illegal and will be spilt into
1073: // two 128 bit vector by default then processed by this.
1074: SDValue LoongArchTargetLowering::lowerVECREDUCE(SDValue Op,
1075:                                                 SelectionDAG &DAG) const {
1076:   SDLoc DL(Op);
1077: 
1078:   MVT OpVT = Op.getSimpleValueType();
1079:   SDValue Val = Op.getOperand(0);
1080: 
1081:   unsigned NumEles = Val.getSimpleValueType().getVectorNumElements();
1082:   unsigned EleBits = Val.getSimpleValueType().getScalarSizeInBits();
1083: 
1084:   // Ensure operand type legal or enable it legal.
1085:   while (!isTypeLegal(Val.getSimpleValueType())) {
1086:     Val = DAG.WidenVector(Val, DL);
1087:   }
1088: 
1089:   unsigned Opcode = ISD::getVecReduceBaseOpcode(Op.getOpcode());
1090:   MVT VecTy = Val.getSimpleValueType();
1091:   MVT GRLenVT = Subtarget.getGRLenVT();
1092: 
1093:   for (int i = NumEles; i > 1; i /= 2) {
1094:     SDValue ShiftAmt = DAG.getConstant(i * EleBits / 16, DL, GRLenVT);
1095:     SDValue Tmp = DAG.getNode(LoongArchISD::VBSRL, DL, VecTy, Val, ShiftAmt);
1096:     Val = DAG.getNode(Opcode, DL, VecTy, Tmp, Val);
1097:   }
1098: 
1099:   return DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, OpVT, Val,
1100:                      DAG.getConstant(0, DL, GRLenVT));
1101: }
1102: 
1103: SDValue LoongArchTargetLowering::lowerPREFETCH(SDValue Op,
1104:                                                SelectionDAG &DAG) const {
1105:   unsigned IsData = Op.getConstantOperandVal(4);
1106: 
1107:   // We don't support non-data prefetch.
1108:   // Just preserve the chain.
1109:   if (!IsData)
1110:     return Op.getOperand(0);
1111: 
1112:   return Op;
1113: }
1114: 
1115: SDValue LoongArchTargetLowering::lowerRotate(SDValue Op,
1116:                                              SelectionDAG &DAG) const {
1117:   MVT VT = Op.getSimpleValueType();
1118:   assert(VT.isVector() && "Unexpected type");
1119: 
1120:   SDLoc DL(Op);
1121:   SDValue R = Op.getOperand(0);
1122:   SDValue Amt = Op.getOperand(1);
1123:   unsigned Opcode = Op.getOpcode();
1124:   unsigned EltSizeInBits = VT.getScalarSizeInBits();
1125: 
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerVECREDUCE`, `LoongArchTargetLowering::lowerPREFETCH`, `LoongArchTargetLowering::lowerRotate`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerVECREDUCE`, `LoongArchTargetLowering::lowerPREFETCH`, `LoongArchTargetLowering::lowerRotate` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1126-1200 / 第 1126-1200 行
```cpp
1126:   auto checkCstSplat = [](SDValue V, APInt &CstSplatValue) {
1127:     if (V.getOpcode() != ISD::BUILD_VECTOR)
1128:       return false;
1129:     if (SDValue SplatValue =
1130:             cast<BuildVectorSDNode>(V.getNode())->getSplatValue()) {
1131:       if (auto *C = dyn_cast<ConstantSDNode>(SplatValue)) {
1132:         CstSplatValue = C->getAPIntValue();
1133:         return true;
1134:       }
1135:     }
1136:     return false;
1137:   };
1138: 
1139:   // Check for constant splat rotation amount.
1140:   APInt CstSplatValue;
1141:   bool IsCstSplat = checkCstSplat(Amt, CstSplatValue);
1142:   bool isROTL = Opcode == ISD::ROTL;
1143: 
1144:   // Check for splat rotate by zero.
1145:   if (IsCstSplat && CstSplatValue.urem(EltSizeInBits) == 0)
1146:     return R;
1147: 
1148:   // LoongArch targets always prefer ISD::ROTR.
1149:   if (isROTL) {
1150:     SDValue Zero = DAG.getConstant(0, DL, VT);
1151:     return DAG.getNode(ISD::ROTR, DL, VT, R,
1152:                        DAG.getNode(ISD::SUB, DL, VT, Zero, Amt));
1153:   }
1154: 
1155:   // Rotate by a immediate.
1156:   if (IsCstSplat) {
1157:     // ISD::ROTR: Attemp to rotate by a positive immediate.
1158:     SDValue Bits = DAG.getConstant(EltSizeInBits, DL, VT);
1159:     if (SDValue Urem =
1160:             DAG.FoldConstantArithmetic(ISD::UREM, DL, VT, {Amt, Bits}))
1161:       return DAG.getNode(Opcode, DL, VT, R, Urem);
1162:   }
1163: 
1164:   return Op;
1165: }
1166: 
1167: // Return true if Val is equal to (setcc LHS, RHS, CC).
1168: // Return false if Val is the inverse of (setcc LHS, RHS, CC).
1169: // Otherwise, return std::nullopt.
1170: static std::optional<bool> matchSetCC(SDValue LHS, SDValue RHS,
1171:                                       ISD::CondCode CC, SDValue Val) {
1172:   assert(Val->getOpcode() == ISD::SETCC);
1173:   SDValue LHS2 = Val.getOperand(0);
1174:   SDValue RHS2 = Val.getOperand(1);
1175:   ISD::CondCode CC2 = cast<CondCodeSDNode>(Val.getOperand(2))->get();
1176: 
1177:   if (LHS == LHS2 && RHS == RHS2) {
1178:     if (CC == CC2)
1179:       return true;
1180:     if (CC == ISD::getSetCCInverse(CC2, LHS2.getValueType()))
1181:       return false;
1182:   } else if (LHS == RHS2 && RHS == LHS2) {
1183:     CC2 = ISD::getSetCCSwappedOperands(CC2);
1184:     if (CC == CC2)
1185:       return true;
1186:     if (CC == ISD::getSetCCInverse(CC2, LHS2.getValueType()))
1187:       return false;
1188:   }
1189: 
1190:   return std::nullopt;
1191: }
1192: 
1193: static SDValue combineSelectToBinOp(SDNode *N, SelectionDAG &DAG,
1194:                                     const LoongArchSubtarget &Subtarget) {
1195:   SDValue CondV = N->getOperand(0);
1196:   SDValue TrueV = N->getOperand(1);
1197:   SDValue FalseV = N->getOperand(2);
1198:   MVT VT = N->getSimpleValueType(0);
1199:   SDLoc DL(N);
1200: 
```
- **EN**: The range implements or declares functions including `matchSetCC`, `combineSelectToBinOp`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `matchSetCC`, `combineSelectToBinOp` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1201-1275 / 第 1201-1275 行
```cpp
1201:   // (select c, -1, y) -> -c | y
1202:   if (isAllOnesConstant(TrueV)) {
1203:     SDValue Neg = DAG.getNegative(CondV, DL, VT);
1204:     return DAG.getNode(ISD::OR, DL, VT, Neg, DAG.getFreeze(FalseV));
1205:   }
1206:   // (select c, y, -1) -> (c-1) | y
1207:   if (isAllOnesConstant(FalseV)) {
1208:     SDValue Neg =
1209:         DAG.getNode(ISD::ADD, DL, VT, CondV, DAG.getAllOnesConstant(DL, VT));
1210:     return DAG.getNode(ISD::OR, DL, VT, Neg, DAG.getFreeze(TrueV));
1211:   }
1212: 
1213:   // (select c, 0, y) -> (c-1) & y
1214:   if (isNullConstant(TrueV)) {
1215:     SDValue Neg =
1216:         DAG.getNode(ISD::ADD, DL, VT, CondV, DAG.getAllOnesConstant(DL, VT));
1217:     return DAG.getNode(ISD::AND, DL, VT, Neg, DAG.getFreeze(FalseV));
1218:   }
1219:   // (select c, y, 0) -> -c & y
1220:   if (isNullConstant(FalseV)) {
1221:     SDValue Neg = DAG.getNegative(CondV, DL, VT);
1222:     return DAG.getNode(ISD::AND, DL, VT, Neg, DAG.getFreeze(TrueV));
1223:   }
1224: 
1225:   // select c, ~x, x --> xor -c, x
1226:   if (isa<ConstantSDNode>(TrueV) && isa<ConstantSDNode>(FalseV)) {
1227:     const APInt &TrueVal = TrueV->getAsAPIntVal();
1228:     const APInt &FalseVal = FalseV->getAsAPIntVal();
1229:     if (~TrueVal == FalseVal) {
1230:       SDValue Neg = DAG.getNegative(CondV, DL, VT);
1231:       return DAG.getNode(ISD::XOR, DL, VT, Neg, FalseV);
1232:     }
1233:   }
1234: 
1235:   // Try to fold (select (setcc lhs, rhs, cc), truev, falsev) into bitwise ops
1236:   // when both truev and falsev are also setcc.
1237:   if (CondV.getOpcode() == ISD::SETCC && TrueV.getOpcode() == ISD::SETCC &&
1238:       FalseV.getOpcode() == ISD::SETCC) {
1239:     SDValue LHS = CondV.getOperand(0);
1240:     SDValue RHS = CondV.getOperand(1);
1241:     ISD::CondCode CC = cast<CondCodeSDNode>(CondV.getOperand(2))->get();
1242: 
1243:     // (select x, x, y) -> x | y
1244:     // (select !x, x, y) -> x & y
1245:     if (std::optional<bool> MatchResult = matchSetCC(LHS, RHS, CC, TrueV)) {
1246:       return DAG.getNode(*MatchResult ? ISD::OR : ISD::AND, DL, VT, TrueV,
1247:                          DAG.getFreeze(FalseV));
1248:     }
1249:     // (select x, y, x) -> x & y
1250:     // (select !x, y, x) -> x | y
1251:     if (std::optional<bool> MatchResult = matchSetCC(LHS, RHS, CC, FalseV)) {
1252:       return DAG.getNode(*MatchResult ? ISD::AND : ISD::OR, DL, VT,
1253:                          DAG.getFreeze(TrueV), FalseV);
1254:     }
1255:   }
1256: 
1257:   return SDValue();
1258: }
1259: 
1260: // Transform `binOp (select cond, x, c0), c1` where `c0` and `c1` are constants
1261: // into `select cond, binOp(x, c1), binOp(c0, c1)` if profitable.
1262: // For now we only consider transformation profitable if `binOp(c0, c1)` ends up
1263: // being `0` or `-1`. In such cases we can replace `select` with `and`.
1264: // TODO: Should we also do this if `binOp(c0, c1)` is cheaper to materialize
1265: // than `c0`?
1266: static SDValue
1267: foldBinOpIntoSelectIfProfitable(SDNode *BO, SelectionDAG &DAG,
1268:                                 const LoongArchSubtarget &Subtarget) {
1269:   unsigned SelOpNo = 0;
1270:   SDValue Sel = BO->getOperand(0);
1271:   if (Sel.getOpcode() != ISD::SELECT || !Sel.hasOneUse()) {
1272:     SelOpNo = 1;
1273:     Sel = BO->getOperand(1);
1274:   }
1275: 
```
- **EN**: The range implements or declares functions including `foldBinOpIntoSelectIfProfitable`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `foldBinOpIntoSelectIfProfitable` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1276-1350 / 第 1276-1350 行
```cpp
1276:   if (Sel.getOpcode() != ISD::SELECT || !Sel.hasOneUse())
1277:     return SDValue();
1278: 
1279:   unsigned ConstSelOpNo = 1;
1280:   unsigned OtherSelOpNo = 2;
1281:   if (!isa<ConstantSDNode>(Sel->getOperand(ConstSelOpNo))) {
1282:     ConstSelOpNo = 2;
1283:     OtherSelOpNo = 1;
1284:   }
1285:   SDValue ConstSelOp = Sel->getOperand(ConstSelOpNo);
1286:   ConstantSDNode *ConstSelOpNode = dyn_cast<ConstantSDNode>(ConstSelOp);
1287:   if (!ConstSelOpNode || ConstSelOpNode->isOpaque())
1288:     return SDValue();
1289: 
1290:   SDValue ConstBinOp = BO->getOperand(SelOpNo ^ 1);
1291:   ConstantSDNode *ConstBinOpNode = dyn_cast<ConstantSDNode>(ConstBinOp);
1292:   if (!ConstBinOpNode || ConstBinOpNode->isOpaque())
1293:     return SDValue();
1294: 
1295:   SDLoc DL(Sel);
1296:   EVT VT = BO->getValueType(0);
1297: 
1298:   SDValue NewConstOps[2] = {ConstSelOp, ConstBinOp};
1299:   if (SelOpNo == 1)
1300:     std::swap(NewConstOps[0], NewConstOps[1]);
1301: 
1302:   SDValue NewConstOp =
1303:       DAG.FoldConstantArithmetic(BO->getOpcode(), DL, VT, NewConstOps);
1304:   if (!NewConstOp)
1305:     return SDValue();
1306: 
1307:   const APInt &NewConstAPInt = NewConstOp->getAsAPIntVal();
1308:   if (!NewConstAPInt.isZero() && !NewConstAPInt.isAllOnes())
1309:     return SDValue();
1310: 
1311:   SDValue OtherSelOp = Sel->getOperand(OtherSelOpNo);
1312:   SDValue NewNonConstOps[2] = {OtherSelOp, ConstBinOp};
1313:   if (SelOpNo == 1)
1314:     std::swap(NewNonConstOps[0], NewNonConstOps[1]);
1315:   SDValue NewNonConstOp = DAG.getNode(BO->getOpcode(), DL, VT, NewNonConstOps);
1316: 
1317:   SDValue NewT = (ConstSelOpNo == 1) ? NewConstOp : NewNonConstOp;
1318:   SDValue NewF = (ConstSelOpNo == 1) ? NewNonConstOp : NewConstOp;
1319:   return DAG.getSelect(DL, VT, Sel.getOperand(0), NewT, NewF);
1320: }
1321: 
1322: // Changes the condition code and swaps operands if necessary, so the SetCC
1323: // operation matches one of the comparisons supported directly by branches
1324: // in the LoongArch ISA. May adjust compares to favor compare with 0 over
1325: // compare with 1/-1.
1326: static void translateSetCCForBranch(const SDLoc &DL, SDValue &LHS, SDValue &RHS,
1327:                                     ISD::CondCode &CC, SelectionDAG &DAG) {
1328:   // If this is a single bit test that can't be handled by ANDI, shift the
1329:   // bit to be tested to the MSB and perform a signed compare with 0.
1330:   if (isIntEqualitySetCC(CC) && isNullConstant(RHS) &&
1331:       LHS.getOpcode() == ISD::AND && LHS.hasOneUse() &&
1332:       isa<ConstantSDNode>(LHS.getOperand(1))) {
1333:     uint64_t Mask = LHS.getConstantOperandVal(1);
1334:     if ((isPowerOf2_64(Mask) || isMask_64(Mask)) && !isInt<12>(Mask)) {
1335:       unsigned ShAmt = 0;
1336:       if (isPowerOf2_64(Mask)) {
1337:         CC = CC == ISD::SETEQ ? ISD::SETGE : ISD::SETLT;
1338:         ShAmt = LHS.getValueSizeInBits() - 1 - Log2_64(Mask);
1339:       } else {
1340:         ShAmt = LHS.getValueSizeInBits() - llvm::bit_width(Mask);
1341:       }
1342: 
1343:       LHS = LHS.getOperand(0);
1344:       if (ShAmt != 0)
1345:         LHS = DAG.getNode(ISD::SHL, DL, LHS.getValueType(), LHS,
1346:                           DAG.getConstant(ShAmt, DL, LHS.getValueType()));
1347:       return;
1348:     }
1349:   }
1350: 
```
- **EN**: The range implements or declares functions including `translateSetCCForBranch`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `translateSetCCForBranch` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1351-1425 / 第 1351-1425 行
```cpp
1351:   if (auto *RHSC = dyn_cast<ConstantSDNode>(RHS)) {
1352:     int64_t C = RHSC->getSExtValue();
1353:     switch (CC) {
1354:     default:
1355:       break;
1356:     case ISD::SETGT:
1357:       // Convert X > -1 to X >= 0.
1358:       if (C == -1) {
1359:         RHS = DAG.getConstant(0, DL, RHS.getValueType());
1360:         CC = ISD::SETGE;
1361:         return;
1362:       }
1363:       break;
1364:     case ISD::SETLT:
1365:       // Convert X < 1 to 0 >= X.
1366:       if (C == 1) {
1367:         RHS = LHS;
1368:         LHS = DAG.getConstant(0, DL, RHS.getValueType());
1369:         CC = ISD::SETGE;
1370:         return;
1371:       }
1372:       break;
1373:     }
1374:   }
1375: 
1376:   switch (CC) {
1377:   default:
1378:     break;
1379:   case ISD::SETGT:
1380:   case ISD::SETLE:
1381:   case ISD::SETUGT:
1382:   case ISD::SETULE:
1383:     CC = ISD::getSetCCSwappedOperands(CC);
1384:     std::swap(LHS, RHS);
1385:     break;
1386:   }
1387: }
1388: 
1389: SDValue LoongArchTargetLowering::lowerSELECT(SDValue Op,
1390:                                              SelectionDAG &DAG) const {
1391:   SDValue CondV = Op.getOperand(0);
1392:   SDValue TrueV = Op.getOperand(1);
1393:   SDValue FalseV = Op.getOperand(2);
1394:   SDLoc DL(Op);
1395:   MVT VT = Op.getSimpleValueType();
1396:   MVT GRLenVT = Subtarget.getGRLenVT();
1397: 
1398:   if (SDValue V = combineSelectToBinOp(Op.getNode(), DAG, Subtarget))
1399:     return V;
1400: 
1401:   if (Op.hasOneUse()) {
1402:     unsigned UseOpc = Op->user_begin()->getOpcode();
1403:     if (isBinOp(UseOpc) && DAG.isSafeToSpeculativelyExecute(UseOpc)) {
1404:       SDNode *BinOp = *Op->user_begin();
1405:       if (SDValue NewSel = foldBinOpIntoSelectIfProfitable(*Op->user_begin(),
1406:                                                            DAG, Subtarget)) {
1407:         DAG.ReplaceAllUsesWith(BinOp, &NewSel);
1408:         // Opcode check is necessary because foldBinOpIntoSelectIfProfitable
1409:         // may return a constant node and cause crash in lowerSELECT.
1410:         if (NewSel.getOpcode() == ISD::SELECT)
1411:           return lowerSELECT(NewSel, DAG);
1412:         return NewSel;
1413:       }
1414:     }
1415:   }
1416: 
1417:   // If the condition is not an integer SETCC which operates on GRLenVT, we need
1418:   // to emit a LoongArchISD::SELECT_CC comparing the condition to zero. i.e.:
1419:   // (select condv, truev, falsev)
1420:   // -> (loongarchisd::select_cc condv, zero, setne, truev, falsev)
1421:   if (CondV.getOpcode() != ISD::SETCC ||
1422:       CondV.getOperand(0).getSimpleValueType() != GRLenVT) {
1423:     SDValue Zero = DAG.getConstant(0, DL, GRLenVT);
1424:     SDValue SetNE = DAG.getCondCode(ISD::SETNE);
1425: 
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerSELECT`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerSELECT` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1426-1500 / 第 1426-1500 行
```cpp
1426:     SDValue Ops[] = {CondV, Zero, SetNE, TrueV, FalseV};
1427: 
1428:     return DAG.getNode(LoongArchISD::SELECT_CC, DL, VT, Ops);
1429:   }
1430: 
1431:   // If the CondV is the output of a SETCC node which operates on GRLenVT
1432:   // inputs, then merge the SETCC node into the lowered LoongArchISD::SELECT_CC
1433:   // to take advantage of the integer compare+branch instructions. i.e.: (select
1434:   // (setcc lhs, rhs, cc), truev, falsev)
1435:   // -> (loongarchisd::select_cc lhs, rhs, cc, truev, falsev)
1436:   SDValue LHS = CondV.getOperand(0);
1437:   SDValue RHS = CondV.getOperand(1);
1438:   ISD::CondCode CCVal = cast<CondCodeSDNode>(CondV.getOperand(2))->get();
1439: 
1440:   // Special case for a select of 2 constants that have a difference of 1.
1441:   // Normally this is done by DAGCombine, but if the select is introduced by
1442:   // type legalization or op legalization, we miss it. Restricting to SETLT
1443:   // case for now because that is what signed saturating add/sub need.
1444:   // FIXME: We don't need the condition to be SETLT or even a SETCC,
1445:   // but we would probably want to swap the true/false values if the condition
1446:   // is SETGE/SETLE to avoid an XORI.
1447:   if (isa<ConstantSDNode>(TrueV) && isa<ConstantSDNode>(FalseV) &&
1448:       CCVal == ISD::SETLT) {
1449:     const APInt &TrueVal = TrueV->getAsAPIntVal();
1450:     const APInt &FalseVal = FalseV->getAsAPIntVal();
1451:     if (TrueVal - 1 == FalseVal)
1452:       return DAG.getNode(ISD::ADD, DL, VT, CondV, FalseV);
1453:     if (TrueVal + 1 == FalseVal)
1454:       return DAG.getNode(ISD::SUB, DL, VT, FalseV, CondV);
1455:   }
1456: 
1457:   translateSetCCForBranch(DL, LHS, RHS, CCVal, DAG);
1458:   // 1 < x ? x : 1 -> 0 < x ? x : 1
1459:   if (isOneConstant(LHS) && (CCVal == ISD::SETLT || CCVal == ISD::SETULT) &&
1460:       RHS == TrueV && LHS == FalseV) {
1461:     LHS = DAG.getConstant(0, DL, VT);
1462:     // 0 <u x is the same as x != 0.
1463:     if (CCVal == ISD::SETULT) {
1464:       std::swap(LHS, RHS);
1465:       CCVal = ISD::SETNE;
1466:     }
1467:   }
1468: 
1469:   // x <s -1 ? x : -1 -> x <s 0 ? x : -1
1470:   if (isAllOnesConstant(RHS) && CCVal == ISD::SETLT && LHS == TrueV &&
1471:       RHS == FalseV) {
1472:     RHS = DAG.getConstant(0, DL, VT);
1473:   }
1474: 
1475:   SDValue TargetCC = DAG.getCondCode(CCVal);
1476: 
1477:   if (isa<ConstantSDNode>(TrueV) && !isa<ConstantSDNode>(FalseV)) {
1478:     // (select (setcc lhs, rhs, CC), constant, falsev)
1479:     // -> (select (setcc lhs, rhs, InverseCC), falsev, constant)
1480:     std::swap(TrueV, FalseV);
1481:     TargetCC = DAG.getCondCode(ISD::getSetCCInverse(CCVal, LHS.getValueType()));
1482:   }
1483: 
1484:   SDValue Ops[] = {LHS, RHS, TargetCC, TrueV, FalseV};
1485:   return DAG.getNode(LoongArchISD::SELECT_CC, DL, VT, Ops);
1486: }
1487: 
1488: SDValue LoongArchTargetLowering::lowerBRCOND(SDValue Op,
1489:                                              SelectionDAG &DAG) const {
1490:   SDValue CondV = Op.getOperand(1);
1491:   SDLoc DL(Op);
1492:   MVT GRLenVT = Subtarget.getGRLenVT();
1493: 
1494:   if (CondV.getOpcode() == ISD::SETCC) {
1495:     if (CondV.getOperand(0).getValueType() == GRLenVT) {
1496:       SDValue LHS = CondV.getOperand(0);
1497:       SDValue RHS = CondV.getOperand(1);
1498:       ISD::CondCode CCVal = cast<CondCodeSDNode>(CondV.getOperand(2))->get();
1499: 
1500:       translateSetCCForBranch(DL, LHS, RHS, CCVal, DAG);
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerBRCOND`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerBRCOND` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1501-1575 / 第 1501-1575 行
```cpp
1501: 
1502:       SDValue TargetCC = DAG.getCondCode(CCVal);
1503:       return DAG.getNode(LoongArchISD::BR_CC, DL, Op.getValueType(),
1504:                          Op.getOperand(0), LHS, RHS, TargetCC,
1505:                          Op.getOperand(2));
1506:     } else if (CondV.getOperand(0).getValueType().isFloatingPoint()) {
1507:       return DAG.getNode(LoongArchISD::BRCOND, DL, Op.getValueType(),
1508:                          Op.getOperand(0), CondV, Op.getOperand(2));
1509:     }
1510:   }
1511: 
1512:   return DAG.getNode(LoongArchISD::BR_CC, DL, Op.getValueType(),
1513:                      Op.getOperand(0), CondV, DAG.getConstant(0, DL, GRLenVT),
1514:                      DAG.getCondCode(ISD::SETNE), Op.getOperand(2));
1515: }
1516: 
1517: SDValue
1518: LoongArchTargetLowering::lowerSCALAR_TO_VECTOR(SDValue Op,
1519:                                                SelectionDAG &DAG) const {
1520:   SDLoc DL(Op);
1521:   MVT OpVT = Op.getSimpleValueType();
1522: 
1523:   SDValue Vector = DAG.getUNDEF(OpVT);
1524:   SDValue Val = Op.getOperand(0);
1525:   SDValue Idx = DAG.getConstant(0, DL, Subtarget.getGRLenVT());
1526: 
1527:   return DAG.getNode(ISD::INSERT_VECTOR_ELT, DL, OpVT, Vector, Val, Idx);
1528: }
1529: 
1530: SDValue LoongArchTargetLowering::lowerBITREVERSE(SDValue Op,
1531:                                                  SelectionDAG &DAG) const {
1532:   EVT ResTy = Op->getValueType(0);
1533:   SDValue Src = Op->getOperand(0);
1534:   SDLoc DL(Op);
1535: 
1536:   // LoongArchISD::BITREV_8B is not supported on LA32.
1537:   if (!Subtarget.is64Bit() && (ResTy == MVT::v16i8 || ResTy == MVT::v32i8))
1538:     return SDValue();
1539: 
1540:   EVT NewVT = ResTy.is128BitVector() ? MVT::v2i64 : MVT::v4i64;
1541:   unsigned int OrigEltNum = ResTy.getVectorNumElements();
1542:   unsigned int NewEltNum = NewVT.getVectorNumElements();
1543: 
1544:   SDValue NewSrc = DAG.getNode(ISD::BITCAST, DL, NewVT, Src);
1545: 
1546:   SmallVector<SDValue, 8> Ops;
1547:   for (unsigned int i = 0; i < NewEltNum; i++) {
1548:     SDValue Op = DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, MVT::i64, NewSrc,
1549:                              DAG.getConstant(i, DL, Subtarget.getGRLenVT()));
1550:     unsigned RevOp = (ResTy == MVT::v16i8 || ResTy == MVT::v32i8)
1551:                          ? (unsigned)LoongArchISD::BITREV_8B
1552:                          : (unsigned)ISD::BITREVERSE;
1553:     Ops.push_back(DAG.getNode(RevOp, DL, MVT::i64, Op));
1554:   }
1555:   SDValue Res =
1556:       DAG.getNode(ISD::BITCAST, DL, ResTy, DAG.getBuildVector(NewVT, DL, Ops));
1557: 
1558:   switch (ResTy.getSimpleVT().SimpleTy) {
1559:   default:
1560:     return SDValue();
1561:   case MVT::v16i8:
1562:   case MVT::v32i8:
1563:     return Res;
1564:   case MVT::v8i16:
1565:   case MVT::v16i16:
1566:   case MVT::v4i32:
1567:   case MVT::v8i32: {
1568:     SmallVector<int, 32> Mask;
1569:     for (unsigned int i = 0; i < NewEltNum; i++)
1570:       for (int j = OrigEltNum / NewEltNum - 1; j >= 0; j--)
1571:         Mask.push_back(j + (OrigEltNum / NewEltNum) * i);
1572:     return DAG.getVectorShuffle(ResTy, DL, Res, DAG.getUNDEF(ResTy), Mask);
1573:   }
1574:   }
1575: }
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerSCALAR_TO_VECTOR`, `LoongArchTargetLowering::lowerBITREVERSE`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerSCALAR_TO_VECTOR`, `LoongArchTargetLowering::lowerBITREVERSE` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1576-1650 / 第 1576-1650 行
```cpp
1576: 
1577: // Widen element type to get a new mask value (if possible).
1578: // For example:
1579: //  shufflevector <4 x i32> %a, <4 x i32> %b,
1580: //                <4 x i32> <i32 6, i32 7, i32 2, i32 3>
1581: // is equivalent to:
1582: //  shufflevector <2 x i64> %a, <2 x i64> %b, <2 x i32> <i32 3, i32 1>
1583: // can be lowered to:
1584: //  VPACKOD_D vr0, vr0, vr1
1585: static SDValue widenShuffleMask(const SDLoc &DL, ArrayRef<int> Mask, MVT VT,
1586:                                 SDValue V1, SDValue V2, SelectionDAG &DAG) {
1587:   unsigned EltBits = VT.getScalarSizeInBits();
1588: 
1589:   if (EltBits > 32 || EltBits == 1)
1590:     return SDValue();
1591: 
1592:   SmallVector<int, 8> NewMask;
1593:   if (widenShuffleMaskElts(Mask, NewMask)) {
1594:     MVT NewEltVT = VT.isFloatingPoint() ? MVT::getFloatingPointVT(EltBits * 2)
1595:                                         : MVT::getIntegerVT(EltBits * 2);
1596:     MVT NewVT = MVT::getVectorVT(NewEltVT, VT.getVectorNumElements() / 2);
1597:     if (DAG.getTargetLoweringInfo().isTypeLegal(NewVT)) {
1598:       SDValue NewV1 = DAG.getBitcast(NewVT, V1);
1599:       SDValue NewV2 = DAG.getBitcast(NewVT, V2);
1600:       return DAG.getBitcast(
1601:           VT, DAG.getVectorShuffle(NewVT, DL, NewV1, NewV2, NewMask));
1602:     }
1603:   }
1604: 
1605:   return SDValue();
1606: }
1607: 
1608: /// Attempts to match a shuffle mask against the VBSLL, VBSRL, VSLLI and VSRLI
1609: /// instruction.
1610: // The funciton matches elements from one of the input vector shuffled to the
1611: // left or right with zeroable elements 'shifted in'. It handles both the
1612: // strictly bit-wise element shifts and the byte shfit across an entire 128-bit
1613: // lane.
1614: // Mostly copied from X86.
1615: static int matchShuffleAsShift(MVT &ShiftVT, unsigned &Opcode,
1616:                                unsigned ScalarSizeInBits, ArrayRef<int> Mask,
1617:                                int MaskOffset, const APInt &Zeroable) {
1618:   int Size = Mask.size();
1619:   unsigned SizeInBits = Size * ScalarSizeInBits;
1620: 
1621:   auto CheckZeros = [&](int Shift, int Scale, bool Left) {
1622:     for (int i = 0; i < Size; i += Scale)
1623:       for (int j = 0; j < Shift; ++j)
1624:         if (!Zeroable[i + j + (Left ? 0 : (Scale - Shift))])
1625:           return false;
1626: 
1627:     return true;
1628:   };
1629: 
1630:   auto isSequentialOrUndefInRange = [&](unsigned Pos, unsigned Size, int Low,
1631:                                         int Step = 1) {
1632:     for (unsigned i = Pos, e = Pos + Size; i != e; ++i, Low += Step)
1633:       if (!(Mask[i] == -1 || Mask[i] == Low))
1634:         return false;
1635:     return true;
1636:   };
1637: 
1638:   auto MatchShift = [&](int Shift, int Scale, bool Left) {
1639:     for (int i = 0; i != Size; i += Scale) {
1640:       unsigned Pos = Left ? i + Shift : i;
1641:       unsigned Low = Left ? i : i + Shift;
1642:       unsigned Len = Scale - Shift;
1643:       if (!isSequentialOrUndefInRange(Pos, Len, Low + MaskOffset))
1644:         return -1;
1645:     }
1646: 
1647:     int ShiftEltBits = ScalarSizeInBits * Scale;
1648:     bool ByteShift = ShiftEltBits > 64;
1649:     Opcode = Left ? (ByteShift ? LoongArchISD::VBSLL : LoongArchISD::VSLLI)
1650:                   : (ByteShift ? LoongArchISD::VBSRL : LoongArchISD::VSRLI);
```
- **EN**: The range implements or declares functions including `widenShuffleMask`, `matchShuffleAsShift`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `widenShuffleMask`, `matchShuffleAsShift` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1651-1725 / 第 1651-1725 行
```cpp
1651:     int ShiftAmt = Shift * ScalarSizeInBits / (ByteShift ? 8 : 1);
1652: 
1653:     // Normalize the scale for byte shifts to still produce an i64 element
1654:     // type.
1655:     Scale = ByteShift ? Scale / 2 : Scale;
1656: 
1657:     // We need to round trip through the appropriate type for the shift.
1658:     MVT ShiftSVT = MVT::getIntegerVT(ScalarSizeInBits * Scale);
1659:     ShiftVT = ByteShift ? MVT::getVectorVT(MVT::i8, SizeInBits / 8)
1660:                         : MVT::getVectorVT(ShiftSVT, Size / Scale);
1661:     return (int)ShiftAmt;
1662:   };
1663: 
1664:   unsigned MaxWidth = 128;
1665:   for (int Scale = 2; Scale * ScalarSizeInBits <= MaxWidth; Scale *= 2)
1666:     for (int Shift = 1; Shift != Scale; ++Shift)
1667:       for (bool Left : {true, false})
1668:         if (CheckZeros(Shift, Scale, Left)) {
1669:           int ShiftAmt = MatchShift(Shift, Scale, Left);
1670:           if (0 < ShiftAmt)
1671:             return ShiftAmt;
1672:         }
1673: 
1674:   // no match
1675:   return -1;
1676: }
1677: 
1678: /// Lower VECTOR_SHUFFLE as shift (if possible).
1679: ///
1680: /// For example:
1681: ///   %2 = shufflevector <4 x i32> %0, <4 x i32> zeroinitializer,
1682: ///                      <4 x i32> <i32 4, i32 0, i32 1, i32 2>
1683: /// is lowered to:
1684: ///     (VBSLL_V $v0, $v0, 4)
1685: ///
1686: ///   %2 = shufflevector <4 x i32> %0, <4 x i32> zeroinitializer,
1687: ///                      <4 x i32> <i32 4, i32 0, i32 4, i32 2>
1688: /// is lowered to:
1689: ///     (VSLLI_D $v0, $v0, 32)
1690: static SDValue lowerVECTOR_SHUFFLEAsShift(const SDLoc &DL, ArrayRef<int> Mask,
1691:                                           MVT VT, SDValue V1, SDValue V2,
1692:                                           SelectionDAG &DAG,
1693:                                           const LoongArchSubtarget &Subtarget,
1694:                                           const APInt &Zeroable) {
1695:   int Size = Mask.size();
1696:   assert(Size == (int)VT.getVectorNumElements() && "Unexpected mask size");
1697: 
1698:   MVT ShiftVT;
1699:   SDValue V = V1;
1700:   unsigned Opcode;
1701: 
1702:   // Try to match shuffle against V1 shift.
1703:   int ShiftAmt = matchShuffleAsShift(ShiftVT, Opcode, VT.getScalarSizeInBits(),
1704:                                      Mask, 0, Zeroable);
1705: 
1706:   // If V1 failed, try to match shuffle against V2 shift.
1707:   if (ShiftAmt < 0) {
1708:     ShiftAmt = matchShuffleAsShift(ShiftVT, Opcode, VT.getScalarSizeInBits(),
1709:                                    Mask, Size, Zeroable);
1710:     V = V2;
1711:   }
1712: 
1713:   if (ShiftAmt < 0)
1714:     return SDValue();
1715: 
1716:   assert(DAG.getTargetLoweringInfo().isTypeLegal(ShiftVT) &&
1717:          "Illegal integer vector type");
1718:   V = DAG.getBitcast(ShiftVT, V);
1719:   V = DAG.getNode(Opcode, DL, ShiftVT, V,
1720:                   DAG.getConstant(ShiftAmt, DL, Subtarget.getGRLenVT()));
1721:   return DAG.getBitcast(VT, V);
1722: }
1723: 
1724: /// Determine whether a range fits a regular pattern of values.
1725: /// This function accounts for the possibility of jumping over the End iterator.
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLEAsShift`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLEAsShift` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1726-1800 / 第 1726-1800 行
```cpp
1726: template <typename ValType>
1727: static bool
1728: fitsRegularPattern(typename SmallVectorImpl<ValType>::const_iterator Begin,
1729:                    unsigned CheckStride,
1730:                    typename SmallVectorImpl<ValType>::const_iterator End,
1731:                    ValType ExpectedIndex, unsigned ExpectedIndexStride) {
1732:   auto &I = Begin;
1733: 
1734:   while (I != End) {
1735:     if (*I != -1 && *I != ExpectedIndex)
1736:       return false;
1737:     ExpectedIndex += ExpectedIndexStride;
1738: 
1739:     // Incrementing past End is undefined behaviour so we must increment one
1740:     // step at a time and check for End at each step.
1741:     for (unsigned n = 0; n < CheckStride && I != End; ++n, ++I)
1742:       ; // Empty loop body.
1743:   }
1744:   return true;
1745: }
1746: 
1747: /// Compute whether each element of a shuffle is zeroable.
1748: ///
1749: /// A "zeroable" vector shuffle element is one which can be lowered to zero.
1750: static void computeZeroableShuffleElements(ArrayRef<int> Mask, SDValue V1,
1751:                                            SDValue V2, APInt &KnownUndef,
1752:                                            APInt &KnownZero) {
1753:   int Size = Mask.size();
1754:   KnownUndef = KnownZero = APInt::getZero(Size);
1755: 
1756:   V1 = peekThroughBitcasts(V1);
1757:   V2 = peekThroughBitcasts(V2);
1758: 
1759:   bool V1IsZero = ISD::isBuildVectorAllZeros(V1.getNode());
1760:   bool V2IsZero = ISD::isBuildVectorAllZeros(V2.getNode());
1761: 
1762:   int VectorSizeInBits = V1.getValueSizeInBits();
1763:   int ScalarSizeInBits = VectorSizeInBits / Size;
1764:   assert(!(VectorSizeInBits % ScalarSizeInBits) && "Illegal shuffle mask size");
1765:   (void)ScalarSizeInBits;
1766: 
1767:   for (int i = 0; i < Size; ++i) {
1768:     int M = Mask[i];
1769:     if (M < 0) {
1770:       KnownUndef.setBit(i);
1771:       continue;
1772:     }
1773:     if ((M >= 0 && M < Size && V1IsZero) || (M >= Size && V2IsZero)) {
1774:       KnownZero.setBit(i);
1775:       continue;
1776:     }
1777:   }
1778: }
1779: 
1780: /// Test whether a shuffle mask is equivalent within each sub-lane.
1781: ///
1782: /// The specific repeated shuffle mask is populated in \p RepeatedMask, as it is
1783: /// non-trivial to compute in the face of undef lanes. The representation is
1784: /// suitable for use with existing 128-bit shuffles as entries from the second
1785: /// vector have been remapped to [LaneSize, 2*LaneSize).
1786: static bool isRepeatedShuffleMask(unsigned LaneSizeInBits, MVT VT,
1787:                                   ArrayRef<int> Mask,
1788:                                   SmallVectorImpl<int> &RepeatedMask) {
1789:   auto LaneSize = LaneSizeInBits / VT.getScalarSizeInBits();
1790:   RepeatedMask.assign(LaneSize, -1);
1791:   int Size = Mask.size();
1792:   for (int i = 0; i < Size; ++i) {
1793:     assert(Mask[i] == -1 || Mask[i] >= 0);
1794:     if (Mask[i] < 0)
1795:       continue;
1796:     if ((Mask[i] % Size) / LaneSize != i / LaneSize)
1797:       // This entry crosses lanes, so there is no way to model this shuffle.
1798:       return false;
1799: 
1800:     // Ok, handle the in-lane shuffles by detecting if and when they repeat.
```
- **EN**: The range implements or declares functions including `fitsRegularPattern`, `computeZeroableShuffleElements`, `isRepeatedShuffleMask`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `fitsRegularPattern`, `computeZeroableShuffleElements`, `isRepeatedShuffleMask` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1801-1875 / 第 1801-1875 行
```cpp
1801:     // Adjust second vector indices to start at LaneSize instead of Size.
1802:     int LocalM =
1803:         Mask[i] < Size ? Mask[i] % LaneSize : Mask[i] % LaneSize + LaneSize;
1804:     if (RepeatedMask[i % LaneSize] < 0)
1805:       // This is the first non-undef entry in this slot of a 128-bit lane.
1806:       RepeatedMask[i % LaneSize] = LocalM;
1807:     else if (RepeatedMask[i % LaneSize] != LocalM)
1808:       // Found a mismatch with the repeated mask.
1809:       return false;
1810:   }
1811:   return true;
1812: }
1813: 
1814: /// Attempts to match vector shuffle as byte rotation.
1815: static int matchShuffleAsByteRotate(MVT VT, SDValue &V1, SDValue &V2,
1816:                                     ArrayRef<int> Mask) {
1817: 
1818:   SDValue Lo, Hi;
1819:   SmallVector<int, 16> RepeatedMask;
1820: 
1821:   if (!isRepeatedShuffleMask(128, VT, Mask, RepeatedMask))
1822:     return -1;
1823: 
1824:   int NumElts = RepeatedMask.size();
1825:   int Rotation = 0;
1826:   int Scale = 16 / NumElts;
1827: 
1828:   for (int i = 0; i < NumElts; ++i) {
1829:     int M = RepeatedMask[i];
1830:     assert((M == -1 || (0 <= M && M < (2 * NumElts))) &&
1831:            "Unexpected mask index.");
1832:     if (M < 0)
1833:       continue;
1834: 
1835:     // Determine where a rotated vector would have started.
1836:     int StartIdx = i - (M % NumElts);
1837:     if (StartIdx == 0)
1838:       return -1;
1839: 
1840:     // If we found the tail of a vector the rotation must be the missing
1841:     // front. If we found the head of a vector, it must be how much of the
1842:     // head.
1843:     int CandidateRotation = StartIdx < 0 ? -StartIdx : NumElts - StartIdx;
1844: 
1845:     if (Rotation == 0)
1846:       Rotation = CandidateRotation;
1847:     else if (Rotation != CandidateRotation)
1848:       return -1;
1849: 
1850:     // Compute which value this mask is pointing at.
1851:     SDValue MaskV = M < NumElts ? V1 : V2;
1852: 
1853:     // Compute which of the two target values this index should be assigned
1854:     // to. This reflects whether the high elements are remaining or the low
1855:     // elements are remaining.
1856:     SDValue &TargetV = StartIdx < 0 ? Hi : Lo;
1857: 
1858:     // Either set up this value if we've not encountered it before, or check
1859:     // that it remains consistent.
1860:     if (!TargetV)
1861:       TargetV = MaskV;
1862:     else if (TargetV != MaskV)
1863:       return -1;
1864:   }
1865: 
1866:   // Check that we successfully analyzed the mask, and normalize the results.
1867:   assert(Rotation != 0 && "Failed to locate a viable rotation!");
1868:   assert((Lo || Hi) && "Failed to find a rotated input vector!");
1869:   if (!Lo)
1870:     Lo = Hi;
1871:   else if (!Hi)
1872:     Hi = Lo;
1873: 
1874:   V1 = Lo;
1875:   V2 = Hi;
```
- **EN**: The range implements or declares functions including `matchShuffleAsByteRotate`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `matchShuffleAsByteRotate` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1876-1950 / 第 1876-1950 行
```cpp
1876: 
1877:   return Rotation * Scale;
1878: }
1879: 
1880: /// Lower VECTOR_SHUFFLE as byte rotate (if possible).
1881: ///
1882: /// For example:
1883: ///   %shuffle = shufflevector <2 x i64> %a, <2 x i64> %b,
1884: ///                            <2 x i32> <i32 3, i32 0>
1885: /// is lowered to:
1886: ///      (VBSRL_V $v1, $v1, 8)
1887: ///      (VBSLL_V $v0, $v0, 8)
1888: ///      (VOR_V $v0, $V0, $v1)
1889: static SDValue
1890: lowerVECTOR_SHUFFLEAsByteRotate(const SDLoc &DL, ArrayRef<int> Mask, MVT VT,
1891:                                 SDValue V1, SDValue V2, SelectionDAG &DAG,
1892:                                 const LoongArchSubtarget &Subtarget) {
1893: 
1894:   SDValue Lo = V1, Hi = V2;
1895:   int ByteRotation = matchShuffleAsByteRotate(VT, Lo, Hi, Mask);
1896:   if (ByteRotation <= 0)
1897:     return SDValue();
1898: 
1899:   MVT ByteVT = MVT::getVectorVT(MVT::i8, VT.getSizeInBits() / 8);
1900:   Lo = DAG.getBitcast(ByteVT, Lo);
1901:   Hi = DAG.getBitcast(ByteVT, Hi);
1902: 
1903:   int LoByteShift = 16 - ByteRotation;
1904:   int HiByteShift = ByteRotation;
1905:   MVT GRLenVT = Subtarget.getGRLenVT();
1906: 
1907:   SDValue LoShift = DAG.getNode(LoongArchISD::VBSLL, DL, ByteVT, Lo,
1908:                                 DAG.getConstant(LoByteShift, DL, GRLenVT));
1909:   SDValue HiShift = DAG.getNode(LoongArchISD::VBSRL, DL, ByteVT, Hi,
1910:                                 DAG.getConstant(HiByteShift, DL, GRLenVT));
1911:   return DAG.getBitcast(VT, DAG.getNode(ISD::OR, DL, ByteVT, LoShift, HiShift));
1912: }
1913: 
1914: /// Lower VECTOR_SHUFFLE as ZERO_EXTEND Or ANY_EXTEND (if possible).
1915: ///
1916: /// For example:
1917: ///   %2 = shufflevector <4 x i32> %0, <4 x i32> zeroinitializer,
1918: ///                      <4 x i32> <i32 0, i32 4, i32 1, i32 4>
1919: ///   %3 = bitcast <4 x i32> %2 to <2 x i64>
1920: /// is lowered to:
1921: ///     (VREPLI $v1, 0)
1922: ///     (VILVL $v0, $v1, $v0)
1923: static SDValue lowerVECTOR_SHUFFLEAsZeroOrAnyExtend(const SDLoc &DL,
1924:                                                     ArrayRef<int> Mask, MVT VT,
1925:                                                     SDValue V1, SDValue V2,
1926:                                                     SelectionDAG &DAG,
1927:                                                     const APInt &Zeroable) {
1928:   int Bits = VT.getSizeInBits();
1929:   int EltBits = VT.getScalarSizeInBits();
1930:   int NumElements = VT.getVectorNumElements();
1931: 
1932:   if (Zeroable.isAllOnes())
1933:     return DAG.getConstant(0, DL, VT);
1934: 
1935:   // Define a helper function to check a particular ext-scale and lower to it if
1936:   // valid.
1937:   auto Lower = [&](int Scale) -> SDValue {
1938:     SDValue InputV;
1939:     bool AnyExt = true;
1940:     int Offset = 0;
1941:     for (int i = 0; i < NumElements; i++) {
1942:       int M = Mask[i];
1943:       if (M < 0)
1944:         continue;
1945:       if (i % Scale != 0) {
1946:         // Each of the extended elements need to be zeroable.
1947:         if (!Zeroable[i])
1948:           return SDValue();
1949: 
1950:         AnyExt = false;
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLEAsByteRotate`, `lowerVECTOR_SHUFFLEAsZeroOrAnyExtend`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLEAsByteRotate`, `lowerVECTOR_SHUFFLEAsZeroOrAnyExtend` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1951-2025 / 第 1951-2025 行
```cpp
1951:         continue;
1952:       }
1953: 
1954:       // Each of the base elements needs to be consecutive indices into the
1955:       // same input vector.
1956:       SDValue V = M < NumElements ? V1 : V2;
1957:       M = M % NumElements;
1958:       if (!InputV) {
1959:         InputV = V;
1960:         Offset = M - (i / Scale);
1961: 
1962:         // These offset can't be handled
1963:         if (Offset % (NumElements / Scale))
1964:           return SDValue();
1965:       } else if (InputV != V)
1966:         return SDValue();
1967: 
1968:       if (M != (Offset + (i / Scale)))
1969:         return SDValue(); // Non-consecutive strided elements.
1970:     }
1971: 
1972:     // If we fail to find an input, we have a zero-shuffle which should always
1973:     // have already been handled.
1974:     if (!InputV)
1975:       return SDValue();
1976: 
1977:     do {
1978:       unsigned VilVLoHi = LoongArchISD::VILVL;
1979:       if (Offset >= (NumElements / 2)) {
1980:         VilVLoHi = LoongArchISD::VILVH;
1981:         Offset -= (NumElements / 2);
1982:       }
1983: 
1984:       MVT InputVT = MVT::getVectorVT(MVT::getIntegerVT(EltBits), NumElements);
1985:       SDValue Ext =
1986:           AnyExt ? DAG.getFreeze(InputV) : DAG.getConstant(0, DL, InputVT);
1987:       InputV = DAG.getBitcast(InputVT, InputV);
1988:       InputV = DAG.getNode(VilVLoHi, DL, InputVT, Ext, InputV);
1989:       Scale /= 2;
1990:       EltBits *= 2;
1991:       NumElements /= 2;
1992:     } while (Scale > 1);
1993:     return DAG.getBitcast(VT, InputV);
1994:   };
1995: 
1996:   // Each iteration, try extending the elements half as much, but into twice as
1997:   // many elements.
1998:   for (int NumExtElements = Bits / 64; NumExtElements < NumElements;
1999:        NumExtElements *= 2) {
2000:     if (SDValue V = Lower(NumElements / NumExtElements))
2001:       return V;
2002:   }
2003:   return SDValue();
2004: }
2005: 
2006: /// Lower VECTOR_SHUFFLE into VREPLVEI (if possible).
2007: ///
2008: /// VREPLVEI performs vector broadcast based on an element specified by an
2009: /// integer immediate, with its mask being similar to:
2010: ///   <x, x, x, ...>
2011: /// where x is any valid index.
2012: ///
2013: /// When undef's appear in the mask they are treated as if they were whatever
2014: /// value is necessary in order to fit the above form.
2015: static SDValue
2016: lowerVECTOR_SHUFFLE_VREPLVEI(const SDLoc &DL, ArrayRef<int> Mask, MVT VT,
2017:                              SDValue V1, SelectionDAG &DAG,
2018:                              const LoongArchSubtarget &Subtarget) {
2019:   int SplatIndex = -1;
2020:   for (const auto &M : Mask) {
2021:     if (M != -1) {
2022:       SplatIndex = M;
2023:       break;
2024:     }
2025:   }
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLE_VREPLVEI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLE_VREPLVEI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2026-2100 / 第 2026-2100 行
```cpp
2026: 
2027:   if (SplatIndex == -1)
2028:     return DAG.getUNDEF(VT);
2029: 
2030:   assert(SplatIndex < (int)Mask.size() && "Out of bounds mask index");
2031:   if (fitsRegularPattern<int>(Mask.begin(), 1, Mask.end(), SplatIndex, 0)) {
2032:     return DAG.getNode(LoongArchISD::VREPLVEI, DL, VT, V1,
2033:                        DAG.getConstant(SplatIndex, DL, Subtarget.getGRLenVT()));
2034:   }
2035: 
2036:   return SDValue();
2037: }
2038: 
2039: /// Lower VECTOR_SHUFFLE into VSHUF4I (if possible).
2040: ///
2041: /// VSHUF4I splits the vector into blocks of four elements, then shuffles these
2042: /// elements according to a <4 x i2> constant (encoded as an integer immediate).
2043: ///
2044: /// It is therefore possible to lower into VSHUF4I when the mask takes the form:
2045: ///   <a, b, c, d, a+4, b+4, c+4, d+4, a+8, b+8, c+8, d+8, ...>
2046: /// When undef's appear they are treated as if they were whatever value is
2047: /// necessary in order to fit the above forms.
2048: ///
2049: /// For example:
2050: ///   %2 = shufflevector <8 x i16> %0, <8 x i16> undef,
2051: ///                      <8 x i32> <i32 3, i32 2, i32 1, i32 0,
2052: ///                                 i32 7, i32 6, i32 5, i32 4>
2053: /// is lowered to:
2054: ///   (VSHUF4I_H $v0, $v1, 27)
2055: /// where the 27 comes from:
2056: ///   3 + (2 << 2) + (1 << 4) + (0 << 6)
2057: static SDValue
2058: lowerVECTOR_SHUFFLE_VSHUF4I(const SDLoc &DL, ArrayRef<int> Mask, MVT VT,
2059:                             SDValue V1, SDValue V2, SelectionDAG &DAG,
2060:                             const LoongArchSubtarget &Subtarget) {
2061: 
2062:   unsigned SubVecSize = 4;
2063:   if (VT == MVT::v2f64 || VT == MVT::v2i64)
2064:     SubVecSize = 2;
2065: 
2066:   int SubMask[4] = {-1, -1, -1, -1};
2067:   for (unsigned i = 0; i < SubVecSize; ++i) {
2068:     for (unsigned j = i; j < Mask.size(); j += SubVecSize) {
2069:       int M = Mask[j];
2070: 
2071:       // Convert from vector index to 4-element subvector index
2072:       // If an index refers to an element outside of the subvector then give up
2073:       if (M != -1) {
2074:         M -= 4 * (j / SubVecSize);
2075:         if (M < 0 || M >= 4)
2076:           return SDValue();
2077:       }
2078: 
2079:       // If the mask has an undef, replace it with the current index.
2080:       // Note that it might still be undef if the current index is also undef
2081:       if (SubMask[i] == -1)
2082:         SubMask[i] = M;
2083:       // Check that non-undef values are the same as in the mask. If they
2084:       // aren't then give up
2085:       else if (M != -1 && M != SubMask[i])
2086:         return SDValue();
2087:     }
2088:   }
2089: 
2090:   // Calculate the immediate. Replace any remaining undefs with zero
2091:   int Imm = 0;
2092:   for (int i = SubVecSize - 1; i >= 0; --i) {
2093:     int M = SubMask[i];
2094: 
2095:     if (M == -1)
2096:       M = 0;
2097: 
2098:     Imm <<= 2;
2099:     Imm |= M & 0x3;
2100:   }
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLE_VSHUF4I`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLE_VSHUF4I` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2101-2175 / 第 2101-2175 行
```cpp
2101: 
2102:   MVT GRLenVT = Subtarget.getGRLenVT();
2103: 
2104:   // Return vshuf4i.d
2105:   if (VT == MVT::v2f64 || VT == MVT::v2i64)
2106:     return DAG.getNode(LoongArchISD::VSHUF4I_D, DL, VT, V1, V2,
2107:                        DAG.getConstant(Imm, DL, GRLenVT));
2108: 
2109:   return DAG.getNode(LoongArchISD::VSHUF4I, DL, VT, V1,
2110:                      DAG.getConstant(Imm, DL, GRLenVT));
2111: }
2112: 
2113: /// Lower VECTOR_SHUFFLE whose result is the reversed source vector.
2114: ///
2115: /// It is possible to do optimization for VECTOR_SHUFFLE performing vector
2116: /// reverse whose mask likes:
2117: ///   <7, 6, 5, 4, 3, 2, 1, 0>
2118: ///
2119: /// When undef's appear in the mask they are treated as if they were whatever
2120: /// value is necessary in order to fit the above forms.
2121: static SDValue
2122: lowerVECTOR_SHUFFLE_IsReverse(const SDLoc &DL, ArrayRef<int> Mask, MVT VT,
2123:                               SDValue V1, SelectionDAG &DAG,
2124:                               const LoongArchSubtarget &Subtarget) {
2125:   // Only vectors with i8/i16 elements which cannot match other patterns
2126:   // directly needs to do this.
2127:   if (VT != MVT::v16i8 && VT != MVT::v8i16 && VT != MVT::v32i8 &&
2128:       VT != MVT::v16i16)
2129:     return SDValue();
2130: 
2131:   if (!ShuffleVectorInst::isReverseMask(Mask, Mask.size()))
2132:     return SDValue();
2133: 
2134:   int WidenNumElts = VT.getVectorNumElements() / 4;
2135:   SmallVector<int, 16> WidenMask(WidenNumElts, -1);
2136:   for (int i = 0; i < WidenNumElts; ++i)
2137:     WidenMask[i] = WidenNumElts - 1 - i;
2138: 
2139:   MVT WidenVT = MVT::getVectorVT(
2140:       VT.getVectorElementType() == MVT::i8 ? MVT::i32 : MVT::i64, WidenNumElts);
2141:   SDValue NewV1 = DAG.getBitcast(WidenVT, V1);
2142:   SDValue WidenRev = DAG.getVectorShuffle(WidenVT, DL, NewV1,
2143:                                           DAG.getUNDEF(WidenVT), WidenMask);
2144: 
2145:   return DAG.getNode(LoongArchISD::VSHUF4I, DL, VT,
2146:                      DAG.getBitcast(VT, WidenRev),
2147:                      DAG.getConstant(27, DL, Subtarget.getGRLenVT()));
2148: }
2149: 
2150: /// Lower VECTOR_SHUFFLE into VPACKEV (if possible).
2151: ///
2152: /// VPACKEV interleaves the even elements from each vector.
2153: ///
2154: /// It is possible to lower into VPACKEV when the mask consists of two of the
2155: /// following forms interleaved:
2156: ///   <0, 2, 4, ...>
2157: ///   <n, n+2, n+4, ...>
2158: /// where n is the number of elements in the vector.
2159: /// For example:
2160: ///   <0, 0, 2, 2, 4, 4, ...>
2161: ///   <0, n, 2, n+2, 4, n+4, ...>
2162: ///
2163: /// When undef's appear in the mask they are treated as if they were whatever
2164: /// value is necessary in order to fit the above forms.
2165: static SDValue lowerVECTOR_SHUFFLE_VPACKEV(const SDLoc &DL, ArrayRef<int> Mask,
2166:                                            MVT VT, SDValue V1, SDValue V2,
2167:                                            SelectionDAG &DAG) {
2168: 
2169:   const auto &Begin = Mask.begin();
2170:   const auto &End = Mask.end();
2171:   SDValue OriV1 = V1, OriV2 = V2;
2172: 
2173:   if (fitsRegularPattern<int>(Begin, 2, End, 0, 2))
2174:     V1 = OriV1;
2175:   else if (fitsRegularPattern<int>(Begin, 2, End, Mask.size(), 2))
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLE_IsReverse`, `lowerVECTOR_SHUFFLE_VPACKEV`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLE_IsReverse`, `lowerVECTOR_SHUFFLE_VPACKEV` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2176-2250 / 第 2176-2250 行
```cpp
2176:     V1 = OriV2;
2177:   else
2178:     return SDValue();
2179: 
2180:   if (fitsRegularPattern<int>(Begin + 1, 2, End, 0, 2))
2181:     V2 = OriV1;
2182:   else if (fitsRegularPattern<int>(Begin + 1, 2, End, Mask.size(), 2))
2183:     V2 = OriV2;
2184:   else
2185:     return SDValue();
2186: 
2187:   return DAG.getNode(LoongArchISD::VPACKEV, DL, VT, V2, V1);
2188: }
2189: 
2190: /// Lower VECTOR_SHUFFLE into VPACKOD (if possible).
2191: ///
2192: /// VPACKOD interleaves the odd elements from each vector.
2193: ///
2194: /// It is possible to lower into VPACKOD when the mask consists of two of the
2195: /// following forms interleaved:
2196: ///   <1, 3, 5, ...>
2197: ///   <n+1, n+3, n+5, ...>
2198: /// where n is the number of elements in the vector.
2199: /// For example:
2200: ///   <1, 1, 3, 3, 5, 5, ...>
2201: ///   <1, n+1, 3, n+3, 5, n+5, ...>
2202: ///
2203: /// When undef's appear in the mask they are treated as if they were whatever
2204: /// value is necessary in order to fit the above forms.
2205: static SDValue lowerVECTOR_SHUFFLE_VPACKOD(const SDLoc &DL, ArrayRef<int> Mask,
2206:                                            MVT VT, SDValue V1, SDValue V2,
2207:                                            SelectionDAG &DAG) {
2208: 
2209:   const auto &Begin = Mask.begin();
2210:   const auto &End = Mask.end();
2211:   SDValue OriV1 = V1, OriV2 = V2;
2212: 
2213:   if (fitsRegularPattern<int>(Begin, 2, End, 1, 2))
2214:     V1 = OriV1;
2215:   else if (fitsRegularPattern<int>(Begin, 2, End, Mask.size() + 1, 2))
2216:     V1 = OriV2;
2217:   else
2218:     return SDValue();
2219: 
2220:   if (fitsRegularPattern<int>(Begin + 1, 2, End, 1, 2))
2221:     V2 = OriV1;
2222:   else if (fitsRegularPattern<int>(Begin + 1, 2, End, Mask.size() + 1, 2))
2223:     V2 = OriV2;
2224:   else
2225:     return SDValue();
2226: 
2227:   return DAG.getNode(LoongArchISD::VPACKOD, DL, VT, V2, V1);
2228: }
2229: 
2230: /// Lower VECTOR_SHUFFLE into VILVH (if possible).
2231: ///
2232: /// VILVH interleaves consecutive elements from the left (highest-indexed) half
2233: /// of each vector.
2234: ///
2235: /// It is possible to lower into VILVH when the mask consists of two of the
2236: /// following forms interleaved:
2237: ///   <x, x+1, x+2, ...>
2238: ///   <n+x, n+x+1, n+x+2, ...>
2239: /// where n is the number of elements in the vector and x is half n.
2240: /// For example:
2241: ///   <x, x, x+1, x+1, x+2, x+2, ...>
2242: ///   <x, n+x, x+1, n+x+1, x+2, n+x+2, ...>
2243: ///
2244: /// When undef's appear in the mask they are treated as if they were whatever
2245: /// value is necessary in order to fit the above forms.
2246: static SDValue lowerVECTOR_SHUFFLE_VILVH(const SDLoc &DL, ArrayRef<int> Mask,
2247:                                          MVT VT, SDValue V1, SDValue V2,
2248:                                          SelectionDAG &DAG) {
2249: 
2250:   const auto &Begin = Mask.begin();
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLE_VPACKOD`, `lowerVECTOR_SHUFFLE_VILVH`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLE_VPACKOD`, `lowerVECTOR_SHUFFLE_VILVH` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2251-2325 / 第 2251-2325 行
```cpp
2251:   const auto &End = Mask.end();
2252:   unsigned HalfSize = Mask.size() / 2;
2253:   SDValue OriV1 = V1, OriV2 = V2;
2254: 
2255:   if (fitsRegularPattern<int>(Begin, 2, End, HalfSize, 1))
2256:     V1 = OriV1;
2257:   else if (fitsRegularPattern<int>(Begin, 2, End, Mask.size() + HalfSize, 1))
2258:     V1 = OriV2;
2259:   else
2260:     return SDValue();
2261: 
2262:   if (fitsRegularPattern<int>(Begin + 1, 2, End, HalfSize, 1))
2263:     V2 = OriV1;
2264:   else if (fitsRegularPattern<int>(Begin + 1, 2, End, Mask.size() + HalfSize,
2265:                                    1))
2266:     V2 = OriV2;
2267:   else
2268:     return SDValue();
2269: 
2270:   return DAG.getNode(LoongArchISD::VILVH, DL, VT, V2, V1);
2271: }
2272: 
2273: /// Lower VECTOR_SHUFFLE into VILVL (if possible).
2274: ///
2275: /// VILVL interleaves consecutive elements from the right (lowest-indexed) half
2276: /// of each vector.
2277: ///
2278: /// It is possible to lower into VILVL when the mask consists of two of the
2279: /// following forms interleaved:
2280: ///   <0, 1, 2, ...>
2281: ///   <n, n+1, n+2, ...>
2282: /// where n is the number of elements in the vector.
2283: /// For example:
2284: ///   <0, 0, 1, 1, 2, 2, ...>
2285: ///   <0, n, 1, n+1, 2, n+2, ...>
2286: ///
2287: /// When undef's appear in the mask they are treated as if they were whatever
2288: /// value is necessary in order to fit the above forms.
2289: static SDValue lowerVECTOR_SHUFFLE_VILVL(const SDLoc &DL, ArrayRef<int> Mask,
2290:                                          MVT VT, SDValue V1, SDValue V2,
2291:                                          SelectionDAG &DAG) {
2292: 
2293:   const auto &Begin = Mask.begin();
2294:   const auto &End = Mask.end();
2295:   SDValue OriV1 = V1, OriV2 = V2;
2296: 
2297:   if (fitsRegularPattern<int>(Begin, 2, End, 0, 1))
2298:     V1 = OriV1;
2299:   else if (fitsRegularPattern<int>(Begin, 2, End, Mask.size(), 1))
2300:     V1 = OriV2;
2301:   else
2302:     return SDValue();
2303: 
2304:   if (fitsRegularPattern<int>(Begin + 1, 2, End, 0, 1))
2305:     V2 = OriV1;
2306:   else if (fitsRegularPattern<int>(Begin + 1, 2, End, Mask.size(), 1))
2307:     V2 = OriV2;
2308:   else
2309:     return SDValue();
2310: 
2311:   return DAG.getNode(LoongArchISD::VILVL, DL, VT, V2, V1);
2312: }
2313: 
2314: /// Lower VECTOR_SHUFFLE into VPICKEV (if possible).
2315: ///
2316: /// VPICKEV copies the even elements of each vector into the result vector.
2317: ///
2318: /// It is possible to lower into VPICKEV when the mask consists of two of the
2319: /// following forms concatenated:
2320: ///   <0, 2, 4, ...>
2321: ///   <n, n+2, n+4, ...>
2322: /// where n is the number of elements in the vector.
2323: /// For example:
2324: ///   <0, 2, 4, ..., 0, 2, 4, ...>
2325: ///   <0, 2, 4, ..., n, n+2, n+4, ...>
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLE_VILVL`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLE_VILVL` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2326-2400 / 第 2326-2400 行
```cpp
2326: ///
2327: /// When undef's appear in the mask they are treated as if they were whatever
2328: /// value is necessary in order to fit the above forms.
2329: static SDValue lowerVECTOR_SHUFFLE_VPICKEV(const SDLoc &DL, ArrayRef<int> Mask,
2330:                                            MVT VT, SDValue V1, SDValue V2,
2331:                                            SelectionDAG &DAG) {
2332: 
2333:   const auto &Begin = Mask.begin();
2334:   const auto &Mid = Mask.begin() + Mask.size() / 2;
2335:   const auto &End = Mask.end();
2336:   SDValue OriV1 = V1, OriV2 = V2;
2337: 
2338:   if (fitsRegularPattern<int>(Begin, 1, Mid, 0, 2))
2339:     V1 = OriV1;
2340:   else if (fitsRegularPattern<int>(Begin, 1, Mid, Mask.size(), 2))
2341:     V1 = OriV2;
2342:   else
2343:     return SDValue();
2344: 
2345:   if (fitsRegularPattern<int>(Mid, 1, End, 0, 2))
2346:     V2 = OriV1;
2347:   else if (fitsRegularPattern<int>(Mid, 1, End, Mask.size(), 2))
2348:     V2 = OriV2;
2349: 
2350:   else
2351:     return SDValue();
2352: 
2353:   return DAG.getNode(LoongArchISD::VPICKEV, DL, VT, V2, V1);
2354: }
2355: 
2356: /// Lower VECTOR_SHUFFLE into VPICKOD (if possible).
2357: ///
2358: /// VPICKOD copies the odd elements of each vector into the result vector.
2359: ///
2360: /// It is possible to lower into VPICKOD when the mask consists of two of the
2361: /// following forms concatenated:
2362: ///   <1, 3, 5, ...>
2363: ///   <n+1, n+3, n+5, ...>
2364: /// where n is the number of elements in the vector.
2365: /// For example:
2366: ///   <1, 3, 5, ..., 1, 3, 5, ...>
2367: ///   <1, 3, 5, ..., n+1, n+3, n+5, ...>
2368: ///
2369: /// When undef's appear in the mask they are treated as if they were whatever
2370: /// value is necessary in order to fit the above forms.
2371: static SDValue lowerVECTOR_SHUFFLE_VPICKOD(const SDLoc &DL, ArrayRef<int> Mask,
2372:                                            MVT VT, SDValue V1, SDValue V2,
2373:                                            SelectionDAG &DAG) {
2374: 
2375:   const auto &Begin = Mask.begin();
2376:   const auto &Mid = Mask.begin() + Mask.size() / 2;
2377:   const auto &End = Mask.end();
2378:   SDValue OriV1 = V1, OriV2 = V2;
2379: 
2380:   if (fitsRegularPattern<int>(Begin, 1, Mid, 1, 2))
2381:     V1 = OriV1;
2382:   else if (fitsRegularPattern<int>(Begin, 1, Mid, Mask.size() + 1, 2))
2383:     V1 = OriV2;
2384:   else
2385:     return SDValue();
2386: 
2387:   if (fitsRegularPattern<int>(Mid, 1, End, 1, 2))
2388:     V2 = OriV1;
2389:   else if (fitsRegularPattern<int>(Mid, 1, End, Mask.size() + 1, 2))
2390:     V2 = OriV2;
2391:   else
2392:     return SDValue();
2393: 
2394:   return DAG.getNode(LoongArchISD::VPICKOD, DL, VT, V2, V1);
2395: }
2396: 
2397: /// Lower VECTOR_SHUFFLE into VEXTRINS (if possible).
2398: ///
2399: /// VEXTRINS copies one element of a vector into any place of the result
2400: /// vector and makes no change to the rest elements of the result vector.
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLE_VPICKEV`, `lowerVECTOR_SHUFFLE_VPICKOD`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLE_VPICKEV`, `lowerVECTOR_SHUFFLE_VPICKOD` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2401-2475 / 第 2401-2475 行
```cpp
2401: ///
2402: /// It is possible to lower into VEXTRINS when the mask takes the form:
2403: ///   <0, 1, 2, ..., n+i, ..., n-1>  or  <n, n+1, n+2, ..., i, ..., 2n-1> or
2404: ///   <0, 1, 2, ..., i, ..., n-1>  or  <n, n+1, n+2, ..., n+i, ..., 2n-1>
2405: /// where n is the number of elements in the vector and i is in [0, n).
2406: /// For example:
2407: ///   <0, 1, 2, 3, 4, 5, 6, 8> , <2, 9, 10, 11, 12, 13, 14, 15> ,
2408: ///   <0, 1, 2, 6, 4, 5, 6, 7> , <8, 9, 10, 11, 12, 9, 14, 15>
2409: ///
2410: /// When undef's appear in the mask they are treated as if they were whatever
2411: /// value is necessary in order to fit the above forms.
2412: static SDValue
2413: lowerVECTOR_SHUFFLE_VEXTRINS(const SDLoc &DL, ArrayRef<int> Mask, MVT VT,
2414:                              SDValue V1, SDValue V2, SelectionDAG &DAG,
2415:                              const LoongArchSubtarget &Subtarget) {
2416:   unsigned NumElts = VT.getVectorNumElements();
2417:   MVT EltVT = VT.getVectorElementType();
2418:   MVT GRLenVT = Subtarget.getGRLenVT();
2419: 
2420:   if (Mask.size() != NumElts)
2421:     return SDValue();
2422: 
2423:   auto tryLowerToExtrAndIns = [&](unsigned Base) -> SDValue {
2424:     int DiffCount = 0;
2425:     int DiffPos = -1;
2426:     for (unsigned i = 0; i < NumElts; ++i) {
2427:       if (Mask[i] == -1)
2428:         continue;
2429:       if (Mask[i] != int(Base + i)) {
2430:         ++DiffCount;
2431:         DiffPos = int(i);
2432:         if (DiffCount > 1)
2433:           return SDValue();
2434:       }
2435:     }
2436: 
2437:     // Need exactly one differing element to lower into VEXTRINS.
2438:     if (DiffCount != 1)
2439:       return SDValue();
2440: 
2441:     // DiffMask must be in [0, 2N).
2442:     int DiffMask = Mask[DiffPos];
2443:     if (DiffMask < 0 || DiffMask >= int(2 * NumElts))
2444:       return SDValue();
2445: 
2446:     // Determine source vector and source index.
2447:     SDValue SrcVec;
2448:     unsigned SrcIdx;
2449:     if (unsigned(DiffMask) < NumElts) {
2450:       SrcVec = V1;
2451:       SrcIdx = unsigned(DiffMask);
2452:     } else {
2453:       SrcVec = V2;
2454:       SrcIdx = unsigned(DiffMask) - NumElts;
2455:     }
2456: 
2457:     // Replace with EXTRACT_VECTOR_ELT + INSERT_VECTOR_ELT, it will match the
2458:     // patterns of VEXTRINS in tablegen.
2459:     SDValue Extracted = DAG.getNode(
2460:         ISD::EXTRACT_VECTOR_ELT, DL, EltVT.isFloatingPoint() ? EltVT : GRLenVT,
2461:         SrcVec, DAG.getConstant(SrcIdx, DL, GRLenVT));
2462:     SDValue Result =
2463:         DAG.getNode(ISD::INSERT_VECTOR_ELT, DL, VT, (Base == 0) ? V1 : V2,
2464:                     Extracted, DAG.getConstant(DiffPos, DL, GRLenVT));
2465: 
2466:     return Result;
2467:   };
2468: 
2469:   // Try [0, n-1) insertion then [n, 2n-1) insertion.
2470:   if (SDValue Result = tryLowerToExtrAndIns(0))
2471:     return Result;
2472:   return tryLowerToExtrAndIns(NumElts);
2473: }
2474: 
2475: // Check the Mask and then build SrcVec and MaskImm infos which will
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLE_VEXTRINS`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLE_VEXTRINS` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2476-2550 / 第 2476-2550 行
```cpp
2476: // be used to build LoongArchISD nodes for VPERMI_W or XVPERMI_W.
2477: // On success, return true. Otherwise, return false.
2478: static bool buildVPERMIInfo(ArrayRef<int> Mask, SDValue V1, SDValue V2,
2479:                             SmallVectorImpl<SDValue> &SrcVec,
2480:                             unsigned &MaskImm) {
2481:   unsigned MaskSize = Mask.size();
2482: 
2483:   auto isValid = [&](int M, int Off) {
2484:     return (M == -1) || (M >= Off && M < Off + 4);
2485:   };
2486: 
2487:   auto buildImm = [&](int MLo, int MHi, unsigned Off, unsigned I) {
2488:     auto immPart = [&](int M, unsigned Off) {
2489:       return (M == -1 ? 0 : (M - Off)) & 0x3;
2490:     };
2491:     MaskImm |= immPart(MLo, Off) << (I * 2);
2492:     MaskImm |= immPart(MHi, Off) << ((I + 1) * 2);
2493:   };
2494: 
2495:   for (unsigned i = 0; i < 4; i += 2) {
2496:     int MLo = Mask[i];
2497:     int MHi = Mask[i + 1];
2498: 
2499:     if (MaskSize == 8) { // Only v8i32/v8f32 need this check.
2500:       int M2Lo = Mask[i + 4];
2501:       int M2Hi = Mask[i + 5];
2502:       if (M2Lo != MLo + 4 || M2Hi != MHi + 4)
2503:         return false;
2504:     }
2505: 
2506:     if (isValid(MLo, 0) && isValid(MHi, 0)) {
2507:       SrcVec.push_back(V1);
2508:       buildImm(MLo, MHi, 0, i);
2509:     } else if (isValid(MLo, MaskSize) && isValid(MHi, MaskSize)) {
2510:       SrcVec.push_back(V2);
2511:       buildImm(MLo, MHi, MaskSize, i);
2512:     } else {
2513:       return false;
2514:     }
2515:   }
2516: 
2517:   return true;
2518: }
2519: 
2520: /// Lower VECTOR_SHUFFLE into VPERMI (if possible).
2521: ///
2522: /// VPERMI selects two elements from each of the two vectors based on the
2523: /// mask and places them in the corresponding positions of the result vector
2524: /// in order. Only v4i32 and v4f32 types are allowed.
2525: ///
2526: /// It is possible to lower into VPERMI when the mask consists of two of the
2527: /// following forms concatenated:
2528: ///   <i, j, u, v>
2529: ///   <u, v, i, j>
2530: /// where i,j are in [0,4) and u,v are in [4, 8).
2531: /// For example:
2532: ///   <2, 3, 4, 5>
2533: ///   <5, 7, 0, 2>
2534: ///
2535: /// When undef's appear in the mask they are treated as if they were whatever
2536: /// value is necessary in order to fit the above forms.
2537: static SDValue lowerVECTOR_SHUFFLE_VPERMI(const SDLoc &DL, ArrayRef<int> Mask,
2538:                                           MVT VT, SDValue V1, SDValue V2,
2539:                                           SelectionDAG &DAG,
2540:                                           const LoongArchSubtarget &Subtarget) {
2541:   if ((VT != MVT::v4i32 && VT != MVT::v4f32) ||
2542:       Mask.size() != VT.getVectorNumElements())
2543:     return SDValue();
2544: 
2545:   SmallVector<SDValue, 2> SrcVec;
2546:   unsigned MaskImm = 0;
2547:   if (!buildVPERMIInfo(Mask, V1, V2, SrcVec, MaskImm))
2548:     return SDValue();
2549: 
2550:   return DAG.getNode(LoongArchISD::VPERMI, DL, VT, SrcVec[1], SrcVec[0],
```
- **EN**: The range implements or declares functions including `buildVPERMIInfo`, `lowerVECTOR_SHUFFLE_VPERMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `buildVPERMIInfo`, `lowerVECTOR_SHUFFLE_VPERMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2551-2625 / 第 2551-2625 行
```cpp
2551:                      DAG.getConstant(MaskImm, DL, Subtarget.getGRLenVT()));
2552: }
2553: 
2554: /// Lower VECTOR_SHUFFLE into VSHUF.
2555: ///
2556: /// This mostly consists of converting the shuffle mask into a BUILD_VECTOR and
2557: /// adding it as an operand to the resulting VSHUF.
2558: static SDValue lowerVECTOR_SHUFFLE_VSHUF(const SDLoc &DL, ArrayRef<int> Mask,
2559:                                          MVT VT, SDValue V1, SDValue V2,
2560:                                          SelectionDAG &DAG,
2561:                                          const LoongArchSubtarget &Subtarget) {
2562: 
2563:   SmallVector<SDValue, 16> Ops;
2564:   for (auto M : Mask)
2565:     Ops.push_back(DAG.getSignedConstant(M, DL, Subtarget.getGRLenVT()));
2566: 
2567:   EVT MaskVecTy = VT.changeVectorElementTypeToInteger();
2568:   SDValue MaskVec = DAG.getBuildVector(MaskVecTy, DL, Ops);
2569: 
2570:   // VECTOR_SHUFFLE concatenates the vectors in an vectorwise fashion.
2571:   // <0b00, 0b01> + <0b10, 0b11> -> <0b00, 0b01, 0b10, 0b11>
2572:   // VSHF concatenates the vectors in a bitwise fashion:
2573:   // <0b00, 0b01> + <0b10, 0b11> ->
2574:   // 0b0100       + 0b1110       -> 0b01001110
2575:   //                                <0b10, 0b11, 0b00, 0b01>
2576:   // We must therefore swap the operands to get the correct result.
2577:   return DAG.getNode(LoongArchISD::VSHUF, DL, VT, MaskVec, V2, V1);
2578: }
2579: 
2580: /// Dispatching routine to lower various 128-bit LoongArch vector shuffles.
2581: ///
2582: /// This routine breaks down the specific type of 128-bit shuffle and
2583: /// dispatches to the lowering routines accordingly.
2584: static SDValue lower128BitShuffle(const SDLoc &DL, ArrayRef<int> Mask, MVT VT,
2585:                                   SDValue V1, SDValue V2, SelectionDAG &DAG,
2586:                                   const LoongArchSubtarget &Subtarget) {
2587:   assert((VT.SimpleTy == MVT::v16i8 || VT.SimpleTy == MVT::v8i16 ||
2588:           VT.SimpleTy == MVT::v4i32 || VT.SimpleTy == MVT::v2i64 ||
2589:           VT.SimpleTy == MVT::v4f32 || VT.SimpleTy == MVT::v2f64) &&
2590:          "Vector type is unsupported for lsx!");
2591:   assert(V1.getSimpleValueType() == V2.getSimpleValueType() &&
2592:          "Two operands have different types!");
2593:   assert(VT.getVectorNumElements() == Mask.size() &&
2594:          "Unexpected mask size for shuffle!");
2595:   assert(Mask.size() % 2 == 0 && "Expected even mask size.");
2596: 
2597:   APInt KnownUndef, KnownZero;
2598:   computeZeroableShuffleElements(Mask, V1, V2, KnownUndef, KnownZero);
2599:   APInt Zeroable = KnownUndef | KnownZero;
2600: 
2601:   SDValue Result;
2602:   // TODO: Add more comparison patterns.
2603:   if (V2.isUndef()) {
2604:     if ((Result =
2605:              lowerVECTOR_SHUFFLE_VREPLVEI(DL, Mask, VT, V1, DAG, Subtarget)))
2606:       return Result;
2607:     if ((Result =
2608:              lowerVECTOR_SHUFFLE_VSHUF4I(DL, Mask, VT, V1, V2, DAG, Subtarget)))
2609:       return Result;
2610:     if ((Result =
2611:              lowerVECTOR_SHUFFLE_IsReverse(DL, Mask, VT, V1, DAG, Subtarget)))
2612:       return Result;
2613: 
2614:     // TODO: This comment may be enabled in the future to better match the
2615:     // pattern for instruction selection.
2616:     /* V2 = V1; */
2617:   }
2618: 
2619:   // It is recommended not to change the pattern comparison order for better
2620:   // performance.
2621:   if ((Result = lowerVECTOR_SHUFFLE_VPACKEV(DL, Mask, VT, V1, V2, DAG)))
2622:     return Result;
2623:   if ((Result = lowerVECTOR_SHUFFLE_VPACKOD(DL, Mask, VT, V1, V2, DAG)))
2624:     return Result;
2625:   if ((Result = lowerVECTOR_SHUFFLE_VILVH(DL, Mask, VT, V1, V2, DAG)))
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLE_VSHUF`, `lower128BitShuffle`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLE_VSHUF`, `lower128BitShuffle` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2626-2700 / 第 2626-2700 行
```cpp
2626:     return Result;
2627:   if ((Result = lowerVECTOR_SHUFFLE_VILVL(DL, Mask, VT, V1, V2, DAG)))
2628:     return Result;
2629:   if ((Result = lowerVECTOR_SHUFFLE_VPICKEV(DL, Mask, VT, V1, V2, DAG)))
2630:     return Result;
2631:   if ((Result = lowerVECTOR_SHUFFLE_VPICKOD(DL, Mask, VT, V1, V2, DAG)))
2632:     return Result;
2633:   if ((VT.SimpleTy == MVT::v2i64 || VT.SimpleTy == MVT::v2f64) &&
2634:       (Result =
2635:            lowerVECTOR_SHUFFLE_VSHUF4I(DL, Mask, VT, V1, V2, DAG, Subtarget)))
2636:     return Result;
2637:   if ((Result =
2638:            lowerVECTOR_SHUFFLE_VEXTRINS(DL, Mask, VT, V1, V2, DAG, Subtarget)))
2639:     return Result;
2640:   if ((Result = lowerVECTOR_SHUFFLEAsShift(DL, Mask, VT, V1, V2, DAG, Subtarget,
2641:                                            Zeroable)))
2642:     return Result;
2643:   if ((Result =
2644:            lowerVECTOR_SHUFFLE_VPERMI(DL, Mask, VT, V1, V2, DAG, Subtarget)))
2645:     return Result;
2646:   if ((Result = lowerVECTOR_SHUFFLEAsZeroOrAnyExtend(DL, Mask, VT, V1, V2, DAG,
2647:                                                      Zeroable)))
2648:     return Result;
2649:   if ((Result = lowerVECTOR_SHUFFLEAsByteRotate(DL, Mask, VT, V1, V2, DAG,
2650:                                                 Subtarget)))
2651:     return Result;
2652:   if (SDValue NewShuffle = widenShuffleMask(DL, Mask, VT, V1, V2, DAG))
2653:     return NewShuffle;
2654:   if ((Result =
2655:            lowerVECTOR_SHUFFLE_VSHUF(DL, Mask, VT, V1, V2, DAG, Subtarget)))
2656:     return Result;
2657:   return SDValue();
2658: }
2659: 
2660: /// Lower VECTOR_SHUFFLE into XVREPLVEI (if possible).
2661: ///
2662: /// It is a XVREPLVEI when the mask is:
2663: ///   <x, x, x, ..., x+n, x+n, x+n, ...>
2664: /// where the number of x is equal to n and n is half the length of vector.
2665: ///
2666: /// When undef's appear in the mask they are treated as if they were whatever
2667: /// value is necessary in order to fit the above form.
2668: static SDValue
2669: lowerVECTOR_SHUFFLE_XVREPLVEI(const SDLoc &DL, ArrayRef<int> Mask, MVT VT,
2670:                               SDValue V1, SelectionDAG &DAG,
2671:                               const LoongArchSubtarget &Subtarget) {
2672:   int SplatIndex = -1;
2673:   for (const auto &M : Mask) {
2674:     if (M != -1) {
2675:       SplatIndex = M;
2676:       break;
2677:     }
2678:   }
2679: 
2680:   if (SplatIndex == -1)
2681:     return DAG.getUNDEF(VT);
2682: 
2683:   const auto &Begin = Mask.begin();
2684:   const auto &End = Mask.end();
2685:   int HalfSize = Mask.size() / 2;
2686: 
2687:   if (SplatIndex >= HalfSize)
2688:     return SDValue();
2689: 
2690:   assert(SplatIndex < (int)Mask.size() && "Out of bounds mask index");
2691:   if (fitsRegularPattern<int>(Begin, 1, End - HalfSize, SplatIndex, 0) &&
2692:       fitsRegularPattern<int>(Begin + HalfSize, 1, End, SplatIndex + HalfSize,
2693:                               0)) {
2694:     return DAG.getNode(LoongArchISD::VREPLVEI, DL, VT, V1,
2695:                        DAG.getConstant(SplatIndex, DL, Subtarget.getGRLenVT()));
2696:   }
2697: 
2698:   return SDValue();
2699: }
2700: 
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLE_XVREPLVEI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLE_XVREPLVEI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2701-2775 / 第 2701-2775 行
```cpp
2701: /// Lower VECTOR_SHUFFLE into XVSHUF4I (if possible).
2702: static SDValue
2703: lowerVECTOR_SHUFFLE_XVSHUF4I(const SDLoc &DL, ArrayRef<int> Mask, MVT VT,
2704:                              SDValue V1, SDValue V2, SelectionDAG &DAG,
2705:                              const LoongArchSubtarget &Subtarget) {
2706:   // XVSHUF4I_D must be handled separately because it is different from other
2707:   // types of [X]VSHUF4I instructions.
2708:   if (Mask.size() == 4) {
2709:     unsigned MaskImm = 0;
2710:     for (int i = 1; i >= 0; --i) {
2711:       int MLo = Mask[i];
2712:       int MHi = Mask[i + 2];
2713:       if (!(MLo == -1 || (MLo >= 0 && MLo <= 1) || (MLo >= 4 && MLo <= 5)) ||
2714:           !(MHi == -1 || (MHi >= 2 && MHi <= 3) || (MHi >= 6 && MHi <= 7)))
2715:         return SDValue();
2716:       if (MHi != -1 && MLo != -1 && MHi != MLo + 2)
2717:         return SDValue();
2718: 
2719:       MaskImm <<= 2;
2720:       if (MLo != -1)
2721:         MaskImm |= ((MLo <= 1) ? MLo : (MLo - 2)) & 0x3;
2722:       else if (MHi != -1)
2723:         MaskImm |= ((MHi <= 3) ? (MHi - 2) : (MHi - 4)) & 0x3;
2724:     }
2725: 
2726:     return DAG.getNode(LoongArchISD::VSHUF4I_D, DL, VT, V1, V2,
2727:                        DAG.getConstant(MaskImm, DL, Subtarget.getGRLenVT()));
2728:   }
2729: 
2730:   return lowerVECTOR_SHUFFLE_VSHUF4I(DL, Mask, VT, V1, V2, DAG, Subtarget);
2731: }
2732: 
2733: /// Lower VECTOR_SHUFFLE into XVPERMI (if possible).
2734: static SDValue
2735: lowerVECTOR_SHUFFLE_XVPERMI(const SDLoc &DL, ArrayRef<int> Mask, MVT VT,
2736:                             SDValue V1, SDValue V2, SelectionDAG &DAG,
2737:                             const LoongArchSubtarget &Subtarget) {
2738:   MVT GRLenVT = Subtarget.getGRLenVT();
2739:   unsigned MaskSize = Mask.size();
2740:   if (MaskSize != VT.getVectorNumElements())
2741:     return SDValue();
2742: 
2743:   // Consider XVPERMI_W.
2744:   if (VT == MVT::v8i32 || VT == MVT::v8f32) {
2745:     SmallVector<SDValue, 2> SrcVec;
2746:     unsigned MaskImm = 0;
2747:     if (!buildVPERMIInfo(Mask, V1, V2, SrcVec, MaskImm))
2748:       return SDValue();
2749: 
2750:     return DAG.getNode(LoongArchISD::VPERMI, DL, VT, SrcVec[1], SrcVec[0],
2751:                        DAG.getConstant(MaskImm, DL, GRLenVT));
2752:   }
2753: 
2754:   // Consider XVPERMI_D.
2755:   if (VT == MVT::v4i64 || VT == MVT::v4f64) {
2756:     unsigned MaskImm = 0;
2757:     for (unsigned i = 0; i < MaskSize; ++i) {
2758:       if (Mask[i] == -1)
2759:         continue;
2760:       if (Mask[i] >= (int)MaskSize)
2761:         return SDValue();
2762:       MaskImm |= Mask[i] << (i * 2);
2763:     }
2764: 
2765:     return DAG.getNode(LoongArchISD::XVPERMI, DL, VT, V1,
2766:                        DAG.getConstant(MaskImm, DL, GRLenVT));
2767:   }
2768: 
2769:   return SDValue();
2770: }
2771: 
2772: /// Lower VECTOR_SHUFFLE into XVPERM (if possible).
2773: static SDValue lowerVECTOR_SHUFFLE_XVPERM(const SDLoc &DL, ArrayRef<int> Mask,
2774:                                           MVT VT, SDValue V1, SelectionDAG &DAG,
2775:                                           const LoongArchSubtarget &Subtarget) {
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLE_XVSHUF4I`, `lowerVECTOR_SHUFFLE_XVPERMI`, `lowerVECTOR_SHUFFLE_XVPERM`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLE_XVSHUF4I`, `lowerVECTOR_SHUFFLE_XVPERMI`, `lowerVECTOR_SHUFFLE_XVPERM` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2776-2850 / 第 2776-2850 行
```cpp
2776:   // LoongArch LASX only have XVPERM_W.
2777:   if (Mask.size() != 8 || (VT != MVT::v8i32 && VT != MVT::v8f32))
2778:     return SDValue();
2779: 
2780:   unsigned NumElts = VT.getVectorNumElements();
2781:   unsigned HalfSize = NumElts / 2;
2782:   bool FrontLo = true, FrontHi = true;
2783:   bool BackLo = true, BackHi = true;
2784: 
2785:   auto inRange = [](int val, int low, int high) {
2786:     return (val == -1) || (val >= low && val < high);
2787:   };
2788: 
2789:   for (unsigned i = 0; i < HalfSize; ++i) {
2790:     int Fronti = Mask[i];
2791:     int Backi = Mask[i + HalfSize];
2792: 
2793:     FrontLo &= inRange(Fronti, 0, HalfSize);
2794:     FrontHi &= inRange(Fronti, HalfSize, NumElts);
2795:     BackLo &= inRange(Backi, 0, HalfSize);
2796:     BackHi &= inRange(Backi, HalfSize, NumElts);
2797:   }
2798: 
2799:   // If both the lower and upper 128-bit parts access only one half of the
2800:   // vector (either lower or upper), avoid using xvperm.w. The latency of
2801:   // xvperm.w(3) is higher than using xvshuf(1) and xvori(1).
2802:   if ((FrontLo || FrontHi) && (BackLo || BackHi))
2803:     return SDValue();
2804: 
2805:   SmallVector<SDValue, 8> Masks;
2806:   MVT GRLenVT = Subtarget.getGRLenVT();
2807:   for (unsigned i = 0; i < NumElts; ++i)
2808:     Masks.push_back(Mask[i] == -1 ? DAG.getUNDEF(GRLenVT)
2809:                                   : DAG.getConstant(Mask[i], DL, GRLenVT));
2810:   SDValue MaskVec = DAG.getBuildVector(MVT::v8i32, DL, Masks);
2811: 
2812:   return DAG.getNode(LoongArchISD::XVPERM, DL, VT, V1, MaskVec);
2813: }
2814: 
2815: /// Lower VECTOR_SHUFFLE into XVPACKEV (if possible).
2816: static SDValue lowerVECTOR_SHUFFLE_XVPACKEV(const SDLoc &DL, ArrayRef<int> Mask,
2817:                                             MVT VT, SDValue V1, SDValue V2,
2818:                                             SelectionDAG &DAG) {
2819:   return lowerVECTOR_SHUFFLE_VPACKEV(DL, Mask, VT, V1, V2, DAG);
2820: }
2821: 
2822: /// Lower VECTOR_SHUFFLE into XVPACKOD (if possible).
2823: static SDValue lowerVECTOR_SHUFFLE_XVPACKOD(const SDLoc &DL, ArrayRef<int> Mask,
2824:                                             MVT VT, SDValue V1, SDValue V2,
2825:                                             SelectionDAG &DAG) {
2826:   return lowerVECTOR_SHUFFLE_VPACKOD(DL, Mask, VT, V1, V2, DAG);
2827: }
2828: 
2829: /// Lower VECTOR_SHUFFLE into XVILVH (if possible).
2830: static SDValue lowerVECTOR_SHUFFLE_XVILVH(const SDLoc &DL, ArrayRef<int> Mask,
2831:                                           MVT VT, SDValue V1, SDValue V2,
2832:                                           SelectionDAG &DAG) {
2833: 
2834:   const auto &Begin = Mask.begin();
2835:   const auto &End = Mask.end();
2836:   unsigned HalfSize = Mask.size() / 2;
2837:   unsigned LeftSize = HalfSize / 2;
2838:   SDValue OriV1 = V1, OriV2 = V2;
2839: 
2840:   if (fitsRegularPattern<int>(Begin, 2, End - HalfSize, HalfSize - LeftSize,
2841:                               1) &&
2842:       fitsRegularPattern<int>(Begin + HalfSize, 2, End, HalfSize + LeftSize, 1))
2843:     V1 = OriV1;
2844:   else if (fitsRegularPattern<int>(Begin, 2, End - HalfSize,
2845:                                    Mask.size() + HalfSize - LeftSize, 1) &&
2846:            fitsRegularPattern<int>(Begin + HalfSize, 2, End,
2847:                                    Mask.size() + HalfSize + LeftSize, 1))
2848:     V1 = OriV2;
2849:   else
2850:     return SDValue();
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLE_XVPACKEV`, `lowerVECTOR_SHUFFLE_XVPACKOD`, `lowerVECTOR_SHUFFLE_XVILVH`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLE_XVPACKEV`, `lowerVECTOR_SHUFFLE_XVPACKOD`, `lowerVECTOR_SHUFFLE_XVILVH` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2851-2925 / 第 2851-2925 行
```cpp
2851: 
2852:   if (fitsRegularPattern<int>(Begin + 1, 2, End - HalfSize, HalfSize - LeftSize,
2853:                               1) &&
2854:       fitsRegularPattern<int>(Begin + 1 + HalfSize, 2, End, HalfSize + LeftSize,
2855:                               1))
2856:     V2 = OriV1;
2857:   else if (fitsRegularPattern<int>(Begin + 1, 2, End - HalfSize,
2858:                                    Mask.size() + HalfSize - LeftSize, 1) &&
2859:            fitsRegularPattern<int>(Begin + 1 + HalfSize, 2, End,
2860:                                    Mask.size() + HalfSize + LeftSize, 1))
2861:     V2 = OriV2;
2862:   else
2863:     return SDValue();
2864: 
2865:   return DAG.getNode(LoongArchISD::VILVH, DL, VT, V2, V1);
2866: }
2867: 
2868: /// Lower VECTOR_SHUFFLE into XVILVL (if possible).
2869: static SDValue lowerVECTOR_SHUFFLE_XVILVL(const SDLoc &DL, ArrayRef<int> Mask,
2870:                                           MVT VT, SDValue V1, SDValue V2,
2871:                                           SelectionDAG &DAG) {
2872: 
2873:   const auto &Begin = Mask.begin();
2874:   const auto &End = Mask.end();
2875:   unsigned HalfSize = Mask.size() / 2;
2876:   SDValue OriV1 = V1, OriV2 = V2;
2877: 
2878:   if (fitsRegularPattern<int>(Begin, 2, End - HalfSize, 0, 1) &&
2879:       fitsRegularPattern<int>(Begin + HalfSize, 2, End, HalfSize, 1))
2880:     V1 = OriV1;
2881:   else if (fitsRegularPattern<int>(Begin, 2, End - HalfSize, Mask.size(), 1) &&
2882:            fitsRegularPattern<int>(Begin + HalfSize, 2, End,
2883:                                    Mask.size() + HalfSize, 1))
2884:     V1 = OriV2;
2885:   else
2886:     return SDValue();
2887: 
2888:   if (fitsRegularPattern<int>(Begin + 1, 2, End - HalfSize, 0, 1) &&
2889:       fitsRegularPattern<int>(Begin + 1 + HalfSize, 2, End, HalfSize, 1))
2890:     V2 = OriV1;
2891:   else if (fitsRegularPattern<int>(Begin + 1, 2, End - HalfSize, Mask.size(),
2892:                                    1) &&
2893:            fitsRegularPattern<int>(Begin + 1 + HalfSize, 2, End,
2894:                                    Mask.size() + HalfSize, 1))
2895:     V2 = OriV2;
2896:   else
2897:     return SDValue();
2898: 
2899:   return DAG.getNode(LoongArchISD::VILVL, DL, VT, V2, V1);
2900: }
2901: 
2902: /// Lower VECTOR_SHUFFLE into XVPICKEV (if possible).
2903: static SDValue lowerVECTOR_SHUFFLE_XVPICKEV(const SDLoc &DL, ArrayRef<int> Mask,
2904:                                             MVT VT, SDValue V1, SDValue V2,
2905:                                             SelectionDAG &DAG) {
2906: 
2907:   const auto &Begin = Mask.begin();
2908:   const auto &LeftMid = Mask.begin() + Mask.size() / 4;
2909:   const auto &Mid = Mask.begin() + Mask.size() / 2;
2910:   const auto &RightMid = Mask.end() - Mask.size() / 4;
2911:   const auto &End = Mask.end();
2912:   unsigned HalfSize = Mask.size() / 2;
2913:   SDValue OriV1 = V1, OriV2 = V2;
2914: 
2915:   if (fitsRegularPattern<int>(Begin, 1, LeftMid, 0, 2) &&
2916:       fitsRegularPattern<int>(Mid, 1, RightMid, HalfSize, 2))
2917:     V1 = OriV1;
2918:   else if (fitsRegularPattern<int>(Begin, 1, LeftMid, Mask.size(), 2) &&
2919:            fitsRegularPattern<int>(Mid, 1, RightMid, Mask.size() + HalfSize, 2))
2920:     V1 = OriV2;
2921:   else
2922:     return SDValue();
2923: 
2924:   if (fitsRegularPattern<int>(LeftMid, 1, Mid, 0, 2) &&
2925:       fitsRegularPattern<int>(RightMid, 1, End, HalfSize, 2))
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLE_XVILVL`, `lowerVECTOR_SHUFFLE_XVPICKEV`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLE_XVILVL`, `lowerVECTOR_SHUFFLE_XVPICKEV` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2926-3000 / 第 2926-3000 行
```cpp
2926:     V2 = OriV1;
2927:   else if (fitsRegularPattern<int>(LeftMid, 1, Mid, Mask.size(), 2) &&
2928:            fitsRegularPattern<int>(RightMid, 1, End, Mask.size() + HalfSize, 2))
2929:     V2 = OriV2;
2930: 
2931:   else
2932:     return SDValue();
2933: 
2934:   return DAG.getNode(LoongArchISD::VPICKEV, DL, VT, V2, V1);
2935: }
2936: 
2937: /// Lower VECTOR_SHUFFLE into XVPICKOD (if possible).
2938: static SDValue lowerVECTOR_SHUFFLE_XVPICKOD(const SDLoc &DL, ArrayRef<int> Mask,
2939:                                             MVT VT, SDValue V1, SDValue V2,
2940:                                             SelectionDAG &DAG) {
2941: 
2942:   const auto &Begin = Mask.begin();
2943:   const auto &LeftMid = Mask.begin() + Mask.size() / 4;
2944:   const auto &Mid = Mask.begin() + Mask.size() / 2;
2945:   const auto &RightMid = Mask.end() - Mask.size() / 4;
2946:   const auto &End = Mask.end();
2947:   unsigned HalfSize = Mask.size() / 2;
2948:   SDValue OriV1 = V1, OriV2 = V2;
2949: 
2950:   if (fitsRegularPattern<int>(Begin, 1, LeftMid, 1, 2) &&
2951:       fitsRegularPattern<int>(Mid, 1, RightMid, HalfSize + 1, 2))
2952:     V1 = OriV1;
2953:   else if (fitsRegularPattern<int>(Begin, 1, LeftMid, Mask.size() + 1, 2) &&
2954:            fitsRegularPattern<int>(Mid, 1, RightMid, Mask.size() + HalfSize + 1,
2955:                                    2))
2956:     V1 = OriV2;
2957:   else
2958:     return SDValue();
2959: 
2960:   if (fitsRegularPattern<int>(LeftMid, 1, Mid, 1, 2) &&
2961:       fitsRegularPattern<int>(RightMid, 1, End, HalfSize + 1, 2))
2962:     V2 = OriV1;
2963:   else if (fitsRegularPattern<int>(LeftMid, 1, Mid, Mask.size() + 1, 2) &&
2964:            fitsRegularPattern<int>(RightMid, 1, End, Mask.size() + HalfSize + 1,
2965:                                    2))
2966:     V2 = OriV2;
2967:   else
2968:     return SDValue();
2969: 
2970:   return DAG.getNode(LoongArchISD::VPICKOD, DL, VT, V2, V1);
2971: }
2972: 
2973: /// Lower VECTOR_SHUFFLE into XVEXTRINS (if possible).
2974: static SDValue
2975: lowerVECTOR_SHUFFLE_XVEXTRINS(const SDLoc &DL, ArrayRef<int> Mask, MVT VT,
2976:                               SDValue V1, SDValue V2, SelectionDAG &DAG,
2977:                               const LoongArchSubtarget &Subtarget) {
2978:   int NumElts = VT.getVectorNumElements();
2979:   int HalfSize = NumElts / 2;
2980:   MVT EltVT = VT.getVectorElementType();
2981:   MVT GRLenVT = Subtarget.getGRLenVT();
2982: 
2983:   if ((int)Mask.size() != NumElts)
2984:     return SDValue();
2985: 
2986:   auto tryLowerToExtrAndIns = [&](int Base) -> SDValue {
2987:     SmallVector<int> DiffPos;
2988:     for (int i = 0; i < NumElts; ++i) {
2989:       if (Mask[i] == -1)
2990:         continue;
2991:       if (Mask[i] != Base + i) {
2992:         DiffPos.push_back(i);
2993:         if (DiffPos.size() > 2)
2994:           return SDValue();
2995:       }
2996:     }
2997: 
2998:     // Need exactly two differing element to lower into XVEXTRINS.
2999:     // If only one differing element, the element at a distance of
3000:     // HalfSize from it must be undef.
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLE_XVPICKOD`, `lowerVECTOR_SHUFFLE_XVEXTRINS`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLE_XVPICKOD`, `lowerVECTOR_SHUFFLE_XVEXTRINS` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 3001-3075 / 第 3001-3075 行
```cpp
3001:     if (DiffPos.size() == 1) {
3002:       if (DiffPos[0] < HalfSize && Mask[DiffPos[0] + HalfSize] == -1)
3003:         DiffPos.push_back(DiffPos[0] + HalfSize);
3004:       else if (DiffPos[0] >= HalfSize && Mask[DiffPos[0] - HalfSize] == -1)
3005:         DiffPos.insert(DiffPos.begin(), DiffPos[0] - HalfSize);
3006:       else
3007:         return SDValue();
3008:     }
3009:     if (DiffPos.size() != 2 || DiffPos[1] != DiffPos[0] + HalfSize)
3010:       return SDValue();
3011: 
3012:     // DiffMask must be in its low or high part.
3013:     int DiffMaskLo = Mask[DiffPos[0]];
3014:     int DiffMaskHi = Mask[DiffPos[1]];
3015:     DiffMaskLo = DiffMaskLo == -1 ? DiffMaskHi - HalfSize : DiffMaskLo;
3016:     DiffMaskHi = DiffMaskHi == -1 ? DiffMaskLo + HalfSize : DiffMaskHi;
3017:     if (!(DiffMaskLo >= 0 && DiffMaskLo < HalfSize) &&
3018:         !(DiffMaskLo >= NumElts && DiffMaskLo < NumElts + HalfSize))
3019:       return SDValue();
3020:     if (!(DiffMaskHi >= HalfSize && DiffMaskHi < NumElts) &&
3021:         !(DiffMaskHi >= NumElts + HalfSize && DiffMaskHi < 2 * NumElts))
3022:       return SDValue();
3023:     if (DiffMaskHi != DiffMaskLo + HalfSize)
3024:       return SDValue();
3025: 
3026:     // Determine source vector and source index.
3027:     SDValue SrcVec = (DiffMaskLo < HalfSize) ? V1 : V2;
3028:     int SrcIdxLo =
3029:         (DiffMaskLo < HalfSize) ? DiffMaskLo : (DiffMaskLo - NumElts);
3030:     bool IsEltFP = EltVT.isFloatingPoint();
3031: 
3032:     // Replace with 2*EXTRACT_VECTOR_ELT + 2*INSERT_VECTOR_ELT, it will match
3033:     // the patterns of XVEXTRINS in tablegen.
3034:     SDValue BaseVec = (Base == 0) ? V1 : V2;
3035:     SDValue EltLo =
3036:         DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, IsEltFP ? EltVT : GRLenVT,
3037:                     SrcVec, DAG.getConstant(SrcIdxLo, DL, GRLenVT));
3038:     SDValue InsLo = DAG.getNode(ISD::INSERT_VECTOR_ELT, DL, VT, BaseVec, EltLo,
3039:                                 DAG.getConstant(DiffPos[0], DL, GRLenVT));
3040:     SDValue EltHi =
3041:         DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, IsEltFP ? EltVT : GRLenVT,
3042:                     SrcVec, DAG.getConstant(SrcIdxLo + HalfSize, DL, GRLenVT));
3043:     SDValue Result = DAG.getNode(ISD::INSERT_VECTOR_ELT, DL, VT, InsLo, EltHi,
3044:                                  DAG.getConstant(DiffPos[1], DL, GRLenVT));
3045: 
3046:     return Result;
3047:   };
3048: 
3049:   // Try [0, n-1) insertion then [n, 2n-1) insertion.
3050:   if (SDValue Result = tryLowerToExtrAndIns(0))
3051:     return Result;
3052:   return tryLowerToExtrAndIns(NumElts);
3053: }
3054: 
3055: /// Lower VECTOR_SHUFFLE into XVINSVE0 (if possible).
3056: static SDValue
3057: lowerVECTOR_SHUFFLE_XVINSVE0(const SDLoc &DL, ArrayRef<int> Mask, MVT VT,
3058:                              SDValue V1, SDValue V2, SelectionDAG &DAG,
3059:                              const LoongArchSubtarget &Subtarget) {
3060:   // LoongArch LASX only supports xvinsve0.{w/d}.
3061:   if (VT != MVT::v8i32 && VT != MVT::v8f32 && VT != MVT::v4i64 &&
3062:       VT != MVT::v4f64)
3063:     return SDValue();
3064: 
3065:   MVT GRLenVT = Subtarget.getGRLenVT();
3066:   int MaskSize = Mask.size();
3067:   assert(MaskSize == (int)VT.getVectorNumElements() && "Unexpected mask size");
3068: 
3069:   // Check if exactly one element of the Mask is replaced by 'Replaced', while
3070:   // all other elements are either 'Base + i' or undef (-1). On success, return
3071:   // the index of the replaced element. Otherwise, just return -1.
3072:   auto checkReplaceOne = [&](int Base, int Replaced) -> int {
3073:     int Idx = -1;
3074:     for (int i = 0; i < MaskSize; ++i) {
3075:       if (Mask[i] == Base + i || Mask[i] == -1)
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLE_XVINSVE0`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLE_XVINSVE0` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 3076-3150 / 第 3076-3150 行
```cpp
3076:         continue;
3077:       if (Mask[i] != Replaced)
3078:         return -1;
3079:       if (Idx == -1)
3080:         Idx = i;
3081:       else
3082:         return -1;
3083:     }
3084:     return Idx;
3085:   };
3086: 
3087:   // Case 1: the lowest element of V2 replaces one element in V1.
3088:   int Idx = checkReplaceOne(0, MaskSize);
3089:   if (Idx != -1)
3090:     return DAG.getNode(LoongArchISD::XVINSVE0, DL, VT, V1, V2,
3091:                        DAG.getConstant(Idx, DL, GRLenVT));
3092: 
3093:   // Case 2: the lowest element of V1 replaces one element in V2.
3094:   Idx = checkReplaceOne(MaskSize, 0);
3095:   if (Idx != -1)
3096:     return DAG.getNode(LoongArchISD::XVINSVE0, DL, VT, V2, V1,
3097:                        DAG.getConstant(Idx, DL, GRLenVT));
3098: 
3099:   return SDValue();
3100: }
3101: 
3102: /// Lower VECTOR_SHUFFLE into XVSHUF (if possible).
3103: static SDValue lowerVECTOR_SHUFFLE_XVSHUF(const SDLoc &DL, ArrayRef<int> Mask,
3104:                                           MVT VT, SDValue V1, SDValue V2,
3105:                                           SelectionDAG &DAG) {
3106: 
3107:   int MaskSize = Mask.size();
3108:   int HalfSize = Mask.size() / 2;
3109:   const auto &Begin = Mask.begin();
3110:   const auto &Mid = Mask.begin() + HalfSize;
3111:   const auto &End = Mask.end();
3112: 
3113:   // VECTOR_SHUFFLE concatenates the vectors:
3114:   //  <0, 1, 2, 3, 4, 5, 6, 7> + <8, 9, 10, 11, 12, 13, 14, 15>
3115:   //  shuffling ->
3116:   //  <0, 1, 2, 3, 8, 9, 10, 11> <4, 5, 6, 7, 12, 13, 14, 15>
3117:   //
3118:   // XVSHUF concatenates the vectors:
3119:   //  <a0, a1, a2, a3, b0, b1, b2, b3> + <a4, a5, a6, a7, b4, b5, b6, b7>
3120:   //  shuffling ->
3121:   //  <a0, a1, a2, a3, a4, a5, a6, a7> + <b0, b1, b2, b3, b4, b5, b6, b7>
3122:   SmallVector<SDValue, 8> MaskAlloc;
3123:   for (auto it = Begin; it < Mid; it++) {
3124:     if (*it < 0) // UNDEF
3125:       MaskAlloc.push_back(DAG.getTargetConstant(0, DL, MVT::i64));
3126:     else if ((*it >= 0 && *it < HalfSize) ||
3127:              (*it >= MaskSize && *it < MaskSize + HalfSize)) {
3128:       int M = *it < HalfSize ? *it : *it - HalfSize;
3129:       MaskAlloc.push_back(DAG.getTargetConstant(M, DL, MVT::i64));
3130:     } else
3131:       return SDValue();
3132:   }
3133:   assert((int)MaskAlloc.size() == HalfSize && "xvshuf convert failed!");
3134: 
3135:   for (auto it = Mid; it < End; it++) {
3136:     if (*it < 0) // UNDEF
3137:       MaskAlloc.push_back(DAG.getTargetConstant(0, DL, MVT::i64));
3138:     else if ((*it >= HalfSize && *it < MaskSize) ||
3139:              (*it >= MaskSize + HalfSize && *it < MaskSize * 2)) {
3140:       int M = *it < MaskSize ? *it - HalfSize : *it - MaskSize;
3141:       MaskAlloc.push_back(DAG.getTargetConstant(M, DL, MVT::i64));
3142:     } else
3143:       return SDValue();
3144:   }
3145:   assert((int)MaskAlloc.size() == MaskSize && "xvshuf convert failed!");
3146: 
3147:   EVT MaskVecTy = VT.changeVectorElementTypeToInteger();
3148:   SDValue MaskVec = DAG.getBuildVector(MaskVecTy, DL, MaskAlloc);
3149:   return DAG.getNode(LoongArchISD::VSHUF, DL, VT, MaskVec, V2, V1);
3150: }
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLE_XVSHUF`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLE_XVSHUF` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 3151-3225 / 第 3151-3225 行
```cpp
3151: 
3152: /// Shuffle vectors by lane to generate more optimized instructions.
3153: /// 256-bit shuffles are always considered as 2-lane 128-bit shuffles.
3154: ///
3155: /// Therefore, except for the following four cases, other cases are regarded
3156: /// as cross-lane shuffles, where optimization is relatively limited.
3157: ///
3158: /// - Shuffle high, low lanes of two inputs vector
3159: ///   <0, 1, 2, 3> + <4, 5, 6, 7> --- <0, 5, 3, 6>
3160: /// - Shuffle low, high lanes of two inputs vector
3161: ///   <0, 1, 2, 3> + <4, 5, 6, 7> --- <3, 6, 0, 5>
3162: /// - Shuffle low, low lanes of two inputs vector
3163: ///   <0, 1, 2, 3> + <4, 5, 6, 7> --- <3, 6, 3, 6>
3164: /// - Shuffle high, high lanes of two inputs vector
3165: ///   <0, 1, 2, 3> + <4, 5, 6, 7> --- <0, 5, 0, 5>
3166: ///
3167: /// The first case is the closest to LoongArch instructions and the other
3168: /// cases need to be converted to it for processing.
3169: ///
3170: /// This function will return true for the last three cases above and will
3171: /// modify V1, V2 and Mask. Otherwise, return false for the first case and
3172: /// cross-lane shuffle cases.
3173: static bool canonicalizeShuffleVectorByLane(
3174:     const SDLoc &DL, MutableArrayRef<int> Mask, MVT VT, SDValue &V1,
3175:     SDValue &V2, SelectionDAG &DAG, const LoongArchSubtarget &Subtarget) {
3176: 
3177:   enum HalfMaskType { HighLaneTy, LowLaneTy, None };
3178: 
3179:   int MaskSize = Mask.size();
3180:   int HalfSize = Mask.size() / 2;
3181:   MVT GRLenVT = Subtarget.getGRLenVT();
3182: 
3183:   HalfMaskType preMask = None, postMask = None;
3184: 
3185:   if (std::all_of(Mask.begin(), Mask.begin() + HalfSize, [&](int M) {
3186:         return M < 0 || (M >= 0 && M < HalfSize) ||
3187:                (M >= MaskSize && M < MaskSize + HalfSize);
3188:       }))
3189:     preMask = HighLaneTy;
3190:   else if (std::all_of(Mask.begin(), Mask.begin() + HalfSize, [&](int M) {
3191:              return M < 0 || (M >= HalfSize && M < MaskSize) ||
3192:                     (M >= MaskSize + HalfSize && M < MaskSize * 2);
3193:            }))
3194:     preMask = LowLaneTy;
3195: 
3196:   if (std::all_of(Mask.begin() + HalfSize, Mask.end(), [&](int M) {
3197:         return M < 0 || (M >= HalfSize && M < MaskSize) ||
3198:                (M >= MaskSize + HalfSize && M < MaskSize * 2);
3199:       }))
3200:     postMask = LowLaneTy;
3201:   else if (std::all_of(Mask.begin() + HalfSize, Mask.end(), [&](int M) {
3202:              return M < 0 || (M >= 0 && M < HalfSize) ||
3203:                     (M >= MaskSize && M < MaskSize + HalfSize);
3204:            }))
3205:     postMask = HighLaneTy;
3206: 
3207:   // The pre-half of mask is high lane type, and the post-half of mask
3208:   // is low lane type, which is closest to the LoongArch instructions.
3209:   //
3210:   // Note: In the LoongArch architecture, the high lane of mask corresponds
3211:   // to the lower 128-bit of vector register, and the low lane of mask
3212:   // corresponds the higher 128-bit of vector register.
3213:   if (preMask == HighLaneTy && postMask == LowLaneTy) {
3214:     return false;
3215:   }
3216:   if (preMask == LowLaneTy && postMask == HighLaneTy) {
3217:     V1 = DAG.getBitcast(MVT::v4i64, V1);
3218:     V1 = DAG.getNode(LoongArchISD::XVPERMI, DL, MVT::v4i64, V1,
3219:                      DAG.getConstant(0b01001110, DL, GRLenVT));
3220:     V1 = DAG.getBitcast(VT, V1);
3221: 
3222:     if (!V2.isUndef()) {
3223:       V2 = DAG.getBitcast(MVT::v4i64, V2);
3224:       V2 = DAG.getNode(LoongArchISD::XVPERMI, DL, MVT::v4i64, V2,
3225:                        DAG.getConstant(0b01001110, DL, GRLenVT));
```
- **EN**: The range implements or declares functions including `canonicalizeShuffleVectorByLane`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `canonicalizeShuffleVectorByLane` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3226-3300 / 第 3226-3300 行
```cpp
3226:       V2 = DAG.getBitcast(VT, V2);
3227:     }
3228: 
3229:     for (auto it = Mask.begin(); it < Mask.begin() + HalfSize; it++) {
3230:       *it = *it < 0 ? *it : *it - HalfSize;
3231:     }
3232:     for (auto it = Mask.begin() + HalfSize; it < Mask.end(); it++) {
3233:       *it = *it < 0 ? *it : *it + HalfSize;
3234:     }
3235:   } else if (preMask == LowLaneTy && postMask == LowLaneTy) {
3236:     V1 = DAG.getBitcast(MVT::v4i64, V1);
3237:     V1 = DAG.getNode(LoongArchISD::XVPERMI, DL, MVT::v4i64, V1,
3238:                      DAG.getConstant(0b11101110, DL, GRLenVT));
3239:     V1 = DAG.getBitcast(VT, V1);
3240: 
3241:     if (!V2.isUndef()) {
3242:       V2 = DAG.getBitcast(MVT::v4i64, V2);
3243:       V2 = DAG.getNode(LoongArchISD::XVPERMI, DL, MVT::v4i64, V2,
3244:                        DAG.getConstant(0b11101110, DL, GRLenVT));
3245:       V2 = DAG.getBitcast(VT, V2);
3246:     }
3247: 
3248:     for (auto it = Mask.begin(); it < Mask.begin() + HalfSize; it++) {
3249:       *it = *it < 0 ? *it : *it - HalfSize;
3250:     }
3251:   } else if (preMask == HighLaneTy && postMask == HighLaneTy) {
3252:     V1 = DAG.getBitcast(MVT::v4i64, V1);
3253:     V1 = DAG.getNode(LoongArchISD::XVPERMI, DL, MVT::v4i64, V1,
3254:                      DAG.getConstant(0b01000100, DL, GRLenVT));
3255:     V1 = DAG.getBitcast(VT, V1);
3256: 
3257:     if (!V2.isUndef()) {
3258:       V2 = DAG.getBitcast(MVT::v4i64, V2);
3259:       V2 = DAG.getNode(LoongArchISD::XVPERMI, DL, MVT::v4i64, V2,
3260:                        DAG.getConstant(0b01000100, DL, GRLenVT));
3261:       V2 = DAG.getBitcast(VT, V2);
3262:     }
3263: 
3264:     for (auto it = Mask.begin() + HalfSize; it < Mask.end(); it++) {
3265:       *it = *it < 0 ? *it : *it + HalfSize;
3266:     }
3267:   } else { // cross-lane
3268:     return false;
3269:   }
3270: 
3271:   return true;
3272: }
3273: 
3274: /// Lower VECTOR_SHUFFLE as lane permute and then shuffle (if possible).
3275: /// Only for 256-bit vector.
3276: ///
3277: /// For example:
3278: /// %2 = shufflevector <4 x i64> %0, <4 x i64> posion,
3279: ///                    <4 x i64> <i32 0, i32 3, i32 2, i32 0>
3280: /// is lowerded to:
3281: ///     (XVPERMI $xr2, $xr0, 78)
3282: ///     (XVSHUF  $xr1, $xr2, $xr0)
3283: ///     (XVORI   $xr0, $xr1, 0)
3284: static SDValue lowerVECTOR_SHUFFLEAsLanePermuteAndShuffle(const SDLoc &DL,
3285:                                                           ArrayRef<int> Mask,
3286:                                                           MVT VT, SDValue V1,
3287:                                                           SDValue V2,
3288:                                                           SelectionDAG &DAG) {
3289:   assert(VT.is256BitVector() && "Only for 256-bit vector shuffles!");
3290:   int Size = Mask.size();
3291:   int LaneSize = Size / 2;
3292: 
3293:   bool LaneCrossing[2] = {false, false};
3294:   for (int i = 0; i < Size; ++i)
3295:     if (Mask[i] >= 0 && ((Mask[i] % Size) / LaneSize) != (i / LaneSize))
3296:       LaneCrossing[(Mask[i] % Size) / LaneSize] = true;
3297: 
3298:   // Ensure that all lanes ared involved.
3299:   if (!LaneCrossing[0] && !LaneCrossing[1])
3300:     return SDValue();
```
- **EN**: The range implements or declares functions including `lowerVECTOR_SHUFFLEAsLanePermuteAndShuffle`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerVECTOR_SHUFFLEAsLanePermuteAndShuffle` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 3301-3375 / 第 3301-3375 行
```cpp
3301: 
3302:   SmallVector<int> InLaneMask;
3303:   InLaneMask.assign(Mask.begin(), Mask.end());
3304:   for (int i = 0; i < Size; ++i) {
3305:     int &M = InLaneMask[i];
3306:     if (M < 0)
3307:       continue;
3308:     if (((M % Size) / LaneSize) != (i / LaneSize))
3309:       M = (M % LaneSize) + ((i / LaneSize) * LaneSize) + Size;
3310:   }
3311: 
3312:   SDValue Flipped = DAG.getBitcast(MVT::v4i64, V1);
3313:   Flipped = DAG.getVectorShuffle(MVT::v4i64, DL, Flipped,
3314:                                  DAG.getUNDEF(MVT::v4i64), {2, 3, 0, 1});
3315:   Flipped = DAG.getBitcast(VT, Flipped);
3316:   return DAG.getVectorShuffle(VT, DL, V1, Flipped, InLaneMask);
3317: }
3318: 
3319: /// Dispatching routine to lower various 256-bit LoongArch vector shuffles.
3320: ///
3321: /// This routine breaks down the specific type of 256-bit shuffle and
3322: /// dispatches to the lowering routines accordingly.
3323: static SDValue lower256BitShuffle(const SDLoc &DL, ArrayRef<int> Mask, MVT VT,
3324:                                   SDValue V1, SDValue V2, SelectionDAG &DAG,
3325:                                   const LoongArchSubtarget &Subtarget) {
3326:   assert((VT.SimpleTy == MVT::v32i8 || VT.SimpleTy == MVT::v16i16 ||
3327:           VT.SimpleTy == MVT::v8i32 || VT.SimpleTy == MVT::v4i64 ||
3328:           VT.SimpleTy == MVT::v8f32 || VT.SimpleTy == MVT::v4f64) &&
3329:          "Vector type is unsupported for lasx!");
3330:   assert(V1.getSimpleValueType() == V2.getSimpleValueType() &&
3331:          "Two operands have different types!");
3332:   assert(VT.getVectorNumElements() == Mask.size() &&
3333:          "Unexpected mask size for shuffle!");
3334:   assert(Mask.size() % 2 == 0 && "Expected even mask size.");
3335:   assert(Mask.size() >= 4 && "Mask size is less than 4.");
3336: 
3337:   APInt KnownUndef, KnownZero;
3338:   computeZeroableShuffleElements(Mask, V1, V2, KnownUndef, KnownZero);
3339:   APInt Zeroable = KnownUndef | KnownZero;
3340: 
3341:   SDValue Result;
3342:   // TODO: Add more comparison patterns.
3343:   if (V2.isUndef()) {
3344:     if ((Result =
3345:              lowerVECTOR_SHUFFLE_XVREPLVEI(DL, Mask, VT, V1, DAG, Subtarget)))
3346:       return Result;
3347:     if ((Result = lowerVECTOR_SHUFFLE_XVSHUF4I(DL, Mask, VT, V1, V2, DAG,
3348:                                                Subtarget)))
3349:       return Result;
3350:     // Try to widen vectors to gain more optimization opportunities.
3351:     if (SDValue NewShuffle = widenShuffleMask(DL, Mask, VT, V1, V2, DAG))
3352:       return NewShuffle;
3353:     if ((Result =
3354:              lowerVECTOR_SHUFFLE_XVPERMI(DL, Mask, VT, V1, V2, DAG, Subtarget)))
3355:       return Result;
3356:     if ((Result = lowerVECTOR_SHUFFLE_XVPERM(DL, Mask, VT, V1, DAG, Subtarget)))
3357:       return Result;
3358:     if ((Result =
3359:              lowerVECTOR_SHUFFLE_IsReverse(DL, Mask, VT, V1, DAG, Subtarget)))
3360:       return Result;
3361: 
3362:     // TODO: This comment may be enabled in the future to better match the
3363:     // pattern for instruction selection.
3364:     /* V2 = V1; */
3365:   }
3366: 
3367:   // It is recommended not to change the pattern comparison order for better
3368:   // performance.
3369:   if ((Result = lowerVECTOR_SHUFFLE_XVPACKEV(DL, Mask, VT, V1, V2, DAG)))
3370:     return Result;
3371:   if ((Result = lowerVECTOR_SHUFFLE_XVPACKOD(DL, Mask, VT, V1, V2, DAG)))
3372:     return Result;
3373:   if ((Result = lowerVECTOR_SHUFFLE_XVILVH(DL, Mask, VT, V1, V2, DAG)))
3374:     return Result;
3375:   if ((Result = lowerVECTOR_SHUFFLE_XVILVL(DL, Mask, VT, V1, V2, DAG)))
```
- **EN**: The range implements or declares functions including `lower256BitShuffle`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lower256BitShuffle` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 3376-3450 / 第 3376-3450 行
```cpp
3376:     return Result;
3377:   if ((Result = lowerVECTOR_SHUFFLE_XVPICKEV(DL, Mask, VT, V1, V2, DAG)))
3378:     return Result;
3379:   if ((Result = lowerVECTOR_SHUFFLE_XVPICKOD(DL, Mask, VT, V1, V2, DAG)))
3380:     return Result;
3381:   if ((VT.SimpleTy == MVT::v4i64 || VT.SimpleTy == MVT::v4f64) &&
3382:       (Result =
3383:            lowerVECTOR_SHUFFLE_XVSHUF4I(DL, Mask, VT, V1, V2, DAG, Subtarget)))
3384:     return Result;
3385:   if ((Result =
3386:            lowerVECTOR_SHUFFLE_XVEXTRINS(DL, Mask, VT, V1, V2, DAG, Subtarget)))
3387:     return Result;
3388:   if ((Result = lowerVECTOR_SHUFFLEAsShift(DL, Mask, VT, V1, V2, DAG, Subtarget,
3389:                                            Zeroable)))
3390:     return Result;
3391:   if ((Result =
3392:            lowerVECTOR_SHUFFLE_XVPERMI(DL, Mask, VT, V1, V2, DAG, Subtarget)))
3393:     return Result;
3394:   if ((Result =
3395:            lowerVECTOR_SHUFFLE_XVINSVE0(DL, Mask, VT, V1, V2, DAG, Subtarget)))
3396:     return Result;
3397:   if ((Result = lowerVECTOR_SHUFFLEAsByteRotate(DL, Mask, VT, V1, V2, DAG,
3398:                                                 Subtarget)))
3399:     return Result;
3400: 
3401:   // canonicalize non cross-lane shuffle vector
3402:   SmallVector<int> NewMask(Mask);
3403:   if (canonicalizeShuffleVectorByLane(DL, NewMask, VT, V1, V2, DAG, Subtarget))
3404:     return lower256BitShuffle(DL, NewMask, VT, V1, V2, DAG, Subtarget);
3405: 
3406:   // FIXME: Handling the remaining cases earlier can degrade performance
3407:   // in some situations. Further analysis is required to enable more
3408:   // effective optimizations.
3409:   if (V2.isUndef()) {
3410:     if ((Result = lowerVECTOR_SHUFFLEAsLanePermuteAndShuffle(DL, NewMask, VT,
3411:                                                              V1, V2, DAG)))
3412:       return Result;
3413:   }
3414: 
3415:   if (SDValue NewShuffle = widenShuffleMask(DL, NewMask, VT, V1, V2, DAG))
3416:     return NewShuffle;
3417:   if ((Result = lowerVECTOR_SHUFFLE_XVSHUF(DL, NewMask, VT, V1, V2, DAG)))
3418:     return Result;
3419: 
3420:   return SDValue();
3421: }
3422: 
3423: SDValue LoongArchTargetLowering::lowerVECTOR_SHUFFLE(SDValue Op,
3424:                                                      SelectionDAG &DAG) const {
3425:   ShuffleVectorSDNode *SVOp = cast<ShuffleVectorSDNode>(Op);
3426:   ArrayRef<int> OrigMask = SVOp->getMask();
3427:   SDValue V1 = Op.getOperand(0);
3428:   SDValue V2 = Op.getOperand(1);
3429:   MVT VT = Op.getSimpleValueType();
3430:   int NumElements = VT.getVectorNumElements();
3431:   SDLoc DL(Op);
3432: 
3433:   bool V1IsUndef = V1.isUndef();
3434:   bool V2IsUndef = V2.isUndef();
3435:   if (V1IsUndef && V2IsUndef)
3436:     return DAG.getUNDEF(VT);
3437: 
3438:   // When we create a shuffle node we put the UNDEF node to second operand,
3439:   // but in some cases the first operand may be transformed to UNDEF.
3440:   // In this case we should just commute the node.
3441:   if (V1IsUndef)
3442:     return DAG.getCommutedVectorShuffle(*SVOp);
3443: 
3444:   // Check for non-undef masks pointing at an undef vector and make the masks
3445:   // undef as well. This makes it easier to match the shuffle based solely on
3446:   // the mask.
3447:   if (V2IsUndef &&
3448:       any_of(OrigMask, [NumElements](int M) { return M >= NumElements; })) {
3449:     SmallVector<int, 8> NewMask(OrigMask);
3450:     for (int &M : NewMask)
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerVECTOR_SHUFFLE`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerVECTOR_SHUFFLE` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 3451-3525 / 第 3451-3525 行
```cpp
3451:       if (M >= NumElements)
3452:         M = -1;
3453:     return DAG.getVectorShuffle(VT, DL, V1, V2, NewMask);
3454:   }
3455: 
3456:   // Check for illegal shuffle mask element index values.
3457:   int MaskUpperLimit = OrigMask.size() * (V2IsUndef ? 1 : 2);
3458:   (void)MaskUpperLimit;
3459:   assert(llvm::all_of(OrigMask,
3460:                       [&](int M) { return -1 <= M && M < MaskUpperLimit; }) &&
3461:          "Out of bounds shuffle index");
3462: 
3463:   // For each vector width, delegate to a specialized lowering routine.
3464:   if (VT.is128BitVector())
3465:     return lower128BitShuffle(DL, OrigMask, VT, V1, V2, DAG, Subtarget);
3466: 
3467:   if (VT.is256BitVector())
3468:     return lower256BitShuffle(DL, OrigMask, VT, V1, V2, DAG, Subtarget);
3469: 
3470:   return SDValue();
3471: }
3472: 
3473: SDValue LoongArchTargetLowering::lowerFP_TO_FP16(SDValue Op,
3474:                                                  SelectionDAG &DAG) const {
3475:   // Custom lower to ensure the libcall return is passed in an FPR on hard
3476:   // float ABIs.
3477:   SDLoc DL(Op);
3478:   MakeLibCallOptions CallOptions;
3479:   SDValue Op0 = Op.getOperand(0);
3480:   SDValue Chain = SDValue();
3481:   RTLIB::Libcall LC = RTLIB::getFPROUND(Op0.getValueType(), MVT::f16);
3482:   SDValue Res;
3483:   std::tie(Res, Chain) =
3484:       makeLibCall(DAG, LC, MVT::f32, Op0, CallOptions, DL, Chain);
3485:   if (Subtarget.is64Bit())
3486:     return DAG.getNode(LoongArchISD::MOVFR2GR_S_LA64, DL, MVT::i64, Res);
3487:   return DAG.getBitcast(MVT::i32, Res);
3488: }
3489: 
3490: SDValue LoongArchTargetLowering::lowerFP16_TO_FP(SDValue Op,
3491:                                                  SelectionDAG &DAG) const {
3492:   // Custom lower to ensure the libcall argument is passed in an FPR on hard
3493:   // float ABIs.
3494:   SDLoc DL(Op);
3495:   MakeLibCallOptions CallOptions;
3496:   SDValue Op0 = Op.getOperand(0);
3497:   SDValue Chain = SDValue();
3498:   SDValue Arg = Subtarget.is64Bit() ? DAG.getNode(LoongArchISD::MOVGR2FR_W_LA64,
3499:                                                   DL, MVT::f32, Op0)
3500:                                     : DAG.getBitcast(MVT::f32, Op0);
3501:   SDValue Res;
3502:   std::tie(Res, Chain) = makeLibCall(DAG, RTLIB::FPEXT_F16_F32, MVT::f32, Arg,
3503:                                      CallOptions, DL, Chain);
3504:   return Res;
3505: }
3506: 
3507: SDValue LoongArchTargetLowering::lowerFP_TO_BF16(SDValue Op,
3508:                                                  SelectionDAG &DAG) const {
3509:   assert(Subtarget.hasBasicF() && "Unexpected custom legalization");
3510:   SDLoc DL(Op);
3511:   MakeLibCallOptions CallOptions;
3512:   RTLIB::Libcall LC =
3513:       RTLIB::getFPROUND(Op.getOperand(0).getValueType(), MVT::bf16);
3514:   SDValue Res =
3515:       makeLibCall(DAG, LC, MVT::f32, Op.getOperand(0), CallOptions, DL).first;
3516:   if (Subtarget.is64Bit())
3517:     return DAG.getNode(LoongArchISD::MOVFR2GR_S_LA64, DL, MVT::i64, Res);
3518:   return DAG.getBitcast(MVT::i32, Res);
3519: }
3520: 
3521: SDValue LoongArchTargetLowering::lowerBF16_TO_FP(SDValue Op,
3522:                                                  SelectionDAG &DAG) const {
3523:   assert(Subtarget.hasBasicF() && "Unexpected custom legalization");
3524:   MVT VT = Op.getSimpleValueType();
3525:   SDLoc DL(Op);
```
- **EN**: The range implements or declares functions including `assert`, `LoongArchTargetLowering::lowerFP_TO_FP16`, `LoongArchTargetLowering::lowerFP16_TO_FP`, `LoongArchTargetLowering::lowerFP_TO_BF16`, `LoongArchTargetLowering::lowerBF16_TO_FP`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `assert`, `LoongArchTargetLowering::lowerFP_TO_FP16`, `LoongArchTargetLowering::lowerFP16_TO_FP`, `LoongArchTargetLowering::lowerFP_TO_BF16`, `LoongArchTargetLowering::lowerBF16_TO_FP` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 3526-3600 / 第 3526-3600 行
```cpp
3526:   Op = DAG.getNode(
3527:       ISD::SHL, DL, Op.getOperand(0).getValueType(), Op.getOperand(0),
3528:       DAG.getShiftAmountConstant(16, Op.getOperand(0).getValueType(), DL));
3529:   SDValue Res = Subtarget.is64Bit() ? DAG.getNode(LoongArchISD::MOVGR2FR_W_LA64,
3530:                                                   DL, MVT::f32, Op)
3531:                                     : DAG.getBitcast(MVT::f32, Op);
3532:   if (VT != MVT::f32)
3533:     return DAG.getNode(ISD::FP_EXTEND, DL, VT, Res);
3534:   return Res;
3535: }
3536: 
3537: // Lower BUILD_VECTOR as broadcast load (if possible).
3538: // For example:
3539: //   %a = load i8, ptr %ptr
3540: //   %b = build_vector %a, %a, %a, %a
3541: // is lowered to :
3542: //   (VLDREPL_B $a0, 0)
3543: static SDValue lowerBUILD_VECTORAsBroadCastLoad(BuildVectorSDNode *BVOp,
3544:                                                 const SDLoc &DL,
3545:                                                 SelectionDAG &DAG) {
3546:   MVT VT = BVOp->getSimpleValueType(0);
3547:   int NumOps = BVOp->getNumOperands();
3548: 
3549:   assert((VT.is128BitVector() || VT.is256BitVector()) &&
3550:          "Unsupported vector type for broadcast.");
3551: 
3552:   SDValue IdentitySrc;
3553:   bool IsIdeneity = true;
3554: 
3555:   for (int i = 0; i != NumOps; i++) {
3556:     SDValue Op = BVOp->getOperand(i);
3557:     if (Op.getOpcode() != ISD::LOAD || (IdentitySrc && Op != IdentitySrc)) {
3558:       IsIdeneity = false;
3559:       break;
3560:     }
3561:     IdentitySrc = BVOp->getOperand(0);
3562:   }
3563: 
3564:   // make sure that this load is valid and only has one user.
3565:   if (!IsIdeneity || !IdentitySrc || !BVOp->isOnlyUserOf(IdentitySrc.getNode()))
3566:     return SDValue();
3567: 
3568:   auto *LN = cast<LoadSDNode>(IdentitySrc);
3569:   auto ExtType = LN->getExtensionType();
3570: 
3571:   if ((ExtType == ISD::EXTLOAD || ExtType == ISD::NON_EXTLOAD) &&
3572:       VT.getScalarSizeInBits() == LN->getMemoryVT().getScalarSizeInBits()) {
3573:     // Indexed loads and stores are not supported on LoongArch.
3574:     assert(LN->isUnindexed() && "Unexpected indexed load.");
3575: 
3576:     SDVTList Tys = DAG.getVTList(VT, MVT::Other);
3577:     // The offset operand of unindexed load is always undefined, so there is
3578:     // no need to pass it to VLDREPL.
3579:     SDValue Ops[] = {LN->getChain(), LN->getBasePtr()};
3580:     SDValue BCast = DAG.getNode(LoongArchISD::VLDREPL, DL, Tys, Ops);
3581:     DAG.ReplaceAllUsesOfValueWith(SDValue(LN, 1), BCast.getValue(1));
3582:     return BCast;
3583:   }
3584:   return SDValue();
3585: }
3586: 
3587: // Sequentially insert elements from Ops into Vector, from low to high indices.
3588: // Note: Ops can have fewer elements than Vector.
3589: static void fillVector(ArrayRef<SDValue> Ops, SelectionDAG &DAG, SDLoc DL,
3590:                        const LoongArchSubtarget &Subtarget, SDValue &Vector,
3591:                        EVT ResTy) {
3592:   assert(Ops.size() <= ResTy.getVectorNumElements());
3593: 
3594:   SDValue Op0 = Ops[0];
3595:   if (!Op0.isUndef())
3596:     Vector = DAG.getNode(ISD::SCALAR_TO_VECTOR, DL, ResTy, Op0);
3597:   for (unsigned i = 1; i < Ops.size(); ++i) {
3598:     SDValue Opi = Ops[i];
3599:     if (Opi.isUndef())
3600:       continue;
```
- **EN**: The range implements or declares functions including `lowerBUILD_VECTORAsBroadCastLoad`, `fillVector`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `lowerBUILD_VECTORAsBroadCastLoad`, `fillVector` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 3601-3675 / 第 3601-3675 行
```cpp
3601:     Vector = DAG.getNode(ISD::INSERT_VECTOR_ELT, DL, ResTy, Vector, Opi,
3602:                          DAG.getConstant(i, DL, Subtarget.getGRLenVT()));
3603:   }
3604: }
3605: 
3606: // Build a ResTy subvector from Node, taking NumElts elements starting at index
3607: // 'first'.
3608: static SDValue fillSubVectorFromBuildVector(BuildVectorSDNode *Node,
3609:                                             SelectionDAG &DAG, SDLoc DL,
3610:                                             const LoongArchSubtarget &Subtarget,
3611:                                             EVT ResTy, unsigned first) {
3612:   unsigned NumElts = ResTy.getVectorNumElements();
3613: 
3614:   assert(first + NumElts <= Node->getSimpleValueType(0).getVectorNumElements());
3615: 
3616:   SmallVector<SDValue, 16> Ops(Node->op_begin() + first,
3617:                                Node->op_begin() + first + NumElts);
3618:   SDValue Vector = DAG.getUNDEF(ResTy);
3619:   fillVector(Ops, DAG, DL, Subtarget, Vector, ResTy);
3620:   return Vector;
3621: }
3622: 
3623: SDValue LoongArchTargetLowering::lowerBUILD_VECTOR(SDValue Op,
3624:                                                    SelectionDAG &DAG) const {
3625:   BuildVectorSDNode *Node = cast<BuildVectorSDNode>(Op);
3626:   MVT VT = Node->getSimpleValueType(0);
3627:   EVT ResTy = Op->getValueType(0);
3628:   unsigned NumElts = ResTy.getVectorNumElements();
3629:   SDLoc DL(Op);
3630:   APInt SplatValue, SplatUndef;
3631:   unsigned SplatBitSize;
3632:   bool HasAnyUndefs;
3633:   bool IsConstant = false;
3634:   bool UseSameConstant = true;
3635:   SDValue ConstantValue;
3636:   bool Is128Vec = ResTy.is128BitVector();
3637:   bool Is256Vec = ResTy.is256BitVector();
3638: 
3639:   if ((!Subtarget.hasExtLSX() || !Is128Vec) &&
3640:       (!Subtarget.hasExtLASX() || !Is256Vec))
3641:     return SDValue();
3642: 
3643:   if (SDValue Result = lowerBUILD_VECTORAsBroadCastLoad(Node, DL, DAG))
3644:     return Result;
3645: 
3646:   if (Node->isConstantSplat(SplatValue, SplatUndef, SplatBitSize, HasAnyUndefs,
3647:                             /*MinSplatBits=*/8) &&
3648:       SplatBitSize <= 64) {
3649:     // We can only cope with 8, 16, 32, or 64-bit elements.
3650:     if (SplatBitSize != 8 && SplatBitSize != 16 && SplatBitSize != 32 &&
3651:         SplatBitSize != 64)
3652:       return SDValue();
3653: 
3654:     if (SplatBitSize == 64 && !Subtarget.is64Bit()) {
3655:       // We can only handle 64-bit elements that are within
3656:       // the signed 10-bit range or match vldi patterns on 32-bit targets.
3657:       // See the BUILD_VECTOR case in LoongArchDAGToDAGISel::Select().
3658:       if (!SplatValue.isSignedIntN(10) &&
3659:           !isImmVLDILegalForMode1(SplatValue, SplatBitSize).first)
3660:         return SDValue();
3661:       if ((Is128Vec && ResTy == MVT::v4i32) ||
3662:           (Is256Vec && ResTy == MVT::v8i32))
3663:         return Op;
3664:     }
3665: 
3666:     EVT ViaVecTy;
3667: 
3668:     switch (SplatBitSize) {
3669:     default:
3670:       return SDValue();
3671:     case 8:
3672:       ViaVecTy = Is128Vec ? MVT::v16i8 : MVT::v32i8;
3673:       break;
3674:     case 16:
3675:       ViaVecTy = Is128Vec ? MVT::v8i16 : MVT::v16i16;
```
- **EN**: The range implements or declares functions including `fillSubVectorFromBuildVector`, `LoongArchTargetLowering::lowerBUILD_VECTOR`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `fillSubVectorFromBuildVector`, `LoongArchTargetLowering::lowerBUILD_VECTOR` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3676-3750 / 第 3676-3750 行
```cpp
3676:       break;
3677:     case 32:
3678:       ViaVecTy = Is128Vec ? MVT::v4i32 : MVT::v8i32;
3679:       break;
3680:     case 64:
3681:       ViaVecTy = Is128Vec ? MVT::v2i64 : MVT::v4i64;
3682:       break;
3683:     }
3684: 
3685:     // SelectionDAG::getConstant will promote SplatValue appropriately.
3686:     SDValue Result = DAG.getConstant(SplatValue, DL, ViaVecTy);
3687: 
3688:     // Bitcast to the type we originally wanted.
3689:     if (ViaVecTy != ResTy)
3690:       Result = DAG.getNode(ISD::BITCAST, SDLoc(Node), ResTy, Result);
3691: 
3692:     return Result;
3693:   }
3694: 
3695:   if (DAG.isSplatValue(Op, /*AllowUndefs=*/false))
3696:     return Op;
3697: 
3698:   for (unsigned i = 0; i < NumElts; ++i) {
3699:     SDValue Opi = Node->getOperand(i);
3700:     if (isIntOrFPConstant(Opi)) {
3701:       IsConstant = true;
3702:       if (!ConstantValue.getNode())
3703:         ConstantValue = Opi;
3704:       else if (ConstantValue != Opi)
3705:         UseSameConstant = false;
3706:     }
3707:   }
3708: 
3709:   // If the type of BUILD_VECTOR is v2f64, custom legalizing it has no benefits.
3710:   if (IsConstant && UseSameConstant && ResTy != MVT::v2f64) {
3711:     SDValue Result = DAG.getSplatBuildVector(ResTy, DL, ConstantValue);
3712:     for (unsigned i = 0; i < NumElts; ++i) {
3713:       SDValue Opi = Node->getOperand(i);
3714:       if (!isIntOrFPConstant(Opi))
3715:         Result = DAG.getNode(ISD::INSERT_VECTOR_ELT, DL, ResTy, Result, Opi,
3716:                              DAG.getConstant(i, DL, Subtarget.getGRLenVT()));
3717:     }
3718:     return Result;
3719:   }
3720: 
3721:   if (!IsConstant) {
3722:     // If the BUILD_VECTOR has a repeated pattern, use INSERT_VECTOR_ELT to fill
3723:     // the sub-sequence of the vector and then broadcast the sub-sequence.
3724:     //
3725:     // TODO: If the BUILD_VECTOR contains undef elements, consider falling
3726:     // back to use INSERT_VECTOR_ELT to materialize the vector, because it
3727:     // generates worse code in some cases. This could be further optimized
3728:     // with more consideration.
3729:     SmallVector<SDValue> Sequence;
3730:     BitVector UndefElements;
3731:     if (Node->getRepeatedSequence(Sequence, &UndefElements) &&
3732:         UndefElements.count() == 0) {
3733:       // Using LSX instructions to fill the sub-sequence of 256-bits vector,
3734:       // because the high part can be simply treated as undef.
3735:       SDValue Vector = DAG.getUNDEF(ResTy);
3736:       EVT FillTy = Is256Vec
3737:                        ? ResTy.getHalfNumVectorElementsVT(*DAG.getContext())
3738:                        : ResTy;
3739:       SDValue FillVec =
3740:           Is256Vec ? DAG.getExtractSubvector(DL, FillTy, Vector, 0) : Vector;
3741: 
3742:       fillVector(Sequence, DAG, DL, Subtarget, FillVec, FillTy);
3743: 
3744:       unsigned SeqLen = Sequence.size();
3745:       unsigned SplatLen = NumElts / SeqLen;
3746:       MVT SplatEltTy = MVT::getIntegerVT(VT.getScalarSizeInBits() * SeqLen);
3747:       MVT SplatTy = MVT::getVectorVT(SplatEltTy, SplatLen);
3748: 
3749:       // If size of the sub-sequence is half of a 256-bits vector, bitcast the
3750:       // vector to v4i64 type in order to match the pattern of XVREPLVE0Q.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 3751-3825 / 第 3751-3825 行
```cpp
3751:       if (SplatEltTy == MVT::i128)
3752:         SplatTy = MVT::v4i64;
3753: 
3754:       SDValue SplatVec;
3755:       SDValue SrcVec = DAG.getBitcast(
3756:           SplatTy,
3757:           Is256Vec ? DAG.getInsertSubvector(DL, Vector, FillVec, 0) : FillVec);
3758:       if (Is256Vec) {
3759:         SplatVec =
3760:             DAG.getNode((SplatEltTy == MVT::i128) ? LoongArchISD::XVREPLVE0Q
3761:                                                   : LoongArchISD::XVREPLVE0,
3762:                         DL, SplatTy, SrcVec);
3763:       } else {
3764:         SplatVec = DAG.getNode(LoongArchISD::VREPLVEI, DL, SplatTy, SrcVec,
3765:                                DAG.getConstant(0, DL, Subtarget.getGRLenVT()));
3766:       }
3767: 
3768:       return DAG.getBitcast(ResTy, SplatVec);
3769:     }
3770: 
3771:     // Use INSERT_VECTOR_ELT operations rather than expand to stores, because
3772:     // using memory operations is much lower.
3773:     //
3774:     // For 256-bit vectors, normally split into two halves and concatenate.
3775:     // Special case: for v8i32/v8f32/v4i64/v4f64, if the upper half has only
3776:     // one non-undef element, skip spliting to avoid a worse result.
3777:     if (ResTy == MVT::v8i32 || ResTy == MVT::v8f32 || ResTy == MVT::v4i64 ||
3778:         ResTy == MVT::v4f64) {
3779:       unsigned NonUndefCount = 0;
3780:       for (unsigned i = NumElts / 2; i < NumElts; ++i) {
3781:         if (!Node->getOperand(i).isUndef()) {
3782:           ++NonUndefCount;
3783:           if (NonUndefCount > 1)
3784:             break;
3785:         }
3786:       }
3787:       if (NonUndefCount == 1)
3788:         return fillSubVectorFromBuildVector(Node, DAG, DL, Subtarget, ResTy, 0);
3789:     }
3790: 
3791:     EVT VecTy =
3792:         Is256Vec ? ResTy.getHalfNumVectorElementsVT(*DAG.getContext()) : ResTy;
3793:     SDValue Vector =
3794:         fillSubVectorFromBuildVector(Node, DAG, DL, Subtarget, VecTy, 0);
3795: 
3796:     if (Is128Vec)
3797:       return Vector;
3798: 
3799:     SDValue VectorHi = fillSubVectorFromBuildVector(Node, DAG, DL, Subtarget,
3800:                                                     VecTy, NumElts / 2);
3801: 
3802:     return DAG.getNode(ISD::CONCAT_VECTORS, DL, ResTy, Vector, VectorHi);
3803:   }
3804: 
3805:   return SDValue();
3806: }
3807: 
3808: SDValue LoongArchTargetLowering::lowerCONCAT_VECTORS(SDValue Op,
3809:                                                      SelectionDAG &DAG) const {
3810:   SDLoc DL(Op);
3811:   MVT ResVT = Op.getSimpleValueType();
3812:   assert(ResVT.is256BitVector() && Op.getNumOperands() == 2);
3813: 
3814:   unsigned NumOperands = Op.getNumOperands();
3815:   unsigned NumFreezeUndef = 0;
3816:   unsigned NumZero = 0;
3817:   unsigned NumNonZero = 0;
3818:   unsigned NonZeros = 0;
3819:   SmallSet<SDValue, 4> Undefs;
3820:   for (unsigned i = 0; i != NumOperands; ++i) {
3821:     SDValue SubVec = Op.getOperand(i);
3822:     if (SubVec.isUndef())
3823:       continue;
3824:     if (ISD::isFreezeUndef(SubVec.getNode())) {
3825:       // If the freeze(undef) has multiple uses then we must fold to zero.
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerCONCAT_VECTORS`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerCONCAT_VECTORS` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 3826-3900 / 第 3826-3900 行
```cpp
3826:       if (SubVec.hasOneUse()) {
3827:         ++NumFreezeUndef;
3828:       } else {
3829:         ++NumZero;
3830:         Undefs.insert(SubVec);
3831:       }
3832:     } else if (ISD::isBuildVectorAllZeros(SubVec.getNode()))
3833:       ++NumZero;
3834:     else {
3835:       assert(i < sizeof(NonZeros) * CHAR_BIT); // Ensure the shift is in range.
3836:       NonZeros |= 1 << i;
3837:       ++NumNonZero;
3838:     }
3839:   }
3840: 
3841:   // If we have more than 2 non-zeros, build each half separately.
3842:   if (NumNonZero > 2) {
3843:     MVT HalfVT = ResVT.getHalfNumVectorElementsVT();
3844:     ArrayRef<SDUse> Ops = Op->ops();
3845:     SDValue Lo = DAG.getNode(ISD::CONCAT_VECTORS, DL, HalfVT,
3846:                              Ops.slice(0, NumOperands / 2));
3847:     SDValue Hi = DAG.getNode(ISD::CONCAT_VECTORS, DL, HalfVT,
3848:                              Ops.slice(NumOperands / 2));
3849:     return DAG.getNode(ISD::CONCAT_VECTORS, DL, ResVT, Lo, Hi);
3850:   }
3851: 
3852:   // Otherwise, build it up through insert_subvectors.
3853:   SDValue Vec = NumZero ? DAG.getConstant(0, DL, ResVT)
3854:                         : (NumFreezeUndef ? DAG.getFreeze(DAG.getUNDEF(ResVT))
3855:                                           : DAG.getUNDEF(ResVT));
3856: 
3857:   // Replace Undef operands with ZeroVector.
3858:   for (SDValue U : Undefs)
3859:     DAG.ReplaceAllUsesWith(U, DAG.getConstant(0, DL, U.getSimpleValueType()));
3860: 
3861:   MVT SubVT = Op.getOperand(0).getSimpleValueType();
3862:   unsigned NumSubElems = SubVT.getVectorNumElements();
3863:   for (unsigned i = 0; i != NumOperands; ++i) {
3864:     if ((NonZeros & (1 << i)) == 0)
3865:       continue;
3866: 
3867:     Vec = DAG.getNode(ISD::INSERT_SUBVECTOR, DL, ResVT, Vec, Op.getOperand(i),
3868:                       DAG.getVectorIdxConstant(i * NumSubElems, DL));
3869:   }
3870: 
3871:   return Vec;
3872: }
3873: 
3874: SDValue
3875: LoongArchTargetLowering::lowerEXTRACT_VECTOR_ELT(SDValue Op,
3876:                                                  SelectionDAG &DAG) const {
3877:   MVT EltVT = Op.getSimpleValueType();
3878:   SDValue Vec = Op->getOperand(0);
3879:   EVT VecTy = Vec->getValueType(0);
3880:   SDValue Idx = Op->getOperand(1);
3881:   SDLoc DL(Op);
3882:   MVT GRLenVT = Subtarget.getGRLenVT();
3883: 
3884:   assert(VecTy.is256BitVector() && "Unexpected EXTRACT_VECTOR_ELT vector type");
3885: 
3886:   if (isa<ConstantSDNode>(Idx))
3887:     return Op;
3888: 
3889:   switch (VecTy.getSimpleVT().SimpleTy) {
3890:   default:
3891:     llvm_unreachable("Unexpected type");
3892:   case MVT::v32i8:
3893:   case MVT::v16i16:
3894:   case MVT::v4i64:
3895:   case MVT::v4f64: {
3896:     // Extract the high half subvector and place it to the low half of a new
3897:     // vector. It doesn't matter what the high half of the new vector is.
3898:     EVT HalfTy = VecTy.getHalfNumVectorElementsVT(*DAG.getContext());
3899:     SDValue VecHi =
3900:         DAG.getExtractSubvector(DL, HalfTy, Vec, HalfTy.getVectorNumElements());
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerEXTRACT_VECTOR_ELT`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerEXTRACT_VECTOR_ELT` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3901-3975 / 第 3901-3975 行
```cpp
3901:     SDValue TmpVec =
3902:         DAG.getNode(ISD::INSERT_SUBVECTOR, DL, VecTy, DAG.getUNDEF(VecTy),
3903:                     VecHi, DAG.getConstant(0, DL, GRLenVT));
3904: 
3905:     // Shuffle the origin Vec and the TmpVec using MaskVec, the lowest element
3906:     // of MaskVec is Idx, the rest do not matter. ResVec[0] will hold the
3907:     // desired element.
3908:     SDValue IdxCp =
3909:         Subtarget.is64Bit()
3910:             ? DAG.getNode(LoongArchISD::MOVGR2FR_W_LA64, DL, MVT::f32, Idx)
3911:             : DAG.getBitcast(MVT::f32, Idx);
3912:     SDValue IdxVec = DAG.getNode(ISD::SCALAR_TO_VECTOR, DL, MVT::v8f32, IdxCp);
3913:     SDValue MaskVec =
3914:         DAG.getBitcast((VecTy == MVT::v4f64) ? MVT::v4i64 : VecTy, IdxVec);
3915:     SDValue ResVec =
3916:         DAG.getNode(LoongArchISD::VSHUF, DL, VecTy, MaskVec, TmpVec, Vec);
3917: 
3918:     return DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, EltVT, ResVec,
3919:                        DAG.getConstant(0, DL, GRLenVT));
3920:   }
3921:   case MVT::v8i32:
3922:   case MVT::v8f32: {
3923:     SDValue SplatIdx = DAG.getSplatBuildVector(MVT::v8i32, DL, Idx);
3924:     SDValue SplatValue =
3925:         DAG.getNode(LoongArchISD::XVPERM, DL, VecTy, Vec, SplatIdx);
3926: 
3927:     return DAG.getNode(ISD::EXTRACT_VECTOR_ELT, DL, EltVT, SplatValue,
3928:                        DAG.getConstant(0, DL, GRLenVT));
3929:   }
3930:   }
3931: }
3932: 
3933: SDValue
3934: LoongArchTargetLowering::lowerINSERT_VECTOR_ELT(SDValue Op,
3935:                                                 SelectionDAG &DAG) const {
3936:   MVT VT = Op.getSimpleValueType();
3937:   MVT EltVT = VT.getVectorElementType();
3938:   unsigned NumElts = VT.getVectorNumElements();
3939:   unsigned EltSizeInBits = EltVT.getScalarSizeInBits();
3940:   SDLoc DL(Op);
3941:   SDValue Op0 = Op.getOperand(0);
3942:   SDValue Op1 = Op.getOperand(1);
3943:   SDValue Op2 = Op.getOperand(2);
3944: 
3945:   if (isa<ConstantSDNode>(Op2))
3946:     return Op;
3947: 
3948:   MVT IdxTy = MVT::getIntegerVT(EltSizeInBits);
3949:   MVT IdxVTy = MVT::getVectorVT(IdxTy, NumElts);
3950: 
3951:   if (!isTypeLegal(VT) || !isTypeLegal(IdxVTy))
3952:     return SDValue();
3953: 
3954:   SDValue SplatElt = DAG.getSplatBuildVector(VT, DL, Op1);
3955:   SmallVector<SDValue, 32> RawIndices;
3956:   SDValue SplatIdx;
3957:   SDValue Indices;
3958: 
3959:   if (!Subtarget.is64Bit() && IdxTy == MVT::i64) {
3960:     MVT PairVTy = MVT::getVectorVT(MVT::i32, NumElts * 2);
3961:     for (unsigned i = 0; i < NumElts; ++i) {
3962:       RawIndices.push_back(Op2);
3963:       RawIndices.push_back(DAG.getConstant(0, DL, MVT::i32));
3964:     }
3965:     SplatIdx = DAG.getBuildVector(PairVTy, DL, RawIndices);
3966:     SplatIdx = DAG.getBitcast(IdxVTy, SplatIdx);
3967: 
3968:     RawIndices.clear();
3969:     for (unsigned i = 0; i < NumElts; ++i) {
3970:       RawIndices.push_back(DAG.getConstant(i, DL, MVT::i32));
3971:       RawIndices.push_back(DAG.getConstant(0, DL, MVT::i32));
3972:     }
3973:     Indices = DAG.getBuildVector(PairVTy, DL, RawIndices);
3974:     Indices = DAG.getBitcast(IdxVTy, Indices);
3975:   } else {
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerINSERT_VECTOR_ELT`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerINSERT_VECTOR_ELT` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 3976-4050 / 第 3976-4050 行
```cpp
3976:     SplatIdx = DAG.getSplatBuildVector(IdxVTy, DL, Op2);
3977: 
3978:     for (unsigned i = 0; i < NumElts; ++i)
3979:       RawIndices.push_back(DAG.getConstant(i, DL, Subtarget.getGRLenVT()));
3980:     Indices = DAG.getBuildVector(IdxVTy, DL, RawIndices);
3981:   }
3982: 
3983:   // insert vec, elt, idx
3984:   // =>
3985:   // select (splatidx == {0,1,2...}) ? splatelt : vec
3986:   SDValue SelectCC =
3987:       DAG.getSetCC(DL, IdxVTy, SplatIdx, Indices, ISD::CondCode::SETEQ);
3988:   return DAG.getNode(ISD::VSELECT, DL, VT, SelectCC, SplatElt, Op0);
3989: }
3990: 
3991: SDValue LoongArchTargetLowering::lowerATOMIC_FENCE(SDValue Op,
3992:                                                    SelectionDAG &DAG) const {
3993:   SDLoc DL(Op);
3994:   SyncScope::ID FenceSSID =
3995:       static_cast<SyncScope::ID>(Op.getConstantOperandVal(2));
3996: 
3997:   // singlethread fences only synchronize with signal handlers on the same
3998:   // thread and thus only need to preserve instruction order, not actually
3999:   // enforce memory ordering.
4000:   if (FenceSSID == SyncScope::SingleThread)
4001:     // MEMBARRIER is a compiler barrier; it codegens to a no-op.
4002:     return DAG.getNode(ISD::MEMBARRIER, DL, MVT::Other, Op.getOperand(0));
4003: 
4004:   return Op;
4005: }
4006: 
4007: SDValue LoongArchTargetLowering::lowerWRITE_REGISTER(SDValue Op,
4008:                                                      SelectionDAG &DAG) const {
4009: 
4010:   if (Subtarget.is64Bit() && Op.getOperand(2).getValueType() == MVT::i32) {
4011:     DAG.getContext()->emitError(
4012:         "On LA64, only 64-bit registers can be written.");
4013:     return Op.getOperand(0);
4014:   }
4015: 
4016:   if (!Subtarget.is64Bit() && Op.getOperand(2).getValueType() == MVT::i64) {
4017:     DAG.getContext()->emitError(
4018:         "On LA32, only 32-bit registers can be written.");
4019:     return Op.getOperand(0);
4020:   }
4021: 
4022:   return Op;
4023: }
4024: 
4025: SDValue LoongArchTargetLowering::lowerFRAMEADDR(SDValue Op,
4026:                                                 SelectionDAG &DAG) const {
4027:   if (!isa<ConstantSDNode>(Op.getOperand(0))) {
4028:     DAG.getContext()->emitError("argument to '__builtin_frame_address' must "
4029:                                 "be a constant integer");
4030:     return SDValue();
4031:   }
4032: 
4033:   MachineFunction &MF = DAG.getMachineFunction();
4034:   MF.getFrameInfo().setFrameAddressIsTaken(true);
4035:   Register FrameReg = Subtarget.getRegisterInfo()->getFrameRegister(MF);
4036:   EVT VT = Op.getValueType();
4037:   SDLoc DL(Op);
4038:   SDValue FrameAddr = DAG.getCopyFromReg(DAG.getEntryNode(), DL, FrameReg, VT);
4039:   unsigned Depth = Op.getConstantOperandVal(0);
4040:   int GRLenInBytes = Subtarget.getGRLen() / 8;
4041: 
4042:   while (Depth--) {
4043:     int Offset = -(GRLenInBytes * 2);
4044:     SDValue Ptr = DAG.getNode(ISD::ADD, DL, VT, FrameAddr,
4045:                               DAG.getSignedConstant(Offset, DL, VT));
4046:     FrameAddr =
4047:         DAG.getLoad(VT, DL, DAG.getEntryNode(), Ptr, MachinePointerInfo());
4048:   }
4049:   return FrameAddr;
4050: }
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerATOMIC_FENCE`, `LoongArchTargetLowering::lowerWRITE_REGISTER`, `LoongArchTargetLowering::lowerFRAMEADDR`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerATOMIC_FENCE`, `LoongArchTargetLowering::lowerWRITE_REGISTER`, `LoongArchTargetLowering::lowerFRAMEADDR` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 4051-4125 / 第 4051-4125 行
```cpp
4051: 
4052: SDValue LoongArchTargetLowering::lowerRETURNADDR(SDValue Op,
4053:                                                  SelectionDAG &DAG) const {
4054:   // Currently only support lowering return address for current frame.
4055:   if (Op.getConstantOperandVal(0) != 0) {
4056:     DAG.getContext()->emitError(
4057:         "return address can only be determined for the current frame");
4058:     return SDValue();
4059:   }
4060: 
4061:   MachineFunction &MF = DAG.getMachineFunction();
4062:   MF.getFrameInfo().setReturnAddressIsTaken(true);
4063:   MVT GRLenVT = Subtarget.getGRLenVT();
4064: 
4065:   // Return the value of the return address register, marking it an implicit
4066:   // live-in.
4067:   Register Reg = MF.addLiveIn(Subtarget.getRegisterInfo()->getRARegister(),
4068:                               getRegClassFor(GRLenVT));
4069:   return DAG.getCopyFromReg(DAG.getEntryNode(), SDLoc(Op), Reg, GRLenVT);
4070: }
4071: 
4072: SDValue LoongArchTargetLowering::lowerEH_DWARF_CFA(SDValue Op,
4073:                                                    SelectionDAG &DAG) const {
4074:   MachineFunction &MF = DAG.getMachineFunction();
4075:   auto Size = Subtarget.getGRLen() / 8;
4076:   auto FI = MF.getFrameInfo().CreateFixedObject(Size, 0, false);
4077:   return DAG.getFrameIndex(FI, getPointerTy(DAG.getDataLayout()));
4078: }
4079: 
4080: SDValue LoongArchTargetLowering::lowerVASTART(SDValue Op,
4081:                                               SelectionDAG &DAG) const {
4082:   MachineFunction &MF = DAG.getMachineFunction();
4083:   auto *FuncInfo = MF.getInfo<LoongArchMachineFunctionInfo>();
4084: 
4085:   SDLoc DL(Op);
4086:   SDValue FI = DAG.getFrameIndex(FuncInfo->getVarArgsFrameIndex(),
4087:                                  getPointerTy(MF.getDataLayout()));
4088: 
4089:   // vastart just stores the address of the VarArgsFrameIndex slot into the
4090:   // memory location argument.
4091:   const Value *SV = cast<SrcValueSDNode>(Op.getOperand(2))->getValue();
4092:   return DAG.getStore(Op.getOperand(0), DL, FI, Op.getOperand(1),
4093:                       MachinePointerInfo(SV));
4094: }
4095: 
4096: SDValue LoongArchTargetLowering::lowerUINT_TO_FP(SDValue Op,
4097:                                                  SelectionDAG &DAG) const {
4098:   assert(Subtarget.is64Bit() && Subtarget.hasBasicF() &&
4099:          !Subtarget.hasBasicD() && "unexpected target features");
4100: 
4101:   SDLoc DL(Op);
4102:   SDValue Op0 = Op.getOperand(0);
4103:   if (Op0->getOpcode() == ISD::AND) {
4104:     auto *C = dyn_cast<ConstantSDNode>(Op0.getOperand(1));
4105:     if (C && C->getZExtValue() < UINT64_C(0xFFFFFFFF))
4106:       return Op;
4107:   }
4108: 
4109:   if (Op0->getOpcode() == LoongArchISD::BSTRPICK &&
4110:       Op0.getConstantOperandVal(1) < UINT64_C(0X1F) &&
4111:       Op0.getConstantOperandVal(2) == UINT64_C(0))
4112:     return Op;
4113: 
4114:   if (Op0.getOpcode() == ISD::AssertZext &&
4115:       dyn_cast<VTSDNode>(Op0.getOperand(1))->getVT().bitsLT(MVT::i32))
4116:     return Op;
4117: 
4118:   EVT OpVT = Op0.getValueType();
4119:   EVT RetVT = Op.getValueType();
4120:   RTLIB::Libcall LC = RTLIB::getUINTTOFP(OpVT, RetVT);
4121:   MakeLibCallOptions CallOptions;
4122:   CallOptions.setTypeListBeforeSoften(OpVT, RetVT);
4123:   SDValue Chain = SDValue();
4124:   SDValue Result;
4125:   std::tie(Result, Chain) =
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerRETURNADDR`, `LoongArchTargetLowering::lowerEH_DWARF_CFA`, `LoongArchTargetLowering::lowerVASTART`, `LoongArchTargetLowering::lowerUINT_TO_FP`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerRETURNADDR`, `LoongArchTargetLowering::lowerEH_DWARF_CFA`, `LoongArchTargetLowering::lowerVASTART`, `LoongArchTargetLowering::lowerUINT_TO_FP` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 4126-4200 / 第 4126-4200 行
```cpp
4126:       makeLibCall(DAG, LC, Op.getValueType(), Op0, CallOptions, DL, Chain);
4127:   return Result;
4128: }
4129: 
4130: SDValue LoongArchTargetLowering::lowerSINT_TO_FP(SDValue Op,
4131:                                                  SelectionDAG &DAG) const {
4132:   assert(Subtarget.is64Bit() && Subtarget.hasBasicF() &&
4133:          !Subtarget.hasBasicD() && "unexpected target features");
4134: 
4135:   SDLoc DL(Op);
4136:   SDValue Op0 = Op.getOperand(0);
4137: 
4138:   if ((Op0.getOpcode() == ISD::AssertSext ||
4139:        Op0.getOpcode() == ISD::SIGN_EXTEND_INREG) &&
4140:       dyn_cast<VTSDNode>(Op0.getOperand(1))->getVT().bitsLE(MVT::i32))
4141:     return Op;
4142: 
4143:   EVT OpVT = Op0.getValueType();
4144:   EVT RetVT = Op.getValueType();
4145:   RTLIB::Libcall LC = RTLIB::getSINTTOFP(OpVT, RetVT);
4146:   MakeLibCallOptions CallOptions;
4147:   CallOptions.setTypeListBeforeSoften(OpVT, RetVT);
4148:   SDValue Chain = SDValue();
4149:   SDValue Result;
4150:   std::tie(Result, Chain) =
4151:       makeLibCall(DAG, LC, Op.getValueType(), Op0, CallOptions, DL, Chain);
4152:   return Result;
4153: }
4154: 
4155: SDValue LoongArchTargetLowering::lowerBITCAST(SDValue Op,
4156:                                               SelectionDAG &DAG) const {
4157: 
4158:   SDLoc DL(Op);
4159:   EVT VT = Op.getValueType();
4160:   SDValue Op0 = Op.getOperand(0);
4161:   EVT Op0VT = Op0.getValueType();
4162: 
4163:   if (Op.getValueType() == MVT::f32 && Op0VT == MVT::i32 &&
4164:       Subtarget.is64Bit() && Subtarget.hasBasicF()) {
4165:     SDValue NewOp0 = DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, Op0);
4166:     return DAG.getNode(LoongArchISD::MOVGR2FR_W_LA64, DL, MVT::f32, NewOp0);
4167:   }
4168:   if (VT == MVT::f64 && Op0VT == MVT::i64 && !Subtarget.is64Bit()) {
4169:     SDValue Lo, Hi;
4170:     std::tie(Lo, Hi) = DAG.SplitScalar(Op0, DL, MVT::i32, MVT::i32);
4171:     return DAG.getNode(LoongArchISD::BUILD_PAIR_F64, DL, MVT::f64, Lo, Hi);
4172:   }
4173:   return Op;
4174: }
4175: 
4176: SDValue LoongArchTargetLowering::lowerFP_TO_SINT(SDValue Op,
4177:                                                  SelectionDAG &DAG) const {
4178: 
4179:   SDLoc DL(Op);
4180:   SDValue Op0 = Op.getOperand(0);
4181: 
4182:   if (Op0.getValueType() == MVT::f16)
4183:     Op0 = DAG.getNode(ISD::FP_EXTEND, DL, MVT::f32, Op0);
4184: 
4185:   if (Op.getValueSizeInBits() > 32 && Subtarget.hasBasicF() &&
4186:       !Subtarget.hasBasicD()) {
4187:     SDValue Dst = DAG.getNode(LoongArchISD::FTINT, DL, MVT::f32, Op0);
4188:     return DAG.getNode(LoongArchISD::MOVFR2GR_S_LA64, DL, MVT::i64, Dst);
4189:   }
4190: 
4191:   EVT FPTy = EVT::getFloatingPointVT(Op.getValueSizeInBits());
4192:   SDValue Trunc = DAG.getNode(LoongArchISD::FTINT, DL, FPTy, Op0);
4193:   return DAG.getNode(ISD::BITCAST, DL, Op.getValueType(), Trunc);
4194: }
4195: 
4196: static SDValue getTargetNode(GlobalAddressSDNode *N, SDLoc DL, EVT Ty,
4197:                              SelectionDAG &DAG, unsigned Flags) {
4198:   return DAG.getTargetGlobalAddress(N->getGlobal(), DL, Ty, 0, Flags);
4199: }
4200: 
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerSINT_TO_FP`, `LoongArchTargetLowering::lowerBITCAST`, `LoongArchTargetLowering::lowerFP_TO_SINT`, `getTargetNode`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerSINT_TO_FP`, `LoongArchTargetLowering::lowerBITCAST`, `LoongArchTargetLowering::lowerFP_TO_SINT`, `getTargetNode` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 4201-4275 / 第 4201-4275 行
```cpp
4201: static SDValue getTargetNode(BlockAddressSDNode *N, SDLoc DL, EVT Ty,
4202:                              SelectionDAG &DAG, unsigned Flags) {
4203:   return DAG.getTargetBlockAddress(N->getBlockAddress(), Ty, N->getOffset(),
4204:                                    Flags);
4205: }
4206: 
4207: static SDValue getTargetNode(ConstantPoolSDNode *N, SDLoc DL, EVT Ty,
4208:                              SelectionDAG &DAG, unsigned Flags) {
4209:   return DAG.getTargetConstantPool(N->getConstVal(), Ty, N->getAlign(),
4210:                                    N->getOffset(), Flags);
4211: }
4212: 
4213: static SDValue getTargetNode(JumpTableSDNode *N, SDLoc DL, EVT Ty,
4214:                              SelectionDAG &DAG, unsigned Flags) {
4215:   return DAG.getTargetJumpTable(N->getIndex(), Ty, Flags);
4216: }
4217: 
4218: template <class NodeTy>
4219: SDValue LoongArchTargetLowering::getAddr(NodeTy *N, SelectionDAG &DAG,
4220:                                          CodeModel::Model M,
4221:                                          bool IsLocal) const {
4222:   SDLoc DL(N);
4223:   EVT Ty = getPointerTy(DAG.getDataLayout());
4224:   SDValue Addr = getTargetNode(N, DL, Ty, DAG, 0);
4225:   SDValue Load;
4226: 
4227:   switch (M) {
4228:   default:
4229:     report_fatal_error("Unsupported code model");
4230: 
4231:   case CodeModel::Large: {
4232:     assert(Subtarget.is64Bit() && "Large code model requires LA64");
4233: 
4234:     // This is not actually used, but is necessary for successfully matching
4235:     // the PseudoLA_*_LARGE nodes.
4236:     SDValue Tmp = DAG.getConstant(0, DL, Ty);
4237:     if (IsLocal) {
4238:       // This generates the pattern (PseudoLA_PCREL_LARGE tmp sym), that
4239:       // eventually becomes the desired 5-insn code sequence.
4240:       Load = SDValue(DAG.getMachineNode(LoongArch::PseudoLA_PCREL_LARGE, DL, Ty,
4241:                                         Tmp, Addr),
4242:                      0);
4243:     } else {
4244:       // This generates the pattern (PseudoLA_GOT_LARGE tmp sym), that
4245:       // eventually becomes the desired 5-insn code sequence.
4246:       Load = SDValue(
4247:           DAG.getMachineNode(LoongArch::PseudoLA_GOT_LARGE, DL, Ty, Tmp, Addr),
4248:           0);
4249:     }
4250:     break;
4251:   }
4252: 
4253:   case CodeModel::Small:
4254:   case CodeModel::Medium:
4255:     if (IsLocal) {
4256:       // This generates the pattern (PseudoLA_PCREL sym), which
4257:       //
4258:       // for la32r expands to:
4259:       //   (addi.w (pcaddu12i %pcadd_hi20(sym)) %pcadd_lo12(.Lpcadd_hi)).
4260:       //
4261:       // for la32s and la64 expands to:
4262:       //   (addi.w/d (pcalau12i %pc_hi20(sym)) %pc_lo12(sym)).
4263:       Load = SDValue(
4264:           DAG.getMachineNode(LoongArch::PseudoLA_PCREL, DL, Ty, Addr), 0);
4265:     } else {
4266:       // This generates the pattern (PseudoLA_GOT sym), which
4267:       //
4268:       // for la32r expands to:
4269:       //   (ld.w (pcaddu12i %got_pcadd_hi20(sym)) %pcadd_lo12(.Lpcadd_hi)).
4270:       //
4271:       // for la32s and la64 expands to:
4272:       //   (ld.w/d (pcalau12i %got_pc_hi20(sym)) %got_pc_lo12(sym)).
4273:       Load =
4274:           SDValue(DAG.getMachineNode(LoongArch::PseudoLA_GOT, DL, Ty, Addr), 0);
4275:     }
```
- **EN**: The range implements or declares functions including `getTargetNode`, `getTargetNode`, `getTargetNode`, `LoongArchTargetLowering::getAddr`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getTargetNode`, `getTargetNode`, `getTargetNode`, `LoongArchTargetLowering::getAddr` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 4276-4350 / 第 4276-4350 行
```cpp
4276:   }
4277: 
4278:   if (!IsLocal) {
4279:     // Mark the load instruction as invariant to enable hoisting in MachineLICM.
4280:     MachineFunction &MF = DAG.getMachineFunction();
4281:     MachineMemOperand *MemOp = MF.getMachineMemOperand(
4282:         MachinePointerInfo::getGOT(MF),
4283:         MachineMemOperand::MOLoad | MachineMemOperand::MODereferenceable |
4284:             MachineMemOperand::MOInvariant,
4285:         LLT(Ty.getSimpleVT()), Align(Ty.getFixedSizeInBits() / 8));
4286:     DAG.setNodeMemRefs(cast<MachineSDNode>(Load.getNode()), {MemOp});
4287:   }
4288: 
4289:   return Load;
4290: }
4291: 
4292: SDValue LoongArchTargetLowering::lowerBlockAddress(SDValue Op,
4293:                                                    SelectionDAG &DAG) const {
4294:   return getAddr(cast<BlockAddressSDNode>(Op), DAG,
4295:                  DAG.getTarget().getCodeModel());
4296: }
4297: 
4298: SDValue LoongArchTargetLowering::lowerJumpTable(SDValue Op,
4299:                                                 SelectionDAG &DAG) const {
4300:   return getAddr(cast<JumpTableSDNode>(Op), DAG,
4301:                  DAG.getTarget().getCodeModel());
4302: }
4303: 
4304: SDValue LoongArchTargetLowering::lowerConstantPool(SDValue Op,
4305:                                                    SelectionDAG &DAG) const {
4306:   return getAddr(cast<ConstantPoolSDNode>(Op), DAG,
4307:                  DAG.getTarget().getCodeModel());
4308: }
4309: 
4310: SDValue LoongArchTargetLowering::lowerGlobalAddress(SDValue Op,
4311:                                                     SelectionDAG &DAG) const {
4312:   GlobalAddressSDNode *N = cast<GlobalAddressSDNode>(Op);
4313:   assert(N->getOffset() == 0 && "unexpected offset in global node");
4314:   auto CM = DAG.getTarget().getCodeModel();
4315:   const GlobalValue *GV = N->getGlobal();
4316: 
4317:   if (GV->isDSOLocal() && isa<GlobalVariable>(GV)) {
4318:     if (auto GCM = dyn_cast<GlobalVariable>(GV)->getCodeModel())
4319:       CM = *GCM;
4320:   }
4321: 
4322:   return getAddr(N, DAG, CM, GV->isDSOLocal());
4323: }
4324: 
4325: SDValue LoongArchTargetLowering::getStaticTLSAddr(GlobalAddressSDNode *N,
4326:                                                   SelectionDAG &DAG,
4327:                                                   unsigned Opc, bool UseGOT,
4328:                                                   bool Large) const {
4329:   SDLoc DL(N);
4330:   EVT Ty = getPointerTy(DAG.getDataLayout());
4331:   MVT GRLenVT = Subtarget.getGRLenVT();
4332: 
4333:   // This is not actually used, but is necessary for successfully matching the
4334:   // PseudoLA_*_LARGE nodes.
4335:   SDValue Tmp = DAG.getConstant(0, DL, Ty);
4336:   SDValue Addr = DAG.getTargetGlobalAddress(N->getGlobal(), DL, Ty, 0, 0);
4337: 
4338:   // Only IE needs an extra argument for large code model.
4339:   SDValue Offset = Opc == LoongArch::PseudoLA_TLS_IE_LARGE
4340:                        ? SDValue(DAG.getMachineNode(Opc, DL, Ty, Tmp, Addr), 0)
4341:                        : SDValue(DAG.getMachineNode(Opc, DL, Ty, Addr), 0);
4342: 
4343:   // If it is LE for normal/medium code model, the add tp operation will occur
4344:   // during the pseudo-instruction expansion.
4345:   if (Opc == LoongArch::PseudoLA_TLS_LE && !Large)
4346:     return Offset;
4347: 
4348:   if (UseGOT) {
4349:     // Mark the load instruction as invariant to enable hoisting in MachineLICM.
4350:     MachineFunction &MF = DAG.getMachineFunction();
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerBlockAddress`, `LoongArchTargetLowering::lowerJumpTable`, `LoongArchTargetLowering::lowerConstantPool`, `LoongArchTargetLowering::lowerGlobalAddress`, `LoongArchTargetLowering::getStaticTLSAddr`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerBlockAddress`, `LoongArchTargetLowering::lowerJumpTable`, `LoongArchTargetLowering::lowerConstantPool`, `LoongArchTargetLowering::lowerGlobalAddress`, `LoongArchTargetLowering::getStaticTLSAddr` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 4351-4425 / 第 4351-4425 行
```cpp
4351:     MachineMemOperand *MemOp = MF.getMachineMemOperand(
4352:         MachinePointerInfo::getGOT(MF),
4353:         MachineMemOperand::MOLoad | MachineMemOperand::MODereferenceable |
4354:             MachineMemOperand::MOInvariant,
4355:         LLT(Ty.getSimpleVT()), Align(Ty.getFixedSizeInBits() / 8));
4356:     DAG.setNodeMemRefs(cast<MachineSDNode>(Offset.getNode()), {MemOp});
4357:   }
4358: 
4359:   // Add the thread pointer.
4360:   return DAG.getNode(ISD::ADD, DL, Ty, Offset,
4361:                      DAG.getRegister(LoongArch::R2, GRLenVT));
4362: }
4363: 
4364: SDValue LoongArchTargetLowering::getDynamicTLSAddr(GlobalAddressSDNode *N,
4365:                                                    SelectionDAG &DAG,
4366:                                                    unsigned Opc,
4367:                                                    bool Large) const {
4368:   SDLoc DL(N);
4369:   EVT Ty = getPointerTy(DAG.getDataLayout());
4370:   IntegerType *CallTy = Type::getIntNTy(*DAG.getContext(), Ty.getSizeInBits());
4371: 
4372:   // This is not actually used, but is necessary for successfully matching the
4373:   // PseudoLA_*_LARGE nodes.
4374:   SDValue Tmp = DAG.getConstant(0, DL, Ty);
4375: 
4376:   // Use a PC-relative addressing mode to access the dynamic GOT address.
4377:   SDValue Addr = DAG.getTargetGlobalAddress(N->getGlobal(), DL, Ty, 0, 0);
4378:   SDValue Load = Large ? SDValue(DAG.getMachineNode(Opc, DL, Ty, Tmp, Addr), 0)
4379:                        : SDValue(DAG.getMachineNode(Opc, DL, Ty, Addr), 0);
4380: 
4381:   // Prepare argument list to generate call.
4382:   ArgListTy Args;
4383:   Args.emplace_back(Load, CallTy);
4384: 
4385:   // Setup call to __tls_get_addr.
4386:   TargetLowering::CallLoweringInfo CLI(DAG);
4387:   CLI.setDebugLoc(DL)
4388:       .setChain(DAG.getEntryNode())
4389:       .setLibCallee(CallingConv::C, CallTy,
4390:                     DAG.getExternalSymbol("__tls_get_addr", Ty),
4391:                     std::move(Args));
4392: 
4393:   return LowerCallTo(CLI).first;
4394: }
4395: 
4396: SDValue LoongArchTargetLowering::getTLSDescAddr(GlobalAddressSDNode *N,
4397:                                                 SelectionDAG &DAG, unsigned Opc,
4398:                                                 bool Large) const {
4399:   SDLoc DL(N);
4400:   EVT Ty = getPointerTy(DAG.getDataLayout());
4401:   const GlobalValue *GV = N->getGlobal();
4402: 
4403:   // This is not actually used, but is necessary for successfully matching the
4404:   // PseudoLA_*_LARGE nodes.
4405:   SDValue Tmp = DAG.getConstant(0, DL, Ty);
4406: 
4407:   // Use a PC-relative addressing mode to access the global dynamic GOT address.
4408:   // This generates the pattern (PseudoLA_TLS_DESC_PC{,LARGE} sym).
4409:   SDValue Addr = DAG.getTargetGlobalAddress(GV, DL, Ty, 0, 0);
4410:   return Large ? SDValue(DAG.getMachineNode(Opc, DL, Ty, Tmp, Addr), 0)
4411:                : SDValue(DAG.getMachineNode(Opc, DL, Ty, Addr), 0);
4412: }
4413: 
4414: SDValue
4415: LoongArchTargetLowering::lowerGlobalTLSAddress(SDValue Op,
4416:                                                SelectionDAG &DAG) const {
4417:   if (DAG.getMachineFunction().getFunction().getCallingConv() ==
4418:       CallingConv::GHC)
4419:     report_fatal_error("In GHC calling convention TLS is not supported");
4420: 
4421:   bool Large = DAG.getTarget().getCodeModel() == CodeModel::Large;
4422:   assert((!Large || Subtarget.is64Bit()) && "Large code model requires LA64");
4423: 
4424:   GlobalAddressSDNode *N = cast<GlobalAddressSDNode>(Op);
4425:   assert(N->getOffset() == 0 && "unexpected offset in global node");
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::getDynamicTLSAddr`, `LoongArchTargetLowering::getTLSDescAddr`, `LoongArchTargetLowering::lowerGlobalTLSAddress`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::getDynamicTLSAddr`, `LoongArchTargetLowering::getTLSDescAddr`, `LoongArchTargetLowering::lowerGlobalTLSAddress` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 4426-4500 / 第 4426-4500 行
```cpp
4426: 
4427:   if (DAG.getTarget().useEmulatedTLS())
4428:     reportFatalUsageError("the emulated TLS is prohibited");
4429: 
4430:   bool IsDesc = DAG.getTarget().useTLSDESC();
4431: 
4432:   switch (getTargetMachine().getTLSModel(N->getGlobal())) {
4433:   case TLSModel::GeneralDynamic:
4434:     // In this model, application code calls the dynamic linker function
4435:     // __tls_get_addr to locate TLS offsets into the dynamic thread vector at
4436:     // runtime.
4437:     if (!IsDesc)
4438:       return getDynamicTLSAddr(N, DAG,
4439:                                Large ? LoongArch::PseudoLA_TLS_GD_LARGE
4440:                                      : LoongArch::PseudoLA_TLS_GD,
4441:                                Large);
4442:     break;
4443:   case TLSModel::LocalDynamic:
4444:     // Same as GeneralDynamic, except for assembly modifiers and relocation
4445:     // records.
4446:     if (!IsDesc)
4447:       return getDynamicTLSAddr(N, DAG,
4448:                                Large ? LoongArch::PseudoLA_TLS_LD_LARGE
4449:                                      : LoongArch::PseudoLA_TLS_LD,
4450:                                Large);
4451:     break;
4452:   case TLSModel::InitialExec:
4453:     // This model uses the GOT to resolve TLS offsets.
4454:     return getStaticTLSAddr(N, DAG,
4455:                             Large ? LoongArch::PseudoLA_TLS_IE_LARGE
4456:                                   : LoongArch::PseudoLA_TLS_IE,
4457:                             /*UseGOT=*/true, Large);
4458:   case TLSModel::LocalExec:
4459:     // This model is used when static linking as the TLS offsets are resolved
4460:     // during program linking.
4461:     //
4462:     // This node doesn't need an extra argument for the large code model.
4463:     return getStaticTLSAddr(N, DAG, LoongArch::PseudoLA_TLS_LE,
4464:                             /*UseGOT=*/false, Large);
4465:   }
4466: 
4467:   return getTLSDescAddr(N, DAG,
4468:                         Large ? LoongArch::PseudoLA_TLS_DESC_LARGE
4469:                               : LoongArch::PseudoLA_TLS_DESC,
4470:                         Large);
4471: }
4472: 
4473: template <unsigned N>
4474: static SDValue checkIntrinsicImmArg(SDValue Op, unsigned ImmOp,
4475:                                     SelectionDAG &DAG, bool IsSigned = false) {
4476:   auto *CImm = cast<ConstantSDNode>(Op->getOperand(ImmOp));
4477:   // Check the ImmArg.
4478:   if ((IsSigned && !isInt<N>(CImm->getSExtValue())) ||
4479:       (!IsSigned && !isUInt<N>(CImm->getZExtValue()))) {
4480:     DAG.getContext()->emitError(Op->getOperationName(0) +
4481:                                 ": argument out of range.");
4482:     return DAG.getNode(ISD::UNDEF, SDLoc(Op), Op.getValueType());
4483:   }
4484:   return SDValue();
4485: }
4486: 
4487: SDValue
4488: LoongArchTargetLowering::lowerINTRINSIC_WO_CHAIN(SDValue Op,
4489:                                                  SelectionDAG &DAG) const {
4490:   switch (Op.getConstantOperandVal(0)) {
4491:   default:
4492:     return SDValue(); // Don't custom lower most intrinsics.
4493:   case Intrinsic::thread_pointer: {
4494:     EVT PtrVT = getPointerTy(DAG.getDataLayout());
4495:     return DAG.getRegister(LoongArch::R2, PtrVT);
4496:   }
4497:   case Intrinsic::loongarch_lsx_vpickve2gr_d:
4498:   case Intrinsic::loongarch_lsx_vpickve2gr_du:
4499:   case Intrinsic::loongarch_lsx_vreplvei_d:
4500:   case Intrinsic::loongarch_lasx_xvrepl128vei_d:
```
- **EN**: The range implements or declares functions including `checkIntrinsicImmArg`, `LoongArchTargetLowering::lowerINTRINSIC_WO_CHAIN`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `checkIntrinsicImmArg`, `LoongArchTargetLowering::lowerINTRINSIC_WO_CHAIN` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 4501-4575 / 第 4501-4575 行
```cpp
4501:     return checkIntrinsicImmArg<1>(Op, 2, DAG);
4502:   case Intrinsic::loongarch_lsx_vreplvei_w:
4503:   case Intrinsic::loongarch_lasx_xvrepl128vei_w:
4504:   case Intrinsic::loongarch_lasx_xvpickve2gr_d:
4505:   case Intrinsic::loongarch_lasx_xvpickve2gr_du:
4506:   case Intrinsic::loongarch_lasx_xvpickve_d:
4507:   case Intrinsic::loongarch_lasx_xvpickve_d_f:
4508:     return checkIntrinsicImmArg<2>(Op, 2, DAG);
4509:   case Intrinsic::loongarch_lasx_xvinsve0_d:
4510:     return checkIntrinsicImmArg<2>(Op, 3, DAG);
4511:   case Intrinsic::loongarch_lsx_vsat_b:
4512:   case Intrinsic::loongarch_lsx_vsat_bu:
4513:   case Intrinsic::loongarch_lsx_vrotri_b:
4514:   case Intrinsic::loongarch_lsx_vsllwil_h_b:
4515:   case Intrinsic::loongarch_lsx_vsllwil_hu_bu:
4516:   case Intrinsic::loongarch_lsx_vsrlri_b:
4517:   case Intrinsic::loongarch_lsx_vsrari_b:
4518:   case Intrinsic::loongarch_lsx_vreplvei_h:
4519:   case Intrinsic::loongarch_lasx_xvsat_b:
4520:   case Intrinsic::loongarch_lasx_xvsat_bu:
4521:   case Intrinsic::loongarch_lasx_xvrotri_b:
4522:   case Intrinsic::loongarch_lasx_xvsllwil_h_b:
4523:   case Intrinsic::loongarch_lasx_xvsllwil_hu_bu:
4524:   case Intrinsic::loongarch_lasx_xvsrlri_b:
4525:   case Intrinsic::loongarch_lasx_xvsrari_b:
4526:   case Intrinsic::loongarch_lasx_xvrepl128vei_h:
4527:   case Intrinsic::loongarch_lasx_xvpickve_w:
4528:   case Intrinsic::loongarch_lasx_xvpickve_w_f:
4529:     return checkIntrinsicImmArg<3>(Op, 2, DAG);
4530:   case Intrinsic::loongarch_lasx_xvinsve0_w:
4531:     return checkIntrinsicImmArg<3>(Op, 3, DAG);
4532:   case Intrinsic::loongarch_lsx_vsat_h:
4533:   case Intrinsic::loongarch_lsx_vsat_hu:
4534:   case Intrinsic::loongarch_lsx_vrotri_h:
4535:   case Intrinsic::loongarch_lsx_vsllwil_w_h:
4536:   case Intrinsic::loongarch_lsx_vsllwil_wu_hu:
4537:   case Intrinsic::loongarch_lsx_vsrlri_h:
4538:   case Intrinsic::loongarch_lsx_vsrari_h:
4539:   case Intrinsic::loongarch_lsx_vreplvei_b:
4540:   case Intrinsic::loongarch_lasx_xvsat_h:
4541:   case Intrinsic::loongarch_lasx_xvsat_hu:
4542:   case Intrinsic::loongarch_lasx_xvrotri_h:
4543:   case Intrinsic::loongarch_lasx_xvsllwil_w_h:
4544:   case Intrinsic::loongarch_lasx_xvsllwil_wu_hu:
4545:   case Intrinsic::loongarch_lasx_xvsrlri_h:
4546:   case Intrinsic::loongarch_lasx_xvsrari_h:
4547:   case Intrinsic::loongarch_lasx_xvrepl128vei_b:
4548:     return checkIntrinsicImmArg<4>(Op, 2, DAG);
4549:   case Intrinsic::loongarch_lsx_vsrlni_b_h:
4550:   case Intrinsic::loongarch_lsx_vsrani_b_h:
4551:   case Intrinsic::loongarch_lsx_vsrlrni_b_h:
4552:   case Intrinsic::loongarch_lsx_vsrarni_b_h:
4553:   case Intrinsic::loongarch_lsx_vssrlni_b_h:
4554:   case Intrinsic::loongarch_lsx_vssrani_b_h:
4555:   case Intrinsic::loongarch_lsx_vssrlni_bu_h:
4556:   case Intrinsic::loongarch_lsx_vssrani_bu_h:
4557:   case Intrinsic::loongarch_lsx_vssrlrni_b_h:
4558:   case Intrinsic::loongarch_lsx_vssrarni_b_h:
4559:   case Intrinsic::loongarch_lsx_vssrlrni_bu_h:
4560:   case Intrinsic::loongarch_lsx_vssrarni_bu_h:
4561:   case Intrinsic::loongarch_lasx_xvsrlni_b_h:
4562:   case Intrinsic::loongarch_lasx_xvsrani_b_h:
4563:   case Intrinsic::loongarch_lasx_xvsrlrni_b_h:
4564:   case Intrinsic::loongarch_lasx_xvsrarni_b_h:
4565:   case Intrinsic::loongarch_lasx_xvssrlni_b_h:
4566:   case Intrinsic::loongarch_lasx_xvssrani_b_h:
4567:   case Intrinsic::loongarch_lasx_xvssrlni_bu_h:
4568:   case Intrinsic::loongarch_lasx_xvssrani_bu_h:
4569:   case Intrinsic::loongarch_lasx_xvssrlrni_b_h:
4570:   case Intrinsic::loongarch_lasx_xvssrarni_b_h:
4571:   case Intrinsic::loongarch_lasx_xvssrlrni_bu_h:
4572:   case Intrinsic::loongarch_lasx_xvssrarni_bu_h:
4573:     return checkIntrinsicImmArg<4>(Op, 3, DAG);
4574:   case Intrinsic::loongarch_lsx_vsat_w:
4575:   case Intrinsic::loongarch_lsx_vsat_wu:
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 4576-4650 / 第 4576-4650 行
```cpp
4576:   case Intrinsic::loongarch_lsx_vrotri_w:
4577:   case Intrinsic::loongarch_lsx_vsllwil_d_w:
4578:   case Intrinsic::loongarch_lsx_vsllwil_du_wu:
4579:   case Intrinsic::loongarch_lsx_vsrlri_w:
4580:   case Intrinsic::loongarch_lsx_vsrari_w:
4581:   case Intrinsic::loongarch_lsx_vslei_bu:
4582:   case Intrinsic::loongarch_lsx_vslei_hu:
4583:   case Intrinsic::loongarch_lsx_vslei_wu:
4584:   case Intrinsic::loongarch_lsx_vslei_du:
4585:   case Intrinsic::loongarch_lsx_vslti_bu:
4586:   case Intrinsic::loongarch_lsx_vslti_hu:
4587:   case Intrinsic::loongarch_lsx_vslti_wu:
4588:   case Intrinsic::loongarch_lsx_vslti_du:
4589:   case Intrinsic::loongarch_lsx_vbsll_v:
4590:   case Intrinsic::loongarch_lsx_vbsrl_v:
4591:   case Intrinsic::loongarch_lasx_xvsat_w:
4592:   case Intrinsic::loongarch_lasx_xvsat_wu:
4593:   case Intrinsic::loongarch_lasx_xvrotri_w:
4594:   case Intrinsic::loongarch_lasx_xvsllwil_d_w:
4595:   case Intrinsic::loongarch_lasx_xvsllwil_du_wu:
4596:   case Intrinsic::loongarch_lasx_xvsrlri_w:
4597:   case Intrinsic::loongarch_lasx_xvsrari_w:
4598:   case Intrinsic::loongarch_lasx_xvslei_bu:
4599:   case Intrinsic::loongarch_lasx_xvslei_hu:
4600:   case Intrinsic::loongarch_lasx_xvslei_wu:
4601:   case Intrinsic::loongarch_lasx_xvslei_du:
4602:   case Intrinsic::loongarch_lasx_xvslti_bu:
4603:   case Intrinsic::loongarch_lasx_xvslti_hu:
4604:   case Intrinsic::loongarch_lasx_xvslti_wu:
4605:   case Intrinsic::loongarch_lasx_xvslti_du:
4606:   case Intrinsic::loongarch_lasx_xvbsll_v:
4607:   case Intrinsic::loongarch_lasx_xvbsrl_v:
4608:     return checkIntrinsicImmArg<5>(Op, 2, DAG);
4609:   case Intrinsic::loongarch_lsx_vseqi_b:
4610:   case Intrinsic::loongarch_lsx_vseqi_h:
4611:   case Intrinsic::loongarch_lsx_vseqi_w:
4612:   case Intrinsic::loongarch_lsx_vseqi_d:
4613:   case Intrinsic::loongarch_lsx_vslei_b:
4614:   case Intrinsic::loongarch_lsx_vslei_h:
4615:   case Intrinsic::loongarch_lsx_vslei_w:
4616:   case Intrinsic::loongarch_lsx_vslei_d:
4617:   case Intrinsic::loongarch_lsx_vslti_b:
4618:   case Intrinsic::loongarch_lsx_vslti_h:
4619:   case Intrinsic::loongarch_lsx_vslti_w:
4620:   case Intrinsic::loongarch_lsx_vslti_d:
4621:   case Intrinsic::loongarch_lasx_xvseqi_b:
4622:   case Intrinsic::loongarch_lasx_xvseqi_h:
4623:   case Intrinsic::loongarch_lasx_xvseqi_w:
4624:   case Intrinsic::loongarch_lasx_xvseqi_d:
4625:   case Intrinsic::loongarch_lasx_xvslei_b:
4626:   case Intrinsic::loongarch_lasx_xvslei_h:
4627:   case Intrinsic::loongarch_lasx_xvslei_w:
4628:   case Intrinsic::loongarch_lasx_xvslei_d:
4629:   case Intrinsic::loongarch_lasx_xvslti_b:
4630:   case Intrinsic::loongarch_lasx_xvslti_h:
4631:   case Intrinsic::loongarch_lasx_xvslti_w:
4632:   case Intrinsic::loongarch_lasx_xvslti_d:
4633:     return checkIntrinsicImmArg<5>(Op, 2, DAG, /*IsSigned=*/true);
4634:   case Intrinsic::loongarch_lsx_vsrlni_h_w:
4635:   case Intrinsic::loongarch_lsx_vsrani_h_w:
4636:   case Intrinsic::loongarch_lsx_vsrlrni_h_w:
4637:   case Intrinsic::loongarch_lsx_vsrarni_h_w:
4638:   case Intrinsic::loongarch_lsx_vssrlni_h_w:
4639:   case Intrinsic::loongarch_lsx_vssrani_h_w:
4640:   case Intrinsic::loongarch_lsx_vssrlni_hu_w:
4641:   case Intrinsic::loongarch_lsx_vssrani_hu_w:
4642:   case Intrinsic::loongarch_lsx_vssrlrni_h_w:
4643:   case Intrinsic::loongarch_lsx_vssrarni_h_w:
4644:   case Intrinsic::loongarch_lsx_vssrlrni_hu_w:
4645:   case Intrinsic::loongarch_lsx_vssrarni_hu_w:
4646:   case Intrinsic::loongarch_lsx_vfrstpi_b:
4647:   case Intrinsic::loongarch_lsx_vfrstpi_h:
4648:   case Intrinsic::loongarch_lasx_xvsrlni_h_w:
4649:   case Intrinsic::loongarch_lasx_xvsrani_h_w:
4650:   case Intrinsic::loongarch_lasx_xvsrlrni_h_w:
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 4651-4725 / 第 4651-4725 行
```cpp
4651:   case Intrinsic::loongarch_lasx_xvsrarni_h_w:
4652:   case Intrinsic::loongarch_lasx_xvssrlni_h_w:
4653:   case Intrinsic::loongarch_lasx_xvssrani_h_w:
4654:   case Intrinsic::loongarch_lasx_xvssrlni_hu_w:
4655:   case Intrinsic::loongarch_lasx_xvssrani_hu_w:
4656:   case Intrinsic::loongarch_lasx_xvssrlrni_h_w:
4657:   case Intrinsic::loongarch_lasx_xvssrarni_h_w:
4658:   case Intrinsic::loongarch_lasx_xvssrlrni_hu_w:
4659:   case Intrinsic::loongarch_lasx_xvssrarni_hu_w:
4660:   case Intrinsic::loongarch_lasx_xvfrstpi_b:
4661:   case Intrinsic::loongarch_lasx_xvfrstpi_h:
4662:     return checkIntrinsicImmArg<5>(Op, 3, DAG);
4663:   case Intrinsic::loongarch_lsx_vsat_d:
4664:   case Intrinsic::loongarch_lsx_vsat_du:
4665:   case Intrinsic::loongarch_lsx_vrotri_d:
4666:   case Intrinsic::loongarch_lsx_vsrlri_d:
4667:   case Intrinsic::loongarch_lsx_vsrari_d:
4668:   case Intrinsic::loongarch_lasx_xvsat_d:
4669:   case Intrinsic::loongarch_lasx_xvsat_du:
4670:   case Intrinsic::loongarch_lasx_xvrotri_d:
4671:   case Intrinsic::loongarch_lasx_xvsrlri_d:
4672:   case Intrinsic::loongarch_lasx_xvsrari_d:
4673:     return checkIntrinsicImmArg<6>(Op, 2, DAG);
4674:   case Intrinsic::loongarch_lsx_vsrlni_w_d:
4675:   case Intrinsic::loongarch_lsx_vsrani_w_d:
4676:   case Intrinsic::loongarch_lsx_vsrlrni_w_d:
4677:   case Intrinsic::loongarch_lsx_vsrarni_w_d:
4678:   case Intrinsic::loongarch_lsx_vssrlni_w_d:
4679:   case Intrinsic::loongarch_lsx_vssrani_w_d:
4680:   case Intrinsic::loongarch_lsx_vssrlni_wu_d:
4681:   case Intrinsic::loongarch_lsx_vssrani_wu_d:
4682:   case Intrinsic::loongarch_lsx_vssrlrni_w_d:
4683:   case Intrinsic::loongarch_lsx_vssrarni_w_d:
4684:   case Intrinsic::loongarch_lsx_vssrlrni_wu_d:
4685:   case Intrinsic::loongarch_lsx_vssrarni_wu_d:
4686:   case Intrinsic::loongarch_lasx_xvsrlni_w_d:
4687:   case Intrinsic::loongarch_lasx_xvsrani_w_d:
4688:   case Intrinsic::loongarch_lasx_xvsrlrni_w_d:
4689:   case Intrinsic::loongarch_lasx_xvsrarni_w_d:
4690:   case Intrinsic::loongarch_lasx_xvssrlni_w_d:
4691:   case Intrinsic::loongarch_lasx_xvssrani_w_d:
4692:   case Intrinsic::loongarch_lasx_xvssrlni_wu_d:
4693:   case Intrinsic::loongarch_lasx_xvssrani_wu_d:
4694:   case Intrinsic::loongarch_lasx_xvssrlrni_w_d:
4695:   case Intrinsic::loongarch_lasx_xvssrarni_w_d:
4696:   case Intrinsic::loongarch_lasx_xvssrlrni_wu_d:
4697:   case Intrinsic::loongarch_lasx_xvssrarni_wu_d:
4698:     return checkIntrinsicImmArg<6>(Op, 3, DAG);
4699:   case Intrinsic::loongarch_lsx_vsrlni_d_q:
4700:   case Intrinsic::loongarch_lsx_vsrani_d_q:
4701:   case Intrinsic::loongarch_lsx_vsrlrni_d_q:
4702:   case Intrinsic::loongarch_lsx_vsrarni_d_q:
4703:   case Intrinsic::loongarch_lsx_vssrlni_d_q:
4704:   case Intrinsic::loongarch_lsx_vssrani_d_q:
4705:   case Intrinsic::loongarch_lsx_vssrlni_du_q:
4706:   case Intrinsic::loongarch_lsx_vssrani_du_q:
4707:   case Intrinsic::loongarch_lsx_vssrlrni_d_q:
4708:   case Intrinsic::loongarch_lsx_vssrarni_d_q:
4709:   case Intrinsic::loongarch_lsx_vssrlrni_du_q:
4710:   case Intrinsic::loongarch_lsx_vssrarni_du_q:
4711:   case Intrinsic::loongarch_lasx_xvsrlni_d_q:
4712:   case Intrinsic::loongarch_lasx_xvsrani_d_q:
4713:   case Intrinsic::loongarch_lasx_xvsrlrni_d_q:
4714:   case Intrinsic::loongarch_lasx_xvsrarni_d_q:
4715:   case Intrinsic::loongarch_lasx_xvssrlni_d_q:
4716:   case Intrinsic::loongarch_lasx_xvssrani_d_q:
4717:   case Intrinsic::loongarch_lasx_xvssrlni_du_q:
4718:   case Intrinsic::loongarch_lasx_xvssrani_du_q:
4719:   case Intrinsic::loongarch_lasx_xvssrlrni_d_q:
4720:   case Intrinsic::loongarch_lasx_xvssrarni_d_q:
4721:   case Intrinsic::loongarch_lasx_xvssrlrni_du_q:
4722:   case Intrinsic::loongarch_lasx_xvssrarni_du_q:
4723:     return checkIntrinsicImmArg<7>(Op, 3, DAG);
4724:   case Intrinsic::loongarch_lsx_vnori_b:
4725:   case Intrinsic::loongarch_lsx_vshuf4i_b:
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 4726-4800 / 第 4726-4800 行
```cpp
4726:   case Intrinsic::loongarch_lsx_vshuf4i_h:
4727:   case Intrinsic::loongarch_lsx_vshuf4i_w:
4728:   case Intrinsic::loongarch_lasx_xvnori_b:
4729:   case Intrinsic::loongarch_lasx_xvshuf4i_b:
4730:   case Intrinsic::loongarch_lasx_xvshuf4i_h:
4731:   case Intrinsic::loongarch_lasx_xvshuf4i_w:
4732:   case Intrinsic::loongarch_lasx_xvpermi_d:
4733:     return checkIntrinsicImmArg<8>(Op, 2, DAG);
4734:   case Intrinsic::loongarch_lsx_vshuf4i_d:
4735:   case Intrinsic::loongarch_lsx_vpermi_w:
4736:   case Intrinsic::loongarch_lsx_vbitseli_b:
4737:   case Intrinsic::loongarch_lsx_vextrins_b:
4738:   case Intrinsic::loongarch_lsx_vextrins_h:
4739:   case Intrinsic::loongarch_lsx_vextrins_w:
4740:   case Intrinsic::loongarch_lsx_vextrins_d:
4741:   case Intrinsic::loongarch_lasx_xvshuf4i_d:
4742:   case Intrinsic::loongarch_lasx_xvpermi_w:
4743:   case Intrinsic::loongarch_lasx_xvpermi_q:
4744:   case Intrinsic::loongarch_lasx_xvbitseli_b:
4745:   case Intrinsic::loongarch_lasx_xvextrins_b:
4746:   case Intrinsic::loongarch_lasx_xvextrins_h:
4747:   case Intrinsic::loongarch_lasx_xvextrins_w:
4748:   case Intrinsic::loongarch_lasx_xvextrins_d:
4749:     return checkIntrinsicImmArg<8>(Op, 3, DAG);
4750:   case Intrinsic::loongarch_lsx_vrepli_b:
4751:   case Intrinsic::loongarch_lsx_vrepli_h:
4752:   case Intrinsic::loongarch_lsx_vrepli_w:
4753:   case Intrinsic::loongarch_lsx_vrepli_d:
4754:   case Intrinsic::loongarch_lasx_xvrepli_b:
4755:   case Intrinsic::loongarch_lasx_xvrepli_h:
4756:   case Intrinsic::loongarch_lasx_xvrepli_w:
4757:   case Intrinsic::loongarch_lasx_xvrepli_d:
4758:     return checkIntrinsicImmArg<10>(Op, 1, DAG, /*IsSigned=*/true);
4759:   case Intrinsic::loongarch_lsx_vldi:
4760:   case Intrinsic::loongarch_lasx_xvldi:
4761:     return checkIntrinsicImmArg<13>(Op, 1, DAG, /*IsSigned=*/true);
4762:   }
4763: }
4764: 
4765: // Helper function that emits error message for intrinsics with chain and return
4766: // merge values of a UNDEF and the chain.
4767: static SDValue emitIntrinsicWithChainErrorMessage(SDValue Op,
4768:                                                   StringRef ErrorMsg,
4769:                                                   SelectionDAG &DAG) {
4770:   DAG.getContext()->emitError(Op->getOperationName(0) + ": " + ErrorMsg + ".");
4771:   return DAG.getMergeValues({DAG.getUNDEF(Op.getValueType()), Op.getOperand(0)},
4772:                             SDLoc(Op));
4773: }
4774: 
4775: SDValue
4776: LoongArchTargetLowering::lowerINTRINSIC_W_CHAIN(SDValue Op,
4777:                                                 SelectionDAG &DAG) const {
4778:   SDLoc DL(Op);
4779:   MVT GRLenVT = Subtarget.getGRLenVT();
4780:   EVT VT = Op.getValueType();
4781:   SDValue Chain = Op.getOperand(0);
4782:   const StringRef ErrorMsgOOR = "argument out of range";
4783:   const StringRef ErrorMsgReqLA64 = "requires loongarch64";
4784:   const StringRef ErrorMsgReqF = "requires basic 'f' target feature";
4785: 
4786:   switch (Op.getConstantOperandVal(1)) {
4787:   default:
4788:     return Op;
4789:   case Intrinsic::loongarch_crc_w_b_w:
4790:   case Intrinsic::loongarch_crc_w_h_w:
4791:   case Intrinsic::loongarch_crc_w_w_w:
4792:   case Intrinsic::loongarch_crc_w_d_w:
4793:   case Intrinsic::loongarch_crcc_w_b_w:
4794:   case Intrinsic::loongarch_crcc_w_h_w:
4795:   case Intrinsic::loongarch_crcc_w_w_w:
4796:   case Intrinsic::loongarch_crcc_w_d_w:
4797:     return emitIntrinsicWithChainErrorMessage(Op, ErrorMsgReqLA64, DAG);
4798:   case Intrinsic::loongarch_csrrd_w:
4799:   case Intrinsic::loongarch_csrrd_d: {
4800:     unsigned Imm = Op.getConstantOperandVal(2);
```
- **EN**: The range implements or declares functions including `emitIntrinsicWithChainErrorMessage`, `LoongArchTargetLowering::lowerINTRINSIC_W_CHAIN`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这一段实现或声明了 `emitIntrinsicWithChainErrorMessage`, `LoongArchTargetLowering::lowerINTRINSIC_W_CHAIN` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 4801-4875 / 第 4801-4875 行
```cpp
4801:     return !isUInt<14>(Imm)
4802:                ? emitIntrinsicWithChainErrorMessage(Op, ErrorMsgOOR, DAG)
4803:                : DAG.getNode(LoongArchISD::CSRRD, DL, {GRLenVT, MVT::Other},
4804:                              {Chain, DAG.getConstant(Imm, DL, GRLenVT)});
4805:   }
4806:   case Intrinsic::loongarch_csrwr_w:
4807:   case Intrinsic::loongarch_csrwr_d: {
4808:     unsigned Imm = Op.getConstantOperandVal(3);
4809:     return !isUInt<14>(Imm)
4810:                ? emitIntrinsicWithChainErrorMessage(Op, ErrorMsgOOR, DAG)
4811:                : DAG.getNode(LoongArchISD::CSRWR, DL, {GRLenVT, MVT::Other},
4812:                              {Chain, Op.getOperand(2),
4813:                               DAG.getConstant(Imm, DL, GRLenVT)});
4814:   }
4815:   case Intrinsic::loongarch_csrxchg_w:
4816:   case Intrinsic::loongarch_csrxchg_d: {
4817:     unsigned Imm = Op.getConstantOperandVal(4);
4818:     return !isUInt<14>(Imm)
4819:                ? emitIntrinsicWithChainErrorMessage(Op, ErrorMsgOOR, DAG)
4820:                : DAG.getNode(LoongArchISD::CSRXCHG, DL, {GRLenVT, MVT::Other},
4821:                              {Chain, Op.getOperand(2), Op.getOperand(3),
4822:                               DAG.getConstant(Imm, DL, GRLenVT)});
4823:   }
4824:   case Intrinsic::loongarch_iocsrrd_d: {
4825:     return DAG.getNode(
4826:         LoongArchISD::IOCSRRD_D, DL, {GRLenVT, MVT::Other},
4827:         {Chain, DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, Op.getOperand(2))});
4828:   }
4829: #define IOCSRRD_CASE(NAME, NODE)                                               \
4830:   case Intrinsic::loongarch_##NAME: {                                          \
4831:     return DAG.getNode(LoongArchISD::NODE, DL, {GRLenVT, MVT::Other},          \
4832:                        {Chain, Op.getOperand(2)});                             \
4833:   }
4834:     IOCSRRD_CASE(iocsrrd_b, IOCSRRD_B);
4835:     IOCSRRD_CASE(iocsrrd_h, IOCSRRD_H);
4836:     IOCSRRD_CASE(iocsrrd_w, IOCSRRD_W);
4837: #undef IOCSRRD_CASE
4838:   case Intrinsic::loongarch_cpucfg: {
4839:     return DAG.getNode(LoongArchISD::CPUCFG, DL, {GRLenVT, MVT::Other},
4840:                        {Chain, Op.getOperand(2)});
4841:   }
4842:   case Intrinsic::loongarch_lddir_d: {
4843:     unsigned Imm = Op.getConstantOperandVal(3);
4844:     return !isUInt<8>(Imm)
4845:                ? emitIntrinsicWithChainErrorMessage(Op, ErrorMsgOOR, DAG)
4846:                : Op;
4847:   }
4848:   case Intrinsic::loongarch_movfcsr2gr: {
4849:     if (!Subtarget.hasBasicF())
4850:       return emitIntrinsicWithChainErrorMessage(Op, ErrorMsgReqF, DAG);
4851:     unsigned Imm = Op.getConstantOperandVal(2);
4852:     return !isUInt<2>(Imm)
4853:                ? emitIntrinsicWithChainErrorMessage(Op, ErrorMsgOOR, DAG)
4854:                : DAG.getNode(LoongArchISD::MOVFCSR2GR, DL, {VT, MVT::Other},
4855:                              {Chain, DAG.getConstant(Imm, DL, GRLenVT)});
4856:   }
4857:   case Intrinsic::loongarch_lsx_vld:
4858:   case Intrinsic::loongarch_lsx_vldrepl_b:
4859:   case Intrinsic::loongarch_lasx_xvld:
4860:   case Intrinsic::loongarch_lasx_xvldrepl_b:
4861:     return !isInt<12>(cast<ConstantSDNode>(Op.getOperand(3))->getSExtValue())
4862:                ? emitIntrinsicWithChainErrorMessage(Op, ErrorMsgOOR, DAG)
4863:                : SDValue();
4864:   case Intrinsic::loongarch_lsx_vldrepl_h:
4865:   case Intrinsic::loongarch_lasx_xvldrepl_h:
4866:     return !isShiftedInt<11, 1>(
4867:                cast<ConstantSDNode>(Op.getOperand(3))->getSExtValue())
4868:                ? emitIntrinsicWithChainErrorMessage(
4869:                      Op, "argument out of range or not a multiple of 2", DAG)
4870:                : SDValue();
4871:   case Intrinsic::loongarch_lsx_vldrepl_w:
4872:   case Intrinsic::loongarch_lasx_xvldrepl_w:
4873:     return !isShiftedInt<10, 2>(
4874:                cast<ConstantSDNode>(Op.getOperand(3))->getSExtValue())
4875:                ? emitIntrinsicWithChainErrorMessage(
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 4876-4950 / 第 4876-4950 行
```cpp
4876:                      Op, "argument out of range or not a multiple of 4", DAG)
4877:                : SDValue();
4878:   case Intrinsic::loongarch_lsx_vldrepl_d:
4879:   case Intrinsic::loongarch_lasx_xvldrepl_d:
4880:     return !isShiftedInt<9, 3>(
4881:                cast<ConstantSDNode>(Op.getOperand(3))->getSExtValue())
4882:                ? emitIntrinsicWithChainErrorMessage(
4883:                      Op, "argument out of range or not a multiple of 8", DAG)
4884:                : SDValue();
4885:   }
4886: }
4887: 
4888: // Helper function that emits error message for intrinsics with void return
4889: // value and return the chain.
4890: static SDValue emitIntrinsicErrorMessage(SDValue Op, StringRef ErrorMsg,
4891:                                          SelectionDAG &DAG) {
4892: 
4893:   DAG.getContext()->emitError(Op->getOperationName(0) + ": " + ErrorMsg + ".");
4894:   return Op.getOperand(0);
4895: }
4896: 
4897: SDValue LoongArchTargetLowering::lowerINTRINSIC_VOID(SDValue Op,
4898:                                                      SelectionDAG &DAG) const {
4899:   SDLoc DL(Op);
4900:   MVT GRLenVT = Subtarget.getGRLenVT();
4901:   SDValue Chain = Op.getOperand(0);
4902:   uint64_t IntrinsicEnum = Op.getConstantOperandVal(1);
4903:   SDValue Op2 = Op.getOperand(2);
4904:   const StringRef ErrorMsgOOR = "argument out of range";
4905:   const StringRef ErrorMsgReqLA64 = "requires loongarch64";
4906:   const StringRef ErrorMsgReqLA32 = "requires loongarch32";
4907:   const StringRef ErrorMsgReqF = "requires basic 'f' target feature";
4908: 
4909:   switch (IntrinsicEnum) {
4910:   default:
4911:     // TODO: Add more Intrinsics.
4912:     return SDValue();
4913:   case Intrinsic::loongarch_cacop_d:
4914:   case Intrinsic::loongarch_cacop_w: {
4915:     if (IntrinsicEnum == Intrinsic::loongarch_cacop_d && !Subtarget.is64Bit())
4916:       return emitIntrinsicErrorMessage(Op, ErrorMsgReqLA64, DAG);
4917:     if (IntrinsicEnum == Intrinsic::loongarch_cacop_w && Subtarget.is64Bit())
4918:       return emitIntrinsicErrorMessage(Op, ErrorMsgReqLA32, DAG);
4919:     // call void @llvm.loongarch.cacop.[d/w](uimm5, rj, simm12)
4920:     unsigned Imm1 = Op2->getAsZExtVal();
4921:     int Imm2 = cast<ConstantSDNode>(Op.getOperand(4))->getSExtValue();
4922:     if (!isUInt<5>(Imm1) || !isInt<12>(Imm2))
4923:       return emitIntrinsicErrorMessage(Op, ErrorMsgOOR, DAG);
4924:     return Op;
4925:   }
4926:   case Intrinsic::loongarch_dbar: {
4927:     unsigned Imm = Op2->getAsZExtVal();
4928:     return !isUInt<15>(Imm)
4929:                ? emitIntrinsicErrorMessage(Op, ErrorMsgOOR, DAG)
4930:                : DAG.getNode(LoongArchISD::DBAR, DL, MVT::Other, Chain,
4931:                              DAG.getConstant(Imm, DL, GRLenVT));
4932:   }
4933:   case Intrinsic::loongarch_ibar: {
4934:     unsigned Imm = Op2->getAsZExtVal();
4935:     return !isUInt<15>(Imm)
4936:                ? emitIntrinsicErrorMessage(Op, ErrorMsgOOR, DAG)
4937:                : DAG.getNode(LoongArchISD::IBAR, DL, MVT::Other, Chain,
4938:                              DAG.getConstant(Imm, DL, GRLenVT));
4939:   }
4940:   case Intrinsic::loongarch_break: {
4941:     unsigned Imm = Op2->getAsZExtVal();
4942:     return !isUInt<15>(Imm)
4943:                ? emitIntrinsicErrorMessage(Op, ErrorMsgOOR, DAG)
4944:                : DAG.getNode(LoongArchISD::BREAK, DL, MVT::Other, Chain,
4945:                              DAG.getConstant(Imm, DL, GRLenVT));
4946:   }
4947:   case Intrinsic::loongarch_movgr2fcsr: {
4948:     if (!Subtarget.hasBasicF())
4949:       return emitIntrinsicErrorMessage(Op, ErrorMsgReqF, DAG);
4950:     unsigned Imm = Op2->getAsZExtVal();
```
- **EN**: The range implements or declares functions including `emitIntrinsicErrorMessage`, `LoongArchTargetLowering::lowerINTRINSIC_VOID`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `emitIntrinsicErrorMessage`, `LoongArchTargetLowering::lowerINTRINSIC_VOID` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 4951-5025 / 第 4951-5025 行
```cpp
4951:     return !isUInt<2>(Imm)
4952:                ? emitIntrinsicErrorMessage(Op, ErrorMsgOOR, DAG)
4953:                : DAG.getNode(LoongArchISD::MOVGR2FCSR, DL, MVT::Other, Chain,
4954:                              DAG.getConstant(Imm, DL, GRLenVT),
4955:                              DAG.getNode(ISD::ANY_EXTEND, DL, GRLenVT,
4956:                                          Op.getOperand(3)));
4957:   }
4958:   case Intrinsic::loongarch_syscall: {
4959:     unsigned Imm = Op2->getAsZExtVal();
4960:     return !isUInt<15>(Imm)
4961:                ? emitIntrinsicErrorMessage(Op, ErrorMsgOOR, DAG)
4962:                : DAG.getNode(LoongArchISD::SYSCALL, DL, MVT::Other, Chain,
4963:                              DAG.getConstant(Imm, DL, GRLenVT));
4964:   }
4965: #define IOCSRWR_CASE(NAME, NODE)                                               \
4966:   case Intrinsic::loongarch_##NAME: {                                          \
4967:     SDValue Op3 = Op.getOperand(3);                                            \
4968:     return Subtarget.is64Bit()                                                 \
4969:                ? DAG.getNode(LoongArchISD::NODE, DL, MVT::Other, Chain,        \
4970:                              DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, Op2),  \
4971:                              DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, Op3))  \
4972:                : DAG.getNode(LoongArchISD::NODE, DL, MVT::Other, Chain, Op2,   \
4973:                              Op3);                                             \
4974:   }
4975:     IOCSRWR_CASE(iocsrwr_b, IOCSRWR_B);
4976:     IOCSRWR_CASE(iocsrwr_h, IOCSRWR_H);
4977:     IOCSRWR_CASE(iocsrwr_w, IOCSRWR_W);
4978: #undef IOCSRWR_CASE
4979:   case Intrinsic::loongarch_iocsrwr_d: {
4980:     return !Subtarget.is64Bit()
4981:                ? emitIntrinsicErrorMessage(Op, ErrorMsgReqLA64, DAG)
4982:                : DAG.getNode(LoongArchISD::IOCSRWR_D, DL, MVT::Other, Chain,
4983:                              Op2,
4984:                              DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64,
4985:                                          Op.getOperand(3)));
4986:   }
4987: #define ASRT_LE_GT_CASE(NAME)                                                  \
4988:   case Intrinsic::loongarch_##NAME: {                                          \
4989:     return !Subtarget.is64Bit()                                                \
4990:                ? emitIntrinsicErrorMessage(Op, ErrorMsgReqLA64, DAG)           \
4991:                : Op;                                                           \
4992:   }
4993:     ASRT_LE_GT_CASE(asrtle_d)
4994:     ASRT_LE_GT_CASE(asrtgt_d)
4995: #undef ASRT_LE_GT_CASE
4996:   case Intrinsic::loongarch_ldpte_d: {
4997:     unsigned Imm = Op.getConstantOperandVal(3);
4998:     return !Subtarget.is64Bit()
4999:                ? emitIntrinsicErrorMessage(Op, ErrorMsgReqLA64, DAG)
5000:            : !isUInt<8>(Imm) ? emitIntrinsicErrorMessage(Op, ErrorMsgOOR, DAG)
5001:                              : Op;
5002:   }
5003:   case Intrinsic::loongarch_lsx_vst:
5004:   case Intrinsic::loongarch_lasx_xvst:
5005:     return !isInt<12>(cast<ConstantSDNode>(Op.getOperand(4))->getSExtValue())
5006:                ? emitIntrinsicErrorMessage(Op, ErrorMsgOOR, DAG)
5007:                : SDValue();
5008:   case Intrinsic::loongarch_lasx_xvstelm_b:
5009:     return (!isInt<8>(cast<ConstantSDNode>(Op.getOperand(4))->getSExtValue()) ||
5010:             !isUInt<5>(Op.getConstantOperandVal(5)))
5011:                ? emitIntrinsicErrorMessage(Op, ErrorMsgOOR, DAG)
5012:                : SDValue();
5013:   case Intrinsic::loongarch_lsx_vstelm_b:
5014:     return (!isInt<8>(cast<ConstantSDNode>(Op.getOperand(4))->getSExtValue()) ||
5015:             !isUInt<4>(Op.getConstantOperandVal(5)))
5016:                ? emitIntrinsicErrorMessage(Op, ErrorMsgOOR, DAG)
5017:                : SDValue();
5018:   case Intrinsic::loongarch_lasx_xvstelm_h:
5019:     return (!isShiftedInt<8, 1>(
5020:                 cast<ConstantSDNode>(Op.getOperand(4))->getSExtValue()) ||
5021:             !isUInt<4>(Op.getConstantOperandVal(5)))
5022:                ? emitIntrinsicErrorMessage(
5023:                      Op, "argument out of range or not a multiple of 2", DAG)
5024:                : SDValue();
5025:   case Intrinsic::loongarch_lsx_vstelm_h:
```
- **EN**: The range implements or declares functions including `ASRT_LE_GT_CASE`. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 这一段实现或声明了 `ASRT_LE_GT_CASE` 等函数。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 5026-5100 / 第 5026-5100 行
```cpp
5026:     return (!isShiftedInt<8, 1>(
5027:                 cast<ConstantSDNode>(Op.getOperand(4))->getSExtValue()) ||
5028:             !isUInt<3>(Op.getConstantOperandVal(5)))
5029:                ? emitIntrinsicErrorMessage(
5030:                      Op, "argument out of range or not a multiple of 2", DAG)
5031:                : SDValue();
5032:   case Intrinsic::loongarch_lasx_xvstelm_w:
5033:     return (!isShiftedInt<8, 2>(
5034:                 cast<ConstantSDNode>(Op.getOperand(4))->getSExtValue()) ||
5035:             !isUInt<3>(Op.getConstantOperandVal(5)))
5036:                ? emitIntrinsicErrorMessage(
5037:                      Op, "argument out of range or not a multiple of 4", DAG)
5038:                : SDValue();
5039:   case Intrinsic::loongarch_lsx_vstelm_w:
5040:     return (!isShiftedInt<8, 2>(
5041:                 cast<ConstantSDNode>(Op.getOperand(4))->getSExtValue()) ||
5042:             !isUInt<2>(Op.getConstantOperandVal(5)))
5043:                ? emitIntrinsicErrorMessage(
5044:                      Op, "argument out of range or not a multiple of 4", DAG)
5045:                : SDValue();
5046:   case Intrinsic::loongarch_lasx_xvstelm_d:
5047:     return (!isShiftedInt<8, 3>(
5048:                 cast<ConstantSDNode>(Op.getOperand(4))->getSExtValue()) ||
5049:             !isUInt<2>(Op.getConstantOperandVal(5)))
5050:                ? emitIntrinsicErrorMessage(
5051:                      Op, "argument out of range or not a multiple of 8", DAG)
5052:                : SDValue();
5053:   case Intrinsic::loongarch_lsx_vstelm_d:
5054:     return (!isShiftedInt<8, 3>(
5055:                 cast<ConstantSDNode>(Op.getOperand(4))->getSExtValue()) ||
5056:             !isUInt<1>(Op.getConstantOperandVal(5)))
5057:                ? emitIntrinsicErrorMessage(
5058:                      Op, "argument out of range or not a multiple of 8", DAG)
5059:                : SDValue();
5060:   }
5061: }
5062: 
5063: SDValue LoongArchTargetLowering::lowerShiftLeftParts(SDValue Op,
5064:                                                      SelectionDAG &DAG) const {
5065:   SDLoc DL(Op);
5066:   SDValue Lo = Op.getOperand(0);
5067:   SDValue Hi = Op.getOperand(1);
5068:   SDValue Shamt = Op.getOperand(2);
5069:   EVT VT = Lo.getValueType();
5070: 
5071:   // if Shamt-GRLen < 0: // Shamt < GRLen
5072:   //   Lo = Lo << Shamt
5073:   //   Hi = (Hi << Shamt) | ((Lo >>u 1) >>u (GRLen-1 ^ Shamt))
5074:   // else:
5075:   //   Lo = 0
5076:   //   Hi = Lo << (Shamt-GRLen)
5077: 
5078:   SDValue Zero = DAG.getConstant(0, DL, VT);
5079:   SDValue One = DAG.getConstant(1, DL, VT);
5080:   SDValue MinusGRLen =
5081:       DAG.getSignedConstant(-(int)Subtarget.getGRLen(), DL, VT);
5082:   SDValue GRLenMinus1 = DAG.getConstant(Subtarget.getGRLen() - 1, DL, VT);
5083:   SDValue ShamtMinusGRLen = DAG.getNode(ISD::ADD, DL, VT, Shamt, MinusGRLen);
5084:   SDValue GRLenMinus1Shamt = DAG.getNode(ISD::XOR, DL, VT, Shamt, GRLenMinus1);
5085: 
5086:   SDValue LoTrue = DAG.getNode(ISD::SHL, DL, VT, Lo, Shamt);
5087:   SDValue ShiftRight1Lo = DAG.getNode(ISD::SRL, DL, VT, Lo, One);
5088:   SDValue ShiftRightLo =
5089:       DAG.getNode(ISD::SRL, DL, VT, ShiftRight1Lo, GRLenMinus1Shamt);
5090:   SDValue ShiftLeftHi = DAG.getNode(ISD::SHL, DL, VT, Hi, Shamt);
5091:   SDValue HiTrue = DAG.getNode(ISD::OR, DL, VT, ShiftLeftHi, ShiftRightLo);
5092:   SDValue HiFalse = DAG.getNode(ISD::SHL, DL, VT, Lo, ShamtMinusGRLen);
5093: 
5094:   SDValue CC = DAG.getSetCC(DL, VT, ShamtMinusGRLen, Zero, ISD::SETLT);
5095: 
5096:   Lo = DAG.getNode(ISD::SELECT, DL, VT, CC, LoTrue, Zero);
5097:   Hi = DAG.getNode(ISD::SELECT, DL, VT, CC, HiTrue, HiFalse);
5098: 
5099:   SDValue Parts[2] = {Lo, Hi};
5100:   return DAG.getMergeValues(Parts, DL);
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerShiftLeftParts`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerShiftLeftParts` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 5101-5175 / 第 5101-5175 行
```cpp
5101: }
5102: 
5103: SDValue LoongArchTargetLowering::lowerShiftRightParts(SDValue Op,
5104:                                                       SelectionDAG &DAG,
5105:                                                       bool IsSRA) const {
5106:   SDLoc DL(Op);
5107:   SDValue Lo = Op.getOperand(0);
5108:   SDValue Hi = Op.getOperand(1);
5109:   SDValue Shamt = Op.getOperand(2);
5110:   EVT VT = Lo.getValueType();
5111: 
5112:   // SRA expansion:
5113:   //   if Shamt-GRLen < 0: // Shamt < GRLen
5114:   //     Lo = (Lo >>u Shamt) | ((Hi << 1) << (ShAmt ^ GRLen-1))
5115:   //     Hi = Hi >>s Shamt
5116:   //   else:
5117:   //     Lo = Hi >>s (Shamt-GRLen);
5118:   //     Hi = Hi >>s (GRLen-1)
5119:   //
5120:   // SRL expansion:
5121:   //   if Shamt-GRLen < 0: // Shamt < GRLen
5122:   //     Lo = (Lo >>u Shamt) | ((Hi << 1) << (ShAmt ^ GRLen-1))
5123:   //     Hi = Hi >>u Shamt
5124:   //   else:
5125:   //     Lo = Hi >>u (Shamt-GRLen);
5126:   //     Hi = 0;
5127: 
5128:   unsigned ShiftRightOp = IsSRA ? ISD::SRA : ISD::SRL;
5129: 
5130:   SDValue Zero = DAG.getConstant(0, DL, VT);
5131:   SDValue One = DAG.getConstant(1, DL, VT);
5132:   SDValue MinusGRLen =
5133:       DAG.getSignedConstant(-(int)Subtarget.getGRLen(), DL, VT);
5134:   SDValue GRLenMinus1 = DAG.getConstant(Subtarget.getGRLen() - 1, DL, VT);
5135:   SDValue ShamtMinusGRLen = DAG.getNode(ISD::ADD, DL, VT, Shamt, MinusGRLen);
5136:   SDValue GRLenMinus1Shamt = DAG.getNode(ISD::XOR, DL, VT, Shamt, GRLenMinus1);
5137: 
5138:   SDValue ShiftRightLo = DAG.getNode(ISD::SRL, DL, VT, Lo, Shamt);
5139:   SDValue ShiftLeftHi1 = DAG.getNode(ISD::SHL, DL, VT, Hi, One);
5140:   SDValue ShiftLeftHi =
5141:       DAG.getNode(ISD::SHL, DL, VT, ShiftLeftHi1, GRLenMinus1Shamt);
5142:   SDValue LoTrue = DAG.getNode(ISD::OR, DL, VT, ShiftRightLo, ShiftLeftHi);
5143:   SDValue HiTrue = DAG.getNode(ShiftRightOp, DL, VT, Hi, Shamt);
5144:   SDValue LoFalse = DAG.getNode(ShiftRightOp, DL, VT, Hi, ShamtMinusGRLen);
5145:   SDValue HiFalse =
5146:       IsSRA ? DAG.getNode(ISD::SRA, DL, VT, Hi, GRLenMinus1) : Zero;
5147: 
5148:   SDValue CC = DAG.getSetCC(DL, VT, ShamtMinusGRLen, Zero, ISD::SETLT);
5149: 
5150:   Lo = DAG.getNode(ISD::SELECT, DL, VT, CC, LoTrue, LoFalse);
5151:   Hi = DAG.getNode(ISD::SELECT, DL, VT, CC, HiTrue, HiFalse);
5152: 
5153:   SDValue Parts[2] = {Lo, Hi};
5154:   return DAG.getMergeValues(Parts, DL);
5155: }
5156: 
5157: // Returns the opcode of the target-specific SDNode that implements the 32-bit
5158: // form of the given Opcode.
5159: static unsigned getLoongArchWOpcode(unsigned Opcode) {
5160:   switch (Opcode) {
5161:   default:
5162:     llvm_unreachable("Unexpected opcode");
5163:   case ISD::SDIV:
5164:     return LoongArchISD::DIV_W;
5165:   case ISD::UDIV:
5166:     return LoongArchISD::DIV_WU;
5167:   case ISD::SREM:
5168:     return LoongArchISD::MOD_W;
5169:   case ISD::UREM:
5170:     return LoongArchISD::MOD_WU;
5171:   case ISD::SHL:
5172:     return LoongArchISD::SLL_W;
5173:   case ISD::SRA:
5174:     return LoongArchISD::SRA_W;
5175:   case ISD::SRL:
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::lowerShiftRightParts`, `getLoongArchWOpcode`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::lowerShiftRightParts`, `getLoongArchWOpcode` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 5176-5250 / 第 5176-5250 行
```cpp
5176:     return LoongArchISD::SRL_W;
5177:   case ISD::ROTL:
5178:   case ISD::ROTR:
5179:     return LoongArchISD::ROTR_W;
5180:   case ISD::CTTZ:
5181:     return LoongArchISD::CTZ_W;
5182:   case ISD::CTLZ:
5183:     return LoongArchISD::CLZ_W;
5184:   }
5185: }
5186: 
5187: // Converts the given i8/i16/i32 operation to a target-specific SelectionDAG
5188: // node. Because i8/i16/i32 isn't a legal type for LA64, these operations would
5189: // otherwise be promoted to i64, making it difficult to select the
5190: // SLL_W/.../*W later one because the fact the operation was originally of
5191: // type i8/i16/i32 is lost.
5192: static SDValue customLegalizeToWOp(SDNode *N, SelectionDAG &DAG, int NumOp,
5193:                                    unsigned ExtOpc = ISD::ANY_EXTEND) {
5194:   SDLoc DL(N);
5195:   unsigned WOpcode = getLoongArchWOpcode(N->getOpcode());
5196:   SDValue NewOp0, NewRes;
5197: 
5198:   switch (NumOp) {
5199:   default:
5200:     llvm_unreachable("Unexpected NumOp");
5201:   case 1: {
5202:     NewOp0 = DAG.getNode(ExtOpc, DL, MVT::i64, N->getOperand(0));
5203:     NewRes = DAG.getNode(WOpcode, DL, MVT::i64, NewOp0);
5204:     break;
5205:   }
5206:   case 2: {
5207:     NewOp0 = DAG.getNode(ExtOpc, DL, MVT::i64, N->getOperand(0));
5208:     SDValue NewOp1 = DAG.getNode(ExtOpc, DL, MVT::i64, N->getOperand(1));
5209:     if (N->getOpcode() == ISD::ROTL) {
5210:       SDValue TmpOp = DAG.getConstant(32, DL, MVT::i64);
5211:       NewOp1 = DAG.getNode(ISD::SUB, DL, MVT::i64, TmpOp, NewOp1);
5212:     }
5213:     NewRes = DAG.getNode(WOpcode, DL, MVT::i64, NewOp0, NewOp1);
5214:     break;
5215:   }
5216:     // TODO:Handle more NumOp.
5217:   }
5218: 
5219:   // ReplaceNodeResults requires we maintain the same type for the return
5220:   // value.
5221:   return DAG.getNode(ISD::TRUNCATE, DL, N->getValueType(0), NewRes);
5222: }
5223: 
5224: // Converts the given 32-bit operation to a i64 operation with signed extension
5225: // semantic to reduce the signed extension instructions.
5226: static SDValue customLegalizeToWOpWithSExt(SDNode *N, SelectionDAG &DAG) {
5227:   SDLoc DL(N);
5228:   SDValue NewOp0 = DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, N->getOperand(0));
5229:   SDValue NewOp1 = DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, N->getOperand(1));
5230:   SDValue NewWOp = DAG.getNode(N->getOpcode(), DL, MVT::i64, NewOp0, NewOp1);
5231:   SDValue NewRes = DAG.getNode(ISD::SIGN_EXTEND_INREG, DL, MVT::i64, NewWOp,
5232:                                DAG.getValueType(MVT::i32));
5233:   return DAG.getNode(ISD::TRUNCATE, DL, MVT::i32, NewRes);
5234: }
5235: 
5236: // Helper function that emits error message for intrinsics with/without chain
5237: // and return a UNDEF or and the chain as the results.
5238: static void emitErrorAndReplaceIntrinsicResults(
5239:     SDNode *N, SmallVectorImpl<SDValue> &Results, SelectionDAG &DAG,
5240:     StringRef ErrorMsg, bool WithChain = true) {
5241:   DAG.getContext()->emitError(N->getOperationName(0) + ": " + ErrorMsg + ".");
5242:   Results.push_back(DAG.getUNDEF(N->getValueType(0)));
5243:   if (!WithChain)
5244:     return;
5245:   Results.push_back(N->getOperand(0));
5246: }
5247: 
5248: template <unsigned N>
5249: static void
5250: replaceVPICKVE2GRResults(SDNode *Node, SmallVectorImpl<SDValue> &Results,
```
- **EN**: The range implements or declares functions including `customLegalizeToWOp`, `customLegalizeToWOpWithSExt`, `emitErrorAndReplaceIntrinsicResults`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `customLegalizeToWOp`, `customLegalizeToWOpWithSExt`, `emitErrorAndReplaceIntrinsicResults` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 5251-5325 / 第 5251-5325 行
```cpp
5251:                          SelectionDAG &DAG, const LoongArchSubtarget &Subtarget,
5252:                          unsigned ResOp) {
5253:   const StringRef ErrorMsgOOR = "argument out of range";
5254:   unsigned Imm = Node->getConstantOperandVal(2);
5255:   if (!isUInt<N>(Imm)) {
5256:     emitErrorAndReplaceIntrinsicResults(Node, Results, DAG, ErrorMsgOOR,
5257:                                         /*WithChain=*/false);
5258:     return;
5259:   }
5260:   SDLoc DL(Node);
5261:   SDValue Vec = Node->getOperand(1);
5262: 
5263:   SDValue PickElt =
5264:       DAG.getNode(ResOp, DL, Subtarget.getGRLenVT(), Vec,
5265:                   DAG.getConstant(Imm, DL, Subtarget.getGRLenVT()),
5266:                   DAG.getValueType(Vec.getValueType().getVectorElementType()));
5267:   Results.push_back(DAG.getNode(ISD::TRUNCATE, DL, Node->getValueType(0),
5268:                                 PickElt.getValue(0)));
5269: }
5270: 
5271: static void replaceVecCondBranchResults(SDNode *N,
5272:                                         SmallVectorImpl<SDValue> &Results,
5273:                                         SelectionDAG &DAG,
5274:                                         const LoongArchSubtarget &Subtarget,
5275:                                         unsigned ResOp) {
5276:   SDLoc DL(N);
5277:   SDValue Vec = N->getOperand(1);
5278: 
5279:   SDValue CB = DAG.getNode(ResOp, DL, Subtarget.getGRLenVT(), Vec);
5280:   Results.push_back(
5281:       DAG.getNode(ISD::TRUNCATE, DL, N->getValueType(0), CB.getValue(0)));
5282: }
5283: 
5284: static void
5285: replaceINTRINSIC_WO_CHAINResults(SDNode *N, SmallVectorImpl<SDValue> &Results,
5286:                                  SelectionDAG &DAG,
5287:                                  const LoongArchSubtarget &Subtarget) {
5288:   switch (N->getConstantOperandVal(0)) {
5289:   default:
5290:     llvm_unreachable("Unexpected Intrinsic.");
5291:   case Intrinsic::loongarch_lsx_vpickve2gr_b:
5292:     replaceVPICKVE2GRResults<4>(N, Results, DAG, Subtarget,
5293:                                 LoongArchISD::VPICK_SEXT_ELT);
5294:     break;
5295:   case Intrinsic::loongarch_lsx_vpickve2gr_h:
5296:   case Intrinsic::loongarch_lasx_xvpickve2gr_w:
5297:     replaceVPICKVE2GRResults<3>(N, Results, DAG, Subtarget,
5298:                                 LoongArchISD::VPICK_SEXT_ELT);
5299:     break;
5300:   case Intrinsic::loongarch_lsx_vpickve2gr_w:
5301:     replaceVPICKVE2GRResults<2>(N, Results, DAG, Subtarget,
5302:                                 LoongArchISD::VPICK_SEXT_ELT);
5303:     break;
5304:   case Intrinsic::loongarch_lsx_vpickve2gr_bu:
5305:     replaceVPICKVE2GRResults<4>(N, Results, DAG, Subtarget,
5306:                                 LoongArchISD::VPICK_ZEXT_ELT);
5307:     break;
5308:   case Intrinsic::loongarch_lsx_vpickve2gr_hu:
5309:   case Intrinsic::loongarch_lasx_xvpickve2gr_wu:
5310:     replaceVPICKVE2GRResults<3>(N, Results, DAG, Subtarget,
5311:                                 LoongArchISD::VPICK_ZEXT_ELT);
5312:     break;
5313:   case Intrinsic::loongarch_lsx_vpickve2gr_wu:
5314:     replaceVPICKVE2GRResults<2>(N, Results, DAG, Subtarget,
5315:                                 LoongArchISD::VPICK_ZEXT_ELT);
5316:     break;
5317:   case Intrinsic::loongarch_lsx_bz_b:
5318:   case Intrinsic::loongarch_lsx_bz_h:
5319:   case Intrinsic::loongarch_lsx_bz_w:
5320:   case Intrinsic::loongarch_lsx_bz_d:
5321:   case Intrinsic::loongarch_lasx_xbz_b:
5322:   case Intrinsic::loongarch_lasx_xbz_h:
5323:   case Intrinsic::loongarch_lasx_xbz_w:
5324:   case Intrinsic::loongarch_lasx_xbz_d:
5325:     replaceVecCondBranchResults(N, Results, DAG, Subtarget,
```
- **EN**: The range implements or declares functions including `replaceVecCondBranchResults`, `replaceINTRINSIC_WO_CHAINResults`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `replaceVecCondBranchResults`, `replaceINTRINSIC_WO_CHAINResults` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 5326-5400 / 第 5326-5400 行
```cpp
5326:                                 LoongArchISD::VALL_ZERO);
5327:     break;
5328:   case Intrinsic::loongarch_lsx_bz_v:
5329:   case Intrinsic::loongarch_lasx_xbz_v:
5330:     replaceVecCondBranchResults(N, Results, DAG, Subtarget,
5331:                                 LoongArchISD::VANY_ZERO);
5332:     break;
5333:   case Intrinsic::loongarch_lsx_bnz_b:
5334:   case Intrinsic::loongarch_lsx_bnz_h:
5335:   case Intrinsic::loongarch_lsx_bnz_w:
5336:   case Intrinsic::loongarch_lsx_bnz_d:
5337:   case Intrinsic::loongarch_lasx_xbnz_b:
5338:   case Intrinsic::loongarch_lasx_xbnz_h:
5339:   case Intrinsic::loongarch_lasx_xbnz_w:
5340:   case Intrinsic::loongarch_lasx_xbnz_d:
5341:     replaceVecCondBranchResults(N, Results, DAG, Subtarget,
5342:                                 LoongArchISD::VALL_NONZERO);
5343:     break;
5344:   case Intrinsic::loongarch_lsx_bnz_v:
5345:   case Intrinsic::loongarch_lasx_xbnz_v:
5346:     replaceVecCondBranchResults(N, Results, DAG, Subtarget,
5347:                                 LoongArchISD::VANY_NONZERO);
5348:     break;
5349:   }
5350: }
5351: 
5352: static void replaceCMP_XCHG_128Results(SDNode *N,
5353:                                        SmallVectorImpl<SDValue> &Results,
5354:                                        SelectionDAG &DAG) {
5355:   assert(N->getValueType(0) == MVT::i128 &&
5356:          "AtomicCmpSwap on types less than 128 should be legal");
5357:   MachineMemOperand *MemOp = cast<MemSDNode>(N)->getMemOperand();
5358: 
5359:   unsigned Opcode;
5360:   switch (MemOp->getMergedOrdering()) {
5361:   case AtomicOrdering::Acquire:
5362:   case AtomicOrdering::AcquireRelease:
5363:   case AtomicOrdering::SequentiallyConsistent:
5364:     Opcode = LoongArch::PseudoCmpXchg128Acquire;
5365:     break;
5366:   case AtomicOrdering::Monotonic:
5367:   case AtomicOrdering::Release:
5368:     Opcode = LoongArch::PseudoCmpXchg128;
5369:     break;
5370:   default:
5371:     llvm_unreachable("Unexpected ordering!");
5372:   }
5373: 
5374:   SDLoc DL(N);
5375:   auto CmpVal = DAG.SplitScalar(N->getOperand(2), DL, MVT::i64, MVT::i64);
5376:   auto NewVal = DAG.SplitScalar(N->getOperand(3), DL, MVT::i64, MVT::i64);
5377:   SDValue Ops[] = {N->getOperand(1), CmpVal.first,  CmpVal.second,
5378:                    NewVal.first,     NewVal.second, N->getOperand(0)};
5379: 
5380:   SDNode *CmpSwap = DAG.getMachineNode(
5381:       Opcode, SDLoc(N), DAG.getVTList(MVT::i64, MVT::i64, MVT::i64, MVT::Other),
5382:       Ops);
5383:   DAG.setNodeMemRefs(cast<MachineSDNode>(CmpSwap), {MemOp});
5384:   Results.push_back(DAG.getNode(ISD::BUILD_PAIR, DL, MVT::i128,
5385:                                 SDValue(CmpSwap, 0), SDValue(CmpSwap, 1)));
5386:   Results.push_back(SDValue(CmpSwap, 3));
5387: }
5388: 
5389: void LoongArchTargetLowering::ReplaceNodeResults(
5390:     SDNode *N, SmallVectorImpl<SDValue> &Results, SelectionDAG &DAG) const {
5391:   SDLoc DL(N);
5392:   EVT VT = N->getValueType(0);
5393:   switch (N->getOpcode()) {
5394:   default:
5395:     llvm_unreachable("Don't know how to legalize this operation");
5396:   case ISD::ADD:
5397:   case ISD::SUB:
5398:     assert(N->getValueType(0) == MVT::i32 && Subtarget.is64Bit() &&
5399:            "Unexpected custom legalisation");
5400:     Results.push_back(customLegalizeToWOpWithSExt(N, DAG));
```
- **EN**: The range implements or declares functions including `replaceCMP_XCHG_128Results`, `LoongArchTargetLowering::ReplaceNodeResults`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `replaceCMP_XCHG_128Results`, `LoongArchTargetLowering::ReplaceNodeResults` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 5401-5475 / 第 5401-5475 行
```cpp
5401:     break;
5402:   case ISD::SDIV:
5403:   case ISD::UDIV:
5404:   case ISD::SREM:
5405:   case ISD::UREM:
5406:     assert(VT == MVT::i32 && Subtarget.is64Bit() &&
5407:            "Unexpected custom legalisation");
5408:     Results.push_back(customLegalizeToWOp(N, DAG, 2,
5409:                                           Subtarget.hasDiv32() && VT == MVT::i32
5410:                                               ? ISD::ANY_EXTEND
5411:                                               : ISD::SIGN_EXTEND));
5412:     break;
5413:   case ISD::SHL:
5414:   case ISD::SRA:
5415:   case ISD::SRL:
5416:     assert(VT == MVT::i32 && Subtarget.is64Bit() &&
5417:            "Unexpected custom legalisation");
5418:     if (N->getOperand(1).getOpcode() != ISD::Constant) {
5419:       Results.push_back(customLegalizeToWOp(N, DAG, 2));
5420:       break;
5421:     }
5422:     break;
5423:   case ISD::ROTL:
5424:   case ISD::ROTR:
5425:     assert(VT == MVT::i32 && Subtarget.is64Bit() &&
5426:            "Unexpected custom legalisation");
5427:     Results.push_back(customLegalizeToWOp(N, DAG, 2));
5428:     break;
5429:   case ISD::LOAD: {
5430:     // Use an f64 load and a scalar_to_vector for v2f32 loads. This avoids
5431:     // scalarizing in 32-bit mode. In 64-bit mode this avoids a int->fp
5432:     // cast since type legalization will try to use an i64 load.
5433:     MVT VT = N->getSimpleValueType(0);
5434:     assert(VT == MVT::v2f32 && Subtarget.hasExtLSX() &&
5435:            "Unexpected custom legalisation");
5436:     assert(getTypeAction(*DAG.getContext(), VT) == TypeWidenVector &&
5437:            "Unexpected type action!");
5438:     if (!ISD::isNON_EXTLoad(N))
5439:       return;
5440:     auto *Ld = cast<LoadSDNode>(N);
5441:     SDValue Res = DAG.getLoad(MVT::f64, DL, Ld->getChain(), Ld->getBasePtr(),
5442:                               Ld->getPointerInfo(), Ld->getBaseAlign(),
5443:                               Ld->getMemOperand()->getFlags());
5444:     SDValue Chain = Res.getValue(1);
5445:     MVT VecVT = MVT::getVectorVT(MVT::f64, 2);
5446:     Res = DAG.getNode(ISD::SCALAR_TO_VECTOR, DL, VecVT, Res);
5447:     EVT WideVT = getTypeToTransformTo(*DAG.getContext(), VT);
5448:     Res = DAG.getBitcast(WideVT, Res);
5449:     Results.push_back(Res);
5450:     Results.push_back(Chain);
5451:     break;
5452:   }
5453:   case ISD::FP_TO_SINT: {
5454:     assert(VT == MVT::i32 && Subtarget.is64Bit() &&
5455:            "Unexpected custom legalisation");
5456:     SDValue Src = N->getOperand(0);
5457:     EVT FVT = EVT::getFloatingPointVT(N->getValueSizeInBits(0));
5458:     if (getTypeAction(*DAG.getContext(), Src.getValueType()) !=
5459:         TargetLowering::TypeSoftenFloat) {
5460:       if (!isTypeLegal(Src.getValueType()))
5461:         return;
5462:       if (Src.getValueType() == MVT::f16)
5463:         Src = DAG.getNode(ISD::FP_EXTEND, DL, MVT::f32, Src);
5464:       SDValue Dst = DAG.getNode(LoongArchISD::FTINT, DL, FVT, Src);
5465:       Results.push_back(DAG.getNode(ISD::BITCAST, DL, VT, Dst));
5466:       return;
5467:     }
5468:     // If the FP type needs to be softened, emit a library call using the 'si'
5469:     // version. If we left it to default legalization we'd end up with 'di'.
5470:     RTLIB::Libcall LC;
5471:     LC = RTLIB::getFPTOSINT(Src.getValueType(), VT);
5472:     MakeLibCallOptions CallOptions;
5473:     EVT OpVT = Src.getValueType();
5474:     CallOptions.setTypeListBeforeSoften(OpVT, VT);
5475:     SDValue Chain = SDValue();
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 5476-5550 / 第 5476-5550 行
```cpp
5476:     SDValue Result;
5477:     std::tie(Result, Chain) =
5478:         makeLibCall(DAG, LC, VT, Src, CallOptions, DL, Chain);
5479:     Results.push_back(Result);
5480:     break;
5481:   }
5482:   case ISD::BITCAST: {
5483:     SDValue Src = N->getOperand(0);
5484:     EVT SrcVT = Src.getValueType();
5485:     if (VT == MVT::i32 && SrcVT == MVT::f32 && Subtarget.is64Bit() &&
5486:         Subtarget.hasBasicF()) {
5487:       SDValue Dst =
5488:           DAG.getNode(LoongArchISD::MOVFR2GR_S_LA64, DL, MVT::i64, Src);
5489:       Results.push_back(DAG.getNode(ISD::TRUNCATE, DL, MVT::i32, Dst));
5490:     } else if (VT == MVT::i64 && SrcVT == MVT::f64 && !Subtarget.is64Bit()) {
5491:       SDValue NewReg = DAG.getNode(LoongArchISD::SPLIT_PAIR_F64, DL,
5492:                                    DAG.getVTList(MVT::i32, MVT::i32), Src);
5493:       SDValue RetReg = DAG.getNode(ISD::BUILD_PAIR, DL, MVT::i64,
5494:                                    NewReg.getValue(0), NewReg.getValue(1));
5495:       Results.push_back(RetReg);
5496:     }
5497:     break;
5498:   }
5499:   case ISD::FP_TO_UINT: {
5500:     assert(VT == MVT::i32 && Subtarget.is64Bit() &&
5501:            "Unexpected custom legalisation");
5502:     auto &TLI = DAG.getTargetLoweringInfo();
5503:     SDValue Tmp1, Tmp2;
5504:     TLI.expandFP_TO_UINT(N, Tmp1, Tmp2, DAG);
5505:     Results.push_back(DAG.getNode(ISD::TRUNCATE, DL, MVT::i32, Tmp1));
5506:     break;
5507:   }
5508:   case ISD::FP_ROUND: {
5509:     assert(VT == MVT::v2f32 && Subtarget.hasExtLSX() &&
5510:            "Unexpected custom legalisation");
5511:     // On LSX platforms, rounding from v2f64 to v4f32 (after legalization from
5512:     // v2f32) is scalarized. Add a customized v2f32 widening to convert it into
5513:     // a target-specific LoongArchISD::VFCVT to optimize it.
5514:     SDValue Op0 = N->getOperand(0);
5515:     EVT OpVT = Op0.getValueType();
5516:     if (OpVT == MVT::v2f64) {
5517:       SDValue Undef = DAG.getUNDEF(OpVT);
5518:       SDValue Dst =
5519:           DAG.getNode(LoongArchISD::VFCVT, DL, MVT::v4f32, Undef, Op0);
5520:       Results.push_back(Dst);
5521:     }
5522:     break;
5523:   }
5524:   case ISD::BSWAP: {
5525:     SDValue Src = N->getOperand(0);
5526:     assert((VT == MVT::i16 || VT == MVT::i32) &&
5527:            "Unexpected custom legalization");
5528:     MVT GRLenVT = Subtarget.getGRLenVT();
5529:     SDValue NewSrc = DAG.getNode(ISD::ANY_EXTEND, DL, GRLenVT, Src);
5530:     SDValue Tmp;
5531:     switch (VT.getSizeInBits()) {
5532:     default:
5533:       llvm_unreachable("Unexpected operand width");
5534:     case 16:
5535:       Tmp = DAG.getNode(LoongArchISD::REVB_2H, DL, GRLenVT, NewSrc);
5536:       break;
5537:     case 32:
5538:       // Only LA64 will get to here due to the size mismatch between VT and
5539:       // GRLenVT, LA32 lowering is directly defined in LoongArchInstrInfo.
5540:       Tmp = DAG.getNode(LoongArchISD::REVB_2W, DL, GRLenVT, NewSrc);
5541:       break;
5542:     }
5543:     Results.push_back(DAG.getNode(ISD::TRUNCATE, DL, VT, Tmp));
5544:     break;
5545:   }
5546:   case ISD::BITREVERSE: {
5547:     SDValue Src = N->getOperand(0);
5548:     assert((VT == MVT::i8 || (VT == MVT::i32 && Subtarget.is64Bit())) &&
5549:            "Unexpected custom legalization");
5550:     MVT GRLenVT = Subtarget.getGRLenVT();
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 5551-5625 / 第 5551-5625 行
```cpp
5551:     SDValue NewSrc = DAG.getNode(ISD::ANY_EXTEND, DL, GRLenVT, Src);
5552:     SDValue Tmp;
5553:     switch (VT.getSizeInBits()) {
5554:     default:
5555:       llvm_unreachable("Unexpected operand width");
5556:     case 8:
5557:       Tmp = DAG.getNode(LoongArchISD::BITREV_4B, DL, GRLenVT, NewSrc);
5558:       break;
5559:     case 32:
5560:       Tmp = DAG.getNode(LoongArchISD::BITREV_W, DL, GRLenVT, NewSrc);
5561:       break;
5562:     }
5563:     Results.push_back(DAG.getNode(ISD::TRUNCATE, DL, VT, Tmp));
5564:     break;
5565:   }
5566:   case ISD::CTLZ:
5567:   case ISD::CTTZ: {
5568:     assert(VT == MVT::i32 && Subtarget.is64Bit() &&
5569:            "Unexpected custom legalisation");
5570:     Results.push_back(customLegalizeToWOp(N, DAG, 1));
5571:     break;
5572:   }
5573:   case ISD::INTRINSIC_W_CHAIN: {
5574:     SDValue Chain = N->getOperand(0);
5575:     SDValue Op2 = N->getOperand(2);
5576:     MVT GRLenVT = Subtarget.getGRLenVT();
5577:     const StringRef ErrorMsgOOR = "argument out of range";
5578:     const StringRef ErrorMsgReqLA64 = "requires loongarch64";
5579:     const StringRef ErrorMsgReqF = "requires basic 'f' target feature";
5580: 
5581:     switch (N->getConstantOperandVal(1)) {
5582:     default:
5583:       llvm_unreachable("Unexpected Intrinsic.");
5584:     case Intrinsic::loongarch_movfcsr2gr: {
5585:       if (!Subtarget.hasBasicF()) {
5586:         emitErrorAndReplaceIntrinsicResults(N, Results, DAG, ErrorMsgReqF);
5587:         return;
5588:       }
5589:       unsigned Imm = Op2->getAsZExtVal();
5590:       if (!isUInt<2>(Imm)) {
5591:         emitErrorAndReplaceIntrinsicResults(N, Results, DAG, ErrorMsgOOR);
5592:         return;
5593:       }
5594:       SDValue MOVFCSR2GRResults = DAG.getNode(
5595:           LoongArchISD::MOVFCSR2GR, SDLoc(N), {MVT::i64, MVT::Other},
5596:           {Chain, DAG.getConstant(Imm, DL, GRLenVT)});
5597:       Results.push_back(
5598:           DAG.getNode(ISD::TRUNCATE, DL, VT, MOVFCSR2GRResults.getValue(0)));
5599:       Results.push_back(MOVFCSR2GRResults.getValue(1));
5600:       break;
5601:     }
5602: #define CRC_CASE_EXT_BINARYOP(NAME, NODE)                                      \
5603:   case Intrinsic::loongarch_##NAME: {                                          \
5604:     SDValue NODE = DAG.getNode(                                                \
5605:         LoongArchISD::NODE, DL, {MVT::i64, MVT::Other},                        \
5606:         {Chain, DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, Op2),               \
5607:          DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, N->getOperand(3))});       \
5608:     Results.push_back(DAG.getNode(ISD::TRUNCATE, DL, VT, NODE.getValue(0)));   \
5609:     Results.push_back(NODE.getValue(1));                                       \
5610:     break;                                                                     \
5611:   }
5612:       CRC_CASE_EXT_BINARYOP(crc_w_b_w, CRC_W_B_W)
5613:       CRC_CASE_EXT_BINARYOP(crc_w_h_w, CRC_W_H_W)
5614:       CRC_CASE_EXT_BINARYOP(crc_w_w_w, CRC_W_W_W)
5615:       CRC_CASE_EXT_BINARYOP(crcc_w_b_w, CRCC_W_B_W)
5616:       CRC_CASE_EXT_BINARYOP(crcc_w_h_w, CRCC_W_H_W)
5617:       CRC_CASE_EXT_BINARYOP(crcc_w_w_w, CRCC_W_W_W)
5618: #undef CRC_CASE_EXT_BINARYOP
5619: 
5620: #define CRC_CASE_EXT_UNARYOP(NAME, NODE)                                       \
5621:   case Intrinsic::loongarch_##NAME: {                                          \
5622:     SDValue NODE = DAG.getNode(                                                \
5623:         LoongArchISD::NODE, DL, {MVT::i64, MVT::Other},                        \
5624:         {Chain, Op2,                                                           \
5625:          DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, N->getOperand(3))});       \
```
- **EN**: The range implements or declares functions including `CRC_CASE_EXT_BINARYOP`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `CRC_CASE_EXT_BINARYOP` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 5626-5700 / 第 5626-5700 行
```cpp
5626:     Results.push_back(DAG.getNode(ISD::TRUNCATE, DL, VT, NODE.getValue(0)));   \
5627:     Results.push_back(NODE.getValue(1));                                       \
5628:     break;                                                                     \
5629:   }
5630:       CRC_CASE_EXT_UNARYOP(crc_w_d_w, CRC_W_D_W)
5631:       CRC_CASE_EXT_UNARYOP(crcc_w_d_w, CRCC_W_D_W)
5632: #undef CRC_CASE_EXT_UNARYOP
5633: #define CSR_CASE(ID)                                                           \
5634:   case Intrinsic::loongarch_##ID: {                                            \
5635:     if (!Subtarget.is64Bit())                                                  \
5636:       emitErrorAndReplaceIntrinsicResults(N, Results, DAG, ErrorMsgReqLA64);   \
5637:     break;                                                                     \
5638:   }
5639:       CSR_CASE(csrrd_d);
5640:       CSR_CASE(csrwr_d);
5641:       CSR_CASE(csrxchg_d);
5642:       CSR_CASE(iocsrrd_d);
5643: #undef CSR_CASE
5644:     case Intrinsic::loongarch_csrrd_w: {
5645:       unsigned Imm = Op2->getAsZExtVal();
5646:       if (!isUInt<14>(Imm)) {
5647:         emitErrorAndReplaceIntrinsicResults(N, Results, DAG, ErrorMsgOOR);
5648:         return;
5649:       }
5650:       SDValue CSRRDResults =
5651:           DAG.getNode(LoongArchISD::CSRRD, DL, {GRLenVT, MVT::Other},
5652:                       {Chain, DAG.getConstant(Imm, DL, GRLenVT)});
5653:       Results.push_back(
5654:           DAG.getNode(ISD::TRUNCATE, DL, VT, CSRRDResults.getValue(0)));
5655:       Results.push_back(CSRRDResults.getValue(1));
5656:       break;
5657:     }
5658:     case Intrinsic::loongarch_csrwr_w: {
5659:       unsigned Imm = N->getConstantOperandVal(3);
5660:       if (!isUInt<14>(Imm)) {
5661:         emitErrorAndReplaceIntrinsicResults(N, Results, DAG, ErrorMsgOOR);
5662:         return;
5663:       }
5664:       SDValue CSRWRResults =
5665:           DAG.getNode(LoongArchISD::CSRWR, DL, {GRLenVT, MVT::Other},
5666:                       {Chain, DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, Op2),
5667:                        DAG.getConstant(Imm, DL, GRLenVT)});
5668:       Results.push_back(
5669:           DAG.getNode(ISD::TRUNCATE, DL, VT, CSRWRResults.getValue(0)));
5670:       Results.push_back(CSRWRResults.getValue(1));
5671:       break;
5672:     }
5673:     case Intrinsic::loongarch_csrxchg_w: {
5674:       unsigned Imm = N->getConstantOperandVal(4);
5675:       if (!isUInt<14>(Imm)) {
5676:         emitErrorAndReplaceIntrinsicResults(N, Results, DAG, ErrorMsgOOR);
5677:         return;
5678:       }
5679:       SDValue CSRXCHGResults = DAG.getNode(
5680:           LoongArchISD::CSRXCHG, DL, {GRLenVT, MVT::Other},
5681:           {Chain, DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, Op2),
5682:            DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, N->getOperand(3)),
5683:            DAG.getConstant(Imm, DL, GRLenVT)});
5684:       Results.push_back(
5685:           DAG.getNode(ISD::TRUNCATE, DL, VT, CSRXCHGResults.getValue(0)));
5686:       Results.push_back(CSRXCHGResults.getValue(1));
5687:       break;
5688:     }
5689: #define IOCSRRD_CASE(NAME, NODE)                                               \
5690:   case Intrinsic::loongarch_##NAME: {                                          \
5691:     SDValue IOCSRRDResults =                                                   \
5692:         DAG.getNode(LoongArchISD::NODE, DL, {MVT::i64, MVT::Other},            \
5693:                     {Chain, DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, Op2)}); \
5694:     Results.push_back(                                                         \
5695:         DAG.getNode(ISD::TRUNCATE, DL, VT, IOCSRRDResults.getValue(0)));       \
5696:     Results.push_back(IOCSRRDResults.getValue(1));                             \
5697:     break;                                                                     \
5698:   }
5699:       IOCSRRD_CASE(iocsrrd_b, IOCSRRD_B);
5700:       IOCSRRD_CASE(iocsrrd_h, IOCSRRD_H);
```
- **EN**: The range implements or declares functions including `CRC_CASE_EXT_UNARYOP`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 这一段实现或声明了 `CRC_CASE_EXT_UNARYOP` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 5701-5775 / 第 5701-5775 行
```cpp
5701:       IOCSRRD_CASE(iocsrrd_w, IOCSRRD_W);
5702: #undef IOCSRRD_CASE
5703:     case Intrinsic::loongarch_cpucfg: {
5704:       SDValue CPUCFGResults =
5705:           DAG.getNode(LoongArchISD::CPUCFG, DL, {GRLenVT, MVT::Other},
5706:                       {Chain, DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i64, Op2)});
5707:       Results.push_back(
5708:           DAG.getNode(ISD::TRUNCATE, DL, VT, CPUCFGResults.getValue(0)));
5709:       Results.push_back(CPUCFGResults.getValue(1));
5710:       break;
5711:     }
5712:     case Intrinsic::loongarch_lddir_d: {
5713:       if (!Subtarget.is64Bit()) {
5714:         emitErrorAndReplaceIntrinsicResults(N, Results, DAG, ErrorMsgReqLA64);
5715:         return;
5716:       }
5717:       break;
5718:     }
5719:     }
5720:     break;
5721:   }
5722:   case ISD::READ_REGISTER: {
5723:     if (Subtarget.is64Bit())
5724:       DAG.getContext()->emitError(
5725:           "On LA64, only 64-bit registers can be read.");
5726:     else
5727:       DAG.getContext()->emitError(
5728:           "On LA32, only 32-bit registers can be read.");
5729:     Results.push_back(DAG.getUNDEF(VT));
5730:     Results.push_back(N->getOperand(0));
5731:     break;
5732:   }
5733:   case ISD::INTRINSIC_WO_CHAIN: {
5734:     replaceINTRINSIC_WO_CHAINResults(N, Results, DAG, Subtarget);
5735:     break;
5736:   }
5737:   case ISD::LROUND: {
5738:     SDValue Op0 = N->getOperand(0);
5739:     EVT OpVT = Op0.getValueType();
5740:     RTLIB::Libcall LC =
5741:         OpVT == MVT::f64 ? RTLIB::LROUND_F64 : RTLIB::LROUND_F32;
5742:     MakeLibCallOptions CallOptions;
5743:     CallOptions.setTypeListBeforeSoften(OpVT, MVT::i64);
5744:     SDValue Result = makeLibCall(DAG, LC, MVT::i64, Op0, CallOptions, DL).first;
5745:     Result = DAG.getNode(ISD::TRUNCATE, DL, MVT::i32, Result);
5746:     Results.push_back(Result);
5747:     break;
5748:   }
5749:   case ISD::ATOMIC_CMP_SWAP: {
5750:     replaceCMP_XCHG_128Results(N, Results, DAG);
5751:     break;
5752:   }
5753:   case ISD::TRUNCATE: {
5754:     MVT VT = N->getSimpleValueType(0);
5755:     if (getTypeAction(*DAG.getContext(), VT) != TypeWidenVector)
5756:       return;
5757: 
5758:     MVT WidenVT = getTypeToTransformTo(*DAG.getContext(), VT).getSimpleVT();
5759:     SDValue In = N->getOperand(0);
5760:     EVT InVT = In.getValueType();
5761:     EVT InEltVT = InVT.getVectorElementType();
5762:     EVT EltVT = VT.getVectorElementType();
5763:     unsigned MinElts = VT.getVectorNumElements();
5764:     unsigned WidenNumElts = WidenVT.getVectorNumElements();
5765:     unsigned InBits = InVT.getSizeInBits();
5766: 
5767:     if ((128 % InBits) == 0 && WidenVT.is128BitVector()) {
5768:       if ((InEltVT.getSizeInBits() % EltVT.getSizeInBits()) == 0) {
5769:         int Scale = InEltVT.getSizeInBits() / EltVT.getSizeInBits();
5770:         SmallVector<int, 16> TruncMask(WidenNumElts, -1);
5771:         for (unsigned I = 0; I < MinElts; ++I)
5772:           TruncMask[I] = Scale * I;
5773: 
5774:         unsigned WidenNumElts = 128 / In.getScalarValueSizeInBits();
5775:         MVT SVT = In.getSimpleValueType().getScalarType();
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. Preprocessor definitions in this span wire generated fragments or local compile-time helpers into the file.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 这一段中的预处理器定义把生成片段或局部编译期辅助逻辑接入文件。

### Lines 5776-5850 / 第 5776-5850 行
```cpp
5776:         MVT VT = MVT::getVectorVT(SVT, WidenNumElts);
5777:         SDValue WidenIn =
5778:             DAG.getNode(ISD::INSERT_SUBVECTOR, DL, VT, DAG.getUNDEF(VT), In,
5779:                         DAG.getVectorIdxConstant(0, DL));
5780:         assert(isTypeLegal(WidenVT) && isTypeLegal(WidenIn.getValueType()) &&
5781:                "Illegal vector type in truncation");
5782:         WidenIn = DAG.getBitcast(WidenVT, WidenIn);
5783:         Results.push_back(
5784:             DAG.getVectorShuffle(WidenVT, DL, WidenIn, WidenIn, TruncMask));
5785:         return;
5786:       }
5787:     }
5788: 
5789:     break;
5790:   }
5791:   case ISD::SIGN_EXTEND: {
5792:     // LASX has native VEXT2XV_* for sign extension.
5793:     if (!Subtarget.hasExtLSX() || Subtarget.hasExtLASX())
5794:       return;
5795: 
5796:     EVT DstVT = N->getValueType(0);
5797:     SDValue Src = N->getOperand(0);
5798:     MVT SrcVT = Src.getSimpleValueType();
5799: 
5800:     unsigned SrcEltBits = SrcVT.getScalarSizeInBits();
5801:     unsigned DstEltBits = DstVT.getScalarSizeInBits();
5802:     unsigned NumElts = DstVT.getVectorNumElements();
5803: 
5804:     if (SrcVT.getSizeInBits() > 128)
5805:       return;
5806: 
5807:     if (!DstVT.isVector() || DstVT.getSizeInBits() <= 128)
5808:       return;
5809: 
5810:     // Legalize and extend the src to 128-bit first.
5811:     if (SrcVT.getSizeInBits() < 128) {
5812:       unsigned WidenSrcElts = 128 / SrcEltBits;
5813:       MVT WidenSrcVT = MVT::getVectorVT(SrcVT.getScalarType(), WidenSrcElts);
5814:       Src = DAG.getNode(ISD::INSERT_SUBVECTOR, DL, WidenSrcVT,
5815:                         DAG.getUNDEF(WidenSrcVT), Src,
5816:                         DAG.getVectorIdxConstant(0, DL));
5817:       SrcVT = WidenSrcVT;
5818: 
5819:       unsigned FirstStageEltBits = 128 / NumElts;
5820:       MVT FirstStageEltVT = MVT::getIntegerVT(FirstStageEltBits);
5821:       MVT FirstStageVT = MVT::getVectorVT(FirstStageEltVT, NumElts);
5822:       Src = DAG.getNode(ISD::SIGN_EXTEND_VECTOR_INREG, DL, FirstStageVT, Src);
5823:       SrcVT = FirstStageVT;
5824:       SrcEltBits = FirstStageEltBits;
5825:     }
5826: 
5827:     SmallVector<SDValue, 8> Blocks;
5828:     Blocks.push_back(Src);
5829: 
5830:     // Sign-extend the src by using SLTI + VILVL + VILVH recursively.
5831:     while (SrcEltBits < DstEltBits) {
5832:       unsigned NextEltBits = SrcEltBits * 2;
5833:       MVT NextEltVT = MVT::getIntegerVT(NextEltBits);
5834:       unsigned CurEltsPerBlock = SrcVT.getVectorNumElements();
5835:       unsigned NextEltsPerBlock = CurEltsPerBlock / 2;
5836:       MVT NextBlockVT = MVT::getVectorVT(NextEltVT, NextEltsPerBlock);
5837: 
5838:       SmallVector<SDValue, 8> NextBlocks;
5839:       NextBlocks.reserve(Blocks.size() * 2);
5840:       for (SDValue Block : Blocks) {
5841:         SDValue Zero = DAG.getConstant(0, DL, SrcVT);
5842:         SDValue Mask = DAG.getNode(ISD::SETCC, DL, SrcVT, Block, Zero,
5843:                                    DAG.getCondCode(ISD::SETLT));
5844:         SDValue LoInterleaved =
5845:             DAG.getNode(LoongArchISD::VILVL, DL, SrcVT, Mask, Block);
5846:         SDValue HiInterleaved =
5847:             DAG.getNode(LoongArchISD::VILVH, DL, SrcVT, Mask, Block);
5848: 
5849:         NextBlocks.push_back(DAG.getBitcast(NextBlockVT, LoInterleaved));
5850:         NextBlocks.push_back(DAG.getBitcast(NextBlockVT, HiInterleaved));
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 5851-5925 / 第 5851-5925 行
```cpp
5851:       }
5852: 
5853:       Blocks = std::move(NextBlocks);
5854:       SrcVT = NextBlockVT;
5855:       SrcEltBits = NextEltBits;
5856:     }
5857: 
5858:     Results.push_back(DAG.getNode(ISD::CONCAT_VECTORS, DL, DstVT, Blocks));
5859:     break;
5860:   }
5861:   }
5862: }
5863: 
5864: /// Try to fold: (and (xor X, -1), Y) -> (vandn X, Y).
5865: static SDValue combineAndNotIntoVANDN(SDNode *N, const SDLoc &DL,
5866:                                       SelectionDAG &DAG) {
5867:   assert(N->getOpcode() == ISD::AND && "Unexpected opcode combine into ANDN");
5868: 
5869:   MVT VT = N->getSimpleValueType(0);
5870:   if (!VT.is128BitVector() && !VT.is256BitVector())
5871:     return SDValue();
5872: 
5873:   SDValue X, Y;
5874:   SDValue N0 = N->getOperand(0);
5875:   SDValue N1 = N->getOperand(1);
5876: 
5877:   if (SDValue Not = isNOT(N0, DAG)) {
5878:     X = Not;
5879:     Y = N1;
5880:   } else if (SDValue Not = isNOT(N1, DAG)) {
5881:     X = Not;
5882:     Y = N0;
5883:   } else
5884:     return SDValue();
5885: 
5886:   X = DAG.getBitcast(VT, X);
5887:   Y = DAG.getBitcast(VT, Y);
5888:   return DAG.getNode(LoongArchISD::VANDN, DL, VT, X, Y);
5889: }
5890: 
5891: static bool isConstantSplatVector(SDValue N, APInt &SplatValue,
5892:                                   unsigned MinSizeInBits) {
5893:   N = peekThroughBitcasts(N);
5894:   BuildVectorSDNode *Node = dyn_cast<BuildVectorSDNode>(N);
5895: 
5896:   if (!Node)
5897:     return false;
5898: 
5899:   APInt SplatUndef;
5900:   unsigned SplatBitSize;
5901:   bool HasAnyUndefs;
5902: 
5903:   return Node->isConstantSplat(SplatValue, SplatUndef, SplatBitSize,
5904:                                HasAnyUndefs, MinSizeInBits,
5905:                                /*IsBigEndian=*/false);
5906: }
5907: 
5908: static SDValue performADDCombine(SDNode *N, SelectionDAG &DAG,
5909:                                  TargetLowering::DAGCombinerInfo &DCI,
5910:                                  const LoongArchSubtarget &Subtarget) {
5911:   if (DCI.isBeforeLegalizeOps())
5912:     return SDValue();
5913: 
5914:   EVT VT = N->getValueType(0);
5915:   if (!VT.isVector())
5916:     return SDValue();
5917: 
5918:   if (!DAG.getTargetLoweringInfo().isTypeLegal(VT))
5919:     return SDValue();
5920: 
5921:   EVT EltVT = VT.getVectorElementType();
5922:   if (!EltVT.isInteger())
5923:     return SDValue();
5924: 
5925:   // match:
```
- **EN**: The range implements or declares functions including `combineAndNotIntoVANDN`, `isConstantSplatVector`, `performADDCombine`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `combineAndNotIntoVANDN`, `isConstantSplatVector`, `performADDCombine` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 5926-6000 / 第 5926-6000 行
```cpp
5926:   //
5927:   //   add
5928:   //     (and
5929:   //        (srl X, shift-1) / X
5930:   //        1)
5931:   //     (srl/sra X, shift)
5932: 
5933:   SDValue Add0 = N->getOperand(0);
5934:   SDValue Add1 = N->getOperand(1);
5935:   SDValue And;
5936:   SDValue Shr;
5937: 
5938:   if (Add0.getOpcode() == ISD::AND) {
5939:     And = Add0;
5940:     Shr = Add1;
5941:   } else if (Add1.getOpcode() == ISD::AND) {
5942:     And = Add1;
5943:     Shr = Add0;
5944:   } else {
5945:     return SDValue();
5946:   }
5947: 
5948:   // match:
5949:   //
5950:   //   srl/sra X, shift
5951: 
5952:   if (Shr.getOpcode() != ISD::SRL && Shr.getOpcode() != ISD::SRA)
5953:     return SDValue();
5954: 
5955:   SDValue X = Shr.getOperand(0);
5956:   SDValue Shift = Shr.getOperand(1);
5957:   APInt ShiftVal;
5958: 
5959:   if (!isConstantSplatVector(Shift, ShiftVal, EltVT.getSizeInBits()))
5960:     return SDValue();
5961: 
5962:   if (ShiftVal == 0)
5963:     return SDValue();
5964: 
5965:   // match:
5966:   //
5967:   //   and
5968:   //      (srl X, shift-1) / X
5969:   //      1
5970: 
5971:   SDValue One = And.getOperand(1);
5972:   APInt SplatVal;
5973: 
5974:   if (!isConstantSplatVector(One, SplatVal, EltVT.getSizeInBits()))
5975:     return SDValue();
5976: 
5977:   if (SplatVal != 1)
5978:     return SDValue();
5979: 
5980:   if (And.getOperand(0) == X) {
5981:     // match:
5982:     //
5983:     // shift == 1
5984: 
5985:     if (ShiftVal != 1)
5986:       return SDValue();
5987:   } else {
5988:     // match:
5989:     //
5990:     // srl X, shift-1
5991: 
5992:     SDValue Srl = And.getOperand(0);
5993: 
5994:     if (Srl.getOpcode() != ISD::SRL)
5995:       return SDValue();
5996: 
5997:     if (Srl.getOperand(0) != X)
5998:       return SDValue();
5999: 
6000:     // match:
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 6001-6075 / 第 6001-6075 行
```cpp
6001:     //
6002:     // shift-1
6003: 
6004:     SDValue ShiftMinus1 = Srl.getOperand(1);
6005: 
6006:     if (!isConstantSplatVector(ShiftMinus1, SplatVal, EltVT.getSizeInBits()))
6007:       return SDValue();
6008: 
6009:     if (ShiftVal != (SplatVal + 1))
6010:       return SDValue();
6011:   }
6012: 
6013:   // We matched a rounded right shift pattern and can lower it
6014:   // to a single vector rounded shift instruction.
6015: 
6016:   SDLoc DL(N);
6017:   return DAG.getNode(Shr.getOpcode() == ISD::SRL ? LoongArchISD::VSRLR
6018:                                                  : LoongArchISD::VSRAR,
6019:                      DL, VT, X, Shift);
6020: }
6021: 
6022: static SDValue performANDCombine(SDNode *N, SelectionDAG &DAG,
6023:                                  TargetLowering::DAGCombinerInfo &DCI,
6024:                                  const LoongArchSubtarget &Subtarget) {
6025:   if (DCI.isBeforeLegalizeOps())
6026:     return SDValue();
6027: 
6028:   SDValue FirstOperand = N->getOperand(0);
6029:   SDValue SecondOperand = N->getOperand(1);
6030:   unsigned FirstOperandOpc = FirstOperand.getOpcode();
6031:   EVT ValTy = N->getValueType(0);
6032:   SDLoc DL(N);
6033:   uint64_t lsb, msb;
6034:   unsigned SMIdx, SMLen;
6035:   ConstantSDNode *CN;
6036:   SDValue NewOperand;
6037:   MVT GRLenVT = Subtarget.getGRLenVT();
6038: 
6039:   if (SDValue R = combineAndNotIntoVANDN(N, DL, DAG))
6040:     return R;
6041: 
6042:   // BSTRPICK requires the 32S feature.
6043:   if (!Subtarget.has32S())
6044:     return SDValue();
6045: 
6046:   // Op's second operand must be a shifted mask.
6047:   if (!(CN = dyn_cast<ConstantSDNode>(SecondOperand)) ||
6048:       !isShiftedMask_64(CN->getZExtValue(), SMIdx, SMLen))
6049:     return SDValue();
6050: 
6051:   if (FirstOperandOpc == ISD::SRA || FirstOperandOpc == ISD::SRL) {
6052:     // Pattern match BSTRPICK.
6053:     //  $dst = and ((sra or srl) $src , lsb), (2**len - 1)
6054:     //  => BSTRPICK $dst, $src, msb, lsb
6055:     //  where msb = lsb + len - 1
6056: 
6057:     // The second operand of the shift must be an immediate.
6058:     if (!(CN = dyn_cast<ConstantSDNode>(FirstOperand.getOperand(1))))
6059:       return SDValue();
6060: 
6061:     lsb = CN->getZExtValue();
6062: 
6063:     // Return if the shifted mask does not start at bit 0 or the sum of its
6064:     // length and lsb exceeds the word's size.
6065:     if (SMIdx != 0 || lsb + SMLen > ValTy.getSizeInBits())
6066:       return SDValue();
6067: 
6068:     NewOperand = FirstOperand.getOperand(0);
6069:   } else {
6070:     // Pattern match BSTRPICK.
6071:     //  $dst = and $src, (2**len- 1) , if len > 12
6072:     //  => BSTRPICK $dst, $src, msb, lsb
6073:     //  where lsb = 0 and msb = len - 1
6074: 
6075:     // If the mask is <= 0xfff, andi can be used instead.
```
- **EN**: The range implements or declares functions including `performANDCombine`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `performANDCombine` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 6076-6150 / 第 6076-6150 行
```cpp
6076:     if (CN->getZExtValue() <= 0xfff)
6077:       return SDValue();
6078: 
6079:     // Return if the MSB exceeds.
6080:     if (SMIdx + SMLen > ValTy.getSizeInBits())
6081:       return SDValue();
6082: 
6083:     if (SMIdx > 0) {
6084:       // Omit if the constant has more than 2 uses. This a conservative
6085:       // decision. Whether it is a win depends on the HW microarchitecture.
6086:       // However it should always be better for 1 and 2 uses.
6087:       if (CN->use_size() > 2)
6088:         return SDValue();
6089:       // Return if the constant can be composed by a single LU12I.W.
6090:       if ((CN->getZExtValue() & 0xfff) == 0)
6091:         return SDValue();
6092:       // Return if the constand can be composed by a single ADDI with
6093:       // the zero register.
6094:       if (CN->getSExtValue() >= -2048 && CN->getSExtValue() < 0)
6095:         return SDValue();
6096:     }
6097: 
6098:     lsb = SMIdx;
6099:     NewOperand = FirstOperand;
6100:   }
6101: 
6102:   msb = lsb + SMLen - 1;
6103:   SDValue NR0 = DAG.getNode(LoongArchISD::BSTRPICK, DL, ValTy, NewOperand,
6104:                             DAG.getConstant(msb, DL, GRLenVT),
6105:                             DAG.getConstant(lsb, DL, GRLenVT));
6106:   if (FirstOperandOpc == ISD::SRA || FirstOperandOpc == ISD::SRL || lsb == 0)
6107:     return NR0;
6108:   // Try to optimize to
6109:   //   bstrpick $Rd, $Rs, msb, lsb
6110:   //   slli     $Rd, $Rd, lsb
6111:   return DAG.getNode(ISD::SHL, DL, ValTy, NR0,
6112:                      DAG.getConstant(lsb, DL, GRLenVT));
6113: }
6114: 
6115: static SDValue performSRLCombine(SDNode *N, SelectionDAG &DAG,
6116:                                  TargetLowering::DAGCombinerInfo &DCI,
6117:                                  const LoongArchSubtarget &Subtarget) {
6118:   // BSTRPICK requires the 32S feature.
6119:   if (!Subtarget.has32S())
6120:     return SDValue();
6121: 
6122:   if (DCI.isBeforeLegalizeOps())
6123:     return SDValue();
6124: 
6125:   // $dst = srl (and $src, Mask), Shamt
6126:   // =>
6127:   // BSTRPICK $dst, $src, MaskIdx+MaskLen-1, Shamt
6128:   // when Mask is a shifted mask, and MaskIdx <= Shamt <= MaskIdx+MaskLen-1
6129:   //
6130: 
6131:   SDValue FirstOperand = N->getOperand(0);
6132:   ConstantSDNode *CN;
6133:   EVT ValTy = N->getValueType(0);
6134:   SDLoc DL(N);
6135:   MVT GRLenVT = Subtarget.getGRLenVT();
6136:   unsigned MaskIdx, MaskLen;
6137:   uint64_t Shamt;
6138: 
6139:   // The first operand must be an AND and the second operand of the AND must be
6140:   // a shifted mask.
6141:   if (FirstOperand.getOpcode() != ISD::AND ||
6142:       !(CN = dyn_cast<ConstantSDNode>(FirstOperand.getOperand(1))) ||
6143:       !isShiftedMask_64(CN->getZExtValue(), MaskIdx, MaskLen))
6144:     return SDValue();
6145: 
6146:   // The second operand (shift amount) must be an immediate.
6147:   if (!(CN = dyn_cast<ConstantSDNode>(N->getOperand(1))))
6148:     return SDValue();
6149: 
6150:   Shamt = CN->getZExtValue();
```
- **EN**: The range implements or declares functions including `performSRLCombine`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `performSRLCombine` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 6151-6225 / 第 6151-6225 行
```cpp
6151:   if (MaskIdx <= Shamt && Shamt <= MaskIdx + MaskLen - 1)
6152:     return DAG.getNode(LoongArchISD::BSTRPICK, DL, ValTy,
6153:                        FirstOperand->getOperand(0),
6154:                        DAG.getConstant(MaskIdx + MaskLen - 1, DL, GRLenVT),
6155:                        DAG.getConstant(Shamt, DL, GRLenVT));
6156: 
6157:   return SDValue();
6158: }
6159: 
6160: // Helper to peek through bitops/trunc/setcc to determine size of source vector.
6161: // Allows BITCASTCombine to determine what size vector generated a <X x i1>.
6162: static bool checkBitcastSrcVectorSize(SDValue Src, unsigned Size,
6163:                                       unsigned Depth) {
6164:   // Limit recursion.
6165:   if (Depth >= SelectionDAG::MaxRecursionDepth)
6166:     return false;
6167:   switch (Src.getOpcode()) {
6168:   case ISD::SETCC:
6169:   case ISD::TRUNCATE:
6170:     return Src.getOperand(0).getValueSizeInBits() == Size;
6171:   case ISD::FREEZE:
6172:     return checkBitcastSrcVectorSize(Src.getOperand(0), Size, Depth + 1);
6173:   case ISD::AND:
6174:   case ISD::XOR:
6175:   case ISD::OR:
6176:     return checkBitcastSrcVectorSize(Src.getOperand(0), Size, Depth + 1) &&
6177:            checkBitcastSrcVectorSize(Src.getOperand(1), Size, Depth + 1);
6178:   case ISD::SELECT:
6179:   case ISD::VSELECT:
6180:     return Src.getOperand(0).getScalarValueSizeInBits() == 1 &&
6181:            checkBitcastSrcVectorSize(Src.getOperand(1), Size, Depth + 1) &&
6182:            checkBitcastSrcVectorSize(Src.getOperand(2), Size, Depth + 1);
6183:   case ISD::BUILD_VECTOR:
6184:     return ISD::isBuildVectorAllZeros(Src.getNode()) ||
6185:            ISD::isBuildVectorAllOnes(Src.getNode());
6186:   }
6187:   return false;
6188: }
6189: 
6190: // Helper to push sign extension of vXi1 SETCC result through bitops.
6191: static SDValue signExtendBitcastSrcVector(SelectionDAG &DAG, EVT SExtVT,
6192:                                           SDValue Src, const SDLoc &DL) {
6193:   switch (Src.getOpcode()) {
6194:   case ISD::SETCC:
6195:   case ISD::FREEZE:
6196:   case ISD::TRUNCATE:
6197:   case ISD::BUILD_VECTOR:
6198:     return DAG.getNode(ISD::SIGN_EXTEND, DL, SExtVT, Src);
6199:   case ISD::AND:
6200:   case ISD::XOR:
6201:   case ISD::OR:
6202:     return DAG.getNode(
6203:         Src.getOpcode(), DL, SExtVT,
6204:         signExtendBitcastSrcVector(DAG, SExtVT, Src.getOperand(0), DL),
6205:         signExtendBitcastSrcVector(DAG, SExtVT, Src.getOperand(1), DL));
6206:   case ISD::SELECT:
6207:   case ISD::VSELECT:
6208:     return DAG.getSelect(
6209:         DL, SExtVT, Src.getOperand(0),
6210:         signExtendBitcastSrcVector(DAG, SExtVT, Src.getOperand(1), DL),
6211:         signExtendBitcastSrcVector(DAG, SExtVT, Src.getOperand(2), DL));
6212:   }
6213:   llvm_unreachable("Unexpected node type for vXi1 sign extension");
6214: }
6215: 
6216: static SDValue
6217: performSETCC_BITCASTCombine(SDNode *N, SelectionDAG &DAG,
6218:                             TargetLowering::DAGCombinerInfo &DCI,
6219:                             const LoongArchSubtarget &Subtarget) {
6220:   SDLoc DL(N);
6221:   EVT VT = N->getValueType(0);
6222:   SDValue Src = N->getOperand(0);
6223:   EVT SrcVT = Src.getValueType();
6224: 
6225:   if (Src.getOpcode() != ISD::SETCC || !Src.hasOneUse())
```
- **EN**: The range implements or declares functions including `checkBitcastSrcVectorSize`, `signExtendBitcastSrcVector`, `performSETCC_BITCASTCombine`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `checkBitcastSrcVectorSize`, `signExtendBitcastSrcVector`, `performSETCC_BITCASTCombine` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 6226-6300 / 第 6226-6300 行
```cpp
6226:     return SDValue();
6227: 
6228:   bool UseLASX;
6229:   unsigned Opc = ISD::DELETED_NODE;
6230:   EVT CmpVT = Src.getOperand(0).getValueType();
6231:   EVT EltVT = CmpVT.getVectorElementType();
6232: 
6233:   if (Subtarget.hasExtLSX() && CmpVT.getSizeInBits() == 128)
6234:     UseLASX = false;
6235:   else if (Subtarget.has32S() && Subtarget.hasExtLASX() &&
6236:            CmpVT.getSizeInBits() == 256)
6237:     UseLASX = true;
6238:   else
6239:     return SDValue();
6240: 
6241:   SDValue SrcN1 = Src.getOperand(1);
6242:   switch (cast<CondCodeSDNode>(Src.getOperand(2))->get()) {
6243:   default:
6244:     break;
6245:   case ISD::SETEQ:
6246:     // x == 0 => not (vmsknez.b x)
6247:     if (ISD::isBuildVectorAllZeros(SrcN1.getNode()) && EltVT == MVT::i8)
6248:       Opc = UseLASX ? LoongArchISD::XVMSKEQZ : LoongArchISD::VMSKEQZ;
6249:     break;
6250:   case ISD::SETGT:
6251:     // x > -1 => vmskgez.b x
6252:     if (ISD::isBuildVectorAllOnes(SrcN1.getNode()) && EltVT == MVT::i8)
6253:       Opc = UseLASX ? LoongArchISD::XVMSKGEZ : LoongArchISD::VMSKGEZ;
6254:     break;
6255:   case ISD::SETGE:
6256:     // x >= 0 => vmskgez.b x
6257:     if (ISD::isBuildVectorAllZeros(SrcN1.getNode()) && EltVT == MVT::i8)
6258:       Opc = UseLASX ? LoongArchISD::XVMSKGEZ : LoongArchISD::VMSKGEZ;
6259:     break;
6260:   case ISD::SETLT:
6261:     // x < 0 => vmskltz.{b,h,w,d} x
6262:     if (ISD::isBuildVectorAllZeros(SrcN1.getNode()) &&
6263:         (EltVT == MVT::i8 || EltVT == MVT::i16 || EltVT == MVT::i32 ||
6264:          EltVT == MVT::i64))
6265:       Opc = UseLASX ? LoongArchISD::XVMSKLTZ : LoongArchISD::VMSKLTZ;
6266:     break;
6267:   case ISD::SETLE:
6268:     // x <= -1 => vmskltz.{b,h,w,d} x
6269:     if (ISD::isBuildVectorAllOnes(SrcN1.getNode()) &&
6270:         (EltVT == MVT::i8 || EltVT == MVT::i16 || EltVT == MVT::i32 ||
6271:          EltVT == MVT::i64))
6272:       Opc = UseLASX ? LoongArchISD::XVMSKLTZ : LoongArchISD::VMSKLTZ;
6273:     break;
6274:   case ISD::SETNE:
6275:     // x != 0 => vmsknez.b x
6276:     if (ISD::isBuildVectorAllZeros(SrcN1.getNode()) && EltVT == MVT::i8)
6277:       Opc = UseLASX ? LoongArchISD::XVMSKNEZ : LoongArchISD::VMSKNEZ;
6278:     break;
6279:   }
6280: 
6281:   if (Opc == ISD::DELETED_NODE)
6282:     return SDValue();
6283: 
6284:   SDValue V = DAG.getNode(Opc, DL, Subtarget.getGRLenVT(), Src.getOperand(0));
6285:   EVT T = EVT::getIntegerVT(*DAG.getContext(), SrcVT.getVectorNumElements());
6286:   V = DAG.getZExtOrTrunc(V, DL, T);
6287:   return DAG.getBitcast(VT, V);
6288: }
6289: 
6290: static SDValue performBITCASTCombine(SDNode *N, SelectionDAG &DAG,
6291:                                      TargetLowering::DAGCombinerInfo &DCI,
6292:                                      const LoongArchSubtarget &Subtarget) {
6293:   SDLoc DL(N);
6294:   EVT VT = N->getValueType(0);
6295:   SDValue Src = N->getOperand(0);
6296:   EVT SrcVT = Src.getValueType();
6297:   MVT GRLenVT = Subtarget.getGRLenVT();
6298: 
6299:   if (!DCI.isBeforeLegalizeOps())
6300:     return SDValue();
```
- **EN**: The range implements or declares functions including `performBITCASTCombine`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `performBITCASTCombine` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 6301-6375 / 第 6301-6375 行
```cpp
6301: 
6302:   if (!SrcVT.isSimple() || SrcVT.getScalarType() != MVT::i1)
6303:     return SDValue();
6304: 
6305:   // Combine SETCC and BITCAST into [X]VMSK{LT,GE,NE} when possible
6306:   SDValue Res = performSETCC_BITCASTCombine(N, DAG, DCI, Subtarget);
6307:   if (Res)
6308:     return Res;
6309: 
6310:   // Generate vXi1 using [X]VMSKLTZ
6311:   MVT SExtVT;
6312:   unsigned Opc;
6313:   bool UseLASX = false;
6314:   bool PropagateSExt = false;
6315: 
6316:   if (Src.getOpcode() == ISD::SETCC && Src.hasOneUse()) {
6317:     EVT CmpVT = Src.getOperand(0).getValueType();
6318:     if (CmpVT.getSizeInBits() > 256)
6319:       return SDValue();
6320:   }
6321: 
6322:   switch (SrcVT.getSimpleVT().SimpleTy) {
6323:   default:
6324:     return SDValue();
6325:   case MVT::v2i1:
6326:     SExtVT = MVT::v2i64;
6327:     break;
6328:   case MVT::v4i1:
6329:     SExtVT = MVT::v4i32;
6330:     if (Subtarget.hasExtLASX() && checkBitcastSrcVectorSize(Src, 256, 0)) {
6331:       SExtVT = MVT::v4i64;
6332:       UseLASX = true;
6333:       PropagateSExt = true;
6334:     }
6335:     break;
6336:   case MVT::v8i1:
6337:     SExtVT = MVT::v8i16;
6338:     if (Subtarget.hasExtLASX() && checkBitcastSrcVectorSize(Src, 256, 0)) {
6339:       SExtVT = MVT::v8i32;
6340:       UseLASX = true;
6341:       PropagateSExt = true;
6342:     }
6343:     break;
6344:   case MVT::v16i1:
6345:     SExtVT = MVT::v16i8;
6346:     if (Subtarget.hasExtLASX() && checkBitcastSrcVectorSize(Src, 256, 0)) {
6347:       SExtVT = MVT::v16i16;
6348:       UseLASX = true;
6349:       PropagateSExt = true;
6350:     }
6351:     break;
6352:   case MVT::v32i1:
6353:     SExtVT = MVT::v32i8;
6354:     UseLASX = true;
6355:     break;
6356:   };
6357:   Src = PropagateSExt ? signExtendBitcastSrcVector(DAG, SExtVT, Src, DL)
6358:                       : DAG.getNode(ISD::SIGN_EXTEND, DL, SExtVT, Src);
6359: 
6360:   SDValue V;
6361:   if (!Subtarget.has32S() || !Subtarget.hasExtLASX()) {
6362:     if (Src.getSimpleValueType() == MVT::v32i8) {
6363:       SDValue Lo, Hi;
6364:       std::tie(Lo, Hi) = DAG.SplitVector(Src, DL);
6365:       Lo = DAG.getNode(LoongArchISD::VMSKLTZ, DL, GRLenVT, Lo);
6366:       Hi = DAG.getNode(LoongArchISD::VMSKLTZ, DL, GRLenVT, Hi);
6367:       Hi = DAG.getNode(ISD::SHL, DL, GRLenVT, Hi,
6368:                        DAG.getShiftAmountConstant(16, GRLenVT, DL));
6369:       V = DAG.getNode(ISD::OR, DL, GRLenVT, Lo, Hi);
6370:     } else if (UseLASX) {
6371:       return SDValue();
6372:     }
6373:   }
6374: 
6375:   if (!V) {
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 6376-6450 / 第 6376-6450 行
```cpp
6376:     Opc = UseLASX ? LoongArchISD::XVMSKLTZ : LoongArchISD::VMSKLTZ;
6377:     V = DAG.getNode(Opc, DL, GRLenVT, Src);
6378:   }
6379: 
6380:   EVT T = EVT::getIntegerVT(*DAG.getContext(), SrcVT.getVectorNumElements());
6381:   V = DAG.getZExtOrTrunc(V, DL, T);
6382:   return DAG.getBitcast(VT, V);
6383: }
6384: 
6385: static SDValue performORCombine(SDNode *N, SelectionDAG &DAG,
6386:                                 TargetLowering::DAGCombinerInfo &DCI,
6387:                                 const LoongArchSubtarget &Subtarget) {
6388:   MVT GRLenVT = Subtarget.getGRLenVT();
6389:   EVT ValTy = N->getValueType(0);
6390:   SDValue N0 = N->getOperand(0), N1 = N->getOperand(1);
6391:   ConstantSDNode *CN0, *CN1;
6392:   SDLoc DL(N);
6393:   unsigned ValBits = ValTy.getSizeInBits();
6394:   unsigned MaskIdx0, MaskLen0, MaskIdx1, MaskLen1;
6395:   unsigned Shamt;
6396:   bool SwapAndRetried = false;
6397: 
6398:   // BSTRPICK requires the 32S feature.
6399:   if (!Subtarget.has32S())
6400:     return SDValue();
6401: 
6402:   if (DCI.isBeforeLegalizeOps())
6403:     return SDValue();
6404: 
6405:   if (ValBits != 32 && ValBits != 64)
6406:     return SDValue();
6407: 
6408: Retry:
6409:   // 1st pattern to match BSTRINS:
6410:   //  R = or (and X, mask0), (and (shl Y, lsb), mask1)
6411:   //  where mask1 = (2**size - 1) << lsb, mask0 = ~mask1
6412:   //  =>
6413:   //  R = BSTRINS X, Y, msb, lsb (where msb = lsb + size - 1)
6414:   if (N0.getOpcode() == ISD::AND &&
6415:       (CN0 = dyn_cast<ConstantSDNode>(N0.getOperand(1))) &&
6416:       isShiftedMask_64(~CN0->getSExtValue(), MaskIdx0, MaskLen0) &&
6417:       N1.getOpcode() == ISD::AND && N1.getOperand(0).getOpcode() == ISD::SHL &&
6418:       (CN1 = dyn_cast<ConstantSDNode>(N1.getOperand(1))) &&
6419:       isShiftedMask_64(CN1->getZExtValue(), MaskIdx1, MaskLen1) &&
6420:       MaskIdx0 == MaskIdx1 && MaskLen0 == MaskLen1 &&
6421:       (CN1 = dyn_cast<ConstantSDNode>(N1.getOperand(0).getOperand(1))) &&
6422:       (Shamt = CN1->getZExtValue()) == MaskIdx0 &&
6423:       (MaskIdx0 + MaskLen0 <= ValBits)) {
6424:     LLVM_DEBUG(dbgs() << "Perform OR combine: match pattern 1\n");
6425:     return DAG.getNode(LoongArchISD::BSTRINS, DL, ValTy, N0.getOperand(0),
6426:                        N1.getOperand(0).getOperand(0),
6427:                        DAG.getConstant((MaskIdx0 + MaskLen0 - 1), DL, GRLenVT),
6428:                        DAG.getConstant(MaskIdx0, DL, GRLenVT));
6429:   }
6430: 
6431:   // 2nd pattern to match BSTRINS:
6432:   //  R = or (and X, mask0), (shl (and Y, mask1), lsb)
6433:   //  where mask1 = (2**size - 1), mask0 = ~(mask1 << lsb)
6434:   //  =>
6435:   //  R = BSTRINS X, Y, msb, lsb (where msb = lsb + size - 1)
6436:   if (N0.getOpcode() == ISD::AND &&
6437:       (CN0 = dyn_cast<ConstantSDNode>(N0.getOperand(1))) &&
6438:       isShiftedMask_64(~CN0->getSExtValue(), MaskIdx0, MaskLen0) &&
6439:       N1.getOpcode() == ISD::SHL && N1.getOperand(0).getOpcode() == ISD::AND &&
6440:       (CN1 = dyn_cast<ConstantSDNode>(N1.getOperand(1))) &&
6441:       (Shamt = CN1->getZExtValue()) == MaskIdx0 &&
6442:       (CN1 = dyn_cast<ConstantSDNode>(N1.getOperand(0).getOperand(1))) &&
6443:       isShiftedMask_64(CN1->getZExtValue(), MaskIdx1, MaskLen1) &&
6444:       MaskLen0 == MaskLen1 && MaskIdx1 == 0 &&
6445:       (MaskIdx0 + MaskLen0 <= ValBits)) {
6446:     LLVM_DEBUG(dbgs() << "Perform OR combine: match pattern 2\n");
6447:     return DAG.getNode(LoongArchISD::BSTRINS, DL, ValTy, N0.getOperand(0),
6448:                        N1.getOperand(0).getOperand(0),
6449:                        DAG.getConstant((MaskIdx0 + MaskLen0 - 1), DL, GRLenVT),
6450:                        DAG.getConstant(MaskIdx0, DL, GRLenVT));
```
- **EN**: The range implements or declares functions including `performORCombine`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `performORCombine` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 6451-6525 / 第 6451-6525 行
```cpp
6451:   }
6452: 
6453:   // 3rd pattern to match BSTRINS:
6454:   //  R = or (and X, mask0), (and Y, mask1)
6455:   //  where ~mask0 = (2**size - 1) << lsb, mask0 & mask1 = 0
6456:   //  =>
6457:   //  R = BSTRINS X, (shr (and Y, mask1), lsb), msb, lsb
6458:   //  where msb = lsb + size - 1
6459:   if (N0.getOpcode() == ISD::AND && N1.getOpcode() == ISD::AND &&
6460:       (CN0 = dyn_cast<ConstantSDNode>(N0.getOperand(1))) &&
6461:       isShiftedMask_64(~CN0->getSExtValue(), MaskIdx0, MaskLen0) &&
6462:       (MaskIdx0 + MaskLen0 <= 64) &&
6463:       (CN1 = dyn_cast<ConstantSDNode>(N1->getOperand(1))) &&
6464:       (CN1->getSExtValue() & CN0->getSExtValue()) == 0) {
6465:     LLVM_DEBUG(dbgs() << "Perform OR combine: match pattern 3\n");
6466:     return DAG.getNode(LoongArchISD::BSTRINS, DL, ValTy, N0.getOperand(0),
6467:                        DAG.getNode(ISD::SRL, DL, N1->getValueType(0), N1,
6468:                                    DAG.getConstant(MaskIdx0, DL, GRLenVT)),
6469:                        DAG.getConstant(ValBits == 32
6470:                                            ? (MaskIdx0 + (MaskLen0 & 31) - 1)
6471:                                            : (MaskIdx0 + MaskLen0 - 1),
6472:                                        DL, GRLenVT),
6473:                        DAG.getConstant(MaskIdx0, DL, GRLenVT));
6474:   }
6475: 
6476:   // 4th pattern to match BSTRINS:
6477:   //  R = or (and X, mask), (shl Y, shamt)
6478:   //  where mask = (2**shamt - 1)
6479:   //  =>
6480:   //  R = BSTRINS X, Y, ValBits - 1, shamt
6481:   //  where ValBits = 32 or 64
6482:   if (N0.getOpcode() == ISD::AND && N1.getOpcode() == ISD::SHL &&
6483:       (CN0 = dyn_cast<ConstantSDNode>(N0.getOperand(1))) &&
6484:       isShiftedMask_64(CN0->getZExtValue(), MaskIdx0, MaskLen0) &&
6485:       MaskIdx0 == 0 && (CN1 = dyn_cast<ConstantSDNode>(N1.getOperand(1))) &&
6486:       (Shamt = CN1->getZExtValue()) == MaskLen0 &&
6487:       (MaskIdx0 + MaskLen0 <= ValBits)) {
6488:     LLVM_DEBUG(dbgs() << "Perform OR combine: match pattern 4\n");
6489:     return DAG.getNode(LoongArchISD::BSTRINS, DL, ValTy, N0.getOperand(0),
6490:                        N1.getOperand(0),
6491:                        DAG.getConstant((ValBits - 1), DL, GRLenVT),
6492:                        DAG.getConstant(Shamt, DL, GRLenVT));
6493:   }
6494: 
6495:   // 5th pattern to match BSTRINS:
6496:   //  R = or (and X, mask), const
6497:   //  where ~mask = (2**size - 1) << lsb, mask & const = 0
6498:   //  =>
6499:   //  R = BSTRINS X, (const >> lsb), msb, lsb
6500:   //  where msb = lsb + size - 1
6501:   if (N0.getOpcode() == ISD::AND &&
6502:       (CN0 = dyn_cast<ConstantSDNode>(N0.getOperand(1))) &&
6503:       isShiftedMask_64(~CN0->getSExtValue(), MaskIdx0, MaskLen0) &&
6504:       (CN1 = dyn_cast<ConstantSDNode>(N1)) &&
6505:       (CN1->getSExtValue() & CN0->getSExtValue()) == 0) {
6506:     LLVM_DEBUG(dbgs() << "Perform OR combine: match pattern 5\n");
6507:     return DAG.getNode(
6508:         LoongArchISD::BSTRINS, DL, ValTy, N0.getOperand(0),
6509:         DAG.getSignedConstant(CN1->getSExtValue() >> MaskIdx0, DL, ValTy),
6510:         DAG.getConstant(ValBits == 32 ? (MaskIdx0 + (MaskLen0 & 31) - 1)
6511:                                       : (MaskIdx0 + MaskLen0 - 1),
6512:                         DL, GRLenVT),
6513:         DAG.getConstant(MaskIdx0, DL, GRLenVT));
6514:   }
6515: 
6516:   // 6th pattern.
6517:   // a = b | ((c & mask) << shamt), where all positions in b to be overwritten
6518:   // by the incoming bits are known to be zero.
6519:   // =>
6520:   // a = BSTRINS b, c, shamt + MaskLen - 1, shamt
6521:   //
6522:   // Note that the 1st pattern is a special situation of the 6th, i.e. the 6th
6523:   // pattern is more common than the 1st. So we put the 1st before the 6th in
6524:   // order to match as many nodes as possible.
6525:   ConstantSDNode *CNMask, *CNShamt;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 6526-6600 / 第 6526-6600 行
```cpp
6526:   unsigned MaskIdx, MaskLen;
6527:   if (N1.getOpcode() == ISD::SHL && N1.getOperand(0).getOpcode() == ISD::AND &&
6528:       (CNMask = dyn_cast<ConstantSDNode>(N1.getOperand(0).getOperand(1))) &&
6529:       isShiftedMask_64(CNMask->getZExtValue(), MaskIdx, MaskLen) &&
6530:       MaskIdx == 0 && (CNShamt = dyn_cast<ConstantSDNode>(N1.getOperand(1))) &&
6531:       CNShamt->getZExtValue() + MaskLen <= ValBits) {
6532:     Shamt = CNShamt->getZExtValue();
6533:     APInt ShMask(ValBits, CNMask->getZExtValue() << Shamt);
6534:     if (ShMask.isSubsetOf(DAG.computeKnownBits(N0).Zero)) {
6535:       LLVM_DEBUG(dbgs() << "Perform OR combine: match pattern 6\n");
6536:       return DAG.getNode(LoongArchISD::BSTRINS, DL, ValTy, N0,
6537:                          N1.getOperand(0).getOperand(0),
6538:                          DAG.getConstant(Shamt + MaskLen - 1, DL, GRLenVT),
6539:                          DAG.getConstant(Shamt, DL, GRLenVT));
6540:     }
6541:   }
6542: 
6543:   // 7th pattern.
6544:   // a = b | ((c << shamt) & shifted_mask), where all positions in b to be
6545:   // overwritten by the incoming bits are known to be zero.
6546:   // =>
6547:   // a = BSTRINS b, c, MaskIdx + MaskLen - 1, MaskIdx
6548:   //
6549:   // Similarly, the 7th pattern is more common than the 2nd. So we put the 2nd
6550:   // before the 7th in order to match as many nodes as possible.
6551:   if (N1.getOpcode() == ISD::AND &&
6552:       (CNMask = dyn_cast<ConstantSDNode>(N1.getOperand(1))) &&
6553:       isShiftedMask_64(CNMask->getZExtValue(), MaskIdx, MaskLen) &&
6554:       N1.getOperand(0).getOpcode() == ISD::SHL &&
6555:       (CNShamt = dyn_cast<ConstantSDNode>(N1.getOperand(0).getOperand(1))) &&
6556:       CNShamt->getZExtValue() == MaskIdx) {
6557:     APInt ShMask(ValBits, CNMask->getZExtValue());
6558:     if (ShMask.isSubsetOf(DAG.computeKnownBits(N0).Zero)) {
6559:       LLVM_DEBUG(dbgs() << "Perform OR combine: match pattern 7\n");
6560:       return DAG.getNode(LoongArchISD::BSTRINS, DL, ValTy, N0,
6561:                          N1.getOperand(0).getOperand(0),
6562:                          DAG.getConstant(MaskIdx + MaskLen - 1, DL, GRLenVT),
6563:                          DAG.getConstant(MaskIdx, DL, GRLenVT));
6564:     }
6565:   }
6566: 
6567:   // (or a, b) and (or b, a) are equivalent, so swap the operands and retry.
6568:   if (!SwapAndRetried) {
6569:     std::swap(N0, N1);
6570:     SwapAndRetried = true;
6571:     goto Retry;
6572:   }
6573: 
6574:   SwapAndRetried = false;
6575: Retry2:
6576:   // 8th pattern.
6577:   // a = b | (c & shifted_mask), where all positions in b to be overwritten by
6578:   // the incoming bits are known to be zero.
6579:   // =>
6580:   // a = BSTRINS b, c >> MaskIdx, MaskIdx + MaskLen - 1, MaskIdx
6581:   //
6582:   // Similarly, the 8th pattern is more common than the 4th and 5th patterns. So
6583:   // we put it here in order to match as many nodes as possible or generate less
6584:   // instructions.
6585:   if (N1.getOpcode() == ISD::AND &&
6586:       (CNMask = dyn_cast<ConstantSDNode>(N1.getOperand(1))) &&
6587:       isShiftedMask_64(CNMask->getZExtValue(), MaskIdx, MaskLen)) {
6588:     APInt ShMask(ValBits, CNMask->getZExtValue());
6589:     if (ShMask.isSubsetOf(DAG.computeKnownBits(N0).Zero)) {
6590:       LLVM_DEBUG(dbgs() << "Perform OR combine: match pattern 8\n");
6591:       return DAG.getNode(LoongArchISD::BSTRINS, DL, ValTy, N0,
6592:                          DAG.getNode(ISD::SRL, DL, N1->getValueType(0),
6593:                                      N1->getOperand(0),
6594:                                      DAG.getConstant(MaskIdx, DL, GRLenVT)),
6595:                          DAG.getConstant(MaskIdx + MaskLen - 1, DL, GRLenVT),
6596:                          DAG.getConstant(MaskIdx, DL, GRLenVT));
6597:     }
6598:   }
6599:   // Swap N0/N1 and retry.
6600:   if (!SwapAndRetried) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 6601-6675 / 第 6601-6675 行
```cpp
6601:     std::swap(N0, N1);
6602:     SwapAndRetried = true;
6603:     goto Retry2;
6604:   }
6605: 
6606:   return SDValue();
6607: }
6608: 
6609: static bool checkValueWidth(SDValue V, ISD::LoadExtType &ExtType) {
6610:   ExtType = ISD::NON_EXTLOAD;
6611: 
6612:   switch (V.getNode()->getOpcode()) {
6613:   case ISD::LOAD: {
6614:     LoadSDNode *LoadNode = cast<LoadSDNode>(V.getNode());
6615:     if ((LoadNode->getMemoryVT() == MVT::i8) ||
6616:         (LoadNode->getMemoryVT() == MVT::i16)) {
6617:       ExtType = LoadNode->getExtensionType();
6618:       return true;
6619:     }
6620:     return false;
6621:   }
6622:   case ISD::AssertSext: {
6623:     VTSDNode *TypeNode = cast<VTSDNode>(V.getNode()->getOperand(1));
6624:     if ((TypeNode->getVT() == MVT::i8) || (TypeNode->getVT() == MVT::i16)) {
6625:       ExtType = ISD::SEXTLOAD;
6626:       return true;
6627:     }
6628:     return false;
6629:   }
6630:   case ISD::AssertZext: {
6631:     VTSDNode *TypeNode = cast<VTSDNode>(V.getNode()->getOperand(1));
6632:     if ((TypeNode->getVT() == MVT::i8) || (TypeNode->getVT() == MVT::i16)) {
6633:       ExtType = ISD::ZEXTLOAD;
6634:       return true;
6635:     }
6636:     return false;
6637:   }
6638:   default:
6639:     return false;
6640:   }
6641: 
6642:   return false;
6643: }
6644: 
6645: // Eliminate redundant truncation and zero-extension nodes.
6646: // * Case 1:
6647: //  +------------+ +------------+ +------------+
6648: //  |   Input1   | |   Input2   | |     CC     |
6649: //  +------------+ +------------+ +------------+
6650: //         |              |              |
6651: //         V              V              +----+
6652: //  +------------+ +------------+             |
6653: //  |  TRUNCATE  | |  TRUNCATE  |             |
6654: //  +------------+ +------------+             |
6655: //         |              |                   |
6656: //         V              V                   |
6657: //  +------------+ +------------+             |
6658: //  |  ZERO_EXT  | |  ZERO_EXT  |             |
6659: //  +------------+ +------------+             |
6660: //         |              |                   |
6661: //         |              +-------------+     |
6662: //         V              V             |     |
6663: //        +----------------+            |     |
6664: //        |      AND       |            |     |
6665: //        +----------------+            |     |
6666: //                |                     |     |
6667: //                +---------------+     |     |
6668: //                                |     |     |
6669: //                                V     V     V
6670: //                               +-------------+
6671: //                               |     CMP     |
6672: //                               +-------------+
6673: // * Case 2:
6674: //  +------------+ +------------+ +-------------+ +------------+ +------------+
6675: //  |   Input1   | |   Input2   | | Constant -1 | | Constant 0 | |     CC     |
```
- **EN**: The range implements or declares functions including `checkValueWidth`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `checkValueWidth` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 6676-6750 / 第 6676-6750 行
```cpp
6676: //  +------------+ +------------+ +-------------+ +------------+ +------------+
6677: //         |              |             |               |               |
6678: //         V              |             |               |               |
6679: //  +------------+        |             |               |               |
6680: //  |     XOR    |<---------------------+               |               |
6681: //  +------------+        |                             |               |
6682: //         |              |                             |               |
6683: //         V              V             +---------------+               |
6684: //  +------------+ +------------+       |                               |
6685: //  |  TRUNCATE  | |  TRUNCATE  |       |     +-------------------------+
6686: //  +------------+ +------------+       |     |
6687: //         |              |             |     |
6688: //         V              V             |     |
6689: //  +------------+ +------------+       |     |
6690: //  |  ZERO_EXT  | |  ZERO_EXT  |       |     |
6691: //  +------------+ +------------+       |     |
6692: //         |              |             |     |
6693: //         V              V             |     |
6694: //        +----------------+            |     |
6695: //        |      AND       |            |     |
6696: //        +----------------+            |     |
6697: //                |                     |     |
6698: //                +---------------+     |     |
6699: //                                |     |     |
6700: //                                V     V     V
6701: //                               +-------------+
6702: //                               |     CMP     |
6703: //                               +-------------+
6704: static SDValue performSETCCCombine(SDNode *N, SelectionDAG &DAG,
6705:                                    TargetLowering::DAGCombinerInfo &DCI,
6706:                                    const LoongArchSubtarget &Subtarget) {
6707:   ISD::CondCode CC = cast<CondCodeSDNode>(N->getOperand(2))->get();
6708: 
6709:   SDNode *AndNode = N->getOperand(0).getNode();
6710:   if (AndNode->getOpcode() != ISD::AND)
6711:     return SDValue();
6712: 
6713:   SDValue AndInputValue2 = AndNode->getOperand(1);
6714:   if (AndInputValue2.getOpcode() != ISD::ZERO_EXTEND)
6715:     return SDValue();
6716: 
6717:   SDValue CmpInputValue = N->getOperand(1);
6718:   SDValue AndInputValue1 = AndNode->getOperand(0);
6719:   if (AndInputValue1.getOpcode() == ISD::XOR) {
6720:     if (CC != ISD::SETEQ && CC != ISD::SETNE)
6721:       return SDValue();
6722:     ConstantSDNode *CN = dyn_cast<ConstantSDNode>(AndInputValue1.getOperand(1));
6723:     if (!CN || !CN->isAllOnes())
6724:       return SDValue();
6725:     CN = dyn_cast<ConstantSDNode>(CmpInputValue);
6726:     if (!CN || !CN->isZero())
6727:       return SDValue();
6728:     AndInputValue1 = AndInputValue1.getOperand(0);
6729:     if (AndInputValue1.getOpcode() != ISD::ZERO_EXTEND)
6730:       return SDValue();
6731:   } else if (AndInputValue1.getOpcode() == ISD::ZERO_EXTEND) {
6732:     if (AndInputValue2 != CmpInputValue)
6733:       return SDValue();
6734:   } else {
6735:     return SDValue();
6736:   }
6737: 
6738:   SDValue TruncValue1 = AndInputValue1.getNode()->getOperand(0);
6739:   if (TruncValue1.getOpcode() != ISD::TRUNCATE)
6740:     return SDValue();
6741: 
6742:   SDValue TruncValue2 = AndInputValue2.getNode()->getOperand(0);
6743:   if (TruncValue2.getOpcode() != ISD::TRUNCATE)
6744:     return SDValue();
6745: 
6746:   SDValue TruncInputValue1 = TruncValue1.getNode()->getOperand(0);
6747:   SDValue TruncInputValue2 = TruncValue2.getNode()->getOperand(0);
6748:   ISD::LoadExtType ExtType1;
6749:   ISD::LoadExtType ExtType2;
6750: 
```
- **EN**: The range implements or declares functions including `performSETCCCombine`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `performSETCCCombine` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 6751-6825 / 第 6751-6825 行
```cpp
6751:   if (!checkValueWidth(TruncInputValue1, ExtType1) ||
6752:       !checkValueWidth(TruncInputValue2, ExtType2))
6753:     return SDValue();
6754: 
6755:   if (TruncInputValue1->getValueType(0) != TruncInputValue2->getValueType(0) ||
6756:       AndNode->getValueType(0) != TruncInputValue1->getValueType(0))
6757:     return SDValue();
6758: 
6759:   if ((ExtType2 != ISD::ZEXTLOAD) &&
6760:       ((ExtType2 != ISD::SEXTLOAD) && (ExtType1 != ISD::SEXTLOAD)))
6761:     return SDValue();
6762: 
6763:   // These truncation and zero-extension nodes are not necessary, remove them.
6764:   SDValue NewAnd = DAG.getNode(ISD::AND, SDLoc(N), AndNode->getValueType(0),
6765:                                TruncInputValue1, TruncInputValue2);
6766:   SDValue NewSetCC =
6767:       DAG.getSetCC(SDLoc(N), N->getValueType(0), NewAnd, TruncInputValue2, CC);
6768:   DAG.ReplaceAllUsesWith(N, NewSetCC.getNode());
6769:   return SDValue(N, 0);
6770: }
6771: 
6772: // Combine (loongarch_bitrev_w (loongarch_revb_2w X)) to loongarch_bitrev_4b.
6773: static SDValue performBITREV_WCombine(SDNode *N, SelectionDAG &DAG,
6774:                                       TargetLowering::DAGCombinerInfo &DCI,
6775:                                       const LoongArchSubtarget &Subtarget) {
6776:   if (DCI.isBeforeLegalizeOps())
6777:     return SDValue();
6778: 
6779:   SDValue Src = N->getOperand(0);
6780:   if (Src.getOpcode() != LoongArchISD::REVB_2W)
6781:     return SDValue();
6782: 
6783:   return DAG.getNode(LoongArchISD::BITREV_4B, SDLoc(N), N->getValueType(0),
6784:                      Src.getOperand(0));
6785: }
6786: 
6787: // Perform common combines for BR_CC and SELECT_CC conditions.
6788: static bool combine_CC(SDValue &LHS, SDValue &RHS, SDValue &CC, const SDLoc &DL,
6789:                        SelectionDAG &DAG, const LoongArchSubtarget &Subtarget) {
6790:   ISD::CondCode CCVal = cast<CondCodeSDNode>(CC)->get();
6791: 
6792:   // As far as arithmetic right shift always saves the sign,
6793:   // shift can be omitted.
6794:   // Fold setlt (sra X, N), 0 -> setlt X, 0 and
6795:   // setge (sra X, N), 0 -> setge X, 0
6796:   if (isNullConstant(RHS) && (CCVal == ISD::SETGE || CCVal == ISD::SETLT) &&
6797:       LHS.getOpcode() == ISD::SRA) {
6798:     LHS = LHS.getOperand(0);
6799:     return true;
6800:   }
6801: 
6802:   if (!ISD::isIntEqualitySetCC(CCVal))
6803:     return false;
6804: 
6805:   // Fold ((setlt X, Y), 0, ne) -> (X, Y, lt)
6806:   // Sometimes the setcc is introduced after br_cc/select_cc has been formed.
6807:   if (LHS.getOpcode() == ISD::SETCC && isNullConstant(RHS) &&
6808:       LHS.getOperand(0).getValueType() == Subtarget.getGRLenVT()) {
6809:     // If we're looking for eq 0 instead of ne 0, we need to invert the
6810:     // condition.
6811:     bool Invert = CCVal == ISD::SETEQ;
6812:     CCVal = cast<CondCodeSDNode>(LHS.getOperand(2))->get();
6813:     if (Invert)
6814:       CCVal = ISD::getSetCCInverse(CCVal, LHS.getValueType());
6815: 
6816:     RHS = LHS.getOperand(1);
6817:     LHS = LHS.getOperand(0);
6818:     translateSetCCForBranch(DL, LHS, RHS, CCVal, DAG);
6819: 
6820:     CC = DAG.getCondCode(CCVal);
6821:     return true;
6822:   }
6823: 
6824:   // Fold ((srl (and X, 1<<C), C), 0, eq/ne) -> ((shl X, GRLen-1-C), 0, ge/lt)
6825:   if (isNullConstant(RHS) && LHS.getOpcode() == ISD::SRL && LHS.hasOneUse() &&
```
- **EN**: The range implements or declares functions including `performBITREV_WCombine`, `combine_CC`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `performBITREV_WCombine`, `combine_CC` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 6826-6900 / 第 6826-6900 行
```cpp
6826:       LHS.getOperand(1).getOpcode() == ISD::Constant) {
6827:     SDValue LHS0 = LHS.getOperand(0);
6828:     if (LHS0.getOpcode() == ISD::AND &&
6829:         LHS0.getOperand(1).getOpcode() == ISD::Constant) {
6830:       uint64_t Mask = LHS0.getConstantOperandVal(1);
6831:       uint64_t ShAmt = LHS.getConstantOperandVal(1);
6832:       if (isPowerOf2_64(Mask) && Log2_64(Mask) == ShAmt) {
6833:         CCVal = CCVal == ISD::SETEQ ? ISD::SETGE : ISD::SETLT;
6834:         CC = DAG.getCondCode(CCVal);
6835: 
6836:         ShAmt = LHS.getValueSizeInBits() - 1 - ShAmt;
6837:         LHS = LHS0.getOperand(0);
6838:         if (ShAmt != 0)
6839:           LHS =
6840:               DAG.getNode(ISD::SHL, DL, LHS.getValueType(), LHS0.getOperand(0),
6841:                           DAG.getConstant(ShAmt, DL, LHS.getValueType()));
6842:         return true;
6843:       }
6844:     }
6845:   }
6846: 
6847:   // (X, 1, setne) -> (X, 0, seteq) if we can prove X is 0/1.
6848:   // This can occur when legalizing some floating point comparisons.
6849:   APInt Mask = APInt::getBitsSetFrom(LHS.getValueSizeInBits(), 1);
6850:   if (isOneConstant(RHS) && DAG.MaskedValueIsZero(LHS, Mask)) {
6851:     CCVal = ISD::getSetCCInverse(CCVal, LHS.getValueType());
6852:     CC = DAG.getCondCode(CCVal);
6853:     RHS = DAG.getConstant(0, DL, LHS.getValueType());
6854:     return true;
6855:   }
6856: 
6857:   return false;
6858: }
6859: 
6860: static SDValue performBR_CCCombine(SDNode *N, SelectionDAG &DAG,
6861:                                    TargetLowering::DAGCombinerInfo &DCI,
6862:                                    const LoongArchSubtarget &Subtarget) {
6863:   SDValue LHS = N->getOperand(1);
6864:   SDValue RHS = N->getOperand(2);
6865:   SDValue CC = N->getOperand(3);
6866:   SDLoc DL(N);
6867: 
6868:   if (combine_CC(LHS, RHS, CC, DL, DAG, Subtarget))
6869:     return DAG.getNode(LoongArchISD::BR_CC, DL, N->getValueType(0),
6870:                        N->getOperand(0), LHS, RHS, CC, N->getOperand(4));
6871: 
6872:   return SDValue();
6873: }
6874: 
6875: static SDValue performSELECT_CCCombine(SDNode *N, SelectionDAG &DAG,
6876:                                        TargetLowering::DAGCombinerInfo &DCI,
6877:                                        const LoongArchSubtarget &Subtarget) {
6878:   // Transform
6879:   SDValue LHS = N->getOperand(0);
6880:   SDValue RHS = N->getOperand(1);
6881:   SDValue CC = N->getOperand(2);
6882:   ISD::CondCode CCVal = cast<CondCodeSDNode>(CC)->get();
6883:   SDValue TrueV = N->getOperand(3);
6884:   SDValue FalseV = N->getOperand(4);
6885:   SDLoc DL(N);
6886:   EVT VT = N->getValueType(0);
6887: 
6888:   // If the True and False values are the same, we don't need a select_cc.
6889:   if (TrueV == FalseV)
6890:     return TrueV;
6891: 
6892:   // (select (x < 0), y, z)  -> x >> (GRLEN - 1) & (y - z) + z
6893:   // (select (x >= 0), y, z) -> x >> (GRLEN - 1) & (z - y) + y
6894:   if (isa<ConstantSDNode>(TrueV) && isa<ConstantSDNode>(FalseV) &&
6895:       isNullConstant(RHS) &&
6896:       (CCVal == ISD::CondCode::SETLT || CCVal == ISD::CondCode::SETGE)) {
6897:     if (CCVal == ISD::CondCode::SETGE)
6898:       std::swap(TrueV, FalseV);
6899: 
6900:     int64_t TrueSImm = cast<ConstantSDNode>(TrueV)->getSExtValue();
```
- **EN**: The range implements or declares functions including `performBR_CCCombine`, `performSELECT_CCCombine`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `performBR_CCCombine`, `performSELECT_CCCombine` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 6901-6975 / 第 6901-6975 行
```cpp
6901:     int64_t FalseSImm = cast<ConstantSDNode>(FalseV)->getSExtValue();
6902:     // Only handle simm12, if it is not in this range, it can be considered as
6903:     // register.
6904:     if (isInt<12>(TrueSImm) && isInt<12>(FalseSImm) &&
6905:         isInt<12>(TrueSImm - FalseSImm)) {
6906:       SDValue SRA =
6907:           DAG.getNode(ISD::SRA, DL, VT, LHS,
6908:                       DAG.getConstant(Subtarget.getGRLen() - 1, DL, VT));
6909:       SDValue AND =
6910:           DAG.getNode(ISD::AND, DL, VT, SRA,
6911:                       DAG.getSignedConstant(TrueSImm - FalseSImm, DL, VT));
6912:       return DAG.getNode(ISD::ADD, DL, VT, AND, FalseV);
6913:     }
6914: 
6915:     if (CCVal == ISD::CondCode::SETGE)
6916:       std::swap(TrueV, FalseV);
6917:   }
6918: 
6919:   if (combine_CC(LHS, RHS, CC, DL, DAG, Subtarget))
6920:     return DAG.getNode(LoongArchISD::SELECT_CC, DL, N->getValueType(0),
6921:                        {LHS, RHS, CC, TrueV, FalseV});
6922: 
6923:   return SDValue();
6924: }
6925: 
6926: template <unsigned N>
6927: static SDValue legalizeIntrinsicImmArg(SDNode *Node, unsigned ImmOp,
6928:                                        SelectionDAG &DAG,
6929:                                        const LoongArchSubtarget &Subtarget,
6930:                                        bool IsSigned = false) {
6931:   SDLoc DL(Node);
6932:   auto *CImm = cast<ConstantSDNode>(Node->getOperand(ImmOp));
6933:   // Check the ImmArg.
6934:   if ((IsSigned && !isInt<N>(CImm->getSExtValue())) ||
6935:       (!IsSigned && !isUInt<N>(CImm->getZExtValue()))) {
6936:     DAG.getContext()->emitError(Node->getOperationName(0) +
6937:                                 ": argument out of range.");
6938:     return DAG.getNode(ISD::UNDEF, DL, Subtarget.getGRLenVT());
6939:   }
6940:   return DAG.getConstant(CImm->getZExtValue(), DL, Subtarget.getGRLenVT());
6941: }
6942: 
6943: template <unsigned N>
6944: static SDValue lowerVectorSplatImm(SDNode *Node, unsigned ImmOp,
6945:                                    SelectionDAG &DAG, bool IsSigned = false) {
6946:   SDLoc DL(Node);
6947:   EVT ResTy = Node->getValueType(0);
6948:   auto *CImm = cast<ConstantSDNode>(Node->getOperand(ImmOp));
6949: 
6950:   // Check the ImmArg.
6951:   if ((IsSigned && !isInt<N>(CImm->getSExtValue())) ||
6952:       (!IsSigned && !isUInt<N>(CImm->getZExtValue()))) {
6953:     DAG.getContext()->emitError(Node->getOperationName(0) +
6954:                                 ": argument out of range.");
6955:     return DAG.getNode(ISD::UNDEF, DL, ResTy);
6956:   }
6957:   return DAG.getConstant(
6958:       APInt(ResTy.getScalarType().getSizeInBits(),
6959:             IsSigned ? CImm->getSExtValue() : CImm->getZExtValue(), IsSigned),
6960:       DL, ResTy);
6961: }
6962: 
6963: static SDValue truncateVecElts(SDNode *Node, SelectionDAG &DAG) {
6964:   SDLoc DL(Node);
6965:   EVT ResTy = Node->getValueType(0);
6966:   SDValue Vec = Node->getOperand(2);
6967:   SDValue Mask = DAG.getConstant(Vec.getScalarValueSizeInBits() - 1, DL, ResTy);
6968:   return DAG.getNode(ISD::AND, DL, ResTy, Vec, Mask);
6969: }
6970: 
6971: static SDValue lowerVectorBitClear(SDNode *Node, SelectionDAG &DAG) {
6972:   SDLoc DL(Node);
6973:   EVT ResTy = Node->getValueType(0);
6974:   SDValue One = DAG.getConstant(1, DL, ResTy);
6975:   SDValue Bit =
```
- **EN**: The range implements or declares functions including `legalizeIntrinsicImmArg`, `lowerVectorSplatImm`, `truncateVecElts`, `lowerVectorBitClear`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `legalizeIntrinsicImmArg`, `lowerVectorSplatImm`, `truncateVecElts`, `lowerVectorBitClear` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 6976-7050 / 第 6976-7050 行
```cpp
6976:       DAG.getNode(ISD::SHL, DL, ResTy, One, truncateVecElts(Node, DAG));
6977: 
6978:   return DAG.getNode(ISD::AND, DL, ResTy, Node->getOperand(1),
6979:                      DAG.getNOT(DL, Bit, ResTy));
6980: }
6981: 
6982: template <unsigned N>
6983: static SDValue lowerVectorBitClearImm(SDNode *Node, SelectionDAG &DAG) {
6984:   SDLoc DL(Node);
6985:   EVT ResTy = Node->getValueType(0);
6986:   auto *CImm = cast<ConstantSDNode>(Node->getOperand(2));
6987:   // Check the unsigned ImmArg.
6988:   if (!isUInt<N>(CImm->getZExtValue())) {
6989:     DAG.getContext()->emitError(Node->getOperationName(0) +
6990:                                 ": argument out of range.");
6991:     return DAG.getNode(ISD::UNDEF, DL, ResTy);
6992:   }
6993: 
6994:   APInt BitImm = APInt(ResTy.getScalarSizeInBits(), 1) << CImm->getAPIntValue();
6995:   SDValue Mask = DAG.getConstant(~BitImm, DL, ResTy);
6996: 
6997:   return DAG.getNode(ISD::AND, DL, ResTy, Node->getOperand(1), Mask);
6998: }
6999: 
7000: template <unsigned N>
7001: static SDValue lowerVectorBitSetImm(SDNode *Node, SelectionDAG &DAG) {
7002:   SDLoc DL(Node);
7003:   EVT ResTy = Node->getValueType(0);
7004:   auto *CImm = cast<ConstantSDNode>(Node->getOperand(2));
7005:   // Check the unsigned ImmArg.
7006:   if (!isUInt<N>(CImm->getZExtValue())) {
7007:     DAG.getContext()->emitError(Node->getOperationName(0) +
7008:                                 ": argument out of range.");
7009:     return DAG.getNode(ISD::UNDEF, DL, ResTy);
7010:   }
7011: 
7012:   APInt Imm = APInt(ResTy.getScalarSizeInBits(), 1) << CImm->getAPIntValue();
7013:   SDValue BitImm = DAG.getConstant(Imm, DL, ResTy);
7014:   return DAG.getNode(ISD::OR, DL, ResTy, Node->getOperand(1), BitImm);
7015: }
7016: 
7017: template <unsigned N>
7018: static SDValue lowerVectorBitRevImm(SDNode *Node, SelectionDAG &DAG) {
7019:   SDLoc DL(Node);
7020:   EVT ResTy = Node->getValueType(0);
7021:   auto *CImm = cast<ConstantSDNode>(Node->getOperand(2));
7022:   // Check the unsigned ImmArg.
7023:   if (!isUInt<N>(CImm->getZExtValue())) {
7024:     DAG.getContext()->emitError(Node->getOperationName(0) +
7025:                                 ": argument out of range.");
7026:     return DAG.getNode(ISD::UNDEF, DL, ResTy);
7027:   }
7028: 
7029:   APInt Imm = APInt(ResTy.getScalarSizeInBits(), 1) << CImm->getAPIntValue();
7030:   SDValue BitImm = DAG.getConstant(Imm, DL, ResTy);
7031:   return DAG.getNode(ISD::XOR, DL, ResTy, Node->getOperand(1), BitImm);
7032: }
7033: 
7034: template <unsigned W>
7035: static SDValue lowerVectorPickVE2GR(SDNode *N, SelectionDAG &DAG,
7036:                                     unsigned ResOp) {
7037:   unsigned Imm = N->getConstantOperandVal(2);
7038:   if (!isUInt<W>(Imm)) {
7039:     const StringRef ErrorMsg = "argument out of range";
7040:     DAG.getContext()->emitError(N->getOperationName(0) + ": " + ErrorMsg + ".");
7041:     return DAG.getUNDEF(N->getValueType(0));
7042:   }
7043:   SDLoc DL(N);
7044:   SDValue Vec = N->getOperand(1);
7045:   SDValue Idx = DAG.getConstant(Imm, DL, MVT::i32);
7046:   SDValue EltVT = DAG.getValueType(Vec.getValueType().getVectorElementType());
7047:   return DAG.getNode(ResOp, DL, N->getValueType(0), Vec, Idx, EltVT);
7048: }
7049: 
7050: static SDValue
```
- **EN**: The range implements or declares functions including `lowerVectorBitClearImm`, `lowerVectorBitSetImm`, `lowerVectorBitRevImm`, `lowerVectorPickVE2GR`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `lowerVectorBitClearImm`, `lowerVectorBitSetImm`, `lowerVectorBitRevImm`, `lowerVectorPickVE2GR` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 7051-7125 / 第 7051-7125 行
```cpp
7051: performINTRINSIC_WO_CHAINCombine(SDNode *N, SelectionDAG &DAG,
7052:                                  TargetLowering::DAGCombinerInfo &DCI,
7053:                                  const LoongArchSubtarget &Subtarget) {
7054:   SDLoc DL(N);
7055:   switch (N->getConstantOperandVal(0)) {
7056:   default:
7057:     break;
7058:   case Intrinsic::loongarch_lsx_vadd_b:
7059:   case Intrinsic::loongarch_lsx_vadd_h:
7060:   case Intrinsic::loongarch_lsx_vadd_w:
7061:   case Intrinsic::loongarch_lsx_vadd_d:
7062:   case Intrinsic::loongarch_lasx_xvadd_b:
7063:   case Intrinsic::loongarch_lasx_xvadd_h:
7064:   case Intrinsic::loongarch_lasx_xvadd_w:
7065:   case Intrinsic::loongarch_lasx_xvadd_d:
7066:     return DAG.getNode(ISD::ADD, DL, N->getValueType(0), N->getOperand(1),
7067:                        N->getOperand(2));
7068:   case Intrinsic::loongarch_lsx_vaddi_bu:
7069:   case Intrinsic::loongarch_lsx_vaddi_hu:
7070:   case Intrinsic::loongarch_lsx_vaddi_wu:
7071:   case Intrinsic::loongarch_lsx_vaddi_du:
7072:   case Intrinsic::loongarch_lasx_xvaddi_bu:
7073:   case Intrinsic::loongarch_lasx_xvaddi_hu:
7074:   case Intrinsic::loongarch_lasx_xvaddi_wu:
7075:   case Intrinsic::loongarch_lasx_xvaddi_du:
7076:     return DAG.getNode(ISD::ADD, DL, N->getValueType(0), N->getOperand(1),
7077:                        lowerVectorSplatImm<5>(N, 2, DAG));
7078:   case Intrinsic::loongarch_lsx_vsub_b:
7079:   case Intrinsic::loongarch_lsx_vsub_h:
7080:   case Intrinsic::loongarch_lsx_vsub_w:
7081:   case Intrinsic::loongarch_lsx_vsub_d:
7082:   case Intrinsic::loongarch_lasx_xvsub_b:
7083:   case Intrinsic::loongarch_lasx_xvsub_h:
7084:   case Intrinsic::loongarch_lasx_xvsub_w:
7085:   case Intrinsic::loongarch_lasx_xvsub_d:
7086:     return DAG.getNode(ISD::SUB, DL, N->getValueType(0), N->getOperand(1),
7087:                        N->getOperand(2));
7088:   case Intrinsic::loongarch_lsx_vsubi_bu:
7089:   case Intrinsic::loongarch_lsx_vsubi_hu:
7090:   case Intrinsic::loongarch_lsx_vsubi_wu:
7091:   case Intrinsic::loongarch_lsx_vsubi_du:
7092:   case Intrinsic::loongarch_lasx_xvsubi_bu:
7093:   case Intrinsic::loongarch_lasx_xvsubi_hu:
7094:   case Intrinsic::loongarch_lasx_xvsubi_wu:
7095:   case Intrinsic::loongarch_lasx_xvsubi_du:
7096:     return DAG.getNode(ISD::SUB, DL, N->getValueType(0), N->getOperand(1),
7097:                        lowerVectorSplatImm<5>(N, 2, DAG));
7098:   case Intrinsic::loongarch_lsx_vneg_b:
7099:   case Intrinsic::loongarch_lsx_vneg_h:
7100:   case Intrinsic::loongarch_lsx_vneg_w:
7101:   case Intrinsic::loongarch_lsx_vneg_d:
7102:   case Intrinsic::loongarch_lasx_xvneg_b:
7103:   case Intrinsic::loongarch_lasx_xvneg_h:
7104:   case Intrinsic::loongarch_lasx_xvneg_w:
7105:   case Intrinsic::loongarch_lasx_xvneg_d:
7106:     return DAG.getNode(
7107:         ISD::SUB, DL, N->getValueType(0),
7108:         DAG.getConstant(
7109:             APInt(N->getValueType(0).getScalarType().getSizeInBits(), 0,
7110:                   /*isSigned=*/true),
7111:             SDLoc(N), N->getValueType(0)),
7112:         N->getOperand(1));
7113:   case Intrinsic::loongarch_lsx_vmax_b:
7114:   case Intrinsic::loongarch_lsx_vmax_h:
7115:   case Intrinsic::loongarch_lsx_vmax_w:
7116:   case Intrinsic::loongarch_lsx_vmax_d:
7117:   case Intrinsic::loongarch_lasx_xvmax_b:
7118:   case Intrinsic::loongarch_lasx_xvmax_h:
7119:   case Intrinsic::loongarch_lasx_xvmax_w:
7120:   case Intrinsic::loongarch_lasx_xvmax_d:
7121:     return DAG.getNode(ISD::SMAX, DL, N->getValueType(0), N->getOperand(1),
7122:                        N->getOperand(2));
7123:   case Intrinsic::loongarch_lsx_vmax_bu:
7124:   case Intrinsic::loongarch_lsx_vmax_hu:
7125:   case Intrinsic::loongarch_lsx_vmax_wu:
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 7126-7200 / 第 7126-7200 行
```cpp
7126:   case Intrinsic::loongarch_lsx_vmax_du:
7127:   case Intrinsic::loongarch_lasx_xvmax_bu:
7128:   case Intrinsic::loongarch_lasx_xvmax_hu:
7129:   case Intrinsic::loongarch_lasx_xvmax_wu:
7130:   case Intrinsic::loongarch_lasx_xvmax_du:
7131:     return DAG.getNode(ISD::UMAX, DL, N->getValueType(0), N->getOperand(1),
7132:                        N->getOperand(2));
7133:   case Intrinsic::loongarch_lsx_vmaxi_b:
7134:   case Intrinsic::loongarch_lsx_vmaxi_h:
7135:   case Intrinsic::loongarch_lsx_vmaxi_w:
7136:   case Intrinsic::loongarch_lsx_vmaxi_d:
7137:   case Intrinsic::loongarch_lasx_xvmaxi_b:
7138:   case Intrinsic::loongarch_lasx_xvmaxi_h:
7139:   case Intrinsic::loongarch_lasx_xvmaxi_w:
7140:   case Intrinsic::loongarch_lasx_xvmaxi_d:
7141:     return DAG.getNode(ISD::SMAX, DL, N->getValueType(0), N->getOperand(1),
7142:                        lowerVectorSplatImm<5>(N, 2, DAG, /*IsSigned=*/true));
7143:   case Intrinsic::loongarch_lsx_vmaxi_bu:
7144:   case Intrinsic::loongarch_lsx_vmaxi_hu:
7145:   case Intrinsic::loongarch_lsx_vmaxi_wu:
7146:   case Intrinsic::loongarch_lsx_vmaxi_du:
7147:   case Intrinsic::loongarch_lasx_xvmaxi_bu:
7148:   case Intrinsic::loongarch_lasx_xvmaxi_hu:
7149:   case Intrinsic::loongarch_lasx_xvmaxi_wu:
7150:   case Intrinsic::loongarch_lasx_xvmaxi_du:
7151:     return DAG.getNode(ISD::UMAX, DL, N->getValueType(0), N->getOperand(1),
7152:                        lowerVectorSplatImm<5>(N, 2, DAG));
7153:   case Intrinsic::loongarch_lsx_vmin_b:
7154:   case Intrinsic::loongarch_lsx_vmin_h:
7155:   case Intrinsic::loongarch_lsx_vmin_w:
7156:   case Intrinsic::loongarch_lsx_vmin_d:
7157:   case Intrinsic::loongarch_lasx_xvmin_b:
7158:   case Intrinsic::loongarch_lasx_xvmin_h:
7159:   case Intrinsic::loongarch_lasx_xvmin_w:
7160:   case Intrinsic::loongarch_lasx_xvmin_d:
7161:     return DAG.getNode(ISD::SMIN, DL, N->getValueType(0), N->getOperand(1),
7162:                        N->getOperand(2));
7163:   case Intrinsic::loongarch_lsx_vmin_bu:
7164:   case Intrinsic::loongarch_lsx_vmin_hu:
7165:   case Intrinsic::loongarch_lsx_vmin_wu:
7166:   case Intrinsic::loongarch_lsx_vmin_du:
7167:   case Intrinsic::loongarch_lasx_xvmin_bu:
7168:   case Intrinsic::loongarch_lasx_xvmin_hu:
7169:   case Intrinsic::loongarch_lasx_xvmin_wu:
7170:   case Intrinsic::loongarch_lasx_xvmin_du:
7171:     return DAG.getNode(ISD::UMIN, DL, N->getValueType(0), N->getOperand(1),
7172:                        N->getOperand(2));
7173:   case Intrinsic::loongarch_lsx_vmini_b:
7174:   case Intrinsic::loongarch_lsx_vmini_h:
7175:   case Intrinsic::loongarch_lsx_vmini_w:
7176:   case Intrinsic::loongarch_lsx_vmini_d:
7177:   case Intrinsic::loongarch_lasx_xvmini_b:
7178:   case Intrinsic::loongarch_lasx_xvmini_h:
7179:   case Intrinsic::loongarch_lasx_xvmini_w:
7180:   case Intrinsic::loongarch_lasx_xvmini_d:
7181:     return DAG.getNode(ISD::SMIN, DL, N->getValueType(0), N->getOperand(1),
7182:                        lowerVectorSplatImm<5>(N, 2, DAG, /*IsSigned=*/true));
7183:   case Intrinsic::loongarch_lsx_vmini_bu:
7184:   case Intrinsic::loongarch_lsx_vmini_hu:
7185:   case Intrinsic::loongarch_lsx_vmini_wu:
7186:   case Intrinsic::loongarch_lsx_vmini_du:
7187:   case Intrinsic::loongarch_lasx_xvmini_bu:
7188:   case Intrinsic::loongarch_lasx_xvmini_hu:
7189:   case Intrinsic::loongarch_lasx_xvmini_wu:
7190:   case Intrinsic::loongarch_lasx_xvmini_du:
7191:     return DAG.getNode(ISD::UMIN, DL, N->getValueType(0), N->getOperand(1),
7192:                        lowerVectorSplatImm<5>(N, 2, DAG));
7193:   case Intrinsic::loongarch_lsx_vmul_b:
7194:   case Intrinsic::loongarch_lsx_vmul_h:
7195:   case Intrinsic::loongarch_lsx_vmul_w:
7196:   case Intrinsic::loongarch_lsx_vmul_d:
7197:   case Intrinsic::loongarch_lasx_xvmul_b:
7198:   case Intrinsic::loongarch_lasx_xvmul_h:
7199:   case Intrinsic::loongarch_lasx_xvmul_w:
7200:   case Intrinsic::loongarch_lasx_xvmul_d:
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 7201-7275 / 第 7201-7275 行
```cpp
7201:     return DAG.getNode(ISD::MUL, DL, N->getValueType(0), N->getOperand(1),
7202:                        N->getOperand(2));
7203:   case Intrinsic::loongarch_lsx_vmadd_b:
7204:   case Intrinsic::loongarch_lsx_vmadd_h:
7205:   case Intrinsic::loongarch_lsx_vmadd_w:
7206:   case Intrinsic::loongarch_lsx_vmadd_d:
7207:   case Intrinsic::loongarch_lasx_xvmadd_b:
7208:   case Intrinsic::loongarch_lasx_xvmadd_h:
7209:   case Intrinsic::loongarch_lasx_xvmadd_w:
7210:   case Intrinsic::loongarch_lasx_xvmadd_d: {
7211:     EVT ResTy = N->getValueType(0);
7212:     return DAG.getNode(ISD::ADD, SDLoc(N), ResTy, N->getOperand(1),
7213:                        DAG.getNode(ISD::MUL, SDLoc(N), ResTy, N->getOperand(2),
7214:                                    N->getOperand(3)));
7215:   }
7216:   case Intrinsic::loongarch_lsx_vmsub_b:
7217:   case Intrinsic::loongarch_lsx_vmsub_h:
7218:   case Intrinsic::loongarch_lsx_vmsub_w:
7219:   case Intrinsic::loongarch_lsx_vmsub_d:
7220:   case Intrinsic::loongarch_lasx_xvmsub_b:
7221:   case Intrinsic::loongarch_lasx_xvmsub_h:
7222:   case Intrinsic::loongarch_lasx_xvmsub_w:
7223:   case Intrinsic::loongarch_lasx_xvmsub_d: {
7224:     EVT ResTy = N->getValueType(0);
7225:     return DAG.getNode(ISD::SUB, SDLoc(N), ResTy, N->getOperand(1),
7226:                        DAG.getNode(ISD::MUL, SDLoc(N), ResTy, N->getOperand(2),
7227:                                    N->getOperand(3)));
7228:   }
7229:   case Intrinsic::loongarch_lsx_vdiv_b:
7230:   case Intrinsic::loongarch_lsx_vdiv_h:
7231:   case Intrinsic::loongarch_lsx_vdiv_w:
7232:   case Intrinsic::loongarch_lsx_vdiv_d:
7233:   case Intrinsic::loongarch_lasx_xvdiv_b:
7234:   case Intrinsic::loongarch_lasx_xvdiv_h:
7235:   case Intrinsic::loongarch_lasx_xvdiv_w:
7236:   case Intrinsic::loongarch_lasx_xvdiv_d:
7237:     return DAG.getNode(ISD::SDIV, DL, N->getValueType(0), N->getOperand(1),
7238:                        N->getOperand(2));
7239:   case Intrinsic::loongarch_lsx_vdiv_bu:
7240:   case Intrinsic::loongarch_lsx_vdiv_hu:
7241:   case Intrinsic::loongarch_lsx_vdiv_wu:
7242:   case Intrinsic::loongarch_lsx_vdiv_du:
7243:   case Intrinsic::loongarch_lasx_xvdiv_bu:
7244:   case Intrinsic::loongarch_lasx_xvdiv_hu:
7245:   case Intrinsic::loongarch_lasx_xvdiv_wu:
7246:   case Intrinsic::loongarch_lasx_xvdiv_du:
7247:     return DAG.getNode(ISD::UDIV, DL, N->getValueType(0), N->getOperand(1),
7248:                        N->getOperand(2));
7249:   case Intrinsic::loongarch_lsx_vmod_b:
7250:   case Intrinsic::loongarch_lsx_vmod_h:
7251:   case Intrinsic::loongarch_lsx_vmod_w:
7252:   case Intrinsic::loongarch_lsx_vmod_d:
7253:   case Intrinsic::loongarch_lasx_xvmod_b:
7254:   case Intrinsic::loongarch_lasx_xvmod_h:
7255:   case Intrinsic::loongarch_lasx_xvmod_w:
7256:   case Intrinsic::loongarch_lasx_xvmod_d:
7257:     return DAG.getNode(ISD::SREM, DL, N->getValueType(0), N->getOperand(1),
7258:                        N->getOperand(2));
7259:   case Intrinsic::loongarch_lsx_vmod_bu:
7260:   case Intrinsic::loongarch_lsx_vmod_hu:
7261:   case Intrinsic::loongarch_lsx_vmod_wu:
7262:   case Intrinsic::loongarch_lsx_vmod_du:
7263:   case Intrinsic::loongarch_lasx_xvmod_bu:
7264:   case Intrinsic::loongarch_lasx_xvmod_hu:
7265:   case Intrinsic::loongarch_lasx_xvmod_wu:
7266:   case Intrinsic::loongarch_lasx_xvmod_du:
7267:     return DAG.getNode(ISD::UREM, DL, N->getValueType(0), N->getOperand(1),
7268:                        N->getOperand(2));
7269:   case Intrinsic::loongarch_lsx_vand_v:
7270:   case Intrinsic::loongarch_lasx_xvand_v:
7271:     return DAG.getNode(ISD::AND, DL, N->getValueType(0), N->getOperand(1),
7272:                        N->getOperand(2));
7273:   case Intrinsic::loongarch_lsx_vor_v:
7274:   case Intrinsic::loongarch_lasx_xvor_v:
7275:     return DAG.getNode(ISD::OR, DL, N->getValueType(0), N->getOperand(1),
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 7276-7350 / 第 7276-7350 行
```cpp
7276:                        N->getOperand(2));
7277:   case Intrinsic::loongarch_lsx_vxor_v:
7278:   case Intrinsic::loongarch_lasx_xvxor_v:
7279:     return DAG.getNode(ISD::XOR, DL, N->getValueType(0), N->getOperand(1),
7280:                        N->getOperand(2));
7281:   case Intrinsic::loongarch_lsx_vnor_v:
7282:   case Intrinsic::loongarch_lasx_xvnor_v: {
7283:     SDValue Res = DAG.getNode(ISD::OR, DL, N->getValueType(0), N->getOperand(1),
7284:                               N->getOperand(2));
7285:     return DAG.getNOT(DL, Res, Res->getValueType(0));
7286:   }
7287:   case Intrinsic::loongarch_lsx_vandi_b:
7288:   case Intrinsic::loongarch_lasx_xvandi_b:
7289:     return DAG.getNode(ISD::AND, DL, N->getValueType(0), N->getOperand(1),
7290:                        lowerVectorSplatImm<8>(N, 2, DAG));
7291:   case Intrinsic::loongarch_lsx_vori_b:
7292:   case Intrinsic::loongarch_lasx_xvori_b:
7293:     return DAG.getNode(ISD::OR, DL, N->getValueType(0), N->getOperand(1),
7294:                        lowerVectorSplatImm<8>(N, 2, DAG));
7295:   case Intrinsic::loongarch_lsx_vxori_b:
7296:   case Intrinsic::loongarch_lasx_xvxori_b:
7297:     return DAG.getNode(ISD::XOR, DL, N->getValueType(0), N->getOperand(1),
7298:                        lowerVectorSplatImm<8>(N, 2, DAG));
7299:   case Intrinsic::loongarch_lsx_vsll_b:
7300:   case Intrinsic::loongarch_lsx_vsll_h:
7301:   case Intrinsic::loongarch_lsx_vsll_w:
7302:   case Intrinsic::loongarch_lsx_vsll_d:
7303:   case Intrinsic::loongarch_lasx_xvsll_b:
7304:   case Intrinsic::loongarch_lasx_xvsll_h:
7305:   case Intrinsic::loongarch_lasx_xvsll_w:
7306:   case Intrinsic::loongarch_lasx_xvsll_d:
7307:     return DAG.getNode(ISD::SHL, DL, N->getValueType(0), N->getOperand(1),
7308:                        truncateVecElts(N, DAG));
7309:   case Intrinsic::loongarch_lsx_vslli_b:
7310:   case Intrinsic::loongarch_lasx_xvslli_b:
7311:     return DAG.getNode(ISD::SHL, DL, N->getValueType(0), N->getOperand(1),
7312:                        lowerVectorSplatImm<3>(N, 2, DAG));
7313:   case Intrinsic::loongarch_lsx_vslli_h:
7314:   case Intrinsic::loongarch_lasx_xvslli_h:
7315:     return DAG.getNode(ISD::SHL, DL, N->getValueType(0), N->getOperand(1),
7316:                        lowerVectorSplatImm<4>(N, 2, DAG));
7317:   case Intrinsic::loongarch_lsx_vslli_w:
7318:   case Intrinsic::loongarch_lasx_xvslli_w:
7319:     return DAG.getNode(ISD::SHL, DL, N->getValueType(0), N->getOperand(1),
7320:                        lowerVectorSplatImm<5>(N, 2, DAG));
7321:   case Intrinsic::loongarch_lsx_vslli_d:
7322:   case Intrinsic::loongarch_lasx_xvslli_d:
7323:     return DAG.getNode(ISD::SHL, DL, N->getValueType(0), N->getOperand(1),
7324:                        lowerVectorSplatImm<6>(N, 2, DAG));
7325:   case Intrinsic::loongarch_lsx_vsrl_b:
7326:   case Intrinsic::loongarch_lsx_vsrl_h:
7327:   case Intrinsic::loongarch_lsx_vsrl_w:
7328:   case Intrinsic::loongarch_lsx_vsrl_d:
7329:   case Intrinsic::loongarch_lasx_xvsrl_b:
7330:   case Intrinsic::loongarch_lasx_xvsrl_h:
7331:   case Intrinsic::loongarch_lasx_xvsrl_w:
7332:   case Intrinsic::loongarch_lasx_xvsrl_d:
7333:     return DAG.getNode(ISD::SRL, DL, N->getValueType(0), N->getOperand(1),
7334:                        truncateVecElts(N, DAG));
7335:   case Intrinsic::loongarch_lsx_vsrli_b:
7336:   case Intrinsic::loongarch_lasx_xvsrli_b:
7337:     return DAG.getNode(ISD::SRL, DL, N->getValueType(0), N->getOperand(1),
7338:                        lowerVectorSplatImm<3>(N, 2, DAG));
7339:   case Intrinsic::loongarch_lsx_vsrli_h:
7340:   case Intrinsic::loongarch_lasx_xvsrli_h:
7341:     return DAG.getNode(ISD::SRL, DL, N->getValueType(0), N->getOperand(1),
7342:                        lowerVectorSplatImm<4>(N, 2, DAG));
7343:   case Intrinsic::loongarch_lsx_vsrli_w:
7344:   case Intrinsic::loongarch_lasx_xvsrli_w:
7345:     return DAG.getNode(ISD::SRL, DL, N->getValueType(0), N->getOperand(1),
7346:                        lowerVectorSplatImm<5>(N, 2, DAG));
7347:   case Intrinsic::loongarch_lsx_vsrli_d:
7348:   case Intrinsic::loongarch_lasx_xvsrli_d:
7349:     return DAG.getNode(ISD::SRL, DL, N->getValueType(0), N->getOperand(1),
7350:                        lowerVectorSplatImm<6>(N, 2, DAG));
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 7351-7425 / 第 7351-7425 行
```cpp
7351:   case Intrinsic::loongarch_lsx_vsra_b:
7352:   case Intrinsic::loongarch_lsx_vsra_h:
7353:   case Intrinsic::loongarch_lsx_vsra_w:
7354:   case Intrinsic::loongarch_lsx_vsra_d:
7355:   case Intrinsic::loongarch_lasx_xvsra_b:
7356:   case Intrinsic::loongarch_lasx_xvsra_h:
7357:   case Intrinsic::loongarch_lasx_xvsra_w:
7358:   case Intrinsic::loongarch_lasx_xvsra_d:
7359:     return DAG.getNode(ISD::SRA, DL, N->getValueType(0), N->getOperand(1),
7360:                        truncateVecElts(N, DAG));
7361:   case Intrinsic::loongarch_lsx_vsrai_b:
7362:   case Intrinsic::loongarch_lasx_xvsrai_b:
7363:     return DAG.getNode(ISD::SRA, DL, N->getValueType(0), N->getOperand(1),
7364:                        lowerVectorSplatImm<3>(N, 2, DAG));
7365:   case Intrinsic::loongarch_lsx_vsrai_h:
7366:   case Intrinsic::loongarch_lasx_xvsrai_h:
7367:     return DAG.getNode(ISD::SRA, DL, N->getValueType(0), N->getOperand(1),
7368:                        lowerVectorSplatImm<4>(N, 2, DAG));
7369:   case Intrinsic::loongarch_lsx_vsrai_w:
7370:   case Intrinsic::loongarch_lasx_xvsrai_w:
7371:     return DAG.getNode(ISD::SRA, DL, N->getValueType(0), N->getOperand(1),
7372:                        lowerVectorSplatImm<5>(N, 2, DAG));
7373:   case Intrinsic::loongarch_lsx_vsrai_d:
7374:   case Intrinsic::loongarch_lasx_xvsrai_d:
7375:     return DAG.getNode(ISD::SRA, DL, N->getValueType(0), N->getOperand(1),
7376:                        lowerVectorSplatImm<6>(N, 2, DAG));
7377:   case Intrinsic::loongarch_lsx_vclz_b:
7378:   case Intrinsic::loongarch_lsx_vclz_h:
7379:   case Intrinsic::loongarch_lsx_vclz_w:
7380:   case Intrinsic::loongarch_lsx_vclz_d:
7381:   case Intrinsic::loongarch_lasx_xvclz_b:
7382:   case Intrinsic::loongarch_lasx_xvclz_h:
7383:   case Intrinsic::loongarch_lasx_xvclz_w:
7384:   case Intrinsic::loongarch_lasx_xvclz_d:
7385:     return DAG.getNode(ISD::CTLZ, DL, N->getValueType(0), N->getOperand(1));
7386:   case Intrinsic::loongarch_lsx_vpcnt_b:
7387:   case Intrinsic::loongarch_lsx_vpcnt_h:
7388:   case Intrinsic::loongarch_lsx_vpcnt_w:
7389:   case Intrinsic::loongarch_lsx_vpcnt_d:
7390:   case Intrinsic::loongarch_lasx_xvpcnt_b:
7391:   case Intrinsic::loongarch_lasx_xvpcnt_h:
7392:   case Intrinsic::loongarch_lasx_xvpcnt_w:
7393:   case Intrinsic::loongarch_lasx_xvpcnt_d:
7394:     return DAG.getNode(ISD::CTPOP, DL, N->getValueType(0), N->getOperand(1));
7395:   case Intrinsic::loongarch_lsx_vbitclr_b:
7396:   case Intrinsic::loongarch_lsx_vbitclr_h:
7397:   case Intrinsic::loongarch_lsx_vbitclr_w:
7398:   case Intrinsic::loongarch_lsx_vbitclr_d:
7399:   case Intrinsic::loongarch_lasx_xvbitclr_b:
7400:   case Intrinsic::loongarch_lasx_xvbitclr_h:
7401:   case Intrinsic::loongarch_lasx_xvbitclr_w:
7402:   case Intrinsic::loongarch_lasx_xvbitclr_d:
7403:     return lowerVectorBitClear(N, DAG);
7404:   case Intrinsic::loongarch_lsx_vbitclri_b:
7405:   case Intrinsic::loongarch_lasx_xvbitclri_b:
7406:     return lowerVectorBitClearImm<3>(N, DAG);
7407:   case Intrinsic::loongarch_lsx_vbitclri_h:
7408:   case Intrinsic::loongarch_lasx_xvbitclri_h:
7409:     return lowerVectorBitClearImm<4>(N, DAG);
7410:   case Intrinsic::loongarch_lsx_vbitclri_w:
7411:   case Intrinsic::loongarch_lasx_xvbitclri_w:
7412:     return lowerVectorBitClearImm<5>(N, DAG);
7413:   case Intrinsic::loongarch_lsx_vbitclri_d:
7414:   case Intrinsic::loongarch_lasx_xvbitclri_d:
7415:     return lowerVectorBitClearImm<6>(N, DAG);
7416:   case Intrinsic::loongarch_lsx_vbitset_b:
7417:   case Intrinsic::loongarch_lsx_vbitset_h:
7418:   case Intrinsic::loongarch_lsx_vbitset_w:
7419:   case Intrinsic::loongarch_lsx_vbitset_d:
7420:   case Intrinsic::loongarch_lasx_xvbitset_b:
7421:   case Intrinsic::loongarch_lasx_xvbitset_h:
7422:   case Intrinsic::loongarch_lasx_xvbitset_w:
7423:   case Intrinsic::loongarch_lasx_xvbitset_d: {
7424:     EVT VecTy = N->getValueType(0);
7425:     SDValue One = DAG.getConstant(1, DL, VecTy);
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 7426-7500 / 第 7426-7500 行
```cpp
7426:     return DAG.getNode(
7427:         ISD::OR, DL, VecTy, N->getOperand(1),
7428:         DAG.getNode(ISD::SHL, DL, VecTy, One, truncateVecElts(N, DAG)));
7429:   }
7430:   case Intrinsic::loongarch_lsx_vbitseti_b:
7431:   case Intrinsic::loongarch_lasx_xvbitseti_b:
7432:     return lowerVectorBitSetImm<3>(N, DAG);
7433:   case Intrinsic::loongarch_lsx_vbitseti_h:
7434:   case Intrinsic::loongarch_lasx_xvbitseti_h:
7435:     return lowerVectorBitSetImm<4>(N, DAG);
7436:   case Intrinsic::loongarch_lsx_vbitseti_w:
7437:   case Intrinsic::loongarch_lasx_xvbitseti_w:
7438:     return lowerVectorBitSetImm<5>(N, DAG);
7439:   case Intrinsic::loongarch_lsx_vbitseti_d:
7440:   case Intrinsic::loongarch_lasx_xvbitseti_d:
7441:     return lowerVectorBitSetImm<6>(N, DAG);
7442:   case Intrinsic::loongarch_lsx_vbitrev_b:
7443:   case Intrinsic::loongarch_lsx_vbitrev_h:
7444:   case Intrinsic::loongarch_lsx_vbitrev_w:
7445:   case Intrinsic::loongarch_lsx_vbitrev_d:
7446:   case Intrinsic::loongarch_lasx_xvbitrev_b:
7447:   case Intrinsic::loongarch_lasx_xvbitrev_h:
7448:   case Intrinsic::loongarch_lasx_xvbitrev_w:
7449:   case Intrinsic::loongarch_lasx_xvbitrev_d: {
7450:     EVT VecTy = N->getValueType(0);
7451:     SDValue One = DAG.getConstant(1, DL, VecTy);
7452:     return DAG.getNode(
7453:         ISD::XOR, DL, VecTy, N->getOperand(1),
7454:         DAG.getNode(ISD::SHL, DL, VecTy, One, truncateVecElts(N, DAG)));
7455:   }
7456:   case Intrinsic::loongarch_lsx_vbitrevi_b:
7457:   case Intrinsic::loongarch_lasx_xvbitrevi_b:
7458:     return lowerVectorBitRevImm<3>(N, DAG);
7459:   case Intrinsic::loongarch_lsx_vbitrevi_h:
7460:   case Intrinsic::loongarch_lasx_xvbitrevi_h:
7461:     return lowerVectorBitRevImm<4>(N, DAG);
7462:   case Intrinsic::loongarch_lsx_vbitrevi_w:
7463:   case Intrinsic::loongarch_lasx_xvbitrevi_w:
7464:     return lowerVectorBitRevImm<5>(N, DAG);
7465:   case Intrinsic::loongarch_lsx_vbitrevi_d:
7466:   case Intrinsic::loongarch_lasx_xvbitrevi_d:
7467:     return lowerVectorBitRevImm<6>(N, DAG);
7468:   case Intrinsic::loongarch_lsx_vfadd_s:
7469:   case Intrinsic::loongarch_lsx_vfadd_d:
7470:   case Intrinsic::loongarch_lasx_xvfadd_s:
7471:   case Intrinsic::loongarch_lasx_xvfadd_d:
7472:     return DAG.getNode(ISD::FADD, DL, N->getValueType(0), N->getOperand(1),
7473:                        N->getOperand(2));
7474:   case Intrinsic::loongarch_lsx_vfsub_s:
7475:   case Intrinsic::loongarch_lsx_vfsub_d:
7476:   case Intrinsic::loongarch_lasx_xvfsub_s:
7477:   case Intrinsic::loongarch_lasx_xvfsub_d:
7478:     return DAG.getNode(ISD::FSUB, DL, N->getValueType(0), N->getOperand(1),
7479:                        N->getOperand(2));
7480:   case Intrinsic::loongarch_lsx_vfmul_s:
7481:   case Intrinsic::loongarch_lsx_vfmul_d:
7482:   case Intrinsic::loongarch_lasx_xvfmul_s:
7483:   case Intrinsic::loongarch_lasx_xvfmul_d:
7484:     return DAG.getNode(ISD::FMUL, DL, N->getValueType(0), N->getOperand(1),
7485:                        N->getOperand(2));
7486:   case Intrinsic::loongarch_lsx_vfdiv_s:
7487:   case Intrinsic::loongarch_lsx_vfdiv_d:
7488:   case Intrinsic::loongarch_lasx_xvfdiv_s:
7489:   case Intrinsic::loongarch_lasx_xvfdiv_d:
7490:     return DAG.getNode(ISD::FDIV, DL, N->getValueType(0), N->getOperand(1),
7491:                        N->getOperand(2));
7492:   case Intrinsic::loongarch_lsx_vfmadd_s:
7493:   case Intrinsic::loongarch_lsx_vfmadd_d:
7494:   case Intrinsic::loongarch_lasx_xvfmadd_s:
7495:   case Intrinsic::loongarch_lasx_xvfmadd_d:
7496:     return DAG.getNode(ISD::FMA, DL, N->getValueType(0), N->getOperand(1),
7497:                        N->getOperand(2), N->getOperand(3));
7498:   case Intrinsic::loongarch_lsx_vinsgr2vr_b:
7499:     return DAG.getNode(ISD::INSERT_VECTOR_ELT, SDLoc(N), N->getValueType(0),
7500:                        N->getOperand(1), N->getOperand(2),
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 7501-7575 / 第 7501-7575 行
```cpp
7501:                        legalizeIntrinsicImmArg<4>(N, 3, DAG, Subtarget));
7502:   case Intrinsic::loongarch_lsx_vinsgr2vr_h:
7503:   case Intrinsic::loongarch_lasx_xvinsgr2vr_w:
7504:     return DAG.getNode(ISD::INSERT_VECTOR_ELT, SDLoc(N), N->getValueType(0),
7505:                        N->getOperand(1), N->getOperand(2),
7506:                        legalizeIntrinsicImmArg<3>(N, 3, DAG, Subtarget));
7507:   case Intrinsic::loongarch_lsx_vinsgr2vr_w:
7508:   case Intrinsic::loongarch_lasx_xvinsgr2vr_d:
7509:     return DAG.getNode(ISD::INSERT_VECTOR_ELT, SDLoc(N), N->getValueType(0),
7510:                        N->getOperand(1), N->getOperand(2),
7511:                        legalizeIntrinsicImmArg<2>(N, 3, DAG, Subtarget));
7512:   case Intrinsic::loongarch_lsx_vinsgr2vr_d:
7513:     return DAG.getNode(ISD::INSERT_VECTOR_ELT, SDLoc(N), N->getValueType(0),
7514:                        N->getOperand(1), N->getOperand(2),
7515:                        legalizeIntrinsicImmArg<1>(N, 3, DAG, Subtarget));
7516:   case Intrinsic::loongarch_lsx_vreplgr2vr_b:
7517:   case Intrinsic::loongarch_lsx_vreplgr2vr_h:
7518:   case Intrinsic::loongarch_lsx_vreplgr2vr_w:
7519:   case Intrinsic::loongarch_lsx_vreplgr2vr_d:
7520:   case Intrinsic::loongarch_lasx_xvreplgr2vr_b:
7521:   case Intrinsic::loongarch_lasx_xvreplgr2vr_h:
7522:   case Intrinsic::loongarch_lasx_xvreplgr2vr_w:
7523:   case Intrinsic::loongarch_lasx_xvreplgr2vr_d:
7524:     return DAG.getNode(LoongArchISD::VREPLGR2VR, DL, N->getValueType(0),
7525:                        DAG.getNode(ISD::ANY_EXTEND, DL, Subtarget.getGRLenVT(),
7526:                                    N->getOperand(1)));
7527:   case Intrinsic::loongarch_lsx_vreplve_b:
7528:   case Intrinsic::loongarch_lsx_vreplve_h:
7529:   case Intrinsic::loongarch_lsx_vreplve_w:
7530:   case Intrinsic::loongarch_lsx_vreplve_d:
7531:   case Intrinsic::loongarch_lasx_xvreplve_b:
7532:   case Intrinsic::loongarch_lasx_xvreplve_h:
7533:   case Intrinsic::loongarch_lasx_xvreplve_w:
7534:   case Intrinsic::loongarch_lasx_xvreplve_d:
7535:     return DAG.getNode(LoongArchISD::VREPLVE, DL, N->getValueType(0),
7536:                        N->getOperand(1),
7537:                        DAG.getNode(ISD::ANY_EXTEND, DL, Subtarget.getGRLenVT(),
7538:                                    N->getOperand(2)));
7539:   case Intrinsic::loongarch_lsx_vpickve2gr_b:
7540:     if (!Subtarget.is64Bit())
7541:       return lowerVectorPickVE2GR<4>(N, DAG, LoongArchISD::VPICK_SEXT_ELT);
7542:     break;
7543:   case Intrinsic::loongarch_lsx_vpickve2gr_h:
7544:   case Intrinsic::loongarch_lasx_xvpickve2gr_w:
7545:     if (!Subtarget.is64Bit())
7546:       return lowerVectorPickVE2GR<3>(N, DAG, LoongArchISD::VPICK_SEXT_ELT);
7547:     break;
7548:   case Intrinsic::loongarch_lsx_vpickve2gr_w:
7549:     if (!Subtarget.is64Bit())
7550:       return lowerVectorPickVE2GR<2>(N, DAG, LoongArchISD::VPICK_SEXT_ELT);
7551:     break;
7552:   case Intrinsic::loongarch_lsx_vpickve2gr_bu:
7553:     if (!Subtarget.is64Bit())
7554:       return lowerVectorPickVE2GR<4>(N, DAG, LoongArchISD::VPICK_ZEXT_ELT);
7555:     break;
7556:   case Intrinsic::loongarch_lsx_vpickve2gr_hu:
7557:   case Intrinsic::loongarch_lasx_xvpickve2gr_wu:
7558:     if (!Subtarget.is64Bit())
7559:       return lowerVectorPickVE2GR<3>(N, DAG, LoongArchISD::VPICK_ZEXT_ELT);
7560:     break;
7561:   case Intrinsic::loongarch_lsx_vpickve2gr_wu:
7562:     if (!Subtarget.is64Bit())
7563:       return lowerVectorPickVE2GR<2>(N, DAG, LoongArchISD::VPICK_ZEXT_ELT);
7564:     break;
7565:   case Intrinsic::loongarch_lsx_bz_b:
7566:   case Intrinsic::loongarch_lsx_bz_h:
7567:   case Intrinsic::loongarch_lsx_bz_w:
7568:   case Intrinsic::loongarch_lsx_bz_d:
7569:   case Intrinsic::loongarch_lasx_xbz_b:
7570:   case Intrinsic::loongarch_lasx_xbz_h:
7571:   case Intrinsic::loongarch_lasx_xbz_w:
7572:   case Intrinsic::loongarch_lasx_xbz_d:
7573:     if (!Subtarget.is64Bit())
7574:       return DAG.getNode(LoongArchISD::VALL_ZERO, DL, N->getValueType(0),
7575:                          N->getOperand(1));
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 7576-7650 / 第 7576-7650 行
```cpp
7576:     break;
7577:   case Intrinsic::loongarch_lsx_bz_v:
7578:   case Intrinsic::loongarch_lasx_xbz_v:
7579:     if (!Subtarget.is64Bit())
7580:       return DAG.getNode(LoongArchISD::VANY_ZERO, DL, N->getValueType(0),
7581:                          N->getOperand(1));
7582:     break;
7583:   case Intrinsic::loongarch_lsx_bnz_b:
7584:   case Intrinsic::loongarch_lsx_bnz_h:
7585:   case Intrinsic::loongarch_lsx_bnz_w:
7586:   case Intrinsic::loongarch_lsx_bnz_d:
7587:   case Intrinsic::loongarch_lasx_xbnz_b:
7588:   case Intrinsic::loongarch_lasx_xbnz_h:
7589:   case Intrinsic::loongarch_lasx_xbnz_w:
7590:   case Intrinsic::loongarch_lasx_xbnz_d:
7591:     if (!Subtarget.is64Bit())
7592:       return DAG.getNode(LoongArchISD::VALL_NONZERO, DL, N->getValueType(0),
7593:                          N->getOperand(1));
7594:     break;
7595:   case Intrinsic::loongarch_lsx_bnz_v:
7596:   case Intrinsic::loongarch_lasx_xbnz_v:
7597:     if (!Subtarget.is64Bit())
7598:       return DAG.getNode(LoongArchISD::VANY_NONZERO, DL, N->getValueType(0),
7599:                          N->getOperand(1));
7600:     break;
7601:   case Intrinsic::loongarch_lasx_concat_128_s:
7602:   case Intrinsic::loongarch_lasx_concat_128_d:
7603:   case Intrinsic::loongarch_lasx_concat_128:
7604:     return DAG.getNode(ISD::CONCAT_VECTORS, DL, N->getValueType(0),
7605:                        N->getOperand(1), N->getOperand(2));
7606:   }
7607:   return SDValue();
7608: }
7609: 
7610: static SDValue performMOVGR2FR_WCombine(SDNode *N, SelectionDAG &DAG,
7611:                                         TargetLowering::DAGCombinerInfo &DCI,
7612:                                         const LoongArchSubtarget &Subtarget) {
7613:   // If the input to MOVGR2FR_W_LA64 is just MOVFR2GR_S_LA64 the the
7614:   // conversion is unnecessary and can be replaced with the
7615:   // MOVFR2GR_S_LA64 operand.
7616:   SDValue Op0 = N->getOperand(0);
7617:   if (Op0.getOpcode() == LoongArchISD::MOVFR2GR_S_LA64)
7618:     return Op0.getOperand(0);
7619:   return SDValue();
7620: }
7621: 
7622: static SDValue performMOVFR2GR_SCombine(SDNode *N, SelectionDAG &DAG,
7623:                                         TargetLowering::DAGCombinerInfo &DCI,
7624:                                         const LoongArchSubtarget &Subtarget) {
7625:   // If the input to MOVFR2GR_S_LA64 is just MOVGR2FR_W_LA64 then the
7626:   // conversion is unnecessary and can be replaced with the MOVGR2FR_W_LA64
7627:   // operand.
7628:   SDValue Op0 = N->getOperand(0);
7629:   if (Op0->getOpcode() == LoongArchISD::MOVGR2FR_W_LA64) {
7630:     assert(Op0.getOperand(0).getValueType() == N->getSimpleValueType(0) &&
7631:            "Unexpected value type!");
7632:     return Op0.getOperand(0);
7633:   }
7634:   return SDValue();
7635: }
7636: 
7637: static SDValue performVMSKLTZCombine(SDNode *N, SelectionDAG &DAG,
7638:                                      TargetLowering::DAGCombinerInfo &DCI,
7639:                                      const LoongArchSubtarget &Subtarget) {
7640:   MVT VT = N->getSimpleValueType(0);
7641:   unsigned NumBits = VT.getScalarSizeInBits();
7642: 
7643:   // Simplify the inputs.
7644:   const TargetLowering &TLI = DAG.getTargetLoweringInfo();
7645:   APInt DemandedMask(APInt::getAllOnes(NumBits));
7646:   if (TLI.SimplifyDemandedBits(SDValue(N, 0), DemandedMask, DCI))
7647:     return SDValue(N, 0);
7648: 
7649:   return SDValue();
7650: }
```
- **EN**: The range implements or declares functions including `performMOVGR2FR_WCombine`, `performMOVFR2GR_SCombine`, `performVMSKLTZCombine`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `performMOVGR2FR_WCombine`, `performMOVFR2GR_SCombine`, `performVMSKLTZCombine` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 7651-7725 / 第 7651-7725 行
```cpp
7651: 
7652: static SDValue
7653: performSPLIT_PAIR_F64Combine(SDNode *N, SelectionDAG &DAG,
7654:                              TargetLowering::DAGCombinerInfo &DCI,
7655:                              const LoongArchSubtarget &Subtarget) {
7656:   SDValue Op0 = N->getOperand(0);
7657:   SDLoc DL(N);
7658: 
7659:   // If the input to SplitPairF64 is just BuildPairF64 then the operation is
7660:   // redundant. Instead, use BuildPairF64's operands directly.
7661:   if (Op0->getOpcode() == LoongArchISD::BUILD_PAIR_F64)
7662:     return DCI.CombineTo(N, Op0.getOperand(0), Op0.getOperand(1));
7663: 
7664:   if (Op0->isUndef()) {
7665:     SDValue Lo = DAG.getUNDEF(MVT::i32);
7666:     SDValue Hi = DAG.getUNDEF(MVT::i32);
7667:     return DCI.CombineTo(N, Lo, Hi);
7668:   }
7669: 
7670:   // It's cheaper to materialise two 32-bit integers than to load a double
7671:   // from the constant pool and transfer it to integer registers through the
7672:   // stack.
7673:   if (ConstantFPSDNode *C = dyn_cast<ConstantFPSDNode>(Op0)) {
7674:     APInt V = C->getValueAPF().bitcastToAPInt();
7675:     SDValue Lo = DAG.getConstant(V.trunc(32), DL, MVT::i32);
7676:     SDValue Hi = DAG.getConstant(V.lshr(32).trunc(32), DL, MVT::i32);
7677:     return DCI.CombineTo(N, Lo, Hi);
7678:   }
7679: 
7680:   return SDValue();
7681: }
7682: 
7683: /// Do target-specific dag combines on LoongArchISD::VANDN nodes.
7684: static SDValue performVANDNCombine(SDNode *N, SelectionDAG &DAG,
7685:                                    TargetLowering::DAGCombinerInfo &DCI,
7686:                                    const LoongArchSubtarget &Subtarget) {
7687:   SDValue N0 = N->getOperand(0);
7688:   SDValue N1 = N->getOperand(1);
7689:   MVT VT = N->getSimpleValueType(0);
7690:   SDLoc DL(N);
7691: 
7692:   // VANDN(undef, x) -> 0
7693:   // VANDN(x, undef) -> 0
7694:   if (N0.isUndef() || N1.isUndef())
7695:     return DAG.getConstant(0, DL, VT);
7696: 
7697:   // VANDN(0, x) -> x
7698:   if (ISD::isBuildVectorAllZeros(N0.getNode()))
7699:     return N1;
7700: 
7701:   // VANDN(x, 0) -> 0
7702:   if (ISD::isBuildVectorAllZeros(N1.getNode()))
7703:     return DAG.getConstant(0, DL, VT);
7704: 
7705:   // VANDN(x, -1) -> NOT(x) -> XOR(x, -1)
7706:   if (ISD::isBuildVectorAllOnes(N1.getNode()))
7707:     return DAG.getNOT(DL, N0, VT);
7708: 
7709:   // Turn VANDN back to AND if input is inverted.
7710:   if (SDValue Not = isNOT(N0, DAG))
7711:     return DAG.getNode(ISD::AND, DL, VT, DAG.getBitcast(VT, Not), N1);
7712: 
7713:   // Folds for better commutativity:
7714:   if (N1->hasOneUse()) {
7715:     // VANDN(x,NOT(y)) -> AND(NOT(x),NOT(y)) -> NOT(OR(X,Y)).
7716:     if (SDValue Not = isNOT(N1, DAG))
7717:       return DAG.getNOT(
7718:           DL, DAG.getNode(ISD::OR, DL, VT, N0, DAG.getBitcast(VT, Not)), VT);
7719: 
7720:     // VANDN(x, SplatVector(Imm)) -> AND(NOT(x), NOT(SplatVector(~Imm)))
7721:     // -> NOT(OR(x, SplatVector(-Imm))
7722:     // Combination is performed only when VT is v16i8/v32i8, using `vnori.b` to
7723:     // gain benefits.
7724:     if (!DCI.isBeforeLegalizeOps() && (VT == MVT::v16i8 || VT == MVT::v32i8) &&
7725:         N1.getOpcode() == ISD::BUILD_VECTOR) {
```
- **EN**: The range implements or declares functions including `performSPLIT_PAIR_F64Combine`, `performVANDNCombine`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `performSPLIT_PAIR_F64Combine`, `performVANDNCombine` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 7726-7800 / 第 7726-7800 行
```cpp
7726:       if (SDValue SplatValue =
7727:               cast<BuildVectorSDNode>(N1.getNode())->getSplatValue()) {
7728:         if (!N1->isOnlyUserOf(SplatValue.getNode()))
7729:           return SDValue();
7730: 
7731:         if (auto *C = dyn_cast<ConstantSDNode>(SplatValue)) {
7732:           uint8_t NCVal = static_cast<uint8_t>(~(C->getSExtValue()));
7733:           SDValue Not =
7734:               DAG.getSplat(VT, DL, DAG.getTargetConstant(NCVal, DL, MVT::i8));
7735:           return DAG.getNOT(
7736:               DL, DAG.getNode(ISD::OR, DL, VT, N0, DAG.getBitcast(VT, Not)),
7737:               VT);
7738:         }
7739:       }
7740:     }
7741:   }
7742: 
7743:   return SDValue();
7744: }
7745: 
7746: static SDValue performSINT_TO_FPCombine(SDNode *N, SelectionDAG &DAG,
7747:                                         TargetLowering::DAGCombinerInfo &DCI,
7748:                                         const LoongArchSubtarget &Subtarget) {
7749:   SDLoc DL(N);
7750:   EVT VT = N->getValueType(0);
7751: 
7752:   if (VT != MVT::f32 && VT != MVT::f64)
7753:     return SDValue();
7754:   if (VT == MVT::f32 && !Subtarget.hasBasicF())
7755:     return SDValue();
7756:   if (VT == MVT::f64 && !Subtarget.hasBasicD())
7757:     return SDValue();
7758: 
7759:   // Only optimize when the source and destination types have the same width.
7760:   if (VT.getSizeInBits() != N->getOperand(0).getValueSizeInBits())
7761:     return SDValue();
7762: 
7763:   SDValue Src = N->getOperand(0);
7764:   // If the result of an integer load is only used by an integer-to-float
7765:   // conversion, use a fp load instead. This eliminates an integer-to-float-move
7766:   // (movgr2fr) instruction.
7767:   if (ISD::isNormalLoad(Src.getNode()) && Src.hasOneUse() &&
7768:       // Do not change the width of a volatile load. This condition check is
7769:       // inspired by AArch64.
7770:       !cast<LoadSDNode>(Src)->isVolatile()) {
7771:     LoadSDNode *LN0 = cast<LoadSDNode>(Src);
7772:     SDValue Load = DAG.getLoad(VT, DL, LN0->getChain(), LN0->getBasePtr(),
7773:                                LN0->getPointerInfo(), LN0->getAlign(),
7774:                                LN0->getMemOperand()->getFlags());
7775: 
7776:     // Make sure successors of the original load stay after it by updating them
7777:     // to use the new Chain.
7778:     DAG.ReplaceAllUsesOfValueWith(SDValue(LN0, 1), Load.getValue(1));
7779:     return DAG.getNode(LoongArchISD::SITOF, SDLoc(N), VT, Load);
7780:   }
7781: 
7782:   return SDValue();
7783: }
7784: 
7785: // Try to widen AND, OR and XOR nodes to VT in order to remove casts around
7786: // logical operations, like in the example below.
7787: //   or (and (truncate x, truncate y)),
7788: //      (xor (truncate z, build_vector (constants)))
7789: // Given a target type \p VT, we generate
7790: //   or (and x, y), (xor z, zext(build_vector (constants)))
7791: // given x, y and z are of type \p VT. We can do so, if operands are either
7792: // truncates from VT types, the second operand is a vector of constants, can
7793: // be recursively promoted or is an existing extension we can extend further.
7794: static SDValue PromoteMaskArithmetic(SDValue N, const SDLoc &DL, EVT VT,
7795:                                      SelectionDAG &DAG,
7796:                                      const LoongArchSubtarget &Subtarget,
7797:                                      unsigned Depth) {
7798:   // Limit recursion to avoid excessive compile times.
7799:   if (Depth >= SelectionDAG::MaxRecursionDepth)
7800:     return SDValue();
```
- **EN**: The range implements or declares functions including `performSINT_TO_FPCombine`, `PromoteMaskArithmetic`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `performSINT_TO_FPCombine`, `PromoteMaskArithmetic` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 7801-7875 / 第 7801-7875 行
```cpp
7801: 
7802:   if (!ISD::isBitwiseLogicOp(N.getOpcode()))
7803:     return SDValue();
7804: 
7805:   SDValue N0 = N.getOperand(0);
7806:   SDValue N1 = N.getOperand(1);
7807: 
7808:   const TargetLowering &TLI = DAG.getTargetLoweringInfo();
7809:   if (!TLI.isOperationLegalOrPromote(N.getOpcode(), VT))
7810:     return SDValue();
7811: 
7812:   if (SDValue NN0 =
7813:           PromoteMaskArithmetic(N0, DL, VT, DAG, Subtarget, Depth + 1))
7814:     N0 = NN0;
7815:   else {
7816:     // The left side has to be a 'trunc'.
7817:     bool LHSTrunc = N0.getOpcode() == ISD::TRUNCATE &&
7818:                     N0.getOperand(0).getValueType() == VT;
7819:     if (LHSTrunc)
7820:       N0 = N0.getOperand(0);
7821:     else
7822:       return SDValue();
7823:   }
7824: 
7825:   if (SDValue NN1 =
7826:           PromoteMaskArithmetic(N1, DL, VT, DAG, Subtarget, Depth + 1))
7827:     N1 = NN1;
7828:   else {
7829:     // The right side has to be a 'trunc', a (foldable) constant or an
7830:     // existing extension we can extend further.
7831:     bool RHSTrunc = N1.getOpcode() == ISD::TRUNCATE &&
7832:                     N1.getOperand(0).getValueType() == VT;
7833:     if (RHSTrunc)
7834:       N1 = N1.getOperand(0);
7835:     else if (ISD::isExtVecInRegOpcode(N1.getOpcode()) && VT.is256BitVector() &&
7836:              Subtarget.hasExtLASX() && N1.hasOneUse())
7837:       N1 = DAG.getNode(N1.getOpcode(), DL, VT, N1.getOperand(0));
7838:     // On 32-bit platform, i64 is an illegal integer scalar type, and
7839:     // FoldConstantArithmetic will fail for v4i64. This may be optimized in the
7840:     // future.
7841:     else if (SDValue Cst =
7842:                  DAG.FoldConstantArithmetic(ISD::ZERO_EXTEND, DL, VT, {N1}))
7843:       N1 = Cst;
7844:     else
7845:       return SDValue();
7846:   }
7847: 
7848:   return DAG.getNode(N.getOpcode(), DL, VT, N0, N1);
7849: }
7850: 
7851: // On LASX the type v4i1/v8i1/v16i1 may be legalized to v4i32/v8i16/v16i8, which
7852: // is LSX-sized register. In most cases we actually compare or select LASX-sized
7853: // registers and mixing the two types creates horrible code. This method
7854: // optimizes some of the transition sequences.
7855: static SDValue PromoteMaskArithmetic(SDValue N, const SDLoc &DL,
7856:                                      SelectionDAG &DAG,
7857:                                      const LoongArchSubtarget &Subtarget) {
7858:   EVT VT = N.getValueType();
7859:   assert(VT.isVector() && "Expected vector type");
7860:   assert((N.getOpcode() == ISD::ANY_EXTEND ||
7861:           N.getOpcode() == ISD::ZERO_EXTEND ||
7862:           N.getOpcode() == ISD::SIGN_EXTEND) &&
7863:          "Invalid Node");
7864: 
7865:   if (!Subtarget.hasExtLASX() || !VT.is256BitVector())
7866:     return SDValue();
7867: 
7868:   SDValue Narrow = N.getOperand(0);
7869:   EVT NarrowVT = Narrow.getValueType();
7870: 
7871:   // Generate the wide operation.
7872:   SDValue Op = PromoteMaskArithmetic(Narrow, DL, VT, DAG, Subtarget, 0);
7873:   if (!Op)
7874:     return SDValue();
7875:   switch (N.getOpcode()) {
```
- **EN**: The range implements or declares functions including `PromoteMaskArithmetic`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `PromoteMaskArithmetic` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 7876-7950 / 第 7876-7950 行
```cpp
7876:   default:
7877:     llvm_unreachable("Unexpected opcode");
7878:   case ISD::ANY_EXTEND:
7879:     return Op;
7880:   case ISD::ZERO_EXTEND:
7881:     return DAG.getZeroExtendInReg(Op, DL, NarrowVT);
7882:   case ISD::SIGN_EXTEND:
7883:     return DAG.getNode(ISD::SIGN_EXTEND_INREG, DL, VT, Op,
7884:                        DAG.getValueType(NarrowVT));
7885:   }
7886: }
7887: 
7888: static SDValue performEXTENDCombine(SDNode *N, SelectionDAG &DAG,
7889:                                     TargetLowering::DAGCombinerInfo &DCI,
7890:                                     const LoongArchSubtarget &Subtarget) {
7891:   EVT VT = N->getValueType(0);
7892:   SDLoc DL(N);
7893: 
7894:   if (VT.isVector())
7895:     if (SDValue R = PromoteMaskArithmetic(SDValue(N, 0), DL, DAG, Subtarget))
7896:       return R;
7897: 
7898:   return SDValue();
7899: }
7900: 
7901: static SDValue
7902: performCONCAT_VECTORSCombine(SDNode *N, SelectionDAG &DAG,
7903:                              TargetLowering::DAGCombinerInfo &DCI,
7904:                              const LoongArchSubtarget &Subtarget) {
7905:   SDLoc DL(N);
7906:   EVT VT = N->getValueType(0);
7907: 
7908:   if (VT.isVector() && N->getNumOperands() == 2)
7909:     if (SDValue R = combineFP_ROUND(SDValue(N, 0), DL, DAG, Subtarget))
7910:       return R;
7911: 
7912:   return SDValue();
7913: }
7914: 
7915: static SDValue performVSELECTCombine(SDNode *N, SelectionDAG &DAG,
7916:                                      TargetLowering::DAGCombinerInfo &DCI,
7917:                                      const LoongArchSubtarget &Subtarget) {
7918:   if (DCI.isBeforeLegalizeOps())
7919:     return SDValue();
7920: 
7921:   EVT VT = N->getValueType(0);
7922:   if (!VT.isVector())
7923:     return SDValue();
7924: 
7925:   if (!DAG.getTargetLoweringInfo().isTypeLegal(VT))
7926:     return SDValue();
7927: 
7928:   EVT EltVT = VT.getVectorElementType();
7929:   if (!EltVT.isInteger())
7930:     return SDValue();
7931: 
7932:   SDValue Cond = N->getOperand(0);
7933:   SDValue TrueVal = N->getOperand(1);
7934:   SDValue FalseVal = N->getOperand(2);
7935: 
7936:   // match:
7937:   //
7938:   // vselect (setcc shift, 0, seteq),
7939:   //         x,
7940:   //         rounded_shift
7941: 
7942:   if (Cond.getOpcode() != ISD::SETCC)
7943:     return SDValue();
7944: 
7945:   if (!ISD::isConstantSplatVectorAllZeros(Cond.getOperand(1).getNode()))
7946:     return SDValue();
7947: 
7948:   auto *CC = cast<CondCodeSDNode>(Cond.getOperand(2));
7949:   if (CC->get() != ISD::SETEQ)
7950:     return SDValue();
```
- **EN**: The range implements or declares functions including `performEXTENDCombine`, `performCONCAT_VECTORSCombine`, `performVSELECTCombine`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `performEXTENDCombine`, `performCONCAT_VECTORSCombine`, `performVSELECTCombine` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 7951-8025 / 第 7951-8025 行
```cpp
7951: 
7952:   SDValue Shift = Cond.getOperand(0);
7953: 
7954:   // True branch must be original value:
7955:   //
7956:   //   vselect cond, x, ...
7957: 
7958:   SDValue X = TrueVal;
7959: 
7960:   // Now match rounded shift pattern:
7961:   //
7962:   //   add
7963:   //     (and
7964:   //        (srl X, shift-1)
7965:   //        1)
7966:   //     (srl/sra X, shift)
7967: 
7968:   if (FalseVal.getOpcode() != ISD::ADD)
7969:     return SDValue();
7970: 
7971:   SDValue Add0 = FalseVal.getOperand(0);
7972:   SDValue Add1 = FalseVal.getOperand(1);
7973:   SDValue And;
7974:   SDValue Shr;
7975: 
7976:   if (Add0.getOpcode() == ISD::AND) {
7977:     And = Add0;
7978:     Shr = Add1;
7979:   } else if (Add1.getOpcode() == ISD::AND) {
7980:     And = Add1;
7981:     Shr = Add0;
7982:   } else {
7983:     return SDValue();
7984:   }
7985: 
7986:   // match:
7987:   //
7988:   //   srl/sra X, shift
7989: 
7990:   if (Shr.getOpcode() != ISD::SRL && Shr.getOpcode() != ISD::SRA)
7991:     return SDValue();
7992: 
7993:   if (Shr.getOperand(0) != X)
7994:     return SDValue();
7995: 
7996:   if (Shr.getOperand(1) != Shift)
7997:     return SDValue();
7998: 
7999:   // match:
8000:   //
8001:   //   and
8002:   //      (srl X, shift-1)
8003:   //      1
8004: 
8005:   SDValue Srl = And.getOperand(0);
8006:   SDValue One = And.getOperand(1);
8007:   APInt SplatVal;
8008: 
8009:   if (Srl.getOpcode() != ISD::SRL)
8010:     return SDValue();
8011: 
8012:   One = peekThroughBitcasts(One);
8013:   if (!isConstantSplatVector(One, SplatVal, EltVT.getSizeInBits()))
8014:     return SDValue();
8015: 
8016:   if (SplatVal != 1)
8017:     return SDValue();
8018: 
8019:   if (Srl.getOperand(0) != X)
8020:     return SDValue();
8021: 
8022:   // match:
8023:   //
8024:   // shift-1
8025: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 8026-8100 / 第 8026-8100 行
```cpp
8026:   SDValue ShiftMinus1 = Srl.getOperand(1);
8027: 
8028:   if (ShiftMinus1.getOpcode() != ISD::ADD)
8029:     return SDValue();
8030: 
8031:   if (ShiftMinus1.getOperand(0) != Shift)
8032:     return SDValue();
8033: 
8034:   if (!ISD::isConstantSplatVectorAllOnes(ShiftMinus1.getOperand(1).getNode()))
8035:     return SDValue();
8036: 
8037:   // We matched a rounded right shift pattern and can lower it
8038:   // to a single vector rounded shift instruction.
8039: 
8040:   SDLoc DL(N);
8041:   return DAG.getNode(Shr.getOpcode() == ISD::SRL ? LoongArchISD::VSRLR
8042:                                                  : LoongArchISD::VSRAR,
8043:                      DL, VT, X, Shift);
8044: }
8045: 
8046: SDValue LoongArchTargetLowering::PerformDAGCombine(SDNode *N,
8047:                                                    DAGCombinerInfo &DCI) const {
8048:   SelectionDAG &DAG = DCI.DAG;
8049:   switch (N->getOpcode()) {
8050:   default:
8051:     break;
8052:   case ISD::ADD:
8053:     return performADDCombine(N, DAG, DCI, Subtarget);
8054:   case ISD::AND:
8055:     return performANDCombine(N, DAG, DCI, Subtarget);
8056:   case ISD::OR:
8057:     return performORCombine(N, DAG, DCI, Subtarget);
8058:   case ISD::SETCC:
8059:     return performSETCCCombine(N, DAG, DCI, Subtarget);
8060:   case ISD::SRL:
8061:     return performSRLCombine(N, DAG, DCI, Subtarget);
8062:   case ISD::BITCAST:
8063:     return performBITCASTCombine(N, DAG, DCI, Subtarget);
8064:   case ISD::ANY_EXTEND:
8065:   case ISD::ZERO_EXTEND:
8066:   case ISD::SIGN_EXTEND:
8067:     return performEXTENDCombine(N, DAG, DCI, Subtarget);
8068:   case ISD::SINT_TO_FP:
8069:     return performSINT_TO_FPCombine(N, DAG, DCI, Subtarget);
8070:   case LoongArchISD::BITREV_W:
8071:     return performBITREV_WCombine(N, DAG, DCI, Subtarget);
8072:   case LoongArchISD::BR_CC:
8073:     return performBR_CCCombine(N, DAG, DCI, Subtarget);
8074:   case LoongArchISD::SELECT_CC:
8075:     return performSELECT_CCCombine(N, DAG, DCI, Subtarget);
8076:   case ISD::INTRINSIC_WO_CHAIN:
8077:     return performINTRINSIC_WO_CHAINCombine(N, DAG, DCI, Subtarget);
8078:   case LoongArchISD::MOVGR2FR_W_LA64:
8079:     return performMOVGR2FR_WCombine(N, DAG, DCI, Subtarget);
8080:   case LoongArchISD::MOVFR2GR_S_LA64:
8081:     return performMOVFR2GR_SCombine(N, DAG, DCI, Subtarget);
8082:   case LoongArchISD::VMSKLTZ:
8083:   case LoongArchISD::XVMSKLTZ:
8084:     return performVMSKLTZCombine(N, DAG, DCI, Subtarget);
8085:   case LoongArchISD::SPLIT_PAIR_F64:
8086:     return performSPLIT_PAIR_F64Combine(N, DAG, DCI, Subtarget);
8087:   case LoongArchISD::VANDN:
8088:     return performVANDNCombine(N, DAG, DCI, Subtarget);
8089:   case ISD::CONCAT_VECTORS:
8090:     return performCONCAT_VECTORSCombine(N, DAG, DCI, Subtarget);
8091:   case ISD::VSELECT:
8092:     return performVSELECTCombine(N, DAG, DCI, Subtarget);
8093:   case LoongArchISD::VPACKEV:
8094:   case LoongArchISD::VPERMI:
8095:     if (SDValue Result =
8096:             combineFP_ROUND(SDValue(N, 0), SDLoc(N), DAG, Subtarget))
8097:       return Result;
8098:   }
8099:   return SDValue();
8100: }
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::PerformDAGCombine`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::PerformDAGCombine` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 8101-8175 / 第 8101-8175 行
```cpp
8101: 
8102: static MachineBasicBlock *insertDivByZeroTrap(MachineInstr &MI,
8103:                                               MachineBasicBlock *MBB) {
8104:   if (!ZeroDivCheck)
8105:     return MBB;
8106: 
8107:   // Build instructions:
8108:   // MBB:
8109:   //   div(or mod)   $dst, $dividend, $divisor
8110:   //   bne           $divisor, $zero, SinkMBB
8111:   // BreakMBB:
8112:   //   break         7 // BRK_DIVZERO
8113:   // SinkMBB:
8114:   //   fallthrough
8115:   const BasicBlock *LLVM_BB = MBB->getBasicBlock();
8116:   MachineFunction::iterator It = ++MBB->getIterator();
8117:   MachineFunction *MF = MBB->getParent();
8118:   auto BreakMBB = MF->CreateMachineBasicBlock(LLVM_BB);
8119:   auto SinkMBB = MF->CreateMachineBasicBlock(LLVM_BB);
8120:   MF->insert(It, BreakMBB);
8121:   MF->insert(It, SinkMBB);
8122: 
8123:   // Transfer the remainder of MBB and its successor edges to SinkMBB.
8124:   SinkMBB->splice(SinkMBB->end(), MBB, std::next(MI.getIterator()), MBB->end());
8125:   SinkMBB->transferSuccessorsAndUpdatePHIs(MBB);
8126: 
8127:   const TargetInstrInfo &TII = *MF->getSubtarget().getInstrInfo();
8128:   DebugLoc DL = MI.getDebugLoc();
8129:   MachineOperand &Divisor = MI.getOperand(2);
8130:   Register DivisorReg = Divisor.getReg();
8131: 
8132:   // MBB:
8133:   BuildMI(MBB, DL, TII.get(LoongArch::BNE))
8134:       .addReg(DivisorReg, getKillRegState(Divisor.isKill()))
8135:       .addReg(LoongArch::R0)
8136:       .addMBB(SinkMBB);
8137:   MBB->addSuccessor(BreakMBB);
8138:   MBB->addSuccessor(SinkMBB);
8139: 
8140:   // BreakMBB:
8141:   // See linux header file arch/loongarch/include/uapi/asm/break.h for the
8142:   // definition of BRK_DIVZERO.
8143:   BuildMI(BreakMBB, DL, TII.get(LoongArch::BREAK)).addImm(7 /*BRK_DIVZERO*/);
8144:   BreakMBB->addSuccessor(SinkMBB);
8145: 
8146:   // Clear Divisor's kill flag.
8147:   Divisor.setIsKill(false);
8148: 
8149:   return SinkMBB;
8150: }
8151: 
8152: static MachineBasicBlock *
8153: emitVecCondBranchPseudo(MachineInstr &MI, MachineBasicBlock *BB,
8154:                         const LoongArchSubtarget &Subtarget) {
8155:   unsigned CondOpc;
8156:   switch (MI.getOpcode()) {
8157:   default:
8158:     llvm_unreachable("Unexpected opcode");
8159:   case LoongArch::PseudoVBZ:
8160:     CondOpc = LoongArch::VSETEQZ_V;
8161:     break;
8162:   case LoongArch::PseudoVBZ_B:
8163:     CondOpc = LoongArch::VSETANYEQZ_B;
8164:     break;
8165:   case LoongArch::PseudoVBZ_H:
8166:     CondOpc = LoongArch::VSETANYEQZ_H;
8167:     break;
8168:   case LoongArch::PseudoVBZ_W:
8169:     CondOpc = LoongArch::VSETANYEQZ_W;
8170:     break;
8171:   case LoongArch::PseudoVBZ_D:
8172:     CondOpc = LoongArch::VSETANYEQZ_D;
8173:     break;
8174:   case LoongArch::PseudoVBNZ:
8175:     CondOpc = LoongArch::VSETNEZ_V;
```
- **EN**: The range implements or declares functions including `BuildMI`, `emitVecCondBranchPseudo`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `emitVecCondBranchPseudo` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 8176-8250 / 第 8176-8250 行
```cpp
8176:     break;
8177:   case LoongArch::PseudoVBNZ_B:
8178:     CondOpc = LoongArch::VSETALLNEZ_B;
8179:     break;
8180:   case LoongArch::PseudoVBNZ_H:
8181:     CondOpc = LoongArch::VSETALLNEZ_H;
8182:     break;
8183:   case LoongArch::PseudoVBNZ_W:
8184:     CondOpc = LoongArch::VSETALLNEZ_W;
8185:     break;
8186:   case LoongArch::PseudoVBNZ_D:
8187:     CondOpc = LoongArch::VSETALLNEZ_D;
8188:     break;
8189:   case LoongArch::PseudoXVBZ:
8190:     CondOpc = LoongArch::XVSETEQZ_V;
8191:     break;
8192:   case LoongArch::PseudoXVBZ_B:
8193:     CondOpc = LoongArch::XVSETANYEQZ_B;
8194:     break;
8195:   case LoongArch::PseudoXVBZ_H:
8196:     CondOpc = LoongArch::XVSETANYEQZ_H;
8197:     break;
8198:   case LoongArch::PseudoXVBZ_W:
8199:     CondOpc = LoongArch::XVSETANYEQZ_W;
8200:     break;
8201:   case LoongArch::PseudoXVBZ_D:
8202:     CondOpc = LoongArch::XVSETANYEQZ_D;
8203:     break;
8204:   case LoongArch::PseudoXVBNZ:
8205:     CondOpc = LoongArch::XVSETNEZ_V;
8206:     break;
8207:   case LoongArch::PseudoXVBNZ_B:
8208:     CondOpc = LoongArch::XVSETALLNEZ_B;
8209:     break;
8210:   case LoongArch::PseudoXVBNZ_H:
8211:     CondOpc = LoongArch::XVSETALLNEZ_H;
8212:     break;
8213:   case LoongArch::PseudoXVBNZ_W:
8214:     CondOpc = LoongArch::XVSETALLNEZ_W;
8215:     break;
8216:   case LoongArch::PseudoXVBNZ_D:
8217:     CondOpc = LoongArch::XVSETALLNEZ_D;
8218:     break;
8219:   }
8220: 
8221:   const TargetInstrInfo *TII = Subtarget.getInstrInfo();
8222:   const BasicBlock *LLVM_BB = BB->getBasicBlock();
8223:   DebugLoc DL = MI.getDebugLoc();
8224:   MachineRegisterInfo &MRI = BB->getParent()->getRegInfo();
8225:   MachineFunction::iterator It = ++BB->getIterator();
8226: 
8227:   MachineFunction *F = BB->getParent();
8228:   MachineBasicBlock *FalseBB = F->CreateMachineBasicBlock(LLVM_BB);
8229:   MachineBasicBlock *TrueBB = F->CreateMachineBasicBlock(LLVM_BB);
8230:   MachineBasicBlock *SinkBB = F->CreateMachineBasicBlock(LLVM_BB);
8231: 
8232:   F->insert(It, FalseBB);
8233:   F->insert(It, TrueBB);
8234:   F->insert(It, SinkBB);
8235: 
8236:   // Transfer the remainder of MBB and its successor edges to Sink.
8237:   SinkBB->splice(SinkBB->end(), BB, std::next(MI.getIterator()), BB->end());
8238:   SinkBB->transferSuccessorsAndUpdatePHIs(BB);
8239: 
8240:   // Insert the real instruction to BB.
8241:   Register FCC = MRI.createVirtualRegister(&LoongArch::CFRRegClass);
8242:   BuildMI(BB, DL, TII->get(CondOpc), FCC).addReg(MI.getOperand(1).getReg());
8243: 
8244:   // Insert branch.
8245:   BuildMI(BB, DL, TII->get(LoongArch::BCNEZ)).addReg(FCC).addMBB(TrueBB);
8246:   BB->addSuccessor(FalseBB);
8247:   BB->addSuccessor(TrueBB);
8248: 
8249:   // FalseBB.
8250:   Register RD1 = MRI.createVirtualRegister(&LoongArch::GPRRegClass);
```
- **EN**: This span continues the file's main responsibility: this file implements SelectionDAG lowering for the target backend for the LoongArch backend.
- **CN**: 这一段延续了该文件的主要职责，继续推进目标相关的后端实现。

### Lines 8251-8325 / 第 8251-8325 行
```cpp
8251:   BuildMI(FalseBB, DL, TII->get(LoongArch::ADDI_W), RD1)
8252:       .addReg(LoongArch::R0)
8253:       .addImm(0);
8254:   BuildMI(FalseBB, DL, TII->get(LoongArch::PseudoBR)).addMBB(SinkBB);
8255:   FalseBB->addSuccessor(SinkBB);
8256: 
8257:   // TrueBB.
8258:   Register RD2 = MRI.createVirtualRegister(&LoongArch::GPRRegClass);
8259:   BuildMI(TrueBB, DL, TII->get(LoongArch::ADDI_W), RD2)
8260:       .addReg(LoongArch::R0)
8261:       .addImm(1);
8262:   TrueBB->addSuccessor(SinkBB);
8263: 
8264:   // SinkBB: merge the results.
8265:   BuildMI(*SinkBB, SinkBB->begin(), DL, TII->get(LoongArch::PHI),
8266:           MI.getOperand(0).getReg())
8267:       .addReg(RD1)
8268:       .addMBB(FalseBB)
8269:       .addReg(RD2)
8270:       .addMBB(TrueBB);
8271: 
8272:   // The pseudo instruction is gone now.
8273:   MI.eraseFromParent();
8274:   return SinkBB;
8275: }
8276: 
8277: static MachineBasicBlock *
8278: emitPseudoXVINSGR2VR(MachineInstr &MI, MachineBasicBlock *BB,
8279:                      const LoongArchSubtarget &Subtarget) {
8280:   unsigned InsOp;
8281:   unsigned BroadcastOp;
8282:   unsigned HalfSize;
8283:   switch (MI.getOpcode()) {
8284:   default:
8285:     llvm_unreachable("Unexpected opcode");
8286:   case LoongArch::PseudoXVINSGR2VR_B:
8287:     HalfSize = 16;
8288:     BroadcastOp = LoongArch::XVREPLGR2VR_B;
8289:     InsOp = LoongArch::XVEXTRINS_B;
8290:     break;
8291:   case LoongArch::PseudoXVINSGR2VR_H:
8292:     HalfSize = 8;
8293:     BroadcastOp = LoongArch::XVREPLGR2VR_H;
8294:     InsOp = LoongArch::XVEXTRINS_H;
8295:     break;
8296:   }
8297:   const TargetInstrInfo *TII = Subtarget.getInstrInfo();
8298:   const TargetRegisterClass *RC = &LoongArch::LASX256RegClass;
8299:   const TargetRegisterClass *SubRC = &LoongArch::LSX128RegClass;
8300:   DebugLoc DL = MI.getDebugLoc();
8301:   MachineRegisterInfo &MRI = BB->getParent()->getRegInfo();
8302:   // XDst = vector_insert XSrc, Elt, Idx
8303:   Register XDst = MI.getOperand(0).getReg();
8304:   Register XSrc = MI.getOperand(1).getReg();
8305:   Register Elt = MI.getOperand(2).getReg();
8306:   unsigned Idx = MI.getOperand(3).getImm();
8307: 
8308:   if (XSrc.isVirtual() && MRI.getVRegDef(XSrc)->isImplicitDef() &&
8309:       Idx < HalfSize) {
8310:     Register ScratchSubReg1 = MRI.createVirtualRegister(SubRC);
8311:     Register ScratchSubReg2 = MRI.createVirtualRegister(SubRC);
8312: 
8313:     BuildMI(*BB, MI, DL, TII->get(LoongArch::COPY), ScratchSubReg1)
8314:         .addReg(XSrc, {}, LoongArch::sub_128);
8315:     BuildMI(*BB, MI, DL,
8316:             TII->get(HalfSize == 8 ? LoongArch::VINSGR2VR_H
8317:                                    : LoongArch::VINSGR2VR_B),
8318:             ScratchSubReg2)
8319:         .addReg(ScratchSubReg1)
8320:         .addReg(Elt)
8321:         .addImm(Idx);
8322: 
8323:     BuildMI(*BB, MI, DL, TII->get(LoongArch::SUBREG_TO_REG), XDst)
8324:         .addReg(ScratchSubReg2)
8325:         .addImm(LoongArch::sub_128);
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `emitPseudoXVINSGR2VR`, `BuildMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `emitPseudoXVINSGR2VR`, `BuildMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 8326-8400 / 第 8326-8400 行
```cpp
8326:   } else {
8327:     Register ScratchReg1 = MRI.createVirtualRegister(RC);
8328:     Register ScratchReg2 = MRI.createVirtualRegister(RC);
8329: 
8330:     BuildMI(*BB, MI, DL, TII->get(BroadcastOp), ScratchReg1).addReg(Elt);
8331: 
8332:     BuildMI(*BB, MI, DL, TII->get(LoongArch::XVPERMI_Q), ScratchReg2)
8333:         .addReg(ScratchReg1)
8334:         .addReg(XSrc)
8335:         .addImm(Idx >= HalfSize ? 48 : 18);
8336: 
8337:     BuildMI(*BB, MI, DL, TII->get(InsOp), XDst)
8338:         .addReg(XSrc)
8339:         .addReg(ScratchReg2)
8340:         .addImm((Idx >= HalfSize ? Idx - HalfSize : Idx) * 17);
8341:   }
8342: 
8343:   MI.eraseFromParent();
8344:   return BB;
8345: }
8346: 
8347: static MachineBasicBlock *emitPseudoCTPOP(MachineInstr &MI,
8348:                                           MachineBasicBlock *BB,
8349:                                           const LoongArchSubtarget &Subtarget) {
8350:   assert(Subtarget.hasExtLSX());
8351:   const TargetInstrInfo *TII = Subtarget.getInstrInfo();
8352:   const TargetRegisterClass *RC = &LoongArch::LSX128RegClass;
8353:   DebugLoc DL = MI.getDebugLoc();
8354:   MachineRegisterInfo &MRI = BB->getParent()->getRegInfo();
8355:   Register Dst = MI.getOperand(0).getReg();
8356:   Register Src = MI.getOperand(1).getReg();
8357: 
8358:   unsigned BroadcastOp, CTOp, PickOp;
8359:   switch (MI.getOpcode()) {
8360:   default:
8361:     llvm_unreachable("Unexpected opcode");
8362:   case LoongArch::PseudoCTPOP_B:
8363:     BroadcastOp = LoongArch::VREPLGR2VR_B;
8364:     CTOp = LoongArch::VPCNT_B;
8365:     PickOp = LoongArch::VPICKVE2GR_B;
8366:     break;
8367:   case LoongArch::PseudoCTPOP_H:
8368:   case LoongArch::PseudoCTPOP_H_LA32:
8369:     BroadcastOp = LoongArch::VREPLGR2VR_H;
8370:     CTOp = LoongArch::VPCNT_H;
8371:     PickOp = LoongArch::VPICKVE2GR_H;
8372:     break;
8373:   case LoongArch::PseudoCTPOP_W:
8374:   case LoongArch::PseudoCTPOP_W_LA32:
8375:     BroadcastOp = LoongArch::VREPLGR2VR_W;
8376:     CTOp = LoongArch::VPCNT_W;
8377:     PickOp = LoongArch::VPICKVE2GR_W;
8378:     break;
8379:   case LoongArch::PseudoCTPOP_D:
8380:     BroadcastOp = LoongArch::VREPLGR2VR_D;
8381:     CTOp = LoongArch::VPCNT_D;
8382:     PickOp = LoongArch::VPICKVE2GR_D;
8383:     break;
8384:   }
8385: 
8386:   Register ScratchReg1 = MRI.createVirtualRegister(RC);
8387:   Register ScratchReg2 = MRI.createVirtualRegister(RC);
8388:   BuildMI(*BB, MI, DL, TII->get(BroadcastOp), ScratchReg1).addReg(Src);
8389:   BuildMI(*BB, MI, DL, TII->get(CTOp), ScratchReg2).addReg(ScratchReg1);
8390:   BuildMI(*BB, MI, DL, TII->get(PickOp), Dst).addReg(ScratchReg2).addImm(0);
8391: 
8392:   MI.eraseFromParent();
8393:   return BB;
8394: }
8395: 
8396: static MachineBasicBlock *
8397: emitPseudoVMSKCOND(MachineInstr &MI, MachineBasicBlock *BB,
8398:                    const LoongArchSubtarget &Subtarget) {
8399:   const TargetInstrInfo *TII = Subtarget.getInstrInfo();
8400:   const TargetRegisterClass *RC = &LoongArch::LSX128RegClass;
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `emitPseudoVMSKCOND`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `emitPseudoVMSKCOND` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 8401-8475 / 第 8401-8475 行
```cpp
8401:   const LoongArchRegisterInfo *TRI = Subtarget.getRegisterInfo();
8402:   MachineRegisterInfo &MRI = BB->getParent()->getRegInfo();
8403:   Register Dst = MI.getOperand(0).getReg();
8404:   Register Src = MI.getOperand(1).getReg();
8405:   DebugLoc DL = MI.getDebugLoc();
8406:   unsigned EleBits = 8;
8407:   unsigned NotOpc = 0;
8408:   unsigned MskOpc;
8409: 
8410:   switch (MI.getOpcode()) {
8411:   default:
8412:     llvm_unreachable("Unexpected opcode");
8413:   case LoongArch::PseudoVMSKLTZ_B:
8414:     MskOpc = LoongArch::VMSKLTZ_B;
8415:     break;
8416:   case LoongArch::PseudoVMSKLTZ_H:
8417:     MskOpc = LoongArch::VMSKLTZ_H;
8418:     EleBits = 16;
8419:     break;
8420:   case LoongArch::PseudoVMSKLTZ_W:
8421:     MskOpc = LoongArch::VMSKLTZ_W;
8422:     EleBits = 32;
8423:     break;
8424:   case LoongArch::PseudoVMSKLTZ_D:
8425:     MskOpc = LoongArch::VMSKLTZ_D;
8426:     EleBits = 64;
8427:     break;
8428:   case LoongArch::PseudoVMSKGEZ_B:
8429:     MskOpc = LoongArch::VMSKGEZ_B;
8430:     break;
8431:   case LoongArch::PseudoVMSKEQZ_B:
8432:     MskOpc = LoongArch::VMSKNZ_B;
8433:     NotOpc = LoongArch::VNOR_V;
8434:     break;
8435:   case LoongArch::PseudoVMSKNEZ_B:
8436:     MskOpc = LoongArch::VMSKNZ_B;
8437:     break;
8438:   case LoongArch::PseudoXVMSKLTZ_B:
8439:     MskOpc = LoongArch::XVMSKLTZ_B;
8440:     RC = &LoongArch::LASX256RegClass;
8441:     break;
8442:   case LoongArch::PseudoXVMSKLTZ_H:
8443:     MskOpc = LoongArch::XVMSKLTZ_H;
8444:     RC = &LoongArch::LASX256RegClass;
8445:     EleBits = 16;
8446:     break;
8447:   case LoongArch::PseudoXVMSKLTZ_W:
8448:     MskOpc = LoongArch::XVMSKLTZ_W;
8449:     RC = &LoongArch::LASX256RegClass;
8450:     EleBits = 32;
8451:     break;
8452:   case LoongArch::PseudoXVMSKLTZ_D:
8453:     MskOpc = LoongArch::XVMSKLTZ_D;
8454:     RC = &LoongArch::LASX256RegClass;
8455:     EleBits = 64;
8456:     break;
8457:   case LoongArch::PseudoXVMSKGEZ_B:
8458:     MskOpc = LoongArch::XVMSKGEZ_B;
8459:     RC = &LoongArch::LASX256RegClass;
8460:     break;
8461:   case LoongArch::PseudoXVMSKEQZ_B:
8462:     MskOpc = LoongArch::XVMSKNZ_B;
8463:     NotOpc = LoongArch::XVNOR_V;
8464:     RC = &LoongArch::LASX256RegClass;
8465:     break;
8466:   case LoongArch::PseudoXVMSKNEZ_B:
8467:     MskOpc = LoongArch::XVMSKNZ_B;
8468:     RC = &LoongArch::LASX256RegClass;
8469:     break;
8470:   }
8471: 
8472:   Register Msk = MRI.createVirtualRegister(RC);
8473:   if (NotOpc) {
8474:     Register Tmp = MRI.createVirtualRegister(RC);
8475:     BuildMI(*BB, MI, DL, TII->get(MskOpc), Tmp).addReg(Src);
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 8476-8550 / 第 8476-8550 行
```cpp
8476:     BuildMI(*BB, MI, DL, TII->get(NotOpc), Msk)
8477:         .addReg(Tmp, RegState::Kill)
8478:         .addReg(Tmp, RegState::Kill);
8479:   } else {
8480:     BuildMI(*BB, MI, DL, TII->get(MskOpc), Msk).addReg(Src);
8481:   }
8482: 
8483:   if (TRI->getRegSizeInBits(*RC) > 128) {
8484:     Register Lo = MRI.createVirtualRegister(&LoongArch::GPRRegClass);
8485:     Register Hi = MRI.createVirtualRegister(&LoongArch::GPRRegClass);
8486:     BuildMI(*BB, MI, DL, TII->get(LoongArch::XVPICKVE2GR_WU), Lo)
8487:         .addReg(Msk)
8488:         .addImm(0);
8489:     BuildMI(*BB, MI, DL, TII->get(LoongArch::XVPICKVE2GR_WU), Hi)
8490:         .addReg(Msk, RegState::Kill)
8491:         .addImm(4);
8492:     BuildMI(*BB, MI, DL,
8493:             TII->get(Subtarget.is64Bit() ? LoongArch::BSTRINS_D
8494:                                          : LoongArch::BSTRINS_W),
8495:             Dst)
8496:         .addReg(Lo, RegState::Kill)
8497:         .addReg(Hi, RegState::Kill)
8498:         .addImm(256 / EleBits - 1)
8499:         .addImm(128 / EleBits);
8500:   } else {
8501:     BuildMI(*BB, MI, DL, TII->get(LoongArch::VPICKVE2GR_HU), Dst)
8502:         .addReg(Msk, RegState::Kill)
8503:         .addImm(0);
8504:   }
8505: 
8506:   MI.eraseFromParent();
8507:   return BB;
8508: }
8509: 
8510: static MachineBasicBlock *
8511: emitSplitPairF64Pseudo(MachineInstr &MI, MachineBasicBlock *BB,
8512:                        const LoongArchSubtarget &Subtarget) {
8513:   assert(MI.getOpcode() == LoongArch::SplitPairF64Pseudo &&
8514:          "Unexpected instruction");
8515: 
8516:   MachineFunction &MF = *BB->getParent();
8517:   DebugLoc DL = MI.getDebugLoc();
8518:   const TargetInstrInfo &TII = *MF.getSubtarget().getInstrInfo();
8519:   Register LoReg = MI.getOperand(0).getReg();
8520:   Register HiReg = MI.getOperand(1).getReg();
8521:   Register SrcReg = MI.getOperand(2).getReg();
8522: 
8523:   BuildMI(*BB, MI, DL, TII.get(LoongArch::MOVFR2GR_S_64), LoReg).addReg(SrcReg);
8524:   BuildMI(*BB, MI, DL, TII.get(LoongArch::MOVFRH2GR_S), HiReg)
8525:       .addReg(SrcReg, getKillRegState(MI.getOperand(2).isKill()));
8526:   MI.eraseFromParent(); // The pseudo instruction is gone now.
8527:   return BB;
8528: }
8529: 
8530: static MachineBasicBlock *
8531: emitBuildPairF64Pseudo(MachineInstr &MI, MachineBasicBlock *BB,
8532:                        const LoongArchSubtarget &Subtarget) {
8533:   assert(MI.getOpcode() == LoongArch::BuildPairF64Pseudo &&
8534:          "Unexpected instruction");
8535: 
8536:   MachineFunction &MF = *BB->getParent();
8537:   DebugLoc DL = MI.getDebugLoc();
8538:   const TargetInstrInfo &TII = *MF.getSubtarget().getInstrInfo();
8539:   MachineRegisterInfo &MRI = BB->getParent()->getRegInfo();
8540:   Register TmpReg = MRI.createVirtualRegister(&LoongArch::FPR64RegClass);
8541:   Register DstReg = MI.getOperand(0).getReg();
8542:   Register LoReg = MI.getOperand(1).getReg();
8543:   Register HiReg = MI.getOperand(2).getReg();
8544: 
8545:   BuildMI(*BB, MI, DL, TII.get(LoongArch::MOVGR2FR_W_64), TmpReg)
8546:       .addReg(LoReg, getKillRegState(MI.getOperand(1).isKill()));
8547:   BuildMI(*BB, MI, DL, TII.get(LoongArch::MOVGR2FRH_W), DstReg)
8548:       .addReg(TmpReg, RegState::Kill)
8549:       .addReg(HiReg, getKillRegState(MI.getOperand(2).isKill()));
8550:   MI.eraseFromParent(); // The pseudo instruction is gone now.
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI`, `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 8551-8625 / 第 8551-8625 行
```cpp
8551:   return BB;
8552: }
8553: 
8554: static bool isSelectPseudo(MachineInstr &MI) {
8555:   switch (MI.getOpcode()) {
8556:   default:
8557:     return false;
8558:   case LoongArch::Select_GPR_Using_CC_GPR:
8559:     return true;
8560:   }
8561: }
8562: 
8563: static MachineBasicBlock *
8564: emitSelectPseudo(MachineInstr &MI, MachineBasicBlock *BB,
8565:                  const LoongArchSubtarget &Subtarget) {
8566:   // To "insert" Select_* instructions, we actually have to insert the triangle
8567:   // control-flow pattern.  The incoming instructions know the destination vreg
8568:   // to set, the condition code register to branch on, the true/false values to
8569:   // select between, and the condcode to use to select the appropriate branch.
8570:   //
8571:   // We produce the following control flow:
8572:   //     HeadMBB
8573:   //     |  \
8574:   //     |  IfFalseMBB
8575:   //     | /
8576:   //    TailMBB
8577:   //
8578:   // When we find a sequence of selects we attempt to optimize their emission
8579:   // by sharing the control flow. Currently we only handle cases where we have
8580:   // multiple selects with the exact same condition (same LHS, RHS and CC).
8581:   // The selects may be interleaved with other instructions if the other
8582:   // instructions meet some requirements we deem safe:
8583:   // - They are not pseudo instructions.
8584:   // - They are debug instructions. Otherwise,
8585:   // - They do not have side-effects, do not access memory and their inputs do
8586:   //   not depend on the results of the select pseudo-instructions.
8587:   // The TrueV/FalseV operands of the selects cannot depend on the result of
8588:   // previous selects in the sequence.
8589:   // These conditions could be further relaxed. See the X86 target for a
8590:   // related approach and more information.
8591: 
8592:   Register LHS = MI.getOperand(1).getReg();
8593:   Register RHS;
8594:   if (MI.getOperand(2).isReg())
8595:     RHS = MI.getOperand(2).getReg();
8596:   auto CC = static_cast<unsigned>(MI.getOperand(3).getImm());
8597: 
8598:   SmallVector<MachineInstr *, 4> SelectDebugValues;
8599:   SmallSet<Register, 4> SelectDests;
8600:   SelectDests.insert(MI.getOperand(0).getReg());
8601: 
8602:   MachineInstr *LastSelectPseudo = &MI;
8603:   for (auto E = BB->end(), SequenceMBBI = MachineBasicBlock::iterator(MI);
8604:        SequenceMBBI != E; ++SequenceMBBI) {
8605:     if (SequenceMBBI->isDebugInstr())
8606:       continue;
8607:     if (isSelectPseudo(*SequenceMBBI)) {
8608:       if (SequenceMBBI->getOperand(1).getReg() != LHS ||
8609:           !SequenceMBBI->getOperand(2).isReg() ||
8610:           SequenceMBBI->getOperand(2).getReg() != RHS ||
8611:           SequenceMBBI->getOperand(3).getImm() != CC ||
8612:           SelectDests.count(SequenceMBBI->getOperand(4).getReg()) ||
8613:           SelectDests.count(SequenceMBBI->getOperand(5).getReg()))
8614:         break;
8615:       LastSelectPseudo = &*SequenceMBBI;
8616:       SequenceMBBI->collectDebugValues(SelectDebugValues);
8617:       SelectDests.insert(SequenceMBBI->getOperand(0).getReg());
8618:       continue;
8619:     }
8620:     if (SequenceMBBI->hasUnmodeledSideEffects() ||
8621:         SequenceMBBI->mayLoadOrStore() ||
8622:         SequenceMBBI->usesCustomInsertionHook())
8623:       break;
8624:     if (llvm::any_of(SequenceMBBI->operands(), [&](MachineOperand &MO) {
8625:           return MO.isReg() && MO.isUse() && SelectDests.count(MO.getReg());
```
- **EN**: The range implements or declares functions including `isSelectPseudo`, `emitSelectPseudo`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isSelectPseudo`, `emitSelectPseudo` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 8626-8700 / 第 8626-8700 行
```cpp
8626:         }))
8627:       break;
8628:   }
8629: 
8630:   const LoongArchInstrInfo &TII = *Subtarget.getInstrInfo();
8631:   const BasicBlock *LLVM_BB = BB->getBasicBlock();
8632:   DebugLoc DL = MI.getDebugLoc();
8633:   MachineFunction::iterator I = ++BB->getIterator();
8634: 
8635:   MachineBasicBlock *HeadMBB = BB;
8636:   MachineFunction *F = BB->getParent();
8637:   MachineBasicBlock *TailMBB = F->CreateMachineBasicBlock(LLVM_BB);
8638:   MachineBasicBlock *IfFalseMBB = F->CreateMachineBasicBlock(LLVM_BB);
8639: 
8640:   F->insert(I, IfFalseMBB);
8641:   F->insert(I, TailMBB);
8642: 
8643:   // Set the call frame size on entry to the new basic blocks.
8644:   unsigned CallFrameSize = TII.getCallFrameSizeAt(*LastSelectPseudo);
8645:   IfFalseMBB->setCallFrameSize(CallFrameSize);
8646:   TailMBB->setCallFrameSize(CallFrameSize);
8647: 
8648:   // Transfer debug instructions associated with the selects to TailMBB.
8649:   for (MachineInstr *DebugInstr : SelectDebugValues) {
8650:     TailMBB->push_back(DebugInstr->removeFromParent());
8651:   }
8652: 
8653:   // Move all instructions after the sequence to TailMBB.
8654:   TailMBB->splice(TailMBB->end(), HeadMBB,
8655:                   std::next(LastSelectPseudo->getIterator()), HeadMBB->end());
8656:   // Update machine-CFG edges by transferring all successors of the current
8657:   // block to the new block which will contain the Phi nodes for the selects.
8658:   TailMBB->transferSuccessorsAndUpdatePHIs(HeadMBB);
8659:   // Set the successors for HeadMBB.
8660:   HeadMBB->addSuccessor(IfFalseMBB);
8661:   HeadMBB->addSuccessor(TailMBB);
8662: 
8663:   // Insert appropriate branch.
8664:   if (MI.getOperand(2).isImm())
8665:     BuildMI(HeadMBB, DL, TII.get(CC))
8666:         .addReg(LHS)
8667:         .addImm(MI.getOperand(2).getImm())
8668:         .addMBB(TailMBB);
8669:   else
8670:     BuildMI(HeadMBB, DL, TII.get(CC)).addReg(LHS).addReg(RHS).addMBB(TailMBB);
8671: 
8672:   // IfFalseMBB just falls through to TailMBB.
8673:   IfFalseMBB->addSuccessor(TailMBB);
8674: 
8675:   // Create PHIs for all of the select pseudo-instructions.
8676:   auto SelectMBBI = MI.getIterator();
8677:   auto SelectEnd = std::next(LastSelectPseudo->getIterator());
8678:   auto InsertionPoint = TailMBB->begin();
8679:   while (SelectMBBI != SelectEnd) {
8680:     auto Next = std::next(SelectMBBI);
8681:     if (isSelectPseudo(*SelectMBBI)) {
8682:       // %Result = phi [ %TrueValue, HeadMBB ], [ %FalseValue, IfFalseMBB ]
8683:       BuildMI(*TailMBB, InsertionPoint, SelectMBBI->getDebugLoc(),
8684:               TII.get(LoongArch::PHI), SelectMBBI->getOperand(0).getReg())
8685:           .addReg(SelectMBBI->getOperand(4).getReg())
8686:           .addMBB(HeadMBB)
8687:           .addReg(SelectMBBI->getOperand(5).getReg())
8688:           .addMBB(IfFalseMBB);
8689:       SelectMBBI->eraseFromParent();
8690:     }
8691:     SelectMBBI = Next;
8692:   }
8693: 
8694:   F->getProperties().resetNoPHIs();
8695:   return TailMBB;
8696: }
8697: 
8698: MachineBasicBlock *LoongArchTargetLowering::EmitInstrWithCustomInserter(
8699:     MachineInstr &MI, MachineBasicBlock *BB) const {
8700:   const TargetInstrInfo *TII = Subtarget.getInstrInfo();
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 8701-8775 / 第 8701-8775 行
```cpp
8701:   DebugLoc DL = MI.getDebugLoc();
8702: 
8703:   switch (MI.getOpcode()) {
8704:   default:
8705:     llvm_unreachable("Unexpected instr type to insert");
8706:   case LoongArch::DIV_W:
8707:   case LoongArch::DIV_WU:
8708:   case LoongArch::MOD_W:
8709:   case LoongArch::MOD_WU:
8710:   case LoongArch::DIV_D:
8711:   case LoongArch::DIV_DU:
8712:   case LoongArch::MOD_D:
8713:   case LoongArch::MOD_DU:
8714:     return insertDivByZeroTrap(MI, BB);
8715:     break;
8716:   case LoongArch::WRFCSR: {
8717:     BuildMI(*BB, MI, DL, TII->get(LoongArch::MOVGR2FCSR),
8718:             LoongArch::FCSR0 + MI.getOperand(0).getImm())
8719:         .addReg(MI.getOperand(1).getReg());
8720:     MI.eraseFromParent();
8721:     return BB;
8722:   }
8723:   case LoongArch::RDFCSR: {
8724:     MachineInstr *ReadFCSR =
8725:         BuildMI(*BB, MI, DL, TII->get(LoongArch::MOVFCSR2GR),
8726:                 MI.getOperand(0).getReg())
8727:             .addReg(LoongArch::FCSR0 + MI.getOperand(1).getImm());
8728:     ReadFCSR->getOperand(1).setIsUndef();
8729:     MI.eraseFromParent();
8730:     return BB;
8731:   }
8732:   case LoongArch::Select_GPR_Using_CC_GPR:
8733:     return emitSelectPseudo(MI, BB, Subtarget);
8734:   case LoongArch::BuildPairF64Pseudo:
8735:     return emitBuildPairF64Pseudo(MI, BB, Subtarget);
8736:   case LoongArch::SplitPairF64Pseudo:
8737:     return emitSplitPairF64Pseudo(MI, BB, Subtarget);
8738:   case LoongArch::PseudoVBZ:
8739:   case LoongArch::PseudoVBZ_B:
8740:   case LoongArch::PseudoVBZ_H:
8741:   case LoongArch::PseudoVBZ_W:
8742:   case LoongArch::PseudoVBZ_D:
8743:   case LoongArch::PseudoVBNZ:
8744:   case LoongArch::PseudoVBNZ_B:
8745:   case LoongArch::PseudoVBNZ_H:
8746:   case LoongArch::PseudoVBNZ_W:
8747:   case LoongArch::PseudoVBNZ_D:
8748:   case LoongArch::PseudoXVBZ:
8749:   case LoongArch::PseudoXVBZ_B:
8750:   case LoongArch::PseudoXVBZ_H:
8751:   case LoongArch::PseudoXVBZ_W:
8752:   case LoongArch::PseudoXVBZ_D:
8753:   case LoongArch::PseudoXVBNZ:
8754:   case LoongArch::PseudoXVBNZ_B:
8755:   case LoongArch::PseudoXVBNZ_H:
8756:   case LoongArch::PseudoXVBNZ_W:
8757:   case LoongArch::PseudoXVBNZ_D:
8758:     return emitVecCondBranchPseudo(MI, BB, Subtarget);
8759:   case LoongArch::PseudoXVINSGR2VR_B:
8760:   case LoongArch::PseudoXVINSGR2VR_H:
8761:     return emitPseudoXVINSGR2VR(MI, BB, Subtarget);
8762:   case LoongArch::PseudoCTPOP_B:
8763:   case LoongArch::PseudoCTPOP_H:
8764:   case LoongArch::PseudoCTPOP_W:
8765:   case LoongArch::PseudoCTPOP_D:
8766:   case LoongArch::PseudoCTPOP_H_LA32:
8767:   case LoongArch::PseudoCTPOP_W_LA32:
8768:     return emitPseudoCTPOP(MI, BB, Subtarget);
8769:   case LoongArch::PseudoVMSKLTZ_B:
8770:   case LoongArch::PseudoVMSKLTZ_H:
8771:   case LoongArch::PseudoVMSKLTZ_W:
8772:   case LoongArch::PseudoVMSKLTZ_D:
8773:   case LoongArch::PseudoVMSKGEZ_B:
8774:   case LoongArch::PseudoVMSKEQZ_B:
8775:   case LoongArch::PseudoVMSKNEZ_B:
```
- **EN**: The range implements or declares functions including `BuildMI`, `BuildMI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `BuildMI`, `BuildMI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 8776-8850 / 第 8776-8850 行
```cpp
8776:   case LoongArch::PseudoXVMSKLTZ_B:
8777:   case LoongArch::PseudoXVMSKLTZ_H:
8778:   case LoongArch::PseudoXVMSKLTZ_W:
8779:   case LoongArch::PseudoXVMSKLTZ_D:
8780:   case LoongArch::PseudoXVMSKGEZ_B:
8781:   case LoongArch::PseudoXVMSKEQZ_B:
8782:   case LoongArch::PseudoXVMSKNEZ_B:
8783:     return emitPseudoVMSKCOND(MI, BB, Subtarget);
8784:   case TargetOpcode::STATEPOINT:
8785:     // STATEPOINT is a pseudo instruction which has no implicit defs/uses
8786:     // while bl call instruction (where statepoint will be lowered at the
8787:     // end) has implicit def. This def is early-clobber as it will be set at
8788:     // the moment of the call and earlier than any use is read.
8789:     // Add this implicit dead def here as a workaround.
8790:     MI.addOperand(*MI.getMF(),
8791:                   MachineOperand::CreateReg(
8792:                       LoongArch::R1, /*isDef*/ true,
8793:                       /*isImp*/ true, /*isKill*/ false, /*isDead*/ true,
8794:                       /*isUndef*/ false, /*isEarlyClobber*/ true));
8795:     if (!Subtarget.is64Bit())
8796:       report_fatal_error("STATEPOINT is only supported on 64-bit targets");
8797:     return emitPatchPoint(MI, BB);
8798:   }
8799: }
8800: 
8801: bool LoongArchTargetLowering::allowsMisalignedMemoryAccesses(
8802:     EVT VT, unsigned AddrSpace, Align Alignment, MachineMemOperand::Flags Flags,
8803:     unsigned *Fast) const {
8804:   if (!Subtarget.hasUAL())
8805:     return false;
8806: 
8807:   // TODO: set reasonable speed number.
8808:   if (Fast)
8809:     *Fast = 1;
8810:   return true;
8811: }
8812: 
8813: //===----------------------------------------------------------------------===//
8814: //                     Calling Convention Implementation
8815: //===----------------------------------------------------------------------===//
8816: 
8817: // Eight general-purpose registers a0-a7 used for passing integer arguments,
8818: // with a0-a1 reused to return values. Generally, the GPRs are used to pass
8819: // fixed-point arguments, and floating-point arguments when no FPR is available
8820: // or with soft float ABI.
8821: const MCPhysReg ArgGPRs[] = {LoongArch::R4,  LoongArch::R5, LoongArch::R6,
8822:                              LoongArch::R7,  LoongArch::R8, LoongArch::R9,
8823:                              LoongArch::R10, LoongArch::R11};
8824: 
8825: // PreserveNone calling convention:
8826: // Arguments may be passed in any general-purpose registers except:
8827: //  - R1  : return address register
8828: //  - R22 : frame pointer
8829: //  - R31 : base pointer
8830: //
8831: // All general-purpose registers are treated as caller-saved,
8832: // except R1 (RA) and R22 (FP).
8833: //
8834: // Non-volatile registers are allocated first so that a function
8835: // can call normal functions without having to spill and reload
8836: // argument registers.
8837: const MCPhysReg PreserveNoneArgGPRs[] = {
8838:     LoongArch::R23, LoongArch::R24, LoongArch::R25, LoongArch::R26,
8839:     LoongArch::R27, LoongArch::R28, LoongArch::R29, LoongArch::R30,
8840:     LoongArch::R4,  LoongArch::R5,  LoongArch::R6,  LoongArch::R7,
8841:     LoongArch::R8,  LoongArch::R9,  LoongArch::R10, LoongArch::R11,
8842:     LoongArch::R12, LoongArch::R13, LoongArch::R14, LoongArch::R15,
8843:     LoongArch::R16, LoongArch::R17, LoongArch::R18, LoongArch::R19,
8844:     LoongArch::R20};
8845: 
8846: // Eight floating-point registers fa0-fa7 used for passing floating-point
8847: // arguments, and fa0-fa1 are also used to return values.
8848: const MCPhysReg ArgFPR32s[] = {LoongArch::F0, LoongArch::F1, LoongArch::F2,
8849:                                LoongArch::F3, LoongArch::F4, LoongArch::F5,
8850:                                LoongArch::F6, LoongArch::F7};
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::allowsMisalignedMemoryAccesses`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::allowsMisalignedMemoryAccesses` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 8851-8925 / 第 8851-8925 行
```cpp
8851: // FPR32 and FPR64 alias each other.
8852: const MCPhysReg ArgFPR64s[] = {
8853:     LoongArch::F0_64, LoongArch::F1_64, LoongArch::F2_64, LoongArch::F3_64,
8854:     LoongArch::F4_64, LoongArch::F5_64, LoongArch::F6_64, LoongArch::F7_64};
8855: 
8856: const MCPhysReg ArgVRs[] = {LoongArch::VR0, LoongArch::VR1, LoongArch::VR2,
8857:                             LoongArch::VR3, LoongArch::VR4, LoongArch::VR5,
8858:                             LoongArch::VR6, LoongArch::VR7};
8859: 
8860: const MCPhysReg ArgXRs[] = {LoongArch::XR0, LoongArch::XR1, LoongArch::XR2,
8861:                             LoongArch::XR3, LoongArch::XR4, LoongArch::XR5,
8862:                             LoongArch::XR6, LoongArch::XR7};
8863: 
8864: static Register allocateArgGPR(CCState &State) {
8865:   switch (State.getCallingConv()) {
8866:   case CallingConv::PreserveNone:
8867:     if (!State.isVarArg())
8868:       return State.AllocateReg(PreserveNoneArgGPRs);
8869:     [[fallthrough]];
8870:   default:
8871:     return State.AllocateReg(ArgGPRs);
8872:   }
8873: }
8874: 
8875: // Pass a 2*GRLen argument that has been split into two GRLen values through
8876: // registers or the stack as necessary.
8877: static bool CC_LoongArchAssign2GRLen(unsigned GRLen, CCState &State,
8878:                                      CCValAssign VA1, ISD::ArgFlagsTy ArgFlags1,
8879:                                      unsigned ValNo2, MVT ValVT2, MVT LocVT2,
8880:                                      ISD::ArgFlagsTy ArgFlags2) {
8881:   unsigned GRLenInBytes = GRLen / 8;
8882:   if (Register Reg = allocateArgGPR(State)) {
8883:     // At least one half can be passed via register.
8884:     State.addLoc(CCValAssign::getReg(VA1.getValNo(), VA1.getValVT(), Reg,
8885:                                      VA1.getLocVT(), CCValAssign::Full));
8886:   } else {
8887:     // Both halves must be passed on the stack, with proper alignment.
8888:     Align StackAlign =
8889:         std::max(Align(GRLenInBytes), ArgFlags1.getNonZeroOrigAlign());
8890:     State.addLoc(
8891:         CCValAssign::getMem(VA1.getValNo(), VA1.getValVT(),
8892:                             State.AllocateStack(GRLenInBytes, StackAlign),
8893:                             VA1.getLocVT(), CCValAssign::Full));
8894:     State.addLoc(CCValAssign::getMem(
8895:         ValNo2, ValVT2, State.AllocateStack(GRLenInBytes, Align(GRLenInBytes)),
8896:         LocVT2, CCValAssign::Full));
8897:     return false;
8898:   }
8899:   if (Register Reg = allocateArgGPR(State)) {
8900:     // The second half can also be passed via register.
8901:     State.addLoc(
8902:         CCValAssign::getReg(ValNo2, ValVT2, Reg, LocVT2, CCValAssign::Full));
8903:   } else {
8904:     // The second half is passed via the stack, without additional alignment.
8905:     State.addLoc(CCValAssign::getMem(
8906:         ValNo2, ValVT2, State.AllocateStack(GRLenInBytes, Align(GRLenInBytes)),
8907:         LocVT2, CCValAssign::Full));
8908:   }
8909:   return false;
8910: }
8911: 
8912: // Implements the LoongArch calling convention. Returns true upon failure.
8913: static bool CC_LoongArch(const DataLayout &DL, LoongArchABI::ABI ABI,
8914:                          unsigned ValNo, MVT ValVT,
8915:                          CCValAssign::LocInfo LocInfo, ISD::ArgFlagsTy ArgFlags,
8916:                          CCState &State, bool IsRet, Type *OrigTy) {
8917:   unsigned GRLen = DL.getLargestLegalIntTypeSizeInBits();
8918:   assert((GRLen == 32 || GRLen == 64) && "Unspport GRLen");
8919:   MVT GRLenVT = GRLen == 32 ? MVT::i32 : MVT::i64;
8920:   MVT LocVT = ValVT;
8921: 
8922:   // Any return value split into more than two values can't be returned
8923:   // directly.
8924:   if (IsRet && ValNo > 1)
8925:     return true;
```
- **EN**: The range implements or declares functions including `allocateArgGPR`, `CC_LoongArchAssign2GRLen`, `CC_LoongArch`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `allocateArgGPR`, `CC_LoongArchAssign2GRLen`, `CC_LoongArch` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 8926-9000 / 第 8926-9000 行
```cpp
8926: 
8927:   // If passing a variadic argument, or if no FPR is available.
8928:   bool UseGPRForFloat = true;
8929: 
8930:   switch (ABI) {
8931:   default:
8932:     llvm_unreachable("Unexpected ABI");
8933:     break;
8934:   case LoongArchABI::ABI_ILP32F:
8935:   case LoongArchABI::ABI_LP64F:
8936:   case LoongArchABI::ABI_ILP32D:
8937:   case LoongArchABI::ABI_LP64D:
8938:     UseGPRForFloat = ArgFlags.isVarArg();
8939:     break;
8940:   case LoongArchABI::ABI_ILP32S:
8941:   case LoongArchABI::ABI_LP64S:
8942:     break;
8943:   }
8944: 
8945:   // If this is a variadic argument, the LoongArch calling convention requires
8946:   // that it is assigned an 'even' or 'aligned' register if it has (2*GRLen)/8
8947:   // byte alignment. An aligned register should be used regardless of whether
8948:   // the original argument was split during legalisation or not. The argument
8949:   // will not be passed by registers if the original type is larger than
8950:   // 2*GRLen, so the register alignment rule does not apply.
8951:   unsigned TwoGRLenInBytes = (2 * GRLen) / 8;
8952:   if (ArgFlags.isVarArg() &&
8953:       ArgFlags.getNonZeroOrigAlign() == TwoGRLenInBytes &&
8954:       DL.getTypeAllocSize(OrigTy) == TwoGRLenInBytes) {
8955:     unsigned RegIdx = State.getFirstUnallocated(ArgGPRs);
8956:     // Skip 'odd' register if necessary.
8957:     if (RegIdx != std::size(ArgGPRs) && RegIdx % 2 == 1)
8958:       State.AllocateReg(ArgGPRs);
8959:   }
8960: 
8961:   SmallVectorImpl<CCValAssign> &PendingLocs = State.getPendingLocs();
8962:   SmallVectorImpl<ISD::ArgFlagsTy> &PendingArgFlags =
8963:       State.getPendingArgFlags();
8964: 
8965:   assert(PendingLocs.size() == PendingArgFlags.size() &&
8966:          "PendingLocs and PendingArgFlags out of sync");
8967: 
8968:   // FPR32 and FPR64 alias each other.
8969:   if (State.getFirstUnallocated(ArgFPR32s) == std::size(ArgFPR32s))
8970:     UseGPRForFloat = true;
8971: 
8972:   if (UseGPRForFloat && ValVT == MVT::f32) {
8973:     LocVT = GRLenVT;
8974:     LocInfo = CCValAssign::BCvt;
8975:   } else if (UseGPRForFloat && GRLen == 64 && ValVT == MVT::f64) {
8976:     LocVT = MVT::i64;
8977:     LocInfo = CCValAssign::BCvt;
8978:   } else if (UseGPRForFloat && GRLen == 32 && ValVT == MVT::f64) {
8979:     // Handle passing f64 on LA32D with a soft float ABI or when floating point
8980:     // registers are exhausted.
8981:     assert(PendingLocs.empty() && "Can't lower f64 if it is split");
8982:     // Depending on available argument GPRS, f64 may be passed in a pair of
8983:     // GPRs, split between a GPR and the stack, or passed completely on the
8984:     // stack. LowerCall/LowerFormalArguments/LowerReturn must recognise these
8985:     // cases.
8986:     MCRegister Reg = allocateArgGPR(State);
8987:     if (!Reg) {
8988:       int64_t StackOffset = State.AllocateStack(8, Align(8));
8989:       State.addLoc(
8990:           CCValAssign::getMem(ValNo, ValVT, StackOffset, LocVT, LocInfo));
8991:       return false;
8992:     }
8993:     LocVT = MVT::i32;
8994:     State.addLoc(CCValAssign::getCustomReg(ValNo, ValVT, Reg, LocVT, LocInfo));
8995:     MCRegister HiReg = allocateArgGPR(State);
8996:     if (HiReg) {
8997:       State.addLoc(
8998:           CCValAssign::getCustomReg(ValNo, ValVT, HiReg, LocVT, LocInfo));
8999:     } else {
9000:       int64_t StackOffset = State.AllocateStack(4, Align(4));
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 9001-9075 / 第 9001-9075 行
```cpp
9001:       State.addLoc(
9002:           CCValAssign::getCustomMem(ValNo, ValVT, StackOffset, LocVT, LocInfo));
9003:     }
9004:     return false;
9005:   }
9006: 
9007:   // Split arguments might be passed indirectly, so keep track of the pending
9008:   // values.
9009:   if (ValVT.isScalarInteger() && (ArgFlags.isSplit() || !PendingLocs.empty())) {
9010:     LocVT = GRLenVT;
9011:     LocInfo = CCValAssign::Indirect;
9012:     PendingLocs.push_back(
9013:         CCValAssign::getPending(ValNo, ValVT, LocVT, LocInfo));
9014:     PendingArgFlags.push_back(ArgFlags);
9015:     if (!ArgFlags.isSplitEnd()) {
9016:       return false;
9017:     }
9018:   }
9019: 
9020:   // If the split argument only had two elements, it should be passed directly
9021:   // in registers or on the stack.
9022:   if (ValVT.isScalarInteger() && ArgFlags.isSplitEnd() &&
9023:       PendingLocs.size() <= 2) {
9024:     assert(PendingLocs.size() == 2 && "Unexpected PendingLocs.size()");
9025:     // Apply the normal calling convention rules to the first half of the
9026:     // split argument.
9027:     CCValAssign VA = PendingLocs[0];
9028:     ISD::ArgFlagsTy AF = PendingArgFlags[0];
9029:     PendingLocs.clear();
9030:     PendingArgFlags.clear();
9031:     return CC_LoongArchAssign2GRLen(GRLen, State, VA, AF, ValNo, ValVT, LocVT,
9032:                                     ArgFlags);
9033:   }
9034: 
9035:   // Allocate to a register if possible, or else a stack slot.
9036:   Register Reg;
9037:   unsigned StoreSizeBytes = GRLen / 8;
9038:   Align StackAlign = Align(GRLen / 8);
9039: 
9040:   if (ValVT == MVT::f32 && !UseGPRForFloat) {
9041:     Reg = State.AllocateReg(ArgFPR32s);
9042:   } else if (ValVT == MVT::f64 && !UseGPRForFloat) {
9043:     Reg = State.AllocateReg(ArgFPR64s);
9044:   } else if (ValVT.is128BitVector()) {
9045:     Reg = State.AllocateReg(ArgVRs);
9046:     UseGPRForFloat = false;
9047:     StoreSizeBytes = 16;
9048:     StackAlign = Align(16);
9049:   } else if (ValVT.is256BitVector()) {
9050:     Reg = State.AllocateReg(ArgXRs);
9051:     UseGPRForFloat = false;
9052:     StoreSizeBytes = 32;
9053:     StackAlign = Align(32);
9054:   } else {
9055:     Reg = allocateArgGPR(State);
9056:   }
9057: 
9058:   unsigned StackOffset =
9059:       Reg ? 0 : State.AllocateStack(StoreSizeBytes, StackAlign);
9060: 
9061:   // If we reach this point and PendingLocs is non-empty, we must be at the
9062:   // end of a split argument that must be passed indirectly.
9063:   if (!PendingLocs.empty()) {
9064:     assert(ArgFlags.isSplitEnd() && "Expected ArgFlags.isSplitEnd()");
9065:     assert(PendingLocs.size() > 2 && "Unexpected PendingLocs.size()");
9066:     for (auto &It : PendingLocs) {
9067:       if (Reg)
9068:         It.convertToReg(Reg);
9069:       else
9070:         It.convertToMem(StackOffset);
9071:       State.addLoc(It);
9072:     }
9073:     PendingLocs.clear();
9074:     PendingArgFlags.clear();
9075:     return false;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 9076-9150 / 第 9076-9150 行
```cpp
9076:   }
9077:   assert((!UseGPRForFloat || LocVT == GRLenVT) &&
9078:          "Expected an GRLenVT at this stage");
9079: 
9080:   if (Reg) {
9081:     State.addLoc(CCValAssign::getReg(ValNo, ValVT, Reg, LocVT, LocInfo));
9082:     return false;
9083:   }
9084: 
9085:   // When a floating-point value is passed on the stack, no bit-cast is needed.
9086:   if (ValVT.isFloatingPoint()) {
9087:     LocVT = ValVT;
9088:     LocInfo = CCValAssign::Full;
9089:   }
9090: 
9091:   State.addLoc(CCValAssign::getMem(ValNo, ValVT, StackOffset, LocVT, LocInfo));
9092:   return false;
9093: }
9094: 
9095: void LoongArchTargetLowering::analyzeInputArgs(
9096:     MachineFunction &MF, CCState &CCInfo,
9097:     const SmallVectorImpl<ISD::InputArg> &Ins, bool IsRet,
9098:     LoongArchCCAssignFn Fn) const {
9099:   FunctionType *FType = MF.getFunction().getFunctionType();
9100:   for (unsigned i = 0, e = Ins.size(); i != e; ++i) {
9101:     MVT ArgVT = Ins[i].VT;
9102:     Type *ArgTy = nullptr;
9103:     if (IsRet)
9104:       ArgTy = FType->getReturnType();
9105:     else if (Ins[i].isOrigArg())
9106:       ArgTy = FType->getParamType(Ins[i].getOrigArgIndex());
9107:     LoongArchABI::ABI ABI =
9108:         MF.getSubtarget<LoongArchSubtarget>().getTargetABI();
9109:     if (Fn(MF.getDataLayout(), ABI, i, ArgVT, CCValAssign::Full, Ins[i].Flags,
9110:            CCInfo, IsRet, ArgTy)) {
9111:       LLVM_DEBUG(dbgs() << "InputArg #" << i << " has unhandled type " << ArgVT
9112:                         << '\n');
9113:       llvm_unreachable("");
9114:     }
9115:   }
9116: }
9117: 
9118: void LoongArchTargetLowering::analyzeOutputArgs(
9119:     MachineFunction &MF, CCState &CCInfo,
9120:     const SmallVectorImpl<ISD::OutputArg> &Outs, bool IsRet,
9121:     CallLoweringInfo *CLI, LoongArchCCAssignFn Fn) const {
9122:   for (unsigned i = 0, e = Outs.size(); i != e; ++i) {
9123:     MVT ArgVT = Outs[i].VT;
9124:     Type *OrigTy = CLI ? CLI->getArgs()[Outs[i].OrigArgIndex].Ty : nullptr;
9125:     LoongArchABI::ABI ABI =
9126:         MF.getSubtarget<LoongArchSubtarget>().getTargetABI();
9127:     if (Fn(MF.getDataLayout(), ABI, i, ArgVT, CCValAssign::Full, Outs[i].Flags,
9128:            CCInfo, IsRet, OrigTy)) {
9129:       LLVM_DEBUG(dbgs() << "OutputArg #" << i << " has unhandled type " << ArgVT
9130:                         << "\n");
9131:       llvm_unreachable("");
9132:     }
9133:   }
9134: }
9135: 
9136: // Convert Val to a ValVT. Should not be called for CCValAssign::Indirect
9137: // values.
9138: static SDValue convertLocVTToValVT(SelectionDAG &DAG, SDValue Val,
9139:                                    const CCValAssign &VA, const SDLoc &DL) {
9140:   switch (VA.getLocInfo()) {
9141:   default:
9142:     llvm_unreachable("Unexpected CCValAssign::LocInfo");
9143:   case CCValAssign::Full:
9144:   case CCValAssign::Indirect:
9145:     break;
9146:   case CCValAssign::BCvt:
9147:     if (VA.getLocVT() == MVT::i64 && VA.getValVT() == MVT::f32)
9148:       Val = DAG.getNode(LoongArchISD::MOVGR2FR_W_LA64, DL, MVT::f32, Val);
9149:     else
9150:       Val = DAG.getNode(ISD::BITCAST, DL, VA.getValVT(), Val);
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::analyzeInputArgs`, `LoongArchTargetLowering::analyzeOutputArgs`, `convertLocVTToValVT`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::analyzeInputArgs`, `LoongArchTargetLowering::analyzeOutputArgs`, `convertLocVTToValVT` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 9151-9225 / 第 9151-9225 行
```cpp
9151:     break;
9152:   }
9153:   return Val;
9154: }
9155: 
9156: static SDValue unpackFromRegLoc(SelectionDAG &DAG, SDValue Chain,
9157:                                 const CCValAssign &VA, const SDLoc &DL,
9158:                                 const ISD::InputArg &In,
9159:                                 const LoongArchTargetLowering &TLI) {
9160:   MachineFunction &MF = DAG.getMachineFunction();
9161:   MachineRegisterInfo &RegInfo = MF.getRegInfo();
9162:   EVT LocVT = VA.getLocVT();
9163:   SDValue Val;
9164:   const TargetRegisterClass *RC = TLI.getRegClassFor(LocVT.getSimpleVT());
9165:   Register VReg = RegInfo.createVirtualRegister(RC);
9166:   RegInfo.addLiveIn(VA.getLocReg(), VReg);
9167:   Val = DAG.getCopyFromReg(Chain, DL, VReg, LocVT);
9168: 
9169:   // If input is sign extended from 32 bits, note it for the OptW pass.
9170:   if (In.isOrigArg()) {
9171:     Argument *OrigArg = MF.getFunction().getArg(In.getOrigArgIndex());
9172:     if (OrigArg->getType()->isIntegerTy()) {
9173:       unsigned BitWidth = OrigArg->getType()->getIntegerBitWidth();
9174:       // An input zero extended from i31 can also be considered sign extended.
9175:       if ((BitWidth <= 32 && In.Flags.isSExt()) ||
9176:           (BitWidth < 32 && In.Flags.isZExt())) {
9177:         LoongArchMachineFunctionInfo *LAFI =
9178:             MF.getInfo<LoongArchMachineFunctionInfo>();
9179:         LAFI->addSExt32Register(VReg);
9180:       }
9181:     }
9182:   }
9183: 
9184:   return convertLocVTToValVT(DAG, Val, VA, DL);
9185: }
9186: 
9187: // The caller is responsible for loading the full value if the argument is
9188: // passed with CCValAssign::Indirect.
9189: static SDValue unpackFromMemLoc(SelectionDAG &DAG, SDValue Chain,
9190:                                 const CCValAssign &VA, const SDLoc &DL) {
9191:   MachineFunction &MF = DAG.getMachineFunction();
9192:   MachineFrameInfo &MFI = MF.getFrameInfo();
9193:   EVT ValVT = VA.getValVT();
9194:   int FI = MFI.CreateFixedObject(ValVT.getStoreSize(), VA.getLocMemOffset(),
9195:                                  /*IsImmutable=*/true);
9196:   SDValue FIN = DAG.getFrameIndex(
9197:       FI, MVT::getIntegerVT(DAG.getDataLayout().getPointerSizeInBits(0)));
9198: 
9199:   ISD::LoadExtType ExtType;
9200:   switch (VA.getLocInfo()) {
9201:   default:
9202:     llvm_unreachable("Unexpected CCValAssign::LocInfo");
9203:   case CCValAssign::Full:
9204:   case CCValAssign::Indirect:
9205:   case CCValAssign::BCvt:
9206:     ExtType = ISD::NON_EXTLOAD;
9207:     break;
9208:   }
9209:   return DAG.getExtLoad(
9210:       ExtType, DL, VA.getLocVT(), Chain, FIN,
9211:       MachinePointerInfo::getFixedStack(DAG.getMachineFunction(), FI), ValVT);
9212: }
9213: 
9214: static SDValue unpackF64OnLA32DSoftABI(SelectionDAG &DAG, SDValue Chain,
9215:                                        const CCValAssign &VA,
9216:                                        const CCValAssign &HiVA,
9217:                                        const SDLoc &DL) {
9218:   assert(VA.getLocVT() == MVT::i32 && VA.getValVT() == MVT::f64 &&
9219:          "Unexpected VA");
9220:   MachineFunction &MF = DAG.getMachineFunction();
9221:   MachineFrameInfo &MFI = MF.getFrameInfo();
9222:   MachineRegisterInfo &RegInfo = MF.getRegInfo();
9223: 
9224:   assert(VA.isRegLoc() && "Expected register VA assignment");
9225: 
```
- **EN**: The range implements or declares functions including `unpackFromRegLoc`, `unpackFromMemLoc`, `unpackF64OnLA32DSoftABI`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `unpackFromRegLoc`, `unpackFromMemLoc`, `unpackF64OnLA32DSoftABI` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 9226-9300 / 第 9226-9300 行
```cpp
9226:   Register LoVReg = RegInfo.createVirtualRegister(&LoongArch::GPRRegClass);
9227:   RegInfo.addLiveIn(VA.getLocReg(), LoVReg);
9228:   SDValue Lo = DAG.getCopyFromReg(Chain, DL, LoVReg, MVT::i32);
9229:   SDValue Hi;
9230:   if (HiVA.isMemLoc()) {
9231:     // Second half of f64 is passed on the stack.
9232:     int FI = MFI.CreateFixedObject(4, HiVA.getLocMemOffset(),
9233:                                    /*IsImmutable=*/true);
9234:     SDValue FIN = DAG.getFrameIndex(FI, MVT::i32);
9235:     Hi = DAG.getLoad(MVT::i32, DL, Chain, FIN,
9236:                      MachinePointerInfo::getFixedStack(MF, FI));
9237:   } else {
9238:     // Second half of f64 is passed in another GPR.
9239:     Register HiVReg = RegInfo.createVirtualRegister(&LoongArch::GPRRegClass);
9240:     RegInfo.addLiveIn(HiVA.getLocReg(), HiVReg);
9241:     Hi = DAG.getCopyFromReg(Chain, DL, HiVReg, MVT::i32);
9242:   }
9243:   return DAG.getNode(LoongArchISD::BUILD_PAIR_F64, DL, MVT::f64, Lo, Hi);
9244: }
9245: 
9246: static SDValue convertValVTToLocVT(SelectionDAG &DAG, SDValue Val,
9247:                                    const CCValAssign &VA, const SDLoc &DL) {
9248:   EVT LocVT = VA.getLocVT();
9249: 
9250:   switch (VA.getLocInfo()) {
9251:   default:
9252:     llvm_unreachable("Unexpected CCValAssign::LocInfo");
9253:   case CCValAssign::Full:
9254:     break;
9255:   case CCValAssign::BCvt:
9256:     if (VA.getLocVT() == MVT::i64 && VA.getValVT() == MVT::f32)
9257:       Val = DAG.getNode(LoongArchISD::MOVFR2GR_S_LA64, DL, MVT::i64, Val);
9258:     else
9259:       Val = DAG.getNode(ISD::BITCAST, DL, LocVT, Val);
9260:     break;
9261:   }
9262:   return Val;
9263: }
9264: 
9265: static bool CC_LoongArch_GHC(unsigned ValNo, MVT ValVT, MVT LocVT,
9266:                              CCValAssign::LocInfo LocInfo,
9267:                              ISD::ArgFlagsTy ArgFlags, Type *OrigTy,
9268:                              CCState &State) {
9269:   if (LocVT == MVT::i32 || LocVT == MVT::i64) {
9270:     // Pass in STG registers: Base, Sp, Hp, R1, R2, R3, R4, R5, SpLim
9271:     //                        s0    s1  s2  s3  s4  s5  s6  s7  s8
9272:     static const MCPhysReg GPRList[] = {
9273:         LoongArch::R23, LoongArch::R24, LoongArch::R25,
9274:         LoongArch::R26, LoongArch::R27, LoongArch::R28,
9275:         LoongArch::R29, LoongArch::R30, LoongArch::R31};
9276:     if (MCRegister Reg = State.AllocateReg(GPRList)) {
9277:       State.addLoc(CCValAssign::getReg(ValNo, ValVT, Reg, LocVT, LocInfo));
9278:       return false;
9279:     }
9280:   }
9281: 
9282:   if (LocVT == MVT::f32) {
9283:     // Pass in STG registers: F1, F2, F3, F4
9284:     //                        fs0,fs1,fs2,fs3
9285:     static const MCPhysReg FPR32List[] = {LoongArch::F24, LoongArch::F25,
9286:                                           LoongArch::F26, LoongArch::F27};
9287:     if (MCRegister Reg = State.AllocateReg(FPR32List)) {
9288:       State.addLoc(CCValAssign::getReg(ValNo, ValVT, Reg, LocVT, LocInfo));
9289:       return false;
9290:     }
9291:   }
9292: 
9293:   if (LocVT == MVT::f64) {
9294:     // Pass in STG registers: D1, D2, D3, D4
9295:     //                        fs4,fs5,fs6,fs7
9296:     static const MCPhysReg FPR64List[] = {LoongArch::F28_64, LoongArch::F29_64,
9297:                                           LoongArch::F30_64, LoongArch::F31_64};
9298:     if (MCRegister Reg = State.AllocateReg(FPR64List)) {
9299:       State.addLoc(CCValAssign::getReg(ValNo, ValVT, Reg, LocVT, LocInfo));
9300:       return false;
```
- **EN**: The range implements or declares functions including `convertValVTToLocVT`, `CC_LoongArch_GHC`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `convertValVTToLocVT`, `CC_LoongArch_GHC` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 9301-9375 / 第 9301-9375 行
```cpp
9301:     }
9302:   }
9303: 
9304:   report_fatal_error("No registers left in GHC calling convention");
9305:   return true;
9306: }
9307: 
9308: // Transform physical registers into virtual registers.
9309: SDValue LoongArchTargetLowering::LowerFormalArguments(
9310:     SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
9311:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &DL,
9312:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
9313: 
9314:   MachineFunction &MF = DAG.getMachineFunction();
9315: 
9316:   switch (CallConv) {
9317:   default:
9318:     llvm_unreachable("Unsupported calling convention");
9319:   case CallingConv::C:
9320:   case CallingConv::Fast:
9321:   case CallingConv::PreserveNone:
9322:   case CallingConv::PreserveMost:
9323:     break;
9324:   case CallingConv::GHC:
9325:     if (!MF.getSubtarget().hasFeature(LoongArch::FeatureBasicF) ||
9326:         !MF.getSubtarget().hasFeature(LoongArch::FeatureBasicD))
9327:       report_fatal_error(
9328:           "GHC calling convention requires the F and D extensions");
9329:   }
9330: 
9331:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
9332:   MVT GRLenVT = Subtarget.getGRLenVT();
9333:   unsigned GRLenInBytes = Subtarget.getGRLen() / 8;
9334:   // Used with varargs to acumulate store chains.
9335:   std::vector<SDValue> OutChains;
9336: 
9337:   // Assign locations to all of the incoming arguments.
9338:   SmallVector<CCValAssign> ArgLocs;
9339:   CCState CCInfo(CallConv, IsVarArg, MF, ArgLocs, *DAG.getContext());
9340: 
9341:   if (CallConv == CallingConv::GHC)
9342:     CCInfo.AnalyzeFormalArguments(Ins, CC_LoongArch_GHC);
9343:   else
9344:     analyzeInputArgs(MF, CCInfo, Ins, /*IsRet=*/false, CC_LoongArch);
9345: 
9346:   for (unsigned i = 0, e = ArgLocs.size(), InsIdx = 0; i != e; ++i, ++InsIdx) {
9347:     CCValAssign &VA = ArgLocs[i];
9348:     SDValue ArgValue;
9349:     // Passing f64 on LA32D with a soft float ABI must be handled as a special
9350:     // case.
9351:     if (VA.getLocVT() == MVT::i32 && VA.getValVT() == MVT::f64) {
9352:       assert(VA.needsCustom());
9353:       ArgValue = unpackF64OnLA32DSoftABI(DAG, Chain, VA, ArgLocs[++i], DL);
9354:     } else if (VA.isRegLoc())
9355:       ArgValue = unpackFromRegLoc(DAG, Chain, VA, DL, Ins[InsIdx], *this);
9356:     else
9357:       ArgValue = unpackFromMemLoc(DAG, Chain, VA, DL);
9358:     if (VA.getLocInfo() == CCValAssign::Indirect) {
9359:       // If the original argument was split and passed by reference, we need to
9360:       // load all parts of it here (using the same address).
9361:       InVals.push_back(DAG.getLoad(VA.getValVT(), DL, Chain, ArgValue,
9362:                                    MachinePointerInfo()));
9363:       unsigned ArgIndex = Ins[InsIdx].OrigArgIndex;
9364:       unsigned ArgPartOffset = Ins[InsIdx].PartOffset;
9365:       assert(ArgPartOffset == 0);
9366:       while (i + 1 != e && Ins[InsIdx + 1].OrigArgIndex == ArgIndex) {
9367:         CCValAssign &PartVA = ArgLocs[i + 1];
9368:         unsigned PartOffset = Ins[InsIdx + 1].PartOffset - ArgPartOffset;
9369:         SDValue Offset = DAG.getIntPtrConstant(PartOffset, DL);
9370:         SDValue Address = DAG.getNode(ISD::ADD, DL, PtrVT, ArgValue, Offset);
9371:         InVals.push_back(DAG.getLoad(PartVA.getValVT(), DL, Chain, Address,
9372:                                      MachinePointerInfo()));
9373:         ++i;
9374:         ++InsIdx;
9375:       }
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::LowerFormalArguments`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::LowerFormalArguments` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 9376-9450 / 第 9376-9450 行
```cpp
9376:       continue;
9377:     }
9378:     InVals.push_back(ArgValue);
9379:   }
9380: 
9381:   if (IsVarArg) {
9382:     ArrayRef<MCPhysReg> ArgRegs = ArrayRef(ArgGPRs);
9383:     unsigned Idx = CCInfo.getFirstUnallocated(ArgRegs);
9384:     const TargetRegisterClass *RC = &LoongArch::GPRRegClass;
9385:     MachineFrameInfo &MFI = MF.getFrameInfo();
9386:     MachineRegisterInfo &RegInfo = MF.getRegInfo();
9387:     auto *LoongArchFI = MF.getInfo<LoongArchMachineFunctionInfo>();
9388: 
9389:     // Offset of the first variable argument from stack pointer, and size of
9390:     // the vararg save area. For now, the varargs save area is either zero or
9391:     // large enough to hold a0-a7.
9392:     int VaArgOffset, VarArgsSaveSize;
9393: 
9394:     // If all registers are allocated, then all varargs must be passed on the
9395:     // stack and we don't need to save any argregs.
9396:     if (ArgRegs.size() == Idx) {
9397:       VaArgOffset = CCInfo.getStackSize();
9398:       VarArgsSaveSize = 0;
9399:     } else {
9400:       VarArgsSaveSize = GRLenInBytes * (ArgRegs.size() - Idx);
9401:       VaArgOffset = -VarArgsSaveSize;
9402:     }
9403: 
9404:     // Record the frame index of the first variable argument
9405:     // which is a value necessary to VASTART.
9406:     int FI = MFI.CreateFixedObject(GRLenInBytes, VaArgOffset, true);
9407:     LoongArchFI->setVarArgsFrameIndex(FI);
9408: 
9409:     // If saving an odd number of registers then create an extra stack slot to
9410:     // ensure that the frame pointer is 2*GRLen-aligned, which in turn ensures
9411:     // offsets to even-numbered registered remain 2*GRLen-aligned.
9412:     if (Idx % 2) {
9413:       MFI.CreateFixedObject(GRLenInBytes, VaArgOffset - (int)GRLenInBytes,
9414:                             true);
9415:       VarArgsSaveSize += GRLenInBytes;
9416:     }
9417: 
9418:     // Copy the integer registers that may have been used for passing varargs
9419:     // to the vararg save area.
9420:     for (unsigned I = Idx; I < ArgRegs.size();
9421:          ++I, VaArgOffset += GRLenInBytes) {
9422:       const Register Reg = RegInfo.createVirtualRegister(RC);
9423:       RegInfo.addLiveIn(ArgRegs[I], Reg);
9424:       SDValue ArgValue = DAG.getCopyFromReg(Chain, DL, Reg, GRLenVT);
9425:       FI = MFI.CreateFixedObject(GRLenInBytes, VaArgOffset, true);
9426:       SDValue PtrOff = DAG.getFrameIndex(FI, getPointerTy(DAG.getDataLayout()));
9427:       SDValue Store = DAG.getStore(Chain, DL, ArgValue, PtrOff,
9428:                                    MachinePointerInfo::getFixedStack(MF, FI));
9429:       cast<StoreSDNode>(Store.getNode())
9430:           ->getMemOperand()
9431:           ->setValue((Value *)nullptr);
9432:       OutChains.push_back(Store);
9433:     }
9434:     LoongArchFI->setVarArgsSaveSize(VarArgsSaveSize);
9435:   }
9436: 
9437:   // All stores are grouped in one node to allow the matching between
9438:   // the size of Ins and InVals. This only happens for vararg functions.
9439:   if (!OutChains.empty()) {
9440:     OutChains.push_back(Chain);
9441:     Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, OutChains);
9442:   }
9443: 
9444:   return Chain;
9445: }
9446: 
9447: bool LoongArchTargetLowering::mayBeEmittedAsTailCall(const CallInst *CI) const {
9448:   return CI->isTailCall();
9449: }
9450: 
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::mayBeEmittedAsTailCall`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::mayBeEmittedAsTailCall` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 9451-9525 / 第 9451-9525 行
```cpp
9451: // Check if the return value is used as only a return value, as otherwise
9452: // we can't perform a tail-call.
9453: bool LoongArchTargetLowering::isUsedByReturnOnly(SDNode *N,
9454:                                                  SDValue &Chain) const {
9455:   if (N->getNumValues() != 1)
9456:     return false;
9457:   if (!N->hasNUsesOfValue(1, 0))
9458:     return false;
9459: 
9460:   SDNode *Copy = *N->user_begin();
9461:   if (Copy->getOpcode() != ISD::CopyToReg)
9462:     return false;
9463: 
9464:   // If the ISD::CopyToReg has a glue operand, we conservatively assume it
9465:   // isn't safe to perform a tail call.
9466:   if (Copy->getGluedNode())
9467:     return false;
9468: 
9469:   // The copy must be used by a LoongArchISD::RET, and nothing else.
9470:   bool HasRet = false;
9471:   for (SDNode *Node : Copy->users()) {
9472:     if (Node->getOpcode() != LoongArchISD::RET)
9473:       return false;
9474:     HasRet = true;
9475:   }
9476: 
9477:   if (!HasRet)
9478:     return false;
9479: 
9480:   Chain = Copy->getOperand(0);
9481:   return true;
9482: }
9483: 
9484: // Check whether the call is eligible for tail call optimization.
9485: bool LoongArchTargetLowering::isEligibleForTailCallOptimization(
9486:     CCState &CCInfo, CallLoweringInfo &CLI, MachineFunction &MF,
9487:     const SmallVectorImpl<CCValAssign> &ArgLocs) const {
9488: 
9489:   auto CalleeCC = CLI.CallConv;
9490:   auto &Outs = CLI.Outs;
9491:   auto &Caller = MF.getFunction();
9492:   auto CallerCC = Caller.getCallingConv();
9493: 
9494:   // Do not tail call opt if the stack is used to pass parameters.
9495:   if (CCInfo.getStackSize() != 0)
9496:     return false;
9497: 
9498:   // Do not tail call opt if any parameters need to be passed indirectly.
9499:   for (auto &VA : ArgLocs)
9500:     if (VA.getLocInfo() == CCValAssign::Indirect)
9501:       return false;
9502: 
9503:   // Do not tail call opt if either caller or callee uses struct return
9504:   // semantics.
9505:   auto IsCallerStructRet = Caller.hasStructRetAttr();
9506:   auto IsCalleeStructRet = Outs.empty() ? false : Outs[0].Flags.isSRet();
9507:   if (IsCallerStructRet || IsCalleeStructRet)
9508:     return false;
9509: 
9510:   // Do not tail call opt if either the callee or caller has a byval argument.
9511:   for (auto &Arg : Outs)
9512:     if (Arg.Flags.isByVal())
9513:       return false;
9514: 
9515:   // The callee has to preserve all registers the caller needs to preserve.
9516:   const LoongArchRegisterInfo *TRI = Subtarget.getRegisterInfo();
9517:   const uint32_t *CallerPreserved = TRI->getCallPreservedMask(MF, CallerCC);
9518:   if (CalleeCC != CallerCC) {
9519:     const uint32_t *CalleePreserved = TRI->getCallPreservedMask(MF, CalleeCC);
9520:     if (!TRI->regmaskSubsetEqual(CallerPreserved, CalleePreserved))
9521:       return false;
9522:   }
9523:   return true;
9524: }
9525: 
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::isUsedByReturnOnly`, `LoongArchTargetLowering::isEligibleForTailCallOptimization`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::isUsedByReturnOnly`, `LoongArchTargetLowering::isEligibleForTailCallOptimization` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 9526-9600 / 第 9526-9600 行
```cpp
9526: static Align getPrefTypeAlign(EVT VT, SelectionDAG &DAG) {
9527:   return DAG.getDataLayout().getPrefTypeAlign(
9528:       VT.getTypeForEVT(*DAG.getContext()));
9529: }
9530: 
9531: // Lower a call to a callseq_start + CALL + callseq_end chain, and add input
9532: // and output parameter nodes.
9533: SDValue
9534: LoongArchTargetLowering::LowerCall(CallLoweringInfo &CLI,
9535:                                    SmallVectorImpl<SDValue> &InVals) const {
9536:   SelectionDAG &DAG = CLI.DAG;
9537:   SDLoc &DL = CLI.DL;
9538:   SmallVectorImpl<ISD::OutputArg> &Outs = CLI.Outs;
9539:   SmallVectorImpl<SDValue> &OutVals = CLI.OutVals;
9540:   SmallVectorImpl<ISD::InputArg> &Ins = CLI.Ins;
9541:   SDValue Chain = CLI.Chain;
9542:   SDValue Callee = CLI.Callee;
9543:   CallingConv::ID CallConv = CLI.CallConv;
9544:   bool IsVarArg = CLI.IsVarArg;
9545:   EVT PtrVT = getPointerTy(DAG.getDataLayout());
9546:   MVT GRLenVT = Subtarget.getGRLenVT();
9547:   bool &IsTailCall = CLI.IsTailCall;
9548: 
9549:   MachineFunction &MF = DAG.getMachineFunction();
9550: 
9551:   // Analyze the operands of the call, assigning locations to each operand.
9552:   SmallVector<CCValAssign> ArgLocs;
9553:   CCState ArgCCInfo(CallConv, IsVarArg, MF, ArgLocs, *DAG.getContext());
9554: 
9555:   if (CallConv == CallingConv::GHC)
9556:     ArgCCInfo.AnalyzeCallOperands(Outs, CC_LoongArch_GHC);
9557:   else
9558:     analyzeOutputArgs(MF, ArgCCInfo, Outs, /*IsRet=*/false, &CLI, CC_LoongArch);
9559: 
9560:   // Check if it's really possible to do a tail call.
9561:   if (IsTailCall)
9562:     IsTailCall = isEligibleForTailCallOptimization(ArgCCInfo, CLI, MF, ArgLocs);
9563: 
9564:   if (IsTailCall)
9565:     ++NumTailCalls;
9566:   else if (CLI.CB && CLI.CB->isMustTailCall())
9567:     report_fatal_error("failed to perform tail call elimination on a call "
9568:                        "site marked musttail");
9569: 
9570:   // Get a count of how many bytes are to be pushed on the stack.
9571:   unsigned NumBytes = ArgCCInfo.getStackSize();
9572: 
9573:   // Create local copies for byval args.
9574:   SmallVector<SDValue> ByValArgs;
9575:   for (unsigned i = 0, e = Outs.size(); i != e; ++i) {
9576:     ISD::ArgFlagsTy Flags = Outs[i].Flags;
9577:     if (!Flags.isByVal())
9578:       continue;
9579: 
9580:     SDValue Arg = OutVals[i];
9581:     unsigned Size = Flags.getByValSize();
9582:     Align Alignment = Flags.getNonZeroByValAlign();
9583: 
9584:     int FI =
9585:         MF.getFrameInfo().CreateStackObject(Size, Alignment, /*isSS=*/false);
9586:     SDValue FIPtr = DAG.getFrameIndex(FI, getPointerTy(DAG.getDataLayout()));
9587:     SDValue SizeNode = DAG.getConstant(Size, DL, GRLenVT);
9588: 
9589:     Chain = DAG.getMemcpy(Chain, DL, FIPtr, Arg, SizeNode, Alignment,
9590:                           /*IsVolatile=*/false,
9591:                           /*AlwaysInline=*/false, /*CI=*/nullptr, std::nullopt,
9592:                           MachinePointerInfo(), MachinePointerInfo());
9593:     ByValArgs.push_back(FIPtr);
9594:   }
9595: 
9596:   if (!IsTailCall)
9597:     Chain = DAG.getCALLSEQ_START(Chain, NumBytes, 0, CLI.DL);
9598: 
9599:   // Copy argument values to their designated locations.
9600:   SmallVector<std::pair<Register, SDValue>> RegsToPass;
```
- **EN**: The range implements or declares functions including `getPrefTypeAlign`, `LoongArchTargetLowering::LowerCall`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `getPrefTypeAlign`, `LoongArchTargetLowering::LowerCall` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 9601-9675 / 第 9601-9675 行
```cpp
9601:   SmallVector<SDValue> MemOpChains;
9602:   SDValue StackPtr;
9603:   for (unsigned i = 0, j = 0, e = ArgLocs.size(), OutIdx = 0; i != e;
9604:        ++i, ++OutIdx) {
9605:     CCValAssign &VA = ArgLocs[i];
9606:     SDValue ArgValue = OutVals[OutIdx];
9607:     ISD::ArgFlagsTy Flags = Outs[OutIdx].Flags;
9608: 
9609:     // Handle passing f64 on LA32D with a soft float ABI as a special case.
9610:     if (VA.getLocVT() == MVT::i32 && VA.getValVT() == MVT::f64) {
9611:       assert(VA.isRegLoc() && "Expected register VA assignment");
9612:       assert(VA.needsCustom());
9613:       SDValue SplitF64 =
9614:           DAG.getNode(LoongArchISD::SPLIT_PAIR_F64, DL,
9615:                       DAG.getVTList(MVT::i32, MVT::i32), ArgValue);
9616:       SDValue Lo = SplitF64.getValue(0);
9617:       SDValue Hi = SplitF64.getValue(1);
9618: 
9619:       Register RegLo = VA.getLocReg();
9620:       RegsToPass.push_back(std::make_pair(RegLo, Lo));
9621: 
9622:       // Get the CCValAssign for the Hi part.
9623:       CCValAssign &HiVA = ArgLocs[++i];
9624: 
9625:       if (HiVA.isMemLoc()) {
9626:         // Second half of f64 is passed on the stack.
9627:         if (!StackPtr.getNode())
9628:           StackPtr = DAG.getCopyFromReg(Chain, DL, LoongArch::R3, PtrVT);
9629:         SDValue Address =
9630:             DAG.getNode(ISD::ADD, DL, PtrVT, StackPtr,
9631:                         DAG.getIntPtrConstant(HiVA.getLocMemOffset(), DL));
9632:         // Emit the store.
9633:         MemOpChains.push_back(DAG.getStore(
9634:             Chain, DL, Hi, Address,
9635:             MachinePointerInfo::getStack(MF, HiVA.getLocMemOffset())));
9636:       } else {
9637:         // Second half of f64 is passed in another GPR.
9638:         Register RegHigh = HiVA.getLocReg();
9639:         RegsToPass.push_back(std::make_pair(RegHigh, Hi));
9640:       }
9641:       continue;
9642:     }
9643: 
9644:     // Promote the value if needed.
9645:     // For now, only handle fully promoted and indirect arguments.
9646:     if (VA.getLocInfo() == CCValAssign::Indirect) {
9647:       // Store the argument in a stack slot and pass its address.
9648:       Align StackAlign =
9649:           std::max(getPrefTypeAlign(Outs[OutIdx].ArgVT, DAG),
9650:                    getPrefTypeAlign(ArgValue.getValueType(), DAG));
9651:       TypeSize StoredSize = ArgValue.getValueType().getStoreSize();
9652:       // If the original argument was split and passed by reference, we need to
9653:       // store the required parts of it here (and pass just one address).
9654:       unsigned ArgIndex = Outs[OutIdx].OrigArgIndex;
9655:       unsigned ArgPartOffset = Outs[OutIdx].PartOffset;
9656:       assert(ArgPartOffset == 0);
9657:       // Calculate the total size to store. We don't have access to what we're
9658:       // actually storing other than performing the loop and collecting the
9659:       // info.
9660:       SmallVector<std::pair<SDValue, SDValue>> Parts;
9661:       while (i + 1 != e && Outs[OutIdx + 1].OrigArgIndex == ArgIndex) {
9662:         SDValue PartValue = OutVals[OutIdx + 1];
9663:         unsigned PartOffset = Outs[OutIdx + 1].PartOffset - ArgPartOffset;
9664:         SDValue Offset = DAG.getIntPtrConstant(PartOffset, DL);
9665:         EVT PartVT = PartValue.getValueType();
9666: 
9667:         StoredSize += PartVT.getStoreSize();
9668:         StackAlign = std::max(StackAlign, getPrefTypeAlign(PartVT, DAG));
9669:         Parts.push_back(std::make_pair(PartValue, Offset));
9670:         ++i;
9671:         ++OutIdx;
9672:       }
9673:       SDValue SpillSlot = DAG.CreateStackTemporary(StoredSize, StackAlign);
9674:       int FI = cast<FrameIndexSDNode>(SpillSlot)->getIndex();
9675:       MemOpChains.push_back(
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 9676-9750 / 第 9676-9750 行
```cpp
9676:           DAG.getStore(Chain, DL, ArgValue, SpillSlot,
9677:                        MachinePointerInfo::getFixedStack(MF, FI)));
9678:       for (const auto &Part : Parts) {
9679:         SDValue PartValue = Part.first;
9680:         SDValue PartOffset = Part.second;
9681:         SDValue Address =
9682:             DAG.getNode(ISD::ADD, DL, PtrVT, SpillSlot, PartOffset);
9683:         MemOpChains.push_back(
9684:             DAG.getStore(Chain, DL, PartValue, Address,
9685:                          MachinePointerInfo::getFixedStack(MF, FI)));
9686:       }
9687:       ArgValue = SpillSlot;
9688:     } else {
9689:       ArgValue = convertValVTToLocVT(DAG, ArgValue, VA, DL);
9690:     }
9691: 
9692:     // Use local copy if it is a byval arg.
9693:     if (Flags.isByVal())
9694:       ArgValue = ByValArgs[j++];
9695: 
9696:     if (VA.isRegLoc()) {
9697:       // Queue up the argument copies and emit them at the end.
9698:       RegsToPass.push_back(std::make_pair(VA.getLocReg(), ArgValue));
9699:     } else {
9700:       assert(VA.isMemLoc() && "Argument not register or memory");
9701:       assert(!IsTailCall && "Tail call not allowed if stack is used "
9702:                             "for passing parameters");
9703: 
9704:       // Work out the address of the stack slot.
9705:       if (!StackPtr.getNode())
9706:         StackPtr = DAG.getCopyFromReg(Chain, DL, LoongArch::R3, PtrVT);
9707:       SDValue Address =
9708:           DAG.getNode(ISD::ADD, DL, PtrVT, StackPtr,
9709:                       DAG.getIntPtrConstant(VA.getLocMemOffset(), DL));
9710: 
9711:       // Emit the store.
9712:       MemOpChains.push_back(
9713:           DAG.getStore(Chain, DL, ArgValue, Address, MachinePointerInfo()));
9714:     }
9715:   }
9716: 
9717:   // Join the stores, which are independent of one another.
9718:   if (!MemOpChains.empty())
9719:     Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, MemOpChains);
9720: 
9721:   SDValue Glue;
9722: 
9723:   // Build a sequence of copy-to-reg nodes, chained and glued together.
9724:   for (auto &Reg : RegsToPass) {
9725:     Chain = DAG.getCopyToReg(Chain, DL, Reg.first, Reg.second, Glue);
9726:     Glue = Chain.getValue(1);
9727:   }
9728: 
9729:   // If the callee is a GlobalAddress/ExternalSymbol node, turn it into a
9730:   // TargetGlobalAddress/TargetExternalSymbol node so that legalize won't
9731:   // split it and then direct call can be matched by PseudoCALL_SMALL.
9732:   if (GlobalAddressSDNode *S = dyn_cast<GlobalAddressSDNode>(Callee)) {
9733:     const GlobalValue *GV = S->getGlobal();
9734:     unsigned OpFlags = getTargetMachine().shouldAssumeDSOLocal(GV)
9735:                            ? LoongArchII::MO_CALL
9736:                            : LoongArchII::MO_CALL_PLT;
9737:     Callee = DAG.getTargetGlobalAddress(S->getGlobal(), DL, PtrVT, 0, OpFlags);
9738:   } else if (ExternalSymbolSDNode *S = dyn_cast<ExternalSymbolSDNode>(Callee)) {
9739:     unsigned OpFlags = getTargetMachine().shouldAssumeDSOLocal(nullptr)
9740:                            ? LoongArchII::MO_CALL
9741:                            : LoongArchII::MO_CALL_PLT;
9742:     Callee = DAG.getTargetExternalSymbol(S->getSymbol(), PtrVT, OpFlags);
9743:   }
9744: 
9745:   // The first call operand is the chain and the second is the target address.
9746:   SmallVector<SDValue> Ops;
9747:   Ops.push_back(Chain);
9748:   Ops.push_back(Callee);
9749: 
9750:   // Add argument registers to the end of the list so that they are
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 9751-9825 / 第 9751-9825 行
```cpp
9751:   // known live into the call.
9752:   for (auto &Reg : RegsToPass)
9753:     Ops.push_back(DAG.getRegister(Reg.first, Reg.second.getValueType()));
9754: 
9755:   if (!IsTailCall) {
9756:     // Add a register mask operand representing the call-preserved registers.
9757:     const TargetRegisterInfo *TRI = Subtarget.getRegisterInfo();
9758:     const uint32_t *Mask = TRI->getCallPreservedMask(MF, CallConv);
9759:     assert(Mask && "Missing call preserved mask for calling convention");
9760:     Ops.push_back(DAG.getRegisterMask(Mask));
9761:   }
9762: 
9763:   // Glue the call to the argument copies, if any.
9764:   if (Glue.getNode())
9765:     Ops.push_back(Glue);
9766: 
9767:   // Emit the call.
9768:   SDVTList NodeTys = DAG.getVTList(MVT::Other, MVT::Glue);
9769:   unsigned Op;
9770:   switch (DAG.getTarget().getCodeModel()) {
9771:   default:
9772:     report_fatal_error("Unsupported code model");
9773:   case CodeModel::Small:
9774:     Op = IsTailCall ? LoongArchISD::TAIL : LoongArchISD::CALL;
9775:     break;
9776:   case CodeModel::Medium:
9777:     Op = IsTailCall ? LoongArchISD::TAIL_MEDIUM : LoongArchISD::CALL_MEDIUM;
9778:     break;
9779:   case CodeModel::Large:
9780:     assert(Subtarget.is64Bit() && "Large code model requires LA64");
9781:     Op = IsTailCall ? LoongArchISD::TAIL_LARGE : LoongArchISD::CALL_LARGE;
9782:     break;
9783:   }
9784: 
9785:   if (IsTailCall) {
9786:     MF.getFrameInfo().setHasTailCall();
9787:     SDValue Ret = DAG.getNode(Op, DL, NodeTys, Ops);
9788:     DAG.addNoMergeSiteInfo(Ret.getNode(), CLI.NoMerge);
9789:     return Ret;
9790:   }
9791: 
9792:   Chain = DAG.getNode(Op, DL, NodeTys, Ops);
9793:   DAG.addNoMergeSiteInfo(Chain.getNode(), CLI.NoMerge);
9794:   Glue = Chain.getValue(1);
9795: 
9796:   // Mark the end of the call, which is glued to the call itself.
9797:   Chain = DAG.getCALLSEQ_END(Chain, NumBytes, 0, Glue, DL);
9798:   Glue = Chain.getValue(1);
9799: 
9800:   // Assign locations to each value returned by this call.
9801:   SmallVector<CCValAssign> RVLocs;
9802:   CCState RetCCInfo(CallConv, IsVarArg, MF, RVLocs, *DAG.getContext());
9803:   analyzeInputArgs(MF, RetCCInfo, Ins, /*IsRet=*/true, CC_LoongArch);
9804: 
9805:   // Copy all of the result registers out of their specified physreg.
9806:   for (unsigned i = 0, e = RVLocs.size(); i != e; ++i) {
9807:     auto &VA = RVLocs[i];
9808:     // Copy the value out.
9809:     SDValue RetValue =
9810:         DAG.getCopyFromReg(Chain, DL, VA.getLocReg(), VA.getLocVT(), Glue);
9811:     // Glue the RetValue to the end of the call sequence.
9812:     Chain = RetValue.getValue(1);
9813:     Glue = RetValue.getValue(2);
9814: 
9815:     if (VA.getLocVT() == MVT::i32 && VA.getValVT() == MVT::f64) {
9816:       assert(VA.needsCustom());
9817:       SDValue RetValue2 = DAG.getCopyFromReg(Chain, DL, RVLocs[++i].getLocReg(),
9818:                                              MVT::i32, Glue);
9819:       Chain = RetValue2.getValue(1);
9820:       Glue = RetValue2.getValue(2);
9821:       RetValue = DAG.getNode(LoongArchISD::BUILD_PAIR_F64, DL, MVT::f64,
9822:                              RetValue, RetValue2);
9823:     } else
9824:       RetValue = convertLocVTToValVT(DAG, RetValue, VA, DL);
9825: 
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 9826-9900 / 第 9826-9900 行
```cpp
9826:     InVals.push_back(RetValue);
9827:   }
9828: 
9829:   return Chain;
9830: }
9831: 
9832: bool LoongArchTargetLowering::CanLowerReturn(
9833:     CallingConv::ID CallConv, MachineFunction &MF, bool IsVarArg,
9834:     const SmallVectorImpl<ISD::OutputArg> &Outs, LLVMContext &Context,
9835:     const Type *RetTy) const {
9836:   SmallVector<CCValAssign> RVLocs;
9837:   CCState CCInfo(CallConv, IsVarArg, MF, RVLocs, Context);
9838: 
9839:   for (unsigned i = 0, e = Outs.size(); i != e; ++i) {
9840:     LoongArchABI::ABI ABI =
9841:         MF.getSubtarget<LoongArchSubtarget>().getTargetABI();
9842:     if (CC_LoongArch(MF.getDataLayout(), ABI, i, Outs[i].VT, CCValAssign::Full,
9843:                      Outs[i].Flags, CCInfo, /*IsRet=*/true, nullptr))
9844:       return false;
9845:   }
9846:   return true;
9847: }
9848: 
9849: SDValue LoongArchTargetLowering::LowerReturn(
9850:     SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
9851:     const SmallVectorImpl<ISD::OutputArg> &Outs,
9852:     const SmallVectorImpl<SDValue> &OutVals, const SDLoc &DL,
9853:     SelectionDAG &DAG) const {
9854:   // Stores the assignment of the return value to a location.
9855:   SmallVector<CCValAssign> RVLocs;
9856: 
9857:   // Info about the registers and stack slot.
9858:   CCState CCInfo(CallConv, IsVarArg, DAG.getMachineFunction(), RVLocs,
9859:                  *DAG.getContext());
9860: 
9861:   analyzeOutputArgs(DAG.getMachineFunction(), CCInfo, Outs, /*IsRet=*/true,
9862:                     nullptr, CC_LoongArch);
9863:   if (CallConv == CallingConv::GHC && !RVLocs.empty())
9864:     report_fatal_error("GHC functions return void only");
9865:   SDValue Glue;
9866:   SmallVector<SDValue, 4> RetOps(1, Chain);
9867: 
9868:   // Copy the result values into the output registers.
9869:   for (unsigned i = 0, e = RVLocs.size(), OutIdx = 0; i < e; ++i, ++OutIdx) {
9870:     SDValue Val = OutVals[OutIdx];
9871:     CCValAssign &VA = RVLocs[i];
9872:     assert(VA.isRegLoc() && "Can only return in registers!");
9873: 
9874:     if (VA.getLocVT() == MVT::i32 && VA.getValVT() == MVT::f64) {
9875:       // Handle returning f64 on LA32D with a soft float ABI.
9876:       assert(VA.isRegLoc() && "Expected return via registers");
9877:       assert(VA.needsCustom());
9878:       SDValue SplitF64 = DAG.getNode(LoongArchISD::SPLIT_PAIR_F64, DL,
9879:                                      DAG.getVTList(MVT::i32, MVT::i32), Val);
9880:       SDValue Lo = SplitF64.getValue(0);
9881:       SDValue Hi = SplitF64.getValue(1);
9882:       Register RegLo = VA.getLocReg();
9883:       Register RegHi = RVLocs[++i].getLocReg();
9884: 
9885:       Chain = DAG.getCopyToReg(Chain, DL, RegLo, Lo, Glue);
9886:       Glue = Chain.getValue(1);
9887:       RetOps.push_back(DAG.getRegister(RegLo, MVT::i32));
9888:       Chain = DAG.getCopyToReg(Chain, DL, RegHi, Hi, Glue);
9889:       Glue = Chain.getValue(1);
9890:       RetOps.push_back(DAG.getRegister(RegHi, MVT::i32));
9891:     } else {
9892:       // Handle a 'normal' return.
9893:       Val = convertValVTToLocVT(DAG, Val, VA, DL);
9894:       Chain = DAG.getCopyToReg(Chain, DL, VA.getLocReg(), Val, Glue);
9895: 
9896:       // Guarantee that all emitted copies are stuck together.
9897:       Glue = Chain.getValue(1);
9898:       RetOps.push_back(DAG.getRegister(VA.getLocReg(), VA.getLocVT()));
9899:     }
9900:   }
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::CanLowerReturn`, `LoongArchTargetLowering::LowerReturn`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::CanLowerReturn`, `LoongArchTargetLowering::LowerReturn` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 9901-9975 / 第 9901-9975 行
```cpp
9901: 
9902:   RetOps[0] = Chain; // Update chain.
9903: 
9904:   // Add the glue node if we have it.
9905:   if (Glue.getNode())
9906:     RetOps.push_back(Glue);
9907: 
9908:   return DAG.getNode(LoongArchISD::RET, DL, MVT::Other, RetOps);
9909: }
9910: 
9911: // Check if a constant splat can be generated using [x]vldi, where imm[12] == 1.
9912: // Note: The following prefixes are excluded:
9913: //   imm[11:8] == 4'b0000, 4'b0100, 4'b1000
9914: // as they can be represented using [x]vrepli.[whb]
9915: std::pair<bool, uint64_t> LoongArchTargetLowering::isImmVLDILegalForMode1(
9916:     const APInt &SplatValue, const unsigned SplatBitSize) const {
9917:   uint64_t RequiredImm = 0;
9918:   uint64_t V = SplatValue.getZExtValue();
9919:   if (SplatBitSize == 16 && !(V & 0x00FF)) {
9920:     // 4'b0101
9921:     RequiredImm = (0b10101 << 8) | (V >> 8);
9922:     return {true, RequiredImm};
9923:   } else if (SplatBitSize == 32) {
9924:     // 4'b0001
9925:     if (!(V & 0xFFFF00FF)) {
9926:       RequiredImm = (0b10001 << 8) | (V >> 8);
9927:       return {true, RequiredImm};
9928:     }
9929:     // 4'b0010
9930:     if (!(V & 0xFF00FFFF)) {
9931:       RequiredImm = (0b10010 << 8) | (V >> 16);
9932:       return {true, RequiredImm};
9933:     }
9934:     // 4'b0011
9935:     if (!(V & 0x00FFFFFF)) {
9936:       RequiredImm = (0b10011 << 8) | (V >> 24);
9937:       return {true, RequiredImm};
9938:     }
9939:     // 4'b0110
9940:     if ((V & 0xFFFF00FF) == 0xFF) {
9941:       RequiredImm = (0b10110 << 8) | (V >> 8);
9942:       return {true, RequiredImm};
9943:     }
9944:     // 4'b0111
9945:     if ((V & 0xFF00FFFF) == 0xFFFF) {
9946:       RequiredImm = (0b10111 << 8) | (V >> 16);
9947:       return {true, RequiredImm};
9948:     }
9949:     // 4'b1010
9950:     if ((V & 0x7E07FFFF) == 0x3E000000 || (V & 0x7E07FFFF) == 0x40000000) {
9951:       RequiredImm =
9952:           (0b11010 << 8) | (((V >> 24) & 0xC0) ^ 0x40) | ((V >> 19) & 0x3F);
9953:       return {true, RequiredImm};
9954:     }
9955:   } else if (SplatBitSize == 64) {
9956:     // 4'b1011
9957:     if ((V & 0xFFFFFFFF7E07FFFFULL) == 0x3E000000ULL ||
9958:         (V & 0xFFFFFFFF7E07FFFFULL) == 0x40000000ULL) {
9959:       RequiredImm =
9960:           (0b11011 << 8) | (((V >> 24) & 0xC0) ^ 0x40) | ((V >> 19) & 0x3F);
9961:       return {true, RequiredImm};
9962:     }
9963:     // 4'b1100
9964:     if ((V & 0x7FC0FFFFFFFFFFFFULL) == 0x4000000000000000ULL ||
9965:         (V & 0x7FC0FFFFFFFFFFFFULL) == 0x3FC0000000000000ULL) {
9966:       RequiredImm =
9967:           (0b11100 << 8) | (((V >> 56) & 0xC0) ^ 0x40) | ((V >> 48) & 0x3F);
9968:       return {true, RequiredImm};
9969:     }
9970:     // 4'b1001
9971:     auto sameBitsPreByte = [](uint64_t x) -> std::pair<bool, uint8_t> {
9972:       uint8_t res = 0;
9973:       for (int i = 0; i < 8; ++i) {
9974:         uint8_t byte = x & 0xFF;
9975:         if (byte == 0 || byte == 0xFF)
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::isImmVLDILegalForMode1`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::isImmVLDILegalForMode1` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 9976-10050 / 第 9976-10050 行
```cpp
9976:           res |= ((byte & 1) << i);
9977:         else
9978:           return {false, 0};
9979:         x >>= 8;
9980:       }
9981:       return {true, res};
9982:     };
9983:     auto [IsSame, Suffix] = sameBitsPreByte(V);
9984:     if (IsSame) {
9985:       RequiredImm = (0b11001 << 8) | Suffix;
9986:       return {true, RequiredImm};
9987:     }
9988:   }
9989:   return {false, RequiredImm};
9990: }
9991: 
9992: bool LoongArchTargetLowering::isFPImmVLDILegal(const APFloat &Imm,
9993:                                                EVT VT) const {
9994:   if (!Subtarget.hasExtLSX())
9995:     return false;
9996: 
9997:   if (VT == MVT::f32) {
9998:     uint64_t masked = Imm.bitcastToAPInt().getZExtValue() & 0x7e07ffff;
9999:     return (masked == 0x3e000000 || masked == 0x40000000);
10000:   }
10001: 
10002:   if (VT == MVT::f64) {
10003:     uint64_t masked = Imm.bitcastToAPInt().getZExtValue() & 0x7fc0ffffffffffff;
10004:     return (masked == 0x3fc0000000000000 || masked == 0x4000000000000000);
10005:   }
10006: 
10007:   return false;
10008: }
10009: 
10010: bool LoongArchTargetLowering::isFPImmLegal(const APFloat &Imm, EVT VT,
10011:                                            bool ForCodeSize) const {
10012:   // TODO: Maybe need more checks here after vector extension is supported.
10013:   if (VT == MVT::f32 && !Subtarget.hasBasicF())
10014:     return false;
10015:   if (VT == MVT::f64 && !Subtarget.hasBasicD())
10016:     return false;
10017:   return (Imm.isZero() || Imm.isExactlyValue(1.0) || isFPImmVLDILegal(Imm, VT));
10018: }
10019: 
10020: bool LoongArchTargetLowering::isCheapToSpeculateCttz(Type *) const {
10021:   return true;
10022: }
10023: 
10024: bool LoongArchTargetLowering::isCheapToSpeculateCtlz(Type *) const {
10025:   return true;
10026: }
10027: 
10028: bool LoongArchTargetLowering::shouldInsertFencesForAtomic(
10029:     const Instruction *I) const {
10030:   if (!Subtarget.is64Bit())
10031:     return isa<LoadInst>(I) || isa<StoreInst>(I);
10032: 
10033:   if (isa<LoadInst>(I))
10034:     return true;
10035: 
10036:   // On LA64, atomic store operations with IntegerBitWidth of 32 and 64 do not
10037:   // require fences beacuse we can use amswap_db.[w/d].
10038:   Type *Ty = I->getOperand(0)->getType();
10039:   if (isa<StoreInst>(I) && Ty->isIntegerTy()) {
10040:     unsigned Size = Ty->getIntegerBitWidth();
10041:     return (Size == 8 || Size == 16);
10042:   }
10043: 
10044:   return false;
10045: }
10046: 
10047: EVT LoongArchTargetLowering::getSetCCResultType(const DataLayout &DL,
10048:                                                 LLVMContext &Context,
10049:                                                 EVT VT) const {
10050:   if (!VT.isVector())
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::isFPImmVLDILegal`, `LoongArchTargetLowering::isFPImmLegal`, `LoongArchTargetLowering::isCheapToSpeculateCttz`, `LoongArchTargetLowering::isCheapToSpeculateCtlz`, `LoongArchTargetLowering::shouldInsertFencesForAtomic`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::isFPImmVLDILegal`, `LoongArchTargetLowering::isFPImmLegal`, `LoongArchTargetLowering::isCheapToSpeculateCttz`, `LoongArchTargetLowering::isCheapToSpeculateCtlz`, `LoongArchTargetLowering::shouldInsertFencesForAtomic` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10051-10125 / 第 10051-10125 行
```cpp
10051:     return getPointerTy(DL);
10052:   return VT.changeVectorElementTypeToInteger();
10053: }
10054: 
10055: bool LoongArchTargetLowering::canMergeStoresTo(
10056:     unsigned AddressSpace, EVT MemVT, const MachineFunction &MF) const {
10057:   // Do not merge to float value size (128 or 256 bits) if no implicit
10058:   // float attribute is set.
10059:   bool NoFloat = MF.getFunction().hasFnAttribute(Attribute::NoImplicitFloat);
10060:   unsigned MaxIntSize = Subtarget.is64Bit() ? 64 : 32;
10061:   if (NoFloat)
10062:     return MemVT.getSizeInBits() <= MaxIntSize;
10063: 
10064:   // Make sure we don't merge greater than our maximum supported vector width.
10065:   if (Subtarget.hasExtLASX())
10066:     MaxIntSize = 256;
10067:   else if (Subtarget.hasExtLSX())
10068:     MaxIntSize = 128;
10069: 
10070:   return MemVT.getSizeInBits() <= MaxIntSize;
10071: }
10072: 
10073: bool LoongArchTargetLowering::hasAndNot(SDValue Y) const {
10074:   EVT VT = Y.getValueType();
10075: 
10076:   if (VT.isVector())
10077:     return Subtarget.hasExtLSX() && VT.isInteger();
10078: 
10079:   return VT.isScalarInteger() && !isa<ConstantSDNode>(Y);
10080: }
10081: 
10082: void LoongArchTargetLowering::getTgtMemIntrinsic(
10083:     SmallVectorImpl<IntrinsicInfo> &Infos, const CallBase &I,
10084:     MachineFunction &MF, unsigned Intrinsic) const {
10085:   switch (Intrinsic) {
10086:   default:
10087:     return;
10088:   case Intrinsic::loongarch_masked_atomicrmw_xchg_i32:
10089:   case Intrinsic::loongarch_masked_atomicrmw_add_i32:
10090:   case Intrinsic::loongarch_masked_atomicrmw_sub_i32:
10091:   case Intrinsic::loongarch_masked_atomicrmw_nand_i32: {
10092:     IntrinsicInfo Info;
10093:     Info.opc = ISD::INTRINSIC_W_CHAIN;
10094:     Info.memVT = MVT::i32;
10095:     Info.ptrVal = I.getArgOperand(0);
10096:     Info.offset = 0;
10097:     Info.align = Align(4);
10098:     Info.flags = MachineMemOperand::MOLoad | MachineMemOperand::MOStore |
10099:                  MachineMemOperand::MOVolatile;
10100:     Infos.push_back(Info);
10101:     return;
10102:     // TODO: Add more Intrinsics later.
10103:   }
10104:   }
10105: }
10106: 
10107: // When -mlamcas is enabled, MinCmpXchgSizeInBits will be set to 8,
10108: // atomicrmw and/or/xor operations with operands less than 32 bits cannot be
10109: // expanded to am{and/or/xor}[_db].w through AtomicExpandPass. To prevent
10110: // regression, we need to implement it manually.
10111: void LoongArchTargetLowering::emitExpandAtomicRMW(AtomicRMWInst *AI) const {
10112:   AtomicRMWInst::BinOp Op = AI->getOperation();
10113: 
10114:   assert((Op == AtomicRMWInst::Or || Op == AtomicRMWInst::Xor ||
10115:           Op == AtomicRMWInst::And) &&
10116:          "Unable to expand");
10117:   unsigned MinWordSize = 4;
10118: 
10119:   IRBuilder<> Builder(AI);
10120:   LLVMContext &Ctx = Builder.getContext();
10121:   const DataLayout &DL = AI->getDataLayout();
10122:   Type *ValueType = AI->getType();
10123:   Type *WordType = Type::getIntNTy(Ctx, MinWordSize * 8);
10124: 
10125:   Value *Addr = AI->getPointerOperand();
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::canMergeStoresTo`, `LoongArchTargetLowering::hasAndNot`, `LoongArchTargetLowering::getTgtMemIntrinsic`, `LoongArchTargetLowering::emitExpandAtomicRMW`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::canMergeStoresTo`, `LoongArchTargetLowering::hasAndNot`, `LoongArchTargetLowering::getTgtMemIntrinsic`, `LoongArchTargetLowering::emitExpandAtomicRMW` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10126-10200 / 第 10126-10200 行
```cpp
10126:   PointerType *PtrTy = cast<PointerType>(Addr->getType());
10127:   IntegerType *IntTy = DL.getIndexType(Ctx, PtrTy->getAddressSpace());
10128: 
10129:   Value *AlignedAddr = Builder.CreateIntrinsic(
10130:       Intrinsic::ptrmask, {PtrTy, IntTy},
10131:       {Addr, ConstantInt::get(IntTy, ~(uint64_t)(MinWordSize - 1))}, nullptr,
10132:       "AlignedAddr");
10133: 
10134:   Value *AddrInt = Builder.CreatePtrToInt(Addr, IntTy);
10135:   Value *PtrLSB = Builder.CreateAnd(AddrInt, MinWordSize - 1, "PtrLSB");
10136:   Value *ShiftAmt = Builder.CreateShl(PtrLSB, 3);
10137:   ShiftAmt = Builder.CreateTrunc(ShiftAmt, WordType, "ShiftAmt");
10138:   Value *Mask = Builder.CreateShl(
10139:       ConstantInt::get(WordType,
10140:                        (1 << (DL.getTypeStoreSize(ValueType) * 8)) - 1),
10141:       ShiftAmt, "Mask");
10142:   Value *Inv_Mask = Builder.CreateNot(Mask, "Inv_Mask");
10143:   Value *ValOperand_Shifted =
10144:       Builder.CreateShl(Builder.CreateZExt(AI->getValOperand(), WordType),
10145:                         ShiftAmt, "ValOperand_Shifted");
10146:   Value *NewOperand;
10147:   if (Op == AtomicRMWInst::And)
10148:     NewOperand = Builder.CreateOr(ValOperand_Shifted, Inv_Mask, "AndOperand");
10149:   else
10150:     NewOperand = ValOperand_Shifted;
10151: 
10152:   AtomicRMWInst *NewAI =
10153:       Builder.CreateAtomicRMW(Op, AlignedAddr, NewOperand, Align(MinWordSize),
10154:                               AI->getOrdering(), AI->getSyncScopeID());
10155: 
10156:   Value *Shift = Builder.CreateLShr(NewAI, ShiftAmt, "shifted");
10157:   Value *Trunc = Builder.CreateTrunc(Shift, ValueType, "extracted");
10158:   Value *FinalOldResult = Builder.CreateBitCast(Trunc, ValueType);
10159:   AI->replaceAllUsesWith(FinalOldResult);
10160:   AI->eraseFromParent();
10161: }
10162: 
10163: TargetLowering::AtomicExpansionKind
10164: LoongArchTargetLowering::shouldExpandAtomicRMWInIR(
10165:     const AtomicRMWInst *AI) const {
10166:   // TODO: Add more AtomicRMWInst that needs to be extended.
10167: 
10168:   // Since floating-point operation requires a non-trivial set of data
10169:   // operations, use CmpXChg to expand.
10170:   if (AI->isFloatingPointOperation() ||
10171:       AI->getOperation() == AtomicRMWInst::UIncWrap ||
10172:       AI->getOperation() == AtomicRMWInst::UDecWrap ||
10173:       AI->getOperation() == AtomicRMWInst::USubCond ||
10174:       AI->getOperation() == AtomicRMWInst::USubSat)
10175:     return AtomicExpansionKind::CmpXChg;
10176: 
10177:   if (Subtarget.hasLAM_BH() && Subtarget.is64Bit() &&
10178:       (AI->getOperation() == AtomicRMWInst::Xchg ||
10179:        AI->getOperation() == AtomicRMWInst::Add ||
10180:        AI->getOperation() == AtomicRMWInst::Sub)) {
10181:     return AtomicExpansionKind::None;
10182:   }
10183: 
10184:   unsigned Size = AI->getType()->getPrimitiveSizeInBits();
10185:   if (Subtarget.hasLAMCAS()) {
10186:     if (Size < 32 && (AI->getOperation() == AtomicRMWInst::And ||
10187:                       AI->getOperation() == AtomicRMWInst::Or ||
10188:                       AI->getOperation() == AtomicRMWInst::Xor))
10189:       return AtomicExpansionKind::CustomExpand;
10190:     if (AI->getOperation() == AtomicRMWInst::Nand || Size < 32)
10191:       return AtomicExpansionKind::CmpXChg;
10192:   }
10193: 
10194:   if (Size == 8 || Size == 16)
10195:     return AtomicExpansionKind::MaskedIntrinsic;
10196:   return AtomicExpansionKind::None;
10197: }
10198: 
10199: static Intrinsic::ID
10200: getIntrinsicForMaskedAtomicRMWBinOp(unsigned GRLen,
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::shouldExpandAtomicRMWInIR`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::shouldExpandAtomicRMWInIR` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10201-10275 / 第 10201-10275 行
```cpp
10201:                                     AtomicRMWInst::BinOp BinOp) {
10202:   if (GRLen == 64) {
10203:     switch (BinOp) {
10204:     default:
10205:       llvm_unreachable("Unexpected AtomicRMW BinOp");
10206:     case AtomicRMWInst::Xchg:
10207:       return Intrinsic::loongarch_masked_atomicrmw_xchg_i64;
10208:     case AtomicRMWInst::Add:
10209:       return Intrinsic::loongarch_masked_atomicrmw_add_i64;
10210:     case AtomicRMWInst::Sub:
10211:       return Intrinsic::loongarch_masked_atomicrmw_sub_i64;
10212:     case AtomicRMWInst::Nand:
10213:       return Intrinsic::loongarch_masked_atomicrmw_nand_i64;
10214:     case AtomicRMWInst::UMax:
10215:       return Intrinsic::loongarch_masked_atomicrmw_umax_i64;
10216:     case AtomicRMWInst::UMin:
10217:       return Intrinsic::loongarch_masked_atomicrmw_umin_i64;
10218:     case AtomicRMWInst::Max:
10219:       return Intrinsic::loongarch_masked_atomicrmw_max_i64;
10220:     case AtomicRMWInst::Min:
10221:       return Intrinsic::loongarch_masked_atomicrmw_min_i64;
10222:       // TODO: support other AtomicRMWInst.
10223:     }
10224:   }
10225: 
10226:   if (GRLen == 32) {
10227:     switch (BinOp) {
10228:     default:
10229:       llvm_unreachable("Unexpected AtomicRMW BinOp");
10230:     case AtomicRMWInst::Xchg:
10231:       return Intrinsic::loongarch_masked_atomicrmw_xchg_i32;
10232:     case AtomicRMWInst::Add:
10233:       return Intrinsic::loongarch_masked_atomicrmw_add_i32;
10234:     case AtomicRMWInst::Sub:
10235:       return Intrinsic::loongarch_masked_atomicrmw_sub_i32;
10236:     case AtomicRMWInst::Nand:
10237:       return Intrinsic::loongarch_masked_atomicrmw_nand_i32;
10238:     case AtomicRMWInst::UMax:
10239:       return Intrinsic::loongarch_masked_atomicrmw_umax_i32;
10240:     case AtomicRMWInst::UMin:
10241:       return Intrinsic::loongarch_masked_atomicrmw_umin_i32;
10242:     case AtomicRMWInst::Max:
10243:       return Intrinsic::loongarch_masked_atomicrmw_max_i32;
10244:     case AtomicRMWInst::Min:
10245:       return Intrinsic::loongarch_masked_atomicrmw_min_i32;
10246:       // TODO: support other AtomicRMWInst.
10247:     }
10248:   }
10249: 
10250:   llvm_unreachable("Unexpected GRLen\n");
10251: }
10252: 
10253: TargetLowering::AtomicExpansionKind
10254: LoongArchTargetLowering::shouldExpandAtomicCmpXchgInIR(
10255:     const AtomicCmpXchgInst *CI) const {
10256: 
10257:   if (Subtarget.hasLAMCAS())
10258:     return AtomicExpansionKind::None;
10259: 
10260:   unsigned Size = CI->getCompareOperand()->getType()->getPrimitiveSizeInBits();
10261:   if (Size == 8 || Size == 16)
10262:     return AtomicExpansionKind::MaskedIntrinsic;
10263:   return AtomicExpansionKind::None;
10264: }
10265: 
10266: Value *LoongArchTargetLowering::emitMaskedAtomicCmpXchgIntrinsic(
10267:     IRBuilderBase &Builder, AtomicCmpXchgInst *CI, Value *AlignedAddr,
10268:     Value *CmpVal, Value *NewVal, Value *Mask, AtomicOrdering Ord) const {
10269:   unsigned GRLen = Subtarget.getGRLen();
10270:   AtomicOrdering FailOrd = CI->getFailureOrdering();
10271:   Value *FailureOrdering =
10272:       Builder.getIntN(Subtarget.getGRLen(), static_cast<uint64_t>(FailOrd));
10273:   Intrinsic::ID CmpXchgIntrID = Intrinsic::loongarch_masked_cmpxchg_i32;
10274:   if (GRLen == 64) {
10275:     CmpXchgIntrID = Intrinsic::loongarch_masked_cmpxchg_i64;
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::shouldExpandAtomicCmpXchgInIR`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::shouldExpandAtomicCmpXchgInIR` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10276-10350 / 第 10276-10350 行
```cpp
10276:     CmpVal = Builder.CreateSExt(CmpVal, Builder.getInt64Ty());
10277:     NewVal = Builder.CreateSExt(NewVal, Builder.getInt64Ty());
10278:     Mask = Builder.CreateSExt(Mask, Builder.getInt64Ty());
10279:   }
10280:   Type *Tys[] = {AlignedAddr->getType()};
10281:   Value *Result = Builder.CreateIntrinsic(
10282:       CmpXchgIntrID, Tys, {AlignedAddr, CmpVal, NewVal, Mask, FailureOrdering});
10283:   if (GRLen == 64)
10284:     Result = Builder.CreateTrunc(Result, Builder.getInt32Ty());
10285:   return Result;
10286: }
10287: 
10288: Value *LoongArchTargetLowering::emitMaskedAtomicRMWIntrinsic(
10289:     IRBuilderBase &Builder, AtomicRMWInst *AI, Value *AlignedAddr, Value *Incr,
10290:     Value *Mask, Value *ShiftAmt, AtomicOrdering Ord) const {
10291:   // In the case of an atomicrmw xchg with a constant 0/-1 operand, replace
10292:   // the atomic instruction with an AtomicRMWInst::And/Or with appropriate
10293:   // mask, as this produces better code than the LL/SC loop emitted by
10294:   // int_loongarch_masked_atomicrmw_xchg.
10295:   if (AI->getOperation() == AtomicRMWInst::Xchg &&
10296:       isa<ConstantInt>(AI->getValOperand())) {
10297:     ConstantInt *CVal = cast<ConstantInt>(AI->getValOperand());
10298:     if (CVal->isZero())
10299:       return Builder.CreateAtomicRMW(AtomicRMWInst::And, AlignedAddr,
10300:                                      Builder.CreateNot(Mask, "Inv_Mask"),
10301:                                      AI->getAlign(), Ord);
10302:     if (CVal->isMinusOne())
10303:       return Builder.CreateAtomicRMW(AtomicRMWInst::Or, AlignedAddr, Mask,
10304:                                      AI->getAlign(), Ord);
10305:   }
10306: 
10307:   unsigned GRLen = Subtarget.getGRLen();
10308:   Value *Ordering =
10309:       Builder.getIntN(GRLen, static_cast<uint64_t>(AI->getOrdering()));
10310:   Type *Tys[] = {AlignedAddr->getType()};
10311:   Function *LlwOpScwLoop = Intrinsic::getOrInsertDeclaration(
10312:       AI->getModule(),
10313:       getIntrinsicForMaskedAtomicRMWBinOp(GRLen, AI->getOperation()), Tys);
10314: 
10315:   if (GRLen == 64) {
10316:     Incr = Builder.CreateSExt(Incr, Builder.getInt64Ty());
10317:     Mask = Builder.CreateSExt(Mask, Builder.getInt64Ty());
10318:     ShiftAmt = Builder.CreateSExt(ShiftAmt, Builder.getInt64Ty());
10319:   }
10320: 
10321:   Value *Result;
10322: 
10323:   // Must pass the shift amount needed to sign extend the loaded value prior
10324:   // to performing a signed comparison for min/max. ShiftAmt is the number of
10325:   // bits to shift the value into position. Pass GRLen-ShiftAmt-ValWidth, which
10326:   // is the number of bits to left+right shift the value in order to
10327:   // sign-extend.
10328:   if (AI->getOperation() == AtomicRMWInst::Min ||
10329:       AI->getOperation() == AtomicRMWInst::Max) {
10330:     const DataLayout &DL = AI->getDataLayout();
10331:     unsigned ValWidth =
10332:         DL.getTypeStoreSizeInBits(AI->getValOperand()->getType());
10333:     Value *SextShamt =
10334:         Builder.CreateSub(Builder.getIntN(GRLen, GRLen - ValWidth), ShiftAmt);
10335:     Result = Builder.CreateCall(LlwOpScwLoop,
10336:                                 {AlignedAddr, Incr, Mask, SextShamt, Ordering});
10337:   } else {
10338:     Result =
10339:         Builder.CreateCall(LlwOpScwLoop, {AlignedAddr, Incr, Mask, Ordering});
10340:   }
10341: 
10342:   if (GRLen == 64)
10343:     Result = Builder.CreateTrunc(Result, Builder.getInt32Ty());
10344:   return Result;
10345: }
10346: 
10347: bool LoongArchTargetLowering::isFMAFasterThanFMulAndFAdd(
10348:     const MachineFunction &MF, EVT VT) const {
10349:   VT = VT.getScalarType();
10350: 
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::isFMAFasterThanFMulAndFAdd`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::isFMAFasterThanFMulAndFAdd` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10351-10425 / 第 10351-10425 行
```cpp
10351:   if (!VT.isSimple())
10352:     return false;
10353: 
10354:   switch (VT.getSimpleVT().SimpleTy) {
10355:   case MVT::f32:
10356:   case MVT::f64:
10357:     return true;
10358:   default:
10359:     break;
10360:   }
10361: 
10362:   return false;
10363: }
10364: 
10365: Register LoongArchTargetLowering::getExceptionPointerRegister(
10366:     const Constant *PersonalityFn) const {
10367:   return LoongArch::R4;
10368: }
10369: 
10370: Register LoongArchTargetLowering::getExceptionSelectorRegister(
10371:     const Constant *PersonalityFn) const {
10372:   return LoongArch::R5;
10373: }
10374: 
10375: //===----------------------------------------------------------------------===//
10376: // Target Optimization Hooks
10377: //===----------------------------------------------------------------------===//
10378: 
10379: static int getEstimateRefinementSteps(EVT VT,
10380:                                       const LoongArchSubtarget &Subtarget) {
10381:   // Feature FRECIPE instrucions relative accuracy is 2^-14.
10382:   // IEEE float has 23 digits and double has 52 digits.
10383:   int RefinementSteps = VT.getScalarType() == MVT::f64 ? 2 : 1;
10384:   return RefinementSteps;
10385: }
10386: 
10387: static bool
10388: isSupportedReciprocalEstimateType(EVT VT, const LoongArchSubtarget &Subtarget) {
10389:   assert(Subtarget.hasFrecipe() &&
10390:          "Reciprocal estimate queried on unsupported target");
10391: 
10392:   if (!VT.isSimple())
10393:     return false;
10394: 
10395:   switch (VT.getSimpleVT().SimpleTy) {
10396:   case MVT::f32:
10397:     // f32 is the base type for reciprocal estimate instructions.
10398:     return true;
10399: 
10400:   case MVT::f64:
10401:     return Subtarget.hasBasicD();
10402: 
10403:   case MVT::v4f32:
10404:   case MVT::v2f64:
10405:     return Subtarget.hasExtLSX();
10406: 
10407:   case MVT::v8f32:
10408:   case MVT::v4f64:
10409:     return Subtarget.hasExtLASX();
10410: 
10411:   default:
10412:     return false;
10413:   }
10414: }
10415: 
10416: SDValue LoongArchTargetLowering::getSqrtEstimate(SDValue Operand,
10417:                                                  SelectionDAG &DAG, int Enabled,
10418:                                                  int &RefinementSteps,
10419:                                                  bool &UseOneConstNR,
10420:                                                  bool Reciprocal) const {
10421:   assert(Enabled != ReciprocalEstimate::Disabled &&
10422:          "Enabled should never be Disabled here");
10423: 
10424:   if (!Subtarget.hasFrecipe())
10425:     return SDValue();
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::getExceptionPointerRegister`, `LoongArchTargetLowering::getExceptionSelectorRegister`, `getEstimateRefinementSteps`, `isSupportedReciprocalEstimateType`, `LoongArchTargetLowering::getSqrtEstimate`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::getExceptionPointerRegister`, `LoongArchTargetLowering::getExceptionSelectorRegister`, `getEstimateRefinementSteps`, `isSupportedReciprocalEstimateType`, `LoongArchTargetLowering::getSqrtEstimate` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10426-10500 / 第 10426-10500 行
```cpp
10426: 
10427:   SDLoc DL(Operand);
10428:   EVT VT = Operand.getValueType();
10429: 
10430:   // Check supported types.
10431:   if (!isSupportedReciprocalEstimateType(VT, Subtarget))
10432:     return SDValue();
10433: 
10434:   // Handle refinement steps.
10435:   if (RefinementSteps == ReciprocalEstimate::Unspecified)
10436:     RefinementSteps = getEstimateRefinementSteps(VT, Subtarget);
10437: 
10438:   // LoongArch only has FRSQRTE which is 1.0 / sqrt(x).
10439:   UseOneConstNR = false;
10440:   SDValue Rsqrt = DAG.getNode(LoongArchISD::FRSQRTE, DL, VT, Operand);
10441: 
10442:   // If the caller wants 1.0 / sqrt(x), or if further refinement steps
10443:   // are needed (which rely on the reciprocal form), return the raw reciprocal
10444:   // estimate.
10445:   if (Reciprocal || RefinementSteps > 0)
10446:     return Rsqrt;
10447: 
10448:   // Otherwise, return sqrt(x) by multiplying with the operand.
10449:   return DAG.getNode(ISD::FMUL, DL, VT, Operand, Rsqrt);
10450: }
10451: 
10452: SDValue LoongArchTargetLowering::getRecipEstimate(SDValue Operand,
10453:                                                   SelectionDAG &DAG,
10454:                                                   int Enabled,
10455:                                                   int &RefinementSteps) const {
10456:   assert(Enabled != ReciprocalEstimate::Disabled &&
10457:          "Enabled should never be Disabled here");
10458: 
10459:   if (!Subtarget.hasFrecipe())
10460:     return SDValue();
10461: 
10462:   SDLoc DL(Operand);
10463:   EVT VT = Operand.getValueType();
10464: 
10465:   // Check supported types.
10466:   if (!isSupportedReciprocalEstimateType(VT, Subtarget))
10467:     return SDValue();
10468: 
10469:   if (RefinementSteps == ReciprocalEstimate::Unspecified)
10470:     RefinementSteps = getEstimateRefinementSteps(VT, Subtarget);
10471: 
10472:   // FRECIPE computes 1.0 / x.
10473:   return DAG.getNode(LoongArchISD::FRECIPE, DL, VT, Operand);
10474: }
10475: 
10476: //===----------------------------------------------------------------------===//
10477: //                           LoongArch Inline Assembly Support
10478: //===----------------------------------------------------------------------===//
10479: 
10480: LoongArchTargetLowering::ConstraintType
10481: LoongArchTargetLowering::getConstraintType(StringRef Constraint) const {
10482:   // LoongArch specific constraints in GCC: config/loongarch/constraints.md
10483:   //
10484:   // 'f':  A floating-point register (if available).
10485:   // 'k':  A memory operand whose address is formed by a base register and
10486:   //       (optionally scaled) index register.
10487:   // 'l':  A signed 16-bit constant.
10488:   // 'm':  A memory operand whose address is formed by a base register and
10489:   //       offset that is suitable for use in instructions with the same
10490:   //       addressing mode as st.w and ld.w.
10491:   // 'q':  A general-purpose register except for $r0 and $r1 (for the csrxchg
10492:   //       instruction)
10493:   // 'I':  A signed 12-bit constant (for arithmetic instructions).
10494:   // 'J':  Integer zero.
10495:   // 'K':  An unsigned 12-bit constant (for logic instructions).
10496:   // "ZB": An address that is held in a general-purpose register. The offset is
10497:   //       zero.
10498:   // "ZC": A memory operand whose address is formed by a base register and
10499:   //       offset that is suitable for use in instructions with the same
10500:   //       addressing mode as ll.w and sc.w.
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::getRecipEstimate`, `LoongArchTargetLowering::getConstraintType`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::getRecipEstimate`, `LoongArchTargetLowering::getConstraintType` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 10501-10575 / 第 10501-10575 行
```cpp
10501:   if (Constraint.size() == 1) {
10502:     switch (Constraint[0]) {
10503:     default:
10504:       break;
10505:     case 'f':
10506:     case 'q':
10507:       return C_RegisterClass;
10508:     case 'l':
10509:     case 'I':
10510:     case 'J':
10511:     case 'K':
10512:       return C_Immediate;
10513:     case 'k':
10514:       return C_Memory;
10515:     }
10516:   }
10517: 
10518:   if (Constraint == "ZC" || Constraint == "ZB")
10519:     return C_Memory;
10520: 
10521:   // 'm' is handled here.
10522:   return TargetLowering::getConstraintType(Constraint);
10523: }
10524: 
10525: InlineAsm::ConstraintCode LoongArchTargetLowering::getInlineAsmMemConstraint(
10526:     StringRef ConstraintCode) const {
10527:   return StringSwitch<InlineAsm::ConstraintCode>(ConstraintCode)
10528:       .Case("k", InlineAsm::ConstraintCode::k)
10529:       .Case("ZB", InlineAsm::ConstraintCode::ZB)
10530:       .Case("ZC", InlineAsm::ConstraintCode::ZC)
10531:       .Default(TargetLowering::getInlineAsmMemConstraint(ConstraintCode));
10532: }
10533: 
10534: std::pair<unsigned, const TargetRegisterClass *>
10535: LoongArchTargetLowering::getRegForInlineAsmConstraint(
10536:     const TargetRegisterInfo *TRI, StringRef Constraint, MVT VT) const {
10537:   // First, see if this is a constraint that directly corresponds to a LoongArch
10538:   // register class.
10539:   if (Constraint.size() == 1) {
10540:     switch (Constraint[0]) {
10541:     case 'r':
10542:       // TODO: Support fixed vectors up to GRLen?
10543:       if (VT.isVector())
10544:         break;
10545:       return std::make_pair(0U, &LoongArch::GPRRegClass);
10546:     case 'q':
10547:       return std::make_pair(0U, &LoongArch::GPRNoR0R1RegClass);
10548:     case 'f':
10549:       if (Subtarget.hasBasicF() && VT == MVT::f32)
10550:         return std::make_pair(0U, &LoongArch::FPR32RegClass);
10551:       if (Subtarget.hasBasicD() && VT == MVT::f64)
10552:         return std::make_pair(0U, &LoongArch::FPR64RegClass);
10553:       if (Subtarget.hasExtLSX() &&
10554:           TRI->isTypeLegalForClass(LoongArch::LSX128RegClass, VT))
10555:         return std::make_pair(0U, &LoongArch::LSX128RegClass);
10556:       if (Subtarget.hasExtLASX() &&
10557:           TRI->isTypeLegalForClass(LoongArch::LASX256RegClass, VT))
10558:         return std::make_pair(0U, &LoongArch::LASX256RegClass);
10559:       break;
10560:     default:
10561:       break;
10562:     }
10563:   }
10564: 
10565:   // TargetLowering::getRegForInlineAsmConstraint uses the name of the TableGen
10566:   // record (e.g. the "R0" in `def R0`) to choose registers for InlineAsm
10567:   // constraints while the official register name is prefixed with a '$'. So we
10568:   // clip the '$' from the original constraint string (e.g. {$r0} to {r0}.)
10569:   // before it being parsed. And TargetLowering::getRegForInlineAsmConstraint is
10570:   // case insensitive, so no need to convert the constraint to upper case here.
10571:   //
10572:   // For now, no need to support ABI names (e.g. `$a0`) as clang will correctly
10573:   // decode the usage of register name aliases into their official names. And
10574:   // AFAIK, the not yet upstreamed `rustc` for LoongArch will always use
10575:   // official register names.
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::getInlineAsmMemConstraint`, `LoongArchTargetLowering::getRegForInlineAsmConstraint`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::getInlineAsmMemConstraint`, `LoongArchTargetLowering::getRegForInlineAsmConstraint` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10576-10650 / 第 10576-10650 行
```cpp
10576:   if (Constraint.starts_with("{$r") || Constraint.starts_with("{$f") ||
10577:       Constraint.starts_with("{$vr") || Constraint.starts_with("{$xr")) {
10578:     bool IsFP = Constraint[2] == 'f';
10579:     std::pair<StringRef, StringRef> Temp = Constraint.split('$');
10580:     std::pair<unsigned, const TargetRegisterClass *> R;
10581:     R = TargetLowering::getRegForInlineAsmConstraint(
10582:         TRI, join_items("", Temp.first, Temp.second), VT);
10583:     // Match those names to the widest floating point register type available.
10584:     if (IsFP) {
10585:       unsigned RegNo = R.first;
10586:       if (LoongArch::F0 <= RegNo && RegNo <= LoongArch::F31) {
10587:         if (Subtarget.hasBasicD() && (VT == MVT::f64 || VT == MVT::Other)) {
10588:           unsigned DReg = RegNo - LoongArch::F0 + LoongArch::F0_64;
10589:           return std::make_pair(DReg, &LoongArch::FPR64RegClass);
10590:         }
10591:       }
10592:     }
10593:     return R;
10594:   }
10595: 
10596:   return TargetLowering::getRegForInlineAsmConstraint(TRI, Constraint, VT);
10597: }
10598: 
10599: void LoongArchTargetLowering::LowerAsmOperandForConstraint(
10600:     SDValue Op, StringRef Constraint, std::vector<SDValue> &Ops,
10601:     SelectionDAG &DAG) const {
10602:   // Currently only support length 1 constraints.
10603:   if (Constraint.size() == 1) {
10604:     switch (Constraint[0]) {
10605:     case 'l':
10606:       // Validate & create a 16-bit signed immediate operand.
10607:       if (auto *C = dyn_cast<ConstantSDNode>(Op)) {
10608:         uint64_t CVal = C->getSExtValue();
10609:         if (isInt<16>(CVal))
10610:           Ops.push_back(DAG.getSignedTargetConstant(CVal, SDLoc(Op),
10611:                                                     Subtarget.getGRLenVT()));
10612:       }
10613:       return;
10614:     case 'I':
10615:       // Validate & create a 12-bit signed immediate operand.
10616:       if (auto *C = dyn_cast<ConstantSDNode>(Op)) {
10617:         uint64_t CVal = C->getSExtValue();
10618:         if (isInt<12>(CVal))
10619:           Ops.push_back(DAG.getSignedTargetConstant(CVal, SDLoc(Op),
10620:                                                     Subtarget.getGRLenVT()));
10621:       }
10622:       return;
10623:     case 'J':
10624:       // Validate & create an integer zero operand.
10625:       if (auto *C = dyn_cast<ConstantSDNode>(Op))
10626:         if (C->getZExtValue() == 0)
10627:           Ops.push_back(
10628:               DAG.getTargetConstant(0, SDLoc(Op), Subtarget.getGRLenVT()));
10629:       return;
10630:     case 'K':
10631:       // Validate & create a 12-bit unsigned immediate operand.
10632:       if (auto *C = dyn_cast<ConstantSDNode>(Op)) {
10633:         uint64_t CVal = C->getZExtValue();
10634:         if (isUInt<12>(CVal))
10635:           Ops.push_back(
10636:               DAG.getTargetConstant(CVal, SDLoc(Op), Subtarget.getGRLenVT()));
10637:       }
10638:       return;
10639:     default:
10640:       break;
10641:     }
10642:   }
10643:   TargetLowering::LowerAsmOperandForConstraint(Op, Constraint, Ops, DAG);
10644: }
10645: 
10646: #define GET_REGISTER_MATCHER
10647: #include "LoongArchGenAsmMatcher.inc"
10648: 
10649: Register
10650: LoongArchTargetLowering::getRegisterByName(const char *RegName, LLT VT,
```
- **EN**: It imports dependencies such as `LoongArchGenAsmMatcher.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `LoongArchTargetLowering::LowerAsmOperandForConstraint`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories.
- **CN**: 它引入了 `LoongArchGenAsmMatcher.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `LoongArchTargetLowering::LowerAsmOperandForConstraint` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。

### Lines 10651-10725 / 第 10651-10725 行
```cpp
10651:                                            const MachineFunction &MF) const {
10652:   std::pair<StringRef, StringRef> Name = StringRef(RegName).split('$');
10653:   std::string NewRegName = Name.second.str();
10654:   Register Reg = MatchRegisterAltName(NewRegName);
10655:   if (!Reg)
10656:     Reg = MatchRegisterName(NewRegName);
10657:   if (!Reg)
10658:     return Reg;
10659:   BitVector ReservedRegs = Subtarget.getRegisterInfo()->getReservedRegs(MF);
10660:   if (!ReservedRegs.test(Reg))
10661:     report_fatal_error(Twine("Trying to obtain non-reserved register \"" +
10662:                              StringRef(RegName) + "\"."));
10663:   return Reg;
10664: }
10665: 
10666: bool LoongArchTargetLowering::decomposeMulByConstant(LLVMContext &Context,
10667:                                                      EVT VT, SDValue C) const {
10668:   // TODO: Support vectors.
10669:   if (!VT.isScalarInteger())
10670:     return false;
10671: 
10672:   // Omit the optimization if the data size exceeds GRLen.
10673:   if (VT.getSizeInBits() > Subtarget.getGRLen())
10674:     return false;
10675: 
10676:   if (auto *ConstNode = dyn_cast<ConstantSDNode>(C.getNode())) {
10677:     const APInt &Imm = ConstNode->getAPIntValue();
10678:     // Break MUL into (SLLI + ADD/SUB) or ALSL.
10679:     if ((Imm + 1).isPowerOf2() || (Imm - 1).isPowerOf2() ||
10680:         (1 - Imm).isPowerOf2() || (-1 - Imm).isPowerOf2())
10681:       return true;
10682:     // Break MUL into (ALSL x, (SLLI x, imm0), imm1).
10683:     if (ConstNode->hasOneUse() &&
10684:         ((Imm - 2).isPowerOf2() || (Imm - 4).isPowerOf2() ||
10685:          (Imm - 8).isPowerOf2() || (Imm - 16).isPowerOf2()))
10686:       return true;
10687:     // Break (MUL x, imm) into (ADD (SLLI x, s0), (SLLI x, s1)),
10688:     // in which the immediate has two set bits. Or Break (MUL x, imm)
10689:     // into (SUB (SLLI x, s0), (SLLI x, s1)), in which the immediate
10690:     // equals to (1 << s0) - (1 << s1).
10691:     if (ConstNode->hasOneUse() && !(Imm.sge(-2048) && Imm.sle(4095))) {
10692:       unsigned Shifts = Imm.countr_zero();
10693:       // Reject immediates which can be composed via a single LUI.
10694:       if (Shifts >= 12)
10695:         return false;
10696:       // Reject multiplications can be optimized to
10697:       // (SLLI (ALSL x, x, 1/2/3/4), s).
10698:       APInt ImmPop = Imm.ashr(Shifts);
10699:       if (ImmPop == 3 || ImmPop == 5 || ImmPop == 9 || ImmPop == 17)
10700:         return false;
10701:       // We do not consider the case `(-Imm - ImmSmall).isPowerOf2()`,
10702:       // since it needs one more instruction than other 3 cases.
10703:       APInt ImmSmall = APInt(Imm.getBitWidth(), 1ULL << Shifts, true);
10704:       if ((Imm - ImmSmall).isPowerOf2() || (Imm + ImmSmall).isPowerOf2() ||
10705:           (ImmSmall - Imm).isPowerOf2())
10706:         return true;
10707:     }
10708:   }
10709: 
10710:   return false;
10711: }
10712: 
10713: bool LoongArchTargetLowering::isLegalAddressingMode(const DataLayout &DL,
10714:                                                     const AddrMode &AM,
10715:                                                     Type *Ty, unsigned AS,
10716:                                                     Instruction *I) const {
10717:   // LoongArch has four basic addressing modes:
10718:   //  1. reg
10719:   //  2. reg + 12-bit signed offset
10720:   //  3. reg + 14-bit signed offset left-shifted by 2
10721:   //  4. reg1 + reg2
10722:   // TODO: Add more checks after support vector extension.
10723: 
10724:   // No global is ever allowed as a base.
10725:   if (AM.BaseGV)
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::decomposeMulByConstant`, `LoongArchTargetLowering::isLegalAddressingMode`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::decomposeMulByConstant`, `LoongArchTargetLowering::isLegalAddressingMode` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10726-10800 / 第 10726-10800 行
```cpp
10726:     return false;
10727: 
10728:   // Require a 12-bit signed offset or 14-bit signed offset left-shifted by 2
10729:   // with `UAL` feature.
10730:   if (!isInt<12>(AM.BaseOffs) &&
10731:       !(isShiftedInt<14, 2>(AM.BaseOffs) && Subtarget.hasUAL()))
10732:     return false;
10733: 
10734:   switch (AM.Scale) {
10735:   case 0:
10736:     // "r+i" or just "i", depending on HasBaseReg.
10737:     break;
10738:   case 1:
10739:     // "r+r+i" is not allowed.
10740:     if (AM.HasBaseReg && AM.BaseOffs)
10741:       return false;
10742:     // Otherwise we have "r+r" or "r+i".
10743:     break;
10744:   case 2:
10745:     // "2*r+r" or "2*r+i" is not allowed.
10746:     if (AM.HasBaseReg || AM.BaseOffs)
10747:       return false;
10748:     // Allow "2*r" as "r+r".
10749:     break;
10750:   default:
10751:     return false;
10752:   }
10753: 
10754:   return true;
10755: }
10756: 
10757: bool LoongArchTargetLowering::isLegalICmpImmediate(int64_t Imm) const {
10758:   return isInt<12>(Imm);
10759: }
10760: 
10761: bool LoongArchTargetLowering::isLegalAddImmediate(int64_t Imm) const {
10762:   return isInt<12>(Imm);
10763: }
10764: 
10765: bool LoongArchTargetLowering::isZExtFree(SDValue Val, EVT VT2) const {
10766:   // Zexts are free if they can be combined with a load.
10767:   // Don't advertise i32->i64 zextload as being free for LA64. It interacts
10768:   // poorly with type legalization of compares preferring sext.
10769:   if (auto *LD = dyn_cast<LoadSDNode>(Val)) {
10770:     EVT MemVT = LD->getMemoryVT();
10771:     if ((MemVT == MVT::i8 || MemVT == MVT::i16) &&
10772:         (LD->getExtensionType() == ISD::NON_EXTLOAD ||
10773:          LD->getExtensionType() == ISD::ZEXTLOAD))
10774:       return true;
10775:   }
10776: 
10777:   return TargetLowering::isZExtFree(Val, VT2);
10778: }
10779: 
10780: bool LoongArchTargetLowering::isSExtCheaperThanZExt(EVT SrcVT,
10781:                                                     EVT DstVT) const {
10782:   return Subtarget.is64Bit() && SrcVT == MVT::i32 && DstVT == MVT::i64;
10783: }
10784: 
10785: bool LoongArchTargetLowering::signExtendConstant(const ConstantInt *CI) const {
10786:   return Subtarget.is64Bit() && CI->getType()->isIntegerTy(32);
10787: }
10788: 
10789: bool LoongArchTargetLowering::hasAndNotCompare(SDValue Y) const {
10790:   // TODO: Support vectors.
10791:   if (Y.getValueType().isVector())
10792:     return false;
10793: 
10794:   return !isa<ConstantSDNode>(Y);
10795: }
10796: 
10797: ISD::NodeType LoongArchTargetLowering::getExtendForAtomicCmpSwapArg() const {
10798:   // LAMCAS will use amcas[_DB].{b/h/w/d} which does not require extension.
10799:   return Subtarget.hasLAMCAS() ? ISD::ANY_EXTEND : ISD::SIGN_EXTEND;
10800: }
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::isLegalICmpImmediate`, `LoongArchTargetLowering::isLegalAddImmediate`, `LoongArchTargetLowering::isZExtFree`, `LoongArchTargetLowering::isSExtCheaperThanZExt`, `LoongArchTargetLowering::signExtendConstant`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::isLegalICmpImmediate`, `LoongArchTargetLowering::isLegalAddImmediate`, `LoongArchTargetLowering::isZExtFree`, `LoongArchTargetLowering::isSExtCheaperThanZExt`, `LoongArchTargetLowering::signExtendConstant` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10801-10875 / 第 10801-10875 行
```cpp
10801: 
10802: bool LoongArchTargetLowering::shouldSignExtendTypeInLibCall(
10803:     Type *Ty, bool IsSigned) const {
10804:   if (Subtarget.is64Bit() && Ty->isIntegerTy(32))
10805:     return true;
10806: 
10807:   return IsSigned;
10808: }
10809: 
10810: bool LoongArchTargetLowering::shouldExtendTypeInLibCall(EVT Type) const {
10811:   // Return false to suppress the unnecessary extensions if the LibCall
10812:   // arguments or return value is a float narrower than GRLEN on a soft FP ABI.
10813:   if (Subtarget.isSoftFPABI() && (Type.isFloatingPoint() && !Type.isVector() &&
10814:                                   Type.getSizeInBits() < Subtarget.getGRLen()))
10815:     return false;
10816:   return true;
10817: }
10818: 
10819: // memcpy, and other memory intrinsics, typically tries to use wider load/store
10820: // if the source/dest is aligned and the copy size is large enough. We therefore
10821: // want to align such objects passed to memory intrinsics.
10822: bool LoongArchTargetLowering::shouldAlignPointerArgs(CallInst *CI,
10823:                                                      unsigned &MinSize,
10824:                                                      Align &PrefAlign) const {
10825:   if (!isa<MemIntrinsic>(CI))
10826:     return false;
10827: 
10828:   if (Subtarget.is64Bit()) {
10829:     MinSize = 8;
10830:     PrefAlign = Align(8);
10831:   } else {
10832:     MinSize = 4;
10833:     PrefAlign = Align(4);
10834:   }
10835: 
10836:   return true;
10837: }
10838: 
10839: TargetLoweringBase::LegalizeTypeAction
10840: LoongArchTargetLowering::getPreferredVectorAction(MVT VT) const {
10841:   if (!VT.isScalableVector() && VT.getVectorNumElements() != 1 &&
10842:       VT.getVectorElementType() != MVT::i1)
10843:     return TypeWidenVector;
10844: 
10845:   return TargetLoweringBase::getPreferredVectorAction(VT);
10846: }
10847: 
10848: bool LoongArchTargetLowering::splitValueIntoRegisterParts(
10849:     SelectionDAG &DAG, const SDLoc &DL, SDValue Val, SDValue *Parts,
10850:     unsigned NumParts, MVT PartVT, std::optional<CallingConv::ID> CC) const {
10851:   bool IsABIRegCopy = CC.has_value();
10852:   EVT ValueVT = Val.getValueType();
10853: 
10854:   if (IsABIRegCopy && (ValueVT == MVT::f16 || ValueVT == MVT::bf16) &&
10855:       PartVT == MVT::f32) {
10856:     // Cast the [b]f16 to i16, extend to i32, pad with ones to make a float
10857:     // nan, and cast to f32.
10858:     Val = DAG.getNode(ISD::BITCAST, DL, MVT::i16, Val);
10859:     Val = DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i32, Val);
10860:     Val = DAG.getNode(ISD::OR, DL, MVT::i32, Val,
10861:                       DAG.getConstant(0xFFFF0000, DL, MVT::i32));
10862:     Val = DAG.getNode(ISD::BITCAST, DL, MVT::f32, Val);
10863:     Parts[0] = Val;
10864:     return true;
10865:   }
10866: 
10867:   return false;
10868: }
10869: 
10870: SDValue LoongArchTargetLowering::joinRegisterPartsIntoValue(
10871:     SelectionDAG &DAG, const SDLoc &DL, const SDValue *Parts, unsigned NumParts,
10872:     MVT PartVT, EVT ValueVT, std::optional<CallingConv::ID> CC) const {
10873:   bool IsABIRegCopy = CC.has_value();
10874: 
10875:   if (IsABIRegCopy && (ValueVT == MVT::f16 || ValueVT == MVT::bf16) &&
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::shouldSignExtendTypeInLibCall`, `LoongArchTargetLowering::shouldExtendTypeInLibCall`, `LoongArchTargetLowering::shouldAlignPointerArgs`, `LoongArchTargetLowering::getPreferredVectorAction`, `LoongArchTargetLowering::splitValueIntoRegisterParts`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::shouldSignExtendTypeInLibCall`, `LoongArchTargetLowering::shouldExtendTypeInLibCall`, `LoongArchTargetLowering::shouldAlignPointerArgs`, `LoongArchTargetLowering::getPreferredVectorAction`, `LoongArchTargetLowering::splitValueIntoRegisterParts` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10876-10950 / 第 10876-10950 行
```cpp
10876:       PartVT == MVT::f32) {
10877:     SDValue Val = Parts[0];
10878: 
10879:     // Cast the f32 to i32, truncate to i16, and cast back to [b]f16.
10880:     Val = DAG.getNode(ISD::BITCAST, DL, MVT::i32, Val);
10881:     Val = DAG.getNode(ISD::TRUNCATE, DL, MVT::i16, Val);
10882:     Val = DAG.getNode(ISD::BITCAST, DL, ValueVT, Val);
10883:     return Val;
10884:   }
10885: 
10886:   return SDValue();
10887: }
10888: 
10889: MVT LoongArchTargetLowering::getRegisterTypeForCallingConv(LLVMContext &Context,
10890:                                                            CallingConv::ID CC,
10891:                                                            EVT VT) const {
10892:   // Use f32 to pass f16.
10893:   if (VT == MVT::f16 && Subtarget.hasBasicF())
10894:     return MVT::f32;
10895: 
10896:   return TargetLowering::getRegisterTypeForCallingConv(Context, CC, VT);
10897: }
10898: 
10899: unsigned LoongArchTargetLowering::getNumRegistersForCallingConv(
10900:     LLVMContext &Context, CallingConv::ID CC, EVT VT) const {
10901:   // Use f32 to pass f16.
10902:   if (VT == MVT::f16 && Subtarget.hasBasicF())
10903:     return 1;
10904: 
10905:   return TargetLowering::getNumRegistersForCallingConv(Context, CC, VT);
10906: }
10907: 
10908: void LoongArchTargetLowering::computeKnownBitsForTargetNode(
10909:     const SDValue Op, KnownBits &Known, const APInt &DemandedElts,
10910:     const SelectionDAG &DAG, unsigned Depth) const {
10911:   unsigned Opc = Op.getOpcode();
10912:   Known.resetAll();
10913:   switch (Opc) {
10914:   default:
10915:     break;
10916:   case LoongArchISD::VPICK_ZEXT_ELT: {
10917:     assert(isa<VTSDNode>(Op->getOperand(2)) && "Unexpected operand!");
10918:     EVT VT = cast<VTSDNode>(Op->getOperand(2))->getVT();
10919:     unsigned VTBits = VT.getScalarSizeInBits();
10920:     assert(Known.getBitWidth() >= VTBits && "Unexpected width!");
10921:     Known.Zero.setBitsFrom(VTBits);
10922:     break;
10923:   }
10924:   }
10925: }
10926: 
10927: bool LoongArchTargetLowering::SimplifyDemandedBitsForTargetNode(
10928:     SDValue Op, const APInt &OriginalDemandedBits,
10929:     const APInt &OriginalDemandedElts, KnownBits &Known, TargetLoweringOpt &TLO,
10930:     unsigned Depth) const {
10931:   EVT VT = Op.getValueType();
10932:   unsigned BitWidth = OriginalDemandedBits.getBitWidth();
10933:   unsigned Opc = Op.getOpcode();
10934:   switch (Opc) {
10935:   default:
10936:     break;
10937:   case LoongArchISD::VMSKLTZ:
10938:   case LoongArchISD::XVMSKLTZ: {
10939:     SDValue Src = Op.getOperand(0);
10940:     MVT SrcVT = Src.getSimpleValueType();
10941:     unsigned SrcBits = SrcVT.getScalarSizeInBits();
10942:     unsigned NumElts = SrcVT.getVectorNumElements();
10943: 
10944:     // If we don't need the sign bits at all just return zero.
10945:     if (OriginalDemandedBits.countr_zero() >= NumElts)
10946:       return TLO.CombineTo(Op, TLO.DAG.getConstant(0, SDLoc(Op), VT));
10947: 
10948:     // Only demand the vector elements of the sign bits we need.
10949:     APInt KnownUndef, KnownZero;
10950:     APInt DemandedElts = OriginalDemandedBits.zextOrTrunc(NumElts);
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::getRegisterTypeForCallingConv`, `LoongArchTargetLowering::getNumRegistersForCallingConv`, `LoongArchTargetLowering::computeKnownBitsForTargetNode`, `LoongArchTargetLowering::SimplifyDemandedBitsForTargetNode`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::getRegisterTypeForCallingConv`, `LoongArchTargetLowering::getNumRegistersForCallingConv`, `LoongArchTargetLowering::computeKnownBitsForTargetNode`, `LoongArchTargetLowering::SimplifyDemandedBitsForTargetNode` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 10951-11016 / 第 10951-11016 行
```cpp
10951:     if (SimplifyDemandedVectorElts(Src, DemandedElts, KnownUndef, KnownZero,
10952:                                    TLO, Depth + 1))
10953:       return true;
10954: 
10955:     Known.Zero = KnownZero.zext(BitWidth);
10956:     Known.Zero.setHighBits(BitWidth - NumElts);
10957: 
10958:     // [X]VMSKLTZ only uses the MSB from each vector element.
10959:     KnownBits KnownSrc;
10960:     APInt DemandedSrcBits = APInt::getSignMask(SrcBits);
10961:     if (SimplifyDemandedBits(Src, DemandedSrcBits, DemandedElts, KnownSrc, TLO,
10962:                              Depth + 1))
10963:       return true;
10964: 
10965:     if (KnownSrc.One[SrcBits - 1])
10966:       Known.One.setLowBits(NumElts);
10967:     else if (KnownSrc.Zero[SrcBits - 1])
10968:       Known.Zero.setLowBits(NumElts);
10969: 
10970:     // Attempt to avoid multi-use ops if we don't need anything from it.
10971:     if (SDValue NewSrc = SimplifyMultipleUseDemandedBits(
10972:             Src, DemandedSrcBits, DemandedElts, TLO.DAG, Depth + 1))
10973:       return TLO.CombineTo(Op, TLO.DAG.getNode(Opc, SDLoc(Op), VT, NewSrc));
10974:     return false;
10975:   }
10976:   }
10977: 
10978:   return TargetLowering::SimplifyDemandedBitsForTargetNode(
10979:       Op, OriginalDemandedBits, OriginalDemandedElts, Known, TLO, Depth);
10980: }
10981: 
10982: bool LoongArchTargetLowering::shouldScalarizeBinop(SDValue VecOp) const {
10983:   unsigned Opc = VecOp.getOpcode();
10984: 
10985:   // Assume target opcodes can't be scalarized.
10986:   // TODO - do we have any exceptions?
10987:   if (Opc >= ISD::BUILTIN_OP_END || !isBinOp(Opc))
10988:     return false;
10989: 
10990:   // If the vector op is not supported, try to convert to scalar.
10991:   EVT VecVT = VecOp.getValueType();
10992:   if (!isOperationLegalOrCustomOrPromote(Opc, VecVT))
10993:     return true;
10994: 
10995:   // If the vector op is supported, but the scalar op is not, the transform may
10996:   // not be worthwhile.
10997:   EVT ScalarVT = VecVT.getScalarType();
10998:   return isOperationLegalOrCustomOrPromote(Opc, ScalarVT);
10999: }
11000: 
11001: bool LoongArchTargetLowering::isExtractSubvectorCheap(EVT ResVT, EVT SrcVT,
11002:                                                       unsigned Index) const {
11003:   if (!isOperationLegalOrCustom(ISD::EXTRACT_SUBVECTOR, ResVT))
11004:     return false;
11005: 
11006:   // Extract a 128-bit subvector from index 0 of a 256-bit vector is free.
11007:   return Index == 0;
11008: }
11009: 
11010: bool LoongArchTargetLowering::isExtractVecEltCheap(EVT VT,
11011:                                                    unsigned Index) const {
11012:   EVT EltVT = VT.getScalarType();
11013: 
11014:   // Extract a scalar FP value from index 0 of a vector is free.
11015:   return (EltVT == MVT::f32 || EltVT == MVT::f64) && Index == 0;
11016: }
```
- **EN**: The range implements or declares functions including `LoongArchTargetLowering::shouldScalarizeBinop`, `LoongArchTargetLowering::isExtractSubvectorCheap`, `LoongArchTargetLowering::isExtractVecEltCheap`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `LoongArchTargetLowering::shouldScalarizeBinop`, `LoongArchTargetLowering::isExtractSubvectorCheap`, `LoongArchTargetLowering::isExtractVecEltCheap` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **TableGen DSL**: Describes backend metadata with declarative records and multiclasses. / 使用声明式记录和 multiclass 描述后端元数据。
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **GlobalISel**: Uses legalization, register banks, and instruction selection after generic IR lowering. / 在通用 IR 降低后使用合法化、寄存器银行和指令选择。
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。

## Dependencies / 依赖关系
- `LoongArchISelLowering.h`
- `LoongArch.h`
- `LoongArchMachineFunctionInfo.h`
- `LoongArchRegisterInfo.h`
- `LoongArchSelectionDAGInfo.h`
- `LoongArchSubtarget.h`
- `MCTargetDesc/LoongArchBaseInfo.h`
- `MCTargetDesc/LoongArchMCTargetDesc.h`
- `MCTargetDesc/LoongArchMatInt.h`
- `llvm/ADT/SmallSet.h`
- `llvm/ADT/Statistic.h`
- `llvm/ADT/StringExtras.h`
- `llvm/CodeGen/ISDOpcodes.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/RuntimeLibcallUtil.h`
- `llvm/CodeGen/SelectionDAGNodes.h`
- `...` (10 more include dependencies omitted for brevity / 其余 10 个 include 依赖已省略)
