# NewGVN.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/NewGVN.cpp` | `llvm/lib/Transforms/Scalar/NewGVN.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements global Value Numbering Pass within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 NewGVN 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-293

```cpp
//===- NewGVN.cpp - Global Value Numbering Pass ---------------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
/// \file
/// This file implements the new LLVM's Global Value Numbering pass.
/// GVN partitions values computed by a function into congruence classes.
/// Values ending up in the same congruence class are guaranteed to be the same
/// for every execution of the program. In that respect, congruency is a
/// compile-time approximation of equivalence of values at runtime.
/// The algorithm implemented here uses a sparse formulation and it's based
/// on the ideas described in the paper:
/// "A Sparse Algorithm for Predicated Global Value Numbering" from
/// Karthik Gargi.
///
/// A brief overview of the algorithm: The algorithm is essentially the same as
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
// the value numbering of the stores we just value numbered (because the memory
// congruence class is part of the value number).  So while theoretically
// possible to split them up, it turns out to be *incredibly* complicated to get
// it to work right, because of the interdependency.  While structurally
// slightly messier, it is algorithmically much simpler and faster to do what we
// do here, and track them both at once in the same class.
// Note: The default iterators for this class iterate over values
class CongruenceClass {
public:
  using MemberType = Value;
  using MemberSet = SmallPtrSet<MemberType *, 4>;
  using MemoryMemberType = MemoryPhi;
  using MemoryMemberSet = SmallPtrSet<const MemoryMemberType *, 2>;

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 294-567

```cpp
  explicit CongruenceClass(unsigned ID) : ID(ID) {}
  CongruenceClass(unsigned ID, std::pair<Value *, unsigned int> Leader,
                  const Expression *E)
      : ID(ID), RepLeader(Leader), DefiningExpr(E) {}

  unsigned getID() const { return ID; }

  // True if this class has no members left.  This is mainly used for assertion
  // purposes, and for skipping empty classes.
  bool isDead() const {
    // If it's both dead from a value perspective, and dead from a memory
    // perspective, it's really dead.
    return empty() && memory_empty();
  }

  // Leader functions
  Value *getLeader() const { return RepLeader.first; }
  void setLeader(std::pair<Value *, unsigned int> Leader) {
    RepLeader = std::move(Leader);
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // through the normal user marking we perform.
  mutable DenseMap<const Value *, SmallPtrSet<Value *, 2>> AdditionalUsers;
  DenseMap<const Expression *, SmallPtrSet<Instruction *, 2>>
      ExpressionToPhiOfOps;

  // Map from temporary operation to MemoryAccess.
  DenseMap<const Instruction *, MemoryUseOrDef *> TempToMemory;

  // Set of all temporary instructions we created.
  // Note: This will include instructions that were just created during value
  // numbering.  The way to test if something is using them is to check
  // RealToTemp.
  DenseSet<Instruction *> AllTempInstructions;

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include isDead, setLeader, addPossibleLeader, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis, AssumptionCache, DataLayout, DominatorTree.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 isDead, setLeader, addPossibleLeader，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis, AssumptionCache, DataLayout, DominatorTree 等分析结果。

### Lines 568-839

```cpp
  // This is the set of instructions to revisit on a reachability change.  At
  // the end of the main iteration loop it will contain at least all the phi of
  // ops instructions that will be changed to phis, as well as regular phis.
  // During the iteration loop, it may contain other things, such as phi of ops
  // instructions that used edge reachability to reach a result, and so need to
  // be revisited when the edge changes, independent of whether the phi they
  // depended on changes.
  DenseMap<BasicBlock *, SparseBitVector<>> RevisitOnReachabilityChange;

  // Mapping from predicate info we used to the instructions we used it with.
  // In order to correctly ensure propagation, we must keep track of what
  // comparisons we used, so that when the values of the comparisons change, we
  // propagate the information to the places we used the comparison.
  mutable DenseMap<const Value *, SmallPtrSet<Instruction *, 2>>
      PredicateToUsers;

  // the same reasoning as PredicateToUsers.  When we skip MemoryAccesses for
  // stores, we no longer can rely solely on the def-use chains of MemorySSA.
  mutable DenseMap<const MemoryAccess *, SmallPtrSet<MemoryAccess *, 2>>
      MemoryToUsers;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  const MemoryAccess *lookupMemoryLeader(const MemoryAccess *) const;
  bool isMemoryAccessTOP(const MemoryAccess *) const;

  // Ranking
  unsigned int getRank(const Value *) const;
  bool shouldSwapOperands(const Value *, const Value *) const;
  bool shouldSwapOperandsForPredicate(const Value *, const Value *,
                                      const BitCastInst *I) const;

  // Reachability handling.
  void updateReachableEdge(BasicBlock *, BasicBlock *);
  void processOutgoingEdges(Instruction *, BasicBlock *);
  Value *findConditionEquivalence(Value *) const;

```
- EN: Core entities appearing here include MemoryPhiState, InstCycleState, ExprResult, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AliasAnalysis, AssumptionCache, DataLayout, DominatorTree.
- CN: 此处出现的核心实体包括 MemoryPhiState, InstCycleState, ExprResult，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AliasAnalysis, AssumptionCache, DataLayout, DominatorTree 等分析结果。

### Lines 840-1119

```cpp
  // Elimination.
  struct ValueDFS;
  void convertClassToDFSOrdered(const CongruenceClass &,
                                SmallVectorImpl<ValueDFS> &,
                                DenseMap<const Value *, unsigned int> &,
                                SmallPtrSetImpl<Instruction *> &) const;
  void convertClassToLoadsAndStores(const CongruenceClass &,
                                    SmallVectorImpl<ValueDFS> &) const;

  bool eliminateInstructions(Function &);
  void replaceInstruction(Instruction *, Value *);
  void markInstructionForDeletion(Instruction *);
  void deleteInstructionsInBlock(BasicBlock *);
  Value *findPHIOfOpsLeader(const Expression *, const Instruction *,
                            const BasicBlock *) const;

  // Various instruction touch utilities
  template <typename Map, typename KeyType>
  void touchAndErase(Map &, const KeyType &);
  void markUsersTouched(Value *);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  E->setType(T);
  E->setOpcode(Opcode);
  E->allocateOperands(ArgRecycler, ExpressionAllocator);
  if (Instruction::isCommutative(Opcode)) {
    // Ensure that commutative instructions that only differ by a permutation
    // of their operands get the same value number by sorting the operand value
    // numbers.  Since all commutative instructions have two operands it is more
    // efficient to sort by hand rather than using, say, std::sort.
    if (shouldSwapOperands(Arg1, Arg2))
      std::swap(Arg1, Arg2);
  }
  E->op_push_back(lookupOperandLeader(Arg1));
  E->op_push_back(lookupOperandLeader(Arg2));

```
- EN: Core entities appearing here include ValueDFS, InstrToDFSNum, MemoryToDFSNum, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 ValueDFS, InstrToDFSNum, MemoryToDFSNum，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1120-1395

```cpp
  Value *V = simplifyBinOp(Opcode, E->getOperand(0), E->getOperand(1), Q);
  if (auto Simplified = checkExprResults(E, I, V)) {
    addAdditionalUsers(Simplified, I);
    return Simplified.Expr;
  }
  return E;
}

// Take a Value returned by simplification of Expression E/Instruction
// I, and see if it resulted in a simpler expression. If so, return
// that expression.
NewGVN::ExprResult NewGVN::checkExprResults(Expression *E, Instruction *I,
                                            Value *V) const {
  if (!V)
    return ExprResult::none();

  if (auto *C = dyn_cast<Constant>(V)) {
    if (I)
      LLVM_DEBUG(dbgs() << "Simplified " << *I << " to "
                        << " constant " << *C << "\n");
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
// equivalent to the lattice value "TOP" in most lattices.  This is the initial
// state of all MemoryAccesses.
bool NewGVN::isMemoryAccessTOP(const MemoryAccess *MA) const {
  return getMemoryClass(MA) == TOPClass;
}

LoadExpression *NewGVN::createLoadExpression(Type *LoadType, Value *PointerOp,
                                             LoadInst *LI,
                                             const MemoryAccess *MA) const {
  auto *E =
      new (ExpressionAllocator) LoadExpression(1, LI, lookupMemoryLeader(MA));
  E->allocateOperands(ArgRecycler, ExpressionAllocator);
  E->setType(LoadType);

```
- EN: Core entities appearing here include createExpression, createAggregateValueExpression, createCallExpression, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 createExpression, createAggregateValueExpression, createCallExpression，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1396-1668

```cpp
  // Give store and loads same opcode so they value number together.
  E->setOpcode(0);
  E->op_push_back(PointerOp);

  // TODO: Value number heap versions. We may be able to discover
  // things alias analysis can't on it's own (IE that a store and a
  // load have the same value, and thus, it isn't clobbering the load).
  return E;
}

const StoreExpression *
NewGVN::createStoreExpression(StoreInst *SI, const MemoryAccess *MA) const {
  auto *StoredValueLeader = lookupOperandLeader(SI->getValueOperand());
  auto *E = new (ExpressionAllocator)
      StoreExpression(SI->getNumOperands(), SI, StoredValueLeader, MA);
  E->allocateOperands(ArgRecycler, ExpressionAllocator);
  E->setType(SI->getValueOperand()->getType());

  // Give store and loads same opcode so they value number together.
  E->setOpcode(0);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  if (AA->doesNotAccessMemory(CI)) {
    return ExprResult::some(
        createCallExpression(CI, TOPClass->getMemoryLeader()));
  } else if (AA->onlyReadsMemory(CI)) {
    if (auto *MA = MSSA->getMemoryAccess(CI)) {
      auto *DefiningAccess = MSSAWalker->getClobberingMemoryAccess(MA);
      return ExprResult::some(createCallExpression(CI, DefiningAccess));
    } else // MSSA determined that CI does not access memory.
      return ExprResult::some(
          createCallExpression(CI, TOPClass->getMemoryLeader()));
  }
  return ExprResult::none();
}

```
- EN: Core entities appearing here include createStoreExpression, lookupOperandLeader, getConstantValueForLoad, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 createStoreExpression, lookupOperandLeader, getConstantValueForLoad，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1669-1981

```cpp
// Retrieve the memory class for a given MemoryAccess.
CongruenceClass *NewGVN::getMemoryClass(const MemoryAccess *MA) const {
  auto *Result = MemoryAccessToClass.lookup(MA);
  assert(Result && "Should have found memory class");
  return Result;
}

// Update the MemoryAccess equivalence table to say that From is equal to To,
// and return true if this is different from what already existed in the table.
bool NewGVN::setMemoryClass(const MemoryAccess *From,
                            CongruenceClass *NewClass) {
  assert(NewClass &&
         "Every MemoryAccess should be getting mapped to a non-null class");
  LLVM_DEBUG(dbgs() << "Setting " << *From);
  LLVM_DEBUG(dbgs() << " equivalent to congruence class ");
  LLVM_DEBUG(dbgs() << NewClass->getID()
                    << " with current MemoryAccess leader ");
  LLVM_DEBUG(dbgs() << *NewClass->getMemoryLeader() << "\n");

  auto LookupResult = MemoryAccessToClass.find(From);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
                     CmpInst::getInversePredicate(OurPredicate)) {
            // Inverse predicate, we know the other was false, so this is true.
            return ExprResult::some(
                createConstantExpression(ConstantInt::getTrue(CI->getType())),
                PI);
          }
        }
      }
    }
  }
  // Create expression will take care of simplifyCmpInst
  return createExpression(I);
}

```
- EN: Core entities appearing here include isCycleFree, performSymbolicAggrValueEvaluation, performSymbolicCmpEvaluation, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isCycleFree, performSymbolicAggrValueEvaluation, performSymbolicCmpEvaluation，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1982-2261

```cpp
// Substitute and symbolize the instruction before value numbering.
NewGVN::ExprResult
NewGVN::performSymbolicEvaluation(Instruction *I,
                                  SmallPtrSetImpl<Value *> &Visited) const {

  const Expression *E = nullptr;
  // TODO: memory intrinsics.
  // TODO: Some day, we should do the forward propagation and reassociation
  // parts of the algorithm.
  switch (I->getOpcode()) {
  case Instruction::ExtractValue:
  case Instruction::InsertValue:
    E = performSymbolicAggrValueEvaluation(I);
    break;
  case Instruction::PHI: {
    SmallVector<ValPair, 3> Ops;
    auto *PN = cast<PHINode>(I);
    for (unsigned i = 0; i < PN->getNumOperands(); ++i)
      Ops.push_back({PN->getIncomingValue(i), PN->getIncomingBlock(i)});
    // Sort to ensure the invariant createPHIExpression requires is met.
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Now, fixup the old class if necessary
  if (OldClass->getMemoryLeader() == InstMA) {
    if (!OldClass->definesNoMemory()) {
      OldClass->setMemoryLeader(getNextMemoryLeader(OldClass));
      LLVM_DEBUG(dbgs() << "Memory class leader change for class "
                        << OldClass->getID() << " to "
                        << *OldClass->getMemoryLeader()
                        << " due to removal of old leader " << *InstMA << "\n");
      markMemoryLeaderChangeTouched(OldClass);
    } else
      OldClass->setMemoryLeader(nullptr);
  }
}

```
- EN: Core entities appearing here include touchAndErase, addAdditionalUsers, markUsersTouched, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 touchAndErase, addAdditionalUsers, markUsersTouched，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 2262-2541

```cpp
// Move a value, currently in OldClass, to be part of NewClass
// Update OldClass and NewClass for the move (including changing leaders, etc).
void NewGVN::moveValueToNewCongruenceClass(Instruction *I, const Expression *E,
                                           CongruenceClass *OldClass,
                                           CongruenceClass *NewClass) {
  if (I == OldClass->getNextLeader().first)
    OldClass->resetNextLeader();

  OldClass->erase(I);
  NewClass->insert(I);

  // Ensure that the leader has the lowest RPO. If the leader changed notify all
  // members of the class.
  if (NewClass->getLeader() != I &&
      NewClass->addPossibleLeader({I, InstrToDFSNum(I)})) {
    markValueLeaderChangeTouched(NewClass);
  }

  // Handle our special casing of stores.
  if (auto *SI = dyn_cast<StoreInst>(I)) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        updateReachableEdge(B, TrueSucc);
      } else if (CI->isZero()) {
        LLVM_DEBUG(dbgs() << "Condition for Terminator " << *TI
                          << " evaluated to false\n");
        updateReachableEdge(B, FalseSucc);
      }
    } else {
      updateReachableEdge(B, TrueSucc);
      updateReachableEdge(B, FalseSucc);
    }
  } else if (auto *SI = dyn_cast<SwitchInst>(TI)) {
    // For switches, propagate the case values into the case
    // destinations.

```
- EN: Core entities appearing here include markPhiOfOpsChanged, performCongruenceFinding, updateReachableEdge, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 markPhiOfOpsChanged, performCongruenceFinding, updateReachableEdge，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2542-2834

```cpp
    Value *SwitchCond = SI->getCondition();
    Value *CondEvaluated = findConditionEquivalence(SwitchCond);
    // See if we were able to turn this switch statement into a constant.
    if (CondEvaluated && isa<ConstantInt>(CondEvaluated)) {
      auto *CondVal = cast<ConstantInt>(CondEvaluated);
      // We should be able to get case value for this.
      auto Case = *SI->findCaseValue(CondVal);
      if (Case.getCaseSuccessor() == SI->getDefaultDest()) {
        // We proved the value is outside of the range of the case.
        // We can't do anything other than mark the default dest as reachable,
        // and go home.
        updateReachableEdge(B, SI->getDefaultDest());
        return;
      }
      // Now get where it goes and mark it reachable.
      BasicBlock *TargetBlock = Case.getCaseSuccessor();
      updateReachableEdge(B, TargetBlock);
    } else {
      for (BasicBlock *TargetBlock : successors(SI->getParent()))
        updateReachableEdge(B, TargetBlock);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      // constant.  For anything where that is true, and unsafe, we should
      // have made a phi-of-ops (or value numbered it equivalent to something)
      // for the pieces already.
      FoundVal = !SafeForPHIOfOps ? nullptr
                                  : findLeaderForInst(ValueOp, Visited,
                                                      MemAccess, I, PredBB);
      ValueOp->eraseFromParent();
      if (!FoundVal) {
        // We failed to find a leader for the current ValueOp, but this might
        // change in case of the translated operands change.
        if (SafeForPHIOfOps)
          for (auto *Dep : CurrentDeps)
            addAdditionalUsers(Dep, I);

```
- EN: Core entities appearing here include removePhiOfOps, okayForPHIOfOps, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 removePhiOfOps, okayForPHIOfOps，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2835-3115

```cpp
        return nullptr;
      }
      Deps.insert_range(CurrentDeps);
    } else {
      LLVM_DEBUG(dbgs() << "Skipping phi of ops operand for incoming block "
                        << getBlockName(PredBB)
                        << " because the block is unreachable\n");
      FoundVal = PoisonValue::get(I->getType());
      RevisitOnReachabilityChange[PHIBlock].set(InstrToDFSNum(I));
    }

    PHIOps.push_back({FoundVal, PredBB});
    LLVM_DEBUG(dbgs() << "Found phi of ops operand " << *FoundVal << " in "
                      << getBlockName(PredBB) << "\n");
  }
  for (auto *Dep : Deps)
    addAdditionalUsers(Dep, I);
  sortPHIOps(PHIOps);
  auto *E = performSymbolicPHIEvaluation(PHIOps, I, PHIBlock);
  if (isa<ConstantExpression>(E) || isa<VariableExpression>(E)) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // a class where it is the leader (other things may be equivalent to it, but
  // it needs to start off in its own class, which means it must have been the
  // leader, and it can't have stopped being the leader because it was never
  // removed).
  CongruenceClass *CC =
      AllEqual ? getMemoryClass(AllSameValue) : ensureLeaderOfMemoryClass(MP);
  auto OldState = MemoryPhiState.lookup(MP);
  assert(OldState != MPS_Invalid && "Invalid memory phi state");
  auto NewState = AllEqual ? MPS_Equivalent : MPS_Unique;
  MemoryPhiState[MP] = NewState;
  if (setMemoryClass(MP, CC) || OldState != NewState)
    markMemoryUsersTouched(MP);
}

```
- EN: Core entities appearing here include initializeCongruenceClasses, cleanupTables, updateProcessedCount, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 initializeCongruenceClasses, cleanupTables, updateProcessedCount，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3116-3400

```cpp
// Value number a single instruction, symbolically evaluating, performing
// congruence finding, and updating mappings.
void NewGVN::valueNumberInstruction(Instruction *I) {
  LLVM_DEBUG(dbgs() << "Processing instruction " << *I << "\n");
  if (!I->isTerminator()) {
    const Expression *Symbolized = nullptr;
    SmallPtrSet<Value *, 2> Visited;
    if (DebugCounter::shouldExecute(VNCounter)) {
      auto Res = performSymbolicEvaluation(I, Visited);
      Symbolized = Res.Expr;
      addAdditionalUsers(Res, I);

      // Make a phi of ops if necessary
      if (Symbolized && !isa<ConstantExpression>(Symbolized) &&
          !isa<VariableExpression>(Symbolized) && PHINodeUses.count(I)) {
        auto *PHIE = makePossiblePHIOfOps(I, Visited);
        // If we created a phi of ops, use it.
        // If we couldn't create one, make sure we don't leave one lying around
        if (PHIE) {
          Symbolized = PHIE;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Figure out where touchedinstructions starts
  int FirstInstr = TouchedInstructions.find_first();
  // Nothing set, nothing to iterate, just return.
  if (FirstInstr == -1)
    return;
  const BasicBlock *LastBlock = getBlockForValue(InstrFromDFSNum(FirstInstr));
  while (TouchedInstructions.any()) {
    ++Iterations;
    // Walk through all the instructions in all the blocks in RPO.
    // TODO: As we hit a new block, we should push and pop equalities into a
    // table lookupOperandLeader can use, to catch things PredicateInfo
    // might miss, like edge-only equivalences.
    for (unsigned InstrNum : TouchedInstructions.set_bits()) {

```
- EN: Core entities appearing here include valueNumberInstruction, verifyMemoryCongruency, back_inserter, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 valueNumberInstruction, verifyMemoryCongruency, back_inserter，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3401-3678

```cpp
      // This instruction was found to be dead. We don't bother looking
      // at it again.
      if (InstrNum == 0) {
        TouchedInstructions.reset(InstrNum);
        continue;
      }

      Value *V = InstrFromDFSNum(InstrNum);
      const BasicBlock *CurrBlock = getBlockForValue(V);

      // If we hit a new block, do reachability processing.
      if (CurrBlock != LastBlock) {
        LastBlock = CurrBlock;
        bool BlockReachable = ReachableBlocks.count(CurrBlock);
        const auto &CurrInstRange = BlockInstRange.lookup(CurrBlock);

        // If it's not reachable, erase any touched instructions and move on.
        if (!BlockReachable) {
          TouchedInstructions.reset(CurrInstRange.first, CurrInstRange.second);
          LLVM_DEBUG(dbgs() << "Skipping instructions in block "
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
          continue;
        ValueDFS VDUse;
        // Put the phi node uses in the incoming block.
        BasicBlock *IBlock;
        if (auto *P = dyn_cast<PHINode>(I)) {
          IBlock = P->getIncomingBlock(U);
          // Make phi node users appear last in the incoming block
          // they are from.
          VDUse.LocalNum = InstrDFS.size() + 1;
        } else {
          IBlock = getBlockForValue(I);
          VDUse.LocalNum = InstrToDFSNum(I);
        }

```
- EN: Core entities appearing here include runGVN, sort, NewGVN, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 runGVN, sort, NewGVN，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3679-3951

```cpp
        // Skip uses in unreachable blocks, as we're going
        // to delete them.
        if (!ReachableBlocks.contains(IBlock))
          continue;

        DomTreeNode *DomNode = DT->getNode(IBlock);
        VDUse.DFSIn = DomNode->getDFSNumIn();
        VDUse.DFSOut = DomNode->getDFSNumOut();
        VDUse.U = &U;
        ++UseCount;
        DFSOrderedSet.emplace_back(VDUse);
      }
    }

    // If there are no uses, it's probably dead (but it may have side-effects,
    // so not definitely dead. Otherwise, store the number of uses so we can
    // track if it becomes dead later).
    if (UseCount == 0)
      ProbablyDead.insert(Def);
    else
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // Everything still in the TOP class is unreachable or dead.
    if (CC == TOPClass) {
      for (auto *M : *CC) {
        auto *VTE = ValueToExpression.lookup(M);
        if (VTE && isa<DeadExpression>(VTE))
          markInstructionForDeletion(cast<Instruction>(M));
        assert((!ReachableBlocks.count(cast<Instruction>(M)->getParent()) ||
                InstructionsToErase.count(cast<Instruction>(M))) &&
               "Everything in TOP should be unreachable or dead at this "
               "point");
      }
      continue;
    }

```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include patchAndReplaceAllUsesWith, deleteInstructionsInBlock, markInstructionForDeletion, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 patchAndReplaceAllUsesWith, deleteInstructionsInBlock, markInstructionForDeletion，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 3952-4235

```cpp
    assert(CC->getLeader() && "We should have had a leader");
    // If this is a leader that is always available, and it's a
    // constant or has no equivalences, just replace everything with
    // it. We then update the congruence class with whatever members
    // are left.
    Value *Leader =
        CC->getStoredValue() ? CC->getStoredValue() : CC->getLeader();
    if (alwaysAvailable(Leader)) {
      CongruenceClass::MemberSet MembersLeft;
      for (auto *M : *CC) {
        Value *Member = M;
        // Void things have no uses we can replace.
        if (Member == Leader || !isa<Instruction>(Member) ||
            Member->getType()->isVoidTy()) {
          MembersLeft.insert(Member);
          continue;
        }

        LLVM_DEBUG(dbgs() << "Found replacement " << *(Leader) << " for "
                          << *Member << "\n");
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Prefer poison to undef as it's less defined.
  // Prefer smaller constants to constantexprs
  // Note that the order here matters because of class inheritance
  if (isa<ConstantExpr>(V))
    return 3;
  if (isa<PoisonValue>(V))
    return 1;
  if (isa<UndefValue>(V))
    return 2;
  if (isa<Constant>(V))
    return 0;
  if (auto *A = dyn_cast<Argument>(V))
    return 4 + A->getArgNo();

```
- EN: Core entities appearing here include getRank, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getRank，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 4236-4292

```cpp
  // Need to shift the instruction DFS by number of arguments + 5 to account for
  // the constant and argument ranking above.
  unsigned Result = InstrToDFSNum(V);
  if (Result > 0)
    return 5 + NumFuncArgs + Result;
  // Unreachable or something else, just return a really large number.
  return ~0;
}

// This is a function that says whether two commutative operations should
// have their order swapped when canonicalizing.
bool NewGVN::shouldSwapOperands(const Value *A, const Value *B) const {
  // Because we only care about a total ordering, and don't rewrite expressions
  // in this order, we order by rank, which will give a strict weak ordering to
  // everything but constants, and then we order by pointer address.
  return std::make_pair(getRank(A), A) > std::make_pair(getRank(B), B);
}

bool NewGVN::shouldSwapOperandsForPredicate(const Value *A, const Value *B,
                                            const BitCastInst *I) const {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  auto &AC = AM.getResult<AssumptionAnalysis>(F);
  auto &DT = AM.getResult<DominatorTreeAnalysis>(F);
  auto &TLI = AM.getResult<TargetLibraryAnalysis>(F);
  auto &AA = AM.getResult<AAManager>(F);
  auto &MSSA = AM.getResult<MemorySSAAnalysis>(F).getMSSA();
  bool Changed =
      NewGVN(F, &DT, &AC, &TLI, &AA, &MSSA, F.getDataLayout())
          .runGVN();
  if (!Changed)
    return PreservedAnalyses::all();
  PreservedAnalyses PA;
  PA.preserve<DominatorTreeAnalysis>();
  return PA;
}
```
- EN: Core entities appearing here include shouldSwapOperands, run, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAManager, DataLayout, DominatorTree, MemorySSA.
- CN: 此处出现的核心实体包括 shouldSwapOperands, run，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAManager, DataLayout, DominatorTree, MemorySSA 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `TarjanSCC, CongruenceClass, ExactEqualsExpression, NewGVN, MemoryPhiState, InstCycleState, ExprResult, ValueDFS` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`TarjanSCC, CongruenceClass, ExactEqualsExpression, NewGVN, MemoryPhiState, InstCycleState, ExprResult, ValueDFS` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAManager, AliasAnalysis, AssumptionCache, DataLayout, DominatorTree, MemorySSA, TargetLibraryInfo, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAManager, AliasAnalysis, AssumptionCache, DataLayout, DominatorTree, MemorySSA, TargetLibraryInfo, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `EnableStoreRefinement, EnablePhiOfOps` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `EnableStoreRefinement, EnablePhiOfOps` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/CFGPrinter.h`, `llvm/Analysis/ConstantFolding.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/MemoryBuiltins.h`, `llvm/Analysis/MemorySSA.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/AliasAnalysis.h`, `llvm/Analysis/AssumptionCache.h`, `llvm/Analysis/CFGPrinter.h`, `llvm/Analysis/ConstantFolding.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/InstructionSimplify.h`, `llvm/Analysis/MemoryBuiltins.h`, `llvm/Analysis/MemorySSA.h`, `llvm/Analysis/TargetLibraryInfo.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Argument.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/PatternMatch.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Argument.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/DebugInfo.h`, `llvm/IR/Dominators.h`, `llvm/IR/Function.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/IntrinsicInst.h`, `llvm/IR/PatternMatch.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Allocator.h`, `llvm/Support/ArrayRecycler.h`, `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/DebugCounter.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/PointerLikeTypeTraits.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/ArrayRef.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Allocator.h`, `llvm/Support/ArrayRecycler.h`, `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/DebugCounter.h`, `llvm/Support/ErrorHandling.h`, `llvm/Support/PointerLikeTypeTraits.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/ArrayRef.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `algorithm`, `cassert`, `cstdint`, `iterator`, `map`, `memory`, `set`, `string` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`algorithm`, `cassert`, `cstdint`, `iterator`, `map`, `memory`, `set`, `string` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AAManager`, `AliasAnalysis`, `AssumptionCache`, `DataLayout`, `DominatorTree`, `MemorySSA`, `TargetLibraryInfo`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAManager`, `AliasAnalysis`, `AssumptionCache`, `DataLayout`, `DominatorTree`, `MemorySSA`, `TargetLibraryInfo`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
