# ScopDetection.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `polly/include/polly/ScopDetection.h` | `polly/include/polly/ScopDetection.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Declares public Polly interfaces, passes, analyses, and data structures. The leading comment describes it as: Detect Scops. | 声明 Polly 的公共接口、Pass、分析能力与数据结构。 文件开头注释将其概括为：Detect Scops。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===- ScopDetection.h - Detect Scops ---------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// Detect the maximal Scops of a function.
//
// A static control part (Scop) is a subgraph of the control flow graph (CFG)
// that only has statically known control flow and can therefore be described
// within the polyhedral model.
//
// Every Scop fulfills these restrictions:
//
// * It is a single entry single exit region
//
// * Only affine linear bounds in the loops
//
// Every natural loop in a Scop must have a number of loop iterations that can
// be described as an affine linear function in surrounding loop iterators or
// parameters. (A parameter is a scalar that does not change its value during
// execution of the Scop).
````
- **EN**: This block records the standard LLVM file banner and license metadata; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 给出 LLVM 标准文件头与许可证元数据; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 25-45

````cpp
//
// * Only comparisons of affine linear expressions in conditions
//
// * All loops and conditions perfectly nested
//
// The control flow needs to be structured such that it could be written using
// just 'for' and 'if' statements, without the need for any 'goto', 'break' or
// 'continue'.
//
// * Side effect free functions call
//
// Only function calls and intrinsics that do not have side effects are allowed
// (readnone).
//
// The Scop detection finds the largest Scops by checking if the largest
// region is a Scop. If this is not the case, its canonical subregions are
// checked until a region is a Scop. It is now tried to extend this Scop by
// creating a larger non canonical region.
//
//===----------------------------------------------------------------------===//

````
- **EN**: This block touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 46-69

````cpp
#ifndef POLLY_SCOPDETECTION_H
#define POLLY_SCOPDETECTION_H

#include "polly/ScopDetectionDiagnostic.h"
#include "polly/Support/ScopHelper.h"
#include "llvm/Analysis/AliasAnalysis.h"
#include "llvm/Analysis/AliasSetTracker.h"
#include "llvm/Analysis/RegionInfo.h"
#include "llvm/Analysis/ScalarEvolutionExpressions.h"
#include <set>

namespace polly {
using llvm::AAResults;
using llvm::AliasSetTracker;
using llvm::AnalysisInfoMixin;
using llvm::AnalysisKey;
using llvm::AnalysisUsage;
using llvm::BatchAAResults;
using llvm::CallInst;
using llvm::CondBrInst;
using llvm::DenseMap;
using llvm::DominatorTree;
using llvm::Function;
using llvm::FunctionAnalysisManager;
````
- **EN**: This block imports Polly, LLVM-family, system/standard headers needed by the surrounding code; uses preprocessor directives to control compilation, macros, or include guards; opens or organizes declarations inside a C++ namespace; defines macros like `POLLY_SCOPDETECTION_H`; and continues the surrounding implementation details.
- **CN**: 该代码块 引入周边逻辑所需的 Polly、LLVM-family、system/standard 头文件; 使用预处理指令控制编译、宏或头文件保护; 在 C++ 命名空间中组织声明或实现; 定义宏，例如 `POLLY_SCOPDETECTION_H`；并延续周边实现细节。

### Lines 70-92

````cpp
using llvm::IntrinsicInst;
using llvm::LoopInfo;
using llvm::Module;
using llvm::OptimizationRemarkEmitter;
using llvm::OptionalPassInfoMixin;
using llvm::PreservedAnalyses;
using llvm::RegionInfo;
using llvm::RequiredPassInfoMixin;
using llvm::ScalarEvolution;
using llvm::SCEVUnknown;
using llvm::SetVector;
using llvm::SmallSetVector;
using llvm::SmallVectorImpl;
using llvm::StringRef;
using llvm::SwitchInst;

using ParamSetType = std::set<const SCEV *>;

// Description of the shape of an array.
struct ArrayShape {
  // Base pointer identifying all accesses to this array.
  const SCEVUnknown *BasePointer;

````
- **EN**: This block declares or references types such as `ArrayShape`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或引用类型，例如 `ArrayShape`; 保留用于解释意图、用法或算法背景的注释.

### Lines 93-117

````cpp
  // Sizes of each delinearized dimension.
  SmallVector<const SCEV *, 4> DelinearizedSizes;

  ArrayShape(const SCEVUnknown *B) : BasePointer(B) {}
};

struct MemAcc {
  const Instruction *Insn;

  // A pointer to the shape description of the array.
  std::shared_ptr<ArrayShape> Shape;

  // Subscripts computed by delinearization.
  SmallVector<const SCEV *, 4> DelinearizedSubscripts;

  MemAcc(const Instruction *I, std::shared_ptr<ArrayShape> S)
      : Insn(I), Shape(S) {}
};

using MapInsnToMemAcc = std::map<const Instruction *, MemAcc>;
using PairInstSCEV = std::pair<const Instruction *, const SCEV *>;
using AFs = std::vector<PairInstSCEV>;
using BaseToAFs = std::map<const SCEVUnknown *, AFs>;
using BaseToElSize = std::map<const SCEVUnknown *, const SCEV *>;

````
- **EN**: This block declares or references types such as `MemAcc`; declares or defines routines around `ArrayShape`, `MemAcc`, `Insn`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或引用类型，例如 `MemAcc`; 声明或定义与 `ArrayShape`, `MemAcc`, `Insn` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 118-138

````cpp
extern bool PollyTrackFailures;
extern bool PollyDelinearize;
extern bool PollyUseRuntimeAliasChecks;
extern bool PollyProcessUnprofitable;
extern bool PollyInvariantLoadHoisting;
extern bool PollyAllowUnsignedOperations;
extern bool PollyAllowFullFunction;

/// A function attribute which will cause Polly to skip the function
extern StringRef PollySkipFnAttr;

//===----------------------------------------------------------------------===//
/// Pass to detect the maximal static control parts (Scops) of a
/// function.
class ScopDetection {
public:
  using RegionSet = SetVector<const Region *>;

  // Remember the valid regions
  RegionSet ValidRegions;

````
- **EN**: This block declares or references types such as `ScopDetection`; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或引用类型，例如 `ScopDetection`; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 139-158

````cpp
  /// Context variables for SCoP detection.
  struct DetectionContext {
    Region &CurRegion;   // The region to check.
    BatchAAResults BAA;  // The batched alias analysis results.
    AliasSetTracker AST; // The AliasSetTracker to hold the alias information.
    bool Verifying;      // If we are in the verification phase?

    /// If this flag is set, the SCoP must eventually be rejected, even with
    /// KeepGoing.
    bool IsInvalid = false;

    /// Container to remember rejection reasons for this region.
    RejectLog Log;

    /// Map a base pointer to all access functions accessing it.
    ///
    /// This map is indexed by the base pointer. Each element of the map
    /// is a list of memory accesses that reference this base pointer.
    BaseToAFs Accesses;

````
- **EN**: This block declares or references types such as `DetectionContext`; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或引用类型，例如 `DetectionContext`; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 159-180

````cpp
    /// The set of base pointers with non-affine accesses.
    ///
    /// This set contains all base pointers and the locations where they are
    /// used for memory accesses that can not be detected as affine accesses.
    llvm::SetVector<std::pair<const SCEVUnknown *, Loop *>> NonAffineAccesses;
    BaseToElSize ElementSize;

    /// The region has at least one load instruction.
    bool hasLoads = false;

    /// The region has at least one store instruction.
    bool hasStores = false;

    /// Flag to indicate the region has at least one unknown access.
    bool HasUnknownAccess = false;

    /// The set of non-affine subregions in the region we analyze.
    RegionSet NonAffineSubRegionSet;

    /// The set of loops contained in non-affine regions.
    BoxedLoopsSetTy BoxedLoopsSet;

````
- **EN**: This block preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 保留用于解释意图、用法或算法背景的注释.

### Lines 181-201

````cpp
    /// Loads that need to be invariant during execution.
    InvariantLoadsSetTy RequiredILS;

    /// Map to memory access description for the corresponding LLVM
    ///        instructions.
    MapInsnToMemAcc InsnToMemAcc;

    /// Initialize a DetectionContext from scratch.
    DetectionContext(Region &R, AAResults &AA, bool Verify)
        : CurRegion(R), BAA(AA), AST(BAA), Verifying(Verify), Log(&R) {}
  };

  /// Helper data structure to collect statistics about loop counts.
  struct LoopStats {
    int NumLoops;
    int MaxDepth;
  };

  int NextScopID = 0;
  int getNextID() { return NextScopID++; }

````
- **EN**: This block declares or references types such as `LoopStats`; declares or defines routines around `DetectionContext`, `CurRegion`, `getNextID`; emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或引用类型，例如 `LoopStats`; 声明或定义与 `DetectionContext`, `CurRegion`, `getNextID` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域；并延续周边实现细节。

### Lines 202-222

````cpp
private:
  //===--------------------------------------------------------------------===//

  /// Analyses used
  //@{
  const DominatorTree &DT;
  ScalarEvolution &SE;
  LoopInfo &LI;
  RegionInfo &RI;
  AAResults &AA;
  //@}

  /// Map to remember detection contexts for all regions.
  using DetectionContextMapTy =
      DenseMap<BBPair, std::unique_ptr<DetectionContext>>;
  DetectionContextMapTy DetectionContextMap;

  /// Cache for the isErrorBlock function.
  DenseMap<std::tuple<const BasicBlock *, const Region *>, bool>
      ErrorBlockCache;

````
- **EN**: This block preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 保留用于解释意图、用法或算法背景的注释.

### Lines 223-246

````cpp
  /// Remove cached results for @p R.
  void removeCachedResults(const Region &R);

  /// Remove cached results for the children of @p R recursively.
  void removeCachedResultsRecursively(const Region &R);

  /// Check if @p S0 and @p S1 do contain multiple possibly aliasing pointers.
  ///
  /// @param S0    A expression to check.
  /// @param S1    Another expression to check or nullptr.
  /// @param Scope The loop/scope the expressions are checked in.
  ///
  /// @returns True, if multiple possibly aliasing pointers are used in @p S0
  ///          (and @p S1 if given).
  bool involvesMultiplePtrs(const SCEV *S0, const SCEV *S1, Loop *Scope) const;

  /// Add the region @p AR as over approximated sub-region in @p Context.
  ///
  /// @param AR      The non-affine subregion.
  /// @param Context The current detection context.
  ///
  /// @returns True if the subregion can be over approximated, false otherwise.
  bool addOverApproximatedRegion(Region *AR, DetectionContext &Context) const;

````
- **EN**: This block declares or defines routines around `removeCachedResults`, `removeCachedResultsRecursively`, `involvesMultiplePtrs`, `addOverApproximatedRegion`; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `removeCachedResults`, `removeCachedResultsRecursively`, `involvesMultiplePtrs`, `addOverApproximatedRegion` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 247-268

````cpp
  /// Find for a given base pointer terms that hint towards dimension
  ///        sizes of a multi-dimensional array.
  ///
  /// @param Context      The current detection context.
  /// @param BasePointer  A base pointer indicating the virtual array we are
  ///                     interested in.
  SmallVector<const SCEV *, 4>
  getDelinearizationTerms(DetectionContext &Context,
                          const SCEVUnknown *BasePointer) const;

  /// Check if the dimension size of a delinearized array is valid.
  ///
  /// @param Context     The current detection context.
  /// @param Sizes       The sizes of the different array dimensions.
  /// @param BasePointer The base pointer we are interested in.
  /// @param Scope       The location where @p BasePointer is being used.
  /// @returns True if one or more array sizes could be derived - meaning: we
  ///          see this array as multi-dimensional.
  bool hasValidArraySizes(DetectionContext &Context,
                          SmallVectorImpl<const SCEV *> &Sizes,
                          const SCEVUnknown *BasePointer, Loop *Scope) const;

````
- **EN**: This block declares or defines routines around `getDelinearizationTerms`, `hasValidArraySizes`; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getDelinearizationTerms`, `hasValidArraySizes` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 269-292

````cpp
  /// Derive access functions for a given base pointer.
  ///
  /// @param Context     The current detection context.
  /// @param Sizes       The sizes of the different array dimensions.
  /// @param BasePointer The base pointer of all the array for which to compute
  ///                    access functions.
  /// @param Shape       The shape that describes the derived array sizes and
  ///                    which should be filled with newly computed access
  ///                    functions.
  /// @returns True if a set of affine access functions could be derived.
  bool computeAccessFunctions(DetectionContext &Context,
                              const SCEVUnknown *BasePointer,
                              std::shared_ptr<ArrayShape> Shape) const;

  /// Check if all accesses to a given BasePointer are affine.
  ///
  /// @param Context     The current detection context.
  /// @param BasePointer the base pointer we are interested in.
  /// @param Scope       The location where @p BasePointer is being used.
  /// @param True if consistent (multi-dimensional) array accesses could be
  ///        derived for this array.
  bool hasBaseAffineAccesses(DetectionContext &Context,
                             const SCEVUnknown *BasePointer, Loop *Scope) const;

````
- **EN**: This block declares or defines routines around `computeAccessFunctions`, `hasBaseAffineAccesses`; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `computeAccessFunctions`, `hasBaseAffineAccesses` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 293-316

````cpp
  /// Delinearize all non affine memory accesses and return false when there
  /// exists a non affine memory access that cannot be delinearized. Return true
  /// when all array accesses are affine after delinearization.
  bool hasAffineMemoryAccesses(DetectionContext &Context) const;

  /// Try to expand the region R. If R can be expanded return the expanded
  /// region, NULL otherwise.
  Region *expandRegion(Region &R);

  /// Find the Scops in this region tree.
  ///
  /// @param The region tree to scan for scops.
  void findScops(Region &R);

  /// Check if all basic block in the region are valid.
  ///
  /// @param Context The context of scop detection.
  bool allBlocksValid(DetectionContext &Context);

  /// Check if a region has sufficient compute instructions.
  ///
  /// This function checks if a region has a non-trivial number of instructions
  /// in each loop. This can be used as an indicator whether a loop is worth
  /// optimizing.
````
- **EN**: This block declares or defines routines around `hasAffineMemoryAccesses`, `expandRegion`, `findScops`, `allBlocksValid`; emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; works with memory-access metadata or access relations; and continues the surrounding implementation details.
- **CN**: 该代码块 声明或定义与 `hasAffineMemoryAccesses`, `expandRegion`, `findScops`, `allBlocksValid` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 处理内存访问元数据或访问关系；并延续周边实现细节。

### Lines 317-336

````cpp
  ///
  /// @param Context  The context of scop detection.
  /// @param NumLoops The number of loops in the region.
  ///
  /// @return True if region is has sufficient compute instructions,
  ///         false otherwise.
  bool hasSufficientCompute(DetectionContext &Context,
                            int NumAffineLoops) const;

  /// Check if the unique affine loop might be amendable to distribution.
  ///
  /// This function checks if the number of non-trivial blocks in the unique
  /// affine loop in Context.CurRegion is at least two, thus if the loop might
  /// be amendable to distribution.
  ///
  /// @param Context  The context of scop detection.
  ///
  /// @return True only if the affine loop might be amendable to distributable.
  bool hasPossiblyDistributableLoop(DetectionContext &Context) const;

````
- **EN**: This block declares or defines routines around `hasSufficientCompute`, `hasPossiblyDistributableLoop`; emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `hasSufficientCompute`, `hasPossiblyDistributableLoop` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 337-361

````cpp
  /// Check if a region is profitable to optimize.
  ///
  /// Regions that are unlikely to expose interesting optimization opportunities
  /// are called 'unprofitable' and may be skipped during scop detection.
  ///
  /// @param Context The context of scop detection.
  ///
  /// @return True if region is profitable to optimize, false otherwise.
  bool isProfitableRegion(DetectionContext &Context) const;

  /// Check if a region is a Scop.
  ///
  /// @param Context The context of scop detection.
  ///
  /// @return If we short-circuited early to not waste time on known-invalid
  ///         SCoPs. Use Context.IsInvalid to determine whether the region is a
  ///         valid SCoP.
  bool isValidRegion(DetectionContext &Context);

  /// Check if an intrinsic call can be part of a Scop.
  ///
  /// @param II      The intrinsic call instruction to check.
  /// @param Context The current detection context.
  bool isValidIntrinsicInst(IntrinsicInst &II, DetectionContext &Context) const;

````
- **EN**: This block declares or defines routines around `isProfitableRegion`, `isValidRegion`, `isValidIntrinsicInst`; emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isProfitableRegion`, `isValidRegion`, `isValidIntrinsicInst` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 362-389

````cpp
  /// Check if a call instruction can be part of a Scop.
  ///
  /// @param CI      The call instruction to check.
  /// @param Context The current detection context.
  bool isValidCallInst(CallInst &CI, DetectionContext &Context) const;

  /// Check if the given loads could be invariant and can be hoisted.
  ///
  /// If true is returned the loads are added to the required invariant loads
  /// contained in the @p Context.
  ///
  /// @param RequiredILS The loads to check.
  /// @param Context     The current detection context.
  ///
  /// @return True if all loads can be assumed invariant.
  bool onlyValidRequiredInvariantLoads(InvariantLoadsSetTy &RequiredILS,
                                       DetectionContext &Context) const;

  /// Check if a value is invariant in the region Reg.
  ///
  /// @param Val Value to check for invariance.
  /// @param Reg The region to consider for the invariance of Val.
  /// @param Ctx The current detection context.
  ///
  /// @return True if the value represented by Val is invariant in the region
  ///         identified by Reg.
  bool isInvariant(Value &Val, const Region &Reg, DetectionContext &Ctx) const;

````
- **EN**: This block declares or defines routines around `isValidCallInst`, `onlyValidRequiredInvariantLoads`, `isInvariant`; emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isValidCallInst`, `onlyValidRequiredInvariantLoads`, `isInvariant` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 390-414

````cpp
  /// Check if the memory access caused by @p Inst is valid.
  ///
  /// @param Inst    The access instruction.
  /// @param AF      The access function.
  /// @param BP      The access base pointer.
  /// @param Context The current detection context.
  bool isValidAccess(Instruction *Inst, const SCEV *AF, const SCEVUnknown *BP,
                     DetectionContext &Context) const;

  /// Check if a memory access can be part of a Scop.
  ///
  /// @param Inst The instruction accessing the memory.
  /// @param Context The context of scop detection.
  bool isValidMemoryAccess(MemAccInst Inst, DetectionContext &Context) const;

  /// Filter out types that we do not support.
  bool isCompatibleType(Instruction *Inst, llvm::Type *Ty,
                        DetectionContext &Context);

  /// Check if an instruction can be part of a Scop.
  ///
  /// @param Inst The instruction to check.
  /// @param Context The context of scop detection.
  bool isValidInstruction(Instruction &Inst, DetectionContext &Context);

````
- **EN**: This block declares or defines routines around `isValidAccess`, `isValidMemoryAccess`, `isCompatibleType`, `isValidInstruction`; touches Polly SCoP abstractions that model analyzable regions; works with memory-access metadata or access relations; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isValidAccess`, `isValidMemoryAccess`, `isCompatibleType`, `isValidInstruction` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 处理内存访问元数据或访问关系; 保留用于解释意图、用法或算法背景的注释.

### Lines 415-434

````cpp
  /// Check if the switch @p SI with condition @p Condition is valid.
  ///
  /// @param BB           The block to check.
  /// @param SI           The switch to check.
  /// @param Condition    The switch condition.
  /// @param IsLoopBranch Flag to indicate the branch is a loop exit/latch.
  /// @param Context      The context of scop detection.
  bool isValidSwitch(BasicBlock &BB, SwitchInst *SI, Value *Condition,
                     bool IsLoopBranch, DetectionContext &Context) const;

  /// Check if the branch @p BI with condition @p Condition is valid.
  ///
  /// @param BB           The block to check.
  /// @param BI           The branch to check.
  /// @param Condition    The branch condition.
  /// @param IsLoopBranch Flag to indicate the branch is a loop exit/latch.
  /// @param Context      The context of scop detection.
  bool isValidBranch(BasicBlock &BB, CondBrInst *BI, Value *Condition,
                     bool IsLoopBranch, DetectionContext &Context);

````
- **EN**: This block declares or defines routines around `isValidSwitch`, `isValidBranch`; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isValidSwitch`, `isValidBranch` 相关的例程; 保留用于解释意图、用法或算法背景的注释.

### Lines 435-460

````cpp
  /// Check if the SCEV @p S is affine in the current @p Context.
  ///
  /// This will also use a heuristic to decide if we want to require loads to be
  /// invariant to make the expression affine or if we want to treat is as
  /// non-affine.
  ///
  /// @param S           The expression to be checked.
  /// @param Scope       The loop nest in which @p S is used.
  /// @param Context     The context of scop detection.
  bool isAffine(const SCEV *S, Loop *Scope, DetectionContext &Context) const;

  /// Check if the control flow in a basic block is valid.
  ///
  /// This function checks if a certain basic block is terminated by a
  /// Terminator instruction we can handle or, if this is not the case,
  /// registers this basic block as the start of a non-affine region.
  ///
  /// This function optionally allows unreachable statements.
  ///
  /// @param BB               The BB to check the control flow.
  /// @param IsLoopBranch     Flag to indicate the branch is a loop exit/latch.
  ///  @param AllowUnreachable Allow unreachable statements.
  /// @param Context          The context of scop detection.
  bool isValidCFG(BasicBlock &BB, bool IsLoopBranch, bool AllowUnreachable,
                  DetectionContext &Context);

````
- **EN**: This block declares or defines routines around `isAffine`, `isValidCFG`; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isAffine`, `isValidCFG` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 461-483

````cpp
  /// Is a loop valid with respect to a given region.
  ///
  /// @param L The loop to check.
  /// @param Context The context of scop detection.
  bool isValidLoop(Loop *L, DetectionContext &Context);

  /// Count the number of loops and the maximal loop depth in @p L.
  ///
  /// @param L The loop to check.
  /// @param SE The scalar evolution analysis.
  /// @param MinProfitableTrips The minimum number of trip counts from which
  ///                           a loop is assumed to be profitable and
  ///                           consequently is counted.
  /// returns A tuple of number of loops and their maximal depth.
  static ScopDetection::LoopStats
  countBeneficialSubLoops(Loop *L, ScalarEvolution &SE,
                          unsigned MinProfitableTrips);

  /// Check if the function @p F is marked as invalid.
  ///
  /// @note An OpenMP subfunction will be marked as invalid.
  static bool isValidFunction(Function &F);

````
- **EN**: This block declares or defines routines around `isValidLoop`, `countBeneficialSubLoops`, `isValidFunction`; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isValidLoop`, `countBeneficialSubLoops`, `isValidFunction` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 484-503

````cpp
  /// Can ISL compute the trip count of a loop.
  ///
  /// @param L The loop to check.
  /// @param Context The context of scop detection.
  ///
  /// @return True if ISL can compute the trip count of the loop.
  bool canUseISLTripCount(Loop *L, DetectionContext &Context);

  /// Print the locations of all detected scops.
  void printLocations(Function &F);

  /// Check if a region is reducible or not.
  ///
  /// @param Region The region to check.
  /// @param DbgLoc Parameter to save the location of instruction that
  ///               causes irregular control flow if the region is irreducible.
  ///
  /// @return True if R is reducible, false otherwise.
  bool isReducibleRegion(Region &R, DebugLoc &DbgLoc) const;

````
- **EN**: This block declares or defines routines around `canUseISLTripCount`, `printLocations`, `isReducibleRegion`; emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `canUseISLTripCount`, `printLocations`, `isReducibleRegion` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 504-523

````cpp
  /// Track diagnostics for invalid scops.
  ///
  /// @param Context The context of scop detection.
  /// @param Assert Throw an assert in verify mode or not.
  /// @param Args Argument list that gets passed to the constructor of RR.
  template <class RR, typename... Args>
  inline bool invalid(DetectionContext &Context, bool Assert,
                      Args &&...Arguments) const;

public:
  ScopDetection(const DominatorTree &DT, ScalarEvolution &SE, LoopInfo &LI,
                RegionInfo &RI, AAResults &AA, OptimizationRemarkEmitter &ORE);

  void detect(Function &F);

  /// Get the RegionInfo stored in this pass.
  ///
  /// This was added to give the DOT printer easy access to this information.
  RegionInfo *getRI() const { return &RI; }

````
- **EN**: This block declares or defines routines around `invalid`, `ScopDetection`, `detect`, `getRI`; emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `invalid`, `ScopDetection`, `detect`, `getRI` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 524-549

````cpp
  /// Get the LoopInfo stored in this pass.
  LoopInfo *getLI() const { return &LI; }

  /// Is the region is the maximum region of a Scop?
  ///
  /// @param R The Region to test if it is maximum.
  /// @param Verify Rerun the scop detection to verify SCoP was not invalidated
  ///               meanwhile. Do not use if the region's DetectionContect is
  ///               referenced by a Scop that is still to be processed.
  ///
  /// @return Return true if R is the maximum Region in a Scop, false otherwise.
  bool isMaxRegionInScop(const Region &R, bool Verify = true);

  /// Return the detection context for @p R, nullptr if @p R was invalid.
  DetectionContext *getDetectionContext(const Region *R) const;

  /// Return the set of rejection causes for @p R.
  const RejectLog *lookupRejectionLog(const Region *R) const;

  /// Get a message why a region is invalid
  ///
  /// @param R The region for which we get the error message
  ///
  /// @return The error or "" if no error appeared.
  std::string regionIsInvalidBecause(const Region *R) const;

````
- **EN**: This block declares or defines routines around `getLI`, `isMaxRegionInScop`, `getDetectionContext`, `lookupRejectionLog` (+1 more); emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `getLI`, `isMaxRegionInScop`, `getDetectionContext`, `lookupRejectionLog` (+1 more) 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 550-569

````cpp
  /// @name Maximum Region In Scops Iterators
  ///
  /// These iterators iterator over all maximum region in Scops of this
  /// function.
  //@{
  using iterator = RegionSet::iterator;
  using const_iterator = RegionSet::const_iterator;

  iterator begin() { return ValidRegions.begin(); }
  iterator end() { return ValidRegions.end(); }

  const_iterator begin() const { return ValidRegions.begin(); }
  const_iterator end() const { return ValidRegions.end(); }
  //@}

  /// Emit rejection remarks for all rejected regions.
  ///
  /// @param F The function to emit remarks for.
  void emitMissedRemarks(const Function &F);

````
- **EN**: This block declares or defines routines around `begin`, `end`, `emitMissedRemarks`; emits return paths that hand results or status codes back to callers; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `begin`, `end`, `emitMissedRemarks` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 570-596

````cpp
  /// Mark the function as invalid so we will not extract any scop from
  ///        the function.
  ///
  /// @param F The function to mark as invalid.
  static void markFunctionAsInvalid(Function *F);

  /// Verify if all valid Regions in this Function are still valid
  /// after some transformations.
  void verifyAnalysis();

  /// Verify if R is still a valid part of Scop after some transformations.
  ///
  /// @param R The Region to verify.
  void verifyRegion(const Region &R);

  /// Count the number of loops and the maximal loop depth in @p R.
  ///
  /// @param R The region to check
  /// @param SE The scalar evolution analysis.
  /// @param MinProfitableTrips The minimum number of trip counts from which
  ///                           a loop is assumed to be profitable and
  ///                           consequently is counted.
  /// returns A tuple of number of loops and their maximal depth.
  static ScopDetection::LoopStats
  countBeneficialLoops(Region *R, ScalarEvolution &SE, LoopInfo &LI,
                       unsigned MinProfitableTrips);

````
- **EN**: This block declares or defines routines around `markFunctionAsInvalid`, `verifyAnalysis`, `verifyRegion`, `countBeneficialLoops`; touches Polly SCoP abstractions that model analyzable regions; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `markFunctionAsInvalid`, `verifyAnalysis`, `verifyRegion`, `countBeneficialLoops` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域; 保留用于解释意图、用法或算法背景的注释.

### Lines 597-619

````cpp
  /// Check if the block is a error block.
  ///
  /// A error block is currently any block that fulfills at least one of
  /// the following conditions:
  ///
  ///  - It is terminated by an unreachable instruction
  ///  - It contains a call to a non-pure function that is not immediately
  ///    dominated by a loop header and that does not dominate the region exit.
  ///    This is a heuristic to pick only error blocks that are conditionally
  ///    executed and can be assumed to be not executed at all without the
  ///    domains being available.
  ///
  /// @param BB The block to check.
  /// @param R  The analyzed region.
  ///
  /// @return True if the block is a error block, false otherwise.
  bool isErrorBlock(llvm::BasicBlock &BB, const llvm::Region &R);

private:
  /// OptimizationRemarkEmitter object used to emit diagnostic remarks
  OptimizationRemarkEmitter &ORE;
};

````
- **EN**: This block declares or defines routines around `isErrorBlock`; emits return paths that hand results or status codes back to callers; preserves comments that explain intent, usage, or algorithmic background.
- **CN**: 该代码块 声明或定义与 `isErrorBlock` 相关的例程; 包含返回路径，用于向调用者交回结果或状态码; 保留用于解释意图、用法或算法背景的注释.

### Lines 620-640

````cpp
struct ScopAnalysis : AnalysisInfoMixin<ScopAnalysis> {
  static AnalysisKey Key;

  using Result = ScopDetection;

  ScopAnalysis();

  Result run(Function &F, FunctionAnalysisManager &FAM);
};

struct ScopAnalysisPrinterPass final
    : RequiredPassInfoMixin<ScopAnalysisPrinterPass> {
  ScopAnalysisPrinterPass(raw_ostream &OS) : OS(OS) {}

  PreservedAnalyses run(Function &F, FunctionAnalysisManager &FAM);

  raw_ostream &OS;
};
} // namespace polly

#endif // POLLY_SCOPDETECTION_H
````
- **EN**: This block uses preprocessor directives to control compilation, macros, or include guards; declares or references types such as `ScopAnalysis`, `ScopAnalysisPrinterPass`; declares or defines routines around `ScopAnalysis`, `run`, `ScopAnalysisPrinterPass`; touches Polly SCoP abstractions that model analyzable regions.
- **CN**: 该代码块 使用预处理指令控制编译、宏或头文件保护; 声明或引用类型，例如 `ScopAnalysis`, `ScopAnalysisPrinterPass`; 声明或定义与 `ScopAnalysis`, `run`, `ScopAnalysisPrinterPass` 相关的例程; 涉及 Polly 的 SCoP 抽象，用于表示可分析区域.

## Key Concepts / 关键概念

- **SCoP modeling**
  - **CN**: SCoP 建模
- **Loop transformation**
  - **CN**: 循环变换
- **Memory access tracking**
  - **CN**: 内存访问跟踪
- **Template/type wrappers**
  - **CN**: 模板与类型包装
- **LLVM pass integration**
  - **CN**: LLVM Pass 集成
- **OpenMP/runtime support**
  - **CN**: OpenMP/运行时支持

## Dependencies / 依赖关系

- **Polly headers**: `polly/ScopDetectionDiagnostic.h`, `polly/Support/ScopHelper.h` — Direct Polly APIs, data structures, or pass interfaces used here.
  **Polly headers（CN）**：`polly/ScopDetectionDiagnostic.h`, `polly/Support/ScopHelper.h` —— 此处直接使用的 Polly API、数据结构或 Pass 接口。
- **LLVM-family headers**: `llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/AliasSetTracker.h`, `llvm/Analysis/RegionInfo.h`, `llvm/Analysis/ScalarEvolutionExpressions.h` — LLVM/Clang/MLIR infrastructure for IR, passes, utilities, or diagnostics.
  **LLVM-family headers（CN）**：`llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/AliasSetTracker.h`, `llvm/Analysis/RegionInfo.h`, `llvm/Analysis/ScalarEvolutionExpressions.h` —— 用于 IR、Pass、工具或诊断的 LLVM/Clang/MLIR 基础设施。
- **System/standard headers**: `set` — Standard-library or system declarations required by the implementation.
  **System/standard headers（CN）**：`set` —— 实现所需的标准库或系统声明。
