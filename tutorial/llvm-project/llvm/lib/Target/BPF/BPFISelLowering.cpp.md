# BPFISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFISelLowering.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file defines the interfaces that BPF uses to lower LLVM code into a selection DAG.
- 目的（中文）: 实现 SelectionDAG 降级钩子，将通用 LLVM IR 操作映射到目标专用 DAG 节点与调用约定。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===-- BPFISelLowering.cpp - BPF DAG Lowering Implementation  ------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This file defines the interfaces that BPF uses to lower LLVM code into a
  10: // selection DAG.
  11: //
  12: //===----------------------------------------------------------------------===//
  13: 
  14: #include "BPFISelLowering.h"
  15: #include "BPF.h"
  16: #include "BPFSubtarget.h"
  17: #include "llvm/CodeGen/CallingConvLower.h"
  18: #include "llvm/CodeGen/MachineFrameInfo.h"
  19: #include "llvm/CodeGen/MachineFunction.h"
  20: #include "llvm/CodeGen/MachineInstrBuilder.h"
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 21-40

```cpp
  21: #include "llvm/CodeGen/MachineJumpTableInfo.h"
  22: #include "llvm/CodeGen/MachineRegisterInfo.h"
  23: #include "llvm/CodeGen/TargetLoweringObjectFileImpl.h"
  24: #include "llvm/CodeGen/ValueTypes.h"
  25: #include "llvm/IR/DIBuilder.h"
  26: #include "llvm/IR/DiagnosticInfo.h"
  27: #include "llvm/IR/DiagnosticPrinter.h"
  28: #include "llvm/IR/Module.h"
  29: #include "llvm/Support/Debug.h"
  30: #include "llvm/Support/ErrorHandling.h"
  31: #include "llvm/Support/MathExtras.h"
  32: #include "llvm/Support/raw_ostream.h"
  33: 
  34: using namespace llvm;
  35: 
  36: #define DEBUG_TYPE "bpf-lower"
  37: 
  38: static cl::opt<bool> BPFExpandMemcpyInOrder("bpf-expand-memcpy-in-order",
  39:   cl::Hidden, cl::init(false),
  40:   cl::desc("Expand memcpy into load/store pairs in order"));
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 41-60

```cpp
  41: 
  42: static cl::opt<unsigned> BPFMinimumJumpTableEntries(
  43:     "bpf-min-jump-table-entries", cl::init(13), cl::Hidden,
  44:     cl::desc("Set minimum number of entries to use a jump table on BPF"));
  45: 
  46: static void fail(const SDLoc &DL, SelectionDAG &DAG, const Twine &Msg,
  47:                  SDValue Val = {}) {
  48:   std::string Str;
  49:   if (Val) {
  50:     raw_string_ostream OS(Str);
  51:     Val->print(OS);
  52:     OS << ' ';
  53:   }
  54:   MachineFunction &MF = DAG.getMachineFunction();
  55:   DAG.getContext()->diagnose(DiagnosticInfoUnsupported(
  56:       MF.getFunction(), Twine(Str).concat(Msg), DL.getDebugLoc()));
  57: }
  58: 
  59: BPFTargetLowering::BPFTargetLowering(const TargetMachine &TM,
  60:                                      const BPFSubtarget &STI)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 61-80

```cpp
  61:     : TargetLowering(TM, STI) {
  62: 
  63:   // Set up the register classes.
  64:   addRegisterClass(MVT::i64, &BPF::GPRRegClass);
  65:   if (STI.getHasAlu32())
  66:     addRegisterClass(MVT::i32, &BPF::GPR32RegClass);
  67: 
  68:   // Compute derived properties from the register classes
  69:   computeRegisterProperties(STI.getRegisterInfo());
  70: 
  71:   setStackPointerRegisterToSaveRestore(BPF::R11);
  72: 
  73:   setOperationAction(ISD::BR_CC, MVT::i64, Custom);
  74:   setOperationAction(ISD::BR_JT, MVT::Other, Expand);
  75:   setOperationAction(ISD::BRCOND, MVT::Other, Expand);
  76: 
  77:   if (!STI.hasGotox())
  78:     setOperationAction(ISD::BRIND, MVT::Other, Expand);
  79: 
  80:   setOperationAction(ISD::TRAP, MVT::Other, Custom);
```

- EN: Function bodies or method definitions such as TargetLowering contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: TargetLowering 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 81-100

```cpp
  81: 
  82:   setOperationAction({ISD::GlobalAddress, ISD::ConstantPool}, MVT::i64, Custom);
  83:   if (STI.hasGotox())
  84:     setOperationAction({ISD::JumpTable, ISD::BlockAddress}, MVT::i64, Custom);
  85: 
  86:   setOperationAction(ISD::DYNAMIC_STACKALLOC, MVT::i64, Custom);
  87:   setOperationAction(ISD::STACKSAVE, MVT::Other, Expand);
  88:   setOperationAction(ISD::STACKRESTORE, MVT::Other, Expand);
  89: 
  90:   // Set unsupported atomic operations as Custom so
  91:   // we can emit better error messages than fatal error
  92:   // from selectiondag.
  93:   for (auto VT : {MVT::i8, MVT::i16, MVT::i32}) {
  94:     if (VT == MVT::i32) {
  95:       if (STI.getHasAlu32())
  96:         continue;
  97:     } else {
  98:       setOperationAction(ISD::ATOMIC_LOAD_ADD, VT, Custom);
  99:     }
 100: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 101-120

```cpp
 101:     setOperationAction(ISD::ATOMIC_LOAD_AND, VT, Custom);
 102:     setOperationAction(ISD::ATOMIC_LOAD_OR, VT, Custom);
 103:     setOperationAction(ISD::ATOMIC_LOAD_XOR, VT, Custom);
 104:     setOperationAction(ISD::ATOMIC_SWAP, VT, Custom);
 105:     setOperationAction(ISD::ATOMIC_CMP_SWAP_WITH_SUCCESS, VT, Custom);
 106:   }
 107: 
 108:   for (auto VT : {MVT::i32, MVT::i64}) {
 109:     setOperationAction(ISD::ATOMIC_LOAD, VT, Custom);
 110:     setOperationAction(ISD::ATOMIC_STORE, VT, Custom);
 111:   }
 112: 
 113:   for (auto VT : { MVT::i32, MVT::i64 }) {
 114:     if (VT == MVT::i32 && !STI.getHasAlu32())
 115:       continue;
 116: 
 117:     setOperationAction(ISD::SDIVREM, VT, Expand);
 118:     setOperationAction(ISD::UDIVREM, VT, Expand);
 119:     if (!STI.hasSdivSmod()) {
 120:       setOperationAction(ISD::SDIV, VT, Custom);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 121-140

```cpp
 121:       setOperationAction(ISD::SREM, VT, Custom);
 122:     }
 123:     setOperationAction(ISD::MULHU, VT, Expand);
 124:     setOperationAction(ISD::MULHS, VT, Expand);
 125:     setOperationAction(ISD::UMUL_LOHI, VT, Expand);
 126:     setOperationAction(ISD::SMUL_LOHI, VT, Expand);
 127:     setOperationAction(ISD::ROTR, VT, Expand);
 128:     setOperationAction(ISD::ROTL, VT, Expand);
 129:     setOperationAction(ISD::SHL_PARTS, VT, Custom);
 130:     setOperationAction(ISD::SRL_PARTS, VT, Custom);
 131:     setOperationAction(ISD::SRA_PARTS, VT, Custom);
 132:     setOperationAction(ISD::CTPOP, VT, Expand);
 133:     setOperationAction(ISD::CTTZ, VT, Expand);
 134:     setOperationAction(ISD::CTLZ, VT, Expand);
 135:     setOperationAction(ISD::CTTZ_ZERO_POISON, VT, Expand);
 136:     setOperationAction(ISD::CTLZ_ZERO_POISON, VT, Expand);
 137: 
 138:     setOperationAction(ISD::SETCC, VT, Expand);
 139:     setOperationAction(ISD::SELECT, VT, Expand);
 140:     setOperationAction(ISD::SELECT_CC, VT, Custom);
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 141-160

```cpp
 141:   }
 142: 
 143:   if (STI.getHasAlu32()) {
 144:     setOperationAction(ISD::BSWAP, MVT::i32, Promote);
 145:     setOperationAction(ISD::BR_CC, MVT::i32,
 146:                        STI.getHasJmp32() ? Custom : Promote);
 147:   }
 148: 
 149:   setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i1, Expand);
 150:   if (!STI.hasMovsx()) {
 151:     setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i8, Expand);
 152:     setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i16, Expand);
 153:     setOperationAction(ISD::SIGN_EXTEND_INREG, MVT::i32, Expand);
 154:   }
 155: 
 156:   // Extended load operations for i1 types must be promoted
 157:   for (MVT VT : MVT::integer_valuetypes()) {
 158:     setLoadExtAction(ISD::EXTLOAD, VT, MVT::i1, Promote);
 159:     setLoadExtAction(ISD::ZEXTLOAD, VT, MVT::i1, Promote);
 160:     setLoadExtAction(ISD::SEXTLOAD, VT, MVT::i1, Promote);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 161-180

```cpp
 161: 
 162:     if (!STI.hasLdsx()) {
 163:       setLoadExtAction(ISD::SEXTLOAD, VT, MVT::i8, Expand);
 164:       setLoadExtAction(ISD::SEXTLOAD, VT, MVT::i16, Expand);
 165:       setLoadExtAction(ISD::SEXTLOAD, VT, MVT::i32, Expand);
 166:     }
 167:   }
 168: 
 169:   setBooleanContents(ZeroOrOneBooleanContent);
 170:   setMaxAtomicSizeInBitsSupported(64);
 171:   setMinimumJumpTableEntries(BPFMinimumJumpTableEntries);
 172: 
 173:   // Function alignments
 174:   setMinFunctionAlignment(Align(8));
 175:   setPrefFunctionAlignment(Align(8));
 176: 
 177:   if (BPFExpandMemcpyInOrder) {
 178:     // LLVM generic code will try to expand memcpy into load/store pairs at this
 179:     // stage which is before quite a few IR optimization passes, therefore the
 180:     // loads and stores could potentially be moved apart from each other which
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 181-200

```cpp
 181:     // will cause trouble to memcpy pattern matcher inside kernel eBPF JIT
 182:     // compilers.
 183:     //
 184:     // When -bpf-expand-memcpy-in-order specified, we want to defer the expand
 185:     // of memcpy to later stage in IR optimization pipeline so those load/store
 186:     // pairs won't be touched and could be kept in order. Hence, we set
 187:     // MaxStoresPerMem* to zero to disable the generic getMemcpyLoadsAndStores
 188:     // code path, and ask LLVM to use target expander EmitTargetCodeForMemcpy.
 189:     MaxStoresPerMemset = MaxStoresPerMemsetOptSize = 0;
 190:     MaxStoresPerMemcpy = MaxStoresPerMemcpyOptSize = 0;
 191:     MaxStoresPerMemmove = MaxStoresPerMemmoveOptSize = 0;
 192:     MaxLoadsPerMemcmp = 0;
 193:   } else {
 194:     // inline memcpy() for kernel to see explicit copy
 195:     unsigned CommonMaxStores =
 196:       STI.getSelectionDAGInfo()->getCommonMaxStoresPerMemFunc();
 197: 
 198:     MaxStoresPerMemset = MaxStoresPerMemsetOptSize = CommonMaxStores;
 199:     MaxStoresPerMemcpy = MaxStoresPerMemcpyOptSize = CommonMaxStores;
 200:     MaxStoresPerMemmove = MaxStoresPerMemmoveOptSize = CommonMaxStores;
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 201-220

```cpp
 201:     MaxLoadsPerMemcmp = MaxLoadsPerMemcmpOptSize = CommonMaxStores;
 202:   }
 203: 
 204:   // CPU/Feature control
 205:   HasAlu32 = STI.getHasAlu32();
 206:   HasJmp32 = STI.getHasJmp32();
 207:   HasJmpExt = STI.getHasJmpExt();
 208:   HasMovsx = STI.hasMovsx();
 209: 
 210:   AllowsMisalignedMemAccess = STI.getAllowsMisalignedMemAccess();
 211: }
 212: 
 213: bool BPFTargetLowering::allowsMisalignedMemoryAccesses(EVT VT, unsigned, Align,
 214:                                                        MachineMemOperand::Flags,
 215:                                                        unsigned *Fast) const {
 216:   // allows-misaligned-mem-access is disabled
 217:   if (!AllowsMisalignedMemAccess)
 218:     return false;
 219: 
 220:   // only allow misalignment for simple value types
```

- EN: Function bodies or method definitions such as allowsMisalignedMemoryAccesses contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: allowsMisalignedMemoryAccesses 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 221-240

```cpp
 221:   if (!VT.isSimple())
 222:     return false;
 223: 
 224:   // always assume fast mode when misalignment is allowed
 225:   if (Fast)
 226:     *Fast = true;
 227: 
 228:   return true;
 229: }
 230: 
 231: bool BPFTargetLowering::isOffsetFoldingLegal(const GlobalAddressSDNode *GA) const {
 232:   return false;
 233: }
 234: 
 235: bool BPFTargetLowering::isTruncateFree(Type *Ty1, Type *Ty2) const {
 236:   if (!Ty1->isIntegerTy() || !Ty2->isIntegerTy())
 237:     return false;
 238:   unsigned NumBits1 = Ty1->getPrimitiveSizeInBits();
 239:   unsigned NumBits2 = Ty2->getPrimitiveSizeInBits();
 240:   return NumBits1 > NumBits2;
```

- EN: Function bodies or method definitions such as isOffsetFoldingLegal, isTruncateFree contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: isOffsetFoldingLegal, isTruncateFree 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 241-260

```cpp
 241: }
 242: 
 243: bool BPFTargetLowering::isTruncateFree(EVT VT1, EVT VT2) const {
 244:   if (!VT1.isInteger() || !VT2.isInteger())
 245:     return false;
 246:   unsigned NumBits1 = VT1.getSizeInBits();
 247:   unsigned NumBits2 = VT2.getSizeInBits();
 248:   return NumBits1 > NumBits2;
 249: }
 250: 
 251: bool BPFTargetLowering::isZExtFree(Type *Ty1, Type *Ty2) const {
 252:   if (!getHasAlu32() || !Ty1->isIntegerTy() || !Ty2->isIntegerTy())
 253:     return false;
 254:   unsigned NumBits1 = Ty1->getPrimitiveSizeInBits();
 255:   unsigned NumBits2 = Ty2->getPrimitiveSizeInBits();
 256:   return NumBits1 == 32 && NumBits2 == 64;
 257: }
 258: 
 259: bool BPFTargetLowering::isZExtFree(EVT VT1, EVT VT2) const {
 260:   if (!getHasAlu32() || !VT1.isInteger() || !VT2.isInteger())
```

- EN: Function bodies or method definitions such as isTruncateFree, isZExtFree contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: isTruncateFree, isZExtFree 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 261-280

```cpp
 261:     return false;
 262:   unsigned NumBits1 = VT1.getSizeInBits();
 263:   unsigned NumBits2 = VT2.getSizeInBits();
 264:   return NumBits1 == 32 && NumBits2 == 64;
 265: }
 266: 
 267: bool BPFTargetLowering::isZExtFree(SDValue Val, EVT VT2) const {
 268:   EVT VT1 = Val.getValueType();
 269:   if (Val.getOpcode() == ISD::LOAD && VT1.isSimple() && VT2.isSimple()) {
 270:     MVT MT1 = VT1.getSimpleVT().SimpleTy;
 271:     MVT MT2 = VT2.getSimpleVT().SimpleTy;
 272:     if ((MT1 == MVT::i8 || MT1 == MVT::i16 || MT1 == MVT::i32) &&
 273:         (MT2 == MVT::i32 || MT2 == MVT::i64))
 274:       return true;
 275:   }
 276:   return TargetLoweringBase::isZExtFree(Val, VT2);
 277: }
 278: 
 279: unsigned BPFTargetLowering::getJumpTableEncoding() const {
 280:   return MachineJumpTableInfo::EK_BlockAddress;
```

- EN: Function bodies or method definitions such as isZExtFree, getJumpTableEncoding contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: isZExtFree, getJumpTableEncoding 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 281-300

```cpp
 281: }
 282: 
 283: BPFTargetLowering::ConstraintType
 284: BPFTargetLowering::getConstraintType(StringRef Constraint) const {
 285:   if (Constraint.size() == 1) {
 286:     switch (Constraint[0]) {
 287:     default:
 288:       break;
 289:     case 'w':
 290:       return C_RegisterClass;
 291:     }
 292:   }
 293: 
 294:   return TargetLowering::getConstraintType(Constraint);
 295: }
 296: 
 297: std::pair<unsigned, const TargetRegisterClass *>
 298: BPFTargetLowering::getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
 299:                                                 StringRef Constraint,
 300:                                                 MVT VT) const {
```

- EN: Function bodies or method definitions such as getConstraintType contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getConstraintType 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 301-320

```cpp
 301:   if (Constraint.size() == 1) {
 302:     // GCC Constraint Letters
 303:     switch (Constraint[0]) {
 304:     case 'r': // GENERAL_REGS
 305:       return std::make_pair(0U, &BPF::GPRRegClass);
 306:     case 'w':
 307:       if (HasAlu32)
 308:         return std::make_pair(0U, &BPF::GPR32RegClass);
 309:       break;
 310:     default:
 311:       break;
 312:     }
 313:   }
 314: 
 315:   return TargetLowering::getRegForInlineAsmConstraint(TRI, Constraint, VT);
 316: }
 317: 
 318: void BPFTargetLowering::ReplaceNodeResults(
 319:   SDNode *N, SmallVectorImpl<SDValue> &Results, SelectionDAG &DAG) const {
 320:   const char *Msg;
```

- EN: Function bodies or method definitions such as ReplaceNodeResults contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: ReplaceNodeResults 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 321-340

```cpp
 321:   uint32_t Opcode = N->getOpcode();
 322:   switch (Opcode) {
 323:   default:
 324:     report_fatal_error("unhandled custom legalization: " + Twine(Opcode));
 325:   case ISD::ATOMIC_LOAD_ADD:
 326:   case ISD::ATOMIC_LOAD_AND:
 327:   case ISD::ATOMIC_LOAD_OR:
 328:   case ISD::ATOMIC_LOAD_XOR:
 329:   case ISD::ATOMIC_SWAP:
 330:   case ISD::ATOMIC_CMP_SWAP_WITH_SUCCESS:
 331:     if (HasAlu32 || Opcode == ISD::ATOMIC_LOAD_ADD)
 332:       Msg = "unsupported atomic operation, please use 32/64 bit version";
 333:     else
 334:       Msg = "unsupported atomic operation, please use 64 bit version";
 335:     break;
 336:   case ISD::ATOMIC_LOAD:
 337:   case ISD::ATOMIC_STORE:
 338:     return;
 339:   }
 340: 
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 341-360

```cpp
 341:   SDLoc DL(N);
 342:   // We'll still produce a fatal error downstream, but this diagnostic is more
 343:   // user-friendly.
 344:   fail(DL, DAG, Msg);
 345: }
 346: 
 347: SDValue BPFTargetLowering::LowerOperation(SDValue Op, SelectionDAG &DAG) const {
 348:   switch (Op.getOpcode()) {
 349:   default:
 350:     report_fatal_error("unimplemented opcode: " + Twine(Op.getOpcode()));
 351:   case ISD::BR_CC:
 352:     return LowerBR_CC(Op, DAG);
 353:   case ISD::JumpTable:
 354:     return LowerJumpTable(Op, DAG);
 355:   case ISD::GlobalAddress:
 356:     return LowerGlobalAddress(Op, DAG);
 357:   case ISD::ConstantPool:
 358:     return LowerConstantPool(Op, DAG);
 359:   case ISD::BlockAddress:
 360:     return LowerBlockAddress(Op, DAG);
```

- EN: Function bodies or method definitions such as LowerOperation contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerOperation 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 361-380

```cpp
 361:   case ISD::SELECT_CC:
 362:     return LowerSELECT_CC(Op, DAG);
 363:   case ISD::SDIV:
 364:   case ISD::SREM:
 365:     return LowerSDIVSREM(Op, DAG);
 366:   case ISD::SHL_PARTS:
 367:   case ISD::SRL_PARTS:
 368:   case ISD::SRA_PARTS:
 369:     return LowerShiftParts(Op, DAG);
 370:   case ISD::DYNAMIC_STACKALLOC:
 371:     return LowerDYNAMIC_STACKALLOC(Op, DAG);
 372:   case ISD::ATOMIC_LOAD:
 373:   case ISD::ATOMIC_STORE:
 374:     return LowerATOMIC_LOAD_STORE(Op, DAG);
 375:   case ISD::TRAP:
 376:     return LowerTRAP(Op, DAG);
 377:   }
 378: }
 379: 
 380: // Calling Convention Implementation
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 381-400

```cpp
 381: #include "BPFGenCallingConv.inc"
 382: 
 383: // Apply AssertSext/AssertZext and truncate based on VA's LocInfo.
 384: static SDValue convertLocValType(SelectionDAG &DAG, const SDLoc &DL,
 385:                                  const CCValAssign &VA, EVT RegVT,
 386:                                  SDValue ArgValue) {
 387:   if (VA.getLocInfo() == CCValAssign::SExt)
 388:     ArgValue = DAG.getNode(ISD::AssertSext, DL, RegVT, ArgValue,
 389:                            DAG.getValueType(VA.getValVT()));
 390:   else if (VA.getLocInfo() == CCValAssign::ZExt)
 391:     ArgValue = DAG.getNode(ISD::AssertZext, DL, RegVT, ArgValue,
 392:                            DAG.getValueType(VA.getValVT()));
 393:   if (VA.getLocInfo() != CCValAssign::Full)
 394:     ArgValue = DAG.getNode(ISD::TRUNCATE, DL, VA.getValVT(), ArgValue);
 395:   return ArgValue;
 396: }
 397: 
 398: SDValue BPFTargetLowering::LowerFormalArguments(
 399:     SDValue Chain, CallingConv::ID CallConv, bool IsVarArg,
 400:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &DL,
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. Function bodies or method definitions such as convertLocValType contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 convertLocValType 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 401-420

```cpp
 401:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
 402:   switch (CallConv) {
 403:   default:
 404:     report_fatal_error("unimplemented calling convention: " + Twine(CallConv));
 405:   case CallingConv::C:
 406:   case CallingConv::Fast:
 407:     break;
 408:   }
 409: 
 410:   MachineFunction &MF = DAG.getMachineFunction();
 411:   MachineRegisterInfo &RegInfo = MF.getRegInfo();
 412: 
 413:   // Assign locations to all of the incoming arguments.
 414:   SmallVector<CCValAssign, 16> ArgLocs;
 415:   CCState CCInfo(CallConv, IsVarArg, MF, ArgLocs, *DAG.getContext());
 416:   CCInfo.AnalyzeFormalArguments(Ins, getHasAlu32() ? CC_BPF32 : CC_BPF64);
 417: 
 418:   for (size_t I = 0; I < ArgLocs.size(); ++I) {
 419:     auto &VA = ArgLocs[I];
 420:     EVT RegVT = VA.getLocVT();
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 421-440

```cpp
 421: 
 422:     if (VA.isRegLoc()) {
 423:       // Arguments passed in registers
 424:       MVT::SimpleValueType SimpleTy = RegVT.getSimpleVT().SimpleTy;
 425:       switch (SimpleTy) {
 426:       default: {
 427:         std::string Str;
 428:         {
 429:           raw_string_ostream OS(Str);
 430:           RegVT.print(OS);
 431:         }
 432:         report_fatal_error("unhandled argument type: " + Twine(Str));
 433:       }
 434:       case MVT::i32:
 435:       case MVT::i64:
 436:         Register VReg = RegInfo.createVirtualRegister(
 437:             SimpleTy == MVT::i64 ? &BPF::GPRRegClass : &BPF::GPR32RegClass);
 438:         RegInfo.addLiveIn(VA.getLocReg(), VReg);
 439:         SDValue ArgValue = DAG.getCopyFromReg(Chain, DL, VReg, RegVT);
 440:         InVals.push_back(convertLocValType(DAG, DL, VA, RegVT, ArgValue));
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 441-460

```cpp
 441:         break;
 442:       }
 443:       continue;
 444:     }
 445: 
 446:     if (VA.isMemLoc()) {
 447:       // For example, two stack arguments,
 448:       //   arg1:  Off = 8
 449:       //   arg2:  off = 16
 450:       int Off = VA.getLocMemOffset() + 8;
 451:       if (Off > INT16_MAX) {
 452:         fail(DL, DAG, "extra parameter stack depth exceeded limit");
 453:         break;
 454:       }
 455: 
 456:       // Physical extra argument slot is always 64-bit.
 457:       SDValue StackVal = DAG.getNode(BPFISD::LOAD_STACK_ARG, DL,
 458:                                      DAG.getVTList(MVT::i64, MVT::Other), Chain,
 459:                                      DAG.getConstant(Off, DL, MVT::i64));
 460:       SDValue ArgValue = StackVal.getValue(0);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 461-480

```cpp
 461:       Chain = StackVal.getValue(1);
 462:       InVals.push_back(convertLocValType(DAG, DL, VA, MVT::i64, ArgValue));
 463:       continue;
 464:     }
 465:   }
 466: 
 467:   if (IsVarArg)
 468:     fail(DL, DAG, "variadic functions are not supported");
 469:   return Chain;
 470: }
 471: 
 472: static void resetRegMaskBit(const TargetRegisterInfo *TRI, uint32_t *RegMask,
 473:                             MCRegister Reg) {
 474:   for (MCPhysReg SubReg : TRI->subregs_inclusive(Reg))
 475:     RegMask[SubReg / 32] &= ~(1u << (SubReg % 32));
 476: }
 477: 
 478: static uint32_t *regMaskFromTemplate(const TargetRegisterInfo *TRI,
 479:                                      MachineFunction &MF,
 480:                                      const uint32_t *BaseRegMask) {
```

- EN: Function bodies or method definitions such as resetRegMaskBit contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: resetRegMaskBit 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 481-500

```cpp
 481:   uint32_t *RegMask = MF.allocateRegMask();
 482:   unsigned RegMaskSize = MachineOperand::getRegMaskSize(TRI->getNumRegs());
 483:   memcpy(RegMask, BaseRegMask, sizeof(RegMask[0]) * RegMaskSize);
 484:   return RegMask;
 485: }
 486: 
 487: SDValue BPFTargetLowering::LowerCall(TargetLowering::CallLoweringInfo &CLI,
 488:                                      SmallVectorImpl<SDValue> &InVals) const {
 489:   SelectionDAG &DAG = CLI.DAG;
 490:   auto &Outs = CLI.Outs;
 491:   auto &OutVals = CLI.OutVals;
 492:   auto &Ins = CLI.Ins;
 493:   SDValue Chain = CLI.Chain;
 494:   SDValue Callee = CLI.Callee;
 495:   bool &IsTailCall = CLI.IsTailCall;
 496:   CallingConv::ID CallConv = CLI.CallConv;
 497:   bool IsVarArg = CLI.IsVarArg;
 498:   MachineFunction &MF = DAG.getMachineFunction();
 499: 
 500:   // BPF target does not support tail call optimization.
```

- EN: Function bodies or method definitions such as LowerCall contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerCall 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 501-520

```cpp
 501:   IsTailCall = false;
 502: 
 503:   switch (CallConv) {
 504:   default:
 505:     report_fatal_error("unsupported calling convention: " + Twine(CallConv));
 506:   case CallingConv::Fast:
 507:   case CallingConv::C:
 508:     break;
 509:   }
 510: 
 511:   // Analyze operands of the call, assigning locations to each operand.
 512:   SmallVector<CCValAssign, 16> ArgLocs;
 513:   CCState CCInfo(CallConv, IsVarArg, MF, ArgLocs, *DAG.getContext());
 514: 
 515:   CCInfo.AnalyzeCallOperands(Outs, getHasAlu32() ? CC_BPF32 : CC_BPF64);
 516: 
 517:   unsigned NumBytes = CCInfo.getStackSize();
 518: 
 519:   for (auto &Arg : Outs) {
 520:     ISD::ArgFlagsTy Flags = Arg.Flags;
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 521-540

```cpp
 521:     if (!Flags.isByVal())
 522:       continue;
 523:     fail(CLI.DL, DAG, "pass by value not supported", Callee);
 524:     break;
 525:   }
 526: 
 527:   auto PtrVT = getPointerTy(MF.getDataLayout());
 528:   Chain = DAG.getCALLSEQ_START(Chain, NumBytes, 0, CLI.DL);
 529: 
 530:   SmallVector<std::pair<unsigned, SDValue>, 8> RegsToPass;
 531: 
 532:   // Walk arg assignments
 533:   for (size_t i = 0; i < OutVals.size(); ++i) {
 534:     CCValAssign &VA = ArgLocs[i];
 535:     SDValue &Arg = OutVals[i];
 536: 
 537:     // Promote the value if needed.
 538:     switch (VA.getLocInfo()) {
 539:     default:
 540:       report_fatal_error("unhandled location info: " + Twine(VA.getLocInfo()));
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 541-560

```cpp
 541:     case CCValAssign::Full:
 542:       break;
 543:     case CCValAssign::SExt:
 544:       Arg = DAG.getNode(ISD::SIGN_EXTEND, CLI.DL, VA.getLocVT(), Arg);
 545:       break;
 546:     case CCValAssign::ZExt:
 547:       Arg = DAG.getNode(ISD::ZERO_EXTEND, CLI.DL, VA.getLocVT(), Arg);
 548:       break;
 549:     case CCValAssign::AExt:
 550:       Arg = DAG.getNode(ISD::ANY_EXTEND, CLI.DL, VA.getLocVT(), Arg);
 551:       break;
 552:     }
 553: 
 554:     // Push arguments into RegsToPass vector
 555:     if (VA.isRegLoc()) {
 556:       RegsToPass.push_back(std::make_pair(VA.getLocReg(), Arg));
 557:       continue;
 558:     }
 559: 
 560:     if (VA.isMemLoc()) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 561-580

```cpp
 561:       int Off = -8 - VA.getLocMemOffset();
 562:       if (Off < INT16_MIN) {
 563:         fail(CLI.DL, DAG, "extra parameter stack depth exceeded limit");
 564:         break;
 565:       }
 566: 
 567:       // STORE_STACK_ARG requires i64 operands. With ALU32 mode, the CC
 568:       // promotion may only extend to i32, so extend to i64 if needed.
 569:       if (Arg.getValueType() != MVT::i64)
 570:         Arg = DAG.getNode(ISD::ANY_EXTEND, CLI.DL, MVT::i64, Arg);
 571: 
 572:       SDValue OffVal = DAG.getConstant(Off, CLI.DL, MVT::i64);
 573:       Chain = DAG.getNode(BPFISD::STORE_STACK_ARG, CLI.DL, MVT::Other, Chain,
 574:                           OffVal, Arg);
 575:       continue;
 576:     }
 577: 
 578:     report_fatal_error("unhandled argument location");
 579:   }
 580: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 581-600

```cpp
 581:   SDValue InGlue;
 582: 
 583:   // Build a sequence of copy-to-reg nodes chained together with token chain and
 584:   // flag operands which copy the outgoing args into registers.  The InGlue in
 585:   // necessary since all emitted instructions must be stuck together.
 586:   for (auto &Reg : RegsToPass) {
 587:     Chain = DAG.getCopyToReg(Chain, CLI.DL, Reg.first, Reg.second, InGlue);
 588:     InGlue = Chain.getValue(1);
 589:   }
 590: 
 591:   // If the callee is a GlobalAddress node (quite common, every direct call is)
 592:   // turn it into a TargetGlobalAddress node so that legalize doesn't hack it.
 593:   // Likewise ExternalSymbol -> TargetExternalSymbol.
 594:   if (GlobalAddressSDNode *G = dyn_cast<GlobalAddressSDNode>(Callee)) {
 595:     Callee = DAG.getTargetGlobalAddress(G->getGlobal(), CLI.DL, PtrVT,
 596:                                         G->getOffset(), 0);
 597:   } else if (ExternalSymbolSDNode *E = dyn_cast<ExternalSymbolSDNode>(Callee)) {
 598:     Callee = DAG.getTargetExternalSymbol(E->getSymbol(), PtrVT, 0);
 599:     StringRef Sym = E->getSymbol();
 600:     if (Sym != BPF_TRAP && Sym != "__multi3" && Sym != "__divti3" &&
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 601-620

```cpp
 601:         Sym != "__modti3" && Sym != "__udivti3" && Sym != "__umodti3" &&
 602:         Sym != "memcpy" && Sym != "memset" && Sym != "memmove")
 603:       fail(
 604:           CLI.DL, DAG,
 605:           Twine("A call to built-in function '" + Sym + "' is not supported."));
 606:   }
 607: 
 608:   // Returns a chain & a flag for retval copy to use.
 609:   SDVTList NodeTys = DAG.getVTList(MVT::Other, MVT::Glue);
 610:   SmallVector<SDValue, 8> Ops;
 611:   Ops.push_back(Chain);
 612:   Ops.push_back(Callee);
 613: 
 614:   // Add argument registers to the end of the list so that they are
 615:   // known live into the call.
 616:   for (auto &Reg : RegsToPass)
 617:     Ops.push_back(DAG.getRegister(Reg.first, Reg.second.getValueType()));
 618: 
 619:   bool HasFastCall =
 620:       (CLI.CB && isa<CallInst>(CLI.CB) && CLI.CB->hasFnAttr("bpf_fastcall"));
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 621-640

```cpp
 621:   const TargetRegisterInfo *TRI = MF.getSubtarget().getRegisterInfo();
 622:   if (HasFastCall) {
 623:     uint32_t *RegMask = regMaskFromTemplate(
 624:         TRI, MF, TRI->getCallPreservedMask(MF, CallingConv::PreserveAll));
 625:     for (auto const &RegPair : RegsToPass)
 626:       resetRegMaskBit(TRI, RegMask, RegPair.first);
 627:     if (!CLI.CB->getType()->isVoidTy())
 628:       resetRegMaskBit(TRI, RegMask, BPF::R0);
 629:     Ops.push_back(DAG.getRegisterMask(RegMask));
 630:   } else {
 631:     Ops.push_back(
 632:         DAG.getRegisterMask(TRI->getCallPreservedMask(MF, CLI.CallConv)));
 633:   }
 634: 
 635:   if (InGlue.getNode())
 636:     Ops.push_back(InGlue);
 637: 
 638:   Chain = DAG.getNode(BPFISD::CALL, CLI.DL, NodeTys, Ops);
 639:   InGlue = Chain.getValue(1);
 640: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 641-660

```cpp
 641:   DAG.addNoMergeSiteInfo(Chain.getNode(), CLI.NoMerge);
 642: 
 643:   // Create the CALLSEQ_END node.
 644:   Chain = DAG.getCALLSEQ_END(Chain, NumBytes, 0, InGlue, CLI.DL);
 645:   InGlue = Chain.getValue(1);
 646: 
 647:   // Handle result values, copying them out of physregs into vregs that we
 648:   // return.
 649:   return LowerCallResult(Chain, InGlue, CallConv, IsVarArg, Ins, CLI.DL, DAG,
 650:                          InVals);
 651: }
 652: 
 653: SDValue
 654: BPFTargetLowering::LowerReturn(SDValue Chain, CallingConv::ID CallConv,
 655:                                bool IsVarArg,
 656:                                const SmallVectorImpl<ISD::OutputArg> &Outs,
 657:                                const SmallVectorImpl<SDValue> &OutVals,
 658:                                const SDLoc &DL, SelectionDAG &DAG) const {
 659:   unsigned Opc = BPFISD::RET_GLUE;
 660: 
```

- EN: Function bodies or method definitions such as LowerReturn contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerReturn 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 661-680

```cpp
 661:   // CCValAssign - represent the assignment of the return value to a location
 662:   SmallVector<CCValAssign, 16> RVLocs;
 663:   MachineFunction &MF = DAG.getMachineFunction();
 664: 
 665:   // CCState - Info about the registers and stack slot.
 666:   CCState CCInfo(CallConv, IsVarArg, MF, RVLocs, *DAG.getContext());
 667: 
 668:   // Analize return values.
 669:   CCInfo.AnalyzeReturn(Outs, getHasAlu32() ? RetCC_BPF32 : RetCC_BPF64);
 670: 
 671:   SDValue Glue;
 672:   SmallVector<SDValue, 4> RetOps(1, Chain);
 673: 
 674:   // Copy the result values into the output registers.
 675:   for (size_t i = 0; i != RVLocs.size(); ++i) {
 676:     CCValAssign &VA = RVLocs[i];
 677:     if (!VA.isRegLoc())
 678:       report_fatal_error("stack return values are not supported");
 679: 
 680:     Chain = DAG.getCopyToReg(Chain, DL, VA.getLocReg(), OutVals[i], Glue);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 681-700

```cpp
 681: 
 682:     // Guarantee that all emitted copies are stuck together,
 683:     // avoiding something bad.
 684:     Glue = Chain.getValue(1);
 685:     RetOps.push_back(DAG.getRegister(VA.getLocReg(), VA.getLocVT()));
 686:   }
 687: 
 688:   RetOps[0] = Chain; // Update chain.
 689: 
 690:   // Add the glue if we have it.
 691:   if (Glue.getNode())
 692:     RetOps.push_back(Glue);
 693: 
 694:   return DAG.getNode(Opc, DL, MVT::Other, RetOps);
 695: }
 696: 
 697: SDValue BPFTargetLowering::LowerCallResult(
 698:     SDValue Chain, SDValue InGlue, CallingConv::ID CallConv, bool IsVarArg,
 699:     const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &DL,
 700:     SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
```

- EN: Function bodies or method definitions such as LowerCallResult contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerCallResult 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 701-720

```cpp
 701: 
 702:   MachineFunction &MF = DAG.getMachineFunction();
 703:   // Assign locations to each value returned by this call.
 704:   SmallVector<CCValAssign, 16> RVLocs;
 705:   CCState CCInfo(CallConv, IsVarArg, MF, RVLocs, *DAG.getContext());
 706: 
 707:   CCInfo.AnalyzeCallResult(Ins, getHasAlu32() ? RetCC_BPF32 : RetCC_BPF64);
 708: 
 709:   // Copy all of the result registers out of their specified physreg.
 710:   for (auto &Val : RVLocs) {
 711:     Chain = DAG.getCopyFromReg(Chain, DL, Val.getLocReg(),
 712:                                Val.getValVT(), InGlue).getValue(1);
 713:     InGlue = Chain.getValue(2);
 714:     InVals.push_back(Chain.getValue(0));
 715:   }
 716: 
 717:   return Chain;
 718: }
 719: 
 720: static void NegateCC(SDValue &LHS, SDValue &RHS, ISD::CondCode &CC) {
```

- EN: Function bodies or method definitions such as NegateCC contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: NegateCC 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 721-740

```cpp
 721:   switch (CC) {
 722:   default:
 723:     break;
 724:   case ISD::SETULT:
 725:   case ISD::SETULE:
 726:   case ISD::SETLT:
 727:   case ISD::SETLE:
 728:     CC = ISD::getSetCCSwappedOperands(CC);
 729:     std::swap(LHS, RHS);
 730:     break;
 731:   }
 732: }
 733: 
 734: SDValue BPFTargetLowering::LowerSDIVSREM(SDValue Op, SelectionDAG &DAG) const {
 735:   SDLoc DL(Op);
 736:   fail(DL, DAG,
 737:        "unsupported signed division, please convert to unsigned div/mod.");
 738:   return DAG.getUNDEF(Op->getValueType(0));
 739: }
 740: 
```

- EN: Function bodies or method definitions such as LowerSDIVSREM contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerSDIVSREM 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 741-760

```cpp
 741: SDValue BPFTargetLowering::LowerShiftParts(SDValue Op,
 742:                                            SelectionDAG &DAG) const {
 743:   SDValue Lo, Hi;
 744:   expandShiftParts(Op.getNode(), Lo, Hi, DAG);
 745:   return DAG.getMergeValues({Lo, Hi}, SDLoc(Op));
 746: }
 747: 
 748: SDValue BPFTargetLowering::LowerDYNAMIC_STACKALLOC(SDValue Op,
 749:                                                    SelectionDAG &DAG) const {
 750:   SDLoc DL(Op);
 751:   fail(DL, DAG, "unsupported dynamic stack allocation");
 752:   auto Ops = {DAG.getConstant(0, SDLoc(), Op.getValueType()), Op.getOperand(0)};
 753:   return DAG.getMergeValues(Ops, SDLoc());
 754: }
 755: 
 756: SDValue BPFTargetLowering::LowerBR_CC(SDValue Op, SelectionDAG &DAG) const {
 757:   SDValue Chain = Op.getOperand(0);
 758:   ISD::CondCode CC = cast<CondCodeSDNode>(Op.getOperand(1))->get();
 759:   SDValue LHS = Op.getOperand(2);
 760:   SDValue RHS = Op.getOperand(3);
```

- EN: Function bodies or method definitions such as LowerShiftParts, LowerDYNAMIC_STACKALLOC, LowerBR_CC contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerShiftParts, LowerDYNAMIC_STACKALLOC, LowerBR_CC 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 761-780

```cpp
 761:   SDValue Dest = Op.getOperand(4);
 762:   SDLoc DL(Op);
 763: 
 764:   if (!getHasJmpExt())
 765:     NegateCC(LHS, RHS, CC);
 766: 
 767:   return DAG.getNode(BPFISD::BR_CC, DL, Op.getValueType(), Chain, LHS, RHS,
 768:                      DAG.getConstant(CC, DL, LHS.getValueType()), Dest);
 769: }
 770: 
 771: SDValue BPFTargetLowering::LowerSELECT_CC(SDValue Op, SelectionDAG &DAG) const {
 772:   SDValue LHS = Op.getOperand(0);
 773:   SDValue RHS = Op.getOperand(1);
 774:   SDValue TrueV = Op.getOperand(2);
 775:   SDValue FalseV = Op.getOperand(3);
 776:   ISD::CondCode CC = cast<CondCodeSDNode>(Op.getOperand(4))->get();
 777:   SDLoc DL(Op);
 778: 
 779:   if (!getHasJmpExt())
 780:     NegateCC(LHS, RHS, CC);
```

- EN: Function bodies or method definitions such as LowerSELECT_CC contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerSELECT_CC 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 781-800

```cpp
 781: 
 782:   SDValue TargetCC = DAG.getConstant(CC, DL, LHS.getValueType());
 783:   SDValue Ops[] = {LHS, RHS, TargetCC, TrueV, FalseV};
 784: 
 785:   return DAG.getNode(BPFISD::SELECT_CC, DL, Op.getValueType(), Ops);
 786: }
 787: 
 788: SDValue BPFTargetLowering::LowerATOMIC_LOAD_STORE(SDValue Op,
 789:                                                   SelectionDAG &DAG) const {
 790:   SDNode *N = Op.getNode();
 791:   SDLoc DL(N);
 792: 
 793:   if (cast<AtomicSDNode>(N)->getMergedOrdering() ==
 794:       AtomicOrdering::SequentiallyConsistent)
 795:     fail(DL, DAG,
 796:          "sequentially consistent (seq_cst) "
 797:          "atomic load/store is not supported");
 798: 
 799:   return Op;
 800: }
```

- EN: Function bodies or method definitions such as LowerATOMIC_LOAD_STORE contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerATOMIC_LOAD_STORE 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 801-820

```cpp
 801: 
 802: static Function *createBPFUnreachable(Module *M) {
 803:   if (auto *Fn = M->getFunction(BPF_TRAP))
 804:     return Fn;
 805: 
 806:   FunctionType *FT = FunctionType::get(Type::getVoidTy(M->getContext()), false);
 807:   Function *NewF =
 808:       Function::Create(FT, GlobalValue::ExternalWeakLinkage, BPF_TRAP, M);
 809:   NewF->setDSOLocal(true);
 810:   NewF->setCallingConv(CallingConv::C);
 811:   NewF->setSection(".ksyms");
 812: 
 813:   if (M->debug_compile_units().empty())
 814:     return NewF;
 815: 
 816:   DIBuilder DBuilder(*M);
 817:   DITypeArray ParamTypes =
 818:       DBuilder.getOrCreateTypeArray({nullptr /*void return*/});
 819:   DISubroutineType *FuncType = DBuilder.createSubroutineType(ParamTypes);
 820:   DICompileUnit *CU = *M->debug_compile_units_begin();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 821-840

```cpp
 821:   DISubprogram *SP =
 822:       DBuilder.createFunction(CU, BPF_TRAP, BPF_TRAP, nullptr, 0, FuncType, 0,
 823:                               DINode::FlagZero, DISubprogram::SPFlagZero);
 824:   NewF->setSubprogram(SP);
 825:   return NewF;
 826: }
 827: 
 828: SDValue BPFTargetLowering::LowerTRAP(SDValue Op, SelectionDAG &DAG) const {
 829:   MachineFunction &MF = DAG.getMachineFunction();
 830:   TargetLowering::CallLoweringInfo CLI(DAG);
 831:   SmallVector<SDValue> InVals;
 832:   SDNode *N = Op.getNode();
 833:   SDLoc DL(N);
 834: 
 835:   Function *Fn = createBPFUnreachable(MF.getFunction().getParent());
 836:   auto PtrVT = getPointerTy(MF.getDataLayout());
 837:   CLI.Callee = DAG.getTargetGlobalAddress(Fn, DL, PtrVT);
 838:   CLI.Chain = N->getOperand(0);
 839:   CLI.IsTailCall = false;
 840:   CLI.CallConv = CallingConv::C;
```

- EN: Function bodies or method definitions such as LowerTRAP contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerTRAP 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 841-860

```cpp
 841:   CLI.IsVarArg = false;
 842:   CLI.DL = std::move(DL);
 843:   CLI.NoMerge = false;
 844:   CLI.DoesNotReturn = true;
 845:   return LowerCall(CLI, InVals);
 846: }
 847: 
 848: SDValue BPFTargetLowering::LowerJumpTable(SDValue Op, SelectionDAG &DAG) const {
 849:   JumpTableSDNode *N = cast<JumpTableSDNode>(Op);
 850:   return getAddr(N, DAG);
 851: }
 852: 
 853: static SDValue getTargetNode(ConstantPoolSDNode *N, const SDLoc &DL, EVT Ty,
 854:                              SelectionDAG &DAG, unsigned Flags) {
 855:   return DAG.getTargetConstantPool(N->getConstVal(), Ty, N->getAlign(),
 856:                                    N->getOffset(), Flags);
 857: }
 858: 
 859: static SDValue getTargetNode(JumpTableSDNode *N, const SDLoc &DL, EVT Ty,
 860:                              SelectionDAG &DAG, unsigned Flags) {
```

- EN: Function bodies or method definitions such as LowerJumpTable, getTargetNode contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerJumpTable, getTargetNode 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 861-880

```cpp
 861:   return DAG.getTargetJumpTable(N->getIndex(), Ty, Flags);
 862: }
 863: 
 864: template <class NodeTy>
 865: SDValue BPFTargetLowering::getAddr(NodeTy *N, SelectionDAG &DAG,
 866:                                    unsigned Flags) const {
 867:   SDLoc DL(N);
 868: 
 869:   SDValue GA = getTargetNode(N, DL, MVT::i64, DAG, Flags);
 870: 
 871:   return DAG.getNode(BPFISD::Wrapper, DL, MVT::i64, GA);
 872: }
 873: 
 874: SDValue BPFTargetLowering::LowerGlobalAddress(SDValue Op,
 875:                                               SelectionDAG &DAG) const {
 876:   GlobalAddressSDNode *N = cast<GlobalAddressSDNode>(Op);
 877:   if (N->getOffset() != 0)
 878:     report_fatal_error("invalid offset for global address: " +
 879:                        Twine(N->getOffset()));
 880: 
```

- EN: This chunk introduces interfaces or data structures such as NodeTy, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as getAddr, LowerGlobalAddress contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这一段引入了 NodeTy 等接口或数据结构，用于组织该文件暴露的目标专用行为。 getAddr, LowerGlobalAddress 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 881-900

```cpp
 881:   const GlobalValue *GVal = N->getGlobal();
 882:   SDLoc DL(Op);
 883: 
 884:   // Wrap it in a TargetGlobalAddress
 885:   SDValue Addr = DAG.getTargetGlobalAddress(GVal, DL, MVT::i64);
 886: 
 887:   // Emit pseudo instruction
 888:   return SDValue(DAG.getMachineNode(BPF::LDIMM64, DL, MVT::i64, Addr), 0);
 889: }
 890: 
 891: SDValue BPFTargetLowering::LowerConstantPool(SDValue Op,
 892:                                              SelectionDAG &DAG) const {
 893:   ConstantPoolSDNode *N = cast<ConstantPoolSDNode>(Op);
 894: 
 895:   return getAddr(N, DAG);
 896: }
 897: 
 898: SDValue BPFTargetLowering::LowerBlockAddress(SDValue Op,
 899:                                              SelectionDAG &DAG) const {
 900:   const BlockAddress *BA = cast<BlockAddressSDNode>(Op)->getBlockAddress();
```

- EN: Function bodies or method definitions such as LowerConstantPool, LowerBlockAddress contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: LowerConstantPool, LowerBlockAddress 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 901-920

```cpp
 901:   SDLoc DL(Op);
 902: 
 903:   // Wrap it in a TargetBlockAddress
 904:   SDValue Addr = DAG.getTargetBlockAddress(BA, MVT::i64);
 905: 
 906:   // Emit pseudo instruction
 907:   return SDValue(DAG.getMachineNode(BPF::LDIMM64, DL, MVT::i64, Addr), 0);
 908: }
 909: 
 910: unsigned
 911: BPFTargetLowering::EmitSubregExt(MachineInstr &MI, MachineBasicBlock *BB,
 912:                                  unsigned Reg, bool isSigned) const {
 913:   const TargetInstrInfo &TII = *BB->getParent()->getSubtarget().getInstrInfo();
 914:   const TargetRegisterClass *RC = getRegClassFor(MVT::i64);
 915:   int RShiftOp = isSigned ? BPF::SRA_ri : BPF::SRL_ri;
 916:   MachineFunction *F = BB->getParent();
 917:   DebugLoc DL = MI.getDebugLoc();
 918: 
 919:   MachineRegisterInfo &RegInfo = F->getRegInfo();
 920: 
```

- EN: Function bodies or method definitions such as EmitSubregExt contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: EmitSubregExt 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 921-940

```cpp
 921:   if (!isSigned) {
 922:     Register PromotedReg0 = RegInfo.createVirtualRegister(RC);
 923:     BuildMI(BB, DL, TII.get(BPF::MOV_32_64), PromotedReg0).addReg(Reg);
 924:     return PromotedReg0;
 925:   }
 926:   Register PromotedReg0 = RegInfo.createVirtualRegister(RC);
 927:   Register PromotedReg1 = RegInfo.createVirtualRegister(RC);
 928:   Register PromotedReg2 = RegInfo.createVirtualRegister(RC);
 929:   if (HasMovsx) {
 930:     BuildMI(BB, DL, TII.get(BPF::MOVSX_rr_32), PromotedReg0).addReg(Reg);
 931:   } else {
 932:     BuildMI(BB, DL, TII.get(BPF::MOV_32_64), PromotedReg0).addReg(Reg);
 933:     BuildMI(BB, DL, TII.get(BPF::SLL_ri), PromotedReg1)
 934:       .addReg(PromotedReg0).addImm(32);
 935:     BuildMI(BB, DL, TII.get(RShiftOp), PromotedReg2)
 936:       .addReg(PromotedReg1).addImm(32);
 937:   }
 938: 
 939:   return PromotedReg2;
 940: }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 941-960

```cpp
 941: 
 942: MachineBasicBlock *
 943: BPFTargetLowering::EmitInstrWithCustomInserterMemcpy(MachineInstr &MI,
 944:                                                      MachineBasicBlock *BB)
 945:                                                      const {
 946:   MachineFunction *MF = MI.getParent()->getParent();
 947:   MachineRegisterInfo &MRI = MF->getRegInfo();
 948:   MachineInstrBuilder MIB(*MF, MI);
 949:   unsigned ScratchReg;
 950: 
 951:   // This function does custom insertion during lowering BPFISD::MEMCPY which
 952:   // only has two register operands from memcpy semantics, the copy source
 953:   // address and the copy destination address.
 954:   //
 955:   // Because we will expand BPFISD::MEMCPY into load/store pairs, we will need
 956:   // a third scratch register to serve as the destination register of load and
 957:   // source register of store.
 958:   //
 959:   // The scratch register here is with the Define | Dead | EarlyClobber flags.
 960:   // The EarlyClobber flag has the semantic property that the operand it is
```

- EN: Function bodies or method definitions such as EmitInstrWithCustomInserterMemcpy contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: EmitInstrWithCustomInserterMemcpy 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 961-980

```cpp
 961:   // attached to is clobbered before the rest of the inputs are read. Hence it
 962:   // must be unique among the operands to the instruction. The Define flag is
 963:   // needed to coerce the machine verifier that an Undef value isn't a problem
 964:   // as we anyway is loading memory into it. The Dead flag is needed as the
 965:   // value in scratch isn't supposed to be used by any other instruction.
 966:   ScratchReg = MRI.createVirtualRegister(&BPF::GPRRegClass);
 967:   MIB.addReg(ScratchReg,
 968:              RegState::Define | RegState::Dead | RegState::EarlyClobber);
 969: 
 970:   return BB;
 971: }
 972: 
 973: MachineBasicBlock *BPFTargetLowering::EmitInstrWithCustomInserterLDimm64(
 974:     MachineInstr &MI, MachineBasicBlock *BB) const {
 975:   MachineFunction *MF = BB->getParent();
 976:   const BPFInstrInfo *TII = MF->getSubtarget<BPFSubtarget>().getInstrInfo();
 977:   const TargetRegisterClass *RC = getRegClassFor(MVT::i64);
 978:   MachineRegisterInfo &RegInfo = MF->getRegInfo();
 979:   DebugLoc DL = MI.getDebugLoc();
 980: 
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 981-1000

```cpp
 981:   // Build address taken map for Global Varaibles and BlockAddresses
 982:   DenseMap<const BasicBlock *, MachineBasicBlock *> AddressTakenBBs;
 983:   for (MachineBasicBlock &MBB : *MF) {
 984:     if (const BasicBlock *BB = MBB.getBasicBlock())
 985:       if (BB->hasAddressTaken())
 986:         AddressTakenBBs[BB] = &MBB;
 987:   }
 988: 
 989:   MachineOperand &MO = MI.getOperand(1);
 990:   assert(MO.isBlockAddress() || MO.isGlobal());
 991: 
 992:   Register ResultReg = MI.getOperand(0).getReg();
 993:   Register TmpReg = RegInfo.createVirtualRegister(RC);
 994: 
 995:   std::vector<MachineBasicBlock *> Targets;
 996:   unsigned JTI;
 997: 
 998:   if (MO.isBlockAddress()) {
 999:     auto *BA = MO.getBlockAddress();
1000:     MachineBasicBlock *TgtMBB = AddressTakenBBs[BA->getBasicBlock()];
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1001-1020

```cpp
1001:     assert(TgtMBB);
1002: 
1003:     Targets.push_back(TgtMBB);
1004:     JTI = MF->getOrCreateJumpTableInfo(getJumpTableEncoding())
1005:               ->createJumpTableIndex(Targets);
1006: 
1007:     BuildMI(*BB, MI, DL, TII->get(BPF::LD_imm64), TmpReg)
1008:         .addJumpTableIndex(JTI);
1009:     BuildMI(*BB, MI, DL, TII->get(BPF::LDD), ResultReg)
1010:         .addReg(TmpReg)
1011:         .addImm(0);
1012:     MI.eraseFromParent();
1013:     return BB;
1014:   }
1015: 
1016:   // Helper: emit LD_imm64 with operand GlobalAddress or JumpTable
1017:   auto emitLDImm64 = [&](const GlobalValue *GV = nullptr, unsigned JTI = -1) {
1018:     auto MIB = BuildMI(*BB, MI, DL, TII->get(BPF::LD_imm64), ResultReg);
1019:     if (GV)
1020:       MIB.addGlobalAddress(GV);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1021-1040

```cpp
1021:     else
1022:       MIB.addJumpTableIndex(JTI);
1023:     MI.eraseFromParent();
1024:     return BB;
1025:   };
1026: 
1027:   // Must be a global at this point
1028:   const GlobalValue *GVal = MO.getGlobal();
1029:   const auto *GV = dyn_cast<GlobalVariable>(GVal);
1030: 
1031:   if (!GV || GV->getLinkage() != GlobalValue::PrivateLinkage ||
1032:       !GV->isConstant() || !GV->hasInitializer())
1033:     return emitLDImm64(GVal);
1034: 
1035:   const auto *CA = dyn_cast<ConstantArray>(GV->getInitializer());
1036:   if (!CA)
1037:     return emitLDImm64(GVal);
1038: 
1039:   for (const Use &Op : CA->operands()) {
1040:     if (!isa<BlockAddress>(Op))
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1041-1060

```cpp
1041:       return emitLDImm64(GVal);
1042:     auto *BA = cast<BlockAddress>(Op);
1043:     MachineBasicBlock *TgtMBB = AddressTakenBBs[BA->getBasicBlock()];
1044:     assert(TgtMBB);
1045:     Targets.push_back(TgtMBB);
1046:   }
1047: 
1048:   JTI = MF->getOrCreateJumpTableInfo(getJumpTableEncoding())
1049:             ->createJumpTableIndex(Targets);
1050:   return emitLDImm64(nullptr, JTI);
1051: }
1052: 
1053: MachineBasicBlock *
1054: BPFTargetLowering::EmitInstrWithCustomInserter(MachineInstr &MI,
1055:                                                MachineBasicBlock *BB) const {
1056:   const TargetInstrInfo &TII = *BB->getParent()->getSubtarget().getInstrInfo();
1057:   DebugLoc DL = MI.getDebugLoc();
1058:   unsigned Opc = MI.getOpcode();
1059:   bool isSelectRROp = (Opc == BPF::Select ||
1060:                        Opc == BPF::Select_64_32 ||
```

- EN: Function bodies or method definitions such as EmitInstrWithCustomInserter contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: EmitInstrWithCustomInserter 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1061-1080

```cpp
1061:                        Opc == BPF::Select_32 ||
1062:                        Opc == BPF::Select_32_64);
1063: 
1064:   bool isMemcpyOp = Opc == BPF::MEMCPY;
1065:   bool isLDimm64Op = Opc == BPF::LDIMM64;
1066: 
1067: #ifndef NDEBUG
1068:   bool isSelectRIOp = (Opc == BPF::Select_Ri ||
1069:                        Opc == BPF::Select_Ri_64_32 ||
1070:                        Opc == BPF::Select_Ri_32 ||
1071:                        Opc == BPF::Select_Ri_32_64);
1072: 
1073:   if (!(isSelectRROp || isSelectRIOp || isMemcpyOp || isLDimm64Op))
1074:     report_fatal_error("unhandled instruction type: " + Twine(Opc));
1075: #endif
1076: 
1077:   if (isMemcpyOp)
1078:     return EmitInstrWithCustomInserterMemcpy(MI, BB);
1079: 
1080:   if (isLDimm64Op)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1081-1100

```cpp
1081:     return EmitInstrWithCustomInserterLDimm64(MI, BB);
1082: 
1083:   bool is32BitCmp = (Opc == BPF::Select_32 ||
1084:                      Opc == BPF::Select_32_64 ||
1085:                      Opc == BPF::Select_Ri_32 ||
1086:                      Opc == BPF::Select_Ri_32_64);
1087: 
1088:   // To "insert" a SELECT instruction, we actually have to insert the diamond
1089:   // control-flow pattern.  The incoming instruction knows the destination vreg
1090:   // to set, the condition code register to branch on, the true/false values to
1091:   // select between, and a branch opcode to use.
1092:   const BasicBlock *LLVM_BB = BB->getBasicBlock();
1093:   MachineFunction::iterator I = ++BB->getIterator();
1094: 
1095:   // ThisMBB:
1096:   // ...
1097:   //  TrueVal = ...
1098:   //  jmp_XX r1, r2 goto Copy1MBB
1099:   //  fallthrough --> Copy0MBB
1100:   MachineBasicBlock *ThisMBB = BB;
```

- EN: At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1101-1120

```cpp
1101:   MachineFunction *F = BB->getParent();
1102:   MachineBasicBlock *Copy0MBB = F->CreateMachineBasicBlock(LLVM_BB);
1103:   MachineBasicBlock *Copy1MBB = F->CreateMachineBasicBlock(LLVM_BB);
1104: 
1105:   F->insert(I, Copy0MBB);
1106:   F->insert(I, Copy1MBB);
1107:   // Update machine-CFG edges by transferring all successors of the current
1108:   // block to the new block which will contain the Phi node for the select.
1109:   Copy1MBB->splice(Copy1MBB->begin(), BB,
1110:                    std::next(MachineBasicBlock::iterator(MI)), BB->end());
1111:   Copy1MBB->transferSuccessorsAndUpdatePHIs(BB);
1112:   // Next, add the true and fallthrough blocks as its successors.
1113:   BB->addSuccessor(Copy0MBB);
1114:   BB->addSuccessor(Copy1MBB);
1115: 
1116:   // Insert Branch if Flag
1117:   int CC = MI.getOperand(3).getImm();
1118:   int NewCC;
1119:   switch (CC) {
1120: #define SET_NEWCC(X, Y) \
```

- EN: Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1121-1140

```cpp
1121:   case ISD::X: \
1122:     if (is32BitCmp && HasJmp32) \
1123:       NewCC = isSelectRROp ? BPF::Y##_rr_32 : BPF::Y##_ri_32; \
1124:     else \
1125:       NewCC = isSelectRROp ? BPF::Y##_rr : BPF::Y##_ri; \
1126:     break
1127:   SET_NEWCC(SETGT, JSGT);
1128:   SET_NEWCC(SETUGT, JUGT);
1129:   SET_NEWCC(SETGE, JSGE);
1130:   SET_NEWCC(SETUGE, JUGE);
1131:   SET_NEWCC(SETEQ, JEQ);
1132:   SET_NEWCC(SETNE, JNE);
1133:   SET_NEWCC(SETLT, JSLT);
1134:   SET_NEWCC(SETULT, JULT);
1135:   SET_NEWCC(SETLE, JSLE);
1136:   SET_NEWCC(SETULE, JULE);
1137:   default:
1138:     report_fatal_error("unimplemented select CondCode " + Twine(CC));
1139:   }
1140: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1141-1160

```cpp
1141:   Register LHS = MI.getOperand(1).getReg();
1142:   bool isSignedCmp = (CC == ISD::SETGT ||
1143:                       CC == ISD::SETGE ||
1144:                       CC == ISD::SETLT ||
1145:                       CC == ISD::SETLE);
1146: 
1147:   // eBPF at the moment only has 64-bit comparison. Any 32-bit comparison need
1148:   // to be promoted, however if the 32-bit comparison operands are destination
1149:   // registers then they are implicitly zero-extended already, there is no
1150:   // need of explicit zero-extend sequence for them.
1151:   //
1152:   // We simply do extension for all situations in this method, but we will
1153:   // try to remove those unnecessary in BPFMIPeephole pass.
1154:   if (is32BitCmp && !HasJmp32)
1155:     LHS = EmitSubregExt(MI, BB, LHS, isSignedCmp);
1156: 
1157:   if (isSelectRROp) {
1158:     Register RHS = MI.getOperand(2).getReg();
1159: 
1160:     if (is32BitCmp && !HasJmp32)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1161-1180

```cpp
1161:       RHS = EmitSubregExt(MI, BB, RHS, isSignedCmp);
1162: 
1163:     BuildMI(BB, DL, TII.get(NewCC)).addReg(LHS).addReg(RHS).addMBB(Copy1MBB);
1164:   } else {
1165:     int64_t imm32 = MI.getOperand(2).getImm();
1166:     // Check before we build J*_ri instruction.
1167:     if (!isInt<32>(imm32))
1168:       report_fatal_error("immediate overflows 32 bits: " + Twine(imm32));
1169:     BuildMI(BB, DL, TII.get(NewCC))
1170:         .addReg(LHS).addImm(imm32).addMBB(Copy1MBB);
1171:   }
1172: 
1173:   // Copy0MBB:
1174:   //  %FalseValue = ...
1175:   //  # fallthrough to Copy1MBB
1176:   BB = Copy0MBB;
1177: 
1178:   // Update machine-CFG edges
1179:   BB->addSuccessor(Copy1MBB);
1180: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1181-1200

```cpp
1181:   // Copy1MBB:
1182:   //  %Result = phi [ %FalseValue, Copy0MBB ], [ %TrueValue, ThisMBB ]
1183:   // ...
1184:   BB = Copy1MBB;
1185:   BuildMI(*BB, BB->begin(), DL, TII.get(BPF::PHI), MI.getOperand(0).getReg())
1186:       .addReg(MI.getOperand(5).getReg())
1187:       .addMBB(Copy0MBB)
1188:       .addReg(MI.getOperand(4).getReg())
1189:       .addMBB(ThisMBB);
1190: 
1191:   MI.eraseFromParent(); // The pseudo instruction is gone now.
1192:   return BB;
1193: }
1194: 
1195: EVT BPFTargetLowering::getSetCCResultType(const DataLayout &, LLVMContext &,
1196:                                           EVT VT) const {
1197:   return getHasAlu32() ? MVT::i32 : MVT::i64;
1198: }
1199: 
1200: MVT BPFTargetLowering::getScalarShiftAmountTy(const DataLayout &DL,
```

- EN: Function bodies or method definitions such as getSetCCResultType contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: getSetCCResultType 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1201-1220

```cpp
1201:                                               EVT VT) const {
1202:   return (getHasAlu32() && VT == MVT::i32) ? MVT::i32 : MVT::i64;
1203: }
1204: 
1205: bool BPFTargetLowering::isLegalAddressingMode(const DataLayout &DL,
1206:                                               const AddrMode &AM, Type *Ty,
1207:                                               unsigned AS,
1208:                                               Instruction *I) const {
1209:   // No global is ever allowed as a base.
1210:   if (AM.BaseGV)
1211:     return false;
1212: 
1213:   switch (AM.Scale) {
1214:   case 0: // "r+i" or just "i", depending on HasBaseReg.
1215:     break;
1216:   case 1:
1217:     if (!AM.HasBaseReg) // allow "r+i".
1218:       break;
1219:     return false; // disallow "r+r" or "r+r+i".
1220:   default:
```

- EN: Function bodies or method definitions such as isLegalAddressingMode contain the concrete backend logic executed by LLVM passes or MC helpers. Switch-based dispatch is used here to separate behavior by opcode, operand kind, or subtarget condition. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: isLegalAddressingMode 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里通过 switch 分派不同的操作码、操作数类型或子目标条件下的行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

### Lines 1221-1234

```cpp
1221:     return false;
1222:   }
1223: 
1224:   return true;
1225: }
1226: 
1227: bool BPFTargetLowering::CanLowerReturn(
1228:     CallingConv::ID CallConv, MachineFunction &MF, bool IsVarArg,
1229:     const SmallVectorImpl<ISD::OutputArg> &Outs, LLVMContext &Context,
1230:     const Type *RetTy) const {
1231:   SmallVector<CCValAssign, 16> RVLocs;
1232:   CCState CCInfo(CallConv, IsVarArg, MF, RVLocs, Context);
1233:   return CCInfo.CheckReturn(Outs, getHasAlu32() ? RetCC_BPF32 : RetCC_BPF64);
1234: }
```

- EN: Function bodies or method definitions such as CanLowerReturn contain the concrete backend logic executed by LLVM passes or MC helpers. At the backend level, this chunk supports lowering generic IR or DAG operations into target-specific forms.
- 中文: CanLowerReturn 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 在后端层面，这一段支持把通用 IR 或 DAG 操作降级为目标专用形式。

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

- Direct includes / 直接包含: `BPFISelLowering.h`, `BPF.h`, `BPFSubtarget.h`, `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineJumpTableInfo.h`
- LLVM subsystems / LLVM 子系统: LLVM CodeGen, SelectionDAG
- Generated or companion files / 生成或配套文件: `BPFGenCallingConv.inc`
- Local companions / 本地配套文件: `BPFISelLowering.h`
