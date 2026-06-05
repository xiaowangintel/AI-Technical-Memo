# BPFPreserveStaticOffset.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFPreserveStaticOffset.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): This file implements BPFPreserveStaticOffsetPass transformation.
- 目的（中文）: 提供目标后端使用的专用支持代码、声明或辅助例程。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===------ BPFPreserveStaticOffset.cpp -----------------------------------===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // TLDR: replaces llvm.preserve.static.offset + GEP + load / store
  10: //           with llvm.bpf.getelementptr.and.load / store
  11: //
  12: // This file implements BPFPreserveStaticOffsetPass transformation.
  13: // This transformation address two BPF verifier specific issues:
  14: //
  15: // (a) Access to the fields of some structural types is allowed only
  16: //     using load and store instructions with static immediate offsets.
  17: //
  18: //     Examples of such types are `struct __sk_buff` and `struct
  19: //     bpf_sock_ops`.  This is so because offsets of the fields of
  20: //     these structures do not match real offsets in the running
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This chunk introduces interfaces or data structures such as __sk_buff, which organize the target-specific behavior exposed by the file.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段引入了 __sk_buff 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 21-40

```cpp
  21: //     kernel. During BPF program load LDX and STX instructions
  22: //     referring to the fields of these types are rewritten so that
  23: //     offsets match real offsets. For this rewrite to happen field
  24: //     offsets have to be encoded as immediate operands of the
  25: //     instructions.
  26: //
  27: //     See kernel/bpf/verifier.c:convert_ctx_access function in the
  28: //     Linux kernel source tree for details.
  29: //
  30: // (b) Pointers to context parameters of BPF programs must not be
  31: //     modified before access.
  32: //
  33: //     During BPF program verification a tag PTR_TO_CTX is tracked for
  34: //     register values. In case if register with such tag is modified
  35: //     BPF program is not allowed to read or write memory using this
  36: //     register. See kernel/bpf/verifier.c:check_mem_access function
  37: //     in the Linux kernel source tree for details.
  38: //
  39: // The following sequence of the IR instructions:
  40: //
```

- EN: This range continues the implementation of the backend component described by BPFPreserveStaticOffset.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 41-60

```cpp
  41: //   %x = getelementptr %ptr, %constant_offset
  42: //   %y = load %x
  43: //
  44: // Is translated as a single machine instruction:
  45: //
  46: //   LDW %ptr, %constant_offset
  47: //
  48: // In order for cases (a) and (b) to work the sequence %x-%y above has
  49: // to be preserved by the IR passes.
  50: //
  51: // However, several optimization passes might sink `load` instruction
  52: // or hoist `getelementptr` instruction so that the instructions are
  53: // no longer in sequence. Examples of such passes are:
  54: // SimplifyCFGPass, InstCombinePass, GVNPass.
  55: // After such modification the verifier would reject the BPF program.
  56: //
  57: // To avoid this issue the patterns like (load/store (getelementptr ...))
  58: // are replaced by calls to BPF specific intrinsic functions:
  59: // - llvm.bpf.getelementptr.and.load
  60: // - llvm.bpf.getelementptr.and.store
```

- EN: This range continues the implementation of the backend component described by BPFPreserveStaticOffset.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 61-80

```cpp
  61: //
  62: // These calls are lowered back to (load/store (getelementptr ...))
  63: // by BPFCheckAndAdjustIR pass right before the translation from IR to
  64: // machine instructions.
  65: //
  66: // The transformation is split into the following steps:
  67: // - When IR is generated from AST the calls to intrinsic function
  68: //   llvm.preserve.static.offset are inserted.
  69: // - BPFPreserveStaticOffsetPass is executed as early as possible
  70: //   with AllowPatial set to true, this handles marked GEP chains
  71: //   with constant offsets.
  72: // - BPFPreserveStaticOffsetPass is executed at ScalarOptimizerLateEPCallback
  73: //   with AllowPatial set to false, this handles marked GEP chains
  74: //   with offsets that became constant after loop unrolling, e.g.
  75: //   to handle the following code:
  76: //
  77: // struct context { int x[4]; } __attribute__((preserve_static_offset));
  78: //
  79: //   struct context *ctx = ...;
  80: // #pragma clang loop unroll(full)
```

- EN: This chunk introduces interfaces or data structures such as context, which organize the target-specific behavior exposed by the file.
- 中文: 这一段引入了 context 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 81-100

```cpp
  81: //   for (int i = 0; i < 4; ++i)
  82: //     foo(ctx->x[i]);
  83: //
  84: // The early BPFPreserveStaticOffsetPass run is necessary to allow
  85: // additional GVN / CSE opportunities after functions inlining.
  86: // The relative order of optimization applied to function:
  87: // - early stage (1)
  88: // - ...
  89: // - function inlining (2)
  90: // - ...
  91: // - loop unrolling
  92: // - ...
  93: // - ScalarOptimizerLateEPCallback (3)
  94: //
  95: // When function A is inlined into function B all optimizations for A
  96: // are already done, while some passes remain for B. In case if
  97: // BPFPreserveStaticOffsetPass is done at (3) but not done at (1)
  98: // the code after (2) would contain a mix of
  99: // (load (gep %p)) and (get.and.load %p) usages:
 100: // - the (load (gep %p)) would come from the calling function;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 101-120

```cpp
 101: // - the (get.and.load %p) would come from the callee function.
 102: // Thus clobbering CSE / GVN passes done after inlining.
 103: 
 104: #include "BPF.h"
 105: #include "BPFCORE.h"
 106: #include "llvm/ADT/SmallPtrSet.h"
 107: #include "llvm/ADT/SmallVector.h"
 108: #include "llvm/IR/Attributes.h"
 109: #include "llvm/IR/BasicBlock.h"
 110: #include "llvm/IR/Constants.h"
 111: #include "llvm/IR/DiagnosticInfo.h"
 112: #include "llvm/IR/IRBuilder.h"
 113: #include "llvm/IR/InstIterator.h"
 114: #include "llvm/IR/Instructions.h"
 115: #include "llvm/IR/Intrinsics.h"
 116: #include "llvm/IR/IntrinsicsBPF.h"
 117: #include "llvm/IR/Module.h"
 118: #include "llvm/Support/Debug.h"
 119: #include "llvm/Support/ErrorHandling.h"
 120: 
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 121-140

```cpp
 121: #define DEBUG_TYPE "bpf-preserve-static-offset"
 122: 
 123: using namespace llvm;
 124: 
 125: static const unsigned GepAndLoadFirstIdxArg = 6;
 126: static const unsigned GepAndStoreFirstIdxArg = 7;
 127: 
 128: static bool isIntrinsicCall(Value *I, Intrinsic::ID Id) {
 129:   if (auto *Call = dyn_cast<CallInst>(I))
 130:     if (Function *Func = Call->getCalledFunction())
 131:       return Func->getIntrinsicID() == Id;
 132:   return false;
 133: }
 134: 
 135: static bool isPreserveStaticOffsetCall(Value *I) {
 136:   return isIntrinsicCall(I, Intrinsic::preserve_static_offset);
 137: }
 138: 
 139: static CallInst *isGEPAndLoad(Value *I) {
 140:   if (isIntrinsicCall(I, Intrinsic::bpf_getelementptr_and_load))
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. Function bodies or method definitions such as isIntrinsicCall, isPreserveStaticOffsetCall contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 isIntrinsicCall, isPreserveStaticOffsetCall 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 141-160

```cpp
 141:     return cast<CallInst>(I);
 142:   return nullptr;
 143: }
 144: 
 145: static CallInst *isGEPAndStore(Value *I) {
 146:   if (isIntrinsicCall(I, Intrinsic::bpf_getelementptr_and_store))
 147:     return cast<CallInst>(I);
 148:   return nullptr;
 149: }
 150: 
 151: template <class T = Instruction>
 152: static DebugLoc mergeDebugLocs(SmallVector<T *> &Insns) {
 153:   DebugLoc Merged = (*Insns.begin())->getDebugLoc();
 154:   for (T *I : Insns)
 155:     Merged = DebugLoc::getMergedLocation(Merged, I->getDebugLoc());
 156:   return Merged;
 157: }
 158: 
 159: static CallInst *makeIntrinsicCall(Module *M,
 160:                                    Intrinsic::BPFIntrinsics Intrinsic,
```

- EN: This chunk introduces interfaces or data structures such as T, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as mergeDebugLocs contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这一段引入了 T 等接口或数据结构，用于组织该文件暴露的目标专用行为。 mergeDebugLocs 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 161-180

```cpp
 161:                                    ArrayRef<Type *> Types,
 162:                                    ArrayRef<Value *> Args) {
 163: 
 164:   Function *Fn = Intrinsic::getOrInsertDeclaration(M, Intrinsic, Types);
 165:   return CallInst::Create(Fn, Args);
 166: }
 167: 
 168: static void setParamElementType(CallInst *Call, unsigned ArgNo, Type *Type) {
 169:   LLVMContext &C = Call->getContext();
 170:   Call->addParamAttr(ArgNo, Attribute::get(C, Attribute::ElementType, Type));
 171: }
 172: 
 173: static void setParamReadNone(CallInst *Call, unsigned ArgNo) {
 174:   LLVMContext &C = Call->getContext();
 175:   Call->addParamAttr(ArgNo, Attribute::get(C, Attribute::ReadNone));
 176: }
 177: 
 178: static void setParamReadOnly(CallInst *Call, unsigned ArgNo) {
 179:   LLVMContext &C = Call->getContext();
 180:   Call->addParamAttr(ArgNo, Attribute::get(C, Attribute::ReadOnly));
```

- EN: Function bodies or method definitions such as setParamElementType, setParamReadNone, setParamReadOnly contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: setParamElementType, setParamReadNone, setParamReadOnly 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 181-200

```cpp
 181: }
 182: 
 183: static void setParamWriteOnly(CallInst *Call, unsigned ArgNo) {
 184:   LLVMContext &C = Call->getContext();
 185:   Call->addParamAttr(ArgNo, Attribute::get(C, Attribute::WriteOnly));
 186: }
 187: 
 188: namespace {
 189: struct GEPChainInfo {
 190:   bool InBounds;
 191:   Type *SourceElementType;
 192:   SmallVector<Value *> Indices;
 193:   SmallVector<GetElementPtrInst *> Members;
 194: 
 195:   GEPChainInfo() { reset(); }
 196: 
 197:   void reset() {
 198:     InBounds = true;
 199:     SourceElementType = nullptr;
 200:     Indices.clear();
```

- EN: This chunk introduces interfaces or data structures such as GEPChainInfo, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as setParamWriteOnly, reset contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 这一段引入了 GEPChainInfo 等接口或数据结构，用于组织该文件暴露的目标专用行为。 setParamWriteOnly, reset 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 201-220

```cpp
 201:     Members.clear();
 202:   }
 203: };
 204: } // Anonymous namespace
 205: 
 206: template <class T = std::disjunction<LoadInst, StoreInst>>
 207: static void fillCommonArgs(LLVMContext &C, SmallVector<Value *> &Args,
 208:                            GEPChainInfo &GEP, T *Insn) {
 209:   Type *Int8Ty = Type::getInt8Ty(C);
 210:   Type *Int1Ty = Type::getInt1Ty(C);
 211:   // Implementation of Align guarantees that ShiftValue < 64
 212:   unsigned AlignShiftValue = Log2_64(Insn->getAlign().value());
 213:   Args.push_back(GEP.Members[0]->getPointerOperand());
 214:   Args.push_back(ConstantInt::get(Int1Ty, Insn->isVolatile()));
 215:   Args.push_back(ConstantInt::get(Int8Ty, (unsigned)Insn->getOrdering()));
 216:   Args.push_back(ConstantInt::get(Int8Ty, (unsigned)Insn->getSyncScopeID()));
 217:   Args.push_back(ConstantInt::get(Int8Ty, AlignShiftValue));
 218:   Args.push_back(ConstantInt::get(Int1Ty, GEP.InBounds));
 219:   Args.append(GEP.Indices.begin(), GEP.Indices.end());
 220: }
```

- EN: This chunk introduces interfaces or data structures such as T, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as fillCommonArgs contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 这一段引入了 T 等接口或数据结构，用于组织该文件暴露的目标专用行为。 fillCommonArgs 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 221-240

```cpp
 221: 
 222: static Instruction *makeGEPAndLoad(Module *M, GEPChainInfo &GEP,
 223:                                    LoadInst *Load) {
 224:   SmallVector<Value *> Args;
 225:   fillCommonArgs(M->getContext(), Args, GEP, Load);
 226:   CallInst *Call = makeIntrinsicCall(M, Intrinsic::bpf_getelementptr_and_load,
 227:                                      {Load->getType()}, Args);
 228:   setParamElementType(Call, 0, GEP.SourceElementType);
 229:   Call->applyMergedLocation(mergeDebugLocs(GEP.Members), Load->getDebugLoc());
 230:   Call->setName((*GEP.Members.rbegin())->getName());
 231:   if (Load->isUnordered()) {
 232:     Call->setOnlyReadsMemory();
 233:     Call->setOnlyAccessesArgMemory();
 234:     setParamReadOnly(Call, 0);
 235:   }
 236:   for (unsigned I = GepAndLoadFirstIdxArg; I < Args.size(); ++I)
 237:     Call->addParamAttr(I, Attribute::ImmArg);
 238:   Call->setAAMetadata(Load->getAAMetadata());
 239:   return Call;
 240: }
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 241-260

```cpp
 241: 
 242: static Instruction *makeGEPAndStore(Module *M, GEPChainInfo &GEP,
 243:                                     StoreInst *Store) {
 244:   SmallVector<Value *> Args;
 245:   Args.push_back(Store->getValueOperand());
 246:   fillCommonArgs(M->getContext(), Args, GEP, Store);
 247:   CallInst *Call =
 248:       makeIntrinsicCall(M, Intrinsic::bpf_getelementptr_and_store,
 249:                         {Store->getValueOperand()->getType()}, Args);
 250:   setParamElementType(Call, 1, GEP.SourceElementType);
 251:   if (Store->getValueOperand()->getType()->isPointerTy())
 252:     setParamReadNone(Call, 0);
 253:   Call->applyMergedLocation(mergeDebugLocs(GEP.Members), Store->getDebugLoc());
 254:   if (Store->isUnordered()) {
 255:     Call->setOnlyWritesMemory();
 256:     Call->setOnlyAccessesArgMemory();
 257:     setParamWriteOnly(Call, 1);
 258:   }
 259:   for (unsigned I = GepAndStoreFirstIdxArg; I < Args.size(); ++I)
 260:     Call->addParamAttr(I, Attribute::ImmArg);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 261-280

```cpp
 261:   Call->setAAMetadata(Store->getAAMetadata());
 262:   return Call;
 263: }
 264: 
 265: static unsigned getOperandAsUnsigned(CallInst *Call, unsigned ArgNo) {
 266:   if (auto *Int = dyn_cast<ConstantInt>(Call->getOperand(ArgNo)))
 267:     return Int->getValue().getZExtValue();
 268:   std::string Report;
 269:   raw_string_ostream ReportS(Report);
 270:   ReportS << "Expecting ConstantInt as argument #" << ArgNo << " of " << *Call
 271:           << "\n";
 272:   report_fatal_error(StringRef(Report));
 273: }
 274: 
 275: static GetElementPtrInst *reconstructGEP(CallInst *Call, int Delta) {
 276:   SmallVector<Value *> Indices;
 277:   Indices.append(Call->data_operands_begin() + 6 + Delta,
 278:                  Call->data_operands_end());
 279:   Type *GEPPointeeType = Call->getParamElementType(Delta);
 280:   auto *GEP =
```

- EN: Function bodies or method definitions such as getOperandAsUnsigned contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: getOperandAsUnsigned 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 281-300

```cpp
 281:       GetElementPtrInst::Create(GEPPointeeType, Call->getOperand(Delta),
 282:                                 ArrayRef<Value *>(Indices), Call->getName());
 283:   GEP->setIsInBounds(getOperandAsUnsigned(Call, 5 + Delta));
 284:   return GEP;
 285: }
 286: 
 287: template <class T = std::disjunction<LoadInst, StoreInst>>
 288: static void reconstructCommon(CallInst *Call, GetElementPtrInst *GEP, T *Insn,
 289:                               int Delta) {
 290:   Insn->setVolatile(getOperandAsUnsigned(Call, 1 + Delta));
 291:   Insn->setOrdering((AtomicOrdering)getOperandAsUnsigned(Call, 2 + Delta));
 292:   Insn->setSyncScopeID(getOperandAsUnsigned(Call, 3 + Delta));
 293:   unsigned AlignShiftValue = getOperandAsUnsigned(Call, 4 + Delta);
 294:   Insn->setAlignment(Align(1ULL << AlignShiftValue));
 295:   GEP->setDebugLoc(Call->getDebugLoc());
 296:   Insn->setDebugLoc(Call->getDebugLoc());
 297:   Insn->setAAMetadata(Call->getAAMetadata());
 298: }
 299: 
 300: std::pair<GetElementPtrInst *, LoadInst *>
```

- EN: This chunk introduces interfaces or data structures such as T, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as reconstructCommon contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: 这一段引入了 T 等接口或数据结构，用于组织该文件暴露的目标专用行为。 reconstructCommon 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 301-320

```cpp
 301: BPFPreserveStaticOffsetPass::reconstructLoad(CallInst *Call) {
 302:   GetElementPtrInst *GEP = reconstructGEP(Call, 0);
 303:   Type *ReturnType = Call->getFunctionType()->getReturnType();
 304:   auto *Load = new LoadInst(ReturnType, GEP, "",
 305:                             /* These would be set in reconstructCommon */
 306:                             false, Align(1));
 307:   reconstructCommon(Call, GEP, Load, 0);
 308:   return std::pair{GEP, Load};
 309: }
 310: 
 311: std::pair<GetElementPtrInst *, StoreInst *>
 312: BPFPreserveStaticOffsetPass::reconstructStore(CallInst *Call) {
 313:   GetElementPtrInst *GEP = reconstructGEP(Call, 1);
 314:   auto *Store = new StoreInst(Call->getOperand(0), GEP,
 315:                               /* These would be set in reconstructCommon */
 316:                               false, Align(1));
 317:   reconstructCommon(Call, GEP, Store, 1);
 318:   return std::pair{GEP, Store};
 319: }
 320: 
```

- EN: This range continues the implementation of the backend component described by BPFPreserveStaticOffset.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 321-340

```cpp
 321: static bool isZero(Value *V) {
 322:   auto *CI = dyn_cast<ConstantInt>(V);
 323:   return CI && CI->isZero();
 324: }
 325: 
 326: // Given a chain of GEP instructions collect information necessary to
 327: // merge this chain as a single GEP instruction of form:
 328: //   getelementptr %<type>, ptr %p, i32 0, <field_idx1>, <field_idx2>, ...
 329: static bool foldGEPChainAsStructAccess(SmallVector<GetElementPtrInst *> &GEPs,
 330:                                        GEPChainInfo &Info) {
 331:   if (GEPs.empty())
 332:     return false;
 333: 
 334:   if (!all_of(GEPs, [=](GetElementPtrInst *GEP) {
 335:         return GEP->hasAllConstantIndices();
 336:       }))
 337:     return false;
 338: 
 339:   GetElementPtrInst *First = GEPs[0];
 340:   Info.InBounds = First->isInBounds();
```

- EN: Function bodies or method definitions such as isZero, foldGEPChainAsStructAccess contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: isZero, foldGEPChainAsStructAccess 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 341-360

```cpp
 341:   Info.SourceElementType = First->getSourceElementType();
 342:   Type *ResultElementType = First->getResultElementType();
 343:   Info.Indices.append(First->idx_begin(), First->idx_end());
 344:   Info.Members.push_back(First);
 345: 
 346:   for (GetElementPtrInst *GEP : drop_begin(GEPs)) {
 347:     if (!isZero(*GEP->idx_begin())) {
 348:       Info.reset();
 349:       return false;
 350:     }
 351:     if (!GEP->getSourceElementType() ||
 352:         GEP->getSourceElementType() != ResultElementType) {
 353:       Info.reset();
 354:       return false;
 355:     }
 356:     Info.InBounds &= GEP->isInBounds();
 357:     Info.Indices.append(GEP->idx_begin() + 1, GEP->idx_end());
 358:     Info.Members.push_back(GEP);
 359:     ResultElementType = GEP->getResultElementType();
 360:   }
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 361-380

```cpp
 361: 
 362:   return true;
 363: }
 364: 
 365: // Given a chain of GEP instructions collect information necessary to
 366: // merge this chain as a single GEP instruction of form:
 367: //   getelementptr i8, ptr %p, i64 %offset
 368: static bool foldGEPChainAsU8Access(SmallVector<GetElementPtrInst *> &GEPs,
 369:                                    GEPChainInfo &Info) {
 370:   if (GEPs.empty())
 371:     return false;
 372: 
 373:   GetElementPtrInst *First = GEPs[0];
 374:   const DataLayout &DL = First->getDataLayout();
 375:   LLVMContext &C = First->getContext();
 376:   Type *PtrTy = First->getType()->getScalarType();
 377:   APInt Offset(DL.getIndexTypeSizeInBits(PtrTy), 0);
 378:   for (GetElementPtrInst *GEP : GEPs) {
 379:     if (!GEP->accumulateConstantOffset(DL, Offset)) {
 380:       Info.reset();
```

- EN: Function bodies or method definitions such as foldGEPChainAsU8Access contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: foldGEPChainAsU8Access 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 381-400

```cpp
 381:       return false;
 382:     }
 383:     Info.InBounds &= GEP->isInBounds();
 384:     Info.Members.push_back(GEP);
 385:   }
 386:   Info.SourceElementType = Type::getInt8Ty(C);
 387:   Info.Indices.push_back(ConstantInt::get(C, Offset));
 388: 
 389:   return true;
 390: }
 391: 
 392: static void reportNonStaticGEPChain(Instruction *Insn) {
 393:   Insn->getContext().diagnose(DiagnosticInfoUnsupported(
 394:       *Insn->getFunction(),
 395:       Twine("Non-constant offset in access to a field of a type marked "
 396:             "with preserve_static_offset might be rejected by BPF verifier")
 397:           .concat(Insn->getDebugLoc()
 398:                       ? ""
 399:                       : " (pass -g option to get exact location)"),
 400:       Insn->getDebugLoc(), DS_Warning));
```

- EN: Function bodies or method definitions such as reportNonStaticGEPChain contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: reportNonStaticGEPChain 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 401-420

```cpp
 401: }
 402: 
 403: static bool allZeroIndices(SmallVector<GetElementPtrInst *> &GEPs) {
 404:   return GEPs.empty() || all_of(GEPs, [=](GetElementPtrInst *GEP) {
 405:            return GEP->hasAllZeroIndices();
 406:          });
 407: }
 408: 
 409: static bool tryToReplaceWithGEPBuiltin(Instruction *LoadOrStoreTemplate,
 410:                                        SmallVector<GetElementPtrInst *> &GEPs,
 411:                                        Instruction *InsnToReplace) {
 412:   GEPChainInfo GEPChain;
 413:   if (!foldGEPChainAsStructAccess(GEPs, GEPChain) &&
 414:       !foldGEPChainAsU8Access(GEPs, GEPChain)) {
 415:     return false;
 416:   }
 417:   Module *M = InsnToReplace->getModule();
 418:   if (auto *Load = dyn_cast<LoadInst>(LoadOrStoreTemplate)) {
 419:     Instruction *Replacement = makeGEPAndLoad(M, GEPChain, Load);
 420:     Replacement->insertBefore(InsnToReplace->getIterator());
```

- EN: Function bodies or method definitions such as allZeroIndices, tryToReplaceWithGEPBuiltin contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: allZeroIndices, tryToReplaceWithGEPBuiltin 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 421-440

```cpp
 421:     InsnToReplace->replaceAllUsesWith(Replacement);
 422:   }
 423:   if (auto *Store = dyn_cast<StoreInst>(LoadOrStoreTemplate)) {
 424:     Instruction *Replacement = makeGEPAndStore(M, GEPChain, Store);
 425:     Replacement->insertBefore(InsnToReplace->getIterator());
 426:   }
 427:   return true;
 428: }
 429: 
 430: // Check if U->getPointerOperand() == I
 431: static bool isPointerOperand(Value *I, User *U) {
 432:   if (auto *L = dyn_cast<LoadInst>(U))
 433:     return L->getPointerOperand() == I;
 434:   if (auto *S = dyn_cast<StoreInst>(U))
 435:     return S->getPointerOperand() == I;
 436:   if (auto *GEP = dyn_cast<GetElementPtrInst>(U))
 437:     return GEP->getPointerOperand() == I;
 438:   if (auto *Call = isGEPAndLoad(U))
 439:     return Call->getArgOperand(0) == I;
 440:   if (auto *Call = isGEPAndStore(U))
```

- EN: Function bodies or method definitions such as isPointerOperand contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: isPointerOperand 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 441-460

```cpp
 441:     return Call->getArgOperand(1) == I;
 442:   return false;
 443: }
 444: 
 445: static bool isInlineableCall(User *U) {
 446:   if (auto *Call = dyn_cast<CallInst>(U))
 447:     return Call->hasFnAttr(Attribute::InlineHint);
 448:   return false;
 449: }
 450: 
 451: static void rewriteAccessChain(Instruction *Insn,
 452:                                SmallVector<GetElementPtrInst *> &GEPs,
 453:                                SmallVector<Instruction *> &Visited,
 454:                                bool AllowPatial, bool &StillUsed);
 455: 
 456: static void rewriteUses(Instruction *Insn,
 457:                         SmallVector<GetElementPtrInst *> &GEPs,
 458:                         SmallVector<Instruction *> &Visited, bool AllowPatial,
 459:                         bool &StillUsed) {
 460:   for (User *U : Insn->users()) {
```

- EN: Function bodies or method definitions such as isInlineableCall, rewriteUses contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: isInlineableCall, rewriteUses 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 461-480

```cpp
 461:     auto *UI = dyn_cast<Instruction>(U);
 462:     if (UI && (isPointerOperand(Insn, UI) || isPreserveStaticOffsetCall(UI) ||
 463:                isInlineableCall(UI)))
 464:       rewriteAccessChain(UI, GEPs, Visited, AllowPatial, StillUsed);
 465:     else
 466:       LLVM_DEBUG({
 467:         llvm::dbgs() << "unsupported usage in BPFPreserveStaticOffsetPass:\n";
 468:         llvm::dbgs() << "  Insn: " << *Insn << "\n";
 469:         llvm::dbgs() << "  User: " << *U << "\n";
 470:       });
 471:   }
 472: }
 473: 
 474: // A DFS traversal of GEP chain trees starting from Root.
 475: //
 476: // Recursion descends through GEP instructions and
 477: // llvm.preserve.static.offset calls. Recursion stops at any other
 478: // instruction. If load or store instruction is reached it is replaced
 479: // by a call to `llvm.bpf.getelementptr.and.load` or
 480: // `llvm.bpf.getelementptr.and.store` intrinsic.
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 481-500

```cpp
 481: // If `llvm.bpf.getelementptr.and.load/store` is reached the accumulated
 482: // GEPs are merged into the intrinsic call.
 483: // If nested calls to `llvm.preserve.static.offset` are encountered these
 484: // calls are marked for deletion.
 485: //
 486: // Parameters description:
 487: // - Insn - current position in the tree
 488: // - GEPs - GEP instructions for the current branch
 489: // - Visited - a list of visited instructions in DFS order,
 490: //   order is important for unused instruction deletion.
 491: // - AllowPartial - when true GEP chains that can't be folded are
 492: //   not reported, otherwise diagnostic message is show for such chains.
 493: // - StillUsed - set to true if one of the GEP chains could not be
 494: //   folded, makes sense when AllowPartial is false, means that root
 495: //   preserve.static.offset call is still in use and should remain
 496: //   until the next run of this pass.
 497: static void rewriteAccessChain(Instruction *Insn,
 498:                                SmallVector<GetElementPtrInst *> &GEPs,
 499:                                SmallVector<Instruction *> &Visited,
 500:                                bool AllowPatial, bool &StillUsed) {
```

- EN: Function bodies or method definitions such as rewriteAccessChain contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: rewriteAccessChain 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 501-520

```cpp
 501:   auto MarkAndTraverseUses = [&]() {
 502:     Visited.push_back(Insn);
 503:     rewriteUses(Insn, GEPs, Visited, AllowPatial, StillUsed);
 504:   };
 505:   auto TryToReplace = [&](Instruction *LoadOrStore) {
 506:     // Do nothing for (preserve.static.offset (load/store ..)) or for
 507:     // GEPs with zero indices. Such constructs lead to zero offset and
 508:     // are simplified by other passes.
 509:     if (allZeroIndices(GEPs))
 510:       return;
 511:     if (tryToReplaceWithGEPBuiltin(LoadOrStore, GEPs, Insn)) {
 512:       Visited.push_back(Insn);
 513:       return;
 514:     }
 515:     if (!AllowPatial)
 516:       reportNonStaticGEPChain(Insn);
 517:     StillUsed = true;
 518:   };
 519:   if (isa<LoadInst>(Insn) || isa<StoreInst>(Insn)) {
 520:     TryToReplace(Insn);
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 521-540

```cpp
 521:   } else if (isGEPAndLoad(Insn)) {
 522:     auto [GEP, Load] =
 523:         BPFPreserveStaticOffsetPass::reconstructLoad(cast<CallInst>(Insn));
 524:     GEPs.push_back(GEP);
 525:     TryToReplace(Load);
 526:     GEPs.pop_back();
 527:     delete Load;
 528:     delete GEP;
 529:   } else if (isGEPAndStore(Insn)) {
 530:     // This  case can't be merged with the above because
 531:     // `delete Load` / `delete Store` wants a concrete type,
 532:     // destructor of Instruction is protected.
 533:     auto [GEP, Store] =
 534:         BPFPreserveStaticOffsetPass::reconstructStore(cast<CallInst>(Insn));
 535:     GEPs.push_back(GEP);
 536:     TryToReplace(Store);
 537:     GEPs.pop_back();
 538:     delete Store;
 539:     delete GEP;
 540:   } else if (auto *GEP = dyn_cast<GetElementPtrInst>(Insn)) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 541-560

```cpp
 541:     GEPs.push_back(GEP);
 542:     MarkAndTraverseUses();
 543:     GEPs.pop_back();
 544:   } else if (isPreserveStaticOffsetCall(Insn)) {
 545:     MarkAndTraverseUses();
 546:   } else if (isInlineableCall(Insn)) {
 547:     // Preserve preserve.static.offset call for parameters of
 548:     // functions that might be inlined. These would be removed on a
 549:     // second pass after inlining.
 550:     // Might happen when a pointer to a preserve_static_offset
 551:     // structure is passed as parameter of a function that would be
 552:     // inlined inside a loop that would be unrolled.
 553:     if (AllowPatial)
 554:       StillUsed = true;
 555:   } else {
 556:     SmallString<128> Buf;
 557:     raw_svector_ostream BufStream(Buf);
 558:     BufStream << *Insn;
 559:     report_fatal_error(
 560:         Twine("Unexpected rewriteAccessChain Insn = ").concat(Buf));
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 561-580

```cpp
 561:   }
 562: }
 563: 
 564: static void removeMarkerCall(Instruction *Marker) {
 565:   Marker->replaceAllUsesWith(Marker->getOperand(0));
 566:   Marker->eraseFromParent();
 567: }
 568: 
 569: static bool rewriteAccessChain(Instruction *Marker, bool AllowPatial,
 570:                                SmallPtrSetImpl<Instruction *> &RemovedMarkers) {
 571:   SmallVector<GetElementPtrInst *> GEPs;
 572:   SmallVector<Instruction *> Visited;
 573:   bool StillUsed = false;
 574:   rewriteUses(Marker, GEPs, Visited, AllowPatial, StillUsed);
 575:   // Check if Visited instructions could be removed, iterate in
 576:   // reverse to unblock instructions higher in the chain.
 577:   for (auto V = Visited.rbegin(); V != Visited.rend(); ++V) {
 578:     if (isPreserveStaticOffsetCall(*V)) {
 579:       removeMarkerCall(*V);
 580:       RemovedMarkers.insert(*V);
```

- EN: Function bodies or method definitions such as removeMarkerCall, rewriteAccessChain contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: removeMarkerCall, rewriteAccessChain 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 581-600

```cpp
 581:     } else if ((*V)->use_empty()) {
 582:       (*V)->eraseFromParent();
 583:     }
 584:   }
 585:   return StillUsed;
 586: }
 587: 
 588: static std::vector<Instruction *>
 589: collectPreserveStaticOffsetCalls(Function &F) {
 590:   std::vector<Instruction *> Calls;
 591:   for (Instruction &Insn : instructions(F))
 592:     if (isPreserveStaticOffsetCall(&Insn))
 593:       Calls.push_back(&Insn);
 594:   return Calls;
 595: }
 596: 
 597: bool isPreserveArrayIndex(Value *V) {
 598:   return isIntrinsicCall(V, Intrinsic::preserve_array_access_index);
 599: }
 600: 
```

- EN: Function bodies or method definitions such as collectPreserveStaticOffsetCalls, isPreserveArrayIndex contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: collectPreserveStaticOffsetCalls, isPreserveArrayIndex 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 601-620

```cpp
 601: bool isPreserveStructIndex(Value *V) {
 602:   return isIntrinsicCall(V, Intrinsic::preserve_struct_access_index);
 603: }
 604: 
 605: bool isPreserveUnionIndex(Value *V) {
 606:   return isIntrinsicCall(V, Intrinsic::preserve_union_access_index);
 607: }
 608: 
 609: static void removePAICalls(Instruction *Marker) {
 610:   auto IsPointerOperand = [](Value *Op, User *U) {
 611:     if (auto *GEP = dyn_cast<GetElementPtrInst>(U))
 612:       return GEP->getPointerOperand() == Op;
 613:     if (isPreserveStaticOffsetCall(U) || isPreserveArrayIndex(U) ||
 614:         isPreserveStructIndex(U) || isPreserveUnionIndex(U))
 615:       return cast<CallInst>(U)->getArgOperand(0) == Op;
 616:     return false;
 617:   };
 618: 
 619:   SmallVector<Value *, 32> WorkList;
 620:   WorkList.push_back(Marker);
```

- EN: Function bodies or method definitions such as isPreserveStructIndex, isPreserveUnionIndex, removePAICalls contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: isPreserveStructIndex, isPreserveUnionIndex, removePAICalls 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 621-640

```cpp
 621:   do {
 622:     Value *V = WorkList.pop_back_val();
 623:     for (User *U : V->users())
 624:       if (IsPointerOperand(V, U))
 625:         WorkList.push_back(U);
 626:     auto *Call = dyn_cast<CallInst>(V);
 627:     if (!Call)
 628:       continue;
 629:     if (isPreserveArrayIndex(V))
 630:       BPFCoreSharedInfo::removeArrayAccessCall(Call);
 631:     else if (isPreserveStructIndex(V))
 632:       BPFCoreSharedInfo::removeStructAccessCall(Call);
 633:     else if (isPreserveUnionIndex(V))
 634:       BPFCoreSharedInfo::removeUnionAccessCall(Call);
 635:   } while (!WorkList.empty());
 636: }
 637: 
 638: // Look for sequences:
 639: // - llvm.preserve.static.offset -> getelementptr... -> load
 640: // - llvm.preserve.static.offset -> getelementptr... -> store
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 641-660

```cpp
 641: // And replace those with calls to intrinsics:
 642: // - llvm.bpf.getelementptr.and.load
 643: // - llvm.bpf.getelementptr.and.store
 644: static bool rewriteFunction(Function &F, bool AllowPartial) {
 645:   LLVM_DEBUG(dbgs() << "********** BPFPreserveStaticOffsetPass (AllowPartial="
 646:                     << AllowPartial << ") ************\n");
 647: 
 648:   auto MarkerCalls = collectPreserveStaticOffsetCalls(F);
 649:   SmallPtrSet<Instruction *, 16> RemovedMarkers;
 650: 
 651:   LLVM_DEBUG(dbgs() << "There are " << MarkerCalls.size()
 652:                     << " preserve.static.offset calls\n");
 653: 
 654:   if (MarkerCalls.empty())
 655:     return false;
 656: 
 657:   for (auto *Call : MarkerCalls)
 658:     removePAICalls(Call);
 659: 
 660:   for (auto *Call : MarkerCalls) {
```

- EN: Function bodies or method definitions such as rewriteFunction contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: rewriteFunction 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 661-676

```cpp
 661:     if (RemovedMarkers.contains(Call))
 662:       continue;
 663:     bool StillUsed = rewriteAccessChain(Call, AllowPartial, RemovedMarkers);
 664:     if (!StillUsed || !AllowPartial)
 665:       removeMarkerCall(Call);
 666:   }
 667: 
 668:   return true;
 669: }
 670: 
 671: PreservedAnalyses
 672: llvm::BPFPreserveStaticOffsetPass::run(Function &F,
 673:                                        FunctionAnalysisManager &AM) {
 674:   return rewriteFunction(F, AllowPartial) ? PreservedAnalyses::none()
 675:                                           : PreservedAnalyses::all();
 676: }
```

- EN: Function bodies or method definitions such as run contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: run 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

## Key Concepts / 关键概念

- Target backend support code / 目标后端支持代码
- LLVM CodeGen integration / LLVM CodeGen 集成

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BPF.h`, `BPFCORE.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/IR/Attributes.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/Constants.h`, `llvm/IR/DiagnosticInfo.h`
