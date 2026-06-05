# HexagonISelLoweringHVX.cpp — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `llvm/lib/Target/Hexagon/HexagonISelLoweringHVX.cpp`
- Repository / 仓库: `/root/xw/llvm-project/`
- Purpose / 作用 (EN): Implements Hexagon HVX-specific lowering from LLVM IR into SelectionDAG operations.
- Purpose / 作用 (CN): 该文件实现 Hexagon 后端组件的核心逻辑。 重点涉及 HVX/向量处理。

## Line-by-Line Analysis / 逐行分析

### Lines 1-200 / 第 1-200 行

```cpp
     1: //===-- HexagonISelLoweringHVX.cpp --- Lowering HVX operations ------------===//
     2: //
     3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
     4: // See https://llvm.org/LICENSE.txt for license information.
     5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
     6: //
     7: //===----------------------------------------------------------------------===//
     8: 
     9: #include "HexagonISelLowering.h"
    10: #include "HexagonRegisterInfo.h"
    11: #include "HexagonSubtarget.h"
    12: #include "llvm/ADT/SetVector.h"
    13: #include "llvm/ADT/SmallVector.h"
    14: #include "llvm/Analysis/MemoryLocation.h"
    15: #include "llvm/CodeGen/MachineBasicBlock.h"
    16: #include "llvm/CodeGen/MachineFunction.h"
    17: #include "llvm/CodeGen/MachineInstr.h"
    18: #include "llvm/CodeGen/MachineOperand.h"
    19: #include "llvm/CodeGen/MachineRegisterInfo.h"
    20: #include "llvm/CodeGen/TargetInstrInfo.h"
    21: #include "llvm/IR/IntrinsicsHexagon.h"
    22: #include "llvm/Support/CommandLine.h"
    23: 
    24: #include <algorithm>
    25: #include <string>
    26: #include <utility>
    27: 
    28: using namespace llvm;
    29: 
    30: static cl::opt<unsigned> HvxWidenThreshold("hexagon-hvx-widen",
    31:   cl::Hidden, cl::init(16),
    32:   cl::desc("Lower threshold (in bytes) for widening to HVX vectors"));
    33: 
    34: static cl::opt<bool>
    35:     EnableFpFastConvert("hexagon-fp-fast-convert", cl::Hidden, cl::init(false),
    36:                         cl::desc("Enable FP fast conversion routine."));
    37: 
    38: static const MVT LegalV64[] =  { MVT::v64i8,  MVT::v32i16,  MVT::v16i32 };
    39: static const MVT LegalW64[] =  { MVT::v128i8, MVT::v64i16,  MVT::v32i32 };
    40: static const MVT LegalV128[] = { MVT::v128i8, MVT::v64i16,  MVT::v32i32 };
    41: static const MVT LegalW128[] = { MVT::v256i8, MVT::v128i16, MVT::v64i32 };
    42: 
    43: static const unsigned MaxExpandMLA = 8;
    44: 
    45: static std::tuple<unsigned, unsigned, unsigned> getIEEEProperties(MVT Ty) {
    46:   // For a float scalar type, return (exp-bits, exp-bias, fraction-bits)
    47:   MVT ElemTy = Ty.getScalarType();
    48:   switch (ElemTy.SimpleTy) {
    49:     case MVT::f16:
    50:       return std::make_tuple(5, 15, 10);
    51:     case MVT::f32:
    52:       return std::make_tuple(8, 127, 23);
    53:     case MVT::f64:
    54:       return std::make_tuple(11, 1023, 52);
    55:     default:
    56:       break;
    57:   }
    58:   llvm_unreachable(("Unexpected type: " + EVT(ElemTy).getEVTString()).c_str());
    59: }
    60: 
    61: void
    62: HexagonTargetLowering::initializeHVXLowering() {
    63:   if (Subtarget.useHVX64BOps()) {
    64:     addRegisterClass(MVT::v64i8,  &Hexagon::HvxVRRegClass);
    65:     addRegisterClass(MVT::v32i16, &Hexagon::HvxVRRegClass);
    66:     addRegisterClass(MVT::v16i32, &Hexagon::HvxVRRegClass);
    67:     addRegisterClass(MVT::v128i8, &Hexagon::HvxWRRegClass);
    68:     addRegisterClass(MVT::v64i16, &Hexagon::HvxWRRegClass);
    69:     addRegisterClass(MVT::v32i32, &Hexagon::HvxWRRegClass);
    70:     // These "short" boolean vector types should be legal because
    71:     // they will appear as results of vector compares. If they were
    72:     // not legal, type legalization would try to make them legal
    73:     // and that would require using operations that do not use or
    74:     // produce such types. That, in turn, would imply using custom
    75:     // nodes, which would be unoptimizable by the DAG combiner.
    76:     // The idea is to rely on target-independent operations as much
    77:     // as possible.
    78:     addRegisterClass(MVT::v16i1, &Hexagon::HvxQRRegClass);
    79:     addRegisterClass(MVT::v32i1, &Hexagon::HvxQRRegClass);
    80:     addRegisterClass(MVT::v64i1, &Hexagon::HvxQRRegClass);
    81:   } else if (Subtarget.useHVX128BOps()) {
    82:     addRegisterClass(MVT::v128i8,  &Hexagon::HvxVRRegClass);
    83:     addRegisterClass(MVT::v64i16,  &Hexagon::HvxVRRegClass);
    84:     addRegisterClass(MVT::v32i32,  &Hexagon::HvxVRRegClass);
    85:     addRegisterClass(MVT::v256i8,  &Hexagon::HvxWRRegClass);
    86:     addRegisterClass(MVT::v128i16, &Hexagon::HvxWRRegClass);
    87:     addRegisterClass(MVT::v64i32,  &Hexagon::HvxWRRegClass);
    88:     addRegisterClass(MVT::v32i1, &Hexagon::HvxQRRegClass);
    89:     addRegisterClass(MVT::v64i1, &Hexagon::HvxQRRegClass);
    90:     addRegisterClass(MVT::v128i1, &Hexagon::HvxQRRegClass);
    91:     if (Subtarget.useHVXV68Ops() && Subtarget.useHVXFloatingPoint()) {
    92:       addRegisterClass(MVT::v32f32, &Hexagon::HvxVRRegClass);
    93:       addRegisterClass(MVT::v64f16, &Hexagon::HvxVRRegClass);
    94:       addRegisterClass(MVT::v64f32, &Hexagon::HvxWRRegClass);
    95:       addRegisterClass(MVT::v128f16, &Hexagon::HvxWRRegClass);
    96:     }
    97:     if (Subtarget.useHVXV81Ops()) {
    98:       addRegisterClass(MVT::v64bf16, &Hexagon::HvxVRRegClass);
    99:       addRegisterClass(MVT::v128bf16, &Hexagon::HvxWRRegClass);
   100:     }
   101:   }
   102: 
   103:   // Set up operation actions.
   104: 
   105:   bool Use64b = Subtarget.useHVX64BOps();
   106:   ArrayRef<MVT> LegalV = Use64b ? LegalV64 : LegalV128;
   107:   ArrayRef<MVT> LegalW = Use64b ? LegalW64 : LegalW128;
   108:   MVT ByteV = Use64b ?  MVT::v64i8 : MVT::v128i8;
   109:   MVT WordV = Use64b ? MVT::v16i32 : MVT::v32i32;
   110:   MVT ByteW = Use64b ? MVT::v128i8 : MVT::v256i8;
   111: 
   112:   auto setPromoteTo = [this] (unsigned Opc, MVT FromTy, MVT ToTy) {
   113:     setOperationAction(Opc, FromTy, Promote);
   114:     AddPromotedToType(Opc, FromTy, ToTy);
   115:   };
   116: 
   117:   // Handle bitcasts of vector predicates to scalars (e.g. v32i1 to i32).
   118:   // Note: v16i1 -> i16 is handled in type legalization instead of op
   119:   // legalization.
   120:   setOperationAction(ISD::BITCAST,              MVT::i16, Custom);
   121:   setOperationAction(ISD::BITCAST,              MVT::i32, Custom);
   122:   setOperationAction(ISD::BITCAST,              MVT::i64, Custom);
   123:   setOperationAction(ISD::BITCAST,            MVT::v16i1, Custom);
   124:   setOperationAction(ISD::BITCAST,           MVT::v128i1, Custom);
   125:   setOperationAction(ISD::BITCAST,             MVT::i128, Custom);
   126:   setOperationAction(ISD::VECTOR_SHUFFLE,          ByteV, Legal);
   127:   setOperationAction(ISD::VECTOR_SHUFFLE,          ByteW, Legal);
   128:   setOperationAction(ISD::INTRINSIC_WO_CHAIN, MVT::Other, Custom);
   129: 
   130:   if (Subtarget.useHVX128BOps()) {
   131:     setOperationAction(ISD::BITCAST, MVT::v32i1, Custom);
   132:     setOperationAction(ISD::BITCAST, MVT::v64i1, Custom);
   133:     setOperationAction(ISD::STORE, MVT::v32i1, Custom);
   134:     setOperationAction(ISD::LOAD, MVT::v32i1, Custom);
   135:     setOperationAction(ISD::STORE, MVT::v64i1, Custom);
   136:     setOperationAction(ISD::LOAD, MVT::v64i1, Custom);
   137:     setOperationAction(ISD::STORE, MVT::v128i1, Custom);
   138:     setOperationAction(ISD::LOAD, MVT::v128i1, Custom);
   139:   }
   140:   if (Subtarget.useHVX128BOps() && Subtarget.useHVXV68Ops() &&
   141:       Subtarget.useHVXFloatingPoint()) {
   142: 
   143:     static const MVT FloatV[] = { MVT::v64f16, MVT::v32f32 };
   144:     static const MVT FloatW[] = { MVT::v128f16, MVT::v64f32 };
   145: 
   146:     for (MVT T : FloatV) {
   147:       setOperationAction(ISD::FADD,              T, Legal);
   148:       setOperationAction(ISD::FSUB,              T, Legal);
   149:       setOperationAction(ISD::FMUL,              T, Legal);
   150:       setOperationAction(ISD::FMINIMUMNUM, T, Legal);
   151:       setOperationAction(ISD::FMAXIMUMNUM, T, Legal);
   152: 
   153:       setOperationAction(ISD::INSERT_SUBVECTOR,  T, Custom);
   154:       setOperationAction(ISD::EXTRACT_SUBVECTOR, T, Custom);
   155: 
   156:       setOperationAction(ISD::SPLAT_VECTOR,      T, Legal);
   157:       setOperationAction(ISD::SPLAT_VECTOR,      T, Legal);
   158: 
   159:       setOperationAction(ISD::MLOAD,             T, Custom);
   160:       setOperationAction(ISD::MSTORE,            T, Custom);
   161:       // Custom-lower BUILD_VECTOR. The standard (target-independent)
   162:       // handling of it would convert it to a load, which is not always
   163:       // the optimal choice.
   164:       setOperationAction(ISD::BUILD_VECTOR,      T, Custom);
   165:     }
   166: 
   167: 
   168:     // BUILD_VECTOR with f16 operands cannot be promoted without
   169:     // promoting the result, so lower the node to vsplat or constant pool
   170:     setOperationAction(ISD::BUILD_VECTOR,      MVT::f16, Custom);
   171:     setOperationAction(ISD::INSERT_VECTOR_ELT, MVT::f16, Custom);
   172:     setOperationAction(ISD::SPLAT_VECTOR,      MVT::f16, Custom);
   173: 
   174:     // Vector shuffle is always promoted to ByteV and a bitcast to f16 is
   175:     // generated.
   176:     setPromoteTo(ISD::VECTOR_SHUFFLE, MVT::v128f16, ByteW);
   177:     setPromoteTo(ISD::VECTOR_SHUFFLE,  MVT::v64f16, ByteV);
   178:     setPromoteTo(ISD::VECTOR_SHUFFLE,  MVT::v64f32, ByteW);
   179:     setPromoteTo(ISD::VECTOR_SHUFFLE,  MVT::v32f32, ByteV);
   180: 
   181:     if (Subtarget.useHVXV81Ops()) {
   182:       setPromoteTo(ISD::VECTOR_SHUFFLE, MVT::v128bf16, ByteW);
   183:       setPromoteTo(ISD::VECTOR_SHUFFLE, MVT::v64bf16, ByteV);
   184:       setPromoteTo(ISD::SETCC, MVT::v64bf16, MVT::v64f32);
   185:       setPromoteTo(ISD::FADD, MVT::v64bf16, MVT::v64f32);
   186:       setPromoteTo(ISD::FSUB, MVT::v64bf16, MVT::v64f32);
   187:       setPromoteTo(ISD::FMUL, MVT::v64bf16, MVT::v64f32);
   188:       setPromoteTo(ISD::FMINNUM, MVT::v64bf16, MVT::v64f32);
   189:       setPromoteTo(ISD::FMAXNUM, MVT::v64bf16, MVT::v64f32);
   190: 
   191:       setOperationAction(ISD::SPLAT_VECTOR, MVT::v64bf16, Legal);
   192:       setOperationAction(ISD::INSERT_SUBVECTOR, MVT::v64bf16, Custom);
   193:       setOperationAction(ISD::EXTRACT_SUBVECTOR, MVT::v64bf16, Custom);
   194: 
   195:       setOperationAction(ISD::LOAD, MVT::v128bf16, Custom);
   196:       setOperationAction(ISD::STORE, MVT::v128bf16, Custom);
   197: 
   198:       setOperationAction(ISD::MLOAD, MVT::v64bf16, Custom);
   199:       setOperationAction(ISD::MSTORE, MVT::v64bf16, Custom);
   200:       setOperationAction(ISD::BUILD_VECTOR, MVT::v64bf16, Custom);
```
- EN: It imports headers such as HexagonISelLowering.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/SetVector.h, ... (17 total), establishing the LLVM/Hexagon APIs used below. It opens namespaces (llvm) to keep the implementation scoped to LLVM/Hexagon components. It declares or implements routines such as HvxWidenThreshold, EnableFpFastConvert, getIEEEProperties, std::make_tuple, ... (13 total), translating Hexagon-specific policy into reusable code paths. Command-line options appear here to gate diagnostics or target-specific tuning behavior.
- CN: 这里引入了 HexagonISelLowering.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/SetVector.h, ... (17 total) 等头文件，确定了后续代码依赖的 LLVM/Hexagon API。 这里打开了命名空间（llvm），把实现限定在 LLVM/Hexagon 组件作用域中。 这里声明或实现了 HvxWidenThreshold, EnableFpFastConvert, getIEEEProperties, std::make_tuple, ... (13 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这里出现了命令行选项，用于控制诊断开关或目标相关的调优行为。

### Lines 201-400 / 第 201-400 行

```cpp
   201:       setOperationAction(ISD::CONCAT_VECTORS, MVT::v64bf16, Custom);
   202: 
   203:       setOperationAction(ISD::MLOAD, MVT::v128bf16, Custom);
   204:       setOperationAction(ISD::MSTORE, MVT::v128bf16, Custom);
   205:       setOperationAction(ISD::BUILD_VECTOR, MVT::v128bf16, Custom);
   206:       setOperationAction(ISD::CONCAT_VECTORS, MVT::v128bf16, Custom);
   207: 
   208:       setOperationAction(ISD::SPLAT_VECTOR, MVT::bf16, Custom);
   209:       setOperationAction(ISD::INSERT_VECTOR_ELT, MVT::bf16, Custom);
   210:       setOperationAction(ISD::BUILD_VECTOR, MVT::bf16, Custom);
   211:     }
   212: 
   213:     for (MVT P : FloatW) {
   214:       setOperationAction(ISD::LOAD,           P, Custom);
   215:       setOperationAction(ISD::STORE,          P, Custom);
   216:       setOperationAction(ISD::FADD,           P, Custom);
   217:       setOperationAction(ISD::FSUB,           P, Custom);
   218:       setOperationAction(ISD::FMUL,           P, Custom);
   219:       setOperationAction(ISD::FMINIMUMNUM, P, Custom);
   220:       setOperationAction(ISD::FMAXIMUMNUM, P, Custom);
   221:       setOperationAction(ISD::SETCC,          P, Custom);
   222:       setOperationAction(ISD::VSELECT,        P, Custom);
   223: 
   224:       // Custom-lower BUILD_VECTOR. The standard (target-independent)
   225:       // handling of it would convert it to a load, which is not always
   226:       // the optimal choice.
   227:       setOperationAction(ISD::BUILD_VECTOR,   P, Custom);
   228:       // Make concat-vectors custom to handle concats of more than 2 vectors.
   229:       setOperationAction(ISD::CONCAT_VECTORS, P, Custom);
   230: 
   231:       setOperationAction(ISD::MLOAD,          P, Custom);
   232:       setOperationAction(ISD::MSTORE,         P, Custom);
   233:     }
   234: 
   235:     if (Subtarget.useHVXQFloatOps()) {
   236:       setOperationAction(ISD::FP_EXTEND, MVT::v64f32, Custom);
   237:       setOperationAction(ISD::FP_ROUND,  MVT::v64f16, Legal);
   238:     } else if (Subtarget.useHVXIEEEFPOps()) {
   239:       setOperationAction(ISD::FP_EXTEND, MVT::v64f32, Legal);
   240:       setOperationAction(ISD::FP_ROUND,  MVT::v64f16, Legal);
   241:     }
   242:   }
   243: 
   244:   for (MVT T : LegalV) {
   245:     setIndexedLoadAction(ISD::POST_INC,  T, Legal);
   246:     setIndexedStoreAction(ISD::POST_INC, T, Legal);
   247: 
   248:     setOperationAction(ISD::ABS,            T, Legal);
   249:     setOperationAction(ISD::AND,            T, Legal);
   250:     setOperationAction(ISD::OR,             T, Legal);
   251:     setOperationAction(ISD::XOR,            T, Legal);
   252:     setOperationAction(ISD::ADD,            T, Legal);
   253:     setOperationAction(ISD::SUB,            T, Legal);
   254:     setOperationAction(ISD::MUL,            T, Legal);
   255:     setOperationAction(ISD::CTPOP,          T, Legal);
   256:     setOperationAction(ISD::CTLZ,           T, Legal);
   257:     setOperationAction(ISD::SELECT,         T, Legal);
   258:     setOperationAction(ISD::SPLAT_VECTOR,   T, Legal);
   259:     setOperationAction(ISD::UADDSAT, T, Legal);
   260:     setOperationAction(ISD::SADDSAT, T, Legal);
   261:     setOperationAction(ISD::USUBSAT, T, Legal);
   262:     setOperationAction(ISD::SSUBSAT, T, Legal);
   263:     if (T != ByteV) {
   264:       setOperationAction(ISD::SIGN_EXTEND_VECTOR_INREG, T, Legal);
   265:       setOperationAction(ISD::ZERO_EXTEND_VECTOR_INREG, T, Legal);
   266:       setOperationAction(ISD::BSWAP,                    T, Legal);
   267:     }
   268: 
   269:     setOperationAction(ISD::SMIN,           T, Legal);
   270:     setOperationAction(ISD::SMAX,           T, Legal);
   271:     if (T.getScalarType() != MVT::i32) {
   272:       setOperationAction(ISD::UMIN,         T, Legal);
   273:       setOperationAction(ISD::UMAX,         T, Legal);
   274:     }
   275: 
   276:     setOperationAction(ISD::CTTZ,               T, Custom);
   277:     setOperationAction(ISD::LOAD,               T, Custom);
   278:     setOperationAction(ISD::MLOAD,              T, Custom);
   279:     setOperationAction(ISD::MSTORE,             T, Custom);
   280:     if (T.getScalarType() != MVT::i32) {
   281:       setOperationAction(ISD::MULHS,              T, Legal);
   282:       setOperationAction(ISD::MULHU,              T, Legal);
   283:     }
   284: 
   285:     setOperationAction(ISD::BUILD_VECTOR,       T, Custom);
   286:     // Make concat-vectors custom to handle concats of more than 2 vectors.
   287:     setOperationAction(ISD::CONCAT_VECTORS,     T, Custom);
   288:     setOperationAction(ISD::INSERT_SUBVECTOR,   T, Custom);
   289:     setOperationAction(ISD::INSERT_VECTOR_ELT,  T, Custom);
   290:     setOperationAction(ISD::EXTRACT_SUBVECTOR,  T, Custom);
   291:     setOperationAction(ISD::EXTRACT_VECTOR_ELT, T, Custom);
   292:     setOperationAction(ISD::ANY_EXTEND,         T, Custom);
   293:     setOperationAction(ISD::SIGN_EXTEND,        T, Custom);
   294:     setOperationAction(ISD::ZERO_EXTEND,        T, Custom);
   295:     setOperationAction(ISD::FSHL,               T, Custom);
   296:     setOperationAction(ISD::FSHR,               T, Custom);
   297:     if (T != ByteV) {
   298:       setOperationAction(ISD::ANY_EXTEND_VECTOR_INREG, T, Custom);
   299:       // HVX only has shifts of words and halfwords.
   300:       setOperationAction(ISD::SRA,                     T, Custom);
   301:       setOperationAction(ISD::SHL,                     T, Custom);
   302:       setOperationAction(ISD::SRL,                     T, Custom);
   303: 
   304:       // Promote all shuffles to operate on vectors of bytes.
   305:       setPromoteTo(ISD::VECTOR_SHUFFLE, T, ByteV);
   306:     }
   307: 
   308:     if (Subtarget.useHVXFloatingPoint()) {
   309:       // Same action for both QFloat and IEEE.
   310:       setOperationAction(ISD::SINT_TO_FP, T, Custom);
   311:       setOperationAction(ISD::UINT_TO_FP, T, Custom);
   312:       setOperationAction(ISD::FP_TO_SINT, T, Custom);
   313:       setOperationAction(ISD::FP_TO_UINT, T, Custom);
   314:     }
   315: 
   316:     setCondCodeAction(ISD::SETNE,  T, Expand);
   317:     setCondCodeAction(ISD::SETLE,  T, Expand);
   318:     setCondCodeAction(ISD::SETGE,  T, Expand);
   319:     setCondCodeAction(ISD::SETLT,  T, Expand);
   320:     setCondCodeAction(ISD::SETULE, T, Expand);
   321:     setCondCodeAction(ISD::SETUGE, T, Expand);
   322:     setCondCodeAction(ISD::SETULT, T, Expand);
   323:   }
   324: 
   325:   for (MVT T : LegalW) {
   326:     // Custom-lower BUILD_VECTOR for vector pairs. The standard (target-
   327:     // independent) handling of it would convert it to a load, which is
   328:     // not always the optimal choice.
   329:     setOperationAction(ISD::BUILD_VECTOR,   T, Custom);
   330:     // Make concat-vectors custom to handle concats of more than 2 vectors.
   331:     setOperationAction(ISD::CONCAT_VECTORS, T, Custom);
   332: 
   333:     // Custom-lower these operations for pairs. Expand them into a concat
   334:     // of the corresponding operations on individual vectors.
   335:     setOperationAction(ISD::ANY_EXTEND,               T, Custom);
   336:     setOperationAction(ISD::SIGN_EXTEND,              T, Custom);
   337:     setOperationAction(ISD::ZERO_EXTEND,              T, Custom);
   338:     setOperationAction(ISD::SIGN_EXTEND_INREG,        T, Custom);
   339:     setOperationAction(ISD::ANY_EXTEND_VECTOR_INREG,  T, Custom);
   340:     setOperationAction(ISD::SIGN_EXTEND_VECTOR_INREG, T, Legal);
   341:     setOperationAction(ISD::ZERO_EXTEND_VECTOR_INREG, T, Legal);
   342:     setOperationAction(ISD::SPLAT_VECTOR,             T, Custom);
   343: 
   344:     setOperationAction(ISD::LOAD,     T, Custom);
   345:     setOperationAction(ISD::STORE,    T, Custom);
   346:     setOperationAction(ISD::MLOAD,    T, Custom);
   347:     setOperationAction(ISD::MSTORE,   T, Custom);
   348:     setOperationAction(ISD::ABS,      T, Custom);
   349:     setOperationAction(ISD::CTLZ,     T, Custom);
   350:     setOperationAction(ISD::CTTZ,     T, Custom);
   351:     setOperationAction(ISD::CTPOP,    T, Custom);
   352: 
   353:     setOperationAction(ISD::ADD,      T, Legal);
   354:     setOperationAction(ISD::UADDSAT, T, Legal);
   355:     setOperationAction(ISD::SADDSAT, T, Legal);
   356:     setOperationAction(ISD::SUB,      T, Legal);
   357:     setOperationAction(ISD::USUBSAT, T, Legal);
   358:     setOperationAction(ISD::SSUBSAT, T, Legal);
   359:     setOperationAction(ISD::MUL,      T, Custom);
   360:     setOperationAction(ISD::MULHS,    T, Custom);
   361:     setOperationAction(ISD::MULHU,    T, Custom);
   362:     setOperationAction(ISD::AND,      T, Custom);
   363:     setOperationAction(ISD::OR,       T, Custom);
   364:     setOperationAction(ISD::XOR,      T, Custom);
   365:     setOperationAction(ISD::SETCC,    T, Custom);
   366:     setOperationAction(ISD::VSELECT,  T, Custom);
   367:     if (T != ByteW) {
   368:       setOperationAction(ISD::SRA,      T, Custom);
   369:       setOperationAction(ISD::SHL,      T, Custom);
   370:       setOperationAction(ISD::SRL,      T, Custom);
   371: 
   372:       // Promote all shuffles to operate on vectors of bytes.
   373:       setPromoteTo(ISD::VECTOR_SHUFFLE, T, ByteW);
   374:     }
   375:     setOperationAction(ISD::FSHL,     T, Custom);
   376:     setOperationAction(ISD::FSHR,     T, Custom);
   377: 
   378:     setOperationAction(ISD::SMIN,     T, Custom);
   379:     setOperationAction(ISD::SMAX,     T, Custom);
   380:     if (T.getScalarType() != MVT::i32) {
   381:       setOperationAction(ISD::UMIN,     T, Custom);
   382:       setOperationAction(ISD::UMAX,     T, Custom);
   383:     }
   384: 
   385:     if (Subtarget.useHVXFloatingPoint()) {
   386:       // Same action for both QFloat and IEEE.
   387:       setOperationAction(ISD::SINT_TO_FP, T, Custom);
   388:       setOperationAction(ISD::UINT_TO_FP, T, Custom);
   389:       setOperationAction(ISD::FP_TO_SINT, T, Custom);
   390:       setOperationAction(ISD::FP_TO_UINT, T, Custom);
   391:     }
   392:   }
   393: 
   394:   // Legalize all of these to HexagonISD::[SU]MUL_LOHI.
   395:   setOperationAction(ISD::MULHS,      WordV, Custom); // -> _LOHI
   396:   setOperationAction(ISD::MULHU,      WordV, Custom); // -> _LOHI
   397:   setOperationAction(ISD::SMUL_LOHI,  WordV, Custom);
   398:   setOperationAction(ISD::UMUL_LOHI,  WordV, Custom);
   399: 
   400:   setCondCodeAction(ISD::SETNE,  MVT::v64f16, Expand);
```
- EN: It declares or implements routines such as setOperationAction, standard, setIndexedLoadAction, setIndexedStoreAction, ... (6 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 setOperationAction, standard, setIndexedLoadAction, setIndexedStoreAction, ... (6 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 401-600 / 第 401-600 行

```cpp
   401:   setCondCodeAction(ISD::SETLE,  MVT::v64f16, Expand);
   402:   setCondCodeAction(ISD::SETGE,  MVT::v64f16, Expand);
   403:   setCondCodeAction(ISD::SETLT,  MVT::v64f16, Expand);
   404:   setCondCodeAction(ISD::SETONE, MVT::v64f16, Expand);
   405:   setCondCodeAction(ISD::SETOLE, MVT::v64f16, Expand);
   406:   setCondCodeAction(ISD::SETOGE, MVT::v64f16, Expand);
   407:   setCondCodeAction(ISD::SETOLT, MVT::v64f16, Expand);
   408:   setCondCodeAction(ISD::SETUNE, MVT::v64f16, Expand);
   409:   setCondCodeAction(ISD::SETULE, MVT::v64f16, Expand);
   410:   setCondCodeAction(ISD::SETUGE, MVT::v64f16, Expand);
   411:   setCondCodeAction(ISD::SETULT, MVT::v64f16, Expand);
   412:   setCondCodeAction(ISD::SETUO, MVT::v64f16, Expand);
   413:   setCondCodeAction(ISD::SETO, MVT::v64f16, Expand);
   414: 
   415:   setCondCodeAction(ISD::SETNE,  MVT::v32f32, Expand);
   416:   setCondCodeAction(ISD::SETLE,  MVT::v32f32, Expand);
   417:   setCondCodeAction(ISD::SETGE,  MVT::v32f32, Expand);
   418:   setCondCodeAction(ISD::SETLT,  MVT::v32f32, Expand);
   419:   setCondCodeAction(ISD::SETONE, MVT::v32f32, Expand);
   420:   setCondCodeAction(ISD::SETOLE, MVT::v32f32, Expand);
   421:   setCondCodeAction(ISD::SETOGE, MVT::v32f32, Expand);
   422:   setCondCodeAction(ISD::SETOLT, MVT::v32f32, Expand);
   423:   setCondCodeAction(ISD::SETUNE, MVT::v32f32, Expand);
   424:   setCondCodeAction(ISD::SETULE, MVT::v32f32, Expand);
   425:   setCondCodeAction(ISD::SETUGE, MVT::v32f32, Expand);
   426:   setCondCodeAction(ISD::SETULT, MVT::v32f32, Expand);
   427:   setCondCodeAction(ISD::SETUO, MVT::v32f32, Expand);
   428:   setCondCodeAction(ISD::SETO, MVT::v32f32, Expand);
   429: 
   430:   // Boolean vectors.
   431: 
   432:   for (MVT T : LegalW) {
   433:     // Boolean types for vector pairs will overlap with the boolean
   434:     // types for single vectors, e.g.
   435:     //   v64i8  -> v64i1 (single)
   436:     //   v64i16 -> v64i1 (pair)
   437:     // Set these actions first, and allow the single actions to overwrite
   438:     // any duplicates.
   439:     MVT BoolW = MVT::getVectorVT(MVT::i1, T.getVectorNumElements());
   440:     setOperationAction(ISD::SETCC,              BoolW, Custom);
   441:     setOperationAction(ISD::AND,                BoolW, Custom);
   442:     setOperationAction(ISD::OR,                 BoolW, Custom);
   443:     setOperationAction(ISD::XOR,                BoolW, Custom);
   444:     // Masked load/store takes a mask that may need splitting.
   445:     setOperationAction(ISD::MLOAD,              BoolW, Custom);
   446:     setOperationAction(ISD::MSTORE,             BoolW, Custom);
   447:   }
   448: 
   449:   for (MVT T : LegalV) {
   450:     MVT BoolV = MVT::getVectorVT(MVT::i1, T.getVectorNumElements());
   451:     setOperationAction(ISD::BUILD_VECTOR,       BoolV, Custom);
   452:     setOperationAction(ISD::CONCAT_VECTORS,     BoolV, Custom);
   453:     setOperationAction(ISD::INSERT_SUBVECTOR,   BoolV, Custom);
   454:     setOperationAction(ISD::INSERT_VECTOR_ELT,  BoolV, Custom);
   455:     setOperationAction(ISD::EXTRACT_SUBVECTOR,  BoolV, Custom);
   456:     setOperationAction(ISD::EXTRACT_VECTOR_ELT, BoolV, Custom);
   457:     setOperationAction(ISD::SELECT,             BoolV, Custom);
   458:     setOperationAction(ISD::AND,                BoolV, Legal);
   459:     setOperationAction(ISD::OR,                 BoolV, Legal);
   460:     setOperationAction(ISD::XOR,                BoolV, Legal);
   461:   }
   462: 
   463:   if (Use64b) {
   464:     for (MVT T: {MVT::v32i8, MVT::v32i16, MVT::v16i8, MVT::v16i16, MVT::v16i32})
   465:       setOperationAction(ISD::SIGN_EXTEND_INREG, T, Legal);
   466:   } else {
   467:     for (MVT T: {MVT::v64i8, MVT::v64i16, MVT::v32i8, MVT::v32i16, MVT::v32i32})
   468:       setOperationAction(ISD::SIGN_EXTEND_INREG, T, Legal);
   469:   }
   470: 
   471:   // Handle store widening for short vectors.
   472:   unsigned HwLen = Subtarget.getVectorLength();
   473:   for (MVT ElemTy : Subtarget.getHVXElementTypes()) {
   474:     if (ElemTy == MVT::i1)
   475:       continue;
   476:     int ElemWidth = ElemTy.getFixedSizeInBits();
   477:     int MaxElems = (8*HwLen) / ElemWidth;
   478:     for (int N = 2; N < MaxElems; N *= 2) {
   479:       MVT VecTy = MVT::getVectorVT(ElemTy, N);
   480:       auto Action = getPreferredVectorAction(VecTy);
   481:       if (Action == TargetLoweringBase::TypeWidenVector) {
   482:         setOperationAction(ISD::LOAD,         VecTy, Custom);
   483:         setOperationAction(ISD::STORE,        VecTy, Custom);
   484:         setOperationAction(ISD::SETCC,        VecTy, Custom);
   485:         setOperationAction(ISD::TRUNCATE,     VecTy, Custom);
   486:         setOperationAction(ISD::ANY_EXTEND,   VecTy, Custom);
   487:         setOperationAction(ISD::SIGN_EXTEND,  VecTy, Custom);
   488:         setOperationAction(ISD::ZERO_EXTEND, VecTy, Custom);
   489:         if (Subtarget.useHVXFloatingPoint()) {
   490:           setOperationAction(ISD::FP_TO_SINT,   VecTy, Custom);
   491:           setOperationAction(ISD::FP_TO_UINT,   VecTy, Custom);
   492:           setOperationAction(ISD::SINT_TO_FP,   VecTy, Custom);
   493:           setOperationAction(ISD::UINT_TO_FP,   VecTy, Custom);
   494:         }
   495: 
   496:         MVT BoolTy = MVT::getVectorVT(MVT::i1, N);
   497:         if (!isTypeLegal(BoolTy))
   498:           setOperationAction(ISD::SETCC, BoolTy, Custom);
   499:       }
   500:     }
   501:   }
   502: 
   503:   // Include cases which are not hander earlier
   504:   setOperationAction(ISD::UINT_TO_FP, MVT::v32i1, Custom);
   505:   setOperationAction(ISD::UINT_TO_FP, MVT::v64i1, Custom);
   506:   setOperationAction(ISD::SINT_TO_FP, MVT::v32i1, Custom);
   507: 
   508:   setTargetDAGCombine({ISD::CONCAT_VECTORS, ISD::TRUNCATE, ISD::VSELECT});
   509: 
   510:   setTargetDAGCombine({ISD::PARTIAL_REDUCE_SMLA, ISD::PARTIAL_REDUCE_UMLA,
   511:                        ISD::PARTIAL_REDUCE_SUMLA});
   512: 
   513:   // Partial MLA reductions.
   514:   {
   515:     static const unsigned MLAOps[] = {ISD::PARTIAL_REDUCE_SMLA,
   516:                                       ISD::PARTIAL_REDUCE_UMLA,
   517:                                       ISD::PARTIAL_REDUCE_SUMLA};
   518: 
   519:     auto HvxType = [=](MVT ScalarT, unsigned Factor = 1) {
   520:       return MVT::getVectorVT(ScalarT, Subtarget.getVectorLength() * Factor *
   521:                                            8 / ScalarT.getSizeInBits());
   522:     };
   523: 
   524:     // Tuple of (Acc element type, input element type, vector pair).
   525:     // The assumption is both the input and reduction result are of the same
   526:     // size so the reduction ratio is the same as the ratio of element type
   527:     // sizes. This may not hold for all available instructions.
   528:     typedef std::tuple<MVT, MVT, bool> ReductionSignature;
   529: 
   530:     static const std::vector<ReductionSignature> NativeReductions = {
   531:         {MVT::i32, MVT::i8, false},
   532:     };
   533: 
   534:     for (const auto &R : NativeReductions) {
   535: 
   536:       MVT AccType = std::get<0>(R);
   537:       MVT InputType = std::get<1>(R);
   538:       unsigned Factor = std::get<2>(R) ? 2 : 1;
   539: 
   540:       // The native size is legal.
   541:       setPartialReduceMLAAction(MLAOps, HvxType(AccType), HvxType(InputType),
   542:                                 Legal);
   543: 
   544:       // Allow custom partial MLA reductions on larger vectors than legally
   545:       // supported. These reduction must be declared as Custom (or Legal)
   546:       // for foldPartialReduceMLAMulOp() to fold the multiply by one pattern
   547:       // inserted when the partial reduction intrinsic is converted to
   548:       // PARTIAL_REDUCE_U/S/SUMLA. Otherwise, the Split action will apply
   549:       // on the original pattern, including the extensions and multiplies,
   550:       // which will make it impossible to match.
   551:       // There are two independent ways to extend the
   552:       // input size: 1. to concatenate the result - output vector is
   553:       // proportionally extended, 2) to reduce the result - the output vector
   554:       // size stays the same. We limit allowed combinations so that the total
   555:       // number of generated reduction instructions is limited by a constant
   556:       // number. This limit is arbitrary and can be revised. On one hand, it is
   557:       // convenient to have more choices; on the other hand, there is a
   558:       // diminishing benefit of very long sequences, which should probably be
   559:       // written as loops instead.
   560:       for (unsigned ConcatFactor = 1; ConcatFactor <= MaxExpandMLA;
   561:            ConcatFactor <<= 1)
   562:         for (unsigned ReductionFactor = 1; ReductionFactor <= MaxExpandMLA;
   563:              ReductionFactor <<= 1)
   564:           if (ConcatFactor * ReductionFactor != 1 &&
   565:               ConcatFactor * ReductionFactor <= MaxExpandMLA)
   566:             setPartialReduceMLAAction(
   567:                 MLAOps, HvxType(AccType, Factor * ConcatFactor),
   568:                 HvxType(InputType, Factor * ConcatFactor * ReductionFactor),
   569:                 Custom);
   570:     }
   571:   }
   572: }
   573: 
   574: unsigned
   575: HexagonTargetLowering::getPreferredHvxVectorAction(MVT VecTy) const {
   576:   // Early exit for invalid input types
   577:   if (!VecTy.isVector())
   578:     return ~0u;
   579: 
   580:   MVT ElemTy = VecTy.getVectorElementType();
   581:   unsigned VecLen = VecTy.getVectorNumElements();
   582:   unsigned HwLen = Subtarget.getVectorLength();
   583: 
   584:   // Split vectors of i1 that exceed byte vector length.
   585:   if (ElemTy == MVT::i1 && VecLen > HwLen)
   586:     return TargetLoweringBase::TypeSplitVector;
   587: 
   588:   ArrayRef<MVT> Tys = Subtarget.getHVXElementTypes();
   589:   // For shorter vectors of i1, widen them if any of the corresponding
   590:   // vectors of integers needs to be widened.
   591:   if (ElemTy == MVT::i1) {
   592:     for (MVT T : Tys) {
   593:       assert(T != MVT::i1);
   594:       auto A = getPreferredHvxVectorAction(MVT::getVectorVT(T, VecLen));
   595:       if (A != ~0u)
   596:         return A;
   597:     }
   598:     return ~0u;
   599:   }
   600: 
```
- EN: It declares or implements routines such as setCondCodeAction, v64i1, setOperationAction, MVT::getVectorVT, ... (16 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 setCondCodeAction, v64i1, setOperationAction, MVT::getVectorVT, ... (16 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering，说明了它与同级后端组件的连接关系。

### Lines 601-800 / 第 601-800 行

```cpp
   601:   // If the size of VecTy is at least half of the vector length,
   602:   // widen the vector. Note: the threshold was not selected in
   603:   // any scientific way.
   604:   if (llvm::is_contained(Tys, ElemTy)) {
   605:     unsigned VecWidth = VecTy.getSizeInBits();
   606:     unsigned HwWidth = 8*HwLen;
   607:     if (VecWidth > 2*HwWidth)
   608:       return TargetLoweringBase::TypeSplitVector;
   609: 
   610:     bool HaveThreshold = HvxWidenThreshold.getNumOccurrences() > 0;
   611:     if (HaveThreshold && 8*HvxWidenThreshold <= VecWidth)
   612:       return TargetLoweringBase::TypeWidenVector;
   613:     if (VecWidth >= HwWidth/2 && VecWidth < HwWidth)
   614:       return TargetLoweringBase::TypeWidenVector;
   615:   }
   616: 
   617:   // Defer to default.
   618:   return ~0u;
   619: }
   620: 
   621: unsigned
   622: HexagonTargetLowering::getCustomHvxOperationAction(SDNode &Op) const {
   623:   unsigned Opc = Op.getOpcode();
   624:   switch (Opc) {
   625:   case HexagonISD::SMUL_LOHI:
   626:   case HexagonISD::UMUL_LOHI:
   627:   case HexagonISD::USMUL_LOHI:
   628:     return TargetLoweringBase::Custom;
   629:   }
   630:   return TargetLoweringBase::Legal;
   631: }
   632: 
   633: SDValue
   634: HexagonTargetLowering::getInt(unsigned IntId, MVT ResTy, ArrayRef<SDValue> Ops,
   635:                               const SDLoc &dl, SelectionDAG &DAG) const {
   636:   SmallVector<SDValue,4> IntOps;
   637:   IntOps.push_back(DAG.getConstant(IntId, dl, MVT::i32));
   638:   append_range(IntOps, Ops);
   639:   return DAG.getNode(ISD::INTRINSIC_WO_CHAIN, dl, ResTy, IntOps);
   640: }
   641: 
   642: MVT
   643: HexagonTargetLowering::typeJoin(const TypePair &Tys) const {
   644:   assert(Tys.first.getVectorElementType() == Tys.second.getVectorElementType());
   645: 
   646:   MVT ElemTy = Tys.first.getVectorElementType();
   647:   return MVT::getVectorVT(ElemTy, Tys.first.getVectorNumElements() +
   648:                                   Tys.second.getVectorNumElements());
   649: }
   650: 
   651: HexagonTargetLowering::TypePair
   652: HexagonTargetLowering::typeSplit(MVT VecTy) const {
   653:   assert(VecTy.isVector());
   654:   unsigned NumElem = VecTy.getVectorNumElements();
   655:   assert((NumElem % 2) == 0 && "Expecting even-sized vector type");
   656:   MVT HalfTy = MVT::getVectorVT(VecTy.getVectorElementType(), NumElem/2);
   657:   return { HalfTy, HalfTy };
   658: }
   659: 
   660: MVT
   661: HexagonTargetLowering::typeExtElem(MVT VecTy, unsigned Factor) const {
   662:   MVT ElemTy = VecTy.getVectorElementType();
   663:   MVT NewElemTy = MVT::getIntegerVT(ElemTy.getSizeInBits() * Factor);
   664:   return MVT::getVectorVT(NewElemTy, VecTy.getVectorNumElements());
   665: }
   666: 
   667: MVT
   668: HexagonTargetLowering::typeTruncElem(MVT VecTy, unsigned Factor) const {
   669:   MVT ElemTy = VecTy.getVectorElementType();
   670:   MVT NewElemTy = MVT::getIntegerVT(ElemTy.getSizeInBits() / Factor);
   671:   return MVT::getVectorVT(NewElemTy, VecTy.getVectorNumElements());
   672: }
   673: 
   674: SDValue
   675: HexagonTargetLowering::opCastElem(SDValue Vec, MVT ElemTy,
   676:                                   SelectionDAG &DAG) const {
   677:   if (ty(Vec).getVectorElementType() == ElemTy)
   678:     return Vec;
   679:   MVT CastTy = tyVector(Vec.getValueType().getSimpleVT(), ElemTy);
   680:   return DAG.getBitcast(CastTy, Vec);
   681: }
   682: 
   683: SDValue
   684: HexagonTargetLowering::opJoin(const VectorPair &Ops, const SDLoc &dl,
   685:                               SelectionDAG &DAG) const {
   686:   return DAG.getNode(ISD::CONCAT_VECTORS, dl, typeJoin(ty(Ops)),
   687:                      Ops.first, Ops.second);
   688: }
   689: 
   690: HexagonTargetLowering::VectorPair
   691: HexagonTargetLowering::opSplit(SDValue Vec, const SDLoc &dl,
   692:                                SelectionDAG &DAG) const {
   693:   TypePair Tys = typeSplit(ty(Vec));
   694:   if (Vec.getOpcode() == HexagonISD::QCAT)
   695:     return VectorPair(Vec.getOperand(0), Vec.getOperand(1));
   696:   return DAG.SplitVector(Vec, dl, Tys.first, Tys.second);
   697: }
   698: 
   699: bool
   700: HexagonTargetLowering::isHvxSingleTy(MVT Ty) const {
   701:   return Subtarget.isHVXVectorType(Ty) &&
   702:          Ty.getSizeInBits() == 8 * Subtarget.getVectorLength();
   703: }
   704: 
   705: bool
   706: HexagonTargetLowering::isHvxPairTy(MVT Ty) const {
   707:   return Subtarget.isHVXVectorType(Ty) &&
   708:          Ty.getSizeInBits() == 16 * Subtarget.getVectorLength();
   709: }
   710: 
   711: bool
   712: HexagonTargetLowering::isHvxBoolTy(MVT Ty) const {
   713:   return Subtarget.isHVXVectorType(Ty, true) &&
   714:          Ty.getVectorElementType() == MVT::i1;
   715: }
   716: 
   717: bool HexagonTargetLowering::allowsHvxMemoryAccess(
   718:     MVT VecTy, MachineMemOperand::Flags Flags, unsigned *Fast) const {
   719:   // Bool vectors are excluded by default, but make it explicit to
   720:   // emphasize that bool vectors cannot be loaded or stored.
   721:   // Also, disallow double vector stores (to prevent unnecessary
   722:   // store widening in DAG combiner).
   723:   if (VecTy.getSizeInBits() > 8*Subtarget.getVectorLength())
   724:     return false;
   725:   if (!Subtarget.isHVXVectorType(VecTy, /*IncludeBool=*/false))
   726:     return false;
   727:   if (Fast)
   728:     *Fast = 1;
   729:   return true;
   730: }
   731: 
   732: bool HexagonTargetLowering::allowsHvxMisalignedMemoryAccesses(
   733:     MVT VecTy, MachineMemOperand::Flags Flags, unsigned *Fast) const {
   734:   if (!Subtarget.isHVXVectorType(VecTy))
   735:     return false;
   736:   // XXX Should this be false?  vmemu are a bit slower than vmem.
   737:   if (Fast)
   738:     *Fast = 1;
   739:   return true;
   740: }
   741: 
   742: void HexagonTargetLowering::AdjustHvxInstrPostInstrSelection(
   743:     MachineInstr &MI, SDNode *Node) const {
   744:   unsigned Opc = MI.getOpcode();
   745:   const TargetInstrInfo &TII = *Subtarget.getInstrInfo();
   746:   MachineBasicBlock &MB = *MI.getParent();
   747:   MachineFunction &MF = *MB.getParent();
   748:   MachineRegisterInfo &MRI = MF.getRegInfo();
   749:   DebugLoc DL = MI.getDebugLoc();
   750:   auto At = MI.getIterator();
   751: 
   752:   switch (Opc) {
   753:   case Hexagon::PS_vsplatib:
   754:     if (Subtarget.useHVXV62Ops()) {
   755:       // SplatV = A2_tfrsi #imm
   756:       // OutV = V6_lvsplatb SplatV
   757:       Register SplatV = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
   758:       BuildMI(MB, At, DL, TII.get(Hexagon::A2_tfrsi), SplatV)
   759:         .add(MI.getOperand(1));
   760:       Register OutV = MI.getOperand(0).getReg();
   761:       BuildMI(MB, At, DL, TII.get(Hexagon::V6_lvsplatb), OutV)
   762:         .addReg(SplatV);
   763:     } else {
   764:       // SplatV = A2_tfrsi #imm:#imm:#imm:#imm
   765:       // OutV = V6_lvsplatw SplatV
   766:       Register SplatV = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
   767:       const MachineOperand &InpOp = MI.getOperand(1);
   768:       assert(InpOp.isImm());
   769:       uint32_t V = InpOp.getImm() & 0xFF;
   770:       BuildMI(MB, At, DL, TII.get(Hexagon::A2_tfrsi), SplatV)
   771:           .addImm(V << 24 | V << 16 | V << 8 | V);
   772:       Register OutV = MI.getOperand(0).getReg();
   773:       BuildMI(MB, At, DL, TII.get(Hexagon::V6_lvsplatw), OutV).addReg(SplatV);
   774:     }
   775:     MB.erase(At);
   776:     break;
   777:   case Hexagon::PS_vsplatrb:
   778:     if (Subtarget.useHVXV62Ops()) {
   779:       // OutV = V6_lvsplatb Inp
   780:       Register OutV = MI.getOperand(0).getReg();
   781:       BuildMI(MB, At, DL, TII.get(Hexagon::V6_lvsplatb), OutV)
   782:         .add(MI.getOperand(1));
   783:     } else {
   784:       Register SplatV = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
   785:       const MachineOperand &InpOp = MI.getOperand(1);
   786:       BuildMI(MB, At, DL, TII.get(Hexagon::S2_vsplatrb), SplatV)
   787:           .addReg(InpOp.getReg(), {}, InpOp.getSubReg());
   788:       Register OutV = MI.getOperand(0).getReg();
   789:       BuildMI(MB, At, DL, TII.get(Hexagon::V6_lvsplatw), OutV)
   790:           .addReg(SplatV);
   791:     }
   792:     MB.erase(At);
   793:     break;
   794:   case Hexagon::PS_vsplatih:
   795:     if (Subtarget.useHVXV62Ops()) {
   796:       // SplatV = A2_tfrsi #imm
   797:       // OutV = V6_lvsplath SplatV
   798:       Register SplatV = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
   799:       BuildMI(MB, At, DL, TII.get(Hexagon::A2_tfrsi), SplatV)
   800:         .add(MI.getOperand(1));
```
- EN: It declares or implements routines such as getSizeInBits, HexagonTargetLowering::getCustomHvxOperationAction, getOpcode, HexagonTargetLowering::getInt, ... (40 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getSizeInBits, HexagonTargetLowering::getCustomHvxOperationAction, getOpcode, HexagonTargetLowering::getInt, ... (40 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 801-1000 / 第 801-1000 行

```cpp
   801:       Register OutV = MI.getOperand(0).getReg();
   802:       BuildMI(MB, At, DL, TII.get(Hexagon::V6_lvsplath), OutV)
   803:         .addReg(SplatV);
   804:     } else {
   805:       // SplatV = A2_tfrsi #imm:#imm
   806:       // OutV = V6_lvsplatw SplatV
   807:       Register SplatV = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
   808:       const MachineOperand &InpOp = MI.getOperand(1);
   809:       assert(InpOp.isImm());
   810:       uint32_t V = InpOp.getImm() & 0xFFFF;
   811:       BuildMI(MB, At, DL, TII.get(Hexagon::A2_tfrsi), SplatV)
   812:           .addImm(V << 16 | V);
   813:       Register OutV = MI.getOperand(0).getReg();
   814:       BuildMI(MB, At, DL, TII.get(Hexagon::V6_lvsplatw), OutV).addReg(SplatV);
   815:     }
   816:     MB.erase(At);
   817:     break;
   818:   case Hexagon::PS_vsplatrh:
   819:     if (Subtarget.useHVXV62Ops()) {
   820:       // OutV = V6_lvsplath Inp
   821:       Register OutV = MI.getOperand(0).getReg();
   822:       BuildMI(MB, At, DL, TII.get(Hexagon::V6_lvsplath), OutV)
   823:         .add(MI.getOperand(1));
   824:     } else {
   825:       // SplatV = A2_combine_ll Inp, Inp
   826:       // OutV = V6_lvsplatw SplatV
   827:       Register SplatV = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
   828:       const MachineOperand &InpOp = MI.getOperand(1);
   829:       BuildMI(MB, At, DL, TII.get(Hexagon::A2_combine_ll), SplatV)
   830:           .addReg(InpOp.getReg(), {}, InpOp.getSubReg())
   831:           .addReg(InpOp.getReg(), {}, InpOp.getSubReg());
   832:       Register OutV = MI.getOperand(0).getReg();
   833:       BuildMI(MB, At, DL, TII.get(Hexagon::V6_lvsplatw), OutV).addReg(SplatV);
   834:     }
   835:     MB.erase(At);
   836:     break;
   837:   case Hexagon::PS_vsplatiw:
   838:   case Hexagon::PS_vsplatrw:
   839:     if (Opc == Hexagon::PS_vsplatiw) {
   840:       // SplatV = A2_tfrsi #imm
   841:       Register SplatV = MRI.createVirtualRegister(&Hexagon::IntRegsRegClass);
   842:       BuildMI(MB, At, DL, TII.get(Hexagon::A2_tfrsi), SplatV)
   843:         .add(MI.getOperand(1));
   844:       MI.getOperand(1).ChangeToRegister(SplatV, false);
   845:     }
   846:     // OutV = V6_lvsplatw SplatV/Inp
   847:     MI.setDesc(TII.get(Hexagon::V6_lvsplatw));
   848:     break;
   849:   }
   850: }
   851: 
   852: SDValue
   853: HexagonTargetLowering::convertToByteIndex(SDValue ElemIdx, MVT ElemTy,
   854:                                           SelectionDAG &DAG) const {
   855:   if (ElemIdx.getValueType().getSimpleVT() != MVT::i32)
   856:     ElemIdx = DAG.getBitcast(MVT::i32, ElemIdx);
   857: 
   858:   unsigned ElemWidth = ElemTy.getSizeInBits();
   859:   if (ElemWidth == 8)
   860:     return ElemIdx;
   861: 
   862:   unsigned L = Log2_32(ElemWidth/8);
   863:   const SDLoc &dl(ElemIdx);
   864:   return DAG.getNode(ISD::SHL, dl, MVT::i32,
   865:                      {ElemIdx, DAG.getConstant(L, dl, MVT::i32)});
   866: }
   867: 
   868: SDValue
   869: HexagonTargetLowering::getIndexInWord32(SDValue Idx, MVT ElemTy,
   870:                                         SelectionDAG &DAG) const {
   871:   unsigned ElemWidth = ElemTy.getSizeInBits();
   872:   assert(ElemWidth >= 8 && ElemWidth <= 32);
   873:   if (ElemWidth == 32)
   874:     return Idx;
   875: 
   876:   if (ty(Idx) != MVT::i32)
   877:     Idx = DAG.getBitcast(MVT::i32, Idx);
   878:   const SDLoc &dl(Idx);
   879:   SDValue Mask = DAG.getConstant(32/ElemWidth - 1, dl, MVT::i32);
   880:   SDValue SubIdx = DAG.getNode(ISD::AND, dl, MVT::i32, {Idx, Mask});
   881:   return SubIdx;
   882: }
   883: 
   884: SDValue
   885: HexagonTargetLowering::getByteShuffle(const SDLoc &dl, SDValue Op0,
   886:                                       SDValue Op1, ArrayRef<int> Mask,
   887:                                       SelectionDAG &DAG) const {
   888:   MVT OpTy = ty(Op0);
   889:   assert(OpTy == ty(Op1));
   890: 
   891:   MVT ElemTy = OpTy.getVectorElementType();
   892:   if (ElemTy == MVT::i8)
   893:     return DAG.getVectorShuffle(OpTy, dl, Op0, Op1, Mask);
   894:   assert(ElemTy.getSizeInBits() >= 8);
   895: 
   896:   MVT ResTy = tyVector(OpTy, MVT::i8);
   897:   unsigned ElemSize = ElemTy.getSizeInBits() / 8;
   898: 
   899:   SmallVector<int,128> ByteMask;
   900:   for (int M : Mask) {
   901:     if (M < 0) {
   902:       for (unsigned I = 0; I != ElemSize; ++I)
   903:         ByteMask.push_back(-1);
   904:     } else {
   905:       int NewM = M*ElemSize;
   906:       for (unsigned I = 0; I != ElemSize; ++I)
   907:         ByteMask.push_back(NewM+I);
   908:     }
   909:   }
   910:   assert(ResTy.getVectorNumElements() == ByteMask.size());
   911:   return DAG.getVectorShuffle(ResTy, dl, opCastElem(Op0, MVT::i8, DAG),
   912:                               opCastElem(Op1, MVT::i8, DAG), ByteMask);
   913: }
   914: 
   915: SDValue
   916: HexagonTargetLowering::buildHvxVectorReg(ArrayRef<SDValue> Values,
   917:                                          const SDLoc &dl, MVT VecTy,
   918:                                          SelectionDAG &DAG) const {
   919:   unsigned VecLen = Values.size();
   920:   MachineFunction &MF = DAG.getMachineFunction();
   921:   MVT ElemTy = VecTy.getVectorElementType();
   922:   unsigned ElemWidth = ElemTy.getSizeInBits();
   923:   unsigned HwLen = Subtarget.getVectorLength();
   924: 
   925:   unsigned ElemSize = ElemWidth / 8;
   926:   assert(ElemSize*VecLen == HwLen);
   927:   SmallVector<SDValue,32> Words;
   928: 
   929:   if (VecTy.getVectorElementType() != MVT::i32 &&
   930:       !(Subtarget.useHVXFloatingPoint() &&
   931:       VecTy.getVectorElementType() == MVT::f32)) {
   932:     assert((ElemSize == 1 || ElemSize == 2) && "Invalid element size");
   933:     unsigned OpsPerWord = (ElemSize == 1) ? 4 : 2;
   934:     MVT PartVT = MVT::getVectorVT(VecTy.getVectorElementType(), OpsPerWord);
   935:     for (unsigned i = 0; i != VecLen; i += OpsPerWord) {
   936:       SDValue W = buildVector32(Values.slice(i, OpsPerWord), dl, PartVT, DAG);
   937:       Words.push_back(DAG.getBitcast(MVT::i32, W));
   938:     }
   939:   } else {
   940:     for (SDValue V : Values)
   941:       Words.push_back(DAG.getBitcast(MVT::i32, V));
   942:   }
   943:   auto isSplat = [] (ArrayRef<SDValue> Values, SDValue &SplatV) {
   944:     unsigned NumValues = Values.size();
   945:     assert(NumValues > 0);
   946:     bool IsUndef = true;
   947:     for (unsigned i = 0; i != NumValues; ++i) {
   948:       if (Values[i].isUndef())
   949:         continue;
   950:       IsUndef = false;
   951:       if (!SplatV.getNode())
   952:         SplatV = Values[i];
   953:       else if (SplatV != Values[i])
   954:         return false;
   955:     }
   956:     if (IsUndef)
   957:       SplatV = Values[0];
   958:     return true;
   959:   };
   960: 
   961:   unsigned NumWords = Words.size();
   962:   SDValue SplatV;
   963:   bool IsSplat = isSplat(Words, SplatV);
   964:   if (IsSplat && isUndef(SplatV))
   965:     return DAG.getUNDEF(VecTy);
   966:   if (IsSplat) {
   967:     assert(SplatV.getNode());
   968:     if (isNullConstant(SplatV))
   969:       return getZero(dl, VecTy, DAG);
   970:     MVT WordTy = MVT::getVectorVT(MVT::i32, HwLen/4);
   971:     SDValue S = DAG.getNode(ISD::SPLAT_VECTOR, dl, WordTy, SplatV);
   972:     return DAG.getBitcast(VecTy, S);
   973:   }
   974: 
   975:   // Delay recognizing constant vectors until here, so that we can generate
   976:   // a vsplat.
   977:   SmallVector<ConstantInt*, 128> Consts(VecLen);
   978:   bool AllConst = getBuildVectorConstInts(Values, VecTy, DAG, Consts);
   979:   if (AllConst) {
   980:     ArrayRef<Constant*> Tmp((Constant**)Consts.begin(),
   981:                             (Constant**)Consts.end());
   982:     Constant *CV = ConstantVector::get(Tmp);
   983:     Align Alignment(HwLen);
   984:     SDValue CP = LowerConstantPool(
   985:         DAG.getConstantPool(CV, getPointerTy(DAG.getDataLayout()), Alignment),
   986:         DAG);
   987:     return DAG.getLoad(VecTy, dl, DAG.getEntryNode(), CP,
   988:                        MachinePointerInfo::getConstantPool(MF), Alignment);
   989:   }
   990: 
   991:   // A special case is a situation where the vector is built entirely from
   992:   // elements extracted from another vector. This could be done via a shuffle
   993:   // more efficiently, but typically, the size of the source vector will not
   994:   // match the size of the vector being built (which precludes the use of a
   995:   // shuffle directly).
   996:   // This only handles a single source vector, and the vector being built
   997:   // should be of a sub-vector type of the source vector type.
   998:   auto IsBuildFromExtracts = [this,&Values] (SDValue &SrcVec,
   999:                                              SmallVectorImpl<int> &SrcIdx) {
  1000:     SDValue Vec;
```
- EN: It declares or implements routines such as getOperand, BuildMI, createVirtualRegister, assert, ... (36 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getOperand, BuildMI, createVirtualRegister, assert, ... (36 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering，说明了它与同级后端组件的连接关系。

### Lines 1001-1200 / 第 1001-1200 行

```cpp
  1001:     for (SDValue V : Values) {
  1002:       if (isUndef(V)) {
  1003:         SrcIdx.push_back(-1);
  1004:         continue;
  1005:       }
  1006:       if (V.getOpcode() != ISD::EXTRACT_VECTOR_ELT)
  1007:         return false;
  1008:       // All extracts should come from the same vector.
  1009:       SDValue T = V.getOperand(0);
  1010:       if (Vec.getNode() != nullptr && T.getNode() != Vec.getNode())
  1011:         return false;
  1012:       Vec = T;
  1013:       ConstantSDNode *C = dyn_cast<ConstantSDNode>(V.getOperand(1));
  1014:       if (C == nullptr)
  1015:         return false;
  1016:       int I = C->getSExtValue();
  1017:       assert(I >= 0 && "Negative element index");
  1018:       SrcIdx.push_back(I);
  1019:     }
  1020:     SrcVec = Vec;
  1021:     return true;
  1022:   };
  1023: 
  1024:   SmallVector<int,128> ExtIdx;
  1025:   SDValue ExtVec;
  1026:   if (IsBuildFromExtracts(ExtVec, ExtIdx)) {
  1027:     MVT ExtTy = ty(ExtVec);
  1028:     unsigned ExtLen = ExtTy.getVectorNumElements();
  1029:     if (ExtLen == VecLen || ExtLen == 2*VecLen) {
  1030:       // Construct a new shuffle mask that will produce a vector with the same
  1031:       // number of elements as the input vector, and such that the vector we
  1032:       // want will be the initial subvector of it.
  1033:       SmallVector<int,128> Mask;
  1034:       BitVector Used(ExtLen);
  1035: 
  1036:       for (int M : ExtIdx) {
  1037:         Mask.push_back(M);
  1038:         if (M >= 0)
  1039:           Used.set(M);
  1040:       }
  1041:       // Fill the rest of the mask with the unused elements of ExtVec in hopes
  1042:       // that it will result in a permutation of ExtVec's elements. It's still
  1043:       // fine if it doesn't (e.g. if undefs are present, or elements are
  1044:       // repeated), but permutations can always be done efficiently via vdelta
  1045:       // and vrdelta.
  1046:       for (unsigned I = 0; I != ExtLen; ++I) {
  1047:         if (Mask.size() == ExtLen)
  1048:           break;
  1049:         if (!Used.test(I))
  1050:           Mask.push_back(I);
  1051:       }
  1052: 
  1053:       SDValue S = DAG.getVectorShuffle(ExtTy, dl, ExtVec,
  1054:                                        DAG.getUNDEF(ExtTy), Mask);
  1055:       return ExtLen == VecLen ? S : LoHalf(S, DAG);
  1056:     }
  1057:   }
  1058: 
  1059:   // Find most common element to initialize vector with. This is to avoid
  1060:   // unnecessary vinsert/valign for cases where the same value is present
  1061:   // many times. Creates a histogram of the vector's elements to find the
  1062:   // most common element n.
  1063:   assert(4*Words.size() == Subtarget.getVectorLength());
  1064:   int VecHist[32];
  1065:   int n = 0;
  1066:   for (unsigned i = 0; i != NumWords; ++i) {
  1067:     VecHist[i] = 0;
  1068:     if (Words[i].isUndef())
  1069:       continue;
  1070:     for (unsigned j = i; j != NumWords; ++j)
  1071:       if (Words[i] == Words[j])
  1072:         VecHist[i]++;
  1073: 
  1074:     if (VecHist[i] > VecHist[n])
  1075:       n = i;
  1076:   }
  1077: 
  1078:   SDValue HalfV = getZero(dl, VecTy, DAG);
  1079:   if (VecHist[n] > 1) {
  1080:     SDValue SplatV = DAG.getNode(ISD::SPLAT_VECTOR, dl, VecTy, Words[n]);
  1081:     HalfV = DAG.getNode(HexagonISD::VALIGN, dl, VecTy,
  1082:                        {HalfV, SplatV, DAG.getConstant(HwLen/2, dl, MVT::i32)});
  1083:   }
  1084:   SDValue HalfV0 = HalfV;
  1085:   SDValue HalfV1 = HalfV;
  1086: 
  1087:   // Construct two halves in parallel, then or them together. Rn and Rm count
  1088:   // number of rotations needed before the next element. One last rotation is
  1089:   // performed post-loop to position the last element.
  1090:   int Rn = 0, Rm = 0;
  1091:   SDValue Sn, Sm;
  1092:   SDValue N = HalfV0;
  1093:   SDValue M = HalfV1;
  1094:   for (unsigned i = 0; i != NumWords/2; ++i) {
  1095:     // Rotate by element count since last insertion.
  1096:     if (Words[i] != Words[n] || VecHist[n] <= 1) {
  1097:       Sn = DAG.getConstant(Rn, dl, MVT::i32);
  1098:       HalfV0 = DAG.getNode(HexagonISD::VROR, dl, VecTy, {N, Sn});
  1099:       N = DAG.getNode(HexagonISD::VINSERTW0, dl, VecTy,
  1100:                       {HalfV0, Words[i]});
  1101:       Rn = 0;
  1102:     }
  1103:     if (Words[i+NumWords/2] != Words[n] || VecHist[n] <= 1) {
  1104:       Sm = DAG.getConstant(Rm, dl, MVT::i32);
  1105:       HalfV1 = DAG.getNode(HexagonISD::VROR, dl, VecTy, {M, Sm});
  1106:       M = DAG.getNode(HexagonISD::VINSERTW0, dl, VecTy,
  1107:                       {HalfV1, Words[i+NumWords/2]});
  1108:       Rm = 0;
  1109:     }
  1110:     Rn += 4;
  1111:     Rm += 4;
  1112:   }
  1113:   // Perform last rotation.
  1114:   Sn = DAG.getConstant(Rn+HwLen/2, dl, MVT::i32);
  1115:   Sm = DAG.getConstant(Rm, dl, MVT::i32);
  1116:   HalfV0 = DAG.getNode(HexagonISD::VROR, dl, VecTy, {N, Sn});
  1117:   HalfV1 = DAG.getNode(HexagonISD::VROR, dl, VecTy, {M, Sm});
  1118: 
  1119:   SDValue T0 = DAG.getBitcast(tyVector(VecTy, MVT::i32), HalfV0);
  1120:   SDValue T1 = DAG.getBitcast(tyVector(VecTy, MVT::i32), HalfV1);
  1121: 
  1122:   SDValue DstV = DAG.getNode(ISD::OR, dl, ty(T0), {T0, T1});
  1123: 
  1124:   SDValue OutV =
  1125:       DAG.getBitcast(tyVector(ty(DstV), VecTy.getVectorElementType()), DstV);
  1126:   return OutV;
  1127: }
  1128: 
  1129: SDValue
  1130: HexagonTargetLowering::createHvxPrefixPred(SDValue PredV, const SDLoc &dl,
  1131:       unsigned BitBytes, bool ZeroFill, SelectionDAG &DAG) const {
  1132:   MVT PredTy = ty(PredV);
  1133:   unsigned HwLen = Subtarget.getVectorLength();
  1134:   MVT ByteTy = MVT::getVectorVT(MVT::i8, HwLen);
  1135: 
  1136:   if (Subtarget.isHVXVectorType(PredTy, true)) {
  1137:     // Move the vector predicate SubV to a vector register, and scale it
  1138:     // down to match the representation (bytes per type element) that VecV
  1139:     // uses. The scaling down will pick every 2nd or 4th (every Scale-th
  1140:     // in general) element and put them at the front of the resulting
  1141:     // vector. This subvector will then be inserted into the Q2V of VecV.
  1142:     // To avoid having an operation that generates an illegal type (short
  1143:     // vector), generate a full size vector.
  1144:     //
  1145:     SDValue T = DAG.getNode(HexagonISD::Q2V, dl, ByteTy, PredV);
  1146:     SmallVector<int,128> Mask(HwLen);
  1147:     // Scale = BitBytes(PredV) / Given BitBytes.
  1148:     unsigned Scale = HwLen / (PredTy.getVectorNumElements() * BitBytes);
  1149:     unsigned BlockLen = PredTy.getVectorNumElements() * BitBytes;
  1150: 
  1151:     for (unsigned i = 0; i != HwLen; ++i) {
  1152:       unsigned Num = i % Scale;
  1153:       unsigned Off = i / Scale;
  1154:       Mask[BlockLen*Num + Off] = i;
  1155:     }
  1156:     SDValue S = DAG.getVectorShuffle(ByteTy, dl, T, DAG.getUNDEF(ByteTy), Mask);
  1157:     if (!ZeroFill)
  1158:       return S;
  1159:     // Fill the bytes beyond BlockLen with 0s.
  1160:     // V6_pred_scalar2 cannot fill the entire predicate, so it only works
  1161:     // when BlockLen < HwLen.
  1162:     assert(BlockLen < HwLen && "vsetq(v1) prerequisite");
  1163:     MVT BoolTy = MVT::getVectorVT(MVT::i1, HwLen);
  1164:     SDValue Q = getInstr(Hexagon::V6_pred_scalar2, dl, BoolTy,
  1165:                          {DAG.getConstant(BlockLen, dl, MVT::i32)}, DAG);
  1166:     SDValue M = DAG.getNode(HexagonISD::Q2V, dl, ByteTy, Q);
  1167:     return DAG.getNode(ISD::AND, dl, ByteTy, S, M);
  1168:   }
  1169: 
  1170:   // Make sure that this is a valid scalar predicate.
  1171:   assert(PredTy == MVT::v2i1 || PredTy == MVT::v4i1 || PredTy == MVT::v8i1);
  1172: 
  1173:   unsigned Bytes = 8 / PredTy.getVectorNumElements();
  1174:   SmallVector<SDValue,4> Words[2];
  1175:   unsigned IdxW = 0;
  1176: 
  1177:   SDValue W0 = isUndef(PredV)
  1178:                   ? DAG.getUNDEF(MVT::i64)
  1179:                   : DAG.getNode(HexagonISD::P2D, dl, MVT::i64, PredV);
  1180:   Words[IdxW].push_back(HiHalf(W0, DAG));
  1181:   Words[IdxW].push_back(LoHalf(W0, DAG));
  1182: 
  1183:   while (Bytes < BitBytes) {
  1184:     IdxW ^= 1;
  1185:     Words[IdxW].clear();
  1186: 
  1187:     if (Bytes < 4) {
  1188:       for (const SDValue &W : Words[IdxW ^ 1]) {
  1189:         SDValue T = expandPredicate(W, dl, DAG);
  1190:         Words[IdxW].push_back(HiHalf(T, DAG));
  1191:         Words[IdxW].push_back(LoHalf(T, DAG));
  1192:       }
  1193:     } else {
  1194:       for (const SDValue &W : Words[IdxW ^ 1]) {
  1195:         Words[IdxW].push_back(W);
  1196:         Words[IdxW].push_back(W);
  1197:       }
  1198:     }
  1199:     Bytes *= 2;
  1200:   }
```
- EN: It declares or implements routines such as push_back, getOperand, dyn_cast<ConstantSDNode>, getSExtValue, ... (23 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonISD, HexagonTargetLowering, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 push_back, getOperand, dyn_cast<ConstantSDNode>, getSExtValue, ... (23 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonISD, HexagonTargetLowering，说明了它与同级后端组件的连接关系。

### Lines 1201-1400 / 第 1201-1400 行

```cpp
  1201: 
  1202:   assert(Bytes == BitBytes);
  1203:   SDValue Vec = ZeroFill ? getZero(dl, ByteTy, DAG) : DAG.getUNDEF(ByteTy);
  1204:   SDValue S4 = DAG.getConstant(HwLen-4, dl, MVT::i32);
  1205:   for (const SDValue &W : Words[IdxW]) {
  1206:     Vec = DAG.getNode(HexagonISD::VROR, dl, ByteTy, Vec, S4);
  1207:     Vec = DAG.getNode(HexagonISD::VINSERTW0, dl, ByteTy, Vec, W);
  1208:   }
  1209: 
  1210:   return Vec;
  1211: }
  1212: 
  1213: SDValue
  1214: HexagonTargetLowering::buildHvxVectorPred(ArrayRef<SDValue> Values,
  1215:                                           const SDLoc &dl, MVT VecTy,
  1216:                                           SelectionDAG &DAG) const {
  1217:   // Construct a vector V of bytes, such that a comparison V >u 0 would
  1218:   // produce the required vector predicate.
  1219:   unsigned VecLen = Values.size();
  1220:   unsigned HwLen = Subtarget.getVectorLength();
  1221:   assert(VecLen <= HwLen || VecLen == 8*HwLen);
  1222:   SmallVector<SDValue,128> Bytes;
  1223:   bool AllT = true, AllF = true;
  1224: 
  1225:   auto IsTrue = [] (SDValue V) {
  1226:     if (const auto *N = dyn_cast<ConstantSDNode>(V.getNode()))
  1227:       return !N->isZero();
  1228:     return false;
  1229:   };
  1230:   auto IsFalse = [] (SDValue V) {
  1231:     if (const auto *N = dyn_cast<ConstantSDNode>(V.getNode()))
  1232:       return N->isZero();
  1233:     return false;
  1234:   };
  1235: 
  1236:   if (VecLen <= HwLen) {
  1237:     // In the hardware, each bit of a vector predicate corresponds to a byte
  1238:     // of a vector register. Calculate how many bytes does a bit of VecTy
  1239:     // correspond to.
  1240:     assert(HwLen % VecLen == 0);
  1241:     unsigned BitBytes = HwLen / VecLen;
  1242:     for (SDValue V : Values) {
  1243:       AllT &= IsTrue(V);
  1244:       AllF &= IsFalse(V);
  1245: 
  1246:       SDValue Ext = !V.isUndef() ? DAG.getZExtOrTrunc(V, dl, MVT::i8)
  1247:                                  : DAG.getUNDEF(MVT::i8);
  1248:       for (unsigned B = 0; B != BitBytes; ++B)
  1249:         Bytes.push_back(Ext);
  1250:     }
  1251:   } else {
  1252:     // There are as many i1 values, as there are bits in a vector register.
  1253:     // Divide the values into groups of 8 and check that each group consists
  1254:     // of the same value (ignoring undefs).
  1255:     for (unsigned I = 0; I != VecLen; I += 8) {
  1256:       unsigned B = 0;
  1257:       // Find the first non-undef value in this group.
  1258:       for (; B != 8; ++B) {
  1259:         if (!Values[I+B].isUndef())
  1260:           break;
  1261:       }
  1262:       SDValue F = Values[I+B];
  1263:       AllT &= IsTrue(F);
  1264:       AllF &= IsFalse(F);
  1265: 
  1266:       SDValue Ext = (B < 8) ? DAG.getZExtOrTrunc(F, dl, MVT::i8)
  1267:                             : DAG.getUNDEF(MVT::i8);
  1268:       Bytes.push_back(Ext);
  1269:       // Verify that the rest of values in the group are the same as the
  1270:       // first.
  1271:       for (; B != 8; ++B)
  1272:         assert(Values[I+B].isUndef() || Values[I+B] == F);
  1273:     }
  1274:   }
  1275: 
  1276:   if (AllT)
  1277:     return DAG.getNode(HexagonISD::QTRUE, dl, VecTy);
  1278:   if (AllF)
  1279:     return DAG.getNode(HexagonISD::QFALSE, dl, VecTy);
  1280: 
  1281:   MVT ByteTy = MVT::getVectorVT(MVT::i8, HwLen);
  1282:   SDValue ByteVec = buildHvxVectorReg(Bytes, dl, ByteTy, DAG);
  1283:   return DAG.getNode(HexagonISD::V2Q, dl, VecTy, ByteVec);
  1284: }
  1285: 
  1286: SDValue
  1287: HexagonTargetLowering::extractHvxElementReg(SDValue VecV, SDValue IdxV,
  1288:       const SDLoc &dl, MVT ResTy, SelectionDAG &DAG) const {
  1289:   MVT ElemTy = ty(VecV).getVectorElementType();
  1290: 
  1291:   unsigned ElemWidth = ElemTy.getSizeInBits();
  1292:   assert(ElemWidth >= 8 && ElemWidth <= 32);
  1293:   (void)ElemWidth;
  1294: 
  1295:   SDValue ByteIdx = convertToByteIndex(IdxV, ElemTy, DAG);
  1296:   SDValue ExWord = DAG.getNode(HexagonISD::VEXTRACTW, dl, MVT::i32,
  1297:                                {VecV, ByteIdx});
  1298:   if (ElemTy == MVT::i32)
  1299:     return ExWord;
  1300: 
  1301:   // Have an extracted word, need to extract the smaller element out of it.
  1302:   // 1. Extract the bits of (the original) IdxV that correspond to the index
  1303:   //    of the desired element in the 32-bit word.
  1304:   SDValue SubIdx = getIndexInWord32(IdxV, ElemTy, DAG);
  1305:   // 2. Extract the element from the word.
  1306:   SDValue ExVec = DAG.getBitcast(tyVector(ty(ExWord), ElemTy), ExWord);
  1307:   return extractVector(ExVec, SubIdx, dl, ElemTy, MVT::i32, DAG);
  1308: }
  1309: 
  1310: SDValue
  1311: HexagonTargetLowering::extractHvxElementPred(SDValue VecV, SDValue IdxV,
  1312:       const SDLoc &dl, MVT ResTy, SelectionDAG &DAG) const {
  1313:   // Implement other return types if necessary.
  1314:   assert(ResTy == MVT::i1);
  1315: 
  1316:   unsigned HwLen = Subtarget.getVectorLength();
  1317:   MVT ByteTy = MVT::getVectorVT(MVT::i8, HwLen);
  1318:   SDValue ByteVec = DAG.getNode(HexagonISD::Q2V, dl, ByteTy, VecV);
  1319: 
  1320:   unsigned Scale = HwLen / ty(VecV).getVectorNumElements();
  1321:   SDValue ScV = DAG.getConstant(Scale, dl, MVT::i32);
  1322:   IdxV = DAG.getNode(ISD::MUL, dl, MVT::i32, IdxV, ScV);
  1323: 
  1324:   SDValue ExtB = extractHvxElementReg(ByteVec, IdxV, dl, MVT::i32, DAG);
  1325:   SDValue Zero = DAG.getTargetConstant(0, dl, MVT::i32);
  1326:   return getInstr(Hexagon::C2_cmpgtui, dl, MVT::i1, {ExtB, Zero}, DAG);
  1327: }
  1328: 
  1329: SDValue
  1330: HexagonTargetLowering::insertHvxElementReg(SDValue VecV, SDValue IdxV,
  1331:       SDValue ValV, const SDLoc &dl, SelectionDAG &DAG) const {
  1332:   MVT ElemTy = ty(VecV).getVectorElementType();
  1333: 
  1334:   unsigned ElemWidth = ElemTy.getSizeInBits();
  1335:   assert(ElemWidth >= 8 && ElemWidth <= 32);
  1336:   (void)ElemWidth;
  1337: 
  1338:   auto InsertWord = [&DAG,&dl,this] (SDValue VecV, SDValue ValV,
  1339:                                      SDValue ByteIdxV) {
  1340:     MVT VecTy = ty(VecV);
  1341:     unsigned HwLen = Subtarget.getVectorLength();
  1342:     SDValue MaskV =
  1343:         DAG.getNode(ISD::AND, dl, MVT::i32,
  1344:                     {ByteIdxV, DAG.getSignedConstant(-4, dl, MVT::i32)});
  1345:     SDValue RotV = DAG.getNode(HexagonISD::VROR, dl, VecTy, {VecV, MaskV});
  1346:     SDValue InsV = DAG.getNode(HexagonISD::VINSERTW0, dl, VecTy, {RotV, ValV});
  1347:     SDValue SubV = DAG.getNode(ISD::SUB, dl, MVT::i32,
  1348:                                {DAG.getConstant(HwLen, dl, MVT::i32), MaskV});
  1349:     SDValue TorV = DAG.getNode(HexagonISD::VROR, dl, VecTy, {InsV, SubV});
  1350:     return TorV;
  1351:   };
  1352: 
  1353:   SDValue ByteIdx = convertToByteIndex(IdxV, ElemTy, DAG);
  1354:   if (ElemTy == MVT::i32)
  1355:     return InsertWord(VecV, ValV, ByteIdx);
  1356: 
  1357:   // If this is not inserting a 32-bit word, convert it into such a thing.
  1358:   // 1. Extract the existing word from the target vector.
  1359:   SDValue WordIdx = DAG.getNode(ISD::SRL, dl, MVT::i32,
  1360:                                 {ByteIdx, DAG.getConstant(2, dl, MVT::i32)});
  1361:   SDValue Ext = extractHvxElementReg(opCastElem(VecV, MVT::i32, DAG), WordIdx,
  1362:                                      dl, MVT::i32, DAG);
  1363: 
  1364:   // 2. Treating the extracted word as a 32-bit vector, insert the given
  1365:   //    value into it.
  1366:   SDValue SubIdx = getIndexInWord32(IdxV, ElemTy, DAG);
  1367:   MVT SubVecTy = tyVector(ty(Ext), ElemTy);
  1368:   SDValue Ins = insertVector(DAG.getBitcast(SubVecTy, Ext),
  1369:                              ValV, SubIdx, dl, ElemTy, DAG);
  1370: 
  1371:   // 3. Insert the 32-bit word back into the original vector.
  1372:   return InsertWord(VecV, Ins, ByteIdx);
  1373: }
  1374: 
  1375: SDValue
  1376: HexagonTargetLowering::insertHvxElementPred(SDValue VecV, SDValue IdxV,
  1377:       SDValue ValV, const SDLoc &dl, SelectionDAG &DAG) const {
  1378:   unsigned HwLen = Subtarget.getVectorLength();
  1379:   MVT ByteTy = MVT::getVectorVT(MVT::i8, HwLen);
  1380:   SDValue ByteVec = DAG.getNode(HexagonISD::Q2V, dl, ByteTy, VecV);
  1381: 
  1382:   unsigned Scale = HwLen / ty(VecV).getVectorNumElements();
  1383:   SDValue ScV = DAG.getConstant(Scale, dl, MVT::i32);
  1384:   IdxV = DAG.getNode(ISD::MUL, dl, MVT::i32, IdxV, ScV);
  1385:   ValV = DAG.getNode(ISD::SIGN_EXTEND, dl, MVT::i32, ValV);
  1386: 
  1387:   SDValue InsV = insertHvxElementReg(ByteVec, IdxV, ValV, dl, DAG);
  1388:   return DAG.getNode(HexagonISD::V2Q, dl, ty(VecV), InsV);
  1389: }
  1390: 
  1391: SDValue
  1392: HexagonTargetLowering::extractHvxSubvectorReg(SDValue OrigOp, SDValue VecV,
  1393:       SDValue IdxV, const SDLoc &dl, MVT ResTy, SelectionDAG &DAG) const {
  1394:   MVT VecTy = ty(VecV);
  1395:   unsigned HwLen = Subtarget.getVectorLength();
  1396:   unsigned Idx = IdxV.getNode()->getAsZExtVal();
  1397:   MVT ElemTy = VecTy.getVectorElementType();
  1398:   unsigned ElemWidth = ElemTy.getSizeInBits();
  1399: 
  1400:   // If the source vector is a vector pair, get the single vector containing
```
- EN: It declares or implements routines such as assert, getZero, getConstant, getNode, ... (33 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonISD, HexagonTargetLowering, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 assert, getZero, getConstant, getNode, ... (33 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonISD, HexagonTargetLowering，说明了它与同级后端组件的连接关系。

### Lines 1401-1600 / 第 1401-1600 行

```cpp
  1401:   // the subvector of interest. The subvector will never overlap two single
  1402:   // vectors.
  1403:   if (isHvxPairTy(VecTy)) {
  1404:     unsigned SubIdx = Hexagon::vsub_lo;
  1405:     if (Idx * ElemWidth >= 8 * HwLen) {
  1406:       SubIdx = Hexagon::vsub_hi;
  1407:       Idx -= VecTy.getVectorNumElements() / 2;
  1408:     }
  1409: 
  1410:     VecTy = typeSplit(VecTy).first;
  1411:     VecV = DAG.getTargetExtractSubreg(SubIdx, dl, VecTy, VecV);
  1412:     if (VecTy == ResTy)
  1413:       return VecV;
  1414:   }
  1415: 
  1416:   // The only meaningful subvectors of a single HVX vector are those that
  1417:   // fit in a scalar register.
  1418:   assert(ResTy.getSizeInBits() == 32 || ResTy.getSizeInBits() == 64);
  1419: 
  1420:   MVT WordTy = tyVector(VecTy, MVT::i32);
  1421:   SDValue WordVec = DAG.getBitcast(WordTy, VecV);
  1422:   unsigned WordIdx = (Idx*ElemWidth) / 32;
  1423: 
  1424:   SDValue W0Idx = DAG.getConstant(WordIdx, dl, MVT::i32);
  1425:   SDValue W0 = extractHvxElementReg(WordVec, W0Idx, dl, MVT::i32, DAG);
  1426:   if (ResTy.getSizeInBits() == 32)
  1427:     return DAG.getBitcast(ResTy, W0);
  1428: 
  1429:   SDValue W1Idx = DAG.getConstant(WordIdx+1, dl, MVT::i32);
  1430:   SDValue W1 = extractHvxElementReg(WordVec, W1Idx, dl, MVT::i32, DAG);
  1431:   SDValue WW = getCombine(W1, W0, dl, MVT::i64, DAG);
  1432:   return DAG.getBitcast(ResTy, WW);
  1433: }
  1434: 
  1435: SDValue
  1436: HexagonTargetLowering::extractHvxSubvectorPred(SDValue VecV, SDValue IdxV,
  1437:       const SDLoc &dl, MVT ResTy, SelectionDAG &DAG) const {
  1438:   MVT VecTy = ty(VecV);
  1439:   unsigned HwLen = Subtarget.getVectorLength();
  1440:   MVT ByteTy = MVT::getVectorVT(MVT::i8, HwLen);
  1441:   SDValue ByteVec = DAG.getNode(HexagonISD::Q2V, dl, ByteTy, VecV);
  1442:   // IdxV is required to be a constant.
  1443:   unsigned Idx = IdxV.getNode()->getAsZExtVal();
  1444: 
  1445:   unsigned ResLen = ResTy.getVectorNumElements();
  1446:   unsigned BitBytes = HwLen / VecTy.getVectorNumElements();
  1447:   unsigned Offset = Idx * BitBytes;
  1448:   SDValue Undef = DAG.getUNDEF(ByteTy);
  1449:   SmallVector<int,128> Mask;
  1450: 
  1451:   if (Subtarget.isHVXVectorType(ResTy, true)) {
  1452:     // Converting between two vector predicates. Since the result is shorter
  1453:     // than the source, it will correspond to a vector predicate with the
  1454:     // relevant bits replicated. The replication count is the ratio of the
  1455:     // source and target vector lengths.
  1456:     unsigned Rep = VecTy.getVectorNumElements() / ResLen;
  1457:     assert(isPowerOf2_32(Rep) && HwLen % Rep == 0);
  1458:     for (unsigned i = 0; i != HwLen/Rep; ++i) {
  1459:       for (unsigned j = 0; j != Rep; ++j)
  1460:         Mask.push_back(i + Offset);
  1461:     }
  1462:     SDValue ShuffV = DAG.getVectorShuffle(ByteTy, dl, ByteVec, Undef, Mask);
  1463:     return DAG.getNode(HexagonISD::V2Q, dl, ResTy, ShuffV);
  1464:   }
  1465: 
  1466:   // Converting between a vector predicate and a scalar predicate. In the
  1467:   // vector predicate, a group of BitBytes bits will correspond to a single
  1468:   // i1 element of the source vector type. Those bits will all have the same
  1469:   // value. The same will be true for ByteVec, where each byte corresponds
  1470:   // to a bit in the vector predicate.
  1471:   // The algorithm is to traverse the ByteVec, going over the i1 values from
  1472:   // the source vector, and generate the corresponding representation in an
  1473:   // 8-byte vector. To avoid repeated extracts from ByteVec, shuffle the
  1474:   // elements so that the interesting 8 bytes will be in the low end of the
  1475:   // vector.
  1476:   unsigned Rep = 8 / ResLen;
  1477:   // Make sure the output fill the entire vector register, so repeat the
  1478:   // 8-byte groups as many times as necessary.
  1479:   for (unsigned r = 0; r != HwLen / 8; ++r) {
  1480:     // This will generate the indexes of the 8 interesting bytes.
  1481:     for (unsigned i = 0; i != ResLen; ++i) {
  1482:       for (unsigned j = 0; j != Rep; ++j)
  1483:         Mask.push_back(Offset + i*BitBytes);
  1484:     }
  1485:   }
  1486: 
  1487:   SDValue Zero = getZero(dl, MVT::i32, DAG);
  1488:   SDValue ShuffV = DAG.getVectorShuffle(ByteTy, dl, ByteVec, Undef, Mask);
  1489:   // Combine the two low words from ShuffV into a v8i8, and byte-compare
  1490:   // them against 0.
  1491:   SDValue W0 = DAG.getNode(HexagonISD::VEXTRACTW, dl, MVT::i32, {ShuffV, Zero});
  1492:   SDValue W1 = DAG.getNode(HexagonISD::VEXTRACTW, dl, MVT::i32,
  1493:                            {ShuffV, DAG.getConstant(4, dl, MVT::i32)});
  1494:   SDValue Vec64 = getCombine(W1, W0, dl, MVT::v8i8, DAG);
  1495:   return getInstr(Hexagon::A4_vcmpbgtui, dl, ResTy,
  1496:                   {Vec64, DAG.getTargetConstant(0, dl, MVT::i32)}, DAG);
  1497: }
  1498: 
  1499: SDValue
  1500: HexagonTargetLowering::insertHvxSubvectorReg(SDValue VecV, SDValue SubV,
  1501:       SDValue IdxV, const SDLoc &dl, SelectionDAG &DAG) const {
  1502:   MVT VecTy = ty(VecV);
  1503:   MVT SubTy = ty(SubV);
  1504:   unsigned HwLen = Subtarget.getVectorLength();
  1505:   MVT ElemTy = VecTy.getVectorElementType();
  1506:   unsigned ElemWidth = ElemTy.getSizeInBits();
  1507: 
  1508:   bool IsPair = isHvxPairTy(VecTy);
  1509:   MVT SingleTy = MVT::getVectorVT(ElemTy, (8*HwLen)/ElemWidth);
  1510:   // The two single vectors that VecV consists of, if it's a pair.
  1511:   SDValue V0, V1;
  1512:   SDValue SingleV = VecV;
  1513:   SDValue PickHi;
  1514: 
  1515:   if (IsPair) {
  1516:     V0 = LoHalf(VecV, DAG);
  1517:     V1 = HiHalf(VecV, DAG);
  1518: 
  1519:     SDValue HalfV = DAG.getConstant(SingleTy.getVectorNumElements(),
  1520:                                     dl, MVT::i32);
  1521:     PickHi = DAG.getSetCC(dl, MVT::i1, IdxV, HalfV, ISD::SETUGT);
  1522:     if (isHvxSingleTy(SubTy)) {
  1523:       if (const auto *CN = dyn_cast<const ConstantSDNode>(IdxV.getNode())) {
  1524:         unsigned Idx = CN->getZExtValue();
  1525:         assert(Idx == 0 || Idx == VecTy.getVectorNumElements()/2);
  1526:         unsigned SubIdx = (Idx == 0) ? Hexagon::vsub_lo : Hexagon::vsub_hi;
  1527:         return DAG.getTargetInsertSubreg(SubIdx, dl, VecTy, VecV, SubV);
  1528:       }
  1529:       // If IdxV is not a constant, generate the two variants: with the
  1530:       // SubV as the high and as the low subregister, and select the right
  1531:       // pair based on the IdxV.
  1532:       SDValue InLo = DAG.getNode(ISD::CONCAT_VECTORS, dl, VecTy, {SubV, V1});
  1533:       SDValue InHi = DAG.getNode(ISD::CONCAT_VECTORS, dl, VecTy, {V0, SubV});
  1534:       return DAG.getNode(ISD::SELECT, dl, VecTy, PickHi, InHi, InLo);
  1535:     }
  1536:     // The subvector being inserted must be entirely contained in one of
  1537:     // the vectors V0 or V1. Set SingleV to the correct one, and update
  1538:     // IdxV to be the index relative to the beginning of that vector.
  1539:     SDValue S = DAG.getNode(ISD::SUB, dl, MVT::i32, IdxV, HalfV);
  1540:     IdxV = DAG.getNode(ISD::SELECT, dl, MVT::i32, PickHi, S, IdxV);
  1541:     SingleV = DAG.getNode(ISD::SELECT, dl, SingleTy, PickHi, V1, V0);
  1542:   }
  1543: 
  1544:   // The only meaningful subvectors of a single HVX vector are those that
  1545:   // fit in a scalar register.
  1546:   assert(SubTy.getSizeInBits() == 32 || SubTy.getSizeInBits() == 64);
  1547:   // Convert IdxV to be index in bytes.
  1548:   auto *IdxN = dyn_cast<ConstantSDNode>(IdxV.getNode());
  1549:   if (!IdxN || !IdxN->isZero()) {
  1550:     IdxV = DAG.getNode(ISD::MUL, dl, MVT::i32, IdxV,
  1551:                        DAG.getConstant(ElemWidth/8, dl, MVT::i32));
  1552:     SingleV = DAG.getNode(HexagonISD::VROR, dl, SingleTy, SingleV, IdxV);
  1553:   }
  1554:   // When inserting a single word, the rotation back to the original position
  1555:   // would be by HwLen-Idx, but if two words are inserted, it will need to be
  1556:   // by (HwLen-4)-Idx.
  1557:   unsigned RolBase = HwLen;
  1558:   if (SubTy.getSizeInBits() == 32) {
  1559:     SDValue V = DAG.getBitcast(MVT::i32, SubV);
  1560:     SingleV = DAG.getNode(HexagonISD::VINSERTW0, dl, SingleTy, SingleV, V);
  1561:   } else {
  1562:     SDValue V = DAG.getBitcast(MVT::i64, SubV);
  1563:     SDValue R0 = LoHalf(V, DAG);
  1564:     SDValue R1 = HiHalf(V, DAG);
  1565:     SingleV = DAG.getNode(HexagonISD::VINSERTW0, dl, SingleTy, SingleV, R0);
  1566:     SingleV = DAG.getNode(HexagonISD::VROR, dl, SingleTy, SingleV,
  1567:                           DAG.getConstant(4, dl, MVT::i32));
  1568:     SingleV = DAG.getNode(HexagonISD::VINSERTW0, dl, SingleTy, SingleV, R1);
  1569:     RolBase = HwLen-4;
  1570:   }
  1571:   // If the vector wasn't ror'ed, don't ror it back.
  1572:   if (RolBase != 4 || !IdxN || !IdxN->isZero()) {
  1573:     SDValue RolV = DAG.getNode(ISD::SUB, dl, MVT::i32,
  1574:                                DAG.getConstant(RolBase, dl, MVT::i32), IdxV);
  1575:     SingleV = DAG.getNode(HexagonISD::VROR, dl, SingleTy, SingleV, RolV);
  1576:   }
  1577: 
  1578:   if (IsPair) {
  1579:     SDValue InLo = DAG.getNode(ISD::CONCAT_VECTORS, dl, VecTy, {SingleV, V1});
  1580:     SDValue InHi = DAG.getNode(ISD::CONCAT_VECTORS, dl, VecTy, {V0, SingleV});
  1581:     return DAG.getNode(ISD::SELECT, dl, VecTy, PickHi, InHi, InLo);
  1582:   }
  1583:   return SingleV;
  1584: }
  1585: 
  1586: SDValue
  1587: HexagonTargetLowering::insertHvxSubvectorPred(SDValue VecV, SDValue SubV,
  1588:       SDValue IdxV, const SDLoc &dl, SelectionDAG &DAG) const {
  1589:   MVT VecTy = ty(VecV);
  1590:   MVT SubTy = ty(SubV);
  1591:   assert(Subtarget.isHVXVectorType(VecTy, true));
  1592:   // VecV is an HVX vector predicate. SubV may be either an HVX vector
  1593:   // predicate as well, or it can be a scalar predicate.
  1594: 
  1595:   unsigned VecLen = VecTy.getVectorNumElements();
  1596:   unsigned HwLen = Subtarget.getVectorLength();
  1597:   assert(HwLen % VecLen == 0 && "Unexpected vector type");
  1598: 
  1599:   unsigned Scale = VecLen / SubTy.getVectorNumElements();
  1600:   unsigned BitBytes = HwLen / VecLen;
```
- EN: It declares or implements routines such as getTargetExtractSubreg, assert, tyVector, getBitcast, ... (28 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getTargetExtractSubreg, assert, tyVector, getBitcast, ... (28 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 1601-1800 / 第 1601-1800 行

```cpp
  1601:   unsigned BlockLen = HwLen / Scale;
  1602: 
  1603:   MVT ByteTy = MVT::getVectorVT(MVT::i8, HwLen);
  1604:   SDValue ByteVec = DAG.getNode(HexagonISD::Q2V, dl, ByteTy, VecV);
  1605:   SDValue ByteSub = createHvxPrefixPred(SubV, dl, BitBytes, false, DAG);
  1606:   SDValue ByteIdx;
  1607: 
  1608:   auto *IdxN = dyn_cast<ConstantSDNode>(IdxV.getNode());
  1609:   if (!IdxN || !IdxN->isZero()) {
  1610:     ByteIdx = DAG.getNode(ISD::MUL, dl, MVT::i32, IdxV,
  1611:                           DAG.getConstant(BitBytes, dl, MVT::i32));
  1612:     ByteVec = DAG.getNode(HexagonISD::VROR, dl, ByteTy, ByteVec, ByteIdx);
  1613:   }
  1614: 
  1615:   // ByteVec is the target vector VecV rotated in such a way that the
  1616:   // subvector should be inserted at index 0. Generate a predicate mask
  1617:   // and use vmux to do the insertion.
  1618:   assert(BlockLen < HwLen && "vsetq(v1) prerequisite");
  1619:   MVT BoolTy = MVT::getVectorVT(MVT::i1, HwLen);
  1620:   SDValue Q = getInstr(Hexagon::V6_pred_scalar2, dl, BoolTy,
  1621:                        {DAG.getConstant(BlockLen, dl, MVT::i32)}, DAG);
  1622:   ByteVec = getInstr(Hexagon::V6_vmux, dl, ByteTy, {Q, ByteSub, ByteVec}, DAG);
  1623:   // Rotate ByteVec back, and convert to a vector predicate.
  1624:   if (!IdxN || !IdxN->isZero()) {
  1625:     SDValue HwLenV = DAG.getConstant(HwLen, dl, MVT::i32);
  1626:     SDValue ByteXdi = DAG.getNode(ISD::SUB, dl, MVT::i32, HwLenV, ByteIdx);
  1627:     ByteVec = DAG.getNode(HexagonISD::VROR, dl, ByteTy, ByteVec, ByteXdi);
  1628:   }
  1629:   return DAG.getNode(HexagonISD::V2Q, dl, VecTy, ByteVec);
  1630: }
  1631: 
  1632: SDValue
  1633: HexagonTargetLowering::extendHvxVectorPred(SDValue VecV, const SDLoc &dl,
  1634:       MVT ResTy, bool ZeroExt, SelectionDAG &DAG) const {
  1635:   // Sign- and any-extending of a vector predicate to a vector register is
  1636:   // equivalent to Q2V. For zero-extensions, generate a vmux between 0 and
  1637:   // a vector of 1s (where the 1s are of type matching the vector type).
  1638:   assert(Subtarget.isHVXVectorType(ResTy));
  1639:   if (!ZeroExt)
  1640:     return DAG.getNode(HexagonISD::Q2V, dl, ResTy, VecV);
  1641: 
  1642:   assert(ty(VecV).getVectorNumElements() == ResTy.getVectorNumElements());
  1643:   SDValue True = DAG.getNode(ISD::SPLAT_VECTOR, dl, ResTy,
  1644:                              DAG.getConstant(1, dl, MVT::i32));
  1645:   SDValue False = getZero(dl, ResTy, DAG);
  1646:   return DAG.getSelect(dl, ResTy, VecV, True, False);
  1647: }
  1648: 
  1649: SDValue
  1650: HexagonTargetLowering::compressHvxPred(SDValue VecQ, const SDLoc &dl,
  1651:       MVT ResTy, SelectionDAG &DAG) const {
  1652:   // Given a predicate register VecQ, transfer bits VecQ[0..HwLen-1]
  1653:   // (i.e. the entire predicate register) to bits [0..HwLen-1] of a
  1654:   // vector register. The remaining bits of the vector register are
  1655:   // unspecified.
  1656: 
  1657:   MachineFunction &MF = DAG.getMachineFunction();
  1658:   unsigned HwLen = Subtarget.getVectorLength();
  1659:   MVT ByteTy = MVT::getVectorVT(MVT::i8, HwLen);
  1660:   MVT PredTy = ty(VecQ);
  1661:   unsigned PredLen = PredTy.getVectorNumElements();
  1662:   assert(HwLen % PredLen == 0);
  1663:   MVT VecTy = MVT::getVectorVT(MVT::getIntegerVT(8*HwLen/PredLen), PredLen);
  1664: 
  1665:   Type *Int8Ty = Type::getInt8Ty(*DAG.getContext());
  1666:   SmallVector<Constant*, 128> Tmp;
  1667:   // Create an array of bytes (hex): 01,02,04,08,10,20,40,80, 01,02,04,08,...
  1668:   // These are bytes with the LSB rotated left with respect to their index.
  1669:   for (unsigned i = 0; i != HwLen/8; ++i) {
  1670:     for (unsigned j = 0; j != 8; ++j)
  1671:       Tmp.push_back(ConstantInt::get(Int8Ty, 1ull << j));
  1672:   }
  1673:   Constant *CV = ConstantVector::get(Tmp);
  1674:   Align Alignment(HwLen);
  1675:   SDValue CP = LowerConstantPool(
  1676:       DAG.getConstantPool(CV, getPointerTy(DAG.getDataLayout()), Alignment),
  1677:       DAG);
  1678:   SDValue Bytes =
  1679:       DAG.getLoad(ByteTy, dl, DAG.getEntryNode(), CP,
  1680:                   MachinePointerInfo::getConstantPool(MF), Alignment);
  1681: 
  1682:   // Select the bytes that correspond to true bits in the vector predicate.
  1683:   SDValue Sel = DAG.getSelect(dl, VecTy, VecQ, DAG.getBitcast(VecTy, Bytes),
  1684:       getZero(dl, VecTy, DAG));
  1685:   // Calculate the OR of all bytes in each group of 8. That will compress
  1686:   // all the individual bits into a single byte.
  1687:   // First, OR groups of 4, via vrmpy with 0x01010101.
  1688:   SDValue All1 =
  1689:       DAG.getSplatBuildVector(MVT::v4i8, dl, DAG.getConstant(1, dl, MVT::i32));
  1690:   SDValue Vrmpy = getInstr(Hexagon::V6_vrmpyub, dl, ByteTy, {Sel, All1}, DAG);
  1691:   // Then rotate the accumulated vector by 4 bytes, and do the final OR.
  1692:   SDValue Rot = getInstr(Hexagon::V6_valignbi, dl, ByteTy,
  1693:       {Vrmpy, Vrmpy, DAG.getTargetConstant(4, dl, MVT::i32)}, DAG);
  1694:   SDValue Vor = DAG.getNode(ISD::OR, dl, ByteTy, {Vrmpy, Rot});
  1695: 
  1696:   // Pick every 8th byte and coalesce them at the beginning of the output.
  1697:   // For symmetry, coalesce every 1+8th byte after that, then every 2+8th
  1698:   // byte and so on.
  1699:   SmallVector<int,128> Mask;
  1700:   for (unsigned i = 0; i != HwLen; ++i)
  1701:     Mask.push_back((8*i) % HwLen + i/(HwLen/8));
  1702:   SDValue Collect =
  1703:       DAG.getVectorShuffle(ByteTy, dl, Vor, DAG.getUNDEF(ByteTy), Mask);
  1704:   return DAG.getBitcast(ResTy, Collect);
  1705: }
  1706: 
  1707: SDValue
  1708: HexagonTargetLowering::resizeToWidth(SDValue VecV, MVT ResTy, bool Signed,
  1709:                                      const SDLoc &dl, SelectionDAG &DAG) const {
  1710:   // Take a vector and resize the element type to match the given type.
  1711:   MVT InpTy = ty(VecV);
  1712:   if (InpTy == ResTy)
  1713:     return VecV;
  1714: 
  1715:   unsigned InpWidth = InpTy.getSizeInBits();
  1716:   unsigned ResWidth = ResTy.getSizeInBits();
  1717: 
  1718:   if (InpTy.isFloatingPoint()) {
  1719:     return InpWidth < ResWidth
  1720:                ? DAG.getNode(ISD::FP_EXTEND, dl, ResTy, VecV)
  1721:                : DAG.getNode(ISD::FP_ROUND, dl, ResTy, VecV,
  1722:                              DAG.getTargetConstant(0, dl, MVT::i32));
  1723:   }
  1724: 
  1725:   assert(InpTy.isInteger());
  1726: 
  1727:   if (InpWidth < ResWidth) {
  1728:     unsigned ExtOpc = Signed ? ISD::SIGN_EXTEND : ISD::ZERO_EXTEND;
  1729:     return DAG.getNode(ExtOpc, dl, ResTy, VecV);
  1730:   } else {
  1731:     unsigned NarOpc = Signed ? HexagonISD::SSAT : HexagonISD::USAT;
  1732:     return DAG.getNode(NarOpc, dl, ResTy, VecV, DAG.getValueType(ResTy));
  1733:   }
  1734: }
  1735: 
  1736: SDValue
  1737: HexagonTargetLowering::extractSubvector(SDValue Vec, MVT SubTy, unsigned SubIdx,
  1738:       SelectionDAG &DAG) const {
  1739:   assert(ty(Vec).getSizeInBits() % SubTy.getSizeInBits() == 0);
  1740: 
  1741:   const SDLoc &dl(Vec);
  1742:   unsigned ElemIdx = SubIdx * SubTy.getVectorNumElements();
  1743:   return DAG.getNode(ISD::EXTRACT_SUBVECTOR, dl, SubTy,
  1744:                      {Vec, DAG.getConstant(ElemIdx, dl, MVT::i32)});
  1745: }
  1746: 
  1747: SDValue
  1748: HexagonTargetLowering::LowerHvxBuildVector(SDValue Op, SelectionDAG &DAG)
  1749:       const {
  1750:   const SDLoc &dl(Op);
  1751:   MVT VecTy = ty(Op);
  1752: 
  1753:   unsigned Size = Op.getNumOperands();
  1754:   SmallVector<SDValue,128> Ops;
  1755:   for (unsigned i = 0; i != Size; ++i)
  1756:     Ops.push_back(Op.getOperand(i));
  1757: 
  1758:   if (VecTy.getVectorElementType() == MVT::i1)
  1759:     return buildHvxVectorPred(Ops, dl, VecTy, DAG);
  1760: 
  1761:   // In case of MVT::f16 BUILD_VECTOR, since MVT::f16 is
  1762:   // not a legal type, just bitcast the node to use i16
  1763:   // types and bitcast the result back to f16
  1764:   if (VecTy.getVectorElementType() == MVT::f16 ||
  1765:       VecTy.getVectorElementType() == MVT::bf16) {
  1766:     SmallVector<SDValue, 64> NewOps;
  1767:     for (unsigned i = 0; i != Size; i++)
  1768:       NewOps.push_back(DAG.getBitcast(MVT::i16, Ops[i]));
  1769: 
  1770:     SDValue T0 =
  1771:         DAG.getNode(ISD::BUILD_VECTOR, dl, tyVector(VecTy, MVT::i16), NewOps);
  1772:     return DAG.getBitcast(tyVector(VecTy, VecTy.getVectorElementType()), T0);
  1773:   }
  1774: 
  1775:   // First, split the BUILD_VECTOR for vector pairs. We could generate
  1776:   // some pairs directly (via splat), but splats should be generated
  1777:   // by the combiner prior to getting here.
  1778:   if (VecTy.getSizeInBits() == 16 * Subtarget.getVectorLength()) {
  1779:     ArrayRef<SDValue> A(Ops);
  1780:     MVT SingleTy = typeSplit(VecTy).first;
  1781:     SDValue V0 = buildHvxVectorReg(A.take_front(Size / 2), dl, SingleTy, DAG);
  1782:     SDValue V1 = buildHvxVectorReg(A.drop_front(Size / 2), dl, SingleTy, DAG);
  1783:     return DAG.getNode(ISD::CONCAT_VECTORS, dl, VecTy, V0, V1);
  1784:   }
  1785: 
  1786:   return buildHvxVectorReg(Ops, dl, VecTy, DAG);
  1787: }
  1788: 
  1789: SDValue
  1790: HexagonTargetLowering::LowerHvxSplatVector(SDValue Op, SelectionDAG &DAG)
  1791:       const {
  1792:   const SDLoc &dl(Op);
  1793:   MVT VecTy = ty(Op);
  1794:   MVT ArgTy = ty(Op.getOperand(0));
  1795: 
  1796:   if (ArgTy == MVT::f16 || ArgTy == MVT::bf16) {
  1797:     MVT SplatTy =  MVT::getVectorVT(MVT::i16, VecTy.getVectorNumElements());
  1798:     SDValue ToInt16 = DAG.getBitcast(MVT::i16, Op.getOperand(0));
  1799:     SDValue ToInt32 = DAG.getNode(ISD::ANY_EXTEND, dl, MVT::i32, ToInt16);
  1800:     SDValue Splat = DAG.getNode(ISD::SPLAT_VECTOR, dl, SplatTy, ToInt32);
```
- EN: It declares or implements routines such as MVT::getVectorVT, getNode, createHvxPrefixPred, dyn_cast<ConstantSDNode>, ... (33 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonISD, HexagonTargetLowering, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 MVT::getVectorVT, getNode, createHvxPrefixPred, dyn_cast<ConstantSDNode>, ... (33 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonISD, HexagonTargetLowering，说明了它与同级后端组件的连接关系。

### Lines 1801-2000 / 第 1801-2000 行

```cpp
  1801:     return DAG.getBitcast(VecTy, Splat);
  1802:   }
  1803: 
  1804:   return SDValue();
  1805: }
  1806: 
  1807: SDValue
  1808: HexagonTargetLowering::LowerHvxConcatVectors(SDValue Op, SelectionDAG &DAG)
  1809:       const {
  1810:   // Vector concatenation of two integer (non-bool) vectors does not need
  1811:   // special lowering. Custom-lower concats of bool vectors and expand
  1812:   // concats of more than 2 vectors.
  1813:   MVT VecTy = ty(Op);
  1814:   const SDLoc &dl(Op);
  1815:   unsigned NumOp = Op.getNumOperands();
  1816:   if (VecTy.getVectorElementType() != MVT::i1) {
  1817:     if (NumOp == 2)
  1818:       return Op;
  1819:     // Expand the other cases into a build-vector.
  1820:     SmallVector<SDValue,8> Elems;
  1821:     for (SDValue V : Op.getNode()->ops())
  1822:       DAG.ExtractVectorElements(V, Elems);
  1823:     // A vector of i16 will be broken up into a build_vector of i16's.
  1824:     // This is a problem, since at the time of operation legalization,
  1825:     // all operations are expected to be type-legalized, and i16 is not
  1826:     // a legal type. If any of the extracted elements is not of a valid
  1827:     // type, sign-extend it to a valid one.
  1828:     for (SDValue &V : Elems) {
  1829:       MVT Ty = ty(V);
  1830:       if (!isTypeLegal(Ty)) {
  1831:         MVT NTy = typeLegalize(Ty, DAG);
  1832:         if (V.getOpcode() == ISD::EXTRACT_VECTOR_ELT) {
  1833:           V = DAG.getNode(ISD::SIGN_EXTEND_INREG, dl, NTy,
  1834:                           DAG.getNode(ISD::EXTRACT_VECTOR_ELT, dl, NTy,
  1835:                                       V.getOperand(0), V.getOperand(1)),
  1836:                           DAG.getValueType(Ty));
  1837:           continue;
  1838:         }
  1839:         // A few less complicated cases.
  1840:         switch (V.getOpcode()) {
  1841:           case ISD::Constant:
  1842:             V = DAG.getSExtOrTrunc(V, dl, NTy);
  1843:             break;
  1844:           case ISD::UNDEF:
  1845:             V = DAG.getUNDEF(NTy);
  1846:             break;
  1847:           case ISD::TRUNCATE:
  1848:             V = V.getOperand(0);
  1849:             break;
  1850:           default:
  1851:             llvm_unreachable("Unexpected vector element");
  1852:         }
  1853:       }
  1854:     }
  1855:     return DAG.getBuildVector(VecTy, dl, Elems);
  1856:   }
  1857: 
  1858:   assert(VecTy.getVectorElementType() == MVT::i1);
  1859:   unsigned HwLen = Subtarget.getVectorLength();
  1860:   assert(isPowerOf2_32(NumOp) && HwLen % NumOp == 0);
  1861: 
  1862:   SDValue Op0 = Op.getOperand(0);
  1863: 
  1864:   // If the operands are HVX types (i.e. not scalar predicates), then
  1865:   // defer the concatenation, and create QCAT instead.
  1866:   if (Subtarget.isHVXVectorType(ty(Op0), true)) {
  1867:     if (NumOp == 2)
  1868:       return DAG.getNode(HexagonISD::QCAT, dl, VecTy, Op0, Op.getOperand(1));
  1869: 
  1870:     ArrayRef<SDUse> U(Op.getNode()->ops());
  1871:     SmallVector<SDValue, 4> SV(U);
  1872:     ArrayRef<SDValue> Ops(SV);
  1873: 
  1874:     MVT HalfTy = typeSplit(VecTy).first;
  1875:     SDValue V0 = DAG.getNode(ISD::CONCAT_VECTORS, dl, HalfTy,
  1876:                              Ops.take_front(NumOp/2));
  1877:     SDValue V1 = DAG.getNode(ISD::CONCAT_VECTORS, dl, HalfTy,
  1878:                              Ops.take_back(NumOp/2));
  1879:     return DAG.getNode(HexagonISD::QCAT, dl, VecTy, V0, V1);
  1880:   }
  1881: 
  1882:   // Count how many bytes (in a vector register) each bit in VecTy
  1883:   // corresponds to.
  1884:   unsigned BitBytes = HwLen / VecTy.getVectorNumElements();
  1885: 
  1886:   // Make sure that createHvxPrefixPred will only ever need to expand
  1887:   // the predicate, i.e. bytes-per-bit in the input is not greater than
  1888:   // the target bytes-per-bit in the result.
  1889:   SDValue Combined = combineConcatOfScalarPreds(Op, BitBytes, DAG);
  1890:   SmallVector<SDValue,8> Prefixes;
  1891:   for (SDValue V : Combined.getNode()->op_values()) {
  1892:     SDValue P = createHvxPrefixPred(V, dl, BitBytes, true, DAG);
  1893:     Prefixes.push_back(P);
  1894:   }
  1895: 
  1896:   unsigned InpLen = ty(Combined.getOperand(0)).getVectorNumElements();
  1897:   MVT ByteTy = MVT::getVectorVT(MVT::i8, HwLen);
  1898:   SDValue S = DAG.getConstant(HwLen - InpLen*BitBytes, dl, MVT::i32);
  1899:   SDValue Res = getZero(dl, ByteTy, DAG);
  1900:   for (unsigned i = 0, e = Prefixes.size(); i != e; ++i) {
  1901:     Res = DAG.getNode(HexagonISD::VROR, dl, ByteTy, Res, S);
  1902:     Res = DAG.getNode(ISD::OR, dl, ByteTy, Res, Prefixes[e-i-1]);
  1903:   }
  1904:   return DAG.getNode(HexagonISD::V2Q, dl, VecTy, Res);
  1905: }
  1906: 
  1907: SDValue
  1908: HexagonTargetLowering::LowerHvxExtractElement(SDValue Op, SelectionDAG &DAG)
  1909:       const {
  1910:   // Change the type of the extracted element to i32.
  1911:   SDValue VecV = Op.getOperand(0);
  1912:   MVT ElemTy = ty(VecV).getVectorElementType();
  1913:   const SDLoc &dl(Op);
  1914:   SDValue IdxV = Op.getOperand(1);
  1915:   if (ElemTy == MVT::i1)
  1916:     return extractHvxElementPred(VecV, IdxV, dl, ty(Op), DAG);
  1917: 
  1918:   return extractHvxElementReg(VecV, IdxV, dl, ty(Op), DAG);
  1919: }
  1920: 
  1921: SDValue
  1922: HexagonTargetLowering::LowerHvxInsertElement(SDValue Op, SelectionDAG &DAG)
  1923:       const {
  1924:   const SDLoc &dl(Op);
  1925:   MVT VecTy = ty(Op);
  1926:   SDValue VecV = Op.getOperand(0);
  1927:   SDValue ValV = Op.getOperand(1);
  1928:   SDValue IdxV = Op.getOperand(2);
  1929:   MVT ElemTy = ty(VecV).getVectorElementType();
  1930:   if (ElemTy == MVT::i1)
  1931:     return insertHvxElementPred(VecV, IdxV, ValV, dl, DAG);
  1932: 
  1933:   if (ElemTy == MVT::f16 || ElemTy == MVT::bf16) {
  1934:     SDValue T0 = DAG.getNode(ISD::INSERT_VECTOR_ELT, dl,
  1935:         tyVector(VecTy, MVT::i16),
  1936:         DAG.getBitcast(tyVector(VecTy, MVT::i16), VecV),
  1937:         DAG.getBitcast(MVT::i16, ValV), IdxV);
  1938:     return DAG.getBitcast(tyVector(VecTy, ElemTy), T0);
  1939:   }
  1940: 
  1941:   return insertHvxElementReg(VecV, IdxV, ValV, dl, DAG);
  1942: }
  1943: 
  1944: SDValue
  1945: HexagonTargetLowering::LowerHvxExtractSubvector(SDValue Op, SelectionDAG &DAG)
  1946:       const {
  1947:   SDValue SrcV = Op.getOperand(0);
  1948:   MVT SrcTy = ty(SrcV);
  1949:   MVT DstTy = ty(Op);
  1950:   SDValue IdxV = Op.getOperand(1);
  1951:   unsigned Idx = IdxV.getNode()->getAsZExtVal();
  1952:   assert(Idx % DstTy.getVectorNumElements() == 0);
  1953:   (void)Idx;
  1954:   const SDLoc &dl(Op);
  1955: 
  1956:   MVT ElemTy = SrcTy.getVectorElementType();
  1957:   if (ElemTy == MVT::i1)
  1958:     return extractHvxSubvectorPred(SrcV, IdxV, dl, DstTy, DAG);
  1959: 
  1960:   return extractHvxSubvectorReg(Op, SrcV, IdxV, dl, DstTy, DAG);
  1961: }
  1962: 
  1963: SDValue
  1964: HexagonTargetLowering::LowerHvxInsertSubvector(SDValue Op, SelectionDAG &DAG)
  1965:       const {
  1966:   // Idx does not need to be a constant.
  1967:   SDValue VecV = Op.getOperand(0);
  1968:   SDValue ValV = Op.getOperand(1);
  1969:   SDValue IdxV = Op.getOperand(2);
  1970: 
  1971:   const SDLoc &dl(Op);
  1972:   MVT VecTy = ty(VecV);
  1973:   MVT ElemTy = VecTy.getVectorElementType();
  1974:   if (ElemTy == MVT::i1)
  1975:     return insertHvxSubvectorPred(VecV, ValV, IdxV, dl, DAG);
  1976: 
  1977:   return insertHvxSubvectorReg(VecV, ValV, IdxV, dl, DAG);
  1978: }
  1979: 
  1980: SDValue
  1981: HexagonTargetLowering::LowerHvxAnyExt(SDValue Op, SelectionDAG &DAG) const {
  1982:   // Lower any-extends of boolean vectors to sign-extends, since they
  1983:   // translate directly to Q2V. Zero-extending could also be done equally
  1984:   // fast, but Q2V is used/recognized in more places.
  1985:   // For all other vectors, use zero-extend.
  1986:   MVT ResTy = ty(Op);
  1987:   SDValue InpV = Op.getOperand(0);
  1988:   MVT ElemTy = ty(InpV).getVectorElementType();
  1989:   if (ElemTy == MVT::i1 && Subtarget.isHVXVectorType(ResTy))
  1990:     return LowerHvxSignExt(Op, DAG);
  1991:   return DAG.getNode(ISD::ZERO_EXTEND, SDLoc(Op), ResTy, InpV);
  1992: }
  1993: 
  1994: SDValue
  1995: HexagonTargetLowering::LowerHvxSignExt(SDValue Op, SelectionDAG &DAG) const {
  1996:   MVT ResTy = ty(Op);
  1997:   SDValue InpV = Op.getOperand(0);
  1998:   MVT ElemTy = ty(InpV).getVectorElementType();
  1999:   if (ElemTy == MVT::i1 && Subtarget.isHVXVectorType(ResTy))
  2000:     return extendHvxVectorPred(InpV, SDLoc(Op), ty(Op), false, DAG);
```
- EN: It declares or implements routines such as getBitcast, SDValue, HexagonTargetLowering::LowerHvxConcatVectors, integer, ... (38 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getBitcast, SDValue, HexagonTargetLowering::LowerHvxConcatVectors, integer, ... (38 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 2001-2200 / 第 2001-2200 行

```cpp
  2001:   return Op;
  2002: }
  2003: 
  2004: SDValue
  2005: HexagonTargetLowering::LowerHvxZeroExt(SDValue Op, SelectionDAG &DAG) const {
  2006:   MVT ResTy = ty(Op);
  2007:   SDValue InpV = Op.getOperand(0);
  2008:   MVT ElemTy = ty(InpV).getVectorElementType();
  2009:   if (ElemTy == MVT::i1 && Subtarget.isHVXVectorType(ResTy))
  2010:     return extendHvxVectorPred(InpV, SDLoc(Op), ty(Op), true, DAG);
  2011:   return Op;
  2012: }
  2013: 
  2014: SDValue
  2015: HexagonTargetLowering::LowerHvxCttz(SDValue Op, SelectionDAG &DAG) const {
  2016:   // Lower vector CTTZ into a computation using CTLZ (Hacker's Delight):
  2017:   // cttz(x) = bitwidth(x) - ctlz(~x & (x-1))
  2018:   const SDLoc &dl(Op);
  2019:   MVT ResTy = ty(Op);
  2020:   SDValue InpV = Op.getOperand(0);
  2021:   assert(ResTy == ty(InpV));
  2022: 
  2023:   // Calculate the vectors of 1 and bitwidth(x).
  2024:   MVT ElemTy = ty(InpV).getVectorElementType();
  2025:   unsigned ElemWidth = ElemTy.getSizeInBits();
  2026: 
  2027:   SDValue Vec1 = DAG.getNode(ISD::SPLAT_VECTOR, dl, ResTy,
  2028:                              DAG.getConstant(1, dl, MVT::i32));
  2029:   SDValue VecW = DAG.getNode(ISD::SPLAT_VECTOR, dl, ResTy,
  2030:                              DAG.getConstant(ElemWidth, dl, MVT::i32));
  2031:   SDValue VecN1 = DAG.getNode(ISD::SPLAT_VECTOR, dl, ResTy,
  2032:                               DAG.getAllOnesConstant(dl, MVT::i32));
  2033: 
  2034:   // Do not use DAG.getNOT, because that would create BUILD_VECTOR with
  2035:   // a BITCAST. Here we can skip the BITCAST (so we don't have to handle
  2036:   // it separately in custom combine or selection).
  2037:   SDValue A = DAG.getNode(ISD::AND, dl, ResTy,
  2038:                           {DAG.getNode(ISD::XOR, dl, ResTy, {InpV, VecN1}),
  2039:                            DAG.getNode(ISD::SUB, dl, ResTy, {InpV, Vec1})});
  2040:   return DAG.getNode(ISD::SUB, dl, ResTy,
  2041:                      {VecW, DAG.getNode(ISD::CTLZ, dl, ResTy, A)});
  2042: }
  2043: 
  2044: SDValue
  2045: HexagonTargetLowering::LowerHvxMulh(SDValue Op, SelectionDAG &DAG) const {
  2046:   const SDLoc &dl(Op);
  2047:   MVT ResTy = ty(Op);
  2048:   assert(ResTy.getVectorElementType() == MVT::i32);
  2049: 
  2050:   SDValue Vs = Op.getOperand(0);
  2051:   SDValue Vt = Op.getOperand(1);
  2052: 
  2053:   SDVTList ResTys = DAG.getVTList(ResTy, ResTy);
  2054:   unsigned Opc = Op.getOpcode();
  2055: 
  2056:   // On HVX v62+ producing the full product is cheap, so legalize MULH to LOHI.
  2057:   if (Opc == ISD::MULHU)
  2058:     return DAG.getNode(HexagonISD::UMUL_LOHI, dl, ResTys, {Vs, Vt}).getValue(1);
  2059:   if (Opc == ISD::MULHS)
  2060:     return DAG.getNode(HexagonISD::SMUL_LOHI, dl, ResTys, {Vs, Vt}).getValue(1);
  2061: 
  2062: #ifndef NDEBUG
  2063:   Op.dump(&DAG);
  2064: #endif
  2065:   llvm_unreachable("Unexpected mulh operation");
  2066: }
  2067: 
  2068: SDValue
  2069: HexagonTargetLowering::LowerHvxMulLoHi(SDValue Op, SelectionDAG &DAG) const {
  2070:   const SDLoc &dl(Op);
  2071:   unsigned Opc = Op.getOpcode();
  2072:   SDValue Vu = Op.getOperand(0);
  2073:   SDValue Vv = Op.getOperand(1);
  2074: 
  2075:   // If the HI part is not used, convert it to a regular MUL.
  2076:   if (auto HiVal = Op.getValue(1); HiVal.use_empty()) {
  2077:     // Need to preserve the types and the number of values.
  2078:     SDValue Hi = DAG.getUNDEF(ty(HiVal));
  2079:     SDValue Lo = DAG.getNode(ISD::MUL, dl, ty(Op), {Vu, Vv});
  2080:     return DAG.getMergeValues({Lo, Hi}, dl);
  2081:   }
  2082: 
  2083:   bool SignedVu = Opc == HexagonISD::SMUL_LOHI;
  2084:   bool SignedVv = Opc == HexagonISD::SMUL_LOHI || Opc == HexagonISD::USMUL_LOHI;
  2085: 
  2086:   // Legal on HVX v62+, but lower it here because patterns can't handle multi-
  2087:   // valued nodes.
  2088:   if (Subtarget.useHVXV62Ops())
  2089:     return emitHvxMulLoHiV62(Vu, SignedVu, Vv, SignedVv, dl, DAG);
  2090: 
  2091:   if (Opc == HexagonISD::SMUL_LOHI) {
  2092:     // Direct MULHS expansion is cheaper than doing the whole SMUL_LOHI,
  2093:     // for other signedness LOHI is cheaper.
  2094:     if (auto LoVal = Op.getValue(0); LoVal.use_empty()) {
  2095:       SDValue Hi = emitHvxMulHsV60(Vu, Vv, dl, DAG);
  2096:       SDValue Lo = DAG.getUNDEF(ty(LoVal));
  2097:       return DAG.getMergeValues({Lo, Hi}, dl);
  2098:     }
  2099:   }
  2100: 
  2101:   return emitHvxMulLoHiV60(Vu, SignedVu, Vv, SignedVv, dl, DAG);
  2102: }
  2103: 
  2104: SDValue
  2105: HexagonTargetLowering::LowerHvxBitcast(SDValue Op, SelectionDAG &DAG) const {
  2106:   SDValue Val = Op.getOperand(0);
  2107:   MVT ResTy = ty(Op);
  2108:   MVT ValTy = ty(Val);
  2109:   const SDLoc &dl(Op);
  2110: 
  2111:   if (isHvxBoolTy(ValTy) && ResTy.isScalarInteger()) {
  2112:     unsigned HwLen = Subtarget.getVectorLength();
  2113:     MVT WordTy = MVT::getVectorVT(MVT::i32, HwLen/4);
  2114:     SDValue VQ = compressHvxPred(Val, dl, WordTy, DAG);
  2115:     unsigned BitWidth = ResTy.getSizeInBits();
  2116: 
  2117:     if (BitWidth < 64) {
  2118:       SDValue W0 = extractHvxElementReg(VQ, DAG.getConstant(0, dl, MVT::i32),
  2119:           dl, MVT::i32, DAG);
  2120:       if (BitWidth == 32)
  2121:         return W0;
  2122:       assert(BitWidth < 32u);
  2123:       return DAG.getZExtOrTrunc(W0, dl, ResTy);
  2124:     }
  2125: 
  2126:     // The result is >= 64 bits. The only options are 64 or 128.
  2127:     assert(BitWidth == 64 || BitWidth == 128);
  2128:     SmallVector<SDValue,4> Words;
  2129:     for (unsigned i = 0; i != BitWidth/32; ++i) {
  2130:       SDValue W = extractHvxElementReg(
  2131:           VQ, DAG.getConstant(i, dl, MVT::i32), dl, MVT::i32, DAG);
  2132:       Words.push_back(W);
  2133:     }
  2134:     SmallVector<SDValue,2> Combines;
  2135:     assert(Words.size() % 2 == 0);
  2136:     for (unsigned i = 0, e = Words.size(); i < e; i += 2) {
  2137:       SDValue C = getCombine(Words[i+1], Words[i], dl, MVT::i64, DAG);
  2138:       Combines.push_back(C);
  2139:     }
  2140: 
  2141:     if (BitWidth == 64)
  2142:       return Combines[0];
  2143: 
  2144:     return DAG.getNode(ISD::BUILD_PAIR, dl, ResTy, Combines);
  2145:   }
  2146: 
  2147:   // Handle bitcast from i32, v2i16, and v4i8 to v32i1.
  2148:   // Splat the input into a 32-element i32 vector, then AND each element
  2149:   // with a unique bitmask to isolate individual bits.
  2150:   auto bitcastI32ToV32I1 = [&](SDValue Val32) {
  2151:     assert(Val32.getValueType().getSizeInBits() == 32 &&
  2152:            "Input must be 32 bits");
  2153:     MVT VecTy = MVT::getVectorVT(MVT::i32, 32);
  2154:     SDValue Splat = DAG.getNode(ISD::SPLAT_VECTOR, dl, VecTy, Val32);
  2155:     SmallVector<SDValue, 32> Mask;
  2156:     for (unsigned i = 0; i < 32; ++i)
  2157:       Mask.push_back(DAG.getConstant(1ull << i, dl, MVT::i32));
  2158: 
  2159:     SDValue MaskVec = DAG.getBuildVector(VecTy, dl, Mask);
  2160:     SDValue Anded = DAG.getNode(ISD::AND, dl, VecTy, Splat, MaskVec);
  2161:     return DAG.getNode(HexagonISD::V2Q, dl, MVT::v32i1, Anded);
  2162:   };
  2163:   // === Case: v32i1 ===
  2164:   if (ResTy == MVT::v32i1 &&
  2165:       (ValTy == MVT::i32 || ValTy == MVT::v2i16 || ValTy == MVT::v4i8) &&
  2166:       Subtarget.useHVX128BOps()) {
  2167:     SDValue Val32 = Val;
  2168:     if (ValTy == MVT::v2i16 || ValTy == MVT::v4i8)
  2169:       Val32 = DAG.getNode(ISD::BITCAST, dl, MVT::i32, Val);
  2170:     return bitcastI32ToV32I1(Val32);
  2171:   }
  2172:   // === Case: v64i1 ===
  2173:   if (ResTy == MVT::v64i1 && ValTy == MVT::i64 && Subtarget.useHVX128BOps()) {
  2174:     // Split i64 into lo/hi 32-bit halves.
  2175:     SDValue Lo = DAG.getNode(ISD::TRUNCATE, dl, MVT::i32, Val);
  2176:     SDValue HiShifted = DAG.getNode(ISD::SRL, dl, MVT::i64, Val,
  2177:                                     DAG.getConstant(32, dl, MVT::i64));
  2178:     SDValue Hi = DAG.getNode(ISD::TRUNCATE, dl, MVT::i32, HiShifted);
  2179: 
  2180:     // Reuse the same 32-bit logic twice.
  2181:     SDValue LoRes = bitcastI32ToV32I1(Lo);
  2182:     SDValue HiRes = bitcastI32ToV32I1(Hi);
  2183: 
  2184:     // Concatenate into a v64i1 predicate.
  2185:     return DAG.getNode(ISD::CONCAT_VECTORS, dl, MVT::v64i1, LoRes, HiRes);
  2186:   }
  2187: 
  2188:   if (isHvxBoolTy(ResTy) && ValTy.isScalarInteger()) {
  2189:     // Handle bitcast from i128 -> v128i1 and i64 -> v64i1.
  2190:     unsigned BitWidth = ValTy.getSizeInBits();
  2191:     unsigned HwLen = Subtarget.getVectorLength();
  2192:     assert(BitWidth == HwLen);
  2193: 
  2194:     MVT ValAsVecTy = MVT::getVectorVT(MVT::i8, BitWidth / 8);
  2195:     SDValue ValAsVec = DAG.getBitcast(ValAsVecTy, Val);
  2196:     // Splat each byte of Val 8 times.
  2197:     // Bytes = [(b0)x8, (b1)x8, ...., (b15)x8]
  2198:     // where b0, b1,..., b15 are least to most significant bytes of I.
  2199:     SmallVector<SDValue, 128> Bytes;
  2200:     // Tmp: 0x01,0x02,0x04,0x08,0x10,0x20,0x40,0x80, 0x01,0x02,0x04,0x08,...
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as HexagonTargetLowering::LowerHvxZeroExt, ty, getOperand, HexagonTargetLowering::LowerHvxCttz, ... (32 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 HexagonTargetLowering::LowerHvxZeroExt, ty, getOperand, HexagonTargetLowering::LowerHvxCttz, ... (32 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 2201-2400 / 第 2201-2400 行

```cpp
  2201:     // These are bytes with the LSB rotated left with respect to their index.
  2202:     SmallVector<SDValue, 128> Tmp;
  2203:     for (unsigned I = 0; I != HwLen / 8; ++I) {
  2204:       SDValue Idx = DAG.getConstant(I, dl, MVT::i32);
  2205:       SDValue Byte =
  2206:           DAG.getNode(ISD::EXTRACT_VECTOR_ELT, dl, MVT::i8, ValAsVec, Idx);
  2207:       for (unsigned J = 0; J != 8; ++J) {
  2208:         Bytes.push_back(Byte);
  2209:         Tmp.push_back(DAG.getConstant(1ull << J, dl, MVT::i8));
  2210:       }
  2211:     }
  2212: 
  2213:     MVT ConstantVecTy = MVT::getVectorVT(MVT::i8, HwLen);
  2214:     SDValue ConstantVec = DAG.getBuildVector(ConstantVecTy, dl, Tmp);
  2215:     SDValue I2V = buildHvxVectorReg(Bytes, dl, ConstantVecTy, DAG);
  2216: 
  2217:     // Each Byte in the I2V will be set iff corresponding bit is set in Val.
  2218:     I2V = DAG.getNode(ISD::AND, dl, ConstantVecTy, {I2V, ConstantVec});
  2219:     return DAG.getNode(HexagonISD::V2Q, dl, ResTy, I2V);
  2220:   }
  2221: 
  2222:   return Op;
  2223: }
  2224: 
  2225: SDValue HexagonTargetLowering::LowerHvxStore(SDValue Op,
  2226:                                              SelectionDAG &DAG) const {
  2227:   const SDLoc &dl(Op);
  2228:   StoreSDNode *SN = cast<StoreSDNode>(Op.getNode());
  2229:   SDValue Val = SN->getValue();
  2230:   MVT ValTy = ty(Val);
  2231: 
  2232:   // Check if this is a store of an HVX bool vector (predicate)
  2233:   if (!isHvxBoolTy(ValTy))
  2234:     return SDValue();
  2235: 
  2236:   unsigned NumBits = ValTy.getVectorNumElements();
  2237:   MachineMemOperand *MMO = SN->getMemOperand();
  2238: 
  2239:   // Check alignment requirements based on predicate size
  2240:   unsigned RequiredAlign = (NumBits == 32) ? 4 : 8;
  2241:   if (MMO->getBaseAlign().value() % RequiredAlign != 0)
  2242:     return SDValue();
  2243: 
  2244:   unsigned HwLen = Subtarget.getVectorLength();
  2245:   MVT WordTy = MVT::getVectorVT(MVT::i32, HwLen / 4);
  2246: 
  2247:   // Compress the predicate into a vector register
  2248:   SDValue VQ = compressHvxPred(Val, dl, WordTy, DAG);
  2249: 
  2250:   // Extract words from the compressed vector
  2251:   SmallVector<SDValue, 4> Words;
  2252:   for (unsigned i = 0; i != NumBits / 32; ++i) {
  2253:     SDValue W = extractHvxElementReg(VQ, DAG.getConstant(i, dl, MVT::i32), dl,
  2254:                                      MVT::i32, DAG);
  2255:     Words.push_back(W);
  2256:   }
  2257: 
  2258:   SDValue Chain = SN->getChain();
  2259:   SDValue BasePtr = SN->getBasePtr();
  2260:   MachinePointerInfo PtrInfo = MMO->getPointerInfo();
  2261: 
  2262:   if (NumBits == 32)
  2263:     return DAG.getStore(Chain, dl, Words[0], BasePtr, PtrInfo,
  2264:                         MMO->getBaseAlign());
  2265: 
  2266:   if (NumBits == 64) {
  2267:     SDValue W64 = getCombine(Words[1], Words[0], dl, MVT::i64, DAG);
  2268:     return DAG.getStore(Chain, dl, W64, BasePtr, PtrInfo, MMO->getBaseAlign());
  2269:   }
  2270: 
  2271:   if (NumBits == 128) {
  2272:     SDValue Lo64 = getCombine(Words[1], Words[0], dl, MVT::i64, DAG);
  2273:     SDValue Hi64 = getCombine(Words[3], Words[2], dl, MVT::i64, DAG);
  2274: 
  2275:     Chain =
  2276:         DAG.getStore(Chain, dl, Lo64, BasePtr, PtrInfo, MMO->getBaseAlign());
  2277: 
  2278:     SDValue Offset8 = DAG.getConstant(8, dl, MVT::i32);
  2279:     SDValue Ptr8 = DAG.getNode(ISD::ADD, dl, MVT::i32, BasePtr, Offset8);
  2280:     return DAG.getStore(Chain, dl, Hi64, Ptr8, PtrInfo.getWithOffset(8),
  2281:                         Align(8));
  2282:   }
  2283: 
  2284:   return SDValue();
  2285: }
  2286: 
  2287: SDValue HexagonTargetLowering::LowerHvxLoad(SDValue Op,
  2288:                                             SelectionDAG &DAG) const {
  2289:   const SDLoc &dl(Op);
  2290:   LoadSDNode *LN = cast<LoadSDNode>(Op.getNode());
  2291:   MVT ResTy = ty(Op);
  2292: 
  2293:   // Check if this is a load of an HVX bool vector (predicate)
  2294:   if (!isHvxBoolTy(ResTy))
  2295:     return SDValue();
  2296: 
  2297:   unsigned NumBits = ResTy.getVectorNumElements();
  2298:   MachineMemOperand *MMO = LN->getMemOperand();
  2299: 
  2300:   unsigned RequiredAlign = (NumBits == 32) ? 4 : 8;
  2301:   if (MMO->getBaseAlign().value() % RequiredAlign != 0)
  2302:     return SDValue();
  2303: 
  2304:   SDValue Chain = LN->getChain();
  2305:   SDValue BasePtr = LN->getBasePtr();
  2306:   MachinePointerInfo PtrInfo = MMO->getPointerInfo();
  2307: 
  2308:   if (NumBits == 32) {
  2309:     SDValue W32 =
  2310:         DAG.getLoad(MVT::i32, dl, Chain, BasePtr, PtrInfo, MMO->getBaseAlign());
  2311:     SDValue Pred = DAG.getNode(ISD::BITCAST, dl, MVT::v32i1, W32);
  2312:     SDValue Ops[] = {Pred, W32.getValue(1)};
  2313:     return DAG.getMergeValues(Ops, dl);
  2314:   }
  2315: 
  2316:   if (NumBits == 64) {
  2317:     SDValue W64 =
  2318:         DAG.getLoad(MVT::i64, dl, Chain, BasePtr, PtrInfo, MMO->getBaseAlign());
  2319:     SDValue Pred = DAG.getNode(ISD::BITCAST, dl, MVT::v64i1, W64);
  2320:     SDValue Ops[] = {Pred, W64.getValue(1)};
  2321:     return DAG.getMergeValues(Ops, dl);
  2322:   }
  2323: 
  2324:   if (NumBits == 128) {
  2325:     SDValue Lo64 =
  2326:         DAG.getLoad(MVT::i64, dl, Chain, BasePtr, PtrInfo, MMO->getBaseAlign());
  2327:     Chain = Lo64.getValue(1);
  2328: 
  2329:     SDValue Offset8 = DAG.getConstant(8, dl, MVT::i32);
  2330:     SDValue Ptr8 = DAG.getNode(ISD::ADD, dl, MVT::i32, BasePtr, Offset8);
  2331:     SDValue Hi64 = DAG.getLoad(MVT::i64, dl, Chain, Ptr8,
  2332:                                PtrInfo.getWithOffset(8), Align(8));
  2333: 
  2334:     SDValue LoPred = DAG.getNode(ISD::BITCAST, dl, MVT::v64i1, Lo64);
  2335:     SDValue HiPred = DAG.getNode(ISD::BITCAST, dl, MVT::v64i1, Hi64);
  2336:     SDValue Pred =
  2337:         DAG.getNode(ISD::CONCAT_VECTORS, dl, MVT::v128i1, LoPred, HiPred);
  2338: 
  2339:     SDValue Ops[] = {Pred, Hi64.getValue(1)};
  2340:     return DAG.getMergeValues(Ops, dl);
  2341:   }
  2342: 
  2343:   return SDValue();
  2344: }
  2345: 
  2346: SDValue
  2347: HexagonTargetLowering::LowerHvxExtend(SDValue Op, SelectionDAG &DAG) const {
  2348:   // Sign- and zero-extends are legal.
  2349:   assert(Op.getOpcode() == ISD::ANY_EXTEND_VECTOR_INREG);
  2350:   return DAG.getNode(ISD::ZERO_EXTEND_VECTOR_INREG, SDLoc(Op), ty(Op),
  2351:                      Op.getOperand(0));
  2352: }
  2353: 
  2354: SDValue
  2355: HexagonTargetLowering::LowerHvxSelect(SDValue Op, SelectionDAG &DAG) const {
  2356:   MVT ResTy = ty(Op);
  2357:   if (ResTy.getVectorElementType() != MVT::i1)
  2358:     return Op;
  2359: 
  2360:   const SDLoc &dl(Op);
  2361:   unsigned HwLen = Subtarget.getVectorLength();
  2362:   unsigned VecLen = ResTy.getVectorNumElements();
  2363:   assert(HwLen % VecLen == 0);
  2364:   unsigned ElemSize = HwLen / VecLen;
  2365: 
  2366:   MVT VecTy = MVT::getVectorVT(MVT::getIntegerVT(ElemSize * 8), VecLen);
  2367:   SDValue S =
  2368:       DAG.getNode(ISD::SELECT, dl, VecTy, Op.getOperand(0),
  2369:                   DAG.getNode(HexagonISD::Q2V, dl, VecTy, Op.getOperand(1)),
  2370:                   DAG.getNode(HexagonISD::Q2V, dl, VecTy, Op.getOperand(2)));
  2371:   return DAG.getNode(HexagonISD::V2Q, dl, ResTy, S);
  2372: }
  2373: 
  2374: SDValue
  2375: HexagonTargetLowering::LowerHvxShift(SDValue Op, SelectionDAG &DAG) const {
  2376:   if (SDValue S = getVectorShiftByInt(Op, DAG))
  2377:     return S;
  2378:   return Op;
  2379: }
  2380: 
  2381: SDValue
  2382: HexagonTargetLowering::LowerHvxFunnelShift(SDValue Op,
  2383:                                            SelectionDAG &DAG) const {
  2384:   unsigned Opc = Op.getOpcode();
  2385:   assert(Opc == ISD::FSHL || Opc == ISD::FSHR);
  2386: 
  2387:   // Make sure the shift amount is within the range of the bitwidth
  2388:   // of the element type.
  2389:   SDValue A = Op.getOperand(0);
  2390:   SDValue B = Op.getOperand(1);
  2391:   SDValue S = Op.getOperand(2);
  2392: 
  2393:   MVT InpTy = ty(A);
  2394:   MVT ElemTy = InpTy.getVectorElementType();
  2395: 
  2396:   const SDLoc &dl(Op);
  2397:   unsigned ElemWidth = ElemTy.getSizeInBits();
  2398:   bool IsLeft = Opc == ISD::FSHL;
  2399: 
  2400:   // The expansion into regular shifts produces worse code for i8 and for
```
- EN: It declares or implements routines such as getConstant, getNode, push_back, MVT::getVectorVT, ... (36 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonISD, HexagonTargetLowering, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getConstant, getNode, push_back, MVT::getVectorVT, ... (36 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonISD, HexagonTargetLowering，说明了它与同级后端组件的连接关系。

### Lines 2401-2600 / 第 2401-2600 行

```cpp
  2401:   // right shift of i32 on v65+.
  2402:   bool UseShifts = ElemTy != MVT::i8;
  2403:   if (Subtarget.useHVXV65Ops() && ElemTy == MVT::i32)
  2404:     UseShifts = false;
  2405: 
  2406:   if (SDValue SplatV = getSplatValue(S, DAG); SplatV && UseShifts) {
  2407:     // If this is a funnel shift by a scalar, lower it into regular shifts.
  2408:     SDValue Mask = DAG.getConstant(ElemWidth - 1, dl, MVT::i32);
  2409:     SDValue ModS =
  2410:         DAG.getNode(ISD::AND, dl, MVT::i32,
  2411:                     {DAG.getZExtOrTrunc(SplatV, dl, MVT::i32), Mask});
  2412:     SDValue NegS =
  2413:         DAG.getNode(ISD::SUB, dl, MVT::i32,
  2414:                     {DAG.getConstant(ElemWidth, dl, MVT::i32), ModS});
  2415:     SDValue IsZero =
  2416:         DAG.getSetCC(dl, MVT::i1, ModS, getZero(dl, MVT::i32, DAG), ISD::SETEQ);
  2417:     // FSHL A, B  =>  A <<  | B >>n
  2418:     // FSHR A, B  =>  A <<n | B >>
  2419:     SDValue Part1 =
  2420:         DAG.getNode(HexagonISD::VASL, dl, InpTy, {A, IsLeft ? ModS : NegS});
  2421:     SDValue Part2 =
  2422:         DAG.getNode(HexagonISD::VLSR, dl, InpTy, {B, IsLeft ? NegS : ModS});
  2423:     SDValue Or = DAG.getNode(ISD::OR, dl, InpTy, {Part1, Part2});
  2424:     // If the shift amount was 0, pick A or B, depending on the direction.
  2425:     // The opposite shift will also be by 0, so the "Or" will be incorrect.
  2426:     return DAG.getNode(ISD::SELECT, dl, InpTy, {IsZero, (IsLeft ? A : B), Or});
  2427:   }
  2428: 
  2429:   SDValue Mask = DAG.getSplatBuildVector(
  2430:       InpTy, dl, DAG.getConstant(ElemWidth - 1, dl, ElemTy));
  2431: 
  2432:   unsigned MOpc = Opc == ISD::FSHL ? HexagonISD::MFSHL : HexagonISD::MFSHR;
  2433:   return DAG.getNode(MOpc, dl, ty(Op),
  2434:                      {A, B, DAG.getNode(ISD::AND, dl, InpTy, {S, Mask})});
  2435: }
  2436: 
  2437: SDValue
  2438: HexagonTargetLowering::LowerHvxIntrinsic(SDValue Op, SelectionDAG &DAG) const {
  2439:   const SDLoc &dl(Op);
  2440:   unsigned IntNo = Op.getConstantOperandVal(0);
  2441:   SmallVector<SDValue> Ops(Op->ops());
  2442: 
  2443:   auto Swap = [&](SDValue P) {
  2444:     return DAG.getMergeValues({P.getValue(1), P.getValue(0)}, dl);
  2445:   };
  2446: 
  2447:   switch (IntNo) {
  2448:   case Intrinsic::hexagon_V6_pred_typecast:
  2449:   case Intrinsic::hexagon_V6_pred_typecast_128B: {
  2450:     MVT ResTy = ty(Op), InpTy = ty(Ops[1]);
  2451:     if (isHvxBoolTy(ResTy) && isHvxBoolTy(InpTy)) {
  2452:       if (ResTy == InpTy)
  2453:         return Ops[1];
  2454:       return DAG.getNode(HexagonISD::TYPECAST, dl, ResTy, Ops[1]);
  2455:     }
  2456:     break;
  2457:   }
  2458:   case Intrinsic::hexagon_V6_vmpyss_parts:
  2459:   case Intrinsic::hexagon_V6_vmpyss_parts_128B:
  2460:     return Swap(DAG.getNode(HexagonISD::SMUL_LOHI, dl, Op->getVTList(),
  2461:                             {Ops[1], Ops[2]}));
  2462:   case Intrinsic::hexagon_V6_vmpyuu_parts:
  2463:   case Intrinsic::hexagon_V6_vmpyuu_parts_128B:
  2464:     return Swap(DAG.getNode(HexagonISD::UMUL_LOHI, dl, Op->getVTList(),
  2465:                             {Ops[1], Ops[2]}));
  2466:   case Intrinsic::hexagon_V6_vmpyus_parts:
  2467:   case Intrinsic::hexagon_V6_vmpyus_parts_128B: {
  2468:     return Swap(DAG.getNode(HexagonISD::USMUL_LOHI, dl, Op->getVTList(),
  2469:                             {Ops[1], Ops[2]}));
  2470:   }
  2471:   } // switch
  2472: 
  2473:   return Op;
  2474: }
  2475: 
  2476: SDValue
  2477: HexagonTargetLowering::LowerHvxMaskedOp(SDValue Op, SelectionDAG &DAG) const {
  2478:   const SDLoc &dl(Op);
  2479:   unsigned HwLen = Subtarget.getVectorLength();
  2480:   MachineFunction &MF = DAG.getMachineFunction();
  2481:   auto *MaskN = cast<MaskedLoadStoreSDNode>(Op.getNode());
  2482:   SDValue Mask = MaskN->getMask();
  2483:   SDValue Chain = MaskN->getChain();
  2484:   SDValue Base = MaskN->getBasePtr();
  2485:   auto *MemOp = MF.getMachineMemOperand(MaskN->getMemOperand(), 0, HwLen);
  2486: 
  2487:   unsigned Opc = Op->getOpcode();
  2488:   assert(Opc == ISD::MLOAD || Opc == ISD::MSTORE);
  2489: 
  2490:   if (Opc == ISD::MLOAD) {
  2491:     MVT ValTy = ty(Op);
  2492:     SDValue Load = DAG.getLoad(ValTy, dl, Chain, Base, MemOp);
  2493:     SDValue Thru = cast<MaskedLoadSDNode>(MaskN)->getPassThru();
  2494:     if (isUndef(Thru))
  2495:       return Load;
  2496:     SDValue VSel = DAG.getNode(ISD::VSELECT, dl, ValTy, Mask, Load, Thru);
  2497:     return DAG.getMergeValues({VSel, Load.getValue(1)}, dl);
  2498:   }
  2499: 
  2500:   // MSTORE
  2501:   // HVX only has aligned masked stores.
  2502: 
  2503:   // TODO: Fold negations of the mask into the store.
  2504:   unsigned StoreOpc = Hexagon::V6_vS32b_qpred_ai;
  2505:   SDValue Value = cast<MaskedStoreSDNode>(MaskN)->getValue();
  2506:   SDValue Offset0 = DAG.getTargetConstant(0, dl, ty(Base));
  2507: 
  2508:   if (MaskN->getAlign().value() % HwLen == 0) {
  2509:     SDValue Store = getInstr(StoreOpc, dl, MVT::Other,
  2510:                              {Mask, Base, Offset0, Value, Chain}, DAG);
  2511:     DAG.setNodeMemRefs(cast<MachineSDNode>(Store.getNode()), {MemOp});
  2512:     return Store;
  2513:   }
  2514: 
  2515:   // Unaligned case.
  2516:   auto StoreAlign = [&](SDValue V, SDValue A) {
  2517:     SDValue Z = getZero(dl, ty(V), DAG);
  2518:     // TODO: use funnel shifts?
  2519:     // vlalign(Vu,Vv,Rt) rotates the pair Vu:Vv left by Rt and takes the
  2520:     // upper half.
  2521:     SDValue LoV = getInstr(Hexagon::V6_vlalignb, dl, ty(V), {V, Z, A}, DAG);
  2522:     SDValue HiV = getInstr(Hexagon::V6_vlalignb, dl, ty(V), {Z, V, A}, DAG);
  2523:     return std::make_pair(LoV, HiV);
  2524:   };
  2525: 
  2526:   MVT ByteTy = MVT::getVectorVT(MVT::i8, HwLen);
  2527:   MVT BoolTy = MVT::getVectorVT(MVT::i1, HwLen);
  2528:   SDValue MaskV = DAG.getNode(HexagonISD::Q2V, dl, ByteTy, Mask);
  2529:   VectorPair Tmp = StoreAlign(MaskV, Base);
  2530:   VectorPair MaskU = {DAG.getNode(HexagonISD::V2Q, dl, BoolTy, Tmp.first),
  2531:                       DAG.getNode(HexagonISD::V2Q, dl, BoolTy, Tmp.second)};
  2532:   VectorPair ValueU = StoreAlign(Value, Base);
  2533: 
  2534:   SDValue Offset1 = DAG.getTargetConstant(HwLen, dl, MVT::i32);
  2535:   SDValue StoreLo =
  2536:       getInstr(StoreOpc, dl, MVT::Other,
  2537:                {MaskU.first, Base, Offset0, ValueU.first, Chain}, DAG);
  2538:   SDValue StoreHi =
  2539:       getInstr(StoreOpc, dl, MVT::Other,
  2540:                {MaskU.second, Base, Offset1, ValueU.second, Chain}, DAG);
  2541:   DAG.setNodeMemRefs(cast<MachineSDNode>(StoreLo.getNode()), {MemOp});
  2542:   DAG.setNodeMemRefs(cast<MachineSDNode>(StoreHi.getNode()), {MemOp});
  2543:   return DAG.getNode(ISD::TokenFactor, dl, MVT::Other, {StoreLo, StoreHi});
  2544: }
  2545: 
  2546: SDValue HexagonTargetLowering::LowerHvxFpExtend(SDValue Op,
  2547:                                                 SelectionDAG &DAG) const {
  2548:   // This conversion only applies to QFloat. IEEE extension from f16 to f32
  2549:   // is legal (done via a pattern).
  2550:   assert(Subtarget.useHVXQFloatOps());
  2551: 
  2552:   assert(Op->getOpcode() == ISD::FP_EXTEND);
  2553: 
  2554:   MVT VecTy = ty(Op);
  2555:   MVT ArgTy = ty(Op.getOperand(0));
  2556:   const SDLoc &dl(Op);
  2557: 
  2558:   if (ArgTy == MVT::v64bf16) {
  2559:     MVT HalfTy = typeSplit(VecTy).first;
  2560:     SDValue BF16Vec = Op.getOperand(0);
  2561:     SDValue Zeroes =
  2562:         getInstr(Hexagon::V6_vxor, dl, HalfTy, {BF16Vec, BF16Vec}, DAG);
  2563:     // Interleave zero vector with the bf16 vector, with zeroes in the lower
  2564:     // half of each 32 bit lane, effectively extending the bf16 values to fp32
  2565:     // values.
  2566:     SDValue ShuffVec =
  2567:         getInstr(Hexagon::V6_vshufoeh, dl, VecTy, {BF16Vec, Zeroes}, DAG);
  2568:     VectorPair VecPair = opSplit(ShuffVec, dl, DAG);
  2569:     SDValue Result = getInstr(Hexagon::V6_vshuffvdd, dl, VecTy,
  2570:                               {VecPair.second, VecPair.first,
  2571:                                DAG.getSignedConstant(-4, dl, MVT::i32)},
  2572:                               DAG);
  2573:     return Result;
  2574:   }
  2575: 
  2576:   assert(VecTy == MVT::v64f32 && ArgTy == MVT::v64f16);
  2577: 
  2578:   SDValue F16Vec = Op.getOperand(0);
  2579: 
  2580:   APFloat FloatVal = APFloat(1.0f);
  2581:   bool Ignored;
  2582:   FloatVal.convert(APFloat::IEEEhalf(), APFloat::rmNearestTiesToEven, &Ignored);
  2583:   SDValue Fp16Ones = DAG.getConstantFP(FloatVal, dl, ArgTy);
  2584:   SDValue VmpyVec =
  2585:       getInstr(Hexagon::V6_vmpy_qf32_hf, dl, VecTy, {F16Vec, Fp16Ones}, DAG);
  2586: 
  2587:   MVT HalfTy = typeSplit(VecTy).first;
  2588:   VectorPair Pair = opSplit(VmpyVec, dl, DAG);
  2589:   SDValue LoVec =
  2590:       getInstr(Hexagon::V6_vconv_sf_qf32, dl, HalfTy, {Pair.first}, DAG);
  2591:   SDValue HiVec =
  2592:       getInstr(Hexagon::V6_vconv_sf_qf32, dl, HalfTy, {Pair.second}, DAG);
  2593: 
  2594:   SDValue ShuffVec =
  2595:       getInstr(Hexagon::V6_vshuffvdd, dl, VecTy,
  2596:                {HiVec, LoVec, DAG.getSignedConstant(-4, dl, MVT::i32)}, DAG);
  2597: 
  2598:   return ShuffVec;
  2599: }
  2600: 
```
- EN: It declares or implements routines such as getConstant, getSetCC, getSplatBuildVector, HexagonTargetLowering::LowerHvxIntrinsic, ... (34 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonISD, HexagonTargetLowering, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getConstant, getSetCC, getSplatBuildVector, HexagonTargetLowering::LowerHvxIntrinsic, ... (34 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonISD, HexagonTargetLowering，说明了它与同级后端组件的连接关系。

### Lines 2601-2800 / 第 2601-2800 行

```cpp
  2601: SDValue
  2602: HexagonTargetLowering::LowerHvxFpToInt(SDValue Op, SelectionDAG &DAG) const {
  2603:   // Catch invalid conversion ops (just in case).
  2604:   assert(Op.getOpcode() == ISD::FP_TO_SINT ||
  2605:          Op.getOpcode() == ISD::FP_TO_UINT);
  2606: 
  2607:   MVT ResTy = ty(Op);
  2608:   MVT FpTy = ty(Op.getOperand(0)).getVectorElementType();
  2609:   MVT IntTy = ResTy.getVectorElementType();
  2610: 
  2611:   if (Subtarget.useHVXIEEEFPOps()) {
  2612:     // There are only conversions from f16.
  2613:     if (FpTy == MVT::f16) {
  2614:       // Other int types aren't legal in HVX, so we shouldn't see them here.
  2615:       assert(IntTy == MVT::i8 || IntTy == MVT::i16 || IntTy == MVT::i32);
  2616:       // Conversions to i8 and i16 are legal.
  2617:       if (IntTy == MVT::i8 || IntTy == MVT::i16)
  2618:         return Op;
  2619:     }
  2620:   }
  2621: 
  2622:   if (IntTy.getSizeInBits() != FpTy.getSizeInBits())
  2623:     return EqualizeFpIntConversion(Op, DAG);
  2624: 
  2625:   return ExpandHvxFpToInt(Op, DAG);
  2626: }
  2627: 
  2628: // For vector type v32i1 uint_to_fp/sint_to_fp to v32f32:
  2629: // R1 = #1, R2 holds the v32i1 param
  2630: // V1 = vsplat(R1)
  2631: // V2 = vsplat(R2)
  2632: // Q0 = vand(V1,R1)
  2633: // V0.w=prefixsum(Q0)
  2634: // V0.w=vsub(V0.w,V1.w)
  2635: // V2.w = vlsr(V2.w,V0.w)
  2636: // V2 = vand(V2,V1)
  2637: // V2.sf = V2.w
  2638: SDValue HexagonTargetLowering::LowerHvxPred32ToFp(SDValue PredOp,
  2639:                                                   SelectionDAG &DAG) const {
  2640: 
  2641:   MVT ResTy = ty(PredOp);
  2642:   const SDLoc &dl(PredOp);
  2643: 
  2644:   SDValue Const = DAG.getTargetConstant(0x1, dl, MVT::i32);
  2645:   SDNode *RegConst = DAG.getMachineNode(Hexagon::A2_tfrsi, dl, MVT::i32, Const);
  2646:   SDNode *SplatConst = DAG.getMachineNode(Hexagon::V6_lvsplatw, dl, MVT::v32i32,
  2647:                                           SDValue(RegConst, 0));
  2648:   SDNode *PredTransfer =
  2649:       DAG.getMachineNode(Hexagon::V6_vandvrt, dl, MVT::v32i1,
  2650:                          SDValue(SplatConst, 0), SDValue(RegConst, 0));
  2651:   SDNode *PrefixSum = DAG.getMachineNode(Hexagon::V6_vprefixqw, dl, MVT::v32i32,
  2652:                                          SDValue(PredTransfer, 0));
  2653:   SDNode *SplatParam = DAG.getMachineNode(
  2654:       Hexagon::V6_lvsplatw, dl, MVT::v32i32,
  2655:       DAG.getNode(ISD::BITCAST, dl, MVT::i32, PredOp.getOperand(0)));
  2656:   SDNode *Vsub =
  2657:       DAG.getMachineNode(Hexagon::V6_vsubw, dl, MVT::v32i32,
  2658:                          SDValue(PrefixSum, 0), SDValue(SplatConst, 0));
  2659:   SDNode *IndexShift =
  2660:       DAG.getMachineNode(Hexagon::V6_vlsrwv, dl, MVT::v32i32,
  2661:                          SDValue(SplatParam, 0), SDValue(Vsub, 0));
  2662:   SDNode *MaskOff =
  2663:       DAG.getMachineNode(Hexagon::V6_vand, dl, MVT::v32i32,
  2664:                          SDValue(IndexShift, 0), SDValue(SplatConst, 0));
  2665:   SDNode *Convert = DAG.getMachineNode(Hexagon::V6_vconv_sf_w, dl, ResTy,
  2666:                                        SDValue(MaskOff, 0));
  2667:   return SDValue(Convert, 0);
  2668: }
  2669: 
  2670: // For vector type v64i1 uint_to_fo to v64f16:
  2671: // i64 R32 = bitcast v64i1 R3:2 (R3:2 holds v64i1)
  2672: // R3 = subreg_high (R32)
  2673: // R2 = subreg_low (R32)
  2674: // R1 = #1
  2675: // V1 = vsplat(R1)
  2676: // V2 = vsplat(R2)
  2677: // V3 = vsplat(R3)
  2678: // Q0 = vand(V1,R1)
  2679: // V0.w=prefixsum(Q0)
  2680: // V0.w=vsub(V0.w,V1.w)
  2681: // V2.w = vlsr(V2.w,V0.w)
  2682: // V3.w = vlsr(V3.w,V0.w)
  2683: // V2 = vand(V2,V1)
  2684: // V3 = vand(V3,V1)
  2685: // V2.h = vpacke(V3.w,V2.w)
  2686: // V2.hf = V2.h
  2687: SDValue HexagonTargetLowering::LowerHvxPred64ToFp(SDValue PredOp,
  2688:                                                   SelectionDAG &DAG) const {
  2689: 
  2690:   MVT ResTy = ty(PredOp);
  2691:   const SDLoc &dl(PredOp);
  2692: 
  2693:   SDValue Inp = DAG.getNode(ISD::BITCAST, dl, MVT::i64, PredOp.getOperand(0));
  2694:   // Get the hi and lo regs
  2695:   SDValue HiReg =
  2696:       DAG.getTargetExtractSubreg(Hexagon::isub_hi, dl, MVT::i32, Inp);
  2697:   SDValue LoReg =
  2698:       DAG.getTargetExtractSubreg(Hexagon::isub_lo, dl, MVT::i32, Inp);
  2699:   // Get constant #1 and splat into vector V1
  2700:   SDValue Const = DAG.getTargetConstant(0x1, dl, MVT::i32);
  2701:   SDNode *RegConst = DAG.getMachineNode(Hexagon::A2_tfrsi, dl, MVT::i32, Const);
  2702:   SDNode *SplatConst = DAG.getMachineNode(Hexagon::V6_lvsplatw, dl, MVT::v32i32,
  2703:                                           SDValue(RegConst, 0));
  2704:   // Splat the hi and lo args
  2705:   SDNode *SplatHi =
  2706:       DAG.getMachineNode(Hexagon::V6_lvsplatw, dl, MVT::v32i32,
  2707:                          DAG.getNode(ISD::BITCAST, dl, MVT::i32, HiReg));
  2708:   SDNode *SplatLo =
  2709:       DAG.getMachineNode(Hexagon::V6_lvsplatw, dl, MVT::v32i32,
  2710:                          DAG.getNode(ISD::BITCAST, dl, MVT::i32, LoReg));
  2711:   // vand between splatted const and const
  2712:   SDNode *PredTransfer =
  2713:       DAG.getMachineNode(Hexagon::V6_vandvrt, dl, MVT::v32i1,
  2714:                          SDValue(SplatConst, 0), SDValue(RegConst, 0));
  2715:   // Get the prefixsum
  2716:   SDNode *PrefixSum = DAG.getMachineNode(Hexagon::V6_vprefixqw, dl, MVT::v32i32,
  2717:                                          SDValue(PredTransfer, 0));
  2718:   // Get the vsub
  2719:   SDNode *Vsub =
  2720:       DAG.getMachineNode(Hexagon::V6_vsubw, dl, MVT::v32i32,
  2721:                          SDValue(PrefixSum, 0), SDValue(SplatConst, 0));
  2722:   // Get vlsr for hi and lo
  2723:   SDNode *IndexShift_hi =
  2724:       DAG.getMachineNode(Hexagon::V6_vlsrwv, dl, MVT::v32i32,
  2725:                          SDValue(SplatHi, 0), SDValue(Vsub, 0));
  2726:   SDNode *IndexShift_lo =
  2727:       DAG.getMachineNode(Hexagon::V6_vlsrwv, dl, MVT::v32i32,
  2728:                          SDValue(SplatLo, 0), SDValue(Vsub, 0));
  2729:   // Get vand of hi and lo
  2730:   SDNode *MaskOff_hi =
  2731:       DAG.getMachineNode(Hexagon::V6_vand, dl, MVT::v32i32,
  2732:                          SDValue(IndexShift_hi, 0), SDValue(SplatConst, 0));
  2733:   SDNode *MaskOff_lo =
  2734:       DAG.getMachineNode(Hexagon::V6_vand, dl, MVT::v32i32,
  2735:                          SDValue(IndexShift_lo, 0), SDValue(SplatConst, 0));
  2736:   // Pack them
  2737:   SDNode *Pack =
  2738:       DAG.getMachineNode(Hexagon::V6_vpackeh, dl, MVT::v64i16,
  2739:                          SDValue(MaskOff_hi, 0), SDValue(MaskOff_lo, 0));
  2740:   SDNode *Convert =
  2741:       DAG.getMachineNode(Hexagon::V6_vconv_hf_h, dl, ResTy, SDValue(Pack, 0));
  2742:   return SDValue(Convert, 0);
  2743: }
  2744: 
  2745: SDValue
  2746: HexagonTargetLowering::LowerHvxIntToFp(SDValue Op, SelectionDAG &DAG) const {
  2747:   // Catch invalid conversion ops (just in case).
  2748:   assert(Op.getOpcode() == ISD::SINT_TO_FP ||
  2749:          Op.getOpcode() == ISD::UINT_TO_FP);
  2750: 
  2751:   MVT ResTy = ty(Op);
  2752:   MVT IntTy = ty(Op.getOperand(0)).getVectorElementType();
  2753:   MVT FpTy = ResTy.getVectorElementType();
  2754: 
  2755:   if (Op.getOpcode() == ISD::UINT_TO_FP || Op.getOpcode() == ISD::SINT_TO_FP) {
  2756:     if (ResTy == MVT::v32f32 && ty(Op.getOperand(0)) == MVT::v32i1)
  2757:       return LowerHvxPred32ToFp(Op, DAG);
  2758:     if (ResTy == MVT::v64f16 && ty(Op.getOperand(0)) == MVT::v64i1)
  2759:       return LowerHvxPred64ToFp(Op, DAG);
  2760:   }
  2761: 
  2762:   if (Subtarget.useHVXIEEEFPOps()) {
  2763:     // There are only conversions to f16.
  2764:     if (FpTy == MVT::f16) {
  2765:       // Other int types aren't legal in HVX, so we shouldn't see them here.
  2766:       assert(IntTy == MVT::i8 || IntTy == MVT::i16 || IntTy == MVT::i32);
  2767:       // i8, i16 -> f16 is legal.
  2768:       if (IntTy == MVT::i8 || IntTy == MVT::i16)
  2769:         return Op;
  2770:     }
  2771:   }
  2772: 
  2773:   if (IntTy.getSizeInBits() != FpTy.getSizeInBits())
  2774:     return EqualizeFpIntConversion(Op, DAG);
  2775: 
  2776:   return ExpandHvxIntToFp(Op, DAG);
  2777: }
  2778: 
  2779: HexagonTargetLowering::TypePair
  2780: HexagonTargetLowering::typeExtendToWider(MVT Ty0, MVT Ty1) const {
  2781:   // Compare the widths of elements of the two types, and extend the narrower
  2782:   // type to match the with of the wider type. For vector types, apply this
  2783:   // to the element type.
  2784:   assert(Ty0.isVector() == Ty1.isVector());
  2785: 
  2786:   MVT ElemTy0 = Ty0.getScalarType();
  2787:   MVT ElemTy1 = Ty1.getScalarType();
  2788: 
  2789:   unsigned Width0 = ElemTy0.getSizeInBits();
  2790:   unsigned Width1 = ElemTy1.getSizeInBits();
  2791:   unsigned MaxWidth = std::max(Width0, Width1);
  2792: 
  2793:   auto getScalarWithWidth = [](MVT ScalarTy, unsigned Width) {
  2794:     if (ScalarTy.isInteger())
  2795:       return MVT::getIntegerVT(Width);
  2796:     assert(ScalarTy.isFloatingPoint());
  2797:     return MVT::getFloatingPointVT(Width);
  2798:   };
  2799: 
  2800:   MVT WideETy0 = getScalarWithWidth(ElemTy0, MaxWidth);
```
- EN: It declares or implements routines such as HexagonTargetLowering::LowerHvxFpToInt, ops, ty, getVectorElementType, ... (22 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonTargetLowering::LowerHvxFpToInt, ops, ty, getVectorElementType, ... (22 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering，说明了它与同级后端组件的连接关系。

### Lines 2801-3000 / 第 2801-3000 行

```cpp
  2801:   MVT WideETy1 = getScalarWithWidth(ElemTy1, MaxWidth);
  2802: 
  2803:   if (!Ty0.isVector()) {
  2804:     // Both types are scalars.
  2805:     return {WideETy0, WideETy1};
  2806:   }
  2807: 
  2808:   // Vector types.
  2809:   unsigned NumElem = Ty0.getVectorNumElements();
  2810:   assert(NumElem == Ty1.getVectorNumElements());
  2811: 
  2812:   return {MVT::getVectorVT(WideETy0, NumElem),
  2813:           MVT::getVectorVT(WideETy1, NumElem)};
  2814: }
  2815: 
  2816: HexagonTargetLowering::TypePair
  2817: HexagonTargetLowering::typeWidenToWider(MVT Ty0, MVT Ty1) const {
  2818:   // Compare the numbers of elements of two vector types, and widen the
  2819:   // narrower one to match the number of elements in the wider one.
  2820:   assert(Ty0.isVector() && Ty1.isVector());
  2821: 
  2822:   unsigned Len0 = Ty0.getVectorNumElements();
  2823:   unsigned Len1 = Ty1.getVectorNumElements();
  2824:   if (Len0 == Len1)
  2825:     return {Ty0, Ty1};
  2826: 
  2827:   unsigned MaxLen = std::max(Len0, Len1);
  2828:   return {MVT::getVectorVT(Ty0.getVectorElementType(), MaxLen),
  2829:           MVT::getVectorVT(Ty1.getVectorElementType(), MaxLen)};
  2830: }
  2831: 
  2832: MVT
  2833: HexagonTargetLowering::typeLegalize(MVT Ty, SelectionDAG &DAG) const {
  2834:   EVT LegalTy = getTypeToTransformTo(*DAG.getContext(), Ty);
  2835:   assert(LegalTy.isSimple());
  2836:   return LegalTy.getSimpleVT();
  2837: }
  2838: 
  2839: MVT
  2840: HexagonTargetLowering::typeWidenToHvx(MVT Ty) const {
  2841:   unsigned HwWidth = 8 * Subtarget.getVectorLength();
  2842:   assert(Ty.getSizeInBits() <= HwWidth);
  2843:   if (Ty.getSizeInBits() == HwWidth)
  2844:     return Ty;
  2845: 
  2846:   MVT ElemTy = Ty.getScalarType();
  2847:   return MVT::getVectorVT(ElemTy, HwWidth / ElemTy.getSizeInBits());
  2848: }
  2849: 
  2850: HexagonTargetLowering::VectorPair
  2851: HexagonTargetLowering::emitHvxAddWithOverflow(SDValue A, SDValue B,
  2852:       const SDLoc &dl, bool Signed, SelectionDAG &DAG) const {
  2853:   // Compute A+B, return {A+B, O}, where O = vector predicate indicating
  2854:   // whether an overflow has occurred.
  2855:   MVT ResTy = ty(A);
  2856:   assert(ResTy == ty(B));
  2857:   MVT PredTy = MVT::getVectorVT(MVT::i1, ResTy.getVectorNumElements());
  2858: 
  2859:   if (!Signed) {
  2860:     // V62+ has V6_vaddcarry, but it requires input predicate, so it doesn't
  2861:     // save any instructions.
  2862:     SDValue Add = DAG.getNode(ISD::ADD, dl, ResTy, {A, B});
  2863:     SDValue Ovf = DAG.getSetCC(dl, PredTy, Add, A, ISD::SETULT);
  2864:     return {Add, Ovf};
  2865:   }
  2866: 
  2867:   // Signed overflow has happened, if:
  2868:   // (A, B have the same sign) and (A+B has a different sign from either)
  2869:   // i.e. (~A xor B) & ((A+B) xor B), then check the sign bit
  2870:   SDValue Add = DAG.getNode(ISD::ADD, dl, ResTy, {A, B});
  2871:   SDValue NotA =
  2872:       DAG.getNode(ISD::XOR, dl, ResTy, {A, DAG.getAllOnesConstant(dl, ResTy)});
  2873:   SDValue Xor0 = DAG.getNode(ISD::XOR, dl, ResTy, {NotA, B});
  2874:   SDValue Xor1 = DAG.getNode(ISD::XOR, dl, ResTy, {Add, B});
  2875:   SDValue And = DAG.getNode(ISD::AND, dl, ResTy, {Xor0, Xor1});
  2876:   SDValue MSB =
  2877:       DAG.getSetCC(dl, PredTy, And, getZero(dl, ResTy, DAG), ISD::SETLT);
  2878:   return {Add, MSB};
  2879: }
  2880: 
  2881: HexagonTargetLowering::VectorPair
  2882: HexagonTargetLowering::emitHvxShiftRightRnd(SDValue Val, unsigned Amt,
  2883:       bool Signed, SelectionDAG &DAG) const {
  2884:   // Shift Val right by Amt bits, round the result to the nearest integer,
  2885:   // tie-break by rounding halves to even integer.
  2886: 
  2887:   const SDLoc &dl(Val);
  2888:   MVT ValTy = ty(Val);
  2889: 
  2890:   // This should also work for signed integers.
  2891:   //
  2892:   //   uint tmp0 = inp + ((1 << (Amt-1)) - 1);
  2893:   //   bool ovf = (inp > tmp0);
  2894:   //   uint rup = inp & (1 << (Amt+1));
  2895:   //
  2896:   //   uint tmp1 = inp >> (Amt-1);    // tmp1 == tmp2 iff
  2897:   //   uint tmp2 = tmp0 >> (Amt-1);   // the Amt-1 lower bits were all 0
  2898:   //   uint tmp3 = tmp2 + rup;
  2899:   //   uint frac = (tmp1 != tmp2) ? tmp2 >> 1 : tmp3 >> 1;
  2900:   unsigned ElemWidth = ValTy.getVectorElementType().getSizeInBits();
  2901:   MVT ElemTy = MVT::getIntegerVT(ElemWidth);
  2902:   MVT IntTy = tyVector(ValTy, ElemTy);
  2903:   MVT PredTy = MVT::getVectorVT(MVT::i1, IntTy.getVectorNumElements());
  2904:   unsigned ShRight = Signed ? ISD::SRA : ISD::SRL;
  2905: 
  2906:   SDValue Inp = DAG.getBitcast(IntTy, Val);
  2907:   SDValue LowBits = DAG.getConstant((1ull << (Amt - 1)) - 1, dl, IntTy);
  2908: 
  2909:   SDValue AmtP1 = DAG.getConstant(1ull << Amt, dl, IntTy);
  2910:   SDValue And = DAG.getNode(ISD::AND, dl, IntTy, {Inp, AmtP1});
  2911:   SDValue Zero = getZero(dl, IntTy, DAG);
  2912:   SDValue Bit = DAG.getSetCC(dl, PredTy, And, Zero, ISD::SETNE);
  2913:   SDValue Rup = DAG.getZExtOrTrunc(Bit, dl, IntTy);
  2914:   auto [Tmp0, Ovf] = emitHvxAddWithOverflow(Inp, LowBits, dl, Signed, DAG);
  2915: 
  2916:   SDValue AmtM1 = DAG.getConstant(Amt - 1, dl, IntTy);
  2917:   SDValue Tmp1 = DAG.getNode(ShRight, dl, IntTy, Inp, AmtM1);
  2918:   SDValue Tmp2 = DAG.getNode(ShRight, dl, IntTy, Tmp0, AmtM1);
  2919:   SDValue Tmp3 = DAG.getNode(ISD::ADD, dl, IntTy, Tmp2, Rup);
  2920: 
  2921:   SDValue Eq = DAG.getSetCC(dl, PredTy, Tmp1, Tmp2, ISD::SETEQ);
  2922:   SDValue One = DAG.getConstant(1, dl, IntTy);
  2923:   SDValue Tmp4 = DAG.getNode(ShRight, dl, IntTy, {Tmp2, One});
  2924:   SDValue Tmp5 = DAG.getNode(ShRight, dl, IntTy, {Tmp3, One});
  2925:   SDValue Mux = DAG.getNode(ISD::VSELECT, dl, IntTy, {Eq, Tmp5, Tmp4});
  2926:   return {Mux, Ovf};
  2927: }
  2928: 
  2929: SDValue
  2930: HexagonTargetLowering::emitHvxMulHsV60(SDValue A, SDValue B, const SDLoc &dl,
  2931:                                        SelectionDAG &DAG) const {
  2932:   MVT VecTy = ty(A);
  2933:   MVT PairTy = typeJoin({VecTy, VecTy});
  2934:   assert(VecTy.getVectorElementType() == MVT::i32);
  2935: 
  2936:   SDValue S16 = DAG.getConstant(16, dl, MVT::i32);
  2937: 
  2938:   // mulhs(A,B) =
  2939:   //   = [(Hi(A)*2^16 + Lo(A)) *s (Hi(B)*2^16 + Lo(B))] >> 32
  2940:   //   = [Hi(A)*2^16 *s Hi(B)*2^16 + Hi(A) *su Lo(B)*2^16
  2941:   //      + Lo(A) *us (Hi(B)*2^16 + Lo(B))] >> 32
  2942:   //   = [Hi(A) *s Hi(B)*2^32 + Hi(A) *su Lo(B)*2^16 + Lo(A) *us B] >> 32
  2943:   // The low half of Lo(A)*Lo(B) will be discarded (it's not added to
  2944:   // anything, so it cannot produce any carry over to higher bits),
  2945:   // so everything in [] can be shifted by 16 without loss of precision.
  2946:   //   = [Hi(A) *s Hi(B)*2^16 + Hi(A)*su Lo(B) + Lo(A)*B >> 16] >> 16
  2947:   //   = [Hi(A) *s Hi(B)*2^16 + Hi(A)*su Lo(B) + V6_vmpyewuh(A,B)] >> 16
  2948:   // The final additions need to make sure to properly maintain any carry-
  2949:   // out bits.
  2950:   //
  2951:   //                Hi(B) Lo(B)
  2952:   //                Hi(A) Lo(A)
  2953:   //               --------------
  2954:   //                Lo(B)*Lo(A)  | T0 = V6_vmpyewuh(B,A) does this,
  2955:   //         Hi(B)*Lo(A)         |      + dropping the low 16 bits
  2956:   //         Hi(A)*Lo(B)   | T2
  2957:   //  Hi(B)*Hi(A)
  2958: 
  2959:   SDValue T0 = getInstr(Hexagon::V6_vmpyewuh, dl, VecTy, {B, A}, DAG);
  2960:   // T1 = get Hi(A) into low halves.
  2961:   SDValue T1 = getInstr(Hexagon::V6_vasrw, dl, VecTy, {A, S16}, DAG);
  2962:   // P0 = interleaved T1.h*B.uh (full precision product)
  2963:   SDValue P0 = getInstr(Hexagon::V6_vmpyhus, dl, PairTy, {T1, B}, DAG);
  2964:   // T2 = T1.even(h) * B.even(uh), i.e. Hi(A)*Lo(B)
  2965:   SDValue T2 = LoHalf(P0, DAG);
  2966:   // We need to add T0+T2, recording the carry-out, which will be 1<<16
  2967:   // added to the final sum.
  2968:   // P1 = interleaved even/odd 32-bit (unsigned) sums of 16-bit halves
  2969:   SDValue P1 = getInstr(Hexagon::V6_vadduhw, dl, PairTy, {T0, T2}, DAG);
  2970:   // P2 = interleaved even/odd 32-bit (signed) sums of 16-bit halves
  2971:   SDValue P2 = getInstr(Hexagon::V6_vaddhw, dl, PairTy, {T0, T2}, DAG);
  2972:   // T3 = full-precision(T0+T2) >> 16
  2973:   // The low halves are added-unsigned, the high ones are added-signed.
  2974:   SDValue T3 = getInstr(Hexagon::V6_vasrw_acc, dl, VecTy,
  2975:                         {HiHalf(P2, DAG), LoHalf(P1, DAG), S16}, DAG);
  2976:   SDValue T4 = getInstr(Hexagon::V6_vasrw, dl, VecTy, {B, S16}, DAG);
  2977:   // P3 = interleaved Hi(B)*Hi(A) (full precision),
  2978:   // which is now Lo(T1)*Lo(T4), so we want to keep the even product.
  2979:   SDValue P3 = getInstr(Hexagon::V6_vmpyhv, dl, PairTy, {T1, T4}, DAG);
  2980:   SDValue T5 = LoHalf(P3, DAG);
  2981:   // Add:
  2982:   SDValue T6 = DAG.getNode(ISD::ADD, dl, VecTy, {T3, T5});
  2983:   return T6;
  2984: }
  2985: 
  2986: SDValue
  2987: HexagonTargetLowering::emitHvxMulLoHiV60(SDValue A, bool SignedA, SDValue B,
  2988:                                          bool SignedB, const SDLoc &dl,
  2989:                                          SelectionDAG &DAG) const {
  2990:   MVT VecTy = ty(A);
  2991:   MVT PairTy = typeJoin({VecTy, VecTy});
  2992:   assert(VecTy.getVectorElementType() == MVT::i32);
  2993: 
  2994:   SDValue S16 = DAG.getConstant(16, dl, MVT::i32);
  2995: 
  2996:   if (SignedA && !SignedB) {
  2997:     // Make A:unsigned, B:signed.
  2998:     std::swap(A, B);
  2999:     std::swap(SignedA, SignedB);
  3000:   }
```
- EN: It declares or implements routines such as getScalarWithWidth, getVectorNumElements, assert, HexagonTargetLowering::typeWidenToWider, ... (31 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getScalarWithWidth, getVectorNumElements, assert, HexagonTargetLowering::typeWidenToWider, ... (31 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering，说明了它与同级后端组件的连接关系。

### Lines 3001-3200 / 第 3001-3200 行

```cpp
  3001: 
  3002:   // Do halfword-wise multiplications for unsigned*unsigned product, then
  3003:   // add corrections for signed and unsigned*signed.
  3004: 
  3005:   SDValue Lo, Hi;
  3006: 
  3007:   // P0:lo = (uu) products of low halves of A and B,
  3008:   // P0:hi = (uu) products of high halves.
  3009:   SDValue P0 = getInstr(Hexagon::V6_vmpyuhv, dl, PairTy, {A, B}, DAG);
  3010: 
  3011:   // Swap low/high halves in B
  3012:   SDValue T0 = getInstr(Hexagon::V6_lvsplatw, dl, VecTy,
  3013:                         {DAG.getConstant(0x02020202, dl, MVT::i32)}, DAG);
  3014:   SDValue T1 = getInstr(Hexagon::V6_vdelta, dl, VecTy, {B, T0}, DAG);
  3015:   // P1 = products of even/odd halfwords.
  3016:   // P1:lo = (uu) products of even(A.uh) * odd(B.uh)
  3017:   // P1:hi = (uu) products of odd(A.uh) * even(B.uh)
  3018:   SDValue P1 = getInstr(Hexagon::V6_vmpyuhv, dl, PairTy, {A, T1}, DAG);
  3019: 
  3020:   // P2:lo = low halves of P1:lo + P1:hi,
  3021:   // P2:hi = high halves of P1:lo + P1:hi.
  3022:   SDValue P2 = getInstr(Hexagon::V6_vadduhw, dl, PairTy,
  3023:                         {HiHalf(P1, DAG), LoHalf(P1, DAG)}, DAG);
  3024:   // Still need to add the high halves of P0:lo to P2:lo
  3025:   SDValue T2 =
  3026:       getInstr(Hexagon::V6_vlsrw, dl, VecTy, {LoHalf(P0, DAG), S16}, DAG);
  3027:   SDValue T3 = DAG.getNode(ISD::ADD, dl, VecTy, {LoHalf(P2, DAG), T2});
  3028: 
  3029:   // The high halves of T3 will contribute to the HI part of LOHI.
  3030:   SDValue T4 = getInstr(Hexagon::V6_vasrw_acc, dl, VecTy,
  3031:                         {HiHalf(P2, DAG), T3, S16}, DAG);
  3032: 
  3033:   // The low halves of P2 need to be added to high halves of the LO part.
  3034:   Lo = getInstr(Hexagon::V6_vaslw_acc, dl, VecTy,
  3035:                 {LoHalf(P0, DAG), LoHalf(P2, DAG), S16}, DAG);
  3036:   Hi = DAG.getNode(ISD::ADD, dl, VecTy, {HiHalf(P0, DAG), T4});
  3037: 
  3038:   if (SignedA) {
  3039:     assert(SignedB && "Signed A and unsigned B should have been inverted");
  3040: 
  3041:     MVT PredTy = MVT::getVectorVT(MVT::i1, VecTy.getVectorNumElements());
  3042:     SDValue Zero = getZero(dl, VecTy, DAG);
  3043:     SDValue Q0 = DAG.getSetCC(dl, PredTy, A, Zero, ISD::SETLT);
  3044:     SDValue Q1 = DAG.getSetCC(dl, PredTy, B, Zero, ISD::SETLT);
  3045:     SDValue X0 = DAG.getNode(ISD::VSELECT, dl, VecTy, {Q0, B, Zero});
  3046:     SDValue X1 = getInstr(Hexagon::V6_vaddwq, dl, VecTy, {Q1, X0, A}, DAG);
  3047:     Hi = getInstr(Hexagon::V6_vsubw, dl, VecTy, {Hi, X1}, DAG);
  3048:   } else if (SignedB) {
  3049:     // Same correction as for mulhus:
  3050:     // mulhus(A.uw,B.w) = mulhu(A.uw,B.uw) - (A.w if B < 0)
  3051:     MVT PredTy = MVT::getVectorVT(MVT::i1, VecTy.getVectorNumElements());
  3052:     SDValue Zero = getZero(dl, VecTy, DAG);
  3053:     SDValue Q1 = DAG.getSetCC(dl, PredTy, B, Zero, ISD::SETLT);
  3054:     Hi = getInstr(Hexagon::V6_vsubwq, dl, VecTy, {Q1, Hi, A}, DAG);
  3055:   } else {
  3056:     assert(!SignedA && !SignedB);
  3057:   }
  3058: 
  3059:   return DAG.getMergeValues({Lo, Hi}, dl);
  3060: }
  3061: 
  3062: SDValue
  3063: HexagonTargetLowering::emitHvxMulLoHiV62(SDValue A, bool SignedA,
  3064:                                          SDValue B, bool SignedB,
  3065:                                          const SDLoc &dl,
  3066:                                          SelectionDAG &DAG) const {
  3067:   MVT VecTy = ty(A);
  3068:   MVT PairTy = typeJoin({VecTy, VecTy});
  3069:   assert(VecTy.getVectorElementType() == MVT::i32);
  3070: 
  3071:   if (SignedA && !SignedB) {
  3072:     // Make A:unsigned, B:signed.
  3073:     std::swap(A, B);
  3074:     std::swap(SignedA, SignedB);
  3075:   }
  3076: 
  3077:   // Do S*S first, then make corrections for U*S or U*U if needed.
  3078:   SDValue P0 = getInstr(Hexagon::V6_vmpyewuh_64, dl, PairTy, {A, B}, DAG);
  3079:   SDValue P1 =
  3080:       getInstr(Hexagon::V6_vmpyowh_64_acc, dl, PairTy, {P0, A, B}, DAG);
  3081:   SDValue Lo = LoHalf(P1, DAG);
  3082:   SDValue Hi = HiHalf(P1, DAG);
  3083: 
  3084:   if (!SignedB) {
  3085:     assert(!SignedA && "Signed A and unsigned B should have been inverted");
  3086:     SDValue Zero = getZero(dl, VecTy, DAG);
  3087:     MVT PredTy = MVT::getVectorVT(MVT::i1, VecTy.getVectorNumElements());
  3088: 
  3089:     // Mulhu(X, Y) = Mulhs(X, Y) + (X, if Y < 0) + (Y, if X < 0).
  3090:     // def: Pat<(VecI32 (mulhu HVI32:$A, HVI32:$B)),
  3091:     //          (V6_vaddw (HiHalf (Muls64O $A, $B)),
  3092:     //                    (V6_vaddwq (V6_vgtw (V6_vd0), $B),
  3093:     //                               (V6_vandvqv (V6_vgtw (V6_vd0), $A), $B),
  3094:     //                               $A))>;
  3095:     SDValue Q0 = DAG.getSetCC(dl, PredTy, A, Zero, ISD::SETLT);
  3096:     SDValue Q1 = DAG.getSetCC(dl, PredTy, B, Zero, ISD::SETLT);
  3097:     SDValue T0 = getInstr(Hexagon::V6_vandvqv, dl, VecTy, {Q0, B}, DAG);
  3098:     SDValue T1 = getInstr(Hexagon::V6_vaddwq, dl, VecTy, {Q1, T0, A}, DAG);
  3099:     Hi = getInstr(Hexagon::V6_vaddw, dl, VecTy, {Hi, T1}, DAG);
  3100:   } else if (!SignedA) {
  3101:     SDValue Zero = getZero(dl, VecTy, DAG);
  3102:     MVT PredTy = MVT::getVectorVT(MVT::i1, VecTy.getVectorNumElements());
  3103: 
  3104:     // Mulhus(unsigned X, signed Y) = Mulhs(X, Y) + (Y, if X < 0).
  3105:     // def: Pat<(VecI32 (HexagonMULHUS HVI32:$A, HVI32:$B)),
  3106:     //          (V6_vaddwq (V6_vgtw (V6_vd0), $A),
  3107:     //                     (HiHalf (Muls64O $A, $B)),
  3108:     //                     $B)>;
  3109:     SDValue Q0 = DAG.getSetCC(dl, PredTy, A, Zero, ISD::SETLT);
  3110:     Hi = getInstr(Hexagon::V6_vaddwq, dl, VecTy, {Q0, Hi, B}, DAG);
  3111:   }
  3112: 
  3113:   return DAG.getMergeValues({Lo, Hi}, dl);
  3114: }
  3115: 
  3116: SDValue
  3117: HexagonTargetLowering::EqualizeFpIntConversion(SDValue Op, SelectionDAG &DAG)
  3118:       const {
  3119:   // Rewrite conversion between integer and floating-point in such a way that
  3120:   // the integer type is extended/narrowed to match the bitwidth of the
  3121:   // floating-point type, combined with additional integer-integer extensions
  3122:   // or narrowings to match the original input/result types.
  3123:   // E.g.  f32 -> i8  ==>  f32 -> i32 -> i8
  3124:   //
  3125:   // The input/result types are not required to be legal, but if they are
  3126:   // legal, this function should not introduce illegal types.
  3127: 
  3128:   unsigned Opc = Op.getOpcode();
  3129:   assert(Opc == ISD::FP_TO_SINT || Opc == ISD::FP_TO_UINT ||
  3130:          Opc == ISD::SINT_TO_FP || Opc == ISD::UINT_TO_FP);
  3131: 
  3132:   SDValue Inp = Op.getOperand(0);
  3133:   MVT InpTy = ty(Inp);
  3134:   MVT ResTy = ty(Op);
  3135: 
  3136:   if (InpTy == ResTy)
  3137:     return Op;
  3138: 
  3139:   const SDLoc &dl(Op);
  3140:   bool Signed = Opc == ISD::FP_TO_SINT || Opc == ISD::SINT_TO_FP;
  3141: 
  3142:   auto [WInpTy, WResTy] = typeExtendToWider(InpTy, ResTy);
  3143:   SDValue WInp = resizeToWidth(Inp, WInpTy, Signed, dl, DAG);
  3144:   SDValue Conv = DAG.getNode(Opc, dl, WResTy, WInp);
  3145:   SDValue Res = resizeToWidth(Conv, ResTy, Signed, dl, DAG);
  3146:   return Res;
  3147: }
  3148: 
  3149: SDValue
  3150: HexagonTargetLowering::ExpandHvxFpToInt(SDValue Op, SelectionDAG &DAG) const {
  3151:   unsigned Opc = Op.getOpcode();
  3152:   assert(Opc == ISD::FP_TO_SINT || Opc == ISD::FP_TO_UINT);
  3153: 
  3154:   const SDLoc &dl(Op);
  3155:   SDValue Op0 = Op.getOperand(0);
  3156:   MVT InpTy = ty(Op0);
  3157:   MVT ResTy = ty(Op);
  3158:   assert(InpTy.changeTypeToInteger() == ResTy);
  3159: 
  3160:   // At this point this is an experiment under a flag.
  3161:   // In arch before V81 the rounding mode is towards nearest value.
  3162:   // The C/C++ standard requires rounding towards zero:
  3163:   // C (C99 and later): ISO/IEC 9899:2018 (C18), section 6.3.1.4 — "When a
  3164:   // finite value of real floating type is converted to an integer type, the
  3165:   // fractional part is discarded (i.e., the value is truncated toward zero)."
  3166:   // C++: ISO/IEC 14882:2020 (C++20), section 7.3.7 — "A prvalue of a
  3167:   // floating-point type can be converted to a prvalue of an integer type. The
  3168:   // conversion truncates; that is, the fractional part is discarded."
  3169:   if (InpTy == MVT::v64f16) {
  3170:     if (Subtarget.useHVXV81Ops()) {
  3171:       // This is c/c++ compliant
  3172:       SDValue ConvVec =
  3173:           getInstr(Hexagon::V6_vconv_h_hf_rnd, dl, ResTy, {Op0}, DAG);
  3174:       return ConvVec;
  3175:     } else if (EnableFpFastConvert) {
  3176:       // Vd32.h=Vu32.hf same as Q6_Vh_equals_Vhf
  3177:       SDValue ConvVec = getInstr(Hexagon::V6_vconv_h_hf, dl, ResTy, {Op0}, DAG);
  3178:       return ConvVec;
  3179:     }
  3180:   } else if (EnableFpFastConvert && InpTy == MVT::v32f32) {
  3181:     // Vd32.w=Vu32.sf same as Q6_Vw_equals_Vsf
  3182:     SDValue ConvVec = getInstr(Hexagon::V6_vconv_w_sf, dl, ResTy, {Op0}, DAG);
  3183:     return ConvVec;
  3184:   }
  3185: 
  3186:   // int32_t conv_f32_to_i32(uint32_t inp) {
  3187:   //   // s | exp8 | frac23
  3188:   //
  3189:   //   int neg = (int32_t)inp < 0;
  3190:   //
  3191:   //   // "expm1" is the actual exponent minus 1: instead of "bias", subtract
  3192:   //   // "bias+1". When the encoded exp is "all-1" (i.e. inf/nan), this will
  3193:   //   // produce a large positive "expm1", which will result in max u/int.
  3194:   //   // In all IEEE formats, bias is the largest positive number that can be
  3195:   //   // represented in bias-width bits (i.e. 011..1).
  3196:   //   int32_t expm1 = (inp << 1) - 0x80000000;
  3197:   //   expm1 >>= 24;
  3198:   //
  3199:   //   // Always insert the "implicit 1". Subnormal numbers will become 0
  3200:   //   // regardless.
```
- EN: It declares or implements routines such as assert, MVT::getVectorVT, getZero, getSetCC, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonMULHUS, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 assert, MVT::getVectorVT, getZero, getSetCC, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonMULHUS，说明了它与同级后端组件的连接关系。

### Lines 3201-3400 / 第 3201-3400 行

```cpp
  3201:   //   uint32_t frac = (inp << 8) | 0x80000000;
  3202:   //
  3203:   //   // "frac" is the fraction part represented as Q1.31. If it was
  3204:   //   // interpreted as uint32_t, it would be the fraction part multiplied
  3205:   //   // by 2^31.
  3206:   //
  3207:   //   // Calculate the amount of right shift, since shifting further to the
  3208:   //   // left would lose significant bits. Limit it to 32, because we want
  3209:   //   // shifts by 32+ to produce 0, whereas V6_vlsrwv treats the shift
  3210:   //   // amount as a 6-bit signed value (so 33 is same as -31, i.e. shift
  3211:   //   // left by 31). "rsh" can be negative.
  3212:   //   int32_t rsh = min(31 - (expm1 + 1), 32);
  3213:   //
  3214:   //   frac >>= rsh;   // rsh == 32 will produce 0
  3215:   //
  3216:   //   // Everything up to this point is the same for conversion to signed
  3217:   //   // unsigned integer.
  3218:   //
  3219:   //   if (neg)                 // Only for signed int
  3220:   //     frac = -frac;          //
  3221:   //   if (rsh <= 0 && neg)     //   bound = neg ? 0x80000000 : 0x7fffffff
  3222:   //     frac = 0x80000000;     //   frac = rsh <= 0 ? bound : frac
  3223:   //   if (rsh <= 0 && !neg)    //
  3224:   //     frac = 0x7fffffff;     //
  3225:   //
  3226:   //   if (neg)                 // Only for unsigned int
  3227:   //     frac = 0;              //
  3228:   //   if (rsh < 0 && !neg)     //   frac = rsh < 0 ? 0x7fffffff : frac;
  3229:   //     frac = 0x7fffffff;     //   frac = neg ? 0 : frac;
  3230:   //
  3231:   //   return frac;
  3232:   // }
  3233: 
  3234:   MVT PredTy = MVT::getVectorVT(MVT::i1, ResTy.getVectorElementCount());
  3235: 
  3236:   // Zero = V6_vd0();
  3237:   // Neg = V6_vgtw(Zero, Inp);
  3238:   // One = V6_lvsplatw(1);
  3239:   // M80 = V6_lvsplatw(0x80000000);
  3240:   // Exp00 = V6_vaslwv(Inp, One);
  3241:   // Exp01 = V6_vsubw(Exp00, M80);
  3242:   // ExpM1 = V6_vasrw(Exp01, 24);
  3243:   // Frc00 = V6_vaslw(Inp, 8);
  3244:   // Frc01 = V6_vor(Frc00, M80);
  3245:   // Rsh00 = V6_vsubw(V6_lvsplatw(30), ExpM1);
  3246:   // Rsh01 = V6_vminw(Rsh00, V6_lvsplatw(32));
  3247:   // Frc02 = V6_vlsrwv(Frc01, Rsh01);
  3248: 
  3249:   // if signed int:
  3250:   // Bnd = V6_vmux(Neg, M80, V6_lvsplatw(0x7fffffff))
  3251:   // Pos = V6_vgtw(Rsh01, Zero);
  3252:   // Frc13 = V6_vsubw(Zero, Frc02);
  3253:   // Frc14 = V6_vmux(Neg, Frc13, Frc02);
  3254:   // Int = V6_vmux(Pos, Frc14, Bnd);
  3255:   //
  3256:   // if unsigned int:
  3257:   // Rsn = V6_vgtw(Zero, Rsh01)
  3258:   // Frc23 = V6_vmux(Rsn, V6_lvsplatw(0x7fffffff), Frc02)
  3259:   // Int = V6_vmux(Neg, Zero, Frc23)
  3260: 
  3261:   auto [ExpWidth, ExpBias, FracWidth] = getIEEEProperties(InpTy);
  3262:   unsigned ElemWidth = 1 + ExpWidth + FracWidth;
  3263:   assert((1ull << (ExpWidth - 1)) == (1 + ExpBias));
  3264: 
  3265:   SDValue Inp = DAG.getBitcast(ResTy, Op0);
  3266:   SDValue Zero = getZero(dl, ResTy, DAG);
  3267:   SDValue Neg = DAG.getSetCC(dl, PredTy, Inp, Zero, ISD::SETLT);
  3268:   SDValue M80 = DAG.getConstant(1ull << (ElemWidth - 1), dl, ResTy);
  3269:   SDValue M7F = DAG.getConstant((1ull << (ElemWidth - 1)) - 1, dl, ResTy);
  3270:   SDValue One = DAG.getConstant(1, dl, ResTy);
  3271:   SDValue Exp00 = DAG.getNode(ISD::SHL, dl, ResTy, {Inp, One});
  3272:   SDValue Exp01 = DAG.getNode(ISD::SUB, dl, ResTy, {Exp00, M80});
  3273:   SDValue MNE = DAG.getConstant(ElemWidth - ExpWidth, dl, ResTy);
  3274:   SDValue ExpM1 = DAG.getNode(ISD::SRA, dl, ResTy, {Exp01, MNE});
  3275: 
  3276:   SDValue ExpW = DAG.getConstant(ExpWidth, dl, ResTy);
  3277:   SDValue Frc00 = DAG.getNode(ISD::SHL, dl, ResTy, {Inp, ExpW});
  3278:   SDValue Frc01 = DAG.getNode(ISD::OR, dl, ResTy, {Frc00, M80});
  3279: 
  3280:   SDValue MN2 = DAG.getConstant(ElemWidth - 2, dl, ResTy);
  3281:   SDValue Rsh00 = DAG.getNode(ISD::SUB, dl, ResTy, {MN2, ExpM1});
  3282:   SDValue MW = DAG.getConstant(ElemWidth, dl, ResTy);
  3283:   SDValue Rsh01 = DAG.getNode(ISD::SMIN, dl, ResTy, {Rsh00, MW});
  3284:   SDValue Frc02 = DAG.getNode(ISD::SRL, dl, ResTy, {Frc01, Rsh01});
  3285: 
  3286:   SDValue Int;
  3287: 
  3288:   if (Opc == ISD::FP_TO_SINT) {
  3289:     SDValue Bnd = DAG.getNode(ISD::VSELECT, dl, ResTy, {Neg, M80, M7F});
  3290:     SDValue Pos = DAG.getSetCC(dl, PredTy, Rsh01, Zero, ISD::SETGT);
  3291:     SDValue Frc13 = DAG.getNode(ISD::SUB, dl, ResTy, {Zero, Frc02});
  3292:     SDValue Frc14 = DAG.getNode(ISD::VSELECT, dl, ResTy, {Neg, Frc13, Frc02});
  3293:     Int = DAG.getNode(ISD::VSELECT, dl, ResTy, {Pos, Frc14, Bnd});
  3294:   } else {
  3295:     assert(Opc == ISD::FP_TO_UINT);
  3296:     SDValue Rsn = DAG.getSetCC(dl, PredTy, Rsh01, Zero, ISD::SETLT);
  3297:     SDValue Frc23 = DAG.getNode(ISD::VSELECT, dl, ResTy, Rsn, M7F, Frc02);
  3298:     Int = DAG.getNode(ISD::VSELECT, dl, ResTy, Neg, Zero, Frc23);
  3299:   }
  3300: 
  3301:   return Int;
  3302: }
  3303: 
  3304: SDValue
  3305: HexagonTargetLowering::ExpandHvxIntToFp(SDValue Op, SelectionDAG &DAG) const {
  3306:   unsigned Opc = Op.getOpcode();
  3307:   assert(Opc == ISD::SINT_TO_FP || Opc == ISD::UINT_TO_FP);
  3308: 
  3309:   const SDLoc &dl(Op);
  3310:   SDValue Op0 = Op.getOperand(0);
  3311:   MVT InpTy = ty(Op0);
  3312:   MVT ResTy = ty(Op);
  3313:   assert(ResTy.changeTypeToInteger() == InpTy);
  3314: 
  3315:   // uint32_t vnoc1_rnd(int32_t w) {
  3316:   //   int32_t iszero = w == 0;
  3317:   //   int32_t isneg = w < 0;
  3318:   //   uint32_t u = __builtin_HEXAGON_A2_abs(w);
  3319:   //
  3320:   //   uint32_t norm_left = __builtin_HEXAGON_S2_cl0(u) + 1;
  3321:   //   uint32_t frac0 = (uint64_t)u << norm_left;
  3322:   //
  3323:   //   // Rounding:
  3324:   //   uint32_t frac1 = frac0 + ((1 << 8) - 1);
  3325:   //   uint32_t renorm = (frac0 > frac1);
  3326:   //   uint32_t rup = (int)(frac0 << 22) < 0;
  3327:   //
  3328:   //   uint32_t frac2 = frac0 >> 8;
  3329:   //   uint32_t frac3 = frac1 >> 8;
  3330:   //   uint32_t frac = (frac2 != frac3) ? frac3 >> 1 : (frac3 + rup) >> 1;
  3331:   //
  3332:   //   int32_t exp = 32 - norm_left + renorm + 127;
  3333:   //   exp <<= 23;
  3334:   //
  3335:   //   uint32_t sign = 0x80000000 * isneg;
  3336:   //   uint32_t f = sign | exp | frac;
  3337:   //   return iszero ? 0 : f;
  3338:   // }
  3339: 
  3340:   MVT PredTy = MVT::getVectorVT(MVT::i1, InpTy.getVectorElementCount());
  3341:   bool Signed = Opc == ISD::SINT_TO_FP;
  3342: 
  3343:   auto [ExpWidth, ExpBias, FracWidth] = getIEEEProperties(ResTy);
  3344:   unsigned ElemWidth = 1 + ExpWidth + FracWidth;
  3345: 
  3346:   SDValue Zero = getZero(dl, InpTy, DAG);
  3347:   SDValue One = DAG.getConstant(1, dl, InpTy);
  3348:   SDValue IsZero = DAG.getSetCC(dl, PredTy, Op0, Zero, ISD::SETEQ);
  3349:   SDValue Abs = Signed ? DAG.getNode(ISD::ABS, dl, InpTy, Op0) : Op0;
  3350:   SDValue Clz = DAG.getNode(ISD::CTLZ, dl, InpTy, Abs);
  3351:   SDValue NLeft = DAG.getNode(ISD::ADD, dl, InpTy, {Clz, One});
  3352:   SDValue Frac0 = DAG.getNode(ISD::SHL, dl, InpTy, {Abs, NLeft});
  3353: 
  3354:   auto [Frac, Ovf] = emitHvxShiftRightRnd(Frac0, ExpWidth + 1, false, DAG);
  3355:   if (Signed) {
  3356:     SDValue IsNeg = DAG.getSetCC(dl, PredTy, Op0, Zero, ISD::SETLT);
  3357:     SDValue M80 = DAG.getConstant(1ull << (ElemWidth - 1), dl, InpTy);
  3358:     SDValue Sign = DAG.getNode(ISD::VSELECT, dl, InpTy, {IsNeg, M80, Zero});
  3359:     Frac = DAG.getNode(ISD::OR, dl, InpTy, {Sign, Frac});
  3360:   }
  3361: 
  3362:   SDValue Rnrm = DAG.getZExtOrTrunc(Ovf, dl, InpTy);
  3363:   SDValue Exp0 = DAG.getConstant(ElemWidth + ExpBias, dl, InpTy);
  3364:   SDValue Exp1 = DAG.getNode(ISD::ADD, dl, InpTy, {Rnrm, Exp0});
  3365:   SDValue Exp2 = DAG.getNode(ISD::SUB, dl, InpTy, {Exp1, NLeft});
  3366:   SDValue Exp3 = DAG.getNode(ISD::SHL, dl, InpTy,
  3367:                              {Exp2, DAG.getConstant(FracWidth, dl, InpTy)});
  3368:   SDValue Flt0 = DAG.getNode(ISD::OR, dl, InpTy, {Frac, Exp3});
  3369:   SDValue Flt1 = DAG.getNode(ISD::VSELECT, dl, InpTy, {IsZero, Zero, Flt0});
  3370:   SDValue Flt = DAG.getBitcast(ResTy, Flt1);
  3371: 
  3372:   return Flt;
  3373: }
  3374: 
  3375: SDValue
  3376: HexagonTargetLowering::CreateTLWrapper(SDValue Op, SelectionDAG &DAG) const {
  3377:   unsigned Opc = Op.getOpcode();
  3378:   unsigned TLOpc;
  3379:   switch (Opc) {
  3380:   case ISD::ANY_EXTEND:
  3381:   case ISD::SIGN_EXTEND:
  3382:   case ISD::ZERO_EXTEND:
  3383:     TLOpc = HexagonISD::TL_EXTEND;
  3384:     break;
  3385:   case ISD::TRUNCATE:
  3386:     TLOpc = HexagonISD::TL_TRUNCATE;
  3387:     break;
  3388: #ifndef NDEBUG
  3389:     Op.dump(&DAG);
  3390: #endif
  3391:     llvm_unreachable("Unexpected operator");
  3392:   }
  3393: 
  3394:   const SDLoc &dl(Op);
  3395:   return DAG.getNode(TLOpc, dl, ty(Op), Op.getOperand(0),
  3396:                      DAG.getUNDEF(MVT::i128), // illegal type
  3397:                      DAG.getConstant(Opc, dl, MVT::i32));
  3398: }
  3399: 
  3400: SDValue
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as value, MVT::getVectorVT, V6_vd0, V6_vgtw, ... (32 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 value, MVT::getVectorVT, V6_vd0, V6_vgtw, ... (32 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 3401-3600 / 第 3401-3600 行

```cpp
  3401: HexagonTargetLowering::RemoveTLWrapper(SDValue Op, SelectionDAG &DAG) const {
  3402:   assert(Op.getOpcode() == HexagonISD::TL_EXTEND ||
  3403:          Op.getOpcode() == HexagonISD::TL_TRUNCATE);
  3404:   unsigned Opc = Op.getConstantOperandVal(2);
  3405:   return DAG.getNode(Opc, SDLoc(Op), ty(Op), Op.getOperand(0));
  3406: }
  3407: 
  3408: HexagonTargetLowering::VectorPair
  3409: HexagonTargetLowering::SplitVectorOp(SDValue Op, SelectionDAG &DAG) const {
  3410:   assert(!Op.isMachineOpcode());
  3411:   SmallVector<SDValue, 2> OpsL, OpsH;
  3412:   const SDLoc &dl(Op);
  3413: 
  3414:   auto SplitVTNode = [&DAG, this](const VTSDNode *N) {
  3415:     MVT Ty = typeSplit(N->getVT().getSimpleVT()).first;
  3416:     SDValue TV = DAG.getValueType(Ty);
  3417:     return std::make_pair(TV, TV);
  3418:   };
  3419: 
  3420:   for (SDValue A : Op.getNode()->ops()) {
  3421:     auto [Lo, Hi] =
  3422:         ty(A).isVector() ? opSplit(A, dl, DAG) : std::make_pair(A, A);
  3423:     // Special case for type operand.
  3424:     switch (Op.getOpcode()) {
  3425:       case ISD::SIGN_EXTEND_INREG:
  3426:       case HexagonISD::SSAT:
  3427:       case HexagonISD::USAT:
  3428:         if (const auto *N = dyn_cast<const VTSDNode>(A.getNode()))
  3429:           std::tie(Lo, Hi) = SplitVTNode(N);
  3430:       break;
  3431:     }
  3432:     OpsL.push_back(Lo);
  3433:     OpsH.push_back(Hi);
  3434:   }
  3435: 
  3436:   MVT ResTy = ty(Op);
  3437:   MVT HalfTy = typeSplit(ResTy).first;
  3438:   SDValue L = DAG.getNode(Op.getOpcode(), dl, HalfTy, OpsL);
  3439:   SDValue H = DAG.getNode(Op.getOpcode(), dl, HalfTy, OpsH);
  3440:   return {L, H};
  3441: }
  3442: 
  3443: SDValue
  3444: HexagonTargetLowering::SplitHvxMemOp(SDValue Op, SelectionDAG &DAG) const {
  3445:   auto *MemN = cast<MemSDNode>(Op.getNode());
  3446:   unsigned MemOpc = MemN->getOpcode();
  3447:   EVT MemTy = MemN->getMemoryVT();
  3448: 
  3449:   if ((MemOpc == ISD::STORE || MemOpc == ISD::LOAD) &&
  3450:       (!MemTy.isSimple() || !isHvxPairTy(MemTy.getSimpleVT())))
  3451:     return Op;
  3452: 
  3453:   EVT ValueType;
  3454:   if (MemOpc == ISD::STORE)
  3455:     ValueType = ty(cast<StoreSDNode>(Op)->getValue());
  3456:   else if (MemOpc == ISD::MSTORE)
  3457:     ValueType = ty(cast<MaskedStoreSDNode>(Op)->getValue());
  3458:   else // ISD::LOAD, ISD::MLOAD.
  3459:     ValueType = MemN->getValueType(0);
  3460: 
  3461:   EVT LoVT, HiVT;
  3462:   std::tie(LoVT, HiVT) = DAG.GetSplitDestVTs(ValueType);
  3463: 
  3464:   EVT LoMemVT, HiMemVT;
  3465:   bool HiIsEmpty = false;
  3466:   std::tie(LoMemVT, HiMemVT) =
  3467:       DAG.GetDependentSplitDestVTs(MemTy, LoVT, &HiIsEmpty);
  3468: 
  3469:   uint64_t LoSize = LoMemVT.getSizeInBits().getFixedValue() / 8;
  3470:   uint64_t HiSize = HiMemVT.getSizeInBits().getFixedValue() / 8;
  3471: 
  3472:   const SDLoc &dl(Op);
  3473:   SDValue Chain = MemN->getChain();
  3474:   SDValue Base0 = MemN->getBasePtr();
  3475:   SDValue Base1 =
  3476:       DAG.getMemBasePlusOffset(Base0, TypeSize::getFixed(LoSize), dl);
  3477: 
  3478:   MachineMemOperand *MOp0 = nullptr, *MOp1 = nullptr;
  3479:   if (MachineMemOperand *MMO = MemN->getMemOperand()) {
  3480:     MachineFunction &MF = DAG.getMachineFunction();
  3481:     auto MemSize = [=](uint64_t Size) {
  3482:       return (MemOpc == ISD::MLOAD || MemOpc == ISD::MSTORE)
  3483:                  ? (uint64_t)MemoryLocation::UnknownSize
  3484:                  : Size;
  3485:     };
  3486:     // MOp1 will not be used if HiIsEmpty for masked loads and stores (MLOAD and
  3487:     // MSTORE). Non-masked loads and store are always of double-vector size (see
  3488:     // isHvxPairTy() check above).
  3489:     MOp0 = MF.getMachineMemOperand(MMO, 0, MemSize(LoSize));
  3490:     MOp1 = MF.getMachineMemOperand(MMO, LoSize, MemSize(HiSize));
  3491:   }
  3492: 
  3493:   if (MemOpc == ISD::LOAD) {
  3494:     assert(cast<LoadSDNode>(Op)->isUnindexed());
  3495:     SDValue Load0 = DAG.getLoad(LoVT, dl, Chain, Base0, MOp0);
  3496:     SDValue Load1 = DAG.getLoad(HiVT, dl, Chain, Base1, MOp1);
  3497:     return DAG.getMergeValues(
  3498:         {DAG.getNode(ISD::CONCAT_VECTORS, dl, MemN->getValueType(0), Load0,
  3499:                      Load1),
  3500:          DAG.getNode(ISD::TokenFactor, dl, MVT::Other, Load0.getValue(1),
  3501:                      Load1.getValue(1))},
  3502:         dl);
  3503:   }
  3504:   if (MemOpc == ISD::STORE) {
  3505:     assert(cast<StoreSDNode>(Op)->isUnindexed());
  3506:     VectorPair Vals = opSplit(cast<StoreSDNode>(Op)->getValue(), dl, DAG);
  3507:     SDValue Store0 = DAG.getStore(Chain, dl, Vals.first, Base0, MOp0);
  3508:     SDValue Store1 = DAG.getStore(Chain, dl, Vals.second, Base1, MOp1);
  3509:     return DAG.getNode(ISD::TokenFactor, dl, MVT::Other, Store0, Store1);
  3510:   }
  3511: 
  3512:   assert(MemOpc == ISD::MLOAD || MemOpc == ISD::MSTORE);
  3513: 
  3514:   auto MaskN = cast<MaskedLoadStoreSDNode>(Op);
  3515:   assert(MaskN->isUnindexed());
  3516:   VectorPair Masks = opSplit(MaskN->getMask(), dl, DAG);
  3517:   SDValue Offset = DAG.getUNDEF(MVT::i32);
  3518: 
  3519:   if (MemOpc == ISD::MLOAD) {
  3520:     VectorPair Thru =
  3521:         opSplit(cast<MaskedLoadSDNode>(Op)->getPassThru(), dl, DAG);
  3522:     SDValue MLoad0 = DAG.getMaskedLoad(LoVT, dl, Chain, Base0, Offset,
  3523:                                        Masks.first, Thru.first, LoMemVT, MOp0,
  3524:                                        ISD::UNINDEXED, ISD::NON_EXTLOAD, false);
  3525: 
  3526:     // The hi masked load has zero storage size. We therefore simply set it to
  3527:     // the low masked load and rely on subsequent removal from the chain as it
  3528:     // is unused. See DAGTypeLegalizer::SplitVecRes_MLOAD() for the same logic.
  3529:     SDValue MLoad1 =
  3530:         HiIsEmpty ? MLoad0
  3531:                   : DAG.getMaskedLoad(HiVT, dl, Chain, Base1, Offset,
  3532:                                       Masks.second, Thru.second, HiMemVT, MOp1,
  3533:                                       ISD::UNINDEXED, ISD::NON_EXTLOAD, false);
  3534:     return DAG.getMergeValues(
  3535:         {DAG.getNode(ISD::CONCAT_VECTORS, dl, MemN->getValueType(0), MLoad0,
  3536:                      MLoad1),
  3537:          DAG.getNode(ISD::TokenFactor, dl, MVT::Other, MLoad0.getValue(1),
  3538:                      MLoad1.getValue(1))},
  3539:         dl);
  3540:   }
  3541:   if (MemOpc == ISD::MSTORE) {
  3542:     VectorPair Vals = opSplit(cast<MaskedStoreSDNode>(Op)->getValue(), dl, DAG);
  3543:     SDValue MStore0 =
  3544:         DAG.getMaskedStore(Chain, dl, Vals.first, Base0, Offset, Masks.first,
  3545:                            LoMemVT, MOp0, ISD::UNINDEXED, false, false);
  3546:     if (HiIsEmpty)
  3547:       return MStore0;
  3548:     SDValue MStore1 =
  3549:         DAG.getMaskedStore(Chain, dl, Vals.second, Base1, Offset, Masks.second,
  3550:                            HiMemVT, MOp1, ISD::UNINDEXED, false, false);
  3551:     return DAG.getNode(ISD::TokenFactor, dl, MVT::Other, MStore0, MStore1);
  3552:   }
  3553: 
  3554:   std::string Name = "Unexpected operation: " + Op->getOperationName(&DAG);
  3555:   llvm_unreachable(Name.c_str());
  3556: }
  3557: 
  3558: SDValue
  3559: HexagonTargetLowering::WidenHvxLoad(SDValue Op, SelectionDAG &DAG) const {
  3560:   const SDLoc &dl(Op);
  3561:   auto *LoadN = cast<LoadSDNode>(Op.getNode());
  3562:   assert(LoadN->isUnindexed() && "Not widening indexed loads yet");
  3563:   assert(LoadN->getMemoryVT().getVectorElementType() != MVT::i1 &&
  3564:          "Not widening loads of i1 yet");
  3565: 
  3566:   SDValue Chain = LoadN->getChain();
  3567:   SDValue Base = LoadN->getBasePtr();
  3568:   SDValue Offset = DAG.getUNDEF(MVT::i32);
  3569: 
  3570:   MVT ResTy = ty(Op);
  3571:   unsigned HwLen = Subtarget.getVectorLength();
  3572:   unsigned ResLen = ResTy.getStoreSize();
  3573:   assert(ResLen < HwLen && "vsetq(v1) prerequisite");
  3574: 
  3575:   MVT BoolTy = MVT::getVectorVT(MVT::i1, HwLen);
  3576:   SDValue Mask = getInstr(Hexagon::V6_pred_scalar2, dl, BoolTy,
  3577:                           {DAG.getConstant(ResLen, dl, MVT::i32)}, DAG);
  3578: 
  3579:   MVT LoadTy = MVT::getVectorVT(MVT::i8, HwLen);
  3580:   MachineFunction &MF = DAG.getMachineFunction();
  3581:   auto *MemOp = MF.getMachineMemOperand(LoadN->getMemOperand(), 0, HwLen);
  3582: 
  3583:   SDValue Load = DAG.getMaskedLoad(LoadTy, dl, Chain, Base, Offset, Mask,
  3584:                                    DAG.getUNDEF(LoadTy), LoadTy, MemOp,
  3585:                                    ISD::UNINDEXED, ISD::NON_EXTLOAD, false);
  3586:   SDValue Value = opCastElem(Load, ResTy.getVectorElementType(), DAG);
  3587:   return DAG.getMergeValues({Value, Load.getValue(1)}, dl);
  3588: }
  3589: 
  3590: SDValue
  3591: HexagonTargetLowering::WidenHvxStore(SDValue Op, SelectionDAG &DAG) const {
  3592:   const SDLoc &dl(Op);
  3593:   auto *StoreN = cast<StoreSDNode>(Op.getNode());
  3594:   assert(StoreN->isUnindexed() && "Not widening indexed stores yet");
  3595:   assert(StoreN->getMemoryVT().getVectorElementType() != MVT::i1 &&
  3596:          "Not widening stores of i1 yet");
  3597: 
  3598:   SDValue Chain = StoreN->getChain();
  3599:   SDValue Base = StoreN->getBasePtr();
  3600:   SDValue Offset = DAG.getUNDEF(MVT::i32);
```
- EN: It declares or implements routines such as HexagonTargetLowering::RemoveTLWrapper, assert, getConstantOperandVal, getNode, ... (39 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonTargetLowering::RemoveTLWrapper, assert, getConstantOperandVal, getNode, ... (39 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 3601-3800 / 第 3601-3800 行

```cpp
  3601: 
  3602:   SDValue Value = opCastElem(StoreN->getValue(), MVT::i8, DAG);
  3603:   MVT ValueTy = ty(Value);
  3604:   unsigned ValueLen = ValueTy.getVectorNumElements();
  3605:   unsigned HwLen = Subtarget.getVectorLength();
  3606:   assert(isPowerOf2_32(ValueLen));
  3607: 
  3608:   for (unsigned Len = ValueLen; Len < HwLen; ) {
  3609:     Value = opJoin({Value, DAG.getUNDEF(ty(Value))}, dl, DAG);
  3610:     Len = ty(Value).getVectorNumElements(); // This is Len *= 2
  3611:   }
  3612:   assert(ty(Value).getVectorNumElements() == HwLen);  // Paranoia
  3613: 
  3614:   assert(ValueLen < HwLen && "vsetq(v1) prerequisite");
  3615:   MVT BoolTy = MVT::getVectorVT(MVT::i1, HwLen);
  3616:   SDValue Mask = getInstr(Hexagon::V6_pred_scalar2, dl, BoolTy,
  3617:                           {DAG.getConstant(ValueLen, dl, MVT::i32)}, DAG);
  3618:   MachineFunction &MF = DAG.getMachineFunction();
  3619:   auto *MemOp = MF.getMachineMemOperand(StoreN->getMemOperand(), 0, HwLen);
  3620:   return DAG.getMaskedStore(Chain, dl, Value, Base, Offset, Mask, ty(Value),
  3621:                             MemOp, ISD::UNINDEXED, false, false);
  3622: }
  3623: 
  3624: SDValue
  3625: HexagonTargetLowering::WidenHvxSetCC(SDValue Op, SelectionDAG &DAG) const {
  3626:   const SDLoc &dl(Op);
  3627:   SDValue Op0 = Op.getOperand(0), Op1 = Op.getOperand(1);
  3628:   MVT ElemTy = ty(Op0).getVectorElementType();
  3629:   unsigned HwLen = Subtarget.getVectorLength();
  3630: 
  3631:   unsigned WideOpLen = (8 * HwLen) / ElemTy.getSizeInBits();
  3632:   assert(WideOpLen * ElemTy.getSizeInBits() == 8 * HwLen);
  3633:   MVT WideOpTy = MVT::getVectorVT(ElemTy, WideOpLen);
  3634:   if (!Subtarget.isHVXVectorType(WideOpTy, true))
  3635:     return SDValue();
  3636: 
  3637:   SDValue WideOp0 = appendUndef(Op0, WideOpTy, DAG);
  3638:   SDValue WideOp1 = appendUndef(Op1, WideOpTy, DAG);
  3639:   EVT ResTy =
  3640:       getSetCCResultType(DAG.getDataLayout(), *DAG.getContext(), WideOpTy);
  3641:   SDValue SetCC = DAG.getNode(ISD::SETCC, dl, ResTy,
  3642:                               {WideOp0, WideOp1, Op.getOperand(2)});
  3643: 
  3644:   EVT RetTy = typeLegalize(ty(Op), DAG);
  3645:   return DAG.getNode(ISD::EXTRACT_SUBVECTOR, dl, RetTy,
  3646:                      {SetCC, getZero(dl, MVT::i32, DAG)});
  3647: }
  3648: 
  3649: SDValue HexagonTargetLowering::WidenHvxTruncateToBool(SDValue Op,
  3650:                                                       SelectionDAG &DAG) const {
  3651:   // Handle truncation to boolean vector where the result boolean type
  3652:   // needs widening (e.g., v16i32 -> v16i1 where v16i1 is not a standard
  3653:   // HVX predicate type, or v16i8 -> v16i1 in 128-byte mode).
  3654:   // Widen the input to HVX width, perform the truncate to the widened
  3655:   // boolean type, then extract the result.
  3656:   const SDLoc &dl(Op);
  3657:   SDValue Inp = Op.getOperand(0);
  3658:   MVT InpTy = ty(Inp);
  3659:   MVT ResTy = ty(Op);
  3660: 
  3661:   assert(ResTy.getVectorElementType() == MVT::i1 &&
  3662:          "Expected boolean result type");
  3663: 
  3664:   MVT ElemTy = InpTy.getVectorElementType();
  3665:   unsigned HwLen = Subtarget.getVectorLength();
  3666: 
  3667:   // Calculate the widened input type that fills the HVX register.
  3668:   unsigned WideLen = (8 * HwLen) / ElemTy.getSizeInBits();
  3669:   MVT WideInpTy = MVT::getVectorVT(ElemTy, WideLen);
  3670:   if (!Subtarget.isHVXVectorType(WideInpTy, false))
  3671:     return SDValue();
  3672: 
  3673:   // Widen the input to HVX width.
  3674:   SDValue WideInp = appendUndef(Inp, WideInpTy, DAG);
  3675: 
  3676:   // Perform the truncate to widened boolean type.
  3677:   MVT WideBoolTy = MVT::getVectorVT(MVT::i1, WideLen);
  3678:   SDValue WideTrunc = DAG.getNode(ISD::TRUNCATE, dl, WideBoolTy, WideInp);
  3679: 
  3680:   // Extract the result.
  3681:   EVT RetTy = typeLegalize(ResTy, DAG);
  3682:   return DAG.getNode(ISD::EXTRACT_SUBVECTOR, dl, RetTy,
  3683:                      {WideTrunc, getZero(dl, MVT::i32, DAG)});
  3684: }
  3685: 
  3686: SDValue
  3687: HexagonTargetLowering::LowerHvxOperation(SDValue Op, SelectionDAG &DAG) const {
  3688:   unsigned Opc = Op.getOpcode();
  3689:   bool IsPairOp = isHvxPairTy(ty(Op)) ||
  3690:                   llvm::any_of(Op.getNode()->ops(), [this] (SDValue V) {
  3691:                     return isHvxPairTy(ty(V));
  3692:                   });
  3693: 
  3694:   if (IsPairOp) {
  3695:     switch (Opc) {
  3696:       default:
  3697:         break;
  3698:       case ISD::LOAD:
  3699:       case ISD::STORE:
  3700:       case ISD::MLOAD:
  3701:       case ISD::MSTORE:
  3702:         return SplitHvxMemOp(Op, DAG);
  3703:       case ISD::SINT_TO_FP:
  3704:       case ISD::UINT_TO_FP:
  3705:       case ISD::FP_TO_SINT:
  3706:       case ISD::FP_TO_UINT:
  3707:         if (ty(Op).getSizeInBits() == ty(Op.getOperand(0)).getSizeInBits())
  3708:           return opJoin(SplitVectorOp(Op, DAG), SDLoc(Op), DAG);
  3709:         break;
  3710:       case ISD::ABS:
  3711:       case ISD::CTPOP:
  3712:       case ISD::CTLZ:
  3713:       case ISD::CTTZ:
  3714:       case ISD::MUL:
  3715:       case ISD::FADD:
  3716:       case ISD::FSUB:
  3717:       case ISD::FMUL:
  3718:       case ISD::FMINIMUMNUM:
  3719:       case ISD::FMAXIMUMNUM:
  3720:       case ISD::MULHS:
  3721:       case ISD::MULHU:
  3722:       case ISD::AND:
  3723:       case ISD::OR:
  3724:       case ISD::XOR:
  3725:       case ISD::SRA:
  3726:       case ISD::SHL:
  3727:       case ISD::SRL:
  3728:       case ISD::FSHL:
  3729:       case ISD::FSHR:
  3730:       case ISD::SMIN:
  3731:       case ISD::SMAX:
  3732:       case ISD::UMIN:
  3733:       case ISD::UMAX:
  3734:       case ISD::SETCC:
  3735:       case ISD::VSELECT:
  3736:       case ISD::SIGN_EXTEND_INREG:
  3737:       case ISD::SPLAT_VECTOR:
  3738:         return opJoin(SplitVectorOp(Op, DAG), SDLoc(Op), DAG);
  3739:       case ISD::SIGN_EXTEND:
  3740:       case ISD::ZERO_EXTEND:
  3741:         // In general, sign- and zero-extends can't be split and still
  3742:         // be legal. The only exception is extending bool vectors.
  3743:         if (ty(Op.getOperand(0)).getVectorElementType() == MVT::i1)
  3744:           return opJoin(SplitVectorOp(Op, DAG), SDLoc(Op), DAG);
  3745:         break;
  3746:     }
  3747:   }
  3748: 
  3749:   switch (Opc) {
  3750:     default:
  3751:       break;
  3752:       // clang-format off
  3753:     case ISD::BUILD_VECTOR:            return LowerHvxBuildVector(Op, DAG);
  3754:     case ISD::SPLAT_VECTOR:            return LowerHvxSplatVector(Op, DAG);
  3755:     case ISD::CONCAT_VECTORS:          return LowerHvxConcatVectors(Op, DAG);
  3756:     case ISD::INSERT_SUBVECTOR:        return LowerHvxInsertSubvector(Op, DAG);
  3757:     case ISD::INSERT_VECTOR_ELT:       return LowerHvxInsertElement(Op, DAG);
  3758:     case ISD::EXTRACT_SUBVECTOR:       return LowerHvxExtractSubvector(Op, DAG);
  3759:     case ISD::EXTRACT_VECTOR_ELT:      return LowerHvxExtractElement(Op, DAG);
  3760:     case ISD::BITCAST:                 return LowerHvxBitcast(Op, DAG);
  3761:     case ISD::ANY_EXTEND:              return LowerHvxAnyExt(Op, DAG);
  3762:     case ISD::SIGN_EXTEND:             return LowerHvxSignExt(Op, DAG);
  3763:     case ISD::ZERO_EXTEND:             return LowerHvxZeroExt(Op, DAG);
  3764:     case ISD::CTTZ:                    return LowerHvxCttz(Op, DAG);
  3765:     case ISD::SELECT:                  return LowerHvxSelect(Op, DAG);
  3766:     case ISD::SRA:
  3767:     case ISD::SHL:
  3768:     case ISD::SRL:                     return LowerHvxShift(Op, DAG);
  3769:     case ISD::FSHL:
  3770:     case ISD::FSHR:                    return LowerHvxFunnelShift(Op, DAG);
  3771:     case ISD::MULHS:
  3772:     case ISD::MULHU:                   return LowerHvxMulh(Op, DAG);
  3773:     case ISD::SMUL_LOHI:
  3774:     case ISD::UMUL_LOHI:               return LowerHvxMulLoHi(Op, DAG);
  3775:     case ISD::ANY_EXTEND_VECTOR_INREG: return LowerHvxExtend(Op, DAG);
  3776:     case ISD::SETCC:
  3777:     case ISD::INTRINSIC_VOID:          return Op;
  3778:     case ISD::INTRINSIC_WO_CHAIN:      return LowerHvxIntrinsic(Op, DAG);
  3779:     case ISD::MLOAD:
  3780:     case ISD::MSTORE:                  return LowerHvxMaskedOp(Op, DAG);
  3781:     // Unaligned loads will be handled by the default lowering.
  3782:     case ISD::LOAD:                    return LowerHvxLoad(Op, DAG);
  3783:     case ISD::STORE:                   return LowerHvxStore(Op, DAG);
  3784:     case ISD::FP_EXTEND:               return LowerHvxFpExtend(Op, DAG);
  3785:     case ISD::FP_TO_SINT:
  3786:     case ISD::FP_TO_UINT:              return LowerHvxFpToInt(Op, DAG);
  3787:     case ISD::SINT_TO_FP:
  3788:     case ISD::UINT_TO_FP:              return LowerHvxIntToFp(Op, DAG);
  3789: 
  3790:     // Special nodes:
  3791:     case HexagonISD::SMUL_LOHI:
  3792:     case HexagonISD::UMUL_LOHI:
  3793:     case HexagonISD::USMUL_LOHI:       return LowerHvxMulLoHi(Op, DAG);
  3794: 
  3795:     case ISD::PARTIAL_REDUCE_SMLA:
  3796:     case ISD::PARTIAL_REDUCE_UMLA:
  3797:     case ISD::PARTIAL_REDUCE_SUMLA:
  3798:       return LowerHvxPartialReduceMLA(Op, DAG);
  3799:       // clang-format on
  3800:   }
```
- EN: It declares or implements routines such as opCastElem, ty, getVectorNumElements, getVectorLength, ... (51 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 opCastElem, ty, getVectorNumElements, getVectorLength, ... (51 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 3801-4000 / 第 3801-4000 行

```cpp
  3801: #ifndef NDEBUG
  3802:   Op.dumpr(&DAG);
  3803: #endif
  3804:   llvm_unreachable("Unhandled HVX operation");
  3805: }
  3806: 
  3807: SDValue
  3808: HexagonTargetLowering::ExpandHvxResizeIntoSteps(SDValue Op, SelectionDAG &DAG)
  3809:       const {
  3810:   // Rewrite the extension/truncation/saturation op into steps where each
  3811:   // step changes the type widths by a factor of 2.
  3812:   // E.g.  i8 -> i16 remains unchanged, but i8 -> i32  ==>  i8 -> i16 -> i32.
  3813:   //
  3814:   // Some of the vector types in Op may not be legal.
  3815: 
  3816:   unsigned Opc = Op.getOpcode();
  3817:   switch (Opc) {
  3818:     case HexagonISD::SSAT:
  3819:     case HexagonISD::USAT:
  3820:     case HexagonISD::TL_EXTEND:
  3821:     case HexagonISD::TL_TRUNCATE:
  3822:       break;
  3823:     case ISD::ANY_EXTEND:
  3824:     case ISD::ZERO_EXTEND:
  3825:     case ISD::SIGN_EXTEND:
  3826:     case ISD::TRUNCATE:
  3827:       llvm_unreachable("ISD:: ops will be auto-folded");
  3828:       break;
  3829: #ifndef NDEBUG
  3830:     Op.dump(&DAG);
  3831: #endif
  3832:     llvm_unreachable("Unexpected operation");
  3833:   }
  3834: 
  3835:   SDValue Inp = Op.getOperand(0);
  3836:   MVT InpTy = ty(Inp);
  3837:   MVT ResTy = ty(Op);
  3838: 
  3839:   unsigned InpWidth = InpTy.getVectorElementType().getSizeInBits();
  3840:   unsigned ResWidth = ResTy.getVectorElementType().getSizeInBits();
  3841:   assert(InpWidth != ResWidth);
  3842: 
  3843:   if (InpWidth == 2 * ResWidth || ResWidth == 2 * InpWidth)
  3844:     return Op;
  3845: 
  3846:   const SDLoc &dl(Op);
  3847:   unsigned NumElems = InpTy.getVectorNumElements();
  3848:   assert(NumElems == ResTy.getVectorNumElements());
  3849: 
  3850:   auto repeatOp = [&](unsigned NewWidth, SDValue Arg) {
  3851:     MVT Ty = MVT::getVectorVT(MVT::getIntegerVT(NewWidth), NumElems);
  3852:     switch (Opc) {
  3853:       case HexagonISD::SSAT:
  3854:       case HexagonISD::USAT:
  3855:         return DAG.getNode(Opc, dl, Ty, {Arg, DAG.getValueType(Ty)});
  3856:       case HexagonISD::TL_EXTEND:
  3857:       case HexagonISD::TL_TRUNCATE:
  3858:         return DAG.getNode(Opc, dl, Ty, {Arg, Op.getOperand(1), Op.getOperand(2)});
  3859:       default:
  3860:         llvm_unreachable("Unexpected opcode");
  3861:     }
  3862:   };
  3863: 
  3864:   SDValue S = Inp;
  3865:   if (InpWidth < ResWidth) {
  3866:     assert(ResWidth % InpWidth == 0 && isPowerOf2_32(ResWidth / InpWidth));
  3867:     while (InpWidth * 2 <= ResWidth)
  3868:       S = repeatOp(InpWidth *= 2, S);
  3869:   } else {
  3870:     // InpWidth > ResWidth
  3871:     assert(InpWidth % ResWidth == 0 && isPowerOf2_32(InpWidth / ResWidth));
  3872:     while (InpWidth / 2 >= ResWidth)
  3873:       S = repeatOp(InpWidth /= 2, S);
  3874:   }
  3875:   return S;
  3876: }
  3877: 
  3878: SDValue
  3879: HexagonTargetLowering::LegalizeHvxResize(SDValue Op, SelectionDAG &DAG) const {
  3880:   SDValue Inp0 = Op.getOperand(0);
  3881:   MVT InpTy = ty(Inp0);
  3882:   MVT ResTy = ty(Op);
  3883:   unsigned InpWidth = InpTy.getSizeInBits();
  3884:   unsigned ResWidth = ResTy.getSizeInBits();
  3885:   unsigned Opc = Op.getOpcode();
  3886: 
  3887:   if (shouldWidenToHvx(InpTy, DAG) || shouldWidenToHvx(ResTy, DAG)) {
  3888:     // First, make sure that the narrower type is widened to HVX.
  3889:     // This may cause the result to be wider than what the legalizer
  3890:     // expects, so insert EXTRACT_SUBVECTOR to bring it back to the
  3891:     // desired type.
  3892:     auto [WInpTy, WResTy] =
  3893:         InpWidth < ResWidth ? typeWidenToWider(typeWidenToHvx(InpTy), ResTy)
  3894:                             : typeWidenToWider(InpTy, typeWidenToHvx(ResTy));
  3895:     SDValue W = appendUndef(Inp0, WInpTy, DAG);
  3896:     SDValue S;
  3897:     if (Opc == HexagonISD::TL_EXTEND || Opc == HexagonISD::TL_TRUNCATE) {
  3898:       S = DAG.getNode(Opc, SDLoc(Op), WResTy, W, Op.getOperand(1),
  3899:                       Op.getOperand(2));
  3900:     } else {
  3901:       S = DAG.getNode(Opc, SDLoc(Op), WResTy, W, DAG.getValueType(WResTy));
  3902:     }
  3903:     SDValue T = ExpandHvxResizeIntoSteps(S, DAG);
  3904:     return extractSubvector(T, typeLegalize(ResTy, DAG), 0, DAG);
  3905:   } else if (shouldSplitToHvx(InpWidth < ResWidth ? ResTy : InpTy, DAG)) {
  3906:     // For multi-step extends/truncates (e.g., i8->i32), expand into
  3907:     // single-step operations first. Splitting a multi-step TL_EXTEND
  3908:     // would halve the operand type to a sub-HVX size (e.g., v128i8 ->
  3909:     // v64i8), creating illegal types that cause issues in the type
  3910:     // legalizer's map tracking. Single-step operations (e.g., i16->i32)
  3911:     // are safe to split because their halved operand types remain legal.
  3912:     SDValue T = ExpandHvxResizeIntoSteps(Op, DAG);
  3913:     if (T != Op)
  3914:       return T;
  3915:     return opJoin(SplitVectorOp(Op, DAG), SDLoc(Op), DAG);
  3916:   } else {
  3917:     assert(isTypeLegal(InpTy) && isTypeLegal(ResTy));
  3918:     return RemoveTLWrapper(Op, DAG);
  3919:   }
  3920:   llvm_unreachable("Unexpected situation");
  3921: }
  3922: 
  3923: void
  3924: HexagonTargetLowering::LowerHvxOperationWrapper(SDNode *N,
  3925:       SmallVectorImpl<SDValue> &Results, SelectionDAG &DAG) const {
  3926:   unsigned Opc = N->getOpcode();
  3927:   SDValue Op(N, 0);
  3928:   SDValue Inp0;   // Optional first argument.
  3929:   if (N->getNumOperands() > 0)
  3930:     Inp0 = Op.getOperand(0);
  3931: 
  3932:   switch (Opc) {
  3933:     case ISD::ANY_EXTEND:
  3934:     case ISD::SIGN_EXTEND:
  3935:     case ISD::ZERO_EXTEND:
  3936:       if (Subtarget.isHVXElementType(ty(Op)) &&
  3937:           Subtarget.isHVXElementType(ty(Inp0))) {
  3938:         Results.push_back(CreateTLWrapper(Op, DAG));
  3939:       }
  3940:       break;
  3941:     case ISD::TRUNCATE:
  3942:       // Handle truncate to boolean vector when the input is not a
  3943:       // standard HVX vector type (single or pair). This covers cases
  3944:       // where the input needs widening (e.g., v64i8 -> v64i1 in
  3945:       // 128-byte mode) and cases where the result boolean type itself
  3946:       // needs widening (e.g., v16i32 -> v16i1). When the input is
  3947:       // already an HVX type, tablegen patterns handle the truncation
  3948:       // directly (e.g., v64i16 -> v64i1 via V6_vandvrt).
  3949:       if (ty(Op).getVectorElementType() == MVT::i1 &&
  3950:           !Subtarget.isHVXVectorType(ty(Inp0), false)) {
  3951:         if (SDValue T = WidenHvxTruncateToBool(Op, DAG))
  3952:           Results.push_back(T);
  3953:       } else if (Subtarget.isHVXElementType(ty(Op)) &&
  3954:                  Subtarget.isHVXElementType(ty(Inp0))) {
  3955:         Results.push_back(CreateTLWrapper(Op, DAG));
  3956:       }
  3957:       break;
  3958:     case ISD::SETCC:
  3959:       if (shouldWidenToHvx(ty(Inp0), DAG)) {
  3960:         if (SDValue T = WidenHvxSetCC(Op, DAG))
  3961:           Results.push_back(T);
  3962:       }
  3963:       break;
  3964:     case ISD::STORE: {
  3965:       if (shouldWidenToHvx(ty(cast<StoreSDNode>(N)->getValue()), DAG)) {
  3966:         SDValue Store = WidenHvxStore(Op, DAG);
  3967:         Results.push_back(Store);
  3968:       }
  3969:       break;
  3970:     }
  3971:     case ISD::MLOAD:
  3972:       if (isHvxPairTy(ty(Op))) {
  3973:         SDValue S = SplitHvxMemOp(Op, DAG);
  3974:         assert(S->getOpcode() == ISD::MERGE_VALUES);
  3975:         Results.push_back(S.getOperand(0));
  3976:         Results.push_back(S.getOperand(1));
  3977:       }
  3978:       break;
  3979:     case ISD::MSTORE:
  3980:       if (isHvxPairTy(ty(Op->getOperand(1)))) {    // Stored value
  3981:         SDValue S = SplitHvxMemOp(Op, DAG);
  3982:         Results.push_back(S);
  3983:       }
  3984:       break;
  3985:     case ISD::SINT_TO_FP:
  3986:     case ISD::UINT_TO_FP:
  3987:     case ISD::FP_TO_SINT:
  3988:     case ISD::FP_TO_UINT:
  3989:       if (ty(Op).getSizeInBits() != ty(Inp0).getSizeInBits()) {
  3990:         SDValue T = EqualizeFpIntConversion(Op, DAG);
  3991:         Results.push_back(T);
  3992:       }
  3993:       break;
  3994:     case HexagonISD::SSAT:
  3995:     case HexagonISD::USAT:
  3996:     case HexagonISD::TL_EXTEND:
  3997:     case HexagonISD::TL_TRUNCATE:
  3998:       Results.push_back(LegalizeHvxResize(Op, DAG));
  3999:       break;
  4000:     default:
```
- EN: Header guards in this range prevent duplicate inclusion and define the interface boundary. It declares or implements routines such as dumpr, llvm_unreachable, HexagonTargetLowering::ExpandHvxResizeIntoSteps, getOpcode, ... (29 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage.
- CN: 这一段中的头文件保护宏用于防止重复包含，并明确接口边界。 这里声明或实现了 dumpr, llvm_unreachable, HexagonTargetLowering::ExpandHvxResizeIntoSteps, getOpcode, ... (29 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。

### Lines 4001-4200 / 第 4001-4200 行

```cpp
  4001:       break;
  4002:   }
  4003: }
  4004: 
  4005: void
  4006: HexagonTargetLowering::ReplaceHvxNodeResults(SDNode *N,
  4007:       SmallVectorImpl<SDValue> &Results, SelectionDAG &DAG) const {
  4008:   unsigned Opc = N->getOpcode();
  4009:   SDValue Op(N, 0);
  4010:   SDValue Inp0;   // Optional first argument.
  4011:   if (N->getNumOperands() > 0)
  4012:     Inp0 = Op.getOperand(0);
  4013: 
  4014:   switch (Opc) {
  4015:     case ISD::ANY_EXTEND:
  4016:     case ISD::SIGN_EXTEND:
  4017:     case ISD::ZERO_EXTEND:
  4018:       if (Subtarget.isHVXElementType(ty(Op)) &&
  4019:           Subtarget.isHVXElementType(ty(Inp0))) {
  4020:         Results.push_back(CreateTLWrapper(Op, DAG));
  4021:       }
  4022:       break;
  4023:     case ISD::TRUNCATE:
  4024:       // Handle truncate to boolean vector when the input is not a
  4025:       // standard HVX vector type. See comment in LowerHvxOperationWrapper.
  4026:       if (ty(Op).getVectorElementType() == MVT::i1 &&
  4027:           !Subtarget.isHVXVectorType(ty(Inp0), false)) {
  4028:         if (SDValue T = WidenHvxTruncateToBool(Op, DAG))
  4029:           Results.push_back(T);
  4030:       } else if (Subtarget.isHVXElementType(ty(Op)) &&
  4031:                  Subtarget.isHVXElementType(ty(Inp0))) {
  4032:         Results.push_back(CreateTLWrapper(Op, DAG));
  4033:       }
  4034:       break;
  4035:     case ISD::SETCC:
  4036:       if (shouldWidenToHvx(ty(Op), DAG)) {
  4037:         if (SDValue T = WidenHvxSetCC(Op, DAG))
  4038:           Results.push_back(T);
  4039:       }
  4040:       break;
  4041:     case ISD::LOAD: {
  4042:       if (shouldWidenToHvx(ty(Op), DAG)) {
  4043:         SDValue Load = WidenHvxLoad(Op, DAG);
  4044:         assert(Load->getOpcode() == ISD::MERGE_VALUES);
  4045:         Results.push_back(Load.getOperand(0));
  4046:         Results.push_back(Load.getOperand(1));
  4047:       }
  4048:       break;
  4049:     }
  4050:     case ISD::BITCAST:
  4051:       if (isHvxBoolTy(ty(Inp0))) {
  4052:         SDValue C = LowerHvxBitcast(Op, DAG);
  4053:         Results.push_back(C);
  4054:       }
  4055:       break;
  4056:     case ISD::FP_TO_SINT:
  4057:     case ISD::FP_TO_UINT:
  4058:       if (ty(Op).getSizeInBits() != ty(Inp0).getSizeInBits()) {
  4059:         SDValue T = EqualizeFpIntConversion(Op, DAG);
  4060:         Results.push_back(T);
  4061:       }
  4062:       break;
  4063:     case HexagonISD::SSAT:
  4064:     case HexagonISD::USAT:
  4065:     case HexagonISD::TL_EXTEND:
  4066:     case HexagonISD::TL_TRUNCATE:
  4067:       Results.push_back(LegalizeHvxResize(Op, DAG));
  4068:       break;
  4069:     default:
  4070:       break;
  4071:   }
  4072: }
  4073: 
  4074: SDValue
  4075: HexagonTargetLowering::combineTruncateBeforeLegal(SDValue Op,
  4076:                                                   DAGCombinerInfo &DCI) const {
  4077:   // Simplify V:v2NiB --(bitcast)--> vNi2B --(truncate)--> vNiB
  4078:   // to extract-subvector (shuffle V, pick even, pick odd)
  4079: 
  4080:   assert(Op.getOpcode() == ISD::TRUNCATE);
  4081:   SelectionDAG &DAG = DCI.DAG;
  4082:   const SDLoc &dl(Op);
  4083: 
  4084:   if (Op.getOperand(0).getOpcode() == ISD::BITCAST)
  4085:     return SDValue();
  4086:   SDValue Cast = Op.getOperand(0);
  4087:   SDValue Src = Cast.getOperand(0);
  4088: 
  4089:   EVT TruncTy = Op.getValueType();
  4090:   EVT CastTy = Cast.getValueType();
  4091:   EVT SrcTy = Src.getValueType();
  4092:   if (SrcTy.isSimple())
  4093:     return SDValue();
  4094:   if (SrcTy.getVectorElementType() != TruncTy.getVectorElementType())
  4095:     return SDValue();
  4096:   unsigned SrcLen = SrcTy.getVectorNumElements();
  4097:   unsigned CastLen = CastTy.getVectorNumElements();
  4098:   if (2 * CastLen != SrcLen)
  4099:     return SDValue();
  4100: 
  4101:   SmallVector<int, 128> Mask(SrcLen);
  4102:   for (int i = 0; i != static_cast<int>(CastLen); ++i) {
  4103:     Mask[i] = 2 * i;
  4104:     Mask[i + CastLen] = 2 * i + 1;
  4105:   }
  4106:   SDValue Deal =
  4107:       DAG.getVectorShuffle(SrcTy, dl, Src, DAG.getUNDEF(SrcTy), Mask);
  4108:   return opSplit(Deal, dl, DAG).first;
  4109: }
  4110: 
  4111: SDValue
  4112: HexagonTargetLowering::combineConcatOfShuffles(SDValue Op,
  4113:                                                SelectionDAG &DAG) const {
  4114:   // Fold
  4115:   //   concat (shuffle x, y, m1), (shuffle x, y, m2)
  4116:   // into
  4117:   //   shuffle (concat x, y), undef, m3
  4118:   if (Op.getNumOperands() != 2)
  4119:     return SDValue();
  4120: 
  4121:   const SDLoc &dl(Op);
  4122:   SDValue V0 = Op.getOperand(0);
  4123:   SDValue V1 = Op.getOperand(1);
  4124: 
  4125:   if (V0.getOpcode() != ISD::VECTOR_SHUFFLE)
  4126:     return SDValue();
  4127:   if (V1.getOpcode() != ISD::VECTOR_SHUFFLE)
  4128:     return SDValue();
  4129: 
  4130:   SetVector<SDValue> Order;
  4131:   Order.insert(V0.getOperand(0));
  4132:   Order.insert(V0.getOperand(1));
  4133:   Order.insert(V1.getOperand(0));
  4134:   Order.insert(V1.getOperand(1));
  4135: 
  4136:   if (Order.size() > 2)
  4137:     return SDValue();
  4138: 
  4139:   // In ISD::VECTOR_SHUFFLE, the types of each input and the type of the
  4140:   // result must be the same.
  4141:   EVT InpTy = V0.getValueType();
  4142:   assert(InpTy.isVector());
  4143:   unsigned InpLen = InpTy.getVectorNumElements();
  4144: 
  4145:   SmallVector<int, 128> LongMask;
  4146:   auto AppendToMask = [&](SDValue Shuffle) {
  4147:     auto *SV = cast<ShuffleVectorSDNode>(Shuffle.getNode());
  4148:     ArrayRef<int> Mask = SV->getMask();
  4149:     SDValue X = Shuffle.getOperand(0);
  4150:     SDValue Y = Shuffle.getOperand(1);
  4151:     for (int M : Mask) {
  4152:       if (M == -1) {
  4153:         LongMask.push_back(M);
  4154:         continue;
  4155:       }
  4156:       SDValue Src = static_cast<unsigned>(M) < InpLen ? X : Y;
  4157:       if (static_cast<unsigned>(M) >= InpLen)
  4158:         M -= InpLen;
  4159: 
  4160:       int OutOffset = Order[0] == Src ? 0 : InpLen;
  4161:       LongMask.push_back(M + OutOffset);
  4162:     }
  4163:   };
  4164: 
  4165:   AppendToMask(V0);
  4166:   AppendToMask(V1);
  4167: 
  4168:   SDValue C0 = Order.front();
  4169:   SDValue C1 = Order.back();  // Can be same as front
  4170:   EVT LongTy = InpTy.getDoubleNumVectorElementsVT(*DAG.getContext());
  4171: 
  4172:   SDValue Cat = DAG.getNode(ISD::CONCAT_VECTORS, dl, LongTy, {C0, C1});
  4173:   return DAG.getVectorShuffle(LongTy, dl, Cat, DAG.getUNDEF(LongTy), LongMask);
  4174: }
  4175: 
  4176: // Reassociate concat(p1, p2, ...) into
  4177: //   concat(concat(p1, ...), concat(pi, ...), ...)
  4178: // where each inner concat produces a predicate where each bit corresponds
  4179: // to at most BitBytes bytes.
  4180: // Concatenating predicates decreases the number of bytes per each predicate
  4181: // bit.
  4182: SDValue
  4183: HexagonTargetLowering::combineConcatOfScalarPreds(SDValue Op, unsigned BitBytes,
  4184:                                                   SelectionDAG &DAG) const {
  4185:   const SDLoc &dl(Op);
  4186:   SmallVector<SDValue> Ops(Op->ops());
  4187:   MVT ResTy = ty(Op);
  4188:   MVT InpTy = ty(Ops[0]);
  4189:   unsigned InpLen = InpTy.getVectorNumElements(); // Scalar predicate
  4190:   unsigned ResLen = ResTy.getVectorNumElements(); // HVX vector predicate
  4191:   assert(InpLen <= 8 && "Too long for scalar predicate");
  4192:   assert(ResLen > 8 && "Too short for HVX vector predicate");
  4193: 
  4194:   unsigned Bytes = 8 / InpLen; // Bytes-per-bit in input
  4195: 
  4196:   // Already in the right form?
  4197:   if (Bytes <= BitBytes)
  4198:     return Op;
  4199: 
  4200:   ArrayRef<SDValue> Inputs(Ops);
```
- EN: It declares or implements routines such as HexagonTargetLowering::ReplaceHvxNodeResults, getOpcode, Op, push_back, ... (29 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 HexagonTargetLowering::ReplaceHvxNodeResults, getOpcode, Op, push_back, ... (29 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonISD，说明了它与同级后端组件的连接关系。

### Lines 4201-4400 / 第 4201-4400 行

```cpp
  4201:   unsigned SliceLen = Bytes / BitBytes;
  4202: 
  4203:   SmallVector<SDValue> Cats;
  4204:   // (8 / BitBytes) is the desired length of the result of the inner concat.
  4205:   MVT InnerTy = MVT::getVectorVT(MVT::i1, 8 / BitBytes);
  4206:   for (unsigned i = 0; i != ResLen / (8 / BitBytes); ++i) {
  4207:     SDValue Cat = DAG.getNode(ISD::CONCAT_VECTORS, dl, InnerTy,
  4208:                               Inputs.slice(SliceLen * i, SliceLen));
  4209:     Cats.push_back(Cat);
  4210:   }
  4211: 
  4212:   return DAG.getNode(ISD::CONCAT_VECTORS, dl, ResTy, Cats);
  4213: }
  4214: 
  4215: SDValue HexagonTargetLowering::combineConcatVectorsBeforeLegal(
  4216:     SDValue Op, DAGCombinerInfo &DCI) const {
  4217:   MVT ResTy = ty(Op);
  4218:   MVT ElemTy = ResTy.getVectorElementType();
  4219: 
  4220:   if (ElemTy != MVT::i1) {
  4221:     return combineConcatOfShuffles(Op, DCI.DAG);
  4222:   }
  4223:   return SDValue();
  4224: }
  4225: 
  4226: // Create the inner partial reduction MLA that can be efficiently lowered. This
  4227: // function is used by partial and full reductions.
  4228: SDValue HexagonTargetLowering::createExtendingPartialReduceMLA(
  4229:     unsigned Opcode, EVT AccEltType, unsigned AccNumElements, EVT InputType,
  4230:     const SDValue &A, const SDValue &B, unsigned &RemainingReductionRatio,
  4231:     const SDLoc &DL, SelectionDAG &DAG) const {
  4232:   const auto &Subtarget = DAG.getSubtarget<HexagonSubtarget>();
  4233:   if (!Subtarget.useHVXOps())
  4234:     return SDValue();
  4235: 
  4236:   EVT InputEltType = InputType.getVectorElementType();
  4237: 
  4238:   // Find if an optimized instruction for the sub-reduction is available.
  4239:   unsigned NativeRatio;
  4240:   if (AccEltType == MVT::i32 && InputEltType == MVT::i8)
  4241:     NativeRatio = 4;
  4242:   else
  4243:     return SDValue();
  4244: 
  4245:   // We only handle the case when additional reduction will be needed, i.e.
  4246:   // input is longer by a larger factor than the result.
  4247:   ElementCount InputEC = InputType.getVectorElementCount();
  4248:   if (!InputEC.isKnownMultipleOf(AccNumElements * NativeRatio))
  4249:     return SDValue();
  4250: 
  4251:   unsigned InputNumElements = InputEC.getFixedValue();
  4252:   RemainingReductionRatio = InputNumElements / (AccNumElements * NativeRatio);
  4253:   if (RemainingReductionRatio == 1)
  4254:     return SDValue();
  4255: 
  4256:   // Create a reduction by the natively supported factor.
  4257:   EVT IntermediateType = EVT::getVectorVT(*DAG.getContext(), AccEltType,
  4258:                                           InputNumElements / NativeRatio);
  4259: 
  4260:   SDValue Zero = DAG.getConstant(0, DL, IntermediateType);
  4261:   return DAG.getNode(Opcode, DL, IntermediateType, Zero, A, B);
  4262: }
  4263: 
  4264: static bool DetectExtendingMultiply(const SDValue &N, EVT ScalarType,
  4265:                                     unsigned &Opcode, SDValue &A, SDValue &B) {
  4266:   SDValue Mul = N;
  4267:   EVT AccType = Mul.getValueType(); // Vector input type after extension.
  4268:   if (ScalarType != AccType.getVectorElementType())
  4269:     return false;
  4270:   bool swap = false;
  4271:   if (Mul->getOpcode() != ISD::MUL)
  4272:     return false;
  4273:   A = Mul->getOperand(0);
  4274:   B = Mul->getOperand(1);
  4275:   if (A.getOpcode() == ISD::ZERO_EXTEND) {
  4276:     if (B.getOpcode() == ISD::ZERO_EXTEND)
  4277:       Opcode = ISD::PARTIAL_REDUCE_UMLA;
  4278:     else if (B.getOpcode() == ISD::SIGN_EXTEND) {
  4279:       swap = true;
  4280:       Opcode = ISD::PARTIAL_REDUCE_SUMLA;
  4281:     } else
  4282:       return false;
  4283:   } else if (A.getOpcode() == ISD::SIGN_EXTEND) {
  4284:     if (B.getOpcode() == ISD::ZERO_EXTEND)
  4285:       Opcode = ISD::PARTIAL_REDUCE_SUMLA;
  4286:     else if (B.getOpcode() == ISD::SIGN_EXTEND)
  4287:       Opcode = ISD::PARTIAL_REDUCE_SMLA;
  4288:     else
  4289:       return false;
  4290:   } else
  4291:     return false;
  4292: 
  4293:   // Get multiplication arguments before extension.
  4294:   A = A->getOperand(0);
  4295:   B = B->getOperand(0);
  4296:   if (A.getValueType() != B.getValueType())
  4297:     return false;
  4298: 
  4299:   if (swap)
  4300:     std::swap(A, B);
  4301: 
  4302:   return true;
  4303: }
  4304: 
  4305: SDValue HexagonTargetLowering::splitVecReduceAdd(SDNode *N,
  4306:                                                  SelectionDAG &DAG) const {
  4307:   if (!Subtarget.useHVXOps())
  4308:     return SDValue();
  4309: 
  4310:   EVT ScalarType = N->getValueType(0);
  4311:   unsigned Opcode;
  4312:   SDValue A, B;
  4313:   if (!DetectExtendingMultiply(N->getOperand(0), ScalarType, Opcode, A, B))
  4314:     return SDValue();
  4315: 
  4316:   SDLoc DL(N);
  4317:   unsigned RemainingReductionRatio;
  4318:   SDValue Partial =
  4319:       createExtendingPartialReduceMLA(Opcode, ScalarType, 1, A.getValueType(),
  4320:                                       A, B, RemainingReductionRatio, DL, DAG);
  4321:   if (!Partial)
  4322:     return SDValue();
  4323: 
  4324:   // We could have inserted a trivial MLA and rely on the folding action,
  4325:   // similar to how vector_partial_reduce_add is lowered to an MLA in
  4326:   // SelectionDAGBuilder. However, we just replace the final result since we
  4327:   // have analyzed the input completely.
  4328:   return DAG.getNode(ISD::VECREDUCE_ADD, DL, ScalarType, Partial);
  4329: }
  4330: 
  4331: // When possible, separate an MLA reduction with extended operands but
  4332: // unsupported reduction factor into an extending partial reduction that
  4333: // can be efficiently lowered, and a follow-up partial reduction.
  4334: // partial_reduce_mla(a, x, y) ->
  4335: //     partial_reduce_mla(a, partial_reduce_mla(0, x, y), 1)
  4336: SDValue
  4337: HexagonTargetLowering::splitExtendingPartialReduceMLA(SDNode *N,
  4338:                                                       SelectionDAG &DAG) const {
  4339:   if (!Subtarget.useHVXOps())
  4340:     return SDValue();
  4341: 
  4342:   SDValue Acc = N->getOperand(0);
  4343:   SDValue A = N->getOperand(1);
  4344:   SDValue B = N->getOperand(2);
  4345:   if (A.getValueType() != B.getValueType())
  4346:     return SDValue();
  4347: 
  4348:   // The types should be declared as custom, but do not split already legal
  4349:   // operation.
  4350:   EVT AccType = Acc.getValueType();
  4351:   EVT InputType = A.getValueType();
  4352:   if (getPartialReduceMLAAction(N->getOpcode(), AccType, InputType) != Custom)
  4353:     return SDValue();
  4354: 
  4355:   SDLoc DL(N);
  4356:   unsigned RemainingReductionRatio;
  4357:   SDValue Partial = createExtendingPartialReduceMLA(
  4358:       N->getOpcode(), AccType.getVectorElementType(),
  4359:       AccType.getVectorNumElements(), InputType, A, B, RemainingReductionRatio,
  4360:       DL, DAG);
  4361:   if (!Partial)
  4362:     return SDValue();
  4363:   assert(RemainingReductionRatio <= MaxExpandMLA);
  4364: 
  4365:   // Create the reduction for the remaining ratio.
  4366:   EVT IntermediateType = Partial->getOperand(0).getValueType();
  4367:   SDValue One = DAG.getConstant(1, DL, IntermediateType);
  4368:   return DAG.getNode(N->getOpcode() == ISD::PARTIAL_REDUCE_UMLA
  4369:                          ? ISD::PARTIAL_REDUCE_UMLA
  4370:                          : ISD::PARTIAL_REDUCE_SUMLA,
  4371:                      DL, AccType, Acc, Partial, One);
  4372: }
  4373: 
  4374: SDValue
  4375: HexagonTargetLowering::LowerHvxPartialReduceMLA(SDValue Op,
  4376:                                                 SelectionDAG &DAG) const {
  4377:   const SDLoc &DL(Op);
  4378:   SDValue Acc = Op.getOperand(0);
  4379:   SDValue A = Op.getOperand(1);
  4380:   SDValue B = Op.getOperand(2);
  4381: 
  4382:   // Split the input vectors into units of one HVX vector length.
  4383:   unsigned HwVectorSizeInBits = Subtarget.getVectorLength() * 8;
  4384: 
  4385:   EVT AccType = Acc.getValueType();
  4386:   EVT AccEltType = AccType.getVectorElementType();
  4387:   unsigned AccSubvectorNumElements =
  4388:       HwVectorSizeInBits / AccEltType.getSizeInBits();
  4389:   EVT AccSubvectorType =
  4390:       EVT::getVectorVT(*DAG.getContext(), AccEltType, AccSubvectorNumElements);
  4391: 
  4392:   EVT InputType = A.getValueType();
  4393:   assert(InputType.getSizeInBits() % HwVectorSizeInBits == 0);
  4394:   EVT InputEltType = InputType.getVectorElementType();
  4395:   unsigned InputSubvectorNumElements =
  4396:       HwVectorSizeInBits / InputEltType.getSizeInBits();
  4397:   EVT InputSubvectorType = EVT::getVectorVT(*DAG.getContext(), InputEltType,
  4398:                                             InputSubvectorNumElements);
  4399: 
  4400:   unsigned SubvectorNum = InputType.getFixedSizeInBits() / HwVectorSizeInBits;
```
- EN: It declares or implements routines such as MVT::getVectorVT, getNode, push_back, HexagonTargetLowering::combineConcatVectorsBeforeLegal, ... (24 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Assertions document invariants that the backend expects to hold at this stage. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonSubtarget, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 MVT::getVectorVT, getNode, push_back, HexagonTargetLowering::combineConcatVectorsBeforeLegal, ... (24 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 断言表达了该后端在当前阶段要求满足的不变量。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonSubtarget，说明了它与同级后端组件的连接关系。

### Lines 4401-4518 / 第 4401-4518 行

```cpp
  4401:   SmallVector<SDValue, MaxExpandMLA> Subvectors;
  4402: 
  4403:   for (unsigned I = 0; I != SubvectorNum; ++I) {
  4404:     SDValue SubvectorAcc = DAG.getExtractSubvector(DL, AccSubvectorType, Acc,
  4405:                                                    I * AccSubvectorNumElements);
  4406:     SDValue SubvectorA = DAG.getExtractSubvector(DL, InputSubvectorType, A,
  4407:                                                  I * InputSubvectorNumElements);
  4408:     SDValue SubvectorB = DAG.getExtractSubvector(DL, InputSubvectorType, B,
  4409:                                                  I * InputSubvectorNumElements);
  4410:     SDValue SubvectorMLA = DAG.getNode(Op.getOpcode(), DL, AccSubvectorType,
  4411:                                        SubvectorAcc, SubvectorA, SubvectorB);
  4412:     Subvectors.push_back(SubvectorMLA);
  4413:   }
  4414: 
  4415:   return DAG.getNode(ISD::CONCAT_VECTORS, DL, AccType, Subvectors);
  4416: }
  4417: 
  4418: SDValue
  4419: HexagonTargetLowering::PerformHvxDAGCombine(SDNode *N, DAGCombinerInfo &DCI)
  4420:       const {
  4421:   const SDLoc &dl(N);
  4422:   SelectionDAG &DAG = DCI.DAG;
  4423:   SDValue Op(N, 0);
  4424:   unsigned Opc = Op.getOpcode();
  4425: 
  4426:   SmallVector<SDValue, 4> Ops(N->ops());
  4427: 
  4428:   if (Opc == ISD::TRUNCATE)
  4429:     return combineTruncateBeforeLegal(Op, DCI);
  4430:   if (Opc == ISD::CONCAT_VECTORS)
  4431:     return combineConcatVectorsBeforeLegal(Op, DCI);
  4432: 
  4433:   if (DCI.isBeforeLegalizeOps())
  4434:     return SDValue();
  4435: 
  4436:   switch (Opc) {
  4437:   case HexagonISD::V2Q:
  4438:     if (Ops[0].getOpcode() == ISD::SPLAT_VECTOR) {
  4439:       if (const auto *C = dyn_cast<ConstantSDNode>(Ops[0].getOperand(0)))
  4440:         return C->isZero() ? DAG.getNode(HexagonISD::QFALSE, dl, ty(Op))
  4441:                            : DAG.getNode(HexagonISD::QTRUE, dl, ty(Op));
  4442:     }
  4443:     break;
  4444:   case HexagonISD::Q2V:
  4445:     if (Ops[0].getOpcode() == HexagonISD::QTRUE)
  4446:       return DAG.getNode(ISD::SPLAT_VECTOR, dl, ty(Op),
  4447:                          DAG.getAllOnesConstant(dl, MVT::i32));
  4448:     if (Ops[0].getOpcode() == HexagonISD::QFALSE)
  4449:       return getZero(dl, ty(Op), DAG);
  4450:     break;
  4451:   case HexagonISD::VINSERTW0:
  4452:     if (isUndef(Ops[1]))
  4453:       return Ops[0];
  4454:     break;
  4455:   case HexagonISD::VROR: {
  4456:     if (Ops[0].getOpcode() == HexagonISD::VROR) {
  4457:       SDValue Vec = Ops[0].getOperand(0);
  4458:       SDValue Rot0 = Ops[1], Rot1 = Ops[0].getOperand(1);
  4459:       SDValue Rot = DAG.getNode(ISD::ADD, dl, ty(Rot0), {Rot0, Rot1});
  4460:       return DAG.getNode(HexagonISD::VROR, dl, ty(Op), {Vec, Rot});
  4461:     }
  4462:     break;
  4463:   }
  4464:   }
  4465: 
  4466:   return SDValue();
  4467: }
  4468: 
  4469: bool
  4470: HexagonTargetLowering::shouldSplitToHvx(MVT Ty, SelectionDAG &DAG) const {
  4471:   if (Subtarget.isHVXVectorType(Ty, true))
  4472:     return false;
  4473:   auto Action = getPreferredHvxVectorAction(Ty);
  4474:   if (Action == TargetLoweringBase::TypeSplitVector)
  4475:     return Subtarget.isHVXVectorType(typeLegalize(Ty, DAG), true);
  4476:   return false;
  4477: }
  4478: 
  4479: bool
  4480: HexagonTargetLowering::shouldWidenToHvx(MVT Ty, SelectionDAG &DAG) const {
  4481:   if (Subtarget.isHVXVectorType(Ty, true))
  4482:     return false;
  4483:   auto Action = getPreferredHvxVectorAction(Ty);
  4484:   if (Action == TargetLoweringBase::TypeWidenVector)
  4485:     return Subtarget.isHVXVectorType(typeLegalize(Ty, DAG), true);
  4486:   return false;
  4487: }
  4488: 
  4489: bool
  4490: HexagonTargetLowering::isHvxOperation(SDNode *N, SelectionDAG &DAG) const {
  4491:   if (!Subtarget.useHVXOps())
  4492:     return false;
  4493:   // If the type of any result, or any operand type are HVX vector types,
  4494:   // this is an HVX operation.
  4495:   auto IsHvxTy = [this](EVT Ty) {
  4496:     return Ty.isSimple() && Subtarget.isHVXVectorType(Ty.getSimpleVT(), true);
  4497:   };
  4498:   auto IsHvxOp = [this](SDValue Op) {
  4499:     return Op.getValueType().isSimple() &&
  4500:            Subtarget.isHVXVectorType(ty(Op), true);
  4501:   };
  4502:   if (llvm::any_of(N->values(), IsHvxTy) || llvm::any_of(N->ops(), IsHvxOp))
  4503:     return true;
  4504: 
  4505:   // Check if this could be an HVX operation after type widening.
  4506:   auto IsWidenedToHvx = [this, &DAG](SDValue Op) {
  4507:     if (!Op.getValueType().isSimple())
  4508:       return false;
  4509:     MVT ValTy = ty(Op);
  4510:     return ValTy.isVector() && shouldWidenToHvx(ValTy, DAG);
  4511:   };
  4512: 
  4513:   for (int i = 0, e = N->getNumValues(); i != e; ++i) {
  4514:     if (IsWidenedToHvx(SDValue(N, i)))
  4515:       return true;
  4516:   }
  4517:   return llvm::any_of(N->ops(), IsWidenedToHvx);
  4518: }
```
- EN: It declares or implements routines such as getExtractSubvector, getNode, push_back, HexagonTargetLowering::PerformHvxDAGCombine, ... (19 total), translating Hexagon-specific policy into reusable code paths. Conditional and iterative logic in this range performs validation, selection, rewriting, or metadata traversal. Notable Hexagon symbols referenced here include HexagonTargetLowering, HexagonISD, showing how the code connects to sibling backend components.
- CN: 这里声明或实现了 getExtractSubvector, getNode, push_back, HexagonTargetLowering::PerformHvxDAGCombine, ... (19 total) 等例程，把 Hexagon 特定策略落实为可复用的代码路径。 这一段中的条件分支与循环负责校验、选择、重写或遍历元数据。 这里引用的重要 Hexagon 符号包括 HexagonTargetLowering, HexagonISD，说明了它与同级后端组件的连接关系。

## Key Concepts / 关键概念

- HVX vector ISA modeling / HVX 向量 ISA 建模
- register modeling / 寄存器建模
- instruction semantics / 指令语义
- subtarget feature gating / 子目标特性控制
- intrinsic mapping / Intrinsic 映射

## Dependencies / 依赖关系

- Direct includes / 直接包含: `HexagonISelLowering.h, HexagonRegisterInfo.h, HexagonSubtarget.h, llvm/ADT/SetVector.h, llvm/ADT/SmallVector.h, llvm/Analysis/MemoryLocation.h, llvm/CodeGen/MachineBasicBlock.h, llvm/CodeGen/MachineFunction.h, llvm/CodeGen/MachineInstr.h, llvm/CodeGen/MachineOperand.h, ... (17 total)`
- Hexagon symbols / Hexagon 符号: `HexagonISelLoweringHVX, HexagonISelLowering, HexagonRegisterInfo, HexagonSubtarget, HexagonTargetLowering, HexagonISD, HexagonMULHUS`
- Build role / 构建角色: compiled into LLVM's Hexagon backend library and linked with CodeGen/MC infrastructure. / 被编译进 LLVM 的 Hexagon 后端库，并与 CodeGen/MC 基础设施联动。
