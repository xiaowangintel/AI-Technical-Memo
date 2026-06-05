# M68kISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/M68k/M68kISelLowering.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file implements SelectionDAG lowering for the target backend for the M68k backend.
- **用途 (CN)**: 该文件用于 M68k 后端，负责实现目标后端的 SelectionDAG 降低。

## Line-by-Line Analysis / 逐行分析
### Lines 1-55 / 第 1-55 行
```cpp
   1: //===-- M68kISelLowering.cpp - M68k DAG Lowering Impl -----------*- C++ -*-===//
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
  15: #include "M68kISelLowering.h"
  16: #include "M68kCallingConv.h"
  17: #include "M68kMachineFunction.h"
  18: #include "M68kSelectionDAGInfo.h"
  19: #include "M68kSubtarget.h"
  20: #include "M68kTargetMachine.h"
  21: #include "M68kTargetObjectFile.h"
  22: #include "MCTargetDesc/M68kMCAsmInfo.h"
  23: 
  24: #include "llvm/ADT/Statistic.h"
  25: #include "llvm/CodeGen/CallingConvLower.h"
  26: #include "llvm/CodeGen/MachineFrameInfo.h"
  27: #include "llvm/CodeGen/MachineFunction.h"
  28: #include "llvm/CodeGen/MachineInstrBuilder.h"
  29: #include "llvm/CodeGen/MachineJumpTableInfo.h"
  30: #include "llvm/CodeGen/MachineRegisterInfo.h"
  31: #include "llvm/CodeGen/SelectionDAG.h"
  32: #include "llvm/CodeGen/ValueTypes.h"
  33: #include "llvm/IR/CallingConv.h"
  34: #include "llvm/IR/DerivedTypes.h"
  35: #include "llvm/IR/GlobalVariable.h"
  36: #include "llvm/Support/CommandLine.h"
  37: #include "llvm/Support/Debug.h"
  38: #include "llvm/Support/ErrorHandling.h"
  39: #include "llvm/Support/KnownBits.h"
  40: #include "llvm/Support/raw_ostream.h"
  41: 
  42: using namespace llvm;
  43: 
  44: #define DEBUG_TYPE "M68k-isel"
  45: 
  46: STATISTIC(NumTailCalls, "Number of tail calls");
  47: 
  48: M68kTargetLowering::M68kTargetLowering(const M68kTargetMachine &TM,
  49:                                        const M68kSubtarget &STI)
  50:     : TargetLowering(TM, STI), Subtarget(STI), TM(TM) {
  51: 
  52:   MVT PtrVT = MVT::i32;
  53: 
  54:   // This is based on M68k SetCC (scc) setting the destination byte to all 1s.
  55:   // See also getSetCCResultType().
```
- **EN**: The opening lines establish the compilation unit, banner, and early context for the file. It imports dependencies such as `M68kISelLowering.h`, `M68kCallingConv.h`, `M68kMachineFunction.h`, `M68kSelectionDAGInfo.h`, `M68kSubtarget.h`, `M68kTargetMachine.h` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `TargetLowering`.
- **CN**: 开头部分建立了该编译单元的横幅信息以及文件的早期上下文。 它引入了 `M68kISelLowering.h`, `M68kCallingConv.h`, `M68kMachineFunction.h`, `M68kSelectionDAGInfo.h`, `M68kSubtarget.h`, `M68kTargetMachine.h` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `TargetLowering` 等函数。

### Lines 56-110 / 第 56-110 行
```cpp
  56:   setBooleanContents(ZeroOrNegativeOneBooleanContent);
  57: 
  58:   auto *RegInfo = Subtarget.getRegisterInfo();
  59:   setStackPointerRegisterToSaveRestore(RegInfo->getStackRegister());
  60: 
  61:   // Set up the register classes.
  62:   addRegisterClass(MVT::i8, &M68k::DR8RegClass);
  63:   addRegisterClass(MVT::i16, &M68k::XR16RegClass);
  64:   addRegisterClass(MVT::i32, &M68k::XR32RegClass);
  65: 
  66:   for (auto VT : MVT::integer_valuetypes()) {
  67:     setLoadExtAction(ISD::SEXTLOAD, VT, MVT::i1, Promote);
  68:     setLoadExtAction(ISD::ZEXTLOAD, VT, MVT::i1, Promote);
  69:     setLoadExtAction(ISD::EXTLOAD, VT, MVT::i1, Promote);
  70:   }
  71: 
  72:   // We don't accept any truncstore of integer registers.
  73:   setTruncStoreAction(MVT::i64, MVT::i32, Expand);
  74:   setTruncStoreAction(MVT::i64, MVT::i16, Expand);
  75:   setTruncStoreAction(MVT::i64, MVT::i8, Expand);
  76:   setTruncStoreAction(MVT::i32, MVT::i16, Expand);
  77:   setTruncStoreAction(MVT::i32, MVT::i8, Expand);
  78:   setTruncStoreAction(MVT::i16, MVT::i8, Expand);
  79: 
  80:   setOperationAction({ISD::MUL, ISD::SDIV, ISD::UDIV}, MVT::i8, Promote);
  81:   setOperationAction({ISD::MUL, ISD::SDIV, ISD::UDIV}, MVT::i16, Legal);
  82:   if (Subtarget.atLeastM68020())
  83:     setOperationAction({ISD::MUL, ISD::SDIV, ISD::UDIV}, MVT::i32, Legal);
  84:   else
  85:     setOperationAction({ISD::MUL, ISD::SDIV, ISD::UDIV}, MVT::i32, LibCall);
  86:   setOperationAction(ISD::MUL, MVT::i64, LibCall);
  87: 
  88:   for (auto OP :
  89:        {ISD::SREM, ISD::UREM, ISD::UDIVREM, ISD::SDIVREM,
  90:         ISD::MULHS, ISD::MULHU, ISD::UMUL_LOHI, ISD::SMUL_LOHI}) {
  91:     setOperationAction(OP, MVT::i8, Promote);
  92:     setOperationAction(OP, MVT::i16, Legal);
  93:     setOperationAction(OP, MVT::i32, LibCall);
  94:   }
  95: 
  96:   for (auto OP : {ISD::UMUL_LOHI, ISD::SMUL_LOHI}) {
  97:     setOperationAction(OP, MVT::i8, Expand);
  98:     setOperationAction(OP, MVT::i16, Expand);
  99:   }
 100: 
 101:   for (auto OP : {ISD::SMULO, ISD::UMULO}) {
 102:     setOperationAction(OP, MVT::i8,  Custom);
 103:     setOperationAction(OP, MVT::i16, Custom);
 104:     setOperationAction(OP, MVT::i32, Custom);
 105:   }
 106: 
 107:   for (auto OP : {ISD::SHL_PARTS, ISD::SRA_PARTS, ISD::SRL_PARTS})
 108:     setOperationAction(OP, MVT::i32, Custom);
 109: 
 110:   // Add/Sub overflow ops with MVT::Glues are lowered to CCR dependences.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 111-165 / 第 111-165 行
```cpp
 111:   for (auto VT : {MVT::i8, MVT::i16, MVT::i32}) {
 112:     setOperationAction(ISD::ADDC, VT, Custom);
 113:     setOperationAction(ISD::ADDE, VT, Custom);
 114:     setOperationAction(ISD::SUBC, VT, Custom);
 115:     setOperationAction(ISD::SUBE, VT, Custom);
 116:   }
 117: 
 118:   // SADDO and friends are legal with this setup, i hope
 119:   for (auto VT : {MVT::i8, MVT::i16, MVT::i32}) {
 120:     setOperationAction(ISD::SADDO, VT, Custom);
 121:     setOperationAction(ISD::UADDO, VT, Custom);
 122:     setOperationAction(ISD::SSUBO, VT, Custom);
 123:     setOperationAction(ISD::USUBO, VT, Custom);
 124:   }
 125: 
 126:   setOperationAction(ISD::BR_JT, MVT::Other, Expand);
 127:   setOperationAction(ISD::BRCOND, MVT::Other, Custom);
 128: 
 129:   for (auto VT : {MVT::i8, MVT::i16, MVT::i32}) {
 130:     setOperationAction(ISD::BR_CC, VT, Expand);
 131:     setOperationAction(ISD::SELECT, VT, Custom);
 132:     setOperationAction(ISD::SELECT_CC, VT, Expand);
 133:     setOperationAction(ISD::SETCC, VT, Custom);
 134:     setOperationAction(ISD::SETCCCARRY, VT, Custom);
 135:   }
 136: 
 137:   for (auto VT : {MVT::i8, MVT::i16, MVT::i32}) {
 138:     setOperationAction(ISD::BSWAP, VT, Expand);
 139:     setOperationAction(ISD::CTTZ, VT, Expand);
 140:     setOperationAction(ISD::CTLZ, VT, Expand);
 141:     setOperationAction(ISD::CTPOP, VT, Expand);
 142:   }
 143: 
 144:   setOperationAction(ISD::ConstantPool, MVT::i32, Custom);
 145:   setOperationAction(ISD::JumpTable, MVT::i32, Custom);
 146:   setOperationAction(ISD::GlobalAddress, MVT::i32, Custom);
 147:   setOperationAction(ISD::GlobalTLSAddress, MVT::i32, Custom);
 148:   setOperationAction(ISD::ExternalSymbol, MVT::i32, Custom);
 149:   setOperationAction(ISD::BlockAddress, MVT::i32, Custom);
 150: 
 151:   setOperationAction(ISD::VASTART, MVT::Other, Custom);
 152:   setOperationAction(ISD::VAEND, MVT::Other, Expand);
 153:   setOperationAction(ISD::VAARG, MVT::Other, Expand);
 154:   setOperationAction(ISD::VACOPY, MVT::Other, Expand);
 155: 
 156:   setOperationAction(ISD::STACKSAVE, MVT::Other, Expand);
 157:   setOperationAction(ISD::STACKRESTORE, MVT::Other, Expand);
 158: 
 159:   setOperationAction(ISD::DYNAMIC_STACKALLOC, PtrVT, Custom);
 160: 
 161:   computeRegisterProperties(STI.getRegisterInfo());
 162: 
 163:   // We lower the `atomic-compare-and-swap` to `__sync_val_compare_and_swap`
 164:   // for subtarget < M68020
 165:   setMaxAtomicSizeInBitsSupported(32);
```
- **EN**: Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 166-220 / 第 166-220 行
```cpp
 166:   setOperationAction(ISD::ATOMIC_CMP_SWAP, {MVT::i8, MVT::i16, MVT::i32},
 167:                      Subtarget.atLeastM68020() ? Legal : LibCall);
 168: 
 169:   setOperationAction(ISD::ATOMIC_FENCE, MVT::Other, Custom);
 170: 
 171:   // M68k does not have native read-modify-write support, so expand all of them
 172:   // to `__sync_fetch_*` for target < M68020, otherwise expand to CmpxChg.
 173:   // See `shouldExpandAtomicRMWInIR` below.
 174:   setOperationAction(
 175:       {
 176:           ISD::ATOMIC_LOAD_ADD,
 177:           ISD::ATOMIC_LOAD_SUB,
 178:           ISD::ATOMIC_LOAD_AND,
 179:           ISD::ATOMIC_LOAD_OR,
 180:           ISD::ATOMIC_LOAD_XOR,
 181:           ISD::ATOMIC_LOAD_NAND,
 182:           ISD::ATOMIC_LOAD_MIN,
 183:           ISD::ATOMIC_LOAD_MAX,
 184:           ISD::ATOMIC_LOAD_UMIN,
 185:           ISD::ATOMIC_LOAD_UMAX,
 186:           ISD::ATOMIC_SWAP,
 187:       },
 188:       {MVT::i8, MVT::i16, MVT::i32}, LibCall);
 189: 
 190:   setMinFunctionAlignment(Align(2));
 191: }
 192: 
 193: TargetLoweringBase::AtomicExpansionKind
 194: M68kTargetLowering::shouldExpandAtomicRMWInIR(const AtomicRMWInst *RMW) const {
 195:   return Subtarget.atLeastM68020()
 196:              ? TargetLoweringBase::AtomicExpansionKind::CmpXChg
 197:              : TargetLoweringBase::AtomicExpansionKind::None;
 198: }
 199: 
 200: Register
 201: M68kTargetLowering::getExceptionPointerRegister(const Constant *) const {
 202:   return M68k::D0;
 203: }
 204: 
 205: Register
 206: M68kTargetLowering::getExceptionSelectorRegister(const Constant *) const {
 207:   return M68k::D1;
 208: }
 209: 
 210: InlineAsm::ConstraintCode
 211: M68kTargetLowering::getInlineAsmMemConstraint(StringRef ConstraintCode) const {
 212:   return StringSwitch<InlineAsm::ConstraintCode>(ConstraintCode)
 213:       .Case("Q", InlineAsm::ConstraintCode::Q)
 214:       // We borrow ConstraintCode::Um for 'U'.
 215:       .Case("U", InlineAsm::ConstraintCode::Um)
 216:       .Default(TargetLowering::getInlineAsmMemConstraint(ConstraintCode));
 217: }
 218: 
 219: EVT M68kTargetLowering::getSetCCResultType(const DataLayout &DL,
 220:                                            LLVMContext &Context, EVT VT) const {
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::shouldExpandAtomicRMWInIR`, `M68kTargetLowering::getExceptionPointerRegister`, `M68kTargetLowering::getExceptionSelectorRegister`, `M68kTargetLowering::getInlineAsmMemConstraint`, `M68kTargetLowering::getSetCCResultType`.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::shouldExpandAtomicRMWInIR`, `M68kTargetLowering::getExceptionPointerRegister`, `M68kTargetLowering::getExceptionSelectorRegister`, `M68kTargetLowering::getInlineAsmMemConstraint`, `M68kTargetLowering::getSetCCResultType` 等函数。

### Lines 221-275 / 第 221-275 行
```cpp
 221:   // M68k SETcc producess either 0x00 or 0xFF
 222:   return MVT::i8;
 223: }
 224: 
 225: MVT M68kTargetLowering::getScalarShiftAmountTy(const DataLayout &DL,
 226:                                                EVT Ty) const {
 227:   if (Ty.isSimple()) {
 228:     return Ty.getSimpleVT();
 229:   }
 230:   return MVT::getIntegerVT(DL.getPointerSizeInBits(0));
 231: }
 232: 
 233: #include "M68kGenCallingConv.inc"
 234: 
 235: enum StructReturnType { NotStructReturn, RegStructReturn, StackStructReturn };
 236: 
 237: static StructReturnType
 238: callIsStructReturn(const SmallVectorImpl<ISD::OutputArg> &Outs) {
 239:   if (Outs.empty())
 240:     return NotStructReturn;
 241: 
 242:   const ISD::ArgFlagsTy &Flags = Outs[0].Flags;
 243:   if (!Flags.isSRet())
 244:     return NotStructReturn;
 245:   if (Flags.isInReg())
 246:     return RegStructReturn;
 247:   return StackStructReturn;
 248: }
 249: 
 250: /// Determines whether a function uses struct return semantics.
 251: static StructReturnType
 252: argsAreStructReturn(const SmallVectorImpl<ISD::InputArg> &Ins) {
 253:   if (Ins.empty())
 254:     return NotStructReturn;
 255: 
 256:   const ISD::ArgFlagsTy &Flags = Ins[0].Flags;
 257:   if (!Flags.isSRet())
 258:     return NotStructReturn;
 259:   if (Flags.isInReg())
 260:     return RegStructReturn;
 261:   return StackStructReturn;
 262: }
 263: 
 264: /// Make a copy of an aggregate at address specified by "Src" to address
 265: /// "Dst" with size and alignment information specified by the specific
 266: /// parameter attribute. The copy will be passed as a byval function parameter.
 267: static SDValue CreateCopyOfByValArgument(SDValue Src, SDValue Dst,
 268:                                          SDValue Chain, ISD::ArgFlagsTy Flags,
 269:                                          SelectionDAG &DAG, const SDLoc &DL) {
 270:   SDValue SizeNode = DAG.getConstant(Flags.getByValSize(), DL, MVT::i32);
 271: 
 272:   return DAG.getMemcpy(
 273:       Chain, DL, Dst, Src, SizeNode, Flags.getNonZeroByValAlign(),
 274:       /*isVolatile=*/false, /*AlwaysInline=*/true,
 275:       /*CI=*/nullptr, std::nullopt, MachinePointerInfo(), MachinePointerInfo());
```
- **EN**: It imports dependencies such as `M68kGenCallingConv.inc` that expose the LLVM and target interfaces used in later logic. The range implements or declares functions including `M68kTargetLowering::getScalarShiftAmountTy`, `callIsStructReturn`, `argsAreStructReturn`, `CreateCopyOfByValArgument`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 它引入了 `M68kGenCallingConv.inc` 等依赖，为后续逻辑提供 LLVM 与目标后端接口。 这一段实现或声明了 `M68kTargetLowering::getScalarShiftAmountTy`, `callIsStructReturn`, `argsAreStructReturn`, `CreateCopyOfByValArgument` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 276-330 / 第 276-330 行
```cpp
 276: }
 277: 
 278: /// Return true if the calling convention is one that we can guarantee TCO for.
 279: static bool canGuaranteeTCO(CallingConv::ID CC) { return false; }
 280: 
 281: /// Return true if we might ever do TCO for calls with this calling convention.
 282: static bool mayTailCallThisCC(CallingConv::ID CC) {
 283:   switch (CC) {
 284:   // C calling conventions:
 285:   case CallingConv::C:
 286:     return true;
 287:   default:
 288:     return canGuaranteeTCO(CC);
 289:   }
 290: }
 291: 
 292: /// Return true if the function is being made into a tailcall target by
 293: /// changing its ABI.
 294: static bool shouldGuaranteeTCO(CallingConv::ID CC, bool GuaranteedTailCallOpt) {
 295:   return GuaranteedTailCallOpt && canGuaranteeTCO(CC);
 296: }
 297: 
 298: /// Return true if the given stack call argument is already available in the
 299: /// same position (relatively) of the caller's incoming argument stack.
 300: static bool MatchingStackOffset(SDValue Arg, unsigned Offset,
 301:                                 ISD::ArgFlagsTy Flags, MachineFrameInfo &MFI,
 302:                                 const MachineRegisterInfo *MRI,
 303:                                 const M68kInstrInfo *TII,
 304:                                 const CCValAssign &VA) {
 305:   unsigned Bytes = Arg.getValueType().getSizeInBits() / 8;
 306: 
 307:   for (;;) {
 308:     // Look through nodes that don't alter the bits of the incoming value.
 309:     unsigned Op = Arg.getOpcode();
 310:     if (Op == ISD::ZERO_EXTEND || Op == ISD::ANY_EXTEND || Op == ISD::BITCAST) {
 311:       Arg = Arg.getOperand(0);
 312:       continue;
 313:     }
 314:     if (Op == ISD::TRUNCATE) {
 315:       const SDValue &TruncInput = Arg.getOperand(0);
 316:       if (TruncInput.getOpcode() == ISD::AssertZext &&
 317:           cast<VTSDNode>(TruncInput.getOperand(1))->getVT() ==
 318:               Arg.getValueType()) {
 319:         Arg = TruncInput.getOperand(0);
 320:         continue;
 321:       }
 322:     }
 323:     break;
 324:   }
 325: 
 326:   int FI = INT_MAX;
 327:   if (Arg.getOpcode() == ISD::CopyFromReg) {
 328:     Register VR = cast<RegisterSDNode>(Arg.getOperand(1))->getReg();
 329:     if (!Register::isVirtualRegister(VR))
 330:       return false;
```
- **EN**: The range implements or declares functions including `canGuaranteeTCO`, `mayTailCallThisCC`, `shouldGuaranteeTCO`, `MatchingStackOffset`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `canGuaranteeTCO`, `mayTailCallThisCC`, `shouldGuaranteeTCO`, `MatchingStackOffset` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 331-385 / 第 331-385 行
```cpp
 331:     MachineInstr *Def = MRI->getVRegDef(VR);
 332:     if (!Def)
 333:       return false;
 334:     if (!Flags.isByVal()) {
 335:       if (!TII->isLoadFromStackSlot(*Def, FI))
 336:         return false;
 337:     } else {
 338:       unsigned Opcode = Def->getOpcode();
 339:       if ((Opcode == M68k::LEA32p || Opcode == M68k::LEA32f) &&
 340:           Def->getOperand(1).isFI()) {
 341:         FI = Def->getOperand(1).getIndex();
 342:         Bytes = Flags.getByValSize();
 343:       } else
 344:         return false;
 345:     }
 346:   } else if (auto *Ld = dyn_cast<LoadSDNode>(Arg)) {
 347:     if (Flags.isByVal())
 348:       // ByVal argument is passed in as a pointer but it's now being
 349:       // dereferenced. e.g.
 350:       // define @foo(%struct.X* %A) {
 351:       //   tail call @bar(%struct.X* byval %A)
 352:       // }
 353:       return false;
 354:     SDValue Ptr = Ld->getBasePtr();
 355:     FrameIndexSDNode *FINode = dyn_cast<FrameIndexSDNode>(Ptr);
 356:     if (!FINode)
 357:       return false;
 358:     FI = FINode->getIndex();
 359:   } else if (Arg.getOpcode() == ISD::FrameIndex && Flags.isByVal()) {
 360:     FrameIndexSDNode *FINode = cast<FrameIndexSDNode>(Arg);
 361:     FI = FINode->getIndex();
 362:     Bytes = Flags.getByValSize();
 363:   } else
 364:     return false;
 365: 
 366:   assert(FI != INT_MAX);
 367:   if (!MFI.isFixedObjectIndex(FI))
 368:     return false;
 369: 
 370:   if (Offset != MFI.getObjectOffset(FI))
 371:     return false;
 372: 
 373:   if (VA.getLocVT().getSizeInBits() > Arg.getValueType().getSizeInBits()) {
 374:     // If the argument location is wider than the argument type, check that any
 375:     // extension flags match.
 376:     if (Flags.isZExt() != MFI.isObjectZExt(FI) ||
 377:         Flags.isSExt() != MFI.isObjectSExt(FI)) {
 378:       return false;
 379:     }
 380:   }
 381: 
 382:   return Bytes == MFI.getObjectSize(FI);
 383: }
 384: 
 385: SDValue
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 386-440 / 第 386-440 行
```cpp
 386: M68kTargetLowering::getReturnAddressFrameIndex(SelectionDAG &DAG) const {
 387:   MachineFunction &MF = DAG.getMachineFunction();
 388:   M68kMachineFunctionInfo *FuncInfo = MF.getInfo<M68kMachineFunctionInfo>();
 389:   int ReturnAddrIndex = FuncInfo->getRAIndex();
 390: 
 391:   if (ReturnAddrIndex == 0) {
 392:     // Set up a frame object for the return address.
 393:     unsigned SlotSize = Subtarget.getSlotSize();
 394:     ReturnAddrIndex = MF.getFrameInfo().CreateFixedObject(
 395:         SlotSize, -(int64_t)SlotSize, false);
 396:     FuncInfo->setRAIndex(ReturnAddrIndex);
 397:   }
 398: 
 399:   return DAG.getFrameIndex(ReturnAddrIndex, getPointerTy(DAG.getDataLayout()));
 400: }
 401: 
 402: SDValue M68kTargetLowering::EmitTailCallLoadRetAddr(SelectionDAG &DAG,
 403:                                                     SDValue &OutRetAddr,
 404:                                                     SDValue Chain,
 405:                                                     bool IsTailCall, int FPDiff,
 406:                                                     const SDLoc &DL) const {
 407:   EVT VT = getPointerTy(DAG.getDataLayout());
 408:   OutRetAddr = getReturnAddressFrameIndex(DAG);
 409: 
 410:   // Load the "old" Return address.
 411:   OutRetAddr = DAG.getLoad(VT, DL, Chain, OutRetAddr, MachinePointerInfo());
 412:   return SDValue(OutRetAddr.getNode(), 1);
 413: }
 414: 
 415: SDValue M68kTargetLowering::EmitTailCallStoreRetAddr(
 416:     SelectionDAG &DAG, MachineFunction &MF, SDValue Chain, SDValue RetFI,
 417:     EVT PtrVT, unsigned SlotSize, int FPDiff, const SDLoc &DL) const {
 418:   if (!FPDiff)
 419:     return Chain;
 420: 
 421:   // Calculate the new stack slot for the return address.
 422:   int NewFO = MF.getFrameInfo().CreateFixedObject(
 423:       SlotSize, (int64_t)FPDiff - SlotSize, false);
 424: 
 425:   SDValue NewFI = DAG.getFrameIndex(NewFO, PtrVT);
 426:   // Store the return address to the appropriate stack slot.
 427:   Chain = DAG.getStore(
 428:       Chain, DL, RetFI, NewFI,
 429:       MachinePointerInfo::getFixedStack(DAG.getMachineFunction(), NewFO));
 430:   return Chain;
 431: }
 432: 
 433: SDValue
 434: M68kTargetLowering::LowerMemArgument(SDValue Chain, CallingConv::ID CallConv,
 435:                                      const SmallVectorImpl<ISD::InputArg> &Ins,
 436:                                      const SDLoc &DL, SelectionDAG &DAG,
 437:                                      const CCValAssign &VA,
 438:                                      MachineFrameInfo &MFI,
 439:                                      unsigned ArgIdx) const {
 440:   // Create the nodes corresponding to a load from this parameter slot.
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::EmitTailCallLoadRetAddr`, `M68kTargetLowering::EmitTailCallStoreRetAddr`, `M68kTargetLowering::LowerMemArgument`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::EmitTailCallLoadRetAddr`, `M68kTargetLowering::EmitTailCallStoreRetAddr`, `M68kTargetLowering::LowerMemArgument` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 441-495 / 第 441-495 行
```cpp
 441:   ISD::ArgFlagsTy Flags = Ins[ArgIdx].Flags;
 442:   EVT ValVT;
 443: 
 444:   // If value is passed by pointer we have address passed instead of the value
 445:   // itself.
 446:   if (VA.getLocInfo() == CCValAssign::Indirect)
 447:     ValVT = VA.getLocVT();
 448:   else
 449:     ValVT = VA.getValVT();
 450: 
 451:   // Because we are dealing with BE architecture we need to offset loading of
 452:   // partial types
 453:   int Offset = VA.getLocMemOffset();
 454:   if (VA.getValVT() == MVT::i8) {
 455:     Offset += 3;
 456:   } else if (VA.getValVT() == MVT::i16) {
 457:     Offset += 2;
 458:   }
 459: 
 460:   // TODO Interrupt handlers
 461:   // Calculate SP offset of interrupt parameter, re-arrange the slot normally
 462:   // taken by a return address.
 463: 
 464:   // FIXME For now, all byval parameter objects are marked mutable. This can
 465:   // be changed with more analysis. In case of tail call optimization mark all
 466:   // arguments mutable. Since they could be overwritten by lowering of arguments
 467:   // in case of a tail call.
 468:   bool AlwaysUseMutable = shouldGuaranteeTCO(
 469:       CallConv, DAG.getTarget().Options.GuaranteedTailCallOpt);
 470:   bool IsImmutable = !AlwaysUseMutable && !Flags.isByVal();
 471: 
 472:   if (Flags.isByVal()) {
 473:     unsigned Bytes = Flags.getByValSize();
 474:     if (Bytes == 0)
 475:       Bytes = 1; // Don't create zero-sized stack objects.
 476:     int FI = MFI.CreateFixedObject(Bytes, Offset, IsImmutable);
 477:     // TODO Interrupt handlers
 478:     // Adjust SP offset of interrupt parameter.
 479:     return DAG.getFrameIndex(FI, getPointerTy(DAG.getDataLayout()));
 480:   } else {
 481:     int FI =
 482:         MFI.CreateFixedObject(ValVT.getSizeInBits() / 8, Offset, IsImmutable);
 483: 
 484:     // Set SExt or ZExt flag.
 485:     if (VA.getLocInfo() == CCValAssign::ZExt) {
 486:       MFI.setObjectZExt(FI, true);
 487:     } else if (VA.getLocInfo() == CCValAssign::SExt) {
 488:       MFI.setObjectSExt(FI, true);
 489:     }
 490: 
 491:     // TODO Interrupt handlers
 492:     // Adjust SP offset of interrupt parameter.
 493: 
 494:     SDValue FIN = DAG.getFrameIndex(FI, getPointerTy(DAG.getDataLayout()));
 495:     SDValue Val = DAG.getLoad(
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 496-550 / 第 496-550 行
```cpp
 496:         ValVT, DL, Chain, FIN,
 497:         MachinePointerInfo::getFixedStack(DAG.getMachineFunction(), FI));
 498:     return VA.isExtInLoc() ? DAG.getNode(ISD::TRUNCATE, DL, VA.getValVT(), Val)
 499:                            : Val;
 500:   }
 501: }
 502: 
 503: SDValue M68kTargetLowering::LowerMemOpCallTo(SDValue Chain, SDValue StackPtr,
 504:                                              SDValue Arg, const SDLoc &DL,
 505:                                              SelectionDAG &DAG,
 506:                                              const CCValAssign &VA,
 507:                                              ISD::ArgFlagsTy Flags) const {
 508:   unsigned LocMemOffset = VA.getLocMemOffset();
 509:   SDValue PtrOff = DAG.getIntPtrConstant(LocMemOffset, DL);
 510:   PtrOff = DAG.getNode(ISD::ADD, DL, getPointerTy(DAG.getDataLayout()),
 511:                        StackPtr, PtrOff);
 512:   if (Flags.isByVal())
 513:     return CreateCopyOfByValArgument(Arg, PtrOff, Chain, Flags, DAG, DL);
 514: 
 515:   return DAG.getStore(
 516:       Chain, DL, Arg, PtrOff,
 517:       MachinePointerInfo::getStack(DAG.getMachineFunction(), LocMemOffset));
 518: }
 519: 
 520: //===----------------------------------------------------------------------===//
 521: //                                   Call
 522: //===----------------------------------------------------------------------===//
 523: 
 524: SDValue M68kTargetLowering::LowerCall(TargetLowering::CallLoweringInfo &CLI,
 525:                                       SmallVectorImpl<SDValue> &InVals) const {
 526:   SelectionDAG &DAG = CLI.DAG;
 527:   SDLoc &DL = CLI.DL;
 528:   SmallVectorImpl<ISD::OutputArg> &Outs = CLI.Outs;
 529:   SmallVectorImpl<SDValue> &OutVals = CLI.OutVals;
 530:   SmallVectorImpl<ISD::InputArg> &Ins = CLI.Ins;
 531:   SDValue Chain = CLI.Chain;
 532:   SDValue Callee = CLI.Callee;
 533:   CallingConv::ID CallConv = CLI.CallConv;
 534:   bool &IsTailCall = CLI.IsTailCall;
 535:   bool IsVarArg = CLI.IsVarArg;
 536: 
 537:   MachineFunction &MF = DAG.getMachineFunction();
 538:   StructReturnType SR = callIsStructReturn(Outs);
 539:   bool IsSibcall = false;
 540:   M68kMachineFunctionInfo *MFI = MF.getInfo<M68kMachineFunctionInfo>();
 541:   // const M68kRegisterInfo *TRI = Subtarget.getRegisterInfo();
 542: 
 543:   if (CallConv == CallingConv::M68k_INTR)
 544:     report_fatal_error("M68k interrupts may not be called directly");
 545: 
 546:   auto Attr = MF.getFunction().getFnAttribute("disable-tail-calls");
 547:   if (Attr.getValueAsBool())
 548:     IsTailCall = false;
 549: 
 550:   // FIXME Add tailcalls support
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::LowerMemOpCallTo`, `M68kTargetLowering::LowerCall`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::LowerMemOpCallTo`, `M68kTargetLowering::LowerCall` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 551-605 / 第 551-605 行
```cpp
 551: 
 552:   bool IsMustTail = CLI.CB && CLI.CB->isMustTailCall();
 553:   if (IsMustTail) {
 554:     // Force this to be a tail call.  The verifier rules are enough to ensure
 555:     // that we can lower this successfully without moving the return address
 556:     // around.
 557:     IsTailCall = true;
 558:   } else if (IsTailCall) {
 559:     // Check if it's really possible to do a tail call.
 560:     IsTailCall = IsEligibleForTailCallOptimization(
 561:         Callee, CallConv, IsVarArg, SR != NotStructReturn,
 562:         MF.getFunction().hasStructRetAttr(), CLI.RetTy, Outs, OutVals, Ins,
 563:         DAG);
 564: 
 565:     // Sibcalls are automatically detected tailcalls which do not require
 566:     // ABI changes.
 567:     if (!MF.getTarget().Options.GuaranteedTailCallOpt && IsTailCall)
 568:       IsSibcall = true;
 569: 
 570:     if (IsTailCall)
 571:       ++NumTailCalls;
 572:   }
 573: 
 574:   assert(!(IsVarArg && canGuaranteeTCO(CallConv)) &&
 575:          "Var args not supported with calling convention fastcc");
 576: 
 577:   // Analyze operands of the call, assigning locations to each operand.
 578:   SmallVector<CCValAssign, 16> ArgLocs;
 579:   SmallVector<Type *, 4> ArgTypes;
 580:   for (const auto &Arg : CLI.getArgs())
 581:     ArgTypes.emplace_back(Arg.Ty);
 582:   M68kCCState CCInfo(ArgTypes, CallConv, IsVarArg, MF, ArgLocs,
 583:                      *DAG.getContext());
 584:   CCInfo.AnalyzeCallOperands(Outs, CC_M68k);
 585: 
 586:   // Get a count of how many bytes are to be pushed on the stack.
 587:   unsigned NumBytes = CCInfo.getAlignedCallFrameSize();
 588:   if (IsSibcall) {
 589:     // This is a sibcall. The memory operands are available in caller's
 590:     // own caller's stack.
 591:     NumBytes = 0;
 592:   } else if (MF.getTarget().Options.GuaranteedTailCallOpt &&
 593:              canGuaranteeTCO(CallConv)) {
 594:     NumBytes = GetAlignedArgumentStackSize(NumBytes, DAG);
 595:   }
 596: 
 597:   int FPDiff = 0;
 598:   if (IsTailCall && !IsSibcall && !IsMustTail) {
 599:     // Lower arguments at fp - stackoffset + fpdiff.
 600:     unsigned NumBytesCallerPushed = MFI->getBytesToPopOnReturn();
 601: 
 602:     FPDiff = NumBytesCallerPushed - NumBytes;
 603: 
 604:     // Set the delta of movement of the returnaddr stackslot.
 605:     // But only set if delta is greater than previous delta.
```
- **EN**: The range implements or declares functions including `canGuaranteeTCO`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `canGuaranteeTCO` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 606-660 / 第 606-660 行
```cpp
 606:     if (FPDiff < MFI->getTCReturnAddrDelta())
 607:       MFI->setTCReturnAddrDelta(FPDiff);
 608:   }
 609: 
 610:   unsigned NumBytesToPush = NumBytes;
 611:   unsigned NumBytesToPop = NumBytes;
 612: 
 613:   // If we have an inalloca argument, all stack space has already been allocated
 614:   // for us and be right at the top of the stack.  We don't support multiple
 615:   // arguments passed in memory when using inalloca.
 616:   if (!Outs.empty() && Outs.back().Flags.isInAlloca()) {
 617:     NumBytesToPush = 0;
 618:     if (!ArgLocs.back().isMemLoc())
 619:       report_fatal_error("cannot use inalloca attribute on a register "
 620:                          "parameter");
 621:     if (ArgLocs.back().getLocMemOffset() != 0)
 622:       report_fatal_error("any parameter with the inalloca attribute must be "
 623:                          "the only memory argument");
 624:   }
 625: 
 626:   if (!IsSibcall)
 627:     Chain = DAG.getCALLSEQ_START(Chain, NumBytesToPush,
 628:                                  NumBytes - NumBytesToPush, DL);
 629: 
 630:   SDValue RetFI;
 631:   // Load return address for tail calls.
 632:   if (IsTailCall && FPDiff)
 633:     Chain = EmitTailCallLoadRetAddr(DAG, RetFI, Chain, IsTailCall, FPDiff, DL);
 634: 
 635:   SmallVector<std::pair<unsigned, SDValue>, 8> RegsToPass;
 636:   SmallVector<SDValue, 8> MemOpChains;
 637:   SDValue StackPtr;
 638: 
 639:   // Walk the register/memloc assignments, inserting copies/loads.  In the case
 640:   // of tail call optimization arguments are handle later.
 641:   const M68kRegisterInfo *RegInfo = Subtarget.getRegisterInfo();
 642:   for (unsigned i = 0, e = ArgLocs.size(); i != e; ++i) {
 643:     ISD::ArgFlagsTy Flags = Outs[i].Flags;
 644: 
 645:     // Skip inalloca arguments, they have already been written.
 646:     if (Flags.isInAlloca())
 647:       continue;
 648: 
 649:     CCValAssign &VA = ArgLocs[i];
 650:     EVT RegVT = VA.getLocVT();
 651:     SDValue Arg = OutVals[i];
 652:     bool IsByVal = Flags.isByVal();
 653: 
 654:     // Promote the value if needed.
 655:     switch (VA.getLocInfo()) {
 656:     default:
 657:       llvm_unreachable("Unknown loc info!");
 658:     case CCValAssign::Full:
 659:       break;
 660:     case CCValAssign::SExt:
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 661-715 / 第 661-715 行
```cpp
 661:       Arg = DAG.getNode(ISD::SIGN_EXTEND, DL, RegVT, Arg);
 662:       break;
 663:     case CCValAssign::ZExt:
 664:       Arg = DAG.getNode(ISD::ZERO_EXTEND, DL, RegVT, Arg);
 665:       break;
 666:     case CCValAssign::AExt:
 667:       Arg = DAG.getNode(ISD::ANY_EXTEND, DL, RegVT, Arg);
 668:       break;
 669:     case CCValAssign::BCvt:
 670:       Arg = DAG.getBitcast(RegVT, Arg);
 671:       break;
 672:     case CCValAssign::Indirect: {
 673:       // Store the argument.
 674:       SDValue SpillSlot = DAG.CreateStackTemporary(VA.getValVT());
 675:       int FI = cast<FrameIndexSDNode>(SpillSlot)->getIndex();
 676:       Chain = DAG.getStore(
 677:           Chain, DL, Arg, SpillSlot,
 678:           MachinePointerInfo::getFixedStack(DAG.getMachineFunction(), FI));
 679:       Arg = SpillSlot;
 680:       break;
 681:     }
 682:     }
 683: 
 684:     if (VA.isRegLoc()) {
 685:       RegsToPass.push_back(std::make_pair(VA.getLocReg(), Arg));
 686:     } else if (!IsSibcall && (!IsTailCall || IsByVal)) {
 687:       assert(VA.isMemLoc());
 688:       if (!StackPtr.getNode()) {
 689:         StackPtr = DAG.getCopyFromReg(Chain, DL, RegInfo->getStackRegister(),
 690:                                       getPointerTy(DAG.getDataLayout()));
 691:       }
 692:       MemOpChains.push_back(
 693:           LowerMemOpCallTo(Chain, StackPtr, Arg, DL, DAG, VA, Flags));
 694:     }
 695:   }
 696: 
 697:   if (!MemOpChains.empty())
 698:     Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, MemOpChains);
 699: 
 700:   // FIXME Make sure PIC style GOT works as expected
 701:   // The only time GOT is really needed is for Medium-PIC static data
 702:   // otherwise we are happy with pc-rel or static references
 703: 
 704:   if (IsVarArg && IsMustTail) {
 705:     const auto &Forwards = MFI->getForwardedMustTailRegParms();
 706:     for (const auto &F : Forwards) {
 707:       SDValue Val = DAG.getCopyFromReg(Chain, DL, F.VReg, F.VT);
 708:       RegsToPass.push_back(std::make_pair(unsigned(F.PReg), Val));
 709:     }
 710:   }
 711: 
 712:   // For tail calls lower the arguments to the 'real' stack slots.  Sibcalls
 713:   // don't need this because the eligibility check rejects calls that require
 714:   // shuffling arguments passed in memory.
 715:   if (!IsSibcall && IsTailCall) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 716-770 / 第 716-770 行
```cpp
 716:     // Force all the incoming stack arguments to be loaded from the stack
 717:     // before any new outgoing arguments are stored to the stack, because the
 718:     // outgoing stack slots may alias the incoming argument stack slots, and
 719:     // the alias isn't otherwise explicit. This is slightly more conservative
 720:     // than necessary, because it means that each store effectively depends
 721:     // on every argument instead of just those arguments it would clobber.
 722:     SDValue ArgChain = DAG.getStackArgumentTokenFactor(Chain);
 723: 
 724:     SmallVector<SDValue, 8> MemOpChains2;
 725:     SDValue FIN;
 726:     int FI = 0;
 727:     for (unsigned i = 0, e = ArgLocs.size(); i != e; ++i) {
 728:       CCValAssign &VA = ArgLocs[i];
 729:       if (VA.isRegLoc())
 730:         continue;
 731:       assert(VA.isMemLoc());
 732:       SDValue Arg = OutVals[i];
 733:       ISD::ArgFlagsTy Flags = Outs[i].Flags;
 734:       // Skip inalloca arguments.  They don't require any work.
 735:       if (Flags.isInAlloca())
 736:         continue;
 737:       // Create frame index.
 738:       int32_t Offset = VA.getLocMemOffset() + FPDiff;
 739:       uint32_t OpSize = (VA.getLocVT().getSizeInBits() + 7) / 8;
 740:       FI = MF.getFrameInfo().CreateFixedObject(OpSize, Offset, true);
 741:       FIN = DAG.getFrameIndex(FI, getPointerTy(DAG.getDataLayout()));
 742: 
 743:       if (Flags.isByVal()) {
 744:         // Copy relative to framepointer.
 745:         SDValue Source = DAG.getIntPtrConstant(VA.getLocMemOffset(), DL);
 746:         if (!StackPtr.getNode()) {
 747:           StackPtr = DAG.getCopyFromReg(Chain, DL, RegInfo->getStackRegister(),
 748:                                         getPointerTy(DAG.getDataLayout()));
 749:         }
 750:         Source = DAG.getNode(ISD::ADD, DL, getPointerTy(DAG.getDataLayout()),
 751:                              StackPtr, Source);
 752: 
 753:         MemOpChains2.push_back(
 754:             CreateCopyOfByValArgument(Source, FIN, ArgChain, Flags, DAG, DL));
 755:       } else {
 756:         // Store relative to framepointer.
 757:         MemOpChains2.push_back(DAG.getStore(
 758:             ArgChain, DL, Arg, FIN,
 759:             MachinePointerInfo::getFixedStack(DAG.getMachineFunction(), FI)));
 760:       }
 761:     }
 762: 
 763:     if (!MemOpChains2.empty())
 764:       Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, MemOpChains2);
 765: 
 766:     // Store the return address to the appropriate stack slot.
 767:     Chain = EmitTailCallStoreRetAddr(DAG, MF, Chain, RetFI,
 768:                                      getPointerTy(DAG.getDataLayout()),
 769:                                      Subtarget.getSlotSize(), FPDiff, DL);
 770:   }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 771-825 / 第 771-825 行
```cpp
 771: 
 772:   // Build a sequence of copy-to-reg nodes chained together with token chain
 773:   // and flag operands which copy the outgoing args into registers.
 774:   SDValue InGlue;
 775:   for (unsigned i = 0, e = RegsToPass.size(); i != e; ++i) {
 776:     Chain = DAG.getCopyToReg(Chain, DL, RegsToPass[i].first,
 777:                              RegsToPass[i].second, InGlue);
 778:     InGlue = Chain.getValue(1);
 779:   }
 780: 
 781:   if (Callee->getOpcode() == ISD::GlobalAddress) {
 782:     // If the callee is a GlobalAddress node (quite common, every direct call
 783:     // is) turn it into a TargetGlobalAddress node so that legalize doesn't hack
 784:     // it.
 785:     GlobalAddressSDNode *G = cast<GlobalAddressSDNode>(Callee);
 786: 
 787:     // We should use extra load for direct calls to dllimported functions in
 788:     // non-JIT mode.
 789:     const GlobalValue *GV = G->getGlobal();
 790:     if (!GV->hasDLLImportStorageClass()) {
 791:       unsigned char OpFlags = Subtarget.classifyGlobalFunctionReference(GV);
 792: 
 793:       Callee = DAG.getTargetGlobalAddress(
 794:           GV, DL, getPointerTy(DAG.getDataLayout()), G->getOffset(), OpFlags);
 795: 
 796:       if (OpFlags == M68kII::MO_GOTPCREL) {
 797: 
 798:         // Add a wrapper.
 799:         Callee = DAG.getNode(M68kISD::WrapperPC, DL,
 800:                              getPointerTy(DAG.getDataLayout()), Callee);
 801: 
 802:         // Add extra indirection
 803:         Callee = DAG.getLoad(
 804:             getPointerTy(DAG.getDataLayout()), DL, DAG.getEntryNode(), Callee,
 805:             MachinePointerInfo::getGOT(DAG.getMachineFunction()));
 806:       }
 807:     }
 808:   } else if (ExternalSymbolSDNode *S = dyn_cast<ExternalSymbolSDNode>(Callee)) {
 809:     const Module *Mod = DAG.getMachineFunction().getFunction().getParent();
 810:     unsigned char OpFlags =
 811:         Subtarget.classifyGlobalFunctionReference(nullptr, *Mod);
 812: 
 813:     Callee = DAG.getTargetExternalSymbol(
 814:         S->getSymbol(), getPointerTy(DAG.getDataLayout()), OpFlags);
 815:   }
 816: 
 817:   SmallVector<SDValue, 8> Ops;
 818: 
 819:   if (!IsSibcall && IsTailCall) {
 820:     Chain = DAG.getCALLSEQ_END(Chain, NumBytesToPop, 0, InGlue, DL);
 821:     InGlue = Chain.getValue(1);
 822:   }
 823: 
 824:   Ops.push_back(Chain);
 825:   Ops.push_back(Callee);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 826-880 / 第 826-880 行
```cpp
 826: 
 827:   if (IsTailCall)
 828:     Ops.push_back(DAG.getConstant(FPDiff, DL, MVT::i32));
 829: 
 830:   // Add argument registers to the end of the list so that they are known live
 831:   // into the call.
 832:   for (unsigned i = 0, e = RegsToPass.size(); i != e; ++i)
 833:     Ops.push_back(DAG.getRegister(RegsToPass[i].first,
 834:                                   RegsToPass[i].second.getValueType()));
 835: 
 836:   // Add a register mask operand representing the call-preserved registers.
 837:   const uint32_t *Mask = RegInfo->getCallPreservedMask(MF, CallConv);
 838:   assert(Mask && "Missing call preserved mask for calling convention");
 839: 
 840:   Ops.push_back(DAG.getRegisterMask(Mask));
 841: 
 842:   if (InGlue.getNode())
 843:     Ops.push_back(InGlue);
 844: 
 845:   if (IsTailCall) {
 846:     MF.getFrameInfo().setHasTailCall();
 847:     return DAG.getNode(M68kISD::TC_RETURN, DL, MVT::Other, Ops);
 848:   }
 849: 
 850:   // Returns a chain & a flag for retval copy to use.
 851:   Chain = DAG.getNode(M68kISD::CALL, DL, {MVT::Other, MVT::Glue}, Ops);
 852:   InGlue = Chain.getValue(1);
 853: 
 854:   // Create the CALLSEQ_END node.
 855:   unsigned NumBytesForCalleeToPop;
 856:   if (M68k::isCalleePop(CallConv, IsVarArg,
 857:                         DAG.getTarget().Options.GuaranteedTailCallOpt)) {
 858:     NumBytesForCalleeToPop = NumBytes; // Callee pops everything
 859:   } else if (!canGuaranteeTCO(CallConv) && SR == StackStructReturn) {
 860:     // If this is a call to a struct-return function, the callee
 861:     // pops the hidden struct pointer, so we have to push it back.
 862:     NumBytesForCalleeToPop = 4;
 863:   } else {
 864:     NumBytesForCalleeToPop = 0; // Callee pops nothing.
 865:   }
 866: 
 867:   if (CLI.DoesNotReturn && !getTargetMachine().Options.TrapUnreachable) {
 868:     // No need to reset the stack after the call if the call doesn't return. To
 869:     // make the MI verify, we'll pretend the callee does it for us.
 870:     NumBytesForCalleeToPop = NumBytes;
 871:   }
 872: 
 873:   // Returns a flag for retval copy to use.
 874:   if (!IsSibcall) {
 875:     Chain = DAG.getCALLSEQ_END(Chain, NumBytesToPop, NumBytesForCalleeToPop,
 876:                                InGlue, DL);
 877:     InGlue = Chain.getValue(1);
 878:   }
 879: 
 880:   // Handle result values, copying them out of physregs into vregs that we
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 881-935 / 第 881-935 行
```cpp
 881:   // return.
 882:   return LowerCallResult(Chain, InGlue, CallConv, IsVarArg, Ins, DL, DAG,
 883:                          InVals);
 884: }
 885: 
 886: SDValue M68kTargetLowering::LowerCallResult(
 887:     SDValue Chain, SDValue InGlue, CallingConv::ID CallConv, bool IsVarArg,
 888:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &DL,
 889:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
 890: 
 891:   // Assign locations to each value returned by this call.
 892:   SmallVector<CCValAssign, 16> RVLocs;
 893:   CCState CCInfo(CallConv, IsVarArg, DAG.getMachineFunction(), RVLocs,
 894:                  *DAG.getContext());
 895:   CCInfo.AnalyzeCallResult(Ins, RetCC_M68k);
 896: 
 897:   // Copy all of the result registers out of their specified physreg.
 898:   for (unsigned i = 0, e = RVLocs.size(); i != e; ++i) {
 899:     CCValAssign &VA = RVLocs[i];
 900:     EVT CopyVT = VA.getLocVT();
 901: 
 902:     /// ??? is this correct?
 903:     Chain = DAG.getCopyFromReg(Chain, DL, VA.getLocReg(), CopyVT, InGlue)
 904:                 .getValue(1);
 905:     SDValue Val = Chain.getValue(0);
 906: 
 907:     if (VA.isExtInLoc() && VA.getValVT().getScalarType() == MVT::i1)
 908:       Val = DAG.getNode(ISD::TRUNCATE, DL, VA.getValVT(), Val);
 909: 
 910:     InGlue = Chain.getValue(2);
 911:     InVals.push_back(Val);
 912:   }
 913: 
 914:   return Chain;
 915: }
 916: 
 917: //===----------------------------------------------------------------------===//
 918: //            Formal Arguments Calling Convention Implementation
 919: //===----------------------------------------------------------------------===//
 920: 
 921: SDValue M68kTargetLowering::LowerFormalArguments(
 922:     SDValue Chain, CallingConv::ID CCID, bool IsVarArg,
 923:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &DL,
 924:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
 925:   MachineFunction &MF = DAG.getMachineFunction();
 926:   M68kMachineFunctionInfo *MMFI = MF.getInfo<M68kMachineFunctionInfo>();
 927:   // const TargetFrameLowering &TFL = *Subtarget.getFrameLowering();
 928: 
 929:   MachineFrameInfo &MFI = MF.getFrameInfo();
 930: 
 931:   // Assign locations to all of the incoming arguments.
 932:   SmallVector<CCValAssign, 16> ArgLocs;
 933:   SmallVector<Type *, 4> ArgTypes;
 934:   for (const Argument &Arg : MF.getFunction().args())
 935:     ArgTypes.emplace_back(Arg.getType());
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::LowerCallResult`, `M68kTargetLowering::LowerFormalArguments`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::LowerCallResult`, `M68kTargetLowering::LowerFormalArguments` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 936-990 / 第 936-990 行
```cpp
 936:   M68kCCState CCInfo(ArgTypes, CCID, IsVarArg, MF, ArgLocs, *DAG.getContext());
 937: 
 938:   CCInfo.AnalyzeFormalArguments(Ins, CC_M68k);
 939: 
 940:   unsigned LastVal = ~0U;
 941:   SDValue ArgValue;
 942:   for (unsigned i = 0, e = ArgLocs.size(); i != e; ++i) {
 943:     CCValAssign &VA = ArgLocs[i];
 944:     assert(VA.getValNo() != LastVal && "Same value in different locations");
 945:     (void)LastVal;
 946: 
 947:     LastVal = VA.getValNo();
 948: 
 949:     if (VA.isRegLoc()) {
 950:       EVT RegVT = VA.getLocVT();
 951:       const TargetRegisterClass *RC;
 952:       if (RegVT == MVT::i32)
 953:         RC = &M68k::XR32RegClass;
 954:       else
 955:         llvm_unreachable("Unknown argument type!");
 956: 
 957:       Register Reg = MF.addLiveIn(VA.getLocReg(), RC);
 958:       ArgValue = DAG.getCopyFromReg(Chain, DL, Reg, RegVT);
 959: 
 960:       // If this is an 8 or 16-bit value, it is really passed promoted to 32
 961:       // bits.  Insert an assert[sz]ext to capture this, then truncate to the
 962:       // right size.
 963:       if (VA.getLocInfo() == CCValAssign::SExt) {
 964:         ArgValue = DAG.getNode(ISD::AssertSext, DL, RegVT, ArgValue,
 965:                                DAG.getValueType(VA.getValVT()));
 966:       } else if (VA.getLocInfo() == CCValAssign::ZExt) {
 967:         ArgValue = DAG.getNode(ISD::AssertZext, DL, RegVT, ArgValue,
 968:                                DAG.getValueType(VA.getValVT()));
 969:       } else if (VA.getLocInfo() == CCValAssign::BCvt) {
 970:         ArgValue = DAG.getBitcast(VA.getValVT(), ArgValue);
 971:       }
 972: 
 973:       if (VA.isExtInLoc()) {
 974:         ArgValue = DAG.getNode(ISD::TRUNCATE, DL, VA.getValVT(), ArgValue);
 975:       }
 976:     } else {
 977:       assert(VA.isMemLoc());
 978:       ArgValue = LowerMemArgument(Chain, CCID, Ins, DL, DAG, VA, MFI, i);
 979:     }
 980: 
 981:     // If value is passed via pointer - do a load.
 982:     // TODO Make sure this handling on indirect arguments is correct
 983:     if (VA.getLocInfo() == CCValAssign::Indirect)
 984:       ArgValue =
 985:           DAG.getLoad(VA.getValVT(), DL, Chain, ArgValue, MachinePointerInfo());
 986: 
 987:     InVals.push_back(ArgValue);
 988:   }
 989: 
 990:   for (unsigned i = 0, e = ArgLocs.size(); i != e; ++i) {
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 991-1045 / 第 991-1045 行
```cpp
 991:     // Swift calling convention does not require we copy the sret argument
 992:     // into %D0 for the return. We don't set SRetReturnReg for Swift.
 993:     if (CCID == CallingConv::Swift)
 994:       continue;
 995: 
 996:     // ABI require that for returning structs by value we copy the sret argument
 997:     // into %D0 for the return. Save the argument into a virtual register so
 998:     // that we can access it from the return points.
 999:     if (Ins[i].Flags.isSRet()) {
1000:       unsigned Reg = MMFI->getSRetReturnReg();
1001:       if (!Reg) {
1002:         MVT PtrTy = getPointerTy(DAG.getDataLayout());
1003:         Reg = MF.getRegInfo().createVirtualRegister(getRegClassFor(PtrTy));
1004:         MMFI->setSRetReturnReg(Reg);
1005:       }
1006:       SDValue Copy = DAG.getCopyToReg(DAG.getEntryNode(), DL, Reg, InVals[i]);
1007:       Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, Copy, Chain);
1008:       break;
1009:     }
1010:   }
1011: 
1012:   unsigned StackSize = CCInfo.getStackSize();
1013:   // Align stack specially for tail calls.
1014:   if (shouldGuaranteeTCO(CCID, MF.getTarget().Options.GuaranteedTailCallOpt))
1015:     StackSize = GetAlignedArgumentStackSize(StackSize, DAG);
1016: 
1017:   // If the function takes variable number of arguments, make a frame index for
1018:   // the start of the first vararg value... for expansion of llvm.va_start. We
1019:   // can skip this if there are no va_start calls.
1020:   if (MFI.hasVAStart()) {
1021:     MMFI->setVarArgsFrameIndex(MFI.CreateFixedObject(1, StackSize, true));
1022:   }
1023: 
1024:   if (IsVarArg && MFI.hasMustTailInVarArgFunc()) {
1025:     // We forward some GPRs and some vector types.
1026:     SmallVector<MVT, 2> RegParmTypes;
1027:     MVT IntVT = MVT::i32;
1028:     RegParmTypes.push_back(IntVT);
1029: 
1030:     // Compute the set of forwarded registers. The rest are scratch.
1031:     // ??? what is this for?
1032:     SmallVectorImpl<ForwardedRegister> &Forwards =
1033:         MMFI->getForwardedMustTailRegParms();
1034:     CCInfo.analyzeMustTailForwardedRegisters(Forwards, RegParmTypes, CC_M68k);
1035: 
1036:     // Copy all forwards from physical to virtual registers.
1037:     for (ForwardedRegister &F : Forwards) {
1038:       // FIXME Can we use a less constrained schedule?
1039:       SDValue RegVal = DAG.getCopyFromReg(Chain, DL, F.VReg, F.VT);
1040:       F.VReg = MF.getRegInfo().createVirtualRegister(getRegClassFor(F.VT));
1041:       Chain = DAG.getCopyToReg(Chain, DL, F.VReg, RegVal);
1042:     }
1043:   }
1044: 
1045:   // Some CCs need callee pop.
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1046-1100 / 第 1046-1100 行
```cpp
1046:   if (M68k::isCalleePop(CCID, IsVarArg,
1047:                         MF.getTarget().Options.GuaranteedTailCallOpt)) {
1048:     MMFI->setBytesToPopOnReturn(StackSize); // Callee pops everything.
1049:   } else {
1050:     MMFI->setBytesToPopOnReturn(0); // Callee pops nothing.
1051:     // If this is an sret function, the return should pop the hidden pointer.
1052:     if (!canGuaranteeTCO(CCID) && argsAreStructReturn(Ins) == StackStructReturn)
1053:       MMFI->setBytesToPopOnReturn(4);
1054:   }
1055: 
1056:   MMFI->setArgumentStackSize(StackSize);
1057: 
1058:   return Chain;
1059: }
1060: 
1061: //===----------------------------------------------------------------------===//
1062: //              Return Value Calling Convention Implementation
1063: //===----------------------------------------------------------------------===//
1064: 
1065: bool M68kTargetLowering::CanLowerReturn(
1066:     CallingConv::ID CCID, MachineFunction &MF, bool IsVarArg,
1067:     const SmallVectorImpl<ISD::OutputArg> &Outs, LLVMContext &Context,
1068:     const Type *RetTy) const {
1069:   SmallVector<CCValAssign, 16> RVLocs;
1070:   CCState CCInfo(CCID, IsVarArg, MF, RVLocs, Context);
1071:   return CCInfo.CheckReturn(Outs, RetCC_M68k);
1072: }
1073: 
1074: SDValue
1075: M68kTargetLowering::LowerReturn(SDValue Chain, CallingConv::ID CCID,
1076:                                 bool IsVarArg,
1077:                                 const SmallVectorImpl<ISD::OutputArg> &Outs,
1078:                                 const SmallVectorImpl<SDValue> &OutVals,
1079:                                 const SDLoc &DL, SelectionDAG &DAG) const {
1080:   MachineFunction &MF = DAG.getMachineFunction();
1081:   M68kMachineFunctionInfo *MFI = MF.getInfo<M68kMachineFunctionInfo>();
1082: 
1083:   SmallVector<CCValAssign, 16> RVLocs;
1084:   CCState CCInfo(CCID, IsVarArg, MF, RVLocs, *DAG.getContext());
1085:   CCInfo.AnalyzeReturn(Outs, RetCC_M68k);
1086: 
1087:   SDValue Glue;
1088:   SmallVector<SDValue, 6> RetOps;
1089:   // Operand #0 = Chain (updated below)
1090:   RetOps.push_back(Chain);
1091:   // Operand #1 = Bytes To Pop
1092:   RetOps.push_back(
1093:       DAG.getTargetConstant(MFI->getBytesToPopOnReturn(), DL, MVT::i32));
1094: 
1095:   // Copy the result values into the output registers.
1096:   for (unsigned i = 0, e = RVLocs.size(); i != e; ++i) {
1097:     CCValAssign &VA = RVLocs[i];
1098:     assert(VA.isRegLoc() && "Can only return in registers!");
1099:     SDValue ValToCopy = OutVals[i];
1100:     EVT ValVT = ValToCopy.getValueType();
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::CanLowerReturn`, `M68kTargetLowering::LowerReturn`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::CanLowerReturn`, `M68kTargetLowering::LowerReturn` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1101-1155 / 第 1101-1155 行
```cpp
1101: 
1102:     // Promote values to the appropriate types.
1103:     if (VA.getLocInfo() == CCValAssign::SExt)
1104:       ValToCopy = DAG.getNode(ISD::SIGN_EXTEND, DL, VA.getLocVT(), ValToCopy);
1105:     else if (VA.getLocInfo() == CCValAssign::ZExt)
1106:       ValToCopy = DAG.getNode(ISD::ZERO_EXTEND, DL, VA.getLocVT(), ValToCopy);
1107:     else if (VA.getLocInfo() == CCValAssign::AExt) {
1108:       if (ValVT.isVector() && ValVT.getVectorElementType() == MVT::i1)
1109:         ValToCopy = DAG.getNode(ISD::SIGN_EXTEND, DL, VA.getLocVT(), ValToCopy);
1110:       else
1111:         ValToCopy = DAG.getNode(ISD::ANY_EXTEND, DL, VA.getLocVT(), ValToCopy);
1112:     } else if (VA.getLocInfo() == CCValAssign::BCvt)
1113:       ValToCopy = DAG.getBitcast(VA.getLocVT(), ValToCopy);
1114: 
1115:     Chain = DAG.getCopyToReg(Chain, DL, VA.getLocReg(), ValToCopy, Glue);
1116:     Glue = Chain.getValue(1);
1117:     RetOps.push_back(DAG.getRegister(VA.getLocReg(), VA.getLocVT()));
1118:   }
1119: 
1120:   // Swift calling convention does not require we copy the sret argument
1121:   // into %d0 for the return, and SRetReturnReg is not set for Swift.
1122: 
1123:   // ABI require that for returning structs by value we copy the sret argument
1124:   // into %D0 for the return. Save the argument into a virtual register so that
1125:   // we can access it from the return points.
1126:   //
1127:   // Checking Function.hasStructRetAttr() here is insufficient because the IR
1128:   // may not have an explicit sret argument. If MFI.CanLowerReturn is
1129:   // false, then an sret argument may be implicitly inserted in the SelDAG. In
1130:   // either case MFI->setSRetReturnReg() will have been called.
1131:   if (unsigned SRetReg = MFI->getSRetReturnReg()) {
1132:     // ??? Can i just move this to the top and escape this explanation?
1133:     // When we have both sret and another return value, we should use the
1134:     // original Chain stored in RetOps[0], instead of the current Chain updated
1135:     // in the above loop. If we only have sret, RetOps[0] equals to Chain.
1136: 
1137:     // For the case of sret and another return value, we have
1138:     //   Chain_0 at the function entry
1139:     //   Chain_1 = getCopyToReg(Chain_0) in the above loop
1140:     // If we use Chain_1 in getCopyFromReg, we will have
1141:     //   Val = getCopyFromReg(Chain_1)
1142:     //   Chain_2 = getCopyToReg(Chain_1, Val) from below
1143: 
1144:     // getCopyToReg(Chain_0) will be glued together with
1145:     // getCopyToReg(Chain_1, Val) into Unit A, getCopyFromReg(Chain_1) will be
1146:     // in Unit B, and we will have cyclic dependency between Unit A and Unit B:
1147:     //   Data dependency from Unit B to Unit A due to usage of Val in
1148:     //     getCopyToReg(Chain_1, Val)
1149:     //   Chain dependency from Unit A to Unit B
1150: 
1151:     // So here, we use RetOps[0] (i.e Chain_0) for getCopyFromReg.
1152:     SDValue Val = DAG.getCopyFromReg(RetOps[0], DL, SRetReg,
1153:                                      getPointerTy(MF.getDataLayout()));
1154: 
1155:     // ??? How will this work if CC does not use registers for args passing?
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1156-1210 / 第 1156-1210 行
```cpp
1156:     // ??? What if I return multiple structs?
1157:     unsigned RetValReg = M68k::D0;
1158:     Chain = DAG.getCopyToReg(Chain, DL, RetValReg, Val, Glue);
1159:     Glue = Chain.getValue(1);
1160: 
1161:     RetOps.push_back(
1162:         DAG.getRegister(RetValReg, getPointerTy(DAG.getDataLayout())));
1163:   }
1164: 
1165:   RetOps[0] = Chain; // Update chain.
1166: 
1167:   // Add the glue if we have it.
1168:   if (Glue.getNode())
1169:     RetOps.push_back(Glue);
1170: 
1171:   return DAG.getNode(M68kISD::RET, DL, MVT::Other, RetOps);
1172: }
1173: 
1174: //===----------------------------------------------------------------------===//
1175: //                Fast Calling Convention (tail call) implementation
1176: //===----------------------------------------------------------------------===//
1177: 
1178: //  Like std call, callee cleans arguments, convention except that ECX is
1179: //  reserved for storing the tail called function address. Only 2 registers are
1180: //  free for argument passing (inreg). Tail call optimization is performed
1181: //  provided:
1182: //                * tailcallopt is enabled
1183: //                * caller/callee are fastcc
1184: //  On M68k_64 architecture with GOT-style position independent code only
1185: //  local (within module) calls are supported at the moment. To keep the stack
1186: //  aligned according to platform abi the function GetAlignedArgumentStackSize
1187: //  ensures that argument delta is always multiples of stack alignment. (Dynamic
1188: //  linkers need this - darwin's dyld for example) If a tail called function
1189: //  callee has more arguments than the caller the caller needs to make sure that
1190: //  there is room to move the RETADDR to. This is achieved by reserving an area
1191: //  the size of the argument delta right after the original RETADDR, but before
1192: //  the saved framepointer or the spilled registers e.g. caller(arg1, arg2)
1193: //  calls callee(arg1, arg2,arg3,arg4) stack layout:
1194: //    arg1
1195: //    arg2
1196: //    RETADDR
1197: //    [ new RETADDR
1198: //      move area ]
1199: //    (possible EBP)
1200: //    ESI
1201: //    EDI
1202: //    local1 ..
1203: 
1204: /// Make the stack size align e.g 16n + 12 aligned for a 16-byte align
1205: /// requirement.
1206: unsigned
1207: M68kTargetLowering::GetAlignedArgumentStackSize(unsigned StackSize,
1208:                                                 SelectionDAG &DAG) const {
1209:   const TargetFrameLowering &TFI = *Subtarget.getFrameLowering();
1210:   unsigned StackAlignment = TFI.getStackAlignment();
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::GetAlignedArgumentStackSize`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::GetAlignedArgumentStackSize` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1211-1265 / 第 1211-1265 行
```cpp
1211:   uint64_t AlignMask = StackAlignment - 1;
1212:   int64_t Offset = StackSize;
1213:   unsigned SlotSize = Subtarget.getSlotSize();
1214:   if ((Offset & AlignMask) <= (StackAlignment - SlotSize)) {
1215:     // Number smaller than 12 so just add the difference.
1216:     Offset += ((StackAlignment - SlotSize) - (Offset & AlignMask));
1217:   } else {
1218:     // Mask out lower bits, add stackalignment once plus the 12 bytes.
1219:     Offset =
1220:         ((~AlignMask) & Offset) + StackAlignment + (StackAlignment - SlotSize);
1221:   }
1222:   return Offset;
1223: }
1224: 
1225: /// Check whether the call is eligible for tail call optimization. Targets
1226: /// that want to do tail call optimization should implement this function.
1227: bool M68kTargetLowering::IsEligibleForTailCallOptimization(
1228:     SDValue Callee, CallingConv::ID CalleeCC, bool IsVarArg,
1229:     bool IsCalleeStructRet, bool IsCallerStructRet, Type *RetTy,
1230:     const SmallVectorImpl<ISD::OutputArg> &Outs,
1231:     const SmallVectorImpl<SDValue> &OutVals,
1232:     const SmallVectorImpl<ISD::InputArg> &Ins, SelectionDAG &DAG) const {
1233:   if (!mayTailCallThisCC(CalleeCC))
1234:     return false;
1235: 
1236:   // If -tailcallopt is specified, make fastcc functions tail-callable.
1237:   MachineFunction &MF = DAG.getMachineFunction();
1238:   const auto &CallerF = MF.getFunction();
1239: 
1240:   CallingConv::ID CallerCC = CallerF.getCallingConv();
1241:   bool CCMatch = CallerCC == CalleeCC;
1242: 
1243:   if (DAG.getTarget().Options.GuaranteedTailCallOpt) {
1244:     if (canGuaranteeTCO(CalleeCC) && CCMatch)
1245:       return true;
1246:     return false;
1247:   }
1248: 
1249:   // Look for obvious safe cases to perform tail call optimization that do not
1250:   // require ABI changes. This is what gcc calls sibcall.
1251: 
1252:   // Can't do sibcall if stack needs to be dynamically re-aligned. PEI needs to
1253:   // emit a special epilogue.
1254:   const M68kRegisterInfo *RegInfo = Subtarget.getRegisterInfo();
1255:   if (RegInfo->hasStackRealignment(MF))
1256:     return false;
1257: 
1258:   // Also avoid sibcall optimization if either caller or callee uses struct
1259:   // return semantics.
1260:   if (IsCalleeStructRet || IsCallerStructRet)
1261:     return false;
1262: 
1263:   // Do not sibcall optimize vararg calls unless all arguments are passed via
1264:   // registers.
1265:   LLVMContext &C = *DAG.getContext();
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::IsEligibleForTailCallOptimization`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::IsEligibleForTailCallOptimization` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1266-1320 / 第 1266-1320 行
```cpp
1266:   if (IsVarArg && !Outs.empty()) {
1267: 
1268:     SmallVector<CCValAssign, 16> ArgLocs;
1269:     CCState CCInfo(CalleeCC, IsVarArg, MF, ArgLocs, C);
1270: 
1271:     CCInfo.AnalyzeCallOperands(Outs, CC_M68k);
1272:     for (unsigned i = 0, e = ArgLocs.size(); i != e; ++i)
1273:       if (!ArgLocs[i].isRegLoc())
1274:         return false;
1275:   }
1276: 
1277:   // Check that the call results are passed in the same way.
1278:   if (!CCState::resultsCompatible(CalleeCC, CallerCC, MF, C, Ins, RetCC_M68k,
1279:                                   RetCC_M68k))
1280:     return false;
1281: 
1282:   // The callee has to preserve all registers the caller needs to preserve.
1283:   const M68kRegisterInfo *TRI = Subtarget.getRegisterInfo();
1284:   const uint32_t *CallerPreserved = TRI->getCallPreservedMask(MF, CallerCC);
1285:   if (!CCMatch) {
1286:     const uint32_t *CalleePreserved = TRI->getCallPreservedMask(MF, CalleeCC);
1287:     if (!TRI->regmaskSubsetEqual(CallerPreserved, CalleePreserved))
1288:       return false;
1289:   }
1290: 
1291:   unsigned StackArgsSize = 0;
1292: 
1293:   // If the callee takes no arguments then go on to check the results of the
1294:   // call.
1295:   if (!Outs.empty()) {
1296:     // Check if stack adjustment is needed. For now, do not do this if any
1297:     // argument is passed on the stack.
1298:     SmallVector<CCValAssign, 16> ArgLocs;
1299:     CCState CCInfo(CalleeCC, IsVarArg, MF, ArgLocs, C);
1300: 
1301:     CCInfo.AnalyzeCallOperands(Outs, CC_M68k);
1302:     StackArgsSize = CCInfo.getStackSize();
1303: 
1304:     if (StackArgsSize) {
1305:       // Check if the arguments are already laid out in the right way as
1306:       // the caller's fixed stack objects.
1307:       MachineFrameInfo &MFI = MF.getFrameInfo();
1308:       const MachineRegisterInfo *MRI = &MF.getRegInfo();
1309:       const M68kInstrInfo *TII = Subtarget.getInstrInfo();
1310:       for (unsigned i = 0, e = ArgLocs.size(); i != e; ++i) {
1311:         CCValAssign &VA = ArgLocs[i];
1312:         SDValue Arg = OutVals[i];
1313:         ISD::ArgFlagsTy Flags = Outs[i].Flags;
1314:         if (VA.getLocInfo() == CCValAssign::Indirect)
1315:           return false;
1316:         if (!VA.isRegLoc()) {
1317:           if (!MatchingStackOffset(Arg, VA.getLocMemOffset(), Flags, MFI, MRI,
1318:                                    TII, VA))
1319:             return false;
1320:         }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1321-1375 / 第 1321-1375 行
```cpp
1321:       }
1322:     }
1323: 
1324:     bool PositionIndependent = isPositionIndependent();
1325:     // If the tailcall address may be in a register, then make sure it's
1326:     // possible to register allocate for it. The call address can
1327:     // only target %A0 or %A1 since the tail call must be scheduled after
1328:     // callee-saved registers are restored. These happen to be the same
1329:     // registers used to pass 'inreg' arguments so watch out for those.
1330:     if ((!isa<GlobalAddressSDNode>(Callee) &&
1331:          !isa<ExternalSymbolSDNode>(Callee)) ||
1332:         PositionIndependent) {
1333:       unsigned NumInRegs = 0;
1334:       // In PIC we need an extra register to formulate the address computation
1335:       // for the callee.
1336:       unsigned MaxInRegs = PositionIndependent ? 1 : 2;
1337: 
1338:       for (unsigned i = 0, e = ArgLocs.size(); i != e; ++i) {
1339:         CCValAssign &VA = ArgLocs[i];
1340:         if (!VA.isRegLoc())
1341:           continue;
1342:         Register Reg = VA.getLocReg();
1343:         switch (Reg) {
1344:         default:
1345:           break;
1346:         case M68k::A0:
1347:         case M68k::A1:
1348:           if (++NumInRegs == MaxInRegs)
1349:             return false;
1350:           break;
1351:         }
1352:       }
1353:     }
1354: 
1355:     const MachineRegisterInfo &MRI = MF.getRegInfo();
1356:     if (!parametersInCSRMatch(MRI, CallerPreserved, ArgLocs, OutVals))
1357:       return false;
1358:   }
1359: 
1360:   bool CalleeWillPop = M68k::isCalleePop(
1361:       CalleeCC, IsVarArg, MF.getTarget().Options.GuaranteedTailCallOpt);
1362: 
1363:   if (unsigned BytesToPop =
1364:           MF.getInfo<M68kMachineFunctionInfo>()->getBytesToPopOnReturn()) {
1365:     // If we have bytes to pop, the callee must pop them.
1366:     bool CalleePopMatches = CalleeWillPop && BytesToPop == StackArgsSize;
1367:     if (!CalleePopMatches)
1368:       return false;
1369:   } else if (CalleeWillPop && StackArgsSize > 0) {
1370:     // If we don't have bytes to pop, make sure the callee doesn't pop any.
1371:     return false;
1372:   }
1373: 
1374:   return true;
1375: }
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1376-1430 / 第 1376-1430 行
```cpp
1376: 
1377: //===----------------------------------------------------------------------===//
1378: // Custom Lower
1379: //===----------------------------------------------------------------------===//
1380: 
1381: SDValue M68kTargetLowering::LowerOperation(SDValue Op,
1382:                                            SelectionDAG &DAG) const {
1383:   switch (Op.getOpcode()) {
1384:   default:
1385:     llvm_unreachable("Should not custom lower this!");
1386:   case ISD::SADDO:
1387:   case ISD::UADDO:
1388:   case ISD::SSUBO:
1389:   case ISD::USUBO:
1390:   case ISD::SMULO:
1391:   case ISD::UMULO:
1392:     return LowerXALUO(Op, DAG);
1393:   case ISD::SETCC:
1394:     return LowerSETCC(Op, DAG);
1395:   case ISD::SETCCCARRY:
1396:     return LowerSETCCCARRY(Op, DAG);
1397:   case ISD::SELECT:
1398:     return LowerSELECT(Op, DAG);
1399:   case ISD::BRCOND:
1400:     return LowerBRCOND(Op, DAG);
1401:   case ISD::ADDC:
1402:   case ISD::ADDE:
1403:   case ISD::SUBC:
1404:   case ISD::SUBE:
1405:     return LowerADDC_ADDE_SUBC_SUBE(Op, DAG);
1406:   case ISD::ConstantPool:
1407:     return LowerConstantPool(Op, DAG);
1408:   case ISD::GlobalAddress:
1409:     return LowerGlobalAddress(Op, DAG);
1410:   case ISD::ExternalSymbol:
1411:     return LowerExternalSymbol(Op, DAG);
1412:   case ISD::BlockAddress:
1413:     return LowerBlockAddress(Op, DAG);
1414:   case ISD::JumpTable:
1415:     return LowerJumpTable(Op, DAG);
1416:   case ISD::VASTART:
1417:     return LowerVASTART(Op, DAG);
1418:   case ISD::DYNAMIC_STACKALLOC:
1419:     return LowerDYNAMIC_STACKALLOC(Op, DAG);
1420:   case ISD::SHL_PARTS:
1421:     return LowerShiftLeftParts(Op, DAG);
1422:   case ISD::SRA_PARTS:
1423:     return LowerShiftRightParts(Op, DAG, true);
1424:   case ISD::SRL_PARTS:
1425:     return LowerShiftRightParts(Op, DAG, false);
1426:   case ISD::ATOMIC_FENCE:
1427:     return LowerATOMICFENCE(Op, DAG);
1428:   case ISD::GlobalTLSAddress:
1429:     return LowerGlobalTLSAddress(Op, DAG);
1430:   }
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::LowerOperation`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::LowerOperation` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1431-1485 / 第 1431-1485 行
```cpp
1431: }
1432: 
1433: SDValue M68kTargetLowering::LowerExternalSymbolCall(SelectionDAG &DAG,
1434:                                                     SDLoc Loc,
1435:                                                     llvm::StringRef SymbolName,
1436:                                                     ArgListTy &&ArgList) const {
1437:   PointerType *PtrTy = PointerType::get(*DAG.getContext(), 0);
1438:   CallLoweringInfo CLI(DAG);
1439:   CLI.setDebugLoc(Loc)
1440:       .setChain(DAG.getEntryNode())
1441:       .setLibCallee(CallingConv::C, PtrTy,
1442:                     DAG.getExternalSymbol(SymbolName.data(),
1443:                                           getPointerMemTy(DAG.getDataLayout())),
1444:                     std::move(ArgList));
1445:   return LowerCallTo(CLI).first;
1446: }
1447: 
1448: SDValue M68kTargetLowering::getTLSGetAddr(GlobalAddressSDNode *GA,
1449:                                           SelectionDAG &DAG,
1450:                                           unsigned TargetFlags) const {
1451:   SDValue GOT = DAG.getGLOBAL_OFFSET_TABLE(MVT::i32);
1452:   SDValue TGA = DAG.getTargetGlobalAddress(
1453:       GA->getGlobal(), GA, GA->getValueType(0), GA->getOffset(), TargetFlags);
1454:   SDValue Arg = DAG.getNode(ISD::ADD, SDLoc(GA), MVT::i32, GOT, TGA);
1455: 
1456:   PointerType *PtrTy = PointerType::get(*DAG.getContext(), 0);
1457: 
1458:   ArgListTy Args;
1459:   Args.emplace_back(Arg, PtrTy);
1460:   return LowerExternalSymbolCall(DAG, SDLoc(GA), "__tls_get_addr",
1461:                                  std::move(Args));
1462: }
1463: 
1464: SDValue M68kTargetLowering::getM68kReadTp(SDLoc Loc, SelectionDAG &DAG) const {
1465:   return LowerExternalSymbolCall(DAG, Loc, "__m68k_read_tp", ArgListTy());
1466: }
1467: 
1468: SDValue M68kTargetLowering::LowerTLSGeneralDynamic(GlobalAddressSDNode *GA,
1469:                                                    SelectionDAG &DAG) const {
1470:   return getTLSGetAddr(GA, DAG, M68kII::MO_TLSGD);
1471: }
1472: 
1473: SDValue M68kTargetLowering::LowerTLSLocalDynamic(GlobalAddressSDNode *GA,
1474:                                                  SelectionDAG &DAG) const {
1475:   SDValue Addr = getTLSGetAddr(GA, DAG, M68kII::MO_TLSLDM);
1476:   SDValue TGA =
1477:       DAG.getTargetGlobalAddress(GA->getGlobal(), GA, GA->getValueType(0),
1478:                                  GA->getOffset(), M68kII::MO_TLSLD);
1479:   return DAG.getNode(ISD::ADD, SDLoc(GA), MVT::i32, TGA, Addr);
1480: }
1481: 
1482: SDValue M68kTargetLowering::LowerTLSInitialExec(GlobalAddressSDNode *GA,
1483:                                                 SelectionDAG &DAG) const {
1484:   SDValue GOT = DAG.getGLOBAL_OFFSET_TABLE(MVT::i32);
1485:   SDValue Tp = getM68kReadTp(SDLoc(GA), DAG);
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::LowerExternalSymbolCall`, `M68kTargetLowering::getTLSGetAddr`, `M68kTargetLowering::getM68kReadTp`, `M68kTargetLowering::LowerTLSGeneralDynamic`, `M68kTargetLowering::LowerTLSLocalDynamic`.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::LowerExternalSymbolCall`, `M68kTargetLowering::getTLSGetAddr`, `M68kTargetLowering::getM68kReadTp`, `M68kTargetLowering::LowerTLSGeneralDynamic`, `M68kTargetLowering::LowerTLSLocalDynamic` 等函数。

### Lines 1486-1540 / 第 1486-1540 行
```cpp
1486:   SDValue TGA =
1487:       DAG.getTargetGlobalAddress(GA->getGlobal(), GA, GA->getValueType(0),
1488:                                  GA->getOffset(), M68kII::MO_TLSIE);
1489:   SDValue Addr = DAG.getNode(ISD::ADD, SDLoc(GA), MVT::i32, TGA, GOT);
1490:   SDValue Offset =
1491:       DAG.getLoad(MVT::i32, SDLoc(GA), DAG.getEntryNode(), Addr,
1492:                   MachinePointerInfo::getGOT(DAG.getMachineFunction()));
1493: 
1494:   return DAG.getNode(ISD::ADD, SDLoc(GA), MVT::i32, Offset, Tp);
1495: }
1496: 
1497: SDValue M68kTargetLowering::LowerTLSLocalExec(GlobalAddressSDNode *GA,
1498:                                               SelectionDAG &DAG) const {
1499:   SDValue Tp = getM68kReadTp(SDLoc(GA), DAG);
1500:   SDValue TGA =
1501:       DAG.getTargetGlobalAddress(GA->getGlobal(), GA, GA->getValueType(0),
1502:                                  GA->getOffset(), M68kII::MO_TLSLE);
1503:   return DAG.getNode(ISD::ADD, SDLoc(GA), MVT::i32, TGA, Tp);
1504: }
1505: 
1506: SDValue M68kTargetLowering::LowerGlobalTLSAddress(SDValue Op,
1507:                                                   SelectionDAG &DAG) const {
1508:   assert(Subtarget.isTargetELF());
1509: 
1510:   auto *GA = cast<GlobalAddressSDNode>(Op);
1511:   TLSModel::Model AccessModel = DAG.getTarget().getTLSModel(GA->getGlobal());
1512: 
1513:   switch (AccessModel) {
1514:   case TLSModel::GeneralDynamic:
1515:     return LowerTLSGeneralDynamic(GA, DAG);
1516:   case TLSModel::LocalDynamic:
1517:     return LowerTLSLocalDynamic(GA, DAG);
1518:   case TLSModel::InitialExec:
1519:     return LowerTLSInitialExec(GA, DAG);
1520:   case TLSModel::LocalExec:
1521:     return LowerTLSLocalExec(GA, DAG);
1522:   }
1523: 
1524:   llvm_unreachable("Unexpected TLS access model type");
1525: }
1526: 
1527: bool M68kTargetLowering::decomposeMulByConstant(LLVMContext &Context, EVT VT,
1528:                                                 SDValue C) const {
1529:   // Shifts and add instructions in M68000 and M68010 support
1530:   // up to 32 bits, but mul only has 16-bit variant. So it's almost
1531:   // certainly beneficial to lower 8/16/32-bit mul to their
1532:   // add / shifts counterparts. But for 64-bits mul, it might be
1533:   // safer to just leave it to compiler runtime implementations.
1534:   return VT.bitsLE(MVT::i32) || Subtarget.atLeastM68020();
1535: }
1536: 
1537: static bool isOverflowArithmetic(unsigned Opcode) {
1538:   switch (Opcode) {
1539:   case ISD::UADDO:
1540:   case ISD::SADDO:
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::LowerTLSLocalExec`, `M68kTargetLowering::LowerGlobalTLSAddress`, `M68kTargetLowering::decomposeMulByConstant`, `isOverflowArithmetic`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::LowerTLSLocalExec`, `M68kTargetLowering::LowerGlobalTLSAddress`, `M68kTargetLowering::decomposeMulByConstant`, `isOverflowArithmetic` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1541-1595 / 第 1541-1595 行
```cpp
1541:   case ISD::USUBO:
1542:   case ISD::SSUBO:
1543:   case ISD::UMULO:
1544:   case ISD::SMULO:
1545:     return true;
1546:   default:
1547:     return false;
1548:   }
1549: }
1550: 
1551: static void lowerOverflowArithmetic(SDValue Op, SelectionDAG &DAG,
1552:                                     SDValue &Result, SDValue &CCR,
1553:                                     unsigned &CC) {
1554:   SDNode *N = Op.getNode();
1555:   EVT VT = N->getValueType(0);
1556:   SDValue LHS = N->getOperand(0);
1557:   SDValue RHS = N->getOperand(1);
1558:   SDLoc DL(Op);
1559: 
1560:   unsigned TruncOp = 0;
1561:   auto PromoteMULO = [&](unsigned ExtOp) {
1562:     // We don't have 8-bit multiplications, so promote i8 version of U/SMULO
1563:     // to i16.
1564:     // Ideally this should be done by legalizer but sadly there is no promotion
1565:     // rule for U/SMULO at this moment.
1566:     if (VT == MVT::i8) {
1567:       LHS = DAG.getNode(ExtOp, DL, MVT::i16, LHS);
1568:       RHS = DAG.getNode(ExtOp, DL, MVT::i16, RHS);
1569:       VT = MVT::i16;
1570:       TruncOp = ISD::TRUNCATE;
1571:     }
1572:   };
1573: 
1574:   bool NoOverflow = false;
1575:   unsigned BaseOp = 0;
1576:   switch (Op.getOpcode()) {
1577:   default:
1578:     llvm_unreachable("Unknown ovf instruction!");
1579:   case ISD::SADDO:
1580:     BaseOp = M68kISD::ADD;
1581:     CC = M68k::COND_VS;
1582:     break;
1583:   case ISD::UADDO:
1584:     BaseOp = M68kISD::ADD;
1585:     CC = M68k::COND_CS;
1586:     break;
1587:   case ISD::SSUBO:
1588:     BaseOp = M68kISD::SUB;
1589:     CC = M68k::COND_VS;
1590:     break;
1591:   case ISD::USUBO:
1592:     BaseOp = M68kISD::SUB;
1593:     CC = M68k::COND_CS;
1594:     break;
1595:   case ISD::UMULO:
```
- **EN**: The range implements or declares functions including `lowerOverflowArithmetic`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `lowerOverflowArithmetic` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1596-1650 / 第 1596-1650 行
```cpp
1596:     PromoteMULO(ISD::ZERO_EXTEND);
1597:     NoOverflow = VT != MVT::i32;
1598:     BaseOp = NoOverflow ? (unsigned)ISD::MUL : (unsigned)M68kISD::UMUL;
1599:     CC = M68k::COND_VS;
1600:     break;
1601:   case ISD::SMULO:
1602:     PromoteMULO(ISD::SIGN_EXTEND);
1603:     NoOverflow = VT != MVT::i32;
1604:     BaseOp = NoOverflow ? (unsigned)ISD::MUL : (unsigned)M68kISD::SMUL;
1605:     CC = M68k::COND_VS;
1606:     break;
1607:   }
1608: 
1609:   SDVTList VTs;
1610:   if (NoOverflow)
1611:     VTs = DAG.getVTList(VT);
1612:   else
1613:     // Also sets CCR.
1614:     VTs = DAG.getVTList(VT, MVT::i8);
1615: 
1616:   SDValue Arith = DAG.getNode(BaseOp, DL, VTs, LHS, RHS);
1617:   Result = Arith.getValue(0);
1618:   if (TruncOp)
1619:     // Right now the only place to truncate is from i16 to i8.
1620:     Result = DAG.getNode(TruncOp, DL, MVT::i8, Arith);
1621: 
1622:   if (NoOverflow)
1623:     CCR = DAG.getConstant(0, DL, N->getValueType(1));
1624:   else
1625:     CCR = Arith.getValue(1);
1626: }
1627: 
1628: SDValue M68kTargetLowering::LowerXALUO(SDValue Op, SelectionDAG &DAG) const {
1629:   SDNode *N = Op.getNode();
1630:   SDLoc DL(Op);
1631: 
1632:   // Lower the "add/sub/mul with overflow" instruction into a regular ins plus
1633:   // a "setcc" instruction that checks the overflow flag.
1634:   SDValue Result, CCR;
1635:   unsigned CC;
1636:   lowerOverflowArithmetic(Op, DAG, Result, CCR, CC);
1637: 
1638:   SDValue Overflow;
1639:   if (isa<ConstantSDNode>(CCR)) {
1640:     // It's likely a result of operations that will not overflow
1641:     // hence no setcc is needed.
1642:     Overflow = CCR;
1643:   } else {
1644:     // Generate a M68kISD::SETCC.
1645:     Overflow = DAG.getNode(M68kISD::SETCC, DL, N->getValueType(1),
1646:                            DAG.getConstant(CC, DL, MVT::i8), CCR);
1647:   }
1648: 
1649:   return DAG.getNode(ISD::MERGE_VALUES, DL, N->getVTList(), Result, Overflow);
1650: }
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::LowerXALUO`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::LowerXALUO` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1651-1705 / 第 1651-1705 行
```cpp
1651: 
1652: /// Create a BTST (Bit Test) node - Test bit \p BitNo in \p Src and set
1653: /// condition according to equal/not-equal condition code \p CC.
1654: static SDValue getBitTestCondition(SDValue Src, SDValue BitNo, ISD::CondCode CC,
1655:                                    const SDLoc &DL, SelectionDAG &DAG) {
1656:   // If Src is i8, promote it to i32 with any_extend.  There is no i8 BTST
1657:   // instruction.  Since the shift amount is in-range-or-undefined, we know
1658:   // that doing a bittest on the i32 value is ok.
1659:   if (Src.getValueType() == MVT::i8 || Src.getValueType() == MVT::i16)
1660:     Src = DAG.getNode(ISD::ANY_EXTEND, DL, MVT::i32, Src);
1661: 
1662:   // If the operand types disagree, extend the shift amount to match.  Since
1663:   // BTST ignores high bits (like shifts) we can use anyextend.
1664:   if (Src.getValueType() != BitNo.getValueType())
1665:     BitNo = DAG.getNode(ISD::ANY_EXTEND, DL, Src.getValueType(), BitNo);
1666: 
1667:   SDValue BTST = DAG.getNode(M68kISD::BTST, DL, MVT::i8, Src, BitNo);
1668: 
1669:   // NOTE BTST sets CCR.Z flag if bit is 0, same as AND with bitmask
1670:   M68k::CondCode Cond = CC == ISD::SETEQ ? M68k::COND_EQ : M68k::COND_NE;
1671:   return DAG.getNode(M68kISD::SETCC, DL, MVT::i8,
1672:                      DAG.getConstant(Cond, DL, MVT::i8), BTST);
1673: }
1674: 
1675: /// Result of 'and' is compared against zero. Change to a BTST node if possible.
1676: static SDValue LowerAndToBTST(SDValue And, ISD::CondCode CC, const SDLoc &DL,
1677:                               SelectionDAG &DAG) {
1678:   SDValue Op0 = And.getOperand(0);
1679:   SDValue Op1 = And.getOperand(1);
1680:   if (Op0.getOpcode() == ISD::TRUNCATE)
1681:     Op0 = Op0.getOperand(0);
1682:   if (Op1.getOpcode() == ISD::TRUNCATE)
1683:     Op1 = Op1.getOperand(0);
1684: 
1685:   SDValue LHS, RHS;
1686:   if (Op1.getOpcode() == ISD::SHL)
1687:     std::swap(Op0, Op1);
1688:   if (Op0.getOpcode() == ISD::SHL) {
1689:     if (isOneConstant(Op0.getOperand(0))) {
1690:       // If we looked past a truncate, check that it's only truncating away
1691:       // known zeros.
1692:       unsigned BitWidth = Op0.getValueSizeInBits();
1693:       unsigned AndBitWidth = And.getValueSizeInBits();
1694:       if (BitWidth > AndBitWidth) {
1695:         auto Known = DAG.computeKnownBits(Op0);
1696:         if (Known.countMinLeadingZeros() < BitWidth - AndBitWidth)
1697:           return SDValue();
1698:       }
1699:       LHS = Op1;
1700:       RHS = Op0.getOperand(1);
1701:     }
1702:   } else if (auto *AndRHS = dyn_cast<ConstantSDNode>(Op1)) {
1703:     uint64_t AndRHSVal = AndRHS->getZExtValue();
1704:     SDValue AndLHS = Op0;
1705: 
```
- **EN**: The range implements or declares functions including `getBitTestCondition`, `LowerAndToBTST`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `getBitTestCondition`, `LowerAndToBTST` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1706-1760 / 第 1706-1760 行
```cpp
1706:     if (AndRHSVal == 1 && AndLHS.getOpcode() == ISD::SRL) {
1707:       LHS = AndLHS.getOperand(0);
1708:       RHS = AndLHS.getOperand(1);
1709:     }
1710: 
1711:     // Use BTST if the immediate can't be encoded in a TEST instruction.
1712:     if (!isUInt<32>(AndRHSVal) && isPowerOf2_64(AndRHSVal)) {
1713:       LHS = AndLHS;
1714:       RHS = DAG.getConstant(Log2_64_Ceil(AndRHSVal), DL, LHS.getValueType());
1715:     }
1716:   }
1717: 
1718:   if (LHS.getNode())
1719:     return getBitTestCondition(LHS, RHS, CC, DL, DAG);
1720: 
1721:   return SDValue();
1722: }
1723: 
1724: static M68k::CondCode TranslateIntegerM68kCC(ISD::CondCode SetCCOpcode) {
1725:   switch (SetCCOpcode) {
1726:   default:
1727:     llvm_unreachable("Invalid integer condition!");
1728:   case ISD::SETEQ:
1729:     return M68k::COND_EQ;
1730:   case ISD::SETGT:
1731:     return M68k::COND_GT;
1732:   case ISD::SETGE:
1733:     return M68k::COND_GE;
1734:   case ISD::SETLT:
1735:     return M68k::COND_LT;
1736:   case ISD::SETLE:
1737:     return M68k::COND_LE;
1738:   case ISD::SETNE:
1739:     return M68k::COND_NE;
1740:   case ISD::SETULT:
1741:     return M68k::COND_CS;
1742:   case ISD::SETUGE:
1743:     return M68k::COND_CC;
1744:   case ISD::SETUGT:
1745:     return M68k::COND_HI;
1746:   case ISD::SETULE:
1747:     return M68k::COND_LS;
1748:   }
1749: }
1750: 
1751: /// Do a one-to-one translation of a ISD::CondCode to the M68k-specific
1752: /// condition code, returning the condition code and the LHS/RHS of the
1753: /// comparison to make.
1754: static unsigned TranslateM68kCC(ISD::CondCode SetCCOpcode, const SDLoc &DL,
1755:                                 bool IsFP, SDValue &LHS, SDValue &RHS,
1756:                                 SelectionDAG &DAG) {
1757:   if (!IsFP) {
1758:     if (ConstantSDNode *RHSC = dyn_cast<ConstantSDNode>(RHS)) {
1759:       if (SetCCOpcode == ISD::SETGT && RHSC->isAllOnes()) {
1760:         // X > -1   -> X == 0, jump !sign.
```
- **EN**: The range implements or declares functions including `TranslateIntegerM68kCC`, `TranslateM68kCC`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `TranslateIntegerM68kCC`, `TranslateM68kCC` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1761-1815 / 第 1761-1815 行
```cpp
1761:         RHS = DAG.getConstant(0, DL, RHS.getValueType());
1762:         return M68k::COND_PL;
1763:       }
1764:       if (SetCCOpcode == ISD::SETLT && RHSC->isZero()) {
1765:         // X < 0   -> X == 0, jump on sign.
1766:         return M68k::COND_MI;
1767:       }
1768:       if (SetCCOpcode == ISD::SETLT && RHSC->getZExtValue() == 1) {
1769:         // X < 1   -> X <= 0
1770:         RHS = DAG.getConstant(0, DL, RHS.getValueType());
1771:         return M68k::COND_LE;
1772:       }
1773:     }
1774: 
1775:     return TranslateIntegerM68kCC(SetCCOpcode);
1776:   }
1777: 
1778:   // First determine if it is required or is profitable to flip the operands.
1779: 
1780:   // If LHS is a foldable load, but RHS is not, flip the condition.
1781:   if (ISD::isNON_EXTLoad(LHS.getNode()) && !ISD::isNON_EXTLoad(RHS.getNode())) {
1782:     SetCCOpcode = getSetCCSwappedOperands(SetCCOpcode);
1783:     std::swap(LHS, RHS);
1784:   }
1785: 
1786:   switch (SetCCOpcode) {
1787:   default:
1788:     break;
1789:   case ISD::SETOLT:
1790:   case ISD::SETOLE:
1791:   case ISD::SETUGT:
1792:   case ISD::SETUGE:
1793:     std::swap(LHS, RHS);
1794:     break;
1795:   }
1796: 
1797:   // On a floating point condition, the flags are set as follows:
1798:   // ZF  PF  CF   op
1799:   //  0 | 0 | 0 | X > Y
1800:   //  0 | 0 | 1 | X < Y
1801:   //  1 | 0 | 0 | X == Y
1802:   //  1 | 1 | 1 | unordered
1803:   switch (SetCCOpcode) {
1804:   default:
1805:     llvm_unreachable("Condcode should be pre-legalized away");
1806:   case ISD::SETUEQ:
1807:   case ISD::SETEQ:
1808:     return M68k::COND_EQ;
1809:   case ISD::SETOLT: // flipped
1810:   case ISD::SETOGT:
1811:   case ISD::SETGT:
1812:     return M68k::COND_HI;
1813:   case ISD::SETOLE: // flipped
1814:   case ISD::SETOGE:
1815:   case ISD::SETGE:
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 1816-1870 / 第 1816-1870 行
```cpp
1816:     return M68k::COND_CC;
1817:   case ISD::SETUGT: // flipped
1818:   case ISD::SETULT:
1819:   case ISD::SETLT:
1820:     return M68k::COND_CS;
1821:   case ISD::SETUGE: // flipped
1822:   case ISD::SETULE:
1823:   case ISD::SETLE:
1824:     return M68k::COND_LS;
1825:   case ISD::SETONE:
1826:   case ISD::SETNE:
1827:     return M68k::COND_NE;
1828:   case ISD::SETOEQ:
1829:   case ISD::SETUNE:
1830:     return M68k::COND_INVALID;
1831:   }
1832: }
1833: 
1834: // Convert (truncate (srl X, N) to i1) to (bt X, N)
1835: static SDValue LowerTruncateToBTST(SDValue Op, ISD::CondCode CC,
1836:                                    const SDLoc &DL, SelectionDAG &DAG) {
1837: 
1838:   assert(Op.getOpcode() == ISD::TRUNCATE && Op.getValueType() == MVT::i1 &&
1839:          "Expected TRUNCATE to i1 node");
1840: 
1841:   if (Op.getOperand(0).getOpcode() != ISD::SRL)
1842:     return SDValue();
1843: 
1844:   SDValue ShiftRight = Op.getOperand(0);
1845:   return getBitTestCondition(ShiftRight.getOperand(0), ShiftRight.getOperand(1),
1846:                              CC, DL, DAG);
1847: }
1848: 
1849: /// \brief return true if \c Op has a use that doesn't just read flags.
1850: static bool hasNonFlagsUse(SDValue Op) {
1851:   for (SDNode::use_iterator UI = Op->use_begin(), UE = Op->use_end(); UI != UE;
1852:        ++UI) {
1853:     SDNode *User = UI->getUser();
1854:     unsigned UOpNo = UI->getOperandNo();
1855:     if (User->getOpcode() == ISD::TRUNCATE && User->hasOneUse()) {
1856:       // Look past truncate.
1857:       UOpNo = User->use_begin()->getOperandNo();
1858:       User = User->use_begin()->getUser();
1859:     }
1860: 
1861:     if (User->getOpcode() != ISD::BRCOND && User->getOpcode() != ISD::SETCC &&
1862:         !(User->getOpcode() == ISD::SELECT && UOpNo == 0))
1863:       return true;
1864:   }
1865:   return false;
1866: }
1867: 
1868: SDValue M68kTargetLowering::EmitTest(SDValue Op, unsigned M68kCC,
1869:                                      const SDLoc &DL, SelectionDAG &DAG) const {
1870: 
```
- **EN**: The range implements or declares functions including `LowerTruncateToBTST`, `hasNonFlagsUse`, `M68kTargetLowering::EmitTest`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `LowerTruncateToBTST`, `hasNonFlagsUse`, `M68kTargetLowering::EmitTest` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 1871-1925 / 第 1871-1925 行
```cpp
1871:   // CF and OF aren't always set the way we want. Determine which
1872:   // of these we need.
1873:   bool NeedCF = false;
1874:   bool NeedOF = false;
1875:   switch (M68kCC) {
1876:   default:
1877:     break;
1878:   case M68k::COND_HI:
1879:   case M68k::COND_CC:
1880:   case M68k::COND_CS:
1881:   case M68k::COND_LS:
1882:     NeedCF = true;
1883:     break;
1884:   case M68k::COND_GT:
1885:   case M68k::COND_GE:
1886:   case M68k::COND_LT:
1887:   case M68k::COND_LE:
1888:   case M68k::COND_VS:
1889:   case M68k::COND_VC: {
1890:     // Check if we really need to set the
1891:     // Overflow flag. If NoSignedWrap is present
1892:     // that is not actually needed.
1893:     switch (Op->getOpcode()) {
1894:     case ISD::ADD:
1895:     case ISD::SUB:
1896:     case ISD::MUL:
1897:     case ISD::SHL: {
1898:       if (Op.getNode()->getFlags().hasNoSignedWrap())
1899:         break;
1900:       [[fallthrough]];
1901:     }
1902:     default:
1903:       NeedOF = true;
1904:       break;
1905:     }
1906:     break;
1907:   }
1908:   }
1909:   // See if we can use the CCR value from the operand instead of
1910:   // doing a separate TEST. TEST always sets OF and CF to 0, so unless
1911:   // we prove that the arithmetic won't overflow, we can't use OF or CF.
1912:   if (Op.getResNo() != 0 || NeedOF || NeedCF) {
1913:     // Emit a CMP with 0, which is the TEST pattern.
1914:     return DAG.getNode(M68kISD::CMP, DL, MVT::i8,
1915:                        DAG.getConstant(0, DL, Op.getValueType()), Op);
1916:   }
1917:   unsigned Opcode = 0;
1918:   unsigned NumOperands = 0;
1919: 
1920:   // Truncate operations may prevent the merge of the SETCC instruction
1921:   // and the arithmetic instruction before it. Attempt to truncate the operands
1922:   // of the arithmetic instruction and use a reduced bit-width instruction.
1923:   bool NeedTruncation = false;
1924:   SDValue ArithOp = Op;
1925:   if (Op->getOpcode() == ISD::TRUNCATE && Op->hasOneUse()) {
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1926-1980 / 第 1926-1980 行
```cpp
1926:     SDValue Arith = Op->getOperand(0);
1927:     // Both the trunc and the arithmetic op need to have one user each.
1928:     if (Arith->hasOneUse())
1929:       switch (Arith.getOpcode()) {
1930:       default:
1931:         break;
1932:       case ISD::ADD:
1933:       case ISD::SUB:
1934:       case ISD::AND:
1935:       case ISD::OR:
1936:       case ISD::XOR: {
1937:         NeedTruncation = true;
1938:         ArithOp = Arith;
1939:       }
1940:       }
1941:   }
1942: 
1943:   // NOTICE: In the code below we use ArithOp to hold the arithmetic operation
1944:   // which may be the result of a CAST.  We use the variable 'Op', which is the
1945:   // non-casted variable when we check for possible users.
1946:   switch (ArithOp.getOpcode()) {
1947:   case ISD::ADD:
1948:     Opcode = M68kISD::ADD;
1949:     NumOperands = 2;
1950:     break;
1951:   case ISD::SHL:
1952:   case ISD::SRL:
1953:     // If we have a constant logical shift that's only used in a comparison
1954:     // against zero turn it into an equivalent AND. This allows turning it into
1955:     // a TEST instruction later.
1956:     if ((M68kCC == M68k::COND_EQ || M68kCC == M68k::COND_NE) &&
1957:         Op->hasOneUse() && isa<ConstantSDNode>(Op->getOperand(1)) &&
1958:         !hasNonFlagsUse(Op)) {
1959:       EVT VT = Op.getValueType();
1960:       unsigned BitWidth = VT.getSizeInBits();
1961:       unsigned ShAmt = Op->getConstantOperandVal(1);
1962:       if (ShAmt >= BitWidth) // Avoid undefined shifts.
1963:         break;
1964:       APInt Mask = ArithOp.getOpcode() == ISD::SRL
1965:                        ? APInt::getHighBitsSet(BitWidth, BitWidth - ShAmt)
1966:                        : APInt::getLowBitsSet(BitWidth, BitWidth - ShAmt);
1967:       if (!Mask.isSignedIntN(32)) // Avoid large immediates.
1968:         break;
1969:       Op = DAG.getNode(ISD::AND, DL, VT, Op->getOperand(0),
1970:                        DAG.getConstant(Mask, DL, VT));
1971:     }
1972:     break;
1973: 
1974:   case ISD::AND:
1975:     // If the primary 'and' result isn't used, don't bother using
1976:     // M68kISD::AND, because a TEST instruction will be better.
1977:     if (!hasNonFlagsUse(Op)) {
1978:       SDValue Op0 = ArithOp->getOperand(0);
1979:       SDValue Op1 = ArithOp->getOperand(1);
1980:       EVT VT = ArithOp.getValueType();
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 1981-2035 / 第 1981-2035 行
```cpp
1981:       bool IsAndn = isBitwiseNot(Op0) || isBitwiseNot(Op1);
1982:       bool IsLegalAndnType = VT == MVT::i32 || VT == MVT::i64;
1983: 
1984:       // But if we can combine this into an ANDN operation, then create an AND
1985:       // now and allow it to be pattern matched into an ANDN.
1986:       if (/*!Subtarget.hasBMI() ||*/ !IsAndn || !IsLegalAndnType)
1987:         break;
1988:     }
1989:     [[fallthrough]];
1990:   case ISD::SUB:
1991:   case ISD::OR:
1992:   case ISD::XOR:
1993:     // Due to the ISEL shortcoming noted above, be conservative if this op is
1994:     // likely to be selected as part of a load-modify-store instruction.
1995:     for (const auto *U : Op.getNode()->users())
1996:       if (U->getOpcode() == ISD::STORE)
1997:         goto default_case;
1998: 
1999:     // Otherwise use a regular CCR-setting instruction.
2000:     switch (ArithOp.getOpcode()) {
2001:     default:
2002:       llvm_unreachable("unexpected operator!");
2003:     case ISD::SUB:
2004:       Opcode = M68kISD::SUB;
2005:       break;
2006:     case ISD::XOR:
2007:       Opcode = M68kISD::XOR;
2008:       break;
2009:     case ISD::AND:
2010:       Opcode = M68kISD::AND;
2011:       break;
2012:     case ISD::OR:
2013:       Opcode = M68kISD::OR;
2014:       break;
2015:     }
2016: 
2017:     NumOperands = 2;
2018:     break;
2019:   case M68kISD::ADD:
2020:   case M68kISD::SUB:
2021:   case M68kISD::OR:
2022:   case M68kISD::XOR:
2023:   case M68kISD::AND:
2024:     return SDValue(Op.getNode(), 1);
2025:   default:
2026:   default_case:
2027:     break;
2028:   }
2029: 
2030:   // If we found that truncation is beneficial, perform the truncation and
2031:   // update 'Op'.
2032:   if (NeedTruncation) {
2033:     EVT VT = Op.getValueType();
2034:     SDValue WideVal = Op->getOperand(0);
2035:     EVT WideVT = WideVal.getValueType();
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 2036-2090 / 第 2036-2090 行
```cpp
2036:     unsigned ConvertedOp = 0;
2037:     // Use a target machine opcode to prevent further DAGCombine
2038:     // optimizations that may separate the arithmetic operations
2039:     // from the setcc node.
2040:     switch (WideVal.getOpcode()) {
2041:     default:
2042:       break;
2043:     case ISD::ADD:
2044:       ConvertedOp = M68kISD::ADD;
2045:       break;
2046:     case ISD::SUB:
2047:       ConvertedOp = M68kISD::SUB;
2048:       break;
2049:     case ISD::AND:
2050:       ConvertedOp = M68kISD::AND;
2051:       break;
2052:     case ISD::OR:
2053:       ConvertedOp = M68kISD::OR;
2054:       break;
2055:     case ISD::XOR:
2056:       ConvertedOp = M68kISD::XOR;
2057:       break;
2058:     }
2059: 
2060:     if (ConvertedOp) {
2061:       const TargetLowering &TLI = DAG.getTargetLoweringInfo();
2062:       if (TLI.isOperationLegal(WideVal.getOpcode(), WideVT)) {
2063:         SDValue V0 = DAG.getNode(ISD::TRUNCATE, DL, VT, WideVal.getOperand(0));
2064:         SDValue V1 = DAG.getNode(ISD::TRUNCATE, DL, VT, WideVal.getOperand(1));
2065:         Op = DAG.getNode(ConvertedOp, DL, VT, V0, V1);
2066:       }
2067:     }
2068:   }
2069: 
2070:   if (Opcode == 0) {
2071:     // Emit a CMP with 0, which is the TEST pattern.
2072:     return DAG.getNode(M68kISD::CMP, DL, MVT::i8,
2073:                        DAG.getConstant(0, DL, Op.getValueType()), Op);
2074:   }
2075:   SDVTList VTs = DAG.getVTList(Op.getValueType(), MVT::i8);
2076:   SmallVector<SDValue, 4> Ops(Op->op_begin(), Op->op_begin() + NumOperands);
2077: 
2078:   SDValue New = DAG.getNode(Opcode, DL, VTs, Ops);
2079:   DAG.ReplaceAllUsesWith(Op, New);
2080:   return SDValue(New.getNode(), 1);
2081: }
2082: 
2083: /// \brief Return true if the condition is an unsigned comparison operation.
2084: static bool isM68kCCUnsigned(unsigned M68kCC) {
2085:   switch (M68kCC) {
2086:   default:
2087:     llvm_unreachable("Invalid integer condition!");
2088:   case M68k::COND_EQ:
2089:   case M68k::COND_NE:
2090:   case M68k::COND_CS:
```
- **EN**: The range implements or declares functions including `isM68kCCUnsigned`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isM68kCCUnsigned` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2091-2145 / 第 2091-2145 行
```cpp
2091:   case M68k::COND_HI:
2092:   case M68k::COND_LS:
2093:   case M68k::COND_CC:
2094:     return true;
2095:   case M68k::COND_GT:
2096:   case M68k::COND_GE:
2097:   case M68k::COND_LT:
2098:   case M68k::COND_LE:
2099:     return false;
2100:   }
2101: }
2102: 
2103: SDValue M68kTargetLowering::EmitCmp(SDValue Op0, SDValue Op1, unsigned M68kCC,
2104:                                     const SDLoc &DL, SelectionDAG &DAG) const {
2105:   if (isNullConstant(Op1))
2106:     return EmitTest(Op0, M68kCC, DL, DAG);
2107: 
2108:   assert(!(isa<ConstantSDNode>(Op1) && Op0.getValueType() == MVT::i1) &&
2109:          "Unexpected comparison operation for MVT::i1 operands");
2110: 
2111:   if ((Op0.getValueType() == MVT::i8 || Op0.getValueType() == MVT::i16 ||
2112:        Op0.getValueType() == MVT::i32 || Op0.getValueType() == MVT::i64)) {
2113:     // Only promote the compare up to I32 if it is a 16 bit operation
2114:     // with an immediate.  16 bit immediates are to be avoided.
2115:     if ((Op0.getValueType() == MVT::i16 &&
2116:          (isa<ConstantSDNode>(Op0) || isa<ConstantSDNode>(Op1))) &&
2117:         !DAG.getMachineFunction().getFunction().hasMinSize()) {
2118:       unsigned ExtendOp =
2119:           isM68kCCUnsigned(M68kCC) ? ISD::ZERO_EXTEND : ISD::SIGN_EXTEND;
2120:       Op0 = DAG.getNode(ExtendOp, DL, MVT::i32, Op0);
2121:       Op1 = DAG.getNode(ExtendOp, DL, MVT::i32, Op1);
2122:     }
2123:     // Use SUB instead of CMP to enable CSE between SUB and CMP.
2124:     SDVTList VTs = DAG.getVTList(Op0.getValueType(), MVT::i8);
2125:     SDValue Sub = DAG.getNode(M68kISD::SUB, DL, VTs, Op0, Op1);
2126:     return SDValue(Sub.getNode(), 1);
2127:   }
2128:   return DAG.getNode(M68kISD::CMP, DL, MVT::i8, Op0, Op1);
2129: }
2130: 
2131: /// Result of 'and' or 'trunc to i1' is compared against zero.
2132: /// Change to a BTST node if possible.
2133: SDValue M68kTargetLowering::LowerToBTST(SDValue Op, ISD::CondCode CC,
2134:                                         const SDLoc &DL,
2135:                                         SelectionDAG &DAG) const {
2136:   if (Op.getOpcode() == ISD::AND)
2137:     return LowerAndToBTST(Op, CC, DL, DAG);
2138:   if (Op.getOpcode() == ISD::TRUNCATE && Op.getValueType() == MVT::i1)
2139:     return LowerTruncateToBTST(Op, CC, DL, DAG);
2140:   return SDValue();
2141: }
2142: 
2143: SDValue M68kTargetLowering::LowerSETCC(SDValue Op, SelectionDAG &DAG) const {
2144:   MVT VT = Op.getSimpleValueType();
2145:   assert(VT == MVT::i8 && "SetCC type must be 8-bit integer");
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::EmitCmp`, `M68kTargetLowering::LowerToBTST`, `M68kTargetLowering::LowerSETCC`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::EmitCmp`, `M68kTargetLowering::LowerToBTST`, `M68kTargetLowering::LowerSETCC` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 2146-2200 / 第 2146-2200 行
```cpp
2146: 
2147:   SDValue Op0 = Op.getOperand(0);
2148:   SDValue Op1 = Op.getOperand(1);
2149:   SDLoc DL(Op);
2150:   ISD::CondCode CC = cast<CondCodeSDNode>(Op.getOperand(2))->get();
2151: 
2152:   // Optimize to BTST if possible.
2153:   // Lower (X & (1 << N)) == 0 to BTST(X, N).
2154:   // Lower ((X >>u N) & 1) != 0 to BTST(X, N).
2155:   // Lower ((X >>s N) & 1) != 0 to BTST(X, N).
2156:   // Lower (trunc (X >> N) to i1) to BTST(X, N).
2157:   if (Op0.hasOneUse() && isNullConstant(Op1) &&
2158:       (CC == ISD::SETEQ || CC == ISD::SETNE)) {
2159:     if (SDValue NewSetCC = LowerToBTST(Op0, CC, DL, DAG)) {
2160:       if (VT == MVT::i1)
2161:         return DAG.getNode(ISD::TRUNCATE, DL, MVT::i1, NewSetCC);
2162:       return NewSetCC;
2163:     }
2164:   }
2165: 
2166:   // Look for X == 0, X == 1, X != 0, or X != 1.  We can simplify some forms of
2167:   // these.
2168:   if ((isOneConstant(Op1) || isNullConstant(Op1)) &&
2169:       (CC == ISD::SETEQ || CC == ISD::SETNE)) {
2170: 
2171:     // If the input is a setcc, then reuse the input setcc or use a new one with
2172:     // the inverted condition.
2173:     if (Op0.getOpcode() == M68kISD::SETCC) {
2174:       M68k::CondCode CCode = (M68k::CondCode)Op0.getConstantOperandVal(0);
2175:       bool Invert = (CC == ISD::SETNE) ^ isNullConstant(Op1);
2176:       if (!Invert)
2177:         return Op0;
2178: 
2179:       CCode = M68k::GetOppositeBranchCondition(CCode);
2180:       SDValue SetCC =
2181:           DAG.getNode(M68kISD::SETCC, DL, MVT::i8,
2182:                       DAG.getConstant(CCode, DL, MVT::i8), Op0.getOperand(1));
2183:       if (VT == MVT::i1)
2184:         return DAG.getNode(ISD::TRUNCATE, DL, MVT::i1, SetCC);
2185:       return SetCC;
2186:     }
2187:   }
2188:   if (Op0.getValueType() == MVT::i1 && (CC == ISD::SETEQ || CC == ISD::SETNE)) {
2189:     if (isOneConstant(Op1)) {
2190:       ISD::CondCode NewCC = ISD::GlobalISel::getSetCCInverse(CC, true);
2191:       return DAG.getSetCC(DL, VT, Op0, DAG.getConstant(0, DL, MVT::i1), NewCC);
2192:     }
2193:     if (!isNullConstant(Op1)) {
2194:       SDValue Xor = DAG.getNode(ISD::XOR, DL, MVT::i1, Op0, Op1);
2195:       return DAG.getSetCC(DL, VT, Xor, DAG.getConstant(0, DL, MVT::i1), CC);
2196:     }
2197:   }
2198: 
2199:   bool IsFP = Op1.getSimpleValueType().isFloatingPoint();
2200:   unsigned M68kCC = TranslateM68kCC(CC, DL, IsFP, Op0, Op1, DAG);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2201-2255 / 第 2201-2255 行
```cpp
2201:   if (M68kCC == M68k::COND_INVALID)
2202:     return SDValue();
2203: 
2204:   SDValue CCR = EmitCmp(Op0, Op1, M68kCC, DL, DAG);
2205:   return DAG.getNode(M68kISD::SETCC, DL, MVT::i8,
2206:                      DAG.getConstant(M68kCC, DL, MVT::i8), CCR);
2207: }
2208: 
2209: SDValue M68kTargetLowering::LowerSETCCCARRY(SDValue Op,
2210:                                             SelectionDAG &DAG) const {
2211:   SDValue LHS = Op.getOperand(0);
2212:   SDValue RHS = Op.getOperand(1);
2213:   SDValue Carry = Op.getOperand(2);
2214:   SDValue Cond = Op.getOperand(3);
2215:   SDLoc DL(Op);
2216: 
2217:   assert(LHS.getSimpleValueType().isInteger() && "SETCCCARRY is integer only.");
2218:   M68k::CondCode CC = TranslateIntegerM68kCC(cast<CondCodeSDNode>(Cond)->get());
2219: 
2220:   EVT CarryVT = Carry.getValueType();
2221:   APInt NegOne = APInt::getAllOnes(CarryVT.getScalarSizeInBits());
2222:   Carry = DAG.getNode(M68kISD::ADD, DL, DAG.getVTList(CarryVT, MVT::i32), Carry,
2223:                       DAG.getConstant(NegOne, DL, CarryVT));
2224: 
2225:   SDVTList VTs = DAG.getVTList(LHS.getValueType(), MVT::i32);
2226:   SDValue Cmp =
2227:       DAG.getNode(M68kISD::SUBX, DL, VTs, LHS, RHS, Carry.getValue(1));
2228: 
2229:   return DAG.getNode(M68kISD::SETCC, DL, MVT::i8,
2230:                      DAG.getConstant(CC, DL, MVT::i8), Cmp.getValue(1));
2231: }
2232: 
2233: /// Return true if opcode is a M68k logical comparison.
2234: static bool isM68kLogicalCmp(SDValue Op) {
2235:   unsigned Opc = Op.getNode()->getOpcode();
2236:   if (Opc == M68kISD::CMP)
2237:     return true;
2238:   if (Op.getResNo() == 1 &&
2239:       (Opc == M68kISD::ADD || Opc == M68kISD::SUB || Opc == M68kISD::ADDX ||
2240:        Opc == M68kISD::SUBX || Opc == M68kISD::SMUL || Opc == M68kISD::UMUL ||
2241:        Opc == M68kISD::OR || Opc == M68kISD::XOR || Opc == M68kISD::AND))
2242:     return true;
2243: 
2244:   if (Op.getResNo() == 2 && Opc == M68kISD::UMUL)
2245:     return true;
2246: 
2247:   return false;
2248: }
2249: 
2250: static bool isTruncWithZeroHighBitsInput(SDValue V, SelectionDAG &DAG) {
2251:   if (V.getOpcode() != ISD::TRUNCATE)
2252:     return false;
2253: 
2254:   SDValue VOp0 = V.getOperand(0);
2255:   unsigned InBits = VOp0.getValueSizeInBits();
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::LowerSETCCCARRY`, `isM68kLogicalCmp`, `isTruncWithZeroHighBitsInput`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::LowerSETCCCARRY`, `isM68kLogicalCmp`, `isTruncWithZeroHighBitsInput` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 2256-2310 / 第 2256-2310 行
```cpp
2256:   unsigned Bits = V.getValueSizeInBits();
2257:   return DAG.MaskedValueIsZero(VOp0,
2258:                                APInt::getHighBitsSet(InBits, InBits - Bits));
2259: }
2260: 
2261: SDValue M68kTargetLowering::LowerSELECT(SDValue Op, SelectionDAG &DAG) const {
2262:   bool addTest = true;
2263:   SDValue Cond = Op.getOperand(0);
2264:   SDValue Op1 = Op.getOperand(1);
2265:   SDValue Op2 = Op.getOperand(2);
2266:   SDLoc DL(Op);
2267:   SDValue CC;
2268: 
2269:   if (Cond.getOpcode() == ISD::SETCC) {
2270:     if (SDValue NewCond = LowerSETCC(Cond, DAG))
2271:       Cond = NewCond;
2272:   }
2273: 
2274:   // (select (x == 0), -1, y) -> (sign_bit (x - 1)) | y
2275:   // (select (x == 0), y, -1) -> ~(sign_bit (x - 1)) | y
2276:   // (select (x != 0), y, -1) -> (sign_bit (x - 1)) | y
2277:   // (select (x != 0), -1, y) -> ~(sign_bit (x - 1)) | y
2278:   if (Cond.getOpcode() == M68kISD::SETCC &&
2279:       Cond.getOperand(1).getOpcode() == M68kISD::CMP &&
2280:       isNullConstant(Cond.getOperand(1).getOperand(0))) {
2281:     SDValue Cmp = Cond.getOperand(1);
2282: 
2283:     unsigned CondCode = Cond.getConstantOperandVal(0);
2284: 
2285:     if ((isAllOnesConstant(Op1) || isAllOnesConstant(Op2)) &&
2286:         (CondCode == M68k::COND_EQ || CondCode == M68k::COND_NE)) {
2287:       SDValue Y = isAllOnesConstant(Op2) ? Op1 : Op2;
2288: 
2289:       SDValue CmpOp0 = Cmp.getOperand(1);
2290:       // Apply further optimizations for special cases
2291:       // (select (x != 0), -1, 0) -> neg & sbb
2292:       // (select (x == 0), 0, -1) -> neg & sbb
2293:       if (isNullConstant(Y) &&
2294:           (isAllOnesConstant(Op1) == (CondCode == M68k::COND_NE))) {
2295: 
2296:         SDVTList VTs = DAG.getVTList(CmpOp0.getValueType(), MVT::i32);
2297: 
2298:         SDValue Neg =
2299:             DAG.getNode(M68kISD::SUB, DL, VTs,
2300:                         DAG.getConstant(0, DL, CmpOp0.getValueType()), CmpOp0);
2301: 
2302:         SDValue Res = DAG.getNode(M68kISD::SETCC_CARRY, DL, Op.getValueType(),
2303:                                   DAG.getConstant(M68k::COND_CS, DL, MVT::i8),
2304:                                   SDValue(Neg.getNode(), 1));
2305:         return Res;
2306:       }
2307: 
2308:       Cmp = DAG.getNode(M68kISD::CMP, DL, MVT::i8,
2309:                         DAG.getConstant(1, DL, CmpOp0.getValueType()), CmpOp0);
2310: 
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::LowerSELECT`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::LowerSELECT` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2311-2365 / 第 2311-2365 行
```cpp
2311:       SDValue Res = // Res = 0 or -1.
2312:           DAG.getNode(M68kISD::SETCC_CARRY, DL, Op.getValueType(),
2313:                       DAG.getConstant(M68k::COND_CS, DL, MVT::i8), Cmp);
2314: 
2315:       if (isAllOnesConstant(Op1) != (CondCode == M68k::COND_EQ))
2316:         Res = DAG.getNOT(DL, Res, Res.getValueType());
2317: 
2318:       if (!isNullConstant(Op2))
2319:         Res = DAG.getNode(ISD::OR, DL, Res.getValueType(), Res, Y);
2320:       return Res;
2321:     }
2322:   }
2323: 
2324:   // Look past (and (setcc_carry (cmp ...)), 1).
2325:   if (Cond.getOpcode() == ISD::AND &&
2326:       Cond.getOperand(0).getOpcode() == M68kISD::SETCC_CARRY &&
2327:       isOneConstant(Cond.getOperand(1)))
2328:     Cond = Cond.getOperand(0);
2329: 
2330:   // If condition flag is set by a M68kISD::CMP, then use it as the condition
2331:   // setting operand in place of the M68kISD::SETCC.
2332:   unsigned CondOpcode = Cond.getOpcode();
2333:   if (CondOpcode == M68kISD::SETCC || CondOpcode == M68kISD::SETCC_CARRY) {
2334:     CC = Cond.getOperand(0);
2335: 
2336:     SDValue Cmp = Cond.getOperand(1);
2337:     unsigned Opc = Cmp.getOpcode();
2338: 
2339:     bool IllegalFPCMov = false;
2340: 
2341:     if ((isM68kLogicalCmp(Cmp) && !IllegalFPCMov) || Opc == M68kISD::BTST) {
2342:       Cond = Cmp;
2343:       addTest = false;
2344:     }
2345:   } else if (isOverflowArithmetic(CondOpcode)) {
2346:     // Result is unused here.
2347:     SDValue Result;
2348:     unsigned CCode;
2349:     lowerOverflowArithmetic(Cond, DAG, Result, Cond, CCode);
2350:     CC = DAG.getConstant(CCode, DL, MVT::i8);
2351:     addTest = false;
2352:   }
2353: 
2354:   if (addTest) {
2355:     // Look past the truncate if the high bits are known zero.
2356:     if (isTruncWithZeroHighBitsInput(Cond, DAG))
2357:       Cond = Cond.getOperand(0);
2358: 
2359:     // We know the result of AND is compared against zero. Try to match
2360:     // it to BT.
2361:     if (Cond.getOpcode() == ISD::AND && Cond.hasOneUse()) {
2362:       if (SDValue NewSetCC = LowerToBTST(Cond, ISD::SETNE, DL, DAG)) {
2363:         CC = NewSetCC.getOperand(0);
2364:         Cond = NewSetCC.getOperand(1);
2365:         addTest = false;
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2366-2420 / 第 2366-2420 行
```cpp
2366:       }
2367:     }
2368:   }
2369: 
2370:   if (addTest) {
2371:     CC = DAG.getConstant(M68k::COND_NE, DL, MVT::i8);
2372:     Cond = EmitTest(Cond, M68k::COND_NE, DL, DAG);
2373:   }
2374: 
2375:   // a <  b ? -1 :  0 -> RES = ~setcc_carry
2376:   // a <  b ?  0 : -1 -> RES = setcc_carry
2377:   // a >= b ? -1 :  0 -> RES = setcc_carry
2378:   // a >= b ?  0 : -1 -> RES = ~setcc_carry
2379:   if (Cond.getOpcode() == M68kISD::SUB) {
2380:     unsigned CondCode = CC->getAsZExtVal();
2381: 
2382:     if ((CondCode == M68k::COND_CC || CondCode == M68k::COND_CS) &&
2383:         (isAllOnesConstant(Op1) || isAllOnesConstant(Op2)) &&
2384:         (isNullConstant(Op1) || isNullConstant(Op2))) {
2385:       SDValue Res =
2386:           DAG.getNode(M68kISD::SETCC_CARRY, DL, Op.getValueType(),
2387:                       DAG.getConstant(M68k::COND_CS, DL, MVT::i8), Cond);
2388:       if (isAllOnesConstant(Op1) != (CondCode == M68k::COND_CS))
2389:         return DAG.getNOT(DL, Res, Res.getValueType());
2390:       return Res;
2391:     }
2392:   }
2393: 
2394:   // M68k doesn't have an i8 cmov. If both operands are the result of a
2395:   // truncate widen the cmov and push the truncate through. This avoids
2396:   // introducing a new branch during isel and doesn't add any extensions.
2397:   if (Op.getValueType() == MVT::i8 && Op1.getOpcode() == ISD::TRUNCATE &&
2398:       Op2.getOpcode() == ISD::TRUNCATE) {
2399:     SDValue T1 = Op1.getOperand(0), T2 = Op2.getOperand(0);
2400:     if (T1.getValueType() == T2.getValueType() &&
2401:         // Block CopyFromReg so partial register stalls are avoided.
2402:         T1.getOpcode() != ISD::CopyFromReg &&
2403:         T2.getOpcode() != ISD::CopyFromReg) {
2404:       SDValue Cmov =
2405:           DAG.getNode(M68kISD::CMOV, DL, T1.getValueType(), T2, T1, CC, Cond);
2406:       return DAG.getNode(ISD::TRUNCATE, DL, Op.getValueType(), Cmov);
2407:     }
2408:   }
2409: 
2410:   // Simple optimization when Cond is a constant to avoid generating
2411:   // M68kISD::CMOV if possible.
2412:   // TODO: Generalize this to use SelectionDAG::computeKnownBits.
2413:   if (auto *Const = dyn_cast<ConstantSDNode>(Cond.getNode())) {
2414:     const APInt &C = Const->getAPIntValue();
2415:     if (C.countr_zero() >= 5)
2416:       return Op2;
2417:     else if (C.countr_one() >= 5)
2418:       return Op1;
2419:   }
2420: 
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2421-2475 / 第 2421-2475 行
```cpp
2421:   // M68kISD::CMOV means set the result (which is operand 1) to the RHS if
2422:   // condition is true.
2423:   SDValue Ops[] = {Op2, Op1, CC, Cond};
2424:   return DAG.getNode(M68kISD::CMOV, DL, Op.getValueType(), Ops);
2425: }
2426: 
2427: /// Return true if node is an ISD::AND or ISD::OR of two M68k::SETcc nodes
2428: /// each of which has no other use apart from the AND / OR.
2429: static bool isAndOrOfSetCCs(SDValue Op, unsigned &Opc) {
2430:   Opc = Op.getOpcode();
2431:   if (Opc != ISD::OR && Opc != ISD::AND)
2432:     return false;
2433:   return (M68k::IsSETCC(Op.getOperand(0).getOpcode()) &&
2434:           Op.getOperand(0).hasOneUse() &&
2435:           M68k::IsSETCC(Op.getOperand(1).getOpcode()) &&
2436:           Op.getOperand(1).hasOneUse());
2437: }
2438: 
2439: /// Return true if node is an ISD::XOR of a M68kISD::SETCC and 1 and that the
2440: /// SETCC node has a single use.
2441: static bool isXor1OfSetCC(SDValue Op) {
2442:   if (Op.getOpcode() != ISD::XOR)
2443:     return false;
2444:   if (isOneConstant(Op.getOperand(1)))
2445:     return Op.getOperand(0).getOpcode() == M68kISD::SETCC &&
2446:            Op.getOperand(0).hasOneUse();
2447:   return false;
2448: }
2449: 
2450: SDValue M68kTargetLowering::LowerBRCOND(SDValue Op, SelectionDAG &DAG) const {
2451:   bool AddTest = true;
2452:   SDValue Chain = Op.getOperand(0);
2453:   SDValue Cond = Op.getOperand(1);
2454:   SDValue Dest = Op.getOperand(2);
2455:   SDLoc DL(Op);
2456:   SDValue CC;
2457:   bool Inverted = false;
2458: 
2459:   if (Cond.getOpcode() == ISD::SETCC) {
2460:     // Check for setcc([su]{add,sub}o == 0).
2461:     if (cast<CondCodeSDNode>(Cond.getOperand(2))->get() == ISD::SETEQ &&
2462:         isNullConstant(Cond.getOperand(1)) &&
2463:         Cond.getOperand(0).getResNo() == 1 &&
2464:         (Cond.getOperand(0).getOpcode() == ISD::SADDO ||
2465:          Cond.getOperand(0).getOpcode() == ISD::UADDO ||
2466:          Cond.getOperand(0).getOpcode() == ISD::SSUBO ||
2467:          Cond.getOperand(0).getOpcode() == ISD::USUBO)) {
2468:       Inverted = true;
2469:       Cond = Cond.getOperand(0);
2470:     } else {
2471:       if (SDValue NewCond = LowerSETCC(Cond, DAG))
2472:         Cond = NewCond;
2473:     }
2474:   }
2475: 
```
- **EN**: The range implements or declares functions including `isAndOrOfSetCCs`, `isXor1OfSetCC`, `M68kTargetLowering::LowerBRCOND`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `isAndOrOfSetCCs`, `isXor1OfSetCC`, `M68kTargetLowering::LowerBRCOND` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2476-2530 / 第 2476-2530 行
```cpp
2476:   // Look pass (and (setcc_carry (cmp ...)), 1).
2477:   if (Cond.getOpcode() == ISD::AND &&
2478:       Cond.getOperand(0).getOpcode() == M68kISD::SETCC_CARRY &&
2479:       isOneConstant(Cond.getOperand(1)))
2480:     Cond = Cond.getOperand(0);
2481: 
2482:   // If condition flag is set by a M68kISD::CMP, then use it as the condition
2483:   // setting operand in place of the M68kISD::SETCC.
2484:   unsigned CondOpcode = Cond.getOpcode();
2485:   if (CondOpcode == M68kISD::SETCC || CondOpcode == M68kISD::SETCC_CARRY) {
2486:     CC = Cond.getOperand(0);
2487: 
2488:     SDValue Cmp = Cond.getOperand(1);
2489:     unsigned Opc = Cmp.getOpcode();
2490: 
2491:     if (isM68kLogicalCmp(Cmp) || Opc == M68kISD::BTST) {
2492:       Cond = Cmp;
2493:       AddTest = false;
2494:     } else {
2495:       switch (CC->getAsZExtVal()) {
2496:       default:
2497:         break;
2498:       case M68k::COND_VS:
2499:       case M68k::COND_CS:
2500:         // These can only come from an arithmetic instruction with overflow,
2501:         // e.g. SADDO, UADDO.
2502:         Cond = Cond.getNode()->getOperand(1);
2503:         AddTest = false;
2504:         break;
2505:       }
2506:     }
2507:   }
2508:   CondOpcode = Cond.getOpcode();
2509:   if (isOverflowArithmetic(CondOpcode)) {
2510:     SDValue Result;
2511:     unsigned CCode;
2512:     lowerOverflowArithmetic(Cond, DAG, Result, Cond, CCode);
2513: 
2514:     if (Inverted)
2515:       CCode = M68k::GetOppositeBranchCondition((M68k::CondCode)CCode);
2516:     CC = DAG.getConstant(CCode, DL, MVT::i8);
2517: 
2518:     AddTest = false;
2519:   } else {
2520:     unsigned CondOpc;
2521:     if (Cond.hasOneUse() && isAndOrOfSetCCs(Cond, CondOpc)) {
2522:       SDValue Cmp = Cond.getOperand(0).getOperand(1);
2523:       if (CondOpc == ISD::OR) {
2524:         // Also, recognize the pattern generated by an FCMP_UNE. We can emit
2525:         // two branches instead of an explicit OR instruction with a
2526:         // separate test.
2527:         if (Cmp == Cond.getOperand(1).getOperand(1) && isM68kLogicalCmp(Cmp)) {
2528:           CC = Cond.getOperand(0).getOperand(0);
2529:           Chain = DAG.getNode(M68kISD::BRCOND, DL, Op.getValueType(), Chain,
2530:                               Dest, CC, Cmp);
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2531-2585 / 第 2531-2585 行
```cpp
2531:           CC = Cond.getOperand(1).getOperand(0);
2532:           Cond = Cmp;
2533:           AddTest = false;
2534:         }
2535:       } else { // ISD::AND
2536:         // Also, recognize the pattern generated by an FCMP_OEQ. We can emit
2537:         // two branches instead of an explicit AND instruction with a
2538:         // separate test. However, we only do this if this block doesn't
2539:         // have a fall-through edge, because this requires an explicit
2540:         // jmp when the condition is false.
2541:         if (Cmp == Cond.getOperand(1).getOperand(1) && isM68kLogicalCmp(Cmp) &&
2542:             Op.getNode()->hasOneUse()) {
2543:           M68k::CondCode CCode =
2544:               (M68k::CondCode)Cond.getOperand(0).getConstantOperandVal(0);
2545:           CCode = M68k::GetOppositeBranchCondition(CCode);
2546:           CC = DAG.getConstant(CCode, DL, MVT::i8);
2547:           SDNode *User = *Op.getNode()->user_begin();
2548:           // Look for an unconditional branch following this conditional branch.
2549:           // We need this because we need to reverse the successors in order
2550:           // to implement FCMP_OEQ.
2551:           if (User->getOpcode() == ISD::BR) {
2552:             SDValue FalseBB = User->getOperand(1);
2553:             SDNode *NewBR =
2554:                 DAG.UpdateNodeOperands(User, User->getOperand(0), Dest);
2555:             assert(NewBR == User);
2556:             (void)NewBR;
2557:             Dest = FalseBB;
2558: 
2559:             Chain = DAG.getNode(M68kISD::BRCOND, DL, Op.getValueType(), Chain,
2560:                                 Dest, CC, Cmp);
2561:             M68k::CondCode CCode =
2562:                 (M68k::CondCode)Cond.getOperand(1).getConstantOperandVal(0);
2563:             CCode = M68k::GetOppositeBranchCondition(CCode);
2564:             CC = DAG.getConstant(CCode, DL, MVT::i8);
2565:             Cond = Cmp;
2566:             AddTest = false;
2567:           }
2568:         }
2569:       }
2570:     } else if (Cond.hasOneUse() && isXor1OfSetCC(Cond)) {
2571:       // Recognize for xorb (setcc), 1 patterns. The xor inverts the condition.
2572:       // It should be transformed during dag combiner except when the condition
2573:       // is set by a arithmetics with overflow node.
2574:       M68k::CondCode CCode =
2575:           (M68k::CondCode)Cond.getOperand(0).getConstantOperandVal(0);
2576:       CCode = M68k::GetOppositeBranchCondition(CCode);
2577:       CC = DAG.getConstant(CCode, DL, MVT::i8);
2578:       Cond = Cond.getOperand(0).getOperand(1);
2579:       AddTest = false;
2580:     }
2581:   }
2582: 
2583:   if (AddTest) {
2584:     // Look pass the truncate if the high bits are known zero.
2585:     if (isTruncWithZeroHighBitsInput(Cond, DAG))
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 2586-2640 / 第 2586-2640 行
```cpp
2586:       Cond = Cond.getOperand(0);
2587: 
2588:     // We know the result is compared against zero. Try to match it to BT.
2589:     if (Cond.hasOneUse()) {
2590:       if (SDValue NewSetCC = LowerToBTST(Cond, ISD::SETNE, DL, DAG)) {
2591:         CC = NewSetCC.getOperand(0);
2592:         Cond = NewSetCC.getOperand(1);
2593:         AddTest = false;
2594:       }
2595:     }
2596:   }
2597: 
2598:   if (AddTest) {
2599:     M68k::CondCode MxCond = Inverted ? M68k::COND_EQ : M68k::COND_NE;
2600:     CC = DAG.getConstant(MxCond, DL, MVT::i8);
2601:     Cond = EmitTest(Cond, MxCond, DL, DAG);
2602:   }
2603:   return DAG.getNode(M68kISD::BRCOND, DL, Op.getValueType(), Chain, Dest, CC,
2604:                      Cond);
2605: }
2606: 
2607: SDValue M68kTargetLowering::LowerADDC_ADDE_SUBC_SUBE(SDValue Op,
2608:                                                      SelectionDAG &DAG) const {
2609:   MVT VT = Op.getNode()->getSimpleValueType(0);
2610: 
2611:   // Let legalize expand this if it isn't a legal type yet.
2612:   if (!DAG.getTargetLoweringInfo().isTypeLegal(VT))
2613:     return SDValue();
2614: 
2615:   SDVTList VTs = DAG.getVTList(VT, MVT::i8);
2616: 
2617:   unsigned Opc;
2618:   bool ExtraOp = false;
2619:   switch (Op.getOpcode()) {
2620:   default:
2621:     llvm_unreachable("Invalid code");
2622:   case ISD::ADDC:
2623:     Opc = M68kISD::ADD;
2624:     break;
2625:   case ISD::ADDE:
2626:     Opc = M68kISD::ADDX;
2627:     ExtraOp = true;
2628:     break;
2629:   case ISD::SUBC:
2630:     Opc = M68kISD::SUB;
2631:     break;
2632:   case ISD::SUBE:
2633:     Opc = M68kISD::SUBX;
2634:     ExtraOp = true;
2635:     break;
2636:   }
2637: 
2638:   if (!ExtraOp)
2639:     return DAG.getNode(Opc, SDLoc(Op), VTs, Op.getOperand(0), Op.getOperand(1));
2640:   return DAG.getNode(Opc, SDLoc(Op), VTs, Op.getOperand(0), Op.getOperand(1),
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::LowerADDC_ADDE_SUBC_SUBE`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::LowerADDC_ADDE_SUBC_SUBE` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2641-2695 / 第 2641-2695 行
```cpp
2641:                      Op.getOperand(2));
2642: }
2643: 
2644: // ConstantPool, JumpTable, GlobalAddress, and ExternalSymbol are lowered as
2645: // their target countpart wrapped in the M68kISD::Wrapper node. Suppose N is
2646: // one of the above mentioned nodes. It has to be wrapped because otherwise
2647: // Select(N) returns N. So the raw TargetGlobalAddress nodes, etc. can only
2648: // be used to form addressing mode. These wrapped nodes will be selected
2649: // into MOV32ri.
2650: SDValue M68kTargetLowering::LowerConstantPool(SDValue Op,
2651:                                               SelectionDAG &DAG) const {
2652:   ConstantPoolSDNode *CP = cast<ConstantPoolSDNode>(Op);
2653: 
2654:   // In PIC mode (unless we're in PCRel PIC mode) we add an offset to the
2655:   // global base reg.
2656:   unsigned char OpFlag = Subtarget.classifyLocalReference(nullptr);
2657: 
2658:   unsigned WrapperKind = M68kISD::Wrapper;
2659:   if (M68kII::isPCRelGlobalReference(OpFlag)) {
2660:     WrapperKind = M68kISD::WrapperPC;
2661:   }
2662: 
2663:   MVT PtrVT = getPointerTy(DAG.getDataLayout());
2664:   SDValue Result = DAG.getTargetConstantPool(
2665:       CP->getConstVal(), PtrVT, CP->getAlign(), CP->getOffset(), OpFlag);
2666: 
2667:   SDLoc DL(CP);
2668:   Result = DAG.getNode(WrapperKind, DL, PtrVT, Result);
2669: 
2670:   // With PIC, the address is actually $g + Offset.
2671:   if (M68kII::isGlobalRelativeToPICBase(OpFlag)) {
2672:     Result = DAG.getNode(ISD::ADD, DL, PtrVT,
2673:                          DAG.getNode(M68kISD::GLOBAL_BASE_REG, SDLoc(), PtrVT),
2674:                          Result);
2675:   }
2676: 
2677:   return Result;
2678: }
2679: 
2680: SDValue M68kTargetLowering::LowerExternalSymbol(SDValue Op,
2681:                                                 SelectionDAG &DAG) const {
2682:   const char *Sym = cast<ExternalSymbolSDNode>(Op)->getSymbol();
2683: 
2684:   // In PIC mode (unless we're in PCRel PIC mode) we add an offset to the
2685:   // global base reg.
2686:   const Module *Mod = DAG.getMachineFunction().getFunction().getParent();
2687:   unsigned char OpFlag = Subtarget.classifyExternalReference(*Mod);
2688: 
2689:   unsigned WrapperKind = M68kISD::Wrapper;
2690:   if (M68kII::isPCRelGlobalReference(OpFlag)) {
2691:     WrapperKind = M68kISD::WrapperPC;
2692:   }
2693: 
2694:   auto PtrVT = getPointerTy(DAG.getDataLayout());
2695:   SDValue Result = DAG.getTargetExternalSymbol(Sym, PtrVT, OpFlag);
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::LowerConstantPool`, `M68kTargetLowering::LowerExternalSymbol`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::LowerConstantPool`, `M68kTargetLowering::LowerExternalSymbol` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2696-2750 / 第 2696-2750 行
```cpp
2696: 
2697:   SDLoc DL(Op);
2698:   Result = DAG.getNode(WrapperKind, DL, PtrVT, Result);
2699: 
2700:   // With PIC, the address is actually $g + Offset.
2701:   if (M68kII::isGlobalRelativeToPICBase(OpFlag)) {
2702:     Result = DAG.getNode(ISD::ADD, DL, PtrVT,
2703:                          DAG.getNode(M68kISD::GLOBAL_BASE_REG, SDLoc(), PtrVT),
2704:                          Result);
2705:   }
2706: 
2707:   // For symbols that require a load from a stub to get the address, emit the
2708:   // load.
2709:   if (M68kII::isGlobalStubReference(OpFlag)) {
2710:     Result = DAG.getLoad(PtrVT, DL, DAG.getEntryNode(), Result,
2711:                          MachinePointerInfo::getGOT(DAG.getMachineFunction()));
2712:   }
2713: 
2714:   return Result;
2715: }
2716: 
2717: SDValue M68kTargetLowering::LowerBlockAddress(SDValue Op,
2718:                                               SelectionDAG &DAG) const {
2719:   unsigned char OpFlags = Subtarget.classifyBlockAddressReference();
2720:   const BlockAddress *BA = cast<BlockAddressSDNode>(Op)->getBlockAddress();
2721:   int64_t Offset = cast<BlockAddressSDNode>(Op)->getOffset();
2722:   SDLoc DL(Op);
2723:   auto PtrVT = getPointerTy(DAG.getDataLayout());
2724: 
2725:   // Create the TargetBlockAddressAddress node.
2726:   SDValue Result = DAG.getTargetBlockAddress(BA, PtrVT, Offset, OpFlags);
2727: 
2728:   if (M68kII::isPCRelBlockReference(OpFlags)) {
2729:     Result = DAG.getNode(M68kISD::WrapperPC, DL, PtrVT, Result);
2730:   } else {
2731:     Result = DAG.getNode(M68kISD::Wrapper, DL, PtrVT, Result);
2732:   }
2733: 
2734:   // With PIC, the address is actually $g + Offset.
2735:   if (M68kII::isGlobalRelativeToPICBase(OpFlags)) {
2736:     Result =
2737:         DAG.getNode(ISD::ADD, DL, PtrVT,
2738:                     DAG.getNode(M68kISD::GLOBAL_BASE_REG, DL, PtrVT), Result);
2739:   }
2740: 
2741:   return Result;
2742: }
2743: 
2744: SDValue M68kTargetLowering::LowerGlobalAddress(const GlobalValue *GV,
2745:                                                const SDLoc &DL, int64_t Offset,
2746:                                                SelectionDAG &DAG) const {
2747:   unsigned char OpFlags = Subtarget.classifyGlobalReference(GV);
2748:   auto PtrVT = getPointerTy(DAG.getDataLayout());
2749: 
2750:   // Create the TargetGlobalAddress node, folding in the constant
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::LowerBlockAddress`, `M68kTargetLowering::LowerGlobalAddress`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::LowerBlockAddress`, `M68kTargetLowering::LowerGlobalAddress` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2751-2805 / 第 2751-2805 行
```cpp
2751:   // offset if it is legal.
2752:   SDValue Result;
2753:   if (M68kII::isDirectGlobalReference(OpFlags)) {
2754:     Result = DAG.getTargetGlobalAddress(GV, DL, PtrVT, Offset);
2755:     Offset = 0;
2756:   } else {
2757:     Result = DAG.getTargetGlobalAddress(GV, DL, PtrVT, 0, OpFlags);
2758:   }
2759: 
2760:   if (M68kII::isPCRelGlobalReference(OpFlags))
2761:     Result = DAG.getNode(M68kISD::WrapperPC, DL, PtrVT, Result);
2762:   else
2763:     Result = DAG.getNode(M68kISD::Wrapper, DL, PtrVT, Result);
2764: 
2765:   // With PIC, the address is actually $g + Offset.
2766:   if (M68kII::isGlobalRelativeToPICBase(OpFlags)) {
2767:     Result =
2768:         DAG.getNode(ISD::ADD, DL, PtrVT,
2769:                     DAG.getNode(M68kISD::GLOBAL_BASE_REG, DL, PtrVT), Result);
2770:   }
2771: 
2772:   // For globals that require a load from a stub to get the address, emit the
2773:   // load.
2774:   if (M68kII::isGlobalStubReference(OpFlags)) {
2775:     Result = DAG.getLoad(PtrVT, DL, DAG.getEntryNode(), Result,
2776:                          MachinePointerInfo::getGOT(DAG.getMachineFunction()));
2777:   }
2778: 
2779:   // If there was a non-zero offset that we didn't fold, create an explicit
2780:   // addition for it.
2781:   if (Offset != 0) {
2782:     Result = DAG.getNode(ISD::ADD, DL, PtrVT, Result,
2783:                          DAG.getConstant(Offset, DL, PtrVT));
2784:   }
2785: 
2786:   return Result;
2787: }
2788: 
2789: SDValue M68kTargetLowering::LowerGlobalAddress(SDValue Op,
2790:                                                SelectionDAG &DAG) const {
2791:   const GlobalValue *GV = cast<GlobalAddressSDNode>(Op)->getGlobal();
2792:   int64_t Offset = cast<GlobalAddressSDNode>(Op)->getOffset();
2793:   return LowerGlobalAddress(GV, SDLoc(Op), Offset, DAG);
2794: }
2795: 
2796: //===----------------------------------------------------------------------===//
2797: // Custom Lower Jump Table
2798: //===----------------------------------------------------------------------===//
2799: 
2800: SDValue M68kTargetLowering::LowerJumpTable(SDValue Op,
2801:                                            SelectionDAG &DAG) const {
2802:   JumpTableSDNode *JT = cast<JumpTableSDNode>(Op);
2803: 
2804:   // In PIC mode (unless we're in PCRel PIC mode) we add an offset to the
2805:   // global base reg.
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::LowerGlobalAddress`, `M68kTargetLowering::LowerJumpTable`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::LowerGlobalAddress`, `M68kTargetLowering::LowerJumpTable` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2806-2860 / 第 2806-2860 行
```cpp
2806:   unsigned char OpFlag = Subtarget.classifyLocalReference(nullptr);
2807: 
2808:   unsigned WrapperKind = M68kISD::Wrapper;
2809:   if (M68kII::isPCRelGlobalReference(OpFlag)) {
2810:     WrapperKind = M68kISD::WrapperPC;
2811:   }
2812: 
2813:   auto PtrVT = getPointerTy(DAG.getDataLayout());
2814:   SDValue Result = DAG.getTargetJumpTable(JT->getIndex(), PtrVT, OpFlag);
2815:   SDLoc DL(JT);
2816:   Result = DAG.getNode(WrapperKind, DL, PtrVT, Result);
2817: 
2818:   // With PIC, the address is actually $g + Offset.
2819:   if (M68kII::isGlobalRelativeToPICBase(OpFlag)) {
2820:     Result = DAG.getNode(ISD::ADD, DL, PtrVT,
2821:                          DAG.getNode(M68kISD::GLOBAL_BASE_REG, SDLoc(), PtrVT),
2822:                          Result);
2823:   }
2824: 
2825:   return Result;
2826: }
2827: 
2828: unsigned M68kTargetLowering::getJumpTableEncoding() const {
2829:   return Subtarget.getJumpTableEncoding();
2830: }
2831: 
2832: const MCExpr *M68kTargetLowering::LowerCustomJumpTableEntry(
2833:     const MachineJumpTableInfo *MJTI, const MachineBasicBlock *MBB,
2834:     unsigned uid, MCContext &Ctx) const {
2835:   return MCSymbolRefExpr::create(MBB->getSymbol(), M68k::S_GOTOFF, Ctx);
2836: }
2837: 
2838: SDValue M68kTargetLowering::getPICJumpTableRelocBase(SDValue Table,
2839:                                                      SelectionDAG &DAG) const {
2840:   if (getJumpTableEncoding() == MachineJumpTableInfo::EK_Custom32)
2841:     return DAG.getNode(M68kISD::GLOBAL_BASE_REG, SDLoc(),
2842:                        getPointerTy(DAG.getDataLayout()));
2843: 
2844:   // MachineJumpTableInfo::EK_LabelDifference32 entry
2845:   return Table;
2846: }
2847: 
2848: // NOTE This only used for MachineJumpTableInfo::EK_LabelDifference32 entries
2849: const MCExpr *M68kTargetLowering::getPICJumpTableRelocBaseExpr(
2850:     const MachineFunction *MF, unsigned JTI, MCContext &Ctx) const {
2851:   return MCSymbolRefExpr::create(MF->getJTISymbol(JTI, Ctx), Ctx);
2852: }
2853: 
2854: M68kTargetLowering::ConstraintType
2855: M68kTargetLowering::getConstraintType(StringRef Constraint) const {
2856:   if (Constraint.size() > 0) {
2857:     switch (Constraint[0]) {
2858:     case 'a':
2859:     case 'd':
2860:       return C_RegisterClass;
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::getJumpTableEncoding`, `M68kTargetLowering::getPICJumpTableRelocBase`, `M68kTargetLowering::getConstraintType`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::getJumpTableEncoding`, `M68kTargetLowering::getPICJumpTableRelocBase`, `M68kTargetLowering::getConstraintType` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2861-2915 / 第 2861-2915 行
```cpp
2861:     case 'I':
2862:     case 'J':
2863:     case 'K':
2864:     case 'L':
2865:     case 'M':
2866:     case 'N':
2867:     case 'O':
2868:     case 'P':
2869:       return C_Immediate;
2870:     case 'C':
2871:       if (Constraint.size() == 2)
2872:         switch (Constraint[1]) {
2873:         case '0':
2874:         case 'i':
2875:         case 'j':
2876:           return C_Immediate;
2877:         default:
2878:           break;
2879:         }
2880:       break;
2881:     case 'Q':
2882:     case 'U':
2883:       return C_Memory;
2884:     default:
2885:       break;
2886:     }
2887:   }
2888: 
2889:   return TargetLowering::getConstraintType(Constraint);
2890: }
2891: 
2892: void M68kTargetLowering::LowerAsmOperandForConstraint(SDValue Op,
2893:                                                       StringRef Constraint,
2894:                                                       std::vector<SDValue> &Ops,
2895:                                                       SelectionDAG &DAG) const {
2896:   SDValue Result;
2897: 
2898:   if (Constraint.size() == 1) {
2899:     // Constant constraints
2900:     switch (Constraint[0]) {
2901:     case 'I':
2902:     case 'J':
2903:     case 'K':
2904:     case 'L':
2905:     case 'M':
2906:     case 'N':
2907:     case 'O':
2908:     case 'P': {
2909:       auto *C = dyn_cast<ConstantSDNode>(Op);
2910:       if (!C)
2911:         return;
2912: 
2913:       int64_t Val = C->getSExtValue();
2914:       switch (Constraint[0]) {
2915:       case 'I': // constant integer in the range [1,8]
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::LowerAsmOperandForConstraint`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::LowerAsmOperandForConstraint` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 2916-2970 / 第 2916-2970 行
```cpp
2916:         if (Val > 0 && Val <= 8)
2917:           break;
2918:         return;
2919:       case 'J': // constant signed 16-bit integer
2920:         if (isInt<16>(Val))
2921:           break;
2922:         return;
2923:       case 'K': // constant that is NOT in the range of [-0x80, 0x80)
2924:         if (Val < -0x80 || Val >= 0x80)
2925:           break;
2926:         return;
2927:       case 'L': // constant integer in the range [-8,-1]
2928:         if (Val < 0 && Val >= -8)
2929:           break;
2930:         return;
2931:       case 'M': // constant that is NOT in the range of [-0x100, 0x100]
2932:         if (Val < -0x100 || Val >= 0x100)
2933:           break;
2934:         return;
2935:       case 'N': // constant integer in the range [24,31]
2936:         if (Val >= 24 && Val <= 31)
2937:           break;
2938:         return;
2939:       case 'O': // constant integer 16
2940:         if (Val == 16)
2941:           break;
2942:         return;
2943:       case 'P': // constant integer in the range [8,15]
2944:         if (Val >= 8 && Val <= 15)
2945:           break;
2946:         return;
2947:       default:
2948:         llvm_unreachable("Unhandled constant constraint");
2949:       }
2950: 
2951:       Result = DAG.getSignedTargetConstant(Val, SDLoc(Op), Op.getValueType());
2952:       break;
2953:     }
2954:     default:
2955:       break;
2956:     }
2957:   }
2958: 
2959:   if (Constraint.size() == 2) {
2960:     switch (Constraint[0]) {
2961:     case 'C':
2962:       // Constant constraints start with 'C'
2963:       switch (Constraint[1]) {
2964:       case '0':
2965:       case 'i':
2966:       case 'j': {
2967:         auto *C = dyn_cast<ConstantSDNode>(Op);
2968:         if (!C)
2969:           break;
2970: 
```
- **EN**: A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 2971-3025 / 第 2971-3025 行
```cpp
2971:         int64_t Val = C->getSExtValue();
2972:         switch (Constraint[1]) {
2973:         case '0': // constant integer 0
2974:           if (!Val)
2975:             break;
2976:           return;
2977:         case 'i': // constant integer
2978:           break;
2979:         case 'j': // integer constant that doesn't fit in 16 bits
2980:           if (!isInt<16>(C->getSExtValue()))
2981:             break;
2982:           return;
2983:         default:
2984:           llvm_unreachable("Unhandled constant constraint");
2985:         }
2986: 
2987:         Result = DAG.getSignedTargetConstant(Val, SDLoc(Op), Op.getValueType());
2988:         break;
2989:       }
2990:       default:
2991:         break;
2992:       }
2993:       break;
2994:     default:
2995:       break;
2996:     }
2997:   }
2998: 
2999:   if (Result.getNode()) {
3000:     Ops.push_back(Result);
3001:     return;
3002:   }
3003: 
3004:   TargetLowering::LowerAsmOperandForConstraint(Op, Constraint, Ops, DAG);
3005: }
3006: 
3007: std::pair<unsigned, const TargetRegisterClass *>
3008: M68kTargetLowering::getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
3009:                                                  StringRef Constraint,
3010:                                                  MVT VT) const {
3011:   if (Constraint.size() == 1) {
3012:     switch (Constraint[0]) {
3013:     case 'r':
3014:     case 'd':
3015:       switch (VT.SimpleTy) {
3016:       case MVT::i8:
3017:         return std::make_pair(0U, &M68k::DR8RegClass);
3018:       case MVT::i16:
3019:         return std::make_pair(0U, &M68k::DR16RegClass);
3020:       case MVT::i32:
3021:         return std::make_pair(0U, &M68k::DR32RegClass);
3022:       default:
3023:         break;
3024:       }
3025:       break;
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::getRegForInlineAsmConstraint`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::getRegForInlineAsmConstraint` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3026-3080 / 第 3026-3080 行
```cpp
3026:     case 'a':
3027:       switch (VT.SimpleTy) {
3028:       case MVT::i16:
3029:         return std::make_pair(0U, &M68k::AR16RegClass);
3030:       case MVT::i32:
3031:         return std::make_pair(0U, &M68k::AR32RegClass);
3032:       default:
3033:         break;
3034:       }
3035:       break;
3036:     default:
3037:       break;
3038:     }
3039:   }
3040: 
3041:   return TargetLowering::getRegForInlineAsmConstraint(TRI, Constraint, VT);
3042: }
3043: 
3044: /// Determines whether the callee is required to pop its own arguments.
3045: /// Callee pop is necessary to support tail calls.
3046: bool M68k::isCalleePop(CallingConv::ID CC, bool IsVarArg, bool GuaranteeTCO) {
3047:   return CC == CallingConv::M68k_RTD && !IsVarArg;
3048: }
3049: 
3050: // Return true if it is OK for this CMOV pseudo-opcode to be cascaded
3051: // together with other CMOV pseudo-opcodes into a single basic-block with
3052: // conditional jump around it.
3053: static bool isCMOVPseudo(MachineInstr &MI) {
3054:   switch (MI.getOpcode()) {
3055:   case M68k::CMOV8d:
3056:   case M68k::CMOV16d:
3057:   case M68k::CMOV32r:
3058:     return true;
3059: 
3060:   default:
3061:     return false;
3062:   }
3063: }
3064: 
3065: // The CCR operand of SelectItr might be missing a kill marker
3066: // because there were multiple uses of CCR, and ISel didn't know
3067: // which to mark. Figure out whether SelectItr should have had a
3068: // kill marker, and set it if it should. Returns the correct kill
3069: // marker value.
3070: static bool checkAndUpdateCCRKill(MachineBasicBlock::iterator SelectItr,
3071:                                   MachineBasicBlock *BB,
3072:                                   const TargetRegisterInfo *TRI) {
3073:   // Scan forward through BB for a use/def of CCR.
3074:   MachineBasicBlock::iterator miI(std::next(SelectItr));
3075:   for (MachineBasicBlock::iterator miE = BB->end(); miI != miE; ++miI) {
3076:     const MachineInstr &mi = *miI;
3077:     if (mi.readsRegister(M68k::CCR, /*TRI=*/nullptr))
3078:       return false;
3079:     if (mi.definesRegister(M68k::CCR, /*TRI=*/nullptr))
3080:       break; // Should have kill-flag - update below.
```
- **EN**: The range implements or declares functions including `M68k::isCalleePop`, `isCMOVPseudo`, `checkAndUpdateCCRKill`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68k::isCalleePop`, `isCMOVPseudo`, `checkAndUpdateCCRKill` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3081-3135 / 第 3081-3135 行
```cpp
3081:   }
3082: 
3083:   // If we hit the end of the block, check whether CCR is live into a
3084:   // successor.
3085:   if (miI == BB->end())
3086:     for (const auto *SBB : BB->successors())
3087:       if (SBB->isLiveIn(M68k::CCR))
3088:         return false;
3089: 
3090:   // We found a def, or hit the end of the basic block and CCR wasn't live
3091:   // out. SelectMI should have a kill flag on CCR.
3092:   SelectItr->addRegisterKilled(M68k::CCR, TRI);
3093:   return true;
3094: }
3095: 
3096: MachineBasicBlock *
3097: M68kTargetLowering::EmitLoweredSelect(MachineInstr &MI,
3098:                                       MachineBasicBlock *MBB) const {
3099:   const TargetInstrInfo *TII = Subtarget.getInstrInfo();
3100:   DebugLoc DL = MI.getDebugLoc();
3101: 
3102:   // To "insert" a SELECT_CC instruction, we actually have to insert the
3103:   // diamond control-flow pattern.  The incoming instruction knows the
3104:   // destination vreg to set, the condition code register to branch on, the
3105:   // true/false values to select between, and a branch opcode to use.
3106:   const BasicBlock *BB = MBB->getBasicBlock();
3107:   MachineFunction::iterator It = ++MBB->getIterator();
3108: 
3109:   //  ThisMBB:
3110:   //  ...
3111:   //   TrueVal = ...
3112:   //   cmp ccX, r1, r2
3113:   //   bcc Copy1MBB
3114:   //   fallthrough --> Copy0MBB
3115:   MachineBasicBlock *ThisMBB = MBB;
3116:   MachineFunction *F = MBB->getParent();
3117: 
3118:   // This code lowers all pseudo-CMOV instructions. Generally it lowers these
3119:   // as described above, by inserting a MBB, and then making a PHI at the join
3120:   // point to select the true and false operands of the CMOV in the PHI.
3121:   //
3122:   // The code also handles two different cases of multiple CMOV opcodes
3123:   // in a row.
3124:   //
3125:   // Case 1:
3126:   // In this case, there are multiple CMOVs in a row, all which are based on
3127:   // the same condition setting (or the exact opposite condition setting).
3128:   // In this case we can lower all the CMOVs using a single inserted MBB, and
3129:   // then make a number of PHIs at the join point to model the CMOVs. The only
3130:   // trickiness here, is that in a case like:
3131:   //
3132:   // t2 = CMOV cond1 t1, f1
3133:   // t3 = CMOV cond1 t2, f2
3134:   //
3135:   // when rewriting this into PHIs, we have to perform some renaming on the
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::EmitLoweredSelect`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::EmitLoweredSelect` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 3136-3190 / 第 3136-3190 行
```cpp
3136:   // temps since you cannot have a PHI operand refer to a PHI result earlier
3137:   // in the same block.  The "simple" but wrong lowering would be:
3138:   //
3139:   // t2 = PHI t1(BB1), f1(BB2)
3140:   // t3 = PHI t2(BB1), f2(BB2)
3141:   //
3142:   // but clearly t2 is not defined in BB1, so that is incorrect. The proper
3143:   // renaming is to note that on the path through BB1, t2 is really just a
3144:   // copy of t1, and do that renaming, properly generating:
3145:   //
3146:   // t2 = PHI t1(BB1), f1(BB2)
3147:   // t3 = PHI t1(BB1), f2(BB2)
3148:   //
3149:   // Case 2, we lower cascaded CMOVs such as
3150:   //
3151:   //   (CMOV (CMOV F, T, cc1), T, cc2)
3152:   //
3153:   // to two successives branches.
3154:   MachineInstr *CascadedCMOV = nullptr;
3155:   MachineInstr *LastCMOV = &MI;
3156:   M68k::CondCode CC = M68k::CondCode(MI.getOperand(3).getImm());
3157:   M68k::CondCode OppCC = M68k::GetOppositeBranchCondition(CC);
3158:   MachineBasicBlock::iterator NextMIIt =
3159:       std::next(MachineBasicBlock::iterator(MI));
3160: 
3161:   // Check for case 1, where there are multiple CMOVs with the same condition
3162:   // first.  Of the two cases of multiple CMOV lowerings, case 1 reduces the
3163:   // number of jumps the most.
3164: 
3165:   if (isCMOVPseudo(MI)) {
3166:     // See if we have a string of CMOVS with the same condition.
3167:     while (NextMIIt != MBB->end() && isCMOVPseudo(*NextMIIt) &&
3168:            (NextMIIt->getOperand(3).getImm() == CC ||
3169:             NextMIIt->getOperand(3).getImm() == OppCC)) {
3170:       LastCMOV = &*NextMIIt;
3171:       ++NextMIIt;
3172:     }
3173:   }
3174: 
3175:   // This checks for case 2, but only do this if we didn't already find
3176:   // case 1, as indicated by LastCMOV == MI.
3177:   if (LastCMOV == &MI && NextMIIt != MBB->end() &&
3178:       NextMIIt->getOpcode() == MI.getOpcode() &&
3179:       NextMIIt->getOperand(2).getReg() == MI.getOperand(2).getReg() &&
3180:       NextMIIt->getOperand(1).getReg() == MI.getOperand(0).getReg() &&
3181:       NextMIIt->getOperand(1).isKill()) {
3182:     CascadedCMOV = &*NextMIIt;
3183:   }
3184: 
3185:   MachineBasicBlock *Jcc1MBB = nullptr;
3186: 
3187:   // If we have a cascaded CMOV, we lower it to two successive branches to
3188:   // the same block.  CCR is used by both, so mark it as live in the second.
3189:   if (CascadedCMOV) {
3190:     Jcc1MBB = F->CreateMachineBasicBlock(BB);
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 3191-3245 / 第 3191-3245 行
```cpp
3191:     F->insert(It, Jcc1MBB);
3192:     Jcc1MBB->addLiveIn(M68k::CCR);
3193:   }
3194: 
3195:   MachineBasicBlock *Copy0MBB = F->CreateMachineBasicBlock(BB);
3196:   MachineBasicBlock *SinkMBB = F->CreateMachineBasicBlock(BB);
3197:   F->insert(It, Copy0MBB);
3198:   F->insert(It, SinkMBB);
3199: 
3200:   // Set the call frame size on entry to the new basic blocks.
3201:   unsigned CallFrameSize = TII->getCallFrameSizeAt(MI);
3202:   Copy0MBB->setCallFrameSize(CallFrameSize);
3203:   SinkMBB->setCallFrameSize(CallFrameSize);
3204: 
3205:   // If the CCR register isn't dead in the terminator, then claim that it's
3206:   // live into the sink and copy blocks.
3207:   const TargetRegisterInfo *TRI = Subtarget.getRegisterInfo();
3208: 
3209:   MachineInstr *LastCCRSUser = CascadedCMOV ? CascadedCMOV : LastCMOV;
3210:   if (!LastCCRSUser->killsRegister(M68k::CCR, /*TRI=*/nullptr) &&
3211:       !checkAndUpdateCCRKill(LastCCRSUser, MBB, TRI)) {
3212:     Copy0MBB->addLiveIn(M68k::CCR);
3213:     SinkMBB->addLiveIn(M68k::CCR);
3214:   }
3215: 
3216:   // Transfer the remainder of MBB and its successor edges to SinkMBB.
3217:   SinkMBB->splice(SinkMBB->begin(), MBB,
3218:                   std::next(MachineBasicBlock::iterator(LastCMOV)), MBB->end());
3219:   SinkMBB->transferSuccessorsAndUpdatePHIs(MBB);
3220: 
3221:   // Add the true and fallthrough blocks as its successors.
3222:   if (CascadedCMOV) {
3223:     // The fallthrough block may be Jcc1MBB, if we have a cascaded CMOV.
3224:     MBB->addSuccessor(Jcc1MBB);
3225: 
3226:     // In that case, Jcc1MBB will itself fallthrough the Copy0MBB, and
3227:     // jump to the SinkMBB.
3228:     Jcc1MBB->addSuccessor(Copy0MBB);
3229:     Jcc1MBB->addSuccessor(SinkMBB);
3230:   } else {
3231:     MBB->addSuccessor(Copy0MBB);
3232:   }
3233: 
3234:   // The true block target of the first (or only) branch is always SinkMBB.
3235:   MBB->addSuccessor(SinkMBB);
3236: 
3237:   // Create the conditional branch instruction.
3238:   unsigned Opc = M68k::GetCondBranchFromCond(CC);
3239:   BuildMI(MBB, DL, TII->get(Opc)).addMBB(SinkMBB);
3240: 
3241:   if (CascadedCMOV) {
3242:     unsigned Opc2 = M68k::GetCondBranchFromCond(
3243:         (M68k::CondCode)CascadedCMOV->getOperand(3).getImm());
3244:     BuildMI(Jcc1MBB, DL, TII->get(Opc2)).addMBB(SinkMBB);
3245:   }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3246-3300 / 第 3246-3300 行
```cpp
3246: 
3247:   //  Copy0MBB:
3248:   //   %FalseValue = ...
3249:   //   # fallthrough to SinkMBB
3250:   Copy0MBB->addSuccessor(SinkMBB);
3251: 
3252:   //  SinkMBB:
3253:   //   %Result = phi [ %FalseValue, Copy0MBB ], [ %TrueValue, ThisMBB ]
3254:   //  ...
3255:   MachineBasicBlock::iterator MIItBegin = MachineBasicBlock::iterator(MI);
3256:   MachineBasicBlock::iterator MIItEnd =
3257:       std::next(MachineBasicBlock::iterator(LastCMOV));
3258:   MachineBasicBlock::iterator SinkInsertionPoint = SinkMBB->begin();
3259:   DenseMap<unsigned, std::pair<unsigned, unsigned>> RegRewriteTable;
3260:   MachineInstrBuilder MIB;
3261: 
3262:   // As we are creating the PHIs, we have to be careful if there is more than
3263:   // one.  Later CMOVs may reference the results of earlier CMOVs, but later
3264:   // PHIs have to reference the individual true/false inputs from earlier PHIs.
3265:   // That also means that PHI construction must work forward from earlier to
3266:   // later, and that the code must maintain a mapping from earlier PHI's
3267:   // destination registers, and the registers that went into the PHI.
3268: 
3269:   for (MachineBasicBlock::iterator MIIt = MIItBegin; MIIt != MIItEnd; ++MIIt) {
3270:     Register DestReg = MIIt->getOperand(0).getReg();
3271:     Register Op1Reg = MIIt->getOperand(1).getReg();
3272:     Register Op2Reg = MIIt->getOperand(2).getReg();
3273: 
3274:     // If this CMOV we are generating is the opposite condition from
3275:     // the jump we generated, then we have to swap the operands for the
3276:     // PHI that is going to be generated.
3277:     if (MIIt->getOperand(3).getImm() == OppCC)
3278:       std::swap(Op1Reg, Op2Reg);
3279: 
3280:     if (RegRewriteTable.find(Op1Reg) != RegRewriteTable.end())
3281:       Op1Reg = RegRewriteTable[Op1Reg].first;
3282: 
3283:     if (RegRewriteTable.find(Op2Reg) != RegRewriteTable.end())
3284:       Op2Reg = RegRewriteTable[Op2Reg].second;
3285: 
3286:     MIB =
3287:         BuildMI(*SinkMBB, SinkInsertionPoint, DL, TII->get(M68k::PHI), DestReg)
3288:             .addReg(Op1Reg)
3289:             .addMBB(Copy0MBB)
3290:             .addReg(Op2Reg)
3291:             .addMBB(ThisMBB);
3292: 
3293:     // Add this PHI to the rewrite table.
3294:     RegRewriteTable[DestReg] = std::make_pair(Op1Reg, Op2Reg);
3295:   }
3296: 
3297:   // If we have a cascaded CMOV, the second Jcc provides the same incoming
3298:   // value as the first Jcc (the True operand of the SELECT_CC/CMOV nodes).
3299:   if (CascadedCMOV) {
3300:     MIB.addReg(MI.getOperand(2).getReg()).addMBB(Jcc1MBB);
```
- **EN**: The range implements or declares functions including `BuildMI`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `BuildMI` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 3301-3355 / 第 3301-3355 行
```cpp
3301:     // Copy the PHI result to the register defined by the second CMOV.
3302:     BuildMI(*SinkMBB, std::next(MachineBasicBlock::iterator(MIB.getInstr())),
3303:             DL, TII->get(TargetOpcode::COPY),
3304:             CascadedCMOV->getOperand(0).getReg())
3305:         .addReg(MI.getOperand(0).getReg());
3306:     CascadedCMOV->eraseFromParent();
3307:   }
3308: 
3309:   // Now remove the CMOV(s).
3310:   for (MachineBasicBlock::iterator MIIt = MIItBegin; MIIt != MIItEnd;)
3311:     (MIIt++)->eraseFromParent();
3312: 
3313:   return SinkMBB;
3314: }
3315: 
3316: MachineBasicBlock *
3317: M68kTargetLowering::EmitLoweredSegAlloca(MachineInstr &MI,
3318:                                          MachineBasicBlock *BB) const {
3319:   llvm_unreachable("Cannot lower Segmented Stack Alloca with stack-split on");
3320: }
3321: 
3322: MachineBasicBlock *
3323: M68kTargetLowering::EmitInstrWithCustomInserter(MachineInstr &MI,
3324:                                                 MachineBasicBlock *BB) const {
3325:   switch (MI.getOpcode()) {
3326:   default:
3327:     llvm_unreachable("Unexpected instr type to insert");
3328:   case M68k::CMOV8d:
3329:   case M68k::CMOV16d:
3330:   case M68k::CMOV32r:
3331:     return EmitLoweredSelect(MI, BB);
3332:   case M68k::SALLOCA:
3333:     return EmitLoweredSegAlloca(MI, BB);
3334:   }
3335: }
3336: 
3337: SDValue M68kTargetLowering::LowerVASTART(SDValue Op, SelectionDAG &DAG) const {
3338:   MachineFunction &MF = DAG.getMachineFunction();
3339:   auto PtrVT = getPointerTy(MF.getDataLayout());
3340:   M68kMachineFunctionInfo *FuncInfo = MF.getInfo<M68kMachineFunctionInfo>();
3341: 
3342:   const Value *SV = cast<SrcValueSDNode>(Op.getOperand(2))->getValue();
3343:   SDLoc DL(Op);
3344: 
3345:   // vastart just stores the address of the VarArgsFrameIndex slot into the
3346:   // memory location argument.
3347:   SDValue FR = DAG.getFrameIndex(FuncInfo->getVarArgsFrameIndex(), PtrVT);
3348:   return DAG.getStore(Op.getOperand(0), DL, FR, Op.getOperand(1),
3349:                       MachinePointerInfo(SV));
3350: }
3351: 
3352: SDValue M68kTargetLowering::LowerATOMICFENCE(SDValue Op,
3353:                                              SelectionDAG &DAG) const {
3354:   // Lower to a memory barrier created from inline asm.
3355:   const TargetLowering &TLI = DAG.getTargetLoweringInfo();
```
- **EN**: The range implements or declares functions including `BuildMI`, `M68kTargetLowering::EmitLoweredSegAlloca`, `M68kTargetLowering::EmitInstrWithCustomInserter`, `M68kTargetLowering::LowerVASTART`, `M68kTargetLowering::LowerATOMICFENCE`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `BuildMI`, `M68kTargetLowering::EmitLoweredSegAlloca`, `M68kTargetLowering::EmitInstrWithCustomInserter`, `M68kTargetLowering::LowerVASTART`, `M68kTargetLowering::LowerATOMICFENCE` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 3356-3410 / 第 3356-3410 行
```cpp
3356:   LLVMContext &Ctx = *DAG.getContext();
3357: 
3358:   const unsigned Flags = InlineAsm::Extra_MayLoad | InlineAsm::Extra_MayStore |
3359:                          InlineAsm::Extra_HasSideEffects;
3360:   const SDValue AsmOperands[4] = {
3361:       Op.getOperand(0), // Input chain
3362:       DAG.getTargetExternalSymbol(
3363:           "", TLI.getProgramPointerTy(
3364:                   DAG.getDataLayout())),   // Empty inline asm string
3365:       DAG.getMDNode(MDNode::get(Ctx, {})), // (empty) srcloc
3366:       DAG.getTargetConstant(Flags, SDLoc(Op),
3367:                             TLI.getPointerTy(DAG.getDataLayout())), // Flags
3368:   };
3369: 
3370:   return DAG.getNode(ISD::INLINEASM, SDLoc(Op),
3371:                      DAG.getVTList(MVT::Other, MVT::Glue), AsmOperands);
3372: }
3373: 
3374: // Lower dynamic stack allocation to _alloca call for Cygwin/Mingw targets.
3375: // Calls to _alloca are needed to probe the stack when allocating more than 4k
3376: // bytes in one go. Touching the stack at 4K increments is necessary to ensure
3377: // that the guard pages used by the OS virtual memory manager are allocated in
3378: // correct sequence.
3379: SDValue M68kTargetLowering::LowerDYNAMIC_STACKALLOC(SDValue Op,
3380:                                                     SelectionDAG &DAG) const {
3381:   MachineFunction &MF = DAG.getMachineFunction();
3382:   bool SplitStack = MF.shouldSplitStack();
3383: 
3384:   SDLoc DL(Op);
3385: 
3386:   // Get the inputs.
3387:   SDNode *Node = Op.getNode();
3388:   SDValue Chain = Op.getOperand(0);
3389:   SDValue Size = Op.getOperand(1);
3390:   unsigned Align = Op.getConstantOperandVal(2);
3391:   EVT VT = Node->getValueType(0);
3392: 
3393:   // Chain the dynamic stack allocation so that it doesn't modify the stack
3394:   // pointer when other instructions are using the stack.
3395:   Chain = DAG.getCALLSEQ_START(Chain, 0, 0, DL);
3396: 
3397:   SDValue Result;
3398:   if (SplitStack) {
3399:     auto &MRI = MF.getRegInfo();
3400:     auto SPTy = getPointerTy(DAG.getDataLayout());
3401:     auto *ARClass = getRegClassFor(SPTy);
3402:     Register Vreg = MRI.createVirtualRegister(ARClass);
3403:     Chain = DAG.getCopyToReg(Chain, DL, Vreg, Size);
3404:     Result = DAG.getNode(M68kISD::SEG_ALLOCA, DL, SPTy, Chain,
3405:                          DAG.getRegister(Vreg, SPTy));
3406:   } else {
3407:     auto &TLI = DAG.getTargetLoweringInfo();
3408:     Register SPReg = TLI.getStackPointerRegisterToSaveRestore();
3409:     assert(SPReg && "Target cannot require DYNAMIC_STACKALLOC expansion and"
3410:                     " not tell us which reg is the stack pointer!");
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::LowerDYNAMIC_STACKALLOC`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. The code enforces invariants and documents assumptions with assertions or unreachable markers.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::LowerDYNAMIC_STACKALLOC` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 代码使用断言或不可达标记来强化不变量并记录其假设。

### Lines 3411-3465 / 第 3411-3465 行
```cpp
3411: 
3412:     SDValue SP = DAG.getCopyFromReg(Chain, DL, SPReg, VT);
3413:     Chain = SP.getValue(1);
3414:     const TargetFrameLowering &TFI = *Subtarget.getFrameLowering();
3415:     unsigned StackAlign = TFI.getStackAlignment();
3416:     Result = DAG.getNode(ISD::SUB, DL, VT, SP, Size); // Value
3417:     if (Align > StackAlign)
3418:       Result = DAG.getNode(ISD::AND, DL, VT, Result,
3419:                            DAG.getSignedConstant(-(uint64_t)Align, DL, VT));
3420:     Chain = DAG.getCopyToReg(Chain, DL, SPReg, Result); // Output chain
3421:   }
3422: 
3423:   Chain = DAG.getCALLSEQ_END(Chain, 0, 0, SDValue(), DL);
3424: 
3425:   SDValue Ops[2] = {Result, Chain};
3426:   return DAG.getMergeValues(Ops, DL);
3427: }
3428: 
3429: SDValue M68kTargetLowering::LowerShiftLeftParts(SDValue Op,
3430:                                                 SelectionDAG &DAG) const {
3431:   SDLoc DL(Op);
3432:   SDValue Lo = Op.getOperand(0);
3433:   SDValue Hi = Op.getOperand(1);
3434:   SDValue Shamt = Op.getOperand(2);
3435:   EVT VT = Lo.getValueType();
3436: 
3437:   // if Shamt - register size < 0: // Shamt < register size
3438:   //   Lo = Lo << Shamt
3439:   //   Hi = (Hi << Shamt) | ((Lo >>u 1) >>u (register size - 1 ^ Shamt))
3440:   // else:
3441:   //   Lo = 0
3442:   //   Hi = Lo << (Shamt - register size)
3443: 
3444:   SDValue Zero = DAG.getConstant(0, DL, VT);
3445:   SDValue One = DAG.getConstant(1, DL, VT);
3446:   SDValue MinusRegisterSize = DAG.getSignedConstant(-32, DL, VT);
3447:   SDValue RegisterSizeMinus1 = DAG.getConstant(32 - 1, DL, VT);
3448:   SDValue ShamtMinusRegisterSize =
3449:       DAG.getNode(ISD::ADD, DL, VT, Shamt, MinusRegisterSize);
3450:   SDValue RegisterSizeMinus1Shamt =
3451:       DAG.getNode(ISD::XOR, DL, VT, RegisterSizeMinus1, Shamt);
3452: 
3453:   SDValue LoTrue = DAG.getNode(ISD::SHL, DL, VT, Lo, Shamt);
3454:   SDValue ShiftRight1Lo = DAG.getNode(ISD::SRL, DL, VT, Lo, One);
3455:   SDValue ShiftRightLo =
3456:       DAG.getNode(ISD::SRL, DL, VT, ShiftRight1Lo, RegisterSizeMinus1Shamt);
3457:   SDValue ShiftLeftHi = DAG.getNode(ISD::SHL, DL, VT, Hi, Shamt);
3458:   SDValue HiTrue = DAG.getNode(ISD::OR, DL, VT, ShiftLeftHi, ShiftRightLo);
3459:   SDValue HiFalse = DAG.getNode(ISD::SHL, DL, VT, Lo, ShamtMinusRegisterSize);
3460: 
3461:   SDValue CC =
3462:       DAG.getSetCC(DL, MVT::i8, ShamtMinusRegisterSize, Zero, ISD::SETLT);
3463: 
3464:   Lo = DAG.getNode(ISD::SELECT, DL, VT, CC, LoTrue, Zero);
3465:   Hi = DAG.getNode(ISD::SELECT, DL, VT, CC, HiTrue, HiFalse);
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::LowerShiftLeftParts`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::LowerShiftLeftParts` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3466-3520 / 第 3466-3520 行
```cpp
3466: 
3467:   return DAG.getMergeValues({Lo, Hi}, DL);
3468: }
3469: 
3470: SDValue M68kTargetLowering::LowerShiftRightParts(SDValue Op, SelectionDAG &DAG,
3471:                                                  bool IsSRA) const {
3472:   SDLoc DL(Op);
3473:   SDValue Lo = Op.getOperand(0);
3474:   SDValue Hi = Op.getOperand(1);
3475:   SDValue Shamt = Op.getOperand(2);
3476:   EVT VT = Lo.getValueType();
3477: 
3478:   // SRA expansion:
3479:   //   if Shamt - register size < 0: // Shamt < register size
3480:   //     Lo = (Lo >>u Shamt) | ((Hi << 1) << (register size - 1 ^ Shamt))
3481:   //     Hi = Hi >>s Shamt
3482:   //   else:
3483:   //     Lo = Hi >>s (Shamt - register size);
3484:   //     Hi = Hi >>s (register size - 1)
3485:   //
3486:   // SRL expansion:
3487:   //   if Shamt - register size < 0: // Shamt < register size
3488:   //     Lo = (Lo >>u Shamt) | ((Hi << 1) << (register size - 1 ^ Shamt))
3489:   //     Hi = Hi >>u Shamt
3490:   //   else:
3491:   //     Lo = Hi >>u (Shamt - register size);
3492:   //     Hi = 0;
3493: 
3494:   unsigned ShiftRightOp = IsSRA ? ISD::SRA : ISD::SRL;
3495: 
3496:   SDValue Zero = DAG.getConstant(0, DL, VT);
3497:   SDValue One = DAG.getConstant(1, DL, VT);
3498:   SDValue MinusRegisterSize = DAG.getSignedConstant(-32, DL, VT);
3499:   SDValue RegisterSizeMinus1 = DAG.getConstant(32 - 1, DL, VT);
3500:   SDValue ShamtMinusRegisterSize =
3501:       DAG.getNode(ISD::ADD, DL, VT, Shamt, MinusRegisterSize);
3502:   SDValue RegisterSizeMinus1Shamt =
3503:       DAG.getNode(ISD::XOR, DL, VT, RegisterSizeMinus1, Shamt);
3504: 
3505:   SDValue ShiftRightLo = DAG.getNode(ISD::SRL, DL, VT, Lo, Shamt);
3506:   SDValue ShiftLeftHi1 = DAG.getNode(ISD::SHL, DL, VT, Hi, One);
3507:   SDValue ShiftLeftHi =
3508:       DAG.getNode(ISD::SHL, DL, VT, ShiftLeftHi1, RegisterSizeMinus1Shamt);
3509:   SDValue LoTrue = DAG.getNode(ISD::OR, DL, VT, ShiftRightLo, ShiftLeftHi);
3510:   SDValue HiTrue = DAG.getNode(ShiftRightOp, DL, VT, Hi, Shamt);
3511:   SDValue LoFalse =
3512:       DAG.getNode(ShiftRightOp, DL, VT, Hi, ShamtMinusRegisterSize);
3513:   SDValue HiFalse =
3514:       IsSRA ? DAG.getNode(ISD::SRA, DL, VT, Hi, RegisterSizeMinus1) : Zero;
3515: 
3516:   SDValue CC =
3517:       DAG.getSetCC(DL, MVT::i8, ShamtMinusRegisterSize, Zero, ISD::SETLT);
3518: 
3519:   Lo = DAG.getNode(ISD::SELECT, DL, VT, CC, LoTrue, LoFalse);
3520:   Hi = DAG.getNode(ISD::SELECT, DL, VT, CC, HiTrue, HiFalse);
```
- **EN**: The range implements or declares functions including `M68kTargetLowering::LowerShiftRightParts`. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `M68kTargetLowering::LowerShiftRightParts` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3521-3575 / 第 3521-3575 行
```cpp
3521: 
3522:   return DAG.getMergeValues({Lo, Hi}, DL);
3523: }
3524: 
3525: //===----------------------------------------------------------------------===//
3526: // DAG Combine
3527: //===----------------------------------------------------------------------===//
3528: 
3529: static SDValue getSETCC(M68k::CondCode Cond, SDValue CCR, const SDLoc &dl,
3530:                         SelectionDAG &DAG) {
3531:   return DAG.getNode(M68kISD::SETCC, dl, MVT::i8,
3532:                      DAG.getConstant(Cond, dl, MVT::i8), CCR);
3533: }
3534: // When legalizing carry, we create carries via add X, -1
3535: // If that comes from an actual carry, via setcc, we use the
3536: // carry directly.
3537: static SDValue combineCarryThroughADD(SDValue CCR) {
3538:   if (CCR.getOpcode() == M68kISD::ADD) {
3539:     if (isAllOnesConstant(CCR.getOperand(1))) {
3540:       SDValue Carry = CCR.getOperand(0);
3541:       while (Carry.getOpcode() == ISD::TRUNCATE ||
3542:              Carry.getOpcode() == ISD::ZERO_EXTEND ||
3543:              Carry.getOpcode() == ISD::SIGN_EXTEND ||
3544:              Carry.getOpcode() == ISD::ANY_EXTEND ||
3545:              (Carry.getOpcode() == ISD::AND &&
3546:               isOneConstant(Carry.getOperand(1))))
3547:         Carry = Carry.getOperand(0);
3548:       if (Carry.getOpcode() == M68kISD::SETCC ||
3549:           Carry.getOpcode() == M68kISD::SETCC_CARRY) {
3550:         if (Carry.getConstantOperandVal(0) == M68k::COND_CS)
3551:           return Carry.getOperand(1);
3552:       }
3553:     }
3554:   }
3555: 
3556:   return SDValue();
3557: }
3558: 
3559: /// Optimize a CCR definition used according to the condition code \p CC into
3560: /// a simpler CCR value, potentially returning a new \p CC and replacing uses
3561: /// of chain values.
3562: static SDValue combineSetCCCCR(SDValue CCR, M68k::CondCode &CC,
3563:                                SelectionDAG &DAG,
3564:                                const M68kSubtarget &Subtarget) {
3565:   if (CC == M68k::COND_CS)
3566:     if (SDValue Flags = combineCarryThroughADD(CCR))
3567:       return Flags;
3568: 
3569:   return SDValue();
3570: }
3571: 
3572: // Optimize  RES = M68kISD::SETCC CONDCODE, CCR_INPUT
3573: static SDValue combineM68kSetCC(SDNode *N, SelectionDAG &DAG,
3574:                                 const M68kSubtarget &Subtarget) {
3575:   SDLoc DL(N);
```
- **EN**: The range implements or declares functions including `getSETCC`, `combineCarryThroughADD`, `combineSetCCCCR`, `combineM68kSetCC`. Conditional branches guard special cases, feature checks, or fast paths in the target logic. Iteration is used to walk operands, records, or instruction-related collections.
- **CN**: 这一段实现或声明了 `getSETCC`, `combineCarryThroughADD`, `combineSetCCCCR`, `combineM68kSetCC` 等函数。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。 这里使用迭代来遍历操作数、记录或与指令相关的集合。

### Lines 3576-3630 / 第 3576-3630 行
```cpp
3576:   M68k::CondCode CC = M68k::CondCode(N->getConstantOperandVal(0));
3577:   SDValue CCR = N->getOperand(1);
3578: 
3579:   // Try to simplify the CCR and condition code operands.
3580:   if (SDValue Flags = combineSetCCCCR(CCR, CC, DAG, Subtarget))
3581:     return getSETCC(CC, Flags, DL, DAG);
3582: 
3583:   return SDValue();
3584: }
3585: static SDValue combineM68kBrCond(SDNode *N, SelectionDAG &DAG,
3586:                                  const M68kSubtarget &Subtarget) {
3587:   SDLoc DL(N);
3588:   M68k::CondCode CC = M68k::CondCode(N->getConstantOperandVal(2));
3589:   SDValue CCR = N->getOperand(3);
3590: 
3591:   // Try to simplify the CCR and condition code operands.
3592:   // Make sure to not keep references to operands, as combineSetCCCCR can
3593:   // RAUW them under us.
3594:   if (SDValue Flags = combineSetCCCCR(CCR, CC, DAG, Subtarget)) {
3595:     SDValue Cond = DAG.getConstant(CC, DL, MVT::i8);
3596:     return DAG.getNode(M68kISD::BRCOND, DL, N->getVTList(), N->getOperand(0),
3597:                        N->getOperand(1), Cond, Flags);
3598:   }
3599: 
3600:   return SDValue();
3601: }
3602: 
3603: static SDValue combineSUBX(SDNode *N, SelectionDAG &DAG) {
3604:   if (SDValue Flags = combineCarryThroughADD(N->getOperand(2))) {
3605:     MVT VT = N->getSimpleValueType(0);
3606:     SDVTList VTs = DAG.getVTList(VT, MVT::i32);
3607:     return DAG.getNode(M68kISD::SUBX, SDLoc(N), VTs, N->getOperand(0),
3608:                        N->getOperand(1), Flags);
3609:   }
3610: 
3611:   return SDValue();
3612: }
3613: 
3614: // Optimize RES, CCR = M68kISD::ADDX LHS, RHS, CCR
3615: static SDValue combineADDX(SDNode *N, SelectionDAG &DAG,
3616:                            TargetLowering::DAGCombinerInfo &DCI) {
3617:   if (SDValue Flags = combineCarryThroughADD(N->getOperand(2))) {
3618:     MVT VT = N->getSimpleValueType(0);
3619:     SDVTList VTs = DAG.getVTList(VT, MVT::i32);
3620:     return DAG.getNode(M68kISD::ADDX, SDLoc(N), VTs, N->getOperand(0),
3621:                        N->getOperand(1), Flags);
3622:   }
3623: 
3624:   return SDValue();
3625: }
3626: 
3627: SDValue M68kTargetLowering::PerformDAGCombine(SDNode *N,
3628:                                               DAGCombinerInfo &DCI) const {
3629:   SelectionDAG &DAG = DCI.DAG;
3630:   switch (N->getOpcode()) {
```
- **EN**: The range implements or declares functions including `combineM68kBrCond`, `combineSUBX`, `combineADDX`, `M68kTargetLowering::PerformDAGCombine`. A switch-based dispatch appears here, selecting behavior from opcode, mode, or record categories. Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 这一段实现或声明了 `combineM68kBrCond`, `combineSUBX`, `combineADDX`, `M68kTargetLowering::PerformDAGCombine` 等函数。 这里出现了基于 switch 的分派逻辑，用于按操作码、模式或记录类别选择行为。 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

### Lines 3631-3650 / 第 3631-3650 行
```cpp
3631:   case M68kISD::SUBX:
3632:     return combineSUBX(N, DAG);
3633:   case M68kISD::ADDX:
3634:     return combineADDX(N, DAG, DCI);
3635:   case M68kISD::SETCC:
3636:     return combineM68kSetCC(N, DAG, Subtarget);
3637:   case M68kISD::BRCOND:
3638:     return combineM68kBrCond(N, DAG, Subtarget);
3639:   }
3640: 
3641:   return SDValue();
3642: }
3643: 
3644: CCAssignFn *M68kTargetLowering::getCCAssignFn(CallingConv::ID CC, bool Return,
3645:                                               bool IsVarArg) const {
3646:   if (Return)
3647:     return RetCC_M68k_C;
3648:   else
3649:     return CC_M68k_C;
3650: }
```
- **EN**: Conditional branches guard special cases, feature checks, or fast paths in the target logic.
- **CN**: 条件分支用于处理特殊情况、特性检查或目标逻辑中的快速路径。

## Key Concepts / 关键概念
- **SelectionDAG lowering**: Bridges LLVM IR and target-specific DAG nodes/instructions. / 连接 LLVM IR 与目标相关 DAG 节点/指令。
- **GlobalISel**: Uses legalization, register banks, and instruction selection after generic IR lowering. / 在通用 IR 降低后使用合法化、寄存器银行和指令选择。
- **MachineInstr**: Represents target-aware machine instructions during late code generation. / 表示代码生成后期的目标相关机器指令。
- **MachineFunction**: Carries per-function machine-level state and basic blocks. / 保存每个函数的机器级状态和基本块。
- **Calling convention**: Defines how arguments, returns, and preserved registers are assigned. / 定义参数、返回值和保留寄存器的分配方式。
- **Registers**: Describes physical registers, classes, or allocation-facing metadata. / 描述物理寄存器、寄存器类或面向分配器的元数据。

## Dependencies / 依赖关系
- `M68kISelLowering.h`
- `M68kCallingConv.h`
- `M68kMachineFunction.h`
- `M68kSelectionDAGInfo.h`
- `M68kSubtarget.h`
- `M68kTargetMachine.h`
- `M68kTargetObjectFile.h`
- `MCTargetDesc/M68kMCAsmInfo.h`
- `llvm/ADT/Statistic.h`
- `llvm/CodeGen/CallingConvLower.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineJumpTableInfo.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/CodeGen/SelectionDAG.h`
- `...` (10 more include dependencies omitted for brevity / 其余 10 个 include 依赖已省略)
