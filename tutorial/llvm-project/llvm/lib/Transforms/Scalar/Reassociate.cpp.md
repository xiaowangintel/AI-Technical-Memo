# Reassociate.cpp — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/lib/Transforms/Scalar/Reassociate.cpp` | `llvm/lib/Transforms/Scalar/Reassociate.cpp` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | This source file implements reassociate binary expressions within LLVM's scalar optimizations and loop/CFG transforms layer. | 该源文件实现了 LLVM 标量优化以及循环/CFG 变换层中的 Reassociate 相关逻辑。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-170

```cpp
//===- Reassociate.cpp - Reassociate binary expressions -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This pass reassociates commutative expressions in an order that is designed
// to promote better constant propagation, GCSE, LICM, PRE, etc.
//
// For example: 4 + (x + 5) -> x + (4 + 5)
//
// In the implementation of this algorithm, constants are assigned rank = 0,
// function arguments are rank = 1, and other values are assigned ranks
// corresponding to the reverse post order traversal of current function
// (starting at 2), which effectively gives values in deep loops higher rank
// than values not in loops.
//
//===----------------------------------------------------------------------===//
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  assert(I && isa<FPMathOperator>(I) && "Should only check FP ops");
  return I->hasAllowReassoc() && I->hasNoSignedZeros();
}

/// Return true if V is an instruction of the specified opcode and if it
/// only has one use.
static BinaryOperator *isReassociableOp(Value *V, unsigned Opcode) {
  auto *BO = dyn_cast<BinaryOperator>(V);
  if (BO && BO->hasOneUse() && BO->getOpcode() == Opcode)
    if (!isa<FPMathOperator>(BO) || hasFPAssociativeFlags(BO))
      return BO;
  return nullptr;
}

```
- EN: It sets up the translation unit by importing LLVM IR, analysis, utility, and pass-manager dependencies used later in the file. Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. This portion also defines debug/statistic counters or command-line tuning knobs that control profitability and diagnostics.
- CN: 这里通过导入 LLVM IR、分析、工具以及 pass 管理器相关头文件，为后续实现建立编译单元环境。 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 这一段还定义了调试/统计计数器或命令行调节选项，用来控制收益判定和诊断输出。

### Lines 171-337

```cpp
static BinaryOperator *isReassociableOp(Value *V, unsigned Opcode1,
                                        unsigned Opcode2) {
  auto *BO = dyn_cast<BinaryOperator>(V);
  if (BO && BO->hasOneUse() &&
      (BO->getOpcode() == Opcode1 || BO->getOpcode() == Opcode2))
    if (!isa<FPMathOperator>(BO) || hasFPAssociativeFlags(BO))
      return BO;
  return nullptr;
}

void ReassociatePass::BuildRankMap(Function &F,
                                   ReversePostOrderTraversal<Function*> &RPOT) {
  unsigned Rank = 2;

  // Assign distinct ranks to function arguments.
  for (auto &Arg : F.args()) {
    ValueRankMap[&Arg] = ++Rank;
    LLVM_DEBUG(dbgs() << "Calculated Rank[" << Arg.getName() << "] = " << Rank
                      << "\n");
  }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
/// to something completely different.  Thus if the routine returns 'true' then
/// you MUST either replace I with a new expression computed from the Ops array,
/// or use RewriteExprTree to put the values back in.
///
/// A leaf node is either not a binary operation of the same kind as the root
/// node 'I' (i.e. is not a binary operator at all, or is, but with a different
/// opcode), or is the same kind of binary operator but has a use which either
/// does not belong to the expression, or does belong to the expression but is
/// a leaf node.  Every leaf node has at least one use that is a non-leaf node
/// of the expression, while for non-leaf nodes (except for the root 'I') every
/// use is a non-leaf node of the expression.
///
/// For example:
///           expression graph        node names
```
- EN: Core entities appearing here include getRank, canonicalizeOperands, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 getRank, canonicalizeOperands，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 338-518

```cpp
///
///                     +        |        I
///                    / \       |
///                   +   +      |      A,  B
///                  / \ / \     |
///                 *   +   *    |    C,  D,  E
///                / \ / \ / \   |
///                   +   *      |      F,  G
///
/// The leaf nodes are C, E, F and G.  The Ops array will contain (maybe not in
/// that order) (C, 1), (E, 1), (F, 2), (G, 2).
///
/// The expression is maximal: if some instruction is a binary operator of the
/// same kind as 'I', and all of its uses are non-leaf nodes of the expression,
/// then the instruction also belongs to the expression, is not a leaf node of
/// it, and its operands also belong to the expression (but may be leaf nodes).
///
/// NOTE: This routine will set operands of non-leaf non-root nodes to undef in
/// order to ensure that every non-root node in the expression has *exactly one*
/// use by a non-leaf node of the expression.  This destruction means that the
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        LLVM_DEBUG(dbgs()
                   << "MORPH LEAF: " << *Op << " (" << Weight << ") TO ");
        Instruction *Mul = LowerNegateToMultiply(Neg);
        LLVM_DEBUG(dbgs() << *Mul << '\n');
        Worklist.push_back(std::make_pair(Mul, Weight));
        for (User *U : Mul->users()) {
          if (BinaryOperator *UserBO = dyn_cast<BinaryOperator>(U))
            ToRedo.insert(UserBO);
        }
        ToRedo.insert(Neg);
        Changed = true;
        continue;
      }

```
- EN: Core entities appearing here include match, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout.
- CN: 此处出现的核心实体包括 match，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。

### Lines 519-693

```cpp
      // Failed to morph into an expression of the right type.  This really is
      // a leaf.
      LLVM_DEBUG(dbgs() << "ADD LEAF: " << *Op << " (" << Weight << ")\n");
      assert(!isReassociableOp(Op, Opcode) && "Value was morphed?");
      LeafOrder.push_back(Op);
      Leaves[Op] = Weight;
    }
  }

  // The leaves, repeated according to their weights, represent the linearized
  // form of the expression.
  for (Value *V : LeafOrder) {
    LeafMap::iterator It = Leaves.find(V);
    if (It == Leaves.end())
      // Node initially thought to be a leaf wasn't.
      continue;
    assert(!isReassociableOp(V, Opcode) && "Shouldn't be a leaf!");
    uint64_t Weight = It->second;
    // Ensure the leaf is only output once.
    It->second = 0;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      LLVM_DEBUG(dbgs() << "TO: " << *Op << '\n');
      MadeChange = true;
      ++NumChanged;
    }

    // Now deal with the left-hand side.  If this is already an operation node
    // from the original expression then just rewrite the rest of the expression
    // into it.
    BinaryOperator *BO = isReassociableOp(Op->getOperand(0), Opcode);
    if (BO && !NotRewritable.count(BO)) {
      Op = BO;
      continue;
    }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 694-870

```cpp
    // Otherwise, grab a spare node from the original expression and use that as
    // the left-hand side.  If there are no nodes left then the optimizers made
    // an expression with more nodes than the original!  This usually means that
    // they did something stupid but it might mean that the problem was just too
    // hard (finding the mimimal number of multiplications needed to realize a
    // multiplication expression is NP-complete).  Whatever the reason, smart or
    // stupid, create a new node if there are none left.
    BinaryOperator *NewOp;
    if (NodesToRewrite.empty()) {
      Constant *Poison = PoisonValue::get(I->getType());
      NewOp = BinaryOperator::Create(Instruction::BinaryOps(Opcode), Poison,
                                     Poison, "", I->getIterator());
      if (isa<FPMathOperator>(NewOp))
        NewOp->setFastMathFlags(I->getFastMathFlags());
    } else {
      NewOp = NodesToRewrite.pop_back_val();
    }

    LLVM_DEBUG(dbgs() << "RA: " << *Op << '\n');
    salvageDebugInfo(*Op);
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    ToRedo.insert(TheNeg);
    return TheNeg;
  }

  // Insert a 'neg' instruction that subtracts the value from zero to get the
  // negation.
  Instruction *NewNeg =
      CreateNeg(V, V->getName() + ".neg", BI->getIterator(), BI);
  // NewNeg is generated to potentially replace BI, so use its DebugLoc.
  NewNeg->setDebugLoc(BI->getDebugLoc());
  ToRedo.insert(NewNeg);
  return NewNeg;
}

```
- EN: Core entities appearing here include isReassociableOp, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isReassociableOp，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 871-1048

```cpp
// See if this `or` looks like an load widening reduction, i.e. that it
// consists of an `or`/`shl`/`zext`/`load` nodes only. Note that we don't
// ensure that the pattern is *really* a load widening reduction,
// we do not ensure that it can really be replaced with a widened load,
// only that it mostly looks like one.
static bool isLoadCombineCandidate(Instruction *Or) {
  SmallVector<Instruction *, 8> Worklist;
  SmallPtrSet<Instruction *, 8> Visited;

  auto Enqueue = [&](Value *V) {
    auto *I = dyn_cast<Instruction>(V);
    // Each node of an `or` reduction must be an instruction,
    if (!I)
      return false; // Node is certainly not part of an `or` load reduction.
    // Only process instructions we have never processed before.
    if (Visited.insert(I).second)
      Worklist.emplace_back(I);
    return true; // Will need to look at parent nodes.
  };

// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  // We can safely preserve the nuw flag in all cases.  It's also safe to turn a
  // nuw nsw shl into a nuw nsw mul.  However, nsw in isolation requires special
  // handling.  It can be preserved as long as we're not left shifting by
  // bitwidth - 1.
  bool NSW = cast<BinaryOperator>(Shl)->hasNoSignedWrap();
  bool NUW = cast<BinaryOperator>(Shl)->hasNoUnsignedWrap();
  unsigned BitWidth = Shl->getType()->getScalarSizeInBits();
  if (NSW && (NUW || SA->getValue().ult(BitWidth - 1)))
    Mul->setHasNoSignedWrap(true);
  Mul->setHasNoUnsignedWrap(NUW);
  return Mul;
}

```
- EN: Core entities appearing here include isLoadCombineCandidate, shouldConvertOrWithNoCommonBitsToAdd, ShouldBreakUpSubtract, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 isLoadCombineCandidate, shouldConvertOrWithNoCommonBitsToAdd, ShouldBreakUpSubtract，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1049-1220

```cpp
/// Scan backwards and forwards among values with the same rank as element i
/// to see if X exists.  If X does not exist, return i.  This is useful when
/// scanning for 'x' when we see '-x' because they both get the same rank.
static unsigned FindInOperandList(const SmallVectorImpl<ValueEntry> &Ops,
                                  unsigned i, Value *X) {
  unsigned XRank = Ops[i].Rank;
  unsigned e = Ops.size();
  for (unsigned j = i+1; j != e && Ops[j].Rank == XRank; ++j) {
    if (Ops[j].Op == X)
      return j;
    if (Instruction *I1 = dyn_cast<Instruction>(Ops[j].Op))
      if (Instruction *I2 = dyn_cast<Instruction>(X))
        if (I1->isIdenticalTo(I2))
          return j;
  }
  // Scan backwards.
  for (unsigned j = i-1; j != ~0U && Ops[j].Rank == XRank; --j) {
    if (Ops[j].Op == X)
      return j;
    if (Instruction *I1 = dyn_cast<Instruction>(Ops[j].Op))
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    if (i+1 != Ops.size() && Ops[i+1].Op == Ops[i].Op) {
      if (Opcode == Instruction::And || Opcode == Instruction::Or) {
        // Drop duplicate values for And and Or.
        Ops.erase(Ops.begin()+i);
        --i; --e;
        ++NumAnnihil;
        continue;
      }

      // Drop pairs of values for Xor.
      assert(Opcode == Instruction::Xor);
      if (e == 2)
        return Constant::getNullValue(Ops[0].Op->getType());

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1221-1391

```cpp
      // Y ^ X^X -> Y
      Ops.erase(Ops.begin()+i, Ops.begin()+i+2);
      i -= 1; e -= 2;
      ++NumAnnihil;
    }
  }
  return nullptr;
}

/// Helper function of CombineXorOpnd(). It creates a bitwise-and
/// instruction with the given two operands, and return the resulting
/// instruction. There are two special cases: 1) if the constant operand is 0,
/// it will return NULL. 2) if the constant is ~0, the symbolic operand will
/// be returned.
static Value *createAndInstr(BasicBlock::iterator InsertBefore, Value *Opnd,
                             const APInt &ConstOpnd) {
  if (ConstOpnd.isZero())
    return nullptr;

  if (ConstOpnd.isAllOnes())
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Step 1: Convert ValueEntry to XorOpnd
  for (const ValueEntry &Op : Ops) {
    Value *V = Op.Op;
    const APInt *C;
    // TODO: Support non-splat vectors.
    if (match(V, m_APInt(C))) {
      ConstOpnd ^= *C;
    } else {
      XorOpnd O(V);
      O.setSymbolicRank(getRank(O.getSymbolicPart()));
      Opnds.push_back(O);
    }
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1392-1568

```cpp
  // NOTE: From this point on, do *NOT* add/delete element to/from "Opnds".
  //  It would otherwise invalidate the "Opnds"'s iterator, and hence invalidate
  //  the "OpndPtrs" as well. For the similar reason, do not fuse this loop
  //  with the previous loop --- the iterator of the "Opnds" may be invalidated
  //  when new elements are added to the vector.
  for (XorOpnd &Op : Opnds)
    OpndPtrs.push_back(&Op);

  // Step 2: Sort the Xor-Operands in a way such that the operands containing
  //  the same symbolic value cluster together. For instance, the input operand
  //  sequence ("x | 123", "y & 456", "x & 789") will be sorted into:
  //  ("x | 123", "x & 789", "y & 456").
  //
  //  The purpose is twofold:
  //  1) Cluster together the operands sharing the same symbolic-value.
  //  2) Operand having smaller symbolic-value-rank is permuted earlier, which
  //     could potentially shorten crital path, and expose more loop-invariants.
  //     Note that values' rank are basically defined in RPO order (FIXME).
  //     So, if Rank(X) < Rank(Y) < Rank(Z), it means X is defined earlier
  //     than Y which is defined earlier than Z. Permute "x | 1", "Y & 2",
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
    // Remove X and ~X from the operand list.
    if (Ops.size() == 2 && match(TheOp, m_Not(m_Value())))
      return Constant::getAllOnesValue(X->getType());

    Ops.erase(Ops.begin()+i);
    if (i < FoundX)
      --FoundX;
    else
      --i;   // Need to back up an extra one.
    Ops.erase(Ops.begin()+FoundX);
    ++NumAnnihil;
    --i;     // Revisit element.
    e -= 2;  // Removed two elements.

```
- EN: Core entities appearing here include stable_sort, CombineXorOpnd, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 stable_sort, CombineXorOpnd，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1569-1743

```cpp
    // if X and ~X we append -1 to the operand list.
    if (match(TheOp, m_Not(m_Value()))) {
      Value *V = Constant::getAllOnesValue(X->getType());
      Ops.insert(Ops.end(), ValueEntry(getRank(V), V));
      e += 1;
    }
  }

  // Scan the operand list, checking to see if there are any common factors
  // between operands.  Consider something like A*A+A*B*C+D.  We would like to
  // reassociate this to A*(A+B*C)+D, which reduces the number of multiplies.
  // To efficiently find this, we count the number of times a factor occurs
  // for any ADD operands that are MULs.
  DenseMap<Value*, unsigned> FactorOccurrences;

  // Keep track of each multiply we see, to avoid triggering on (X*4)+(X*4)
  // where they are actually the same multiply.
  unsigned MaxOcc = 0;
  Value *MaxOccVal = nullptr;
  for (const ValueEntry &Op : Ops) {
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  // Compute the sum of powers of simplifiable factors.
  unsigned FactorPowerSum = 0;
  for (unsigned Idx = 1, Size = Ops.size(); Idx < Size; ++Idx) {
    Value *Op = Ops[Idx-1].Op;

    // Count the number of occurrences of this value.
    unsigned Count = 1;
    for (; Idx < Size && Ops[Idx].Op == Op; ++Idx)
      ++Count;
    // Track for simplification all factors which occur 2 or more times.
    if (Count > 1)
      FactorPowerSum += Count;
  }

```
- EN: The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1744-1916

```cpp
  // We can only simplify factors if the sum of the powers of our simplifiable
  // factors is 4 or higher. When that is the case, we will *always* have
  // a simplification. This is an important invariant to prevent cyclicly
  // trying to simplify already minimal formations.
  if (FactorPowerSum < 4)
    return false;

  // Now gather the simplifiable factors, removing them from Ops.
  FactorPowerSum = 0;
  for (unsigned Idx = 1; Idx < Ops.size(); ++Idx) {
    Value *Op = Ops[Idx-1].Op;

    // Count the number of occurrences of this value.
    unsigned Count = 1;
    for (; Idx < Ops.size() && Ops[Idx].Op == Op; ++Idx)
      ++Count;
    if (Count == 1)
      continue;
    // Move an even number of occurrences to Factors.
    Count &= ~1U;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        continue;
      }
      if (Constant *Res = ConstantFoldBinaryOpOperands(Opcode, C, Cst, DL)) {
        Ops.pop_back();
        Cst = Res;
        continue;
      }
    }
    break;
  }
  // If there was nothing but constants then we are done.
  if (Ops.empty())
    return Cst;

```
- EN: Core entities appearing here include stable_sort, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 stable_sort，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 1917-2101

```cpp
  // Put the combined constant back at the end of the operand list, except if
  // there is no point.  For example, an add of 0 gets dropped here, while a
  // multiplication by zero turns the whole expression into zero.
  if (Cst && Cst != ConstantExpr::getBinOpIdentity(Opcode, I->getType())) {
    if (Cst == ConstantExpr::getBinOpAbsorber(Opcode, I->getType()))
      return Cst;
    Ops.push_back(ValueEntry(0, Cst));
  }

  if (Ops.size() == 1) return Ops[0].Op;

  // Handle destructive annihilation due to identities between elements in the
  // argument list here.
  unsigned NumOps = Ops.size();
  switch (Opcode) {
  default: break;
  case Instruction::And:
  case Instruction::Or:
    if (Value *Result = OptimizeAndOrXor(Opcode, Ops))
      return Result;
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
      assert(C->isNegative() && "Expected negative FP constant");
      Negatible->setOperand(0, ConstantFP::get(Negatible->getType(), abs(*C)));
      MadeChange = true;
    }
    if (match(Negatible->getOperand(1), m_APFloat(C))) {
      assert(!match(Negatible->getOperand(0), m_Constant()) &&
             "Expecting only 1 constant operand");
      assert(C->isNegative() && "Expected negative FP constant");
      Negatible->setOperand(1, ConstantFP::get(Negatible->getType(), abs(*C)));
      MadeChange = true;
    }
  }
  assert(MadeChange == true && "Negative constant candidate was not changed");

```
- EN: Core entities appearing here include EraseInst, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 EraseInst，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2102-2271

```cpp
  // Negations cancelled out.
  if (Candidates.size() % 2 == 0)
    return I;

  // Negate the final operand in the expression by flipping the opcode of this
  // fadd/fsub.
  assert(Candidates.size() % 2 == 1 && "Expected odd number");
  IRBuilder<> Builder(I);
  Value *NewInst = IsFSub ? Builder.CreateFAddFMF(OtherOp, Op, I)
                          : Builder.CreateFSubFMF(OtherOp, Op, I);
  I->replaceAllUsesWith(NewInst);
  RedoInsts.insert(I);
  return dyn_cast<Instruction>(NewInst);
}

/// Canonicalize expressions that contain a negative floating-point constant
/// of the following form:
///   OtherOp + (subtree) -> OtherOp {+/-} (canonical subtree)
///   (subtree) + OtherOp -> OtherOp {+/-} (canonical subtree)
///   OtherOp - (subtree) -> OtherOp {+/-} (canonical subtree)
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...

  // If this is an interior node of a reassociable tree, ignore it until we
  // get to the root of the tree, to avoid N^2 analysis.
  unsigned Opcode = BO->getOpcode();
  if (BO->hasOneUse() && BO->user_back()->getOpcode() == Opcode) {
    // During the initial run we will get to the root of the tree.
    // But if we get here while we are redoing instructions, there is no
    // guarantee that the root will be visited. So Redo later
    if (BO->user_back() != BO &&
        BO->getParent() == BO->user_back()->getParent())
      RedoInsts.insert(BO->user_back());
    return;
  }

```
- EN: Core entities appearing here include OptimizeInst, isReassociableOp, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as DataLayout. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 OptimizeInst, isReassociableOp，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 DataLayout 等分析结果。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2272-2447

```cpp
  // If this is an add tree that is used by a sub instruction, ignore it
  // until we process the subtract.
  if (BO->hasOneUse() && BO->getOpcode() == Instruction::Add &&
      cast<Instruction>(BO->user_back())->getOpcode() == Instruction::Sub)
    return;
  if (BO->hasOneUse() && BO->getOpcode() == Instruction::FAdd &&
      cast<Instruction>(BO->user_back())->getOpcode() == Instruction::FSub)
    return;

  ReassociateExpression(BO);
}

void ReassociatePass::ReassociateExpression(BinaryOperator *I) {
  // First, walk the expression tree, linearizing the tree, collecting the
  // operand information.
  SmallVector<RepeatedValue, 8> Tree;
  OverflowTracking Flags;
  MadeChange |= LinearizeExprTree(I, Tree, RedoInsts, Flags);
  SmallVector<ValueEntry, 8> Ops;
  Ops.reserve(Tree.size());
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
        Value *Op1 = Ops[j].Op;
        if (std::less<Value *>()(Op1, Op0))
          std::swap(Op0, Op1);
        auto it = PairMap[Idx].find({Op0, Op1});
        if (it != PairMap[Idx].end()) {
          // Functions like BreakUpSubtract() can erase the Values we're using
          // as keys and create new Values after we built the PairMap. There's a
          // small chance that the new nodes can have the same address as
          // something already in the table. We shouldn't accumulate the stored
          // score in that case as it refers to the wrong Value.
          if (it->second.isValid())
            Score += it->second.Score;
        }

```
- EN: Core entities appearing here include ReassociateExpression, which carry the local state, declarations, or transformation entry points for this region. The implementation here performs or prepares concrete IR rewrites, CFG updates, cloning, hoisting/sinking, or vectorization bookkeeping.
- CN: 此处出现的核心实体包括 ReassociateExpression，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段会执行或准备具体的 IR 改写、CFG 更新、克隆、提升/下沉，或向量化记账逻辑。

### Lines 2448-2620

```cpp
        unsigned MaxRank = std::max(Ops[i].Rank, Ops[j].Rank);

        // By construction, the operands are sorted in reverse order of their
        // topological order.
        // So we tend to form (sub) expressions with values that are close to
        // each other.
        //
        // Now to expose more CSE opportunities we want to expose the pair of
        // operands that occur the most (as statically computed in
        // BuildPairMap.) as the first sub-expression.
        //
        // If two pairs occur as many times, we pick the one with the
        // lowest rank, meaning the one with both operands appearing first in
        // the topological order.
        if (Score > Max || (Score == Max && MaxRank < BestRank)) {
          BestPair = {j, i};
          Max = Score;
          BestRank = MaxRank;
        }
      }
// ... excerpt omitted for brevity / 为简洁省略中间代码 ...
  }

  // We are done with the rank map and pair map.
  RankMap.clear();
  ValueRankMap.clear();
  for (auto &Entry : PairMap)
    Entry.clear();

  if (MadeChange) {
    PreservedAnalyses PA;
    PA.preserveSet<CFGAnalyses>();
    return PA;
  }

```
- EN: Core entities appearing here include BuildPairMap, run, which carry the local state, declarations, or transformation entry points for this region.
- CN: 此处出现的核心实体包括 BuildPairMap, run，它们承载了该区间涉及的局部状态、声明或变换入口。

### Lines 2621-2663

```cpp
  return PreservedAnalyses::all();
}

namespace {

class ReassociateLegacyPass : public FunctionPass {
  ReassociatePass Impl;

public:
  static char ID; // Pass identification, replacement for typeid

  ReassociateLegacyPass() : FunctionPass(ID) {
    initializeReassociateLegacyPassPass(*PassRegistry::getPassRegistry());
  }

  bool runOnFunction(Function &F) override {
    if (skipFunction(F))
      return false;

    FunctionAnalysisManager DummyFAM;
    auto PA = Impl.run(F, DummyFAM);
    return !PA.areAllPreserved();
  }

  void getAnalysisUsage(AnalysisUsage &AU) const override {
    AU.setPreservesCFG();
    AU.addPreserved<AAResultsWrapperPass>();
    AU.addPreserved<BasicAAWrapperPass>();
    AU.addPreserved<GlobalsAAWrapperPass>();
  }
};

} // end anonymous namespace

char ReassociateLegacyPass::ID = 0;

INITIALIZE_PASS(ReassociateLegacyPass, "reassociate",
                "Reassociate expressions", false, false)

// Public interface to the Reassociate pass
FunctionPass *llvm::createReassociatePass() {
  return new ReassociateLegacyPass();
}
```
- EN: Namespace blocks organize the helpers into LLVM-owned APIs and keep internal names scoped correctly. Core entities appearing here include ReassociateLegacyPass, which carry the local state, declarations, or transformation entry points for this region. Legality, profitability, or canonical-form checks in this region depend on analyses such as AAResults.
- CN: 命名空间块把辅助逻辑组织到 LLVM 自身的 API 中，并保持内部名称的作用域清晰。 此处出现的核心实体包括 ReassociateLegacyPass，它们承载了该区间涉及的局部状态、声明或变换入口。 这一段中的合法性、收益性或规范形态检查依赖 AAResults 等分析结果。

## Key Concepts / 关键概念

- EN: Domain: `Scalar` focuses on scalar optimizations and loop/CFG transforms.
  - CN: 领域：`Scalar` 主要处理 标量优化以及循环/CFG 变换。
- EN: Primary entities: `llvm, ReassociateLegacyPass, PrintOps, XorOpnd, hasFPAssociativeFlags, getRank, canonicalizeOperands, match` define the public API surface and/or the major implementation hooks in this file.
  - CN: 核心实体：`llvm, ReassociateLegacyPass, PrintOps, XorOpnd, hasFPAssociativeFlags, getRank, canonicalizeOperands, match` 构成该文件对外 API 与主要实现挂钩。
- EN: Analysis coupling: it cooperates with `AAResults, AliasAnalysis, DataLayout, ValueTracking` to prove legality, preserve canonical forms, understand memory/control flow, or estimate profitability.
  - CN: 分析耦合：它与 `AAResults, AliasAnalysis, DataLayout, ValueTracking` 等分析协同，用来证明合法性、维持规范形态、理解内存/控制流或评估收益。
- EN: Tuning surface: command-line knobs such as `UseCSELocalOpt` expose heuristics or diagnostics to developers.
  - CN: 调节界面：诸如 `UseCSELocalOpt` 之类的命令行开关向开发者暴露了启发式和诊断控制点。
- EN: Pass-manager integration: the file contains hooks for LLVM's new and/or legacy pass managers, including analysis-preservation bookkeeping.
  - CN: Pass 管理器集成：该文件包含 LLVM 新/旧 pass 管理器的挂钩，并负责保留分析结果的记账。

## Dependencies / 依赖关系

- EN: Analysis headers: `llvm/Analysis/BasicAliasAnalysis.h`, `llvm/Analysis/ConstantFolding.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/ValueTracking.h` provide data-flow, alias, loop, dominance, or cost models consumed by this file.
  - CN: 分析头文件：`llvm/Analysis/BasicAliasAnalysis.h`, `llvm/Analysis/ConstantFolding.h`, `llvm/Analysis/GlobalsModRef.h`, `llvm/Analysis/ValueTracking.h` 提供了该文件会使用到的数据流、别名、循环、支配关系或代价模型。
- EN: IR/transform headers: `llvm/IR/Argument.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/Operator.h`, `llvm/IR/PassManager.h` expose the IR objects and transformation utilities that this code manipulates or extends.
  - CN: IR/变换头文件：`llvm/IR/Argument.h`, `llvm/IR/BasicBlock.h`, `llvm/IR/CFG.h`, `llvm/IR/Constant.h`, `llvm/IR/Constants.h`, `llvm/IR/Function.h`, `llvm/IR/IRBuilder.h`, `llvm/IR/InstrTypes.h`, `llvm/IR/Instruction.h`, `llvm/IR/Instructions.h`, `llvm/IR/Operator.h`, `llvm/IR/PassManager.h` 暴露了此处要操作或扩展的 IR 对象与变换工具。
- EN: Utility headers: `llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/APFloat.h`, `llvm/ADT/APInt.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallSet.h` contribute containers, diagnostics, casting, hashing, and other infrastructure services.
  - CN: 工具头文件：`llvm/Support/Casting.h`, `llvm/Support/CommandLine.h`, `llvm/Support/Debug.h`, `llvm/Support/raw_ostream.h`, `llvm/ADT/APFloat.h`, `llvm/ADT/APInt.h`, `llvm/ADT/DenseMap.h`, `llvm/ADT/PostOrderIterator.h`, `llvm/ADT/SmallPtrSet.h`, `llvm/ADT/SmallSet.h` 提供了容器、诊断、类型转换、哈希等基础设施能力。
- EN: Standard/other headers: `algorithm`, `cassert`, `utility` supply C++ language utilities used alongside LLVM APIs.
  - CN: 标准库/其他头文件：`algorithm`, `cassert`, `utility` 为 LLVM API 之外的 C++ 语言工具提供支持。
- EN: Runtime analysis dependencies: `AAResults`, `AliasAnalysis`, `DataLayout`, `ValueTracking` appear in declarations or implementation paths that prove safety, canonicality, or profitability.
  - CN: 运行期分析依赖：`AAResults`, `AliasAnalysis`, `DataLayout`, `ValueTracking` 出现在声明或实现路径中，用来证明安全性、规范性或收益性。
