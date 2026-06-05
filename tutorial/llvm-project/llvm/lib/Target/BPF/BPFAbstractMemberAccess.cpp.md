# BPFAbstractMemberAccess.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `llvm/lib/Target/BPF/BPFAbstractMemberAccess.cpp`
- Repository: `/root/xw/llvm-project`
- Purpose (EN): Implements target-specific rewrites that recognize and simplify abstract member accesses.
- 目的（中文）: 实现目标专用重写逻辑，用于识别并简化抽象成员访问。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20

```cpp
   1: //===------ BPFAbstractMemberAccess.cpp - Abstracting Member Accesses -----===//
   2: //
   3: // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
   4: // See https://llvm.org/LICENSE.txt for license information.
   5: // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
   6: //
   7: //===----------------------------------------------------------------------===//
   8: //
   9: // This pass abstracted struct/union member accesses in order to support
  10: // compile-once run-everywhere (CO-RE). The CO-RE intends to compile the program
  11: // which can run on different kernels. In particular, if bpf program tries to
  12: // access a particular kernel data structure member, the details of the
  13: // intermediate member access will be remembered so bpf loader can do
  14: // necessary adjustment right before program loading.
  15: //
  16: // For example,
  17: //
  18: //   struct s {
  19: //     int a;
  20: //     int b;
```

- EN: This opening chunk establishes the file context, including comments, includes, and the first declarations that set up the backend component. This chunk introduces interfaces or data structures such as s, which organize the target-specific behavior exposed by the file.
- 中文: 这一开头部分建立了文件上下文，包括注释、头文件包含以及用于搭建后端组件的首批声明。 这一段引入了 s 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 21-40

```cpp
  21: //   };
  22: //   struct t {
  23: //     struct s c;
  24: //     int d;
  25: //   };
  26: //   struct t e;
  27: //
  28: // For the member access e.c.b, the compiler will generate code
  29: //   &e + 4
  30: //
  31: // The compile-once run-everywhere instead generates the following code
  32: //   r = 4
  33: //   &e + r
  34: // The "4" in "r = 4" can be changed based on a particular kernel version.
  35: // For example, on a particular kernel version, if struct s is changed to
  36: //
  37: //   struct s {
  38: //     int new_field;
  39: //     int a;
  40: //     int b;
```

- EN: This chunk introduces interfaces or data structures such as t, s, which organize the target-specific behavior exposed by the file.
- 中文: 这一段引入了 t, s 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 41-60

```cpp
  41: //   }
  42: //
  43: // By repeating the member access on the host, the bpf loader can
  44: // adjust "r = 4" as "r = 8".
  45: //
  46: // This feature relies on the following three intrinsic calls:
  47: //   addr = preserve_array_access_index(base, dimension, index)
  48: //   addr = preserve_union_access_index(base, di_index)
  49: //          !llvm.preserve.access.index <union_ditype>
  50: //   addr = preserve_struct_access_index(base, gep_index, di_index)
  51: //          !llvm.preserve.access.index <struct_ditype>
  52: //
  53: // Bitfield member access needs special attention. User cannot take the
  54: // address of a bitfield acceess. To facilitate kernel verifier
  55: // for easy bitfield code optimization, a new clang intrinsic is introduced:
  56: //   uint32_t __builtin_preserve_field_info(member_access, info_kind)
  57: // In IR, a chain with two (or more) intrinsic calls will be generated:
  58: //   ...
  59: //   addr = preserve_struct_access_index(base, 1, 1) !struct s
  60: //   uint32_t result = bpf_preserve_field_info(addr, info_kind)
```

- EN: This chunk introduces interfaces or data structures such as s, which organize the target-specific behavior exposed by the file.
- 中文: 这一段引入了 s 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 61-80

```cpp
  61: //
  62: // Suppose the info_kind is FIELD_SIGNEDNESS,
  63: // The above two IR intrinsics will be replaced with
  64: // a relocatable insn:
  65: //   signness = /* signness of member_access */
  66: // and signness can be changed by bpf loader based on the
  67: // types on the host.
  68: //
  69: // User can also test whether a field exists or not with
  70: //   uint32_t result = bpf_preserve_field_info(member_access, FIELD_EXISTENCE)
  71: // The field will be always available (result = 1) during initial
  72: // compilation, but bpf loader can patch with the correct value
  73: // on the target host where the member_access may or may not be available
  74: //
  75: //===----------------------------------------------------------------------===//
  76: 
  77: #include "BPF.h"
  78: #include "BPFCORE.h"
  79: #include "BPFTargetMachine.h"
  80: #include "llvm/BinaryFormat/Dwarf.h"
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。

### Lines 81-100

```cpp
  81: #include "llvm/DebugInfo/BTF/BTF.h"
  82: #include "llvm/IR/DebugInfoMetadata.h"
  83: #include "llvm/IR/GlobalVariable.h"
  84: #include "llvm/IR/Instruction.h"
  85: #include "llvm/IR/Instructions.h"
  86: #include "llvm/IR/IntrinsicsBPF.h"
  87: #include "llvm/IR/Module.h"
  88: #include "llvm/IR/PassManager.h"
  89: #include "llvm/IR/Type.h"
  90: #include "llvm/IR/User.h"
  91: #include "llvm/IR/Value.h"
  92: #include "llvm/IR/ValueHandle.h"
  93: #include "llvm/Pass.h"
  94: #include "llvm/Transforms/Utils/BasicBlockUtils.h"
  95: #include <stack>
  96: 
  97: #define DEBUG_TYPE "bpf-abstract-member-access"
  98: 
  99: namespace llvm {
 100: uint32_t BPFCoreSharedInfo::SeqNum;
```

- EN: The included headers pull in LLVM core, CodeGen, MC, and sibling target declarations needed by this compilation unit. The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure.
- 中文: 这些头文件引入了 LLVM 核心、CodeGen、MC 以及同目标后端的相关声明，供当前编译单元使用。 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。

### Lines 101-120

```cpp
 101: 
 102: Instruction *BPFCoreSharedInfo::insertPassThrough(Module *M, BasicBlock *BB,
 103:                                                   Instruction *Input,
 104:                                                   Instruction *Before) {
 105:   Function *Fn = Intrinsic::getOrInsertDeclaration(
 106:       M, Intrinsic::bpf_passthrough, {Input->getType(), Input->getType()});
 107:   Constant *SeqNumVal = ConstantInt::get(Type::getInt32Ty(BB->getContext()),
 108:                                          BPFCoreSharedInfo::SeqNum++);
 109: 
 110:   auto *NewInst = CallInst::Create(Fn, {SeqNumVal, Input});
 111:   NewInst->insertBefore(Before->getIterator());
 112:   return NewInst;
 113: }
 114: } // namespace llvm
 115: 
 116: using namespace llvm;
 117: 
 118: namespace {
 119: class BPFAbstractMemberAccess final {
 120: public:
```

- EN: The code enters the LLVM namespace, placing the target implementation into LLVM’s standard backend structure. This chunk introduces interfaces or data structures such as BPFAbstractMemberAccess, which organize the target-specific behavior exposed by the file.
- 中文: 代码进入 LLVM 命名空间，使该目标实现位于 LLVM 标准后端结构之中。 这一段引入了 BPFAbstractMemberAccess 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 121-140

```cpp
 121:   BPFAbstractMemberAccess(BPFTargetMachine *TM) : TM(TM) {}
 122: 
 123:   bool run(Function &F);
 124: 
 125:   struct CallInfo {
 126:     uint32_t Kind;
 127:     uint32_t AccessIndex;
 128:     MaybeAlign RecordAlignment;
 129:     MDNode *Metadata;
 130:     WeakTrackingVH Base;
 131:   };
 132:   typedef std::stack<std::pair<CallInst *, CallInfo>> CallInfoStack;
 133: 
 134: private:
 135:   enum : uint32_t {
 136:     BPFPreserveArrayAI = 1,
 137:     BPFPreserveUnionAI = 2,
 138:     BPFPreserveStructAI = 3,
 139:     BPFPreserveFieldInfoAI = 4,
 140:   };
```

- EN: This chunk introduces interfaces or data structures such as CallInfo, which organize the target-specific behavior exposed by the file.
- 中文: 这一段引入了 CallInfo 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 141-160

```cpp
 141: 
 142:   TargetMachine *TM;
 143:   const DataLayout *DL = nullptr;
 144:   Module *M = nullptr;
 145: 
 146:   static std::map<std::string, GlobalVariable *> GEPGlobals;
 147:   // A map to link preserve_*_access_index intrinsic calls.
 148:   std::map<CallInst *, std::pair<CallInst *, CallInfo>> AIChain;
 149:   // A map to hold all the base preserve_*_access_index intrinsic calls.
 150:   // The base call is not an input of any other preserve_*
 151:   // intrinsics.
 152:   std::map<CallInst *, CallInfo> BaseAICalls;
 153:   // A map to hold <AnonRecord, TypeDef> relationships
 154:   std::map<DICompositeType *, DIDerivedType *> AnonRecords;
 155: 
 156:   void CheckAnonRecordType(DIDerivedType *ParentTy, DIType *Ty);
 157:   void CheckCompositeType(DIDerivedType *ParentTy, DICompositeType *CTy);
 158:   void CheckDerivedType(DIDerivedType *ParentTy, DIDerivedType *DTy);
 159:   void ResetMetadata(struct CallInfo &CInfo);
 160: 
```

- EN: This chunk introduces interfaces or data structures such as CallInfo, which organize the target-specific behavior exposed by the file.
- 中文: 这一段引入了 CallInfo 等接口或数据结构，用于组织该文件暴露的目标专用行为。

### Lines 161-180

```cpp
 161:   bool doTransformation(Function &F);
 162: 
 163:   void traceAICall(CallInst *Call, CallInfo &ParentInfo);
 164:   void traceBitCast(BitCastInst *BitCast, CallInst *Parent,
 165:                     CallInfo &ParentInfo);
 166:   void traceGEP(GetElementPtrInst *GEP, CallInst *Parent,
 167:                 CallInfo &ParentInfo);
 168:   void collectAICallChains(Function &F);
 169: 
 170:   bool IsPreserveDIAccessIndexCall(const CallInst *Call, CallInfo &Cinfo);
 171:   bool IsValidAIChain(const MDNode *ParentMeta, uint32_t ParentAI,
 172:                       const MDNode *ChildMeta);
 173:   bool removePreserveAccessIndexIntrinsic(Function &F);
 174:   bool HasPreserveFieldInfoCall(CallInfoStack &CallStack);
 175:   void GetStorageBitRange(DIDerivedType *MemberTy, Align RecordAlignment,
 176:                           uint32_t &StartBitOffset, uint32_t &EndBitOffset);
 177:   uint32_t GetFieldInfo(uint32_t InfoKind, DICompositeType *CTy,
 178:                         uint32_t AccessIndex, uint32_t PatchImm,
 179:                         MaybeAlign RecordAlignment);
 180: 
```

- EN: This range continues the implementation of the backend component described by BPFAbstractMemberAccess.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 181-200

```cpp
 181:   Value *computeBaseAndAccessKey(CallInst *Call, CallInfo &CInfo,
 182:                                  std::string &AccessKey, MDNode *&BaseMeta);
 183:   MDNode *computeAccessKey(CallInst *Call, CallInfo &CInfo,
 184:                            std::string &AccessKey, bool &IsInt32Ret);
 185:   bool transformGEPChain(CallInst *Call, CallInfo &CInfo);
 186: };
 187: 
 188: std::map<std::string, GlobalVariable *> BPFAbstractMemberAccess::GEPGlobals;
 189: } // End anonymous namespace
 190: 
 191: bool BPFAbstractMemberAccess::run(Function &F) {
 192:   LLVM_DEBUG(dbgs() << "********** Abstract Member Accesses **********\n");
 193: 
 194:   M = F.getParent();
 195:   if (!M)
 196:     return false;
 197: 
 198:   // Bail out if no debug info.
 199:   if (M->debug_compile_units().empty())
 200:     return false;
```

- EN: Function bodies or method definitions such as run contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: run 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 201-220

```cpp
 201: 
 202:   // For each argument/return/local_variable type, trace the type
 203:   // pattern like '[derived_type]* [composite_type]' to check
 204:   // and remember (anon record -> typedef) relations where the
 205:   // anon record is defined as
 206:   //   typedef [const/volatile/restrict]* [anon record]
 207:   DISubprogram *SP = F.getSubprogram();
 208:   if (SP && SP->isDefinition()) {
 209:     for (DIType *Ty: SP->getType()->getTypeArray())
 210:       CheckAnonRecordType(nullptr, Ty);
 211:     for (const DINode *DN : SP->getRetainedNodes()) {
 212:       if (const auto *DV = dyn_cast<DILocalVariable>(DN))
 213:         CheckAnonRecordType(nullptr, DV->getType());
 214:     }
 215:   }
 216: 
 217:   DL = &M->getDataLayout();
 218:   return doTransformation(F);
 219: }
 220: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 221-240

```cpp
 221: void BPFAbstractMemberAccess::ResetMetadata(struct CallInfo &CInfo) {
 222:   if (auto Ty = dyn_cast<DICompositeType>(CInfo.Metadata)) {
 223:     auto It = AnonRecords.find(Ty);
 224:     if (It != AnonRecords.end() && It->second != nullptr)
 225:       CInfo.Metadata = It->second;
 226:   }
 227: }
 228: 
 229: void BPFAbstractMemberAccess::CheckCompositeType(DIDerivedType *ParentTy,
 230:                                                  DICompositeType *CTy) {
 231:   if (!CTy->getName().empty() || !ParentTy ||
 232:       ParentTy->getTag() != dwarf::DW_TAG_typedef)
 233:     return;
 234: 
 235:   auto [It, Inserted] = AnonRecords.try_emplace(CTy, ParentTy);
 236:   // Two or more typedef's may point to the same anon record.
 237:   // If this is the case, set the typedef DIType to be nullptr
 238:   // to indicate the duplication case.
 239:   if (!Inserted && It->second != ParentTy)
 240:     It->second = nullptr;
```

- EN: This chunk introduces interfaces or data structures such as CallInfo, which organize the target-specific behavior exposed by the file. Function bodies or method definitions such as ResetMetadata, CheckCompositeType contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这一段引入了 CallInfo 等接口或数据结构，用于组织该文件暴露的目标专用行为。 ResetMetadata, CheckCompositeType 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 241-260

```cpp
 241: }
 242: 
 243: void BPFAbstractMemberAccess::CheckDerivedType(DIDerivedType *ParentTy,
 244:                                                DIDerivedType *DTy) {
 245:   DIType *BaseType = DTy->getBaseType();
 246:   if (!BaseType)
 247:     return;
 248: 
 249:   unsigned Tag = DTy->getTag();
 250:   if (Tag == dwarf::DW_TAG_pointer_type)
 251:     CheckAnonRecordType(nullptr, BaseType);
 252:   else if (Tag == dwarf::DW_TAG_typedef)
 253:     CheckAnonRecordType(DTy, BaseType);
 254:   else
 255:     CheckAnonRecordType(ParentTy, BaseType);
 256: }
 257: 
 258: void BPFAbstractMemberAccess::CheckAnonRecordType(DIDerivedType *ParentTy,
 259:                                                   DIType *Ty) {
 260:   if (!Ty)
```

- EN: Function bodies or method definitions such as CheckDerivedType, CheckAnonRecordType contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: CheckDerivedType, CheckAnonRecordType 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 261-280

```cpp
 261:     return;
 262: 
 263:   if (auto *CTy = dyn_cast<DICompositeType>(Ty))
 264:     return CheckCompositeType(ParentTy, CTy);
 265:   else if (auto *DTy = dyn_cast<DIDerivedType>(Ty))
 266:     return CheckDerivedType(ParentTy, DTy);
 267: }
 268: 
 269: static bool SkipDIDerivedTag(unsigned Tag, bool skipTypedef) {
 270:   if (Tag != dwarf::DW_TAG_typedef && Tag != dwarf::DW_TAG_const_type &&
 271:       Tag != dwarf::DW_TAG_volatile_type &&
 272:       Tag != dwarf::DW_TAG_restrict_type &&
 273:       Tag != dwarf::DW_TAG_member)
 274:     return false;
 275:   if (Tag == dwarf::DW_TAG_typedef && !skipTypedef)
 276:     return false;
 277:   return true;
 278: }
 279: 
 280: static DIType * stripQualifiers(DIType *Ty, bool skipTypedef = true) {
```

- EN: Function bodies or method definitions such as SkipDIDerivedTag, stripQualifiers contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: SkipDIDerivedTag, stripQualifiers 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 281-300

```cpp
 281:   while (auto *DTy = dyn_cast<DIDerivedType>(Ty)) {
 282:     if (!SkipDIDerivedTag(DTy->getTag(), skipTypedef))
 283:       break;
 284:     Ty = DTy->getBaseType();
 285:   }
 286:   return Ty;
 287: }
 288: 
 289: static const DIType * stripQualifiers(const DIType *Ty) {
 290:   while (auto *DTy = dyn_cast<DIDerivedType>(Ty)) {
 291:     if (!SkipDIDerivedTag(DTy->getTag(), true))
 292:       break;
 293:     Ty = DTy->getBaseType();
 294:   }
 295:   return Ty;
 296: }
 297: 
 298: static uint32_t calcArraySize(const DICompositeType *CTy, uint32_t StartDim) {
 299:   DINodeArray Elements = CTy->getElements();
 300:   uint32_t DimSize = 1;
```

- EN: Function bodies or method definitions such as stripQualifiers, calcArraySize contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: stripQualifiers, calcArraySize 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 301-320

```cpp
 301:   for (uint32_t I = StartDim; I < Elements.size(); ++I) {
 302:     if (auto *Element = dyn_cast_or_null<DINode>(Elements[I]))
 303:       if (Element->getTag() == dwarf::DW_TAG_subrange_type) {
 304:         const DISubrange *SR = cast<DISubrange>(Element);
 305:         auto *CI = dyn_cast<ConstantInt *>(SR->getCount());
 306:         DimSize *= CI->getSExtValue();
 307:       }
 308:   }
 309: 
 310:   return DimSize;
 311: }
 312: 
 313: static Type *getBaseElementType(const CallInst *Call) {
 314:   // Element type is stored in an elementtype() attribute on the first param.
 315:   return Call->getParamElementType(0);
 316: }
 317: 
 318: static uint64_t getConstant(const Value *IndexValue) {
 319:   const ConstantInt *CV = dyn_cast<ConstantInt>(IndexValue);
 320:   assert(CV);
```

- EN: Function bodies or method definitions such as getConstant contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: getConstant 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 321-340

```cpp
 321:   return CV->getValue().getZExtValue();
 322: }
 323: 
 324: /// Check whether a call is a preserve_*_access_index intrinsic call or not.
 325: bool BPFAbstractMemberAccess::IsPreserveDIAccessIndexCall(const CallInst *Call,
 326:                                                           CallInfo &CInfo) {
 327:   if (!Call)
 328:     return false;
 329: 
 330:   const auto *GV = dyn_cast<GlobalValue>(Call->getCalledOperand());
 331:   if (!GV)
 332:     return false;
 333:   if (GV->getName().starts_with("llvm.preserve.array.access.index")) {
 334:     CInfo.Kind = BPFPreserveArrayAI;
 335:     CInfo.Metadata = Call->getMetadata(LLVMContext::MD_preserve_access_index);
 336:     if (!CInfo.Metadata)
 337:       report_fatal_error("Missing metadata for llvm.preserve.array.access.index intrinsic");
 338:     CInfo.AccessIndex = getConstant(Call->getArgOperand(2));
 339:     CInfo.Base = Call->getArgOperand(0);
 340:     CInfo.RecordAlignment = DL->getABITypeAlign(getBaseElementType(Call));
```

- EN: Function bodies or method definitions such as IsPreserveDIAccessIndexCall contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: IsPreserveDIAccessIndexCall 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 341-360

```cpp
 341:     return true;
 342:   }
 343:   if (GV->getName().starts_with("llvm.preserve.union.access.index")) {
 344:     CInfo.Kind = BPFPreserveUnionAI;
 345:     CInfo.Metadata = Call->getMetadata(LLVMContext::MD_preserve_access_index);
 346:     if (!CInfo.Metadata)
 347:       report_fatal_error("Missing metadata for llvm.preserve.union.access.index intrinsic");
 348:     ResetMetadata(CInfo);
 349:     CInfo.AccessIndex = getConstant(Call->getArgOperand(1));
 350:     CInfo.Base = Call->getArgOperand(0);
 351:     return true;
 352:   }
 353:   if (GV->getName().starts_with("llvm.preserve.struct.access.index")) {
 354:     CInfo.Kind = BPFPreserveStructAI;
 355:     CInfo.Metadata = Call->getMetadata(LLVMContext::MD_preserve_access_index);
 356:     if (!CInfo.Metadata)
 357:       report_fatal_error("Missing metadata for llvm.preserve.struct.access.index intrinsic");
 358:     ResetMetadata(CInfo);
 359:     CInfo.AccessIndex = getConstant(Call->getArgOperand(2));
 360:     CInfo.Base = Call->getArgOperand(0);
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 361-380

```cpp
 361:     CInfo.RecordAlignment = DL->getABITypeAlign(getBaseElementType(Call));
 362:     return true;
 363:   }
 364:   if (GV->getName().starts_with("llvm.bpf.preserve.field.info")) {
 365:     CInfo.Kind = BPFPreserveFieldInfoAI;
 366:     CInfo.Metadata = nullptr;
 367:     // Check validity of info_kind as clang did not check this.
 368:     uint64_t InfoKind = getConstant(Call->getArgOperand(1));
 369:     if (InfoKind >= BTF::MAX_FIELD_RELOC_KIND)
 370:       report_fatal_error("Incorrect info_kind for llvm.bpf.preserve.field.info intrinsic");
 371:     CInfo.AccessIndex = InfoKind;
 372:     return true;
 373:   }
 374:   if (GV->getName().starts_with("llvm.bpf.preserve.type.info")) {
 375:     CInfo.Kind = BPFPreserveFieldInfoAI;
 376:     CInfo.Metadata = Call->getMetadata(LLVMContext::MD_preserve_access_index);
 377:     if (!CInfo.Metadata)
 378:       report_fatal_error("Missing metadata for llvm.preserve.type.info intrinsic");
 379:     uint64_t Flag = getConstant(Call->getArgOperand(1));
 380:     if (Flag >= BPFCoreSharedInfo::MAX_PRESERVE_TYPE_INFO_FLAG)
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 381-400

```cpp
 381:       report_fatal_error("Incorrect flag for llvm.bpf.preserve.type.info intrinsic");
 382:     if (Flag == BPFCoreSharedInfo::PRESERVE_TYPE_INFO_EXISTENCE)
 383:       CInfo.AccessIndex = BTF::TYPE_EXISTENCE;
 384:     else if (Flag == BPFCoreSharedInfo::PRESERVE_TYPE_INFO_MATCH)
 385:       CInfo.AccessIndex = BTF::TYPE_MATCH;
 386:     else
 387:       CInfo.AccessIndex = BTF::TYPE_SIZE;
 388:     return true;
 389:   }
 390:   if (GV->getName().starts_with("llvm.bpf.preserve.enum.value")) {
 391:     CInfo.Kind = BPFPreserveFieldInfoAI;
 392:     CInfo.Metadata = Call->getMetadata(LLVMContext::MD_preserve_access_index);
 393:     if (!CInfo.Metadata)
 394:       report_fatal_error("Missing metadata for llvm.preserve.enum.value intrinsic");
 395:     uint64_t Flag = getConstant(Call->getArgOperand(2));
 396:     if (Flag >= BPFCoreSharedInfo::MAX_PRESERVE_ENUM_VALUE_FLAG)
 397:       report_fatal_error("Incorrect flag for llvm.bpf.preserve.enum.value intrinsic");
 398:     if (Flag == BPFCoreSharedInfo::PRESERVE_ENUM_VALUE_EXISTENCE)
 399:       CInfo.AccessIndex = BTF::ENUM_VALUE_EXISTENCE;
 400:     else
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 401-420

```cpp
 401:       CInfo.AccessIndex = BTF::ENUM_VALUE;
 402:     return true;
 403:   }
 404: 
 405:   return false;
 406: }
 407: 
 408: static void replaceWithGEP(CallInst *Call, uint32_t DimensionIndex,
 409:                            uint32_t GEPIndex) {
 410:   uint32_t Dimension = 1;
 411:   if (DimensionIndex > 0)
 412:     Dimension = getConstant(Call->getArgOperand(DimensionIndex));
 413: 
 414:   Constant *Zero =
 415:       ConstantInt::get(Type::getInt32Ty(Call->getParent()->getContext()), 0);
 416:   SmallVector<Value *, 4> IdxList(Dimension, Zero);
 417:   IdxList.push_back(Call->getArgOperand(GEPIndex));
 418: 
 419:   auto *GEP = GetElementPtrInst::CreateInBounds(getBaseElementType(Call),
 420:                                                 Call->getArgOperand(0), IdxList,
```

- EN: Function bodies or method definitions such as replaceWithGEP contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: replaceWithGEP 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 421-440

```cpp
 421:                                                 "", Call->getIterator());
 422:   Call->replaceAllUsesWith(GEP);
 423:   Call->eraseFromParent();
 424: }
 425: 
 426: void BPFCoreSharedInfo::removeArrayAccessCall(CallInst *Call) {
 427:   replaceWithGEP(Call, 1, 2);
 428: }
 429: 
 430: void BPFCoreSharedInfo::removeStructAccessCall(CallInst *Call) {
 431:   replaceWithGEP(Call, 0, 1);
 432: }
 433: 
 434: void BPFCoreSharedInfo::removeUnionAccessCall(CallInst *Call) {
 435:   Call->replaceAllUsesWith(Call->getArgOperand(0));
 436:   Call->eraseFromParent();
 437: }
 438: 
 439: bool BPFAbstractMemberAccess::removePreserveAccessIndexIntrinsic(Function &F) {
 440:   std::vector<CallInst *> PreserveArrayIndexCalls;
```

- EN: Function bodies or method definitions such as removeArrayAccessCall, removeStructAccessCall, removeUnionAccessCall contain the concrete backend logic executed by LLVM passes or MC helpers.
- 中文: removeArrayAccessCall, removeStructAccessCall, removeUnionAccessCall 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。

### Lines 441-460

```cpp
 441:   std::vector<CallInst *> PreserveUnionIndexCalls;
 442:   std::vector<CallInst *> PreserveStructIndexCalls;
 443:   bool Found = false;
 444: 
 445:   for (auto &BB : F)
 446:     for (auto &I : BB) {
 447:       auto *Call = dyn_cast<CallInst>(&I);
 448:       CallInfo CInfo;
 449:       if (!IsPreserveDIAccessIndexCall(Call, CInfo))
 450:         continue;
 451: 
 452:       Found = true;
 453:       if (CInfo.Kind == BPFPreserveArrayAI)
 454:         PreserveArrayIndexCalls.push_back(Call);
 455:       else if (CInfo.Kind == BPFPreserveUnionAI)
 456:         PreserveUnionIndexCalls.push_back(Call);
 457:       else
 458:         PreserveStructIndexCalls.push_back(Call);
 459:     }
 460: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 461-480

```cpp
 461:   // do the following transformation:
 462:   // . addr = preserve_array_access_index(base, dimension, index)
 463:   //   is transformed to
 464:   //     addr = GEP(base, dimenion's zero's, index)
 465:   // . addr = preserve_union_access_index(base, di_index)
 466:   //   is transformed to
 467:   //     addr = base, i.e., all usages of "addr" are replaced by "base".
 468:   // . addr = preserve_struct_access_index(base, gep_index, di_index)
 469:   //   is transformed to
 470:   //     addr = GEP(base, 0, gep_index)
 471:   for (CallInst *Call : PreserveArrayIndexCalls)
 472:     BPFCoreSharedInfo::removeArrayAccessCall(Call);
 473:   for (CallInst *Call : PreserveStructIndexCalls)
 474:     BPFCoreSharedInfo::removeStructAccessCall(Call);
 475:   for (CallInst *Call : PreserveUnionIndexCalls)
 476:     BPFCoreSharedInfo::removeUnionAccessCall(Call);
 477: 
 478:   return Found;
 479: }
 480: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 481-500

```cpp
 481: /// Check whether the access index chain is valid. We check
 482: /// here because there may be type casts between two
 483: /// access indexes. We want to ensure memory access still valid.
 484: bool BPFAbstractMemberAccess::IsValidAIChain(const MDNode *ParentType,
 485:                                              uint32_t ParentAI,
 486:                                              const MDNode *ChildType) {
 487:   if (!ChildType)
 488:     return true; // preserve_field_info, no type comparison needed.
 489: 
 490:   const DIType *PType = stripQualifiers(cast<DIType>(ParentType));
 491:   const DIType *CType = stripQualifiers(cast<DIType>(ChildType));
 492: 
 493:   // Child is a derived/pointer type, which is due to type casting.
 494:   // Pointer type cannot be in the middle of chain.
 495:   if (isa<DIDerivedType>(CType))
 496:     return false;
 497: 
 498:   // Parent is a pointer type.
 499:   if (const auto *PtrTy = dyn_cast<DIDerivedType>(PType)) {
 500:     if (PtrTy->getTag() != dwarf::DW_TAG_pointer_type)
```

- EN: Function bodies or method definitions such as IsValidAIChain contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: IsValidAIChain 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 501-520

```cpp
 501:       return false;
 502:     return stripQualifiers(PtrTy->getBaseType()) == CType;
 503:   }
 504: 
 505:   // Otherwise, struct/union/array types
 506:   const auto *PTy = dyn_cast<DICompositeType>(PType);
 507:   const auto *CTy = dyn_cast<DICompositeType>(CType);
 508:   assert(PTy && CTy && "ParentType or ChildType is null or not composite");
 509: 
 510:   uint32_t PTyTag = PTy->getTag();
 511:   assert(PTyTag == dwarf::DW_TAG_array_type ||
 512:          PTyTag == dwarf::DW_TAG_structure_type ||
 513:          PTyTag == dwarf::DW_TAG_union_type);
 514: 
 515:   uint32_t CTyTag = CTy->getTag();
 516:   assert(CTyTag == dwarf::DW_TAG_array_type ||
 517:          CTyTag == dwarf::DW_TAG_structure_type ||
 518:          CTyTag == dwarf::DW_TAG_union_type);
 519: 
 520:   // Multi dimensional arrays, base element should be the same
```

- EN: This range continues the implementation of the backend component described by BPFAbstractMemberAccess.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 521-540

```cpp
 521:   if (PTyTag == dwarf::DW_TAG_array_type && PTyTag == CTyTag)
 522:     return PTy->getBaseType() == CTy->getBaseType();
 523: 
 524:   DIType *Ty;
 525:   if (PTyTag == dwarf::DW_TAG_array_type)
 526:     Ty = PTy->getBaseType();
 527:   else
 528:     Ty = dyn_cast<DIType>(PTy->getElements()[ParentAI]);
 529: 
 530:   return dyn_cast<DICompositeType>(stripQualifiers(Ty)) == CTy;
 531: }
 532: 
 533: void BPFAbstractMemberAccess::traceAICall(CallInst *Call,
 534:                                           CallInfo &ParentInfo) {
 535:   for (User *U : Call->users()) {
 536:     Instruction *Inst = dyn_cast<Instruction>(U);
 537:     if (!Inst)
 538:       continue;
 539: 
 540:     if (auto *BI = dyn_cast<BitCastInst>(Inst)) {
```

- EN: Function bodies or method definitions such as traceAICall contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: traceAICall 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 541-560

```cpp
 541:       traceBitCast(BI, Call, ParentInfo);
 542:     } else if (auto *CI = dyn_cast<CallInst>(Inst)) {
 543:       CallInfo ChildInfo;
 544: 
 545:       if (IsPreserveDIAccessIndexCall(CI, ChildInfo) &&
 546:           IsValidAIChain(ParentInfo.Metadata, ParentInfo.AccessIndex,
 547:                          ChildInfo.Metadata)) {
 548:         AIChain[CI] = std::make_pair(Call, ParentInfo);
 549:         traceAICall(CI, ChildInfo);
 550:       } else {
 551:         BaseAICalls[Call] = ParentInfo;
 552:       }
 553:     } else if (auto *GI = dyn_cast<GetElementPtrInst>(Inst)) {
 554:       if (GI->hasAllZeroIndices())
 555:         traceGEP(GI, Call, ParentInfo);
 556:       else
 557:         BaseAICalls[Call] = ParentInfo;
 558:     } else {
 559:       BaseAICalls[Call] = ParentInfo;
 560:     }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 561-580

```cpp
 561:   }
 562: }
 563: 
 564: void BPFAbstractMemberAccess::traceBitCast(BitCastInst *BitCast,
 565:                                            CallInst *Parent,
 566:                                            CallInfo &ParentInfo) {
 567:   for (User *U : BitCast->users()) {
 568:     Instruction *Inst = dyn_cast<Instruction>(U);
 569:     if (!Inst)
 570:       continue;
 571: 
 572:     if (auto *BI = dyn_cast<BitCastInst>(Inst)) {
 573:       traceBitCast(BI, Parent, ParentInfo);
 574:     } else if (auto *CI = dyn_cast<CallInst>(Inst)) {
 575:       CallInfo ChildInfo;
 576:       if (IsPreserveDIAccessIndexCall(CI, ChildInfo) &&
 577:           IsValidAIChain(ParentInfo.Metadata, ParentInfo.AccessIndex,
 578:                          ChildInfo.Metadata)) {
 579:         AIChain[CI] = std::make_pair(Parent, ParentInfo);
 580:         traceAICall(CI, ChildInfo);
```

- EN: Function bodies or method definitions such as traceBitCast contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: traceBitCast 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 581-600

```cpp
 581:       } else {
 582:         BaseAICalls[Parent] = ParentInfo;
 583:       }
 584:     } else if (auto *GI = dyn_cast<GetElementPtrInst>(Inst)) {
 585:       if (GI->hasAllZeroIndices())
 586:         traceGEP(GI, Parent, ParentInfo);
 587:       else
 588:         BaseAICalls[Parent] = ParentInfo;
 589:     } else {
 590:       BaseAICalls[Parent] = ParentInfo;
 591:     }
 592:   }
 593: }
 594: 
 595: void BPFAbstractMemberAccess::traceGEP(GetElementPtrInst *GEP, CallInst *Parent,
 596:                                        CallInfo &ParentInfo) {
 597:   for (User *U : GEP->users()) {
 598:     Instruction *Inst = dyn_cast<Instruction>(U);
 599:     if (!Inst)
 600:       continue;
```

- EN: Function bodies or method definitions such as traceGEP contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: traceGEP 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 601-620

```cpp
 601: 
 602:     if (auto *BI = dyn_cast<BitCastInst>(Inst)) {
 603:       traceBitCast(BI, Parent, ParentInfo);
 604:     } else if (auto *CI = dyn_cast<CallInst>(Inst)) {
 605:       CallInfo ChildInfo;
 606:       if (IsPreserveDIAccessIndexCall(CI, ChildInfo) &&
 607:           IsValidAIChain(ParentInfo.Metadata, ParentInfo.AccessIndex,
 608:                          ChildInfo.Metadata)) {
 609:         AIChain[CI] = std::make_pair(Parent, ParentInfo);
 610:         traceAICall(CI, ChildInfo);
 611:       } else {
 612:         BaseAICalls[Parent] = ParentInfo;
 613:       }
 614:     } else if (auto *GI = dyn_cast<GetElementPtrInst>(Inst)) {
 615:       if (GI->hasAllZeroIndices())
 616:         traceGEP(GI, Parent, ParentInfo);
 617:       else
 618:         BaseAICalls[Parent] = ParentInfo;
 619:     } else {
 620:       BaseAICalls[Parent] = ParentInfo;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 621-640

```cpp
 621:     }
 622:   }
 623: }
 624: 
 625: void BPFAbstractMemberAccess::collectAICallChains(Function &F) {
 626:   AIChain.clear();
 627:   BaseAICalls.clear();
 628: 
 629:   for (auto &BB : F)
 630:     for (auto &I : BB) {
 631:       CallInfo CInfo;
 632:       auto *Call = dyn_cast<CallInst>(&I);
 633:       if (!IsPreserveDIAccessIndexCall(Call, CInfo) ||
 634:           AIChain.find(Call) != AIChain.end())
 635:         continue;
 636: 
 637:       traceAICall(Call, CInfo);
 638:     }
 639: }
 640: 
```

- EN: Function bodies or method definitions such as collectAICallChains contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: collectAICallChains 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 641-660

```cpp
 641: /// Get the start and the end of storage offset for \p MemberTy.
 642: void BPFAbstractMemberAccess::GetStorageBitRange(DIDerivedType *MemberTy,
 643:                                                  Align RecordAlignment,
 644:                                                  uint32_t &StartBitOffset,
 645:                                                  uint32_t &EndBitOffset) {
 646:   uint32_t MemberBitSize = MemberTy->getSizeInBits();
 647:   uint32_t MemberBitOffset = MemberTy->getOffsetInBits();
 648: 
 649:   if (RecordAlignment > 8) {
 650:     // If the Bits are within an aligned 8-byte, set the RecordAlignment
 651:     // to 8, other report the fatal error.
 652:     if (MemberBitOffset / 64 != (MemberBitOffset + MemberBitSize) / 64)
 653:       report_fatal_error("Unsupported field expression for llvm.bpf.preserve.field.info, "
 654:                          "requiring too big alignment");
 655:     RecordAlignment = Align(8);
 656:   }
 657: 
 658:   uint32_t AlignBits = RecordAlignment.value() * 8;
 659:   if (MemberBitSize > AlignBits)
 660:     report_fatal_error("Unsupported field expression for llvm.bpf.preserve.field.info, "
```

- EN: Function bodies or method definitions such as GetStorageBitRange contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: GetStorageBitRange 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 661-680

```cpp
 661:                        "bitfield size greater than record alignment");
 662: 
 663:   StartBitOffset = MemberBitOffset & ~(AlignBits - 1);
 664:   if ((StartBitOffset + AlignBits) < (MemberBitOffset + MemberBitSize))
 665:     report_fatal_error("Unsupported field expression for llvm.bpf.preserve.field.info, "
 666:                        "cross alignment boundary");
 667:   EndBitOffset = StartBitOffset + AlignBits;
 668: }
 669: 
 670: uint32_t BPFAbstractMemberAccess::GetFieldInfo(uint32_t InfoKind,
 671:                                                DICompositeType *CTy,
 672:                                                uint32_t AccessIndex,
 673:                                                uint32_t PatchImm,
 674:                                                MaybeAlign RecordAlignment) {
 675:   if (InfoKind == BTF::FIELD_EXISTENCE)
 676:     return 1;
 677: 
 678:   uint32_t Tag = CTy->getTag();
 679:   if (InfoKind == BTF::FIELD_BYTE_OFFSET) {
 680:     if (Tag == dwarf::DW_TAG_array_type) {
```

- EN: Function bodies or method definitions such as GetFieldInfo contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: GetFieldInfo 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 681-700

```cpp
 681:       auto *EltTy = stripQualifiers(CTy->getBaseType());
 682:       PatchImm += AccessIndex * calcArraySize(CTy, 1) *
 683:                   (EltTy->getSizeInBits() >> 3);
 684:     } else if (Tag == dwarf::DW_TAG_structure_type) {
 685:       auto *MemberTy = cast<DIDerivedType>(CTy->getElements()[AccessIndex]);
 686:       if (!MemberTy->isBitField()) {
 687:         PatchImm += MemberTy->getOffsetInBits() >> 3;
 688:       } else {
 689:         unsigned SBitOffset, NextSBitOffset;
 690:         GetStorageBitRange(MemberTy, *RecordAlignment, SBitOffset,
 691:                            NextSBitOffset);
 692:         PatchImm += SBitOffset >> 3;
 693:       }
 694:     }
 695:     return PatchImm;
 696:   }
 697: 
 698:   if (InfoKind == BTF::FIELD_BYTE_SIZE) {
 699:     if (Tag == dwarf::DW_TAG_array_type) {
 700:       auto *EltTy = stripQualifiers(CTy->getBaseType());
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 701-720

```cpp
 701:       return calcArraySize(CTy, 1) * (EltTy->getSizeInBits() >> 3);
 702:     } else {
 703:       auto *MemberTy = cast<DIDerivedType>(CTy->getElements()[AccessIndex]);
 704:       uint32_t SizeInBits = MemberTy->getSizeInBits();
 705:       if (!MemberTy->isBitField())
 706:         return SizeInBits >> 3;
 707: 
 708:       unsigned SBitOffset, NextSBitOffset;
 709:       GetStorageBitRange(MemberTy, *RecordAlignment, SBitOffset,
 710:                          NextSBitOffset);
 711:       SizeInBits = NextSBitOffset - SBitOffset;
 712:       if (SizeInBits & (SizeInBits - 1))
 713:         report_fatal_error("Unsupported field expression for llvm.bpf.preserve.field.info");
 714:       return SizeInBits >> 3;
 715:     }
 716:   }
 717: 
 718:   if (InfoKind == BTF::FIELD_SIGNEDNESS) {
 719:     const DIType *BaseTy;
 720:     if (Tag == dwarf::DW_TAG_array_type) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 721-740

```cpp
 721:       // Signedness only checked when final array elements are accessed.
 722:       if (CTy->getElements().size() != 1)
 723:         report_fatal_error("Invalid array expression for llvm.bpf.preserve.field.info");
 724:       BaseTy = stripQualifiers(CTy->getBaseType());
 725:     } else {
 726:       auto *MemberTy = cast<DIDerivedType>(CTy->getElements()[AccessIndex]);
 727:       BaseTy = stripQualifiers(MemberTy->getBaseType());
 728:     }
 729: 
 730:     // Only basic types and enum types have signedness.
 731:     const auto *BTy = dyn_cast<DIBasicType>(BaseTy);
 732:     while (!BTy) {
 733:       const auto *CompTy = dyn_cast<DICompositeType>(BaseTy);
 734:       // Report an error if the field expression does not have signedness.
 735:       if (!CompTy || CompTy->getTag() != dwarf::DW_TAG_enumeration_type)
 736:         report_fatal_error("Invalid field expression for llvm.bpf.preserve.field.info");
 737:       BaseTy = stripQualifiers(CompTy->getBaseType());
 738:       BTy = dyn_cast<DIBasicType>(BaseTy);
 739:     }
 740:     uint32_t Encoding = BTy->getEncoding();
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 741-760

```cpp
 741:     return (Encoding == dwarf::DW_ATE_signed || Encoding == dwarf::DW_ATE_signed_char);
 742:   }
 743: 
 744:   if (InfoKind == BTF::FIELD_LSHIFT_U64) {
 745:     // The value is loaded into a value with FIELD_BYTE_SIZE size,
 746:     // and then zero or sign extended to U64.
 747:     // FIELD_LSHIFT_U64 and FIELD_RSHIFT_U64 are operations
 748:     // to extract the original value.
 749:     const Triple &Triple = TM->getTargetTriple();
 750:     DIDerivedType *MemberTy = nullptr;
 751:     bool IsBitField = false;
 752:     uint32_t SizeInBits;
 753: 
 754:     if (Tag == dwarf::DW_TAG_array_type) {
 755:       auto *EltTy = stripQualifiers(CTy->getBaseType());
 756:       SizeInBits = calcArraySize(CTy, 1) * EltTy->getSizeInBits();
 757:     } else {
 758:       MemberTy = cast<DIDerivedType>(CTy->getElements()[AccessIndex]);
 759:       SizeInBits = MemberTy->getSizeInBits();
 760:       IsBitField = MemberTy->isBitField();
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 761-780

```cpp
 761:     }
 762: 
 763:     if (!IsBitField) {
 764:       if (SizeInBits > 64)
 765:         report_fatal_error("too big field size for llvm.bpf.preserve.field.info");
 766:       return 64 - SizeInBits;
 767:     }
 768: 
 769:     unsigned SBitOffset, NextSBitOffset;
 770:     GetStorageBitRange(MemberTy, *RecordAlignment, SBitOffset, NextSBitOffset);
 771:     if (NextSBitOffset - SBitOffset > 64)
 772:       report_fatal_error("too big field size for llvm.bpf.preserve.field.info");
 773: 
 774:     unsigned OffsetInBits = MemberTy->getOffsetInBits();
 775:     if (Triple.getArch() == Triple::bpfel)
 776:       return SBitOffset + 64 - OffsetInBits - SizeInBits;
 777:     else
 778:       return OffsetInBits + 64 - NextSBitOffset;
 779:   }
 780: 
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 781-800

```cpp
 781:   if (InfoKind == BTF::FIELD_RSHIFT_U64) {
 782:     DIDerivedType *MemberTy = nullptr;
 783:     bool IsBitField = false;
 784:     uint32_t SizeInBits;
 785:     if (Tag == dwarf::DW_TAG_array_type) {
 786:       auto *EltTy = stripQualifiers(CTy->getBaseType());
 787:       SizeInBits = calcArraySize(CTy, 1) * EltTy->getSizeInBits();
 788:     } else {
 789:       MemberTy = cast<DIDerivedType>(CTy->getElements()[AccessIndex]);
 790:       SizeInBits = MemberTy->getSizeInBits();
 791:       IsBitField = MemberTy->isBitField();
 792:     }
 793: 
 794:     if (!IsBitField) {
 795:       if (SizeInBits > 64)
 796:         report_fatal_error("too big field size for llvm.bpf.preserve.field.info");
 797:       return 64 - SizeInBits;
 798:     }
 799: 
 800:     unsigned SBitOffset, NextSBitOffset;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 801-820

```cpp
 801:     GetStorageBitRange(MemberTy, *RecordAlignment, SBitOffset, NextSBitOffset);
 802:     if (NextSBitOffset - SBitOffset > 64)
 803:       report_fatal_error("too big field size for llvm.bpf.preserve.field.info");
 804: 
 805:     return 64 - SizeInBits;
 806:   }
 807: 
 808:   llvm_unreachable("Unknown llvm.bpf.preserve.field.info info kind");
 809: }
 810: 
 811: bool BPFAbstractMemberAccess::HasPreserveFieldInfoCall(CallInfoStack &CallStack) {
 812:   // This is called in error return path, no need to maintain CallStack.
 813:   while (CallStack.size()) {
 814:     auto StackElem = CallStack.top();
 815:     if (StackElem.second.Kind == BPFPreserveFieldInfoAI)
 816:       return true;
 817:     CallStack.pop();
 818:   }
 819:   return false;
 820: }
```

- EN: Function bodies or method definitions such as HasPreserveFieldInfoCall contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: HasPreserveFieldInfoCall 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 821-840

```cpp
 821: 
 822: /// Compute the base of the whole preserve_* intrinsics chains, i.e., the base
 823: /// pointer of the first preserve_*_access_index call, and construct the access
 824: /// string, which will be the name of a global variable.
 825: Value *BPFAbstractMemberAccess::computeBaseAndAccessKey(CallInst *Call,
 826:                                                         CallInfo &CInfo,
 827:                                                         std::string &AccessKey,
 828:                                                         MDNode *&TypeMeta) {
 829:   Value *Base = nullptr;
 830:   std::string TypeName;
 831:   CallInfoStack CallStack;
 832: 
 833:   // Put the access chain into a stack with the top as the head of the chain.
 834:   while (Call) {
 835:     CallStack.push(std::make_pair(Call, CInfo));
 836:     auto &Chain = AIChain[Call];
 837:     CInfo = Chain.second;
 838:     Call = Chain.first;
 839:   }
 840: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 841-860

```cpp
 841:   // The access offset from the base of the head of chain is also
 842:   // calculated here as all debuginfo types are available.
 843: 
 844:   // Get type name and calculate the first index.
 845:   // We only want to get type name from typedef, structure or union.
 846:   // If user wants a relocation like
 847:   //    int *p; ... __builtin_preserve_access_index(&p[4]) ...
 848:   // or
 849:   //    int a[10][20]; ... __builtin_preserve_access_index(&a[2][3]) ...
 850:   // we will skip them.
 851:   uint32_t FirstIndex = 0;
 852:   uint32_t PatchImm = 0; // AccessOffset or the requested field info
 853:   uint32_t InfoKind = BTF::FIELD_BYTE_OFFSET;
 854:   while (CallStack.size()) {
 855:     auto StackElem = CallStack.top();
 856:     Call = StackElem.first;
 857:     CInfo = StackElem.second;
 858: 
 859:     if (!Base)
 860:       Base = CInfo.Base;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 861-880

```cpp
 861: 
 862:     DIType *PossibleTypeDef = stripQualifiers(cast<DIType>(CInfo.Metadata),
 863:                                               false);
 864:     DIType *Ty = stripQualifiers(PossibleTypeDef);
 865:     if (CInfo.Kind == BPFPreserveUnionAI ||
 866:         CInfo.Kind == BPFPreserveStructAI) {
 867:       // struct or union type. If the typedef is in the metadata, always
 868:       // use the typedef.
 869:       TypeName = std::string(PossibleTypeDef->getName());
 870:       TypeMeta = PossibleTypeDef;
 871:       PatchImm += FirstIndex * (Ty->getSizeInBits() >> 3);
 872:       break;
 873:     }
 874: 
 875:     assert(CInfo.Kind == BPFPreserveArrayAI);
 876: 
 877:     // Array entries will always be consumed for accumulative initial index.
 878:     CallStack.pop();
 879: 
 880:     // BPFPreserveArrayAI
```

- EN: This chunk introduces interfaces or data structures such as or, which organize the target-specific behavior exposed by the file. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这一段引入了 or 等接口或数据结构，用于组织该文件暴露的目标专用行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 881-900

```cpp
 881:     uint64_t AccessIndex = CInfo.AccessIndex;
 882: 
 883:     DIType *BaseTy = nullptr;
 884:     bool CheckElemType = false;
 885:     if (const auto *CTy = dyn_cast<DICompositeType>(Ty)) {
 886:       // array type
 887:       assert(CTy->getTag() == dwarf::DW_TAG_array_type);
 888: 
 889: 
 890:       FirstIndex += AccessIndex * calcArraySize(CTy, 1);
 891:       BaseTy = stripQualifiers(CTy->getBaseType());
 892:       CheckElemType = CTy->getElements().size() == 1;
 893:     } else {
 894:       // pointer type
 895:       auto *DTy = cast<DIDerivedType>(Ty);
 896:       assert(DTy->getTag() == dwarf::DW_TAG_pointer_type);
 897: 
 898:       BaseTy = stripQualifiers(DTy->getBaseType());
 899:       CTy = dyn_cast<DICompositeType>(BaseTy);
 900:       if (!CTy) {
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 901-920

```cpp
 901:         CheckElemType = true;
 902:       } else if (CTy->getTag() != dwarf::DW_TAG_array_type) {
 903:         FirstIndex += AccessIndex;
 904:         CheckElemType = true;
 905:       } else {
 906:         FirstIndex += AccessIndex * calcArraySize(CTy, 0);
 907:       }
 908:     }
 909: 
 910:     if (CheckElemType) {
 911:       auto *CTy = dyn_cast<DICompositeType>(BaseTy);
 912:       if (!CTy) {
 913:         if (HasPreserveFieldInfoCall(CallStack))
 914:           report_fatal_error("Invalid field access for llvm.preserve.field.info intrinsic");
 915:         return nullptr;
 916:       }
 917: 
 918:       unsigned CTag = CTy->getTag();
 919:       if (CTag == dwarf::DW_TAG_structure_type || CTag == dwarf::DW_TAG_union_type) {
 920:         TypeName = std::string(CTy->getName());
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 921-940

```cpp
 921:       } else {
 922:         if (HasPreserveFieldInfoCall(CallStack))
 923:           report_fatal_error("Invalid field access for llvm.preserve.field.info intrinsic");
 924:         return nullptr;
 925:       }
 926:       TypeMeta = CTy;
 927:       PatchImm += FirstIndex * (CTy->getSizeInBits() >> 3);
 928:       break;
 929:     }
 930:   }
 931:   assert(TypeName.size());
 932:   AccessKey += std::to_string(FirstIndex);
 933: 
 934:   // Traverse the rest of access chain to complete offset calculation
 935:   // and access key construction.
 936:   while (CallStack.size()) {
 937:     auto StackElem = CallStack.top();
 938:     CInfo = StackElem.second;
 939:     CallStack.pop();
 940: 
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 941-960

```cpp
 941:     if (CInfo.Kind == BPFPreserveFieldInfoAI) {
 942:       InfoKind = CInfo.AccessIndex;
 943:       if (InfoKind == BTF::FIELD_EXISTENCE)
 944:         PatchImm = 1;
 945:       break;
 946:     }
 947: 
 948:     // If the next Call (the top of the stack) is a BPFPreserveFieldInfoAI,
 949:     // the action will be extracting field info.
 950:     if (CallStack.size()) {
 951:       auto StackElem2 = CallStack.top();
 952:       CallInfo CInfo2 = StackElem2.second;
 953:       if (CInfo2.Kind == BPFPreserveFieldInfoAI) {
 954:         InfoKind = CInfo2.AccessIndex;
 955:         assert(CallStack.size() == 1);
 956:       }
 957:     }
 958: 
 959:     // Access Index
 960:     uint64_t AccessIndex = CInfo.AccessIndex;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 961-980

```cpp
 961:     AccessKey += ":" + std::to_string(AccessIndex);
 962: 
 963:     MDNode *MDN = CInfo.Metadata;
 964:     // At this stage, it cannot be pointer type.
 965:     auto *CTy = cast<DICompositeType>(stripQualifiers(cast<DIType>(MDN)));
 966:     PatchImm = GetFieldInfo(InfoKind, CTy, AccessIndex, PatchImm,
 967:                             CInfo.RecordAlignment);
 968:   }
 969: 
 970:   // Access key is the
 971:   //   "llvm." + type name + ":" + reloc type + ":" + patched imm + "$" +
 972:   //   access string,
 973:   // uniquely identifying one relocation.
 974:   // The prefix "llvm." indicates this is a temporary global, which should
 975:   // not be emitted to ELF file.
 976:   AccessKey = "llvm." + TypeName + ":" + std::to_string(InfoKind) + ":" +
 977:               std::to_string(PatchImm) + "$" + AccessKey;
 978: 
 979:   return Base;
 980: }
```

- EN: This range continues the implementation of the backend component described by BPFAbstractMemberAccess.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 981-1000

```cpp
 981: 
 982: MDNode *BPFAbstractMemberAccess::computeAccessKey(CallInst *Call,
 983:                                                   CallInfo &CInfo,
 984:                                                   std::string &AccessKey,
 985:                                                   bool &IsInt32Ret) {
 986:   DIType *Ty = stripQualifiers(cast<DIType>(CInfo.Metadata), false);
 987:   assert(!Ty->getName().empty());
 988: 
 989:   int64_t PatchImm;
 990:   std::string AccessStr("0");
 991:   if (CInfo.AccessIndex == BTF::TYPE_EXISTENCE ||
 992:       CInfo.AccessIndex == BTF::TYPE_MATCH) {
 993:     PatchImm = 1;
 994:   } else if (CInfo.AccessIndex == BTF::TYPE_SIZE) {
 995:     // typedef debuginfo type has size 0, get the eventual base type.
 996:     DIType *BaseTy = stripQualifiers(Ty, true);
 997:     PatchImm = BaseTy->getSizeInBits() / 8;
 998:   } else {
 999:     // ENUM_VALUE_EXISTENCE and ENUM_VALUE
1000:     IsInt32Ret = false;
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1001-1020

```cpp
1001: 
1002:     // The argument could be a global variable or a getelementptr with base to
1003:     // a global variable depending on whether the clang option `opaque-options`
1004:     // is set or not.
1005:     const GlobalVariable *GV =
1006:         cast<GlobalVariable>(Call->getArgOperand(1)->stripPointerCasts());
1007:     assert(GV->hasInitializer());
1008:     const ConstantDataArray *DA = cast<ConstantDataArray>(GV->getInitializer());
1009:     assert(DA->isString());
1010:     StringRef ValueStr = DA->getAsString();
1011: 
1012:     // ValueStr format: <EnumeratorStr>:<Value>
1013:     size_t Separator = ValueStr.find_first_of(':');
1014:     StringRef EnumeratorStr = ValueStr.substr(0, Separator);
1015: 
1016:     // Find enumerator index in the debuginfo
1017:     DIType *BaseTy = stripQualifiers(Ty, true);
1018:     const auto *CTy = cast<DICompositeType>(BaseTy);
1019:     assert(CTy->getTag() == dwarf::DW_TAG_enumeration_type);
1020:     int EnumIndex = 0;
```

- EN: This range continues the implementation of the backend component described by BPFAbstractMemberAccess.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1021-1040

```cpp
1021:     for (const auto Element : CTy->getElements()) {
1022:       const auto *Enum = cast<DIEnumerator>(Element);
1023:       if (Enum->getName() == EnumeratorStr) {
1024:         AccessStr = std::to_string(EnumIndex);
1025:         break;
1026:       }
1027:       EnumIndex++;
1028:     }
1029: 
1030:     if (CInfo.AccessIndex == BTF::ENUM_VALUE) {
1031:       StringRef EValueStr = ValueStr.substr(Separator + 1);
1032:       PatchImm = std::stoll(std::string(EValueStr));
1033:     } else {
1034:       PatchImm = 1;
1035:     }
1036:   }
1037: 
1038:   AccessKey = "llvm." + Ty->getName().str() + ":" +
1039:               std::to_string(CInfo.AccessIndex) + std::string(":") +
1040:               std::to_string(PatchImm) + std::string("$") + AccessStr;
```

- EN: The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1041-1060

```cpp
1041: 
1042:   return Ty;
1043: }
1044: 
1045: /// Call/Kind is the base preserve_*_access_index() call. Attempts to do
1046: /// transformation to a chain of relocable GEPs.
1047: bool BPFAbstractMemberAccess::transformGEPChain(CallInst *Call,
1048:                                                 CallInfo &CInfo) {
1049:   std::string AccessKey;
1050:   MDNode *TypeMeta;
1051:   Value *Base = nullptr;
1052:   bool IsInt32Ret;
1053: 
1054:   IsInt32Ret = CInfo.Kind == BPFPreserveFieldInfoAI;
1055:   if (CInfo.Kind == BPFPreserveFieldInfoAI && CInfo.Metadata) {
1056:     TypeMeta = computeAccessKey(Call, CInfo, AccessKey, IsInt32Ret);
1057:   } else {
1058:     Base = computeBaseAndAccessKey(Call, CInfo, AccessKey, TypeMeta);
1059:     if (!Base)
1060:       return false;
```

- EN: Function bodies or method definitions such as transformGEPChain contain the concrete backend logic executed by LLVM passes or MC helpers. Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: transformGEPChain 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1061-1080

```cpp
1061:   }
1062: 
1063:   BasicBlock *BB = Call->getParent();
1064:   GlobalVariable *GV;
1065: 
1066:   if (GEPGlobals.find(AccessKey) == GEPGlobals.end()) {
1067:     IntegerType *VarType;
1068:     if (IsInt32Ret)
1069:       VarType = Type::getInt32Ty(BB->getContext()); // 32bit return value
1070:     else
1071:       VarType = Type::getInt64Ty(BB->getContext()); // 64bit ptr or enum value
1072: 
1073:     GV = new GlobalVariable(*M, VarType, false, GlobalVariable::ExternalLinkage,
1074:                             nullptr, AccessKey);
1075:     GV->addAttribute(BPFCoreSharedInfo::AmaAttr);
1076:     GV->setMetadata(LLVMContext::MD_preserve_access_index, TypeMeta);
1077:     GEPGlobals[AccessKey] = GV;
1078:   } else {
1079:     GV = GEPGlobals[AccessKey];
1080:   }
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1081-1100

```cpp
1081: 
1082:   if (CInfo.Kind == BPFPreserveFieldInfoAI) {
1083:     // Load the global variable which represents the returned field info.
1084:     LoadInst *LDInst;
1085:     if (IsInt32Ret)
1086:       LDInst = new LoadInst(Type::getInt32Ty(BB->getContext()), GV, "",
1087:                             Call->getIterator());
1088:     else
1089:       LDInst = new LoadInst(Type::getInt64Ty(BB->getContext()), GV, "",
1090:                             Call->getIterator());
1091: 
1092:     Instruction *PassThroughInst =
1093:         BPFCoreSharedInfo::insertPassThrough(M, BB, LDInst, Call);
1094:     Call->replaceAllUsesWith(PassThroughInst);
1095:     Call->eraseFromParent();
1096:     return true;
1097:   }
1098: 
1099:   // For any original GEP Call and Base %2 like
1100:   //   %4 = bitcast %struct.net_device** %dev1 to i64*
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1101-1120

```cpp
1101:   // it is transformed to:
1102:   //   %6 = load llvm.sk_buff:0:50$0:0:0:2:0
1103:   //   %8 = getelementptr i8, i8* %2, %6
1104:   //   using %8 instead of %4
1105:   // The original Call inst is removed.
1106: 
1107:   // Load the global variable.
1108:   auto *LDInst = new LoadInst(Type::getInt64Ty(BB->getContext()), GV, "",
1109:                               Call->getIterator());
1110: 
1111:   // Generate a GetElementPtr
1112:   auto *GEP = GetElementPtrInst::Create(Type::getInt8Ty(BB->getContext()), Base,
1113:                                         LDInst);
1114:   GEP->insertBefore(Call->getIterator());
1115: 
1116:   // For the following code,
1117:   //    Block0:
1118:   //      ...
1119:   //      if (...) goto Block1 else ...
1120:   //    Block1:
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1121-1140

```cpp
1121:   //      %6 = load llvm.sk_buff:0:50$0:0:0:2:0
1122:   //      %8 = getelementptr i8, i8* %2, %6
1123:   //      ...
1124:   //      goto CommonExit
1125:   //    Block2:
1126:   //      ...
1127:   //      if (...) goto Block3 else ...
1128:   //    Block3:
1129:   //      %6 = load llvm.bpf_map:0:40$0:0:0:2:0
1130:   //      %8 = getelementptr i8, i8* %2, %6
1131:   //      ...
1132:   //      goto CommonExit
1133:   //    CommonExit
1134:   // SimplifyCFG may generate:
1135:   //    Block0:
1136:   //      ...
1137:   //      if (...) goto Block_Common else ...
1138:   //     Block2:
1139:   //       ...
1140:   //      if (...) goto Block_Common else ...
```

- EN: Conditional branches encode ABI rules, legality checks, or feature-dependent behavior.
- 中文: 条件分支体现了 ABI 规则、合法性检查或依赖目标特性的行为选择。

### Lines 1141-1160

```cpp
1141:   //    Block_Common:
1142:   //      PHI = [llvm.sk_buff:0:50$0:0:0:2:0, llvm.bpf_map:0:40$0:0:0:2:0]
1143:   //      %6 = load PHI
1144:   //      %8 = getelementptr i8, i8* %2, %6
1145:   //      ...
1146:   //      goto CommonExit
1147:   //  For the above code, we cannot perform proper relocation since
1148:   //  "load PHI" has two possible relocations.
1149:   //
1150:   // To prevent above tail merging, we use __builtin_bpf_passthrough()
1151:   // where one of its parameters is a seq_num. Since two
1152:   // __builtin_bpf_passthrough() funcs will always have different seq_num,
1153:   // tail merging cannot happen. The __builtin_bpf_passthrough() will be
1154:   // removed in the beginning of Target IR passes.
1155:   //
1156:   // This approach is also used in other places when global var
1157:   // representing a relocation is used.
1158:   Instruction *PassThroughInst =
1159:       BPFCoreSharedInfo::insertPassThrough(M, BB, GEP, Call);
1160:   Call->replaceAllUsesWith(PassThroughInst);
```

- EN: This range continues the implementation of the backend component described by BPFAbstractMemberAccess.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

### Lines 1161-1180

```cpp
1161:   Call->eraseFromParent();
1162: 
1163:   return true;
1164: }
1165: 
1166: bool BPFAbstractMemberAccess::doTransformation(Function &F) {
1167:   bool Transformed = false;
1168: 
1169:   // Collect PreserveDIAccessIndex Intrinsic call chains.
1170:   // The call chains will be used to generate the access
1171:   // patterns similar to GEP.
1172:   collectAICallChains(F);
1173: 
1174:   for (auto &C : BaseAICalls)
1175:     Transformed = transformGEPChain(C.first, C.second) || Transformed;
1176: 
1177:   return removePreserveAccessIndexIntrinsic(F) || Transformed;
1178: }
1179: 
1180: PreservedAnalyses
```

- EN: Function bodies or method definitions such as doTransformation contain the concrete backend logic executed by LLVM passes or MC helpers. The loop logic walks instructions, operands, or blocks to apply a target-specific transformation or analysis.
- 中文: doTransformation 等函数/方法在这里给出具体实现，承载 LLVM Pass 或 MC 辅助逻辑实际执行的后端行为。 这里的循环遍历指令、操作数或基本块，以执行目标专用的变换或分析。

### Lines 1181-1184

```cpp
1181: BPFAbstractMemberAccessPass::run(Function &F, FunctionAnalysisManager &AM) {
1182:   return BPFAbstractMemberAccess(TM).run(F) ? PreservedAnalyses::none()
1183:                                             : PreservedAnalyses::all();
1184: }
```

- EN: This range continues the implementation of the backend component described by BPFAbstractMemberAccess.cpp, adding declarations or executable logic tied to the file’s main purpose.
- 中文: 这一段继续实现该后端组件的核心职责，补充与文件主要目标相关的声明或可执行逻辑。

## Key Concepts / 关键概念

- Target machine configuration / 目标机器配置
- ELF object support / ELF 目标文件支持

## Dependencies / 依赖关系

- Direct includes / 直接包含: `BPF.h`, `BPFCORE.h`, `BPFTargetMachine.h`, `llvm/BinaryFormat/Dwarf.h`, `llvm/DebugInfo/BTF/BTF.h`, `llvm/IR/DebugInfoMetadata.h`, `llvm/IR/GlobalVariable.h`, `llvm/IR/Instruction.h`
