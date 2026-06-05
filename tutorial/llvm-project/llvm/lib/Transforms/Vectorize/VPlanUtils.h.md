# VPlanUtils.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/VPlanUtils.h` | `llvm/lib/Transforms/Vectorize/VPlanUtils.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This header defines the public interface for vPlan-related utilities within LLVM's vectorization and VPlan infrastructure layer. | 该头文件定义了对外接口 LLVM 向量化与 VPlan 基础设施层中的 VPlanUtils 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-41

```cpp
//===- VPlanUtils.h - VPlan-related utilities -------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_TRANSFORMS_VECTORIZE_VPLANUTILS_H
#define LLVM_TRANSFORMS_VECTORIZE_VPLANUTILS_H

#include "VPlan.h"
#include "VPlanPatternMatch.h"
#include "llvm/Support/Compiler.h"

namespace llvm {
class MemoryLocation;
class ScalarEvolution;
class SCEV;
class PredicatedScalarEvolution;
} // namespace llvm

namespace llvm {

namespace vputils {
/// Returns true if only the first lane of \p Def is used.
bool onlyFirstLaneUsed(const VPValue *Def);

/// Returns true if only the first part of \p Def is used.
bool onlyFirstPartUsed(const VPValue *Def);

/// Returns true if only scalar values of \p Def are used by all users.
bool onlyScalarValuesUsed(const VPValue *Def);

/// Get or create a VPValue that corresponds to the expansion of \p Expr. If \p
/// Expr is a SCEVConstant or SCEVUnknown, return a VPValue wrapping the live-in
/// value. Otherwise return a VPExpandSCEVRecipe to expand \p Expr. If \p Plan's
/// pre-header already contains a recipe expanding \p Expr, return it. If not,
/// create a new one.
VPValue *getOrCreateVPValueForSCEVExpr(VPlan &Plan, const SCEV *Expr);

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. The header guard ensures the declarations are safe to include from multiple compilation units. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 头文件保护宏保证这些声明可以被多个编译单元安全地重复包含。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。

### Lines 42-79

```cpp
/// Return the SCEV expression for \p V. Returns SCEVCouldNotCompute if no
/// SCEV expression could be constructed.
const SCEV *getSCEVExprForVPValue(const VPValue *V,
                                  PredicatedScalarEvolution &PSE,
                                  const Loop *L = nullptr);

/// Returns true if \p Addr is an address SCEV that can be passed to
/// TTI::getAddressComputationCost, i.e. the address SCEV is loop invariant, an
/// affine AddRec (i.e. induction ), or an add expression of such operands or a
/// sign-extended AddRec.
bool isAddressSCEVForCost(const SCEV *Addr, ScalarEvolution &SE, const Loop *L);

/// Returns true if \p VPV is a single scalar, either because it produces the
/// same value for all lanes or only has its first lane used.
bool isSingleScalar(const VPValue *VPV);

/// Return true if \p V is a header mask in \p Plan.
bool isHeaderMask(const VPValue *V, const VPlan &Plan);

/// Checks if \p V is uniform across all VF lanes and UF parts. It is considered
/// as such if it is either loop invariant (defined outside the vector region)
/// or its operands are known to be uniform across all VFs and UFs (e.g.
/// VPDerivedIV or the canonical IV).
bool isUniformAcrossVFsAndUFs(const VPValue *V);

/// Returns the header block of the first, top-level loop, or null if none
/// exist.
VPBasicBlock *getFirstLoopHeader(VPlan &Plan, VPDominatorTree &VPDT);

/// Get the VF scaling factor applied to the recipe's output, if the recipe has
/// one.
unsigned getVFScaleFactor(VPRecipeBase *R);

/// Return true if we do not know how to (mechanically) hoist or sink \p R.
/// When sinking, passing \p Sinking = true ensures that assumes aren't sunk.
/// Returns true for recipes that access memory.
bool cannotHoistOrSinkRecipe(const VPRecipeBase &R, bool Sinking = false);

```
- EN: The declarations explicitly reference analyses such as DominatorTree, ScalarEvolution, SCEV, showing what later implementations must consult to reason about safety or cost. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 这些声明显式涉及 DominatorTree, ScalarEvolution, SCEV 等分析，表明后续实现需要依赖它们来判断安全性或代价。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 80-112

```cpp
/// Returns the VPValue representing the uncountable exit comparison used by
/// AnyOf if the recipes it depends on can be traced back to live-ins and
/// the addresses (in GEP/PtrAdd form) of any (non-masked) load used in
/// generating the values for the comparison. The recipes are stored in
/// \p Recipes, and recipes forming an address for a load are also added to
/// \p GEPs.
LLVM_ABI_FOR_TEST
std::optional<VPValue *>
getRecipesForUncountableExit(SmallVectorImpl<VPInstruction *> &Recipes,
                             SmallVectorImpl<VPInstruction *> &GEPs,
                             VPBasicBlock *LatchVPBB);

/// Return a MemoryLocation for \p R with noalias metadata populated from
/// \p R, if the recipe is supported and std::nullopt otherwise. The pointer of
/// the location is conservatively set to nullptr.
std::optional<MemoryLocation> getMemoryLocation(const VPRecipeBase &R);

/// Extracts and returns NoWrap and FastMath flags from the induction binop in
/// \p ID.
inline VPIRFlags getFlagsFromIndDesc(const InductionDescriptor &ID) {
  if (ID.getKind() == InductionDescriptor::IK_FpInduction)
    return ID.getInductionBinOp()->getFastMathFlags();

  if (auto *OBO = dyn_cast_if_present<OverflowingBinaryOperator>(
          ID.getInductionBinOp()))
    return VPIRFlags::WrapFlagsTy(OBO->hasNoUnsignedWrap(),
                                  OBO->hasNoSignedWrap());

  assert(ID.getKind() == InductionDescriptor::IK_IntInduction &&
         "Expected int induction");
  return VPIRFlags::WrapFlagsTy(false, false);
}

```
- EN: Core entities appearing here include getFlagsFromIndDesc, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 getFlagsFromIndDesc，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 113-149

```cpp
/// Search \p Start's users for a recipe satisfying \p Pred, looking through
/// recipes with definitions.
template <typename PredT>
inline VPRecipeBase *findRecipe(VPValue *Start, PredT Pred) {
  SetVector<VPValue *> Worklist;
  Worklist.insert(Start);
  for (unsigned I = 0; I != Worklist.size(); ++I) {
    VPValue *Cur = Worklist[I];
    auto *R = Cur->getDefiningRecipe();
    if (!R)
      continue;
    if (Pred(R))
      return R;
    for (VPUser *U : Cur->users()) {
      for (VPValue *V : cast<VPRecipeBase>(U)->definedValues())
        Worklist.insert(V);
    }
  }
  return nullptr;
}

/// If \p V is used by a recipe matching pattern \p P, return it. Otherwise
/// return nullptr;
template <typename MatchT>
static VPRecipeBase *findUserOf(VPValue *V, const MatchT &P) {
  using namespace llvm::VPlanPatternMatch;
  auto It = find_if(V->users(), match_fn(P));
  return It == V->user_end() ? nullptr : cast<VPRecipeBase>(*It);
}

/// If \p V is used by a VPInstruction with \p Opcode, return it. Otherwise
/// return nullptr.
template <unsigned Opcode> static VPInstruction *findUserOf(VPValue *V) {
  using namespace llvm::VPlanPatternMatch;
  return cast_or_null<VPInstruction>(findUserOf(V, m_VPInstruction<Opcode>()));
}

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include f, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 f，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 150-187

```cpp
template <typename RecipeTy> static RecipeTy *findUserOf(VPValue *V) {
  using namespace llvm::VPlanPatternMatch;
  return cast_or_null<RecipeTy>(findUserOf(V, m_Isa<RecipeTy>()));
}

/// Find the canonical IV increment of \p Plan's vector loop region. Returns
/// nullptr if not found.
VPInstruction *findCanonicalIVIncrement(VPlan &Plan);

/// Returns the GEP nowrap flags for \p Ptr, looking through pointer casts
/// mirroring Value::stripPointerCasts.
GEPNoWrapFlags getGEPFlagsForPtr(VPValue *Ptr);

/// Returns true if \p V is used as part of the address of another load or
/// store.
bool isUsedByLoadStoreAddress(const VPValue *V);

/// Find the ComputeReductionResult recipe for \p PhiR, looking through selects
/// inserted for predicated reductions or tail folding.
VPInstruction *findComputeReductionResult(VPReductionPHIRecipe *PhiR);

/// Collect the header mask with the pattern:
/// (ICMP_ULE, WideCanonicalIV, backedge-taken-count)
/// TODO: Introduce explicit recipe for header-mask instead of searching
/// the header-mask pattern manually.
VPSingleDefRecipe *findHeaderMask(VPlan &Plan);

} // namespace vputils

//===----------------------------------------------------------------------===//
// Utilities for modifying predecessors and successors of VPlan blocks.
//===----------------------------------------------------------------------===//

/// Class that provides utilities for VPBlockBases in VPlan.
class VPBlockUtils {
public:
  VPBlockUtils() = delete;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include f, VPBlockUtils, which carry the local state, declarations, or transformation entry points for this region.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 f, VPBlockUtils，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 188-223

```cpp
  /// Insert disconnected VPBlockBase \p NewBlock after \p BlockPtr. Add \p
  /// NewBlock as successor of \p BlockPtr and \p BlockPtr as predecessor of \p
  /// NewBlock, and propagate \p BlockPtr parent to \p NewBlock. \p BlockPtr's
  /// successors are moved from \p BlockPtr to \p NewBlock. \p NewBlock must
  /// have neither successors nor predecessors.
  static void insertBlockAfter(VPBlockBase *NewBlock, VPBlockBase *BlockPtr) {
    assert(!NewBlock->hasSuccessors() && !NewBlock->hasPredecessors() &&
           "Can't insert new block with predecessors or successors.");
    NewBlock->setParent(BlockPtr->getParent());
    transferSuccessors(BlockPtr, NewBlock);
    connectBlocks(BlockPtr, NewBlock);
  }

  /// Insert disconnected block \p NewBlock before \p Blockptr. First
  /// disconnects all predecessors of \p BlockPtr and connects them to \p
  /// NewBlock. Add \p NewBlock as predecessor of \p BlockPtr and \p BlockPtr as
  /// successor of \p NewBlock.
  static void insertBlockBefore(VPBlockBase *NewBlock, VPBlockBase *BlockPtr) {
    assert(!NewBlock->hasSuccessors() && !NewBlock->hasPredecessors() &&
           "Can't insert new block with predecessors or successors.");
    NewBlock->setParent(BlockPtr->getParent());
    for (VPBlockBase *Pred : to_vector(BlockPtr->predecessors())) {
      Pred->replaceSuccessor(BlockPtr, NewBlock);
      NewBlock->appendPredecessor(Pred);
    }
    BlockPtr->clearPredecessors();
    connectBlocks(NewBlock, BlockPtr);
  }

  /// Insert disconnected VPBlockBases \p IfTrue and \p IfFalse after \p
  /// BlockPtr. Add \p IfTrue and \p IfFalse as succesors of \p BlockPtr and \p
  /// BlockPtr as predecessor of \p IfTrue and \p IfFalse. Propagate \p BlockPtr
  /// parent to \p IfTrue and \p IfFalse. \p BlockPtr must have no successors
  /// and \p IfTrue and \p IfFalse must have neither successors nor
  /// predecessors.
  static void insertTwoBlocksAfter(VPBlockBase *IfTrue, VPBlockBase *IfFalse,
```
- EN: Core entities appearing here include insertBlockAfter, insertBlockBefore, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 insertBlockAfter, insertBlockBefore，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 224-264

```cpp
                                   VPBlockBase *BlockPtr) {
    assert(!IfTrue->hasSuccessors() && "Can't insert IfTrue with successors.");
    assert(!IfFalse->hasSuccessors() &&
           "Can't insert IfFalse with successors.");
    BlockPtr->setTwoSuccessors(IfTrue, IfFalse);
    IfTrue->setPredecessors({BlockPtr});
    IfFalse->setPredecessors({BlockPtr});
    IfTrue->setParent(BlockPtr->getParent());
    IfFalse->setParent(BlockPtr->getParent());
  }

  /// Connect VPBlockBases \p From and \p To bi-directionally. If \p PredIdx is
  /// -1, append \p From to the predecessors of \p To, otherwise set \p To's
  /// predecessor at \p PredIdx to \p From. If \p SuccIdx is -1, append \p To to
  /// the successors of \p From, otherwise set \p From's successor at \p SuccIdx
  /// to \p To. Both VPBlockBases must have the same parent, which can be null.
  /// Both VPBlockBases can be already connected to other VPBlockBases.
  static void connectBlocks(VPBlockBase *From, VPBlockBase *To,
                            unsigned PredIdx = -1u, unsigned SuccIdx = -1u) {
    assert((From->getParent() == To->getParent()) &&
           "Can't connect two block with different parents");

    if (SuccIdx == -1u)
      From->appendSuccessor(To);
    else
      From->getSuccessors()[SuccIdx] = To;

    if (PredIdx == -1u)
      To->appendPredecessor(From);
    else
      To->getPredecessors()[PredIdx] = From;
  }

  /// Disconnect VPBlockBases \p From and \p To bi-directionally. Remove \p To
  /// from the successors of \p From and \p From from the predecessors of \p To.
  static void disconnectBlocks(VPBlockBase *From, VPBlockBase *To) {
    assert(To && "Successor to disconnect is null.");
    From->removeSuccessor(To);
    To->removePredecessor(From);
  }

```
- EN: Core entities appearing here include disconnectBlocks, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 disconnectBlocks，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 265-299

```cpp
  /// Reassociate all the blocks connected to \p Old so that they now point to
  /// \p New.
  static void reassociateBlocks(VPBlockBase *Old, VPBlockBase *New) {
    for (auto *Pred : to_vector(Old->getPredecessors()))
      Pred->replaceSuccessor(Old, New);
    for (auto *Succ : to_vector(Old->getSuccessors()))
      Succ->replacePredecessor(Old, New);
    New->setPredecessors(Old->getPredecessors());
    New->setSuccessors(Old->getSuccessors());
    Old->clearPredecessors();
    Old->clearSuccessors();
  }

  /// Transfer successors from \p Old to \p New. \p New must have no successors.
  static void transferSuccessors(VPBlockBase *Old, VPBlockBase *New) {
    for (auto *Succ : Old->getSuccessors())
      Succ->replacePredecessor(Old, New);
    New->setSuccessors(Old->getSuccessors());
    Old->clearSuccessors();
  }

  /// Clone the CFG for all nodes reachable from \p Entry, including cloning
  /// the blocks and their recipes. Operands of cloned recipes will NOT be
  /// updated. Remapping of operands must be done separately. Returns a pair
  /// with the new entry and exiting blocks of the cloned region. If \p Entry
  /// isn't part of a region, return nullptr for the exiting block.
  static std::pair<VPBlockBase *, VPBlockBase *> cloneFrom(VPBlockBase *Entry);

  /// Return an iterator range over \p Range which only includes \p BlockTy
  /// blocks. The accesses are casted to \p BlockTy.
  template <typename BlockTy, typename T> static auto blocksOnly(T &&Range) {
    // Create BaseTy with correct const-ness based on BlockTy.
    using BaseTy = std::conditional_t<std::is_const<BlockTy>::value,
                                      const VPBlockBase, VPBlockBase>;

```
- EN: Core entities appearing here include reassociateBlocks, transferSuccessors, y, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 reassociateBlocks, transferSuccessors, y，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 300-340

```cpp
    // We need to first create an iterator range over (const) BlocktTy & instead
    // of (const) BlockTy * for filter_range to work properly.
    auto Mapped =
        map_range(Range, [](BaseTy *Block) -> BaseTy & { return *Block; });
    auto Filter = make_filter_range(
        Mapped, [](BaseTy &Block) { return isa<BlockTy>(&Block); });
    return map_range(Filter, [](BaseTy &Block) -> BlockTy * {
      return cast<BlockTy>(&Block);
    });
  }

  /// Return an iterator range over \p Range with each block cast to \p
  /// BlockTy. Unlike blocksOnly, all blocks in \p Range must be of type
  /// \p BlockTy.
  template <typename BlockTy, typename T> static auto blocksAs(T &&Range) {
    // Create BaseTy with correct const-ness based on BlockTy.
    using BaseTy = std::conditional_t<std::is_const<BlockTy>::value,
                                      const VPBlockBase, VPBlockBase>;
    return map_range(
        Range, [](BaseTy *Block) -> BlockTy * { return cast<BlockTy>(Block); });
  }

  /// Returns the blocks between \p FirstBB and \p LastBB, where FirstBB
  /// to LastBB forms a single-sucessor chain.
  static SmallVector<VPBasicBlock *>
  blocksInSingleSuccessorChainBetween(VPBasicBlock *FirstBB,
                                      VPBasicBlock *LastBB);

  /// Inserts \p BlockPtr on the edge between \p From and \p To. That is, update
  /// \p From's successor to \p To to point to \p BlockPtr and \p To's
  /// predecessor from \p From to \p BlockPtr. \p From and \p To are added to \p
  /// BlockPtr's predecessors and successors respectively. There must be a
  /// single edge between \p From and \p To.
  static void insertOnEdge(VPBlockBase *From, VPBlockBase *To,
                           VPBlockBase *BlockPtr) {
    unsigned SuccIdx = From->getIndexForSuccessor(To);
    unsigned PredIx = To->getIndexForPredecessor(From);
    VPBlockUtils::connectBlocks(From, BlockPtr, -1, SuccIdx);
    VPBlockUtils::connectBlocks(BlockPtr, To, PredIx, -1);
  }

```
- EN: Core entities appearing here include s, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 s，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 341-351

```cpp
  /// Returns true if \p VPB is a loop header, based on regions or \p VPDT in
  /// their absence.
  static bool isHeader(const VPBlockBase *VPB, const VPDominatorTree &VPDT);

  /// Returns true if \p VPB is a loop latch, using isHeader().
  static bool isLatch(const VPBlockBase *VPB, const VPDominatorTree &VPDT);
};

} // namespace llvm

#endif
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. The declarations explicitly reference analyses such as DominatorTree, showing what later implementations must consult to reason about safety or cost.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这些声明显式涉及 DominatorTree 等分析，表明后续实现需要依赖它们来判断安全性或代价。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `MemoryLocation, ScalarEvolution, SCEV, PredicatedScalarEvolution, VPBlockUtils, getFlagsFromIndDesc, f, insertBlockAfter` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`MemoryLocation, ScalarEvolution, SCEV, PredicatedScalarEvolution, VPBlockUtils, getFlagsFromIndDesc, f, insertBlockAfter` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DominatorTree, ScalarEvolution, SCEV` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DominatorTree, ScalarEvolution, SCEV` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Header role: it mostly exposes declarations, contracts, inline helpers, and reusable utilities rather than driving a standalone transform by itself.
  - CN: 头文件角色：它主要暴露声明、约定、内联辅助函数和可复用工具，而不是独立执行某个完整变换。

## Dependencies / 依赖关系

- EN: Utility headers: `llvm/Support/Compiler.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Compiler.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `VPlan.h`, `VPlanPatternMatch.h` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`VPlan.h`, `VPlanPatternMatch.h` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DominatorTree`, `ScalarEvolution`, `SCEV` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DominatorTree`, `ScalarEvolution`, `SCEV` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
