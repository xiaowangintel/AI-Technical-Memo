# VPlan.h — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Vectorize/VPlan.h` | `llvm/lib/Transforms/Vectorize/VPlan.h` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This header defines the public interface for represent A Vectorizer Plan within LLVM's vectorization and VPlan infrastructure layer. | 该头文件定义了对外接口 LLVM 向量化与 VPlan 基础设施层中的 VPlan 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-310

```cpp
//===- VPlan.h - Represent A Vectorizer Plan --------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This file contains the declarations of the Vectorization Plan base classes:
/// 1. VPBasicBlock and VPRegionBlock that inherit from a common pure virtual
///    VPBlockBase, together implementing a Hierarchical CFG;
/// 2. Pure virtual VPRecipeBase serving as the base class for recipes contained
///    within VPBasicBlocks;
/// 3. Pure virtual VPSingleDefRecipe serving as a base class for recipes that
///    also inherit from VPValue.
/// 4. VPInstruction, a concrete Recipe and VPUser modeling a single planned
///    instruction;
/// 5. The VPlan class holding a candidate for vectorization;
/// These are documented in docs/VectorizationPlan.rst.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    assert(Successors.empty() && "Setting two successors when others exist.");
    appendSuccessor(IfTrue);
    appendSuccessor(IfFalse);
  }

  /// Set each VPBasicBlock in \p NewPreds as predecessor of this VPBlockBase.
  /// This VPBlockBase must have no predecessors. This VPBlockBase is not added
  /// as successor of any VPBasicBlock in \p NewPreds.
  void setPredecessors(ArrayRef<VPBlockBase *> NewPreds) {
    assert(Predecessors.empty() && "Block predecessors already set.");
    for (auto *Pred : NewPreds)
      appendPredecessor(Pred);
  }

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. The header guard ensures the declarations are safe to include from multiple compilation units. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 头文件保护宏保证这些声明可以被多个编译单元安全地重复包含。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。

### Lines 311-656

```cpp
  /// Set each VPBasicBlock in \p NewSuccss as successor of this VPBlockBase.
  /// This VPBlockBase must have no successors. This VPBlockBase is not added
  /// as predecessor of any VPBasicBlock in \p NewSuccs.
  void setSuccessors(ArrayRef<VPBlockBase *> NewSuccs) {
    assert(Successors.empty() && "Block successors already set.");
    for (auto *Succ : NewSuccs)
      appendSuccessor(Succ);
  }

  /// Remove all the predecessor of this block.
  void clearPredecessors() { Predecessors.clear(); }

  /// Remove all the successors of this block.
  void clearSuccessors() { Successors.clear(); }

  /// Swap predecessors of the block. The block must have exactly 2
  /// predecessors.
  void swapPredecessors() {
    assert(Predecessors.size() == 2 && "must have 2 predecessors to swap");
    std::swap(Predecessors[0], Predecessors[1]);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    case VPRecipeBase::VPWidenLoadSC:
      return true;
    case VPRecipeBase::VPBranchOnMaskSC:
    case VPRecipeBase::VPInterleaveEVLSC:
    case VPRecipeBase::VPInterleaveSC:
    case VPRecipeBase::VPIRInstructionSC:
    case VPRecipeBase::VPWidenStoreEVLSC:
    case VPRecipeBase::VPWidenStoreSC:
    case VPRecipeBase::VPHistogramSC:
      return false;
    }
    llvm_unreachable("Unhandled VPRecipeID");
  }

```
- EN: The header guard ensures the declarations are safe to include from multiple compilation units. Core entities appearing here include setSuccessors, swapPredecessors, swapSuccessors, which carry the local state, declarations, or transformation entry points for this region. The declarations explicitly reference analyses such as SCEV, showing what later implementations must consult to reason about safety or cost.
- CN: 头文件保护宏保证这些声明可以被多个编译单元安全地重复包含。 此处出现的核心实体包括 setSuccessors, swapPredecessors, swapSuccessors，它们承载了该区间涉及的局部状态、声明或变换入口。 这些声明显式涉及 SCEV 等分析，表明后续实现需要依赖它们来判断安全性或代价。

### Lines 657-969

```cpp
  static inline bool classof(const VPValue *V) {
    auto *R = V->getDefiningRecipe();
    return R && classof(R);
  }

  static inline bool classof(const VPUser *U) {
    auto *R = dyn_cast<VPRecipeBase>(U);
    return R && classof(R);
  }

  VPSingleDefRecipe *clone() override = 0;

  /// Returns the underlying instruction.
  Instruction *getUnderlyingInstr() {
    return cast<Instruction>(getUnderlyingValue());
  }
  const Instruction *getUnderlyingInstr() const {
    return cast<Instruction>(getUnderlyingValue());
  }

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    case OperationType::Cmp:
    case OperationType::Other:
      break;
    }
  }

  CmpInst::Predicate getPredicate() const {
    assert((OpType == OperationType::Cmp || OpType == OperationType::FCmp) &&
           "recipe doesn't have a compare predicate");
    uint8_t Storage = OpType == OperationType::FCmp ? FCmpFlags.CmpPredStorage
                                                    : CmpPredStorage;
    return Bitfield::get<CmpInst::PredicateField>(Storage);
  }

```
- EN: Core entities appearing here include classof, VPIRFlags, OperationType, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 classof, VPIRFlags, OperationType，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 970-1302

```cpp
  void setPredicate(CmpInst::Predicate Pred) {
    assert((OpType == OperationType::Cmp || OpType == OperationType::FCmp) &&
           "recipe doesn't have a compare predicate");
    if (OpType == OperationType::FCmp)
      Bitfield::set<CmpInst::PredicateField>(FCmpFlags.CmpPredStorage, Pred);
    else
      Bitfield::set<CmpInst::PredicateField>(CmpPredStorage, Pred);
    assert(getPredicate() == Pred && "predicate truncated");
  }

  GEPNoWrapFlags getGEPNoWrapFlags() const {
    return GEPNoWrapFlags::fromRaw(GEPFlagsStorage);
  }

  /// Returns true if the recipe has a comparison predicate.
  bool hasPredicate() const {
    return OpType == OperationType::Cmp || OpType == OperationType::FCmp;
  }

  /// Returns true if the recipe has fast-math flags.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // add all copies of its original operand as additional operands.
    // Implemented with @llvm.experimental.cttz.elts, but returns the expected
    // result even with operands that are all zeroes.
    FirstActiveLane,
    // Calculates the last active lane index of the vector predicate operands.
    // The predicates must be prefix-masks (all 1s before all 0s). Used when
    // tail-folding to extract the correct live-out value from the last active
    // iteration. It produces the lane index across all unrolled iterations.
    // Unrolling will add all copies of its original operand as additional
    // operands.
    LastActiveLane,
    // Returns a reversed vector for the operand.
    Reverse,

```
- EN: Core entities appearing here include setPredicate, getGEPNoWrapFlags, hasPredicate, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 setPredicate, getGEPNoWrapFlags, hasPredicate，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 1303-1618

```cpp
    // The opcodes below are used for VPInstructionWithType.
    //
    /// Scale the first operand (vector step) by the second operand
    /// (scalar-step).  Casts both operands to the result type if needed.
    WideIVStep,
    /// Start vector for reductions with 3 operands: the original start value,
    /// the identity value for the reduction and an integer indicating the
    /// scaling factor.
    ReductionStartVector,
    // Creates a step vector starting from 0 to VF with a step of 1.
    StepVector,
    /// Extracts a single lane (first operand) from a set of vector operands.
    /// The lane specifies an index into a vector formed by combining all vector
    /// operands (all operands after the first one).
    ExtractLane,
    /// Explicit user for the resume phi of the canonical induction in the main
    /// VPlan, used by the epilogue vector loop.
    ResumeForEpilogue,
    /// Extracts the last active lane from a set of vectors. The first operand
    /// is the default value if no lanes in the masks are active. Conceptually,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                      getAsRecipe()->op_begin() + getNumIncoming());
  }

  using const_incoming_blocks_range = iterator_range<mapped_iterator<
      detail::index_iterator, std::function<const VPBasicBlock *(size_t)>>>;

  /// Returns an iterator range over the incoming blocks.
  const_incoming_blocks_range incoming_blocks() const {
    std::function<const VPBasicBlock *(size_t)> GetBlock = [this](size_t Idx) {
      return getIncomingBlock(Idx);
    };
    return map_range(index_range(0, getNumIncoming()), GetBlock);
  }

```
- EN: Core entities appearing here include alwaysUnmasked, hasResult, isMasked, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 alwaysUnmasked, hasResult, isMasked，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 1619-1931

```cpp
  /// Returns an iterator range over pairs of incoming values and corresponding
  /// incoming blocks.
  detail::zippy<llvm::detail::zip_first, VPUser::const_operand_range,
                const_incoming_blocks_range>
  incoming_values_and_blocks() const {
    return zip_equal(incoming_values(), incoming_blocks());
  }

  /// Removes the incoming value for \p IncomingBlock, which must be a
  /// predecessor.
  void removeIncomingValueFor(VPBlockBase *IncomingBlock) const;

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
  /// Print the recipe.
  void printPhiOperands(raw_ostream &O, VPSlotTracker &SlotTracker) const;
#endif
};

struct LLVM_ABI_FOR_TEST VPPhi : public VPInstruction, public VPPhiAccessors {
  VPPhi(ArrayRef<VPValue *> Operands, const VPIRFlags &Flags, DebugLoc DL,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      : VPRecipeWithIRFlags(VPRecipeBase::VPWidenIntrinsicSC, CallArguments,
                            Flags, DL),
        VPIRMetadata(Metadata), VectorIntrinsicID(VectorIntrinsicID),
        ResultTy(Ty) {
    LLVMContext &Ctx = Ty->getContext();
    AttributeSet Attrs = Intrinsic::getFnAttributes(Ctx, VectorIntrinsicID);
    MemoryEffects ME = Attrs.getMemoryEffects();
    MayReadFromMemory = !ME.onlyWritesMemory();
    MayWriteToMemory = !ME.onlyReadsMemory();
    MayHaveSideEffects = MayWriteToMemory ||
                         !Attrs.hasAttribute(Attribute::NoUnwind) ||
                         !Attrs.hasAttribute(Attribute::WillReturn);
  }

```
- EN: Core entities appearing here include incoming_values_and_blocks, LLVM_ABI_FOR_TEST, classof, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 incoming_values_and_blocks, LLVM_ABI_FOR_TEST, classof，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 1932-2243

```cpp
  ~VPWidenIntrinsicRecipe() override = default;

  VPWidenIntrinsicRecipe *clone() override {
    if (Value *CI = getUnderlyingValue())
      return new VPWidenIntrinsicRecipe(*cast<CallInst>(CI), VectorIntrinsicID,
                                        operands(), ResultTy, *this, *this,
                                        getDebugLoc());
    return new VPWidenIntrinsicRecipe(VectorIntrinsicID, operands(), ResultTy,
                                      *this, *this, getDebugLoc());
  }

  VP_CLASSOF_IMPL(VPRecipeBase::VPWidenIntrinsicSC)

  /// Produce a widened version of the vector intrinsic.
  LLVM_ABI_FOR_TEST void execute(VPTransformState &State) override;

  /// Compute the cost of a vector intrinsic with \p ID and \p Operands.
  static InstructionCost computeCallCost(Intrinsic::ID ID,
                                         ArrayRef<const VPValue *> Operands,
                                         const VPRecipeWithIRFlags &R,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  Type *SourceElementTy;

public:
  VPVectorPointerRecipe(VPValue *Ptr, Type *SourceElementTy,
                        GEPNoWrapFlags GEPFlags, DebugLoc DL)
      : VPRecipeWithIRFlags(VPRecipeBase::VPVectorPointerSC, Ptr, GEPFlags, DL),
        SourceElementTy(SourceElementTy) {}

  VP_CLASSOF_IMPL(VPRecipeBase::VPVectorPointerSC)

  VPValue *getOffset() {
    return getNumOperands() == 2 ? getOperand(1) : nullptr;
  }

```
- EN: Core entities appearing here include LLVM_ABI_FOR_TEST, VPIRMetadata, VPHistogramRecipe, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 LLVM_ABI_FOR_TEST, VPIRMetadata, VPHistogramRecipe，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 2244-2555

```cpp
  void execute(VPTransformState &State) override;

  Type *getSourceElementType() const { return SourceElementTy; }

  bool usesFirstLaneOnly(const VPValue *Op) const override {
    assert(is_contained(operands(), Op) &&
           "Op must be an operand of the recipe");
    return true;
  }

  /// Returns true if the recipe only uses the first part of operand \p Op.
  bool usesFirstPartOnly(const VPValue *Op) const override {
    assert(is_contained(operands(), Op) &&
           "Op must be an operand of the recipe");
    assert(getNumOperands() <= 2 && "must have at most two operands");
    return true;
  }

  VPVectorPointerRecipe *clone() override {
    auto *Clone = new VPVectorPointerRecipe(getOperand(0), SourceElementTy,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

class VPWidenPointerInductionRecipe : public VPWidenInductionRecipe {
public:
  /// Create a new VPWidenPointerInductionRecipe for \p Phi with start value \p
  /// Start and the number of elements unrolled \p NumUnrolledElems, typically
  /// VF*UF.
  VPWidenPointerInductionRecipe(PHINode *Phi, VPValue *Start, VPValue *Step,
                                VPValue *NumUnrolledElems,
                                const InductionDescriptor &IndDesc, DebugLoc DL)
      : VPWidenInductionRecipe(VPRecipeBase::VPWidenPointerInductionSC, Phi,
                               Start, Step, IndDesc, DL) {
    addOperand(NumUnrolledElems);
  }

```
- EN: Core entities appearing here include LLVM_ABI_FOR_TEST, classof, VPWidenInductionRecipe, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 LLVM_ABI_FOR_TEST, classof, VPWidenInductionRecipe，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 2556-2867

```cpp
  ~VPWidenPointerInductionRecipe() override = default;

  VPWidenPointerInductionRecipe *clone() override {
    return new VPWidenPointerInductionRecipe(
        cast<PHINode>(getUnderlyingInstr()), getOperand(0), getOperand(1),
        getOperand(2), getInductionDescriptor(), getDebugLoc());
  }

  VP_CLASSOF_IMPL(VPRecipeBase::VPWidenPointerInductionSC)

  /// Generate vector values for the pointer induction.
  void execute(VPTransformState &State) override {
    llvm_unreachable("cannot execute this recipe, should be expanded via "
                     "expandVPWidenPointerInduction");
  };

  /// Returns true if only scalar values will be generated.
  bool onlyScalarsGenerated(bool IsScalable);

protected:
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// The first operand is the start address. The optional operands are, in order,
/// the stored values and the mask.
class LLVM_ABI_FOR_TEST VPInterleaveBase : public VPRecipeBase,
                                           public VPIRMetadata {
  const InterleaveGroup<Instruction> *IG;

  /// Indicates if the interleave group is in a conditional block and requires a
  /// mask.
  bool HasMask = false;

  /// Indicates if gaps between members of the group need to be masked out or if
  /// unusued gaps can be loaded speculatively.
  bool NeedsMaskForGaps = false;

```
- EN: Core entities appearing here include LLVM_ABI_FOR_TEST, VPFirstOrderRecurrencePHIRecipe, RdxOrdered, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 LLVM_ABI_FOR_TEST, VPFirstOrderRecurrencePHIRecipe, RdxOrdered，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 2868-3179

```cpp
protected:
  VPInterleaveBase(const unsigned char SC,
                   const InterleaveGroup<Instruction> *IG,
                   ArrayRef<VPValue *> Operands,
                   ArrayRef<VPValue *> StoredValues, VPValue *Mask,
                   bool NeedsMaskForGaps, const VPIRMetadata &MD, DebugLoc DL)
      : VPRecipeBase(SC, Operands, DL), VPIRMetadata(MD), IG(IG),
        NeedsMaskForGaps(NeedsMaskForGaps) {
    // TODO: extend the masked interleaved-group support to reversed access.
    assert((!Mask || !IG->isReverse()) &&
           "Reversed masked interleave-group not supported.");
    if (StoredValues.empty()) {
      for (Instruction *Inst : IG->members()) {
        assert(!Inst->getType()->isVoidTy() && "must have result");
        new VPMultiDefValue(this, Inst);
      }
    } else {
      for (auto *SV : StoredValues)
        addOperand(SV);
    }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  ~VPReductionEVLRecipe() override = default;

  VPReductionEVLRecipe *clone() override {
    llvm_unreachable("cloning not implemented yet");
  }

  VP_CLASSOF_IMPL(VPRecipeBase::VPReductionEVLSC)

  /// Generate the reduction in the loop
  void execute(VPTransformState &State) override;

  /// The VPValue of the explicit vector length.
  VPValue *getEVL() const { return getOperand(2); }

```
- EN: Core entities appearing here include NeedsMaskForGaps, classof, getStoredValues, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 NeedsMaskForGaps, classof, getStoredValues，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 3180-3494

```cpp
  /// Returns true if the recipe only uses the first lane of operand \p Op.
  bool usesFirstLaneOnly(const VPValue *Op) const override {
    assert(is_contained(operands(), Op) &&
           "Op must be an operand of the recipe");
    return Op == getEVL();
  }

protected:
#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
  /// Print the recipe.
  void printRecipe(raw_ostream &O, const Twine &Indent,
                   VPSlotTracker &SlotTracker) const override;
#endif
};

/// VPReplicateRecipe replicates a given instruction producing multiple scalar
/// copies of the original scalar type, one per lane, instead of producing a
/// single copy of widened type for all lanes. If the instruction is known to be
/// a single scalar, only one copy will be generated.
class LLVM_ABI_FOR_TEST VPReplicateRecipe : public VPRecipeWithIRFlags,
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  VP_CLASSOF_IMPL(VPRecipeBase::VPPredInstPHISC)

  /// Generates phi nodes for live-outs (from a replicate region) as needed to
  /// retain SSA form.
  void execute(VPTransformState &State) override;

  /// Return the cost of this VPPredInstPHIRecipe.
  InstructionCost computeCost(ElementCount VF,
                              VPCostContext &Ctx) const override {
    // TODO: Compute accurate cost after retiring the legacy cost model.
    return 0;
  }

```
- EN: Core entities appearing here include LLVM_ABI_FOR_TEST, IsPredicated, VPExpressionRecipe, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 LLVM_ABI_FOR_TEST, IsPredicated, VPExpressionRecipe，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 3495-3811

```cpp
protected:
#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
  /// Print the recipe.
  void printRecipe(raw_ostream &O, const Twine &Indent,
                   VPSlotTracker &SlotTracker) const override;
#endif
};

/// A common mixin class for widening memory operations. An optional mask can be
/// provided as the last operand.
class LLVM_ABI_FOR_TEST VPWidenMemoryRecipe : public VPIRMetadata {
protected:
  Instruction &Ingredient;

  /// Alignment information for this memory access.
  Align Alignment;

  /// Whether the accessed addresses are consecutive.
  bool Consecutive;

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  /// Print the recipe.
  void printRecipe(raw_ostream &O, const Twine &Indent,
                   VPSlotTracker &SlotTracker) const override;
#endif
};

/// A recipe for generating the active lane mask for the vector loop that is
/// used to predicate the vector operations.
class VPActiveLaneMaskPHIRecipe : public VPHeaderPHIRecipe {
public:
  VPActiveLaneMaskPHIRecipe(VPValue *StartMask, DebugLoc DL)
      : VPHeaderPHIRecipe(VPRecipeBase::VPActiveLaneMaskPHISC, nullptr,
                          StartMask, DL) {}

```
- EN: Core entities appearing here include LLVM_ABI_FOR_TEST, setMask, VPWidenMemoryRecipe, which carry the local state, declarations, or transformation entry points for this region. The declarations explicitly reference analyses such as SCEV, showing what later implementations must consult to reason about safety or cost. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 LLVM_ABI_FOR_TEST, setMask, VPWidenMemoryRecipe，它们承载了该区间涉及的局部状态、声明或变换入口。 这些声明显式涉及 SCEV 等分析，表明后续实现需要依赖它们来判断安全性或代价。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 3812-4126

```cpp
  ~VPActiveLaneMaskPHIRecipe() override = default;

  VPActiveLaneMaskPHIRecipe *clone() override {
    auto *R = new VPActiveLaneMaskPHIRecipe(getOperand(0), getDebugLoc());
    if (getNumOperands() == 2)
      R->addOperand(getOperand(1));
    return R;
  }

  VP_CLASSOF_IMPL(VPRecipeBase::VPActiveLaneMaskPHISC)

  /// Generate the active lane mask phi of the vector loop.
  void execute(VPTransformState &State) override;

protected:
#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
  /// Print the recipe.
  void printRecipe(raw_ostream &O, const Twine &Indent,
                   VPSlotTracker &SlotTracker) const override;
#endif
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                                  CastInfo<VPPhiAccessors, VPRecipeBase *>> {};

/// Support casting from VPRecipeBase / VPUser -> VPWidenMemoryRecipe.
template <>
struct CastInfo<VPWidenMemoryRecipe, VPRecipeBase *>
    : vpdetail::CastInfoMixinImpl<VPWidenMemoryRecipe, VPWidenLoadRecipe,
                                  VPWidenLoadEVLRecipe, VPWidenStoreRecipe,
                                  VPWidenStoreEVLRecipe> {};
template <>
struct CastInfo<VPWidenMemoryRecipe, const VPRecipeBase *>
    : public ConstStrippingForwardingCast<
          VPWidenMemoryRecipe, const VPRecipeBase *,
          CastInfo<VPWidenMemoryRecipe, VPRecipeBase *>> {};

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include VPCurrentIterationPHIRecipe, VPWidenCanonicalIVRecipe, VPDerivedIVRecipe, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 VPCurrentIterationPHIRecipe, VPWidenCanonicalIVRecipe, VPDerivedIVRecipe，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 4127-4439

```cpp
/// Support casting from VPRecipeBase -> VPIRMetadata.
template <>
struct CastInfo<VPIRMetadata, VPRecipeBase *>
    : vpdetail::CastInfoMixinImpl<VPIRMetadata, VPInstruction, VPWidenRecipe,
                                  VPWidenCastRecipe, VPWidenIntrinsicRecipe,
                                  VPWidenCallRecipe, VPReplicateRecipe,
                                  VPInterleaveBase, VPWidenMemoryRecipe> {};

template <>
struct CastInfo<VPIRMetadata, const VPRecipeBase *>
    : public ConstStrippingForwardingCast<
          VPIRMetadata, const VPRecipeBase *,
          CastInfo<VPIRMetadata, VPRecipeBase *>> {};
template <>
struct CastInfo<VPIRMetadata, VPRecipeBase>
    : public ForwardToPointerCast<VPIRMetadata, VPRecipeBase *,
                                  CastInfo<VPIRMetadata, VPRecipeBase *>> {};

/// VPBasicBlock serves as the leaf of the Hierarchical Control-Flow Graph. It
/// holds a sequence of zero or more VPRecipe's each representing a sequence of
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  /// Returns the pre-header VPBasicBlock of the loop region.
  VPBasicBlock *getPreheaderVPBB() {
    assert(!isReplicator() && "should only get pre-header of loop regions");
    return getSinglePredecessor()->getExitingBasicBlock();
  }

  /// An indicator whether this region is to generate multiple replicated
  /// instances of output IR corresponding to its VPBlockBases.
  bool isReplicator() const { return !CanIVInfo; }

  /// The method which generates the output IR instructions that correspond to
  /// this VPRegionBlock, thereby "executing" the VPlan.
  void execute(VPTransformState *State) override;

```
- EN: Core entities appearing here include CastInfo, LLVM_ABI_FOR_TEST, VPBlockBase, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 CastInfo, LLVM_ABI_FOR_TEST, VPBlockBase，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 4440-4753

```cpp
  // Return the cost of this region.
  InstructionCost cost(ElementCount VF, VPCostContext &Ctx) override;

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
  /// Print this VPRegionBlock to \p O (recursively), prefixing all lines with
  /// \p Indent. \p SlotTracker is used to print unnamed VPValue's using
  /// consequtive numbers.
  ///
  /// Note that the numbering is applied to the whole VPlan, so printing
  /// individual regions is consistent with the whole VPlan printing.
  void print(raw_ostream &O, const Twine &Indent,
             VPSlotTracker &SlotTracker) const override;
  using VPBlockBase::print; // Get the print(raw_stream &O) version.
#endif

  /// Clone all blocks in the single-entry single-exit region of the block and
  /// their recipes without updating the operands of the cloned recipes.
  VPRegionBlock *clone() override;

  /// Remove the current region from its VPlan, connecting its predecessor to
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  /// Returns the single VF of the plan, asserting that the plan has exactly
  /// one VF.
  ElementCount getSingleVF() const {
    assert(VFs.size() == 1 && "expected plan with single VF");
    return VFs[0];
  }

  bool hasScalarVFOnly() const {
    bool HasScalarVFOnly = VFs.size() == 1 && VFs[0].isScalar();
    assert(HasScalarVFOnly == hasVF(ElementCount::getFixed(1)) &&
           "Plan with scalar VF should only have a single VF");
    return HasScalarVFOnly;
  }

```
- EN: Core entities appearing here include clearCanonicalIVNUW, VPlan, VF, which carry the local state, declarations, or transformation entry points for this region. The declarations explicitly reference analyses such as DataLayout, SCEV, showing what later implementations must consult to reason about safety or cost. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 此处出现的核心实体包括 clearCanonicalIVNUW, VPlan, VF，它们承载了该区间涉及的局部状态、声明或变换入口。 这些声明显式涉及 DataLayout, SCEV 等分析，表明后续实现需要依赖它们来判断安全性或代价。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

### Lines 4754-4931

```cpp
  bool hasUF(unsigned UF) const { return UFs.empty() || UFs.contains(UF); }

  /// Returns the concrete UF of the plan, after unrolling.
  unsigned getConcreteUF() const {
    assert(UFs.size() == 1 && "Expected a single UF");
    return UFs[0];
  }

  void setUF(unsigned UF) {
    assert(hasUF(UF) && "Cannot set the UF not already in plan");
    UFs.clear();
    UFs.insert(UF);
  }

  /// Returns true if the VPlan already has been unrolled, i.e. it has a single
  /// concrete UF.
  bool isUnrolled() const { return UFs.size() == 1; }

  /// Return a string with the name of the plan and the applicable VFs and UFs.
  std::string getName() const;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  /// The type of the canonical induction variable of the vector loop.
  Type *getIndexType() const { return VF.getType(); }
};

#if !defined(NDEBUG) || defined(LLVM_ENABLE_DUMP)
inline raw_ostream &operator<<(raw_ostream &OS, const VPlan &Plan) {
  Plan.print(OS);
  return OS;
}
#endif

} // end namespace llvm

#endif // LLVM_TRANSFORMS_VECTORIZE_VPLAN_H
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include getConcreteUF, setUF, hasEarlyExit, which carry the local state, declarations, or transformation entry points for this region. The declarations or inline helpers describe how later code will build recipes, update CFG structure, or model IR rewrites.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 getConcreteUF, setUF, hasEarlyExit，它们承载了该区间涉及的局部状态、声明或变换入口。 这里的声明或内联辅助函数描述了后续代码如何构建 recipe、更新 CFG 结构或建模 IR 改写。

## Key Concepts / 关键概念

- EN: Domain: `Vectorize` focuses on vectorization and VPlan infrastructure.
  - CN: 领域：`Vectorize` 主要处理 向量化与 VPlan 基础设施。
- EN: Primary entities: `BasicBlock, DominatorTree, InnerLoopVectorizer, IRBuilderBase, VPTransformState, raw_ostream, RecurrenceDescriptor, SCEV` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`BasicBlock, DominatorTree, InnerLoopVectorizer, IRBuilderBase, VPTransformState, raw_ostream, RecurrenceDescriptor, SCEV` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `DataLayout, DominatorTree, SCEV` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `DataLayout, DominatorTree, SCEV` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Header role: it mostly exposes declarations, contracts, inline helpers, and reusable utilities rather than driving a standalone transform by itself.
  - CN: 头文件角色：它主要暴露声明、约定、内联辅助函数和可复用工具，而不是独立执行某个完整变换。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/IVDescriptors.h`, `llvm/Analysis/MemoryLocation.h`, `llvm/Analysis/VectorUtils.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/IVDescriptors.h`, `llvm/Analysis/MemoryLocation.h`, `llvm/Analysis/VectorUtils.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/DebugLoc.h`, `llvm/IR/FMF.h`, `llvm/IR/Operator.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/DebugLoc.h`, `llvm/IR/FMF.h`, `llvm/IR/Operator.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Compiler.h`, `llvm/Support/InstructionCost.h`, `llvm/ADT/Bitfields.h`, `llvm/ADT/MapVector.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Twine.h`, `llvm/ADT/ilist.h`, `llvm/ADT/ilist_node.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Compiler.h`, `llvm/Support/InstructionCost.h`, `llvm/ADT/Bitfields.h`, `llvm/ADT/MapVector.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallVector.h`, `llvm/ADT/Twine.h`, `llvm/ADT/ilist.h`, `llvm/ADT/ilist_node.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `VPlanValue.h`, `cassert`, `cstddef`, `functional`, `string`, `utility`, `variant` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`VPlanValue.h`, `cassert`, `cstddef`, `functional`, `string`, `utility`, `variant` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `DataLayout`, `DominatorTree`, `SCEV` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`DataLayout`, `DominatorTree`, `SCEV` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
