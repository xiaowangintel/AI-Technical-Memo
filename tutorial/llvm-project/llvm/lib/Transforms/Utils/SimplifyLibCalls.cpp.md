# SimplifyLibCalls.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Utils/SimplifyLibCalls.cpp` | `llvm/lib/Transforms/Utils/SimplifyLibCalls.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements library calls simplifier within LLVM's shared transform utilities layer. | 该源文件实现了 LLVM 共享变换工具层中的 SimplifyLibCalls 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-304

```cpp
//===------ SimplifyLibCalls.cpp - Library calls simplifier ---------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the library calls simplifier. It does not implement
// any pass, but can be used by other passes to do simplifications.
//
//===----------------------------------------------------------------------===//

#include "llvm/Transforms/Utils/SimplifyLibCalls.h"
#include "llvm/ADT/APFloat.h"
#include "llvm/ADT/APSInt.h"
#include "llvm/ADT/SmallString.h"
#include "llvm/ADT/StringExtras.h"
#include "llvm/Analysis/ConstantFolding.h"
#include "llvm/Analysis/Loads.h"
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      DerefBytes = std::max(CI->getParamDereferenceableOrNullBytes(ArgNo),
                            DereferenceableBytes);

    if (CI->getParamDereferenceableBytes(ArgNo) < DerefBytes) {
      CI->removeParamAttr(ArgNo, Attribute::Dereferenceable);
      if (!llvm::NullPointerIsDefined(F, AS) ||
          CI->paramHasAttr(ArgNo, Attribute::NonNull))
        CI->removeParamAttr(ArgNo, Attribute::DereferenceableOrNull);
      CI->addParamAttr(ArgNo, Attribute::getWithDereferenceableBytes(
                                  CI->getContext(), DerefBytes));
    }
  }
}

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 305-602

```cpp
static void annotateNonNullNoUndefBasedOnAccess(CallInst *CI,
                                         ArrayRef<unsigned> ArgNos) {
  Function *F = CI->getCaller();
  if (!F)
    return;

  for (unsigned ArgNo : ArgNos) {
    if (!CI->paramHasAttr(ArgNo, Attribute::NoUndef))
      CI->addParamAttr(ArgNo, Attribute::NoUndef);

    if (!CI->paramHasAttr(ArgNo, Attribute::NonNull)) {
      unsigned AS =
          CI->getArgOperand(ArgNo)->getType()->getPointerAddressSpace();
      if (llvm::NullPointerIsDefined(F, AS))
        continue;
      CI->addParamAttr(ArgNo, Attribute::NonNull);
    }

    annotateDereferenceableBytes(CI, ArgNo, 1);
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        B.CreateLoad(B.getInt8Ty(), Str2P, "strcmpload"), CI->getType()));

  if (HasStr2 && Str2.empty()) // strcmp(x,"") -> *x
    return B.CreateZExt(B.CreateLoad(B.getInt8Ty(), Str1P, "strcmpload"),
                        CI->getType());

  // strcmp(P, "x") -> memcmp(P, "x", 2)
  uint64_t Len1 = GetStringLength(Str1P);
  if (Len1)
    annotateDereferenceableBytes(CI, 0, Len1);
  uint64_t Len2 = GetStringLength(Str2P);
  if (Len2)
    annotateDereferenceableBytes(CI, 1, Len2);

```
- EN: Core entities appearing here include substr, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 substr，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 603-904

```cpp
  if (Len1 && Len2) {
    return copyFlags(
        *CI, emitMemCmp(Str1P, Str2P,
                        TLI->getAsSizeT(std::min(Len1, Len2), *CI->getModule()),
                        B, DL, TLI));
  }

  // strcmp to memcmp
  if (!HasStr1 && HasStr2) {
    if (canTransformToMemCmp(CI, Str1P, Len2, DL))
      return copyFlags(*CI, emitMemCmp(Str1P, Str2P,
                                       TLI->getAsSizeT(Len2, *CI->getModule()),
                                       B, DL, TLI));
  } else if (HasStr1 && !HasStr2) {
    if (canTransformToMemCmp(CI, Str2P, Len1, DL))
      return copyFlags(*CI, emitMemCmp(Str1P, Str2P,
                                       TLI->getAsSizeT(Len1, *CI->getModule()),
                                       B, DL, TLI));
  }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  --SrcLen; // Unbias length.

  if (SrcLen == 0) {
    // Transform st{p,r}ncpy(D, "", N) to memset(D, '\0', N) for any N.
    Align MemSetAlign =
      CI->getAttributes().getParamAttrs(0).getAlignment().valueOrOne();
    CallInst *NewCI = B.CreateMemSet(Dst, B.getInt8('\0'), Size, MemSetAlign);
    AttrBuilder ArgAttrs(CI->getContext(), CI->getAttributes().getParamAttrs(0));
    NewCI->setAttributes(NewCI->getAttributes().addParamAttributes(
        CI->getContext(), 0, ArgAttrs));
    copyFlags(*CI, NewCI);
    return Dst;
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 905-1199

```cpp
  if (N > SrcLen + 1) {
    if (N > 128)
      // Bail if N is large or unknown.
      return nullptr;

    // st{p,r}ncpy(D, "a", N) -> memcpy(D, "a\0\0\0", N) for N <= 128.
    StringRef Str;
    if (!getConstantStringInfo(Src, Str))
      return nullptr;
    std::string SrcStr = Str.str();
    // Create a bigger, nul-padded array with the same length, SrcLen,
    // as the original string.
    SrcStr.resize(N, '\0');
    Src = B.CreateGlobalString(SrcStr, "str", /*AddressSpace=*/0,
                               /*M=*/nullptr, /*AddNull=*/false);
  }

  // st{p,r}ncpy(D, S, N) -> memcpy(align 1 D, align 1 S, N) when both
  // S and N are constant.
  CallInst *NewCI = B.CreateMemCpy(Dst, Align(1), Src, Align(1),
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      replaceAllUsesWith(Old, Cmp);
    }
    return CI;
  }

  // See if either input string is a constant string.
  StringRef SearchStr, ToFindStr;
  bool HasStr1 = getConstantStringInfo(CI->getArgOperand(0), SearchStr);
  bool HasStr2 = getConstantStringInfo(CI->getArgOperand(1), ToFindStr);

  // fold strstr(x, "") -> x.
  if (HasStr2 && ToFindStr.empty())
    return CI->getArgOperand(0);

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1200-1495

```cpp
  // If both strings are known, constant fold it.
  if (HasStr1 && HasStr2) {
    size_t Offset = SearchStr.find(ToFindStr);

    if (Offset == StringRef::npos) // strstr("foo", "bar") -> null
      return Constant::getNullValue(CI->getType());

    // strstr("abcd", "bc") -> gep((char*)"abcd", 1)
    return B.CreateConstInBoundsGEP1_64(B.getInt8Ty(), CI->getArgOperand(0),
                                        Offset, "strstr");
  }

  // fold strstr(x, "y") -> strchr(x, 'y').
  if (HasStr2 && ToFindStr.size() == 1) {
    return emitStrChr(CI->getArgOperand(0), ToFindStr[0], B, TLI);
  }

  annotateNonNullNoUndefBasedOnAccess(CI, {0, 1});
  return nullptr;
}
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  Value *BitfieldC = B.getInt(Bitfield);

  // Adjust width of "C" to the bitfield width, then mask off the high bits.
  Value *C = B.CreateZExtOrTrunc(CharVal, BitfieldC->getType());
  C = B.CreateAnd(C, B.getIntN(Width, 0xFF));

  // First check that the bit field access is within bounds.
  Value *Bounds = B.CreateICmp(ICmpInst::ICMP_ULT, C, B.getIntN(Width, Width),
                               "memchr.bounds");

  // Create code that checks if the given bit is set in the field.
  Value *Shl = B.CreateShl(B.getIntN(Width, 1ULL), C);
  Value *Bits = B.CreateIsNotNull(B.CreateAnd(Shl, BitfieldC), "memchr.bits");

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1496-1791

```cpp
  // Finally merge both checks and cast to pointer type. The inttoptr
  // implicitly zexts the i1 to intptr type.
  return B.CreateIntToPtr(B.CreateLogicalAnd(Bounds, Bits, "memchr"),
                          CI->getType());
}

// Optimize a memcmp or, when StrNCmp is true, strncmp call CI with constant
// arrays LHS and RHS and nonconstant Size.
static Value *optimizeMemCmpVarSize(CallInst *CI, Value *LHS, Value *RHS,
                                    Value *Size, bool StrNCmp,
                                    IRBuilderBase &B, const DataLayout &DL) {
  if (LHS == RHS) // memcmp(s,s,x) -> 0
    return Constant::getNullValue(CI->getType());

  StringRef LStr, RStr;
  if (!getConstantStringInfo(LHS, LStr, /*TrimAtNul=*/false) ||
      !getConstantStringInfo(RHS, RStr, /*TrimAtNul=*/false))
    return nullptr;

  // If the contents of both constant arrays are known, fold a call to
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return nullptr;
  }
  return optimizeNew(CI, B, Func);
}

// When enabled, replace operator new() calls marked with a hot or cold memprof
// attribute with an operator new() call that takes a __hot_cold_t parameter.
// Currently this is supported by the open source version of tcmalloc, see:
// https://github.com/google/tcmalloc/blob/master/tcmalloc/new_extension.h
Value *LibCallSimplifier::optimizeNew(CallInst *CI, IRBuilderBase &B,
                                      LibFunc &Func) {
  if (!OptimizeHotColdNew)
    return nullptr;

```
- EN: Core entities appearing here include isOnlyUsedInZeroEqualityComparison, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isOnlyUsedInZeroEqualityComparison，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1792-2088

```cpp
  uint8_t HotCold;
  if (CI->getAttributes().getFnAttr("memprof").getValueAsString() == "cold")
    HotCold = ColdNewHintValue;
  else if (CI->getAttributes().getFnAttr("memprof").getValueAsString() ==
           "notcold")
    HotCold = NotColdNewHintValue;
  else if (CI->getAttributes().getFnAttr("memprof").getValueAsString() == "hot")
    HotCold = HotNewHintValue;
  else if (CI->getAttributes().getFnAttr("memprof").getValueAsString() ==
           "ambiguous")
    HotCold = AmbiguousNewHintValue;
  else
    return nullptr;

  // For calls that already pass a hot/cold hint, only update the hint if
  // directed by OptimizeExistingHotColdNew. For other calls to new, add a hint
  // if cold or hot, and leave as-is for default handling if "notcold" aka warm.
  // Note that in cases where we decide it is "notcold", it might be slightly
  // better to replace the hinted call with a non hinted call, to avoid the
  // extra parameter and the if condition check of the hint value in the
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        AbsOp = Imag;

    } else if (ConstantFP *ConstImag = dyn_cast<ConstantFP>(Imag)) {
      if (ConstImag->isZero())
        AbsOp = Real;
    }

    if (AbsOp)
      return copyFlags(*CI, B.CreateFAbs(AbsOp, CI, "cabs"));

    if (!CI->isFast())
      return nullptr;
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2089-2386

```cpp
  // Propagate fast-math flags from the existing call to new instructions.
  Value *RealReal = B.CreateFMulFMF(Real, Real, CI);
  Value *ImagImag = B.CreateFMulFMF(Imag, Imag, CI);
  return copyFlags(
      *CI, B.CreateUnaryIntrinsic(Intrinsic::sqrt,
                                  B.CreateFAddFMF(RealReal, ImagImag, CI), CI,
                                  "cabs"));
}

// Return a properly extended integer (DstWidth bits wide) if the operation is
// an itofp.
static Value *getIntToFPVal(Value *I2F, IRBuilderBase &B, unsigned DstWidth) {
  if (isa<SIToFPInst>(I2F) || isa<UIToFPInst>(I2F)) {
    Value *Op = cast<Instruction>(I2F)->getOperand(0);
    // Make sure that the exponent fits inside an "int" of size DstWidth,
    // thus avoiding any range issues that FP has not.
    unsigned BitWidth = Op->getType()->getScalarSizeInBits();
    if (BitWidth < DstWidth || (BitWidth == DstWidth && isa<SIToFPInst>(I2F))) {
      Type *IntTy = Op->getType()->getWithNewBitWidth(DstWidth);
      return isa<SIToFPInst>(I2F) ? B.CreateSExt(Op, IntTy)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  Value *Base = Pow->getArgOperand(0);
  Value *Expo = Pow->getArgOperand(1);
  Function *Callee = Pow->getCalledFunction();
  StringRef Name = Callee->getName();
  Type *Ty = Pow->getType();
  Module *M = Pow->getModule();
  bool AllowApprox = Pow->hasApproxFunc();
  bool Ignored;

  // Propagate the math semantics from the call to any created instructions.
  IRBuilderBase::FastMathFlagGuard Guard(B);
  B.setFastMathFlags(Pow->getFastMathFlags());
  // Evaluate special cases related to the base.

```
- EN: Core entities appearing here include isLibFuncEmittable, hasFloatFn, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isLibFuncEmittable, hasFloatFn，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2387-2693

```cpp
  // pow(1.0, x) -> 1.0
  if (match(Base, m_FPOne()))
    return Base;

  if (Value *Exp = replacePowWithExp(Pow, B))
    return Exp;

  // Evaluate special cases related to the exponent.

  // pow(x, -1.0) -> 1.0 / x
  if (match(Expo, m_SpecificFP(-1.0)))
    return B.CreateFDiv(ConstantFP::get(Ty, 1.0), Base, "reciprocal");

  // pow(x, +/-0.0) -> 1.0
  if (match(Expo, m_AnyZeroFP()))
    return ConstantFP::get(Ty, 1.0);

  // pow(x, 1.0) -> x
  if (match(Expo, m_FPOne()))
    return Base;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        Log->doesNotAccessMemory()
            ? B.CreateUnaryIntrinsic(LogID, Arg->getOperand(0), nullptr, "log")
            : emitUnaryFloatFnCall(Arg->getOperand(0), TLI, LogNm, B, NoAttrs);
    Value *Y = Arg->getArgOperand(1);
    // Cast exponent to FP if integer.
    if (ArgID == Intrinsic::powi)
      Y = B.CreateSIToFP(Y, Ty, "cast");
    Value *MulY = B.CreateFMul(Y, LogX, "mul");
    // Since pow() may have side effects, e.g. errno,
    // dead code elimination may not be trusted to remove it.
    substituteInParent(Arg, MulY);
    return MulY;
  }

```
- EN: Core entities appearing here include hasFloatVersion, hasFloatFn, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 hasFloatVersion, hasFloatFn，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2694-2994

```cpp
  // log(exp{,2,10}(y)) -> y*log({e,2,10})
  // TODO: There is no exp10() intrinsic yet.
  if (ArgLb == ExpLb || ArgLb == Exp2Lb || ArgLb == Exp10Lb ||
           ArgID == Intrinsic::exp || ArgID == Intrinsic::exp2) {
    Constant *Eul;
    if (ArgLb == ExpLb || ArgID == Intrinsic::exp)
      // FIXME: Add more precise value of e for long double.
      Eul = ConstantFP::get(Log->getType(), numbers::e);
    else if (ArgLb == Exp2Lb || ArgID == Intrinsic::exp2)
      Eul = ConstantFP::get(Log->getType(), 2.0);
    else
      Eul = ConstantFP::get(Log->getType(), 10.0);
    Value *LogE = Log->doesNotAccessMemory()
                      ? B.CreateUnaryIntrinsic(LogID, Eul, nullptr, "log")
                      : emitUnaryFloatFnCall(Eul, TLI, LogNm, B, NoAttrs);
    Value *MulY = B.CreateFMul(Arg->getArgOperand(0), LogE, "mul");
    // Since exp() may have side effects, e.g. errno,
    // dead code elimination may not be trusted to remove it.
    substituteInParent(Arg, MulY);
    return MulY;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }

  SinCos = B.CreateCall(Callee, Arg, "sincospi");

  if (SinCos->getType()->isStructTy()) {
    Sin = B.CreateExtractValue(SinCos, 0, "sinpi");
    Cos = B.CreateExtractValue(SinCos, 1, "cospi");
  } else {
    Sin = B.CreateExtractElement(SinCos, ConstantInt::get(B.getInt32Ty(), 0),
                                 "sinpi");
    Cos = B.CreateExtractElement(SinCos, ConstantInt::get(B.getInt32Ty(), 1),
                                 "cospi");
  }

```
- EN: Core entities appearing here include isLibFuncEmittable, isTrigLibCall, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isLibFuncEmittable, isTrigLibCall，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2995-3289

```cpp
  return true;
}

static Value *optimizeSymmetricCall(CallInst *CI, bool IsEven,
                                    IRBuilderBase &B) {
  Value *X;
  Value *Src = CI->getArgOperand(0);

  if (match(Src, m_OneUse(m_FNeg(m_Value(X))))) {
    auto *Call = B.CreateCall(CI->getCalledFunction(), {X});
    Call->copyFastMathFlags(CI);
    auto *CallInst = copyFlags(*CI, Call);
    if (IsEven) {
      // Even function: f(-x) = f(x)
      return CallInst;
    }
    // Odd function: f(-x) = -f(x)
    return B.CreateFNegFMF(CallInst, CI);
  }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

// Fold calls to strtol, strtoll, strtoul, and strtoull.
Value *LibCallSimplifier::optimizeStrToInt(CallInst *CI, IRBuilderBase &B,
                                           bool AsSigned) {
  Value *EndPtr = CI->getArgOperand(1);
  if (isa<ConstantPointerNull>(EndPtr)) {
    // With a null EndPtr, this function won't capture the main argument.
    // It would be readonly too, except that it still may write to errno.
    CI->addParamAttr(0, Attribute::getWithCaptureInfo(CI->getContext(),
                                                      CaptureInfo::none()));
    EndPtr = nullptr;
  } else if (!isKnownNonZero(EndPtr, DL))
    return nullptr;

```
- EN: Core entities appearing here include match, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 match，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3290-3583

```cpp
  StringRef Str;
  if (!getConstantStringInfo(CI->getArgOperand(0), Str))
    return nullptr;

  if (ConstantInt *CInt = dyn_cast<ConstantInt>(CI->getArgOperand(2))) {
    return convertStrToInt(CI, Str, EndPtr, CInt->getSExtValue(), AsSigned, B);
  }

  return nullptr;
}

//===----------------------------------------------------------------------===//
// Formatting and IO Library Call Optimizations
//===----------------------------------------------------------------------===//

static bool isReportingError(Function *Callee, CallInst *CI, int StreamArg);

Value *LibCallSimplifier::optimizeErrorReporting(CallInst *CI, IRBuilderBase &B,
                                                 int StreamArg) {
  Function *Callee = CI->getCalledFunction();
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // floating point arguments.
  if (isLibFuncEmittable(M, TLI, LibFunc_small_sprintf) &&
      !callHasFP128Argument(CI)) {
    auto SmallSPrintFFn = getOrInsertLibFunc(M, *TLI, LibFunc_small_sprintf, FT,
                                             Callee->getAttributes());
    CallInst *New = cast<CallInst>(CI->clone());
    New->setCalledFunction(SmallSPrintFFn);
    B.Insert(New);
    return New;
  }

  return nullptr;
}

```
- EN: Core entities appearing here include isReportingError, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isReportingError，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3584-3878

```cpp
// Transform an snprintf call CI with the bound N to format the string Str
// either to a call to memcpy, or to single character a store, or to nothing,
// and fold the result to a constant.  A nonnull StrArg refers to the string
// argument being formatted.  Otherwise the call is one with N < 2 and
// the "%c" directive to format a single character.
Value *LibCallSimplifier::emitSnPrintfMemCpy(CallInst *CI, Value *StrArg,
                                             StringRef Str, uint64_t N,
                                             IRBuilderBase &B) {
  assert(StrArg || (N < 2 && Str.size() == 1));

  unsigned IntBits = TLI->getIntSize();
  uint64_t IntMax = maxIntN(IntBits);
  if (Str.size() > IntMax)
    // Bail if the string is longer than INT_MAX.  POSIX requires
    // implementations to set errno to EOVERFLOW in this case, in
    // addition to when N is larger than that (checked by the caller).
    return nullptr;

  Value *StrLen = ConstantInt::get(CI->getType(), Str.size());
  if (N == 0)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // puts("") -> putchar('\n')
  StringRef Str;
  if (getConstantStringInfo(CI->getArgOperand(0), Str) && Str.empty()) {
    // putchar takes an argument of the same type as puts returns, i.e.,
    // int, which need not be 32 bits wide.
    Type *IntTy = CI->getType();
    return copyFlags(*CI, emitPutChar(ConstantInt::get(IntTy, '\n'), B, TLI));
  }

  return nullptr;
}

Value *LibCallSimplifier::optimizeExit(CallInst *CI) {

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3879-4178

```cpp
  // Mark 'exit' as cold if its not exit(0) (success).
  const APInt *C;
  if (!CI->hasFnAttr(Attribute::Cold) &&
      match(CI->getArgOperand(0), m_APInt(C)) && !C->isZero()) {
    CI->addFnAttr(Attribute::Cold);
  }
  return nullptr;
}

Value *LibCallSimplifier::optimizeBCopy(CallInst *CI, IRBuilderBase &B) {
  // bcopy(src, dst, n) -> llvm.memmove(dst, src, n)
  return copyFlags(*CI, B.CreateMemMove(CI->getArgOperand(1), Align(1),
                                        CI->getArgOperand(0), Align(1),
                                        CI->getArgOperand(2)));
}

bool LibCallSimplifier::hasFloatVersion(const Module *M, StringRef FuncName) {
  SmallString<20> FloatFuncName = FuncName;
  FloatFuncName += 'f';
  return isLibFuncEmittable(M, TLI, FloatFuncName);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

Value *LibCallSimplifier::optimizeCall(CallInst *CI, IRBuilderBase &Builder) {
  Module *M = CI->getModule();
  assert(!CI->isMustTailCall() && "These transforms aren't musttail safe.");

  // TODO: Split out the code below that operates on FP calls so that
  //       we can all non-FP calls with the StrictFP attribute to be
  //       optimized.
  if (CI->isNoBuiltin()) {
    // Optionally update operator new calls.
    return maybeOptimizeNoBuiltinOperatorNew(CI, Builder);
  }

```
- EN: Core entities appearing here include match, hasFloatVersion, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 match, hasFloatVersion，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 4179-4477

```cpp
  LibFunc Func;
  Function *Callee = CI->getCalledFunction();
  bool IsCallingConvC = TargetLibraryInfoImpl::isCallingConvCCompatible(CI);

  SmallVector<OperandBundleDef, 2> OpBundles;
  CI->getOperandBundlesAsDefs(OpBundles);

  IRBuilderBase::OperandBundlesGuard Guard(Builder);
  Builder.setDefaultOperandBundles(OpBundles);

  // Command-line parameter overrides instruction attribute.
  // This can't be moved to optimizeFloatingPointLibCall() because it may be
  // used by the intrinsic optimizations.
  if (EnableUnsafeFPShrink.getNumOccurrences() > 0)
    UnsafeFPShrink = EnableUnsafeFPShrink;
  else if (isa<FPMathOperator>(CI) && CI->isFast())
    UnsafeFPShrink = true;

  // First, check for intrinsics.
  if (IntrinsicInst *II = dyn_cast<IntrinsicInst>(CI)) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    else
      return copyFlags(*CI, emitStpCpy(Dst, Src, B, TLI));
  }

  if (OnlyLowerUnknownSize)
    return nullptr;

  // Maybe we can stil fold __st[rp]cpy_chk to __memcpy_chk.
  uint64_t Len = GetStringLength(Src);
  if (Len)
    annotateDereferenceableBytes(CI, 1, Len);
  else
    return nullptr;

```
- EN: Core entities appearing here include replaceAllUsesWith, eraseFromParent, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AssumptionCache, BlockFrequencyInfo, DataLayout, DominatorTree. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 replaceAllUsesWith, eraseFromParent，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AssumptionCache, BlockFrequencyInfo, DataLayout, DominatorTree 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 4478-4685

```cpp
  unsigned SizeTBits = TLI->getSizeTSize(*CI->getModule());
  Type *SizeTTy = IntegerType::get(CI->getContext(), SizeTBits);
  Value *LenV = ConstantInt::get(SizeTTy, Len);
  Value *Ret = emitMemCpyChk(Dst, Src, LenV, ObjSize, B, DL, TLI);
  // If the function was an __stpcpy_chk, and we were able to fold it into
  // a __memcpy_chk, we still need to return the correct end pointer.
  if (Ret && Func == LibFunc_stpcpy_chk)
    return B.CreateInBoundsGEP(B.getInt8Ty(), Dst,
                               ConstantInt::get(SizeTTy, Len - 1));
  return copyFlags(*CI, cast<CallInst>(Ret));
}

Value *FortifiedLibCallSimplifier::optimizeStrLenChk(CallInst *CI,
                                                     IRBuilderBase &B) {
  if (isFortifiedCallFoldable(CI, 1, std::nullopt, 0))
    return copyFlags(*CI, emitStrLen(CI->getArgOperand(0), B,
                                     CI->getDataLayout(), TLI));
  return nullptr;
}

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return optimizeStrLCpyChk(CI, Builder);
  case LibFunc_vsnprintf_chk:
    return optimizeVSNPrintfChk(CI, Builder);
  case LibFunc_vsprintf_chk:
    return optimizeVSPrintfChk(CI, Builder);
  default:
    break;
  }
  return nullptr;
}

FortifiedLibCallSimplifier::FortifiedLibCallSimplifier(
    const TargetLibraryInfo *TLI, bool OnlyLowerUnknownSize)
    : TLI(TLI), OnlyLowerUnknownSize(OnlyLowerUnknownSize) {}
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Utils` focuses on shared transform utilities.
  - CN: 领域：`Utils` 主要处理 共享变换工具。
- EN: Primary entities: `HotColdHintParser, parse, ignoreCallingConv, isOnlyUsedInEqualityComparison, callHasFloatingPointArgument, callHasFP128Argument, isOnlyUsedInComparisonWithZero, substr` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`HotColdHintParser, parse, ignoreCallingConv, isOnlyUsedInEqualityComparison, callHasFloatingPointArgument, callHasFP128Argument, isOnlyUsedInComparisonWithZero, substr` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AssumptionCache, BlockFrequencyInfo, DataLayout, DominatorTree, OptimizationRemarkEmitter, ProfileSummaryInfo, TargetLibraryInfo, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AssumptionCache, BlockFrequencyInfo, DataLayout, DominatorTree, OptimizationRemarkEmitter, ProfileSummaryInfo, TargetLibraryInfo, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `EnableUnsafeFPShrink, OptimizeHotColdNew, OptimizeExistingHotColdNew, OptimizeNoBuiltinHotColdNew, ColdNewHintValue` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `EnableUnsafeFPShrink, OptimizeHotColdNew, OptimizeExistingHotColdNew, OptimizeNoBuiltinHotColdNew, ColdNewHintValue` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Implementation role: it carries executable transformation logic that rewrites IR, updates CFG state, or builds vectorization/planning structures.
  - CN: 实现角色：它包含可执行的变换逻辑，会改写 IR、更新 CFG 状态，或构建向量化/规划结构。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/ConstantFolding.h`, `llvm/Analysis/Loads.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/Utils/Local.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/ConstantFolding.h`, `llvm/Analysis/Loads.h`, `llvm/Analysis/OptimizationRemarkEmitter.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/Utils/Local.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/AttributeMask.h`, `llvm/IR/DataLayout.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/Module.h`, `llvm/IR/PatternMatch.h`, `llvm/Transforms/Utils/SimplifyLibCalls.h`, `llvm/Transforms/Utils/BuildLibCalls.h`, `llvm/Transforms/Utils/Local.h`, `llvm/Transforms/Utils/SizeOpts.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/AttributeMask.h`, `llvm/IR/DataLayout.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/Intrinsics.h`, `llvm/IR/Module.h`, `llvm/IR/PatternMatch.h`, `llvm/Transforms/Utils/SimplifyLibCalls.h`, `llvm/Transforms/Utils/BuildLibCalls.h`, `llvm/Transforms/Utils/Local.h`, `llvm/Transforms/Utils/SizeOpts.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/KnownBits.h`, `llvm/Support/KnownFPClass.h`, `llvm/Support/MathExtras.h`, `llvm/ADT/APFloat.h`, `llvm/ADT/APSInt.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringExtras.h`, `llvm/TargetParser/Triple.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/KnownBits.h`, `llvm/Support/KnownFPClass.h`, `llvm/Support/MathExtras.h`, `llvm/ADT/APFloat.h`, `llvm/ADT/APSInt.h`, `llvm/ADT/SmallString.h`, `llvm/ADT/StringExtras.h`, `llvm/TargetParser/Triple.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cmath` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cmath` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AssumptionCache`, `BlockFrequencyInfo`, `DataLayout`, `DominatorTree`, `OptimizationRemarkEmitter`, `ProfileSummaryInfo`, `TargetLibraryInfo`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AssumptionCache`, `BlockFrequencyInfo`, `DataLayout`, `DominatorTree`, `OptimizationRemarkEmitter`, `ProfileSummaryInfo`, `TargetLibraryInfo`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
