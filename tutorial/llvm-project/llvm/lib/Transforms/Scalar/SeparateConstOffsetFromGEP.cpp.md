# SeparateConstOffsetFromGEP.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/SeparateConstOffsetFromGEP.cpp` | `llvm/lib/Transforms/Scalar/SeparateConstOffsetFromGEP.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements Separate Const Offset From GEP within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 SeparateConstOffsetFromGEP 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-98

```cpp
//===- SeparateConstOffsetFromGEP.cpp -------------------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Loop unrolling may create many similar GEPs for array accesses.
// e.g., a 2-level loop
//
// float a[32][32]; // global variable
//
// for (int i = 0; i < 2; ++i) {
//   for (int j = 0; j < 2; ++j) {
//     ...
//     ... = a[x + i][y + j];
//     ...
//   }
// }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
// (2) After such Lowering, there are more optimization opportunities such as
//     CSE, LICM and CGP.
//
// E.g. The following GEPs have multiple indices:
//  BB1:
//    %p = getelementptr [10 x %struct], ptr %ptr, i64 %i, i64 %j1, i32 3
//    load %p
//    ...
//  BB2:
//    %p2 = getelementptr [10 x %struct], ptr %ptr, i64 %i, i64 %j1, i32 2
//    load %p2
//    ...
//
// We can not do CSE to the common part related to index "i64 %i". Lowering
```
- EN: This commented region explains the pass goal, legality assumptions, or the high-level algorithm before readers dive into the executable code. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段注释先说明 pass 的目标、合法性假设或整体算法，再引导读者进入真正的实现代码。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 99-188

```cpp
// GEPs can achieve such goals.
//
// This pass will lower a GEP with multiple indices into multiple GEPs with a
// single index:
//  BB1:
//    %2 = mul i64 %i, length_of_10xstruct          ; CSE opportunity
//    %3 = getelementptr i8, ptr %ptr, i64 %2       ; CSE opportunity
//    %4 = mul i64 %j1, length_of_struct
//    %5 = getelementptr i8, ptr %3, i64 %4
//    %p = getelementptr i8, ptr %5, struct_field_3 ; Constant offset
//    load %p
//    ...
//  BB2:
//    %8 = mul i64 %i, length_of_10xstruct            ; CSE opportunity
//    %9 = getelementptr i8, ptr %ptr, i64 %8         ; CSE opportunity
//    %10 = mul i64 %j2, length_of_struct
//    %11 = getelementptr i8, ptr %9, i64 %10
//    %p2 = getelementptr i8, ptr %11, struct_field_2 ; Constant offset
//    load %p2
//    ...
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    "disable-separate-const-offset-from-gep", cl::init(false),
    cl::desc("Do not separate the constant offset from a GEP instruction"),
    cl::Hidden);

// Setting this flag may emit false positives when the input module already
// contains dead instructions. Therefore, we set it only in unit tests that are
// free of dead code.
static cl::opt<bool>
    VerifyNoDeadCode("reassociate-geps-verify-no-dead-code", cl::init(false),
                     cl::desc("Verify this pass produces no dead code"),
                     cl::Hidden);

namespace {

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 189-288

```cpp
/// A helper class for separating a constant offset from a GEP index.
///
/// In real programs, a GEP index may be more complicated than a simple addition
/// of something and a constant integer which can be trivially splitted. For
/// example, to split ((a << 3) | 5) + b, we need to search deeper for the
/// constant offset, so that we can separate the index to (a << 3) + b and 5.
///
/// Therefore, this class looks into the expression that computes a given GEP
/// index, and tries to find a constant integer that can be hoisted to the
/// outermost level of the expression as an addition. Not every constant in an
/// expression can jump out. e.g., we cannot transform (b * (a + 5)) to (b * a +
/// 5); nor can we transform (3 * (a + 5)) to (3 * a + 5), however in this case,
/// -instcombine probably already optimized (3 * (a + 5)) to (3 * a + 15).
class ConstantOffsetExtractor {
public:
  /// Extracts a constant offset from the given GEP index. It returns the
  /// new index representing the remainder (equal to the original index minus
  /// the constant offset), or nullptr if we cannot extract a constant offset.
  /// \p Idx The given GEP index
  /// \p GEP The given GEP
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  ///
  /// \p ChainIndex The index to UserChain. ChainIndex is initially
  ///               UserChain.size() - 1, and is decremented during
  ///               the recursion.
  Value *distributeCastsAndCloneChain(unsigned ChainIndex);

  /// Reassociates the GEP index to the form I' + C and returns I'.
  Value *removeConstOffset(unsigned ChainIndex);

  /// A helper function to apply CastInsts, a list of sext/zext/trunc, to value
  /// V.  e.g., if CastInsts = [sext i32 to i64, zext i16 to i32], this function
  /// returns "sext i32 (zext i16 V to i32) to i64".
  Value *applyCasts(Value *V);

```
- EN: Core entities appearing here include ConstantOffsetExtractor, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 ConstantOffsetExtractor，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 289-395

```cpp
  /// A helper function that returns whether we can trace into the operands
  /// of binary operator BO for a constant offset.
  ///
  /// \p SignExtended Whether BO is surrounded by sext
  /// \p ZeroExtended Whether BO is surrounded by zext
  /// \p GEP          The base GEP instruction, used for determining relevant
  ///                 types and flags needed for safe reassociation.
  /// \p Idx          The original index of the GEP
  bool canTraceInto(bool SignExtended, bool ZeroExtended, BinaryOperator *BO,
                    GetElementPtrInst *GEP, Value *Idx);

  /// The path from the constant offset to the old GEP index. e.g., if the GEP
  /// index is "a * b + (c + 5)". After running function find, UserChain[0] will
  /// be the constant 5, UserChain[1] will be the subexpression "c + 5", and
  /// UserChain[2] will be the entire expression "a * b + (c + 5)".
  ///
  /// This path helps to rebuild the new GEP index.
  SmallVector<User *, 8> UserChain;

  /// A data structure used in rebuildWithoutConstOffset. Contains all
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  /// \p Variadic                  The variadic part of the original GEP.
  /// \p AccumulativeByteOffset    The constant offset.
  void lowerToSingleIndexGEPs(GetElementPtrInst *Variadic,
                              const APInt &AccumulativeByteOffset);

  /// Finds the constant offset within each index and accumulates them. If
  /// LowerGEP is true, it finds in indices of both sequential and structure
  /// types, otherwise it only finds in sequential indices. The output
  /// NeedsExtraction indicates whether we successfully find a non-zero constant
  /// offset, and SignedOverflow indicates if there was signed overflow in
  /// offset calculation.
  APInt accumulateByteOffset(GetElementPtrInst *GEP, bool &NeedsExtraction,
                             bool &SignedOverflow);

```
- EN: Core entities appearing here include SeparateConstOffsetFromGEPLegacyPass, FunctionPass, SeparateConstOffsetFromGEP, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree, LoopInfo, TargetLibraryInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 SeparateConstOffsetFromGEPLegacyPass, FunctionPass, SeparateConstOffsetFromGEP，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree, LoopInfo, TargetLibraryInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 396-493

```cpp
  /// Canonicalize array indices to pointer-size integers. This helps to
  /// simplify the logic of splitting a GEP. For example, if a + b is a
  /// pointer-size integer, we have
  ///   gep base, a + b = gep (gep base, a), b
  /// However, this equality may not hold if the size of a + b is smaller than
  /// the pointer size, because LLVM conceptually sign-extends GEP indices to
  /// pointer size before computing the address
  /// (http://llvm.org/docs/LangRef.html#id181).
  ///
  /// This canonicalization is very likely already done in clang and
  /// instcombine. Therefore, the program will probably remain the same.
  ///
  /// Returns true if the module changes.
  ///
  /// Verified in @i32_add in split-gep.ll
  bool canonicalizeArrayIndicesToIndexSize(GetElementPtrInst *GEP);

  /// Optimize sext(a)+sext(b) to sext(a+b) when a+b can't sign overflow.
  /// SeparateConstOffsetFromGEP distributes a sext to leaves before extracting
  /// the constant offset. After extraction, it becomes desirable to reunion the
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

// Checks if it is safe to reorder an add/sext result used in a GEP.
//
// An inbounds GEP does not guarantee that the index is non-negative.
// This helper checks first if the index is known non-negative. If the index is
// non-negative, the transform is always safe.
// Second, it checks whether the GEP is inbounds and directly based on a global
// or an alloca, which are required to prove futher transform validity.
// If the GEP:
// - Has a zero offset from the base, the index is non-negative (any negative
//   value would produce poison/UB)
// - Has ObjectSize < (2^(N-1) - C + 1) * stride, where C is a constant from the
//   add, stride is the element size of Idx, and N is bitwidth of Idx.
//   This is because with this pattern:
```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout, DominatorTree, LoopInfo, ScalarEvolution. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout, DominatorTree, LoopInfo, ScalarEvolution 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 494-600

```cpp
//     %add = add iN %val, C
//     %sext = sext iN %add to i64
//     %gep = getelementptr inbounds TYPE, %sext
//   The worst-case is when %val sign-flips to produce the smallest magnitude
//   negative value, at 2^(N-1)-1. In this case, the add/sext is -(2^(N-1)-C+1),
//   and the sext/add is 2^(N-1)+C-1 (2^N difference). The original add/sext
//   only produces a defined GEP when -(2^(N-1)-C+1) is inbounds. So, if
//   ObjectSize < (2^(N-1) - C + 1) * stride, it is impossible for the
//   worst-case sign-flip to be defined.
//   Note that in this case the GEP is not neccesarily non-negative, but any
//   negative results will still produce the same behavior in the reordered
//   version with a defined GEP.
//   This can also work for negative C, but the threshold is instead
//   (2^(N-1)+C)*stride, since the sign-flip is done in reverse and is instead
//   producing a large positive value that still needs to be inbounds to the
//   object size. If C is negative, we cannot make any useful assumptions based
//   on the offset, since it would need to be extremely large.
static bool canReorderAddSextToGEP(const GetElementPtrInst *GEP,
                                   const Value *Idx, const BinaryOperator *Add,
                                   const DataLayout &DL) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

bool ConstantOffsetExtractor::canTraceInto(bool SignExtended, bool ZeroExtended,
                                           BinaryOperator *BO,
                                           GetElementPtrInst *GEP, Value *Idx) {
  // We only consider ADD, SUB and OR, because a non-zero constant found in
  // expressions composed of these operations can be easily hoisted as a
  // constant offset by reassociation.
  if (BO->getOpcode() != Instruction::Add &&
      BO->getOpcode() != Instruction::Sub &&
      BO->getOpcode() != Instruction::Or) {
    return false;
  }

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 601-699

```cpp
  // Do not trace into "or" unless it is equivalent to "add nuw nsw".
  // This is the case if the or's disjoint flag is set.
  if (BO->getOpcode() == Instruction::Or &&
      !cast<PossiblyDisjointInst>(BO)->isDisjoint())
    return false;

  // FIXME: We don't currently support constants from the RHS of subs,
  // when we are zero-extended, because we need a way to zero-extended
  // them before they are negated.
  if (ZeroExtended && !SignExtended && BO->getOpcode() == Instruction::Sub)
    return false;

  // In addition, tracing into BO requires that its surrounding sext/zext/trunc
  // (if any) is distributable to both operands.
  //
  // Suppose BO = A op B.
  //  SignExtended | ZeroExtended | Distributable?
  // --------------+--------------+----------------------------------
  //       0       |      0       | true because no s/zext exists
  //       0       |      1       | zext(BO) == zext(A) op zext(B)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (ConstantOffset == 0)
    UserChain.resize(ChainLength);

  return ConstantOffset;
}

APInt ConstantOffsetExtractor::find(Value *V, GetElementPtrInst *GEP,
                                    Value *Idx, bool SignExtended,
                                    bool ZeroExtended) {
  // TODO(jingyue): We could trace into integer/pointer casts, such as
  // inttoptr, ptrtoint, bitcast, and addrspacecast. We choose to handle only
  // integers because it gives good enough results for our benchmarks.
  unsigned BitWidth = cast<IntegerType>(V->getType())->getBitWidth();

```
- EN: This region continues the SeparateConstOffsetFromGEP implementation with local helper logic centered on This, Instruction, PossiblyDisjointInst, FIXME.
- CN: 这一段延续了 SeparateConstOffsetFromGEP 的主体实现，围绕 This, Instruction, PossiblyDisjointInst, FIXME 等局部辅助逻辑展开。

### Lines 700-801

```cpp
  // We cannot do much with Values that are not a User, such as an Argument.
  User *U = dyn_cast<User>(V);
  if (U == nullptr) return APInt(BitWidth, 0);

  APInt ConstantOffset(BitWidth, 0);
  if (ConstantInt *CI = dyn_cast<ConstantInt>(V)) {
    // Hooray, we found it!
    ConstantOffset = CI->getValue();
  } else if (BinaryOperator *BO = dyn_cast<BinaryOperator>(V)) {
    // Trace into subexpressions for more hoisting opportunities.
    if (canTraceInto(SignExtended, ZeroExtended, BO, GEP, Idx))
      ConstantOffset = findInEitherOperand(BO, SignExtended, ZeroExtended);
  } else if (isa<TruncInst>(V)) {
    ConstantOffset =
        find(U->getOperand(0), GEP, Idx, SignExtended, ZeroExtended)
            .trunc(BitWidth);
  } else if (isa<SExtInst>(V)) {
    ConstantOffset =
        find(U->getOperand(0), GEP, Idx, /* SignExtended */ true, ZeroExtended)
            .sext(BitWidth);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        (isa<SExtInst>(Cast) || isa<ZExtInst>(Cast) || isa<TruncInst>(Cast)) &&
        "Only following instructions can be traced: sext, zext & trunc");
    CastInsts.push_back(Cast);
    UserChain[ChainIndex] = nullptr;
    return distributeCastsAndCloneChain(ChainIndex - 1);
  }

  // Function find only trace into BinaryOperator and CastInst.
  BinaryOperator *BO = cast<BinaryOperator>(U);
  // OpNo = which operand of BO is UserChain[ChainIndex - 1]
  unsigned OpNo = (BO->getOperand(0) == UserChain[ChainIndex - 1] ? 0 : 1);
  Value *TheOther = applyCasts(BO->getOperand(1 - OpNo));
  Value *NextInChain = distributeCastsAndCloneChain(ChainIndex - 1);

```
- EN: Core entities appearing here include distributeCastsAndCloneChain, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 distributeCastsAndCloneChain，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 802-901

```cpp
  BinaryOperator *NewBO = nullptr;
  if (OpNo == 0) {
    NewBO = BinaryOperator::Create(BO->getOpcode(), NextInChain, TheOther,
                                   BO->getName(), IP);
  } else {
    NewBO = BinaryOperator::Create(BO->getOpcode(), TheOther, NextInChain,
                                   BO->getName(), IP);
  }
  return UserChain[ChainIndex] = NewBO;
}

Value *ConstantOffsetExtractor::removeConstOffset(unsigned ChainIndex) {
  if (ChainIndex == 0) {
    assert(isa<ConstantInt>(UserChain[ChainIndex]));
    return ConstantInt::getNullValue(UserChain[ChainIndex]->getType());
  }

  BinaryOperator *BO = cast<BinaryOperator>(UserChain[ChainIndex]);
  assert((BO->use_empty() || BO->hasOneUse()) &&
         "distributeCastsAndCloneChain clones each BinaryOperator in "
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

Value *ConstantOffsetExtractor::Extract(Value *Idx, GetElementPtrInst *GEP,
                                        User *&UserChainTail,
                                        bool &PreservesNUW) {
  ConstantOffsetExtractor Extractor(GEP->getIterator());
  // Find a non-zero constant offset first.
  APInt ConstantOffset = Extractor.find(Idx, GEP, Idx, /* SignExtended */ false,
                                        /* ZeroExtended */ false);
  if (ConstantOffset == 0) {
    UserChainTail = nullptr;
    PreservesNUW = true;
    return nullptr;
  }

```
- EN: Core entities appearing here include allowsPreservingNUW, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 allowsPreservingNUW，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 902-1005

```cpp
  PreservesNUW = all_of(Extractor.UserChain, allowsPreservingNUW);

  // Separates the constant offset from the GEP index.
  Value *IdxWithoutConstOffset = Extractor.rebuildWithoutConstOffset();
  UserChainTail = Extractor.UserChain.back();
  return IdxWithoutConstOffset;
}

APInt ConstantOffsetExtractor::Find(Value *Idx, GetElementPtrInst *GEP) {
  return ConstantOffsetExtractor(GEP->getIterator())
      .find(Idx, GEP, Idx, /* SignExtended */ false, /* ZeroExtended */ false);
}

bool SeparateConstOffsetFromGEP::canonicalizeArrayIndicesToIndexSize(
    GetElementPtrInst *GEP) {
  bool Changed = false;
  Type *PtrIdxTy = DL->getIndexType(GEP->getType());
  gep_type_iterator GTI = gep_type_begin(*GEP);
  for (User::op_iterator I = GEP->op_begin() + 1, E = GEP->op_end();
       I != E; ++I, ++GTI) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      !hasMoreThanOneUseInLoop(ResultPtr, L);
  Value *FirstResult = nullptr;

  gep_type_iterator GTI = gep_type_begin(*Variadic);
  // Create an ugly GEP for each sequential index. We don't create GEPs for
  // structure indices, as they are accumulated in the constant offset index.
  for (unsigned I = 1, E = Variadic->getNumOperands(); I != E; ++I, ++GTI) {
    if (GTI.isSequential()) {
      Value *Idx = Variadic->getOperand(I);
      // Skip zero indices.
      if (ConstantInt *CI = dyn_cast<ConstantInt>(Idx))
        if (CI->isZero())
          continue;

```
- EN: Core entities appearing here include Find, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 Find，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1006-1104

```cpp
      APInt ElementSize = APInt(PtrIndexTy->getIntegerBitWidth(),
                                GTI.getSequentialElementStride(*DL));
      // Scale the index by element size.
      if (ElementSize != 1) {
        if (ElementSize.isPowerOf2()) {
          Idx = Builder.CreateShl(
              Idx, ConstantInt::get(PtrIndexTy, ElementSize.logBase2()));
        } else {
          Idx =
              Builder.CreateMul(Idx, ConstantInt::get(PtrIndexTy, ElementSize));
        }
      }
      // Create an ugly GEP with a single index for each index.
      ResultPtr = Builder.CreatePtrAdd(ResultPtr, Idx, "uglygep");
      if (FirstResult == nullptr)
        FirstResult = ResultPtr;
    }
  }

  // Create a GEP with the constant offset index.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    return false;

  // If the base of this GEP is a ptradd of a constant, lets pass the constant
  // along. This ensures that when we have a chain of GEPs the constant
  // offset from each is accumulated.
  Value *NewBase;
  const APInt *BaseOffset;
  bool ExtractBase = match(GEP->getPointerOperand(),
                           m_PtrAdd(m_Value(NewBase), m_APInt(BaseOffset)));

  unsigned IdxWidth = DL->getIndexTypeSizeInBits(GEP->getType());
  APInt BaseByteOffset =
      ExtractBase ? BaseOffset->sextOrTrunc(IdxWidth) : APInt(IdxWidth, 0);

```
- EN: Core entities appearing here include splitGEP, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 splitGEP，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1105-1206

```cpp
  // The backend can already nicely handle the case where all indices are
  // constant.
  if (GEP->hasAllConstantIndices() && !ExtractBase)
    return false;

  bool Changed = canonicalizeArrayIndicesToIndexSize(GEP);

  bool NeedsExtraction, OffsetOverflow;
  APInt NonBaseByteOffset =
      accumulateByteOffset(GEP, NeedsExtraction, OffsetOverflow);
  bool AddOverflow;
  APInt AccumulativeByteOffset =
      BaseByteOffset.sadd_ov(NonBaseByteOffset, AddOverflow);
  OffsetOverflow |= AddOverflow;

  TargetTransformInfo &TTI = GetTTI(*GEP->getFunction());

  if (!NeedsExtraction && !ExtractBase) {
    Changed |= reorderGEP(GEP, TTI);
    return Changed;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        Idx = NewIdx;
        AllNUWPreserved &= PreservesNUW;
      }
      AllOffsetsNonNegative =
          AllOffsetsNonNegative && isKnownNonNegative(Idx, *DL);
    }
  }
  if (ExtractBase) {
    GEPOperator *Base = cast<GEPOperator>(GEP->getPointerOperand());
    AllNUWPreserved &= Base->hasNoUnsignedWrap();
    NewGEPInBounds &= Base->isInBounds();
    NewGEPNUSW &= Base->hasNoUnsignedSignedWrap();
    AllOffsetsNonNegative &= BaseByteOffset.isNonNegative();

```
- EN: Legality, profitability, or canonical-form checks in this region depend on analyses such as TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段中的合法性、收益性或规范形态检查依赖 TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1207-1312

```cpp
    GEP->setOperand(0, NewBase);
    RecursivelyDeleteTriviallyDeadInstructions(Base);
  }

  // Clear the inbounds attribute because the new index may be off-bound.
  // e.g.,
  //
  //   b     = add i64 a, 5
  //   addr  = gep inbounds float, float* p, i64 b
  //
  // is transformed to:
  //
  //   addr2 = gep float, float* p, i64 a ; inbounds removed
  //   addr  = gep float, float* addr2, i64 5 ; inbounds removed
  //
  // If a is -4, although the old index b is in bounds, the new index a is
  // off-bound. http://llvm.org/docs/LangRef.html#id181 says "if the
  // inbounds keyword is not present, the offsets are added to the base
  // address with silently-wrapping two's complement arithmetic".
  // Therefore, the final code will be a semantically equivalent.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

bool SeparateConstOffsetFromGEPLegacyPass::runOnFunction(Function &F) {
  if (skipFunction(F))
    return false;
  auto *DT = &getAnalysis<DominatorTreeWrapperPass>().getDomTree();
  auto *LI = &getAnalysis<LoopInfoWrapperPass>().getLoopInfo();
  auto *TLI = &getAnalysis<TargetLibraryInfoWrapperPass>().getTLI(F);
  auto GetTTI = [this](Function &F) -> TargetTransformInfo & {
    return this->getAnalysis<TargetTransformInfoWrapperPass>().getTTI(F);
  };
  SeparateConstOffsetFromGEP Impl(DT, LI, TLI, GetTTI, LowerGEP);
  return Impl.run(F);
}

```
- EN: Core entities appearing here include runOnFunction, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, LoopInfo, TargetLibraryInfo, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 runOnFunction，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, LoopInfo, TargetLibraryInfo, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1313-1422

```cpp
bool SeparateConstOffsetFromGEP::run(Function &F) {
  if (DisableSeparateConstOffsetFromGEP)
    return false;

  DL = &F.getDataLayout();
  bool Changed = false;

  ReversePostOrderTraversal<Function *> RPOT(&F);
  for (BasicBlock *B : RPOT) {
    if (!DT->isReachableFromEntry(B))
      continue;

    for (Instruction &I : llvm::make_early_inc_range(*B))
      if (GetElementPtrInst *GEP = dyn_cast<GetElementPtrInst>(&I))
        Changed |= splitGEP(GEP);
    // No need to split GEP ConstantExprs because all its indices are constant
    // already.
  }

  Changed |= reuniteExts(F);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
}

bool SeparateConstOffsetFromGEP::reuniteExts(Function &F) {
  bool Changed = false;
  DominatingAdds.clear();
  DominatingSubs.clear();
  for (const auto Node : depth_first(DT)) {
    BasicBlock *BB = Node->getBlock();
    for (Instruction &I : llvm::make_early_inc_range(*BB))
      Changed |= reuniteExts(&I);
  }
  return Changed;
}

```
- EN: Core entities appearing here include run, reuniteExts, findClosestMatchingDominator, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 run, reuniteExts, findClosestMatchingDominator，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1423-1531

```cpp
void SeparateConstOffsetFromGEP::verifyNoDeadCode(Function &F) {
  for (BasicBlock &B : F) {
    for (Instruction &I : B) {
      if (isInstructionTriviallyDead(&I)) {
        std::string ErrMessage;
        raw_string_ostream RSO(ErrMessage);
        RSO << "Dead instruction detected!\n" << I << "\n";
        llvm_unreachable(RSO.str().c_str());
      }
    }
  }
}

bool SeparateConstOffsetFromGEP::isLegalToSwapOperand(
    GetElementPtrInst *FirstGEP, GetElementPtrInst *SecondGEP, Loop *CurLoop) {
  if (!FirstGEP || !FirstGEP->hasOneUse())
    return false;

  if (!SecondGEP || FirstGEP->getParent() != SecondGEP->getParent())
    return false;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                   cast<PointerType>(First->getType())->getAddressSpace()),
               0);
  Value *NewBase =
      First->stripAndAccumulateInBoundsConstantOffsets(DAL, Offset);
  uint64_t ObjectSize;
  if (!getObjectSize(NewBase, ObjectSize, DAL, TLI) ||
     Offset.ugt(ObjectSize)) {
    // TODO(gep_nowrap): Make flag preservation more precise.
    First->setNoWrapFlags(GEPNoWrapFlags::none());
    Second->setNoWrapFlags(GEPNoWrapFlags::none());
  } else
    First->setIsInBounds(true);
}

```
- EN: Core entities appearing here include verifyNoDeadCode, hasMoreThanOneUseInLoop, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 verifyNoDeadCode, hasMoreThanOneUseInLoop，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1532-1556

```cpp
void SeparateConstOffsetFromGEPPass::printPipeline(
    raw_ostream &OS, function_ref<StringRef(StringRef)> MapClassName2PassName) {
  static_cast<PassInfoMixin<SeparateConstOffsetFromGEPPass> *>(this)
      ->printPipeline(OS, MapClassName2PassName);
  OS << '<';
  if (LowerGEP)
    OS << "lower-gep";
  OS << '>';
}

PreservedAnalyses
SeparateConstOffsetFromGEPPass::run(Function &F, FunctionAnalysisManager &AM) {
  auto *DT = &AM.getResult<DominatorTreeAnalysis>(F);
  auto *LI = &AM.getResult<LoopAnalysis>(F);
  auto *TLI = &AM.getResult<TargetLibraryAnalysis>(F);
  auto GetTTI = [&AM](Function &F) -> TargetTransformInfo & {
    return AM.getResult<TargetIRAnalysis>(F);
  };
  SeparateConstOffsetFromGEP Impl(DT, LI, TLI, GetTTI, LowerGEP);
  if (!Impl.run(F))
    return PreservedAnalyses::all();
  PreservedAnalyses PA;
  PA.preserveSet<CFGAnalyses>();
  return PA;
}
```
- EN: Core entities appearing here include run, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DominatorTree, TargetTransformInfo. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DominatorTree, TargetTransformInfo 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `ConstantOffsetExtractor, SeparateConstOffsetFromGEPLegacyPass, SeparateConstOffsetFromGEP, FunctionPass, createNormalizedCommutablePair, distributeCastsAndCloneChain, allowsPreservingNUW, Find` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`ConstantOffsetExtractor, SeparateConstOffsetFromGEPLegacyPass, SeparateConstOffsetFromGEP, FunctionPass, createNormalizedCommutablePair, distributeCastsAndCloneChain, allowsPreservingNUW, Find` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, DominatorTree, LoopInfo, ScalarEvolution, TargetLibraryInfo, TargetTransformInfo, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, DominatorTree, LoopInfo, ScalarEvolution, TargetLibraryInfo, TargetTransformInfo, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `DisableSeparateConstOffsetFromGEP, VerifyNoDeadCode` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `DisableSeparateConstOffsetFromGEP, VerifyNoDeadCode` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/LoopInfo.h`, `llvm/Analysis/MemoryBuiltins.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/LoopInfo.h`, `llvm/Analysis/MemoryBuiltins.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/TargetTransformInfo.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/BasicBlock.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/GetElementPtrTypeIterator.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/BasicBlock.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/DataLayout.h`, `llvm/IR/DerivedTypes.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/GetElementPtrTypeIterator.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/APInt.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/DepthFirstIterator.h`, `llvm/ADT/SmallVector.h`, `llvm/InitializePasses.h`, `llvm/Pass.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/APInt.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/DepthFirstIterator.h`, `llvm/ADT/SmallVector.h`, `llvm/InitializePasses.h`, `llvm/Pass.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `cassert`, `cstdint`, `string` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`cassert`, `cstdint`, `string` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout`, `DominatorTree`, `LoopInfo`, `ScalarEvolution`, `TargetLibraryInfo`, `TargetTransformInfo`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `DominatorTree`, `LoopInfo`, `ScalarEvolution`, `TargetLibraryInfo`, `TargetTransformInfo`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
