# BuildLibCalls.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/BuildLibCalls.cpp` | `llvm/lib/Transforms/Utils/BuildLibCalls.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements utility builder for libcalls within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 BuildLibCalls 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-148

```cpp
//===- BuildLibCalls.cpp - Utility builder for libcalls -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements some functions that will create standard C libcalls.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/BuildLibCalls.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/Statistic.h"
#include "llvm/Analysis/MemoryBuiltins.h"
#include "llvm/Analysis/TargetLibraryInfo.h"
#include "llvm/IR/Argument.h"
#include "llvm/IR/CallingConv.h"
#include "llvm/IR/Constants.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (!setMemoryEffects(F, MemoryEffects::inaccessibleOrArgOrErrnoMemOnly(
                               ModRefInfo::ModRef, ModRefInfo::Mod)))
    return false;
  ++NumInaccessibleMemOrArgMemOrErrnoMemOnly;
  return true;
}

static bool setOnlyWritesErrnoMemory(Function &F) {
  if (!setMemoryEffects(F, MemoryEffects::errnoMemOnly(ModRefInfo::Mod)))
    return false;
  ++NumWriteErrnoMemOnly;
  return true;
}

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 149-294

```cpp
static bool setOnlyWritesArgMemOrErrnoMem(Function &F) {
  if (!setMemoryEffects(F, MemoryEffects::argumentOrErrnoMemOnly(
                               ModRefInfo::Mod, ModRefInfo::Mod)))
    return false;
  ++NumWriteArgumentMemOrErrnoMemOnly;
  return true;
}

static bool setDoesNotThrow(Function &F) {
  if (F.doesNotThrow())
    return false;
  F.setDoesNotThrow();
  ++NumNoUnwind;
  return true;
}

static bool setDoesNotCallback(Function &F) {
  if (F.hasFnAttribute(Attribute::NoCallback))
    return false;
  F.addFnAttr(Attribute::NoCallback);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
static bool setAlignedAllocParam(Function &F, unsigned ArgNo) {
  if (F.hasParamAttribute(ArgNo, Attribute::AllocAlign))
    return false;
  F.addParamAttr(ArgNo, Attribute::AllocAlign);
  return true;
}

static bool setAllocatedPointerParam(Function &F, unsigned ArgNo) {
  if (F.hasParamAttribute(ArgNo, Attribute::AllocatedPointer))
    return false;
  F.addParamAttr(ArgNo, Attribute::AllocatedPointer);
  return true;
}

```
- EN: Core entities appearing here include setOnlyWritesArgMemOrErrnoMem, setDoesNotThrow, setDoesNotCallback, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 setOnlyWritesArgMemOrErrnoMem, setDoesNotThrow, setDoesNotCallback，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 295-436

```cpp
static bool setAllocSize(Function &F, unsigned ElemSizeArg,
                         std::optional<unsigned> NumElemsArg) {
  if (F.hasFnAttribute(Attribute::AllocSize))
    return false;
  F.addFnAttr(Attribute::getWithAllocSizeArgs(F.getContext(), ElemSizeArg,
                                              NumElemsArg));
  return true;
}

static bool setAllocFamily(Function &F, StringRef Family) {
  if (F.hasFnAttribute("alloc-family"))
    return false;
  F.addFnAttr("alloc-family", Family);
  return true;
}

static bool setAllocKind(Function &F, AllocFnKind K) {
  if (F.hasFnAttribute(Attribute::AllocKind))
    return false;
  F.addFnAttr(
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  case LibFunc_strncasecmp: //
    // Those functions may depend on the locale, which may be accessed through
    // global memory.
    Changed |= setOnlyReadsMemory(F);
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCallback(F);
    Changed |= setWillReturn(F);
    Changed |= setDoesNotCapture(F, 0);
    Changed |= setDoesNotCapture(F, 1);
    break;
  case LibFunc_strstr:
  case LibFunc_strpbrk:
    Changed |= setOnlyAccessesArgMemory(F);
    Changed |= setOnlyReadsMemory(F);
```
- EN: Core entities appearing here include setAllocFamily, setAllocKind, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetLibraryInfo.
- CN: 此处出现的核心实体包括 setAllocFamily, setAllocKind，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetLibraryInfo 等分析结果。

### Lines 437-578

```cpp
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCallback(F);
    Changed |= setWillReturn(F);
    Changed |= setDoesNotCapture(F, 1);
    break;
  case LibFunc_strtok:
  case LibFunc_strtok_r:
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCallback(F);
    Changed |= setWillReturn(F);
    Changed |= setDoesNotCapture(F, 1);
    Changed |= setOnlyReadsMemory(F, 1);
    break;
  case LibFunc_scanf:
    Changed |= setRetAndArgsNoUndef(F);
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCapture(F, 0);
    Changed |= setOnlyReadsMemory(F, 0);
    break;
  case LibFunc_setbuf:
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Changed |= setOnlyAccessesArgMemory(F);
    Changed |= setOnlyWritesMemory(F);
    Changed |= setDoesNotCapture(F, 1);
    break;
  case LibFunc_memcpy:
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCallback(F);
    Changed |= setOnlyAccessesArgMemory(F);
    Changed |= setWillReturn(F);
    Changed |= setDoesNotAlias(F, 0);
    Changed |= setReturnedArg(F, 0);
    Changed |= setOnlyWritesMemory(F, 0);
    Changed |= setDoesNotAlias(F, 1);
    Changed |= setDoesNotCapture(F, 1);
```
- EN: This region continues the BuildLibCalls implementation with local helper logic centered on Changed, LibFunc_strtok, LibFunc_strtok_r, LibFunc_scanf.
- CN: 这一段延续了 BuildLibCalls 的主体实现，围绕 Changed, LibFunc_strtok, LibFunc_strtok_r, LibFunc_scanf 等局部辅助逻辑展开。

### Lines 579-720

```cpp
    Changed |= setOnlyReadsMemory(F, 1);
    break;
  case LibFunc_memmove:
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCallback(F);
    Changed |= setOnlyAccessesArgMemory(F);
    Changed |= setWillReturn(F);
    Changed |= setReturnedArg(F, 0);
    Changed |= setOnlyWritesMemory(F, 0);
    Changed |= setDoesNotCapture(F, 1);
    Changed |= setOnlyReadsMemory(F, 1);
    break;
  case LibFunc_mempcpy:
  case LibFunc_memccpy:
    Changed |= setWillReturn(F);
    [[fallthrough]];
  case LibFunc_memcpy_chk:
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCallback(F);
    Changed |= setOnlyAccessesArgMemory(F);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    break;
  case LibFunc_bcmp:
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCallback(F);
    Changed |= setOnlyAccessesArgMemory(F);
    Changed |= setOnlyReadsMemory(F);
    Changed |= setWillReturn(F);
    Changed |= setDoesNotCapture(F, 0);
    Changed |= setDoesNotCapture(F, 1);
    break;
  case LibFunc_bzero:
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCallback(F);
    Changed |= setOnlyAccessesArgMemory(F);
```
- EN: This region continues the BuildLibCalls implementation with local helper logic centered on Changed, LibFunc_memmove, LibFunc_mempcpy, LibFunc_memccpy.
- CN: 这一段延续了 BuildLibCalls 的主体实现，围绕 Changed, LibFunc_memmove, LibFunc_mempcpy, LibFunc_memccpy 等局部辅助逻辑展开。

### Lines 721-862

```cpp
    Changed |= setWillReturn(F);
    Changed |= setDoesNotCapture(F, 0);
    Changed |= setOnlyWritesMemory(F, 0);
    break;
  case LibFunc_calloc:
  case LibFunc_vec_calloc:
    Changed |= setAllocFamily(F, TheLibFunc == LibFunc_vec_calloc ? "vec_malloc"
                                                                  : "malloc");
    Changed |= setAllocKind(F, AllocFnKind::Alloc | AllocFnKind::Zeroed);
    Changed |= setAllocSize(F, 0, 1);
    Changed |= setOnlyAccessesInaccessibleMemOrErrnoMem(F);
    Changed |= setRetAndArgsNoUndef(F);
    Changed |= setDoesNotThrow(F);
    Changed |= setRetDoesNotAlias(F);
    Changed |= setWillReturn(F);
    break;
  case LibFunc_chmod:
  case LibFunc_chown:
    Changed |= setRetAndArgsNoUndef(F);
    Changed |= setDoesNotThrow(F);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Changed |= setDoesNotCapture(F, 2);
    Changed |= setOnlyWritesMemory(F, 0);
    break;
  case LibFunc_fread:
  case LibFunc_fread_unlocked:
    Changed |= setRetAndArgsNoUndef(F);
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCapture(F, 0);
    Changed |= setDoesNotCapture(F, 3);
    Changed |= setOnlyWritesMemory(F, 0);
    break;
  case LibFunc_fwrite:
  case LibFunc_fwrite_unlocked:
    Changed |= setRetAndArgsNoUndef(F);
```
- EN: This region continues the BuildLibCalls implementation with local helper logic centered on Changed, LibFunc_calloc, LibFunc_vec_calloc, TheLibFunc.
- CN: 这一段延续了 BuildLibCalls 的主体实现，围绕 Changed, LibFunc_calloc, LibFunc_vec_calloc, TheLibFunc 等局部辅助逻辑展开。

### Lines 863-1004

```cpp
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCapture(F, 0);
    Changed |= setDoesNotCapture(F, 3);
    Changed |= setOnlyReadsMemory(F, 0);
    break;
  case LibFunc_fputs:
  case LibFunc_fputs_unlocked:
    Changed |= setRetAndArgsNoUndef(F);
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCapture(F, 0);
    Changed |= setDoesNotCapture(F, 1);
    Changed |= setOnlyReadsMemory(F, 0);
    break;
  case LibFunc_fscanf:
  case LibFunc_fprintf:
    Changed |= setRetAndArgsNoUndef(F);
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCapture(F, 0);
    Changed |= setDoesNotCapture(F, 1);
    Changed |= setOnlyReadsMemory(F, 1);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Changed |= setDoesNotThrow(F);
    Changed |= setRetDoesNotAlias(F);
    Changed |= setDoesNotCapture(F, 0);
    Changed |= setDoesNotCapture(F, 1);
    Changed |= setOnlyReadsMemory(F, 0);
    Changed |= setOnlyReadsMemory(F, 1);
    break;
  case LibFunc_pclose:
    Changed |= setRetAndArgsNoUndef(F);
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCapture(F, 0);
    break;
  case LibFunc_vscanf:
    Changed |= setRetAndArgsNoUndef(F);
```
- EN: This region continues the BuildLibCalls implementation with local helper logic centered on Changed, LibFunc_fputs, LibFunc_fputs_unlocked, LibFunc_fscanf.
- CN: 这一段延续了 BuildLibCalls 的主体实现，围绕 Changed, LibFunc_fputs, LibFunc_fputs_unlocked, LibFunc_fscanf 等局部辅助逻辑展开。

### Lines 1005-1146

```cpp
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCapture(F, 0);
    Changed |= setOnlyReadsMemory(F, 0);
    break;
  case LibFunc_vsscanf:
    Changed |= setRetAndArgsNoUndef(F);
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCapture(F, 0);
    Changed |= setDoesNotCapture(F, 1);
    Changed |= setOnlyReadsMemory(F, 0);
    Changed |= setOnlyReadsMemory(F, 1);
    break;
  case LibFunc_vfscanf:
    Changed |= setRetAndArgsNoUndef(F);
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCapture(F, 0);
    Changed |= setDoesNotCapture(F, 1);
    Changed |= setOnlyReadsMemory(F, 1);
    break;
  case LibFunc_vprintf:
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    Changed |= setDoesNotCapture(F, 1);
    Changed |= setOnlyReadsMemory(F, 0);
    break;
  case LibFunc_dunder_isoc99_sscanf:
    Changed |= setRetAndArgsNoUndef(F);
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCapture(F, 0);
    Changed |= setDoesNotCapture(F, 1);
    Changed |= setOnlyReadsMemory(F, 0);
    Changed |= setOnlyReadsMemory(F, 1);
    break;
  case LibFunc_fopen64:
    Changed |= setRetAndArgsNoUndef(F);
    Changed |= setDoesNotThrow(F);
```
- EN: This region continues the BuildLibCalls implementation with local helper logic centered on Changed, LibFunc_vsscanf, LibFunc_vfscanf, LibFunc_vprintf.
- CN: 这一段延续了 BuildLibCalls 的主体实现，围绕 Changed, LibFunc_vsscanf, LibFunc_vfscanf, LibFunc_vprintf 等局部辅助逻辑展开。

### Lines 1147-1288

```cpp
    Changed |= setRetDoesNotAlias(F);
    Changed |= setDoesNotCapture(F, 0);
    Changed |= setDoesNotCapture(F, 1);
    Changed |= setOnlyReadsMemory(F, 0);
    Changed |= setOnlyReadsMemory(F, 1);
    break;
  case LibFunc_fseeko64:
  case LibFunc_ftello64:
    Changed |= setRetAndArgsNoUndef(F);
    Changed |= setDoesNotThrow(F);
    Changed |= setDoesNotCapture(F, 0);
    break;
  case LibFunc_tmpfile64:
    Changed |= setRetAndArgsNoUndef(F);
    Changed |= setDoesNotThrow(F);
    Changed |= setRetDoesNotAlias(F);
    break;
  case LibFunc_fstat64:
  case LibFunc_fstatvfs64:
    Changed |= setRetAndArgsNoUndef(F);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  case LibFunc_ldexp:
  case LibFunc_ldexpf:
  case LibFunc_ldexpl:
  case LibFunc_log:
  case LibFunc_log10:
  case LibFunc_log10f:
  case LibFunc_log10l:
  case LibFunc_log1p:
  case LibFunc_log1pf:
  case LibFunc_log1pl:
  case LibFunc_log2:
  case LibFunc_log2f:
  case LibFunc_log2l:
  case LibFunc_logb:
```
- EN: This region continues the BuildLibCalls implementation with local helper logic centered on Changed, LibFunc_fseeko64, LibFunc_ftello64, LibFunc_tmpfile64.
- CN: 这一段延续了 BuildLibCalls 的主体实现，围绕 Changed, LibFunc_fseeko64, LibFunc_ftello64, LibFunc_tmpfile64 等局部辅助逻辑展开。

### Lines 1289-1434

```cpp
  case LibFunc_logbf:
  case LibFunc_logbl:
  case LibFunc_ilogb:
  case LibFunc_ilogbf:
  case LibFunc_ilogbl:
  case LibFunc_logf:
  case LibFunc_logl:
  case LibFunc_nextafter:
  case LibFunc_nextafterf:
  case LibFunc_nextafterl:
  case LibFunc_nexttoward:
  case LibFunc_nexttowardf:
  case LibFunc_nexttowardl:
  case LibFunc_pow:
  case LibFunc_powf:
  case LibFunc_powl:
  case LibFunc_remainder:
  case LibFunc_remainderf:
  case LibFunc_remainderl:
  case LibFunc_rint:
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // We have to do this step after AllocKind has been inferred on functions so
  // we can reliably identify free-like and realloc-like functions.
  if (!isLibFreeFunction(&F, TheLibFunc) && !isReallocLikeFn(&F))
    Changed |= setDoesNotFreeMemory(F);
  return Changed;
}

static void setArgExtAttr(Function &F, unsigned ArgNo,
                          const TargetLibraryInfo &TLI, bool Signed = true) {
  Attribute::AttrKind ExtAttr = TLI.getExtAttrForI32Param(Signed);
  if (ExtAttr != Attribute::None && !F.hasParamAttribute(ArgNo, ExtAttr))
    F.addParamAttr(ArgNo, ExtAttr);
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetLibraryInfo.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 TargetLibraryInfo 等分析结果。

### Lines 1435-1578

```cpp
static void setRetExtAttr(Function &F,
                          const TargetLibraryInfo &TLI, bool Signed = true) {
  Attribute::AttrKind ExtAttr = TLI.getExtAttrForI32Return(Signed);
  if (ExtAttr != Attribute::None && !F.hasRetAttribute(ExtAttr))
    F.addRetAttr(ExtAttr);
}

// Modeled after X86TargetLowering::markLibCallAttributes.
void llvm::markRegisterParameterAttributes(Function *F) {
  if (!F->arg_size() || F->isVarArg())
    return;

  const CallingConv::ID CC = F->getCallingConv();
  if (CC != CallingConv::C && CC != CallingConv::X86_StdCall)
    return;

  const Module *M = F->getParent();
  unsigned N = M->getNumberRegisterParameters();
  if (!N)
    return;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      return TLI->isValidProtoForLibFunc(*F->getFunctionType(), TheLibFunc, *M);
    return false;
  }

  return true;
}

bool llvm::isLibFuncEmittable(const Module *M, const TargetLibraryInfo *TLI,
                              StringRef Name) {
  LibFunc TheLibFunc;
  return TLI->getLibFunc(Name, TheLibFunc) &&
         isLibFuncEmittable(M, TLI, TheLibFunc);
}

```
- EN: Core entities appearing here include markRegisterParameterAttributes, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 markRegisterParameterAttributes，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1579-1722

```cpp
bool llvm::hasFloatFn(const Module *M, const TargetLibraryInfo *TLI, Type *Ty,
                      LibFunc DoubleFn, LibFunc FloatFn, LibFunc LongDoubleFn) {
  switch (Ty->getTypeID()) {
  case Type::HalfTyID:
    return false;
  case Type::FloatTyID:
    return isLibFuncEmittable(M, TLI, FloatFn);
  case Type::DoubleTyID:
    return isLibFuncEmittable(M, TLI, DoubleFn);
  default:
    return isLibFuncEmittable(M, TLI, LongDoubleFn);
  }
}

StringRef llvm::getFloatFn(const Module *M, const TargetLibraryInfo *TLI,
                           Type *Ty, LibFunc DoubleFn, LibFunc FloatFn,
                           LibFunc LongDoubleFn, LibFunc &TheLibFunc) {
  assert(hasFloatFn(M, TLI, Ty, DoubleFn, FloatFn, LongDoubleFn) &&
         "Cannot get name for unavailable function!");

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                         const TargetLibraryInfo *TLI) {
  Type *CharPtrTy = B.getPtrTy();
  Type *SizeTTy = getSizeTTy(B, TLI);
  return emitLibCall(LibFunc_stpncpy, CharPtrTy, {CharPtrTy, CharPtrTy, SizeTTy},
                     {Dst, Src, Len}, B, TLI);
}

Value *llvm::emitMemCpyChk(Value *Dst, Value *Src, Value *Len, Value *ObjSize,
                           IRBuilderBase &B, const DataLayout &DL,
                           const TargetLibraryInfo *TLI) {
  Module *M = B.GetInsertBlock()->getModule();
  if (!isLibFuncEmittable(M, TLI, LibFunc_memcpy_chk))
    return nullptr;

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1723-1867

```cpp
  AttributeList AS;
  AS = AttributeList::get(M->getContext(), AttributeList::FunctionIndex,
                          Attribute::NoUnwind);
  Type *VoidPtrTy = B.getPtrTy();
  Type *SizeTTy = getSizeTTy(B, TLI);
  FunctionCallee MemCpy = getOrInsertLibFunc(M, *TLI, LibFunc_memcpy_chk,
      AttributeList::get(M->getContext(), AS), VoidPtrTy,
      VoidPtrTy, VoidPtrTy, SizeTTy, SizeTTy);
  CallInst *CI = B.CreateCall(MemCpy, {Dst, Src, Len, ObjSize});
  if (const Function *F =
          dyn_cast<Function>(MemCpy.getCallee()->stripPointerCasts()))
    CI->setCallingConv(F->getCallingConv());
  return CI;
}

Value *llvm::emitMemPCpy(Value *Dst, Value *Src, Value *Len, IRBuilderBase &B,
                         const DataLayout &DL, const TargetLibraryInfo *TLI) {
  Type *VoidPtrTy = B.getPtrTy();
  Type *SizeTTy = getSizeTTy(B, TLI);
  return emitLibCall(LibFunc_mempcpy, VoidPtrTy,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                     {Dest, Src, Size}, B, TLI);
}

Value *llvm::emitVSNPrintf(Value *Dest, Value *Size, Value *Fmt, Value *VAList,
                           IRBuilderBase &B, const TargetLibraryInfo *TLI) {
  Type *CharPtrTy = B.getPtrTy();
  Type *IntTy = getIntTy(B, TLI);
  Type *SizeTTy = getSizeTTy(B, TLI);
  return emitLibCall(
      LibFunc_vsnprintf, IntTy,
      {CharPtrTy, SizeTTy, CharPtrTy, VAList->getType()},
      {Dest, Size, Fmt, VAList}, B, TLI);
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1868-2014

```cpp
Value *llvm::emitVSPrintf(Value *Dest, Value *Fmt, Value *VAList,
                          IRBuilderBase &B, const TargetLibraryInfo *TLI) {
  Type *CharPtrTy = B.getPtrTy();
  Type *IntTy = getIntTy(B, TLI);
  return emitLibCall(LibFunc_vsprintf, IntTy,
                     {CharPtrTy, CharPtrTy, VAList->getType()},
                     {Dest, Fmt, VAList}, B, TLI);
}

/// Append a suffix to the function name according to the type of 'Op'.
static void appendTypeSuffix(Value *Op, StringRef &Name,
                             SmallString<20> &NameBuffer) {
  if (!Op->getType()->isDoubleTy()) {
      NameBuffer += Name;

    if (Op->getType()->isFloatTy())
      NameBuffer += 'f';
    else
      NameBuffer += 'l';

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  Type *IntTy = getIntTy(B, TLI);
  StringRef PutCharName = TLI->getName(LibFunc_putchar);
  FunctionCallee PutChar = getOrInsertLibFunc(M, *TLI, LibFunc_putchar,
                                              IntTy, IntTy);
  inferNonMandatoryLibFuncAttrs(M, PutCharName, *TLI);
  CallInst *CI = B.CreateCall(PutChar, Char, PutCharName);

  if (const Function *F =
          dyn_cast<Function>(PutChar.getCallee()->stripPointerCasts()))
    CI->setCallingConv(F->getCallingConv());
  return CI;
}

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2015-2157

```cpp
Value *llvm::emitPutS(Value *Str, IRBuilderBase &B,
                      const TargetLibraryInfo *TLI) {
  Module *M = B.GetInsertBlock()->getModule();
  if (!isLibFuncEmittable(M, TLI, LibFunc_puts))
    return nullptr;

  Type *IntTy = getIntTy(B, TLI);
  StringRef PutsName = TLI->getName(LibFunc_puts);
  FunctionCallee PutS =
      getOrInsertLibFunc(M, *TLI, LibFunc_puts, IntTy, B.getPtrTy());
  inferNonMandatoryLibFuncAttrs(M, PutsName, *TLI);
  CallInst *CI = B.CreateCall(PutS, Str, PutsName);
  if (const Function *F =
          dyn_cast<Function>(PutS.getCallee()->stripPointerCasts()))
    CI->setCallingConv(F->getCallingConv());
  return CI;
}

Value *llvm::emitFPutC(Value *Char, Value *File, IRBuilderBase &B,
                       const TargetLibraryInfo *TLI) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  StringRef Name = TLI->getName(SizeFeedbackNewFunc);

  // __sized_ptr_t struct return type { void*, size_t }
  StructType *SizedPtrT =
      StructType::get(M->getContext(), {B.getPtrTy(), Num->getType()});
  FunctionCallee Func =
      M->getOrInsertFunction(Name, SizedPtrT, Num->getType(), B.getInt8Ty());
  inferNonMandatoryLibFuncAttrs(M, Name, *TLI);
  CallInst *CI = B.CreateCall(Func, {Num, B.getInt8(HotCold)}, "sized_ptr");

  if (const Function *F = dyn_cast<Function>(Func.getCallee()))
    CI->setCallingConv(F->getCallingConv());

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2158-2268

```cpp
  return CI;
}

Value *llvm::emitHotColdSizeReturningNewAligned(Value *Num, Value *Align,
                                                IRBuilderBase &B,
                                                const TargetLibraryInfo *TLI,
                                                LibFunc SizeFeedbackNewFunc,
                                                uint8_t HotCold) {
  Module *M = B.GetInsertBlock()->getModule();
  if (!isLibFuncEmittable(M, TLI, SizeFeedbackNewFunc))
    return nullptr;

  StringRef Name = TLI->getName(SizeFeedbackNewFunc);

  // __sized_ptr_t struct return type { void*, size_t }
  StructType *SizedPtrT =
      StructType::get(M->getContext(), {B.getPtrTy(), Num->getType()});
  FunctionCallee Func = M->getOrInsertFunction(Name, SizedPtrT, Num->getType(),
                                               Align->getType(), B.getInt8Ty());
  inferNonMandatoryLibFuncAttrs(M, Name, *TLI);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  StringRef Name = TLI->getName(NewFunc);
  FunctionCallee Func = M->getOrInsertFunction(
      Name, B.getPtrTy(), Num->getType(), Align->getType(), NoThrow->getType(),
      B.getInt8Ty());
  inferNonMandatoryLibFuncAttrs(M, Name, *TLI);
  CallInst *CI =
      B.CreateCall(Func, {Num, Align, NoThrow, B.getInt8(HotCold)}, Name);

  if (const Function *F =
          dyn_cast<Function>(Func.getCallee()->stripPointerCasts()))
    CI->setCallingConv(F->getCallingConv());

  return CI;
}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `setDoesNotAccessMemory, setIsCold, setNoReturn, setMemoryEffects, setOnlyReadsMemory, setOnlyWritesMemory, setOnlyAccessesArgMemory, setOnlyAccessesInaccessibleMemOrArgMem` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`setDoesNotAccessMemory, setIsCold, setNoReturn, setMemoryEffects, setOnlyReadsMemory, setOnlyWritesMemory, setOnlyAccessesArgMemory, setOnlyAccessesInaccessibleMemOrArgMem` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, TargetLibraryInfo` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, TargetLibraryInfo` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/MemoryBuiltins.h`, `llvm/Analysis/TargetLibraryInfo.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/MemoryBuiltins.h`, `llvm/Analysis/TargetLibraryInfo.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Argument.h`, `llvm/IR/CallingConv.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Module.h`, `llvm/IR/Type.h`, `llvm/Transforms/Utils/BuildLibCalls.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Argument.h`, `llvm/IR/CallingConv.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/Module.h`, `llvm/IR/Type.h`, `llvm/Transforms/Utils/BuildLibCalls.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/TypeSize.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/Statistic.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/TypeSize.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/Statistic.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `optional` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`optional` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout`, `TargetLibraryInfo` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `TargetLibraryInfo` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
